#### Forward+
Forward+包含三个 pass
深度预处理、光源剔除和着色
第一个 pass 是 pre-z，得到场景的深度缓冲。
第二个 pass 剔除光源，将屏幕划分为 16 x 16 的 tile，tile 的 light list 记录了该 tile 的 light，片元只计算自己所在 tile 的光源的光照。这个 pass 通常使用 compute shader。根据 tile 在屏幕空间的坐标和深度可以还原出 tile 的 frustum 将计算出 tile 的视锥中的最大深度和最小深度，得到子视锥。
第三个 pass 和所在 tile 的光源进行光照计算。

#### Tiled Deferred
延迟渲染也可以使用 Tiled 方案来减少灯光与片元的计算量。