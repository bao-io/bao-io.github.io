---
title: 搭建vpn服务器
date: 2023-07-27
duration: 10min
category:
  - Docker
  - Linux
---

### VPN 原理

`VPN`其实原理就是通过一些协议，例如**PPTP**、**L2TP**等协议建立两个终端的通道，使得两个端点之间可以相互进行操作的行为。那么我们平时的`梯子`也就是我们手机或者电脑通过 VPN 连接国外的服务器，因为国外的服务器可以正常访问国外的网站。所以一旦连接成功的话，我们的连接设备也就拥有了访问国外网站的能力啦～～～

### Software 代理软件

有了 vpn 服务，我们还需给我们的设备安装代理软件，尽管我们手机或者电脑自带 vpn 的代理配置，但有些高级 VPN 配置是系统不支持的。所以我们得下载一些代理软件来连接我们的 VPN 服务。市面上有开源免费的`Clash`，没错就是那只猫 😂

![](/images/b3f7cebd-372c-4553-b90a-753a3df98f39.webp)

但他只提供电脑端。手机端用的比较多的就是`Shadowrocket`，但只有非大陆的苹果账号才能下载。那么以下分享会以 Clash 为例子教大家如何搭建 VPN 服务器。

### 订阅地址

一般代理商都会提供，下载之后就会得到各个节点的配置信息，这里就不做推广了，有兴趣可以自己百度了解一下。

### Clash

首先 clash 很多客户端，其中用的比较多的是一个仓库名为`clash_for_windows_pkg`的开源库，里面发布了很多版本的 GUI 界面。

![](/images/7c251092-53c5-4c10-812a-cbf72e16e0a3.webp)

但我们既然要搭建 VPN 的服务器，那么必然就要首先 pass 掉这种 GUI 的界面，我们需要的是用命令行去执行 VPN 的代理。所以我要需要用到他们的官方库，仓库地址为：`https://github.com/Dreamacro/clash`，点击 releases 中查看，我们可以发现有很多操作系统的版本。一般服务器都是 centos 的 x86 架构，我们直接选择`linux-386`下载即可。

![](/images/6723df89-8b3f-479f-96c0-127f3c543adb.webp)

1. 下载完解压会发现里面就是一个二进制文件。我们需要在终端赋予我们执行的权限

```bash
chmod -R 777 ./clash-linux-386-v1.17.0
```

2. 下载订阅地址对应的配置文件
   我们可以直接用命令行下载

```bash
wget "订阅链接" -O subscription.yaml
```

也可以直接从 GUI 上右键打开配置文件的文件夹获取

![](/images/8feb6d43-630b-45a9-992a-2c4df929f828.webp)

3. 然后就是输入以下命令直接启动 vpn 啦

```bash
./clash -f subscription.yaml ## ./clash -d .
```

- `-f`: 指定配置文件
- `-d`: 指定配置文件所在的目录

### webui

官网作者(go 语言大佬)考虑到命令行终端操作的便捷性，拉了前端大佬写了一个 react 前端项目。仓库在`https://github.com/Dreamacro/clash-dashboard`。我们如果想对服务器上的 vpn 代理节点进行便捷切换的话，这个还是非常有必要安装的。具体使用如下。

```bash
git clone https://github.com/Dreamacro/clash-dashboard.git && pnpm i && pnpm build
```

执行完之后会看到一个 dist 目录，这就是构建之后的产物，我们需要在上面订阅配置文件加入三行配置即可启动 weiui 界面。

```yaml
external-controller: 0.0.0.0:9090
external-ui: /app/clash-dashboard/dist
secret: 123
```

- `external-controller`: webui 服务启动在哪个端口
- `external-ui`: 构建产物位置在哪
- `secret`: 访问 webui 的密钥
  配置完之后，我们直接上面配置的`external-controller`地址。

就是可以看到如下界面啦，我们可以在上面直接切换 vpn 代理节点。

![](/images/564b99bc-679b-4d57-884e-1c9a92c92de6.webp)

### Docker 部署

clash 也是支持**docker 部署**的，以下是一个 docker-compose.yaml 的配置文件，仅供参考...

```yaml
version: '3'
services:
  clash:
    image: ghcr.io/dreamacro/clash
    restart: always
    # 默认7890代理端口
    ports:
      - 7890:7890
    # 挂在vpn代理配置文件，一般代理商都会给你订阅地址，直接下载即可
    volumes:
      - $HOME/.config/clash:/root/.config/clash/
```

**`clash 启动`**

```bash
docker-compose up -d
```

### 系统服务

clash 也支持在 linux 中以系统服务的方式进行控制管理。以下为一个系统服务的配置文件。

```bash
[Unit]
Description=Clash daemon, A rule-based proxy in Go.
After=network-online.target

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/clash -d /etc/clash

[Install]
WantedBy=multi-user.target
```

> 请注意，这个文件是在`/etc/systemd/system`文件夹中创建才生效，以及确保上面的 ExecStart 执行命令的路径是否和你的一致。

然后执行一下命令

```bash
## 开始守护进程
systemctl daemon-reload
## 注册clash服务
systemctl enable clash
## 开启clash服务
systemctl start clash
```
