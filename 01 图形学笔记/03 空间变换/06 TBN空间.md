切线空间
设三角形边 E 1 和 E 2。取三角形上一点的切线 T 与副切线 B，映射到三维空间有以下关系：

$$
\begin{array}{l}
E_{1}=\Delta U_{1} T+\Delta V_{1} B 
\\\\
E_{2}=\Delta U_{2} T+\Delta V_{2} B
\end{array}
$$

其中的的Δu 和Δv 是顶点之间 uv 差值。经过计算我们可以得到切线和副切线在三维空间中的映射：

$$
\begin{array}{l}
T=\frac{\Delta V_{1} E_{2}-\Delta V_{2} E_{1}}{\Delta V_{1} \Delta U_{2}-\Delta V_{2} \Delta U_{1}} 
\\\\
B=\frac{-\Delta U_{1} E_{2}+\Delta U_{2} E_{1}}{\Delta V_{1} \Delta U_{2}-\Delta V_{2} \Delta V_{1}}
\end{array}
$$

一般从模型数据中存储了 tangent 的信息，在顶点着色器中将切线变换到世界空间

```cpp
o.tangent = UnityObjectToWorldDir(v.tangent) * v.tangent.w;
```

在片元着色器中构建 TBNN 矩阵。

```cpp
half3 T = normalize(i.tangent);  
half3 N = i.normal;  
half3 B = normalize(cross(N, T));
half3x3 TBN = half3x3(T, B, N);
```

从法线贴图中可以读取局部空间法线，经过世界空间 TBN 变换后得到了新的世界空间法线

```cpp
half3 Normal = UnpackNormal(tex2D(_NormalTex, uv));    
N = normalize(mul(Normal, TBN));
```