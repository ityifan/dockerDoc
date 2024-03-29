```shell
# 关闭防火墙

# 三台机器全部操作

systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 关闭完swap后，一定要重启一下虚拟机！！！
# 根据规划设置主机名
hostnamectl set-hostname <hostname>

hostnamectl set-hostname k8s-master


cat >> /etc/hosts << EOF
192.168.1.106 k8s-master
192.168.1.163 k8s-node1
192.168.1.184 k8s-node2
EOF


# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system  # 生效


# 时间同步
yum install ntpdate -y
ntpdate time.windows.com
```



```javascript
192.168.1.106 master
192.168.1.163 node1
192.168.1.184 node2
```

给三个机器都安装docker



```shell
#安装versio以下版本20

# sudo yum install docker-ce-20.10.5 docker-ce-cli-20.10.5 containerd.io
```

三个机器添加阿里云yum源

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

arm如下

vim /etc/yum.repos.d/kubernetes.repo

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-aarch64
enabled=1
gpgcheck=0
repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

三个机器安装 kubeadm kubelet kubectl 



```shell
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6

systemctl enable kubelet

# 配置关闭 Docker 的 cgroups，修改 /etc/docker/daemon.json，加入以下内容
"exec-opts": ["native.cgroupdriver=systemd"]

# 重启 docker
systemctl daemon-reload
systemctl restart docker
```



部署kuberneters master

```shell

kubeadm init \
      --apiserver-advertise-address=192.168.1.106 \
      --image-repository registry.aliyuncs.com/google_containers \
      --kubernetes-version v1.23.6 \
      --service-cidr=10.96.0.0/12 \
      --pod-network-cidr=10.244.0.0/16




sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.23.5 --apiserver-advertise-address 192.168.2.6 --pod-network-cidr=10.244.0.0/16 --token-ttl 0
```



其余两个机器运行 node1-node2

```
kubeadm join 192.168.1.155:6443 --token 8i2eux.9oa33vt8kw66xwze \
        --discovery-token-ca-cert-hash sha256:3601db0ee0cd63d14e4645b98d6c35321045b9a22bd9675644bce0879165766d
```





部署cni网络插件





```shell
 curl https://docs.tigera.io/archive/v3.25/manifests/calico.yaml -O


 
 # 修改 calico.yaml 文件中的 CALICO_IPV4POOL_CIDR 配置，修改为与初始化的 cidr 相同

# 修改 IP_AUTODETECTION_METHOD 下的网卡名称

# 删除镜像 docker.io/ 前缀，避免下载过慢导致失败
sed -i 's#docker.io/##g' calico.yaml
 
 #输入/寻找
```



测试kubernetes集群



```shell
# 创建部署
kubectl create deployment nginx --image=nginx

# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看 pod 以及服务信息
kubectl get pod,svc
```



在任意节点使用kubectl  即给其他节点 admin权限





```shell
# 1. 将 master 节点中 /etc/kubernetes/admin.conf 拷贝到需要运行的服务器的 /etc/kubernetes 目录中
scp /etc/kubernetes/admin.conf root@k8s-node1:/etc/kubernetes

# 2. 在对应的服务器上配置环境变量
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```



基础命令



```shell
#获取所以pod
kubectl get pods 

#获取所有deploy
kubectl get deploy

#停止运行的nginx
kubectl delete deploy nginx

kubectl get deploy

#获取所有的services
kubectl get services（svc）

#杀死死nginx services
kubectl delete svc nginx

kubectl get services（svc）
```
