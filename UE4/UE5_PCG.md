# PCG



### 1. 基础地形采样形式

| 基础地形采样形式 | 功能效果描述                                                 |
| ---------------- | ------------------------------------------------------------ |
| Surface Sampler  | 方形内创建生成植物密度,可设置铺满全场景配置不高的别试        |
| Spline Sampler   | 曲线采样                                                     |
| Mesh Sampler     | 网格采样                                                     |
| Get Spline Data  | 样条线采样(需设置采样目标为All World Actors/全世界行为资产,然后在Actor Selection Tag里输入设置好的样条线标签名)<br />新建蓝图,添加Spline样条线,并在属性栏的Tags添加命名标签(添加好后给PCG样条线的GSD节点)<br />新建Spline Sampler(曲线采样)并连接它<br />建议勾选select multiple |
| CreatePointsGrid | 相当于在任何采样点上再次生成采样点,类似于粒子碰撞的二次发射,而再次生成的点可以用编辑主采样点的随机,过度等一样的编辑 |
| MeshToPoints     | 在网格体(模型)上进行采样(石头上,朽木上种植草,蘑菇等)         |



#### 1.1 Surface Sampler



##### 1.1.1 Points Per Squared Meter

- 每平方米的点数，与Point Extents和Looseness互相影响，默认0.1
- 值越大，理论上点数越多，但在Point Extents和Looseness的限制下，效果并不明显
- 默认情况下增大这个值生成点数不会发生变化



##### 1.1.2 Point Extend

- 每个点的体积，默认100 * 100 * 100
- Points Per Squared Meter值不变，降低Point Extend值，生成点数增加，因为PCGVolume总体积没变，每个生成点体积变小，Points Per Squared Meter和Looseness不变
- 此时再增大Points Per Squared Meter的值，可以看到生成点数增加了



##### 1.1.3 Looseness

- 松散度，默认是1.0
- 降低这个值，生成点数增加，但继续降低，生成点数怎加不在明显，甚至开始出现下降
- 同样收到Points Per Squared Meter和Point Extend的影响
- 增大Points Per Squared Meter的值，当发现生成点数不在变化时，降低Looseness值，生成点数增加
- 同样的情况也可以降低Point Extents的值，使得生成点变小，其它值不变，整体生成点数也会增加
- 注意：当该值足够小时，虽然生成点数在增加，但松散度降低，意味着生成点的分布趋近于规整，因为生成点的体积之间的间隙开始减少，所以出现这种情况



##### 1.1.5 Unbounded

- 无边界，默认是false，即有边界，生成点不会超过PCGVolume的范围，这也是Points Per Squared Meter，Point Extents，Looseness之间会影响到整体生成点数
- Input为LandScape或LandScape，生成点会附着在LandScape表面
- 当设置为true时，生成点将超过PCGVolume边界继续生效，直至覆盖到所有的LandScape表面上
- 当LandScape足够大，生成点数会暴增，注意使用规范



##### 1.1.6 Seed

- 随机种子
- PCGVolume中的生成点数足够多时，使用效果明显
- 本身不影响生成点数量，但在其他规则的影响下，可以重新计算生成点的随机性，可能会间接影响到生成点数，位置



##### 1.1.7 Point Scale

- 点的缩放，受到Scale Method的影响，默认在Extends下不可修改
- 值越大，生成点的体积越大



##### 1.1.8 Scale Method

- 缩放方式
- 默认是Extends，只受到Points Per Squared Meter，Point Extend，Looseness的影响，即生成点的大小受到PCGVolume范围内规则影响
- Relative，即相对自身，此时Point Scale值可以任意修改，默认在受到Points Per Squared Meter，Point Extend，Looseness的影响下，也可以改变生成点的体积大小，但生成点数和分布位置不会改变，即生成点的大小不受到PCGVolume范围内规则影响，即使PCGVolume的体积整体缩放，也不会影响到该模式下生成点的体积大小
- Absolute，即绝对，大概情况类似于Relative（具体影响暂时不清楚）



#### 1.3 Mesh Sampler



##### 1.3.1 Sampling Method

- 取样方法，默认One Point Per Triangle（每个三角形一个点）：对网格的每个三角形采样一个点（在中心）
- One Point Per Vertex（每个顶点一个点）
- Poisson Sampling（泊松抽样）：使用泊松采样对网格上的点进行采样。 可能费性能，因此它不受框架限制



##### 1.3.2 StaticMesh

- 这是用于采样的样本



### 2. 控制采样点节点

| 控制采样点节点              | 功能效果描述                                                 |
| --------------------------- | ------------------------------------------------------------ |
| Transform Points            | 控制植物的旋转,大小,缩放和法线处置于水平的随机性             |
| ProjectPointsOnLandscape    | 将随机设置位置的树木拉回地面(可用可不用的节点，也可用于增加更多随机性，其后可继续搭配TransformPoints) |
| Density Filter              | 控制中间到周边的从密到稀的过滤器,过滤编辑时只显示存在的采样点 |
| DensityNoise                | 调整密度随机性                                               |
| DensityRemap                | 控制样条线框由内向外的显示过度                               |
| BoundsModifier              | 类似AE中的蒙板,让区域样条线给道路样条线让路(也可规整二次采样点的大小设置) |
| Difference                  | 让区域给道路让路之间要做识别差异化对比,把区域和样条线连接起来对比的节点 |
| Distance                    | 让不规则样条框内设置好的内向外的过度距离统一(显示黑白通道方便Density Filter过滤黑还是白采样点) |
| Copy Points                 | 可以用于二次生成的采样点绑定到主采得到的点上                 |
| ScaleByDensity              | 按密度的大小,自动缩放模型大小                                |
| Self Pruning                | 修剪采样点情况，将重合的部分精简 (该功能视情况而定可有可无） |
| NormalToDensity             | 控制采样点显示的方向是上下左右哪边                           |
| Projection                  | 投影                                                         |
| IntersectWithTaggedActorGeo | 与标记的 Actor 相交，通过场景中的Actor和采样面相交           |



#### 2.1 Transform Points



##### 2.1.1 Apply to Attribute

- 应用属性：默认是false，搭配Attribute Name一起使用

- （目前用法不明）

  

##### 2.1.2 Offset Min / Max

- 偏移量，默认都是 x,y,z（0，0，0）

- 两者搭配使用

- 假设offset min(10，0，0)，offset max（0，0，0），则所有生成点至少向x轴方向偏移10个单位

- 假设offset min(0，0，0)，offset max（100，0，0），则所有生成点向x轴方向偏移0-100个单位，最大偏移100，最小偏移0

- 即使偏移后，生成点溢出PCGVolume，生成点依旧有效，不会影响到生成点的数量

  

##### 2.1.3 Absolute Offset

- 在世界空间中设置偏移量，默认是false
- （目前用处不明）



##### 2.1.4 Rotation Min / Max

- 旋转量，默认是都是 x,y,z（0，0，0）
- 两者搭配使用
- 应用原理与 Offset Min / Max相同
- 一般常用的情况下Rotation Max设置为（0，0，360），并应用的所有生成点的自身旋转量在Z轴方向，旋转（0 - 360）



##### 2.1.5 Absolute Rotation

- 直接设置旋转而不是附加，默认是false
- 当采用面是平的，开不开意义不大
- 当采用面是高低起伏，凹凸不平的LandScape时，默认false的情况下，生成点的旋转量与生成点所在的Landscape面的法线保持一致
- 当生成点最总要生成staticmesh时，例如树木和房屋，则需要开启Absolute Rotation，使得旋转量与世界默认旋转量保持一致



##### 2.1.6 Scale Min / Max

- 缩放量，默认是都是 x,y,z（1，1，1）
- 与Offset Min / Max和Rotation Min / Max使用原理相似
- 一般情况下，Min和Max搭配使用，例如ScaleMin（0.8，0.8，0.8），ScaleMax（1.1，1.1，1.1），所有被作用的生成点体积在（0.8- 1.1）之间随机缩放



##### 2.1.7 Absolute Scale

- 直接设置比例而不是乘法，默认是false
- （用处不明）



##### 2.1.8 Uniform Scale

- 每个轴上的比例均匀，使用 scale min 和 scale max 的 x 分量
- 默认为true
- （用处不明）



#### 2.3 Density Filter

- 密度过滤器

- 主要通过生成点之间的间距比为过滤参数

- 依据Lower和Upper进行范围筛选，符合范围的可以继续有效生成

- 最终结果还受到Invert Filter影响

- 补充：

  - 也可以先用 Normal To Density 进行方向过滤，例如不希望草长在几乎垂直的岩壁上

  - 直接默认的 Normal To Density 就是Z轴方向，使用Density Filter就可实现效果

  

##### 2.3.1 Lower Bound

- 下限，默认0.5，取值范围 0 - 1
- Upper Bound值不变的情况下，Lower越大，理论上生成点越少，反之亦然



##### 2.3.2 Upper Bound

- 上限，默认1，取值范围 0 - 1
- Lower Bound值不变的情况下，Upper越小，理论上生成点越少，反之亦然



##### 2.3.3 Invert Filter

- 反转过滤器，默认为false
- 默认Lower和upper的范围为0.5 - 1
- 修改为 0.8 - 1，理论上生成点数减少，此时勾选Invert Filter为true，生成点数会增大
- 即false时生成的点数，在true不再生成，原本不生成的点此时开始生成



#### 2.4 Density Noise

- （目前不明）



#### 2.6 Bounds Modifier

- 边界修饰符
- 默认值Mode：Scale，BoundsMin / Max （1，1，1） 
- 默认情况下，比如石头的分布，例如Surface Sample的Point Extents的值可以设置石头的边界间距
- 但遇到其他情况，例如草使用differences包围石头分布，草会被石头的Point Extents排在边界外
- 需要Bounds Modifier重新修改石头的边界，该边界仅影响草的分布，不影响原有的边界效果



##### 2.6.1 Mode

- 边界模式：默认Scale
- (其他不明)



##### 2.6.2 BoundsMin / Max

- 生成点体积边界的范围，越小，其余生成点就越靠近，甚至穿模



##### 2.6.3 Affect Steepness

- 影响陡度，默认是false
- 开启后可设置Steepness



##### 2.6.4 Steepness

- 陡度，默认是1
- （目前不明，似乎越小，边界会变大）



#### 2.7 Difference

- 差异



##### 2.7.1 Density Function

- 密度函数，默认是Minimum
- 一般实现草丛围绕石头，但不穿模石头，可以切换为Binary
- 草（主动）连Source，石头（被动）连Differences，Out连草的StaticMeshSpawner
- 假设草铺满，连Source，但是有石头的地方不长，石头连Differences，即铺满的草地减去石头的地方，就是草实际会长的地方
- 补充：
  - 如果发现草虽然只长在石头外边，但间隔很大，甚至没有草，可能的原因是石头的类似Surface Sample节点的Point Extents值较大
  - 可以减小Point Extents值，但这会破会石头原本设好的分布逻辑，所以可以在石头的Transform Points节点后使用BoundsModifier节点
  - 不破坏石头原有设定，进行边界值修改

- （其他不明）



#### 2.9 Copy Points

- Source是需要采样的数据
- Target是Source的采样数据用于Target
- Out数据用于给Source的数据进行生成
- （参数不明）默认够用



#### 2.10 Normal To Density

- 法向密度



##### 2.10.1 Normal

- 法向
- 默认值（0，0，1.0）
- 在Z轴上进行过滤



#### 2.11 Self Pruning

- 自我修剪



##### 2.11.1 Pruning Type

- 修剪类型：默认是Large To Small
- （具体不明），默认够用



#### 2.12 Normal To Density

- 垂直于密度



##### 2.12.1 Normal

- 法线方向：默认（0，0，1），垂直地面向上的方向，适用于石头，木头上面长草和蘑菇之类
- （其他不明）默认够用



#### 2.14 IntersectWithTaggedActorGeo

- 与标记的 Actor 相交



##### 2.14.1 Actor Tags

- 假设场景中获取某个Actor例如Cube的采样面，使得Cude所在地不生成草
- 为Actor添加Tag，在IntersectWithTaggedActorGeo节点中添加tag
- 节点前面需获取采样，例如Surface Sample节点
- 一般搭配Differences节点即可实现效果



### 3. 网络体代理节点

| 网格体代理节点      | 功能效果描述                                                 |
| ------------------- | ------------------------------------------------------------ |
| StaticMeshSpawner   | 替换点为网格体(可多添加和控制多网格体的数量比例)             |
| PropertyToParamData | 将制作好的各类PCG封装到蓝图后,可以通过此节点把需要在蓝图细节框里显示出PCG内的随机大小,旋转排列等数据表露到蓝图细节框里快捷调试参数 |
| SpawnActor          | 在生成点上生成Actor                                          |



#### 3.1 StaticMeshSpawner

- 静态网格生成器



##### 3.1.1 Mesh Entries

- 网格实体，默认是空
- 通过加号添加，可以添加多个StaticMesh
- 大量staticMesh生成会占用大量性能资源，例如草和小石子，可以该节点内统一修改“碰撞预设”



#### 3.3 Spawn Actor

- 生成Actor



##### 3.3.1 Template Actor Class

- 生成的Actor模板类
- 例如生成Character
- 可以用于生成场景中可交互的Actor