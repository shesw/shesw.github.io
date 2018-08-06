title: kooHandler preHandle

date: 2018-08-06

author: 生忘

subtitle: kooHandler对元素的预处理，使得pen能够支持动态元素

tags: [小程序]

## categories: 【小程序，canvas，dev design】



现在考虑两点。

* 在kooHandler中，根据拥有事件的类型对元素进行分类，使得以后每一次交互事件都能减少大量的遍历搜索操作。
* 在kooHandler中，识别出动态组件（拥有animation属性），对其进行标记。

以上两点，可以在一次遍历的条件下分类完成。

#### 元素根据事件分类

目前支持的事件以及分类以及分类池：

|事件名|事件池名|备注|
|:---|---|---|
|tap|tapPool|拥有tap事件的元素|
|longpress|longpressPool|拥有longpress事件的元素|
|touchstart|touchstartPool|拥有touchstart事件的元素|
|touchmove|touchmovePool|拥有touchmove事件的元素|
|touchend|touchendPool|拥有touchend事件的元素|

此后，每一次emit()，都直接从对应的池中去取元素。



#### 分离动态元素和静态元素

如何对动态元素进行标记，是一个迷人的问题。

1. palette属性中携带animation

| 属性       | 赋值 | 备注 |
| :---- | ---- | ---------: |
| fingerMove |  boolean    | 是否能随手指移动     |
|translate|destX,destY，duration|平移|
|scale|centerX,centerY,ratio(0~1)，duration|缩放|
|rotate|centerX,centerY,angle(0~360)，duration|旋转|
|set|{}|动画组合|

在animation中，这些属性会编写按顺序执行（fingerMove除外）。示例如下：

```json
{
    animation:{
        fingerMove: true,
        translate: '50 50 200',
        scale: '50 50 2 1000',
        rotate: '50 50 180 1000',
        set:{
            duration: 1000,
            scale: '100 100 3',
            rotate: '100 100 360'
        }
    }
}
```



2. 标记传值

   在kooHandler中标记好了这些动态的view，接下来要告诉pen。问题开始展现她的真正的迷人之处。太迷了。一脸懵。

3. pen获取到了这些动态元素的信息后，先将静态元素绘制到画布上，调用canvas的draw()。同时将动态元素保存在一个元素池中，在静态元素完成绘制后，进行draw(true)