# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя |  |
| GitHub |  |
| Часовой пояс |  |
| Сколько часов в неделю готов уделять |  |

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
| control-plane-1 | control-plane | 2 | 4 | 30 | Ubuntu 24.04.4 LTS | Deeplab-control-plane-1 | 192.168.0.92 |
| worker-1 | worker | 2 | 4 | 40 | Ubuntu 24.04.4 LTS | Deeplab-worker-1 | 192.168.0.96 |
| worker-2 | worker | 2 | 4 | 40 | Ubuntu 24.04.4 LTS | Deeplab-worker-2 | 192.168.0.97 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | static |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже? - **Созданы**
- Есть ли уже установленный Kubernetes? - **Нет**
- Готовы ли снести текущий кластер и начать с чистых VM? - **Да**
- Есть ли ограничения по ресурсам? - **Максимум могу по 4 Cpu и 8 Mem на 3 ноды дать**

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl

```
    Static hostname: Deeplab-control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 7c3bb5eced98490392c025963ce2d7e5
         Boot ID: 6616e8d1d33148b080bb896deaeda697
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 2d                 
```
```
     Static hostname: Deeplab-worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 3770e93cd1ee4c76a53a29f23738808c
         Boot ID: 70ca858e11b94dc9b875bedebd5520ef
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 2d 
```
```
     Static hostname: Deeplab-worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: f529db0636ed482faf0ef05bc791aa4e
         Boot ID: d8ac187b7f954423ad2f9f571c425d92
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.16.3-0-ga6ed6b701f0a-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 2d
```

### ip -br addr

```
Deeplab-control-plane-1
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.0.92/24 fe80::be24:11ff:fe52:84b8/64 
```
```
Deeplab-worker-1
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.0.96/24 fe80::be24:11ff:fe0b:d6d1/64 
```
```
Deeplab-worker-2
lo               UNKNODeeplab-worker-1WN        127.0.0.1/8 ::1/128 
eth0             UP             192.168.0.97/24 fe80::be24:11ff:fea1:889f/64
```

### free -h

```
Deeplab-control-plane-1
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       397Mi       3.0Gi       1.0Mi       628Mi       3.4Gi
Swap:             0B          0B          0B
```

```
    Deeplab-worker-1
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       382Mi       3.0Gi       1.0Mi       651Mi       3.5Gi
Swap:             0B          0B          0B
```
```
    Deeplab-worker-2
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       405Mi       3.0Gi       1.0Mi       631Mi       3.4Gi
Swap:             0B          0B          0B
```


### df -h /

```
Deeplab-control-plane-1
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        29G  2.3G   26G   9% /
```

```
Deeplab-worker-1
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        38G  2.3G   36G   7% /
```

```
Deeplab-worker-2
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        38G  2.3G   36G   7% /
```

### nproc

```
 Deeplab-control-plane-1
 2
 ```
 ```
 Deeplab-worker-1
 2
 ```
 ```
 Deeplab-worker-2
 2
 ```


## Вопросы и риски

**Работаю в графике 2/2 в рабочие дни возможно могу ночью садиться, но не факт**
