# 第三章: 对象

在第一和第二章中，我们讲解了`this`绑定如何根据函数调用点指向不同的对象。但究竟什么是对象，为什么我们需要指向他们？

## 语法

对象来自于两种形式: 声明(字面)形式， 和构造形式。

一个对象的字面语法看起来像这样:

```js
var myObj = {
  key: value
};
```

构造看起来像这样:

```js
var myObj = new Object();
myObj.key = value;
```

构造形式和字面形式的结果是完全同种类型的对象。唯一真正的区别在于你可以向字面声明一次性添加一个或多个键/值对，而对于构造形式，你必须一个一个地添加属性。

**注意**: 像刚才展示的那样使用"构造形式"来创建对象是极其少见的。你很有可能总是想使用字面语法形式。这对大多数内建的对象也一样。

## 类型

对象是大多数JS程序依赖的基本构建块儿。它们是JS的六种主要类型(在语言规范中称为"语言类型")中的一种：

* string
* number
* boolean
* null
* undefiend
* objectt

**注意**:简单基本类型(string, number, boolean, null 和 undefined)自身不是object。null 有时会被当成一个对象类型，但是这种误解源自于一个语言中的Bug,它使得typeof null 错误地(而且令人困惑地)返回字符串"object"。实际上，null 是它自己的基本类型。

**一个常见的错误论断是"JavaScript中的一切都是对象"。这明显是不对的。**

对比来看，存在几种特殊的对象子类型，我们可以称之为复杂基本类型。

`function`是对象的一种子类型(技术上讲，叫做"可调用对象")。函数在JS中被称为"头等"类型，是因为它们基本上就是普通的对象，而且它们可以像其他普通的对象那样被处理。

数组也是一种形式的对象，带有特别的行为。数组在内容的组织上要稍稍比一般的对象更加结构化。

## 内建对象

有几种其他的对象子类型，通常称为內建对象。对于其中的一些来说，它们的名称看起来暗示着它们和它们对应的基本类型有着直接的联系，但事实上，它们的关系更复杂。

* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error

如果你依照和其他语言的相似性来看的话，比如Java语言的`String`类，这些内建类型有着实际类型的外观，甚至是类的外观，但是在JS中，它们实际上仅仅是内建的函数。这些内建的函数的每一个都可以被用作构造函数(也就是一个可以通过`new`操作符调用的函数),其结果是一个新构建的相应子类型的对象。

```js
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

var strObject = new String("I am a string");
typeof strObject; // "object"
strObject instanceof String; // true

// 考察 object 子类型
Object.prototype.toString.call(strObject); // [object String]
```

我们会在本章稍后详细地看到`Object.prototype.toString...`到底是如何工作的，但是简单地说，我们可以通过借用基本的默认`toString()`方法来考察内部子类型，而且你可以看到它揭示了`strObject`实际上是一个由`String`构造器创建的对象。

基本类型值`"I am a string"`不是一个对象，它是一个不可变的基本字面值。为了对它进行操作，比如检查它的长度，访问它的各个独立字符内容等等，都需要一个`String`对象。

幸运的是，在必要的时候语言会自动地将"string"基本类型强制转换为"String"对象类型，这意味着你几乎从不需要明确地创建对象。JS社区的绝大部分人都**强烈推荐**尽可能地使用字面量形式的值，而非使用构造的对象形式。

```js
var strPrimitive = "I am a string";
console.log(strPrimitive.length); // 13
console.log(strPrimitive.charAt(3)); // "m"
```

在这两个例子中，我们在字符串的基本类型上调用属性和方法，引擎会自动地将它强制转换为`String`对象，所以这些属性/方法的访问可以工作。

当使用如`42.359.toFixed(2)`这样的方法时，同样的强制转换也发生在数字基本字面量`42`和包装对象`new Number(42)`之间。同样的还有`Boolean`对象和`"boolean"`基本类型。

`null`和`undefined`没有对象包装的形式，仅有它们的基本类型值。相比之下，`Date`的值仅可以由它们的构造对象形式创建， 因为他们没有对应的字面形式。

无论使用字面还是构造形式，`Object`,`Array`,`Function`,`RegExp`(正则表达式)都是对象。在某些情况下，构造形式确实会比对应的字面形式提供更多的创建选项。因为对象可以被任意一种方式创建，更简单的字面形式几乎是所有人的首选。**仅仅在你需要使用额外的选项时使用构建形式**。

`Error`对象很少在代码中明示地被创建，它们通常在抛出异常时自动地被创建。它们可以由`new Error(..)`构造形式创建，但通常是不必要的。

## 内容

正如刚才提到的，对象的内容有存储在特定命名的位置上的(任意类型的)值组成，我们称这些值为属性。

有一个重要的事情需要注意: 当我们说"内容"时，似乎暗示着这些值实际上存储在对象内部，但那只不过是表面现象。引擎会根据自己的实现来存储这些值，而且通常都不是把它们存储在容器对象内部。在容器内存储的都是这些属性的名称，它们像指针(技术上讲，叫引用)一样指向值存储的地方。

```js
var myObject = {
  a: 2
};

myObject.a; // 2
myObject["a"]; // 2
```

为了访问`myObject`在位置`a`的值，我们需要使用`.`或`[]`操作符。`.a`语法通常称为"属性"访问，而`["a"]`语法通常称为"键"访问。在现实中，它们俩都访问相同的位置，而且会拿出相同的值，`2`,所以这些术语可以互换使用。从现在起，我们将使用最常见的术语 --- "属性访问"。

两种语法的主要区别在于,`.`操作符后面需要一个`标识符(Identifier)`兼容的属性名，而`[".."]`语法基本可以接受任何兼容UTF-8/unicode的字符串作为属性名。举个例子，为了引用一个名为"Super-Fun!"的属性，你不得不使用`[Super-Fun!]`语法访问，因为`Super-Fun!`不是一个合法的`Identifier`属性名。

而且，由于`[".."]`语法使用字符串的值来指定位置，这意味着程序可以动态地组建字符串的值

```js
var wantA = true;
var myObject = {
  a: 2
};

var idx;

if(wantA){
  idx = "a";
}

console.log(myObject[idx]; // 2
```

在对象中，属性名**总是**字符串。如果你使用`string`以外的(基本)类型值，它会首先被转换为字符串。这甚至包括在数组中常用于索引的数字，所以要小心不要将对象和数组使用的数字搞混了。

```js
var myObject = {};

myOject[true] = "foo";
myOject[3] = "bar";
myOject[myOject] = "baz";

myOject["true"]; // "foo"
myOject["3"]; // "bar"
myOject["[object Object]"]; // "baz"
```

## 计算型属性名

如果你需要将一个计算表达式作为一个键名称，那么我们刚刚描述的`myObject[..]`属性访问语法是十分有用的，比如`myObject[prefix + name]`。但是当使用字面对象语法声明时则没有什么帮助。

ES6加入了计算型属性名，在一个字面对象声明的键名称位置，你可以指定一个表达式，用`[]`括起来:

```js
var prefix = "foo";

var myObject = {
  [prefix + "bar"]: "hello",
  [prefix + "baz"]: "world"
}
myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

计算机属性名的最常见用法，可能是用于ES6的`Symbol`，简单地说，它们是新的基本数据类型，拥有一个不透明不可知的值(技术上讲是一个`string`值)。你将会被强烈地不鼓励使用一个`Symbol`的实际值(这个值理论上会因为JS引擎的不同而不同),所以`Symbol`的名称，比如`Symbol.Something`才是你会使用的：

```js
var myObject = {
  [Symbol.Something]: "hello world"
};
```

## 属性(Property) vs. 方法(Method)

有些开发者喜欢在讨论对一个对象的属性访问时做一个区别，如果这个被访问的值恰好是一个函数的话。因为这个诱使人们认为函数属于这个对象，而且在其他语言中，属于对象的函数被称作"方法",所以相对于"属性访问"，我们常能听到"方法访问"。

有趣的是，**语法规范也做出了同样的区别**。

从技术上讲，函数绝不会"属于"对象，所以, 说一个偶然在对象的引用上被访问的函数就自动地成为了一个"方法"，看起来有些像是牵强附会。

有些函数内部确实拥有`this`引用，而且有时这些`this`引用指向调用点的对象引用。但是这个用法确实没有使这个函数比其他函数更像"方法",因为`this`是在运行时在调用点动态绑定的，这使得它与这个对象的关系至多是间接的。

每次你访问一个对象的属性都是一个**属性访问**, 无论你得到什么类型的值。如果你恰好从属性访问中得到一个函数，它也没有魔法般地在那时成为一个"方法"。从一个属性访问得来的函数没有任何特殊性(隐含地`this`绑定的情况已经解释过了)。

```js
function foo(){
  console.log("foo");
}

var someFoo = foo; // 对`foo`的变量引用

var myObject = {
  someFoo: foo
};

foo; // function foo(){..}
someFoo; // function foo(){..}
myObject.someFoo; // function foo(){..}
```

`someFoo`和`myObject.someFoo`只不过是同一个函数的两个分离的引用，它们中的任何一个都不意味着这个函数很特别或被其他对象所"拥有"。如果上面的`foo()`定义里面拥有一个`this`引用，那么`myObject.someFoo`的隐含绑定将会是这个两个引用间**唯一**可以观察到的不同。它们中的任何一个都没有称为**方法**的道理。

**也有人会争辩**，函数成了方法，不是在定义期间，而是在调用的执行期间，根据它是如何在调用点被调用的(是否带有一个环境对象引用)。即便是这种解读也是有些牵强。

可能最安全的结论是，在JavaScript中，"函数"和"方法"是可以互换使用的。

**注意**:ES6加入了`super`引用，它通常是和`class`一起使用的。`super`的行为方式(静态绑定， 而非像`this`一样延迟绑定)，给了这种说法更多的权重:一个被`super`绑定到某处的函数比起"函数"更像一个"方法"。但是同样地，这仅仅是微妙的语义上的细微区别。

就算你声明一个函数表达式作为字面对象的一部分，那个函数都不会魔法般地属于这个对象 --- 仍然仅仅是同一个函数对象的多个引用罢了。

```js
var myObject = {
  foo: function foo(){
    console.log("foo");
  }
};

var someFoo = myObject.foo;
someFoo; // function foo(){..}
myObject.foo; // function foo(){..}
```

## 数组

数组也使用`[]`访问形式，但是如上面提到的，在存储值的方式和位置上它们的组织更加结构化(虽然仍然在存储值和类型上没有限制)。数组采用数字索引，这意味着值被存储的位置，通常称为下标，是一个非负整数，比如 `0`或`42`。

```js
var myArray = ["foo", 42, "bar"];

myArray.length; // 3
myArray[0]; // "foo"
myArray[2]; // "bar"
```

数组也是对象，所以虽然每个索引都是正整数，你还可以在数组上添加属性：

```js
var myArray = ["foo", 42, "bar"];

myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```

**注意**: 添加命名属性(不论是使用`.`还是`[]`操作符语法)不会改变数组的`length`所报告的值。

你可以把一个数组当做普通的键/值对象使用，并且从不添加任何数字下标，但这不是一个好主意，因为数组对它本来的用途有着特定的行为和优化方式，普通对象也一样。使用对象来存储键/值对，而用数组在数字下标上存储值。

**小心**: 如果你试图在一个数组上添加属性，但是属性名看起来像一个数字，那么最终它会成为一个数字索引(也就是改变了数组的内容)：

```js
var myArray = ["foo", 42, "bar"];

myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

## 复制对象

当开发者们初次拿起JavaScript语言时，最常需要的特性就是如何复制一个对象。看起来应该有一个内建的`copy()`方法，但是事情实际上比这复杂一些，因为在默认的请款下，复制的算法应当是什么，并不十分明确。

```js
function anotherFunction(){/**/}

var anotherObject = {
  c: true
};

var anotherArray = [];

var myObject = {
  a:2,
  b:anotherObject, // 引用，不是拷贝！
  c:anotherArray,  // 又一个引用！
  d:anotherFunction
};

anotherArray.push(anotherObject, myObject);
```

一个`myObject`的拷贝究竟应该怎么表现？

首先，我们应该回答它是一个浅(shallow)还是一个深(deep)拷贝？一个浅拷贝会得到一个新对象，它的`a`是值`2`的拷贝，但`b`，`c`和`d`属性仅仅是引用，它们指向被拷贝对象引用的相同位置。一个深拷贝将不仅复制`myObject`,还会复制`anotherObject`和`anotherArray`。但之后我们让`anotherArray`拥有`anotherObject`和`myObject`的引用，所以那些也应当被复制而不是保留引用。现在由于循环引用，我们得到了一个无限循环复制的问题。

我们应当检测循环引用并打破循环遍历吗(不管位于深处的，没有完全复制的元素)？我们应当报错退出吗？或者介于两者之间？

另外，"复制"一个函数意味着什么,也不是很清楚。有一些技巧，比如提取一个函数源代码的`toString()`序列化表达(这个源代码因实现不同而不同，而且根据被考察的函数的类型，其结果甚至在所有引擎上都不可靠)。

那么我们如何解决这些刁钻的问题？不同的JS框架都各自挑选自己的解释并且做出自己的选择。但是哪一种(如果有的话)才是JS应当作为标准采用的呢？

一个解决方案是，JSON安全的对象(也就是，可以被序列化为一个JSON字符串，之后还可以被重新解析为拥有相同的结构的值的对象)可以简单地这样复制:

```js
var newObj = JSON.parse(JSON.stringify(someObj));
```

当然，这要求你保证你的对象是JSON安全地。对于某些情况，这没有什么大不了的。而对另一些情况，这还不够。

同时，浅拷贝相当易懂，而且没有那么多问题，所以ES6为此任务已经定义了`Object.assign()`。`Object.assign(..)`接收目标作为第一个参数，然后是一个或多个源对象作为后续参数。它会在源对象上迭代所有的可枚举，owned keys(直接拥有的键),并把它们拷贝到目标对象上(仅通过 `=` 赋值)。它还会很方便地返回目标对象，正如下面你可以看到的:

```js
var newObj = Object.assign({}, myObject);

newObj.a; // 2
newObj.b === anotherObject; // true
newObj.c === anotherArray; // true
newObj.d === anotherFunction; // true
```

**注意**: 在下一部中，我们将讨论"属性描述符"并展示`Object.defineProperty(..)`的使用，然而在`Object.assign(..)`中发生的复制是单纯的`=`式赋值，所以任何在源对象属性的特殊性质(比如`writable`)在目标对象上**都不会保留**

## 属性描述符(Property Descriptors)

在ES5之前，JavaScript语言没有给出直接的方法，让你的代码可以考察或描述属性性质间的区别，比如属性是否为只读。

在ES5中，所有的属性都用**属性描述符**来描述。