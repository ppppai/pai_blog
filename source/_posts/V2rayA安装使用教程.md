---
title: V2rayA安装使用教程
categories:
  - null
tags:
  - V2rayA
date: 2022-10-04 14:25:10

---

[原文地址](https://zhuanlan.zhihu.com/p/414998586)

在安装ROS和其它很多的网站时，代理能方便许多，因此写了一个初步的教程。

在这个网站里，有着很详细的安装步骤。

[v2rayA](https://link.zhihu.com/?target=https%3A//v2raya.org/)

防丢初步准备总结一下：

## 安装：

## 1.安装 V2Ray 内核：

**v2rayA 提供的镜像脚本（推荐）**

```
curl -Ls https://mirrors.v2raya.org/go.sh | sudo bash
```

**安装后可以关掉服务，因为 v2rayA 不依赖于该 systemd 服务**

```
sudo systemctl disable v2ray --now ### Xray 需要替换服务为 xray
```

### 方法一：通过软件源安装

### 添加公钥

```
wget -qO - https://apt.v2raya.mzz.pub/key/public-key.asc | sudo apt-key add -
```

### 添加 V2RayA 软件源

```
echo "deb https://apt.v2raya.mzz.pub/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
```

### 安装 V2RayA

```sudo apt install v2raya
sudo apt install v2raya
```

### 方法二：手动安装 deb 包

[下载 deb 包](https://link.zhihu.com/?target=https%3A//github.com/v2rayA/v2rayA/releases)

后可以使用 Gdebi、QApt 等图形化工具来安装，也可以使用命令行：

```
sudo apt install /path/download/installer_debian_xxx_vxxx.deb ### 自行替换 deb 包所在的实际路径
```

## 启动：

## 1.启动 v2rayA/ 设置 v2rayA 自动启动

> 从 1.5 版开始将不再默认为用户启动 v2rayA 及设置开机自动。

**启动 v2rayA**

```
sudo systemctl start v2raya.service
```

**设置开机自动启动**

```
sudo systemctl enable v2raya.service
```

## 2.开始设置

通过 2017 端口 如 [http://localhost:2017](https://link.zhihu.com/?target=http%3A//localhost%3A2017/)访问 UI 界面。

（如果无法访问，请检查你的服务是否已经启动，[相关问题](https://link.zhihu.com/?target=https%3A//github.com/v2rayA/v2rayA/issues/237)）

## 3.创建账号

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-e50d8d16f746ea596f3f157ca4e0b_a65d520723a74dc98.jpg)

在第一次进入页面时，你需要创建一个管理员账号，请妥善保管你的用户名密码，如果遗忘，使用`sudo v2raya --reset-password`命令重置。

## **4.导入节点**

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-748385ba1e2960d44d51e80502d5e_9732ade20c1944d3a.jpg)

以创建或导入的方式导入节点，导入支持节点链接、订阅链接、扫描二维码和批量导入等方式。

## **5.连接节点和启动服务**

### 连接节点

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-14c6d4ee3dd29d78374f296afbedd_baf42fa9b4e74b2ea.jpg)

导入成功后，节点将显示在 `SERVER` 或新的标签中。如图是导入了一个订阅后的界面。

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-91fac87fa097a634f67b0bec2eac1_2b6310dcc01248dfa.jpg)

切换到该标签页，选择一个或多个节点连接。这里不建议选择过多的节点，6 个以内为佳。

### 启动服务

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-990905d7a51d7a2a553feba1d5e9a_e1243bee7f4b4435b.jpg)

在未启动服务时，连接的节点呈现柚红色。我们在左上角点击相应按钮启动服务。

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-3c3e839830071779a65579afd0042_aa05120f8b8b4ee19.jpg)

在启动服务后，所连接的节点呈现蓝色，左上角的图标也显示为蓝色的正在运行，代表服务启动成功

## 代理设置：

### 配置代理

由于默认情况下 v2rayA 会通过核心开放 20170(socks5), 20171(http), 20172(带分流规则的http) 端口。修改端口可参阅[后端地址和入站端口设置](https://link.zhihu.com/?target=https%3A//v2raya.org/docs/manual/address-port/) 一节。

如果是需要为局域网中的其他机器提供代理，请在设置中打开“局域网共享”，并检查防火墙开放情况。这里记录三种方式使用代理。

### 1.透明代理

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-01b4470eaac3b927397e58d109bec_818c7844a3e649f8b.jpg)

这种方法是 v2rayA 推荐的方法。它相比于其他方法具有诸多优势，v2rayA 可以一键开启透明代理，为**几乎所有程序**提供代理服务。

在设置中选择透明代理的分流方式，以及实现方式，然后保存即可。具体细节可参阅[透明代理](https://link.zhihu.com/?target=https%3A//v2raya.org/docs/manual/transparent-proxy/) 一节。

注意，如需选择 GFWList，需要下载对应的规则库，请点击右上角的更新以完成下载。

### 2.系统代理

系统代理可为**主动支持代理的程序**提供代理服务。在不同的桌面环境中设置的位置不尽相同，请通过搜索引擎自行搜索。

### 3.浏览器代理

SwitchyOmega 等浏览器插件可为**浏览器**提供代理服务。

在所使用的浏览器上安装插件

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-1dae8e342775d8f26ca4fff259e8a_9936d7fa162d40cea.jpg)

按照下图进行配置，并应用配置

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-784b84e5cbee84525097367285899_9b50cb057bef49f49.jpg)

在v2rayA右上角的设置中查看端口信息

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-9bfbd08bbeee4e6b5d8f3d3090119_aa2cc44ecfe64fb3b.png)

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-b422e3287271a93308e9489b43f0a_a7427a9d56b54dc7a.jpg)

在浏览器代理软件中，设置代理方式

![](../../../AAAAAAAAAAAAAAAAAAAAA/_resources/v2-9a81104a1cb1b5a0446a11b6ad7df_c9f470ad1b414e488.jpg)

## **4.linux客户端代理**

```
export https_proxy=https://127.0.0.1:20171
```