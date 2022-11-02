---
title: MAVROS与NX通讯
categories:
  - PX4
tags:
  - PX4
date: 2022-11-01 21:08:24
---

[toc]

[常用MAVROS话题和服务](https://zhuanlan.zhihu.com/p/364872655)

# 真机

## 串口

[通过MAVROS连接机载电脑（NANO/TX2/NX）与Pixhawk](https://zhuanlan.zhihu.com/p/364390798)

### 1.配置Pixhawk上的Telem2作为MAVLINK端口

==在QGC中设置参数==

- `MAV_1_CONFIG`= `TELEM 2`
- `MAV_1_MODE` = `Onboard`
- `SER_TEL2_BAUD` = `921600 8N1`

一开始参数里可能只有MAV_1_CONFIG，搜不到其他的参数，只需要先把MAV_1_CONFIG设置为TELEM 2，然后把飞控重启后就有了。

### 2.在机载电脑上启动MAVROS

我这里用的是jetson nano的串口2，也就是`dev/ttyTHS1`，这个按照自己实际情况写就好了。最后的921600是波特率，就是2.2.1中设置的`SER_TEL2_BAUD`，改成设置的值就行了。

```cpp
roslaunch mavros px4.launch fcu_url:=serial:///dev/ttyTHS1:921600 gcs_url:=udp://@192.168.0.0
```

`gcs_url`为运行QGC的主机的IP

设置为以下参数表示自动寻址，直到连上QGC

```cpp
roslaunch mavros px4.launch fcu_url:=serial:///dev/ttyTHS1:921600 gcs_url:=udp-b://@
```

可能会报错

```cpp
FCU: DeviceError:serial:open: Permission denied
```

解决方法是给对应的串口权限

```cpp
sudo chmod 777 /dev/ttyTHS1
```

**虽然这样子mavros就正常运行了，但是节点信息会卡在xxxxxx start xxxxxxx，然后/mavros/state中的connected是false。这个问题卡了我一天，后来我把波特率往下降到460800以下就能正常使用了。如果知道这是什么原因欢迎私信或评论，谢谢。（评论区有大佬指出是因为数据线太长，导致传输过程中到达不了那么高的波特率，所以不能成功连接）**

可能会报`timesync`的异常，解决方法是：

```cpp
sudo gedit /opt/ros/melodic/share/mavros/launch/px4_config.yaml
```

把第12行改为

```cpp
timesync_rate: 0.0
```

### 3.一劳永逸

每次要在后面加一堆参数很烦，所以直接修改`launch`文件，使用`sudo`权限打开`/opt/ros/melodic/share/mavros/launch/px4.launch`，将

```text
<arg name="fcu_url" default="/dev/ttyACM0:57600" />
```

修改为自己的端口和波特率

```text
<arg name="fcu_url" default="/dev/ttyTHS1:921600" />
```

### 4.运行外部控制节点

```cpp
rosrun offb_ctrl offb_ctrl_node
```





[[PX4开发指南-11.4.MAVROS外部控制例程](https://www.ncnynl.com/archives/201709/2078.html)

[[Pixhawk入门指南-电子围栏介绍](https://www.ncnynl.com/archives/202106/4307.html)](https://www.ncnynl.com/archives/202106/4307.html)





**PX4接入Jetson系列机载时，因为Jetson系列机载为ARM架构，没有对应的QGC地面站可以安装使用。在平时的连接都是通过将PX4接USB线连接至电脑，通过电脑的QGC进行各种校准和参数更改。接入机载的PX4可以通过数传进行连接电脑地面站，同时也可以通过wifi进行连接。**

[PX4机载连接通过IP连接电脑QGC地面站](https://blog.csdn.net/weixin_50060664/article/details/123909638)



## USB连接

PX4的microUSB串口通过数据线连接电脑的USB口，同时，电池给PX4板供电。在电脑中输入下命令检查：

```
ls /dev/ttyACM*
```

```
sudo chmod 777 /dev/ttyACM0 
roslaunch mavros px4.launch fcu_url:=/dev/ttyACM0
roslaunch mavros px4.launch fcu_url:=/dev/ttyACM0 gcs_url:=udp-b://@
```



## 通讯中需要注意的地方

### PX4固件更换

[px4固件版本](https://github.com/PX4/PX4-Autopilot/releases?page=3)

### MAVROS坐标系

FCU 使用 NED 框架，ROS 使用 ENU 框架。
Mavros 负责两帧之间的转换。
所以如果你想向 FCU 发送一个设定点，它必须在 ENU 帧中定义（X 向前和 Z 向上），Mavros 将在 NED 坐标中进行转换。
相反，当你想读取四旋翼的当前位姿时，你可以使用mavros/local_position/pose，mavros会做转换，这样你就可以读取ENU中的位姿

![](image-20221101205317106.png)

### 提高IMU发布频率

[提高mavros中IMU话题的发布频率](https://blog.csdn.net/qq_38649880/article/details/89419736)

使用`rostopic hz`查看imu发布频率

```
//原始数据
rostopic hz /mavros/imu/data_raw
//飞控计算过后的IMU数据
rostopic hz /mavros/imu/data
```

在新终端中输入下面命令

```
//原始数据
rosrun mavros mavcmd long 511 105 10000 0 0 0 0 0
//飞控计算过后的IMU数据
rosrun mavros mavcmd long 511 31 10000 0 0 0 0 0
```

==如果想要提到更高的频率只需要减小`10000`这个参数，这个就是设置时间间隔的现在间隔为10000us所以是100hz。==

> 待解决：如何集成到launch中

### 发布位置信息

使用`local_pos_pub.publish(pose);`

### 掩码设置

[PX4 offboard模式能接收的mavros指令（转载，如果同时发送期望位置和期望速度，是位置控制，速度作为前馈。现在才真正理清楚）](https://blog.csdn.net/sinat_16643223/article/details/120746386)

==谁被掩谁就是NaN，谁没被掩谁就被**px4**接受为setpoint==

就v1.11.3来说，掩码的使用还并不规范，并不是掩了就一定不接收，不掩就一定被接收，整个offboard下的接收setpoint是比较混乱的。可行的组合如下：

- 单独的p、v、acc是没问题的，但是要求x、y、z方向的同时给出。acc是转化为了四元数和归一化油门（读[PositionControl.cpp](https://hub.fastgit.org/PX4/PX4-Autopilot/blob/master/src/modules/mc_pos_control/PositionControl/PositionControl.cpp)，由悬停油门线性化估计）。给p、v时10-20Hz就行，给加速度或者角度时需要50-100Hz，否则不会按照期望的加速度来飞行的！
- p全给+v全给、pz+v全给。全给指x、y、z同时给出。总之v必须全给，如果不想给，就给前馈0。这其实是v1.11.3中mavlink_receiver**.cpp**和FlightTaskOffborad**.cpp**中掩码判断不对导致的。
- p全给+a全给、pz+a全给。总之a必须全给，如果不想给，就给前馈0。和v类似。反正高度环和yaw、yaw_rate是单独控制的，可以组合，但是x、y必须同时给出，有时也会影响到z。
- p全给+v全给+a全给。

### PX4 mavros可以切换的模式

```
//! PX4 custom mode -> string
static const cmode_map px4_cmode_map{{
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_MANUAL),           "MANUAL" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_ACRO),             "ACRO" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_ALTCTL),           "ALTCTL" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_POSCTL),           "POSCTL" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_OFFBOARD),         "OFFBOARD" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_STABILIZED),       "STABILIZED" },
	{ px4::define_mode(px4::custom_mode::MAIN_MODE_RATTITUDE),        "RATTITUDE" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_MISSION), "AUTO.MISSION" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_LOITER),  "AUTO.LOITER" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_RTL),     "AUTO.RTL" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_LAND),    "AUTO.LAND" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_RTGS),    "AUTO.RTGS" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_READY),   "AUTO.READY" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_TAKEOFF), "AUTO.TAKEOFF" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_FOLLOW_TARGET), "AUTO.FOLLOW_TARGET" },
	{ px4::define_mode_auto(px4::custom_mode::SUB_MODE_AUTO_PRECLAND), "AUTO.PRECLAND" },
}};
```



## 特别注意

1. 不要用程序切offboard模式！！！
2. 不要用程序切offboard模式！！！
3. 不要用程序切offboard模式！！！
4. 示例程序在循环里反复切offboard模式，要是访问不到机载电脑，关闭不了控制节点，那就一直会处于offboard模式。如果发送的指令还是速度控制指令就完了。所以设立电子围栏也很重要，至少砸到人的几率小了。
5. 建议用遥控器切换模式，如果程序出现问题立即用遥控器切换到自稳等其他模式。
6. 消息需要以大于2Hz的频率发送，否则PX4会切换回offboard之前的模式。
7. 切换到offboard需要位置信息，比如GPS提供的位置信息，在使用前要确保无人机在定点模式！
8. ROS中的坐标是ENU(东北天)，而PX4中是NED(北东地)。MAVROS已经对坐标进行了转换。

| ROS  | PX4  |
| ---- | ---- |
| X    | Y    |
| Y    | X    |
| Z    | -Z   |

