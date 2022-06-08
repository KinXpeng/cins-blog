---
title: JS常见问题及常用方法
date: 2022-06-08 10:49:00
id: 0602
tags: JS
categories: JS
---

#### ES6 常用语法

- 解构赋值

  ```js
  const { a, b, c, d, e } = obj;
  // ES6的解构赋值虽然好用。但是要注意解构的对象不能为undefined、null。否则会报错，故要给被解构的对象一个默认值。
  const { a, b, c, d, e } = obj || {};
  ```

- 合并数据

  ```js
  // 数组合并
  const a = [1, 2, 3];
  const b = [1, 5, 6];
  const c = [...new Set([...a, ...b])]; //[1,2,3,5,6]
  // 对象合并
  const obj1 = {
    a: 1,
  };
  const obj2 = {
    b: 1,
  };
  const obj = { ...obj1, ...obj2 }; //{a:1,b:1}
  ```

- includes

  ```js
  const condition = [1, 2, 3, 4];
  if (condition.includes(type)) {
    //...
  }
  console.log([1, 2, 3].includes(2) === true);
  console.log([1, 2, 3].includes(4) === false);
  console.log([1, 2, NaN].includes(NaN) === true);
  ```

- indexOf

  ```js
  // 查找一个元素是否在数组中，并返回索引
  [1, 2, 3].indexOf(1) >=
    (0)[
      // 无法查找NaN
      (1, 2, NaN)
    ].indexOf(NaN) >=
    0;
  ```

- filter 和 find

  ```js
  // find方法中找到符合条件的项，就不会继续遍历数组。
  const a = [1, 2, 3, 4, 5];
  const result = a.find((item) => {
    return item === 3;
  });
  ```

- 关于输入框非空判断

  ```js
  // 之前
  if (value !== null && value !== undefined && value !== '') {
    //...
  }
  //ES6
  if (value ?? '' !== '') {
    //...
  }
  ```

- Exponentiation operator 幂运算

  ```js
  let squared = 2 ** 2;
  // same as: 2 * 2

  let cubed = 2 ** 3;
  // same as: 2 * 2 * 2

  let a = 2;
  a **= 2;
  // same as: a = a * a;

  let b = 3;
  b **= 3;
  // same as: b = b * b * b;
  ```

- Object.values / Object.entries 对象值、对象对

  ```js
  // Object.values 方法返回一个给定对象自身的所有可枚举属性值的数组，值的顺序与使用for...in循环的顺序相同（区别在于 for-in 循环枚举原型链中的属性）
  let point = { x: 12, y: 6 };
  Object.values(point);

  // 结果: [12, 6]

  // Object.entries 方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环还会枚举原型链中的属性）
  let point = { x: 12, y: 6 };
  Object.entries(point);

  // 结果: [["x", 12], ["y", 6]]

  // 字符串也可
  Object.entries('hello world');

  // 结果: [["0","h"],["1","e"],["2","l"],["3","l"],["4","o"],["5"," "],["6","w"],["7","o"],["8","r"],["9","l"],["10","d"]]
  ```

- 用同步的方法来书写异步的 `Promise`

  ```js
  async function a() {
    return 'a';
  }
  function b() {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve('b');
      }, 3000);
    });
  }
  async function c() {
    return 'c';
  }
  async function d() {
    let a = await a();
    console.log(a);
    let b = await b();
    console.log(b);
    let c = await c();
    console.log(c);
  }
  console.log(d());

  // 结果：
  // Promise { <pending> } `async` 标记的方法返回了一个 `Promise` 对象
  // a
  // b  等待三秒之后继续输出后续内容
  // c
  ```

- 。。。

#### 常用 JS 代码

- 随机获得一个布尔值

  ```js
  const randomBoolean = () => Math.random() >= 0.5;
  console.log(randomBoolean());
  ```

- 判断今天是否为工作日

  ```js
  const isWeekday = (date) => date.getDay() % 6 !== 0;
  console.log(isWeekday(new Date(2021, 0, 11)));
  // Result: true (Monday)
  console.log(isWeekday(new Date(2021, 0, 10)));
  // Result: false (Sunday)
  ```

- 反转字符串

  ```js
  const reverse = str => str.split(  ).reverse().join(  );
  reverse( hello world );
  // Result:  dlrow olleh
  ```

- 检查当前 Tab 页是否在前台

  ```js
  const isBrowserTabInView = () => document.hidden;
  isBrowserTabInView();
  // Result: returns true or false depending on if tab is in view / focus
  ```

- 从日期中获取时间

  - 通过使用 `toTimeString()` 方法，在正确的位置对字符串进行切片，我们可以从提供的日期中获取时间或者当前时间。

  ```js
  const timeFromDate = (date) => date.toTimeString().slice(0, 8);
  console.log(timeFromDate(new Date(2021, 0, 10, 17, 30, 0)));
  // Result: "17:30:00"
  console.log(timeFromDate(new Date()));
  // Result: will log the current time
  ```

- 检查当前用户是否为苹果设备

  ```js
  const isAppleDevice = /Mac|iPod|iPhone|iPad/.test(navigator.platform);
  console.log(isAppleDevice);
  // Result: will return true if user is on an Apple device
  ```

- 滚动到页面底部

  ```js
  export const scrollToBottom = () => {
    window.scrollTo(0, document.documentElement.clientHeight);
  };
  ```

- 滚动到页面顶部

  ```js
  const goToTop = () => window.scrollTo(0, 0);
  goToTop();
  // Result: will scroll the browser to the top of the page
  ```

- 获取所有参数平均值

  ```js
  const average = (...args) => args.reduce((a, b) => a + b) / args.length;
  average(1, 2, 3, 4);
  // Result: 2.5
  ```

- 判断是移动还是 PC 设备

  ```js
  export const isMobile = () => {
    if (navigator.userAgent.match(/(iPhone|iPod|Android|ios|iOS|iPad|Backerry|WebOS|Symbian|Windows Phone|Phone)/i)) {
      return 'mobile';
    }
    return 'desktop';
  };
  ```

- 浏览器型号和版本

  ```js
  export const getExplorerInfo = () => {
    let t = navigator.userAgent.toLowerCase();
    return 0 <= t.indexOf('msie')
      ? {
          //ie < 11
          type: 'IE',
          version: Number(t.match(/msie ([\d]+)/)[1]),
        }
      : !!t.match(/trident\/.+?rv:(([\d.]+))/)
      ? {
          // ie 11
          type: 'IE',
          version: 11,
        }
      : 0 <= t.indexOf('edge')
      ? {
          type: 'Edge',
          version: Number(t.match(/edge\/([\d]+)/)[1]),
        }
      : 0 <= t.indexOf('firefox')
      ? {
          type: 'Firefox',
          version: Number(t.match(/firefox\/([\d]+)/)[1]),
        }
      : 0 <= t.indexOf('chrome')
      ? {
          type: 'Chrome',
          version: Number(t.match(/chrome\/([\d]+)/)[1]),
        }
      : 0 <= t.indexOf('opera')
      ? {
          type: 'Opera',
          version: Number(t.match(/opera.([\d]+)/)[1]),
        }
      : 0 <= t.indexOf('Safari')
      ? {
          type: 'Safari',
          version: Number(t.match(/version\/([\d]+)/)[1]),
        }
      : {
          type: t,
          version: -1,
        };
  };
  ```

- 打开浏览器全屏

  ```js
  export const toFullScreen = () => {
    let element = document.body;
    if (element.requestFullscreen) {
      element.requestFullscreen();
    } else if (element.mozRequestFullScreen) {
      element.mozRequestFullScreen();
    } else if (element.msRequestFullscreen) {
      element.msRequestFullscreen();
    } else if (element.webkitRequestFullscreen) {
      element.webkitRequestFullScreen();
    }
  };
  ```

- 防抖函数

  ```js
  export const debounce = (fn, wait) => {
    let timer = null;

    return function () {
      let context = this,
        args = arguments;

      if (timer) {
        clearTimeout(timer);
        timer = null;
      }

      timer = setTimeout(() => {
        fn.apply(context, args);
      }, wait);
    };
  };
  ```

- 节流函数

  ```js
  export const throttle = (fn, delay) => {
    let curTime = Date.now();

    return function () {
      let context = this,
        args = arguments,
        nowTime = Date.now();

      if (nowTime - curTime >= delay) {
        curTime = Date.now();
        return fn.apply(context, args);
      }
    };
  };
  ```

- 数据类型判断

  ```js
  export const getType = (value) => {
    if (value === null) {
      return value + '';
    }
    // 判断数据是引用类型的情况
    if (typeof value === 'object') {
      let valueClass = Object.prototype.toString.call(value),
        type = valueClass.split(' ')[1].split('');
      type.pop();
      return type.join('').toLowerCase();
    } else {
      // 判断数据是基本数据类型的情况和函数的情况
      return typeof value;
    }
  };
  ```

- 笛卡尔积算法

  ```js
  calcDescartes(array) {
    if (array.length < 2) return array[0] || [];
    return array.reduce((total, currentValue) => {
      let res = [];
      total.forEach((t) => {
        currentValue.forEach((cv) => {
          if (t instanceof Array) {
            // 或者使用 Array.isArray(t)
            res.push([...t, cv]);
          } else {
            res.push([t, cv]);
          }
        });
      });
      return res;
    });
  }
  ```

- 上传图片（base64 编码）

  ```js
  // 选择上传图片
  const handleSelectFile = () => {
    const input = document.createElement('input'); // 创建input
    input.type = 'file';
    input.accept = 'image/*'; // 上传限制类型
    input.onchange = (e) => {
      const file = e.target.files[0];
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = (e) => {
        state.fileList.push({
          name: file.name,
          url: e.target.result,
        });
      };
    };
    input.click();
  };
  ```

- 打开原生应用

  ```js
  <a href="weixin://">打开微信</a>
  <a href="alipays://">打开支付宝</a>
  <a href="alipays://platformapi/startapp?saId=10000007">打开支付宝的扫一扫功能</a>
  <a href="alipays://platformapi/startapp?appId=60000002">打开支付宝的蚂蚁森林</a>
  /*
       行为(应用的某个功能/页面)
              |
  scheme://[path][?query]
     |               |
  应用标识       功能需要的参数
  */
  ```

- 禁止浏览器的默认行为

  ```css
  // 禁止长按图片保存
  img {
    -webkit-touch-callout: none;
    pointer-events: none; // 像微信浏览器还是无法禁止，加上这行样式即可
  }

  // 禁止长按选择文字
  div {
    -webkit-user-select: none;
  }

  // 禁止长按呼出菜单
  div {
    -webkit-touch-callout: none;
  }
  ```

- 滑动不顺畅，粘手

  ```css
  div {
    -webkit-overflow-scrolling: touch;
  }
  ```

- 屏幕旋转为横屏时，字体大小会变

  ```css
  * {
    -webkit-text-size-adjust: 100%;
  }
  ```

- 未完待续。。。
