### 布尔类型

布尔 `bool` 是 Move 的基本类型，有 `true` 和 `false` 两个值。布尔类型字面值只能是 `true` 或者 `false` 其中的一个 。

### 布尔类型的逻辑运算

`bool` supports three logical operations:

| Syntax                    | Description                  | Equivalent Expression                                               |
| ------------------------- | ---------------------------- | ------------------------------------------------------------------- |
| `&&`                      | short-circuiting logical and | `p && q` is equivalent to `if (p) q else false`                     |
| <code>&vert;&vert;</code> | short-circuiting logical or  | <code>p &vert;&vert; q</code> is equivalent to `if (p) true else q` |
| `!`                       | logical negation             | `!p` is equivalent to `if (p) false else true`                      |

`bool` 类型支持如下三种逻辑运算：

| 句法 | 描述                  | 等价表达式                           |
| ------ | ---------------------------- | ----------------------------------------------- |
| `&&`   | 短路逻辑与 | `p && q` 等价于 `if (p) q else false` |
| <code>&vert;&vert;</code>   | 短路逻辑或(short-circuiting logical or)  | `p || q` 等价于 `if (p) true else q`  |
| `!`    | 逻辑非            | `!p` 等价于 `if (p) false else true`  |

布尔类型还可用于如下多个 Move 的控制流结构：

- `[if (bool) { ... }](<./chapter_13_conditionals.html>)`
- `[while (bool) { .. }](<./chapter_14_loops.html>)`
- `[assert!(bool, u64)](<./chapte_12_abort-and-assert.html>)`

与语言内置的其他标量值一样，布尔值是隐式可复制的，这意味着它们可以在没有明确指令如 [`copy`](./variables.md#move-and-copy) 的情况下进行复制，这一点复用了 Rust 的特性。