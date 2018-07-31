title: Painter Event Dev Design

date: 2018-07-27

author: 生忘

subtitle: painter的点击触发事件设计

tags: [小程序]

## categories: 【小程序，canvas】



事件类型包括：

| event name | description |
| ---------- | ----------- |
| tap        | 点击        |
|longpress|长按|
|touchstart|触屏|
|touchmove|移动|
|touchend|离开屏幕|



##### 在Painter的框架中，由palette决定画面整体的借口和样式。为了能使主canvas响应相对应的点击事件，对palette进行扩展：

1. 在各个view的属性中，添加methods对象作为新的属性，该对象响应、处理页面上对应的点击事件。例如:

   ```js
   {
       type:'image',
       url:'/res/img.png',
       css:{
   		width:'100rpx',
           height:'100rpx',
           left:0,
           right:0
       },
       methods:{
           //    methods     
           tap(){},
           longpress(){},
           touchstart(){},
           touchmove(){},
           touchend(){}
       }
   }
   ```

   

2. 每一个view，都能够接受以上类型的5个事件，不写则不触发相应事件。



##### 在pen.js中的扩展：

1. 对于canvas上触发的点击事件，对其进行定位，分发到对应的view。

   监听canvas组件的所有事件，根据事件发生的坐标，决定分配给那一个组件。

   分配机制：
   ```flow
   st=>start: Start
   e=>end: End
   op1=>operation: 获取整个canvas的点击事件
   op2=>operation: 根据点击区域过滤出拥有该事件的元素(view)，并排出它们消耗该事件的顺序，依次放入序列。
   op3=>operation: 从序列中取出第一个元素，消耗该元素的对应事件
   con1=>condition: 当前元素是否拦截当前事件？
   op4=>operation: 消除序列中所有元素
   op5=>operation: 从序列中移除当前元素
   st->op1->op2->op3->con1
   con1(yes)->op4->e
   con1(no)->op5->op3
   ```

2. 支持事件的冒泡与拦截

   事件的冒泡与拦截机制见流程图；事件是自上而下传递或是相反，取决于元素事件序列的排序规则

3. 支持canvas全图点击事件的获取。

   canvas上任意一点上触发的点击事件，都可以是一个全图点击事件。获取整个canvas的点击事件后，先对全图的点击事件进行消耗。全图事件也可以被拦截，这样的话，其它的任何组件都得不到点击事件。

   在上述流程图中，添加环节：

   ```flow
   st=>start: start
   e=>end: e
   op1=>operation: 获取整个canvas的点击事件
   op3=>operation: 消耗全图点击事件
   con=>condition: 是否拦截
   op2=>operation: 根据点击区域过滤出拥有该事件的元素(view)，并排出它们消耗该事件的顺序，依次放入序列
   st->op1->op3->con
   con(yes)->e
   con(no)->op2
   ```

   

4. 动态改变一个元素（view）的位置以及其它状态。
   对于某一个元素位置的动态控制能力。这一点是我认为最复杂的一点，在canvas上控制元素的移动和状态，就意味着不断地刷新重绘这个元素。可行的思路有一下两点：

   1. 每一次移动都完全重绘整个页面。毫无疑问，这对性能的损耗可能是无法接受的。
   2. 移动的操作：擦除原先的元素，再绘上新位置的元素。在绘制的时候，将静态元素先行绘制完毕，然后储存画布的当前状态，然后进行动态元素的绘制。


