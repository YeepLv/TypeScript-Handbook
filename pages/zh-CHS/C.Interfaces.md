# Introduction

TypeScript的核心原则之一，是类型检查会集中关注数据的“结构”（shape）。这一行为有时被称作“鸭子类型”（duck typing）或“结构子类型化”（structural subtyping）。在TypeScript中，接口起到了为这些数据类型命名的作用，同时接口也是定义你代码之间的关系，或你的代码和其他项目代码之间关系的有效方法。

# 我们的第一个接口

让我们来看看下面这个简单的例子，来了解接口是如何工作的：

```ts
function printLabel(labelledObj: {label: string}) {
    console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

类型检查器会检查'printLabel'的调用。其中'printLabel'方法有一个参数，我们需要给这个参数传入一个带有名为'label'的字符串类型属性的对象。注意我们传入的这个对象实际上不只有'label'属性，但编译器只检查那些指定的属性，查看它们的类型是否相符。

让我们重写上面的例子，这次我们将使用接口来描述参数的需求，即传入的对象要有字符串类型的label属性。

```ts
interface LabelledValue {
    label: string;
}

function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
}

var myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

我们可以用这个名为'LabelledValue'的接口来描述我们前面例子中的需求。它仍旧表示需要有一个名为'label'的字符串属性。值得注意的是，与其他编程语言不同，我们不需要明确地说 传给'printLabel'的对象实现了这个接口(即鸭子类型)。这里只关注数据的“结构”。只要我们传给函数的对象满足指定的需求，那这个对象就是合法的。

必须指出的是，类型检查器并不要求这些属性遵循一定的顺序。只要接口要求的属性存在，并符合类型即可。

# 可选属性

接口中的属性并不都是必要的。在遵循一定的条件时，有些属性甚至可以不存在。在创建“option bags”这样的模式时，用户传给函数作为参数的对象，往往只包含部分属性在里面。在这种情况下，可选属性就显得很有用了。

下面是这种模式的一个例子：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    var newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

var mySquare = createSquare({color: "black"});
```

除了在声明可选属性时需要加上'?'作为标识以外，带有可选属性的接口的写法与其他接口相似。

使用可选属性的优势在于，我们可以在描述可能存在的属性的同时，捕捉那些我们不希望存在的属性。举例来说，如果我们错误地拼写了传给'createSquare'方法的属性名的话，就会有一条错误信息提示我们：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    var newSquare = {color: "white", area: 100};
    if (config.color) {
        // 错误，类型 'SquareConfig' 没有名为 'collor' 的属性
        newSquare.color = config.collor;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

var mySquare = createSquare({color: "black"});
```

# 函数类型

接口可以描述各式各样的JavaScript对象。然而我们除了用接口来描述一个对象的属性以外，也可以用它来描述函数类型。

我们需要给接口一个调用标记来描述函数类型。它看起来就像是只有参数列表和返回类型的函数的定义。这样就可以枚举函数的所有参数的名称和类型。

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
```

定义好了这个接口以后，我们就可以像使用其他接口一样使用这个函数类型接口。下面展示了我们要如何创建一个函数类型变量并给它赋值一个同样类型的函数值。

```ts
var mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    var result = source.search(subString);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

对于函数类型接口的类型检查，我们允许参数名称与接口中的参数名称不一致。就是说上面的例子也可以这么写：

```ts
var mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    var result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

在对函数的参数进行类型检查时，只会检查在接口对应位置上的参数的类型与其是否一致，无名称无关。
如果你在函数声明时没有指定类型（例如上面的src和sub），那么TypeScript会根据你当前键入的上下文信息推导出此函数符合SearchFunc接口，从而自动认为src和sub的类型是string，
而我们也会对函数表达式的返回类型进行检查（这里是true和false）。
如果这里函数返回的是数字或字符串，那类型检查器就会警告我们返回的类型与SearchFunc接口不相符。

```ts
var mySearch: SearchFunc;
mySearch = function(src, sub) {
    // 这里的src和sub是string，而不是any
    var result = src.search(sub);
    if (result == -1) {
        return false;
    }
    else {
        return true;
    }
}
```

# 数组类型

我们也可以用接口来描述数组类型，它的声明方式与函数类型相似。数组类型会有一个'index'类型，我们用它来表示数组索引（数组下标）的类型。同样我们也需要索引所对应的返回值的类型。

```ts
interface StringArray {
    [index: number]: string;
}

var myArray: StringArray;
myArray = ["Bob", "Fred"];
```

TypeScript支持两种索引类型：string和number。同时使用这两种类型的索引也是允许的，只要我们保证number类型的索引所对应的返回类型是string索引对应的返回类型的子类型。

虽然索引标识是描述数组和字典类型的数据的好方法，它同时也会强迫其他所有属性都与索引的返回类型相同。
这是因为`obj.property`也包含一个`obj["property"]`这样的字符串索引。
在下面的例子中，'name'属性的类型不符合索引的返回类型，这会导致类型检查抛出错误：

```ts
interface NumberDictionary {
    [index: string]: number;
    length: number;    // 正确，length是number类型
    name: string;      // 错误，name的返回类型string不是number的子类型。
}
```

# 类

## 实现接口

在C#和Java中，让一个类符合某种特定的约定，是一种很常见的接口的使用方式。在TypeScript中我们也可以这样使用接口。

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

我们可以在一个接口中描述一个类需要实现的方法。就像下面的例子中的`setTime`方法：

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口只能描述类的公共部分，而不关注私有部分。这种机制不允许我们通过接口来检查一个类的实例的私有部分。

## 类的静态和实例的差异

当使用类和接口时，我们应该要记得一个类有静态部分和实例特有的部分。你可能注意到了，如果创建一个带有构造函数标记的接口，并尝试创建一个类来实现这个接口的话，我们会收到错误：

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

这是因为当一个类实现一个接口时，只有实例的部分会被进行检查。构造函数属于静态的部分，它并不在检查的范围之内。

对应地，我们应该直接检查类的静态部分。
在这个例子中，我们声明了两个接口，`ClockConstructor` 接口用于构造，`ClockInterface`接口用于实例的方法。
然后，我们为方便调用，定义了一个`createClock`方法用于创建实例并将参数传递给实例。

```ts
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

var digital = createClock(DigitalClock, 12, 17);
var analog = createClock(AnalogClock, 7, 32);
```

Because `createClock`'s first parameter is of type `ClockConstructor`, in `createClock(AnalogClock, 12, 17)`, it checks that `AnalogClock` has the correct constructor signature.
因为`createClock`的第一个参数是接口类型`ClockConstructor`，在方法`createClock(AnalogClock, 12, 17)`的调用中，将检查`AnalogClock`的构造参数是否符合接口签名。

# 扩展接口

同类一样，接口也可以相互扩展。扩展机制负责将一个接口中的成员拷贝到另一个接口中，这意味着我们可以根据自己的意愿把接口分离成可重用的组件。

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

个接口可以扩展多个接口，从而成为多个接口的组合。

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

var square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

# 混合类型

就像我们之前提到过的，接口可以描述现实中JavaScript所能表现的丰富的数据类型。由于JavaScript动态、灵活的特性，我们有时可能会碰到需要综合使用前面描述的接口的使用方法，来处理一个对象的情景。

举个例子，一个对象既是一个函数，同时还包含一个方法，和一个额外的属性。

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    var counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

var c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

当同第三方JavaScript代码进行交互时，我们可能就需要使用上面的模式，来完整地描述一个数据的类型和结构。

# 接口扩展类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现。
就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。
接口同样会继承到类的private和protected成员。
这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

这是很有用的，当你有一个很深层次的继承，但是只想你的代码只是针对拥有特定属性的子类起作用的时候。子类除了继承自基类外与基类没有任何联系。
例：

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control {
    select() { }
}

class TextBox extends Control {
    select() { }
}

class Image extends Control {
}

class Location {
    select() { }
}
```
在上面的例子里，`SelectableControl`包含了`Control`的所有成员，包括私有成员`state`。
因为`state`是私有成员，所以只能够是`Control`的子类们才能实现`SelectableControl`接口。
因为只有`Control`的子类才能够拥有一个声明于`Control`的私有成员`state`，这对私有成员的兼容性是必需的。

在`Control`类内部，是允许通过`SelectableControl`的实例来访问私有成员`state`的。
实际上，`SelectableControl`就像`Control`一样，并拥有一个`select`方法。
`Button`和`TextBox`类是`SelectableControl`的子类（类为它们都继承自`Control`并有`select`方法），但`Image`和`Location`类并不是这样的。

# 感谢翻译
- oyyd      https://github.com/oyyd/typescript-handbook-zh
- 编写人生  https://github.com/MyErpSoft/TypeScript-Handbook
- zhongsp   https://github.com/zhongsp/TypeScript
