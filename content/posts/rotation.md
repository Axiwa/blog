---
title: Euler Angle - Quaternion - Axis Angle
categories: Computer graphics
date: 2023-04-02
---
三维旋转

<!--more-->

## Euler Angle
每次沿着X/Y/Z轴旋转一个角度，根据旋转的顺序（先绕X旋转后绕Y旋转），旋转轴是global还是local（轴的方向是固定的 or 随着物体转动），每次涉及到什么轴（X-Y-X or X-Y-Z）有12种转动方法。根据涉及到的轴的个数分为**Proper Euler Angle**（绕轴顺序可能为XYX,YXY,XZX,ZXZ,YZY,ZYZ）和**Tait-Brayan Euler Angle**（绕轴顺序可能为XYZ,XZY,YXZ,YZX,ZXY,ZYX）

### 外旋+右手系+YXZ
对于外旋，$Y_1X_2Z_3$实际上是先绕Z轴旋转$R_z()$，再绕X轴旋转$R_x()$，再绕Y轴旋转$R_y()$。如果是内旋，则是按照YXZ的顺序旋转

右手系的旋转矩阵:

`$$
\begin{aligned}
R_z(\gamma) = \begin{bmatrix}
\cos \gamma & -\sin \gamma & 0 \\
\sin \gamma& \cos \gamma & 0 \\
0 & 0 & 1\\
\end{bmatrix} \\
R_x(\beta) = \begin{bmatrix}
1 & 0 & 0\\
0 & \cos \beta & -\sin \beta \\
0 & \sin \beta & \cos \beta \\
\end{bmatrix} \\
R_y(\alpha) = \begin{bmatrix}
\cos \alpha & 0 & \sin \alpha \\
0 & 1 & 0 \\
-\sin \alpha & 0 & \cos \alpha\\
\end{bmatrix} 
\end{aligned}
$$`

如果我们需要旋转一个$\mathbf{x} = \begin{bmatrix} x \\ y \\ z \end{bmatrix}$ 的向量，则作用在它上面的旋转矩阵为()

`$$
R = R_yR_xR_z\mathbf{x} = \\
\left[\begin{array}{ccc}
c_1 c_3+s_1 s_2 s_3 & c_3 s_1 s_2-c_1 s_3 & c_2 s_1 \\
c_2 s_3 & c_2 c_3 & -s_2 \\
c_1 s_2 s_3-s_1 c_3 & s_1 s_3+c_1 c_3 s_2 & c_1 c_2
\end{array}\right]\mathbf{x}
$$`
其中$c_1 = cos(\alpha), s_1 = sin(\alpha)$...

维基百科: [https://en.wikipedia.org/wiki/Euler_angles](https://en.wikipedia.org/wiki/Euler_angles)

### 万向锁

