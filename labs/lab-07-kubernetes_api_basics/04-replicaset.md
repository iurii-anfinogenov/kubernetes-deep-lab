# Lab 07.04 - ReplicaSet

В этой части Lab 07 мы разберем объект `ReplicaSet`.

До этого мы создавали одиночный Pod. Мы увидели важное ограничение: если Pod удалить, Kubernetes не обязан создать его заново, если над ним нет controller.

Теперь мы создадим ReplicaSet и посмотрим, как Kubernetes поддерживает нужное количество Pods.

## Что изучаем

В этой части вы разберете:

- что такое ReplicaSet;
- зачем ReplicaSet нужен поверх Pod;
- из каких частей состоит ReplicaSet manifest;
- что такое `spec.replicas`;
- что такое `spec.selector`;
- что такое `spec.template`;
- почему selector должен совпадать с labels в Pod template;
- как ReplicaSet создает Pods;
- как ReplicaSet восстанавливает удаленный Pod;
- как масштабировать ReplicaSet;
- чем изменение через `kubectl scale` отличается от изменения manifest;
- что происходит, если изменить label у Pod;
- почему ReplicaSet обычно не создают вручную в production, а используют Deployment.

## Что такое ReplicaSet

ReplicaSet - это Kubernetes controller, который поддерживает нужное количество одинаковых Pods.

Пример:

```text
replicas: 3
```

Это означает:

```text
В cluster должно быть 3 подходящих Pod.
Если Pod меньше - создать новые.
Если Pod больше - удалить лишние.
```

ReplicaSet постоянно сравнивает desired state и actual state.

```text
desired state:
  replicas = 3

actual state:
  running Pods = 2

действие controller:
  создать еще 1 Pod
```

## Главное отличие от одиночного Pod

Одиночный Pod:

```text
Pod deleted
  -> Pod исчез
```

Pod под ReplicaSet:

```text
Pod deleted
  -> ReplicaSet замечает, что replicas меньше нужного
    -> ReplicaSet создает новый Pod
```

ReplicaSet не делает приложение "бессмертным", но поддерживает заданное количество Pods.

## Из чего состоит ReplicaSet

У ReplicaSet есть три ключевые части:

```yaml
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.5
```

Разбор:

| Поле | Что означает |
|---|---|
| `spec.replicas` | Сколько Pods нужно поддерживать |
| `spec.selector` | Какие Pods относятся к этому ReplicaSet |
| `spec.template` | Шаблон Pod, из которого ReplicaSet создает новые Pods |

Важно: `selector.matchLabels` должен совпадать с labels в `template.metadata.labels`.

Если selector и template labels не совпадают, Kubernetes не сможет корректно понять, какие Pods относятся к ReplicaSet.

## Почему ReplicaSet выбирает Pods через selector

ReplicaSet не хранит список имен Pods вручную.

Он смотрит на labels.

```text
ReplicaSet selector:
  app=nginx
  kdl/component=replicaset

Pod labels:
  app=nginx
  kdl/component=replicaset
```

Если labels совпадают, Pod подходит под ReplicaSet.

Именно поэтому в предыдущей части мы отдельно изучали labels и selectors.

## Проверка namespace

Все объекты создаем в namespace `kdl-lab07`.

Риск: LOW.

```bash
kubectl get namespace kdl-lab07
```

Если namespace отсутствует, вернитесь к `01-namespace.md`.

## Проверка, что старые Pods не мешают

Перед созданием ReplicaSet лучше проверить, что в namespace нет Pods из прошлых частей с такими же labels.

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 --show-labels
```

Если остались Pods из `03-labels-and-selectors.md`, удалите только их по label.

Сначала dry-run-проверка через get:

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=labels
```

Если вывод содержит только Pods из предыдущей части, удалите их.

Риск: MEDIUM.

```bash
kubectl delete pods -n kdl-lab07 -l kdl/component=labels
```

Не удаляйте namespace.

## Manifest для ReplicaSet

Создайте файл:

```text
manifests/replicaset-nginx.yaml
```

Содержимое:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/component: replicaset
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
      kdl/component: replicaset
  template:
    metadata:
      labels:
        app: nginx
        app.kubernetes.io/name: nginx
        app.kubernetes.io/part-of: kubernetes-deep-lab
        kdl/lab: "07"
        kdl/component: replicaset
    spec:
      containers:
        - name: nginx
          image: nginx:1.27.5
          ports:
            - containerPort: 80
```

Обратите внимание на `apiVersion`:

```yaml
apiVersion: apps/v1
```

ReplicaSet относится не к core API group `v1`, а к API group `apps/v1`.

## Разбор manifest

Ключевая часть:

```yaml
spec:
  replicas: 2
```

Это желаемое количество Pods.

Selector:

```yaml
selector:
  matchLabels:
    app: nginx
    kdl/component: replicaset
```

Он говорит ReplicaSet, какие Pods считать своими.

Template:

```yaml
template:
  metadata:
    labels:
      app: nginx
      kdl/component: replicaset
  spec:
    containers:
      - name: nginx
        image: nginx:1.27.5
```

Это шаблон для создания новых Pods.

Важно:

```text
selector.matchLabels должен совпадать с template.metadata.labels
```

## Проверка структуры через kubectl explain

Риск: LOW.

```bash
kubectl explain replicaset
```

Посмотрите `spec`:

```bash
kubectl explain replicaset.spec
```

Посмотрите replicas:

```bash
kubectl explain replicaset.spec.replicas
```

Посмотрите selector:

```bash
kubectl explain replicaset.spec.selector
```

Посмотрите template:

```bash
kubectl explain replicaset.spec.template
```

Задача: увидеть, что ReplicaSet manifest состоит из желаемого количества replicas, selector и Pod template.

## Dry-run перед созданием

Client-side dry-run.

Риск: LOW.

```bash
kubectl apply --dry-run=client -f manifests/replicaset-nginx.yaml
```

Server-side dry-run.

Риск: LOW.

```bash
kubectl apply --dry-run=server -f manifests/replicaset-nginx.yaml
```

Если dry-run прошел успешно, примените manifest.

## Создание ReplicaSet

Риск: MEDIUM.

Что изменится: в namespace `kdl-lab07` будет создан ReplicaSet `nginx-rs`, который создаст 2 Pods.

```bash
kubectl apply -f manifests/replicaset-nginx.yaml
```

Ожидаемый результат:

```text
replicaset.apps/nginx-rs created
```

## Проверка ReplicaSet

Риск: LOW.

```bash
kubectl get replicasets -n kdl-lab07
```

Сокращенная форма:

```bash
kubectl get rs -n kdl-lab07
```

Ожидаемо:

```text
NAME       DESIRED   CURRENT   READY
nginx-rs   2         2         2
```

Поля:

| Поле | Что означает |
|---|---|
| `DESIRED` | Сколько Pods должно быть |
| `CURRENT` | Сколько Pods сейчас создано |
| `READY` | Сколько Pods готовы |

## Проверка Pods, созданных ReplicaSet

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset -o wide
```

Вы должны увидеть два Pod.

Их имена будут не просто `nginx-rs`, а с автоматически добавленным суффиксом, например:

```text
nginx-rs-abcde
nginx-rs-fghij
```

Это нормально. ReplicaSet создает Pods из template и дает им уникальные имена.

## Проверка ownerReferences

Посмотрите один из Pods в YAML.

Сначала получите имя Pod:

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset -o name
```

Затем подставьте имя Pod:

```bash
kubectl get pod <pod-name> -n kdl-lab07 -o yaml
```

Найдите:

```text
metadata.ownerReferences
```

Там должно быть указано, что владельцем Pod является ReplicaSet `nginx-rs`.

Это важное отличие от одиночного Pod: Pod создан и управляется controller.

## kubectl describe ReplicaSet

Риск: LOW.

```bash
kubectl describe rs nginx-rs -n kdl-lab07
```

Обратите внимание на:

```text
Selector
Labels
Replicas
Pods Status
Pod Template
Events
```

В `Events` обычно видно, что ReplicaSet создал Pods.

## Проверка events

Риск: LOW.

```bash
kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
```

Вы можете увидеть события создания Pods, скачивания image и запуска containers.

## Проверка самовосстановления

Теперь удалим один Pod и посмотрим, что сделает ReplicaSet.

Сначала получите список Pods:

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Удалите один Pod.

Риск: MEDIUM.

Что изменится: один Pod будет удален, но ReplicaSet должен создать новый, чтобы вернуть replicas к 2.

```bash
kubectl delete pod <pod-name> -n kdl-lab07
```

Сразу проверьте Pods:

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Через несколько секунд снова проверьте:

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset -o wide
```

Ожидаемый вывод: снова должно быть 2 Pods.

Главный вывод:

```text
ReplicaSet поддерживает desired number of Pods.
Удаленный Pod не вернулся сам.
ReplicaSet создал новый Pod вместо него.
```

## Проверка ReplicaSet после удаления Pod

Риск: LOW.

```bash
kubectl get rs nginx-rs -n kdl-lab07
```

Ожидаемо:

```text
DESIRED = 2
CURRENT = 2
READY = 2
```

## Масштабирование через kubectl scale

Теперь увеличим количество replicas до 3.

Риск: MEDIUM.

Что изменится: ReplicaSet создаст дополнительный Pod.

```bash
kubectl scale rs nginx-rs -n kdl-lab07 --replicas=3
```

Проверьте:

```bash
kubectl get rs nginx-rs -n kdl-lab07
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Должно стать 3 Pods.

Команда `kubectl scale` предназначена для изменения размера Deployment, ReplicaSet, ReplicationController или Job. Она может использовать дополнительные preconditions вроде `--current-replicas`, чтобы изменение выполнилось только при ожидаемом текущем состоянии.

## Важный вывод про kubectl scale

Мы изменили replicas через команду:

```bash
kubectl scale rs nginx-rs -n kdl-lab07 --replicas=3
```

Но в файле `manifests/replicaset-nginx.yaml` все еще написано:

```yaml
replicas: 2
```

Значит, live-состояние cluster и manifest разошлись.

Проверьте diff:

Риск: LOW.

```bash
kubectl diff -f manifests/replicaset-nginx.yaml
```

Вы должны увидеть, что manifest хочет вернуть `replicas` к 2.

## Перенос scale-изменения в manifest

Обновите файл `manifests/replicaset-nginx.yaml`.

Замените:

```yaml
replicas: 2
```

на:

```yaml
replicas: 3
```

Проверьте diff:

Риск: LOW.

```bash
kubectl diff -f manifests/replicaset-nginx.yaml
```

Если diff пустой, значит manifest и live-состояние совпали.

Примените manifest:

Риск: MEDIUM.

```bash
kubectl apply -f manifests/replicaset-nginx.yaml
```

## Масштабирование вниз через manifest

Теперь уменьшим replicas обратно до 2 декларативно.

В файле `manifests/replicaset-nginx.yaml` замените:

```yaml
replicas: 3
```

на:

```yaml
replicas: 2
```

Проверьте diff:

Риск: LOW.

```bash
kubectl diff -f manifests/replicaset-nginx.yaml
```

Примените:

Риск: MEDIUM.

```bash
kubectl apply -f manifests/replicaset-nginx.yaml
```

Проверьте:

```bash
kubectl get rs nginx-rs -n kdl-lab07
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Ожидаемо снова 2 Pods.

## Что будет, если изменить label у Pod

ReplicaSet считает своими только Pods, которые подходят под selector.

Selector нашего ReplicaSet:

```yaml
selector:
  matchLabels:
    app: nginx
    kdl/component: replicaset
```

Посмотрите Pods:

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset --show-labels
```

Выберите один Pod и измените label `kdl/component`.

Риск: MEDIUM.

Что изменится: выбранный Pod перестанет подходить под selector ReplicaSet.

```bash
kubectl label pod <pod-name> -n kdl-lab07 kdl/component=orphan-test --overwrite
```

Проверьте все Pods:

```bash
kubectl get pods -n kdl-lab07 --show-labels
```

Что должно произойти:

- измененный Pod останется существовать;
- ReplicaSet перестанет считать его своим по selector;
- ReplicaSet создаст новый Pod, чтобы снова было 2 подходящих Pods.

Проверьте:

```bash
kubectl get rs nginx-rs -n kdl-lab07
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
kubectl get pods -n kdl-lab07 -l kdl/component=orphan-test
```

Главный вывод:

```text
Selector определяет, какие Pods ReplicaSet считает подходящими.
Если Pod перестал подходить под selector, ReplicaSet создает замену.
```

## Cleanup orphan Pod

Pod с label `kdl/component=orphan-test` больше не нужен.

Сначала проверьте, что будет удалено.

Риск: LOW.

```bash
kubectl get pods -n kdl-lab07 -l kdl/component=orphan-test
```

Удалите только этот Pod.

Риск: MEDIUM.

```bash
kubectl delete pod -n kdl-lab07 -l kdl/component=orphan-test
```

Проверьте:

```bash
kubectl get pods -n kdl-lab07 --show-labels
```

Должны остаться только Pods, которыми управляет ReplicaSet.

## Почему ReplicaSet обычно не создают вручную

ReplicaSet важен для понимания Kubernetes controllers, но в обычной работе его редко создают напрямую.

Обычно используют Deployment.

Почему:

- Deployment управляет ReplicaSet;
- Deployment умеет rolling update;
- Deployment хранит историю rollout;
- Deployment поддерживает rollback;
- при обновлении Pod template Deployment создает новый ReplicaSet.

Упрощенно:

```text
Deployment
  -> ReplicaSet
    -> Pods
```

ReplicaSet - важный внутренний слой. Но для stateless-приложений обычно создают Deployment.

## Cleanup ReplicaSet

Если вы переходите к следующей части про Deployment, лучше удалить ReplicaSet, чтобы его Pods не мешали.

Сначала проверьте, что будет удалено.

Риск: LOW.

```bash
kubectl get rs nginx-rs -n kdl-lab07
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Удаление ReplicaSet.

Риск: MEDIUM.

Что будет изменено: будет удален ReplicaSet `nginx-rs` и Pods, которыми он управляет.

```bash
kubectl delete rs nginx-rs -n kdl-lab07
```

Проверьте:

```bash
kubectl get rs -n kdl-lab07
kubectl get pods -n kdl-lab07 -l kdl/component=replicaset
```

Не удаляйте namespace `kdl-lab07`, если продолжаете Lab 07.

## Инженерная заметка / Вопрос с собеседований

Вопрос: что такое ReplicaSet?

Краткий ответ: ReplicaSet - это Kubernetes controller, который поддерживает заданное количество Pods. Он использует selector, чтобы определить, какие Pods относятся к нему, и Pod template, чтобы создавать новые Pods при необходимости.

Как это видно в этой лабе: мы создали ReplicaSet с `replicas: 2`. Kubernetes создал два Pods. Когда один Pod был удален, ReplicaSet создал новый Pod и снова вернул количество подходящих Pods к двум.

Что важно помнить: ReplicaSet управляет Pods через selector и labels, а не через ручной список имен Pod.

## Инженерная заметка / Вопрос с собеседований

Вопрос: зачем в ReplicaSet нужны `selector` и `template`?

Краткий ответ: `selector` определяет, какие Pods ReplicaSet считает своими. `template` описывает, какие Pods нужно создавать, если подходящих Pods меньше, чем указано в `replicas`.

Как это видно в этой лабе: selector выбирал Pods с labels `app=nginx` и `kdl/component=replicaset`. Когда мы изменили label у одного Pod, он перестал подходить под selector, и ReplicaSet создал новый Pod.

Что важно помнить: labels в `template.metadata.labels` должны соответствовать `spec.selector.matchLabels`, иначе ReplicaSet не сможет корректно управлять Pods.

## Инженерная заметка / Вопрос с собеседований

Вопрос: почему ReplicaSet обычно не создают вручную?

Краткий ответ: ReplicaSet умеет поддерживать количество Pods, но не дает удобного управления обновлениями приложения. Для stateless-приложений обычно создают Deployment, который управляет ReplicaSet и добавляет rollout, rollback и историю обновлений.

Как это видно в этой лабе: ReplicaSet хорошо восстановил удаленный Pod, но для обновления версии приложения удобнее использовать Deployment, который будет следующим объектом в Lab 07.

Что важно помнить: на собеседовании важно объяснить цепочку: Deployment управляет ReplicaSet, ReplicaSet управляет Pods.

## Контрольные вопросы

Ответьте в отчете:

1. Что такое ReplicaSet?
2. Чем ReplicaSet отличается от одиночного Pod?
3. Какие три ключевые части есть в `spec` ReplicaSet?
4. Что означает `spec.replicas`?
5. Что означает `spec.selector`?
6. Что означает `spec.template`?
7. Почему selector должен совпадать с labels в Pod template?
8. Как ReplicaSet понимает, какие Pods относятся к нему?
9. Что произойдет, если удалить Pod, которым управляет ReplicaSet?
10. Что произойдет, если изменить label у Pod так, что он перестанет подходить под selector?
11. Почему изменение через `kubectl scale` нужно переносить обратно в manifest?
12. Что показывает `kubectl get rs`?
13. Где посмотреть ownerReferences у Pod?
14. Почему ReplicaSet обычно не создают вручную?
15. Какая связь между Deployment, ReplicaSet и Pod?

## Что добавить в отчет

Добавьте в отчет по Lab 07:

```text
Файл: 04-replicaset.md

Что было сделано:
- создан ReplicaSet nginx-rs через manifest;
- проверены replicas, selector и template;
- проверены Pods, созданные ReplicaSet;
- проверен ownerReferences у Pod;
- удален один Pod и проверено самовосстановление;
- выполнено масштабирование через kubectl scale;
- изменение replicas перенесено обратно в manifest;
- выполнено масштабирование вниз через manifest;
- изменен label у Pod и проверено поведение selector;
- удален orphan Pod;
- удален ReplicaSet перед переходом к Deployment.

Команды:
- kubectl explain replicaset
- kubectl explain replicaset.spec
- kubectl apply --dry-run=server -f manifests/replicaset-nginx.yaml
- kubectl apply -f manifests/replicaset-nginx.yaml
- kubectl get rs -n kdl-lab07
- kubectl get pods -n kdl-lab07 -l kdl/component=replicaset -o wide
- kubectl get pod <pod-name> -n kdl-lab07 -o yaml
- kubectl describe rs nginx-rs -n kdl-lab07
- kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
- kubectl delete pod <pod-name> -n kdl-lab07
- kubectl scale rs nginx-rs -n kdl-lab07 --replicas=3
- kubectl diff -f manifests/replicaset-nginx.yaml
- kubectl label pod <pod-name> -n kdl-lab07 kdl/component=orphan-test --overwrite
- kubectl delete pod -n kdl-lab07 -l kdl/component=orphan-test
- kubectl delete rs nginx-rs -n kdl-lab07

Вывод:
- своими словами объяснить, как ReplicaSet поддерживает desired number of Pods и почему для приложения дальше нужен Deployment.
```
