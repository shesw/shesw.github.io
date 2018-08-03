title: Painter Views Handler

date: 2018-07-31

author: 生忘

subtitle: painter对元素的管理机制设计

tags: [小程序]

## categories: 【小程序，canvas，dev design】


#### 获取事件
对主canvas进行监听。摘取以下交互事件：

|name|description|
|:---|:---|
|tap|点击|
|longpress|长按|
|touchstart|开始触摸|
|touchmove|移动|
|touchend|触摸结束|

事件冒泡：
1. catch
2. bind

当组件设置了catch事件，则其会拦截事件。因此，应该默认使用bind方式，允许冒泡，同时允许手动拦截事件。

手动拦截机制：每一个methods中调用的方法，都返回一个布尔值，来告诉kooHandler是否拦截事件。


#### 事件分发

在painter.js中获取点击事件，并将该事件封装，传入事件处理器KooHandler。

##### 封装事件event

```json
{ 	
	position: {x,y},
	type: "tap",
	mode: "bind"	
}
```

##### KooHandler结构。

kooHandler用于处理和分发事件。其生命周期为：

![](http://on-img.com/chart_image/5b616352e4b053a09c1ff85d.png)


1. 构造函数。接收所有组件元素的实例和组件上下文对象context.
2. 拥有一个组件池viewsPool，接收所有被传入的组件，kooHandler可以通过坐标来定位符合条件的组件们。
3. 一个接收事件的公共接口，获取点击事件。
4. 根据点击事件，在viewsPool中获取符合条件的所有元素，按照顺序放入运行池execPool。
5. 依次调用这些元素的相应方法，直到遍历完成，或事件被某个元素拦截。
6. 值得注意的是，在这里全屏事件的处理方式，与普通view保持一致。在实际操作中，将整个palette的json作为一个大view。
7. 在实际操作中，execPool可以不必放入完整的view，而是将这些view中符合条件的方法提取出来放入即可。

