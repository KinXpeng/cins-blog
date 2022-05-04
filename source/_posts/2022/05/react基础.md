---
title: React 基础配置信息
date: 2022-05-04 14:21:09
id: 1
tags: React
categories: React
---

#### React 配置注意项

1. `component` 和 `render` 同时存在时，`component` 优先。

2. 创建项目

   ```bash
   npx create-react-app my-app
   ```

3. 使用 scss

   ```bash
   // react默认有scss，只需要安装node-sass(不过下面一行命令即可)
   yarn add sass-loader node-sass
   ```

4. 安装路由(使用 yarn add 安装或者使用 npm install)

   ```bash
   yarn add react-router --save
   yarn add react-router-dom --save
   ```

5. axios 请求的配置

   - 安装`axios`

     ```
     npm install axios --save
     ```

   - 封装 `axios`

     - 在`/src`下新建 http 目录，新建`axios.js`

     ```js
     // 引入axios
     import axios from 'axios';

     const $http = axios.create({
       // 固定请求地址
       baseURL: 'https://www.fastmock.site/mock/40987d446991a56b19af3159f9c7b491/api',
       // 请求超时时间
       timeout: 2000,
       // 当前请求默认请求头
       // header:{

       // }
     });

     // 创建请求拦截
     $http.interceptors.request.use((config) => {
       // console.log(config);
       config.headers.token = sessionStorage.getItem('token');
       return config;
     });

     // 创建响应拦截
     $http.interceptors.response.use((res) => {
       if (res.data.code === 0) {
         return res.data.data;
       } else {
         return Promise.reject(res.data);
       }
     });

     export default $http;
     ```

   - 使用 `axios`

     ```js
     // 引入请求
     import $http from '../../http/axios';

     // 发起请求
     $http
       .post('/login', this.state)
       .then((res) => {
         console.log(res);
         this.props.history.push('/');
       })
       .catch((err) => {
         console.log(err);
       });
     ```

##### React 路由：子组件中使用 this.props 找不到 history

```js
// 子组件中使用this.props.history进行跳转，发现this.props中为空
// 查阅发现只有用Route包裹的组件才会有history信息
// 解决办法
import React, { Component } from 'react';
import { withRouter } from 'react-router-dom'; // 引入withRouter高阶组件

class HeaderCom extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <div>header</div>;
  }
}
export default withRouter(HeaderCom); // 导出时用高阶组件包裹
```

- 路由确切匹配用`exact`

  ```js
  import './App.css';
  import { BrowserRouter as Router, Switch, Route } from 'react-router-dom';
  import Home from './views/home/home';
  import List from './views/list/list';
  import Login from './views/login/login';
  import Info from './views/info/info';

  function App() {
    return (
      <div className="App">
        <Router>
          <Switch>
            <Route exact path="/" component={Home} />
            <Route path="/list" component={List} />
            <Route path="/login" component={Login} />
            <Route path="/info" component={Info} />
            {/* <Route path="/" component={Home} /> */}
          </Switch>
        </Router>
      </div>
    );
  }

  export default App;
  ```

- 函数式组件中使用`useState`，`useEffect`

```js
// useEffect写多个，按顺序执行，第二个参数可为空数组（只执行一次）
import React, { useState, useEffect } from 'react';
import Link from 'umi/link';
import { Button } from 'antd';
function List() {
  const [count, setState] = useState(0);
  // useEffect(() =>{
  //   //更新⻚⾯标题
  //   document.title=`您点击了${count}次了哦`
  // },[count])
  const handleAddClick = () => {
    console.log(111);
    setState(count + 1);
  };
  return (
    <div>
      <Link to="/">
        <span>返回首页</span>
      </Link>
      <div>你点击了{count}次</div>
      <div>
        <Button onClick={() => handleAddClick()}>按钮</Button>
      </div>
    </div>
  );
}
export default List;
```

- 解决 ts 中引入模块出现`any`类型的问题

  ```js
  // 或者使用重新安装axios依赖
  npm install @types/teaset
  ```

- umi 命令(使用 umi3 版本更加便捷)

  ```js
  umi g // 命令创建页面或者组件
  // 在项目根路径下执行
  umi g page xxx（要创建页面名）或umi g page xxx/xxx
  umi g page xxx --typescript --less // 创建ts页面与less
  umi g page 新建的文件名/—layout --typescript --less // 创建ts页面与less
  ```

- umi 中跨域请求代理设置

  ```js
  // 在项目根目录下的.umirc.js/ts中加入proxy配置
  proxy: {
    '/api': {
      target: 'https://api.kinxpeng.com',
      pathRewrite: { '^/api': '' },
      changeOrigin: true
    }
  },
  ```

- umi 项目中多环境打包配置

  - 安装 `cross-env`

  ```js
  yarn add cross-env
  ```

  - 根目录下创建 `.umirc.pro.ts` 文件（生产）

    ```js
    import { defineConfig } from 'umi';

    export default defineConfig({
      define: {
        ENV: 'pro',
        BASE_URL: 'http://localhost:8080/pro/', // 生产环境地址
      },
    });
    ```

  - 根目录下的默认 `.umirc.ts` 中的配置为开发环境

##### 项目中打断点使用 `debugger`

```js
$http
  .post('/login')
  .then((res) => {
    // console.log(res);
    debugger; // 此处为断点，请求停止
    message.success(res.data.msg);
  })
  .catch((err) => {
    console.log(err);
  });
```

##### 字符串类型的 ref 存在效率问题，可能会在未来版本移除

- 可以采用 createRef 的方式

- 以及回调函数的方式

  ```html
  <div ref={(c)=>{this.div = c}}></div>
  ```

##### React Input 无法修改 value

```js
const [InputValue,setInputValue] = useState('')

return (
  <div> // 这里的input用的是ant-design的UI库 所以是大写 但是写法就类似于这一种
    <Input value={InputValue} onChange={(event) => {
      setInputValue(event.target.value)
    }}>
  <div/>
)
```

##### umi3 中更改语言为中文

```js
// .umirc.ts文件中加入下面的配置
locale: {
  default: 'zh-CN',
  baseNavigator: true,
},
```

##### umi3.5.0 以上版本启用热更新

```js
// .umirc.ts文件中加入热更新配置
mfsu: {},
```

##### React 中的 input 动态绑定

```tsx
// 绑定onChange
const columns = [
  {
    title: '工单号',
    dataIndex: 'workOrderNo',
    render: (text: string, recode: any, index: number) => <Input value={text} onChange={(e) => handleTableInput(e, index)} />,
  },
];
// onChange事件
function handleTableInput(e: any, index: number) {
  let item: any = tableData[index]; // 获取修改该行数据
  item.workOrderNo = e.target.value; // 修改赋值
  tableData.splice(index, 1); // 删除原有数据行
  let newTableData: any = [item, ...tableData]; // 组合新数组
  newTableData.sort((a: any, b: any) => a.key - b.key); // 排序
  setTableData(newTableData); // 视图更新
}
```

#### React 中利用 EventBus 进行组件传值

```tsx
// 新建文件bus.ts
import { EventEmitter } from 'events'; // 引入event
export default new EventEmitter(); // 导出

// 需要传值的组件
import $bus from '@/utils/bus'; // 引入公共bus组件传值
$bus.emit('singleTab', data); // 使用emit触发

// 接收数据的组件
import $bus from '@/utils/bus'; // 引入公共bus组件传值
$bus.on('singleTab', (data) => {
  // 使用on监听
  console.log(data);
});
```

### NextJS 的安装及其基本配置

- 安装

  - 安装创建应用程序

    ```js
    npx create-next-app@latest
    # or
    yarn create next-app
    ```

  - 安装 axios

    ```js
    npm install axios -S
    ```

  - axios 的封装

    ```js
    import axios from 'axios';

    let baseURL;
    if (process.env.NODE_ENV === 'development') {
      // 开发(根据下方package.json中的环境变量的配置)
      baseURL = '/api';
    } else {
      // 生产
      baseURL = '/api';
    }
    const $axios = axios.create({
      // 固定请求地址
      baseURL: baseURL,
      // 请求超时时间
      timeout: 20000,
      // 当前请求默认请求头
      // header:{

      // }
    });

    // 创建请求拦截
    $axios.interceptors.request.use(
      (config) => {
        // config.headers.token = '123';
        return config;
      },
      (error) => {
        // 错误抛到业务代码
        return Promise.resolve(error);
      }
    );

    // 创建响应拦截
    $axios.interceptors.response.use((res) => {
      if (res.status === 200) {
        return res;
      } else {
        return Promise.reject(res);
      }
    });

    export default $axios;
    ```

- 环境变量的配置

  - package.json

    ```js
    // 前缀加上NODE_ENV=development
    "scripts": {
        "dev": "NODE_ENV=development node server.js -p 3006",
        "build": "NODE_ENV=production next build",
        "start": "next start",
        "lint": "next lint"
      },
    ```

  - 123

- 跨域代理

  - 安装好 express 和 http-proxy-middleware 中间件

    ```js
    yarn add express http-proxy-middleware
    ```

  - 在项目根目录下新建 server.js 文件，写入以下代码

    ```js
    const express = require('express');
    const next = require('next');
    const { createProxyMiddleware } = require('http-proxy-middleware');

    const devProxy = {
      '/api': {
        target: 'https://www.kinxpeng.com/demo/vue', // 端口自己配置合适的
        pathRewrite: {
          '^/api': '/',
        },
        changeOrigin: true,
      },
    };

    const port = parseInt(process.env.PORT, 10) || 3001;
    const dev = process.env.NODE_ENV !== 'production';
    const app = next({
      dev,
    });
    const handle = app.getRequestHandler();

    app
      .prepare()
      .then(() => {
        const server = express();
        if (dev && devProxy) {
          Object.keys(devProxy).forEach(function (context) {
            server.use(createProxyMiddleware(context, devProxy[context]));
          });
        }
        server.all('*', (req, res) => {
          handle(req, res);
        });

        server.listen(port, (err) => {
          if (err) {
            throw err;
          }
          console.log(`> Ready on http://localhost:${port}`);
        });
      })
      .catch((err) => {
        console.log('An error occurred, unable to start the server');
        console.log('发生错误，无法启动服务器');
        console.log(err);
      });
    ```

  - 相应地修改 package.json

    ```json
    "scripts": {
      "dev": "node server.js",
      "build": "next build",
      "start": "NODE_ENV=production node server.js"
    },
    ```

### React 18(reate-react-app 搭建)

- build 打包部署后，资源文件报 404
  ```json
  // package.json 增加如下配置
  "homepage": "./",
  ```
- 未完待续。。。
