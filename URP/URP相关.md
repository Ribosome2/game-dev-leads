# Unity的URP
* B站官方解释：https://www.bilibili.com/video/BV1fK4y1a78s/?spm_id_from=333.337.search-card.all.click&vd_source=72868cdf66be986462ce798fe9e21050
* URP C#源码：https://github.com/Unity-Technologies/Graphics
* Blog:https://blog.unity.com/technology/achieve-beautiful-scalable-and-performant-graphics-with-the-universal-render-pipeline
 
## SRP Batcher 原理
* 传统的优化师通过降低DC
* SRP Batcher 是通过减少 draw calls 之间的 render-state 来较少消耗
  * 把一系列的bind 和 draw GPU 指令合并起来，每一个序列，就是一个SRP Batch
    * 在FrameDebugger 里面可以看到一个个SRP Batch
  * 多个对象的数据会通过 offset 合并成一个buffer
  * GPU Instancing 和 SRP Batcher 不兼容，如果能用instancing， 会比SRP Batcher效率更高
  * 对GameObject的要求
    * 必须有mesh或者skinned mesh,粒子不行
    * GameObject 不能使用MaterialPropertyBlocks
    * Shader 得兼容
      * 引擎自带的 属性必须用一个 名为UnityPerDraw 的ConstantBuffer 里面
      * 非自带的属性 必须用名为 UnityPerMaterial 的 constant buffer 里面
      * 不要使用MaterialBlock,(默认管线使用这个是为了，修改材质属性，但是不打断DC合批)，但是SRP 不兼容
* BatchRenderGroup
* 