<template>
<div class="wrap">
    <div class="progress-wrap">
        <div ref="inner" class="inner-bar"></div>
        <div ref="button" class="button" @mousedown="down" @mousemove="move" @mouseup="up"></div>
    </div>
</div>
</template>

<script>
export default {

    data() {
        return {
            isDown: false,
            x: 0,
            curX: 0,
            diff: 0,
        }
    },

    mounted() {
        this.$nextTick(() => {
            let that = this
            document.addEventListener('mouseup', that.up)
            document.addEventListener('mousedown', that.down)
        })
    },
    watch: {
        diff(newV) {
            this.$emit('change', newV)
        }
    },
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

}
</script>

<style lang="less" scoped>
.wrap {
    padding: 20px 0 10px;

    .progress-wrap {
        height: 30px;
        background-color: #ccc;
        border-radius: 15px;
        position: relative;

        .inner-bar {
            height: 30px;
            background-color: #999;
            width: 0;
            border-top-left-radius: 15px;
            border-bottom-left-radius: 15px;
            z-index: 1;
        }

        .button {
            height: 40px;
            width: 40px;
            position: absolute;
            transform: translateX(0);
            top: -5px;
            border-radius: 100%;
            border: 1px solid #ccc;
            background-color: #fff;
            cursor: pointer;
            z-index: 2;
        }
    }

}
</style>
