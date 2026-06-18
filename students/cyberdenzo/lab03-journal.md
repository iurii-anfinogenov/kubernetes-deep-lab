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
| Lab 03 - kubeadm prerequisites | done |  |  |

## Lab 02 - Container Runtime

### Дата

2026-06-08

### Цель

Подготовить все nodes к созданию Kubernetes cluster через kubeadm

### Что было сделано

- Проверка текущего состояния
- Установка зависимостей для apt repository
- Подключение Kubernetes apt repository
- Установка kubelet, kubeadm и kubectl
- Проверка файлов kubelet

### Команды
**k8s-master-01** **k8s-worker-01** **k8s-worker-02**
```
PRETTY_NAME="Ubuntu 24.04.4 LTS"
VERSION_ID="24.04"
6.8.0-117-generic
containerd github.com/containerd/containerd/v2 2.2.1
crictl version v1.34.0
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

dpkg -l | grep -E 'apt-transport-https|ca-certificates|curl|gpg'
ii  apt-transport-https                   2.8.3                                            all          transitional package for https support
ii  ca-certificates                       20240203                                         all          Common CA certificates
ii  curl                                  8.5.0-2ubuntu10.9                                amd64        command line tool for transferring data with URL syntax
ii  gpg                                   2.4.4-2ubuntu17.4                                amd64        GNU Privacy Guard -- minimalist public key operations
ii  gpg-agent                             2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - cryptographic agent
ii  gpg-wks-client                        2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - Web Key Service client
ii  gpgconf                               2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - core configuration utilities
ii  gpgsm                                 2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - S/MIME version
ii  gpgv                                  2.4.4-2ubuntu17.4                                amd64        GNU privacy guard - signature verification tool
ii  libcurl3t64-gnutls:amd64              8.5.0-2ubuntu10.9                                amd64        easy-to-use client-side URL transfer library (GnuTLS flavour)
ii  libcurl4t64:amd64                     8.5.0-2ubuntu10.9                                amd64        easy-to-use client-side URL transfer library (OpenSSL flavour)
ii  libgpg-error-l10n                     1.47-3build2.1                                   all          library of error values and messages in GnuPG (localization files)
ii  libgpg-error0:amd64                   1.47-3build2.1                                   amd64        GnuPG development runtime library
ii  libgpgme11t64:amd64                   1.18.0-4.1ubuntu4                                amd64        GPGME - GnuPG Made Easy (library)

apt-cache madison kubeadm
apt-cache madison kubelet
apt-cache madison kubectl
   kubeadm | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubeadm | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubelet | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubelet | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubectl | 1.36.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages
   kubectl | 1.36.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.36/deb  Packages

dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl is-enabled kubelet
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
hi  kubeadm                               1.36.1-1.1                                       amd64        Command-line utility for administering a Kubernetes cluster
hi  kubectl                               1.36.1-1.1                                       amd64        Command-line utility for interacting with a Kubernetes cluster
hi  kubelet                               1.36.1-1.1                                       amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                                        amd64        Binaries required to provision kubernetes container networking
v1.36.1
Kubernetes v1.36.1
Client Version: v1.36.1
Kustomize Version: v5.8.1
kubeadm
kubectl
kubelet
enabled
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Mon 2026-06-08 18:59:05 UTC; 1s ago
       Docs: https://kubernetes.io/docs/
    Process: 1227 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 1227 (code=exited, status=1/FAILURE)
        CPU: 37ms
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"

Проверка повторена на всех nodes
```


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  критичных ошибок не обнаружено |

### Вопросы

- На данный момент все понятно
