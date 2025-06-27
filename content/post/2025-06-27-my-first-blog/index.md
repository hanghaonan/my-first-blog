---
title: "FRP内网穿透教程"
author: "zhang hao nan"
date: "2025-06-27"
---

## 项目简介

FRP（Fast Reverse Proxy）是一个高性能的反向代理应用，主要用于内网穿透。它允许用户通过公共网络访问位于私有网络中的服务。FRP 的主要特点包括：

*   **高效性**：FRP 采用了高效的网络协议和数据传输机制，能够处理大量并发连接。
*   **灵活的配置**：支持多种代理模式，包括 TCP、UDP 和 HTTP，用户可以根据需求灵活配置。
*   **跨平台支持**：FRP 可以在多种操作系统上运行，如 Windows、Linux 和 macOS。
*   **安全性**：FRP 支持 SSL 加密，可以确保数据传输的安全性。
*   **简易的使用**：FRP 提供了简单易用的命令行工具和配置文件，使用户能够快速上手。
*   **可扩展性**：用户可以根据具体需求自定义和扩展 FRP 的功能。

FRP 适用于需要远程访问内网服务的场景，比如远程桌面、数据库访问和开发测试环境等。

简单来说，比如你家里有台服务器且没有公网IP。你在学校或者公司想要访问服务器，由于没有公网，你是访问不了的。实现内网穿透，必须要有一台有公网IP的服务器。比如各大厂商的云服务器等等。这台服务器就称之为代理服务器(Proxy Server)。

## 1. 搭建环境
*   **服务器**：这边使用的是莱卡云国内服务器，服务器配置没有要求，最低配即可。
*   **系统**：CentOS 7.9

## 2. FRP官网
更多信息请访问官网: [https://gofrp.org](https://gofrp.org)

## 3. 服务端配置 (公网IP服务器)

### 3.1 下载
根据你的环境下载，这边使用的是Linux系统，所以下载 `frp_0.60.0_linux_amd64` 的版本。
下载地址：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)

### 3.2 上传服务器并解压
```bash
# 打开/usr/local路径文件夹 
cd /usr/local

# 创建frp文件夹，并且进入 
mkdir frp && cd frp
```
把刚刚下载的FRP压缩包上传安装包到 `/usr/local/frp` 目录。
```bash
# 解压frp_0.60.0_linux_amd64.tar.gz文件，并进入解压后的目录
tar -zxvf frp_0.60.0_linux_amd64.tar.gz
cd frp_0.60.0_linux_amd64
```

### 3.3 配置
编辑`frps.toml`配置文件。
```bash
# 编辑frps配置文件 
vi frps.toml
```
进入`frps.toml`之后按`a`进入编辑模式，粘贴以下内容：
```toml
# 客户端与服务连接端口 
bindPort = 7000 
# 客户端连接服务端时认证的密码 
auth.token = "abcjc" 
# http协议监听端口 
vhostHTTPPort = 28080 
# web界面配置 
webServer.addr = "0.0.0.0" 
webServer.port = 7500 
webServer.user = "admin" 
webServer.password = "admin"
```
粘贴完之后保存编辑文件，按`Esc`然后输入 `:wq` 回车（强制保存退出）。

### 3.4 运行frps服务
为了方便管理，我们将`frps`创建为一个 `systemd` 服务，使其可以开机自启。

**创建 frps.service 文件**
在 `/etc/systemd/system` 目录下创建一个 `frps.service` 文件。
```bash
# 编辑/etc/systemd/system/frps.service文件 
sudo vi /etc/systemd/system/frps.service
```
进入`frps.service`之后按`a`进入编辑模式，粘贴以下内容：
```ini
[Unit] 
Description=frp server 
After=network.target syslog.target 
Wants=network.target 

[Service] 
Type=simple 
ExecStart=/usr/local/frp/frp_0.60.0_linux_amd64/frps -c /usr/local/frp/frp_0.60.0_linux_amd64/frps.toml 

[Install] 
WantedBy=multi-user.target
```
粘贴完之后保存编辑文件。

### 3.5 systemd 命令管理 frps 服务
```bash
# 启动frp 
sudo systemctl start frps 

# 设置开机自启
sudo systemctl enable frps

# 检查frp服务状态
sudo systemctl status frps

# 停止frp 
# sudo systemctl stop frps 

# 重启frp 
# sudo systemctl restart frps
```

## 4. 客户端配置 (没有公网IP的设备)
### 4.1 下载客户端
我下载的是 `frp_0.60.0_windows_amd64.zip`。
下载地址：[https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases)

### 4.2 本地解压
本地解压下载的压缩包。

<!-- 在这里插入图片: image-20240916155910986 -->

### 4.3 配置
编辑解压后文件夹中的配置文件 `frpc.toml`。

<!-- 在这里插入图片: image-20240916155942479 -->

写入内容并保存：
```toml
serverAddr = "服务器的公网IP" 
serverPort = 7000 
auth.token = "abcjc"

# HTTP的配置 
[[proxies]] 
name = "blog" 
type = "http" 
localIP = "127.0.0.1" 
localPort = 8080 
customDomains = ["abcj.cn"]

# TCP的配置 
[[proxies]] 
name = "rdp" 
type = "tcp" 
localIP = "127.0.0.1" 
localPort = 3389 
remotePort = 23389
```

### 4.4 运行
在客户端的 `frp` 文件夹中打开命令行工具 (CMD 或 PowerShell)，运行以下命令：
```bash
frpc.exe -c frpc.toml
```
回车即可。 
