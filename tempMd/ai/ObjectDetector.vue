<template>
  <div class="container">123</div>
</template>

<script>
export default {
  data() {
    return {
      net: "",
    };
  },
  async mounted() {
    this.net = await this.$tf.loadGraphModel("model/model.json");
    // setInterval(() => {
    this.detect();
    // }, 16.7);
  },
  methods: {
    detect() {
      const image = new Image();
      image.onload = async () => {
        const img = this.$tf.browser.fromPixels(image);
        const resized = this.$tf.image.resizeBilinear(img, [640, 480]);
        const casted = resized.cast("int32");
        const expanded = casted.expandDims(0);
        const obj = await this.net.executeAsync(expanded,['detection_classes']);
        const boxes = await obj[4].array();
        const classes = await obj[5].array();
        const scores = await obj[6].array();
        const aaaaa0 = await obj[0].array();
        const aaaaa1 = await obj[1].array();
        const aaaaa2 = await obj[2].array();
        const aaaaa3 = await obj[3].array();
        const aaaaa4 = await obj[4].array();
        const aaaaa5 = await obj[5].array();
        const aaaaa6 = await obj[6].array();
        const aaaaa7 = await obj[7].array();
        console.log(
          "---0--",
          aaaaa0[0],
          "---1--",
          aaaaa1[0],
          "---2--",
          aaaaa2[0],
          "---3--",
          aaaaa3[0],
          "---4--",
          aaaaa4[0],
          "---5--",
          aaaaa5[0],
          "---6--",
          aaaaa6[0],
          "---7--",
          aaaaa7[0],
        );
        // console.log(obj,boxes[0], "------------", classes[0],"------------", scores[0]);
        this.getResult(boxes[0], classes[0], scores[0], 0.0);
        this.$tf.dispose(img);
        this.$tf.dispose(resized);
        this.$tf.dispose(casted);
        this.$tf.dispose(expanded);
        this.$tf.dispose(obj);
      };
      image.src = require("@/assets/ai/input13.png");
    },
    getResult(boxes, classes, scores, threshold) {
      for (let i = 0; i <= boxes.length; i++) {
        if (boxes[i] && classes[i] && scores[i] > threshold) {
          // Extract variables
          const [y, x, height, width] = boxes[i];
          const text = classes[i];
          console.log(text);
        }
      }
      console.log("finished");
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
}
</style>
