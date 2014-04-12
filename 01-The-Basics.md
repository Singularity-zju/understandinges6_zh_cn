#基础知识

ECMAScript 6在ECMAScript 5标准的基础上做出了很多变化。其中有些变化是巨大的，比如添加了新类型或语法，而其他的一些则相当小，在原有的语言之上做出了不断的改进。本章主要介绍这些增量改进，这些改进可能并不会得到很多关注，但所提供的一些重要的功能可能会使某些类型的问题变得更容易解决一些。

##更好地支持Unicode

在ECMAScript 6之前，JavaScript的字符串实现完全基于16位字符编码的想法。所有的字符串属性和方法，如`length`和`charAt()`，都是根据“每16位序列表示一个字符的”的想法来实现的。ECMAScript 5标准允许JavaScript引擎来决定使用哪两种编码，是UCS-2还是UTF-16（两个编码都采用16位为*编码单位*，使得所有可以被观察到的操作都有相同的结果）。虽然在曾经某个时间内，世界上所有的字符都能够使用16位来表示，但是现在，这个情况已经改变了。

继续使用16位的话将不可能达到Unicode的“为世界上每一个字符创建一个唯一的标示符”的目标。这些被称为*码点*的全局唯一标识符仅仅是从0开始的数字（你可能会认为这些是字符编码，但其实是有细微的差别的）。一种字符编码负责将一个码点转换成内部一致的编码单元。而UCS-2对于码点到编码单元的映射是一对一映射，UTF-16则有更多可能性。

在UTF-16里，最开始的2^16个码点表现为一个16位的编码单元。这就是所谓的*基本多文种平面*（BMP）。超出该范围内的一切被认为是处在一个*补充平面*中，在这里，码点不再能够用仅仅16位来表示。UTF-16通过引进*代理编码对*解决了这个问题，在这里，一个单一的码点是由两个16位编码单元来表示的。这意味着在一个字符串中的任何单个字符可以是一个编码单元（支持BMP，共16位）或者两个（辅助平面的字符，总共有32位）。

ECMAScript 5将所有的操作都保持在16位编码单元中，这意味着你也许会得到意想不到的结果，如果你操作的字符串包含代理编码对的话。例如：

```js
var text = "𠮷";

console.log(text.length); // 2
console.log(/^.$./.test(text));  //false
console.log(text.charAt(0)) // '"'
console.log(text.charAt(1)) // '"'
console.log(text.charCodeAt(0)) // 55362
console.log(text.charCodeAt(1)) // 57271
```

在这个例子中，一个单一的Unicode字符是使用代理编码对来表示的，也就是说，JavaScript的字符串操作把字符串作为两个16位字符来操作。这意味着 字符串的`length` 为2，正则表达式试图匹配单个字符失败，并且`charAt()`无法返回一个有效的字符串。`charCodeAt()`方法对每个编码单元返回对应的16位数字编码，这已经是你可以在ECMAScript 5中得到的最接近真实值的东西了。

ECMAScript 6强制使用UTF-16字符串的编码。字符编码的标准化意味着该语言现在可以支持那些被设计用来对付代理编码对的方法了。

### codePointAt() 方法

完全支持UTF-16的第一个例子是`codePointAt()`方法，它可以用于获取映射到一个给定字符的Unicode码点。这个方法接受一个字符位置（而不是编码单元的位置），并返回一个整数值：

```js
var text = "𠮷a";

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 97
```

返回的值是Unicode码点的值。对于BMP字符来说，这个结果将和使用 `charCodeAt()` 的结果是一致的，因此 `"a"`将会返回97。这种方法是确定一个给定的字符是由一个还是两个码点来表示的最简单的方法：

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

16位字符表示的上界为十六进制的'FFFF'，所以任何在此数字之上的码点都必须由两个编码单元来表示。

### String.fromCodePoint()

当ECMAScript中提供了一种方法来做一些事情，一般它也会提供一种方法做相反的事。您可以使用`codePointAt()`来在一个字符串中获取字符的码点，而`String.fromCodePoint()' 则会通过一个指定码点来产生一个单字符字符串。例如：

```js
console.log(String.fromCodePoint(134071));  // "𠮷"
```

你可以把`String.fromCodePoint()`想成`String.fromCharCode()`的一个更完整版本。对于在BMP中的所有字符，每种方法都有相同的结果。唯一的区别是对于在该范围之外的字符。

### 编码 Non-BMP 字符

ECMAScript 5允许字符串包含由*转义序列*来表示的16位Unicode字符。转义序列是`\U`后面跟着四个十六进制值。例如，转义序列'\u0061'表示字母`"a"`:

```js
console.log("\u0061");      // "a"
```

如果您尝试使用超过'FFFF'，即BMP的上界，的转义序列，那么你就可以得到一些令人吃惊的结果：

```js
console.log("\u20BB7");     // "₻7"
```

由于Unicode转义序列被定义为总是严格的四个十六进制字符，所以ECMAScript将`\u20BB7`作为两个字符看待：`\u20BB`和'"7"'。第一个字符是不可打印的，第二个是数字7。 

ECMAScript 6通过引入一个扩展的Unicode转义序列来解决这个问题，在这种序列中十六进制数字被包含在大括号中。这允许最多8个十六进制字符来指定单个字符： 

```js
console.log("\u{20BB7}");     // "𠮷"   
```

使用扩展转义序列，正确的字符会被包含在字符串中。

>请确保您只在一个支持ECMAScript 6的环境中使用这个新的转义序列。在所有其他环境中，这样做会导致一个语法错误。你可能需要检查环境是否支持扩展转义序列功能，可以使用以下函数来检测：
```js
    function supportsExtendedEscape() {
     try {
         "\u{00FF1}";
         return true;
     } catch (ex) {
         return false;
     }
    }
```
>

### normalize() 方法

Unicode的另一个有趣的方面是，不同的字符在进行排序或者或其他基于比较的操作时，有可能被视为相同的。有两种方法来定义这些关系。首先，*规范等价*表示两个码点的序列在各方面都被认为是通用的。这甚至意味着两个字符的组合，可以标准地等同于一个字符。第二关系是*兼容性*，这意味两个码点的序列具有不同的外观，但在某些情况下可以互相通用。

需要了解的一件重要的事情是，由于这些关系，可能会存在两个字符串，它们从根本上来说表示的是同一文本，却含有不同的码点序列。例如，
字符 "Ã¦" 和字符 "ae" 也许能够互相通用，虽然它们是不同的码点。因此，这两个字符串在JavaScript中就是不相等的，除非它们被以某种方式进行归一化。

ECMAScript 6中通过一个新的`normalize()`方法来支持以下四种对字符串的Unicode归一化形式。该方法选择性地接受一个参数，`"NFC"`（默认值），`"NFD"`，`"NFKC"` 或 `"NFKD"`。解释这四种形式之间的差异超出了本书的范围。请记住，为了使用，您必须正常化被比较，以相同的形式两个字符串。例如：

```js
var normalized = values.map(text => text.normalize());
normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

在这段代码中，在一个`values` 数组中的字符串被转换成一个标准化的形式以使该数组可以被适当地排序。你可以通过在原始数组上调用`normalize()`作为比较器的一部分来完成排序：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

再次，要记住最重要的一点是，这两个值都必须以相同的方式进行归一化。这些例子都使用默认值NFC，但你可以很容易地指定它们其中的另一个：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

如果你在此前从来没有担心过Unicode的归一化，那么这个方法可能对你来说用处不大。然而，知道它是可用的将会帮助你在一个国际化的应用程序中工作得更好。

### 正则表达式的 u 标志

许多常见的字符串操作是通过使用正则表达式来完成的。然而，正如前面所提到的，正则表达式的工作也建立在16位的编码单元，每个单元代表一个字符的基础上。这就是为什么在前面的示例中，单字符匹配没有像我们所预想的一样工作。为了解决这个问题，ECMAScript 6定义了正则表达式中的一个新的标志 `u` 来代表 `Unicode`。

当一个正则表达式设置了标志`u`时，它会把工作模式从编码单元切换到字符。这意味着正则表达式再也不会对字符串中的代理编码感到困惑，它将像预期一样正常地工作。例如：

```js
var text = "ð ®·";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

添加`u`标志允许正则表达式按照字符来进行正常的字符串匹配。不幸的是，ECMAScript 6还没有一种方法来确定一个字符串中含有多少编码点的方式，但是幸运的是，正则表达式可以做到：

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("ð ®·bc"));   // 3
```

在这个例子中，正则表达式匹配空格和非空白字符，这适用于通用的所有Unicode字符串。在至少有一个结果匹配时，`result`会包含一个匹配结果的数组，因此数组的长度就是这个字符串中的编码点的数量。

W>虽然这种方法有效，但它不是很快，尤其是当你应用到长字符串时，因此请尽可能减少编码点的计数操作。希望未来的ECMAScript 7将带来一种更高性能的计算方法。

## 更多的String方法

JavaScript的字符串在类似的功能上一直落后于其它语言。比如，直到ECMAScript 5中字符串才终于获得了`trim()`方法，而ECMAScript 6将会继续扩展字符串的新功能。

## contains(), startsWith(), endsWith()

自从有了JavaScript以来，开发人员一直使用`indexOf()`来确定某个字符串是否被包含在另一个字符串中。在ECMAScript 6中，新增加了三个新的方法，用以判断一个字符串是否包含其它的字符串。

* `contains()` - 如果给定的文本在字符串中任意地方被发现，则会返回true。 否则会返回false。
* `startsWith()` - 如果给定的文本在字符串的开始处被发现，则返回true。否则返回false。
* `endsWith()` - 如果给定的文本在字符串的结尾处被发现，则返回true。否则返回false。

所有这些方法会接受两个参数：需要搜索的文本，（可选的）从字符串中开始搜索的位置。如果省略了第二个参数，`contains()`和`startsWith()`将从字符串的开头开始搜索，而`endsWith()`则从结尾搜索。实际上，第二个参数会减少被搜索的字符串部分。下面是一些例子：

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.contains("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.contains("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.contains("o", 8));          // false
```

这三种方法使我们能够更容易判断字符子串，而无需担心它们精确位置的识别。

I> 所有的这些方法都会返回一个布尔值，如果你需要获得一个字符串在另一个字符串中的位置，请使用`indexOf()`和`lastIndexOf()`方法。

### repeat()

ECMAScript 6还为字符串增加了一个`repeat()`方法，这个方法接受一个参数，为重复该字符串的次数，并返回原始字符串重复了指定次数后的一个新的字符串。例如：

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

无论怎样，这确实是一个非常方便的函数，尤其是在文本处理中。看其中一个例子，这里我们需要为代码格式化工具创建给定的缩进级数：

```js
// indent using a specified number of spaces
var indent = " ".repeat(size),
    indentLevel = 0;

// whenever you increase the indent
var newIndent = indent.repeat(++indentLevel);
```

## Object.is()

当你想比较两个值时，你可能习惯于使用或者等号运算符(`==`)或恒等于操作符(`===`) 。许多人喜欢使用后者，以避免在比较期间进行强制类型转换。然而，即使是恒等于操作符也不是完全准确。例如，值+0和-0在`===`操作符下被认为是相等的。即使它们在不同的JavaScript引擎下会表现得不一样。同样的，`NaN === NaN` 会返回 `false`，这迫使我们使用`isNaN()`来正确地检测`NaN`。

ECMAScript 6引入了`Object.is()` 来弥补恒等于操作符所遗留的一些诡异之处。这个方法接受两个参数，并在两个值是相等的时返回`true`。两个值只有在它们拥有同样的值并且同样的类型时，才会被认为是相等的。在在许多情况下，`Object.is()` 的工作方式与`===`相同。唯一的区别是+0和-0会被认为是不等价的，而且`NaN`会被认为等同于`NaN`。下面是一些例子：

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false
```

在大多数情况下，您可能仍然想使用`==`或`===`来用于比较，因为`Object.is()`所涵盖的特殊情况可能并不会对你造成影响。

## Block bindings

传统上，JavaScript的棘手的部分之一，一直被认为是`var`变量声明的工作方式。在大多数基于C的语言中，变量在哪里声明就在那里被创建。然而，在JavaScript中，却并非如此。使用`var`声明的变量会被*悬挂*到函数（或全局空间）的顶部，而不管实际上声明是在哪里产生的。例如：

```js
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

如果你不熟悉JavaScript，您可能会认为该变量`value`只会在`condition`值为true的时候被声明和定义。而事实上，变量`value`无论如何都是会被声明的。JavaScript引擎中，代码会被转换成这样：

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

`value`的声明被移到了顶部（悬挂），而初始化却留在了原有的地方。这意味着变量`value`的值其实在`else`子句中还是能够被访问的，它只是具有`undefined`的值，因为它此时并没有被初始化。

这种特性往往需要新的JavaScript开发者花费一些时间来习惯变量悬挂，而且，这种独特的行为有可能最终会导致一些错误。 出于这个原因，ECMAScript 6中引入了块级作用域选项，使得对于变量生命周期的控制能够更加有力。

### Let declarations

`let`声明语句的格式与`var`是完全一样的。基本上来说，你可以用`let`代替`var`来声明一个变量，但保留其范围到当前的代码块。例如：

```js
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
```

这个功能现在的行为更接近于其他基于C的语言。变量`value`是使用`let`而不是`var`来声明的。这意味着该声明不会悬挂在顶端，而且变量`value`在一旦执行流程超出了`if`语句块时就会被销毁。如果`condition`总是计算出false，那么`value`酱永远不会被声明或初始化。

也许，开发人员最想要变量块级作用域的其中一处地方是`for`循环。这样的代码我们常常可以看到：

```js
for (var i=0; i < items.length; i++) {
    process(items[i]);
}

// i is still accessible here
```

在其他那些默认含有块级作用域的语言中，像这样的代码会按预期工作。然而在JavaScript中，因为`var`的声明悬挂。变量`i`在循环完成后仍然可以被访问。使用`let`则可以让你得到预期的行为：

```js
for (let i=0; i < items.length; i++) {
    process(items[i]);
}

// i is not accessible here
```

在这个例子中，变量`i`只存在于`for`循环之内。一旦循环完成后，该变量就会被摧毁，其他地方无法再次访问它。

不同于`var`，`let`没有悬挂特性。使用了`let`声明的变量在`let`语句之前不能被访问。任何试图这么做的行为都将会引发一个格式错误：

```js
if (condition) {
    console.log(value);     // error!
    let value = "blue";
}
```

在这段代码中，变量`value`使用了`let`来定义和初始化，但该语句永远不会执行，因为上一行会抛出一个错误。

如果标识符已在块中定义，那么在'let'声明中使用该标识符将会导致抛出一个错误。例如：

```js
var count = 30;

// Throws an error
let count = 40;
```

在这个例子中，`count`被声明了两次，一次用`var`，一次用`let`。因为`let`不会重新定义已经存在于同一范围内的标识符，所以声明会抛出一个错误。然而，如果一个`let`在作用域A中声明了一个新的变量，同时这个变量的变量名在作用域B中已经存在，并且作用域B包含了作用域A，则不会抛出错误，如：

```js
var count = 30;

// Does not throw an error
if (condition) {

    let count = 40;

    // more code
}
```

在这里，`let`声明将不会抛出一个错误，因为它在`if`语句的作用域中创建了一个新的`count`变量。这个新的变量会屏蔽全局的`count`，导致我们无法从`if`语句块中访问到它。

提出`let`的目的在长远看来是取代`var`，因为前者行为与其他语言中的变量声明能够保持一致。如果你正在编写一段将只在ECMAScript 6或更高的环境中执行的JavaScript，你可能会想试试使用`let`并且只在那些为需要向后兼容的其它脚本中使用`var`。

注：由于所有`let`声明不会被悬挂在封闭块的顶部，你可能需要自己把`let`声明放在第一步。

### 常量声明

另一种定义变量的新方式是使用`const`声明语法。使用`const`来定义的变量被认为是*常量*，所以一旦设定，它的值不能被改变。出于这个原因，每个`const`常量必须被初始化。例如：

```js
// Valid constant
const MAX_ITEMS = 30;

// Syntax error: missing initialization
const NAME;
```

常量也是块级的声明，类似于`let`。也就是说，一旦执行流跑出了它们被声明的代码块，常量就会被销毁。并且常量声明也会被提升到块的顶部。例如：

```js
if (condition) {
    const MAX_ITEMS = 5;

    // more code
}

// MAX_ITEMS isn't accessible here
```

在这段代码中，常量的`MAX_ITEMS`是在`if`语句的代码块中声明的。一旦该语句执行完毕后，`MAX_ITEMS`就会被销毁，所以不能从块的外部来访问它。

并且，类似于`let`，如果一个`const`声明的常量命名与在同一个作用域中的其它已经定义的变量/常量相同的话，就会抛出异常。无论该变量是使用`var`（全局或函数范围内）还是使用`let`（在块作用域）中声明。例如：

```js
var message = "Hello!";
let age = 25;

// Each of these would cause an error given the previous declarations
const message = "Goodbye!";
const age = 30;
```

注：一些浏览器实现了 ECMAScript 6预览版本的`const`语句。它们实现的范围从单纯的`var`的代名词（即允许被覆盖的值），到确实符合定义，但只能在全局或函数范围内有效都有。因此，在生成系统中，你应该谨慎使用`const`，它可能无法给你提供你所期望的功能。



## 数字和数学

TODO：介绍

### 八进制和二进制字面量


ECMAScript 5 试图通过在`paseInt()`和strict mode这两处移除之前引入的八进制整数字面量符号来简化一些常见的数值错误。在ECMAScript 3和更早的版本中，八进制数使用一个`0`后跟任意数量的数字来表示。例如:

```js
// ECMAScript 3
var number = 071;       // 57 in decimal

var value1 = parseInt("71");    // 71
var value2 = parseInt("071");   // 57
```

许多开发者对于这一版本的八进制字面量数字表示感到疑惑，也因为对于前导零在不同地方所产生的不同影响的误解而犯下了许多错误。最令人震惊的是`parseInt()`，在其中前导零意味着该值将被视为八进制而不是十进制。这也导致了Douglas Crockford 的第一个JSLint的规则之一：始终使用`parseInt()`函数的第二个参数来指定字符串应该怎样被解释。

ECMAScript 5减少了对八进制数字的使用。首先，`parseInt()`方法已经被更改，因此它在没有第二个参数时会忽略第一个参数的前导零。这意味着数字不再会被意外地视为八进制。第二个变化是去除了在严格模式下的八进制字面量符号。在严格模式下尝试使用一个八进制字面量会导致一个语法错误。

```js
// ECMAScript 5
var number = 071;       // 57 in decimal

var value1 = parseInt("71");        // 71
var value2 = parseInt("071");       // 71
var value3 = parseInt("071", 8);    // 57

function getValue() {
    "use strict";
    return 071;     // syntax error
}
```

通过引入这两个变化，ECMAScript 5尝试消除了很多与八进制字面量相关的混乱和错误。

ECMAScript 6又更进了一步，它重新采用八进制字面量符号，以及一个二进制字面量符号。这两个符号通过在值的前面加上`0x`或`0X`来代表十六进制字面量符号。新的八进制字面量格式以`0o`或`0O`而新的二进制字面量格式开始于`0b`或`0B`。每个字面量类型后面必须跟一个或多个数字，0-7为八进制，0-1二进制。如下例：

```js
// ECMAScript 6
var value1 = 0o71;      // 57 in decimal
var value2 = 0b101;     // 5 in decimal
```

添加这两个字面量类型将允许JavaScript开发人员快速，轻松地引入包括二进制，八进制，十进制和十六进制格式在内的数字值，这对于某些类型的数学运算来说是非常重要的。

`parseInt()`方法不会处理看起来像八进制或二进制字面量的字符串：

```js
console.log(parseInt("0o71"));      // 0
console.log(parseInt("0b101"));     // 0
```

然而，`Number()` 函数则会正确地转换八进制或二进制字面量的字符串：

```js
console.log(Number("0o71"));      // 57
console.log(Number("0b101"));     // 5
```

当使用八进制或二进制字面量字符串时，一定要了解您的使用情况，并使用最适当的方法将其转换为数字值。

## 更多

本章可能的内容：

*`Number`中引入的新方法

|方法|说明|
| -------- | ------------- |
|`Math.imul(x, y)`|两个数字的快速32位乘法。|
|`Math.log10(x)`| TODO |
|`Math.log2(x)`| TODO |
|`Math.log1p(x)`| TODO |
|`Math.expm1(x)`| TODO |
|`Math.cosh(x)`| TODO |
|`Math.sinh(x)`| TODO |
|`Math.tanh(x)`| TODO |
|`Math.acosh(x)`| TODO |
|`Math.asinh(x)`| TODO |
|`Math.atanh(x)`| TODO |
|`Math.hypot(...values)`| TODO |
|`Math.trunc(x)`|删除小数位数的浮点数并返回一个整数。|
|`Math.sign(x)`|如果`x`是+0或-0，返回-1，如果`x`是正数，则返回1|
|`Math.fround(x)`| TODO |


##小结

待补充

