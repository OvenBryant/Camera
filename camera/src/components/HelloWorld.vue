<template>
  <div class="video-wrap">
    <div
      id="video-container"
      class="video-container"
      :style="{ CameraBackgroundColor }"
    ></div>

    <div class="box" v-if="coverDiv">
        <span style="color:white;font-size:30px">已 关 闭</span> 
    </div> 

    <div class="ctrl-panel">
      <div class="ctrl-row">
        <el-button @click="moveUp">上</el-button>
      </div>
      <div class="ctrl-row">
        <el-button @click="moveLeft">左</el-button>
        <el-button @click="moveDown">下</el-button>
        <el-button @click="moveRight">右</el-button>
      </div>
      <div class="ctrl-row">
        <el-button @click="ConRecording">连接</el-button>
        <el-button @click="stopRecording">关闭</el-button>
        <el-button @click="takeSnapshot">抓拍</el-button>
        <el-button @click="showParamsDialog">参数设置</el-button>
      </div>

      <div class="ctrl-row">
        <el-button v-on:click="ezopenStartTalk">开始对讲</el-button>
        <el-button v-on:click="ezopenStopTalk">结束对讲</el-button>
      </div>
      <div class="ctrl-row">
        <el-button @click="stopVideo">停止录像</el-button>
        <el-button @click="startVideo">
          开始录像
        </el-button>
      </div>

      <!-- 摄像头参数设置 -->
      <el-drawer
        title="摄像头参数设置"
        style="font-weight: bold; font-size: 25px"
        :visible.sync="CameraConfigvisible"
        custom-class="demo-drawer"
      >
        <div style="display: flex; margin-top: 50px">
          <el-form
            :model="cameraConfig"
            ref="CameraConfigRef"
            style="width: 100%"
          >
            <el-form-item
              style="font-weight: bold"
              label="accessToken"
              :label-width="formLabelWidth"
            >
              <el-input
                class="addManageInput"
                v-model="cameraConfig.accessToken"
                style="width: 300px"
                type="password"
                placeholder="请输入accessToken"
                autocomplete="off"
              ></el-input>
            </el-form-item>

            <el-form-item
              prop="clientId"
              style="font-weight: bold"
              label="url"
              :label-width="formLabelWidth"
            >
              <el-input
                class="addManageInput"
                v-model="cameraConfig.url"
                style="width: 300px"
                placeholder="Url"
                autocomplete="off"
              ></el-input>
            </el-form-item>

            <el-form-item
              style="font-weight: bold"
              :label-width="formLabelWidth"
            >
              <el-button type="primary" @click="ConCameraService()"
                >连 接</el-button
              >
              <el-button
                type="danger"
                @click="unConCameraService()"
                style="width: 100px; font-weight: bolder"
                >取消</el-button
              >
            </el-form-item>
            <!-- ====================底部  @click="$refs.drawer.closeDrawer()"============= -->
          </el-form>
        </div>
      </el-drawer>
    </div>
  </div>
</template>

<script>
import EZUIKit from "ezuikit-js";

export default {
  data() {
    return {
      channelNo: 1, // 通道号
      direction: 0, // 方向 0-上，1-下，2-左，3-右，(只支持上下左右) 4-左上，5-左下，6-右上，7-右下，8-放大，9-缩小，10-近焦距，11-远焦距
      speed: 1, // 云台速度 0慢 1适中 2快
      deviceSerial:'',

      CameraConfigvisible: false,
      formLabelWidth: "130px",
      cameraConfig: {
        accessToken: "",
        url: "",
      },

      CameraBackgroundColor: "transparent",
      player: null,
      coverDiv:false
    };
  },
  // 初始化连接一次
  mounted() {
    this.player = new EZUIKit.EZUIKitPlayer({
      id: "video-container", // 视频容器ID
      accessToken: this.cameraConfig.accessToken,
      url: this.cameraConfig.url,
      // simple - 极简版; pcLive-pc直播；pcRec-pc回放；mobileLive-移动端直播；mobileRec-移动端回放;security - 安防版;voice-语音版;
      //template: 'pcRec',
      plugin: ["talk"], // 加载插件，talk-对讲
      width: 900,
      height: 700,
    });

    // const video = document.getElementById("video-container");
    // video.addEventListener("mousewheel", this.preventWheel);

    console.log("player对象数据: ", this.player);

    encryptOff(this.$request); // 关闭设备视频加密

    //this.startVideo(); // 自动录屏

  },

  beforeDestroy() {

    //this.stopVideo(); // 停止录屏

    if (this.player != null) {
      this.stopRecording(); // 停止
    }
  },
  methods: {
    encryptOff(){
        const CloseEncryUrl = "https://open.ys7.com/api/lapp/device/encrypt/off";
        const payload = {
          accessToken: this.cameraConfig.accessToken,
          deviceSerial: this.deviceSerial,
        };
        const config = {
          headers: {
            "Content-Type": "application/x-www-form-urlencoded",
          },
        };
        try {
          request.post(CloseEncryUrl, payload, config).then(res=>{
            console.log('===============================')
            console.log('当前code',res.code)
            console.log('当前msg',res.msg)
            console.log('===============================')

          })

  
          // window.setTimeout(()=>{
          //   this.request.post(stopUrl,payload,config)
          // },200)
        } catch (error) {
          console.error("关闭设备视频加密-Error: ", error);
        }
    },
    preventWheel(e) {
      e.preventDefault();
    },
    // 上下左右开启
    async startPTZ(direction) {
      const startUrl = "https://open.ys7.com/api/lapp/device/ptz/start";
      const stopUrl = "https://open.ys7.com/api/lapp/device/ptz/stop";
      const payload = {
        accessToken: this.cameraConfig.accessToken,
        deviceSerial: this.deviceSerial,
        channelNo: this.channelNo,
        direction: direction,
        speed: this.speed,
      };
      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
        },
      };
      try {
        const response = await this.$request.post(startUrl, payload, config);
        console.log("开启: ", response);
        //当摄像头向左移动，如果不调用停止接口，那么摄像头会一直向左移动，直到转不动或者调用了停止接口才会停下来
        // 要关闭的话，可以弄个延时的可以延长时间
        const response1 = await this.$request.post(stopUrl, payload, config);
        console.log("关闭: ", response1);

        // window.setTimeout(()=>{
        //   this.request.post(stopUrl,payload,config)
        // },200)
      } catch (error) {
        console.error("Error: ", error);
      }
    },

    // 通知
    notifyShow(_title, _message) {
      this.$notify({
        title: _title,
        message: this.$createElement("b", { style: "color:red" }, _message),
      });
    },

    // 上
    moveUp() {
      this.notifyShow("上", "上方向移动");
      this.startPTZ(0);
    },
    // 左
    moveLeft() {
      this.notifyShow("左", "左方向移动");
      this.startPTZ(2);
    },
    // 下
    moveDown() {
      this.notifyShow("下", "下方向移动");
      this.startPTZ(1);
    },
    // 右
    moveRight() {
      this.notifyShow("右", "右方向移动");
      this.startPTZ(3);
    },
    // 拍照
    takeSnapshot() {
      var capturePicturePromise = this.player.capturePicture(
        `${new Date().getTime()}`
      );
      capturePicturePromise.then((data) => {
        console.log("promise 获取 数据", data);
      });
    },

    // 连 接
    ConRecording() {

      this.coverDiv = false;

      var playPromise = this.player.play();
      playPromise.then((data) => {
        console.log("promise 获取 数据", data);
      });
    },
    // 关闭
    stopRecording() {

      this.coverDiv = true;  // 显示盒子，覆盖摄像头

      if (this.player != null) {
        var stopPromise = this.player.stop();
        stopPromise.then((data) => {
          console.log("promise 获取 数据", data);
        });
      }

      this.CameraBackgroundColor = "red"; // 设置为红色背景
    },
    // 参数设置
    showParamsDialog() {
      this.CameraConfigvisible = true;
    },

    // 停止录像
    stopVideo() {
      this.notifyShow("停止", "已停止录像...");
      var stopSavePromise = this.player.stopSave();
      stopSavePromise.then(data => {
        console.log("promise 获取 数据", data);
      }).catch(error=>{
        console.error('录像失败:', error);
      });
    },

    // 开启录像
    startVideo() {
      this.notifyShow("开始", "正在开始录像...");
      var startSavePromise = this.player.startSave(`${new Date().getTime()}`);
      startSavePromise.then((data) => {
        console.log("promise 获取 数据", data);
      }).catch(error=>{
         console.error('录像失败:', error);
      });
    },

    // 开始对讲
    ezopenStartTalk() {
      this.notifyShow("开始", "已开启对讲...");
      this.player.startTalk();
    },
    // 停止对讲
    ezopenStopTalk() {
      this.notifyShow("停止", "已停止对讲...");
      this.player.stopTalk();
    },
    // 销毁
    destroy() {
      var destroyPromise = this.player.destroy();
      destroyPromise.then((data) => {
        console.log("promise 获取 数据", data);
      });
    },

    // 连接摄像头参数按钮
    ConCameraService() {

      this.coverDiv = false;  // 显示摄像头


     if (this.player != null) {
        var stopPromise = this.player.stop();
        stopPromise.then((data) => {
          console.log("promise 获取 数据", data);
        });
      }

      this.player = new EZUIKit.EZUIKitPlayer({
        id: "video-container", // 视频容器ID
        accessToken: this.cameraConfig.accessToken,
        url: this.cameraConfig.url,
        validCode: "IDRSKI",
        plugin: ["talk"], // 加载插件，talk-对讲
        width: 900,
        height: 700,
      });

      encryptOff(this.$request); // 关闭设备视频加密

      if (this.player != null) {
        this.notifyShow("摄像头状态 : ", "连接成功!");
      } else {
        this.notifyShow("摄像头状态 : ", "连接失败!");
      }

     
      this.CameraConfigvisible = false; 
    },

    unConCameraService() {
      this.CameraConfigvisible = false;
    },
  },
};
</script>

<style scoped>
.video-wrap {
  display: flex;
  flex-wrap: wrap;
}
.video-container {
  width: 900px;
  height: 700px;
  background-color: #000;
}
.ctrl-panel {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 50px;
}
.ctrl-row {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 5px 0;
}
.ctrl-btn {
  margin: 0 5px;
  padding: 8px 12px;
  font-size: 16px;
  border-radius: 4px;
  cursor: pointer;
  background-color: #ccc;
  border: none;
  outline: none;
  box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.4);
}
.ctrl-btn.active {
  background-color: #f00;
  color: #fff;
}
.black-screen {
  background: black;
}

.box{
  width: 900px;
  height: 700px;
  background-color: #000;
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
}

</style>