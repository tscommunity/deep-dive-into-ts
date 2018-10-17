# Union types
# 联合类型

### Overview
### 概览

Union types are a powerful way to express a value that can be one of several types. For example, you might have an API for running a program that takes a commandline as either a `string`, a `string[]` or a function that returns a `string`. You can now write:

联合类型是一种强大的方式来表示一个值可以是几种类型中的一种。例如，你可能有一个用于运行某个程序的 API，它接收命令行的参数为 `string`，`string[]` 或一个返回 `string` 的函数类型。你现在可以这样写：

```ts
interface RunOptions {
   program: string;
   commandline: string[]|string|(() => string);
}
```

Assignment to union types works very intuitively -- anything you could assign to one of the union type's members is assignable to the union:

赋值给联合类型的工作机制很直观——任何你能够赋值给联合类型的成员的值都可以赋值给该联合类型：

```ts
var opts: RunOptions = /* ... */;
opts.commandline = '-hello world'; // OK
opts.commandline = ['-hello', 'world']; // OK
opts.commandline = [42]; // Error, number is not string or string[]
```

When reading from a union type, you can see any properties that are shared by them:

当读取联合类型时，你可以看到任何它们共有的属性：

```ts
if (opts.length === 0) { // OK, string and string[] both have 'length' property
  console.log("it's empty");
}
```

Using Type Guards, you can easily work with a variable of a union type:

使用类型保护，你可以很容易地处理联合类型变量：

```ts
function formatCommandline(c: string|string[]) {
    if (typeof c === 'string') {
        return c.trim();
    }
    else {
        return c.join(' ');
    }
}
```

### Stricter Generics
### 更严格地泛型

With union types able to represent a wide range of type scenarios, we've decided to improve the strictness of certain generic calls. Previously, code like this would (surprisingly) compile without error:

因联合类型能够表示广泛的类型场景，我们决定提升某些泛型调用的严格性。早先版本中，这样的代码将（令人惊讶地）编译无误：

```ts
function equal<T>(lhs: T, rhs: T): boolean {
  return lhs === rhs;
}

// Previously: No error
// 早先版本：无错
// New behavior: Error, no best common type between 'string' and 'number'
// 新的行为：错误， 'string' 和 'number' 之间没有共同类型
var e = equal(42, 'hello');
```

With union types, you can now specify the desired behavior at both the function declaration site and the call site:

使用联合类型，你现在可以在函数声明和调用指定所需的行为：

```ts
// 'choose' function where types must match
function choose1<T>(a: T, b: T): T { return Math.random() > 0.5 ? a : b }
var a = choose1('hello', 42); // Error
var b = choose1<string|number>('hello', 42); // OK

// 'choose' function where types need not match
function choose2<T, U>(a: T, b: U): T|U { return Math.random() > 0.5 ? a : b }
var c = choose2('bar', 'foo'); // OK, c: string
var d = choose2('hello', 42); // OK, d: string|number
```

### Better Type Inference
### 更好的类型推导

Union types also allow for better type inference in arrays and other places where you might have multiple kinds of values in a collection:

联合类型同样允许在数组和其它地方（有多种值的集合）中的更好的类型推导：

```ts
var x = [1, 'hello']; // x: Array<string|number>
x[0] = 'world'; // OK
x[0] = false; // Error, boolean is not string or number
```

# `let` declarations
# `let` 声明

In JavaScript, `var` declarations are "hoisted" to the top of their enclosing scope. This can result in confusing bugs:

在 JavaScript 中，`var` 声明都会被“提升”到它们所在域的顶部。这样会导致令人迷惑的 bugs：

```ts
console.log(x); // meant to write 'y' here
/* later in the same block */
var x = 'hello';
```

The new ES6 keyword `let`, now supported in TypeScript, declares a variable with more intuitive "block" semantics. A `let` variable can only be referred to after its declaration, and is scoped to the syntactic block where it is defined:

新的 ES6 关键词 `let` ，现在已在 TypeScript 中受支持，声明了一个在更直观的语义“块”中的变量。一个 `let` 变量仅能够在其被声明之后引用，且仅作用于其被定义时所在的句法块：

```ts
if (foo) {
    // Error, cannot refer to x before its declaration
    // 错误，不能在 x 声明之前引用它
    console.log(x); 
    let x = 'hello';
}
else {
    // Error, x is not declared in this block
    // 错误，x 未定义在此句法块中
    console.log(x); 
}
```

`let` is only available when targeting ECMAScript 6 (`--target ES6`).

仅当将编译目标设置为 ECMAScript 6 （`--target ES6`）时 `let` 才可用。

# `const` declarations
# `const` 声明

The other new ES6 declaration type supported in TypeScript is `const`. A `const` variable may not be assigned to, and must be initialized where it is declared. This is useful for declarations where you don't want to change the value after its initialization:

其它在 TypeScript 中受支持的新的 ES6 声明类型是 `const`。一个 `const` 变量不能被赋值且必须在声明它时初始化它。

```ts
const halfPi = Math.PI / 2;
halfPi = 2; // Error, can't assign to a `const`
```

`const` is only available when targeting ECMAScript 6 (`--target ES6`).

仅当将编译目标设置为 ECMAScript 6 （`--target ES6`）时 `const` 才可用。

# Template strings
# 模板字符串

TypeScript now supports ES6 template strings. These are an easy way to embed arbitrary expressions in strings:

TypeScript 现在支持 ES6 的模板字符串。有一种简单的方式来将任意表达式嵌入字符串中：

```ts
var name = "TypeScript";
var greeting  = `Hello, ${name}! Your name has ${name.length} characters`;
```

When compiling to pre-ES6 targets, the string is decomposed:

当编译为较 ES6 早的目标时，字符串被分解：

```js
var name = "TypeScript!";
var greeting = "Hello, " + name + "! Your name has " + name.length + " characters";
```

# Type Guards
# 类型保护

A common pattern in JavaScript is to use `typeof` or `instanceof` to examine the type of an expression at runtime. TypeScript now understands these conditions and will change type inference accordingly when used in an `if` block.

JavaScript 中，在运行时检查一个表达式的类型的常用模式是使用 `typeof` 或 `instanceof` 。当在 `if` 块中使用时，TypeScript 现在理解这些条件且将相应地改变类型推导。

Using `typeof` to test a variable:

使用 `typeof` 来测试一个变量（的类型）：

```ts
var x: any = /* ... */;
if(typeof x === 'string') {
    console.log(x.subtr(1)); // Error, 'subtr' does not exist on 'string'
}
// x is still any here
x.unknown(); // OK
```

Using `typeof` with union types and `else`:

使用 `typeof` 与联合类型及 `else`

```ts
var x: string | HTMLElement = /* ... */;
if(typeof x === 'string') {
    // x is string here, as shown above
}
else {
    // x is HTMLElement here
    console.log(x.innerHTML);
}
```

Using `instanceof` with classes and union types:

与类和联合类型使用 `instanceof` 

```ts
class Dog { woof() { } }
class Cat { meow() { } }
var pet: Dog|Cat = /* ... */;
if (pet instanceof Dog) {
    pet.woof(); // OK
}
else {
    pet.woof(); // Error
}
```

# Type Aliases
# 类型别名

You can now define an *alias* for a type using the `type` keyword:

现在你可以使用 `type` 关键字为某个类型定义一个*别名*：

```ts
type PrimitiveArray = Array<string|number|boolean>;
type MyNumber = number;
type NgScope = ng.IScope;
type Callback = () => void;
```

Type aliases are exactly the same as their original types; they are simply alternative names.

类型别名与它们原始类型是完全一样的；它们仅仅是替代名。

# `const enum` (completely inlined enums)
# `const enum` 全内联枚举

Enums are very useful, but some programs don't actually need the generated code and would benefit from simply inlining all instances of enum members with their numeric equivalents. The new `const enum` declaration works just like a regular `enum` for type safety, but erases completely at compile time.

枚举是非常有用的，但某些程序并不真的需要所生成的代码且将从用所有枚举成员对应的数字值来进行简单的内联中受益。新的 `const enum` 声明对类型安全的作用和常规的 `enum` 一样，但在编译时完全擦除。

```ts
const enum Suit { Clubs, Diamonds, Hearts, Spades }
var d = Suit.Diamonds;
```

Compiles to exactly:

编译为：

```js
var d = 1;
```

TypeScript will also now compute enum values when possible:

TypeScript 将在可能的时候计算枚举值：

```ts
enum MyFlags {
  None = 0,
  Neat = 1,
  Cool = 2,
  Awesome = 4,
  Best = Neat | Cool | Awesome
}
var b = MyFlags.Best; // emits var b = 7;
```

# `-noEmitOnError` commandline option
# `-noEmitOnError` 命令行选项

The default behavior for the TypeScript compiler is to still emit .js files if there were type errors (for example, an attempt to assign a `string` to a `number`). This can be undesirable on build servers or other scenarios where only output from a "clean" build is desired. The new flag `noEmitOnError` prevents the compiler from emitting .js code if there were any errors.

TypeScript 编译器的默认行为是如果有类型错误（例如，企图将一个 `string` 类型赋值给一个 `number` 类型）仍然生成 .js 文件。在构建服务器或其它场景中这是不需要的，需要的只是来自一个“干净”的构建的输出。这个新的标志 `noEmitOnError` 在有任何错误时阻止编译器生成 .js 代码。

This is now the default for MSBuild projects; this allows MSBuild incremental build to work as expected, as outputs are only generated on clean builds.

对于 MSBuild 项目这是默认的；这允许 MSBuild 增量构建能如期望的那样工作，因为输出仅在干净构建之上生成。

# AMD Module names
# AMD 模块名称

By default AMD modules are generated anonymous. This can lead to problems when other tools are used to process the resulting modules like a bundlers (e.g. `r.js`).

默认的 AMD 模块都被匿名地生成。当其它工具如资源捆绑器（例如 `r.js`）被用来处理这些结果模块时这将导致问题。

The new `amd-module name` tag allows passing an optional module name to the compiler:

新的 `amd-module name` 标签允许传递一个可选的模块名给编译器：

```ts
//// [amdModule.ts]
///<amd-module name='NamedModule'/>
export class C {
}
```

Will result in assigning the name `NamedModule` to the module as part of calling the AMD `define`:

将造成将名称 `NamedModule` 赋值给模块以作为 AMD `define` 的一部分：

```js
//// [amdModule.js]
define("NamedModule", ["require", "exports"], function (require, exports) {
    var C = (function () {
        function C() {
        }
        return C;
    })();
    exports.C = C;
});
```