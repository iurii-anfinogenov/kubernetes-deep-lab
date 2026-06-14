# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

## Участник

| Поле | Значение |
|---|---|
| Имя | Игорь |
| GitHub | https://github.com/g0dsha |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | done |  |  |
| Lab 01 - Node baseline | done |  |  |
| Lab 02 - Container Runtime: containerd | done |  |  |
| Lab 03 - kubeadm prerequisites | done |  |  |
| Lab 04 - kubeadm cluster bootstrap | done |  |  |
| Lab 05 - CNI bootstrap with Calico | done |  |  |
| Lab 06 - Разбор kubeadm-кластера после установки | done |  |  |

## Lab 06 - Разбор kubeadm-кластера после установки

### Дата

2026-06-11

### Цель

Разобрать уже созданный Kubernetes cluster и понять, что именно подготовил kubeadm

### Что было сделано

- Проверки с разбором состояния.

### Команды
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   20h   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          Ready    <none>          19h   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          Ready    <none>          19h   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
```
У всех нод состояние Ready, у control-plane-1 в ROLES указана label: control-plane

```
user@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-fwct6                  1/1     Running   0          20h   192.168.247.132   control-plane-1   <none>           <none>
coredns-589f44dc88-zx2qb                  1/1     Running   0          20h   192.168.247.133   control-plane-1   <none>           <none>
etcd-control-plane-1                      1/1     Running   0          20h   192.168.88.184    control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   0          20h   192.168.88.184    control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   0          20h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-hdvtm                          1/1     Running   0          20h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-jc7lk                          1/1     Running   0          19h   192.168.88.134    worker-1          <none>           <none>
kube-proxy-spstk                          1/1     Running   0          19h   192.168.88.243    worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   0          20h   192.168.88.184    control-plane-1   <none>           <none>
```
Static pods косвенно можно отличить по формату имени (component)-(node-name). Control plane components в kubeadm cluster запущены как static pods, чтобы обеспечить отказоустойчивость и независимость от самого API Server, например, если он упадет.

```
user@control-plane-1:~$ sudo ls -l /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2622 Jun 10 21:32 etcd.yaml
-rw------- 1 root root 3964 Jun 10 21:32 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun 10 21:32 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun 10 21:32 kube-scheduler.yaml
```
Здесь kubeadm хранит манифесты для static pod control plane, необходим контроль целостности и компрометации

```
user@control-plane-1:~$ sudo grep 'advertise-address\|--secure-port\|--etcd-servers\|--client-ca-file\|--tls-cert-file\|--tls-private-key-file\|--service-cluster-ip-range\|--authorization-mode\|--enable
-admission-plugins' /etc/kubernetes/manifests/kube-apiserver.yaml
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.88.184:6443
    - --advertise-address=192.168.88.184
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --etcd-servers=https://127.0.0.1:2379
    - --secure-port=6443
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```
Параметры apiserver, на которые обратить внимание

```
user@control-plane-1:~$ sudo grep 'data-dir\|--listen-client-urls\|--advertise-client-urls\|--cert-file\|--key-file\|--trusted-ca-file' /etc/kubernetes/manifests/etcd.yaml 
    - --advertise-client-urls=https://192.168.88.184:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --data-dir=/var/lib/etcd
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.88.184:2379
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
Параметры etcd, на которые обратить внимание

```
user@control-plane-1:~$ sudo ls -l /etc/kubernetes/*.conf
-rw------- 1 root root 5638 Jun 10 21:32 /etc/kubernetes/admin.conf
-rw------- 1 root root 5662 Jun 10 21:32 /etc/kubernetes/controller-manager.conf
-rw------- 1 root root 1990 Jun 10 21:32 /etc/kubernetes/kubelet.conf
-rw------- 1 root root 5610 Jun 10 21:32 /etc/kubernetes/scheduler.conf
-rw------- 1 root root 5666 Jun 10 21:32 /etc/kubernetes/super-admin.conf
```
super-admin? kubeconfig - файл с настройками доступа к кластеру, нужен не только для kubectl, но и для других компонентов control plane, например, kube-controller-manager и kube-scheduler.

```
user@control-plane-1:~$ sudo find /etc/kubernetes/pki -maxdepth 2 -type f | sort
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/etcd/healthcheck-client.crt
/etc/kubernetes/pki/etcd/healthcheck-client.key
/etc/kubernetes/pki/etcd/peer.crt
/etc/kubernetes/pki/etcd/peer.key
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/front-proxy-ca.key
/etc/kubernetes/pki/front-proxy-client.crt
/etc/kubernetes/pki/front-proxy-client.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/sa.pub
```
Тут сертификаты и ключи для компонентов кластера. private keys нельзя публиковать, потому что это компрометация безопасности кластера. Например, зная ca.key, злоумышленник может подписать свои сертификаты и получить полный доступ к управлению кластером.

```
user@control-plane-1:~$ sudo kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 10, 2027 18:30 UTC   364d            ca                      no      
apiserver                  Jun 10, 2027 18:30 UTC   364d            ca                      no      
apiserver-etcd-client      Jun 10, 2027 18:30 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jun 10, 2027 18:30 UTC   364d            ca                      no      
controller-manager.conf    Jun 10, 2027 18:30 UTC   364d            ca                      no      
etcd-healthcheck-client    Jun 10, 2027 18:30 UTC   364d            etcd-ca                 no      
etcd-peer                  Jun 10, 2027 18:30 UTC   364d            etcd-ca                 no      
etcd-server                Jun 10, 2027 18:30 UTC   364d            etcd-ca                 no      
front-proxy-client         Jun 10, 2027 18:30 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jun 10, 2027 18:30 UTC   364d            ca                      no      
super-admin.conf           Jun 10, 2027 18:30 UTC   364d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jun 07, 2036 18:30 UTC   9y              no      
etcd-ca                 Jun 07, 2036 18:30 UTC   9y              no      
front-proxy-ca          Jun 07, 2036 18:30 UTC   9y              no
```
Есть 3 центра сертификации подписывающие остальные сертификаты, у них срок действия 10 лет, у остальных 1 год. Судя по значениям no в колонке EXTERNALLY MANAGED все сертификаты управляются kubeadm, внешние центры сертификации не используются

```
user@control-plane-1:~$ systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2026-06-10 21:32:33 MSK; 21h ago
       Docs: https://kubernetes.io/docs/
   Main PID: 110120 (kubelet)
      Tasks: 11 (limit: 4653)
     Memory: 58.3M (peak: 59.4M)
        CPU: 29min 31.537s
     CGroup: /system.slice/kubelet.service
             └─110120 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml
```
Выполнено на всех нодах, вывод аналогичный. kubelet отвечает за свою ноду и общается с containerd через интерфейс CRI. Если kubelet на control-plane остановится, то перестанут обновляться статусы подов на этой ноде, static pods продолжат работать, но Kubernetes не сможет управлять ими. Нода получит статус NotReady и Scheduler перестанет размещать новые поды на эту ноду.

```
user@control-plane-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                                       NAMESPACE           ATTEMPT             RUNTIME
823dda3915a19       3 hours ago         Ready               goldmane-6885dcb7d-ncwc8                   calico-system       0                   (default)
2db4e4b94795a       3 hours ago         Ready               calico-apiserver-9dcbdf8f6-rtt4g           calico-system       0                   (default)
4bef584269e83       3 hours ago         Ready               calico-apiserver-9dcbdf8f6-v4465           calico-system       0                   (default)
d36f89229dfd0       3 hours ago         Ready               coredns-589f44dc88-zx2qb                   kube-system         0                   (default)
008f784d327fd       3 hours ago         Ready               coredns-589f44dc88-fwct6                   kube-system         0                   (default)
fd27430823b2a       3 hours ago         Ready               calico-kube-controllers-6c677558b8-8hb9n   calico-system       0                   (default)
83c31853d93e2       3 hours ago         Ready               whisker-5674dc44cb-njzf9                   calico-system       0                   (default)
6f0c298001bd8       3 hours ago         Ready               csi-node-driver-ccj4j                      calico-system       0                   (default)
ee1b5b891153b       3 hours ago         Ready               calico-node-2zpr6                          calico-system       0                   (default)
aabadac12e25e       22 hours ago        Ready               kube-proxy-hdvtm                           kube-system         0                   (default)
0bbc344206857       22 hours ago        Ready               kube-apiserver-control-plane-1             kube-system         0                   (default)
79d527514bad4       22 hours ago        Ready               kube-controller-manager-control-plane-1    kube-system         0                   (default)
4b0106ec811e1       22 hours ago        Ready               kube-scheduler-control-plane-1             kube-system         0                   (default)
28f5505442ef2       22 hours ago        Ready               etcd-control-plane-1                       kube-system         0                   (default)

user@control-plane-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                        NAMESPACE
1fe9df8ad649a       5f531f3cb4757       3 hours ago         Running             whisker-backend             0                   83c31853d93e2       whisker-5674dc44cb-njzf9                   calico-system
b686c5269f91c       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   0                   6f0c298001bd8       csi-node-driver-ccj4j                      calico-system
29a2e8ccfb0da       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            0                   2db4e4b94795a       calico-apiserver-9dcbdf8f6-rtt4g           calico-system
1b69c84125bd0       8b5156bd00aba       3 hours ago         Running             goldmane                    0                   823dda3915a19       goldmane-6885dcb7d-ncwc8                   calico-system
9bb33e20ed782       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            0                   4bef584269e83       calico-apiserver-9dcbdf8f6-v4465           calico-system
07cef613c41d8       e67ce8d034ec5       3 hours ago         Running             calico-kube-controllers     0                   fd27430823b2a       calico-kube-controllers-6c677558b8-8hb9n   calico-system
6b057a64897d8       b4638c5ba691c       3 hours ago         Running             whisker                     0                   83c31853d93e2       whisker-5674dc44cb-njzf9                   calico-system
9ce6942ceae53       a665cad044872       3 hours ago         Running             calico-csi                  0                   6f0c298001bd8       csi-node-driver-ccj4j                      calico-system
7ed84b27c0769       38667dd9be96c       3 hours ago         Running             coredns                     0                   d36f89229dfd0       coredns-589f44dc88-zx2qb                   kube-system
907b8250daadb       38667dd9be96c       3 hours ago         Running             coredns                     0                   008f784d327fd       coredns-589f44dc88-fwct6                   kube-system
714ee2ecb0eeb       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 0                   ee1b5b891153b       calico-node-2zpr6                          calico-system
cecd1b3485759       78282b5844742       22 hours ago        Running             kube-proxy                  0                   aabadac12e25e       kube-proxy-hdvtm                           kube-system
e56d2873c1b48       ee85eb1f0edd2       22 hours ago        Running             etcd                        0                   28f5505442ef2       etcd-control-plane-1                       kube-system
590dd929784f9       c2616bc3c6956       22 hours ago        Running             kube-controller-manager     0                   79d527514bad4       kube-controller-manager-control-plane-1    kube-system
c62fb5d838314       315157c5c4d76       22 hours ago        Running             kube-apiserver              0                   0bbc344206857       kube-apiserver-control-plane-1             kube-system
0f4e4a86daae4       b665198f31ae0       22 hours ago        Running             kube-scheduler              0                   4b0106ec811e1       kube-scheduler-control-plane-1             kube-system

user@worker-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                            NAMESPACE           ATTEMPT             RUNTIME
740b684f8ee0d       3 hours ago         Ready               csi-node-driver-hj7cc           calico-system       0                   (default)
dfe160d4793c1       3 hours ago         Ready               calico-node-5kqm7               calico-system       0                   (default)
bec87b69bfb48       3 hours ago         Ready               calico-typha-648595f9df-w2952   calico-system       0                   (default)
56418f08a67f7       21 hours ago        Ready               kube-proxy-jc7lk                kube-system         0                   (default)

user@worker-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                             NAMESPACE
262c19f0c29c9       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   0                   740b684f8ee0d       csi-node-driver-hj7cc           calico-system
967441fcf9a34       a665cad044872       3 hours ago         Running             calico-csi                  0                   740b684f8ee0d       csi-node-driver-hj7cc           calico-system
c55dd30f97ffa       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 0                   dfe160d4793c1       calico-node-5kqm7               calico-system
2c4db041d70ed       a4433ba6d0851       3 hours ago         Running             calico-typha                0                   bec87b69bfb48       calico-typha-648595f9df-w2952   calico-system
dda2573258527       78282b5844742       21 hours ago        Running             kube-proxy                  0                   56418f08a67f7       kube-proxy-jc7lk                kube-system

user@worker-2:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                               NAMESPACE           ATTEMPT             RUNTIME
5329d16c1a72f       3 hours ago         Ready               csi-node-driver-r4n6x              calico-system       0                   (default)
b835f6a972c3e       3 hours ago         Ready               calico-typha-648595f9df-tg4fb      calico-system       0                   (default)
ec43c6ba2dd20       3 hours ago         Ready               calico-node-tc25r                  calico-system       0                   (default)
1916d2cdcfea9       4 hours ago         Ready               tigera-operator-85dbff4478-ntdvh   tigera-operator     0                   (default)
61658d911f2d6       21 hours ago        Ready               kube-proxy-spstk                   kube-system         0                   (default)

user@worker-2:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                NAMESPACE
1c4556015a93c       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   0                   5329d16c1a72f       csi-node-driver-r4n6x              calico-system
dd2b547c386f4       a665cad044872       3 hours ago         Running             calico-csi                  0                   5329d16c1a72f       csi-node-driver-r4n6x              calico-system
d638cc099f80f       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 0                   ec43c6ba2dd20       calico-node-tc25r                  calico-system
a102a6ce8a931       a4433ba6d0851       3 hours ago         Running             calico-typha                0                   b835f6a972c3e       calico-typha-648595f9df-tg4fb      calico-system
5c123ff18f295       2d636341e3971       4 hours ago         Running             tigera-operator             0                   1916d2cdcfea9       tigera-operator-85dbff4478-ntdvh   tigera-operator
b1b1e10eef20c       78282b5844742       21 hours ago        Running             kube-proxy                  0                   61658d911f2d6       kube-proxy-spstk                   kube-system
```
На control-plane гораздо больше подов и контейнеров. На worker нодах только поды связаные с сетью и хранением. Примечательно, что обе реплики coreDNS на control-plane. kubectl get pods показывает поды на уровне Kubernetes со всех узлов кластера и видит только те поды, которые зарегистрированы в etcd. Crictl ps показывает контейнеры только на текущей ноде.

```
user@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-fwct6                  1/1     Running   0          21h   192.168.247.132   control-plane-1   <none>           <none>
coredns-589f44dc88-zx2qb                  1/1     Running   0          21h   192.168.247.133   control-plane-1   <none>           <none>
etcd-control-plane-1                      1/1     Running   0          21h   192.168.88.184    control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   0          21h   192.168.88.184    control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   0          21h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-hdvtm                          1/1     Running   0          21h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-jc7lk                          1/1     Running   0          21h   192.168.88.134    worker-1          <none>           <none>
kube-proxy-spstk                          1/1     Running   0          21h   192.168.88.243    worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   0          21h   192.168.88.184    control-plane-1   <none>           <none>

user@control-plane-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                                       NAMESPACE           ATTEMPT             RUNTIME
823dda3915a19       3 hours ago         Ready               goldmane-6885dcb7d-ncwc8                   calico-system       0                   (default)
2db4e4b94795a       3 hours ago         Ready               calico-apiserver-9dcbdf8f6-rtt4g           calico-system       0                   (default)
4bef584269e83       3 hours ago         Ready               calico-apiserver-9dcbdf8f6-v4465           calico-system       0                   (default)
d36f89229dfd0       3 hours ago         Ready               coredns-589f44dc88-zx2qb                   kube-system         0                   (default)
008f784d327fd       3 hours ago         Ready               coredns-589f44dc88-fwct6                   kube-system         0                   (default)
fd27430823b2a       3 hours ago         Ready               calico-kube-controllers-6c677558b8-8hb9n   calico-system       0                   (default)
83c31853d93e2       3 hours ago         Ready               whisker-5674dc44cb-njzf9                   calico-system       0                   (default)
6f0c298001bd8       3 hours ago         Ready               csi-node-driver-ccj4j                      calico-system       0                   (default)
ee1b5b891153b       3 hours ago         Ready               calico-node-2zpr6                          calico-system       0                   (default)
aabadac12e25e       22 hours ago        Ready               kube-proxy-hdvtm                           kube-system         0                   (default)
0bbc344206857       22 hours ago        Ready               kube-apiserver-control-plane-1             kube-system         0                   (default)
79d527514bad4       22 hours ago        Ready               kube-controller-manager-control-plane-1    kube-system         0                   (default)
4b0106ec811e1       22 hours ago        Ready               kube-scheduler-control-plane-1             kube-system         0                   (default)
28f5505442ef2       22 hours ago        Ready               etcd-control-plane-1                       kube-system         0                   (default)

user@control-plane-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                        NAMESPACE
1fe9df8ad649a       5f531f3cb4757       3 hours ago         Running             whisker-backend             0                   83c31853d93e2       whisker-5674dc44cb-njzf9                   calico-system
b686c5269f91c       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   0                   6f0c298001bd8       csi-node-driver-ccj4j                      calico-system
29a2e8ccfb0da       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            0                   2db4e4b94795a       calico-apiserver-9dcbdf8f6-rtt4g           calico-system
1b69c84125bd0       8b5156bd00aba       3 hours ago         Running             goldmane                    0                   823dda3915a19       goldmane-6885dcb7d-ncwc8                   calico-system
9bb33e20ed782       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            0                   4bef584269e83       calico-apiserver-9dcbdf8f6-v4465           calico-system
07cef613c41d8       e67ce8d034ec5       3 hours ago         Running             calico-kube-controllers     0                   fd27430823b2a       calico-kube-controllers-6c677558b8-8hb9n   calico-system
6b057a64897d8       b4638c5ba691c       3 hours ago         Running             whisker                     0                   83c31853d93e2       whisker-5674dc44cb-njzf9                   calico-system
9ce6942ceae53       a665cad044872       3 hours ago         Running             calico-csi                  0                   6f0c298001bd8       csi-node-driver-ccj4j                      calico-system
7ed84b27c0769       38667dd9be96c       3 hours ago         Running             coredns                     0                   d36f89229dfd0       coredns-589f44dc88-zx2qb                   kube-system
907b8250daadb       38667dd9be96c       3 hours ago         Running             coredns                     0                   008f784d327fd       coredns-589f44dc88-fwct6                   kube-system
714ee2ecb0eeb       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 0                   ee1b5b891153b       calico-node-2zpr6                          calico-system
cecd1b3485759       78282b5844742       22 hours ago        Running             kube-proxy                  0                   aabadac12e25e       kube-proxy-hdvtm                           kube-system
e56d2873c1b48       ee85eb1f0edd2       22 hours ago        Running             etcd                        0                   28f5505442ef2       etcd-control-plane-1                       kube-system
590dd929784f9       c2616bc3c6956       22 hours ago        Running             kube-controller-manager     0                   79d527514bad4       kube-controller-manager-control-plane-1    kube-system
c62fb5d838314       315157c5c4d76       22 hours ago        Running             kube-apiserver              0                   0bbc344206857       kube-apiserver-control-plane-1             kube-system
0f4e4a86daae4       b665198f31ae0       22 hours ago        Running             kube-scheduler              0                   4b0106ec811e1       kube-scheduler-control-plane-1             kube-system
user@control-plane-1:~$ 
```
Как видим kubectl показывает поды только из пространства имен kube-system, зато со всех узлов кластера, crictl pods показывает поды только на текущем узле, но из всех пространств имен, поэтому вывод команд показывает разное, но, например, под coredns-589f44dc88-fwct6 есть в выводе всех команд:
1. kubectl: под coredns-589f44dc88-fwct6
2. crictl pods: запись с именем coredns-589f44dc88-fwct6 и уникальным POD ID 008f784d327fd
3. crictl ps: контейнер с именем coredns, привязанный к POD ID 008f784d327fd

### Ключевые выводы команд

 - 


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

- Kubeadm выполнил начальную конфигурацию: сгенерировал сертификаты, создал файлы конфигурации, записал манифесты static pods и инициировал создание кластера. Kubelet затем работает как systemd service. Cледит за манифестами и API Server, обращается к containerd через CRI.
- Контрольные вопросы - полезный блок).

### Вопросы

- Написано "worker node с ролью - нормальное состояние для kubeadm cluster, если label роли не был добавлен отдельно.", возможно имелось ввиду worker node без роли ?
- Есть еще /etc/kubernetes/super-admin.conf про него не сказано.
- В блоке "Зафиксировать" как-то все смешалось, на контрольные вопросы отвечал походу, надеюсь это ок.

### Статус

done