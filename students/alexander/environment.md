# Environment

## Участник

| Поле                                 | Значение  |
| ------------------------------------ | --------- |
| Имя                                  | Александр |
| GitHub                               | Folau1    |
| Часовой пояс                         | UTC+5     |
| Сколько часов в неделю готов уделять | 10-15     |

## Платформа

| Поле                                 | Значение   |
| ------------------------------------ | ---------- |
| Тип стенда                           | local      |
| Гипервизор                           | VirtualBox |
| Где расположен стенд                 | home lab   |
| Есть ли возможность пересоздавать VM | yes        |

## VM layout

| Node            | Role          | CPU |  RAM |  Disk | OS                        | Hostname        | IP           |
| --------------- | ------------- | --: | ---: | ----: | ------------------------- | --------------- | ------------ |
| control-plane-1 | control-plane |   2 | 4 GB | 40 GB | Ubuntu Server 24.04.4 LTS | control-plane-1 | 192.168.1.15 |
| worker-1        | worker        |   2 | 4 GB | 30 GB | Ubuntu Server 24.04.4 LTS | worker-1        | 192.168.1.16 |
| worker-2        | worker        |   2 | 4 GB | 30 GB | Ubuntu Server 24.04.4 LTS | worker-2        | 192.168.1.17 |

## Network

| Check                               | Result |
| ----------------------------------- | ------ |
| Все VM видят друг друга по сети     | yes    |
| Есть SSH-доступ ко всем VM          | yes    |
| Есть интернет с каждой VM           | yes    |
| IP статические или DHCP reservation | static |

## Current state

* VM уже созданы.
* Kubernetes, kubeadm, Helm, CNI plugins и Argo CD не установлены.
* Используется Ubuntu Server 24.04.4 LTS.
* Стенд готов для дальнейших Kubernetes labs.
* При необходимости VM можно пересоздать с нуля.
* Критичных ограничений по ресурсам нет.

## Команды проверки

### control-plane-1

#### hostnamectl

```text id="34c6qk"
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 9ad1bce9da55425cbfa3bac436d603b8
         Boot ID: c14e2dd1c41440f4be0cf7ddda9540c1
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 5month 3w 2d
```

#### ip -br addr

```text id="f7u9xm"
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.15/24 fdc1:30bd:dce:0:a00:27ff:fe67:6b11/64 fe80::a00:27ff:fe67:6b11/64
```

#### free -h

```text id="j2v4np"
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       416Mi       3.4Gi       1.0Mi       211Mi       3.4Gi
Swap:             0B          0B          0B
```

#### df -h /

```text id="r8q3wk"
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  2.8G   35G   8% /
```

#### nproc

```text id="t6m1pc"
2
```

---

### worker-1

#### hostnamectl

```text id="z4k8rv"
 Static hostname: Worker1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 073f7868b1964915ae98ba90571f0bf4
         Boot ID: 7add506f1e06485b9f4567d7b924ab20
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 5month 3w 2d
```

#### ip -br addr

```text id="q1w7tm"
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.16/24 fdc1:30bd:dce:0:a00:27ff:fedc:9b5/64 fe80::a00:27ff:fedc:9b5/64
```

#### free -h

```text id="p5n2dx"
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       467Mi       2.9Gi       1.1Mi       676Mi       3.4Gi
Swap:             0B          0B          0B
```

#### df -h /

```text id="m9r6qv"
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  2.8G   26G  10% /
```

#### nproc

```text id="d3v8nk"
2
```

---

### worker-2

#### hostnamectl

```text id="u8m4xr"
 Static hostname: Worker2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 37dbcca896764718866c8f1e799af019
         Boot ID: fd9ef1c51f614ef9ad7afdd3000f3117
  Virtualization: oracle
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 19y 5month 3w 2d
```

#### ip -br addr

```text id="x2k7qc"
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             192.168.1.17/24 fdc1:30bd:dce:0:a00:27ff:fe57:3beb/64 fe80::a00:27ff:fe57:3beb/64
```

#### free -h

```text id="c4p9md"
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       468Mi       2.9Gi       1.1Mi       670Mi       3.4Gi
Swap:             0B          0B          0B
```

#### df -h /

```text id="n7v1tz"
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  2.8G   26G  10% /
```

#### nproc

```text id="k5m2rw"
2
```

## Вопросы и риски

На текущий момент критичных рисков нет.
