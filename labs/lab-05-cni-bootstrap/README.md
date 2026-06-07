# Lab 05 - CNI bootstrap with Calico

## Цель

Установить CNI plugin в kubeadm cluster и довести кластер до состояния, в котором:

- nodes переходят из `NotReady` в `Ready`;
- `containerd` показывает `NetworkReady: true`;
- CoreDNS получает возможность запуститься;
- Pod-to-Pod networking начинает работать между nodes;
- становится понятно, где заканчивается `kubeadm` и где начинается CNI.

В этой лабораторной используем Calico Open Source.

## Что важно понять

`kubeadm init` создает минимальный Kubernetes control plane, но не устанавливает Pod network.

До установки CNI нормальны такие симптомы:

- nodes находятся в `NotReady`;
- `crictl info` показывает `NetworkReady: false`;
- kubelet сообщает `cni plugin not initialized`;
- CoreDNS может быть в `Pending` или `ContainerCreating`;
- pods не могут получить полноценную сетевую связность.

После установки CNI kubelet получает возможность создавать network namespace для pods через CNI plugin.

## Где выполнять

Все команды этой лабораторной выполняются на control-plane node:

```text
kdl-cp-1
```

На worker nodes команды выполнять не нужно, если в инструкции явно не сказано обратное.

## Предварительные требования

Перед началом должны быть завершены:

- `lab-00-environment`;
- `lab-01-node-baseline`;
- `lab-02-containerd-runtime`;
- `lab-03-kubeadm-prerequisites`;
- `lab-04-kubeadm-cluster-bootstrap`.

Кластер должен быть создан через `kubeadm init`, worker nodes должны быть добавлены через `kubeadm join`.

Ожидаемое состояние перед Lab 05:

- `kubectl` работает на control-plane node;
- control-plane node видна в API;
- worker nodes видны в API;
- nodes могут быть `NotReady` до установки CNI;
- CoreDNS может быть не готов до установки CNI.

## Параметры этой лабораторной

В Lab 04 кластер должен быть создан с Pod CIDR:

```text
192.168.0.0/16
```

Это значение совпадает с типичным Calico default IP pool для kubeadm-лабораторий.

Используем Calico version:

```text
v3.32.0
```

Используем installation path через Tigera Operator.

## Проверка состояния до установки CNI

Команды ничего не меняют.

Выполнить на control-plane node:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl -n kube-system get pods -o wide
```

Проверить состояние kubelet/container runtime на каждой node можно так:

```bash
sudo crictl info | jq '.status.conditions'
```

Ожидаемо до CNI:

```text
RuntimeReady: true
NetworkReady: false
reason: NetworkPluginNotReady
message: cni plugin not initialized
```

Если `RuntimeReady` не `true`, не продолжайте установку CNI. Сначала нужно вернуться к диагностике containerd/kubelet.

## Подготовка каталога для Calico manifests

Создаем локальный каталог для manifest-файлов, чтобы не применять удаленные YAML напрямую без просмотра.

```bash
mkdir -p ~/kdl-manifests/calico-v3.32.0
cd ~/kdl-manifests/calico-v3.32.0
```

## Скачивание Calico manifests

Команды скачивают YAML-файлы, но еще ничего не применяют в cluster.

```bash
curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/v1_crd_projectcalico_org.yaml
curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/tigera-operator.yaml
curl -fL -O https://raw.githubusercontent.com/projectcalico/calico/v3.32.0/manifests/custom-resources.yaml
```

Проверить файлы:

```bash
ls -lh
wc -l *.yaml
```

Ожидаем файлы:

```text
custom-resources.yaml
tigera-operator.yaml
v1_crd_projectcalico_org.yaml
```

## Проверка Pod CIDR в Calico custom resources

Команды ничего не меняют.

```bash
grep -n "cidr\|blockSize\|encapsulation\|natOutgoing\|nodeSelector" custom-resources.yaml || true
```

Для этой лабораторной ожидаем, что Calico IP pool использует CIDR:

```text
192.168.0.0/16
```

Если в `custom-resources.yaml` указан другой CIDR, не продолжайте. Нужно привести Calico IP pool к Pod CIDR, который использовался в `kubeadm init --pod-network-cidr`.

## Dry-run применения Calico manifests

Команды проверяют YAML через Kubernetes API, но не создают ресурсы.

Сначала проверяем CRDs:

```bash
kubectl apply --dry-run=server -f v1_crd_projectcalico_org.yaml
```

Затем operator manifests:

```bash
kubectl apply --dry-run=server -f tigera-operator.yaml
```

Custom resources могут пройти dry-run только после появления CRDs в API. Поэтому `custom-resources.yaml` проверим после применения CRDs и operator manifests.

## Установка Calico CRDs и Tigera Operator

Что будет изменено:

- в cluster будут созданы Calico CRDs;
- будет создан namespace/operator resources;
- будет запущен Tigera Operator;
- Kubernetes начнет создавать Calico components.

Выполнить:

```bash
kubectl apply -f v1_crd_projectcalico_org.yaml
kubectl apply -f tigera-operator.yaml
```

Проверить:

```bash
kubectl get ns
kubectl get pods -A | grep -E 'tigera|calico' || true
kubectl get crd | grep -E 'projectcalico|operator.tigera' || true
```

Дождаться запуска operator:

```bash
kubectl -n tigera-operator get pods -o wide
kubectl -n tigera-operator wait --for=condition=Available deployment/tigera-operator --timeout=180s
```

Если wait завершился timeout, не повторяйте команды установки. Сначала посмотрите:

```bash
kubectl -n tigera-operator get pods -o wide
kubectl -n tigera-operator describe pods
kubectl -n tigera-operator logs deployment/tigera-operator --tail=100
```

## Применение Calico custom resources

Что будет изменено:

- будет создан Calico Installation custom resource;
- operator начнет разворачивать Calico CNI components;
- на nodes появятся CNI config и CNI plugin binaries;
- kubelet сможет инициализировать Pod network.

Сначала dry-run:

```bash
kubectl apply --dry-run=server -f custom-resources.yaml
```

Если dry-run успешен, применить manifest:

```bash
kubectl apply -f custom-resources.yaml
```

Проверить custom resources:

```bash
kubectl get installation.operator.tigera.io
kubectl get ippool.crd.projectcalico.org -A || true
```

## Наблюдение за разворачиванием Calico

Команды ничего не меняют.

```bash
kubectl get pods -A -o wide | grep -E 'calico|tigera'
```

Если доступен ресурс `tigerastatus`, проверить:

```bash
kubectl get tigerastatus
```

Ожидаем, что Calico components постепенно переходят в доступное состояние.

Также смотреть:

```bash
kubectl -n calico-system get pods -o wide
kubectl -n calico-system get daemonset
kubectl -n calico-system get deployment
```

## Проверка nodes после установки CNI

```bash
kubectl get nodes -o wide
```

Ожидаем:

```text
STATUS: Ready
```

Если nodes остаются `NotReady`, проверить причину:

```bash
kubectl describe node kdl-cp-1 | grep -A5 -E 'Ready|NetworkUnavailable|KubeletNotReady'
kubectl describe node kdl-w-5 | grep -A5 -E 'Ready|NetworkUnavailable|KubeletNotReady'
kubectl describe node kdl-w-6 | grep -A5 -E 'Ready|NetworkUnavailable|KubeletNotReady'
```

## Проверка CoreDNS

```bash
kubectl -n kube-system get pods -o wide
kubectl -n kube-system get deployment coredns
```

Ожидаем, что `coredns` pods переходят в `Running`.

Если CoreDNS не стартует, проверить:

```bash
kubectl -n kube-system describe deployment coredns
kubectl -n kube-system describe pods -l k8s-app=kube-dns
kubectl -n kube-system logs -l k8s-app=kube-dns --tail=100
```

## Проверка container runtime NetworkReady

Выполнить на каждой node:

```bash
sudo crictl info | jq '.status.conditions'
```

Ожидаем:

```text
RuntimeReady: true
NetworkReady: true
```

Если `NetworkReady` остается `false`, проверить на проблемной node:

```bash
ls -l /etc/cni/net.d/ || true
ls -l /opt/cni/bin/ | head
journalctl -u kubelet -n 100 --no-pager
```

## Smoke test: Pod-to-Pod networking

Создать временные test pods:

```bash
kubectl create namespace kdl-smoke
kubectl -n kdl-smoke run net-a --image=busybox:1.36 --restart=Never -- sleep 3600
kubectl -n kdl-smoke run net-b --image=busybox:1.36 --restart=Never -- sleep 3600
```

Дождаться запуска:

```bash
kubectl -n kdl-smoke wait --for=condition=Ready pod/net-a --timeout=120s
kubectl -n kdl-smoke wait --for=condition=Ready pod/net-b --timeout=120s
kubectl -n kdl-smoke get pods -o wide
```

Получить IP pod `net-b`:

```bash
NET_B_IP=$(kubectl -n kdl-smoke get pod net-b -o jsonpath='{.status.podIP}')
echo "$NET_B_IP"
```

Проверить связность:

```bash
kubectl -n kdl-smoke exec net-a -- ping -c 3 "$NET_B_IP"
```

Если ping не проходит, не удаляйте test namespace сразу. Сначала сохраните вывод:

```bash
kubectl -n kdl-smoke get pods -o wide
kubectl get nodes -o wide
kubectl -n calico-system get pods -o wide
```

## Smoke test: DNS inside cluster

```bash
kubectl -n kdl-smoke exec net-a -- nslookup kubernetes.default.svc.cluster.local
```

Ожидаем успешное DNS-разрешение service `kubernetes.default.svc.cluster.local`.

Если `nslookup` отсутствует в busybox image или не работает, проверьте сначала CoreDNS pods:

```bash
kubectl -n kube-system get pods -l k8s-app=kube-dns -o wide
```

## Очистка smoke test ресурсов

Удаляем только временный namespace этой лабораторной.

Dry-run:

```bash
kubectl delete namespace kdl-smoke --dry-run=server
```

Удаление:

```bash
kubectl delete namespace kdl-smoke
```

Проверка:

```bash
kubectl get namespace kdl-smoke 2>/dev/null || true
```

## Финальная проверка Lab 05

Выполнить на control-plane node:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get tigerastatus 2>/dev/null || true
kubectl -n calico-system get pods -o wide
kubectl -n kube-system get pods -o wide
```

Выполнить на каждой node:

```bash
sudo crictl info | jq '.status.conditions'
ls -l /etc/cni/net.d/ || true
```

## Критерии готовности

Lab 05 считается завершенной, если:

- Calico manifests применены без ошибок;
- Calico operator запущен;
- Calico components запущены;
- все Kubernetes nodes имеют статус `Ready`;
- `RuntimeReady: true` на всех nodes;
- `NetworkReady: true` на всех nodes;
- CoreDNS pods находятся в `Running`;
- test pod может пинговать другой test pod по Pod IP;
- test pod может резолвить `kubernetes.default.svc.cluster.local`.

## Что приложить в lab journal

Добавьте в lab journal:

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get tigerastatus 2>/dev/null || true
kubectl -n calico-system get pods -o wide
kubectl -n kube-system get pods -o wide
```

И с каждой node:

```bash
hostname
sudo crictl info | jq '.status.conditions'
ls -l /etc/cni/net.d/ || true
```

Для smoke test приложить:

```bash
kubectl -n kdl-smoke get pods -o wide
kubectl -n kdl-smoke exec net-a -- ping -c 3 <net-b-pod-ip>
kubectl -n kdl-smoke exec net-a -- nslookup kubernetes.default.svc.cluster.local
```

Если namespace уже удален, достаточно приложить вывод, сохраненный до удаления.

## Rollback

Rollback CNI может нарушить Pod networking и вернуть nodes в `NotReady`.

Не выполняйте rollback без причины.

Dry-run удаления smoke namespace:

```bash
kubectl delete namespace kdl-smoke --dry-run=server
```

Удаление Calico resources выполняется только если нужно полностью откатить Lab 05:

```bash
kubectl delete -f custom-resources.yaml
kubectl delete -f tigera-operator.yaml
kubectl delete -f v1_crd_projectcalico_org.yaml
```

После удаления CNI на nodes могут остаться локальные CNI files в `/etc/cni/net.d/` и runtime/network state. Для учебной лабораторной предпочтительнее пересоздать cluster через `kubeadm reset` в рамках отдельного rollback-плана, а не вручную чистить сетевое состояние.

## Инженерная заметка / Вопрос с собеседований

Вопрос:

Почему после `kubeadm init` кластер может быть создан, но nodes остаются `NotReady`?

Краткий ответ:

`kubeadm` создает control plane и базовые Kubernetes resources, но не устанавливает Pod network. Kubelet не может полноценно создавать Pod sandbox, пока CNI plugin не установлен и не настроен. Поэтому container runtime показывает `NetworkReady: false`, а node может оставаться `NotReady`.

Привязка к этой лабораторной:

В Lab 04 мы получили минимальный kubeadm cluster. В Lab 05 устанавливаем Calico, после чего CNI config появляется на nodes, kubelet начинает создавать Pod networking, CoreDNS стартует, а nodes переходят в `Ready`.

## Следующий шаг

Следующая лабораторная:

```text
lab-06-cluster-dissection
```

В ней нужно разобрать, что именно создали `kubeadm` и Calico:

- static pods;
- `/etc/kubernetes/manifests`;
- certificates;
- kubeconfigs;
- kubelet config;
- CNI config в `/etc/cni/net.d`;
- Calico components;
- CoreDNS;
- kube-proxy;
- базовые API resources.
