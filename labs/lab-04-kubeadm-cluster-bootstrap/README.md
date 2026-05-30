# Lab 04 - kubeadm cluster bootstrap

## Цель

Создать первый Kubernetes cluster через `kubeadm`.

В этой лабораторной мы:

- выполним preflight-проверки `kubeadm`;
- выполним `kubeadm init` на control-plane node;
- настроим `kubectl` для обычного пользователя;
- получим `kubeadm join` command;
- подключим worker nodes к cluster;
- проверим static pods control plane;
- проверим состояние nodes до установки CNI.

CNI будет установлен отдельно в `lab-05-cni-bootstrap`.

## Где выполнять

Команды разделены по месту выполнения:

- control-plane node: `kdl-cp-1`;
- worker nodes: `kdl-w-5`, `kdl-w-6`.

Если действие одинаковое для worker nodes, оно выполняется последовательно на каждом worker.

## Исходные параметры lab-кластера

Для этого reference run используем:

```text
control-plane node: kdl-cp-1
control-plane IP:   192.168.20.11
worker nodes:       kdl-w-5, kdl-w-6
Kubernetes:         v1.36.1
container runtime:  containerd
pod CIDR:           192.168.0.0/16
service CIDR:       default kubeadm value
CNI:                будет установлен в Lab 05
```

`pod CIDR` указываем уже на этапе `kubeadm init`, потому что в следующей лабораторной будет установлен CNI, которому нужен согласованный диапазон Pod network.

## Предварительные требования

Перед началом должны быть завершены:

- `lab-00-environment`;
- `lab-01-node-baseline`;
- `lab-02-containerd-runtime`;
- `lab-03-kubeadm-prerequisites`.

На всех nodes должны быть готовы:

- `containerd` работает;
- `crictl` настроен;
- `RuntimeReady: true`;
- `SystemdCgroup: true`;
- swap отключен;
- `kubeadm`, `kubelet`, `kubectl` установлены;
- версии Kubernetes packages совпадают;
- `kubeadm`, `kubelet`, `kubectl` находятся в `apt-mark hold`.

## Проверка состояния перед bootstrap

Команды ничего не меняют.

Выполнить на каждой node:

```bash
hostname
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
swapon --show
sudo crictl info | jq '.status.conditions'
```

Ожидаем:

- `kubeadm`, `kubelet`, `kubectl` имеют версию `v1.36.1`;
- `swapon --show` не выводит активный swap;
- `RuntimeReady` имеет `status: true`;
- `NetworkReady` может иметь `status: false` с причиной `NetworkPluginNotReady`.

`NetworkReady: false` до установки CNI - ожидаемое состояние.

## Проверка IP control-plane node

Команды ничего не меняют.

Выполнить на control-plane node:

```bash
hostname
ip -br addr
ip route
```

Убедиться, что IP control-plane node:

```text
192.168.20.11
```

Если IP отличается, не продолжайте `kubeadm init`. Сначала зафиксируйте актуальный IP и замените его в командах ниже.

## Проверка, что cluster еще не был инициализирован


Команды ничего не меняют.

Выполнить на control-plane node:

```bash
ls -la /etc/kubernetes || true
ls -la /etc/kubernetes/manifests || true
ls -la /var/lib/etcd || true
```

Ожидаем для чистой node:

- `/etc/kubernetes` может отсутствовать или быть пустым;
- `/etc/kubernetes/manifests` не содержит static pod manifests;
- `/var/lib/etcd` отсутствует или пустой.

Если там уже есть файлы от прошлого `kubeadm init`, не продолжайте. Нужно отдельно разобрать состояние и только потом выполнять reset.

## kubeadm preflight checks


Команда выполняет проверки, но не создает cluster.

## kubeadm init dry-run

Команда показывает, что будет сделано, но не применяет изменения.

Выполнить на control-plane node:

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.20.11 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1 \
  --dry-run
```

Ожидаем:

- preflight checks проходят;
- kubeadm показывает план bootstrap;
- нет фатальных ошибок.

Если dry-run прошел успешно, можно переходить к настоящему `kubeadm init`.

## kubeadm init

Что будет изменено:

- будут созданы certificates в `/etc/kubernetes/pki`;
- будут созданы kubeconfig files в `/etc/kubernetes`;
- будут созданы static pod manifests в `/etc/kubernetes/manifests`;
- kubelet начнет запускать control plane static pods;
- будет создан local etcd;
- будет создан bootstrap token;
- будут созданы базовые addons, включая CoreDNS и kube-proxy.

Важно: CoreDNS может быть `Pending` до установки CNI. Это ожидаемо.

Выполнить на control-plane node:

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.20.11 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1
```

После успешного выполнения сохраните из вывода `kubeadm join ...` command. Он понадобится для worker nodes.
Пример:
```sh
kubeadm join 192.168.20.11:6443 --token 1kfg5z.ytvmbpwy3vh02l58 \
        --discovery-token-ca-cert-hash sha256:4389e7667fdb6a132e9a91b409fd524b26902f8bcc65d6d427f146dfe61720d5 
```        

Если забыли сохранить join command, его можно получить позже командой:

```bash
sudo kubeadm token create --print-join-command
```

## Настройка kubeconfig для пользователя

Что будет изменено:

- будет создан каталог `$HOME/.kube`;
- admin kubeconfig будет скопирован в `$HOME/.kube/config`;
- владелец файла будет изменен на текущего пользователя.

Выполнить на control-plane node от обычного пользователя:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Проверка:

```bash
kubectl version
kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods -A -o wide
```

Ожидаем:

- `kubectl` подключается к API Server;
- control-plane node видна в `kubectl get nodes`;
- node может быть `NotReady` до установки CNI;
- CoreDNS может быть `Pending` до установки CNI;
- control plane static pods должны быть `Running`.

## Проверка static pods на control-plane

Команды ничего не меняют.

Выполнить на control-plane node:

```bash
ls -l /etc/kubernetes/manifests
sudo crictl ps | grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler|etcd'
kubectl -n kube-system get pods -o wide
```

Ожидаем static pod manifests:

```text
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

Важно понять:

- control plane components запущены kubelet как static pods;
- они не были созданы Deployment или DaemonSet;
- kubelet смотрит в `/etc/kubernetes/manifests` и запускает эти pods через containerd.

## Проверка certificates и kubeconfigs

Команды ничего не меняют.

Выполнить на control-plane node:

```bash
sudo kubeadm certs check-expiration
ls -l /etc/kubernetes
ls -l /etc/kubernetes/pki
```

Ожидаем:

- certificates созданы;
- kubeconfig files созданы;
- `admin.conf` существует;
- `kubelet.conf` существует после bootstrap.

## Подключение worker nodes

Что будет изменено на worker nodes:

- worker node пройдет bootstrap;
- kubelet получит конфигурацию для подключения к API Server;
- worker node зарегистрируется в cluster;
- kubelet начнет работать как Kubernetes node agent.

Выполнить на каждом worker node команду `kubeadm join`, полученную после `kubeadm init`.

Пример формата команды:

```bash
sudo kubeadm join 192.168.20.11:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

Не копируйте пример как готовую команду. Используйте реальную команду из своего `kubeadm init` output или получите ее на control-plane node:

```bash
sudo kubeadm token create --print-join-command
```

Перед выполнением на worker node можно сделать preflight-only проверку:

```bash
sudo kubeadm join 192.168.20.11:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --dry-run
```

Если dry-run прошел успешно, выполнить реальный join command без `--dry-run`.

## Проверка cluster после join

Команды ничего не меняют.

Выполнить на control-plane node:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl -n kube-system get pods
```

Ожидаем:

- control-plane node видна в списке nodes;
- worker nodes видны в списке nodes;
- nodes могут быть `NotReady` до установки CNI;
- CoreDNS может быть `Pending` до установки CNI;
- `kube-proxy` pods могут быть созданы;
- control plane static pods должны быть `Running`.

Это нормальное состояние для конца Lab 04:

```text
cluster создан
API Server работает
nodes зарегистрированы
CNI еще не установлен
nodes могут быть NotReady
CoreDNS может быть Pending
```

## Проверка kubelet на worker nodes

Команды ничего не меняют.

Выполнить на каждой worker node:

```bash
systemctl status kubelet --no-pager || true
journalctl -u kubelet -n 50 --no-pager
sudo crictl info | jq '.status.conditions'
```

Ожидаем:

- kubelet активен или пытается работать как node agent;
- `RuntimeReady: true`;
- `NetworkReady: false` возможно до установки CNI;
- в логах может быть сообщение о том, что network plugin не готов.

## Что считается нормальным в конце Lab 04

Нормально:

- `kubectl` работает на control-plane node;
- `kubectl get nodes` показывает все nodes;
- nodes могут быть `NotReady`;
- CoreDNS может быть `Pending`;
- `NetworkReady` может быть `false`;
- сообщение `cni plugin not initialized` допустимо.

Не нормально:

- `kubectl` не может подключиться к API Server;
- control plane static pods не запускаются;
- `kubeadm join` не может достучаться до `192.168.20.11:6443`;
- worker node не появляется в `kubectl get nodes` после успешного join;
- `RuntimeReady: false` у containerd.

## Rollback

Rollback удаляет состояние, созданное `kubeadm init` или `kubeadm join`.

Не выполняйте rollback без причины. Если cluster частично работает, сначала соберите диагностику.

### Сначала собрать диагностику

На control-plane node:

```bash
kubectl get nodes -o wide || true
kubectl get pods -A -o wide || true
sudo crictl ps -a | head -50
journalctl -u kubelet -n 100 --no-pager
```

На проблемной worker node:

```bash
systemctl status kubelet --no-pager || true
journalctl -u kubelet -n 100 --no-pager
sudo crictl info | jq '.status.conditions'
```

### Reset worker node

Выполнять только если нужно удалить worker node bootstrap state.

На worker node:

```bash
sudo kubeadm reset
```

После подтверждения kubeadm удалит состояние, созданное `kubeadm join`.

### Reset control-plane node

Выполнять только если нужно удалить control-plane bootstrap state.

На control-plane node:

```bash
sudo kubeadm reset
```

После reset могут остаться сетевые и runtime artifacts. Полная очистка CNI будет рассматриваться после Lab 05, когда CNI уже появится.

В этой лабораторной не используйте force-команды и ручное удаление директорий без отдельного разбора.

## Финальная проверка Lab 04

Выполнить на control-plane node:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
ls -l /etc/kubernetes/manifests
sudo kubeadm certs check-expiration
sudo kubeadm token list
```

Выполнить на каждой worker node:

```bash
hostname
systemctl status kubelet --no-pager || true
sudo crictl info | jq '.status.conditions'
```

## Критерии готовности

Lab 04 считается завершенной, если:

- `kubeadm init` успешно выполнен на control-plane node;
- `kubectl` настроен и работает на control-plane node;
- control-plane static pods запущены;
- worker nodes успешно выполнили `kubeadm join`;
- все nodes видны через `kubectl get nodes`;
- понятно, почему nodes могут быть `NotReady` до установки CNI;
- понятно, что CoreDNS может быть `Pending` до установки CNI;
- понятна роль `/etc/kubernetes/manifests` для static pods.

## Что приложить в lab journal

Добавьте в lab journal:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
ls -l /etc/kubernetes/manifests
sudo kubeadm certs check-expiration
sudo kubeadm token list
```

С worker nodes приложить:

```bash
hostname
systemctl status kubelet --no-pager || true
sudo crictl info | jq '.status.conditions'
```

Если вывод worker nodes одинаковый, достаточно приложить вывод с одной worker node и написать, что проверка повторена на остальных.

## Инженерная заметка / Вопрос с собеседований

Вопрос:

Почему после успешного `kubeadm init` и `kubeadm join` nodes могут оставаться `NotReady`?

Краткий ответ:

Потому что `kubeadm` создает control plane и выполняет bootstrap nodes, но не устанавливает CNI plugin. Без CNI kubelet не может полноценно подготовить Pod network на node, поэтому `NetworkReady` остается `false`, а node может быть `NotReady`.

Привязка к этой лабораторной:

В Lab 04 мы сознательно останавливаемся до установки CNI. Это позволяет увидеть границу ответственности: `kubeadm` поднимает control plane и регистрирует nodes, а CNI отвечает за Pod networking.

## Следующий шаг

Следующая лабораторная:

```text
lab-05-cni-bootstrap
```

В ней будет выполнено:

- установка CNI;
- проверка `NetworkReady`;
- проверка перехода nodes в `Ready`;
- проверка CoreDNS;
- первичная проверка pod-to-pod networking.
