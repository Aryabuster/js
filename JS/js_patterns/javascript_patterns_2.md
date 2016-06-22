#javascript patterns 2

##编写可维护的代码

https://github.com/TooBug/javascript.patterns/blob/master/chapter2.markdown

- 设计模式
- 编码模式
- 反模式

最后一个需要注意的是，对象有两大类：

- 本地对象（Native）：由ECMAScript标准定义的对象
- 宿主对象（Host）：由宿主环境创建的对象（比如浏览器环境）

**编写可维护的代码**

可维护的代码意味着代码是：

- 可读的
- 风格一致的
- 可预测的
- 看起来像是同一个人写的
- 有文档的

**减少全局变量**

“全局变量”是不在任何函数体内部声明的变量，或者是直接使用而未声明的变量。

在一个函数内定义的变量称作“本地变量”，本地变量在函数外部是不能被访问的。

容易产生命名冲突：

- 网页中使用了第三方的JavaScript库
- 网页中使用了广告代码
- 网页中使用了用以分析流量和点击率的第三方统计代码
- 网页中使用了很多组件、挂件和按钮等等

假设某一段第三方提供的脚本定义了一个全局变量result。随后你在自己写的某个函数中也定义了一个全局变量result。这时，第二个变量就会覆盖第一个，会导致第三方脚本工作不正常。

**减少全局变量的技巧和策略：**

比如使用命名空间或者即时函数等，但减少全局变量最有效的方法还是坚持使用var来声明变量。

**创建全局变量的方法**

- 你可以不声明而直接使用变量
- JavaScirpt中具有“隐式全局对象”的概念，也就是说任何不通过var声明的变量都会成为全局对象的一个属性

**创建全局变量的反模式**

就是在var声明中使用链式赋值的方法。在下面这个代码片段中，a是局部变量，但b是全局变量，而作者的意图显然不是这样：

```
// 反模式
function foo() {
    var a = b = 0;
    // ...
}
```

因为这里的计算顺序是从右至左的：首先计算表达式b=0，这里的b是未声明的；这个表达式的结果是0，然后通过var创建了本地变量a，并赋值为0。换言之，可以将代码写成这样：

```
var a = (b = 0);
```

如果变量b已经被声明，这种链式赋值的写法是可以使用的，不会意外地创建全局变量

```
function foo() {
    var a, b;
    // ...
    a = b = 0; // 两个都是本地变量
}
```

> 避免使用全局变量的另一个原因是出于可移植性考虑，如果你希望将你的代码运行于不同的平台环境（宿主），那么使用全局变量就非常危险。因为很有可能你无意间创建的某个全局变量在当前的平台环境中是不存在的，你以为可以安全地使用，而在另一个环境中却是本来就存在的。

**忘记var时的副作用**

隐式创建的全局变量和显式定义的全局变量之间有着细微的差别，就是通过delete来删除它们的时候表现不一致。

- 通过var创建的全局变量（在任何函数体之外创建的变量）不能被删除。
- 没有用var创建的隐式全局变量（不考虑函数内的情况）可以被删除。

```
// 定义三个全局变量
var global_var = 1;
global_novar = 2; // 反模式
(function () {
    global_fromfunc = 3; // 反模式
}());

// 尝试删除
delete global_var; // false
delete global_novar; // true
delete global_fromfunc; // true

// 测试删除结果
typeof global_var; // "number"
typeof global_novar; // "undefined"
typeof global_fromfunc; // "undefined"
```

在ES5严格模式中，给未声明的变量赋值会报错（比如这段代码中提到的两个反模式）。

**访问全局对象**

在浏览器中，我们可以随时随地通过window属性来访问全局对象（除非你定义了一个名叫window的局部变量）。但换一个运行环境这个window可能就换成了别的名字（甚至根本就被禁止访问全局对象了）。如果不想通过这种写死window的方式来访问全局变量，那么你可以在任意函数作用域内执行：

```
var global = (function () {
    return this;
}());
```
这种方式总是可以访问到全局对象，因为在被当作函数（而不是构造函数）执行的函数体内，**this总是指向全局对象**。但这种情况在ECMAScript5的严格模式中行不通，因此在严格模式中你不得不寻求其他的替代方案。比如，如果你在开发一个库，你会将你的代码包装在一个即时函数中（在第四章会讲到），然后从全局作用域给这个匿名函数传入一个指向this的参数。

**单var模式**

在函数的顶部使用唯一一个var语句是非常推荐的一种模式，它有如下一些好处：

- 可以在同一个位置找到函数所需的所有变量
- 避免在变量声明之前使用这个变量时产生的逻辑错误（参考下一小节“声明提前：分散的var带来的问题”）
- 提醒你不要忘记声明变量，顺便减少潜在的全局变量
- 代码量更少（输入代码更少且更易做代码优化）

```
function func() {
    var a = 1,
        b = 2,
        sum = a + b,
        myobject = {},
        i,
        j;
    // 函数体…
}
```

你可以使用一个var语句来声明多个变量，变量之间用逗号分隔，也可以在这个语句中加入变量初始化的部分。这是一种非常好的实践方式，可以避免逻辑错误（所有未初始化的变量都被声明了，且值为undefined），并增加了代码的可读性。过段时间后再看这段代码，你可以从初始化的值中大概知道这个变量的用法，比如你一眼就可看出某个变量是对象还是整数。

你可以在声明变量时做一些额外的工作，比如在这个例子中就写了sum=a+b这种代码。另一个例子就是当代码中用到对DOM元素时，你可以把DOM引用赋值的操作也放在这个变量声明语句中，比如下面这段代码：

```
function updateElement() {
    var el = document.getElementById("result"),
        style = el.style;
    // 使用el和style…
}
```
**声明提前：分散的var带来的问题**

JavaScript允许在函数的任意地方写任意多个var语句，但它们的行为会像在函数体顶部声明变量一样，这种现象被称为“声明提前”，当你在声明语句之前使用这个变量时，可能会造成逻辑错误。对于JavaScript来说，一旦在某个作用域（同一个函数内）里声明了一个变量，那么这个变量在整个作用域内都是存在的，包括在var声明语句之前的位置。看一下这个例子：

```
// 反模式
myname = "global"; // 全局变量
function func() {
    alert(myname); // "undefined"
    var myname = "local";
    alert(myname); // "local"
}
func();
```

这个例子中，你可能会期望第一个alert()弹出“global”，第二个alert()弹出“local”。这种结果看起来是合乎常理的，因为在第一个alert()执行时，myname还没有被声明，这时就应该“寻找”全局变量myname。但实际情况并不是这样，第一个alert()弹出“undefined”，因为myname已经在函数内被声明了（尽管声明语句在后面）。所有的变量声明都会被提前到函数的顶部，因此，为了避免类似带有“歧义”的程序逻辑，最好在使用之前一起声明它们。

上一个代码片段等价于下面这个代码片段：

```
myname = "global"; // 全局变量
function func() {
    var myname; // 等价于 -> var myname = undefined;
    alert(myname); // "undefined"
    myname = "local";
    alert(myname); // "local"
}
func();
```

>这里有必要对“变量提前”做进一步补充，实际上从JavaScript引擎的工作机制上看，这个过程稍微有点复杂。代码处理经过了两个阶段：**第一阶段是创建变量、函数和形参，也就是预编译的过程**，它会扫描整段代码的上下文；**第二阶段是在代码的运行时（runtime），这一阶段将创建函数表达式和一些非法的标识符（未声明的变量）。**（译注：这两个阶段并没有包含代码的执行，是在执行前的处理过程。）从实用性角度来讲，我们更愿意将这两个阶段归成一个概念“变量提前”，尽管这个概念并没有在ECMAScript标准中定义，但我们常常用它来解释预编译的行为过程。

###for循环

在for循环中，可以对数组或类似数组的对象（比如arguments和HTMLCollection对象）进行遍历，通常for循环模式形如：

```
// 非最优的循环方式
for (var i = 0; i < myarray.length; i++) {
    // 访问myarray[i]…
}
```

这种模式的问题是，每次遍历都会访问数组的length属性，这会降低代码运行效率，特别是当myarray不是一个数组而是一个HTMLCollection对象的时候。

HTMLCollection是由DOM方法返回的对象集合，比如：

- document.getElementsByName()
- document.getElementsByClassName()
- document.getElementsByTagName()

还有一些HTMLCollection是在DOM标准诞生之前就已经在用了并且现在仍然可用，包括：

- document.images  页面中所有的IMG元素
- document.links  页面中所有的A元素
- document.forms  页面中所有的表单
- document.forms[0].elements  页面中第一个表单的所有字段

这些对象的问题在于，它们都会实时查询文档（HTML页面）中的对象。也就是说每次通过它们访问集合的length属性时，总是都会去查询DOM，而DOM操则是很耗资源的。

更好的办法是在for循环中缓存要遍历的数组的长度，比如下面这段代码：

```
for (var i = 0, max = myarray.length; i < max; i++) {
    // 访问myarray[i]…
}
```

通过这种方法只需要获取length一次，然后在整个循环过程中使用它。

不管在什么浏览器中，在遍历HTMLCollection时缓存length都可以让程序执行的更快，可以提速2倍（Safari3）到190倍（IE7）不等。更多细节可以参照Nicholas Zakas的《高性能JavaScript》，这本书也是由O'Reilly出版。

需要注意的是，当你在循环过程中需要修改这个元素集合（比如增加DOM元素）时，你可能需要更新length。

按照“单var模式”，你可以将var提到循环的外部，比如：

```
function looper() {
    var i = 0,
        max,
        myarray = [];
    // …
    for (i = 0, max = myarray.length; i < max; i++) {
        // 访问myarray[i]…
    }
}

```

当你越来越依赖“单var模式”时，带来的好处就是提高了代码的一致性。而缺点则是在重构代码的时候不能直接复制粘贴一个循环体，比如，你正在将某个循环从一个函数复制至另外一个函数中，那么必须确保i和max也复制到新函数里，并且需要从旧函数中将这些没用的变量删除掉。

最后一个需要对循环做出调整的地方是将i++替换成为下面两者之一：

```
i = i + 1
i += 1
```

JSLint提示你这样做，是因为++和--实际上降低了代码的可读性，如果你觉得无所谓，可以将JSLint的plusplus选项设为false（默认为true）。稍后，在本书所介绍的最后一个模式中用到了:i += 1。

关于这种for模式还有两种变化的形式，做了少量改进，原因有二：

- 减少一个变量（没有max）
- 减量循环至0，这种方式速度更快，因为和零比较要比和非零数字或数组长度比较要高效的多

第一种变化形式是：

```
var i, myarray = [];
for (i = myarray.length; i--;) {
    // 访问myarray[i]…
}
```

第二种变化形式用到了while循环：

```
var myarray = [],
    i = myarray.length;
while (i--) {
    // 访问myarray[i]…
}
```

这些小改进只能体现在对性能要求比较苛刻的地方，此外，JSLint不推荐使用i--。

### for-in循环

for-in循环用于对非数组对象进行遍历。通过for-in进行循环也被称作“枚举”（enumeration）。

从技术上讲，for-in循环同样可以用于数组（JavaScript中数组也是对象），但不推荐这样做。当数组对象被扩充了自定义函数时，可能会产生逻辑错误。另外，for-in循环中属性的遍历顺序是不固定的，所以最好数组使用普通的for循环，对象使用for-in循环。

可以使用对象的hasOwnProperty()方法过滤来自原型链中继承来的属性，这一点非常重要。看一下这段代码：

```
// 对象
var man = {
    hands: 2,
    legs: 2,
    heads: 1
};
// 在代码的另一个地方给所有的对象添加了一个方法
if (typeof Object.prototype.clone === "undefined") {
    Object.prototype.clone = function () {};
}
```

在这个例子中，我们使用对象字面量定义了一个名叫man的对象。在代码中的某个地方（可以是man定义之前也可以是之后），给Object的原型增加了一个方法clone()。原型链是实时的，这意味着所有的对象都可以访问到这个新方法。要想在枚举man的时候避免枚举出clone()方法，就需要调用hasOwnProperty()来过滤来自原型的属性。如果不做过滤，clone()也会被遍历到，这是我们不希望看到的：

```
/ 1.for-in循环
for (var i in man) {
    if (man.hasOwnProperty(i)) { // filter
        console.log(i, ":", man[i]);
    }
}
/*
控制台中的结果
hands : 2
legs : 2
heads : 1
*/

// 2.反模式:
// 不使用hasOwnProperty()过滤的for-in循环
for (var i in man) {
    console.log(i, ":", man[i]);
}
/*
控制台中的结果
hands : 2
legs : 2
heads : 1
clone: function()
*/
```

另外一种调用hasOwnProperty()的方法是通过Object.prototype来调用，像这样：

```
for (var i in man) {
    if (Object.prototype.hasOwnProperty.call(man, i)) { // 过滤
        console.log(i, ":", man[i]);
    }
}
```

**hasOwnProperty：**是用来判断一个对象是否有你给出名称的属性或对象。不过需要注意的是，此方法无法检查该对象的原型链中是否具有该属性，该属性必须是对象本身的一个成员。

这种做法的好处是，在man对象中重新定义了hasOwnProperty方法的情况下，可以避免调用时的命名冲突。为了避免查找属性时从Object对象一路找到原型的冗长过程，你可以定义一个变量来“缓存”住它：

```
var i,
    hasOwn = Object.prototype.hasOwnProperty;
for (i in man) {
    if (hasOwn.call(man, i)) { // 过滤
        console.log(i, ":", man[i]);
    }
}
```
>严格说来，省略hasOwnProperty()并不是一个错误。根据具体的任务以及你对代码的自信程度，你可以省略掉它以提高一些程序执行效率。但当你对当前要遍历的对象不确定的时候，添加hasOwnProperty()则更加保险些。

这里介绍一种格式上的变种（这种写法无法通过JSLint检查），这种写法在for循环所在的行加入了if判断条件，他的好处是能让循环语句读起来更完整和通顺（“如果元素包含属性X，则对X做点什么”）：

```
// 警告：无法通过JSLint检查
var i,
    hasOwn = Object.prototype.hasOwnProperty;
for (i in man) if (hasOwn.call(man, i)) { // 过滤
    console.log(i, ":", man[i]);
}
```

**（不）扩充内置原型**

我们可以扩充构造函数的prototype属性来为构造函数增加功能，这个特性非常强大，但有时会强大到超过我们的掌控。

给内置构造函数如Object()、Array()、Function()扩充原型看起来非常诱人，但这种做法会严重降低代码的可维护性，因为它会让你的代码变得难以预测。对于那些基于你的代码来做开发的开发者来说，他们更希望使用原生的JavaScript方法来保持代码的一致性，而不愿意使用你所添加的方法。

另外，如果将属性添加至原型中，很可能导致原型上的属性在那些不使用hasOwnProperty()做过滤的循环中被遍历出来，从而造成混乱。

因此，不扩充内置对象的原型是最好的，你也可以自己定义一个规则，仅当下列条件满足时才考虑扩充内置对象的原型：

- 未来的ECMAScript版本或者JavaScirpt会将你将要实现的方法添加为内置方法。比如，你可以实现ECMAScript5定义的一些方法，直到浏览器升级至支持ES5。这样，你只是提前定义了这些方法。

- 当某个属性或者方法是你在其它地方实现过的，或者是某个JavaScript引擎或浏览器的一部分，而你检查时又发现它不存在时。

- 在有充分的文档说明，并且和团队其他成员做了沟通的时候。







































