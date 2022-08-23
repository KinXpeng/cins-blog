---
title: 基于ElementUI 将常用表单组件input、select等进行封装，提升表单页面效率
date: 2022-08-23 16:55:35
id: 0802
tags: ['Vue', 'Element UI']
categories: Vue
---

> 前言

对于大多的后台管理系统来说，避免不了有很多表单的存在，写着写着就会有一大堆的冗余代码，例如 `select`、`input`、`date` 等这几种常见的表单项，虽然没太大的难度，但写起来会有些烦人，代码量也不可避免形成一大堆；因此，我也抽取了点时间将常用的一些表单组件进行了封装，可根据不同的 `type` 显示不同的组件，预留出了常用的配置，也相应的提升了开发的效率以及表单的可配置性。

### 项目环境

- vue：2.6.11
- element-ui：2.15.1

### 封装过程

- 新建一个 `searchCondition.vue` 的文件，并初始化模版。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8795fd98e0f64585b21bfe2f14ae50b9~tplv-k3u1fbpfcp-watermark.image?)

- 在使用的页面中引入声明并使用。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba48f28e16ef4d6cb70973f7c8f0b5e9~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f9a4afdde8af4a94a10cffe74fdc9a01~tplv-k3u1fbpfcp-watermark.image?)

- 下面开始来封装组件

  - 从基本的输入框开始，需要 `v-model` 将数据进行双向绑定，用 `v-if` 控制显示状态，以及绑定其它的配置。

  ```html
  <div>
    <!-- 输入框 -->
    <el-input v-model="model" v-if="show(['input'])" v-bind="parseConfig"></el-input>
  </div>
  ```

  - `v-model` 的绑定实现，通过 `get` 和 `set` 来实现双向绑定。

  ```js
  model: {
    prop: "value",
    event: "input",
  },
  props:{
      /**
     * @description 双向绑定的值
     * @example
     * <SearchCondition v-model="value" />
     */
    value: {
      type: [String, Number, Array, Object, Date],
    },
  },
  computed: {
    // 主要利用set来emit input 进行双向绑定 需要组件有input事件
    model: {
      get() {
        return this.value || "";
      },
      set(v) {
        this.$emit("input", v);
      },
    },
  ```

  - `props` 中约束值类型

  ```js
   props:{
      /**
     * @description 展示的类型, 必须和组件同名
     * @example
     * 例如你需要el-input
     * 则type 为input
     * <SearchCondition type='input' />
     */
    type: {
      type: String,
      default: () => "input",
    },

    /**
     * @description 双向绑定的值
     * @example
     * <SearchCondition v-model="value" />
     */
    value: {
      type: [String, Number, Array, Object, Date],
    },
   }
  ```

  - `computed` 中增加其它配置

  ```js
  computed: {
    // 主要利用set来emit input 进行双向绑定 需要组件有input事件
    model: {
      get() {
        return this.value || "";
      },
      set(v) {
        this.$emit("input", v);
      },
    },

    // 其它配置，详见config
    parseConfig() {
      return this.config[this.type] ? this.config[this.type] : this.config;
    },
  },
  ```

  - 在 `methods` 中新增 show 方法，控制组件显隐。

  ```js
  methods: {
    // 返回 true/false
    show(typeArr) {
      return typeArr.includes(this.type);
    },
  },
  ```

- 关于 `input` 输入框的组件基本封装完成。
- 其它的常用组件的封装就不一一罗列了，完整版的代码如下。

  - `template` 部分

  ```html
  <template>
    <div>
      <!-- 输入框 -->
      <el-input v-model="model" v-if="show(['input'])" v-bind="parseConfig"></el-input>

      <!-- select选项 -->
      <el-select v-model="model" v-if="show(['select', 'mul_select'])" v-bind="parseConfig" :multiple="type == 'mul_select' ? true : false" filterable>
        <el-option v-for="(item, index) in options" :key="index" :label="item[option.name]" :value="item[option.code]"></el-option>
      </el-select>

      <!-- 时间选择器 -->
      <el-time-picker v-model="model" v-if="show(['time'])" v-bind="parseConfig" value-format="HH:mm:ss"> </el-time-picker>

      <!-- 日期/日期时间 -->
      <el-date-picker v-model="model" :type="type" v-if="show(['date', 'datetime'])" :value-format="type == 'datetime' ? 'yyyy-MM-dd HH:mm:ss' : 'yyyy-MM-dd'" v-bind="parseConfig"> </el-date-picker>
    </div>
  </template>
  ```

  - `script` 部分

  ```js
  export default {
    model: {
      prop: 'value',
      event: 'input',
    },
    props: {
      /**
       * @description 展示的类型, 必须和组件同名
       * @example
       * 例如你需要el-input
       * 则type 为input
       * <SearchCondition type='input' />
       */
      type: {
        type: String,
        default: () => 'input',
      },
      /**
       * @description 双向绑定的值
       * @example
       * <SearchCondition v-model="value" />
       */
      value: {
        type: [String, Number, Array, Object, Date],
      },

      /**
       * @description 组件配置，配置同element
       * @example
       * <SearchCondition v-model="value" :config="{}" />
       * <SearchCondition v-model="value" :config="{ input: {} }" />
       */
      config: {
        type: Object,
        default: () => {
          return {
            size: 'mini',
            clearable: true,
          };
        },
      },

      /**
       * @description select组件配置
       * @example
       * <SearchCondition v-model="value" :option="{code:'code',name:'name'}" />
       */
      option: {
        type: Object,
        default: () => {
          return {
            code: 'code',
            name: 'value',
          };
        },
      },

      /**
       * @description select组件配置
       * @example
       * <SearchCondition v-model="value" :options="[
       *  {code:'code',name:'name'},
       * ]" />
       */
      options: {
        type: Array,
        default: () => [],
      },
    },
    computed: {
      // 主要利用set来emit input 进行双向绑定 需要组件有input事件
      model: {
        get() {
          return this.value || '';
        },
        set(v) {
          this.$emit('input', v);
        },
      },

      // 其它配置，详见config
      parseConfig() {
        return this.config[this.type] ? this.config[this.type] : this.config;
      },
    },
    methods: {
      show(typeArr) {
        return typeArr.includes(this.type);
      },
    },
  };
  ```

  ### 使用方法

  - `input` 输入框

  ```html
  <SearchCondition v-model="inputValue" type="input" />
  ```

  - `select` 下拉框，关于下拉框配置，code，name 可配置为下拉选项显示的字段。

  ```html
  <SearchCondition v-model="selectValue" type="select" :options="options" :option="{ code: 'code', name: 'name' }" />
  ```

  - `mul_select` 多选下拉框

  ```html
  <SearchCondition v-model="mulSelectValue" type="mul_select" :options="options" :option="{ code: 'code', name: 'name' }" />
  ```

  - `time` 时间选择

  ```html
  <SearchCondition v-model="timeValue" type="time" />
  ```

  - `date` 日期选择

  ```html
  <<SearchCondition v-model="dateValue" type="date" />
  ```

  - `datetime` 日期时间选择

  ```html
  <SearchCondition v-model="dateTimeValue" type="datetime" />
  ```

  #### 其它配置可如下配置：

  ```html
  <!-- example -->
  <SearchCondition v-model="inputValue" type="input" :config="{ placeholder: '请输入名称', size: 'mini' }" />
  ```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/611bff8fe3084ab2ad09d7eba344e2e7~tplv-k3u1fbpfcp-watermark.image?)

### 项目地址及预览

- [项目地址](https://github.com/KinXpeng/vue2-demo)
- [预览](https://kinxpeng.github.io/vue2-demo/)
- 常用的组件效果如下：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a143d19c0b674381bc273789d84d13f8~tplv-k3u1fbpfcp-watermark.image?)

_觉得以上内容有帮助的话点个赞吧！_
