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

## Lab 04 - kubeadm cluster bootstrap

### Дата

2026-06-10

### Цель

Создать Kubernetes cluster через kubeadm

### Что было сделано

- Проверка состояния перед bootstrap
- Проверка IP control-plane node
- Проверка, что cluster еще не был инициализирован
- kubeadm init
- Настройка kubeconfig
- Проверка static pods
- Проверка certificates и kubeconfigs
- Подключение worker nodes
- Проверка cluster после join

### Команды

#### Проверка состояния перед bootstrap
Выполнить на всех нодах
```
hostname
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
swapon --show
sudo crictl info | jq '.status.conditions'
```
#### Проверка IP control-plane node
```
user@control-plane-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             192.168.88.184/24 metric 100
user@control-plane-1:~$ ip route
default via 192.168.88.1 dev eth0 proto dhcp src 192.168.88.184 metric 100 
192.168.88.0/24 dev eth0 proto kernel scope link src 192.168.88.184 metric 100 
192.168.88.1 dev eth0 proto dhcp scope link src 192.168.88.184 metric 100
```
#### Проверка, что cluster еще не был инициализирован
```
user@control-plane-1:~$ ls -la /etc/kubernetes || true
total 12
drwxrwxr-x   3 root root 4096 Jun  9 18:22 .
drwxr-xr-x 110 root root 4096 Jun 10 06:27 ..
drwxrwxr-x   2 root root 4096 Jun  9 18:22 manifests
user@control-plane-1:~$ ls -la /etc/kubernetes/manifests || true
total 8
drwxrwxr-x 2 root root 4096 Jun  9 18:22 .
drwxrwxr-x 3 root root 4096 Jun  9 18:22 ..
-rw-r--r-- 1 root root    0 May 12 13:15 .kubelet-keep
user@control-plane-1:~$ ls -la /var/lib/etcd || true
ls: cannot access '/var/lib/etcd': No such file or directory
```
#### kubeadm init dry-run
На control-plane node
```
sudo kubeadm init \
  --apiserver-advertise-address=192.168.88.184 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1 \
  --dry-run
```
#### kubeadm init
На control-plane node
```
sudo kubeadm init \
  --apiserver-advertise-address=192.168.88.184 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1
```
#### Настройка kubeconfig
На control-plane node
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Проверка на control-plane node
```
user@control-plane-1:~$ kubectl version
Client Version: v1.36.1
Kustomize Version: v5.8.1
Server Version: v1.36.1
user@control-plane-1:~$ kubectl cluster-info
Kubernetes control plane is running at https://192.168.88.184:6443
CoreDNS is running at https://192.168.88.184:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS     ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   NotReady   control-plane   6m36s   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
user@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP               NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-fwct6                  0/1     Pending   0          6m38s   <none>           <none>            <none>           <none>
kube-system   coredns-589f44dc88-zx2qb                  0/1     Pending   0          6m38s   <none>           <none>            <none>           <none>
kube-system   etcd-control-plane-1                      1/1     Running   0          6m46s   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-apiserver-control-plane-1            1/1     Running   0          6m46s   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-controller-manager-control-plane-1   1/1     Running   0          6m46s   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-hdvtm                          1/1     Running   0          6m39s   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-scheduler-control-plane-1            1/1     Running   0          6m46s   192.168.88.184   control-plane-1   <none>           <none>
```
#### Проверка static pods
На control-plane node
```
ls -l /etc/kubernetes/manifests
sudo crictl ps | grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler|etcd'
kubectl -n kube-system get pods -o wide
```
#### Проверка certificates и kubeconfigs
На control-plane node
```
sudo kubeadm certs check-expiration
ls -l /etc/kubernetes
ls -l /etc/kubernetes/pki
```
#### Подключение worker nodes
На worker нодах
```
sudo kubeadm join 192.168.88.184:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
#### Проверка cluster после join
На control-plane node
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS     ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   NotReady   control-plane   39m     v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          NotReady   <none>          3m27s   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          NotReady   <none>          2m40s   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
user@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP               NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-fwct6                  0/1     Pending   0          39m     <none>           <none>            <none>           <none>
kube-system   coredns-589f44dc88-zx2qb                  0/1     Pending   0          39m     <none>           <none>            <none>           <none>
kube-system   etcd-control-plane-1                      1/1     Running   0          39m     192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-apiserver-control-plane-1            1/1     Running   0          39m     192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-controller-manager-control-plane-1   1/1     Running   0          39m     192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-hdvtm                          1/1     Running   0          39m     192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-jc7lk                          1/1     Running   0          3m35s   192.168.88.134   worker-1          <none>           <none>
kube-system   kube-proxy-spstk                          1/1     Running   0          2m48s   192.168.88.243   worker-2          <none>           <none>
kube-system   kube-scheduler-control-plane-1            1/1     Running   0          39m     192.168.88.184   control-plane-1   <none>           <none>
user@control-plane-1:~$ kubectl -n kube-system get pods
NAME                                      READY   STATUS    RESTARTS   AGE
coredns-589f44dc88-fwct6                  0/1     Pending   0          39m
coredns-589f44dc88-zx2qb                  0/1     Pending   0          39m
etcd-control-plane-1                      1/1     Running   0          39m
kube-apiserver-control-plane-1            1/1     Running   0          39m
kube-controller-manager-control-plane-1   1/1     Running   0          39m
kube-proxy-hdvtm                          1/1     Running   0          39m
kube-proxy-jc7lk                          1/1     Running   0          3m51s
kube-proxy-spstk                          1/1     Running   0          3m4s
kube-scheduler-control-plane-1            1/1     Running   0          39m
```
#### Проверка kubelet на worker nodes
На каждой worker node
```
user@worker-1:~$ systemctl status kubelet --no-pager || true
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2026-06-10 22:08:07 MSK; 8min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 111085 (kubelet)
      Tasks: 12 (limit: 4653)
     Memory: 29.0M (peak: 30.0M)
        CPU: 7.950s
     CGroup: /system.slice/kubelet.service
             └─111085 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubel…
user@worker-1:~$ sudo journalctl -u kubelet -n 50 --no-pager
Jun 10 22:12:58 worker-1 kubelet[111085]: I0610 22:12:58.105024  111085 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 10 22:13:03 worker-1 kubelet[111085]: I0610 22:13:03.106304  111085 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
...
user@worker-1:~$ sudo crictl info | jq '.status.conditions'
[
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "RuntimeReady"
  },
  {
    "message": "Network plugin returns error: cni plugin not initialized",
    "reason": "NetworkPluginNotReady",
    "status": false,
    "type": "NetworkReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "ContainerdHasNoDeprecationWarnings"
  }
]
```
#### Финальная проверка
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   NotReady   control-plane   51m   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          NotReady   <none>          15m   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          NotReady   <none>          14m   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
user@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE   IP               NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-fwct6                  0/1     Pending   0          51m   <none>           <none>            <none>           <none>
kube-system   coredns-589f44dc88-zx2qb                  0/1     Pending   0          51m   <none>           <none>            <none>           <none>
kube-system   etcd-control-plane-1                      1/1     Running   0          51m   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-apiserver-control-plane-1            1/1     Running   0          51m   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-controller-manager-control-plane-1   1/1     Running   0          51m   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-hdvtm                          1/1     Running   0          51m   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-jc7lk                          1/1     Running   0          15m   192.168.88.134   worker-1          <none>           <none>
kube-system   kube-proxy-spstk                          1/1     Running   0          14m   192.168.88.243   worker-2          <none>           <none>
kube-system   kube-scheduler-control-plane-1            1/1     Running   0          51m   192.168.88.184   control-plane-1   <none>           <none>
user@control-plane-1:~$ ls -l /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2622 Jun 10 21:32 etcd.yaml
-rw------- 1 root root 3964 Jun 10 21:32 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun 10 21:32 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun 10 21:32 kube-scheduler.yaml
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
user@control-plane-1:~$ sudo kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
<token>   23h         2026-06-11T18:34:15Z   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token
<token>   23h         2026-06-11T18:32:32Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
```
На каждой worker node выполнена проверка
```
hostname
systemctl status kubelet --no-pager || true
sudo crictl info | jq '.status.conditions'
```
вывод на всех нодах аналогично
```
worker-1
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2026-06-10 22:08:07 MSK; 19min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 111085 (kubelet)
      Tasks: 12 (limit: 4653)
     Memory: 29.6M (peak: 30.3M)
        CPU: 15.176s
     CGroup: /system.slice/kubelet.service
             └─111085 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml
[
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "RuntimeReady"
  },
  {
    "message": "Network plugin returns error: cni plugin not initialized",
    "reason": "NetworkPluginNotReady",
    "status": false,
    "type": "NetworkReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "ContainerdHasNoDeprecationWarnings"
  }
]
```

### Ключевые выводы команд

 - kubeadm, kubelet, kubectl имеют версию v1.36.1
 - swapon --show не выводит активный swap
 - RuntimeReady имеет status: true
 - NetworkReady имеет status: false с причиной NetworkPluginNotReady, это ожидаемое состояние до установки CNI
 - ip адрес 192.168.88.184
 - etc/kubernetes/manifests не содержит static pod manifests
 - /var/lib/etcd отсутствует
 - kubeadm init dry-run прошел успешно
 - kubeadm init прошел успешно
 - kubectl подключается к API Server
 - control-plane node видна в kubectl get nodes
 - поды CoreDNS в статусе Pending до установки CNI
 - control plane static pods в состоянии Running
 - kubelet через containerd запустил control plane components как static pod из /etc/kubernetes/manifests
 - certificates созданы
 - kubeconfig files созданы
 - kubeadm join на workers успешно выполнен
 - worker nodes видны в списке nodes
 - nodes в состоянии NotReady до установки CNI
 - на worker kubelet активен; RuntimeReady: true; NetworkReady: false


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

- как crictl ps и kubectl показывают поды на разных уровнях абстракции
- как определить static pods
- есть разные CA


### Вопросы

- 

### Статус

done
