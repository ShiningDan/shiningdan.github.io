---
title: TypeScript学习笔记
date: 2018-04-21 17:36:09
categories: coding
tags:
  - TypeScript
---

本文是我学习 TypeScript 的学习笔记

<!--more-->

## TypeScript 个人总结

TypeScript 的类型检查是基于结构的检查，而不是基于声明的检查。所以在 TypeScript 检查的时候，即使是被检查的对象在声明上面不满足要求，但是在结构上满足要求需要的结构，就可以通过检查

具体结构类型的检查和声明类型的检查，可以参考 [Nominal & Structural Typing | Flow](https://flow.org/en/docs/lang/nominal-structural/) 这篇文章

```
Object oriented languages tend to be more nominally typed (C++, Java, Swift), while functional languages tend to be more structurally typed (OCaml, Haskell, Elm)
```

### 基础类型

* boolean
* number
* string
* 数组：number[] 或者 Array<number>
* 元组，用来表示一个已知元素数量和类型的数组，各元素类型不必相同：[string, number]
* 枚举：enum Color {Red, Green, Blue}
* 在编程阶段还不清楚类型的变量指定一个类型。 这些值可能来自于动态的内容，它允许你在编译时可选择地包含或移除类型检查：any
* Object：与 any 的不同之处在于，`Object` 类型的变量只是允许你给它赋任意值 - 但是却不能够在它上面调用任意的方法，即便它真的有这些方法
* void：void类型像是与any类型相反，它表示没有任何类型。 
* undefined/null：undefined和null两者各自有自己的类型分别叫做undefined和null。 和 void相似，它们的本身的类型用处不是很大。默认情况下null和undefined是所有类型的子类型。 就是说你可以把 null和undefined赋值给number类型的变量。
* never：never类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型； 变量也可能是 never类型，当它们被永不为真的类型保护所约束时。

## 变量声明

const是对let的一个增强，它能阻止对一个变量再次赋值。

var 使用的是作用域或函数作用域

let 声明一个变量，它使用的是词法作用域或块作用域。

const 拥有与 let相同的作用域规则，但是不能对它们重新赋值。

## 接口

TypeScript的核心原则之一是对值所具有的**结构进行类型检查**。 它有时被称做“鸭式辨型法”或“结构性子类型化”。

**接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约**，不是传统意义上接口的作用

### 可选属性

```
interface SquareConfig {
  color?: string;
  width?: number;
}
```

如果 SquareConfig带有上面定义的类型的color和width属性，并且还会带有任意数量的其它属性，那么我们可以这样定义它：

```
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

### 只读属性

```
interface Point {
    readonly x: number;
    readonly y: number;
}
```

### 函数类型

它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

### 可索引的类型

我们也可以描述那些能够“通过索引得到”的类型，比如 `a[10]` 或 `ageMap["daniel"]`。

```
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

共有支持两种索引签名：***字符串和数字***。

可以同时使用两种类型的索引，但是数字索引的返回值必须是字符串索引返回值类型的子类型。 这是因为当使用 number来索引时，JavaScript会将它转换成string然后再去索引对象。 也就是说用 100（一个number）去索引等同于使用"100"（一个string）去索引，因此两者需要保持一致。

### 类类型

#### 实现接口

TypeScript也能够用接口来明确的强制一个类去符合某种契约。

```
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

接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员。


### 继承接口

和类一样，接口也可以相互继承。

```
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

### 混合类型

一个对象可以同时做为函数和对象使用，并带有额外的属性。

```
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

### 接口继承类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 

## 类

派生类包含了一个构造函数，它 必须调用 `super()`，它会执行基类的构造函数。 而且，在构造函数里访问 `this` 的属性之前，我们 一定要调用 `super()`。 这个是TypeScript强制执行的一条重要规则。

### 公共，私有与受保护的修饰符

#### 默认为 public

 在TypeScript里，成员都默认为 public。

#### private

当成员被标记成 private时，它就不能在声明它的类的外部访问。

```
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // 错误: Animal 与 Employee 不兼容.
```

虽然 interface 是结构性检查，但是 class 并不是结构性检查，而是生命性检查

#### protected

protected修饰符与 private修饰符的行为很相似，但有一点不同， protected成员在派生类中仍然可以访问。

构造函数也可以被标记成 protected。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。

```
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee 能够继承 Person
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // 错误: 'Person' 的构造函数是被保护的.
```

#### readonly修饰符

你可以使用 readonly关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。

```
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.
```

#### 参数属性

```
class Animal {
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

注意看我们是如何舍弃了 theName，仅在构造函数里使用 private name: string参数来创建和初始化 name成员。 我们把声明和赋值合并至一处。

参数属性通过给构造函数参数添加一个访问限定符来声明。 使用 private限定一个参数属性会声明并初始化一个私有成员；对于 public和 protected来说也是一样。

### 存取器

TypeScript支持通过getters/setters来截取对对象成员的访问。

```
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
    employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

### 静态属性

我们也可以创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。

```
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### 抽象类

抽象类做为其它派生类的基类使用。 它们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。 abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。

```
抽象类做为其它派生类的基类使用。 它们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。 abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。v
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。 抽象方法的语法与接口方法相似。 两者都是定义方法签名但不包含方法体。 然而，抽象方法必须包含 abstract关键字并且可以包含访问修饰符。

## 函数

### 函数类型

```
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

#### 书写完整函数类型

```
let myAdd: (x: number, y: number) => number =
    function(x: number, y: number): number { return x + y; };
```

### 可选参数和默认参数

在TypeScript里我们可以在参数名旁使用 ?实现可选参数的功能。
 
```
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

可选参数必须跟在必须参数后面。 如果上例我们想让first name是可选的，那么就必须调整它们的位置，把first name放在后面。

```
function buildName(firstName: string, lastName?: string) {
    // ...
}
```
和
```
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```

共享同样的类型 `(firstName: string, lastName?: string) => string`。

### 剩余参数

```
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

### this

this.suits[pickedSuit]的类型依旧为any。 这是因为 this来自对象字面量里的函数表达式。 修改的方法是，提供一个显式的 this参数。 

```
 let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: The function now explicitly specifies that its callee must be of type Deck
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}
```

### 重载

JavaScript 的重载通过在函数中对输入参数的类型进行判断来实现。

方法是为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用。 下面我们来重载 pickCard函数。

```
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}
```

## 泛型

我们使用 `any`类型来定义函数：

```
function identity(arg: any): any {
    return arg;
}
```

使用 `any` 类型会导致这个函数可以接收任何类型的 `arg` 参数，这样就丢失了一些信息：传入的类型与返回的类型应该是相同的。

这里，我们使用了 **类型变量**，它是一种特殊的变量，只用于表示类型而不是值。

```
function identity<T>(arg: T): T {
    return arg;
}
```

我们定义了泛型函数后，可以用两种方法使用。 第一种是，传入所有的参数，包含类型参数：

```
let output = identity<string>("myString");  // type of output will be 'string'
```

第二种方法更普遍。利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定T的类型：

```
let output = identity("myString");  // type of output will be 'string'
```

### 使用泛型变量

如果我们想同时打印出arg的长度。 我们很可能会这样做：

```
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

现在假设我们想操作T类型的数组而不直接是T。由于我们操作的是数组，所以.length属性是应该存在的。 我们可以像创建其它数组一样创建这个数组：

```
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

### 泛型接口

```
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

### 泛型类

```
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 泛型约束

我们定义一个接口来描述约束条件。 创建一个包含 `.length` 属性的接口，使用这个接口和 `extends` 关键字来实现约束：

```
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

## 枚举

```
enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

## 类型兼容性

TypeScript里的类型兼容性是基于结构子类型的。 结构类型是一种只使用其成员来描述类型的方式。 它正好与名义（nominal）类型形成对比。

### 比较两个函数

```
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

要查看 `x` 是否能赋值给 `y`，首先看它们的参数列表。 `x` 的每个参数必须能在y里找到对应类型的参数。


下面来看看如何处理返回值类型，创建两个仅是返回值类型不同的函数：

```
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error because x() lacks a location property
```

类型系统强制源函数的返回值类型必须是目标函数返回值类型的子类型。


### 枚举

枚举类型与数字类型兼容，并且数字类型与枚举类型兼容。不同枚举类型之间是不兼容的。

```
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green;  //error
```

### 类

类与对象字面量和接口差不多，但有一点不同：类有静态部分和实例部分的类型。 比较两个类类型的对象时，只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。


```
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  //OK
s = a;  //OK
```

## 高级类型

`Person & Serializable & Loggable` 同时是 Person 和 Serializable 和 Loggable。 就是说这个类型的对象同时拥有了这三种类型的成员。

### 联合类型（Union Types）

一个代码库希望传入 number或 string类型的参数。

```
function padLeft(value: string, padding: string | number) {
    // ...
}
```

 JavaScript里常用来区分2个可能值的方法是检查成员是否存在。为了让这段代码工作，我们要使用类型断言：
 
 ```
 let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
 ```
 
 ### 用户自定义的类型保护
 
```
 function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

pet is Fish就是类型谓词。 谓词为 parameterName is Type这种形式， parameterName必须是来自于当前函数签名里的一个参数名。

还可以使用 `typeof` 进行类型保护

```
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

还可以使用 `instanceof`

```
if (padder instanceof SpaceRepeatingPadder) {
    padder; // 类型细化为'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 类型细化为'StringPadder'
}
```

### 类型别名

类型别名会给一个类型起个新名字。

```
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
```

### 字符串字面量类型

字符串字面量类型允许你指定字符串必须的固定值。 

```
type Easing = "ease-in" | "ease-out" | "ease-in-out";
```

### 数字字面量类型

```
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

### 可辨识联合（Discriminated Unions）

你可以合并单例类型，联合类型，类型保护和类型别名来创建一个叫做 可辨识联合的高级模式，它也称做 标签联合或 代数数据类型。

```
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
type Shape = Square | Rectangle | Circle;
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

## 模块

### 导出

任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加export关键字来导出。

```
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

### 默认导出

```
declare let $: JQuery;
export default $;

import $ from "JQuery";

$("button.continue").html( "Next Step..." );
```

### 导入

```
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
import * as validator from "./ZipCodeValidator";
let myValidator = new ZCV();
```
 
## 命名空间

“内部模块”现在叫做“命名空间”。 另外，任何使用 module关键字来声明一个内部模块的地方都应该使用namespace关键字来替换。

```
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();
```

## tsconfig.json

tsconfig.json文件中指定了用来编译这个项目的根文件和编译选项。

具体项目配置的介绍可以参考 [tsconfig.json
](https://www.tslang.cn/docs/handbook/tsconfig-json.html)

一般常用的选项有：

### compilerOptions

具体的选项，可以在 [编译选项
](https://www.tslang.cn/docs/handbook/compiler-options.html) 中查看

### files

"files"指定一个包含相对或绝对文件路径的列表。 

### include & exclude

指定文件的路径和不搜索的路径

"include"和"exclude"属性指定一个文件glob匹配模式列表。 支持的glob通配符有：

* `*` 匹配0或多个字符（不包括目录分隔符）
* `?` 匹配一个任意字符（不包括目录分隔符）
* `**/` 递归匹配任意子目录

## 声明

用 ts 写的模块在发布的时候仍然是用 js 发布，这就导致一个问题：ts 那么多类型数据都没了，所以需要一个 d.ts 文件来标记某个 js 库里面对象的类型

一组 declare 的合集组成了 .d.ts 文件。同 .ts 的区别在于 .d.ts 不会生成额外的代码（因为 declare 语句不会生成代码）。

### 引用第三方库

在我们引用第三方库的时候，我们需要引用第三方库的声明文件，如果第三方库没有声明文件，则我们需要使用 `declare` 来主动为第三方库声明接口。[声明文件](https://ts.xcatliu.com/basics/declaration-files.html)

### 如何编写 d.ts 文件

[如何编写一个d.ts文件](https://segmentfault.com/a/1190000009247663)

[声明文件](https://www.tslang.cn/docs/handbook/declaration-files/introduction.html)
