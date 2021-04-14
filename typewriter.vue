<template>
  <span
    >{{ outputText }}
    <!--打印闪烁光标 -->
    <span class="blank" :style="aniStyle">|</span>
  </span>
</template>

<script>
export default {
  name: "typerwriter",
  props: {
    text: String,
    texts: {
      type: Array,
      default: new Array(),
    },
    // 打印速度
    wspeed: {
      type: Number,
      default: 150,
    },
    // 开启回溯打印
    isBack: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      loaclTexts: this.texts,
      outputText: "",
      // 当前打印字符串Index
      currentTextCount: 1,
      // 当前打印数组指针
      currentTextArrayIndex: 0,
      timer: Number,
      logPrefix: "typewriter[log]: ",
      forwadFlag: true, // 用来标记当前打印方向
    };
  },
  computed: {
    aniStyle() {
      return {
        // 光标播放速度
        // 根据设置的wspeed调整光标闪烁动画速度，
        "animation-duration": this.wspeed / 1000 + "s",
      };
    },
  },
  created() {
    this.typewriter();
  },
  unmounted() {
    clearInterval(this.timer);
  },
  methods: {
    typewriter() {
      // 参数验证
      if (!this.argumentJudge()) {
        return;
      }
      // 只设置了text，将text放入loaclTexts
      if (this.loaclTexts.length == 0) {
        this.loaclTexts.push(this.text);
      }
      // 开启打印
      this.timer = setInterval(this.forwardWriter, this.wspeed);
    },
    forwardWriter() {
      // 全部打印完判断
      if (this.currentTextArrayIndex >= this.loaclTexts.length) {
        // 往回打印的情况，重置参数
        if (this.isBack) {
          this.currentTextArrayIndex = 0;
          this.currentTextCount = 1;
        } else {
          // 不往回打印, 取消返回
          clearInterval(this.timer);
          return;
        }
      }
      if (this.forwadFlag) {
        // 往前打印
        if (
          this.currentTextCount <=
          this.loaclTexts[this.currentTextArrayIndex].length
        ) {
          this.outputText = this.loaclTexts[
            this.currentTextArrayIndex
          ].substring(0, this.currentTextCount);
          this.currentTextCount++;
        } else {
          // 当前字符串打印完，指向下一个并把索引归1
          // 判断是否需要向后打印
          if (this.isBack) {
            this.forwadFlag = false;
          } else {
            this.currentTextArrayIndex++;
            this.currentTextCount = 1;
          }
        }
      } else {
        // 向后打印
        if (this.isBack) {
          clearInterval(this.timer);
          // 向后打印2.5倍速度
          this.timer = setInterval(this.backWriter, this.wspeed / 2.5);
        }
      }
    },
    backWriter() {
      if (this.currentTextCount >= 0) {
        this.outputText = this.loaclTexts[this.currentTextArrayIndex].substring(
          0,
          this.currentTextCount
        );
        this.currentTextCount--;
      } else {
        // 当前字符串打印完
        // 指针自增打印下一个字符串
        this.currentTextArrayIndex++;
        this.forwadFlag = true;
        this.currentTextCount = 1;
        // 取消当前timer
        clearInterval(this.timer);
        this.timer = setInterval(this.forwardWriter, this.wspeed);
      }
    },
    argumentJudge() {
      if (this.wspeed == null) {
        console.log(this.logPrefix + "未设置打印速度");
        return false;
      } else if (this.wspeed <= 0) {
        console.log(this.logPrefix + "速度参数不对");
        return false;
      } else if (this.text == null && this.texts == null) {
        console.log(this.logPrefix + "no string can writer");
        return false;
      }
      return true;
    },
  },
};
</script>

<style scoped>
/* 光标闪烁动画 */
@keyframes Blink {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
.blank {
  animation-name: Blink;
  animation-timing-function: ease-in;
  animation-iteration-count: infinite;
}
</style>
