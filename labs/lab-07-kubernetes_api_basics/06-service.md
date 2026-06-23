# Service

## Цель урока

В этом уроке мы разбираем объект `Service` и связываем его с тем, что уже было пройдено в Lab 07: `Namespace`, `Pod`, `labels`, `selectors`, `ReplicaSet` и `Deployment`.

К этому моменту у нас уже есть цепочка:

```text
Deployment
  -> ReplicaSet
      -> Pod
```

`Deployment` управляет обновлениями и количеством реплик, `ReplicaSet` поддерживает нужное число Pod, а Pod запускает контейнеры приложения. Но у этой схемы все еще есть проблема: Pod не является стабильной точкой доступа.

Pod можно удалить, он может быть пересоздан на другой node, его IP может измениться. Поэтому клиентам нельзя надежно обращаться напрямую к Pod IP. Для стабильного доступа к группе Pod в Kubernetes используется `Service`.

В этой части Lab 07 мы создадим `ClusterIP Service` для Deployment `kdl-nginx-deployment` из предыдущего урока и проверим, как Service находит Pod через labels и как Kubernetes формирует `EndpointSlice`.

## Что такое Service

`Service` - это Kubernetes-объект, который задает стабильную сетевую точку доступа к одному или нескольким Pod.

Если очень кратко:

```text
Client Pod
  -> Service DNS name / ClusterIP
      -> selected Pod
      -> selected Pod
      -> selected Pod
```

Service не запускает контейнеры и не создает Pod. Он выбирает уже существующие Pod по `selector` и направляет трафик на них.

Связь с предыдущими объектами:

```text
Deployment создает ReplicaSet
ReplicaSet создает Pod
Pod получает labels
Service через selector находит Pod с нужными labels
```

То есть `Service` зависит не от имени Pod, а от labels.

## Почему нельзя использовать Pod IP напрямую

Pod IP существует только пока существует конкретный Pod.

Например, если удалить Pod из Deployment:

```bash
kubectl -n kdl-lab07 get pods -l app.kubernetes.io/name=nginx -o wide
kubectl -n kdl-lab07 delete pod <pod-name>
kubectl -n kdl-lab07 get pods -l app.kubernetes.io/name=nginx -o wide
```

ReplicaSet создаст новый Pod, но новый Pod может получить другой IP.

Это нормальное поведение Kubernetes. Pod - заменяемая единица. Для stateless-приложений предполагается, что любой Pod из набора реплик может быть заменен.

Поэтому в Kubernetes обычно не строят доступ так:

```text
client -> Pod IP
```

Вместо этого используют:

```text
client -> Service -> Pod
```

## Тип Service ClusterIP

Базовый тип Service - `ClusterIP`.

`ClusterIP` создает виртуальный IP внутри кластера. Этот IP доступен только внутри cluster network. Он нужен для связи между приложениями внутри Kubernetes.

Пример:

```text
frontend Pod -> backend Service -> backend Pod
```

Для Lab 07 мы используем именно `ClusterIP`, потому что сейчас цель - понять базовый механизм Service, selector и связь с Pod. `NodePort`, `LoadBalancer`, `Ingress` и `Gateway API` будут разбираться позже в сетевых лабораторных.

## Как Service выбирает Pod

Service использует `spec.selector`.

В нашем Deployment Pod template содержит стабильные labels:

```yaml
labels:
  app.kubernetes.io/name: nginx
  app.kubernetes.io/component: web
  kdl/component: deployment
  kdl/version: "v1"
```

Service должен выбирать Pod по labels, которые не поменяются при обычном rollout. Поэтому хороший selector для этой лабораторной:

```yaml
selector:
  app.kubernetes.io/name: nginx
  app.kubernetes.io/component: web
```

Мы не используем `kdl/version: "v1"` в selector Service. Version label может измениться при обновлении приложения, а Service должен продолжать вести на новую версию Pod.

Пример Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: kdl-lab07
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/component: web
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
```

Здесь важно:

- `selector` означает: Service должен выбрать Pod, у которых есть labels `app.kubernetes.io/name=nginx` и `app.kubernetes.io/component=web`;
- `port: 80` - порт самого Service;
- `targetPort: http` - именованный порт контейнера внутри выбранного Pod;
- `protocol: TCP` - протокол Service port.

`targetPort` может быть числом, например `80`, или именем container port. В Deployment из прошлого урока порт контейнера назван `http`:

```yaml
ports:
  - name: http
    containerPort: 80
```

Именованный `targetPort` удобен тем, что Service остается читаемым: видно, что трафик идет не просто на число `80`, а на HTTP-порт приложения.

## Связь Service и Deployment

Обычно Service создают рядом с Deployment.

Deployment задает labels в двух местах:

```yaml
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      kdl/component: deployment
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
        app.kubernetes.io/component: web
        kdl/component: deployment
```

Важно не путать:

```text
Deployment .spec.selector.matchLabels
  -> выбирает Pod для ReplicaSet/Deployment

Service .spec.selector
  -> выбирает Pod для сетевого доступа
```

Они часто похожи, но это разные selectors у разных объектов.

Service selector не обязан дословно повторять Deployment selector. Он должен совпадать с labels на Pod template и быть достаточно точным, чтобы не выбрать чужие Pod.

В этой лабораторной Service выбирает Pod из Deployment по двум labels:

```yaml
spec:
  selector:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/component: web
```

Эта пара не выбирает старые Pod из уроков про одиночный Pod, labels или ReplicaSet, потому что у них нет label `app.kubernetes.io/component=web`.

## EndpointSlice

Когда Service имеет selector, Kubernetes автоматически создает `EndpointSlice` для Pod, которые подходят под selector Service.

Упрощенно:

```text
Service selector
  -> matching Pods
      -> EndpointSlice
          -> Pod IP:port
          -> Pod IP:port
          -> Pod IP:port
```

`EndpointSlice` - это объект, через который Kubernetes хранит список backend endpoints для Service.

Для диагностики Service важно смотреть не только сам Service, но и EndpointSlice. Старый объект `Endpoints` все еще может отображаться в командах и в `describe service`, но современный механизм масштабирования endpoints - это `EndpointSlice`.

Readiness тоже важна. Если у Pod есть `readinessProbe`, Kubernetes может отметить endpoint как неготовый, пока приложение не готово принимать трафик. Поэтому Service, EndpointSlice и readinessProbe нужно воспринимать как одну практическую цепочку.

## Подготовка

Проверь, что namespace и Deployment из предыдущих уроков существуют.

Риск: LOW.

```bash
kubectl get namespace kdl-lab07
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
```

Если namespace или Deployment отсутствуют, примените манифесты из предыдущих уроков:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/namespace.yaml
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
```

Проверь labels у Pod перед созданием Service:

```bash
kubectl -n kdl-lab07 get pods \
  -l app.kubernetes.io/name=nginx,app.kubernetes.io/component=web \
  --show-labels
```

Если команда ничего не вернула, Service с таким selector тоже не найдет Pod. Сначала нужно разобраться с Deployment и labels.

## Манифест Service

Файл для этой части лабораторной:

```text
labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
```

Содержимое:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/part-of: kubernetes-deep-lab
    app.kubernetes.io/component: web
    kdl/lab: "07"
    kdl/component: service
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/component: web
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: http
```

Этот Service предполагает, что Deployment Pod template уже содержит labels:

```yaml
labels:
  app.kubernetes.io/name: nginx
  app.kubernetes.io/component: web
```

Не меняй selector вслепую. Сначала проверь labels у Pod.

## Проверка перед применением

Риск: LOW.

Что будет изменено: будет создан один объект `Service` в namespace `kdl-lab07`.

Dry-run:

```bash
kubectl apply --dry-run=server \
  -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
```

Ожидаемый результат:

```text
service/nginx created (server dry run)
```

Перед реальным применением полезно посмотреть diff.

Риск: LOW.

```bash
kubectl diff -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
```

Если Service еще не создан, `kubectl diff` покажет добавление нового объекта. Exit code `1` при наличии diff - это нормально.

Если dry-run прошел успешно, применяем:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
```

Ожидаемый результат:

```text
service/nginx created
```

## Проверка Service

Проверить Service:

```bash
kubectl -n kdl-lab07 get service nginx
```

Ожидаемый результат:

```text
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   <cluster-ip>    <none>        80/TCP    <age>
```

Проверить подробности:

```bash
kubectl -n kdl-lab07 describe service nginx
```

Обрати внимание на поля:

```text
Selector: app.kubernetes.io/component=web,app.kubernetes.io/name=nginx
Type: ClusterIP
IP:
Port:
TargetPort:
Endpoints:
```

Если `Endpoints` пустой, Service не нашел готовые Pod или selector не совпал с labels.

## Проверка Pod labels

Проверить labels у Pod:

```bash
kubectl -n kdl-lab07 get pods --show-labels
```

Нужно убедиться, что у Pod есть labels, которые совпадают с Service selector:

```text
app.kubernetes.io/name=nginx
app.kubernetes.io/component=web
```

Проверить выборку вручную:

```bash
kubectl -n kdl-lab07 get pods \
  -l app.kubernetes.io/name=nginx,app.kubernetes.io/component=web \
  -o wide
```

Если команда ничего не вернула, Service тоже не сможет выбрать Pod.

## Проверка EndpointSlice

Проверить EndpointSlice:

```bash
kubectl -n kdl-lab07 get endpointslice
```

Посмотреть EndpointSlice, связанный с Service:

```bash
kubectl -n kdl-lab07 get endpointslice \
  -l kubernetes.io/service-name=nginx
```

Подробно:

```bash
kubectl -n kdl-lab07 describe endpointslice \
  -l kubernetes.io/service-name=nginx
```

Здесь нужно увидеть адреса Pod, которые Service использует как backend endpoints.

Если EndpointSlice есть, но endpoint помечен как неготовый, проверь readiness Pod:

```bash
kubectl -n kdl-lab07 get pods \
  -l app.kubernetes.io/name=nginx,app.kubernetes.io/component=web
kubectl -n kdl-lab07 describe pod <pod-name>
```

## Проверка доступа внутри кластера

Для проверки создадим временный Pod с curl.

Риск: LOW.

Что будет изменено: временно будет создан Pod для диагностики. После выхода он удалится за счет `--rm`.

Команда:

```bash
kubectl -n kdl-lab07 run curl-test \
  --image=curlimages/curl:8.10.1 \
  --rm -it \
  --restart=Never \
  -- curl -sS http://nginx
```

Если приложение отвечает на HTTP, ты увидишь HTML-ответ от nginx за Service.

Если образ не скачивается из-за ограничений сети, это не ошибка Service. В таком случае нужно использовать уже доступный diagnostic image из вашей лабораторной среды или заранее загруженный образ.

## Проверка через DNS-имя

Внутри того же namespace можно обращаться к Service по короткому имени:

```text
http://nginx
```

Полное DNS-имя Service:

```text
nginx.kdl-lab07.svc.cluster.local
```

Проверка:

```bash
kubectl -n kdl-lab07 run curl-test \
  --image=curlimages/curl:8.10.1 \
  --rm -it \
  --restart=Never \
  -- curl -sS http://nginx.kdl-lab07.svc.cluster.local
```

DNS в Kubernetes будет подробно разбираться позже, но сейчас важно увидеть связь:

```text
Service name
  -> Kubernetes DNS
      -> ClusterIP
          -> selected Pod
```

## Что происходит внутри

Упрощенная схема:

```text
kubectl apply -f service.yaml
  -> API Server сохраняет Service в etcd
      -> EndpointSlice controller находит Pod по selector
          -> создает/обновляет EndpointSlice
              -> kube-proxy или другой service implementation обновляет правила на nodes
                  -> трафик на ClusterIP попадает к одному из Pod
```

Важно: Service сам по себе не проверяет, что приложение внутри Pod реально отвечает. Он выбирает Pod по labels и использует readiness-состояние endpoints. Если label совпал, но контейнер не слушает нужный port, Service будет существовать, EndpointSlice может быть заполнен, но запросы будут падать.

## Типичные ошибки

### Selector не совпадает с labels Pod

Симптом:

```bash
kubectl -n kdl-lab07 describe service nginx
```

В выводе:

```text
Endpoints: <none>
```

Проверка:

```bash
kubectl -n kdl-lab07 get pods --show-labels
kubectl -n kdl-lab07 get pods \
  -l app.kubernetes.io/name=nginx,app.kubernetes.io/component=web
```

Исправление: привести Service selector к фактическим labels Pod или исправить labels в Pod template Deployment.

### Неверный targetPort

Симптом: Endpoints есть, но запросы не проходят.

Проверка Service:

```bash
kubectl -n kdl-lab07 describe service nginx
```

Проверка Pod:

```bash
kubectl -n kdl-lab07 describe pod <pod-name>
```

Нужно убедиться, что контейнер реально слушает порт, указанный в `targetPort`. В нашем случае Service использует `targetPort: http`, поэтому в Pod должен быть container port с именем `http`.

### Pod не Ready

Симптом: Pod существует, Service существует, но endpoint не готов или запросы не проходят во время старта.

Проверка:

```bash
kubectl -n kdl-lab07 get pods \
  -l app.kubernetes.io/name=nginx,app.kubernetes.io/component=web
kubectl -n kdl-lab07 describe pod <pod-name>
```

Ищи readiness probe, events и состояние контейнера.

### Приложение слушает только localhost внутри контейнера

Симптом: Pod Running, Service есть, endpoints есть, но подключение не работает.

Причина: приложение внутри контейнера может слушать `127.0.0.1`, а не `0.0.0.0`.

Это уже проблема уровня приложения/контейнера, а не Service.

### Service создан не в том namespace

Service выбирает Pod только внутри своего namespace.

Проверка:

```bash
kubectl get service -A | grep nginx
kubectl get pods -A --show-labels | grep app.kubernetes.io/name=nginx
```

Исправление: создать Service в том же namespace, где находятся Pod.

## Мини-практика

### Задание

1. Проверь labels у Pod, созданных Deployment.
2. Примени `manifests/06-service-nginx.yaml` через dry-run, diff и apply.
3. Проверь Service и EndpointSlice.
4. Проверь доступ через короткое Service DNS name.
5. Проверь доступ через полное Service DNS name.
6. Удали один Pod из Deployment и убедись, что Service продолжает вести на новые Pod.

Команды проверки:

```bash
kubectl -n kdl-lab07 get pods --show-labels
kubectl apply --dry-run=server \
  -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
kubectl diff -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/06-service-nginx.yaml
kubectl -n kdl-lab07 get service nginx
kubectl -n kdl-lab07 describe service nginx
kubectl -n kdl-lab07 get endpointslice -l kubernetes.io/service-name=nginx
kubectl -n kdl-lab07 get pods -o wide
```

Проверка пересоздания Pod:

```bash
kubectl -n kdl-lab07 delete pod <pod-name>
kubectl -n kdl-lab07 get pods -l app.kubernetes.io/name=nginx -o wide
kubectl -n kdl-lab07 get endpointslice -l kubernetes.io/service-name=nginx
```

Ожидаемый вывод: Pod может измениться, но Service остается тем же.

## Что нужно понять после урока

После этого урока ты должен понимать:

- почему Pod IP нельзя считать стабильной точкой доступа;
- зачем нужен Service;
- как Service выбирает Pod через selector;
- почему selector Service должен совпадать с labels Pod, а не с именем Deployment;
- почему labels являются критически важной частью Kubernetes API;
- что такое ClusterIP;
- какую роль играет EndpointSlice;
- как readiness влияет на endpoints;
- как диагностировать Service без догадок;
- почему Service и Deployment связаны через labels, но не являются одним объектом.

## Источники

- Kubernetes Documentation: Services, Load Balancing, and Networking: https://kubernetes.io/docs/concepts/services-networking/
- Kubernetes Documentation: Service: https://kubernetes.io/docs/concepts/services-networking/service/
- Kubernetes Documentation: EndpointSlices: https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/
- Kubernetes Documentation: Deployment: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
