齐次空间裁剪（Homogeneous Clipping Space）可以更细粒度地将视锥体外的三角形剔除。坐标点经过透视投影变换后其 w 值为变换前的 z 值，再经过透视除法后才被变换到[-1,1]×[-1,1]×[0,1]的 NDC 空间。那么在透视除法前则可以得知坐标的取值范围，以此进行齐次裁剪。
$$
(\frac{x'}{w},\frac{y'}{w},\frac{z'}{w})
$$
$$
-w<x'<w,
-w<y'<w,
0<z'<w
$$
透视投影变换中 z 分量代表了该坐标与相机的距离，这意味着离相机越远即 z 分量越大，显得越小。
而投影变换矩使得 w 分量保留变换前的 z 分量，如果有顶点在近平面后，则导致 w'<0，进行透视除法时会使得其 xy 坐标翻转， 
![](Pasted%20image%2020230723163629.png)

如果顶点恰好在视点平面上，则会导致除 0 错误。在透视除法前将不符合以上条件的顶点剔除、裁剪后可以避免除零错误。
对于有顶点落在视点平面 z=0 的三角形，要么直接剔除，要么进行裁剪。避免出现除零错误。将视锥体上的三角形裁剪为新三角形。但裁剪新三角形的操作是比较麻烦的事情，在光栅化时通过视口和平面方程来裁剪超出屏幕范围的部分会更容易。

#### 2.1.4.1 NDC
NDC 空间的坐标是由齐次裁剪空间坐标经过透视除法得到的。根据投影矩阵我们可以得知相机空间和裁剪空间的关系，例如 w_c = z。
相机空间坐标的 z 经过透视投影变换后，被存在齐次裁剪空间坐标的 w 中。经过透视除法后我们可以得到 NDC 空间的深度。观察这个关系式，我们可以看出 NDC 空间坐标的深度，也就是屏幕空间的深度，是与相机空间坐标的 z 成反比的。
$$
\left[
\begin{array}{c}
\cdot \\
\cdot \\
z_{c} \\
w_{c}
\end{array}\right]=\left[\begin{array}{llll}
\cdot & & & \\
& \cdot & & \\
& b & a \\
& 1 &
\end{array}\right]\left[\begin{array}{l}
\cdot \\
\cdot \\
z \\
1
\end{array}\right]
$$


$$
d 
= \frac{z_{c}}{w_{c}}
= \frac{a+b z}{z} 
= a \frac{1}{z}+b
$$
如果要从 NDC 坐标反推世界空间坐标，直觉上我们应该先把 NCD 坐标乘以 w 得到裁剪空间坐标，再经过逆 VP 变换得到世界空间坐标。但由于透视除法是硬件自动执行的，我们想要获取这一数值则需要顶点着色器向片元着色器输出。
齐次裁剪空间坐标是由相机空间经过投影变换后得到的，因此我们可以得到相机空间坐标和 NDC 空间坐标的关系。已知投影变换之前 $\text{w}=1$，即 NDC 坐标经过逆 VP 变换后得到的 $\text{w}$ 与 $\text{w}_{\text{clip}}$ 相乘等于1。

$$
V_{\text {view }}  =M_{\text {project }}^{-1} * V_{\text {ndc }} * w_{\text {clip }} 
$$
$$
1  =\left(M_{\text {project }}^{-1} * V_{\text {ndc }}\right).w * w_{\text {clip }} 
$$

经过观察发现除了 $\text{w}_{clip}$ 其他项都是已知的，将 $\text{w}_{clip}$ 的表达式代回原式得到世界空间坐标。

$$
v_{\text{view}} = M_{\text{project}}^{-1} *   v_{\text{ndc}} * \frac{1}{(M_{\text{project}}^{-1} \cdot  v_{\text{ndc}}).w}
$$
$$
1  =\left(M_{\text {VP }}^{-1} * P_{\text {ndc }}\right).\text{w} * \text{w}_{\text {clip }}
$$

视口变换的目的是将 NDC 空间的三维坐标变换到屏幕空间的二维坐标。根据图形 API 的不同，所规定的屏幕空间的坐标系也不同。以 OpenGL 为标准，我们构建将 NDC 空间的左下变换到原点的变换矩阵。齐次坐标的点坐标经过视口变换后截取 x, yx, yx, y 坐标即为其在屏幕空间下的坐标。
$$
M_{\text {viewport }}=\left(\begin{array}{cccc}
\frac{\text { width }}{2} & 0 & 0 & \frac{\text { width }}{2} \\
0 & \frac{\text { height }}{2} & 0 & \frac{\text { height }}{2} \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{array}\right)
$$
