Tone Mapping 是将颜色从原始色调映射到目标色调，如如高动态范围映射到低动态范围。
对数值直接地阶段会丢失场景中明亮部分地细节，被简单地统一处理为范围极限地颜色。

ACES 颜色编码拟合的曲线最好保留了更多细节，且计算量小。渲染结果的 HDR 是线性的，通过 ACES 中的小部分算法就可以转换到 LDR 或者另一个 HDR 中。
```cpp
float3 ACESToneMapping(float3 color, float adapted_lum)
{
	const float A = 2.51f;
	const float B = 0.03f;
	const float C = 2.43f;
	const float D = 0.59f;
	const float E = 0.14f;

	color *= adapted_lum;
	return (color * (A * color + B)) / (color * (C * color + D) + E);
}
```

![img](0149709ceeb67532d4202a1a450f3584_720w.png)
## 