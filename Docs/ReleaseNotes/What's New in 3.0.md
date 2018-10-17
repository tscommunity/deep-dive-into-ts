# Project References
# 项目引用

TypeScript 3.0 introduces a new concept of project references. Project references allow TypeScript projects to depend on other TypeScript projects - specifically, allowing `tsconfig.json` files to reference other `tsconfig.json` files. Specifying these dependencies makes it easier to split your code into smaller projects, since it gives TypeScript (and tools around it) a way to understand build ordering and output structure.

TypeScript 3.0 引入了一个新的项目引用概念。项目引用允许 TypeScript 项目依赖于别的 TypeScript 项目——特别的，允许 `tsconfig.json` 文件引用别的 `tsconfig.json` 文件。指定这些依赖使得分割你的代码到更小的项目中变得更容易，因为它给了 TypeScript（以及它周边的工具） 一个方式去理解构建顺序和输出结构。

TypeScript 3.0 introduces also introducing a new mode for tsc, the `--build` flag, that works hand-in-hand with project references to enable faster TypeScript builds.

TypeScript 3.0 还为 tsc 引入了一个新的模式——`--build` 标志，它与项目引用协同工作来使得 TypeScript 构建更快。

See [Project References handbook page](../Project%20References.md) for more documentation.

到 [Project References handbook page](../Project%20References.md) 查看更多文档。

# Tuples in rest parameters and spread expressions
# 剩余参数和扩展表达式中的元组

TypeScript 3.0 adds support to multiple new capabilities to interact with function parameter lists as tuple types.
TypeScript 3.0 adds support for:

TypeScript 3.0 添加了多个新功能的支持以与函数参数列表作为元组类型进行交互。TypeScript 3.0 为以下功能添加支持：

* [Expansion of rest parameters with tuple types into discrete parameters.](#rest-parameters-with-tuple-types)
  
  用元组将剩余参数展开为离散的参数。

* [Expansion of spread expressions with tuple types into discrete arguments.](#spread-expressions-with-tuple-types)

  用元组将扩展表达式展开为离散的参数。

* [Generic rest parameters and corresponding inference of tuple types.](#generic-rest-parameters)

  泛型剩余参数及对应的元组类型推导。

* [Optional elements in tuple types.](#optional-elements-in-tuple-types)

  元组中的可选元素。
  
* [Rest elements in tuple types.](#rest-elements-in-tuple-types)

  元组中的剩余元素。

With these features it becomes possible to strongly type a number of higher-order functions that transform functions and their parameter lists.

通过这些特性，强类型化一些高阶函数——转换函数及其参数列表，成为了可能。

## Rest parameters with tuple types
## 剩余参数与元组类型

When a rest parameter has a tuple type, the tuple type is expanded into a sequence of discrete parameters.
For example the following two declarations are equivalent:

当一个剩余参数是元组类型时，元组类型被展开为一个离散的参数序列。例如以下的两个声明是等价的：

```ts
declare function foo(...args: [number, string, boolean]): void;
```

```ts
declare function foo(args_0: number, args_1: string, args_2: boolean): void;
```

## Spread expressions with tuple types
## 展开表达式与元组类型

When a function call includes a spread expression of a tuple type as the last argument, the spread expression corresponds to a sequence of discrete arguments of the tuple element types.

当一个函数调用包含一个元组类型的展开表达式作为最后的参数时，这个展开表达式对应于一个该元组元素类型的离散参数序列。

Thus, the following calls are equivalent:

故以下调用都是等价的：

```ts
const args: [number, string, boolean] = [42, "hello", true];
foo(42, "hello", true);
foo(args[0], args[1], args[2]);
foo(...args);
```

## Generic rest parameters
## 泛型剩余参数

A rest parameter is permitted to have a generic type that is constrained to an array type, and type inference can infer tuple types for such generic rest parameters. This enables higher-order capturing and spreading of partial parameter lists:

一个剩余参数是允许为泛型类型的，该泛型被约束为数组类型，并且类型推导能够为此泛型剩余参数推导出元组类型。这使得更高阶的捕获和展开部分参数列表：

##### Example
##### 示例

```ts
declare function bind<T, U extends any[], V>(f: (x: T, ...args: U) => V, x: T): (...args: U) => V;

declare function f3(x: number, y: string, z: boolean): void;

const f2 = bind(f3, 42);  // (y: string, z: boolean) => void
const f1 = bind(f2, "hello");  // (z: boolean) => void
const f0 = bind(f1, true);  // () => void

f3(42, "hello", true);
f2("hello", true);
f1(true);
f0();
```

In the declaration of `f2` above, type inference infers types `number`, `[string, boolean]` and `void` for `T`, `U` and `V` respectively.

在上述 `f2` 的声明中，类型推断分别为 `T`， `U` 和 `V` 推断出类型 `number`， `[string, boolean]` 以及 `void`。

Note that when a tuple type is inferred from a sequence of parameters and later expanded into a parameter list, as is the case for `U`, the original parameter names are used in the expansion (however, the names have no semantic meaning and are not otherwise observable).

注意，当一个元组类型是从一个参数序列推断而来并且后续被展开到一个参数列表，如 `U` 所示，原始参数的名称被用在展开式中（但是，这些名称没有语义也不可观察）。

## Optional elements in tuple types
## 元组类型中的可选元素

Tuple types now permit a `?` postfix on element types to indicate that the element is optional:

现在元组类型允许元素后带后缀 `?` 以表明该元素是可选的：

##### Example
##### 示例

```ts
let t: [number, string?, boolean?];
t = [42, "hello", true];
t = [42, "hello"];
t = [42];
```

In `--strictNullChecks` mode, a `?` modifier automatically includes `undefined` in the element type, similar to optional parameters.

在启用 `--strictNullChecks` 的模式中， `?` 修饰符自动的将 `undefined` 包含在元素类型里，类似于可选参数。

A tuple type permits an element to be omitted if it has a postfix `?` modifier on its type and all elements to the right of it also have `?` modifiers.

在一个元组类型中，当一个元素的类型有 `?` 后缀修饰符并且从该元素起至右边的所有元素也同样有 `?` 修饰符，则该元素允许被忽略。

When tuple types are inferred for rest parameters, optional parameters in the source become optional tuple elements in the inferred type.

当元组类型被推断为剩余参数，原来的可选参数在推断出的类型中变为可选的元组元素。

The `length` property of a tuple type with optional elements is a union of numeric literal types representing the possible lengths.
For example, the type of the `length` property in the tuple type `[number, string?, boolean?]` is `1 | 2 | 3`.

一个带可选元素的元组类型的 `length` 属性是一个数值字面量联合类型——对应于该元组可能的长度值。例如，元组类型 `[number, string?, boolean?]` 的 `length` 属性是 `1 | 2 | 3`。

### Rest elements in tuple types
### 元组类型中的剩余元素

The last element of a tuple type can be a rest element of the form `...X`, where `X` is an array type.
A rest element indicates that the tuple type is open-ended and may have zero or more additional elements of the array element type.
For example, `[number, ...string[]]` means tuples with a `number` element followed by any number of `string` elements.

元组的最后一个元素可以是 `...X` 形式的剩余元素， `X` 是一个数组类型。一个剩余元素表明该元组类型是尾开放的且可以有零个或多个额外的，类型为该数组元素的类型的元素。例如， `[number, ...string[]]` 意味着元组有一个 `number` 类型的元素，该元素紧跟着任意多个 `string` 类型的元素。

##### Example
##### 示例

```ts
let t: [number, ...string[]];
t = [1];
t = [1, ""];
t = [1, "", ""];

function tuple<T extends any[]>(...args: T): T {
    return args;
}

const numbers: number[] = getArrayOfNumbers();
const t1 = tuple("foo", 1, true);  // [string, number, boolean]
const t2 = tuple("bar", ...numbers);  // [string, ...number[]]
```

The type of the `length` property of a tuple type with a rest element is `number`.

一个有剩余元素的元组类型的 `length` 属性的类型是 `number`。

# New `unknown` top type
# 新的 `unknown` 顶级类型

TypeScript 3.0 introduces a new top type `unknown`.
`unknown` is the type-safe counterpart of `any`.
Anything is assignable to `unknown`, but `unknown` isn't assignable to anything but itself and `any` without a type assertion or a control flow based narrowing.
Likewise, no operations are permitted on an `unknown` without first asserting or narrowing to a more specific type.

TypeScript 3.0 引入了一个新的顶级类型 `unknown`。`unknown` 是 `any` 的一个类型安全的版本。任何值都可以赋值给 `unknown` 类型，但在没有类型断言或者基于收窄的控制流的前提下， `unknown` 类型不能赋值给除了其和 `any` 类型以外的任何类型。同理，未事先断言或者收窄为一个更具体的类型，不允许对
 `unknown` 类型进行操作。

##### Example

```ts
// In an intersection everything absorbs unknown
// 在取交中所有类型都吞并 unknown

type T00 = unknown & null;  // null
type T01 = unknown & undefined;  // undefined
type T02 = unknown & null & undefined;  // null & undefined (which becomes never)
type T03 = unknown & string;  // string
type T04 = unknown & string[];  // string[]
type T05 = unknown & unknown;  // unknown
type T06 = unknown & any;  // any

// In a union an unknown absorbs everything
// 在取并中 unknown 吞并所有类型

type T10 = unknown | null;  // unknown
type T11 = unknown | undefined;  // unknown
type T12 = unknown | null | undefined;  // unknown
type T13 = unknown | string;  // unknown
type T14 = unknown | string[];  // unknown
type T15 = unknown | unknown;  // unknown
type T16 = unknown | any;  // any

// Type variable and unknown in union and intersection
// 取并和取交中的类型变量和 unknown

type T20<T> = T & {};  // T & {}
type T21<T> = T | {};  // T | {}
type T22<T> = T & unknown;  // T
type T23<T> = T | unknown;  // unknown

// unknown in conditional types
// 条件类型中的 unknow

type T30<T> = unknown extends T ? true : false;  // Deferred
type T31<T> = T extends unknown ? true : false;  // Deferred (so it distributes)
type T32<T> = never extends T ? true : false;  // true
type T33<T> = T extends never ? true : false;  // Deferred

// keyof unknown

type T40 = keyof any;  // string | number | symbol
type T41 = keyof unknown;  // never

// Only equality operators are allowed with unknown
// 只允许对 unknown 使用相等操作符

function f10(x: unknown) {
    x == 5;
    x !== 10;
    x >= 0;  // Error
    x + 1;  // Error
    x * 2;  // Error
    -x;  // Error
    +x;  // Error
}

// No property accesses, element accesses, or function calls
// 不可访问属性，元素或者进行函数调用。

function f11(x: unknown) {
    x.foo;  // Error
    x[5];  // Error
    x();  // Error
    new x();  // Error
}

// typeof, instanceof, and user defined type predicates

declare function isFunction(x: unknown): x is Function;

function f20(x: unknown) {
    if (typeof x === "string" || typeof x === "number") {
        x;  // string | number
    }
    if (x instanceof Error) {
        x;  // Error
    }
    if (isFunction(x)) {
        x;  // Function
    }
}

// Homomorphic mapped type over unknown

type T50<T> = { [P in keyof T]: number };
type T51 = T50<any>;  // { [x: string]: number }
type T52 = T50<unknown>;  // {}

// Anything is assignable to unknown
// 任何值都能够赋值给 unknow

function f21<T>(pAny: any, pNever: never, pT: T) {
    let x: unknown;
    x = 123;
    x = "hello";
    x = [1, 2, 3];
    x = new Error();
    x = x;
    x = pAny;
    x = pNever;
    x = pT;
}

// unknown assignable only to itself and any
// unknown 只能够赋值给自身类型和 any 类型

function f22(x: unknown) {
    let v1: any = x;
    let v2: unknown = x;
    let v3: object = x;  // Error
    let v4: string = x;  // Error
    let v5: string[] = x;  // Error
    let v6: {} = x;  // Error
    let v7: {} | null | undefined = x;  // Error
}

// Type parameter 'T extends unknown' not related to object
// 类型参数 'T extends unknown' 与 object 无关

function f23<T extends unknown>(x: T) {
    let y: object = x;  // Error
}

// Anything but primitive assignable to { [x: string]: unknown }
// 除了基本类型外的任何类型都可以赋值给 { [x: string]: unknown }

function f24(x: { [x: string]: unknown }) {
    x = {};
    x = { a: 5 };
    x = [1, 2, 3];
    x = 123;  // Error
}

// Locals of type unknown always considered initialized
// unknown 类型的局部变量总是被认为已经初始化了

function f25() {
    let x: unknown;
    let y = x;
}

// Spread of unknown causes result to be unknown
// 展开 unknow 会导致结果成为 unknown

function f26(x: {}, y: unknown, z: any) {
    let o1 = { a: 42, ...x };  // { a: number }
    let o2 = { a: 42, ...x, ...y };  // unknown
    let o3 = { a: 42, ...x, ...y, ...z };  // any
}

// Functions with unknown return type don't need return expressions
// 返回值为 unknown 类型的函数不需要返回表达式

function f27(): unknown {
}

// Rest type cannot be created from unknown
// 不能从 unknown 类型创建剩余类型

function f28(x: unknown) {
    let { ...a } = x;  // Error
}

// Class properties of type unknown don't need definite assignment
// unknown 类型的类属性不需要在定义时赋值

class C1 {
    a: string;  // Error
    b: unknown;
    c: any;
}
```

# Support for `defaultProps` in JSX
# 支持 JSX 中的 `defaultProps`

TypeScript 2.9 and earlier didn’t leverage [React `defaultProps`](https://reactjs.org/docs/typechecking-with-proptypes.html#default-prop-values) declarations inside JSX components.
Users would often have to declare properties optional and use non-null assertions inside of `render`, or they'd use type-assertions to fix up the type of the component before exporting it.

TypeScript 2.9 及更早版本不支持在 JSX 组件中 [React `defaultProps`](https://reactjs.org/docs/typechecking-with-proptypes.html#default-prop-values) 的声明。用户通常不得不将属性声明为可选的以及在 `render` 内部使用非空断言，或者在导出组件前他们会用类型断言来修复组件的类型。

TypeScript 3.0 adds supports a new type alias in the `JSX` namespace called `LibraryManagedAttributes`.
This helper type defines a transformation on the component's `Props` type, before using to check a JSX expression targeting it; thus allowing customization like: how conflicts between provided props and inferred props are handled, how inferences are mapped, how optionality is handled, and how inferences from differing places should be combined.

TypeScript 3.0 在 `JSX` 命名空间中添加了叫做 `LibraryManagedAttributes` 的新的类型别名。在用于检查以其为目标的 JSX 表达式之前，这个帮助器类型定义了一个在组件的 `Props` 类型上的转换；从而允许自定义诸如：如何处理提供的 props 和推断出的  props 之间的冲突，如何映射推断，如何处理可选性，以及如何组合从不同地方推断得到的结果。

In short using this general type, we can model React's specific behavior for things like `defaultProps` and, to some extent, `propTypes`.

简而言之，使用这种一般类型，我们能够模拟 React 的某些特定行为，如 `defaultProps` 以及一定程度上的 `propTypes`。

<sup id="ts-3-0-no-explicit-type-annotation-for-default-properties">[1]</sup> No explicit type annotation for default properties.

```tsx
export interface Props {
    name: string;
}

export class Greet extends React.Component<Props> {
    render() {
        const { name } = this.props;
        return <div>Hello ${name.toUpperCase()}!</div>;
    }
    static defaultProps = { name: "world"}; // No explicit type annotation for default properties.
}

// Type-checks! No type assertions needed!
let el = <Greet />
```

## Caveats
## 注意事项

### Explicit types on `defaultProps`
###  `defaultProps` 的显式类型

The default-ed properties are inferred from the `defaultProps` property type. If an explicit type annotation is added, e.g. `static defaultProps: Partial<Props>;` the compiler will not be able to identify which properties have defaults (since the type of `defaultProps` include all properties of `Props`).

默认的属性从 `defaultProps` 属性类型推断而来。如果添加显式的类型注解，如 `static defaultProps: Partial<Props>;` （添加了 `Partial<Props>` 这个类型注解）编译器将无法区分哪些属性有默认值（因为 `defaultProps` 类型包含了 `Props` 的所有属性）。

Use `static defaultProps: Pick<Props, "name">;` as an explicit type annotation instead, or do not add a type annotation as done in the example above.

使用 `static defaultProps: Pick<Props, "name">;` 作为显式类型注解来替代（上述 `Partial<Props>` 方式的类型注解），或者像[上述示例](#ts-3-0-no-explicit-type-annotation-for-default-properties)那样不要添加类型注解。

For stateless function components (SFCs) use ES2015 default initializers for SFCs:

```tsx
function Greet({ name = "world" }: Props) {
    return <div>Hello ${name.toUpperCase()}!</div>;
}
```

#### Changes to `@types/React`
####  `@types/React` 的变化

Corresponding changes to add `LibraryManagedAttributes` definition to the `JSX` namespace in `@types/React` are still needed.
Keep in mind that there are some limitations.

为了添加 `LibraryManagedAttributes` 的定义到 `@types/React` 的 `JSX` 命名空间中，相应的变化仍是需要的。

# `/// <reference lib="..." />` reference directives
# `/// <reference lib="..." />` 引用指令

TypeScript adds a new triple-slash-reference directive (`/// <reference lib="name" />`), allowing a file to explicitly include an existing built-in _lib_ file.

TypeScript 添加了一个新的三斜线指令（`/// <reference lib="name" />`），允许一个文件显式地包含一个现有的内置 _lib_ 文件。

Built-in _lib_ files are referenced in the same fashion as the `"lib"` compiler option in _tsconfig.json_ (e.g. use `lib="es2015"` and not `lib="lib.es2015.d.ts"`, etc.).

内置的 _lib_ 文件都被以与在 _tsconfig.json_ 文件中的 `"lib"` 编译器选项相同的方式引用（如，使用 `lib="es2015"` 而不是 `lib="lib.es2015.d.ts"`，等等）。

For declaration file authors who relay on built-in types, e.g. DOM APIs or built-in JS run-time constructors like `Symbol` or `Iterable`, triple-slash-reference lib directives are the recommended. Previously these .d.ts files had to add forward/duplicate declarations of such types.

对于那些依赖于内置类型的声明文件的作者，如 DOM APIs 或者内置的 JS 运行时构造器如 `Symbol` 或 `Iterable`，三斜线引用库指令是推荐的（引用内置类型）方式。早些时候这些 .d.ts 文件必须添加这些内置类型的前置/重复的声明。

##### Example
##### 示例

Using `/// <reference lib="es2017.string" />` to one of the files in a compilation is equivalent to compiling with `--lib es2017.string`.

在编译的某个文件中使用 `/// <reference lib="es2017.string" />` 与使用 `--lib es2017.string` 编译选项是等价的。

```ts
/// <reference lib="es2017.string" />

"foo".padStart(4);
```