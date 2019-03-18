<template>
    <section class="c-line"

    >

        <canvas class="lineCanvas" ref="lineCanvas"
                :style="{width:newCan.w+'px',height:newCan.h+'px',top:newCan.y+'px',left:newCan.x+'px'}"
        ></canvas>
        <p>{{`X:${x},Y:${y},W:${w},H:${h},`}}</p>
        <p>{{`dsx:${dsx},dsy:${dsy},dex:${dex},dey:${dey},`}}</p>
        <p>{{`newLine sx:${newLine.sx}`}}</p>
        <p>{{`newLine sy:${newLine.sy}`}}</p>
        <p>{{`newLine ex:${newLine.ex}`}}</p>
        <p>{{`newLine ey:${newLine.ey}`}}</p>

        <vue-draggable-resizable
                :x="dsx-5"
                :y="dsy-5"
                :w="10"
                :h="10"
                :resizable="false"
                :minw="10"
                :minh="10"
                @dragging="onS"
        >
        </vue-draggable-resizable>
        <vue-draggable-resizable
                :x="dex-5"
                :y="dey-5"
                :w="10"
                :h="10"
                :resizable="false"
                :minw="10"
                :minh="10"
                @dragging="onE"
        >
        </vue-draggable-resizable>
    </section>
</template>
<script>
    import VueDraggableResizable from 'vue-draggable-resizable-gorkys'

    export default {
        components: {VueDraggableResizable},
        props: {
            lineData: {
                type: Object,
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
                w: 0,
                h: 0,
                y: 0,
                x: 0,
                dsx: 0,
                dsy: 0,
                dex: 0,
                dey: 0,
                img: '',
                type: '',
                isEdit: false,
                newLine: false
            }
        },
        computed: {
            newCan: function () {
                const {sx, sy, ex, ey} = this.newLine
                const w = Math.abs(sx - ex)
                const h = Math.abs(sy - ey)
                let x=0;
                let y=0;
                return {x, y, w, h}
            }
        },
        methods: {
            onS: function (x, y) {
                this.newLine.sx = x
                this.newLine.sy = y
            },
            onE: function (x, y) {
                this.newLine.ex = x
                this.newLine.ey = y
            },
            draw: function () {
                console.log('111')
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
                this.line()


            },
            line: function () {
                const {w, h} = this
                const {sx, sy, ex, ey} = this.lineData
                this.dsx = sx < ex ? 1 : w - 1
                this.dsy = sy < ey ? 1 : h - 1
                this.dex = sx > ex ? 1 : w - 1
                this.dey = sy > ey ? 1 : h - 1

                this.newLine = {sx: this.dsx, sy: this.dsy, ex: this.dex, ey: this.dey}

                this.ctx.beginPath()
                this.ctx.moveTo(this.dsx, this.dsy)
                this.ctx.lineTo(this.dex, this.dey)
                this.ctx.stroke()
                this.ctx.closePath()
            },

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
    .c-line {
        position: absolute;

        .draggable {
            background-color: red;
            border-radius: 50%;
        }

        .lineCanvas {
            position: absolute;
            width: 100%;
            height: 100%;
            background-color: red;

        }
    }
</style>
