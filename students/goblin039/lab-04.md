# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Вячеслав  |
| GitHub | goblin039 |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | Done |  |  |
| Lab 01 - Node Baseline | Done |  |  |
| Lab 02 - Container Runtime: containerd | Done | | |
| Lab 03 - kubeadm prerequisites | Done | | |

## Lab 04 - kubeadm cluster bootstrap

### Дата

2026-06-01

### Цель

Создать Kubernetes cluster через kubeadm.

### Что было сделано

- Проверка, что cluster еще не был инициализирован 
- kubeadm init
- Настройка kubeconfig для пользователя
- Проверка static pods на control-plane
- Проверка certificates и kubeconfigs
- Подключение worker nodes
- Проверка cluster после join

### Команды

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# ls -la /etc/kubernetes || true
total 12
drwxrwxr-x  3 root root 4096 May 31 18:56 .
drwxr-xr-x 88 root root 4096 May 31 18:56 ..
drwxrwxr-x  2 root root 4096 May 31 18:56 manifests
root@deep-cp-01:/home/goblin# ls -la /etc/kubernetes/manifests || true
total 8
drwxrwxr-x 2 root root 4096 May 31 18:56 .
drwxrwxr-x 3 root root 4096 May 31 18:56 ..
-rw-r--r-- 1 root root    0 May 12 10:15 .kubelet-keep
root@deep-cp-01:/home/goblin# ls -la /var/lib/etcd || true
ls: cannot access '/var/lib/etcd': No such file or directory

root@deep-wrk-02:/home/goblin# kubeadm init \
  --apiserver-advertise-address=192.168.100.50 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1 \
  --dry-run
<Прошёл без косяков, очень большой вывод с разными манифестами...>  

root@deep-wrk-02:/home/goblin# kubeadm init \
  --apiserver-advertise-address=192.168.100.50 \
  --pod-network-cidr=192.168.0.0/16 \
  --kubernetes-version=v1.36.1

.....
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

kubeadm join 192.168.100.50:6443 --token tb48az.3fginanlv1qjzb4a \
        --discovery-token-ca-cert-hash sha256:<...token...>

root@deep-cp-01:/home/goblin# mkdir -p /home/goblin/.kube
root@deep-cp-01:/home/goblin# cp -i /etc/kubernetes/admin.conf /home/goblin/.kube/config
root@deep-cp-01:/home/goblin# chown goblin:goblin /home/goblin/.kube/config
root@deep-cp-01:/home/goblin# mkdir -p $HOME/.kube
root@deep-cp-01:/home/goblin# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

root@deep-cp-01:/home/goblin# ls -l /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2612 Jun  1 17:40 etcd.yaml
-rw------- 1 root root 3964 Jun  1 17:40 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun  1 17:40 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun  1 17:40 kube-scheduler.yaml
root@deep-cp-01:/home/goblin# sudo crictl ps | grep -E 'kube-apiserver|kube-controller-manager|kube-scheduler|etcd'
382afd9e849af       c2616bc3c6956       16 minutes ago      Running             kube-controller-manager   0                   bbc19653c6a95       kube-controller-manager-deep-cp-01   kube-system
7b3d0f03fb9d9       315157c5c4d76       16 minutes ago      Running             kube-apiserver            0                   5e78a717f5805       kube-apiserver-deep-cp-01            kube-system
7b81664a297a4       ee85eb1f0edd2       16 minutes ago      Running             etcd                      0                   162054a4fd94c       etcd-deep-cp-01                      kube-system
da759ed555c4e       b665198f31ae0       16 minutes ago      Running             kube-scheduler            0                   b23f73f2eb0e6       kube-scheduler-deep-cp-01            kube-system
root@deep-cp-01:/home/goblin# kubectl -n kube-system get pods -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
coredns-589f44dc88-mslt6             0/1     Pending   0          16m   <none>           <none>       <none>           <none>
coredns-589f44dc88-sd4kh             0/1     Pending   0          16m   <none>           <none>       <none>           <none>
etcd-deep-cp-01                      1/1     Running   0          16m   192.168.100.50   deep-cp-01   <none>           <none>
kube-apiserver-deep-cp-01            1/1     Running   0          16m   192.168.100.50   deep-cp-01   <none>           <none>
kube-controller-manager-deep-cp-01   1/1     Running   0          16m   192.168.100.50   deep-cp-01   <none>           <none>
kube-proxy-94t92                     1/1     Running   0          16m   192.168.100.50   deep-cp-01   <none>           <none>
kube-scheduler-deep-cp-01            1/1     Running   0          16m   192.168.100.50   deep-cp-01   <none>           <none>

root@deep-cp-01:/home/goblin# sudo kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 01, 2027 17:39 UTC   364d            ca                      no      
apiserver                  Jun 01, 2027 17:39 UTC   364d            ca                      no      
apiserver-etcd-client      Jun 01, 2027 17:39 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jun 01, 2027 17:39 UTC   364d            ca                      no      
controller-manager.conf    Jun 01, 2027 17:39 UTC   364d            ca                      no      
etcd-healthcheck-client    Jun 01, 2027 17:39 UTC   364d            etcd-ca                 no      
etcd-peer                  Jun 01, 2027 17:39 UTC   364d            etcd-ca                 no      
etcd-server                Jun 01, 2027 17:39 UTC   364d            etcd-ca                 no      
front-proxy-client         Jun 01, 2027 17:39 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jun 01, 2027 17:39 UTC   364d            ca                      no      
super-admin.conf           Jun 01, 2027 17:39 UTC   364d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      May 29, 2036 17:39 UTC   9y              no      
etcd-ca                 May 29, 2036 17:39 UTC   9y              no      
front-proxy-ca          May 29, 2036 17:39 UTC   9y              no

root@deep-cp-01:/home/goblin# ls -l /etc/kubernetes
total 48
-rw------- 1 root root 5638 Jun  1 17:40 admin.conf
-rw------- 1 root root 5662 Jun  1 17:40 controller-manager.conf
-rw------- 1 root root 1970 Jun  1 17:40 kubelet.conf
drwxrwxr-x 2 root root 4096 Jun  1 17:40 manifests
drwxr-xr-x 3 root root 4096 Jun  1 17:40 pki
-rw------- 1 root root 5610 Jun  1 17:40 scheduler.conf
-rw------- 1 root root 5666 Jun  1 17:40 super-admin.conf
drwx------ 3 root root 4096 Jun  1 17:36 tmp
root@deep-cp-01:/home/goblin# ls -l /etc/kubernetes/pki
total 60
-rw-r--r-- 1 root root 1123 Jun  1 17:40 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Jun  1 17:40 apiserver-etcd-client.key
-rw-r--r-- 1 root root 1131 Jun  1 17:40 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Jun  1 17:40 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1285 Jun  1 17:40 apiserver.crt
-rw------- 1 root root 1679 Jun  1 17:40 apiserver.key
-rw-r--r-- 1 root root 1107 Jun  1 17:40 ca.crt
-rw------- 1 root root 1675 Jun  1 17:40 ca.key
drwxr-xr-x 2 root root 4096 Jun  1 17:40 etcd
-rw-r--r-- 1 root root 1123 Jun  1 17:40 front-proxy-ca.crt
-rw------- 1 root root 1679 Jun  1 17:40 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jun  1 17:40 front-proxy-client.crt
-rw------- 1 root root 1675 Jun  1 17:40 front-proxy-client.key
-rw------- 1 root root 1675 Jun  1 17:40 sa.key
-rw------- 1 root root  451 Jun  1 17:40 sa.pub


```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# ls -la /etc/kubernetes || true
total 12
drwxrwxr-x  3 root root 4096 May 31 19:32 .
drwxr-xr-x 88 root root 4096 May 31 19:32 ..
drwxrwxr-x  2 root root 4096 May 31 19:32 manifests
root@deep-wrk-01:/home/goblin# ls -la /etc/kubernetes/manifests || true
total 8
drwxrwxr-x 2 root root 4096 May 31 19:32 .
drwxrwxr-x 3 root root 4096 May 31 19:32 ..
-rw-r--r-- 1 root root    0 May 12 10:15 .kubelet-keep
root@deep-wrk-01:/home/goblin# ls -la /var/lib/etcd || true
ls: cannot access '/var/lib/etcd': No such file or directory

kubeadm join 192.168.100.50:6443 --token tb48az.3fginanlv1qjzb4a \
        --discovery-token-ca-cert-hash sha256:<...token...>
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# ls -la /etc/kubernetes || true
total 12
drwxrwxr-x  3 root root 4096 May 31 18:48 .
drwxr-xr-x 88 root root 4096 May 31 20:21 ..
drwxrwxr-x  2 root root 4096 May 31 18:48 manifests
root@deep-wrk-02:/home/goblin# ls -la /etc/kubernetes/manifests || true
total 8
drwxrwxr-x 2 root root 4096 May 31 18:48 .
drwxrwxr-x 3 root root 4096 May 31 18:48 ..
-rw-r--r-- 1 root root    0 May 12 10:15 .kubelet-keep
root@deep-wrk-02:/home/goblin# ls -la /var/lib/etcd || true
ls: cannot access '/var/lib/etcd': No such file or directory

kubeadm join 192.168.100.50:6443 --token tb48az.3fginanlv1qjzb4a \
        --discovery-token-ca-cert-hash sha256:<...token...>
```

### Ключевые выводы команд

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# kubectl get nodes -o wide
NAME          STATUS     ROLES           AGE    VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
deep-cp-01    NotReady   control-plane   22m    v1.36.1   192.168.100.50   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-01   NotReady   <none>          101s   v1.36.1   192.168.100.51   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-02   NotReady   <none>          77s    v1.36.1   192.168.100.52   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
root@deep-cp-01:/home/goblin# kubectl get pods -A -o wide
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE    IP               NODE          NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-mslt6             0/1     Pending   0          22m    <none>           <none>        <none>           <none>
kube-system   coredns-589f44dc88-sd4kh             0/1     Pending   0          22m    <none>           <none>        <none>           <none>
kube-system   etcd-deep-cp-01                      1/1     Running   0          22m    192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-apiserver-deep-cp-01            1/1     Running   0          22m    192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-controller-manager-deep-cp-01   1/1     Running   0          22m    192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-proxy-94t92                     1/1     Running   0          22m    192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-proxy-gcrzz                     1/1     Running   0          90s    192.168.100.52   deep-wrk-02   <none>           <none>
kube-system   kube-proxy-p7dff                     1/1     Running   0          114s   192.168.100.51   deep-wrk-01   <none>           <none>
kube-system   kube-scheduler-deep-cp-01            1/1     Running   0          22m    192.168.100.50   deep-cp-01    <none>           <none>
root@deep-cp-01:/home/goblin# kubectl -n kube-system get pods
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-589f44dc88-mslt6             0/1     Pending   0          22m
coredns-589f44dc88-sd4kh             0/1     Pending   0          22m
etcd-deep-cp-01                      1/1     Running   0          22m
kube-apiserver-deep-cp-01            1/1     Running   0          22m
kube-controller-manager-deep-cp-01   1/1     Running   0          22m
kube-proxy-94t92                     1/1     Running   0          22m
kube-proxy-gcrzz                     1/1     Running   0          107s
kube-proxy-p7dff                     1/1     Running   0          2m11s
kube-scheduler-deep-cp-01            1/1     Running   0          22m
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# systemctl status kubelet --no-pager || true
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2026-06-01 18:01:26 UTC; 9min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 3045 (kubelet)
      Tasks: 11 (limit: 9317)
     Memory: 33.3M (peak: 34.2M)
        CPU: 13.041s
     CGroup: /system.slice/kubelet.service
             └─3045 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 01 18:10:02 deep-wrk-01 kubelet[3045]: I0601 18:10:02.318185    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:07 deep-wrk-01 kubelet[3045]: I0601 18:10:07.320108    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:12 deep-wrk-01 kubelet[3045]: I0601 18:10:12.322393    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:17 deep-wrk-01 kubelet[3045]: I0601 18:10:17.325947    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:22 deep-wrk-01 kubelet[3045]: I0601 18:10:22.328296    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:27 deep-wrk-01 kubelet[3045]: I0601 18:10:27.330745    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:32 deep-wrk-01 kubelet[3045]: I0601 18:10:32.332434    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:37 deep-wrk-01 kubelet[3045]: I0601 18:10:37.333984    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:42 deep-wrk-01 kubelet[3045]: I0601 18:10:42.335824    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:10:47 deep-wrk-01 kubelet[3045]: I0601 18:10:47.338481    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Hint: Some lines were ellipsized, use -l to show in full.

root@deep-wrk-01:/home/goblin# journalctl -u kubelet -n 5 --no-pager
Jun 01 18:10:57 deep-wrk-01 kubelet[3045]: I0601 18:10:57.343013    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:11:02 deep-wrk-01 kubelet[3045]: I0601 18:11:02.344421    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:11:07 deep-wrk-01 kubelet[3045]: I0601 18:11:07.346438    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:11:12 deep-wrk-01 kubelet[3045]: I0601 18:11:12.348205    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:11:17 deep-wrk-01 kubelet[3045]: I0601 18:11:17.350369    3045 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"

root@deep-wrk-01:/home/goblin# crictl info | jq '.status.conditions'
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
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# systemctl status kubelet --no-pager || true
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2026-06-01 18:01:50 UTC; 11min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 3052 (kubelet)
      Tasks: 11 (limit: 9317)
     Memory: 32.8M (peak: 34.3M)
        CPU: 15.639s
     CGroup: /system.slice/kubelet.service
             └─3052 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 01 18:12:56 deep-wrk-02 kubelet[3052]: I0601 18:12:56.388877    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:01 deep-wrk-02 kubelet[3052]: I0601 18:13:01.391669    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:06 deep-wrk-02 kubelet[3052]: I0601 18:13:06.393401    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:11 deep-wrk-02 kubelet[3052]: I0601 18:13:11.395242    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:16 deep-wrk-02 kubelet[3052]: I0601 18:13:16.397664    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:21 deep-wrk-02 kubelet[3052]: I0601 18:13:21.399259    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:26 deep-wrk-02 kubelet[3052]: I0601 18:13:26.401131    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:31 deep-wrk-02 kubelet[3052]: I0601 18:13:31.403022    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:36 deep-wrk-02 kubelet[3052]: I0601 18:13:36.405054    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Jun 01 18:13:41 deep-wrk-02 kubelet[3052]: I0601 18:13:41.407911    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network…n not initialized"
Hint: Some lines were ellipsized, use -l to show in full.

root@deep-wrk-02:/home/goblin# journalctl -u kubelet -n 5 --no-pager
Jun 01 18:13:41 deep-wrk-02 kubelet[3052]: I0601 18:13:41.407911    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:13:46 deep-wrk-02 kubelet[3052]: I0601 18:13:46.409521    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:13:51 deep-wrk-02 kubelet[3052]: I0601 18:13:51.411295    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:13:56 deep-wrk-02 kubelet[3052]: I0601 18:13:56.413468    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
Jun 01 18:14:01 deep-wrk-02 kubelet[3052]: I0601 18:14:01.415155    3052 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"

root@deep-wrk-02:/home/goblin# crictl info | jq '.status.conditions'
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

### Что стало понятнее

- При привычке работать из-под рута нужно не забывать раскладывать config и пользователю тоже :)
- При «простой» установки нужно добавлять cni или прописывать роутинг руками, но бз FRR оно будет работать только до ребута.
- 

### Статус

Done

