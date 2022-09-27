## 你好，我是量子猿宇宙。本节为大家讲解**模块**与**脚本**。

Move 有两种不同类型的程序：***模块（Module）***和***脚本（Script）***。模块是定义结构类型以及对这些类型进行操作的函数的库。*结构类型*定义了 Move 的[全局存储](./global-storage-structure.md)的模式，*模块函数*定义了更新存储的规则。模块本身也存储在全局存储中。脚本是[可执行文件](https://en.wikipedia.org/wiki/Executable)的入口点，类似于传统语言中的主函数 `main`。脚本通常调用已发布模块的函数来更新全局存储。脚本是临时代码片段，不会发布在全局存储中。

一个 Move 源文件（或**编译单元**）可能包含多个模块和脚本。然而，发布一个模块或执行一个脚本是由虚拟机（VM）分开操作的。

## 语法

### 脚本

脚本的结构如下所示：

```text
script {
    <use>*
    <constants>*
    fun <identifier><[type parameters: constraint]*>([identifier: type]*) <function_body>
}
```

一个 `script` 代码块必须以它的所有 [`use`](./uses.md) 声明开头，然后是 [常量（constant）](./constants.md) 声明，最后是主[函数](./functions.md)声明。主函数的名称可以是任意的（也就是说，它不一定命名为 `main`），它是脚本块中唯一的函数，可以有任意数量的参数，并且不能有返回值。下面是每个组件的示例：

```move
script {
    // 导入在命名账户地址 std 上发布的 debug 模块。
    use std::debug;

    const ONE: u64 = 1;

    fun main(x: u64) {
        let sum = x + ONE;
        debug::print(&sum)
    }
}
```

脚本的功能非常有限 —— 它们不能声明友元（friend）、结构类型或访问全局存储。他们的主要作用是调用*模块函数*。

### 模块

模块的结构如下所示：

```text
module <address>::<identifier> {
    (<use> | <friend> | <type> | <function> | <constant>)*
}
```

其中 `<address>` 是一个有效的[命名或字面量地址](./address.md)。

例子如下所示：

```move
module 0x42::test {
    struct Example has copy, drop { i: u64 }

    use std::debug;
    friend 0x42::another_test;

    const ONE: u64 = 1;

    public fun print(x: u64) {
        let sum = x + ONE;
        let example = Example { i: sum };
        debug::print(&sum)
    }
}
```

`module 0x42::test` 这部分代码指定模块 `test` 将在[全局存储](./global-storage-structure.md)中的[账户地址](./address.md) `0x42` 下发布。

模块也可以使用[命名地址](./address.md)来声明，例子如下所示：

```move
module test_addr::test {
    struct Example has copy, drop { a: address }

    use std::debug;
    friend test_addr::another_test;

    public fun print() {
        let example = Example { a: @test_addr };
        debug::print(&example)
    }
}
```

因为命名地址只存在于源语言级别和编译期间，所以命名地址将在字节码级别彻底替换它们的值。例如，有以下代码所示：

```move
script {
    fun example() {
        my_addr::m::foo(@my_addr);
    }
}
```

我们再把 `my_addr` 设置为 `0xC0FFEE` 的情况下编译它，那么它在操作上等同于以下代码：

```move
script {
    fun example() {
        0xC0FFEE::m::foo(@0xC0FFEE);
    }
}
```

然而，在源代码级别，这些是*不等价的* —— 函数 `m::foo` *必须*通过 `my_addr` 命名地址来访问，而不是通过分配给该地址的数值来访问。

模块名称可以以小写字母 `a` 到 `z` 或大写字母 `A` 到 `Z` 开头。在第一个字符之后，模块名可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
module my_module {}
module foo_bar_42 {}
```

通常情况下，模块名称以小写字母开头。名为 `my_module` 的模块应该存放在名为 `my_module.move` 的源文件中。

`module` 代码块内的所有元素都可以按任意顺序出现。从根本上说，模块是[`类型（type）`](./structs-and-resources.md)和[`函数（function）`](./functions.md)的集合。[`use`](./uses.md) 关键字用来从其他模块导入类型。[`friend`](./friends.md) 关键字指定一个可信的模块列表。[`const`](./constants.md) 关键字定义了可以在模块函数中使用的私有常量。

#### 本节我们介绍了模块与脚本的语法。了解到了脚本是类似于其它语言的一个入口代码，用于调用模块函数；模块是类型和函数的集合，用于编写智能合约的主体逻辑。这对我们下一节《10分钟掌握 Move 智能合约开发》十分重要。那么，下一节我们就开始正式讲解如何在10分钟之内学会 Move 智能合约编程语言的开发。











