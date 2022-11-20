# React中如何编写css?
整个前端已经是组件化的天下了，而css的设计就不是为组件化而生的。所以目前组件化的框架中都需要一种合适的CSS解决方案。
## 组件化中css的解决方案应该满足
* 可以编写局部css：不会随意污染其他组件内的元素
* 可以编写动态的css，获取当前组件的一些状态，根据状态的变化生成不同的css样式
* 支持所有的css特性：伪类，动画，媒体查询。

## 内联样式css
内联样式是官方推荐的一种css样式的写法。
### 写法
* style接受一个采用小驼峰命名属性的js对象，而不是css字符串。
* 并且可以引用state中的状态来设置相关的样式。
```jsx
export class App extends PureComponent {
  constructor(){
    super()
    this.state={
      titleSize:30
    }
  }
  render() {
    const {titleSize} = this.state
    return (
      <div>
        <h2 style={{color:"red",fontSize:`${titleSize}px`}}>我是标题</h2>
        <p style={{color:"blue",fontSize:"20px"}}>我是内容，哈哈哈</p>
      </div>
    )
  }
}
```

### 优点
* 内联样式之间不会有冲突
* 可以动态获取当前state中的状态
### 缺点
* 写法上都需要驼峰标识(font-size => fontSize)
* 某些样式没有提示
* 大量样式，代码混乱
* 某些样式无法编写（伪类）
## 普通css文件写法
普通css我们通常会单独编写到一个文件中，再通过import导入。
### 优点
* 简单

### 缺点
* 不论是同一层级还是不同层级，css的样式会相互污染。没有作用域。
* 导致样式层叠
## css Module写法
css module在类似于webpack配置的环境下都可以使用。解决了作用域相互影响的问题。
### 用法
* css文件必须命名含有module(App.css => App.module.css).
* 所有的className都必须使用className={style.className}的形式来编写.

### 优点
解决了普通文件css作用域相互影响的问题。
### 缺点
* 引用的类名不能使用连接符，在js中不识别
* 所有的className都必须使用{style.className}的形式来编写.
* 动态修改样式不方便，依然需要使用内联样式的方式。

```jsx
import appStyle from './App.module.css'

export class App extends PureComponent {
    render() {
        return (
            <div>
               <h2 className={appStyle.title}>我是标题</h2>
               <p className={appStyle.content}>我是内容，哈哈哈</p>
               <Home />
               <Profile/>
            </div>
        )
    }
}
```
## less的编写方式
采用craco工具对webpack进行配置，解析less文件。
### 安装
```
 //create-react-app : 4.*
 npm install @craco/craco

 //create-react-app : 5.*
 npm install @craco/craco@alpha
```
### webpack配置
启动方式改为craco启动
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
### 引用craco-less来帮助加载less样式和修改变量
安装craco-less
````
//create-react-app:4.*
$ npm install craco-less

//create-react-app:5.*
$ npm install craco-less@alpha
````
### 创建craco.config.js用于修改默认配置
```js
/* craco.config.js */
module.exports = {
  // ...
};
```
对craco.config.js进行配置
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

## css in js解决方案
CSS-in-JS是一种模式，其中CSS由js生成而不是在外部文件中定义，它通过js来为css赋予一些能力，包括类似于css预处理器一样的样式嵌套，函数定义，逻辑复用，动态修改状态等等。
### styled-components库
styled-components库是CSS-in-JS的库
* 安装
   ````
    $ npm install styled-components
   ````
* ES6模板字符串
  ```js
    let name = "why";
    let age = 18;
  //1.普通使用
    const str =  `my name is ${name},age is ${18}`
    console.log(str)//my name is why,age is 18
  //2.标签模板字符串的使用
    function foo(...args){
      console.log(args)
    }
    //foo函数的调用
    foo`my name is ${name},age is ${age}`
    /*打印出来一个数组
    * 0:['my name is','age is',' '],
    * 1:name
    * 2:age
    */

  ``` 
  ### 使用方法
  1. 单独建立一个js文件，导入styled-components包之后，使用模板字符串对要做样式的区域进行样式编写。，并将其export导出
```js
  /* style.js*/
  import styled from 'styled-components'
 //基本使用
  export const AppWrapper = styled.div`
      .section{
          color:red;
          .title{
              font-size:30px;
          }
      }
      .footer{
          color:grey;
      }
  `
```
2. 在对应的jsx文件中导入，对需要做样式的区域进行包裹。
```js
  /*App.jsx*/
  import {AppWrapper} from './style'
  export class App extends PureComponent {
      render() {
          return (
              <AppWrapper>
                  <div className="section">
                      <h2 className="title">我是标题</h2>
                      <p className="content">我是内容，哈哈哈</p>
                  </div>
                  <div className="footer">
                      <p>免责声明</p>
                      <p>版权声明</p>
                  </div>
              </AppWrapper>
          )
      }
  }

  export default App
```
3. 对于动态的数据,直接在组件的state中定义。从js文件返回的组件上传递过去，并其对应的js文件中进行调用.
```js
/*App.jsx*/
import {AppWrapper, SectionWrapper} from './style'
export class App extends PureComponent {
    constructor(){
        super()
        this.state={
            color:"blue",
            size:15
        }
    }
    render() {
        const {color,size} = this.state
        return (
            <AppWrapper>
                <SectionWrapper color={color} size={size}>
                    <h2 className="title">我是标题</h2>
                    <p className="content">我是内容，哈哈哈</p>
                </SectionWrapper>
            </AppWrapper>
        )
    }
}

/*style.js*/
import styled from 'styled-components'
//子元素单独抽取到一个样式组件
//可以接收外部的props
export const SectionWrapper = styled.div`
        color:red;
        .title{
            font-size:${props=>props.size}px;
            color:${props=>props.color};
            &:hover{
                background-color: pink;
            }
        }
    
`
```
4. 可以通过attrs给标签模板字符串中提供默认属性，以及可以在外部统一设定一些默认规定，然后引入使用。
```js
/*variable.js*/
export const primaryColor = "#ff8800"
export const secondColor = "#ff7788"

export const smallSize = "12px"
export const middleSize = "14px"
export const largeSize = "18px"

/*styled.js*/
import styled from 'styled-components'
import {
    primaryColor,
    secondColor,
    smallSize,
    middleSize,
    largeSize
} from './style/variable'

export const SectionWrapper = styled.div.attrs(props=>({
  tColor:props.color || "yellow"
}))`
        color:red;
        .title{
            font-size:${props=>props.size}px;
            color:${props=>props.tColor};
            &:hover{
                background-color: pink;
            }
        }
        .content{
            font-size: ${smallSize};
            color: ${primaryColor};
        }
    
`
```
## classnames库使用方案
快捷的动态添加class
### 安装
```
$ npm install classnames
```
### 使用
```jsx
import React, { PureComponent } from 'react'
import classNames  from 'classnames'

export class App extends PureComponent {
    constructor(){
        super()
        this.state={
            isbbb:true,
            isccc:false
        }
    }
  render() {
    const {isbbb,isccc} = this.state
    return (
      <div>
        {/* 常规用法 */}
        <h2 className={`aaa ${isbbb ? 'bbb':''} ${isccc?'ccc':''}`}>哈哈哈</h2>
        {/* classnames的用法 */}
        <h2 className={classNames("aaa",{bbb:isbbb,ccc:isccc})}>呵呵呵</h2>
      </div>
    )
  }
}
```