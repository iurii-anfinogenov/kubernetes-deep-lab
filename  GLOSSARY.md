# Glossary

Этот файл фиксирует термины, которые используются в Kubernetes Deep Lab.

Цель файла - уменьшить количество непонятных англицизмов в материалах и дать единое объяснение терминов на русском языке.

## Общие термины

### Runtime

**Runtime** - среда выполнения.

В контексте Kubernetes чаще всего имеется в виду container runtime - компонент, который запускает и обслуживает контейнеры на node.

Примеры:

* `containerd`;
* CRI-O.

В материалах лучше писать:

* "среда выполнения контейнеров";
* при первом упоминании: "среда выполнения контейнеров, далее container runtime".

### Bootstrap

**Bootstrap** - начальная подготовка или первичная инициализация системы.

В Kubernetes это процесс, при котором пустая node или пустой cluster приводятся в рабочее состояние.

Примеры:

* `kubeadm init` выполняет начальную инициализацию control plane;
* `kubeadm join` присоединяет worker node к cluster.

В материалах лучше писать:

* "первичная инициализация";
* "начальная настройка";
* "подготовка к запуску".

### Cluster

**Cluster** - кластер.

В Kubernetes это группа nodes, которыми управляет control plane.

В материалах можно писать:

* "кластер";
* "Kubernetes-кластер".

### Node

**Node** - узел кластера.

Это физический сервер или виртуальная машина, на которой работает kubelet и запускаются pods.

В материалах лучше писать:

* "node";
* при первом упоминании: "node, то есть узел Kubernetes-кластера".

### Control plane

**Control plane** - управляющая плоскость Kubernetes.

Это набор компонентов, которые принимают решения и хранят состояние кластера:

* API Server;
* etcd;
* Scheduler;
* Controller Manager.

В материалах лучше писать:

* "control plane";
* при первом упоминании: "control plane, то есть управляющая плоскость Kubernetes".

### Worker node

**Worker node** - рабочий узел.

Это node, на которой обычно запускаются пользовательские workloads.

В материалах лучше писать:

* "worker node";
* "рабочая node";
* "рабочий узел".

### Workload

**Workload** - прикладная нагрузка.

В Kubernetes так называют пользовательские приложения и объекты, которые управляют их запуском.

Примеры:

* Pod;
* Deployment;
* StatefulSet;
* DaemonSet;
* Job;
* CronJob.

В материалах лучше писать:

* "прикладная нагрузка";
* "рабочая нагрузка";
* при необходимости оставить "workload" как Kubernetes-термин.

## Компоненты Kubernetes

### API Server

**API Server** - центральный API-компонент Kubernetes.

Через него проходят почти все операции управления кластером.

Примеры:

* `kubectl get pods`;
* создание Deployment;
* изменение ConfigMap;
* запросы kubelet;
* запросы controller-manager.

В материалах лучше писать:

* "API Server";
* "сервер Kubernetes API".

### etcd

**etcd** - распределенное key-value хранилище состояния Kubernetes.

В нем хранится состояние кластера:

* nodes;
* pods;
* services;
* secrets;
* configmaps;
* другие Kubernetes objects.

В материалах лучше писать:

* `etcd`;
* "хранилище состояния кластера".

### Scheduler

**Scheduler** - планировщик Kubernetes.

Он выбирает, на какую node назначить Pod.

В материалах лучше писать:

* "планировщик";
* при первом упоминании: "Scheduler, то есть планировщик Kubernetes".

### Controller Manager

**Controller Manager** - компонент, который запускает контроллеры Kubernetes.

Контроллеры постоянно сравнивают желаемое состояние с фактическим и пытаются привести кластер к желаемому состоянию.

В материалах лучше писать:

* "Controller Manager";
* "менеджер контроллеров".

### Kubelet

**Kubelet** - агент Kubernetes на каждой node.

Он отвечает за запуск и контроль pods на конкретной node.

Kubelet:

* регистрирует node в API Server;
* получает описание pods;
* обращается к container runtime;
* следит за состоянием containers;
* отчитывается в API Server.

В материалах лучше писать:

* `kubelet`;
* "агент node".

### kube-proxy

**kube-proxy** - компонент сетевого уровня Kubernetes, который реализует Service networking на node.

Он может настраивать правила через:

* iptables;
* nftables;
* IPVS.

В некоторых CNI-решениях kube-proxy может быть заменен другим механизмом.

## Контейнеры и Linux

### Container runtime

**Container runtime** - среда выполнения контейнеров.

Она отвечает за запуск containers на node.

В нашей лаборатории используется `containerd`.

### containerd

**containerd** - среда выполнения контейнеров.

Kubernetes через kubelet обращается к containerd по CRI, а containerd уже запускает containers через более низкоуровневые механизмы.

### CRI

**CRI** - Container Runtime Interface.

Это интерфейс, через который kubelet взаимодействует со средой выполнения контейнеров.

Проще:

```text
kubelet -> CRI -> containerd -> runc -> Linux kernel
```

### runc

**runc** - низкоуровневый инструмент запуска containers.

Он работает ближе к Linux kernel и использует namespaces, cgroups и другие механизмы Linux.

### Namespace

**Namespace** - механизм Linux для изоляции процессов.

Не путать с Kubernetes Namespace.

Linux namespaces изолируют:

* процессы;
* сеть;
* mount points;
* hostname;
* пользователей;
* IPC.

Kubernetes Namespace - это логическое разделение объектов внутри Kubernetes API.

### cgroups

**cgroups** - механизм Linux для ограничения и учета ресурсов.

С помощью cgroups ограничиваются:

* CPU;
* memory;
* I/O;
* pids.

Kubernetes использует cgroups через kubelet и container runtime.

## kubeadm

### kubeadm

**kubeadm** - инструмент для первичной инициализации Kubernetes-кластера.

Он не является постоянным управляющим компонентом.

`kubeadm` помогает:

* создать control plane;
* сгенерировать certificates;
* создать kubeconfig-файлы;
* создать static pod manifests;
* подготовить join-command для worker nodes.

После установки кластером управляют Kubernetes components, kubelet, container runtime и CNI.

### kubeadm init

**kubeadm init** - команда первичной инициализации control plane.

Она создает первый control-plane node.

### kubeadm join

**kubeadm join** - команда присоединения node к существующему Kubernetes-кластеру.

Обычно используется для worker nodes.

### Static pod

**Static pod** - pod, который создается kubelet напрямую из локального manifest-файла на node.

В kubeadm-кластере control plane components обычно запускаются как static pods.

Файлы находятся здесь:

```text
/etc/kubernetes/manifests/
```

Важно:

* static pod не создается через Deployment;
* static pod не назначается Scheduler;
* kubelet сам следит за manifest-файлом;
* если manifest изменить, kubelet пересоздаст pod.

### Manifest

**Manifest** - YAML-файл с описанием Kubernetes object или static pod.

Примеры:

* Pod manifest;
* Deployment manifest;
* Service manifest;
* static pod manifest.

В материалах лучше писать:

* "manifest";
* "YAML-описание";
* "файл описания объекта".

## Конфигурация и сертификаты

### kubeconfig

**kubeconfig** - файл подключения к Kubernetes API.

Он описывает:

* адрес API Server;
* cluster;
* пользователя;
* certificates или token;
* context.

Важно:

kubeconfig используется не только `kubectl`.

Его также используют Kubernetes components, например:

* scheduler;
* controller-manager;
* kubelet.

### Certificate

**Certificate** - сертификат.

В Kubernetes сертификаты используются для TLS, аутентификации компонентов и защищенного взаимодействия.

В материалах лучше писать:

* "сертификат";
* при необходимости: "TLS certificate".

### CA

**CA** - Certificate Authority, центр сертификации.

CA подписывает сертификаты компонентов Kubernetes.

В материалах лучше писать:

* "центр сертификации";
* при первом упоминании: "CA, то есть центр сертификации".

### Private key

**Private key** - закрытый ключ.

Его нельзя публиковать в lab journal, GitHub, чатах и отчетах.

В материалах лучше писать:

* "закрытый ключ";
* при первом упоминании: "private key, то есть закрытый ключ".

### Token

**Token** - токен доступа.

В Kubernetes может использоваться для аутентификации или присоединения node к cluster.

Пример:

* bootstrap token для `kubeadm join`.

## Сеть Kubernetes

### CNI

**CNI** - Container Network Interface.

Это интерфейс и набор плагинов для настройки сети containers.

Примеры CNI-решений:

* Calico;
* Cilium;
* Flannel.

В материалах лучше писать:

* "CNI";
* при первом упоминании: "CNI, то есть сетевой интерфейс контейнеров".

### Service

**Service** - объект Kubernetes для стабильного доступа к pods.

Service дает постоянный виртуальный адрес или DNS-имя для группы pods.

### ClusterIP

**ClusterIP** - тип Service, доступный внутри кластера.

### NodePort

**NodePort** - тип Service, который открывает порт на каждой node.

### LoadBalancer

**LoadBalancer** - тип Service, который обычно создает внешний балансировщик через cloud provider или внешний контроллер.

В bare metal или домашней лаборатории требует отдельного решения, например MetalLB.

### DNS

**DNS** - система имен.

В Kubernetes обычно используется CoreDNS, чтобы pods могли обращаться к services по именам.

### CoreDNS

**CoreDNS** - DNS-сервер внутри Kubernetes-кластера.

Он обслуживает Kubernetes DNS-зоны и позволяет обращаться к services по DNS-именам.

## Объекты Kubernetes

### Pod

**Pod** - минимальная единица запуска в Kubernetes.

Pod содержит один или несколько containers, которые разделяют сеть и некоторые namespaces.

### Deployment

**Deployment** - объект для управления stateless-приложениями.

Он управляет ReplicaSet и обеспечивает обновления pods.

### ReplicaSet

**ReplicaSet** - объект, который поддерживает нужное количество одинаковых pods.

Обычно создается и управляется Deployment.

### DaemonSet

**DaemonSet** - объект, который запускает pod на каждой подходящей node.

Примеры:

* CNI-agent;
* log collector;
* node exporter.

### StatefulSet

**StatefulSet** - объект для stateful-приложений.

Он дает pods стабильные имена, порядок запуска и связь с постоянным хранилищем.

### Job

**Job** - объект для выполнения задачи до успешного завершения.

### CronJob

**CronJob** - объект для запуска Jobs по расписанию.

## Безопасность и доступ

### RBAC

**RBAC** - Role-Based Access Control.

Это модель управления доступом по ролям.

В Kubernetes RBAC отвечает на вопрос:

"Кто, что и с какими объектами может делать?"

### ServiceAccount

**ServiceAccount** - учетная запись для workload внутри Kubernetes.

Pods используют ServiceAccount для доступа к Kubernetes API.

### Admission controller

**Admission controller** - механизм проверки или изменения запроса перед сохранением объекта в Kubernetes API.

Примеры задач:

* запрет небезопасных pods;
* добавление default values;
* проверка policy.

В материалах лучше писать:

* "admission controller";
* "контроллер допуска".

## Эксплуатация

### Drain

**Drain** - подготовка node к обслуживанию.

Команда `kubectl drain` вытесняет pods с node, чтобы node можно было безопасно обновить или выключить.

Это изменяющее действие. В лабораториях перед ним нужен отдельный шаг и подтверждение.

### Cordon

**Cordon** - запрет планирования новых pods на node.

Старые pods при этом не удаляются.

Это изменяющее действие.

### Uncordon

**Uncordon** - разрешение снова планировать pods на node.

### Rollback

**Rollback** - откат изменения.

В лабораториях перед рискованными действиями нужно заранее понимать, как вернуть систему в предыдущее состояние.

### Verification

**Verification** - проверка результата.

После каждого изменения нужно проверить, что система пришла в ожидаемое состояние.

В материалах лучше писать:

* "проверка результата";
* "verification steps" можно оставлять только в шаблонах риска.


