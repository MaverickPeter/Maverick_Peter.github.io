---
layout:     post
title:      "Elevation Mapping (GPU version)"
subtitle:   " \"工作记录\""
date:       2020-01-21 15:00:00
author:     "xxc"
header-img: "img/post-bg-elevation-mapping.jpg"
catalog: true
tags:
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

### Elevation Mapping介绍
考虑到地面移动机器人移动时避障与导航的需求，基于ETHZ的Elevation map，提出了一种实时可扩展的GPU实现的Elevation map. 在整个原始Elevation map的框架中，将部分可并行部分进行cuda加速，此外实现了ray tracing算法进行了实时障碍物消除。原始几何数据的基础上进行了可行域分析，并添加了相应的地图层，为地面移动机器人避障与导航提供了较为便捷的地图。
![elevation-architecture](/img/in-post/post-elevation-mapping/post-architecture.png)

### Elevation 模块 - 地图融合
该建图模块接受前端定位的位姿输出。位姿信息用于更新局部地图原点与融合多帧点云。
其中包括多帧点云融合以及局部地图维护更新是在GPU中进行加速。

### Elevation 模块 - Ray tracing
原始Elevation mapping由于并没有进行并行加速，因此实时性不好，从而会出现障碍物残影。在原始Elevation map上加入光线追踪方法，GPU加速的光线追踪方法可以实现实时的障碍物检测。
![ray-tracing](/img/in-post/post-elevation-mapping/post-ray-tracing.png)

### 实验
实验平台如下，采用激光里程计。
![experiment](/img/in-post/post-elevation-mapping/post-experiment.png)

实验结果：
下图左图为原始Elevation map在PC上的效果，中间为GPU加速的方法在TX2上的效果，右图为GPU加速的方法在PC上的效果
![experiment-map](/img/in-post/post-elevation-mapping/post-map.png)
![experiment-table](/img/in-post/post-elevation-mapping/post-table.png)
![experiment-drivable](/img/in-post/post-elevation-mapping/post-drivable.png)
