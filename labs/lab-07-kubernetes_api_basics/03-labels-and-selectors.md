# Lab 07.03 - Labels and selectors

В этой части Lab 07 мы разберем `labels` и `selectors`.

Labels и selectors - один из самых важных механизмов Kubernetes. Через них Kubernetes objects связываются друг с другом.

Service не хранит список Pod names вручную. Он выбирает Pods по labels.

ReplicaSet не управляет Pods по именам. Он выбирает Pods по labels.

Deployment через ReplicaSet тоже опирается на labels и selectors.

Поэтому перед ReplicaSet, Deployment и Service нужно отдельно понять labels и selectors.

## Что изучаем

В этой части вы разберете:

* что такое label;
* что такое selector;
* чем labels отличаются от annotations;
* как посмотреть labels у объектов;
* как выбрать Pods по label;
* как использовать equality-based selector;
* как использовать set-based selector;
* как добавить label через manifest;
* как добавить label через `kubectl label`;
* почему imperative-изменение нужно переносить обратно в manifest;
* как удалить label;
* почему неправильные labels ломают связь между объектами.

## Что такое label

Label - это пара `key=value`, которая добавляется в `metadata.labels` Kubernetes object.

Пример:

```yaml id="03labels-001"
metadata:
  labels:
    app: nginx
    environment: lab
    tier: frontend
```

Labels нужны для:

* группировки объектов;
* поиска объектов;
* выбора объектов через selector;
* связи Service с Pods;
* связи ReplicaSet с Pods;
* организации объектов для инструментов и людей.

Важно: label должен описывать идентифицирующий признак объекта.

Например:

```text id="03labels-002"
app=nginx
tier=frontend
environment=lab
```

## Что такое selector

Selector - это правило выбора объектов по labels.

Пример:

```bash id="03labels-003"
kubectl get pods -l app=nginx
```

Эта команда означает:

```text id="03labels-004"
показать Pods, у которых есть label app=nginx
```

Selector не выбирает объект по имени. Он выбирает группу объектов по labels.

## Labels и annotations

Labels и annotations оба находятся в `metadata`, но используются по-разному.

| Механизм      | Для чего нужен                                                          |
| ------------- | ----------------------------------------------------------------------- |
| `labels`      | Для выбора, группировки и связи объектов                                |
| `annotations` | Для дополнительной информации, которую обычно не используют в selectors |

Пример label:

```yaml id="03labels-005"
labels:
  app: nginx
```

Пример annotation:

```yaml id="03labels-006"
annotations:
  kdl/description: "test nginx pod for labels lab"
```

Правило:

```text id="03labels-007"
Нужно выбирать объект через selector - используйте label.
Нужно сохранить описание, ссылку, заметку или служебную информацию - используйте annotation.
```

## Проверка namespace

Все объекты создаем в namespace `kdl-lab07`.

Риск: LOW.

```bash id="03labels-008"
kubectl get namespace kdl-lab07
```

Если namespace отсутствует, вернитесь к `01-namespace.md`.

## Подготовка manifest-файла

Создайте файл:

```text id="03labels-009"
manifests/pods-labels.yaml
```

Содержимое:

```yaml id="03labels-010"
apiVersion: v1
kind: Pod
metadata:
  name: labels-nginx-a
  namespace: kdl-lab07
  labels:
    app: nginx
    environment: lab
    tier: frontend
    track: stable
    kdl/lab: "07"
    kdl/component: labels
  annotations:
    kdl/description: "First nginx Pod for labels and selectors practice"
spec:
  containers:
    - name: nginx
      image: nginx:1.27.5
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: labels-nginx-b
  namespace: kdl-lab07
  labels:
    app: nginx
    environment: lab
    tier: frontend
    track: canary
    kdl/lab: "07"
    kdl/component: labels
  annotations:
    kdl/description: "Second nginx Pod for labels and selectors practice"
spec:
  containers:
    - name: nginx
      image: nginx:1.27.5
      ports:
        - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: labels-busybox-a
  namespace: kdl-lab07
  labels:
    app: busybox
    environment: lab
    tier: tools
    track: stable
    kdl/lab: "07"
    kdl/component: labels
  annotations:
    kdl/description: "BusyBox Pod for selectors practice"
spec:
  containers:
    - name: busybox
      image: busybox:1.37.0
      command: ["sh", "-c", "sleep 3600"]
```

В этом файле описаны три Pod:

| Pod                | app       | tier       | track    |
| ------------------ | --------- | ---------- | -------- |
| `labels-nginx-a`   | `nginx`   | `frontend` | `stable` |
| `labels-nginx-b`   | `nginx`   | `frontend` | `canary` |
| `labels-busybox-a` | `busybox` | `tools`    | `stable` |

Зачем так: нам нужны разные labels, чтобы проверить выборку через selectors.

## Dry-run перед созданием

Client-side dry-run.

Риск: LOW.

```bash id="03labels-011"
kubectl apply --dry-run=client -f manifests/pods-labels.yaml
```

Server-side dry-run.

Риск: LOW.

```bash id="03labels-012"
kubectl apply --dry-run=server -f manifests/pods-labels.yaml
```

Если dry-run прошел успешно, примените manifest.

## Создание Pods

Риск: MEDIUM.

Что изменится: в namespace `kdl-lab07` будут созданы три Pods.

```bash id="03labels-013"
kubectl apply -f manifests/pods-labels.yaml
```

Ожидаемый результат:

```text id="03labels-014"
pod/labels-nginx-a created
pod/labels-nginx-b created
pod/labels-busybox-a created
```

## Проверка Pods

Риск: LOW.

```bash id="03labels-015"
kubectl get pods -n kdl-lab07
```

Дождитесь готовности Pods:

```bash id="03labels-016"
kubectl wait --for=condition=Ready pod/labels-nginx-a -n kdl-lab07 --timeout=120s
kubectl wait --for=condition=Ready pod/labels-nginx-b -n kdl-lab07 --timeout=120s
kubectl wait --for=condition=Ready pod/labels-busybox-a -n kdl-lab07 --timeout=120s
```

Если какой-то Pod не стал Ready, проверьте:

```bash id="03labels-017"
kubectl describe pod <pod-name> -n kdl-lab07
kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
```

## Просмотр labels

Посмотрите Pods вместе с labels.

Риск: LOW.

```bash id="03labels-018"
kubectl get pods -n kdl-lab07 --show-labels
```

Вы должны увидеть labels для каждого Pod.

Более удобный вывод по отдельным labels:

```bash id="03labels-019"
kubectl get pods -n kdl-lab07 -L app,tier,track
```

Что означает:

* `-L app,tier,track` добавляет колонки с выбранными labels;
* если у объекта нет такого label, колонка будет пустой.

## Equality-based selector

Equality-based selector выбирает объекты по равенству или неравенству label.

Выбрать Pods с `app=nginx`.

Риск: LOW.

```bash id="03labels-020"
kubectl get pods -n kdl-lab07 -l app=nginx
```

Ожидаемо будут выбраны:

```text id="03labels-021"
labels-nginx-a
labels-nginx-b
```

Выбрать Pods с `app=busybox`.

```bash id="03labels-022"
kubectl get pods -n kdl-lab07 -l app=busybox
```

Ожидаемо будет выбран:

```text id="03labels-023"
labels-busybox-a
```

Выбрать Pods, у которых `track` не равен `canary`.

```bash id="03labels-024"
kubectl get pods -n kdl-lab07 -l track!=canary
```

Обратите внимание: такой selector может выбрать больше объектов, чем ожидается, если в namespace есть другие Pods без label `track`.

Поэтому в учебных и production-командах selector лучше делать достаточно точным.

Например:

```bash id="03labels-025"
kubectl get pods -n kdl-lab07 -l kdl/component=labels,track!=canary
```

## Несколько условий в selector

Выберите только stable nginx Pod.

Риск: LOW.

```bash id="03labels-026"
kubectl get pods -n kdl-lab07 -l app=nginx,track=stable
```

Ожидаемо будет выбран:

```text id="03labels-027"
labels-nginx-a
```

Важно: условия через запятую работают как logical AND.

То есть:

```text id="03labels-028"
app=nginx,track=stable
```

означает:

```text id="03labels-029"
app должен быть nginx
и
track должен быть stable
```

## Set-based selector

Set-based selector выбирает объекты по множеству значений.

Выбрать Pods, у которых `track` входит в `stable` или `canary`.

Риск: LOW.

```bash id="03labels-030"
kubectl get pods -n kdl-lab07 -l 'track in (stable,canary)'
```

Выбрать Pods, у которых `tier` не входит в `tools`.

```bash id="03labels-031"
kubectl get pods -n kdl-lab07 -l 'tier notin (tools)'
```

Ожидаемо будут выбраны nginx Pods.

Проверить существование label `track`:

```bash id="03labels-032"
kubectl get pods -n kdl-lab07 -l track
```

Выбрать Pods, у которых нет label `debug`.

```bash id="03labels-033"
kubectl get pods -n kdl-lab07 -l '!debug'
```

Важно: в shell selector с пробелами и скобками лучше брать в одинарные кавычки.

## Получение имен выбранных объектов

Иногда нужно получить только имена объектов.

Риск: LOW.

```bash id="03labels-034"
kubectl get pods -n kdl-lab07 -l app=nginx -o name
```

Ожидаемо:

```text id="03labels-035"
pod/labels-nginx-a
pod/labels-nginx-b
```

Это удобно для диагностики и скриптов, но в production-скриптах нужно быть осторожным с selector, чтобы не выбрать лишние объекты.

## Просмотр labels через jsonpath

Выведите labels конкретного Pod.

Риск: LOW.

```bash id="03labels-036"
kubectl get pod labels-nginx-a -n kdl-lab07 -o jsonpath='{.metadata.labels}'; echo
```

Выведите конкретный label:

```bash id="03labels-037"
kubectl get pod labels-nginx-a -n kdl-lab07 -o jsonpath='{.metadata.labels.track}'; echo
```

Ожидаемо:

```text id="03labels-038"
stable
```

## Добавление label через kubectl label

Добавьте label `debug=true` к Pod `labels-nginx-a`.

Риск: MEDIUM.

Что изменится: у Pod `labels-nginx-a` появится новый label.

```bash id="03labels-039"
kubectl label pod labels-nginx-a -n kdl-lab07 debug=true
```

Проверьте:

```bash id="03labels-040"
kubectl get pod labels-nginx-a -n kdl-lab07 --show-labels
```

Выберите Pods с label `debug=true`.

```bash id="03labels-041"
kubectl get pods -n kdl-lab07 -l debug=true
```

Ожидаемо будет выбран:

```text id="03labels-042"
labels-nginx-a
```

## Важный вывод про imperative-изменение

Мы добавили label командой:

```bash id="03labels-043"
kubectl label pod labels-nginx-a -n kdl-lab07 debug=true
```

Но в файле `manifests/pods-labels.yaml` этого label нет.

Это означает:

```text id="03labels-044"
live-состояние cluster изменилось
manifest в Git не изменился
```

В production-like workflow это плохо, потому что появляется расхождение между тем, что записано в manifest, и тем, что реально находится в cluster.

## Перенос label в manifest

Добавьте label в первый Pod `labels-nginx-a` в файле `manifests/pods-labels.yaml`.

Должно получиться так:

```yaml id="03labels-045"
metadata:
  name: labels-nginx-a
  namespace: kdl-lab07
  labels:
    app: nginx
    environment: lab
    tier: frontend
    track: stable
    debug: "true"
    kdl/lab: "07"
    kdl/component: labels
```

Проверьте diff.

Риск: LOW.

```bash id="03labels-046"
kubectl diff -f manifests/pods-labels.yaml
```

Если diff пустой, значит live-состояние и manifest совпадают.

Примените manifest.

Риск: MEDIUM.

```bash id="03labels-047"
kubectl apply -f manifests/pods-labels.yaml
```

## Изменение label

Теперь изменим label `track` у `labels-nginx-b`.

Сейчас у него:

```text id="03labels-048"
track=canary
```

Изменим на:

```text id="03labels-049"
track=stable
```

Сделаем это сначала через imperative-команду.

Риск: MEDIUM.

Что изменится: label `track` у Pod `labels-nginx-b` будет перезаписан.

```bash id="03labels-050"
kubectl label pod labels-nginx-b -n kdl-lab07 track=stable --overwrite
```

Проверьте:

```bash id="03labels-051"
kubectl get pods -n kdl-lab07 -L app,tier,track
```

Теперь selector:

```bash id="03labels-052"
kubectl get pods -n kdl-lab07 -l app=nginx,track=stable
```

должен выбрать оба nginx Pods:

```text id="03labels-053"
labels-nginx-a
labels-nginx-b
```

## Почему нужен `--overwrite`

Если label уже существует, `kubectl label` без `--overwrite` не перезапишет его.

Это защита от случайного изменения label.

Пример команды, которая должна завершиться ошибкой:

```bash id="03labels-054"
kubectl label pod labels-nginx-b -n kdl-lab07 track=canary
```

Ожидаемо Kubernetes сообщит, что label уже имеет значение и для перезаписи нужен `--overwrite`.

## Перенос изменения в manifest

Теперь обновите `manifests/pods-labels.yaml`.

Для Pod `labels-nginx-b` замените:

```yaml id="03labels-055"
track: canary
```

на:

```yaml id="03labels-056"
track: stable
```

Проверьте diff.

Риск: LOW.

```bash id="03labels-057"
kubectl diff -f manifests/pods-labels.yaml
```

Примените:

Риск: MEDIUM.

```bash id="03labels-058"
kubectl apply -f manifests/pods-labels.yaml
```

## Удаление label

Удалим label `debug` у `labels-nginx-a`.

Риск: MEDIUM.

Что изменится: у Pod `labels-nginx-a` будет удален label `debug`.

```bash id="03labels-059"
kubectl label pod labels-nginx-a -n kdl-lab07 debug-
```

Проверьте:

```bash id="03labels-060"
kubectl get pod labels-nginx-a -n kdl-lab07 --show-labels
```

Selector больше не должен выбрать этот Pod:

```bash id="03labels-061"
kubectl get pods -n kdl-lab07 -l debug=true
```

Ожидаемо:

```text id="03labels-062"
No resources found in kdl-lab07 namespace.
```

Теперь удалите `debug: "true"` из manifest-файла, чтобы manifest снова совпадал с live-состоянием.

Проверьте diff:

```bash id="03labels-063"
kubectl diff -f manifests/pods-labels.yaml
```

Примените:

```bash id="03labels-064"
kubectl apply -f manifests/pods-labels.yaml
```

## Проверка annotations

Посмотрите annotations у Pod.

Риск: LOW.

```bash id="03labels-065"
kubectl get pod labels-nginx-a -n kdl-lab07 -o jsonpath='{.metadata.annotations}'; echo
```

Попробуйте выбрать Pod по annotation через `-l`.

Риск: LOW.

```bash id="03labels-066"
kubectl get pods -n kdl-lab07 -l 'kdl/description=First nginx Pod for labels and selectors practice'
```

Ожидаемо selector ничего не выберет, потому что `-l` работает с labels, а не с annotations.

Главный вывод:

```text id="03labels-067"
labels используются для выбора объектов
annotations не используются selector-механизмом
```

## Ситуация с неправильным selector

Теперь посмотрим, как selector может не выбрать ничего.

Риск: LOW.

```bash id="03labels-068"
kubectl get pods -n kdl-lab07 -l app=redis
```

Ожидаемо:

```text id="03labels-069"
No resources found in kdl-lab07 namespace.
```

Это не ошибка Kubernetes. Это значит, что в namespace нет Pods с label:

```text id="03labels-070"
app=redis
```

Такой же тип проблемы позже может быть у Service или ReplicaSet: объект существует, но selector не находит подходящие Pods.

## Проверка events

Посмотрите events.

Риск: LOW.

```bash id="03labels-071"
kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
```

Изменение labels обычно не создает таких же заметных runtime-events, как запуск container или ошибка скачивания image.

Это важный нюанс: не все изменения объекта приводят к полезным events. Поэтому labels нужно проверять через `kubectl get --show-labels`, `-L`, `jsonpath` и YAML.

## Cleanup

Если вы переходите к следующей части Lab 07, можно удалить Pods из этой части, чтобы они не мешали ReplicaSet и Deployment.

Сначала посмотрите, что будет затронуто selector-командой.

Риск: LOW.

```bash id="03labels-072"
kubectl get pods -n kdl-lab07 -l kdl/component=labels
```

Если вывод содержит только Pods этой части:

```text id="03labels-073"
labels-nginx-a
labels-nginx-b
labels-busybox-a
```

можно удалить их по selector.

Риск: MEDIUM.

Что будет изменено: будут удалены Pods с label `kdl/component=labels`.

```bash id="03labels-074"
kubectl delete pods -n kdl-lab07 -l kdl/component=labels
```

Проверьте:

```bash id="03labels-075"
kubectl get pods -n kdl-lab07 -l kdl/component=labels
```

Ожидаемо:

```text id="03labels-076"
No resources found in kdl-lab07 namespace.
```

Не удаляйте namespace `kdl-lab07`, если продолжаете Lab 07.

## Инженерная заметка / Вопрос с собеседований

Вопрос: что такое labels в Kubernetes?

Краткий ответ: labels - это пары `key=value` в `metadata.labels`, которые используются для группировки, поиска и выбора Kubernetes objects. Labels нужны не только людям, но и другим объектам Kubernetes. Например, Service и ReplicaSet выбирают Pods через selectors по labels.

Как это видно в этой лабе: мы создали три Pods с разными labels и выбирали их командами `kubectl get pods -l app=nginx`, `kubectl get pods -l app=nginx,track=stable`, `kubectl get pods -l 'track in (stable,canary)'`.

Что важно помнить: labels должны описывать идентифицирующие признаки объекта. Если label выбран неправильно или потерян, другие объекты могут перестать находить этот Pod.

## Инженерная заметка / Вопрос с собеседований

Вопрос: что такое selector в Kubernetes?

Краткий ответ: selector - это правило выбора объектов по labels. Selector не выбирает объект по имени. Он выбирает один или несколько объектов, у которых labels соответствуют условию.

Как это видно в этой лабе: selector `app=nginx` выбрал два nginx Pods, а selector `app=redis` не выбрал ничего, потому что Pods с таким label не было.

Что важно помнить: неправильный selector - частая причина проблем в Kubernetes. Например, Service может существовать, но не иметь endpoints, если его selector не совпадает с labels Pods.

## Инженерная заметка / Вопрос с собеседований

Вопрос: чем labels отличаются от annotations?

Краткий ответ: labels используются для выбора и группировки объектов. Annotations используются для дополнительной неидентифицирующей информации. Selector работает с labels, но не с annotations.

Как это видно в этой лабе: мы смогли выбрать Pods через `-l app=nginx`, но не смогли выбрать Pod через annotation `kdl/description`.

Что важно помнить: если поле должно участвовать в выборе объектов, это должен быть label. Если нужно сохранить описание, ссылку, JSON, комментарий или данные для внешнего инструмента, это чаще annotation.

## Контрольные вопросы

Ответьте в отчете:

1. Что такое label?
2. Что такое selector?
3. Где в manifest находятся labels?
4. Чем labels отличаются от annotations?
5. Почему selector не выбирает объекты по имени?
6. Что означает selector `app=nginx,track=stable`?
7. Что означает selector `track in (stable,canary)`?
8. Почему selector `track!=canary` может выбрать больше объектов, чем ожидалось?
9. Зачем нужен `--show-labels`?
10. Что делает команда `kubectl get pods -L app,tier,track`?
11. Почему `kubectl label` может создать расхождение между live-состоянием и manifest?
12. Зачем нужен `--overwrite` при изменении существующего label?
13. Как удалить label через `kubectl label`?
14. Почему annotations не подходят для selectors?
15. Как неправильный selector может повлиять на Service или ReplicaSet?

## Что добавить в отчет

Добавьте в отчет по Lab 07:

```text id="03labels-077"
Файл: 03-labels-and-selectors.md

Что было сделано:
- созданы три Pods с разными labels;
- проверены labels через --show-labels;
- проверены labels через -L;
- выполнена выборка через equality-based selectors;
- выполнена выборка через set-based selectors;
- добавлен label через kubectl label;
- изменение перенесено обратно в manifest;
- изменен существующий label через --overwrite;
- удален label;
- проверено отличие labels от annotations;
- показан selector, который не выбирает ни одного объекта.

Команды:
- kubectl apply --dry-run=server -f manifests/pods-labels.yaml
- kubectl apply -f manifests/pods-labels.yaml
- kubectl get pods -n kdl-lab07 --show-labels
- kubectl get pods -n kdl-lab07 -L app,tier,track
- kubectl get pods -n kdl-lab07 -l app=nginx
- kubectl get pods -n kdl-lab07 -l app=nginx,track=stable
- kubectl get pods -n kdl-lab07 -l 'track in (stable,canary)'
- kubectl get pod labels-nginx-a -n kdl-lab07 -o jsonpath='{.metadata.labels}'
- kubectl label pod labels-nginx-a -n kdl-lab07 debug=true
- kubectl label pod labels-nginx-b -n kdl-lab07 track=stable --overwrite
- kubectl label pod labels-nginx-a -n kdl-lab07 debug-
- kubectl get pods -n kdl-lab07 -l app=redis
- kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp

Вывод:
- своими словами объяснить, как labels и selectors связывают Kubernetes objects и почему без этого нельзя понять ReplicaSet, Deployment и Service.

