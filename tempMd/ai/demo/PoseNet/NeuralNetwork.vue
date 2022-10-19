<template>
  <div>
    <div>姿势</div>
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
      poseNet: "",
      video: "",
      brain: "",
      result: [],
    };
  },
  mounted() {
    // this.init();
  },
  methods: {
    init() {
      this.draw_video();
      this.draw_video_canvas();
      this.poseNet = this.$ml5.poseNet(this.video, this.modelLoaded);
      this.poseNet.on("pose", (result) => {
        console.log(result);
        let array = result[0].pose.keypoints;
        let input = [];
        for (let i = 0; i < array.length; i++) {
          input.push(array[i].position.x);
          input.push(array[i].position.y);
        }
        let target = ['x']
        this.brain.addData(input,target);
        // this.brain.saveData();
      });
      let options = {
        inputs: 34,
        output: 4,
        task: "classification",
        debug: true,
      };
      this.brain = this.$ml5.neuralNetwork(options);
    },
    modelLoaded() {
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
  },
};
</script>

<style scoped></style>
