# Protected
# 受保护的

The new `protected` modifier in classes works like it does in familiar languages like C++, C#, and Java. A `protected` member of a class is visible only inside subclasses of the class in which it is declared:

类中新的 `protected` 修饰符像在其它我们熟悉的语言如 C++，C# 及 Java 中那样工作。类中 `protected` 修饰的成员仅在该类子类的声明中可见：

```ts
class Thing {
  protected doSomething() { /* ... */ }
}

class MyThing extends Thing {
  public myMethod() {
    // OK, can access protected member from subclass
    // 可以从子类访问受保护的成员
    this.doSomething();
  }
}
var t = new MyThing();
// Error, cannot call protected member from outside class
// 错误，不能从类外部调用受保护成员
t.doSomething(); 
```

# Tuple types
# 元组类型

Tuple types express an array where the type of certain elements is known, but need not be the same. For example, you may want to represent an array with a `string` at position 0 and a `number` at position 1:

元组表示一种元素类型已知但不需要元素类型相同的数组。例如，你可能想要表示一个这样的数组——它的第 0 个位置的元素是 `string` 类型，第 1 个位置的元素类型是 `number` ：

```ts
// Declare a tuple type
// 声明一个元组类型
var x: [string, number];

// Initialize it
// 初始化它
x = ['hello', 10]; // OK

// Initialize it incorrectly
// 不正确的初始化它
x = [10, 'hello']; // Error
```

When accessing an element with a known index, the correct type is retrieved:

当使用一个已知的下标去访问（元组中的）一个元素，能确定该元素的正确类型：

```ts
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

Note that in TypeScript 1.4, when accessing an element outside the set of known indices, a union type is used instead:

注意，在 TypeScript 1.4 中，当访问一个不在已知索引值范围内的元素时，使用元组中已确定的各个元素的类型的联合类型作为该元素的类型的替代：

```ts
x[3] = 'world'; // OK
console.log(x[5].toString()); // OK, 'string' and 'number' both have toString
x[6] = true; // Error, boolean isn't number or string
```