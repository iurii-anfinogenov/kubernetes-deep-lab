# Lab 05 - CNI bootstrap with Calico

### Дата

2026-06-05

### Цель

Установить CNI plugin в kubeadm cluster 

## вывод приложения

```
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   5h4m    v1.36.1   192.168.1.40   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
worker-1          Ready    <none>          4h51m   v1.36.1   192.168.1.43   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
worker-2          Ready    <none>          4h51m   v1.36.1   192.168.1.46   <none>        Ubuntu 24.04.4 LTS   6.8.0-117-generic (amd64)   containerd://2.2.1
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get pods -A -o wide
NAMESPACE         NAME                                      READY   STATUS    RESTARTS      AGE     IP                NODE              NOMINATED NODE   READINESS GATES
calico-system     calico-apiserver-6465b79596-55xmq         1/1     Running   0             43m     192.168.226.69    worker-1          <none>           <none>
calico-system     calico-apiserver-6465b79596-px4q8         1/1     Running   0             43m     192.168.133.195   worker-2          <none>           <none>
calico-system     calico-kube-controllers-c67bd4bdd-fszg6   1/1     Running   0             43m     192.168.226.67    worker-1          <none>           <none>
calico-system     calico-node-cjhgd                         1/1     Running   0             43m     192.168.1.40      control-plane-1   <none>           <none>
calico-system     calico-node-jmj5r                         1/1     Running   2 (36m ago)   43m     192.168.1.46      worker-2          <none>           <none>
calico-system     calico-node-qn88q                         1/1     Running   0             43m     192.168.1.43      worker-1          <none>           <none>
calico-system     calico-typha-7c9696784c-49xff             1/1     Running   0             43m     192.168.1.43      worker-1          <none>           <none>
calico-system     calico-typha-7c9696784c-5bgjp             1/1     Running   0             43m     192.168.1.46      worker-2          <none>           <none>
calico-system     csi-node-driver-bcvhd                     2/2     Running   0             43m     192.168.247.129   control-plane-1   <none>           <none>
calico-system     csi-node-driver-pnrcf                     2/2     Running   0             43m     192.168.226.65    worker-1          <none>           <none>
calico-system     csi-node-driver-t2w6b                     2/2     Running   0             43m     192.168.133.193   worker-2          <none>           <none>
calico-system     goldmane-6885dcb7d-z4ppt                  1/1     Running   0             43m     192.168.226.68    worker-1          <none>           <none>
calico-system     whisker-75fcc899c7-5njbv                  2/2     Running   0             28m     192.168.133.197   worker-2          <none>           <none>
kube-system       coredns-589f44dc88-7hmjk                  1/1     Running   0             5h3m    192.168.226.66    worker-1          <none>           <none>
kube-system       coredns-589f44dc88-cmbt2                  1/1     Running   0             5h3m    192.168.133.194   worker-2          <none>           <none>
kube-system       etcd-control-plane-1                      1/1     Running   0             5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-system       kube-apiserver-control-plane-1            1/1     Running   1 (39m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-system       kube-controller-manager-control-plane-1   1/1     Running   3 (38m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-system       kube-proxy-4kvmt                          1/1     Running   0             5h3m    192.168.1.40      control-plane-1   <none>           <none>
kube-system       kube-proxy-62wxd                          1/1     Running   0             4h51m   192.168.1.43      worker-1          <none>           <none>
kube-system       kube-proxy-hgk4d                          1/1     Running   0             4h51m   192.168.1.46      worker-2          <none>           <none>
kube-system       kube-scheduler-control-plane-1            1/1     Running   3 (37m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
tigera-operator   tigera-operator-85dbff4478-cjnpc          1/1     Running   4 (37m ago)   44m     192.168.1.46      worker-2          <none>           <none>
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get tigerastatus 2>/dev/null || true
NAME        AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
apiserver   True        False         False      30m     All objects available
calico      True        False         False      31m     All objects available
goldmane    True        False         False      30m     All objects available
ippools     True        False         False      43m     All objects available
tiers       True        False         False      30m     All objects available
whisker     True        False         False      28m     All objects available
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n calico-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS      AGE   IP                NODE              NOMINATED NODE   READINESS GATES
calico-apiserver-6465b79596-55xmq         1/1     Running   0             43m   192.168.226.69    worker-1          <none>           <none>
calico-apiserver-6465b79596-px4q8         1/1     Running   0             43m   192.168.133.195   worker-2          <none>           <none>
calico-kube-controllers-c67bd4bdd-fszg6   1/1     Running   0             43m   192.168.226.67    worker-1          <none>           <none>
calico-node-cjhgd                         1/1     Running   0             43m   192.168.1.40      control-plane-1   <none>           <none>
calico-node-jmj5r                         1/1     Running   2 (37m ago)   43m   192.168.1.46      worker-2          <none>           <none>
calico-node-qn88q                         1/1     Running   0             43m   192.168.1.43      worker-1          <none>           <none>
calico-typha-7c9696784c-49xff             1/1     Running   0             43m   192.168.1.43      worker-1          <none>           <none>
calico-typha-7c9696784c-5bgjp             1/1     Running   0             43m   192.168.1.46      worker-2          <none>           <none>
csi-node-driver-bcvhd                     2/2     Running   0             43m   192.168.247.129   control-plane-1   <none>           <none>
csi-node-driver-pnrcf                     2/2     Running   0             43m   192.168.226.65    worker-1          <none>           <none>
csi-node-driver-t2w6b                     2/2     Running   0             43m   192.168.133.193   worker-2          <none>           <none>
goldmane-6885dcb7d-z4ppt                  1/1     Running   0             43m   192.168.226.68    worker-1          <none>           <none>
whisker-75fcc899c7-5njbv                  2/2     Running   0             28m   192.168.133.197   worker-2          <none>           <none>
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS      AGE     IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-7hmjk                  1/1     Running   0             5h4m    192.168.226.66    worker-1          <none>           <none>
coredns-589f44dc88-cmbt2                  1/1     Running   0             5h4m    192.168.133.194   worker-2          <none>           <none>
etcd-control-plane-1                      1/1     Running   0             5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   1 (39m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   3 (38m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-proxy-4kvmt                          1/1     Running   0             5h4m    192.168.1.40      control-plane-1   <none>           <none>
kube-proxy-62wxd                          1/1     Running   0             4h52m   192.168.1.43      worker-1          <none>           <none>
kube-proxy-hgk4d                          1/1     Running   0             4h51m   192.168.1.46      worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   3 (37m ago)   5h4m    192.168.1.40      control-plane-1   <none>           <none>
```


``` cp
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ hostname
control-plane-1
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ sudo crictl info | jq '.status.conditions'
[
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "RuntimeReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "NetworkReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "ContainerdHasNoDeprecationWarnings"
  }
]
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  6 19:56 10-calico.conflist
-rw------- 1 root root 2801 Jun  6 19:56 calico-kubeconfig
```

```
root@worker-1:~# hostname
worker-1
root@worker-1:~# sudo crictl info | jq '.status.conditions'
[
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "RuntimeReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "NetworkReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "ContainerdHasNoDeprecationWarnings"
  }
]
root@worker-1:~# sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  6 19:56 10-calico.conflist
-rw------- 1 root root 2801 Jun  6 19:56 calico-kubeconfig
```

```
root@worker-2:~# hostname
worker-2
root@worker-2:~# sudo crictl info | jq '.status.conditions'
[
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "RuntimeReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "NetworkReady"
  },
  {
    "message": "",
    "reason": "",
    "status": true,
    "type": "ContainerdHasNoDeprecationWarnings"
  }
]
root@worker-2:~# sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  6 19:55 10-calico.conflist
-rw------- 1 root root 2801 Jun  6 19:56 calico-kubeconfig
```

``` smoke test
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ NET_B_IP=$(kubectl -n kdl-smoke get pod net-b -o jsonpath='{.status.podIP}')tatus.podIP}')
echo "$NET_B_IP"
192.168.226.71
ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n kdl-smoke exec net-a -- ping -c 3 "$NET_B_IP"
PING 192.168.226.71 (192.168.226.71): 56 data bytes
64 bytes from 192.168.226.71: seq=0 ttl=63 time=0.301 ms
64 bytes from 192.168.226.71: seq=1 ttl=63 time=0.114 ms
64 bytes from 192.168.226.71: seq=2 ttl=63 time=0.122 ms

--- 192.168.226.71 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.114/0.179/0.301 ms

ae@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n kdl-smoke exec net-a -- nslookup kubernetes.default.svc.cluster.local
Server:		10.96.0.10
Address:	10.96.0.10:53

Name:	kubernetes.default.svc.cluster.local
Address: 10.96.0.1

```
### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| CoreDNS не запускался | на каждой ноде не было образа |  | вручную спулил образы на каждую ноду |


### Что стало понятнее



### Статус

done