IBL 模拟了间接光，将环境光提前烘焙到 cubemap，在运行时根据法线方向查询间接光的 radiance。
对于漫反射的渲染方程经过简化和近似后，积分项就只有入射方向 Radiance 和入射方向余弦了，这部分可以提前计算，得到的是一个相比原图很模糊的结果。
对于镜面反射，可以根据积分近似拆成两项，一部分是入射方向 Radiance 的积分，可以预计算，这部分额外考虑了粗糙度，越粗糙采样的向量会越分散，因此需要生成 mipmap。在 shader 中可以根据粗糙度映射到 mipmap 的 level。
虽然镜面反射受到观察方向的影响，预计算无法得知，但是我们可以假设镜面反射的方向总是等于采样的方向。这会导致当视线趋近于掠射角时，预计算的结果就不是很准确了。
另一部分是 BRDF 和入射方向余弦的积分，考虑 Cook-Torrance 的镜面反射项 BRDF，经过变形后 BRDF 只受到粗糙度和半程向量夹角余弦的影响。这部分也可以预计算。
### 漫反射项
对于漫反射，表面接收来自各个方向的入射光。观察渲染方程，如果想实时地得到环境光地在出射方向上的 radiance：$L_o(p,\omega_o)$，那么我们要积分整个半球面入射的 brdf。
$$
L_{o}\left(p, \omega_{o}\right)=\int_{\Omega}\left(k_{d} \frac{c}{\pi}+k_{s} \frac{D F G}{4\left(\omega_{o} \cdot n\right)\left(\omega_{i} \cdot n\right)}\right) L_{i}\left(p, \omega_{i}\right) n \cdot \omega_{i} d \omega_{i}
$$
对于一个环境光 cubemap，我们可以查询到任意入射方向 $\omega_i$ 的 radiance:
```c
vec3 radiance = texture(_cubemap_env, w_i).rgb;
```

其 BRDF 为常数，那么可以将每个法线都采样所有方向的环境光，计算后保存在 cubemap 中。在运行时再用法线去查询积分结果。具体过程就是，遍历所有法线，然后遍历所有 cubemap 的像素位置，累加 cosine 加权的值后再平均
$$
L_o(p,\omega_o)=\int_{\Omega} L_i(p,\omega_i){{\cos\theta_i}d\omega_i}
$$
### 镜面反射项
对于镜面反射，按照积分近似公式，我们可以将渲染方程拆成两个部分，前者是光照在半球面上积分结果的规范化，这就相当于做 Prefiltering 预过滤环境光，使得在实时只需要采样 1 次，查询过滤后的值。
$$
\int_{\Omega} f(x) g(x) \mathrm{d} x \approx \frac{\int_{\Omega_{G}} f(x) \mathrm{d} x}{\int_{\Omega_{G}} \mathrm{~d} x} \cdot \int_{\Omega} g(x) \mathrm{d} x
$$
$$
L_{o}\left(p, \omega_{o}\right) \approx  \frac{\int_{\Omega_{f_{r}}} L_{i}\left(p, \omega_{i}\right) \mathrm{d} \omega_{i}}{\int_{\Omega_{f_{r}}} \mathrm{~d} \omega_{i}} \cdot \int_{\Omega^{+}} f_{r}\left(p, \omega_{i}, \omega_{o}\right) \cos \theta_{i} \mathrm{~d} \omega_{i}
$$

![](Pasted%20image%2020230628152706.png)

#### BRDF 预计算
首先处理 BRDF 的积分，采取预计算的思路，以 Cook-Torrance BRDF 为例，我们先观察 BRDF 的变量
$$
f\left({i,o}\right)=\frac{D(h) G(i,o,h) F(i,h)}{4 \cos \theta_i \cos \theta_o}
$$
观察其中的菲涅尔项和法线分布函数
$$
F=F_0+(1-F_0)(1-\cos \theta)^5
$$
$$
D(h)=\frac{e^{-\frac{\tan ^{2} \theta_{h}}{\alpha^{2}}}} {\pi\alpha^{2} \cos ^{4}\left(\theta_{h}\right)}
$$
可以看出菲涅尔项取决于 $F_0$ 和 $\cos \theta_h$，法线分布函数取决于粗糙度 $\alpha$ 和 $\cos \theta_h$。将菲涅尔项中的 $F_0$ 提出，然后把参数带入积分
$$
F=F_0(1-(1-\cos\theta)^5)+(1-\cos \theta)^5
$$
$$
F_{0} \int_{\Omega^{+}} {f_{r}\left(p, \omega_{i}, \omega_{o}\right)}\left(1-\left(1-\cos \theta_{i}\right)^{5}\right) \cos \theta_{i} \mathrm{~d} \omega_{i}+\int_{\Omega^{+}} {f_{r}\left(p, \omega_{i}, \omega_{o}\right)}\left(1-\cos \theta_{i}\right)^{5} \cos \theta_{i} \mathrm{~d} \omega_{i}
$$
由此 BRDF 的积分只受到粗糙度 $\alpha$ 和 $\cos \theta_h$，二者的取值范围都是[0,1]的，因此可以得到一张预计算积分的二维纹理。
![](ibl_brdf_lut.png)

#### 入射光预计算
总结：
镜面反射部分，拆成了入射光积分和 BRDF 积分再相乘。
假定了环境光在各个方向上均匀且为 1 的
计算 BRDF 积分时，以粗糙度和 NdotH 作为自变量进行预计算，生成一张纹理
计算入射光积分时，假设了观察方向与法线方向相同。

镜面反射