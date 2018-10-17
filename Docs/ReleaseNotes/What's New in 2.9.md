# Support `number` and `symbol` named properties with `keyof` and mapped types
# 为 `keyof` 和映射类型支持 `number` 和 `symbol` 命名属性 

TypeScript 2.9 adds support for `number` and `symbol` named properties in index types and mapped types.
Previously, the `keyof` operator and mapped types only supported `string` named properties.

TypeScript 2.9 为索引类型和映射类型添加了 `number` 和 `symbol` 命名属性支持。早先的版本 `keyof` 运算符和映射类型只支持 `string` 命名属性。

Changes include:

改变包括：

* An index type `keyof T` for some type `T` is a subtype of `string | number | symbol`.

  某个类型  `T` 的索引类型 `keyof T` 是 `string | number | symbol` 的子类型。

* A mapped type `{ [P in K]: XXX }` permits any `K` assignable to `string | number | symbol`.

  映射类型 `{ [P in K]: XXX }` 允许任何 `K` 赋值给 `string | number | symbol`。

* In a `for...in` statement for an object of a generic type `T`, the inferred type of the iteration variable was previously `keyof T` but is now `Extract<keyof T, string>`. (In other words, the subset of `keyof T` that includes only string-like values.)

  在一个泛型对象 `T` 的 `for...in` 语句中，推导出的迭代变量的类型在早先版本中是 `keyof T` 而现在是`Extract<keyof T, string>`（换言之， `keyof T` 的子集仅包括类字符串的值）。

Given an object type `X`, `keyof X` is resolved as follows:

给定一个对象 `X`， `keyof X` 按以下方式解析：

* If `X` contains a string index signature, `keyof X` is a union of `string`, `number`, and the literal types representing symbol-like properties, otherwise

  如果 `X` 包含一个字符串索引签名， `keyof X` 是一个 `string`，`number` 和字面量类型（代表类似 symbol 的属性）的联合类型，否则

* If `X` contains a numeric index signature, `keyof X` is a union of `number` and the literal types representing string-like and symbol-like properties, otherwise

  如果 `X` 包含一个数值索引签名，`keyof X` 是一个 `number` 和字面量类型（代表类似字符串和类似 symbol 的属性）的联合类型，否则

* `keyof X` is a union of the literal types representing string-like, number-like, and symbol-like properties.

  `keyof X` 是一个字面量类型（代表类似字符串，类似数值和类似 symbol 的属性）的联合类型。

Where:

其中：

* String-like properties of an object type are those declared using an identifier, a string literal, or a computed property name of a string literal type.

  对象的类似字符串属性是指那些使用标识符，字符串字面量，或者一个字符串字面量类型的计算属性名来进行声明的属性。

* Number-like properties of an object type are those declared using a numeric literal or computed property name of a numeric literal type.

  对象的类似数值属性是指那些使用数值字面量或数值字面量的计算属性名来声明的属性。

* Symbol-like properties of an object type are those declared using a computed property name of a unique symbol type.

  对象的类似 symbol 属性是指那些使用唯一 symbol 类型的计算属性名声明的属性。

In a mapped type `{ [P in K]: XXX }`, each string literal type in `K` introduces a property with a string name, each numeric literal type in `K` introduces a property with a numeric name, and each unique symbol type in `K` introduces a property with a unique symbol name.
Furthermore, if `K` includes type `string`, a string index signature is introduced, and if `K` includes type `number`, a numeric index signature is introduced.

在一个映射类型 `{ [P in K]: XXX }` 中， `K` 中的每个字符串字面量类型都引入了一个带字符串名的属性，`K` 中的每个数值型字面量类型都引入了一个带数值型名称的属性，`K` 中的每个唯一的 symbol 型都引入了一个带唯一 symbol 名称的属性。

##### Example
##### 示例

```ts
const c = "c";
const d = 10;
const e = Symbol();

const enum E1 { A, B, C }
const enum E2 { A = "A", B = "B", C = "C" }

type Foo = {
    a: string;       // String-like name
    5: string;       // Number-like name
    [c]: string;     // String-like name
    [d]: string;     // Number-like name
    [e]: string;     // Symbol-like name
    [E1.A]: string;  // Number-like name
    [E2.A]: string;  // String-like name
}

type K1 = keyof Foo;  // "a" | 5 | "c" | 10 | typeof e | E1.A | E2.A
type K2 = Extract<keyof Foo, string>;  // "a" | "c" | E2.A
type K3 = Extract<keyof Foo, number>;  // 5 | 10 | E1.A
type K4 = Extract<keyof Foo, symbol>;  // typeof e
```

Since `keyof` now reflects the presence of a numeric index signature by including type `number` in the key type, mapped types such as `Partial<T>` and `Readonly<T>` work correctly when applied to object types with numeric index signatures:

由于 `keyof` 现在通过在键类型中包含 `number` 类型反应出数值型索引签名，当映射类型如 `Partial<T>` 和 `Readonly<T>` 应用到带数值型索引签名的对象类型时能够正确的工作了：

```ts
type Arrayish<T> = {
    length: number;
    [x: number]: T;
}

type ReadonlyArrayish<T> = Readonly<Arrayish<T>>;

declare const map: ReadonlyArrayish<string>;
let n = map.length;
let x = map[123];  // Previously of type any (or an error with --noImplicitAny) but now get correct type.
```

Furthermore, with the `keyof` operator's support for `number` and `symbol` named keys, it is now possible to abstract over access to properties of objects that are indexed by numeric literals (such as numeric enum types) and unique symbols.

此外，有了 `keyof` 运算符支持 `number` 和 `symbol` 的命名键，现在通过访问由数值字面量进行索引（如数值型枚举类型）或唯一 symbol 进行索引的对象的属性来进行抽象成为了可能。

```ts
const enum Enum { A, B, C }

const enumToStringMap = {
    [Enum.A]: "Name A",
    [Enum.B]: "Name B",
    [Enum.C]: "Name C"
}

const sym1 = Symbol();
const sym2 = Symbol();
const sym3 = Symbol();

const symbolToNumberMap = {
    [sym1]: 1,
    [sym2]: 2,
    [sym3]: 3
};

type KE = keyof typeof enumToStringMap;     // Enum (i.e. Enum.A | Enum.B | Enum.C)
type KS = keyof typeof symbolToNumberMap;   // typeof sym1 | typeof sym2 | typeof sym3

function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}

let x1 = getValue(enumToStringMap, Enum.C);  // Returns "Name C"
let x2 = getValue(symbolToNumberMap, sym3);  // Returns 3
```

This is a breaking change; previously, the `keyof` operator and mapped types only supported `string` named properties.
Code that assumed values typed with `keyof T` were always `string`s, will now be flagged as error.

这是一个破坏性的变更；早先版本中， `keyof` 运算符及映射类型只支持 `string` 命名属性。假定 `keyof T` 类型的值总是 `string` 的代码现在将被标记为错误。

##### Example
##### 示例

```ts
function useKey<T, K extends keyof T>(o: T, k: K) {
    var name: string = k;  // Error: keyof T is not assignable to string
}
```

#### Recommendations
#### 推荐做法

* If your functions are only able to handle string named property keys, use `Extract<keyof T, string>` in the declaration:

  如果你的函数只能够处理字符串命名属性的键，则在声明中使用 `Extract<keyof T, string>` ：

  ```ts
  function useKey<T, K extends Extract<keyof T, string>>(o: T, k: K) {
    var name: string = k;  // OK
  }
  ```

* If your functions are open to handling all property keys, then the changes should be done down-stream:

  如果你的函数是开放处理所有的属性键，那么应该改成为如下形式：

  ```ts
  function useKey<T, K extends keyof T>(o: T, k: K) {
    var name: string | number | symbol = k;
  }
  ```

* Otherwise use `--keyofStringsOnly` compiler option to disable the new behavior.

  否则使用 `--keyofStringsOnly` 编译器选项来禁用该新行为。

# Generic type arguments in JSX elements
# JSX 元素中的泛型参数

JSX elements now allow passing type arguments to generic components.

JSX 元素现在允许传递类型参数给泛型组件。

##### Example
##### 示例

```ts
class GenericComponent<P> extends React.Component<P> {
    internalProp: P;
}

type Props = { a: number; b: string; };

const x = <GenericComponent<Props> a={10} b="hi"/>; // OK

const y = <GenericComponent<Props> a={10} b={20} />; // Error
```

# Generic type arguments in generic tagged templates
# 泛型标签模板中的泛型参数

Tagged templates are a form of invocation introduced in ECMAScript 2015.
Like call expressions, generic functions may be used in a tagged template and TypeScript will infer the type arguments utilized.

标签模板是在 ECMAScript 2015 中引入的一种调用形式。像调用表达式一样，泛型函数可能会被用到一个标签模板中，TypeScript 将推断使用的类型参数。

TypeScript 2.9  allows passing generic type arguments to tagged template strings.

TypeScript 2.9 允许传递泛型参数到标签模板字符串中。

##### Example
##### 示例

```ts
declare function styledComponent<Props>(strs: TemplateStringsArray): Component<Props>;

interface MyProps {
  name: string;
  age: number;
}

styledComponent<MyProps> `
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

declare function tag<T>(strs: TemplateStringsArray, ...args: T[]): T;

// inference fails because 'number' and 'string' are both candidates that conflict
// 推断失败因为 'number' 和 'string' 都是候选类型，造成冲突。

let a = tag<string | number> `${100} ${"hello"}`;
```

# `import` types
# `import` 类型

Modules can import types declared in other modules. But non-module global scripts cannot access types declared in modules. Enter `import` types.

模块可以导入在其它模块中声明的类型。但非模块的全局脚本不能访问定义在模块中的类型。进入 `import` 类型。

Using `import("mod")` in a type annotation allows for reaching in a module and accessing its exported declaration without importing it.

在类型注解中使用 `import("mod")` 允许直达一个模块并访问它导出的声明而不用导入它。

##### Example
##### 示例

Given a declaration of a class `Pet` in a module file:

在一个模块文件中给定一个 `Pet` 类的声明：

```ts
// module.d.ts

export declare class Pet {
   name: string;
}
```

Can be used in a non-module file `global-script.ts`:

可以被使用在一个非模块文件 `global-script.ts` 中：

```ts
// global-script.ts

function adopt(p: import("./module").Pet) {
    console.log(`Adopting ${p.name}...`);
}
```

This also works in JSDoc comments to refer to types from other modules in `.js`:

这个（import 特性）同样适用于在 JSDoc 注释中引用 `.js` 中的来自其它模块的类型：

```js
// a.js

/**
 * @param p { import("./module").Pet }
 */
function walk(p) {
    console.log(`Walking ${p.name}...`);
}
```

# Relaxing declaration emit visiblity rules
# 放松声明发出可见性规则

With `import` types available, many of the visibility errors reported during declaration file generation can be handled by the compiler without the need to change the input.

有了 `import` 类型，许多在生成声明文件期间报告的可见性错误能够被编译器处理而不用改变输入。

For instance:

例如

```ts
import { createHash } from "crypto";

export const hash = createHash("sha256");
//           ^^^^
// Exported variable 'hash' has or is using name 'Hash' from external module "crypto" but cannot be named.
// 导出的变量 'hash' 已经或正在使用来自外部模块 "crypto" 中的名称 'Hash' 但不能命名。
```

With TypeScript 2.9, no errors are reported, and now the generated file looks like:

TypeScript 2.9 后不再报告错误，并且现在的生成的文件看起来是这样的：

```ts
export declare const hash: import("crypto").Hash;
```

# Support for `import.meta`
# `import.meta` 的支持

TypeScript 2.9 introduces support for `import.meta`, a new meta-property as described by the current [TC39 proposal](https://github.com/tc39/proposal-import-meta).

TypeScript 2.9 引入了 `import.meta` 的支持，一个新的如当前 [TC39 proposal](https://github.com/tc39/proposal-import-meta) 中描述的元属性。

The type of `import.meta` is the global `ImportMeta` type which is defined in `lib.es5.d.ts`.
This interface is extremely limited.
Adding well-known properties for Node or browsers requires interface merging and possibly a global augmentation depending on the context.

 `import.meta` 的类型是定义在 `lib.es5.d.ts` 中的全局 `ImportMeta` 类型。这个接口相当的局限。为 Node 或者浏览器添加一个熟知的属性需要接口合并以及可能的依赖于上下文的全局参数。

##### Example
##### 示例

Assuming that `__dirname` is always available on `import.meta`, the declaration would be done through reopening `ImportMeta` interface:

假定 `__dirname` 在 `import.meta` 总是可用的，声明能够通过重新打开 `ImportMeta` 接口完成：

```ts
// node.d.ts
interface ImportMeta {
    __dirname: string;
}
```

And usage would be:

使用方式将会是：

```ts
import.meta.__dirname // Has type 'string'
```

`import.meta` is only allowed when targeting `ESNext` modules and ECMAScript targets.

`import.meta` 仅当目标为 `ESNext` 模块和 ECMAScript 时允许使用。

# New `--resolveJsonModule`
# 新的 `--resolveJsonModule`

Often in Node.js applications a `.json` is needed. With TypeScript 2.9, `--resolveJsonModule` allows for importing, extracting types from and generating `.json` files.

Node.js 应用程序中经常需要一个 `.json`。TypeScript 2.9 中的 `--resolveJsonModule` 允许导入，提取类型自 `.json` 文件，以及生成 `.json` 文件。

##### Example
##### 示例

```ts
// settings.json

{
    "repo": "TypeScript",
    "dry": false,
    "debug": false
}
```

```ts
// a.ts

import settings from "./settings.json";

settings.debug === true;  // OK
settings.dry === 2;  // Error: Operator '===' cannot be applied boolean and number

```

```ts
// tsconfig.json

{
    "compilerOptions": {
        "module": "commonjs",
        "resolveJsonModule": true,
        "esModuleInterop": true
    }
}
```

# `--pretty` output by default
# 默认开启 `--pretty` 输出

Starting TypeScript 2.9 errors are displayed under `--pretty` by default if the output device is applicable for colorful text.
TypeScript will check if the output steam has [`isTty`](https://nodejs.org/api/tty.html) property set.

自 TypeScript 2.9 起，如果输出设备适用于彩色文本，则错误都在 `--pretty` 默认开启的情况下显示。TypeScript 将会检查输出是否设置了 [`isTty`](https://nodejs.org/api/tty.html)  属性。

Use `--pretty false` on the command line or set `"pretty": false` in your `tsconfig.json` to disable `--pretty` output.

在命令行中使用 `--pretty false` 或者在 `tsconfig.json` 中设置 `"pretty": false` 来关闭 `--pretty` 输出。

# New `--declarationMap`
# 新的 `--declarationMap`

Enabling `--declarationMap` alongside `--declaration` causes the compiler to emit `.d.ts.map` files alongside the output `.d.ts` files.
Language Services can also now understand these map files, and uses them to map declaration-file based definition locations to their original source, when available.

同时启用 `--declarationMap` 和 `--declaration` 会导致编译器生成 `.d.ts.map` 和 `.d.ts` 文件。语言服务现在也能够理解这些映射文件了，并且使用它们去映射声明文件——基于定义位置至它们的原始源（当可用时）。

In other words, hitting go-to-definition on a declaration from a `.d.ts` file generated with `--declarationMap` will take you to the source file (`.ts`) location where that declaration was defined, and not to the `.d.ts`.

换言之，在来自开启 `--declarationMap` 选项生成的 `.d.ts` 文件中的声明上点击跳转到定义将会把你带到源文件（`.ts`）中定义了那个声明的位置，而不是到 `.d.ts` 文件中。