# Lab 04 - kubeadm cluster bootstrap

### Дата

2026-06-05

### Цель

Создать первый Kubernetes cluster через `kubeadm`.

## вывод приложения
```
ae@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS     ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   NotReady   control-plane   22m     v1.36.1   192.168.1.40   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
worker-1          NotReady   <none>          9m52s   v1.36.1   192.168.1.43   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
worker-2          NotReady   <none>          9m30s   v1.36.1   192.168.1.46   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
ae@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS             RESTARTS   AGE     IP             NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-7hmjk                  0/1     Pending            0          22m     <none>         <none>            <none>           <none>
kube-system   coredns-589f44dc88-cmbt2                  0/1     Pending            0          22m     <none>         <none>            <none>           <none>
kube-system   etcd-control-plane-1                      1/1     Running            0          22m     192.168.1.40   control-plane-1   <none>           <none>
kube-system   kube-apiserver-control-plane-1            1/1     Running            0          22m     192.168.1.40   control-plane-1   <none>           <none>
kube-system   kube-controller-manager-control-plane-1   1/1     Running            0          22m     192.168.1.40   control-plane-1   <none>           <none>
kube-system   kube-proxy-4kvmt                          1/1     Running            0          22m     192.168.1.40   control-plane-1   <none>           <none>
kube-system   kube-proxy-62wxd                          0/1     ImagePullBackOff   0          9m58s   192.168.1.43   worker-1          <none>           <none>
kube-system   kube-proxy-hgk4d                          0/1     ErrImagePull       0          9m36s   192.168.1.46   worker-2          <none>           <none>
kube-system   kube-scheduler-control-plane-1            1/1     Running            0          22m     192.168.1.40   control-plane-1   <none>           <none>
ae@control-plane-1:~$ ls -l /etc/kubernetes/manifests
ls: cannot open directory '/etc/kubernetes/manifests': Permission denied
ae@control-plane-1:~$ sudo ls -l /etc/kubernetes/manifests
[sudo] password for ae: 
total 16
-rw------- 1 root root 2610 Jun  6 15:28 etcd.yaml
-rw------- 1 root root 3954 Jun  6 15:28 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun  6 15:28 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun  6 15:28 kube-scheduler.yaml
ae@control-plane-1:~$ sudo kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 06, 2027 15:28 UTC   364d            ca                      no      
apiserver                  Jun 06, 2027 15:28 UTC   364d            ca                      no      
apiserver-etcd-client      Jun 06, 2027 15:28 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Jun 06, 2027 15:28 UTC   364d            ca                      no      
controller-manager.conf    Jun 06, 2027 15:28 UTC   364d            ca                      no      
etcd-healthcheck-client    Jun 06, 2027 15:28 UTC   364d            etcd-ca                 no      
etcd-peer                  Jun 06, 2027 15:28 UTC   364d            etcd-ca                 no      
etcd-server                Jun 06, 2027 15:28 UTC   364d            etcd-ca                 no      
front-proxy-client         Jun 06, 2027 15:28 UTC   364d            front-proxy-ca          no      
scheduler.conf             Jun 06, 2027 15:28 UTC   364d            ca                      no      
super-admin.conf           Jun 06, 2027 15:28 UTC   364d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jun 03, 2036 15:28 UTC   9y              no      
etcd-ca                 Jun 03, 2036 15:28 UTC   9y              no      
front-proxy-ca          Jun 03, 2036 15:28 UTC   9y              no      
ae@control-plane-1:~$ sudo kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
e8265w.p1v9z852q5143f0b   23h         2026-06-07T15:29:03Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
```

```
ae@worker-1:~$ hostname
worker-1
ae@worker-1:~$ systemctl status kubelet --no-pager || true
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sat 2026-06-06 15:41:26 UTC; 11min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 11364 (kubelet)
      Tasks: 11 (limit: 4601)
     Memory: 32.1M (peak: 33.1M)
        CPU: 10.104s
     CGroup: /system.slice/kubelet.service
             └─11364 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 06 15:51:43 worker-1 kubelet[11364]: I0606 15:51:43.172770   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:51:48 worker-1 kubelet[11364]: I0606 15:51:48.173686   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:51:53 worker-1 kubelet[11364]: I0606 15:51:53.175744   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:51:58 worker-1 kubelet[11364]: I0606 15:51:58.176940   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:03 worker-1 kubelet[11364]: I0606 15:52:03.178252   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:08 worker-1 kubelet[11364]: I0606 15:52:08.179219   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:13 worker-1 kubelet[11364]: I0606 15:52:13.180064   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:18 worker-1 kubelet[11364]: I0606 15:52:18.181228   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:23 worker-1 kubelet[11364]: I0606 15:52:23.182709   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:28 worker-1 kubelet[11364]: I0606 15:52:28.184698   11364 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Hint: Some lines were ellipsized, use -l to show in full.
ae@worker-1:~$ sudo crictl info | jq '.status.conditions'
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

```
ae@worker-2:~$ hostname
worker-2
ae@worker-2:~$ systemctl status kubelet --no-pager || true
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Sat 2026-06-06 15:41:48 UTC; 11min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 11096 (kubelet)
      Tasks: 12 (limit: 4601)
     Memory: 31.9M (peak: 32.6M)
        CPU: 9.853s
     CGroup: /system.slice/kubelet.service
             └─11096 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 06 15:52:20 worker-2 kubelet[11096]: I0606 15:52:20.132903   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:25 worker-2 kubelet[11096]: I0606 15:52:25.134968   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:30 worker-2 kubelet[11096]: I0606 15:52:30.136271   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:35 worker-2 kubelet[11096]: I0606 15:52:35.138119   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:40 worker-2 kubelet[11096]: I0606 15:52:40.139953   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:45 worker-2 kubelet[11096]: I0606 15:52:45.141602   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:50 worker-2 kubelet[11096]: I0606 15:52:50.143359   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:52:55 worker-2 kubelet[11096]: I0606 15:52:55.145322   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:53:00 worker-2 kubelet[11096]: I0606 15:53:00.146957   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Jun 06 15:53:05 worker-2 kubelet[11096]: I0606 15:53:05.147952   11096 kubelet.go:3262] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message… not initialized"
Hint: Some lines were ellipsized, use -l to show in full.
ae@worker-2:~$ sudo crictl info | jq '.status.conditions'
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

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| E0605 PullImage from image service failed" Недоступен официальный реестр образов | создание кластера/инициализация |  | настроил зеркало из первого совета гугла |


### Что стало понятнее

Был недоступен официальный реестр образов.

Решение:
```
sudo mkdir -p /etc/containerd/certs.d/docker.io
sudo vim /etc/containerd/certs.d/docker.io/hosts.toml
```
Добавляем:
```
server = "https://registry-1.docker.io"

[host."https://dockerhub.timeweb.cloud"]
	capabilities = ["pull", "resolve"]
```
перезапускаем 
```
sudo systemctl restart containerd
```

### Статус

done