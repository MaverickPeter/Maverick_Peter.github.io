---
layout:     post
title:      "UTS教授报告笔记"
subtitle:   " \"HRI项目分享\""
date:       2018-12-14 11:07:00
author:     "xxc"
header-img: "img/post-bg-HRI.png"
catalog: true
tags:
    - Deep Learning
    - HRI

---

# 报告学习

### JaIme (CAS - UTS)

#### Motivation - Pipes fail (metallic)
1. NDT sensing (do not break the pipes)
2. 3D laser cameras for GTruth

#### Sensing - Data Analytics Sensor modeling
Network: Simulations (Training data) + sensor data -> gt (finally prediction)

#### Robotic design for pipe inspection

#### Data Modelling - ATP Framework
Probablistic model

#### Project1: Assistive Robotics: Intelligent wheelchair - no map
Laser-based no map
* CONCEPT: Users drive the wheelchair with the help of assistive robotics: Shared control between the user and wheelchair (real-time)

#### Project2: Smart Hoist

#### Project3: Autonomous navigation in rugged terrains
* ZMP-based stable path planning

#### Continuous Mapping and exploration with Gaussian processes

#### Project4: Indoor people tracking with RF (BLE)



## HRI
Decoupled observation models (Learning mobility aid assistance)
* AIM: provide intelligent assistance to powered mobility devices
* Build model from disabled (experts)
1. expert users data collection
2. expert modelling: CNN heapmap & GP joystick heatmap model - JOYSTICK as a input not directly control the wheelchair
3. Assisting new users

### Assistive wheelchair (EXPLORATION)
#### Expert data collection
1. simulated 2D environment
2. record odom, LIDAR and joystick inputs
3. record safe joystick (collision avoidance)

#### CNN model (environment)
* Local free-space map -> likelihood
* Training data: free-space
* Goal: end point in local free-space -> likelihood map

#### User model (GP - Gaussian Processes)
* Add preknowledge more uncertainty in the GP model
* Distance filter

#### Planning - Qlearning (or any MDP path planning tool)
* Heatmap for Path Planning
* CNN heatmap + User heatmap + obstacle safety map