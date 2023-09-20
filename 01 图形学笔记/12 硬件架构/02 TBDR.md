### TBR/TBDR
传统 gpu 在片元着色后要将数据从内存和显存之间搬来搬去，尤其是 MRT 有多个 RT，使得带宽的开销过大，加速手机发热。移动端 GPU 则把 RT 切分为多个 Tile，在 tile 上完成 pass 后再把 tile 读回内存

![The Immediate Mode Rendering (IMR) graphics pipeline ](IMR-Pipeline-1-e1500627612820.jpgwidth=990&height=241&name=IMR-Pipeline-1-e1500627612820.jpg)

相比之下 IMR GPU 经过顶点着色器、光栅化、early-z 后直接进行片元着色，而 TBR GPU 则在顶点着色后会进行 Binning/Tiling。Tiling 将三角形顶点 index 写入 tile 的 polygon list，renderpass 中的所有 drawcall 的 tiling 都结束后，则该 renderpass 所有的图元都写入到了 tile 对应的 polygonlist，这些信息存在 system memory 中，需要等待 tiling 全部结束才会逐 tile 光栅化，在 tile 中顺序读取 polygon list 中的图元，逐片元光栅化，fragment shading 等，在此期间，光栅化、深度模板测试、ps 和混合等需要对 framebuffer 大量读写的操作都可以在 GPU on-clip memory 进行，每个 Tile 绘制完就写入 FrameBuffer，因为 on chip memory 没有那么充足的空间存储整个 FrameBuffer。

![The Tile Based Rendering (TBR) graphics pipeline](TBR-Pipeline-1-e1500627587478.jpgwidth=990&height=301&name=TBR-Pipeline-1-e1500627587478.jpg)

显然写入 polygon list 到 system memory 的过程也是一个性能瓶颈，对 TBR 架构的优化之一就是 Hierarchical Tiling。即在 Tiling 阶段，先确认三角形覆盖的 tile。（三角形过大会导致覆盖过多 tile，给 tile 做层级可以避免带宽浪费）将多个 tile 组合成一个 bin，对于覆盖多个 tile 的大三角形，只需要写一个 polygon list。例如 bin 的，光栅化时依旧是逐 tile 遍历，但是会把 tile 的所有 level 的 bin 都读一遍，例如左上角的 tile 1需要额外16x16,32x32,64x64的 bin

![img](v2-9f8f823d67e10ff88f828b3b9d2b1f9e_720w.png)

为了减少 overdraw，early-z 可以一定程度上进行优化，但是 early-z 要求渲染次序时从前到后的，显然排序对 cpu 是不小的开销，且对于前后交错的物体排序是没用的，即这个排序的粒度只到 mesh。而 TBDR 在 TBR 的基础上，在逐 tile 处理时，tile 上的图元是按照 drawcall 的顺序的，因此我们可以对图元进行一些处理，以优化 overdraw

![The Tile Based Deferred Rendering (TBDR) graphics pipeline (PowerVR Series5XT)](TBDR-Pipeline-1024x275.jpg)

PowerVR 实现了真正的 TBDR，在片元通过 early-z 后先不继续绘制，而是记录需要绘制的片元，等到处理完 tile 中所有的图元后，每个像素要绘制哪个片元也就确定下来了，再送入后面的管线。