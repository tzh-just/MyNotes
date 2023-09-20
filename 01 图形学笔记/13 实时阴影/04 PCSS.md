如果在 shadowmap 上取到像素 p 后，自定义个范围，将范围内像素的深度与 p 比较，判断哪些像素是遮挡物，将遮挡物深度取均值作为 blocker 的深度。这方法的问题是我们不知道这个 filter 应该是多少合适。
一个复杂但准确的方法是假设光源有面积，求出着色点到光源边缘，在近平面（shadowmap）上的覆盖范围，求出遮挡物的平均深度作为 blocker 的深度。
在此我们人为规定 filter 的大小。
```c
int blocker_count = 0;
float blocker_depth=0.0;
float textureSize=2048.0;
float filterStride =50.0;
```

和 PCF 的过程相同，但是计算的是遮挡部分的平均深度
```c
poissonDiskSamples(uv);
for(int i=0;i<BLOCKER_SEARCH_NUM_SAMPLES;i++){
	vec4 shadow_color=texture2D(shadowMap, uv+poissonDisk[i] * filterStride / textureSize);
	float shadow_depth = unpack(shadow_color);
	if(zReceiver>shadow_depth+EPS){
	  blocker_count++;
	  blocker_depth+=shadow_depth;
	}
}
if(blocker_count==0)return 1.0;
return blocker_depth/float(blocker_count);
```

取得 blocker 的平均深度后，根据相似三角形法则得到半影区域长度 $w_{\text{Penumbra}}$，该宽度即为 PCF 中 filter 滤波核的大小。
![](Pasted%20image%2020230704155013.png)
$$
w_{\text {Penumbra }}=\left(d_{\text {Receiver }}-d_{\text {Blocker }}\right) \cdot w_{\text {Light }} / d_{\text {Blocker }}
$$
在此我们假定了光源大小为 1，根据公式求出范围
```c
float d_blocker = findBlocker(shadowMap, coords.xy, coords.z);
float w_light = 1.0;
float d_receiver = coords.z;
float w_penumbra = w_light*(d_receiver-d_blocker)/d_blocker;
```
然后和 PCF 相同，进行滤波，区别是滤波的大小会考虑 blocker 距离的远近
```c
float textureSize = 2048.0;
float filterStride= 50.0;
float visibility=0.0;

for(int i=0;i<NUM_SAMPLES;i++){
	vec4 shadow_color = texture2D(
	  shadowMap,
	  coords.xy+poissonDisk[i]*filterStride/textureSize*w_penumbra
	);
	float shadow_depth = unpack(shadow_color);
	visibility += d_receiver-Bias()<shadow_depth+EPS?1.0:0.0;
}
return visibility/float(NUM_SAMPLES);
```
