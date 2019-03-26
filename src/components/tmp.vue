<template>
    <div>
        <h1>临时ajax测试</h1>
        <p><input type="text" v-model="url">
            <button @click="send('get')">GET请求</button>
        </p>
        <p>{{result}}</p>
    </div>
</template>
<script>
    import axios from 'axios'

    export default {
        data: function () {
            return {
                url: '',
                result: {}
            }
        },
        methods: {
            send: function () {
                axios.get(this.url).then(resp => {
                    this.result = resp.data
                })
            }
        }
    }
</script>
