---
title: debian12 安装 Containerd 容器运行时
excerpt: debian12 安装 Containerd 容器运行时
index_img: /img/2023/containerd.png
date: 2025-04-03 11:47:00
tags:
  - Linux
  - Docker
  - Containerd
categories:
  - 容器化
---

# 安装 debian12.10 系统

最小标准化安装 server 版

## 配置系统

`init_debain12.sh`

```bash
wget https://raw.githubusercontent.com/cuilan/shell_scripts/refs/heads/main/debian/init_debain12.sh
```

---

# 安装 containerd

## Step 1: Installing containerd

v2.0.0

```bash
wget https://github.com/containerd/containerd/releases/download/v2.0.0/containerd-2.0.0-linux-amd64.tar.gz

tar Cxzvf /usr/local containerd-2.0.0-linux-amd64.tar.gz

containerd --version
```

## Step2: enable systemd

```bash
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

sudo mkdir -p /usr/local/lib/systemd/system
sudo mv containerd.service /usr/local/lib/systemd/system/containerd.service

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

systemd

```systemd
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

## Step 3: Installing runc

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.2.6/runc.amd64

sudo install -m 755 runc.amd64 /usr/local/sbin/runc

runc --version
# runc version 1.2.6
# commit: v1.2.6-0-ge89a2992
# spec: 1.2.0
# go: go1.23.7
# libseccomp: 2.5.5

runc spec
runc run --help
```

## Step 4: Installing CNI plugins

```bash
wget https://github.com/containernetworking/plugins/releases/download/v1.6.2/cni-plugins-linux-amd64-v1.6.2.tgz

mkdir -p /opt/cni/bin/

tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.6.2.tgz

ls /opt/cni/bin/
# bridge  dhcp  flannel  host-local  ipvlan  loopback  macvlan  portmap  ptp  sbr  tuning  vlan
```

配置CNI（可选）

如果用于 Kubernetes 或容器运行时（如 CRI-O、containerd），需配置 CNI 网络：

```bash
sudo mkdir -p /etc/cni/net.d/

touch /etc/cni/net.d/10-mynet.conf
```

```json
{
  "cniVersion": "0.4.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.22.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
```

---

# 安装 nerdctl

## Minimal installation

```bash
wget https://github.com/containerd/nerdctl/releases/download/v2.0.4/nerdctl-2.0.4-linux-amd64.tar.gz

tar Cxzvf /usr/local/bin nerdctl-2.0.4-linux-amd64.tar.gz
```

```bash
ls -l /usr/local/bin/
-rwxr-xr-x root/root  27721912 2025-03-20 15:04 nerdctl
-rwxr-xr-x root/root     22657 2025-03-20 15:04 containerd-rootless-setuptool.sh
-rwxr-xr-x root/root      8708 2025-03-20 15:04 containerd-rootless.sh
```

## Full installation

```bash
wget https://github.com/containerd/nerdctl/releases/download/v2.0.4/nerdctl-full-2.0.4-linux-amd64.tar.gz

tar Cxzvf /usr/local/bin nerdctl-full-2.0.4-linux-amd64.tar.gz
```

```bash
ls -l /usr/local/bin/
drwxr-xr-x 0/0               0 2025-03-20 15:11 bin/
-rwxr-xr-x 0/0        31114986 2015-10-21 00:00 bin/buildctl
-rwxr-xr-x 0/0        23724032 2022-09-05 09:52 bin/buildg
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-LICENSE -> ../libexec/cni/LICENSE
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-README.md -> ../libexec/cni/README.md
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-bandwidth -> ../libexec/cni/bandwidth
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-bridge -> ../libexec/cni/bridge
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-dhcp -> ../libexec/cni/dhcp
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-dummy -> ../libexec/cni/dummy
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-firewall -> ../libexec/cni/firewall
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-host-device -> ../libexec/cni/host-device
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-host-local -> ../libexec/cni/host-local
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-ipvlan -> ../libexec/cni/ipvlan
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-loopback -> ../libexec/cni/loopback
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-macvlan -> ../libexec/cni/macvlan
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-portmap -> ../libexec/cni/portmap
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-ptp -> ../libexec/cni/ptp
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-sbr -> ../libexec/cni/sbr
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-static -> ../libexec/cni/static
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-tap -> ../libexec/cni/tap
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-tuning -> ../libexec/cni/tuning
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-vlan -> ../libexec/cni/vlan
lrwxrwxrwx 0/0               0 2025-03-20 15:10 bin/buildkit-cni-vrf -> ../libexec/cni/vrf
-rwxr-xr-x 0/0        63159823 2015-10-21 00:00 bin/buildkitd
-rwxr-xr-x 0/0        16435808 2025-03-20 15:09 bin/bypass4netns
-rwxr-xr-x 0/0         6320312 2025-03-20 15:09 bin/bypass4netnsd
-rwxr-xr-x 0/0        39849568 2025-03-20 15:10 bin/containerd
-rwxr-xr-x 0/0        11874488 2025-03-19 10:45 bin/containerd-fuse-overlayfs-grpc
-rwxr-xr-x 0/0           22657 2025-03-20 15:11 bin/containerd-rootless-setuptool.sh
-rwxr-xr-x 0/0            8708 2025-03-20 15:11 bin/containerd-rootless.sh
-rwxr-xr-x 0/0         8028344 2025-03-20 15:10 bin/containerd-shim-runc-v2
-rwxr-xr-x 0/0        53432472 2024-12-12 14:54 bin/containerd-stargz-grpc
-rwxr-xr-x 0/0        23513933 2025-03-20 15:11 bin/ctd-decoder
-rwxr-xr-x 0/0        20504760 2025-03-20 15:10 bin/ctr
-rwxr-xr-x 0/0        31163196 2025-03-20 15:11 bin/ctr-enc
-rwxr-xr-x 0/0        21078168 2024-12-12 14:54 bin/ctr-remote
-rwxr-xr-x 0/0         1789968 2025-03-20 15:11 bin/fuse-overlayfs
-rwxr-xr-x 0/0        27689144 2025-03-20 15:11 bin/nerdctl
-rwxr-xr-x 0/0        12235661 2025-03-10 02:16 bin/rootlessctl
-rwxr-xr-x 0/0        14172281 2025-03-10 02:16 bin/rootlesskit
-rwxr-xr-x 0/0        17433000 2025-03-20 15:09 bin/runc
-rwxr-xr-x 0/0         2383224 2025-03-20 15:11 bin/slirp4netns
-rwxr-xr-x 0/0         9707672 2024-12-12 14:54 bin/stargz-store-helper
-rwxr-xr-x 0/0          870496 2025-03-20 15:11 bin/tini
drwxr-xr-x 0/0               0 2025-03-20 15:10 lib/
drwxr-xr-x 0/0               0 2025-03-20 15:10 lib/systemd/
drwxr-xr-x 0/0               0 2025-03-20 15:10 lib/systemd/system/
-rw-r--r-- 0/0            1325 2025-03-20 15:10 lib/systemd/system/buildkit.service
-rw-r--r-- 0/0            1264 2025-03-20 15:09 lib/systemd/system/containerd.service
-rw-r--r-- 0/0             312 2025-03-20 15:10 lib/systemd/system/stargz-snapshotter.service
drwxr-xr-x 0/0               0 2025-03-20 15:10 libexec/
drwxr-xr-x 0/0               0 2025-03-20 15:10 libexec/cni/
-rw-r--r-- 0/0           11357 2025-01-06 16:12 libexec/cni/LICENSE
-rw-r--r-- 0/0            2343 2025-01-06 16:12 libexec/cni/README.md
-rwxr-xr-x 0/0         4655178 2025-01-06 16:12 libexec/cni/bandwidth
-rwxr-xr-x 0/0         5287212 2025-01-06 16:12 libexec/cni/bridge
-rwxr-xr-x 0/0        12762814 2025-01-06 16:12 libexec/cni/dhcp
-rwxr-xr-x 0/0         4847854 2025-01-06 16:12 libexec/cni/dummy
-rwxr-xr-x 0/0         5315134 2025-01-06 16:12 libexec/cni/firewall
-rwxr-xr-x 0/0         4792010 2025-01-06 16:12 libexec/cni/host-device
-rwxr-xr-x 0/0         4060355 2025-01-06 16:12 libexec/cni/host-local
-rwxr-xr-x 0/0         4870719 2025-01-06 16:12 libexec/cni/ipvlan
-rwxr-xr-x 0/0         4114939 2025-01-06 16:12 libexec/cni/loopback
-rwxr-xr-x 0/0         4903324 2025-01-06 16:12 libexec/cni/macvlan
-rwxr-xr-x 0/0         4713429 2025-01-06 16:12 libexec/cni/portmap
-rwxr-xr-x 0/0         5076613 2025-01-06 16:12 libexec/cni/ptp
-rwxr-xr-x 0/0         4333422 2025-01-06 16:12 libexec/cni/sbr
-rwxr-xr-x 0/0         3651755 2025-01-06 16:12 libexec/cni/static
-rwxr-xr-x 0/0         4928874 2025-01-06 16:12 libexec/cni/tap
-rwxr-xr-x 0/0         4208424 2025-01-06 16:12 libexec/cni/tuning
-rwxr-xr-x 0/0         4868252 2025-01-06 16:12 libexec/cni/vlan
-rwxr-xr-x 0/0         4488658 2025-01-06 16:12 libexec/cni/vrf
drwxr-xr-x 0/0               0 2025-03-20 15:08 share/
drwxr-xr-x 0/0               0 2025-03-20 15:11 share/doc/
drwxr-xr-x 0/0               0 2025-03-20 15:11 share/doc/nerdctl/
-rw-r--r-- 0/0           12101 2025-03-20 15:04 share/doc/nerdctl/README.md
drwxr-xr-x 0/0               0 2025-03-20 15:04 share/doc/nerdctl/docs/
-rw-r--r-- 0/0            3953 2025-03-20 15:04 share/doc/nerdctl/docs/build.md
-rw-r--r-- 0/0            2570 2025-03-20 15:04 share/doc/nerdctl/docs/builder-debug.md
-rw-r--r-- 0/0            4779 2025-03-20 15:04 share/doc/nerdctl/docs/cni.md
-rw-r--r-- 0/0           81663 2025-03-20 15:04 share/doc/nerdctl/docs/command-reference.md
-rw-r--r-- 0/0            1814 2025-03-20 15:04 share/doc/nerdctl/docs/compose.md
-rw-r--r-- 0/0            5814 2025-03-20 15:04 share/doc/nerdctl/docs/config.md
-rw-r--r-- 0/0            9128 2025-03-20 15:04 share/doc/nerdctl/docs/cosign.md
-rw-r--r-- 0/0            5660 2025-03-20 15:04 share/doc/nerdctl/docs/cvmfs.md
drwxr-xr-x 0/0               0 2025-03-20 15:04 share/doc/nerdctl/docs/dev/
-rw-r--r-- 0/0           12859 2025-03-20 15:04 share/doc/nerdctl/docs/dev/auditing_dockerfile.md
-rw-r--r-- 0/0            8587 2025-03-20 15:04 share/doc/nerdctl/docs/dev/store.md
-rw-r--r-- 0/0            2776 2025-03-20 15:04 share/doc/nerdctl/docs/dir.md
-rw-r--r-- 0/0             906 2025-03-20 15:04 share/doc/nerdctl/docs/experimental.md
-rw-r--r-- 0/0           14687 2025-03-20 15:04 share/doc/nerdctl/docs/faq.md
-rw-r--r-- 0/0             884 2025-03-20 15:04 share/doc/nerdctl/docs/freebsd.md
-rw-r--r-- 0/0            3273 2025-03-20 15:04 share/doc/nerdctl/docs/gpu.md
drwxr-xr-x 0/0               0 2025-03-20 15:04 share/doc/nerdctl/docs/images/
-rw-r--r-- 0/0            1540 2025-03-20 15:04 share/doc/nerdctl/docs/images/nerdctl-white.svg
-rw-r--r-- 0/0            1462 2025-03-20 15:04 share/doc/nerdctl/docs/images/nerdctl.svg
-rw-r--r-- 0/0          684421 2025-03-20 15:04 share/doc/nerdctl/docs/images/rootlessKit-network-design.png
-rw-r--r-- 0/0           14462 2025-03-20 15:04 share/doc/nerdctl/docs/ipfs.md
-rw-r--r-- 0/0            2384 2025-03-20 15:04 share/doc/nerdctl/docs/multi-platform.md
-rw-r--r-- 0/0            2960 2025-03-20 15:04 share/doc/nerdctl/docs/notation.md
-rw-r--r-- 0/0            2610 2025-03-20 15:04 share/doc/nerdctl/docs/nydus.md
-rw-r--r-- 0/0            3277 2025-03-20 15:04 share/doc/nerdctl/docs/ocicrypt.md
-rw-r--r-- 0/0            1876 2025-03-20 15:04 share/doc/nerdctl/docs/overlaybd.md
-rw-r--r-- 0/0           15657 2025-03-20 15:04 share/doc/nerdctl/docs/registry.md
-rw-r--r-- 0/0            9147 2025-03-20 15:04 share/doc/nerdctl/docs/rootless.md
-rw-r--r-- 0/0            2015 2025-03-20 15:04 share/doc/nerdctl/docs/soci.md
-rw-r--r-- 0/0           10312 2025-03-20 15:04 share/doc/nerdctl/docs/stargz.md
drwxr-xr-x 0/0               0 2025-03-20 15:04 share/doc/nerdctl/docs/testing/
-rw-r--r-- 0/0            4916 2025-03-20 15:04 share/doc/nerdctl/docs/testing/README.md
-rw-r--r-- 0/0           15584 2025-03-20 15:04 share/doc/nerdctl/docs/testing/tools.md
drwxr-xr-x 0/0               0 2025-03-20 15:11 share/doc/nerdctl-full/
-rw-r--r-- 0/0             999 2025-03-20 15:11 share/doc/nerdctl-full/README.md
-rw-r--r-- 0/0            9232 2025-03-20 15:11 share/doc/nerdctl-full/SHA256SUMS
```

## Quick start

### Rootful root用户运行容器

```bash
$ sudo systemctl enable --now containerd
$ sudo nerdctl run -d --name nginx -p 80:80 nginx:alpine
```

### Rootless 非root用户运行容器

```bash
$ containerd-rootless-setuptool.sh install
$ nerdctl run -d --name nginx -p 8080:80 nginx:alpine
```

---

# 配置 containerd

## 生成默认配置文件

```bash
containerd config default > /etc/containerd/config.toml
```

## 配置私有仓库

* In containerd 2.x

```bash
version = 3

[plugins.'io.containerd.cri.v1.images'.registry]
  config_path = '/etc/containerd/certs.d'
```

* In containerd 1.x

```bash
version = 2

[plugins."io.containerd.grpc.v1.cri".registry]
   config_path = "/etc/containerd/certs.d"
```

## 配置认证信息

```bash
# 创建目录
sudo mkdir -p /etc/containerd/certs.d/reg.weattech.com

sudo touch /etc/containerd/certs.d/reg.weattech.com/hosts.toml

sudo tee /etc/containerd/certs.d/reg.weattech.com/hosts.toml <<EOF
server = "https://reg.weattech.com"

[host."https://reg.weattech.com"]
  capabilities = ["pull", "resolve", "push"]
  [host."https://reg.weattech.com".auth]
    username = "admin"
    password = "password"
EOF

sudo systemctl restart containerd
```

## nerdctl login

nerdctl 需要单独登录并认证，提供类似 docker login 的功能，认证文件存储在：`/root/.docker/config.json`。

```bash
$ nerdctl login reg.weattech.com
Enter Username:
Enter Password: 

WARNING! Your credentials are stored unencrypted in '/root/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded

$ nerdctl pull reg.weattech.com/base/alpine:3.20
$ nerdctl pull reg.weattech.com/dockerhub/nginx:1.22-alpine
```
