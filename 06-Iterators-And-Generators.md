# 迭代器和生成器 ( Iterators and Generators )


## 表达语句 ( Statements )


### for-of

在 ECMAScript 5和更早之前的版本, 对数组进行迭代有多种选择(方法). 当然, 你肯定可以使用久经考验的`for`循环:

``` js
var i, len;

for (i=0, len=values.length; i < len; i++) {
    process(values[i]);
}
```

你也可以使用`forEach()`方法:

```js
values.forEach(function(value) {
    process(value);
});
```

然而,这些方法有一些限制. `for` 循环需要进行变量设置; `forEach()` 只能对数组进行使用,其他数组类似的对象不能使用. ECMAScript 6用新的表达语句解决了这些问题.

`for-of` 便是设计用来对数组类似的对象进行迭代的. 与其设置一个变量来跟踪数组中正被处理的元素, 你可以简单地使用一个变量, 该变量会不断地被赋予数组里面的新值,直到整个数组都被过一遍.

示例如下:

```js
For example

    for (let value of values) {
        process(value);
    }
```

在上面的代码中, `for-of` 定义了 `value` 这个变量来保存数组 `values` 的值. 循环从数组的第一个值开始, 直到最后一个. 因为这个语句是对数组类似的值作用的,所以你也能对 `arguments` 和 DOM `NodeList` 等这些对象使用(这两个对象都没有 `forEach()` 方法):

```js
// Print out all arguments
function printArgs() {
    for (let arg of arguments) {
        console.log(arg);
    }
}

// Iterate over all <div> elements
var divs = document.getElementsByTagName("div");

for (let div of divs) {
    div.innerHTML = "Processed...";
}
```

相对于其他必须考虑对象类型的迭代方法, `for-of` 语句对任何数组类似对象的兼容性是一个重大的改善.

注意: `for-of` 语句能对任何迭代对象使用, 包括 arrays, arguments, DOM `NodeList` objects 和 generators. 但它不能对常规对象( regular objects )使用.


#### @@iterator


