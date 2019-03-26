<template>
    <canvas ref="lineCanvas"
            :style="{width:'100%',height: '100%'}"
    ></canvas>
</template>
<script>
    export default {
        props: {
            img: {
                type: Object
            }
        },
        created: function () {
            this.$nextTick(function () {
                this.ctx = this.$el.getContext('2d')
                this.draw()
            })
        },
        data: function () {
            return {
                w: 0,
                h: 0,
                y: 0,
                x: 0,
            }
        },
        methods: {
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
            draw: function () {
                const {imgW: w, imgH: h, rotate: deg} = this.img
                this.w = this.img.w
                this.h = this.img.h
                this.x = this.img.x
                this.y = this.img.y
                this.$el.width = this.img.w
                this.$el.height = this.img.h
                this.ctx.clearRect(0, 0, w, h)
                switch (deg) {
                    case 90:
                        this.ctx.translate(h, 0);
                        break
                    case 180:
                        this.ctx.translate(w, h);
                        break
                    case 270:
                        this.ctx.translate(0, w);
                        break
                }
                this.ctx.rotate(deg * Math.PI / 180);
                this._loadImage(this.img.img).then(img => {
                    this.ctx.drawImage(img, 0, 0, w, h)
                    if (this.img.isVertical) this.vertical()
                    if (this.img.isHorizontal) this.horizontal()
                }).catch(e => {
                    console.log(e)
                })
            },
            // 水平翻转
            vertical: function () {
                var img_data = this.ctx.getImageData(0, 0, this.w, this.h),
                    i, i2, t,
                    h = img_data.height,
                    w = img_data.width,
                    w_2 = w / 2;
                // 将 img_data 的数据水平翻转
                for (var dy = 0; dy < h; dy++) {
                    for (var dx = 0; dx < w_2; dx++) {
                        i = (dy << 2) * w + (dx << 2)
                        i2 = ((dy + 1) << 2) * w - ((dx + 1) << 2)
                        for (var p = 0; p < 4; p++) {
                            t = img_data.data[i + p]
                            img_data.data[i + p] = img_data.data[i2 + p]
                            img_data.data[i2 + p] = t
                        }
                    }
                }
                this.ctx.clearRect(0, 0, this.w, this.h)
                this.ctx.putImageData(img_data, 0, 0)
            },
            horizontal: function () {
                var img_data = this.ctx.getImageData(0, 0, this.w, this.h),
                    i, i2, t,
                    h = img_data.height,
                    w = img_data.width,
                    h_2 = h / 2;
                for (var dx = 0; dx < w; dx++) {
                    for (var dy = 0; dy < h_2; dy++) {
                        i = dy * w * 4 + dx * 4
                        i2 = (h - dy) * w * 4 + dx * 4
                        for (var p = 0; p < 4; p++) {
                            t = img_data.data[i + p]
                            img_data.data[i + p] = img_data.data[i2 + p]
                            img_data.data[i2 + p] = t
                        }
                    }
                }
                this.ctx.clearRect(0, 0, this.w, this.h)
                this.ctx.putImageData(img_data, 0, 0)
            }
        },
        watch: {
            img: {
                deep: true,
                handler: function () {
                    this.draw()
                }
            }
        }

    }
</script>

