# slide-check

## 介绍

> 使用Vue开发的一个滑块验证组件


## 进度

* 2019.11.13

    * 完成图片模块的编写

    * 滑动模块编写

> 完成后的效果图:       


![](https://github.com/bettermu/blog-picture-store/blob/master/20191113/1.gif?raw=true)

> 目前还没有加上一些边界的判断，参考了下bilibili网站的实现，后续时间会完善

* 2019.11.16
    * 对组件进行了容错的相关完善


## 如何用vue手写一个拼图滑块的组件

### 想手写一个的原因

> 一般来说，网站只要涉及到用户相关的，登录一定会有需要输入验证码之类的功能，用来防止非人为操作的指令。大部分网站实现这部分都是会使用一张五颜六色的验证码图片，用户通过输入图中的文字或者字母，来通过登录请求。

> 还有一种方式，类似bilibili网站的验证操作一样，是通过拼图滑块拖动实现的验证：

![](https://github.com/bettermu/blog-picture-store/blob/master/20191113/bilibili.png?raw=true)

> 第一眼看到它的时候，不由得会好奇，究竟是怎么实现的，这种拼图的样式和交互。期初我以为是两张图片，都是后台返回来的，然后再给个距离的合适值，前端只需要对比移动的距离值就可以判断。但后来仔细想想，这种方式还没有前端使用canvas绘图来的方便，但是具体怎么做，怎么实现，自己的脑子里暂时是没有具体的思路的。

> 平时写业务遇到多的验证基本都是图片码验证的那种，太过平常的交互，秉着不断学习的心态，决定自己一定要手写一个这样的验证组件，写之前，有需要准备的，首先，去canvasAPI，温习一下相关的canvas知识：[传送门](https://www.canvasapi.cn/)，然后呢，我们需要对其中的一些功能，进行一些详细的思路分析，为我们开始手写，做好充分的准备。

### 整体的思路梳理

> 大致看一下，想象一下具体的交互和流程，对其进行解耦：

* 图片模块：
    * 底图的实现
    * 碎片的实现
* 滑动模块：
    * 按钮拖动的实现 

* 共享值：滑动距离
* 边界值计算
* 容差值计算

> 上面大概分析了下，整个组件分为两个部分，一个是图片模块，可以看出来，它有两个部分，一个是底图，另一个是碎片。其中涉及的值为移动的距离，而移动距离这个值可以通过第二个部分，滑动模块来获取到，因此这里值的共享和触发，使用父子传值$emit实现，另外，两边分别对该值进行watch，来分别触发对应的位置更新。包括边界值和容差值，都是需要根据这个距离值来计算。

```
<template>
  <div id="app">
    <Paper></Paper>
    <Slide></Slide>
  </div>
</template>
```

### 拼图模块的实现

> 在写这部分内容之前，先要弄清楚，如果是使用canvas来实现，应该怎样去做？
> 图片模块有底图和碎片，两个部分，肯定是使用两个canvas去实现，并且，两个canvas是需要绝对定位的:

```
<template>
<div class="wrap">
    <div class="canvas-wrap">
        <canvas class="paper" ref="paper" width="300" height="180"></canvas>
        <canvas class="block" ref="block" width="300" height="180"></canvas>
    </div>

</div>
</template>

<style lang="less" scoped>
.canvas-wrap {
    width: 300px;
    height: 180px;
    margin: 0 auto;
    position: relative;

    canvas {
        position: absolute;
        top: 0;
        left: 0;
    }

    .block {
        transform:translateX(-180px)
    }
}
</style>
```

> 碎片的canvas绝对定位，并且left是个往左偏移的负值，这里我们没有使用left去实现，而是用了性能更好一点的transform。

> 至于两张canvas大小一模一样，因为其实看上去底图和碎片是两个部分，但其实使用的是同一张图，只是canvas绘制的原理不一样而已。

> 画布有了，这时候我们需要一张图片了，也就是用来绘制到canvas上面的图片文件，我们可以在mounted里面，create一个img元素:

```

mounted() {
        this.$nextTick(() => {
            let paper = this.$refs.paper
            let block = this.$refs.block
            let ctx = paper.getContext('2d')
            let block_ctx = block.getContext('2d')
            let img = document.createElement('img')
            img.onload = () => { 
                ctx.drawImage(img, 0, 0, 300, 180)
                this.draw(ctx,'fill')
                //绘制碎片形状
                this.draw(block_ctx,'clip') //这里要注意，先画路径，之后再填充图片
                block_ctx.drawImage(img, 0, 0, 300, 180)
            }
            img.src = require("../assets/bg-img.jpg")

        })
    },
    
```
> 这里，我们在nextTick的回调里进行相关的dom操作，是为了保证这个时候，dom确实都已经加载渲染完成。我们获取两个canvas，分别获取对应的上下文，生成一个Img元素，给它的src赋值，注意，我们必须要在图片加载的回调里，进行相关的绘制工作，否则，canvas画布肯定是空白一片的。

> 在img的onload里，进行两个部分的绘制处理，暂时先不看具体的绘制操作，我们先理一下两个部分的区别和共同点：共同点很明显，就是绘制路径一模一样，这部分我们可以共用，区别就是，一个是填充空白，另一个是去掉区域外的其他图片像素，下面我们来看下，具体的绘制逻辑:

```
methods: {
        //获取随机范围
        getRandom(min, max) {
            let c = max - min + 1
            return Math.random() * c + min
        },

        draw(ctx,opt) {
            ctx.beginPath()
            ctx.moveTo(this.x, this.y)
            ctx.lineTo(this.x + 10, this.y)
            //绘制上方弧形
            ctx.arc(this.x + 25, this.y, 10, this.deg2arc(-180), this.deg2arc(0))
            ctx.lineTo(this.x + 50, this.y)
            ctx.lineTo(this.x + 50, this.y + 10)
            //绘制右侧弧形
            ctx.arc(this.x + 50, this.y + 25, 10, this.deg2arc(-90), this.deg2arc(90))
            ctx.lineTo(this.x + 50, this.y + 50)
            ctx.lineTo(this.x, this.y + 50)
            ctx.lineTo(this.x, this.y + 40)
            //绘制左侧弧形,true表示逆时针绘制
            ctx.arc(this.x, this.y + 25, 10, this.deg2arc(90), this.deg2arc(-90), true)
            ctx.lineTo(this.x, this.y)
            ctx.lineWidth = 1;
            ctx.fillStyle = "rgba(255, 255, 255, 1)";
            ctx.strokeStyle = "rgba(255, 255, 255, 1)";
            ctx.stroke();
            ctx[opt]() 
            ctx.globalCompositeOperation = "xor";
        },
        //角度转弧度
        deg2arc(deg) {
            return deg / 180 * Math.PI
        }
    }

```

> 这里主要逻辑就是，首先画路径，其中canvas.arc这个函数的最后一个参数，代表是否逆时针方向绘制，这个在我们绘制左侧凹进去的弧形的时候，很有作用。我们吧'fill'和'clip'当做参数传入，最后的一步调用不同的处理方式，fill就是填充的意思，默认填充白色。clip则是将画布沿着路径裁剪，最后我们在裁剪之后的画布上绘制原来的图片，并且ctx.globalCompositeOperation = "xor"，是改变混合模式，意思就是，选择图片与路径重合的区域展示，其余区域则透明。

> 这样我们就能看到，其实两张canvas都展示了，为了表现明显一些，加了一点描边，突出：

![](https://github.com/bettermu/blog-picture-store/blob/master/20191113/2.png?raw=true)

### 滑动模块的实现

> 滑块模块的实现，主要思路就是，监听鼠标的mousedown mousemove mouseup事件，这里为了保持移动的顺滑，我们需要将监听代理到document对象上面，同时我们需要在up事件中，移出掉move事件，这就意味着，我们的事件函数，需要是具名的，因为removeListener需要的是具名函数参数。

```
<template>
<div class="wrap">
    <div class="progress-wrap">
        <div ref="inner" class="inner-bar"></div>
        <div ref="button" class="button" @mousedown="down" @mousemove="move" @mouseup="up"></div>
    </div>
</div>
</template>

```

```
methods: {

        //按下按钮处理
        down(e) {
            if (e.target === this.$refs.button) {
                this.isDown = true
                this.x = e.pageX
                document.addEventListener("mousemove", this.move)
            }

        },

        //移动按钮处理
        move(e) {
            if (this.isDown) {
                    this.curX = e.pageX
                    this.diff = (this.curX - this.x) < -2 ? 0 : (this.curX - this.x) > 220 ? 220 : this.curX - this.x
                    if (this.diff > -2 && this.diff < 260) {
                        this.$refs.inner.style.width = (this.diff + 20) + 'px'
                        this.$refs.button.style.transform = `translateX(${this.diff}px)`
                    }
            }
        },
        //鼠标抬起处理
        up() {
            if (this.diff > 178 && this.diff < 182) {
                alert("验证成功")
            }
            document.removeEventListener('mousemove', this.move)
            this.reset()
        },

        reset() {
            this.isDown = false
            this.diff = 0
            this.$refs.inner.style.width = 20 + 'px'
            this.$refs.button.style.transform = `translateX(0)`
        }

    }
```

> 我们通过mousedown记录起始点的pageX,在移动滑块的时候，t计算move的事件对象的pageX与起点的差值，来移动滑块，改变滑块的transform，并且改变进度条的颜色值。

### 关于差值在组件间的传递和实时监听

> 还有一个重要的一点，现在这两个组件各自的交互和样式，都实现了，那么如何去让滑块事件产生的diff差值，传递到图片模块去，让碎片跟随滑块移动？这里就要介绍下vue组件之间的传值方式，这里使用的是emit的方式，slide组件，watch diff的变化，并且触发change事件：回传给父组件，父组件拿到diff值之后，传入paper组件，paper组件里，通过watch diff的变化，来改变碎片的偏移值：

```
//slide.vue
watch: {
        diff(newV) {
            this.$emit('change', newV)
        }
    },
    
//父组件
<div id="app">
    <Paper :diff="diff"></Paper>
    <Slide @change="change"></Slide>
  </div>
  
 //paper.vue
 watch:{
        diff(newV){
            let v = newV - 180
            this.$refs.block.style.transform = `translateX(${v}px)`
        }
    },
```

### 最终效果   

![](https://github.com/bettermu/blog-picture-store/blob/master/20191113/1.gif?raw=true)

### 需要注意的一些点

> 需要注意的点:

* 首先，canvas的api要做到了解和熟悉，至少，路径绘制和混合模式，要清楚，这是拼图的核心实现。特别是圆弧的逆时针绘制，参数要掌握。
* 滑动事件的原理，需要document委托，还有，进行标志位判断，只有在鼠标按下之后，move的事件才能触发，并且在up事件里，及时移出掉监听。
* 在img的onload事件中，进行canvas绘制，否则画布不会有任何的响应。
* 需要对滑块的偏移进行限制，计算出边界值，防止滑块超出预料的范围。

### 总结

> 纸上得来终觉浅，绝知此事要躬行。只有自己亲自试试，才能更熟悉其中的一些原理和技巧。关于canvas的东西，光看文档，作用不太大，亲自试了，感受才能更深刻。
