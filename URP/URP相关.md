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
  * BRG 生成draw commands
  * 用FilterSetting来什么时候渲染实例，
  * Draw ranges
  * 项目设置
    * 启用SRP Batcher
    * 保留BRG shader variants:Edit>ProjectSettings>Graphics 设置BatchRendererGroup variants 为Keep All
    * 如果是用了URP,最好关掉Strip Unused Variants
    * 允许 unsafe code
    * BatchRendererGroup使用DOTS Instancing shader,但并要求任何DOTS Packages,名字只体现面向数据的方式加载实例数据
  * 在URP创建BRG Renderer
    * 初始化BRG:
      * OnPerformCulling是主要入口，用来剔除可视对象的回调
        * 这个函数的两个主要作用：
          * 根据BatchCullingContext 决定是否剔除对象
          * 输出渲染对象的draw commands:写入BatchCullingOutput 参数
    * 把网格和材质注册到BRG
      * BatchRendererGroup.RegisterMesh
      * BatchRenderGroup.RegisterMaterial
    * 创建Batches
      * 每个batch 由 matadata 数据集 和 一个共享的GraphicBuffer
        * 创建元数据值后不能修改这些值，如果需要更改任何数据，需要创建一个新的Batch并删除旧的
        * 但是可以通过SetBatchBuffer随时修改Batch的GraphicsBuffer
      * BatchRendererGroup.AddBatch