<template>
  <div>
    <div>神经网络</div>
    <div>
      <canvas
        ref="canvas"
        width="640"
        height="480"
        @click="canvasClick"
      ></canvas>
    </div>
    <button @click="text = 'C'">C</button>
    <button @click="text = 'D'">D</button>
    <button @click="text = 'E'">E</button>
    <button @click="train">train</button>
    <button @click="saveData">saveData</button>
    <button @click="save">save</button>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      model: "",
      text: "",
      state: "begin",
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      // let options = {
      //   inputs: ["x", "y"],
      //   outputs: ["label"],
      //   task: "classification",
      //   debug: true,
      //   learningRate: 0.01,
      // };
      let options = {
        inputs: ["x", "y"],
        outputs: ["frequency"],
        task: "regression",
        debug: true,
        learningRate: 0.01,
      };
      this.model = this.$ml5.neuralNetwork(options);
      // this.model.loadData('model.json')
      // let modelInfo = {
      //   model: "model/model.json",
      //   metadata: "model/model_meta.json",
      //   weights: "model/model.weights.bin",
      // };
      // this.model.load(modelInfo,()=>{
      //   console.log('modelLoaded');
      // })
    },
    canvasClick(e) {
      let { offsetX: x, offsetY: y } = e;
      let inputs = {
        x,
        y,
      };
      // let target = { label: this.text };
      let targetFrequency={
        'C':1,
        'D':0.5,
        'E':0
      }
      let target = { frequency: targetFrequency[this.text] };
      if (this.state === "begin") {
        this.addModel(inputs, target);
      } else if (this.state === "over") {
        // this.classify(inputs);
        this.predict(inputs);
      }
    },
    addModel(inputs, target) {
      this.model.addData(inputs, target);
      this.drawText(inputs.x, inputs.y);
    },
    train() {
      this.model.normalizeData();
      let options = {
        epochs: 200,
      };
      this.model.train(
        options,
        (epoch, loss) => {
          console.log("training", epoch, loss);
        },
        () => {
          console.log("training finished");
          this.state = "over";
        }
      );
    },
    drawText(x, y) {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      ctx.beginPath();
      ctx.strokeStyle = "black";
      ctx.arc(x, y, 10, 0, Math.PI * 2, true);
      ctx.stroke();
      ctx.moveTo(300, 300);
      ctx.fillStyle = "black";
      ctx.font = '12px "微软雅黑"';
      ctx.textBaseline = "middle";
      ctx.textAlign = "center";
      ctx.fillText(this.text, x, y);
    },
    classify(inputs) {
      this.model.classify(inputs, (err, result) => {
        if (err) {
          console.log(err);
        } else {
          console.log(result);
          this.text = result[0].label;
          this.drawText(inputs.x, inputs.y);
        }
      });
    },
    predict(inputs) {
      this.model.predict(inputs, (err, result) => {
        if (err) {
          console.log(err);
        } else {
          console.log(result);
          this.text = result[0].label;
          this.drawText(inputs.x, inputs.y);
        }
      });
    },
    saveData() {
      this.model.saveData("model.json");
    },
    save() {
      this.model.save("model.json");
    },
  },
};
</script>

<style scoped></style>
