# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Игорь |
| GitHub | https://github.com/g0dsha |
| Часовой пояс | MSK |
| Сколько часов в неделю готов уделять | 7-10 |

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
| control-plane-1 | control-plane | 2 | 4 | 30 | Ubuntu 24.04.3 LTS | control-plane-1 | 192.168.88.184 |
| worker-1 | worker | 2 | 4 | 40 | Ubuntu 24.04.3 LTS | worker-1 | 192.168.88.134 |
| worker-2 | worker | 2 | 4 | 40 | Ubuntu 24.04.3 LTS | worker-2 | 192.168.88.243 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | dhcp-reservation |

## Current state

- VM уже созданы или будут созданы позже? - Созданы
- Есть ли уже установленный Kubernetes? - Нет
- Готовы ли снести текущий кластер и начать с чистых VM? - Да
- Есть ли ограничения по ресурсам? - Нет

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl
```
     Static hostname: control-plane-1
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: 2fc87f2cd6904906861e9cc1186a522a
             Boot ID: 684a235df34a45bfbf697cf5b385ec45
      Virtualization: kvm
    Operating System: Ubuntu 24.04.3 LTS                          
              Kernel: Linux 6.8.0-90-generic
        Architecture: x86-64
     Hardware Vendor: QEMU
      Hardware Model: Standard PC _i440FX + PIIX, 1996_
    Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
       Firmware Date: Tue 2014-04-01
        Firmware Age: 12y 1month 3w 1d
```
```
     Static hostname: worker-1
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: c2d669e40e7441a3a8b374273ea0b7a6
             Boot ID: 59e3028ec5584c81a1e9e891d1814d3c
      Virtualization: kvm
    Operating System: Ubuntu 24.04.3 LTS                          
              Kernel: Linux 6.8.0-90-generic
        Architecture: x86-64
     Hardware Vendor: QEMU
      Hardware Model: Standard PC _i440FX + PIIX, 1996_
    Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
       Firmware Date: Tue 2014-04-01
        Firmware Age: 12y 1month 3w 1d
```
```
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: cbf35d397c8c40fdb8c261be03f89289
         Boot ID: b0f9d8a2136f46ca81f8bf9fdc00b085
  Virtualization: kvm
Operating System: Ubuntu 24.04.3 LTS                          
          Kernel: Linux 6.8.0-90-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d
```

### ip -br addr
```
user@control-plane-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.88.184/24 metric 100 fe80::be24:11ff:fe12:fa7f/64
```
```
user@worker-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.88.134/24 metric 100 fe80::be24:11ff:fe6c:af48/64 
```
```
user@worker-2:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.88.243/24 metric 100 fe80::be24:11ff:fea2:bd38/64
```
### free -h

```
user@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       361Mi       3.4Gi       1.0Mi       304Mi       3.5Gi
Swap:             0B          0B          0B
```
```
user@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       376Mi       3.4Gi       1.0Mi       303Mi       3.5Gi
Swap:             0B          0B          0B
```
```
user@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       348Mi       2.9Gi       1.0Mi       794Mi       3.5Gi
Swap:             0B          0B          0B
```
### df -h /

```
user@control-plane-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  2.0G   29G   7% /
```
```
user@worker-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        38G  2.0G   36G   6% /
```
```
user@worker-2:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        38G  2.1G   36G   6% /
```
### nproc
```
user@control-plane-1:~$ nproc
2
```
```
user@worker-1:~$ nproc
2
```
```
user@worker-2:~$ nproc
2
```
## Вопросы и риски