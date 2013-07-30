# JavaScript 编码规范指南

## 背景

JavaScript 是一种弱类型的脚本语言，由于其强大的表现力与非常自由的语法，使其容易变得 buggy 与难以维护，这份指南列出了编写 JavaScript 时需要遵守的规则以避免出现上文中的问题

## JavaScript 语言规范

### 声明变量

声明变量必须加上 `var` 关键字且写在函数作用域的顶部。如果你没有写 `var` ，变量就会暴露在全局上下文 `global` 中，这样很可能会和现有变量冲突。

### 分号

可以。

可以自由选择是否在行末加上分号，视已有的项目而定：如果项目代码在行末加上分号，那么就遵守项目代码的风格；如果项目代码没有加上，则不要在行末增加分号。

如果不需要在行末增加分号，那么当第一个字符为 `[`, `{`, `+`, `-` 时需要在行首加上分号。

参见：

* http://www.zhihu.com/question/20298345/answer/14670020
* http://alloyteam.github.io/JX/doc/specification/google-javascript.xml?showone=%E5%88%86%E5%8F%B7#%E5%88%86%E5%8F%B7

### 块内函数声明

不要在代码块内声明一个函数。

不要这么写：

```javascript
if (x) {
  function foo() {}
}
```

虽然很多 JS 引擎都支持块内声明函数，但它不属于 [ECMAScript](http://www.ecma-international.org/publications/standards/Ecma-262.htm) 规范（见 ECMA-262 ，第 13 和 14 条）。各个浏览器糟糕的实现相互不兼容，有些也与未来 ECMAScript 草案相违背。ECMAScript 只允许在脚本的根语句或函数中声明函数。如果确实需要在块中定义函数，建议使用函数表达式来初始化变量：

```javascript
var foo

if (x) {
  foo = function () {}
}
```

### 封装基本类型

不要。

没有任何理由去封装基本类型，另外还存在一些风险：

```javascript
var x = new Boolean(false)

if (x) {
  alert('hi') // Shown 'hi'
}
```

除非明确用于类型转换，其他情况请千万不要这样做！

```javascript
var x = Boolean(0)

if (x) {
  alert('hi') // This will never be alerted
}

typeof Boolean(0) === 'boolean'
typeof new Boolean(0) === 'object'
```

有时用作 `number`, `string` 或 `boolean` 时，类型的转换会非常实用。

### 多级原型结构

不是首选。

多级原型结构是指 JavaScript 中的继承关系。当你自定义一个 D 类，且把 B 类作为其原型，那么这就获得了一个多级原型结构，这些原型结构会变得越来越复杂！

记住“继承是魔鬼”。

### 方法定义

    Foo.prototype.bar = function() { ... }

有很多方法给构造器添加方法或成员，我们比较推荐上方的这种形式，因为这样更加节省内存。

### 闭包

可以，但小心使用。

闭包也许是 JS 中最有用的特性了。有一份比较好的介绍闭包原理的[文档](http://jibbering。com/faq/faq_notes/closures。html)。

有一点需要牢记，闭包保留了一个指向它封闭作用域的指针，所以，在给 DOM 元素附加闭包时，很可能会产生循环引用，进一步导致内存泄漏。比如下面的代码:

```javascript
function foo(element, a, b) {
  element.onclick = function() { /* uses a and b */ };
}
```

这里，即使没有使用 element ，闭包也保留了 element ，a 和 b 的引用。由于 element 也保留了对闭包的引用，这就产生了循环引用，这就不能被 GC 回收。这种情况下，可将代码重构为:

```javascript
function foo(element, a, b) {
  element.onclick = bar(a, b);
}

function bar(a, b) {
  return function() { /* uses a and b */ }
}
```

### this

仅在对象构造器，方法，闭包中使用。

`this` 的语义很特别。有时它引用一个全局对象（大多数情况下），调用者的作用域（使用 `eval` 时），DOM 树中的节点（添加事件处理函数时），新创建的对象（使用一个构造器），或者其他对象（如果函数被 `call()` 或 `apply()`）。

使用时很容易出错，所以只有在下面两个情况时才能使用：

* 在构造器中
* 对象的方法（包括创建的闭包）中

不要使用 `self` 或者 `_this` ，请使用变量名 `that` 将当前上下文传入到闭包中：

```javascript
var a = function() {
  var that = this
  $(some).click(function(event) {
    some.click(that)
  })
}
```

原因：

* CoffeeScript 使用变量名 `_this`
* 如果 `self` 没有定义，会取到 `window.self`

### for-in 循环

只允许用于 `object` 的遍历。

对 `Array` 用 `for-in` 循环时会出错，因为它并不是从 `0` 到 `length - 1` 进行遍历，而是所有出现在其对象及其原型链的键值，即使使用了方法 `hasOwnProperty` ，还是会遍历到属性 `length` 。

### 多行字符串

不要这样写长字符串：

```javascript
var myString = 'A rather long string of English text, an error message \
                actually that just keeps going and going -- an error \
                message to make the Energizer bunny blush (right through \
                those Schwarzenegger shades)! Where was I? Oh yes, \
                you\'ve got an error and all the extraneous whitespace is \
                just gravy.  Have a nice day.'
```

编译时是不能忽略行起始位置的空白字符的；"\" 后的空白字符会产生奇怪的错误；虽然大多数脚本引擎支持这种写法，但它不是 ECMAScript 的标准规范。

### 修改内置对象的原型

不要。

千万不要修改内置对象，如 `Object.prototype` 和 `Array.prototype` 的原型。而修改内置对象，如 `Function.prototype` 的原型，虽然少危险些，但仍会导致调试时的诡异现象。所以也要避免修改其原型。

记住，只有在 shim 时才可以使用这个黑魔法。

### IE 下的条件注释

不要。

不要这样子写：

```javascript
var f = function () {
  /*@cc_on if (@_jscript) { return 2* @*/  3; /*@ } @*/
}
```

条件注释妨碍自动化工具的执行，因为在运行时，它们会改变 JavaScript 语法树。

### 等于符号

使用 `===` 。

你没有义务去记住那些脑残的[规则](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators?redirectlocale=en-US&redirectslug=JavaScript%2FReference%2FOperators%2FComparison_Operators)，所以推荐你使用 `===` 。

除非在判断一个变量的值是否为 `null` 和 `undefined` 之一的时候：

```javascript
null == null
undefined == null
```

### 条件判断

将超过一行的条件语句的结果赋值给一个易读的变量。

```
// ok
if (user.isAdmin() || user.isModerator()) {
  console.log('winning')
}

// terrible
if (searchableCollection(allYourStuff).contains(theStuffYouWant) &&
    !ambientNotification.isActive() && (client.isAmbientSupported() ||
    client.alwaysTryAmbientAnyways()) {

  ambientNotification.activate();
})
```

### Undefined

总是使用 `void 0` 替代 `undefined` 。

因为 `undefined` 是能够被重新赋值的。

### 全局变量

不推荐。

尽量避免全局变量，如果真的要使用，请在代码中明明白白的写出来：

```javascript
// NO
globalVar = "aaa"

// YES
window.globalvar = "globalvar"
global.globalvar = "globalvar"
```

## JavaScript 编码风格

### 变量命名

变量名使用小驼峰式的写法（lower camel-case）。

### 类名（构造函数）

使用首字母大写的驼峰式（upper camel-case），且必须为单数。

### 常量

常量名全部大写，单词间用下划线分隔。如："CSS_BTN_CLOSE", "TXT_LOADING"。

### 括号

只在需要的时候使用。

不要滥用括号，只在必要的时候使用它。

对于一元操作符（如 `delete`, `typeof` 和 `void` ），或是在某些关键词（如 `return`, `throw`, `case`, `new` ）之后，不要使用括号。

### 字符串字面量

使用 `'` 优于 `"` 。

因为在很多其他的语言里面，是不允许单引号的，因此使用单引号在 JavaScript 中嵌入其他语言的语句会更加方便（不需要转义其他语言的字符串字面量）。

### 空格

* 不允许空格，`tab` 混用
* 除非特殊情况（函数参数等，会在下文说明），缩进都以两个空格为单位
* 数值操作符（如 `+`, `-`, `*`, `%` 等）两边留空
* 赋值操作符/等价判断符两边留一空格
* for 循环条件中, 分号后留一空格
* 空行不要有空格
* 行尾不要有空格
* 逗号和冒号后一定要跟空格
* 点号前后不要出现空格
* 空对象和数组不需要填入空格
* 函数名末尾和左括号之间不要出现空格
* 匿名函数的 `function` 关键字后不要出现空格

### 大括号

分号会被隐式的插入到代码中，所以务必在同一行上插入大括号，例如：

```javascript
if (x) {
} else {
}
```

### 数据和对象的初始化

如果初始值不长，就保持写在单行上：

```javascript
var arr = [1, 2, 3],
    obj = {a: 1, b: 2, c: 3}
```

初始值占用多行时，缩进2个空格：

```javascript
// Object initializer.
var inset = {
  top: 10,
  right: 20,
  bottom: 15,
  left: 12
}

// Array initializer.
this.rows = [
  '"Slartibartfast" <fjordmaster@magrathea.com>',
  '"Zaphod Beeblebrox" <theprez@universe.gov>',
  '"Ford Prefect" <ford@theguide.com>',
  '"Arthur Dent" <has.no.tea@gmail.com>',
  '"Marvin the Paranoid Android" <marv@googlemail.com>',
  'the.mice@magrathea.com'
]
```

比较长的标识符或者数值，不要为了让代码好看些而手工对齐。如：

```javascript
CORRECT_Object.prototype = {
  a: 0,
  b: 1,
  lengthyName: 2
}
```

不要这样做:

```javascript
WRONG_Object.prototype = {
  a          : 0,
  b          : 1,
  lengthyName: 2
}
```

### 函数参数

尽量让函数参数在同一行上。如果一行超过 80 字符，每个参数独占一行，并以 4 个空格缩进，或者与括号对齐，以提高可读性。尽可能不要让每行超过 80 个字符，比如下面这样：

```javascript
// Four-space, one argument per line.  Works with long function names,
// survives renaming, and emphasizes each argument.
doThingThatIsVeryDifficultToExplain = function(
    veryDescriptiveArgumentNumberOne,
    veryDescriptiveArgumentTwo,
    tableModelEventHandlerProxy,
    artichokeDescriptorAdapterIterator) {

  // ...
}

// Parenthesis-aligned, one argument per line.  Visually groups and
// emphasizes each individual argument.
function bar(veryDescriptiveArgumentNumberOne,
             veryDescriptiveArgumentTwo,
             tableModelEventHandlerProxy,
             artichokeDescriptorAdapterIterator) {
  // ...
}
```

### 传递匿名函数

如果参数中有匿名函数，函数体从调用该函数的左边开始缩进 2 个空格，而不是从 `function` 这个关键字开始。这让匿名函数更加易读（不要增加很多没必要的缩进让函数体显示在屏幕的右侧）。

```javascript
// wrong
var names = items.map(function(item) {
                        return item.name;
                      })

// right
prefix.something.reallyLongFunctionName('whatever', function(a1, a2) {
  if (a1.equals(a2)) {
    someOtherLongFunctionName(a1);
  } else {
    andNowForSomethingCompletelyDifferent(a2.parrot);
  }
})
```

### 更多的缩进

事实上，除了初始化数组和对象和传递匿名函数外，所有被拆开的多行文本要么选择与之前的表达式左对齐，要么以 4 个（而不是 2 个）空格作为一缩进层次。

```javascript
someWonderfulHtml = '' +
                    getEvenMoreHtml(someReallyInterestingValues, moreValues,
                                    evenMoreParams, 'a duck', true, 72,
                                    slightlyMoreMonkeys(0xfff)) +
                    ''

thisIsAVeryLongVariableName =
    hereIsAnEvenLongerOtherFunctionNameThatWillNotFitOnPrevLine()

thisIsAVeryLongVariableName = 'expressionPartOne' +
    someMethodThatIsLong() +
    thisIsAnEvenLongerOtherFunctionNameThatCannotBeIndentedMore()

someValue = this.foo(
    shortArg,
    'Some really long string arg - this is a pretty common case, actually.',
    shorty2,
    this.bar()
)
```

### 二元和三元操作符

操作符始终跟随着前行，这样就不用顾虑分号的隐式插入问题。如果一行实在放不下，还是按照上述的缩进风格来换行。

```javascript
var x = a ? b : c  // All on one line if it will fit.

// Indentation +4 is OK.
var y = a ?
    longButSimpleOperandB :
    longButSimpleOperandC

// Indenting to the line position of the first operand is also OK.
var z = a ?
        moreComplicatedB :
        moreComplicatedC
```

### 空行

文件末尾留 1~2 个空行。

不要吝啬空行，尽量使用空行将逻辑相关的代码块分割开，以提高程序的可读性。

```javascript
doSomethingTo(x)
doSomethingElseTo(x)
andThen(x)

nowDoSomethingWith(y)

andNowWith(z)
```

## CoffeeScript 编码风格

* CoffeeScript 只是一种 JavaScript 的方言，所以上文在大部分情况下依旧适用
* 更多与 JavaScript 不同的地方请遵循 https://github.com/polarmobile/coffeescript-style-guide 。如果链接中的某些编码风格和 JavaScript 编码风格冲突，请遵循 JavaScript 编码风格
* 使用原生的或者库函数（`Array::forEach`, `_.forEach`, `Object.keys()`, `_.keys()`）替代 `for...in` 或者 `for...of` 循环，因为 `for` 循环是不会创建作用域的，同时它还会创建一些“隐形”的变量，比如 `_i` ，因而它更容易引入 BUG ；同时代码编译为 JavaScript 后也不易于阅读。

## 参考

感谢以下文档提供的想法：

* http://alloyteam.github.io/JX/doc/specification/google-javascript.xml
* http://docs.kissyui.com/docs/html/tutorials/style-guide/js-style-rules.html
* http://nodeguide.com/style.html
* https://github.com/rwldrn/idiomatic.js/tree/master/translations/zh_CN
