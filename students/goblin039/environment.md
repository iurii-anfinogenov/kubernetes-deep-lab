# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Вячеслав  |
| GitHub | goblin039 |
| Часовой пояс |  msk-1 |
| Сколько часов в неделю готов уделять |  8-12 |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | local |
| Гипервизор | KVM  |
| Где расположен стенд | home lab |
| Есть ли возможность пересоздавать VM | yes |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 2 | 8G | 50G | Ubuntu 24.04.4 LTS | deep-cp-01 | 192.168.100.50/24 |
| worker-1 | worker | 2 | 8G | 50G | Ubuntu 24.04.4 LTS | deep-wrk-01 | 192.168.100.51/24 |
| worker-2 | worker | 2 | 8G | 50G | Ubuntu 24.04.4 LTS | deep-wrk-02 | 192.168.100.52/24 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | static |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже?
  * созданы.
- Есть ли уже установленный Kubernetes?
  * есть на других виртуалках.
- Готовы ли снести текущий кластер и начать с чистых VM?
  * не понятно зачем, одновременно они не запускаются.
- Есть ли ограничения по ресурсам?
  * 4 ядра хостовой машины
  * 32G RAM хостовой машины

## Команды проверки

Выполните на каждой VM и вставьте вывод.

### hostnamectl

    deep-cp-01
    ```
    Static hostname: deep-cp-01
    Icon name: computer-vm
    Chassis: vm 🖴
    Machine ID: 209934fd84394599bf0237beb6ca8e2c
    Boot ID: 8e64b6ab243d4217a4d8f7ed786715c6
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d 
    ```
    deep-wrk-01
    ```
    Static hostname: deep-wrk-01
    Icon name: computer-vm
    Chassis: vm 🖴
    Machine ID: b83ef4e8468d4841bdcad82c2079917e
    Boot ID: 3c91a89dd44746f783135f952f92e386
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d
    ```
    deep-wrk-02
    ```
    Static hostname: deep-wrk-02
    Icon name: computer-vm
    Chassis: vm 🖴
    Machine ID: c502c7bd481c4fddb64c21c4a1461dcc
    Boot ID: a4e16e6178804a0e9ec8b91227f26056
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 1d
    ```        

### ip -br addr

    deep-cp-01
    ```
    lo               UNKNOWN        127.0.0.1/8 ::1/128 
    enp1s0           UP             192.168.100.50/24
    ```
    deep-wrk-01
    ```
    lo               UNKNOWN        127.0.0.1/8 ::1/128 
    enp1s0           UP             192.168.100.51/24
    ```
    deep-wrk-02
    ```
    lo               UNKNOWN        127.0.0.1/8 ::1/128 
    enp1s0           UP             192.168.100.52/24
    ```

### free -h

    deep-cp-01
    ```
                   total        used        free      shared  buff/cache   available
    Mem:           7.7Gi       422Mi       7.2Gi       4.7Mi       337Mi       7.2Gi
    Swap:          4.0Gi          0B       4.0Gi
    ```
    deep-wrk-01
    ```
                   total        used        free      shared  buff/cache   available
    Mem:           7.7Gi       385Mi       7.3Gi       4.7Mi       177Mi       7.3Gi
    Swap:          4.0Gi          0B       4.0Gi
    ```
    deep-wrk-02
    ```
                   total        used        free      shared  buff/cache   available
    Mem:           7.7Gi       369Mi       7.4Gi       4.7Mi       177Mi       7.3Gi
    Swap:          4.0Gi          0B       4.0Gi
    ```

### df -h /

    deep-cp-01
    ```
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda2        49G  6.5G   40G  14% /
    ```
    deep-wrk-01
    ```
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda2        49G  6.5G   40G  14% /
    ```
    deep-wrk-02
    ```
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/vda2        49G  6.5G   40G  14% /
    ```

### nproc

    deep-cp-01
    ```
    2
    ```
    deep-wrk-01
    ```
    2
    ```
    deep-wrk-02
    ```
    2
    ```

## Вопросы и риски

Опишите вопросы или риски по стенду, если они есть.
