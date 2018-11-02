# Type parameters as constraints
# 类型参数作为约束

With TypeScript 1.8 it becomes possible for a type parameter constraint to reference type parameters from the same type parameter list.
Previously this was an error.
This capability is usually referred to as [F-Bounded Polymorphism](https://en.wikipedia.org/wiki/Bounded_quantification#F-bounded_quantification).

TypeScript 1.8 让类型参数约束引用同一个类型参数列表中的类型参数成为可能。在早先版本中这是错误的。这种能力通常被称作 [F-Bounded Polymorphism](https://en.wikipedia.org/wiki/Bounded_quantification#F-bounded_quantification)。

##### Example
##### 示例

```ts
function assign<T extends U, U>(target: T, source: U): T {
    for (let id in source) {
        target[id] = source[id];
    }
    return target;
}

let x = { a: 1, b: 2, c: 3, d: 4 };
assign(x, { b: 10, d: 20 });
assign(x, { e: 0 });  // Error
```

# Control flow analysis errors
# 控制流分析错误

TypeScript 1.8 introduces control flow analysis to help catch common errors that users tend to run into.
Read on to get more details, and check out these errors in action:

TypeScript 1.8 引入了控制流分析以帮助捕获用户往往会遇到的常规错误。继续阅读以获取更多详细信息并马上检查这些错误：

![cfa](https://cloud.githubusercontent.com/assets/8052307/5210657/c5ae0f28-7585-11e4-97d8-86169ef2a160.gif)

### Unreachable code
### 不可达代码

Statements guaranteed to not be executed at run time are now correctly flagged as unreachable code errors.
For instance, statements following unconditional `return`, `throw`, `break` or `continue` statements are considered unreachable.
Use `--allowUnreachableCode` to disable unreachable code detection and reporting.

在运行时一定不会被执行的语句现在能够被正确的标记为不可达代码错误。例如，那些紧随非条件性的 `return`, `throw`, `break` 或 `continue` 都被认为是不可达的。使用 `--allowUnreachableCode` 来禁用不可达代码检测和报告。

##### Example
##### 示例

Here's a simple example of an unreachable code error:

这是个不可达代码错误的简单例子：

```ts
function f(x) {
    if (x) {
       return true;
    }
    else {
       return false;
    }

    x = 0; // Error: Unreachable code detected.
}
```

A more common error that this feature catches is adding a newline after a `return` statement:

这个特性捕获的另一个常见错误是在 `return` 语句后面添加新行：

```ts
function f() {
    // Automatic Semicolon Insertion triggered at newline
    // 触发自动为新行插入分号。
    return            
    {
        x: "string"   // Error: Unreachable code detected.
    }
}
```

Since JavaScript automatically terminates the `return` statement at the end of the line, the object literal becomes a block.

由于 JavaScript 会自动在行末终止 `return` 语句，这个对象字面量变成了一个块。

### Unused labels
### 未使用的标签

Unused labels are also flagged.
Just like unreachable code checks, these are turned on by default;
use `--allowUnusedLabels` to stop reporting these errors.

未使用的标签也会被标记。和不可达代码检查一样，这个也是默认开启的；使用 `--allowUnusedLabels` 来停止报告这些错误。

##### Example
##### 示例

```ts
loop: while (x > 0) {  // Error: Unused label.
    x++;
}
```

### Implicit returns
### 隐式返回

Functions with code paths that do not return a value in JS implicitly return `undefined`.
These can now be flagged by the compiler as implicit returns.
The check is turned *off* by default; use `--noImplicitReturns` to turn it on.

在 JS 中带无返回值代码路径的函数将隐式返回 `undefined`。这现在能够被编译器标记为隐式返回。这个检查默认被*关闭*；使用 `--noImplicitReturns` 来开启它。

##### Example
##### 示例

```ts
// Error: Not all code paths return a value.
// 不是所有的代码路径都返回了值。
function f(x) { 
    if (x) {
        return false;
    }

    // implicitly returns `undefined`
}
```

### Case clause fall-throughs
### case 子句贯穿

TypeScript can reports errors for fall-through cases in switch statement where the case clause is non-empty.
This check is turned *off* by default, and can be enabled using `--noFallthroughCasesInSwitch`.

TypeScript 能够报告 switch 语句中非空的 case 子句贯穿错误。这个检查默认被关闭可以使用 `--noFallthroughCasesInSwitch` 来启用。

##### Example
##### 示例

With `--noFallthroughCasesInSwitch`, this example will trigger an error:

使用 `--noFallthroughCasesInSwitch` 这个例子将会触发一个错误：

```ts
switch (x % 2) {
    case 0: // Error: Fallthrough case in switch.
        console.log("even");

    case 1:
        console.log("odd");
        break;
}
```

However, in the following example, no error will be reported because the fall-through case is empty:

然而，如下例子所示，因为贯穿子句是空的，所以将不会报告错误：

```ts
switch (x % 3) {
    case 0:
    case 1:
        console.log("Acceptable");
        break;

    case 2:
        console.log("This is *two much*!");
        break;
}
```

# Stateless Function Components in React
# React 中的无状态函数

TypeScript now supports [Stateless Function components](https://reactjs.org/docs/components-and-props.html#functional-and-class-components).
These are lightweight components that easily compose other components:

TypeScript 现在支持 [Stateless Function components](https://reactjs.org/docs/components-and-props.html#functional-and-class-components)（无状态函数组件）。这是轻量级的组件能容易地组合其它组件：

```ts
// Use parameter destructuring and defaults for easy definition of 'props' type
const Greeter = ({name = 'world'}) => <div>Hello, {name}!</div>;

// Properties get validated
let example = <Greeter name='TypeScript 1.8' />;
```

For this feature and simplified props, be sure to be use the [latest version of react.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react).

对于此特性和简化的 props，请确保使用[最新版的 react.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/react)。

# Simplified `props` type management in React
# React 中简化的 `props` 类型管理

In TypeScript 1.8 with the latest version of react.d.ts (see above), we've also greatly simplified the declaration of `props` types.

使用 TypeScript 1.8 和最新版本的 react.d.ts（见上述），我们极大地简化了 `props` 类型的声明。

Specifically:

特别地：

* You no longer need to either explicitly declare `ref` and `key` or `extend React.Props`

  你不再必须显式地声明 `ref` 和 `key` 或者 `extend React.Props`

* The `ref` and `key` properties will appear with correct types on all components

  在所有组件中 `ref` 和 `key` 属性将会以正确的类型出现

* The `ref` property is correctly disallowed on instances of Stateless Function components

   `ref` 属性已正确地在无状态函数组件上被禁止了

# Augmenting global/module scope from modules
# 从模块扩充全局/模块域

Users can now declare any augmentations that they want to make, or that any other consumers already have made, to an existing module.
Module augmentations look like plain old ambient module declarations (i.e. the `declare module "foo" { }` syntax), and are directly nested either your own modules, or in another top level ambient external module.

用户现在可以向已有模块声明他们想要进行的任何扩充或者任何其它消费者已经做出的扩充。模块扩充看起来就像纯旧式的周围模块声明（例如，`declare module "foo" { }` 语法），直接内嵌在你自己的模块或者其它顶层周围外部模块。

Furthermore, TypeScript also has the notion of *global* augmentations of the form `declare global { }`.
This allows modules to augment global types such as `Array` if necessary.

此外，TypeScript 还具有 `declare global { }` 形式的*全局*扩充概念。这允许模块在必要时扩充全局类型，如 `Array` 。

The name of a module augmentation is resolved using the same set of rules as module specifiers in `import` and `export` declarations.
The declarations in a module augmentation are merged with any existing declarations the same way they would if they were declared in the same file.

使用与导入和导出声明中的模块描述符相同的规则集来解析模块扩充的名称。模块扩充中的声明与任何现有的声明合并，就好像它们在相同的文件中声明一样。

Neither module augmentations nor global augmentations can add new items to the top level scope - they can only "patch" existing declarations.

无论是模块扩充还是全局扩充都可以添加新项到顶层的域——它们只能“修补”现有的声明。

##### Example
##### 示例

Here `map.ts` can declare that it will internally patch the `Observable` type from `observable.ts` and add the `map` method to it.

此处 `map.ts` 声明它将在内部修补这个来自 `observable.ts` 的 `Observable` 类型，为其添加 `map` 方法。

```ts
// observable.ts
export class Observable<T> {
    // ...
}
```

```ts
// map.ts
import { Observable } from "./observable";

// Create an augmentation for "./observable"
// 为“./observable”创建一个扩充
declare module "./observable" {

    // Augment the 'Observable' class definition with interface merging
    // 使用接口合并来扩充“Observable”类定义
    interface Observable<T> {
        map<U>(proj: (el: T) => U): Observable<U>;
    }

}

Observable.prototype.map = /*...*/;
```

```ts
// consumer.ts
import { Observable } from "./observable";
import "./map";

let o: Observable<number>;
o.map(x => x.toFixed());
```

Similarly, the global scope can be augmented from modules using a `declare global` declarations:

类似地，全局域可以通过使用 `declare global` 声明被从模块中进行扩充：

##### Example
##### 示例

```ts
// Ensure this is treated as a module.
// 确保这个被当作模块。
export {};

declare global {
    interface Array<T> {
        mapToNumbers(): number[];
    }
}

Array.prototype.mapToNumbers = function () { /* ... */ }
```

# String literal types
# 字符串字面量类型

It's not uncommon for an API to expect a specific set of strings for certain values.
For instance, consider a UI library that can move elements across the screen while controlling the ["easing" of the animation.](https://en.wikipedia.org/wiki/Inbetweening)

对于一个 API 而言，期望一个有确定值的特定字符串集合并不罕见。例如，一个 UI 库它可以在屏幕上移动元素并同时控制动画的 [“easing”](https://en.wikipedia.org/wiki/Inbetweening) 缓动效果。

```ts
declare class UIElement {
    animate(options: AnimationOptions): void;
}

interface AnimationOptions {
    deltaX: number;
    deltaY: number;
    easing: string; // Can be "ease-in", "ease-out", "ease-in-out"
}
```

However, this is error prone - there is nothing stopping a user from accidentally misspelling one of the valid easing values:

然而，这很容易出错——没有什么能阻止用户无意地错误拼写了 easing 的有效值：

```ts
// No errors
// 不报错
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: "ease-inout" });
```

With TypeScript 1.8, we've introduced string literal types.
These types are written the same way string literals are, but in type positions.

随着 TypeScript 1.8，我们引入了字符串字面量类型。这些类型的书写方式与字符串字面量的书写方式相同，但是是在类型的位置。

Users can now ensure that the type system will catch such errors.
Here's our new `AnimationOptions` using string literal types:

用户现在可以确保类型相同会捕获这种错误。这是我们新的使用了字符串字面量类型的 `AnimationOptions` ：

```ts
interface AnimationOptions {
    deltaX: number;
    deltaY: number;
    easing: "ease-in" | "ease-out" | "ease-in-out";
}

// Error: Type '"ease-inout"' is not assignable to type '"ease-in" | "ease-out" | "ease-in-out"'
// 错误：类型“ease-inout”不能赋值给“ease-in”|“ease-out”|“ease-in-out”类型
new UIElement().animate({ deltaX: 100, deltaY: 100, easing: "ease-inout" });
```

# Improved union/intersection type inference
# 改进联合/交叉类型推导

TypeScript 1.8 improves type inference involving source and target sides that are both union or intersection types.
For example, when inferring from `string | string[]` to `string | T`, we reduce the types to `string[]` and `T`, thus inferring `string[]` for `T`.

TypeScript 1.8 改进了涉及到源和目标端的类型推导，其中源和目标端是联合类型或交叉类型。例如，当从 `string | string[]` 推导到 `string | T` 时，我们简化类型为 `string[]` 和 `T`，因此为 `T` 推导出类型  `string[]` 。

##### Example
##### 示例

```ts
type Maybe<T> = T | void;

function isDefined<T>(x: Maybe<T>): x is T {
    return x !== undefined && x !== null;
}

function isUndefined<T>(x: Maybe<T>): x is void {
    return x === undefined || x === null;
}

function getOrElse<T>(x: Maybe<T>, defaultValue: T): T {
    return isDefined(x) ? x : defaultValue;
}

function test1(x: Maybe<string>) {
    let x1 = getOrElse(x, "Undefined");         // string
    let x2 = isDefined(x) ? x : "Undefined";    // string
    let x3 = isUndefined(x) ? "Undefined" : x;  // string
}

function test2(x: Maybe<number>) {
    let x1 = getOrElse(x, -1);         // number
    let x2 = isDefined(x) ? x : -1;    // number
    let x3 = isUndefined(x) ? -1 : x;  // number
}
```

# Concatenate `AMD` and `System` modules with `--outFile`
# 使用 `--outFile` 选项串联 `AMD` 和 `System` 模块

Specifying `--outFile` in conjunction with `--module amd` or `--module system` will concatenate all modules in the compilation into a single output file containing multiple module closures.

指定 `--outFile` 选项并同时结合使用 `--module amd` 或 `--module system` 选项将会把编译的所有模块串联输出到一个单独的输出文件中，其中包含有多个模块闭包。

A module name will be computed for each module based on its relative location to `rootDir`.

将为每个模块基于它相对于 `rootDir` 的位置来计算出一个模块名。

##### Example
##### 示例

```ts
// file src/a.ts
import * as B from "./lib/b";
export function createA() {
    return B.createB();
}
```

```ts
// file src/lib/b.ts
export function createB() {
    return { };
}
```

Results in:

```js
define("lib/b", ["require", "exports"], function (require, exports) {
    "use strict";
    function createB() {
        return {};
    }
    exports.createB = createB;
});
define("a", ["require", "exports", "lib/b"], function (require, exports, B) {
    "use strict";
    function createA() {
        return B.createB();
    }
    exports.createA = createA;
});
```

# Support for `default` import interop with SystemJS
# 支持与 SystemJS 互操作的 `default` 导入

Module loaders like SystemJS wrap CommonJS modules and expose then as a `default` ES6 import. This makes it impossible to share the definition files between the SystemJS and CommonJS implementation of the module as the module shape looks different based on the loader.

模块加载器如 SystemJS 封装了 CommonJS 模块并将它作为 ES6 `default` 导入暴露。这使得无法在 SystemJS 和 CommonJS 的模块实现间共享声明文件因为在加载器看来模块的外形是不同的。

Setting the new compiler flag `--allowSyntheticDefaultImports` indicates that the module loader performs some kind of synthetic default import member creation not indicated in the imported .ts or .d.ts. The compiler will infer the existence of a `default` export that has the shape of the entire module itself.

设置新的编译器标志 `--allowSyntheticDefaultImports` 意味着模块加载器执行
未在导入的 .ts 或 .d.ts 中指示的。模块加载器将会推导一个具有整个模块自身形状的 `default` 导出的存在性。

System modules have this flag on by default.

系统模块默认打开该标志。

# Allow captured `let`/`const` in loops
# 允许在循环中捕获 `let`/`const`

Previously an error, now supported in TypeScript 1.8.
`let`/`const` declarations within loops and captured in functions are now emitted to correctly match `let`/`const` freshness semantics.

早先版本中是个错误，现在在 TypeScript 1.8 中已得到支持。在循环中以及在函数中被捕获的`let`/`const` 声明现在能生成正确匹配的 `let`/`const` 新语义。

##### Example
##### 示例

```ts
let list = [];
for (let i = 0; i < 5; i++) {
    list.push(() => i);
}

list.forEach(f => console.log(f()));
```

is compiled to:

被编译为：

```js
var list = [];
var _loop_1 = function(i) {
    list.push(function () { return i; });
};
for (var i = 0; i < 5; i++) {
    _loop_1(i);
}
list.forEach(function (f) { return console.log(f()); });
```

And results in

结果为

```cmd
0
1
2
3
4
```

# Improved checking for `for..in` statements
# 改进 `for..in` 语句的检查

Previously the type of a `for..in` variable is inferred to `any`; that allowed the compiler to ignore invalid uses within the `for..in` body.

在早先的版本中 `for..in` 变量的类型被推导为 `any`，这允许编译器忽略 `for..in` 中的无效使用。

Starting with TypeScript 1.8:

自 TypeScript 1.8 开始：

* The type of a variable declared in a `for..in` statement is implicitly `string`.
  
  在 `for..in` 语句中声明的变量的类型隐式地具有`string`类型。

* When an object with a numeric index signature of type `T` (such as an array) is indexed by a `for..in` variable of a containing `for..in` statement for an object *with* a numeric index signature and *without* a string index signature (again such as an array), the value produced is of type `T`.

  当一个具有数值型索引签名，索引值为 `T` 类型的对象<sup>①</sup>（例如数组）被一个包含在 `for..in` 语句中（该 `for..in` 语句是针对一个*具有*数值型索引签名但*没有*字符串索引签名的对象<sup>②</sup>，例如数组）的变量<sup>③</sup>索引时，索引值的类型是 `T`。

##### Example
##### 示例

```ts
var a: MyObject[]; // 此处的“a”即为上述的“对象①”

// Type of x is implicitly string
// x 隐式地具有类型 string
for (var x in a) { // 此处的“x”即为“变量③”，“a”为“对象②”
    // Type of obj is MyObject
    // obj 地类型是 MyObject
    var obj = a[x]; // 此处的“a”即为上述的“对象①”
}
```

# Modules are now emitted with a `"use strict";` prologue
# 现在生成的模块都会在开始处有 `"use strict";`

Modules were always parsed in strict mode as per ES6, but for non-ES6 targets this was not respected in the generated code. Starting with TypeScript 1.8, emitted modules are always in strict mode. This shouldn't have any visible changes in most code as TS considers most strict mode errors as errors at compile time, but it means that some things which used to silently fail at runtime in your TS code, like assigning to `NaN`, will now loudly fail. You can reference the [MDN Article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) on strict mode for a detailed list of the differences between strict mode and non-strict mode.

依照 ES6，模块总是在严格模式下解析，但对于非 ES6 目标，在生成的代码中不遵守这一点。自 TypeScript 1.8 起，生成的模块总是严格模式的。这在大多数代码中不应该有任何明显的变化因为在编译期 TS 把最严格模式错误认为是错误，但这意味着你 TS 代码中的以前会在运行时静默地失败的东西，例如赋值给 `NaN`，现在将会明显地失败。关于严格模式和非严格模式之间的详细差异，你可以参考 [MDN Article](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) 中的严格模式的有关内容。

# Including `.js` files with `--allowJs`
# 用 `--allowJs` 包括 `.js` 文件

Often there are external source files in your project that may not be authored in TypeScript.
Alternatively, you might be in the middle of converting a JS code base into TS, but still want to bundle all your JS code into a single file with the output of your new TS code.

经常地，在项目中有外部源文件可能未写在 TypeScript 中。或者，你可能正在将 JS 代码库转换到 TS，但是仍想将你的 JS 代码与你的新的 TS 代码的输出捆绑到一个单一的文件。

`.js` files are now allowed as input to `tsc`.
The TypeScript compiler checks the input `.js` files for syntax errors, and emits valid output based on the `--target` and `--module` flags.
The output can be combined with other `.ts` files as well.
Source maps are still generated for `.js` files just like with `.ts` files.

`.js` 文件现在被允许作为 `tsc` 的输入了。TypeScript 编译器检查输入的 `.js` 文件的语法错误，并基于 `--target` 和 `--module` 标志生成有效的输出。输出也可以和其它的 `.ts` 文件合并。仍然为 `.js` 文件生成源映射，就像使用 `.ts` 文件一样。

# Custom JSX factories using `--reactNamespace`
# 使用 `--reactNamespace` 来定制 JSX 工厂

Passing `--reactNamespace <JSX factory Name>` along with `--jsx react` allows for using a different JSX factory from the default `React`.

传递 `--reactNamespace <JSX factory Name>` 与 `--jsx react` 允许使用与默认的 `React` 不同的 JSX 工厂。

The new factory name will be used to call `createElement` and `__spread` functions.

这个新的工厂名称将会被用于调用 `createElement` 和 `__spread` 函数。

##### Example
##### 示例

```ts
import {jsxFactory} from "jsxFactory";

var div = <div>Hello JSX!</div>
```

Compiled with:

使用如下编译：

```shell
tsc --jsx react --reactNamespace jsxFactory --m commonJS
```

Results in:

结果为：

```js
"use strict";
var jsxFactory_1 = require("jsxFactory");
var div = jsxFactory_1.jsxFactory.createElement("div", null, "Hello JSX!");
```

# `this`-based type guards
# 基于 `this` 的类型保护

TypeScript 1.8 extends [user-defined type guard functions](./typescript-1.6.html#user-defined-type-guard-functions) to class and interface methods.

TypeScript 1.8 将 [用户定义的类型保护函数](./typescript-1.6.html#user-defined-type-guard-functions) 扩展到类和接口的方法。

`this is T` is now valid return type annotation for methods in classes and interfaces.
When used in a type narowing position (e.g. `if` statement), the type of the call expression target object would be narrowed to `T`.

现在 `this is T` 是类和接口中的方法的有效返回类型注解。当用在类型收窄位置（例如，`if` 语句），调用表达式目标对象的类型将会收窄为 `T`。

##### Example
##### 示例

```ts
class FileSystemObject {
    isFile(): this is File { return this instanceof File; }
    isDirectory(): this is Directory { return this instanceof Directory;}
    isNetworked(): this is (Networked & this) { return this.networked; }
    constructor(public path: string, private networked: boolean) {}
}

class File extends FileSystemObject {
    constructor(path: string, public content: string) { super(path, false); }
}
class Directory extends FileSystemObject {
    children: FileSystemObject[];
}
interface Networked {
    host: string;
}

let fso: FileSystemObject = new File("foo/bar.txt", "foo");
if (fso.isFile()) {
    fso.content; // fso is File
}
else if (fso.isDirectory()) {
    fso.children; // fso is Directory
}
else if (fso.isNetworked()) {
    fso.host; // fso is networked
}
```

# Official TypeScript NuGet package
# 官方的 TypeScript NuGet 包

Starting with TypeScript 1.8, official NuGet packages are available for the Typescript Compiler (`tsc.exe`) as well as the MSBuild integration (`Microsoft.TypeScript.targets` and `Microsoft.TypeScript.Tasks.dll`).

自 TypeScript 1.8 起，官方的 NuGet 包对 TypeScript 编译器（`tsc.exe`）以及 MSBuild 集成（`Microsoft.TypeScript.targets` 和 `Microsoft.TypeScript.Tasks.dll`）都可用。

Stable packages are available here:

稳定包在这里：

* [Microsoft.TypeScript.Compiler](https://www.nuget.org/packages/Microsoft.TypeScript.Compiler/)
* [Microsoft.TypeScript.MSBuild](https://www.nuget.org/packages/Microsoft.TypeScript.MSBuild/)

Also, a nightly NuGet package to match the [nightly npm package](http://blogs.msdn.com/b/typescript/archive/2015/07/27/introducing-typescript-nightlies.aspx) is available on [myget](https://myget.org):

还有，匹配 [nightly npm package](http://blogs.msdn.com/b/typescript/archive/2015/07/27/introducing-typescript-nightlies.aspx) 的每日版 NuGet 包在 [myget](https://myget.org)：

* [TypeScript-Preview](https://www.myget.org/gallery/typescript-preview)

# Prettier error messages from `tsc`
# 更美观的来自 `tsc` 的错误消息

We understand that a ton of monochrome output can be a little difficult on the eyes.
Colors can help discern where a message starts and ends, and these visual clues are important when error output gets overwhelming.

我们知道大量的单色输出对于眼睛（区分各种类型的信息）是有点困难的。颜色可以帮助识别消息的开始和结束位置，当错误输出变得无法控制时，这些视觉线索非常重要。

By just passing the `--pretty` command line option, TypeScript gives more colorful output with context about where things are going wrong.

![Showing off pretty error messages in ConEmu](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/pretty01.png)

通过传递 `--pretty` 命令行选项，TypeScript 提供更丰富多彩的输出，并提供相关错误的上下文。

# Colorization of JSX code in VS 2015
# VS 2015 中的 JSX 代码着色

With TypeScript 1.8, JSX tags are now classified and colorized in Visual Studio 2015.

伴随 TypeScript 1.8，JSX 标签在 Visual Studio 2015 中得到了分类和着色。

![jsx](https://cloud.githubusercontent.com/assets/8052307/12271404/b875c502-b90f-11e5-93d8-c6740be354d1.png)

The classification can be further customized by changing the font and color settings for the `VB XML` color and font settings through `Tools`->`Options`->`Environment`->`Fonts and Colors` page.

在 `Tools`->`Options`->`Environment`->`Fonts and Colors` 页面，通过修改字体和颜色的配置为 `VB XML` 颜色和字体设置，可以对分类进行更深入地定制。

# The `--project` (`-p`) flag can now take any file path
# `--project` (`-p`) 标志现在接受任何文件路径

The `--project` command line option originally could only take paths to a folder containing a `tsconfig.json`.
Given the different scenarios for build configurations, it made sense to allow `--project` to point to any other compatible JSON file.
For instance, a user might want to target ES2015 with CommonJS modules for Node 5, but ES5 with AMD modules for the browser.
With this new work, users can easily manage two separate build targets using `tsc` alone without having to perform hacky workarounds like placing `tsconfig.json` files in separate directories.

 `--project` 命令行选项最初只能获取包含了 `tsconfig.json` 文件的文件夹的路径。给定不同的构建配置场景，允许 `--project` 来指向任何其它兼容的 JSON 文件是有意义的。例如，用户可能想要为 Node 5 设置编译目标配置为 ES2015 并使用 CommonJS 模块方案，而为浏览器设置编译目标为 ES5，采用 AMD 模块方案。有了这项新的成果，用户能够很容易地管理两个单独的构建目标，使用 `tsc` 而不必再使用类似于把多个 `tsconfig.json` 文件放到不同的目录中这种变通办法了。

The old behavior still remains the same if given a directory - the compiler will try to find a file in the directory named `tsconfig.json`.

如果给定一个目录，旧的行为仍然保持相同——编译器将会尝试在给定的目录里面查找一个叫 `tsconfig.json` 的文件。

# Allow comments in tsconfig.json
# 允许在 tsconfig.json 中注释

It's always nice to be able to document your configuration!
`tsconfig.json` now accepts single and multi-line comments.

能够对你的配置编写文档是很好的！`tsconfig.json` 现在接受单行和多行注释。

```ts
{
    "compilerOptions": {
        "target": "ES2015", // running on node v5, yaay!
        "sourceMap": true   // makes debugging easier
    },
    /*
     * Excluded files
      */
    "exclude": [
        "file.d.ts"
    ]
}
```

# Support output to IPC-driven files
# 支持输出 IPC 驱动的文件

TypeScript 1.8 allows users to use the `--outFile` argument with special file system entities like named pipes, devices, etc.

TypeScript 1.8 允许用户把 `--outFile` 和特殊的文件系统实体如命名管道，设备等搭配使用。

As an example, on many Unix-like systems, the standard output stream is accessible by the file `/dev/stdout`.

作为一个例子，在许多类 Unix 的系统中，标准输出流通过 `/dev/stdout` 访问。

```shell
tsc foo.ts --outFile /dev/stdout
```

This can be used to pipe output between commands as well.

这也可以用于将输出导流到命令之间。

As an example, we can pipe our emitted JavaScript into a pretty printer like [pretty-js](https://www.npmjs.com/package/pretty-js):

作为例子，我们可以将生成的 JavaScript 导流到一个打印器如 [pretty-js](https://www.npmjs.com/package/pretty-js) ：

```shell
tsc foo.ts --outFile /dev/stdout | pretty-js
```

# Improved support for `tsconfig.json` in Visual Studio 2015
# 改进 Visual Studio 2015 中对 `tsconfig.json` 的支持

TypeScript 1.8 allows `tsconfig.json` files in all project types.
This includes ASP.NET v4 projects, *Console Application*, and the *Html Application with TypeScript* project types.
Further, you are no longer limited to a single `tsconfig.json` file but can add multiple, and each will be built as part of the project.
This allows you to separate the configuration for different parts of your application without having to use multiple different projects.

TypeScript 1.8 允许任何项目类型使用 `tsconfig.json` 文件。这包括 ASP.NET v4 项目，*控制台应用程序*，以及*使用 TypeScript 的 HTML 应用程序* 项目类型。更进一步，你不再被限制只能有一个 `tsconfig.json` 文件而是可以有多个，并且每个都将被构建为该项目的一部分。这允许你将配置拆分成你的应用的不同部分而不必使用多个不同的项目。

![Showing off tsconfig.json in Visual Studio](https://raw.githubusercontent.com/wiki/Microsoft/TypeScript/images/new-in-typescript/tsconfig-in-vs.png)

We also disable the project properties page when you add a `tsconfig.json` file.
This means that all configuration changes have to be made in the `tsconfig.json` file itself.

当添加了一个 `tsconfig.json` 文件时会禁用项目属性页。这意味着所有的配置更改都必须在这个 `tsconfig.json` 文件中进行。

### A couple of limitations
### 几个限制

* If you add a `tsconfig.json` file, TypeScript files that are not considered part of that context are not compiled.

  如果添加了 `tsconfig.json` 文件，那些不被认为是上下文环境中的一部分的 TypeScript 文件将不被编译。

* Apache Cordova Apps still have the existing limitation of a single `tsconfig.json` file, which must be in either the root or the `scripts` folder.

  Apache Cordova Apps 仍然有现在的单个 `tsconfig.json` 文件限制，它必须在根目录或者 `scripts` 目录。

* There is no template for `tsconfig.json` in most project types.

  在大多数的项目类型中没有 `tsconfig.json` 文件的模板。