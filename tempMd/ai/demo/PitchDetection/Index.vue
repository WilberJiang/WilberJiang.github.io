<template>
  <div>
    <div>高音检测器</div>
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
      audioContext: "",
      stream: "",
      pitch: "",
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    async init() {
      this.audioContext = new AudioContext();
      this.stream = await navigator.mediaDevices.getUserMedia({
        audio: true,
        video: false,
      });
      this.pitch = this.$ml5.pitchDetection(
        "./model/",
        this.audioContext,
        this.stream,
        this.modelLoaded
      );
    },
    modelLoaded() {
      console.log("modelLoaded");
      this.getPitch();
    },
    getPitch() {
      this.pitch.getPitch((err, result) => {
        if (err) {
          console.log(err);
        } else {
          this.getPitch();
          if (result) console.log(result);
        }
      });
    },
  },
};
</script>

<style scoped></style>
