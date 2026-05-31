# Lab 03 - kubeadm prerequisites

## Цель

Подготовить все nodes к созданию Kubernetes cluster через `kubeadm`.

В этой лабораторной мы:

- подключим официальный Kubernetes apt repository;
- установим `kubelet`, `kubeadm`, `kubectl`;
- зафиксируем версии Kubernetes packages через `apt-mark hold`;
- проверим состояние `kubelet` до создания кластера;
- убедимся, что все nodes готовы к следующей лабораторной.


## Где выполнять

Команды выполняются последовательно на каждой node:

- control-plane node;
- worker nodes.

Если действие одинаковое для всех nodes, оно не расписывается отдельно для каждой node.

## Предварительные требования

Перед началом должны быть завершены:

- `lab-00-environment`;
- `lab-01-node-baseline`;
- `lab-02-containerd-runtime`.

На всех nodes должны быть готовы:

- корректный hostname;
- корректный hostname resolution;
- отключенный swap;
- загруженные kernel modules;
- нужные sysctl параметры;
- установленный и работающий `containerd`;
- настроенный `crictl`;
- `RuntimeReady: true`;
- `SystemdCgroup: true`.

## Kubernetes version

В этой лабораторной используем Kubernetes minor release:

```text
v1.36
```

Официальный Kubernetes package repository разделен по minor versions. Для `v1.36` используется repository:

```text
https://pkgs.k8s.io/core:/stable:/v1.36/deb/
```

Если в будущем будет выбран другой minor release, путь repository нужно менять явно.

## Проверка текущего состояния

Команды ничего не меняют. Они только показывают текущее состояние node.

Выполнить на каждой node:

```bash
hostname
cat /etc/os-release | grep -E '^(PRETTY_NAME|VERSION_ID)='
uname -r

containerd --version
crictl --version
cat /etc/crictl.yaml

sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'

swapon --show
lsmod | grep -E 'br_netfilter|overlay' || true
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward 2>/dev/null || true
```

Ожидаем:

- `containerd` установлен;
- `crictl` установлен;
- `crictl` смотрит в socket `containerd`;
- `RuntimeReady` имеет значение `true`;
- `SystemdCgroup` имеет значение `true`;
- swap отключен;
- `overlay` и `br_netfilter` доступны;
- `net.ipv4.ip_forward = 1`;
- `net.bridge.bridge-nf-call-iptables = 1`;
- `net.bridge.bridge-nf-call-ip6tables = 1`.

`NetworkReady` может быть `false` до установки CNI. На этом этапе это нормально.

## Установка зависимостей для apt repository

Что будет изменено:

- будет обновлен apt package index;
- будут установлены пакеты, необходимые для подключения apt repository.

Кластер Kubernetes создан не будет.

Выполнить на каждой node:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

Проверка:

```bash
dpkg -l | grep -E 'apt-transport-https|ca-certificates|curl|gpg'
```

## Подключение Kubernetes apt repository

Что будет изменено:

- будет создан каталог `/etc/apt/keyrings`, если он еще не существует;
- будет добавлен Kubernetes repository signing key;
- будет создан файл `/etc/apt/sources.list.d/kubernetes.list`.

Выполнить на каждой node:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Проверка:

```bash
cat /etc/apt/sources.list.d/kubernetes.list
ls -l /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Ожидаем:

```text
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
```

## Проверка доступных версий

Команды ничего не меняют, кроме обновления apt package index.

Выполнить на каждой node:

```bash
sudo apt-get update
apt-cache madison kubeadm
apt-cache madison kubelet
apt-cache madison kubectl
```

Ожидаем:

- `kubeadm`, `kubelet`, `kubectl` доступны из repository `pkgs.k8s.io`;
- версии относятся к выбранному minor release `v1.36`.

## Установка kubelet, kubeadm и kubectl

Что будет изменено:

- будут установлены `kubelet`, `kubeadm`, `kubectl`;
- будет установлен systemd unit для `kubelet`;
- `kubelet` может стартовать, но до `kubeadm init` или `kubeadm join` он еще не сможет полноценно работать как node agent;
- версии пакетов будут зафиксированы через `apt-mark hold`.

Выполнить на каждой node:

```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Проверка:

```bash
kubeadm version
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
```

Ожидаем:

- `kubeadm` установлен;
- `kubelet` установлен;
- `kubectl` установлен;
- все три пакета находятся в hold.

## Проверка kubelet до bootstrap


Команды ничего не меняют.

Выполнить на каждой node:

```bash
systemctl status kubelet --no-pager || true
```

```bash
journalctl -u kubelet -n 50 --no-pager
```

На этом этапе `kubelet` может быть в состоянии ошибки или постоянно перезапускаться.

Это ожидаемо до выполнения:

- `kubeadm init` на control-plane;
- `kubeadm join` на worker nodes.

Причина: `kubelet` уже установлен как systemd service, но node еще не прошла bootstrap. У kubelet еще может не быть полноценной конфигурации, kubeconfig и связи с Kubernetes API Server.

## Проверка файлов kubelet

Команды ничего не меняют.

Выполнить на каждой node:

```bash
ls -l /etc/systemd/system/kubelet.service.d/ || true
ls -l /var/lib/kubelet/ || true
```

Посмотреть kubeadm drop-in для kubelet:

```bash
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

Важно понять:

- `kubelet` запускается через systemd;
- `kubelet` не является Pod;
- `kubeadm` добавляет drop-in конфигурацию для kubelet;
- полноценная конфигурация kubelet появится после bootstrap node.

## Финальная проверка Lab 03

Выполнить на каждой node:

```bash
hostname
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl is-enabled kubelet
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
```

Ожидаем:

- версии Kubernetes packages совпадают на всех nodes;
- `kubelet`, `kubeadm`, `kubectl` находятся в hold;
- `containerd` по-прежнему работает;
- `RuntimeReady` по-прежнему `true`;
- `NetworkReady` может оставаться `false` до установки CNI.

## Rollback

Rollback нужен только если установка пакетов прошла некорректно или выбран неправильный Kubernetes minor release.

Сначала dry-run:

```bash
sudo apt-get remove --dry-run kubelet kubeadm kubectl
```

Если dry-run выглядит ожидаемо, можно удалить пакеты:

```bash
sudo apt-mark unhold kubelet kubeadm kubectl
sudo apt-get remove -y kubelet kubeadm kubectl
```

Удаление repository configuration:

```bash
sudo rm /etc/apt/sources.list.d/kubernetes.list
sudo rm /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo apt-get update
```

Важно: не выполнять rollback без причины. Если `kubelet` после установки находится в ошибке до `kubeadm init/join`, это само по себе не является причиной для rollback.

## Критерии готовности

Lab 03 считается завершенной, если:

- `kubeadm` установлен на всех nodes;
- `kubelet` установлен на всех nodes;
- `kubectl` установлен на всех nodes;
- версии Kubernetes packages совпадают на всех nodes;
- пакеты `kubelet`, `kubeadm`, `kubectl` находятся в `apt-mark hold`;
- понятно, почему `kubelet` до `kubeadm init/join` может быть не ready;
- `containerd` по-прежнему работает;
- `crictl` по-прежнему показывает `RuntimeReady: true`.

## Что приложить в lab journal

Добавьте в lab journal вывод с control-plane и только отличия с worker nodes:

```bash
hostname
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
```

Если вывод на worker nodes совпадает с control-plane, достаточно написать, что проверка повторена на всех nodes, и приложить только отличия.

## Инженерная заметка / Вопрос с собеседований

Вопрос:

Почему `kubelet` устанавливается и запускается до создания Kubernetes cluster, но может быть не ready?

Краткий ответ:

Потому что `kubelet` - это обычный systemd service на Linux node. После установки пакета service уже существует, но node еще не прошла bootstrap. До `kubeadm init` или `kubeadm join` у kubelet может не быть полноценной конфигурации, kubeconfig и рабочей связи с API Server.

Привязка к этой лабораторной:

В Lab 03 мы устанавливаем `kubelet`, но еще не создаем кластер. Поэтому ошибка или restart loop kubelet на этом этапе не всегда означает поломку. Нужно смотреть конкретные сообщения в `journalctl -u kubelet`, но сам факт, что kubelet еще не стал полноценной Kubernetes node, ожидаем.

## Следующий шаг

Следующая лабораторная:

```text
lab-04-kubeadm-cluster-bootstrap
```

В ней будет выполнено:

- `kubeadm init` на control-plane;
- настройка kubeconfig;
- получение `kubeadm join`;
- подключение worker nodes;
- первичная проверка nodes и static pods.

CNI будет установлен отдельно в следующей лабораторной.
