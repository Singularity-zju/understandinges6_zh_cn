# 代理 （Proxies）

在ECMAScript 6中Proxies有一个漫长而复杂的历史。在TC-39决定以一个非常激动人心的方式来改变proxies之前，一个早期的提案已经被Firefox和Chrome所实现了。在我眼中，这些变化将原有的proxies的提案中粗糙的那一部分都理顺了，这是件好事。

通过实验，我发现大多数的proxy都围绕一个目标对象来使用，实际上这样使得proxies更像一个在开发者和对象之间的一个过滤器，而并非一个独立概念。之后，Proxies以一种新形式重生了，这其中目标对象成为了proxy定义的一部分。这便是最终被标准化的ECMAScript 6 Proxy 设计。

## Proxy理论知识

TODO

## 创建Proxies

TODO

```js
var proxy = new Proxy({}, {
    get: function(target, property) {
        return 35;
    }
});

console.log(proxy.time);        // 35
console.log(proxy.name);        // 35
console.log(proxy.title);       // 35
```

## 用途


### 防御型对象

了解如何拦截`[[Get]]`操作是创建一个"防御型"的对象的充分必要条件。我将它们称之为防御型，因为它们的行为就好像一个防御性很强的青少年，试图从它们父母的视角中维护它们的独立。我们的目标是当一个不存在的属性被访问时，抛出一个错误（“我又`不是`一只鸭子，为什么你要一直像对待一只鸭子一样对待我！”）。这个目标可以通过一小段设置`get`陷阱的代码来完成：

```js
function createDefensiveObject(target) {

    return new Proxy(target, {
        get: function(target, property) {
            if (property in target) {
                return target[property];
            } else {
                throw new ReferenceError("Property \"" + property + "\" does not exist.");
            }
        }
    });
}
```

该`createDefensiveObject()` 函数接受一个目标对象，并为它创建一个防御型对象。这里proxy有一个`get`陷阱，会当属性被读取时做一些检查。如果属性确实存在于目标对象上，则属性值会被返回。另一方面，如果对象上并不存在这个属性，则会抛出一个错误。下面是一个例子：

```js
let person = {
    name: "Nicholas"
};

let defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);      // "Nicholas"
console.log(defensivePerson.age);       // ReferenceError!
```

在这里，`name`属性照常工作，而`age`则抛出了一个错误。
防御型对象允许读取存在的属性，但是对于不存在的属性则会在读取时抛出错误。当然，你仍然可以毫无压力地向对象添加新属性。


```js
var person = {
    name: "Nicholas"
};

var defensivePerson = createDefensiveObject(person);

console.log(defensivePerson.name);        // "Nicholas"

defensivePerson.age = 13;
console.log(defensivePerson.age);         // 13
```

所以，对象将保持它们变化的能力，除非你想要做一些事情去改变这个事实。属性可以随时添加，但独读取不存在的属性将会抛出一个错误，而不仅仅是返回一个`undefined`。

然后，您可以真正地捍卫一个对象的接口，不允许添加属性，也不允许访问一个不存在的属性，通过以下的步骤可以实现：

```js
function Person(name) {
    this.name = name;

    return createDefensiveObject(name);
}
```

TODO

### 类型安全

在类型安全背后的思想是每个变量或者属性都应该只包含一个特定类型的值。在类型安全的语言中，类型在声明中被同时定义。当然，在JavaScript中，我们本身是没有办法做出这样的声明的。然而，很多时候，属性是用一个值来初始化的，而该值就预示着数据应包含的类型。例如：

```js
var person = {
    name: "Nicholas",
    age: 16
};
```

在这段代码中，我们可以很容易地看出，`name`应该包含一个字符串，`age`应该包含一个数字。在该对象的使用期间，你肯定不会想要这些属性包含其他类型的数据。通过使用proxies，我们有可能利用这些信息来保证以后被赋给这些属性的新值也是同一类型的。

由于复制是我们需要考虑的操作（即将一个新值赋给一个属性），你需要使用proxy的`set`陷阱。该`set`陷阱在当属性值被设置的时候调用，并且会接受四个参数：操作的目标，属性名，新值，接受的对象。目标和接受者一直是相同的（我可以这样确定）为了保证属性不被赋予错误的值，我们只需要简单地对于现有的值和新值的类型进行评估，并在不匹配时抛出错误。

```js
function createTypeSafeObject(object) {

    return new Proxy(object, {
          set: function(target, property, value) {
              var currentType = typeof target[property],
                  newType = typeof value;

              if (property in target && currentType !== newType) {
                  throw new Error("Property " + property + " must be a " + currentType + ".");
              } else {
                  target[property] = value;
              }
          }
    });
}
```

该`createTypeSafeObject()`方法接受一个对象，并且为其建立一个带`set`陷阱的proxy。这个陷阱使用`typeof`来获得已有属性和被传入的值的类型。如果属性已存在并且这两个类型不匹配，则会抛出一个异常。如果该属性并不存在，或者两个类型匹配，则这次赋值操作会成功如常。这也允许了对象在接受新属性时不引发错误。例如：

```js
var person = {
    name: "Nicholas"
};

var typeSafePerson = createTypeSafeObject(person);

typeSafePerson.name = "Mike";        // succeeds, same type
typeSafePerson.age = 13;             // succeeds, new property
typeSafePerson.age = "red";          // throws an error, different types
```

在这段代码中，`name`属性被更改时并没有出错，因为它只是改成了另一个字符串。而`age`属性作为一个数字类型被添加，并且，当它的值被设置成字符串时，抛出了一个错误。既然属性在第一次使用时就被初始化为合适的类型了，那么所有后续的改变将被保证是正确的。这意味着，你必须正确地初始化那些还无效的值。`typeof null`返回"object"的奇怪现象在这里也能够正常工作，因为一个`null`值的属性应当能够在之后被赋予任何对象。

与防御型对象一样，你也可以将此方法应用于构造函数：

```js
function Person(name) {
    this.name = name;
    return createTypeSafeObject(this);
}

var person = new Person("Nicholas");

console.log(person instanceof Person);    // true
console.log(person.name);                 // "Nicholas"
```

由于proxies是透明的，所以所有的返回对象作为`Person`的一个实例都拥有相同的可被观察到的特征，这允许你创建一个类型安全的对象的多个实例，而仅仅调用`createTypeSafeObject()`一次。