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

## Maps

Maps对于那些来自于其它语言的开发者来说也是一个熟悉的话题。其基本思路是将一个值映射到一个唯一的键上，使用这个键你可以在任何时间轻松检索到值。在JavaScript中，开发人员通常使用普通对象来作为maps的替代品。实际上，JSON正是建立在对象所表示的键值对上的。然而，正如像模拟sets一样，对象对maps的模拟也受到相同的限制：不能创建含有非字符串的键。

在ECMAScript 6之前，你可能已经看到过看起来像这样的代码：

```js
var map = {};

// later
if (!map[key]) {
    map[key] = value;
}
```

这段代码用了普通对象来模拟map，检测一个给定的键是否存在。这段代码最大的限制就是`key`将一定会被转换为字符串。这当然不是什么很重要的事情，除非你要用一个非字符串值来作为键。例如，也许你打算存储一些与特定的DOM元素相关的值。你也许会尝试着这样做：

```js
// element gets converted to a string
var data = {},
    element = document.getElementById("my-div");

data[element] = metadata;
```

不幸的是，`element`将hui被转换成字符串`"[Object HTMLDivElement]"`或类似的东西（精确的值可能会根据不同的浏览器而有所不同）。这样做是存在问题的，因为每一个`<div>`元素会被转换成相同的字符串，这意味着你将会不断重复使用相同的键，虽然你看起来是使用了不同的元素。出于这个原因，`Map`类型成为了JavaScript中的一个受欢迎的增强功能。

ECMAScript 6的`Map`类型是一个键值对的有序列表，其中键和值都能够使用任意的类型。键`5`和键`"5"`是两个不同的键，引擎会使用`Object.is()`方法来确定两个键是否相同。你可以使用`set()`和`get()`方法来在一个map中存储和检索数据，例如：

```js
var map = new Map();
map.set("name", "Nicholas");
map.set(document.getElementById("my-div"), { flagged: false });

// later
var name = map.get("name"),
    meta = map.get(document.getElementById("my-div"));
```

在这个例子中，我们存储了两个键值对。键`"name"`中存储了一个字符串，而键`document.getElementById("my-div")`用来存储DOM元素相关的元数据。如果一个键在map中不存在，则调用`get()`时将会返回一个特殊的值`undefined`。

Maps和Sets有一些相同的方法，比如`has()`用来确定一个键是否在map中存在，而`delete()`则用来从map中删除一个键值对。你也可以使用`size`来确定map中含有多少条目。

```js
var map = new Map();
map.set("name", "Nicholas");

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.size);        // 1

map.delete("name");
console.log(map.has("name"));   // false
console.log(map.get("name"));   // undefined
console.log(map.size);        // 0
```

为了更方便地一次性添加大量数据到map中，你可以给`Map`构造函数传递一个二维数组。在实现内部，每个键值对会被存储为一个含有两个条目的数组，第一个会作为键，第二个会作为值。因此，整个map就是一个内容为两元素数组的一个数组，所以maps能够使用这种格式的数组来初始化：

```js
var map = new Map([ ["name", "Nicholas"], ["title", "Author"]]);

console.log(map.has("name"));   // true
console.log(map.get("name"));   // "Nicholas"
console.log(map.has("title"));  // true
console.log(map.get("title"));  // "Author"
console.log(map.size);        // 2
```

如果你想处理map中的所有数据，那么你有几种选择。实际上有三种生成器方法供你选择：`keys`，它将迭代map中的所有键，`values`，将迭代map中的所有值，以及`entries`，它将迭代所有的键值对，通过返回一个包含键值对的数组。（`entries`是maps的默认迭代器）。要利用这些迭代器，最简单的方法是使用一个`for-of`循环：

```js
for (let key of map.keys()) {
    console.log("Key: %s", key);
}

for (let value of map.values()) {
    console.log("Value: %s", value);
}

for (let item of map.entries()) {
    console.log("Key: %s, Value: %s", item[0], item[1]);
}

// same as using map.entries()
for (let item of map) {
    console.log("Key: %s, Value: %s", item[0], item[1]);
}
```

当遍历键或值时，你将会每次从循环中接收到一个值。当遍历条目时，你会接收到一个数组，其第一项是键，第二项是值。

另一种遍历项的方法是使用`forEach()`方法。这个方法的工作方式和数组的`forEach()`方法类似。你需要传递一个函数，这个函数将在调用时接收三个参数：值，键和map本身。例如：

```js
map.forEach(function(value, key, map)) {
    console.log("Key: %s, Value: %s", key, value);
});
```

类似于数组的`forEach()`，你可以传递可选的第二个参数来指定回调函数中的`this`值：

```js
var reporter = {
    report: function(key, value) {
        console.log("Key: %s, Value: %s", key, value);
    }
};

map.forEach(function(value, key, map) {
    this.report(key, value);
}, reporter);
```

在这里，回调函数中的`this`值是`reporter`。这使得`this.report()`能够正常工作。

与此相对的是对普通对象中迭代值的笨拙方式。

```js
for (let key in object) {

    // make sure it's not from the prototype!
    if (object.hasOwnProperty(key)) {
        console.log("Key: %s, Value: %s", key, object[key]);
    }

}
```

当你将对象作为map使用时，在`for-in`循环中原型的属性也许会泄露，这将始终是一个需要考虑的问题。你必须使用`hasOwnProperty()`来保证你只取得你所需要的那些属性。当然，如果对象有方法，则你也必须筛选掉它们：

```js
for (let key in object) {

    // make sure it's not from the prototype or a function!
    if (object.hasOwnProperty(key) && typeof object[key] !== "function") {
        console.log("Key: %s, Value: %s", key, object[key]);
    }

}
```

Map的迭代功能让您可以集中精力来处理数据，而无需担心额外的信息出现在你的代码中。这是map在存储键值对中，相对于普通对象来说所具备的的巨大优势。

### 浏览器支持

Firefox和Chrome浏览器已经实现了`Map`，然而，在Chrome中你需要手动启用ECMAScript 6 特性支持：访问`chrome://flags`并且启用"Experimental JavaScript Features"。这两个浏览器的实现都是不完整的。两个浏览器都没有实现任何一个能够在`for-of`中使用的生成器方法，并且Chrome的实现还缺乏了`size`方法（这个方法目前在ECMAScript 6草案中）。


## Weakmaps

Weakmaps类似于普通的Maps，因为它们的值都映射到一个唯一的键上 。该键可以在之后用来检索值。Weakmaps和Maps有所不同因为它的键必须是一个对象，而不能是一个基本类型。这看起来似乎是一个奇怪的限制，但是它实际上是使得weakmaps与众不同并且有用的核心。

一个weakmap仅持有键的弱引用，这意味着在weakmap内部的引用并不会阻止对于对象的垃圾回收。当对象被垃圾收集器销毁时，weakmap将自动删除该对象所标识的键值对。weakmaps的典型使用场景是创建一个与特定DOM元素相关联的对象。例如，jQuery在内部对于每个引用过的DOM元素都维护了一个对象缓存。使用weakmap将允许jQuery在DOM元素被从文档中删除时，能够自动释放与该DOM元素相关联的内存。

ECMAScript 6的`WeakMap`类型是一个键值对的无序列表，其中键必须是一个非空的对象，值则可以是任何类型。`WeakMap`的接口非常类似于`Map`，从它们都用`set()`和`get()`来增加和检索数据就可以看出：

```js
var map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

// later
var value = map.get(element);
console.log(value);             // "Original"

// later still - remove reference
element.parentNode.removeChild(element);
element = null;

value = map.get(element);
console.log(value);             // undefined
```

在这个例子中，一个键值对被存储了起来。键是一个DOM元素用于存储一个对应的字符串值。该值后来通过将DOM元素传递给`get()`方法来被检索出。如果在这之后，DOM元素被从文档中删除，并且对应的变量被设置成`null`，那么weakmap中的所对应的数据也会被同时删除，在这之后，下一次对这个DOM元素所关联的数据的检索行为会失败。

这个例子有一点点误导，因为第二次对`map.get(element)`的调用是使用的`null`值（因为`element`变量此时已经被设置为`null`），而并不是DOM元素的引用。你不能使用`null`作为weakmaps的键，所以这段代码其实并没有真正执行一次合法的检索。遗憾的是，并没有一个接口来允许你查询一个引用是否已经被清理（通常由于引用不再存在）

> weakmap的`set()`方法将会在你试图使用基本类型作为键时抛出一个错误。如果你想用基本类型作为键，那么你最好使用`Map`而不是`Weakmap`。

Weakmaps也有`has()`方法来确定一个键是否存在于weakmap中，同时，也有`delete()`方法来删除一个键值对。

```js
var map = new WeakMap(),
    element = document.querySelector(".element");

map.set(element, "Original");

console.log(map.has(element));   // true
console.log(map.get(element));   // "Original"

map.delete(element);
console.log(map.has(element));   // false
console.log(map.get(element));   // undefined
```

这里，DOM元素被再次用作一个weakmap的键。`has()`方法在检查一个引用是否已经作为一个键存在于weakmap中时是非常有用的。请记住，这个语句只有在你的键关联到一个非null的引用时才有效。通过使用`delete()`方法，键能够从weakmap中被强制移除，此时再检索该键，`has()`将返回false而`get()`将返回`undefined`。

### 用途和局限性

Weakmaps有一​​个非常特殊的使用场景你需要牢牢记住，即用于映射那些将来可能会消失的对象。这种能够释放那些与对象相关联的内存的能力对于那些要用自定义的对象去包装DOM元素的JavaScript库（例如jQuery和YUI）来说是非常有用的。一旦Weakmaps的实现被完成并且广泛使用开来，也许会有更多的使用场景。所以在短期内，如果你无法找到一个使用weakmaps的好场景，请不要感到悲伤。

在许多情况下，一个普通的map可能就是你想要的东西。Weakmaps的一大限制在于它是无法枚举的，你无法跟踪记录其中到底有多少条目。此外，你也没有一种方法来检索其中所有的键。如果你需要这型功能，那么你就需要使用一个普通的map。如果你并不需要这样，并且你只想使用对象来作为键，则使用一个weakmap可能是你的好选择。

### 浏览器支持

Firefox和Chrome浏览器已经实现了`WeakMap`，然而，在Chrome中你需要手动启用ECMAScript 6 特性支持：访问`chrome://flags`并且启用"Experimental JavaScript Features"。这两种浏览器的实现都是按照目前的稻草人规范（strawman specification）来完成的。（weakmaps还尚未出现在ECMAScript 6草案中）

## 小结

TODO

ECMAScript 6的sets是一个很受欢迎的语言增强功能。它们允许你轻松地创建唯一值的集合，而不用担心强制类型转换。你可以非常轻松地给一个set添加或删除条目，即使我们并没有一种方法来直接存取set中的条目。但如果需要的话，我们仍旧可以通过ECMAScript 6的`for-of`来遍历set中的所有项来达到这个目的。

由于ECMAScript 6还没有完成，它的实现和规范有可能在其它浏览器开始加入`Set`之前，就会发生变更。在现在，它仍然被认为是一个试验性的API，并且不应该被用在产品代码中。这篇文章，以及其它有关ECMAScript 6的文章，仅仅是为了给即将到来的这些功能做一个预览。