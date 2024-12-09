---
title: 自动驾驶中的单目测距
author: lyoid
date: 2024-12-09 14:10:00 +0800
categories: [Algo]
tags: [Calibration]
render_with_liquid: false
---
# 问题
已知相机内外参数，求图像中地面一点的像素，对应的实际距离。

# 原理

已知像素坐标系转像空间坐标系如下：

$$\begin{bmatrix}
 u  \\
 v  \\
 1
\end{bmatrix}
=
\begin{bmatrix}
 f_x & 0 & c_x\\
 0 & f_y & c_y  \\
 0 & 0 & 1
\end{bmatrix} 
*
 \begin{bmatrix}
 X_w/Z_w  \\
 Y_w/Z_w  \\
 1
\end{bmatrix}$$

抽取第二行公式：

$$v = f_y * Y_w/Z_w + c_y$$

分析已知条件：
- 相机高度Y_w工厂标定给出
- 内参f_y,c_y 模组厂给出。
- v由当前感知估计得到在像素坐标系中的位置。

由此可以计算出Z_w相机相距物体的直线距离（像空间坐标系下，如果需要vcs坐标系下，需要再乘以外参）。
