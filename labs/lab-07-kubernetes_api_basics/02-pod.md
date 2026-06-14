# Lab 07.02 - Pod

В этой части Lab 07 мы разберем объект `Pod`.

Pod - это минимальная единица запуска workload в Kubernetes. Kubernetes не запускает container как самостоятельный объект. Он запускает Pod, а container находится внутри Pod.

В этой части мы создадим один Pod через YAML manifest, посмотрим его `metadata`, `spec`, `status`, проверим logs, выполним команду внутри container, посмотрим events, изменим manifest и разберем, почему одиночный Pod не является хорошим способом запуска приложения в production-like сценарии.

## Что изучаем

В этой части вы разберете:

* что такое Pod;
* почему Pod является минимальной единицей запуска в Kubernetes;
* как Pod связан с container runtime;
* как создать Pod через manifest;
* как посмотреть Pod IP и node, на которой он запущен;
* что находится в `metadata`;
* что находится в `spec`;
* что находится в `status`;
* что такое Pod phase;
* что такое container state;
* как смотреть logs;
* как выполнить команду внутри container через `kubectl exec`;
* как смотреть events;
* почему Pod можно удалить, и Kubernetes не обязан восстановить его без controller;
* почему в реальной работе обычно используют Deployment, а не одиночный Pod.

## Что такое Pod

Pod - это минимальный deployable object в Kubernetes.

Pod может содержать:

* один container;
* несколько containers;
* volumes;
* общую network namespace;
* общий Pod IP;
* настройки restart policy;
* настройки service account;
* security context;
* resource requests и limits.

В этой лабораторной мы используем самый простой вариант: один Pod с одним container `nginx`.

Упрощенная схема:

```text id="02pod-001"
Pod
  -> container: nginx
  -> Pod IP
  -> volumes
  -> network namespace
```

## Почему Kubernetes запускает Pod, а не container напрямую

Container - это сущность container runtime.

Pod - это сущность Kubernetes API.

Kubernetes описывает желаемое состояние через Pod. После этого kubelet на выбранной node обращается к container runtime через CRI и просит создать нужные containers.

Упрощенно:

```text id="02pod-002"
Pod object in API Server
  -> scheduler selects node
    -> kubelet watches assigned Pod
      -> kubelet asks container runtime
        -> containerd creates container
```

Важно: `kubectl` не запускает container напрямую. Он создает объект в API Server.

## Где будет создан Pod

В предыдущей части мы создали namespace:

```text id="02pod-003"
kdl-lab07
```

Все объекты этой части создаем только в нем.

Проверьте namespace.

Риск: LOW.

```bash id="02pod-004"
kubectl get namespace kdl-lab07
```

Если namespace отсутствует, вернитесь к `01-namespace.md` и создайте его через manifest.

## Manifest для Pod

Создайте файл:

```text id="02pod-005"
manifests/pod-nginx.yaml
```

Содержимое:

```yaml id="02pod-006"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/component: pod
spec:
  containers:
    - name: nginx
      image: nginx:1.27.5
      ports:
        - containerPort: 80
```

Разбор полей:

| Поле                 | Значение                                                        |
| -------------------- | --------------------------------------------------------------- |
| `apiVersion: v1`     | Pod относится к core API group версии `v1`                      |
| `kind: Pod`          | Тип объекта                                                     |
| `metadata.name`      | Имя Pod                                                         |
| `metadata.namespace` | Namespace, где будет создан Pod                                 |
| `metadata.labels`    | Метки для поиска и будущей связи с selectors                    |
| `spec.containers`    | Список containers внутри Pod                                    |
| `containers[].name`  | Имя container внутри Pod                                        |
| `containers[].image` | Container image                                                 |
| `containerPort: 80`  | Документирует порт, который слушает приложение внутри container |

Важно: `containerPort` сам по себе не публикует порт наружу. Он только описывает порт container в Pod spec. Для доступа к Pod позже понадобится Service или другие механизмы.

## Проверка структуры Pod через kubectl explain

Перед применением manifest посмотрите справку по Pod.

Риск: LOW.

```bash id="02pod-007"
kubectl explain pod
```

Посмотрите `spec`:

```bash id="02pod-008"
kubectl explain pod.spec
```

Посмотрите containers:

```bash id="02pod-009"
kubectl explain pod.spec.containers
```

Посмотрите image:

```bash id="02pod-010"
kubectl explain pod.spec.containers.image
```

Задача: увидеть, что manifest не является произвольным YAML. Его структура описана Kubernetes API.

## Dry-run перед созданием

Сначала client-side dry-run.

Риск: LOW.

```bash id="02pod-011"
kubectl apply --dry-run=client -f manifests/pod-nginx.yaml
```

Теперь server-side dry-run.

Риск: LOW.

```bash id="02pod-012"
kubectl apply --dry-run=server -f manifests/pod-nginx.yaml
```

Если обе команды прошли успешно, manifest можно применять.

## Создание Pod

Создайте Pod.

Риск: MEDIUM.

Что изменится: в namespace `kdl-lab07` будет создан Pod `nginx-pod`.

```bash id="02pod-013"
kubectl apply -f manifests/pod-nginx.yaml
```

Ожидаемый результат:

```text id="02pod-014"
pod/nginx-pod created
```

Если Pod уже существует и manifest не изменился:

```text id="02pod-015"
pod/nginx-pod unchanged
```

## Проверка Pod

Посмотрите Pods в namespace.

Риск: LOW.

```bash id="02pod-016"
kubectl get pods -n kdl-lab07
```

Более подробный вывод:

```bash id="02pod-017"
kubectl get pods -n kdl-lab07 -o wide
```

Обратите внимание на поля:

| Поле       | Что означает                       |
| ---------- | ---------------------------------- |
| `READY`    | Сколько containers готовы          |
| `STATUS`   | Текущее состояние Pod              |
| `RESTARTS` | Количество перезапусков containers |
| `AGE`      | Возраст Pod                        |
| `IP`       | Pod IP                             |
| `NODE`     | Node, на которой запущен Pod       |

Ожидаемо Pod должен перейти в состояние:

```text id="02pod-018"
Running
```

Если Pod находится в `ContainerCreating`, подождите немного и повторите команду.

## Подождать готовность Pod

Команда `kubectl wait` позволяет ждать condition у объекта.

Риск: LOW.

```bash id="02pod-019"
kubectl wait --for=condition=Ready pod/nginx-pod -n kdl-lab07 --timeout=120s
```

Ожидаемый результат:

```text id="02pod-020"
pod/nginx-pod condition met
```

Если timeout истек, переходите к диагностике через `describe` и events.

## Посмотреть Pod в YAML

Посмотрите live-объект из API Server.

Риск: LOW.

```bash id="02pod-021"
kubectl get pod nginx-pod -n kdl-lab07 -o yaml
```

Найдите в выводе:

```text id="02pod-022"
metadata
spec
status
```

Важно: live YAML будет намного больше исходного manifest. Kubernetes добавляет служебные поля:

```text id="02pod-023"
uid
resourceVersion
creationTimestamp
managedFields
status
podIP
conditions
containerStatuses
```

Не нужно копировать весь live YAML обратно в manifest. Manifest должен содержать только нужное желаемое состояние.

## Разбор metadata

В `metadata` найдите:

```text id="02pod-024"
name
namespace
labels
uid
resourceVersion
creationTimestamp
```

Что важно:

| Поле                | Значение                     |
| ------------------- | ---------------------------- |
| `name`              | Имя объекта внутри namespace |
| `namespace`         | Namespace, где находится Pod |
| `labels`            | Метки для поиска и selectors |
| `uid`               | Уникальный ID объекта        |
| `resourceVersion`   | Версия объекта в API Server  |
| `creationTimestamp` | Время создания               |

`uid` отличается от `name`. Если удалить Pod и создать новый с тем же именем, `name` может быть таким же, но `uid` будет другим.

## Разбор spec

В `spec` найдите:

```text id="02pod-025"
containers
nodeName
restartPolicy
dnsPolicy
serviceAccountName
schedulerName
```

Что важно:

| Поле                 | Значение                                   |
| -------------------- | ------------------------------------------ |
| `containers`         | Containers, которые должны быть внутри Pod |
| `nodeName`           | Node, куда scheduler назначил Pod          |
| `restartPolicy`      | Политика перезапуска containers внутри Pod |
| `dnsPolicy`          | DNS policy для Pod                         |
| `serviceAccountName` | ServiceAccount, под которым работает Pod   |
| `schedulerName`      | Scheduler, который назначает Pod           |

В исходном manifest мы не указывали все эти поля. Kubernetes применил значения по умолчанию.

## Разбор status

В `status` найдите:

```text id="02pod-026"
phase
podIP
hostIP
conditions
containerStatuses
startTime
```

Что важно:

| Поле                | Значение                        |
| ------------------- | ------------------------------- |
| `phase`             | Общая фаза Pod                  |
| `podIP`             | IP address Pod                  |
| `hostIP`            | IP address node                 |
| `conditions`        | Набор условий Pod               |
| `containerStatuses` | Состояние containers внутри Pod |
| `startTime`         | Время старта Pod                |

Pod `status.phase` может быть, например:

```text id="02pod-027"
Pending
Running
Succeeded
Failed
Unknown
```

Не путайте `phase` с детальными причинами вроде `ImagePullBackOff` или `CrashLoopBackOff`. Эти причины обычно видны в container status, `kubectl get pods`, `describe` и events.

## kubectl describe

Посмотрите подробное описание Pod.

Риск: LOW.

```bash id="02pod-028"
kubectl describe pod nginx-pod -n kdl-lab07
```

Обратите внимание на блоки:

```text id="02pod-029"
Name
Namespace
Node
IP
Containers
Conditions
Volumes
QoS Class
Events
```

`kubectl describe` удобен тем, что показывает не только поля объекта, но и связанные события.

## Проверка logs

Посмотрите logs container.

Риск: LOW.

```bash id="02pod-030"
kubectl logs pod/nginx-pod -n kdl-lab07
```

У nginx logs могут быть пустыми, если к нему еще не было запросов.

Это нормально.

Чтобы сгенерировать запрос к nginx, используем `kubectl exec` внутри Pod.

## kubectl exec

Выполните команду внутри container.

Риск: LOW.

```bash id="02pod-031"
kubectl exec -n kdl-lab07 pod/nginx-pod -- nginx -v
```

Ожидаемо команда покажет версию nginx.

Теперь попробуйте выполнить HTTP-запрос к localhost внутри самого Pod.

Риск: LOW.

```bash id="02pod-032"
kubectl exec -n kdl-lab07 pod/nginx-pod -- curl -I http://127.0.0.1:80
```

Если в image нет `curl`, команда завершится ошибкой. Это не проблема Kubernetes. Это означает, что внутри container нет бинарника `curl`.

В таком случае используйте:

```bash id="02pod-033"
kubectl exec -n kdl-lab07 pod/nginx-pod -- nginx -T
```

Важно: `kubectl exec` выполняет команду внутри container. Если нужной команды нет в image, Kubernetes ее не добавит.

## Повторная проверка logs

После запроса снова посмотрите logs.

Риск: LOW.

```bash id="02pod-034"
kubectl logs pod/nginx-pod -n kdl-lab07
```

Если запрос дошел до nginx, в logs появится запись.

Если logs пустые, проверьте, получилось ли выполнить HTTP-запрос внутри container.

## Проверка events

Посмотрите events в namespace.

Риск: LOW.

```bash id="02pod-035"
kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
```

Ожидаемо вы можете увидеть события вида:

```text id="02pod-036"
Scheduled
Pulling
Pulled
Created
Started
```

Что они означают:

| Event       | Значение                         |
| ----------- | -------------------------------- |
| `Scheduled` | Scheduler назначил Pod на node   |
| `Pulling`   | Kubelet начал скачивать image    |
| `Pulled`    | Image скачан или уже был на node |
| `Created`   | Container создан                 |
| `Started`   | Container запущен                |

Events помогают понять, на каком этапе возникла проблема.

## Проверка Pod IP

Сохраните Pod IP из команды:

```bash id="02pod-037"
kubectl get pod nginx-pod -n kdl-lab07 -o wide
```

Можно вывести только IP:

```bash id="02pod-038"
kubectl get pod nginx-pod -n kdl-lab07 -o jsonpath='{.status.podIP}'; echo
```

Важно: Pod IP принадлежит конкретному Pod. Если Pod удалить и создать заново, IP может измениться.

Не нужно строить стабильный доступ к приложению через Pod IP. Для этого позже будет Service.

## Изменение Pod manifest

Теперь изменим manifest и посмотрим, что произойдет.

Создайте файл:

```text id="02pod-039"
manifests/pod-nginx-v2.yaml
```

Содержимое:

```yaml id="02pod-040"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: nginx
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/component: pod
    kdl/version: "v2"
spec:
  containers:
    - name: nginx
      image: nginx:1.27.5
      ports:
        - containerPort: 80
```

Отличие: мы добавили label:

```text id="02pod-041"
kdl/version: "v2"
```

Проверьте diff.

Риск: LOW.

```bash id="02pod-042"
kubectl diff -f manifests/pod-nginx-v2.yaml
```

Примените изменение.

Риск: MEDIUM.

```bash id="02pod-043"
kubectl apply -f manifests/pod-nginx-v2.yaml
```

Проверьте labels:

```bash id="02pod-044"
kubectl get pod nginx-pod -n kdl-lab07 --show-labels
```

Вы должны увидеть:

```text id="02pod-045"
kdl/version=v2
```

## Почему мы не меняем image у существующего Pod

У Pod есть поля, которые нельзя свободно менять после создания.

Например, для уже созданного Pod нельзя просто так изменить многие поля `spec`. В реальной работе для обновления приложения используют controllers, например Deployment.

Если нужно заменить container image у приложения, обычно меняют Deployment, а Deployment создает новый ReplicaSet и новые Pods.

В этой части мы специально не меняем image у одиночного Pod, чтобы не смешивать тему Pod с темой Deployment.

## Проверка, что одиночный Pod не самовосстанавливается как приложение

Сейчас мы покажем важное ограничение одиночного Pod.

Сначала убедитесь, что Pod существует.

Риск: LOW.

```bash id="02pod-046"
kubectl get pod nginx-pod -n kdl-lab07
```

Удаление Pod.

Риск: MEDIUM.

Что будет изменено: Pod `nginx-pod` будет удален. Так как мы создали его напрямую, controller не обязан создать новый Pod вместо него.

```bash id="02pod-047"
kubectl delete pod nginx-pod -n kdl-lab07
```

Проверьте:

```bash id="02pod-048"
kubectl get pods -n kdl-lab07
```

Ожидаемо:

```text id="02pod-049"
No resources found in kdl-lab07 namespace.
```

Главный вывод: одиночный Pod после удаления не восстанавливается, если за ним не стоит controller.

## Восстановление Pod из manifest

Создайте Pod заново.

Риск: MEDIUM.

```bash id="02pod-050"
kubectl apply -f manifests/pod-nginx-v2.yaml
```

Проверьте:

```bash id="02pod-051"
kubectl get pod nginx-pod -n kdl-lab07 -o wide
```

Сравните новый Pod IP с прежним, если вы его записали.

С высокой вероятностью Pod IP изменился.

Это еще одна причина, почему нельзя строить стабильный доступ к приложению через Pod IP.

## Мини-диагностика: неправильный image

Создайте файл:

```text id="02pod-052"
manifests/pod-broken-image.yaml
```

Содержимое:

```yaml id="02pod-053"
apiVersion: v1
kind: Pod
metadata:
  name: broken-image-pod
  namespace: kdl-lab07
  labels:
    app.kubernetes.io/name: broken-image
    app.kubernetes.io/part-of: kubernetes-deep-lab
    kdl/lab: "07"
    kdl/component: pod
spec:
  containers:
    - name: nginx
      image: nginx:this-tag-does-not-exist
      ports:
        - containerPort: 80
```

Проверьте dry-run.

Риск: LOW.

```bash id="02pod-054"
kubectl apply --dry-run=server -f manifests/pod-broken-image.yaml
```

Примените.

Риск: MEDIUM.

```bash id="02pod-055"
kubectl apply -f manifests/pod-broken-image.yaml
```

Посмотрите состояние.

```bash id="02pod-056"
kubectl get pod broken-image-pod -n kdl-lab07
```

Ожидаемо Pod не станет `Running`. Вы можете увидеть:

```text id="02pod-057"
ErrImagePull
ImagePullBackOff
```

Подробности:

```bash id="02pod-058"
kubectl describe pod broken-image-pod -n kdl-lab07
```

Events:

```bash id="02pod-059"
kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
```

Главный вывод: объект Pod создан в API Server, но container не запущен, потому что kubelet/container runtime не смог скачать image.

## Cleanup broken Pod

Удалите только ошибочный Pod.

Риск: MEDIUM.

Что будет изменено: будет удален только Pod `broken-image-pod`.

```bash id="02pod-060"
kubectl delete pod broken-image-pod -n kdl-lab07
```

Проверьте:

```bash id="02pod-061"
kubectl get pods -n kdl-lab07
```

Pod `nginx-pod` должен остаться.

## Cleanup основного Pod

Если вы переходите к следующей части про labels и selectors, Pod можно оставить.

Если нужно очистить только Pod из этой части:

Риск: MEDIUM.

Что будет изменено: будет удален Pod `nginx-pod`.

```bash id="02pod-062"
kubectl delete pod nginx-pod -n kdl-lab07
```

Проверьте:

```bash id="02pod-063"
kubectl get pods -n kdl-lab07
```

Не удаляйте namespace `kdl-lab07`, если продолжаете Lab 07.

## Инженерная заметка / Вопрос с собеседований

Вопрос: что такое Pod в Kubernetes?

Краткий ответ: Pod - это минимальная единица запуска workload в Kubernetes. Внутри Pod находится один или несколько containers, которые разделяют network namespace, Pod IP и могут использовать общие volumes. Kubernetes планирует и запускает Pod, а containers внутри Pod создает container runtime по запросу kubelet.

Как это видно в этой лабе: мы создали объект `Pod` через YAML manifest. После этого scheduler назначил Pod на node, kubelet скачал image и запустил container через runtime. Это видно через `kubectl get pod -o wide`, `kubectl describe pod` и events.

Что важно помнить: Pod обычно не является стабильной сущностью. Если Pod удалить, он может не восстановиться, если над ним нет controller. Pod IP тоже не нужно считать стабильным.

## Инженерная заметка / Вопрос с собеседований

Вопрос: почему в production обычно не создают одиночные Pods напрямую?

Краткий ответ: одиночный Pod не управляет replicas, rollout, rollback и самовосстановлением приложения. Если такой Pod удалить, Kubernetes не обязан создать новый. Для приложений обычно используют controllers: Deployment, StatefulSet, DaemonSet или Job.

Как это видно в этой лабе: мы удалили `nginx-pod`, и он не восстановился автоматически. Чтобы вернуть его, пришлось снова применить manifest.

Что важно помнить: Pod - это базовый building block. Для управления жизненным циклом приложения нужен controller.

## Инженерная заметка / Вопрос с собеседований

Вопрос: что означает `ImagePullBackOff`?

Краткий ответ: `ImagePullBackOff` означает, что kubelet не смог скачать container image и временно откладывает повторные попытки. Причины могут быть разные: неправильное имя image, несуществующий tag, недоступный registry, отсутствие credentials или сетевая проблема.

Как это видно в этой лабе: мы создали Pod с image `nginx:this-tag-does-not-exist`. Kubernetes принял объект Pod, но container не запустился. Причина видна в `kubectl describe pod` и events.

Что важно помнить: `ImagePullBackOff` - это не проблема `kubectl apply`. Это проблема на этапе kubelet -> container runtime -> image registry.

## Контрольные вопросы

Ответьте в отчете:

1. Что такое Pod?
2. Почему Kubernetes запускает Pod, а не container напрямую?
3. Что находится в `metadata` Pod?
4. Что находится в `spec` Pod?
5. Что находится в `status` Pod?
6. Чем `status.phase` отличается от `containerStatuses`?
7. Что показывает `kubectl get pod -o wide`?
8. Что показывает `kubectl describe pod`?
9. Почему logs nginx могут быть пустыми после запуска?
10. Что делает `kubectl exec`?
11. Почему команда внутри `kubectl exec` может не найтись?
12. Почему Pod IP нельзя считать стабильным?
13. Что произойдет, если удалить одиночный Pod без controller?
14. Почему для приложений обычно используют Deployment, а не Pod напрямую?
15. Что означает `ImagePullBackOff`?

## Что добавить в отчет

Добавьте в отчет по Lab 07:

```text id="02pod-064"
Файл: 02-pod.md

Что было сделано:
- создан Pod nginx-pod через manifest;
- проверены metadata, spec и status;
- проверен Pod IP и node;
- выполнен kubectl describe;
- проверены logs;
- выполнена команда внутри container через kubectl exec;
- проверены events;
- добавлен label через обновленный manifest;
- проверено поведение после удаления одиночного Pod;
- создан Pod с ошибочным image;
- разобран ImagePullBackOff.

Команды:
- kubectl explain pod
- kubectl explain pod.spec
- kubectl explain pod.spec.containers
- kubectl apply --dry-run=server -f manifests/pod-nginx.yaml
- kubectl apply -f manifests/pod-nginx.yaml
- kubectl get pods -n kdl-lab07 -o wide
- kubectl wait --for=condition=Ready pod/nginx-pod -n kdl-lab07 --timeout=120s
- kubectl get pod nginx-pod -n kdl-lab07 -o yaml
- kubectl describe pod nginx-pod -n kdl-lab07
- kubectl logs pod/nginx-pod -n kdl-lab07
- kubectl exec -n kdl-lab07 pod/nginx-pod -- nginx -v
- kubectl get events -n kdl-lab07 --sort-by=.metadata.creationTimestamp
- kubectl diff -f manifests/pod-nginx-v2.yaml
- kubectl delete pod nginx-pod -n kdl-lab07
- kubectl apply -f manifests/pod-broken-image.yaml
- kubectl describe pod broken-image-pod -n kdl-lab07

Вывод:
- своими словами объяснить, что такое Pod, почему он не является стабильной единицей приложения и зачем дальше нужен controller.
