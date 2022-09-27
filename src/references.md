### 引用类型

Move 支持两种类型的引用：不可变引用 `&` 和可变引用 `&mut`。不可变引用是只读的，不能修改相关的值或者其任何字段。可变引用通过写入该引用进行修改。Move 的类型系统强制执行所有权规则，以避免引用错误。

## 引用运算符

Move 提供了用于创建和扩展引用以及将可变引用转换为不可变引用的运算符。在这里和其他地方，我们使用符号 `e: T` 来表示“表达式 `e` 的类型是 `T`”

| 语法 | 类型 | 描述 |
| ------      | ------ |------ |
| `&e`        | `&T` 其中 `e: T` 和 `T` 是非引用类型      | 创建一个不可变的引用 `e`
| `&mut e`    | `&mut T` 其中 `e: T` 和 `T` 是非引用类型  | 创建一个可变的引用 `e`
| `&e.f`      | `&T` 其中 `e.f: T`                       | 创建结构 `e` 的字段 `f` 的不可变引用
| `&mut e.f`  | `&mut T` 其中`e.f: T`                    | 创建结构 `e` 的字段 `f` 的可变引用
| `freeze(e)` | `&T` 其中`e: &mut T`                     | 将可变引用 `e` 转换为不可变引用

`&e.f` 和 `&mut e.f` 运算符既可以用于在结构中创建一个新引用，也可以用于扩展一个现有引用，如下代码所示：

```move
let s = S { f: 10 };
let f_ref1: &u64 = &s.f; // 创建新引用
let s_ref: &S = &s;
let f_ref2: &u64 = &s_ref.f // 扩展现有的引用
```

只要两个结构都在同一个模块中，具有多个字段的引用表达式就可以嵌套使用：

```move
struct A { b: B }
struct B { c : u64 }
fun f(a: &A): &u64 {
  &a.b.c
}
```

最后请注意，不允许引用 "引用"（Move 不支持多重引用, 但 Rust 可以）：

```move
let x = 7;
let y: &u64 = &x;
let z: &&u64 = &y; // 编译失败
```

### 通过引用进行读写操作

可以通过生成引用值的副本来读取可变和不可变引用。只能写入可变引用。写入表达式 `*x = v` 会丢弃先前存储在 x 中的值，并用 `v` 的值进行覆盖。

两种操作都使用类似 C 语言的 `*` 语法。但是请注意，读取是一个表达式，而写入是一个必须发生在等号左侧的改动。

| 语法 | 类型 | 描述 |
| ---------- | ----------------------------------- | ----------------------------------- |
| `&e` | `T` 其中 `e` 为 `&T` 或 `&mut T` | 读取 `e` 所指向的值 |
| `*e1 = e2` | `()` 其中 `e1: &mut T` 和 `e2: T` | 用 `e2` 更新 `e1` 中的值 |

为了读取引用，相关类型必须具备[`copy` 能力](./chapter_19_abilities.html)，因为读取引用会创建值的新副本。此规则会防止复制资源的值：

```move=
fun copy_resource_via_ref_bad(c: Coin) {
    let c_ref = &c;
    let counterfeit: Coin = *c_ref; // Coin 不具备 `copy` 能力，所以此处编码是不被允许的
    pay(c);
    pay(counterfeit);
}
```

双重性：为了写入引用，相关类型必须具备[`drop` 能力](./chapter_19_abilities.html)，因为写入引用将丢弃（或删除）旧值。此规则可防止破坏资源的值：

```move=
fun destroy_resource_via_ref_bad(ten_coins: Coin, c: Coin) {
    let ref = &mut ten_coins;
    *ref = c; // 因为会销毁10个coin，所以此处编码是不被允许的
}
```

### `freeze` 推断

可变引用可以被用在预期不可变引用的上下文中：

```move
let x = 7;
let y: &mut u64 = &mut x;
```

这是因为编译器会在底层需要的地方插入 `freeze` 指令。以下是更多 `freeze` 实际推断行为的示例：

```move=
fun takes_immut_returns_immut(x: &u64): &u64 { x }

// freeze inference on return value
fun takes_mut_returns_immut(x: &mut u64): &u64 { x }

fun expression_examples() {
    let x = 0;
    let y = 0;
    takes_immut_returns_immut(&x); // no inference
    takes_immut_returns_immut(&mut x); // inferred freeze(&mut x) 只有在不可变转为可变时才会发生推断
    takes_mut_returns_immut(&mut x); // no inference

    assert!(&x == &mut y, 42); // inferred freeze(&mut y)
}

fun assignment_examples() {
    let x = 0;
    let y = 0;
    let imm_ref: &u64 = &x;

    imm_ref = &x; // no inference
    imm_ref = &mut y; // inferred freeze(&mut y)
}
```

###  子类型化

通过 `freeze` 推断，Move 类型检查器可以将 `&mut T` 视为 `&T` 的子类型。 如上面的代码所示，这意味着对于使用 `&T` 值的任何表达式，也可以使用 `&mut T` 值。此术语用于错误消息中，以简明扼要地表明在提供 `&T` 的地方需要 `&mut T` 。代码如下所示:

```move=
address 0x42 {
    module example {
        fun read_and_assign(store: &mut u64, new_value: &u64) {
            *store = *new_value
        }

        fun subtype_examples() {
            let x: &u64 = &0;
            let y: &mut u64 = &mut 1;

            x = &mut 1; // valid
            y = &2; // invalid!

            read_and_assign(y, x); // valid
            read_and_assign(x, y); // invalid!
        }
    }
}
```

运行此代码将产生类似以下错误消息：

```text
error:
    ┌── example.move:12:9 ───
    │
 12 │         y = &2; // invalid!
    │         ^ Invalid assignment to local 'y'
    ·
 12 │         y = &2; // invalid!
    │             -- The type: '&{integer}'
    ·
  9 │         let y: &mut u64 = &mut 1;
    │                -------- Is not a subtype of: '&mut u64'
    │

error:
    ┌── example.move:15:9 ───
    │
 15 │         read_and_assign(x, y); // invalid!
    │         ^^^^^^^^^^^^^^^^^^^^^ Invalid call of '0x42::example::read_and_assign'. Invalid argument for parameter 'store'
    ·
  8 │         let x: &u64 = &0;
    │                ---- The type: '&u64'
    ·
  3 │     fun read_and_assign(store: &mut u64, new_value: &u64) {
    │                                -------- Is not a subtype of: '&mut u64'
    │
```

当前唯一具有子类型的其他类型是 [tuple (元组)](./chapter_9_tuples.html)

### 所有权

Both mutable and immutable references can always be copied and extended _even if there are existing
copies or extensions of the same reference_:

_即使同一引用已经存在副本或扩展_，可变引用和不可变引用始终还是可以被复制和扩展，代码如下所示：

```move
fun reference_copies(s: &mut S) {
  let s_copy1 = s; // ok
  let s_extension = &mut s.f; // also ok
  let s_copy2 = s; // still ok
  ...
}
```

对于熟悉 Rust 所有权系统的程序员来说，这可能会令人惊讶，因为他们会拒绝上面的代码。Move 的类型系统在处理[副本](./chapter_10_variables.html#move-and-copy)方面更加宽松，但对于可变引用的写入，它的唯一所有权同样严格，仅有一个。

### 无法被存储的引用

引用和元组是唯一不能存储为结构体的字段值的类型，这也意味着它们不能存在于全局存储中。当 Move 程序终止时，程序执行期间创建的所有引用都将被销毁；它们完全是短暂的。这种不变量也适用于没有 [`store` 能力](./chatper_19_abilities.html)的类型的值，但请注意，引用和元组更进一步，从一开始就不允许出现在结构体之中。

这是 Move 和 Rust 之间的另一个区别，Rust 允许将引用存储在结构体之中。

目前，Move 无法支持这一点，因为引用无法被[序列化](https://en.wikipedia.org/wiki/Serialization)，但 _每个 Move 值都必须是可序列化的_。这个要求来自于 Move 的[持久化全局存储](./global-storage-structure.html)，它需要在程序执行期间序列化值以持久化它们。结构体可以写入全局存储，因此它们必须是可序列化的。

可以想象一种更奇特、更有表现力的类型系统，它允许将引用存储在结构体中，并禁止这些结构体存在于全局存储中。我们也许可以允许在没有 [`store` 能力](./chapter_19_abilities.html)的结构体内部使用引用，但这并不能完全解决问题：Move 有一个相当复杂的系统来跟踪静态引用安全性，并且类型系统的这一方面也必须扩展以支持在结构体内部存储引用。简而言之，Move 的类型系统（尤其是与引用安全相关的方面）需要扩展以支持存储的引用。随着语言的发展，我们会继续关注这一点。
