---
​---
title: PX4+ROS开发
categories:
  - PX4
tags:
  - PX4
  - ROS
date: 2022-10-17 21:34:55

​---

[参考博客](https://www.cnblogs.com/cporoske/p/11641477.html)

[视频教程](https://www.bilibili.com/video/BV1ib4y1q7DJ/?spm_id_from=333.999.0.0&vd_source=c580acb208d0cf11f651aa5bf3869e1c)

### 注意：路径要用自己的路径

## offboard控制

### 1.创建工作空间

[参考](https://blog.csdn.net/weixin_42237429/article/details/90238000)

### 2.创建文件

在`catkin_ws/src`目录中，运行命令

```c++
catkin_create_pkg offboard_pkg roscpp std_msgs geometry_msgs mavros_msgs
```

然后定位到目录`~/catkin_ws/src/offboard_pkg/src/`，新建一个文件`offboard_node.cpp`。

```
touch offboard_node.cpp
```

打开文件使用`gedit`命令

将代码复制进去(官方示例):

```
/**
 * @file offb_node.cpp
 * @brief Offboard control example node, written with MAVROS version 0.19.x, PX4 Pro Flight
 * Stack and tested in Gazebo SITL
 */

#include <ros/ros.h>
#include <geometry_msgs/PoseStamped.h>
#include <mavros_msgs/CommandBool.h>
#include <mavros_msgs/SetMode.h>
#include <mavros_msgs/State.h>

mavros_msgs::State current_state;
void state_cb(const mavros_msgs::State::ConstPtr& msg){
    current_state = *msg;
}

int main(int argc, char **argv)
{
    ros::init(argc, argv, "offb_node");
    ros::NodeHandle nh;

    ros::Subscriber state_sub = nh.subscribe<mavros_msgs::State>
            ("mavros/state", 10, state_cb);
    ros::Publisher local_pos_pub = nh.advertise<geometry_msgs::PoseStamped>
            ("mavros/setpoint_position/local", 10);
    ros::ServiceClient arming_client = nh.serviceClient<mavros_msgs::CommandBool>
            ("mavros/cmd/arming");
    ros::ServiceClient set_mode_client = nh.serviceClient<mavros_msgs::SetMode>
            ("mavros/set_mode");

    //the setpoint publishing rate MUST be faster than 2Hz
    ros::Rate rate(20.0);

    // wait for FCU connection
    while(ros::ok() && !current_state.connected){
        ros::spinOnce();
        rate.sleep();
    }

    geometry_msgs::PoseStamped pose;
    pose.pose.position.x = 0;
    pose.pose.position.y = 0;
    pose.pose.position.z = 2;

    //send a few setpoints before starting
    for(int i = 100; ros::ok() && i > 0; --i){
        local_pos_pub.publish(pose);
        ros::spinOnce();
        rate.sleep();
    }

    mavros_msgs::SetMode offb_set_mode;
    offb_set_mode.request.custom_mode = "OFFBOARD";

    mavros_msgs::CommandBool arm_cmd;
    arm_cmd.request.value = true;

    ros::Time last_request = ros::Time::now();

    while(ros::ok()){
        if( current_state.mode != "OFFBOARD" &&
            (ros::Time::now() - last_request > ros::Duration(5.0))){
            if( set_mode_client.call(offb_set_mode) &&
                offb_set_mode.response.mode_sent){
                ROS_INFO("Offboard enabled");
            }
            last_request = ros::Time::now();
        } else {
            if( !current_state.armed &&
                (ros::Time::now() - last_request > ros::Duration(5.0))){
                if( arming_client.call(arm_cmd) &&
                    arm_cmd.response.success){
                    ROS_INFO("Vehicle armed");
                }
                last_request = ros::Time::now();
            }
        }

        local_pos_pub.publish(pose);

        ros::spinOnce();
        rate.sleep();
    }

    return 0;
}

```

然后打开目录`~/catkin_ws/src/offboard_pkg/`下的`CMakeLists.txt`添加下面的两行：

```
add_executable(offboard_node src/offboard_node.cpp)
target_link_libraries(offboard_node ${catkin_LIBRARIES})
```

然后到`catkin_ws`下，运行命令

```c++
catkin_make
//博客原文写的是 catkin build 原因是作者在创建空间的时候使用的是 build 
```

### 3.运行仿真

等待编译完成后，如果你要在`gazebo`中仿真，在 PX4 中运行命令

```bash
make px4_sitl gazebo_iris
```

打开`QGroundControl`。

然后在终端下运行命令：

```bash
roslaunch mavros px4.launch fcu_url:="udp://:14540@127.0.0.1:14557"
```

启动`PX4`与`Mavros`之间的连接，然后在`catkin_ws`运行命令

```bash
catkin_make
source ~/.bashrc
rosrun offboard_pkg offboard_node
```

然后进入`gazebo`中进行观察。
---
