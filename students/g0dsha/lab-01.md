# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

## Участник

| Поле | Значение |
|---|---|
| Имя | Игорь |
| GitHub | https://github.com/g0dsha |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | done |  |  |

## Lab 01 - Node baseline

### Дата

2026-05-30

### Цель

Привести каждую VM к предсказуемому базовому состоянию, подготовить к установке Kubernetes

### Что было сделано

- Отключил IPv6
- Выполнил подготовительные настройки
- Выполнил проверки

### Команды

#### Проверка OS, hostname, IP и ресурсов

```
user@control-plane-1:~$ hostnamectl
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 2fc87f2cd6904906861e9cc1186a522a
         Boot ID: dc220f2b8b2e424d8b80fc70bf81a0e7
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-124-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 4w

user@control-plane-1:~$ ip -br a
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             192.168.88.184/24 metric 100

user@control-plane-1:~$ ip r
default via 192.168.88.1 dev eth0 proto dhcp src 192.168.88.184 metric 100 
192.168.88.0/24 dev eth0 proto kernel scope link src 192.168.88.184 metric 100 
192.168.88.1 dev eth0 proto dhcp scope link src 192.168.88.184 metric 100

user@control-plane-1:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

user@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       371Mi       3.4Gi       1.0Mi       283Mi       3.5Gi
Swap:             0B          0B          0B

user@control-plane-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  2.4G   28G   8% /

user@control-plane-1:~$ nproc
2
```
```
user@worker-1:~$ hostnamectl
 Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: c2d669e40e7441a3a8b374273ea0b7a6
         Boot ID: 951f4835a4ae4456a618f6eb182362bb
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-124-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 4w                               

user@worker-1:~$ ip -br a
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             192.168.88.134/24 metric 100 

user@worker-1:~$ ip r
default via 192.168.88.1 dev eth0 proto dhcp src 192.168.88.134 metric 100 
192.168.88.0/24 dev eth0 proto kernel scope link src 192.168.88.134 metric 100 
192.168.88.1 dev eth0 proto dhcp scope link src 192.168.88.134 metric 100 

user@worker-1:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

user@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       362Mi       3.4Gi       1.0Mi       283Mi       3.5Gi
Swap:             0B          0B          0B

user@worker-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        38G  2.4G   36G   7% /

user@worker-1:~$ nproc
2
```
```
user@worker-2:~$ hostnamectl
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: cbf35d397c8c40fdb8c261be03f89289
         Boot ID: 7b48a2181d344b0ca16034be8b817c59
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-124-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 4w                               

user@worker-2:~$ ip -br a
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             192.168.88.243/24 metric 100 

user@worker-2:~$ ip r
default via 192.168.88.1 dev eth0 proto dhcp src 192.168.88.243 metric 100 
192.168.88.0/24 dev eth0 proto kernel scope link src 192.168.88.243 metric 100 
192.168.88.1 dev eth0 proto dhcp scope link src 192.168.88.243 metric 100 

user@worker-2:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

user@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       384Mi       3.4Gi       1.0Mi       289Mi       3.4Gi
Swap:             0B          0B          0B

user@worker-2:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        38G  2.4G   36G   7% /

user@worker-2:~$ nproc
2
```
#### Проверка DNS и internet access
```
user@control-plane-1:~$ getent hosts archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

user@control-plane-1:~$ ping -c 3 192.168.88.1
PING 192.168.88.1 (192.168.88.1) 56(84) bytes of data.
64 bytes from 192.168.88.1: icmp_seq=1 ttl=64 time=0.175 ms
64 bytes from 192.168.88.1: icmp_seq=2 ttl=64 time=0.343 ms
64 bytes from 192.168.88.1: icmp_seq=3 ttl=64 time=0.213 ms

--- 192.168.88.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2065ms
rtt min/avg/max/mdev = 0.175/0.243/0.343/0.071 ms

user@control-plane-1:~$ ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=58 time=24.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=58 time=28.8 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=58 time=24.6 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 24.183/25.846/28.751/2.060 ms

user@control-plane-1:~$ ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (172.66.152.176) 56(84) bytes of data.
64 bytes from 172.66.152.176: icmp_seq=1 ttl=57 time=52.3 ms
64 bytes from 172.66.152.176: icmp_seq=2 ttl=57 time=50.3 ms
64 bytes from 172.66.152.176: icmp_seq=3 ttl=57 time=48.8 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 48.792/50.453/52.274/1.426 ms
```
#### Проверка доступности package repositories
```
user@control-plane-1:~$ getent hosts archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

user@control-plane-1:~$ getent hosts security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com

user@control-plane-1:~$ getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io

user@control-plane-1:~$ getent hosts github.com
140.82.121.4    github.com

user@control-plane-1:~$ curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200 
...

user@control-plane-1:~$ curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302 
...

user@control-plane-1:~$ curl -I --connect-timeout 10 https://github.com/
HTTP/2 200
...

user@control-plane-1:~$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:3 http://archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:4 http://archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:5 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [42.4 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]
Get:7 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [178 kB]
Get:8 http://archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]
Get:9 http://archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:10 http://archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [5736 B]
Get:11 http://archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]
Fetched 1076 kB in 1s (1183 kB/s)                                          
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

user@control-plane-1:~$ apt-cache policy containerd runc
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://archive.ubuntu.com/ubuntu noble/main amd64 Packages
```
#### Проверка time sync
```
user@control-plane-1:~$ timedatectl
               Local time: Sat 2026-05-30 11:27:25 MSK
           Universal time: Sat 2026-05-30 08:27:25 UTC
                 RTC time: Sat 2026-05-30 08:27:25
                Time zone: Europe/Moscow (MSK, +0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

user@control-plane-1:~$ systemctl is-active systemd-timesyncd
active

user@control-plane-1:~$ timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Europe/Moscow
NTPSynchronized=yes
TimeUSec=Sat 2026-05-30 11:28:48 MSK
```
#### Проверка hostname resolution
```
user@control-plane-1:~$ hostname
control-plane-1

user@control-plane-1:~$ hostname -f
control-plane-1.int

user@control-plane-1:~$ getent hosts "$(hostname)"
192.168.88.184  control-plane-1.int control-plane-1

user@control-plane-1:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.88.184 control-plane-1.int control-plane-1
192.168.88.134 worker-1.int worker-1
192.168.88.243 worker-2.int worker-2

user@worker-1:~$ hostname
worker-1

user@worker-1:~$ hostname -f
worker-1.int

user@worker-1:~$ getent hosts $(hostname)
192.168.88.134  worker-1.int worker-1

user@worker-1:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.88.134 worker-1.int worker-1
192.168.88.184 control-plane-1.int control-plane-1
192.168.88.243 worker-2.int worker-2

user@worker-2:~$ hostname
worker-2

user@worker-2:~$ hostname -f
worker-2.int

user@worker-2:~$ getent hosts $(hostname)
192.168.88.243  worker-2.int worker-2

user@worker-2:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.88.243 worker-2.int worker-2
192.168.88.184 control-plane-1.int control-plane-1
192.168.88.134 worker-1.int worker-1
```
#### Проверка node-to-node L3 connectivity
```
user@control-plane-1:~$ ping -c3 192.168.88.134
PING 192.168.88.134 (192.168.88.134) 56(84) bytes of data.
64 bytes from 192.168.88.134: icmp_seq=1 ttl=64 time=0.528 ms
64 bytes from 192.168.88.134: icmp_seq=2 ttl=64 time=0.238 ms
64 bytes from 192.168.88.134: icmp_seq=3 ttl=64 time=0.270 ms

--- 192.168.88.134 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2066ms
rtt min/avg/max/mdev = 0.238/0.345/0.528/0.129 ms

user@control-plane-1:~$ ping -c3 192.168.88.243
PING 192.168.88.243 (192.168.88.243) 56(84) bytes of data.
64 bytes from 192.168.88.243: icmp_seq=1 ttl=64 time=0.443 ms
64 bytes from 192.168.88.243: icmp_seq=2 ttl=64 time=0.270 ms
64 bytes from 192.168.88.243: icmp_seq=3 ttl=64 time=0.372 ms

--- 192.168.88.243 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2082ms
rtt min/avg/max/mdev = 0.270/0.361/0.443/0.071 ms

user@worker-1:~$ ping -c3 192.168.88.184
PING 192.168.88.184 (192.168.88.184) 56(84) bytes of data.
64 bytes from 192.168.88.184: icmp_seq=1 ttl=64 time=0.229 ms
64 bytes from 192.168.88.184: icmp_seq=2 ttl=64 time=0.216 ms
64 bytes from 192.168.88.184: icmp_seq=3 ttl=64 time=0.251 ms

--- 192.168.88.184 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2059ms
rtt min/avg/max/mdev = 0.216/0.232/0.251/0.014 ms

user@worker-1:~$ ping -c3 192.168.88.243
PING 192.168.88.243 (192.168.88.243) 56(84) bytes of data.
64 bytes from 192.168.88.243: icmp_seq=1 ttl=64 time=0.486 ms
64 bytes from 192.168.88.243: icmp_seq=2 ttl=64 time=0.250 ms
64 bytes from 192.168.88.243: icmp_seq=3 ttl=64 time=0.230 ms

--- 192.168.88.243 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2035ms
rtt min/avg/max/mdev = 0.230/0.322/0.486/0.116 ms

user@worker-2:~$ ping -c3 192.168.88.184
PING 192.168.88.184 (192.168.88.184) 56(84) bytes of data.
64 bytes from 192.168.88.184: icmp_seq=1 ttl=64 time=0.318 ms
64 bytes from 192.168.88.184: icmp_seq=2 ttl=64 time=0.251 ms
64 bytes from 192.168.88.184: icmp_seq=3 ttl=64 time=0.302 ms

--- 192.168.88.184 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.251/0.290/0.318/0.028 ms

user@worker-2:~$ ping -c3 192.168.88.134
PING 192.168.88.134 (192.168.88.134) 56(84) bytes of data.
64 bytes from 192.168.88.134: icmp_seq=1 ttl=64 time=0.281 ms
64 bytes from 192.168.88.134: icmp_seq=2 ttl=64 time=0.247 ms
64 bytes from 192.168.88.134: icmp_seq=3 ttl=64 time=0.299 ms

--- 192.168.88.134 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2037ms
rtt min/avg/max/mdev = 0.247/0.275/0.299/0.021 ms
```
#### Проверка swap
```
user@control-plane-1:~$ swapon --show

user@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       347Mi       3.0Gi       1.0Mi       720Mi       3.5Gi
Swap:             0B          0B          0B

user@worker-1:~$ swapon --show

user@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       376Mi       3.3Gi       1.0Mi       428Mi       3.5Gi
Swap:             0B          0B          0B

user@worker-2:~$ swapon --show

user@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       370Mi       3.2Gi       1.0Mi       453Mi       3.5Gi
Swap:             0B          0B          0B
```
#### Kernel modules и sysctl для Kubernetes networking
IPv6 отключен, поэтому net.bridge.bridge-nf-call-ip6tables = 1 не устанавливал
```
user@control-plane-1:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

user@control-plane-1:~$ sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1

user@control-plane-1:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
```
user@worker-1:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

user@worker-1:~$ sysctl net.bridge.bridge-nf-call-iptables 
net.bridge.bridge-nf-call-iptables = 1

user@worker-1:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
```
user@worker-2:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

user@worker-2:~$ sysctl net.bridge.bridge-nf-call-iptables 
net.bridge.bridge-nf-call-iptables = 1

user@worker-2:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
#### Проверка чистого состояния node
```
user@control-plane-1:~$ command -v containerd || true
user@control-plane-1:~$ command -v runc || true
user@control-plane-1:~$ command -v crictl || true
user@control-plane-1:~$ command -v kubeadm || true
user@control-plane-1:~$ command -v kubelet || true
user@control-plane-1:~$ command -v kubectl || true
user@control-plane-1:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
user@control-plane-1:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true

user@worker-1:~$ command -v containerd || true
user@worker-1:~$ command -v runc || true
user@worker-1:~$ command -v crictl || true
user@worker-1:~$ command -v kubeadm || true
user@worker-1:~$ command -v kubelet || true
user@worker-1:~$ command -v kubectl || true
user@worker-1:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
user@worker-1:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true

user@worker-2:~$ command -v containerd || true
user@worker-2:~$ command -v runc || true
user@worker-2:~$ command -v crictl || true
user@worker-2:~$ command -v kubeadm || true
user@worker-2:~$ command -v kubelet || true
user@worker-2:~$ command -v kubectl || true
user@worker-2:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
user@worker-2:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```
### Ключевые выводы команд

- OS: Ubuntu 24.04.4 LTS
- CPU: 2 vCPU
- RAM: 4 GB
- Swap: 0B
- Root disk: достаточно свободного места
- IP: стабильный internal IP
- Default route: присутствует
- gateway доступен
- DNS работает
- Ubuntu mirror доступен
- packet loss: 0%
- DNS-запросы возвращают IP-адреса
- curl возвращает HTTP status 200, 301 или 302
- apt update завершается без ошибок
- apt-cache policy показывает доступные пакеты containerd и runc
- System clock synchronized: yes
- NTP service: active
- NTPSynchronized=yes
- hostname возвращает имя текущей node
- hostname -f не уходит во внешний случайный домен
- getent hosts "$(hostname)" возвращает internal IP текущей node
- swapon --show не выводит активных swap devices/files
- free -h показывает Swap: 0B
- br_netfilter и overlay загружены
- sysctl параметры заданы
- containerd еще не установлен
- kubeadm еще не установлен
- kubelet еще не установлен
- kubectl еще не установлен

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  |  |  |  |

### Что стало понятнее

- Разбирался почему при отключеном ipv6.disable=1 в ядре getent все равно выводит IPv6 адреса, оказалось что на DNS отключение не влияет, но getent ahost показывает уже IPv4 адреса которые фактически будут использоваться.

### Вопросы

- IPv6 отключил и модули с настройки с ним связанные не прописывал, хотя может он в дальнейшем понадобится?
- Почему проверку time sync достаточно выполнить только на control-plane node? Точное время для kubelet, для логов на всех нодах важно.
- Домен, что внутренний DNS навешивает оставил, он не случайный получается, это ок?
- В /etc/hosts все ноды прописывал, считаю так привильнее.


### Статус

done
