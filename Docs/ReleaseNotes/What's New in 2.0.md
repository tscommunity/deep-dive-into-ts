# Null- and undefined-aware types
# null 和 undefined 敏感类型

TypeScript has two special types, Null and Undefined, that have the values `null` and `undefined` respectively.
Previously it was not possible to explicitly name these types, but `null` and `undefined` may now be used as type names regardless of type checking mode.

TypeScript 有两个特殊类型，Null 和 Undefined，值分别为 `null` 和 `undefined`。
以前无法显式地命名这些类型，但 `null` 和 `undefined` 现在可以被用作类型名称而不管类型检查模式如何。

The type checker previously considered `null` and `undefined` assignable to anything.
Effectively, `null` and `undefined` were valid values of *every* type and it wasn't possible to specifically exclude them (and therefore not possible to detect erroneous use of them).

在以前类型检查器认为 `null` 和 `undefined` 可以赋值给任何类型。 `null` 和 `undefined` 过去曾是*任何*类型的有效值且无法专门的将它们排除（也因此无法检查对它们的误用）。

## `--strictNullChecks`
## `--strictNullChecks` 编译器选项

`--strictNullChecks` switches to a new strict null checking mode.

`--strictNullChecks` 切换到一个新的空检查模式。

In strict null checking mode, the `null` and `undefined` values are *not* in the domain of every type and are only assignable to themselves and `any` (the one exception being that `undefined` is also assignable to `void`).
So, whereas `T` and `T | undefined` are considered synonymous in regular type checking mode (because `undefined` is considered a subtype of any `T`), they are different types in strict type checking mode, and only `T | undefined` permits `undefined` values. The same is true for the relationship of `T` to `T | null`.

在严格的空检查模式中， `null` 和 `undefined` *不*在任何类型的值域中且只能赋值给自身和 `any`（一个例外是 `undefined` 也可以赋值给 `void`）。所以， `T` 和 `T | undefined` 在常规的类型检查模式中被认为是同义的（因为 `undefined` 被认为是任何 `T` 的子类型），在严格的类型检查模式中被认为是不同的，且只有 `T | undefined` 允许 `undefined` 作为值。 `T` 和 `T | null` 的关系亦是如此。

##### Example
##### 示例

```ts
// Compiled with --strictNullChecks
// 使用 --strictNullChecks 编译
let x: number;
let y: number | undefined;
let z: number | null | undefined;
x = 1;  // Ok
y = 1;  // Ok
z = 1;  // Ok
x = undefined;  // Error
y = undefined;  // Ok
z = undefined;  // Ok
x = null;  // Error
y = null;  // Error
z = null;  // Ok
x = y;  // Error
x = z;  // Error
y = x;  // Ok
y = z;  // Error
z = x;  // Ok
z = y;  // Ok
```

## Assigned-before-use checking
## 用前赋值检查

In strict null checking mode the compiler requires every reference to a local variable of a type that doesn't include `undefined` to be preceded by an assignment to that variable in every possible preceding code path.

在严格的空检查模式中，编译器要求任何对不包含 `undefined` 的类型的局部变量的引用必须在对该变量赋值的代码之后。

##### Example
##### 示例

```ts
// Compiled with --strictNullChecks
let x: number;
let y: number | null;
let z: number | undefined;
// Error, reference not preceded by assignment
// 错误，在赋值前引用
x;  

// Error, reference not preceded by assignment
// 错误，在赋值前引用
y;  
z;  // Ok
x = 1;
y = null;
x;  // Ok
y;  // Ok
```

The compiler checks that variables are definitely assigned by performing *control flow based type analysis*. See later for further details on this topic.

编译器通过执行*基于控制流的类型分析*来检查变量是否已明确地被赋值了。关于此主题的更多详细信息请参看后面。

## Optional parameters and properties
## 可选参数和属性

Optional parameters and properties automatically have `undefined` added to their types, even when their type annotations don't specifically include `undefined`.
For example, the following two types are identical:

可选参数和属性自动将 `undefined` 添加到它们的类型中，即使它们的类型注解没有专门地将 `undefined` 包括进去。例如，以下的两个是相同的：

```ts
// Compiled with --strictNullChecks
type T1 = (x?: number) => string;              // x has type number | undefined
type T2 = (x?: number | undefined) => string;  // x has type number | undefined
```

## Non-null and non-undefined type guards
## 非 null 和非 undefined 类型保护

A property access or a function call produces a compile-time error if the object or function is of a type that includes `null` or `undefined`.
However, type guards are extended to support non-null and non-undefined checks.

如果对象或函数是一个包含 `null` 或 `undefined` 的类型，则属性访问或函数调用将产生一个运行时错误。然而，类型保护被扩展以支持非 null 和非 undefined 检查。

##### Example
##### 示例

```ts
// Compiled with --strictNullChecks
declare function f(x: number): string;
let x: number | null | undefined;
if (x) {
    f(x);  // Ok, type of x is number here
}
else {
    f(x);  // Error, type of x is number? here
}
let a = x != null ? f(x) : "";  // Type of a is string
let b = x && f(x);  // Type of b is string | 0 | null | undefined
```

Non-null and non-undefined type guards may use the `==`, `!=`, `===`, or `!==` operator to compare to `null` or `undefined`, as in `x != null` or `x === undefined`.
The effects on subject variable types accurately reflect JavaScript semantics (e.g. double-equals operators check for both values no matter which one is specified whereas triple-equals only checks for the specified value).

非 null 和 非 undefined 类型保护可以使用 `==`、`!=`、`===` 或 `!==` 运算符来比较 `null` 或 `undefined`，如 `x != null` 或 `x === undefined`。对主题变量类型的影响确切地反映了 JavaScript 的语义（例如，双等运算符检查两边的值而不管指定了哪个值而三等运算符仅检查指定了的值）。

## Dotted names in type guards
## 类型保护中的点式名称

Type guards previously only supported checking local variables and parameters.
Type guards now support checking "dotted names" consisting of a variable or parameter name followed one or more property accesses.

以前类型保护只支持检查局部变量和参数。类型保护现在支持检查由一个或多个属性访问跟随着的变量或参数名称组成的“点式名称”。

##### Example
##### 示例

```ts
interface Options {
    location?: {
        x?: number;
        y?: number;
    };
}

function foo(options?: Options) {
    if (options && options.location && options.location.x) {
        const x = options.location.x;  // Type of x is number
    }
}
```

Type guards for dotted names also work with user defined type guard functions and the `typeof` and `instanceof` operators and do not depend on the `--strictNullChecks` compiler option.

点式名称类型保护也对用户自定义类型保护函数以及 `typeof` 和 `instanceof` 起作用且不依赖于 `--strictNullChecks` 编译器选项。

A type guard for a dotted name has no effect following an assignment to any part of the dotted name.
For example, a type guard for `x.y.z` will have no effect following an assignment to `x`, `x.y`, or `x.y.z`.

当对点式名称的任何部分进行赋值时，点式名称的类型保护将不再有效。例如，在 `x.y.z` 的类型保护后跟随着对 `x`、`x.y` 或`x.y.z` 的赋值，则该类型保护失效。

示例

```ts
type X_Y_Z = { x?: { y?: { z?: number; } } };

let xyz!: X_Y_Z;
// dotted name type guard
if (xyz && xyz.x && xyz.x.y && xyz.x.y.z) {
  xyz; // ok
  xyz.x; // ok
  xyz.x.y; // ok
  xyz.x.y.z; // ok

  xyz = { x: { y: { z: 123 } } };
  xyz.x; // ok
  xyz.x.y; // Object is possibly 'undefined'
  xyz.x.y.z; // Object is possibly 'undefined'
}
```

## Expression operators
## 表达式运算符

Expression operators permit operand types to include `null` and/or `undefined` but always produce values of non-null and non-undefined types.

表达式运算符允许操作数类型包含 `null` 和/或 `undefined` 但结果总是生成非 null 和非 undefined 类型的值。

```ts
// Compiled with --strictNullChecks
function sum(a: number | null, b: number | null) {
    return a + b;  // Produces value of type number
}
```

The `&&` operator adds `null` and/or `undefined` to the type of the right operand depending on which are present in the type of the left operand, and the `||` operator removes both `null` and `undefined` from the type of the left operand in the resulting union type.

 `&&` 运算符将 `null` 和/或 `undefined` 添加到右侧操作数的类型中，具体取决于哪个出现在左侧的操作数中， `||` 运算符则将 `null` 和 `undefined` 从结果的联合类型中左侧操作数的类型中移除。

```ts
// Compiled with --strictNullChecks
interface Entity {
    name: string;
}
let x: Entity | null;
let s = x && x.name;  // s is of type string | null
let y = x || { name2: "test" };  // y is of type Entity | { name2:string; }
```

## Type widening
## 类型宽化

The `null` and `undefined` types are *not* widened to `any` in strict null checking mode.

在严格空检查模式中 `null` 和 `undefined` 类型不被宽化到 `any` 中。

```ts
let z = null;  // Type of z is null
```

In regular type checking mode the inferred type of `z` is `any` because of widening, but in strict null checking mode the inferred type of `z` is `null` (and therefore, absent a type annotation, `null` is the only possible value for `z`).

因为宽化 ，常规的类型检查模式中 `z` 的推导类型是 `any`，但在严格空检查模式中 `z` 的推导类型是 `null`（因此，缺少类型注解时， `null` 是 `z` 唯一可能的值）。

## Non-null assertion operator
## 非空断言运算符

A new `!` post-fix expression operator may be used to assert that its operand is non-null and non-undefined in contexts where the type checker is unable to conclude that fact.
Specifically, the operation `x!` produces a value of the type of `x` with `null` and `undefined` excluded.
Similar to type assertions of the forms `<T>x` and `x as T`, the `!` non-null assertion operator is simply removed in the emitted JavaScript code.

新的 `!` 后缀表达式运算符可以用来在类型检查器无法得出真实的上下文环境时断言它的操作数是非空和非 undefined 的。很明确地，`x!` 运算产生一个排除了 `null` 和 `undefined` 的 `x` 类型的值。与 `<T>x` 和 `x as T` 形式的类型断言类似，这个 `!` 非空断言运算符被从生成的 JavaScript 代码中移除。

```ts
// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
    // Throw exception if e is null or invalid entity
}

function processEntity(e?: Entity) {
    validateEntity(e);
    let s = e!.name;  // Assert that e is non-null and access name
}
```

## Compatibility
## 兼容性

The new features are designed such that they can be used in both strict null checking mode and regular type checking mode.
In particular, the `null` and `undefined` types are automatically erased from union types in regular type checking mode (because they are subtypes of all other types), and the `!` non-null assertion expression operator is permitted but has no effect in regular type checking mode. Thus, declaration files that are updated to use null- and undefined-aware types can still be used in regular type checking mode for backwards compatibility.

新特性被设计成既可以用于严格空检查模式中也可以用于常规类型检查模式中。特别地在常规类型检查模式中， `null` 和 `undefined` 类型被自动地从联合类型中擦除（因为它们是所有其它类型的子类型）， `!` 非空断言表达式运算符是允许的但没有效果。因此，更新以使用 null 和 undefined 敏感的类型的声明文件仍然可用在常规类型检查模式中以实现向后兼容。

In practical terms, strict null checking mode requires that all files in a compilation are null- and undefined-aware.

实际上，严格空检查模式要求所有编译的文件都是 null 和 undefined 敏感的。

# Control flow based type analysis
# 基于控制流的类型分析

TypeScript 2.0 implements a control flow-based type analysis for local variables and parameters.
Previously, the type analysis performed for type guards was limited to `if` statements and `?:` conditional expressions and didn't include effects of assignments and control flow constructs such as `return` and `break` statements.
With TypeScript 2.0, the type checker analyses all possible flows of control in statements and expressions to produce the most specific type possible (the *narrowed type*) at any given location for a local variable or parameter that is declared to have a union type.

TypeScript 2.0 为局部变量和参数实现了一个基于控制流的类型分析。在以前，为类型保护执行的类型仅限于 `if` 语句和 `?:` 条件表达式且不包括赋值和控制流构造（如 `return` 和 `break` 语句）的效果。在 TypeScript 2.0 中，类型检查器分析语句和表达式中所有可能的控制流以便在任何声明为联合类型的局部变量或参数的位置生成最可能的特定类型（*窄化类型*）。

##### Example
##### 示例

```ts
function foo(x: string | number | boolean) {
    if (typeof x === "string") {
        x; // type of x is string here
        x = 1;
        x; // type of x is number here
    }
    x; // type of x is number | boolean here
}

function bar(x: string | number) {
    if (typeof x === "number") {
        return;
    }
    x; // type of x is string here
}
```

Control flow based type analysis is particularly relevant in `--strictNullChecks` mode because nullable types are represented using union types:

基于控制流的类型分析在 `--strictNullChecks` 模式中特别有意义因为可以使用联合类型来表示可空类型：

```ts
function test(x: string | null) {
    if (x === null) {
        return;
    }
    x; // type of x is string in remainder of function
}
```

Furthermore, in `--strictNullChecks` mode, control flow based type analysis includes *definite assignment analysis* for local variables of types that don't permit the value `undefined`.

此外，在 `--strictNullChecks` 模式中，基于控制流的类型分析为不允许值为 `undefined` 的类型的局部变量包含*明确赋值分析*。

```ts
function mumble(check: boolean) {
    let x: number; // Type doesn't permit undefined
    x; // Error, x is undefined
    if (check) {
        x = 1;
        x; // Ok
    }
    x; // Error, x is possibly undefined
    x = 2;
    x; // Ok
}
```

# Tagged union types
# 标签化联合类型

TypeScript 2.0 implements support for tagged (or discriminated) union types.
Specifically, the TS compiler now support type guards that narrow union types based on tests of a discriminant property and furthermore extend that capability to `switch` statements.

TypeScript 2.0 实现了对标签化（可判别的）联合类型的支持。具体来说，TS 编译器现在支持基于判别式属性测试收窄联合类型的类型保护，并进一步地将该功能扩展到 `switch` 语句。

##### Example
##### 示例

```ts
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
    // In the following switch statement, the type of s is narrowed in each case clause
    // according to the value of the discriminant property, thus allowing the other properties
    // of that variant to be accessed without a type assertion.
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.width * s.height;
        case "circle": return Math.PI * s.radius * s.radius;
    }
}

function test1(s: Shape) {
    if (s.kind === "square") {
        s;  // Square
    }
    else {
        s;  // Rectangle | Circle
    }
}

function test2(s: Shape) {
    if (s.kind === "square" || s.kind === "rectangle") {
        return;
    }
    s;  // Circle
}
```

A *discriminant property type guard* is an expression of the form `x.p == v`, `x.p === v`, `x.p != v`, or `x.p !== v`, where `p` and `v` are a property and an expression of a string literal type or a union of string literal types.
The discriminant property type guard narrows the type of `x` to those constituent types of `x` that have a discriminant property `p` with one of the possible values of `v`.

*判别式属性类型保护*是一个 `x.p == v`、`x.p === v`、`x.p != v` 或 `x.p !== v` 形式的表达式，其中 `p` 和 `v` 是属性及字符串字面量类型表达式或字符串字面量的联合类型。判别式属性类型保护将 `x` 的类型收窄为 `x` 的某个组成类型，该组成类型具有判别式属性 `p` 且 `p` 是 `v` 的一个可能值。

Note that we currently only support discriminant properties of string literal types.
We intend to later add support for boolean and numeric literal types.

注意，我们当前仅支持字符串字面量类型的判别式属性。我们打算后续添加对布尔和数值字面量类型的支持。

# The `never` type
#  `never` 类型

TypeScript 2.0 introduces a new primitive type `never`.
The `never` type represents the type of values that never occur.
Specifically, `never` is the return type for functions that never return and `never` is the type of variables under type guards that are never true.

TypeScript 2.0 引入了一个新的基本类型 `never`。 `never` 类型表示那些永远不会出现的值的类型。具体而言，`never` 是那些永远不返回值的函数以及在类型保护下永远不为真的变量的类型。

The `never` type has the following characteristics:

 `never` 类型具有如下特点：

* `never` is a subtype of and assignable to every type.

  `never` 是任何类型的子类型且可以赋值给任何类型。

* No type is a subtype of or assignable to `never` (except `never` itself).

  没有什么类型是 `never` 的子类型或者可以赋值给 `never`（除了 `never` 自身）。

* In a function expression or arrow function with no return type annotation, if the function has no `return` statements, or only `return` statements with expressions of type `never`, and if the end point of the function is not reachable (as determined by control flow analysis), the inferred return type for the function is `never`.

  在一个没有返回值注解的函数表达式或箭头函数中，如果函数没有返回语句（`return`），或者仅有类型为 `never` 的表达式的返回（`return`）语句，以及如果函数的末端是不可达的（由控制流分析决定），则为该函数推导的返回类型是 `never`。

* In a function with an explicit `never` return type annotation, all `return` statements (if any) must have expressions of type `never` and the end point of the function must not be reachable.

  在一个具有显式的 `never` 返回类型注解的函数中，所有的返回（`return`）语句（如果有）必须具有 `never` 类型的表达式且函数末端必须是不可达的。

Because `never` is a subtype of every type, it is always omitted from union types and it is ignored in function return type inference as long as there are other types being returned.

因为 `never` 是所有类型的子类型，在联合类型中它总是被省略且只要有一个类型被返回它就会被在函数返回类型推断中忽略。

Some examples of functions returning `never`:

一些返回 `never` 的函数的例子：

```ts
// Function returning never must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is never
function fail() {
    return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}
```

Some examples of use of functions returning `never`:

一些使用返回 `never` 的函数的例子：

```ts
// Inferred return type is number
function move1(direction: "up" | "down") {
    switch (direction) {
        case "up":
            return 1;
        case "down":
            return -1;
    }
    return error("Should never get here");
}

// Inferred return type is number
function move2(direction: "up" | "down") {
    return direction === "up" ? 1 :
        direction === "down" ? -1 :
        error("Should never get here");
}

// Inferred return type is T
function check<T>(x: T | undefined) {
    return x || error("Undefined value");
}
```

Because `never` is assignable to every type, a function returning `never` can be used when a callback returning a more specific type is required:

因为 `never` 可以赋值给所有类型，在需要一个返回更具体的类型的回调的地方可以使用返回 `never` 的函数。

```ts
function test(cb: () => string) {
    let s = cb();
    return s;
}

test(() => "hello");
test(() => fail());
test(() => { throw new Error(); })
```

# Read-only properties and index signatures
# 只读属性和索引签名

A property or index signature can now be declared with the `readonly` modifier is considered read-only.

属性或索引签名现在可以通过使用 `readonly` 修饰符声明被认为是只读的。

Read-only properties may have initializers and may be assigned to in constructors within the same class declaration, but otherwise assignments to read-only properties are disallowed.

只读属性可以有初始化器以及可以在其类声明的构造函数中被赋值，否则对只读属性的赋值都是不允许的。

In addition, entities are *implicitly* read-only in several situations:

此外，在以下几种情形中实体是*隐式*只读的：

* A property declared with a `get` accessor and no `set` accessor is considered read-only.

  一个声明了 `get` 访问器而没有 `set` 访问器的属性被认为是只读的。

* In the type of an enum object, enum members are considered read-only properties.

  在一个枚举类型中，枚举成员被认为是只读属性。

* In the type of a module object, exported `const` variables are considered read-only properties.

  在一个模块对象中，导出的 `const` 声明的变量都被认为是只读属性。

* An entity declared in an `import` statement is considered read-only.

  在 `import` 语句中声明的实体被认为是只读的。

* An entity accessed through an ES2015 namespace import is considered read-only (e.g. `foo.x` is read-only when `foo` is declared as `import * as foo from "foo"`).

  通过 ES2015 命名空间导入方式访问的实体被认为是只读的（例如，当 `foo` 被声明为 `import * as foo from "foo"` 时其是只读的）。

##### Example
##### 示例

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}

var p1: Point = { x: 10, y: 20 };
p1.x = 5;  // Error, p1.x is read-only

var p2 = { x: 1, y: 1 };
var p3: Point = p2;  // Ok, read-only alias for p2
p3.x = 5;  // Error, p3.x is read-only
p2.x = 5;  // Ok, but also changes p3.x because of aliasing
```

```ts
class Foo {
    readonly a = 1;
    readonly b: string;
    constructor() {
        this.b = "hello";  // Assignment permitted in constructor
    }
}
```

```ts
let a: Array<number> = [0, 1, 2, 3, 4];
let b: ReadonlyArray<number> = a;
b[5] = 5;      // Error, elements are read-only
b.push(5);     // Error, no push method (because it mutates array)
b.length = 3;  // Error, length is read-only
a = b;         // Error, mutating methods are missing
```

# Specifying the type of `this` for functions
# 为函数指定 `this` 的类型

Following up on specifying the type of `this` in a class or an interface, functions and methods can now declare the type of `this` they expect.

紧跟在类或接口中为 `this` 指定类型之后，函数和方法现在可以声明它们所期望的 `this` 的类型了。

By default the type of `this` inside a function is `any`.
Starting with TypeScript 2.0, you can provide an explicit `this` parameter.
`this` parameters are fake parameters that come first in the parameter list of a function:

默认地，函数内部的 `this` 的类型是 `any`。自 TypeScript 2.0 起，你可以显式地提供一个 `this` 参数。`this` 参数是一个伪参数，它在函数参数列表的第一个位置：

```ts
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```

## `this` parameters in callbacks
## 回调中的 `this` 参数

Libraries can also use `this` parameters to declare how callbacks will be invoked.

库也可以使用 `this` 参数来声明回调将会如何被调用。

##### Example
##### 示例

```ts
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

`this: void` means that `addClickListener` expects `onclick` to be a function that does not require a `this` type.

`this: void` 意味着 `addClickListener` 期望 `onclick` 是一个不需要 `this` 类型的函数。

Now if you annotate calling code with `this`:

现在如果你注解了调用代码的 `this`：

```ts
class Handler {
    info: string;
    onClickBad(this: Handler, e: Event) {
        // oops, used this here. using this callback would crash at runtime
        this.info = e.message;
    };
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!
```

## `--noImplicitThis`
## `--noImplicitThis` 编译选项

A new flag is also added in TypeScript 2.0 to flag all uses of `this` in functions without an explicit type annotation.

TypeScript 2.0 还添加了一个新的标志，用于在没有显式类型注解的函数中标记所有对 `this` 的使用。

# Glob support in `tsconfig.json`
# `tsconfig.json` 中的 glob 支持

Glob support is here!! Glob support has been [one of the most requested features](https://github.com/Microsoft/TypeScript/issues/1927).

glob 支持就在这里！glob 支持已成为[最需要的特性之一](https://github.com/Microsoft/TypeScript/issues/1927)。

Glob-like file patterns are supported two properties `"include"` and `"exclude"`.

类 glob 的文件模式都支持两个属性—— `"include"` 和 `"exclude"`。

##### Example
##### 示例

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "outFile": "../../built/local/tsc.js",
        "sourceMap": true
    },
    "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```

The supported glob wildcards are:

受支持的 glob 通配符：

* `*` matches zero or more characters (excluding directory separators)

  `*` 匹配零个或多个字符（除了目录分隔符）

* `?` matches any one character (excluding directory separators)

  `?` 匹配任何单个字符（除了目录分隔符）

* `**/` recursively matches any subdirectory

  `**/` 递归地匹配任何子目录

If a segment of a glob pattern includes only `*` or `.*`, then only files with supported extensions are included (e.g. `.ts`, `.tsx`, and `.d.ts` by default with `.js` and `.jsx` if `allowJs` is set to true).

如果一个 glob 模式中的某一段（以目录分隔符“/”分隔）仅包含 `*` 或 `.*`，那么只有扩展名受支持的文件会被包含进去（例如，`.ts`、`.tsx` 和 `.d.ts`，如果设置 `allowJs` 为 true，则会默认包含 `.js` 和 `.jsx` 文件）。

If the `"files"` and `"include"` are both left unspecified, the compiler defaults to including all TypeScript (`.ts`, `.d.ts` and `.tsx`) files in the containing directory and subdirectories except those excluded using the `"exclude"` property. JS files (`.js` and `.jsx`) are also included if `allowJs` is set to true.

如果 `“files”` 和 `“include”` 都未指定，则编译器默认地把除了使用 `“exclude”` 属性排除的文件外的所有当前目录及子目录的 TypeScript 文件包括到编译中。如果设置 `allowJs` 为 true 则 JS 文件（`.js` 和 `.jsx`）也会被包括到编译中。

If the `"files"` or `"include"` properties are specified, the compiler will instead include the union of the files included by those two properties.
Files in the directory specified using the `"outDir"` compiler option are always excluded unless explicitly included via the `"files"` property (even when the "`exclude`" property is specified).

如果 `“files”` 或 `“include”` 属性都指定了，则编译器将会把这两个属性所指定的文件都包括到编译中。 `“outDir”` 编译器选项所指定的目录中的文件总是被排除在编译之外，除非显式地使用 `“files”` 属性设置要将 `“outDir”` 目录中文件包括到编译中（即使 `“exclude”` 属性指定了排除 `“outDir”` 目录中的文件，这些文件也会被包括到编译中）。

Files included using `"include"` can be filtered using the `"exclude"` property.
However, files included explicitly using the `"files"` property are always included regardless of `"exclude"`.
The `"exclude"` property defaults to excluding the `node_modules`, `bower_components`, and `jspm_packages` directories when not specified.

使用 `“include”` 包含的文件可以使用 `“exclude”` 属性对其进行过滤。然而，显式地使用 `“files”` 属性包括的文件总是会被包括在编译中而不管 `“exclude”` 是如何设置的。当未指定 `“exclude”` 属性时，默认把 `node_modules`、`bower_components` 和 `jspm_packages` 目录排除。

# Module resolution enhancements: BaseUrl, Path mapping, rootDirs and tracing
# 模块解决方案增强：基 URL，路径映射，根目录和跟踪

TypeScript 2.0 provides a set of additional module resolution knops to *inform* the compiler where to find declarations for a given module.

TypeScript 2.0 提供了一套额外的模块解决方案以告知编译器到哪里去为给定的模块查找声明。

See [Module Resolution](http://www.typescriptlang.org/docs/handbook/module-resolution.html) documentation for more details.

请查看[模块解决方案](http://www.typescriptlang.org/docs/handbook/module-resolution.html)的文档以获取更多详细信息。

## Base URL
## 基 URL

Using a `baseUrl` is a common practice in applications using AMD module loaders where modules are "deployed" to a single folder at run-time.
All module imports with non-relative names are assumed to be relative to the `baseUrl`.

在使用 AMD 模块加载器的应用程序中使用 `baseUrl` 是常见的实践，其中模块会在运行时被“部署”到一个文件夹中。使用非相对路径名称导入的模块都被认为其路径是相对于 `baseUrl` 的。

##### Example
##### 示例

```json
{
  "compilerOptions": {
    "baseUrl": "./modules"
  }
}
```

Now imports to `"moduleA"` would be looked up in `./modules/moduleA`

现在对 `“moduleA”` 的导入将会在 `./modules/moduleA` 中查找。

```ts
import A from "moduleA";
```

## Path mapping
## 路径映射

Sometimes modules are not directly located under *baseUrl*.
Loaders use a mapping configuration to map module names to files at run-time, see [RequireJs documentation](http://requirejs.org/docs/api.html#config-paths) and [SystemJS documentation](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#map-config).

有时模块并不是直接位于*基 URL*目录之下。加载器在运行时使用映射配置来把模块名称映射到文件，请看 [RequireJs 文档](http://requirejs.org/docs/api.html#config-paths) 和 [SystemJS 文档](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#map-config)。

The TypeScript compiler supports the declaration of such mappings using `"paths"` property in `tsconfig.json` files.

TypeScript 编译器支持 `tsconfig.json` 文件中的 `“paths”` 属性声明的这种映射。

##### Example
##### 示例

For instance, an import to a module `"jquery"` would be translated at runtime to `"node_modules/jquery/dist/jquery.slim.min.js"`.

例如，对 `“jquery”` 模块的导入在运行时将会被转换为 `“node_modules/jquery/dist/jquery.slim.min.js”`。

```json
{
  "compilerOptions": {
    "baseUrl": "./node_modules",
    "paths": {
      "jquery": ["jquery/dist/jquery.slim.min"]
    }
}
```

Using `"paths"` also allow for more sophisticated mappings including multiple fall back locations.
Consider a project configuration where only some modules are available in one location, and the rest are in another.

使用 `"paths"` 还允许更复杂的映射，包括多回退位置。想象一个项目配置，其中只有某些模块在某个位置可用，其它的模块在别的位置。

## Virtual Directories with `rootDirs`
## 使用 `rootDirs` 配置虚拟目录

Using 'rootDirs', you can inform the compiler of the *roots* making up this "virtual" directory;
and thus the compiler can resolve relative modules imports within these "virtual" directories *as if* were merged together in one directory.

使用 “rootDirs” 你可以告知编译器这个*根目录*组成了这个“虚拟”的目录；因此编译器可以解析在这些“虚拟”目录中的相对模块导入*仿佛*它们被合并到了一个目录里。

##### Example
##### 示例

Given this project structure:

给定这样的项目结构：

```tree
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

A build step will copy the files in `/src/views` and `/generated/templates/views` to the same directory in the output.
At run-time, a view can expect its template to exist next to it, and thus should import it using a relative name as `"./template"`.

构建的某一步将会复制 `/src/views` 和 `/generated/templates/views` 目录中的文件到输出中的相同目录里。在运行时，一个视图可以期望它的模板就存在于其身旁，因此应该使用一个 `“./template”` 这样的相对名称来导入它。

`"rootDirs"` specify a list of *roots* whose contents are expected to merge at run-time.
So following our example, the `tsconfig.json` file should look like:

`“rootDirs` 指定了一系列的*根目录*，它们的内容都期望在运行时被合并。所以顺着我们的例子，这个 `tsconfig.json` 文件应该看起来是这样的：

```json
{
  "compilerOptions": {
    "rootDirs": [
      "src/views",
      "generated/templates/views"
    ]
  }
}
```

## Tracing module resolution
## 追踪模块解决方案

`--traceResolution` offers a handy way to understand how modules have been resolved by the compiler.

`--traceResolution` 选项提供了一个便捷的方式来理解模块是如何被编译器解析的。

```shell
tsc --traceResolution
```

# Shorthand ambient module declarations
# 速记周围模块声明

If you don't want to take the time to write out declarations before using a new module, you can now just use a shorthand declaration to get started quickly.

如果你不想在使用一个新的模块时花费时间去写出声明，你现在可以仅使用速记式的声明来快速开始。

##### declarations.d.ts

```ts
declare module "hot-new-module";
```

All imports from a shorthand module will have the any type.

所有来自速记式模块的导入都将是 any 类型：

```ts
import x, {y} from "hot-new-module";
x(y);
```

# Wildcard character in module names
# 模块名称中的通配符

Importing none-code resources using module loaders extension (e.g. [AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md) or [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/creating-plugins.md)) has not been easy before;
previously an ambient module declaration had to be defined for each resource.

在以前使用模块加载器扩展（例如，[AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md) 或 [SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/creating-plugins.md)）来加载非代码资源并不容易；以前必须为每个资源定义一个周围模块声明。

TypeScript 2.0 supports the use of the wildcard character (`*`) to declare a "family" of module names;
this way, a declaration is only required once for an extension, and not for every resource.

TypeScript 2.0 支持使用通配符（`*`）来声明一系列的模块名称；这种方式下，一个文件扩展名只需要一个声明而不必为每个资源文件都声明一次。

##### Example
##### 示例

```ts
declare module "*!text" {
    const content: string;
    export default content;
}
// Some do it the other way around.
declare module "json!*" {
    const value: any;
    export default value;
}
```

Now you can import things that match `"*!text"` or `"json!*"`.

现在你可以导入匹配 `“*!text”` 和 `“json!*”` 的东西。

```ts
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

Wildcard module names can be even more useful when migrating from an un-typed code base.
Combined with Shorthand ambient module declarations, a set of modules can be easily declared as `any`.

在从一个非类型化的代码库迁移过来时通配符模块名称会显得尤其有用。与速记式周围模块声明结合使用，一组模块可以很容易的被声明为 `any`。

##### Example
##### 示例

```ts
declare module "myLibrary/*";
```

All imports to any module under `myLibrary` would be considered to have the type `any` by the compiler;
thus, shutting down any checking on the shapes or types of these modules.

对 `myLibrary` 目录下的任何模块的所有导入都会被编译器认为具有 `any` 类型；因此，关闭了对这些模块的外形和类型的任何检查。

```ts
import { readFile } from "myLibrary/fileSystem/readFile`;

readFile(); // readFile is 'any'
```

# Support for UMD module definitions
# 支持 UMD 模块定义

Some libraries are designed to be used in many module loaders, or with no module loading (global variables).
These are known as [UMD](https://github.com/umdjs/umd) or [Isomorphic](http://isomorphic.net) modules.
These libraries can be accessed through either an import or a global variable.

某些模块被设计用于多种模块加载器，或者不使用模块加载（全局变量）。这些被称为 [UMD](https://github.com/umdjs/umd) 或 [Isomorphic](http://isomorphic.net) 模块。可以通过导入或者全局变量访问这些库。

For example:

例如：

##### math-lib.d.ts

```ts
export const isPrime(x: number): boolean;
export as namespace mathLib;
```

The library can then be used as an import within modules:

这个库可以在模块中作为一个导入来使用：

```ts
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2); // ERROR: can't use the global definition from inside a module
```

It can also be used as a global variable, but only inside of a script.
(A script is a file with no imports or exports.)

它也可以被用作全局变量，但仅限于脚本文件中使用（一个脚本文件是不包含任何导入或导出的文件）。

```ts
mathLib.isPrime(2);
```

# Optional class properties
# 可选的类属性

Optional properties and methods can now be declared in classes, similar to what is already permitted in interfaces.

现在可以在类中声明可选的属性和方法了，类似于接口中所允许的那样。

##### Example
##### 示例

```ts
class Bar {
    a: number;
    b?: number;
    f() {
        return 1;
    }
    g?(): number;  // Body of optional method can be omitted
    h?() {
        return 2;
    }
}
```

When compiled in `--strictNullChecks` mode, optional properties and methods automatically have `undefined` included in their type. Thus, the `b` property above is of type `number | undefined` and the `g` method above is of type `(() => number) | undefined`.
Type guards can be used to strip away the `undefined` part of the type:

当使用 `--strictNullChecks` 模式编译时，可选属性和方法自动地将 `undefined` 添加到它们的类型中。因此，上面的 `b` 属性是 `number | undefined` 类型而 `g` 方法是 `(() => number) | undefined` 类型。类型保护可用于去掉类型中的 `undefined` 部分：

```ts
function test(x: Bar) {
    x.a;  // number
    x.b;  // number | undefined
    x.f;  // () => number
    x.g;  // (() => number) | undefined
    let f1 = x.f();            // number
    let g1 = x.g && x.g();     // number | undefined
    let g2 = x.g ? x.g() : 0;  // number
}
```

# Private and Protected Constructors
# 私有的和受保护的构造函数

A class constructor may be marked `private` or `protected`.
A class with private constructor cannot be instantiated outside the class body, and cannot be extended.
A class with protected constructor cannot be instantiated outside the class body, but can be extended.

类的构造函数可以标志为 `private` 或 `protected`。一个具有私有的构造函数的类不能在该类主体外实例化以及被扩展。一个具有受保护的构造函数的类不能在该类主体外实例化但可以被扩展。

##### Example
##### 示例

```ts
class Singleton {
    private static instance: Singleton;

    private constructor() { }

    static getInstance() {
        if (!Singleton.instance) {
            Singleton.instance = new Singleton();
        }
        return Singleton.instance;
    }
}

let e = new Singleton(); // Error: constructor of 'Singleton' is private.
let v = Singleton.getInstance();
```

# Abstract properties and accessors
# 抽象属性和访问器

An abstract class can declare abstract properties and/or accessors.
Any sub class will need to declare the abstract properties or be marked as abstract.
Abstract properties cannot have an initializer.
Abstract accessors cannot have bodies.

一个抽象类可以声明抽象属性和/或访问器。抽象类的任何子类需要声明这些抽象属性或者子类被声明为抽象的。抽象属性不能有初始化器。抽象访问器不能有实现主体。

##### Example
##### 示例

```ts
abstract class Base {
    abstract name: string;
    abstract get value();
    abstract set value(v: number);
}

class Derived extends Base {
    name = "derived";

    value = 1;
}
```

# Implicit index signatures
# 隐式的索引签名

An object literal type is now assignable to a type with an index signature if all known properties in the object literal are assignable to that index signature. This makes it possible to pass a variable that was initialized with an object literal as parameter to a function that expects a map or dictionary:

现在一个对象字面量类型可以赋值给一个带索引签名的类型仅当该对象字面量中的所有已知属性都可以赋值给那个索引签名。这使得把用对象字面量初始化的变量作为参数传递给一个参数类型为映射或字典的函数成为可能。

```ts
function httpService(path: string, headers: { [x: string]: string }) { }

const headers = {
    "Content-Type": "application/x-www-form-urlencoded"
};

httpService("", { "Content-Type": "application/x-www-form-urlencoded" });  // Ok
httpService("", headers);  // Now ok, previously wasn't
```

# Including built-in type declarations with `--lib`
# 使用 `--lib` 来包含内建类型的声明

Getting to ES6/ES2015 built-in API declarations were only limited to `target: ES6`.
Enter `--lib`; with `--lib` you can specify a list of built-in API declaration groups that you can chose to include in your project.
For instance, if you expect your runtime to have support for `Map`, `Set` and `Promise` (e.g. most evergreen browsers today), just include `--lib es2015.collection,es2015.promise`.
Similarly you can exclude declarations you do not want to include in your project, e.g. DOM if you are working on a node project using `--lib es5,es6`.

过去为了获得 ES6/ES2015 内建 API 声明只能通过设置 `target: ES6` 来实现。如今使用 `--lib` 你可以选择指定一系列的内建 API 声明来将它们包含进你的项目中。例如，你希望你的运行时支持 `Map`、`Set` 和 `Promise`（例如，当今最流行的浏览器），就包含 `--lib es2015.collection,es2015.promise` 即可。类似地你可以排除你不想要包含进项目中的声明，例如 DOM，如果你的项目是 node 项目则使用`--lib es5,es6`。

Here is a list of available API groups:

以下是可用的 API 组列表：

* dom
* webworker
* es5
* es6 / es2015
* es2015.core
* es2015.collection
* es2015.iterable
* es2015.promise
* es2015.proxy
* es2015.reflect
* es2015.generator
* es2015.symbol
* es2015.symbol.wellknown
* es2016
* es2016.array.include
* es2017
* es2017.object
* es2017.sharedmemory
* scripthost

##### Example
##### 示例

```bash
tsc --target es5 --lib es5,es2015.promise
```

```json
"compilerOptions": {
    "lib": ["es5", "es2015.promise"]
}
```

# Flag unused declarations with `--noUnusedParameters` and `--noUnusedLocals`
# 使用 `--noUnusedParameters` 和 `--noUnusedLocals` 来标记未使用的声明

TypeScript 2.0 has two new flags to help you maintain a clean code base.
`--noUnusedParameters` flags any unused function or method parameters errors.
`--noUnusedLocals` flags any unused local (un-exported) declaration like variables, functions, classes, imports, etc...
Also, unused private members of a class would be flagged as errors under `--noUnusedLocals`.

TypeScript 2.0 有两个新的标志可以帮助你维护一个干净的代码库。`--noUnusedParameters` 标记任何未使用的函数或方法的参数为错误。`--noUnusedLocals` 标记任何未使用的局部（未导出的）声明，如变量，函数，类，导入等。同样地，在 `--noUnusedLocals` 模式下类中未使用的私有成员也将会被标记为错误。

##### Example
##### 示例

```ts
import B, { readFile } from "./b";
//     ^ Error: `B` declared but never used
readFile();


export function write(message: string, args: string[]) {
    //                                 ^^^^  Error: 'arg' declared but never used.
    console.log(message);
}
```

Parameters declaration with names starting with `_` are exempt from the unused parameter checking.
e.g.:

使用 `_` 作为名称开头声明的参数不受未使用参数检查的影响。例如：

```ts
function returnNull(_a) { // OK
    return null;
}
```

# Module identifiers allow for `.js` extension
# 模块标识符允许 `.js` 扩展

Before TypeScript 2.0, a module identifier was always assumed to be extension-less;
for instance, given an import as `import d from "./moduleA.js"`, the compiler looked up the definition of `"moduleA.js"` in `./moduleA.js.ts` or `./moduleA.js.d.ts`.
This made it hard to use bundling/loading tools like [SystemJS](https://github.com/systemjs/systemjs) that expect URI's in their module identifier.

在 TypeScript 2.0 以前，模块标识符总是被认为是不带扩展的；例如，给定一个 `import d from "./moduleA.js"` 这样的导入，编译器将在 `./moduleA.js.ts` 或 `./moduleA.js.d.ts` 中查找 `"moduleA.js"` 的定义。这使得很难使用像 [SystemJS](https://github.com/systemjs/systemjs) 这样的捆绑/加载工具（如 ），这些工具在模块标识符中需要 URI。

With TypeScript 2.0, the compiler will look up definition of `"moduleA.js"` in  `./moduleA.ts` or `./moduleA.d.t`.

在 TypeScript 2.0 中，编译器将会在 `./moduleA.ts` 或 `./moduleA.d.t` 中查找 `“moduleA.js”` 的定义。

# Support 'target : es5' with 'module: es6'
# 支持模块为 es6（'module: es6'） 的 es5 编译目标（'target : es5'） 

Previously flagged as an invalid flag combination, `target: es5` and 'module: es6' is now supported.
This should facilitate using ES2015-based tree shakers like [rollup](https://github.com/rollup/rollup).

以前被标记为无效的标志组合 `target: es5` 和 'module: es6' 现在得到了支持（是有效的组合了）。这应该有助于使用基于 ES2015 的摇树器（如 [rollup](https://github.com/rollup/rollup)）。

![摇树](../../src/assets/imgs/Tree-Shaking.gif)

<center>Tree-Shaking</center>

✍ <u>上图直观的展示了 Tree-Shaking 的本意——摇树，将果子抖落。前端打包中的 Tree-Shaking 即借用此意，指通过工具把 JS 文件中没有用到的代码“抖掉”（剔除掉），是一个性能优化手段。比如使用 webpack 打包，有一个入口文件，它依赖很多的模块。那么这个入口文件就相当于一棵树的主干，而其所依赖的模块则相当于树枝，树枝上有叶子和果子——模块里面的变量，函数，类等代码。有时虽然依赖某个模块，但是只使用其中的某些功能，那么可以通过 Tree-Shaking 操作将没有用到的代码剔除，以实现精简打包输出的目的，加快前端资源加载。</u>

# Trailing commas in function parameter and argument lists
# 函数形参<sup>①</sup>和实参<sup>②</sup>列表尾部的逗号

Trailing comma in function parameter and argument lists are now allowed.
This is an implementation for a [Stage-3 ECMAScript proposal](https://jeffmo.github.io/es-trailing-function-commas/) that emits down to valid ES3/ES5/ES6.

现在允许函数形参和实参列表尾部有逗号。这是 [阶段 3 中的 ECMAScript 提案](https://jeffmo.github.io/es-trailing-function-commas/) 的一个实现，以便向下降级生成有效的 ES3/ES5/ES6 代码。

✍ <u>形参（parameter）指函数定义时的“参数”，而实参（argument）指函数调用时实际传递的“参数”。</u>

##### Example
##### 示例

```ts
// bar 和 baz 为形参。
function foo(
  bar: Bar,
  baz: Baz, // trailing commas are OK in parameter lists
) {
  // Implementation...
}

let x:Bar,y:Baz; 

// x 和 y 为实参。
foo(
  x,
  y, // and in argument lists
); 
```

# New `--skipLibCheck`
# 新的 `--skipLibCheck` 编译器选项

TypeScript 2.0 adds a new `--skipLibCheck` compiler option that causes type checking of declaration files (files with extension `.d.ts`) to be skipped.
When a program includes large declaration files, the compiler spends a lot of time type checking declarations that are already known to not contain errors, and compile times may be significantly shortened by skipping declaration file type checks.

TypeScript 2.0 添加了一个新的编译器选项 `--skipLibCheck`，它会导致跳过对声明文件（扩展名为 `.d.ts` 的文件）的类型检查。当一个程序包含了大量的声明文件，编译器需要花费许多时间用于声明的类型检查，而这些声明却又是明确知道不含有错误的，那么跳过对这些声明文件的类型检查将极大地缩短编译时间。

Since declarations in one file can affect type checking in other files, some errors may not be detected when `--skipLibCheck` is specified.
For example, if a non-declaration file augments a type declared in a declaration file, errors may result that are only reported when the declaration file is checked.
However, in practice such situations are rare.

由于一个文件中的声明可以影响到别的文件中的类型检查，当指定了 `--skipLibCheck` 的时候某些错误可能不会被检测到。例如，如果一个非声明文件（非 `.d.ts` 类型文件）引入了一个在声明文件中声明的类型，那么只有在检查该声明文件时才会报告错误（如果有错误的话）。然而，实际中这类情形非常少见。

# Allow duplicate identifiers across declarations
# 允许跨声明的重复标识符

This has been one common source of duplicate definition errors.
Multiple declaration files defining the same members on interfaces.

这是重复定义错误的一个常见来源。在接口上定义相同的成员的多声明文件。

TypeScript 2.0 relaxes this constraint and allows duplicate identifiers across blocks, as long as they have *identical* types.

TypeScript 2.0 放松了这个约束并允许跨块的重复标识符，只要它们具有*相同的*类型。

Within the same block duplicate definitions are still disallowed.

相同块中的重复定义仍然是不允许的。

##### Example
##### 示例

```ts
interface Error {
    stack?: string;
}


interface Error {
    code?: string;
    path?: string;
    stack?: string;  // OK
}

```

# New `--declarationDir`
# 新的 `--declarationDir` 编译器选项

`--declarationDir` allows for generating declaration files in a different location than JavaScript files.

`--declarationDir` 允许生成声明文件到不同于 JavaScript 文件所在的目录中。