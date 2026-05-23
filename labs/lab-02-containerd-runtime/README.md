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

## Проверка доступных пакетов

Выполнить на control-plane node:

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

Создать config и применить настройку:

```bash
sudo mkdir -p /etc/containerd
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
   - io.containerd.cri.v1
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