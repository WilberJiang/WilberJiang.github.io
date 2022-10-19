<template>
  <div>
    <div>特征提取器-保存和加载</div>
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
    <button @click="add1">添加杯子</button>
    <button @click="add2">添加手机</button>
    <button @click="train">训练</button>
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
      mobileNet: "",
      classifier: "",
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
      this.mobileNet = this.$ml5.featureExtractor(
        "MobileNet",
        this.modelLoaded
      );
      this.classifier = this.mobileNet.classification(this.video, () => {
        console.log("video is ready");
      });
    },
    modelLoaded() {
      this.classifier.load('model.json',()=>{
        console.log('custom modelLoaded');
        this.classify()
      })
      console.log("modelLoaded");
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
    add1() {
      this.classifier.addImage("杯子");
    },
    add2() {
      this.classifier.addImage("手机");
    },
    train() {
      this.classifier.train((loss) => {
        if (loss === null) {
          console.log('train complete');
          this.classify();
        }
      });
    },
    classify() {
      this.classifier.classify((err, result) => {
        if (err) {
          console.log(err);
        } else {
          this.classify()
          console.log(result);
          this.result = result;
        }
      });
    },
    save(){
      this.classifier.save()
    }
  },
};
</script>

<style scoped></style>
