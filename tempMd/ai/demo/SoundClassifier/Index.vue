<template>
  <div>
    <div>声音分类器</div>
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
      soundClassifier: "",
    };
  },
  mounted() {
    this.init();
  },
  methods: {
    init() {
      const options = { probabilityThreshold: 0.7 }
      this.soundClassifier = this.$ml5.soundClassifier("SpeechCommands18w",options, this.modelLoaded);
    },
    modelLoaded() {
      console.log("modelLoaded");
      this.soundClassifier.classify(this.img, (err, result) => {
        if (err) {
          console.log(err);
        } else {
          console.log(result);
        }
      });
    },
  },
};
</script>

<style scoped></style>
