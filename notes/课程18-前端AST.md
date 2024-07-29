# 前端AST

## 一、什么是编译器

### 1.1 背景

在babel的[官网](https://babeljs.io/)里，最显著的内容就是：

> Babel is a JavaScript compiler

那么什么是所谓的JavaScript compiler？我们应当如何学习和理解compiler？

### 1.2 编译器介绍

> compiler也叫编译器，是一种电脑程序，它会将用某种编程语言写成的源代码，转换成另一种编程语言。

从维基百科的定义来看，编译器就是个将当前语言转为其他语言的过程，回到babel上，它所做的事就是语法糖之类的转换，比如ES6/ES7/JSX转为ES5或者其他指定版本，因此称之为compiler也是正确的，换言之，像我们平时开发过程中所谓的其他工具，如：

- Less/Saas
- TypeScript/coffeeScript
- Eslint
- etc...

都可以看到compiler的身影，也是通过这些工具，才使得目前的前端工程化能走入相对的深水区，以下会详细介绍下compiler的实现思路及具体demo，帮助同学们了解compiler的基本实现。



## 二、编译器的基本思路

> 此处主要讲解compiler的思路

### 2.1 词法分析(Lexical Analysis)

#### 2.1.1 目的

将文本分割成一个个的“token”，例如：init、main、init、x、;、x、=、3、;、}等等。同时它可以去掉一些注释、空格、回车等等无效字符

#### 2.1.2 生成方式

词法分析生成token的办法有2种：

需要写大量的正则表达式，正则之间还有冲突需要处理，不容易维护，性能不高，所以正则只适合一些简单的模板语法，真正复杂的语言并不合适。并且有的语言并不一定自带正则引擎。

自动机可以很好的生成token；

有穷状态自动机（finite state machine）：在有限个输入的情况下，在这些状态中转移并期望最终达到终止状态。

有穷状态自动机根据确定性可以分为：

​	“确定有穷状态自动机”（DFA - Deterministic finite automaton）

在输入一个状态时，只得到一个固定的状态。DFA 可以认为是一种特殊的 NFA；

​	“非确定有穷自动机”（NFA - Non-deterministic finite automaton）

当输入一个字符或者条件得到一个状态机的集合。JavaScript 正则采用的是 NFA 引擎，具体看后文；



### 2.2 语法分析(Syntactic Analysis)

我们日常所说的编译原理就是将一种语言转换为另一种语言。编译原理被称为形式语言，它是一类无需知道太多语言背景、无歧义的语言。而自然语言通常难以处理，主要是因为难以识别语言中哪些是名词哪些是动词哪些是形容词。例如：“进口汽车”这句话，“进口”到底是动词还是形容词？所以我们要解析一门语言，前提是这门语言有严格的语法规定的语言，而定义语言的语法规格称为文法。

1956年，乔姆斯基将文法按照规范的严格性分为0型、1型、2型和3型共4中文法，从0到3文法规则是逐渐增加严的。一般的计算机语言是2型，因为0和1型文法定义宽松，将大大增加解析难度、降低解析效率，而3型文法限制又多，不利于语言设计灵活性。2型文法也叫做上下文无关文法（CFG）。

 语法分析的目的就是通过词法分析器拿到的token流 + 结合文法规则，通过一定算法得到一颗抽象语法树（AST）。抽象语法树是非常重要的概念，尤其在前端领域应用很广。典型应用如babel插件，它的原理就是：es6代码 → Babylon.parse → AST → babel-traverse → 新的AST → es5代码。

 从生成AST效率和实现难度上，前人总结主要有2种解析算法：自顶向下的分析方法和自底向上的分析方法。自底向上算法分析文法范围广，但实现难度大。而自顶向下算法实现相对简单，并且能够解析文法的范围也不错，所以一般的compiler都是采用深度优先索引的方式。



### 2.3 代码转换(Transformation)

在得到AST后，我们一般会先将AST转为另一种AST，目的是生成更符合预期的AST，这一步称为代码转换。

代码转换的优势：主要是产生工程上的意义

- 易移植：与机器无关，所以它作为中间语言可以为生成多种不同型号的目标机器码服务；
- 机器无关优化：对中间码进行机器无关优化，利于提高代码质量；
- 层次清晰：将AST映射成中间代码表示，再映射成目标代码的工作分层进行，使编译算法更加清晰 ；

对于一个Compiler而言，在转换阶段通常有两种形式：

同语言的AST转换；

AST转换为新语言的AST；

这里有一种通用的做法是，对我们之前的AST从上至下的解析（称为traversal），然后会有个映射表（称为visitor），把对应的类型做相应的转换。



### 2.4 代码生成 (Code Generation)

在实际的代码处理过程中，可能会递归的分析（）我们最终生成的AST，然后对于每种type都有个对应的函数处理，当然，这可能是最简单的做法。总之，我们的目标代码会在这一步输出，对于我们的目标语言，它就是HTML了。



### 2..5 完整链路(Compiler)

至此，我们就完成了一个完整的compiler的所有过程：

```JavaScript
input => tokenizer => tokens; // 词法分析
tokens => parser => ast; // 语法分析，生成AST
ast => transformer => newAst; // 中间层代码转换
newAst => generator => output; // 生成目标代码
```



## 三、一个简单的编译器的实现

### 3.1 前置内容

```JavaScript
/**
 * 今天我们要一起写一个编译器。但不仅仅是任何编译器......
 * 超级小的编译器！一个很小的编译器，如果你
 * 删除所有注释，这个文件只有大约 200 行实际代码。
 *
 * 我们将把一些类似语义化代码的函数调用编译成一些类似 C 的函数
 * 函数调用。
 *
 * 如果您不熟悉其中之一。我只是给你一个快速的介绍。
 *
 * 如果我们有两个函数 `add` 和 `subtract` 他们会写成这样：
 *
 * 类似 C
 *
 * 2 + 2 (加 2 2) 加 (2, 2)
 * 4 - 2 (减 4 2) 减 (4, 2)
 * 2 + (4 - 2) (加 2 (减 4 2)) 加 (2, 减 (4, 2))
 *
 *
 * 很好，因为这正是我们要编译的。虽然这
 * 不是完整的 C 语法，它的语法足以
 * 演示现代编译器的许多主要部分。
 */

/**
 * 大多数编译器分为三个主要阶段：解析、转换、
 * 和代码生成
 *
 * 1. *解析* 将原始代码转化为更抽象的代码
 * 代码的表示。
 *
 * 2. *转换* 采用这种抽象表示并进行操作
 * 无论编译器想要什么。
 *
 * 3. *代码生成*采用转换后的代码表示，并
 * 将其转换为新代码。
 */
```

```JavaScript
/**
 * 解析
 * --------
 *
 * 解析通常分为两个阶段：词法分析和
 * 句法分析。
 *
 * 1. *词法分析*获取原始代码并将其拆分成这些东西
 * 被称为标记器（或词法分析器）的东西称为标记。
 *
 * Tokens 是一组微小的对象，描述了一个孤立的部分
 * 的语法。它们可以是数字、标签、标点符号、运算符、
 *    任何。
 *
 * 2. *句法分析*获取标记并将它们重新格式化为
 * 描述语法的每个部分及其关系的表示
 *    彼此。这被称为中间表示或
 * 抽象语法树。
 *
 * 抽象语法树，简称 AST，是一个深度嵌套的对象，
 * 以一种既易于使用又能告诉我们很多信息的方式表示代码
 * 信息。
 *
 * 对于以下语法：
 *
 * (加 2 (减 4 2))
 *
 * 令牌可能看起来像这样：
 *
 *   [
 *     { type: 'paren',  value: '('        },
 *     { type: 'name',   value: 'add'      },
 *     { type: 'number', value: '2'        },
 *     { type: 'paren',  value: '('        },
 *     { type: 'name',   value: 'subtract' },
 *     { type: 'number', value: '4'        },
 *     { type: 'number', value: '2'        },
 *     { type: 'paren',  value: ')'        },
 *     { type: 'paren',  value: ')'        },
 *   ]
 *
 * 抽象语法树 (AST) 可能如下所示：
 *
 *   {
 *     type: 'Program',
 *     body: [{
 *       type: 'CallExpression',
 *       name: 'add',
 *       params: [{
 *         type: 'NumberLiteral',
 *         value: '2',
 *       }, {
 *         type: 'CallExpression',
 *         name: 'subtract',
 *         params: [{
 *           type: 'NumberLiteral',
 *           value: '4',
 *         }, {
 *           type: 'NumberLiteral',
 *           value: '2',
 *         }]
 *       }]
 *     }]
 *   }
 */
```

```JavaScript
/**
 * 转换
 * --------------
 *
 * 编译器的下一个阶段是转换。再次，这只是
 * 从最后一步获取 AST 并对其进行更改。它可以操纵
 * 使用相同语言的 AST，或者它可以将其翻译成全新的
 * 语。
 *
 * 让我们看看如何转换 AST。
 *
 * 您可能会注意到我们的 AST 中的元素看起来非常相似。
 * 这些对象具有类型属性。这些中的每一个都被称为
 * AST 节点。这些节点在它们上定义了描述一个
 * 树的隔离部分。
 *
 * 我们可以有一个“NumberLiteral”的节点：
 *
*   {
 *     type: 'NumberLiteral',
 *     value: '2',
 *   }
 *
 * Or maybe a node for a "CallExpression":
 *
 *   {
 *     type: 'CallExpression',
 *     name: 'subtract',
 *     params: [...nested nodes go here...],
 *   }
 *
 * 转换 AST 时，我们可以通过以下方式操作节点
 * 添加/删除/替换属性，我们可以添加新节点，删除节点，或者
 * 我们可以不理会现有的 AST 并创建一个全新的基于
 * 在上面。
 *
 * 由于我们的目标是一种新语言，我们将专注于创建一个
 * 特定于目标语言的全新 AST。
 *
 * 遍历
 * ---------
 *
 * 为了浏览所有这些节点，我们需要能够
 * 遍历它们。这个遍历过程会到达 AST 中的每个节点
 * 深度优先。
 *
 *   {
 *     type: 'Program',
 *     body: [{
 *       type: 'CallExpression',
 *       name: 'add',
 *       params: [{
 *         type: 'NumberLiteral',
 *         value: '2'
 *       }, {
 *         type: 'CallExpression',
 *         name: 'subtract',
 *         params: [{
 *           type: 'NumberLiteral',
 *           value: '4'
 *         }, {
 *           type: 'NumberLiteral',
 *           value: '2'
 *         }]
 *       }]
 *     }]
 *   }
 *
 * 所以对于上面的 AST，我们会去：
 *
 * 1. Program - 从 AST 的顶层开始
 * 2. CallExpression (add) - 移动到程序主体的第一个元素
 * 3. NumberLiteral (2) - 移动到 CallExpression 参数的第一个元素
 * 4. CallExpression (subtract) - 移动到 CallExpression 参数的第二个元素
 * 5. NumberLiteral (4) - 移动到 CallExpression 参数的第一个元素
 * 6. NumberLiteral (2) - 移动到 CallExpression 参数的第二个元素
 *
 * 如果我们直接操作这个 AST，而不是创建一个单独的 AST，
 * 我们可能会在这里引入各种抽象。但只是参观
 * 树中的每个节点都足以完成我们正在尝试做的事情。
 *
 * 我使用“访问”这个词的原因是因为有这样的模式
 * 表示对对象结构元素的操作。
*
 * Visitors
 * --------
 *
 * 这里的基本思想是我们将创建一个“访问者”对象，
 * 具有将接受不同节点类型的方法。
 *
 *   var visitor = {
 *     NumberLiteral() {},
 *     CallExpression() {},
 *   };
 *
 * 当我们遍历我们的 AST 时，我们会在任何时候调用这个访问者的方法
 * “输入”一个匹配类型的节点。
 *
 * 为了使它有用，我们还将传递节点和引用
 * 父节点。
 *
 *   var visitor = {
 *     NumberLiteral(node, parent) {},
 *     CallExpression(node, parent) {},
 *   };
 *
 * 但是，也存在在“退出”时调用事物的可能性。想象
 * 我们之前的树形结构以列表形式：
 *
 *   - Program
 *     - CallExpression
 *       - NumberLiteral
 *       - CallExpression
 *         - NumberLiteral
 *         - NumberLiteral
 *
 * 当我们向下遍历时，我们将到达有死胡同的分支。正如我们
 * 完成我们“退出”它的树的每个分支。所以我们顺着树走
 *“进入”每个节点，然后返回我们“退出”。
 *
 *   -> Program (enter)
 *     -> CallExpression (enter)
 *       -> Number Literal (enter)
 *       <- Number Literal (exit)
 *       -> Call Expression (enter)
 *          -> Number Literal (enter)
 *          <- Number Literal (exit)
 *          -> Number Literal (enter)
 *          <- Number Literal (exit)
 *       <- CallExpression (exit)
 *     <- CallExpression (exit)
 *   <- Program (exit)
 *
 * 为了支持这一点，我们的访问者的最终形式将如下所示：
 *
 *   var visitor = {
 *     NumberLiteral: {
 *       enter(node, parent) {},
 *       exit(node, parent) {},
 *     }
 *   };
 */
```

```JavaScript
/**
 * 代码生成
 * ---------------
 *
 * 编译器的最后阶段是代码生成。有时编译器会做
 * 与转换重叠的东西，但大部分是代码
 * 生成只是意味着取出我们的 AST 和字符串化代码。
 *
 * 代码生成器有几种不同的工作方式，一些编译器会重用
 * 早期的令牌，其他人将创建一个单独的表示
 *代码，以便他们可以线性打印节点，但据我所知
 * 将使用我们刚刚创建的相同 AST，这是我们将重点关注的内容。
 *
 * 实际上，我们的代码生成器将知道如何“打印”所有不同的
 * AST的节点类型，它会递归调用自己打印嵌套
 * 节点，直到所有内容都打印成一长串代码。
 */

/**
 * 就是这样！这就是编译器的所有不同部分。
 * 现在这并不是说每个编译器看起来都和我在这里描述的完全一样。
 * 编译器有许多不同的用途，它们可能需要更多的步骤
 * 但是现在您应该对大多数编译器的外观有一个大致的高级概念
 */
```



### 3.2 词法分析

```JavaScript
function tokenizer(input) {
  let current = 0;

  let tokens = [];

  while (current < input.length) {
    let char = input[current];

    if (char === '(') {
      tokens.push({
        type: 'paren',
        value: '(',
      });

      current++;

      continue;
    }

    if (char === ')') {
      tokens.push({
        type: 'paren',
        value: ')',
      });
      current++;
      continue;
    }

    let WHITESPACE = /\s/;
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    let NUMBERS = /[0-9]/;
    if (NUMBERS.test(char)) {
      let value = '';

      while (NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }

      tokens.push({ type: 'number', value });

      continue;
    }

    if (char === '"') {
      let value = '';

      char = input[++current];

      while (char !== '"') {
        value += char;
        char = input[++current];
      }

      char = input[++current];

      tokens.push({ type: 'string', value });

      continue;
    }

    let LETTERS = /[a-z]/i;
    if (LETTERS.test(char)) {
      let value = '';

      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      tokens.push({ type: 'name', value });

      continue;
    }

    throw new TypeError('I dont know what this character is: ' + char);
  }
  return tokens;
}
```



### 3.3 语法分析

```JavaScript
function parser(tokens) {
  let current = 0;

  function walk() {
    let token = tokens[current];

    if (token.type === 'number') {
      current++;

      return {
        type: 'NumberLiteral',
        value: token.value,
      };
    }

    if (token.type === 'string') {
      current++;

      return {
        type: 'StringLiteral',
        value: token.value,
      };
    }

    if (token.type === 'paren' && token.value === '(') {
      token = tokens[++current];

      let node = {
        type: 'CallExpression',
        name: token.value,
        params: [],
      };

      token = tokens[++current];

      while (token.type !== 'paren' || (token.type === 'paren' && token.value !== ')')) {
        node.params.push(walk());
        token = tokens[current];
      }

      current++;

      return node;
    }

    throw new TypeError(token.type);
  }

  let ast = {
    type: 'Program',
    body: [],
  };

  while (current < tokens.length) {
    ast.body.push(walk());
  }
  return ast;
}
```



### 3.4 代码转换

```JavaScript
function traverser(ast, visitor) {
  function traverseArray(array, parent) {
    array.forEach(child => {
      traverseNode(child, parent);
    });
  }

  function traverseNode(node, parent) {
    let methods = visitor[node.type];

    if (methods && methods.enter) {
      methods.enter(node, parent);
    }

    switch (node.type) {
      case 'Program':
        traverseArray(node.body, node);
        break;

      case 'CallExpression':
        traverseArray(node.params, node);
        break;

      case 'NumberLiteral':
      case 'StringLiteral':
        break;

      default:
        throw new TypeError(node.type);
    }

    if (methods && methods.exit) {
      methods.exit(node, parent);
    }
  }

  traverseNode(ast, null);
}



/**
 *
 * ----------------------------------------------------------------------------
 *   Original AST                     |   Transformed AST
 * ----------------------------------------------------------------------------
 *   {                                |   {
 *     type: 'Program',               |     type: 'Program',
 *     body: [{                       |     body: [{
 *       type: 'CallExpression',      |       type: 'ExpressionStatement',
 *       name: 'add',                 |       expression: {
 *       params: [{                   |         type: 'CallExpression',
 *         type: 'NumberLiteral',     |         callee: {
 *         value: '2'                 |           type: 'Identifier',
 *       }, {                         |           name: 'add'
 *         type: 'CallExpression',    |         },
 *         name: 'subtract',          |         arguments: [{
 *         params: [{                 |           type: 'NumberLiteral',
 *           type: 'NumberLiteral',   |           value: '2'
 *           value: '4'               |         }, {
 *         }, {                       |           type: 'CallExpression',
 *           type: 'NumberLiteral',   |           callee: {
 *           value: '2'               |             type: 'Identifier',
 *         }]                         |             name: 'subtract'
 *       }]                           |           },
 *     }]                             |           arguments: [{
 *   }                                |             type: 'NumberLiteral',
 *                                    |             value: '4'
 * ---------------------------------- |           }, {
 *                                    |             type: 'NumberLiteral',
 *                                    |             value: '2'
 *                                    |           }]
 *  (sorry the other one is longer.)  |         }
 *                                    |       }
 *                                    |     }]
 *                                    |   }
 * ----------------------------------------------------------------------------
 */

function transformer(ast) {
  let newAst = {
    type: 'Program',
    body: [],
  };

  ast._context = newAst.body;

  traverser(ast, {
    NumberLiteral: {
      enter(node, parent) {
        parent._context.push({
          type: 'NumberLiteral',
          value: node.value,
        });
      },
    },

    StringLiteral: {
      enter(node, parent) {
        parent._context.push({
          type: 'StringLiteral',
          value: node.value,
        });
      },
    },

    CallExpression: {
      enter(node, parent) {
        let expression = {
          type: 'CallExpression',
          callee: {
            type: 'Identifier',
            name: node.name,
          },
          arguments: [],
        };

        node._context = expression.arguments;

        if (parent.type !== 'CallExpression') {
          expression = {
            type: 'ExpressionStatement',
            expression: expression,
          };
        }

        parent._context.push(expression);
      },
    },
  });

  return newAst;
}
```



### 3.5 代码生成

```JavaScript
function codeGenerator(node) {
  switch (node.type) {
    case 'Program':
      return node.body.map(codeGenerator).join('\n');

    case 'ExpressionStatement':
      return (
        codeGenerator(node.expression) + ';' // << (...because we like to code the *correct* way)
      );

    case 'CallExpression':
      return codeGenerator(node.callee) + '(' + node.arguments.map(codeGenerator).join(', ') + ')';

    case 'Identifier':
      return node.name;

    case 'NumberLiteral':
      return node.value;

    case 'StringLiteral':
      return '"' + node.value + '"';

    default:
      throw new TypeError(node.type);
  }
}
```



### 3.6 完整流程

```JavaScript
/**
 *
 *   1. input  => tokenizer   => tokens
 *   2. tokens => parser      => ast
 *   3. ast    => transformer => newAst
 *   4. newAst => generator   => output
 */

function compiler(input) {
  let tokens = tokenizer(input);
  let ast = parser(tokens);
  let newAst = transformer(ast);
  let output = codeGenerator(newAst);

  return output;
}
```



## 四、实战AST DEMO

### 4.1 介绍Babel

#### 用途

- 转译 esnext、typescript等到目标环境支持的js
- 代码的静态检查
  - linter工具、也是基于AST，对代码检查

从 babel7 开始，所有的官方插件和主要模块，都放在了 @babel 的命名空间下。从而可以避免在 npm 仓库中 babel 相关名称被抢注的问题，并且采用了Babel Monorepo风格的仓库。

- **@babel/parser**: 接受源码，进行词法分析、语法分析，生成AST。
- **@babel/traverse**：接受一个AST，并对其遍历，根据preset、plugin进行逻辑处理，进行替换、删除、添加节点。
- **@babel/generator**：接受最终生成的AST，并将其转换为代码字符串，同时此过程也可以创建**source map**。
- **@babel/types**：用于检验、构建和改变AST树的节点
- **@babel/core:** Babel 的编译器，核心 API 都在这里面，比如常见的 `transform`、`parse`，并实现了插件功能



### 4.2 babel插件

babel 插件是一个简单的函数，它必须返回一个匹配以下接口的对象。

![image-20240729144326995](https://gitee.com/dx-smallpig/typora-image/raw/master/images/image-20240729144326995.png)

```JavaScript
export default function(api, options, dirname) {
  return {
    visitor: {
      StringLiteral(path, state) {},
    }
  };
};
```

Babel 插件接受 3 个参数：

- api：一个对象，包含了 types (@babel/types)、traverse (@babel/traverse)、template(@babel/template) 等实用方法，我们能从这个对象中访问到 @babel/core dependecies 中包含的方法。
- options：插件参数。
- dirname：目录名。

返回的对象有 **name、manipulateOptions、pre、visitor、post、inherits** 等属性：

- name：插件名字。
- inherits：指定继承某个插件，通过 Object.assign 的方式，和当前插件的 options 合并。
- visitor：指定 traverse 时调用的函数。
- pre 和 post 分别在遍历前后调用，可以做一些插件调用前后的逻辑，比如可以往 file（表示文件的对象，在插件里面通过 state.file 拿到）中放一些东西，在遍历的过程中取出来。
- **manipulateOptions**：用于修改 options，是在插件里面修改配置的方式。



#### 4.2.1 Path路径

![image-20240729145808659](https://gitee.com/dx-smallpig/typora-image/raw/master/images/image-20240729145808659.png)

除了能在 Path 对象上访问到当前 AST 节点、父级 AST 节点、父级 Path 对象，还能访问到添加、更新、移动和删除节点等其他方法，这些方法提高了我们对 AST 增删改的效率。

```JavaScript
path {
    // 属性：
    node // 当前 AST 节点
    parent // 父 AST 节点
    parentPath // 父 AST 节点的 path
    scope // 作用域
    hub // 可以通过 path.hub.file 拿到最外层 File 对象， path.hub.getScope 拿到最外层作用域，path.hub.getCode 拿到源码字符串
    container // 当前 AST 节点所在的父节点属性的属性值
    key // 当前 AST 节点所在父节点属性的属性名或所在数组的下标
    listKey // 当前 AST 节点所在父节点属性的属性值为数组时 listkey 为该属性名，否则为 undefined
}=
```

我们常用的主要有`node`, `parentPath`, `parent`



#### 4.2.2 path的方法

`inList() ` 判断节点是否在数组中，如果 container 为数组，也就是有 listkey 的时候，返回 true

`get(key) ` 获取某个属性的 path

`set(key, node) ` 设置某个属性的值

`getSibling(key) ` 获取某个下标的兄弟节点

`getNextSibling() ` 获取下一个兄弟节点

`getPrevSibling() ` 获取上一个兄弟节点

`getAllPrevSiblings() ` 获取之前的所有兄弟节点

`getAllNextSiblings() ` 获取之后的所有兄弟节点

`find(callback) ` 从当前节点到根节点来查找节点（包括当前节点），调用 callback（传入 path）来决定是否终止查找

`findParent(callback) ` 从当前节点到根节点来查找节点（不包括当前节点），调用 callback（传入 path）来决定是否终止查找

`isXxx(opts) ` 判断当前节点是否是某个类型，可以传入属性和属性值进一步判断，比如path.isIdentifier({name: 'a'})

`assertXxx(opts) ` 同 isXxx，但是不返回布尔值，而是抛出异常

`insertBefore(nodes) ` 在之前插入节点，可以是单个节点或者节点数组

`insertAfter(nodes) ` 在之后插入节点，可以是单个节点或者节点数组

`replaceWith(replacement) ` 用某个节点替换当前节点

`replaceWithMultiple(nodes) ` 用多个节点替换当前节点

`replaceWithSourceString(replacement) ` 解析源码成 AST，然后替换当前节点

`remove() ` 删除当前节点

`traverse(visitor, state) ` 遍历当前节点的子节点，传入 visitor 和 state（state 是不同节点间传递数据的方式）

`skip() ` 跳过当前节点的子节点的遍历

`stop() ` 结束所有遍历



#### 4.2.3 State状态

在实际编写插件的过程中，某一类型节点的处理可能需要依赖其他类型节点的处理结果，但由于 visitor 属性之间互不关联，因此**需要 state 帮助我们在不同的 visitor 之间传递状态**。

state 是该 ast 的数据源的数据，如来自哪个文件、执行位置、source、整个 ast 等等

```JavaScript
const core = require("@babel/core"); //babel核心模块
const pathlib = require("path");

const sourceCode = `
var a = 1;
console.log(a);
var b = 2;
`;

//no-console 禁用 console fix=true：自动修复
const eslintPlugin = ({ fix }) => {
  return {
    //遍历前
    pre(file) {
      file.set("errors", []);
    },
    visitor: {
      CallExpression(path, state) {
        const errors = state.file.get("errors");
        const { node } = path;
        if (node.callee.object && node.callee.object.name === "console") {
          errors.push(
            path.buildCodeFrameError(`代码中不能出现console语句`, Error)  //抛出一个语法错误
          );
          if (fix) {
            //如果启动了fix，就删掉该节点
            path.parentPath.remove();
          }
        }
      },
    },
    //遍历后
    post(file) {
      console.log(...file.get("errors"));
    },
  };
};
let targetSource = core.transform(sourceCode, {
  plugins: [eslintPlugin({ fix: true })], //使用插件
});

console.log(targetSource.code);
```



#### 4.2.4 类型汇总

```JavaScript
Program: 代表整个 JavaScript 程序的根节点。
FunctionDeclaration / FunctionExpression: 代表函数声明和函数表达式。
VariableDeclaration / VariableDeclarator: 代表变量声明，其中 VariableDeclaration 是包含一个或多个 VariableDeclarator 的父节点。
Identifier: 代表标识符，如变量名、函数名等。
Literal: 代表字面量，如数字、字符串、布尔值等。
CallExpression: 代表函数调用。
MemberExpression: 代表成员访问，如 object.property 或 object[method]。
BinaryExpression: 代表二元表达式，如 a + b 或 a > b。
UnaryExpression: 代表一元表达式，如 !a 或 ++b。
AssignmentExpression: 代表赋值表达式，如 a = b 或 a += b。
BlockStatement: 代表代码块，通常由大括号 {} 包围。
IfStatement: 代表 if 条件语句。
ForStatement / WhileStatement: 代表 for 循环和 while 循环。
DoWhileStatement: 代表 do...while 循环。
SwitchStatement: 代表 switch 语句。
CaseClause / DefaultClause: 代表 switch 语句中的 case 和 default 分支。
ReturnStatement: 代表 return 语句。
BreakStatement / ContinueStatement: 代表 break 和 continue 语句，用于控制循环的执行。
TryStatement / CatchClause / FinallyBlock: 代表 try...catch...finally 异常处理结构。
ThrowStatement: 代表 throw 语句，用于抛出异常。
DebuggerStatement: 代表调试器语句，用于断点调试。
ThisExpression: 代表 this 关键字。
ArrayExpression: 代表数组字面量。
ObjectExpression: 代表对象字面量。
Property: 代表对象字面量中的属性。
SpreadElement: 代表在数组或对象字面量中的扩展操作符 ...。
TemplateLiteral: 代表模板字符串。
TaggedTemplateExpression: 代表标签模板字符串。
ArrowFunctionExpression: 代表箭头函数。
ClassDeclaration / ClassExpression: 代表类声明和类表达式。
Super: 代表 super 关键字，用于调用父类方法。
ImportDeclaration / ExportNamedDeclaration / ExportDefaultDeclaration: 代表 ES6 模块的导入和导出语句。
```

















