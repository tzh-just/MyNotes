在片元深度与 shadowmap 进行比较后，得到非 0 即 1 的离散值，然后再对这些 0-1 的值进行滤波，采样周围多个点来得到一个 0 到 1 的连续值，被称为百分比近似滤波。滤波可以使用 box、双线性、泊松圆盘等滤波核，例如泊松圆盘。
![img](v2-8282554dbaa69bc6f80b6e7b31f16613_720w.webp)

根据 shadowmap 的分辨率和滤波核的大小 filterStride，可以得到在 shadowmap 上的采样步长。将 0-1 的泊松圆盘分布的 xy 坐标加权到 shadowmap 的坐标上，对于小于片元深度的 shadowmap 深度记为阴影，采样结束后再除以总采样数。
```c
float PCF(sampler2D shadowMap, vec4 coords) {
  float curr_depth = coords.z;
  float visibility=0.0;
  poissonDiskSamples(coords.xy);
  float textureSize = 2048.0;
  float filterStride= 2.0;
  float filterRangle = filterStride/textureSize;
  for(int i=0;i<NUM_SAMPLES;i++){
    vec4 shadowColor = texture2D(shadowMap, coords.xy+poissonDisk[i]*filterRangle);
    float shadow_depth = unpack(shadowColor);
    visibility+= curr_depth<shadow_depth+EPS?1.0:0.0;
  }
  return visibility/float(NUM_SAMPLES);
}
```
