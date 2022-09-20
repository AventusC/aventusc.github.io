---
date: 2022-09-20
layout: post
title: vscode中solidity错误的解决办法
description: 解决vscode中solidity编译和插件问题
image: https://user-images.githubusercontent.com/90261136/191211961-0de6f903-c73e-4d21-9840-7d1258213128.jpg
optimized_image: https://user-images.githubusercontent.com/90261136/191211961-0de6f903-c73e-4d21-9840-7d1258213128.jpg
subtitle:
category: STUDY
tags:
 - contract
 - errors
author: aventusc
---

# vscode中solidity报错的解决方法

## 出现编译器版本不兼容的问题

### 错误展示

![image-20220920160856056](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920160856056.png)

### 解决办法

![image-20220920160928565](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920160928565.png)

点击错误位置，并点击鼠标右键，选择`change workspace compiler version(remote)`这个选项，从而更改本地solidity版本为远程的版本，并且版本可以任意选择，如下图：

![image-20220920161301936](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920161301936.png)

## 编译器版本正确，但代码报错

### 问题展示

如下图所示，代码按照正确语法编写，但是代码仍然报错的情况

![image-20220920161406690](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920161406690.png)

### 解决办法

- 先在remix和chainIDE等IDE上粘贴相同的代码，并测试是否可以编译通过，如果不可以通过，则代表代码确实有问题；如果可以通过则执行下一步

![image-20220920161749652](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920161749652.png)

- 用vscode中编译solidity智能合约的快捷键来尝试编译：在代码页面，按fn模式下的`F5`键来编译，结果如下：

![image-20220920162256707](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920162256707.png)

上图显示编译成功，那么就代表是你安装的其他错误显示插件或者多种语言编译插件出现不兼容的问题，这种情况你就可以不用管它了。

![image-20220920162459490](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920162459490.png)

![image-20220920162407896](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920162407896.png



![image-20220920162444649](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220920162444649.png)

我的代码报错大概就是code runner或者error lens出现问题。

后面有其他的问题再补充，over。





