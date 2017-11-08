# 第二章：this豁然开朗

在第一章中，我们摒弃了种种对`this`的误解，并且知道了`this`是一个完全根据**调用点**(函数时如何被调用的)而为每次函数调用建立的绑定。

## 调用点(Call-site)

为了理解`this`绑定，我们不得不理解调用点:函数在代码中被调用的位置(**不是被声明的位置**)。我们必须考察调用点来回答这个问题：这个`this`指向什么？

一般来说寻找调用点就是："找到一个函数是在哪里被调用的", 但它不总是那么简单，比如某些特定的编码模式会使真正的调用点变得不那么明确。

考虑**调用栈(call-stack)**(使我们到达当前执行位置而被调用的所有方法的堆栈)是十分重要的。我们关心的调用点就位于当前执行中的函数之前的调用。

我们来展示一下调用栈和调用点：

```js
function baz(){
  // 调用栈是： `baz`
  // 我们的调用点是 global scope(全局作用域)

  console.log("baz");
  bar(); // <-- `bar` 的调用点
}

function bar(){
  // 调用栈是： `baz` -> `bar`
  // 我们的调用点位于 `baz`

  console.log("bar");
  foo(); // <-- `foo` 的 call-site
}

function foo(){
  // 调用栈是： `baz` -> `bar` -> `foo`
  // 我们的调用点位于 `bar`

  console.log("foo");
}

baz(); // <-- `baz`的调用点
```

在分析代码来寻找(从调用栈中)真正的调用点时要小心，因为它是影响`this`绑定的唯一因素。

**注意**：你可以通过按顺序观察函数的调用链在你的大脑中建立调用栈的视图，就像我们在上面代码段中的注释那样。但是这很痛苦而且易错。另一种观察调用栈的方式是使用你的浏览器的调试工具。大多数现代的桌面浏览器都內建开发者工具，其中就包含JS调试器。在上面的代码段中，你可以在调试工具中为`foo()`函数的第一行设置一个断点，或者简单的在这第一行上插入一个`debugger`语句。当你运行这样网页时，调试工具将会停止在这个位置，并且像你展示一个达到这一行之前所有被调用过的函数的列表，这就是你的函数调用栈。所以，如果你想调查`this`绑定，可以使用开发者工具取得调用栈，之后从上向下找到第二个记录，那就是你真正的调用点。

## 仅仅是规则

现在我们将注意力转移到调用点如何决定在函数执行期间`this`指向哪里。

你必须考察调用点并判定4种规则中的哪一种适用。我们将首先独立地解释一下这四种规则中的每一种，之后我们来展示一下如果有多种规则可以适用于调用点时，它们的优先顺序。

## 默认绑定

我们要考察的第一种规则源于函数调用的最常见的情况：独立函数调用。可以认为这种`this`规则是在没有其他规则适用的默认规则。

```js
function foo(){
  console.log(this.a);
}

var a = 2;

foo(); // 2
```

第一点要注意的，如果你还没有察觉到，是在全局作用域中的声明变量，也就是`var a = 2`，是全局对象的同名属性的同意词。他们不是互相拷贝对方，他们就是彼此。

第二，我们看到当`foo()`被调用时，`this.a`解析为我们全局变`a`。为什么？因为在这种情况下，对此方法调用的`this`实施了默认绑定，所以使`this`指向了全局对象。

我们怎么知道这里适用默认绑定？我们考察调用点来看看`foo()`是如何被调用的。在我们的代码段中，`foo()`是被一个直白的，毫无修饰的函数引用调用的。没有其他的我们将要展示的规则适用于这里，所以默认绑定在这里适用。

如果`strict mode`在这里生效，那么对于默认绑定来说全局对象是不合法的，所以`this`将被设置为`undefined`。

```js
function foo(){
  "use strict"
  console.log(this.a);
}

var a = 2;

foo();  // TypeError:`this` is `undefined`
```

一个微妙但是重要的细节是：即使所有的`this`绑定规则都是完全基于调用点的，但如果`foo()`的内容没有在`strict mode`下执行，对于默认绑定来说全局对象是唯一合法的；`foo()`的调用点的`strict mode`状态与此无关。

```js
function foo(){
  console.log(this.a);
}

var a = 2;

(function(){
  "use strict"
  foo(); // 2
})();
```

**注意**：在你的代码中故意混用`strict mode`和非`strict mode`通常是让人皱眉的。你的程序整体可能应当不是**Strict**就是**非Strict**。然而，有时你可能会引用与你的**Strict**模式不同的第三方包，所以对这些微妙的兼容性细节要多加小心。

## 隐含绑定

另一种要考虑的规则是：调用点是否有一个环境对象，也称为拥有者或容器对象。

```js
function foo(){
  console.log(this.a);
}

var obj ={
  a:2,
  foo:foo
};

obj.foo();  //2
```

首先，注意`foo()`被声明然后作为引用属性添加到`obj`上的方式。无论`foo()`是否一开始就在`obj`上被声明，还是后来作为引用添加(如上面代码所示)。这个**函数**都不被`obj`所真正"拥有"或"包含"。

然而，调用点使用`obj`环境来**引用**函数，所以你可以说`obj`对象在函数被调用的时间点上"拥有"或"包含"这个**函数引用**。

无论你怎么称呼这个模式，在`foo()`被调用的位置上，它被冠以一个指向`obj`的对象引用。当一个方法引用存在一个环节对象时，隐含绑定规则会说：是这个对象应当被用于这个函数调用的`this`绑定。

因为`obj`是`foo()`调用的`this`,所以`this.a`就是`obj.a`的同义词。

只有对象属性引用链的最后一层是影响调用点的。比如：

```js
function foo(){
  console.log(this.a);
}

var obj2 = {
  a:42,
  foo:foo
};

var obj1 = {
  a:2,
  obj2:obj2
};

obj1.obj2.foo(); // 42
```

## 隐含丢失

`this`绑定最让人沮丧的事情之一，就是当一个隐含绑定丢失了它的绑定，这通常意味着它会退回到默认绑定，根据`strict mode`的状态，其结果不是全局对象就是`undefined`。

```js
function foo(){
  console.log(this.a);
}

var obj = {
  a:2,
  foo:foo
};

var bar = obj.foo;  // 函数引用！
var a = "oops, global"; // `a` 也是一个全局对象的属性

bar();  // "oops, global"
```

尽管`bar`似乎是`obj.foo`的引用，实际上它只是另一个`foo`本身的引用而已。另外，起作用的调用点是`bar()`,一个直白，毫无修饰的调用，因此默认绑定适用于这里。

这种情况发生的更加微妙，更常见，而且更意外的方式，是当我们考虑传递一个回调函数时：

```js
function foo(){
  console.log(this.a);
}

function doFoo(fn){
  // `fn`不过是`foo`的另一个引用

  fn(); // <-- 调用点
}

var obj = {
  a:2,
  foo: foo
};

var a = "oop, golbal"; // `a` 也是一个全局对象的属性

doFoo(obj.foo);  // "oops, global"
```

参数传递仅仅是一种隐含地赋值，而且因为我们在传递一个函数，它是一个隐含地引用赋值,所以最终结果和我们前一个代码段一样。

那么如果接收你所传递回调的函数不是你的，而是语言内建的呢？没有区别，同样的结果。

```js
function foo(){
  console.log(this.a);
}

var obj = {
  a:2,
  foo:foo
};

var a = "oops, global";  // `a`也是一个全局对象的属性

setTimeout(obj.foo, 100);  // "oops, global"
```

把这个粗糙的，理论上的`setTimeout()`假想实现当做JavaScript环境内建的实现的话：

```js
function setTimeout(fn, delay){
  // (通过某种方法)等待 `delay`毫秒

  fn();  // <-- 调用点！
}
```

正如我们刚刚看到的，我们的回调函数丢掉他们的`this`绑定是十分常见的事情。但是`this`使我们吃惊的另一种方式是，接收我们回调的函数故意改变调用的`this`。那些很流行的JavaScript库中的事件处理器就十分喜欢强制你的回调的`this`指向触发事件的DOM元素。虽然有时这很好用，但其他时候这简直能气死人。不幸的是，这些工具很少给你选择。

不管哪一种意外改变`this`的方式，你都不能真正地控制你的回到函数引用将如何被执行，所以没有办法控制调用点给你一个故意的绑定。

## 明确绑定

用我们刚看到的隐含绑定，我们不得不改变目标对象使它自身包含一个对函数的引用，而后使用这个函数引用属性来间接地(隐含地)将`this`绑定到这个对象上。

但是，如果你想强制一个函数调用使某个特定对象作为`this`绑定，而不在这个对象上放置一个函数引用属性呢？

JavaScript语言中的"所有"函数都有一些工具(通过他们的`[[Prototype]]`)可以用于这个任务。具体地的说，函数拥有`call(..)`和`apply(..)`方法。从技术上讲，JavaScript宿主环境有时会提供一些(说的好听点儿)很特别的函数，它们没有这些功能。但这很少见。绝大多数被提供的函数，当然你还将创建的所有的函数，都可以访问`call(..)`和`apply(..)`。

这些工具如何工作？它们接收的第一个参数都是一个用于`this`的对象，之后使用这个指定的`this`来调用函数。因为你已经直接指明你想让`this`是什么，所以我们称这种方式为明确绑定。

```js
function foo(){
  console.log(this.a);
}

var obj = {
  a:2
};

foo.call(obj); // 2
```

通过`foo.call(..)`使用明确绑定来调用`foo`,允许我们强制函数的`this`指向`obj`。

如果你传递一个简单基本类型值(`string`, `boolean`,或`number`类型)作为`this`绑定，那么这个基本类型值会被包装在它的对象类型中(分别是`new String(..)`, `new Boolean(..)`,`new Number(..)`)。这通常被称为"封箱"。

**注意**：就`this`绑定的角度来讲，`call(..)`和`apply(..)`是完全一样的。它们确实在处理其他参数上的方式不同，但那不是我们当关心的。

不幸的是,单独依靠明确绑定仍然不能为我们先前提到的问题提供解决方案，也就是函数"丢失"自己原本的`this`绑定，或者被第三方框架覆盖等等问题。

## 硬绑定

但是有一个明确绑定的变种确实可以实现这个技巧。

```js
function foo(){
  console.log(this.a);
}

var obj = {
  a:2
}

var bar = function(){
  foo.call(obj);
};

bar(); // 2
setTimeout(bar, 100);  //2

// `bar`将`foo`的`this`硬绑定到`obj`
// 所以它不可以被覆盖
bar.call(window);  // 2
```

我们来看看这个变种是如何工作的。我们创建了一个函数`bar()`,在它的内部手动调用`foo.call(obj)`,由此强制`this`绑定到`obj`并调用`foo`。无论你过后怎么样调用函数`bar`,它总是手动使用`obj`调用`foo`。这种绑定即明确又坚定，所以我们称之为硬绑定。

用硬绑定将一个函数包装起来的最典型的方法，是为所有传入的参数和传出的返回值创建一个通道：

```js
function foo(something){
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a:2
};

var bar = function(){
  return foo.apply(obj, arguments);
};

var b = bar(3);  // 2 3
console.log(b); // 5
```

另一种表达这种模式的方法是创建一个可复用的帮助函数：

```js
function foo(something){
  console.log(this.a, something);
  return this.a + something;
}

// 简单的`bind`帮助函数
function bind(fn, obj){
  return function(){
    return fn.apply(obj, arguments);
  };
}

var obj = {
  a:2
};

var bar = bind(foo, obj);
var b = bar(3); // 2 3
console.log(b); // 5
```

由于硬绑定是一个如此常用的模式，它已作为ES5的内建工具提供： `Function.prototype.bind`：

```js
function foo(something){
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a:2
};

var bar = foo.bind(obj);
var b = bar(3); // 2 3
console.log(b); // 5
```

`bind(..)`返回一个硬绑定的新函数，它使用你指定的`this`环境来调用原本的函数。

**注意**:在ES6中,`bind(..)`生成的硬绑定函数有一个名为`.name`的属性，它源自于原始的目标函数(target function)。举例来说：`bar = foo.bind(..)`应该有一个`bar.name`属性，它的值为`"bound foo"`，这个值应当会显示在调用栈轨迹的函数调用名称中。

## API调用的"环境"

确实，许多库中的函数，和许多在JavaScript语言以及宿主环境中的内建函数，都提供一个可选参数，通常称为"环境(context)"，这种设计作为一种替代方案来确保你的回调函数使用特定的`this`而不必非得使用`bind(..)`。

```js
function foo(el){
  console.log(el, this.id);
}

var obj = {
  id: "awesome"
};

// 使用`obj`作为`this`来调用`foo(..)`
[1,2,3].forEach(foo, obj); // 1 awesone   2 awesome  3 awesome
```

## new 绑定(new Binding)

第四种也是最后一种`this`绑定规则，要求我们重新思考JavaScript 中关于函数和对象的常见误解。

在传统的面向类语言中，"构造器"是附着在类上的一种特殊方法，当使用`new`操作符来初始化一个类时，这个类的构造器就会被调用。通常看起来像这样：

```js
something = new MyClass(..);
```

JavaScript 拥有`new`操作符，而且使用它的代码模式看起来和我们在面向类语言中看到的基本一样；大多数开发者猜测JavaScript机制在作某种相似的事情。但是，实际上JavaScript的机制和`new`在JS中的用法所暗示的面向类的功能没有任何联系。

首先，让我们重新定义JavaScript的"构造器"是什么。在JS中，构造器**仅仅是一个函数**,它们偶然地与前置的`new`操作符一起调用。它们不依附于类，它们也不初始化一个类。它们甚至不是一种特殊的函数类型。它们本质上只是一般的函数，在被使用`new`来调用时改变了行为。

例如,引用ES5.1的语言规范`Number(..)`函数作为一个构造器来说：

> 15.7.2 Number构造器
> 当Number作为new表达式的一部分被调用时，它是一个构造器：它初始化这个新创建的对象。

所以，可以说任何函数，包括像`Number(..)`这样的内建对象函数都可以在前面加上`new`来被调用，这使函数调用成为一个 构造器调用(constructor call)。这是一个重要而微妙的区别：实际上不存在"构造器函数"这样的东西，而只有函数的构造器调用。

当在函数前面被加入`new` 调用时，也就是构造器调用时，下面这些事情会自动完成:

1  一个全新的对象会凭空创建(就是被构建)。

2  这个新构建的对象会被接入原型链([[Prototype]] -linked)。

3  这个新构建的对象被设置为函数调用的`this`绑定。

4  除非函数返回一个它自己的其他**对象**,否则这个被`new`调用的函数将自动返回这个新构建的对象。

步骤1，3和4是我们当下要讨论的。我们现在跳过第二步。

```js
function foo(a){
  this.a = a;
}

var bar = new foo(2);
console.log(bar.a);  // 2
```

通过在前面使用`new`来调用`foo(..)`，我们构建了一个新的对象作为`foo(..)`调用的`this`。`new`是函数调用可以绑定`this`的最后一种方式，我们称之为new绑定(new binding)。

## 一切皆有顺序

如此，我们已经揭示了函数调用中的四种`this`绑定规则。你需要做的一切就是找到调用点然后考察哪一种规则适用于它。但是，如果调用点上有多种规则使用呢？这些规则一定有一个优先顺序，我们下面就来展示这些规则以什么样的优先顺序实施。

很显然，默认绑定在四种规则中的优先权最低的。

隐含绑定和明确绑定哪一个更优先呢？我们来测试一下：

```js
function foo(){
  console.log(this.a);
}

var obj1 = {
  a:2,
  foo:foo
};

var obj2 = {
  a:3,
  foo:foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call(obj2); // 3
obj2.foo.call(obj1); // 2
```

所以，明确绑定的优先权要高于隐含绑定，这意味着你应当在考察隐含绑定之前首先考察明确绑定是否适用。

现在，我们只需要搞清楚new绑定的优先级于何处。

```js
function foo(something){
  this.a = something;
}

var obj1 = {
  foo:foo
};

var obj2 = {};

obj1.foo(2);
console.log(obj1.a); // 2

obj1.foo.call(obj2, 3);
console.log(obj2.a); // 3

var bar = new obj1.foo(4);
console.log(obj1.a); // 2
console.log(bar.a); // 4
```

好了，new绑定的优先级要高于隐含绑定。

**注意**:`new`和`call/apply`不能同时使用,所有`new foo.call(obj1)`是不允许的，也就是不能直接对比测试new绑定和明确绑定。但是我们依然可以使用硬绑定测试这两个规则的优先级。

在我们进入代码探索之前，回想一下硬绑定物理上是如何工作的，也就是`Function.prototype.bind(..)`创建了一个新的包装函数,这个函数被硬编码为忽略它自己的`this`绑定(不管它是什么)，转而手动使用我们提供的。

因此，这似乎看起来很明显，硬绑定的优先级要比new绑定高，而且不能被`new`覆盖。

```js
function foo(something){
  this.a = something;
}

var obj1 = {};
var bar = foo.bind(obj1);
bar(2);

console.log(obj1.a);  // 2

var baz = new bar(3);
console.log(obj1.a); // 2
console.log(baz.a);  // 3
```

`bar`是硬绑定到`obj1`的，但是`new bar(3)`并**没有**像我们期待的那样将`obj1.a`变为`3`。反而，应绑定(到obj1)的`bar(..)`调用可以被`new`所覆盖。因为`new`被实施，我们得到一个名为`baz`的新创建的对象，而且我们确实看到`baz.a`的值为`3`。

如果你回头看看我们的山寨绑定帮助函数，这很令人吃惊：

```js
function bind(fn, obj){
  return function(){
    fn.apply(obj, arguments);
  }
}
```

如果你推导这段帮助代码如何工作，会发现对于`new`操作符调用来说没有办法像我们观察到的那样，将绑定到的`obj`的硬绑定覆盖。

但是ES5内建`Function.prototype.bind(..)`更加精妙，实际上十分精妙。这里是MDN网页上为`bind(..)`提供的(稍稍格式化后的)polyfill：

```js
if(!Function.prototype.bind){
  Function.prototype.bind = function(oThis){
    if(typeof this !== "function"){
      // 可能与ECMAScript 5 内部的 IsCallable 函数最接近的东西
      throw new TypeError("Function.prototype.bind - what " +
      "is trying to be bound is not callable");
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP = function(){},
        fBound = function(){
          return fToBind.apply(
                        (this instanceof fNOP &&
                         oThis ? this : oThis
                        ),
                        aArgs.concat(Array.prototype.slice.call(arguments))
          );
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  }
};
```

**注意**：就将与`new`一起使用的硬绑定函数(参照下面来看为什么这有用)而言，上面的`bind(..)`polyfill与ES5中内建的`bind(..)`是不同的。因为polyfill不能像内建工具那样，没有`.prototype`就能创建函数，这里使用了一些微妙而间接的方法来近似模拟相同的行为。如果你打算将硬绑定函数和`new`一起使用而且依赖于这个polyfill，应当多加小心。

允许`new`进行覆盖的部分是这里:

```js
this instanceof fNOP &&
oThis ? this : oThis

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

我们不会实际深入解释这个花招是如何工作的(这很复杂而且超出了我们当前讨论范围)，但实质上这个工具判断硬绑定函数是否通过`new`被调用的(导致一个新构建的对象作为它的`this`)，如果是，它就用那个新构建的`this`而非先前为`this`指定的硬绑定。

为什么`new`可以覆盖 硬绑定这件事很有用？

这种行为的主要原因是，创建一个实质上忽略`this`的硬绑定而预先设置一部分或所有的参数的函数(这个函数可以与`new`一起使用来构建对象)。`bind(..)`的一个能力是，任何在第一个`this`绑定参数之后被传入的参数，默认地作为当前函数的标准参数(技术上这称为"局部应用"，是一种柯里化)。

```js
function foo(p1, p2){
  this.val = p1 + p2;
}

// 在这里使用`null`是因为在这种场景下我们不关心`this`的硬绑定
// 而且反正它将会被 `new` 调用覆盖掉！

var bar = foo.bind(null, "p1");
var baz = new bar("p2");

baz.val; // p1p2
```

## 判定 this

现在，我们可以按照优先顺序来总结一下从函数调用点来判定`this`的规则了。按照这个顺序来问问题，然后在第一个规则适用的地方停下来。

1. 函数是通过`new`被调用的吗(new绑定)?如果是，`this`就是新构建的对象。

`var bar = new foo()`

2. 函数是通过`call`或者`apply`被调用(明确绑定),甚至是隐藏在`bind`硬绑定之中吗？如果是，`this`就是那个被明确指定的对象。

`var bar = foo.call(obj2)`

3. 函数时通过环境对象(也称为拥有者或者容器对象)被调用的吗(隐含绑定)?如果是，`this`就是那个环境对象。

`var bar = obj1.foo()`

4. 否则，使用默认的`this`(默认绑定)。如果`strict mode`下就是`undefined`，否则是`global`对象。

`var bar = foo()`

以上，就是理解对于普通的函数调用来说的`this`绑定规则所需的全部。

## 绑定的特例

正如通常的那样，对于"规则"总有一些例外。

在某些场景下`this`绑定会让人很吃惊，比如在你视图实施一种绑定，然而最终得到的确实默认绑定规则的绑定行为。

## 被忽略的`this`

如果你传递`null`或`undefiend`作为`call`、`apply`或`bind`的`this`绑定参数，那么这些值会被忽略掉，取而代之的是默认绑定规则将适用于这个调用。

```js
function foo(){
  console.log(this.a);
}

var a = 2;

foo.call(null); // 2
```

为什么你会向`this`绑定故意传递像`null`这样的值？

一个很常见的做法是，使用`apply(..)`来将一个数组散开，从而作为函数调用的参数。相似地，`bind(..)`可以柯里化参数(预设值),也可能非常有用。

```js
function foo(a, b){
  console.log("a:" + a + ",b:" + b);
}

// 将数组散开作为参数
foo.apply(null, [2, 3]); // a:2, b:3

// 用`bind(..)`进行柯里化
var bar = foo.bind(null, 2);
bar(3); // a:2, b:3
```

这两种工具都要求第一个参数是`this`绑定。如果目标函数不关心`this`，你就需要一个占位值，而且正如这个代码段中展示的，`null`看起来是一个合理的选择。

**注意**：虽然我们在这本书中没有涵盖，但是ES6中有一个扩散操作符: `...`, 它让看无需使用`apply(..)`而在语法上将一个数组"散开"作为参数，比如`foo(...[1,2])`表示`foo(1,2)` --- 如果`this`绑定没有必要，可以在语法上回避它。不幸的是柯里化在ES6中没有语法上的替代品，所以`bind(..)`调用的`this`参数依然需要注意。

可是，在你不关心`this`绑定而一直使用`null`的时候，有些潜在的"危险"。如果你这样处理一些函数调用(比如，不归你管控的第三方包)，而且那些函数确实使用了`this`引用，那么默认绑定规则意味着它可能会不经意间引用(或者改变)`global`对象(在浏览器中是`window`)。

很显然，这样的陷阱会导致多种非常难诊断和追踪的bug。

## 更安全地`this`

也许某些"更安全"的做法是：为了`this`而传递一个特殊创建好的对象，这个对象保证不会对你的程序产生副作用。从网络学或军事上借用一个词，我们可以建立一个"DMZ"(非军事区)对象 --- 只不过是一个完全为空， 没有委托的对象。

如果我们为了忽略自己认为不用关心的`this`绑定，而总是传递一个DMZ对象，那么我们就可以确定任何对`this`的隐藏或意外的使用将会被限制在这个对象中，也就是说这个对象将`global`对象和副作用隔离开来。

因为这个对象是完全空的，我个人喜欢给它一个变量名为`ø`(空集合的数学符号的小写)。如果你不喜欢`ø`符号，或者你的键盘没那么好打，你当然可以叫它任意你希望的名字。

无论你叫它什么，创建**完全为空额对象**的最简单方法就是 `Object.create(null)`。这个方法和`{}`很相似，但是没有指向`Object.prototype`的委托，所以它比`{}`"空得更彻底"。

```js
function foo(a, b){
  console.log("a:" + a + ",b:" + b);
}

// 我们的 DMZ 空对象
var ø = Object.create(null);

// 将数组散开作为参数
foo.apply(ø, [2,3]); // a:2, b:3

// 用`bind(..)`进行 currying
var bar = foo.bind(ø, 2);
bar(3); // a:2, b:3
```

不仅在功能上更"安全"，`ø`还会在代码风格上产生些好处，它在语义上可能会比`null`更清晰的表达"我想让`this`为空"。当然，你可以随自己喜欢来称呼你的DMZ对象。

## 间接

另一个要注意的是，你可以(有意无意地！)创建对函数的"间接引用"，在那样的情况下，当那个函数引用被调用时，默认绑定规则也会适用。

一个最常见的间接引用产生方式是通过赋值:

```js
function foo(){
  console.log(this.a);
}

var a = 2;
var o = {a:3, foo:foo};
var p = {a:4};

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式`p.foo = o.foo`的结果值是一个刚好指向底层函数对象的引用。如此，起作用的调用点就是`foo()`，而非你期待的`p.foo()`或`o.foo()`。根据上面的规则，默认绑定适用。

提醒：无论你如何得到适用默认绑定的函数调用，被调用函数的**内容**的`strict mode`状态 --- 而非函数的调用点 --- 决定了 `this`引用的值： 不是`global`对象(在非`strict mode`下)，就是`undefined`(在`strict mode`下)。

## 软化绑定(Softening Binding)

我们之前看到的硬绑定是一种通过将函数强制绑定到特定的`this`上，来防止函数调用在不经意间回退到默认绑定的策略(除非你用`new`去覆盖它!)。问题是，硬绑定极大地降低了函数的灵活性，阻止我们手动使用隐含绑定或后续的明确绑定来覆盖`this`。

如果有这样的办法就好了:为默认绑定提供不同的默认值(不是`global`或`undefined`)，同时保持函数可以通过隐含绑定或明确绑定技术来手动绑定`this`。

```js

if(!Function.prototype.softBind){
  Function.prototype.softBind = function(obj){
    var fn = this, // 调用函数
        curried = [].slice.call(arguments, 1), // 调用参数
        bound = function bound(){
          return fn.apply(
            (!this ||
                      (typeof window !== 'undefined' &&
                              this === window) ||
                      (typeof global !== 'undefined' &&
                              this === global)
            ) ? obj : this,
            curred.concat.apply(curried, arguments)
          );
        };
    bound.prototype = Object.create(fn.prototype);
    return bound;
  };
}

```

这里提供的`softBind(..)`工具的工作方式和ES5内建的`bind(..)`工具很相似，除了我们的软绑定行为。它用一种逻辑将指定的函数包装起来，这个逻辑在函数调用时检查`this`，如果它是`global`或`undefined`，就使用预先指定的默认值`object`，否则保持`this`不变。它也提供了可选的柯里化行为。

我们来看看它的用法：

```js
function foo(){
  console.log("name: " + this.name);
}

var obj = {name: "obj"},
    obj2 = {name: "obj2"},
    obj3 = {name: "obj3"};

var fooOBJ = foo.softBind(obj);

fooOBJ();  // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2

fooOBJ.call(obj3); // name: obj3

setTimeout(obj2.foo, 10); // name:obj  <-- 退回到软绑定

```

软绑定版本的`foo()`函数可以如展示的那样被手动`this`绑定到`obj2`或`obj3`，如果默认绑定使用时会退到`obj`。

## 词法`this`

我们刚刚涵盖了一般函数遵循的四种规则。但是ES6引入了一种不适用于这些规则的特殊的函数： 箭头函数。

箭头函数不是通过`function`关键字声明的，而是通过所谓的"大箭头"操作符: `=>`。 与使用四种标准的`this`规则不同的是，箭头函数从封闭它的(函数或全局)作用域采用`this`绑定。

```js
function foo(){
  // 返回一个箭头函数
  return (a) => {
    // 这里的`this`是词法上从`foo()`采用的
    console.log(this.a);
  }
}

var obj1 = {
  a: 2
};

var obj2 = {
  a: 3
}

var bar = foo.call(obj1);
bar.call(obj2); // 2, 不是3！
```

在`foo()`中创建的箭头函数在词法上捕获`foo()`被调用时的`this`，不管它是什么。因为`foo()`被`this`绑定到`obj1`, `bar`(被返回的箭头函数的一个引用)也将被`this`绑定到`obj1`。一个箭头函数的词法绑定是不能被覆盖的(就连`new`也不行!)。

最常见的用法就是用于回调

```js

function foo(){
  setTimeout(() => {
    // 这里的`this`是词法上从`foo`采用
    console.log(this.a);
  }, 100);
}

var obj = {
  a: 2
};
foo.call(obj); // 2
```

虽然箭头函数提供了使用`bind(..)`外，另外一种在函数上确保`this`的方式，这看起来很吸引人，但是重要的是要注意它们本质是使用广为人知的词法作用域来禁止了传统的`this`机制，在ES6之前，为此我们已经有了相当常用的模式，这些模式几乎和ES6的箭头函数的精神没有区别:

```js
function foo(){
  var self = this; // 词法上捕获`this`
  setTimeout(function(){
    console.log(self.a);
  }, 100);
}

var obj = {
  a:2
};

foo.call(obj); // 2
```

虽然对不想要`bind(..)`的人来说`self = this`和箭头函数都是看起来不错的"解决方案",但它们实质上逃避了`this`而非理解和接收它。

如果你发现你在写`this`风格的代码，但是大多数或全部时候，你都用词法上的`self = this`或箭头函数"技巧"抵御`this`机制，那么也许你应该:

1. 仅使用词法作用域并忘掉虚伪的`this`代码风格。
2. 完全接受`this`风格机制，包括在必要的时候使用`bind(..)`,并尝试避开`self = this`和箭头函数的"词法this"技巧。

一个程序可以有效地同时利用两种风格的代码，但是在同一个函数内部，特别是对同种类型的查找，混合这两中机制通常是自找很难维护的代码。
