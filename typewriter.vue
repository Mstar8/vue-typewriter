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
    texts: Array,
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
      // 先判断打印字符串还是字符数组
      if (this.text == null && this.texts != null) {
        // 打印字符数组
        this.timer = setInterval(this.writerArray, this.wspeed);
      } else if (this.text != null && this.texts == null) {
        // 打印字符
        this.timer = setInterval(this.writerString, this.wspeed);
      } else if (this.texts != null && this.text != null) {
        // 两者都有打印字符数组
        this.timer = setInterval(this.writerArray, this.wspeed);
      }
    },
    writerArray() {
      if (this.currentTextArrayIndex >= this.texts.length) {
        // 往回打印的情况
        if (this.isBack) {
          this.currentTextArrayIndex = 0;
          this.currentTextCount = 1;
        } else {
          return;
        }
      }
      if (this.forwadFlag) {
        if (
          // 打印
          this.currentTextCount <= this.texts[this.currentTextArrayIndex].length
        ) {
          this.outputText = this.texts[this.currentTextArrayIndex].substring(
            0,
            this.currentTextCount
          );
          this.currentTextCount++;
        } else {
          // 当前字符串打印完，指向下一个并把索引归1
          // 判断是否需要向后消除打印
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
          // 向后打印2倍速度
          this.timer = setInterval(this.backWriterArray, this.wspeed / 2.5);
        }
      }
    },
    writerString() {
      // 先判断是否向前打印
      if (this.forwadFlag) {
        // 向前打印
        if (this.currentTextCount <= this.text.length) {
          this.outputText = this.text.substring(0, this.currentTextCount);
          this.currentTextCount++;
        } else {
          // 打印完判断是否需要向后打印
          if (this.isBack) {
            this.forwadFlag = false;
          } else {
            // 不需要取消定时器
            clearInterval(this.timer);
          }
        }
      } else {
        // 向后打印
        if (this.isBack) {
          clearInterval(this.timer);
          // 向后打印2倍速度
          this.timer = setInterval(this.backWriterString, this.wspeed / 2.5);
        }
      }
    },
    backWriterString() {
      if (this.currentTextCount >= 0) {
        this.outputText = this.text.substring(0, this.currentTextCount);
        this.currentTextCount--;
      } else {
        // 转回去从第一个开始打印
        this.currentTextCount = 1;
        clearInterval(this.timer);
        // 打印完转为向前打印
        this.forwadFlag = true;
        this.timer = setInterval(this.writerString, this.wspeed);
      }
    },
    backWriterArray() {
      if (this.currentTextCount >= 0) {
        this.outputText = this.texts[this.currentTextArrayIndex].substring(
          0,
          this.currentTextCount
        );
        this.currentTextCount--;
      } else {
        // 当前字符串打印完
        // 指针自增打印下一个
        this.currentTextArrayIndex++;
        this.forwadFlag = true;
        this.currentTextCount = 1;
        // 取消当前timer
        clearInterval(this.timer);
        this.timer = setInterval(this.writerArray, this.wspeed);
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
