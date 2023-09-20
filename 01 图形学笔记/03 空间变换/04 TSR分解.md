TRS 分解
平移向量可以从第四列中直接读取。由于旋转矩阵是正交矩阵，因此旋转矩阵的逆为旋转矩阵的转置，而缩放矩阵是一个对角线矩阵，其转置矩阵为其自身。

$$
(RS)^TRS= S^TR^TRS=SR^{-1}RS=SES
$$

因此取变换矩阵的坐上 3 x 3 子矩阵 A 的乘以其转置矩阵即得到缩放向量的平方 AA^T=SS。得到 S 矩阵后使 3 x 3 子矩阵除以缩放矩阵即可得到旋转矩阵 R=A/S。实际上，相比复杂的矩阵乘法，直接计算 3 x 3 矩阵的列向量长度是与之等价的。

```cpp
pos=m[3] ; 
for (int  i=0 ; i<3 ; i++) 
	scale [i]=glm::length(vec3(m[i]));
const glm::mat3 rotMtx(
	glm::vec3(m[0]) / scale[0],
	glm::vec3(m[1]) / scale[1],
	glm::vec3(m[2]) / scale[2]
);
rot = glm::quat_cast(rotMtx);
```

