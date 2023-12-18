---
title: 使用kind构建k8s开发环境
excerpt: kind 使用 Docker 容器运行本地 Kubernetes 集群的工具。
index_img: /img/2023/kind.png
date: 2023-12-15 12:28:00
tags:
  - docker
  - k8s
categories:
  - docker
  - k8s
---

# 一、依赖

## 1、docker

* Linux 环境安装 docker
* MacOS 环境建议安装 `lima`
* Windows 环境安装 docker-desktop

修改 docker 配置，镜像加速、日志配置、存储路径等。

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<- 'EOF'
{
    "data-root": "/data/docker",
    "debug": true,
    "experimental": true,
    "insecure-registries": [
        "docker.mirrors.ustc.edu.cn"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-file": "3",
        "max-size": "50m"
    },
    "registry-mirrors": [
        "http://docker.mirrors.ustc.edu.cn",
        "http://hub-mirror.c.163.com"
    ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2、kubectl 工具

### 2.1 MacOS

#### curl 安装

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
```

#### Homebrew

```bash
brew install kubectl
```

### 2.2 Windows

```PowerShell
curl.exe -LO "https://dl.k8s.io/release/v1.29.0/bin/windows/amd64/kubectl.exe"
```

### 2.3 Linux

#### 下载安装

```bash
# 下载最新稳定版
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# 下载指定版本，如：v1.29.0
curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl
```

#### 包管理方式安装

##### Debian

```bash
sudo apt-get update

# apt-transport-https 可以是一个虚拟包；如果是这样，你可以跳过这个包
sudo apt-get install -y apt-transport-https ca-certificates curl

# 下载 Kubernetes 软件包仓库的公共签名密钥
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# 这会覆盖 /etc/apt/sources.list.d/kubernetes.list 中的所有现存配置
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl
```

##### RedHat

```bash
# 这会覆盖 /etc/yum.repos.d/kubernetes.repo 中现存的所有配置
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
EOF

yum update

sudo yum install -y kubectl
```

# 二、安装 kind

## 1、MacOS

### 1.1 Homebrew

```bash
brew install kind
```

### 1.2 MacPorts

```bash
sudo port selfupdate && sudo port install kind
```

## 2、Windows

### 2.1 Chocolatey

```PowerShell
choco install kind
```

## 3、Binaries

### 3.1 Linux

```bash
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
# 验证版本
kind version
```

### 3.2 MacOS

```bash
# For Intel Macs
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-darwin-amd64
# For M1 / ARM Macs
[ $(uname -m) = arm64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-darwin-arm64
chmod +x ./kind
mv ./kind /some-dir-in-your-PATH/kind
# 验证版本
kind version
```

### 3.3 PowerShell

```PowerShell
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.20.0/kind-windows-amd64
Move-Item .\kind-windows-amd64.exe c:\some-dir-in-your-PATH\kind.exe
```

# 三、创建集群

## 1、创建初始化脚本

创建一个控制面节点、两个工作节点的集群，对外暴露端口：31000

```bash
mkdir -p /home/kind
cd /home/kind
cat > 1c2w.yaml <<- 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
# One control plane node and two "workers".
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 31000
    hostPort: 31000
    # Optional, defaults to "0.0.0.0"
    listenAddress: "0.0.0.0"
    # Optional, defaults to tcp
    protocol: tcp
- role: worker
- role: worker
EOF
```

## 2、运行集群

```bash
kind create cluster --config /home/kind/1c2w.yaml --name 1c2w
```

终端输出：

```bash
Creating cluster "1c2w" ...
 ✓ Ensuring node image (kindest/node:v1.27.1) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-1c2w"
You can now use your cluster with:

kubectl cluster-info --context kind-1c2w

Have a nice day! 👋
```

## 3、查看集群状态

```bash
# 使用 kubectl 查看集群信息
kubectl cluster-info --context kind-1c2w
```

终端输出：

```bash
Kubernetes control plane is running at https://127.0.0.1:25909
CoreDNS is running at https://127.0.0.1:25909/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```bash
# 使用 docker 命令查看节点
docker ps -a
```

```bash
CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS         PORTS                                                 NAMES
e65f01903581   kindest/node:v1.27.1   "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes                                                         1c2w-worker2
2fea583a6b6b   kindest/node:v1.27.1   "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes   0.0.0.0:31000->31000/udp, 127.0.0.1:25909->6443/tcp   1c2w-control-plane
0a0512191340   kindest/node:v1.27.1   "/usr/local/bin/entr…"   3 minutes ago   Up 3 minutes                                                         1c2w-worker
```

# 四、部署应用

## 1、deploy nginx pod

```bash
mkdir -p /home/k8s
cd /home/k8s

cat > ./deploy-nginx.yaml <<- 'EOF'
# 使用的API版本，最新版本Kubernetes位于apps/v1的API组
apiVersion: apps/v1
# 声明定义的是一个Deployment对象
kind: Deployment
# metadata 部分定义Deployment的名字和标签
metadata:
  # 部署名称
  name: deploy-nginx
  labels:
    app: deploy-nginx
# spec 下的内容都与Pod有关
spec:
  # spec.replicas 告诉k8s需要部署多少个Pod副本
  replicas: 3
  # spec.selector 表明Deployment要管理的Pod所必备的标签
  selector:
    # 匹配所有标签app且值为deploy-nginx
    matchLabels:
      app: deploy-nginx
  minReadySeconds: 10
  # spec.template 下的内容定义了Deployment管理的Pod的模板
  template:
    # 元数据定义每一个Pod拥有的标签 key=app, value=deploy-nginx
    metadata:
      labels:
        app: deploy-nginx
    # spec.template.spec 说明具体部署的容器与镜像信息
    spec:
      containers:
      - name: deploy-nginx
        image: nginx:1.20.2-alpine
        # 镜像拉取策略，Always：每次启动或重启都会拉取，IfNotPresent：不存在从远程仓库拉取
        imagePullPolicy: Always
        # 容器对外暴露端口
        ports:
        - containerPort: 80
EOF
```

```bash
# delete old
kubectl delete deploy deploy-nginx
# deploy new
kubectl apply -f ./deploy-nginx.yaml
```

## 2、create svc

```bash
cd /home/k8s

cat > ./svc-nginx.yaml <<- 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: deploy-nginx
  labels:
    svc: deploy-nginx
spec:
  type: NodePort
  selector:
    app: deploy-nginx
  ports:
    - port: 8000
      targetPort: 80
      nodePort: 31000
EOF
```

```bash
kubectl delete svc deploy-nginx
kubectl apply -f ./svc-nginx.yaml
```

## 3、查看运行状态

```bash
kubectl get pods

NAME                            READY   STATUS    RESTARTS   AGE
deploy-nginx-55b87cfcf7-85d48   1/1     Running   0          21s
deploy-nginx-55b87cfcf7-qvcnf   1/1     Running   0          21s
deploy-nginx-55b87cfcf7-xnqm8   1/1     Running   0          21s
```

```bash
kubectl get svc

NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
deploy-nginx   NodePort    10.96.47.102   <none>        8000:31000/TCP   3m17s
kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP          4m16s
```

访问 `http://{your host/ip}:31000`
