<template>
<div class="wrap">
    <div class="canvas-wrap">
        <canvas class="paper" ref="paper" width="300" height="180"></canvas>
        <canvas class="block" ref="block" width="300" height="180"></canvas>
    </div>

</div>
</template>

<script>
export default {

    props:{
        diff:{
            default:0,
            type:Number
        }
    },

    data() {
        return {
            x: 200,
            y: this.getRandom(20, 90),
        }
    },

    watch:{
        diff(newV){
            let v = newV - 180
            //console.log(v)
            this.$refs.block.style.transform = `translateX(${v}px)`
        }
    },

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

}
</script>

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
