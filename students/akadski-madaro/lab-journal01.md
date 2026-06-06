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
| Lab 00 - Environment Validation | done |  |  |
| Lab o1 - Environment Validation | | | |

## Шаблон записи по лабораторной

## Lab XX - Название

### Дата

2026-05-31

### Цель

Кратко описать цель лабораторной.

### Что было сделано

## Проверка DNS и internet access:
```
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
PING 192.168.88.51 (192.168.88.51) 56(84) bytes of data.
64 bytes from 192.168.88.51: icmp_seq=1 ttl=64 time=0.027 ms
```
## Проверка доступности package repositories

Ошибок не получил
Пакеты доступны

## Проверка time sync
```
timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
               Local time: Sun 2026-05-31 07:24:24 UTC
           Universal time: Sun 2026-05-31 07:24:24 UTC
                 RTC time: Sun 2026-05-31 07:46:57
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
active
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Sun 2026-05-31 07:24:24 UTC
На всех машинах
```
## Ручное исправление /etc/hosts

Выполнено на всех машинах

## Проверка node-to-node L3 connectivity

Done

## Проверка swap

Выключен на всех машинах

## Persistent настройка kernel modules и sysctl

```
overlay
br_netfilter
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

Узнал как выполнить первоначальную подготовку\настройку виртуальных машин для дальнейшей установки\настройки kubernetes

### Вопросы

Нет вопросов

### Статус

/ done /
