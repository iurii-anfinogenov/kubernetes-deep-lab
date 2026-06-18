# Lab 06 - Разбор kubeadm-кластера после установки

## Цель

Разобрать уже созданный Kubernetes cluster и понять, что именно подготовил `kubeadm`.

В этой лабораторной мы не устанавливаем Kubernetes заново и не сравниваем разные способы установки. Сначала нужно понять текущий cluster:

- какие nodes входят в cluster;
- какие system pods запущены;
- где находятся компоненты control plane;
- почему control plane запущен как static pods;
- как kubelet связан с containerd;
- где лежат kubeconfig-файлы;
- где лежат certificates;
- как проверить базовое состояние cluster после установки.

## Предварительные условия

Перед началом должны быть выполнены предыдущие labs:

- Lab 00 - Environment Validation;
- Lab 01 - Node Baseline;
- Lab 02 - Container Runtime: containerd;
- Lab 03 - kubeadm prerequisites;
- Lab 04 - kubeadm cluster bootstrap;
- Lab 05 - CNI bootstrap.

Ожидаемое состояние:

- есть 1 control-plane node;
- есть 2 worker nodes;
- `kubectl` настроен;
- все nodes находятся в состоянии `Ready`;
- CNI установлен;
- smoke tests прошли успешно.

## Что важно понять

`kubeadm` не является отдельным runtime или постоянным управляющим компонентом Kubernetes.

`kubeadm` выполняет bootstrap cluster:

- генерирует certificates;
- создает kubeconfig-файлы;
- создает static pod manifests для control plane;
- настраивает kubelet;
- подготавливает bootstrap tokens;
- помогает присоединить worker nodes.

После установки основную работу выполняют:

- `kubelet`;
- `containerd`;
- control plane components;
- CNI;
- kube-proxy, если он используется в выбранной сетевой схеме.

## Шаг - проверить nodes

Выполнить на машине, где настроен `kubectl`:

```bash
kubectl get nodes -o wide
```
Ожидаем:

все nodes в состоянии Ready;
видны роли nodes;
видны версии Kubernetes;
видны internal IP;
container runtime показывает containerd.

**Важно:**

Ready означает, что kubelet на node успешно отчитывается в API Server;
ROLES - это label, а не отдельный системный режим node;
worker node с ролью <none> - нормальное состояние для kubeadm cluster, если label роли не был добавлен отдельно.
## Шаг - проверить system pods

Выполнить:
```sh
kubectl -n kube-system get pods -o wide
```
Ожидаем увидеть:
```txt
kube-apiserver-*;
kube-controller-manager-*;
kube-scheduler-*;
etcd-*;
coredns-*;
CNI pods;
kube-proxy-*, если kube-proxy используется.
```
Важно:

control plane components в kubeadm cluster обычно запущены как static pods на control-plane node.

Это значит:

они описаны локальными manifest-файлами на node;
kubelet читает эти manifests;
kubelet просит container runtime запустить containers;
API Server потом видит эти pods как обычные pods, но управляет ими не scheduler.  
## Шаг - посмотреть static pod manifests

Выполнить на control-plane node:
```sh
sudo ls -l /etc/kubernetes/manifests/
```
Ожидаем файлы:
```txt
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```
Эти файлы являются источником правды для static pods control plane.

Важно:

их не создавал scheduler;
они не зависят от Deployment или ReplicaSet;
kubelet следит за этой директорией;
изменение файла manifest приводит к пересозданию соответствующего static pod.

Риск изменения этих файлов: HIGH.

В этой lab мы их только читаем. Не редактировать.

## Шаг - прочитать manifest API Server

Выполнить на control-plane node:
```sh
sudo sed -n '1,220p' /etc/kubernetes/manifests/kube-apiserver.yaml
```
Нужно найти и зафиксировать:

```txt
container image;
command;
bind address;
secure port;
paths к certificates;
путь к etcd certificates;
admission plugins;
service cluster IP range;
client CA file.
```
Обратить внимание на параметры:
```txt
--advertise-address
--secure-port
--etcd-servers
--client-ca-file
--tls-cert-file
--tls-private-key-file
--service-cluster-ip-range
--authorization-mode
--enable-admission-plugins
```
## Шаг - прочитать manifest etcd

```sh
sudo sed -n '1,220p' /etc/kubernetes/manifests/etcd.yaml
```
Нужно найти и зафиксировать:

* где хранится data directory;
* какие certificates использует etcd;
* на каких адресах слушает etcd;
* как API Server подключается к etcd.

Обратить внимание на параметры:
```txt
--data-dir
--listen-client-urls
--advertise-client-urls
--cert-file
--key-file
--trusted-ca-file
```
Важно:
```txt
etcd - это key-value storage всего cluster state.
```
> Если etcd недоступен, API Server не сможет нормально работать с состоянием cluster.

## Шаг - посмотреть kubeconfig-файлы

Выполнить на control-plane node:
```sh
sudo ls -l /etc/kubernetes/*.conf
```
Ожидаем:
```txt
admin.conf
controller-manager.conf
kubelet.conf
scheduler.conf
```
Назначение:
```txt
admin.conf - kubeconfig администратора cluster;
controller-manager.conf - kubeconfig для kube-controller-manager;
scheduler.conf - kubeconfig для kube-scheduler;
kubelet.conf - kubeconfig для kubelet на этой node.
```
Важно:
```txt
kubeconfig - это не просто "настройка kubectl".
```
Это файл, который описывает:

* куда подключаться;
* каким пользователем;
* каким certificate/key или token;
* какой cluster context использовать.

## Шаг - посмотреть certificates

Выполнить на control-plane node:
```sh
sudo find /etc/kubernetes/pki -maxdepth 2 -type f | sort
```
Ожидаем увидеть certificates и keys для:
```txt
cluster CA;
API Server;
API Server kubelet client;
front-proxy;
etcd;
service account keys.
```
Важно:

не копировать private keys в публичные отчеты.  

В lab journal можно фиксировать только имена файлов, назначение и сроки действия certificates.

## Шаг - проверить сроки certificates

Выполнить на control-plane node:
```sh
sudo kubeadm certs check-expiration
```
Нужно зафиксировать:

* какие certificates есть;
* когда они истекают;
* какие certificates управляются kubeadm;
* какие certificates относятся к external CA, если такая схема используется.

## Шаг - проверить kubelet

Выполнить последовательно на каждой node:
```sh
systemctl status kubelet --no-pager
```
Нужно понять:

kubelet запущен через systemd;
kubelet регистрирует node в API Server;
kubelet работает с container runtime;
kubelet отвечает за pods на своей node.

Важно:

kubelet не запускает containers напрямую. Он обращается к container runtime через CRI.

В нашем случае runtime - containerd.

## Шаг - проверить container runtime через crictl

Выполнить последовательно на каждой node:
```sh
sudo crictl pods
sudo crictl ps
```
На control-plane node должны быть видны pods и containers control plane.

На worker nodes должны быть видны containers CNI, kube-proxy, CoreDNS или workload pods, если они там запущены.

Важно:

crictl показывает состояние глазами CRI/runtime layer, а не глазами API Server.

## Шаг - сопоставить Kubernetes и runtime layer

Выполнить через kubectl:
```sh
kubectl -n kube-system get pods -o wide
```
Затем на node, где расположен выбранный pod, выполнить:
```sh
sudo crictl pods
sudo crictl ps
```
Задача:

сопоставить один pod из kubectl с pod/container на runtime layer.

Важно понять разницу:

* kubectl показывает объект через API Server;
* crictl показывает то, что реально видит container runtime на node;
* kubelet связывает эти два мира.

### Схема
```txt
kubectl
  |
  v
API Server
  |
  v
etcd

control-plane static pods:
  /etc/kubernetes/manifests/
        |
        v
      kubelet
        |
        v
      containerd
        |
        v
      runc / Linux kernel
```  

На worker node:  

```txt      
API Server
  |
  v
kubelet on worker
  |
  v
containerd
  |
  v
pod containers
  |
  v
CNI network
```
Что записать в lab journal  

Зафиксировать:  

* вывод kubectl get nodes -o wide;
* вывод kubectl -n kube-system get pods -o wide;
* список файлов из /etc/kubernetes/manifests/;
* какие static pods есть;
* список kubeconfig-файлов;
* список certificates без private key content;
* результат kubeadm certs check-expiration;
* статус kubelet на каждой node;
* пример сопоставления одного Kubernetes pod с crictl pods и crictl ps;
* краткий вывод своими словами: что сделал kubeadm и что теперь делает kubelet.
* Контрольные вопросы
* Почему control plane components в kubeadm cluster запущены как static pods?
* Кто запускает kube-apiserver после reboot control-plane node?
* Чем static pod отличается от pod, созданного Deployment?
* Где kubeadm хранит manifests control plane?
* Где лежат основные certificates cluster?
* Что такое kubeconfig и почему он нужен не только для kubectl?
* Как kubelet связан с containerd?
* Чем отличается вывод kubectl get pods от crictl ps?
* Что произойдет, если kubelet на control-plane node остановится?
* Почему private keys нельзя публиковать в lab journal?
* Критерии готовности

Lab считается выполненной, если участник:

* показал, что cluster находится в рабочем состоянии;
* нашел static pod manifests;
* объяснил, почему control plane запущен через static pods;
* нашел kubeconfig-файлы;
* нашел certificates и не раскрыл private keys;
* проверил сроки certificates;
* проверил kubelet через systemd и journal;
* сопоставил pod из Kubernetes API с pod/container через crictl;
* написал краткий вывод своими словами.