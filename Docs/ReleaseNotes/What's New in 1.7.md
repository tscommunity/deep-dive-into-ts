# `async`/`await` support in ES6 targets (Node v4+)
# ES6 为编译目标（需要 Node v4+ 版本）时的 `async`/`await` 支持

TypeScript now supports asynchronous functions for engines that have native support for ES6 generators, e.g. Node v4 and above.
Asynchronous functions are prefixed with the `async` keyword;
`await` suspends the execution until an asynchronous function return promise is fulfilled and unwraps the value from the `Promise` returned.

TypeScript 现在支持对 ES6 生成器具有本地支持的引擎的异步函数，例如 Node v4 及更高版本。异步函数以 `async` 关键字作为前缀；`await` 挂起执行直到异步函数返回 promise 执行完并从返回的 `Promise` 中将值拆包出来。

##### Example
##### 例子

In the following example, each input element will be printed out one at a time with a 400ms delay:

在以下例子中，每个输入元素将会被间隔400毫秒一次一个地打印：

```ts
"use strict";

// printDelayed is a 'Promise<void>'
async function printDelayed(elements: string[]) {
    for (const element of elements) {
        await delay(400);
        console.log(element);
    }
}

async function delay(milliseconds: number) {
    return new Promise<void>(resolve => {
        setTimeout(resolve, milliseconds);
    });
}

printDelayed(["Hello", "beautiful", "asynchronous", "world"]).then(() => {
    console.log();
    console.log("Printed every element!");
});
```

For more information see [Async Functions](http://blogs.msdn.com/b/typescript/archive/2015/11/03/what-about-async-await.aspx) blog post.

更多信息请看 [Async Functions](http://blogs.msdn.com/b/typescript/archive/2015/11/03/what-about-async-await.aspx) 博客文章。

# Support for `--target ES6` with `--module`
# 支持带 `--module` 选项的 `--target ES6`

TypeScript 1.7 adds `ES6` to the list of options available for the `--module` flag and allows you to specify the module output when targeting `ES6`.
This provides more flexibility to target exactly the features you want in specific runtimes.

TypeScript 1.7 为 `--module` 标志的可用选项列表添加了 `ES6` 并且当编译目标为 `ES6` 时允许你指定模块输出。这为在特定的运行时里指定你想要的特性提供了更多的灵活性。

##### Example
##### 例子

```json
{
    "compilerOptions": {
        "module": "amd",
        "target": "es6"
    }
}
```

# `this`-typing
# `this` 类型

It is a common pattern to return the current object (i.e. `this`) from a method to create [fluent-style APIs](https://en.wikipedia.org/wiki/Fluent_interface).
For instance, consider the following `BasicCalculator` module:

这是一种常见的模式——从一个方法返回当前对象以创建 [fluent-style APIs](https://en.wikipedia.org/wiki/Fluent_interface)。例如，考虑以下 `BasicCalculator` 模块：

```ts
export default class BasicCalculator {
    public constructor(protected value: number = 0) { }

    public currentValue(): number {
        return this.value;
    }

    public add(operand: number) {
        this.value += operand;
        return this;
    }

    public subtract(operand: number) {
        this.value -= operand;
        return this;
    }

    public multiply(operand: number) {
        this.value *= operand;
        return this;
    }

    public divide(operand: number) {
        this.value /= operand;
        return this;
    }
}
```

A user could express `2 * 5 + 1` as

可以将 `2 * 5 + 1` 表示为

```ts
import calc from "./BasicCalculator";

let v = new calc(2)
    .multiply(5)
    .add(1)
    .currentValue();
```

This often opens up very elegant ways of writing code; however, there was a problem for classes that wanted to extend from `BasicCalculator`.
Imagine a user wanted to start writing a `ScientificCalculator`:

这通常会开启非常优雅的编码方式；然而，对于想要从 `BasicCalculator` 进行扩展的类存在一个问题。想象一下想要编写一个 `ScientificCalculator`：

```ts
import BasicCalculator from "./BasicCalculator";

export default class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }

    public square() {
        this.value = this.value ** 2;
        return this;
    }

    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
}
```

Because TypeScript used to infer the type `BasicCalculator` for each method in `BasicCalculator` that returned `this`, the type system would forget that it had `ScientificCalculator` whenever using a `BasicCalculator` method.

因为 TypeScript 过去为 `BasicCalculator` 中返回 `this` 的每个方法推导出类型 `BasicCalculator`，每当使用 `BasicCalculator` 中的方法时类型系统将会忘记类实例实际上有 `ScientificCalculator` 这个类型。

For instance:

例如：

```ts
import calc from "./ScientificCalculator";

let v = new calc(0.5)
    .square()
    .divide(2)
    .sin()    // Error: 'BasicCalculator' has no 'sin' method.
    .currentValue();
```

This is no longer the case - TypeScript now infers `this` to have a special type called `this` whenever inside an instance method of a class.
The `this` type is written as so, and basically means "the type of the left side of the dot in a method call".

现在不在是这样的情形了——无论何时，在类的实例方法中， TypeScript 现在将 `this` 推导为拥有一个叫做 `this` 的特殊类型。这个 `this` 是这样写的，基本意思是“方法调用表达式中点号左侧的类型”。

The `this` type is also useful with intersection types in describing libraries (e.g. Ember.js) that use mixin-style patterns to describe inheritance:

这个 `this` 类型对于交叉类型在描述使用混入风格模式来描述继承关系的类库（例如，Ember.js）时也是很有用的：

```ts
interface MyType {
    extend<T>(other: T): this & T;
}
```

# ES7 exponentiation operator
# ES7 幂运算符

TypeScript 1.7 supports upcoming [ES7/ES2016 exponentiation operators](https://github.com/rwaldron/exponentiation-operator): `**` and `**=`.
The operators will be transformed in the output to ES3/ES5 using `Math.pow`.

TypeScript 1.7 支持即将到来的 [ES7/ES2016 幂运算符](https://github.com/rwaldron/exponentiation-operator)：`**` 和 `**=`。当编译目标设置为 ES3/ES5 时，这个运算符将被 `Math.pow` 替代转换到输出中。

##### Example

```ts
var x = 2 ** 3;
var y = 10;
y **= 2;
var z =  -(4 ** 3);
```

Will generate the following JavaScript output:

将会生成如下 JavaScript 输出：

```js
var x = Math.pow(2, 3);
var y = 10;
y = Math.pow(y, 2);
var z = -(Math.pow(4, 3));
```

# Improved checking for destructuring object literal
# 改进解构对象字面量检查

TypeScript 1.7 makes checking of destructuring patterns with an object literal or array literal initializers less rigid and more intuitive.

TypeScript 1.7 使得对象字面量或数组字面量的初始化器的解构模式的检查不那么刻板，更直观。

When an object literal is contextually typed by the implied type of an object binding pattern:

当对象字面量由对象绑定模式的隐含类型进行上下文式地类型化时：

* Properties with default values in the object binding pattern become optional in the object literal.

  对象绑定模式中具有默认值的属性在对象字面量中变为可选。

* Properties in the object binding pattern that have no match in the object literal are required to have a default value in the object binding pattern and are automatically added to the object literal type.

  对象绑定模式中未在对象字面量中匹配的属性都要在对象绑定模式中有一个默认值且被自动添加到该对象字面量的类型中。

* Properties in the object literal that have no match in the object binding pattern are an error.

  未在对象绑定模式中匹配的对象字面量中的属性都是错误的。

When an array literal is contextually typed by the implied type of an array binding pattern:

当数组字面量由数组绑定模式的隐含类型进行上下文式的类型化时：

* Elements in the array binding pattern that have no match in the array literal are required to have a default value in the array binding pattern and are automatically added to the array literal type.

  数组绑定模式中未在数组字面量中匹配的元素都要在数组绑定模式中有一个默认值且被自动的添加到该数组字面量类型中。

##### Example
##### 例子

```ts
// “f({x = 0, y = 0})”这种称之为对象绑定模式。
function f({x = 0, y = 0}){}

// Type of f1 is (arg?: { x?: number, y?: number }) => void
// “{ x, y = 0 }”中的“x”和“y”即为对象绑定模式中具有默认值的属性，故其在对象字面量“{}”中是可选的。
function f1({ x = 0, y = 0 } = {}) { }

// And can be called as:
f1();
f1({});
f1({ x: 1 });
f1({ y: 1 });
f1({ x: 1, y: 1 });

// Type of f2 is (arg?: (x: number, y?: number)) => void
// 对象绑定模式“{ x, y = 0 }”中未在对象字面量“{ x: 0 }”中匹配的属性（此例子的“y”）都要在对象绑定模式中有一个默认值“y = 0”且被自动添加到该对象字面量“{ x: 0 }”的类型中。
function f2({ x, y = 0 } = { x: 0 }) { }

f2();

// Error, x not optional
// 错误，x 不是可选的。
f2({});       

f2({ x: 1 });

// Error, x not optional
// 错误，x 不是可选的。
f2({ y: 1 });  

f2({ x: 1, y: 1 });
```

# Support for decorators when targeting ES3
# 支持编译目标为 ES3 的装饰器

Decorators are now allowed when targeting ES3.
TypeScript 1.7 removes the ES5-specific use of `reduceRight` from the `__decorate` helper.
The changes also inline calls `Object.getOwnPropertyDescriptor` and `Object.defineProperty` in a backwards-compatible fashion that allows for a to clean up the emit for ES5 and later by removing various repetitive calls to the aforementioned `Object` methods.

现在允许在编译目标设置为 ES3 时使用装饰器。TypeScript 1.7 从 `__decorate` 帮助器中移除了 ES5 专用的 `reduceRight`。此变更也以向后兼容的方式——允许清理针对 ES5 的生成并稍后通过移除对上述 `Object` 方法的各种重复调用，内联了 `Object.getOwnPropertyDescriptor` 和 `Object.defineProperty` 的调用。