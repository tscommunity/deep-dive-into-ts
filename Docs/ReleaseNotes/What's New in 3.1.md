# TypeScript 3.1

# Mapped types on tuples and arrays
# 元组和数组上的映射类型

In TypeScript 3.1, mapped object types<sup>[[1]](#ts-3-1-only-homomorphic)</sup> over tuples and arrays now produce new tuples/arrays, rather than creating a new type where members like `push()`, `pop()`, and `length` are converted.
For example:

在 TypeScript 3.1 中，元组和数组上的映射对象类型现在会生成新的元组/数组而不是创建一个新的，把 `push()`、`pop()` 以及 `length` 成员都转换掉的类型。例如：

```ts
type MapToPromise<T> = { [K in keyof T]: Promise<T[K]> };

type Coordinate = [number, number]

type PromiseCoordinate = MapToPromise<Coordinate>; // [Promise<number>, Promise<number>]
```

`MapToPromise` takes a type `T`, and when that type is a tuple like `Coordinate`, only the numeric properties are converted.
In `[number, number]`, there are two numerically named properties: `0` and `1`.
When given a tuple like that, `MapToPromise` will create a new tuple where the `0` and `1` properties are `Promise`s of the original type.
So the resulting type `PromiseCoordinate` ends up with the type `[Promise<number>, Promise<number>]`.

`MapToPromise` 接受一个类型 `T`，并且当这个类型是一个像 `Coordinate` 一样的元组时，只转换该类型中数值类型的属性。在 `[number, number]` 中，有两个数值类型的命名属性：`0` 和 `1`。当给定一个这样的元组时，`MapToPromise` 将会创建一个新的元组，其中 `0` 和 `1` 这两个属性变成了原来类型的 `Promise` 版本。所以 `PromiseCoordinate` 的结果就是 `[Promise<number>, Promise<number>]` 这样的类型。


# Properties declarations on functions
# 函数的属性声明

TypeScript 3.1 brings the ability to define properties on function declarations and `const`-declared functons, simply by assigning to properties on these functions in the same scope.
This allows us to write canonical JavaScript code without resorting to `namespace` hacks.
For example:

TypeScript 3.1 带来了在函数声明以及用 `const` 声明的函数上定义属性的能力，通过简单地在相同的作用域内对这些函数的属性赋值即可。这允许我们编写典型的 JavaScript 代码而不用寻求 `namespace` 的方式。例如：

```ts
// now in 3.1
function readImage(path: string, callback: (err: any, image: Image) => void) {
    // ...
}

readImage.sync = (path: string) => {
    const contents = fs.readFileSync(path);
    return decodeImageSync(contents);
}

// before 3.1, namespace hacks
function readImage(path:string, callback:(err:any, image:Image) => void) {
  // ...
}

namespace readImage {
  let sync = (path:string) => {
    const contents = fs.readFileSync(path);
    return decodeImageSync(contents);
  }
}
```

Here, we have a function `readImage` which reads an image in a non-blocking asynchronous way.
In addition to `readImage`, we've provided a convenience function on `readImage` itself called `readImage.sync`.

这里 `readImage` 函数以非阻塞异步的方式读取图片。除了 `readImage` 外，我们还在 `readImage` 上提供了一个便利的函数 `readImage.sync`。 

While ECMAScript exports are often a better way of providing this functionality, this new support allows code written in this style to "just work" TypeScript.
Additionaly, this approach for property declarations allows us to express common patterns like `defaultProps` and `propTypes` on React stateless function components (SFCs).

虽然 ECMAScript 的导出是一个更佳的提供这种功能的方式，但是这种新的支持允许这种风格的代码在 TypeScript 中“有效”。此外，这种声明属性的途径允许我们表达常见的模式，如在 React 中无状态的函数组件的 `defaultProps` 和 `propTypes`。

```ts
export const FooComponent => ({ name }) => (
    <div>Hello! I am {name}</div>
);

FooComponent.defaultProps = {
    name: "(anonymous)",
};
```

<!--
fs.readFile(path, (err, data) => {
        if (err) callback(err, undefined);
        else decodeImage(data, (err, image) => {
            if (err) callback(err, undefined);
            else callback(undefined, image);
        });
    });
-->

------

<sup id="ts-3-1-only-homomorphic">[1]</sup> More specifically, homomorphic mapped types like in the above form.

更具体地，同态映射类型像上面的形式。

# Version selection with `typesVersions`
# 用 `typesVersions` 选择版本

Feedback from our community, as well as our own experience, has shown us that leveraging the newest TypeScript features while also accomodating users on the older versions are difficult.
TypeScript introduces a new feature called `typesVersions` to help accomodate these scenarios.

来自我们社区的反馈，一如我们自己的体会一样，已经让我们知道利用最新的 TypeScript 特性同时兼顾老版本的用户是非常困难的。TypeScript 引入了一个叫做 `typesVersions` 的新特性来帮助实现兼顾。

When using Node module resolution in TypeScript 3.1, when TypeScript cracks open a `package.json` file to figure out which files it needs to read, it first looks at a new field called `typesVersions`.
A `package.json` with a `typesVersions` field might look like this:

当在 TypeScript 3.1 中使用 Node 模块解决方案时，当 TypeScript 打开一个 `package.json` 文件以确定它需要读取哪些文件时，它将首先查看一个新的叫做 `typesVersions` 的字段。一个包含 `typesVersions` 字段的 `package.json` 可能看起来像这样：

```json
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}
```

This `package.json` tells TypeScript to check whether the current version of TypeScript is running.
If it's 3.1 or later, it figures out the path you've imported relative to the package, and reads from the package's `ts3.1` folder.
That's what that `{ "*": ["ts3.1/*"] }` means - if you're familiar with path mapping today, it works exactly like that.

这个 `package.json` 告诉 TypeScript 去检查当前版本的 TypeScript 是否正在运行中。如果是 3.1 或更新的版本，它将确定你已导入的相对于这个包的路径，并从这个包的 `ts3.1` 文件夹中读取。这就是 `{ "*": ["ts3.1/*"] }` 代表的含义——如果你已熟悉路径映射，这和它非常相似。

So in the above example, if we're importing from `"package-name"`, we'll try to resolve from `[...]/node_modules/package-name/ts3.1/index.d.ts` (and other relevant paths) when running in TypeScript 3.1.
If we import from `package-name/foo`, we'll try to look for `[...]/node_modules/package-name/ts3.1/foo.d.ts` and `[...]/node_modules/package-name/ts3.1/foo/index.d.ts`.

所以在上述示例中，如果从 `"package-name"` 导入，当运行在 TypeScript 3.1 中时，将尝试从 `[...]/node_modules/package-name/ts3.1/index.d.ts` （以及其它相关的路径）解析。如果从 `package-name/foo` 导入，将尝试查找 `[...]/node_modules/package-name/ts3.1/foo.d.ts` 和 `[...]/node_modules/package-name/ts3.1/foo/index.d.ts`。

What if we're not running in TypeScript 3.1 in this example?
Well, if none of the fields in `typesVersions` get matched, TypeScript falls back to the `types` field, so here TypeScript 3.0 and earlier will be redirected to `[...]/node_modules/package-name/index.d.ts`.

如果这个示例不是运行在 TypeScript 3.1 的版本将会怎样呢？如果没有匹配到任何 `typesVersions` 中的字段，TypeScript 将退回到 `types` 字段，所以在这里 TypeScript 3.0 和更早的版本将会重定向到 `[...]/node_modules/package-name/index.d.ts`。

## Matching behavior
## 匹配行为

The way that TypeScript decides on whether a version of the compiler & language matches is by using Node's [semver ranges](https://github.com/npm/node-semver#ranges).

TypeScript 决定编译器和语言的版本是否匹配的方式是使用 Node 的 [semver 范围](https://github.com/npm/node-semver#ranges)。

## Multiple fields
## 多字段

`typesVersions` can support multiple fields where each field name is specified by the range to match on.

`typesVersions` 支持多字段，每个字段名由要匹配的范围值指定。

```json
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    ">=3.2": { "*": ["ts3.2/*"] },
    ">=3.1": { "*": ["ts3.1/*"] }
  }
}
```

Since ranges have the potential to overlap, determining which redirect applies is order-specific.
That means in the above example, even though both the `>=3.2` and the `>=3.1` matchers support TypeScript 3.2 and above, reversing the order could have different behavior, so the above sample would not be equivalent to the following.

由于范围有可能会重叠，确定应用哪个重定向是和顺序关联的。那意味着在上述示例中，即使 `>=3.2` 和 `>=3.1` 匹配器都支持 TypeScript 3.2 和更高版本，交换顺序会有不同的行为，故上述示例与以下示例是不同的。

```json5
{
  "name": "package-name",
  "version": "1.0",
  "types": "./index.d.ts",
  "typesVersions": {
    // NOTE: this doesn't work!
    ">=3.1": { "*": ["ts3.1/*"] },
    ">=3.2": { "*": ["ts3.2/*"] }
  }
}
```