# Lab 06 - Разбор kubeadm-кластера после установки

### Дата

2026-06-18

## Цель

Разобрать уже созданный Kubernetes cluster и понять, что именно подготовил `kubeadm`.

``` nodes
ae@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   12d   v1.36.1   192.168.1.40   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          Ready    <none>          12d   v1.36.1   192.168.1.43   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          Ready    <none>          12d   v1.36.1   192.168.1.46   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
```

``` system pods
ae@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS       AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-7hmjk                  1/1     Running   3 (98m ago)    12d   192.168.226.83    worker-1          <none>           <none>
coredns-589f44dc88-cmbt2                  1/1     Running   3 (99m ago)    12d   192.168.133.207   worker-2          <none>           <none>
etcd-control-plane-1                      1/1     Running   14 (99m ago)   12d   192.168.1.40      control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   14 (99m ago)   12d   192.168.1.40      control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   8 (97m ago)    12d   192.168.1.40      control-plane-1   <none>           <none>
kube-proxy-4kvmt                          1/1     Running   3 (99m ago)    12d   192.168.1.40      control-plane-1   <none>           <none>
kube-proxy-62wxd                          1/1     Running   3 (98m ago)    12d   192.168.1.43      worker-1          <none>           <none>
kube-proxy-hgk4d                          1/1     Running   3 (99m ago)    12d   192.168.1.46      worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   6 (99m ago)    12d   192.168.1.40      control-plane-1   <none>           <none>
```

## Шаг - прочитать manifest API Server
``` manifests
ae@control-plane-1:~$ sudo ls -l /etc/kubernetes/manifests/
[sudo] password for ae: 
total 16
-rw------- 1 root root 2610 Jun  6 15:28 etcd.yaml
-rw------- 1 root root 3954 Jun  6 15:28 kube-apiserver.yaml
-rw------- 1 root root 3230 Jun  6 15:28 kube-controller-manager.yaml
-rw------- 1 root root 1726 Jun  6 15:28 kube-scheduler.yaml
```

```kube-apiserver.yaml
container image : registry.k8s.io/kube-apiserver:v1.36.1
command:
 kube-apiserver
    - --advertise-address=192.168.1.40
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
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key |
bind address: 192.168.1.40
secure port: 6443
paths к certificates: /etc/kubernetes/pki/apiserver.crt
путь к etcd certificates: /etc/kubernetes/pki/apiserver-etcd-client.crt
admission plugins: NodeRestriction
service cluster IP range: 10.96.0.0/12
client CA file: /etc/kubernetes/pki/ca.crt
```

## Шаг - прочитать manifest etcd

``` etcd
data-dir=/var/lib/etcd
cert-file=/etc/kubernetes/pki/etcd/server.crt
peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
listen-client-urls=https://127.0.0.1:2379,https://192.168.1.40:2379
listen-metrics-urls=http://127.0.0.1:2381
listen-peer-urls=https://192.168.1.40:2380
advertise-client-urls=https://192.168.1.40:2379
```

как API Server подключается к etcd: в kube-apiserver.yaml --etcd-servers=https://127.0.0.1:2379

## Шаг - посмотреть kubeconfig-файлы

```
ae@control-plane-1:~$ sudo ls -l /etc/kubernetes/*.conf
-rw------- 1 root root 5640 Jun  6 15:28 /etc/kubernetes/admin.conf
-rw------- 1 root root 5660 Jun  6 15:28 /etc/kubernetes/controller-manager.conf
-rw------- 1 root root 1988 Jun  6 15:29 /etc/kubernetes/kubelet.conf
-rw------- 1 root root 5608 Jun  6 15:28 /etc/kubernetes/scheduler.conf
-rw------- 1 root root 5660 Jun  6 15:28 /etc/kubernetes/super-admin.conf
```

## Шаг - посмотреть certificates

```
ae@control-plane-1:~$ sudo find /etc/kubernetes/pki -maxdepth 2 -type f | sort
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
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

## Шаг - проверить сроки certificates
```
ae@control-plane-1:~$ sudo kubeadm certs check-expiration
[check-expiration] Reading configuration from the "kubeadm-config" ConfigMap in namespace "kube-system"...
[check-expiration] Use 'kubeadm init phase upload-config kubeadm --config your-config-file' to re-upload it.

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Jun 06, 2027 15:28 UTC   352d            ca                      no      
apiserver                  Jun 06, 2027 15:28 UTC   352d            ca                      no      
apiserver-etcd-client      Jun 06, 2027 15:28 UTC   352d            etcd-ca                 no      
apiserver-kubelet-client   Jun 06, 2027 15:28 UTC   352d            ca                      no      
controller-manager.conf    Jun 06, 2027 15:28 UTC   352d            ca                      no      
etcd-healthcheck-client    Jun 06, 2027 15:28 UTC   352d            etcd-ca                 no      
etcd-peer                  Jun 06, 2027 15:28 UTC   352d            etcd-ca                 no      
etcd-server                Jun 06, 2027 15:28 UTC   352d            etcd-ca                 no      
front-proxy-client         Jun 06, 2027 15:28 UTC   352d            front-proxy-ca          no      
scheduler.conf             Jun 06, 2027 15:28 UTC   352d            ca                      no      
super-admin.conf           Jun 06, 2027 15:28 UTC   352d            ca                      no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Jun 03, 2036 15:28 UTC   9y              no      
etcd-ca                 Jun 03, 2036 15:28 UTC   9y              no      
front-proxy-ca          Jun 03, 2036 15:28 UTC   9y              no      
```

## Шаг - проверить kubelet

``` cp
ae@control-plane-1:~$ systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Thu 2026-06-18 16:31:15 UTC; 4h 14min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 1120 (kubelet)
      Tasks: 14 (limit: 4601)
     Memory: 49.9M (peak: 51.0M)
        CPU: 5min 46.277s
     CGroup: /system.slice/kubelet.service
             └─1120 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 18 16:34:33 control-plane-1 kubelet[1120]: E0618 16:34:33.927475    1120 kuberuntime_gc.go:182] "Failed to stop sandbox before removing" err="rpc error: code = DeadlineExceeded desc = context deadline exceeded" sandboxID…4d1bb14a8f06b94c8c6"
Jun 18 16:34:39 control-plane-1 kubelet[1120]: E0618 16:34:39.282032    1120 remote_runtime.go:267] "StopPodSandbox from runtime service failed" err="rpc error: code = DeadlineExceeded desc = stream terminated by RST_STREAM …81ba17f9f695a4b7e34"
Jun 18 16:34:39 control-plane-1 kubelet[1120]: E0618 16:34:39.282085    1120 kuberuntime_manager.go:1967] "Failed to stop sandbox" podSandboxID={"Type":"containerd","ID":"59b1cff4ad6dd4f9761bd516b9a4a282f65d1b3a5bc2681ba17f9f695a4b7e34"}
Jun 18 16:34:51 control-plane-1 kubelet[1120]: E0618 16:34:51.684304    1120 kuberuntime_manager.go:1478] "killPodWithSyncResult failed" err="failed to \"KillPodSandbox\" for \"daa4f30f-04cb-44ed-8394-b746f92d591e\" with Kil…rror code: CANCEL\""
Jun 18 16:34:51 control-plane-1 kubelet[1120]: E0618 16:34:51.684408    1120 pod_workers.go:1324] "Error syncing pod, skipping" err="failed to \"KillPodSandbox\" for \"daa4f30f-04cb-44ed-8394-b746f92d591e\" with KillPodSandboxError: \"rpc error…
Jun 18 16:35:47 control-plane-1 kubelet[1120]: I0618 16:35:47.346219    1120 csi_plugin.go:106] kubernetes.io/csi: Trying to validate a new CSI Driver with name: csi.tigera.io endpoint: /var/lib/kubelet/plugins/csi.tigera.io…sock versions: 1.0.0
Jun 18 16:35:47 control-plane-1 kubelet[1120]: I0618 16:35:47.346255    1120 csi_plugin.go:119] kubernetes.io/csi: Register new plugin with name: csi.tigera.io at endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock
Jun 18 16:36:21 control-plane-1 kubelet[1120]: I0618 16:36:21.970111    1120 scope.go:122] "RemoveContainer" containerID="78f1eae24b75c68e610d9dbfe2464d5a01400beffb8dcf033ca5ef9a045a510e"
Jun 18 16:43:50 control-plane-1 kubelet[1120]: E0618 16:43:50.492354    1120 controller.go:251] "Failed to update lease" err="etcdserver: request timed out"
Jun 18 16:43:52 control-plane-1 kubelet[1120]: E0618 16:43:52.396545    1120 controller.go:251] "Failed to update lease" err="Operation cannot be fulfilled on leases.coordination.k8s.io \"control-plane-1\": the object has be…rsion and try again"
Hint: Some lines were ellipsized, use -l to show in full.
```

``` w1
ae@worker-1:~$ systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Thu 2026-06-18 16:31:37 UTC; 3h 9min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 899 (kubelet)
      Tasks: 20 (limit: 4601)
     Memory: 48.3M (peak: 49.1M)
        CPU: 3min 8.335s
     CGroup: /system.slice/kubelet.service
             └─899 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 18 16:36:29 worker-1 kubelet[899]: I0618 16:36:29.677797     899 scope.go:122] "RemoveContainer" containerID="bed31f20ac8e3088740a25b3a52d52945ee1b5ebc95cecec69572e28a358622c"
Jun 18 16:36:34 worker-1 kubelet[899]: E0618 16:36:34.730373     899 remote_runtime.go:636] "ExecSync cmd from runtime service failed" err="rpc error: code = DeadlineExceeded desc = failed to exec in container: timeout 5s exceeded: context dead…
Jun 18 16:36:41 worker-1 kubelet[899]: E0618 16:36:41.234074     899 remote_runtime.go:636] "ExecSync cmd from runtime service failed" err="rpc error: code = DeadlineExceeded desc = failed to exec in container: timeout 5s exceeded: context dead…
Jun 18 16:36:49 worker-1 kubelet[899]: E0618 16:36:48.989169     899 remote_runtime.go:636] "ExecSync cmd from runtime service failed" err="rpc error: code = DeadlineExceeded desc = failed to exec in container: timeout 5s exceeded: context dead…
Jun 18 16:36:57 worker-1 kubelet[899]: I0618 16:36:57.289041     899 csi_plugin.go:106] kubernetes.io/csi: Trying to validate a new CSI Driver with name: csi.tigera.io endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock versions: 1.0.0
Jun 18 16:36:57 worker-1 kubelet[899]: I0618 16:36:57.289086     899 csi_plugin.go:119] kubernetes.io/csi: Register new plugin with name: csi.tigera.io at endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock
Jun 18 16:37:10 worker-1 kubelet[899]: E0618 16:37:10.712185     899 remote_runtime.go:636] "ExecSync cmd from runtime service failed" err="rpc error: code = DeadlineExceeded desc = failed to exec in container: timeout 5s exceeded: context dead…
Jun 18 16:37:40 worker-1 kubelet[899]: E0618 16:37:40.718802     899 remote_runtime.go:636] "ExecSync cmd from runtime service failed" err="rpc error: code = DeadlineExceeded desc = failed to exec in container: timeout 5s exceeded: context dead…
Jun 18 16:43:50 worker-1 kubelet[899]: E0618 16:43:50.481751     899 controller.go:251] "Failed to update lease" err="etcdserver: request timed out"
Jun 18 16:43:52 worker-1 kubelet[899]: E0618 16:43:52.391499     899 controller.go:251] "Failed to update lease" err="Operation cannot be fulfilled on leases.coordination.k8s.io \"worker-1\": the object has been modified; pl…rsion and try again"
Hint: Some lines were ellipsized, use -l to show in full.
```

```w2
ae@worker-2:~$ systemctl status kubelet --no-pager
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Thu 2026-06-18 16:31:29 UTC; 3h 10min ago
       Docs: https://kubernetes.io/docs/
   Main PID: 928 (kubelet)
      Tasks: 18 (limit: 4601)
     Memory: 48.0M (peak: 48.8M)
        CPU: 3min 9.697s
     CGroup: /system.slice/kubelet.service
             └─928 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml

Jun 18 16:35:10 worker-2 kubelet[928]: E0618 16:35:10.409821     928 pod_workers.go:1324] "Error syncing pod, skipping" err="failed to \"KillPodSandbox\" for \"73ea7818-2ccd-4bfc-a3b6-7eb71b34e53f\" with KillPodSandboxError: \"rpc error: code =…
Jun 18 16:35:10 worker-2 kubelet[928]: E0618 16:35:10.420188     928 kuberuntime_manager.go:1478] "killPodWithSyncResult failed" err="failed to \"KillPodSandbox\" for \"54c13c03-6ef1-4efc-8c4b-b14e825bf9c7\" with KillPodSandboxError: \"rpc erro…
Jun 18 16:35:10 worker-2 kubelet[928]: E0618 16:35:10.420261     928 pod_workers.go:1324] "Error syncing pod, skipping" err="failed to \"KillPodSandbox\" for \"54c13c03-6ef1-4efc-8c4b-b14e825bf9c7\" with KillPodSandboxError: \"rpc error: code =…
Jun 18 16:35:11 worker-2 kubelet[928]: E0618 16:35:11.871187     928 kuberuntime_manager.go:1478] "killPodWithSyncResult failed" err="failed to \"KillPodSandbox\" for \"83ca93e8-9ff5-4d76-9a20-c7f6cd86d360\" with KillPodSandboxError: \"rpc erro…
Jun 18 16:35:11 worker-2 kubelet[928]: E0618 16:35:11.871239     928 pod_workers.go:1324] "Error syncing pod, skipping" err="failed to \"KillPodSandbox\" for \"83ca93e8-9ff5-4d76-9a20-c7f6cd86d360\" with KillPodSandboxError: \"rpc error: code =…
Jun 18 16:37:14 worker-2 kubelet[928]: I0618 16:37:14.484987     928 scope.go:122] "RemoveContainer" containerID="4f612a6d48df0de242c0d1db6c502f288dc2f5c807ecda7c2cef6b4cc4e2bcd6"
Jun 18 16:37:44 worker-2 kubelet[928]: I0618 16:37:44.682016     928 csi_plugin.go:106] kubernetes.io/csi: Trying to validate a new CSI Driver with name: csi.tigera.io endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock versions: 1.0.0
Jun 18 16:37:44 worker-2 kubelet[928]: I0618 16:37:44.682080     928 csi_plugin.go:119] kubernetes.io/csi: Register new plugin with name: csi.tigera.io at endpoint: /var/lib/kubelet/plugins/csi.tigera.io/csi.sock
Jun 18 16:43:50 worker-2 kubelet[928]: E0618 16:43:50.482906     928 controller.go:251] "Failed to update lease" err="etcdserver: request timed out"
Jun 18 16:43:52 worker-2 kubelet[928]: E0618 16:43:52.391616     928 controller.go:251] "Failed to update lease" err="Operation cannot be fulfilled on leases.coordination.k8s.io \"worker-2\": the object has been modified; pl…rsion and try again"
Hint: Some lines were ellipsized, use -l to show in full.
```

## Шаг - проверить container runtime через crictl

```cp
ae@control-plane-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                                      NAMESPACE           ATTEMPT             RUNTIME
82549d488bc1d       4 hours ago         Ready               csi-node-driver-bcvhd                     calico-system       3                   (default)
f3faa858ba4a2       4 hours ago         Ready               calico-node-cjhgd                         calico-system       3                   (default)
d7f62b780e616       4 hours ago         Ready               kube-proxy-4kvmt                          kube-system         3                   (default)
5bd6d115dcdd0       4 hours ago         Ready               kube-scheduler-control-plane-1            kube-system         3                   (default)
d77fc82ee10d9       4 hours ago         Ready               etcd-control-plane-1                      kube-system         3                   (default)
f893f5c85e6cb       4 hours ago         Ready               kube-controller-manager-control-plane-1   kube-system         3                   (default)
105984f5b3c7d       4 hours ago         Ready               kube-apiserver-control-plane-1            kube-system         3                   (default)
59b1cff4ad6dd       24 hours ago        NotReady            csi-node-driver-bcvhd                     calico-system       2                   (default)
00d4cf151b31a       24 hours ago        NotReady            calico-node-cjhgd                         calico-system       2                   (default)
0d7ed3a464a39       24 hours ago        NotReady            kube-proxy-4kvmt                          kube-system         2                   (default)
df3250f1312f0       24 hours ago        NotReady            etcd-control-plane-1                      kube-system         2                   (default)
1e76e66e0cb04       24 hours ago        NotReady            kube-apiserver-control-plane-1            kube-system         2                   (default)
831ed998c90a7       24 hours ago        NotReady            kube-scheduler-control-plane-1            kube-system         2                   (default)
ae@control-plane-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                       NAMESPACE
8cb4983e937af       6bc9fa4dc2b10       4 hours ago         Running             calico-node                 3                   f3faa858ba4a2       calico-node-cjhgd                         calico-system
a1cadd889418a       ec9c47a649284       4 hours ago         Running             csi-node-driver-registrar   3                   82549d488bc1d       csi-node-driver-bcvhd                     calico-system
fcce9c3638b69       a665cad044872       4 hours ago         Running             calico-csi                  3                   82549d488bc1d       csi-node-driver-bcvhd                     calico-system
d21699bac9535       c2616bc3c6956       4 hours ago         Running             kube-controller-manager     8                   f893f5c85e6cb       kube-controller-manager-control-plane-1   kube-system
db7b41ce1a677       78282b5844742       4 hours ago         Running             kube-proxy                  3                   d7f62b780e616       kube-proxy-4kvmt                          kube-system
54a9f28ede95a       ee85eb1f0edd2       4 hours ago         Running             etcd                        14                  d77fc82ee10d9       etcd-control-plane-1                      kube-system
8471697ebf113       315157c5c4d76       4 hours ago         Running             kube-apiserver              14                  105984f5b3c7d       kube-apiserver-control-plane-1            kube-system
624a56f555f11       b665198f31ae0       4 hours ago         Running             kube-scheduler              6                   5bd6d115dcdd0       kube-scheduler-control-plane-1            kube-system
```

```w1
ae@worker-1:~$ sudo crictl pods
[sudo] password for ae: 
POD ID              CREATED             STATE               NAME                                      NAMESPACE           ATTEMPT             RUNTIME
7a56f65e361f8       3 hours ago         Ready               csi-node-driver-pnrcf                     calico-system       3                   (default)
3b5c604c38322       3 hours ago         Ready               calico-apiserver-6465b79596-55xmq         calico-system       3                   (default)
22c8035ad641c       3 hours ago         Ready               calico-kube-controllers-c67bd4bdd-fszg6   calico-system       3                   (default)
256b7e37789b1       3 hours ago         Ready               coredns-589f44dc88-7hmjk                  kube-system         3                   (default)
1edb42480efef       3 hours ago         Ready               goldmane-6885dcb7d-z4ppt                  calico-system       3                   (default)
196aeb88170eb       3 hours ago         Ready               calico-node-qn88q                         calico-system       3                   (default)
cd3aa0395179f       3 hours ago         Ready               kube-proxy-62wxd                          kube-system         3                   (default)
4eaf57ae61c55       3 hours ago         Ready               calico-typha-7c9696784c-49xff             calico-system       3                   (default)
4a224e6bc07c5       23 hours ago        NotReady            csi-node-driver-pnrcf                     calico-system       2                   (default)
19bbc63580777       23 hours ago        NotReady            goldmane-6885dcb7d-z4ppt                  calico-system       2                   (default)
0683accd27960       23 hours ago        NotReady            coredns-589f44dc88-7hmjk                  kube-system         2                   (default)
c0b4fe689801c       23 hours ago        NotReady            calico-kube-controllers-c67bd4bdd-fszg6   calico-system       2                   (default)
a0dd9a719da1d       23 hours ago        NotReady            calico-apiserver-6465b79596-55xmq         calico-system       2                   (default)
00d8df520a960       23 hours ago        NotReady            calico-node-qn88q                         calico-system       2                   (default)
b710b6406475f       23 hours ago        NotReady            kube-proxy-62wxd                          kube-system         2                   (default)
b2de4c96477e0       23 hours ago        NotReady            calico-typha-7c9696784c-49xff             calico-system       2                   (default)
ae@worker-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                       NAMESPACE
f047300b3cf62       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 3                   196aeb88170eb       calico-node-qn88q                         calico-system
dd55b6a19b4fc       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   3                   7a56f65e361f8       csi-node-driver-pnrcf                     calico-system
c126c64778b4c       a665cad044872       3 hours ago         Running             calico-csi                  3                   7a56f65e361f8       csi-node-driver-pnrcf                     calico-system
4effb5ff7dd9e       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            3                   3b5c604c38322       calico-apiserver-6465b79596-55xmq         calico-system
ccca886956008       e67ce8d034ec5       3 hours ago         Running             calico-kube-controllers     3                   22c8035ad641c       calico-kube-controllers-c67bd4bdd-fszg6   calico-system
cdb0319ec28ab       38667dd9be96c       3 hours ago         Running             coredns                     3                   256b7e37789b1       coredns-589f44dc88-7hmjk                  kube-system
8fa85ff0bcc25       8b5156bd00aba       3 hours ago         Running             goldmane                    3                   1edb42480efef       goldmane-6885dcb7d-z4ppt                  calico-system
6fdebbfb104e9       78282b5844742       3 hours ago         Running             kube-proxy                  3                   cd3aa0395179f       kube-proxy-62wxd                          kube-system
d5c7377a3ee60       a4433ba6d0851       3 hours ago         Running             calico-typha                3                   4eaf57ae61c55       calico-typha-7c9696784c-49xff             calico-system

```


```w2
ae@worker-2:~$ sudo crictl pods
[sudo] password for ae: 
POD ID              CREATED             STATE               NAME                                NAMESPACE           ATTEMPT             RUNTIME
ae92f730c9b3f       3 hours ago         Ready               csi-node-driver-t2w6b               calico-system       3                   (default)
2d6a03b89d590       3 hours ago         Ready               whisker-75fcc899c7-5njbv            calico-system       3                   (default)
6f0a4a2f2bfc1       3 hours ago         Ready               coredns-589f44dc88-cmbt2            kube-system         3                   (default)
bc19cc232a469       3 hours ago         Ready               calico-apiserver-6465b79596-px4q8   calico-system       3                   (default)
7e3383b695e37       3 hours ago         Ready               calico-node-jmj5r                   calico-system       3                   (default)
162d4622f4c8c       3 hours ago         Ready               calico-typha-7c9696784c-5bgjp       calico-system       3                   (default)
f930a3d3c0c0e       3 hours ago         Ready               tigera-operator-85dbff4478-cjnpc    tigera-operator     3                   (default)
d9988a72e4fff       3 hours ago         Ready               kube-proxy-hgk4d                    kube-system         3                   (default)
76a02927f377d       23 hours ago        NotReady            whisker-75fcc899c7-5njbv            calico-system       2                   (default)
ca6cffebd1869       23 hours ago        NotReady            csi-node-driver-t2w6b               calico-system       2                   (default)
caa231459ae58       23 hours ago        NotReady            coredns-589f44dc88-cmbt2            kube-system         2                   (default)
6cc58d00037ef       23 hours ago        NotReady            calico-apiserver-6465b79596-px4q8   calico-system       2                   (default)
8f0c61ba546fa       23 hours ago        NotReady            calico-node-jmj5r                   calico-system       2                   (default)
5a9080dabbba9       23 hours ago        NotReady            calico-typha-7c9696784c-5bgjp       calico-system       2                   (default)
0c53aa071ae0b       23 hours ago        NotReady            kube-proxy-hgk4d                    kube-system         2                   (default)
8bcb46158d599       23 hours ago        NotReady            tigera-operator-85dbff4478-cjnpc    tigera-operator     2                   (default)
ae@worker-2:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                        ATTEMPT             POD ID              POD                                 NAMESPACE
04a1ea3f814d6       ec9c47a649284       3 hours ago         Running             csi-node-driver-registrar   3                   ae92f730c9b3f       csi-node-driver-t2w6b               calico-system
d9c22e5524772       6bc9fa4dc2b10       3 hours ago         Running             calico-node                 5                   7e3383b695e37       calico-node-jmj5r                   calico-system
ea774e71817ab       a665cad044872       3 hours ago         Running             calico-csi                  3                   ae92f730c9b3f       csi-node-driver-t2w6b               calico-system
99bf6c7d61026       5f531f3cb4757       3 hours ago         Running             whisker-backend             3                   2d6a03b89d590       whisker-75fcc899c7-5njbv            calico-system
b1d565ef356ab       b4638c5ba691c       3 hours ago         Running             whisker                     3                   2d6a03b89d590       whisker-75fcc899c7-5njbv            calico-system
1b842cb2c6a72       38667dd9be96c       3 hours ago         Running             coredns                     3                   6f0a4a2f2bfc1       coredns-589f44dc88-cmbt2            kube-system
d9d19d5e01afd       c6246c3f0b5d8       3 hours ago         Running             calico-apiserver            5                   bc19cc232a469       calico-apiserver-6465b79596-px4q8   calico-system
4955e3935e512       a4433ba6d0851       3 hours ago         Running             calico-typha                3                   162d4622f4c8c       calico-typha-7c9696784c-5bgjp       calico-system
35e2b38b9b8db       78282b5844742       3 hours ago         Running             kube-proxy                  3                   d9988a72e4fff       kube-proxy-hgk4d                    kube-system
ff0b76c4ae4a3       2d636341e3971       3 hours ago         Running             tigera-operator             8                   f930a3d3c0c0e       tigera-operator-85dbff4478-cjnpc    tigera-operator
```

## Шаг - сопоставить Kubernetes и runtime layer

```
ae@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS         AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-7hmjk                  1/1     Running   3 (4h17m ago)    12d   192.168.226.83    worker-1          <none>           <none>
...
```

```
ae@worker-1:~$ sudo crictl pods
[sudo] password for ae: 
POD ID              CREATED             STATE               NAME                                      NAMESPACE           ATTEMPT             RUNTIME
...
256b7e37789b1       4 hours ago         Ready               coredns-589f44dc88-7hmjk                  kube-system         3                   (default)
...

cdb0319ec28ab       38667dd9be96c       4 hours ago         Running             coredns                     3                   256b7e37789b1       coredns-589f44dc88-7hmjk                  kube-system
```

## Контрольные вопросы
* Почему control plane components в kubeadm cluster запущены как static pods?

На этапе создания кластера ещё нет работающего кубера, который бы мог управлять обычными подами, поэтому создаются статики, т.к. они позволяют обойти ограничение что обычные поды управляются апи сервером. Kubelet запускает их из локальных манифестов на ноде. Kubelet следит за директорией с манифестами и обеспечивает высокую доступность ядра кластера. 

* Кто запускает kube-apiserver после reboot control-plane node?

После перезагрузки ноды kube-apiserver запускает kubelet.
процесс запуска: 1 - OS 2 - runtime 3 - kubelet 4 - чтение манифестов 5 - запуск контейнеров

* Чем static pod отличается от pod, созданного Deployment?

Static Pod: Создается локально. Kubelet читает YAML-файл из папки /etc/kubernetes/manifests и запускает контейнер. API-сервер для этого не нужен.
Pod от Deployment: Создается контроллером (DeploymentController -> ReplicaSetController). Запрос идет в API-сервер, записывается в etcd, а планировщик (scheduler) решает, на какую ноду его поместить

* Где kubeadm хранит manifests control plane?

/etc/kubernetes/manifests/

* Где лежат основные certificates cluster?

/etc/kubernetes/pki/

* Что такое kubeconfig и почему он нужен не только для kubectl?

это конфигурационный файл (обычно в формате YAML), который содержит параметры подключения к кластеру Kubernetes. Он содержит адреса API-серверов, сертификаты безопасности и токены для аутентификации.

* Как kubelet связан с containerd?

через cri

В этой связке kubelet выступает в роли «руководителя», который решает, что нужно сделать, а containerd — в роли «исполнителя», который знает, как управлять контейнерами на уровне операционной системы.

* Чем отличается вывод kubectl get pods от crictl ps?

kubectl get pods смотрит на кластер через api Kubernetes
crictl смотрит на конкретную ноду на уровне ОС этой ноды 

* Что произойдет, если kubelet на control-plane node остановится?

запущенные на этой ноде компоненты (API-сервер, etcd и др.) продолжат работать, но потеряют управление.

* Почему private keys нельзя публиковать в lab journal?

это условие для успешной сдачи это лабораторной работы
вообще приватные ключи должны лежать там, где они сгенерированы для безопасной безопасности, чтобы никто не смог подделать цифровой след пользователя


### Статус

done