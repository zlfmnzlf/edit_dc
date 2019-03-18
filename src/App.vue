<template>

    <main id="app" class="app" @mousemove="onMovePage" @click="escTool">
        <div>
            <button @click="genImg">生成图片</button>
            <button @click="setTool('line')">线</button>
            <button @click="setTool('circle')">圆</button>
            <button @click="setTool('rect')">矩形</button>
            <button>册除</button>
        </div>
        <icon-list class="icon-list" @sel="selImg"></icon-list>
        <section class="img-wrap" @contextmenu.stop.prevent="onRightClick">
            <!--事件接收-->
            <div v-if="newImg" class="tool-mask img"
                 :class="{'move':newTool}"
                 @click.stop="addImg"
            ></div>
            <div v-if="newTool" class="tool-mask line"
                 :class="{'move':newTool}"
                 @mousedown.stop="sDarw"
                 @mouseup="eDarw"
                 @mousemove="onMoveTool"

            ></div>
            <!--绘制-->
            <vue-draggable-resizable
                    v-for="(item,key) in tmpJson" :key="`tmp${key}`"
                    :x="item.x"
                    :y="item.y"
                    :w="item.w"
                    :h="item.h"
                    :z="key+100"
                    :resizable="item.isSize"
                    :active="item.active"
                    :minw="20"
                    :minh="20"
                    :parent="true"
                    :snap="true"
                    :is-conflict-check="true"
            >
                <tmp-darw class="lineJson"
                          :lineData="item" ref="lineCanvas"
                ></tmp-darw>

            </vue-draggable-resizable>
            <!--线条-->
            <vue-draggable-resizable
                    class="noBorder line"
                    v-for="(item,key) in lineJson" :key="`line${key}`"
                    :x="item.x"
                    :y="item.y"
                    :w="item.w"
                    :h="item.h"
                    :z="key+100"
                    :resizable="false"
                    :active="item.active"
                    :minw="20"
                    :minh="20"
                    :parent="true"
            >
                <line-darw class="lineJson" ref="canImg"  :lineData="item"></line-darw>
                <span class="del" @click="lineJson.splice(key,1)">删</span>

            </vue-draggable-resizable>
            <!--图形-->
            <vue-draggable-resizable
                    v-for="(item,key) in canvasJson" :key="`can${key}`"
                    :x="item.x"
                    :y="item.y"
                    :w="item.w"
                    :h="item.h"
                    :z="key+100"
                    :resizable="item.isSize"
                    :active="item.active"
                    :minw="20"
                    :minh="20"
                    :parent="true"
                    @resizing="(x,y,w,h)=>{item.w= w;item.h=h}"
            >
                <canvas-darw class="lineJson" ref="canImg" :w="item.w" :h="item.h" :type="item.type"></canvas-darw>
                <span class="del" @click="canvasJson.splice(key,1)">删</span>

            </vue-draggable-resizable>

            <!--图片-->
            <vue-draggable-resizable
                    v-for="(item,key) in imgJson" :key="`img${key}`"
                    :x="item.x"
                    :y="item.y"
                    :w="item.w"
                    :h="item.h"
                    :z="key+100"
                    :resizable="item.isSize"
                    :active="item.active"
                    :minw="20"
                    :minh="20"
                    :parent="true"
                    :snap="true"
                    :is-conflict-check="true"
            >
                <div class="img" :style="{backgroundImage:`url('${item.img}')`}">
                    <span class="del" @click="imgJson.splice(key,1)">删</span>
                </div>
            </vue-draggable-resizable>
            <!--信息点-->
            <vue-draggable-resizable
                    v-for="(item,key) in infoJson" :key="`info${key}`"
                    :x="item.x"
                    :y="item.y"
                    :w="item.w"
                    :h="item.h"
                    :z="key+2000"
                    :resizable="item.isSize"
                    :active="item.active"
                    :minw="20"
                    :minh="20"
                    :parent="true"
                    :snap="true"
                    :is-conflict-check="true"
                    :handles="['mr','ml']"
            >
                <div @click="currentInfoAttr=item">
                    <span class="del" @click="infoJson.splice(key,1)">删</span>
                    <p class="info-box text" v-if="item.type=='text'">
                        <span>{{item.name}}</span> <span></span>
                    </p>
                    <p class="info-box label" v-if="item.type=='label'">
                        <span :style="{backgroundColor:item.bg}">{{item.name}}</span>
                    </p>
                    <div class="info-box chart" v-if="item.type=='chart'">
                        <h1>{{item.name}}</h1>
                        <p>
                            图<br>图
                        </p>
                    </div>
                </div>
            </vue-draggable-resizable>

            <div class="table-wrap">
                <div>
                    <p class="head">PCS</p>
                    <table>
                        <thead>
                        <tr>
                            <th>名称</th>
                            <th>测量值</th>
                            <th>设定值</th>
                            <th width="100">备注</th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr v-for="(item,key) in pcsJson" :key="key">
                            <td><span class="del" @click="onDelPCS(key)">删</span>{{item.name}}</td>
                            <td>{{item.testVal}}</td>
                            <td>{{item.setVal}}</td>
                            <td>{{item.desc}}</td>
                        </tr>
                        <tr>
                            <td></td>
                            <td></td>
                            <td></td>
                            <td>
                                <button @click="onAddPCS">+添加</button>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                    <p class="tip-text red">注：红色为不可设置，黄色为可设置</p>
                </div>
                <div>
                    <p class="head">BMS</p>
                    <table>
                        <thead>
                        <tr>
                            <th>名称</th>
                            <th>测量值</th>
                            <th>设定值</th>
                            <th width="100">备注</th>
                        </tr>
                        </thead>
                        <tbody>
                        <tr v-for="(item,key) in bmsJson" :key="key">
                            <td><span class="del" @click="onDelBMS(key)">删</span>{{item.name}}</td>
                            <td>{{item.testVal}}</td>
                            <td>{{item.setVal}}</td>
                            <td>{{item.desc}}</td>
                        </tr>
                        <tr>
                            <td></td>
                            <td></td>
                            <td></td>
                            <td>
                                <button @click="onAddBMS">+添加</button>
                            </td>
                        </tr>
                        </tbody>
                    </table>
                </div>
            </div>


        </section>
        <section class="attr">
            <p>属性编辑</p>
            <p>名称：{{currentInfoAttr.name}}</p>
            <p>TODO:{{currentInfoAttr}}</p>
        </section>
        <div class="newImg" v-if="newImg"
             :style="{top:(newImg.y+1)+'px',left:(newImg.x+1)+'px',width:newImg.w+'px',height:newImg.h+'px',backgroundImage:`url('${newImg.img}')`}"
        ></div>

        <canvas class="canvas" ref="mainCanvas"></canvas>
        <div class="rightMenu" ref="rightMenu"
             v-if="isShowRightMenu"
             :style="{top:isShowRightMenu.y+'px',left:isShowRightMenu.x+'px'}"
        >
            <ul>
                <li @click="addInfo('text')">添加信息点</li>
                <li @click="addInfo('label')">添加标签</li>
                <li @click="addInfo('chart')">添加图表</li>
            </ul>
        </div>
    </main>
</template>

<script>
    import Vue from 'vue'
    import tmpDarw from './components/draw'
    import canvasDarw from './components/canvas'
    import iconList from './components/icon-list'
    import lineDarw from './components/line'
    import VueDraggableResizable from 'vue-draggable-resizable-gorkys'
    // import 'vue-draggable-resizable-gorkys/dist/VueDraggableResizable.css'

    export default {
        name: 'app',
        components: {tmpDarw, lineDarw, canvasDarw, iconList, VueDraggableResizable},
        created: function () {
            this.$nextTick(function () {
                this.$canvas.width = 1024
                this.$canvas.height = 768
                this.ctx = this.$canvas.getContext('2d')
            })
        },
        computed: {
            $canvas: function () {
                return this.$refs.mainCanvas
            },
            $tmpCanvas: function () {
                return this.$refs.tmpCanvas
            }
        },
        data: function () {
            return {
                isShowRightMenu: false,
                ctx: {},
                tmpCtx: {},
                newImg: false,
                newTool: false,
                isDarw: false,
                currentLineIndex: false,
                currentInfoAttr: {},
                imgJson: [],
                tmpJson: [],
                lineJson: [],
                canvasJson: [],
                infoJson: [],
                pcsJson: [],
                bmsJson: []
            }
        },
        methods: {
            escTool: function () {
                this.newImg = false
                this.isShowRightMenu = false
                // this.newTool = false
                // this.isDarw = false
            },
            setTool: function (type) {
                this.newTool = type
            },
            selImg: function (arg, item) {
                this.newTool = false
                this.isDarw = false
                const ev = arg[0]
                const {x, y} = ev
                const {img, w, h, isSize} = item
                this.newImg = {img, w, h, isSize, x, y, active: true}
                // this.imgJson.push({img, w, h, isSize, x: 0, y: 0,active:true})
            },
            addInfo: function (type) {
                const {sx, sy} = this.isShowRightMenu
                let _ = {x: sx, y: sy, type: type, isSize: false}
                switch (type) {
                    case 'text':
                        _.name = prompt("请选择信息点", "")
                        _.w = 300
                        _.h = 20
                        break
                    case 'label':
                        _.name = prompt("请输入标签名称", "")
                        _.w = _.name.length * 16 + 10
                        _.h = 20
                        break
                    case 'chart':
                        _.name = prompt("请输入图表名称", "")
                        _.w = 200
                        _.h = 200
                        break
                }
                this.infoJson.push(_)
                this.isShowRightMenu = false
            },
            addImg: function (ev) {
                if (this.newImg) {
                    this.newImg.x = ev.offsetX
                    this.newImg.y = ev.offsetY
                    this.imgJson.push(this.newImg)
                    this.newImg = false
                }
            },
            sDarw: function (ev) {
                if (this.newTool) {
                    this.isDarw = true
                    this.tmpJson.push({
                        ix: ev.x,
                        iy: ev.y,
                        sx: ev.offsetX,
                        sy: ev.offsetY,
                        ex: 0,
                        ey: 0,
                        type: this.newTool
                    })
                    this.currentLineIndex = this.tmpJson.length - 1
                }

            },
            eDarw: function (ev) {
                this.isDarw = false
                if (this.newTool) {
                    const data = this.tmpJson[this.currentLineIndex]
                    const {ix, iy, sx, sy, type} = data
                    const ex = ev.offsetX
                    const ey = ev.offsetY
                    if (this.newTool !== 'line') {
                        const {x, y, w, h, type} = this.$refs.lineCanvas[this.currentLineIndex]
                        this.canvasJson.push({x, y, w, h, type})
                        this.tmpJson.splice(this.currentLineIndex, 1)
                    } else {
                        const {x, y, w, h} = this.$refs.lineCanvas[this.currentLineIndex]
                        this.lineJson.push({x, y, w, h,ix, iy, sx, sy, ex, ey, type})
                        this.tmpJson.splice(this.currentLineIndex, 1)
                    }
                }
                this.newTool = false

            },
            onMovePage: function (ev) {
                const {x, y} = ev
                if (this.newImg) {
                    const {img, w, h, isSize, active} = this.newImg
                    this.newImg = {img, w, h, isSize, x, y, active}
                }

            },
            onMoveTool: function (ev) {
                if (this.newTool && this.isDarw) {
                    const data = this.tmpJson[this.currentLineIndex]
                    const {ix, iy, sx, sy, type} = data
                    const ex = ev.offsetX
                    const ey = ev.offsetY
                    Vue.set(this.tmpJson, this.currentLineIndex, {ix, iy, sx, sy, ex, ey, type})
                }
            },
            onRightClick: function (ev) {
                this.newTool = false
                this.isDarw = false
                this.isShowRightMenu = {x: ev.x, y: ev.y, sx: ev.offsetX, sy: ev.offsetY}

            },
            onAddBMS: function () {
                this.bmsJson.push({
                    name: '名称BMS',
                    testVal: '0.0',
                    setVal: '0.0',
                    desc: '备注'
                })
            },
            onDelBMS: function (index) {
                this.bmsJson.splice(index, 1)
            },
            onAddPCS: function () {
                this.pcsJson.push({
                    name: '名称PCS',
                    testVal: '0.0',
                    setVal: '0.0',
                    desc: '备注'
                })
            },
            onDelPCS: function (index) {
                this.pcsJson.splice(index, 1)
            },
            _loadImage: function (src) {
                return new Promise((resolve, reject) => {
                    const image = new Image();
                    image.src = src;
                    if (image.complete) {
                        resolve(image)
                    }
                    image.onload = function () {
                        resolve(image)
                    };
                    image.onerror = function (e) {
                        reject('Error: image error!', e)
                    };
                })

            },
            genImg: function () {
                this.imgJson.forEach(item => {
                    this._loadImage(item.img).then(img => {
                        const {x, y, w, h} = item
                        this.ctx.drawImage(img, x, y, w, h)
                    }).catch(e => {
                        console.log(e)
                    })
                })
                this.$refs.lineCanvas.forEach(item => {
                    const subCanvas = item.$refs.lineCanvas
                    const {x, y, w, h} = item
                    this.ctx.drawImage(subCanvas, x, y, w, h)
                })
            }


        }
    }
</script>

<style lang="scss">
    * {
        box-sizing: border-box;
    }

    html, body {
        padding: 0;
        margin: 0;
    }

    .app {
        position: relative;
        width: 1542px;
        margin: 0 auto;

        .icon-list {
            float: left;
            width: 245px;
        }

        .img-wrap {
            position: relative;
            float: left;
            width: 1024px;
            height: 768px;
            background-color: #218a8a;

            .move {
                cursor: crosshair;
            }

            .del {
                position: absolute;
                right: -20px;
                display: none;
            }

            .draggable.active {
                border: 1px solid red;
                /*&.active{*/
                    /*border: none;*/
                /*}*/
                .del {
                    display: inline-block;
                }

            }

            .img {
                width: 100%;
                height: 100%;
                background-size: contain;
                background-repeat: no-repeat;
                background-position: center center;
            }

            .canvas {
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                background: red;
            }

            .tool-mask {
                position: absolute;
                top: 0;
                left: 0;
                width: 100%;
                height: 100%;
                z-index: 9999;
            }

            .info-box {
                margin: 0;
                cursor: pointer;

                &.label {
                    display: inline-block;
                    background-color: green;
                    padding: 0 4px;
                    color: #fff;
                }
            }

            .table-wrap {
                position: absolute;
                top: 20px;
                right: 20px;
                color: #fff;
                font-size: 14px;
                font-weight: normal;

                & > div {
                    margin-bottom: 20px;
                }

                .red {
                    color: red;
                }

                .tip-text {
                    margin: 0;
                    font-size: 12px;
                }

                .head {
                    margin: 0;
                    color: red;
                    font-size: 16px;
                    font-weight: bold;
                    text-align: center;
                }

                table {
                    border-collapse: collapse;
                }

                table td, table th {
                    border: 1px solid #333;
                    padding: 0 4px;

                    .del {
                        display: inline-block;
                        width: 25px;
                        margin-left: -25px;
                    }
                }

                table th {
                    text-align: center;
                }

                table tr:hover td {
                    background-color: rgba(red, 0.1);
                }
            }

        }

        .attr {
            float: left;
            width: 245px;
        }

        .newImg {
            position: fixed;
            top: 0;
            left: 0;
            background-size: contain;
            background-repeat: no-repeat;
            background-position: center center;
            background-color: red;
            z-index: 10000;
        }

        .lineMousemove {
            position: fixed;
            top: 0;
            left: 0;
            width: 20px;
            height: 20px;
            background-size: contain;
            background-repeat: no-repeat;
            background-position: center center;
            background-color: red;

        }

        .rightMenu {
            position: fixed;
            top: 0;
            left: 0;
            width: 156px;
            background: #ccc;
            z-index: 99999;

            li {
                cursor: pointer;
            }
        }
    }
</style>
