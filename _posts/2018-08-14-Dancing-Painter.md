title: 小程序绘图-升级版Painter-一段json生成分享图-支持点击和拖拽事件

date: 2018-08-14

author: 生忘

subtitle: 在原有painter的基础上，引入交互事件

tags: [小程序，canvas]

## categories: 【小程序，canvas，分享图，交互】

距离首次开源Painter库已经有一段时间了，这期间获益于各路道友的帮助和提点，Painter进行了几波更新（[原项目地址：](https://github.com/Kujiale-Mobile/Painter)， [使用参考](https://juejin.im/post/5b40b158e51d4518f543c7b0))：

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

此外，也更新了[demo](https://github.com/Kujiale-Mobile/Painter)，并切探索了一下Painter用在其它框架中的可行性。

新功能一览：

![](http://sinacloud.net/music-store/markdownPic/ts1534220572855Screen%20Shot%202018-08-14%20at%2012.22.19%20PM.png?KID=sina%2C2o3w9tlWumQRMwg2TQqi&ssig=gDHsB48dzJ&Expires=1535084616 )





这里有一篇[wiki](https://github.com/Kujiale-Mobile/Painter/wiki/mpvue-%E6%8E%A5%E5%85%A5%E6%96%B9%E5%BC%8F)简单讲述了怎样在mpvue中使用Painter。

！！！请注意！！！：mpve的实现并不能完全贴近原生vue的开发，例如mpvue在更新某一个页面元素的值的时候，会同时把所有data中存在的元素都更新一遍。

这就造成了[这个问题](https://github.com/Kujiale-Mobile/Painter/issues/24)：Painter绘制完成后，会触发onImgOK函数，传出图片的url。这时将该url传入某image的src中去，同时就会触发Painter的template的再赋值，从而导致无限重绘。

##### demo地址：[https://github.com/Kujiale-Mobile/Painter](https://github.com/Kujiale-Mobile/Painter)

##### 使用参考： [https://juejin.im/post/5b40b158e51d4518f543c7b0](https://juejin.im/post/5b40b158e51d4518f543c7b0)

------------------

#### 交互版Painter -- Dancing-Painter

Painter是伴随着[酷咖名片小程序](http://help.kujiale.com/hc/kb/article/1163606/)的需求应运而生的。接下去为了更好地服务于实际应用，我们研发人员自作主张地认为Painter还需要一些交互的功能。这一版的Painter拥有以下几个新特性：

1. 能够给任意元素添加点击事件，包括tap, longpress, touchstart, touchmove, touchend 五种。
2. 能够实现元素的拖拽。

这版Painter因为一些原因没有合到我司的主项目上去，还在我自己的github上。不过组里老大让我就这么发了。老大的话总是不会错的。嗯。

#### 使用

本项目的demo地址为：[DancingPainter](https://github.com/shesw/Painter)

核心库地址：[dancingPainterCore](https://github.com/shesw/PainterCore.git)

若要使用demo，直接clone即可；

在自己的项目里使用Painter，可以有三种方式：

1. 下载.zip包解压，把painter作为组件引入您自己的项目
2. submodule管理方式。git代码：

```shell
git submodule add https://github.com/shesw/PainterCore.git components/painter
```

3. subtree管理方式。

```shell
git subtree add --prefix=components/painter https://github.com/shesw/PainterCore.git master
```

具体的操作请自行google。

总之Painter就是一个原生形式开发的组件，引入项目后，就和组件一样使用了:

* 作为组件加入：

```js
"usingComponents":{
  "painter":"/components/painter/painter"
}
```

* 页面内引用：

```html
<painter palette="{{data}}" bind:imgOK="onImgOK" />
```

* bind回调：

```js
bind:imgOK="onImgOK"
bind:imgErr="onImgErr"

onImgOK(e) {
  其中 e.detail.path 为生成的图片路径
},
```



#### 传入数据

Painter是通过pallete来向组件传递绘图所需要的信息的。在上一版的操作中，需要先准备一个palette文件，再在js中生成包含绘图信息的json文件，然后通过palette传入Painter：

```js
import Card from '../../palette/card';


this.setData({
 	data: new Card().palette(),
});
```

这一版保留了这种导入方式，同时新定义了通过palette文件绝对路径来向Painter传递信息的方式。并且，只有通过这种方式，才能使用painter的交互功能。

至于原因，小程序在由页面向组件传递对象数据的时候，应该是用了stringfy的方法来拷贝数据，致使对象中的方法都丢失。

```js
const template = {
      path: '/palette/dancing-card.js',
      data: {},
    };
    this.setData({
      data: template,
});
```

传入路径的方式如上所示。painter会在其内部根据路径require到palette的信息，并组合为painter所需要的json格式数据。

必须使用绝对路径。

可以向data里传入数据，在palette生成的时候使用。



#### palette的写法

注意两点：

1. painter的路径必须为/components/painter/。原因的话，有兴趣瞅一眼代码就知道了哈哈哈哈哈

2. 由于在painter内部用require方法获取，在palette代码的最后，需要加上：

   ```js
   module.exports = 类名;
   ```



#### 实现点击效果

Painter的元素绘制是以json的形式给出的，其交互行为和拖拽效果也定义在相应的json文件里。

因此在需要设定某一个元素的点击事件的时候，只需要在其相应的json代码里加入methods属性即可。该属性支持一下几种点击方式：

| name       | description |
| :--------- | :---------- |
| tap        | 点击        |
| longpress  | 长按        |
| touchstart | 开始触摸    |
| touchmove  | 移动        |
| touchend   | 触摸结束    |

每一个方法可以返回一个boolean值，以表示是否拦截该事件。在dancing-painter中，如果在页面上元素有重叠，则方法的传递默认是由下而上的；在任意一个元素的methods方法中，可以返回true来拦截该事件。

例如：

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
            console.log('rect');
          },
        },
        animation: {
          drag: true,
        },
      }]
}
```



上例中是两个部分重叠的二维码和方形：

![](http://sinacloud.net/music-store/markdownPic/ts1534253500835Screen%20Shot%202018-08-14%20at%209.31.06%20PM.png?KID=sina%2C2o3w9tlWumQRMwg2TQqi&ssig=kJP3wCmKYV&Expires=1535117577)



点击二者的重合部分，控制台输出：

![](http://sinacloud.net/music-store/markdownPic/ts1534253500835Screen%20Shot%202018-08-14%20at%209.31.12%20PM.png?KID=sina%2C2o3w9tlWumQRMwg2TQqi&ssig=aLzBHkpGUY&Expires=1535117578)





