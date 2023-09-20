GPU Instance 适用于相同的 mesh，相同的材质。对于同一 mesh 的不同实例只提交一份，同时提交 mesh 各自的 M 矩阵。
![](Pasted%20image%2020230830095332.png)
在着色器中通过 StructuredBuffer 标记该 buffer
顶点着色器中将 M 矩阵替换掉，方便继续使用 unity 内置的 MVP 相关的方法
![](Pasted%20image%2020230830095447.png)
