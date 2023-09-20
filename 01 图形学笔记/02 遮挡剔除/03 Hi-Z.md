【思路】在绘制前先得到场景的深度图，将物体的包围盒投影到屏幕空间后判断包围盒与深度图的关系。
【要点】为了减少包围盒内像素与深度图的遍历比较消耗，可以通过 mipmap 使得一次查询就可以获得一块区域的深度，只是 mipmap 的生成不再是平均值，而是最远值。如果最远处的遮挡物都遮挡住了包围盒，才被保守地视为被遮挡。因此 mipmap 需要自己利用 compute shader 或者 pixel shader 生成。

```cpp
fixed4 frag(v2f i) : SV_Target  {
	float2 offset = 0.5 * _MainTex_ST.xy;  
	float a = tex2D(_MainTex, i.uv + offset * float2(-0.5, -0.5)).r;  
	float b = tex2D(_MainTex, i.uv + offset * float2(-0.5, 0.5)).r;  
	float c = tex2D(_MainTex, i.uv + offset * float2(0.5, -0.5)).r;  
	float d = tex2D(_MainTex, i.uv + offset * float2(0.5, 0.5)).r;  
	float maxz = max(max(a, b), max(c, d));
	return maxz;
}
```

【改进 1】利用上一帧的深度图进行剔除，将上一帧的 VP 矩阵传给着色器，将顶点变换到上一帧的 NDC 空间后与 hiz 进行比较。挑选最近的若干物体做遮挡体进行全屏分辨率渲染，改深度图不仅用于 hiz 还可以用于 early-z，结合历史帧的深度图可以进行补洞。
【改进 2】同样是利用上一帧的深度图进行剔除，但是不渲染遮挡体，会导致提出命中率不高以及错误剔除，因此需要等深度信息更新后重新做一次剔除，将被错误剔除掉的物体重新渲染一遍。
【缺点】相机移动过快会导致剔除效果很差。二次剔除会导致性能开销翻倍。读取上一帧结果需要数据回读，这是移动端的性能瓶颈。