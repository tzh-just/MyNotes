Z-prepass 配合 early-z 使用，在第一个 pass 中只写入深度，在第二个 pass 中关闭深度写入，开启 early-z，深度比较函数设为 Equal。无论渲染顺序如何，片元着色阶段没有计算消耗，深度缓冲中的结果是完全正确的。
Z-prepass 与延迟渲染相似而又不同，z-prepass 的两次 pass 都对场景内的几何进行了处理，而延迟渲染的第二个 pass 则是根据 GBuffer 的信息在屏幕一样大的 Quad 上绘制最终结果。因此在 overdraw 不多的场景，z-prepass 的两个 pass 比不使用 z-prepass 更加耗时。
在渲染透明物体时，如果 z-prepass 的第一个 pass 不写入深度，则可以看到模型的背面，若写入了深度，则完全透明。