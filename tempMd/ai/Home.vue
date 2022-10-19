<template>
  <div class="main">
    <span style="color: #b81917">Beigin-To-Machine-Learning!</span>
    <div class="scale-up-center" v-if="showStart">
      <el-button type="text" size="large" class="start" @click="begin"
        >开始使用</el-button
      >
    </div>
  </div>
</template>

<script>
export default {
  props: ["show"],
  data() {
    return {
      step: 2,
      showStart: false,
    };
  },
  mounted() {
    setTimeout(() => {
      this.showStart = true;
    }, 2100);
  },
  methods: {
    begin() {
      this.$emit("changeShow");
    },
  },
};
</script>

<style scoped>
.main {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
span {
  display: inline-block;
  width: 27ch;
  font: bold 200% Consolas, Monaco, monospace; /*等宽字体*/
  overflow: hidden;
  white-space: nowrap;
  font-weight: 500;
  border-right: 1px solid transparent;
  animation: typing 2s steps(27), caret 0.5s steps(1) infinite;
}
@keyframes typing {
  from {
    width: 0;
  }
}
@keyframes caret {
  50% {
    border-right-color: currentColor;
  }
}
.scale-up-center {
  display: flex;
  margin-top: 20px;
  justify-content: center;
  animation: scale-up-center 0.6s cubic-bezier(0.39, 0.575, 0.565, 1) both;
}

@keyframes scale-up-center {
  0% {
    transform: scale(0.5);
  }
  100% {
    transform: scale(1);
  }
}
.start {
  animation: blink-normal 0.6s infinite alternate;
}
@keyframes blink-normal {
  0% {
    color: rgba(45, 140, 240, 0.3);
  }
  100% {
    color: rgba(45, 140, 240, 0.8);
  }
}
</style>
