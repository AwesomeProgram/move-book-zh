### 地址类型

虽然 `地址（address）` 在底层是一个 128 位整数，但 Move 语言有意让其不透明，它们不能从整数创建，不支持算术运算，也不能修改。即使可能有一些有趣的程序会使用这种特性（例如，C 语言中的指针算法实现了类似壁龛（niche）的功能），但 Move 语言不允许这种动态行为，因为它一开始就被设计为仅支持静态验证。*（壁龛指安装在墙壁上的小格子或在墙身上留出的作为贮藏设施的空间，最早在宗教上是指排放佛像的小空间，现在多用在家庭装修上，因其不占建筑面积，使用比较方便，通常在装修中有很多这样的选择）*

你可以通过使用运行时的地址值（`address` 类型的值）来访问该地址处的资源，但无法在运行时通过使用地址值访问模块。

### 地址及其语法

地址有两种形式：*命名的*或*数值的*。命名地址的语法遵循 Move 命名标识符的规则。数值地址的语法不受十六进制编码值的限制，任何有效的 [`u128` 数值](./integers.md)都可以用作地址值。例如，`42`，`0xCFAE` 和 `2021` 都是合法有效的数值地址字面量（literal）。

为了区分何时在表达式上下文中使用地址，使用地址时的语法根据使用地址的上下文而有所不同：

* 当地址被用作表达式时，地址必须以 `@` 字符为前缀，例如：[`@<numerical_value>`](./integers.md) 或 `@<named_address_identifier>`。
* 在表达式上下文之外，地址可以不带前缀字符 `@`。例如：[`<numerical_value>`](./integers.md) 或 `<named_address_identifier>`。

通常情况下，可以将 `@` 字符视为将地址从命名空间项变为表达式项的运算符。

### 命名地址

命名地址是一项特性，它允许在使用地址的任何地方使用标识符代替数值，而不仅仅是在值级别。命名地址被声明并绑定为 Move 包中，除了模块和脚本之外其它的顶级元素，或者也可以作为参数传递给 Move 编译器。

命名地址仅存在于源语言级别，并将在字节码级别完全替代它们的值。因此，模块和模块成员*必须*通过模块的命名地址而不是编译期间分配给命名地址的值来访问，例如：`use my_addr::foo` *不等于* `use 0x2::foo`，即使 Move 程序编译时将 `my_addr` 设置成 `0x2`。两者的区别我们在之前的[模块和脚本](./modules-and-scripts.md)一节中有详细的讨论过。

### 使用命名地址的一些示例

```move
let a1: address = @0x1; // 0x00000000000000000000000000000001 的缩写
let a2: address = @0x42; // 0x00000000000000000000000000000042 的缩写
let a3: address = @0xDEADBEEF; // 0x000000000000000000000000DEADBEEF 的缩写
let a4: address = @0x0000000000000000000000000000000A;
let a5: address = @std; // 将命名地址 `std` 的值赋给 `a5`
let a6: address = @66;
let a7: address = @0x42;

module 66::some_module {   // 不在表达式上下文中，所以不需要 @
    use 0x1::other_module; // 不在表达式上下文中，所以不需要 @
    use std::vector;       // 使用其他模块时，可以使用命名地址作为命名空间项
    ...
}

module std::other_module {  // 可以使用命名地址作为命名空间项来声明模块
    ...
}
```

### 全局存储操作

`address` 值主要用来与全局存储操作进行交互。

`address` 值与 `exists`、`borrow_global`、`borrow_global_mut` 和 `move_from` [操作](./global-storage-operators.md)一起使用。

唯一*不使用* `address` 的全局存储操作是 `move_to`，它使用的是 [`signer`](./signer.md)。

As with the other scalar values built-in to the language, `address` values are implicitly copyable, meaning they can be copied without an explicit instruction such as [`copy`](./variables.md#move-and-copy).

与 Move 语言内置的其他标量值一样，`address` 值是隐式可复制的，这意味着它们可以在没有明确指令如 [`copy`](./variables.md#move-and-copy) 的情况下进行复制，这一点复用了 Rust 的特性。
