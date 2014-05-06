# 对象 （Objects）

A lot of ECMAScript 6 focused on improving the utility of objects. The focus makes sense given that nearly every value in JavaScript is represented by some type of object. Additionally, the number of objects used in an average Javascript program continues to increase, meaning that developers are writing more objects all the time. With more objects comes the necessity to use them more effectively.
ECMAScript 6的很多方面都致力于改善对象的使用。重点合理地考虑到了JavaScript中的每个值都被表现为对象。此外，平均每个JavaScript程序中对象的数量持续增长，意味着开发者每时每刻在写更多的对象。更多的对象带来了更有效率的使用它们的必要性。
ECMAScript 6 improves objects in a number of ways, from simple syntax to new ways of manipulating and interacting with objects.
ECMAScript 6 通过一些方式改善了对象， 从简洁的语法到新的操作以及与对象的交互的方式。

## 对象字面量扩展 （Object Literal Extensions）

One of the most popular patterns in JavaScript is the object literal. It's the syntax upon which JSON is built and can be seen in nearly every JavaScript file on the Internet. The reason for the popularity is clear: a succinct syntax for creating objects that otherwise would take several lines of code to accomplish. ECMAScript 6 recognized the popularity of the object literal and extends the syntax in several ways to make object literals more powerful and even more succinct.
在JavaScript中使用最多的模式之一是对象字面量。对象字面量的语法结构建立在JSON的语法之上，可以在因特网上的几乎每个JavaScript文件里都看到。对象字面量被广泛使用的原因很明确：通过简洁的语法结构来创建对象，不然还需要写几行代码来实现。ECMAScript 6 认可对象字面量的广泛使用，并用一些其它的方式扩展了语法，让对象字面量更强劲更简洁。

### 属性初始化简写 （Property Initializer Shorthand）

In ECMAScript 5 and earlier, object literals were simply collections of name-value pairs. That meant there could be some duplication when property values are being initialized. For example:
在ECMAScript 5及更早版本 , 对象字面量是简单的 名-值 对的集合。当属性值被初始化的时候，这意味着会有一些重复。例如:

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

The `createPerson()` function creates an object whose property names are the same as the function parameter names. The result is what appears to be duplication of `name` and `age` even though each represents a different aspect of the process.
'createPerson'函数创建了一个属性名跟函数参数名一样的对象。结果似乎'name'和'age'重复了，即使每个表示着不同的处理方面。
In ECMAScript 6, you can eliminate the duplication that exists around property names and local variables by using the property initializer shorthand. When the property name is going to be the same as the local variable name, you can simply include the name without a colon and value. For example, `createPerson()` can be rewritten as follows:
在ECMAScript 6中，可以通过使用属性初始化简写（property initializer shorthand）省略已存在的属性名和局部变量名的重复。当属性名和局部变量名相同的时候，可以去掉冒号和值。例如，`createPerson()` 可以被重写如下：
```js
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

When a property in an object literal only has a name and no value, the JavaScript engine looks into the surrounding scope for a variable of the same name. If found, that value is assigned to the same name on the object literal. So in this example, the object literal property `name` is assigned the value of the local variable `name`.
当一个对象字面量的属性只有名字没有值时，JavaScript引擎会在附近的域寻找有同样名称的变量。如果找到了，值会被赋给在对象字面量中相同名称的变量。所以在这个例子中，对象字面量属性'name'就是局部变量'name'的值。
The purpose of this extension is to make object literal initialization even more succinct than it already was. Assigning a property with the same name as a local variable is a very common pattern in JavaScript and so this extension is a welcome addition.
这个扩展的目的是使对象字面量比现在更加简洁。在JavaScript中，分配给属性和局部变量相同的名称是非常常见的模式。所以这个扩展很受欢迎。

### 方法初始化简写（Method Initializer Shorthand）

ECMAScript 6 also improves syntax for assigning methods to object literals. In ECMAScript 5 and earlier, you must specify a name and then the full function definition to add a method to an object. For example:
ECMAScript 6 也改善了为对象字面量分配方法的语法。在ECMAScript 5及更早，必须指定一个方法名，最后再为对象的方法加入完整的函数定义。例如：
```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

In ECMAScript 6, the syntax is made more sustained by eliminating the colon and the `function` keyword. you can then rewrite this example as follows:
在ECMAScript 6中，语法原来还可以继续省略冒号和 `function` 关键字。你可以重写这个例子如下所示：
```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

This shorthand syntax creates a method on the `person` object just as the previous example did. There is no difference aside from saving you some keystrokes.
这种简写语法像前一个例子一样在 `person` 对象中创建了一个方法。除了省下了一下按键，跟之前的没有区别。

### 可计算属性名 （Computed Property Names）

JavaScript objects have long had computed property names through the use of square brackets instead of dot notation. The square brackets allow you to specify property names using variables and string literals that may contain characters that would be a syntax error if used in an identifier. For example:
JavaScript对象早就已经可以通过方括号代替点表示法来计算属性名（computed property names）了。方括号允许使用包含字符的变量和字符串字面量指定属性名，如果在标识符中包含字符的话，这会导致一个语法错误。例如：
```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

Both of the property names in this example have a space, making it impossible to reference those names using dot notation. However, bracket notation allows any string value to be used as a property name.
这个例子中的属性名都包含空格，这让使用点表示法来引用这些属性名变得不可能。不过，方括号表示法可以像使用属性名那样使用字符串值。

In ECMAScript 5, you could use string literals as property names in object literals, such as:
在ECMAScript 5，在对象字面量中，你可以像使用属性名那样使用字符串字面量，比如：

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

If you could provide the string literal inside of the object literal property definition then you were all set. If, however, the property name was contained in a variable or had to be calculated, then there was no way to define that property using an object literal.
如果可以在函数字面量属性定义内部保存字符串字面量，那所有就都设置好了。可是，如果在一个变量中包含属性名，或者属性名需要被计算，那对象字面量中没有方法来定义那个属性。
ECMAScript 6 adds computed property names to object literal syntax by using the same square bracket notation that has been used to reference computed property names in object instances. For example:
ECMAScript 6 通过使用相同的方括号表示法为对象字面量语法引入了可计算属性名（computed property names），这种方式可以在对象实例中被用来引用可计算对象属性名（computed property names），例如：
```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

The square brackets inside of the object literal indicate that the property name is computed, so its contents are evaluated as a string. That means you can also include expressions such as:
在函数字面量中的方括号表示属性名可以被计算，于是它的内容以字符串的形式求值。这意味着可以包含表达式，比如：
```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);          // "Zakas"
```

Anything you would be inside of square brackets while using bracket notation on object instances will also work for computed property names inside of object literals.
在对象实例使用方括号语法的同时，同样适用于对象字面量中计算属性名（computed property names）。

## Symbols

ECMAScript 6 symbols began as a way to create private object members, a feature JavaScript developers have long wanted. The focus was around creating properties that were not identified by string names. Any property with a string name was easy picking to access regardless of the obscurity of the name. The initial "private names" feature aimed to create non-string property names. That way, normal techniques for detecting these private names wouldn't work.
ECMAScript 6中的symbol开始视为创建私有成员的方式，一个JavaScript开发者想要很久的功能。重点围绕在不通过字符串名创建属性。
任何以字符串命名的属性无论名称有多费解，都可以被很容易的被访问到。初始“私有变量”功能是为了创建非字符串属性名。那么，检测这些私有名称的一般方法不会工作。
The private names proposal eventually evolved into ECMAScript 6 symbols. While the implementation details remained the same (non-string values for property identifiers), TC-39 dropped the requirement that these properties be private. Instead, the properties would be categorized separately, being non-enumerable by default by still discoverable.
私有变量提案最终进化成了ECMAScript 6的Symbol。实现细节的时候仍然是相同的（属性标志符的非字符串值），TC-39停止了这些属性被私有的要求。
属性应该另分类，默认不可枚举『』。
Symbols are actually a new kind of primitive value, joining strings, numbers, booleans, `null`, and `undefined`. They are unique among JavaScript primitives in that they do not have a literal form. The ECMAScript 6 standard uses a special notation to indicate symbols, prefixing the identifier with '@@', such as '@@create'. This book uses this same convention for ease of understanding.
Symbol实际上是一种新的基本类型，加入到了strings,numbers,booleans,'null'和'undefined'当中。symbol在JavaScript基本类型中的独特在于它们没有字面量格式。ECMAScript 6标准使用了特殊的表示法来表示symbol，用‘@@’作为标识符的前缀，比如‘@@create’。为便于理解，本书使用相同的规范。

W> Despite the notation, symbols do not exactly map to strings beginning with "@@". Don't try to use string values where symbols are required.
标志符不精确对应以"@@"为开头的字符串。当标志符(symbol)被需要时，不要试图使用字符串类型的值。
### Creating Symbols

You can create a symbol by using the `Symbol` function, such as:
你可以使用'Symbol'函数创建一个标志符（symbol）,比如：
```js
var firstName = Symbol();
var person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

In this example, the symbol `firstName` is created and used to assign a new property on `person`. That symbol must be used each time you want to access that same property. It's a good idea to name the symbol variable appropriately so you can easily tell what the symbol represents.
在这个例子中，symbol'firstName'被创建并作为新属性分配给了'person'。symbol必须在你想访问相同的属性时重复使用。合理的命名symbol是一个非常好的主义，这样你可以轻松的说出来symbol要表达什么。
W> Because symbols are primitive values, `new Symbol()` throws an error when called. It's not possible to create an instance of `Symbol`, which also differentiates it from `String`, `Number`, and `Boolean`.
W> 由于symbol是基本类型值，所以'new Symbol()'方法会在调用时会抛出一个错误。创建一个symbol和'String','Number','Boolean'有些不同。
The `Symbol` function accepts an optional argument that is the description of the symbol. The description itself cannot be used to access the property but is used for debugging purposes. For example:
'Symbol'函数接收一个可选参数用于描述symbol。这个表述不能被用于访问属性，却可以用作调试用途。例如：
```js
var firstName = Symbol("first name");
var person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

A symbol's description is stored internally in a property called `[[Description]]`. This property is read whenever the symbol's `toString()` method is called either explicitly or implicitly (as in this example). It is not otherwise possible to access `[[Description]]` directly from code. It's recommended to always provide a description to make both reading and debugging code using symbols easier.
一个symbol的描述存储在属性中被称作[[Description]]的地方。当symbol的'toString()'方法被显式或隐式被调用（来看这个例子），这个属性被读取。从代码里直接访问'[[Description]]'是不可能的。推荐的方式是在早些使用symbol的时候提供一个描述来读和调试代码。
### Identifying Symbols

Since symbols are primitive values, you can use the `typeof` operator to identify them. ECMAScript 6 extends `typeof` to return `"symbol"` when used on a symbol. For example:
由于symbol是基本类型值，你可以使用'typeof'运算符来识别它们。ECMAScript 6 扩展了'typeof'，当被用于一个symbol时，'typeof'会返回'symbol'。例如：
```js
var symbol = Symbol("test symbol");
console.log(typeof symbol);         // "symbol"
```

While there are other indirect ways of determining whether a variable is a symbol, `typeof` is the most accurate and preferred way of doing so.
同时也有一些间接的方法来考虑一个变量是否是一个symbol，‘typeof’是最精确同时是首选方式。
### Using Symbols

You can use symbols anywhere you would use a computed property name. You've already seen the use of bracket notation in the previous sections, but you can use symbols in computed object literal property names as well as with `Object.defineProperty()`, and `Object.defineProperties()`, such as:
你可以在打算计算属性名（computed property name）的任何地方使用symbol。在之前的章节中，你已经见过了使用方括号表示法的方式，不过你也可以在计算对象字面量属性名时以及使用'Object.defineProperty()'和'Object.defineProperties'时使用symbol，比如：
```js
var firstName = Symbol("first name");
var person = {
    [firstName]: "Nicholas"
};

// make the property read only
Object.defineProperty(person, firstName, { writable: false });

var lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas"
```

With computer property names in object literals, symbols are very easy to work with.
在对象字面量中计算属性名，使用symbol非常简单。
### 分享标志符(Sharing Symbols)

You may find that you want different parts of your code to use the same symbols. For example, suppose you have two different object types in your application that should use the same symbol property to represent a unique identifier. Keeping track of symbols across files or large codebases can be difficult and error-prone. That's why ECMAScript 6 provides a global symbol registry that you can access at any point in time.
你也许会想在你的代码中不同部分使用相同的symbol。举个例子，假设在你的应用中有两个不同的对象类型使用相同的symbol属性来表现一个特殊的标识符，跨文件和代码库来跟踪symbol是困难且容易出错的。这就是为什么ECMAScript 6 提供一个全局symbol注册处，这样你可以在任何时刻访问。
When you want to create a symbol to be shared, use the `Symbol.for()` method instead of calling `Symbol()`. The `Symbol.for()` method accepts a single parameter, which is a string identifier for the symbol you want to create (this value doubles as the description). For example:
当你想要创建一个symbol来共享，使用'Symbol.for()'方法代替调用'Symbol()'。'Symbol.for()'方法接受一个单独的参数，一个你想要为symbol创建的字符串标识符。例如：
```js
var uid = Symbol.for("uid");
var object = {};

object[uid] = "12345";

console.log(object[uid]);       // "Nicholas"
console.log(uid);               // "Symbol(uid)"
```

The `Symbol.for()` method first searches the global symbol registry to see if a symbol with the key `"uid"` exists. If so, then it returns the already existing symbol. If no such symbol exists, then a new symbol is created and registered into the global symbol registry using the specified key. The new symbol is then returned. That means subsequent calls to `Symbol.for()` using the same key will return the same symbol:
'Symbol.for()'方法首先在全局symbol注册中搜索一个键为"uid"的symbol是不是存在。如果找到了，则返回已经存在的symbol。如果没有这样的一个symbol存在，则创建一个新的symbol并注册在全局symbol注册环境中使用一个专有的键。然后返回一个新的symbol。这意味着随后使用相同的键来调用'symbol.for()'会返回相同的symbol。
```js
var uid = Symbol.for("uid");
var object = {};

object[uid] = "12345";

console.log(object[uid]);       // "Nicholas"
console.log(uid);               // "Symbol(uid)"

var uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "Nicholas"
console.log(uid2);              // "Symbol(uid)"
```

In this example, `uid` and `uid2` contain the same symbol and so they can be used subsequent. The first call to `Symbol.for()` creates the symbol and second call retrieves the symbol from the global symbol registry.
在这个例子中，'uid'和'uid2'包含相同的symbol，所以他们可以随后调用。第一次调用'Symbol.for()'创建symbol，第二次则在全局symbol注册环境中检索。
Another unique aspect of shared symbols is that you can retrieve the key associated with a symbol in the global symbol registry by using `Symbol.keyFor()`, for example:
你可以在全局symbol注册处通过使用'Symbol.keyFor()'取回与symbol相关的键，这是symbol另一个独特的地方。
```js
var uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

var uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

var uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined
```

Notice that both `uid` and `uid2` return the key `"uid"`. The symbol `uid3` doesn't exist in the global symbol registry, so it has no key associated with it and so `Symbol.keyFor()` returns `undefined`.
注意，'uid'和'uid2'都返回键"uid"。symbol'uid3'在全局symbol注册中不存在，所以它没有相关的键于是'Symbol.keyFor()'返回'undefined'。
W> The global symbol registry is a shared environment, just like the global scope. That means you can't make assumptions about what is or is not already present in that environment. You should use namespacing of symbol keys to reduce the likelihood of naming collisions when using third-party components. For example, jQuery might prefix all keys with `"jquery."`, such as `"jquery.element"`.
全局symbol是在一个共享环境中注册的，就像全局作用域一样。这意味着你不能在那个环境中对已存在和尚未存在的symbol做出判断。当使用第三方控件时，你应该使用symbol的命名空间来减少命名冲突的可能性。例如，jQuery可以以“jquery.”为前缀，比如“jquery.element”

TODO

## Object Destructuring

TODO

## Object.assign()

TODO

## ~~__proto__~~ Object.setPrototypeOf()

TODO

## super

TODO