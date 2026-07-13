---
title: NVLabs/diffpure库的代码复现思路
date: 2026-07-13
cover: /img/cover_for_md2.jpg   
tags: 
- 图像净化
- 扩散模型
categories: 
- 视觉处理
- 图像压缩
---

# 针对NVLabs/diffpure库的代码复现思路

## 1. 代码库的架构介绍
---
README.md文件中介绍了diffpure库的架构，这个库的核心给带有恶意攻击扰动的图像注入少量噪声（Forward 过程），然后利用扩散模型的逆向生成过程（Reverse 过程）重新生成干净的图像，从而“洗掉”对抗扰动，然后再丢给分类器。

这个项目对算力要求较高。
硬件要求： 1~4 张高规格 NVIDIA GPU（显存建议 32 GB，如 V100、A100、A30、A40 等。如果用 Colab 的 T4，可能需要调小 Batch size 防止 OOM）。

系统环境： Python 3.8 (64位)，安装了 CUDA 11.0 驱动。

容器化支持： 必须安装 Docker，因为官方直接提供了环境镜像。
关于环境部署问题，如果有docker环境，可以直接使用docker镜像，能够避免繁琐的CUDA适配和环境依赖报错。
```
# 1. 编译 Docker 镜像（会根据项目中的 dockerfile 自动配好所有环境）
docker build -f diffpure.Dockerfile --tag=diffpure:0.0.1 .

# 2. 启动容器（分配 GPU、共享内存 8G、挂载当前代码目录到容器的 /workspace）
docker run -it -d --gpus 0 --name diffpure --shm-size 8G -v $(pwd):/workspace -p 5001:6006 diffpure:0.0.1

# 3. 进入容器的终端，后续的实验命令都在这个终端里执行
docker exec -it diffpure bash
```