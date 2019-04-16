<template>
    <section class="c-canvas-darw"
             :style="{width:w+'px',height:h+'px'}"
    >
        <canvas class="lineCanvas" ref="lineCanvas"></canvas>
    </section>
</template>
<script>
    export default {
        props: {
            w: {
                type: Number,
                default: 0
            },
            h: {
                type: Number,
                default: 0
            },
            x: {
                type: Number,
                default: 0
            },
            y: {
                type: Number,
                default: 0
            },
            bg: {
                type: String || Boolean,
                default: ''
            },
            txt: {
                type: String,
                default: ''
            },
            color: {
                type: String,
                default: ''
            },
            font: {
                type: Number,
                default: 30
            },
            type: {
                type: String
            }
        },
        created: function () {
            this.$nextTick(function () {
                this.ctx = this.$refs.lineCanvas.getContext('2d')
                this.draw()
            })
        },
        data: function () {
            return {
                ctx: false,
            }
        },
        methods: {
            draw: function () {
                this.$refs.lineCanvas.width = this.w
                this.$refs.lineCanvas.height = this.h
                this.ctx.clearRect(0, 0, this.w, this.h);
                this.ctx.strokeStyle = "black"
                this.ctx.fillStyle = this.bg
                switch (this.type) {
                    case 'rect':
                        this.rect()
                        break
                    case 'txt':
                        this.txtDraw()
                        break
                    case 'circle':
                        this.circle()
                        break
                }
            },
            txtDraw: function () {
                const {w, h} = this
                if (this.bg) this.rect()
                this.ctx.fillStyle = this.color;
                this.ctx.textAlign = "left";
                this.ctx.font = this.font + "px sans-serif";
                let {width: nWidth} = this.ctx.measureText(this.txt); // TextMetrics object
                nWidth = Math.ceil(nWidth)
                if (w != nWidth || h != this.font) {
                    this.$refs.lineCanvas.width = nWidth
                    this.$emit('fixWidth', nWidth, this.font)
                }
                this.ctx.fillText(this.txt, 0, h - 4);
            },
            rect: function () {
                const {w, h} = this
                this.ctx.lineWidth = 2
                if (this.bg) {
                    this.ctx.fillRect(0, 0, w, h);
                } else {
                    this.ctx.strokeRect(0, 0, w, h);
                }
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
                this.ctx.lineWidth = 1
                this.ctx.beginPath()
                this.ctx.moveTo(x - a, y);
                this.ctx.bezierCurveTo(x - a, y - oy, x - ox, y - b, x, y - b);
                this.ctx.bezierCurveTo(x + ox, y - b, x + a, y - oy, x + a, y);
                this.ctx.bezierCurveTo(x + a, y + oy, x + ox, y + b, x, y + b);
                this.ctx.bezierCurveTo(x - ox, y + b, x - a, y + oy, x - a, y);
                this.ctx.closePath();
                if (this.bg) {
                    this.ctx.fill();
                } else {
                    this.ctx.stroke();
                }
            }
        },
        watch: {
            w: function () {
                this.draw()
            },
            h: function () {
                this.draw()
            },
            bg: function () {
                this.draw()
            },
            color: function () {
                this.draw()
            },
            font: function () {
                this.draw()
            },
            txt: function () {
                this.draw()
            }
        }
    }
</script>
