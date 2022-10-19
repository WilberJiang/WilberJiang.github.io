<template>
  <div>
    <div>物体检测器-图片</div>
    <div>
      <canvas
        ref="canvas"
        width="640"
        height="480"
        style="background: rgb(0, 0, 0)"
      ></canvas>
    </div>
    <div v-for="(item, index) in result" :key="index">
      {{ `${index + 1}.${item.label}-${item.confidence}` }}
    </div>
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
      detector: "",
      img: "",
      result: [],
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.draw_image();
      this.detector = this.$ml5.objectDetector("cocossd", this.modelLoaded);
    },
    modelLoaded() {
      console.log("modelLoaded");
      this.detector.detect(this.img, (err, result) => {
        if (err) {
          console.log(err);
        } else {
          console.log(result);
          this.result = result;
          const canvas = this.$refs.canvas;
          const ctx = canvas.getContext("2d");
          ctx.strokeStyle="lightgreen";
          for (let i = 0; i < this.result.length; i++) {
            const object = this.result[i];
            ctx.strokeRect(object.x, object.y, object.width, object.height);
          }
        }
      });
    },
    draw_image() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      // 检测支持性
      if (!ctx) return;
      //获得 2d 上下文对象
      let img = new Image(); //创建img元素
      img.onload = function () {
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      };
      img.src = require("../../assets/dog.png"); //设置图片源地址
      this.img = img;
    },
  },
};
</script>

<style scoped></style>
