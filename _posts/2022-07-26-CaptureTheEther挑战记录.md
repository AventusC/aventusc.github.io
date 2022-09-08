---
date: 2022-07-26
layout: post
title: CaptureTheEther挑战记录
description: 合约漏洞攻击记录
image: https://user-images.githubusercontent.com/90261136/189131394-6003b7f8-2962-4b78-98fb-0cb628b6c8cd.jpg
optimized_image: https://user-images.githubusercontent.com/90261136/189131394-6003b7f8-2962-4b78-98fb-0cb628b6c8cd.jpg
subtitle:
category: STUDY
tags:
 - solidity
 - security
author: aventusc
---

# CTE挑战解答

*前言：*

*这个挑战游戏的设计比ethernaut好了几百倍！！！首先攻击要求给你说清楚了，完成条件也给你设置得很明确，实在是太良心了！*

## warm up

这三个关卡由于题目给出的信息已经说明得很清楚了，所以比较简单，直接略过

## Guess the number

### 要求

- 猜数字

### 漏洞

- 没什么好说的，直接填42

### 攻略

- 无



## Guess secret number

### 要求

- 要求 `keccak256(n) == 0xdb81b4d58595fbbbb592d3661a34cdca14d7ab379441400cbfa1b78bc447c365`, n 为用户输入

### 漏洞

- n的类型为uint8，即其范围为0-256，可以直接写一个脚本来爆破

### 攻略



## Token sale

### 要求

- 将合约中的余额减小到小于1ether

### **漏洞**

未使用safeMath，存在上溢出的可能

### 攻略

- 漏洞位置：

  ```solidity
   require(msg.value == numTokens * PRICE_PER_TOKEN);
  ```

  这一条语句存在上溢出的风险

因此，直接使用一个很大的数num，其满足num>=2^256/10^18+1即可实现攻击。

### 疑问

按道理我上面的思路是没有问题的，但由于我执行buy操作的时候gas fee超出限制，导致revert，所以我无法正常地完成交易。



## Token whale

### 要求

- 将账户里的1000个token超级加倍到超过1000000个token

### 漏洞

- _transfer函数没有对溢出进行检查

  ![image-20220805165330585](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220805165330585.png)

- transferFrom函数没有对msg.sender的balance进行检查

  ![image-20220805165342501](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220805165342501.png)

- 因此可以使用transferFrom函数调用_transfer函数的过程中产生下溢。

### 攻略

```solidity
pragma solidity ^0.4.21;
// 攻击顺序，先调用transferFrom，转走1001个token给任意地址。然后调用_transfer函数，让msg.sender的余额出现下溢出，
// 从而其余额变为2**256，然后再执行一遍上述的转账操作，即可实现token的超级加倍。
// 要点，一个实例地址，一个player地址A，一个msg.sender地址B
contract TokenWhaleChallenge {
    address player;

    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    string public name = "Simple ERC20 Token";
    string public symbol = "SET";
    uint8 public decimals = 18;

    function TokenWhaleChallenge(address _player) public {
        player = _player;
        totalSupply = 1000;
        balanceOf[player] = 1000;
    }

    function isComplete() public view returns (bool) {
        return balanceOf[player] >= 1000000;
    }

    event Transfer(address indexed from, address indexed to, uint256 value);
    
    // 仅供内部调用，没有对溢出进行检查，所以只要value大于1000即可造成下溢出。
    function _transfer(address to, uint256 value) internal {
        balanceOf[msg.sender] -= value;
        balanceOf[to] += value;

        emit Transfer(msg.sender, to, value);
    }

    function transfer(address to, uint256 value) public {
        require(balanceOf[msg.sender] >= value);
        require(balanceOf[to] + value >= balanceOf[to]);

        _transfer(to, value);
    }

    event Approval(address indexed owner, address indexed spender, uint256 value);

    function approve(address spender, uint256 value) public {
        allowance[msg.sender][spender] = value;
        emit Approval(msg.sender, spender, value);
    }

    // 用player先给msg.sender转账小于1000数量的币，引起msg.sender的余额下溢。
    function transferFrom(address from, address to, uint256 value) public {
        // 没有对msg.sender的余额进行检查，所以可以对msg.sender进行攻击
        require(balanceOf[from] >= value);
        require(balanceOf[to] + value >= balanceOf[to]);
        // 要求from授权给msg.sender的token数大于value
        require(allowance[from][msg.sender] >= value);
        // 要求from授权给msg.sender的token数减去value
        allowance[from][msg.sender] -= value;
        // 调用_transfer函数
        _transfer(to, value);
    }
}
```

- 部署该合约

- 使用小号B调用transferFrom，从A转账至少501给B。

  顺应的自动调用了_transfer函数，将B的余额减至小于0，从而出现下溢出

- 再调用transfer函数，给A转账100000000，即可。

## Retirement fund

### 要求

- 从这"可怜人"的养老金里拿走他所有的钱(虽然部署合约的养老金是从我账户上扣的...)

### 漏洞

- 考点就是selfdestruct函数的运用

![image-20220809143153398](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220809143153398.png)

- 让withdrawn>0的思路一：让startBalance>address(this).balance；

  很显然，这点是做不到的

- 另一个思路就是使用selfdestruct函数来强行给address(this).balance转账，从而使address(this).balance>startBalance，造成withdrawn出现下溢，即完成攻击。

### 攻略

- 新建一个合约A，往里面充点钱

  ```solidity
  pragma solidity ^0.6.0;
  
  contract A{
      address public owner;
      uint public balance;
      address public instance;
      constructor(address _owner,address _instance) public payable{
          owner = _owner;
          instance = _instance;
          balance = msg.value;
      }
  
      function withdraw() public payable{
          selfdestruct(instance);
      }
  
  }
  ```

- 调用合约A的withdraw，将余额强行转给合约B的实例地址。

- 调用penaltycollect，即完成挑战。





## Mapping

### 要求

- 更改mapping的映射对应关系（作者使用一个数组来存储key-value)

### 漏洞

- 要求我们深入理解EVM的存储方式

- 有关于为什么set函数里，map.length是和key进行比较的原因如下：

  ![img](https://ctfiot.oss-cn-beijing.aliyuncs.com/uploads/2022/04/6-1650539627.png)

### 攻略（待完善）



## Donation

### 要求

- 使合约余额为0

### 漏洞

- solidity中**结构体的初始化**必须要`声明存储方式`，要么是storage要么是memory。如果没有`显式声明`存储方式（也就是没写），那么就**默认**使用`storage`作为存储方式

### 攻略（待完善）



## Fifty years

### 要求

- 取出合约中所有的余额

### 漏洞（待完善）



### 攻略（待完善）



## badc0de

### 要求

- 盗用合约主人的身份信息

### 漏洞

- IName接口可以直接重写一个，然后返回bytes32为smarx

  ![image-20220811103633531](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220811103633531.png)

- 剩下的isBadCode函数，如需满足条件就需要暴力破解。因为智能合约地址是由部署交易的发送者地址和交易的随机数nonce派生而生。

  所以我们需要写一个程序来计算nonce从0开始依次增加再与部署合约的发送者地址进行派生尝试。

  这个过程需要花费很长的时间。

### 攻略（待完善）



## Public Key

### 要求

- 找到账户的公钥

### 漏洞

- 题目提供我们一个合约的地址，要求我们得到该地址的公钥。 这里涉及到以太坊的**交易签名算法**。当我们知道 r、s、v 和 hash时我们可以恢复出公钥。

### 攻略

- 方法一：

  该owner账户有一个可以调用的交易`authenticate`，可以调用这个函数，然后再使用一个节点来获取它的数据，重新计算交易哈希，并恢复地址的公钥。

  ```solidity
  //莫约是JavaScript脚本
  const firstTxHash = `0xabc467bedd1d17462fcc7942d0af7874d6f8bdefee2b299c9168a216d3ff0edb`;
  const firstTx = await eoa.provider.getTransaction(firstTxHash);
  expect(firstTx).not.to.be.undefined;
  console.log(`firstTx`, JSON.stringify(firstTx, null, 4));
  // ...
  // signature values
  // "r": "0xa5522718c0f95dde27f0827f55de836342ceda594d20458523dd71a539d52ad7",
  // "s": "0x5710e64311d481764b5ae8ca691b05d14054782c7d489f3511a7abf2f5078962",
  // "v": 41,
  ```

  ECDSA 签名由`(r,s,v)`值组成，但实际签名的是*什么*？以太坊根据[EIP 155对](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-155.md)**序列化的交易哈希**进行签名，即.`keccak256(rlp(nonce, gasprice, startgas, to, value, data, chainid, 0, 0))`

  > 这个散列**不是**通常所说的*交易散列*。交易哈希还包括在签名创建时当然不知道的实际签名值。

  使用 ethers.js，我们可以通过序列化事务来为有序参数创建正确的**递归长度前缀 (rlp) 编码：**

  ```solidity
  const txData = {
    gasPrice: firstTx.gasPrice,
    gasLimit: firstTx.gasLimit,
    value: firstTx.value,
    nonce: firstTx.nonce,
    data: firstTx.data,
    to: firstTx.to,
    chainId: firstTx.chainId,
  };
  const signingData = ethers.utils.serializeTransaction(txData);
  const msgHash = ethers.utils.keccak256(signingData);
  ```

  

- 方法二：

  调用`recoverPublicKey`导致我们可以提交未压缩的公钥来解决这个挑战

  ```solidity
  const signature = { r: firstTx.r, s: firstTx.s, v: firstTx.v };
  let rawPublicKey = ethers.utils.recoverPublicKey(msgHash, signature);
  // const compressedPublicKey = ethers.utils.computePublicKey(rawPublicKey, true);
  // need to strip of the 0x04 prefix indicating that it's a raw public key
  expect(rawPublicKey.slice(2, 4), "not a raw public key").to.equal(`04`);
  rawPublicKey = `0x${rawPublicKey.slice(4)}`;
  console.log(`Recovered public key ${rawPublicKey}`);
  // 0x613a8d23bd34f7e568ef4eb1f68058e77620e40079e88f705dfb258d7a06a1a0364dbe56cab53faf26137bec044efd0b07eec8703ba4a31c588d9d94c35c8db4
  
  tx = await contract.authenticate(rawPublicKey);
  ```

  *所有这一切的一个有趣结果是，如果您从未从您的帐户**发送**交易，那么在区块链上唯一可见的就是您的地址——一个 sha256 哈希。这使得从未发送过交易的账户是量子安全的。*

## Account Takeover



### 要求

- 从owner的账户发送一个交易出来

  （也就是获取owner的授权或者owner的账号）

### 漏洞

- owner的地址设定为private

  ![image-20220811113139593](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220811113139593.png)

- private，即**只能**在其**所在的合约**中调用和访问，即使是其子合约也没有权限访问。

### 攻略（待完善）



## Assume ownership

### 要求

- 成为owner

### 漏洞

- 常见错误，构造器名称写错了，导致我们可以直接调用构造器（而通常情况下是不允许调用构造器的）

  ![image-20220811113820841](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220811113820841.png)

- 解决方法：因为有开发者犯下这种问题导致损失严重，所以现在统一要求使用`constructor`作为构造器的关键字了！

### 攻略

- 真没什么说的，直接调用构造器再调用authenticate函数就结束游戏

