<template>

    <main id="app" class="app" :class="{demo:!isEdit}" @mousemove="onMovePage" @click="escTool">
        <section class="clearfix">
            <div v-if="isEdit" class="icon-list">
                <icon-list @sel="selImg"></icon-list>
                <tool-list @sel="setTool"></tool-list>
            </div>
            <div class="edit-wrap">
                <div class="head clearfix">
                    <span v-if="isEdit">
                       <button class="opp-btn" @click="saveData"><i class="save"></i>保存</button>
                       <button class="opp-btn" @click="historyBack"><i class="save"></i>撤消</button>
                       <button class="opp-btn" @click="historyReset"><i class="save"></i>重做</button>
                    </span>
                    <span class="r">
                        <button class="opp-btn"
                                @click="isFullscreen=!isFullscreen;isFullscreen?fullscreen():exitFullscreen()">
                            <i class="" :class="!isFullscreen?'full':'full-a'"></i>全屏
                        </button>
                    </span>
                </div>
                <section class="img-wrap" @click.self="onDeactivated"
                         @contextmenu.stop.prevent="onRightClick"
                >
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
                    <div v-if="demoBg" class="demo-bg" :style="{backgroundImage:`url('${demoBg}')`}"></div>
                    <!--绘制-->
                    <vue-draggable-resizable
                            v-for="(item,key) in tmpJson" :key="`tmp${key}`"
                            :x="item.x"
                            :y="item.y"
                            :w="item.w"
                            :h="item.h"
                            :z="1000+key"
                            :resizable="item.isSize"
                            :active="isEdit?item.active:''"
                            :draggable="isEdit"
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
                            :z="500+key"
                            :resizable="false"
                            :draggable="false"
                            :active="isEdit?item.active:''"
                            :minw="20"
                            :minh="20"
                            :parent="true"
                            @activated="onActivated('line',item,key)"
                            @dragging="onDragLine(arguments,item,key)"
                    >
                        <line-darw class="lineJson" ref="lineImg" :lineData="item"
                                   @change="(nData)=>{item = Object.assign(item,nData)}"
                                   @del="lineJson.splice(key,1)"
                                   :draggable="isEdit"
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
                            :resizable="isEdit?item.isSize:false"
                            :active="isEdit?item.active:''"
                            :draggable="isEdit"
                            :minw="5"
                            :minh="5"
                            :parent="true"
                            @activated="onActivated('can',item,key)"
                            @resizing="(x,y,w,h)=>{item.x= x;item.y=y;item.w= w;item.h=h}"
                            @dragging="(x,y)=>{item.x= x;item.y=y}"
                    >
                        <canvas-darw class="lineJson" ref="canImg"
                                     @fixWidth="fixCanvasWidth(arguments,item)"
                                     :w="item.w" :h="item.h" :x="item.x" :y="item.y"
                                     :bg="item.bg" :type="item.type" :color="item.color" :font="item.font"
                                     :txt="item.txt"></canvas-darw>
                        <div class="tooltip">
                            <a title="删除" class="opp-del" @click="infoJson.splice(key,1)"></a>
                        </div>
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
                            :active="isEdit?item.active:''"
                            :draggable="isEdit"
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
                            :active="isEdit?item.active:''"
                            :draggable="isEdit"
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
                            <div class="tooltip">
                                <a title="删除" class="opp-del" @click="infoJson.splice(key,1)"></a>
                            </div>
                            <p class="info-box text" v-if="item.type=='text'">
                                <span class="name">{{item.info.cn_name}}</span>
                                <span class="val" v-if="valData[item.info.info_uuid]">{{valData[item.info.info_uuid].value}}</span>
                                <span class="unit">{{item.info.unittype_name}}</span>
                            </p>
                            <p class="info-box label" v-if="item.type=='label'">
                                <span :style="{backgroundColor:item.bg}">{{item.info.type_name}}</span>
                            </p>
                            <div class="info-box chart" v-if="item.type=='chart'">
                                <h1>{{item.info.cn_name}}</h1>
                                <chat v-if="chartData[item.info.info_uuid]"
                                      :lineData="chartData[item.info.info_uuid].val"
                                      :xAxisData="chartData[item.info.info_uuid].xAxis"></chat>
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
                                    <td><span class="del-btn" @click="onDelPCS(key)"><i class=""></i></span>{{item.name}}
                                    </td>
                                    <td class="red">{{item.testVal}}</td>
                                    <td class="yellow">{{item.setVal}}</td>
                                    <td>{{item.desc}}</td>
                                </tr>
                                <tr class="table-add-tr">
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
                                    <td><span class="del-btn" @click="onDelBMS(key)"><i class=""></i></span>{{item.name}}
                                    <td>{{item.testVal}}</td>
                                    <td>{{item.setVal}}</td>
                                    <td>{{item.desc}}</td>
                                </tr>
                                <tr class="table-add-tr">
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
                <p v-if="isEdit">提示：右击添加信息点/标签/图表</p>
            </div>

            <section v-if="isEdit" class="attr">
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
                    <div v-if="currentInfoAttr.item.txt||currentInfoAttr.item.txt===''">
                        <p>
                            <label>文字：</label>
                            <input type="text" v-model="currentInfoAttr.item.txt">
                        </p>
                    </div>
                    <div>
                        <p>
                            <label>背景色：</label>
                            <span style="display: block">
                                <swatches style="display: inline-block; vertical-align: middle; margin-right: 10px"
                                          v-model="currentInfoAttr.item.bg"
                                          colors="text-advanced" popover-to="left"/>
                                <span @click="currentInfoAttr.item.bg=false">透明</span>
                            </span>

                        </p>
                    </div>
                    <div v-if="currentInfoAttr.item.color||currentInfoAttr.item.color===''">
                        <p>
                            <label>颜色：</label>
                            <span style="display: block">
                                <swatches style="display: inline-block; vertical-align: middle; margin-right: 10px"
                                          v-model="currentInfoAttr.item.color"
                                          colors="text-advanced" popover-to="left"/>
                            </span>

                        </p>
                    </div>
                    <div v-if="currentInfoAttr.item.font||currentInfoAttr.item.font===''">
                        <p>
                            <label>字体大小：</label>
                            <input type="number" v-model.number="currentInfoAttr.item.font">
                        </p>
                    </div>
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
                <li @click="onClickAddInfo('text')">添加信息点</li>
                <li @click="onClickAddInfo('label')">添加标签</li>
                <li @click="onClickAddInfo('chart')">添加图表</li>
            </ul>
        </div>

        <canvas style="display: none" ref="mainCanvas"></canvas>
        <p class="lable-test">
            <!--标签宽度测试-->
        </p>
        <modal name="info-model" class="model">
            <h1>{{modelTitle}}</h1>
            <div class="body">
                <div v-if="modelData.type!='label'">
                    <p>
                        <label>筛选</label>
                        <!--                    <select v-model="rForm.deviceId">-->
                        <!--                        <option value="">设备类型</option>-->
                        <!--                        <option v-for="(item,key) in rData.deviceList" :key="key" :value="item.type_code">-->
                        <!--                            {{item.type_name}}-->
                        <!--                        </option>-->
                        <!--                    </select>-->
                        <select v-model="rForm.infoTypeId">
                            <option value="">信息点类型</option>
                            <option v-for="(item,key) in rData.infoTypeData" :key="key" :value="item.type_key">
                                {{item.type_name}}
                            </option>
                        </select>

                    </p>
                    <p>
                        <label>信息点</label>
                        <select v-model="modelInfoData">
                            <option value="">选择信息点</option>
                            <option v-for="(item,key) in rData.infoList" :key="key" :value="item">
                                {{item.cn_name}}
                            </option>
                        </select>
                    </p>
                </div>
                <div v-else>
                    <p>
                        <label>设备</label>
                        <select v-model="modelInfoData">
                            <option value="">选择设备</option>
                            <option v-for="(item,key) in rData.deviceList" :key="key" :value="item">
                                {{item.type_name}}
                            </option>
                        </select>
                    </p>
                </div>
                <p>
                    <label></label>
                    <button @click="addInfo">确定</button>
                    <button @click="$modal.hide('info-model')">取消</button>
                </p>
            </div>
        </modal>
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
    import chat from './components/chat'
    import VueDraggableResizable from 'vue-draggable-resizable-gorkys'
    import VModal from 'vue-js-modal'
    import Axios from 'axios'
    import Qs from 'qs'
    import Swatches from 'vue-swatches'
    import "vue-swatches/dist/vue-swatches.min.css"


    Vue.use(VModal)

    // import 'vue-draggable-resizable-gorkys/dist/VueDraggableResizable.css'
    Vue.prototype.$getUrlQueryString = function (name, url) {
        let search = window.location.search
        if (url) {
            var a = document.createElement('a')
            a.href = url
            search = a.search
        }
        var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i')
        search = decodeURI(search)
        var r = search.substr(1).match(reg)
        if (r != null) return unescape(r[2])
        return null
    }
    const $domIsEdit = document.getElementById('isEdit')
    const isEdit = $domIsEdit ? $domIsEdit.value == 'true' : false // ? true : false


    export default {
        name: 'app',
        components: {
            tmpDarw,
            Swatches,
            chat,
            toolList,
            imgCanvas,
            lineDarw,
            canvasDarw,
            iconList,
            VueDraggableResizable
        },
        created: function () {
            const baseUrl = document.getElementById('baseUrl').value
            Vue.prototype.$api = Axios.create({
                baseURL: baseUrl,
            })
            this.$api.interceptors.request.use(config => {
                config.transformRequest = [function (data) {
                    data = Qs.stringify(data);
                    return data;
                }]
                config.headers['Content-Type'] = 'application/x-www-form-urlencoded'

                return config
            }, function (error) {
                return Promise.reject(error)
            })
            this.$api.interceptors.response.use(response => {
                const {success} = response.data
                if (response.config.isJson || success) {
                    return response
                } else {
                    return Promise.reject(response);
                }
            }, error => {
                return Promise.reject(new Error(error));
            })

            this.$api.get('template/getstationinfotype', {params: {stationUuid: this.id}}).then(resp => {
                this.rData.infoTypeData = resp.data.data
            })
            this.$api.get('template/getstationdevicelist', {params: {stationUuid: this.id}}).then(resp => {
                this.rData.deviceList = resp.data.data
            })
            this.getInfoList()
            this.$nextTick(function () {
                this.$canvas.width = 1024
                this.$canvas.height = 682
                this.ctx = this.$canvas.getContext('2d')

                this.$api.get(`template/gettemplate`, {params: {stationUuid: this.id}}).then(resp => {
                    const data = resp.data.data
                    this.$api.get(data.infoJson, {isJson: true}).then(resp => {
                        const _data = resp.data
                        this.infoJson = _data.infoJson
                        this.pcsJson = _data.pcsJson
                        this.bmsJson = _data.bmsJson
                        this.setJson = _data.setJson
                    })
                    if (this.isEdit) {
                        this.$api.get(data.stationJson, {isJson: true}).then(resp => {
                            this.canvasJson = resp.data.canvasJson
                            this.imgJson = resp.data.imgJson
                            this.lineJson = resp.data.lineJson
                        })
                    } else {
                        this.demoBg = data.stationPng
                    }

                })
            })
            setInterval(() => {
                this.getValList()
            }, this.setJson.intervaTime * 1000)
        },
        computed: {
            isEdit: function () {
                return isEdit
            },
            id: function () {
                return this.$getUrlQueryString('stationUuid') // 'c021698dd56a4289a6bc1312cedc0a68'
            },
            infoUuidList: function () {
                const _ = []
                this.infoJson.forEach(item => {
                    _.push(item.info.info_uuid)
                })
                return _
            },
            chatUuidList: function () {
                const _ = []
                this.infoJson.forEach(item => {
                    console.log(item)
                    if (item.type == 'chart')
                        _.push(item.info.info_uuid)
                })
                return _
            },
            $canvas: function () {
                return this.$refs.mainCanvas
            },
            $tmpCanvas: function () {
                return this.$refs.tmpCanvas
            },
            saveList: function () {
                const _ = {
                    lineJson: this.lineJson,
                    imgJson: this.imgJson,
                    canvasJson: this.canvasJson,
                    infoJson: this.infoJson,
                    pcsJson: this.pcsJson,
                    bmsJson: this.bmsJson,
                    setJson: this.setJson
                }
                return _
            }
        },
        data: function () {
            return {
                historyList: [],
                resetList: [],
                historyTime: false,
                isReset: false,
                demoBg: false,
                setJson: {
                    intervaTime: 10
                },
                $api: {},
                rData: {
                    infoTypeData: [],
                    deviceList: [],
                    infoList: []
                },
                rForm: {
                    deviceId: '',
                    infoTypeId: ''
                },
                isShowRightMenu: false,
                isFullscreen: false,
                modelInfoData: '',
                modelTitle: '',
                modelData: {},
                valData: {},
                chartData: {},
                infoShowList: [],
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
                bmsJson: [],
            }
        },
        methods: {
            getValList: function () {
                const params = {
                    stationUuid: this.id,
                    infoList: this.infoUuidList.join(',')
                }
                this.$api.get('template/getstationinfovaluelist', {params}).then(resp => {
                    resp.data.data.forEach(item => {
                        Vue.set(this.valData, item.info_uuid, item)
                        if (this.chatUuidList.indexOf(item.info_uuid) != -1) {
                            if (!this.chartData[item.info_uuid]) {
                                Vue.set(this.chartData, item.info_uuid, {val: [], xAxis: []})
                            }
                            const xAxis = this.chartData[item.info_uuid].xAxis
                            const val = this.chartData[item.info_uuid].val
                            const time = item.create_time.split(' ')[1]
                            if (time != xAxis[xAxis.length - 1]) {
                                val.push(item.value)
                                xAxis.push(time)
                                if (val.length > 10) {
                                    val.shift()
                                    xAxis.shift()
                                }
                            } else {
                                console.log('no updata')
                            }
                        }
                    })
                })
            },
            getInfoList: function () {
                const params = {
                    stationUuid: this.id,
                    deviceType: this.rForm.deviceId,
                    infoTypeId: this.rForm.infoTypeId
                }
                this.$api.get('template/getstationinfolist', {params}).then(resp => {
                    this.rData.infoList = resp.data.data
                })
            },
            exitFullscreen: function () {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                } else if (document.msExitFullscreen) {
                    document.msExitFullscreen();
                } else if (document.mozCancelFullScreen) {
                    document.mozCancelFullScreen();
                } else if (document.webkitExitFullscreen) {
                    document.webkitExitFullscreen();
                }
            },
            fullscreen: function () {
                const element = document.documentElement
                if (element.requestFullscreen) {
                    element.requestFullscreen();
                } else if (element.mozRequestFullScreen) {
                    element.mozRequestFullScreen();
                } else if (element.msRequestFullscreen) {
                    element.msRequestFullscreen();
                } else if (element.webkitRequestFullscreen) {
                    element.webkitRequestFullScreen();
                }
            },
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
            onClickAddInfo: function (type) {
                const {sx, sy} = this.isShowRightMenu
                let _ = {x: sx, y: sy, type: type, isSize: false}
                switch (type) {
                    case 'text':
                        this.modelTitle = '添加信息点'
                        _.w = 300
                        _.h = 20
                        break
                    case 'label':
                        this.modelTitle = '添加设备'
                        _.h = 20
                        break
                    case 'chart':
                        this.modelTitle = '添加图表'
                        _.w = 300
                        _.h = 153
                        break
                }
                this.$modal.show('info-model')
                this.modelData = _
                this.isShowRightMenu = false
            },
            addInfo: function () {
                if (this.modelInfoData == '') {
                    alert(`请选择${this.modelData.type != 'label' ? '信息点' : '设备'}`)
                    return
                }
                if (this.modelData.type == 'label') {
                    const a = document.getElementsByClassName('lable-test')[0]
                    a.innerText = this.modelInfoData.type_name
                    this.modelData.w = a.clientWidth + 1
                    a.innerText = ''
                }
                const data = Object.assign(this.modelData)
                data.info = this.modelInfoData
                this.infoJson.push(data)
                this.rForm = {
                    deviceId: '',
                    infoTypeId: ''
                }
                this.$modal.hide('info-model')
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
            onVerticalImg: function (item, key) {
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
                    _.w = _.imgH
                    _.h = _.imgW
                } else {
                    _.w = _.imgW
                    _.h = _.imgH
                }
                Vue.set(this.imgJson, key, _)

            },
            sDarw: function (ev) {
                if (this.newTool) {
                    if (this.newTool == 'txt') {
                        const txt = prompt('请输入文本')
                        this.canvasJson.push({
                            x: ev.offsetX,
                            y: ev.offsetY,
                            w: 300, h: 30,
                            type: 'txt',
                            bg: false,
                            txt: txt,
                            color: '#000',
                            font: 30,
                            isSize: false,
                        })
                        this.newTool = false
                        return
                    }
                    this.isDarw = true
                    this.tmpJson.push({
                        ix: ev.x,
                        iy: ev.y,
                        sx: ev.offsetX,
                        sy: ev.offsetY,
                        ex: 0,
                        ey: 0,
                        bg: false,
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
                        const {x, y, w, h, type, bg} = this.$refs.tmpCanvas[this.currentLineIndex]
                        this.canvasJson.push({x, y, w, h, type, bg})
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
                    const {ix, iy, sx, sy, type, bg} = data
                    const ex = ev.offsetX
                    const ey = ev.offsetY
                    Vue.set(this.tmpJson, this.currentLineIndex, {ix, iy, sx, sy, ex, ey, type, bg})
                }
            },
            onRightClick: function (ev) {
                if (!this.isEdit) return
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
            fixCanvasWidth: function (args, item) {
                item.w = args[0]
                item.h = args[1]
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
                if (this.$refs.canImg) {
                    this.$refs.canImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        const {x, y, w, h} = item
                        this.ctx.drawImage(subCanvas, x, y, w, h)
                    })
                }
                if (this.$refs.lineImg) {
                    this.$refs.lineImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        this.ctx.drawImage(subCanvas, 0, 0, 1024, 682)
                    })
                }
                if (this.$refs.imgImg) {
                    console.log(this.$refs.imgImg)
                    this.$refs.imgImg.forEach(item => {
                        const subCanvas = item.$refs.lineCanvas
                        const {x, y, w, h} = item
                        this.ctx.drawImage(subCanvas, x, y, w, h)
                    })
                }


            },
            saveData: function () {
                this.genImg()
                console.log(this.$canvas);
                const imgBase64 = this.$canvas.toDataURL('image/png')
                const stationJson = {lineJson: this.lineJson, imgJson: this.imgJson, canvasJson: this.canvasJson}
                const infoJson = {
                    infoJson: this.infoJson,
                    pcsJson: this.pcsJson,
                    bmsJson: this.bmsJson,
                    setJson: this.setJson
                }
                this.$api.post('template/savetemplate', {
                    stationUuid: this.id,
                    imgBase64:imgBase64,
                    stationJson: JSON.stringify(stationJson),
                    imgInfoJson: JSON.stringify(infoJson)
                }).then(resp => {
                    alert(`保存成功`)
                    console.log(resp)
                }).catch(err => {
                    alert(`保存失败`)
                    console.log(err)
                })
            },
            getHistoryStr: function () {
                return JSON.stringify(this.saveList)

            },
            resetHistory: function (str) {

                const {lineJson, imgJson, canvasJson, infoJson, pcsJson, bmsJson, setJson} = JSON.parse(str)
                this.isReset = true
                this.lineJson = lineJson
                this.imgJson = imgJson
                this.canvasJson = canvasJson
                this.infoJson = infoJson
                this.pcsJson = pcsJson
                this.bmsJson = bmsJson
                this.setJson = setJson
                this.$nextTick(function () {
                    this.isReset = false
                })
            },
            addHistory: function () {
                this.historyList.push(this.getHistoryStr())
                if (this.historyList.length > 20) this.historyList.shift()
            },
            historyBack: function () {
                if (this.historyList.length != 0) {
                    const oldStr = this.historyList.pop()
                    if (this.resetList.length == 0) {
                        this.resetList.push(this.getHistoryStr())
                    } else {
                        this.resetList.push(oldStr)
                    }
                    this.resetHistory(oldStr)
                }
            },
            historyReset: function () {
                if (this.resetList.length != 0) {
                    const oldStr = this.resetList.pop()
                    this.historyList.push(oldStr)
                    this.resetHistory(oldStr)
                }
            }


        },
        watch: {
            rForm: {
                deep: true,
                handler: function () {
                    this.modelInfoData = ''
                    this.getInfoList()
                }
            },
            saveList: {
                deep: true,
                handler: function () {
                    if (!this.isReset) {
                        this.resetList = []
                        clearTimeout(this.historyTime)
                        this.historyTime = setTimeout(() => {
                            this.addHistory()
                        }, 200)
                    }
                }
            }

            // infoJson: function(){
            //     this.getValList()
            // }

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
        width: 1336px;
        margin: 0 auto;

        &.demo {
            width: 1024px;
        }

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
                padding: 0 40px;

                .r {
                    float: right;
                }

                .opp-btn {
                    margin-top: 20px;
                    background: transparent;
                    border: 0;

                    i {
                        display: block;
                    }

                    .save {
                        width: 30px;
                        height: 30px;
                        background-image: url('./assets/save.png');
                        background-size: contain;
                    }

                    .full {
                        width: 30px;
                        height: 30px;
                        background-image: url('./assets/full.png');
                        background-size: contain;
                    }

                    .full-a {
                        width: 30px;
                        height: 30px;
                        background-image: url('./assets/full-a.png');
                        background-size: contain;
                    }
                }
            }
        }

        .img-wrap {
            position: relative;
            width: 1024px;
            height: 682px;
            background-color: #218a8a;

            .demo-bg {
                width: 100%;
                height: 100%;
                position: absolute;
                top: 0;
                left: 0;
                z-index: 1;
            }

            .line:hover {
                z-index: 99999 !important;
            }

            .move {
                cursor: crosshair;
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
                /*border: 1px solid red;*/
                /*&.active{*/
                /*border: none;*/
                /*}*/
                box-shadow: 0 0 1px red, 0 0 5px #ccc;
                z-index: 9999 !important;

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
                font-size: 0;
                line-height: 0;
                /*canvas{*/
                /*    position: absolute;*/
                /*    top:0;*/
                /*    left: 0;*/
                /*}*/
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
                    font-size: 16px;
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

                &.chart {
                    h1 {
                        margin: 0;
                        font-size: 14px;
                        font-weight: normal;
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
                z-index: 2;

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

                    .del-btn {
                        position: relative;

                        i {
                            position: absolute;
                            display: inline-block;
                            width: 20px;
                            height: 20px;
                            border-radius: 50%;
                            right: auto;
                            top: 0;
                            left: -30px;
                            font-style: normal;
                            background-image: url('./assets/icon/opp-del.png');
                            background-size: cover;
                            background-color: #ccc;
                        }
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

        .tooltip {
            position: absolute;
            top: -45px;
            left: 0px;
            display: none;
            height: 35px;
            background-color: #ccc;
            border-radius: 5px;
            white-space: nowrap;
            z-index: 9999;

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

        &.demo {
            .tooltip, .del-btn, .table-add-tr {
                display: none;
            }

            .table-wrap {
                & > div {
                    margin-bottom: 20px+ 20px;
                }
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

        .model {
            z-index: 20000;

            h1 {
                margin: 0;
                padding: 10px 10px;
                background-color: #eeeeee;
                color: #555;
                font-size: 16px;
            }

            .body {
                padding: 10px;

                label {
                    display: inline-block;
                    min-width: 100px;
                    text-align: right;
                }

                p {
                    font-size: 14px;
                    color: #252525;
                }

                select, button {
                    display: inline-block;
                    margin-left: 10px;
                }
            }
        }

        .lable-test {
            position: fixed;
            top: 0;
            left: 0;
            display: inline-block;
            padding: 0 4px;
            font-size: 16px;
            opacity: 0;
        }
    }
</style>
