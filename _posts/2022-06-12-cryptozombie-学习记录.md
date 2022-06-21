---
date: 2022-06-21
layout: post
title: CryptoZombie学习记录
subtitle: solidity知识总结
description: 记录我使用cryptozombie学习solidity的过程
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559821648/theme8_knvabs.jpg
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559821648/theme8_knvabs.jpg
category: study
tags:
author: aventusc
---
# CryptoZombies学习

## 1.solidity结构

*类似于其他语言，solidity代码结构也大致如下：*

```
//开头先标明solidity编译器的版本号
pragma solidity ^x.x.x;
//代码主体在合约（contract）中
contract contract_name {
     
     //状态变量：声明在函数体外，存储在区块链上
     dataType permission dataName = xxx ;
     //常量：不可以被修改，因此很节约gas
     dataType permission constant dataName = xxx;
     
     //结构体
     struct struct_name {
            dataType dataName;
     }
     
     function func_name permission returns(args1,args2...){
            //声明在函数内的变量有：
            //1.局部变量：不存储在区块链上，通常存储在memory中
            //2.globe变量：提供有关区块链的信息
            dataType dataName = xxx;
     }
```

*注：所有变量、常量以及函数等的命名规则都最好遵循 **小驼峰命名规则** *

## 2.数据类型

#### 2.1无符号整数：**uint** 

在solidity中较常使用uint作为数据的类型，它是无符号数据类型，即其值不能是负数，取值范围：<br>

```
uint8 ranges from 0 to 2 ** 8 - 1  <br>
uint16 ranges from 0 to 2 ** 16 - 1 <br>
... <br>
uint256 ranges from 0 to 2 ** 256 - 1<br>
```

#### 2.2 有符号整数： **int**

取值范围：<br>

```
int256 ranges from -2 ** 255 to 2 ** 255 - 1 <br>
int128 ranges from -2 ** 127 to 2 ** 127 - 1 <br>
```

#### 2.3 布尔型：**boolean**

取值只有 ： 
```true 和 false ```

#### 2.4 地址：**address**

*因为solidity上的每一个操作都是基于账户交易的，而每个账户需要使用唯一的标识符来彼此区分，这个标识符就是 **address** ，即地址是所有合约的基础，所有合约都继承地址对象*

#### 2.5定长浮点型：**fixed/ufixed**

fixed表示有符号的固定位浮点数
ufixed表示无符号的固定位浮点数

## 3.数学运算

*在solidity中，数学运算与其他程序设计语言相同*

```
-   加法: x + y
-   减法: x - y
-   乘法: x * y
-   除法: x / y
-   取模 / 求余: x % y  (例如, 13 % 5 余 3, 因为13除以5，余3)
-   乘方: x ** y (例如, uint x = 5 ** 2; // equal to 5^2 = 25)
```

## 4.数组

*定义public数组后，solidity会自动创建getter方法*
*内置数组有： string、bytes、byte1、byte2、...、byte32*

#### 4.1 定长字节数组：**byte array**

*关键字有：bytes1, bytes2, bytes3, ..., bytes32。*<br>

- byte/byte1表示存储1个字节，即8位；byte2表示存储2个字节，步长为1字节；依次类推...<br>
- .length函数用来表示这个字节数组的长度（只读），返回值为定长字节数组类型的长度，而不是值的长度；<br>
- 可以通过下标来获取元素，但元素值不可修改（即只读）
  举个栗子：

```
uint256[5] public nums = [1,2,3,4,5];
```

#### 4.2 变长字节数组：**bytes**

特点：

-   支持length，push（在最后追加）方法
-   可以修改
-   支持索引,如果未分配空间（new分配空间），使用下标会报角标越界，其他的会自动分配空间
-   以十六进制格式赋值
    举个栗子：

```
uint256[] public nums = [1,2,3,4,5];
nums.push(6)
delete nums;
```

#### 4.3 new

*在函数中使用 **new** 关键字来为数组分配空间*

举个栗子：

```
uint8[] memory aa = new uint8[](10);// 10个长度的空间
```

## 5.函数

#### 5.1 定义方式

*注：函数默认是public*

```
//格式：
function 函数名（可选参数） 修饰符 返回值{
       函数体
}

```

注：为了区分公开函数和私有函数，一般命名私有函数时默认第一个字符为"_"

#### 5.2 函数修饰符

##### 5.2.1 pure 和 view（constant）

*Getter函数可以声明为view或者pure函数*<br>
*1.用view声明的函数只能读取状态，而不能修改状态*<br>
*2.用pure声明的函数既不能读取也不能修改状态*<br>
*使用view和pure关键字定义的函数不会改变以太坊区块链的状态，这意味着你调用这些函数时，你不会像区块链发送任何交易，因为交易被定义为从一个状态到另一个状态的状态栏变换*

```
以下几种情况认为是修改了状态：

-   写状态变量
-   触发事件（events）
-   创建其他的合约
-   使用 selfdestruct。
-   call 调用附加了以太币
-   调用了任何没有 view 或 pure 修饰的函数
-   使用了低级别的调用（low-level calls）
-   使用了包含特定操作符的内联汇编

以下几种情况被认为是读取了状态：

-   读状态变量
-   访问了 this.balance 或.balance
-   访问了 block, tx, msg 的成员 (msg.sig 和 msg.data 除外).
-   调用了任何没有 pure 修饰的函数
-   使用了包含特定操作符的内联汇编

```

##### 5.2.2 payable

*payable 关键字用来说明，这个函数可以接受以太币，如果没有这个关键字，函数会自动拒绝所有发送给它的以太币。*

##### 5.2.3 external

返回值函数声明中使用external表示：仅合约外部可以直接调用该函数，合约内部需要使用this关键字来调用。

##### 5.2.4 internal

返回值函数声明使用internal表示：仅合约内部和继承的合约可以调用该函数

#### 5.3函数权限修饰符

Solidity定义的函数的属性默认为**公共(public)**，这就意味着任何一方（或其他合约）都可以调用你合约里的函数，这样在某些情况下会给我们带来风险<br>
因此，将自己的函数定义为**私有(private)**，就意味着只有我们合约中的其他函数才能够调用这个函数。

#### 5.4 构造函数

*进入合约就执行，一般内含一些初始化后就不会改变的数据*
举个栗子：

```
constractor() public {
    owner = msg.sender;
}
```

#### 5.5 处理多返回值

```
function multipleReturns() internal returns(uint a, uint b, uint c) {
return (1, 2, 3); 
} 

function processMultipleReturns() external { 
uint a; uint b; uint c; 
// 这样来做批量赋值: 
(a, b, c) = multipleReturns(); 
} 

// 或者如果我们只想返回其中一个变量: 
function getLastReturnValue() external { 
uint c; 
// 可以对其他字段留空: 
(,,c) = multipleReturns(); 
}
```

## 6.Keccak256 和 类型转换

#### 6.1 Keccak256

EVM内部有一个散列函数kccak256，它用了SHA3版本，可以将任意长度的输入转换为一个固定长度为256位的16进制数字。
举个栗子：

```
// 生成一个0到100的随机数: 
uint randNonce = 0; uint random = uint(keccak256(now, msg.sender, randNonce)) % 100; 
randNonce++; uint random2 = uint(keccak256(now, msg.sender, randNonce)) % 100;
```

这个方法首先拿到 `now` 的时间戳、 `msg.sender`、 以及一个自增数 `nonce` （一个仅会被使用一次的数，这样我们就不会对相同的输入值调用一次以上哈希函数了）。

#### 6.2 类型转换

*因solidity中uint实际上默认为uint256，而我们有些时候并不需要为变量分配这么大的空间，因此可以使用类型转换，根据实际情况将其转换成uint8或者uint16等来节省空间和gas*
*并且更为常见的情况是，当我们需要对多个变量进行运算时，必须保证参与运算的变量都是同一数据类型，因此这种情况会经常使用到类型转换*

```

```

uint8 a = 5;
uint b = 6;
// 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8:
uint8 c = a * b;
// 我们需要将 b 转换为 uint8:
uint8 c = a * uint8(b);

```
uint8 a = 5; uint b = 6; 
// 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8: 
uint8 c = a * b; 
// 我们需要将 b 转换为 uint8: 
uint8 c = a * uint8(b);
//上面, `a * b` 返回类型是 `uint`, 但是当我们尝试用 `uint8` 类型接收时,
//就会造成潜在的错误。如果把它的数据类型转换为 `uint8`, 就可以了，编译器也不会出错。
```

## 7.Mapping（映射）

创建格式为：
*mapping(keyType => valueType*)<br>
keyType可以是任何内置值类型、字节、字符串或任何协定。<br>
valueType可以是任何类型，包括另一个映射或数组。<br>
*注：映射是不可迭代的。*

```
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中： 
mapping (address => uint) public accountBalance; 
//或者可以用来通过userId 存储/查找的用户名 
mapping (uint => string) userIdToName;
```

## 8.变量

#### 8.1全局变量（状态变量）

*状态变量默认是私有的，可以使用public来修饰*
*在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是 `msg.sender`，它指的是当前调用者（或智能合约）的 `address`。*
*注意：在 Solidity 中，功能执行始终需要从外部调用者开始。 一个合约只会在区块链上什么也不做，除非有人调用其中的函数。所以 `msg.sender`总是存在的。*

```
//以下是使用 `msg.sender` 来更新 `mapping` 的例子：
mapping (address => uint) favoriteNumber; 
function setMyNumber(uint _myNumber) public
{ 
// 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下 
favoriteNumber[msg.sender] = _myNumber; 
// 存储数据至映射的方法和将数据存储在数组相似 
}
```

#### 8.2局部变量

*局部变量位于函数体中或函数名后的（）中，不能使用public关键字来修饰*

注：为了区分全局变量和局部变量，局部变量声明时的第一个字符都使用"_"。

#### 8.3 Storage和Memory

***Storage*** 变量是指永久存储在区块链中的变量。\
***Memory*** 变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。\
大多数时候你都用不到这些关键字，默认情况下 Solidity 会自动处理它们。 状态变量（在函数之外声明的变量）默认为“存储”形式，并永久写入区块链；而在函数内部声明的变量是“内存”型的，它们函数调用结束后消失。

## 9.事件

#### 9.1 定义：

**事件**是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。
举个栗子：

```
// 这里建立事件
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  //触发事件，通知app
  IntegersAdded(_x, _y, result);
  return result;
}
```

#### 9.2具体操作：

用户界面（当然也包括服务器应用程序）可以监听区块链上正在发送的事件，而不会花费太多成本。一旦它被发出，监听该事件的listener都将收到通知。而所有的事件都包含了 `from` ， `to` 和 `amount` 三个参数，可方便追踪事务。 为了监听这个事件，你可以使用如下代码（javascript 实现）：

```
var abi = /* abi 由编译器产生 */;
var ClientReceipt = web3.eth.contract(abi);
var clientReceipt = ClientReceipt.at("0x1234...ab67" /* 地址 */);

var event = clientReceipt.IntegersAdded();

// 监视变化
event.watch(function(error, result){
    // 结果包括对 `Deposit` 的调用参数在内的各种信息。
    if (!error)
        console.log(result);
});

// 或者通过回调立即开始观察
var event = clientReceipt.IntegersAdded(function(error, result) {
    if (!error)
        console.log(result);
});

```

## 10.继承（inheritance）

同理于其他面向对象的高级语言中的继承。solidity中使用`is`关键字来表示继承关系

```
//继承基本格式：
//父合约
contract A{

}
//子合约
contract B is A{

}
```

## 11.引入（import）

同理于其他高级语言，solidity使用import实现在一个文件中导入另一个文件

```
import "./someothercontract.sol"; 
contract newContract is SomeOtherContract { 

}
```

## 12.接口（interface）

接口的存在就是为了合约之间的通信。\
接口创建的基本格式：

```
contract modelInterface{
   function a(address _myaddress) public view returns (uint);
   function b(address _myaddress) public view returns (uint);
   ...
}
```

注意：接口内没有任何函数是已实现的，同时还有如下限制：

-   不能继承其它合约，或接口。
-   不能定义构造器
-   不能定义变量
-   不能定义结构体
-   不能定义枚举类

## 13.Ownable Contracts

下面是一个 `Ownable` 合约的例子： 来自 **_ OpenZeppelin _** Solidity 库的 `Ownable` 合约。

```
/** * @title Ownable * @dev The Ownable contract has an owner address, and provides basic authorization control * functions, this simplifies the implementation of "user permissions". */ 
contract Ownable { 
address public owner; 
event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

/** * @dev The Ownable constructor sets the original `owner` of the contract to the sender * account. */ 
function Ownable() public { 
owner = msg.sender; 
} 

/** * @dev Throws if called by any account other than the owner. */ 
modifier onlyOwner() { 
require(msg.sender == owner);
_; 
} 

/** * @dev Allows the current owner to transfer control of the contract to a newOwner. * @param newOwner The address to transfer ownership to. */ 
function transferOwnership(address newOwner) public onlyOwner { 
require(newOwner != address(0)); 
OwnershipTransferred(owner, newOwner); 
owner = newOwner; 
} 
}
```

Ownable合约的作用：

1.  合约创建，构造函数先行，将其 `owner` 设置为`msg.sender`（其部署者）
2.  为它加上一个修饰符 `onlyOwner`，它会限制陌生人的访问，将访问某些函数的权限锁定在 `owner` 上。
3.  允许将合约所有权转让给他人。

## 14.时间单位

Solidity 使用自己的本地时间单位。

变量 `now` 将返回当前的unix时间戳（自1970年1月1日以来经过的秒数）。我写这句话时 unix 时间是 `1515527488`。

Solidity 还包含`秒(seconds)`，`分钟(minutes)`，`小时(hours)`，`天(days)`，`周(weeks)` 和 `年(years)` 等时间单位。它们都会转换成对应的秒数放入 `uint` 中。所以 `1分钟` 就是 `60`，`1小时`是 `3600`（60秒×60分钟），`1天`是`86400`（24小时×60分钟×60秒），以此类推。

## 15.结构体（struct）

结构体的创建类似于go语言中的结构体创建，举个栗子：

```
pragma solidity ^0.4.4;
 
contract Students {
 
    struct Person {
        uint age;
        uint stuID;
        string name;
    }
    //括号内的参数和结构体内的变量一一对应，等同于age=18,stuID=101,name="james"
    Person _person = Person(18,101,"james");
    Person jack=Person(28,102,"jack"); 
}
```

关于可见性，目前结构体只支持internal，所以结构体只能在合约内部和子合约内使用。包含结构体的函数必须显性声明为internal

## 16.预防溢出

*在编写智能合约的时候需要注意的一个主要的安全特性：防止溢出和下溢。*\

1. 为了防止这些情况，OpenZeppelin 建立了一个叫做 SafeMath 的  **_库_** (***library***)，默认情况下可以防止这些问题。为了防止溢出和下溢，我们可以在我们的代码里找 `+`， `-`， `*`， 或 `/`，然后替换为 `add`, `sub`, `mul`, `div`.

2. SafeMath 库有四个方法 — `add`， `sub`， `mul`， 以及 `div`。现在我们可以这样来让 `uint256` 调用这些方法：

```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10
```

3. `using`关键字
   它可以实现自动把库里的所有方法添加给一个数据类型：

```
using SafeMath for uint; 
// 这下我们可以为任何 uint 调用这些方法了 
uint test = 2; test = test.mul(3); // test 等于 6 了 
test = test.add(5); // test 等于 11 了
```

## 17.Web3.js

Web3.js 有两个方法来调用我们合约的函数: `call` and `send`.

1. `call` 用来调用 `view` 和 `pure` 函数。它只运行在本地节点，不会在区块链上创建事务。
2. `send` 将创建一个事务并改变区块链上的数据。你需要用 `send` 来调用任何非 `view` 或者 `pure` 的函数。\
3. **注意:**  `send` 一个事务将要求用户支付gas，并会要求弹出对话框请求用户使用 Metamask 对事务签名。在我们使用 Metamask 作为我们的 web3 提供者的时候，所有这一切都会在我们调用 `send()` 的时候自动发生。

## 18.Error报错控制

**错误将撤销事务期间对状态所做的所有更改**

你可以通过调用`require`，`revert`，`assert`函数来抛出一个错误。

- `require`用于在执行前验证输入和条件。
- `revert`类似于`require`。有关详细信息，请参阅下面的代码。
- `assert`用于检查不应该为假的代码。断言失败可能意味着存在错误。

## 19.Fallback回退函数

一个合约最多可以有一个回退函数，函数可声明为： 

`fallback () external [payable]` 或 `fallback (bytes calldata _input) external [payable] returns (bytes memory _output)` (都没有 `function` 关键字)。 它可以是 `virtual` 的，可以被重载也可以有`modifier` 。

### 19.1 fallback 函数的场景：

- 当调用的函数找不到时，就会调用默认的`fallback`函数（如果函数标识符与智能合约中的任何可用函数都不匹配）。
- 当向某个合约中发送以太，接收合约没有`receive`函数，不管`msg.data`是否为空，都会调用`fallback`函数。
- 当向某个合约发送以太时，接收合约有`receive`函数，但`msg.data`不为空，这种情况也会调用`fallback`函数；除非`msg.data`为空，此时就会调用`receive`函数。

## 20.转账

### 20.1 如何发送以太？

你可以向其他合约发送以太通过：

- `transfer` (2300 gas, 抛出错误)
- `send` (2300 gas, 返回布尔值)
- `call` (转发所有 gas 或设置 gas，返回布尔值)

### 20.2 如何接收以太？

接收 Ether 的合约必须至少具有以下函数之一

- `receive() external payable`
- `fallback() external payable`

如果 `msg.data` 为空，则调用 `receive()` ，否则调用 `fallback()`。

### 20.3 您应该使用哪种方法？

`call` 是 2019 年 12 月后推荐使用与重入防护结合使用的方法。

通过以下方式防止重入

- 在调用其他合约之前进行所有状态更改，遵循先判断，后写入变量在进行外部调用的编码规范（Checks-Effects-Interactions）；
- 使用防重入锁

注意点：在使用 call.value 时候，如果调用者是合约对象，会执行 fallback。

具体案例可参考此文章。

https://mp.weixin.qq.com/s/nveh1aVTxIBUDTzzQqSuIQ
