---
date: 2022-09-03
layout: post
title: rust入门篇-crate
description: 介绍集合类型
image: 22.jpg
optimized_image: 22.jpg
subtitle:
category: STUDY
tags:
 - Rust
author: aventusc
---

# rust入门篇_crate

## 包和模块

### 总述

Rust 为我们提供了强大的包管理工具：

- **项目（又称包）(Package)**：可以用来构建、测试和分享**单元包(crate)**
- **工作空间(WorkSpace)**：对于大型项目，可以进一步将多个单元包联合在一起，组织成工作空间
- **单元包(Crate)**：一个由多个模块组成的树形结构，可以作为三方库进行分发，也可以生成可执行文件进行运行
- **模块(Module)**：可以一个文件多个模块，也可以一个文件一个模块，模块可以被认为是真实项目中的代码组织单元
- **路径(Path)**:为struct、function或module等项命名的方法

即：

![image-20220905094644133](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905094644133.png)

### Package和Crate

![image-20220905094829637](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905094829637.png)

![image-20220905095102307](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905095102307.png)

- 使用`cargo new crate_name`的格式来创建binary crate
- 使用`cargo new crate_name -lib`的格式来创建library crate

#### 从crate.io安装crate

- 来源：

  [https://crate.io](https://crate.io)

- 方式：

  使用`crate install`的命令来操作

- 注意事项：

  只能安装二进制crate

- 安装位置

  - cargo install安装的二进制存放在根目录的bin文件夹中
  - 如果使用rustup来安装，没有自定义改动，则二进制文件存放在`$HOME/.cargo/bin`下面，前提要确保该目录在环境变量的$PATH中

### Module

![image-20220905095319986](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905095319986.png)

### Path

![image-20220905095554421](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905095554421.png)

同级的路径之间可以相互调用

![image-20220905100945768](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905100945768.png)

![image-20220905100915979](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905100915979.png)

#### use关键字

- 可以使用use关键字来讲路径导入到作用域中

  - 并且这个过程仍然遵循私有性原则（即不可以调用私有的字段）

- 也可以使用use来指定相对路径

- use的习惯用法

  - 函数：将函数的父级模块引入到作用域（指定到父级）

    *即当你使用use来引入函数时，默认应该将该函数及其父级模块都引入进来，以便于代码的可读性，例如：*

    ```rust
    mod creature {
    pub fn animal(){
        println!("this is an animal!")
    	}
    }
    
    use crate::creature;
    pub fn human(){
    	creature::animal();//这一步不止引用animal，还要引用其父级
    }
    ```

- 使用as关键字给引入的变量在本地取一个新的名字

- pub use的使用

  ![image-20220905104642521](C:\Users\huawei\AppData\Roaming\Typora\typora-user-images\image-20220905104642521.png)



## 工作空间（workspace）

当代码数量变多了之后，项目内部的结构就会变得很复杂，因此我们可以通过创建workspace，让代码结构更加清晰。

### 创建方式

在cargo.toml中启用`workspace`作为开头，例如：

```rust
[workspace]

//写出该项目所关联的crate
members = [
    "addr",
    "add_one",
]
```

### 内部结构

- crate.lock

  每一个工作空间里只能有一个crate.lock问价，在工作空间的顶层目录

  由此保证工作空间中的所有crate的依赖版本都是一样的

- crate数量

  可以有多个二进制crate和多个库crate，但二进制crate里面照样只能有一个crate里有main函数作为入口

- target目录

  用于存放一些配置文件，比如doc等，一个workspace只能有一个target目录（否则会出现每个crate执行时都重新生成一遍target，从而浪费时间）

### 依赖

- 在控制台对应的目录下使用`cargo build`来建立将所需要的依赖下载并编译到本地。

- 每个crate只有在下的cargo.toml引用对应的依赖，才能使用该依赖，否则会报错。



参考：杨旭老师的rust教学视频：[https://www.bilibili.com/video/BV1hp4y1k7SV?p=83&spm_id_from=pageDriver&vd_source=9969a9866a80b3b3bf6aa706276b3b07](https://www.bilibili.com/video/BV1hp4y1k7SV?p=83&spm_id_from=pageDriver&vd_source=9969a9866a80b3b3bf6aa706276b3b07)





























