<template>
    <div class="container">
        <div class="layer-wrapper">
            <img id="bg_certificate" src="../assets/bg_certificate.png" alt="">
            <canvas id="layer1" data-src="../assets/bg_certificate.png"></canvas>
            <canvas id="layer2"
                @mousedown="handleMouseDown"
                @mouseup="handleMouseUp">
            </canvas>
        </div>
        <div class="toolbar">
            <div 
                :class="['icon-tool icon-mosaic', mosaicActive && 'active']"
                @click="handleClickToolbar('mosaic')">
            </div>
            <div class="icon-tool icon-undo" @click="handleClickToolbar('undo')"></div>
            <div class="icon-tool icon-rotate" @click="handleClickToolbar('rotate')"></div>
            <div class="icon-tool icon-zoom-in" @click="handleClickToolbar('zoomIn')"></div>
            <div class="icon-tool icon-zoom-out" @click="handleClickToolbar('zoomOut')"></div>
        </div>
    </div>
</template>

<script>
import throttle from 'lodash/throttle';
export default {
    name: 'ImageMosaic',
    data() {
        return {
            mouseDown: null,
            mouseMove: null,
            layer1: null,
            layer1Ctx: null,
            layer2: null,
            layer2Ctx: null,
            bgImg: null,
            rotateCount: 1,
            throttleMouseMove: null,
            mosaicActive: false,
            imageDate: null,
            undoDatas: []
        };
    },
    mounted() {
        this.$nextTick(() => {
            this.init();
        });
    },
    methods: {
        throttle,
        init() {
            this.layer1 = document.getElementById('layer1');
            this.layer1Ctx = this.layer1.getContext('2d');
            this.layer1.width = 800;
            this.layer1.height = 600;
            this.bgImg = document.getElementById('bg_certificate');
            this.bgImg.onload = () => {
                this.layer1Ctx.drawImage(this.bgImg, 0, 0, this.layer1.width, this.layer1.height);
            }
            
            this.layer2 = document.getElementById('layer2');
            this.layer2.width = 800;
            this.layer2.height = 600;
            this.layer2Ctx = this.layer2.getContext('2d');
            this.layer2Ctx.globalAlpha = 0;
            this.layer2Ctx.fillStyle = '#fff';
            this.layer2Ctx.fillRect(0, 0, 800, 600);
        },
        handleMouseMove(event) {
            if (this.mouseDown) {
                const preImageData = this.undoDatas[this.undoDatas.length - 1];
                if (preImageData) {
                    this.layer2Ctx.putImageData(preImageData, 0, 0, 0, 0, this.layer2.width, this.layer2.height);
                }
                let x = event.offsetX;
                let y = event.offsetY;
                this.mouseMove = {
                    x,
                    y
                };
                let {
                    x: sX,
                    y: sY
                } = this.mouseDown;
                let width = x - sX;
                let height = y - sY;
                this.layer2Ctx.strokeStyle = '#dc143c';
                this.layer2Ctx.globalAlpha = 1;
                this.layer2Ctx.strokeRect(sX, sY, width, height);
            }
        },
        handleMouseUp(event) {
            if (this.mouseDown) {
                this.handleDrawMosaic();
            }
            this.mouseDown = null;
            this.mouseMove = null;
            this.layer2.removeEventListener('mousemove', this.throttleMouseMove);
            this.throttleMouseMove = null;
        },
        handleMouseDown(event) {
            if (!this.mosaicActive) {
                return;
            }
            this.mouseDown = {
                x: event.offsetX,
                y: event.offsetY
            };
            this.throttleMouseMove = throttle(this.handleMouseMove, 100);
            this.layer2.addEventListener('mousemove', this.throttleMouseMove);
            const imageData = this.layer2Ctx.getImageData(0, 0, this.layer2.width, this.layer2.height);
            this.undoDatas.push(imageData);
        },
        handleDrawMosaic() {
            console.log('handleDrawMosaic start');
            const preImageData = this.undoDatas[this.undoDatas.length - 1];
            if (preImageData) {
                this.layer2Ctx.putImageData(preImageData, 0, 0, 0, 0, this.layer2.width, this.layer2.height);
            }
            const {
                x: sX,
                y: sY
            } = this.mouseDown;
            const {
                x: eX,
                y: eY
            } = this.mouseMove;
            this.layer2Ctx.fillStyle = '#000';
            this.layer2Ctx.globalAlpha = 0.99;
            this.layer2Ctx.fillRect(sX, sY, eX - sX, eY - sY);
            return;
            let { x = 0, y = 0 } = {};
            if (eX >= sX) {
                if (eY >= sY) {
                    [x, y] = [eX, eY];
                } else {
                    [x, y] = [sX, eY];
                }
            } else {
                if (eY >= sY) {
                    [x, y] = [eX, sY];
                } else {
                    [x, y] = [eX, eY];
                }
            }
            const width = Math.abs(eX - sX);
            const height = Math.abs(eY - sY);
            const rows = Math.round(height / 4) + 1;
            const columns = Math.round(width / 4) + 1;
            const imageData = this.layer1Ctx.getImageData(0, 0, this.layer1.width, this.layer1.height);
            console.log('handleDrawMosaic ex/ey', eX, eY);
            console.log('handleDrawMosaic sx/sy', sX, sY);
            console.log('handleDrawMosaic x/y', x, y);
            console.log('handleDrawMosaic', rows, columns);
            for (let i = 0; i < rows; i++) {
                for (let j = 0; j < columns; j++) {
                    const cX = x + (j + 1) * 2;
                    const cY = y + (i + 1) * 2;
                    console.log('handleDrawMosaic cX/cY', cX, cY);
                    const r = imageData.data[(cY * this.layer1.width + x) * 4];
                    const g = imageData.data[(cY * this.layer1.width + x) * 4 + 1];
                    const b = imageData.data[(cY * this.layer1.width + x) * 4 + 2];
                    const a = imageData.data[(cY * this.layer1.width + x) * 4 + 3];
                    console.log('handleDrawMosaic rgba', `rgba(${r},${g},${b},${a})`);
                    // this.layer2Ctx.fillStyle = `rgb(${r},${g},${b})`;
                    this.layer2Ctx.fillStyle = 'blue';
                    this.layer2Ctx.globalAlpha = 0.99;
                    this.layer2Ctx.fillRect(cX -2, cY - 2, 4, 4);
                }
            }
            console.log('handleDrawMosaic end');
        },
        handleClickToolbar(action) {
            if (action === 'rotate') {
                this.layer1Ctx.clearRect(0, 0, this.layer1.width, this.layer1.height);
                this.layer1Ctx.translate(this.layer1.width / 2,  this.layer1.height / 2);
                if (this.rotateCount > 4) {
                    this.rotateCount = 1;
                }
                this.layer1Ctx.rotate(this.rotateCount * 90 * Math.PI / 180);
                this.layer1Ctx.drawImage(this.bgImg, -this.layer1.width / 2,  -this.layer1.height / 2, this.layer1.width, this.layer1.height);
                this.layer1Ctx.translate(-this.layer1.width / 2,  -this.layer1.height / 2);
            } else if (action === 'mosaic') {
                this.mosaicActive = !this.mosaicActive;
            } else if (action === 'undo') {
                const imageData = this.undoDatas.pop();
                if (imageData) {
                    this.layer2Ctx.putImageData(imageData, 0, 0, 0, 0, this.layer2.width, this.layer2.height);
                }
            }
        }
    }
}
</script>

<style lang="scss">
.container {
    width: 100%;
    height: 100%;
    .layer-wrapper {
        position: relative;
        margin: 0px auto;
        width: 800px;
        height: 600px;
        canvas {
            width: 100%;
            height: 100%;
        }
        img {
            position: absolute;
            left: 0;
            top: 0;
            right: 0;
            bottom: 0;
            width: 100%;
            height: 100%;
            max-width: 100%;
            max-height: 100%;
            z-index: 1;
        }
        #layer1 {
            position: absolute;
            left: 0;
            top: 0;
            right: 0;
            bottom: 0;
            z-index: 100;
        }
        #layer2 {
            position: absolute;
            left: 0;
            top: 0;
            right: 0;
            bottom: 0;
            z-index: 200;
        }
    }
    .mouse-detail {
        width: 100%;
        height: 35px;
        line-height: 35px;
    }
    .toolbar {
        display: flex;
        justify-content: space-around;
        align-items: center;
        margin: 0 auto;
        width: 350px;
        height: 35px;
        border: 1px solid #ddd;
        .icon-tool {
            width: 20px;
            height: 20px;
            background-color: transparent;
            background-repeat: no-repeat;
            background-position: center center;
            background-size: 100% 100%;
            cursor: pointer;
        }
        .icon-mosaic {
            background-image: url('../assets/icon_mosaic.png');
        }
        .icon-mosaic.active {
            background-image: url('../assets/icon_mosaic_active.png');
        }
        .icon-rotate {
            background-image: url('../assets/icon_rotate_right_90.png');
        }
        .icon-undo {
            background-image: url('../assets/icon_undo.png');
        }
        .icon-zoom-in {
            background-image: url('../assets/icon_zoom_in.png');
        }
        .icon-zoom-out {
            background-image: url('../assets/icon_zoom_out.png');
        }
    }
}
</style>


