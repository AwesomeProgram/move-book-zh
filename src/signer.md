### 签名者类型

`签名者（signer）`是 Move 内置的资源类型。`签名者（signer）`是一种允许持有者代表特定`地址（address）`行使权力的[能力（capability）](https://en.wikipedia.org/wiki/Object-capability_model)。你可以将原生实现（native implementation）看作如下所示的一段代码：

```move
struct signer has drop { a: address }
```

`signer` 有点像 Unix [UID](https://en.wikipedia.org/wiki/User_identifier)，因为它表示一个通过 Move *之外*的代码（例如，通过检查一个加密签名或者密码）进行身份验证的用户。

### `signer` 与 `address` 的比较

Move 程序可以使用地址字面量创建任何`地址（address）`的值，而无需特殊许可,如下所示：

```move
let a1 = @0x1;
let a2 = @0x2;
// ... 等等，所有其他可能的地址
```

然而 `signer` 的值是特殊的，因为它们不能通过字面量或者指令创建，只能通过 Move 虚拟机（VM）创建。在虚拟机运行带有 `signer` 类型参数的脚本之前，它会自动创建 `signer` 值并将它们传递给脚本：

```move
script {
    use std::signer;
    fun main(s: signer) {
        assert!(signer::address_of(&s) == @0x42, 0);
    }
}
```

如果这个脚本不是从 `0x42` 的地址发送的，则此脚本将中止并返回代码 `0`。交易脚本可以有任意数量的 `signer`，只要 `signer` 参数排在其他参数前面。换句话说，所有 `signer` 参数都必须放在第一位，如下代码所示：

```move
script {
    use std::signer;
    fun main(s1: signer, s2: signer, x: u64, y: u8) {
        // ...
    }
}
```

这对于实现具有多方权限原子行为的*多重签名脚本（multi-signer scripts）*很有用。例如，上述脚本的扩展可以在 `s1` 和 `s2` 之间执行原子货币的交易。

### 签名者的操作符

`std::signer` 标准库模块为 `signer` 提供了两个实用函数：

| 函数                                        | 描述                                                          |
| ------------------------------------------- | ------------------------------------------------------------- |
| `signer::address_of(&signer): address`      | 返回由 `&signer` 包装的地址的值。                               |
| `signer::borrow_address(&signer): &address` | 返回由 `&signer` 包装的地址的引用。                           |


另外 `move_to<T>(&signer, T)` [全局存储](./global-storage-operators.md)操作符需要一个 `&signer` 参数在 `signer.address` 的帐户下发布资源 `T`。这确保了只有经过身份验证的用户才能在其地址下发布资源。


与简单的标量值不同，签名者的值是不可复制的，这意味着不管通过任何操作，无论是通过显式 [`copy`](./variables.md#移动和复制move-and-copy)指令还是通过[解引用（dereference）`*`](./references.md#reference-operators)）它们都不能被复制。
