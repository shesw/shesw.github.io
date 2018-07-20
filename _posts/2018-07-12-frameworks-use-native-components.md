### 小程序wepy和mpvue框架中使用原生自定义组件
---
#### 前言
前几天把一个用于生成图片的、叫做painter的库开源了（https://juejin.im/post/5b40b158e51d4518f543c7b0），然后在评论区里发现，有不少人很关心能否在wepy或者mpvue框架里也使用它。

painter是基于原生开发方式而实现的一个自定义组件(Componnets)，而mpvue是可以支持使用原生自定义组件的。wepy还没有尝试过。



#### mpvue

##### 目录简介

主要关心代码在编译前（vue风格）和编译后（原生开发）的变化。

可以看一下本文demo项目的整体目录：

![](/Users/shesw/mygithub/csldev.github.io/myblog/res/f06.png)



src目录下是我们开发时用vue写的代码；dist目录下的内容是最后运行的代码，是原生开发形式的。可以看到dist里面的内容分为：

* pages。这一部分存放的是页面文件，即在src中被编译后的每个页面的wxml, js, wxss, json文件。但是这些文件不直接包含重要代码，而是通过import和require语句导入。json文件例外，它直接使用我们写在mpvue中的main.js里的config内容。


* components里存放的是template内容，它们会通过imort语法被页面的wxml文件调用。


* static目录。里面存放着两类内容。

第一类，是刚才说到的、页面所需要的css样式文件和js文件，它们通过import和require语法被引入页面。 

第二类，原本就放置在这个文件夹中的东西，会原封不动地写入dist目录下的static，如demo中的painter组件。



##### 在mpvue中使用painter

1. 安装mpvue，并新建项目。http://mpvue.com/mpvue/quickstart/
2. 编写所需要的页面：

``` js
<template>
    <div>
        <button @click="save">保存</button>
    </div>
</template>

<script>
export default {
  methods: {
    save () {
      console.log('on save click')
    }
  }
}
</script>
```

这是一个基础页面，只包含了一个按钮组件button。

3. 导入原生自定义组件，放在static目录下。

   本文以painter为例，可以运行代码：

```shell
   git submodule add https://github.com/Kujiale-Mobile/PainterCore.git static/painter
```

   也可以直接把组件放入static目录下。

4. 在编译之后的dist/static/目录下会存在和这里导入完全相同的painter组件。现在要考虑调用它。

5. 在demo/main.js中引入对组件的引用：

   ```js
   import Vue from 'vue'
   import App from './index'
   const app = new Vue(App)
   app.$mount()
   export default {
     config: {
       navigationBarTitleText: 'demo',
       usingComponents: {
         'painter': '/static/painter/painter'
       }
     }
   }
   ```

6. 在demo/index.vue中引用组件:

   ```vue
   <template>
       <div>
           <painter :customStyle="customStyle" :imgOK="onImgOk" :palette="template" />
           <button @click="save">保存</button>
       </div>
   </template>
   
   <script>
   export default {
     methods: {
       save () {
         console.log('on save click')
       },
       onImgOK (e) {
         console.log(e)
       }
     }
   }
   </script>
   ```

7. 此时编译成功。在这里，Painter这个组件需要配合palette进行使用，palette里的代码是用来编写图片内容，具体用法可参考 https://juejin.im/post/5b40b158e51d4518f543c7b0 。在项目/src/下创建palette目录，新建card.js文件。由于palette本身是js文件的形式，所以无需放在static中。

8. 使用palette/card.js为painter导入数据

   ```vue
   <template>
       <div>
           <painter :customStyle="customStyle" @imgOK="onImgOk" :palette="template" />
           <button style="margin-top:40rpx" @click="save">保存</button>
       </div>
   </template>
   
   <script>
   import Card from '../../palette/card'
   const card = new Card()
   const userInfo = {
     avatar: 'https://qhyxpicoss.kujiale.com/r/2017/12/04/L3D123I45VHNYULVSAEYCV3P3X6888_3200x2400.jpg@!70q'
   }
   
   const template = card.do(userInfo)
   const customStyle = 'margin-left:40rpx'
   
   export default {
     data () {
       return {
         customStyle: customStyle,
         template: template
       }
     },
     methods: {
       save () {
         console.log('on save click')
       },
       onImgOK (e) {
         console.log(e)
       }
     }
   }
   </script>
   ```
   编译后可以看到绘制的效果了。

9. 可能出现的错误：

   *  SyntaxError: Unexpected token export/import。es5用的是jsCommon，不支持import语法。开启项目设置的es6转es5即可。
   *  预览模式的真机在请求图片时网络错误，原因是预览模式下会校验域名。如果需要查看真机的效果，请使用远程调试模式，或者把图片的地址设为自己的安全域名下的地址。





本文demo地址：https://github.com/Kujiale-Mobile/Mpvue-Painter-Demo

