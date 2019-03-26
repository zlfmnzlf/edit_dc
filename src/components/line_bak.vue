<template>
    <section class="c-line"

    >

        <canvas class="lineCanvas" ref="lineCanvas"
                :style="{width:newCan.w+'px',height:newCan.h+'px',top:newCan.y+'px',left:newCan.x+'px'}"
        ></canvas>
        <vue-draggable-resizable
                :x="dsx"
                :y="dsy"
                :w="10"
                :h="10"
                :resizable="false"
                :minw="10"
                :minh="10"
                @dragging="onS"
                @dragstop="onMoveEnd"
        >
            <span class="dan"></span>
        </vue-draggable-resizable>
        <vue-draggable-resizable
                :x="dex"
                :y="dey"
                :w="10"
                :h="10"
                :resizable="false"
                :minw="10"
                :minh="10"
                @dragging="onE"
                @dragstop="onMoveEnd"

        ><span class="dan "></span>

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
                let x = sx > ex ? ex : sx
                let y = sy > ey ? ey : sy
                return {x, y, w, h}
            }
        },
        methods: {
            onMoveEnd: function () {
                const o = this.lineData
                const n = this.newCan
                const l = this.newLine
                console.log('oooo-----', o.x, o.y)
                console.log('nnnn=====', n.x, n.y)
                const _ = {
                    x: o.x + n.x,
                    y: o.y + n.y,
                    w: n.w,
                    h: n.h
                }
                _.sx = _.x + l.sx
                _.sy = _.y + l.sy
                _.ex = _.x + l.ex
                _.ey = _.y + l.ey
                this.$emit('change', _)
            },
            onS: function (x, y) {
                this.newLine.sx = x
                this.newLine.sy = y
            },
            onE: function (x, y) {
                this.newLine.ex = x
                this.newLine.ey = y
            },
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
                this.line()


            },
            line: function () {
                const {w, h} = this
                const {sx, sy, ex, ey} = this.lineData
                this.dsx = sx < ex ? 0 : w
                this.dsy = sy < ey ? 0 : h
                this.dex = sx > ex ? 0 : w
                this.dey = sy > ey ? 0 : h

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
            },
            newCan: function (val) {
                const {w, h} = val
                const {sx, sy, ex, ey} = this.newLine
                const dsx = sx < ex ? 0 : w
                const dsy = sy < ey ? 0 : h
                const dex = sx > ex ? 0 : w
                const dey = sy > ey ? 0 : h
                this.$refs.lineCanvas.width = w
                this.$refs.lineCanvas.height = h
                this.ctx.clearRect(0, 0, w, h);
                this.ctx.strokeStyle = "black"
                this.ctx.lineWidth = 2
                this.ctx.beginPath()
                this.ctx.moveTo(dsx, dsy)
                this.ctx.lineTo(dex, dey)
                this.ctx.stroke()
                this.ctx.closePath()
            }

        }
    }
</script>
<style lang="scss">
    .c-line {
        position: absolute;

        .draggable {
            border: none !important;

            .dan {
                display: block;
                width: 100%;
                height: 100%;
                margin-left: -50%;
                margin-top: -50%;
                background-color: red;
                border-radius: 50%;
            }
        }

        .lineCanvas {
            position: absolute;
            width: 100%;
            height: 100%;
        }
    }
</style>
