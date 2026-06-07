# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Артур |
| GitHub | akadski-madaro |
| Часовой пояс | +3 Москва |
| Сколько часов в неделю готов уделять | 5 часов |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | local |
| Гипервизор | Hyper V |
| Где расположен стенд | home lab |
| Есть ли возможность пересоздавать VM | yes |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 vCPU | 4 GB | 30gb | ubuntu | control-plane-1 | 192.168.88.51 |
| worker-1 | worker | 2 vCPU | 4gb | 30gb | ubuntu | worker-1 | 192.168.88.52 |
| worker-2 | worker | 2 vCPU | 4gb | 30gb | ubuntu | worker-2 | 192.168.88.53 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | static |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы
- Есть ли уже установленный Kubernetes? Нет
- Готовы ли снести текущий кластер и начать с чистых VM? Да
- Есть ли ограничения по ресурсам? Есть

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl

 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: f9ec4f533c59470bb3556b0c080cec96
         Boot ID: bea7cef840894b96a75704447ab54155
  Virtualization: microsoft
Operating System: Ubuntu 24.04.2 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 3w 1d

     Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 04c6884df3844032a36feae0c9d5ca74
         Boot ID: e21c07606d6741ea801cda5e508ef309
  Virtualization: microsoft
Operating System: Ubuntu 24.04.2 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 3w 1d

     Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 1f96071e0b8047f999bbbca6ed300f27
         Boot ID: 591f84da6c044eb1a0e7a4690490422f
  Virtualization: microsoft
Operating System: Ubuntu 24.04.2 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 3w 1d

### ip -br addr

control-plane-1:  
- lo               UNKNOWN        127.0.0.1/8 ::1/128
- eth0             UP             192.168.88.51/24 fe80::215:5dff:fe38:119/64

worker-1:  
- lo               UNKNOWN        127.0.0.1/8 ::1/128
- eth0             UP             192.168.88.52/24 fe80::215:5dff:fe38:11c/64

worker-1:  
- lo               UNKNOWN        127.0.0.1/8 ::1/128
- eth0             UP             192.168.88.53/24 fe80::215:5dff:fe38:11d/64


### free -h

```
# control-plane-1:
                total        used        free      shared  buff/cache   available
Mem:           573Mi       413Mi       160Mi       4.0Mi       216Mi       159Mi
Swap:          2.0Gi        12Ki       2.0Gi   
```

```
# worker-1:
               total        used        free      shared  buff/cache   available
Mem:           597Mi       418Mi       112Mi       4.0Mi       286Mi       179Mi
Swap:          3.8Gi          0B       3.8Gi
```

```
# worker-2:
               total        used        free      shared  buff/cache   available
Mem:           545Mi       375Mi       175Mi       4.0Mi       212Mi       170Mi
Swap:          2.0Gi        12Ki       2.0Gi
```

### df -h /

```
# control-plane-1:
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   38G  4.7G   31G  14% /
```
```
# worker-1:
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   38G  6.5G   29G  19% /
```
```
# worker-2:
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   38G  4.7G   31G  14% /
```
### nproc

На всех 2

## Вопросы и риски

Вопросов пока нет
