<template>
  <div>
    <div>图像分类器-图片</div>
    <div>
      <canvas
        ref="canvas"
        width="640"
        height="480"
        style="background: rgb(0, 0, 0)"
      ></canvas>
    </div>
    <div v-for="(item,index) in result" :key="index">{{`${index+1}.${item.label}-${item.confidence}`}}</div>
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
      mobileNet: "",
      img: "",
      result:[]
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.draw_image();
      this.mobileNet = this.$ml5.imageClassifier("MobileNet", this.modelLoaded);
    },
    modelLoaded() {
      console.log("modelLoaded");
      this.mobileNet.predict(this.img, (err, result) => {
        if (err) {
          console.log(err);
        } else {
          console.log(result);
          this.result = result
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
      img.src = require("../../assets/cat.png"); //设置图片源地址
      this.img = img;
    },
  },
};
</script>

<style scoped></style>
