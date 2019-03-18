<template>
    <section class="c-icon-list">
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
    import i1 from '../assets/icon/tmp_03.png'
    import i2 from '../assets/icon/tmp_06.png'
    import i3 from '../assets/icon/tmp_09.png'
    import i4 from '../assets/icon/tmp_12.png'
    import i5 from '../assets/icon/tmp_19.png'

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
                    {name: '111', img: i1, isSize: false, w: 54, h: 113},
                    {name: '111', img: i2, isSize: false, w: 60, h: 112},
                    {name: '111', img: i3, isSize: false, w: 60, h: 110},
                    {name: '111', img: i4, isSize: false, w: 73, h: 56},
                    {name: '111', img: i5, isSize: false, w: 45, h: 85},
                ]
            }
        }
    }
</script>
<style lang="scss">
    .c-icon-list {
        .list {

            padding: 0;
            margin: 0;

            li {
                width: 50%;
                min-height: 100px;
                padding: 20px;
                margin: 0;
                list-style: none;
                float: left;

                .item {
                    width: 100px;
                    height: 100px;
                    background-color: #ccc;
                    background-size: contain;
                    background-repeat: no-repeat;
                    background-position: center center;
                }
            }
        }
    }
</style>
