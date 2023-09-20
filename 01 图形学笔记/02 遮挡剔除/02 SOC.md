UE 方案在 X 轴维度上把 depth buffer 分为六个子块，每一个子块的 mask 都是一个 uint 64 的数组来表示，长度取决于帧缓冲的长度 `uint64 Data[FRAMEBUFFER_HEIGHT]`。
首先标记 mesh 是否为遮挡物，一般是比较大的 mesh。遮挡物经过裁剪后变换到屏幕空间，取三角形的最近深度记录下来。然后按照 Y 轴来排序，计算三角形在 X 轴上的跨度后将 mesh 的三角形分别保存在各个子块中。

```cpp
struct FFramebufferBin{
	uint64 Data[FRAMEBUFFER_HEIGHT];
};
struct FSortedIndexDepth { 
	int32 Index; 
	float Depth; 
}; 
```

![](Pasted%20image%2020230801092144.png)

每个子块都有一个 FOcclusionFrameData，保存了子块中的每个三角形的屏幕空间坐标、最近深度、三角形所属的 mesh id 等等。
将非遮挡物的包围盒变换到屏幕空间坐标，而包围盒可以分为两个三角形添加到子块中。遍历每个子块，按照深度信息将子块内的三角形由近及远地排序。遍历排序后的三角形列表，如果是遮挡物就标记覆盖的 mask 为 1，如果不是遮挡物就还原该三角形的包围盒，检查包围盒内覆盖的 mask 里有没有 0，如果有说明被遮挡物没有被完全遮挡，则需要全部渲染。

```cpp
struct FScreenTriangle { 
	FScreenPosition V[3]; 
};
struct FOcclusionFrameData { 
	TArray<FSortedIndexDepth> SortedTriangles[BIN_NUM]; 
	TArray<FScreenTriangle> ScreenTriangles;
	TArray<FPrimitiveComponentId> ScreenTrianglesPrimID;
	TArray<uint8> ScreenTrianglesFlags; 
};
```

【缺点】显然对于细碎三角形特别多的场景，例如草地和树叶等，会需要占用大量的内存来记录三角形的各种信息。并且只要有一个 mask 为 0，被遮挡物就要被放过送入 GPU，极端情况下收益会很低。
【优化】同样的，如果能减少需要处理的三角形数量，就可以改善软件遮挡性能。例如设置 LOD 来减少三角形的数量。