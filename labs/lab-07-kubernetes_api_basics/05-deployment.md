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

Создай файл:

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
    app.kubernetes.io/part-of: lab-07
    app.kubernetes.io/component: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/part-of: lab-07
      app.kubernetes.io/component: web
  template:
    metadata:
      labels:
        app.kubernetes.io/name: nginx
        app.kubernetes.io/part-of: lab-07
        app.kubernetes.io/component: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.5-alpine
          ports:
            - name: http
              containerPort: 80
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
  app.kubernetes.io/part-of: lab-07
  app.kubernetes.io/component: web
```

Labels помогают находить и группировать объекты.

В этом уроке мы используем recommended labels Kubernetes-стиля, чтобы сразу привыкать к понятной схеме маркировки.

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
    app.kubernetes.io/part-of: lab-07
    app.kubernetes.io/component: web
```

Selector определяет, какими Pod управляет Deployment через ReplicaSet.

Критически важно: selector должен совпадать с labels внутри `spec.template.metadata.labels`.

Если selector и labels Pod template не совпадают, Deployment не сможет корректно управлять Pod.

Также selector у Deployment является важным полем, которое нельзя бездумно менять после создания объекта. В реальной эксплуатации изменение selector может привести к потере связи между контроллером и Pod.

### template

```yaml
template:
  metadata:
    labels:
      app.kubernetes.io/name: nginx
      app.kubernetes.io/part-of: lab-07
      app.kubernetes.io/component: web
  spec:
    containers:
      - name: nginx
        image: nginx:1.27.5-alpine
```

`template` - это шаблон Pod.

Deployment не хранит список конкретных Pod. Он хранит шаблон, по которому ReplicaSet создает Pod.

Если изменить Pod template, например image, Kubernetes создаст новый ReplicaSet и начнет rollout.

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
kubectl -n kdl-lab07 get deployment,replicaset,pod -l app.kubernetes.io/name=nginx
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
kubectl -n kdl-lab07 get pod -l app.kubernetes.io/name=nginx
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
kubectl -n kdl-lab07 get replicaset -l app.kubernetes.io/name=nginx
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

## Стратегия обновления

По умолчанию Deployment использует стратегию `RollingUpdate`.

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

Перед применением выполни dry-run:

```bash
kubectl apply --dry-run=server -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

Применение:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

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
kubectl -n kdl-lab07 get replicaset -l app.kubernetes.io/name=nginx
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
kubectl -n kdl-lab07 get deployment,replicaset,pod -l app.kubernetes.io/name=nginx
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
kubectl -n kdl-lab07 get pods -l app.kubernetes.io/name=nginx -o wide
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
kubectl -n kdl-lab07 get replicaset -l app.kubernetes.io/name=nginx
kubectl -n kdl-lab07 get pods -l app.kubernetes.io/name=nginx -o wide
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
kubectl -n kdl-lab07 get deployment,replicaset,pod -l app.kubernetes.io/name=nginx
```

Rollback plan:

```bash
kubectl apply -f labs/lab-07-kubernetes_api_basics/manifests/05-deployment-nginx.yaml
```

После восстановления проверить:

```bash
kubectl -n kdl-lab07 rollout status deployment/kdl-nginx-deployment
kubectl -n kdl-lab07 get deployment,replicaset,pod -l app.kubernetes.io/name=nginx
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

1. Создай manifest `05-deployment-nginx.yaml`.
2. Выполни server-side dry-run.
3. Примени manifest.
4. Проверь Deployment.
5. Проверь ReplicaSet.
6. Проверь Pod.
7. Найди ownerReferences у Pod и ReplicaSet.
8. Измени image в manifest.
9. Выполни dry-run.
10. Примени manifest.
11. Посмотри rollout status.
12. Посмотри rollout history.
13. Выполни rollback.
14. Проверь image после rollback.
15. Масштабируй Deployment декларативно через manifest.
16. Проверь, сколько Pod стало.
17. Удали Deployment через dry-run и реальное удаление.
18. Восстанови Deployment через manifest.

## Что прислать в lab journal

В отчет нужно добавить:

```text
1. Команду dry-run и результат.
2. Команду apply и результат.
3. Вывод get deployment.
4. Вывод get replicaset.
5. Вывод get pods -o wide.
6. Краткое объяснение связи Deployment -> ReplicaSet -> Pod.
7. Вывод rollout status после обновления image.
8. Вывод rollout history.
9. Вывод image до rollback и после rollback.
10. Краткий вывод: зачем нужен Deployment, если уже есть ReplicaSet.
```

## Критерии готовности

Урок считается выполненным, если студент может объяснить:

- что такое Deployment;
- почему Deployment не создает контейнеры напрямую;
- как Deployment связан с ReplicaSet;
- как ReplicaSet связан с Pod;
- что такое Pod template;
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
