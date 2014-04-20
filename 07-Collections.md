# 集合（Collections）

对于大多数JavaScript的历史中，一直只存在一种类型的集合，以`Array`类型来表示。JavaScript中的数组和其它语言中的一样，然而它起到了两重甚至三重的作用，通过它在JavaScript中能够模拟出队列和栈。因为数组只能使用数字索引，在需要非数字索引时，开发者们不得不使用对象。对象（Objects）开始像map一样被使用，但是它也有着自己的局限性，即所有的属性名必须都是字符串。ECMAScript 6引入了一些新类型的集合，以便我们能够更好，更高效地来存储有序的数据。

## Sets

如果你曾经接触过Java, Ruby, 或者Python的话，Sets对你来说并不是什么新鲜东西。然而它在JavaScript中很长一段时间内都是缺失的。一个Set是一个不能包含重复值的有序列表。通常你不会像访问数组元素一样来访问Set中的元素，因为Set更常见的用法是用来检测一个值是否在其中出现过。

ECMAScrit 6将`Set`类型作为set的简单实现引入了JavaScript中。您可以通过使用'添加（）`方法添加值到一组，看看有多少项目是在集合使用`大小（）`：你可以使用`add()`方法向set中添加新值，或使用`size()`方法来查询目前set中有多少条目。

```js
    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.size());    // 2
```

ECMAScript 6的sets不会强制性地检查其中的值是否重复。因此，一个set中可以同时包含数字`5`和字符串`"5"`（实际上在内部，它使用`Object.is()`来做比较）。如果`add()`方法对同样的值上被调用了不止一次，那么所有在首次调用之后的调用都会被直接忽略掉。

```js
    var items = new Set();
    items.add(5);
    items.add("5");
    items.add(5);     // oops, duplicate - this is ignored

    console.log(items.size());    // 2
```

你可以直接用一个数组来初始化set，`Set`的构造器将会保证只有其中的非重复值会被使用到：

```js
    var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
    console.log(items.size());    // 5
```

在这个例子中，一个包含一系列数字的数组被用来初始化set。数`5`只有即使它出现四次在数组中出现一次的集合。其中数字5在set中只出现了一次，虽然在数组中它却有4次出现。这个功能可以帮助我们很方便地将现有的代码或者JSON结构的数据转化为sets。

你可以用过`has()`方法来检查哪些元素已经存在于set中：

```js
    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.has(5));    // true
    console.log(items.has(6));    // false
```

最后，你可以通过使用`delete()`方法从set中删除一个条目：

```js
    var items = new Set();
    items.add(5);
    items.add("5");

    console.log(items.has(5));    // true

    items.delete(5)

    console.log(items.has(5));    // false
```

所有的这些知识都将帮助你掌握一个非常简单的跟踪唯一无序值的方法。

!! 值


虽然没有办法随机访问set中的一个值，我们可以通过ECMAScript 6中的新语句`for-of`来迭代访问sets中的所有值。该`for-of`语句是一个循环，它会遍历set的所有值，包括数组和类数组结构。你可以通过这样来输出一个set中的所有值：

```js
    var items = new Set([1, 2, 3, 4, 5]);

    for (let num of items) {
        console.log(num);
    }
```

这段代码将会按照条目被添加到set中的顺序将set中的条目输出到控制台中。

如果你只是单纯想要将set中的值塞回数组中去，那么你可以使用`Array.from()`方法来很方便地达成这个目的。

```js
    var items = new Set([1, 2, 3, 4, 5]);
    var array = Array.from(items);
```

通过这种方法，你可以很简单地去除掉数组中的重复值：

```js
    function dedupe(array) {
        return Array.from(new Set(array));
    }

```

### 示例

目前，如果你想跟踪记录一个唯一值，最常用的的方法就是使用一个对象并且将这个唯一值作为具有真值的对象属性添加给对象。例如，这里有个CSS Lint规则用来检查重复的属性。现在我们有一个对象用以跟踪记录像这样的CSS属性：

```js
    var properties = {
        "width": 1,
        "height": 1
    };

    if (properties[someName]) {
        // do something
    }
```

以这种目的来使用对象必须保证给属性赋予一个真值，这样才能保证`if`语句能够正常工作。（另一种选择是使用`in`操作符，但是开发者一般很少选择这样做。）这整个过程可以通过使用一个set来大大简化：

```js
    var properties = new Set();
    properties.add("width");
    properties.add("height");

    if (properties.has(someName)) {
        // do something
    }
```

鉴于我们关心的只是这个属性之前有没有被使用过，而不关心它被使用了多少次（因为这里并没有任何额外的相关联的元数据），所以我们使用set会更加方便。

在此类操作中使用对象属性的另一个缺点在于属性名总会被转换为字符串。所以你不能创建一个具有属性名`5`的对象，你只能创建一个拥有属性名`"5"`的对象。这就意味着你无法轻松地使用相同的方法来跟踪记录对象，因为对象在被赋为属性名时会被强制转换为字符串。然而Set能够接受任意类型的数据，你并不用担心数据会被转换为另一类型。

### 浏览器支持

Firefox和Chrome浏览器已经实现`Set`，但是在Chrome浏览器中你需要手动启用ECMAScript 6 特性支持：访问`chrome://flags` 并且启用"Experimental JavaScript Features". 这两个浏览器中的实现都是不完整的。两个浏览器都没有实现`for-of`并且Chrome的实现还缺少了`size()`方法。