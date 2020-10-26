---
title: 部署k8s+istio 
categories:
- k8s
- 部署
feature_image: "https://picsum.photos/2560/600?image=872"
---

### 目录
* 部署 docker
* 部署 k8s （1.19.3）
* 部署 istio （1.7）


### 安装 docker
```
# 在 docker 官网上找到自己系统所对应的安装包并下载
wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.03.3~ce-0~ubuntu-xenial_amd64.deb

# 安装依赖
sudo apt install libltdl7

# 安装（包名替换成下载的包名）
sudo dpkg -i docker-ce_17.03.3~ce-0~ubuntu-xenial_amd64.deb

# 测试
sudo docker run hello-world
```

### 安装 k8s（kubeadm，version-1.19.3，1+1）
```
## 每一台主机上安装 kubeadm， kubectl， kubelet
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

## 在master 节点上下载所需的镜像

#!/bin/bash
set -e
KUBE_VERSION=v1.19.3
KUBE_PAUSE_VERSION=3.2
ETCD_VERSION=3.4.13-0
CORE_DNS_VERSION=1.7.0

GCR_URL=k8s.gcr.io
ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers

images=(kube-proxy:${KUBE_VERSION}
kube-scheduler:${KUBE_VERSION}
kube-controller-manager:${KUBE_VERSION}
kube-apiserver:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
pause-amd64:${KUBE_PAUSE_VERSION}
etcd:${ETCD_VERSION}
coredns:${CORE_DNS_VERSION})

for imageName in ${images[@]} ; do
  docker pull $ALIYUN_URL/$imageName
  docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
  docker rmi $ALIYUN_URL/$imageName
done

## 关闭 swap
sudo swapoff -a  

### 安装 master
sudo kubeadm init --kubernetes-version=v1.19.3  --pod-netwrok-cidr=10.244.0.0/24

### 初始化用户权限
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

### 安装网络插件
### https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml

### 因为 quay.io 现在国内无法访问，flannel 安装会失败。但是可以通过 github 上的资源下载
wget https://github.com/coreos/flannel/releases/download/v0.13.0/flanneld-v0.13.0-amd64.docker

### 导入镜像, 启动 flannel
docker load --input flanneld-v0.13.0-amd64.docker

### 遇到 nameserver 超出数量限制的问题（flannel 做的限制nameserver 最多三个）

### 加入woker 节点
### 通过 kubeadm init 返回的 kubeadm join 命令 接入

```


### 部署 istio （1.7）

```
## 下载 istio
curl -L https://istio.io/downloadIstio | sh -

## 安装 istio （profile 参数请针对具体情况选择）
istioctl install --set profile=demo

## 安装 bookinfo 实例程序
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

```
