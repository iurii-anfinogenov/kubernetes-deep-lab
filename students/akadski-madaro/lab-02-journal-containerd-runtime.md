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
| Lab 02 containerd runtime | finish |  |  |

## Шаблон записи по лабораторной

## Lab XX - Название

### Дата

2026-06-04

### Цель

Установка и настройка ПО containerd
Установка и настройка crictl

### Что было сделано

containerd установлен
runc установлен

Установка согласованного cgroup driver

Установка crictl version v1.34.0

Создание и изменение файла crictl.yaml

### Ключевые выводы команд

Выполнено на всех трёх машинах:

Команда: containerd --version
runc --version
systemctl is-enabled containerd
systemctl is-active containerd:

Вывод: containerd github.com/containerd/containerd/v2 2.2.1
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
enabled
active

Команда: grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml

Вывод: 1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true

Команда: sudo ctr namespaces list
sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'

Вывод:

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

### Ошибки и диагностика

Ошибки отсутствуют

### Что стало понятнее

Как установить и настроить ПО containerd и подготовить вм к установке kubelet

### Вопросы

Нет вопросов

### Статус

/ done /
