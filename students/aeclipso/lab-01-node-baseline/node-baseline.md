# Lab Journal

Lab journal - рабочий журнал участника Kubernetes Deep Lab.

Он нужен, чтобы фиксировать прогресс, команды, выводы, ошибки, вопросы и выводы по каждой лабораторной.

## Участник

| Поле | Значение |
|---|---|
| Имя | aeclipso |
| GitHub | https://github.com/aeclipso/ |

## Общий прогресс

| Lab | Status | PR | Notes |
|---|---|---|---|
| Lab 01 - Node Baseline | done |  |  |

## Lab 01 - Node Baseline

### Дата

2026-05-27

### Цель

Подготовка виртуалок к установке кубера.

## Проверка OS, hostname, IP и ресурсов

```
ae@control-plane-1:~$ hostnamectl
 Static hostname: control-plane-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: bc9d8086e24841519e3490412e320bac
         Boot ID: c46fc50f3020444581e9f0a63f68064f
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 5d                            
ae@control-plane-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.40/24 metric 100 fe80::be24:11ff:feca:26e2/64 
ae@control-plane-1:~$ 
ae@control-plane-1:~$ ip route
default via 192.168.1.1 dev ens18 proto dhcp src 192.168.1.40 metric 100 
192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.40 metric 100 
192.168.1.1 dev ens18 proto dhcp scope link src 192.168.1.40 metric 100 
ae@control-plane-1:~$ cat /etc/os-release
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
ae@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       385Mi       3.5Gi       960Ki       208Mi       3.4Gi
Swap:             0B          0B          0B
ae@control-plane-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        30G  6.7G   22G  24% /
ae@control-plane-1:~$ nproc
2
```

```
ae@worker-1:~$ hostnamectl
 Static hostname: worker-1
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: bc3717c339394ea18f75f99dfb797a8a
         Boot ID: b04c2d27b0db4f37b064b5d2f322ab07
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 5d                            
ae@worker-1:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.43/24 metric 100 fe80::be24:11ff:fe2b:39bf/64 
ae@worker-1:~$ ip route
default via 192.168.1.1 dev ens18 proto dhcp src 192.168.1.43 metric 100 
192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.43 metric 100 
192.168.1.1 dev ens18 proto dhcp scope link src 192.168.1.43 metric 100 
ae@worker-1:~$ cat /etc/os-release
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
ae@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       391Mi       3.4Gi       964Ki       209Mi       3.4Gi
Swap:             0B          0B          0B
ae@worker-1:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  6.7G   31G  18% /
ae@worker-1:~$ nproc
2
```

```
ae@worker-2:~$ hostnamectl
 Static hostname: worker-2
       Icon name: computer-vm
         Chassis: vm 🖴
      Machine ID: c4fd594a622943e9b6ccd7dd01632aeb
         Boot ID: 9c5d514bf583478bbb8be2413383abdd
  Virtualization: kvm
Operating System: Ubuntu 24.04.4 LTS                          
          Kernel: Linux 6.8.0-117-generic
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
Firmware Version: rel-1.17.0-0-gb52ca86e094d-prebuilt.qemu.org
   Firmware Date: Tue 2014-04-01
    Firmware Age: 12y 1month 3w 5d                            
ae@worker-2:~$ ip -br addr
lo               UNKNOWN        127.0.0.1/8 ::1/128 
ens18            UP             192.168.1.46/24 metric 100 fe80::be24:11ff:fe80:46e4/64 
ae@worker-2:~$ ip route
default via 192.168.1.1 dev ens18 proto dhcp src 192.168.1.46 metric 100 
192.168.1.0/24 dev ens18 proto kernel scope link src 192.168.1.46 metric 100 
192.168.1.1 dev ens18 proto dhcp scope link src 192.168.1.46 metric 100 
ae@worker-2:~$ cat /etc/os-release
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
ae@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       392Mi       3.4Gi       964Ki       208Mi       3.4Gi
Swap:             0B          0B          0B
ae@worker-2:~$ df -h /
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        40G  6.7G   31G  18% /
ae@worker-2:~$ nproc
2
```

## Проверка DNS и internet access

```
ae@control-plane-1:~$ getent hosts archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
ae@control-plane-1:~$ ping -c 3 192.168.1.1
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.328 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.526 ms
64 bytes from 192.168.1.1: icmp_seq=3 ttl=64 time=0.534 ms

--- 192.168.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2086ms
rtt min/avg/max/mdev = 0.328/0.462/0.534/0.095 ms
ae@control-plane-1:~$ ping -c 3 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=57 time=15.6 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=57 time=16.0 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=57 time=25.6 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 15.567/19.067/25.636/4.648 ms
ae@control-plane-1:~$ ping -c 3 archive.ubuntu.com
PING archive.ubuntu.com.cdn.cloudflare.net (104.20.28.246) 56(84) bytes of data.
64 bytes from 104.20.28.246: icmp_seq=1 ttl=54 time=58.8 ms
64 bytes from 104.20.28.246: icmp_seq=2 ttl=54 time=51.5 ms
64 bytes from 104.20.28.246: icmp_seq=3 ttl=54 time=53.9 ms

--- archive.ubuntu.com.cdn.cloudflare.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 51.526/54.745/58.785/3.019 ms

```

## Проверка доступности package repositories
```
ae@control-plane-1:~$ getent hosts archive.ubuntu.com
2606:4700:10::6814:1cf6 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
2606:4700:10::ac42:98b0 archive.ubuntu.com.cdn.cloudflare.net archive.ubuntu.com
ae@control-plane-1:~$ getent hosts security.ubuntu.com
2606:4700:10::6814:1cf6 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
2606:4700:10::ac42:98b0 security.ubuntu.com.cdn.cloudflare.net security.ubuntu.com
ae@control-plane-1:~$ getent hosts pkgs.k8s.io
2600:1901:0:26f3:: redirect.k8s.io pkgs.k8s.io
ae@control-plane-1:~$ getent hosts github.com
140.82.121.3    github.com

ae@control-plane-1:~$ curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
HTTP/2 200 
date: Wed, 27 May 2026 18:39:35 GMT
content-type: text/html;charset=UTF-8
server: cloudflare
set-cookie: __cf_bm=6EzRd1LrS5j1yU.Ka63zMY3pd1a8rFmwtbUnmwdanB8-1779907174.939843-1.0.1.1-T0tpwEKAxAQcxgnPK8ROzVEAv.lVTzpeur7Fz.qnPZ.WcL2zNhS7nTFFSJgDO038clXxD270j4Y_lAFyv4kfj4aXwSrKiybUhl8r8EsZmf9ghcjGQRXA8l5ZzHdssvFu; HttpOnly; SameSite=None; Secure; Path=/; Domain=ubuntu.com; Expires=Wed, 27 May 2026 19:09:35 GMT
cf-cache-status: DYNAMIC
cf-ray: a02724a35ba430ed-FRA

ae@control-plane-1:~$ curl -I --connect-timeout 10 https://pkgs.k8s.io/
HTTP/2 302 
server: nginx
date: Wed, 27 May 2026 18:39:39 GMT
content-type: text/html
content-length: 138
location: https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/
via: 1.1 google
alt-svc: h3=":443"; ma=2592000

ae@control-plane-1:~$ curl -I --connect-timeout 10 https://github.com/
HTTP/2 200 
date: Wed, 27 May 2026 18:39:34 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Accept-Language, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
content-language: en-US
etag: W/"9347d84d89353be40cb781c021b23a77"
cache-control: max-age=0, private, must-revalidate
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: origin-when-cross-origin, strict-origin-when-cross-origin
content-security-policy: default-src 'none'; base-uri 'self'; child-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/; connect-src 'self' uploads.github.com www.githubstatus.com collector.github.com raw.githubusercontent.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com *.rel.tunnels.api.visualstudio.com wss://*.rel.tunnels.api.visualstudio.com github.githubassets.com objects-origin.githubusercontent.com copilot-proxy.githubusercontent.com proxy.individual.githubcopilot.com proxy.business.githubcopilot.com proxy.enterprise.githubcopilot.com *.actions.githubusercontent.com wss://*.actions.githubusercontent.com productionresultssa0.blob.core.windows.net productionresultssa1.blob.core.windows.net productionresultssa2.blob.core.windows.net productionresultssa3.blob.core.windows.net productionresultssa4.blob.core.windows.net productionresultssa5.blob.core.windows.net productionresultssa6.blob.core.windows.net productionresultssa7.blob.core.windows.net productionresultssa8.blob.core.windows.net productionresultssa9.blob.core.windows.net productionresultssa10.blob.core.windows.net productionresultssa11.blob.core.windows.net productionresultssa12.blob.core.windows.net productionresultssa13.blob.core.windows.net productionresultssa14.blob.core.windows.net productionresultssa15.blob.core.windows.net productionresultssa16.blob.core.windows.net productionresultssa17.blob.core.windows.net productionresultssa18.blob.core.windows.net productionresultssa19.blob.core.windows.net github-production-repository-image-32fea6.s3.amazonaws.com github-production-release-asset-2e65be.s3.amazonaws.com insights.github.com wss://alive.github.com wss://alive-staging.github.com api.githubcopilot.com api.individual.githubcopilot.com api.business.githubcopilot.com api.enterprise.githubcopilot.com edge.fullstory.com rs.fullstory.com; font-src github.githubassets.com; form-action 'self' github.com gist.github.com copilot-workspace.githubnext.com objects-origin.githubusercontent.com; frame-ancestors 'none'; frame-src viewscreen.githubusercontent.com notebooks.githubusercontent.com www.youtube-nocookie.com; img-src 'self' data: blob: github.githubassets.com media.githubusercontent.com camo.githubusercontent.com identicons.github.com avatars.githubusercontent.com private-avatars.githubusercontent.com github-cloud.s3.amazonaws.com objects.githubusercontent.com release-assets.githubusercontent.com secured-user-images.githubusercontent.com user-images.githubusercontent.com private-user-images.githubusercontent.com opengraph.githubassets.com marketplace-screenshots.githubusercontent.com copilotprodattachments.blob.core.windows.net/github-production-copilot-attachments/ github-production-user-asset-6210df.s3.amazonaws.com customer-stories-feed.github.com spotlights-feed.github.com explore-feed.github.com objects-origin.githubusercontent.com *.githubusercontent.com images.ctfassets.net/8aevphvgewt8/; manifest-src 'self'; media-src github.com user-images.githubusercontent.com secured-user-images.githubusercontent.com private-user-images.githubusercontent.com github-production-user-asset-6210df.s3.amazonaws.com gist.github.com github.githubassets.com assets.ctfassets.net/8aevphvgewt8/ videos.ctfassets.net/8aevphvgewt8/; script-src github.githubassets.com; style-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
accept-ranges: bytes
set-cookie: _gh_sess=l5RdH7iMD%2FP50fAUIq98iSiIC1tM3g4rB5IqWji5GeqkedvgHT%2FoD5A6QNsBpQFj%2B7K7FtiGcDXHPS8erZdZE5VU8%2Bo0XMrugOEarmBmxJA876vCRNoFkZBjTEiVHud%2FBoWZkPBvgNuMluIqPr8s1rwJx2B2ivsnvtA3zGnMEp7hN7GxI3Hqv36pQW5PMJj9CEECzznn2gkwLyePb3v5WfV0y0GVJ01%2BqWlABq35RPD8GT7ldyHrwzew1NexT0qnQQ3QFIl4OApBueuMv0jmxA%3D%3D--y01qeKGKE1RjyT97--PkEgLklknYDHgt1KrtxpSg%3D%3D; path=/; HttpOnly; secure; SameSite=Lax
set-cookie: _octo=GH1.1.1047510757.1779907184; expires=Thu, 27 May 2027 18:39:44 GMT; domain=.github.com; path=/; secure; SameSite=Lax
set-cookie: logged_in=no; expires=Thu, 27 May 2027 18:39:44 GMT; domain=.github.com; path=/; HttpOnly; secure; SameSite=Lax
x-github-request-id: 3E85:1C7DBF:3FCBD6:2F38B4:6A173A70

ae@control-plane-1:~$ sudo apt update
[sudo] password for ae: 
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease           
Hit:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease         
Hit:4 http://security.ubuntu.com/ubuntu noble-security InRelease            
Reading package lists... Done                         
Building dependency tree... Done
Reading state information... Done
37 packages can be upgraded. Run 'apt list --upgradable' to see them.

ae@control-plane-1:~$ apt-cache policy containerd runc
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

## Проверка time sync

```
ae@control-plane-1:~$ timedatectl
               Local time: Wed 2026-05-27 18:45:12 UTC
           Universal time: Wed 2026-05-27 18:45:12 UTC
                 RTC time: Wed 2026-05-27 18:45:12
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
ae@control-plane-1:~$ systemctl is-active systemd-timesyncd
active
ae@control-plane-1:~$ timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
Timezone=Etc/UTC
NTPSynchronized=yes
TimeUSec=Wed 2026-05-27 18:45:21 UTC
```

## Проверка hostname resolution

control-plane-1

```
ae@control-plane-1:~$ hostname
control-plane-1
ae@control-plane-1:~$ hostname -f
control-plane-1
ae@control-plane-1:~$ getent hosts "$(hostname)"
192.168.1.40    control-plane-1
ae@control-plane-1:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.1.40 control-plane-1

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

worker-1

```
ae@worker-1:~$ hostname
worker-1
ae@worker-1:~$ hostname -f
worker-1
ae@worker-1:~$ getent hosts "$(hostname)"
192.168.1.43    worker-1
ae@worker-1:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.1.43 worker-1

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

worker-2

```
ae@worker-2:~$ hostname
worker-2
ae@worker-2:~$ hostname -f
worker-2
ae@worker-2:~$ getent hosts "$(hostname)"
192.168.1.46    worker-2
ae@worker-2:~$ cat /etc/hosts
127.0.0.1 localhost
192.168.1.46 worker-2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

## Ручное исправление /etc/hosts

Исправил в прошлом пункте, до этого было:
```
127.0.1.1 <node_hostname>
```

## Проверка node-to-node L3 connectivity

control-plane-1
```
ae@control-plane-1:~$ ping -c 3 192.168.1.43
PING 192.168.1.43 (192.168.1.43) 56(84) bytes of data.
64 bytes from 192.168.1.43: icmp_seq=1 ttl=64 time=0.720 ms
64 bytes from 192.168.1.43: icmp_seq=2 ttl=64 time=0.308 ms
64 bytes from 192.168.1.43: icmp_seq=3 ttl=64 time=0.306 ms

--- 192.168.1.43 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2075ms
rtt min/avg/max/mdev = 0.306/0.444/0.720/0.194 ms
ae@control-plane-1:~$ ping -c 3 192.168.1.46
PING 192.168.1.46 (192.168.1.46) 56(84) bytes of data.
64 bytes from 192.168.1.46: icmp_seq=1 ttl=64 time=0.729 ms
64 bytes from 192.168.1.46: icmp_seq=2 ttl=64 time=0.270 ms
64 bytes from 192.168.1.46: icmp_seq=3 ttl=64 time=0.253 ms

--- 192.168.1.46 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2057ms
rtt min/avg/max/mdev = 0.253/0.417/0.729/0.220 ms
```

worker-1
```
ae@worker-1:~$ ping -c 3 192.168.1.40
PING 192.168.1.40 (192.168.1.40) 56(84) bytes of data.
64 bytes from 192.168.1.40: icmp_seq=1 ttl=64 time=0.370 ms
64 bytes from 192.168.1.40: icmp_seq=2 ttl=64 time=0.305 ms
64 bytes from 192.168.1.40: icmp_seq=3 ttl=64 time=0.292 ms

--- 192.168.1.40 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2088ms
rtt min/avg/max/mdev = 0.292/0.322/0.370/0.034 ms
ae@worker-1:~$ ping -c 3 192.168.1.46
PING 192.168.1.46 (192.168.1.46) 56(84) bytes of data.
64 bytes from 192.168.1.46: icmp_seq=1 ttl=64 time=0.931 ms
64 bytes from 192.168.1.46: icmp_seq=2 ttl=64 time=0.340 ms
64 bytes from 192.168.1.46: icmp_seq=3 ttl=64 time=0.212 ms

--- 192.168.1.46 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2055ms
rtt min/avg/max/mdev = 0.212/0.494/0.931/0.313 ms

```

worker-2
```
ae@worker-2:~$ ping -c 3 192.168.1.40
PING 192.168.1.40 (192.168.1.40) 56(84) bytes of data.
64 bytes from 192.168.1.40: icmp_seq=1 ttl=64 time=0.314 ms
64 bytes from 192.168.1.40: icmp_seq=2 ttl=64 time=0.310 ms
64 bytes from 192.168.1.40: icmp_seq=3 ttl=64 time=0.304 ms

--- 192.168.1.40 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2039ms
rtt min/avg/max/mdev = 0.304/0.309/0.314/0.004 ms
ae@worker-2:~$ ping -c 3 192.168.1.43
PING 192.168.1.43 (192.168.1.43) 56(84) bytes of data.
64 bytes from 192.168.1.43: icmp_seq=1 ttl=64 time=0.429 ms
64 bytes from 192.168.1.43: icmp_seq=2 ttl=64 time=0.286 ms
64 bytes from 192.168.1.43: icmp_seq=3 ttl=64 time=0.342 ms

--- 192.168.1.43 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2083ms
rtt min/avg/max/mdev = 0.286/0.352/0.429/0.058 ms
```

## Проверка swap
control-plane-1
```
ae@control-plane-1:~$ swapon --show
ae@control-plane-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       403Mi       3.0Gi       976Ki       624Mi       3.4Gi
Swap:             0B          0B          0B
```

worker-1
```
ae@worker-1:~$ swapon --show
ae@worker-1:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       393Mi       3.4Gi       960Ki       221Mi       3.4Gi
Swap:             0B          0B          0B
```

worker-2
```
ae@worker-2:~$ swapon --show
ae@worker-2:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       401Mi       3.4Gi       972Ki       243Mi       3.4Gi
Swap:             0B          0B          0B
```


## Persistent настройка kernel modules и sysctl

```
ae@control-plane-1:~$ hostname
control-plane-1
ae@control-plane-1:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
ae@control-plane-1:~$ sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
ae@control-plane-1:~$ sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
ae@control-plane-1:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

```
ae@worker-1:~$ hostname
worker-1
ae@worker-1:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
ae@worker-1:~$ sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
ae@worker-1:~$ sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
ae@worker-1:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```

```
ae@worker-2:~$ hostname
worker-2
ae@worker-2:~$ lsmod | grep -E 'br_netfilter|overlay'
br_netfilter           32768  0
bridge                425984  1 br_netfilter
overlay               212992  0
ae@worker-2:~$ sysctl net.bridge.bridge-nf-call-iptables
net.bridge.bridge-nf-call-iptables = 1
ae@worker-2:~$ sysctl net.bridge.bridge-nf-call-ip6tables
net.bridge.bridge-nf-call-ip6tables = 1
ae@worker-2:~$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```


## Проверка чистого состояния node

```
e@control-plane-1:~$ command -v containerd || true
ae@control-plane-1:~$ command -v runc || true
ae@control-plane-1:~$ command -v crictl || true
ae@control-plane-1:~$ command -v kubeadm || true
ae@control-plane-1:~$ command -v kubelet || true
ae@control-plane-1:~$ command -v kubectl || true
ae@control-plane-1:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
ae@control-plane-1:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

```
ae@worker-1:~$ command -v containerd || true
ae@worker-1:~$ command -v runc || true
ae@worker-1:~$ command -v crictl || true
ae@worker-1:~$ command -v kubeadm || true
ae@worker-1:~$ command -v kubelet || true
ae@worker-1:~$ command -v kubectl || true
ae@worker-1:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
ae@worker-1:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

```
ae@worker-2:~$ command -v containerd || true
ae@worker-2:~$ command -v runc || true
ae@worker-2:~$ command -v crictl || true
ae@worker-2:~$ command -v kubeadm || true
ae@worker-2:~$ command -v kubelet || true
ae@worker-2:~$ command -v kubectl || true
ae@worker-2:~$ dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
ae@worker-2:~$ systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```


## Итоговый checkpoint

```text
VM inventory: OK
OS/resources: OK
DNS/internet: OK
outbound HTTPS: OK
time sync: OK
hostname resolution: OK
node-to-node L3 connectivity: OK
swap disabled: OK
kernel/network prerequisites: OK
reboot persistence: OK
package manager readiness: OK
clean nodes, no Kubernetes/runtime: OK
```

### Ошибки и диагностика

| Симптом | Слой | Что проверил | Решение |
|---|---|---|---|
| hostname раньше был 127.0.1.1 |  |  | прописал IPv4 node в рамках сети |

### Что стало понятнее

- overlay - модуль ядра для ФС. Используется для container runtime. 
- br_netfilter - модуль, который заставляет bridge пропускать пакеты через iptables. Правила, которые будет создавать кубер будут обрабатываться iptables   


### Вопросы

- Persistent настройка kernel modules и sysctl, изначальное состояние не сохранилось, надо откатывать на выключенное, чтобы добавить в отчёт?


### Статус

done
