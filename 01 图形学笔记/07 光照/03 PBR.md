渲染方程描述了在半球上积分后着色点 p 在半球面上接收到的 Irradiance 经过反射分布函数的转化后，在出射方向上的 Radiance。
$$
L_{o}\left(p, \omega_{o}\right)=L_{e}\left(p, \omega_{o}\right)+\int_{\Omega^{+}} L_{i}\left(p, \omega_{i}\right) f_{r}\left(p, \omega_{i}, \omega_{o}\right)\left(n \cdot \omega_{i}\right) \mathrm{d} \omega_{i}
$$
## 

BRDF 的定义是出射 Radiance 的微分与入射 Irradiance 的微分之比。
$$
R\left(\mathbf{k}_{i}\right)=\int_{\text {all } \mathbf{k}_{o}} \rho\left(\mathbf{k}_{i}, \mathbf{k}_{o}\right) \cos \theta_{o} d \sigma_{o}
$$
![](Pasted%20image%2020230802103850.png)
### Ideal Diffuse BRDF
漫反射项采用 Lambertian 光照模型计算，在该模型中假设入射光时均匀分布整个半球体，对于所有的观看角度，表面将具有相同的亮度。其中余弦在半球作用域内的积分结果为 pi。

$$
R(k_i)= \int_{\text{all} \space k_o}{C{cos\theta_i}d\sigma_o}=\pi C
$$

根据能量守恒 R=1，因此理想漫反射的常数 BRDF 应为
$$
\rho\left(\mathbf{k}_{i}, \mathbf{k}_{o}\right)=\frac{r}{\pi}
$$
### 镜面反射项
Cook-Torrance BRDF 中，微表面上发生的镜面反射需要考虑决定镜面发射和折射发生比例的菲涅尔项 $F_r(\omega_o,\omega_i)$ 、表示每单位面积、每单位立体角中，所有法线方向为 $h$ 的微表面的面积的法线分布函数 $D(\omega_h)$ 和表达光线路径中的阴影遮挡比例的的几何项 $G(\omega_o,\omega_i)$。

$D(\omega_h)$ 的定义是每单位面积、每单位立体角上所有法线方向为 $h$ 的微表面的面积。类比 Radiance 的定义 $L={d^2\Phi}/({d\omega dA^{\perp})}$，可知所有法线方向为 $h$ 的微表面面积 $A(\omega_h)$ 是与 $\Phi$ 类似的，因此可得公式：
$$
d^2A(\omega_h) = D(\omega_h)d\omega_hdA
$$

由 Irradiance 的定义 $E = {d\Phi}/{dA}$，可得入射光在微分面积 $dA$ 上的辐射通量。所有法线方向为 $h$ 的镜面反射，其入射方向和出射方向都是确定的，因此对微表面入射 Irradiance 的辐射通量 $d\Phi$ ，在入射方向单位立体角上微分、在法线方向为 $h$ 的单位立体角上再次微分。
$$
d\Phi = L_icos\theta_idA
$$

$$
d^3\Phi_i = L_i(\omega_i)d\omega_i  d^2A^{\perp}(\omega_h)
$$

$$
= L_i (\omega_i) d\omega_i cos\theta_h D(\omega_h) d\omega_h dA
$$

暂时不考虑菲涅尔项和阴影遮挡项，则可以视入射和出射的辐通量没有变化。由 Radiance 的定义可知其为辐射通量 $\Phi$ 在单位面积、单位立体角上的微分的两次微分后的辐射通量，对 Radiance 在所有法线方向为 $h$ 的单位立体角上再次微分，由 BRDF 的定义可得
$$
d^3\Phi_o = d^3\Phi_i
$$

$$
dLo(\omega_o)  = \frac{d^3\Phi_o}{d\omega_o  dA^{\perp}}=\frac{ L_i (\omega_i) d\omega_i cos\theta_h D(\omega_h) d\omega_h dA}{d\omega_o  cos\theta_o dA }
$$


$$
f_r(\omega_o,\omega_i)= 
\frac{dL_o{(\omega_o)}}{dE_i(\omega_i)}
=\frac{ L_i (\omega_i) d\omega_i cos\theta_h D(\omega_h) d\omega_h dA}{L_i(\omega_i){cos\theta_i}d\omega_id\omega_o  cos\theta_o dA }=
\frac{cos\theta_h D(\omega_h) d\omega_h}{cos\theta_id\omega_o cos\theta_o}
$$

已知出射方向的角度是半程向量方向的角度的二倍 $\theta_o=2\theta_h$，由微分立体角的定义 $d\omega = sin\theta d\theta d\phi$ 可推出二者的比值为：

$$
d\omega_o = sin2\theta_h d2\theta_h d\phi = 4sin\theta_h cos\theta_h d\theta_h d\phi
$$
$$
\frac{d\omega_h}{d\omega_o} = \frac{1}{4cos\theta_h}
$$

将比值结果带入 BRDF 后化简可得：
$$
f_r(\omega_o,\omega_i)=\frac{ D(\omega_h)}{4cos\theta_i cos\theta_o}
$$
此时加入菲涅尔项 $F_r(\omega_o)$，几何可见项 $G(\omega_o,\omega_i)$ 和可得 BRDF 为：
$$
f_r(\omega_o,\omega_i)=\frac{D(\omega_h)F_r(\omega_o)G(\omega_o,\omega_i)}{4 cos\theta_o cos\theta_i}
$$
考虑到物体的自发光，将漫反射项和镜面反射项的 BRDF 线性混合后，乘上对应的漫反射系数、镜面反射系数 $k_d,k_s$ 后可得完整的渲染方程为：
$$
L_o(p,\omega_o)=L_e(p,\omega_r)+\int_{\Omega} {(k_d  \frac{c}{\pi}+k_s\frac{D(\omega_h)F_r(\omega_o)G(\omega_o,\omega_i)}{4 cos\theta_o cos\theta_i})} {L_i(p,\omega_i){cos\theta_i}d\omega_i}
$$

#### 法线分布函数

法线分布函数 NDF 描述了微观表面的表面法线的统计分布。确定法向量 m 为中心的单位立体角 $d\omega_m$ 与单位表面积 $dA$ ，则 $D(m)d\omega_m dA$ 为所有复合该条件的表面积，因此 NDF 是描述密度的函数，单位为 $1/sr$。NDF 的输入为粗糙的和微表面法线，即宏观法线和视线的半程向量。NDF 的输出是，在该方向上微表面法线的分布强度。将 NDF 视作概率密度函数来解释，宏观表面的综合为 1，即在半球面上对微分立体角积分，微表面之和为宏观表面 1。那么其的值域是 $[0,1)$。
$$
\int_{\Omega} D(\omega)d\omega=1
$$
#### 菲涅尔项

菲涅尔现象是视线垂直于物体表面时反射较弱，而视线与表面越靠近反射越明显。其定义了光线在不同介质表面处如何反射和折射。或者说反射和折射的能量比。

入射角 $\theta_i$ 和折射角 $\theta_r$ 的正弦之比是一个与入射角无关的常数，为相对折射率。其中的 $\eta_i,\eta_r$ 为绝对折射率
$$
\eta_i \sin \theta_i = \eta_r \sin \theta_r
\\
\frac{\sin \theta_i}{\sin \theta_r} = \frac{\eta_r}{\eta_i}
$$
当我们认为空气的折射率为 1 时，则可以得到各种介质相对空气的折射率了。

F 0 是观察方向与物体表面法线夹角为 0°时的菲涅尔反射率，当夹角达到 90°时即掠射角为 F 90，二者的取值都在 0-1 之间。常见电解质的 f 0 为 0.02-0.05，导体为 0.5-1.0，对于金属材质，其 f 0 是有色的，金属的颜色也来自于此

折射率与 f 0 的关系式为
$$
F_0=(\frac{n1-n2}{n1+n2})^2
$$
对于空气和其他介质来说，将空气的折射率视为 1，则可得到简化公式
$$
F_0=(\frac{n-1}{n+1})^2
$$
菲涅尔效应的关键是微表面的法向量 h 和入射角度 v（观察角度）。Cook-Torrance 的 Fresnel 快速近似
$$
F_{\text {Schlick }}\left(v, h, f_{0}, f_{90}\right)=f_{0}+\left(f_{90}-f_{0}\right)(1-v \cdot h)^{5}
$$

#### 几何遮蔽项
G 项，geometry function 描述几何遮蔽和阴影，其值域为 0-1
几何遮蔽项和法线分布函数息息相关，由法线分布而得出。
