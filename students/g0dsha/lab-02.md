# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

## Участник

| Поле | Значение |
|---|---|
| Имя | Игорь |
| GitHub | https://github.com/g0dsha |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | done |  |  |
| Lab 01 - Node baseline | done |  |  |
| Lab 02 - Container Runtime: containerd | started |  |  |

## Lab 02 - Container Runtime: containerd

### Дата

2026-05-30

### Цель

Установка на всех нодах Container Runtime: containerd и утилиты crictl.

### Что было сделано

- Установка на всех нодах containerd
- Установка на всех нодах crictl
- Проверки состояния

### Команды

#### Проверка доступных пакетов
```
user@control-plane-1:~$ apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2

user@control-plane-1:~$ apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
```
```
user@worker-1:~$ apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2

user@worker-1:~$ apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1  
```
```
user@worker-2:~$ apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2

user@worker-2:~$ apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1  
```
#### Установка containerd и runc
На всех нодах
```
sudo apt update
sudo apt install -y containerd runc
```
Проверка
```
user@control-plane-1:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
user@control-plane-1:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
user@control-plane-1:~$ systemctl is-enabled containerd
enabled
user@control-plane-1:~$ systemctl is-active containerd
active

user@worker-1:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
user@worker-1:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
user@worker-1:~$ systemctl is-enabled containerd
enabled
user@worker-1:~$ systemctl is-active containerd
active

user@worker-2:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
user@worker-2:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
user@worker-2:~$ systemctl is-enabled containerd
enabled
user@worker-2:~$ systemctl is-active containerd
active
```
#### Проверка default config containerd
```
user@control-plane-1:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
user@control-plane-1:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]

user@worker-1:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
user@worker-1:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]

user@worker-2:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
user@worker-2:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]
```
#### Настройка SystemdCgroup
На всех нодах
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo journalctl -u containerd --no-pager -n 20
```
Проверка
```
user@control-plane-1:~$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
user@control-plane-1:~$ systemctl is-active containerd
active

user@worker-1:~$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
user@worker-1:~$ systemctl is-active containerd
active

user@worker-2:~$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
user@worker-2:~$ systemctl is-active containerd
active
```
В логах присутвует сообщение **failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config** - это ожидаемо до установки CNI

#### Проверка containerd через ctr
```
user@control-plane-1:~$ sudo ctr namespaces list
NAME LABELS 

user@control-plane-1:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok
```
```
user@worker-1:~$ sudo ctr namespaces list
NAME LABELS 

user@worker-1:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok
```
```
user@worker-2:~$ sudo ctr namespaces list
NAME LABELS 

user@worker-2:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok
```
#### Установка и настройка crictl
Предварительная проверка на всех нодах аналогично
```
user@control-plane-1:~$ command -v crictl || true
user@control-plane-1:~$ CRICTL_VERSION="v1.34.0"
user@control-plane-1:~$ curl -I --connect-timeout 10 -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
HTTP/2 302
...
HTTP/2 200
```
Скачивание и проверка архива на всех нодах аналогично
```
user@control-plane-1:~$ CRICTL_VERSION="v1.34.0"
user@control-plane-1:~$ TMP_DIR="$(mktemp -d)"
user@control-plane-1:~$ curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 18.7M  100 18.7M    0     0  12.7M      0  0:00:01  0:00:01 --:--:-- 22.4M
user@control-plane-1:~$ ls -lh "${TMP_DIR}"
total 19M
-rw-rw-r-- 1 user user 19M May 30 15:57 crictl-v1.34.0-linux-amd64.tar.gz
user@control-plane-1:~$ tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
crictl
```
Установка crictl на всех нодах аналогично
```
user@control-plane-1:~$ sudo tar -C /usr/local/bin -xzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
user@control-plane-1:~$ sudo chmod 0755 /usr/local/bin/crictl
user@control-plane-1:~$ sudo chown root:root /usr/local/bin/crictl
```
Проверка
```
user@control-plane-1:~$ command -v crictl
/usr/local/bin/crictl
user@control-plane-1:~$ crictl --version
crictl version v1.34.0
user@control-plane-1:~$ ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

user@worker-1:~$ command -v crictl
/usr/local/bin/crictl
user@worker-1:~$ crictl --version
crictl version v1.34.0
user@worker-1:~$ ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

user@worker-2:~$ command -v crictl
/usr/local/bin/crictl
user@worker-2:~$ crictl --version
crictl version v1.34.0
user@worker-2:~$ ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl
```
Настройка /etc/crictl.yaml на всех нодах аналогично
```
user@control-plane-1:~$ cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```
Проверка
```
user@control-plane-1:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
user@control-plane-1:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B1
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "status": true,
        "type": "RuntimeReady"
--
        "status": false,
        "type": "NetworkReady"
user@control-plane-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@control-plane-1:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@control-plane-1:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
user@control-plane-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

user@worker-1:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
user@worker-1:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B1
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "status": true,
        "type": "RuntimeReady"
--
        "status": false,
        "type": "NetworkReady"
user@worker-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@worker-1:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@worker-1:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
user@worker-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

user@worker-2:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
user@worker-2:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B1
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "status": true,
        "type": "RuntimeReady"
--
        "status": false,
        "type": "NetworkReady"
user@worker-2:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@worker-2:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
user@worker-2:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
user@worker-2:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```
NetworkReady=false ожидаем, так как CNI plugin еще не установлен, а Pod network еще не настроен.

### Ключевые выводы команд

- containerd установлен
- runc установлен
- containerd enabled
- containerd active
- SystemdCgroup = true
- overlayfs snapshotter ok
- CRI plugin ok
- crictl установлен и настроен
- RuntimeReady = true

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

- ctr не является поддерживаемой утилитой, это инструмент отладки, диагностики. Для ручной работы лучше использовать crictl, он показывает информацию в контексте Kubernetes.


### Вопросы

-

### Статус

done
