---
title: k8s的安装-docker
date: 2025-08-11 15:42:58
update: 2025-08-12 15:42:58
tags: [k8s]
index_img: /img/bg/k8s.jpg
excerpt: ubuntu22安装k8s的1.32版本(docker)
sticky: 100
category: 容器化
comments: true
comment: 'valine'
---
# <center>K8S安装-docker</center>

#### <div style="background-color:cadetblue">一、环境准备</div>
| 机器名       | ip |   系统版本   |   k8s版本   |
| :------------ | :------------: | ---------------: | ----------: |
| k8s-master   |  192.168.12.101   | ubuntu 24.04  | v1.32  |
| k8s-work01   |  192.168.12.101   | ubuntu 24.04  | v1.32  |
| k8s-work02   |  192.168.12.102   | ubuntu 24.04  | v1.32  |


#### <div style="background-color:cadetblue">二、机器配置</div>

<span style="background-color:red">关闭swap<span>
```shell
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab

# 关闭防火墙
sudo systemctl disable ufw
```
时间同步
```shell
timedatectl set-timezone Asia/Shanghai
apt install -y ntpsec-ntpdate
ntpdate ntp.aliyun.com
```
设置 hosts
```
sudo hostnamectl set-hostname k8s-master01
sudo hostnamectl set-hostname k8s-worker1
sudo hostnamectl set-hostname k8s-worker2
sudo hostnamectl set-hostname k8s-worker3
sudo hostnamectl set-hostname k8s-worker4

cat >> /etc/hosts << EOF
192.168.59.101 k8s-master01
192.168.59.102 k8s-master02
192.168.59.103 k8s-master03
192.168.59.111 k8s-worker1
192.168.59.112 k8s-worker2
192.168.59.113 k8s-worker3
192.168.59.114 k8s-worker4
EOF
```
#### <p class="note note-primary"> 允许 iptables 检查桥接流量</p>
#### <p class="note note-primary"> primary</p>
#### <p class="note note-secondary"> secondary</p>
#### <p class="note note-success"> success</p>
#### <p class="note note-danger"> danger</p>
#### <p class="note note-warning"> warning</p>
#### <p class="note note-info"> info</p>
#### <p class="note note-light"> light</p>

{% fold info @title %}
需要折叠的一段内容，支持 markdown
{% endfold %}

{% cb 普通示例 %}
{% cb 默认选中, true %}
{% cb 内联示例, false, true %} 后面文字不换行
{% cb false %} 也可以只传入一个参数，文字写在后边（这样不支持外联）


```
cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```
```
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
```
sysctl --system
```

配置 Kubernetes软件源
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

cat <<EOF >>/etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/core:/stable:/v1.32/deb/ /
# deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/addons:/cri-o:/stable:/v1.32/deb/ /
EOF

apt-get update
apt-cache madison kubeadm


```
```
cat <<EOF >>/etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/core:/stable:/v1.32/deb/ /
# deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/kubernetes/addons:/cri-o:/stable:/v1.32/deb/ /
EOF
apt-get update
```
获取最新软件包 

```
apt-cache madison kubeadm
```
安装并查看版本
```
apt-get -y install kubelet=1.32.8-1.1 kubeadm=1.32.8-1.1 kubectl=1.32.8-1.1 
```

配置 ipvsadm 模块
```
apt install -y ipset ipvsadm

cat << EOF | tee /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_rr
ip_VS_wrr
ip_vs_sh
nf_conntrack
EOF

cat << EOF | tee ipvs.sh
#!/bin/sh
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF

sh ipvs.sh

lsmod | grep ip_vs #验证
```
安装 cri-docker
```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.17/cri-dockerd-0.3.17.amd64.tgz
tar xf cri-dockerd-0.3.16.amd64.tgz
mv cri-dockerd/cri-dockerd /usr/local/bin/
cri-dockerd --version
```


#### <div style="background-color:cadetblue">三、CNI安装</div>
安装 docker-ce
```
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt purge docker-desktop

sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release software-properties-common

curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

systemctl enable --now docker
```
修改docker的home目录
```
systemctl stop docker
mkdir /data/docker && rsync -avz /var/lib/docker/ /data/docker/
vim /etc/docker/daemon.json
{
  "data-root": "/data/docker",
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live",
    "https://docker.xuanyuan.me",
    "https://hub.rat.dev",
    "https://docker.chenby.cn"
  ]
}
systemctl restart docker
```
<p class="note note-danger"> 新版本用data-root，旧的docker版本用graph，docker11及其之前为旧版本</p>


设置docker的镜像加速
```
vim /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live",
    "https://docker.xuanyuan.me",
    "https://hub.rat.dev",
    "https://docker.chenby.cn"
  ]
}
```
查看 cgroup 的管理进程需要为 systemd ubuntu 默认不用修改
```
root@k8s-worker2:~# docker info  | grep "Cgroup Driver:"
 Cgroup Driver: systemd
```
cri-docker安装
```
https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.18/cri-dockerd-0.3.18.amd64.tgz

https://github.com/Mirantis/cri-dockerd/blob/master/packaging/systemd/cri-docker.service

tar xf cri-dockerd-0.3.18.amd64.tgz
mv cri-dockerd/cri-dockerd /usr/local/bin/
```
vim /etc/systemd/system/cri-dockerd.service
```
cat > /etc/systemd/system/cri-dockerd.service<<-EOF
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket     #system cri-docker.socket  文件名
 
[Service]
Type=notify
ExecStart=/usr/local/bin/cri-dockerd --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.10
 --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --container-runtime-endpoint=unix:///var/run/cri-dockerd.sock --cri-dockerd-root-directory=/var/lib/dockershim --docker-endpoint=unix:///var/run/docker.sock --cri-dockerd-root-directory=/var/lib/docker
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF

```

```
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/cri-dockerd --container-runtime-endpoint fd://  --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.10
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
```
vim /etc/systemd/system/cri-docker.socket
```
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service
 
[Socket]
ListenStream=/var/run/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker
 
[Install]
WantedBy=sockets.target
```


#### <div style="background-color:cadetblue">四、k8s安装</div>
```shell
kubeadm init \
--kubernetes-version=1.32.8 \
--control-plane-endpoint=k8s-master01 \
--apiserver-advertise-address=192.168.59.101 \
--pod-network-cidr=10.244.0.0/16 \
--service-cidr=10.96.0.0/12 \
--image-repository=registry.aliyuncs.com/google_containers \
--cri-socket=unix:///var/run/cri-dockerd.sock \
--upload-certs \
--v=9

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes running the following command on each as root:

  kubeadm join k8s-master01:6443 --token 1yz2yk.8q8ri9fu4qqpv21x \
        --discovery-token-ca-cert-hash sha256:4e1620c5424ea5cc67d0c1e6baf7300229a83631a992b82cce5223931b4f1c62 \
        --control-plane --certificate-key 3dbd93a3da9ee74226c7655d74e561b6b710af763414ecdb734f502700e984fd

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master01:6443 --token 1yz2yk.8q8ri9fu4qqpv21x \
        --discovery-token-ca-cert-hash sha256:4e1620c5424ea5cc67d0c1e6baf7300229a83631a992b82cce5223931b4f1c62

```




#### <div style="background-color:cadetblue">五、网络插件安装</div>
在集群上安装操作员
```sh
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/tigera-operator.yaml
```
下载配置 Calico 所需的自定义资源。
```sh
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/custom-resources.yaml -O
```
修改custom-resources.yaml
```
vim custom-resources.yaml
# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default     
spec: 
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26 
      cidr: 10.244.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1 
kind: APIServer
metadata:
  name: default
spec: {}
```

创建资源
```sh
kubectl create -f custom-resources.yaml
```
```sh
root@devops:/home/devops# kubectl get pods -A
NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE
calico-apiserver   calico-apiserver-7c55574965-4fqvc          1/1     Running   0          8m47s
calico-apiserver   calico-apiserver-7c55574965-t2lmp          1/1     Running   0          8m47s
calico-system      calico-kube-controllers-6676fcc8f9-nw8sn   1/1     Running   0          8m47s
calico-system      calico-node-c5mqf                          1/1     Running   0          8m47s
calico-system      calico-node-d8lgt                          1/1     Running   0          8m47s
calico-system      calico-node-ggk59                          1/1     Running   0          8m47s
calico-system      calico-node-wslzv                          1/1     Running   0          8m47s
calico-system      calico-typha-668fc84bf-chkbx               1/1     Running   0          8m47s
calico-system      calico-typha-668fc84bf-hg8f2               1/1     Running   0          8m38s
```

#### <div style="background-color:cadetblue">六、安装验证</div>

#### <div style="background-color:cadetblue">七、镜像代理配置</div>

---
#### 参考
[^1]: 参考资料1
[^2]: 参考资料2