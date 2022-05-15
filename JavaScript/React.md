# React

## 定义

> 用于构建用户界面的JavaScript库
>
> *a javascript library for building user interfaces*
>
> React是声明式方法（declarative），通过声明能快速搭建起交互式的UI，并且当变化时，也能够自动更新和重新渲染变化
>
> React是基于**组件（component）**的方法，将行为封装到基于组件的最小单元中
>
> React没有啥局限性，能够应用在任何技术栈上

React是依赖Node.js和NPM的

JavaScript一开始是被设计用在浏览器上的脚本代码，但是它现在的发展远远超过浏览器范围，走向了全栈（full-stack），这个其中Node.js功不可没，是它让JavaScript从浏览器走向了桌面

JavaScript工程中的代码主要由：`package.json`和js文件夹组成。

`package.json`的作用：

1. 指明在你工程中需要依赖的库有哪些
2. 指定库的特定依赖的版本——为了在其他的电脑上能够完全复现你的代码效果

React相关关键点：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220512212544513.png" alt="image-20220512212544513" style="zoom: 50%;" />

## node.js

### 1. 架构

Node.js是基于Chrome浏览器构建的JavaScript运行时引擎上的，而Chome V8的JavaScript运行时引擎已经从浏览器移植到桌面运行，所以Node.js已经能够JS支持在桌面上运行了

Node.js是基于事件驱动的无阻塞IO模型，所以能够高效的在桌面上运行和同步JavaScript代码

Node.js的架构：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220511155042658.png" alt="image-20220511155042658" style="zoom:50%;" />

### 2. 安装

下载，无脑安装，安装完成之后打开命令行验证：

1. `node -v`
2. `npm -v`

能够显示出对应的版本，即可。

创建工程：

1. 创建一个文件夹，

2. 输入==**`npm init`**==，就会开始进行配置：

   package_name（不能是中文）、版本号、对工程的描述、入口文件（默认是index.js，当然可以自定义，例如index.html）、指令、对应的git工程地址、关键词、作者等，这些都配置完成后，会需要用户查看效果，如果同意就会在当前目录下创建一个`package.json`文件

   <img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220512094715931.png" alt="image-20220512094715931" style="zoom:67%;" />

   ```json
   {
     "name": "test",
     "version": "1.0.0",
     "description": "",
     "main": "index.html",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"		// 注意这个需要修改，否则会报错
     },
     "author": "",
     "license": "ISC"
   }
   ```

3. 安装NPM组件，选择的是轻量级的node服务器lite-server，它仅支持web-app，当文件内容变化时，它能够自动更新

   `npm install lite-server --save-dev`

   就会开始漫长的安装（下载速度不稳定），下载完成后，会发现多了一个文件夹，文件夹里内容很多（暂时不需要关注），并且发现packagen.json有些变化

   ```json
   {
     "name": "test",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "author": "",
     "license": "ISC",
     "devDependencies": {				// 依赖库的版本
       "lite-server": "^2.6.1"
     }
   }
   ```

4. 修改脚本：

   ```json
   "scripts": {
       "dev": "lite-server"
   },
   ```

   然后运行

   ```cmd
   npm run dev		// 这里的dev就是对应脚本中的内容
   ```

   注意：不能直接在命令行中输入`npm run lite-server`会报错（目前不知道原因，估计是npm run后面需要跟script里面的key）

   然后就能启动lite-server

## why需要react

首先明确两个目的性问题：

1. 什么是JavaScript框架和库
2. 为什么需要JavaScript框架和库，react就是一个框架，为啥不直接用原生的JavaScript（并且还有更多的框架，例如著名的Vue、Angular）

首先，当需要对DOM进行复杂的操作时，例如从服务端获取数据然后更新DOM，用原生的JavaScript和jQuery是十分复杂的—— that is where the JavaScript libraries and frameworks shine.——前端的应用架构常见的有MVC、MVVM等，它主要是帮助我们对整个应用有一个很好的架构，那么会发现标准的模式、标准的API能够大量复用，所以一个**被良好定义的有标准功能的实现方法集合被抽离出来就组成了库**——用户可以直接调用它们来快速完成构建，并且也能够实现代码复用，实现模块化构建

1. 框架和库的区别

   框架：抽象，提供一系列通用方法，通过实现它们来自定义具体的功能——框架主要提供的是有特定功能的可重用环境，用户可以在里面添加自定义的代码，当然由于它是一个应用的骨架，所以必然存在一定的限制，和自定义范围的制约，eg：Angular

   库：一组方法，导入之后，简单的调用暴露出来的外部方法即可，主要是一些有用方法的集合，eg：jQuery

   （框架反过来，它会去调用你自定义的方法，从而实现不一样的“人”的行为）

## React安装

`npm install -g create-react-app`，全局安装react环境

`create-react-app my-react-app`，就会在当前路径下创建一个`my-react-app`文件夹，里面存放react相关的文件，cd进入，输入命令行：`npm start`（这个是`package.json`中已经默认写好的脚本）

然后输入网址（正常会自动打开浏览器）：`localhost:3000`，就能看到里面的内容

ps：换下载源（默认的下载源实在是太慢了，经常报timeour错误）

通过`npm get registry`能够获取下载的来源，默认是：`https://registry.npmjs.org/`

可以更换成

## 配置react应用

课程中是使用Bootstrap来修改配置的

输入如下命令：

1. `npm install bootstrap --save`
2. `npm install reactstrap --save`
3. `npm install react-popper --save`

然后在`index.js`中导入文件：`import 'bootstrap/dist/css/bootstrap.min.css';`

然后在`App.js`中导入文件：`import { Navbar, NavbarBrand } from 'reactstrap'`

```javascript
function App() {
  return (
    <div className="App">
      {/* <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <h1>test</h1>
      </header> */}
	// 里面是需要用到导入的 Navbar 和 NavbarBrand
      <Navbar dark color="primary">
        <div className="container">
          <NavbarBrand href='/'>week1</NavbarBrand>
        </div>
      </Navbar>
    </div>
  );
}
```



## React工程脚手架分析

### 1. 找到入口

```javascript
// index.js
// 获取html中的root，并创建ReactDOM的根节点
const root = ReactDOM.createRoot(document.getElementById('root'));	
root.render(			// root里面渲染App里面的内容
  <React.StrictMode>
    <App />			// 获取App里面的内容，将其导入
  </React.StrictMode>
);

// App.js
function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <h1>test</h1>
      </header>
    </div>
  );
}

// ../public/index.html
...
<body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>				// 这边就有一个根节点
</body>
```

所以是从public/index.html开始，然后使用index.js获取root节点，将内容添加到该节点中

## JSX

在React代码中，可以看到很多类似于HTML的代码，这些实际上是JSX代码，只不过是类似HTML的形式，官方推荐在React中配合使用JSX，它可以很好的描述UI应该呈现的交互的本质形式，并且有JavaScript的全部功能

JSX是JavaScript的扩展语法，它能够使用户用类似于HTML的语法来表达react的元素（React element）

```jsx
const element = <h1>hello world</h1>;
```

### why使用JSX

React认为渲染逻辑和UI的其他内在逻辑耦合，eg：某些时刻状态发生变化需要通知到UI、UI中展示准备好的数据

所以没有必要将标记和逻辑进行分离，而是可以一起放在component的单元中，从而实现关注点分离

### 基本语法

```jsx
const name = "amyZhoucc"
const element = <h1>hello, {name}</h1>
```

可以在大括号内存放任何的JavaScript语法

JSX转换成JavaScript的效果：

<img src="C:\Users\surface\AppData\Roaming\Typora\typora-user-images\image-20220512202113636.png" alt="image-20220512202113636" style="zoom:67%;" />

## React Component

一个component可以作为一个整体被看待，它将集合多个React元素，通过对页面上面模块的划分来进行组合，比如说一个特殊的按钮，可以成为一个component，并且该component还能被复用

——所以，以component为单位，可以将整个页面的UI划分成很多较小的可复用性高的组件

注意：**组件名称必须以大写字母开头**

### 1. 自定义

必须要实现的方法：constructor构造方法和render方法

```javascript
class Menu extends Component{			// 继承自类Component
    constructor(props){
        super(props);			// 构造方法，调用父构造方法即可
    }
    render(){
        return(
        	....
        );
    }
}
export default Menu;
```

——上面就是一个自定义component的框架

```javascript
import React from "react";			// React 和 Component 必须要导入的
import { Component } from "react";
import { Media } from "reactstrap";		// 需要导入的包

class Menu extends Component{
    constructor(props){     // 构造方法
        super(props);
        this.state = {			// 变量一般都写在这里
            dishes: [               // dishes是数组，里面每个元素都是一个字典类型的
            {
                id: 0,
                name:'Uthappizza',
                image: 'assets/images/uthappizza.png',
                category: 'mains',
                label:'Hot',
                price:'4.99',
                description:'...'},
             {...},
             {...},
             {...}
             ],
        }
    }

    render(){           // 必须要实现的一个方法
        // 对数组里面的每个元素遍历，并且元素名为dish，并进行后面的操作
        const menu = this.state.dishes.map(dish => {        
            // 里面是JSX语法
            return (    
                // key是用来唯一确定一个元素的，从而方便定位，已经更新时的目标确定
                <div key={dish.id} className="col-12 mt-5">
                    <Media tag="li">
                        <Media left middle>
                            <Media object src={dish.image} alt={dish.name}></Media>
                        </Media>
                        <Media body className="ml-5">
                            <Media heading>{dish.name}</Media>
                            <p>{dish.description}</p>
                        </Media>
                    </Media>
                </div>  // 大部分都是Media的内置语法
            );
        });
        return(
            <div id="container">
                <div className="row">
                    <Media list>		// 里面接收一个list对象
                        {menu}
                    </Media>
                </div>
            </div>
        );
    }
}
export default Menu;            // 导入该类
```

### 2. State

是私有变量，它完全受控于当前的组件。只有控件才能够存储state

当然，state的信息可以通过prop很方便的传递给子组件

state一般是在constructor方法中定义的

```javascript
this.state = {
	dishes: [....],
}
```

而**如果需要更新state，不能直接赋值，而是需要调用方法**：

```javascript
this.setState({
    dishes: dish
})
```

不想修改的不用写，因为它是以merge的形式



参考：

1. https://zh-hans.reactjs.org/（还是看不惯英文）
2. https://www.coursera.org/learn/front-end-react/lecture/l2fQJ/（coursera上的课程，全英文，慢慢学）