# ReactRouter
react官方认可的路由管理库
## 安装
* 安装时我们选择react-router-dom
* react-router会包含一些react-native的内容，web开发并不需要。
```
npm install react-router-dom
```
## 基本使用
react-router最主要的API是给我们提供的一些组件
### 选择BrowserRouter或HashRouter
router中包含了对路径改变的监听，并且会将对应的路径传递给子组件。
* BrowserRouter:history模式
* HashRouter：hash模式
```jsx
import {HashRouter} from 'react-router-dom'

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <HashRouter>
      <App />
    </HashRouter>
  </React.StrictMode>
);
```
### 进行路由映射关系的配置
```jsx
/*App.js*/
import { Link, Route,Routes } from 'react-router-dom'
import Home from './pages/Home'
import About from './pages/About'

export class App extends PureComponent {
  render() {
    return (
      <div className='app'>
      <div className="nav">
      {/*router-link使用*/}
      {/*使用NavLink，会自动给选中的连接加上active标签47*/}
            <Link to="/home">首页</Link>
            <Link to="/about">关于</Link>
          </div>
        <div className="content">
          {/**映射关系：path => Component */}
          <Routes>
            {/* router5 */}
            {/* <Route path='/home' component={<Home/>}/> */}
            {/* router6 */}
            <Route path='/home' element={<Home/>}/>
            <Route path='/about' element={<About/>}/>
          </Routes>
        </div>
      </div>
    )
  }
}

export default App
```
### Navigate导航
Navigate用于**路由重定向**，
#### 应用场景一：登录确认后自动跳转到首页
```jsx
render() {
    const {isLogin} = this.state
    return (
      <div>
        <h1>Login page</h1>
        {!isLogin ? <button onClick={e=>this.login()}>login</button> : <Navigate to="/home"/>}
        
      </div>
    )
  }
```
#### 应用场景二：首次进入“/”无匹配路径，自动进入首页
```jsx
<Route path='/' element={<Navigate to="/home"/>}/>
```

#### 应用场景三：NOT FOUND页面
在所有路由的最后，添加一个Route。**path="*"表示空配**，路径没有匹配成功
```jsx
<Route path="*" element={<NotFound />}/>
```
### 配置路由嵌套
以Home页面和其子页面HomeRanking,HomeRecommend为例。
#### 一、统一配置嵌套路由
```jsx
/*App.jsx*/
<Routes>
  <Route path='/home' element={<Home/>}>
    <Route path='/home/recommend' element={<HomeRecommend/>}/>
    <Route path='/home/ranking' element={<HomeRanking/>}/>
  </Route>
</Routes>
```
#### 二、在home页面中设置占位符Outlet(类似于vue的router-view)
```jsx
/*Home.jsx*/
<div>
  <h1>Home Page</h1>
  <div className="home-nav">
    <Link to="/home/recommend">推荐</Link>
    <Link to="/home/ranking">排行榜</Link>
  </div>
  {/* 占位窗口 */}
  <Outlet/>
</div>
```
### 手动跳转路由——useNavigate
暂时只能在函数式组件/用户自定义hooks中进行跳转，如果要在类组件中进行跳转，需要用一个高阶组件对类组件进行**增强**，并传入useNavigate。
#### 在函数组件中使用
先引入useNavigate方法，该方法返回一个方法A，方法A接收路由参数，跳转到对应路由。
```jsx
export function App(){
  //方法调用得到方法A
    const navigate = useNavigate()
    function navigateTo(path){
      //传参调用进行路由跳转。
      navigate(path);
    }
    return (
      <div className='app'>
        <div className="header">
          <div className="nav">
            <button onClick={e=>navigateTo("/order")}>Order</button>
            {/*方法调用*/}
            <button onClick={e=>navigateTo("/categroy")}>Categroy</button>
          </div>
          <hr />
        </div>
        <div className="content">
          <Routes>{/*一些路由配置*/}</Routes>
        </div>
      </div>
    )
  }

```
#### 在类组件中使用
封装一个高阶组件，对当前的类组件的渲染进行增强。
```js
/*高阶组件*/
import { useNavigate} from 'react-router-dom';

//高阶组件
 export default function withRouter(WrapperComponent){
    return function(props){
      const navigate = useNavigate()
      const router = {navigate}
      return <WrapperComponent {...props} router={router}/>
    }
  }
```
在类组件被导出时使用
```jsx
import withRouter from '../hoc/with_router';
export class Home extends PureComponent {
  navigateTo(path){
    console.log(path)
    const {navigate} = this.props.router
    navigate(path)
  }
  render() {
    return (
      <div>
        <button onClick={e=>this.navigateTo("/home/songmenu")}>歌单</button>
        <Outlet/>
      </div>
    )
  }
}
export default withRouter(Home)
```
### 路由跳转传递参数
#### 动态路由传递
传递参数
```jsx
//动态路由传递参数：
<Route path="/detail/:id" element={<Detail/>}></Route>
```
获取参数，使用**useParams**方法，类组件依然不可直接用。需要通过高阶组件进行增强。
```jsx
/*withRouter*/
import { useNavigate,useParams} from 'react-router-dom';
 export default function withRouter(WrapperComponent){
    return function(props){
      const navigate = useNavigate()
      const params = useParams()
      const router = {navigate,params}
      return <WrapperComponent {...props} router={router}/>
    }
  }
```
组件中的使用
```jsx
import withRouter from '../hoc/with_router';
export class Detail extends PureComponent {
    render(){
        const {router} = this.props
    return (
      <div>
        <h2>id:{router.params.id}</h2>
      </div>
    )
  }
}
export default withRouter(Detail)
```
#### 将数据拼接在path后面
路由定义和普通一样，参数在跳转的时候添加。
```jsx
/*定义*/
<Route path="/user" element={<User/>}/>

/*跳转时携带参数*/
<Link to="/user?name=why&age=18">user</Link>
```
通过useSearchParams方法获取参数，原理同useNavigate,类组件依然需要通过高阶组件封装。
```js
/*高阶组件*/
import {useNavigate,useParams, useSearchParams} from 'react-router-dom';

 export default function withRouter(WrapperComponent){
    return function(props){
      //1.导航
      const navigate = useNavigate()
      //2.动态路由的参数：/detail/:id
      const params = useParams()
      //3.查询字符串的参数：/user?name=why&age=18
      const [searchParams] = useSearchParams()
      //将URLParamsObject转成普通对象
      const query = Object.fromEntries(searchParams)
      const router = {navigate,params,location,query}
      return <WrapperComponent {...props} router={router}/>
    }
  }
```
在组件中使用
```jsx
/*User.jsx*/
import withRouter from '../hoc/with_router';
export class User extends PureComponent {
  render() {
    const {query} = this.props.router
    return (
      <div>
        <h2>username:{query.name}</h2>
        <h2>userage:{query.age}</h2>
      </div>
    )
  }
}

export default withRouter(User)
```

### 将route配置到单独的文件中
```js
const routes=[
  {
    path:'/',
    element:<Navigate to="/home"/>
  },
  {
    path:"/home",
    element:<Home/>,
    children:[
      {
        path:"/home",
        element:<Navigate to="/home/recommend" />
      },
    ]
  }
]

```
```jsx
/*App.jsx*/
  {
    useRoutes(routes)
  }

```
### 分包懒加载
对于需要懒加载的组件,不能直接import导入：
```jsx
const About = React.lazy(()=>import("../pages/About"))
const Login = React.lazy(()=>import("../pages/Login"))
```
上述操作之后，在两个组件在打包的时候会单独打包到一个js文件中，只有页面需要加载了浏览器才会请求下载这部分的数据。在下载过程中可以渲染一个loading界面。
```jsx
/*src/index.js*/
import {Suspense} from 'react'

root.render({
    <Suspense fallback={<h3>Loading...</h3>}>
      <App />
    </Suspense>
})


```