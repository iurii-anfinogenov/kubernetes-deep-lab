# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Артур |
| GitHub | akadski-madaro |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 03 - offline install | finish |  |  |

## Шаблон записи по лабораторной

## Lab XX - Название

### Дата

2026-06-10

### Цель

Установка Kubernetes packages

### Что было сделано

Создал папку kdl-packages
В неё скачал файлы с закрытого репозитория
Установил и зафиксировал версии ПО kubelet kubeadm kubectl

### Команды

  hostname
dpkg --print-architecture
sha256sum -c kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
cd ~/kdl-packages/v1.36.1-amd64
sha256sum -c SHA256SUMS
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'

### Ключевые выводы команд

  На всех вм одинаковый вывод команды, за исключением hostname:

  worker-2
hi  kubeadm                               1.36.1-1.1                              amd64        Command-line utility for administering a Kubernetes cluster
hi  kubectl                               1.36.1-1.1                              amd64        Command-line utility for interacting with a Kubernetes cluster
hi  kubelet                               1.36.1-1.1                              amd64        Node agent for Kubernetes clusters
ii  kubernetes-cni                        1.9.1-1.1                               amd64        Binaries required to provision kubernetes container networking
v1.36.1
Kubernetes v1.36.1
Client Version: v1.36.1
Kustomize Version: v5.8.1
kubeadm
kubectl
kubelet
enabled
○ kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead)
       Docs: https://kubernetes.io/docs/
            "SystemdCgroup": true
          "runtimeType": "io.containerd.runc.v2",
        "type": "RuntimeReady"
        "type": "NetworkReady"

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

Как установить, зафиксировать версии ПО kubelet kubeadm kubectl

### Вопросы

Нет вопросов

### Статус

/ done / 
