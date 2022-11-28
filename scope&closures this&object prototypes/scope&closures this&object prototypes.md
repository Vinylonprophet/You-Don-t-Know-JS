# SCOPE & CLOSURES THIS & OBJECT PROTOTYPES

##  Scope

### 作用域

#### 编译原理

传统编译语言的流程中，源码执行前经历三个步骤，统称为“编译”

- 分词/词法分析

  var a = 2

  会拆分成 var、a、=、2 （空格是否会被当作词法单元，取决于空格在语言中是否具有意义）

- 解析/语法分析

  这个过程将词法单元流转化成一个元素逐级嵌套所组成代表程序语法结构的树，树被成为**“抽象语法树”**

  var a = 2; 的抽象语法树会有一个叫VariableDeclaration的顶级节点，接下来是叫做Identifier（值是a）的子节点，以及一个叫做AssignmentExpression的子节点。AssignmentExpression节点有一个叫NumericLiteral（值为2）的子节点

- 代码生成

  将抽象语法树转换为可执行代码的过程被称为代码生成，过程与语言、目标平台等息息相关

  就是将var a = 2; 的抽象语法树转化为一组机器指令，用来创建一个叫作a的变量，并将一个值存储在a中

比起以上传统编译语言，JavaScript引擎更为复杂，在语法分析和代码生成阶段有特定的步骤来对运行性能进行优化，包括对冗余元素进行优化

首先，JavaScript 不会有大量时间优化，因为 **JavaScript的编译过程不是发生在构建之前**，对于JavaScript而言，大部分情况下编译发生在代码执行前几微秒。



#### 理解作用域

学习作用域的方法：将过程模拟成几个人物之间的对话

##### 演员表

对 var a = 2; 进行处理的演员

- 引擎

  从头到尾负责整个JavaScript程序的编译和执行过程

- 编译器

  引擎的好朋友之一，负责语法分析和代码生成等脏活累活

- 作用域

  引擎的另一位朋友，负责收集并维护由所有声明的标识符（变量）组成一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限

##### 对话

一般而言，var a = 2; 我们认为是一句声明，引擎认为这里有两个完全不同的声明，一个由编译器在编译时处理，一个由引擎在运行时处理

编译器：

1. 遇到var a，会询问作用域是否已经有一个该名称的变量存在于同一个作用域中的集合中，如果是，会忽略该声明继续编译，否则会要求作用域在当前作用域集合中声明一个新的变量，并命名为a
2. 接下来编译器会为引擎生成运行时需要的代码，这些代码被用来处理 a = 2 ，这一操作。引擎运行时会首先询问作用域，在当前作用域集合中是否存在一个叫 a 的变量。如果是就会使用这个变量，如果否会继续查找该变量

如果引擎最后找到了 a，就会将 2 赋给它，否则引擎就会抛出一个异常

**总结：**变量赋值会执行两个动作，首先在当前作用域中声明一个变量，然后在运行时会在作用域中查找该变量，如果找到就进行赋值

##### 编译器有话说

引擎执行编译器生成的代码时，会通过变量a来判断是否已经声明过。查找的过程由作用域协助，但是怎么查找，会影响最终的查找结果

引擎会为a进行LHS查询，另一个查找的类型叫做RHS

L：左	R：右

指的是赋值操作的左侧和右侧

准确的讲：RHS查询与简单地查找某个变量的值别无二致，而LHS查询则是视图找到变量的容器本身，从而对其赋值。在这个角度来说，RHS不是真正意义上的赋值操作的右侧，而是非左侧，

**例子1：**

```javascript
console.log(a);
```

对于a的引用是一个RHS，因为a没有赋予任何值，相应的要查找并取得a的值才能传递给console.log(..)

**例子2：**

```javascript
a = 2;
```

这里对a的引用是LHS，因为实际上我们不关心当前值，只想要为2这个赋值操作找到一个目标

**注意：**概念上最好理解为**（赋值操作的目标是谁）LHS**和**（赋值操作的源头是谁）RHS**

**综合案例：**

```javascript
function foo(a){
	console.log(a);
}
foo(2);
```

最后一行foo(..)进行RHS，“意味着去找foo的值并且把他给我“

代码中隐式的 a = 2 是存在的，操作发生在2被当作参数传递给foo(..)函数时，2会分配给参数a，为了给a分配值，需要进行一次LHS查询

这里还有对a进行的RHS引用，将得到的值传给了console.log(..)

console.log(..)本身也需要一个引用才能执行，因此会对console对象进行RHS查询，并且检查是否有log方法

最后在概念上理解为LHS和RHS之间通过对值2进行交互来将其传递进log(..)

##### 小测验

找到下列代码的所有LHS查询和所有RHS查询

```javascript
function foo(a){
	var b = a;
	return a + b;
}

var c = foo(2);
```

- LHS:   b = ..   、c = ..   、a = 2(隐式)
- RHS：foo(2..   、= a   、a   、b



#### 作用域嵌套

作用域是根据名称查找变量的一套规则，实际情况中，通常需要顾及几个作用域

当一个块或作用域嵌套在另一个块或函数中，就发生了作用域嵌套

```javascript
function foo(a){
	console.log( a + b );
}

var b = 2;

foo(2)
```

对b进行的RHS引用无法在foo函数内部完成，但可以在上一级作用域中完成

##### 把作用域链比喻成一个建筑

第一次层就代表当前的作用域，没有找到就继续往上一层层查找，直到顶层（全局作用域），然后停止（无论是否找到）



#### 异常

区分LHS和RHS是一件重要的事，因为在变量还没声明的情况下，这两种查询的行为不同

```javascript
function foo(a){
	console.log( a + b );
	b = a;
}
foo(2);
```

第一次对b进行查找无论如何都找不到该变量，也就是说这是一个”未声明“的变量，因为在任何作用域中都无法找到它

如果RHS查询在所有嵌套的作用域中遍寻不到所需要的变量，**引擎**会抛出 **ReferenceError** 异常， **ReferenceError** 是非常重要的异常类型

相较之下，引擎执行 LHS 查询时，如果在全局作用域也无法找到目标变量，会创建一个具有该名称的变量，将其返回给引擎，前提是程序运行在**”非严格模式“**下

**严格模式：**ES5开始引入的，此模式在行为上有很多不同，其中一个就是禁止自动或隐式地创建全局变量，因此在严格模式中 LHS 查询失败的话，不会返回一个全局变量，会抛出同 RHS 查询失败时类似的 **ReferenceError** 异常

RHS 查询到了一个变量，但是如果对这个变量的值进行不合理操作，比如试图对一个非函数类型的值进行函数调用，或者引用 null 或 undefined 类型的值中的属性，那么引擎会抛出另外一种类型的异常，叫做 **TypeError**

**重点：**

- **ReferenceError：**同作用域判别失败相关
- **TypeError：**作用域判别成功，但是对操作的结果是非法或者不合理的



### 词法作用域

作用域共有两种主要的工作模型：

- 第一种：最为普通，被大多数编程语言所采用，我们会对这种作用域深入讨论
- 第二种：动态作用域，仍有一些语言在使用（Bash脚本，Perl中的一些模式）



#### 词法阶段

大部分标准语言编译器第一个工作阶段叫词法化（单词化），词法化过程会对源代码中的字符进行检查，如果是有状态的解析过程，还会赋予单词语义

这个概念是理解词法作用域及其名称来历的基础

简单理解，**词法作用域** 就是定义在词法阶段的作用域。换句话说，是在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变（大部分情况是这样）

**注意：**后面会介绍一些欺骗词法作用域的方法，这些方法在词法分析器处理过后依然可以修改作用域，但是这种机制可能会难以理解。事实上，让词法作用域根据词法关系保持书写时的自然关系不变，是一个非常好的最佳实践

**例子：**

```javascript
function foo(a){
	var b = a * 2;
	
	function bar(c){
		console.log(a, b, c);
	}
    
	bar(b * 3);
}
foo(2);
```

1. 全局作用域，只有一个标识符：foo
2. 包含foo创建的作用域，三个标识符：a、bar、b
3. 包含bar创建的作用域，只有一个标识符：c

作用域气泡是逐级包含的，下一章讨论不同类型的作用域，现在假设每一个函数都会创建一个新的作用域气泡

bar的气泡被完全包含在foo所创建的气泡中，唯一的原因是那里就是我们希望定义函数bar的位置

##### 查找

作用域气泡的结构和相互之间的位置关系给引擎提供了足够的位置信息，引擎用这些信息来查找标识符的位置

关于上面的例子：

```javascript
function foo(a){
	var b = a * 2;
	
	function bar(c){
		console.log(a, b, c);
	}
    
	bar(b * 3);
}
foo(2);
```

console.log(a, b, c) 从 bar 开始查找，然后去 foo 没然后找到了 a，因此引擎使用了这个引用

`作用域查找会在找到第一个匹配的标识符时停止。` 在多层嵌套作用域中可以定义同名的标识符，这叫做`遮蔽效应（内部标识符'遮蔽'了外部标识符）`。

**注意：**全局变量会自动成为全局对象，比如浏览器中window对象的`属性`，因此可以不直接通过全局变量的词法名称，而是间接地通过对全局对象属性的引用来对其进行访问

`window.a`

通过这种技术可以访问那些被同名变量所遮蔽的全局变量，但非全局变量如果被遮蔽，则无法访问到



#### 欺骗词法

如果词法作用域完全由写代码期间函数所声明的位置来定义，怎么才能在运行时“修改”词法作用域呢

JavaScript有两种机制来实现这个目的，但是并不是什么好主意，因为有一点**很重要**：欺骗词法作用域会导致性能下降

以下来介绍两种机制：

##### eval

eval(..)函数可以接受一个字符串作为参数，将其中内容视作好像是书写时就在这个位子，引擎按照往常的方法进行查找

```javascript
function foo(str, a){
	eval(str);
	console.log(a, b);
}

var b = 2;

foo("var b = 3;", a);	// 1, 3
```

eval(str) 调用的 var b = 3 会被当作本来就在那里一样来处理，由于那段代码声明了一个新的变量 b，因此它对已经存在的foo(..)词法作用域进行修改，原理：在foo(..)内部创建一个变量b，并遮蔽了外部作用域中的同名变量

**注意：**在严格模式的程序中，eval(..)运行时有自己的词法作用域，意味着其中的声明无法修改所在的作用域

```javascript
function foo(str){
	"use strict";
	eval(str);
	console.log(a);		// ReferenceError: a is not defined
}

foo("var a = 2");
```

JavaScript中还有一些功能效果和eval(..)很相似，setTimeout(..)和setInterval(..)的第一个参数可以是字符串，字符串的内容可以被解释为一段动态生成的函数代码，这些功能已经过时且不被提倡

new function(..)函数行为也很类似，最后一个参数可以接受代码字符串，将其转化为动态生成的函数，虽然比eval(..)安全点，但也要尽量避免使用

##### with

解释角度：with如何同被它所影响的词法作用域进行交互

`with` 通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身

**例子1：**

```javascript
var obj = {
	a: 1,
	b: 2,
	c: 3
}

// 单调乏味的重复"obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// 简单的快捷键
with(obj){
	a = 3;
	b = 4;
	c = 5;
}
```

不仅仅是为了方便地访问对象属性，还考虑以下代码：

```javascript
function foo(obj){
	with(obj){
		a = 2;
	}
}

var o1 = {
	a: 3
}

var o2 = {
	b: 3
}

foo(o1);
console.log(o1.a);	// 2

foo(o2);
console.log(o2.a);	// undefined
console.log(a);		// 2——不好，a被泄漏到全局作用域上了
```

**泄露原因：**with 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对象的属性也会被处理为定义在这个作用域中的词法标识符

尽管 with 可以将一个对象处理为词法作用域，但是这个内部正常的 var 声明并不会被限制在这个块作用域中，而是被添加到 with 所处的函数作用域中

换而言之就是在 o2 中查询 a 没有查到，因此自动创建了一个全局变量（因为是非严格模式）

**注意：**在非严格模式下，eval(..)和with会被严格模式影响，with完全被禁止，而在保留核心功能的前提下，简洁或非安全地使用eval(..)也被禁止了

##### 性能

JavaScript引擎在编译阶段进行数项的性能优化，其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能执行过程中快速找到标识符。

但如果过程中发现了eval(..)或with，只能`简单的假设关于标识符位置的判断无效`，因为无法在词法分析阶段明确知道eval(..)会接收到什么代码，这些代码会如何对作用域进行修改，也无法知道传递给 with 用来创建新词法作用域的对象的内容到底是什么

最悲观的情况就是出现了 eval(..) 和 with ，所有的优化都毫无意义，因此最简单就是完全不做任何优化

如果代码中大量使用 eval(..) 和 with，那么运行速度一定会很慢

所以**不要使用**



### 函数作用域和块作用域

#### 函数中的作用域

JavaScript具有基于函数的作用域，意味着每声明一个函数都会为自身创建一个气泡，而其他结构都不会创建作用域气泡

**例子：**

```javascript
function foo(a){
    var b = 2;
    
    function bar(){
        
    }
    
    var c = 3;
}
```

foo(..)的作用域气泡包含了标识符a，b，c和bar

bar(..)有自己的作用域气泡。全局作用域也有自己的气泡，它只包含了一个标识符foo

函数的作用域的含义是指，属于函数的全部变量都可以在整个函数范围内使用以及复用，能充分利用JavaScript变量可以根据需要改变值类型的“动态”特性



#### 隐藏内部实现

可以把变量和函数包裹在一个函数的作用域中，然后用这个作用域来”隐藏“它们

`隐藏` 变量和函数是一个有用的技术

很多原因促成了这种基于作用域的隐藏方法。大多都是从**最小特权原则**中引申出来的，也叫**最小授权原则**或**最小暴露原则**。这个原则是指在软件设计中，应该最小限度地暴露必要内容，从而将其他内容”隐藏“起来，比如某个模块或对象的API设计

这个原则可以延伸到作用域包含变量和函数。如果所有变量和函数都在全局作用域中，虽然可以在所有内部嵌套作用域访问到他们，但会破坏这种**最小特权原则**，因为会暴露过多的变量和函数

**例子：**

```javascript
function doSomething(a){
	b = a + doSomethingElse( a * 2);
	console.log(b * 3);
}

funciton doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething(2);		// 15
```

在以上代码中，变量b和函数doSomethingElse(..)应该是doSomething(..)内部具体实现的"私有"内容，给予外部作用域对b和doSomethingElse(..)的”访问权限“不仅没有必要，而且可能是”危险“的，因为`可能被有意或无意地以非预期的方式使用，从而导致超出了doSomething(..)的适用条件`，更合理的应该如下：

```javascript
function doSomething(a){

	funciton doSomethingElse(a) {
        return a - 1;
    }

    var b;

	b = a + doSomethingElse( a * 2);
	
	console.log(b * 3);
}

doSomething(2);		// 15
```

现在，b和doSomethingElse(..)都无法被外部访问，只能被doSomething(..)所控制。功能和最终效果都没有受影响，但是设计上将具体内容私有化了。

##### 规避冲突

"隐藏"作用域中变量和函数的另一个好处，是可以规避同名标识符之间的冲突，两个标识符可能具有相同的名字但用途却不一样，无意间可能会造成命名冲突。冲突会导致变量的值被意外覆盖

**例子：**

```javascript
function foo(){
	function bar(a){
		i = 3;		// 修改for循环所属作用域中的i
		console.log(a + i);
	}
	
	for(var i=0; i<10; i++){
		bar(i * 2);		// 糟糕，无限循环了
	}
}

foo();
```

bar(..)内部的赋值表达式 i = 3 意外覆盖了声明在foo(..)内部for循环中的 i 。在这个例子中会导致无限循环，因为i被固定设置为3，永远小于10

bar(..)内部的赋值操作需要声明一个本地变量使用，采用任何名字都可以，var i = 3; 就可以满足这个条件；另一种是采用一个完全不同的标识符名称，比如 var j = 3; 但是软件设计在某种情况下要求使用同样的标识符名称，因此在这种情况下使用作用域”隐藏“内部声明是唯一的最佳选择

###### 全局命名空间

典型的变量冲突存在于全局作用域中，当程序加载了多个第三方库时，如果没有妥善将内部私有的函数或变量隐藏，就会很容易发生冲突

这些库通常会在全局作用域中声明一个名字足够独特的变量，通常是一个对象。对象被用作库的命名空间，所有需要暴露给外界的功能都会成为这个对象的属性，而不是将自己的标识符暴露在顶级的词法作用域中

```javascript
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function(){
		// ...
	},
	doSomethingElse: function(){
		// ...
	}
}
```

###### 模块管理

从众多模块管理器中挑选一个使用。使用这些工具，任何库都无需将标识符加入到全局作用域中，而是通过依赖管理器的机制将库的标识符显式地导入到另一个特定的作用域中

这些工具并没有能够违反词法作用域的”神奇“功能。只是利用作用域的规则强制所有标识符都不能进入共享作用域中，而是保持在私有、无冲突的作用域中，这样可以有效规避掉所有的意外冲突



**注意：**由于挨章打字过于耗费时间，所以从此处开始，下面的笔记进行高度概括



#### 函数作用域

已知在任意代码片段外部添加包装函数，可以将内部的变量和函数定义“隐藏”起来，外部作用域无法访问包装函数内部的任何内容

**例子：**

```javascript
var a = 2;

function foo(){
	var a = 3;
	console.log(a);		// 3
}
foo();

console.log(a);			// 2
```

虽然通过声明具体函数foo()，来使外部无法访问包装函数内部，但是会出现**两个问题：**

1. foo这个名称污染了所在的作用域
2. 必须通过调用foo()函数才能运行其中的代码

以下的例子既不需要函数名，又可以不通过调用来解决上述问题：

```javascript
var a = 2;

(function foo(){
	var a = 3;
	console.log(a);		// 3
})();

console.log(a);			// 2
```

包装函数以 **(function** 开始，此时以及不是`函数声明`而是`函数表达式`

比较前面两者，第一个片段中 foo 被绑定在所在作用域，第二个片 foo 被绑定在函数表达式自身的函数中而不是作用域中，也就是说(function foo() { .. })作为函数表达式意味着foo只能在 .. 所代表的位置中被访问，外部作用域不行，也就不会污染外部作用域了



##### 匿名和具名

**回调函数：**

```javascript
setTimeout( function(){
	console.log('I wait 1 second');
}, 1000);
```

此处为`匿名函数表达式`，因为function()没有名称标识符。**函数表达式**可以是匿名的，但是**函数声明**不可以省略函数名

**缺点：**

1. 函数在栈追踪不会显示出有意义的函数名，调试困难
2. 如果没有函数名，函数需要引用自身时只能使用已经过期的`arguments.callee`引用，比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑自身
3. 匿名函数`省略了`对于代码的可读性 / 可理解性很重要的`函数名`。

行内函数表达式可以给函数表达式指定一个函数名解决问题

```javascript
setTimeout( function timeoutHandler(){
	console.log('I wait 1 second'); 
}, 1000);
```

##### 立即执行函数表达式

**例子：**

```javascript
var a = 2;

(function foo(){
	var a = 3;
	console.log(a);		// 3
})();

console.log(a);			// 2
```

第一个()将函数变成表达式，第二个()执行了这个函数

这种模式有个专业术语：**IIFE**，代表立即执行函数表达式(Immediately Invoked Function Expression)

函数名对IIFE不是必须的，IIFE最常见的用法是使用一个匿名函数表达式。虽然使用具名函数的IIFE并不常见，但是它具有上述匿名函数表达式的所有优势，也是一个值得推广的实践

```javascript
var a = 2;
(function IIFE(){

	var a = 3;
	console.log(a);		// 3
})();

console.log(a);			// 2
```

相较于传统的形式，很多人还喜欢另一个`改进形式`：

```javascript
(function(){..}())
```

功能上是一致的

IIFE**进阶用法：**把它们当作函数调用并传递参数进去

**例子：**

```javascript
var a = 2;

(function IIFE(global){
	
	var a = 3;
	console.log(a);			// 3
	console.log(global.a);	// 2
	
})(window);

console.log(a);				// 2
```

将window对象传递进去，但是将参数命名为global，因此在代码风格上对全局对象的引用变得比引用一个没有”全局“字样的变量更加清晰。可以从外部作用域传递任何需要的东西，并且将变量命名为任何自己觉得合适的名字，对于改进代码风格非常有用

另一个场景：将一个参数命名为undefined，但是在对应位置不传入任何值，这样可以保证在代码块中undefined标识符的值真的为undefined：

```javascript
undefined = true;		// 不要这样做

(function IIFE(undefined){

	var a;
	if(a === undefined){
		console.log("Undefined is safe here!");
	}
	
})()
```

IIFE还能用来倒置代码的运行顺序，将需要运行的函数放在第二位，在IIFE执行之后当作参数传递进去。在**UMD(Universal Module Definition)**项目中被广泛使用。

```javascript
var a = 2;

(function IIFE(def){
	def(window);
})(function def(global){
	
	var a = 3;
	console.log(a);			// 3
	console.log(global.a);	// 2
    
})
```

1. def(global)被当作参数传递进IIFE函数定义的第一部分中
2. def被调用，将window传入当作global的值



#### 块作用域

除了JavaScript外很多语言都支持块作用域，但是这对JavaScript的开发者而言会很陌生

```javascript
for(var i=0; i<10; i++){
	console.log(i);
}
```

我们在for的头部定义了i，通常是因为只想在for循环内部的上下文使用i，而忽略了i会被绑定在外部作用域（函数或全局）的事实

这是块作用域的用处，变量的声明应该距离使用的地方越近越好，并且最大限度本地化

```javascript
var foo = true;

if(foo){
	var bar = foo * 2;
	bar = something(bar);
	console.log(bar);
}
```

上述代码将bar声明在if内部是非常有意义的一件事情，但当var声明变量，它写在哪里都一样，因为都属于外部作用域，这段代码是为了风格更易读而伪装出的形式上的块作用域，如果使用这种形式，要确保没在作用域其他地方意外使用bar只能靠自觉性

这是块作用域的用处，变量的声明应该距离使用的地方越近越好，并且最大限度本地化

块作用域是一个用来对之前的**最小授权原则**进行扩展的工具，将代码从在函数中隐藏信息扩展为在块中隐藏信息

再次考虑for循环的例子，为什么要把只在for循环内容使用的变量 i 污染整个作用域呢

可惜表面上 JavaScript 没有块作用域的相关功能

##### with

之前提到过的with也是块作用域的一个例子，用with从对象中创建出的作用域仅在with声明中而非外部作用域有效

##### try/catch

ES3中规定的try/catch分句会创建一个块作用域，其中声明的变量仅在catch内部有效

**例子：**

```javascript
try{
	undefined();		// 执行一个非法操作来强制制造一个异常
}
catch(err){
	console.log(err);	// 能够正常执行！
}

console.log(err);		// ReferenceError: err not found
```

err仅存在catch中，其他地方引用会报错

##### let

幸好ES6的let提供了除了var以外的变量声明方式

let关键字可以将变量绑定到所在的任意作用域中**（通常{..}内部）**，换句话说，let为其声明的变量隐式地劫持了所在的块作用域

**例子：**

```javascript
var foo = true;

if(foo){
	let bar = foo * 2;
	bar = something(bar);
	console.log(bar);
}

console.log(bar);		// ReferenceError
```

让let将变量附加在一个已经存在的块级作用域上的行为是隐式的。在开发和修改代码的过程中，如果没有密切关注哪些块级作用域中有绑定的变量，并且习惯性的移动这些块或将其包含在其他的块中，就会导致代码混乱。

`为块作用域显式地创建块`可以部分解决这个问题，使变量的附属关系变得更加清晰，如下：

```javascript
var foo = true;

if(foo){
	{
        let bar = foo * 2;
		bar = something(bar);
		console.log(bar);
    }
}

console.log(bar);		// ReferenceError
```

好处在于在if内部`显式`地创建了一个块，如果要进行重构，整个块都可以被方便的移动而不会对外部if声明地位置和语义产生任何影响

**注意：**let的声明不会在块作用域中进行提升，声明的代码被运行之前，声明`不存在`，如下：

```javascript
{
	console.log(bar);		// ReferenceError!
	let bar = 2;
}
```

###### 垃圾收集

`块级作用域非常有用的原因和闭包及回收内存垃圾的回收机制相关`，内部的实现原理也就是闭包的机制会在第五章谈到

**例子：**

```javascript
function process(data){
	// ...
}

var someReallyBigData = {..};

process(someReallyBigData);

var btn = document.getElementById("my_button");

btn.addEventListener("click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase*/false);
```

click的函数点击回调不需要someReallyBigData变量，意味着当process(..)执行后，内存中的大量数据结构可以被回收了。但是，由于click函数形成了一个覆盖整个作用域的闭包，所以引擎很有可能还保留这个结构（取决于实现）

块级作用域可以打消这种顾虑，如下：

```javascript
function process(data){
	// ...
}

// 这个块中定义的可以完全销毁
{
    let someReallyBigData = {..};

	process(someReallyBigData);
}

var btn = document.getElementById("my_button");

btn.addEventListener("click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase*/false);
```

###### let循环

一个let可以发挥优势的典型例子就是for循环

```javascript
for(let i=0; i<10; i++){
	console.log(i);
}
console.log(i);		// ReferenceError
```

**注意：**for循环的let不仅将i绑定到了for循环的块中，事实上还将其`重新绑定`到了每一个迭代中，确保上一个循环迭代结束时的值重新进行赋值

重新绑定的行为如下：

```javascript
{
	let i = 0;
	for(j=0; j<10; j++){
		let i = j;		// 每个迭代重新绑定
		console.log(i);
	}
}
```

如果用let来替代var在代码重构时，需要更多精力：

```javascript
var foo = true, baz = 10;

if(foo){
	var bar = 3;
	
	if(baz > bar){
		console.log(baz);
	}
	
	// ...
}
```

重构如下：

```javascript
var foo = true, baz = 10;

if(foo){
	var bar = 3;
	// ...
}

if(baz > bar){
	console.log(baz);
}
```

使用块级作用域的变量时，需要注意以下变化：

```javascript
var foo = true, baz = 10;

if(foo){
	let bar = 3;
	
	if(baz > bar){		// 重构时得注意bar
		console.log(baz);
	}
	
	// ...
}
```

##### const

ES6除了引入let，还引入了const，同样可以用来创建块作用域变量，但值是固定的常量。之后任何修改都会报错。

```javascript
var foo = true;

if(foo){
	var a = 2;
	const b = 3;	// 包含在if中的块作用域常量
	
	a = 3;			// 正确
	b = 4;			// 错误
}

console.log(a);		// 3
console.log(b);		// ReferenceError
```



### 提升

作用域和其中的变量声明出现的位置有某种微妙的联系，这个细节是我们需要讨论的内容

#### 先有鸡还是先有蛋

直觉上认为JavaScript的代码是从上到下一行行执行，但是并不完全正确，比如：

```javascript
a = 2;

var a;

console.log(a);		// 2
```

很多人会认为是undefined，因为他们认为var a在声明a = 2之后，变量a会被重新赋值，因此被赋予默认值undefined，但是**真正结果是2**

考虑下方：

```javascript
console.log(a);

var a = 2;
```

大多数人有两种观点：

1. 非某种自上而下的行为，这个片段和上方有一样的行为，所以输出2
2. 变量a在使用前没有声明，因此会抛出ReferenceError

**正确答案：**undefined



#### 编译器再度来袭

编译器的正确思考思路：包括变量和函数在内的所有声明都会在任何代码被执行前先被处理

var a = 2 会被编译器拆分成两个声明，一个是var a，一个是a = 2，前者是在编译阶段执行的，后者赋值声明会`被留在原地`等待执行

所以上一节第一个代码片段是：

```javascript
var a;
a = 2;
console.log(a)
```

第二个代码片段是：

```javascript
var a;
console.log(a);
a = 2;
```

这个过程好像是变量和函数声明从它们在代码中出现的位置被移动到了最上面，所以**过程叫做提升**

**先有（蛋）声明后有（鸡）赋值**

**例子：**

```javascript
foo();

function foo(){
	console.log(a);		// undefined
	var a = 2;
}
```

foo函数的声明会被提升，因此第一行调用没问题，要注意每个作用域都会进行提升，foo(..)内部对var也进行了提升，如下：

```javascript
function foo(){
	var a;
	console.log(a);		// undefined
	a = 2;
}

foo();
```

上述证明了函数声明会被提升，下面证明了函数表达式不会被提升：

```javascript
foo();		// 不是Reference，而是TypeError

var foo = function(){
	// ...
}
```

此处foo因为被提升了，所以不会报Reference，但是此时foo没有赋值也就是undefined，但是函数对foo进行了函数调用，所以抛出TypeError异常

**例子：**

```javascript
foo();		// TypeError
bar();		// ReferenceError

var foo = function bar(){
	// ...
}
```

代码片段经过提升后：

```javascript
var foo;

foo();		// TypeError
bar();		// ReferenceError

foo = function bar(){
	var bar = ...self...
	// ...
}
```



#### 函数优先

函数和变量都会被提升，但是要注意先提升函数，才提升变量

```javascript
foo();		// 1

var foo;

function foo(){
	console.log(1);
}

foo = function(){
	console.log(2);
}
```

代码片段经过提升后：

```javascript
function foo(){
	console.log(1)
}

var foo;		// 被忽略

foo();

foo = function(){
	console.log(2);
}
```

**注意：**var foo尽管出现在function foo()...之前，但是是重复声明（所以被忽略了），因为函数声明会被提升到普通变量之前

尽管重复的var声明会被忽略掉，但是出现在后面的函数声明还是可以覆盖前面的

```javascript
foo();			// 3

function foo(){
	console.log(1);
}

var foo = function(){
	console.log(2);
}

function foo(){
	console.log(3);
}
```

普通块内部的函数声明通常会被提升到所在作用域的顶层，不会像下面的代码暗示的那样可以被条件判断控制：

```javascript
foo();		// TypeError: foo is not a function

var a = true;
if(a){
	function foo(){console.log("a");}
}else{
	function foo(){console.log("b");}
}
```

尽可能避免在块内部声明函数



### 作用域闭包

