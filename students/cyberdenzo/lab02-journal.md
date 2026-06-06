# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Денис |
| GitHub | https://github.com/cyberdenzo |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | done |  |  |
| Lab 01 - Node Baseline | done |  |  |
| Lab 02 - Container Runtime | done |  |  |

## Lab 02 - Container Runtime

### Дата

2026-06-06

### Цель

Установить и настроить `containerd` как container runtime для будущего Kubernetes-кластера

### Что было сделано

- Установка containerd и runc
- Проверка default config containerd
- Настройка SystemdCgroup
- Проверка containerd через ctr
- Установка и настройка crictl
- Проверка CRI состояния
- Проверка пустого runtime

### Команды
**k8s-master-01**
```
containerd --version
containerd github.com/containerd/containerd/v2 2.2.1
runc --version
runc version 1.3.4-0ubuntu1~24.04.1
systemctl is-enabled containerd
enabled
systemctl is-active containerd
active

containerd config default
version = 3
default_runtime_name = 'runc'
runtime_type = 'io.containerd.runc.v2'
SystemdCgroup = false

grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

systemctl is-active containerd
active

sudo ctr namespaces list
NAME LABELS

cat /var/log/syslog  | grep "cni config"
2026-06-06T06:06:06.754273+00:00 k8s-master-01 containerd[33437]: time="2026-06-06T06:06:06.754240277Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"

sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip
io.containerd.snapshotter.v1              native                   linux/amd64    ok
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok

-rw-r--r-- 1 root root 19M июн  6 06:18 crictl-v1.34.0-linux-amd64.tar.gz
crictl

command -v crictl
/usr/local/bin/crictl

crictl --version
crictl version v1.34.0

ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 авг 21  2025 /usr/local/bin/crictl

cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

sudo crictl info | grep -B 5 -i "runtimeready"
    "conditions":
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"

sudo crictl ps
sudo crictl ps -a
sudo crictl images
sudo crictl pods
empty

```

**k8s-worker-01**
```
containerd --version
containerd github.com/containerd/containerd/v2 2.2.1
runc --version
runc version 1.3.4-0ubuntu1~24.04.1
systemctl is-enabled containerd
enabled
systemctl is-active containerd
active

containerd config default
version = 3
default_runtime_name = 'runc'
runtime_type = 'io.containerd.runc.v2'
SystemdCgroup = false

grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

systemctl is-active containerd
active

sudo ctr namespaces list
NAME LABELS

cat /var/log/syslog  | grep "cni config"
2026-06-06T06:26:19.122829+00:00 k8s-worker-01 containerd[32917]: time="2026-06-06T06:26:19.122795909Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"

sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip
io.containerd.snapshotter.v1              native                   linux/amd64    ok
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok

-rw-r--r-- 1 root root 19M июн  6 06:18 crictl-v1.34.0-linux-amd64.tar.gz
crictl

command -v crictl
/usr/local/bin/crictl

crictl --version
crictl version v1.34.0

ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 авг 21  2025 /usr/local/bin/crictl

cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

sudo crictl info | grep -B 5 -i "runtimeready"
    "conditions":
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"

sudo crictl ps
sudo crictl ps -a
sudo crictl images
sudo crictl pods
empty

```

**k8s-worker-02**
```
containerd --version
containerd github.com/containerd/containerd/v2 2.2.1
runc --version
runc version 1.3.4-0ubuntu1~24.04.1
systemctl is-enabled containerd
enabled
systemctl is-active containerd
active

containerd config default
version = 3
default_runtime_name = 'runc'
runtime_type = 'io.containerd.runc.v2'
SystemdCgroup = false

grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

systemctl is-active containerd
active

sudo ctr namespaces list
NAME LABELS

cat /var/log/syslog  | grep "cni config"
2026-06-06T06:30:33.451719+00:00 k8s-worker-02 containerd[33041]: time="2026-06-06T06:30:33.451656311Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"

sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip
io.containerd.snapshotter.v1              native                   linux/amd64    ok
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip
io.containerd.runtime.v2                  task                     linux/amd64    ok
io.containerd.cri.v1                      images                   -              ok
io.containerd.cri.v1                      runtime                  linux/amd64    ok
io.containerd.grpc.v1                     cri                      -              ok

-rw-r--r-- 1 root root 19M июн  6 06:18 crictl-v1.34.0-linux-amd64.tar.gz
crictl

command -v crictl
/usr/local/bin/crictl

crictl --version
crictl version v1.34.0

ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 авг 21  2025 /usr/local/bin/crictl

cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

 sudo crictl info | grep -B 5 -i "runtimeready"
    "conditions":
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"

sudo crictl ps
sudo crictl ps -a
sudo crictl images
sudo crictl pods
empty

```

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  критичных ошибок не обнаружено |

### Вопросы

- На данный момент все понятно
