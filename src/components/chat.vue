<template>
    <div class="c-chat-wrap">
        <v-chart class="chat" :options="option"></v-chart>
    </div>
</template>
<script>
    import vChart from 'vue-echarts'
    import 'echarts/lib/chart/line'


    export default {
        components: {vChart},
        props: {
            lineData: {
                type: Array,
                default: function () {
                    return []
                }
            },
            xAxisData: {
                type: Array,
                default: function () {
                    return []
                }
            }
        },
        computed: {
            option: function () {
                return {
                    // animation:false,
                    title: {
                        textAlign: 'center',
                        text: '111',
                        textStyle: {
                            fontSize: 12,
                            fontWeight: 'normal'
                        }
                    },
                    calculable: false,
                    grid: {
                        x: 40,
                        y: 10,
                        x2: 10,
                        y2: 20
                    },
                    yAxis: {
                        type: 'value'
                    },
                    xAxis: {
                        type: 'category',
                        boundaryGap: false,
                        data: this.xAxisData
                    },
                    series: [{
                        data: this.lineData,
                        type: 'line',
                        // 平滑
                        smooth: true,
                        // showSymbol:false,
                        itemStyle : {
                            normal : {
                                lineStyle:{
                                    color:'#000',
                                    width:1
                                }
                            }
                        }
                    }],

                }
            }
        },
        data: function () {
            return {}
        }
    }
</script>
<style lang="scss">
    .c-chat-wrap {
        width: 300-50+45px;
        height: 100+30px;
        overflow: hidden;

        .chat {
            width: 100%;
            height: 100%;
        }
    }
</style>
