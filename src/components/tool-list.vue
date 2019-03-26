<template>
    <section class="c-tool-list">
        <p class="head">基本图形</p>
        <ul class="list">
            <li v-for="(item,key) in list" :key="key">
                <div class="item" @click.stop="$emit('sel',item.tool)"
                     :style="{backgroundImage:` url('${item.img}')`}">
                    {{item.name}}
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
    import i1 from '../assets/icon/tmp_03.png'
    import i2 from '../assets/icon/tmp_06.png'
    import i3 from '../assets/icon/tmp_09.png'


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
                    {name: '线', tool:'line', img: i1 },
                    {name: '圆',tool:'circle', img: i2 },
                    {name: '矩形',tool:'rect', img: i3 },

                ]
            }
        }
    }
</script>
<style lang="scss">
    .c-tool-list {
        background-color: #f0f0f0;
        padding-top: 5px;
        padding-right: 5px;
        border: 1px solid #494949;
        .head{
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
            height: 240px;
            li {
                width: 33%;
                min-height: 100px;
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
