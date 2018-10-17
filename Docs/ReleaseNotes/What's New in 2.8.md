# Conditional Types
# 条件类型

TypeScript 2.8 introduces *conditional types* which add the ability to express non-uniform type mappings.
A conditional type selects one of two possible types based on a condition expressed as a type relationship test:

TypeScript 2.8 引用了*条件类型*，它带了表达非统一的类型映射的能力。条件类型基于条件表达式作为类型关系测试来选择两个可能类型中的一个：

```ts
T extends U ? X : Y
```

The type above means when `T` is assignable to `U` the type is `X`, otherwise the type is `Y`.

上述类型意思是当 `T` 可以赋值给 `U` 时类型是 `X` 否则类型是 `Y`。

A conditional type `T extends U ? X : Y` is either *resolved* to `X` or `Y`, or *deferred* because the condition depends on one or more type variables.
Whether to resolve or defer is determined as follows:

 条件类型 `T extends U ? X : Y` 将被确定为 `X` 或 `Y`，或者被*延迟* 因为条件依赖于一个或多个类型变量。是确定还是延迟取决于如下规则：

* First, given types `T'` and `U'` that are instantiations of `T` and `U` where all occurrences of type parameters are replaced with `any`, if `T'` is not assignable to `U'`, the conditional type is resolved to `Y`. Intuitively, if the most permissive instantiation of `T` is not assignable to the most permissive instantiation of `U`, we know that no instantiation will be and we can just resolve to `Y`.

  首先，给定类型 `T'` 和 `U'` 的实例，其中类型参数出现的位置均替换为 `any` ，如果 `T'` 不能赋值给 `U'`，则条件类型被确定为 `Y`。直观地，如果 `T` 的最宽容实例不能赋值给 `U` 的最宽容实例，我们知道没有实例能够赋值给 `U'`，我们就将条件类型确定为 `Y`。

* Next, for each type variable introduced by an `infer` (more later) declaration within `U` collect a set of candidate types by inferring from `T` to `U` (using the same inference algorithm as type inference for generic functions). For a given `infer` type variable `V`, if any candidates were inferred from co-variant positions, the type inferred for `V` is a union of those candidates. Otherwise, if any candidates were inferred from contra-variant positions, the type inferred for `V` is an intersection of those candidates. Otherwise, the type inferred for `V` is `never`.

  其次，对于由 `U` 中的推断（`infer`）（更晚些）声明引入的每个类型变量，通过从 `T` 到 `U` 推断来收集一组候选类型（使用与泛型函数类型推断一样的推断算法）。对于一个给定的推断（`infer`）类型变量 `V`，如果任何候选项都是从协变位置推断出来的，则为 `V` 推断出的类型是一个这些候选项的联合类型。否则，如果任何候选项都是从逆变位置推断出来的，则为 `V` 推断出的类型是一个这些候选项的交集。否则为 `V` 推断出的类型是 `never`。

* Then, given a type `T''` that is an instantiation of `T` where all `infer` type variables are replaced with the types inferred in the previous step, if `T''` is *definitely assignable* to `U`, the conditional type is resolved to `X`. The definitely assignable relation is the same as the regular assignable relation, except that type variable constraints are not considered. Intuitively, when a type is definitely assignable to another type, we know that it will be assignable for *all instantiations* of those types.

  然后，给定一个 `T` 的实例 `T''` 类型 ，其中所有的推断（`infer`）类型变量都用上一步推断出的类型替换，如果 `T''` *绝对能够赋值*给 `U`，条件类型将确定为 `X`。除开不用考虑类型变量约束，绝对能够赋值关系和常规的能够赋值关系是一样的。直观地，当一个类型绝对能够赋值给另一个类型，我们知道该类型将能够赋值给这些类型的*所有实例*。

* Otherwise, the condition depends on one or more type variables and the conditional type is deferred.

  否则，依赖于一个或多个类型变量的条件和这个条件类型是延迟（确定）的。

##### Example
##### 示例

```ts
type TypeName<T> =
    T extends string ? "string" :
    T extends number ? "number" :
    T extends boolean ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;  // "string"
type T2 = TypeName<true>;  // "boolean"
type T3 = TypeName<() => void>;  // "function"
type T4 = TypeName<string[]>;  // "object"
```

## Distributive conditional types
## 分布式条件类型

Conditional types in which the checked type is a naked type parameter are called *distributive conditional types*.
Distributive conditional types are automatically distributed over union types during instantiation.
For example, an instantiation of `T extends U ? X : Y` with the type argument `A | B | C` for `T` is resolved as `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`.

检查类型是裸类型参数的条件类型被称为分布式条件类型。在实例化期间分布式条件类型被自动分配为联合类型。例如，用 `A | B | C` 作为 `T` 的值实例化 `T extends U ? X : Y` 后被解析为 `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

##### Example
##### 示例

```ts
type T10 = TypeName<string | (() => void)>;  // "string" | "function"
type T12 = TypeName<string | string[] | undefined>;  // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]>;  // "object"
```

In instantiations of a distributive conditional type `T extends U ? X : Y`, references to `T` within the conditional type are resolved to individual constituents of the union type (i.e. `T` refers to the individual constituents *after* the conditional type is distributed over the union type).
Furthermore, references to `T` within `X` have an additional type parameter constraint `U` (i.e. `T` is considered assignable to `U` within `X`).

在一个分布式条件类型 `T extends U ? X : Y` 的实例中，在条件类型中对 `T` 的引用都被解析为联合类型的各个组成部分（即，`T` 指的是条件类型在联合类型上分配后的各个部分）。

##### Example
##### 示例

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>;  // BoxedValue<string>;
type T21 = Boxed<number[]>;  // BoxedArray<number>;
type T22 = Boxed<string | number[]>;  // BoxedValue<string> | BoxedArray<number>;
```

Notice that `T` has the additional constraint `any[]` within the true branch of `Boxed<T>` and it is therefore possible to refer to the element type of the array as `T[number]`. Also, notice how the conditional type is distributed over the union type in the last example.

注意，在 `Boxed<T>` 这个真值分支中 `T` 有额外的约束 `any[]`，因此可以将数组的元素类型称为 `T[number]`。同样要注意最后一个例子中的条件类型是如何在联合类型上分配的。

The distributive property of conditional types can conveniently be used to *filter* union types:

条件类型的分布式属性可以方便地用于过滤联合类型：

```ts
type Diff<T, U> = T extends U ? never : T;  // Remove types from T that are assignable to U
type Filter<T, U> = T extends U ? T : never;  // Remove types from T that are not assignable to U

type T30 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T31 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"
type T32 = Diff<string | number | (() => void), Function>;  // string | number
type T33 = Filter<string | number | (() => void), Function>;  // () => void

type NonNullable<T> = Diff<T, null | undefined>;  // Remove null and undefined from T

type T34 = NonNullable<string | number | undefined>;  // string | number
type T35 = NonNullable<string | string[] | null | undefined>;  // string | string[]

function f1<T>(x: T, y: NonNullable<T>) {
    x = y;  // Ok
    y = x;  // Error
}

function f2<T extends string | undefined>(x: T, y: NonNullable<T>) {
    x = y;  // Ok
    y = x;  // Error
    let s1: string = x;  // Error
    let s2: string = y;  // Ok
}
```

Conditional types are particularly useful when combined with mapped types:

当和映射类型组合时条件类型是格外有用的：

```ts
type FunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? K : never }[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
    id: number;
    name: string;
    subparts: Part[];
    updatePart(newName: string): void;
}

type T40 = FunctionPropertyNames<Part>;  // "updatePart"
type T41 = NonFunctionPropertyNames<Part>;  // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part>;  // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part>;  // { id: number, name: string, subparts: Part[] }
```

Similar to union and intersection types, conditional types are not permitted to reference themselves recursively.
For example the following is an error.

类似于联合类型和取交类型，条件类型不允许递归的引用自身。例如下面是错误的：

##### Example
##### 示例

```ts
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;  // Error
```

## Type inference in conditional types
## 条件类型中的类型推断

Within the `extends` clause of a conditional type, it is now possible to have `infer` declarations that introduce a type variable to be inferred.
Such inferred type variables may be referenced in the true branch of the conditional type.
It is possible to have multiple `infer` locations for the same type variable.

在一个条件类型的 `extends` 子句中，现在能够让引入一个类型变量的 `infer` 声明被推导。这种推导类型变量可能在条件类型的真值分支中被引用。

For example, the following extracts the return type of a function type:

例如，下面的例子抽取函数类型的返回类型：

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

Conditional types can be nested to form a sequence of pattern matches that are evaluated in order:

条件类型能够被嵌套来形成按顺序进行求值的模式匹配序列：

```ts
type Unpacked<T> =
    T extends (infer U)[] ? U :
    T extends (...args: any[]) => infer U ? U :
    T extends Promise<infer U> ? U :
    T;

type T0 = Unpacked<string>;  // string
type T1 = Unpacked<string[]>;  // string
type T2 = Unpacked<() => string>;  // string
type T3 = Unpacked<Promise<string>>;  // string
type T4 = Unpacked<Promise<string>[]>;  // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>;  // string
```

The following example demonstrates how multiple candidates for the same type variable in co-variant positions causes a union type to be inferred:

下面的示例演示了在协变位置的相同类型变量的多个候选项是如何导致推导出联合类型：

```ts
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```

Likewise, multiple candidates for the same type variable in contra-variant positions causes an intersection type to be inferred:

同样的，在逆变位置的相同的类型变量的多个候选项导致推导出交叉类型。

```ts
type Bar<T> = T extends { a: (x: infer U) => void, b: (x: infer U) => void } ? U : never;
type T20 = Bar<{ a: (x: string) => void, b: (x: string) => void }>;  // string
type T21 = Bar<{ a: (x: string) => void, b: (x: number) => void }>;  // string & number
```

When inferring from a type with multiple call signatures (such as the type of an overloaded function), inferences are made from the *last* signature (which, presumably, is the most permissive catch-all case).
It is not possible to perform overload resolution based on a list of argument types.

当从一个带多调用签名的类型（比如重载方法这种类型）进行推导时，推导结果由*最后*的签名确定（这可能是最宽容的包含所有的情形）。无法基于参数类型列表进行重载决策。

```ts
declare function foo(x: string): number;
declare function foo(x: number): string;
declare function foo(x: string | number): string | number;
type T30 = ReturnType<typeof foo>;  // string | number
```

It is not possible to use `infer` declarations in constraint clauses for regular type parameters:

不能将约束子句中的推导（`infer`）声明用作常规类型参数：

```ts
// Error, not supported
// 错误，不受支持
type ReturnType<T extends (...args: any[]) => infer R> = R;  
```

However, much the same effect can be obtained by erasing the type variables in the constraint and instead specifying a conditional type:

然而，通过擦除约束中的类型变量并改为指定条件类型，可以获得相同的效果：

```ts
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R ? R : any;
```

## Predefined conditional types
## 预定义的条件类型

TypeScript 2.8 adds several predefined conditional types to `lib.d.ts`:

TypeScript 2.8 添加了多个预定义的条件类型到 `lib.d.ts` 中：

* `Exclude<T, U>` -- Exclude from `T` those types that are assignable to `U`.

  `Exclude<T, U>` —— 从 `T` 中排除能够赋值给 `U` 的类型。

* `Extract<T, U>` -- Extract from `T` those types that are assignable to `U`.

  `Extract<T, U>` —— 从 `T` 中抽取能够赋值给 `U` 的类型。

* `NonNullable<T>` -- Exclude `null` and `undefined` from `T`.

  `NonNullable<T>` —— 从 `T` 排除 `null` 和 `undefined` 。

* `ReturnType<T>` -- Obtain the return type of a function type.

  `ReturnType<T>` —— 获取函数的返回类型。

* `InstanceType<T>` -- Obtain the instance type of a constructor function type.

  `InstanceType<T>` —— 获取构造函数的实例类型。

##### Example
##### 示例

```ts
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function>;  // string | number
type T03 = Extract<string | number | (() => void), Function>;  // () => void

type T04 = NonNullable<string | number | undefined>;  // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined>;  // (() => string) | string[]

function f1(s: string) {
    return { a: 1, b: s };
}

class C {
    x = 0;
    y = 0;
}

type T10 = ReturnType<() => string>;  // string
type T11 = ReturnType<(s: string) => void>;  // void
type T12 = ReturnType<(<T>() => T)>;  // {}
type T13 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T14 = ReturnType<typeof f1>;  // { a: number, b: string }
type T15 = ReturnType<any>;  // any
type T16 = ReturnType<never>;  // any
type T17 = ReturnType<string>;  // Error
type T18 = ReturnType<Function>;  // Error

type T20 = InstanceType<typeof C>;  // C
type T21 = InstanceType<any>;  // any
type T22 = InstanceType<never>;  // any
type T23 = InstanceType<string>;  // Error
type T24 = InstanceType<Function>;  // Error
```

> Note: The `Exclude` type is a proper implementation of the `Diff` type suggested [here](https://github.com/Microsoft/TypeScript/issues/12215#issuecomment-307871458). We've used the name `Exclude` to avoid breaking existing code that defines a `Diff`, plus we feel that name better conveys the semantics of the type. We did not include the `Omit<T, K>` type because it is trivially written as `Pick<T, Exclude<keyof T, K>>`.



# Improved control over mapped type modifiers
# 改进了对映射类型修饰符的控制

Mapped types support adding a `readonly` or `?` modifier to a mapped property, but they did not provide support the ability to *remove* modifiers.
This matters in [*homomorphic mapped types*](https://github.com/Microsoft/TypeScript/pull/12563) which by default preserve the modifiers of the underlying type.

映射类型支持添加 `readonly` 或者 `?` 修饰符到映射属性，但不提供支持*移除*修饰符的能力。这在同态映射类型（[*homomorphic mapped types*](https://github.com/Microsoft/TypeScript/pull/12563)） 中很重要，默认情况下保留基础类型的修饰符。

TypeScript 2.8 adds the ability for a mapped type to either add or remove a particular modifier.
Specifically, a `readonly` or `?` property modifier in a mapped type can now be prefixed with either `+` or `-` to indicate that the modifier should be added or removed.

TypeScript 2.8 为映射类型增加了添加或删除修饰符的能力。特别地，映射类型中的一个 `readonly` 或 `?` 属性修饰符现在可以附加 `+` 或 `-` 前缀来指示该修饰符应该被添加或删除。

#### Example
#### 示例

```ts
// Remove readonly and ?
// 移除 readonly 和 ?
type MutableRequired<T> = { -readonly [P in keyof T]-?: T[P] }; 

// Add readonly and ?
// 添加 readonly 和 ?
type ReadonlyPartial<T> = { +readonly [P in keyof T]+?: T[P] };  
```

A modifier with no `+` or `-` prefix is the same as a modifier with a `+` prefix. So, the `ReadonlyPartial<T>` type above corresponds to

一个不带 `+` 或 `-` 前缀的修饰符与带 `+` 前缀的修饰符相同。所以，上述的 `ReadonlyPartial<T>` 对应于：

```ts
// Add readonly and ?
// 添加 readonly 和 ?
type ReadonlyPartial<T> = { readonly [P in keyof T]?: T[P] };  
```

Using this ability, `lib.d.ts` now has a new  `Required<T>` type.
This type strips `?` modifiers from all properties of `T`, thus making all properties required.

使用这种能力， `lib.d.ts` 现在有一个新的 `Required<T>` 类型。这个类型为 `T` 中所有的属性去掉了 `?` 修饰符，因此使所有的属性都是必需的。

##### Example
##### 示例

```ts
type Required<T> = { [P in keyof T]-?: T[P] };
```

Note that in `--strictNullChecks` mode, when a homomorphic mapped type removes a `?` modifier from a property in the underlying type it also removes `undefined` from the type of that property:

注意在 `--strictNullChecks` 模式下，当一个同态映射类型从基础类型中的属性移除掉了 `?` 修饰符，它也从那个属性的类型中移除了 `undefined` ：

##### Example
##### 示例

```ts
// Same as { a?: string | undefined }
// 等同于 { a?: string | undefined }
type Foo = { a?: string };  

// Same as { a: string }
// 等同于 { a: string }
type Bar = Required<Foo>;  
```

# Improved `keyof` with intersection types
# 用交叉类型改进了 `keyof` 

With TypeScript 2.8 `keyof` applied to an intersection type is transformed to a union of `keyof` applied to each intersection constituent.
In other words, types of the form `keyof (A & B)` are transformed to be `keyof A | keyof B`.
This change should address inconsistencies with inference from `keyof` expressions.

TypeScript 2.8 中应用于交叉类型的 `keyof` 被转换为应用于每一个交叉类型组分的 `keyof` 的联合类型。换言之， `keyof (A & B)` 形式的类型被转换成`keyof A | keyof B`。这个改变应该解决与从 `keyof` 表达式进行推导不一致的问题。

##### Example
##### 示例

```ts
type A = { a: string };
type B = { b: string };

type T1 = keyof (A & B);  // "a" | "b"
type T2<T> = keyof (T & B);  // keyof T | "b"
type T3<U> = keyof (A & U);  // "a" | keyof U
type T4<T, U> = keyof (T & U);  // keyof T | keyof U
type T5 = T2<A>;  // "a" | "b"
type T6 = T3<B>;  // "a" | "b"
type T7 = T4<A, B>;  // "a" | "b"
```

# Better handling for namespace patterns in `.js` files
# 更好的处理 `.js` 文件中的命名空间模式

TypeScript 2.8 adds support for understanding more namespace patterns in `.js` files.
Empty object literals declarations on top level, just like functions and classes, are now recognized as as namespace declarations in JavaScript.

TypeScript 2.8 添加了对 `.js` 文件中更多命名空间模式的理解的支持。在 JavaScript 中的顶级空对象字面量声明，与函数和类一样，现在能够被识别为命名空间声明。

```js
// recognized as a declaration for a namespace `ns`
// 被识别为一个命名空间 `ns` 的声明
var ns = {};     

// recognized as a declaration for var `constant`
// 被识别为 var `constant` 的声明
ns.constant = 1; 
```

Assignments at the top-level should behave the same way; in other words, a `var` or `const` declaration is not required.

在顶级的赋值应该有同样的行为；换言之，`var` 或 `const` 声明不是必需的。

```js
// does NOT need to be `var app = {}` 
// 不必是 `var app = {}` 的形式
app = {}; 
app.C = class {
};
app.f = function() {
};
app.prop = 1;
```

## IIFEs as namespace declarations
## IIFEs （Immediately Invoked Function Expression 立即调用函数表达式）作为命名空间声明

An IIFE returning a function, class or empty object literal, is also recognized as a namespace:

一个返回函数，类或者空对象字面量的 IIFE 也被识别为命名空间：

```js
var C = (function () {
  function C(n) {
    this.p = n;
  }
  return C;
})();
C.staticProperty = 1;
```

## Defaulted declarations
## 缺省声明

"Defaulted declarations" allow initializers that reference the declared name in the left side of a logical or:

“缺省声明”允许初始值设定项引用逻辑或左侧的声明名称：

```js
my = window.my || {};
my.app = my.app || {};
```

## Prototype assignment
## 原型赋值

You can assign an object literal directly to the prototype property. Individual prototype assignments still work too:

你可以将一个对象字面量赋值给原型属性。单独的原型赋值仍然有效：

```ts
var C = function (p) {
  this.p = p;
};

// assign an object literal directly to the prototype property
// 将一个对象字面量赋值给原型属性
C.prototype = {
  m() {
    console.log(this.p);
  }
};

// Individual prototype assignments
// 单独的原型赋值
C.prototype.q = function(r) {
  return this.p === r;
};
```

## Nested and merged declarations
## 嵌套和合并声明

Nesting works to any level now, and merges correctly across files. Previously neither was the case.

嵌套对任何层级有效且能够正确地跨文件合并。早先版本中这两者都不是这样。

```js
var app = window.app || {};
app.C = class { };
```

# Per-file JSX factories
# 文件级 JSX 工厂

TypeScript 2.8 adds support for a per-file configurable JSX factory name using `@jsx dom` pragma.
JSX factory can be configured for a compilation using `--jsxFactory` (default is `React.createElement`). With TypeScript 2.8 you can override this on a per-file-basis by adding a comment to the beginning of the file.

TypeScript 2.8 通过使用 `@jsx dom` 编译杂注添加了文件级可配置的 JSX 工厂名称。使用 `--jsxFactory` （默认是 `React.createElement`）编译选项能够为编译配置 JSX 工厂。在 TypeScript 2.8 中你可以通过在文件的开始处添加一个注释来为文件覆盖该默认行为。

##### Example
##### 示例

```ts
/** @jsx dom */
import { dom } from "./renderer"
<h></h>
```

Generates:

生成：

```js
var renderer_1 = require("./renderer");
renderer_1.dom("h", null);
```

# Locally scoped JSX namespaces
# 局部作用域内的 JSX 命名空间

JSX type checking is driven by definitions in a JSX namespace, for instance `JSX.Element` for the type of a JSX element, and `JSX.IntrinsicElements` for built-in elements.
Before TypeScript 2.8 the `JSX` namespace was expected to be in the global namespace, and thus only allowing one to be defined in a project.
Starting with TypeScript 2.8 the `JSX` namespace will be looked under the `jsxNamespace` (e.g. `React`) allowing for multiple jsx factories in one compilation.
For backward compatibility the global `JSX` namespace is used as a fallback if none was defined on the factory function.
Combined with the per-file `@jsx` pragma, each file can have a different JSX factory.

JSX 类型检查由 JSX 命名空间中的定义来驱动，例如一个 JSX 元素的类型 `JSX.Element` 和内置的元素 `JSX.IntrinsicElements` 。在 TypeScript 2.8 之前，`JSX` 命名空间被认为是在全局命名空间中，因此一个项目中只允许定义一个（JSX 命名空间）。自 TypeScript 2.8 开始，`JSX` 命名空间将被看作是在 `jsxNamespace` （例如 `React`）之下，这允许在一个编译中有多个 jsx 工厂。为了向后兼容，如果没有在工厂函数上定义任何全局的 `JSX` 命名空间，则该全局 `JSX` 命名空间的作用将被回退。

# New `--emitDeclarationOnly`
# 新的 `--emitDeclarationOnly` 编译选项

`--emitDeclarationOnly` allows for *only* generating declaration files; `.js`/`.jsx` output generation will be skipped with this flag. The flag is useful when the `.js` output generation is handled by a different transpiler like Babel.

`--emitDeclarationOnly` 编译选项允许*只*生成声明文件；启用这个标志 `.js`/`.jsx` 文件的生成输出将会被跳过。当 `.js` 文件的生成输出被不同的转译器（如 Babel）处理时这个标志是很有用的。