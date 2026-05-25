# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Вадим Богомолов |
| GitHub | vadimbergamot |
| Часовой пояс | RTZ-1 (Msk-1) |
| Сколько часов в неделю готов уделять | 10  |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | local  |
| Гипервизор | HyperV|
| Где расположен стенд | home lab |
| Есть ли возможность пересоздавать VM | yes |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 | 4 | 15 |Ubuntu 22.04| kuber-cp-1.homelab.loc | 17.17.17 |
| worker-1 | worker |  2| 4 |15  |Ubuntu 22.04 |Kuber-worker-1.homelab.loc  |  |
| worker-2 | worker |  2| 4 | 15 | Ubuntu 22.04 |kuber-worker-2.homelab.loc  |  |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes  |
| IP статические или DHCP reservation | dhcp-reservation  |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже? создаются чистые машины с 0
- Есть ли уже установленный Kubernetes? нет
- Готовы ли снести текущий кластер и начать с чистых VM? да
- Есть ли ограничения по ресурсам? до 6 ядер на вм

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl

     Static hostname: kuber-cp-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 7e89fcb7459a47cc851de51faaecf80d
         Boot ID: d59cc0a072cc4e89a7e265f3550e2977
  Virtualization: microsoft
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 2w 2d

Static hostname: kuber-worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 8c73bbd1050c475c882477cf0a5b63a5
         Boot ID: c0df82ac6fac469cbf485a7545cbf694
  Virtualization: microsoft
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 2w 2d

 root@kuber-worker-2:/home/bergamot# hostnamectl
 Static hostname: kuber-worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 77c9fb0b76b74bc29a1d6f40d8b12fd8
         Boot ID: 66712931e2a446ecad4acf15ca0cc75f
  Virtualization: microsoft
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: Microsoft Corporation
  Hardware Model: Virtual Machine
Firmware Version: 090008
   Firmware Date: Fri 2018-12-07
    Firmware Age: 7y 5month 2w 2d
   


### ip -br addr

   lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.31.173.92/20 metric 100 fe80::215:5dff:fe01:805/64

lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.31.169.78/20 metric 100 fe80::215:5dff:fe01:806/64

lo               UNKNOWN        127.0.0.1/8 ::1/128
eth0             UP             172.31.160.117/20 metric 100 fe80::215:5dff:fe01:807/64


### free -h

  free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       438Mi       7.4Gi       3.7Mi       183Mi       7.4Gi
Swap:             0B          0B          0B


root@kuber-worker-1:/# free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       396Mi       2.8Gi       3.7Mi       889Mi       3.4Gi
Swap:             0B          0B          0B


 free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       392Mi       3.1Gi       3.7Mi       599Mi       3.4Gi
Swap:             0B          0B          0B



### df -h /

    :/home/bergamot# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              596M  732K  595M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   17G  2.8G   13G  18% /
tmpfs                              3.0G     0  3.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          2.0G  103M  1.7G   6% /boot
tmpfs                              596M   12K  596M   1% /run/user/1000

Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              392M  784K  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  4.2G  5.1G  46% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G  103M  1.5G   7% /boot
tmpfs                              392M   12K  392M   1% /run/user/1000

Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              392M  784K  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  4.2G  5.1G  46% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G  103M  1.5G   7% /boot
tmpfs                              392M   12K  392M   1% /run/user/1000
### nproc

     nproc
2
nproc 
2
nproc
2

## Вопросы и риски

Опишите вопросы или риски по стенду, если они есть.
