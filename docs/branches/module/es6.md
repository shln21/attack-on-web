# Es6中的模块化Module

## 简介

在Es6之前，javascript没有模块系统，它无法将一个大程序拆分成若干个互相依赖的小文件，然后在用简单的方法拼装起来。为了做到模块化,在Es6之前，引入了AMD(Asynchronous module definition)与CMD(common module definition)



## 基本语法

示例

```javascript
// 导出数据
export var  name = "ES6模块Module";// 导出暴露name变量
export let  id = "123321";// 暴露weChatPublic
export const time = 2021;// 暴露time


// 导出函数
 export function sum(num1,num2){
      return num1+num2;
 }
/*
*
* 也可以写成
* function sum(num1,num2){
* 		return num1+num2;
* }
* export sum;// 没有通过export关键字导出,在外部是无法访问该模块的变量或者函数的
* */

// 导出类
export class People{
    introduce(name,age,phone){
        this.name = name;
        this.age = age;
        this.phone = phone;
    }
    info(){
    	return `${this.name}${this.age}岁了,联系方式：${this.phone}!`;
    }
}
```

Es6中如何给导入导出时标识符重命名
从一个模块导入变量，函数或者类时，我们可能不希望使用他们的原始名称，就是导入导出时模块内的标识符(变量名、函数或者类)可以不用一一对应，保持一致，可以在导出和导入过程中改变导出变量对象的名称

使用方式：使用as关键字来指定变量、函数或者类在模块外应该被称为什么名称 

示例

```javascript
function sum(num1,num2){
     return num1+num2;
}
export {sum as add} // as后面是重新指定的函数名
```


函数sum是本地名称，add是导出时使用的名称，换句话说，当另一个模块要导入这个函数时，必须使用add这个名称

若在test.js一模块中，则导入的变量对象应是add而不是sum，是由它导出时变量对象决定的

```javascript
import  {add} from "./test.js"
```

如果模块想使用不同的名称来导入函数，也可以使用as关键字

```javascript
import {add as sum} from "./test.js"
```

## ES6模块与CommonJS模块的差异

讨论Node加载ES6模块之前，必须了解ES6模块与CommonJS模块的差异，具体的两大差异如下：

- CommonJS模块是运行时加载，ES6模块是编译时输出接口。
- CommonJS模块输出的是一个值的复制，ES6模块输出的是值的引用。

第一个差异是因为CommonJS加载的是一个对象（即module.exports属性），该对象只有在脚本运行结束时才会生成。而ES6模块不是对象，它的对外接口只是一种静态定义，在代码静态解析阶段就会生成。

第二个差异。

CommonJS模块输出的是值的复制，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值。请看下面这个模块文件lib.js的例子。

```javascript
// lib.js
var counter = 3;
function incCounter() {
	counter++;
}
module.exports = {
    counter: counter,
    incCounter: incCounter,
};
```

上面的代码输出内部变量counter和改写这个变量的内部方法incCounter。然后，在main.js里面加载这个模块。

```javascript
// main.js
var mod = require('./lib');
console.log(mod.counter);// 3
mod.incCounter();
console.log(mod.counter);// 3
```

上面的代码说明，lib.js模块加载以后，它的内部变化就影响不到输出的mod.counter了。这是因为mod.counter是一个原始类型的值，会被缓存。除非写成一个函数，否则得到内部变动后的值。

```javascript
// lib.js
var counter = 3;
function incCounter() {
	counter++;
}
module.exports = {
    get counter() {
    	return counter
    },
    incCounter: incCounter,
};
```

上面的代码中，输出的counter属性实际上是一个取值器函数。现在再执行main.js就可以正确读取内部变量counter的变动了。

```javascript
$ node main.js
3
4
```

ES6模块的运行机制与CommonJS不一样。JS引擎对脚本静态分析的时候，遇到模块加载命令import就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用到被加载的模块中取值。换句话说，ES6的import有点像Unix系统的“符号连接”，原始值变了，import加载的值也会跟着变。因此，ES6模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块。

还是以上面的代码为例。

```javascript
// lib.js
export let counter = 3;
export function incCounter() {
	counter++;
}


// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

上面的代码说明，ES6模块输入的变量counter是活的，完全反应其所在模块lib.js内部的变化。



## 加载机制



