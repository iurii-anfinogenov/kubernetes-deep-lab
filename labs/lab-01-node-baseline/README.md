# Lab 01 - Node Baseline

Эта лабораторная подготавливает Linux nodes к установке Kubernetes.

Цель Lab 01 - не установить Kubernetes, а привести каждую VM к предсказуемому базовому состоянию:

- корректный hostname и IP;
- стабильный local name resolution;
- рабочая сеть между nodes;
- доступ к package repositories;
- синхронизированное время;
- выключенный swap;
- включенные kernel modules и sysctl для Kubernetes networking;
- отсутствие активных package manager locks.

После этой лабораторной nodes должны быть готовы к следующему шагу: установке container runtime.

## Что уже должно быть готово

Перед началом Lab 01 участник должен пройти `lab-00-environment`.

Должно быть подготовлено минимум 2 VM:

| Node | Role | CPU | RAM | Disk |
|---|---|---:|---:|---:|
| control-plane-1 | control-plane | 2 vCPU | 4 GB | 30 GB или больше |
| worker-1 | worker | 2 vCPU | 4 GB | 40 GB или больше |

Рекомендуемый стенд:

| Node | Role | CPU | RAM | Disk |
|---|---|---:|---:|---:|
| control-plane-1 | control-plane | 2 vCPU | 4 GB | 30 GB или больше |
| worker-1 | worker | 2 vCPU | 4 GB | 40 GB или больше |
| worker-2 | worker | 2 vCPU | 4 GB | 40 GB или больше |

Reference run проекта выполнялся на таком inventory:

| Node | Role | IP | OS |
|---|---|---:|---|
| kdl-cp-1 | control-plane | 192.168.20.11 | Ubuntu 24.04.3 LTS |
| kdl-w-5 | worker | 192.168.20.15 | Ubuntu 24.04.3 LTS |
| kdl-w-6 | worker | 192.168.20.16 | Ubuntu 24.04.3 LTS |

Участник может использовать другие имена и IP, но должен сохранить тот же принцип:

- у каждой node уникальный hostname;
- у каждой node стабильный internal IP;
- все nodes находятся в одной доступной сети или имеют маршрутизацию друг до друга;
- с admin/manager workstation есть SSH-доступ до каждой VM.

SSH между Kubernetes nodes не требуется.

## Выполнение команд

Если действие одинаковое для всех nodes, выполняйте его последовательно на каждой node.

## Проверка OS, hostname, IP и ресурсов

Выполнить на каждой node:

```bash
hostnamectl
ip -br addr
ip route
cat /etc/os-release
free -h
df -h /
nproc
```

Критерий готовности:

```text
OS: Ubuntu 24.04 LTS или явно описанный другой Linux
CPU: минимум 2 vCPU
RAM: минимум 4 GB
Swap: 0B
Root disk: достаточно свободного места
IP: стабильный internal IP
Default route: присутствует
```

Для reference run ожидаемое состояние было таким:

```text
hostname: kdl-cp-1
ip: 192.168.20.11/24
os: Ubuntu 24.04.3 LTS
cpu: 2 vCPU
ram: 3.8 GiB
disk: 38G root
swap: 0B
```

## Проверка DNS и internet access

Выполнить на control-plane node:

```bash
getent hosts archive.ubuntu.com
ping -c 3 192.168.20.1
ping -c 3 1.1.1.1
ping -c 3 archive.ubuntu.com
```

Замените `192.168.20.1` на gateway вашей сети.

Критерий готовности:

```text
gateway доступен
внешний IPv4 доступен
DNS работает
Ubuntu mirror доступен
packet loss: 0%
```

## Проверка доступности package repositories

Участники могут находиться в разных регионах, поэтому доступность репозиториев нужно проверить заранее.

Выполнить на control-plane node:

```bash
getent hosts archive.ubuntu.com
getent hosts security.ubuntu.com
getent hosts pkgs.k8s.io
getent hosts github.com

curl -I --connect-timeout 10 https://archive.ubuntu.com/ubuntu/
curl -I --connect-timeout 10 https://pkgs.k8s.io/
curl -I --connect-timeout 10 https://github.com/
```

Для Ubuntu 24.04 дополнительно:

```bash
sudo apt update
apt-cache policy containerd runc
```

Критерий готовности:

```text
DNS-запросы возвращают IP-адреса
curl возвращает HTTP status 200, 301 или 302
apt update завершается без ошибок
apt-cache policy показывает доступные пакеты containerd и runc
```

Если репозитории недоступны, это технический блокер для online path. Offline / air-gapped установку мы разберем позже отдельным advanced-блоком.

## Проверка time sync

Выполнить на control-plane node:

```bash
timedatectl
systemctl is-active systemd-timesyncd
timedatectl show -p NTPSynchronized -p TimeUSec -p Timezone
```

Критерий готовности:

```text
System clock synchronized: yes
NTP service: active
NTPSynchronized=yes
```

Почему это важно:

```text
Kubernetes использует TLS certificates, kubelet bootstrap и etcd.
Сильный рассинхрон времени может привести к проблемам с сертификатами и cluster health.
```

## Проверка hostname resolution

Выполнить на каждой node:

```bash
hostname
hostname -f
getent hosts "$(hostname)"
cat /etc/hosts
```

Критерий готовности:

```text
hostname возвращает имя текущей node
hostname -f не уходит во внешний случайный домен
getent hosts "$(hostname)" возвращает internal IP текущей node
```

Пример корректного результата:

```text
kdl-cp-1
kdl-cp-1
192.168.20.11 kdl-cp-1
```

Плохой пример:

```text
hostname -f: site.ru
getent hosts kdl-cp-1: 146.66.162.134 site.ru kdl-cp-1.site.ru
```

Такое состояние нужно исправить до установки Kubernetes.

Причина может быть в DNS search domain. Например, короткое имя `kdl-cp-1` может дополняться до `kdl-cp-1.example.com` и резолвиться во внешний DNS.

## Ручное исправление /etc/hosts

На каждой node добавьте локальную запись:

```text
<node_internal_ip> <node_hostname>
```

Пример для `kdl-cp-1`:

```bash
sudo cp /etc/hosts /etc/hosts.bak.$(date +%Y%m%d-%H%M%S)

cat <<'EOF' | sudo tee /etc/hosts
127.0.0.1 localhost
192.168.20.11 kdl-cp-1

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
EOF
```

Для других nodes замените IP и hostname на свои значения.

Rollback:

```bash
ls -1t /etc/hosts.bak.* | head -n 1
sudo cp "$(ls -1t /etc/hosts.bak.* | head -n 1)" /etc/hosts
```

Проверка:

```bash
hostname
hostname -f
getent hosts "$(hostname)"
cat /etc/hosts
```

## Проверка node-to-node L3 connectivity

Выполнить с каждой node до остальных node IP:

```bash
ping -c 3 <other_node_ip>
```

Пример:

```bash
ping -c 3 192.168.20.15
ping -c 3 192.168.20.16
```

## Проверка swap

Выполнить на каждой node:

```bash
swapon --show
free -h
```

Критерий готовности:

```text
swapon --show не выводит активных swap devices/files
free -h показывает Swap: 0B
```

Если swap включен, временно отключить:

```bash
sudo swapoff -a
```

Затем найти swap-запись в `/etc/fstab`:

```bash
grep -nE '\sswap\s' /etc/fstab
```

Дальше нужно аккуратно закомментировать swap entry в `/etc/fstab`.

`/etc/fstab` влияет на загрузку системы. Перед правкой обязательно сделать backup:

```bash
sudo cp /etc/fstab /etc/fstab.bak.$(date +%Y%m%d-%H%M%S)
```

После правки проверить:

```bash
sudo findmnt --verify
swapon --show
free -h
```

После reboot проверить снова:

```bash
swapon --show
free -h
```

## Kernel modules и sysctl для Kubernetes networking

Сначала проверить текущее состояние:

```bash
lsmod | grep -E 'br_netfilter|overlay'
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward
```

Если `br_netfilter` не загружен, параметры `net.bridge.*` могут отсутствовать. Это нормальное состояние для чистой VM.

Включить runtime-only для проверки:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
sudo sysctl -w net.bridge.bridge-nf-call-iptables=1
sudo sysctl -w net.bridge.bridge-nf-call-ip6tables=1
sudo sysctl -w net.ipv4.ip_forward=1
```

Проверить:

```bash
lsmod | grep -E 'br_netfilter|overlay'
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward
```

Критерий готовности:

```text
overlay загружен
br_netfilter загружен
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
```

## Persistent настройка kernel modules и sysctl

Выполнить на каждой node:

```bash
cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

cat <<'EOF' | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
sudo sysctl --system
```

Проверить:

```bash
cat /etc/modules-load.d/k8s.conf
cat /etc/sysctl.d/k8s.conf
lsmod | grep -E 'br_netfilter|overlay'
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward
```

Проверка persistence после reboot:

```bash
sudo reboot
```

После возврата VM:

```bash
hostname
lsmod | grep -E 'br_netfilter|overlay'
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward
```

## Проверка чистого состояния node

Выполнить на каждой node:

```bash
command -v containerd || true
command -v runc || true
command -v crictl || true
command -v kubeadm || true
command -v kubelet || true
command -v kubectl || true

dpkg -l | grep -E 'containerd|runc|kubeadm|kubelet|kubectl|kubernetes' || true
systemctl list-unit-files | grep -E 'containerd|kubelet' || true
```

Критерий готовности для этой лабораторной:

```text
containerd еще не установлен
kubeadm еще не установлен
kubelet еще не установлен
kubectl еще не установлен
```

Если что-то уже установлено, это нужно явно указать в lab report.

## Итоговый checkpoint

К концу Lab 01 должно быть выполнено:

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

## Что приложить в lab report

В `students/<github-username>/` создать или обновить отчет по Lab 01.

Минимально приложить:

```text
1. Inventory nodes: hostname, role, IP, OS.
2. Вывод hostname / hostname -f / getent hosts "$(hostname)".
3. Подтверждение node-to-node ping.
4. Подтверждение Swap: 0B.
5. Подтверждение loaded modules: overlay, br_netfilter.
6. Подтверждение sysctl:
   - net.bridge.bridge-nf-call-iptables = 1
   - net.bridge.bridge-nf-call-ip6tables = 1
   - net.ipv4.ip_forward = 1
7. Подтверждение package repositories access.
8. Подтверждение отсутствия kubeadm/kubelet/containerd до следующей лабораторной.
```

## Следующая лабораторная

Следующий шаг:

```text
Lab 02 - Container Runtime: containerd
```

В ней будут:

```text
containerd
runc
/etc/containerd/config.toml
SystemdCgroup = true
CRI plugin
crictl
разница между ctr и crictl
```
