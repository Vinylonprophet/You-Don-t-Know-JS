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

神话上的概念：**闭包**

#### 启示

JavaScript中闭包无处不在，我们需要识别并拥抱它

闭包是基于词法作用域书写代码时产生的自然结果，不需要去有意识地创建闭包，我们缺少的是根据自己的意愿来识别、拥抱和影响闭包的思维环境



#### 实质问题

当函数可以记住并访问所在的词法作用域时，就产生了闭包，也就是`函数在当前词法作用域之外执行`

**例子：**

```javascript
function foo(){
	var a = 2;
	
	function bar(){
		console.log(a);		// 2
	}
	
	bar();
}

foo();
```

这段代码基于语法作用域的查找规则，函数bar()可以访问外部作用域中的变量a

是否是闭包？从技术上说是，根据前面定义而言不是，bar()对a的引用方法是词法作用域的查找规则，规则`只是`闭包的一部分，但也是`非常重要`的一部分

学术的角度而言，bar()涵盖foo()作用域的闭包（事实上，涵盖了它能访问的所用作用域，比如全局作用域），可以认为bar()封闭在了foo()的作用域中，因为bar()嵌套在foo()内部

但这种方式定义的闭包不能直接进行观察，也无法明白代码片段中闭包是如何进行工作的

**关于闭包的例子：**

```javascript
function foo(){
	var a = 2;
	
	function bar(){
		console.log(a);
	}
	
	return bar;
}

var baz = foo();

baz();		// 2 ———— 闭包的效果
```

bar()的词法作用域能够访问foo()内部作用域，我们将bar()函数本身当作一个值类型进行传递，例子中将bar所引用的函数对象本身当作返回值

foo()执行后，返回值赋值给baz并且调用bar()，实际上只是通过不同的标识符调用了内部的函数

bar()可以被正常执行，但是在自己定义的词法作用域以外的地方执行，一般来说foo()执行后整个内部作用域都会被销毁，而**闭包**的神奇就在于组织被销毁，内部作用域依然存在，因此没有被回收，bar()本身在使用这个内部作用域

bar()拥有涵盖foo()内部作用域的闭包，使该作用域一直存活，以便于bar()之后任何时间进行引用

**bar() 依然有对该作用域的引用，而这个引用就是闭包**

baz()可以访问定义时的词法作用域，这个函数在定义时的词法作用域以外的地方被调用，闭包使得函数可以继续访问定义时的词法作用域

**例子：**

```javascript
function foo(){
	var a = 2;
	
	function baz(){
		console.log(a);		// 2
	}
	
	bar(baz);
}

function bar(fn){
	fn();					// 这就是闭包
}

foo();
```

把内部函数baz传递给bar，调用这个内部函数时，它涵盖的foo()内部作用域的闭包就可以观察到了，因为他能访问a

传递函数也可以是间接的：

```javascript
var fn;

function foo(){
	var a = 2;
	
	function baz(){
		console.log(a);
	}
	
	fn = baz;			// 将baz分给全局变量
}

function bar(){
	fn();				// 这就是闭包
}

foo();

bar();					// 2
```

无论通过哪种手段将内部函数`传递`到所在的词法作用域外，都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包



#### 现在我懂了

上述`解释了`如何使用闭包而人为地在结构上进行了修饰。接下来来`搞懂`闭包地事实

```javascript
function wait(message){

	setTimeout( function timer() {
		console.log(message);
	}, 1000);

}

wait("Hello, closure!");
```

内部函数传递给setTimeout(..)，timer具有涵盖wait(..)作用域的闭包，因此还保留对变量message的引用

wait(..)执行1000ms后，内部作用域不会消失，timer依然保持有wait(..)作用域的闭包

**这就是闭包**

本质上无论何时何地，如果将函数当作第一级的值类型并且到处传，会看到闭包在这些函数中的应用。在定时器、事件监听器、Ajax请求、跨窗口通信、Web Workers或任何其他的异步（或同步）任务中，只要使用了回调函数，实际上就是在使用闭包

**例子：**

通常认为IIFE是典型的闭包例子，但是根据先前的闭包定义并不能这么讲

```javascript
var a = 2;

(function IIFE(){
	console.log(a);
})();
```

为什么不能算是闭包，因为它并没有在本身的词法作用域以外执行，它在定义时所在的作用域中执行（外部作用域，也就是全局作用域也持有a），a通过普通的词法作用域查找而非闭包被发现

IIFE`不是观察闭包的恰当例子`，但它的确创建了闭包，并且也是最长用来创建可以被封闭起来的闭包的工具



#### 循环和闭包

for循环是最常见的闭包

```javascript
for(var i=1; i<=5; i++){
	setTimeout( function timer(){
		console.log(i);
	}, i * 1000);
};
```

正常预期：分别输出1~5，每秒一次，每次一个

实际情况：每秒一次的频率输出五次6

原因如下：

1. 输出显示的i是循环结束时 i 的最终值
2. 所有回调函数在循环结束后才会被执行，所以setTimeout一开始延迟的1s是最初的1s，然后输出的是循环结束的i也就是6

循环的缺陷：循环的五个函数是各个迭代中分别定义的，但是它们都被封装在一个共享的全局作用域中，因此实际上只有一个i

**例子：**

```javascript
for(var i=0; i<=5; i++){
	(function(){
		setTimeout( function timer(){
			console.log(i);
		}, i * 1000);
	})();
}
```

这样不行，虽然IIFE在每次迭代中创建的作用域封闭起来，但是如果作用域为空，仅仅是封闭是不行的，改进如下：

```javascript
for(var i=0; i<=5; i++){
	(function(){
		var j = i
		setTimeout( function timer(){
			console.log(j);
		}, j * 1000);
	})();
}
```

再次改进：

```javascript
for(var i=0; i<=5; i++){
	(function(j){
		setTimeout( function timer(){
			console.log(j);
		}, j * 1000);
	})(i);
}
```

问题解决

##### 重返块作用域

思考上述的解决方法，IIFE每次迭代都创建了一个新的作用域，也就是每次都需要一个块作用域，本质上来说就是`将一个块转换成一个可以被关闭的作用域`，也就可以实现以下代码：

```javascript
for(var i=0; i<=5; i++){
	let j = i;				// 闭包的块作用域
    setTimeout( function timer(){
        console.log(j);
    }, j * 1000);
}
```

for循环头部的let每次迭代都会声明，每一次都会使用上一个迭代结束的值来初始化这个变量

```javascript
for(let i=0; i<=5; i++){
    setTimeout( function timer(){
        console.log(i);
    }, i * 1000);
}
```

块作用域和闭包联手天下无敌



#### 模块

其他模式利用闭包的为例，表面上和回调无关。其中最强大的一个：**模块** 。

```javascript
function foo(){
	var something = "cool";
	var another = [1, 2, 3];
	
	function doSomething(){
		console.log(something);
	}
	
	function doAnother(){
		console.log(another.join("!"));
	}
}
```

此处没有明显的闭包，只有两个私有数据变量something和another，以及doSomething()和doAnother()两个内部函数，它们的词法作用域也就是foo()的内部作用域

考虑如下代码：

```javascript
function CoolModule(){
	var something = "cool";
	var another = [1, 2, 3];
	
	function doSomething(){
		console.log(something);
	}
	
	function doAnother(){
		console.log(another.join(" ! "));
	}
	
	return {
		doSomething: doSomething,
		doAnother: doAnother
	}
}

var foo = CoolModule();

foo.doSomething();		// cool
foo.doAnother();		// 1 ! 2 ! 3
```

这个模式在JavaScript被称为模块，最常见的实现模式模块的方法通常被称为模块暴露，这里是其变体

1. CoolModule()只是一个函数，必须通过调用它来创建一个模块实例，如果不执行外部函数，内部作用域和闭包都无法被创建
2. CooModule()返回一个用对象字面量语法{ key:value, ... }来表示的对象，这个返回的对象中含有对内部函数而不是内部数据变量的引用。保持了内部数据是隐藏且私有的，可以将对象类型的返回值看作本质上是模块的公共API
3. doSomething()和doAnother()函数具有涵盖模块实力内部的作用域的闭包（通过调用CoolModule()实现），当通过返回一个含有属性引用的对象的方式来将函数传递到词法作用域外部时，我们已经创造了可以观察和实践闭包的条件

如此得出，模式模块需要具备**两个必要条件：**

1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新模块实例）
2. 封闭函数必须返回至少一个内部实例，这样内部函数才能在私有作用域中形成闭包，并且可以访问或修改私有的状态

`具有函数属性的对象并不是真正的模块，一个函数调用所返回的，只有数据属性而没有闭包函数的对象并不是真正的模块`

上述代码有一个叫CoolModule()的独立的模块创建器，可以被调用任意次，每次调用都会创建一个新的模块实例，当只需要一个实例时，可以对其进行改进来实现`单例模式`：

```javascript
var foo = (function CoolModule(){
	var something = "cool";
	var another = [1, 2, 3];
	
	function doSomething(){
		console.log(something);
	}
	
	function doAnother(){
		console.log(another.join(" ! "));
	}
	
	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething();		// cool
foo.doAnother();		// 1 ! 2 ! 3
```

将函数模块转换成了IIFE，立即调用这个函数并且返回值直接赋值给单例的模块实例标识符foo

模块也是`普通的函数`，因此也可以接收参数：

```javascript
function CoolModule(id){
	function identify(){
		console.log(id);
	}
	
	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify();		// foo 1
foo2.identify();		// foo 2
```

模块另一个简单但强大的用法是命名将要作为公共API返回的对象：

```javascript
var foo = (function CoolModule(id){
	function change(){
		// 修改公共API
		publicApi.identify = identify2;
	}
	
	function identify1(){
		console.log(id);
	}
	
	function identify2(){
		console.log(id.toUpperCase());
	}
	
	var publicAPI = function(){
		change: change,
		identify: identify1
	};
	
	return publicAPI;
})("foo Module")

foo.identify();			// foo Module
foo.change();
foo.identify();			// FOO MODULE
```

通过在模块实例的内部保留对公共API对象的内部引用，可以从内部对模块实例进行修改，包括添加或删除方法和属性，以及修改他们的值



##### 现代的模块机制

大多数模块依赖加载器/管理器本质上都是将这种模块定义封装进一个友好的API，这里简单介绍一些核心概念：

```javascript
var MyModules = (function Manager() {
	var modules = {};
	
	function define(name, deps, impl) {
		for(var i=0; i<deps.length; i++){
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}
	
	function get(name){
		return modules[name];
	}
	
	return {
		define: define,
		get: get
	};
})();
```

核心：modules[name] = impl.apply(impl, deps)

为了模块的定义引入了包装函数（可以传入任何依赖），并且将返回值，也就是模块API，储存在一个根据名字来管理的模块列表中

```javascript
MyModules.define("bar", [], function() {
	function hello(who){
		return "Let me introduce: " + who;
	}
	
	return {
		hello: hello
	};
});

MyModules.define("foo", ["bar"], function() {
	var hungry = "hippo";

	function awesome(){
		console.log( bar.hello(hungry).toUpperCase() );
	}
	
	return {
		awesome: awesome
	};
});

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
);	// Let me introduce: hippo

foo.awesome();	// LET ME INTRODUCE: HIPPO
```

foo 和 bar 都是通过一个返回公共API的函数来定义的。foo 甚至接受 bar 的实例作为依赖参数，并相应使用它

模块模式的两个特点：调用了包装了函数定义的包装函数，并且将返回值作为该模块的API

##### 未来来的模块机制

ES6为模块增加了一级语法支持，当通过模块系统进行加载时，会将文件当作独立的模块处理。每个模块都可以导入其他模块或特定的API成员，同样也可以导出自己的API成员

ES6没有 ”行内“ 格式，必须被定义在独立的文件中（一个文件一个模块），浏览器或引擎有一个默认的”模块加载器“可以在`导入模块的时候同步地加载模块文件`

**例子：**

bar.js

```javascript
function hello(who) {
	return "Let me introduce: " + who;
}

export hello
```

foo.js

```javascript
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello(hungry).toUpperCase()
	);
}

export awesome
```

baz.js

```javascript
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello("rhino")
);	// Let me introduce: rhino

foo.awesome();		// LET ME INTRODUCE: HIPPO
```

之前的代码片段会分别创建bar.js和foo.js，然后baz.js中的程序会加载或导入这两个模块并使用他们

模块文件中的内容会被当作好像包含在作用域闭包中一样来处理，就和前面介绍的函数闭包模块一样



#### 闭包小结

言简意赅就是：在词法作用域的环境下写代码，其中的函数也是值，可以随意传来传去

**当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包**

闭包可以使用多种形式来实现`模块`等模式，模块有两个特征：

1. 为创建内部作用域而调用了一个包装函数
2. 包装函数的返回值必须至少包括一个对内部函数的引用，这样就会创建涵盖整个包装函数内部作用域的闭包



### 附录A —— 动态作用域

**词法作用域（JavaScript）：**

```javascript
function foo(){
	console.log(a);		// 2
}

function bar(){
	var a = 3;
	foo();
}

var a = 2;

bar();
```

foo()中的a通过RHS引用到了全局作用域中的a，因此输出了2

动态作用域（其他很多语言）—— 不关心函数和作用域如何声明以及在何处声明，只关心从何处调用，也就是说`作用域链基于调用栈，而不是代码中的作用域嵌套`

下面是**动态作用域的例子：**

```javascript
function foo(){
	console.log(a);		// 3
}

function bar(){
	var a = 3;
	foo();
}

var a = 2;

bar();
```

如果是动态作用域下，foo()会无法找到a的变量引用，会顺着调用栈在调用foo()的地方查找a，而不是在嵌套的词法作用域链向上查找，由于foo()在bar()中调用，所以会检查bar()的作用域，并且在其中找到值为3的变量a

**总结：**JavaScript不具有动态作用域，只有词法作用域，但是this机制某种程度上来说很像动态作用域

**主要区别：**

1. 词法作用域是在写代码或定义时确定的，而动态作用域是在运行时确定的（this也是）
2. 词法作用域关注函数在何处声明，动态作用域关注函数从何调用



### 附录B —— 块作用域的替代方案

ES6引入了let，使代码有了创建完整，不受约束的块作用域的能力，其在代码风格和功能上都有很多新特性，但是在ES6之前是如何使用块作用域的呢？

**ES6：**

```javascript
{
	let a = 2;
	console.log(a);		// 2
}

console.log(a);			// ReferenceError
```

**ES6之前：**

```javascript
try{throw 2;}catch(a){
	console.log(a);		// 2
}

console.log(a);			// ReferenceError
```

用到了catch的块作用域，但是过于丑陋



#### B.1 Traceur

Google维护着名为Traceur的项目，项目正是用来将ES6转换为兼容ES6之前的环境（大部分是ES5，但不是全部）。

Traceur会将代码转换成：

```javascript
{
	try{
		throw undefined;
	} catch(a) {
		a = 2;
		console.log(a);
	}
}

console.log(a);
```

try/catch从ES3开始就存在了



#### B.2 隐式和显式作用域

考虑下面的let的使用方法，它被称作let作用域或let声明（对比之前的let定义）：

```javascript
let(a = 2) {
	console.log(a);		// 2
}

console.log(a);			// ReferenceError
```

同之前隐式地劫持一个已经存在的作用域不同，let会创建一个显式地作用域并与其进行绑定，**显示作用域**不仅更加`突出`，在代码重构时也表现得更加`健壮`。语法上，通过强制将所有变量声明提升到块的顶部来产生更简洁的代码。这样更容易判断变量是否属于某个作用域

let声明放块的顶部就会容易辨识和维护

可惜 ES6 中不包括 let 声明，官方的 Traceur 编译器也不接受这种形式的代码

**两个选择：**

做出代码规范性上的妥协：

```javascript
/*let*/ { let a = 2;
	console.log(a);
}

console.log(a);		// ReferenceError
```

使用let-er的工具解决问题，let-er是一个构建时的代码转换器，唯一的作用就是找到let声明并对其进行转换，可以去了解一下



#### B.3 性能

为什么不直接使用IIFE来创建作用域？

两个回答：

1. try/catch确实糟糕，但是自从TC39支持ES6转换器使用try/catch之后，就已经对性能进行改进了
2. IIFE和try/catch并不完全对等，代码中的任意一部分拿出来用函数包裹，会改变这段代码的意义，其中的this、return、break 和 continue 都会发生变化。并不是一个普适的解决方案



### 附录C —— this词法

ES6 添加了一个特殊的语法形式用于函数声明，叫做箭头函数，如下：

```javascript
var foo = a => {
	console.log(a);
}

foo(2);		// 2
```

这里的箭头通常被当作function关键字的简写，但是有着**更重要的作用：**

```javascript
var obj = {
	id: "awesome",
	cool: function coolFn() {
		console.log(this.id);
	}
}

var id = "not awesome";

obj.cool();		// awesome

setTimeout(obj.cool, 100);		// not awesome
```

上述代码理解很容易，setTimeout是windows在调用，所以setTimeout指向的是windows.id，也就是说cool()函数丢失了同this之间的绑定，一般最常用的就是 var self = this;

```javascript
var obj = {
	count: 0;
	cool: function coolFn(){
		var self = this;
		
		if(self.count < 1) {
			setTimeout( function timer(){
				self.count++;
				console.log(" awesome? ");
			}, 100 )；
		}
	}
}

obj.cool();		// awesome?
```

var self = this 圆满解决了理解和正确使用this绑定的问题，并且没有过于复杂化，使用的是词法作用域。self只是一个可以通过`词法作用域`和`闭包`进行引用的标识符

ES6 箭头函数引入了一个叫做this词法的行为：

```javascript
var obj = {
	count: 0,
	cool: function coolFn(){
		if(this.count < 1){
			setTimeout( () => {
            	this.count++;
            	console.log("awesome ?");
            }, 100 );
		}
	}
}

obj.cool();		// awesome?
```

箭头函数在涉及this绑定时和普通函数的行为完全不一致，他用的是当前词法作用域覆盖this本来的值，这里的箭头函数是继承了cool()函数的this绑定

另一个不够理想的原因是：箭头函数是`匿名`而非具名

使用bind()会更理想：

```javascript
var obj = {
	count: 0,
	cool: function(){
		if(this.count < 1){
			setTimeout( function(){
				this.count++;
				console.log("more awesome");
			}.bind(this), 100);
		}
	}
}

obj.cool();		// more awesome
```

无论是箭头还是还是bind，它们之间`有意为之`的不同行为需要我们理解和掌握，才能正确的使用它们



## CLOURES THIS & OBJECT PROTOTYPES

### 关于this

this被自动定义在所有函数的作用域中，this可以是代词也可以是关键字，所以最好是使用"this"代表关键字，this代表代词



#### 为什么要用this

```javascript
function identify(){
	return this.name.toUpperCase();
}

function speak(){
	var greeting = "Hello, I'm " + identify.call(this);
    console.log(greeting);
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call(me);		// KYLE
identify.call(you);		// READER

speak.call(me);			// Hello, I'm KYLE
speak.call(you);		// Hello, I'm READER
```

这段代码可以在不同的上下文对象(me和you)中重复使用identify()和speak()，不用针对每个对象编写不同版本的函数

如果不使用this，就需要给identify()和speak()显示传入一个上下文对象

```javascript
function identify(context){
	return context.name.toUpperCase();
}

function speak(context){
	var greeting = "Hello, I'm " + identify(context);
	console.log(greeting);
}

identify(you);		// READER
speak(me);			// Hello, I'm KYLE
```

this提供了一种更优雅的方式来**隐式的传递**一个对象引用，因此可以将API设计的更加**简洁且易于复用**



#### 误解

this的字面意思会产生一些误解，有两种常见的关于this的解释大多都是错误的

##### 指向自身

很容易把this理解成指向函数自身，从英语语法的角度而言是ok的

为什么需要从函数内部引用函数自身呢，常见的原因是递归或者写一个第一次被调用自身后自己解除绑定的事件处理器

新手一般会把函数看成一个对象（JavaScript中所有函数都是对象），那就可以在调用函数的存储状态（属性值），但实际上除了函数对象还有许多更适合存储状态的地方

先来思考以下代码

```javascript
function foo(num){
	console.log("foo" + num);
	
	// 记录foo被调用的次数
    this.count++;
}

foo.count = 0;

var i = 0;

for(i=0; i<10; i++){
    if(i > 5){
        foo(i);
    }
}
// 6
// 7
// 8
// 9

// foo被调用了多少次
console.log(foo.count);		// 0	why
```

显然字面意思是错误的，因为虽然foo被调用了四次，this也是指向自身，但是count仍然是0

**注意：**如果在foo中添加一个console.log(this.count)，会输出NaN

所以虽然添加了一个属性count，但是函数内部代码的this显然不指向添加的count，虽然`属性名相同`，但`根对象并不相同`

深入探索，这段代码无意间创了一个全局变量count，值为NaN，那为什么是全局的，为什么值是NaN而不是其他的更适合的值呢

**解决方法一：**

```javascript
function foo(num){
	console.log("foo: " + num);
	
	// 记录 foo 被调用的次数
	data.count++;
}

var data = {
	count: 0
};

var i;

for(i=0; i<10; i++){
	if(i>5){
		foo(i);
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// foo 被调用了多少次？
console.log(data.count++);	// 4
```

虽然解决了问题，但是忽略了真正的问题——无法理解this的含义和工作原理，而是返回了舒适区，使用了更熟悉的技术：词法作用域

如果从函数对象内部引用自身，只用this是不够的，一般而言需要通过一个指向函数对象的词法标识符（变量）来引用它

```javascript
function foo(){
	foo.count = 4;
}

setTimeout(function(){
	// 匿名函数没法调用自身
},10 )
```

第二个例子因为没有名称标识符，所以无法从函数内部引用自身

**补充：**有一种传统的但是已经被放弃和批判的是使用arguments.callee来引用当前正在运行的函数对象。是唯一可以从匿名函数对象内部引用自身的方法。然而，更好的方式是避免使用匿名函数，至少在需要自引时使用具名函数

所以，另一种解决方法是使用foo标识符来替代this来引用函数对象：

```javascript
function foo(num){
	console.log("foo: " + num);
	
	// 记录foo被调用的次数
	foo.count++;
}
foo.count=0;
var i;

for(i=0; i<10; i++){
	if(i > 5){
		foo(i);
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// foo被调用了多少次？
console.log(foo.count);		// 4
```

但是，这种方法同样回避了this，并且完全依赖于变量foo的词法作用域

另一种方法是强制this指向foo函数对象：

```javascript
function foo(num){
	console.log("foo: " + num);
	
	// 记录foo被调用的次数
	this.count++;
}

foo.count = 0;

var i;

for(i=0; i<10; i++){
	if(i > 5){
        // 使用call(...)可以确保this指向函数对象foo本身
		foo.call(foo, i);
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// foo被调用了多少次？
console.log(foo.count);		// 4
```



##### 作用域

第二种误解：this指向函数的作用域，某种情况下正确，其他情况下是错误的

**明确一点：**this在任何情况下都不指向函数的词法作用域，在JavaScript内部，内部作用域和对象类似，但是作用域“对象”无法通过代码访问，存在于JavaScript引擎内部

下列代码试图跨越边界，但是没有成功，使用this隐式引用函数的词法作用域：

```javascript
function foo(){
	var a = 2;
	this.bar();
}

function bar(){
	console.log(this.a);
}

foo();		// ReferenceError: a is not defined
```

上述代码不只是一个错误：

1. 代码通过this.bar()来引用bar()函数，这样能调用成功完全是意外，应该直接使用词法引用标识符
2. 这段代码的开发者还试图通过this联通foo和bar的词法作用域，从而让bar()可以访问foo()作用域里的变量a，这是不可能实现的



#### this到底是什么

this是在**运行时进行绑定的**，this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式

当一个函数被调用就会创建一个活动记录，这个记录会包含函数在哪里被调用（调用栈）、函数的调用方式、传入参数等信息，this就是这样一个记录属性，会在函数执行的过程中用到



### this全面解析

#### 调用位置

调用位置：函数在代码中`被调用的位置`

只有仔细分析调用位置，才能回到这个this到底引用的是什么，但是某些编程模式会隐藏真正的调用位置，最重要的就是分析调用栈，我们关心的调用位置就在当前正在执行函数的前一个调用中

下列代码来分析什么是调用栈和调用位置：

```javascript
function baz(){
	// 当前调用栈是：baz
	// 因此，当前调用位置是全局作用域
	
	Console.log("baz");
	bar();		// <-- bar的调用位置
}

function bar(){
	// 当前调用栈是baz -> bar
	// 因此当前的调用位置是baz中
	
	console.log("bar");
	foo();		// <-- foo的调用位置
}

function foo(){
	// 当前调用栈是bar -> bar -> foo
	// 因此当前的调用位置是在bar中
	
	console.log("foo");
}

baz();	// <-- baz的调用位置
```

可以把调用栈想象成一个函数调用链，可以打开浏览器进行调试，查看当前位置的函数调用列表，也就是我的调用栈



#### 绑定规则

调用位置如何决定this的绑定对象，下面列出四条规则，并解释其优先级

##### 默认规则

最常用的函数调用类型：独立函数调用

```javascript
function foo(){
	console.log(this.a);
}

var a = 2;

foo();	// 2
```

我们注意到，全局变量就是全局对象的一个同名属性，本质上就是一个东西

当我们可以看到被调用foo()时，this.a被解析成了全局变量a，是因为`函数调用时应用了this的默认绑定`，因此this指向全局变量

如果使用严格模式(strict mode)，则不能将全局对象用于默认绑定，因此this会绑到undefined

```javascript
function foo(){
	"use strict";
	
	console.log(this.a);
}

var a = 2;

foo();	// TypeError: this is undefined
```

**细节：**虽然this绑定规则完全取决于调用位置，但只有foo()运行在非'strict mode'下时，默认绑定才能绑定到全局对象；在严格模式下调用foo()则不影响默认绑定:

```javascript
function foo(){
	console.log(this.a);
}

var a = 2;

(function(){
	"use strict";
	
	foo();	// 2
})();
```

##### 隐式绑定

调用位置需要考虑是否有上下文对象

思考以下代码：

```javascript
function foo(){
	console.log(this.a);
}

var obj ={
	a: 2;
	foo: foo
}

obj.foo();		// 2
```

**注意：**foo()的声明方式，以及是如何被当作引用属性添加到obj中，但无论是在obj中定义还是先定义后添加为引用属性，这个函数严格来说都不属于obj对象

所以当函数有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象，因为调用foo()时this被绑定到obj，因此this.a和obj.a是一样的

对象属性引用链中`只有上一层`和`最后一层`在调用位置中起作用：

```javascript
function foo(){
	console.log(this.a);
}

var obj2 = {
	a: 42,
	foo: foo
}

var obj1={
	a: 2,
	obj2: obj2
}

obj1.obj2.foo();	// 42
```

###### 隐式丢失

一个最常见的问题就是`被隐式绑定的函数会丢失绑定对象`，也就是说它会应用默认绑定，从而把this绑定到全局对象或undefined上，却决于是否是`严格模式`

思考下列代码：

```javascript
function foo(){
	console.log(this.a);
}

var obj = {
	a: 2,
	foo: foo
}

var bar = obj.foo;			// 函数别名

var a = "oops, global";		// a全局属性

bar();						// oops,global
```

虽然bar是对obj.foo的一个引用，但是实际上引用的是foo函数本身，因此bar()其实是一个`不带任何修饰的函数调用`，因此应用了`默认绑定`

更微妙，更常见，更出乎意料的情况发生在传入回调函数中：

```javascript
function foo(){
	console.log(this.a);
}

function doFoo(fn){
	// fn其实引用的是foo
	
	fn();
}

var obj = {
	a: 2,
	foo: foo
}

var a = "oops, global";

doFoo(obj.foo);		// oops,global
```

参数传递就是一种隐式赋值，因此传入函数时也会被隐式赋值，所以结果一样



如果把函数传入语言内置的函数而不是传入我自己声明的函数：

```javascript
function foo(){
	console.log(this.a);
}

var obj = {
	a: 2,
	foo: foo
}

var a = "oops, global"

setTimeout(obj.foo, 100);		// oops, global
```

结果没有变化，因为只要参数传递，就是隐式赋值

**重点：**JavaScript中内置的setTimeout()函数和下列的伪代码类似：

```javascript
function setTimeout(fn, delay){
	// 等待delay毫秒
	fn();	//	<-- 调用位置！
}
```

就像我们看到的，回调函数丢失this绑定很常见，除此之外，`调用回调函数的函数可能会修改this`，一些流行的JavaScript库中事件处理器常会把`回调函数的this强制绑定到触发事件的DOM上`

无论哪种情况出现，this的改变都是意想不到的，实际上`无法控制回调函数的执行方式`，因此就没有办法控制调用位置以得到期望的绑定，之后会介绍`固定this来修复这个问题`

##### 显示绑定

隐式绑定中，我们必须在一个对象内部包含一个指向函数的属性，并且通过这个属性间接引用函数，从而把this间接（隐式）绑定到这个对象上

那么我们下面介绍不在对象内部包含函数引用，而在某个对象上`强制调用函数`

JavaScript所有的函数都有一些有用的特性，这和`prototype`有关，具体点说，可以使用函数得call(..)和apply(..)方法，严格来说，JavAScript提供得一些特殊的函数没有这两个方法，但是不常见，绝大多数函数都可以使用call(..)和apply(..)

**用法：**第一个参数是一个对象，给this准备的，接着在调用的时候将其绑定到this，可以直接`指定`this的绑定对象，我们称之为`显示绑定`

思考以下代码：

```javascript
function foo(){
	console.log(this.a);
}

var obj = {
	a: 2
}

foo.call(obj);	// 2
```

通过foo.call(..)，可以在调用foo时强制把它的this绑定到obj上

当你`传入了一个原始值（字符串类型、布尔类型或者数字类型）`来当作this的绑定对象，这个原始值会被转换成它的对象形式（也就是new String(..)、new Boolean(..)或者new Numer(..)），这通常被成为`装箱`

**注意：**从this绑定的角度而言，apply(..)和call(..)是一样的，区别体现在其他参数上

但是，`显示绑定`仍然无法解决之前提出的`绑定丢失的问题`

###### 硬绑定

显示绑定的一个变种可以解决这个问题：

```javascript
function foo(){
	console.log(this.a);
}

var obj = {
	a: 2
}

var bar = function(){
	foo.call(obj);
}

bar();		// 2
setTimeout(bar, 100);	// 2

// 硬绑定的bar不可能再修改它的this
bar.call(window);	// 2
```

我们创建了函数bar()，在它的内部手动调用了foo.call(obj)，因此强制把foo的this绑定到了obj上，但之后无论如何调用bar，他总会手动在obj上调用foo，这种绑定是一种显示的强制绑定，因此称之为`硬绑定`

**典型场景：**

1. 负责创建一个包裹函数，负责接受参数并返回值：

   ```javascript
   function foo(something){
   	console.log(this.a, something);
   	return this.a + something;
   }
   
   var obj = {
   	a: 2
   };
   
   var bar = function(){
   	return foo.apply(obj, arguments);
   };
   
   var b = bar(3);		// 2, 3
   console.log(b);		// 5
   ```

2. 创建一个可以重复使用的辅助函数：

   ```javascript
   function foo(something){
   	console.log(this.a, something);
   	return this.a + something;
   }
   
   // 简单的辅助绑定函数
   function bind(fn, obj){
   	return function(){
   		return fn.apply(obj, arguments);
   	};
   }
   
   var obj = {
       a: 2
   }
   
   var bar = bind(foo, obj);
   
   var b = bar(3);		// 2, 3
   console.log(b);		// 5
   ```

   硬绑定是一种非常常用的模式，所以ES5提供了内置的方法，Function.prototype.bind，使用方法如下：

   ```javascript
   function foo(something){
   	console.log(this.a, something);
   }
   
   var obj = {
   	a: 2
   }
   
   var bar = foo.bind(obj);
   
   var b = bar(3);		// 2, 3
   console.log(b);		// 5
   ```

   bind(..)会返回一个硬编码的新函数，会把指定的参数设置为this的上下文并调用

   ###### API调用的“上下文”

   第三方的许多函数，以及JavaScript语言和宿主环境中许多新的内置函数，都提供了一个可选的参数，通常被称为“上下文”(context)，其作用和bind(..)一样，确保了你的回调函数使用指定的this

   举例：

   ```javascript
   function foo(el){
   	console.log(el, this.id);
   }
   
   var obj = {
   	id: "awesome"
   }
   
   // 调用foo(..)时把this绑定到obj
   [1,2,3].forEach(foo, obj);
   // 1 awesome 2 awesome 3 awesome
   ```

   这些函数实际上就是通过call(..)和apply(..)实现了显示绑定

##### new绑定

首先澄清一个关于JavaScript中非常常见的对函数和对象的误解

在`传统`的面向类的语言中，“构造函数”是类的一些特殊方法，使用new`初始化类时会调用类中的构造函数`，通常是：

```java
something = new MyClass(..);
```

JavaScript中也有操作符，使用方法看起来也类似，但是JavaScript中的`机制完全不同`

在JavaScript中的构造函数并不会属于哪个类，也不会实例化一个类，甚至都不能算是一种特殊函数，只是被new操作符调用的普通函数

下列是Numer(..)作为构造函数的行为：

当Number在new表达式中被调用，它是一个构造函数：会初始化新创建的对象

所以内置对象函数在内的所有函数都可以通过new来创建，这种函数调用被称为构造函数调用。

这里存在一个重要但是非常细微的区别：**实际上并不存在所谓的“构造函数”，只有对于函数的调用**

使用new来调用函数时，或者发生构造函数调用时，会自动执行下面的操作：

1. 创建（或者说构造）一个全新对象
2. 新对象会被执行[[Prototype]]连接
3. 新对象会绑定到函数调用的this
4. 如果函数`没有返回其他对象`，那么new表达式中的函数调用会自动返回这个新对象

下列代码：

```javascript
function foo(a){
	this.a = a;
}

var bar = new foo(2);

console.log(bar.a);		// 2
```

使用new来调用foo(..)，会构造一个新对象，并把它绑定到foo(..)调用的this上。new是最后一种可以影响函数调用时this绑定行为的方法，称之为new绑定

#### 优先级

this绑定的四条规则存在优先级之分

毫无疑问，默认绑定的优先级是最低的

先区分隐式和显式绑定：

```javascript
function foo(){
	console.log(this.a);
}

var obj1 = {
    a: 2,
    foo: foo
}

var obj2 = {
    a: 3,
    foo: foo
}

obj1.foo();		// 2
obj2.foo();		// 3

obj1.foo.call(obj2);		// 3
obj2.foo.call(obj1);		// 2
```

由此可见，**显式绑定**优先级更高

new绑定和隐式绑定：

```javascript
function foo(){
	this.a = something;
}

var obj1 = {
	foo: foo
}

var obj2 = {};

obj1.foo(2);
console.log(obj1.a);	// 2

obj1.foo.call(obj2, 3);
console.log(obj2.a);	// 3

var bar = new obj1.foo(4);
console.log(obj1.a);	// 2
cosnole.log(bar.a);		// 4
```

所以**new**优先级高

目前看起来似乎是硬绑定比new绑定优先级更高，无法使用new来控制this：

```javascript
function foo(something){
    this.a = something;
}

var obj1 = {};

var bar = foo.bind(obj1);

bar(2);
console.log(obj1.a);		// 2

var baz = new bar(3);
console.log(obj1.a);		// 2
console.log(baz.a);			// 3
```

出乎意料，并没有因为硬绑定而把obj1.a的值修改为3，而是修改了硬绑定（到obj1调用的）bar(..)的this，因为使用了new绑定，得到了名为baz的新对象，并且baz.a的值为3

 之前介绍的“裸”辅助函数bind是无法修改this绑定的，但是在ES5的bind(..)更为复杂，简单而言，也就是会判断硬绑定函数是否被new调用，如果是的话就会用新创建的this来替换硬绑定的this

在new中使用硬绑定函数主要是预设一些参数，这样在使用new进行初始化时只需要传入其余的参数，bind(..)功能之一就是把除了第一个参数之外的其他参数都传给下层的函数（这种技术称为“部分应用”，是“柯里化”的一种）

```javascript
function foo(p1, p2){
	this.val = p1 + p2;
}

var bar = foo.bind(null, "p1");

var baz = new bar("p2")

baz.val;		// p1p2
```

##### 判断this

我们可以通过优先级来判断函数在某个调用位置应用的是哪条规则，按照下面的顺序来判断：

1. 函数是否在new中调用（new绑定）？如果是的话，this绑定的是新创建的对象

   var bar = new foo()

2. 函数是否通过call、apply（显式绑定）或硬绑定调用？如果是的话，this绑定的是那个指定对象

   var bar = foo.call(obj)

3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象

   var bar = obj1.foo()

4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则默认全局对象

   var bar = foo()

`不过凡事也总有例外`

#### 绑定例外

某些场景下this的绑定行为会出乎意料，有时候认为应当是其他规则时，实际上应用的应该是默认规则

##### 被忽略的this

如果把null或undefined作为this的绑定对象传入call、apply、或者bind，这些值在调用的时候会被`忽略`，实际上运用的就是默认绑定规则：

```javascript
function foo(){
	console.log(this.a);
}

var a = 2;

foo.call(null);		// 2
```

那么什么情况下会传入null呢？

一种常见的做法是使用apply(..)来“展开”一个数组，并当作参数传入一个函数，类似的，bind(..)可以对参数进行柯里化（预先设置一些参数），有时非常有用：

```javascript
function foo(a, b){
	console.log("a: " + a + ", b: " + b);
}

// 把数组"展开"成参数
foo.apply( null, [2, 3]);	// a: 2, b: 3

// 使用bind(..)进行柯里化
var bar = foo.bind( null, 2);
bar(3);		// a: 2, b: 3
```

这两种方法都需要传入一个参数当作this的绑定对象，如果函数不关心this的话，仍需要`传入一个占位符`，这时null可能是一个不错的选择

ES6中，可以使用...操作符来代替apply(..)来展开数组，foo(...[1, 2])和foo(1,2)是一样的，这样还可以避免不必要的this绑定

然而，总是使用null来忽略this绑定会产生一些副作用，如果某个函数使用了this，那默认绑定规则会把this绑定到全局对象，这会导致不可预计的后果

###### 更安全的this

一种更安全的做法是传入一个对象，把this绑定到这个对象“不会对你的程序产生任何副作用”，我们创建一个“DMZ“对象——一个空的非委托的对象

因为这个变量完全是一个空对象，可以使用Ф来表示，最简单的方法就是Object.create(null)，Object.create(null)和{}很像，但是并不会创建Object.prototype，所以比{}”更空“

```javascript
function foo(a, b){
	console.log("a: " + a + ", b: " + b);
}

// 我们的DMZ空对象
var Ф = Object.create(null);

// 把数组展开成参数
foo.apply(Ф, [2, 3]);		// a: 2, b: 3

// 使用bind(..)进行柯里化
var bar = foo.bind(Ф, 2);
bar(3);		// a: 2, b: 3
```

Ф不仅让代码更安全，而且还提高了代码的可读性

##### 间接引用

有可能会有意或无意的创建一个函数的”间接引用“，这种情况下，调用这个函数会应用”默认绑定规则“

间接引用最容易在赋值时发生：

```javascript
function foo(){
	console.log(this.a);
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo();		// 3
(p.foo = o.foo)();		// 2
```

赋值表达式 p.foo = o.foo 的返回值就是目标函数的引用（即foo），所以这里就是默认绑定

注意：决定this绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this会被绑定到undefined，否则this会被绑定到全局对象

##### 软绑定

之前我们看到过，硬绑定把this强制绑定到指定对象（除了new时），防止函数调用应用默认绑定规则，使用硬绑定之后就无法使用隐式绑定或显式绑定来修改this

如果给默认绑定指定一个全局对象和undefined以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式指定和显式绑定修改this的能力

可以通过一种被称为软绑定的方法来实现我们想要的效果：

```javascript
if(!Function.prototype.softBind){
	Function.prototype.softBind = function(obj){
		var fn = this;
		// 捕获所有curried参数
		var curried = [].slice.call( arguments, 1);
		var bound = function(){
			return fn.apply(
				(!this || this === (window || global)) ?
					obj : this,
				curried.concat.apply( curried, arguments )
			);
		};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

除了软绑定，softBind(..)的其他原理和ES5内置的bind(..)类似，首先会检查调用时的this，如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this，否则不会修改this

例子：

```javascript
function foo(){
	console.log("name: " + this.name);
}

var obj = { name: "obj" };
var obj2 = { name: "obj2" };
var obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ();		// name: obj

obj2.foo = foo.softBind( obj );
obj2.foo();		// name: obj2

fooOBJ.call(obj3);		// name: obj3

setTimeout( obj2.foo, 10);		// name: obj
```

#### this词法

ES6有一种无法使用之前四条规则的特殊函数：**箭头函数**

箭头函数不是function定义的，而是 => 定义的，箭头函数不适用四种规则，而是根据外层作用域来决定的

```javascript
function foo(){
	return (a) => {
		console.log( this.a );
	}
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call(obj2);		// 2
```

foo()函数内部创建的箭头函数会捕获调用时的foo()的this，由于foo()绑定到obj1，bar的this也会绑定到obj1，箭头函数的绑定无法修改（new也不行）

箭头函数常用于回调函数中，例如事件处理器或定时器：

```javascript
function foo(){
	setTimeout( () => {
		// 这里的this在词法上继承foo()
		console.log(this.a);
	}, 100);
}

var obj = {
	a: 2
}

foo.call(obj);	// 2
```

箭头函数可以像bind(..)一样确保函数的this被绑定到指定对象，此外，重要性还体现在它用更常见的词法作用域取代了传统的this机制，ES6之前就已经在使用一种和箭头函数完全一样的模式

```javascript
function foo(){
	var self = this;
	setTimeout( function(){
		console.log( self.a );
	}, 100)
}

var obj = {
	a: 2
}

foo.call( obj );
```

虽然self和this看起来都可以取代bind(..)，但是从本质上来说，它们更想替代的是this机制

如果经常编写this风格的代码，但是绝大部分时候都会想使用 self = this 或者箭头函数来否定 this 机制，或许应当：

1. 只使用词法作用域并完全抛弃错误this风格的代码
2. 完全采用this风格，必要时使用bind(..)，尽量避免使用 self = this 和箭头函数



### 对象

函数调用位置的不同会造成this绑定对象的不同，但是对象到底是什么呢？

#### 语法

对象的两种定义形式：**文字形式** 和 **构造形式**

文字形式：

```javascript
var myObj = {
	key: value
	// ..
}
```

构造形式：

```javascript
var myObj = new Object();
myObj.key = value;
```

`唯一区别：` 文字形式可以添加多个键 / 值，构造形式必须逐个添加属性

#### 类型

六种主要类型：

- string
- number
- boolean
- null
- undefined
- object

简单基本类型（上述除了object之外都是）本身不是对象，null有时会被当作一种对象类型，但是这只是语言的bug，（typeof null会返回“object”），但是null还是基本类型

为什么 typeof null 会返回object。因为JavaScript中二进制前三位都为0会被判为object，null的二进制全为0，所以会返回object

实际上，JavaScript中有许多特殊的对象子类型，可以称之为复杂基本类型

`函数`就是对象的一个子类型（技术的角度而言就是“可调用对象”）

`数组`也是对象的一种类型，具备额外的行为

##### 内置对象

JavaScript还有一些对象子类型，通常被称为内置对象：

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp（正则表达式）
- Error

JavaScript中实际上是一些内置函数，内置函数可以当作构造函数来使用，从而可以构造一个对应子类型的新对象，比如:

```javascript
var strPimitive = "I'm a string";
typeof strPimitive;					// "string"
strPimitive instance of String;		// false

var strObject = new String("I'm a string");
typeof strObject;					// "object"
strObject instance of String;		// true

// 检查 sub-type 对象
Object.prototype.toString.call(strObject);		// [object String]
```

可以认为子类型在内部借用了Object中的toString(..)方法，从代码中可以看出strObject是String构造函创建的一个对象

原始值 "I'm a string" 不是一个字面量而是一个对象，是一个不可变的值，如果需要在字面量上进行一系列操作，需要将其转换为 String 对象

有时候语言会自动把字面量转化为一个String对象，不需要显式创建一个对象，大多数时候能使用文字形式就不要使用构造形式

思考：

```javascript
var strPimitive = "I am a string";

console.log(strPimitive.length);		// 13

console.log(strPimitive.charAt(3));		// m
```

之所以可以在字面量上访问属性和方法，是因为**引擎自动把字面量转换成String对象，所以可以访问属性和方法**

数字字面量，使用类似42.359.toFixed(2)方法，会把42转换成 new Number(42)

布尔字面量同样如此

null 和 undefined 没有对应构造形式，只有文字形式，相反，Date只有构造，没有文字形式

对于 Object、Array、Function、RegExp 来说，无论文字形式还是构造形式，都是对象，不是字面量

Error对象很少在代码中显示创建，一般在抛出异常时被自动创建

#### 内容

对象的内容是由一些存储在特定命名位置的值组成的，称之为属性

存储在对象容器内部是这些属性的名称，就像是指针(从技术的角度而言就像是引用)一样，指向这些值真正的存储位置

思考：

```javascript
var myObject = {
	a: 2
};

myObject.a;			// 2
myObject["a"];		// 2
```

访问妈呀Object中a位置上的值，需要使用.操作符或[]操作符。

.a通常被称为“属性访问”；["a"]通常被称为“键访问”，两种语法的主要区别在于：.操作符要求属性名满足标识符的命名规范，[".."]语法可以接受任意UTF-8/Unicode字符串作为属性名，比如可以["Super-fun!"]语法访问，但.操作符就不行

此外，可以在程序中构造字符串：

```javascript
var myObject = {
	a: 2
};

var idx;

if(wantA) {
	idx = "a";
}

console.log(myObject[idx]);		// 2
```

对象中，属性名永远都是字符串，虽然在`数组下标中使用的是数字，但是在对象属性中数字会被转换成字符串：`

```javascript
var myObject = { };

myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"] = "foo";		// "foo"
myObject["3"] = "bar";			// "bar"
myObject["[Object Object]"];	// "baz"
```

##### 可计算属性名

键访问可以使用myObject[prefix + name]，但使用文字形式来声明对象时这样做不行

ES6增加了`可计算属性名`，可以在文字形式使用[]包裹一个表达式来当作属性名：

```javascript
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"];		// hello
myObject["foobaz"];		// world
```

可以算属性名最常用的场景是ES6的符号”Symbol“，不过简单来说，它们是一种新的基础数据类型，包含一个透明且无法预测的值（技术的角度而言可以说是一个字符串），一般不会用到符号的`实际值`，所以通常接触到的是符号的名称，比如Symbol.Something:

```javascript
var myObject = {
	[Symbol.something]: "hello world"
}
```

##### 属性和方法

在JavaScript中，函数永远不会属于一个对象，所以把对象内部引用的函数成为方法有点不妥

有些函数有this引用，确实也会只想调用位置的对象引用，但是本质上来说也没有把一个函数变成一个方法，this是根据位置动态绑定的，所以函数和对象的关系也是间接关系

属性访问的函数和其他函数没有任何区别：

```javascript
function foo(){
    console.log("foo");
}

var someFoo = foo;		// 对foo变量的引用

var myObject = {
    someFoo: foo
};

foo;					// function foo(){..}

someFoo;				// function foo(){..}

myObject.soemFoo;		// function foo(){..}
```

someFoo和myObject.someFoo只是对于同一个函数的不同引用，并不能说明这个函数是特别的或者”属于“某个对象。如果foo()定义时在内部有一个this引用，那么这两个函数引用的唯一区别就是myObject.someFoo中的this会被绑定到一个对象，但是上述无论哪种都不能被称为”方法“

`注意：`ES6新增了super引用，一般来说会被用在class中，可以参考附录A，super的行为可更有理由把super绑定的函数称为”方法“，

即使在对象文字中声明一个函数表达式，也不会”属于“这个对象——只是对于相同函数对象的多个引用

注意：

- super()作为函数代表父类的构造函数
- super()作为对象，指向父类的原型对象

```javascript
var myObject = {
	foo: function(){
		console.log("foo");
	}
}

var someFoo = myObject.foo;

someFoo;			// function foo(){..}

myObject.foo;		// function foo(){..}
```

##### 数组

数组有一套更加结构化的值存储机制（不过依然不限制值的类型），数组期望的是数值下标，也就是值存储位置

```javascript
var myArray = [ "foo", 42, "bar" ];

myArray.length;		// 3
myArray[0];			// "foo"
myArray[2];			// "bar"
```

`数组也是对象`，所以虽然下标都是整数，但是仍然可以给数组添加属性：

```javascript
var myArray = [ "foo", 42, "bar" ];

myArray.baz = "baz";

myArray.length;			// 3

myArray.baz;			// "bar"
```

可以看到虽然添加了命名属性，但length值并未发生变化

注意：如果试图向数组添加一个属性，但属性名看起来像数字，那么会变成一个数值下标：

```javascript
var myArray = [ "foo", 42, "bar" ];

myArray.["3"]= "baz";

myArray.length;			// 4

myArray[3];				// "baz"
```

##### 复制对象（涉及浅拷贝和深拷贝）

JavaScript最常见的问题就是复制一个对象

思考以下代码：

```javascript
function anotherFunction() { /*..*/ }

var anotherObject = {
    c: true
};

var anotherArray = [];

var myObject = {
    a: 2,
    b: anotherObject,		// 引用，不是复制！
    c: anotherArray,		// 另一个引用
    d: anotherFunction
};

anotherArray.push( anotherObject, myObject );
```

如何准确的表示Object的复制呢？

应该判断`浅拷贝`还是`深拷贝`：

- 浅拷贝：a的值会复制旧对象的a值，b，c，d三个属性都是引用
- 深拷贝：除了复制myObject，还会复制anotherObject和anotherArray，所以anotherArray引用了anotherObject和myObject，又需要复制myObject，会由于循环引用导致死循环

有些人也会通过"toString()"来序列化一个函数的源代码

对于JSON安全的对象而言，有一种复制方法：

```javascript
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

这种方法需要保证对象是JSON安全的，所以只用于**部分情况**

相比深拷贝，浅拷贝容易多，ES6提供了 **Object.assign(..)** 方法来实现浅拷贝，第一个参数是目标对象，第二个以及之后还可以跟一个或多个源对象，它会遍历一个或多个源对象的所有可枚举(enumberable)的自有键(owned key)并把他们`复制（使用 = 操作符赋值）到目标对象`，最后返回目标对象，像这样：

```javascript
var newObj = Object.assign( {}, myObject );

newObj.a;										// 2
newObj.b === anotherObject;						// true
newObj.c === anotherArray;						// true
newObj.d === anotherObanotherFunctionject;		// true
```

**补充：**下面介绍”属性描述符“以及Object.defineProperty(..)的用法，但是注意Object.assign(..)使用**=**操作符来赋值的，所以源对象的一些特性（比如writable）不会被复制到目标对象

##### 属性操作符

ES5之前，JavaScript没有提供可以直接检测属性特性的方法，ES5开始，所有属性都具备了属性描述符

思考以下代码:

```javascript
var myObject = {
	a: 2
};

Object.getOwnPropertyDescriptor( myObject, "a" );
// {
// 		value: 2,
// 		writable: true,
// 		enumerable: true,
// 		configurable: true
// }
```

这个普通的对象属性对应的属性描述符（也可以称为“数据描述符”），除了value，还有writable，enumerable，configurable

也可以使用**defineProperty(..)**添加一个新属性或修改一个已有属性并对特性进行配置

例子：

```javascript
var myObject = { };

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: true,
	enumberable: true
} );

myObject.a;		// 2
```

**除非修改属性描述符，不然一般不用defineProperty(..)**

###### 1. Writable

writable决定了是否可以修改属性的值

代码：

```javascript
var myObject = { };

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false,
	configurable: true,
	enumberable: true
})

myObject.a = 3;

myObject.a;		// 2
```

严格模式下：

```javascript
"use strict";

var myObject = { };

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: false,
	configurable: true,
	enumberable: true
})

myObject.a = 3;		// TypeError
```

TypeError表示无法修改一个不可写的属性

**注意：**之后会介绍getter和setter，如果wriable: false，相当于定义了一个空操作setter。严格来说，如果要和writable: false一致，那么setter被调用时应该抛出一个TyperError错误

###### 2. Configurable

defineProperty(..)方法来修改属性描述符:

```javascript
var myObject = {
	a: 2
};

myObject.a = 3;
myObject.a;		// 3

Object.defineProperty( myObject, "a", {
	value: 4,
	writable: true,
	configurable: false,
	enumberable: true
})

myObject.a;			// 4
myObject.a = 5;
myObject.a;			// 5

Object.defineProperty( myObject, "a", {
	value: 6,
	writable: true,
	configurable: true,
	enumberable: true
});					// TypeError
```

configurable是**单向操作**，无法撤销！但还是可以把writable状态从true改为false，但是无法从false改成true

除了无法修改，还会禁止`删除` **(delete)** 这个属性：

```javascript
var myObject = {
	a: 2
};

myObject.a;		// 2

delete myObject.a;
myObject.a;		// undefined

Object.defineProperty( myObject, "a", {
	value: 2,
	writable: true,
	configurable: false,
	enumberable: true
});

myObject.a;		// 2
delete myObject.a;
myObject.a;		// 2
```

如你所见delete失败了，因为属性是不可配置的，本例中delete只是用来直接删除对象的属性，如果对象的某个属性是某个对象/函数的最后一个引用者，那么delete之后，这个未引用的对象/函数就可以被垃圾回收。但是不能把delete看成一个释放内存的工具，仅仅是删除对象属性的操作

###### 3. Enumberable

如果把对象的enumberable设置成false，这个属性就不会出现在枚举中

##### 不变性

有时候希望属性和对象是不可改变的，在ES5中可以通过很多方法来实现

所有的方法创建都是浅不变性，只会影响到目标对象和它的直接属性，如果目标对象引用了其他对象（数组、对象、函数，等），其他对象的内容不受影响，仍然可变：

```javascript
myImmutableObject.foo;			// [1, 2, 3]
myImmutableObject.foo.push(4);
myImmutableObject.foo;          // [1, 2, 3, 4]
```

如果myImmutableObject已经被创建而且是不变的，但是为了保护myImmutableObject.foo，还需要使用下面的方法让foo也不可变

**补充：**JavaScript中很少需要深不变性，如果需要这样做的时候，可以先思考一下程序的设计是否合理，让它更好应对对象值的改变

###### 1. 对象常量

结合 writable: false 和 configurable: false 就可以创建一个真正的常量属性（不可修改、定义或者删除）：

```javascript
var myObject = { };

Object.definePropery( myObject, "FAVORITE_NUMBER", {
	value: 4,
	writable: false,
	configurable: false
} );
```

###### 2. 禁止扩展

如果想禁止一个对象添加新属性并且保留已有属性，可以使用Object.preventExtensions(..)：

```javascript
var myObject = { };

Object.preventExtensions( myObject );

myObject.b = 3;
myObject.b;		// undefined
```

非严格模式下，创建b会静默失败；严格模式下，抛出TypeError错误

###### 3. 密封

Object.seal(..)会创建一个“密封”对象，实际上会在现有对象调用Object.preventExtensions(..)并且把`所有`现有属性标记为configurable: false

密封之后不仅不能添加新属性，也不能重新配置或删除任何现有属性（虽然可以修改属性的值）

###### 4. 冻结

Object.freeze(..)会创建一个冻结对象，这个方法实际上会在一个现有对象上调用Object.seal(..)并且把所有属性标记为writable: false，这样就无法修改它们值

这个方法是对象上的级别最高的不可变性，会禁止对于对象本身及其任意直接属性的修改

可以“深度冻结”一个对象，首先在这个对象上使用Object.freeze(..)，然后遍历它引用的所有对象并在这些对象上调用Object.freeze(..)，但是一定要小心，这样可能会冻结其他（共享）独享【此处我理解为引用一些共有的函数】

##### [[Get]]

思考下列代码：

```javascript
var myObject = {
	a: 2
}

myObject.a;		// 2
```

语言规范中，实际上是myObject.a在myObject实现了[[Get]]操作（有点像函数调用：[[ Get ]] () )，对象默认的内置[[Get]]操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值

如果没找到名称相同的属性，按照定义会执行另一种行为，也就是遍历原型链

如果无论如何都没找到名称相同的属性，那么[[Get]]会返回值undefined:

```javascript
var myObject = {
	a: 2
}

myObject.b;		// undefined
```

**注意：**这种方法和访问变量不一样，如果引用了一个当前词法作用域中不存在的变量，并不会像对象属性一样返回undefined，而是会抛出一个ReferenceError异常：

```javascript
var myObject = {
	a: undefined
}

myObject.a;		// undefined
myObject.b;		// undefined
```

从返回值的角度：这两个引用没有区别——都返回了undefined

从底层的角度：底层的[[Get]]操作对Object.b进行了更复杂的处理

稍后会介绍如何区分

##### [[Put]]

有获取属性的[[Get]]，就一定有对应的[[Put]]，[[Put]]实际的行为取决于很多因素，包括对象中是否存在这个属性

如果存在这个属性，[[Put]]大致会检查下面内容：

1. 属性是否是访问描述符（参考下面的Getter和Setter）？如果是并且存在Setter就调用Setter
2. 属性的writable是否是false？是的话，非严格模式下失败，严格模式下抛出TypeError
3. 如果都不是，将值设为属性值

##### Getter和Setter

[[Put]]和[[Get]]分别可以控制属性的设置和获取

在ES5中可以使用getter和setter部分改写默认操作，但只能在单个属性而不是整个对象上。getter是一个隐藏函数，会在获取属性值时调用，setter也是隐藏函数，会在设置属性值时调用

当给一个属性定义getter或setter或两者，这个叫做“访问描述符”，访问描述符会忽略value和writabe特性，取而代之关心set和get（还有configurable和enumberable）特性

思考下列代码：

```javascript
var myObject = {
	get a() {
		return 2;
	}
};

Object.defineProperty(
	myObject,
	"b",
	{
		get: function(){
			return this.a*2;
			
			enumberable: true
		}
	}
);

myObject.a;		// 2
myObject.b;		// 4
```

不管是对象文字语法的get a() { .. }，还是defineProperty(..)的显式定义，二者都会在对象中创建一个`不包含值的属性`，对于这个属性的访问会自动`调用一个隐藏函数`，它的返回值会被当作前属性访问的返回值：

```javascript
var myObject = {
    // 给 a 定义一个getter
    get a() {
        return 2;
    }
}

myObject.a = 3;

myObject.a;		// 2
```

只定义了a的getter，所以对a的值进行设置时set操作会忽略赋值操作，不会抛出错误，即使有合法的setter，由于我们自定义的getter只会返回2，所以set操作也没有意义

为了让属性更加合理，还应该定义setter，setter会覆盖单个属性默认的[[put]]，通常来说getter和setter是成对出现的：

```javascript
var myObject = {
	// 给 a 定义一个getter
	get a() {
		return this._a_;
	},
	
	// 给 a 定义一个setter
	set a(val){
		this._a_ = val * 2;
	}
};

myObject.a = 2;

myObject.a;		// 4
```

本例中，把赋值操作中的值2存储到另一个变量 _ a _ ，名称 _ a _ 只是惯例，没有任何特殊的行为——和其他普通属性一样

##### 存在性

myObject.a的返回值可能是undefined，这个值可能是属性中的undefined，也可能因为不存在而返回undefined，那么如何区分呢？

可以在不访问属性值的情况下判断对象中是否存在这个属性：

```javascript
var myObject = {
	a: 2
};

("a" in myObject);		// true
("b" in myObject);		// false

myObject.hasOwnProperty("a");		// true
myObject.hasOwnProperty("b");		// false
```

in会检查属性是否在对象以及其[[prototype]]原型链，相比之下，hasOwnProperty(..)只会检查属性是否在myObject对象中，不会检查原型链

所有普通对象都会通过对于Object.prototype来访问，但是有的对象会没有（比如：Object.create(null)），这种情况下，hasOwnProperty(..)就会失败

这时，有一种更加强硬的方法解决：Object.prototype.hasOwnProperty(myObject, "a")

注意：in操作符可以检查容器内是否有某个值，但实际上检查的是某个属性名是否存在，在数组中这个区别很重要

```javascript
4 in [2, 4, 6];		// false
```

属性中有4，但属性名没有，属性名是0，1，2

###### 枚举

介绍一下可枚举性：

```javascript
var myObject = { };

Object.defineProperty(
	myObject,
    "a",
    // 让 a 像普通属性一样可以枚举
    {
        enumerable: true,
        value: 2
    }
);

Object.defineProperty(
	myObject,
    "b",
    // 让 b 不可以被枚举
    {
        enumerable: false,
        value: 3
    }
);

myObject.b;		// 3
("b" in myObject);		// true
myObject.hasOwnProperty("b");		// true

// ... ...

for(var k in myObject) {
    console.log(k, myObject[k]);
}

// "a" 2
```

for...in循环中不会出现myObject.b

注意：最好只在对象上应用for...in循环，数组就普通的for循环遍历

另一种方式区分属性是否可以枚举：

```javascript
var myObject = { };

Object.definePropert(
	myObject,
	"a",
	{
		enumberable: true,
		value: 2
	}
);

Object.definePropert(
	myObject,
	"b",
	{
		enumberable: false,
		value: 3
	}
);

myObject.propertyIsEnumberable("a");		// true
myObject.propertyIsEnumberable("b");		// false

myObject.keys(myObject);					// ["a"]
myObject.getOwenPropertyNames(myObject);	// ["a", "b"]
```

- propertyIsEnumberable：会检查对象是否`直接存在于对象中`（而不是原型链上）且满足enumberable: true
- Object.keys(..)：会返回一个数组，包含所有可枚举属性
- Object.getOwenPropertyNames(..)：会返回一个数组，包含所有属性，无论是否可以枚举

目前没有内置的方法可以获取in操作符使用的属性列表（对象本身的属性以及[[Property]]链中的所有属性），不过可以遍历某个对象的整条[[Property]]链并保存每一层中使用Object.keys(..)得到的属性列表——只包含可枚举属性

#### 遍历

for...in用来遍历对象的可枚举属性列表[包括原型链]，但如何遍历属性的值呢？

对于数值索引的数组来说，标准的for循环：

```javascript
var myArray = [1, 2, 3];

for(var i = 0; i < myArray.length; i++){
	console.log(myArray[i]);
}
// 1 2 3
```

这不是在遍历值，而是下标

ES5新增了一些数组的辅助迭代器，包括forEach(..)、every(..)和some(..)，每个辅助器都可以接受一个回调函数然后应用到数组的每个元素上，唯一的不同是回调函数返回值的处理方式不同：

- forEach(..)遍历数组中所有值并忽略回调函数的返回值
- every(..)一直运行到回调函数返回false
- some(..)一直运行到回调函数返回true

every(..)和some(..)类似于for循环中的break，提前终止遍历

那么如果 **直接遍历值而不是下标** 呢？

**使用ES6中的用来遍历数组的 `for...of`循环语法**

```javascript
var myArray = [ 1, 2, 3];

for(var v of myArray){
	console.log(v);
}

// 1
// 2
// 3
```

for...of循环首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的next()方法来遍历所有返回值

数组有内置@@iterator，因此 for...of 可以直接应用在数组上，内置的@@iterator来手动遍历数组，工作原理：

```javascript
var myArray = [ 1, 2, 3 ];
var it = myArray[Symbol.iterator]();

it.next();		// { value: 1, done: false }
it.next();		// { value: 2, done: false }
it.next();		// { value: 3, done: false }
it.next();		// { value: undefined, done: true }
```

**注意：**这里使用ES6中符号的Symbol.iterator来获取对象的@@iterator内部属性，引用iterator的特殊属性时要使用符号名，而不是符号包含的值，这里是`返回迭代器对象的函数`

value是当前的值，done是布尔值（表示是否还有可以遍历的值）

数组和普通的对象的区别：普通的对象没有内置的@@iterator，所以无法自动完成for...of

当然也可以给任何想遍历的对象定义@@iterator:

```javascript
var myObject = {
	a: 2,
	b: 3
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumberable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys(o);
		return {
			next: function(){
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// 手动便利 myObject
var it = myObject[Symbol.iterator]();
it.next();	// { value: 2, done: false }
it.next();	// { value: 3, done: false }
it.next();	// { value: undefined, done: false }

// for...of遍历object
for(var v of myObject){
	console.log(v);
}
// 2
// 3
```

也可以定义一个”无限“迭代器：

```javascript
var randoms = {
	[Symbol.iterator]: function() {
		return {
			next: function() {
				return {
					value: Math.random()
				};
			}
		}
	}
}

var randoms_pool = [];
for(var n of randoms){
	randoms_pool.push(n);
	
	// 防止无限循环被挂起
	if(randoms_pool.length === 100)
	break;
}
```



### 混合对象 “ 类 ”

先介绍面向类的设计模式：

- 实例化
- 继承
- （相对）多态

以上概念无法运用到JavaScript的对象机制，由一些解决方法（比如**混入、mixin**）

#### 类理论

面向对象强调的是数据和操作数据的行为，本质上互相关联，因此好的设计就是把数据以及它相关行为打包（也可以说是封装），有时就会被成为数据结构

一个单词或者短语会被称为字符串，字符就是数据，所以应用在数据上的行为都被设计成String类方法，所有字符串都是string类的一个实例

类的一个概念是多态，通常指父类的通用行为可以被子类用更特殊的行为重写

实例是原本抽象的数据结构通过初始化代码在从内存里变成实实在在的数据

类理论建议子类和父类使用相同的方法名，从而让子类重写父类，在JavaScript中这么做会降低代码的可读性和健壮性

##### 类设计模式

讨论的最多的是面向对象设计模式，比如迭代器模式、观察者模式、工厂模式、单例模式等等，从这个角度而言，我们是在（低级）面向对象类的基础上实现所有（高级）设计模式

##### JavaScript中的类

JavaScript只有一些近似类的语法元素（new和instanceof），ES6中新增了一些元素，比如class关键字

JavaScript中实际上`没有`类

JavaScript有近似类但是完全不一样，类是一种可选的模式

#### 类的机制

“标准库”会提供stack类，是一种“栈”数据结构（支持压入、弹出等等）。stack类内部会隐藏一些变量来存储数据，同时会提供一些公有的可访问行为（“方法”），可以和隐藏数据进行交互（比如添加、删除数据）

stack必须先实例化才能操作

##### 建造

类和实例概念源于房屋建造

- 工人 -> 蓝图 -> 房屋 （建造）
- 代码 -> 类 -> 实例 （实例化）

类和实例对象之间的关系看作直接关系而不是间接关系，类通过复制操作被实例化为对象形式

##### 构造函数

类实例是一个特殊的类方法构造的，这个方法名和类名相同，被称为构造函数

```javascript
class GoolGuy {
	specialTrick = nothing
	
	GoolGuy( trick ){
		specialTrick = trick
	}
	
	showOff(){
		output("Here's my trick: ", specialTrick)
	}
}
```

可以调用类构造函数来生成一个GoolGuy实例：

```javascript
Joe = new GoolGuy("jumping rope")

Joe.showOff()		// Here's my trick: jumping rope
```

注意，GoolGuy有一个CoolGuy()构造函数，执行new GoolGuy()时实际上调用的就是它，构造函数会返回一个对象，之后可以在这个对象上调用showOff()，来输出指定CoolGuy的特长

构造函数大多数都需要用new来调用，这样引擎才知道你想要构造一个新的类实例

#### 类的继承

子类相对于父类而言是一个独立且完全不同的类，子类会包含父类行为的原始副本，但也可以重写所有继承的行为甚至定义新行为

父类和子类不是实例，我们需要根据他们去进行实例化

例子：

```javascript
class Vehicle {
	engines = 1
	
	ignition(){
		output("Turning on my engine.")
	}
	
	drive(){
		igintion();
		output("Steering and moving forward!")
	}
}

class Car inherits Vehicle {
	wheels = 4
	
	drive() {
		inherited: drive()
		output("Rolling on all", wheels, "wheels!")
	}
}

class SpeedBoat inherits Vehicle {
	engines = 2
	
	igition() {
		output("Turning on my ", engines, " engines.")
	}
	
	pilot() {
		inherited: drive()
		output("Speeding through the water with ease!")
	}
}
```

**注意：**为了方便理解并缩短代码，我们省略了这些类的构造函数

##### 多态

Car重写了继承自父类的drive()的方法，之后Car调用了inherited:drive()方法，说明Car可以引用继承来自原始dirve()方法，快艇同样如此

这个技术被成为`多态`或`虚拟多态`，在本例中更恰当的是`相对多态`

**相对多态：**任何方法都可以引用继承层次中高的方法，之所以说“相对”是因为我们并不会定义想要访问的绝对继承层次（或者说类），而是使用`相对引用“查找上一层”`

许多语言中可以使用super来替代本例中的inherited，它的含义是“超类”，表示当前类的父类/祖先类

多态的另一个方面，在继承链不同层次中`一个方法可以被多次定义`，当调用方法时会自动选择适合的定义

上述例子就有这样的情景：drive()被定义在Vehicle和Car中，ignition()被定义在Vehicle和SpeedBoat中

在ignition()中可以看到，pilot()通过相对多态引用了Vehicle中的drive()，但是那个drive()通过名字引用了ignition()方法

引擎会使用SpeedBoat的ignition()，如果直接实例化Vehicle类然后调用它的drive()，那引擎用的就是Vehicle中的ignition()

换而言之，ignition()方法定义的多态性却决于在哪个类的实例引用

子类也可以相对引用它继承的父类，这种相对引用被称为super

`子类对继承的一个方法重写，不会影响父类中的方法`

`多态并不表示子类和父类有关连，类的继承其实就是复制`

##### 多重继承

JavaScript不提供多重继承

#### 混入

在继承和实例化，JavaScript对象不会自动执行复制，简单来说，JavaScript中只有对象，`并不存在可以被实例化的类`，一个对象并不会被复制到其他对象，它们会被关联起来

JavaScript模拟类的复制行为，这个叫做混入：**显式** 和 **隐式**

##### 显示混入

我们需要手动实现复制功能，在许多库和框架中被称为extend(..)，为了方便理解，我们称之为mixin(..)

```javascript
function mixin( sourceObj, targetObj ) {
    for( var key in sourceObj) {
        if(!(key in targetObj)){
            targetObj[key] = sourceObj[key];
        }
    }
    
    return targetObj;
}

var Vehicle = {
    engines: 1,
    
    ignition: function() {
        console.log("Turning on my engine.");
    },
    
    drive: function() {
        this.ignition();
        console.log("Streering and moving forward!");
    }
};

var Car = mixin( Vehicle, {
    wheels: 4,
    
    drive: function() {
        Vehicle.drive.call(this);
        console.log(
        	"Rolling on all" + this.wheels + " wheels!"
        );
    }
});
```

注意：这里处理的是对象而不是类

Car中的属性ignition只是从Vehicle中复制过来对ignition()的函数的引用

###### 再说多态

**显式多态：**Vehicle.drive.call(this)

**相对多态：**inherited: drive()

ES6之前没有相对多态的机制，所以由于Car和Vehicle中都有drive()，为了指明调用对象，必须使用绝对引用，通过名称显式指定Vehicle对象并调用

###### 混合复制

```javascript
function mixin( sourceObj, targetObj ) {
    for( var key in sourceObj) {
        if(!(key in targetObj)){
            targetObj[key] = sourceObj[key];
        }
    }
    
    return targetObj;
}
```

之前我们是在目标对象初始化之后（ **体现在target初始化自带新内容** ）才进行复制的，因此一定要小心不要覆盖目标对象的原有属性

如果先复制，再重写（新内容），这样效率会低：

```javascript
function mixin( sourceObj, targetObj) {
    for( key in sourceObj) {
        targetObj[key] = sourceObj[key];
    }
    
    return targetObj;
}

var Vehicle = {
    engines: 1,
    
    ignition: function() {
        console.log("Turning on my engine.");
    },
    
    drive: function() {
        this.ignition();
        console.log("Streering and moving forward!");
    }
};

var Car = minxin(Vehicle, {});

mixin({
    wheels: 4,
    
    drive: function() {
        Vehicle.drive.call(this);
        console.log(
        	"Rolling on all" + this.wheels + " wheels!"
        );
    }
}, Vehicle)
```

这个复制并不能完全模拟面向类语言中的复制，因为引用的是同一个函数

**JavaScript**复制共享函数的引用，如果修改了共享函数的对象，比如：ignition()，那么Vehicle和Car都会受到影响

如果在使用混入时感觉越来越困难，那么就应该停止使用它

###### 寄生继承

显示混入另一种变体叫做寄生继承，既是显式又是隐式：

```javascript
function Vehicle(){
	this.engines = 1
}

Vehicle.prototype.ignition = function(){
	console.log("Turning on my engine.");
};

Vehicle.prototype.drive = function(){
    this.ignition();
	console.log("Steering and moving forward!");
};

// "寄生类" Car
function Car() {
	var car = new Vehicle();
	
	car.wheels = 4;
	
	// 保存Vehicle::drive()的特殊引用
	var vehDrive = car.drive;
	
	// 重写Vehicle::drive()
	car.drive = function() {
		vehDrive.call(this);
		console.log("Rolling on all " + this.wheels + " wheels!")
	}
	
	return car;
}

var myCar = new Car();

myCar.drive();
```

先复制一份Vehicle父类的定义，然后混入子类的定义（如果需要的话保留到父类的特殊引用），然后用这个符合对象构建实例

注意：调用new Car()会创建一个新对象绑定到Car的this上，所以最初创建的car会被丢弃，因此可以不使用new关键字调用Car()，这样得到的结果一样，但可以避免创建并丢弃多余对象

##### 隐式混入

隐式混入和之前的显式伪多态很想，也具备同样的问题（复制的是函数引用）

```javascript
var Something = {
	cool: function(){
		this.greeting = "Hello World";
		this.count = this.count ? this.count + 1 : 1;
	}
};

Something.cool();
Something.greeting;		// "Hello World"
Something.count;		// 1

var Another = {
	cool: function(){
		Something.cool.call(this);
	}
};

Another.cool();
Another.greeting;		// "Hello World"
Another.count;			// 1 (count不是共享状态)
```

这里用的是this的隐式绑定，Something.cool.call(this)这行代码使函数在Another的上下文都调用了Something.cool()，但仍无法避免相对引用



### 原型

之前多次提到[[prototyoe]]链，下面开始介绍

#### [[Prototype]]

JavaScript中的对象有一个特殊的[[prototype]]内置属性，也就是对于`其他对象的引用`，几乎所有的对象在创建[[prototype]]属性都会被赋予一个非空值

注意：[[prototype]]链接可以为空，虽然很少见

思考下面代码：

```javascript
var myObject = {
	a: 2
};

myObject.a;		// 2
```

当我们试图引用对象的属性会触发[[Get]]操作，对于默认的[[Get]]操作来说，第一步检查对象本身是否有这个属性，有的话就使用它，如果a不在myObject中，就会使用`[[prototype]]链`

```javascript
var anotherObject = {
	a: 2
};

// 创建一个关联到anotherObject的对象
var myObject = Object.create(anotherObject);

myObject.a;		// 2
```

如果anotherObject也找不到a并且[[prototype]]链不为空的话，就会继续找下去，这个过程会持续找到匹配的属性名和完整的[[prototype]]链，如果后者还找不到就返回undefined

使用for...in遍历对象的原理和查找[[prototype]]链类似，任何通过原型链可以访问（并且是enumberable）到的属性都会被枚举，使用in操作符来检查属性在对象中是否存在同样会查找对象的整条原型链

```javascript
var anotherObject = {
	a: 2
};

// 创建一个关联到anotherObject的对象
var myObject = Object.create(anotherObject);

for(var k in myObject){
	console.log("found: " + k);
}

("a" in myObject);		// true
```

当你通过各种语法进行属性查找时都会查找[[prototype]]链，直到找到属性或者查找完整条原型链

##### Object.prototype

所有普通的[[prototype]]链最终都会指向内置的Object.prototype，可以理解为[[prototype]]链的`顶端`使Object.prototype对象，所以包含JavaScript许多通用功能

比如：.toString(..)、.valueOf(..)、.hasOwnProperty(..)、.isPrototypeOf(..)

##### 属性设置和屏蔽

完整的讲述一下给一个对象设置属性：

```javascript
myObject.foo = "bar";
```

对应关系：

| 对象是否包含属性 | 原型链上foo的状态      | 结果                                  |
| :--------------- | ---------------------- | :------------------------------------ |
| 已存在           | /                      | 修改对象的foo属性                     |
| 不直接存在       | 遍历原型链，如果不存在 | 直接添加到myObject                    |
| 已存在           | （已存在）             | 需要了解myObject会屏蔽原型链的所有foo |
| 不存在           | 存在且writable: true   | 在myObject中添加屏蔽属性foo           |
| 不存在           | 存在且writable: false  | 无法修改已有属性                      |
| 不存在           | foo是个setter          | 一定会调用这个setter                  |

注意：当writable为false，或在myObject创建屏蔽属性，严格模式下会报错，否则会被忽略

大多数开发人员认为一定会发生屏蔽，但是综上可知还有两种情况的存在，如果想在另外两种情况下也屏蔽foo，那么就不能使用=来赋值，而是使用Object.defineProperty(..)向**myObject**添加foo

**注意：**倒数第二种情况最意外，writable: false会**阻止**原型链下层隐式创建（屏蔽）同名属性，主要是为了`模拟类属性的继承`，可以把原型链上层的foo看作是父类的属性，会被myObject继承，这样一来myObject的foo属性也是只读，无法创建。**但请再注意：**事实上不会发生继承复制，但是myObject会因为其他对象中有一个只读foo就不能包含foo属性，而且只存在于 = 赋值中，使用Object.defineProperty(..)不会受到影响

如果要对屏蔽方法**（也就是原型链上层的方法）**进行委托得使用`丑陋的显示伪多态`**（也就是重写）**，通常来讲，使用屏蔽**（也就是writable: false）**，得不偿失，尽量避免使用。之后会介绍不使用屏蔽的更加简洁的设计模式

有些会产生**隐式**的屏蔽，思考下列代码：

```javascript
var anotherObject = {
    a: 2
};

var myObject = Object.create(anotherObject);

anotherObject.a;	// 2
myObject.a;			// 2

anotherObject.hasOwnProperty("a");	// true
myObject.hasOwnProperty("a");		// false

("a" in anotherObject);
("a" in myObject);

myObject.a++;		// 隐式屏蔽！

anotherObject.a;	// 2
myObject.a;			// 3

myObject.hasOwnProperty("a");		// true
```

hasOwnProperty：只检查是否在对象中

**解读：**myObject.a++相当于myObject.a = myObject.a + 1，`++操作先通过[[Prototype]]查找属性a并且从anotherObject.a获取当前属性值2，再给这个值+1，接着用[[Put]]将值3赋给myObject中新创建的屏蔽属性a`

#### “类”

JavaScript只有对象，可以不通过类创建，所以才是真正意义上的面向对象

##### "类函数"

JavaScript中有个无耻的行为就是：模仿类

所有**函数**默认都会拥有一个名为prototype的**公有且不可枚举**的属性，会指向另一个对象：

```javascript
function Foo(){
	// ...
}

Foo.prototype;	// { }
```

这个对象通常被称为Foo的原型，但是这个术语实际上会带来误导

抛开名字不谈，这个对象到底是什么？

**最直接的解释是：**调用new Foo()创建的每个对象最终都会被[[Prototype]]链接到"Foo.prototype"对象

验证一下：

```javascript
function Foo(){
	// ...
}

var a = new Foo();

Object.getPrototypeOf(a) === Foo.prototype;		// true
```

getPrototypeOf()返回指定对象的原型

调用new Foo()会创建a，把a链接到Foo.prototype所指向的对象

**区别：**

- 面向类：类可以被复制（或者实例化）多次，之所以会这样是因为实例化（或继承）一个类就意味着把类的行为复制到物理对象中
- JavaScript：不能创建一个类多个实例，只能创建多个对象，它们[[Prototype]]关联的是同一个对象，默认情况下不会复制，因为这些对象之间不会完全失去联系，是相互关联的

new Foo()产生一个新对象(暂且称为a)，新对象内部链接[[Prototype]]关联到Foo.prototype对象

最终得到了两个对象，它们之间相互关联，所以我们并没有初始化一个类，只是让两个对象相互关联

new Foo()并没有直接关联，这个是间接完成的目标：一个关联到其他对象的新对象

更直接的方法：Object.create(..)

###### 关于名称

[[Prototype]]机制通常被称为原型继承，但是这个组合属于严重影响了大家对于JavaScript的理解

例如：

- **继承：**继承意为赋值操作，但是JavaScript默认不会复制对象的属性，相反只会在对象之间创建关联，这样一个对象就可以通过委托访问另一个对象的属性和函数
- **差异继承：**基本原则是在描述对象行为时，使用其不同于普遍描述的特质，比如在描述汽车时会描述是四个轮子的交通工具，而不会重复描述交通工具的通用特性（比如有引擎）

默认情况下，对象不会像差异继承暗示的那样通过复制生产，当然也不适合用来描述JavaScript的[[Prototype]]机制

##### "构造函数"

代码如下：

```javascript
function Foo(){
	// ...
}

Foo.prototype;	// { }
```

到底是什么让我们认为Foo是一个类呢？

一个是看到了new关键字，另一个是Foo()的调用方式很像初始化类时类构造函数的调用方式

除了令人迷惑的"构造函数"，Foo.prototype还有个绝招

```javascript
function Foo(){
	// ...
}

Foo.prototype.contructor === Foo;	// true

var a = new Foo();
a.constructor === Foo;	// true
```

Foo.prototype默认有一个共有且不可枚举的属性.constructor，这个属性引用的是对象关联的函数，此外可以通过"构造函数"调用new Foo()创建的对象也有.constructor属性，指向“创建这个函数的对象”

“类”名首字母要大写

###### 构造函数还是调用

上面的代码很容易让人误解Foo是一个构造函数，因为我们用new来调用它并且看到它构造了一个对象

**函数本省不是构造函数，**当我们在普通函数前**加上new关键字**之后，就会把函数调用变成一个**构造函数调用，**实际上，new会劫持所有普通函数并且构造对象的形式来调用它

比如：

```javascript
function NothingSpecial(){
	console.log("Don't mind me!");
}

var a = new NothingSpecial();	// Don't mind me!

a;	// {}
```

NothingSpecial是普通函数，只是new调用时，无论如何都会构造一个对象并赋值给a，这个调用是一个构造函数调用，但NothingSpecial本身不是一个构造函数

**“构造函数”是所有带new的函数调用**

##### 明天这一章必定全部记录完毕