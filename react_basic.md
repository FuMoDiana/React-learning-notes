# React基础

## 开发前的准备

### 开发依赖库
1. react:包含react所必须的核心代码
2. react-dom：react渲染在不同平台上所需要的核心代码
3. babel：将jsx转换成react的工具

### 添加依赖
1. cdn引入

    ````html
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>

    ````
2. 下载引入
3. npm下载引入 

### 在html中使用react语法
1. 在scrpt标签中编写jsx语法，要记得加type="text/babel"
    ````html
    <script type="text/babel">
        // ReactDOM.render(
        // <h2>hello world</h2>,document.querySelector('#root'))//16的写法

        const root = ReactDOM.createRoot(document.querySelector('#root'))
        root.render(<h2>hello world</h2>)
        // const app = ReactDOM.createRoot(document.querySelector('#app'))
        // app.render(<h2>app view</h2>)
    </script>
    ````
2. react16和react18的渲染el写法不同，react依赖reactRoot方法。
3. 如果原本root节点中有内容，react渲染的内容会覆盖掉原本内容
4. 一个简单的案例
    ````html
        <script type="text/babel">
            const root = ReactDOM.createRoot(document.querySelector('#root'));
            //1.定义文本变量
            let message = 'Hello World'
            rootRender();
            //2.监听按钮点击
            function btnClick(){
                //1.修改数据
                message = "Hello React";

                //2.重新渲染界面
                rootRender();
            }
            //3.将渲染过程封装成函数
            function rootRender(){
                root.render((
                    <div>
                    <div>{message}</div>
                    <button onClick={btnClick}>修改文本</button>
                    </div>
                ))
            }
            
        </script>
    ````
### Hello React-组件化开发
如果整个逻辑可以看作一个整体，那么我们就可以将其封装成一个组件
1. root.render参数是一个html元素或者一个组件，因此我们可以将之前的业务逻辑封装到一个组件中，然后传入到ReactDOM.render函数中的第一个参数.
    ````javascript
        //加载组件
        //一个业务逻辑完整的组件应该有data，方法以及渲染函数
        root.render(<APP />)
    ````
2. 类组件和函数式组件
    1. 类组件的定义与使用，必须给类组件中的自定义方法**主动绑定this**，【vue的内部会主动绑定this，react不会】因此外部调用类方法会默认将this指向系统，但**babel转换jsx会使用严格模式**,使得this指向undefined。
        ````javascript
                class App extends React.Component{
            //data
            constructor(){
                super()
                //参与页面更新的动态数据需要放在state中
                this.state = {
                    message:"hello world",
                    name:'cw',
                    age:18,
                }
                //如果不在使用方法的时候绑定this，就在构造函数中绑定this
                //this.btnClick = this.btnClick.bind(this);
            }
            
            //function(实例方法)
            btnClick(){
                //不绑定this的话，读取不到app的this，默认是还没渲染上去
                // console.log(this)
                /** 
                 * 内部执行了两件事（setState是super来的）
                 * 1.将state中的message值修改掉
                 * 2.自动重新执行render函数
                */
                this.setState({
                    message : "hello React"
                })
            }
            //渲染内容 render方法
            render(){
                return (
                    <div>
                        <h2>{this.state.message}</h2>
                        <button onClick={this.btnClick.bind(this)}>修改文本</button>    
                    </div>
                )
            }

        }
        //将组件渲染到页面上
        const root = ReactDOM.createRoot(document.querySelector("#root"));
        root.render(<App></App>)
        ````
    
## JSX语法
### 认识jsx
   jsx可以称之为js的语法扩展，在很多地方称之为JavaScript XML，因为看起来就是一段XML语法，它用于描述我们的UI界面，并且其可以和JavaScript融合在一起使用。
### 为什么React选择了JSX
React认为渲染逻辑本质上与其他UI逻辑存在内在耦合
* 比如UI需要绑定事件
* 比如UI中需要展示数据状态
* 比如在某些状态发生改变时，需要改变UI
因此react没有将标记分离到不同的文件中，而是将他们组合到了一起，这个地方就是组件。
### JSX的书写规范
* jsx结构中只能有一个根元素（即最外层需要被包裹）
* jsx结构通常会包裹一个()，将jsx当作一个整体
* jsx可以是单标签，也可以是双标签，但单标签必须以"/>"结尾

### jsx中的注释编写
1. js代码部分直接双杠后注释
2. jsx设置的html部分
    ````jsx
    return (
        <div>
            {/*JSX的注释写法*/}
            <h2>{message}</h2>
        </div>
    )
    ````
### jsx中插入类型的介绍
* 情况一：当变量是Number,String,Array类型时，可以直接显示
    ````jsx
        runder{
            let names = ["abc","cba","nba"];
            return (
                <div>
                    {/*String直接显示，Number同理*/}
                    <h2>{message}</h2>
                    {/*Array中的元素被取出并拼接成一个整体*/}
                    <h2>{names}</h2>
                </div>
            )
        }
    ````
* 情况二：undefined,null,boolean类型,不报错也不显示，如果一定要显示，需要将其转换成字符串
    ````jsx
    runder{
            let aaa = undefined;
            let bbb = null;
            let ccc = true;
            return (
                <div>
                    <h2>{String(aaa)}</h2>
                    <h2>{bbb + ''}</h2>
                    <h2>{ccc.toString()}</h2>
                </div>
            )
        }
    ````
* **情况三：Object类型不能作为子元素进行显示**
    ````jsx
        render(){
            let friend = {
                name:"why",
                age:18
            }

            return(
                <div>
                    <h2>{friend}</h2>
                    {/*报错：Objects are not valid as a React child*/}
                    {/*如果要显示，必须明确显示对象内的某个key/value*/}
                </div>
            )
        }
    ````
* 情况四：插入js的表达式/三元运算符/调用方法获取结果
    ````jsx
        getEles(array){
            let eles = array.map(item=><li>{item}</li>);
            return eles
        }
        render(){
            let firstname = "aaa";
            let lastname = "bbb";
            let age = 20
            let movies = ["星际穿越","独行月球",""]
            return(
                <div>
                    {/*表达式*/}
                    <h2>{firstname + " "+ lastname}</h2>
                    {/*三元运算*/}
                    <h2>{age >= 18 ? "成年" : "未成年"}</h2>
                    {/*方法调用*/}
                    <h2>{getEles(movies)}</h2>
                    <h2>{array.map(item=><li>{item}</li>)}</h2>
                </div>
            )
        }
    ````
### jsx绑定属性
比如元素都会有title属性，img元素会有src属性，a元素会有href属性，元素可能需要绑定class以及一些内联样式等等。
* 基础绑定：直接{}绑定，**没有" "号包裹**
    ````jsx
    class App extends React.Component{
        constructor(){
            super();
            this.state = {
                message:'hello world',
                title:'哈哈哈',
                imgUrl:'https://tse1-mm.cn.bing.net/th/id/OET.3ebd297b197f4a79b6950b52cb9faa5d?w=272&h=135&c=7&rs=1&o=5&dpr=1.3&pid=1.9',
                sreachUrl:'https://cn.bing.com/'
            }   
        }
        render(){
            return (
                <div>
                    {/*1.绑定全局title属性*/}
                    <h2 title={this.state.title}>{this.state.message}</h2>
                    {/*2.绑定imgsrc*/}
                    <img src={this.state.imgUrl} />
                    {/*绑定链接*/}
                    <a href={this.state.sreachUrl}>bing Sreach</a>
                </div>
            )
        }
    }

    ````
* 绑定class
    react可以直接识别class，但是会报警告，因此绑定class最好使用**classname={}**
    ````jsx
    render(){
        //绑定class
        let name = 'documentTitle';
        let isActive = true;
        let className = `abc cba ${isActive ? 'active': '' }`
        return (
            <div>
                {/*1.基本绑定*/}
                <h2 classname={tname}>基本绑定</h2>

                {/*2.字符串拼接,模板字符串*/}
                <h2 classname={className}>字符串拼接</h2>

                {/*3.将所有的class放进数组*/}
                <h2 classname={classList.join(' ')}>数组</h2>

                {/*4.第三方库，classnames -> npm install classnames*/}
            </div>
        )
    }
    ````
* JSX绑定事件
   1. react事件的命名采用小驼峰式(camCase)而不是纯小写
   2. 我们需要通过{}传入一个事件处理函数，这个函数会在发生时被执行。
   3. this绑定的问题，在事件执行后，我们可能需要获取当前类对象中的相关属性，这个时候需要用到this
        * this绑定的三种方法
         ````jsx
            class App extends React.Component{
            constructor(){
                super();
                this.state = {
                    message:'hello world',
                    counter:100
                }
                //方式一：在构造函数处使用bind进行this绑定
                this.btn1Click = this.btn1Click.bind(this);
            }
            //方式1：进行bind绑定
            btn1Click(){
                console.log("btn1Click",this)
                this.setState({
                    counter:this.state.counter+1
                })
            }
            //方式二：es6 class fields字段
            //箭头函数的this是上层作用域的this
            btn2Click = ()=>{
                console.log("btn2Click",this)
            }
            //方式三：传入一个箭头函数，在箭头函数内进行btn3Click执行。
            btn3Click(){
                console.log("btn3Click",this)
            }
            render(){
                return (
                    <div>
                        {/*此处的this.btnClick并不是显式调用，只是将btnClick方法取出，将地址传递*/}
                        <h2>{this.state.counter}</h2>
                        {/*this绑定方法一：bind绑定*/}
                        <button onClick = {this.btn1Click}>btn1</button>
                        {/*this绑定方法二：class fields*/}
                        <button onClick = {this.btn2Click}>btn2</button>
                        {/*this绑定方法三：直接传入一个箭头函数,在箭头函数中进行函数执行*/}
                        <button onClick = {()=>this.btn3Click()}>btn3</button>
                    </div>
                )
            }
        }
         ````
         * 事件绑定中参数传递问题
          ````jsx
            class App extends React.Component{
                constructor(){
                    super();
                    this.state = {
                        message:'hello world'
                    }
                }
                btnClick(event){
                    console.log("btnClick",event)
                }
                btn2Click(event,name,age){
                    console.log("btn2Click",event,name,age)
                }
                render(){
                    return (
                        <div>
                            {/*event参数传递,原生js中方法调用会默认给方法传递一个event参数，react中同理 */}
                            <button onClick={this.btnClick.bind(this)}>button1</button>
                            <button onClick={(e)=>{this.btnClick(e)}}>button2</button>

                            {/*额外的参数传递*/}
                            <button onClick={(event)=>this.btn2Click(event,"why",18)}>btn3</button>
                        </div>
                    )
                }
            }
          ````

* JSX条件渲染
    1. 条件判断
    2. 三元运算符判断
    3. &&逻辑与运算
    ````jsx
        class App extends React.Component{
                constructor(){
                    super();
                    this.state = {
                        message:'hello world',
                        isReady:false,
                        friend:{
                            name:'kobe',
                            desc:"the sun of night four"
                        }
                    }
                }
                changeReady(){
                    this.setState({
                        isReady:!this.state.isReady
                    })
                }
                render(){
                    const {friend,isReady} = this.state
                    let showElement = null
                    //方式一：if-else条件判断
                    if(isReady){
                        showElement = <h2>ready go</h2>
                    }else{
                        showElement = <h2>please ready</h2>
                    }
                    return (
                        <div>
                            {/*方式一：条件判断，逻辑较多*/}
                            {showElement}
                            <button onClick={()=>this.changeReady()}>changeReady</button>
                            {/*方式二：三元运算符：逻辑较简单*/}
                            <div>{isReady? <button>ready go</button>:<h3>please ready</h3>}</div>
                            {/*方式三：&&逻辑与运算,friend为undefined时不显示*/}
                            {/*使用场景：当某个值有可能为undefined时使用*/}
                            <div>{friend && <div>{friend.name+' is '+friend.desc}</div>}</div>
                        </div>
                    )
                }
            }
    ````
* JSX列表渲染
   react中没有v-for指令，需要我们通过js代码的方式组织数据，转成JSX。使用map,fliter,slice等数组高阶函数对列表进行处理即可
   ````jsx
        render(){
        const {students} = this.state
        return (
            <div>
                <h2>Student Data List</h2>
                <div className="list">
                {
                    //分数大于60的人进行展示2个
                    students.filter(item=>item.score>60).slice(0,2).map((item,index)=>{
                        return  (
                            <div className="item" key={item.id}>
                                <h2>id:{item.id}</h2>
                                <h2>name:{item.name}</h2>
                                <h2>score:{item.score}</h2>
                            </div>
                        )
                    })
                }
                </div>
            </div>
        )
    }
   ````
### jsx的本质
jsx仅仅只是React.createElement(component,props,...children)函数的语法糖。**jsx => 虚拟dom树(ReactElement对象) => 真实dom**
  * 参数一：type,当前ReactElement的类型，如果是标签元素，那么字符串表示'div'，如果是组件元素，那么直接使用组件的名称。
  * 后面的参数如果没有的话直接传递null
  * babel转换后的代码
    ````jsx
    <div className="shopping-list">
        <h1>Shopping List for {props.name}</h1>
        <ul>
            <li>Instagram</li>
            <li>WhatsApp</li>
            <li>Oculus</li>
        </ul>
    </div>;
    {/*等同于*/}
    /*#__PURE__*/React.createElement("div", {className: "shopping-list"}, 
    /*#__PURE__*/React.createElement("h1", null, "Shopping List for ", props.name), 
    /*#__PURE__*/React.createElement("ul", null, /*#__PURE__*/React.createElement("li", null, "Instagram"), /*#__PURE__*/React.createElement("li", null, "WhatsApp"), /*#__PURE__*/React.createElement("li", null, "Oculus")));
    ````
  * **虚拟dom的作用**
    * 作用一：更新数据的时候，先在虚拟dom上更新，然后快速通过diff算法进行新旧dom的比较，锁定更新的地方，只重新渲染更新的部分。
    * 作用二：虚拟dom本质上是一个js对象，在React中通过createElement方法渲染在页面上，那么同理可得，虚拟dom也能通过其他方法渲染在其他端(ios,Android的控件)的页面上。达到跨平台开发的作用。
    * 作用三：帮助我们从**命令式编程=>声明式编程**,在这个理念中，UI以一种理想化/虚拟化的方式保存在内存中，并且它是一个相对简单的js对象。我们可以通过root.render让虚拟DOM和真实DOM同步起来，这个过程叫做**协调(Reconciliation)**。






