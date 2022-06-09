---
title: Vue中添加登录验证码功能
date: 2022-06-09 13:57:38
id: 0603
tags: Vue
categories: Vue
---

#### Vue 中实现验证码

##### 前言

- 本来想放个示例图片看看效果的，结果发现自己不知道怎么在.md 上放图片，弄了好几次索性就放弃了。。。直接看代码吧！

##### 验证码组件部分

- `template` 部分

  ```vue
  <template>
    <div class="s-canvas">
      <canvas id="s-canvas" :width="contentWidth" :height="contentHeight"></canvas>
    </div>
  </template>
  ```

- `script` 部分

  ```vue
  <script>
  export default {
    name: 'SIdentify',
    props: {
      fresh: {
        type: Boolean,
        default: true,
      },
      fontSizeMin: {
        type: Number,
        default: 20,
      },
      fontSizeMax: {
        type: Number,
        default: 36,
      },
      backgroundColorMin: {
        type: Number,
        default: 180,
      },
      backgroundColorMax: {
        type: Number,
        default: 240,
      },
      colorMin: {
        type: Number,
        default: 50,
      },
      colorMax: {
        type: Number,
        default: 160,
      },
      lineColorMin: {
        type: Number,
        default: 40,
      },
      lineColorMax: {
        type: Number,
        default: 180,
      },
      dotColorMin: {
        type: Number,
        default: 0,
      },
      dotColorMax: {
        type: Number,
        default: 255,
      },
      contentWidth: {
        type: Number,
        default: 112,
      },
      contentHeight: {
        type: Number,
        default: 38,
      },
    },
    data() {
      return {
        identifyCodes: '1234567890abcdefghijklmnopqrstuvwxyz',
        identifyCode: '',
      };
    },
    methods: {
      // 生成一个随机数
      randomNum(min, max) {
        return Math.floor(Math.random() * (max - min) + min);
      },
      // 生成一个随机的颜色
      randomColor(min, max) {
        let r = this.randomNum(min, max);
        let g = this.randomNum(min, max);
        let b = this.randomNum(min, max);
        return 'rgb(' + r + ',' + g + ',' + b + ')';
      },
      drawPic() {
        let canvas = document.getElementById('s-canvas');
        let ctx = canvas.getContext('2d');
        ctx.textBaseline = 'bottom';
        // 绘制背景
        ctx.fillStyle = this.randomColor(this.backgroundColorMin, this.backgroundColorMax);
        ctx.fillRect(0, 0, this.contentWidth, this.contentHeight);
        // 绘制文字
        for (let i = 0; i < this.identifyCode.length; i++) {
          this.drawText(ctx, this.identifyCode[i], i);
        }
        //    this.drawLine(ctx)
        this.drawDot(ctx);
      },
      drawText(ctx, txt, i) {
        ctx.fillStyle = this.randomColor(this.colorMin, this.colorMax);
        ctx.font = this.randomNum(this.fontSizeMin, this.fontSizeMax) + 'px SimHei';
        let x = (i + 1) * (this.contentWidth / (this.identifyCode.length + 1));
        let y = this.randomNum(this.fontSizeMax, this.contentHeight - 5);
        var deg = this.randomNum(-10, 10);
        // 修改坐标原点和旋转角度
        ctx.translate(x, y);
        ctx.rotate((deg * Math.PI) / 180);
        ctx.fillText(txt, 0, 0);
        // 恢复坐标原点和旋转角度
        ctx.rotate((-deg * Math.PI) / 180);
        ctx.translate(-x, -y);
      },
      drawLine(ctx) {
        // 绘制干扰线
        for (let i = 0; i < 3; i++) {
          ctx.strokeStyle = this.randomColor(this.lineColorMin, this.lineColorMax);
          ctx.beginPath();
          ctx.moveTo(this.randomNum(0, this.contentWidth), this.randomNum(0, this.contentHeight));
          ctx.lineTo(this.randomNum(0, this.contentWidth), this.randomNum(0, this.contentHeight));
          ctx.stroke();
        }
      },
      drawDot(ctx) {
        // 绘制干扰点
        for (let i = 0; i < 30; i++) {
          ctx.fillStyle = this.randomColor(0, 255);
          ctx.beginPath();
          ctx.arc(this.randomNum(0, this.contentWidth), this.randomNum(0, this.contentHeight), 1, 0, 2 * Math.PI);
          ctx.fill();
        }
      },
      // 生成四位随机验证码
      makeCode(o, l) {
        this.identifyCode = '';
        for (let i = 0; i < l; i++) {
          this.identifyCode += this.identifyCodes[this.randomNum(0, this.identifyCodes.length)];
        }

        //绘制图片
        this.drawPic();

        //传值给父组件
        this.$emit('makedCode', this.identifyCode);
      },
    },
    watch: {
      fresh() {
        //监听事件
        this.makeCode(this.identifyCodes, 4);
      },
    },
  };
  </script>
  ```

- css 部分

  ```vue
  <style scoped>
  .s-canvas,
  #s-canvas {
    width: 126px;
    height: 44px;
    cursor: pointer;
  }
  </style>
  ```

##### 使用验证码

- 引入并注册

  ```js
  import SIdentify from '@/components/identify.vue';
  export default {
    components: {
      SIdentify,
    },
    data() {
      return {
        flag: true, //该值变化，就会触发刷新
        code: '', //刷新后的验证码
      };
    },
  };
  ```

- `template` 中使用

  ```vue
  <span class="imgValidCode" @click="refreshCode">
     <s-identify :fresh="flag" @makedCode="getMakedCode"></s-identify>
  </span>
  ```

- `methods`方法

  ```js
  // 切换验证码
  refreshCode() {
  	this.flag = !this.flag;
  },
  getMakedCode(code){
  	this.code = code;
  }
  ```

- 通过以上步骤一个简单的验证码就 OK 了！！！
