# Deployment

## Цель урока

В этом уроке разбираем `Deployment` как основной объект Kubernetes для запуска stateless-приложений.

После предыдущих уроков у нас уже есть базовая цепочка:

```text
Namespace -> Pod -> Labels/Selectors -> ReplicaSet
```

Теперь добавляем следующий уровень:

```text
Deployment -> ReplicaSet -> Pod -> Container
```

Главная цель - понять, зачем нужен `Deployment`, если уже существуют `Pod` и `ReplicaSet`, и как Kubernetes выполняет обновление приложения без ручного удаления Pod.

## Что уже нужно понимать перед уроком

Перед этим уроком студент должен понимать:

- что такое `Namespace`;
- что такое `Pod`;
- почему одиночный Pod не является устойчивым способом запуска приложения;
- как работают `labels` и `selectors`;
- зачем нужен `ReplicaSet`;
- что `ReplicaSet` поддерживает нужное количество Pod, но сам по себе не является удобным инструментом для обновлений приложения.

Если эти темы непонятны, сначала нужно вернуться к файлам:

```text
01-namespace.md
02-pod.md
03-labels-and-selectors.md
04-replicaset.md
```

## Что такое Deployment

`Deployment` - это объект Kubernetes, который описывает желаемое состояние приложения и управляет созданием, обновлением и откатом `ReplicaSet`.

Deployment не запускает контейнеры напрямую.

Он работает через цепочку контроллеров:

```text
Deployment controller
  -> создает или обновляет ReplicaSet
      -> ReplicaSet controller создает или удаляет Pod
          -> kubelet запускает контейнеры через container runtime
```

То есть Deployment - это не "контейнер" и не "виртуальная машина". Это запись в Kubernetes API, на которую реагирует контроллер.

## Как думать о Deployment как инженер

Когда ты читаешь Deployment manifest, не пытайся сразу запомнить все поля. Читай его как ответы на пять инженерных вопросов:

| Вопрос | Где смотреть |
|---|---|
| Что за приложение мы запускаем? | `metadata.name`, `metadata.labels` |
| Сколько экземпляров нужно? | `spec.replicas` |
| Какими Pod управляет Deployment? | `spec.selector.matchLabels` |
| Как создавать новые Pod? | `spec.template` |
| Как обновлять приложение? | `spec.strategy`, rollout status, ReplicaSet history |

Это очень практичная привычка. В реальной работе Deployment чаще всего диагностируют не "сверху вниз по YAML", а именно через эти вопросы.

## Зачем нужен Deployment

`ReplicaSet` умеет поддерживать количество Pod. Например, если нужно 3 Pod, ReplicaSet будет следить, чтобы их было 3.

Но ReplicaSet неудобен для обновлений:

- он не хранит удобную историю изменений;
- он не дает нормальный механизм rollout;
- он не дает удобный rollback;
- при смене версии приложения обычно нужно создавать новый ReplicaSet вручную или менять шаблон напрямую;
- в обычной эксплуатации напрямую ReplicaSet почти не создают.

Deployment решает эти задачи.

Он позволяет:

- описывать приложение декларативно;
- запускать нужное количество реплик;
- обновлять image или Pod template;
- выполнять rolling update;
- смотреть статус обновления;
- откатываться на предыдущую ревизию;
- масштабировать приложение;
- видеть связь между Deployment, ReplicaSet и Pod.

## Важная инженерная идея

Kubernetes не "выполняет manifest" как shell-скрипт.

Когда мы применяем Deployment manifest, мы сохраняем желаемое состояние в Kubernetes API.

Дальше контроллеры постоянно сравнивают:

```text
desired state - что должно быть
actual state  - что есть сейчас
```

Если состояние отличается, контроллеры пытаются привести кластер к нужному состоянию.

Пример:

```text
В Deployment указано replicas: 3
В кластере реально есть 2 подходящих Pod
ReplicaSet controller создает еще один Pod
```

Или:

```text
В Deployment изменился image
Deployment controller создает новый ReplicaSet
Старый ReplicaSet постепенно уменьшается
Новый ReplicaSet постепенно увеличивается
```

## Подготовка

В этом уроке используем отдельный namespace из предыдущих лабораторных.

Проверь, что namespace существует:

```bash
kubectl get namespace kdl-lab07
```

Если namespace отсутствует, создай его через manifest из первого урока Lab 07, а не отдельной императивной командой.

Проверка текущих объектов:

```bash
kubectl -n kdl-lab07 get all
```

На этом этапе в namespace могут оставаться объекты из прошлых уроков. Это нормально, но для чистого выполнения лучше использовать уникальные имена объектов из этого урока.

## Manifest Deployment

Открой файл. Если проходишь лабораторную руками с нуля, создай его:

```text
labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Содержимое:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kdl-nginx-deployment
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/part-of: kubernetes-deep-lab
    app.kubernetes.io/component: web
    kdl/lab: "07"
    kdl/component: deployment
spec:
  replicas: 3
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 120
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      kdl/component: deployment
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
        app.kubernetes.io/part-of: kubernetes-deep-lab
        app.kubernetes.io/component: web
        kdl/lab: "07"
        kdl/component: deployment
        kdl/version: "v1"
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.5-alpine
          ports:
            - name: http
              containerPort: 80
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 3
            periodSeconds: 5
            timeoutSeconds: 2
            failureThreshold: 3
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
```

## Разбор manifest

### apiVersion

```yaml
apiVersion: apps/v1
```

Deployment относится к API group `apps` и версии `v1`.

Это отличается от Pod и Namespace, которые находятся в core API group и обычно имеют `apiVersion: v1`.

### kind

```yaml
kind: Deployment
```

`kind` указывает тип объекта Kubernetes.

### metadata

```yaml
metadata:
  name: kdl-nginx-deployment
  namespace: kdl-lab07
```

Имя объекта уникально внутри namespace.

### labels

```yaml
labels:
  app.kubernetes.io/name: nginx
  app.kubernetes.io/part-of: kubernetes-deep-lab
  app.kubernetes.io/component: web
  kdl/lab: "07"
  kdl/component: deployment
```

Labels помогают находить и группировать объекты.

В этом уроке мы используем recommended labels Kubernetes-стиля и отдельные учебные labels `kdl/*`.

Практический смысл:

- `app.kubernetes.io/name: nginx` говорит, что это nginx;
- `app.kubernetes.io/part-of: kubernetes-deep-lab` связывает объект с учебным проектом;
- `kdl/lab: "07"` показывает номер лабораторной;
- `kdl/component: deployment` отделяет Pod из этого урока от Pod и ReplicaSet из предыдущих частей.

### replicas

```yaml
replicas: 3
```

Это желаемое количество Pod.

Важно: Deployment сам не создает Pod напрямую. Он создает ReplicaSet, а ReplicaSet уже создает Pod.

### selector

```yaml
selector:
  matchLabels:
    app.kubernetes.io/name: nginx
    kdl/component: deployment
```

Selector определяет, какими Pod управляет Deployment через ReplicaSet.

Критически важно: selector должен совпадать с labels внутри `spec.template.metadata.labels`.

Точнее: все labels из `spec.selector.matchLabels` должны присутствовать в `spec.template.metadata.labels` с теми же значениями.

Pod template может содержать дополнительные labels. Например, `kdl/version: "v1"` есть у Pod, но его нет в selector. Это сделано специально: version label можно менять при обновлении, а selector должен оставаться стабильным.

Также selector у Deployment является важным полем, которое нельзя бездумно менять после создания объекта. В реальной эксплуатации изменение selector может привести к потере связи между контроллером и Pod.

### template

```yaml
template:
  metadata:
    labels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/part-of: kubernetes-deep-lab
      app.kubernetes.io/component: web
      kdl/lab: "07"
      kdl/component: deployment
      kdl/version: "v1"
  spec:
    containers:
      - name: nginx
        image: nginx:1.27.5-alpine
```

`template` - это шаблон Pod.

Deployment не хранит список конкретных Pod. Он хранит шаблон, по которому ReplicaSet создает Pod.

Если изменить Pod template, например image, Kubernetes создаст новый ReplicaSet и начнет rollout.

Важно: изменение `spec.replicas` не создает новый ReplicaSet, потому что template Pod не поменялся. А изменение `image`, labels внутри template, probes, env, command или resources обычно меняет Pod template и запускает rollout.

### strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

`strategy` описывает, как Deployment будет менять старые Pod на новые.

В этом уроке мы явно указали простые учебные значения:

- `maxUnavailable: 1` - во время обновления можно временно иметь максимум на 1 доступный Pod меньше желаемого количества;
- `maxSurge: 1` - во время обновления можно временно создать максимум на 1 Pod больше желаемого количества.

Для `replicas: 3` это значит, что Kubernetes будет обновлять приложение постепенно, не удаляя сразу все Pod.

### readinessProbe

```yaml
readinessProbe:
  httpGet:
    path: /
    port: http
```

`readinessProbe` отвечает на вопрос: "Готов ли Pod принимать трафик?"

Для Deployment это особенно важно. Rolling update должен считать новый Pod готовым только после того, как приложение реально отвечает. Без readinessProbe Kubernetes может слишком рано считать контейнер готовым, хотя приложение внутри еще не принимает запросы.

### resources

```yaml
resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 200m
    memory: 128Mi
```

`requests` помогают scheduler выбрать node, где есть достаточно ресурсов. `limits` задают верхнюю границу потребления контейнера.

Это не главная тема урока, но хорошая DevOps-привычка: даже учебный Deployment полезно писать так, чтобы он был похож на аккуратный production manifest.

## Проверка структуры через kubectl explain

Риск: LOW

Перед применением полезно посмотреть схему объекта прямо из Kubernetes API:

```bash
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain deployment.spec.selector
kubectl explain deployment.spec.template
kubectl explain deployment.spec.strategy
```

Главная задача здесь не выучить весь вывод, а увидеть, что Deployment действительно состоит из `replicas`, `selector`, `template` и `strategy`.

## Dry-run перед применением

Риск: LOW

Команда ничего не меняет в кластере. Она проверяет, что manifest можно отправить в API Server.

```bash
kubectl apply --dry-run=server -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Ожидаемый результат:

```text
deployment.apps/kdl-nginx-deployment created (server dry run)
```

Если есть ошибка, не применяй manifest. Сначала нужно исправить YAML или схему объекта.

## Просмотр изменений через diff

Риск: LOW

`kubectl diff` показывает, чем manifest отличается от live-состояния в API Server.

```bash
kubectl diff -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Если Deployment еще не создан, `kubectl diff` покажет добавление нового объекта. Если объект уже существует, команда поможет понять, какие поля изменятся после `apply`.

Важно: `kubectl diff` может завершиться с exit code `1`, если отличия найдены. В данном случае это не поломка, а нормальный сигнал "diff не пустой".

## Применение manifest

Риск: LOW

Будет создан объект Deployment в namespace `kdl-lab07`. Он создаст ReplicaSet, а ReplicaSet создаст 3 Pod.

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Ожидаемый результат:

```text
deployment.apps/kdl-nginx-deployment created
```

## Проверка Deployment

Посмотри Deployment:

```bash
kubectl -n kdl-lab07 get deployment
```

Более точечно:

```bash
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment
```

Ожидаем увидеть, что нужное количество реплик готово:

```text
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
kdl-nginx-deployment   3/3     3            3           ...
```

Поля:

- `READY` - сколько Pod готово из желаемого количества;
- `UP-TO-DATE` - сколько Pod соответствует текущему Pod template;
- `AVAILABLE` - сколько Pod доступно для обслуживания;
- `AGE` - сколько времени существует объект.

## Проверка цепочки Deployment -> ReplicaSet -> Pod

Посмотри все основные объекты:

```bash
kubectl -n kdl-lab07 get deployment,replicaset,pod -l kdl/component=deployment
```

Ожидаем увидеть:

```text
deployment.apps/kdl-nginx-deployment
replicaset.apps/kdl-nginx-deployment-...
pod/kdl-nginx-deployment-...
pod/kdl-nginx-deployment-...
pod/kdl-nginx-deployment-...
```

Это важный момент урока: Deployment создал ReplicaSet, а ReplicaSet создал Pod.

## Проверка ownerReferences

Посмотри один Pod:

```bash
kubectl -n kdl-lab07 get pod -l kdl/component=deployment
```

Выбери имя одного Pod и выполни:

```bash
kubectl -n kdl-lab07 get pod <pod-name> -o yaml
```

Найди блок:

```yaml
metadata:
  ownerReferences:
```

У Pod владельцем будет ReplicaSet.

Теперь посмотри ReplicaSet:

```bash
kubectl -n kdl-lab07 get replicaset -l kdl/component=deployment
```

Выбери имя ReplicaSet и выполни:

```bash
kubectl -n kdl-lab07 get replicaset <replicaset-name> -o yaml
```

У ReplicaSet владельцем будет Deployment.

Итоговая связь:

```text
Deployment owns ReplicaSet
ReplicaSet owns Pod
```

## Describe Deployment

```bash
kubectl -n kdl-lab07 describe deployment kdl-nginx-deployment
```

Обрати внимание на блоки:

- `Replicas`;
- `StrategyType`;
- `RollingUpdateStrategy`;
- `Pod Template`;
- `Conditions`;
- `OldReplicaSets`;
- `NewReplicaSet`;
- `Events`.

Особенно важно читать `Events`. Там видно, какой ReplicaSet был создан и как менялось количество Pod.

## Conditions Deployment

У Deployment есть status conditions. Они отвечают не на вопрос "какой YAML мы хотели", а на вопрос "что сейчас происходит с rollout".

Посмотри conditions коротко:

```bash
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment \
  -o jsonpath='{range .status.conditions[*]}{.type}{"="}{.status}{" reason="}{.reason}{" message="}{.message}{"\n"}{end}'
```

Обычно полезны:

- `Available` - есть ли минимально доступные Pod;
- `Progressing` - продвигается ли rollout;
- `ReplicaFailure` - были ли проблемы при создании Pod.

Если rollout завис, сначала смотри `Conditions`, потом `Events`, потом переходи к ReplicaSet и Pod.

## Стратегия обновления

Deployment обычно использует стратегию `RollingUpdate`. В нашем manifest она указана явно, чтобы студент видел не только результат, но и настройку, которая за него отвечает.

Это значит, что при изменении Pod template Kubernetes не удаляет сразу все старые Pod. Он постепенно создает новые Pod и уменьшает количество старых.

Упрощенно:

```text
Было:
old ReplicaSet: 3 Pod
new ReplicaSet: 0 Pod

Во время обновления:
old ReplicaSet: 2 Pod
new ReplicaSet: 1 Pod

Потом:
old ReplicaSet: 1 Pod
new ReplicaSet: 2 Pod

В конце:
old ReplicaSet: 0 Pod
new ReplicaSet: 3 Pod
```

Так Kubernetes снижает риск простоя приложения во время обновления.

## Обновление image через manifest

Теперь сделаем обновление декларативно.

Открой файл:

```bash
vim labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Замени image:

```yaml
image: nginx:1.27.5-alpine
```

на:

```yaml
image: nginx:1.28.0-alpine
```

И заодно поменяй version label внутри `spec.template.metadata.labels`:

```yaml
kdl/version: "v1"
```

на:

```yaml
kdl/version: "v2"
```

Обрати внимание: мы меняем label в Pod template, но не меняем `spec.selector.matchLabels`. Selector должен оставаться стабильным, иначе Deployment может потерять связь с управляемыми Pod.

Перед применением выполни dry-run:

```bash
kubectl apply --dry-run=server -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Применение:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Если хочешь увидеть rollout вживую, перед `apply` можно открыть второй терминал:

```bash
kubectl -n kdl-lab07 get replicaset,pod -l kdl/component=deployment -w
```

Во время обновления будет видно, как появляется новый ReplicaSet, новые Pod проходят readiness, а старый ReplicaSet уменьшается до нуля.

Проверка rollout:

```bash
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
```

Ожидаемый результат:

```text
deployment "kdl-nginx-deployment" successfully rolled out
```

## Что произошло при обновлении

Посмотри ReplicaSet:

```bash
kubectl -n kdl-lab07 get replicaset -l kdl/component=deployment
```

Теперь должно быть минимум два ReplicaSet:

- старый ReplicaSet с `DESIRED=0`;
- новый ReplicaSet с `DESIRED=3`.

Это нормальное поведение Deployment.

Старый ReplicaSet остается для возможности rollback.

## История rollout

```bash
kubectl -n kdl-lab07 rollout history deployment/kdl-nginx-deployment
```

Ожидаем увидеть список ревизий.

Важно: история rollout полезнее, если при изменениях указывать причину изменения через annotation или вести изменения через Git. В реальной эксплуатации история должна быть связана с commit, pull request или release.

## Rollback

Риск: LOW для учебного namespace.

Rollback изменит активную версию Deployment на предыдущую ревизию.

Перед rollback посмотри текущий image:

```bash
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Откат на предыдущую ревизию:

```bash
kubectl -n kdl-lab07 rollout undo deployment/kdl-nginx-deployment
```

Проверка rollout:

```bash
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
```

Проверка image после rollback:

```bash
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

## Масштабирование Deployment

Масштабирование можно сделать двумя способами.

### Вариант 1. Декларативно через manifest

Открой manifest:

```bash
vim labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Измени:

```yaml
replicas: 3
```

на:

```yaml
replicas: 5
```

Dry-run:

```bash
kubectl apply --dry-run=server -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Применение:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Проверка:

```bash
kubectl -n kdl-lab07 get deployment,replicaset,pod -l kdl/component=deployment
```

### Вариант 2. Императивно через kubectl scale

Для обучения полезно знать команду:

```bash
kubectl -n kdl-lab07 scale deployment/kdl-nginx-deployment --replicas=2
```

Но в GitOps-подходе и в обычной декларативной эксплуатации лучше менять manifest, чтобы Git оставался источником правды.

После императивного scale manifest и фактическое состояние могут начать отличаться. Если потом снова выполнить `kubectl apply -f ...`, Kubernetes вернет количество replicas к значению из manifest.

## Проверка Pod после масштабирования

```bash
kubectl -n kdl-lab07 get pods -l kdl/component=deployment -o wide
```

Обрати внимание:

- сколько Pod существует;
- на каких nodes они размещены;
- все ли Pod в состоянии `Running`;
- все ли Pod имеют `READY 1/1`.

## Диагностика Deployment

Основные команды:

```bash
kubectl -n kdl-lab07 get deployment kdl-nginx-deployment
kubectl -n kdl-lab07 describe deployment kdl-nginx-deployment
kubectl -n kdl-lab07 get replicaset -l kdl/component=deployment
kubectl -n kdl-lab07 get pods -l kdl/component=deployment -o wide
kubectl -n kdl-lab07 describe pod <pod-name>
kubectl -n kdl-lab07 logs <pod-name>
kubectl -n kdl-lab07 get events --sort-by=.lastTimestamp
```

Если Deployment не выходит в готовое состояние, сначала нужно определить слой проблемы:

```text
Deployment не создает ReplicaSet?
  -> смотреть describe deployment и events

ReplicaSet есть, но Pod не создаются?
  -> смотреть describe replicaset и events

Pod есть, но Pending?
  -> смотреть scheduler events, ресурсы nodes, taints, nodeSelector, affinity

Pod есть, но ContainerCreating?
  -> смотреть image pull, CNI, volumes, container runtime на node

Pod есть, но ImagePullBackOff?
  -> смотреть image name, tag, registry access, imagePullSecrets

Pod есть, но CrashLoopBackOff?
  -> смотреть logs, command, args, env, config, exit code
```

## Частая ошибка: selector не совпадает с labels

Неправильный пример:

```yaml
selector:
  matchLabels:
    app: nginx

template:
  metadata:
    labels:
      app: web
```

В таком случае контроллер не сможет корректно связать selector с Pod template.

Правильный принцип:

```text
spec.selector.matchLabels должен совпадать с spec.template.metadata.labels
```

Не обязательно, чтобы labels были только такими. Но все labels из selector должны быть в Pod template.

## Частая ошибка: менять live-объект и забывать manifest

Например:

```bash
kubectl -n kdl-lab07 scale deployment/kdl-nginx-deployment --replicas=10
```

Если в manifest осталось:

```yaml
replicas: 3
```

то следующий `kubectl apply -f ...` вернет `replicas: 3`.

Инженерный вывод: если проект ведется декларативно, источник правды должен быть в manifest и Git, а не в ручных изменениях live-объектов.

## Удаление Deployment

Риск: MEDIUM

Удаление Deployment удалит связанный ReplicaSet и управляемые им Pod через cascading deletion.

Для учебного namespace это допустимо, но в production такое действие может остановить приложение.

Dry-run:

```bash
kubectl -n kdl-lab07 delete deployment kdl-nginx-deployment --dry-run=server
```

Удаление:

```bash
kubectl -n kdl-lab07 delete deployment kdl-nginx-deployment
```

Проверка:

```bash
kubectl -n kdl-lab07 get deployment,replicaset,pod -l kdl/component=deployment
```

Rollback plan:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

После восстановления проверить:

```bash
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
kubectl -n kdl-lab07 get deployment,replicaset,pod -l kdl/component=deployment
```

## Что важно понять

Deployment - это стандартный объект для stateless-приложений.

Он не заменяет Pod и ReplicaSet, а управляет ими.

Главная связь:

```text
Deployment описывает приложение
ReplicaSet поддерживает количество Pod
Pod содержит контейнеры
kubelet запускает контейнеры на node
container runtime запускает container process в Linux
```

Kubernetes abstractions заканчиваются там, где kubelet просит container runtime создать контейнер. Дальше начинается Linux: namespaces, cgroups, filesystem, network stack.

## Мини-практика

Выполни по шагам:

1. Открой или создай manifest `05-deployment-nginx.yaml`.
2. Посмотри структуру Deployment через `kubectl explain`.
3. Выполни server-side dry-run.
4. Посмотри `kubectl diff`.
5. Примени manifest.
6. Проверь Deployment.
7. Проверь ReplicaSet.
8. Проверь Pod.
9. Найди ownerReferences у Pod и ReplicaSet.
10. Посмотри `describe deployment` и `Conditions`.
11. Измени image и `kdl/version` в manifest.
12. Выполни dry-run.
13. Примени manifest.
14. Посмотри rollout status.
15. Посмотри rollout history.
16. Выполни rollback.
17. Проверь image после rollback.
18. Масштабируй Deployment декларативно через manifest.
19. Проверь, сколько Pod стало.
20. Удали Deployment через dry-run и реальное удаление.
21. Восстанови Deployment через manifest.

## Что прислать в lab journal

В отчет нужно добавить:

```text
1. Команду dry-run и результат.
2. Вывод или краткое описание `kubectl diff`.
3. Команду apply и результат.
4. Вывод get deployment.
5. Вывод get replicaset.
6. Вывод get pods -o wide.
7. Краткое объяснение связи Deployment -> ReplicaSet -> Pod.
8. Краткое объяснение, зачем нужны `strategy` и `readinessProbe`.
9. Вывод rollout status после обновления image.
10. Вывод rollout history.
11. Вывод image до rollback и после rollback.
12. Краткий вывод: зачем нужен Deployment, если уже есть ReplicaSet.
```

## Критерии готовности

Урок считается выполненным, если студент может объяснить:

- что такое Deployment;
- почему Deployment не создает контейнеры напрямую;
- как Deployment связан с ReplicaSet;
- как ReplicaSet связан с Pod;
- что такое Pod template;
- почему selector должен быть стабильным;
- зачем Deployment strategy влияет на обновление;
- зачем readinessProbe важен для rolling update;
- почему изменение image запускает rollout;
- зачем Kubernetes хранит старые ReplicaSet;
- как посмотреть rollout status;
- как выполнить rollback;
- почему в декларативном подходе изменения должны фиксироваться в manifest;
- почему напрямую ReplicaSet редко создают в обычной эксплуатации.

## Вопрос с собеседований

Вопрос:

```text
Чем Deployment отличается от ReplicaSet?
```

Краткий ответ:

```text
ReplicaSet поддерживает заданное количество Pod. Deployment управляет ReplicaSet и добавляет механизм обновлений, rollout history и rollback. В обычной эксплуатации stateless-приложения обычно создают через Deployment, а ReplicaSet появляется как управляемый объект под ним.
```

Привязка к лабораторной:

```text
В этом уроке Deployment kdl-nginx-deployment создает ReplicaSet, а ReplicaSet создает Pod. При изменении image Deployment создает новый ReplicaSet, постепенно переводит Pod на новую версию и сохраняет возможность rollback.
```

## Официальная документация

- Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- kubectl rollout: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_rollout/
- kubectl scale: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_scale/
