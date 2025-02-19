---
title: WSL2-Ubuntu-24环境下安装docker无法拉去镜像网络连接超时问题
date: 2025-01-20 11:33:17
comments: true
categories: 技术日志
---
# WSL2-Ubuntu-24环境下安装docker无法拉去镜像网络连接超时问题
## 配置 Docker 守护进程（适用于 systemd 管理的 Docker）
如果你已经完成如下配置

~/.docker/config.json 添加代理信息

etc/docker/daemon.json 添加代理信息

这两个文件后 docker pull 仍然超时，可能 Docker 守护进程（daemon） 也需要代理。

你可以在 /etc/systemd/system/docker.service.d/http-proxy.conf 中配置：

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://your-proxy:port"
Environment="HTTPS_PROXY=http://your-proxy:port"
Environment="NO_PROXY=localhost,127.0.0.1"
EOF
```
然后重新加载 systemd 并重启 Docker：
```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```
测试
```sh
docker pull hello-world
```

如果完成上述配置后，docker pull依然超时，只能尝试修改dns了
```sh
sudo nano /etc/systemd/resolved.conf
```
因为WSL2-Ubuntu-24环境中，默认配置的DNS地址是个10段地址，需要修改成

8.8.8.8

4.4.4.4

1.1.1.1

这种常规的DNS地址