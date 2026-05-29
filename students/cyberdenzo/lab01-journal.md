# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Денис |
| GitHub | https://github.com/cyberdenzo |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | done |  |  |
| Lab 01 - Node Baseline | done |  |  |


## Lab 01 - Node Baseline

### Дата

2026-05-27

### Цель

Привести каждую VM к предсказуемому базовому состоянию.

### Что было сделано

- Проверка OS, hostname, IP и ресурсов
- Проверка DNS и internet access
- Проверка доступности package repositories
- Проверка time sync
- Проверка hostname resolution
- Ручное исправление /etc/hosts
- Проверка node-to-node L3 connectivity
- Проверка swap
- Persistent настройка kernel modules и sysctl
- Проверка чистого состояния node

### Команды
**k8s-master-01**
```
hostnamectl
Static hostname: k8s-master-01
Machine ID: af0d31e002f34ac68a6e7897806b02fb
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU

ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.71/24

ip route
default via 192.168.30.1 dev ens18 proto static
192.168.30.0/24 dev ens18 proto kernel scope link src 192.168.30.71

cat /etc/os-release
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

free -h
               total        used        free      shared  buff/cache   available
Mem:           7,8Gi       413Mi       7,4Gi       988Ki       221Mi       7,3Gi
Swap:             0B          0B          0B

df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        99G  6,9G   87G   8% /

nproc
4

getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

ping -c 3 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.42 ms
64 bytes from 192.168.30.1: icmp_seq=3 ttl=64 time=1.10 ms

--- 192.168.30.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.102/1.230/1.415/0.133 ms

ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=25.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=25.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=25.2 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 25.076/25.143/25.191/0.048 ms

ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (172.66.152.176) 56(84) bytes of data.
64 bytes from 172.66.152.176: icmp_seq=1 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=2 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=3 ttl=53 time=61.7 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 61.672/61.694/61.725/0.022 ms

getent hosts security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com

getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io

getent hosts github.com
140.82.121.4    github.com

curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200
date: Wed, 27 May 2026 19:27:32 GMT
content-type: text/html;charset=UTF-8
server: cloudflare

curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302
server: nginx

curl -I --connect-timeout 10 https://github.com/
HTTP/2 200
server: github.com

sudo apt update
Сущ:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Сущ:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Сущ:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Сущ:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Все пакеты имеют последние версии.

apt-cache policy containerd runc
containerd:
  Установлен: (отсутствует)
  Кандидат:   2.2.1-0ubuntu1~24.04.2
runc:
  Установлен: (отсутствует)
  Кандидат:   1.3.4-0ubuntu1~24.04.1

timedatectl
Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no

systemctl is-active systemd-timesyncd
active

timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Wed 2026-05-27 19:32:58 UTC

hostname
k8s-master-01

hostname -f
k8s-master-01

getent hosts "$(hostname)"
192.168.30.71   k8s-master-01

cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ubuntu
192.168.30.71   k8s-master-01
192.168.30.72   k8s-worker-01
192.168.30.73   k8s-worker-02

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

ping -c 3 192.168.30.72
PING 192.168.30.72 (192.168.30.72) 56(84) bytes of data.
64 bytes from 192.168.30.72: icmp_seq=1 ttl=64 time=0.334 ms
64 bytes from 192.168.30.72: icmp_seq=2 ttl=64 time=0.357 ms
64 bytes from 192.168.30.72: icmp_seq=3 ttl=64 time=0.276 ms

--- 192.168.30.72 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2079ms
rtt min/avg/max/mdev = 0.276/0.322/0.357/0.034 ms

ping -c 3 192.168.30.73
PING 192.168.30.73 (192.168.30.73) 56(84) bytes of data.
64 bytes from 192.168.30.73: icmp_seq=1 ttl=64 time=0.308 ms
64 bytes from 192.168.30.73: icmp_seq=2 ttl=64 time=0.433 ms
64 bytes from 192.168.30.73: icmp_seq=3 ttl=64 time=0.275 ms

--- 192.168.30.73 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2024ms
rtt min/avg/max/mdev = 0.275/0.338/0.433/0.068 ms

lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1

sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true

dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true

systemctl list-unit-files | grep -E 'containerd|kubelet' || true

```

**k8s-worker-01**
```
hostnamectl
Static hostname: k8s-worker-01
Machine ID: 0fdbc68749954b0e9e2ee45a8a4b7b2a
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU

ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.72/24

ip route
default via 192.168.30.1 dev ens18 proto static
192.168.30.0/24 dev ens18 proto kernel scope link src 192.168.30.72

cat /etc/os-release
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

free -h
               total        used        free      shared  buff/cache   available
Mem:           7,8Gi       419Mi       6,9Gi       992Ki       665Mi       7,3Gi
Swap:             0B          0B          0B


df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        99G  6,9G   87G   8% /

nproc
4

getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

ping -c 3 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.42 ms
64 bytes from 192.168.30.1: icmp_seq=3 ttl=64 time=1.10 ms

--- 192.168.30.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.102/1.230/1.415/0.133 ms

ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=25.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=25.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=25.2 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 25.076/25.143/25.191/0.048 ms

ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (172.66.152.176) 56(84) bytes of data.
64 bytes from 172.66.152.176: icmp_seq=1 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=2 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=3 ttl=53 time=61.7 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 61.672/61.694/61.725/0.022 ms

getent hosts security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com

getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io

getent hosts github.com
140.82.121.4    github.com

curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200
content-type: text/html;charset=UTF-8
server: cloudflare

curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302
server: nginx

curl -I --connect-timeout 10 https://github.com/
HTTP/2 200
server: github.com

sudo apt update
Сущ:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Сущ:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Сущ:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Сущ:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Все пакеты имеют последние версии.

apt-cache policy containerd runc
containerd:
  Установлен: (отсутствует)
  Кандидат:   2.2.1-0ubuntu1~24.04.2
runc:
  Установлен: (отсутствует)
  Кандидат:   1.3.4-0ubuntu1~24.04.1

timedatectl
Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no

systemctl is-active systemd-timesyncd
active

timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Wed 2026-05-27 19:41:50 UTC

hostname
k8s-worker-01

hostname -f
k8s-worker-01

getent hosts "$(hostname)"
192.168.30.72   k8s-worker-01

cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ubuntu
192.168.30.71   k8s-master-01
192.168.30.72   k8s-worker-01
192.168.30.73   k8s-worker-02

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

ping -c 3 192.168.30.71
PING 192.168.30.71 (192.168.30.71) 56(84) bytes of data.
64 bytes from 192.168.30.71: icmp_seq=1 ttl=64 time=0.387 ms
64 bytes from 192.168.30.71: icmp_seq=2 ttl=64 time=0.425 ms
64 bytes from 192.168.30.71: icmp_seq=3 ttl=64 time=0.296 ms

--- 192.168.30.71 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2080ms
rtt min/avg/max/mdev = 0.296/0.369/0.425/0.054 ms

ping -c 3 192.168.30.73
PING 192.168.30.73 (192.168.30.73) 56(84) bytes of data.
64 bytes from 192.168.30.73: icmp_seq=1 ttl=64 time=0.308 ms
64 bytes from 192.168.30.73: icmp_seq=2 ttl=64 time=0.433 ms
64 bytes from 192.168.30.73: icmp_seq=3 ttl=64 time=0.275 ms

--- 192.168.30.73 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2024ms
rtt min/avg/max/mdev = 0.275/0.338/0.433/0.068 ms

lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1

sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true

dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true

systemctl list-unit-files | grep -E 'containerd|kubelet' || true

```

**k8s-worker-02**
```
hostnamectl
Static hostname: k8s-worker-02
Machine ID: 76fa61c387ac4a6ab65433ccca03a109
Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.8.0-117-generic
Architecture: x86-64
Hardware Vendor: QEMU

ip -br addr
lo               UNKNOWN        127.0.0.1/8
ens18            UP             192.168.30.73/24

ip route
default via 192.168.30.1 dev ens18 proto static
192.168.30.0/24 dev ens18 proto kernel scope link src 192.168.30.73

cat /etc/os-release
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

free -h
               total        used        free      shared  buff/cache   available
Mem:           7,8Gi       408Mi       6,9Gi       992Ki       667Mi       7,4Gi
Swap:             0B          0B          0B

df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        99G  6,9G   87G   8% /

nproc
4

getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

ping -c 3 192.168.30.1
PING 192.168.30.1 (192.168.30.1) 56(84) bytes of data.
64 bytes from 192.168.30.1: icmp_seq=1 ttl=64 time=1.18 ms
64 bytes from 192.168.30.1: icmp_seq=2 ttl=64 time=1.42 ms
64 bytes from 192.168.30.1: icmp_seq=3 ttl=64 time=1.10 ms

--- 192.168.30.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.102/1.230/1.415/0.133 ms

ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=25.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=25.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=25.2 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 25.076/25.143/25.191/0.048 ms

ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (172.66.152.176) 56(84) bytes of data.
64 bytes from 172.66.152.176: icmp_seq=1 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=2 ttl=53 time=61.7 ms
64 bytes from 172.66.152.176: icmp_seq=3 ttl=53 time=61.7 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 61.672/61.694/61.725/0.022 ms

getent hosts security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com

getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io

getent hosts github.com
140.82.121.4    github.com

curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200
content-type: text/html;charset=UTF-8
server: cloudflare

curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302
server: nginx

curl -I --connect-timeout 10 https://github.com/
HTTP/2 200
server: github.com

sudo apt update
Сущ:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Сущ:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease
Сущ:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease
Сущ:4 http://security.ubuntu.com/ubuntu noble-security InRelease
Чтение списков пакетов… Готово
Построение дерева зависимостей… Готово
Чтение информации о состоянии… Готово
Все пакеты имеют последние версии.

apt-cache policy containerd runc
containerd:
  Установлен: (отсутствует)
  Кандидат:   2.2.1-0ubuntu1~24.04.2
runc:
  Установлен: (отсутствует)
  Кандидат:   1.3.4-0ubuntu1~24.04.1

timedatectl
Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
NTP service: active
RTC in local TZ: no

systemctl is-active systemd-timesyncd
active

timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Wed 2026-05-27 19:49:52 UTC

hostname
k8s-worker-02

hostname -f
k8s-worker-02

getent hosts "$(hostname)"
192.168.30.73   k8s-worker-02

cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 ubuntu
192.168.30.71   k8s-master-01
192.168.30.72   k8s-worker-01
192.168.30.73   k8s-worker-02

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

ping -c 3 192.168.30.71
PING 192.168.30.71 (192.168.30.71) 56(84) bytes of data.
64 bytes from 192.168.30.71: icmp_seq=1 ttl=64 time=0.199 ms
64 bytes from 192.168.30.71: icmp_seq=2 ttl=64 time=0.334 ms
64 bytes from 192.168.30.71: icmp_seq=3 ttl=64 time=0.461 ms

--- 192.168.30.71 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2088ms
rtt min/avg/max/mdev = 0.199/0.331/0.461/0.106 m

ping -c 3 192.168.30.72
PING 192.168.30.72 (192.168.30.72) 56(84) bytes of data.
64 bytes from 192.168.30.72: icmp_seq=1 ttl=64 time=0.469 ms
64 bytes from 192.168.30.72: icmp_seq=2 ttl=64 time=0.276 ms
64 bytes from 192.168.30.72: icmp_seq=3 ttl=64 time=0.373 ms

--- 192.168.30.72 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2037ms
rtt min/avg/max/mdev = 0.276/0.372/0.469/0.078 ms

lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1

sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true

dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true

systemctl list-unit-files | grep -E 'containerd|kubelet' || true

```

### Ключевые выводы команд

```
Серверы готовы к установке kubernetes.
Есть сетевая связность, коррекно настроен DNS.
Доступны репозитории для установки нужных компонентов.
Установлены необходимые модули ядра и sysctl.

lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1

sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1

sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true

dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true

systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
|  критичных ошибок не обнаружено |

### Что стало понятнее

- Какие модули ядра нужны для kubernetes
- Чистое состояние node перед kubeadm

### Вопросы

- На данный момент все понятно
