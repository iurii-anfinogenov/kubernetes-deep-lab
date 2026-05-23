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
    Machine ID: cbf35318e4a2480b93457f4f73b71c9e
    Boot ID: a702cb11060e4450ae6e4d4b5de04a11
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w
    ```
    deep-wrk-01
    ```
    Static hostname: deep-cp-01
    Icon name: computer-vm
    Chassis: vm 🖴
    Machine ID: cbf35318e4a2480b93457f4f73b71c9e
    Boot ID: 672641d8be8140a59c28d6b621a93409
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w
    ```
    deep-wrk-02
    ```
    Static hostname: deep-wrk-02
    Icon name: computer-vm
    Chassis: vm 🖴
    Machine ID: cbf35318e4a2480b93457f4f73b71c9e
    Boot ID: a417f5f70c0f41358a1dfe2b90c6fa21
    Virtualization: kvm
    Operating System: Ubuntu 24.04.4 LTS                
    Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
    Hardware Vendor: QEMU
    Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
    Firmware Version: 1.16.3-debian-1.16.3-2
    Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w
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
