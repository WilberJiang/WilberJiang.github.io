<template>
  <div>
    <div>KNN分类</div>
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
    <button @click="btnClick1">点击左</button>
    <button @click="btnClick2">点击右</button>
    <button @click="check">查看</button>
    <button @click="save">保存</button>
    <div style="display: none">
      <video ref="video" autoplay style="width: 640px; height: 480px"></video>
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
      features: "",
      knn: "",
      video: "",
      result: [],
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      this.draw_video();
      this.draw_video_canvas();
      this.features = this.$ml5.featureExtractor("MobileNet", this.modelLoaded);
    },
    modelLoaded() {
      console.log("modelLoaded");
      this.knn = this.$ml5.KNNClassifier();
      // this.knn.load()
    },
    draw_video() {
      const video = this.$refs.video;
      if (navigator.getUserMedia) {
        navigator.getUserMedia(
          {
            video: true,
          },
          function (stream) {
            video.srcObject = stream;
            video.play();
          },
          () => {
            console.log("error");
          }
        );
      } else if (navigator.webkitGetUserMedia) {
        navigator.webkitGetUserMedia(
          {
            video: true,
          },
          function (stream) {
            video.srcObject = window.webkitURL.createObjectURL(stream);
            video.play();
          },
          () => {
            console.log("error");
          }
        );
      }
      this.video = video;
    },
    draw_video_canvas() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      ctx.drawImage(this.video, 0, 0, canvas.width, canvas.height);
      requestAnimationFrame(this.draw_video_canvas);
    },
    btnClick1() {
      this.logits = this.features.infer(this.video);
      this.knn.addExample(this.logits, "left");
      // this.logits.print()
      console.log(this.logits.dataSync());
    },
    btnClick2() {
      this.logits = this.features.infer(this.video);
      this.knn.addExample(this.logits, "right");
      // this.logits.print()
      console.log(this.logits.dataSync());
    },
    check() {
      if (this.knn.getNumLabels() > 0) {
        this.logits = this.features.infer(this.video);
        this.knn.classify(this.logits, (err, result) => {
          if (err) {
            console.log(err);
          } else {
            console.log(result);
          }
        });
      }
    },
    save(){
      this.knn.save('model.json')
    }
  },
};
</script>

<style scoped></style>
