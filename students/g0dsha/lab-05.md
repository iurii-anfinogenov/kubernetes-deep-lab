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

## Lab 05 - CNI bootstrap with Calico

### Дата

2026-06-11

### Цель

Установить CNI plugin в kubeadm cluster

### Что было сделано

- Проверка состояния до установки CNI
- Подготовка каталога для Calico manifests
- Скачивание Calico manifests
- Проверка Pod CIDR в Calico custom resources
- Dry-run применения Calico manifests
- Установка Calico CRDs и Tigera Operator
- Применение Calico custom resources
- Проверка после установки CNI
- Smoke tests
- Финальная проверка

### Команды
Все команды этой лабораторной выполняются только на control-plane node, если нет уточнения

#### Проверка состояния до установки CNI
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   NotReady   control-plane   17h   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          NotReady   <none>          17h   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          NotReady   <none>          17h   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
user@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE   IP               NODE              NOMINATED NODE   READINESS GATES
kube-system   coredns-589f44dc88-fwct6                  0/1     Pending   0          17h   <none>           <none>            <none>           <none>
kube-system   coredns-589f44dc88-zx2qb                  0/1     Pending   0          17h   <none>           <none>            <none>           <none>
kube-system   etcd-control-plane-1                      1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-apiserver-control-plane-1            1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-controller-manager-control-plane-1   1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-hdvtm                          1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-system   kube-proxy-jc7lk                          1/1     Running   0          17h   192.168.88.134   worker-1          <none>           <none>
kube-system   kube-proxy-spstk                          1/1     Running   0          17h   192.168.88.243   worker-2          <none>           <none>
kube-system   kube-scheduler-control-plane-1            1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
user@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP               NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-fwct6                  0/1     Pending   0          17h   <none>           <none>            <none>           <none>
coredns-589f44dc88-zx2qb                  0/1     Pending   0          17h   <none>           <none>            <none>           <none>
etcd-control-plane-1                      1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-proxy-hdvtm                          1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
kube-proxy-jc7lk                          1/1     Running   0          17h   192.168.88.134   worker-1          <none>           <none>
kube-proxy-spstk                          1/1     Running   0          17h   192.168.88.243   worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   0          17h   192.168.88.184   control-plane-1   <none>           <none>
user@control-plane-1:~$ sudo crictl info | jq '.status.conditions'
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
#### Подготовка каталога для Calico manifests
```
user@control-plane-1:~$ mkdir -p ~/kdl-manifests/calico-v3.32.0
user@control-plane-1:~$ cd ~/kdl-manifests/calico-v3.32.0
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ 
```
#### Скачивание Calico manifests
```
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/v1_crd_projectcalico_org.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 2975k  100 2975k    0     0  1695k      0  0:00:01  0:00:01 --:--:-- 1698k
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/tigera-operator.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 19285  100 19285    0     0  29434      0 --:--:-- --:--:-- --:--:-- 29669
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/custom-resources.yaml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1046  100  1046    0     0   1700      0 --:--:-- --:--:-- --:--:--  1706
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ ls -lh
total 3.0M
-rw-rw-r-- 1 user user 1.1K Jun 11 15:27 custom-resources.yaml
-rw-rw-r-- 1 user user  19K Jun 11 15:27 tigera-operator.yaml
-rw-rw-r-- 1 user user 3.0M Jun 11 15:26 v1_crd_projectcalico_org.yaml
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ wc -l *.yaml
     39 custom-resources.yaml
    765 tigera-operator.yaml
  40126 v1_crd_projectcalico_org.yaml
  40930 total
```
#### Проверка Pod CIDR в Calico custom resources
```
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ grep cidr custom-resources.yaml
        cidr: 192.168.0.0/16
```
#### Dry-run применения Calico manifests
```
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply --dry-run=server -f v1_crd_projectcalico_org.yaml
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/gatewayapis.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/goldmanes.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/istios.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/managementclusterconnections.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/whiskers.operator.tigera.io created (server dry run)
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created (server dry run)
Warning: unrecognized format "cidr"
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/stagedglobalnetworkpolicies.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/stagedkubernetesnetworkpolicies.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/stagednetworkpolicies.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/tiers.crd.projectcalico.org created (server dry run)
customresourcedefinition.apiextensions.k8s.io/clusternetworkpolicies.policy.networking.k8s.io created (server dry run)
The CustomResourceDefinition "installations.operator.tigera.io" is invalid: metadata.annotations: Too long: may not be more than 262144 bytes

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply --dry-run=server -f tigera-operator.yaml
namespace/tigera-operator created (server dry run)
clusterrole.rbac.authorization.k8s.io/tigera-operator-secrets created (server dry run)
clusterrole.rbac.authorization.k8s.io/tigera-operator created (server dry run)
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created (server dry run)
Error from server (NotFound): error when creating "tigera-operator.yaml": namespaces "tigera-operator" not found
Error from server (NotFound): error when creating "tigera-operator.yaml": namespaces "tigera-operator" not found
Error from server (NotFound): error when creating "tigera-operator.yaml": namespaces "tigera-operator" not found
```
#### Установка Calico CRDs и Tigera Operator
```
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply -f v1_crd_projectcalico_org.yaml
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

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply -f tigera-operator.yaml
namespace/tigera-operator created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator-secrets created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
rolebinding.rbac.authorization.k8s.io/tigera-operator-secrets created
deployment.apps/tigera-operator created

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get ns
NAME              STATUS   AGE
default           Active   18h
kube-node-lease   Active   18h
kube-public       Active   18h
kube-system       Active   18h
tigera-operator   Active   2m19s

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get pods -A | grep -E 'tigera|calico' || true
tigera-operator   tigera-operator-85dbff4478-ntdvh          1/1     Running   0          4m28s

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get crd | grep -E 'projectcalico|operator.tigera' || true
apiservers.operator.tigera.io                           2026-06-11T12:55:44Z
bgpconfigurations.crd.projectcalico.org                 2026-06-11T12:55:45Z
bgpfilters.crd.projectcalico.org                        2026-06-11T12:55:45Z
bgppeers.crd.projectcalico.org                          2026-06-11T12:55:45Z
blockaffinities.crd.projectcalico.org                   2026-06-11T12:55:45Z
caliconodestatuses.crd.projectcalico.org                2026-06-11T12:55:45Z
clusterinformations.crd.projectcalico.org               2026-06-11T12:55:45Z
felixconfigurations.crd.projectcalico.org               2026-06-11T12:55:45Z
gatewayapis.operator.tigera.io                          2026-06-11T12:55:44Z
globalnetworkpolicies.crd.projectcalico.org             2026-06-11T12:55:45Z
globalnetworksets.crd.projectcalico.org                 2026-06-11T12:55:45Z
goldmanes.operator.tigera.io                            2026-06-11T12:55:44Z
hostendpoints.crd.projectcalico.org                     2026-06-11T12:55:45Z
imagesets.operator.tigera.io                            2026-06-11T12:55:44Z
installations.operator.tigera.io                        2026-06-11T12:56:02Z
ipamblocks.crd.projectcalico.org                        2026-06-11T12:55:46Z
ipamconfigs.crd.projectcalico.org                       2026-06-11T12:55:46Z
ipamhandles.crd.projectcalico.org                       2026-06-11T12:55:46Z
ippools.crd.projectcalico.org                           2026-06-11T12:55:46Z
ipreservations.crd.projectcalico.org                    2026-06-11T12:55:46Z
istios.operator.tigera.io                               2026-06-11T12:55:44Z
kubecontrollersconfigurations.crd.projectcalico.org     2026-06-11T12:55:46Z
managementclusterconnections.operator.tigera.io         2026-06-11T12:55:45Z
networkpolicies.crd.projectcalico.org                   2026-06-11T12:55:46Z
networksets.crd.projectcalico.org                       2026-06-11T12:55:46Z
stagedglobalnetworkpolicies.crd.projectcalico.org       2026-06-11T12:55:46Z
stagedkubernetesnetworkpolicies.crd.projectcalico.org   2026-06-11T12:55:46Z
stagednetworkpolicies.crd.projectcalico.org             2026-06-11T12:55:46Z
tiers.crd.projectcalico.org                             2026-06-11T12:55:46Z
tigerastatuses.operator.tigera.io                       2026-06-11T12:55:45Z
whiskers.operator.tigera.io                             2026-06-11T12:55:45Z

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n tigera-operator get pods -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP               NODE       NOMINATED NODE   READINESS GATES
tigera-operator-85dbff4478-ntdvh   1/1     Running   0          4m57s   192.168.88.243   worker-2   <none>           <none>

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n tigera-operator wait --for=condition=Available deployment/tigera-operator --timeout=180s
deployment.apps/tigera-operator condition met
```
#### Применение Calico custom resources
```
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply --dry-run=server -f custom-resources.yaml
installation.operator.tigera.io/default created (server dry run)
apiserver.operator.tigera.io/default created (server dry run)
goldmane.operator.tigera.io/default created (server dry run)
whisker.operator.tigera.io/default created (server dry run)
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl apply -f custom-resources.yaml
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
goldmane.operator.tigera.io/default created
whisker.operator.tigera.io/default created
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get installation.operator.tigera.io
NAME      AGE
default   10s
user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get ippool.crd.projectcalico.org -A || true
NAME                  AGE
default-ipv4-ippool   20s

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get pods -A -o wide | grep -E 'calico|tigera'
calico-system     calico-apiserver-9dcbdf8f6-rtt4g           1/1     Running   0          3m2s    192.168.247.135   control-plane-1   <none>           <none>
calico-system     calico-apiserver-9dcbdf8f6-v4465           1/1     Running   0          3m2s    192.168.247.134   control-plane-1   <none>           <none>
calico-system     calico-kube-controllers-6c677558b8-8hb9n   1/1     Running   0          3m1s    192.168.247.131   control-plane-1   <none>           <none>
calico-system     calico-node-2zpr6                          1/1     Running   0          3m1s    192.168.88.184    control-plane-1   <none>           <none>
calico-system     calico-node-5kqm7                          1/1     Running   0          3m1s    192.168.88.134    worker-1          <none>           <none>
calico-system     calico-node-tc25r                          1/1     Running   0          3m1s    192.168.88.243    worker-2          <none>           <none>
calico-system     calico-typha-648595f9df-tg4fb              1/1     Running   0          2m56s   192.168.88.243    worker-2          <none>           <none>
calico-system     calico-typha-648595f9df-w2952              1/1     Running   0          3m2s    192.168.88.134    worker-1          <none>           <none>
calico-system     csi-node-driver-ccj4j                      2/2     Running   0          3m1s    192.168.247.129   control-plane-1   <none>           <none>
calico-system     csi-node-driver-hj7cc                      2/2     Running   0          3m1s    192.168.226.65    worker-1          <none>           <none>
calico-system     csi-node-driver-r4n6x                      2/2     Running   0          3m1s    192.168.133.193   worker-2          <none>           <none>
calico-system     goldmane-6885dcb7d-ncwc8                   1/1     Running   0          3m2s    192.168.247.136   control-plane-1   <none>           <none>
calico-system     whisker-5674dc44cb-njzf9                   2/2     Running   0          2m13s   192.168.247.130   control-plane-1   <none>           <none>
tigera-operator   tigera-operator-85dbff4478-ntdvh           1/1     Running   0          11m     192.168.88.243    worker-2          <none>           <none>

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl get tigerastatus
NAME        AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
apiserver   True        False         False      112s    All objects available
calico      True        False         False      117s    All objects available
goldmane    True        False         False      82s     All objects available
ippools     True        False         False      3m27s   All objects available
tiers       True        False         False      107s    All objects available
whisker     True        False         False      97s     All objects available

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n calico-system get pods -o wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP                NODE              NOMINATED NODE   READINESS GATES
calico-apiserver-9dcbdf8f6-rtt4g           1/1     Running   0          4m27s   192.168.247.135   control-plane-1   <none>           <none>
calico-apiserver-9dcbdf8f6-v4465           1/1     Running   0          4m27s   192.168.247.134   control-plane-1   <none>           <none>
calico-kube-controllers-6c677558b8-8hb9n   1/1     Running   0          4m26s   192.168.247.131   control-plane-1   <none>           <none>
calico-node-2zpr6                          1/1     Running   0          4m26s   192.168.88.184    control-plane-1   <none>           <none>
calico-node-5kqm7                          1/1     Running   0          4m26s   192.168.88.134    worker-1          <none>           <none>
calico-node-tc25r                          1/1     Running   0          4m26s   192.168.88.243    worker-2          <none>           <none>
calico-typha-648595f9df-tg4fb              1/1     Running   0          4m21s   192.168.88.243    worker-2          <none>           <none>
calico-typha-648595f9df-w2952              1/1     Running   0          4m27s   192.168.88.134    worker-1          <none>           <none>
csi-node-driver-ccj4j                      2/2     Running   0          4m26s   192.168.247.129   control-plane-1   <none>           <none>
csi-node-driver-hj7cc                      2/2     Running   0          4m26s   192.168.226.65    worker-1          <none>           <none>
csi-node-driver-r4n6x                      2/2     Running   0          4m26s   192.168.133.193   worker-2          <none>           <none>
goldmane-6885dcb7d-ncwc8                   1/1     Running   0          4m27s   192.168.247.136   control-plane-1   <none>           <none>
whisker-5674dc44cb-njzf9                   2/2     Running   0          3m38s   192.168.247.130   control-plane-1   <none>           <none>

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n calico-system get daemonset
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
calico-node       3         3         3       3            3           kubernetes.io/os=linux   4m34s
csi-node-driver   3         3         3       3            3           kubernetes.io/os=linux   4m34s

user@control-plane-1:~/kdl-manifests/calico-v3.32.0$ kubectl -n calico-system get deployment
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
calico-apiserver          2/2     2            2           4m46s
calico-kube-controllers   1/1     1            1           4m45s
calico-typha              2/2     2            2           4m46s
goldmane                  1/1     1            1           4m46s
whisker                   1/1     1            1           4m46s
```
#### Проверка после установки CNI
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   18h   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          Ready    <none>          18h   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          Ready    <none>          18h   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1

user@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-fwct6                  1/1     Running   0          18h   192.168.247.132   control-plane-1   <none>           <none>
coredns-589f44dc88-zx2qb                  1/1     Running   0          18h   192.168.247.133   control-plane-1   <none>           <none>
etcd-control-plane-1                      1/1     Running   0          18h   192.168.88.184    control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   0          18h   192.168.88.184    control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   0          18h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-hdvtm                          1/1     Running   0          18h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-jc7lk                          1/1     Running   0          18h   192.168.88.134    worker-1          <none>           <none>
kube-proxy-spstk                          1/1     Running   0          18h   192.168.88.243    worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   0          18h   192.168.88.184    control-plane-1   <none>           <none>

user@control-plane-1:~$ kubectl -n kube-system get deployment coredns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           18h

user@control-plane-1:~$ sudo crictl info | jq '.status.conditions'
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
```
#### Smoke tests
```
user@control-plane-1:~$ kubectl create namespace kdl-smoke
namespace/kdl-smoke created

user@control-plane-1:~$ kubectl -n kdl-smoke run net-a --image=busybox:1.36 --restart=Never -- sleep 3600
pod/net-a created

user@control-plane-1:~$ kubectl -n kdl-smoke run net-b --image=busybox:1.36 --restart=Never -- sleep 3600
pod/net-b created

user@control-plane-1:~$ kubectl -n kdl-smoke get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP                NODE       NOMINATED NODE   READINESS GATES
net-a   1/1     Running   0          18m   192.168.226.66    worker-1   <none>           <none>
net-b   1/1     Running   0          17m   192.168.133.194   worker-2   <none>           <none>

user@control-plane-1:~$ NET_B_IP=$(kubectl -n kdl-smoke get pod net-b -o jsonpath='{.status.podIP}')
echo "$NET_B_IP"
192.168.133.194

user@control-plane-1:~$ kubectl -n kdl-smoke exec net-a -- ping -c 3 "$NET_B_IP"
PING 192.168.133.194 (192.168.133.194): 56 data bytes
64 bytes from 192.168.133.194: seq=0 ttl=62 time=1.068 ms
64 bytes from 192.168.133.194: seq=1 ttl=62 time=0.445 ms
64 bytes from 192.168.133.194: seq=2 ttl=62 time=0.508 ms
--- 192.168.133.194 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.445/0.673/1.068 ms

user@control-plane-1:~$ kubectl -n kdl-smoke exec net-a -- nslookup kubernetes.default.svc.cluster.local
Server:         10.96.0.10
Address:        10.96.0.10:53
Name:   kubernetes.default.svc.cluster.local
Address: 10.96.0.1

user@control-plane-1:~$ kubectl delete namespace kdl-smoke
namespace "kdl-smoke" deleted

user@control-plane-1:~$ kubectl get namespace kdl-smoke 2>/dev/null || true
```
#### Финальная проверка
На control-plane node
```
user@control-plane-1:~$ kubectl get nodes -o wide
NAME              STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION              CONTAINER-RUNTIME
control-plane-1   Ready    control-plane   19h   v1.36.1   192.168.88.184   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-1          Ready    <none>          18h   v1.36.1   192.168.88.134   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1
worker-2          Ready    <none>          18h   v1.36.1   192.168.88.243   <none>        Ubuntu 24.04.4 LTS   6.8.0-124-generic (amd64)   containerd://2.2.1

user@control-plane-1:~$ kubectl get pods -A -o wide
NAMESPACE         NAME                                       READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
calico-system     calico-apiserver-9dcbdf8f6-rtt4g           1/1     Running   0          60m   192.168.247.135   control-plane-1   <none>           <none>
calico-system     calico-apiserver-9dcbdf8f6-v4465           1/1     Running   0          60m   192.168.247.134   control-plane-1   <none>           <none>
calico-system     calico-kube-controllers-6c677558b8-8hb9n   1/1     Running   0          60m   192.168.247.131   control-plane-1   <none>           <none>
calico-system     calico-node-2zpr6                          1/1     Running   0          60m   192.168.88.184    control-plane-1   <none>           <none>
calico-system     calico-node-5kqm7                          1/1     Running   0          60m   192.168.88.134    worker-1          <none>           <none>
calico-system     calico-node-tc25r                          1/1     Running   0          60m   192.168.88.243    worker-2          <none>           <none>
calico-system     calico-typha-648595f9df-tg4fb              1/1     Running   0          60m   192.168.88.243    worker-2          <none>           <none>
calico-system     calico-typha-648595f9df-w2952              1/1     Running   0          60m   192.168.88.134    worker-1          <none>           <none>
calico-system     csi-node-driver-ccj4j                      2/2     Running   0          60m   192.168.247.129   control-plane-1   <none>           <none>
calico-system     csi-node-driver-hj7cc                      2/2     Running   0          60m   192.168.226.65    worker-1          <none>           <none>
calico-system     csi-node-driver-r4n6x                      2/2     Running   0          60m   192.168.133.193   worker-2          <none>           <none>
calico-system     goldmane-6885dcb7d-ncwc8                   1/1     Running   0          60m   192.168.247.136   control-plane-1   <none>           <none>
calico-system     whisker-5674dc44cb-njzf9                   2/2     Running   0          59m   192.168.247.130   control-plane-1   <none>           <none>
kube-system       coredns-589f44dc88-fwct6                   1/1     Running   0          19h   192.168.247.132   control-plane-1   <none>           <none>
kube-system       coredns-589f44dc88-zx2qb                   1/1     Running   0          19h   192.168.247.133   control-plane-1   <none>           <none>
kube-system       etcd-control-plane-1                       1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-system       kube-apiserver-control-plane-1             1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-system       kube-controller-manager-control-plane-1    1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-system       kube-proxy-hdvtm                           1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-system       kube-proxy-jc7lk                           1/1     Running   0          18h   192.168.88.134    worker-1          <none>           <none>
kube-system       kube-proxy-spstk                           1/1     Running   0          18h   192.168.88.243    worker-2          <none>           <none>
kube-system       kube-scheduler-control-plane-1             1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
tigera-operator   tigera-operator-85dbff4478-ntdvh           1/1     Running   0          68m   192.168.88.243    worker-2          <none>           <none>

user@control-plane-1:~$ kubectl get tigerastatus 2>/dev/null || true
NAME        AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
apiserver   True        False         False      58m     All objects available
calico      True        False         False      59m     All objects available
goldmane    True        False         False      58m     All objects available
ippools     True        False         False      60m     All objects available
tiers       True        False         False      58m     All objects available
whisker     True        False         False      58m     All objects available

user@control-plane-1:~$ kubectl -n calico-system get pods -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
calico-apiserver-9dcbdf8f6-rtt4g           1/1     Running   0          60m   192.168.247.135   control-plane-1   <none>           <none>
calico-apiserver-9dcbdf8f6-v4465           1/1     Running   0          60m   192.168.247.134   control-plane-1   <none>           <none>
calico-kube-controllers-6c677558b8-8hb9n   1/1     Running   0          60m   192.168.247.131   control-plane-1   <none>           <none>
calico-node-2zpr6                          1/1     Running   0          60m   192.168.88.184    control-plane-1   <none>           <none>
calico-node-5kqm7                          1/1     Running   0          60m   192.168.88.134    worker-1          <none>           <none>
calico-node-tc25r                          1/1     Running   0          60m   192.168.88.243    worker-2          <none>           <none>
calico-typha-648595f9df-tg4fb              1/1     Running   0          60m   192.168.88.243    worker-2          <none>           <none>
calico-typha-648595f9df-w2952              1/1     Running   0          60m   192.168.88.134    worker-1          <none>           <none>
csi-node-driver-ccj4j                      2/2     Running   0          60m   192.168.247.129   control-plane-1   <none>           <none>
csi-node-driver-hj7cc                      2/2     Running   0          60m   192.168.226.65    worker-1          <none>           <none>
csi-node-driver-r4n6x                      2/2     Running   0          60m   192.168.133.193   worker-2          <none>           <none>
goldmane-6885dcb7d-ncwc8                   1/1     Running   0          60m   192.168.247.136   control-plane-1   <none>           <none>
whisker-5674dc44cb-njzf9                   2/2     Running   0          60m   192.168.247.130   control-plane-1   <none>           <none>

user@control-plane-1:~$ kubectl -n kube-system get pods -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
coredns-589f44dc88-fwct6                  1/1     Running   0          19h   192.168.247.132   control-plane-1   <none>           <none>
coredns-589f44dc88-zx2qb                  1/1     Running   0          19h   192.168.247.133   control-plane-1   <none>           <none>
etcd-control-plane-1                      1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-apiserver-control-plane-1            1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-controller-manager-control-plane-1   1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-hdvtm                          1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>
kube-proxy-jc7lk                          1/1     Running   0          18h   192.168.88.134    worker-1          <none>           <none>
kube-proxy-spstk                          1/1     Running   0          18h   192.168.88.243    worker-2          <none>           <none>
kube-scheduler-control-plane-1            1/1     Running   0          19h   192.168.88.184    control-plane-1   <none>           <none>

user@control-plane-1:~$ sudo crictl info | jq '.status.conditions'
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

user@control-plane-1:~$ sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun 11 16:05 10-calico.conflist
-rw------- 1 root root 2801 Jun 11 16:05 calico-kubeconfig
```
На worker nodes:
```
user@worker-1:~$ sudo crictl info | jq '.status.conditions'
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

user@worker-1:~$ sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun 11 16:05 10-calico.conflist
-rw------- 1 root root 2801 Jun 11 16:05 calico-kubeconfig

user@worker-2:~$ sudo crictl info | jq '.status.conditions'
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

user@worker-2:~$ sudo ls -l /etc/cni/net.d/ || true
total 8
-rw------- 1 root root  735 Jun 11 16:05 10-calico.conflist
-rw------- 1 root root 2801 Jun 11 16:05 calico-kubeconfig
```

### Ключевые выводы команд

 - RuntimeReady: true, значит можно продолжить установку
 - В custom-resources.yaml указан нужный IP pool
 - При Dry-run применении Calico manifests получал сообщение "metadata.annotations: Too long".Его можно избежать, если использовать create вместо apply
 - Calico CRDs и Tigera Operator установлены
 - Calico успешно установлен и запущен
 - Ноды перешли в статус Ready
 - smoke test на сетевую связность подов проходит успешно
 - test pod может резолвить kubernetes.default.svc.cluster.local
 - RuntimeReady: true на всех nodes
 - NetworkReady: true на всех nodes
 - CoreDNS pods находятся в Running


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

- Разделение задач между kubelet и CNI
- От чего зависят и когда меняются статусы компонент в процессе развертывания

### Вопросы

- 

### Статус

done