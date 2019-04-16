<template>
    <section class="c-line" :class="{hover:hover}">

        <div class="wrap"
             :style="{left:-tmpLine.x+'px',top:-tmpLine.y+'px'}"
        >
            <div class="hover"
                 :style="{width:(tmpLine.w-10)+'px',height:(tmpLine.h-10)+'px',left:(tmpLine.x+5)+'px',top:(tmpLine.y+5)+'px'}"
                 @mouseenter="showDan"
            ></div>
            <canvas class="lineCanvas" ref="lineCanvas" @mouseleave="hideDan"></canvas>
            <vue-draggable-resizable
                    :x="tmpLine.sx"
                    :y="tmpLine.sy"
                    :w="10"
                    :h="10"
                    :resizable="false"
                    :draggable="draggable"
                    :minw="10"
                    :minh="10"
                    :parent="true"
                    @dragging="onS"
                    @dragstop="onMoveEnd"
                    @mouseenter="hover= true"
            >
                <div class="tooltip">
                    <a title="删除" class="opp-del" @click="infoJson.splice(key,1)"></a>
                </div>
                <span class="dan"></span>
            </vue-draggable-resizable>
            <vue-draggable-resizable
                    :x="tmpLine.ex"
                    :y="tmpLine.ey"
                    :w="10"
                    :h="10"
                    :resizable="false"
                    :draggable="draggable"
                    :minw="10"
                    :minh="10"
                    :parent="true"
                    @dragging="onE"
                    @dragstop="onMoveEnd"
                    @mouseenter="hover= true"

            ><span class="dan "></span>

            </vue-draggable-resizable>
        </div>


    </section>
</template>
<script>
    import VueDraggableResizable from 'vue-draggable-resizable-gorkys'

    export default {
        components: {VueDraggableResizable},
        props: {

            draggable: {
              type: Boolean,
              default: true
            },
            lineData: {
                type: Object,
            }
        },
        created: function () {
            this.$nextTick(function () {
                this.$refs.lineCanvas.width = 1024
                this.$refs.lineCanvas.height = 682
                this.ctx = this.$refs.lineCanvas.getContext('2d')
                this.ctx.strokeStyle = "black"
                this.ctx.lineWidth = 1
                this.draw()
            })
        },
        data: function () {
            return {
                hover: false,
                tHover: false,
                tmpLine: false,
            }
        },
        computed: {},
        methods: {
            showDan: function () {
                window.clearTimeout(this.tHover)
                this.hover = true
            },
            hideDan: function () {
                window.clearTimeout(this.tHover)
                this.tHover = setTimeout(() => {
                    this.hover = false
                }, 500)
            },
            onMoveEnd: function () {
                let {sx, sy, ex, ey} = this.tmpLine
                const x = sx < ex ? sx : ex
                const y = sy < ey ? sy : ey
                const w = Math.abs(sx - ex)
                const h = Math.abs(sy - ey)
                this.$emit('change', {x, y, w, h,active:false})
            },
            onS: function (x, y) {
                this.tmpLine.sx = x
                this.tmpLine.sy = y
            },
            onE: function (x, y) {
                this.tmpLine.ex = x
                this.tmpLine.ey = y
            },
            draw: function () {
                window.clearTimeout(this.tHover)
                this.tmpLine = this.lineData
                const {sx, sy, ex, ey} = this.tmpLine
                this.ctx.clearRect(0, 0, this.$refs.lineCanvas.width, this.$refs.lineCanvas.height)
                this.ctx.beginPath()
                this.ctx.moveTo(sx, sy)
                this.ctx.lineTo(ex, ey)
                this.ctx.stroke()
                this.ctx.closePath()

            }
        },
        watch: {
            hover: function (val) {
                if (val) {
                    window.clearTimeout(this.tHover)
                }
            },
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
        position: relative;
        width: 100%;
        height: 100%;
        overflow: hidden;

        .hover {
            position: absolute;
            z-index: 2;
        }

        .wrap {
            position: absolute;
            width: 1024px;
            height: 682px;
        }

        .draggable {
            display: none;
            border: none !important;
            z-index: 3 !important;

            .dan {
                display: block;
                width: 100%;
                height: 100%;
                margin-left: -50%;
                margin-top: -50%;
                background-color: red;
                border-radius: 50%;
                cursor: pointer;
            }

            .del {
                display: block !important;
            }
        }

        .lineCanvas {
            position: absolute;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
    }

    .active .c-line {
        overflow: visible;

        .draggable {
            display: block;
            .tooltip{
                margin-top: -10px;
                margin-left: -12px;
                display: block;
            }
        }
    }
</style>
