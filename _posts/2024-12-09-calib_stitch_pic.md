---
title: 图像在裁剪和缩放后的内参变化
author: lyoid
date: 2024-12-09 14:10:00 +0800
categories: [Algo]
tags: [Calibration]
render_with_liquid: false
---
# 问题
图像裁剪是深度学习训练必须的一步，这是由于高分辨率影像与现在算力不足的gpu、npu之间的矛盾。


# 原理

## 问题假设
假设相机的内参是：

fx = 2220，fy = 2220

cx = 1404，cy = 1064

k1 = -0.00344584，k2 = 0.120934

相机原始分辨率：2880 x 2160

公式化型：
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
 1 & 0 & k_1 & k_2 & 0  \\
 0 & 1 & k_1 & k_2 & 0  \\
 0 & 0 & 0 & 0 & 1
\end{bmatrix}
*
 \begin{bmatrix}
 x \\
y \\
r^2 \\
r^4 \\
1
\end{bmatrix}$$
$$\begin{bmatrix}
 u  \\
 v  \\
 1
\end{bmatrix}
=
 \begin{bmatrix}
 f_x & 0 & f_x * k_1 & f_x*k_2 & c_x  \\
 0 & f_y & f_y*k_1 & f_y*k_2 & c_y  \\
 0 & 0 & 0 & 0 & 1
\end{bmatrix}
*
 \begin{bmatrix}
 x \\
y \\
r^2 \\
r^4 \\
1
\end{bmatrix}$$
## 图像裁剪
问题：

如果我们以图像中心为基准，将图像裁剪到分辨率：1920x1080，那么该图像对应的内参是？

计算：

由于围绕中心裁剪并不影像像素到达中心点的距离。所以裁剪仅影像像素坐标系的位置变换。
于是新的图像中心点

$$\begin{bmatrix}
 u_0 - 2880/2  \\
 v_0 - 2160/2 \\

\end{bmatrix}
=
\begin{bmatrix}
 u_1 - 1920/2  \\
 v_1 - 1080/2 \\

\end{bmatrix}
$$

$$
\begin{bmatrix}
 u_1   \\
 v_1  \\
1\\
\end{bmatrix}=
\begin{bmatrix}
 u_0\\
 v_0\\
1\\
\end{bmatrix}
-
\begin{bmatrix}
 480  \\
 540 \\
0
\end{bmatrix}
=
\begin{bmatrix}
1 & 0 & -480 \\
0 & 1 & -540\\
0 & 0& 1\\
\end{bmatrix}
*\begin{bmatrix}
 u_0\\
 v_0\\
1\\
\end{bmatrix}$$

于是对于裁剪后的图像，公式为：

$$\begin{bmatrix}
 u  \\
 v  \\
 1
\end{bmatrix}
=
\begin{bmatrix}
1 & 0 & -480 \\
0 & 1 & -540\\
0 & 0& 1\\
\end{bmatrix}*
\begin{bmatrix}
 f_x & 0 & c_x\\
 0 & f_y & c_y  \\
 0 & 0 & 1
\end{bmatrix} 
*
 \begin{bmatrix}
 1 & 0 & k_1 & k_2 & 0  \\
 0 & 1 & k_1 & k_2 & 0  \\
 0 & 0 & 0 & 0 & 1
\end{bmatrix}
*
 \begin{bmatrix}
 x \\
y \\
r^2 \\
r^4 \\
1
\end{bmatrix}$$

结论：

图像裁剪仅影响 c_x, c_y。

## 图像缩放
问题：

如果我们需要将上述裁剪后的图像缩放到分辨率：640x360，那么该图像对应的内参是？

计算：

$$\begin{bmatrix}
 u_1   \\
 v_1  \\
1\\
\end{bmatrix}=
\begin{bmatrix}
 1/3 & 0 & 0   \\
 0 & 1/3 & 0  \\
 0  & 0 & 1 \\
\end{bmatrix}*
\begin{bmatrix}
 u_0   \\
 v_0  \\
1\\
\end{bmatrix}$$
 
直接将这个倍数矩阵乘上公式：

$$\begin{bmatrix}
 u  \\
 v  \\
 1
\end{bmatrix}
=
\begin{bmatrix}
1/3 & 0 & 0 \\
0 & 1/3 & 0\\
0 & 0& 1\\
\end{bmatrix}*
\begin{bmatrix}
 f_x & 0 & c_x -480 \\
 0 & f_y & c_y -540 \\
 0 & 0 & 1
\end{bmatrix} 
*
 \begin{bmatrix}
 1 & 0 & k_1 & k_2 & 0  \\
 0 & 1 & k_1 & k_2 & 0  \\
 0 & 0 & 0 & 0 & 1
\end{bmatrix}
*
 \begin{bmatrix}
 x \\
y \\
r^2 \\
r^4 \\
1
\end{bmatrix}$$

结果：

$$ \begin{bmatrix}
 f_x \\
 f_y \\
 c_x\\
 c_y \\
 k_1 \\
 k_2 \\
\end{bmatrix}=
 \begin{bmatrix}
 1/3*f_x \\
 1/3*f_y \\
 1/3*(c_x - 480)\\
 1/3*(c_y - 540) \\
 k_1 \\
 k_2 \\
\end{bmatrix}$$

