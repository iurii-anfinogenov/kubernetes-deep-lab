# Lab 03 - kubeadm prerequisites

### Дата

2026-06-05

### Цель

Подготовить все nodes к созданию Kubernetes cluster через `kubeadm`.

## вывод приложения

```
ae@worker-2:~$ hostname
worker-2
ae@worker-2:~$ dpkg --print-architecture
amd64
ae@worker-2:~$ sha256sum -c kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
kdl-k8s-packages-v1.36.1-amd64.tar.gz: OK
ae@worker-2:~$ cd v1.36.1-amd64/
ae@worker-2:~/v1.36.1-amd64$ sha256sum -c SHA256SUMS
kubeadm_1.36.1-1.1_amd64.deb: OK
kubectl_1.36.1-1.1_amd64.deb: OK
kubelet_1.36.1-1.1_amd64.deb: OK
kubernetes-cni_1.9.1-1.1_amd64.deb: OK
ae@worker-2:~/v1.36.1-amd64$ kubeadm version -o short
v1.36.1
ae@worker-2:~/v1.36.1-amd64$ kubelet --version
Kubernetes v1.36.1
ae@worker-2:~/v1.36.1-amd64$ kubectl version --client=true
Client Version: v1.36.1
Kustomize Version: v5.8.1
ae@worker-2:~/v1.36.1-amd64$ apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
kubeadm
kubectl
kubelet
ae@worker-2:~/v1.36.1-amd64$ systemctl status kubelet --no-pager || true
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead)
       Docs: https://kubernetes.io/docs/
ae@worker-2:~/v1.36.1-amd64$ sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"

```
проверка повторена на всех nodes, отличий нет, кроме хостнейма

### Что стало понятнее

Было неожиданностью, что репозиторий недоступен. Узнал как устанавливать офлайн. Узнал про механизм холда пакетов.

### Статус

done