# JSX support
# JSX 支持

JSX is an embeddable XML-like syntax.
It is meant to be transformed into valid JavaScript, but the semantics of that transformation are implementation-specific.
JSX came to popularity with the React library but has since seen other applications.
TypeScript 1.6 supports embedding, type checking, and optionally compiling JSX directly into JavaScript.

JSX 是一个可嵌入的类似 XML 的语法。这意味着能被转换成有效的 JavaScript，但转换的语义是特定的实现。JSX 随着 React 库变得流行但其它应用也看到了（JSX）。TypeScript 1.6 支持嵌入，类型检查以及可选性地将 JSX 编译为 JavaScript。

#### New `.tsx` file extension and `as` operator
#### 新的 `.tsx` 文件扩展和 `as` 运算符

TypeScript 1.6 introduces a new `.tsx` file extension.
This extension does two things: it enables JSX inside of TypeScript files, and it makes the new `as` operator the default way to cast (removing any ambiguity between JSX expressions and the TypeScript prefix cast operator).

TypeScript 1.6 引入了一个新的 `.tsx` 文件扩展。这个扩展做了两件事：它使得 JSX 能够在 TypeScript 文件中使用且使得新的 `as` 运算符成为默认的转换方式（消除了 JSX 表达式和 TypeScript 前缀转换运算符的歧义）。

For example:

例如：

```ts
var x = <any> foo;
// is equivalent to:
var x = foo as any;
```

#### Using React
#### 使用 React

To use JSX-support with React you should use the [React typings](https://github.com/borisyankov/DefinitelyTyped/tree/master/react). These typings define the `JSX` namespace so that TypeScript can correctly check JSX expressions for React. For example:

为了在 React 中使用 JSX 支持，你应该使用 [React typings](https://github.com/borisyankov/DefinitelyTyped/tree/master/react)。这些类型定义了 `JSX` 命名空间以便 TypeScript 能够为 React 正确地检查 JSX 表达式。例如：

```ts
/// <reference path="react.d.ts" />

interface Props {
  name: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent name="bar" />; // OK
<MyComponent name={0} />; // error, `name` is not a number
```

#### Using other JSX framworks
#### 使用其它 JSX 框架

JSX element names and properties are validated against the `JSX` namespace.
Please see the [JSX](https://en.wikipedia.org/wiki/JSX_(JavaScript)) wiki page for defining the `JSX` namespace for your framework.

JSX 元素名称和属性在 `JSX` 命名空间中是有效的。要为你的框架定义 `JSX` 命名空间请查看 [JSX](https://en.wikipedia.org/wiki/JSX_(JavaScript)) 维基页面。

#### Output generation
#### 输出生成

TypeScript ships with two JSX modes: `preserve` and `react`.

TypeScript 发布有两个 JSX 模式：`preserve` 和 `react`。

* The `preserve` mode will keep JSX expressions as part of the output to be further consumed by another transform step. *Additionally the output will have a `.jsx` file extension.*

  `preserve` 模式将会保留 JSX 表达式作为输出的一部分以进一步供其它转换步骤使用。*此外输出将是 `.jsx` 文件。*

* The `react` mode will emit `React.createElement`, does not need to go through a JSX transformation before use, and the output will have a `.js` file extension.

  `react` 模式将生成 `React.createElement`，在使用前无需再经过 JSX 转换，且输出将是 `.js` 文件。

See the [[JSX]] wiki page for more information on using JSX in TypeScript.

更过关于在 TypeScript 中使用 JSX 的信息请看 [JSX] 维基页面。

# Intersection types
# 交叉类型

TypeScript 1.6 introduces intersection types, the logical complement of union types.
A union type `A | B` represents an entity that is either of type `A` or type `B`, whereas an intersection type `A & B` represents an entity that is both of type `A` *and* type `B`.

TypeScript 1.6 引入了交叉类型，作为联合类型的逻辑补充。一个联合类型 `A | B` 表示一个实体，它是类型 `A` 或类型 `B`，而一个交叉类型 `A & B` 表示一个实体既是类型 `A` *又是* 类型 `B`。

##### Example
##### 示例

```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U> {};
    for (let id in first) {
        result[id] = first[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            result[id] = second[id];
        }
    }
    return result;
}

var x = extend({ a: "hello" }, { b: 42 });
var s = x.a;
var n = x.b;
```

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

```ts
interface A { a: string }
interface B { b: string }
interface C { c: string }

var abc: A & B & C;
abc.a = "hello";
abc.b = "hello";
abc.c = "hello";
```

See [issue #1256](https://github.com/Microsoft/TypeScript/issues/1256) for more information.

更多信息请看 [问题 #1256](https://github.com/Microsoft/TypeScript/issues/1256)。

# Local type declarations
# 局部类型声明

Local class, interface, enum, and type alias declarations can now appear inside function declarations. Local types are block scoped, similar to variables declared with `let` and `const`. For example:

局部类，接口，枚举和类型别名声明现在可以出现在函数声明的内部了。局部类型是块级作用域的，与用 `let` 和 `const` 声明的变量类似。例如：

```ts
function f() {
    if (true) {
        interface T { x: number }
        let v: T;
        v.x = 5;
    }
    else {
        interface T { x: string }
        let v: T;
        v.x = "hello";
    }
}
```

The inferred return type of a function may be a type declared locally within the function. It is not possible for callers of the function to reference such a local type, but it can of course be matched structurally. For example:

一个函数的返回类型的推导结果可能是在函数内部定义的局部类型。函数调用者不可能引用该局部类型，但是能够在结构上匹配该类型。

```ts
interface Point {
    x: number;
    y: number;
}

function getPointFactory(x: number, y: number) {
    class P {
        x = x;
        y = y;
    }
    return P;
}

var PointZero = getPointFactory(0, 0);
var PointOne = getPointFactory(1, 1);
var p1 = new PointZero();
var p2 = new PointZero();
var p3 = new PointOne();
```

Local types may reference enclosing type parameters and local class and interfaces may themselves be generic. For example:

局部类型可以引用封闭类型参数且局部类和接口可以是泛型的。例如：

```ts
function f3() {
    function f<X, Y>(x: X, y: Y) {
        class C {
            public x = x;
            public y = y;
        }
        return C;
    }
    let C = f(10, "hello");
    let v = new C();
    let x = v.x;  // number
    let y = v.y;  // string
}
```

# Class expressions
# 类表达式

TypeScript 1.6 adds support for ES6 class expressions. In a class expression, the class name is optional and, if specified, is only in scope in the class expression itself. This is similar to the optional name of a function expression. It is not possible to refer to the class instance type of a class expression outside the class expression, but the type can of course be matched structurally. For example:

TypeScript 1.6 添加了对 ES6 中的类表达式的支持。在类表达式中，类名是可选的，且如果指定了类名，则其只作用在类表达式范围内。这和函数表达式的可选名称类似。不能在类表达式外引用该类表达式的类实例的类型，但是能够在结构上匹配该类型。例如：

```ts
let Point = class {
    constructor(public x: number, public y: number) { }
    public length() {
        return Math.sqrt(this.x * this.x + this.y * this.y);
    }
};
var p = new Point(3, 4);  // p has anonymous class type
console.log(p.length());
```

# Extending expressions
# 扩展表达式

TypeScript 1.6 adds support for classes extending arbitrary expression that computes a constructor function. This means that built-in types can now be extended in class declarations.

TypeScript 1.6 添加了对类扩展任意计算结果为构造器函数的表达式的支持。这意味着现在能够在类声明中对内置类型进行扩展。

The `extends` clause of a class previously required a type reference to be specified. It now accepts an expression optionally followed by a type argument list. The type of the expression must be a constructor function type with at least one construct signature that has the same number of type parameters as the number of type arguments specified in the `extends` clause. The return type of the matching construct signature(s) is the base type from which the class instance type inherits. Effectively, this allows both real classes and "class-like" expressions to be specified in the `extends` clause.

在以前，类的 `extends` 需要指定一个类型引用。现在它接受一个伴随着可选的类型参数列表的表达式。该表达式的类型必须是一个构造器函数类型，它至少有一个与 `extends` 子句中指定的类型参数有着相同数量的类型参数的构造签名。

Some examples:

一些例子：

```ts
// Extend built-in types

class MyArray extends Array<number> { }
class MyError extends Error { }

// Extend computed base class

class ThingA {
    getGreeting() { return "Hello from A"; }
}

class ThingB {
    getGreeting() { return "Hello from B"; }
}

interface Greeter {
    getGreeting(): string;
}

interface GreeterConstructor {
    new (): Greeter;
}

function getGreeterBase(): GreeterConstructor {
    return Math.random() >= 0.5 ? ThingA : ThingB;
}

class Test extends getGreeterBase() {
    sayHello() {
        console.log(this.getGreeting());
    }
}
```

# `abstract` classes and methods
# `abstract` 类和方法

TypeScript 1.6 adds support for `abstract` keyword for classes and their methods. An abstract class is allowed to have methods with no implementation, and cannot be constructed.

TypeScript 1.6 添加了对类和它的方法的 `abstract` 关键字的支持。抽象类允许有没有实现的方法且不可以实例化。

##### Examples
##### 示例

```ts
abstract class Base {
    abstract getThing(): string;
    getOtherThing() { return 'hello'; }
}

// Error, 'Base' is abstract
// 错误，“Base”是抽象的
let x = new Base(); 

// Error, must either be 'abstract' or implement concrete 'getThing'
// 错误，必须是“抽象”或者具体实现“getThing”
class Derived1 extends Base { }

class Derived2 extends Base {
    getThing() { return 'hello'; }
    foo() {
        // Error: cannot invoke abstract members through 'super'
        // 错误：不能通过‘super’来调用抽象成员
        super.getThing();
    }
}

var x = new Derived2(); // OK
var y: Base = new Derived2(); // Also OK
y.getThing(); // OK
y.getOtherThing(); // OK
```

# Generic type aliases
# 泛型类型别名

With TypeScript 1.6, type aliases can be generic. For example:

TypeScript 1.6 中，类型别名可以是泛型的。例如：

```ts
type Lazy<T> = T | (() => T);

var s: Lazy<string>;
s = "eager";
s = () => "lazy";

interface Tuple<A, B> {
    a: A;
    b: B;
}

type Pair<T> = Tuple<T, T>;
```

# Stricter object literal assignment checks
# 更严格的对象字面量赋值检查

TypeScript 1.6 enforces stricter object literal assignment checks for the purpose of catching excess or misspelled properties. Specifically, when a fresh object literal is assigned to a variable or passed as an argument for a non-empty target type, it is an error for the object literal to specify properties that don't exist in the target type.

TypeScript 1.6 执行了更严格的对象字面量赋值检查以捕获多余或拼写错误的属性。特别地，当一个新的对象字面量被赋值给一个变量或作为一个参数传递给一个非空的目标类型时，对对象字面量指定目标类型中不存在的属性是错误的。

##### Examples
##### 示例

```ts
var x: { foo: number };
// Error, excess property `baz`
// 错误，多余的属性“baz”
x = { foo: 1, baz: 2 };  

var y: { foo: number, bar?: number };
// Error, excess or misspelled property `baz`
// 错误，多余或错误拼写的属性“baz”
y = { foo: 1, baz: 2 };  
```

A type can include an index signature to explicitly indicate that excess properties are permitted:

一个类型允许包含一个索引签名以显式地表明剩余的属性：

```ts
var x: { foo: number, [x: string]: any };
// Ok, `baz` matched by index signature
// 可以，“baz”匹配索引签名
x = { foo: 1, baz: 2 };  
```

# ES6 generators
# ES6 生成器

TypeScript 1.6 adds support for generators when targeting ES6.

TypeScript 1.6 为编译目标设置为 ES6 时添加了生成器的支持。

A generator function can have a return type annotation, just like a function. The annotation represents the type of the generator returned by the function. Here is an example:

一个生成器函数可以有一个返回类型注解，与函数一样。这个注解表示由函数返回的该生成器的类型。这是一个例子：

```ts
function *g(): Iterable<string> {
    for (var i = 0; i < 100; i++) {
        yield ""; // string is assignable to string
    }
    // otherStringGenerator must be iterable and element type assignable to string
    // otherStringGenerator 必须是 iterable 且元素类型是可以赋值给 string 类型的。
    yield * otherStringGenerator(); 
}
```

A generator function with no type annotation can have the type annotation inferred.
So in the following case, the type will be inferred from the yield statements:

一个没有类型注解的生成器函数可以有类型注解推断。所以以下这个例子，类型将被从 yield 语句推导出来。

```ts
function *g() {
    for (var i = 0; i < 100; i++) {
        yield ""; // infer string
    }
    yield * otherStringGenerator(); // infer element type of otherStringGenerator
}
```

# Experimental support for `async` functions
# 为 `async` 函数提供试验性支持

TypeScript 1.6 introduces experimental support of `async` functions when targeting ES6.
Async functions are expected to invoke an asynchronous operation and await its result without blocking normal execution of the program.
This accomplished through the use of an ES6-compatible `Promise` implementation, and transposition of the function body into a compatible form to resume execution when the awaited asynchronous operation completes.

当编译目标为 ES6 时，TypeScript 1.6 为 `async` 函数提供试验性的支持。异步函数被用来调用异步操作并等待它的结果而不会阻塞程序的正常执行。这是通过使用与 ES6 兼容的 `Promise` 实现，以及将函数体转换为一个兼容的形式以在等待的异步操作完成时恢复执行来实现的。

An *async function* is a function or method that has been prefixed with the `async` modifier. This modifier informs the compiler that function body transposition is required, and that the keyword `await` should be treated as a unary expression instead of an identifier.
An *Async Function* must provide a return type annotation that points to a compatible `Promise` type. Return type inference can only be used if there is a globally defined, compatible `Promise` type.

一个*异步函数*是一个带有 `async` 修饰符前缀的函数或方法。这个修饰符告知编译器函数体需要被转换，以及关键字 `await` 应该被当作一个一元表达式而不是一个标识符。一个*异步函数*必须提供一个指向兼容的 `Promise` 类型的返回类型注解。返回类型推导仅当存在定义于全局范围的，兼容的 `Promise` 类型时可用。

##### Example
##### 示例

```ts
var p: Promise<number> = /* ... */;
async function fn(): Promise<number> {
  // suspend execution until 'p' is settled. 'i' has type "number"
  // 挂起执行直到“p”已经完结。“i”具有“number”类型
  var i = await p; 
  return 1 + i;
}

// suspends execution.
// 挂起执行。
var a = async (): Promise<number> => 1 + await p; 

// suspends execution. return type is inferred as "Promise<number>" when compiling with --target ES6
// 挂起执行。当编译使用 --target ES6 时返回类型被推导为“Promise<number>”
var a = async () => 1 + await p; 
var fe = async function(): Promise<number> {
  var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
  return 1 + i;
}

class C {
  async m(): Promise<number> {
    var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
    return 1 + i;
  }

  async get p(): Promise<number> {
    var i = await p; // suspend execution until 'p' is settled. 'i' has type "number"
    return 1 + i;
  }
}
```

# Nightly builds
# 每晚构建

While not strictly a language change, nightly builds are now available by installing with the following command:

虽然不是严格意义上的语言更改，但现在可以通过使用以下命令进行安装来获得每晚构建：

```Shell
npm install -g typescript@next
```

# Adjustments in module resolution logic
# 模块解析逻辑中的调整

Starting from release 1.6 TypeScript compiler will use different set of rules to resolve module names when targeting 'commonjs'.
These [rules](https://github.com/Microsoft/TypeScript/issues/2338) attempted to model module lookup procedure used by Node.
This effectively mean that node modules can include information about its typings and TypeScript compiler will be able to find it.
User however can override module resolution rules picked by the compiler by using `--moduleResolution` command line option. Possible values are:

自 1.6 版本发布起，当模块目标为“commonjs”时 TypeScript 编译器将使用不同的规则集来解析模块名称。这些[规则](https://github.com/Microsoft/TypeScript/issues/2338)试图模拟 Node 使用的模块查找过程。

* 'classic' - module resolution rules used by pre 1.6 TypeScript compiler

  “classic” —— 1.6 版本之前的 TypeScript 编译器使用的模块解析规则

* 'node' - node-like module resolution

  “node” —— 类似 node 的模块解析

# Merging ambient class and interface declaration
# 合并周围类和接口声明

The instance side of an ambient class declaration can be extended using an interface declaration The class constructor object is unmodified.
For example:

可以使用接口声明来扩展周围类声明的实例端，类构造器对象不被修改。例如：

```ts
declare class Foo {
    public x : number;
}

// 注：对于接口 Foo 而言，“declare class Foo”就是它的周围类声明，而下面 bar 函数的参数“foo”则是周围类声明的实例端。
interface Foo {
    y : string;
}

function bar(foo : Foo)  {
    foo.x = 1; // OK, declared in the class Foo
    foo.y = "1"; // OK, declared in the interface Foo
}
```

# User-defined type guard functions
# 用户定义的类型保护函数

TypeScript 1.6 adds a new way to narrow a variable type inside an `if` block, in addition to `typeof` and `instanceof`.
A user-defined type guard functions is one with a return type annotation of the form `x is T`, where `x` is a declared parameter in the signature, and `T` is any type.
When a user-defined type guard function is invoked on a variable in an `if` block, the type of the variable will be narrowed to `T`.

除了 `typeof` 和 `instanceof` 以外，TypeScript 1.6 添加了一个新方式来收窄 `if` 块中的变量的类型。用户定义的类型保护函数是一个带 `x is T` 形式的返回类型注解的函数，其中 `x` 是在函数签名中声明的参数，`T` 是任何类型。当在 `if` 块中调用一个用户定义的类型保护函数时，变量的类型将被窄化为 `T`。

##### Examples
##### 示例

```ts
function isCat(a: any): a is Cat {
  return a.name === 'kitty';
}

var x: Cat | Dog;
if(isCat(x)) {
  x.meow(); // OK, x is Cat in this block
}
```

# `exclude` property support in tsconfig.json
# 在 tsconfig.json 中支持 `exclude` 属性

A tsconfig.json file that doesn't specify a files property (and therefore implicitly references all *.ts files in all subdirectories) can now contain an exclude property that specifies a list of files and/or directories to exclude from the compilation.
The exclude property must be an array of strings that each specify a file or folder name relative to the location of the tsconfig.json file.
For example:

未指定“files”属性（所以隐式地引用所有子目录中所有的 *.ts 文件）的 tsconfig.json 文件现在可以包含一个“exclude”属性，该属性指定了一个要从编译中排除的文件和/或目录的列表。这个“exclude”属性必须是一个字符串数组，它的每个元素指定了要排除的相对于 tsconfig.json 文件位置的文件或文件夹名称。例如：

```json
{
    "compilerOptions": {
        "out": "test.js"
    },
    "exclude": [
        "node_modules",
        "test.ts",
        "utils/t2.ts"
    ]
}
```

The `exclude` list does not support wilcards. It must simply be a list of files and/or directories.

 `exclude` 列表不支持通配符。它必须只是一个文件和/或目录的列表。

# `--init` command line option
# `--init` 命令行选项

Run `tsc --init` in a directory to create an initial `tsconfig.json` in this directory with preset defaults.
Optionally pass command line arguments along with `--init` to be stored in your initial tsconfig.json on creation.

在一个目录里运行 `tsc --init` 命令以在该目录创建一个带默认预设值的初始 `tsconfig.json` 文件。跟随在 `--init` 之后传递的可选的命令行参数被存储在初始的 tsconfig.json 文件中。