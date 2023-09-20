第一个 pass，从光源看向场景，经过渲染后得到一张深度缓冲，称为 shadow map。在顶点着色器中，将顶点的 zw 传入片元着色器，这是因为 unity shader 中直接使用 SV_POSITION 得到的是光栅化后的坐标，而不是期望的 NDC 坐标，即 XY 为屏幕坐标，而 ZW 暂不知是否进行了透视除法，为保证准确，此处自己传递裁剪空间下的 zw 后在片元着色器中自己进行透视除法得到期望的 NDC 坐标。
```c
v2f vert(appdata_base v)  {  
    v2f o;  
    o.vertex = UnityObjectToClipPos(v.vertex);  
    o.depth = o.vertex.zw;  
    return o;  
}
fixed4 frag(v2f i) : SV_Target  {  
    float depth = i.depth.x / i.depth.y;  
#if defined (UNITY_REVERSED_Z)  
    depth = 1.0 - depth;
#endif  
    fixed4 color = EncodeFloatRGBA(depth);  
    return color;  
}
```
第二个 pass，从相机出发渲染场景，以延迟渲染管线为例，读取 GBuffer Pass 中的 depth，将深度转换为线性深度。然后将划分比例乘以阴影最大距离进行比较，即在线性空间中比较非归一化的深度值
```c
float d = UNITY_SAMPLE_DEPTH(tex2D(_gdepth, uv));  
float d_lin = LinearEyeDepth(d);
```
通过 Blit 函数执行第二个 pass，绘制在 quad 上，因此 $uv*2-1$ 实际上就是片元 NDC 空间下的 xy 坐标。传入 VP 矩阵的逆变换，根据以上信息重建片元的世界坐标。
```c
float4 ndcPos = float4(uv * 2 - 1, d, 1);  
float4 worldPos = mul(_vpMatrixInv, ndcPos);  
worldPos /= worldPos.w;
```
将片元从世界空间变换到光源的 NDC 空间，此时的坐标 xy 变换到 0-1 后即为 shadow map 的 uv 坐标，而 z 值则为相机深度
```c
float4 shadowCoord = mul(_vpMatrixShadow, worldPos);  
shadowCoord /= shadowCoord.w;  
float2 uv = shadowCoord.xy * 0.5 + 0.5;
```
在 0-1 的范围中，如果 shadowmap 的采样深度小于片元深度，则说明 shadowmap 记录的遮挡物更近，相机看到的片元处于阴影之中。
```c
float cameraDepth = shadowCoord.z;  
float sampleDepth = DecodeFloatRGBA(tex2D(_ShadowTex, uv));  
#if defined (UNITY_REVERSED_Z)  
if(sampleDepth>cameraDepth) return 0.0f;  
#else  
if(smapleDepth<cameraDepth) return 0.0f;  
#endif
```

### Depth Bias
由于 shadowmap 在第一个 pass 中在光源空间采样深度是离散的，离散会导致连续的平面中不同深度的一小段会被记录为同一深度。而第二个 pass 中相机采样到相同的片段，但 z 值大于 shadowmap 中的深度时，shadow map 渲染出间隔的条纹的现象。
因此将采样到 shadowmap 深度稍微离相机近一些可以缓解这一现象。或者让片元沿着法线外扩一些。
```c
float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);  
float3 normal = tex2D(_GT1, uv).rgb * 2 - 1;  
half NoL = dot(normal,lightDir);  
worldPos.xyz += normal * 0.01;
```
由于阴影相机是根据主相机的视锥切分而来，随着主相机的移动，阴影相机随之变化。当稍稍移动时，采样的像素会发生微小变化，同一个三角形采样到的深度不同了。
如果能让光源相机每次移动都是 shadowmap 一格像素距离的整数倍，那么采样的三角形像素就不会发生形变。
如果沿着光源到着色点的方向偏移一小段距离，减少变换过来的坐标 z 值大于离散深度的情况，可以缓解自遮挡的现象。但偏移量的多少需要不断调试，太多的偏移量会导致许多本应该是阴影的部分被遗漏，使得阴影像是悬浮起来了。