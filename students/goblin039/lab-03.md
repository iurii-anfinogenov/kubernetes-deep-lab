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
| Lab 02 - Container Runtime: containerd | Done | | |

## Lab 03 - kubeadm prerequisites

### Дата

2026-05-31

### Цель

Подготовить все nodes к созданию Kubernetes cluster через kubeadm.

### Что было сделано

- Подключение Kubernetes apt repository
- установка пакетов kubelet kubeadm kubectl kubernetes-cni

### Команды

**deep-cp-01**:  
```
root@deep-wrk-02:/home/goblin# mkdir -p ~/tmp ; cd ~/tmp
root@deep-wrk-02:~/tmp# curl -fL -O https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 73.1M  100 73.1M    0     0  4467k      0  0:00:16  0:00:16 --:--:-- 11.1M
root@deep-wrk-02:~/tmp# curl -fL -O https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   104  100   104    0     0    915      0 --:--:-- --:--:-- --:--:--   920
root@deep-wrk-02:~/tmp# sha256sum -c kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
kdl-k8s-packages-v1.36.1-amd64.tar.gz: OK

root@deep-wrk-02:~/tmp# scp ./kdl-k8s-packages-v1.36.1-amd64.tar.gz goblin@192.168.100.51:/home/goblin
The authenticity of host '192.168.100.51 (192.168.100.51)' can't be established.
ED25519 key fingerprint is SHA256:VSxLHFaIgZQwiADnHckoyDQpKOXXcBezqYKzU62eu5c.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.100.51' (ED25519) to the list of known hosts.
goblin@192.168.100.51's password: 
kdl-k8s-packages-v1.36.1-amd64.tar.gz                                                                                                                                                         100%   73MB 196.7MB/s   00:00    

root@deep-wrk-02:~/tmp# scp ./kdl-k8s-packages-v1.36.1-amd64.tar.gz goblin@192.168.100.52:/home/goblin
The authenticity of host '192.168.100.52 (192.168.100.52)' can't be established.
ED25519 key fingerprint is SHA256:VSxLHFaIgZQwiADnHckoyDQpKOXXcBezqYKzU62eu5c.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.100.52' (ED25519) to the list of known hosts.
goblin@192.168.100.52's password: 
kdl-k8s-packages-v1.36.1-amd64.tar.gz                                                                                                                                                         100%   73MB 176.8MB/s   00:00

sudo apt install -y ./*.deb 
<много вывода :)>

root@deep-cp-01:/home/goblin/v1.36.1-amd64# dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
ii  kubeadm                               1.36.1-1.1                        amd64        Command-line utility for administering a Kubernetes cluster
ii  kubectl                               1.36.1-1.1                        amd64        Command-line utility for interacting with a Kubernetes cluster
ii  kubelet                               1.36.1-1.1                        amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                         amd64        Binaries required to provision kubernetes container networking
root@deep-cp-01:/home/goblin/v1.36.1-amd64# sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# tar -xzf kdl-k8s-packages-v1.36.1-amd64.tar.gz
root@deep-wrk-01:/home/goblin# cd v1.36.1-amd64
root@deep-wrk-01:/home/goblin# sudo apt install -y ./*.deb
<много вывода :)>

root@deep-wrk-01:/home/goblin/v1.36.1-amd64# dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
ii  kubeadm                               1.36.1-1.1                        amd64        Command-line utility for administering a Kubernetes cluster
ii  kubectl                               1.36.1-1.1                        amd64        Command-line utility for interacting with a Kubernetes cluster
ii  kubelet                               1.36.1-1.1                        amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                         amd64        Binaries required to provision kubernetes container networking
root@deep-wrk-01:/home/goblin/v1.36.1-amd64# sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# tar -xzf kdl-k8s-packages-v1.36.1-amd64.tar.gz
root@deep-wrk-02:/home/goblin# cd v1.36.1-amd64
root@deep-wrk-02:/home/goblin# sudo apt install -y ./*.deb
<много вывода :)>

root@deep-wrk-02:/home/goblin/v1.36.1-amd64# dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
ii  kubeadm                               1.36.1-1.1                        amd64        Command-line utility for administering a Kubernetes cluster
ii  kubectl                               1.36.1-1.1                        amd64        Command-line utility for interacting with a Kubernetes cluster
ii  kubelet                               1.36.1-1.1                        amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                         amd64        Binaries required to provision kubernetes container networking
root@deep-wrk-02:/home/goblin/v1.36.1-amd64# sudo apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold
```

### Ключевые выводы команд

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
hi  kubeadm                               1.36.1-1.1                        amd64        Command-line utility for administering a Kubernetes cluster
hi  kubectl                               1.36.1-1.1                        amd64        Command-line utility for interacting with a Kubernetes cluster
hi  kubelet                               1.36.1-1.1                        amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                         amd64        Binaries required to provision kubernetes container networking

root@deep-cp-01:/home/goblin# kubeadm version ; kubelet --version ; kubectl version --client=true
kubeadm version: &version.Info{Major:"1", Minor:"36", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.36.1", GitCommit:"756939600b9a7180fc2df6550a4585b638875e67", GitTreeState:"clean", BuildDate:"2026-05-12T09:53:52Z", GoVersion:"go1.26.2", Compiler:"gc", Platform:"linux/amd64"}
Kubernetes v1.36.1
Client Version: v1.36.1
Kustomize Version: v5.8.1

root@deep-cp-01:/home/goblin# apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
kubeadm
kubectl
kubelet

root@deep-cp-01:/home/goblin# systemctl is-enabled kubelet
enabled

root@deep-cp-01:/home/goblin# systemctl status kubelet --no-pager || true
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead)
       Docs: https://kubernetes.io/docs/

root@deep-cp-01:/home/goblin# crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"
```
Вывод на нодах **deep-wrk-01** и **deep-wrk-01** аналогичен. 


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| проблемы со скачиванием с репозитормя kubernetes |  |  | offline вариант установки |

### Что стало понятнее

- доступ к репозитория kubernetes так и не появился :( 

### Статус

Done
