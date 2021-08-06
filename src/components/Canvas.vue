<template>
  <div>
    <div>
      <canvas
        ref="canvas"
        class="canvas"
        :width="size.width"
        :height="size.height"
        @mousedown="handleMouseDown"
        @mouseup="handleMouseUp"
        @mousemove="handleMouseMove"
      ></canvas>
    </div>
    <a class="button" @click="clear">clear</a>
    <div v-for="score, iScore in scores" :key="score.id">
      {{iScore}}:{{ (Math.round(score * 1000000) / 1000000).toFixed(5)}}
    </div>

  </div>
</template>

<style scoped>
.canvas {
  background-color: #000; /*黒背景*/
}
</style>

<script>
import * as tf from '@tensorflow/tfjs';

export default {
  data() {
    return {
      size: {
        width: 400,
        height: 400,
      },
      mouse: {
        x: 0,
        y: 0,
        down: false,
      },
      scores: []
    };
  },
  computed: {
    currentMouse() {
      const c = this.$refs.canvas;
      const rect = c.getBoundingClientRect();

      return {
        x: this.mouse.x - rect.left,
        y: this.mouse.y - rect.top,
      };
    },
  },
  mounted() {
    this.clear();
  },
  methods: {
    check:async function() {
        const model = await tf.loadLayersModel('./model/model.json'); // モデルのパスを指定して読み込む

        tf.tidy(()=>{
            const input = tf.browser
                .fromPixels(this.$refs.canvas, 1)
                .toFloat()
                .resizeNearestNeighbor([28, 28])
                .div(tf.scalar(255))
                .expandDims();

            const scores = model.predict(input).dataSync(); // 推論
            this.scores = scores;
        }); // メモリリーク回避

    },
    draw: function() {
      if (this.mouse.down) {
        const ctx = this.$refs.canvas.getContext('2d');
        ctx.lineTo(this.currentMouse.x, this.currentMouse.y);
        ctx.strokeStyle = '#fff'; // 白文字
        ctx.lineWidth = 20;
        ctx.stroke();
      }
    },
    handleMouseDown: function(event) {
    console.log("mousedown");
      this.mouse = {
        x: event.pageX,
        y: event.pageY,
        down: true,
      };

      const ctx = this.$refs.canvas.getContext('2d');
      ctx.moveTo(this.currentMouse.x, this.currentMouse.y);
    },
    handleMouseUp: function() {
      this.mouse.down = false;
      console.log("mouseup");
      this.check();
    },
    handleMouseMove: function(event) {
      Object.assign(this.mouse, {
        x: event.pageX,
        y: event.pageY,
      });

      this.draw();
    },
    clear: function() {
      const ctx = this.$refs.canvas.getContext('2d');
      ctx.clearRect(0, 0, this.size.width, this.size.height);
      ctx.beginPath();
    },
  },
};
</script>