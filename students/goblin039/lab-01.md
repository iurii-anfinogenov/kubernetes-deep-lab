# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | Вячеслав  |
| GitHub | goblin039 |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 00 - Environment Validation | finish |  |  |

## Lab 01 - Node Baseline

### Дата

2026-05-26

### Цель

Подготавливка Linux nodes к установке Kubernetes.

### Что было сделано

- Проверка OS, hostname, IP и ресурсов
- Проверка DNS и internet access
- Проверка доступности package repositories
- Проверка time sync
- Проверка hostname resolution
- Проверка node-to-node L3 connectivity
- Проверка swap
- Kernel modules и sysctl для Kubernetes networking
- 

### Команды

Проверка OS, hostname, IP и ресурсов  
**deep-cp-01**:  
```
root@deep-cp-01:/etc# hostnamectl
 Static hostname: deep-cp-01
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: 209934fd84394599bf0237beb6ca8e2c
         Boot ID: e29fef073f324a1f889d88e0d8f6c2d5
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
Firmware Version: 1.16.3-debian-1.16.3-2
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 3d

root@deep-cp-01:/etc# ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             192.168.100.50/24 

root@deep-cp-01:/etc# ip route
default via 192.168.100.1 dev enp1s0 proto static 
192.168.100.0/24 dev enp1s0 proto kernel scope link src 192.168.100.50

root@deep-cp-01:/etc# cat /etc/os-release
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

root@deep-cp-01:/etc# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       402Mi       7.3Gi       4.7Mi       186Mi       7.3Gi
Swap:             0B          0B          0B

root@deep-cp-01:/etc# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        49G  6.5G   40G  14% /

root@deep-cp-01:/etc# nproc
2
```

**deep-wrk-01**:  
```
root@deep-wrk-01:/etc# hostnamectl
 Static hostname: deep-wrk-01
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: b83ef4e8468d4841bdcad82c2079917e
         Boot ID: c7f37b3b6eaa4579a7a91c799a863c85
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
Firmware Version: 1.16.3-debian-1.16.3-2
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 3d

root@deep-wrk-01:/etc# ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             192.168.100.51/24

root@deep-wrk-01:/etc# ip route
default via 192.168.100.1 dev enp1s0 proto static 
192.168.100.0/24 dev enp1s0 proto kernel scope link src 192.168.100.51

root@deep-wrk-01:/etc# cat /etc/os-release
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

root@deep-wrk-01:/etc# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       403Mi       7.2Gi       4.7Mi       263Mi       7.3Gi
Swap:             0B          0B          0B

root@deep-wrk-01:/etc# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        49G  6.5G   40G  14% /

root@deep-wrk-01:/etc# nproc
2
```

**deep-wrk-02**:  
```
root@deep-wrk-02:/etc# hostnamectl
 Static hostname: deep-wrk-02
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: c502c7bd481c4fddb64c21c4a1461dcc
         Boot ID: d89980c8b2a34a94823f8c981b08e3db
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Ubuntu 24.04 PC _Q35 + ICH9, 2009_
Firmware Version: 1.16.3-debian-1.16.3-2
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 3d

root@deep-wrk-02:/etc# ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp1s0           UP             192.168.100.52/24

root@deep-wrk-02:/etc# ip route
default via 192.168.100.1 dev enp1s0 proto static 
192.168.100.0/24 dev enp1s0 proto kernel scope link src 192.168.100.52

root@deep-wrk-02:/etc# cat /etc/os-release
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

root@deep-wrk-02:/etc# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       394Mi       7.3Gi       4.7Mi       263Mi       7.3Gi
Swap:             0B          0B          0B

root@deep-wrk-02:/etc# df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        49G  6.5G   40G  14% /

root@deep-wrk-02:/etc# nproc
2
```
-----

Проверка DNS и internet access  
**deep-cp-01**:  
```
root@deep-cp-01:/etc# getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com

root@deep-cp-01:/etc# ping -c 3 192.168.100.1
PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
64 bytes from 192.168.100.1: icmp_seq=1 ttl=64 time=0.126 ms
64 bytes from 192.168.100.1: icmp_seq=2 ttl=64 time=0.391 ms
64 bytes from 192.168.100.1: icmp_seq=3 ttl=64 time=0.377 ms

--- 192.168.100.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2067ms
rtt min/avg/max/mdev = 0.126/0.298/0.391/0.121 ms

root@deep-cp-01:/etc# ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=56 time=14.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=56 time=14.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=56 time=14.1 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 14.101/14.153/14.231/0.055 ms

root@deep-cp-01:/etc# ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (104.20.28.246) 56(84) bytes of data.
64 bytes from 104.20.28.246: icmp_seq=1 ttl=56 time=14.5 ms
64 bytes from 104.20.28.246: icmp_seq=2 ttl=56 time=14.3 ms
64 bytes from 104.20.28.246: icmp_seq=3 ttl=56 time=14.5 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 5098ms
rtt min/avg/max/mdev = 14.287/14.424/14.505/0.097 ms
```

-----

Проверка доступности package repositories  
**deep-cp-01**:  
```
root@deep-cp-01:/etc# getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
root@deep-cp-01:/etc# getent hosts security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
root@deep-cp-01:/etc# getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io
root@deep-cp-01:/etc# getent hosts github.com
140.82.121.4    github.com

root@deep-cp-01:/etc# curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200 
date: Mon, 25 May 2026 21:28:11 GMT
content-type: text/html;charset=UTF-8
server: cloudflare
set-cookie: __cf_bm=No9TIDFk_hwQ7ZOj1sEBvS8cwpgS3W3utsmkSh6YxN0-1779744491.02395-1.0.1.1-XlVSziCJosQ_B5PWGPl6UPeL0Bjl.Yg7_SLYfjw7X2.Y5x.RJFFJ98ffs2GszwsXMkZUOD4SPnraCbhh6biR3cf5b4nqzqCX5uk586q0nMaOS3J6ikZkxyKja_jCvOAA; HttpOnly; SameSite=None; Secure; Path=/; Domain=ubuntu.com; Expires=Mon, 25 May 2026 21:58:11 GMT
cf-cache-status: DYNAMIC
cf-ray: a017a0dce93bd90b-WAW

root@deep-cp-01:/etc# curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302 
server: nginx
date: Mon, 25 May 2026 21:28:21 GMT
content-type: text/html
content-length: 138
location: https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
via: 1.1 google
alt-svc: h3=":443"; ma=2592000

root@deep-cp-01:/etc# curl -I --connect-timeout 10 https://github.com/
HTTP/2 200 
date: Mon, 25 May 2026 21:28:26 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Accept-Language, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
content-language: en-US
etag: W/"b3d411652cc33e3ee87f7b38be7c70eb"
cache-control: max-age=0, private, must-revalidate
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy: default-src 'none'; base-uri 'self'; child-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/; connect-src 'self' uploads.github.com www.githubstatus.com collector.github.com raw.githubusercontent.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com *.rel.tunnels.api.visualstudio.com wss://*.rel.tunnels.api.visualstudio.com github.githubassets.com objects-origin.githubusercontent.com copilot-proxy.githubusercontent.com proxy.individual.githubcopilot.com proxy.business.githubcopilot.com proxy.enterprise.githubcopilot.com *.actions.githubusercontent.com wss://*.actions.githubusercontent.com productionresultssa0.blob.core.windows.net productionresultssa1.blob.core.windows.net productionresultssa2.blob.core.windows.net productionresultssa3.blob.core.windows.net productionresultssa4.blob.core.windows.net productionresultssa5.blob.core.windows.net productionresultssa6.blob.core.windows.net productionresultssa7.blob.core.windows.net productionresultssa8.blob.core.windows.net productionresultssa9.blob.core.windows.net productionresultssa10.blob.core.windows.net productionresultssa11.blob.core.windows.net productionresultssa12.blob.core.windows.net productionresultssa13.blob.core.windows.net productionresultssa14.blob.core.windows.net productionresultssa15.blob.core.windows.net productionresultssa16.blob.core.windows.net productionresultssa17.blob.core.windows.net productionresultssa18.blob.core.windows.net productionresultssa19.blob.core.windows.net github-production-repository-image-32fea6.s3.amazonaws.com github-production-release-asset-2e65be.s3.amazonaws.com insights.github.com wss://alive.github.com wss://alive-staging.github.com api.githubcopilot.com api.individual.githubcopilot.com api.business.githubcopilot.com api.enterprise.githubcopilot.com edge.fullstory.com rs.fullstory.com; font-src github.githubassets.com; form-action 'self' github.com gist.github.com copilot-workspace.githubnext.com objects-origin.githubusercontent.com; frame-ancestors 'none'; frame-src viewscreen.githubusercontent.com notebooks.githubusercontent.com www.youtube-nocookie.com; img-src 'self' data: blob: github.githubassets.com media.githubusercontent.com camo.githubusercontent.com identicons.github.com avatars.githubusercontent.com private-avatars.githubusercontent.com github-cloud.s3.amazonaws.com objects.githubusercontent.com release-assets.githubusercontent.com secured-user-images.githubusercontent.com user-images.githubusercontent.com private-user-images.githubusercontent.com opengraph.githubassets.com marketplace-screenshots.githubusercontent.com copilotprodattachments.blob.core.windows.net/github-production-copilot-attachments/ github-production-user-asset-6210df.s3.amazonaws.com customer-stories-feed.github.com spotlights-feed.github.com explore-feed.github.com objects-origin.githubusercontent.com *.githubusercontent.com images.ctfassets.net/8aevphvgewt8/; manifest-src 'self'; media-src github.com user-images.githubusercontent.com secured-user-images.githubusercontent.com private-user-images.githubusercontent.com github-production-user-asset-6210df.s3.amazonaws.com gist.github.com github.githubassets.com assets.ctfassets.net/8aevphvgewt8/ videos.ctfassets.net/8aevphvgewt8/; script-src github.githubassets.com; style-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
accept-ranges: bytes
set-cookie: _gh_sess=oSyHkLHGba581s6o1LfWp7lfOtGyDv42iSnqNK0NOidjLm%2BXQKgHpwoqyKeGDGkL0Q1OUAF6JMnfAEeupTQpSmDTiWCffeMjIBmCPInCQCMyJCX4X%2F0r1mLkmEIRBv4BJfZl%2Be0oqeqA%2FoDaBFyWXOORUdawB3KK1rMBVuy8iGz364WqJ%2FTUJhSc%2FriJEl9nNsme4IDMKCOQ8TB7AuMWPHlMjYYAhWAWEOiv5vCc1QhRQmuNfYTDOGmhJ8Ye2so45SiJUtQpzpZkQL9uAh495g%3D%3D--Gde1prxh6PldOXYG--YP%2FXz4tbqydsCsNccKHlRA%3D%3D; path=/; HttpOnly; secure; SameSite=Lax
set-cookie: _octo=GH1.1.1028532922.1779744509; expires=Tue, 25 May 2027 21:28:29 GMT; domain=.github.com; path=/; secure; SameSite=Lax
set-cookie: logged_in=no; expires=Tue, 25 May 2027 21:28:29 GMT; domain=.github.com; path=/; HttpOnly; secure; SameSite=Lax
x-github-request-id: 812A:23910C:1A799CB:135EF37:6A14BEFD

root@deep-cp-01:/etc# apt-cache policy containerd runc
containerd:
  Installed: (none)
  Candidate: 2.2.1-0ubuntu1~24.04.2
  Version table:
     2.2.1-0ubuntu1~24.04.2 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.7.28-0ubuntu1~24.04.2 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.7.12-0ubuntu4 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
runc:
  Installed: (none)
  Candidate: 1.3.4-0ubuntu1~24.04.1
  Version table:
     1.3.4-0ubuntu1~24.04.1 500
        500 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages
     1.3.3-0ubuntu1~24.04.3 500
        500 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages
     1.1.12-0ubuntu3 500
        500 http://ru.archive.ubuntu.com/ubuntu noble/main amd64 Packages
```

-----

Проверка time sync  
**deep-cp-01**:  
```
root@deep-cp-01:/etc# timedatectl
               Local time: Mon 2026-05-25 20:50:02 UTC
           Universal time: Mon 2026-05-25 20:50:02 UTC
                 RTC time: Mon 2026-05-25 20:50:02
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
root@deep-cp-01:/etc# systemctl is-active systemd-timesyncd
active
root@deep-cp-01:/etc# timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Mon 2026-05-25 20:50:16 UTC
```

-----

Проверка hostname resolution  
**deep-cp-01**:  
```
root@deep-cp-01:/etc/systemd/network# hostname
deep-cp-01

root@deep-cp-01:/etc/systemd/network# hostname -f
deep-cp-01

root@deep-cp-01:/etc/systemd/network# getent hosts "$(hostname)"
192.168.100.50  deep-cp-01

root@deep-cp-01:/etc/systemd/network# cat /etc/hosts
127.0.0.1 localhost
192.168.100.50 deep-cp-01
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

**deep-wrk-01**:  
```
root@deep-wrk-01:/etc/systemd# hostname
deep-wrk-01

root@deep-wrk-01:/etc/systemd# hostname -f
deep-wrk-01

root@deep-wrk-01:/etc/systemd# getent hosts "$(hostname)"
192.168.100.51  deep-wrk-01

root@deep-wrk-01:/etc/systemd# cat /etc/hosts
127.0.0.1 localhost
192.168.100.51 deep-wrk-01
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
```

**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# hostname
deep-wrk-02

root@deep-wrk-02:/home/goblin# hostname -f
deep-wrk-02

root@deep-wrk-02:/home/goblin# getent hosts "$(hostname)"
192.168.100.52  deep-wrk-02

root@deep-wrk-02:/home/goblin# cat /etc/hosts
127.0.0.1 localhost
192.168.100.52 deep-wrk-02
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

-----

Проверка node-to-node L3 connectivity

**deep-cp-01**:  
```
root@deep-cp-01:/etc/systemd/network# ping -c 3 192.168.100.51
PING 192.168.100.51 (192.168.100.51) 56(84) bytes of data.
64 bytes from 192.168.100.51: icmp_seq=1 ttl=64 time=0.559 ms
64 bytes from 192.168.100.51: icmp_seq=2 ttl=64 time=0.823 ms
64 bytes from 192.168.100.51: icmp_seq=3 ttl=64 time=0.346 ms

--- 192.168.100.51 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2088ms
rtt min/avg/max/mdev = 0.346/0.576/0.823/0.195 ms

root@deep-cp-01:/etc/systemd/network# ping -c 3 192.168.100.52
PING 192.168.100.52 (192.168.100.52) 56(84) bytes of data.
64 bytes from 192.168.100.52: icmp_seq=1 ttl=64 time=0.614 ms
64 bytes from 192.168.100.52: icmp_seq=2 ttl=64 time=0.593 ms
64 bytes from 192.168.100.52: icmp_seq=3 ttl=64 time=0.708 ms

--- 192.168.100.52 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2027ms
rtt min/avg/max/mdev = 0.593/0.638/0.708/0.050 ms
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/etc/systemd# ping -c3 192.168.100.50
PING 192.168.100.50 (192.168.100.50) 56(84) bytes of data.
64 bytes from 192.168.100.50: icmp_seq=1 ttl=64 time=0.460 ms
64 bytes from 192.168.100.50: icmp_seq=2 ttl=64 time=0.769 ms
64 bytes from 192.168.100.50: icmp_seq=3 ttl=64 time=0.673 ms

--- 192.168.100.50 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2072ms
rtt min/avg/max/mdev = 0.460/0.634/0.769/0.129 ms

root@deep-wrk-01:/etc/systemd# ping -c3 192.168.100.52
PING 192.168.100.52 (192.168.100.52) 56(84) bytes of data.
64 bytes from 192.168.100.52: icmp_seq=1 ttl=64 time=0.581 ms
64 bytes from 192.168.100.52: icmp_seq=2 ttl=64 time=0.759 ms
64 bytes from 192.168.100.52: icmp_seq=3 ttl=64 time=0.703 ms

--- 192.168.100.52 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2037ms
rtt min/avg/max/mdev = 0.581/0.681/0.759/0.074 ms
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# ping -c 3 192.168.100.50
PING 192.168.100.50 (192.168.100.50) 56(84) bytes of data.
64 bytes from 192.168.100.50: icmp_seq=1 ttl=64 time=0.409 ms
64 bytes from 192.168.100.50: icmp_seq=2 ttl=64 time=0.388 ms
64 bytes from 192.168.100.50: icmp_seq=3 ttl=64 time=0.720 ms

--- 192.168.100.50 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2033ms
rtt min/avg/max/mdev = 0.388/0.505/0.720/0.151 ms

root@deep-wrk-02:/home/goblin# ping -c 3 192.168.100.51
PING 192.168.100.51 (192.168.100.51) 56(84) bytes of data.
64 bytes from 192.168.100.51: icmp_seq=1 ttl=64 time=0.311 ms
64 bytes from 192.168.100.51: icmp_seq=2 ttl=64 time=0.570 ms
64 bytes from 192.168.100.51: icmp_seq=3 ttl=64 time=0.384 ms

--- 192.168.100.51 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2085ms
rtt min/avg/max/mdev = 0.311/0.421/0.570/0.109 ms
```

-----

Проверка swap

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# swapon --show

root@deep-cp-01:/home/goblin# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       374Mi       7.3Gi       4.7Mi       195Mi       7.3Gi
Swap:             0B          0B          0B
```

**deep-wrk-01**:  
```
root@deep-wrk-01:/etc/systemd# swapon --show

root@deep-wrk-01:/etc/systemd# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       381Mi       7.3Gi       4.7Mi       190Mi       7.3Gi
Swap:             0B          0B          0B
```

**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# swapon --show

root@deep-wrk-02:/home/goblin# free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       397Mi       7.3Gi       4.7Mi       176Mi       7.3Gi
Swap:             0B          0B          0B
```

-----

Kernel modules и sysctl для Kubernetes networking

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

root@deep-cp-01:/home/goblin# sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
root@deep-cp-01:/home/goblin# sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
root@deep-cp-01:/home/goblin# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

root@deep-wrk-01:/home/goblin# sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
root@deep-wrk-01:/home/goblin# sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
root@deep-wrk-01:/home/goblin# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0

root@deep-wrk-02:/home/goblin# sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
root@deep-wrk-02:/home/goblin# sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
root@deep-wrk-02:/home/goblin# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

-----

Проверка чистого состояния node

**deep-cp-01**:  
```
root@deep-cp-01:/home/goblin# command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true
dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

**deep-wrk-01**:  
```
root@deep-wrk-01:/home/goblin# command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true
dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

**deep-wrk-02**:  
```
root@deep-wrk-02:/home/goblin# command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true
dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

### Ключевые выводы команд


### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| вывод getent hosts "$(hostname)" |  |  |  удаление строки `Domains=local` из конфигурации сети |

### Что стало понятнее

- 
- 
- 

### Вопросы

- Прописал бы в hosts адеса и имена всех нод, что бы ноды видели друг-друга по имени.
- Были слухи, что отключение свопа не обязятельно с 1.36 версии, но подтверждения не искал :)
- 

### Статус

done




