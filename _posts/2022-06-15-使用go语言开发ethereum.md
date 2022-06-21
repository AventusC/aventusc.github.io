---
date: 2022-06-21
layout: post
title: 使用Go语言开发ethereum
description: 
category: study
tags:
 - 学习
author: aventusc
---

# 使用Go语言开发ethereum

## 	相关概念解释

### Node.js 

**Node.js**是一个开源和跨平台的 JavaScript 运行时环境。 它几乎是任何类型项目的流行工具

### 	npm

1. **npm（“Node 包管理器”）**是 JavaScript 运行时 Node.js 的默认程序包管理器。

2. npm 由两个主要部分组成:

- 用于发布和下载程序包的 CLI（命令行界面）工具

- 托管 JavaScript 程序包的  [在线存储库](https://www.npmjs.com/)

  *两个组成部分之间的关系可以比喻为：[npmjs.com](https://npmjs.com/) 相当于一个物流集散中心，npmCLI就是里面的打工人，负责从卖家（npm包裹的作者）那里接收货物，并将这些货物分发给买家（npm包裹的用户）*

3. package.json

   每个 JavaScript 项目（无论是 Node.js 还是浏览器应用程序）都可以被当作 npm 软件包，并且通过  `package.json` 来描述项目和软件包信息。

   *我们可以将  `package.json` 视为快递盒子上的运输信息。*

   当运行  `npm init` 初始化 JavaScript/Node.js 项目时，将生成  `package.json` 文件，文件内的     内容(基本元数据)由开发人员提供：

- `name`：JavaScript 项目或库的名称。
- `version`：项目的版本。通常，在应用程序开发中，由于没有必要对开源库进行版本控制，因此经常忽略这一块。但是，仍可以用它来定义版本。
- `description`：项目的描述。
- `license`：项目的许可证。

​       `package.json` 还支持一个  `scripts` 属性，可以把它当作在项目本地运行的命令行工具。

4. **用户**如何使用npm

   `npm install`

   这是现在我们开发 JavaScript/Node.js 应用程序时最常用的命令。

   默认情况下，`npm install <package-name>` 将安装带有  `^` 版本号的软件包的最新版本。npm 项目上下文中的  `npm install` 将根据  `package.json` 规范将软件包下载到项目的  `node_modules` 文件夹中，从而升级软件包的版本（并重新生成  `package-lock.json` ）。 `npm install <package-name>` 可以基于  `^` 和  `〜` 版本匹配。

   如果要在全局上下文中安装程序包，可以在机器的任何地方使用它，则可以指定全局标志  `-g`（例如  [live-server](https://github.com/tapio/live-server)）。

5. **作者**如何使用npm

   `npm publish`

   将软件包发送到  [npmjs.com](https://npmjs.com/) 非常容易，因为我们只需要运行  `npm publish` 。 棘手的部分（并非专门针对 npm 软件包作者）是确定软件包的版本。

   根据  [semver.org](https://semver.org/) 的经验法则：

   1. 当你进行不兼容的 API 更改时使用 MAJOR 版本
   2. 以向后兼容的方式添加功能时使用 MINOR 版本
   3. 进行向后兼容的 bug 修复时使用 PATCH 版本

   在发布软件包时，遵循上述规则尤为重要，可以确保你不会破坏任何人的代码，因为 npm 中匹配的默认版本是`^`（又称下一个次版本）。

## 	客户端

*客户端是以太坊网络的入口。客户端需要广播交易和读取区块链数据。*

用Go初始化以太坊客户端是和区块链交互所需的基本步骤。首先，导入go-etherem的`ethclient`包并通过调用接收区块链服务提供者URL的`Dial`来初始化它。

1. 若本地没有以太坊客户端，则可以直接连接到infura网关。Infura管理着一批安全，可靠，可扩展的以太坊[geth和parity]节点，并且在接入以太坊网络时降低了新人的入门门槛。

   ```go
   client, err := ethclient.Dial("https://cloudflare-eth.com")
   ```

2. 若运行了本地geth实例，您还可以将路径传递给IPC端点文件。

   ```go
   client, err := ethclient.Dial("/home/user/.ethereum/geth.ipc")
   ```

## 	账户

### 	地址转换

*以太坊上的所有地址都是用16进制表示的。*

因此想要使用go-ethereum的账户地址，就需要先将其转化为go-ethereum中的common.Address类型，举个栗子：

```go
address := common.HexToAddress("0x71c7656ec7ab88b098defb751b7401b5f6d8976f")

fmt.Println(address.Hex()) // 0x71C7656EC7ab88b098defB751B7401B5f6d8976F
```

### 	账户余额

1. 读取余额的方式：

   调用客户端的`BalanceAt`方法，给它传递账户地址和可选的区块号。将区块号设置为`nil`将返回最新的余额。

   ```go
   account := common.HexToAddress("0x71c7656ec7ab88b098defb751b7401b5f6d8976f")
   balance, err := client.BalanceAt(context.Background(), account, nil)
   if err != nil {
     log.Fatal(err)
   }
   
   fmt.Println(balance) // 25893180161173005034
   ```

     传区块号能让读取该区块时的账户余额。区块号必须是`big.Int`类型。

   ```go
   blockNumber := big.NewInt(5532993)
   balance, err := client.BalanceAt(context.Background(), account, blockNumber)
   if err != nil {
     log.Fatal(err)
   }
   
   fmt.Println(balance) // 25729324269165216042
   ```

2. 精度转换

   以太坊中的数字是使用尽可能小的单位来处理的，因为它们是定点精度，在ETH中它是*wei*。要读取ETH值，必须计算`wei/10^18`。因为我们正在处理大数，我们得导入原生的Go`math`和`math/big`包。

   ```go
   fbalance := new(big.Float)
   fbalance.SetString(balance.String())
   ethValue := new(big.Float).Quo(fbalance, big.NewFloat(math.Pow10(18)))
   
   fmt.Println(ethValue) // 25.729324269165216041
   ```

   3. 待处理的余额

      如果需要知道待处理的账户余额，例如：在提交或等待交易确认后的余额。客户端提供了类似`BalanceAt`的方法，名为`PendingBalanceAt`，它接收账户地址作为参数。

      ```go
      pendingBalance, err := client.PendingBalanceAt(context.Background(), account)
      fmt.Println(pendingBalance) // 25729324269165216042
      ```

### 	生成新钱包

`生成私钥`

要生成一个新的钱包，我们需要导入go-ethereum`crypto`包，该包提供用于生成随机私钥的`GenerateKey`方法。

```
privateKey, err := crypto.GenerateKey()
if err != nil {
  log.Fatal(err)
}
```

通过导入golang`crypto/ecdsa`包并使用`FromECDSA`方法将其转换为字节。

```go
privateKeyBytes := crypto.FromECDSA(privateKey)
```

我们现在可以使用go-ethereum`hexutil`包将它转换为十六进制字符串，该包提供了一个带有字节切片的`Encode`方法。 然后我们在十六进制编码之后删除“0x”。

```go
fmt.Println(hexutil.Encode(privateKeyBytes)[2:]) 
// fad9c8855b740a0b7ed4c221dbad0f33a83a49cad6b3fe8d5817ac83d38b6a19
```

这就是用于签署交易的私钥，将被视为密码，永远不应该被共享给别人，因为谁拥有它可以访问你的所有资产。

`生成公钥`

由于公钥是从私钥派生的，因此go-ethereum的加密私钥具有一个返回公钥的`Public`方法。

```go
publicKey := privateKey.Public()
```

将其转换为十六进制的过程与我们使用转化私钥的过程类似。 我们剥离了`0x`和前2个字符`04`，它始终是EC前缀，不是必需的。

```go
publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
if !ok {
  log.Fatal("cannot assert type: publicKey is not of type *ecdsa.PublicKey")
}

publicKeyBytes := crypto.FromECDSAPub(publicKeyECDSA)
fmt.Println(hexutil.Encode(publicKeyBytes)[4:]) // 9a7df67f79246283fdc93af76d4f8cdd62c4886e8cd870944e817dd0b97934fdd7719d0810951e03418205868a5c1b40b192451367f28e0088dd75e15de40c05
```

`生成地址`

go-ethereum加密包有一个`PubkeyToAddress`方法，它接受一个ECDSA公钥，并返回公共地址。

```go
address := crypto.PubkeyToAddress(*publicKeyECDSA).Hex()
fmt.Println(address) // 0x96216849c49358B10257cb55b28eA603c874b05E
```

公共地址其实就是公钥的Keccak-256哈希，然后我们取**最后40个字符** **（20个字节）**并用“0x”作为前缀。 以下是使用 `golang.org/x/crypto/sha3` 的 Keccak256函数手动完成的方法。

```go
hash := sha3.NewLegacyKeccak256()
hash.Write(publicKeyBytes[1:])
fmt.Println(hexutil.Encode(hash.Sum(nil)[12:])) // 0x96216849c49358b10257cb55b28ea603c874b05e
```

### 	私钥存储

`keystore`

`keystore`是一个包含经过加密了的钱包私钥。go-ethereum中的keystore，每个文件只能包含一个钱包密钥对。

`生成新的keystore`

要生成keystore，首先您必须调用`NewKeyStore`，给它提供保存keystore的目录路径。然后，您可调用`NewAccount`方法创建新的钱包，并给它传入一个用于加密的口令。您每次调用`NewAccount`，它将在磁盘上生成新的keystore文件。

这是一个完整的生成新的keystore账户的示例。

```go
ks := keystore.NewKeyStore("./wallets", keystore.StandardScryptN, keystore.StandardScryptP)
password := "secret"
account, err := ks.NewAccount(password)
if err != nil {
  log.Fatal(err)
}

fmt.Println(account.Address.Hex()) // 0x20F8D42FB0F667F2E53930fed426f225752453b3
```

现在要导入您的keystore，您基本上像往常一样再次调用`NewKeyStore`，然后调用`Import`方法，该方法接收keystore的JSON数据作为字节。第二个参数是用于加密私钥的口令。第三个参数是指定一个新的加密口令，但我们在示例中使用一样的口令。导入账户将允许您按期访问该账户，但它将生成新keystore文件！有两个相同的事物是没有意义的，所以我们将删除旧的。

`导入已有的keystore`

这是一个导入keystore和访问账户的示例。

```go
file := "./wallets/UTC--2018-07-04T09-58-30.122808598Z--20f8d42fb0f667f2e53930fed426f225752453b3"
ks := keystore.NewKeyStore("./tmp", keystore.StandardScryptN, keystore.StandardScryptP)
jsonBytes, err := ioutil.ReadFile(file)
if err != nil {
  log.Fatal(err)
}

password := "secret"
account, err := ks.Import(jsonBytes, password, password)
if err != nil {
  log.Fatal(err)
}

fmt.Println(account.Address.Hex()) // 0x20F8D42FB0F667F2E53930fed426f225752453b3

if err := os.Remove(file); err != nil {
  log.Fatal(err)
}
```

## 	地址检查

### 	检查地址是否有效

可以使用简单的正则表达式来检查以太坊地址是否有效

```go
re := regexp.MustCompile("^0x[0-9a-fA-F]{40}$")

fmt.Printf("is valid: %v\n", re.MatchString("0x323b5d4c32345ced77393b3530b1eed0f346429d")) // is valid: true
fmt.Printf("is valid: %v\n", re.MatchString("0xZYXb5d4c32345ced77393b3530b1eed0f346429d")) // is valid: false
```

### 	检查地址是账户地址还是合约地址

如果该地址存储了字节码，则该地址就是智能合约。这是一个示例，在例子中，我们获取一个代币智能合约的字节码并检查其长度以验证它是一个智能合约：

```go
// 0x Protocol Token (ZRX) smart contract address
address := common.HexToAddress("0xe41d2489571d322189246dafa5ebde1f4699f498")
bytecode, err := client.CodeAt(context.Background(), address, nil) // nil is latest block
if err != nil {
  log.Fatal(err)
}

isContract := len(bytecode) > 0

fmt.Printf("is contract: %v\n", isContract) // is contract: true
```

如果地址上没有字节码，则代表这是一个标准的以太坊账户

```go
// a random user account address
address := common.HexToAddress("0x8e215d06ea7ec1fdb4fc5fd21768f4b34ee92ef4")
bytecode, err := client.CodeAt(context.Background(), address, nil) // nil is latest block
if err != nil {
  log.Fatal(err)
}

isContract = len(bytecode) > 0

fmt.Printf("is contract: %v\n", isContract) // is contract: false
```

## 	交易（Transaction）

*注意这里的交易`transaction` 是指广义的对以太坊状态的更改，它既可以指具体的以太币转账，代币的转账，或者其他对智能合约的创建或者调用。而不仅仅是传统意义的买卖交易。*

### 	查询区块

1. 区块头

   可以调用客户端的`HeaderByNumber`来返回有关一个区块的头信息。若传入`nil`，它将返回最新的区块头。

   ```go
   header, err := client.HeaderByNumber(context.Background(), nil)
   if err != nil {
     log.Fatal(err)
   }
   
   fmt.Println(header.Number.String()) // 5671744
   ```

2. 完整区块

   调用客户端的`BlockByNumber`方法来获得完整区块。可以读取该区块的所有内容和元数据，例如，区块号，区块时间戳，区块摘要，区块难度以及交易列表等等。

   ```go
   blockNumber := big.NewInt(5671744)
   block, err := client.BlockByNumber(context.Background(), blockNumber)
   if err != nil {
     log.Fatal(err)
   }
   
   fmt.Println(block.Number().Uint64())     // 5671744
   fmt.Println(block.Time().Uint64())       // 1527211625
   fmt.Println(block.Difficulty().Uint64()) // 3217000136609065
   fmt.Println(block.Hash().Hex())          // 0x9e8751ebb5069389b855bba72d94902cc385042661498a415979b7b6ee9ba4b9
   fmt.Println(len(block.Transactions()))   // 144
   ```

   调用`Transaction`只返回一个区块的交易数目。

   ```go
   count, err := client.TransactionCount(context.Background(), block.Hash())
   if err != nil {
     log.Fatal(err)
   }
   
   fmt.Println(count) // 144
   ```

### 	查询交易

我们可以通过调用`Transactions`方法来读取块中的事务，该方法返回一个`Transaction`类型的列表。 然后，重复遍历集合并获取有关事务的任何信息就变得简单了。

```go
for _, tx := range block.Transactions() {
  fmt.Println(tx.Hash().Hex())        // 0x5d49fcaa394c97ec8a9c3e7bd9e8388d420fb050a52083ca52ff24b3b65bc9c2
  fmt.Println(tx.Value().String())    // 10000000000000000
  fmt.Println(tx.Gas())               // 105000
  fmt.Println(tx.GasPrice().Uint64()) // 102000000000
  fmt.Println(tx.Nonce())             // 110644
  fmt.Println(tx.Data())              // []
  fmt.Println(tx.To().Hex())          // 0x55fE59D8Ad77035154dDd0AD0388D09Dd4047A8e
}
```

为了读取发送方的地址，我们需要在事务上调用`AsMessage`，它返回一个`Message`类型，其中包含一个返回sender（from）地址的函数。 `AsMessage`方法需要EIP155签名者，这个我们从客户端拿到链ID。

```go
chainID, err := client.NetworkID(context.Background())
if err != nil {
  log.Fatal(err)
}

if msg, err := tx.AsMessage(types.NewEIP155Signer(chainID)); err == nil {
  fmt.Println(msg.From().Hex()) // 0x0fD081e3Bb178dc45c0cb23202069ddA57064258
}
```

每个事务都有一个收据，其中包含执行事务的结果，例如任何返回值和日志，以及为“1”（成功）或“0”（失败）的事件结果状态。

```go
receipt, err := client.TransactionReceipt(context.Background(), tx.Hash())
if err != nil {
  log.Fatal(err)
}

fmt.Println(receipt.Status) // 1
fmt.Println(receipt.Logs) // ...
```

在不获取块的情况下遍历事务的另一种方法是调用客户端的`TransactionInBlock`方法。 此方法仅接受块哈希和块内事务的索引值。 您可以调用`TransactionCount`来了解块中有多少个事务。

```go
blockHash := common.HexToHash("0x9e8751ebb5069389b855bba72d94902cc385042661498a415979b7b6ee9ba4b9")
count, err := client.TransactionCount(context.Background(), blockHash)
if err != nil {
  log.Fatal(err)
}

for idx := uint(0); idx < count; idx++ {
  tx, err := client.TransactionInBlock(context.Background(), blockHash, idx)
  if err != nil {
    log.Fatal(err)
  }

  fmt.Println(tx.Hash().Hex()) // 0x5d49fcaa394c97ec8a9c3e7bd9e8388d420fb050a52083ca52ff24b3b65bc9c2
}
```

您还可以使用`TransactionByHash`在给定具体事务哈希值的情况下直接查询单个事务。

```go
txHash := common.HexToHash("0x5d49fcaa394c97ec8a9c3e7bd9e8388d420fb050a52083ca52ff24b3b65bc9c2")
tx, isPending, err := client.TransactionByHash(context.Background(), txHash)
if err != nil {
  log.Fatal(err)
}

fmt.Println(tx.Hash().Hex()) // 0x5d49fcaa394c97ec8a9c3e7bd9e8388d420fb050a52083ca52ff24b3b65bc9c2
fmt.Println(isPending)       // false
```

## 	转账以太币（ETH）

1. 假设已经连接了客户端，下一步就是加载你的私钥。

   ```go
   privateKey, err := crypto.HexToECDSA("fad9c8855b740a0b7ed4c221dbad0f33a83a49cad6b3fe8d5817ac83d38b6a19")
   if err != nil {
     log.Fatal(err)
   }
   ```

   

2. 之后我们需要获得帐户的随机数(nonce)。 每笔交易都需要一个nonce。 根据定义，nonce是仅使用一次的数字。 如果是发送交易的新帐户，则该随机数将为“0”。 来自帐户的每个新事务都必须具有前一个nonce增加1的nonce。很难对所有nonce进行手动跟踪，于是ethereum客户端提供一个帮助方法`PendingNonceAt`，它将返回你应该使用的下一个nonce。

   该函数需要我们发送的帐户的公共地址 - 这个我们可以从私钥派生。

   ```go
   publicKey := privateKey.Public()
   publicKeyECDSA, ok := publicKey.(*ecdsa.PublicKey)
   if !ok {
     log.Fatal("cannot assert type: publicKey is not of type *ecdsa.PublicKey")
   }
   
   fromAddress := crypto.PubkeyToAddress(*publicKeyECDSA)
   ```

   

































