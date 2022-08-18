---
title: 在vue中封装多功能select，支持分页查询，远程搜索以及数据反显！
date: 2022-08-18 10:46:14
id: 0801
tags: ['Vue', 'Element UI']
categories: Vue
---

> 前言

由于业务需求，现有的 Element UI 的下拉框满足不了所需要的功能，这次的封装也是基于之前改版的`el-select` ，好久之前也是对这个下拉做了些调整，但是经过一段时间的使用，发现使用方法较为麻烦，也会有一些 bug 解决不了，兴起之下索性花些时间做的完整一点。

> 项目版本

- Vue：2.6.11
- Element UI：2.15.1

> 准备工作

- 基于了 Element UI 的远程搜索。
- 因为下拉框需要分页查询，需要加入自定义指令，下拉到底部时触发下一页的查询。

> 实现 loadmore 触底加载指令

- src 目录下创建 `directives.js`，并在`main.js`中全局引入。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/702bf04edd054e6f991cd8877a37f9ee~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b3e7cb27df6e41a0b70efade51f03362~tplv-k3u1fbpfcp-watermark.image?)

- 写入相关指令的内容，大致就是下拉内容触底的时候触发。

  ````js
   // directives.js

   import Vue from 'vue';

   Vue.directive('loadmore', {
     bind(el, binding) {
       // 获取element-ui定义好的scroll盒子
       const SELECTWRAP_DOM = el.querySelector('.el-select-dropdown .el-select-dropdown__wrap');
       SELECTWRAP_DOM.addEventListener('scroll', function() {
         const CONDITION = this.scrollHeight - this.scrollTop - 5 <= this.clientHeight;
         if (CONDITION) {
           binding.value();
         }
       });
     },
   });

    ```
  >  封装组件

  ````

- 在 `components` 中新建 remoteSelect.vue 文件
- template 部分，参考 `el-select` 的部分代码，实现远程搜索
  ```js
  <template>
      <el-select
        v-model="selectValue"
        placeholder="请选择"
        size="mini"
        filterable
        clearable
        remote
        height="auto"
        v-loadmore="loadMore"
        :loading="loading"
        :remote-method="remoteMethod"
        @change="handleValueChange"
        @visible-change="handleVisible"
        @clear="handleClearSelected"
      >
        <el-option
          v-for="item in optionsSelect"
          :key="item[option.code]"
          :label="item[option.name]"
          :value="item[option.code]"
        >
        </el-option>
      </el-select>
    </template>
  ```
- script 部分，props 接收了父组件传过来的数据
  ```js
  <script>
   export default {
     name: "remoteSelect",
     data() {
       return {
         selectValue: "",
         loading: false,
         optionsSelect: [], // 搜索下拉框的数据
         opEntityToName: "", // 下拉框模糊查询
         srcollNum: 1, // 下拉框默认查询页数
         selTotal: 0, // 下拉框总条数
       };
     },
     props: {
       value: {
         type: Object,
         default: () => {
           return {};
         },
       },
       // 远程分页搜索接口
       api: {
         type: String,
         default: "",
         required: true,
       },
       // 模糊查询字段名
       optionName: {
         type: String,
         default: "optionName",
       },
       // 每页条数
       rows: {
         type: Number,
         default: 10,
       },
       // 选项code&name
       option: {
         type: Object,
         required: true,
         default: () => {
           return {
             code: "optionCode",
             name: "optionName",
           };
         },
       },
     },
     methods: {
       // 下拉框加载更多
       loadMore() {
         this.srcollNum++;
         if (this.srcollNum < this.selTotal / this.rows + 1) {
           this.handleLikeSearch(this.opEntityToName);
         }
       },
       // 下拉框远程搜索
       remoteMethod(query) {
         this.srcollNum = 1;
         this.opEntityToName = query;
         if (query !== "") {
           this.loading = true;
           setTimeout(() => {
             this.loading = false;
             this.handleLikeSearch(query);
           }, 200);
         } else {
           this.handleLikeSearch();
         }
       },
       // 下拉框数据
       handleLikeSearch(query) {
         this.$axios
           .post(this.api, {
             page: this.srcollNum,
             [this.optionName]: query,
             rows: this.rows,
           })
           .then((res) => {
             // console.log(res);
             if (this.srcollNum == 1) {
               this.optionsSelect = res.data;
             } else {
               this.optionsSelect.push(...res.data);
             }
             if (Object.keys(this.value).length && !query) {
               this.selectValue = this.value.code;
               this.optionsSelect.unshift({
                 [this.option.code]: this.value.code,
                 [this.option.name]: this.value.name,
               });
             }
             // 数组去重
             this.optionsSelect = this.optionsSelect.reduce((pre, cur) => {
               if (
                 pre.findIndex(
                   (item) => item[this.option.code] === cur[this.option.code]
                 ) === -1
               ) {
                 pre.push(cur);
               }
               return pre;
             }, []);
             this.selTotal = res.records; // 总条数
           })
           .catch((err) => {
             console.log(err);
           });
       },
       // 监听查询的数据
       handleValueChange(value) {
         if (value) {
           this.$emit("updateValue", value);
         } else {
           this.srcollNum = 1;
           this.opEntityToName = "";
           this.handleLikeSearch("");
         }
       },
       // 下拉框出现 / 消失
       handleVisible(flag) {
         if (flag && !this.selectValue) {
           this.srcollNum = 1;
           this.opEntityToName = "";
           this.handleLikeSearch("");
         }
       },
       // 清空下拉
       handleClearSelected() {
         this.$emit("clear");
       },
     },
     created() {},
     mounted() {
       this.handleLikeSearch(); // 远程搜索加载下拉框数据
     },
   };
   </script>
  ```
- 整体上来说，整个组件的代码就是以上的部分了，具体的可以看其中的注释，接下来就是如何使用了。

  > 组件使用

- 引入并声明所需要的组件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7580d6406394d1899c05710b1c6d604~tplv-k3u1fbpfcp-watermark.image?)

- 具体使用，这是完整版的使用了，简单的介绍一下使用方法。
  ```js
  /*
      1. :value=remoteValue 作为反显数据时配置，具体如下
          remoteValue: {
            // 配置需要反显的select
            code: "xa",
            name: "西安",
          },
      2. api="/test/select" 下拉的接口（接口需要配置分页查询，模糊搜索）
      3. optionName="optionName" optionName为模糊搜索的字段，若后台字段为optionName则无需配置。
      4. :rows="10" 分页参数，默认为10，可不配置
      5. :option="{ code: 'optionCode', name: 'optionName' }" 数据反显的code和name字段，需配置。
      6. @updateValue="(v) => (ruleForm.attrInput5 = v)" 其中的ruleForm.attrInput5为页面中select绑定的数据字段，可随意更改，需在data中提前定义。
      7. @clear="() => (remoteValue = {})" 同反显，反显时需配置
  */
  <remote-select
    :value="remoteValue"
    api="/test/select"
    optionName="optionName"
    :rows="10"
    :option="{ code: 'optionCode', name: 'optionName' }"
    @updateValue="(v) => (ruleForm.attrInput5 = v)"
    @clear="() => (remoteValue = {})"
  ></remote-select>
  ```
- 简单使用，无需反显
  ```js
  <remote-select
      api="/test/select"
      optionName="optionName"
      :option="{ code: 'optionCode', name: 'optionName' }"
      @updateValue="(v) => (ruleForm.attrInput5 = v)"
    ></remote-select>
  ```
- 效果图，默认反显数据置顶。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ef009a73525482e92ebc784853fe6a2~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7164018be8b4a0498b24f743cea104c~tplv-k3u1fbpfcp-watermark.image?)

[附上 git 项目地址](https://github.com/KinXpeng/vue2-demo)

> 以上就是 select 的封装以及简单使用了，好用的话点个赞吧！
