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

## Lab 05 - CNI bootstrap with Calico

### Дата

2006-06-03

### Цель

Установить CNI plugin в kubeadm cluster

### Что было сделано

- Проверка состояния до установки CNI
- Подготовка каталога для Calico manifests
- Скачивание Calico manifests
- Проверка Pod CIDR в Calico custom resources
- Установка Calico CRDs и Tigera Operator
- Применение Calico custom resources
- Проверка nodes после установки CNI 
- Проверка CoreDNS
- Проверка container runtime NetworkReady
- Smoke test: Pod-to-Pod networking
- Очистка smoke test ресурсов

### Команды

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# kubectl get nodes -o wide
NAME          STATUS     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
deep-cp-01    NotReady   control-plane   44h   v1.36.1   192.168.100.50   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-01   NotReady   <none>          43h   v1.36.1   192.168.100.51   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-02   NotReady   <none>          43h   v1.36.1   192.168.100.52   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
root@deep-cp-01:/home/goblin# kubectl get pods -A -o wide
NAMESPACE     NAME                                 READY   STATUS    RESTARTS      AGE   IP               NODE          NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-mslt6             0/1     Pending   0             44h   <none>           <none>        <none>           <none>
kube-system   coredns-589f44dc88-sd4kh             0/1     Pending   0             44h   <none>           <none>        <none>           <none>
kube-system   etcd-deep-cp-01                      1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-apiserver-deep-cp-01            1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-controller-manager-deep-cp-01   1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-proxy-94t92                     1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-system   kube-proxy-gcrzz                     1/1     Running   1 (18m ago)   43h   192.168.100.52   deep-wrk-02   <none>           <none>
kube-system   kube-proxy-p7dff                     1/1     Running   1 (19m ago)   43h   192.168.100.51   deep-wrk-01   <none>           <none>
kube-system   kube-scheduler-deep-cp-01            1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
root@deep-cp-01:/home/goblin# kubectl -n kube-system get pods -o wide
NAME                                 READY   STATUS    RESTARTS      AGE   IP               NODE          NOMINATED NODE   READINESS GATES
coredns-589f44dc88-mslt6             0/1     Pending   0             44h   <none>           <none>        <none>           <none>
coredns-589f44dc88-sd4kh             0/1     Pending   0             44h   <none>           <none>        <none>           <none>
etcd-deep-cp-01                      1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-apiserver-deep-cp-01            1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-controller-manager-deep-cp-01   1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-94t92                     1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-gcrzz                     1/1     Running   1 (19m ago)   43h   192.168.100.52   deep-wrk-02   <none>           <none>
kube-proxy-p7dff                     1/1     Running   1 (19m ago)   43h   192.168.100.51   deep-wrk-01   <none>           <none>
kube-scheduler-deep-cp-01            1/1     Running   1 (19m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>

root@deep-cp-01:/home/goblin# crictl info | jq '.status.conditions'
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

root@deep-cp-01:/home/goblin# mkdir cni && cd -
/home/goblin/cni
root@deep-cp-01:/home/goblin/cni# curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/v1_crd_projectcalico_org.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 2975k  100 2975k    0     0  3937k      0 --:--:-- --:--:-- --:--:-- 3935k
root@deep-cp-01:/home/goblin/cni# curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/tigera-operator.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 19285  100 19285    0     0  58288      0 --:--:-- --:--:-- --:--:-- 58262
root@deep-cp-01:/home/goblin/cni# curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/custom-resources.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1046  100  1046    0     0   3194      0 --:--:-- --:--:-- --:--:--  3189

root@deep-cp-01:/home/goblin/cni# grep -n "cidr\|blockSize\|encapsulation\|natOutgoing\|nodeSelector" custom-resources.yaml || true
12:        blockSize: 26
13:        cidr: 192.168.0.0/16
14:        encapsulation: VXLANCrossSubnet
15:        natOutgoing: Enabled
16:        nodeSelector: all()

#====================================================================================================

root@deep-cp-01:/home/goblin/cni# kubectl apply -f v1_crd_projectcalico_org.yaml
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/gatewayapis.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/goldmanes.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/istios.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/managementclusterconnections.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/whiskers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
Warning: unrecognized format "cidr"
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/stagedglobalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/stagedkubernetesnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/stagednetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/tiers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusternetworkpolicies.policy.networking.k8s.io created
The CustomResourceDefinition "installations.operator.tigera.io" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes

root@deep-cp-01:/home/goblin/cni# kubectl apply -f tigera-operator.yaml
namespace/tigera-operator created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator-secrets created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
rolebinding.rbac.authorization.k8s.io/tigera-operator-secrets created
deployment.apps/tigera-operator created

root@deep-cp-01:/home/goblin/cni# kubectl get ns
NAME              STATUS   AGE
default           Active   44h
kube-node-lease   Active   44h
kube-public       Active   44h
kube-system       Active   44h
tigera-operator   Active   31s

root@deep-cp-01:/home/goblin/cni# kubectl get pods -A | grep -E 'tigera|calico' || true
tigera-operator   tigera-operator-85dbff4478-fjdxd     1/1     Running   0             50s

root@deep-cp-01:/home/goblin/cni# kubectl get crd | grep -E 'projectcalico|operator.tigera' || true
apiservers.operator.tigera.io                           2026-06-03T14:08:13Z
bgpconfigurations.crd.projectcalico.org                 2026-06-03T14:08:13Z
bgpfilters.crd.projectcalico.org                        2026-06-03T14:08:13Z
bgppeers.crd.projectcalico.org                          2026-06-03T14:08:13Z
blockaffinities.crd.projectcalico.org                   2026-06-03T14:08:13Z
caliconodestatuses.crd.projectcalico.org                2026-06-03T14:08:13Z
clusterinformations.crd.projectcalico.org               2026-06-03T14:08:13Z
felixconfigurations.crd.projectcalico.org               2026-06-03T14:08:13Z
gatewayapis.operator.tigera.io                          2026-06-03T14:08:13Z
globalnetworkpolicies.crd.projectcalico.org             2026-06-03T14:08:14Z
globalnetworksets.crd.projectcalico.org                 2026-06-03T14:08:14Z
goldmanes.operator.tigera.io                            2026-06-03T14:08:13Z
hostendpoints.crd.projectcalico.org                     2026-06-03T14:08:14Z
imagesets.operator.tigera.io                            2026-06-03T14:08:13Z
installations.operator.tigera.io                        2026-06-03T14:09:48Z
ipamblocks.crd.projectcalico.org                        2026-06-03T14:08:14Z
ipamconfigs.crd.projectcalico.org                       2026-06-03T14:08:14Z
ipamhandles.crd.projectcalico.org                       2026-06-03T14:08:14Z
ippools.crd.projectcalico.org                           2026-06-03T14:08:14Z
ipreservations.crd.projectcalico.org                    2026-06-03T14:08:14Z
istios.operator.tigera.io                               2026-06-03T14:08:13Z
kubecontrollersconfigurations.crd.projectcalico.org     2026-06-03T14:08:14Z
managementclusterconnections.operator.tigera.io         2026-06-03T14:08:13Z
networkpolicies.crd.projectcalico.org                   2026-06-03T14:08:14Z
networksets.crd.projectcalico.org                       2026-06-03T14:08:14Z
stagedglobalnetworkpolicies.crd.projectcalico.org       2026-06-03T14:08:14Z
stagedkubernetesnetworkpolicies.crd.projectcalico.org   2026-06-03T14:08:14Z
stagednetworkpolicies.crd.projectcalico.org             2026-06-03T14:08:14Z
tiers.crd.projectcalico.org                             2026-06-03T14:08:14Z
tigerastatuses.operator.tigera.io                       2026-06-03T14:08:13Z
whiskers.operator.tigera.io                             2026-06-03T14:08:13Z

root@deep-cp-01:/home/goblin/cni# kubectl apply -f custom-resources.yaml
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
goldmane.operator.tigera.io/default created
whisker.operator.tigera.io/default created

#====================================================================================================

root@deep-cp-01:/home/goblin/cni# kubectl -n calico-system get pods -o wide
NAME                                       READY   STATUS              RESTARTS   AGE     IP               NODE          NOMINATED NODE   READINESS GATES
calico-apiserver-9c45c8f8f-5fgwk           1/1     Running             0          3m56s   192.168.62.7     deep-cp-01    <none>           <none>
calico-apiserver-9c45c8f8f-zq7sh           1/1     Running             0          3m56s   192.168.62.4     deep-cp-01    <none>           <none>
calico-kube-controllers-5b8d487f58-6jb5z   1/1     Running             0          3m55s   192.168.62.8     deep-cp-01    <none>           <none>
calico-node-fcxdj                          1/1     Running             0          3m55s   192.168.100.52   deep-wrk-02   <none>           <none>
calico-node-k7rg2                          1/1     Running             0          3m55s   192.168.100.51   deep-wrk-01   <none>           <none>
calico-node-p4sjq                          1/1     Running             0          3m55s   192.168.100.50   deep-cp-01    <none>           <none>
calico-typha-5bb76474df-6h8tk              1/1     Running             0          3m56s   192.168.100.52   deep-wrk-02   <none>           <none>
calico-typha-5bb76474df-9ch4x              1/1     Running             0          3m50s   192.168.100.51   deep-wrk-01   <none>           <none>
csi-node-driver-6xctm                      0/2     ContainerCreating   0          3m55s   <none>           deep-wrk-01   <none>           <none>
csi-node-driver-dc2pq                      2/2     Running             0          3m55s   192.168.19.129   deep-wrk-02   <none>           <none>
csi-node-driver-dwsqc                      2/2     Running             0          3m55s   192.168.62.1     deep-cp-01    <none>           <none>
goldmane-6885dcb7d-hgtvj                   1/1     Running             0          3m56s   192.168.62.2     deep-cp-01    <none>           <none>
whisker-5cb7d57d4c-bk5cm                   2/2     Running             0          2m59s   192.168.62.3     deep-cp-01    <none>           <none>
root@deep-cp-01:/home/goblin/cni# kubectl -n kube-system get pods -o wide
NAME                                 READY   STATUS    RESTARTS      AGE   IP               NODE          NOMINATED NODE   READINESS GATES
coredns-589f44dc88-mslt6             1/1     Running   0             44h   192.168.62.5     deep-cp-01    <none>           <none>
coredns-589f44dc88-sd4kh             1/1     Running   0             44h   192.168.62.6     deep-cp-01    <none>           <none>
etcd-deep-cp-01                      1/1     Running   1 (41m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-apiserver-deep-cp-01            1/1     Running   1 (41m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-controller-manager-deep-cp-01   1/1     Running   1 (41m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-94t92                     1/1     Running   1 (41m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-gcrzz                     1/1     Running   1 (40m ago)   44h   192.168.100.52   deep-wrk-02   <none>           <none>
kube-proxy-p7dff                     1/1     Running   1 (40m ago)   44h   192.168.100.51   deep-wrk-01   <none>           <none>
kube-scheduler-deep-cp-01            1/1     Running   1 (41m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>

root@deep-cp-01:/home/goblin/cni# kubectl -n kube-system get deployment coredns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           44h

#====================================================================================================

root@deep-cp-01:/home/goblin/cni# kubectl create namespace kdl-smoke
namespace/kdl-smoke created
root@deep-cp-01:/home/goblin/cni# kubectl -n kdl-smoke run net-a --image=busybox:1.36 --restart=Never -- sleep 3600
pod/net-a created
root@deep-cp-01:/home/goblin/cni# kubectl -n kdl-smoke run net-b --image=busybox:1.36 --restart=Never -- sleep 3600
pod/net-b created

root@deep-cp-01:/home/goblin/cni# kubectl -n kdl-smoke get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP                NODE          NOMINATED NODE   READINESS GATES
net-a   1/1     Running   0          50s   192.168.19.130    deep-wrk-02   <none>           <none>
net-b   1/1     Running   0          44s   192.168.158.194   deep-wrk-01   <none>           <none>

root@deep-cp-01:/home/goblin/cni# NET_B_IP=$(kubectl -n kdl-smoke get pod net-b -o jsonpath='{.status.podIP}')
echo "$NET_B_IP"
192.168.158.194
root@deep-cp-01:/home/goblin/cni# kubectl -n kdl-smoke exec net-a -- ping -c 3 "$NET_B_IP"
PING 192.168.158.194 (192.168.158.194): 56 data bytes
64 bytes from 192.168.158.194: seq=0 ttl=62 time=0.536 ms
64 bytes from 192.168.158.194: seq=1 ttl=62 time=0.635 ms
64 bytes from 192.168.158.194: seq=2 ttl=62 time=1.409 ms

--- 192.168.158.194 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.536/0.860/1.409 ms

root@deep-cp-01:/home/goblin/cni# kubectl -n kdl-smoke exec net-a -- nslookup kubernetes.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1

root@deep-cp-01:/home/goblin/cni# kubectl delete namespace kdl-smoke
namespace "kdl-smoke" deleted
root@deep-cp-01:/home/goblin/cni# kubectl get namespace kdl-smoke 2>/dev/null || true

```

### Ключевые выводы команд

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin/cni# kubectl get installation.operator.tigera.io
NAME      AGE
default   78s
root@deep-cp-01:/home/goblin/cni# kubectl get ippool.crd.projectcalico.org -A || true
NAME                  AGE
default-ipv4-ippool   92s

root@deep-cp-01:/home/goblin/cni# kubectl get tigerastatus
NAME        AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
apiserver   True        False         False      68s     All objects available
calico      True        False         False      59s     All objects available
goldmane    True        False         False      53s     All objects available
ippools     True        False         False      2m49s   All objects available
tiers       True        False         False      63s     All objects available
whisker     True        False         False      53s     All objects available

root@deep-cp-01:/home/goblin/cni# kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
deep-cp-01    Ready    control-plane   44h   v1.36.1   192.168.100.50   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-01   Ready    <none>          44h   v1.36.1   192.168.100.51   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
deep-wrk-02   Ready    <none>          44h   v1.36.1   192.168.100.52   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1

root@deep-cp-01:/home/goblin/cni# kubectl -n calico-system get pods -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP                NODE          NOMINATED NODE   READINESS GATES
calico-apiserver-9c45c8f8f-5fgwk           1/1     Running   0          26m   192.168.62.7      deep-cp-01    <none>           <none>
calico-apiserver-9c45c8f8f-zq7sh           1/1     Running   0          26m   192.168.62.4      deep-cp-01    <none>           <none>
calico-kube-controllers-5b8d487f58-6jb5z   1/1     Running   0          26m   192.168.62.8      deep-cp-01    <none>           <none>
calico-node-fcxdj                          1/1     Running   0          26m   192.168.100.52    deep-wrk-02   <none>           <none>
calico-node-k7rg2                          1/1     Running   0          26m   192.168.100.51    deep-wrk-01   <none>           <none>
calico-node-p4sjq                          1/1     Running   0          26m   192.168.100.50    deep-cp-01    <none>           <none>
calico-typha-5bb76474df-6h8tk              1/1     Running   0          26m   192.168.100.52    deep-wrk-02   <none>           <none>
calico-typha-5bb76474df-9ch4x              1/1     Running   0          26m   192.168.100.51    deep-wrk-01   <none>           <none>
csi-node-driver-6xctm                      2/2     Running   0          26m   192.168.158.193   deep-wrk-01   <none>           <none>
csi-node-driver-dc2pq                      2/2     Running   0          26m   192.168.19.129    deep-wrk-02   <none>           <none>
csi-node-driver-dwsqc                      2/2     Running   0          26m   192.168.62.1      deep-cp-01    <none>           <none>
goldmane-6885dcb7d-hgtvj                   1/1     Running   0          26m   192.168.62.2      deep-cp-01    <none>           <none>
whisker-5cb7d57d4c-bk5cm                   2/2     Running   0          25m   192.168.62.3      deep-cp-01    <none>           <none>

root@deep-cp-01:/home/goblin/cni# kubectl -n kube-system get pods -o wide
NAME                                 READY   STATUS    RESTARTS      AGE   IP               NODE          NOMINATED NODE   READINESS GATES
coredns-589f44dc88-mslt6             1/1     Running   0             44h   192.168.62.5     deep-cp-01    <none>           <none>
coredns-589f44dc88-sd4kh             1/1     Running   0             44h   192.168.62.6     deep-cp-01    <none>           <none>
etcd-deep-cp-01                      1/1     Running   1 (63m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-apiserver-deep-cp-01            1/1     Running   1 (63m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-controller-manager-deep-cp-01   1/1     Running   1 (63m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-94t92                     1/1     Running   1 (63m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>
kube-proxy-gcrzz                     1/1     Running   1 (63m ago)   44h   192.168.100.52   deep-wrk-02   <none>           <none>
kube-proxy-p7dff                     1/1     Running   1 (63m ago)   44h   192.168.100.51   deep-wrk-01   <none>           <none>
kube-scheduler-deep-cp-01            1/1     Running   1 (63m ago)   44h   192.168.100.50   deep-cp-01    <none>           <none>

root@deep-cp-01:/home/goblin/cni# crictl info | jq '.status.conditions'
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
root@deep-cp-01:/home/goblin/cni# ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  3 14:12 10-calico.conflist
-rw------- 1 root root 2801 Jun  3 14:12 calico-kubeconfig
```

**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# sudo crictl info | jq '.status.conditions'
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
root@deep-wrk-01:/home/goblin# ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  3 14:12 10-calico.conflist
-rw------- 1 root root 2801 Jun  3 14:13 calico-kubeconfig
```

**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# sudo crictl info | jq '.status.conditions'
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
root@deep-wrk-02:/home/goblin# ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun  3 14:13 10-calico.conflist
-rw------- 1 root root 2801 Jun  3 14:13 calico-kubeconfig
```

### Что стало понятнее

- Показалось интересным применение оператора tigera для разворачивания calico, ранее не применял операторов для такой операции. У данного оператора есть helm chart, соответственно, можно использовать с ArgoCD.


### Статус

Done

