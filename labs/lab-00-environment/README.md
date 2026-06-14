# Lab 00 - Environment Validation

Нулевая лабораторная проверяет готовность стенда и GitHub workflow.

Цель этой лабораторной - не установить Kubernetes, а подтвердить, что участник готов к формату Kubernetes Deep Lab.

## Что нужно подготовить

Участник должен подготовить описание своего лабораторного стенда и подтвердить, что может работать через GitHub.

## Требования

Перед выполнением задания участник должен иметь:

- GitHub account
- возможность поднять минимум 2 VM
- SSH access к VM
- internet access с VM
- базовое понимание Linux shell
- готовность вести lab journal

## Что не нужно делать

На этом этапе не нужно:

- устанавливать Kubernetes
- устанавливать kubeadm
- устанавливать containerd
- устанавливать CNI
- настраивать Helm
- ставить Argo CD
- автоматизировать стенд через Ansible
- использовать готовый Kubernetes-дистрибутив

Нам нужны чистые VM или возможность быстро подготовить чистые VM, чтобы все участники прошли одинаковый путь.

## Минимальный стенд

| Node | CPU | RAM | Disk |
|---|---:|---:|---:|
| control-plane-1 | 2 vCPU | 4 GB | 30 GB |
| worker-1 | 2 vCPU | 4 GB | 40 GB |

## Рекомендуемый стенд

| Node | CPU | RAM | Disk |
|---|---:|---:|---:|
| control-plane-1 | 2 vCPU | 4 GB | 30 GB |
| worker-1 | 2 vCPU | 4 GB | 40 GB |
| worker-2 | 2 vCPU | 4 GB | 40 GB |

## Опционально: Proxmox helper repository

Если участник использует Proxmox, можно подготовить VM вручную или воспользоваться helper repository:

- GitHub: https://github.com/iurii-anfinogenov/opentofu-proxmox-k8s-lab
- SourceCraft mirror: https://sourcecraft.dev/anfinogenov/opentofu-proxmox-k8s-lab

Это не обязательный способ подготовки стенда.

Важно:

- OpenTofu используется только для создания VM.
- Kubernetes заранее не устанавливаем.
- kubeadm, containerd, CNI, Helm и Argo CD не ставим до соответствующих лабораторных.
- Если repository не подходит под конкретный Proxmox-стенд, участник может создать VM вручную.

Критерий готовности остается тем же: чистые VM, SSH-доступ, сеть между нодами, internet access и заполненный `environment.md`.

## Что нужно сделать

1. Сделать fork репозитория.
2. Создать папку students/<github-username>/.
3. Скопировать шаблон students/template/environment.md.
4. Заполнить свой файл students/<github-username>/environment.md.
5. Открыть Pull Request.

## Пример

    students/g0dsha/environment.md

## Команды проверки

Выполнить на каждой VM и вставить вывод в environment.md:

```sh
hostnamectl

ip -br addr

free -h

df -h /

nproc
```

## Критерий выполнения

Lab 00 считается выполненной, если участник предоставил:

- GitHub username
- описание платформы
- список VM
- CPU/RAM/Disk по каждой VM
- OS version
- hostname
- IP
- подтверждение SSH access
- подтверждение сетевой связности между VM
- подтверждение internet access
- готовность начать с чистых VM

## Критерий отбора

После Lab 00 кандидат получает один из статусов:

| Status | Meaning |
|---|---|
| accepted | готов к участию |
| waitlist | можно подключить позже |
| blocked | есть технический блокер |
| declined | формат не подходит |
