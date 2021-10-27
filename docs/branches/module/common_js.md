# 浅谈commonJs

## 简介

JS是一种直译式脚本语言，也就是一边编译一边运行，所以没有模块的概念。CommonJS是为了完善JS在这方面的缺失而存在的一种规范。CommonJS不是一个库，他只是一个规范。

CommonJS定义了两个主要概念：

- `require`函数，用于导入模块
- `module.exports`变量，用于导出模块

注意：浏览器都不支持这两个关键字，如果一定要在浏览器上使用CommonJs，那么就需要一些编译库，比如[browserify](http://browserify.org/)来帮助我们将CommonJs编译成浏览器支持的语法，实现require和exports。

但是在nodejs中就可以直接使用require和exports这两个关键词来实现模块的导入和导出。Node应用由模块组成，采用 CommonJS 模块规范。每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。

CommonJS规范规定，每个模块内部，`module`变量代表当前模块。这个变量是一个对象，它的`exports`属性（即`module.exports`）是对外的接口。加载某个模块，其实是加载该模块的`module.exports`属性。

```javascript
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

上面代码通过`module.exports`输出变量`x`和函数`addX`。

`require`方法用于加载模块。

```javascript
var example = require('./example.js');

console.log(example.x); // 5
console.log(example.addX(1)); // 6
```

CommonJS模块的特点如下。

- 所有代码都运行在模块作用域，不会污染全局作用域。
- 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了（缓存在`require.cache`之中），以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。
- 模块加载的顺序，按照其在代码中出现的顺序。





require导入，先解析路径，解析路径获取到模块内容：

- 如果是核心模块，比如`fs`，就直接返回模块

- 如果是带有路径的如'/''或者./'等等，则拼接出一个绝对路径，然后先读取缓存require.cache再读取文件。如果没有加后缀，则自动加后缀然后一一识别。

  - `.js` 解析为JavaScript 文本文件
  - `.json`解析JSON对象
  - `.node`解析为二进制插件模块

- 首次加载后的模块会缓存在`require.cache`之中，所以多次加载`require`，得到的对象是同一个。

- 在执行模块代码的时候，会将模块包装成如下模式，以便于作用域在模块范围之内。(function(exports, require, module, __filename, __dirname) { 

  ​		// 模块的代码实际上在这里 

  });

示例：

utils.js

```javascript
let count=0
function addCount(){
    count++
}
module.exports={count,addCount}
```

引入let {count,addCount}=require("./utils")；

然后根据require执行代码时需要加上的，那么实际上我们的代码长成这样：

```javascript
(function(exports, require, module, __filename, __dirname) {
    let count=0
    function addCount(){
        count++
    }
    module.exports={count,addCount}
});
```

那大家思考一下，`require`的时候究竟`module`发生了什么？





## module对象

Module模式的基本特征：

1. 模块化，可重用；
2. 封装了变量和function，和全局的namaspace不接触，松耦合；
3. 只暴露可用public的方法，其它私有方法全部隐藏；



Node内部提供一个`Module`构建函数。所有模块都是`Module`的实例。

```javascript
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  this.loaded = false;
  this.children = [child1,child2,child3];
  this.exports = {"app": [function]}
}
```

每个模块内部，都有一个`module`对象，代表当前模块。它有以下属性。

- `module.id` 模块的识别符，通常是带有绝对路径的模块文件名。
- `module.filename` 模块的文件名，带有绝对路径。
- `module.loaded` 返回一个布尔值，表示模块是否已经完成加载。
- `module.parent` 返回一个对象，表示调用该模块的模块。
- `module.children` 返回一个数组，表示该模块要用到的其他模块。
- `module.exports` 表示模块对外输出的值。



