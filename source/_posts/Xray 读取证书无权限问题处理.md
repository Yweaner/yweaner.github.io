---
title: Xray 读取证书无权限问题处理
date: 2025-02-20 08:33:17
comments: true
categories: 建站
---
# Xray 读取证书无权限问题处理 /your_certificate_path/fullchain.cer : Permission denied

## 问题描述

使用GitHub上Xray官网的安装脚本安装Xray后，正常获取证书，Xray证书路径也正确配置，但是始终报读取证书无权限问题。

安装脚本：https://github.com/XTLS/Xray-install/blob/main/install-release.sh

异常信息：

```bash
[linuxuser@vultr ~]$ sudo journalctl -u xray -f
/your_certificate_path/fullchain.cer : Permission denied
```

## 问题原因

由于安装脚本要求必须使用sudo执行，Xray默认的执行用户是nobody，所以导致无权限问题。
```bash
[linuxuser@vultr ~]$ sudo nano /etc/systemd/system/xray.service
[Unit]
Description=Xray Service
Documentation=https://github.com/xtls
After=network.target nss-lookup.target

[Service]
User=nobody
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
NoNewPrivileges=true
ExecStart=/usr/local/bin/xray run -config /usr/local/etc/xray/config.json
Restart=on-failure
RestartPreventExitStatus=23
LimitNPROC=10000
LimitNOFILE=1000000

[Install]
WantedBy=multi-user.target
```
linux系统nobody用户的说明。
> 1. nobody 用户的定义
>
> 用户 ID (UID)：nobody 的 UID 通常是 65534（在某些系统中可能是 99 或其他值）。
>
> 组 ID (GID)：nobody 通常属于 nogroup 组，GID 也是 65534。
>
> 家目录：nobody 用户没有家目录。
>
> Shell：nobody 用户通常没有有效的登录 Shell（例如 /usr/sbin/nologin 或 /bin/false）。
>
> 2. nobody 用户的权限
>
> nobody 是一个低权限用户，其权限非常有限：
>
>文件系统权限：nobody 只能访问具有适当权限的文件和目录（例如，文件权限为 644 或目录权限为 755）。
>
> 系统权限：nobody 没有特权（例如，不能安装软件、修改系统配置或访问受保护的系统资源）。
>
> 网络权限：nobody 可以绑定到高于 1024 的端口（非特权端口），但不能绑定到低于 1024 的端口（特权端口）。
>
> 3. nobody 用户的用途
>
> nobody 用户通常用于以下场景：
>
> 运行服务：许多服务（如 Web 服务器、代理服务器等）默认以 nobody 用户运行，以降低安全风险。
>
> 隔离进程：通过以 nobody 用户运行进程，可以限制其对系统资源的访问，防止潜在的漏洞被利用。
>
> 临时任务：某些脚本或临时任务可能以 nobody 用户运行，以避免对系统造成影响。
>
> 4. nobody 用户的限制
>
> 由于 nobody 用户的权限非常低，可能会导致一些问题：
>
> 文件访问问题：如果文件或目录的权限过于严格（例如，仅 root 可读），nobody 用户将无法访问。
>
> 端口绑定问题：nobody 用户无法绑定到低于 1024 的端口（如 HTTP 的 80 端口或 HTTPS 的 443 端口）。
>
> 资源限制：nobody 用户可能受到系统资源限制（如打开文件数、内存使用等）的影响。
>
## 问题处理

修改文件权限：确保服务所需的文件和目录对 nobody 用户可读或可写。


``` bash
sudo chmod 644 /path/to/file
sudo chown nobody:nogroup /path/to/file
```
第二个命令尤其重要，只修改文件权限是不行的，需要让nobody用于目录的读取权限。