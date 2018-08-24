![](https://user-gold-cdn.xitu.io/2018/8/24/1656786fbb17c007?w=1080&h=879&f=jpeg&s=103791)

Painter是由[酷家乐移动前端团队](https://github.com/Kujiale-Mobile)打造的一款小程序绘图组件。

原项目地址：<https://github.com/Kujiale-Mobile/Painter>

新版地址：[https://github.com/shesw/Painter](https://github.com/shesw/Painter)

这款交互版原来是为了针对业务中的新需求而由我自己开发的，后来需求改动，所以并没有用上。组里[大佬](https://github.com/CPPAlien)考虑种种原因（主要是项目没用上，=0=～～），让我先在自己的github上开源。这版painter与原版的区别在于：

1. 添加了交互事件。Painter本质是以canvas为基础的，小程序的canvas有许多限制。允许canvas上元素的交互点击事件，可以实现更为便捷的功能，比如原来需要在canvas上添加功能按钮，现在可以直接画在canvas上
2. 添加拖拽元素的功能。目前这个功能没有完善好，因为它的滑动动作会与小程序的全屏滑动事件冲突，因此，拖拽功能在固定的页面上效果才好，如在拖拽时设置overflow: hidden等。



##### 这里将新版的Painter称为dancing-painter。引入方式请参考readme和demo。



#### 演示：

![](https://user-gold-cdn.xitu.io/2018/8/24/1656787f44caf810?w=328&h=480&f=gif&s=2575274)



#### 主要功能：

指原版的painter的功能。这些功能依然是本项目的主(实)要(用)功能。

##### 简介：

原版的使用简介请参见 [https://juejin.im/post/5b40b158e51d4518f543c7b0](https://juejin.im/post/5b40b158e51d4518f543c7b0)

简单来讲，使用过程如下图所示，可以结合demo来看：

![](https://user-gold-cdn.xitu.io/2018/8/23/1656781c023e1f01?w=1921&h=1867&f=png&s=399088 )

距离首次开源Painter库已经有一段时间了，这期间获益于各路道友的帮助和提点，Painter进行了几波更新（[原项目地址](https://github.com/Kujiale-Mobile/Painter))：

##### 新增特性：

1. 增加align属性，可以使任意元素可以实现左中右对齐。
2. 加入文字换行的能力。对一段文字设置width或者maxLines，都有可能触发文字的换行。
3. 添加文字的一些属性：fontWeight, textDecoration, textStyel(fill, stroke), maxLines, lineHeight
4. 图片mode属性，实现图片裁剪、缩放，默认为aspectFill
5. 图片不设置width, heighti属性，使用默认宽高
6. left, right, top, bottom对负数的支持

##### 修复问题：

1. 某些机型上切边会出现黑线。
2. 安卓机型上圆角无法显示
3. 使用文件前检查文件是否正常
4. 二维码大小显示异常



#### 交互功能：

这一版的特色主要是具备元素的点击事件实现以及拖拽功能，做出来以后因为项目上暂时用不上，所以感觉功能上可能比较鸡肋。不过还蛮好玩的😄

在demo中称之为dancing-painter。

**强调：**要使用交互功能，有一下两点需要注意：

1. 导入palette时，使用绝对路径方法导入
2. palette类，以module.exports输出。

原因：小程序页面向组件传值时，会把对象参数做一个类似于JSON.parse(JSON.stringfy())的拷贝，导致对象中的函数在传递后丢失。因此在dancing-painter中，选择讲palette（即生成图片的json代码）的路径传递给painter组件，在组件内require到这个文件，再合成所需的json数据。

导入代码如下：

```js
    //直接导入数据
	// const template = new DCard().palette(); 
	//导入绝对路径
    const template = {
      path: '/palette/dancing-card.js',
      data: {},
    };
    this.setData({
      template: template,
    });
```

palette：

```js
//位于/palette/dancing-card.js
class PaletteCard {
    context = {}
    constructor(data) {
        this.data = data;
    }
    palette() {
        return{
            ...
        }
    }
}
module.exports = PaletteCard;
```



##### 实现点击效果

Painter的元素绘制是以json的形式给出的，其交互行为和拖拽效果也定义在相应的json文件里。

在需要设定某一个元素的点击事件的时候，只需要在其相应的json代码里加入methods属性即可。该属性支持一下几种点击方式：

| name       | description |
| :--------- | :---------- |
| tap        | 点击        |
| longpress  | 长按        |
| touchstart | 开始触摸    |
| touchmove  | 移动        |
| touchend   | 触摸结束    |

每一个方法可以返回一个boolean值，以表示是否拦截该事件。在dancing-painter中，如果在页面上元素有重叠，则方法的传递默认是由下而上的；在任意一个元素的methods方法中，可以返回true来拦截该事件。

##### 使用拖拽功能

使用animation属性，目前只支持拖拽能力。由于存在与屏幕的滑动冲突（如果有大神知道怎么截获屏幕的滑动事件，如长页面的滚动，请千万不吝赐教告诉我哈），需要在使用时固定住整个页面，如设置overflow: hidden。

使用：设置animation:{drag:true}

##### 示例：

```js
{
      width: '700rpx',
      height: '1000rpx',
      background: '#eee',
      views: [{
        type: 'qrcode',
        content: 'https://github.com/Kujiale-Mobile/Painter',
        css: {
          top: '40rpx',
          left: '180rpx',
          color: 'red',
          borderWidth: '10rpx',
          borderColor: 'blue',
          width: '120rpx',
          height: '120rpx',
          align: 'center',
        },
        methods: {
          tap() {
            console.log('qrcode');
          },
        }
      },
      {
        type: 'rect',
        css: {
          top: '40rpx',
          left: '180rpx',
          color: 'green',
          borderRadius: '20rpx',
          borderWidth: '10rpx',
          width: '120rpx',
          height: '120rpx',
        },
        methods: {
          tap() {
            // 调用当前页面的function方法
            const pages = this.getCurrentPages();
            const currentPage = pages[pages.length-1];
            currentPage.function();
          },
        },
        animation: {
          drag: true,
        },
      }]
}
```



##### 事件冒泡

事件默认是冒泡传递的。示例如下：

上面的代码中是两个部分重叠的二维码和方形：

![](https://user-gold-cdn.xitu.io/2018/8/23/16567818bd5b8887?w=370&h=228&f=png&s=24998)



点击二者的重合部分，控制台输出：

![](https://user-gold-cdn.xitu.io/2018/8/23/165678181b85eee8?w=328&h=104&f=png&s=10677)



##### 性能

拖拽功能是一种动画现象，涉及到canvas的重绘。经过测试，在IDE上重绘速度很快，在真机上有数量级的差别。

动画效果的连贯性主要是由canvas.draw()的速度决定的。

下图展示了在demo主页上，canvas.draw()方法在ide和真机上运行的时间差别（单位：ms）：

IDE:					真机（华为荣耀8）：

![](https://user-gold-cdn.xitu.io/2018/8/23/16567817f3710756?w=184&h=1106&f=png&s=28274 ) 			 ![](https://user-gold-cdn.xitu.io/2018/8/23/1656781801208348?w=200&h=952&f=png&s=35604)



#### 结语

感谢[大佬](https://github.com/CPPAlien)在开发中对我的无限帮助。

感谢 [demi520](https://github.com/demi520) 的 [wxapp-qrcode](https://github.com/demi520/wxapp-qrcode) 库，Painter 中二维码绘制部分使用了该库的部分代码，并做了些修改。

另外这里有一篇[wiki](https://github.com/Kujiale-Mobile/Painter/wiki/mpvue-%E6%8E%A5%E5%85%A5%E6%96%B9%E5%BC%8F)简单介绍了怎样在mpvue中使用Painter。

**坑：**：mpvue在更新某一个页面元素的值的时候，会同时把所有data中存在的元素都更新一遍。

这就造成了[这个问题](https://github.com/Kujiale-Mobile/Painter/issues/24)：Painter绘制完成后，会触发onImgOK函数，传出图片的url。这时将该url传入某image的src中去，同时就会触发Painter的template的再赋值，从而导致无限重绘。



**最后，作为程序员届的新手和菜鸟，诚邀各路大神用issue和建议砸死我😂**

**最后，作为程序员届的新手和菜鸟，诚邀各路大神用issue和建议砸死我😂**

**最后，作为程序员届的新手和菜鸟，诚邀各路大神用issue和建议砸死我😂**



##### demo传送门：[https://github.com/shesw/Painter](https://github.com/shesw/Painter)

##### 使用参考（原版）： [https://juejin.im/post/5b40b158e51d4518f543c7b0](https://juejin.im/post/5b40b158e51d4518f543c7b0)

------------------