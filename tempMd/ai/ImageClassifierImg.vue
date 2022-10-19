<template>
  <div class="container">
    <div class="input">
      <el-card
        class="box-card"
        v-for="(item, index) in classifyList"
        :key="index"
      >
        <div slot="header" class="clearfix">
          <el-input
            :ref="`input-${index}`"
            size="mini"
            v-model="item.name"
            @blur="editName(index, false)"
            style="width: 150px"
            v-show="item.isEdit"
          ></el-input>
          <span v-if="!item.isEdit"
            >{{ item.name
            }}<i
              class="el-icon-edit-outline edit"
              @click="editName(index, true)"
            ></i
          ></span>

          <i class="el-icon-delete delete" @click="deleteClassify(index)"></i>
        </div>
        <div class="box-card-content">
          <div
            v-show="item.status === 'play' && !item.message"
            class="video-content"
          >
            <i
              class="el-icon-circle-close close"
              @click="close_video(index)"
            ></i>
            <canvas
              :ref="`canvas-${index}`"
              width="200"
              height="200"
              class="canvas"
            ></canvas>
            <div class="btns">
              <el-button
                size="mini"
                type="primary"
                icon="el-icon-video-camera"
                @click="start_record(index)"
                >开始记录</el-button
              >
              <el-button
                size="mini"
                type="danger"
                icon="el-icon-coin"
                @click="stop_record(index)"
                >暂停记录</el-button
              >
            </div>
          </div>
          <div
            class="video-content"
            v-show="!(item.status === 'play' && !item.message)"
          >
            <div v-if="item.message">
              {{ item.message }}
            </div>
            <div v-if="item.status !== 'play'" class="upload-img">
              <el-button
                size="mini"
                type="primary"
                icon="el-icon-video-camera-solid"
                @click="open_video(index)"
                style="margin-right: 10px"
              ></el-button>
              <el-upload
                action=""
                :auto-upload="false"
                :show-file-list="false"
                :on-change="
                  (file) => {
                    uploadImg(file, index);
                  }
                "
                multiple
              >
                <el-button
                  icon="el-icon-picture"
                  size="mini"
                  type="primary"
                ></el-button>
              </el-upload>
            </div>
          </div>
          <div class="img-list">
            <img
              class="img-list-item"
              :src="img"
              alt=""
              v-for="(img, imgIndex) in item.imgList"
              :key="imgIndex"
            />
          </div>
        </div>
      </el-card>
      <div class="add-classify" @click="addClassify">
        <span class="add-classify-content">
          <i class="el-icon-plus"></i> 添加分类</span
        >
      </div>
      <!-- <el-button @click="classifier.save()">save</el-button> -->
    </div>
    <div class="train">
      <el-card class="box-card">
        <div slot="header" class="clearfix">模型训练</div>
        <div style="text-align: center">
          <div v-if="!isModelLoaded" style="margin-bottom: 10px">
            模型加载中
            <i class="el-icon-loading"></i>
          </div>
          <div v-if="loadError" style="margin-bottom: 10px">
            <div>模型加载失败，</div>
            <div>请检查网络代理。</div>
          </div>
          <div v-if="training" style="margin-bottom: 10px">
            模型训练中<i class="el-icon-loading"></i>
            <div>
              <span class="margin-right:50px">总数：{{ imgCount }}</span>
              <span>已训练{{ completeImgCount }}</span>
              <div>完成度：{{ lossValue }}%</div>
            </div>
          </div>
          <el-button
            size="mini"
            type="primary"
            :disabled="!isModelLoaded || training || loadError"
            @click="train"
            >开始</el-button
          >
        </div>
      </el-card>
      <el-card class="box-card" v-if="result && result.length > 0 && !training">
        <div slot="header" class="clearfix">模型下载</div>
        <div style="text-align: center">
          <el-button
            size="mini"
            type="primary"
            :disabled="!isModelLoaded || training || loadError"
            @click="classifier.save()"
            >下载</el-button
          >
        </div>
      </el-card>
    </div>
    <div class="output" v-show="result && result.length > 0 && !training">
      <el-card class="box-card">
        <div slot="header" class="clearfix">模型预测</div>
        <div>
          <canvas
            ref="canvas-output"
            width="400"
            height="400"
            class="canvas"
          ></canvas>
        </div>
        <div class="result-list">
          <div v-for="(item, index) in result" :key="index">
            <div>
              {{ item.label }}
            </div>
            <el-progress
              :stroke-width="14"
              :percentage="Number((item.confidence * 100).toFixed(2))"
              :color="customColors"
              class="progress"
            ></el-progress>
          </div>
        </div>
      </el-card>
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
      isModelLoaded: false,
      loadError: false,
      classifyList: [
        {
          name: "分类1",
          isEdit: false,
          message: "",
          status: "",
          imgList: [],
        },
        {
          name: "分类2",
          isEdit: false,
          message: "",
          status: "",
          imgList: [],
        },
      ],
      loadOtipns: {
        numLabels: 100,
      },
      imgCount: 0,
      completeImgCount: 0,
      lossValue: 0,
      isRecording: false,
      training: false,
      completeTraining: false,
      featureExtractor: "",
      classifier: "",
      video: "",
      result: [],
      customColors: [
        { color: "#f56c6c", percentage: 20 },
        { color: "#e6a23c", percentage: 40 },
        { color: "#5cb87a", percentage: 60 },
        { color: "#1989fa", percentage: 80 },
        { color: "#6f7ad3", percentage: 100 },
      ],
    };
  },
  mounted() {
    this.video = this.$refs.video;
    this.init();
  },
  methods: {
    init() {
      this.featureExtractor = this.$ml5.featureExtractor(
        "MobileNet",
        this.loadOtipns,
        this.modelLoaded
      );
    },
    modelLoaded(err) {
      this.isModelLoaded = true;
      if (err) {
        this.loadError = true;
        return;
      }
      this.classifier = this.featureExtractor.classification(
        this.video,
        () => {}
      );
    },
    open_video(index) {
      this.isRecording = false;
      this.classifyList.forEach((item, classify) => {
        this.close_video(classify);
      });
      let options = {
        video: { width: 1000, height: 1000 },
      };
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices
          .getUserMedia(options)
          .then((stream) => {
            this.video.srcObject = stream;
            this.video.play();
            this.draw_video_canvas(index);
            this.classifyList[index].status = "play";
            this.classifyList[index].message = "";
          })
          .catch(() => {
            this.video_error(index);
          });
      }
    },
    close_video(index) {
      this.isRecording = false;
      this.classifyList[index].status = "pause";
      this.classifyList[index].message = "";
    },
    video_error(index) {
      this.classifyList[index].message = "请检查摄像头";
    },
    draw_video_canvas(index) {
      const canvas = this.$refs[`canvas-${index}`][0];
      const ctx = canvas.getContext("2d");
      ctx.drawImage(this.video, 0, 0, canvas.width, canvas.height);
      requestAnimationFrame(() => {
        this.draw_video_canvas(index);
      });
    },
    deleteClassify(index) {
      if (this.classifyList.length <= 2) {
        this.$message.warning("分类不能小于两种");
        return;
      }
      this.isRecording = false;
      this.classifyList.splice(index, 1);
    },
    start_record(index) {
      this.isRecording = true;
      this.pushImg(index);
    },
    stop_record() {
      this.isRecording = false;
    },
    pushImg(index) {
      const canvas = this.$refs[`canvas-${index}`][0];
      this.classifyList[index].imgList.unshift(canvas.toDataURL("image/png"));
      setTimeout(() => {
        if (this.isRecording) {
          this.pushImg(index);
        }
      }, 100);
    },
    editName(index, status) {
      if (status) {
        setTimeout(() => {
          this.$refs[`input-${index}`][0].focus();
        }, 0);
      }
      this.classifyList[index].isEdit = status;
    },
    train() {
      this.imgCount = 0;
      this.completeImgCount = 0;
      this.training = true;
      this.classifyList.forEach((item) => {
        item.imgList.forEach((img) => {
          this.imgCount += 1;
          this.addImage(img, item.name);
        });
      });
      if (this.imgCount === 0) {
        this.$message.warning("请添加训练图片");
        this.training = false;
        return;
      }
      this.classifyList.forEach((item, classify) => {
        this.close_video(classify);
      });
    },
    addImage(src, name) {
      let image = new Image();
      image.onload = () => {
        this.completeImgCount += 1;
        this.classifier.addImage(image, name, (err) => {
          if (this.imgCount === this.completeImgCount) {
            this.classifier.train((lossValue) => {
              if (lossValue) {
                if (lossValue > 1) {
                  this.lossValue = 0;
                } else {
                  this.lossValue = ((1 - lossValue) * 100).toFixed(2);
                }
              } else {
                this.lossValue = 0;
              }
              if (lossValue === null) {
                this.training = false;
                this.completeTraining = true;
                this.begin_predict();
              }
            });
          }
        });
      };
      image.src = src;
    },
    begin_predict() {
      let options = {
        video: { width: 1000, height: 1000 },
      };
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices
          .getUserMedia(options)
          .then((stream) => {
            this.video.srcObject = stream;
            this.video.play();
            this.draw_predict_canvas();
            this.video.onloadeddata = () => {
              this.result = [];
              this.predict();
            };
          })
          .catch(() => {
            this.video_error(index);
          });
      }
    },
    predict() {
      this.classifier.classify(this.video, (err, result) => {
        this.result = result
          ? result.filter((item) => {
              return typeof item.label !== "number";
            })
          : [];
        setTimeout(() => {
          this.predict();
        }, 100);
      });
    },
    draw_predict_canvas() {
      const canvas = this.$refs["canvas-output"];
      const ctx = canvas.getContext("2d");
      ctx.drawImage(this.video, 0, 0, canvas.width, canvas.height);
      requestAnimationFrame(() => {
        this.draw_predict_canvas();
      });
    },
    addClassify() {
      this.classifyList.push({
        name: `分类${this.classifyList.length + 1}`,
        isEdit: false,
        message: "",
        status: "",
        imgList: [],
      });
    },
    uploadImg(file, index) {
      var reader = new FileReader();
      reader.onloadend = (e) => {
        this.classifyList[index].imgList.unshift(e.target.result);
      };
      reader.readAsDataURL(file.raw);
    },
    // uploadResultImg(file) {
    //   var reader = new FileReader();
    //   reader.onloadend = (e) => {
    //     let image = new Image();
    //     image.onload = () => {
    //       this.classifier.classify(image, (err, result) => {
    //         this.result = result
    //           ? result.filter((item) => {
    //               return typeof item.label !== "number";
    //             })
    //           : [];
    //       });
    //     };
    //     image.src = e.target.result;
    //   };
    //   reader.readAsDataURL(file.raw);
    // },
  },
};
</script>

<style scoped>
.container {
  width: 1200px;
  height: 100%;
  display: flex;
  padding: 20px;
  box-sizing: border-box;
  margin: auto;
  overflow: auto;
}
.container .input {
  overflow: auto;
  width: 425px;
  box-sizing: border-box;
  flex-shrink: 0;
}
.container .train {
  width: 200px;
  margin: 0 50px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  flex-shrink: 0;
}
.container .output {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.box-card {
  margin-bottom: 20px;
}
.box-card-content {
  display: flex;
}
.edit {
  color: #409eff;
  cursor: pointer;
  margin-left: 5px;
}
.delete {
  float: right;
  padding: 3px 0;
  color: #f56c6c;
  cursor: pointer;
}
.canvas {
  transform: rotateY(180deg);
  /* background: rgb(0, 0, 0); */
  border-radius: 5px;
}
.btns {
  display: flex;
  justify-content: space-between;
}
.upload-img {
  display: flex;
}
.video-content {
  flex-shrink: 0;
  width: 200px;
  position: relative;
}
.img-list {
  padding-left: 5px;
  display: flex;
  flex-wrap: wrap;
  align-content: baseline;
  overflow: auto;
  max-height: 234px;
}
.img-list-item {
  transform: rotateY(180deg);
  width: 60px;
  height: 60px;
  margin-right: 5px;
  margin-bottom: 5px;
  border-radius: 5px;
}
.close {
  z-index: 10;
  position: absolute;
  top: 5px;
  right: 5px;
  font-size: 18px;
  color: #f56c6c;
  cursor: pointer;
  font-weight: bolder;
}
.progress {
  display: flex;
  align-items: center;
}
.add-classify {
  height: 75px;
  color: #aaa;
  border: 7px dashed #aaa;
  border-radius: 5px;
  position: relative;
}
.add-classify-content {
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  right: -6px;
  left: -6px;
  top: -6px;
  bottom: -6px;
  background: #fff;
  cursor: pointer;
}
.add-classify-content:hover {
  color: #409eff;
}
.result-list {
  height: 150px;
  overflow: auto;
}
::v-deep .el-card {
  font-size: 14px;
}
::v-deep .el-card__header {
  padding: 10px;
}
::v-deep .el-card__body {
  padding: 10px;
}
::v-deep .el-button + .el-button {
  margin-left: 0;
}
::v-deep .el-progress__text {
  flex-shrink: 0;
  width: 70px;
}
::v-deep .el-progress-bar {
  flex-grow: 1;
}
</style>
