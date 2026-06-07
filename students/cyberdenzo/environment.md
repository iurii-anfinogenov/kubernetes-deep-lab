# Environment

## Участник

| Поле | Значение |
|---|---|
| Имя | Денис |
| GitHub | https://github.com/cyberdenzo |
| Часовой пояс | MSK |
| Сколько часов в неделю готов уделять | 6-12 |

## Платформа

| Поле | Значение |
|---|---|
| Тип стенда | selfhosted |
| Гипервизор |Proxmox |
| Где расположен стенд | work lab |
| Есть ли возможность пересоздавать VM | yes |

## Environment

|Поле|Значение|
|---|---:|
| ОС | Ubuntu 24.04 |
| Виртуализация | Proxmox |
| archive.ubuntu.com | OK |
| security.ubuntu.com | OK |
| pkgs.k8s.io | OK |
| github.com | OK |
| apt-cache policy containerd runc | OK |

## VM layout

| Node | Role | CPU | RAM | Disk | OS | Hostname | IP |
|---|---|---:|---:|---:|---|---|---|
| control-plane-1 | control-plane | 4 | 8 | 100 | Ubuntu 24.04 | k8s-master-01 | 192.168.30.71 |
| worker-1 | worker | 4 | 8 | 100 | Ubuntu 24.04 | k8s-worker-01 | 192.168.30.72 |
| worker-2 | worker | 4 | 8 | 100 | Ubuntu 24.04 | k8s-worker-02 | 192.168.30.73 |

## Network

| Check | Result |
|---|---|
| Все VM видят друг друга по сети | yes |
| Есть SSH-доступ ко всем VM | yes |
| Есть интернет с каждой VM | yes |
| IP статические или DHCP reservation | static |

## Current state

Опишите текущее состояние стенда:

- VM уже созданы или будут созданы позже? - **Уже созданы**
- Есть ли уже установленный Kubernetes?  - **Нет**
- Готовы ли снести текущий кластер и начать с чистых VM? - **Да**
- Есть ли ограничения по ресурсам? - **Нет**

## Команды проверки

### hostnamectl
```
Static hostname: k8s-master-01
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU
Hardware Model: Standard PC _i440FX + PIIX, 1996_
```

```
Static hostname: k8s-worker-01
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU
Hardware Model: Standard PC _i440FX + PIIX, 1996_
```
```
Static hostname: k8s-worker-02
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU
Hardware Model: Standard PC _i440FX + PIIX, 1996_
```


### ip -br addr
```
root@k8s-master-01:~# ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.71/24
```
```
root@k8s-worker-01:~# ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.72/24
```
```
root@k8s-worker-02:~# ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.73/24
```

### free -h

```
root@k8s-master-01:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       422Mi       7.3Gi       988Ki       253Mi       7.3Gi
Swap:          4.0Gi          0B       4.0Gi
```
```
root@k8s-worker-01:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       428Mi       7.3Gi       980Ki       219Mi       7.3Gi
Swap:          4.0Gi          0B       4.0Gi
```
```
root@k8s-worker-02:~# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       421Mi       7.4Gi       980Ki       221Mi       7.3Gi
Swap:          4.0Gi          0B       4.0Gi
```

### df -h /

```
root@k8s-master-01:~# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        99G  7.3G   87G   8% /
```
```
root@k8s-worker-01:~# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        99G  7.3G   87G   8% /
```
```
root@k8s-worker-02:~# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        99G  7.3G   87G   8% /
```

### nproc

```
root@k8s-master-01:~# nproc
8
```
```
root@k8s-worker-01:~# nproc
8
```
```
root@k8s-worker-02:~# nproc
8
```

## Вопросы и риски

---
