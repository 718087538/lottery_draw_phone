<template>
  <div id="app">
    <div class="lottery-box" id="app">
      <!-- <h1 class="title">浪漫元旦礼遇季 空格豪礼送不停</h1> -->
      <div id="titleFont">
        <!--<img src="../static/titleFont.png" alt /> -->
      </div>
      <audio
        src="http://congraedu.com.cn/zc/cj/static/mp3/chouJiang.mp3"
        id="music1"
        controls="controls"
        loop
        v-show="false"
      ></audio>

      <img class="play" @click="play" src="../static/play.png" alt />
      <div class="lottery">
        <div class="lottery-item">
          <div class="lottery-start">
            <div class="box gray" v-if="isStart===0">
              <p>活动未开始</p>
            </div>
            <div class="box" :class="{startBtn:canPress}" @click="startLottery" v-if="isStart===1">
              <p>
                <b>抽奖</b>
              </p>
              <!-- <p>消耗{{score}}积分</p> -->
            </div>
            <div class="box gray" v-if="isStart===2">
              <p>活动已过期</p>
            </div>
          </div>
          <ul>
            <li v-for="(item,i) in list" :class="i==index?'on':''" :key="i">
              <div class="box">
                <p>
                  <img :src="item.img" alt />
                </p>
                <p>{{item.title}}</p>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <!-- 中奖弹窗 -->
      <div class="mask" v-if="showToast"></div>
      <div class="lottery-alert" v-if="showToast">
        <h1>恭喜您</h1>
        <p>
          <img :src="list[index].img" alt />
        </p>
        <h2>获得{{list[index].title}}</h2>
        <div class="btnsave" @click="sureBtn">确定</div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      canPress: false, //是否能点击多次按钮
      isStart: 1,
      score: 10, //消耗积分
      list: [
        {
          img: "static/hd.png",
          title: "海岛七日游"
        },
        {
          img: "static/iPhone-11pro-max.png",
          title: "苹果11 pro"
        },
        { img: "static/2999.png", title: "2999现金" },
        { img: "static/1999.png", title: "1999现金" },
        { img: "static/999.png", title: "999现金" },
        {
          img: "static/kx.png",
          title: "烤箱"
        },
        { img: "static/qjf.png", title: "全家福" },
        {
          img: "static/nh.png",
          title: "春节大礼包"
        }
      ], //奖品1-9
      index: -1, // 当前转动到哪个位置，起点位置
      count: 8, // 总共有多少个位置
      timer: 0, // 每次转动定时器
      speed: 200, // 初始转动速度
      times: 0, // 转动次数
      cycle: 50, // 转动基本次数：即至少需要转动多少次再进入抽奖环节
      prize: -1, // 中奖位置
      p4: 1,
      click: true,
      showToast: false //显示中奖弹窗
    };
  },

  mounted() {},

  methods: {
    sureBtn() {
      this.showToast = false;
      this.canPress = false;
    },
    play() {
      var audio = document.getElementById("music1");
      console.log(audio);
      if (audio !== null) {
        //检测播放是否已暂停.audio.paused 在播放器播放时返回false.
        // alert(audio.paused);
        if (audio.paused) {
          audio.play(); //audio.play();// 这个就是播放
        } else {
          audio.pause(); // 这个就是暂停
        }
      }
    },
    startLottery() {
      if (!this.click) {
        return;
      }

      //canpress 为true时代表点击了，抽奖中，不能再次点击

      if (this.canPress) {
        return false;
      }
      //通过了上面的代表刚开始点击，不在抽奖中
      this.canPress = true;
      this.startRoll();
    },
    // 开始转动
    startRoll() {
      this.times += 1; // 转动次数
      this.oneRoll(); // 转动过程调用的每一次转动方法，这里是第一次调用初始化 ,第二次调用时，第1次的方法选中才能生效
      // 如果当前转动次数达到要求 && 目前转到的位置是中奖位置
      if (this.times > this.cycle + 10 && this.prize === this.index) {
        //中奖
        clearTimeout(this.timer); // 清除转动定时器，停止转动    其实我觉得不用清除定时器,只要不进入else,则没有定时器会生效.
        this.prize = -1;
        this.times = 0;
        this.speed = 200;
        this.click = true;

        var that = this;

        setTimeout(res => {
          that.showToast = true;
        }, 500);
      } else {
        if (this.times < this.cycle) {
          //[0-50)
          this.speed -= 10; // 加快转动速度
        } else if (this.times === this.cycle) {
          const index = parseInt(Math.random() * 100, 0) || 0; // 随机获得一个中奖位置
          // this.prize = index; //中奖位置,可由后台返回
          if (index <= 40) {
            this.prize = 5;
          } else if (index > 40 && index <= 100) {
            this.prize = 7;
          } else {
            this.prize = 7;
          }
        } else if (
          this.times > this.cycle + 10 &&
          ((this.prize === 0 && this.index === 7) ||
            this.prize === this.index + 1)
        ) {
          this.speed += 110; //转动次数大于60，且中奖和当前差1个单位时开始大量减速， 为if为true的情况做准备(即显示中奖结果).
        } else {
          //[51,60],确定中奖位置后至少减速10次
          this.speed += 20; //以每次20的速度,一直减速到和中奖位置差1个位.
        }

        if (this.speed < 40) {
          this.speed = 40;
        }
        this.timer = setTimeout(this.startRoll, this.speed);
      }
    },

    // 每一次转动
    oneRoll() {
      let index = this.index; // 当前转动到哪个位置
      const count = this.count; // 总共有多少个位置
      index += 1;
      if (index > count - 1) {
        index = 0;
      } //转动的位置是0，1，2~7； 如果位置处于8，就重置为0。
      this.index = index; //转动1次,转动1个位置;
    },

    test() {
      console.log("sss");
    }
  },
  mounted() {
    // this.$axios({
    //   method: "get",
    //   url: "http://192.168.0.195:7004/api/prize" // 接口地址
    // })
    //   .then(res => {
    //     console.log(res, "success"); // 成功的返回
    //   })
    //   .catch(error => console.log(error, "error")); // 失败的返回
    // setTimeout(this.test, 1);
  }
};
</script>

<style>
.startBtn {
  background: rgb(151, 150, 150) !important;
  border-radius: 6px;
}
@media (min-width: 600px) {
  #titleFont {
    width: 100%;
    margin: 120px 0 10px;
    height: 200px !important;
    text-align: center;
  }
}

@media (min-width: 1000px) {
  #titleFont {
    width: 100%;
    margin: 120px 0 10px;
    height: 300px !important;
    text-align: center;
  }
}

/* iphonex */
@media (max-width: 420px) and (min-height: 700px) {
  #titleFont {
    width: 100%;
    margin: 120px 0 10px;
    height: 100px !important;
    text-align: center;
  }
}

#titleFont {
  width: 100%;
  margin: 120px 0 10px;
  height: 40px;
  text-align: center;
}
.play {
  width: 50px;
  height: 50px;
  bottom: 10px;
  right: 10px;
  position: fixed;
}

@keyframes changeBg {
  0% {
    background-image: url(https://s2.ax1x.com/2019/12/12/Qy2jCF.png);
  }
  100% {
    background-image: url(https://s2.ax1x.com/2019/12/12/Qy2xgJ.png);
  }
}

.lottery-box {
  overflow: hidden;
}
.lottery-box .title {
  text-align: center;
  padding: 20px 0;
  font-size: 18px;
  color: #fff;
}
.lottery {
  overflow: hidden;
  animation: changeBg 0.5s ease infinite;
  overflow: hidden;
  padding: 24px;
  width: 320px;
  height: 320px;
  margin: 0 auto;
  background-repeat: no-repeat;
  background-size: 100% 100%;
}

.lottery .lottery-item {
  height: 100%;
  position: relative;
}

.lottery .lottery-item ul li {
  width: 33.33333333%;
  height: 33.33333333%;
  position: absolute;
  padding-right: 10px;
  padding-bottom: 10px;
}

.lottery .lottery-item ul li:nth-child(2) {
  left: 33.33333333%;
}
.lottery .lottery-item ul li:nth-child(3) {
  left: 66.66666666%;
  padding-right: 0;
}
.lottery .lottery-item ul li:nth-child(4) {
  left: 66.66666666%;
  top: 90px;
  padding-right: 0;
}

.lottery .lottery-item ul li:nth-child(5) {
  left: 66.66666666%;
  top: 180px;
  padding-right: 0;
}
.lottery .lottery-item ul li:nth-child(6) {
  left: 33.33333333%;
  top: 180px;
}
.lottery .lottery-item ul li:nth-child(7) {
  left: 0;
  top: 180px;
}
.lottery .lottery-item ul li:nth-child(8) {
  left: 0;
  top: 90px;
}
.lottery .lottery-item ul li .box {
  height: 100%;
  position: relative;
  text-align: center;
  overflow: hidden;
  background: url(https://s2.ax1x.com/2019/12/12/Qy2HH0.png) no-repeat center;
  background-size: 100% 100%;
}
.lottery .lottery-item ul li .box img {
  display: block;
  height: 54px;
  margin: 0 auto;
  margin-top: 3px;
  /* margin-bottom: 5px; */
}
.lottery .lottery-item ul li .box p {
  color: hsl(220, 38%, 59%);
  /* white-space: nowrap; */
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 10x;
}
.lottery .lottery-item ul li.on .box {
  background: url(https://s2.ax1x.com/2019/12/12/Qy2qEV.png) no-repeat center;
  background-size: 100% 100%;
}
.lottery .lottery-item ul li.on .box p {
  color: #fff;
}

.lottery .lottery-item .lottery-start {
  position: absolute;
  left: 33.33333333%;
  width: 33.33333333%;
  height: 33.3333333%;
  top: 90px;
  padding-right: 10px;
  padding-bottom: 10px;
}
.lottery .lottery-item .lottery-start .box {
  height: 100%;
  font-size: 14px;
  color: #fff;
  cursor: pointer;
  text-align: center;
  overflow: hidden;
  background: url(https://s2.ax1x.com/2019/12/12/Qy2qEV.png) no-repeat center;
  background-size: 100% 100%;
}

.lottery .lottery-item .lottery-start .box p b {
  font-size: 30px;
  margin-top: 14px;
  margin-bottom: 15px;
  line-height: 55px;
  display: block;
}
.lottery .lottery-item .lottery-start .box:active {
  opacity: 0.7;
}
.lottery .lottery-item .lottery-start .box.gray {
  background: url(https://s2.ax1x.com/2019/12/12/Qy2Tun.png) no-repeat center;
  background-size: 100% 100%;
}
.lottery .lottery-item .lottery-start .box.gray p {
  color: #708abf;
  font-weight: bold;
}

/* ------------------------------------- */
* {
  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}
body {
  height: 100vh;
  /* background: radial-gradient(49% 160%, #22b5ff 0, #3a72fa 100%); */
  font-size: 14px;
  background: url(https://s2.ax1x.com/2019/12/13/Qg1MvQ.jpg) no-repeat;
  background-size: 100% 100%;
}
img {
  border: 0px;
}
ul,
li {
  list-style-type: none;
}
/* 这下面没问题 */
.mask {
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  position: fixed;
  overflow: hidden;
  z-index: 222;
  top: 0;
  left: 0;
}
.lottery-alert {
  max-width: 340px;
  text-align: center;
  z-index: 10000;
  border-radius: 10px;
  background: #fff;
  padding: 20px;
  position: fixed;
  left: 0;
  right: 0;
  margin: 20px auto;

  top: 50%;
  transform: translateY(-50%);
}
.lottery-alert h1 {
  font-size: 18px;
  font-weight: bold;
  color: #d92b2f;
}
.lottery-alert img {
  display: block;
  height: 120px;
  margin: 0 auto;
}
.lottery-alert h2 {
  font-weight: normal;
  color: #d92b2f;
  font-size: 15px;
  padding-top: 15px;
}
.lottery-alert p {
  color: #666;
  font-size: 16px;
  padding-top: 5px;
}
.lottery-alert .btnsave {
  border-radius: 3px;
  box-shadow: none;
  height: 40px;
  cursor: pointer;
  line-height: 40px;
  color: #fff;
  margin-top: 12px;
  background: linear-gradient(
    180deg,
    rgba(213, 60, 63, 1) 0%,
    rgba(201, 20, 24, 1) 100%
  );
  font-size: 16px;
}
</style>
