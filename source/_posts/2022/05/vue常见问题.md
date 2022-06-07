---
title: Vue 常见问题
date: 2022-05-28 19:37:56
id: 0502
tags: Vue
categories: Vue
---

#### 路由部分

##### 1.hash 路由

创建 vue 项目时不需要作任何配置

##### 2.route 路由（history 模式）

需要后端专门设置 ngnix，vue 中需要单独配置 `mode:'history'`

###### 导入组件时

路径的写法可写为相对路径或者绝对路径

1）相对路径

`import home from "../views/home.vue"`

2）绝对路径（@相对于 src 的目录）

`import home from "@/views/home.vue"`

##### 动态路由配置时可带上参数

`path:"/home:id"`

`this.$router.push('/home',query:{info:123})`

##### 相对路由参数的变化做出响应的话，可以用 watch 监听路由

```js
watch:{
  "$route":function(to,from,next){
    // 对路由变化做出响应
  }
}
```

##### 或者使用导航守卫`beforeRouteUpdate`:

```js
beforeRouteUpdate(to,from,next){
  // 对路由变化做出响应
}
```

##### 路由跳转打开新页面

```js
let link = this.$router.resolve({
  path: '/standardProUnit', //要跳转的路由
  query: {
    id: 1,
  },
});
window.open(link.href, '_blank'); //打开新窗口
```

##### 路由元信息

```js
routes: [
  {
    path: '/foo',
    component: Foo,
    meta: {
      isLogin: true,
      requireAuth: true,
    },
  },
];
// 可通过this.$route访问元信息
```

#### 父子组件传值

##### 1.父传子

```vue
// 父组件
<com-today :todayData="navList"></com-today>
data(){ return { navList:[1,2,3] } } // 子组件 props:['todayData'] // 可用this.todayData来调用，template里面{{ todayData }}直接使用
```

##### 2.子传父

```vue
// 子组件
<p @click="sendMsg"> 传值</p>

sendMsg(){ this.$emit('receiveMsg',this.msg) },
```

```vue
// 父组件
<child @receiveMsg="receive"></child>

receive(data){ console.log(data); },
```

##### 3.路由传值

###### 区别

params：不在路由地址中显示，相当于 ajax 请求的 post 方式

query：传值会在路由地址中展示出来

```js
// params方式（用name跳转页面）
// 不配置参数名时，页面刷新后参数会消失
1.router/index.js中的配置
{
  path:'/home/:id', //多个参数时 path:'/home/:id&:name',
  name:'home',
  component:()=>import('@/views/home.vue')
},
 2.页面中路由的传参
this.$router.push({
  name:"home",
  params:{id:this.id}  // 多个参数时 params:{id:this.id,name:this.name}
})


// query方式（用path跳转页面）
this.$router.push({
  path:'/home',
  query:{id:this.id}
})
```

#### Vue 中使用 rem 布局方式（2 种）

- 在 public 下的 index.html 页面中添加如下 js

  ```js
  fnResize();
  window.onresize = function () {
    fnResize();
  };
  function fnResize() {
    var deviceWidth = document.documentElement.clientWidth || window.innerWidth;
    if (deviceWidth >= 750) {
      deviceWidth = 750;
    }
    if (deviceWidth <= 320) {
      deviceWidth = 320;
    }
    document.documentElement.style.fontSize = deviceWidth / 7.5 + 'px';
  }
  ```

  - 然后在写 css 就可以将 px 单位换成 rem
  - 设置的比例是 100px=1rem

- 或者安装插件引入

  - 首先安装 amfe-flexible 插件，在 main.js 里引入

    ```js
    // 1.安装插件
    npm i amfe-flexible
    // 2.在main.js中引入
    import 'amfe-flexible'
    ```

  - 再安装 postcss-px2rem 插件

    ```js
    npm i postcss-px2rem
    ```

  - 在 package.json 中配置

    ```json
    "postcss": {
       "plugins": {
         "autoprefixer": {},
         "postcss-px2rem": {
           "remUnit": 37.5
         }
       }
     }
    ```

  - 在.vue 文件里。样式直接写 px 单位就可以了。在浏览器查看时会自动转换为 rem 单位。如果字体还想用 px。那就这样将 px 大写。就不会编译为 rem 单位了。样式就可以实现 px。

#### eslint 的错误提示

```js
// error  'clock_start' is assigned a value but never used
// 在.eslintrc.js中的rules里加入下面这行
rules: {
  "no-unused-vars": 'off'
}
```

#### 滚动条不显示

```css
/**.class为类名**/
.class::-webkit-scrollbar {
  display: none; //去掉滚动条，不能滚动了；
  width: 0; //可以滚动，且滚动条不显示；
}
```

#### vue 中路由跳转怎么新开页面

```js
let wrapUrl = this.$router.resolve({ path: '/wrap' });
window.open(wrapUrl.href, '_blank');
```

#### 文件下载接口的处理

```js
this.$axios({
  method: 'post',
  url: '/api/hlcloud-wms-app/inboundOrderStat/export', // 接口地址
  responseType: 'blob', // 接收格式设置
  data: dataObj, // 参数
}).then((res) => {
  // console.log(res)
  // 如果后台传的请求头中有文件名
  // console.log(encodeURI(res.headers['content-disposition']));
  let excelName = res.headers['content-disposition'].split('=')[1];
  // console.log(excelName)
  const content = res.data;
  const blob = new Blob([content]);
  const fileName = decodeURIComponent(excelName); //文件名解码（需要后端先进行编码,以防中文乱码）
  if ('download' in document.createElement('a')) {
    // 非IE下载
    const elink = document.createElement('a');
    elink.download = fileName;
    elink.style.display = 'none';
    elink.href = URL.createObjectURL(blob);
    document.body.appendChild(elink);
    elink.click();
    URL.revokeObjectURL(elink.href); // 释放URL 对象
    document.body.removeChild(elink);
  } else {
    // IE10+下载
    navigator.msSaveBlob(blob, fileName);
  }
});
```

#### elementUI 中的使用注意事项

- 使用 required="true"给 table 中的 label 加\*号

```html
<el-form-item :required="true" label="活动类型">
  <el-input v-model="input"></el-input>
</el-form-item>
```

- 获取 date 组件的日期格式

```html
<el-date-picker v-model="getdate" type="daterange" range-separator="至" value-format="yyyy-MM-dd HH:mm:ss" start-placeholder="开始日期" end-placeholder="结束日期"> </el-date-picker>
```

- table 中列文字过多悬浮显示并隐藏

  ```js
  :show-overflow-tooltip="true"
  ```

- table 的 selection 表头多选框变为文字标题

  ```css
  /deep/ .el-table__header-wrapper .el-checkbox__input::after {
    content: '全选';
    position: absolute;
    margin-left: 5px;
  }
  ```

- 解决 table 中翻页序号连续递增的问题

  ```html
  <!--page为当前页码，rows为每页条数-->
  <el-table-column type="index" :index="(index)=>{return (index+1) + (page-1)*rows}"></el-table-column>
  ```

- el-input 输入框自动聚焦的方法

  ```js
  this.$nextTick(() => {
    this.$refs.focusInput.focus();
  });
  ```

- tree-node 中拼接每一级的数据

  ```js
  handleNodeClick(data,node) {
    this.placeFullName = [];
    let num = node.level;  // 获取当前点击的层级
    for(let i=0;i<num;i++){
      if(i>0){
        node = node.parent; // 赋值节点
        this.placeFullName.unshift(node.data.placeName);
      }else{
        this.placeFullName.unshift(data.placeName);
      }
    }
    this.detailForm.AssetItem.placeName = this.placeFullName.join('');
  ```

- calendar 组件只生效当前月份点击事件

  ```css
  /* 日历中禁止非当前月点击事件 */
  ::v-deep .el-calendar-table:not(.is-range) td.next,
  ::v-deep .el-calendar-table:not(.is-range) td.prev {
    pointer-events: none;
  }
  ```

- table 加了固定头，内容可滚动，当滚到 table 底边时，点击分页后还在底边

  ```js
  // 先给table设置了ref="gridTable"
  // 点击分页的时候加上这句即可
  this.$refs.gridTable.bodyWrapper.scrollTop = 0;
  ```

-

#### Vue 中定义 cookie 的方法

```js
// 定义设置cookie方法
Vue.prototype.setCookie = function (name, value, day) {
  if (day !== 0) {
    //当设置的时间等于0时，不设置expires属性，cookie在浏览器关闭后删除
    var curDate = new Date();
    var curTamp = curDate.getTime();
    var curWeeHours = new Date(curDate.toLocaleDateString()).getTime() - 1;
    var passedTamp = curTamp - curWeeHours;
    var leftTamp = 24 * 60 * 60 * 1000 - passedTamp;
    var leftTime = new Date();
    leftTime.setTime(leftTamp + curTamp);
    document.cookie = name + '=' + escape(value) + ';expires=' + leftTime.toGMTString();
  } else {
    document.cookie = name + '=' + escape(value);
  }
};
// 设置cookie
this.setCookie('AuthenticationToken', 'a9cd00f0b00b4371822c8f6069aba8d0', 0);
```

#### 开发环境和生产环境的配置

- /src 目录下新建`.env.development`文件

  ```js
  NODE_ENV = 'development';
  VUE_APP_FLAG = 'dev';
  ```

- /src 目录下新建`.env.production`文件

  ```js
  NODE_ENV = 'production';
  VUE_APP_FLAG = 'pro';
  ```

- main.js 中

  ```js
  let url = window.location.href.split('/');
  let baseUrl = url[0] + '//' + url[2];
  // 判断环境
  if (process.env.NODE_ENV === 'production') {
    if (process.env.VUE_APP_FLAG === 'pro') {
      axios.defaults.baseURL = baseUrl;
    } else {
      axios.defaults.baseURL = baseUrl;
    }
  } else {
    axios.defaults.baseURL = '/api';
  }
  ```

- router 文件中 index.js 的配置

  ```js
  const router = new VueRouter({
    mode: 'history',
    base: '/wms/', //process.env.BASE_URL, 将默认的改为服务器下的文件路径
    routes,
  });
  ```

#### Vue 中 cli2 搭建的项目打包时图片/图标不显示

```js
//只需要在build/utils.js文件中找到vue-style-loader配置，为它新增（publicPath: '../../'）即可。
if (options.extract) {
  return ExtractTextPlugin.extract({
    use: loaders,
    fallback: 'vue-style-loader',
    publicPath: '../../', // 新增项
  });
} else {
  return ['vue-style-loader'].concat(loaders);
}
```

#### Vue 中解除双向绑定的方法

```js
// 1.对对象进行深拷贝
// 嵌套for循环赋值的问题也能用此方法解决
let obj = JSON.parse(JSON.stringify(data));
// 2.Vue中自有的方法
Object.freeze(obj);
```

#### 获取地址中的参数

```js
// 获取地址栏中的参数  name为参数的名字 函数会返回参数名所对应的参数值
getQueryString(name){
  var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)");
  var r = window.location.search.substr(1).match(reg);
  if(r!=null)return  unescape(r[2]); return null;
}
```

#### 封装 elementUI 的 treeSelect 下拉框

```vue
<!--
  author:KinXpeng
  date:2020-5-20 09:24:00
  **** 使用案例
  <tree-select
    :options="treeData"
    :defaultProps="defaultProps"
    :clearable="true"
    :accordion="true"
    @selectNode="selectNode"
  ></tree-select>
  *** 1.options 为绑定的树状数据
  *** 2.defaultProps 为配置项，label和children
  *** 3.clearable 为是否可清除选择
  *** 4.accordion 为是否开启手风琴模式，只会展开一项
  *** 5.selectNode 为选中节点的回调函数，有两个参数（data，node），data为节点的数据
  ***   有清除按钮时，点击清除会触发selectNode函数，data为空，可通过判断data来设置选中的value为空
-->
<template>
  <div class="treeSelect">
    <el-select v-model="selectValue" :clearable="clearable" @clear="clearHandle" :popper-class="popperClass" placeholder="请选择" ref="tree_select">
      <el-option :value="selectValue" :label="selectValue" style="height:auto">
        <el-tree class="selectTree" :data="options" :show-checkbox="false" ref="selectTree" highlight-current :props="defaultProps" :accordion="accordion" @node-click="handleNodeClick"> </el-tree>
      </el-option>
    </el-select>
  </div>
</template>

<script>
export default {
  name: 'treeSelect',
  props: {
    // 树结构数据
    options: {
      type: Array,
      default() {
        return [];
      },
    },
    // 配置项
    defaultProps: {
      type: Object,
      default() {
        return {};
      },
    },
    // 清除
    clearable: {
      type: Boolean,
      default() {
        return false;
      },
    },
    // 是否打开手风琴模式
    accordion: {
      type: Boolean,
      default() {
        return false;
      },
    },
  },
  data() {
    return {
      selectValue: '',
      popperClass: 'tree_select',
      isShowSelect: false,
    };
  },
  methods: {
    // 选中节点
    handleNodeClick(data, node) {
      this.selectValue = data[this.defaultProps.label];
      this.$emit('selectNode', data, node);
      this.$refs.tree_select.blur(); // 选中后隐藏下拉框
    },
    // 清除选择回调函数
    clearHandle() {
      this.$emit('selectNode');
    },
  },
  created() {},
};
</script>

<style lang="less">
.treeSelect {
  display: inline-block;
}
.tree_select {
  .el-select-dropdown__item {
    padding: 0;
    .el-tree-node__label {
      font-weight: normal;
    }
    .el-tree-node {
      padding: 5px 0;
    }
  }
  .el-select-dropdown__item.hover {
    background: #fff;
    padding: 0;
    &:hover {
      background: #fff;
    }
  }
}
.hide {
  display: none !important;
}
</style>
```

#### 封装公用请求接口返回为 Promise 对的解决方案

```js
#### bus.js文件
//  在公用的js中封装好统一的函数
// 数据字典通用接口
function dictionaryList(dictCode) {
  return new Promise((resolve,reject)=>{ // 返回Promise对象
    Vue.prototype.$http
      .post(
        "/hlcloud-ubp-app/dictDataQuery/listByExtend?dictCode="+dictCode+"&objectDataOnly=false&softProdCode=EAM"
      )
      .then(res => {
        if (res.data.code == "0") {
          return resolve(res.data.data); // 将请求的数据resolve出去
        }
      })
      .catch(err => {
        return reject(err)
      });
  })
};
// 异步处理返回的数据
async function getDictionaryList(dictCode){
  let listData = await dictionaryList(dictCode);
  return listData;
}
```

```js
#### 调用的页面
// 使用.then() 将需要的数据从Promise对象拿到
// 资源类型
this.bus.getDictionaryList('ResourceType').then(data=>{
  this.resourcetypeList = data;
});
```

#### 获取地址栏参数

```js
// 获取地址栏中的参数(传入参数名称即可)
function getQueryString(name) {
  var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)');
  var r = window.location.search.substr(1).match(reg);
  if (r != null) return unescape(r[2]);
  return null;
}
```

#### 数组去重返回新数组

```js
// a,b 为两个数组  其中a.length > b.length
arrSubtraction2(a, b) {
  if (a && b) {
    return a.filter(i => !b.includes(i))
  }
  throw new Error('error')
}
```

#### Avoided redundant navigation to current location: "/writeArticle".

```js
// this.$router.push() 时报错
// 在router中的index.js中
const originalPush = VueRouter.prototype.push;

VueRouter.prototype.push = function push(location) {
  return originalPush.call(this, location).catch((err) => err);
};
```

#### vue 中使用 scss 设置夜间模式

- 安装 scss 依赖

  ```js
  npm install node-sass --save-dev    //安装node-sass
  npm install sass-loader --save-dev  //安装sass-loader
  npm install style-loader --save-dev //安装style-loader
  ```

- 在 src/assets 新建目录 scss，以及新建 dark.scss 主题文件

  ```scss
  // scss样式
  $themes: (
    light: (
      background_color: #cccccc,
      //背景色
      text-color: rgba(0, 0, 0, 0.65),
      // 主文本色
    ),
    dark: (
      background_color: #181c27,
      //背景
      text-color: rgba(255, 255, 255, 0.65),
      // 主文本色
    ),
  );

  // 处理样式，遍历主题map
  @mixin themeify {
    @each $theme-name, $theme-map in $themes {
      //!global 把局部变量提升为全局变量
      $theme-map: $theme-map !global;
      //判断html的data-theme的属性值  #{}是sass的插值表达式
      //& sass嵌套里的父容器标识   @content是混合器插槽，像vue的slot
      [data-theme='#{$theme-name}'] & {
        @content;
      }
    }
  }
  //声明一个根据Key获取颜色的function
  @function themed($key) {
    @return map-get($theme-map, $key);
  }

  // 获取颜色
  @mixin background_color($color) {
    @include themeify {
      background: themed($color) !important;
    }
  }
  //获取字体颜色
  @mixin font_color($color) {
    @include themeify {
      color: themed($color) !important;
    }
  }
  ```

- 在 main.js 中引入 dark.scss

  ```js
  import './assets/scss/dark.scss';
  ```

- 在 dark.vue 中使用

  ```vue
  <template>
    <div id="darkMode">
      <div>
        <a class="btn" @click="modelChange">模式切换</a>
      </div>
    </div>
  </template>

  <script>
  export default {
    name: 'darkMode',
    data() {
      return {
        dark: false,
      };
    },
    methods: {
      // 使用方法
      modelChange() {
        this.dark = !this.dark;
        if (this.dark) {
          window.document.documentElement.setAttribute('data-theme', 'dark');
        } else {
          window.document.documentElement.setAttribute('data-theme', 'light');
        }
      },
    },
    mounted() {
      window.document.documentElement.setAttribute('data-theme', 'light');
    },
  };
  </script>

  <style scoped lang="scss">
  @import '@/assets/scss/common/common';
  #darkMode {
    //在此使用了背景颜色变量
    @include background_color('background_color');
    //再次使用了文字颜色变量
    @include font_color('text-color');
    width: 100vw;
    height: 100vh;
    .btn {
      width: 100px;
      height: 40px;
      margin: 0 auto;
    }
  }
  </style>
  ```

#### 设置字体小于 12px

    ```css
    span{
      font-size: 12px;
      transform: scale(.8); // 将字体缩小80%，即9.6px；如需更小，修改scale中的数值即可。
      transform-origin: 0 0; // 设置后发现文字在原来的基础上偏移了，解决基点偏移问题。
    }
    ```

### Nuxt.js

- Nuxt.js 的使用

- 安装注意事项

  - 使用`npx create-nuxt-app myapp`安装

  - 配置项信息安装官网默认选择

  - 安装 scss 报错处理

    ```bash
    1.安装scss依赖
    npm install --save-dev node-sass sass-loader
    2.在 nuxt.congig.js 文件里进行如下配置
    modules: [
    	'@nuxtjs/style-resources', // 导入模块
    ],
    3.通常这样安装后会报错，提示node_modules错误等等，换个低版本的scss安装即可
    4.npm install --save-dev sass sass-loader@10 fibers  // 可以指定版本号
    ```

- 安装配置问题

  - 跨域配置（在 nuxt.config.js 中配置）

    ```js
    axios: {
      proxy: true, // 配置代理
      prefix:'/api', // 给请求的url前面加上'/api'
    },
    proxy: {
      '/api': {
        target: 'http://127.0.0.1:3000/',
        changeOrigin: true, // 是否跨域
        pathRewrite: {
          '^/api': '/'
        }
      }
    },
    ```

  - 跨域之后在 vue 文件中如何使用 axios 请求接口

    ```js
    export default {
      asyncData(context) {
        // 使用context对象
        return context.$axios.post('/menuList').then((res) => {
          console.log(res.data.data);
        });
      },
    };
    ```

  - 修改项目启动端口，在 package.json 中配置

    ```json
    "config": {
        "nuxt": {
          "host": "0.0.0.0",
          "port": "3333"  // 所修改的端口号
        }
      }
    ```

  - 1

- 等等

- Nuxt.js 中遇到的问题

##### vue 中切换路由的时候滚动条到达顶部

```js
scrollBehavior (to, from, savedPosition) {
	return { x: 0, y: 0 }
},
```

##### 数组等量切分

```js
// 第一个参数为需要切分的数组，第二个参数为切分的大小
function splitData(list, size) {
  let res = [];
  for (let i = 0, len = list.length; i < len; i += size) {
    res.push(list.slice(i, i + size));
  }
  return res;
}
```

##### css 中的粘性布局

```css
/*此属性会自动监听scroll属性，仅限在外层盒子的滚动中生效，外层有overflow:hidden时不生效*/
position: sticky;
position: -webkit-sticky;
top: 0;
```

##### 监听页面滚动条滚动的方向

```js
// data
data(){
  return {
    scrollAction:{
      x: 'undefined',
      y: 'undefined'
    },
    scrollDirection:null,
  }
}

// methods
scrollFunc() {
  if (typeof this.scrollAction.x == 'undefined') {
    this.scrollAction.x = window.pageXOffset;
    this.scrollAction.y = window.pageYOffset;
  }
  var diffX = this.scrollAction.x - window.pageXOffset;
  var diffY = this.scrollAction.y - window.pageYOffset;
  if (diffX < 0) {
    // Scroll right
    this.scrollDirection = 'right';
  } else if (diffX > 0) {
    // Scroll left
    this.scrollDirection = 'left';
  } else if (diffY < 0) {
    // Scroll down
    this.scrollDirection = 'down';
  } else if (diffY > 0) {
    // Scroll up
    this.scrollDirection = 'up';
  } else {
    // First scroll event
  }
  this.scrollAction.x = window.pageXOffset;
  this.scrollAction.y = window.pageYOffset;
},

// mounted
mounted(){
  window.addEventListener("scroll",this.handleScroll);
},
// destroyed
destroyed() {
  document.removeEventListener('scroll', this.handleScroll);
}
```

##### js 获取当前月份的起止日期

```js
// 格式化日期
getDateStr(date) {
  let year = "";
  let month = "";
  let day = "";
  let now = date;
  year = ""+now.getFullYear();
  if((now.getMonth()+1)<10){
    month = "0"+(now.getMonth()+1);
  }else{
    month = ""+(now.getMonth()+1);
  }
  if((now.getDate())<10){
    day = "0"+(now.getDate());
  }else{
    day = ""+(now.getDate());
  }
  return year+"-"+month+"-"+day;
},
  // 获取当前月份起止日期
  // 参数0，当前月份，-1为上一个月，1为下个月
  getMonthStartAndEnd(AddMonthCount) {
    //起止日期数组
    let startStop = new Array();
    //获取当前时间
    let currentDate = new Date();
    let month=currentDate.getMonth()+AddMonthCount;
    if(month<0){
      let n = parseInt((-month)/12);
      month += n*12;
      currentDate.setFullYear(currentDate.getFullYear()-n);
    }
    currentDate = new Date(currentDate.setMonth(month));
    //获得当前月份0-11
    let currentMonth = currentDate.getMonth();
    //获得当前年份4位年
    let currentYear = currentDate.getFullYear();
    //获得上一个月的第一天
    let currentMonthFirstDay = new Date(currentYear, currentMonth,1);
    //获得上一月的最后一天
    let currentMonthLastDay = new Date(currentYear, currentMonth+1, 0);
    //添加至数组
    startStop.push(this.getDateStr(currentMonthFirstDay));
    startStop.push(this.getDateStr(currentMonthLastDay));
    //返回
    return startStop;
  },
```

#### vue.config.js 配置

- 减少打包体积，去除.map 文件

  ```js
  module.exports = {
    productionSourceMap: false, // 去掉打包后的map文件
  };
  ```

#### Element UI 中—Message 消息提示每次只弹出 1 个，不能同时出现 2 个

- 新建 `/utils/message.js`

  ```js
  //message.js
  /**重置message，防止重复点击重复弹出message消息提示 */
  import { Message } from 'element-ui';

  let messageInstance = null;

  const resetMessage = (options) => {
    if (messageInstance) {
      messageInstance.close();
    }
    Message.closeAll(); //手动关闭所有消息提示实例
    messageInstance = Message(options);
    // console.log(Message(options));
  };

  ['error', 'success', 'info', 'warning'].forEach((type) => {
    resetMessage[type] = (options) => {
      if (typeof options === 'string') {
        options = {
          message: options,
        };
      }
      options.type = type;
      return resetMessage(options);
    };
  });

  export const message = resetMessage;
  ```

- `main.js` 中引入，挂载到全局中

  ```js
  import { message } from '@/utils/message.js';

  Vue.use(ElementUI);
  Vue.prototype.$message = message; // 挂载时在use后面，以便覆盖原有的提示
  ```

- 使用方法和之前一样

  ```js
  this.$message({
    type: 'error',
    message: '提示信息',
  });
  ```
