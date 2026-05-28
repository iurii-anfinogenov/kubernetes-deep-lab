# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Вячеслав  |
| GitHub | goblin039 |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | Done |  |  |
| Lab 01 - Node Baseline | Done |  |  |

## Lab 02 - Container Runtime: containerd

### Дата

2026-05-30

### Цель

Установить cri.

### Что было сделано

- Проверка доступных пакетов
- Установка containerd и runc
- Настройка SystemdCgroup
- Проверка containerd через ctr
- Установка и настройка crictl

### Команды

Проверка доступных пакетов  
**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# sudo apt update
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
root@deep-cp-01:/home/goblin# apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
root@deep-cp-01:/home/goblin# apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# apt update
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
root@deep-wrk-01:/home/goblin# apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
root@deep-wrk-01:/home/goblin# apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# apt update
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:3 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:4 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
root@deep-wrk-02:/home/goblin# apt-cache policy containerd
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
root@deep-wrk-02:/home/goblin# apt-cache policy runc
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

-----

Установка containerd и runc  
**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# sudo apt install -y containerd runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  containerd runc
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 37.7 MB of archives.
After this operation, 140 MB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 runc amd64 1.3.4-0ubuntu1~24.04.1 [9574 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 containerd amd64 2.2.1-0ubuntu1~24.04.2 [28.1 MB]
Fetched 37.7 MB in 4s (8681 kB/s)     
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package runc.
(Reading database ... 75094 files and directories currently installed.)
Preparing to unpack .../runc_1.3.4-0ubuntu1~24.04.1_amd64.deb ...
Unpacking runc (1.3.4-0ubuntu1~24.04.1) ...
Selecting previously unselected package containerd.
Preparing to unpack .../containerd_2.2.1-0ubuntu1~24.04.2_amd64.deb ...
Unpacking containerd (2.2.1-0ubuntu1~24.04.2) ...
Setting up runc (1.3.4-0ubuntu1~24.04.1) ...
Setting up containerd (2.2.1-0ubuntu1~24.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /usr/lib/systemd/system/containerd.service.

root@deep-cp-01:/home/goblin# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
root@deep-cp-01:/home/goblin# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
root@deep-cp-01:/home/goblin# systemctl is-enabled containerd
enabled
root@deep-cp-01:/home/goblin# systemctl is-active containerd
active
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# sudo apt install -y containerd runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  containerd runc
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 37.7 MB of archives.
After this operation, 140 MB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 runc amd64 1.3.4-0ubuntu1~24.04.1 [9574 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 containerd amd64 2.2.1-0ubuntu1~24.04.2 [28.1 MB]
Fetched 37.7 MB in 5s (7515 kB/s)     
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package runc.
(Reading database ... 75094 files and directories currently installed.)
Preparing to unpack .../runc_1.3.4-0ubuntu1~24.04.1_amd64.deb ...
Unpacking runc (1.3.4-0ubuntu1~24.04.1) ...
Selecting previously unselected package containerd.
Preparing to unpack .../containerd_2.2.1-0ubuntu1~24.04.2_amd64.deb ...
Unpacking containerd (2.2.1-0ubuntu1~24.04.2) ...
Setting up runc (1.3.4-0ubuntu1~24.04.1) ...
Setting up containerd (2.2.1-0ubuntu1~24.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /usr/lib/systemd/system/containerd.service.

root@deep-wrk-01:/home/goblin# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
root@deep-wrk-01:/home/goblin# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
root@deep-wrk-01:/home/goblin# systemctl is-enabled containerd
enabled
root@deep-wrk-01:/home/goblin# systemctl is-active containerd
active
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# sudo apt install -y containerd runc
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  containerd runc
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 37.7 MB of archives.
After this operation, 140 MB of additional disk space will be used.
Get:1 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 runc amd64 1.3.4-0ubuntu1~24.04.1 [9574 kB]
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 containerd amd64 2.2.1-0ubuntu1~24.04.2 [28.1 MB]
Fetched 37.7 MB in 5s (7531 kB/s)     
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package runc.
(Reading database ... 75094 files and directories currently installed.)
Preparing to unpack .../runc_1.3.4-0ubuntu1~24.04.1_amd64.deb ...
Unpacking runc (1.3.4-0ubuntu1~24.04.1) ...
Selecting previously unselected package containerd.
Preparing to unpack .../containerd_2.2.1-0ubuntu1~24.04.2_amd64.deb ...
Unpacking containerd (2.2.1-0ubuntu1~24.04.2) ...
Setting up runc (1.3.4-0ubuntu1~24.04.1) ...
Setting up containerd (2.2.1-0ubuntu1~24.04.2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /usr/lib/systemd/system/containerd.service.

root@deep-wrk-02:/home/goblin# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
root@deep-wrk-02:/home/goblin# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
root@deep-wrk-02:/home/goblin# systemctl is-enabled containerd
enabled
root@deep-wrk-02:/home/goblin# systemctl is-active containerd
active
```

-----

Настройка SystemdCgroup  

**deep-cp-01**:  
```
root@deep-cp-01:/etc# mkdir -p /etc/containerd
root@deep-cp-01:/etc# containerd config default > /etc/containerd/config.toml
root@deep-cp-01:/etc# sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
root@deep-cp-01:/etc# systemctl restart containerd
root@deep-cp-01:/etc# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# mkdir -p /etc/containerd
root@deep-wrk-01:/home/goblin# containerd config default > /etc/containerd/config.toml
root@deep-wrk-01:/home/goblin# sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
root@deep-wrk-01:/home/goblin# systemctl restart containerd
root@deep-wrk-01:/home/goblin# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# mkdir -p /etc/containerd
root@deep-wrk-02:/home/goblin# containerd config default > /etc/containerd/config.toml
root@deep-wrk-02:/home/goblin# sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
root@deep-wrk-02:/home/goblin# systemctl restart containerd
root@deep-wrk-02:/home/goblin# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
```

-----

Проверка containerd через ctr
**deep-cp-01**:  
```
root@deep-cp-01:/etc# ctr namespaces list
NAME LABELS 
root@deep-cp-01:/etc# ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok 
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# ctr namespaces list
NAME LABELS 
root@deep-wrk-01:/home/goblin# ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# ctr namespaces list
NAME LABELS 
root@deep-wrk-02:/home/goblin# ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok
```

-----

Установка и настройка crictl

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.34.0/crictl-v1.34.0-linux-amd64.tar.gz
HTTP request sent, awaiting response... 200 OK
Length: 19610258 (19M) [application/octet-stream]
Saving to: ‘crictl-v1.34.0-linux-amd64.tar.gz’

crictl-v1.34.0-linux-amd64.tar.gz                       100%[===============================================================================================================================>]  18.70M  10.6MB/s    in 1.8s    

2026-05-28 11:51:45 (10.6 MB/s) - ‘crictl-v1.34.0-linux-amd64.tar.gz’ saved [19610258/19610258]

root@deep-cp-01:/home/goblin# tar -xzvf crictl-v1.34.0-linux-amd64.tar.gz
crictl

root@deep-cp-01:/home/goblin# chmod 0755 crictl
root@deep-cp-01:/home/goblin# chown root:root crictl
root@deep-cp-01:/home/goblin# mv ./crictl /usr/local/bin/crictl
root@deep-cp-01:/home/goblin# rm -rf crictl-v1.34.0-linux-amd64.tar.gz
root@deep-cp-01:/home/goblin# crictl --version
crictl version v1.34.0
root@deep-cp-01:/home/goblin# ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF

root@deep-cp-01:/home/goblin# crictl info | grep -E 'runtime|SystemCgroup|Runtime|NetworkReady'
      "defaultRuntimeName": "runc",
      "runtimes": {
          "baseRuntimeSpec": "",
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
  "runtimeHandlers": [
        "type": "RuntimeReady"
        "type": "NetworkReady"

root@deep-cp-01:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-cp-01:/home/goblin# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-cp-01:/home/goblin# crictl images
IMAGE               TAG                 IMAGE ID            SIZE
root@deep-cp-01:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```
**deep-wrk-01**:  
```
 wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.34.0/crictl-v1.34.0-linux-amd64.tar.gz
 HTTP request sent, awaiting response... 200 OK
Length: 19610258 (19M) [application/octet-stream]
Saving to: ‘crictl-v1.34.0-linux-amd64.tar.gz’

crictl-v1.34.0-linux-amd64.tar.gz                       100%[===============================================================================================================================>]  18.70M  10.6MB/s    in 1.8s    

2026-05-28 12:27:26 (10.6 MB/s) - ‘crictl-v1.34.0-linux-amd64.tar.gz’ saved [19610258/19610258]
root@deep-wrk-01:/home/goblin# tar -xzvf crictl-v1.34.0-linux-amd64.tar.gz
crictl
root@deep-wrk-01:/home/goblin# chmod 0755 crictl
root@deep-wrk-01:/home/goblin# chown root:root crictl
root@deep-wrk-01:/home/goblin# mv ./crictl /usr/local/bin/crictl
root@deep-wrk-01:/home/goblin# rm -rf crictl-v1.34.0-linux-amd64.tar.gz
root@deep-wrk-01:/home/goblin# crictl --version
crictl version v1.34.0
root@deep-wrk-01:/home/goblin# ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

root@deep-wrk-01:/home/goblin# cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

root@deep-wrk-01:/home/goblin# crictl info | grep -E 'runtime|SystemCgroup|Runtime|NetworkReady'
      "defaultRuntimeName": "runc",
      "runtimes": {
          "baseRuntimeSpec": "",
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
  "runtimeHandlers": [
        "type": "RuntimeReady"
        "type": "NetworkReady"

root@deep-wrk-01:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-wrk-01:/home/goblin# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-wrk-01:/home/goblin# crictl images
IMAGE               TAG                 IMAGE ID            SIZE
root@deep-wrk-01:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME        
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.34.0/crictl-v1.34.0-linux-amd64.tar.gz
HTTP request sent, awaiting response... 200 OK
Length: 19610258 (19M) [application/octet-stream]
Saving to: ‘crictl-v1.34.0-linux-amd64.tar.gz’

crictl-v1.34.0-linux-amd64.tar.gz                       100%[===============================================================================================================================>]  18.70M  10.6MB/s    in 1.8s    

2026-05-28 12:31:41 (10.6 MB/s) - ‘crictl-v1.34.0-linux-amd64.tar.gz’ saved [19610258/19610258]
root@deep-wrk-02:/home/goblin# tar -xzvf crictl-v1.34.0-linux-amd64.tar.gz
crictl
root@deep-wrk-02:/home/goblin# chmod 0755 crictl
root@deep-wrk-02:/home/goblin# chown root:root crictl
root@deep-wrk-02:/home/goblin# mv ./crictl /usr/local/bin/crictl
root@deep-wrk-02:/home/goblin# rm -rf crictl-v1.34.0-linux-amd64.tar.gz
root@deep-wrk-02:/home/goblin# crictl --version
crictl version v1.34.0
root@deep-wrk-02:/home/goblin# ls -l /usr/local/bin/crictl
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl

root@deep-wrk-02:/home/goblin# cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false

root@deep-wrk-02:/home/goblin# crictl info | grep -E 'runtime|SystemCgroup|Runtime|NetworkReady'
      "defaultRuntimeName": "runc",
      "runtimes": {
          "baseRuntimeSpec": "",
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
  "runtimeHandlers": [
        "type": "RuntimeReady"
        "type": "NetworkReady"

root@deep-wrk-02:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-wrk-02:/home/goblin# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-wrk-02:/home/goblin# crictl images
IMAGE               TAG                 IMAGE ID            SIZE
root@deep-wrk-02:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME        
```



### Ключевые выводы команд

На основе deep-cp-01, на остальных нодах вывод аналогичен.
```
root@deep-cp-01:/home/goblin# containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
root@deep-cp-01:/home/goblin# runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
root@deep-cp-01:/home/goblin# systemctl is-enabled containerd
enabled
root@deep-cp-01:/home/goblin# systemctl is-active containerd
active

root@deep-cp-01:/etc# grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true


root@deep-cp-01:/etc# ctr plugins list | grep -E 'cri|runtime|snapshotter' | grep ok
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok

root@deep-cp-01:/home/goblin# crictl --version
crictl version v1.34.0

root@deep-cp-01:/home/goblin# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-cp-01:/home/goblin# crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
root@deep-cp-01:/home/goblin# crictl images
IMAGE               TAG                 IMAGE ID            SIZE
root@deep-cp-01:/home/goblin# crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```

### Вопросы

- Есть более свежая версия 1.36, или мы будем делать upgrade версии впоследствии?

### Статус

Done


