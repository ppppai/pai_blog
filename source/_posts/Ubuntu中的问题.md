---
title: Ubuntu中的问题
categories:
  - Ubuntu
tags:
  - Ubuntu
  - Python
date: 2022-10-19 13:52:28
---

## Ubuntu 升级 python 版本

### PX4 v1.10.0 版本在编译时需要升级python版本才能编译成功

Ubuntu 默认安装了python2.7

输入命令`python`，可以查看当前python版本

`Ctrl+D`退出，pyhton命令行

输入以下命令，升级python

```
sudo add-apt-repository ppa:jonathonf/python-3.6
sudo apt-get update
sudo apt-get install python3.6
```

调整优先级：

```
//调整Python3的优先级，使得3.6优先级较高

sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 1

sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 2

//更改默认值，python默认为Python2，现在修改为Python3

sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 100

sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150
```

再输入`python`查看python版本