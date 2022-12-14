# 你好，我是量子猿宇宙。本节为大家讲解 Move 语言的资源类型

## 整数类型

Move 支持三种无符号整数类型：`u8`、`u64` 和 `u128`。这些类型的值范围从 0 到最大值，最大值的具体取值取决于整数类型的位数大小。

| 类型                      | 取值范围                 |
|---------------------------|--------------------------|
| 无符号 8位 整数, `u8`     | 0 to 2<sup>8</sup> - 1   |
| 无符号 64位 整数, `u64`   | 0 to 2<sup>64</sup> - 1  |
| 无符号 128位 整数, `u128` | 0 to 2<sup>128</sup> - 1 |

在 Move 中这些类型的字面值指定为数字序列（例如：112）或十六进制文字（例如：0xFF），可以选择将字面值的类型定义为后缀, 例如 `112u8`。如果未指定类型，编译器将尝试从使用字面值的上下文推断类型。如果无法推断类型，则默认为 `u64`。如果字面值太大，超出其指定的或推断的大小范围则会报错。例如：

```jsx
// literals with explicit annotations;
let explicit_u8 = 1u8;
let explicit_u64 = 2u64;
let explicit_u128 = 3u128;

// literals with simple inference
let simple_u8: u8 = 1;
let simple_u64: u64 = 2;
let simple_u128: u128 = 3;

// literals with more complex inference
let complex_u8 = 1; // inferred: u8
// right hand argument to shift must be u8
let _unused = 10 << complex_u8;

let x: u8 = 0;
let complex_u8 = 2; // inferred: u8
// arguments to `+` must have the same type
let _unused = x + complex_u8;

let complex_u128 = 3; // inferred: u128
// inferred from function argument type
function_that_takes_u128(complex_u128);

// literals can be written in hex
let hex_u8: u8 = 0x1;
let hex_u64: u64 = 0xCAFE;
let hex_u128: u128 = 0xDEADBEEF;
```

### 整数类型的算术运算

每一种无符号整数类型都支持相同算术运算集。对于所有这些运算，左侧和右侧的这两个操作数必须是同一类型。如果你需要对不同类型的值进行运算，则需要首先执行类型的强制转换。同样如果你预计运算结果超过了当下整数类型的范围，则需要在执行运算之前将之转换为范围更大的整数类型。

这些算术运算在遇到异常时将会中止，而不是类型范围的上溢和下溢，或者甚至出现被零除时等数学整数未定义的输出也是不允许出现的。

| 句法 | 操作           | 中止条件                                |
| ------ | ------------| ---------------------------------------- |
| `+`    | 加法         | 结果对于整数类型来说太大了 |
| `-`    | 减法         | 结果小于零                 |
| `*`    | 乘法         | 结果对于整数类型来说太大了 |
| `%`    | 取余运算     | 除数为 `0`                       |
| `/`    | 截断除法     | 除数为 `0`                       |

### 整数类型的位运算

整数类型支持下列位运算，即将每个数字视为一系列单独的位：0 或 1，而不是整型数值。位运算不会因为意外而中止。

| 句法 | 操作   | 描述                                           |
| ------ | ----------- | ----------------------------------------------------- |
| `&`    | 按位 和 | 对每个位成对执行布尔值和          |
| `|`    | 按位 或  | 对每个位成对执行布尔值或
| `^`    | 按位 异或 | 对每个位成对执行布尔异或 |

### 整数类型的位移运算

与按位运算类似，每种整数类型都支持位移（bit shifts）。但与其他运算不同的是，右侧操作数要移动的位数必须始终是 `u8`  并且不需要与左侧操作数类型（要移位的数字）相匹配。

如果要移位的位数分别大于或等于 `8`、`64`、`128` 的 `u8`、`u64`、`u128` 则移位可以中止。

| 句法 | 操作   | 中止条件                                                    |
| ------ | ----------- | ------------------------------------------------------------ |
| `<<`   | 左移  | 要移位的位数大于整数类型的大小 |
| `>>`   | 右移 | 要移位的位数大于整数类型的大小 |

### 整数类型的比较运算

整数类型是 Move 中唯一可以使用比较运算符的类型。两个参数必须是同一类型。如果你需要比较不同类型的整数，则需要先转换其中一个类型让两边操作数可以进行比较。比较运算不会因为意外而中止。

| 句法 | 操作                |
| ------ | ------------------------ |
| `<`    | 小于                |
| `>`    | 大于             |
| `<=`   | 小于等于    |
| `>=`   | 大于等于 |

###  整数类型的相等运算

与 Move 中的所有具有 [`drop`](./chapter_19_abilities.html) 能力的类型一样，所有整数类型都支持 ["equal(等于)"](./chapter_11_equality.html) 和 ["not equal(不等于)](./chapter_11_equality.html)运算。两边的参数必须是同一类型。如果你需要比较不同类型的整数，则需要先转换其中一个类型让两边操作数可以进行比较。相等运算不会因为意外而中止。

| 句法 | 操作 |
| ------ | --------- |
| `==`   | 等于     |
| `!=`   | 不等于 |

### 整数类型的转换操作

一种大小的整数类型可以转换为另一种大小的整数类型。整数是 Move 中唯一支持强制转换的类型。强制转换不会截断。如果结果对于指定类型来说太大，则转换将会因为意外中止。

| Syntax     | 操作                                            | 中止条件                              |
| ---------- | ---------------------------------------------------- | -------------------------------------- |
| `(e as T)` | 将整数表达式 `e` 转换为整数类型 `T` | `e` 太大而不能表示为 `T` |

注意这里的类型 `e` 必须是 `u8`，`u64`，`u128` 类型之一，并且 `T` 也必须是 `u8`，`u64`，`u128` 其中的类型之一。如下所示：

- `(x as u8)`
- `(2u8 as u64)`
- `(1 + 3 as u128)`

与语言内置的其他标量值一样，整数值是隐式可复制的，这意味着它们可以在没有明确指令如 [`copy`](./variables.md#move-and-copy) 的情况下进行复制，这一点复用了 Rust 的特性。

#### 本节主要介绍了 Move 语言整数类型的算数运算、位运算、位移运算、比较运算、相等运算以及类型转换，下一节我们将详细介绍 Move 语言的布尔类型和地址类型。
