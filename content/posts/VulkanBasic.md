---
title: Vulkan学习(1) Basics
categories: Computer graphics
date: 2023-04-02
---
照着 [https://vulkan-tutorial.com/](https://vulkan-tutorial.com/)画三角形

<!--more-->


### 为什么我画出来的东西只有固定角度才看得到
画了一个正方体，每次只有转到固定角度才会看到旁边的4个面，原来rasterizer的剔除模式设置的是背面剔除，但我的面的normal是反的... 

### 为什么我装了nvcc和nvidia-driver可是Vulkan还是检测不到
因为新版的driver安装已经不需要加`--no-opengl-files`了！
