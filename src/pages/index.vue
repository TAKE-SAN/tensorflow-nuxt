<template>
  <v-layout justify-center wrap>
    <v-flex xs10 md5 sm5 pa-1>
      <v-layout justify-center colmn wrap>
        <v-flex xs12 class="flex-center">
          <div class="content">
            <canvas width="28" height="28" ref="canvas"
                    @mousemove="canvasMouseMove"
                    @touchmove.stop="canvasMouseMove"
                    @touchend.stop="offEdit"
                    @mouseup="offEdit"
                    @touchstart.prevent="onEdit"
                    @mousedown="onEdit"></canvas>
            <svg viewBox="0, 0, 280, 280">
              <rect x="40" y="40" width="200" height="200" stroke="orange" stroke-width="1" stroke-dasharray="4 4" fill="none" />
            </svg>
          </div>
        </v-flex>
        <v-flex xs12 class="flex-center">
          <v-btn depressed color="primary" @click="resetCanvas">Reset</v-btn>
        </v-flex>
      </v-layout>
    </v-flex>
    <v-flex xs10 md5 sm5 pa-1>
      <v-card>
        <v-list>
          <template v-for="(item, index) in result">
            <v-list-tile class="orange lighten-5" v-if="index === maxNum">
              <v-list-tile-action>
                <v-icon color="orange">star</v-icon>
              </v-list-tile-action>
              <v-list-tile-title v-text="index"></v-list-tile-title>
              <v-list-tile-title v-text="item"></v-list-tile-title>
            </v-list-tile>

            <v-list-tile v-else>
              <v-list-tile-action></v-list-tile-action>
              <v-list-tile-title v-text="index"></v-list-tile-title>
              <v-list-tile-title v-text="item"></v-list-tile-title>
            </v-list-tile>

          </template>
        </v-list>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import * as tf from '@tensorflow/tfjs';

export default {
  async mounted(){
    this.resetCanvas();
    this.model = await tf.loadLayersModel('https://raw.githubusercontent.com/TAKE-SAN/tensorflow-nuxt/master/models/model.json');
  },

  data(){
    return {
      result: [],
      edit : false,
      lastPosX : null,
      lastPosY : null,
      model : null
    }
  },

  methods:{
    canvasMouseMove(e) {
      if(!this.edit){ return }
      let clientRect = e.target.getBoundingClientRect();
      let x =  ((e.clientX || e.touches[0].clientX) - clientRect.left) * 28 / e.target.offsetWidth;
      let y =  ((e.clientY || e.touches[0].clientY) - clientRect.top ) * 28 / e.target.offsetHeight;
      this.lastPosX = this.lastPosX || x;
      this.lastPosY = this.lastPosY || y;
      const canvas = this.$refs['canvas'];
      const ctx = canvas.getContext("2d");
      ctx.shadowColor = "white";
      ctx.shadowBlur = 1;
      ctx.strokeStyle = "white";
      ctx.lineWidth = 1;
      ctx.lineCap = "round";
      ctx.beginPath();
      ctx.moveTo(this.lastPosX, this.lastPosY);
      ctx.lineTo(x, y);
      ctx.stroke();
      ctx.closePath();
      this.lastPosX = x;
      this.lastPosY = y
    },

    resetCanvas() {
      const canvas = this.$refs['canvas'];
      const ctx = canvas.getContext("2d");
      ctx.fillStyle = "black";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      this.result = [0,0,0,0,0,0,0,0,0,0].map(v => v.toFixed(10))
    },

    onEdit() {
      console.log('startEdit');
      this.edit = true
    },

    offEdit() {
      if(!this.edit){ return }
      console.log('endEdit');
      this.edit = false;
      this.lastPosX = null;
      this.lastPosY = null;
      this.predict();
    },

    predict() {
      const imgTensolObj = tf.browser.fromPixels(this.$refs['canvas'], 1).reshape([1, 28, 28, 1]).cast('float32').div(tf.scalar(255));
      const predict = this.model.predict(imgTensolObj).dataSync();
      this.result = Array.prototype.slice.call(predict).map((v => v.toFixed(10)));
    }
  },

  computed:{
    maxNum(){
      return this.result
        .map((v, i) => [v, i])
        .sort(([a], [b]) => a < b ? 1 : -1)
        .filter(([v]) => v > 0)
        .map(([v, i]) => i)
        .concat([NaN])[0]
    }
  }
}
</script>

<style lang="scss">
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }
  .flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
  &.column {
     flex-direction: column;
     flex-wrap: wrap;
   }
  }
  .content {
    position: relative;
    width:  100%;
    height: 100%;
  & > * {
      width:  100%;
      height: 100%;
    }
  & svg, & .circular {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }
  & svg {
      pointer-events : none;
    }
  & .circular {
      background-color: rgba(255,255,255,0.9);
    }
  }
</style>
