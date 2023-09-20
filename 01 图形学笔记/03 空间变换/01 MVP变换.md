相机变换
设定摄像机位置为 e ，垂直向上的向量为 t ，观察方向为 g 。则世界空间坐标变换到相机空间则需要做和相机相反的位移。

$$
\left[\begin{array}{cccc}
1 & 0 & 0 & -x_{e} \\
0 & 1 & 0 & -y_{e} \\
0 & 0 & 1 & -z_{e} \\
0 & 0 & 0 & 1
\end{array}\right]
$$

在右手坐标系中，我们约定相机看向世界空间的 -z 轴，并构建正交坐标系，得到 g x t。据此可以直接写出变换矩阵为

$$
\left[\begin{array}{cccc}
x_{\hat{g} \times \hat{t}} & x_{t} & x_{-g} & 0 \\
y_{\hat{g} \times \hat{t}} & y_{t} & y_{-g} & 0 \\
z_{\hat{g} \times \hat{t}} & z_{t} & z_{-g} & 0 \\
0 & 0 & 0 & 1 
\end{array}\right]
$$

对于世界空间坐标，其需要做逆变换，而旋转变换是正交变换，其逆矩阵就等于其转置矩阵，因此可以直接得出相机变换的旋转矩阵。

$$
\left[\begin{array}{cccc}
x_{g \times t} & y_{g \times t} & z_{g \times t} & 0 \\
x_{t} & y_{t} & z_{t} & 0 \\
x_{-g} & y_{-g} & z_{-g} & 0 \\
0 & 0 & 0 & 1
\end{array}\right]
$$

最终完整的观察矩阵可以化简为

$$
\left[\begin{array}{cccc}
x_{g \times t} & y_{g \times t} & z_{g \times t} & -e \cdot {(g \times t)} \\
x_{t} & y_{t} & z_{t} & -e \cdot \ t \\
x_{-g} & y_{-g} & z_{-g} & -e \cdot g \\
0 & 0 & 0 & 1
\end{array}\right]
$$

### 2.1.3 投影变换
设定近平面和远平面的数值 n, f 为正数。取垂直视场角 fov 的一半做正切，即最高参数 t 与最近参数 n 的比值。再通过屏幕宽高比即可得出最右参数 r 。

$$
\tan{\frac{fov}{2}} = \frac{t}{n}
\\
\\
r = t \times \frac{w}{h}
$$
#### 2.1.3.1 正交投影变换
View Space 在对称情形下，r$和 l ，t 和 b 可以相互抵消，并且实际上一般都是对称的。
正交投影变换可以分解为位移变换和缩放变换。
- 首先进行位移变换，将 View Space 移动到观察空间的坐标系原点。
- 然后再进行缩放变换，将 View Space 的长宽高变换为 [-1,1] 的范围，即标准立方体。
- 此时称为齐次裁剪空间。经过齐次裁剪，再对坐标进行透视除法，才变换到 NDC 空间。

$$
\left[\begin{array}{cccc}
\frac{2}{r-l} & 0 & 0 & 0 \\
0 & \frac{2}{t-b} & 0 & 0 \\
0 & 0 & \frac{2}{f-n} & 0 \\
0 & 0 & 0 & 1
\end{array}\right]
\left[\begin{array}{cccc}
1 & 0 & 0 & -\frac{r+l}{2} \\
0 & 1 & 0 & -\frac{t+b}{2} \\
0 & 0 & 1 & -\frac{f+n}{2} \\
0 & 0 & 0 & 1
\end{array}\right]
=
\left[\begin{array}{cccc}
\frac{2}{r-l} & 0 & 0 & -\frac{r+l}{r-l} \\
0 & \frac{2}{t-b} & 0 & -\frac{t+b}{t-b} \\
0 & 0 & \frac{2}{f-n} & -\frac{f+n}{f-n} \\
0 & 0 & 0 & 1
\end{array}\right]
$$

$$
\left[\begin{array}{cccc}
\frac{1}{r} & 0 & 0 & 0 \\
0 & \frac{1}{t} & 0 & 0 \\
0 & 0 & \frac{2}{f-n} & 0 \\
0 & 0 & 0 & 1
\end{array}\right]
\left[\begin{array}{cccc}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & -\frac{f+n}{2} \\
0 & 0 & 0 & 1
\end{array}\right]
=
\left[\begin{array}{cccc}
\frac{1}{r} & 0 & 0 & 0 \\
0 & \frac{1}{t} & 0 & 0 \\
0 & 0 & \frac{2}{f-n} & -\frac{f+n}{f-n} \\
0 & 0& 0 & 1
\end{array}\right]
$$

#### 2.1.3.2 透视投影变换

透视投影变换需要先将截锥体形状的 View Space 挤压成长方体形状的的 View Space 后，再进行正交投影变换。将该矩阵和正交投影矩阵左乘后即可得到完整的透视投影矩阵。


$$
M_{persp \rightarrow ortho}
=
\left(\begin{array}{cccc}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & -(n+f) & -nf\\
0 & 0 & 1 & 0
\end{array}\right)
$$

$$
M_{persp} 
=
\left(\begin{array}{cccc}
\frac{2 n}{r-l} & 0 & -\frac{r+l}{r-l} & 0 \\
0 & \frac{2 n}{t-b} & -\frac{t+b}{t-b} & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 f n}{f-n} \\
0 & 0 & -1 & 0
\end{array}\right)
=
\left(\begin{array}{cccc}
\frac{n}{r} & 0 & 0 & 0 \\
0 & \frac{n}{t} & 0 & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 f n}{f-n} \\
0 & 0 & -1 & 0
\end{array}\right)
$$

三维向量 (x, y, z)中的 z 分量代表了该坐标与相机的距离。在透视投影中物体是近大远小的，这意味着离相机越远即 z 分量越大，显得越小。即与 1/z 正相关。通过如下形式的 4 x 4 矩阵，我们可以将坐标的 z 分量保存到投影后的齐次坐标 w 中，因此 w 的几何含义就是投影的距离，当 w 增大时，坐标被拉伸，w 减小时，坐标被压缩。W 对三维坐标起到缩放的作用。远处的坐标的 w 也就是变换前的 z 是更大的，被压缩的更多，也就符合我们的近大远小的需求。
$$
\left[\begin{array}{llll}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 1 & 0
\end{array}\right]\left[\begin{array}{l}
2 \\
3 \\
4 \\
1
\end{array}\right]=\left[\begin{array}{l}
2 \\
3 \\
4 \\
4
\end{array}\right]
$$