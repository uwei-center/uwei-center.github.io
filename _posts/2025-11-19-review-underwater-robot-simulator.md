---
layout: post
title: "水下机器人仿真平台综述"
date: 2025-11-19 16:07:00
categories: [水下仿真环境开发]
tags: [水下机器人, 仿真平台, 综述]
---

# 前言
最近开始一个项目，关于水下具身智能方向的，但是实际上我更习惯叫水下智能体，总感觉具身智能有点太超前了。众所周知，水下一个非常难做的点在于没有数据，导致陆地上很多前沿的技术没办法用到水下，最近看了点自动驾驶和无人机方面的论文，发现他们也是在对应的仿真器上面在采集数据，如：

- 自动驾驶**CARLA**，基于UE [CARLA github主页](https://github.com/carla-simulator/carla)

- 无人机**AirSim**，基于UE [AirSim github主页](https://github.com/microsoft/AirSim)


使用仿真器的好处至少包括：

1. 仿真器可以提供各种传感器（相机、Lidar、IMU）的真值数据，能够提供参考。

2. 场景也是布置的，所以也能为一些特定任务提供真值标签，如SLAM、3D重建等。

3. 可以以极低的成本获得大量的数据，更不用提水下采集数据的困难度和采到数据的可用性了。


于是我就去看了看水下有没有类似的仿真环境，一搜还真不少，国内在做的就有浙大，开始的应该还挺早的，早在24年他们就开始做水下的微调了，还有MerineGym这种专门给水下强化学习用的环境，也是搭配IsaacSim和IsaacGym来使用的。所以就有了这篇博客，大概描述下截至到2025年11月，国内外的水下环境仿真器现状。


---

## 综述
现有Simulators的综述：[review of Underwater simulator](https://arxiv.org/pdf/2504.06245)

1. **Isaac Sim**里面能实时渲染（Nvidia Omniverse），具有物理仿真（PhysX），以及某些传感器模型（IMU），但是渲染和动力学部分在水下需要重构。


2. **OceanSim**，基于Isaac-sim，针对水下渲染进行修改，添加多传感器模型（相机、声呐、DVL）。<br/>	
Github主页：[umfieldrobotics/OceanSim: [IROS 2025] OceanSim: A GPU-Accelerated Underwater Robot Perception Simulation Framework](https://github.com/umfieldrobotics/OceanSim)


3. **MarineGym**，基于Isaac-sim，有对于水动力模型的实现，以及集成了IsaacGym的RL方法的实现，还实现了悬停、圆形跟踪、螺旋跟踪、双纽线跟踪和着陆的gym基准测试。<br/>	[Marine-RL/MarineGym: [IROS 2025] MarineGym: A High-Performance Reinforcement Learning Platform for Underwater Robotics](https://github.com/Marine-RL/MarineGym)


4. **HoloOcean**，基于UE引擎开发，多传感器（DVL、IMU、相机、成像声呐、侧扫声呐、深度传感器），里面有声呐模型，OceanSim的声呐模型也是基于HoloOcean修改的。基于Fossen的水下动力学，通信，OpenAI gym API，环境模型。<br/>	**Github主页**：[byu-holoocean/HoloOcean: HoloOcean marine robotics simulator developed and maintained by the Field Robotics Systems lab at BYU.](https://github.com/byu-holoocean/HoloOcean)<br/>	**教程**：[Installation — HoloOcean 2.2.2 documentation](https://byu-holoocean.github.io/holoocean-docs/develop/usage/installation.html)

    不过HoloOcean 2.0马上也要出了。<br/>**论文：**[A Preview of HoloOcean 2.0](https://arxiv.org/pdf/2510.06160)

5. **Stonefish**里面有流体动力学计算以及水下环境渲染，使用C++实现，可作为独立程序，也可使用ROS与其他程序结合使用。<br/>	[patrykcieslak/stonefish: Stonefish - an advanced C++ simulation library designed for (but not limited to) marine robotics.](https://github.com/patrykcieslak/stonefish?tab=readme-ov-file)


6. **DAVE**仿真器，基于Gazebo，支持ROS，包含机械臂和各类传感器模型（声呐、水下激光雷达、相机、DVL、USBL），含有水动力学模型，海底地形和洋流的参数化表示。<br/>	[DAVE 官方主页](https://field-robotics-lab.github.io/dave.doc/)<br/>	[DAVE github主页](https://github.com/Field-Robotics-Lab/DAVE)


7. **UNav-Sim**仿真器，基于UE5开发，适配ROS2，具备水下动力学与机器人模型（推进器），传感器模型（IMU，距离传感器，相机）<br/>	[UNav-Sim仿真器 github主页](https://github.com/open-airlab/UNav-Sim)<br/>	[UNav-Sim Arxiv论文](https://arxiv.org/pdf/2310.11927)


8. 其他领域的开源仿真器架构参考<br/>	如自动驾驶**CARLA**，基于UE<br/>	[CARLA github主页](https://github.com/carla-simulator/carla)<br/>	无人机**AirSim**，基于UE<br/>	[AirSim github主页](https://github.com/microsoft/AirSim)


调研完这些，我觉得大部分框架已经有现成的了，剩下的就是如何在IsaacSim下面把这些东西综合起来。大体思路是，把Isaac Sim5.1和MarineGym插件还有OceanSim插件结合起来。再基于Oceansim，参考uuv-simulator、stonefish、Gazebo中一些实现额外扩充一些传感器组件，如侧扫声纳，缆绳动力学等。

正好手上有一个配置还可以的Ubuntu机器（RTX 3080），后面就打算用这个机子来试试上面几个平台了，后面再基于它们进行修改，最终开发一套可以用于项目的仿真环境搭建的平台。