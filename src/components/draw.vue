<template>
    <section class="c-darw"
             :style="{width:w+'px',height:h+'px',top:y+'px',left:x+'px'}"
    >

        <canvas class="lineCanvas" ref="lineCanvas"></canvas>

    </section>
</template>
<script>
    export default {
        props: {
            lineData: {
                type: Object,
            }
        },
        created: function () {
            this.$nextTick(function () {
                this.ctx = this.$refs.lineCanvas.getContext('2d')
            })
        },
        data: function () {
            return {
                ctx: false,
                w: 0,
                h: 0,
                y: 0,
                x: 0,
                img: '',
                type: ''
            }
        },
        methods: {
            draw: function () {
                const {sx, sy, ex, ey, type} = this.lineData
                this.w = Math.abs(sx - ex)
                this.h = Math.abs(sy - ey)
                this.x = sx < ex ? sx : ex
                this.y = sy < ey ? sy : ey
                this.type = type

                this.$refs.lineCanvas.width = this.w
                this.$refs.lineCanvas.height = this.h
                this.ctx.clearRect(0, 0, this.w, this.h);
                this.ctx.strokeStyle = "black"
                this.ctx.lineWidth = 2
                switch (type) {
                    case 'line':
                        this.line()
                        break
                    case 'rect':
                        this.rect()
                        break
                    case 'circle':
                        this.circle()
                        break
                }

            },
            line: function () {
                this.w = 1024
                this.h = 682
                this.x = 0
                this.y = 0
                this.$refs.lineCanvas.width = this.w
                this.$refs.lineCanvas.height = this.h
                // const {w, h} = this
                const {sx, sy, ex, ey} = this.lineData
                // const dsx = sx < ex ? 1 : w - 1
                // const dsy = sy < ey ? 1 : h - 1
                // const dex = sx > ex ? 1 : w - 1
                // const dey = sy > ey ? 1 : h - 1

                this.ctx.beginPath()
                this.ctx.moveTo(sx, sy)
                this.ctx.lineTo(ex, ey)
                this.ctx.stroke()
                this.ctx.closePath()
            },
            rect: function () {
                const {w, h} = this
                this.ctx.strokeRect(0, 0, w, h);
            },
            circle: function () {
                const {w, h} = this
                const x = w / 2
                const a = x - 1
                const y = h / 2
                const b = y - 1
                var k = .5522848,
                    ox = a * k, // 水平控制点偏移量
                    oy = b * k; // 垂直控制点偏移量</p> <p> ctx.beginPath();
                //从椭圆的左端点开始顺时针绘制四条三次贝塞尔曲线
                this.ctx.beginPath()
                this.ctx.moveTo(x - a, y);
                this.ctx.bezierCurveTo(x - a, y - oy, x - ox, y - b, x, y - b);
                this.ctx.bezierCurveTo(x + ox, y - b, x + a, y - oy, x + a, y);
                this.ctx.bezierCurveTo(x + a, y + oy, x + ox, y + b, x, y + b);
                this.ctx.bezierCurveTo(x - ox, y + b, x - a, y + oy, x - a, y);
                this.ctx.closePath();
                this.ctx.stroke();
            }
        },
        watch: {
            lineData: {
                deep: true,
                handler: function () {
                    this.draw()
                }
            }
        }
    }
</script>
<style lang="scss">
    .c-darw {
        position: absolute;

        .lineCanvas {
            width: 100%;
            height: 100%;
        }
    }
</style>
