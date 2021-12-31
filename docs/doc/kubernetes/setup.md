# 在Centos7环境中安装Kubernetes集群
> 以下内容基于Centos7.9环境

## 准备工作
### 安装docker

点击[这里](https://docs.docker.com/engine/install/centos/)查看安装过程，依次执行安装成功即可。

安装完成之后，需要修改docker的storage driver:

```shell
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

## 重启生效
systemctl restart docker
```

## 开始安装Kubernetes

### 修改centos源
依次执行下面的命令:

```shell
# 添加 k8s 源仓库（已替换为阿里云镜像）
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 安装
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 添加开机启动并启动
systemctl enable kubelet && systemctl start kubelet

#查看版本（验证是否安装成功）
kubectl version

#启用网络转发
echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables
```

### 下载kubernetes使用的镜像

```shell
cd ~
echo "source <(kubectl completion bash)" >> ~/.bash_profile
source .bash_profile

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.23.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.23.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.23.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.23.1
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.1-0
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.6

# 修改tag回k8s.gcr.io（重命名）
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.23.1  k8s.gcr.io/kube-apiserver:v1.23.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.23.1  k8s.gcr.io/kube-controller-manager:v1.23.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.23.1  k8s.gcr.io/kube-scheduler:v1.23.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.23.1  k8s.gcr.io/kube-proxy:v1.23.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6  k8s.gcr.io/pause:3.6
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.1-0  k8s.gcr.io/etcd:3.5.1-0
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.8.6  k8s.gcr.io/coredns/coredns:v1.8.6
```

### 通过kubeadm执行
```shell
kubeadm init
```