# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Илья |
| GitHub | https://github.com/ilyabobrusev |
| Часовой пояс | +3 МСК |
| Сколько часов в неделю готов уделять | 4-8 |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | selfhosted |
| Гипервизор | Proxmox |
| Где расположен стенд | home lab |
| Есть ли возможность пересоздавать VM | yes |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 | 4 GB | 45 GB |  Ubuntu 24.04 LTS | control-plane-1 | 192.168.1.141 |
| worker-1 | worker | 2 | 4 GB | 55 GB | Ubuntu 24.04 LTS | worker-1 | 192.168.1.142 |
| worker-2 | worker | 2 | 4 GB | 55 GB | Ubuntu 24.04 LTS | worker-2 | 192.168.1.143 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | static |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже? Да
- Есть ли уже установленный Kubernetes? Нет
- Готовы ли снести текущий кластер и начать с чистых VM? Да
- Есть ли ограничения по ресурсам? 20 cpu, 55 gb ram, 550 gb hdd

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl
```
q@control-plane-1:~$ hostnamectl
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 9df59e63128748d9b00b26b788d1c911
         Boot ID: 0b794a6712a84e1db313a0116827f6d9
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d

q@worker-1:~$ hostnamectl
 Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: cb22ade8f8284358a12b36610e98a446
         Boot ID: 4f03d7690a2943ce9133adbf42e170f7
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d

q@worker-2:~$ hostnamectl
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 2973ed0ca33f46b0b711da9b4bd21681
         Boot ID: f034224ba7164fd89c684dde19e16ee0
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
q@control-plane-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens18            UP             192.168.1.141/24 fe80::be24:11ff:fe4c:3051/64

q@worker-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens18            UP             192.168.1.142/24 fe80::be24:11ff:fedf:96a1/64

q@worker-2:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128
ens18            UP             192.168.1.143/24 fe80::be24:11ff:fe82:9c3d/64
```

### free -h

```
q@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       363Mi       3.5Gi       684Ki       171Mi       3.5Gi
Swap:             0B          0B          0B

q@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       418Mi       3.4Gi       692Ki       244Mi       3.4Gi
Swap:             0B          0B          0B

q@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       362Mi       3.5Gi       684Ki       171Mi       3.5Gi
Swap:             0B          0B          0B
```

### df -h /

```
q@control-plane-1:~$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
tmpfs                         392M  680K  391M   1% /run
/dev/mapper/vg0-lv--root       34G  2.7G   30G   9% /
tmpfs                         2.0G     0  2.0G   0% /dev/shm
tmpfs                         5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg0-lv--var--log  9.8G  109M  9.2G   2% /var/log
/dev/mapper/vg0-lv--boot      974M  197M  710M  22% /boot
tmpfs                         392M   12K  392M   1% /run/user/1000

q@worker-1:~$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
tmpfs                         392M  680K  391M   1% /run
/dev/mapper/vg0-lv--root       44G  2.7G   39G   7% /
tmpfs                         2.0G     0  2.0G   0% /dev/shm
tmpfs                         5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg0-lv--var--log  9.8G  109M  9.2G   2% /var/log
/dev/mapper/vg0-lv--boot      974M  197M  710M  22% /boot
tmpfs                         392M   12K  392M   1% /run/user/1000

q@worker-2:~$ df -h
Filesystem                    Size  Used Avail Use% Mounted on
tmpfs                         392M  680K  391M   1% /run
/dev/mapper/vg0-lv--root       44G  2.7G   39G   7% /
tmpfs                         2.0G     0  2.0G   0% /dev/shm
tmpfs                         5.0M     0  5.0M   0% /run/lock
/dev/mapper/vg0-lv--var--log  9.8G  109M  9.2G   2% /var/log
/dev/mapper/vg0-lv--boot      974M  197M  710M  22% /boot
tmpfs                         392M   12K  392M   1% /run/user/1000
```

### nproc

```
q@control-plane-1:~$ nproc
2

q@worker-1:~$ nproc
2

q@worker-2:~$ nproc
2
```

## Вопросы и риски

Опишите вопросы или риски по стенду, если они есть.
