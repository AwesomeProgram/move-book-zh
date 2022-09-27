# 你好，我是没有情感的量子猿宇宙。本节为大家讲解《10分钟掌握 Move 智能合约开发》

欢迎来到 Move 语言教程，在本教程中我们通过一些具体的步骤进行 Move 语言代码的开发，包括 Move 模块的设计、实现、单元测试和形式化验证。

整个过程共包含9个步骤：

- [Step 0: 安装 Move 开发环境](#Step0)
- [Step 1: 编写第一个 Move 模块](#Step1)
- [Step 2: 给模块添加单元测试](#Step2)
- [Step 3: 设计自己的 `BasicCoin` 模块](#Step3)
- [Step 4: `BasicCoin` 模块的实现](#Step4)
- [Step 5: 给 `BasicCoin` 模块添加单元测试](#Step5)
- [Step 6: 使用泛型编写 `BasicCoin` 模块](#Step6)
- [Step 7: 使用 `Move prover`](#Step7)
- [Step 8: 为 `BasicCoin` 模块编写形式化规范](#Step8)

## Step 0: 安装 Move 开发环境<span id="Step0"><span>

如果你还没有安装过 Move，首先打开命令行终端并 clone [Move代码库](https://github.com/move-language/move):

```bash
git clone https://github.com/move-language/move.git
```

进入到 `move` 文件夹下，执行 `dev_setup.sh` 脚本:

```bash
cd move
./scripts/dev_setup.sh -ypt
```

根据脚本命令的提示，按顺序安装 Move 的所有依赖项。
脚本将会将（move 命令所在路径）环境变量写入到 `~/.profile` 文件中。

执行如下命令使环境变量生效：

```bash
source ~/.profile
````

然后执行如下命令来安装 Move 命令行工具：

```bash
cargo install --path language/tools/move-cli
```

通过如下运行命令可以检查 move 命令是否可正常：

```bash
move --help
```

你应该会看到类似这样的内容以及许多命令的列表和描述：

```
move-package
Execute a package command. Executed in the current directory or the closest containing Move package

USAGE:
    move [OPTIONS] <SUBCOMMAND>

OPTIONS:
        --abi                          Generate ABIs for packages
...
```

如果想了解有哪些命令以及它们的用法, 执行命令或子命令时添加 `--help` 标记，此时会打印帮助文档。

### Visual Studio Code Move 支持

Visual Studio Code 有正式的 Move 语言支持, 你需要先安装 `move analyzer` :

```bash
cargo install --path language/move-analyzer
```

现在你可以打开 VS Code 并安装 Move 扩展插件了，在扩展页面下找到 `move-analyzer` 并安装即可。关于扩展的详细信息可以查看扩展的[README](https://github.com/move-language/move/tree/main/language/move-analyzer/editors/code)文件。

## Step 1: 编写第一个 Move 模块<span id="Step1"><span>

切换当前目录到 [`step_1/BasicCoin`](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_1/BasicCoin) 下，你将看到 `sources` 子目录 -- 这个包(package)下所有的 Move 代码都在此目录中，同时你还会看到一个 `Move.toml` 文件。该文件指定当前包的依赖列表和其他信息。
如果你熟悉 `Rust` 和 `Cargo`，那 `Move.toml` 文件类似 `Cargo.toml` 文件， `sources` 目录类似 `src` 目录（它们的作用是一样的）。

一起来看看 Move 语言代码，会看到如下内容：

```
// sources/FirstModule.move
module 0xCAFE::BasicCoin {
    ...
}
```

这是一个 `Move` [module（模块）](./chpater_1_modules-and-scripts.html)的定义。
模块是 Move 语言的代码块, 并且它使用指定的地址（address）进行定义 -- 模块只能在该地址下发布。
例如当前 `BasicCoin` 模块只能被发布在 `0xCAFE` 地址下。

再看看这个文件的下面一段代码，这里定义了一个具有字段 `value` 的[结构体](./chapter_16_structs-and-resources.html) `Coin`：

```
module 0xCAFE::BasicCoin {
    struct Coin has key {
        value: u64,
    }
    ...
}
```

再看文件剩余部分，我们会看到一个函数，它会创建一个 `Coin` 结构体，并将其保存在某个账号（account）下：

```
module 0xCAFE::BasicCoin {
    struct Coin has key {
        value: u64,
    }

    public fun mint(account: signer, value: u64) {
        move_to(&account, Coin { value })
    }
}
```

让我们来看看这个函数和它的含义:
* 此函数需要一个[`signer`](https://move-language.github.io/move/signer.html)参数 -- 表示受此特定地址控制的不可伪造的 token，和一个需要铸造指定数量的参数 `value`。
* 此函数使用给定的参数值铸造一个 `Coin`，然后通过 `move_to` 操作将其保存在（全局存储中）给定的 `account` 账户下。

我们需要确保此文件的代码能够执行，可以运行 `build` 命令去构建：

```bash
move build
```

* 你可以通过以下命令创建一个空的 Move 包（move package）:
    ```bash
    move new <pkg_name>
    ```
* Move 语言也支持命名地址的概念([named addresses](./address.html#named-addresses)), 命名地址是一种参数化 Move 源代码的方法，就是如果对 `NamedAddr` 使用的不同赋值编译，编译后会获得部署到你控制地址的不同字节码. 这种用法很常见，一般都将地址变量其定义在 `Move.toml` 文件的 `[addresses]` 部分. 例如 SomeNamedAddress：
    ```
    [addresses]
    SomeNamedAddress = "0xC0FFEE"
    ```

* [结构体](./chpater_16_structs-and-resources.html)可以通过给类型设定不同的[能力](./chapter_19_abilities.html)，让类型支持对应的行为。主要有以下有四种能力:
    - `copy`: 允许此类型的值被复制
    - `drop`: 允许此类型的值被弹出或丢弃
    - `store`: 允许此类型的值存在于全局存储的某个结构体中
    - `key`: 允许此类型作为全局存储中的键（具有 `key` 能力的类型才能保存到全局存储中）

    所以 `BasicCoin` 模块下的 `Coin` 结构体可以用作全局存储（global storage）的键（key）， 因为它不具备其他能力，它不能被拷贝，不能被丢弃, 也不能作为非 key 来保存在(全局)存储里。你无法复制 `Coin`，也不能意外丢弃它.
* [函数](./functions.html)默认是私有的（private）, 也可以声明为 `public`，[`public(friend)`](https://move-language.github.io/move/friends.html)，或 `public(script)`。`public(script)` 声明的函数可以被事务脚本调用。`public(script)` 函数也可以被其他 `public(script)` 函数调用。（注意：在最新版本的 Move 中，`public(script)` 已经被废弃，被 `public entry` 取代）
* `move_to` 是[五种不同的全局存储操作](./global-storage-operators.html)之一

## Step 2: 给模块添加单元测试<span id="Step2"><span>

现在已经完成了我们的第一个 Move 模块，确保铸币按预期的方式工作，需要进行测试。如果你熟悉 Rust 的话，Move 中的单元测试类似于 Rust 中的单元测试，测试代码使用 `#[test]` 注解，并像编写普通的 Move 函数一样。

可以通过 `move test` 命令来执行测试:

```bash
move test
```

现在我们来完成 [`FirstModule.move`](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_2/BasicCoin/sources/FirstModule.move) 文件的测试代码，这是第一个新测试用例:

```
module 0xCAFE::BasicCoin {
    ...
    // Declare a unit test. It takes a signer called `account` with an
    // address value of `0xC0FFEE`.
    #[test(account = @0xC0FFEE)]
    fun test_mint_10(account: signer) acquires Coin {
        let addr = signer::address_of(&account);
        mint(account, 10);
        // Make sure there is a `Coin` resource under `addr` with a value of `10`.
        // We can access this resource and its value since we are in the
        // same module that defined the `Coin` resource.
        assert!(borrow_global<Coin>(addr).value == 10, 0);
    }
}
```

这里声明了一个命名为 `test_mint_10` 的单元测试，它在 `account` 账户地址下铸造了一个包含 `value` 为 `10`的 `Coin`，然后通过 `assert!` 断言是否铸造成功，并检查保存在全局存储中的 `Coin` 的值是否与期望值一致。如果断言 `assert` 执行失败，则单元测试用例失败。

* 很多测试相关的注解（annotations）都值得仔细探索，在 `Step 5` 中会看到更多用法。

* 执行测试之前，需要设定 Move 标准库的依赖关系，找到 `Move.toml` 并在 `[dependencies]` 段内进行设定, 例如这段代码

  ```toml
  [dependencies]
  MoveStdlib = { local = "../../../../move-stdlib/", addr_subst = { "Std" = "0x1" } }
  ```
    注意需要修改 `<path_to_move>/language` 中的内容来匹配实际 `move-stdlib` 所在的目录路径。也可以用 `git` 方式指定依赖。

* 将断言值改为 `11` 将导致断言执行失败，找一个可以传递给 `move test` 命令的标志，当测试失败时它会显示全局状态。看起来像这样：
  ```
    ┌── test_mint_10 ──────
    │ error[E11001]: test failure
    │    ┌─ ./sources/FirstModule.move:24:9
    │    │
    │ 18 │     fun test_mint_10(account: signer) acquires Coin {
    │    │         ------------ In this function in 0xcafe::BasicCoin
    │    ·
    │ 24 │         assert!(borrow_global<Coin>(addr).value == 11, 0);
    │    │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Test was not expected to abort but it aborted with 0 here
    │
    │
    │ ────── Storage state at point of failure ──────
    │ 0xc0ffee:
    │       => key 0xcafe::BasicCoin::Coin {
    │           value: 10
    │       }
    │
    └──────────────────
  ```

* 找一个允许你收集测试覆盖率信息的标志，然后使用 `move coverage` 命令查看覆盖率统计信息和源码覆盖率。

## Step 3: 设计 `BasicCoin` 模块<span id="Step3"><span>

在本节中，我们将设计一个具有基础代币和余额（balance）接口功能的模块，通过他们来实现币的铸造，以及不同地址钱包之间的转账。

Move 语言的 `public function` 签名如下：

```
/// Publish an empty balance resource under `account`'s address. This function must be called before
/// minting or transferring to the account.
public fun publish_balance(account: &signer) { ... }

/// Mint `amount` tokens to `mint_addr`. Mint must be approved by the module owner.
public fun mint(module_owner: &signer, mint_addr: address, amount: u64) acquires Balance { ... }

/// Returns the balance of `owner`.
public fun balance_of(owner: address): u64 acquires Balance { ... }

/// Transfers `amount` of tokens from `from` to `to`.
public fun transfer(from: &signer, to: address, amount: u64) acquires Balance { ... }
```

接下来再看看这个模块所需要的各个数据结构.

Move 语言的模块没有自己的数据存储，相反的是 Move 语言提供按地址（addresses）索引的 **全局存储**（也是就是我们所说的区块链状态 blockchain state）。
每个地址之下包含 Move 模块（代码）和 Move 资源（数据）。

在 Rust 语法中，全局存储看起来有点像这样：

```rust
struct GlobalStorage {
    resources: Map<address, Map<ResourceType, ResourceValue>>
    modules: Map<address, Map<ModuleName, ModuleBytecode>>
}
```

The Move resource storage under each address is a map from types to values. (An observant reader might observe that this means each address can only have one value of each type.) This conveniently provides us a native mapping indexed by addresses.
In our `BasicCoin` module, we define the following `Balance` resource representing the number of coins each address holds:

每个地址下的 Move 资源存储是一个类型到数值的映射。需要注意每个地址, 每个类型下只能对应一个具体值。这会很方便地为我们提供了一个按地址索引的本地映射。在 `BasicCoin` 模块中，定义了下面的 `Balance`（钱包，余额）资源表示每个地址下持有的币的数量：

```
/// Struct representing the balance of each address.
struct Balance has key {
    coin: Coin // same Coin from Step 1
}
```

Move 区块链状态（`Move blockchain state`）看起来大致如下：
![](https://awesomeprogram.gitee.io/imgdata/tech_bc_move_state.png)

### `public(script)` functions

只有 `public(script)` 公开函数才能直接被交易调用，所以如果你想要直接从一个交易中调用 `transfer` 方法，那么需要将函数签改成如下格式：

```
public(script) fun transfer(from: signer, to: address, amount: u64) acquires Balance { ... }
```

### 与 以太坊/Solidity 的对比

In most Ethereum [ERC-20]((https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)) contracts, the balance of each address is stored in a _state variable_ of type <code>mapping(address => uint256)</code>. This state variable is stored in the storage of a particular smart contract.

在大多数以太坊 [ERC-20]((https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)) 智能合约中，各个账户地址下的余额保存在类型为 <code>mapping(address => uint256)</code>的 __状态变量__ 中，此状态变量存储在具体的智能合约内部存储中。

以太坊区块链的状态看起来大致如下:
![](https://awesomeprogram.gitee.io/imgdata/tech_bc_solidity_state.png)

## Step 4: 实现 `BasicCoin` 模块span id="Step4"><span>

在这一部分我们会仔细研究 [`BasicCoin.move`](https://github.com/move-language/move/blob/main/language/documentation/tutorial/step_4/BasicCoin/sources/BasicCoin.move) 内部方法的实现。

运行 move build 命令，先尝试编译一下代码：

```bash
move build
```

一起仔细看看 [`BasicCoin.move`](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_4/BasicCoin/sources/BasicCoin.move) 中内部方法的实现。

### `publish_balance` 方法

此方法将 `Balance` 资源发布到指定地址下。由于资源需要通过铸造或转账来接收代币，必须由用户先调用方法 `publish_balance` 才能接收钱，包括模块所有者。

此方法需要使用 `move_to` 操作来发布资源：

```
let empty_coin = Coin { value: 0 };
move_to(account, Balance { coin:  empty_coin });
```

### `mint` 方法

`mint` 方法将代币铸造到指定的帐户，在此要求 `mint` 必须得到模块所有者的批准。我们使用 `assert` 语句强制执行此操作：

```
assert!(signer::address_of(&module_owner) == MODULE_OWNER, errors::requires_address(ENOT_MODULE_OWNER));
```

Move 中的 `assert` 语句可以这样使用：`assert!(<predicate>, <abort_code>);`。这意味着如果 `<predicate>` 为假，则使用中止错误码 `<abort_code>` 来终止交易。此处的 `MODULE_OWNER` 和 `ENOT_MODULE_OWNER` 都是在模块开头定义的常量。`errors` 模块定义了我们可以使用的常见错误种类。重点是我们需要注意 Move 在其执行过程中是事务性的，因此如果触发[中止（abort）](./chapter_12_abort-and-assert.html)，并不用回退已执行状态的，因为该事务的任何更改都不会持久保存到区块链中。

然后将数量为 `amount` 的代币存入 `mint_addr` 的余额中。

```
deposit(mint_addr, Coin { value: amount });
```
### `balance_of` 方法

We use `borrow_global`, one of the global storage operators, to read from the global storage.

我们使用全局存储操其中的 `borrow_global` 从全局存储中读取资源（数据）。

```
borrow_global<Balance>(owner).coin.value
                 |       |       \    /
        resource type  address  field names
```
### `transfer` 方法

该函数从 `from` 的余额中提取代币并将代币存入 `to` 的余额中，我们一起来看看 `withdraw` 函数：

```
fun withdraw(addr: address, amount: u64) : Coin acquires Balance {
    let balance = balance_of(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin { value: amount }
}
```

在方法开始，我们断言提款账户有足够的余额。然后使用 `borrow_global_mut` 来获得全局存储的可变引用，并用 `&mut` 创建结构体字段的[可变引用](./chapter_8_references.html)。然后我们通过这个可变引用修改余额并返回一个带有提取金额的新代币。

### 练习
模块中有两个 TODO，可以试着练习一下：
- 完成 `publish_balance` 方法的实现，添加一个断言去检查 `account` 是否已经有了一个 `Balance` 资源。
- 实现 `deposit` 方法，获取 `address` 下 `balance` 余额的可变引用，然后增加 `amount` 的值。

**试想一下**
- 如果我们在余额中存入太多 token 会发生什么？


## Step 5: 在模块 `BasicCoin` 中添加和使用单元测试<span id="Step5"><span>

In this step we're going to take a look at all the different unit tests we've written to cover the code we wrote in step 4. We're also going to take a look at some tools we can use to help us write tests.

在这一步中，我们来看看为 `step 4` 中的代码编写的所有不同的单元测试用例，而且还有用来帮助我们编写测试用例的一些工具。

首先在可以运行一下 `move test` 命令。

```bash
move test
```

你应会看到如下类似的内容：

```
INCLUDING DEPENDENCY MoveStdlib
BUILDING BasicCoin
Running Move unit tests
[ PASS    ] 0xcafe::BasicCoin::can_withdraw_amount
[ PASS    ] 0xcafe::BasicCoin::init_check_balance
[ PASS    ] 0xcafe::BasicCoin::init_non_owner
[ PASS    ] 0xcafe::BasicCoin::publish_balance_already_exists
[ PASS    ] 0xcafe::BasicCoin::publish_balance_has_zero
[ PASS    ] 0xcafe::BasicCoin::withdraw_dne
[ PASS    ] 0xcafe::BasicCoin::withdraw_too_much
Test result: OK. Total tests: 7; passed: 7; failed: 0
```

Taking a look at the tests in the [`BasicCoin` module](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_5/BasicCoin/sources/BasicCoin.move) we've tried to keep each unit test to testing one particular behavior.

看看 [`BasicCoin` ](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_5/BasicCoin/sources/BasicCoin.move) 模块中的测试用例，我们试图让每个单元测试用例都会覆一个具体的行为。

### 练习

在查看测试这些用例之后，尝试在 `BasicCoin` 模块中编写一个单元测试 `balance_of_dne`，测试地址没有 `Balance` 资源的情况，调用 `balance_of` 方法的执行结果，它应该只有几行代码。


## Step 6: `BasicCoin` 模块泛型化<span id="Step6"><span>

在 Move 语言中，我们可以使用泛型来定义不同输入数据类型的函数和结构体。泛型是一个库代码的重要组成部分。在这部分中，我们将把简单 `BasicCoin` 模块泛型化，以便它可以提供给其他用户，当作可复用的模块库。

首先我们将类型参数添加到数据结构中：

```
struct Coin<phantom CoinType> has store {
    value: u64
}

struct Balance<phantom CoinType> has key {
    coin: Coin<CoinType>
}
```

We also add type parameters to our methods in the same manner. For example, `withdraw` becomes the following:

我们还以相同的方式将类型参数添加到方法中。例如 `withdraw` 是这样实现的：

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance {
    let balance = balance_of<CoinType>(addr);
    assert!(balance >= amount, EINSUFFICIENT_BALANCE);
    let balance_ref = &mut borrow_global_mut<Balance<CoinType>>(addr).coin.value;
    *balance_ref = balance - amount;
    Coin<CoinType> { value: amount }
}
```

此时熟悉以太坊的同学可能会注意到，该模块的用途与 [ERC20 token standard](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) 类似，ERC20 提供了在智能合约中实现可替代的代币的接口。使用泛型的一个关键优势是能够复用代码，因为泛型模块库已经提供了标准实现，并且实例化模块可以通过封装标准实现再次提供定制化的功能。

我们提供了一个名为 [`MyOddCoin`](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_6/BasicCoin/sources/MyOddCoin.move) 的小模块，它实例化了 `Coin` 类型，并且有其自定义转移策略：只能转移奇数个代币。其还包括两个 [tests](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_6/BasicCoin/sources/MyOddCoin.move) 用例来测试具体的行为。你可以使用在第 2 步和第 5 步中学到的命令来运行测试。

### `phantom` 类型参数

In definitions of both `Coin` and `Balance`, we declare the type parameter `CoinType` to be phantom because `CoinType` is not used in the struct definition or is only used as a phantom type parameter.

在 `Coin` 和 `Balance `的定义中，我们将类型参数 `CoinType` 声明为 phantom，因为 `CoinType` 不是在结构体定义中使用的，或者它只可以用作 phantom 类型参数。

### 检查形式化验证的依赖项

在继续下一步之前，确保你已安装所有的验证器依赖项，这对执行形式化验证非常重要。尝试运行 `boogie /version` ，如果出现错误消息“找不到命令：boogie”，则需要再次运行安装脚本并更新环境配置（`source ~/.profile`）：

```bash
# run the following in move repo root directory
./scripts/dev_setup.sh -yp
source ~/.profile
```

## Step 7:  使用 Move 验证器<span id="Step7"><span>

部署在区块链上的智能合约可能会操纵高价值资产。作为一种使用严格的数学方法来描述计算机系统的行为和推理正确性的技术，形式化验证可以被用于区块链，以防止智能合约中 bug 的产生。 [Move验证器](https://github.com/move-language/move/blob/main/language/move-prover/doc/user/prover-guide.md)是一种仍在进化中、用 Move 语言编写的智能合约形式化验证工具。用户可以使用[Move语言规范（Move Specification Language (MSL)）](https://github.com/move-language/move/blob/main/language/move-prover/doc/user/spec-lang.md)指定智能合约的功能属性，然后使用验证器自动静态检查它们。
为了说明如何使用验证器，我们在 [BasicCoin.move](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_7/BasicCoin/sources/BasicCoin.move) 中添加了这样的代码片段：

```
    spec balance_of {
        pragma aborts_if_is_strict;
    }
```

通俗地说，代码块 `spec balance_of {...}` 包含 `balance_of` 方法的属性规范说明。

Let's first run the prover using the following command inside [`BasicCoin` directory](https://github.com/move-language/move/tree/main/language/documentation/tutorial/step_7/BasicCoin/):

首先使用 move prove 命令运行验证器。

```bash
move prove
```

它会输出这样类似的错误信息：

```
error: abort not covered by any of the `aborts_if` clauses
   ┌─ ./sources/BasicCoin.move:38:5
   │
35 │           borrow_global<Balance<CoinType>>(owner).coin.value
   │           ------------- 由于执行失败这里发生中止
   ·
38 │ ╭     spec balance_of {
39 │ │         pragma aborts_if_is_strict;
40 │ │     }
   │ ╰─────^
   │
   =     at ./sources/BasicCoin.move:34: balance_of
   =         owner = 0x29
   =     at ./sources/BasicCoin.move:35: balance_of
   =         中止

Error: exiting with verification errors
```

验证器大体上告诉我们，我们需要明确指定函数 `balance_of` 中止的条件，中止原因是函数调用者 `owner` 在没有 `Balance<CoinType>` 资源的情况下调用 `borrow_global` 函数导致的。要去掉此错误信息，我们需要添加 `aborts_if` 条件：

```
    spec balance_of {
        pragma aborts_if_is_strict;
        aborts_if !exists<Balance<CoinType>>(owner);
    }
```

添加此条件后，再次尝试运行 prove 命令，确认没有验证错误：

```bash
move prove
```

除了中止条件，我们还想定义功能属性。在第 8 步中，我们将通过 `BasicCoin` 模块定义的方法指定属性来更详细地介绍验证器。


## 第 8 步：为 `BasicCoin` 模块编写正式规范<span id="Step8"><span>

### 取款方法
`withdraw` 取款方法的签名如下所示：

```
fun withdraw<CoinType>(addr: address, amount: u64) : Coin<CoinType> acquires Balance
```

该方法从地址 `addr` 中提取数量为 `amount` 的代币，然后创建数量为 `amount` 的代币并将其返回。当地址 `addr` 没有 `Balance<CoinType>` 资源或地址 `addr` 中的代币数量小于 `amount` 时，`withdraw` 方法就会中止执行。所以我们可以这样定义条件：
```
    spec withdraw {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance < amount;
    }
```

正如我们在这里看到的，一个 spec 块可以包含 `let` 绑定，它为表达式绑定变量名称。
`global<T>(address): T` 是一个返回 `addr` 资源值的内置函数。`balance` 是 `addr` 拥有的代币数量。
`exists<T>(address): bool` 是一个内置函数，如果指定的地址 address 在全局存储中有资源 `T`，则返回 `true` 。
两个 `aborts_if` 子句对应 `withdraw` 方法中止执行的两个条件。通常如果一个函数有多个 `aborts_if` 条件，这些条件之间是相互对等的。默认情况下如果用户想要指定中止条件，则需要列出所有可能的条件。否则验证器将会产生验证错误。
然而如果在 `spec` 代码块中定义了 `pragma aborts_if_is_partial`，则组合中止条件（或单个条件）仅 *imply* 函数中止。

下一步是定义功能属性，这些属性在下面的两个 `ensures` 子句中进行了描述。首先通过使用 `let post` 绑定，`balance_post` 表示地址 `addr` 执行后的余额，应该等于 balance 减去 amount。那么返回值 `result` 应该是一个价值为 `amount` 的代币。

```
    spec withdraw {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance < amount;

        let post balance_post = global<Balance<CoinType>>(addr).coin.value;
        ensures balance_post == balance - amount;
        ensures result == Coin<CoinType> { value: amount };
    }
```
### 存款方法

`deposit` 存款方法的签名如下所示：

```
fun deposit<CoinType>(addr: address, check: Coin<CoinType>) acquires Balance
```

该方法将代币 `check` 存入地址 `addr`。规范的定义像这样：

```
    spec deposit {
        let balance = global<Balance<CoinType>>(addr).coin.value;
        let check_value = check.value;

        aborts_if !exists<Balance<CoinType>>(addr);
        aborts_if balance + check_value > MAX_U64;

        let post balance_post = global<Balance<CoinType>>(addr).coin.value;
        ensures balance_post == balance + check_value;
    }
```

`balance` 表示 `addr` 执行前的代币数量，`check_value` 表示要存入的代币数量。当地址 `addr` 没有 `Balance<CoinType>` 资源或者 `balance` 与 `check_value` 之和大于 `u64` 的最大值。`deposit` 方法将会中止执行。该功能属性会检查执行后余额是否正确更新。

### 转账方法

`transfer` 转账方法的签名如下所示：

```
public fun transfer<CoinType: drop>(from: &signer, to: address, amount: u64, _witness: CoinType) acquires Balance
```

该方法将数量为 `amount` 的代币从帐户 `from` 转账给地址 `to`。规范的定义像这样：
```
    spec transfer {
        let addr_from = signer::address_of(from);

        let balance_from = global<Balance<CoinType>>(addr_from).coin.value;
        let balance_to = global<Balance<CoinType>>(to).coin.value;
        let post balance_from_post = global<Balance<CoinType>>(addr_from).coin.value;
        let post balance_to_post = global<Balance<CoinType>>(to).coin.value;

        ensures balance_from_post == balance_from - amount;
        ensures balance_to_post == balance_to + amount;
    }
```

`addr_from` is the address of `from`. Then the balances of `addr_from` and `to` before and after the execution are obtained.
The `ensures` clauses specify that the `amount` number of tokens is deducted from `addr_from` and added to `to`. However, the prover will generate the error information as below:

`addr_from` 是账户 `from` 的地址，然后获取执行前两个地址 `addr_from` 和 `to` 的余额。
 `ensures` 语句指定从 `addr_from` 减去 `amount` 数量的代币，并添加到 `to` 中。然而，执行 move prove 验证器会发生以下类似的错误：

```
error: post-condition does not hold
   ┌─ ./sources/BasicCoin.move:57:9
   │
62 │         ensures balance_from_post == balance_from - amount;
   │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   │
   ...
```

当 `addr_from` 等于 `to` 时，这个属性无效。因此，我们可以在方法中添加一个断言，`assert!(from_addr != to)` 来确保 `addr_from` 不等于 `to`。

### 练习

- 为` transfer` 方法实现 `aborts_if` 条件。
- 为 `mint` 和 `publish_balance` 方法实现一个规范。

#### 本节我们通过手把手的代码编写，仔细的构建了一个 Move 智能合约开发流程，通过本节学习你应该已经掌握了如何从0到1完成一个 Move 智能合约以及它的单元测试和形式化验证。对于有追求的同学，应该会感觉到这还不够，在剩余的教程中，我们会深入整个 Move 语言，以及更详细的介绍每一个概念和开发事项，目的是帮助我们可以成为一名专业的智能合约开发者。下一节我们会从头开始介绍 Move 的资源类型。