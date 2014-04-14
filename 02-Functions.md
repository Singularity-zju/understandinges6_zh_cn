# 函数


## 默认参数 （Default Parameters）

JavaScript中的函数是独一无二的，因为它允许你传递任意数量的参数，而不用管函数定义中所声明的参数数量。这将允许你定义一个能够接受不同数量参数的函数，通常我们通过（给其余参数）填充所提供的默认值来实现。在ECMAScript 5和更早版本中，你可能会使用下面的模式来实现这一点：

```js
    function combineText(start, middle, end) {

        middle = middle || "";
        end = end || "";

        return start + middle + end;
    }
```

逻辑运算符OR(`||`)在第一个操作数是false时，总是返回第二个操作数。由于命名函数中没有显式提供的参数都会被设置为`undefined`，逻辑OR运算符经常被用来为缺少的参数提供默认值。其它的一些用来判断是否有参数未被提供的方法包括检查`arguments.length`来获得被传入的参数个数，或者直接检查每个参数看它是否是`undefined`。

ECMAScript 6通过给那些没有正式传递的参数提供初始化的值来使得给参数设置默认值变得更加简单。例如：

```js
    function combineText(start, middle = "", end = "") {
        return start + middle + end;
    }
```

这里，只有第一个参数是被要求必须被传入的。其他两个参数都有一个空字符串作为默认值，这样使得函数体更加精简，因为你不需要添加任何其它代码来检查参数是否缺失。当`combineText()` 在调用时传入了所有的三个参数，那么默认值就不会被使用。

有默认值的任何参数都被认为是可选参数，而那些没有默认值的参数则会被认为是必须提供的参数。

## 不定参数 （Rest Parameters）

由于JavaScript函数可以传递任意数量的参数，我们并不总是需要专门去定义每个参数。在早期，JavaScript提供了`arguments`对象作为检查所有被传入的函数参数的一种方法，因此我们就不必去单独定义每一个参数。虽然在大多数情况下，这种方法很好，但有时候它也可能会引起一些麻烦。例如：

```js
    function sum(first) {
        var result = first,
            i = 1,
            len = arguments.length;

        while (i < len) {
            result += arguments[i];
            i++;
        }

        return result;
    }
```

这个函数将所有传递给它的参数累加起来，所以你可以调用`sum(1)`或`sum(1,2,3,4)`，它都可以正常运行。此函数有几个地方需要注意，第一，这个函数能够接受不止一个参数这件事并不是显而易见的。虽然您可以再添加几个命名参数，但你这么做也是没用的，因为实际上这个函数可以接受任意数量的参数。其次，因为第一个参数被命名并被直接使用了，所以你必须从索引1在`arguments`对象中寻找其余参数，而不是索引0.虽然记住`arguments`的索引并不困难，但是它毕竟需要额外的精力去记录。ECMAScript 6引入的补丁参数则很好地帮助解决了这些问题。

不定参数是在前面含有三个点(`...`)的命名参数。由此，这个命名参数就变成了一个包含其余参数的`Array`。（这就是为什么它们被称为"rest parameters" ）。例如，`sum()`函数可以用这样的不定参数被改写：

```js
    function sum(first, ...numbers) {
        let result = first,
            i = 0,
            len = numbers.length;

        while (i < len) {
            result += numbers[i];
            i++;
        }

        return result;
    }
```

在这个版本的函数中，`numbers`是一个包含有所有除去第一个参数之外的参数的不定参数。（与`arguments`不同，`arguments`包含了所有参数，包括第一个。）这意味着你可以从开始到结束来遍历`number`，而不用担心任何问题。还有个额外好处，你可以通过查看函数头本身来判断出它是一个可以处理任意多参数的函数。

注意：`sum()` 方法实际上并不需要任何的命名参数。理论上说，你可以只用不定参数，并且让它与之前一样正常工作。然而，在这种情况下，不定参数就会和`arguments`参数一样，所以你没有真正得到任何额外的好处。

对不定参数的唯一限制是，不定参数之后不允许有任何命名参数出现。例如，这将导致语法错误：

```js
    // Syntax error: Can't have a named parameter after rest parameters
    function sum(first, ...numbers, last) {
        let result = first,
            i = 0,
            len = numbers.length;

        while (i < len) {
            result += numbers[i];
            i++;
        }

        return result;
    }
```

这里，参数`last` 在不定参数`numbers`之后，这将导致一个语法错误。

在ECMAScript中，不定参数被设计来取代`arguments`。最初的ECMAScript 4就废除了`arguments`，并添加了不定参数以允许任意数量的参数传递给函数。尽管ECMAScript 4从未能够出现，然而这个想法却保留了下来，并且在ECMAScript 6中被重新引入，即使`arguments`还尚未从语言中被删除。

## 箭头函数（Arrow Functions）

在JavaScript中最容易发生错误的地方就是在函数内对`this`的绑定。由于`this`的值取决于调用它时所处的环境，所以可能在函数内被修改，这样的话就有可能在你本想要它指向一个对象时却错误地指向了另一个。仔细看看下面的例子：

```js
    var PageHandler = {

        id: "123456",

        init: function() {
            document.addEventListener("click", function(event) {
                this.doSomething(event.type);     // error
            }, false);
        },

        doSomething: function(type) {
            console.log("Handling " + type  + " for " + this.id);
        }
    };
```

在这段代码中，`PageHandler`对象被设计用来处理页面上的用户互动。其中`init()`方法被调用来设置互动操作的处理函数，所以这个方法会指派一个事件处理器去调用`this.doSomething()`。然而，这段代码并不会像预期一样成功执行。对`this.doSomething()`的调用是错误的，因为`this`是一个指向元素对象的引用（在这种情况下是`document`），该元素这是这个事件的目标元素。这里`this`并没有绑定到`PageHandler`上。如果您尝试运行这段代码，你会在事件处理程序被触发时得到一个错误，因为`this.doSomething()`在目标对象`document`中并不存在。

你可以通过显式地使用`bind()`来将`this`的值绑定到`PageHandler`。

```js
    var PageHandler = {

        id: "123456",

        init: function() {
            document.addEventListener("click", (function(event) {
                this.doSomething(event.type);     // no error
            }).bind(this), false);
        },

        doSomething: function(type) {
            console.log("Handling " + type  + " for " + this.id);
        }
    };
```

现在代码能够正常工作了，但看起来可能会有点怪。通过调用`bind(this)`，你实际上是在创建一个新的函数，它的`this`被绑定到当前的`this`，而当前的`this`也即是`PageHandler`对象。现在代码像你所预期的那样工作了，但是，你为了搞定它不得不创建出一个新的函数。

箭头函数是ECMAScript 6中的一个定义函数的新途径，它解决了`this`绑定的问题。一个箭头函数的基本语法如下：

```js
    (arg1, arg2, ...argN) => result
```

你可以在括号内指定任意数量的参数，然后使用`=>`来定义函数的主体内容。如果函数主体只返回一个值或执行一个语句，那么你可以直接在`=>`右边写出结果。那个语句被求值后会作为结果返回。例如：

```js
    var sum = (first, middle, last) => first + middle + last;
    console.log(sum(1, 2, 3));  // 6

If, on the other hand, the function body requires multiple steps, you can include those using braces in this form:

    (arg1, arg2, ...argN) => {

        // more code

        return result;
    }
```

当使用大括号来标识箭头函数的函数体时，你必须明确地使用`return`语句来返回函数值。否则，它的行为将会和没有括号时完全一样。如果函数体内什么都没有，那么大括号是必须的。

注意：当只有一个参数传递给函数时，你可以省略掉（参数列表的）括号。

箭头函数含有隐式的`this`绑定，这代表在箭头函数中，`this`的值总是和箭头函数定义时所处环境中的`this`值相同。例如：

```js
    var PageHandler = {

        id: "123456",

        init: function() {
            document.addEventListener("click",
                    event => this.doSomething(event.type), false);
        },

        doSomething: function(type) {
            console.log("Handling " + type  + " for " + this.id);
        }
    };
```

在这个例子中，事件处理程序是一个调用了`this.doSomething()`的箭头函数。其中`this`的值与在`init()`中的`this`值是一样的，所以这个版本的例子的工作方式类似于前面使用`bind()`的例子。尽管`DoSomething()`方法没有返回值，但它仍然是函数体中唯一的必须执行的表达式，所以这里我们没必要使用大括号。

箭头函数被设计为“一次性”的函数，因此它不能被用于定义新类型。通过它与常规函数相比所缺少的`prototype`属性可以明白地看出这一点。如果你对一个箭头函数尝试使用`new`操作符，你会得到一个错误：

```js
    var MyType = () => {},
        object = new MyType();  // error - you can't use arrow functions with 'new'
```

此外，由于`this`值在箭头函数中是静态绑定的，因此你不能使用`call()`，`apply()`或者`bind()`来改变`this`的值。

箭头函数的简洁语法，使得它非常适合用来做数组处理。例如，如果你想用自定义的比较器来对数组进行排序，你通常会这样写：

```js
    var result = values.sort(function(a, b) {
        return a - b;
    });
```

有很多不同的格式来实现这样一个简单的程序。但与此相比，使用箭头函数的版本则显得更加简洁：

```js
    var result = values.sort((a, b) => a - b);
```

类似于`sort()`，`map()`和`reduce()`之类的接受回调函数的数组方法都可以从箭头函数的简单的语法中获益，从而能够将复杂的过程用更简单的代码来实现。

一般来说，箭头函数被设计用在目前我们使用匿名函数的地方。它并没有被设计用来（像命名函数一样）保持很长时间，因此无法将箭头函数用作构造函数。箭头函数最适用于作为传递给其它函数的回调函数，就如同我们在这一节的例子中所看到的一样。



##########

ECMAScript 6中最有趣的新部分之一就是箭头函数。箭头函数，顾名思义，使用一个“箭头”(`=>`)作为语法的一部分，这是一个新的语法定义的函数。然而，在一些重要的方面，箭头函数的行为明显不同于传统的JavaScript函数：

* **词法的`this`绑定**  - 函数中`this`的值是在箭头函数被定义的地方所确定的，而不是在它被调用的地方被确定。
* **不能被`new`**  - 箭头函数无法使用构造函数，在使用`new`时将会抛出一个错误。
* **无法更改`this`的值**  - 在函数内无法更改`this`的值，在函数的整个生命周期内，`this`的值都将保持不变。
* **没有`arguments`对象**  - 你将无法通过`arguments`对象来访问参数，你必须使用命名参数，或者其它ES6特性，例如不定参数。

这里有一些原因能够解释为什么存在这些差别。首先，`this`绑定是Ja​​vaScript中的一个常见错误来源。在函数中，非常容易失去对`this`值的掌握，而这很容易导致意想不到的后果。第二，通过限制箭头函数让其只能简单地带着一个不变的`this`值来执行代码，JavaScript引擎可以更容易地优化这些操作（相对于普通函数来说，因为它们可能被用来作为一个构造函数或以其他方式修改`this`的值）。

## 语法

箭头函数的代码格式可以有许多种不同风格，这取决于你想要用它来完成什么事。所有的变化开始于函数参数，紧接着是箭头，最后则是函数体。取决于用法，函数参数和函数体都可以使用不同的形式。例如，下面的箭头函数接受一个参数，并且仅仅直接返回它：

```js
    var reflect = value => value;

    // effectively equivalent to:

    var reflect = function(value) {
        return value;
    };
```

当箭头函数只有一个参数时，这个参数可以直接被使用而不需要添加任何其它的格式。后面紧跟着箭头，箭头右边是将要被执行并返回的表达式。即使没有明确的`return`语句，这个箭头函数也将返回传入的第一个参数。

如果你需要传递多个参数，则不能省略掉括住这些参数的括号。例如：

```js
    var sum = (num1, num2) => num1 + num2;

    // effectively equivalent to:

    var sum = function(num1, num2) {
        return num1 + num2;
    };
```

`sum()`函数只是简单地将两个参数相加并返回结果。唯一的区别是，参数被括在括号中，并以逗号分隔。（与传统函数相同）。

如果您想提供一个更传统的函数体，比如你需要执行不止一句话的表达式，那么你需要在函数体中使用大括号并且显式定义一个返回值，如：

```js
    var sum = (num1, num2) => { return num1 + num2; }

    // effectively equivalent to:

    var sum = function(num1, num2) {
        return num1 + num2;
    };
```

你可以认为在大括号之间的函数体与传统函数是差不多的，除了这里没有`arguments`对象。

因为大括号是用来表示函数体的，所以如果一个箭头函数想要在函数体之外返回一个对象字面量，那么这个对象字面量必须被包含在括号中。例如：

```js
    var getTempItem = id => ({ id: id, name: "Temp" });

    // effectively equivalent to:

    var getTempItem = function(id) {

        return {
            id: id,
            name: "Temp"
        };
    };
```

将对象字面量包含在括号中代表其中的大括号是表示对象字面量而并非函数体。

## 其它需要知道的事实

箭头函数与传统函数有一些不同，但是也有一些共通之处。例如：

* `typeof`运算符对于箭头函数也会返回"function"
* 箭头函数仍然是`Function`的实例，所以`instanceof`对于箭头函数仍然能够正常工作。
* `call()`，`apply()`和`bind()`方法对于箭头函数来说仍然可用，虽然它们不会再影响到`this`的值。

最大的区别在于箭头函数不能用`new` - 试图这样做的结果将会是被抛出错误。

## 结论

箭头函数是ECMAScript 6中的一个有趣的新功能，并且是那些在目前看来能够确定下来的特性之一。由于将函数作为参数来传递已经变得越来越流行，拥有一个简洁的语法来定义这些函数，对于我们一直在这么做的人来说是一个可喜的变化。对于词法`this`的绑定解决了开发人员的最大痛苦之处，并且对于通过JavaScript引擎优化来提高性能也大有裨益。如果你想尝试一下箭头函数，试试刚刚发行的最新版本Firefox吧，这是第一个搞定箭头函数实现的官方版本。