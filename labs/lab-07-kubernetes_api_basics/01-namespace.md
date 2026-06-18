# Lab 07.01 - Namespace

В этой части Lab 07 мы разберем первый объект Kubernetes - `Namespace`.

Namespace нужен, чтобы логически отделять одни Kubernetes objects от других внутри одного cluster. В этой лабораторной мы будем использовать отдельный namespace `kdl-lab07`, чтобы все учебные объекты были собраны в одном месте и не смешивались с системными объектами.

## Что изучаем

В этой части вы разберете:

* что такое Namespace;
* зачем Kubernetes использует namespaces;
* какие объекты являются namespaced;
* какие объекты живут на уровне всего cluster;
* как создать Namespace через YAML manifest;
* как посмотреть `metadata`, `spec` и `status`;
* как добавить label;
* как добавить annotation;
* как посмотреть events;
* как безопасно удалить учебный namespace.

## Что такое Namespace

Namespace - это логическая область внутри Kubernetes cluster.

Он помогает разделять объекты по проектам, окружениям, командам или лабораторным работам.

Например:

```text
default
kube-system
kube-public
kube-node-lease
kdl-lab07
```

Важно: Namespace не создает отдельный Kubernetes cluster. Все namespaces живут внутри одного cluster и используют один control plane.

## Что Namespace дает

Namespace помогает:

* группировать объекты;
* использовать одинаковые имена объектов в разных namespaces;
* ограничивать область команд `kubectl`;
* отделять учебные объекты от системных;
* позже применять ResourceQuota, LimitRange, RBAC и NetworkPolicy на уровне namespace.

Пример:

```text
Namespace: dev
  Pod: nginx

Namespace: prod
  Pod: nginx
```

Это два разных объекта, потому что они находятся в разных namespaces.

## Что Namespace не дает

Namespace сам по себе не является полноценной границей безопасности.

Он не изолирует:

* Linux processes;
* network traffic сам по себе;
* container runtime;
* nodes;
* control plane;
* доступ пользователя без RBAC;
* ресурсы без ResourceQuota;
* traffic без NetworkPolicy.

То есть Namespace - это область имен и логическая группировка. Для безопасности нужны дополнительные механизмы: RBAC, NetworkPolicy, Pod Security Admission, quotas и другие политики.

## Namespaced и cluster-wide objects

Не все Kubernetes objects находятся внутри namespace.

Namespaced objects:

```text
Pod
Service
Deployment
ReplicaSet
ConfigMap
Secret
ServiceAccount
```

Cluster-wide objects:

```text
Node
Namespace
PersistentVolume
StorageClass
ClusterRole
ClusterRoleBinding
CustomResourceDefinition
```

Проверить это можно командой:

```bash
kubectl api-resources
```

Чтобы увидеть, какие ресурсы являются namespaced:

```bash
kubectl api-resources --namespaced=true
```

Чтобы увидеть cluster-wide ресурсы:

```bash
kubectl api-resources --namespaced=false
```

## Текущий контекст kubectl

Перед началом проверьте, к какому cluster подключен `kubectl`.

Команда безопасная.

Риск: LOW.

```bash
kubectl config current-context
```

Проверьте, какой namespace используется по умолчанию в текущем context:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'; echo
```

Если команда ничего не вывела, значит для текущего context namespace явно не задан, и `kubectl` обычно будет использовать namespace `default`.

## Проверка существующих namespaces

Посмотрите namespaces в cluster.

Риск: LOW.

```bash
kubectl get namespaces
```

Сокращенная форма:

```bash
kubectl get ns
```

Ожидаемо вы увидите системные namespaces, например:

```text
default
kube-node-lease
kube-public
kube-system
```

Если `kdl-lab07` уже существует, не удаляйте его сразу. Сначала посмотрите, что внутри:

```bash
kubectl get all -n kdl-lab07
```

## Подготовка manifest-файла

Создайте файл:

```text
manifests/namespace.yaml
```

Содержимое:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kdl-lab07
  labels:
    app.kubernetes.io/name: kdl-lab07
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
  annotations:
    kdl/description: "Kubernetes Deep Lab 07 namespace"
```

Разбор полей:

| Поле                   | Значение                                         |
| ---------------------- | ------------------------------------------------ |
| `apiVersion: v1`       | Namespace относится к core API group версии `v1` |
| `kind: Namespace`      | Тип создаваемого объекта                         |
| `metadata.name`        | Имя namespace                                    |
| `metadata.labels`      | Метки для поиска и группировки                   |
| `metadata.annotations` | Дополнительные описательные данные               |

Обратите внимание: у Namespace нет поля `metadata.namespace`, потому что сам Namespace является cluster-wide объектом. Он не находится внутри другого namespace.

## Проверка manifest перед применением

Сначала проверьте, какой объект будет отправлен в Kubernetes API.

Риск: LOW.

```bash
kubectl apply --dry-run=client -f manifests/namespace.yaml
```

Что означает:

* `apply` - применить декларативную конфигурацию;
* `--dry-run=client` - проверить объект на стороне клиента, не отправляя изменение в API Server;
* `-f` - путь к manifest-файлу.

Теперь выполните server-side dry-run.

Риск: LOW.

```bash
kubectl apply --dry-run=server -f manifests/namespace.yaml
```

Разница:

| Режим              | Что проверяет                                              |
| ------------------ | ---------------------------------------------------------- |
| `--dry-run=client` | Проверка на стороне `kubectl` без сохранения объекта       |
| `--dry-run=server` | Запрос отправляется в API Server, но объект не сохраняется |

Server-side dry-run полезнее, потому что API Server проверяет объект ближе к реальному созданию.

## Создание Namespace

Теперь примените manifest.

Риск: MEDIUM.

Что изменится: в cluster будет создан или обновлен Namespace `kdl-lab07`.

```bash
kubectl apply -f manifests/namespace.yaml
```

Ожидаемый результат:

```text
namespace/kdl-lab07 created
```

Если namespace уже существовал и manifest не изменился, результат может быть:

```text
namespace/kdl-lab07 unchanged
```

Это нормальное поведение декларативного подхода.

## Проверка созданного Namespace

Посмотрите namespace:

Риск: LOW.

```bash
kubectl get namespace kdl-lab07
```

Сокращенная форма:

```bash
kubectl get ns kdl-lab07
```

Посмотрите labels:

```bash
kubectl get namespace kdl-lab07 --show-labels
```

Посмотрите объект в YAML:

```bash
kubectl get namespace kdl-lab07 -o yaml
```

В выводе найдите:

```text
metadata
spec
status
```

## Разбор metadata, spec и status

В объекте Namespace важно увидеть три уровня.

`metadata` - служебная и пользовательская информация об объекте:

```text
name
uid
resourceVersion
creationTimestamp
labels
annotations
managedFields
```

`spec` - желаемое состояние объекта.

У Namespace `spec` обычно небольшой. При удалении namespace там может появиться информация, связанная с финализацией.

`status` - текущее состояние namespace.

Обычно вы увидите:

```yaml
status:
  phase: Active
```

`Active` означает, что namespace доступен для использования.

При удалении namespace может перейти в состояние:

```text
Terminating
```

## Практика с kubectl describe

Посмотрите подробное описание namespace.

Риск: LOW.

```bash
kubectl describe namespace kdl-lab07
```

Обратите внимание на:

* Name;
* Labels;
* Annotations;
* Status;
* Resource Quotas;
* LimitRanges.

Если ResourceQuota и LimitRange не настроены, это нормально. Мы разберем их позже в отдельных labs.

## Добавление label через kubectl

Сейчас добавим label через imperative-команду, чтобы увидеть разницу между изменением через команду и изменением через manifest.

Риск: MEDIUM.

Что изменится: у Namespace появится новый label.

```bash
kubectl label namespace kdl-lab07 kdl/phase=api-basics
```

Проверьте:

```bash
kubectl get namespace kdl-lab07 --show-labels
```

Вы должны увидеть label:

```text
kdl/phase=api-basics
```

## Важный вывод про imperative-команды

Мы добавили label командой:

```bash
kubectl label namespace kdl-lab07 kdl/phase=api-basics
```

Но в файле `manifests/namespace.yaml` этого label пока нет.

Это важный момент.

Состояние в cluster изменилось, а состояние в Git/manifests - нет.

Поэтому в production-like workflow такие изменения нужно переносить обратно в manifest, иначе появится расхождение между Git и cluster.

## Исправление manifest

Добавьте label в `manifests/namespace.yaml`.

Должно получиться так:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kdl-lab07
  labels:
    app.kubernetes.io/name: kdl-lab07
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/phase: api-basics
  annotations:
    kdl/description: "Kubernetes Deep Lab 07 namespace"
```

Проверьте diff перед применением.

Риск: LOW.

```bash
kubectl diff -f manifests/namespace.yaml
```

Если diff пустой, значит live-состояние и manifest совпадают.

Если diff показывает изменения, внимательно прочитайте их.

Примените manifest:

Риск: MEDIUM.

```bash
kubectl apply -f manifests/namespace.yaml
```

## Добавление annotation

Annotation используется для дополнительных описательных данных. В отличие от labels, annotations обычно не используются для выбора объектов через selector.

Добавьте annotation командой:

Риск: MEDIUM.

```bash
kubectl annotate namespace kdl-lab07 kdl/owner="student"
```

Проверьте:

```bash
kubectl get namespace kdl-lab07 -o yaml
```

Найдите:

```yaml
annotations:
  kdl/owner: student
```

Теперь перенесите annotation в manifest, чтобы файл снова соответствовал live-состоянию.

Итоговый `manifests/namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kdl-lab07
  labels:
    app.kubernetes.io/name: kdl-lab07
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/phase: api-basics
  annotations:
    kdl/description: "Kubernetes Deep Lab 07 namespace"
    kdl/owner: "student"
```

Проверьте diff:

```bash
kubectl diff -f manifests/namespace.yaml
```

Примените:

```bash
kubectl apply -f manifests/namespace.yaml
```

## Проверка objects внутри namespace

Сейчас namespace создан, но внутри него еще нет приложений.

Проверьте:

Риск: LOW.

```bash
kubectl get all -n kdl-lab07
```

Ожидаемый результат:

```text
No resources found in kdl-lab07 namespace.
```

Это нормально.

Namespace существует, но workloads внутри него мы создадим в следующих частях Lab 07.

## Проверка events в namespace

Посмотрите events внутри namespace:

Риск: LOW.

```bash
kubectl get events -n kdl-lab07
```

Скорее всего, вы увидите:

```text
No resources found in kdl-lab07 namespace.
```

Это нормально.

Почему так: сам Namespace является cluster-wide объектом. Его события не всегда будут видны внутри этого namespace так же, как события Pod или Deployment. Подробную работу с events мы разберем позже, когда появятся Pods и ошибки запуска containers.

## Проверка namespaced и cluster-wide поведения

Попробуйте получить namespace с флагом `-n`.

Риск: LOW.

```bash
kubectl get namespace kdl-lab07 -n default
```

Команда сработает, потому что Namespace - cluster-wide объект. Флаг `-n default` здесь не делает Namespace namespaced-объектом.

Теперь сравните с командой для Pods:

```bash
kubectl get pods -n kdl-lab07
```

Pods являются namespaced objects, поэтому для них namespace имеет значение.

## Что важно понять

Namespace - это не "папка" в файловой системе и не отдельная VM.

Правильная модель:

```text
Cluster
  -> Namespace
    -> namespaced objects
```

Но:

```text
Cluster
  -> cluster-wide objects
```

Пример:

```text
Pod lives in Namespace
Service lives in Namespace
Deployment lives in Namespace

Node does not live in Namespace
Namespace does not live in Namespace
StorageClass does not live in Namespace
```

## Cleanup

Удалять namespace сейчас не нужно, если вы продолжаете Lab 07. Он понадобится для следующих частей.

Если нужно полностью очистить Lab 07, сначала выполните dry-run проверки.

Проверка объектов внутри namespace.

Риск: LOW.

```bash
kubectl get all -n kdl-lab07
```

Проверка namespace:

```bash
kubectl get namespace kdl-lab07
```

Удаление namespace удалит все namespaced objects внутри него.

Риск: HIGH.

Что будет изменено: Kubernetes удалит Namespace `kdl-lab07` и все объекты внутри него.

Команда удаления:

```bash
kubectl delete namespace kdl-lab07
```

Не выполняйте эту команду, если планируете продолжать Lab 07.

Rollback: если namespace удален, его можно создать заново через:

```bash
kubectl apply -f manifests/namespace.yaml
```

Но все объекты внутри namespace придется создавать заново из соответствующих manifest-файлов.

## Проверка после cleanup

Если namespace был удален, проверьте:

```bash
kubectl get namespace kdl-lab07
```

Ожидаемый результат:

```text
Error from server (NotFound): namespaces "kdl-lab07" not found
```

Если namespace завис в состоянии `Terminating`, не пытайтесь сразу удалять finalizers вручную. Это отдельная troubleshooting-ситуация, которую нужно разбирать аккуратно.

## Контрольные вопросы

Ответьте в отчете:

1. Что такое Namespace в Kubernetes?
2. Namespace создает отдельный cluster или логическую область внутри cluster?
3. Почему Namespace сам по себе не является полноценной границей безопасности?
4. Какие объекты являются namespaced?
5. Какие объекты являются cluster-wide?
6. Почему у Namespace нет поля `metadata.namespace`?
7. Чем labels отличаются от annotations?
8. Почему изменение через `kubectl label` нужно перенести обратно в manifest?
9. Что означает `status.phase: Active`?
10. Почему удаление namespace считается HIGH-risk операцией?

## Что добавить в отчет

Добавьте в отчет по Lab 07:

```text
Файл: 01-namespace.md

Что было сделано:
- создан Namespace kdl-lab07;
- проверены metadata, spec, status;
- добавлен label;
- добавлена annotation;
- проверено отличие namespaced и cluster-wide objects.

Команды:
- kubectl get namespaces
- kubectl apply --dry-run=client -f manifests/namespace.yaml
- kubectl apply --dry-run=server -f manifests/namespace.yaml
- kubectl apply -f manifests/namespace.yaml
- kubectl get namespace kdl-lab07 -o yaml
- kubectl describe namespace kdl-lab07
- kubectl label namespace kdl-lab07 kdl/phase=api-basics
- kubectl annotate namespace kdl-lab07 kdl/owner="student"
- kubectl diff -f manifests/namespace.yaml
- kubectl api-resources --namespaced=true
- kubectl api-resources --namespaced=false

Вывод:
- своими словами объяснить, зачем нужен Namespace и что он не изолирует.
```
