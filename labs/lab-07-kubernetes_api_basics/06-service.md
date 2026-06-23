# Service

## Цель урока

В этом уроке мы разбираем объект `Service` и связываем его с тем, что уже было пройдено в Lab 07: `Namespace`, `Pod`, `labels`, `selectors`, `ReplicaSet` и `Deployment`.

К этому моменту у нас уже есть важная цепочка:

```text
Deployment
  -> ReplicaSet
      -> Pod
```

`Deployment` управляет обновлениями и количеством реплик, `ReplicaSet` поддерживает нужное число Pod, а Pod запускает контейнеры приложения. Но у этой схемы все еще есть проблема: Pod не является стабильной точкой доступа.

Pod можно удалить, он может быть пересоздан на другой node, его IP может измениться. Поэтому клиентам нельзя надежно обращаться напрямую к Pod IP. Для стабильного доступа к группе Pod в Kubernetes используется `Service`.

Официальная документация Kubernetes определяет Service как абстракцию для доступа к приложению, работающему на наборе Pod. Service может иметь стабильный `ClusterIP`, через который клиенты внутри кластера обращаются к backend Pod.

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

То есть `Service` зависит не от имени Pod, а от меток.

## Почему нельзя использовать Pod IP напрямую

Pod IP существует только пока существует конкретный Pod.

Например, если удалить Pod из Deployment:

```bash
kubectl -n kdl-lab get pods -o wide
kubectl -n kdl-lab delete pod <pod-name>
kubectl -n kdl-lab get pods -o wide
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

Пример Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: kdl-lab
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 80
```

Здесь важно:

`selector.app: web` означает: Service должен выбрать Pod, у которых есть label `app=web`.

`port: 80` - порт самого Service.

`targetPort: 80` - порт контейнера внутри выбранного Pod.

Если у Pod нет label `app=web`, Service не будет направлять на него трафик.

## Связь Service и Deployment

Обычно Service создают рядом с Deployment.

Deployment задает labels в двух местах:

```yaml
spec:
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
```

Service должен использовать selector, который совпадает с labels Pod template:

```yaml
spec:
  selector:
    app: web
```

Важно не путать:

```text
Deployment .spec.selector.matchLabels
  -> выбирает Pod для ReplicaSet/Deployment

Service .spec.selector
  -> выбирает Pod для сетевого доступа
```

Они часто выглядят одинаково, но это разные selectors у разных объектов.

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

Для диагностики Service важно смотреть не только сам Service, но и EndpointSlice.

## Манифест Service

Создай файл:

```text
manifests/web-service.yaml
```

Содержимое:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: kdl-lab
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
    - name: http
      port: 80
      targetPort: 80
```

Этот Service предполагает, что в Deployment Pod template уже содержит label:

```yaml
labels:
  app: web
```

Если в твоем `05-deployment.md` и манифесте Deployment используется другое имя label, например `app: nginx`, тогда Service selector нужно привести к фактическому label из Deployment.

Не меняй selector вслепую. Сначала проверь labels у Pod.

## Проверка перед применением

Риск: LOW.

Что будет изменено: будет создан один объект `Service` в namespace `kdl-lab`.

Dry-run:

```bash
kubectl apply -f manifests/web-service.yaml --dry-run=client
```

Если dry-run прошел успешно, применяем:

```bash
kubectl apply -f manifests/web-service.yaml
```

## Проверка Service

Проверить Service:

```bash
kubectl -n kdl-lab get service
```

Ожидаемый результат:

```text
NAME   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web    ClusterIP   <cluster-ip>    <none>        80/TCP    <age>
```

Проверить подробности:

```bash
kubectl -n kdl-lab describe service web
```

Обрати внимание на поля:

```text
Selector: app=web
Type: ClusterIP
IP:
Port:
TargetPort:
Endpoints:
```

Если `Endpoints` пустой, Service не нашел Pod.

## Проверка Pod labels

Проверить labels у Pod:

```bash
kubectl -n kdl-lab get pods --show-labels
```

Нужно убедиться, что у Pod есть label, который совпадает с Service selector:

```text
app=web
```

Проверить выборку вручную:

```bash
kubectl -n kdl-lab get pods -l app=web -o wide
```

Если команда ничего не вернула, Service тоже не сможет выбрать Pod.

## Проверка EndpointSlice

Проверить EndpointSlice:

```bash
kubectl -n kdl-lab get endpointslice
```

Посмотреть EndpointSlice, связанный с Service:

```bash
kubectl -n kdl-lab get endpointslice -l kubernetes.io/service-name=web
```

Подробно:

```bash
kubectl -n kdl-lab describe endpointslice -l kubernetes.io/service-name=web
```

Здесь нужно увидеть адреса Pod, которые Service использует как backend endpoints.

## Проверка доступа внутри кластера

Для проверки создадим временный Pod с curl.

Риск: LOW.

Что будет изменено: временно будет создан Pod для диагностики. После выхода он удалится за счет `--rm`.

Команда:

```bash
kubectl -n kdl-lab run curl-test \
  --image=curlimages/curl:8.10.1 \
  --rm -it \
  --restart=Never \
  -- curl -sS http://web
```

Если приложение отвечает на HTTP, ты увидишь HTTP-ответ от Pod за Service.

Если образ не скачивается из-за ограничений сети, это не ошибка Service. В таком случае нужно использовать уже доступный diagnostic image из вашей лабораторной среды или заранее загруженный образ.

## Проверка через DNS-имя

Внутри того же namespace можно обращаться к Service по короткому имени:

```text
http://web
```

Полное DNS-имя Service:

```text
web.kdl-lab.svc.cluster.local
```

Проверка:

```bash
kubectl -n kdl-lab run curl-test \
  --image=curlimages/curl:8.10.1 \
  --rm -it \
  --restart=Never \
  -- curl -sS http://web.kdl-lab.svc.cluster.local
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

Важно: Service сам по себе не проверяет, что приложение внутри Pod реально отвечает. Он выбирает Pod по labels. Если label совпал, но контейнер не слушает нужный port, Service будет существовать, EndpointSlice может быть заполнен, но запросы будут падать.

## Типичные ошибки

### Selector не совпадает с labels Pod

Симптом:

```bash
kubectl -n kdl-lab describe service web
```

В выводе:

```text
Endpoints: <none>
```

Проверка:

```bash
kubectl -n kdl-lab get pods --show-labels
kubectl -n kdl-lab get pods -l app=web
```

Исправление: привести Service selector к фактическим labels Pod или исправить labels в Pod template Deployment.

### Неверный targetPort

Симптом: Endpoints есть, но запросы не проходят.

Проверка Service:

```bash
kubectl -n kdl-lab describe service web
```

Проверка Pod:

```bash
kubectl -n kdl-lab describe pod <pod-name>
```

Нужно убедиться, что контейнер реально слушает порт, указанный в `targetPort`.

### Приложение слушает только localhost внутри контейнера

Симптом: Pod Running, Service есть, endpoints есть, но подключение не работает.

Причина: приложение внутри контейнера может слушать `127.0.0.1`, а не `0.0.0.0`.

Это уже проблема уровня приложения/контейнера, а не Service.

### Service создан не в том namespace

Service выбирает Pod только внутри своего namespace.

Проверка:

```bash
kubectl get service -A | grep web
kubectl get pods -A --show-labels | grep app=web
```

Исправление: создать Service в том же namespace, где находятся Pod.

## Мини-практика

### Задание

1. Проверь labels у Pod, созданных Deployment.
2. Создай `manifests/web-service.yaml`.
3. Выполни dry-run.
4. Примени Service.
5. Проверь Service, EndpointSlice и доступ через Service DNS name.
6. Удали один Pod из Deployment и проверь, что Service продолжает вести на новый Pod.

Команды проверки:

```bash
kubectl -n kdl-lab get pods --show-labels
kubectl apply -f manifests/web-service.yaml --dry-run=client
kubectl apply -f manifests/web-service.yaml
kubectl -n kdl-lab get service web
kubectl -n kdl-lab describe service web
kubectl -n kdl-lab get endpointslice -l kubernetes.io/service-name=web
kubectl -n kdl-lab get pods -o wide
```

Проверка пересоздания Pod:

```bash
kubectl -n kdl-lab delete pod <pod-name>
kubectl -n kdl-lab get pods -o wide
kubectl -n kdl-lab get endpointslice -l kubernetes.io/service-name=web
```

Ожидаемый вывод: Pod может измениться, но Service остается тем же.

## Что нужно понять после урока

После этого урока ты должен понимать:

- почему Pod IP нельзя считать стабильной точкой доступа;
- зачем нужен Service;
- как Service выбирает Pod через selector;
- почему labels являются критически важной частью Kubernetes API;
- что такое ClusterIP;
- какую роль играет EndpointSlice;
- как диагностировать Service без догадок;
- почему Service и Deployment связаны через labels, но не являются одним объектом.

## Вставка в `05-deployment.md`

Эту секцию можно добавить в конец урока про Deployment перед переходом к Service.

```markdown
## Переход к Service

После Deployment у нас появляется управляемое приложение:

```text
Deployment
  -> ReplicaSet
      -> Pod
```

Но Deployment не дает стабильную сетевую точку доступа к приложению.

Pod может быть удален и создан заново. Новый Pod может получить другой IP. Поэтому обращаться напрямую к Pod IP нельзя считать надежным способом доступа.

Для стабильного доступа к группе Pod используется `Service`.

Service выбирает Pod по labels:

```text
Service selector
  -> Pod labels
      -> Pod IP:port
```

Поэтому labels, которые мы указываем в Pod template Deployment, важны не только для самого Deployment и ReplicaSet, но и для будущего Service.

В следующем уроке мы создадим `ClusterIP Service`, свяжем его с Pod из Deployment через selector и проверим, как Kubernetes формирует EndpointSlice.
```

## Источники

- Kubernetes Documentation: Service
- Kubernetes Documentation: Connecting Applications with Services
- Kubernetes Documentation: EndpointSlices
- Kubernetes Documentation: Deployment
