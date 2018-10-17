# Performance Improvements
# 性能提升

The 1.1 compiler is typically around 4x faster than any previous release. See [this blog post for some impressive charts.](http://blogs.msdn.com/b/typescript/archive/2014/10/06/announcing-typescript-1-1-ctp.aspx)

1.1 版的编译器通常比先前发布的任何版本快 4 倍。详情见 [这个博文里的令人印象深刻的图表](http://blogs.msdn.com/b/typescript/archive/2014/10/06/announcing-typescript-1-1-ctp.aspx)。

# Better Module Visibility Rules
# 更好的模块可见性规则

TypeScript now only strictly enforces the visibility of types in modules if the `--declaration` flag is provided. This is very useful for Angular scenarios, for example:

当提供了 `--declaration` 标志时，TypeScript 现在仅严格实施模块中类型的可见性。这在 Angular 场景中非常有用，例如：

```ts
module MyControllers {
  interface ZooScope extends ng.IScope {
    animals: Animal[];
  }
  export class ZooController {
    // Used to be an error (cannot expose ZooScope), but now is only
    // an error when trying to generate .d.ts files
    // 过去是一个错误（不能暴露 ZooScope），但现在仅在尝试生成 .d.ts 文件时才报错
    constructor(public $scope: ZooScope) { }
    /* more code */
  }
}
```