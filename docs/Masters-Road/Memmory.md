# 性能优化 - 内存

::: 

阅读本文大概需要15分钟

对内存的掌控在游戏研发中起着至关重要的作用，它直接影响到地图大小、角色数量、物体数量以及用户操作体验。因此，对内存的深入理解和有效控制是提升游戏性能表现和细节品质的必经之路。

本文将介绍口袋方舟中，内存使用上的细节及注意事项。

:::

## 1.增加内存的操作

> 创建**客户端对象**会增加**客户端**的内存
>
> 创建**服务器对象**会增加**服务器**的内存
>
> 创建**双端对象**会增加**客户端**及**服务器**的内存
>
> 当资源被加载，资源就会占用一定的内存，此时想要创建**对象实例**，就能使用该内存中的资源
>
> 每创建一个**对象实例**，也会产生对应的实例占用的内存

1. 拖入资源到主视口。
2. 代码动态创建的**对象实例**产生内存使用。

```typescript
// 通过spawn创建了一个宝箱模型
let go = GameObject.spawn("20689")

// 也可以通过asyncSpawn接口创建模型或预制体
GameObject.asyncSpawn("20689").then((go: GameObject)=>{
    // 做些什么
})
```

3. 加载 UI **对象实例**产生内存使用，当然其用到的图片资源也有资源内存使用。

```typescript
// 通过UIService.show创建UI，第一次调用的时候会创建一个，以后再调用都是使用的第一个
UIService.show(xxUI)

// 通过CreateUI接口创建UI，每次调用创建的都是新的
mw.CreateUI()
```

4. 资源加载内存使用。

```typescript
// 通过代码加载某个资源
AssetUtil.asyncDownloadAsset("xxxx").then(() => {
    // 做些什么
});
```

5. 从资源库拖入到对象管理器的**优先加载**里的资源内存使用。

![image-20231201094537272](https://arkimg.ark.online/image-20231201094537272.webp)



## 2.内存超限导致的后果

> 客户端内存超限：无法创建更多的角色及 NPC、对场景进行拓展、增加细节、扩大地图、严重的可能引起游戏卡顿、甚至是游戏闪退
>
> 服务器内存超限：引起服务器崩溃或玩家掉线

![img](https://arkimg.ark.online/1695707946428-5.webp)

<center>玩家掉线</center>

## 3.如何优化内存

### 创建模型内存上升示例

讲如何优化内存之前，先举两个例子，电脑上打开了任务管理器并运行了游戏。

1. 创建大量基础模型，展示相同资源，大量对象实例导致内存的上升情况。

<video controls src="https://arkimg.ark.online/20231201103717_rec_.mp4"></video>

从视频中能了解到，示例中在电脑上运行游戏，开始的时候内存占用在 491 MB，程序里生成了 2000 个基础方块后，内存占用上升到了 624 MB，总体上升了 133  MB 左右，虽然这个值可能不准确（对象管理器无法精准确定内存变化），但是能说明大量相同对象的创建会造成的内存上升情况。

**结论：** 大量创建相同对象需要特别小心，内存的上升需要进行提前测试控制在合理的范围内。

2. 创建大量不同模型，展示不同资源，不同对象实例导致内存的上升情况。

<video controls src="https://arkimg.ark.online/20231201110418_rec_.mp4"></video>

视频中创建了 50 个对象，内存大概上升了 35 MB，能明显的感知到，复杂模型（不是基础模型的都是复杂模型）的内存上升是高于基础模型的。

**结论：**不同的模型对象因为**模型面数**和**贴图质量**不同的原因，创建占用的内存是不尽相同的，口袋方舟中主要需要注意模型面数，优先选择模型面数较低的来使用。

### 如何查看模型面数

![image-20231201112502316](https://arkimg.ark.online/image-20231201112502316.webp)

<center>如图中，在资源库中选中模型，点击模型右上角的+号，会弹出该模型的具体信息，关注右侧弹出窗口中模型的三角面数，图例中的为 990</center>



### 资源加载相关优化

![img](https://arkimg.ark.online/185923x3ib8pko2piiiigb.webp)

<center>重复资源使用，同色线条链接标记的都是使用的相同模型</center>

1. 多使用相同资源创建对象实例。

通过更换摆放的位置和方式来达到节省资源占用内存的目的。

比如上图中，通过调整旋转朝向来横着斜着放置木箱，缩放大小等来丰富场景元素，实际**资源使用**量就还是很少的几个，其对应资源占用的内存也就只会有几个（比如多个木箱只有一个资源内存占用，木桶只有一个资源内存占用）。

2. 常用资源放在**优先加载**中

由于未使用的资源过段时间会被系统回收，因此常用资源置于优先加载里可以帮助常用资源无需处理加载逻辑，也不用去维护资源去减少内存碎片。

> 内存碎片是指数据存储过程中产生的，不能满足后续新内存使用需求的小块内存

![image-20231201143201933](https://arkimg.ark.online/image-20231201143201933.webp)

如图，比如有一个 2 格大小内存的资源想要加载到内存中，现在图中 1 和 2 位置都无法满足，系统往后寻找到 3 和 4 位置才能使用。

大量的内存碎片将会使得系统不断申请新的内存，申请内存的这个过程将影响性能。

### 对象池相关优化

编写代码时候，可能会循环之类的地方不断创建对象，却又没有对对象进行管理，此时可能引起频繁的垃圾回收甚至是内存泄露。

> GC（Garbage Collector 垃圾回收）：内存管理工具，能帮助软件节省内存的占用，频繁调用可能会造成软件卡顿
>
> 简单理解 - 当口袋方舟里一个对象不再被引用的时候，这个对象占用的内存就将会被系统回收
>
> 通俗理解 - 家里吃饭使用了很多的碗，当碗里的菜被吃光后，会洗干净这个碗回收以方便新的菜有碗可用

```typescript
// for循环里创建
for(let index = 0; index < 100; index++) {
    let go = GameObject.spawn("xxx")
}

// 一个没有终止条件的while循环，做些什么
while(true) {
    let go = GameObject.spawn("xxx")
}
```



> 对象池的通俗理解 - 我有一个书柜里放了几本语文书，当我下次想看书的时候我应该接着使用它们即可，而不是重新买几本相同的书使用后再次放进书柜，这样会导致书柜存放了同样的东西并且最终挤不下

在口袋方舟中，我们可以使用 GameObjPool.spawn 相关接口创建基于对象池管理的对象。

```typescript
// 创建了一个Character功能对象，其资源类型是Asset，对应资源库
let cha = GameObjPool.spawn("Character", mwext.GameObjPoolSourceType.Asset)
cha.worldTransform.position = new Vector(500, 500 , 150)

// 暂时不用了，将该角色对象归还给对象池
GameObjPool.despawn(cha)

// 后续再使用对象池创建对象，就会将之前创建的对象取出来接着用，不会设计到对象的重新创建
```



### 使用 Devtools

DevTools 是 Chrome 浏览器提供的针对 JS 虚拟机的性能、内存分析工具,日常开发中，DevTool 将会是我们使用频率最高的优化工具。

**使用前准备**

1. 下载谷歌浏览器

2. 在MW日志中抓取地址,一般来说地址都是如下两个：

>   服务端地址：devtools://devtools/bundled/inspector.html?v8only=true&ws=127.0.0.1:23300
>
>   客户端地址： devtools://devtools/bundled/inspector.html?v8only=true&ws=127.0.0.1:23301 （端口数逐渐累加）

3. 在谷歌浏览器中输入上面的调试地址

![img](https://arkimg.ark.online/095744wi0iwiderowgwwno.webp)

<center>进入后，界面会呈现下图内容。运行游戏后，点击 Reconnect DevTools 即可开始调试</center>

![image-20231201150211572](https://arkimg.ark.online/image-20231201150211572.webp)

<center>中文界面表现，运行游戏后，点击重新连接 DevTools</center>



![img](https://arkimg.ark.online/100522ef1h7am3f29s6ahl.webp)

<center>连接上 DevTools 后，点击上图中的 Memory 即可开始使用 Memory Profiler</center>

**使用 Memory Profiler 的目的**

1. 检查内存泄漏

2. 检查 GC（Garbage Collector 垃圾回收）问题

**Memory Profiler 的三种采样方式**

![image-20231201150648067](https://arkimg.ark.online/image-20231201150648067.webp)

**堆快照**

顾名思义，堆快照会采集当前时刻整个 JS 虚拟机的内存分布情况。

通常我们会利用堆快照来定位某一具体操作导致的内存异常。在口袋方舟游戏中，我们可以用它去定位某个游戏操作的内存情况。操作方法通常如下：

1. 采样

2. 执行一次想要检测的操作

3. 等待操作完全结束

4. 停止采样

5. 从第一步重新开始重复N次（具体次数根据项目不同，采样次数也不同）



先比对第一张快照和最后一张张快照的内存大小，如果内存总体呈上升趋势，说明 A 操作中出现内存泄露的问题。此时我们就可以针对和 A 操作的关联的函数、对象逐一进行分析和优化。

![img](https://arkimg.ark.online/102443ui36v63l3m3vwtye.webp)

![image-20231201151338234](https://arkimg.ark.online/image-20231201151338234-1701414836783-13.webp)

> 堆快照数据图右上角的 Summary 字样位置，可以选择查看内存快照的方式，可选方式如下：
>
> Summary - 可以显示按构造函数名称分组的对象。使用此视图可以根据按构造函数名称分组的类型深入了解对象（及其内存使用），适用于跟踪编辑器对象泄漏。
>
> Comparison - 可以显示两个快照之间的不同。使用此视图可以比较两个（或多个）内存快照在某个操作前后的差异。检查已释放内存的变化和参考计数，可以确认是否存在内存泄漏及其原因。
>
> Containment - 此视图提供了一种对象结构视图来分析内存使用，由顶级对象作为入口。
>
> Statistic - 内存使用饼状的统计图。



**时间轴快照**

时间轴快照其实和堆快照差不多，但是他的采样频率是固定 50 ms 一次。可以用来在对一个过程进行内存采样。不过该功能目前在口袋方舟中不可用，这里就不多详细介绍了。



**内存信息采样**

内存信息采样，使用采样的方法记录内存分配。此配置文件类型具有最小的性能开销，可用于长时间运行的操作。它提供了由 javascript 执行堆栈细分的良好近似值分配。我们用它来查看在一段游戏过程中整体的内存分配情况。

![img](https://arkimg.ark.online/103912xatdl5qff7d7lk1l.webp)

<center>一般来说，一个游戏的核心逻辑的内存走势应该如上图</center>

**1.** 游戏开始，大量申请内存

**2.** 内存达到顶峰

**3.** 内存走势近乎直线

**4.** 游戏结束，开始释放内存

**5.** 一段时间后内存(等待 GC 回收)回到游戏开始前的水平 (如果游戏中申请的内存本就不希望释放，则可以测试多次，观察整体曲线是否呈上升趋势)

这代表除了在游戏一开始的加载环节的申请内存，整个游戏过程中没有频繁的 GC，也没有内存泄露的问题。

> 如果曲线呈"波浪线"或者有大量的"锯齿"，说明游戏过程中反复的申请和释放对象。这是频繁 GC 的表现。在游戏中呈现的效果则是反复的帧率下降->帧率恢复
> 如果曲线一直呈上升趋势，表示游戏过程中一直在申请内存，说明存在内存泄露。这可能会导致游戏的崩溃和闪退现象

此外，我们还能在内存信息采样中看到某个具体的方法申请的内存，再根据业务逻辑对函数的调用时机或者频率进行优化。

![img](https://arkimg.ark.online/104216cifza0a0dd7a05q7.webp)

