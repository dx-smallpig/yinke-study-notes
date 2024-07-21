# this、闭包和作用域

## 一、原型和原型链

### 构造函数

构造函数和普通函数本质上没什么区别，能通过使用new关键字创建对象的函数，被叫做了构造函数。构造函数的首字母一般是大写，用以区分普通函数，当然不大写也不会有什么错误。

```javascript
function Person() {}
let per1 = new Person();
```

### 实例原型

在JS中，每一个函数类型的数据，都有一个叫做prototype的属性，这个属性指向的是一个对象，就是所谓的实例原型。prototype是函数才会有的属性。

![image.png](https://gitee.com/dx-smallpig/typora-image/raw/master/images/0271cc1c01714be691790c412c0c028a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

对于实例原型来说，它有个constructor属性，指向它的构造函数。

![image.png](https://gitee.com/dx-smallpig/typora-image/raw/master/images/326a4782dc7f498c959fa2dd200d4c4e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 原型链

**`显式原型`**就是利用prototype属性查找的原型。

**`隐式原型`**是利用\__proto__属性查找原型，这个属性指向当前实例的构造函数的实例原型，这个属性是对象类型数据的属性，所以可以在实例上面使用。

![image.png](https://gitee.com/dx-smallpig/typora-image/raw/master/images/76252d36aeea471fa1fc45374af8d93d~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

如果某个对象查找属性，自己和原型对象上都没有，那就会继续往原型对象的原型对象上去找，整个查找过程都是顺着\__proto__属性，一步一步往上查找，形成了像链条一样的结构，这个结构，就是原型链。所以，原型链也叫作隐式原型链。

![image.png](https://gitee.com/dx-smallpig/typora-image/raw/master/images/3fb087ced09e45a5b8e20a4af5be6173~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

```javascript
Person === Person.prototype.constructor	// true
person.__proto__ === Person.prototype	// true
Person.prototype.__proto__ === Object.prototype	// true
Object.prototype.__proto__ === null	// true
// true,实例上没有constructor属性会从原型链上找
person.constructor === Person.prototype.constructor

```

### 函数对象

在JS中，所有函数都可以看做是Function()的实例，而Person()和Object()都是函数，所以它们的构造函数就是Function()。Function()本身也是函数，所以Function()也是自己的实例。

![image.png](https://gitee.com/dx-smallpig/typora-image/raw/master/images/f2b435c6ed064418969d80abcddb44e6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

### 其他

**`proto`**：绝大部分浏览器都支持这个非标准的方法访问原型，然而它并不存在于 `Person.prototype` 中，实际上，它是来自于 `Object.prototype` ，与其说是一个属性，不如说是一个 `getter/setter`，当使用 `obj.__proto__` 时，可以理解成返回了 `Object.getPrototypeOf(obj)`。



> **参考文献**
>
> 作者：hobby爱吃猫的鱼
> 链接：https://juejin.cn/post/7255605810453217335
> 来源：稀土掘金
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## 二、作用域

作用域是指程序源代码中定义变量的区域。

作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。

作用域分为两类

1. **``静态作用域（词法作用域）``**：作用域在函数定义时确定
2. **``动态作用域``**：作用域在函数调用时确定

JavaScript 采用词法作用域`(lexical scoping`)，也就是静态作用域。

```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();	// 1
```

```javascript
// case 1
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();	// "local scope"

// case 2
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();	// "local scope"
```



## 三、执行上下文

### 顺序执行

JavaScript 引擎并非一行一行地分析和执行程序，而是一段一段地分析执行。

由var定义的变量和函数定义都会提升

```javascript
// 变量提升var foo = underfind
// 在使用时重新赋值函数
var foo = function () {
	console.log('foo1');
}
foo();  // foo1

var foo = function () {
	console.log('foo2');
}
foo(); // foo2
```

```javascript
// 函数定义也会提升，重复声明按顺序保留最下面的
function foo() {
	console.log('foo1');
}
foo();  // foo2

function foo() {
	console.log('foo2');
}
foo(); // foo2
```

```javascript
console.log(add2(1,1)); //输出2
function add2(a,b){
    return a+b;
}
console.log(add1(1,1));  //报错：add1 is not a function，add1 = underfind
var add1 = function(a,b){
    return a+b;
}

// 用函数语句创建的函数add2，函数名称和函数体均被提前，在声明它之前就使用它。
// 但是使用var表达式定义函数add1，只有变量声明提前了，变量初始化代码仍然在原来的位置，没法提前执行。
```



### 可执行代码

JavaScript 的可执行代码（`executable code`）的类型：

- 全局代码
- 函数代码
- eval代码

当执行到一个函数的时候，就会生成执行上下文（`execution context`）

### 执行上下文栈

JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：

```javascript
ECStack = [];
```

当 JavaScript 开始要解释执行代码的时候，最先遇到的就是全局代码，所以初始化的时候首先就会向执行上下文栈压入一个全局执行上下文，我们用 `globalContext` 表示它，并且只有当整个应用程序结束的时候，ECStack才会被清空，所以程序结束之前， ECStack 最底部永远有个 `globalContext`：

```javascript
ECStack = [
    globalContext
];
```

当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。

```javascript
function fun3() {
	console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

```javascript
// 执行fun1()
ECStack.push(<fun1> functionContext);

// 执行fun2()
ECStack.push(<fun2> functionContext);

// 执行fun3()
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有个globalContext
```

在上文的作用域中的两个示例，输出结果相同，但执行上下文栈不同

```javascript
// case 1
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();

// 执行上下文栈
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

```javascript
// case 2
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();

// 执行上下文栈
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```



## 四、变量对象

当JavaScript代码执行一段可执行代码（`executable code`）时，会创建对应的执行上下文（`execution context`）。

执行上下文具有三个属性：

- 变量对象（`Variable Object`，VO）
- 作用域链（scope chain）
- this

变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。

### 全局上下文

1. 全局对象是预定义的对象，作为 JavaScript 的全局函数和全局属性的占位符。通过使用全局对象，可以访问所有其他所有预定义的对象、函数和属性。
2. 在顶层 JavaScript 代码中，可以用关键字 this 引用全局对象。因为全局对象是作用域链的头，这意味着所有非限定性的变量和函数名都会作为该对象的属性来查询。
3. 例如，当JavaScript 代码引用 parseInt() 函数时，它引用的是全局对象的 parseInt 属性。全局对象是作用域链的头，还意味着在顶层 JavaScript 代码中声明的所有变量都将成为全局对象的属性。

可以通过 this 引用，在客户端 JavaScript 中，全局对象就是 Window 对象。

```javascript
console.log(this);	// window
```

全局对象是由 Object 构造函数实例化的一个对象。

```javascript
console.log(this instanceof Object);	// true
```

预定义的属性是否可用

```javascript
console.log(Math.random());
console.log(this.Math.random());
```

作为全局变量的宿主

```javascript
var a = 1;
console.log(this.a);	// 1
```

客户端 JavaScript 中，全局对象有 window 属性指向自身

```javascript
var a = 1;
console.log(window.a); 	// 1
console.log(window === this);	// true
```



### 函数上下文

在函数上下文中，我们用活动对象（`Activation Object`, AO）来表示变量对象。

活动对象和变量对象其实是一个东西，只是变量对象是规范上的或者说是引擎实现上的，不可在 JavaScript 环境中访问，只有到当进入一个执行上下文中，这个执行上下文的变量对象才会被激活，所以才叫 Activation Object，而只有被激活的变量对象，也就是活动对象上的各种属性才能被访问。

活动对象是在进入函数上下文时刻被创建的，它通过函数的 arguments 属性初始化。arguments 属性值是 Arguments 对象。

### 执行过程

执行上下文的代码会分成两个阶段进行处理：分析和执行，我们也可以叫做：

1. 进入执行上下文；
2. 代码执行；

进入执行上下文时，这时候还没有执行代码，

变量对象会包括：

- 函数的所有形参 (如果是函数上下文)
  - 由名称和对应值组成的一个变量对象的属性被创建；
  - 没有实参，属性值设为 undefined；
- 函数声明
  - 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建；
  - 如果变量对象已经存在相同名称的属性，则完全替换这个属性；
- 变量声明
  - 由名称和对应值（undefined）组成一个变量对象的属性被创建；
  - 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性；

```javascript
function foo(a) {
  var b = 2;
  function c() {}
  var d = function() {};
  b = 3;
}
foo(1);

// 执行上下文AO
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c(){},
    d: undefined
}
```

代码执行阶段

会顺序执行代码，根据代码，修改变量对象的值

```javascript
// 上述例子执行后
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c(){},
    d: reference to FunctionExpression "d"
}
```

综上述过程

1. 全局上下文的变量对象初始化是全局对象；
2. 函数上下文的变量对象初始化只包括 Arguments 对象；
3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值；
4. 在代码执行阶段，会再次修改变量对象的属性值；

## 五、作用域链

当JavaScript代码执行一段可执行代码（`executable code`）时，会创建对应的执行上下文（`execution context`）。

执行上下文具有三个属性：

- 变量对象（`Variable Object`，VO）
- 作用域链（scope chain）
- this

当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链。

### 函数创建

上文的词法作用域与动态作用域中讲到，函数的作用域在函数定义的时候就决定了。

这是因为函数有一个内部属性 [[scope]]，当函数创建的时候，就会保存所有父变量对象到其中，你可以理解 [[scope]] 就是所有父变量对象的层级链，但是注意：[[scope]] 并不代表完整的作用域链！

```javascript
function foo() {
    function bar() {
        ...
    }
}
```

```javascript
foo.[[scope]] = [
	globalContext.VO
];

bar.[[scope]] = [
    fooContext.AO,
    globalContext.VO
];
```



### 函数激活

当函数激活时，进入函数上下文，创建 VO/AO 后，就会将活动对象添加到作用链的前端。

```javascript
foo.[[scope]] = [
    foo.AO,
	globalContext.VO
];

bar.[[scope]] = [
    bar.AO,
    fooContext.AO,
    globalContext.VO
];
```



## 六、this

this一直指向调用该方法的对象



## 七、闭包

MDN 对闭包的定义为：

> 闭包是指那些能够访问自由变量的函数。

那什么是自由变量呢？

> 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。

由此，我们可以看出闭包共有两部分组成：

> 闭包 = 函数 + 函数能够访问的自由变量

```javascript
var a = 1;

function foo() {
    console.log(a);
}

foo();
// foo 函数可以访问变量 a，但是 a 既不是 foo 函数的局部变量，也不是 foo 函数的参数，所以 a 就是自由变量。
```

所以在《JavaScript权威指南》中就讲到：从技术的角度讲，所有的JavaScript函数都是闭包。

但是，这是理论上的闭包，其实还有一个实践角度上的闭包。

ECMAScript中，闭包指的是：

1. 从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域；
2. 从实践角度：以下函数才算是闭包：
   1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）；
   2. 在代码中引用了自由变量；

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}

var foo = checkscope();
foo();
```

执行过程：

1. 进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈；
2. 全局执行上下文初始化；
3. 执行 `checkscope` 函数，创建 `checkscope` 函数执行上下文，`checkscope` 执行上下文被压入执行上下文栈；
4. `checkscope` 执行上下文初始化，创建变量对象、作用域链、this等；
5. `checkscope` 函数执行完毕，`checkscope` 执行上下文从执行上下文栈中弹出；
6. 执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈；
7. f 执行上下文初始化，创建变量对象、作用域链、this等；
8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出；

```javascript
fContext = {
    Scope: [AO, checkscopeContext.AO, globalContext.VO],
}
```

因为这个作用域链，f 函数依然可以读取到 `checkscopeContext.AO` 的值，说明当 f 函数引用了 `checkscopeContext.AO` 中的值的时候，即使 `checkscopeContext` 被销毁了，但是 JavaScript 依然会让 `checkscopeContext.AO` 活在内存中，f 函数依然可以通过 f 函数的作用域链找到它，正是因为 JavaScript 做到了这一点，从而实现了闭包这个概念。

所以，让我们再看一遍实践角度上闭包的定义：

1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）；
2. 在代码中引用了自由变量；









