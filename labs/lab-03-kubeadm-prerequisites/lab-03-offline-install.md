# Lab 03 - Offline installation path

## Назначение

Этот документ описывает offline/fallback-вариант установки Kubernetes packages для Lab 03.

Используйте этот вариант, если прямой доступ к `pkgs.k8s.io` медленный, нестабильный или недоступен с ваших nodes.

Этот вариант не является официальным Kubernetes mirror. Это учебный package bundle для Kubernetes Deep Lab, заранее собранный из официального Kubernetes package repository и зафиксированный по версиям.

## Что входит в bundle

Bundle предназначен для Ubuntu/Debian nodes с архитектурой `amd64`.

В архиве ожидаются пакеты:

- `kubelet` `1.36.1-1.1`
- `kubeadm` `1.36.1-1.1`
- `kubectl` `1.36.1-1.1`
- `kubernetes-cni` `1.9.1-1.1`

Архив:

```text
kdl-k8s-packages-v1.36.1-amd64.tar.gz
```

Checksum-файл:

```text
kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
```

## Ссылки на bundle

```text
https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz
```

```text
https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
```

## Где выполнять

Выполнять последовательно на каждой node:

- control-plane node;
- worker nodes.

Если действие одинаковое для всех nodes, оно не расписывается отдельно для каждой node.

## Предварительная проверка architecture

Команда ничего не меняет.

Выполнить на каждой node:

```bash
dpkg --print-architecture
```

Ожидаем:

```text
amd64
```

Если вывод отличается от `amd64`, этот bundle использовать нельзя.

## Проверка текущего состояния Kubernetes packages

Команды ничего не меняют.

Выполнить на каждой node:

```bash
dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni' || true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl' || true
apt-cache policy kubelet kubeadm kubectl kubernetes-cni || true
```

Если `kubelet`, `kubeadm`, `kubectl` находятся в `hold`, но не установлены, снимите `hold` перед установкой:

```bash
sudo apt-mark unhold kubelet kubeadm kubectl || true
```

Проверка:

```bash
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl' || true
```

Ожидаемый вывод после снятия `hold` - пустой.

## Скачивание bundle

Что будет изменено:

- в домашнем каталоге пользователя будет создан каталог `~/kdl-packages`;
- в него будут скачаны архив и checksum-файл.

Выполнить на каждой node:

```bash
mkdir -p ~/kdl-packages
cd ~/kdl-packages
```

```bash
curl -fL -O https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz
```

```bash
curl -fL -O https://s3.twcstorage.ru/packages-k8s-v1-36-1-amd64/kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
```

Проверка:

```bash
ls -lh
```

Ожидаем два файла:

```text
kdl-k8s-packages-v1.36.1-amd64.tar.gz
kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
```

## Проверка checksum архива

Команда ничего не меняет.

Выполнить на каждой node:

```bash
sha256sum -c kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
```

Ожидаем:

```text
kdl-k8s-packages-v1.36.1-amd64.tar.gz: OK
```

Если checksum не совпал, не продолжайте установку. Удалите скачанные файлы и скачайте bundle повторно.

## Распаковка bundle

Что будет изменено:

- архив будет распакован в текущий каталог.

Выполнить на каждой node:

```bash
tar -xzf kdl-k8s-packages-v1.36.1-amd64.tar.gz
cd v1.36.1-amd64
ls -lh
```

Ожидаем файлы:

```text
kubeadm_1.36.1-1.1_amd64.deb
kubectl_1.36.1-1.1_amd64.deb
kubelet_1.36.1-1.1_amd64.deb
kubernetes-cni_1.9.1-1.1_amd64.deb
SHA256SUMS
```

## Проверка checksum пакетов внутри bundle

Команда ничего не меняет.

Выполнить на каждой node:

```bash
sha256sum -c SHA256SUMS
```

Ожидаем `OK` для каждого `.deb` файла.

Если checksum не совпал, не продолжайте установку.

## Offline-установка Kubernetes packages

### Если Kubernetes apt repository уже был подключен

Если ранее вы уже пробовали online-установку через `pkgs.k8s.io`, перед offline-установкой нужно временно отключить Kubernetes apt repository.

Иначе `apt` может выбрать пакеты из сети вместо локальных `.deb` и снова попытаться скачать их с `prod-cdn.packages.k8s.io`.

Проверить, подключен ли Kubernetes repository:

```bash
ls -l /etc/apt/sources.list.d/ | grep kubernetes || true
grep -R "pkgs.k8s.io" /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null || true
```
Если файл /etc/apt/sources.list.d/kubernetes.list существует, отключить его:
```sh
sudo mv /etc/apt/sources.list.d/kubernetes.list /etc/apt/sources.list.d/kubernetes.list.disabled
sudo apt-get update
```

Dry-run:

```bash
sudo apt-get install --dry-run ./*.deb
```

Если dry-run выглядит корректно, выполнить установку:

```bash
sudo apt-get install -y ./*.deb
```

Зафиксировать версии:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

## Проверка установки

Команды ничего не меняют.

Выполнить на каждой node:

```bash
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
```

Ожидаем:

```text
v1.36.1
Kubernetes v1.36.1
Client Version: v1.36.1
kubeadm
kubectl
kubelet
```

Формат вывода может немного отличаться, но версии должны быть `v1.36.1`.

## Проверка kubelet до bootstrap


Команды ничего не меняют.

Выполнить на каждой node:

```bash
systemctl status kubelet --no-pager || true
```

```bash
journalctl -u kubelet -n 50 --no-pager
```

На этом этапе `kubelet` может быть в состоянии ошибки или постоянно перезапускаться.

Это ожидаемо до выполнения:

- `kubeadm init` на control-plane;
- `kubeadm join` на worker nodes.

Причина: `kubelet` уже установлен как systemd service, но node еще не прошла bootstrap. У kubelet еще может не быть полноценной конфигурации, kubeconfig и связи с Kubernetes API Server.

## Финальная проверка offline-варианта

Выполнить на каждой node:

```bash
hostname
dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni'
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl is-enabled kubelet
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
```

Ожидаем:

- `kubeadm`, `kubelet`, `kubectl`, `kubernetes-cni` установлены;
- версии Kubernetes packages совпадают на всех nodes;
- `kubelet`, `kubeadm`, `kubectl` находятся в `apt-mark hold`;
- `RuntimeReady` по-прежнему `true`;
- `NetworkReady` может оставаться `false` до установки CNI.

## Rollback


Rollback нужен только если установка пакетов прошла некорректно или выбран неправильный bundle.

Сначала dry-run:

```bash
sudo apt-get remove --dry-run kubelet kubeadm kubectl kubernetes-cni
```

Если dry-run выглядит ожидаемо, можно удалить пакеты:

```bash
sudo apt-mark unhold kubelet kubeadm kubectl || true
sudo apt-get remove -y kubelet kubeadm kubectl kubernetes-cni
```

Проверка после rollback:

```bash
dpkg -l | grep -E 'kubelet|kubeadm|kubectl|kubernetes-cni' || true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl' || true
```

Важно: не выполнять rollback только из-за того, что `kubelet` находится в ошибке до `kubeadm init/join`. На этом этапе это ожидаемое состояние.

## Критерии готовности

Offline-вариант Lab 03 считается завершенным, если:

- archive checksum проверен;
- checksums всех `.deb` packages проверены;
- `kubeadm` установлен на всех nodes;
- `kubelet` установлен на всех nodes;
- `kubectl` установлен на всех nodes;
- `kubernetes-cni` установлен на всех nodes;
- версии Kubernetes packages совпадают на всех nodes;
- пакеты `kubelet`, `kubeadm`, `kubectl` находятся в `apt-mark hold`;
- понятно, почему `kubelet` до `kubeadm init/join` может быть не ready;
- `containerd` по-прежнему работает;
- `crictl` по-прежнему показывает `RuntimeReady: true`.

## Что приложить в lab journal

Добавьте в lab journal вывод с control-plane и только отличия с worker nodes:

```bash
hostname
dpkg --print-architecture
sha256sum -c kdl-k8s-packages-v1.36.1-amd64.tar.gz.sha256
cd ~/kdl-packages/v1.36.1-amd64
sha256sum -c SHA256SUMS
kubeadm version -o short
kubelet --version
kubectl version --client=true
apt-mark showhold | grep -E 'kubelet|kubeadm|kubectl'
systemctl status kubelet --no-pager || true
sudo crictl info | grep -E '"RuntimeReady"|"NetworkReady"|"runtimeType"|"SystemdCgroup"'
```

Если вывод на worker nodes совпадает с control-plane, достаточно написать, что проверка повторена на всех nodes, и приложить только отличия.
