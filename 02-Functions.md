# 函数


## 默认参数

JavaScript中的函数是独一无二的，因为它允许你传递任意数量的参数，而不用管函数定义中所声明的参数数量。这将允许你定义一个能够接受不同数量参数的函数，通常我们通过（给其余参数）填充所提供的默认值来实现。在ECMAScript 5和更早版本中，你可能会使用下面的模式来实现这一点：

    function combineText(start, middle, end) {

        middle = middle || "";
        end = end || "";

        return start + middle + end;
    }

逻辑运算符OR(`||`)在第一个操作数是false时，总是返回第二个操作数。由于命名函数中没有显式提供的参数都会被设置为`undefined`，逻辑OR运算符经常被用来为缺少的参数提供默认值。其它的一些用来判断是否有参数未被提供的方法包括检查`arguments.length`来获得被传入的参数个数，或者直接检查每个参数看它是否是`undefined`。

ECMAScript 6通过给那些没有正式传递的参数提供初始化的值来使得给参数设置默认值变得更加简单。例如：

    function combineText(start, middle = "", end = "") {
        return start + middle + end;
    }

这里，只有第一个参数是被要求必须被传入的。其他两个参数都有一个空字符串作为默认值，这样使得函数体更加精简，因为你不需要添加任何其它代码来检查参数是否缺失。当`combineText()` 在调用时传入了所有的三个参数，那么默认值就不会被使用。

有默认值的任何参数都被认为是可选参数，而那些没有默认值的参数则会被认为是必须提供的参数。

## 不定参数

由于JavaScript函数可以传递任意数量的参数，我们并不总是需要专门去定义每个参数。在早期，JavaScript提供了`arguments`对象作为检查所有被传入的函数参数的一种方法，因此我们就不必去单独定义每一个参数。虽然在大多数情况下，这种方法很好，但有时候它也可能会引起一些麻烦。例如：

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

这个函数将所有传递给它的参数累加起来，所以你可以调用`sum(1)`或`sum(1,2,3,4)`，它都可以正常运行。此函数有几个地方需要注意，第一，这个函数能够接受不止一个参数这件事并不是显而易见的。虽然您可以再添加几个命名参数，但你这么做也是没用的，因为实际上这个函数可以接受任意数量的参数。其次，因为第一个参数被命名并被直接使用了，所以你必须从索引1在`arguments`对象中寻找其余参数，而不是索引0.虽然记住`arguments`的索引并不困难，但是它毕竟需要额外的精力去记录。ECMAScript 6引入的补丁参数则很好地帮助解决了这些问题。

不定参数是在前面含有三个点(`...`)的命名参数。由此，这个命名参数就变成了一个包含其余参数的`Array`。（这就是为什么它们被称为"rest parameters" ）。例如，`sum()`函数可以用这样的不定参数被改写：

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

在这个版本的函数中，`numbers`是一个包含有所有除去第一个参数之外的参数的不定参数。（与`arguments`不同，`arguments`包含了所有参数，包括第一个。）这意味着你可以从开始到结束来遍历`number`，而不用担心任何问题。还有个额外好处，你可以通过查看函数头本身来判断出它是一个可以处理任意多参数的函数。

注意：`sum()` 方法实际上并不需要任何的命名参数。理论上说，你可以只用不定参数，并且让它与之前一样正常工作。然而，在这种情况下，不定参数就会和`arguments`参数一样，所以你没有真正得到任何额外的好处。

对不定参数的唯一限制是，不定参数之后不允许有任何命名参数出现。例如，这将导致语法错误：

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

这里，参数`last` 在不定参数`numbers`之后，这将导致一个语法错误。

在ECMAScript中，不定参数被设计来取代`arguments`。最初的ECMAScript 4就废除了`arguments`，并添加了不定参数以允许任意数量的参数传递给函数。尽管ECMAScript 4从未能够出现，然而这个想法却保留了下来，并且在ECMAScript 6中被重新引入，即使`arguments`还尚未从语言中被删除。

## 箭头函数

TO DO