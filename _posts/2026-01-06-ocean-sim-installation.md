---
layout: post
title: "OceanSim使用示例"
date: 2025-11-20 17:03:00
categories: [水下仿真环境开发]
tags: [水下机器人, OceanSim, IsaacSim]
---


## 一、前言
 
本文用于指导 OceanSim 的安装及配置。本篇中参照[OceanSim官方文档](https://github.com/umfieldrobotics/OceanSim/blob/main/docs/subsections/installation.md)的要求，使用 IsaacSim 5.0.0 进行搭建。并且针对OceanSim里关于传感器模型的实现进行一些分析。


## 二、构建OceanSim镜像
在[OceanSim官方文档](https://github.com/umfieldrobotics/OceanSim/blob/main/docs/subsections/installation.md)中，总体步骤为两步。

1. 下载OceanSim源码

```terminal
cd /path/to/isaacsim/extsUser
git clone https://github.com/umfieldrobotics/OceanSim.git
```

2. 下载OceanSim资产

```terminal
cd /path/to/OceanSim
python3 config/register_asset_path.py /path/to/OceanSim_assets
```

在之前的[IsaacSim安装](/posts/isaac-sim-installation)过程中，我们已经通过container方式成功安装了IsaacSim并通过WebRTC进行显示。只需要重新写一个Dockerfile来把Oceansim的源码和资产放进去build一下就可以了。

### 1. 写Dockerfile

```dockerfile
FROM nvcr.io/nvidia/isaac-sim:5.0.0

# 1. 工作目录
WORKDIR /isaac-sim

# 2. 拷贝 OceanSim extension
# 必须在 extsUser 下面
COPY OceanSim /isaac-sim/extsUser/OceanSim

# 3. 拷贝 OceanSim 资产
COPY OceanSim_assets /isaac-sim/extsUser/OceanSim/OceanSim_assets

# 4. 注册 OceanSim 资产路径
RUN /isaac-sim/python.sh \
    /isaac-sim/extsUser/OceanSim/config/register_asset_path.py \
    /isaac-sim/extsUser/OceanSim/OceanSim_assets
```


### 2. 构建新的镜像
注意要把资产和源码都放在和dockerfile同一个目录下，然后运行下面的构建。

```shell
docker build -t isaac-sim:oceansim .
```



### 3. 创建容器
```shell
#!/usr/bin/env bash
set -e

docker run --name isaac-sim-zsz -d --runtime=nvidia --gpus all -e "ACCEPT_EULA=Y" --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/cache/kit:/isaac-sim/kit/cache:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/cache/ov:/root/.cache/ov:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/cache/pip:/root/.cache/pip:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/cache/glcache:/root/.cache/nvidia/GLCache:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/cache/computecache:/root/.nv/ComputeCache:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/logs:/root/.nvidia-omniverse/logs:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/data:/root/.local/share/ov/data:rw \
    -v /home/isaac-dev/workspaces/isaac-sim/volumes/documents:/root/Documents:rw \
    isaac-sim:oceansim \
    sleep infinity
```




