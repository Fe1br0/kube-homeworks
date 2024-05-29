# Домашнее задание к занятию «Установка Kubernetes»

### Цель задания

Установить кластер K8s.

### Чеклист готовности к домашнему заданию

1. Развёрнутые ВМ с ОС Ubuntu 20.04-lts.


### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция по установке kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/).
2. [Документация kubespray](https://kubespray.io/).

-----

### Задание 1. Установить кластер k8s с 1 master node

1. Подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды.
2. В качестве CRI — containerd.
3. Запуск etcd производить на мастере.
4. Способ установки выбрать самостоятельно.

## Дополнительные задания (со звёздочкой)

**Настоятельно рекомендуем выполнять все задания под звёздочкой.** Их выполнение поможет глубже разобраться в материале.   
Задания под звёздочкой необязательные к выполнению и не повлияют на получение зачёта по этому домашнему заданию. 

------
### Задание 2*. Установить HA кластер

1. Установить кластер в режиме HA.
2. Использовать нечётное количество Master-node.
3. Для cluster ip использовать keepalived или другой способ.

### Правила приёма работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl get nodes`, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
------

### Решение

* Создал ВМ в яндекс облаке 




* Подключился к нашей мастер ноде



* Выполняем установку приложений kubeadm, kubelet, kubectl

```bash
apt-get install -y apt-transport-https ca-certificates curl
mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

* Включаем IP Forward

```bash
root@master-01:~# modprobe br_netfilter
root@master-01:~# modprobe overlay
root@master-01:~# echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-arptables=1" >> /etc/sysctl.conf
root@master-01:~# echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
root@master-01:~# sysctl -p /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
root@master-01:~# cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
> overlay
> br_netfilter
> EOF

overlay

br_netfilter

root@master-01:~#
```

* Ставим и настраиваем контейнеры с помощью `containerd`

```bash
apt-get install ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
root@master-01:~# apt-get update
Hit:1 http://mirror.yandex.ru/ubuntu focal InRelease
Hit:2 http://mirror.yandex.ru/ubuntu focal-updates InRelease
Hit:3 http://mirror.yandex.ru/ubuntu focal-backports InRelease
Hit:4 http://security.ubuntu.com/ubuntu focal-security InRelease
Get:5 https://download.docker.com/linux/ubuntu focal InRelease [57.7 kB]
Hit:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.30/deb  InRelease
Get:7 https://download.docker.com/linux/ubuntu focal/stable amd64 Packages [43.6 kB]
Fetched 101 kB in 1s (168 kB/s)
Reading package lists... Done

apt-get install containerd.io
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```

* Правим `config.toml`

![alt text](image-2.png)

* Перезапускаем сервис containerd и добавляем в автозагрузку сервис kubelet

```bash
root@master-01:~# systemctl restart containerd
root@master-01:~#
root@master-01:~# systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-05-29 18:53:18 UTC; 8s ago
       Docs: https://containerd.io
    Process: 16747 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 16761 (containerd)
      Tasks: 9
     Memory: 13.0M
     CGroup: /system.slice/containerd.service
             └─16761 /usr/bin/containerd

May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270297329Z" level=info msg="Start subscribing containerd event"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270353495Z" level=info msg="Start recovering state"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270424657Z" level=info msg="Start event monitor"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270443627Z" level=info msg="Start snapshots syncer"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270456164Z" level=info msg="Start cni network conf syncer for default"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.270477092Z" level=info msg="Start streaming server"
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.271027868Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.271085931Z" level=info msg=serving... address=/run/containerd/containerd.sock
May 29 18:53:18 master-01 systemd[1]: Started containerd container runtime.
May 29 18:53:18 master-01 containerd[16761]: time="2024-05-29T18:53:18.272406665Z" level=info msg="containerd successfully booted in 0.026711s"
root@master-01:~# systemctl enable kubelet
```

* Выполняем установку 

```bash
root@master-01:~# kubeadm config images pull
I0210 13:34:19.378640    6624 version.go:256] remote version is much newer: v1.29.1; falling back to: stable-1.28
[config/images] Pulled registry.k8s.io/kube-apiserver:v1.28.6
[config/images] Pulled registry.k8s.io/kube-controller-manager:v1.28.6
[config/images] Pulled registry.k8s.io/kube-scheduler:v1.28.6
[config/images] Pulled registry.k8s.io/kube-proxy:v1.28.6
[config/images] Pulled registry.k8s.io/pause:3.9
[config/images] Pulled registry.k8s.io/etcd:3.5.9-0
[config/images] Pulled registry.k8s.io/coredns/coredns:v1.10.1
root@master-01:~#
```
```bash
root@master-01:~# kubeadm init --apiserver-advertise-address=10.131.0.3 --pod-network-cidr 10.244.0.0/16  --apiserver-cert-extra-sans=158.160.173.193,master-01.ru-central1.internal
I0210 13:39:11.552988    6939 version.go:256] remote version is much newer: v1.29.1; falling back to: stable-1.28
[init] Using Kubernetes version: v1.28.6
[preflight] Running pre-flight checks
...
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 6.502601 seconds
...

kubeadm join 10.131.0.3:6443 --token l1uktz.duiyat0apfq71poh \
        --discovery-token-ca-cert-hash sha256:dbb57b15c20eab2a405eea82e5e506ad534b1b837d2f8b2b27f993ddd168374b
root@master-01:~#
```

* Создаём kubeconfig на мастер-ноде

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* Ставим сетевой плагин `flannel` на мастер-ноде

```bash 
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
namespace/kube-flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

* Приступаем конфигурировать воркер-ноды (ниже пример на worker-01)

```bash
apt-get install -y apt-transport-https ca-certificates curl
mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```
```bash
root@worker-01:~# modprobe br_netfilter
root@worker-01:~# modprobe overlay
root@worker-01:~# echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
root@worker-01:~# echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
root@worker-01:~# echo "net.bridge.bridge-nf-call-arptables=1" >> /etc/sysctl.conf
root@worker-01:~# echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
root@worker-01:~# sysctl -p /etc/sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
root@worker-01:~# cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
> overlay
> br_netfilter
> EOF

overlay
br_netfilter
root@worker-01:~#
```
```bash
apt-get install ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install containerd.io
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
```
```bash
root@worker-01:~# systemctl restart containerd
root@worker-01:~# systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-05-29 19:46:47 UTC; 6s ago
       Docs: https://containerd.io
    Process: 16941 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 16942 (containerd)
      Tasks: 9
     Memory: 13.6M
     CGroup: /system.slice/containerd.service
             └─16942 /usr/bin/containerd

May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.963663135Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.963709023Z" level=info msg=serving... address=/run/containerd/containerd.sock
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979265397Z" level=info msg="Start subscribing containerd event"
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979460900Z" level=info msg="Start recovering state"
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979644251Z" level=info msg="Start event monitor"
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979745927Z" level=info msg="Start snapshots syncer"
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979840936Z" level=info msg="Start cni network conf syncer for default"
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.979927549Z" level=info msg="Start streaming server"
May 29 19:46:47 worker-01 systemd[1]: Started containerd container runtime.
May 29 19:46:47 worker-01 containerd[16942]: time="2024-05-29T19:46:47.983201762Z" level=info msg="containerd successfully booted in 0.045133s"
root@worker-01:~# systemctl enable kubelet
```

* Подключаем воркер в кластер кубера

```bash
root@worker-01:~# kubeadm join 10.131.0.3:6443 --token l1uktz.duiyat0apfq71poh \
>       --discovery-token-ca-cert-hash sha256:dbb57b15c20eab2a405eea82e5e506ad534b1b837d2f8b2b27f993ddd168374b
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

root@worker-01:~#
```

* Проверяем удачно ли прошло добавление воркер ноды



* Аналогично настраиваем остальные хосты



