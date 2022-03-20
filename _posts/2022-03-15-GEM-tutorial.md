---
layout:     post
title:      "GEM使用指南"
subtitle:   " \"Elevation Mapping Tutorial\""
date:       2022-03-15 15:00:00
author:     "xxc"
header-img: "img/GEM/GEM.png"
catalog: true
tags:
    - SLAM
    - 工作分享

---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

### Basic Knowledge:

GEM是我和潘一源师兄合作的一篇做建图的文章，全名叫GEM: Online Globally Consistent Dense Elevation Mapping for Unstructured Terrain
，旨在处理户外资源受限场景下带闭环的建图问题。局部地图是基于ETHZ的Elevation map的，当时测试下来，ETHZ的Elevation map是CPU实现的，不太适合实时性较强的移动机器人应用场景，ETHZ用在四足上，运动速度慢，所以基本可行。我们提出了一种实时可扩展的GPU实现的全局Elevation map. 在整个原始Elevation map的框架中，将部分可并行部分进行cuda加速，此外实现了ray tracing算法进行了实时障碍物消除。原始几何数据的基础上进行了可行域分析，并添加了相应的地图层，为地面移动机器人避障与导航提供了较为便捷的地图。

### Motivation for this blog

写这一篇blog主要是因为，这个Mapping太泛用了，我研究生接的移动机器人项目中，都会用到它，很多师弟师妹也在用。但是，他们用的时候总是会反馈给我好多问题，基本上都是配置和安装方面的问题。我自己用的比较熟，所以基本不会关注到这些问题，有点知识诅咒了。所以在此写一个教程，从头开始把GEM跑起来。

### Dependencies

- Ubuntu: 16.04-20.04
- ROS: kinetic-noetic(会有一些frame_id要不要加"/"的问题)
- Packages (以kinetic为例)
```
# apt
sudo apt install ros-kinetic-grid-map* ros-kinetic-octomap* ros-kinetic-costmap* ros-kinetic-nav*
# kindr
git clone https://github.com/ANYbotics/kindr.git
cd kindr && mkdir build && cd build
cmake ..
make && make install
```

### Install GEM

```
cd catkin_workspace/src
git clone https://github.com/ZJU-Robotics-Lab/GEM.git
git clone https://github.com/ZJU-Robotics-Lab/slam_msg.git
git clone https://github.com/ANYbotics/kindr_ros.git
cd ../
catkin_make
```

### Configuration

配置文件主要有两块，一个是local map的参数，一个是基础参数

1. elevation_maps/simple_demo_map.yaml
    ```
    length_in_x: 12.0         # 单位m，局部地图x方向大小
    length_in_y: 12.0
    position_x: 0.0           # 单位m，机器人在局部地图中的位置，中心为(0,0)
    position_y: 0.0
    track_point_x: 0.0        # 单位m，地图跟随点的位置，默认(0,0)
    track_point_y: 0.0
    resolution: 0.1           # 单位m，地图分辨率
    travers_threshold: 0.8    # 判断可通行的阈值，> travers_threshold 为可通行，范围(0~1)
    min_variance: 0.0001
    max_variance: 10000.0
    mahalanobis_distance_threshold: 2.5
    multi_height_noise: 0.00002 
    ```
2. robots/simple_demo_robot.yaml
    ```
    robot_id: "0"                           # 机器人id，用于多机器人系统
    robot_name: "robot0"                    # 机器人名字，用于多机器人系统，需在最后带robotid
    map_frame_id: "/robot0/odom"            # 机器人里程坐标系
    sensor_frame_id: "/PandarQT"            # 机器人传感器坐标系
    robot_base_frame_id: "/PandarQT"        # 机器人本体坐标系
    track_point_frame_id: "/PandarQT"       # 机器人跟随点的坐标系，一般与机器人本体坐标系保持一致
    robot_pose_cache_size: 200
    robot_local_map_size: 20                # 子地图大小
    octomap_road_resolution: 0.2            # 可通行区域octomap分辨率
    octomap_obs_resolution: 0.1             # 不可通行区域octomap分辨率
    map_saving_file: "./map.pcd"            # 地图存储路径 (optional)
    submap_saving_dir: "./submaps/"         # 子地图存储路径 (optional)
    camera_params_yaml: "./param.yaml"      # 相机内参以及激光与相机外参路径，绝对路径 (required)
    orthomosaic_saving_dir: "./image/"      # 正视图存储路径 (optional)
    ```

### FAQ
1. catkin_make报错 
   ```"/usr/local/cuda-10.1/include/crt/common_functions.h:74:24: error: token ""CUDACC_VER is no longer supported. Use CUDACC_VER_MAJOR, CUDACC_VER_MINOR, and CUDACC_VER_BUILD instead."" is not valid in preprocessor expressions #define CUDACC_VER "CUDACC_VER is no longer supported. Use CUDACC_VER_MAJOR, CUDACC_VER_MINOR, and CUDACC_VER_BUILD instead." 
   ```
  - 解决方案：找到/usr/local/cuda-10.1/include/crt/common_functions.h:74 这一行，注释掉。
  
2. ERROR: Wrong path to settings 
  - 解决方案：elevation_mapping/elevation_mapping_demos/config/robots/simple_demo_robot.yaml中的camera_params_yaml需要修改为绝对路径

3. 进不去回调函数
  - 因为GEM默认是激光+相机的，因此相机和激光需要时间戳对齐，这里如果不想用图像，又不想折腾纯激光版本。可以复制一下下面的代码到fake_img.py，用python发一个假的img。
   
    - ```
      #!/usr/bin/env python
      #coding:utf-8

      import rospy
      import sys
      sys.path.append('.')
      import cv2
      import os
      import numpy as np
      from sensor_msgs.msg import Image
      from cv_bridge import CvBridge, CvBridgeError

      def pubImage():
        rospy.init_node('pubFakeImage',anonymous = True)
        pub = rospy.Publisher('robot_1/image_rect', Image, queue_size = 10)
        rate = rospy.Rate(10)
        bridge = CvBridge()

        while not rospy.is_shutdown():
            image = np.zeros((640,480,3), np.uint8)
            msg = bridge.cv2_to_imgmsg(image,"bgr8")
            msg.header.stamp = rospy.Time.now()
            pub.publish(msg)
            rate.sleep()

      if __name__ == '__main__':
        try:
            pubImage()
        except rospy.ROSInterruptException:
            pass
      ```
4. 建图非常慢
  - GPU虽然并行计算快，但是招架不住输入的点云太大，单帧点云控制在10w点以下比较好，建议用仓库里的filter，先滤一下点云。
  - 如果点云非常大，我见过realsense的深度图出的点云，单帧962000个点，filter一下都要好几百毫秒，那建图也没戏。