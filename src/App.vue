<template>

    <main id="app" class="app" @mousemove="onMovePage" @click="escTool">
        <div class="clearfix">
            <button @click="genImg">测试生成图片</button>
            <button>保存</button>
        </div>
        <section class="clearfix">
            <div class="icon-list">
                <icon-list @sel="selImg"></icon-list>
                <tool-list @sel="setTool"></tool-list>
            </div>
            <div class="edit-wrap">
                <div class="head"></div>
                <section class="img-wrap" @click.self="onDeactivated" @contextmenu.stop.prevent="onRightClick">
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
                            :z="1000+key"
                            :resizable="item.isSize"
                            :active="item.active"
                            :minw="20"
                            :minh="20"
                            :parent="true"
                            :snap="true"
                            :is-conflict-check="true"
                    >
                        <tmp-darw class="lineJson"
                                  :lineData="item" ref="tmpCanvas"
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
                            :z="100+key"
                            :resizable="false"
                            :draggable="false"
                            :active="item.active"
                            :minw="20"
                            :minh="20"
                            :parent="true"
                            @activated="onActivated('line',item,key)"
                            @dragging="onDragLine(arguments,item,key)"
                    >
                        <line-darw class="lineJson" ref="lineImg" :lineData="item"
                                   @change="(nData)=>{item = Object.assign(item,nData)}"
                                   @del="lineJson.splice(key,1)"
                        ></line-darw>
                    </vue-draggable-resizable>
                    <!--图形-->
                    <vue-draggable-resizable
                            v-for="(item,key) in canvasJson" :key="`can${key}`"
                            :x="item.x"
                            :y="item.y"
                            :w="item.w"
                            :h="item.h"
                            :z="300+key"
                            :resizable="item.isSize"
                            :active="item.active"
                            :minw="5"
                            :minh="5"
                            :parent="true"
                            @activated="onActivated('can',item,key)"
                            @resizing="(x,y,w,h)=>{item.x= x;item.y=y;item.w= w;item.h=h}"
                            @dragging="(x,y)=>{item.x= x;item.y=y}"
                    >
                        <canvas-darw class="lineJson" ref="canImg" :w="item.w" :h="item.h" :x="item.x" :y="item.y"
                                     :type="item.type"></canvas-darw>
                        <span class="del" @click="canvasJson.splice(key,1)">删</span>
                    </vue-draggable-resizable>

                    <!--图片-->
                    <vue-draggable-resizable
                            v-for="(item,key) in imgJson" :key="`img${key}`"
                            :x="item.x"
                            :y="item.y"
                            :w="item.w"
                            :h="item.h"
                            :z="500+key"
                            :resizable="item.isSize"
                            :active="item.active"
                            :minw="5"
                            :minh="5"
                            :parent="true"
                            :snap="true"
                            :is-conflict-check="true"
                            @activated="onActivated('img',item,key)"
                            @dragging="(x,y)=>{item.x= x;item.y=y}"
                    >
                        <div class="img">
                            <img-canvas ref="imgImg" :img="item"></img-canvas>
                            <div class="tooltip">
                                <a title="删除" class="opp-del" @click="imgJson.splice(key,1)"></a>
                                <a title="左转90度" class="opp-r-90" @click="onRotateImg(item,key,90)"></a>
                                <a title="右转90度" class="opp-r-270" @click="onRotateImg(item,key,-90)"></a>
                                <a title="水平翻转" class="opp-horizontal" @click="onHorizontalImg(item,key)"></a>
                                <a title="垂直翻转" class="opp-vertical" @click="onVerticalImg(item,key)"></a>
                            </div>


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
                            @dragging="(x,y)=>{item.x= x;item.y=y}"
                            @activated="onActivated('info',item,key)"
                    >
                        <div>
                            <span class="del" @click="infoJson.splice(key,1)">删</span>
                            <p class="info-box text" v-if="item.type=='text'">
                                <span class="name">{{item.name}}</span>
                                <span class="val">{{item.val}}</span>
                                <span class="unit">{{item.unit}}</span>
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
                                    <td class="red">{{item.testVal}}</td>
                                    <td class="yellow">{{item.setVal}}</td>
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
                <p>提示：右击添加信息点/标签/图表</p>
            </div>

            <section class="attr">
                <p class="head">属性编辑</p>
                <div v-if="currentInfoAttr">
                    <p v-if="currentInfoAttr.type=='info'">
                        <label>信息点：</label>
                        <select>
                            <option value="1">信息点列表</option>
                        </select>
                    </p>
                    <p class="head">位置</p>
                    <p>
                        <label>X：</label>
                        <input type="number" v-model.number="currentInfoAttr.item.x">
                    </p>
                    <p>
                        <label>Y：</label>
                        <input type="number" v-model.number="currentInfoAttr.item.y">
                    </p>
                    <div v-if="currentInfoAttr.type=='can'&&currentInfoAttr.item.type!='line'">
                        <p>
                            <label>W：</label>
                            <input type="number" v-model.number="currentInfoAttr.item.w">
                        </p>
                        <p>
                            <label>H：</label>
                            <input type="number" v-model.number="currentInfoAttr.item.h">
                        </p>
                    </div>
                    {{currentInfoAttr.item}}
                    <!--<div>-->
                    <!--<p>-->
                    <!--<label>背景色：</label>-->
                    <!--<input type="number" value="1">-->
                    <!--</p>-->
                    <!--</div>-->
                </div>
            </section>
        </section>

        <div class="newImg" v-if="newImg"
             :style="{top:(newImg.y+1)+'px',left:(newImg.x+1)+'px',width:newImg.w+'px',height:newImg.h+'px',backgroundImage:`url('${newImg.img}')`}"
        ></div>

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

        <canvas class="canvas" ref="mainCanvas"></canvas>
    </main>
</template>

<script>
    import Vue from 'vue'
    import tmpDarw from './components/draw'
    import canvasDarw from './components/canvas'
    import imgCanvas from './components/img-canvas'
    import iconList from './components/icon-list'
    import toolList from './components/tool-list'
    import lineDarw from './components/line'
    import VueDraggableResizable from 'vue-draggable-resizable-gorkys'

    // import 'vue-draggable-resizable-gorkys/dist/VueDraggableResizable.css'

    export default {
        name: 'app',
        components: {tmpDarw, toolList, imgCanvas, lineDarw, canvasDarw, iconList, VueDraggableResizable},
        created: function () {
            this.$nextTick(function () {
                this.$canvas.width = 1024
                this.$canvas.height = 682
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
                currentInfoAttr: false,
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
                console.log('111', type)
                this.newTool = type
            },
            selImg: function (arg, item) {
                this.newTool = false
                this.isDarw = false
                const ev = arg[0]
                const {x, y} = ev
                const {img, w, h, isSize} = item
                this.newImg = {
                    img,
                    w,
                    h,
                    imgW: w,
                    imgH: h,
                    isSize,
                    x,
                    y,
                    rotate: 0,
                    active: true,
                    isVertical: false,
                    isHorizontal: false
                }
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
                        _.val = 3.0
                        _.unit = 'kWh'
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
            onHorizontalImg: function (item, key) {
                item.isHorizontal = !item.isHorizontal
                Vue.set(this.imgJson, key, item)

            },
            onVerticalImg: function(item, key){
                item.isVertical = !item.isVertical
                Vue.set(this.imgJson, key, item)
            },
            onRotateImg: function (item, key, deg) {
                const _ = Object.assign({}, item)
                let nDeg = _.rotate + deg
                if (nDeg >= 360) {
                    nDeg = 0
                }
                if (nDeg < 0) {
                    nDeg = 270
                }
                _.rotate = nDeg
                if (_.rotate == 90 || _.rotate == 270) {
                    console.log('RRRRRR', _.imgW, _.imgH)
                    _.w = _.imgH
                    _.h = _.imgW
                } else {
                    _.w = _.imgW
                    _.h = _.imgH
                }
                console.log('RRRRWH', _.w, _.h)
                Vue.set(this.imgJson, key, _)

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
                    const {sx, sy, type} = data
                    const ex = ev.offsetX
                    const ey = ev.offsetY
                    if (this.newTool !== 'line') {
                        const {x, y, w, h, type} = this.$refs.tmpCanvas[this.currentLineIndex]
                        this.canvasJson.push({x, y, w, h, type})
                        this.tmpJson.splice(this.currentLineIndex, 1)
                    } else {
                        const x = sx < ex ? sx : ex
                        const y = sy < ey ? sy : ey
                        const w = Math.abs(sx - ex)
                        const h = Math.abs(sy - ey)
                        this.lineJson.push({x, y, w, h, sx, sy, ex, ey, type, active: true})
                        this.tmpJson.splice(this.currentLineIndex, 1)
                    }
                }
                this.newTool = false

            },
            onMovePage: function (ev) {
                const {x, y} = ev
                if (this.newImg) {
                    const {img, w, h, imgW, imgH, rotate, isSize, active, isVertical, isHorizontal} = this.newImg
                    this.newImg = {img, w, h, x, y, imgW, imgH, rotate, isSize, active, isVertical, isHorizontal}
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
            onDragLine: function (xy, item, i) {
                const [x, y] = xy
                const ox = item.x
                const oy = item.y
                item.x = x
                item.y = y
                item.sx = item.sx - (ox - item.x)
                item.sy = item.sy - (oy - item.y)
                item.ex = item.ex - (ox - item.x)
                item.ey = item.ey - (oy - item.y)
                Vue.set(this.lineJson, i, item)
            },
            onActivated: function (type, item, key) {
                this.currentInfoAttr = {type, item, key}
            },
            onDeactivated: function () {
                this.currentInfoAttr = false
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
                this.ctx.clearRect(0, 0, 1024, 682)
                if (this.$refs.lineImg) {
                    this.$refs.lineImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        this.ctx.drawImage(subCanvas, 0, 0, 1024, 682)
                    })
                }
                if (this.$refs.canImg) {
                    this.$refs.canImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        const {x, y, w, h} = item
                        this.ctx.drawImage(subCanvas, x, y, w, h)
                    })
                }
                if (this.imgJson) {
                    this.$refs.imgImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        const {x, y, w, h} = item
                        this.ctx.drawImage(subCanvas, x, y, w, h)
                    })
                    // this.imgJson.forEach(item => {
                    //     this._loadImage(item.img).then(img => {
                    //         const {x, y, w, h} = item
                    //         this.ctx.drawImage(img, x, y, w, h)
                    //     }).catch(e => {
                    //         console.log(e)
                    //     })
                    // })
                }


            }


        },

    }
</script>

<style lang="scss">
    * {
        box-sizing: border-box;
    }

    html, body {
        padding: 0;
        margin: 0;
        background-color: #ccc;
    }

    .clearfix {
        &:before,
        &:after {
            content: " "; // 1
            display: table; // 2
        }

        &:after {
            clear: both;
        }
    }

    .app {
        position: relative;
        width: 1542px;
        margin: 0 auto;

        .icon-list {
            float: left;
            width: 156px;
        }

        .edit-wrap {
            position: relative;
            float: left;

            & > .head {
                height: 86px;
                background-color: #a9aa80;
            }
        }

        .img-wrap {
            position: relative;
            width: 1024px;
            height: 682px;
            background-color: #218a8a;

            .line:hover {
                z-index: 99999 !important;
            }

            .move {
                cursor: crosshair;
            }

            .tooltip {
                position: absolute;
                top: -45px;
                left: 0px;
                display: none;
                height: 35px;
                background-color: #ccc;
                border-radius: 5px;
                white-space: nowrap;

                a {
                    margin-right: 5px;
                }

                &:before {
                    position: absolute;
                    content: ' ';
                    display: block;
                    top: 35px;
                    left: 8px;
                    width: 0;
                    height: 0;
                    border-width: 10px 5px;
                    border-style: solid;
                    border-color: #ccc transparent transparent transparent;
                }
            }

            .del {
                position: absolute;
                top: -20px;
                left: 0px;
                display: none;
                cursor: pointer;
            }

            .rotate {
                position: absolute;
                top: -20px;

                left: 20px;
                display: none;
                cursor: pointer;
            }

            .draggable.active {
                border: 1px solid red;
                /*&.active{*/
                /*border: none;*/
                /*}*/
                .del, .rotate {
                    display: inline-block;
                }

                .tooltip {
                    display: block;
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

                &.text {
                    span {
                        display: inline-block;
                        margin-right: 10px;
                    }

                    .name {
                        color: #fff;
                    }

                    .val {
                        color: yellow;
                    }

                    .unit {
                        color: red;
                    }
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

                .yellow {
                    color: yellow;
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
                        right: auto;
                        left: -20px;
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
            width: 156px;
            background-color: #f0f0f0;
            padding-top: 5px;
            padding-right: 5px;
            border: 1px solid #494949;

            p {
                padding: 0 8px;

                label {
                    display: inline-block;
                    min-width: 40px;
                    text-align: right;
                }
            }

            .head {
                margin: 2px;
                padding: 0 10px;
                margin-bottom: 5px;
                font-size: 14px;
                box-shadow: 1px 1px 1px #7a7a7a;
            }
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

        .opp-del {
            display: inline-block;
            width: 33px;
            height: 33px;
            background-image: url('./assets/icon/opp-del.png');
        }

        .opp-r-90 {
            display: inline-block;
            width: 33px;
            height: 33px;
            background-image: url('./assets/icon/opp-r-90.png');
        }

        .opp-r-270 {
            display: inline-block;
            width: 33px;
            height: 33px;
            background-image: url('./assets/icon/opp-r-270.png');
        }

        .opp-horizontal {
            display: inline-block;
            width: 33px;
            height: 33px;
            background-image: url('./assets/icon/opp-horizontal.png');
        }

        .opp-vertical {
            display: inline-block;
            width: 33px;
            height: 33px;
            background-image: url('./assets/icon/opp-vertical.png');
        }
    }
</style>
