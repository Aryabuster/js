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

如果你遇到这三种情况之一，你可以给内置原型添加自定义方法，写法如下：

```
if (typeof Object.protoype.myMethod !== "function") {
    Object.protoype.myMethod = function () {
        // 实现…
    };
}
```

### switch模式

```
var inspect_me = 0,
    result = '';
switch (inspect_me) {
case 0:
    result = "zero";
    break;
case 1:
    result = "one";
    break;
default:
    result = "unknown";
}
```

这个简单的例子所遵循的风格约定如下：

- 每个case和switch对齐（这里不考虑花括号相关的缩进规则）。
- 每个case中的代码整齐缩进。
- 每个case都以break作为结束。
- 避免连续执行多个case语句块（省略break时），如果你坚持认为连续执行多个case语句块是最好的方法，请务必补充文档说明，对于其他人来说，会觉得这种情况是错误的写法。
- 以default结束整个switch，以确保即便是在找不到匹配项时也有合理的结果。

### 避免隐式类型转换

在JavaScript对变量进行比较时会有一些隐式的数据类型转换。比如诸如false == 0或"" == 0之类的比较都返回true。

为了避免隐式类型转换对程序造成干扰，推荐使用 === 和 !== 运算符，它们除了比较值还会比较类型：

```
var zero = 0;
if (zero === false) {
    // 不会执行，因为zero是0，不是false
}
// 反模式
if (zero == false) {
    // 代码块会执行…
}
```
有一种观点认为当==够用的时候就不必使用===。比如，当你知道typeof的返回值是一个字符串，就不必使用全等运算符。但JSLint却要求使用全等运算符，这无疑会提高代码风格的一致性，并减少了阅读代码时的思考量（“这里使用==是故意的还是无意的？”）。

### 避免使用eval()

当你想使用eval()的时候，不要忘了那句话“eval() is evil”（eval()是魔鬼）。这个函数的参数是一个字符串，它会将传入的字符串作为JavaScript代码执行。如果用来解决问题的代码是事先知道的（在运行之前），则没有理由使用eval()。如果需要在运行时动态生成并执行代码，那一般都会有更好的方式达到同样的目的，而非一定要使用eval()。例如，访问动态属性时可以使用方括号：

```
// 反模式
var property = "name";
alert(eval("obj." + property));
// 更好的方式
var property = "name";
alert(obj[property]);
```

eval()还有安全隐患，因为你有可能会运行一些被干扰过的代码（比如一段来自于网络的代码）。这是一种在处理Ajax请求所返回的JSON数据时比较常见的反模式。这种情况下最好使用浏览器的内置方法来解析JSON数据，以确保代码的安全性和数据的合法性。如果浏览器不支持JSON.parse()，你可以使用JSON.org所提供的库。

值得一提的是，多数情况下，给setInterval()、setTimeout()和Function()构造函数传入字符串的情形和eval()类似，这种用法也是应当避免的，因为这些情形中JavaScript最终还是会执行传入的字符串参数：

```
// 反模式
setTimeout("myFunc()", 1000);
setTimeout("myFunc(1, 2, 3)", 1000);
// 更好的方式
setTimeout(myFunc, 1000);
setTimeout(function () {
    myFunc(1, 2, 3);
}, 1000);
```

new Function()的用法和eval()非常类似，应当特别注意。这种构造函数的方式很强大，但经常会被误用。如果你不得不使用eval()，你可以尝试用new Function()来代替。这有一个潜在的好处，在new Function()中运行的代码会在一个局部函数作用域内执行，因此源码中所有用var定义的变量不会自动变成全局变量。还有一种方法可以避免eval()中定义的变量被转换为全局变量，即是将eval()包装在一个即时函数内（详细内容请参见第四章）。

看一下这个例子，这里只有un成为全局变量污染了全局命名空间：

```
console.log(typeof un);// "undefined"
console.log(typeof deux); // "undefined"
console.log(typeof trois); // "undefined"

var jsstring = "var un = 1; console.log(un);";
eval(jsstring); // 打印出 "1"

jsstring = "var deux = 2; console.log(deux);";
new Function(jsstring)(); // 打印出 "2"

jsstring = "var trois = 3; console.log(trois);";
(function () {
    eval(jsstring);
}()); // 打印出 "3"

console.log(typeof un); // "number"
console.log(typeof deux); // "undefined"
console.log(typeof trois); // "undefined"
```

eval()和Function()构造函数还有一个区别，就是eval()可以修改作用域链，而Function更像是一个沙箱。不管在什么地方执行Function()，它都只能看到全局作用域。因此它不会太严重的污染局部变量。在下面的示例代码中，eval()可以访问并修改其作用域之外的变量，而Function()则不能（注意，使用Function()和new Function()是完全一样的）。

```
(function () {
    var local = 1;
    eval("local = 3; console.log(local)"); // 打印出 3
    console.log(local); // 打印出 3
}());

(function () {
    var local = 1;
    Function("console.log(typeof local);")(); // 打印出 undefined
}());
```
### 使用parseInt()进行数字转换

你可以使用parseInt()将字符串转换为数字。函数的第二个参数是进制参数，这个参数应该被指定，但却通常被省略。当字符串以0为前缀时转换就会出问题，例如，在表单中输入日期的一个字段。ECMAScript3中以0为前缀的字符串会被当作八进制数处理，这一点在ES5中已经有了改变。为了避免转换类型不一致而导致的意外结果，应当总是指定第二个参数：

```
var month = "06",
    year = "09";
month = parseInt(month, 10);
year = parseInt(year, 10);
```

在这个例子中，如果省略掉parseInt的第二个参数，比如parseInt(year)，返回的值是0，因为“09”被认为是八进制数（等价于parseInt(year,8)），但09是非法的八进制数。

字符串转换为数字还有两种方法：

```
+"08" // 结果为8
Number("08") // 结果为8
```

这两种方法要比parseInt()更快一些，因为顾名思义parseInt()是一种“解析”而不是简单的“转换”。但当你期望将“08 hello”这类字符串转换为数字，则必须使用parseInt()，其他方法都会返回NaN。

### 代码规范

确立并遵守代码规范非常重要，这会让你的代码风格一致、可预测，并且可读性更强。团队新成员通过学习代码规范可以很快进入开发状态，并写出让团队其他成员易于理解的代码。

在开源社区和邮件组中关于编代风格的争论一直不断。（比如关于代码缩进，用tab还是空格？）因此，如果你打算在团队内推行某种编码规范时，要做好应对各种反对意见的心理准备，而且要吸取各种意见。确定并遵守代码规范非常重要，任何一种规范都可以，这甚至比代码规范中的具体约定是怎么样的还要重要。

### 缩进

代码如果没有缩进就几乎不能读了，而不一致的缩进会使情况更加糟糕，因为它看上去像是遵守了规范，但真正读起来却没那么顺利。因此规范地使用缩进非常重要。

有些开发者喜欢使用tab缩进，因为每个人都可以根据自己的喜好来调整tab缩进的空格数，有些人则喜欢使用空格缩进，通常是四个空格。这都无所谓，只要团队每个人都遵守同一个规范即可，本书中所有的示例代码都采用四个空格的缩进写法，这也是JSLint所推荐的。（译注：电子版中看到的是用tab缩进，本译文也保留使用tab缩进。）

那么到底什么时候应该缩进呢？规则很简单，花括号里的内容应当缩进，包括函数体、循环（do、while、for和for-in）体、if语句、switch语句和对象字面量里的属性。下面的代码展示了如何正确地使用缩进：

```
function outer(a, b) {
    var c = 1,
        d = 2,
        inner;
    if (a > b) {
        inner = function () {
            return {
                r: c - d
            };
        };
    } else {
        inner = function () {
            return {
                r: c + d
            };
        };
    }
    return inner;
}
```

### 花括号

在特定的语句中应当总是使用花括号，即便是在可省略花括号的情况下也应当如此。从技术角度讲，如果if或for中只有一个语句，花括号是可以省略的，但最好还是不要省略，这会让你的代码更加工整一致而且易于修改。

假设有这样一段代码，for循环中只有一条语句，你可以省略掉这里的花括号，而且不会有语法错误：

```
// 不好的方式
for (var i = 0; i < 10; i += 1)
    alert(i);
```

但如果过了一段时间，你给这个循环添加了另一行代码会怎样？

```
// 不好的方式
for (var i = 0; i < 10; i += 1)
    alert(i);
    alert(i + " is " + (i % 2 ? "odd" : "even"));
```
第二个alert实际上在循环体之外，但这里的缩进会让你迷惑。从长远考虑最好还是写上花括号，即便是在只有一个语句的语句块中也应如此：

```
// 更好的方式
for (var i = 0; i < 10; i += 1) {
    alert(i);
}
```

同理，if条件句也应当如此：

```
// 不好的方式
if (true)
    alert(1);
else
    alert(2);

// 更好的试
if (true) {
    alert(1);
} else {
    alert(2);
}
```

### 左花括号的位置

开发人员对于左大括号的位置有着不同的偏好，在同一行呢还是在下一行？

```
if (true) {
    alert("It's TRUE!");
}
```

或者：

```
if (true)
{
    alert("It's TRUE!");
}
```

在这个例子中，这个问题只是个人偏好问题。但有时候花括号位置的不同会影响程序的执行，因为JavaScript会“自动插入分号”。JavaScript对行尾是否有分号并没有要求，它会自动将分号补全。因此，当函数的return语句返回了一个对象字面量，而对象的左花括号和return又不在同一行时，程序的执行就和预期的不同了：

```
// 警告：返回值和预期的不同
function func() {
    return
    {
        name: "Batman"
    };
}
```

可以看出程序作者的意图是返回一个包含了name属性的对象，但实际情况不是这样。因为return后会填补一个分号，函数的返回值就是undefined。这段代码等价于：

```
// 警告：返回值和预期的不同
function func() {
    return undefined;
    // 下面的代码不会运行…
    {
        name: "Batman"
    };
}
```

总结一下好的写法，在特写的语句中总是使用花括号，并且总是将左花括号与上一条语句放在同一行：

```
function func() {
    return {
        name: "Batman"
    };
}
```

> 关于分号也值得注意：和花括号一样，应当总是使用分号，尽管在JavaScript解析代码时会补全行末省略的分号，但严格遵守这条规则，可以让代码更加严谨，同时可以避免前面例子中所出现的歧义。

### 空格

空格的使用同样有助于改善代码的可读性和一致性。在写英文句子的时候，在逗号和句号后面会使用间隔，在JavaScript中，你可以按照同样的逻辑在表达式（相当于逗号）和语句结束（相对于完成了某个“想法”的表达）后面添加间隔。

适合使用空格的地方包括：

- for循环中的分号之后，比如for (var i = 0; i < 10; i += 1) {...}
- for循环中初始化多个变量，比如for (var i = 0, max = 10; i < max; i += 1) {...}
- 用于分隔数组元素的逗号之后，比如var a = [1, 2, 3];
- 对象属性后的逗号以及名值对之间的冒号之后，比如var o = {a: 1, b: 2};
- 函数参数中，比如myFunc(a, b, c)
- 函数声明的花括号之前，比如function myFunc() {}
- 匿名函数表达式function之后，比如var myFunc = function () {};

另外，我们推荐在运算符和操作数之间也添加空格。也就是说在\+、\-、\*、=、\<、\>、\<=、\>=、===、!==、\&\&、|、\+=符号前后都添加空格。

```
// 适当且一致的空格给代码留了“呼吸空间”，使代码更易读
var d = 0,
    a = b + 1;
if (a && b && c) {
    d = a % c;
    a += d;
}

// 反模式，缺少或者不正确的空格使得代码不易读
var d= 0,
    a =b+1;
if (a&& b&&c) {
    d=a %c;
    a+= d;
}
```

最后，还应当注意，最好在花括号旁边添加空格：

- 在函数、if-else语句、循环、对象字面量的左花括号之前补充空格
- 在右花括号和else或者while之间补充空格

>垂直空白的使用经常被我们忽略，你可以使用空行来将代码单元分隔开，就像文学作品中使用段落进行分隔一样。

### 命名规范

另外一种可以提升你代码的可预测性和可维护性的方法是采用命名规范。也就是说变量和函数的命名都遵守同样的习惯。

下面是一些建议的命名规范，你可以原样采用，也可以根据自己的喜好作调整。同样，遵循规范要比规范本身是什么样更加重要。

### 构造函数命名中的大小写

JavaScript中没有类，但有构造函数，可以通过new来调用构造函数：

```
var adam = new Person();
```

由于构造函数毕竟还是函数，如果只通过函数名就可分辨出它是构造函数还是普通函数是非常有用的。

首字母大写可以提示你这是一个构造函数，而首字母小写的函数一般只认为它是普通的函数，不应该通过new来调用它：

```
function MyConstructor() {...}
function myFunction() {...}
```

下一章将介绍一些强制将普通函数用作构造函数的编程模式，但遵守我们所提到的命名规范会更好的帮助程序员阅读源码。

### 单词分隔

当你的变量名或函数名中含有多个单词时，单词之间的分隔也应当遵循统一的规范。最常见的是“驼峰式”（camel case）命名，单词都是小写，每个单词的首字母是大写。

对于构造函数，可以使用“大驼峰式”（upper camel case）命名，比如MyConstructor()，对于函数和方法，可以采用“小驼峰式”（lower camel case）命名，比如myFunction()、calculateArea()和getFirstName()。

那么对于那些不是函数的变量应当如何命名呢？变量名通常采用小驼峰式命名，还有一个不错的做法是，变量所有字母都是小写，单词之间用下划线分隔，比如，first_name、favorite_bands和old_company_name，这种方法可以帮助你区分函数和其他标识符如原始类型数据或对象。

ECMAScript的属性和方法均使用驼峰式命名，尽管包含多个单词的属性名称并不多见（正则表达式对象的lastIndex和ignoreCase属性）

### 其他命名风格

有时开发人员使用命名规范来弥补或代替语言特性的不足。

比如，JavaScript中无法定义常量（尽管有一些内置常量比如Number.MAX_VALUE），所以开发者都采用了一种命名规范，对于那些程序运行周期内不会更改的变量使用全大写字母来命名。比如：

```
// 常量，请勿修改
var PI = 3.14,
    MAX_WIDTH = 800;
```

除了使用大写字母的命名方式之外，还有另一种命名规范：全局变量全大写。这种命名方式和“减少全局变量”的约定相辅相成，并让全局变量很容易辨认。

除了常量和全局变量的命名规范，这里讨论另外一种命名规范，即私有变量的命名。尽管在JavaScript是可以实现真正的私有变量的，但开发人员更喜欢在私有成员或方法名之前加上下划线前缀，比如下面的例子：

```
var person = {
    getName: function () {
        return this._getFirst() + ' ' + this._getLast();
    },
    _getFirst: function () {
        // ...
    },
    _getLast: function () {
        // ...
    }
};
```











































