<template>
  <div class="container">
    <div>
      <div class="btns">
        <template v-if="!trainingOver">
          <el-color-picker v-model="color" style="margin-right:10px"></el-color-picker>
          <el-button size="mini" @click="train" :loading="training"
            >{{
              training
                ? `正在训练${(completeImgCount * 100).toFixed(2)}%`
                : "开始训练"
            }}
          </el-button>
        </template>
      </div>

      <canvas
        ref="canvas"
        class="canvas"
        @click="canvasClick"
        width="300"
        height="300"
      ></canvas>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      neuralNetwork: "",
      color: "#409EFF",
      training: false,
      trainingOver: false,
      colorMap: {},
      completeImgCount: 0,
      lossValue: 0,
      neuralNetworkOptions: {
        epochs: 200,
      },
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      let options = {
        inputs: ["x", "y"],
        outputs: ["label"],
        task: "classification",
        // debug: true,
        learningRate: 0.2,
      };
      this.neuralNetwork = this.$ml5.neuralNetwork(options);
      this.drawText()
    },
    canvasClick(e) {
      if (this.training) {
        return;
      }
      let { offsetX: x, offsetY: y } = e;
      let inputs = {
        x,
        y,
      };
      let target = { label: this.color };
      this.addModel(inputs, target);
      this.colorMap[this.color] = true;
    },
    addModel(inputs, target) {
      this.neuralNetwork.addData(inputs, target);
      this.drawColor(inputs.x, inputs.y);
    },
    drawColor(x, y) {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      ctx.beginPath();
      ctx.arc(x, y, 5, 0, Math.PI * 2, true);
      ctx.closePath();
      ctx.fillStyle = this.color;
      ctx.fill();
    },
    drawText() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      // ctx.beginPath();
      // ctx.strokeStyle = "black";
      // ctx.arc(x, y, 10, 0, Math.PI * 2, true);
      // ctx.closePath();
      // ctx.stroke();
      // ctx.moveTo(300, 300);
      ctx.fillStyle = "black";
      ctx.font = '16px "微软雅黑"';
      ctx.textBaseline = "middle";
      ctx.textAlign = "center";
      ctx.fillText('请选择颜色，然后点击画布', canvas.width/2, canvas.height/2);
    },
    classifyResult(x, y, ctx) {
      return new Promise((resolve) => {
        this.neuralNetwork.classify({ x, y }, (err, result) => {
          if (err) {
            console.log(err);
          } else {
            ctx.beginPath();
            ctx.arc(x, y, 1, 0, Math.PI * 2, true);
            ctx.closePath();
            ctx.fillStyle = result[0].label;
            ctx.fill();
            resolve();
            // setTimeout(() => {
            //   resolve();
            // }, 1);
          }
        });
      });
    },
    train() {
      if (Object.keys(this.colorMap).length < 2) {
        this.$message.warning("分类颜色不能小于两种");
        return;
      }
      this.training = true;
      setTimeout(() => {
        this.neuralNetwork.normalizeData();
        this.neuralNetwork.train(
          this.neuralNetworkOptions,
          (epoch, loss) => {
            this.completeImgCount = epoch / this.neuralNetworkOptions.epochs;
            console.log("training", epoch, loss);
          },
          () => {
            this.training = false;
            this.trainingOver = true;
            console.log("training finished");
            this.drawResult();
          }
        );
      }, 1000);
    },
    async drawResult() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      this.drawResultGroup1(ctx, canvas);
      this.drawResultGroup2(ctx, canvas);
      this.drawResultGroup3(ctx, canvas);
      this.drawResultGroup4(ctx, canvas);
    },
    async drawResultGroup1(ctx, canvas) {
      for (let x = 0; x <= canvas.width / 2; x++) {
        for (let y = 0; y <= canvas.height / 2; y++) {
          await this.classifyResult(x, y, ctx);
        }
      }
    },
    async drawResultGroup2(ctx, canvas) {
      for (let y = 0; y <= canvas.height / 2; y++) {
        for (let x = canvas.width / 2; x <= canvas.width; x++) {
          await this.classifyResult(x, y, ctx);
        }
      }
    },
    async drawResultGroup3(ctx, canvas) {
      for (let x = canvas.width; x >= canvas.width / 2; x--) {
        for (let y = canvas.height / 2; y < canvas.height; y++) {
          await this.classifyResult(x, y, ctx);
        }
      }
    },
    async drawResultGroup4(ctx, canvas) {
      for (let y = canvas.height; y >= canvas.height / 2; y--) {
        for (let x = 0; x < canvas.width / 2; x++) {
          await this.classifyResult(x, y, ctx);
        }
      }
    },
  },
};
</script>

<style scoped>
.container {
  width: 1200px;
  height: 100%;
  padding: 20px;
  box-sizing: border-box;
  margin: auto;
  overflow: auto;
  display: flex;
  justify-content: center;
  align-items: center;
}
.btns{
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}
.canvas {
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
}
</style>
