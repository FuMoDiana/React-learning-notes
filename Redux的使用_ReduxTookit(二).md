# ReduxToolkit
createStore已经不被官方推荐使用了，官方推荐使用Redux Toolkit来编辑redux的逻辑。并且代码通常拆分在多个文件中。

## 安装
```
npm install @reduxjs/toolkit react-redux
```
## 使用
配合react-redux的provider和connect的核心使用方式还是和以前一样，只是store和reducer的定义方式更加简洁，不用拆分成多个文件进行代码管理了。

## 核心api
### configureStore
包装createStore以提供简化的配置选项和良好的默认值，它可以自动组合你的slice reducer，添加你提供的任何Redux中间件，redux-thunk默认包含并启用Redux DevTools Extension。

```jsx
/**
 * './store/index.js'
*/
import {configureStore} from '@reduxjs/toolkit'
import counterReducer from './features/counter'

const store = configureStore({
    reducer:{
        counter:counterReducer,
    }
})

export default store;
```
#### 常见参数
* reducer:将slice中的reducer组成一个对象传入此处。
* middleware：可以使用参数，传入其他中间件
* devTools：是否配置devTools工具，默认为true

### createSlice
接收reducer函数的对象，切片名称和初始状态值，并自动生成切片reducer，并带有相应的actions

```jsx
import {createSlice} from '@reduxjs/toolkit'

const counterSlice = createSlice({
    //用户标记slice的名词
    name:"counter",
    //初始化值
    initialState:{
        counter:888
    },
    //相当于之前的reducers函数
    reducers:{
        addNumber(state,action){
            //调用dispatch时给addNumber传递的参数被存在payload中
            //直接对state进行操作，会自动同步更新。
            state.counter = state.counter + action.payload
        }
    }
})

export default counterSlice.reducer;
```
### createAsyncThunk
接收一个动作类型字符串和一个返回promise的函数，并且自动生成一个pending/fulfilled/rejected基于该承诺分派动作类型的thunk.
```js
/*store/features/home.js*/
import {createSlice,createAsyncThunk} from '@reduxjs/toolkit'
import axios from 'axios'

//异步的action不会放入homeSlice中，而是用createAsyncThunk单独创建并导出
export const getHomeMultidataAction = createAsyncThunk("home/Multidata",async (payload,extraInfo)=>{
    const res = await axios.get("http://123.207.32.32:8000/home/multidata")
    return res.data.data
})
```
#### 当createAsyncThunk创建出来的action被dispatch时，会存在三种状态。
* pending：action被发出，但是还没有最终结果
* fulfilled：获取到最终的结果（有返回值的结果）
* rejected：执行过程中有错误或者抛出了异常
#### 通过extraReducer监听异步action的状态.
```jsx
const homeSlice = createSlice({
    name:"home",
    initialState:{
        banners:[],
        recommends:[]
    },
    reducers:{
        changeBanners(state,{payload}){
            state.banners = payload
        },
        changeRecommends(state,{payload}){
            state.recommends = payload
        }
    },
    extraReducers:{
        // [getHomeMultidataAction.pending](state,action){
        //     console.log("getHomeMultidataAction pending")
        // },
        /** 
         * 异步action返回的数据传入监听函数的action.payload中。
        */
        [getHomeMultidataAction.fulfilled](state,action){
            // 数据同上述reducer一样直接修改即可。
            state.banners = action.payload.banner.list
            state.recommends = action.payload.recommend.list
        },
        // [getHomeMultidataAction.rejected](state,action){
        //     console.log("getHomeMultidataAction rejected")
        // }
    }
})

```

## Redux Toolkit的数据不可变性

### 在React开发中，我们总是会强调数据的不可变性：
* 无论是类组件中的state，还是redux管理的state。
* 事实上在整个js编码过程中，数据的不可变性都是非常重要的。
### 所以我们在前面经常会进行浅拷贝来完成某些操偶做，但是浅拷贝事实上也是存在问题的：
* 比如过大的对象，进行浅拷贝会造成性能的浪费；
* 比如浅拷贝后的对象，在深层改变时，依然会对之前的对象产生影响。
### 事实上Redux Toolkit使用了immerjs的一个库来保证数据的不可变性。
* immerjs底层使用Persistent Data Structure(持久化数据结构或一致性数据结构)
  * 用一种数据结构来保存数据
  * 当数据结构被修改时，会返回一个对象，但是新的对象会尽可能的利用之前的数据结构而不会对内存造成浪费。
