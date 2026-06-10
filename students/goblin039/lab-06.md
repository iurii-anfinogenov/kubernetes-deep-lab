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
| Lab 04 - kubeadm cluster bootstrap | Done | | |
| Lab 05 - CNI bootstrap with Calico | Done | | |

## Lab 06 - Разбор kubeadm-кластера после установки

### Дата

2026-06-08

### Цель

Разобрать уже созданный Kubernetes cluster и понять, что именно подготовил `kubeadm`.

### Что было сделано

- 
- 
- 

### Команды

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
deep-cp-01    Ready    control-plane   6d18h   v1.36.1   192.168.100.50   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-01   Ready    <none>          6d17h   v1.36.1   192.168.100.51   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-02   Ready    <none>          6d17h   v1.36.1   192.168.100.52   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1  

root@deep-cp-01:/home/goblin# kubectl -n kube-system get pods -o wide
NAME                                 READY   STATUS    RESTARTS      AGE     IP               NODE          NOMINATED NODE   READINESS GATES
coredns-589f44dc88-mslt6             1/1     Running   1 (81m ago)   6d18h   192.168.62.16    deep-cp-01    <none>           <none>
coredns-589f44dc88-sd4kh             1/1     Running   1 (81m ago)   6d18h   192.168.62.10    deep-cp-01    <none>           <none>
etcd-deep-cp-01                      1/1     Running   2 (81m ago)   6d18h   192.168.100.50   deep-cp-01    <none>           <none>
kube-apiserver-deep-cp-01            1/1     Running   2 (81m ago)   6d18h   192.168.100.50   deep-cp-01    <none>           <none>
kube-controller-manager-deep-cp-01   1/1     Running   2 (81m ago)   6d18h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-94t92                     1/1     Running   2 (81m ago)   6d18h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-gcrzz                     1/1     Running   2 (81m ago)   6d17h   192.168.100.52   deep-wrk-02   <none>           <none>
kube-proxy-p7dff                     1/1     Running   2 (81m ago)   6d17h   192.168.100.51   deep-wrk-01   <none>           <none>
kube-scheduler-deep-cp-01            1/1     Running   2 (81m ago)   6d18h   192.168.100.50   deep-cp-01    <none>           <none>  

root@deep-cp-01:/home/goblin# kubectl -n calico-system get pods -o wide
NAME                                       READY   STATUS    RESTARTS      AGE     IP                NODE          NOMINATED NODE   READINESS GATES
calico-apiserver-9c45c8f8f-5fgwk           1/1     Running   1 (87m ago)   4d21h   192.168.62.13     deep-cp-01    <none>           <none>
calico-apiserver-9c45c8f8f-zq7sh           1/1     Running   1 (87m ago)   4d21h   192.168.62.11     deep-cp-01    <none>           <none>
calico-kube-controllers-5b8d487f58-6jb5z   1/1     Running   1 (87m ago)   4d21h   192.168.62.12     deep-cp-01    <none>           <none>
calico-node-fcxdj                          1/1     Running   1 (87m ago)   4d21h   192.168.100.52    deep-wrk-02   <none>           <none>
calico-node-k7rg2                          1/1     Running   1 (87m ago)   4d21h   192.168.100.51    deep-wrk-01   <none>           <none>
calico-node-p4sjq                          1/1     Running   1 (87m ago)   4d21h   192.168.100.50    deep-cp-01    <none>           <none>
calico-typha-5bb76474df-6h8tk              1/1     Running   1 (87m ago)   4d21h   192.168.100.52    deep-wrk-02   <none>           <none>
calico-typha-5bb76474df-9ch4x              1/1     Running   1 (87m ago)   4d21h   192.168.100.51    deep-wrk-01   <none>           <none>
csi-node-driver-6xctm                      2/2     Running   2 (87m ago)   4d21h   192.168.158.195   deep-wrk-01   <none>           <none>
csi-node-driver-dc2pq                      2/2     Running   2 (87m ago)   4d21h   192.168.19.131    deep-wrk-02   <none>           <none>
csi-node-driver-dwsqc                      2/2     Running   2 (87m ago)   4d21h   192.168.62.9      deep-cp-01    <none>           <none>
goldmane-6885dcb7d-hgtvj                   1/1     Running   1 (87m ago)   4d21h   192.168.62.14     deep-cp-01    <none>           <none>
whisker-5cb7d57d4c-bk5cm                   2/2     Running   2 (87m ago)   4d21h   192.168.62.15     deep-cp-01    <none>           <none>


root@deep-cp-01:/home/goblin# ls -l /etc/kubernetes/manifests/
total 16
-rw------- 1 root root 2612 Jun  1 17:40 etcd.yaml
-rw------- 1 root root 3964 Jun  1 17:40 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun  1 17:40 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun  1 17:40 kube-scheduler.yaml

-----

sudo sed -n '1,220p' /etc/kubernetes/manifests/kube-apiserver.yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.100.50:6443
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.100.50
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: registry.k8s.io/kube-apiserver:v1.36.1   
    name: kube-apiserver
    ports:
    - containerPort: 6443
      name: probe-port
      protocol: TCP

    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates        

-----

sed -n '1,220p' /etc/kubernetes/manifests/etcd.yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.100.50:2379
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.100.50:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --feature-gates=InitialCorruptCheck=true
    - --initial-advertise-peer-urls=https://192.168.100.50:2380
    - --initial-cluster=deep-cp-01=https://192.168.100.50:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.100.50:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.100.50:2380
    - --name=deep-cp-01
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --watch-progress-notify-interval=5s
    image: registry.k8s.io/etcd:3.6.8-0
    imagePullPolicy: IfNotPresent

    name: etcd
    ports:
    - containerPort: 2381
      name: probe-port
      protocol: TCP

    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data            

-----

root@deep-cp-01:/etc/kubernetes# ls -l /etc/kubernetes/*.conf
-rw------- 1 root root 5638 Jun  1 17:40 /etc/kubernetes/admin.conf
-rw------- 1 root root 5662 Jun  1 17:40 /etc/kubernetes/controller-manager.conf
-rw------- 1 root root 1970 Jun  1 17:40 /etc/kubernetes/kubelet.conf
-rw------- 1 root root 5610 Jun  1 17:40 /etc/kubernetes/scheduler.conf
-rw------- 1 root root 5666 Jun  1 17:40 /etc/kubernetes/super-admin.conf 

root@deep-cp-01:/etc/kubernetes# find /etc/kubernetes/pki -maxdepth 2 -type f | sort
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

root@deep-cp-01:/etc/kubernetes# kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 01, 2027 17:39 UTC   358d            ca                      no      
apiserver                  Jun 01, 2027 17:39 UTC   358d            ca                      no      
apiserver-etcd-client      Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
apiserver-kubelet-client   Jun 01, 2027 17:39 UTC   358d            ca                      no      
controller-manager.conf    Jun 01, 2027 17:39 UTC   358d            ca                      no      
etcd-healthcheck-client    Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
etcd-peer                  Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
etcd-server                Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
front-proxy-client         Jun 01, 2027 17:39 UTC   358d            front-proxy-ca          no      
scheduler.conf             Jun 01, 2027 17:39 UTC   358d            ca                      no      
super-admin.conf           Jun 01, 2027 17:39 UTC   358d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      May 29, 2036 17:39 UTC   9y              no      
etcd-ca                 May 29, 2036 17:39 UTC   9y              no      
front-proxy-ca          May 29, 2036 17:39 UTC   9y              no

-----

root@deep-cp-01:/etc/kubernetes# systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2026-06-08 10:37:32 UTC; 2h 52min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 801 (kubelet)
      Tasks: 13 (limit: 9317)
     Memory: 60.5M (peak: 61.3M)
        CPU: 6min 40.192s
     CGroup: /system.slice/kubelet.service
             └─801 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 08 13:27:59 deep-cp-01 kubelet[801]: E0608 13:27:59.527476     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:28:08 deep-cp-01 kubelet[801]: E0608 13:28:08.535734     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:28:20 deep-cp-01 kubelet[801]: E0608 13:28:20.527905     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:28:33 deep-cp-01 kubelet[801]: E0608 13:28:33.527900     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:28:38 deep-cp-01 kubelet[801]: E0608 13:28:38.527884     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:28:53 deep-cp-01 kubelet[801]: E0608 13:28:53.527553     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:29:03 deep-cp-01 kubelet[801]: E0608 13:29:03.527718     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:29:19 deep-cp-01 kubelet[801]: E0608 13:29:19.527575     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:29:21 deep-cp-01 kubelet[801]: E0608 13:29:21.527378     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Jun 08 13:29:26 deep-cp-01 kubelet[801]: E0608 13:29:26.528364     801 dns.go:154] "Nameserver limits exceeded" err="Nameserver limits were exceeded, some nameservers have been omitted, the applied nameser….8.8 192.168.1.12"
Hint: Some lines were ellipsized, use -l to show in full.

-----

root@deep-cp-01:/etc/kubernetes# crictl pods
POD ID              CREATED             STATE               NAME                                       NAMESPACE           ATTEMPT             RUNTIME
5452aeeb80d97       3 hours ago         Ready               coredns-589f44dc88-mslt6                   kube-system         1                   (default)
7854a20d3fbf6       3 hours ago         Ready               whisker-5cb7d57d4c-bk5cm                   calico-system       1                   (default)
3c5fae41ab9c6       3 hours ago         Ready               goldmane-6885dcb7d-hgtvj                   calico-system       1                   (default)
cb365fa8503bb       3 hours ago         Ready               calico-apiserver-9c45c8f8f-5fgwk           calico-system       1                   (default)
7c93b2a5ebc04       3 hours ago         Ready               calico-kube-controllers-5b8d487f58-6jb5z   calico-system       1                   (default)
22076e7c831e9       3 hours ago         Ready               calico-apiserver-9c45c8f8f-zq7sh           calico-system       1                   (default)
f84da673536cd       3 hours ago         Ready               coredns-589f44dc88-sd4kh                   kube-system         1                   (default)
082144d3b609b       3 hours ago         Ready               csi-node-driver-dwsqc                      calico-system       1                   (default)
7e9d477634fa9       3 hours ago         Ready               calico-node-p4sjq                          calico-system       1                   (default)
b6341fdaf2fba       3 hours ago         Ready               kube-proxy-94t92                           kube-system         2                   (default)
497c8e7d5bbb1       3 hours ago         Ready               kube-scheduler-deep-cp-01                  kube-system         2                   (default)
c4768e7302448       3 hours ago         Ready               kube-controller-manager-deep-cp-01         kube-system         2                   (default)
e202779a867c5       3 hours ago         Ready               kube-apiserver-deep-cp-01                  kube-system         2                   (default)
7c9983f6b1632       3 hours ago         Ready               etcd-deep-cp-01                            kube-system         2                   (default)
e47b115d4d1af       4 days ago          NotReady            calico-kube-controllers-5b8d487f58-6jb5z   calico-system       0                   (default)
afc347efb2f77       4 days ago          NotReady            calico-apiserver-9c45c8f8f-5fgwk           calico-system       0                   (default)
e5d61d72f512a       4 days ago          NotReady            coredns-589f44dc88-sd4kh                   kube-system         0                   (default)
08210bf1b4bcc       4 days ago          NotReady            coredns-589f44dc88-mslt6                   kube-system         0                   (default)
02c7cbbc7878f       4 days ago          NotReady            calico-apiserver-9c45c8f8f-zq7sh           calico-system       0                   (default)
3a8875303c2a2       4 days ago          NotReady            whisker-5cb7d57d4c-bk5cm                   calico-system       0                   (default)
a4e0b30e2adab       4 days ago          NotReady            goldmane-6885dcb7d-hgtvj                   calico-system       0                   (default)
7ea8df86374e1       4 days ago          NotReady            csi-node-driver-dwsqc                      calico-system       0                   (default)
006276baa045b       4 days ago          NotReady            calico-node-p4sjq                          calico-system       0                   (default)
3b4dae6dd3aa3       5 days ago          NotReady            kube-proxy-94t92                           kube-system         1                   (default)
9596a071b4cbf       5 days ago          NotReady            kube-scheduler-deep-cp-01                  kube-system         1                   (default)
a7e29169a057a       5 days ago          NotReady            kube-controller-manager-deep-cp-01         kube-system         1                   (default)
499d01a3dd6f6       5 days ago          NotReady            kube-apiserver-deep-cp-01                  kube-system         1                   (default)
cd9f5bc76361d       5 days ago          NotReady            etcd-deep-cp-01                            kube-system         1                   (default)  

root@deep-cp-01:/etc/kubernetes# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                        NAMESPACE
0b33782263531       5f531f3cb4757       3 hours ago         Running             whisker-backend             1                   7854a20d3fbf6       whisker-5cb7d57d4c-bk5cm                   calico-system
9436bc786d86e       b4638c5ba691c       3 hours ago         Running             whisker                     1                   7854a20d3fbf6       whisker-5cb7d57d4c-bk5cm                   calico-system
8835dd3db6e4c       e67ce8d034ec5       3 hours ago         Running             calico-kube-controllers     1                   7c93b2a5ebc04       calico-kube-controllers-5b8d487f58-6jb5z   calico-system
12396584de8aa       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            1                   cb365fa8503bb       calico-apiserver-9c45c8f8f-5fgwk           calico-system
ddc38c21df47e       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   1                   082144d3b609b       csi-node-driver-dwsqc                      calico-system
34dee7160cbf2       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            1                   22076e7c831e9       calico-apiserver-9c45c8f8f-zq7sh           calico-system
9e6812ed0d791       38667dd9be96c       3 hours ago         Running             coredns                     1                   5452aeeb80d97       coredns-589f44dc88-mslt6                   kube-system
870a3afbdd29d       8b5156bd00aba       3 hours ago         Running             goldmane                    1                   3c5fae41ab9c6       goldmane-6885dcb7d-hgtvj                   calico-system
36137422380d0       38667dd9be96c       3 hours ago         Running             coredns                     1                   f84da673536cd       coredns-589f44dc88-sd4kh                   kube-system
30b276a22f336       a665cad044872       3 hours ago         Running             calico-csi                  1                   082144d3b609b       csi-node-driver-dwsqc                      calico-system
d73d49b2e050f       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 1                   7e9d477634fa9       calico-node-p4sjq                          calico-system
5bd993d2eeb2b       78282b5844742       3 hours ago         Running             kube-proxy                  2                   b6341fdaf2fba       kube-proxy-94t92                           kube-system
7aee9a9ff3ccd       ee85eb1f0edd2       3 hours ago         Running             etcd                        2                   7c9983f6b1632       etcd-deep-cp-01                            kube-system
20d421f04244c       b665198f31ae0       3 hours ago         Running             kube-scheduler              2                   497c8e7d5bbb1       kube-scheduler-deep-cp-01                  kube-system
3c17b22ccf365       c2616bc3c6956       3 hours ago         Running             kube-controller-manager     2                   c4768e7302448       kube-controller-manager-deep-cp-01         kube-system
c0b1d3a19f6ba       315157c5c4d76       3 hours ago         Running             kube-apiserver              2                   e202779a867c5       kube-apiserver-deep-cp-01                  kube-system
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2026-06-08 10:37:33 UTC; 2h 52min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 798 (kubelet)
      Tasks: 13 (limit: 9317)
     Memory: 44.1M (peak: 45.0M)
        CPU: 3min 9.814s
     CGroup: /system.slice/kubelet.service
             └─798 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 08 10:37:35 deep-wrk-01 kubelet[798]: I0608 10:37:35.141504     798 kubelet_network.go:48] "Updating Pod CIDR" originalPodCIDR="" newPodCIDR="192.168.1.0/24"
Jun 08 10:37:35 deep-wrk-01 kubelet[798]: I0608 10:37:35.338425     798 scope.go:122] "RemoveContainer" containerID="362185d37fdf8e402bc8e9752cd614235f03b0267332f04721b67ac5d8d37eb4"
Jun 08 10:37:35 deep-wrk-01 kubelet[798]: I0608 10:37:35.367079     798 scope.go:122] "RemoveContainer" containerID="f07615c0e9b3a1fc88f477623dc7b149b0065c2a053495dc2004473cfad30c73"
Jun 08 10:37:35 deep-wrk-01 kubelet[798]: I0608 10:37:35.410578     798 scope.go:122] "RemoveContainer" containerID="3df3b7688ac44e144e30e1634f2be92e5f70ac9f77a3f7e12a6330b279de9071"
Jun 08 10:37:37 deep-wrk-01 kubelet[798]: I0608 10:37:37.372062     798 prober_manager.go:356] "Failed to trigger a manual run" probe="Readiness"
Jun 08 10:37:40 deep-wrk-01 kubelet[798]: I0608 10:37:40.390957     798 scope.go:122] "RemoveContainer" containerID="e7e7cb80a37b3f83a49b584ca9736e131f9337e3cb5596a98d0ddd4f42fbb423"
Jun 08 10:37:51 deep-wrk-01 kubelet[798]: I0608 10:37:51.252436     798 prober_manager.go:356] "Failed to trigger a manual run" probe="Readiness"
Jun 08 10:38:01 deep-wrk-01 kubelet[798]: I0608 10:38:01.254894     798 prober_manager.go:356] "Failed to trigger a manual run" probe="Readiness"
Jun 08 10:38:08 deep-wrk-01 kubelet[798]: I0608 10:38:08.828165     798 csi_plugin.go:106] kubernetes.io/csi: Trying to validate a new CSI Driver with name: csi.tigera.io endpoint: /var/lib/kubelet/plugins…ck versions: 1.0.0
Jun 08 10:38:08 deep-wrk-01 kubelet[798]: I0608 10:38:08.828214     798 csi_plugin.go:119] kubernetes.io/csi: Register new plugin with name: csi.tigera.io at endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock
Hint: Some lines were ellipsized, use -l to show in full.

-----

root@deep-wrk-01:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                               NAMESPACE           ATTEMPT             RUNTIME
44310ff668987       3 hours ago         Ready               csi-node-driver-6xctm              calico-system       1                   (default)
eb65c5e171734       3 hours ago         Ready               calico-node-k7rg2                  calico-system       1                   (default)
5014ac3322c66       3 hours ago         Ready               kube-proxy-p7dff                   kube-system         2                   (default)
6391e6aadf4eb       3 hours ago         Ready               calico-typha-5bb76474df-9ch4x      calico-system       1                   (default)
c690998b716e5       3 hours ago         Ready               tigera-operator-85dbff4478-fjdxd   tigera-operator     1                   (default)
2f57e1bd2507c       4 days ago          NotReady            csi-node-driver-6xctm              calico-system       0                   (default)
cfdb7ed86b8a4       4 days ago          NotReady            calico-typha-5bb76474df-9ch4x      calico-system       0                   (default)
c58e60803ea8c       4 days ago          NotReady            calico-node-k7rg2                  calico-system       0                   (default)
5b586e55dc09d       4 days ago          NotReady            tigera-operator-85dbff4478-fjdxd   tigera-operator     0                   (default)
4d1a8670469a0       5 days ago          NotReady            kube-proxy-p7dff                   kube-system         1                   (default)
root@deep-wrk-01:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                NAMESPACE
91bef206ef79a       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   1                   44310ff668987       csi-node-driver-6xctm              calico-system
98e564647512f       a665cad044872       3 hours ago         Running             calico-csi                  1                   44310ff668987       csi-node-driver-6xctm              calico-system
20b90ca2aa479       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 1                   eb65c5e171734       calico-node-k7rg2                  calico-system
38cc1cb7ac108       a4433ba6d0851       3 hours ago         Running             calico-typha                1                   6391e6aadf4eb       calico-typha-5bb76474df-9ch4x      calico-system
36aa985236a95       78282b5844742       3 hours ago         Running             kube-proxy                  2                   5014ac3322c66       kube-proxy-p7dff                   kube-system
b31924d2db66e       2d636341e3971       3 hours ago         Running             tigera-operator             2                   c690998b716e5       tigera-operator-85dbff4478-fjdxd   tigera-operator  


```

**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2026-06-08 10:37:36 UTC; 2h 53min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 793 (kubelet)
      Tasks: 13 (limit: 9317)
     Memory: 43.4M (peak: 44.1M)
        CPU: 3min 3.357s
     CGroup: /system.slice/kubelet.service
             └─793 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 08 10:37:42 deep-wrk-02 kubelet[793]: I0608 10:37:42.324945     793 scope.go:122] "RemoveContainer" containerID="77fb2bca18dfae958faa1fcf7eb30eae0fc7a52e24378f731062573cd9d127f5"
Jun 08 10:38:04 deep-wrk-02 kubelet[793]: I0608 10:38:04.575439     793 prober_manager.go:356] "Failed to trigger a manual run" probe="Readiness"
Jun 08 10:38:07 deep-wrk-02 kubelet[793]: E0608 10:38:07.217027     793 kubelet.go:2646] "Skipping pod synchronization" err="container runtime is down"
Jun 08 10:38:07 deep-wrk-02 kubelet[793]: E0608 10:38:07.317561     793 kubelet.go:2646] "Skipping pod synchronization" err="container runtime is down"
Jun 08 10:38:07 deep-wrk-02 kubelet[793]: E0608 10:38:07.517940     793 kubelet.go:2646] "Skipping pod synchronization" err="container runtime is down"
Jun 08 10:38:07 deep-wrk-02 kubelet[793]: I0608 10:38:07.818418     793 setters.go:547] "Node became not ready" node="deep-wrk-02" condition={"type":"Ready","status":"False","lastHeartbeatTime":"2026-06-08… runtime is down"}
Jun 08 10:38:07 deep-wrk-02 kubelet[793]: E0608 10:38:07.918105     793 kubelet.go:2646] "Skipping pod synchronization" err="container runtime is down"
Jun 08 10:38:08 deep-wrk-02 kubelet[793]: E0608 10:38:08.718741     793 kubelet.go:2646] "Skipping pod synchronization" err="container runtime is down"
Jun 08 10:38:11 deep-wrk-02 kubelet[793]: I0608 10:38:11.175135     793 csi_plugin.go:106] kubernetes.io/csi: Trying to validate a new CSI Driver with name: csi.tigera.io endpoint: /var/lib/kubelet/plugins…ck versions: 1.0.0
Jun 08 10:38:11 deep-wrk-02 kubelet[793]: I0608 10:38:11.175188     793 csi_plugin.go:119] kubernetes.io/csi: Register new plugin with name: csi.tigera.io at endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock
Hint: Some lines were ellipsized, use -l to show in full.  

-----

root@deep-wrk-02:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                            NAMESPACE           ATTEMPT             RUNTIME
37a8e3e16dbf2       3 hours ago         Ready               csi-node-driver-dc2pq           calico-system       1                   (default)
f49aad5e5dc07       3 hours ago         Ready               calico-node-fcxdj               calico-system       1                   (default)
5b6b241f79651       3 hours ago         Ready               calico-typha-5bb76474df-6h8tk   calico-system       1                   (default)
6545f0c2a2044       3 hours ago         Ready               kube-proxy-gcrzz                kube-system         2                   (default)
27c258ae9dbe9       4 days ago          NotReady            csi-node-driver-dc2pq           calico-system       0                   (default)
e8edc8367f3a4       4 days ago          NotReady            calico-node-fcxdj               calico-system       0                   (default)
27b432ba22cce       4 days ago          NotReady            calico-typha-5bb76474df-6h8tk   calico-system       0                   (default)
267d13796aa48       5 days ago          NotReady            kube-proxy-gcrzz                kube-system         1                   (default)
root@deep-wrk-02:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                             NAMESPACE
714ebc56f164f       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   1                   37a8e3e16dbf2       csi-node-driver-dc2pq           calico-system
98159c55b312e       a665cad044872       3 hours ago         Running             calico-csi                  1                   37a8e3e16dbf2       csi-node-driver-dc2pq           calico-system
d0a80beb7855f       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 1                   f49aad5e5dc07       calico-node-fcxdj               calico-system
aefe886129709       a4433ba6d0851       3 hours ago         Running             calico-typha                1                   5b6b241f79651       calico-typha-5bb76474df-6h8tk   calico-system
869d1bc9fc328       78282b5844742       3 hours ago         Running             kube-proxy                  2                   6545f0c2a2044       kube-proxy-gcrzz                kube-system  


```

### Ключевые выводы команд

**deep-cp-01**:  
```
root@deep-cp-01:/etc/kubernetes# kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 01, 2027 17:39 UTC   358d            ca                      no      
apiserver                  Jun 01, 2027 17:39 UTC   358d            ca                      no      
apiserver-etcd-client      Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
apiserver-kubelet-client   Jun 01, 2027 17:39 UTC   358d            ca                      no      
controller-manager.conf    Jun 01, 2027 17:39 UTC   358d            ca                      no      
etcd-healthcheck-client    Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
etcd-peer                  Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
etcd-server                Jun 01, 2027 17:39 UTC   358d            etcd-ca                 no      
front-proxy-client         Jun 01, 2027 17:39 UTC   358d            front-proxy-ca          no      
scheduler.conf             Jun 01, 2027 17:39 UTC   358d            ca                      no      
super-admin.conf           Jun 01, 2027 17:39 UTC   358d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      May 29, 2036 17:39 UTC   9y              no      
etcd-ca                 May 29, 2036 17:39 UTC   9y              no      
front-proxy-ca          May 29, 2036 17:39 UTC   9y              no

```
**deep-cp-01**:  
```
root@deep-cp-01:/etc/kubernetes# kubectl run alpine-test --image alpine:latest --command -- /bin/sh -c "sleep infinity"
pod/alpine-test created  
root@deep-cp-01:/etc/kubernetes# kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP               NODE          NOMINATED NODE   READINESS GATES
alpine-test   1/1     Running   0          49s   192.168.19.134   deep-wrk-02   <none>           <none>
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# crictl pods
POD ID              CREATED              STATE               NAME                            NAMESPACE           ATTEMPT             RUNTIME
82c3f6fed584a       About a minute ago   Ready               alpine-test                     default             0                   (default)
...
root@deep-wrk-02:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED              STATE               NAME                        ATTEMPT             POD ID              POD                             NAMESPACE
2d3482d6d4760       3cb067eab6096       About a minute ago   Running             alpine-test                 0                   82c3f6fed584a       alpine-test                     default
...
```

### Что стало понятнее

- Tigera Operator не использует для разворачивания `calico` ns `kube-system`, а создаёт другие ns
- Все серты хранятся только локально, нужно обязательно делать копии
- помимо сертов нужно копировать /etc/kubernetes/* и данные etcd, однако в текущей конфигурации это проблемный момент т.к. для снятия снэпшота нужно остановить ноду etcd, но она одна и кластер ляжет на время выполнения операции.

### Вопросы

- Хотя kubeadm собирает кластер быстро, есть проблемное место с организацие отказоустойчивости - ноды etcd оно располагает исключительно на control plane, но 3 controller-managera одновременно работать не будут - два из них будут курить бамбук и ждать смерти лидера - будет возникать временной лаг на время переключения, и, поскольку, нет никакого балансировщика на порту 6443 схема не будет отказоустлйчивой для внешних подключений типа kubectl. В текущей конфигурации это не важно, но в общем смысле такое поведенеи нужно именть ввиду.

#### Вывод  
Кластер работоспособен, через systemd запускается только kubelet. остальная обвязка - static pods, но таких вариантов установки абсолютное большинство. Тот же kubespray делает аналогичным образом, или k3s, к примеру.  
Нужно обязятельно хранить резервную копию всех данных конфигурации/сертов, при этом секъюрно - там будут чуствительные данные.  
Резервная копия данных c etcd тоже проблемна в такой конфигурации, соответственно такой сетап учебный или тестовый :)
Однако, kubeadm позволяет достаточно просто перевыпускать сертификаты, в чём его не сомненный плюс.

### Статус

Done
