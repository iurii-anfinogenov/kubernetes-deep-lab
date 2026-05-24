# Lab 02 - Container Runtime: containerd

Эта лабораторная устанавливает и настраивает `containerd` как container runtime для будущего Kubernetes-кластера.

Цель Lab 02 - не запускать Kubernetes, а подготовить runtime layer, через который позже будет работать `kubelet`.

После этой лабораторной на каждой node должны быть:

- установлен `containerd`;
- установлен `runc`;
- создан `/etc/containerd/config.toml`;
- включен `SystemdCgroup = true`;
- `containerd` запущен и включен в автозагрузку;
- CRI plugin загружен;
- `overlayfs` snapshotter доступен.

## Что уже должно быть готово

Перед началом Lab 02 должна быть завершена Lab 01 - Node Baseline.

На каждой node должно быть проверено:

```text
swap disabled
overlay loaded
br_netfilter loaded
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
hostname resolution OK
package repositories доступные
```

## Reference environment

Reference run выполнялся на:

```text
Ubuntu 24.04.3 LTS
Kernel 6.8.0-88-generic
Architecture amd64
containerd 2.2.1
runc 1.3.4
```

Важно: в Ubuntu 24.04 на момент reference run пакет `containerd` устанавливал ветку `containerd 2.x`.

Это важно для конфигурации, потому что в `containerd 2.x` используется config version `3`.

Точные версии пакетов могут отличаться в зависимости от enabled pockets, mirror и даты обновления package index.

Основной путь ниже рассчитан на `containerd 2.x`. Если `apt-cache policy containerd` показывает ветку `1.7.x`, зафиксируйте это в lab report: для нее ожидаемы config version `2` и другие CRI plugin IDs.

## Проверка доступных пакетов

Выполнить на каждой node, потому что установка тоже будет выполняться на каждой VM:

```bash
sudo apt update
apt-cache policy containerd
apt-cache policy runc
```

В reference run результат был:

```text
containerd candidate: 2.2.1-0ubuntu1~24.04.2
runc candidate: 1.3.4-0ubuntu1~24.04.1
```

Если пакеты недоступны, не продолжайте установку. Сначала нужно решить проблему с package repositories, mirror, DNS, HTTPS или региональными ограничениями.

Offline / air-gapped установку разберем позже отдельным advanced-блоком.

## Установка containerd и runc

Риск: LOW.

Выполнить последовательно на каждой node:

```bash
sudo apt update
sudo apt install -y containerd runc
```

Проверить:

```bash
containerd --version
runc --version
systemctl is-enabled containerd
systemctl is-active containerd
```

Критерий готовности:

```text
containerd установлен
runc установлен
containerd enabled
containerd active
```

В reference run:

```text
containerd github.com/containerd/containerd/v2 2.2.1
runc version 1.3.4-0ubuntu1~24.04.1
containerd enabled
containerd active
```

## Проверка default config containerd

После установки на Ubuntu `/etc/containerd/config.toml` может отсутствовать. В этом случае `containerd` работает с default config.

Проверить:

```bash
sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"

containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
```

В reference run:

```text
no /etc/containerd/config.toml
version = 3
default_runtime_name = 'runc'
runtime_type = 'io.containerd.runc.v2'
SystemdCgroup = false
```

## Настройка SystemdCgroup

Для Kubernetes важно, чтобы `containerd` и будущий `kubelet` использовали согласованный cgroup driver.

На Ubuntu 24.04 используется `systemd`, поэтому для reference run выставляем:

```text
SystemdCgroup = true
```

Создать config и применить настройку.

Если `/etc/containerd/config.toml` уже существует, сначала сохранить backup:

```bash
sudo mkdir -p /etc/containerd

if sudo test -f /etc/containerd/config.toml; then
  sudo cp /etc/containerd/config.toml /etc/containerd/config.toml.bak.$(date +%Y%m%d-%H%M%S)
fi

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
```

Проверить:

```bash
grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
systemctl is-active containerd
sudo journalctl -u containerd --no-pager -n 20
```

Критерий готовности:

```text
version = 3
default_runtime_name = 'runc'
runtime_type = 'io.containerd.runc.v2'
SystemdCgroup = true
containerd active
```

В reference run:

```text
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
active
```

## Ожидаемый CNI warning

После restart в логах может быть сообщение:

```text
failed to load cni during init, please check CRI plugin status before setting up network for pods
cni config load failed: no network config found in /etc/cni/net.d
```

На этом этапе это ожидаемо.

Причина:

```text
CNI plugin еще не установлен.
Pod network еще не настроен.
Директория /etc/cni/net.d еще не содержит CNI config.
```

Это не блокер для текущей лабораторной. Этот вопрос будет закрыт позже после `kubeadm init` и установки CNI plugin.

## Проверка containerd через ctr

`ctr` - низкоуровневый клиент containerd.

Он полезен для проверки самого containerd, но это не тот интерфейс, через который `kubelet` работает с runtime.

Проверить:

```bash
sudo ctr namespaces list
sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
```

Критерий готовности:

```text
overlayfs snapshotter: ok
runtime.v2 task: ok
CRI images plugin: ok
CRI runtime plugin: ok
gRPC CRI plugin: ok
```

В reference run:

```text
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok
```

Пустой список namespaces на чистой node - нормальное состояние, потому что контейнеры еще не запускались.

Если у вас `containerd 1.7.x`, часть CRI plugin IDs будет отличаться от reference run на `containerd 2.x`. Это нужно указать в lab report вместе с фактической версией `containerd`.

## Итоговый checkpoint

К концу текущей части Lab 02 на всех nodes должно быть:

```text
containerd installed
runc installed
/etc/containerd/config.toml exists
SystemdCgroup = true
containerd enabled
containerd active
overlayfs snapshotter ok
CRI plugin ok
```

Reference checkpoint:

```text
kdl-cp-1   containerd 2.2.1   runc 1.3.4   SystemdCgroup=true   active/enabled   CRI OK
kdl-w-5    containerd 2.2.1   runc 1.3.4   SystemdCgroup=true   active/enabled   CRI OK
kdl-w-6    containerd 2.2.1   runc 1.3.4   SystemdCgroup=true   active/enabled   CRI OK
```

## Что приложить в lab report

Минимально приложить:

```text
1. containerd --version
2. runc --version
3. systemctl is-enabled containerd
4. systemctl is-active containerd
5. grep по /etc/containerd/config.toml:
   - version
   - default_runtime_name
   - runtime_type
   - SystemdCgroup
6. вывод ctr plugins list с:
   - overlayfs
   - runtime.v2
   - io.containerd.cri.v1 для containerd 2.x или фактический CRI plugin ID для containerd 1.7.x
   - io.containerd.grpc.v1 cri
7. если есть CNI warning в journalctl - указать, что он ожидаем до установки CNI
```

## Следующий шаг

Следующая часть Lab 02:

```text
CRI tooling with crictl
```

Нужно будет:

```text
установить crictl
настроить /etc/crictl.yaml
проверить crictl info
сравнить ctr и crictl
понять, как kubelet общается с container runtime через CRI
```
## Установка и настройка crictl

`crictl` - это CLI для проверки CRI runtime.

Цель раздела - добавить диагностический инструмент `crictl` и настроить его на socket `containerd`:

```text
unix:///run/containerd/containerd.sock
```

После этого можно будет проверять runtime через CRI interface, то есть ближе к тому, как позже `kubelet` будет работать с `containerd`.

### Предварительная проверка

Выполнить на node:

```bash
command -v crictl || true
```

Ожидаемый результат на чистой node:

```text
пустой вывод
```

Если команда уже показывает путь к `crictl`, не устанавливайте его повторно. Сначала проверьте текущую версию:

```bash
crictl --version
```

### Проверка доступности release archive

Для reference run используется версия:

```text
crictl v1.34.0
```

Проверить доступность архива:

```bash
CRICTL_VERSION="v1.34.0"

curl -I --connect-timeout 10 -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
```

Критерий готовности:

```text
GitHub release URL возвращает HTTP 302
release asset возвращает HTTP 200
archive доступен для скачивания
```

### Скачивание и проверка архива

Риск: LOW.

На этом шаге архив скачивается во временный каталог. В `/usr/local/bin` пока ничего не устанавливается.

```bash
CRICTL_VERSION="v1.34.0"
TMP_DIR="$(mktemp -d)"

curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

ls -lh "${TMP_DIR}"
tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
```

Ожидаемый результат внутри архива:

```text
crictl
```

### Установка crictl

Риск: LOW.

Устанавливается один бинарник:

```text
/usr/local/bin/crictl
```

Выполнить в той же shell-сессии, где заданы `CRICTL_VERSION` и `TMP_DIR`:

```bash
sudo tar -C /usr/local/bin -xzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
sudo chmod 0755 /usr/local/bin/crictl
sudo chown root:root /usr/local/bin/crictl
```

Проверить:

```bash
command -v crictl
crictl --version
ls -l /usr/local/bin/crictl
```

Критерий готовности:

```text
/usr/local/bin/crictl
crictl version v1.34.0
-rwxr-xr-x 1 root root ... /usr/local/bin/crictl
```

### Настройка /etc/crictl.yaml

Без явного config `crictl` может пробовать несколько default endpoints. Лучше сразу указать socket `containerd`.

Создать config:

```bash
cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

Проверить:

```bash
cat /etc/crictl.yaml
sudo crictl info
```

Критерий готовности:

```text
runtime-endpoint указывает на unix:///run/containerd/containerd.sock
image-endpoint указывает на unix:///run/containerd/containerd.sock
crictl info успешно возвращает данные runtime
```

### Проверка CRI состояния

В `crictl info` нужно проверить ключевые признаки:

```text
runtimeType: io.containerd.runc.v2
SystemdCgroup: true
RuntimeReady: true
NetworkReady: false
```

На этом этапе `NetworkReady=false` ожидаем.

Причина:

```text
CNI plugin еще не установлен.
Pod network еще не настроен.
```

Главный успешный критерий текущего шага:

```text
RuntimeReady: true
```

Это значит, что CRI runtime доступен.

### Проверка пустого runtime

До установки `kubelet` и Kubernetes components списки containers, images и pods должны быть пустыми.

```bash
sudo crictl ps
sudo crictl ps -a
sudo crictl images
sudo crictl pods
```

Ожидаемый результат:

```text
containers: empty
images: empty
pods: empty
```

Это нормальное состояние, потому что `kubelet` еще ничего не создавал через CRI.

