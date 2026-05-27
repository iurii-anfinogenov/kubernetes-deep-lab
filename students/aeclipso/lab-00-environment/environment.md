# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | aeclipso |
| GitHub | https://github.com/aeclipso/ |
| Часовой пояс | GMT+3 |
| Сколько часов в неделю готов уделять | 2-15 |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | local |
| Гипервизор | Proxmox |
| Где расположен стенд | home lab  |
| Есть ли возможность пересоздавать VM | yes  |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 | 4096 Mb | 30Gb | Ubuntu server 24.04 | control-plane-1 | 192.168.1.40 |
| worker-1 | worker | 2 | 4096 Mb | 40Gb | Ubuntu server 24.04 | worker-1 | 192.168.1.43 |
| worker-2 | worker | 2 | 4096 Mb | 40Gb | Ubuntu server 24.04 | worker-2 | 192.168.1.46 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | dhcp-reservation |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже?
Уже созданы.
- Есть ли уже установленный Kubernetes?
Нет
- Готовы ли снести текущий кластер и начать с чистых VM?
Да
- Есть ли ограничения по ресурсам?
8 CPU 32 Gb RAM 1Tb HDD

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl
```
ae@control-plane-1:~$ hostnamectl
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: bc9d8086e24841519e3490412e320bac
         Boot ID: 2bcf53f95ff54afe98085d51c5fb7aa8
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d    

ae@worker-1:~$ hostnamectl
 Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: bc3717c339394ea18f75f99dfb797a8a
         Boot ID: 77133109057a4ce4930a5105babe0ce9
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d           

ae@worker-2:~$ hostnamectl
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: c4fd594a622943e9b6ccd7dd01632aeb
         Boot ID: bf11be851f2e4a31a9bad06b4e2fd0a9
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d      
```
### ip -br addr
```
ae@control-plane-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.40/24 metric 100 fe80::be24:11ff:feca:26e2/64 

ae@worker-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.43/24 metric 100 fe80::be24:11ff:fe2b:39bf/64 

ae@worker-2:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.46/24 metric 100 fe80::be24:11ff:fe80:46e4/64 
```

### free -h
```
ae@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       395Mi       3.1Gi       1.0Mi       618Mi       3.4Gi
Swap:          3.8Gi          0B       3.8Gi

ae@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       392Mi       3.4Gi       1.0Mi       299Mi       3.4Gi
Swap:          3.8Gi          0B       3.8Gi

ae@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       405Mi       3.3Gi       1.0Mi       299Mi       3.4Gi
Swap:          3.8Gi          0B       3.8Gi
```
### df -h /
```
ae@control-plane-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  6.6G   22G  24% /

ae@worker-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  6.6G   31G  18% /

ae@worker-2:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  6.6G   31G  18% /


```
### nproc
```
ae@control-plane-1:~$ nproc
2

ae@worker-1:~$ nproc
2

ae@worker-2:~$ nproc
2
```

## Вопросы и риски

Не во все дни недели имею к нему доступ.
