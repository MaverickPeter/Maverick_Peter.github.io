---
layout:     post
title:      "MR_SLAM使用指南"
subtitle:   " \"MR_SLAM Tutorial\""
date:       2022-11-04 12:00:00
author:     "xxc"
header-img: "img/MR_SLAM/dog_demo.jpg"
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

[MR_SLAM](https://github.com/MaverickPeter/MR_SLAM)是用来验证我们最近提出的基于拉东变换的地点重识别方法[RING](https://arxiv.org/abs/2204.07992)/[RING++](https://arxiv.org/abs/2210.05984)的多机器人SLAM系统。在RING++的文章中，我们测试了该系统在数据集上的效果，同时也将其应用到了多个项目中，验证了实际应用的可行性。MR_SLAM我认为是比较模块化的系统，我自己有替换过里程计、地点重识别以及后端优化算法，代码上由于我自己水平有限，还有很多可以提升的地方，如果有意向一起改进的话，可以github上提PR，也可以联系我(xuechengxu@zju.edu.cn)。


### Motivation for this blog

有位同学在github上的repo给我提issue，我发现整套系统用起来还是需要一个教程，同时对于自定义开发，也没有文档支持，这边就想通过这个blog来讲一下整个系统的逻辑。

### Dependencies

- Ubuntu: 16.04-18.04 (20.04 还未测试)
- ROS: kinetic-melodic
- PyTorch：> 1.6.0 (fft module is modified after 1.6.0)

这里PyTorch中的fft模块在1.6.0版本之后修改了接口，disco可能还有部分rfft是<1.6.0版本需要注意。其实要改也很好改，就在models/DiSCO.py里面把torch.rfft()这个函数用新的替代掉就好了。

### A Quick Use

基本的安装步骤和用法在github repo上有涵盖。
此处更详细地介绍一下不同参数的设置要求：
在定位FAST_LIO中，我加入了一个LIO_Publisher来发布一些必要信息，包括tf,submap以及keyframe的信号。这些信息会被mapping以及global_manager用到。

LIO_Publisher的参数有以下几个：
```
    <node pkg="fast_lio" type="my_node" name="LIO_Publisher_client1" output="screen">
      <param name="RobotID" type="int" value="1" />
      <param name="NameSpace" type="string" value="robot_1" /> 
      <param name="dis_th" type="double" value="1.5" />
    </node>
```
Mapping中的elevation_mapping_periodical在原本的[GEM](https://github.com/ZJU-Robotics-Lab/GEM)基础上加入了发布局部地图的service，用于global_manager周期性订阅。其中参数基本与GEM保持一致，可以参考[GEM-Tutorial](https://maverickpeter.github.io/2022/03/15/GEM-tutorial/)

当前有两种发送submap的方式，一种是通过elevation_mapping进行转发，另一种是在FAST_LIO的LIO_Publisher.cpp中，根据行驶距离发送单帧/累积点云，建议使用LIO_Publisher中的，因为elevation_mapping可以不用，但定位基本系统都是需要的。

global_manager是主要处理后端和地图维护的部分，launch里参数有这些：
```
  <node pkg="global_manager" type="global_manager_node" respawn="false" name="global_manager" output="screen">
    
    <param name="global_map_frame" value="map"/>  # 全局地图坐标系
    <param name="robot_namespace" value=""/>   
    <param name="logger_level" value="Info"/>   # 日志级别

    <!-- bool -->
    <param name="publish_tf" value="true"/> # 是否发送tf
    <param name="loop_closure_enable" value="true"/>    # 是否开启回环检测
    <param name="use_other_descriptor" value="true"/>   # 使用其他回环描述子
    <param name="use_refined_initials" value="true"/>   # 使用精匹配后的位姿(默认使用精匹配后的位姿)
    <param name="odometry_loop_enable" value="false"/>  # 使用里程计回环
    <param name="loop_detection_debug" value="false"/>  # 回环检测debug
    <param name="manual_robots_config" value="true"/>   # 手动配置机器人初始位姿，默认全在原点出发
    <param name="enable_elevation_mapping" value="true"/>   # 是否构建2.5D高度地图
    
    <!-- string  -->
    <param name="submap_topic" value="submap"/> # 子地图topic, 子地图为自定义topic，保存有位姿和点云
    <param name="pose_graph_topic" value="graph"/>  # 位姿图发布的topic
    <param name="merged_elevation_map_topic" value="merged_map"/>   # 全局2.5D高度地图的topic
    <param name="merged_point_cloud_topic" value="merged_cloud"/>   # 全局3D点云地图的topic
    <param name="descriptor_topic" value="disco"/>  # 接收的描述子topic,仅当使用粗匹配的位姿(默认使用disco)
    <param name="keyframe_pc_topic" value="merged_keyframe"/>   # 用于回环检测的keyframe topic (debug)

    <!-- file and dir  -->
    <param name="manual_config_dir" value="$(find global_manager)/cfg/real/"/>  # 手动设置的机器人初始位姿的文件夹地址
    <param name="elevation_map_saving_filename" value="$(find global_manager)/log/globalElevationMap.pcd"/>     # 存储2.5D高度地图的位置
    <param name="global_map_saving_filename" value="$(find global_manager)/log/globalMap.pcd"/>    # 存储3D点云地图的位置
    <param name="pg_saving_filename" value="$(find global_manager)/log/full_graph.g2o"/>    # 存储位姿图的位置
    <param name="keyframe_saving_dir" value="$(find global_manager)/log/Keyframes/"/>       # 存储keyframe的位置

    <!-- specific parameters -->
    <param name="start_robot_id" value="1"/>    # 第一台机器人的id,(目前id需要连续),第一台机器人的初始坐标系为多机器人系统的地图坐标系
    <param name="composing_rate" value="3"/>    # 全局地图更新频率
    <param name="tf_publish_rate" value="10"/>  # tf更新频率
    <param name="disco_dim" value="1024"/>      # disco描述子的维度
    <param name="disco_width" value="120"/>     # disco频谱的width
    <param name="disco_height" value="40"/>     # disco频谱的height
    <param name="loop_detection_rate" value="0.1"/>     # 回环检测的频率
    <param name="pose_graph_pub_rate" value="1"/>       # 位姿图更新频率
    <param name="DiSCO_dist_thres" value="17.0"/>       # disco粗判断的阈值

    <!-- icp params: registration_method choose from PCL_ICP, PCL_GICP, FAST_GICP, FAST_VGICP_CUDA-->
    <param name="registration_method" value="FAST_VGICP_CUDA"/>     # 配准方法选择,可选PCL_ICP, PCL_GICP, FAST_GICP, FAST_VGICP_CUDA
    <param name="submap_size" value="1"/>       # 用于配准的keyframe数量, 0表示仅用当前keyframe, 1表示拼接当前与前后1帧的keyframe
    <param name="icp_iters" value="50"/>        # icp迭代次数
    <param name="icp_fitness_score" value="0.10"/>      # icp匹配分数,越小越好,但是回环也会减少
    <param name="icp_filter_size" value="0.2"/>         # icp配准时点云降采样的分辨率
    <param name="submap_voxel_leaf_size" value="0.2"/>  # 子地图存储时点云降采样的分辨率
    <param name="globalmap_voxel_leaf_size" value="0.5"/>   # 发布全局地图时点云降采样的分辨率

    <param name="keyframe_search_candidates" value="10"/>   # 寻找地点候选的数量，仅在粗匹配使用disco时用到
    <param name="keyframe_search_radius" value="8"/>        # 寻找当前位置附近多少米范围内历史轨迹中可能的回环,仅在粗匹配使用disco且开启里程计回环时用到
  </node>
```

### Modification Guidance
#### Localization
定位基本上是在现有的激光里程计上加一个必要信息的publisher和robot_name的定义。

在FAST_LIO上的修改是单独写了一个LIO_Publisher来处理信息并整合发出，同时在原有的FAST_LIO里加入了robot_name字段，这部分自己加robot_name是为了开发更灵活，其实应该也可以使用namespace。

**必要的信息包括**：带robot_name的tf，根据距离判断的submap

#### LoopDetection
回环检测目前提供三种方法：DiSCO/RING/ScanContext
- DiSCO - LoopDetection/src/disco_ros: 这部分在[DiSCO-Tutorial](https://maverickpeter.github.io/2022/03/20/DiSCO-tutorial/)中有介绍，在该仓库中的main.py扩展到多机器人场景，并将disco与icp结合，直接估计6DoF位姿，为mapping提供精匹配的位姿。
- RING - LoopDetection/src/RING_ros: 这部分的安装配置请参考该目录下的README文件，其中main.py包含了基于RING方法的回环检测，其中也包含了icp进行精匹配。
- ScanContext - LoopDetection/src/RING_ros: ScanContext在，其中main.py包含了基于RING方法的回环检测，其中也包含了icp进行精匹配。

回环检测部分目前有个问题，就是回环和global_manager都存了一份keyframe的数据，会对内存有一定影响。目前想法是通过service的方法连接global_manager和回环，进行keyframe的取用。

**如果要扩展使用自定义的回环检测方法，可以参考RING的main.py，其中主要的部分为回环的消息类型Loop_msgs。**

#### Mapping
地图中包含两个部分：1、单机2.5D高度地图，2、多机地图融合

1. 单机2.5D高度地图 - Mapping/src/elevation_mapping_periodical：主要针对地面移动机器人导航规划的使用。详情可以参考[GEM](https://github.com/ZJU-Robotics-Lab/GEM)和[GEM-Tutorial](https://maverickpeter.github.io/2022/03/15/GEM-tutorial/)，该仓库加入了当前局部地图的订阅机制，地图融合模块会定时收取各机器人的当前局部地图，以保证全局2.5D高度地图的实时性。当然，这个模块是可以不开启的，global_manager.launch中有相关的开关配置。
2. 多机地图融合 - Mapping/src/global_manager：**该仓库中最重要的部分。**它收集各机器人的单机信息以及回环检测的结果，进行全局位姿图构建及优化。涉及到以下几个线程：
   1. DiscoveryThread：发现ros中的机器人并注册对应的handler 
   2. MapComposingThread：子地图拼接成全局地图的线程
   3. LoopClosureThread：回环检测结果处理，回环优化的线程
   4. 各种信息发布的线程：
      - TFPublishThread：发布TF
      - PoseGraphPublishThread：发布位姿图
      - GeometryCheckPublishThread：发布icp匹配的中间结果(debug)

##### DiscoveryThread
DiscoveryThread 主要在ROS topic中找到所有机器人并初始化对应的handler。
一个机器人的handler包含以下几个部分：
```
// Define robot handler
typedef struct RobotHandle {
  // protects map
  int robot_id;
  bool initState;     // !deprecated
  std::mutex mutex;   // lock this robot when update variables
  SubMapVec submaps;  // submaps get from GEM
  TrajVec trajectory;   // history trajectory
  KeyFrameVec keyframes;  // keyframe pointcloud
  PointCloudConstPtr map; // map constructed by submaps
  std::vector<int> disco_index; // disco index used in retrieval in kdtree
  OrthoVector ortho_image;  // not used for this version
  PointCloudI lastKeyframe; // for keyframe pointcloud enhance
  PointCloudI keyframePC; // keyframe pointcloud
  TimeStampVec timestamps;  // timestamp

  DiSCOVec disco_base;  // disco database
  DiSCOFFTVec disco_fft;  // disco fft database
  ros::Subscriber map_sub;  // subscriber for submap
  ros::Subscriber disco_sub;  // subscriber for disco
  ros::Publisher keyframe_pub;  // publisher for keyframe viz
  ros::ServiceClient init_map_client_;  // local map services
  std::string robot_name;
  float transformLast[6];   // used in pose graph formation, TODO: remove it by using data in trajectory
  float transformCurrent[6];    // used in pose graph formation
  float transformAftMapped[6];  // used in pose graph formation
  float transformTobeMapped[6]; // used in pose graph formation
  
  pcl::PointCloud<pcl::PointXYZI>::Ptr cloudKeyPoses3D; // used in some trival place, TODO: remove it
  pcl::PointCloud<PointTypePose>::Ptr cloudKeyPoses6D;  // used in some trival place, TODO: remove it
}robotHandle_;
```

##### MapComposingThread
子地图的局部信息均在robotHandler中，包括其自身坐标系下的历史轨迹与子地图。地图拼接模块根据回环优化后的位姿，将各机器人handler中对应的子地图拼接到全局地图中。

##### LoopClosureThread
回环线程中包含了基于disco的内嵌回环检测(早期测试版本，后期会移除，不建议使用)，以及回环信息处理的部分。接收回环检测模块的结果，将回环加入到维护的全局位姿图中，并将完整的位姿图交给correctPoses()函数进行优化。目前内嵌有基于GTSAM开发的[dist-mapper](https://github.com/CogRob/distributed-mapper)以及GTSAM中自带的Gauss-NewtonOptimizer和LevenbergMarquardtOptimizer，不过还没加入config进行选择配置。

global_manager中有很多地方会考虑到异步的情况，还需要理一理。
