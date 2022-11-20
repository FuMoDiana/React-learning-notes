# 基于React全家桶进行项目搭建的步骤

## 创建项目
### create-react-app
````
npx create-react-app my-app
````

### 项目配置
* 配置项目的icon
* 配置项目的标题
* 配置jsconfig.json
  ```json
    {
    "compilerOptions": {
      "target": "es5",
      "module": "esnext",
      "baseUrl": "./",
      "moduleResolution": "node",
      "paths": {
        "@/*": [
          "src/*"
        ]
      },
      "jsx": "preserve",
      "lib": [
        "esnext",
        "dom",
        "dom.iterable",
        "scripthost"
      ]
    }
  }
  ``` 
* 配置目录结构
  * stores(redux)
  * views
  * components
  * router(react-router)
  * hooks
  * assets
  * services(axios)

### craco配置路径别名 @ -> src,less
#### 安装craco
```js
//create-react-app:4.*
$ npm i @craco/craco

//create-react-app:5.*
$ npm i @craco/craco@alpha -D
```
#### 安装craco-less
````js
//create-react-app:4.*
$ npm install craco-less

//create-react-app:5.*
$ npm install craco-less@alpha
//craco:7.*
$ npm i craco-less@2.1.0-alpha.0
````
#### 配置craco.config.js
```js
const path = require('path')
const resolve = (path) => path.resolve
(__dirname,path)
const CracoLessPlugin = require('craco-less');
module.exports = {
  //less
  plugins: [
    {
      plugin: CracoLessPlugin,
    },
  ],
  //webpack
  webpack:{
    alias:{
      "@":resolve("src"),
      "components":resolve("src/components"),
      "utils":resolve("src/utils")
    }
  },
}
```
#### 修改package.json，更改启动方式改为craco启动
```json
/* package.json */
"scripts": {
-   "start": "react-scripts start",
-   "build": "react-scripts build",
-   "test": "react-scripts test",
+   "start": "craco start",
+   "build": "craco build",
+   "test": "craco test",
}
```

### CSS样式重置
#### normalize.css
样式重置库
```js
$ npm install normalize.css
```
在项目中引入
```js
/* src/index.js */
import "normalize.css"
```
#### reset.css
自定义的样式重置
```less
*{
  padding: 0;
  margin: 0;
}
```

### Router配置
#### 安装
```js
$ npm install react-router-dom
```
#### 在index.js中导入
选择路由模式（hash/history）
```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import {HashRouter} from 'react-router-dom';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <HashRouter>
      <App />
    </HashRouter>
  </React.StrictMode>
);

```
#### 配置一个单独的routes文件
```js
/* router/index.js */
const routes = [
  {
    path:"/home",
    element:<Home/>
  }
]
export default routes;
```
#### 在app中引用路由
```jsx
import React, { memo } from 'react'
import routes from '@/router';
import { useRoutes } from 'react-router-dom';

const App = memo(() => {
  return (
    <div className='app'>
      <div className="content">{useRoutes(routes)}</div> 
    </div>
  )
})

export default App
```

### Redux配置
#### 安装(RTK模式)
```js
$ npm i @reduxjs/toolkit react-redux
```
#### 初始化store
```js
/* store/index.js */
import {configureStore} from '@reduxjs/toolkit';

const store = configureStore({
  reducer:{
    
  }
})
export default store;
```
#### 分模块划分store创建slice
以home.js为例
```js
import {createSlice} from '@reduxjs/toolkit';

const homeSlice = createSlice({
  name:'home',
  initialState:{
    counter:0
  },
  reducers:{
    addNumber(state,{payload}){
      state.counter = state.counter+payload
    }
  }
})
export const {addNumber} = homeSlice.actions
export default homeSlice.reducer;
```
#### store根文件导入home.js
```js
import {configureStore} from '@reduxjs/toolkit';
import homeReducer from './modules/home';

const store = configureStore({
  reducer:{
    home:homeReducer,
  }
})

export default store;
```
#### 根文件中注入store
```js
/* src/index.js */
import { Provider } from 'react-redux';
import store from '@/store';
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <HashRouter>
        <App />
      </HashRouter>
    </Provider>
  </React.StrictMode>
);
```

### 网络请求-axios配置
#### 安装
```js
$ npm i axios
```
#### 搭建services目录结构
* modules---存放各个模块的网络请求
* request---封装axios
  * index.js---封装axios
    ```js
      import axios from 'axios';
      import {BASE_URL,TIMEOUT} from './config';

      class HYRequest{
        constructor(baseURL,timeout){
          this.instance = axios.create({
            baseURL,
            timeout
          })
          //拦截器
          this.instance.interceptors.response.use((res)=>{
            return res.data
          },err=>{
            return err
          })
        }

        request(config){
          return this.instance.request(config)
        }

        get(config){
          return this.request({...config,method:"get"})
        }

        post(config){
          return this.request({...config,method:"post"})
        }
      }

      export default new HYRequest(BASE_URL,TIMEOUT);
    ``` 
  * config.js---设置一些常量(baseURL,timeout)
    ```js
      export const BASE_URL = "http://codercba.com:1888/airbnb/api"
      export const TIMEOUT = 1000
    ``` 
* index.js---出口文件

### css配置
安装styled-components库
```js
$ npm i styled-components
```

# 项目开发中遇到的问题以及解决方式
## svg引入的问题
1. 在assets中建了svg文件夹，从网站上爬下来的svg做成组件的形式返回。
2. 在react中内联样式需要采用对象的语法，但是爬下来的svg的style是字符串语法，需要字符串转对象，在npm上安装了一个库(style-to-object),完美解决。

## styled-components管理主题色
### 创建一个存储主题的文件——theme(assets/theme)
在theme文件中创建一个js文件，导出theme对象。
```js
/* theme/index.js */
const theme = {
  color:{
    primaryColor:'#ff385c',
    secondaryColor:'#00848A'
  }
}

export default theme;

```
### 导入ThemeProvider(from styled-components)
在根目录的index.js下导入ThemeProvider，注入theme
```jsx
import {ThemeProvider} from 'styled-components';
import theme from '@/assets/theme';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <ThemeProvider theme={theme}>
    <App/>
  </ThemeProvider>
);
```
### 在子组件的style.js中使用
```js
/* components/app-header/style.js */
import styled from "styled-components";

const LeftWrapper = styled.div`
  flex:1;
  color:${props => props.theme.color.primaryColor};
`

```

### styled-components的样式混入(mixin)
在theme中定义mixin
```js
const theme = {
  mixin:{
    boxShadow:`
    transition: box-shadow 0.2s ease;
    &:hover{
      box-shadow: 0 2px 4px rgba(0,0,0,.18);
    }`
  }
}
```
在组件的style.js中使用
```js
import styled from "styled-components";

const RightWrapper = styled.div`
  flex:1;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  ${props=>props.theme.mixin.boxShadow};
`
export default RightWrapper;
```

#### 背景图片引入
webpack打包时会把图片url识别成string，导致图片不能渲染到页面上。
* 导入图片
  ```js
    import styled from "styled-components";
    import coverImage from '@/assets/img/homeTop.jpg';

    const BannerWrapper = styled.div`
      height: 529px;
      background: url(${coverImage}) center/cover;
    `

    export default BannerWrapper;
  ``` 
* require对字符串进行包裹
  ```js
    import styled from "styled-components";

    const BannerWrapper = styled.div`
      height: 529px;
      background: url(${require("@/assets/img/homeTop.jpg")}) center;
    `

    export default BannerWrapper;
  ``` 

## 网络请求
将异步请求的逻辑代码放在store和services中，导出调用请求的方法。
### 组件调用(dispatch)函数(从store对应模块中导出)
  ```js
    const dispach = useDispatch();
    useEffect(()=>{
      dispatch(xxxAction())
    },[])

    const {xxx} = useSelector((state)=>state.home.xxx,shallowEquall)
  ``` 
### store中的异步action调用services对应模块导出的方法，得到一个promise。通过then得到数据并将数据存入store。
  ```js
    export const fetchHomeDateAction = createAsyncThunk("fetchdate",async ()=>{
      const res = await getHomeGoodPriceData();
      return res;
    })

    const homeSlice = createSlice({
      name:'home',
      initialState:{
        xxx:{}
      },
      extraReducers:{
        [fetchHomeDateAction.fulfilled](state,{payload}){
          state.xxx = payload;
        }
      }
    })
  ```
### services对应模块写入网络请求的相关代码。
  ```js
  import hyRequest from '@/services'

  export function xxxGetData(){
    return hyReauest.get({
      url:"/xxx/xxx"
    })
  }

  ``` 
### 一个异步reducer进行多个网络数据的请求并保存
createAsyncThunk传入的回调函数有两个参数，payload和当前的store对象，可以解构出dispatch方法。
```js
//获取网络数据
export const fetchHomeDateAction = createAsyncThunk("fetchdate",(payload,{dispatch})=>{
  getHomeGoodPriceData().then(res=>{
    dispatch(changeGoodPriceInfoAction(res))
  })
  getHomeHighScoreData().then(res=>{
    dispatch(changeHighScoreInfoAction(res))
  })
})
```

## 组件库应用
### Material UI
#### 安装
Material UI默认使用适配@emotion/styled。要适配styled-components需要额外安装。(默认已经安装了styled-components).**emotion也必须安装！！**
````js
$ npm install @mui/material @mui/styled-engine-sc
$ npm install @emotion/react @emotion/styled
````
#### 使用
```jsx
import * as React from 'react';
import Button from '@mui/material/Button';

function App() {
  return <Button variant="contained" sx={{/*传入一个对象，支持自定义一些样式*/}}>Hello World</Button>;
}

```



### Ant Design
#### 安装
```
$ npm i antd
```
#### 引入对应文件
```less
/* assets/css/index.less */
@import '~antd/dist/antd.css';
```
#### webpack配置less
```js
const CracoLessPlugin = require('craco-less');

module.exports = {
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        lessLoaderOptions: {
          lessOptions: {
            modifyVars: { '@primary-color': '#1DA57A' },
            javascriptEnabled: true,
          },
        },
      },
    },
  ],
};
```


## 一些css解决方案

### styled的属性都可以动态传入，使用插值模板
```js
/*组件内*/
<RoomItemWrapper itemWidth={itemWidth}></RoomItemWrapper>

/*style.js内*/
const RoomItemWrapper = styled.div`
  width:${props=>props.itemWidth};
`

```