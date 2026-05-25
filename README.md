# Kubernetes Deep Lab

Kubernetes Deep Lab - открытая инженерная лаборатория по глубокому изучению Kubernetes.

Цель проекта - не просто запускать Kubernetes-манифесты, а понимать, как Kubernetes устроен внутри: Linux, container runtime, kubeadm, control plane, kubelet, CNI, networking, storage, security, troubleshooting, upgrades, failure scenarios, GitOps и automation.

## Цели проекта

- Глубоко разобраться в Kubernetes на практике.
- Учиться через VM-based lab, а не только через kind или minikube.
- Создать открытые учебные материалы для DevOps/Linux/Kubernetes студентов.
- Запустить небольшую практическую когорту с lab reports и GitHub workflow.
- Документировать реальные troubleshooting-сценарии и выводы.
- Пройти путь от ручного понимания до production-like operations и последующей автоматизации.

## Формат

Проект состоит из двух направлений:

1. Личное глубокое изучение Kubernetes.
2. Открытая когорта для небольшой группы активных участников.

Первая когорта планируется как пилотная группа на 3-5 участников. Если людей будет меньше, проект все равно стартует в меньшем составе.

Участие бесплатное, но формат активный. Это не пассивный курс, а практическая лаборатория: участники выполняют задания, ведут lab journal, присылают выводы команд и разбирают ошибки.

## Учебный путь

Основной путь проекта:

- Linux/container runtime
- containerd
- kubeadm
- control plane internals
- kubelet and node internals
- Pod lifecycle
- workloads
- networking
- CNI
- Services and DNS
- Ingress and Gateway API
- storage and CSI
- RBAC and security
- observability
- upgrades
- failure scenarios
- GitOps
- automation

## Подход к лаборатории

Основной лабораторный стенд - VM-based environment.

kind и minikube можно использовать только для быстрых экспериментов, но не как основной путь глубокого изучения.

## Рекомендуемый стенд

| Node | CPU | RAM | Disk |
|---|---:|---:|---:|
| control-plane-1 | 2 vCPU | 4 GB | 30 GB |
| worker-1 | 2 vCPU | 4 GB | 40 GB |
| worker-2 | 2 vCPU | 4 GB | 40 GB |

## Минимальный стенд

| Node | CPU | RAM | Disk |
|---|---:|---:|---:|
| control-plane-1 | 2 vCPU | 4 GB | 30 GB |
| worker-1 | 2 vCPU | 4 GB | 40 GB |

## Базовый стек

| Component | Choice |
|---|---|
| OS | Ubuntu 24.04 LTS |
| Runtime | containerd |
| Bootstrap | kubeadm |
| First CNI | Calico |
| Later CNI deep dive | Cilium |

## Важное правило

Не устанавливайте Kubernetes заранее до соответствующей лабораторной.

Первый шаг - проверка окружения. Нам нужны чистые VM или возможность быстро пересоздать чистые VM.

До нулевого задания не нужно заранее устанавливать:

- Kubernetes
- kubeadm
- kubelet
- kubectl
- CNI
- Helm
- Argo CD
- готовые Kubernetes-дистрибутивы

## Рабочий процесс когорты

Участники работают через GitHub:

1. Делают fork этого репозитория.
2. Создают папку students/<github-username>/.
3. Добавляют environment.md.
4. Открывают Pull Request.
5. Получают review.
6. Продолжают вести lab reports.

Пример структуры участника:

    students/g0dsha/
      environment.md
      lab-journal.md

## Структура репозитория

    kubernetes-deep-lab/
      README.md
      docs/
        requirements.md
        roadmap.md
        lab-rules.md
      labs/
        lab-00-environment/
          README.md
      students/
        template/
          environment.md
          lab-journal.md

## Текущий статус

Проект готовит запуск Cohort 1.

Текущий этап:

- сбор кандидатов
- подготовка GitHub repository
- подготовка требований к VM
- подготовка нулевого задания
- подготовка шаблонов lab reports
- [Lab 00 - environment](labs/lab-00-environment/README.md)
- [Lab 01 - Container runtime baseline](./labs/lab-01-container-runtime-baseline/README.md)
- [Lab 02 - CRI diagnostics with crictl](./labs/lab-02-containerd-runtime/README.md)



## Лицензия

Будет определено позже.
