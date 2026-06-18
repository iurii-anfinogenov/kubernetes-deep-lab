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
| Lab 02 - Container Runtime: containerd | done |  |  |
| Lab 03 - kubeadm prerequisites | done |  |  |

## Lab 03 - kubeadm prerequisites

### Дата

2026-06-09

### Цель

Подготовить все nodes к созданию Kubernetes cluster через kubeadm.

### Что было сделано

- Проверка текущего состояния
- Установка зависимостей для apt repository
- Подключение Kubernetes apt repository
- Установка kubelet, kubeadm и kubectl
- Проверка kubelet до bootstrap
- Финальная проверка

### Команды

#### Проверить состояние
На всех нодах
```
user@control-plane-1:~$ hostname
control-plane-1
user@control-plane-1:~$ cat /etc/os-release | grep -E '^(PRETTY_NAME|VERSION_ID)='
PRETTY_NAME="Ubuntu 24.04.4 LTS"
VERSION_ID="24.04"
user@control-plane-1:~$ uname -r
6.8.0-124-generic
user@control-plane-1:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
user@control-plane-1:~$ crictl --version
crictl version v1.34.0
user@control-plane-1:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
user@control-plane-1:~$ sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"
user@control-plane-1:~$ swapon --show
user@control-plane-1:~$ lsmod | grep -E 'br_netfilter|overlay' || true
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
user@control-plane-1:~$ sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward 2>/dev/null || true
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```
#### Установка зависимостей
На всех нодах
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
user@control-plane-1:~$ dpkg -l | grep -E 'apt-transport-https|ca-certificates|curl|gpg'
ii  apt-transport-https             2.8.3                                            all          transitional package for https support
ii  ca-certificates                 20240203                                         all          Common CA certificates
ii  curl                            8.5.0-2ubuntu10.9                                amd64        command line tool for transferring data with URL syntax
ii  gpg                             2.4.4-2ubuntu17.4                                amd64        GNU Privacy Guard -- minimalist public key operations
ii  gpg-agent                       2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - cryptographic agent
ii  gpg-wks-client                  2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - Web Key Service client
ii  gpgconf                         2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - core configuration utilities
ii  gpgsm                           2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - S/MIME version
ii  gpgv                            2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - signature verification tool
ii  libcurl3t64-gnutls:amd64        8.5.0-2ubuntu10.9                                amd64        easy-to-use client-side URL transfer library (GnuTLS flavour)
ii  libcurl4t64:amd64               8.5.0-2ubuntu10.9                                amd64        easy-to-use client-side URL transfer library (OpenSSL flavour)
ii  libgpg-error-l10n               1.47-3build2.1                                   all          library of error values and messages in GnuPG (localization files)
ii  libgpg-error0:amd64             1.47-3build2.1                                   amd64        GnuPG development runtime library
ii  libgpgme11t64:amd64             1.18.0-4.1ubuntu4                                amd64        GPGME - GnuPG Made Easy (library)
```
#### Подключение Kubernetes apt repository
На всех нодах
```
user@control-plane-1:~$ sudo mkdir -p -m 755 /etc/apt/keyrings
user@control-plane-1:~$ curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.36/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
user@control-plane-1:~$ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
user@control-plane-1:~$ cat /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.36/deb/ /
user@control-plane-1:~$ ls -l /etc/apt/keyrings/kubernetes-apt-keyring.gpg
-rw-r--r-- 1 root root 1200 Jun  9 17:57 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
#### Проверка доступных версий
На каждой ноде
```
sudo apt-get update
user@control-plane-1:~$ apt-cache madison kubeadm
   kubeadm | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubeadm | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
user@control-plane-1:~$ apt-cache madison kubelet
   kubelet | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubelet | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
user@control-plane-1:~$ apt-cache madison kubectl
   kubectl | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubectl | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
```
#### Установка kubelet, kubeadm и kubectl
На всех нодах выполнить
```
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
Проверка на всех нодах
```
user@worker-2:~$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"36", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.36.1", GitCommit:"756939600b9a7180fc2df6550a4585b638875e67", GitTreeState:"clean", BuildDate:"2026-05-12T09:53:52Z", GoVersion:"go1.26.2", Compiler:"gc", Platform:"linux/amd64"}
user@worker-2:~$ kubelet --version
Kubernetes v1.36.1
user@worker-2:~$ kubectl version --client=true
Client Version: v1.36.1
Kustomize Version: v5.8.1
user@worker-2:~$ apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
kubeadm
kubectl
kubelet
```
#### Проверка kubelet до bootstrap
Выполнить проверку на всех нодах
```
user@control-plane-1:~$ systemctl status kubelet --no-pager || true
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead)
       Docs: https://kubernetes.io/docs/
user@control-plane-1:~$ sudo journalctl -u kubelet
-- No entries --
user@control-plane-1:~$ sudo systemctl start kubelet
user@control-plane-1:~$ sudo systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Tue 2026-06-09 18:39:56 MSK; 5s ago
       Docs: https://kubernetes.io/docs/
    Process: 37507 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 37507 (code=exited, status=1/FAILURE)
        CPU: 67ms
user@control-plane-1:~$ sudo journalctl -u kubelet -n 5 --no-pager
Jun 09 18:43:11 control-plane-1 systemd[1]: Started kubelet.service - kubelet: The Kubernetes Node Agent.
Jun 09 18:43:11 control-plane-1 (kubelet)[37667]: kubelet.service: Referenced but unset environment variable evaluates to an empty string: KUBELET_KUBEADM_ARGS
Jun 09 18:43:11 control-plane-1 kubelet[37667]: E0609 18:43:11.211176   37667 run.go:72] "command failed" err="failed to load kubelet config file, path: /var/lib/kubelet/config.yaml, error: failed to load Kubelet config file /var/lib/kubelet/config.yaml, error failed to read kubelet config file \"/var/lib/kubelet/config.yaml\", error: open /var/lib/kubelet/config.yaml: no such file or directory"
Jun 09 18:43:11 control-plane-1 systemd[1]: kubelet.service: Main process exited, code=exited, status=1/FAILURE
Jun 09 18:43:11 control-plane-1 systemd[1]: kubelet.service: Failed with result 'exit-code'.      
```
#### Проверка файлов kubelet
Выполнить проверку на всех нодах
```
user@control-plane-1:~$ ls -l /usr/lib/systemd/system/kubelet.service.d/ || true
total 4
-rw-r--r-- 1 root root 898 May 12 13:15 10-kubeadm.conf
user@control-plane-1:~$ ls -l /var/lib/kubelet/ || true
total 0
user@control-plane-1:~$ cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```
#### Финальная проверка
На всех нодах выполнить
```
hostname
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E -B1 '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
```
На всех нодах вывод идентичный
```
v1.36.1
Kubernetes v1.36.1
Client Version: v1.36.1
Kustomize Version: v5.8.1
kubeadm
kubectl
kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Tue 2026-06-09 18:54:27 MSK; 3s ago
       Docs: https://kubernetes.io/docs/
    Process: 38346 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 38346 (code=exited, status=1/FAILURE)
        CPU: 59ms
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
```

### Ключевые выводы команд

 - Состояние соответсвует ожидаемому
 - Зависимости для apt repository установлены
 - Подключен Kubernetes apt repository
 - kubeadm, kubelet, kubectl доступны из repository pkgs.k8s.io
 - kubeadm установлен
 - kubelet установлен
 - kubectl установлен
 - все три пакета находятся в hold
 - сервис kubelet установлен, но был не активен, после запуска постоянно падает, это ожидаемое поведение, до bootstrap
 - конфигурация kubelet минимальна, полноценная появится после bootstrap node
 - NetworkReady в false до установки CNI.

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| kubelet после установке был неактивен |  | Попробовал запустить вручную | после запуска постоянно падает, это ожидаемое поведение, до bootstrap |

### Что стало понятнее

-

### Вопросы

- 

### Статус

done
