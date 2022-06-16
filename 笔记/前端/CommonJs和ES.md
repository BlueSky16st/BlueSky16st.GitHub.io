---
title: CommonJS和ES
date: 2022-06-13 14:21
---

>  文章: [https://cloud.tencent.com/developer/article/1884093](https://cloud.tencent.com/developer/article/1884093)

## 早期
早期的 JS 开发存在两个大问题:
- 全局污染
    不同 JS 文件定义相同变量名, 在同一 HTML 文件中同时引入 JS 时, 会以最后一次定义变量的值为准, 导致在这之前的 JS 文件使用到这个变量时出现意外的问题
- 依赖管理
    JS 的执行顺序是以 script 标签的前后顺序进行的, 如果不同 JS 间有依赖关系, 将不好处理
    下层 JS 可以调用上层 JS 方法, 但上层 JS 无法调用下层 JS 方法

## CommonJS
>  NodeJS 借鉴了 CommonJS 的 Module, 实现了良好的模块化管理

### 应用场景
- NodeJS
- Browserify
- Webpack, 打包工具对 CommonJS 的支持和转换, 前端可以在编译之前使用 CommonJS 进行开发

### 特点
- 在 CommonJS 中每一个 JS 文件都是一个单独的模块, 称为 `Module`
- 模块中, 包含 CommonJS 规范的核心变量: `exports, module.exports, require`
- `exports`和`module.exports`可以负责对模块中的内容进行导出
- `require`函数可以导入其它模块(自定义模块, 系统模块, 第三方库模块)中的内容

### 文件加载流程
``` JavaScript
const fs =  require('fs')    // 核心模块
const sayName = require('./hello.js')    // 文件模块
const crypto = require('crypto-js')    // 第三方自定义模块
```

##### 加载标识符原则
- 像`fs, http, path`等标识符, 会被作为 NodeJS 核心模块
- `./`和`../`作为相对路径的文件模块, `/`作为绝对路径的文件模块
- 非路径形式也非核心模块的, 将作为自定义模块

##### 核心模块的处理
核心模块的优先级仅次于缓存加载, 在`Node`源码编译中, 已被编译成二进制代码, 加载过程中速度最快

##### 路径形式的文件模块处理
以`./`, `../,`和`/`开始的标识符, 会被当作文件模块处理, `require()`方法会将路径转换成真实路径, 并以真实路径作为索引, 将编译后的结果缓存起来, 使后续加载的时候会更快

##### 自定义模块处理
一般指非核心的模块, 模块可能是文件或包, 它的查找会遵循以下原则:
- 在当前目录下的`node_modules`目录查找
- 如果没有, 在父级目录的`node_modules`目录查找, 如果没有, 就继续查找上一层的`node_modules`目录
- 沿着路径向上递归, 直到根目录下的`node_modules`目录
- 如果查找到相应的模块, 会查找该模块目录下的`package.json`文件, 找到文件中的`main`属性指向的文件, 如果没有`package.json`文件或`main`属性, 则默认依次查找`index.js`, `index.json`, `index.node`

### require 模块引入与处理

##### module 和 Module
- module: 在 NodeJS 中每一个 JS 文件都是一个 module, module 上保存了 exports 等信息之外, 还保存了一个 `loaded` 变量来表示该模块是否被加载
- Module: NodeJS 在运行之后, 会使用`Module`缓存每一个模块加载的信息

##### require 的流程
- require 接收文件标识符参数, 分析定位文件, 然后会从 `Module`上查找有没有缓存, 如果有缓存, 则返回缓存内容(避免了重复加载和循环引用的问题)
- 如果没有缓存, 会创建一个`module`对象, 缓存到`Module`上, 然后执行文件, 加载完文件后, 将`loaded`属性设置为`true`, 然后返回`module.exports`对象

另外, `exports`和`module.exports`持有相同引用, 因为最后导出的是`module.exports`, 所以对 `exports` 进行赋值会导致 `exports` 操作的不再是`module.exports`的引用

##### require 动态加载
`require` 本质上就是一个函数, 可以在任意的上下文, 动态加载模块

### exports 和 module.exports
##### exports
``` JavaScript
exports.name = ''
exports.say = function() {
    ...
}
```
导出后得到的值: ` {name: '', say: [Funciont]} `

exports 就是传入到当前模块内的一个对象, 本质上就是`module.exports`
但是无法通过`exports={}`进行赋值, `exports, module 和 require`都是作为形参传入到 js 模块中, 直接修改`exports`变量, 相当于重新赋值了形参, 此时会重新赋值一份新的值, 而不会再引用原来的形参

##### module.exports
``` JavaScript
module.exports = {
    name: '',
    say() {
        ...
    }
}
```
`module.exports`本质上就是`exports`, `module.exports`也可以单独导出一个函数或者一个类
``` JavaScript
module.exports = function() {
    ...
}
```

##### 注意点
- 在 JS 文件中, 只选择`exports`和`module.exports`两者之一, 如果同时存在, 有可能会造成覆盖的问题
- 如果不想在 CommonJS 中导出对象, 而只导出一个类或者函数, 使用`module.exports`比较方便
- `exports`会被初始化为一个对象, 也只能在这个对象上绑定属性, 而`module.exports`可以自定义导出对象外的其它类型元素
- `module.exports`导出非对象属性时, 如在循环引用的情况下, 有可能造成属性丢失的情况

## ES Module
>  从 ES6 开始, JavaScript 才真正意义上有自己的模块化规范

- 借助`ES Module`的静态导入导出的优势, 实现了`tree shaking`
- `ES Module`还可以`import()`懒加载方式实现代码分割
- 在`ES Module`中, `export`用来导出模块, `import`用来导入模块

### 导出 export 和导入 import
>  所有通过 export 导出的属性, 在 import 中可以通过结构的方式解构出来

##### 导出 export
导出模块:
``` JavaScript
// a.js
const name = ''
const author = ''
export { name, author }
export const say = function() {
    ...
}
```

导入模块:
``` JavaScript
import { name, author, say } from './a.js'
```

注意点:
- export {}, 与变量名绑定, **命名导出**
- import {} from 'module', 导入`module`的命名导出
- import {} 内部的变量名称, 要与 export {} 完全匹配

##### 默认导出 export default
导出模块:
``` JavaScript
// a.js
const name = ''
const author = ''
const say = function() {
    ...
}

export default = {
    name,
    author,
    say
}
```

导入模块:
``` JavaScript
import obj from './a.js'
console.log(obj) // { name: '', author: '', say: [Function] }
```

注意点:
- `export default ...`导入 module 的**默认导出**, 可以导出函数, 属性方法或者对象
- 对于引入默认导出的模块, `import [any] from 'module'`, `[any]`可以是自定义名称

##### 混合导入, 导出
ES6 Module 可以使用`export default`和`export`导入多个属性

导出模块:
``` JavaScript
// a.js
export const name = ''
export const author = ''
export default function say() {
    ...
}
```

导入模块:
``` JavaScript
// 第一种
import sayFunc, { name, author as authorData } from './a.js'
console.log(
    sayFunc,    // Function say()
    name,    // name: ''
    authorData    // author: ''
)

// 第二种
import sayFunc, * as obj from './a.js'
console.log(
    sayFunc,    // Function say()
    obj    // { name: '', author: '' }
)
```

##### 重定向导出
可以将当前模块作为中转站, 一方面引入`module`内的属性, 然后把属性再导出
``` JavaScript
// 第一种, 重定向导出 module 中的所有导出属性, 但不包括 module 内的 default 属性
export * from 'module'

// 第二种, 从 module 中导入 name, author 等属性, 再以相同的属性名导出
export { name, author, ..., say } from 'module'

// 第三种, 从 module 中导入 bookName 并重命名为 name 后导出
export { bookName as name, bookAuthor as author, ..., say } from 'module'
```

##### 无需导入模块, 只运行模块
``` JavaScript
// 执行 module 不导出值, 多次调用时, 只会执行一次
import 'module'
```

##### 动态导入
``` JavaScript
// 会动态返回一个 Promise, 为了支持这种方式, 需要在 Webpack 中做相应的配置处理
const promise = import('module')
```

### ES6 Module 特性

##### 静态语法
ES6 Module 的导入和导出是静态的, `import`会**自动提升**到代码的顶层, `import, export`不能放在块级作用域或条件语句中
静态语法在编译过程中确定了导入和导出的关系

##### 执行特性
CommonJS 模块同步加载并执行模块文件
ES6 模块提前加载并执行模块文件, ES6 模块在预处理阶段分析模块依赖, 在执行阶段执行模块

##### 导出绑定
**不能修改 import 导入的属性**

- 被导入的模块运行在严格模式下
- 被导入的模块是只读的, 可能理解为 const 装饰, 无法被赋值
- 被导入的模块的变量是与原变量绑定/引用的, 可以理解为 import 导入的变量无论是否为基本类型都是引用传递

##### import()动态引入

``` JavaScript
// a.js
export const name = ''
export default function say() {
    ...
}
```

``` JavaScript
// main.js
setTimeout(() => {
    const result = import('./a.js')
    result.then(res => {
        console.log(res)    // { default: [Function], name: '', __esModule: true }
    })
}, 0)
```

- `import()`可以动态加载模块
- `import()`返回一个`Promise`对象, 返回的`Promise`的 `then` 成功回调中, 可以获取模块的加载成功信息, 其中`__esModule`为 ES Module 的

##### import()的用途
- 动态加载
`import()`可以放在条件语句或者函数执行上下文中

- 懒加载
`import()`可以实现懒加载, 如 vue 中的路由懒加载
``` JavaScript
[
  {
    path: '/',
    name: 'Index',
    meta: {title: '后端便捷工具'},
    component: () => import('@/views/Index/Index.vue'),
  }
]
```

- React 中动态加载
``` JavaScript
// 该组件是动态加载的
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // 显示 <Spinner> 组件直至 OtherComponent 加载完成
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```

- `import()`实现动态加载, 能够实现代码分割, 避免一次性加载大量 JS 文件

##### tree shaking
Webpack 中的Tree Shaking 是用来尽可能地删除没有被使用过的代码

## 总结
### CommonJS
- CommonJS由 JS 运行时实现
- CommonJS 是单个值导出, 本质上导出的就是 exports 属性
- CommonJS 可以动态加载, 对每一个加载都存在缓存, 可以有效的解决循环引用问题
- CommonJS 模块同步加载并执行模块文件

### ES Module
- ES6 是静态的, 不能放在块级作用域内, 代码发生在编译时
- ES6 的值是动态绑定的, 可以通过导出方法修改, 可以直接访问修改结果
- ES6 可以导出多个属性和方法, 可以单个导入导出, 混合导入导出
- ES6 模块提前加载并执行模块文件
- ES6 导入模块在严格模式下
- ES6 的特性可以很容易实现 Tree Shaking 和 Code Splitting












