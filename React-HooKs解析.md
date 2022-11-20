# React-Hooks解析
Hook时React16.8的新增特性，它可以让我们在**不编写class的情况下**使用**state以及其他的React特性**(比如生命周期)。
## 函数组件与类组件对比
### 函数组件的缺陷
1. 修改数据后，组件不会自动重新渲染页面，不会更新ui
2. 如果页面重新渲染，函数会被重新执行，上一次变更的数据会被再次初始化，导致页面回归原始。

### 类组件相对于函数组件的优势
1. 可以定义自己的state，且状态改变时render会自动重新执行。
2. 有自己的生命周期，可以在对应的生命周期中编写逻辑代码

### 类组件的缺点
1. 随着业务增多，class组件会越来越复杂和臃肿
2. 在同一个生命周期中可能会存在大量逻辑代码，比如网络请求，事件监听等等。
3. 逻辑拆分困难，往往类组件的逻辑是混合在一起的，强行拆分只会造成过度设计。（hoc）
4. 组件复用状态很难，需要通过**高阶组件**实现

## Hook的出现
几乎可以解决掉上面提到的这些问题。它可以让我们在**不编写class的情况下**使用**state以及其他的React特性**(比如生命周期)

### Hook使用场景
* 基本可以代替之前所有使用class组件的地方
* Hook完全向下兼容，重构旧代码不需要将class完全变成hook。
* Hook只能在函数组件中使用，不能在类组件或者函数组件之外的地方使用。
### Hook使用的额外规则
1. Hook只能在函数最外层调用Hook，不要在循环，条件判断或者子函数中调用。
2. 只能在React的函数组件中调用Hook，不要在其他js函数中调用

## Hook初体验
### 引入useState,并使用
以实现计数器为例
```jsx
import {memo,useState} from 'react';

function CounterHook(props){
    //useState会返回一个数组，数组第一位为响应式数据，第二位为修改该数据的方法
    const [counter,setCounter] = useState(0)
    return(
        <div>
            <h2>当前计数:{counter}</h2>
            <button onClick={e => setCounter(counter+1)}>+1</button>
        </div>
    )
}
export default memo(CounterHook);
```
## useState
useState来自react，需要从react中导入，是一个hook。useState会帮助我们定义一个变量，与class中的this.state提供的功能完全相同。**初始化值只有组件第一次渲染的时候有用！若为异步数据，会导致渲染不上！**
* 参数：接收唯一一个参数，第一次调用为初始化值，不传递默认为undefined
* 返回值：数组，包含两个元素
  * 元素一：当前状态的值（第一调用为初始化值）
  * 元素二：设置状态值的函数
一般来说，函数在退出后，变量就会消失，而state中的变量会被React保留。

## Effect Hook(useEffect)
可以帮助完成类似类组件中的生命周期的功能。事实上类似于网络请求，手动更新DOM，一些事件的监听，都是React更新DOM的一些副作用。所以对于完成这些功能的Hook被称之为Effect Hook。
```jsx
useEffect(()=>{
        /**
         * 当前传入的回调函数会在组件被渲染完成后自动执行
         * 网络请求/DOM操作(修改标题)
        */
        document.title = counter
    })
```
在class组件的编写过程中，某些副作用(事件监听)的代码，我们需要在componentWillUnmount中清除副作用。useEffect同理。**useEffect传入的回调函数可以返回一个回调函数**，该回调函数会在组件被重新渲染/组件卸载的时候被执行。
```jsx
    useEffect(()=>{
        //当前传入的回调函数会在组件被渲染完成后自动执行
        document.title = counter
        console.log("监听")
        //返回值：回调函数 => 组件被重新渲染/组件卸载时自动执行。
        return ()=>{
            console.log("取消监听")
        }
    })
```
### **为什么要在effect中返回一个函数？**
* 这是effect可选的清除机制，每个effect都可以返回一个清除函数。
* 如此可以将**添加和移除订阅的逻辑放在一起**
* 他们都属于effect(副作用)的一部分。

### **React何时清除effect？**
* React会在组件更新和卸载的时候执行清除操作。
* effect每次渲染的时候都会执行。

### effect逻辑分离
Hook可以解决class生命周期经常将很多逻辑放在一起的问题。它允许我们按照代码的用途分离它们。
```jsx
useEffect(()=>{
    console.log("修改标题")
})
useEffect(()=>{
    console.log("监听事件")
    return ()=>{
        console.log("取消监听")
    }
})
useEffect(()=>{
    console.log("发布订阅")
    return ()=>{
        console.log("取消订阅")
    }
})
```
### effect性能优化
每次组件重新渲染effect都会重新执行，但是对于很多操作来说(网络请求)，在组件刚渲染的时候执行一次就可以了，类似于componentDidMount和componentWillUnmount中执行的事。
#### useEffect的参数
* 参数一:回调函数
* 参数二:数组(表示该useEffect在哪些state发生变化时，才重新执行（受谁的影响）)
当参数二传入空数组时，表示不受任何状态的影响，可模拟componentDidMount函数。不传入数组的话，useEffect则会在每一次渲染后都重新执行。
```jsx
useEffect(()=>{
    console.log("组件渲染了")
})
useEffect(()=>{
    console.log("Component Did Mount")
},[])
useEffect(()=>{
    console.log("依赖counter")
},[counter])
```
* 初始渲染状态：
  ![alt useEffect初始渲染](./pics/useEffect%E5%88%9D%E5%A7%8B.png)
* 点击一次【+1】按钮
  ![alt useEffext中间渲染](./pics/useEffect%E4%B8%AD%E9%97%B4.png)

## useContext
使用过程非常简单。且如果组件依赖的*context*中的数据发生变化了，组件也会重新渲染。*useContext*(MyContext)只是让你能够读取*context*的值以及订阅*context*的变化。你仍然需要在上层组件树中使用 *<MyContext.Provider>*来为下层组件提供*context*。
```js
/*index.js*/
import App from './App';
import {UserContext,ThemeContext} from './context';
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <UserContext.Provider value={{name:"why",age:18}}>
    <ThemeContext.Provider value={{color:"red",size:30}}>
      <App />
    </ThemeContext.Provider>
  </UserContext.Provider>
);
```
App中使用useContext
```jsx
/*App.js*/
import React, { memo,useContext } from 'react'
import {UserContext,ThemeContext} from './context';
const App = memo(() => {
  //使用context
  const user = useContext(UserContext)
  const theme = useContext(ThemeContext)
  return (
    <div>
        <h2 style={{color:theme.color,fontSize:theme.size}}>User:{user.name}</h2>
        <h2>Age:{user.age}</h2>
    </div>
  )
})
export default App
```

## 其他Hooks

### useReducer
*useReducer*仅仅可以作为useState的替代方案。它接收一个形如**(state, action)=>newState*的 *reducer*，并返回当前的*state*以及与其配套的*dispatch*方法。
```jsx
import { useState } from 'react'
function reducer(state,action){
    switch(action.type){
        case "increment":
            return {...state,counter:state.counter+1}
        case "decrement":
            return {...state,counter:state.counter-1}
        default:
            return state
    }
}
const App = memo(() => {
  // const [counter,setCounter] = useState(0)
  const [state,dispatch] = useReducer(reducer,{counter:0,friends:[],user:{}})

  return (
    <div>
        {/*useState*/}
        {/* <h2>当前计数：{counter}</h2>
        <button onClick={e=> setCounter(counter+1)}>+1</button>
        <button onClick={e=> setCounter(counter-1)}>-1</button> */}

        {/*useRedecer*/}
        <h2>当前计数：{counter}</h2>
        <button onClick={e=> dispatch({type:"increment"})}>+1</button>
        <button onClick={e=> dispatch({type:"decrement"})}>-1</button>
    </div>
  )
})

export default App
```

#### 对比useState的好处
useState需要单独对每个数据做定义，如果是useReducer的话，可以整合到一起。但是不推荐使用，如果需要复杂数据的管理，推荐直接使用redux。

### useCallback和useMemo
在函数组件中定义方法，当组件重新渲染的时候，函数就会被重新定义一次，如果组件频繁更新，那函数会反复被定义，导致内存中存在多个地址不同但功能相同的函数。
```jsx
import React, { memo, useCallback,useState } from 'react'

const App = memo(() => {
  const [counter,setCounter] = useState(0)
  const increment =useCallback(e=>{
    setCounter(counter+1)
  })

  return (
    <div>
        <h2>计数：{counter}</h2>
        <button onClick={increment}>+1</button>
    </div>
  )
})

export default App
```
useCallback可以避免组件频繁渲染时，创建多个函数，每次都会返回那个函数。但是传入给useCallback的函数仍然会随着组件每一次的渲染重新定义。
#### 优化了哪里
1. useCallback返回一个 memoized 回调函数。当需要将一个函数传递给子组件时，最好使用useCallback进行优化，将优化之后的函数，传递给子组件。如果这个函数所依赖的数据没有发生改变，那么子组件就不会重新渲染。(不优化的话子组件会重新渲染【引用相等性】)

### useMemo
useMemo和useCallback的使用方法一样，不同的是，**useMemo返回的是传入的回调函数的执行结果**。
```js
const increment = useMemo(fn,[])
/*相当于*/
const increment2 = useCallback(()=>fn(),[])

```
#### 如何优化？
* useMemo返回的也是一个**memoized**值。
* 在依赖不变的情况下，多次定义的时候，返回的值是相同的。

### 需要useCallback和useMemo的场景
* 进行大量的计算操作，是否有必要每次渲染时都重新计算。
* 对子组件传递相同内容的对象时，用useMemo进行性能优化
  * 父组件重新渲染了，父组件中定义的对象会重新定义，地址会变化，导致依赖该对象的子组件也被重新渲染。如果希望子组件仅仅依赖于对象的值(内容)而不是地址的话，使用useMemo处理后再传入子组件。

### useRef
类似于class组件中的createRef，使得React可以获取到dom元素。
#### 基本使用
```jsx
import React, { memo,useRef } from 'react'

const App = memo(() => {
    const titleRef = useRef()
    const inputRef = useRef()
    function showTitleDom(){
        console.log(titleRef.current)
        // `current` 指向已挂载到 DOM 上的文本输入元素
        inputRef.current.focus()
    }
  return (
    <div>
        <h2 ref={titleRef} className='title'>Hello world</h2>
        <input type="text" ref={inputRef}/>
        <button onClick={showTitleDom}>查看title的dom</button>
    </div>
  )
})

export default App
```
useRef 返回一个可变的 ref 对象，其 *.current* 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内持续存在。但是**通过useRef()创建出的对象，数据改变其地址也不会变，结合useCallback使用，可以解决闭包陷阱。**
```jsx
const [counter,setCounter] = useState(0)
const counterRef = useRef()
counterRef.current = counter

const increment = useCallback(()=>{
  //每次重新渲染只能捕获到第一次的counter，
  setCounter(counter+1)
  //current的值会随着counter更新且对象地址不变
  setCounter(counterRef.current + 1)
})

```
#### 总结为两种用法
* 引入dom(或者class组件)元素
* 保存一个数据，这个对象再整个生命周期中可以保存不变。

### useSelector和useDispatch
将redux中的state和dispatch函数直接映射到组件中
```jsx
import React, { memo } from 'react'
import {useDispatch, useSelector} from 'react-redux';
import { addNumber } from './store/modules/counter';
const App = memo((props) => {
  //1.使用useSelector将redux中的store数据映射到组件
  const {counter} =  useSelector((state)=>({
    counter:state.counter.counter}
  ))

  //2.useDispatch直接使用派发函数
  const dispatch = useDispatch()
  function addNum(num){
    dispatch(addNumber(num))
  }
  return (
    <div>
      <h2>当前计数：{counter}</h2>
      <button onClick={e=>addNum(1)}>+1</button>
      <button onClick={e=>addNum(-1)}>-1</button>
    </div>
  )
})

export default App
```
但是useSelector监听的是整个state对象，如果对象的其他值被改变，redex会生成一个新的对象来代替，导致其他不依赖该值但使用了该对象的组件也被重新渲染。
```jsx
//memo高阶组件包裹起来的组件有对应的特点：只有props发生改变时，才会重新渲染
//Home组件只依赖了message，但是state中的counter改变的时候，home组件也会重新渲染
//给useSelector传入第二个参数[shallowEqual]，对对象进行浅层比较，值不变的话就不重新渲染
const Home = memo((props)=>{
  const {message} = useSelector((state)=>({
    message:state.counter.message
  }),shallowEqual)
  const dispatch = useDispatch();
  function changeMes(str){
    dispatch(changeMessage(str))
  }
  console.log("home render")
  return(
    <div>
      <h2>Home Page</h2>
      <h3>message:{message}</h3>
      <button onClick={e=>changeMes("你好啊")}>changeMessage</button>
    </div>
  )
})

const App = memo((props) => {
  const {counter} =  useSelector((state)=>({
    counter:state.counter.counter}
  ),shallowEqual)
    console.log("app render")
  const dispatch = useDispatch()
  function addNum(num){
    dispatch(addNumber(num))
  }
  return (
    <div>
      <h2>当前计数：{counter}</h2>
      <button onClick={e=>addNum(1)}>+1</button>
      <button onClick={e=>addNum(-1)}>-1</button>
      <Home/>
    </div>
  )
})
```

### useId
*useId*是一个用于生成横跨服务端和客户端的稳定的唯一 ID 的同时避免hydration不匹配的hook。hydration是ssr的一部分。

### useTransition
返回一个状态值表示过渡任务的等待状态，以及一个启动该过渡任务的函数。startTransition 允许你通过标记更新将提供的回调函数作为一个过渡任务。
## 自定义Hooks
要求：必须以use开头，function useXXX(){}
### 简单示例，模拟生命周期
```jsx
import React, { memo,useEffect } from 'react'
import { useState } from 'react'
/*自定义hook，打印生命周期*/
function useLogLife(cName){
    useEffect(() => {
      console.log(cName + "组件被创建")
    
      return () => {
        console.log(cName + "组件被销毁")
      }
    }, [])
    
}
/*定义一个组件*/
const Home = memo(() => {
  //使用该Hook
    useLogLife("home")
  return (
    <div>
        <h1>Home Page</h1>
    </div>
  )
})
/*App组件控制home的销毁*/
const App = memo(() => {
    const [isShow,setIsShow] = useState(true) 
  return (
    <div>
        <h1>App Root Component</h1>
        {isShow && <Home />}
        <button onClick={e=>setIsShow(!isShow)}>Show_Component</button>
    </div>
  )
})
//控制台打印
export default App
```