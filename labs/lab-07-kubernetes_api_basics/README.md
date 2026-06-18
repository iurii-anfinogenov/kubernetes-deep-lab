# Lab 07 - Kubernetes API objects and kubectl workflow

В этой лабораторной работе мы начинаем изучать базовые объекты Kubernetes не как "набор YAML-файлов", а как систему связанных сущностей внутри Kubernetes API.

До этого момента мы подготовили nodes, container runtime, kubeadm, CNI и проверили, что cluster работает. Теперь нужно понять, как Kubernetes принимает описание объекта, сохраняет его состояние и через controllers приводит cluster к нужному результату.

## Цель лабораторной

Цель Lab 07 - понять базовые Kubernetes objects и научиться работать с ними через `kubectl` осознанно.

После этой лабораторной вы должны понимать:

* что такое Kubernetes object;
* из чего состоит manifest;
* чем отличаются `metadata`, `spec` и `status`;
* чем отличаются imperative и declarative подходы;
* когда можно использовать `kubectl run`, `kubectl create`, `kubectl apply`, `kubectl delete`;
* почему в учебной и production-like работе основной путь - YAML + `kubectl apply`;
* что такое Namespace;
* что такое Pod;
* как работают labels и selectors;
* что такое ReplicaSet;
* зачем нужен Deployment;
* что такое Service;
* зачем Kubernetes создает EndpointSlice;
* где смотреть events;

## Главная идея

Kubernetes работает через API objects.

Мы не "запускаем container напрямую". Мы создаем объект в Kubernetes API. После этого разные компоненты Kubernetes начинают приводить cluster к описанному состоянию.

Упрощенно это выглядит так:

```text
manifest.yaml
  -> kubectl
    -> Kubernetes API Server
      -> object stored in etcd
        -> controllers watch API
          -> scheduler chooses node
            -> kubelet starts Pod through container runtime
              -> CNI configures Pod networking
```

Важно: `kubectl` сам не запускает container на node. `kubectl` отправляет запрос в API Server. Остальную работу выполняют Kubernetes components.

## Что такое Kubernetes object

Kubernetes object - это запись в Kubernetes API, которая описывает часть состояния cluster.

Примеры objects:

* Namespace;
* Pod;
* ReplicaSet;
* Deployment;
* Service;
* EndpointSlice;
* ConfigMap;
* Secret;
* Node.

В этой лабораторной мы изучаем только базовые объекты, которые нужны для понимания запуска простого приложения.

## Базовая структура manifest

Большинство Kubernetes manifests имеют похожую структуру:

```yaml
apiVersion: ...
kind: ...
metadata:
  name: ...
  namespace: ...
spec:
  ...
```

Основные поля:

| Поле         | Что означает                                                   |
| ------------ | -------------------------------------------------------------- |
| `apiVersion` | Версия Kubernetes API, через которую создается объект          |
| `kind`       | Тип объекта, например `Pod`, `Deployment`, `Service`           |
| `metadata`   | Имя, namespace, labels, annotations и служебные данные объекта |
| `spec`       | Желаемое состояние объекта                                     |
| `status`     | Текущее состояние объекта, которое заполняет Kubernetes        |

Важная разница:

```text
spec   - что мы хотим получить
status - что реально происходит сейчас
```

Обычно пользователь описывает `spec`, а Kubernetes обновляет `status`.

## Desired state и actual state

Kubernetes работает с двумя состояниями:

```text
desired state - желаемое состояние
actual state  - фактическое состояние
```

Пример:

```text
desired state:
  должно быть 2 replicas nginx

actual state:
  сейчас запущен только 1 Pod
```

Controller видит расхождение и пытается создать второй Pod.

Это называется reconciliation - постоянное сравнение желаемого и фактического состояния.

## Почему не начинаем сразу с Deployment

В реальной работе приложения чаще всего запускают через Deployment. Но если сразу начать с Deployment, студент видит только один YAML-файл и не понимает, что Kubernetes создает внутри.

Поэтому в этой лабораторной мы идем по порядку:

```text
Namespace
  -> Pod
    -> labels/selectors
      -> ReplicaSet
        -> Deployment
          -> Service
            -> EndpointSlice
              -> events/logs/troubleshooting
```

Такой порядок нужен, чтобы увидеть, как один объект связан с другим.

## Как мы будем работать с kubectl

`kubectl` - это command-line tool для общения с Kubernetes API Server.

В этой лабораторной мы будем использовать несколько типов команд.

## Read-only команды

Read-only команды ничего не меняют в cluster. Их можно выполнять безопасно.

Примеры:

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events
kubectl explain pod.spec
```

Они нужны для просмотра состояния, диагностики и изучения структуры объектов.

В лабораторных работах read-only команды будут использоваться постоянно.

## Imperative команды

Imperative-команды сразу говорят Kubernetes, что нужно сделать.

Примеры:

```bash
kubectl run nginx --image=nginx
kubectl create namespace kdl-lab07
kubectl delete pod nginx
kubectl scale deployment nginx --replicas=3
```

Такие команды удобны для быстрых экспериментов и диагностики, но у них есть минус: итоговое состояние cluster не всегда полностью описано в файлах.

Проблема imperative-подхода:

```text
Команда была выполнена в терминале.
Файл с описанием изменения может не остаться.
Другой человек не всегда поймет, почему cluster находится именно в таком состоянии.
```

В production-like подходе это неудобно.

## Declarative подход

Declarative-подход означает, что мы описываем желаемое состояние в YAML-файле, а затем применяем его.

Пример:

```bash
kubectl apply -f pod.yaml
```

В этом подходе важен не сам факт выполнения команды, а содержимое файла.

Преимущества:

* состояние описано в manifest-файле;
* manifest можно хранить в Git;
* изменения можно смотреть через diff;
* проще повторить результат на другом cluster;
* проще ревьюить изменения через Pull Request;
* этот подход ближе к GitOps и production operations.

В этой лабораторной основной путь - declarative.

## Почему мы не используем `kubectl run` как основной способ

`kubectl run` может быстро создать Pod для эксперимента.

Например:

```bash
kubectl run nginx --image=nginx
```

Но для обучения объектной модели Kubernetes это плохая основа, если использовать команду вслепую.

Причины:

* команда скрывает от студента итоговый YAML;
* сложнее увидеть `apiVersion`, `kind`, `metadata`, `spec`;
* сложнее понять, какие поля есть у объекта;
* результат может зависеть от версии `kubectl`;
* такой подход плохо подходит для Git-based workflow.

Поэтому в Lab 07 мы будем использовать manifest-файлы. `kubectl run` можно использовать только как дополнительный инструмент для быстрых временных проверок, но не как основной способ выполнения лабораторной.

## `create`, `apply`, `replace`, `patch`, `edit`

Важно различать команды изменения объектов.

| Команда                        | Что делает                                | Когда использовать                                      |
| ------------------------------ | ----------------------------------------- | ------------------------------------------------------- |
| `kubectl create -f file.yaml`  | Создает объект из файла                   | Когда объект точно еще не существует                    |
| `kubectl apply -f file.yaml`   | Создает или обновляет объект декларативно | Основной способ в этой лабораторной                     |
| `kubectl replace -f file.yaml` | Заменяет объект содержимым файла          | Осторожно, можно потерять поля                          |
| `kubectl patch`                | Изменяет часть объекта                    | Для точечных изменений и диагностики                    |
| `kubectl edit`                 | Открывает live-объект в редакторе         | Удобно для эксперимента, но плохо для воспроизводимости |

В этой лабораторной основной способ:

```bash
kubectl apply -f <file>
```

Для удаления будем использовать `kubectl delete`, но только там, где это явно указано.

## Риск операций

В Lab 07 мы работаем в отдельном namespace. Это снижает риск для cluster.

Тем не менее команды делятся по риску.

LOW:

```text
kubectl get
kubectl describe
kubectl logs
kubectl explain
kubectl diff
```

MEDIUM:

```text
kubectl apply
kubectl create
kubectl scale
kubectl patch
kubectl edit
```

HIGH:

```text
kubectl delete namespace
kubectl delete all --all
удаление объектов вне учебного namespace
любые изменения в kube-system
```

В этой лабораторной запрещено изменять `kube-system`, control plane static pods, CNI components и системные DaemonSet.

## Рабочий namespace

Все объекты Lab 07 должны создаваться только в отдельном namespace:

```text
kdl-lab07
```

Это нужно, чтобы:

* не смешивать учебные объекты с системными;
* проще смотреть события;
* проще удалять лабораторные объекты;
* снизить риск случайно затронуть чужие workloads.

## Структура Lab 07

Лабораторная будет разбита на отдельные файлы.

```text
lab-07-kubernetes-api-basics/
├── README.md
├── 01-namespace.md
├── 02-pod.md
├── 03-labels-and-selectors.md
├── 04-replicaset.md
├── 05-deployment.md
├── 06-service.md
├── 07-endpointslice.md
├── 08-events-logs-troubleshooting.md
├── 09-interview-questions.md
├── student-report-template.md
└── manifests/
    ├── namespace.yaml
    ├── pod-nginx.yaml
    ├── pod-nginx-v2.yaml
    ├── replicaset-nginx.yaml
    ├── deployment-nginx.yaml
    ├── deployment-nginx-broken-image.yaml
    └── service-nginx.yaml
```

Каждый файл отвечает за одну сущность или один диагностический слой.

## Порядок прохождения

Проходить нужно строго по порядку.

```text
README
  -> Namespace
    -> Pod
      -> labels/selectors
        -> ReplicaSet
          -> Deployment
            -> Service
              -> EndpointSlice
                -> events/logs/troubleshooting
                  -> interview questions
```

Не переходите к Deployment, пока не разобрали Pod.

Не переходите к Service, пока не разобрали labels и selectors.

Не переходите к EndpointSlice, пока не поняли, как Service выбирает Pods.

## Что нужно фиксировать в отчете

В отчете нужно фиксировать не только команды, но и выводы.

Для каждого объекта нужно указать:

* что это за объект;
* зачем он нужен;
* каким manifest-файлом он создан;
* какие команды использовались для просмотра;
* что видно в `metadata`;
* что видно в `spec`;
* что видно в `status`;
* какие events появились;
* какие ошибки возникли;
* как была выполнена проверка;
* какой вывод вы сделали.

Отчет должен показывать не только "я выполнил команды", а "я понял, что Kubernetes сделал после моих действий".

## Общая схема диагностики

Если объект создан, но что-то не работает, не нужно сразу удалять и пересоздавать все подряд.

Идем по порядку:

```text
kubectl get
  -> kubectl describe
    -> kubectl get events
      -> kubectl logs
        -> kubectl exec
          -> проверка labels/selectors
            -> проверка node/runtime/network layer
```

В Lab 07 мы пока не уходим глубоко в node/runtime/network layer. Это будет позже. Сейчас задача - научиться читать состояние Kubernetes objects.

## Что запрещено в этой лабораторной

Не выполнять:

```bash
kubectl delete namespace kube-system
kubectl delete pod -n kube-system ...
kubectl delete daemonset -n kube-system ...
kubectl edit daemonset -n kube-system ...
kubectl apply -f random-file-from-internet.yaml
```

Не использовать manifests из случайных статей и блогов.

Не менять CNI, CoreDNS, kube-proxy и control plane components.

Не выполнять команды удаления вне namespace `kdl-lab07`.

## Критерии готовности

Лабораторная считается выполненной, если вы можете объяснить:

* что такое Kubernetes object;
* зачем нужен `apiVersion`;
* зачем нужен `kind`;
* что хранится в `metadata`;
* чем `spec` отличается от `status`;
* что такое Namespace;
* что такое Pod;
* почему Pod обычно не создают напрямую в production;
* что такое label;
* что такое selector;
* как ReplicaSet поддерживает количество Pods;
* зачем Deployment управляет ReplicaSet;
* зачем Service нужен поверх Pods;
* что такое ClusterIP;
* что такое EndpointSlice;
* где смотреть events;
* чем отличаются `get`, `describe`, `logs`, `exec`;
* почему основной путь в этой лабораторной - YAML + `kubectl apply`.

## Что сдавать

В Pull Request нужно добавить или обновить отчет по Lab 07.

Минимально в отчете должны быть:

* ссылка на ваш namespace;
* вывод `kubectl get all -n kdl-lab07`;
* вывод по Pod;
* вывод по ReplicaSet;
* вывод по Deployment;
* вывод по Service;
* вывод по EndpointSlice;
* пример events;
* пример logs;
* краткие ответы на вопросы из `09-interview-questions.md`;
* личный вывод: что стало понятнее после лабораторной.

## Источники для самостоятельного изучения

Использовать как основные источники:

- Pods: https://kubernetes.io/docs/concepts/workloads/pods/
- Deployments: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- ReplicaSet: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- Services: https://kubernetes.io/docs/concepts/services-networking/service/
- Labels and Selectors: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
- kubectl reference: https://kubernetes.io/docs/reference/kubectl/generated/
- kubectl logs: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/
- kubectl port-forward: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/

Случайные статьи, устаревшие инструкции и готовые manifests из интернета не считаются источником истины для этой лабораторной.
