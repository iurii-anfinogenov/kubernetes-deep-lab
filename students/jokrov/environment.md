# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Oleg  |
| GitHub | https://github.com/Jokrov |
| Часовой пояс | UTS + 1 |
| Сколько часов в неделю готов уделять | 10 - 20 |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | local |
| Гипервизор | KVM (nested in Hyper-V) |
| Где расположен стенд | home lab |
| Есть ли возможность пересоздавать VM | yes |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 | 4GB | 30GB | Ubuntu 24.04 LTS | control-plane-1 | 192.168.122.90 |
| worker-1 | worker | 2 | 4GB | 40GB | worker-1 | Ubuntu 24.04 LTS | 192.168.122.159 |
| worker-2 | worker | 2 | 4GB | 40GB | worker-2 | Ubuntu 24.04 LTS | 192.168.122.244 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | DHCP-reservation |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже? -Done
- Есть ли уже установленный Kubernetes? -No
- Готовы ли снести текущий кластер и начать с чистых VM? -Yes
- Есть ли ограничения по ресурсам? - 64GB_RAM & 200GB_SSD

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl

     Static hostname: control-plane-1
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: bf022a1a89c0409aa82becf99b9a51b5
             Boot ID: 0d4f1555d01b44deb33163cfc61cadd9
      Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS
              Kernel: Linux 6.8.0-117-generic
        Architecture: x86-64
     Hardware Vendor: QEMU
      Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
       Firmware Date: Tue 2014-04-01
        Firmware Age: 12y 1month 3w 2d

    Static hostname: worker-1
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: e63a596f99dd48879b02ec73f21d8da4
             Boot ID: d870fc45661f42e9b292d055331fa763
      Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS
              Kernel: Linux 6.8.0-117-generic
        Architecture: x86-64
     Hardware Vendor: QEMU
      Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
       Firmware Date: Tue 2014-04-01
        Firmware Age: 12y 1month 3w 2d

     Static hostname: worker-2
           Icon name: computer-vm
             Chassis: vm 🖴
          Machine ID: efbbc98e82384d5f82324f85ca2e045c
             Boot ID: d8688f090d2645d59849de174325ce3b
      Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS
              Kernel: Linux 6.8.0-117-generic
        Architecture: x86-64
     Hardware Vendor: QEMU
      Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
       Firmware Date: Tue 2014-04-01
        Firmware Age: 12y 1month 3w 2d

### ip -br addr

    ===== control-plane-1 =====
    lo               UNKNOWN        127.0.0.1/8 ::1/128
    enp1s0           UP             192.168.122.90/24 metric 100 fe80::5054:ff:fec8:d65b/64

    ===== worker-1 =====
    lo               UNKNOWN        127.0.0.1/8 ::1/128
    enp1s0           UP             192.168.122.159/24 metric 100 fe80::5054:ff:fe2b:95ce/64

    ===== worker-2 =====
    lo               UNKNOWN        127.0.0.1/8 ::1/128
    enp1s0           UP             192.168.122.244/24 metric 100 fe80::5054:ff:fecb:7c12/64

### free -h

    ===== control-plane-1 =====
                   total        used        free      shared  buff/cache   available
    Mem:           3.8Gi       417Mi       3.0Gi       1.0Mi       675Mi       3.4Gi
    Swap:             0B          0B          0B

    ===== worker-1 =====
                   total        used        free      shared  buff/cache   available
    Mem:           3.8Gi       416Mi       3.0Gi       1.0Mi       656Mi       3.4Gi
    Swap:             0B          0B          0B

    ===== worker-2 =====
                   total        used        free      shared  buff/cache   available
    Mem:           3.8Gi       406Mi       3.0Gi       1.1Mi       675Mi       3.4Gi

### df -h /

    ===== control-plane-1 =====
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        29G  2.0G   27G   7% /

    ===== worker-1 =====
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        38G  1.9G   36G   6% /

    ===== worker-2 =====
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda1        38G  2.0G   36G   6% /

### nproc

    ===== control-plane-1 =====
    2

    ===== worker-1 =====
    2

    ===== worker-2 =====
    2

## Вопросы и риски
