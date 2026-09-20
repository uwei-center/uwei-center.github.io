---
layout: post
title: "Isaac Sim安装指南"
date: 2025-11-19 15:00:00
categories: [水下仿真环境开发]
tags: [水下机器人, IsaacSim]
---

## 一、前言
 
本文用于指导 IsaacSim 5.0 的安装及配置。截止当前，IsaacSim版本为5.1.0，但本篇中参照[OceanSim官方文档](https://github.com/umfieldrobotics/OceanSim/blob/main/docs/subsections/installation.md)的要求，使用 IsaacSim 5.0.0 进行搭建。

--- 
## 二、概览
项目和课题要搭建水下智能体，第一步就是要搭建水下环境专用的仿真平台以获取高保真数据，之前选择了Isaac Sim作为平台，原因可以参考[这篇笔记](/posts/review-underwater-robot-simulator)。关于水下仿真平台，核心部分可以分成三部分，如下图：![水下机器人仿真器框架](/assets/img/underwater-robot-simulator-framework.png)



OceanSim是用于搭建水下仿真环境的框架，整体项目基于Isaac Sim进行开发，作为Isaac Sim的插件使用。OceanSim基于GPU加速水下渲染，并提供了多种水下传感器模型，如DVL、相机、成像声纳等，尤其是声纳部分，使用NVIDIA的光线追踪（Ray Tracing, RT）技术来实现，使得效果更加逼真，计算更快。


这篇博客主要是指导IsaacSim的配置过程。


---
## 三、IsaacSim 5.0.0安装

### 1. 兼容性与需求检查
首先进入官网[Nvidia Isaac Sim 官网](https://developer.nvidia.com/isaac/sim)，里面有关于isaacSim的一些介绍，IsaacSim是基于Nvidia Omniverse构建的用于数据仿真、数据采集和机器人训练的仿真平台。

点击下面的[Download Isaac Sim](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/index.html)，

可以通过[IsaacSim 官方文档](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/quick-install.html)来指导安装，在左侧Installation下可以看到有Workstation Installation 和 Container Installation 两种方式，我们这里选择使用容器安装，关于Docker的配置与使用可以参考[这篇教程](/posts/docker-usage-example)，先点击Isaac Sim Requirements查看依赖项，里面会详细的说明5.0.0版本所需要的硬件配置![硬件依赖](/assets/img/isaac-sim-hardware-requirements.png)


可以输入以下命令查看本机硬件配置
```terminal
lscpu
free -h
nvidia-smi
```

> 注意：IsaacSim官方明确表示，**GPUs without RT Cores** (A100, H100) are not supported.     RT Cores也就是NVIDIA GPU中专门用来加速光线追踪计算的物理核心，如果GPU标号是RTX开头，说明就是具有RT Cores的GPU。

下方就是IsaacSim 5.0.0对于存储空间的要求![存储要求](/assets/img/isaac-sim-storage-requirements.png)大体满足即可。

尽管Isaac Sim 5.0.0版本要求的最低VRAM为16GB，大于我的电脑的3080 10GB，但我无视风险，继续安装（确实穷）。如果想一键查看是否满足配置，可以点击上方的Isaac Sim Compatibility Checker，下载检查器，下载完成后解压

```terminal
unzip isaac-sim-comp-check-5.0.0-linux-x86_64.zip -d isaac-sim-comp-check
```

来到isaac-sim-comp-check文件夹下，输入
```terminal
cd isaac-sim-comp-check
./omni.isaac.sim.compatibility_check.sh
```

可以看到结果为
![本地硬件](/assets/img/local-hardware-check.png)
似乎配置也够。

### 2.IsaacSim Container安装
下面正式开始安装IsaacSim，点击左侧栏[Container Installation](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/install_container.html#)

先安装Docker
```bash
# Docker installation using the convenience script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Post-install steps for Docker
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

# Verify Docker
docker run hello-world
```

安装NVIDIA Container Toolkits
```bash
# Configure the repository
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
    && \
    sudo apt-get update

# Install the NVIDIA Container Toolkit packages
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker

# Configure the container runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Verify NVIDIA Container Toolkit
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi

```

> Install the latest version of NVIDIA Container Toolkit to get security fixes. <br/> NVIDIA Container Toolkit 和 NVIDIA Driver 都向后兼容，所以尽可能下载更新版本的。


再次确认GPU驱动的版本符合要求
```bash
nvidia-smi
```

拉取官方镜像，这个过程比较慢，大概15GB
```bash
docker pull nvcr.io/nvidia/isaac-sim:5.0.0
```

查看是否成功拉取
```bash
docker images
```
如果在镜像列表中看到有tag为5.0.0的isaac sim则为成功。


### 3.IsaacSim Container运行
以可交互的bash session运行
```bash
docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \
    -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
    -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
    -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/documents:/root/Documents:rw \
    nvcr.io/nvidia/isaac-sim:5.0.0
```
可以将上述的内容复制到run_isaac-sim_container.sh脚本中，用于一键启动。
```bash
cat << 'EOF' > run_isaac-sim_container.sh
#!/usr/bin/env bash
set -e

docker run --name isaac-sim --entrypoint bash -it --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v ~/docker/isaac-sim/cache/kit:/isaac-sim/kit/cache:rw \
    -v ~/docker/isaac-sim/cache/ov:/root/.cache/ov:rw \
    -v ~/docker/isaac-sim/cache/pip:/root/.cache/pip:rw \
    -v ~/docker/isaac-sim/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/root/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/root/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/data:/root/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/documents:/root/Documents:rw \
    nvcr.io/nvidia/isaac-sim:5.0.0
EOF

./run_isaac-sim_container.sh
```

运行上面的内容，会发现已经进入到container中了，以headless方式运行IsaacSim
```bash
./runheadless.sh -v
```

会开始编译与运行，大概三分钟，看到终端中提示`Isaac Sim Full Streaming App is loaded.`时可以开始下一步LiveStream Client的配置与启动。


### 4.GUI与LiveStream图像转发

我们前面已经成功启动了IsaacSim的容器，并且以无头（headless）模式运行了IsaacSim，如果我们要用GUI进行场景的设置与查看，需要在run_isaac-sim_container.sh中添加DISPLAY设置转发，或者使用NVIDIA专用的[LiveStream](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/manual_livestream_clients.html)，Isaac Sim WebRTC流媒体客户端是推荐的流媒体客户端，可在桌面或工作站远程观看Isaac Sim，无需强大显卡。

但是，WebRTC要求:
> 具备NVENC（NVIDIA Encoder），如A100就没有这个硬件，所以当容器运行在A100上面，无法使用WebRTC进行流媒体转发。

我的RTX 3080满足，所以为选择使用WebRTC进行流媒体转发。

接下来，需要先说清什么是主机Host，什么是客户端Client。在我们这里，发起IsaacSim服务的机器就是Host，要通过WebRTC对IsaacSim进行访问的机器就是Client。

NVIDIA关于[WebRTC streaming client的设置](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/manual_livestream_clients.html)里面有写需要在运行`./runheadless.sh`时，将host的IP地址作为flag传入给shell脚本。我这里的环境是，Host机器与Client机器都在一个内网下，不需要公用IP地址，所以只需要确保Client能访问到Host机器的`TCP 49100`端口和`UDP 47998`端口，可以参考下面的示例脚本。
```python
# 检查TCP 49100端口 
# 在服务器的terminal中运行
nc -l -p 49100


# 在client端，写一个python脚本运行
import socket
s = socket.socket()
try:
    s.connect(("<服务端IP>", 49100))
    print("Connected!")
except Exception as e:
    print("Failed:", e) 

# 检查UDP 
# 在服务器的terminal中运行
nc -u -l 47998

# 在client端，写一个python脚本运行
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.sendto(b"hello", ("<服务端IP>", 47998))
print("sent")
```

如果分别输出`Connected!`与`hello`，则说明端口访问成功。

因此IsaacSim启动命令就要修改成下面的带参数版本了
```shell
./runheadless.sh --/app/livestream/publicEndpointAddress=$PUBLIC_IP --/app/livestream/port=49100
```
上面的`$PUBLIC_IP`输入内网IP即可。


去Client端，下载对应版本的[Isaac Sim WebRTC LiveStream Client](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/download.html#isaac-sim-latest-release)，然后输入上面的内网IP即可，效果如下图![webRTC客户端](/assets/img/isaac-sim-webRTC-livestream-client.png)

IsaacSim的GUI的部分就结束了，IsaacSim也算配置完成了。


关于IsaacSim的使用，请参考[这篇文章](/posts/IsaacSim-example-usage)

### 5.OceanSim的安装与配置



## 参考链接
1. [Isaac ROS安装指南，包含Sim安装，ROS安装，ZED等传感器调试](https://nvidia-isaac-ros.github.io/getting_started/index.html)

2. [Isaac Sim调试视频](https://www.bilibili.com/video/BV1uFGTzSEdz?spm_id_from=333.788.videopod.sections&vd_source=108bdf2b7083151030d235ef8674417e)

3. [OceanSim github](https://github.com/umfieldrobotics/OceanSim)

4. [Isaac Sim主页](https://developer.nvidia.com/isaac/sim)

5. [Isaac ROS ZED 安装](https://nvidia-isaac-ros.github.io/getting_started/hardware_setup/sensors/zed_setup.html)

6. [Isaac Sim下，SLAM采集数据完整搭建流程](https://zhaoxuhui.top/blog/2022/12/16/omniverse-and-isaac-sim-note5-isaac-sim-ros-api-for-image-and-pose-topic-publish.html)

7. [Isaac 环境入门-Bilibili](https://www.bilibili.com/video/BV1gBGTz9EBA?spm_id_from=333.788.videopod.sections&vd_source=108bdf2b7083151030d235ef8674417e)