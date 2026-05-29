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
| Lab 02 - Container Runtime: containerd | done |  |  |

# Lab 02 - Container Runtime: containerd

### Дата

2026-05-28

### Цель

Подготовть conteiner runtime для будущего кластера.

## Проверка доступных пакетов

```
ae@control-plane-1:~$ sudo apt update
[sudo] password for ae: 
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:4 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:5 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [2,016 kB]
Get:6 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,704 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [356 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]          
Get:9 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [3,210 kB]      
Get:10 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [42.5 kB]     
Get:11 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1,192 kB]                   
Get:12 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [230 kB]               
Get:13 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]        
Get:14 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [9,000 B]            
Get:15 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [742 kB]
Get:16 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,693 kB]
Get:17 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]
Get:18 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:19 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [5,760 B]
Get:20 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]
Fetched 12.2 MB in 3s (4,667 kB/s)                                   
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
37 packages can be upgraded. Run 'apt list --upgradable' to see them.
ae@control-plane-1:~$ apt-cache policy containerd
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
ae@control-plane-1:~$ apt-cache policy runc
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
ae@control-plane-1:~$ 
```

```
ae@worker-1:~$ sudo apt update
[sudo] password for ae: 
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Get:5 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [2,016 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [356 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [3,210 kB]
Get:9 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,704 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [742 kB]      
Get:11 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,693 kB]             
Get:12 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [330 kB]           
Get:13 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]       
Get:14 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [11.1 kB]       
Get:15 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]
Get:16 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [5,760 B]        
Get:17 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]        
Get:18 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [268 kB]                  
Get:19 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [42.5 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [3,006 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [698 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1,192 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [230 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [9,000 B]
Fetched 16.5 MB in 3s (5,069 kB/s)                                   
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
ae@worker-1:~$ apt-cache policy containerd
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
ae@worker-1:~$ apt-cache policy runc
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

```
ae@worker-2:~$ sudo apt update
[sudo] password for ae: 
Hit:1 http://ru.archive.ubuntu.com/ubuntu noble InRelease
Get:2 http://ru.archive.ubuntu.com/ubuntu noble-updates InRelease [126 kB]
Get:3 http://ru.archive.ubuntu.com/ubuntu noble-backports InRelease [126 kB]
Get:4 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]              
Get:5 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Packages [2,016 kB]
Get:6 http://ru.archive.ubuntu.com/ubuntu noble-updates/main Translation-en [356 kB]
Get:7 http://ru.archive.ubuntu.com/ubuntu noble-updates/main amd64 Components [177 kB]
Get:8 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted amd64 Packages [3,210 kB]
Get:9 http://security.ubuntu.com/ubuntu noble-security/main amd64 Packages [1,704 kB]
Get:10 http://ru.archive.ubuntu.com/ubuntu noble-updates/restricted Translation-en [742 kB]         
Get:11 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Packages [1,693 kB]              
Get:12 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe Translation-en [330 kB]        
Get:13 http://ru.archive.ubuntu.com/ubuntu noble-updates/universe amd64 Components [386 kB]      
Get:14 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse Translation-en [11.1 kB]   
Get:15 http://ru.archive.ubuntu.com/ubuntu noble-updates/multiverse amd64 Components [940 B]        
Get:16 http://ru.archive.ubuntu.com/ubuntu noble-backports/main amd64 Components [5,760 B]
Get:17 http://ru.archive.ubuntu.com/ubuntu noble-backports/universe amd64 Components [10.5 kB]
Get:18 http://security.ubuntu.com/ubuntu noble-security/main Translation-en [268 kB]
Get:19 http://security.ubuntu.com/ubuntu noble-security/main amd64 Components [42.5 kB]
Get:20 http://security.ubuntu.com/ubuntu noble-security/restricted amd64 Packages [3,006 kB]
Get:21 http://security.ubuntu.com/ubuntu noble-security/restricted Translation-en [698 kB]
Get:22 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Packages [1,192 kB]
Get:23 http://security.ubuntu.com/ubuntu noble-security/universe Translation-en [230 kB]
Get:24 http://security.ubuntu.com/ubuntu noble-security/universe amd64 Components [74.2 kB]
Get:25 http://security.ubuntu.com/ubuntu noble-security/multiverse Translation-en [9,000 B]
Fetched 16.5 MB in 3s (4,920 kB/s)                                
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
43 packages can be upgraded. Run 'apt list --upgradable' to see them.
ae@worker-2:~$ apt-cache policy containerd
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
ae@worker-2:~$ apt-cache policy runc
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

## Установка containerd и runc

```
ae@control-plane-1:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
ae@control-plane-1:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
ae@control-plane-1:~$ systemctl is-enabled containerd
enabled
ae@control-plane-1:~$ systemctl is-active containerd
active
```

```
ae@worker-1:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
ae@worker-1:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
ae@worker-1:~$ systemctl is-enabled containerd
enabled
ae@worker-1:~$ systemctl is-active containerd
active
```

```
ae@worker-2:~$ containerd --version
containerd github.com/containerd/containerd/v2 2.2.1 
ae@worker-2:~$ runc --version
runc version 1.3.4-0ubuntu1~24.04.1
spec: 1.2.1
go: go1.24.4
libseccomp: 2.5.5
ae@worker-2:~$ systemctl is-enabled containerd
enabled
ae@worker-2:~$ systemctl is-active containerd
active
```

## Проверка default config containerd

```
ae@control-plane-1:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
ae@control-plane-1:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]
```

```
ae@worker-1:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
ae@worker-1:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]

```

```
ae@worker-2:~$ sudo test -f /etc/containerd/config.toml && sudo sed -n '1,220p' /etc/containerd/config.toml || echo "no /etc/containerd/config.toml"
no /etc/containerd/config.toml
ae@worker-2:~$ containerd config default | sed -n '1,220p' | grep -nE 'version|SystemdCgroup|io.containerd.cri|runc|sandbox_image'
1:version = 3
39:  [plugins.'io.containerd.cri.v1.images']
50:    [plugins.'io.containerd.cri.v1.images'.pinned_images]
53:    [plugins.'io.containerd.cri.v1.images'.registry]
56:    [plugins.'io.containerd.cri.v1.images'.image_decryption]
59:  [plugins.'io.containerd.cri.v1.runtime']
79:    [plugins.'io.containerd.cri.v1.runtime'.containerd]
80:      default_runtime_name = 'runc'
84:      [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes]
85:        [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc]
86:          runtime_type = 'io.containerd.runc.v2'
100:          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
109:            SystemdCgroup = false
111:    [plugins.'io.containerd.cri.v1.runtime'.cni]
```

## Настройка SystemdCgroup

```
ae@control-plane-1:/etc$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
ae@control-plane-1:/etc$ systemctl is-active containerd
active
ae@control-plane-1:/etc$ sudo journalctl -u containerd --no-pager -n 20
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597568433Z" level=info msg="skip loading plugin" error="skip plugin: tracing endpoint not configured" id=io.containerd.internal.v1.tracing type=io.containerd.internal.v1
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597581335Z" level=info msg="loading plugin" id=io.containerd.ttrpc.v1.otelttrpc type=io.containerd.ttrpc.v1
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597601012Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.healthcheck type=io.containerd.grpc.v1
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597617086Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.cri type=io.containerd.grpc.v1
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597673936Z" level=info msg="Connect containerd service"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.597715444Z" level=info msg="using experimental NRI integration - disable nri plugin to prevent this"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.598219316Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609520454Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609615448Z" level=info msg=serving... address=/run/containerd/containerd.sock
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609648723Z" level=info msg="Start subscribing containerd event"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609680351Z" level=info msg="Start recovering state"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609781135Z" level=info msg="Start event monitor"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609796705Z" level=info msg="Start cni network conf syncer for default"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609807689Z" level=info msg="Start streaming server"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609818767Z" level=info msg="Registered namespace \"k8s.io\" with NRI"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609828570Z" level=info msg="runtime interface starting up..."
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609837363Z" level=info msg="starting plugins..."
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609851145Z" level=info msg="Synchronizing NRI (plugin) with current runtime state"
May 28 19:41:21 control-plane-1 containerd[1953]: time="2026-05-28T19:41:21.609988667Z" level=info msg="containerd successfully booted in 0.032505s"
May 28 19:41:21 control-plane-1 systemd[1]: Started containerd.service - containerd container runtime.
```

```
ae@worker-1:~$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
ae@worker-1:~$ systemctl is-active containerd
active
ae@worker-1:~$ sudo journalctl -u containerd --no-pager -n 20
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717018625Z" level=info msg="skip loading plugin" error="skip plugin: tracing endpoint not configured" id=io.containerd.internal.v1.tracing type=io.containerd.internal.v1
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717031217Z" level=info msg="loading plugin" id=io.containerd.ttrpc.v1.otelttrpc type=io.containerd.ttrpc.v1
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717044735Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.healthcheck type=io.containerd.grpc.v1
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717059368Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.cri type=io.containerd.grpc.v1
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717100099Z" level=info msg="Connect containerd service"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717156138Z" level=info msg="using experimental NRI integration - disable nri plugin to prevent this"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.717666706Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727685327Z" level=info msg="Start subscribing containerd event"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727767287Z" level=info msg="Start recovering state"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727891991Z" level=info msg="Start event monitor"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727911351Z" level=info msg="Start cni network conf syncer for default"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727922844Z" level=info msg="Start streaming server"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727947970Z" level=info msg="Registered namespace \"k8s.io\" with NRI"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727918425Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.728030303Z" level=info msg=serving... address=/run/containerd/containerd.sock
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.727965527Z" level=info msg="runtime interface starting up..."
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.728077870Z" level=info msg="starting plugins..."
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.728094456Z" level=info msg="Synchronizing NRI (plugin) with current runtime state"
May 28 19:44:24 worker-1 containerd[2579]: time="2026-05-28T19:44:24.728228776Z" level=info msg="containerd successfully booted in 0.030302s"
May 28 19:44:24 worker-1 systemd[1]: Started containerd.service - containerd container runtime.
```

```
ae@worker-2:~$ grep -nE 'version|SystemdCgroup|default_runtime_name|runtime_type' /etc/containerd/config.toml
1:version = 3
80:      default_runtime_name = 'runc'
86:          runtime_type = 'io.containerd.runc.v2'
109:            SystemdCgroup = true
ae@worker-2:~$ systemctl is-active containerd
active
ae@worker-2:~$ sudo journalctl -u containerd --no-pager -n 20
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672004917Z" level=info msg="skip loading plugin" error="skip plugin: tracing endpoint not configured" id=io.containerd.internal.v1.tracing type=io.containerd.internal.v1
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672019549Z" level=info msg="loading plugin" id=io.containerd.ttrpc.v1.otelttrpc type=io.containerd.ttrpc.v1
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672040367Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.healthcheck type=io.containerd.grpc.v1
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672071111Z" level=info msg="loading plugin" id=io.containerd.grpc.v1.cri type=io.containerd.grpc.v1
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672113651Z" level=info msg="Connect containerd service"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672157934Z" level=info msg="using experimental NRI integration - disable nri plugin to prevent this"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672621529Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683677890Z" level=info msg="Start subscribing containerd event"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683730847Z" level=info msg="Start recovering state"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683855940Z" level=info msg="Start event monitor"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683873550Z" level=info msg="Start cni network conf syncer for default"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683886117Z" level=info msg="Start streaming server"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683899693Z" level=info msg="Registered namespace \"k8s.io\" with NRI"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683910135Z" level=info msg="runtime interface starting up..."
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683918734Z" level=info msg="starting plugins..."
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.683933194Z" level=info msg="Synchronizing NRI (plugin) with current runtime state"
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.684466200Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.684547012Z" level=info msg=serving... address=/run/containerd/containerd.sock
May 28 19:46:45 worker-2 containerd[2597]: time="2026-05-28T19:46:45.684856655Z" level=info msg="containerd successfully booted in 0.031638s"
May 28 19:46:45 worker-2 systemd[1]: Started containerd.service - containerd container runtime.
```

## Ожидаемый CNI warning
Проверил это поведение.
```
ae@worker-2:~$ cat /var/log/syslog  | grep "cni config"
2026-05-28T19:20:46.242673+00:00 worker-2 containerd[1768]: time="2026-05-28T19:20:46.242622308Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
2026-05-28T19:46:45.673358+00:00 worker-2 containerd[2597]: time="2026-05-28T19:46:45.672621529Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
2026-05-28T19:52:35.726758+00:00 worker-2 containerd[711]: time="2026-05-28T19:52:35.726602246Z" level=error msg="failed to load cni during init, please check CRI plugin status before setting up network for pods" error="cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"
```

## Проверка containerd через ctr

```
ae@control-plane-1:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip      
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip      
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip      
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip      
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip      
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok  
```

```
ae@worker-1:~$ sudo ctr namespaces list
NAME LABELS 
ae@worker-1:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip      
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip      
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip      
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip      
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip      
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok    
```

```
ae@worker-2:~$ sudo ctr namespaces list
NAME LABELS 
ae@worker-2:~$ sudo ctr plugins list | grep -E 'cri|runtime|snapshotter'
io.containerd.snapshotter.v1              blockfile                linux/amd64    skip      
io.containerd.snapshotter.v1              btrfs                    linux/amd64    skip      
io.containerd.snapshotter.v1              devmapper                linux/amd64    skip      
io.containerd.snapshotter.v1              erofs                    linux/amd64    skip      
io.containerd.snapshotter.v1              native                   linux/amd64    ok        
io.containerd.snapshotter.v1              overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1              zfs                      linux/amd64    skip      
io.containerd.runtime.v2                  task                     linux/amd64    ok        
io.containerd.cri.v1                      images                   -              ok        
io.containerd.cri.v1                      runtime                  linux/amd64    ok        
io.containerd.grpc.v1                     cri                      -              ok       
```

## Установка и настройка crictl

### Проверка доступности release archive
```
ae@control-plane-1:~$ CRICTL_VERSION="v1.34.0"

curl -I --connect-timeout 10 -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
HTTP/2 302 
date: Thu, 28 May 2026 20:09:55 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
location: https://release-assets.githubusercontent.com/github-production-release-asset/80172100/5b5868c2-89df-4f43-acb9-fb3b7d977087?sp=r&sv=2018-11-09&sr=b&spr=https&se=2026-05-28T21%3A10%3A36Z&rscd=attachment%3B+filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&rsct=application%2Foctet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2026-05-28T20%3A09%3A55Z&ske=2026-05-28T21%3A10%3A36Z&sks=b&skv=2018-11-09&sig=mX%2Bf2gJ8TRy4eBSW7G2uD1fjMsh%2F6fnoNxzOwDmp%2FZI%3D&jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc4MDAwMDc5NSwibmJmIjoxNzc5OTk4OTk1LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.Ru-QNfivyw1zV9xnP_idva6M_Oi04dQTKaf8Z1dYwNs&response-content-disposition=attachment%3B%20filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
cache-control: no-cache
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: no-referrer-when-downgrade
content-security-policy: default-src 'none'; base-uri 'self'; child-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/; connect-src 'self' uploads.github.com www.githubstatus.com collector.github.com raw.githubusercontent.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com *.rel.tunnels.api.visualstudio.com wss://*.rel.tunnels.api.visualstudio.com github.githubassets.com objects-origin.githubusercontent.com copilot-proxy.githubusercontent.com proxy.individual.githubcopilot.com proxy.business.githubcopilot.com proxy.enterprise.githubcopilot.com *.actions.githubusercontent.com wss://*.actions.githubusercontent.com productionresultssa0.blob.core.windows.net productionresultssa1.blob.core.windows.net productionresultssa2.blob.core.windows.net productionresultssa3.blob.core.windows.net productionresultssa4.blob.core.windows.net productionresultssa5.blob.core.windows.net productionresultssa6.blob.core.windows.net productionresultssa7.blob.core.windows.net productionresultssa8.blob.core.windows.net productionresultssa9.blob.core.windows.net productionresultssa10.blob.core.windows.net productionresultssa11.blob.core.windows.net productionresultssa12.blob.core.windows.net productionresultssa13.blob.core.windows.net productionresultssa14.blob.core.windows.net productionresultssa15.blob.core.windows.net productionresultssa16.blob.core.windows.net productionresultssa17.blob.core.windows.net productionresultssa18.blob.core.windows.net productionresultssa19.blob.core.windows.net github-production-repository-image-32fea6.s3.amazonaws.com github-production-release-asset-2e65be.s3.amazonaws.com insights.github.com wss://alive.github.com wss://alive-staging.github.com api.githubcopilot.com api.individual.githubcopilot.com api.business.githubcopilot.com api.enterprise.githubcopilot.com; font-src github.githubassets.com; form-action 'self' github.com gist.github.com copilot-workspace.githubnext.com objects-origin.githubusercontent.com; frame-ancestors 'none'; frame-src viewscreen.githubusercontent.com notebooks.githubusercontent.com; img-src 'self' data: blob: github.githubassets.com media.githubusercontent.com camo.githubusercontent.com identicons.github.com avatars.githubusercontent.com private-avatars.githubusercontent.com github-cloud.s3.amazonaws.com objects.githubusercontent.com release-assets.githubusercontent.com secured-user-images.githubusercontent.com user-images.githubusercontent.com private-user-images.githubusercontent.com opengraph.githubassets.com marketplace-screenshots.githubusercontent.com copilotprodattachments.blob.core.windows.net/github-production-copilot-attachments/ github-production-user-asset-6210df.s3.amazonaws.com customer-stories-feed.github.com spotlights-feed.github.com explore-feed.github.com objects-origin.githubusercontent.com *.githubusercontent.com; manifest-src 'self'; media-src github.com user-images.githubusercontent.com secured-user-images.githubusercontent.com private-user-images.githubusercontent.com github-production-user-asset-6210df.s3.amazonaws.com gist.github.com github.githubassets.com; script-src github.githubassets.com; style-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
content-length: 0
x-github-request-id: 7255:3392D1:4A45819:3870A4A:6A18A112

HTTP/2 200 
last-modified: Thu, 21 Aug 2025 07:58:34 GMT
etag: "0x8DDE088875498F0"
server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 6af94ae5-701e-0020-16c2-3d9af2000000
x-ms-version: 2018-11-09
x-ms-creation-time: Thu, 21 Aug 2025 07:58:34 GMT
x-ms-lease-status: unlocked
x-ms-lease-state: available
x-ms-blob-type: BlockBlob
x-ms-server-encrypted: true
via: 1.1 varnish, 1.1 varnish
accept-ranges: bytes
age: 4131
date: Thu, 28 May 2026 20:09:55 GMT
x-served-by: cache-iad-kjyo7100161-IAD, cache-bma-essb1270067-BMA
x-cache: HIT, HIT
x-cache-hits: 12246, 0
x-timer: S1779998996.506895,VS0,VE1
content-disposition: attachment; filename=crictl-v1.34.0-linux-amd64.tar.gz
content-type: application/octet-stream
content-length: 19610258
```

```
ae@worker-1:~$ CRICTL_VERSION="v1.34.0"

curl -I --connect-timeout 10 -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
HTTP/2 302 
date: Thu, 28 May 2026 20:09:55 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
location: https://release-assets.githubusercontent.com/github-production-release-asset/80172100/5b5868c2-89df-4f43-acb9-fb3b7d977087?sp=r&sv=2018-11-09&sr=b&spr=https&se=2026-05-28T21%3A10%3A36Z&rscd=attachment%3B+filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&rsct=application%2Foctet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2026-05-28T20%3A09%3A55Z&ske=2026-05-28T21%3A10%3A36Z&sks=b&skv=2018-11-09&sig=mX%2Bf2gJ8TRy4eBSW7G2uD1fjMsh%2F6fnoNxzOwDmp%2FZI%3D&jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc4MDAwMDc5NSwibmJmIjoxNzc5OTk4OTk1LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.Ru-QNfivyw1zV9xnP_idva6M_Oi04dQTKaf8Z1dYwNs&response-content-disposition=attachment%3B%20filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
cache-control: no-cache
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: no-referrer-when-downgrade
content-security-policy: default-src 'none'; base-uri 'self'; child-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/; connect-src 'self' uploads.github.com www.githubstatus.com collector.github.com raw.githubusercontent.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com *.rel.tunnels.api.visualstudio.com wss://*.rel.tunnels.api.visualstudio.com github.githubassets.com objects-origin.githubusercontent.com copilot-proxy.githubusercontent.com proxy.individual.githubcopilot.com proxy.business.githubcopilot.com proxy.enterprise.githubcopilot.com *.actions.githubusercontent.com wss://*.actions.githubusercontent.com productionresultssa0.blob.core.windows.net productionresultssa1.blob.core.windows.net productionresultssa2.blob.core.windows.net productionresultssa3.blob.core.windows.net productionresultssa4.blob.core.windows.net productionresultssa5.blob.core.windows.net productionresultssa6.blob.core.windows.net productionresultssa7.blob.core.windows.net productionresultssa8.blob.core.windows.net productionresultssa9.blob.core.windows.net productionresultssa10.blob.core.windows.net productionresultssa11.blob.core.windows.net productionresultssa12.blob.core.windows.net productionresultssa13.blob.core.windows.net productionresultssa14.blob.core.windows.net productionresultssa15.blob.core.windows.net productionresultssa16.blob.core.windows.net productionresultssa17.blob.core.windows.net productionresultssa18.blob.core.windows.net productionresultssa19.blob.core.windows.net github-production-repository-image-32fea6.s3.amazonaws.com github-production-release-asset-2e65be.s3.amazonaws.com insights.github.com wss://alive.github.com wss://alive-staging.github.com api.githubcopilot.com api.individual.githubcopilot.com api.business.githubcopilot.com api.enterprise.githubcopilot.com; font-src github.githubassets.com; form-action 'self' github.com gist.github.com copilot-workspace.githubnext.com objects-origin.githubusercontent.com; frame-ancestors 'none'; frame-src viewscreen.githubusercontent.com notebooks.githubusercontent.com; img-src 'self' data: blob: github.githubassets.com media.githubusercontent.com camo.githubusercontent.com identicons.github.com avatars.githubusercontent.com private-avatars.githubusercontent.com github-cloud.s3.amazonaws.com objects.githubusercontent.com release-assets.githubusercontent.com secured-user-images.githubusercontent.com user-images.githubusercontent.com private-user-images.githubusercontent.com opengraph.githubassets.com marketplace-screenshots.githubusercontent.com copilotprodattachments.blob.core.windows.net/github-production-copilot-attachments/ github-production-user-asset-6210df.s3.amazonaws.com customer-stories-feed.github.com spotlights-feed.github.com explore-feed.github.com objects-origin.githubusercontent.com *.githubusercontent.com; manifest-src 'self'; media-src github.com user-images.githubusercontent.com secured-user-images.githubusercontent.com private-user-images.githubusercontent.com github-production-user-asset-6210df.s3.amazonaws.com gist.github.com github.githubassets.com; script-src github.githubassets.com; style-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
content-length: 0
x-github-request-id: 695A:3A47C7:4C7C7E8:3A228D0:6A18A174

HTTP/2 200 
last-modified: Thu, 21 Aug 2025 07:58:34 GMT
etag: "0x8DDE088875498F0"
server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 6af94ae5-701e-0020-16c2-3d9af2000000
x-ms-version: 2018-11-09
x-ms-creation-time: Thu, 21 Aug 2025 07:58:34 GMT
x-ms-lease-status: unlocked
x-ms-lease-state: available
x-ms-blob-type: BlockBlob
x-ms-server-encrypted: true
via: 1.1 varnish, 1.1 varnish
accept-ranges: bytes
age: 4228
date: Thu, 28 May 2026 20:11:32 GMT
x-served-by: cache-iad-kjyo7100161-IAD, cache-bma-essb1270043-BMA
x-cache: HIT, HIT
x-cache-hits: 12246, 0
x-timer: S1779999093.608321,VS0,VE1
content-disposition: attachment; filename=crictl-v1.34.0-linux-amd64.tar.gz
content-type: application/octet-stream
content-length: 19610258

```

```
ae@worker-2:~$ CRICTL_VERSION="v1.34.0"

curl -I --connect-timeout 10 -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
HTTP/2 302 
date: Thu, 28 May 2026 20:12:26 GMT
content-type: text/html; charset=utf-8
vary: X-PJAX, X-PJAX-Container, Turbo-Visit, Turbo-Frame, X-Requested-With, Sec-Fetch-Site,Accept-Encoding, Accept, X-Requested-With
location: https://release-assets.githubusercontent.com/github-production-release-asset/80172100/5b5868c2-89df-4f43-acb9-fb3b7d977087?sp=r&sv=2018-11-09&sr=b&spr=https&se=2026-05-28T21%3A10%3A57Z&rscd=attachment%3B+filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&rsct=application%2Foctet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2026-05-28T20%3A10%3A03Z&ske=2026-05-28T21%3A10%3A57Z&sks=b&skv=2018-11-09&sig=x8hGhSEiiyGf1dO1IOdV6AzoplebvMrxsw4VUiPC05U%3D&jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc4MDAwMDk0NiwibmJmIjoxNzc5OTk5MTQ2LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.VwrV1NWY5SrdNs519swkWPRub4vTdl5Ek0YdX1yLw9M&response-content-disposition=attachment%3B%20filename%3Dcrictl-v1.34.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
cache-control: no-cache
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
x-xss-protection: 0
referrer-policy: no-referrer-when-downgrade
content-security-policy: default-src 'none'; base-uri 'self'; child-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/; connect-src 'self' uploads.github.com www.githubstatus.com collector.github.com raw.githubusercontent.com api.github.com github-cloud.s3.amazonaws.com github-production-repository-file-5c1aeb.s3.amazonaws.com github-production-upload-manifest-file-7fdce7.s3.amazonaws.com github-production-user-asset-6210df.s3.amazonaws.com *.rel.tunnels.api.visualstudio.com wss://*.rel.tunnels.api.visualstudio.com github.githubassets.com objects-origin.githubusercontent.com copilot-proxy.githubusercontent.com proxy.individual.githubcopilot.com proxy.business.githubcopilot.com proxy.enterprise.githubcopilot.com *.actions.githubusercontent.com wss://*.actions.githubusercontent.com productionresultssa0.blob.core.windows.net productionresultssa1.blob.core.windows.net productionresultssa2.blob.core.windows.net productionresultssa3.blob.core.windows.net productionresultssa4.blob.core.windows.net productionresultssa5.blob.core.windows.net productionresultssa6.blob.core.windows.net productionresultssa7.blob.core.windows.net productionresultssa8.blob.core.windows.net productionresultssa9.blob.core.windows.net productionresultssa10.blob.core.windows.net productionresultssa11.blob.core.windows.net productionresultssa12.blob.core.windows.net productionresultssa13.blob.core.windows.net productionresultssa14.blob.core.windows.net productionresultssa15.blob.core.windows.net productionresultssa16.blob.core.windows.net productionresultssa17.blob.core.windows.net productionresultssa18.blob.core.windows.net productionresultssa19.blob.core.windows.net github-production-repository-image-32fea6.s3.amazonaws.com github-production-release-asset-2e65be.s3.amazonaws.com insights.github.com wss://alive.github.com wss://alive-staging.github.com api.githubcopilot.com api.individual.githubcopilot.com api.business.githubcopilot.com api.enterprise.githubcopilot.com; font-src github.githubassets.com; form-action 'self' github.com gist.github.com copilot-workspace.githubnext.com objects-origin.githubusercontent.com; frame-ancestors 'none'; frame-src viewscreen.githubusercontent.com notebooks.githubusercontent.com; img-src 'self' data: blob: github.githubassets.com media.githubusercontent.com camo.githubusercontent.com identicons.github.com avatars.githubusercontent.com private-avatars.githubusercontent.com github-cloud.s3.amazonaws.com objects.githubusercontent.com release-assets.githubusercontent.com secured-user-images.githubusercontent.com user-images.githubusercontent.com private-user-images.githubusercontent.com opengraph.githubassets.com marketplace-screenshots.githubusercontent.com copilotprodattachments.blob.core.windows.net/github-production-copilot-attachments/ github-production-user-asset-6210df.s3.amazonaws.com customer-stories-feed.github.com spotlights-feed.github.com explore-feed.github.com objects-origin.githubusercontent.com *.githubusercontent.com; manifest-src 'self'; media-src github.com user-images.githubusercontent.com secured-user-images.githubusercontent.com private-user-images.githubusercontent.com github-production-user-asset-6210df.s3.amazonaws.com gist.github.com github.githubassets.com; script-src github.githubassets.com; style-src 'unsafe-inline' github.githubassets.com; upgrade-insecure-requests; worker-src github.githubassets.com github.com/assets-cdn/worker/ github.com/assets/ gist.github.com/assets-cdn/worker/
server: github.com
content-length: 0
x-github-request-id: 1195:344900:48E0D79:376402A:6A18A1AA

HTTP/2 200 
last-modified: Thu, 21 Aug 2025 07:58:34 GMT
etag: "0x8DDE088875498F0"
server: Windows-Azure-Blob/1.0 Microsoft-HTTPAPI/2.0
x-ms-request-id: 6af94ae5-701e-0020-16c2-3d9af2000000
x-ms-version: 2018-11-09
x-ms-creation-time: Thu, 21 Aug 2025 07:58:34 GMT
x-ms-lease-status: unlocked
x-ms-lease-state: available
x-ms-blob-type: BlockBlob
x-ms-server-encrypted: true
via: 1.1 varnish, 1.1 varnish
accept-ranges: bytes
date: Thu, 28 May 2026 20:12:26 GMT
age: 4282
x-served-by: cache-iad-kjyo7100161-IAD, cache-bma-essb1270033-BMA
x-cache: HIT, HIT
x-cache-hits: 12246, 1
x-timer: S1779999146.491126,VS0,VE71
content-disposition: attachment; filename=crictl-v1.34.0-linux-amd64.tar.gz
content-type: application/octet-stream
content-length: 19610258

```

### Скачивание и проверка архива

```
ae@control-plane-1:~$ CRICTL_VERSION="v1.34.0"
TMP_DIR="$(mktemp -d)"

curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

ls -lh "${TMP_DIR}"
tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 18.7M  100 18.7M    0     0  9541k      0  0:00:02  0:00:02 --:--:-- 10.9M
total 19M
-rw-rw-r-- 1 ae ae 19M May 28 20:14 crictl-v1.34.0-linux-amd64.tar.gz
crictl
```

```
ae@worker-1:~$ CRICTL_VERSION="v1.34.0"
TMP_DIR="$(mktemp -d)"

curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

ls -lh "${TMP_DIR}"
tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 18.7M  100 18.7M    0     0  8650k      0  0:00:02  0:00:02 --:--:-- 10.9M
total 19M
-rw-rw-r-- 1 ae ae 19M May 28 20:22 crictl-v1.34.0-linux-amd64.tar.gz
crictl

```

```
ae@worker-2:~$ CRICTL_VERSION="v1.34.0"
TMP_DIR="$(mktemp -d)"

curl -L \
  "https://github.com/kubernetes-sigs/cri-tools/releases/download/${CRICTL_VERSION}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz" \
  -o "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"

ls -lh "${TMP_DIR}"
tar -tzf "${TMP_DIR}/crictl-${CRICTL_VERSION}-linux-amd64.tar.gz"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 18.7M  100 18.7M    0     0  9496k      0  0:00:02  0:00:02 --:--:-- 10.9M
total 19M
-rw-rw-r-- 1 ae ae 19M May 28 20:22 crictl-v1.34.0-linux-amd64.tar.gz
crictl
```

### Установка crictl

```
ae@control-plane-1:~$ command -v crictl
crictl --version
ls -l /usr/local/bin/crictl
/usr/local/bin/crictl
crictl version v1.34.0
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl
```

```
ae@worker-1:~$ command -v crictl
crictl --version
ls -l /usr/local/bin/crictl
/usr/local/bin/crictl
crictl version v1.34.0
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl
```

```
ae@worker-2:~$ command -v crictl
crictl --version
ls -l /usr/local/bin/crictl
/usr/local/bin/crictl
crictl version v1.34.0
-rwxr-xr-x 1 root root 40548006 Aug 21  2025 /usr/local/bin/crictl
```

### Настройка /etc/crictl.yaml
```
ae@control-plane-1:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
ae@control-plane-1:~$ sudo crictl info
{
  "cniconfig": {
    "Networks": [
      {
        "Config": {
          "CNIVersion": "0.3.1",
          "Name": "cni-loopback",
          "Plugins": [
            {
              "Network": {
                "ipam": {},
                "type": "loopback"
              },
              "Source": "{\"type\":\"loopback\"}"
            }
          ],
          "Source": "{\n\"cniVersion\": \"0.3.1\",\n\"name\": \"cni-loopback\",\n\"plugins\": [{\n  \"type\": \"loopback\"\n}]\n}"
        },
        "IFName": "lo"
      }
    ],
    "PluginConfDir": "/etc/cni/net.d",
    "PluginDirs": [
      "/opt/cni/bin"
    ],
    "PluginMaxConfNum": 1,
    "Prefix": "eth"
  },
  "config": {
    "cdiSpecDirs": [
      "/etc/cdi",
      "/var/run/cdi"
    ],
    "cni": {
      "binDir": "",
      "binDirs": [
        "/opt/cni/bin"
      ],
      "confDir": "/etc/cni/net.d",
      "confTemplate": "",
      "ipPref": "",
      "maxConfNum": 1,
      "setupSerially": false,
      "useInternalLoopback": false
    },
    "containerd": {
      "defaultRuntimeName": "runc",
      "ignoreBlockIONotEnabledErrors": false,
      "ignoreRdtNotEnabledErrors": false,
      "runtimes": {
        "runc": {
          "ContainerAnnotations": [],
          "PodAnnotations": [],
          "baseRuntimeSpec": "",
          "cgroupWritable": false,
          "cniConfDir": "",
          "cniMaxConfNum": 0,
          "io_type": "",
          "options": {
            "BinaryName": "",
            "CriuImagePath": "",
            "CriuWorkPath": "",
            "IoGid": 0,
            "IoUid": 0,
            "NoNewKeyring": false,
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
          },
          "privileged_without_host_devices": false,
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
          "sandboxer": "podsandbox",
          "snapshotter": ""
        }
      }
    },
    "containerdEndpoint": "/run/containerd/containerd.sock",
    "containerdRootDir": "/var/lib/containerd",
    "device_ownership_from_security_context": false,
    "disableApparmor": false,
    "disableHugetlbController": true,
    "disableProcMount": false,
    "drainExecSyncIOTimeout": "0s",
    "enableCDI": true,
    "enableSelinux": false,
    "enableUnprivilegedICMP": true,
    "enableUnprivilegedPorts": true,
    "ignoreDeprecationWarnings": [],
    "ignoreImageDefinedVolumes": false,
    "maxContainerLogLineSize": 16384,
    "netnsMountsUnderStateDir": false,
    "restrictOOMScoreAdj": false,
    "rootDir": "/var/lib/containerd/io.containerd.grpc.v1.cri",
    "selinuxCategoryRange": 1024,
    "stateDir": "/run/containerd/io.containerd.grpc.v1.cri",
    "tolerateMissingHugetlbController": true,
    "unsetSeccompProfile": ""
  },
  "features": {
    "supplemental_groups_policy": true
  },
  "golang": "go1.24.4",
  "lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "lastCNILoadStatus.default": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "runtimeHandlers": [
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      }
    },
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      },
      "name": "runc"
    }
  ],
  "status": {
    "conditions": [
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
      },
      {
        "message": "Network plugin returns error: cni plugin not initialized",
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
      },
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "ContainerdHasNoDeprecationWarnings"
      }
    ]
  }
}
```

```
ae@worker-1:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
ae@worker-1:~$ sudo crictl info
{
  "cniconfig": {
    "Networks": [
      {
        "Config": {
          "CNIVersion": "0.3.1",
          "Name": "cni-loopback",
          "Plugins": [
            {
              "Network": {
                "ipam": {},
                "type": "loopback"
              },
              "Source": "{\"type\":\"loopback\"}"
            }
          ],
          "Source": "{\n\"cniVersion\": \"0.3.1\",\n\"name\": \"cni-loopback\",\n\"plugins\": [{\n  \"type\": \"loopback\"\n}]\n}"
        },
        "IFName": "lo"
      }
    ],
    "PluginConfDir": "/etc/cni/net.d",
    "PluginDirs": [
      "/opt/cni/bin"
    ],
    "PluginMaxConfNum": 1,
    "Prefix": "eth"
  },
  "config": {
    "cdiSpecDirs": [
      "/etc/cdi",
      "/var/run/cdi"
    ],
    "cni": {
      "binDir": "",
      "binDirs": [
        "/opt/cni/bin"
      ],
      "confDir": "/etc/cni/net.d",
      "confTemplate": "",
      "ipPref": "",
      "maxConfNum": 1,
      "setupSerially": false,
      "useInternalLoopback": false
    },
    "containerd": {
      "defaultRuntimeName": "runc",
      "ignoreBlockIONotEnabledErrors": false,
      "ignoreRdtNotEnabledErrors": false,
      "runtimes": {
        "runc": {
          "ContainerAnnotations": [],
          "PodAnnotations": [],
          "baseRuntimeSpec": "",
          "cgroupWritable": false,
          "cniConfDir": "",
          "cniMaxConfNum": 0,
          "io_type": "",
          "options": {
            "BinaryName": "",
            "CriuImagePath": "",
            "CriuWorkPath": "",
            "IoGid": 0,
            "IoUid": 0,
            "NoNewKeyring": false,
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
          },
          "privileged_without_host_devices": false,
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
          "sandboxer": "podsandbox",
          "snapshotter": ""
        }
      }
    },
    "containerdEndpoint": "/run/containerd/containerd.sock",
    "containerdRootDir": "/var/lib/containerd",
    "device_ownership_from_security_context": false,
    "disableApparmor": false,
    "disableHugetlbController": true,
    "disableProcMount": false,
    "drainExecSyncIOTimeout": "0s",
    "enableCDI": true,
    "enableSelinux": false,
    "enableUnprivilegedICMP": true,
    "enableUnprivilegedPorts": true,
    "ignoreDeprecationWarnings": [],
    "ignoreImageDefinedVolumes": false,
    "maxContainerLogLineSize": 16384,
    "netnsMountsUnderStateDir": false,
    "restrictOOMScoreAdj": false,
    "rootDir": "/var/lib/containerd/io.containerd.grpc.v1.cri",
    "selinuxCategoryRange": 1024,
    "stateDir": "/run/containerd/io.containerd.grpc.v1.cri",
    "tolerateMissingHugetlbController": true,
    "unsetSeccompProfile": ""
  },
  "features": {
    "supplemental_groups_policy": true
  },
  "golang": "go1.24.4",
  "lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "lastCNILoadStatus.default": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "runtimeHandlers": [
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      }
    },
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      },
      "name": "runc"
    }
  ],
  "status": {
    "conditions": [
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
      },
      {
        "message": "Network plugin returns error: cni plugin not initialized",
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
      },
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "ContainerdHasNoDeprecationWarnings"
      }
    ]
  }
}
```

```
ae@worker-2:~$ cat /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
ae@worker-2:~$ sudo crictl info
{
  "cniconfig": {
    "Networks": [
      {
        "Config": {
          "CNIVersion": "0.3.1",
          "Name": "cni-loopback",
          "Plugins": [
            {
              "Network": {
                "ipam": {},
                "type": "loopback"
              },
              "Source": "{\"type\":\"loopback\"}"
            }
          ],
          "Source": "{\n\"cniVersion\": \"0.3.1\",\n\"name\": \"cni-loopback\",\n\"plugins\": [{\n  \"type\": \"loopback\"\n}]\n}"
        },
        "IFName": "lo"
      }
    ],
    "PluginConfDir": "/etc/cni/net.d",
    "PluginDirs": [
      "/opt/cni/bin"
    ],
    "PluginMaxConfNum": 1,
    "Prefix": "eth"
  },
  "config": {
    "cdiSpecDirs": [
      "/etc/cdi",
      "/var/run/cdi"
    ],
    "cni": {
      "binDir": "",
      "binDirs": [
        "/opt/cni/bin"
      ],
      "confDir": "/etc/cni/net.d",
      "confTemplate": "",
      "ipPref": "",
      "maxConfNum": 1,
      "setupSerially": false,
      "useInternalLoopback": false
    },
    "containerd": {
      "defaultRuntimeName": "runc",
      "ignoreBlockIONotEnabledErrors": false,
      "ignoreRdtNotEnabledErrors": false,
      "runtimes": {
        "runc": {
          "ContainerAnnotations": [],
          "PodAnnotations": [],
          "baseRuntimeSpec": "",
          "cgroupWritable": false,
          "cniConfDir": "",
          "cniMaxConfNum": 0,
          "io_type": "",
          "options": {
            "BinaryName": "",
            "CriuImagePath": "",
            "CriuWorkPath": "",
            "IoGid": 0,
            "IoUid": 0,
            "NoNewKeyring": false,
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
          },
          "privileged_without_host_devices": false,
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
          "sandboxer": "podsandbox",
          "snapshotter": ""
        }
      }
    },
    "containerdEndpoint": "/run/containerd/containerd.sock",
    "containerdRootDir": "/var/lib/containerd",
    "device_ownership_from_security_context": false,
    "disableApparmor": false,
    "disableHugetlbController": true,
    "disableProcMount": false,
    "drainExecSyncIOTimeout": "0s",
    "enableCDI": true,
    "enableSelinux": false,
    "enableUnprivilegedICMP": true,
    "enableUnprivilegedPorts": true,
    "ignoreDeprecationWarnings": [],
    "ignoreImageDefinedVolumes": false,
    "maxContainerLogLineSize": 16384,
    "netnsMountsUnderStateDir": false,
    "restrictOOMScoreAdj": false,
    "rootDir": "/var/lib/containerd/io.containerd.grpc.v1.cri",
    "selinuxCategoryRange": 1024,
    "stateDir": "/run/containerd/io.containerd.grpc.v1.cri",
    "tolerateMissingHugetlbController": true,
    "unsetSeccompProfile": ""
  },
  "features": {
    "supplemental_groups_policy": true
  },
  "golang": "go1.24.4",
  "lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "lastCNILoadStatus.default": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config",
  "runtimeHandlers": [
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      }
    },
    {
      "features": {
        "recursive_read_only_mounts": true,
        "user_namespaces": true
      },
      "name": "runc"
    }
  ],
  "status": {
    "conditions": [
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
      },
      {
        "message": "Network plugin returns error: cni plugin not initialized",
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
      },
      {
        "message": "",
        "reason": "",
        "status": true,
        "type": "ContainerdHasNoDeprecationWarnings"
      }
    ]
  }
}
```

### Проверка CRI состояния

```
ae@control-plane-1:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B2            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
--
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
```

```
ae@worker-1:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B2
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
--
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
```

```
ae@worker-2:~$ sudo crictl info | grep -E 'runtimeType|SystemdCgroup|RuntimeReady|NetworkReady' -B2
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
--
          "privileged_without_host_devices_all_devices_allowed": false,
          "runtimePath": "",
          "runtimeType": "io.containerd.runc.v2",
--
        "reason": "",
        "status": true,
        "type": "RuntimeReady"
--
        "reason": "NetworkPluginNotReady",
        "status": false,
        "type": "NetworkReady"
```

### Проверка пустого runtime

```
ae@control-plane-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@control-plane-1:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@control-plane-1:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
ae@control-plane-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

```

```
ae@worker-1:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@worker-1:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@worker-1:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
ae@worker-1:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME

```

```
ae@worker-2:~$ sudo crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@worker-2:~$ sudo crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID              POD                 NAMESPACE
ae@worker-2:~$ sudo crictl images
IMAGE               TAG                 IMAGE ID            SIZE
ae@worker-2:~$ sudo crictl pods
POD ID              CREATED             STATE               NAME                NAMESPACE           ATTEMPT             RUNTIME
```

### Что стало понятнее

- Containerd отвечает за загрузку и хранение образов, управление жизненным циклом(создать, запустить, остановить, удалить), управление ресурсами(фс, сеть, диск), взаимодействие с системами через апи.
- Runc отвечает за создание и запуск процесса контейнера, работает напрямую с ядром линукса.  Является официальной реализацией OCI. 
  Для создания и запуска использует:
  1) namespaces для изоляции процессов, фс, сетей
  2) cgroups для управления ресурсами (ЦП, память, диск)
- Containerd вызывает Runc для запуска контейнера.

### Вопросы

- Почему мы используем именно этот рантайм для контейнеров? есть ли другие рантаймы? 


### Статус

done