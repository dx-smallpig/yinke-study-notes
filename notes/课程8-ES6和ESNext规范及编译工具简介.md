# ES6 & ESNext 规范及编译工具简介

## 一、ES6 & ESNext

### 1.1 变量定义新形式

从 ES6 开始，JavaScript 引入了 **let** 和 **const** 关键字来定义变量，这是 ECMAScript 新增的两种声明变量的方式，相较于 var 关键字具有更多的优势。

Q: var 定义变量有什么问题？

1. 变量提升
2. 无法形成词法作用域
3. 可以随意篡改变量值，重复声明



#### 1.1.1 let

let 关键字用于声明一个**块级作用域**的局部变量，可以将 let 声明的变量重新赋值。let 声明的变量只在代码块**内部**有效。

```javascript
if (true) {
    let i = 1;
    console.log(i); // 输出 1
}
console.log(i); // 报错，i 未定义
```



#### 1.1.2 const

const 关键字用于声明一个**块级作用域**的**只读常量**，一旦 const 声明了某个变量，就不能使用赋值语句改变它的值。常量必须在声明时进行初始化。

```javascript
const PI = 3.1415926535;
PI = 3; // 报错，无法修改常量
```



需要这些新增的变量定义形式主要原因如下：

1. 更加安全

使用 let 和 const 可以有效地避免一些变量作用域混淆的问题。通过使用块级作用域，我们可以让变量只在指定代码块内部有效，避免了不必要的变量污染和冲突。

2. 更加简洁

使用 let 和 const 可以大量减少代码量，并且更加易于维护。在使用 var 时，由于变量作用域问题，经常需要添加额外的语句进行变量定义、检查和清除等操作，而使用 let 和 const 可以直接在代码中进行定义和使用，更加简洁和高效。

3. 更加规范

使用 let 和 const 可以使代码更加规范，让代码更好读懂、易于维护。随着 JavaScript 的逐渐发展，代码规范性和可读性越来越重要，let 和 const 关键字的引入正是为了更好地实现这一目标。

综上所述，let 和 const 关键字可以使 JavaScript 更加安全、简洁和规范，有效地解决了以往 JavaScript 变量定义存在的一些问题，带来更佳的开发体验。



#### 1.1.3 实际使用

1. 示例一：循环中使用 let 声明变量避免问题

```javascript
for (let i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2, 3, 4
    }, 1000);
}
```

使用 let 声明的变量 i 有块级作用域，在每一次循环中都会重新定义并赋值，避免了使用 var 声明变量可能导致的变量共享问题。

2. 示例二：使用 const 声明常量

```TypeScript
const PI = 3.1415926;
const URL = "https://www.example.com";
```

使用 const 声明常量可以防止被修改，保证代码的可靠性和稳定性。

3. 示例三：使用 const 声明对象

```TypeScript
const user = {
    name: "张三",
    age: 18,
    gender: "男"
};

user.name = "李四";
console.log(user); // { name: "李四", age: 18, gender: "男" }

Object.freeze(user);

user.age = 20;
console.log(user); // { name: "李四", age: 18, gender: "男" }
```

使用 const 声明引用类型的数据可以避免修改引用的地址，但内部的属性还是能被修改，同时使用 **Object.freeze()** 方法可以将对象冻结，防止意外修改对象属性。

总的来说，使用 let 或者 const 声明变量可以解决一些以往使用 var 变量定义存在的问题，例如变量作用域混乱，变量共享、变量易被修改等情况。使用 let 和 const 可以使代码更加健壮、可维护，提升开发效率和代码质量。

#### 1.1.4 实现原理

底层实现上，let 和 const 的工作方式是通过 **JavaScript** **引擎**来实现的。在 JavaScript 引擎中，每一个变量都会被封装在一个称为“变量对象”的容器中，这个对象包含了所有当前上下文中定义的变量与函数。变量对象类似于一个键/值对的容器，其中键是变量名，值是变量的值。在 JavaScript 引擎中，使用 let 和 const 定义变量，实际上是将该变量定义在了一个块级作用域中，而块级作用域是由编译器在编译阶段中实现的。

![image-20240618140750347](https://gitee.com/dx-smallpig/typora-image/raw/master/images/image-20240618140750347.png)

![image-20240618140812045](https://gitee.com/dx-smallpig/typora-image/raw/master/images/image-20240618140812045.png)

![image-20240618140825589](https://gitee.com/dx-smallpig/typora-image/raw/master/images/image-20240618140825589.png)

##### 1.1.4.1 let的底层实现过程

1. 编译阶段

在代码编译阶段，编译器会扫描整个函数体（或全局作用域），查找所有使用 let 定义的变量，为这些变量生成一个初始值为 undefined 的词法环境（LexicalEnvironment）并将其保存在作用域链中。

2. 进入执行上下文

当进入执行块级作用域（包括 for、if、while 和 switch 等语句块）后，会创建一个新的词法环境。如果执行块级作用域中包含 let 变量声明语句，这些变量将被添加到这个词法环境的环境记录中。

3. 绑定变量值

运行到 let 定义的变量时，JavaScript 引擎会在当前词法环境中搜索该变量。首先在当前环境记录中找到这个变量，如果不存在变量，则向包含当前环境的外部环境记录搜索变量，直到全局作用域为止。如果变量值没有被绑定，JavaScript 引擎会将其绑定为 undefined，否则继续执行其他操作。

4. 实现块级作用域

使用 let 定义变量时，在运行时不会在当前作用域之外创建单独的执行上下文，而是会创建子遮蔽（shadowing）新环境。在子遮蔽的词法环境中，变量的值只在最接近的块级作用域内有效。

##### 1.1.4.2 const的底层实现过程

const 具有与 let 相同的底层实现原理，区别在于 const 定义的变量被视为常量（在赋值之后无法更改），因此变量声明时 必须初始化。此外，应该注意的是，使用 const 声明的对象是可以修改属性的。在定义 const 对象时，对象本身是常量，而不是对象的属性。只有对象本身不能被修改，而对象包含的属性可以任意修改。

**暂时性死区，有点类似 webcomponent 技术中心的 shadowDOM**

### 1.2 面向对象编程——class语法

JavaScript 的**类最终也是一种函数**，，JavaScript 的 class 语法只是**语法糖**，使用 class 关键字创建的类会被编译成一个函数，因此其底层原理与函数有一些相似之处。

Q：JavaScript 原型继承实现继承的有哪些方式？

构造器借用、寄生式继承、组合继承

Q：Class 语法继承，最接近我们自己实现的那种继承方式？

**寄生+组合**

是 JavaScript 引擎将 class 与转化为了原型链实现的方式

#### 1.2.1 类、构造函数

使用 class 关键字来定义类时，在内部会创建一个特殊的函数，称为**构造函数（constructor）**。构造函数用于在创建对象时初始化对象的属性，类似于传统的基于**原型**的代码中的构造函数。

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}

const p = new Person("Tom", 20);
console.log(p.name, p.age); // Tom 20

// 如果直接用构造器的写法
function Person(name, age) {
    // 
}
```

class 中定义的属性和方法分别定义在这个构造函数的 prototype 属性中。并且与原型方式不同的是，类的方法是不可枚举的，因此无法使用 for...in 循环遍历类实例对象的属性和方法。

##### 1.2.1.1 解答可枚举问题

在 JavaScript 中，对象的属性分为两种类型：可枚举属性和不可枚举属性。可枚举属性是指那些可以通过 for...in 循环遍历到的属性，不可枚举属性则是指那些不能通过 for...in 循环遍历到的属性。

如果想要将某个属性设置为不可枚举属性，可以使用 Object.defineProperty() 方法或 Object.defineProperties() 方法，对属性的 enumerable 特征进行设置。示例如下：

```javascript
const obj = {};
Object.defineProperty(obj, 'prop1', {
  value: 'value1',
  enumerable: false
});

for(let key in obj) {
  console.log(key); // 不会输出 'prop1'
}
```

在这个示例中，我们使用 Object.defineProperty() 方法将 obj 对象的 prop1 属性设置为不可枚举属性。最终，for...in 循环语句只会输出 obj 对象中的可枚举属性。

通常情况下，对象的所有普通属性和方法都是可枚举的，例如：

```javascript
const obj = {
  prop1: 'value1',
  prop2: 'value2',
  func1: function() {}
};

for(let key in obj) {
  console.log(key); // 'prop1', 'prop2', 'func1'
}
```

在这个示例中，prop1 和 prop2 是 obj 对象的可枚举属性，而 func1 是 obj 对象的可枚举方法。

需要注意的是，在 ES6 中，通过 class 定义的对象默认不可枚举，就算没有显式地设置 enumerable 属性。这与使用 Object.defineProperty() 方法在 ES5 中设置不可枚举属性的方式不同。如果需要将 class 中的某个属性或方法设置为可枚举属性，需要使用 Object.defineProperty() 方法来进行设置。

#### 1.2.2 继承

在 JavaScript 中，继承是通过类的 prototype 属性实现的。在定义类时，可以使用 extends 关键字来继承其他的类：

```javascript
class Student extends Person {
    constructor(name, age, grade) {
        super(name, age);
        this.grade = grade;
    }
}
```

这段代码中，子类 Student 继承了父类 Person 的构造函数方法并添加了自己的属性和方法。

#### 1.2.3 静态方法和属性

类中的静态方法和属性可以使用 static 关键字来定义，它们不是类实例的属性，而是类本身的属性和方法。

```javascript
class Person {
    static species = "human";

    static saySpecies() {
        console.log(`We are ${this.species}.`);
    }
}
```

这段代码中定义了一个静态方法和一个静态属性，可以通过类本身直接调用静态方法和属性。

#### 1.2.4 getter 和setter

在类中定义 getter 和 setter 方法可以让我们封装实例的内部数据属性，使得这些属性的读写行为更加的安全和合理。

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    get name() {
        return this.name.toUpperCase();
    }

    set age(age) {
        if (age > 0 && age < 120) {
            this.age = age;
        } else {
            console.log("Invalid age value.");
        }
    }

    get age() {
        return this.age;
    }
}
```

在类的实现中，getter 和 setter 其实是被定义在构造函数的 prototype 属性上，从而可以被该类的所有实例对象访问。

#### 1.2.5 class表达式

ES6 还引入了 class 表达式，可以通过这种表达式来创建函数对象。

```javascript
const Person = class {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    sayHi() {
        console.log(`Hi, my name is ${this.name}, I'm ${this.age} years old.`);
    }
};
```

总体来说，JavaScript 的类本质上是一个函数，使用 class 关键字来声明类只是伪代码，这些代码最终都会被转成函数，并存在函数对象的属性和原型属性上。

### 1.3 模板字符串

类名很容易重复冲突，解决这个问题的方法，react 可以使用 **emotion（开发项目）**、styled-components、stitches（组件库可以选他，针对组件变体非常方便）

JavaScript 模板字符串是 ES6 中新增的一种特殊的字符串语法，它允许嵌入表达式和变量，通过 ${} 拼接字符串和表达式，相比传统的字符串拼接来说，模板字符串更具可读性和可维护性。

#### 1.3.1 基础概念和用法

使用模板字符串时，需要使用（\`）来定义字符串，并在字符串中使用 ${expression} 的方式来嵌入表达式和变量，可以使用多行方式来创建较长的字符串。

```javascript
const name = "Tom";
const age = 20;
const str = `My name is ${name}, I'm ${age} years old.
I'm from China.`;
console.log(str);
```

在这个示例中，使用了一个模板字符串来创建一个包括变量和表达式的字符串。

#### 1.3.2 底层实现原理

模板字符串的实现原理，可以大致分为两个步骤：首先，JavaScript 引擎会将模板字符串**解析**成一个函数调用表达式；接着，这个表达式会被执行，并输出一个最终的字符串。

对于第一步，当 JavaScript 引擎解析模板字符串时，它会将特殊字符和变量值分割成多个参数，并将它们作为函数调用的参数传递给一个名为 **Tagged Template** 的函数。该函数的第一个参数是一个数组，其中包含原始模板字符串中的所有字符文字，除了所有插入字符。其余参数则是与模板字符串插值表达式相对应的插入值。

也就是说，上面的示例可以被解析为如下调用:

```javascript
const result = tagFn(["My name is ", ", I'm ", " years old.\nI'm from China."], name, age);
```

```javascript
const tagFn = (temp, ...args) => {
    let str = ''
    for(let i = 0; l < temp.length; i++) {
        // 这里我就不考虑一些容错情况
        str += temp[i] + args[0]
    }
    return str
}
```

DSL 是不可能用 正则去处理的，必须用专业角度的编译原理去实现

其中，tagFn 是一个可被调用的函数，用于实现对模板字符串的自定义处理。我们可以通过这个函数对模板字符串和变量进行任意的处理和操作。也正是由于这种设计，模板字符串才能够像函数一样实现更加复杂的逻辑，比如计算计算、转换等操作。

#### 1.3.3 tagged template

- styled-components
- emotion（mui 使用的）

```javascript
const styled = {
  div: function (strings) {
    const divDom = document.createElement("div");
    strings.forEach((str) => {
      const s = str.split(";");
      for (const kv of s) {
        const [key, value] = kv.split(":");
        // divDom.style[key] = value;
        console.log("🚀 ~ styled.key:", key, value);
      }
    });


    // return react component
    return divDom;
  },
  h1: function (...args) {
    console.log("🚀 ~ styled.args:", args);
  },
};

// styled-components
const StyledDiv = styled.div`
    background: pink
`
```



#### 1.3.4 使用tagged template

除了默认使用模板字符串外，我们还可以自定义 tagged template 来处理模板字符串。标记模板只需要定义一个函数，这个函数的第一个参数为一个标记数组，剩余其他参数是字符串中替换表达式的值，返回值是最终拼接好的字符串。这种模式在一些库中应用广泛，例如 styled-components。

```javascript
function upperCase(strings, ...values) {
  let result = '';
  strings.forEach((str, i) => {
    if (i > 0) {
      result += String(values[i - 1]).toUpperCase();
    }
    result += str;
  });
  return result;
}

const name = 'rocky';
const age = 18;
const str = upperCase`my name is ${name}, i'm ${age} years old.`;
console.log(str);
```

这里我们实现的 upperCase 函数，可以把字符串替换的内容转换成大写。

总的来说，JavaScript 的模板字符串是一种非常实用的语法，它在拼接复杂的字符串时，可以很好地保证代码的可读性和可维护性。其实现原理是利用函数的调用实现的，使得模板字符串可以与 JavaScript 中的函数交互，给程序员带来了更多的可能性。

### 1.4 解构语法

#### 1.4.1 基础概念与原理

JavaScript 的解构是 ES6 中新增的一个语法特性，它可以将数组或对象中的元素提取出来并赋值给变量。解构语法使得对数组和对象的操作更加灵活和便捷。

#### 1.4.2 数组结构底层原理

对数组解构的底层实现分为两个过程：第一步是使用取值函数（getter）读取数组中对应位置的值，第二步是将取得的值赋值给目标变量。

以以下代码为例：

```javascript
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3
```

在这个例子中，JavaScript 引擎背后发生的事情如下：

```javascript
const tempArray = [1, 2, 3];
const a = tempArray[0];
const b = tempArray[1];
const c = tempArray[2];
console.log(a, b, c); // 1 2 3
```

对于数组解构而言，将每个目标变量赋值的过程是独立的，它们并不会相互影响。

#### 1.4.3 对象解构底层原理

解构对象变量的过程与解构数组变量非常类似。将会遍历对象中的每一个属性，然后在解构表达式中查找同名的变量。

```javascript
const {firstname: first, lastname: last, ...rest} = { firstname: 'John', lastname: 'Doe' };
console.log(first, last); // John Doe

// const {name, age, ...restProps} = props
```

在这个例子中，解构对象并赋值给变量的过程可以理解为：

```javascript
const tempObject = { firstname: 'John', lastname: 'Doe' };
const first = tempObject.firstname;
const last = tempObject.lastname;
console.log(first, last); // John Doe
```

需要注意的是，在对象解构语法中，目标变量的名称需与要求的属性名保持相同。当对象中有未定义的目标变量时会被设为 undefined。

#### 1.4.4 嵌套解构底层原理

嵌套解构是指解构表达式中还包含其他的数组或对象解构。嵌套解构的底层实现和单层解构类似，操作每个元素或属性时都需要依次使用取值函数（getter）进行操作。

例如：

```javascript
const [a, [b, [c]]] = [1, [2, [3]]];
console.log(a, b, c); // 1 2 3
```

在这个例子中，解构过程类似于如下代码：

```javascript
const tempArray = [1, [2, [3]]];
const a = tempArray[0];
const tempArray2 = tempArray[1];
const b = tempArray2[0];
const tempArray3= tempArray2[1];
const c = tempArray3[0];
console.log(a, b, c); // 1 2 3
```



#### 1.4.5 默认值

解构语法还支持给目标变量提供默认值，在无法解构出对应的值时会使用默认值。默认值可以是任何 JavaScript 表达式，包括函数调用、变量名、运算符等。

```javascript
const [a = 1, b = 2] = [];
console.log(a, b); // 1 2
```

在这个例子中，解构表达式的值为空数组，因此解构无法成功。但由于设置了默认值，a 和 b 的值会分别设为 1 和 2。

JavaScript 的解构语法的本质是利用 getter 函数和赋值操作将数组或对象的值按照指定的格式赋值给目标变量。这种语法简化了编程的流程，提高了代码的可读性和可维护性。

#### 1.4.6 JavaScript引擎对于解构语法实现的细节

在 JavaScript 引擎处理解构语法时，会执行以下步骤：

1. 如果右边（要解构的对象）是一个具有 Iterator 接口的对象，则需要调用其 iterator 方法，为解构过程创建一个迭代器。迭代器可以让我们对要解构的对象进行遍历，并将其每个属性的取值传递给左边（解构的目标）中相应的变量。
2. 对要解构的目标进行判断，如果为无法解构的值（如 undefined 或 null），则抛出 TypeError。
3. 如果解构语法中指定了默认值，则在对象无法解构到值(undefined)时使用默认值。
4. 嵌套解构会在目标中继续求值以解构嵌套的变量。
5. 解构对于数组和字符串元素时，会按照索引顺序进行赋值；而对于对象中的元素时，会按照属性名进行赋值。
6. 对于对象的解构，会从对象中取出相应属性的值，然后复制到与解构表达式中相应的变量中。如果解构表达式中指定的变量名与属性名不同，需要使用 “key: value” 表示法进行定义。
7. 解构表达式是完全可以包含剩余运算符的，这样即可匹配对象或数组中剩余的属性，将其赋值到相应的变量上。
8. 如果解构表达式中不存在取值函数（getter）和设置函数（setter），则此时解构赋值可以在性能上比原生赋值语句快一些。

JavaScript 引擎在处理解构表达式时，会比使用常规的变量赋值语句多一些步骤。当我们使用解构表达式时，我们可以根据语法的特点编写更加简洁、易读的代码，JavaScript 引擎会自动为我们执行相应的处理过程。

### 1.5 箭头函数

JavaScript中的箭头函数新增的一种函数定义方式，它可以创建一个函数并赋值给变量，使用箭头语法'=>'。在箭头函数中，**this 关键字的作用域与它周围代码作用域相同**，因而有时也被称为“**词法作用域函数**”。

#### 1.5.1 基础概念与使用

箭头函数的原理是基于JavaScript中的闭包、this和参数作用域。在箭头函数中，this关键字始终指向函数所在上下文的this指针，而不是所在作用域的this指针。

箭头函数与普通函数有哪些不同？

1. this 指向，箭头函数不能定义构造器
2. 不能 new
3. 内部无 arguments 对象
4. this 绑定方法失效，比如：call apply bind

举个例子：

```javascript
const add = (a, b) => {
  return a + b;
};
console.log(add(2, 3));
```

在这个示例中，箭头函数会将a和b作为参数，并将其返回值赋值给add。箭头函数的this指针在这里是指向全局对象（即window，在浏览器中）的，而不是函数作用域。

#### 1.5.2 使用场景

1. 当我们需要使用更简洁的语法来定义函数时，可以使用箭头函数代替传统的函数定义语法。箭头函数更为简洁、易读，可以使用单行语法来代替多行语法。

```javascript
// 传统函数定义
function multiply(x, y) {
  return x * y;
}
// 箭头函数
const multiply = (x, y) => x * y;
```



2. 当我们需要引用所在父级的this指针时，可以使用箭头函数，因为箭头函数中的this指向的是全局对象，而不是函数调用时的上下文对象。而不使用箭头函数时，这里会因为this指针指向的问题而带来一些不便。

```javascript
const obj = {
  name: 'Tom',
  age: 20,
  say: function() {
    console.log(`My name is ${this.name}, I'm ${this.age} years old`);
  }
};
setTimeout(obj.say, 1000);
```

在这个示例中，由于setTimeout中的this指针的问题，函数say中的name属性和age属性无法正常被引用。而使用箭头函数就能解决这个问题。

```javascript
const obj = {
  name: 'Tom',
  age: 20,
  say: function() {
    setTimeout(() => {
      console.log(`My name is ${this.name}, I'm ${this.age} years old`);
    }, 1000);
  }
};
obj.say();
```



3. 在函数式编程中，箭头函数用于编写函数式代码时，简化函数的定义和函数调用的过程。

总的来说，箭头函数在 JavaScript 中被广泛地使用，并在许多应用场景中表现出越来越好的优势。箭头函数简洁、易读，易于编写，同时和其他高级语法结合使用时也具有很高的灵活性。

#### 1.5.3 不能使用的场景

1. **不能用作构造函数**：箭头函数没有自己的this指针，所以不能作为构造函数来使用。因此，不能使用new关键字来调用箭头函数来创建一个新对象。

```javascript
const Person = (name, age) => {
  this.name = name;
  this.age = age;
};
const person1 = new Person('Tom', 20); // 报错，Person不是构造函数
```

2. **不能使用 arguments 关键字**：在箭头函数中，函数的参数为指定的参数，没有额外的 arguments 对象。如果需要使用 arguments 参数，必须使用常规的函数语法。

```javascript
const func1 = () => {
 console.log(arguments); // 报错，arguments未定义
};
func1(1, 2, 3);
```

如果需要使用 arguments，则需要使用function函数定义方式：

```javascript
const func2 = function() {
 console.log(arguments);
};
func2(1, 2, 3); // 输出[1, 2, 3]
```

3. **不能通过 call()、apply()方法修改this指向**：对于箭头函数，它的 this 指针指向词法作用域中的this值，无法通过call()、apply()方法来修改。

```javascript
const obj = {
  name: 'Tom',
  age: 20,
  say: () => {
    console.log(`My name is ${this.name}, I'm ${this.age} years old`);
  }
};
obj.say.call({ name: 'Bill', age: 30 }); // 输出 My name is undefined, I'm undefined years old
```

在这个示例中，虽然我们通过call()方法强制修改了say()的this值，但结果表明this值的实际结果并没有得到改变。这是因为箭头函数本身并没有自己的this值，在这里仍然使用的是上面全局对象的this值。

综上所述，尽管箭头函数在许多情况下都表现得非常优异，但仍然有一些限制场景需要我们关注。在处理这些限制场景时，我们可以使用传统的函数语法，或其他语法来满足我们的代码需求。

### 1.6 生成器generator

生成器解决了**函数**的不可中断性

生成器是可中断函数，yield 关键字暂时中断函数执行，在合适的实际从中断位置继续执行。

JavaScript中的生成器（Generator）引入的一种函数类型，它与传统的函数不同之处在于，在生成器中，我们可以**中途停止函数的执行**，并保存相关的上下文信息，待下次继续执行时可以从保存的上下文信息处继续执行。

#### 1.6.1 基本概念

生成器的定义与传统函数非常相似，不同之处在于生成器函数的关键字为“function*”（注意是带星号的function），并使用 **yield** 操作符来指定生成器函数的执行步骤。yield 操作符可以看做是一个暂停器，它可以和外部程序交换数据，并在函数执行停滞时暂停函数的操作，并记录下执行状态信息以备之后恢复时使用。

下面是一个简单的生成器示例：

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

const sequence = generateSequence(); // 获取生成器实例
console.log(sequence.next().value); // 输出 1
console.log(sequence.next().value); // 输出 2
console.log(sequence.next().value); // 输出 3
```

在这个示例中，我们定义了一个 generateSequence 的生成器函数，并在其中使用 yield 停止程序的执行，生成器函数执行到 yield 的时候，就会停止执行并且将 yield 后面的值返回作为值，并记录下当前运行的上下文信息，等待下一次调用 next() 方法时恢复上下文信息和yield后面的值，并继续执行，直到生成器函数执行结束。

生成器可以看做是一种特殊的迭代器，它与普通迭代器不同之处在于，它的 yield 关键字可以返回多个值，并且它具备暂停及恢复执行状态的功能。通常情况下，我们可以使用生成器来处理那些状态化的问题，在文件读写、网络请求、流式计算等处理方式中都可以使用生成器的特性来优化代码效率。

#### 1.6.2 使用场景

生成器的使用场景主要是在需要处理大量异步操作并保持状态的情况下。生成器不仅可以使代码简洁易懂，还可以避免**回调****地狱**和 **Promise 降解**的问题。

生成器可以通过 yield （产出）操作暂停函数的执行，并返回一个值，等待下一次调用 next （下一个）方法重新启动函数的执行，并继续执行到下一个 yield 操作或函数退出。这样就可以在暂停和恢复的过程中，保持函数的状态，避免了频繁的创建/销毁该函数的内部变量，提高了性能。**（串行）**parallel

下面是一个使用生成器处理异步操作的示例：

```javascript
function* myGenerator() {
  const result1 = yield asyncOperation1(); // 发起异步操作1
  const result2 = yield asyncOperation2(result1); // 发起异步操作2，并将异步操作1的结果作为参数
  return result2; // 返回异步操作2的结果
}

function asyncOperation1() {
  return new Promise(resolve => setTimeout(() => resolve('result1'), 1000));
}

function asyncOperation2(arg) {
  return new Promise(resolve => setTimeout(() => resolve(`result2-${arg}`), 1000));
}

const gen = myGenerator(); // 获取生成器实例
const p1 = gen.next(); // 启动异步操作1
p1.value.then(result1 => {
  const p2 = gen.next(result1); // 启动异步操作2，将异步操作1 的结果作为参数传递
  p2.value.then(result2 => {
    console.log(result2); // 输出 result2-result1
  });
});

// while(!p.done) // 一直执行到 done
```

在这个示例中，我们定义了 myGenerator 生成器函数，它使用 yield 操作暂停执行，并在异步操作完成后恢复执行，并传递相应的参数。我们利用 Promise 实例来完成异步操作，并使用then() 方法来获取异步操作的结果，并将结果作为参数传递给下一个yield操作。

在上面这个示例中，我们使用生成器函数来方便地完成了两个异步操作的串联和参数传递。这种方式的优点在于代码可读性和可维护性都得到了大大的提高。同时，生成器自身的状态保持特性，也使得代码的性能得到了提升。

### 1.7 异步处理——callback、Promise、async & await

最早是 callbacks，有回调地狱的问题，其实，我们谈到异步处理，就要结合浏览器的**事件循环机制**

Promise 其实还是没有很好地解决嵌套过深的问题

Promise + generator 实现

async & await 

异步编程是一种处理事件循环等待结果返回的方法，常见的实现方式有 callback、Promise 和 async/await。

#### 1.7.1 callback

callback 是一种异步编程模式，通过回调函数的方式实现，通常用于处理一次性异步请求。callback 的好处在于实现起来简单，但由于回调函数嵌套层数容易过多，使得代码可读性和可维护性受到影响。

```javascript
function fetchData(callback) {
  setTimeout(() => {
    const data = 'Hello World!';
    callback(data);
  }, 1000);
}

fetchData((data) => {
  console.log(data); // 输出 Hello World!
})
```



#### 1.7.2 Promise

Promise 是 ES6 提供的一种处理异步操作的机制，用于解决 callback 回调函数嵌套过多的问题。Promise 可以链式调用，通过 then() 方法来处理返回值，同时还提供了 catch() 方法来处理错误。

```javascript
function fetchData() {
  return new Promise(function(resolve) {
    setTimeout(() => {
      const data = 'Promise';
      resolve(data);
    }, 1000);
  });
}

fetchData().then(function(data){
  console.log(data); // 输出 Promise
});
```

#### 1.7.3 async / await

async/await 是 ES8 的新特性，是基于 Promise 的一种异步编程方式，它可以使异步代码看起来像同步代码，语法简单易懂，可读性较高。async 是用于定义一个异步函数，await 用于等待一个异步操作完成。async 函数返回一个 Promise 对象，await 关键字只能在 async 函数中使用。

```javascript
async function fetchData() {
  return new Promise(resolve => {
    setTimeout(() => {
      const data = 'async/await';
      resolve(data);
    }, 1000);
  });
}

async function printData() {
  const data = await fetchData();
  console.log(data); // 输出 async/await
}

printData();
```

总的来说，Promise 和 async/await 相对于 callback 函数来说更加强大和易用，能够有效**避免回调地狱**的问题。而 async/await 相比 Promise 的链式调用，则可以**更好地表达异步操作**的关系，让代码更易懂。
