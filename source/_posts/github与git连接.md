---
title: github与git连接
categories:
  - null
tags:
  - ssh
date: 2022-10-05 15:02:27
---

# Github添加公钥

## 第一步

`ssh-keygen -t rsa -C "注册GitHub使用的邮箱@email.com" `

一路按Enter

## 第二步：复制公钥

`vim .ssh/id_rsa.pub
或
cat .ssh/id_rsa.pub`

或者直接打开`.ssh`文件夹里的`id_rsa.pub`文件复制

## 第三步：到Github上添加公钥

## 最后一步：检查是否添加成功

执行命令:

`ssh -T git@github.com`

得到下面所示时，表示调加成功

`Hi ppppai! You've successfully authenticated, but GitHub does not provide shell access.`

---

## 注意：

检查时会提示报错`Connection closed by remote host`，一般是网络问题，需要：

### 关闭全局代理软件

v2rayA只有全局代理，所以使用的时候需要关闭

### 也有可能是使用的梯子封禁了Github端口22的连接

### 解决：

将 Github 的连接端口从 22 改为 443 即可。修改 `~/.ssh/config` ，添加如下段落即可

```sh
Host github.com
    HostName ssh.github.com
    User git
    Port 443
```

### 验证：

```sh
$ ssh -T git@github.com

Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```











