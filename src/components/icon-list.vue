<template>
    <section class="c-icon-list">
        <p class="head">元件</p>
        <ul class="list">
            <li v-for="(item,key) in list" :key="key">
                <div class="item" @click.stop="$emit('sel',arguments,item)"
                     :style="{backgroundImage:` url('${item.img}')`}">
                </div>
            </li>

        </ul>
    </section>
</template>
<script>
    // const requireAll = requireContext => requireContext.keys().map(requireContext)
    //
    // const req = require.context('../assets/icon/', false, /\.png$/)
    // // const iconMap = requireAll(req)
    // // console.log(iconMap)
    // requireAll(req)
    import pcs from '../assets/icon/PCS.png'
    import cell15 from '../assets/icon/cell15.png'
    import cellAdmin from '../assets/icon/cell-admin.png'
    import onOff3 from '../assets/icon/on-off-3.png'
    import wattmeter from '../assets/icon/wattmeter.png'
    import cell from '../assets/icon/cell.png'
    import cell1admin from '../assets/icon/cell-1-admin.png'
    import onOff1 from '../assets/icon/on-off-1.png'
    import fastener1 from '../assets/icon/fastener1.png'
    import fastener2 from '../assets/icon/fastener2.png'
    import cellAdminGroup from '../assets/icon/cellAdminGroup.png'
    import fastener from '../assets/icon/fastener.png'
    import voltmeter from '../assets/icon/voltmeter.png'
    import ammeter from '../assets/icon/ammeter.png'
    import contact from '../assets/icon/contact.png'
    import distribution from '../assets/icon/distribution.png'

    export default {
        created: function () {
            // const imgs =
            // imgs.forEach(item => {
            //     this.list.push({img: item, w: 100, h: 80, isSize: false})
            // })
        },
        data: function () {
            return {
                list: [
                    {img: pcs, name: 'pcs', w: 71, h: 72, isSize: false},
                    {img: cell15, name: '一组电池15个', w: 17, h: 242, isSize: false},
                    {img: cellAdmin, name: '一组电池管理单元组', w: 23, h: 254, isSize: false},
                    {img: onOff3, name: '三刀开关', w: 35, h: 14, isSize: false},
                    {img: wattmeter, name: '功率表', w: 20, h: 19, isSize: false},
                    {img: cell, name: '单个电池', w: 16, h: 18, isSize: false},
                    {img: cell1admin, name: '单个电池管理单元', w: 13, h: 17, isSize: false},
                    {img: onOff1, name: '单刀开关', w: 36, h: 11, isSize: false},
                    {img: fastener1, name: '单接线柱', w: 12, h: 17, isSize: false},
                    {img: fastener2, name: '双接线柱', w: 11, h: 33, isSize: false},
                    {img: cellAdminGroup, name: '带有管理单元的电池组', w: 41, h: 258, isSize: false},
                    {img: fastener, name: '接线柱', w: 6, h: 28, isSize: false},
                    {img: voltmeter, name: '电压表', w: 17, h: 16, isSize: false},
                    {img: ammeter, name: '电流表', w: 17, h: 17, isSize: false},
                    {img: contact, name: '触点', w: 6, h: 7, isSize: false},
                    {img: distribution, name: '配电箱', w: 57, h: 33, isSize: false},
                ]
            }
        }
    }
</script>
<style lang="scss">
    .c-icon-list {
        background-color: #f0f0f0;
        padding-top: 5px;
        padding-right: 5px;
        border: 1px solid #494949;

        .head {
            margin: 2px;
            padding: 0 10px;
            margin-bottom: 5px;
            font-size: 14px;
            box-shadow: 1px 1px 1px #7a7a7a;
        }

        .list {
            padding: 0;
            margin: 0;
            overflow-y: scroll;
            overflow-x: hidden;
            height: 460px;

            li {
                width: 50%;
                padding: 10px;
                margin: 0;
                list-style: none;
                float: left;

                .item {
                    width: 100%;
                    padding-top: 100%;
                    background-color: #ccc;
                    background-size: contain;
                    background-repeat: no-repeat;
                    background-position: center center;
                }
            }
        }
    }
</style>
