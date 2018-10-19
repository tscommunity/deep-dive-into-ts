# ES6 Modules
# ES6 模块

TypeScript 1.5 supports ECMAScript 6 (ES6) modules.
ES6 modules are effectively TypeScript external modules with a new syntax: ES6 modules are separately loaded source files that possibly import other modules and provide a number of externally accessible exports.
ES6 modules feature several new export and import declarations.
It is recommended that TypeScript libraries and applications be updated to use the new syntax, but this is not a requirement.
The new ES6 module syntax coexists with TypeScript's original internal and external module constructs and the constructs can be mixed and matched at will.

TypeScript 1.5 支持 ECMAScript 6 (ES6) 模块。使用新的语法 ES6 模块是 TypeScript 的有效外部模块：ES6 模块都是单独进行加载的源文件，这些源文件可能导入其它模块并提供许多外部可访问的导出。ES6 模块具有多个新的导出和导入声明。推荐 TypeScript 库和应用程序更新到新的语法，但这不是必须的。新的 ES6 模块语法与 TypeScript 原来的内部和外部模块构造共存，这些构造能够被随意的混合和匹配。 

#### Export Declarations
#### 导出声明

In addition to the existing TypeScript support for decorating declarations with `export`, module members can also be exported using separate export declarations, optionally specifying different names for exports using `as` clauses.

除了现有的 TypeScript 支持的声明修饰符 `export`，模块成员也可以通过使用单独的导出声明被导出，还可以选择性地使用 `as` 子句为导出指定不同的名称。

```ts
interface Stream { ... }
function writeToStream(stream: Stream, data: string) { ... }
export { Stream, writeToStream as write };  // writeToStream exported as write
```

Import declarations, as well, can optionally use `as` clauses to specify different local names for the imports. For example:

同样地，导入声明也可以选择性地使用 `as` 子句来为导入指定不同的本地名称。例如：

```ts
import { read, write, standardOutput as stdout } from "./inout";
var s = read(stdout);
write(stdout, s);
```

As an alternative to individual imports, a namespace import can be used to import an entire module:

作为单独导入的替代方式，命名空间导入能够用来导入整个模块：

```ts
import * as io from "./inout";
var s = io.read(io.standardOutput);
io.write(io.standardOutput, s);
```

#### Re-exporting
#### 重导出

Using `from` clause a module can copy the exports of a given module to the current module without introducing local names.

使用 `from` 子句一个模块可以复制给定模块的导出到当前模块而不必引入本地名称。

```ts
export { read, write, standardOutput as stdout } from "./inout";
```

`export *` can be used to re-export all exports of another module. This is useful for creating modules that aggregate the exports of several other modules.

`export *` 能够被用来重新导出其它模块的所有导出。这对创建聚合若干个其它模块的导出的模块很有用。

```ts
export function transform(s: string): string { ... }
export * from "./mod1";
export * from "./mod2";
```

#### Default Export
#### 默认导出

An export default declaration specifies an expression that becomes the default export of a module:

一个默认导出声明指定了一个表达式，该表达式成为一个模块的默认导出：

```ts
export default class Greeter {
    sayHello() {
        console.log("Greetings!");
    }
}
```

Which in tern can be imported using default imports:

反过来可以用默认导入来导入它：

```ts
import Greeter from "./greeter";
var g = new Greeter();
g.sayHello();
```

#### Bare Import
#### 裸导入（单纯导入）

A "bare import" can be used to import a module only for its side-effects.

裸导入用于仅为了模块的作用而导入模块（例如在 Angular 开发中需要导入许多兼容浏览器的“填补包”（polyfills）时就只需要单纯的导入那些包即可，无需额外的代码调用）。

```ts
import "./polyfills";
```

For more information about module, please see the [ES6 module support spec](https://github.com/Microsoft/TypeScript/issues/2242).

更多关于模块的信息，请看 [ES6 模块支持规范](https://github.com/Microsoft/TypeScript/issues/2242)。

# Destructuring in declarations and assignments
# 声明和赋值中的解构

TypeScript 1.5 adds support to ES6 destructuring declarations and assignments.

TypeScript 1.5 添加了对 ES6 解构声明和解构赋值的支持。

#### Declarations
#### 声明

A destructuring declaration introduces one or more named variables and initializes them with values extracted from properties of an object or elements of an array.

解构声明引入一个或多个命名变量并用从对象的属性或者数组的元素中提取的值来初始化它们。

For example, the following sample declares variables `x`, `y`, and `z`, and initializes them to `getSomeObject().x`, `getSomeObject().y` and `getSomeObject().z` respectively:

例如，以下例子声明了变量 `x`， `y` 和 `z`，并分别初始化它们为 `getSomeObject().x`， `getSomeObject().y` 和 `getSomeObject().z`： 

```ts
var { x, y, z} = getSomeObject();
```

Destructuring declarations also works for extracting values from arrays:

解构声明同样对提取来自数组的值有效：

```ts
var [x, y, z = 10] = getSomeArray();
```

Similarly, destructuring  can be used in function parameter declarations:

类似地，解构还可以用于函数参数声明：

```ts
function drawText({ text = "", location: [x, y] = [0, 0], bold = false }) {
    // Draw text
}

// Call drawText with an object literal
// 用一个对象字面量调用 drawText
var item = { text: "someText", location: [1,2,3], style: "italics" };
drawText(item);
```

#### Assignments
#### 赋值

Destructuring patterns can also be used in regular assignment expressions.
For instance, swapping two variables can be written as a single destructuring assignment:

解构模式同样可以用于常规的赋值表达式。例如，交换两个变量的值可以写成一个单一的解构赋值：

```ts
var x = 1;
var y = 2;
[x, y] = [y, x];
```

# `namespace` keyword
# `namespace` 关键字

TypeScript used the `module` keyword to define both "internal modules" and "external modules";
this has been a bit of confusion for developers new to TypeScript.
"Internal modules" are closer to what most people would call a namespace; likewise, "external modules" in JS speak really just are modules now.

TypeScript 使用 `module` 这个关键字来定义“内部模块”和“外部模块”；这让新接触 TypeScript 的开发者感到迷惑不解。“内部模块”更接近于大部分人所称的命名空间；同样地，“外部模块”用 JS 的说法现在就是是模块。

> Note: Previous syntax defining internal modules are still supported.

> 注意：早先的定义内部模块的语法仍然受支持。

**Before**:

**以前**：

```ts
module Math {
    export function add(x, y) { ... }
}
```

**After**:

**以后**：

```ts
namespace Math {
    export function add(x, y) { ... }
}
```

# `let` and `const` support
# 支持 `let` 和 `const` 

ES6 `let` and `const` declarations are now supported when targeting ES3 and ES5.

当将编译目标设置为 ES3 和 ES5 时，ES6 中的 `let` 和 `const` 声明现在也受支持了。

#### Const
#### 常量

```ts
const MAX = 100;

++MAX; // Error: The operand of an increment or decrement
       //        operator cannot be a constant.
```

#### Block scoped
#### 块级作用域

```ts
if (true) {
  let a = 4;
  // use a
}
else {
  let a = "string";
  // use a
}

// Error: a is not defined in this scope
// 错误：a 未在此作用域定义
alert(a); 
```

# for..of support
# 支持 for..of

TypeScript 1.5 adds support to ES6 for..of loops on arrays for ES3/ES5 as well as full support for Iterator interfaces when targetting ES6.

TypeScript 1.5 为 ES3/ES5 添加了 ES6 中对数组的 for..of 循环的支持，且当将编译目标设置为 ES6 时完全支持迭代器接口。

##### Example
##### 示例

The TypeScript compiler will transpile for..of arrays to idiomatic ES3/ES5 JavaScript when targeting those versions:

当把编译目标设置为 ES3/ES5 时，TypeScript 编译器将会把 for..of 数组（遍历）转译成惯用版的 JavaScript：

```ts
for (var v of expr) { }
```

will be emitted as:

```js
for (var _i = 0, _a = expr; _i < _a.length; _i++) {
    var v = _a[_i];
}
```

# Decorators
# 装饰器

> TypeScript decorators are based on the [ES7 decorator proposal](https://github.com/wycats/javascript-decorators).

> TypeScript 装饰器是基于 [ES7 装饰器提案](https://github.com/wycats/javascript-decorators)的。

A decorator is:

一个装饰器是：

* an expression

  一个表达式

* that evaluates to a function
  
  计算结果为一个函数
  
* that takes the target, name, and property descriptor as arguments

  接受 target，name 及 property 描述符作为参数

* and optionally returns a property descriptor to install on the target object

  可选择性地返回一个属性描述符来装载到目标对象上  

> For more information, please see the [Decorators](https://github.com/Microsoft/TypeScript/issues/2249) proposal.

> 更多信息，请看[装饰器](https://github.com/Microsoft/TypeScript/issues/2249)提案。

##### Example
##### 示例

Decorators `readonly` and `enumerable(false)` will be applied to the property `method` before it is installed on class `C`.
This allows the decorator to change the implementation, and in this case, augment the descriptor to be writable: false and enumerable: false.

装饰器 `readonly` 和 `enumerable(false)` 将会被应用到属性 `method` 上，在它被装载到类 `C` 上之前。这允许装饰器改变实现，在这个例子中，为描述符增加 writable: false 和 enumerable: false。

```ts
class C {
  @readonly
  @enumerable(false)
  method() { }
}

function readonly(target, key, descriptor) {
    descriptor.writable = false;
}

function enumerable(value) {
  return function (target, key, descriptor) {
     descriptor.enumerable = value;
  }
}
```

# Computed properties
# 计算属性

Initializing an object with dynamic properties can be a bit of a burden. Take the following example:

用动态属性初始化一个对象是很麻烦的事情。考察如下例子：

```ts
type NeighborMap = { [name: string]: Node };
type Node = { name: string; neighbors: NeighborMap;}

function makeNode(name: string, initialNeighbor: Node): Node {
    var neighbors: NeighborMap = {};
    neighbors[initialNeighbor.name] = initialNeighbor;
    return { name: name, neighbors: neighbors };
}
```

Here we need to create a variable to hold on to the neighbor-map so that we can initialize it.
With TypeScript 1.5, we can let the compiler do the heavy lifting:

在此我们需要创建一个变量来抓住 neighbormap 以便我们可以初始化它。使用 TypeScript 1.5，我们可以让编译器来做这个费力的事情：

```ts
function makeNode(name: string, initialNeighbor: Node): Node {
    return {
        name: name,
        neighbors: {
            [initialNeighbor.name]: initialNeighbor
        }
    }
}
```

# Support for `UMD` and `System` module output
# 支持 `UMD` 和 `System` 模块输出

In addition to `AMD` and `CommonJS` module loaders, TypeScript now supports emitting modules `UMD` ([Universal Module Definition](https://github.com/umdjs/umd)) and [`System`](https://github.com/systemjs/systemjs) module formats.

除了 `AMD` 和 `CommonJS` 模块加载器，TypeScript 现在支持生成 `UMD` （[Universal Module Definition](https://github.com/umdjs/umd)）和 [`System`](https://github.com/systemjs/systemjs) 模块格式。

**Usage**:

**用法**：

> tsc --module umd

and

以及

> tsc --module system

# Unicode codepoint escapes in strings
# 字符串中的 Unicode 码点转义

ES6 introduces escapes that allow users to represent a Unicode codepoint using just a single escape.

ES6 中引入了转义以允许用户使用转义来表示一个 Unicode 码点。

As an example, consider the need to escape a string that contains the character '𠮷'.
In UTF-16/UCS2, '𠮷' is represented as a surrogate pair, meaning that it's encoded using a pair of 16-bit code units of values, specifically `0xD842` and `0xDFB7`.
Previously this meant that you'd have to escape the codepoint as `"\uD842\uDFB7"`.
This has the major downside that it’s difficult to discern two independent characters from a surrogate pair.

举个例子，考虑转义一个包含“𠮷”字符的字符串这样的需求。在 UTF-16/UCS2 中，“𠮷” 表示为一个代理对，意思是它被编码位 `0xD842` 和 `0xDFB7` 这对 16 位码元。在此前这意味着你不得不将它转义为 “`\uD842\uDFB7`”。这有一个主要的缺点，即难以从代理对中辨识出两个独立的字符。

With ES6’s codepoint escapes, you can cleanly represent that exact character in strings and template strings with a single escape: `"\u{20bb7}"`.
TypeScript will emit the string in ES3/ES5 as `"\uD842\uDFB7"`.

有了 ES6 的码点转义，你可以在字符串和模板字符串中通过简单地转义就可以清晰地表达那个准确地字符 “`\u{20bb7}`”。TypeScript 将会把 ES3/ES5 作为编译目标的字符串生成为 “`\uD842\uDFB7`”。

# Tagged template strings in ES3/ES5
# ES3/ES5 中的标签模板字符串

In TypeScript 1.4, we added support for template strings for all targets, and tagged templates for just ES6.
Thanks to some considerable work done by [@ivogabe](https://github.com/ivogabe), we bridged the gap for for tagged templates in ES3 and ES5.

在 TypeScript 1.4 中，我们为所有编译目标添加了模板字符串支持，并仅为 ES6 编译目标添加标签模板支持。感谢 [@ivogabe](https://github.com/ivogabe) 所做的大量工作，我们为 ES3 和 ES5 中的标签模板填补了空白。

When targeting ES3/ES5, the following code

当将编译目标设定为 ES3/ES5 时，以下代码

```ts
function oddRawStrings(strs: TemplateStringsArray, n1, n2) {
    return strs.raw.filter((raw, index) => index % 2 === 1);
}

oddRawStrings `Hello \n${123} \t ${456}\n world`
```

will be emitted as

将会被生成为

```js
function oddRawStrings(strs, n1, n2) {
    return strs.raw.filter(function (raw, index) {
        return index % 2 === 1;
    });
}
(_a = ["Hello \n", " \t ", "\n world"], _a.raw = ["Hello \\n", " \\t ", "\\n world"], oddRawStrings(_a, 123, 456));
var _a;
```

# AMD-dependency optional names
# AMD 依赖的可选名称

`/// <amd-dependency path="x" />` informs the compiler about a non-TS module dependency that needs to be injected in the resulting module's require call;
however, there was no way to consume this module in the TS code.

`/// <amd-dependency path="x" />` 告知编译器一个非 TS 模块依赖需要被注入到结果模块的 require 调用中；然而，无法在 TS 代码中使用这个模块。

The new `amd-dependency name` property allows passing an optional name for an amd-dependency:

 `amd-dependency name` 这个新的属性允许传递一个可选的名称给一个 AMD 依赖：

```ts
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA:MyType
moduleA.callStuff()
```

Generated JS code:

生成的 JS 代码：

```js
define(["require", "exports", "legacy/moduleA"], function (require, exports, moduleA) {
    moduleA.callStuff()
});
```

# Project support through `tsconfig.json`
# 通过 `tsconfig.json` 支持项目

Adding a `tsconfig.json` file in a directory indicates that the directory is the root of a TypeScript project.
The tsconfig.json file specifies the root files and the compiler options required to compile the project. A project is compiled in one of the following ways:

在一个目录里添加一个 `tsconfig.json` 文件表明该目录是一个 TypeScript 项目的根目录。 tsconfig.json 文件指定了根文件以及为编译这个项目所需的编译器选项。项目被以如下方式编译：

* By invoking tsc with no input files, in which case the compiler searches for the tsconfig.json file starting in the current directory and continuing up the parent directory chain.

  调用 tsc 而没有指定输入文件，这种方式下编译器从当前目录开始并继续向父目录链上查找 tsconfig.json 文件。

* By invoking tsc with no input files and a -project (or just -p) command line option that specifies the path of a directory containing a tsconfig.json file.

  调用 tsc 而没有指定输入文件但提供了一个 -project（或 -p）命令行选项，该命令行选项指定了包含 tsconfig.json 文件的目录的路径。

##### Example
##### 示例

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "sourceMap": true,
    }
}
```

See the [tsconfig.json wiki page](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) for more details.

更多详细信息请看 [tsconfig.json 维基页面](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json)。

# `--rootDir` command line option
# `--rootDir` 命令行选项

Option `--outDir` duplicates the input hierarchy in the output.
The compiler computes the root of the input files as the longest common path of all input files;
and then uses that to replicate all its substructure in the output.

`--outDir` 选项将输入的目录层次复制到输出。编译器计算输入文件的根目录作为所有输入文件的最长的通用路径，然后使用该路径来复制它所有的子结构到输出目录。

Sometimes this is not desirable, for instance inputs `FolderA\FolderB\1.ts` and `FolderA\FolderB\2.ts` would result in output structure mirroring `FolderA\FolderB\`.
Now if a new file `FolderA\3.ts` is added to the input, the output structure will pop out to mirror `FolderA\`.

有时这并不是让人满意的，例如输入 `FolderA\FolderB\1.ts` 和 `FolderA\FolderB\2.ts` 结果输出结构是 `FolderA\FolderB\`。现在如果添加一个新的文件 `FolderA\3.ts` 到输入，输出结果将是 `FolderA\`。

`--rootDir` specifies the input directory to be mirrored in output instead of computing it.

`--rootDir` 指定输入目录将会被镜像到输出而不是计算它。

# `--noEmitHelpers` command line option
# `--noEmitHelpers` 命令行选项

The TypeSript compiler emits a few helpers like `__extends` when needed.
The helpers are emitted in every file they are referenced in.
If you want to consolidate all helpers in one place, or override the default behavior, use `--noEmitHelpers` to instructs the compiler not to emit them.

当需要时，TypeScript 编译器会生成一些帮助器如 `__extends`。这些帮助器被生成到每个引用它们的文件中。如果你想要把所有的帮助器固定到一个地方，或者覆盖这个默认的行为，使用 `--noEmitHelpers` 来指示编译器不用生成它们。

# `--newLine` command line option
# `--newLine` 命令行选项

By default the output new line character is `\r\n` on Windows based systems and `\n` on *nix based systems.
`--newLine` command line flag allows overriding this behavior and specifying the new line character to be used in generated output files.

默认地，输出结果中的换行符在 Windows 系统中是 `\r\n` 而在 *nix 系统中是 `\n` 。`--newLine` 命令行标志允许覆盖这个行为，且允许指定换行符用于生成输出文件。

# `--inlineSourceMap` and `inlineSources` command line options
# `--inlineSourceMap` 和 `inlineSources` 命令行选项

`--inlineSourceMap` causes source map files to be written inline in the generated `.js` files instead of in a independent `.js.map` file.
`--inlineSources` allows for additionally inlining the source `.ts` file into the `.js` file.

`--inlineSourceMap` 会导致源代码映射文件被内联地写入到生成的 `.js` 文件中而不是独立的 `.js.map` 文件。`--inlineSources` 允许另外地将 `.ts` 源文件内联到 `.js` 文件中。