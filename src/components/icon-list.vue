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
    import Vue from 'vue'
    import img1 from '../assets/icon/PCS.png'
    export default {
        created: function () {
            this.$api.get('template/getimgtoolssrc', {params: {stationUuid: this.id, imageType: 1}}).then(resp => {
                const data = resp.data.data
                data.forEach(item => {
                    // {img: pcs, name: 'pcs', w: 71, h: 72, isSize: false},
                    console.log(item)
                    Vue.set(this.list, item.id, {})
                    this.loadImage(item)
                })
            })
        },
        computed: {
            id: function () {
                return  this.$getUrlQueryString('stationUuid') // 'c021698dd56a4289a6bc1312cedc0a68'
            },
        },
        data: function () {
            return {
                list: {}
            }
        },
        methods: {
            loadImage: function (item) {
                const image = new Image();
                // const baseImgUrl = document.getElementById('baseImgUrl').value
                // image.src = 'http://api.qrmbl.com/PCS.png'
                image.src = item.url;
                const _ = img => {
                    this.list[item.id] = {img: image.src, name: item.cn_name, w: img.width, h: img.height, isSize: false}
                }
                if (image.complete) {
                    _(image)
                }
                image.onload = function () {
                    _(image)
                };
                image.onerror = function (e) {
                    console.log(e)
                };
            },
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
