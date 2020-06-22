---
title: Kubernetes更换阿里云软件源
date: 2020-06-15 21:51:29
tags:
- Kubernetes
- k8s
- docker
categories:
- k8s
---

## kubelet kubeadm kubectl 安装

* 更新软件源，并安装：

```bash
apt-get update && apt-get install -y apt-transport-https curl
```

* 添加阿里软件源：

```bash
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt-get update
```

<!-- more -->

* 安装kubernetes相关：

```bash
apt-get install -y kubelet kubeadm kubectl
```

---

## 更换阿里云镜像仓库

* 关闭swap虚拟内存交换区：

```bash
sudo swapoff -a
```

* 配置阿里云镜像加速器：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://2ltes9ox.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

* 查看kubernetes需要的镜像：

```bash
kubeadm config images list
```

* 下载kubernetes需要的镜像：

```bash
k8s.gcr.io/kube-apiserver:v1.18.3
k8s.gcr.io/kube-controller-manager:v1.18.3
k8s.gcr.io/kube-scheduler:v1.18.3
k8s.gcr.io/kube-proxy:v1.18.3
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
```

* 拉取镜像：

```bash
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.3
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.3
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3
docker pull registry.aliyuncs.com/google_containers/pause:3.2
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.3-0
docker pull registry.aliyuncs.com/google_containers/coredns:1.6.7
```

* 重命名镜像名称：

```bash
docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.3 k8s.gcr.io/kube-apiserver:v1.18.3
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.3 k8s.gcr.io/kube-controller-manager:v1.18.3
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.3 k8s.gcr.io/kube-scheduler:v1.18.3
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.18.3 k8s.gcr.io/kube-proxy:v1.18.3
docker tag registry.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
docker tag registry.aliyuncs.com/google_containers/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
docker tag registry.aliyuncs.com/google_containers/coredns:1.6.7 k8s.gcr.io/coredns:1.6.7
```
