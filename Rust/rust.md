
特点：**无 GC且无需手动内存管理、性能高、工程性强、语言级安全性**

**编译器检查严格（代码质量下限高）：通过语法检查后，保证了代码的效率 无需再次性能优化**

[Rust权威指南](https://learnku.com/docs/rust-lang/2018)

[Rust 语言圣经](https://course.rs/about-book.html)

# 工程管理

## 工程配置

### 环境搭建

1: viusal C++:VS2019、VS2017

2 VSCode(IDE)

3 rustin [安装 Rust - Rust 程序设计语言 (rust-lang.org)](https://www.rust-lang.org/zh-CN/tools/install)

rustin选默认选项（1），安装后需重启

检查安装：rustc -V

4 cargo:包管理工具 rustin同时安装

### 替换镜像源

-   找到当前用户目录下 **/Users/baoyachi/.cargo/** 的**.cargo** 文件夹
    
-   进入**.cargo** 当前目录，在当前目下创建 **config** 文件
    
-   见下图，打开 **config** 文件，编写以下内容：
    
-   cargo update
    

[source.crates-io]  replace-with = 'tuna' #清华源  #replace-with = 'ustc' #科大源  ​  [source.tuna]  registry = "[https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git](https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git)"  #[source.ustc]  #registry = "git://mirrors.ustc.edu.cn/crates.io-index"

[Rust crates 国内镜像源加速配置 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/126204128?ivk_sa=1024320u)

### VSCode快捷键

右键查看所有引用 包括函数，变量等

#### 代码显示

字体放大/缩小: ctrl + ( + 或 - )

显示/隐藏左侧目录栏 ctrl + b

控制台终端显示与隐藏：ctrl + ~

**切换全屏** : F11

**拆分编辑器** : ctrl + 1/2/3

**折叠代码**： ctrl + k， ctrl + 0-9 (0是完全折叠)

展开代码：ctrl + k, ctrl + j

#### 查找

查找文件：**ctrl + p**

查找代码：**ctri + f**

**显示插件的命令**：ctrl + shift + p

#### 便捷操作

**注释：**

ctrl + k + c

ctri + k +u

**跳转定义**：F12、ctrl + 左击

**返回上次光标**：Alt + ←

关闭所有文件：ctrl + k + w

**打开最近打开的文件：**ctrl + r

**打开新的命令窗：**ctrl + shift + c

## 包和管理

-   **工作空间(WorkSpace)**：组织多个**Package**，形成工作空间
    
-   **Package**：管理create，标志：包含**Cargo.toml**文件
    
-   **Crate**：**Module**的集合，编译单位
    
    有两种creat，一种作为lib的形式，一种做为可执行文件
    
    src/lib.rs lib
    
    src/main.rs、main2.rs 可执行文件
    
-   **模块(Module)**：create内部的代码组织
    
    -   src/mat/mod.rs 把mat打包导出
        
    -   src/mat.rs 单个mat导出
        
    
    mod mat：等价于去找mat.rs 或 mat/mod.rs
    

### package、create、module

.  ├── Cargo.toml  ├── Cargo.lock  ├── src  │ ├── main.rs  │ ├── lib.rs  │ └── bin  │ └── main1.rs  │ └── main2.rs  ├── tests  │ └── some_integration_tests.rs  ├── benches  │ └── simple_bench.rs  └── examples  └── simple_example.rs

-   唯一库包：`src/lib.rs`（**函数导出**）
    
-   模块包：src/math/mod.rs 功能类似lib.rs，是lib.rs的子模块，类似于子文件夹，使得lib的结构更清晰
    
-   默认二进制包：`src/main.rs`，编译后生成的可执行文件与 `Package` 同名（**可执行文件.exe**）
    
-   其余二进制包：`src/bin/main1.rs` 和 `src/bin/main2.rs`，它们会分别生成一个文件同名的二进制可执行文件
    
-   集成测试文件：`tests` 目录下
    
-   基准性能测试 `benchmark` 文件：`benches` 目录下
    
-   项目示例：`examples` 目录下
    

### 引用

#### 路径

**绝对路径**

从creat为起点，向下展开

**相对路径**

creat:忽略当前母目录，从同层目录为起点

super:类似..

self:调用自身模块

#### **use**

声明模块、结构体、函数

优先声明函数、结构体；出现同名或其他冲突时，再声明上一层模块

**同名冲突：**

①加上各自作用域来区分

②使用as区分

use std::fmt::Result;  use std::io::Result as IoResult;

**简化引入**

use std::collections::HashMap;  use std::collections::BTreeMap;  use std::collections::HashSet;  ​  use std::collections::{HashMap,BTreeMap,HashSet};  use std::collections::* //std::collections下全引用

use std::io;  use std::io::Write;  ​  use std::io::{self,write}

### 可见性

**rust中代码默认为私有，父模块无权访问子模块；子模块可以访问父模块**

通过**Pub**来标记模块、函数 提供外部模块调用

**Pub 枚举类型：成员默认可见**

pub enum test{ //test可见  num:u32 //num默认可见  }

**Pub 结构体类型：成员默认不可见**

pub struct test{ //test可见  num:u32 //num默认不可见  pub point:f64 //point可见  }

#### 限制可见性

-   `pub(crate)` 表示在当前包可见
    
-   `pub(self)` 在当前模块可见
    
-   `pub(super)` 在父模块可见
    
-   `pub(in <path>)` 表示在某个路径代表的模块中可见，其中 `path` 必须是父模块或者祖先模块
    

### 第三方库

1.  修改 `Cargo.toml` 文件，在 `[dependencies]` 区域添加一行：`rand = "0.8.3"`
    
2.  此时，如果你用的是 `VSCode` 和 `rust-analyzer` 插件，该插件会自动拉取该库，你可能需要等它完成后，再进行下一步（VSCode 左下角有提示）
    

#### 踩坑

[条件编译 Features - Rust语言圣经(Rust Course)](https://course.rs/cargo/reference/features/intro.html)

optional = true，默认不编译，报错：use of undclear type or create

[dependencies]  gif = { version = "0.11.1", optional = true }

仅当开启feature时，才编译git

[dependencies]  ravif = { version = "0.6.3", optional = true }  rgb = { version = "0.8.25", optional = true }

[features]  avif = ["ravif", "rgb"]

use gif;  fn test(){ let a = gif::funA();}  //找不到

use gif;  #[cfg(feature = "avif")]  fn test(){ let a = gif::funA();}  //找到  //!!!!  But test无法被其他未开启avif的地方找到

为啥要有这种特性 ，以及如何正确使用，留待后续研究

## Cargo

`Cargo` 是包管理工具，可以用于依赖包的下载、编译、更新、分发等，与 `Cargo` 一样有名的还有 [`crates.io`](https://crates.io/)，它是社区提供的包注册中心：用户可以将自己的包发布到该注册中心，然后其它用户通过注册中心引入该包。

### 概述

#### 基础工程目录

.  ├── Cargo.lock  ├── Cargo.toml  ├── src/  │ ├── lib.rs  │ ├── main.rs  │ └── bin/  │ ├── named-executable.rs  │ ├── another-executable.rs  │ └── multi-file-executable/  │ ├── main.rs  │ └── some_module.rs  ├── benches/  │ ├── large-input.rs  │ └── multi-file-bench/  │ ├── main.rs  │ └── bench_module.rs  ├── examples/  │ ├── simple.rs  │ └── multi-file-example/  │ ├── main.rs  │ └── ex_module.rs  └── tests/  ├── some-integration-tests.rs  └── multi-file-test/  ├── main.rs  └── test_module.rs

-   `Cargo.toml` 和 `Cargo.lock` 保存在 `package` 根目录下
    
-   源代码放在 `src` 目录下
    
-   默认的 `lib` 包根是 `src/lib.rs`
    
-   默认的二进制包根是 src/main.rs
    
    -   其它二进制包根放在 `src/bin/` 目录下
        
-   基准测试 benchmark 放在 `benches` 目录下
    
-   示例代码放在 `examples` 目录下
    
-   集成测试代码放在 `tests` 目录下
    

#### 构建开源代码

git clone xxx

cd xxx

cargo build //下载依赖项

#### 添加依赖项

Cargo.toml：配置项目清单（编译器环境、依赖项）

[package]  name = "hello_world"  version = "0.1.0"

[dependencies]  regex = { git = "[https://github.com/rust-lang/regex.git](https://github.com/rust-lang/regex.git)" }  //默认最新版本  regex = { git = "[https://github.com/rust-lang/regex.git](https://github.com/rust-lang/regex.git)", rev = "9f9f693" }  //指定版本

Cargo.lock：依赖项所有准确信息 (自动查找指定依赖项所需要的依赖项)

crates.io： Rust 社区维护的中心化注册服务，用户可以在其中寻找和下载所需的包

在Cargo.toml中添加依赖项，再build

#### 快捷查询

创建工程：cargo new project

运行工程：cargo run （--release）

语法检查：cargo check 快速编译，检查语法，相较于cargo build 减少生成可执行文件的开销

安装依赖项：cargo build（--release）

清理依赖项：cargo clean

更新依赖项：cargo update

更新指定依赖项：cargo update -p regex

### 发布配置

rust中可通过预设置Cargo.toml来配置项目发布属性

[profile.dev]  //debug  opt-level = 0

[profile.release]  //release  opt-level = 3

其中，opt-level是编译优化等级，优化级别越高，编译时间越长

对于其他项目配置属性和默认值，参考[这里](https://rustwiki.org/zh-CN/cargo/reference/manifest.html#the-profile-sections)

### cargo工作空间

项目中除了第三方的包，也会有很多协同开发的包

Cargo workspaces帮助管理这些协同开发的包

#### 创建工作空间

新建项目：cargo new add

在cargo.toml中配置工作空间

[workspace]

members = [  "add-one",  "add-two",  ]

创建包：cargo new add-one; cargo new add-two

├── Cargo.lock  ├── Cargo.toml  ├── add-one  │ ├── Cargo.toml  │ └── src  │ └── lib.rs  ├── add-one  │ ├── Cargo.toml  │ └── src  │ └── lib.rs  ├── src  └── target

目录解析

Cargo.lock：整个项目仅有一个，这意味这所有Cargo.toml的第三方库都存储在这个.lock中，因此这些包引用的库版本必须一致

Cargo.toml：每个包都有各自的Cargo.toml,配置各自的属性

src：存放代码.rs

target：存放编译结果

lib.rs API导出

xx.rs 内置包

main.rs 可执行文件包

#### 添加依赖项

**外部依赖项**

在Caro.toml中配置，注意需要保证全局引用到的版本是一致的

**内部依赖项**

[dependencies]

add-one = { path = "../add-one" }

#### 工作空间测试

add目录下

cargo build：编译所有项目

cargo build -p add-one：仅编译add-one

cargo test：测试所有项目

cargo test -p add-one：仅测试add-one

### cargo高级特性

[条件编译 Features - Rust语言圣经(Rust Course)](https://course.rs/cargo/reference/features/intro.html)

## 自动化测试

### 编写测试

#### 测试函数要求

1.  设置函数所需的数据或状态
    
2.  运行需要测试的代码
    
3.  断言其结果是我们所期望的
    

#### 测试函数示例

#[cfg(test)]  mod tests {  #[test]  fn exploration() {  assert_eq!(2 + 2, 4);  }  }

**#[test]**：表明这是一个测试函数，自动测试时被识别的标记

**assert_eq!** ：断言 2 加 2 等于 4

$ cargo test  Compiling adder v0.1.0 (file:///projects/adder)  Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs  Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test  test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

it_works：测试函数

test result：测试结果

measured：性能测试

Doc-tests：测试文档

#### assert!

入参为一个bool的表达式；ture：正常执行，false：调用panic!

形式1：

assert! (fun())  //fun()返回一个bool

形式2：assert_eq!

assert_eq!(A,B)  //判断A、B是否相等

形式3：assert_ne!

assert_ne!(A,B)  //判断A、B是否不等

#### Result<T, E>

用Result<T, E>来替代assert!，对测试结果细化

#![allow(unused)]  fn main() {  #[cfg(test)]  mod tests {  #[test]  fn it_works() -> Result<(), String> {  if 2 + 2 == 4 {  Ok(())  } else {  Err(String::from("two plus two does not equal four"))  }  }  }  }

### 控制测试如何运行

#### cargo test

识别所有 #[test] 标记的测试函数，并执行测试

#### test-threads

cargo test -- --test-thread=1

cargo test默认并行执行所有的测试函数，如果有多个函数需要对同一个本地文件进行读写，这就会导致发生错误

通过显示指定test-thread来规避这种冲突，但是会增加性能消耗

#### show-output

cargo test -- --show-output

cargo test默认截取运行成功的函数输出

show-output：无论函数执行成功与否，打印所有的输出

#![allow(unused)]  fn main() {  fn prints_and_returns_10(a: i32) -> i32 {  println!("I got the value {}", a);  10  }

#[cfg(test)]  mod tests {  use super::*;

#[test]  
fn this_test_will_pass() {  
 let value = prints_and_returns_10(4);  
 assert_eq!(10, value);  
}  
​  
#[test]  
fn this_test_will_fail() {  
 let value = prints_and_returns_10(8);  
 assert_eq!(5, value);  
}

}  }

#### 指定测试函数

cargo test fun1：指定测试fun1()

cargo test add：指定测试add模块或add相关的函数（add_fun()）

#### 忽略测试函数

#[test] #[ignore]

cargo test默认忽略#[ignore]标记的测试函数

cargo test --ignored：仅运行#[ignore]函数

### 测试的组织结构

单元测试：在与其他部分隔离的环境中测试每一个单元的代码，以便于快速而准确的某个单元的代码功能是否符合预期

集成测试：与其他外部代码一样，通过相同的方式使用你的代码，只测试公有接口而且每个测试都有可能会测试多个模块

#### 单元测试

单元测试与他们要测试的代码共同存放在位于 _src_ 目录下相同的文件中。

因此，需要用#[cfg(test)]来标记 测试模块，使其仅在cargo test时进行编译；cargo run会忽略这个模块

#### 集成测试

新建test目录，与src位于同一层；test下的每个文件都被视作一个crate

rust会识别test目录，cargo run会忽略这个目录

集成测试模拟外部使用环境，来调用内部的API

**crate包的测试**

对于crate包，use mod不会导入main.rs下的函数；

因此，需要将导入函数放在lib.rs下，main.rs中调用lib.rs函数

# 通用编程概念

## 变量

#### 可变声明

let x = 5; //默认为不可变声明，且自动推导类型；也可以显示给定类型

let mut x = 5; //可变声明

#### 常量

const声明，必须显示指定类型

const HEIGHT：u16 = 800;

**与不可变变量的最大区别**:

常量在编译期就要确定其值, 不可以在运行期接受函数返回值

#### 变量遮蔽

let x=4;

let x=x+1; 重新声明变量x，遮蔽前一个x，并将x+1赋值

let space = " "; //字符串类型

let space = space.len(); //unsize类型

#### 未使用变量

let _x = 5; //提示IDE忽略

#### 变量解构

let (a,mut b):(bool,bool) = (true, false);

let (a, b, c, d, e);

(a, b) = (1, 2);  // _ 代表匹配一个值，但是我们不关心具体的值是什么，因此没有使用一个变量名而是使用了 _，并且这个值不会在后续被使用  [c, .., d, _] = [1, 2, 3, 4, 5]; //d=5

## 数据类型

-   数值类型: 有符号整数 (`i8`, `i16`, `i32`, `i64`, `isize`)、 无符号整数 (`u8`, `u16`, `u32`, `u64`, `usize`) 、浮点数 (`f32`, `f64`)、以及有理数、复数
    
-   字符串：字符串字面量和字符串切片 `&str`
    
-   布尔类型： `true`和`false`
    
-   字符类型: 表示单个 Unicode 字符，存储为 4 个字节
    
-   单元类型: 即 `()` ，其唯一的值也是 `()`
    

### Scalar：标量

rust是一中**静态类型**语言，需要在**编译器**知道所有变量的类型

relarse模式下，**数值溢出**不会报错，266被转化为0

默认 i32; f64

// 编译器会进行自动推导，给予twenty i32的类型  let twenty = 20;  // 类型标注  let twenty_one: i32 = 21;  // 通过类型后缀的方式进行类型标注：22是i32类型  let twenty_two = 22i32;

**只有同样类型，才能运算**

let addition = twenty + twenty_one + twenty_two;  println!("{} + {} + {} = {}", twenty, twenty_one, twenty_two, addition);

**对于较长的数字，可以用 _ 进行分割，提升可读性**

let one_million: i64 = 1_000_000;  println!("{}", one_million.pow(2));

**序列**

for i in 'a'..='z'{} //[a, z**]**

for i in 'a'..'z'{} //[a, z**)**

#### 字符类型

Rust使用`Unicode，字符4字节`

#### 单元类型

（）；0字节

mian()、println!()返回的就是()

### Compound：复合

#### 元组

多种类型组合

let tup: (i32, f64, u8) = (500, 6.4, 1);

访问元素：

//模式匹配  let (x,y,z) = tup; println("{}",y)

//.访问 let x = tup.0;

常用作**函数返回值**，返回多个类型值

#### 数组

数据元素被存放在栈上，且数组长度固定

let a = [1, 2, 3, 4, 5];

let a:[i32;5] = [1, 2, 3, 4, 5];

let a:[3;5];  //3 3 3 3 3

常被用做**数组切片**：let b= &s[0..3] = a.as_slice();

## 函数

fn function_test(a:u32,b:char)  -> i32{  语句;  语句；  表达式  }

无需声明，任意位置定义即可

**函数参数显示指定类型**

发散函数：返回值类型 ！;无限循环（暂时无法体会使用场景）

### 语句和表达式

#### **语句**

无返回值、无法赋值

let x = 1;

#### **表达式**

**有返回值**

x+y

let a = 8中的8是表达式、**函数调用**也是表达式、**宏调用**也是表达式

let b = (let a = 8); //let a是语句，无返回值；无法赋值  8是表达式

## 注释

#### 代码注释

同C++

#### 文档注释

行注释：///

块：/** */

文档注释需位于：lib.rs中

cargo doc：将文档注释输出成.html格式，存放在target/doc目录下

cargo doc --open：输出并打开

#### 模块注释

模块最上方

//！

/*! */

#### 文档测试

在文档注释中，写测试案例

cargo test运行

仅需要输出文档、隐藏测试案例时；案测试案例+#

## 流程控制

### for循环

使用方法

等价使用方式

所有权

`for item in collection`

`for item in IntoIterator::into_iter(collection)`

转移所有权

`for item in &collection`

`for item in collection.iter()`

不可变借用

`for item in &mut collection`

`for item in collection.iter_mut()`

可变借用

#### 索引循环 元素循环

// 第一种  let collection = [1, 2, 3, 4, 5];  for i in 0..collection.len() {  let item = collection[i];  // ...  }

// 第二种  for item in collection {

}

第一种方式是循环索引，然后通过索引下标去访问集合，第二种方式是**直接循环集合中的元素**，优劣如下：

**性能**：第一种使用方式中 collection[index] 的索引访问，会因为边界检查(Bounds Checking)导致运行时的性能损耗 —— Rust 会检查并确认 index 是否落在集合内，但是第二种直接迭代的方式就不会触发这种检查，因为编译器会在编译时就完成分析并证明这种访问是合法的 **安全**：第一种方式里对 collection 的索引访问是非连续的，存在一定可能性在两次访问之间，collection 发生了变化，导致脏数据产生。而第二种直接迭代的方式是连续访问，因此不存在这种风险

#### continue

同C++

#### break

直接空返回，或**返回一个值**（类似return）

#### if

rust并不会将非布尔值转化为布尔值

let num =3;  if num{  //num 和 condation（bool）不匹配

}

#### let if

let a = if condition {5} else {6};

# 所有权

## 概述

所有权机制使得rust无需**garbage collector**机制，即可保证**内存安全**

#### 内存释放方式：

-   **垃圾回收机制(GC)**，在程序运行时不断寻找不再使用的内存，典型代表：Java、Go //性能开销大
    
-   **手动管理内存的分配和释放**, 在程序中，通过函数调用的方式来申请和释放内存，典型代表：C++ //复杂繁琐
    
-   **通过所有权来管理内存**，编译器在编译时会根据一系列规则进行检查，典型代表：Rust
    

#### 堆、栈

栈：先进后出，所有**数据大小固定**：存储在CPU高速缓存中，访问速度快

堆：**数据大小可变** ：通过栈上的指针，访问堆区内存，访问速度慢

Rust 中，`main` 线程的[栈大小是 `8MB`](https://course.rs/compiler/pitfalls/stack-overflow.html)，普通线程是 `2MB`

## 所有权规则

1.  Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
    
2.  一个值只能拥有一个所有者
    
3.  当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)，包括其指向的堆区内存
    

所有权一共有三种形式：move、copy、clone

**资源获取即初始化**（_Resource Acquisition Is Initialization (RAII)_

C++中的RAII类似于所有权规则3,当堆区内存的指针离开其作用域后，释放其指向的堆区内存

**在rust中，变量离开作用域时，生命周期结束，rust自动释放其栈区内存，并对于实现drop trait的类型，调用drop释放其堆区内存**

#### 所有权和赋值

-   数据存放在栈上，数据固定大小，适用于**copy**，直接在栈上将数据复制一份，离开作用域后
    
    tips：只有实现copy trait的类型，才能使用copy特性；并且，不存在一个类型即实现copy trait，又实现drop trait
    

{  let a = "hello";  let b = a;  //a、b都有效  }  //a、b都被释放

-   数据存放在堆上，数据大小位置，适用于move、clone
    
    -   **move**:拷贝栈上的数据，a丧失访问堆区空间的权力
        
        {  let a = string::from("hello");  let b= a;  //a的所有权被move到b  println!("{}", a); // error：a已经丢失所有权  }  //仅释放b并drop
        
    -   **clone**：在某些特殊场景，可能会将堆区的数据拷贝一份，放在另一空间
        
        {  let a = string::from("hello");  let b= a.clone();  //a的所有权被move到b
        
        }  //释放a,b并drop
        

#### 所有权和函数

函数在传参时，实参的所有权可能被move、copy、clone

fn main() {  let s = String::from("hello"); // s 进入作用域

takes_ownership(s);             // s 的所有权被move到函数takes_ownership里  
 //s到这里不再有效  
​  
let x = 5;                      // x 进入作用域  
​  
makes_copy(x);                  // x 应该移动函数里，  
 // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但s已经被move，无需再次drop

fn takes_ownership(some_string: String) { // some_string 进入作用域  println!("{}", some_string);  } // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域  println!("{}", some_integer);  } // 这里，some_integer 移出作用域。不会有特殊操作

#### 所有权和返回值

变量在接受函数返回值时，返回值的所有权可能被move、copy、clone

fn main() {  let s1 = gives_ownership(); // gives_ownership 将返回值  // 移给 s1

let s2 = String::from("hello");     // s2 进入作用域  
​  
let s3 = takes_and_gives_back(s2);  // s2 被移动到  
 // takes_and_gives_back 中,   
 // 它也将返回值移给 s3

} //s3 移出作用域并被丢弃。  //s2已经被move,无需在此drop  //s1 移出作用域并被丢弃

fn gives_ownership() -> String { // gives_ownership 将返回值移动给  // 调用它的函数

let some_string = String::from("hello"); // some_string 进入作用域.  
​  
some_string                              // 返回 some_string 并移出给调用的函数

}

// takes_and_gives_back 将传入字符串并返回该值  fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

a_string  // 返回 a_string 并移出给调用的函数

}

tips：返回值的所有权move，可以解决垂悬引用的问题(返回引用，但是局部变量在函数结束后，已经drop，返回的引用无效)

#### 参考

[【译】Rust中的Move、Copy和Clone - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/184907190)

#### 踩坑

**问题描述**

Struct和Vec：Struct的成员储存在栈上，Vec的成员储存在堆区

let a = struct.a； 这是合法的，此时发生的是copy

let b = vec[0]; 这是非法的，此时发生的是Move

原因：

首先，深入了解Vec的实现机制，vec[0]这种通过索引来访问Vec的成员，是因为Vec实现了 **Index trait** 和 **IndexMut trait**

impl<T, I: SliceIndex<[T]>, A: Allocator> Index<I> for Vec<T, A> {  type Output = I::Output;

#[inline]  
fn index(&self, index: I) -> &Self::Output {  
 Index::index(&**self, index)  
}

}

impl<T, I: SliceIndex<[T]>, A: Allocator> IndexMut<I> for Vec<T, A> {  #[inline]  fn index_mut(&mut self, index: I) -> &mut Self::Output {  IndexMut::index_mut(&mut **self, index)  }  }

**vec[0]是一个语法糖**

vec[0] ~ *vec.index[0]

注意，index（）返回的是成员变量的引用，vec[0]还需要对返回的引用解引用，Vec的成员存在堆区，**解引用堆区的数据会发生Move**，这就导致了Vec的所有权陷阱

引用是没有所有权的，不能Move其所有权，因此上述问题的本质是发生了***&**，并且操作的对象发生Move

[Rust Vec类型的常见所有权问题解析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/507503510)

**问题描述**

对vec进行for循环迭代后，vec失去所有权

fn main() {  // i32类型实现了Copy trait, 因此读取后赋值时正常copy  let i32_c = vec![1, 2];  let a = vec![  String::from("hello"),  String::from("world"),  ];

for i in a {
    println!("{}", i);
}
println!("a: {:?}", a[0]);  // 错误。value borrowed here after move

}

原因：

for是语法糖，其本质是调用 into_iter()，into_itet()会Move变量的所有权,Vec会丧失所有权

impl<'a, T, A: Allocator> IntoIterator for &'a Vec<T, A> {  type Item = &'a T;  type IntoIter = slice::Iter<'a, T>;

fn into_iter(self) -> slice::Iter<'a, T> {
    self.iter()
}

}

## 引用和解引用

为了防止变量在传参后，丧失所有权；引入引用和解引用的概念

fn calculate_length(s: &String) -> usize { // s 是对 String 的引用  s.len()  } // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权，  //所有不会调用drop

#### 不可变引用

仅允许使用值（无法修改），**不获取所有权**

let b= &a;

change(&a)  fn change(b:& ){}

#### 可变引用

可修改值，**不获取所有权**

let b = &mut a;

change(&mut a);  fn change(b:& mut){}

在一个作用域中，可变引用最多只有一个

可变引用无法与不可变引用共存，持有不可变引用的地方，不希望值在其他地方被修改

#### 引用的作用域

   let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    // 新编译器中，r1,r2作用域在这里结束

    let r3 = &mut s;
    println!("{}", r3);

新版编译器中，为解决部分可变与不可变引用的冲突，**将引用的作用域缩小到该引用最后一次使用的地方**

**警惕引用传递以及copy**

#### ref

let data = 3;  let ref a = data;  //a：&3  等价于  let a = &data;

#### mut a:&T 与 a:&mut T的区别

[mut a:&T 和a:&mut T的区别 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/189337028)

![image-20220805174938110](file://D:/Note/Rust/image/image-20220805174938110.png?lastModify=1673405731?lastModify=1687513606)

**如果想在一个函数中，对引用的参数进行修改，首先：形式参数的类型要是可变引用，其次传入的实际参数要是一个可变引用，最后实参引用的对象，也要是可变的**

fn main() {  let mut s = String::from("hello");

change(&mut s);

}

fn change(some_string: &mut String) {  some_string.push_str(", world");  }

#### 解引用

let a =10;  let b =&a;  assert!(*b,10);

**隐式解引用**

rust在编译期具有可用性特性(usability feature )，会隐式的脑补*操作符进行解引用

let a = 10;  let b = &a;  println!("{}",b);  //此时,等同于 println!("{}",*b);

#### 悬垂引用

引用对象被释放后，指针仍然被错误使用

fn dangle() -> &String { let s = String::from("hello"); // s 是一个新字符串

&s

} //这里s离开作用域并被丢弃，其内存被释放

个人总结：对于悬垂引用比较常见的两种方法

一种是不返回引用，取而代之的是传入一个可修改引用的参数，用来保存原先的返回值

fn dangle(s:&mut String){ s = String::from("hello");  }

另一种是，返回的时候，将变量的所有权move到接受的变量

fn dangle()  -> String{ let s = String::from("hello");

s	//此时，s的所有权被move

} //s的所有权被move，不会调用drop

let s = dangle();

也可以将所有权clone,但是感觉白费资源，且无意义

#### 参考

[【翻译】Rust中的引用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/120382696)

## 切片(slice)

允许你引用集合中部分连续的元素序列，而不是引用整个集合；本质仍是引用，没有所有权

let s = String::from("hello world");

let hello = &s[0..5];  let world = &s[6..11];

左闭、右开；省略代表端点

#### 字符串字面量

字符串字面量是切片，且为**不可变引用**；类型为 &str

let s = 'Hello, world!u

等效于

let s: &str = "Hello, world!";

# 高级类型

## 结构体

### 结构体与元组

形式类似，结构体内部可以存放不同类型的数据

#### 区别

结构体中的成员变量需要命名，这使得结构体比元组更加灵活，不需要依赖顺序来访问

### 结构体定义

必须为每个成员变量，**显示指定类型**

struct User {  active: bool,  username: String,  email: String,  sign_in_count: u64,  }

#### 初始化

初始化时，每个成员都需要进行初始化

let user1 = User {  email: String::from("[someone@example.com](mailto:someone@example.com)"),  username: String::from("someusername123"),  active: true,  sign_in_count: 1,  };

**缺省初始化**

**简化初始化**：

..user1

let user2= User{  email:String::from("xxxx"),  ..user1  }

Tips:此时，user1中发生move的成员变量，失去所有权

#### 打印结构体

通过#[derive(debug)]为结构体派生debug trait；

然后{：?}、{：#} 来打印结构体字段

#### 元组结构体

结构体有名称，成员无名称

struct Color(i32, i32, i32);  struct Point(i32, i32, i32);

fn main() {  let black = Color(0, 0, 0);  //black是color类型  let origin = Point(0, 0, 0);  //origin是point类型  }

#### 类单元结构体

没有任何成员变量，常用于仅为某类型实现trait，但不需要在类型中存储信息时、

struct AlwaysEqual;

let subject = AlwaysEqual;

### Method（方法）

类似class的成员函数，Rust中的方法与对象（Struct、enum）成对出现;

![image-20220808093002312](file://D:/Note/Rust/image/image-20220808093002312.png?lastModify=1673405731?lastModify=1687513606)

#[derive(Debug)]  struct Rectangle {  width: u32,  height: u32,  }

impl Rectangle {  fn area(&self) -> u32 {  self.width * self.height  }  }

fn main() {  let rect1 = Rectangle { width: 30, height: 50 };

println!(
    "The area of the rectangle is {} square pixels.",
    rect1.area()
);

}

#### **入参：**

方法的第一个参数总是**self**

-   self (等同于 self: **Self**)：获得所有权
    
-   &self（等同于 self: &Self）：不可变引用
    
-   & mut self：可变引用
    

TIPS:此处的self为语法糖的简化写法

#### self & Self

self：当前的实例对象 //对象

Self：特征类型的别名 //类型

#### 关联函数

函数参数不需要传入self；常被用做返回结构体对象

ob.method();  //方法

Rectangle::new  //关联函数

#### 自动解引用

C++中，函数调用方法：

//实例化对象调用函数  Obj.fun()  //对象指针调用函数  obj->fun()  //域名调用  OBK::fun()

Rust中，自动解引用机制会隐式添加 &、&mut、*

Obj.fun()  //统一形式

## 枚举

### 枚举定义

enum PickFace{  Double,  Front,  Back,  }  enum Message {  Quit,  Move { x: i32, y: i32 },  Write(String),  ChangeColor(i32, i32, i32),  }

枚举的成员可以不存放数据，仅用来做分支判断

枚举也可以列举**多种可能的类型**，并存放数据，被用来传参和返回值

### Option<T> 与 Null

enum Option<T> {  Some(T),  None,  }

Rust中没有Null空值的概念，并且将None和T封装进Option,Some<T>与T并不是同一类型

**因此，对于Rust中，任何一个非Option<T>的值，都不可能是空值，可以被拿来放心使用**

除非特意取出Some<T>中的T，否则无需担心返回值引发其他误操作

ps:Some、None就是Option的成员名

[Option in std::option](https://doc.rust-lang.org/std/option/enum.Option.html)

#### copied

Option<&T>,T: Copy 保留Some(&T), &T.copy()->Option<T>

#### Cloned

Option<&T>,T: Clone 保留Some(&T), &T.clone()->Option<T>

pub fn cloned(self)->Option<T>
	T:Clone
{
	match self{
		Some(t:&T) => Some(t.clone())
		None=>None
	}
}

#### map

Option<T>中，默认实现了map、map_or、map_or_else等方法，来对Option<T>快速提取其中的值，并作对应分支处理

pub fn map  .... {  match self {  Some(t) => Some(f(t)),  None => None,  }  }  pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {  match self {  Some(t) => f(t),  None => default,  }  }

[(64条消息) rust中的map_or()和map_or_else()函数详解_liberg的博客-CSDN博客](https://blog.csdn.net/linysuccess/article/details/124873579)

## 集合类型

### vector

同C++:std::vector 本质为动态数组，存放同一类型的元素

**Vec<T>**

#### 创建

let v:Vec<i32> = Vec::new();  //显示给定元素类型

let mut v = Vec:new();  v.push(1);  //插入元素后，自动推导类型

let v = vec![1,2,3];  //利用vec!宏在创建的同时，进行初始化

let v:Vec(i32) = Vec::with_capacity(capacity); //预先知道容量，避免频繁插入数据，动态扩容：造成大量的内存申请和数据拷贝

#### 遍历

使用迭代的方式去遍历数组，比用下标的方式去遍历数组更安全也更高效（每次下标访问都会触发数组边界检查）：

let v = vec![1, 2, 3];  for i in &v {  println!("{}", i);  }

迭代过程中，修改数组

let mut v = vec![1,2,3];  for i in &mut v{  *i -= 10;  }

#### 析构

vector内的元素，与vector共存亡，**vector被析构时，内部元素会被析构**

{  let v =vec![1,2,3];  }  //drop（V） vector被析构时，内部元素会被析构

drop(v)使用场景有限，rust自带drop

v = Vec::new()  //覆盖v，清除原本的元素

v.clear()  //清除v的元素，但是保留capacity

#### 储存不同类型元素

**枚举**

vec<>中存放枚举类型 IpAddr

#[derive(Debug)]  enum IpAddr {  V4(String),  V6(String)  }

let v = vec![  IpAddr::V4("127.0.0.1".to_string()),  IpAddr::V6("::1".to_string())  ];

**特征对象**（略）

相较于枚举，更加复杂，也更加灵活 使用更多

### String

rust中的字符，Unicode类型，每个字符4字节

rust中的字符串，UTF-8编码，一个字符占据1~4个字节；常指 str、string; str不可变，string可变

&str 转 string

string::from("Hello')

”hello“.to_string()

string 转 &str

let s = String::from("hello,world!");

let str = &s;

let str = &s[0..5];

let str = s.as_str();

#### 字符串索引

rust中，string的字符连续，1~4字节都有可能，因此需要先遍历统计字符字节大小，再通过索引来访问字符；效率并非O（1）

不要通过字符串索引来访问字符；同理，切片本质也是通过索引字节的区间来访问，容易带来错误

### HashMap

HashMap<K,V>：KV键值对，通过key来查询value，查询平均复杂度O（1）

Key:同类型 Value:同类型

#### 创建

use std::collections::HashMap;

let mut my_gems = HashMap::new();  //通过new来创建  my_gems.insert("红宝石", 1);  my_gems.insert("蓝宝石", 2);

let mut my_gems = HashMap::with_capacity(capacity);  //预设置容量，开辟空间；避免多次动态扩容  my_gems.insert("红宝石", 1);  my_gems.insert("蓝宝石", 2);

//通过迭代器和clllect来创建  use std::collections::HashMap;

let teams_list = vec![  ("中国队".to_string(), 100),  ("美国队".to_string(), 10),  ("日本队".to_string(), 50),  ];

let teams_map: HashMap<_,_> = teams_list.into_iter().collect();  //into_iter 方法将列表转为迭代器  //接着通过 collect 进行收集  //HashMap<_,_>可自动推导，无法自动推导时，显示给出

println!("{:?}",teams_map)

#### 所有权转移

若Key、Value的类型是copy类型，会被复制进HashMap

若不是copy类型，所有权会被移交给HashMap；

对于所有权移交的情况，可以使用引用&,但是必须保证KV和HashMap的生命周期一致

#### 查找

通过get方法查询

use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);  scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");  let score: Option<&i32> = scores.get(&team_name);

-   `get` 方法返回一个 `Option<&i32>` 类型：当查询不到时，会返回一个 `None`，查询到时返回 `Some(&i32)`
    
-   `&i32` 是对 `HashMap` 中值的借用，如果不使用借用，可能会发生所有权的转移
    

#### 遍历

use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);  scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {  //循环遍历  println!("{}: {}", key, value);  }

#### 更新HashMap中的Value

use std::collections::HashMap;

let mut scores = HashMap::new();  scores.insert("Blue", 10);

// 覆盖已有的值  let old = scores.insert("Blue", 20);

// 查询Yellow对应的值，若不存在则插入新值  let v = scores.entry("Yellow").or_insert(5);  //插入成功返回value，不成功返回none  assert_eq!(*v, 5); // 不存在，插入5

// 查询Yellow对应的值，若不存在则插入新值  let v = scores.entry("Yellow").or_insert(50);  assert_eq!(*v, 5); // 已经存在，因此50没有插入

#### 在已有值的基础上进行更新（略）

另一个常用场景如下：查询某个 `key` 对应的值，若不存在则插入新值，若存在则对已有的值进行更新，例如在文本中统计词语出现的次数：

use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();  // 根据空格来切分字符串(英文单词都是通过空格切分)  for word in text.split_whitespace() {  let count = map.entry(word).or_insert(0);  *count += 1;  }

println!("{:?}", map);

上面代码中，新建一个 `map` 用于保存词语出现的次数，插入一个词语时会进行判断：若之前没有插入过，则使用该词语作 `Key`，插入次数 0 作为 `Value`，若之前插入过则取出之前统计的该词语出现的次数，对其加一。

有两点值得注意：

-   `or_insert` 返回了 `&mut v` 引用，因此可以通过该可变引用直接修改 `map` 中对应的值
    
-   使用 `count` 引用时，需要先进行解引用 `*count`，否则会出现类型不匹配
    

#### 哈希函数

为保证k和v的一 一对应性，需要对k进行比较，用以区分；

因此,**key的类型必须要支持比较**，支持**std::cmp::Eq**（f32、f64就不可以作为key，浮点数无法进行比较）

对于支持比较的复杂类型，需要通过哈希函数对key值进行映射，通过映射值来进行查找、比较

哈希函数必须要解决**哈希冲突**：不同值被映射为同一个映射值

# 错误处理

Rust中将错误进行分类，不可恢复的错误 / 可恢复的错误

Rust为了代码的健壮性，需要在编写程序时，明确指出错误的可能性，并做出相应的处理

对于不可恢复的错误：往往属于BUG，停止程序

对于**可恢复的错误**：向用户报错或提示重试，比如未找到本地文件、输入不合法

## panic！

当程序发生**不可恢复的错误**时，调用panic

-   终止线程（子线程终止不会影响主线程）
    
-   输入提示符**RUST_BACKTRACE=1**，进行**栈展开**（backtrace）：逐步回溯函数调用，找到发生错误的地方
    

fn main() {  let v = vec![1, 2, 3];

v[99];	

}

$ cargo run  Compiling panic v0.1.0 (file:///projects/panic)  Finished dev [unoptimized + debuginfo] target(s) in 0.27s  Running `target/debug/panic`  thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', libcore/slice/mod.rs:2448:10  note: Run with `RUST_BACKTRACE=1` for a backtrace.

## Result

**可恢复的错误**

enum Result<T, E> {  Ok(T),  Err(E),  }

#### 接收Result

**match**

let f = File::open("hello.txt");

let f = match f {
    Ok(file) => file,	//成功返回句柄
    Err(error) => {		//失败直接调用panic!
        panic!("Problem opening the file: {:?}", error)
    },
};

**闭包**

use std::fs::File;  use std::io::ErrorKind;

fn main() {  let f = File::open("hello.txt").unwrap_or_else(|error| {  if error.kind() == ErrorKind::NotFound {  File::create("hello.txt").unwrap_or_else(|error| {  panic!("Problem creating the file: {:?}", error);  })  } else {  panic!("Problem opening the file: {:?}", error);  }  });  }

#### 简化panic!：、

**unwrap**

-   open成功直接返回T
    
-   失败调用panic!
    

**expect**

-   open成功直接返回T
    
-   失败调用panic!、并打印提示信息
    

let f = File::open("hello.txt").unwrap();

let f = File::open("hello.txt").expect("Failed to open hello.txt");

#### ？运算符

let mut f = File::open("hello.txt")?;

等同于

let mut f = match f {  // 打开文件成功，将file句柄赋值给f  Ok(file) => file,  // 打开文件失败，将错误返回(向上传播)  Err(e) => return Err(e),  };

？形式简单，且支持**链式调用**

fn read_username_from_file() -> Result<String, io::Error> {  let mut s = String::new();

File::open("hello.txt")?.read_to_string(&mut s)?;

Ok(s)

}

**？宏和 Option<T>**

调用成功，返回Some<T>;失败返回None

注：？的返回结果，需要一个变量来接受（let v）

pub enum Option<T> {  Some(T),  None  }

fn first(arr: &[i32]) -> Option<&i32> {  let v = arr.get(0)?;  Some(v)  }

# 泛型、trait

## Generices（泛型）

将类型属性进行抽象，并用trait进行限制，以达到**处理重复概念**的目的

泛型的概念与**函数的提取**类似，减少不必要的重复代码

任何需要指定数据类型的地方，都可以使用泛型

#### 类型泛型

fn add<T>(a:T, b:T) -> T {  a + b  }

fn main() {  println!("add i8: {}", add(2i8, 3i8));  println!("add i32: {}", add(20, 30));  println!("add f64: {}", add(1.23, 1.23));  }

T：**泛型参数**，可以有多个 T Y U V

泛型需要配合**特征**使用，例如上例中，add的前提是T类型可以相加

fn add<T: std::ops::Add<Output = T>>(a:T, b:T) -> T

#### const泛型

**针对值的泛型**

数组类型 [i32;2] 与 [i32;3]是不同的类型，可用const对其泛型[T; N]

fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {}

#### 默认泛型参数

同其他默认，给一个默认参数，如果不额外声明，则使用默认；额外声明，使用额外声明的类型

trait Add<RHS=Self> {  type Output;

fn add(self, rhs: RHS) -> Self::Output;

}

#### 泛型的性能

泛型的性能开销接近0成本，编译器会对泛型对应的多个类型，生成各自的代码，但这增加了编译速度和生成文件大小

优：性能无消耗

劣：编译慢，生成文件大

enum Option_i32 {  Some(i32),  None,  }

enum Option_f64 {  Some(f64),  None,  }

fn main() {  let integer = Option_i32::Some(5);  let float = Option_f64::Some(5.0);  }

#### 泛型示例

//结构体  struct HashMap<T,U,V>{  }

//const  fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {}

//trait  trait From<D>{}

//结构体方法  impl<T,U,V> Count for HashMap<T,U,V>{  }

//为结构体实现特征  impl<T,U,V,D> From<D> for HashMap<T,U,V>{}

## 特征（Trait）

特征定义了**一个可以被共享的行为**；具备特征的这些类型都可以来调用特征的方法

### 概述

#### 定义特征

pub trait Summary {  //定义特征  fn summarize(&self) -> String;  //只有方法声明，无需具体实现  }

pub struct Post {  pub title: String, // 标题  pub author: String, // 作者  pub content: String, // 内容  }

impl Summary for Post {  //根据类型，为特征方法单独实现  fn summarize(&self) -> String {  format!("文章{}, 作者是{}", self.title, self.author)  }  }

pub struct Weibo {  //根据类型，为特征方法单独实现  pub username: String,  pub content: String  }

impl Summary for Weibo {  fn summarize(&self) -> String {  format!("{}发表了微博{}", self.username, self.content)  }  }

#### **孤儿规则**

关于特征实现与定义的位置，有一条非常重要的原则：

**如果你想要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` 至少有一个是在当前作用域中定义的！**。

例如我们可以为上面的 `Post` 类型实现标准库中的 `Display` 特征，这是因为 `Post`  **类型**定义在当前的作用域中。

同时，我们也可以在当前包中为 `String` 类型实现 `Summary` 特征，因为 `Summary`**特征** 定义在当前作用域中。

但是你无法在当前作用域中，为 `String` 类型实现 `Display` 特征，因为它们俩都定义在标准库中，其定义所在的位置都不在当前作用域，跟你半毛钱关系都没有，看看就行了。

该规则被称为**孤儿规则**，可以确保其它人编写的代码不会破坏你的代码，也确保了你不会莫名其妙就破坏了风马牛不相及的代码。

如果想要为外部类型实现外部特征，可以利用newType将外部类型包一层，定义在本地，在为本地类型实现外部特征

#### 默认实现

特征定义时，提供默认实现；其他类调用特征时，可以直接调用默认实现，或自行重写

struct Scral{  num:u16,  }  pub trait Summary {  fn summarize(&self) -> String {  String::from("(Read more...)")  }  }

impl Summary for Scral{}  Scaral s;  s.simmarize();

#### 使用特征作为函数参数

之前提到过，特征如果仅仅是用来实现方法，那真的有些大材小用，现在我们来讲下，真正可以让特征大放光彩的地方。

现在，先定义一个函数，使用特征用做函数参数：

pub fn notify(item: &impl Summary) {  println!("Breaking news! {}", item.summarize());  }

`impl Summary`，只能说想出这个类型的人真的是起名鬼才，简直太贴切了，故名思义，它的意思是 `实现了Summary特征` 的 `item` 参数。

你可以使用任何实现了 `Summary` 特征的类型作为该函数的参数，同时在函数体内，还可以调用该特征的方法，例如 `summarize` 方法。具体的说，可以传递 `Post` 或 `Weibo` 的实例来作为参数，而其它类如 `String` 或者 `i32` 的类型则不能用做该函数的参数，因为它们没有实现 `Summary` 特征。

本质上是对泛型和trait bound的语法糖

pub fn notify[T:Summary](T:Summary)(item: &T) {  println!("Breaking news! {}", item.summarize());  }

#### 函数返回中的impl Trait

当函数返回一个相当复杂，不知道如何声明变量来接受时（rust所有变量都要指定类型）,可以返回一个具有trait特征的对象；

当函数返回多个分支时（if retrun a; else return b)，a、b必须为同一个类型；否则需要借助trait对象

fn returns_summarizable() -> impl Summary {  Weibo {  username: String::from("sunface"),  content: String::from(  "m1 max太厉害了，电脑再也不会卡",  )  }  }

#### 通过derive派生特征

特征派生语法：`#[derive(Debug)]`

被derive标记的对象会自动实现对应的默认特征代码（Rust），继承相应的功能

例如 `Debug` 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 `println!("{:?}", s)` 的形式打印该结构体的对象。

再如 `Copy` 特征，它也有一套自动实现的默认代码，当标记到一个类型上时，可以让这个类型自动实现 `Copy` 特征，进而可以调用 `copy` 方法，进行自我复制。

#### 常见特征

a. `Deref`，`DerefMut` ： 可以用来重载 `*`操作符，也可以用来 `method resolution` 以及 `deref coercions`（解引用转换）

b. `Drop`： 用于解构，在一个类型的变量被销毁前执行

c. `Copy` ：如果类型实现了这个`Trait`，则该类型的值会使用拷贝语义替代移动语义，并避免所有权的变更。一个类型实现`Copy`有两个前提条件：1、这个类型不能实现`Drop`；2、这个类型的所有字段都必须为`Copy`的。

d. `Clone`：`Copy`的超集，它也是需要编译器生成`implementations`，编译器会为以下类型自动实现`Clone`：

-   内置实现了`Copy`的类型
    
-   由`Clone`类型组成的`Tuples`
    
-   由`Clone`类型组成的`Arrays`
    

e. `Send`：表明一个类型的值是否可以安全的在线程间传递。

f. `Sync`：表明一个类型的值是否可以安全的在线程间共享。

g. `Sized`：编译器可确定大小的类型

i. `Unsize`：编译器无法确定大小的类型

**AsRef & AsMut**:提供as_ref方法，AsRef:将&T转为&U、AsMut将&T转为 &mut U

impl<T: ?Sized, U: ?Sized> AsRef<U> for &T  where  T: AsRef<U>,  {  fn as_ref(&self) -> &U {  <T as AsRef<U>>::as_ref(*self)  }  }

impl<T: ?Sized, U: ?Sized> AsMut<U> for &mut T  where  T: AsMut<U>,  {  fn as_mut(&mut self) -> &mut U {  (*self).as_mut()  }  }

#### **调用方法需要引入特征**

在一些场景中，使用 `as` 关键字做类型转换会有比较大的限制，因为你想要在类型转换上拥有完全的控制，例如处理转换错误，那么你将需要 `TryInto`：

use std::convert::TryInto;

fn main() {  let a: i32 = 10;  let b: u16 = 100;

let b_ = b.try_into()  .unwrap();

if a < b_ {  println!("Ten is less than one hundred.");  }  }

上面代码中引入了 `std::convert::TryInto` 特征，但是却没有使用它，可能有些同学会为此困惑，主要原因在于**如果你要使用一个特征的方法，那么你需要引入该特征到当前的作用域中**，我们在上面用到了 `try_into` 方法，因此需要引入对应的特征。

但是 Rust 又提供了一个非常便利的办法，即把最常用的标准库中的特征通过 [`std::prelude`](https://course.rs/appendix/prelude.html) 模块提前引入到当前作用域中，其中包括了 `std::convert::TryInto`，你可以尝试删除第一行的代码 `use ...`，看看是否会报错。

### 特征约束(trait bound)

pub fn notify<T: Summary>(item1: &T, item2: &T) {}

通过泛型T来对item的类型进行限制，item1、item2都有Summary特征,可以为不同类型

##### **多重约束**

pub fn notify<T: Summary + Display>(item: &T) {}

约束T同时具有Summary和Display特征，item也能在函数内调用这两个特征方法

##### **where约束**

对复杂约束进行形式上的改进

fn some_function<T, U>(t: &T, u: &U) -> i32  where T: Display + Clone,  U: Clone + Debug  {}

##### **使用特征约束有条件地实现方法或特征**

只有 `T` 同时实现了 `Display + PartialOrd` 的 `Pair<T>` 才可以拥有此方法

impl<T: Display + PartialOrd> Pair<T> {  fn cmp_display(&self) {  if self.x >= self.y {  println!("The largest member is x = {}", self.x);  } else {  println!("The largest member is y = {}", self.y);  }  }  }

### trait对象：dyn trait

当不需要关注具体类型，而是想调用类型实现trait的trait方法时，可以使用trait对象来表示那些不同类型，但是都实现了某种trait

由于trait对象表示了不同类型的对象，但rust在编译期确定变量的内存大小

trait对象形式：**&dyn trait** 或 **Box<dyn trait>**

trait Draw {  fn draw(&self) -> String;  }

impl Draw for u8 {  fn draw(&self) -> String {  format!("u8: {}", *self)  }  }

impl Draw for f64 {  fn draw(&self) -> String {  format!("f64: {}", *self)  }  }

// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>  fn draw1(x: Box<dyn Draw>) {  // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法  x.draw();  }

fn draw2(x: &dyn Draw) {  x.draw();  }

fn main() {  let x = 1.1f64;  // do_something(&x);  let y = 8u8;

// x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
// 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
draw1(Box::new(x));
// 基于 y 的值创建一个 Box<u8> 类型的智能指针
draw1(Box::new(y));
draw2(&x);
draw2(&y);

}

#### 对象安全

只有对象安全的trait，才可以拥有trait对象

对象安全：①trait的方法不能返回Self

②trait方法不能有泛型参数

**trait的方法不能返回Self**

以trait clone举例，如果string实现了clone,那string.clone()会返回string类型；Vec实现了clone，Vec.clone()就会返回clone类型

如果存在 pub components:Vec<Box<dyn clone>>，components[0].clone()返回的是Self，无法确定到底返回什么类型

pub trait Clone {  fn clone(&self) -> Self;  }

**trait方法不能有泛型参数**

对于泛型类型参数来说，当使用特征时其会放入具体的类型参数：此具体类型变成了实现该特征的类型的一部分。而当使用特征对象时其具体类型被抹去了，故而无从得知放入泛型参数类型到底是什么

### 高级特性（使用少）

#### 关联类型

在定义trait中，声明一个关联类型Item，然后在trait内部使用该类型，或用作返回值

相比定义泛型，关联类型可以增加代码的可读性，简化代码

pub trait Iterator {  type Item;

fn next(&mut self) -> Option<Self::Item>;

}

#### 同名方法的调用

对于不同trait，可能具有同名方法，如何来进行区分

**非内联函数**

trait Pilot {  fn fly(&self);  }

trait Wizard {  fn fly(&self);  }

struct Human;

impl Pilot for Human {  fn fly(&self) {  println!("This is your captain speaking.");  }  }

impl Wizard for Human {  fn fly(&self) {  println!("Up!");  }  }

impl Human {  fn fly(&self) {  println!("_waving arms furiously_");  }  }

其中，trait Pilot、trait Wizard、Struct Human都具有fly方法

let human = Human；  human.fly()  //如何确定调用的是那个方法？？？

Pilot::fly(&human); // 调用Pilot特征上的方法
Wizard::fly(&human); // 调用Wizard特征上的方法

human.fly(); // 调用Human类型自身的方法

**内联函数**

内联函数没有&self，如果有两个struct都实现了trait Pilot，Pilot::fly()无法确定类型

struct Human;

impl Pilot for Human {  fn fly() {  println!("im human");  }  }

struct Dog  impl Pilot for Dog {  fn fly() {  println!("im dog");  }  }

Pilot::fly()  //应该调用human::fly还是dog::fly？？？

对于关联函数，需要借助**完全限定语法**

<Pilot as Dog>::fly()

### trait中的继承(supertrait)

trait A：B，A继承了B，那么在A中就可以用到trait B的方法

trait B{  fn b();  }  trait A:B{  fn a(){  b();  }  }

trait A发生了继承，如果要为某个struct实现trait A，那么 需要先为stuct实现trait B

struct C;  impl B for C{  fn b(){};  }

impl A for C{  fn a(){  b();  }  }

# 生命周期

生命周期的本质是为确保引用一直有效：**T outlive 'a**

对象的生命周期要包含引用的生命周期，避免dangling pointer

如果有多个生命周期可以满足给定的生命周期参数，**编译器将会使用最小的那个生命周期**

①生命周期在引用中，时刻存在，但是大部分情况下，编译器会隐式标注，提供语法糖

②生命周期标注并不会改变任何引用的实际作用域，只是辅助编译器以标注的生命周期来进行借用检查

③默认标注语法糖失效时，以手动标注为准

#### 悬垂引用

let r;

{
    let x = 5;
    r = &x;
}

println!("r: {}", r);

x变量在离开作用域{}后，被销毁；println!时，r想调用这块已经被销毁的空间

### 常见生命周期

#### 函数

fn main() {  let string1 = String::from("abcd");  let string2 = "xyz";

let result = longest(string1.as_str(), string2);
println!("The longest string is {}", result);

}

fn longest(x: &str, y: &str) -> &str {  if x.len() > y.len() {  x  } else {  y  }  }

借用检查期在编译期，无法确定返回值是x，还是y；也就无法确定返回值的生命周期，进行Borrow checker

#### 结构体

如果结构体的字段是引用的话，需要对其进行标注

标注的意义：确保字段的生命周期会大于结构体的生命周期，否则出现悬垂引用

struct ImportantExcerpt<'a> {  part: &'a str,  }

#### 方法

为带有引用字段的结构体，impl方法时，大部分情况下，依赖隐式标注；极少数情况，需要手动标注

struct ImportantExcerpt<'a> {  part: &'a str,  }

impl<'a> ImportantExcerpt<'a> {  fn level(&self) -> i32 {  3  }  }

#### 静态生命周期

`'static`: 其生命周期能够存活于整个程序期间，大力出奇迹，慎用！！！

标注不改变作用域，因此该释放时自己会释放

let s: &'static str = "I have a static lifetime.";

### 生命周期标注

生命周期的标注并不会改变任何引用的生命周期长短，该释放的时候仍然释放

**只是辅助编译器以标注的生命周期来进行借用检查**

对例子进行标注：约束x、y、返回值的生命周期一样长

它的实际含义是 **`longest` 函数返回的引用的生命周期与传入该函数的引用的生命周期的较小者一致**

fn main() {  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {  if x.len() > y.len() {  x  } else {  y  }  }

let string1 = String::from("long string is long");  {  let string2 = String::from("xyz");  let result = longest(string1.as_str(), string2.as_str());  println!("The longest string is {}", result);  }

let string1 = String::from("long string is long");  let result;  {  let string2 = String::from("xyz");  result = longest(string1.as_str(), string2.as_str());  }  println!("The longest string is {}", result);

编译器按照我们的标注理解，返回的引用生命周期为两个参数中较小着；返回值的生命周期与string2一致

但是，result的生命周期>string2，因此会报错

### 生命周期消除

为简化生命周期的标注，编译器会隐式增加标注，无需显示指出

**如果在隐式标注规则下，编译器无法推导出确定的生命周期，那么需要手动标注来明确**

**隐式标注规则：**

规则一：**每一个是引用的参数都有它自己的生命周期参数**

fn foo(x: &i32)  隐式转换 fn foo<'a>(x: &'a i32)  fn foo(x: &i32, y: &i32)  隐式转换  fn foo<'a, 'b>(x: &'a i32, y: &'b i32)

规则二：**如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数**

规则一：fn foo(x: &i32) ->(&i32,&i32)  隐式转换 fn foo<'a>(x: &'a i32)  ->  (&i32,&i32)  规则二：fn foo<'a>(x: &'a i32)  ->  (&i32,&i32)  隐式转换 fn foo<'a>(x: &'a i32)  ->  (&'a i32,&'a i32)

规则三：如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明是个对象的方法(method)

那么**所有输出生命周期参数被赋予 `self` 的生命周期**

### 进阶

#### 无界生命周期

对裸指针解引用后，再取引用，导致引用的数据凭空出现，

fn f<'a, T>(x: _const T) -> &'a T {  unsafe {  &_x  }  }

#### 生命周期约束 HRTB

**’a:'b**

a outlive b

**T:‘a**

T outlive a //在rust1.31中，T：’a被省略

# 函数式编程

-   使用函数作为参数
    
-   使用函数作为返回值
    
-   将函数赋值给变量
    

### 闭包（Closure）

闭包类似lamda，是一种匿名函数，允许**捕获**调用者作用域范围内的变量

并且，**它可以赋值给变量也可以作为参数传递给其它函数**

|param1, param2,...| {  语句1;  语句2;  返回表达式  }

//仅有表达式时，简化形式  |param1| 返回表达式

#### 简化代码

**case1**

Closure1：函数实现  Closure2: 函数实现

调用函数Closure1  //当改用Closure2时，需要手动更改多处调用  {  Closure1（）  ......  Closure1（）  }

//简化 let action = Closure1;  =》 let action = Closure2;  //完成改动  {  action（）  ......  action（）  }

**case2**

Closure1：  函数  Closure2: 函数  let action = Closure1;

test(x);  {  action(x+1);  //当将传入参数改为 2x  此时，需要引用闭包  }

test(x);  {  let actuon =  ||{  /让action可以捕捉到x，当传入参数改变时，仅改变 action即可  语句...；  语句...；  2*x  }  }

#### 闭包的类型推导

显示标注

let sum = |x: i32, y: i32| -> i32 {  x + y  }

隐式推导

let sum = |x, y| x + y;  let v = sum(1, 2);

虽然类型推导很好用，但是它不是泛型，**当编译器推导出一种类型后，它就会一直使用该类型**：

let example_closure = |x| x;

let s = example_closure(String::from("hello"));  //推导出x为string  let n = example_closure(5);  //不会再次推导，将5当成string传入

#### 结构体中的闭包

struct Cacher<T>  where  T: Fn(u32) -> u32,  //闭包 trait  {  query: T,  value: Option<u32>,  }

**T: Fn(u32) -> u32**

-   Fn(u32) -> u32：这是一个trait，表现为实现了一个闭包，**该闭包拥有一个`u32`类型的参数，同时返回一个`u32`类型的值**
    
-   T：为实现闭包特性的泛型
    

#### 捕获作用域中的值

fn main() {  let x = 4;

let equal_to_x = |z| z == x;	//x没有被传入闭包中，被捕获

let y = 4;

assert!(equal_to_x(y));

}

#### 捕获变量途径

当闭包从环境中捕获一个值时，会分配内存去存储这些值

闭包捕获变量有三种途径，恰好对应函数参数的三种传入方式：转移所有权、可变借用、不可变借用

**FnOnce**

该类型的闭包会拿走被捕获变量的所有权。`Once` 顾名思义，说明该闭包只能运行一次

F: FnOnce(usize) -> bool

**FnMut**

它以可变借用的方式捕获了环境中的值，因此可以修改该值

let mut s = "hello, ".to_string();

let mut update_string = |str| s.push_str(str);

**Fn**

以不可变借用的方式捕获环境中的值

let s = "hello, ".to_string();

let update_string = |str| println!("{},{}",s,str);

#### 闭包作为函数返回值

略

#### 踩坑

闭包**捕获环境变量**，是发生在构建闭包时，而不是闭包调用

    let r3 = &mut s;

    let mut test = || {
        r3.push('1');	//first引用
    };

    let r1 = r3;	//scend引用	//error：r3作为可变引用，闭包和r1使得其作用域重叠，同时存在两个可变引用
    test();	

//提前test()的调用
    let r3 = &mut s;

    let mut test = || {
        r3.push('1');	//first引用
    };

  	test();			//此时，闭包中的引用作用域已结束
    let r1 = r3;	//scend引用	
  

//对于可变引用，不采用捕获的方式，而是作为参数传递进去，此时该引用的作用域，就从调用开始
     let mut s = String::from("hello");

    let mut test = |r: &mut String| {
        r.push('1');
    };

    let r1 = &mut s; //first引用
    let r3 = &mut s;
    test(r3); //scend引用

### 迭代器（Iterator）

迭代器允许我们迭代一个连续的集合，例如数组、动态数组 `Vec`、`HashMap` 等

**在此过程中，只需关心集合中的元素如何处理，而无需关心如何开始、如何结束、按照什么样的索引去访问等问题**

**迭代器**（_iterator_）负责遍历序列中的每一项和决定序列何时结束的逻辑

#### for循环与迭代器

C++风格的索引循环：有明确的索引起始点和终止点，通过索引来访问arr[i]

int arr = [1, 2, 3];  for (int i = 0; i < arr.length（）; ++i) {  auoto temp = arr[i];  }

Rust风格的迭代循环：没有使用索引，把 `arr` 数组当成一个迭代器，直接去遍历其中的元素，从哪里开始，从哪里结束，都无需操心

let arr = [1, 2, 3];  for v in arr {  println!("{}",v);  }

Rust风格的迭代循环，本质上由于数组实现了**IntoIterator特征**

Rust通过for**语法糖**，自动把实现了该特征的数组类型转换为迭代器，最终让我们可以直接对一个数组进行迭代

let arr = [1, 2, 3];  for v in arr.into_iter() {  //显示调用  for v in arr 隐式调用  println!("{}", v);  }

#### Iterator特征

Iterator：迭代器特征，实现该特征本身就是迭代器类型，可以调用next方法

next方法用来从集合中取值，next方法是消耗性的，每次消耗一个元素，最终迭代器中没有任何元素，只能返回none

Item：trait Iterator的关联类型，实现Iterator必须要指定Item类型，并且在trait Iteror中，Item是next方法的返回值类型

pub trait Iterator {  type Item;

fn next(&mut self) -> Option<Self::Item>;

// 省略其余有默认实现的方法

}

for循环 脱糖版

let values = vec![1, 2, 3];

{
    let result = match values.into_iter(){		//into_iter（）数组转换为迭代器
        mut iter => loop {
            match iter.next() {
                Some(x) => { println!("{}", x); },
                None => break,
            }
        },
    };
    result
}

#### IntoIterator 特征

`IntoIterator` ：强调某一个类型如果实现了该特征，它可以通过 `into_iter()`，`iter()`，iter_mut() 等方法变成一个**迭代器**，再来调用next方法

impl<I: Iterator> IntoIterator for I {  type Item = I::Item;  type IntoIter = I;

#[inline]
fn into_iter(self) -> I {
    self
}

}

-   `into_iter` 会夺走所有权
    
-   `iter` 是借用
    
-   `iter_mut` 是可变借用
    

#### 惰性初始化

let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

迭代器是惰性的，创建v1_iter后，此时不会发生任何迭代行为，知道迭代器被使用

#### 消费者与适配器

##### 消费者适配器（非惰性）

定义在 trait iterator中的一种方法，**内部调用next()**来对迭代器进行操作，**返回数值**

let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

let total: i32 = v1_iter.sum();	//sum就是消费者适配器，非惰性，会被执行，total会有值

##### collect

将迭代器中元素，收集到容器中，需要指定容器类型（Vec<>、HashMap<>....）

常对迭代器适配器返回的迭代器进行数据转换

##### 迭代器适配器（惰性）

定义类似消费者适配器，区别在于迭代器适配器是惰性的，**返回一个迭代器**

let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);

迭代器适配器是惰性的，因此上述代码并不会被执行，更接近与创建了一个新的迭代器

需要对新的迭代器调用消费者适配器来返回一个值，往往采用collect方法

let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

**常见迭代器适配器**

map：对迭代器元素进行映射，改变其值

v1.iter().map(|x|x+1).collect()  //v1.iter中的元素+1

filter：对迭代器元素进行过滤，若表达式返回ture，则过滤当前元素

v1.iter().filter(|x|x=1).collect()  //v1.iter中=1的元素

zip：将两个迭代器合并为一个新的迭代器，新的迭代器中的元素为元组（x,y）

let data:HashMap<> = v1.iter().zip(v1.iter()).collect()

enumerate:返回元组（index,value）value:原迭代器的元素，index：索引

let data:HashMap<> = v.iter().enumerate().collect()

filter_map:返回some的会留下来并解构匹配，返回none的会被扔掉

WHY（**迭代器适配器优点**）

通过传入闭包，调用处直接实现对迭代器中元素的处理，并可以捕获环境值

#### 迭代器性能

**In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.**

迭代器是 Rust 的 **零成本抽象**（zero-cost abstractions）之一，意味着创建时不会引入运行时的开销；使用迭代器才会引入运行开销，but!!!需要使用时，你也无法写出性能更优的代码

# 智能指针

Vec、String、Hash等也是智能指针，具体数据存放在堆区，指针存放在栈去，在避免悬垂引用时，直接Move Vec时，只是传递一个结构体

智能指针往往是基于结构体实现，它与我们自定义的结构体最大的区别在于它实现了 `Deref` 和 `Drop` 特征：

-   `Deref` 可以让智能指针像引用那样工作，这样你就可以写出同时支持智能指针和引用的代码，例如 `*T`
    
-   `Drop` 允许你指定智能指针超出作用域后自动执行的代码，例如做一些数据清除等收尾工作
    

常用智能指针

-   `Box<T>`，可以将值分配到堆上
    
-   `Rc<T>`，引用计数类型，允许多所有权存在; 但是仅允许在编译时执行**不可变借用检查**
    
-   `Ref<T>` 和 `RefMut<T>`，允许将借用规则检查从编译期移动到运行期进行
    

### Box<T>堆对象分配

#### 使用场景

-   特意的将数据分配在堆上
    
-   数据较大时，又不想在转移所有权时进行数据拷贝
    
-   类型的大小在编译期无法确定，但是我们又需要固定大小的类型时
    
-   特征对象，用于说明对象实现了一个特征，利用trait的方法，而不是关注其具体的类型
    

**特意的将数据分配在堆上**

let a = Box::new(3);
println!("a = {}", a); // a = 3

// 下面一行代码将报错
// let b = a + 1; // cannot add `{integer}` to `Box<{integer}>`

-   println！访问a时，**Deref特征**进行了隐式解引用 a ~ (*a)
    
-   let b = a + 1; 表达式中，Deref特征无法隐式解引用
    
-   a持有的智能指针会在作用域结束时，被自动释放掉，因为Box<T>实现了**Drop特征**
    

**避免栈上的浅拷贝**

避免大块数据存放在栈上，copy时，性能开销大

应将其存放在堆上，栈上存放一个变量，保存地址、大小、容量等信息；copy时，堆区内存不变，仅拷贝栈上的变量，开销小

// 在栈上创建一个长度为1000的数组
let arr = [0;1000];
// 将arr所有权转移arr1，由于 `arr` 分配在栈上，copy一份数据
let arr1 = arr;

// arr 和 arr1 都拥有各自的栈上数组，因此不会报错
println!("{:?}", arr.len());
println!("{:?}", arr1.len());

// 在堆上创建一个长度为1000的数组，然后使用一个智能指针指向它
let arr = Box::new([0;1000]);
// 将堆上数组的所有权转移给 arr1，由于数据在堆上，因此仅仅拷贝了智能指针的结构体，堆区并没有被拷贝
// 所有权顺利转移给 arr1，arr 不再拥有所有权
let arr1 = arr;
println!("{:?}", arr1.len());
// 由于 arr 不再拥有底层数组的所有权，因此下面代码将报错
// println!("{:?}", arr.len());

**将 DST 转化为 Sized 固定大小**

枚举类型，内部递归；无法确定内存大小

enum List {  Cons(i32, List),  Nil,  }

将无法确定的部分，存放在堆上，栈上仅存放指针，枚举类型大小确定

enum List {  Cons(i32, Box<List>),  Nil,  }

**特征对象**

dyn trait的大小不确定，只能用&dyn trait、Box<dyn trait>的方式，

#### Box内存布局

Vec<Box<i32>>

![image-20220809110144918](file://D:/Note/Rust/image/image-20220809110144918.png?lastModify=1673405731?lastModify=1687513606)

let arr = vec![Box::new(1), Box::new(2)];  let (first, second) = (&arr[0], &arr[1]);  let sum = **first +** second;

tips：let first = &arr[0],而不是 let first = arr[0],

因为此时发生的是 let first = *arr.index(0)，Box没有实现copy trait，此时发生Move，但是&arr.index(0)返回的是一个没有所有权的引用，因此会发生错误

表达式无法解引用， **frist = *Box<x>=x

#### Box::leak

它可以消费掉 `Box` 并且强制目标值从内存中泄漏

略

### Deref特征

Rust可以为智能指针结构体实现Deref特征，实现**对结构体进行解引用**

#### 自定义智能指针

Struct MyBox<T>{  pub value:T,  };

impl<T> MyBox<T>{  fn new（value:T）  ->  MyBox<T>{  MyBox{  vaule  }  }  }

**实现Deref特征**

use std::ops::Deref;

impl<T> Deref for MyBox<T>{  type Target = T;  fn deref(&self)  -> &Self::Target{  &self.value  }  }

在实现Deref triat之前，*my_box无法被编译器识别

在实现Deref trait之后：*my_box = *（my_box.deref()）= *(&self.value)

ps：deref返回引用，再*解引用的原因在于，不想将value的所有权Move，否则可能出现 解引用一次后，数据就丧失所有权，无法再用

#### Deref隐式转换

当一个struct实现了deref trait时，如果**将其引用传递进方法**后，会根据参数签名，自动解引用

fn main() {  let s = Box::new(String::from("hello world"));  display(&s)  }

fn display(s: &str) {  println!("{}",s);  }

并且，这种解引用是**链式**的，例如&s是&Box<String>，先调用一次deref，&s变成 &String；再调用一次deref，&s变成&str，完成匹配

-   当 `T: Deref<Target=U>`，可以将 `&T` 转换成 `&U`
    
-   当 `T: DerefMut<Target=U>`，可以将 `&mut T` 转换成 `&mut U`
    
-   当 `T: Deref<Target=U>`，可以将 `&mut T` 转换成 `&U`
    

### Drop特征

rust中的变量在脱离其作用域后，调用drop方法，free~~

#### Drop的顺序

-   变量级别：先进后出
    
-   结构体字段：先进先出
    

struct HasDrop1;  struct HasDrop2;  impl Drop for HasDrop1 {  fn drop(&mut self) {  println!("Dropping HasDrop1!");  }  }  impl Drop for HasDrop2 {  fn drop(&mut self) {  println!("Dropping HasDrop2!");  }  }  struct HasTwoDrops {  one: HasDrop1,  two: HasDrop2,  }  impl Drop for HasTwoDrops {  fn drop(&mut self) {  println!("Dropping HasTwoDrops!");  }  }

struct Foo;

impl Drop for Foo {  fn drop(&mut self) {  println!("Dropping Foo!")  }  }

fn main() {  let _x = HasTwoDrops {  two: HasDrop2,  one: HasDrop1,  };  let _foo = Foo;  println!("Running!");  }

Running!  Dropping Foo!  Dropping HasTwoDrops!  Dropping HasDrop1!  Dropping HasDrop2!

#### 自动派生drop

实际上，rust会为所有的非Copy类型都实现drop trait

ps：废话....

#### Drop的执行顺序

-   **变量级别，按照逆序的方式**，`c` 在 d之前创建，因此 c 在 `d` 之后被 `drop`
    
-   **结构体内部，按照顺序的方式**，结构体 中的字段按照定义中的顺序依次 `drop`
    

#### 手动回收

rust是禁止主动调用 drop trait的特征的，因为这可能导致你释放了一个内存，但是其他地方仍要使用他

可以调用std::mem::drop

mem::drop与drop特征不同，只会move变量的所有权，无法对变量再操作；但是，变量的释放，仍会在离开作用域后，被隐式调用drop特征

foo.drop()  //drop trait

drop(foo)  //std::mem::drop

tips：rust是如何保证drop的时机，不会清理仍在使用的值

rust的所有权机制，会保障引用总是有效的

#### Drop 与 Copy互斥

无法实现Copy和Drop特征共存，因为Copy类型可以复制，rust无法判断变量的释放时机

### Rc与Arc实现1 v N所有权机制

`Rc` 和 `Arc`，前者适用于单线程，后者适用于多线程

通过**引用计数**的方式，允许一个数据资源在同一时刻拥有多个所有者。

#### Rc<T>

使用引用计数（reference counting），记录一个数据被引用的次数，次数为0时，释放该数据

use std::rc::Rc;  fn main() {  let a = Rc::new(String::from("test ref counting"));  //初始化，引用计数+1  println!("count after creating a = {}", Rc::strong_count(&a));  let b = Rc::clone(&a);  //clone 引用计数+1  println!("count after creating b = {}", Rc::strong_count(&a));  {  let c = Rc::clone(&a);  //clone 引用计数+1  println!("count after creating c = {}", Rc::strong_count(&c));  }  //drop(c) 引用计数-1  println!("count after c goes out of scope = {}", Rc::strong_count(&a));  }

**Rc::new**

new一个Rc<T>，其中包括引用计数、以及指向T的指针

**Rc::clone**

这里的clone并不是深拷贝； **仅仅复制了智能指针并增加了引用计数，并没有克隆底层数据**，因此 `a` 和 `b` 是共享了底层的字符串 `s`

**不可变引用**

Rc<T>是指向底层的**不可变引用**，无法通过其来修改数据

需要配合互斥锁 Mutex<T> 和 内部可变性的 `RefCell<T>` 类型来辅助修改数据

**无法多线程**

Rc<T>没有实现多线程的Send特征

#### Arc<T>

Arc接口与Rc类似，同样是**不可变引用**

但是Arc可用于**多线程**，这也会增加**Arc的性能开销**

所以，单线程下用Rc，多线程下用Arc

### Cell、RefCell & 内部可变性

#### 内部可变性

rust无法对一个不可变的值进行可变引用

let a = 5;  let b =&mut a;

内部可变性可以利用**内部提供的方法**，来对一个不可变的变量或引用进行修改

Struct test{  temp1:i32  ...  temp100:i32  }

//struct的可变性 是整体打包的  let mut a = test{};  //所有字段都可变  let b =test{};  //所有字段都不可变

事实是，我们仅想要改变其中的某个字段

Struct test{  mut_vaule:Cell<i32>,  //Cell  temp1:i32  ...  temp100:i32  }  let a = test{};  a.value.set(2);  //Cell提供内部set的方法

#### Cell<T>

use std::cell::Cell;

let c = Cell::new("asdf");  let one = c.get();  c.set("qwer");  let two = c.get();  println!("{},{}", one, two);

c是**不可变借用**，但set能对c重新赋值

Cell和RefCell区别是，Cell底层set的数据对象要是可Copy的

Cell<T> where T:Copy

tips：Cell、RefCell都只能用在单线程下，否则可能出现date_race

#### RefCell<T>

use std::cell::RefCell;

fn main() {  let s = RefCell::new(String::from("hello, world"));  let s1 = s.borrow();  let s2 = s.borrow_mut();

println!("{},{}", s1, s2);

}

RefCell可以通过内部方法，返回可变或不可变引用，通过不可变引用来实现内部可变性

RefCell可以绕过编译期检查的机制

这是由于RefCell与Cell相比，多了字段borrow，用来记录可变或不可变引用

pub struct Cell<T: ?Sized> {  value: UnsafeCell<T>,  }

pub struct RefCell<T: ?Sized> {  borrow: Cell<BorrowFlag>,  // Stores the location of the earliest currently active borrow.  // This gets updated whenever we go from having zero borrows  // to having a single borrow. When a borrow occurs, this gets included  // in the generated `BorrowError/`BorrowMutError`  #[cfg(feature = "debug_refcell")]  borrowed_at: Cell<Option<&'static crate::panic::Location<'static>>>,  value: UnsafeCell<T>,  }

如果需要申请一个可变引用时，发现已经存在不可变引用或可变引用 panic!

如果需要申请一个不可变引用时，存在可变引用 panic!

**RefCell只是让代码渡过编译器的检查，在运行阶段，可变引用和不可变引用的共存会导致panic**

**RefCell的意义**

①Rust的编译期检查有时过于严格，正确的程序也可能被判错

②对于内部可变性，如果类型不支持Copy，那么需要RefCell

③对于复杂场景，可以借助RefCell的Panic信息来帮助处理

#### Rc + RefCell

Rc实现一份数据拥有多个所有者，但是都是不可变引用；RefCell提供不可变引用的内部可变性

Rc<RefCell<T>>可以使得一个数据被多个持有，并通过修改某一处来更新到全局

use std::cell::RefCell;  use std::rc::Rc;  fn main() {  let s = Rc::new(RefCell::new("我很善变，还拥有多个主人".to_string()));

let s1 = s.clone();
let s2 = s.clone();
// let mut s2 = s.borrow_mut();
s2.borrow_mut().push_str(", on yeah!");

println!("{:?}\n{:?}\n{:?}", s, s1, s2);

}

[Rust源码阅读： Cell/RefCell与内部可变性 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/384388192)

#### 踩坑

struct Data;

struct Node{  pub inner:Rc<RcfCell<Data>>  }  let node =Node{  inner:Rc::new(RcfCell::new(Data{}))  };

//不可变借用  let ptr = node.inner.borrow();  //ptr:Ref<Data>  //可变借用  let ptr = node.inner.borrow_mut();  //ptr:RefMut<Data>

//重头戏！！！：如何拿到Data的数据  let data = *node.inner.borrow();  //data:Data  //error[E0507]: cannot move out of dereference of Ref＜‘_, Node＞  问题的实质是 *&没有所有权，borrow()拿到的引用，解引用出来的数据是没有所有权的  let data 进行了赋值操作，data的所有权要么move，要么copy，要么clone  *&没有所有权，首先排除move  要么 let data = node.inner.borrow().clone()；  //进行clone  要么 let data = *node.inner.borrow();  //Data要求实现copy

Data的结构往往较为复杂，不会派生copy；并且直接clone整个Data浪费资源  往往采取下面的方式：  let ptr = &*(node.inner).borrow();  //ptr：&Data  用ptr去拿到引用，在通过ptr访问Data下的数据，根据需要clone Data的成员

#### Mutex<T>

Cell、RefCell都没有实现Send trait，无法支持多线程

Mutex+Arc可以实现多线程的内部可变性

use std::sync::{Mutex, Arc};  use std::thread;

fn main() {  let counter = Arc::new(Mutex::new(0));  //不可变引用  let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();	//已可变引用的方式，接受不可变引用，并改变

        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());

}

## weak & 循环引用

**List Hell**

1.  在创建了 `a` 后，紧接着就使用 `a` 创建了 `b`，因此 `b` 引用了 `a`
    
2.  然后我们又利用 `Rc` 克隆了 `b`，然后通过 `RefCell` 的可变性，让 `a` 引用了 `b`
    

在 `main` 函数结束前，`a` 和 `b` 的引用计数均是 `2`，随后 `b` 触发 `Drop`，此时引用计数会变为 `1`，并不会归 `0`，因此 `b` 所指向内存不会被释放，同理可得 `a` 指向的内存也不会被释放，最终发生了内存泄漏。

use crate::List::{Cons, Nil};  use std::cell::RefCell;  use std::rc::Rc;

#[derive(Debug)]  enum List {  Cons(i32, RefCell<Rc<List>>),  Nil,  }

impl List {  fn tail(&self) -> Option<&RefCell<Rc<List>>> {  match self {  Cons(_, item) => Some(item),  Nil => None,  }  }  }

fn main() {  let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

println!("a的初始化rc计数 = {}", Rc::strong_count(&a));
println!("a指向的节点 = {:?}", a.tail());

// 创建`b`到`a`的引用
let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

println!("在b创建后，a的rc计数 = {}", Rc::strong_count(&a));
println!("b的初始化rc计数 = {}", Rc::strong_count(&b));
println!("b指向的节点 = {:?}", b.tail());

// 利用RefCell的可变性，创建了`a`到`b`的引用
if let Some(link) = a.tail() {
    *link.borrow_mut() = Rc::clone(&b);
}

println!("在更改a后，b的rc计数 = {}", Rc::strong_count(&b));
println!("在更改a后，a的rc计数 = {}", Rc::strong_count(&a));

// 下面一行println!将导致循环引用
// 我们可怜的8MB大小的main线程栈空间将被它冲垮，最终造成栈溢出
// println!("a next item = {:?}", a.tail());

}

#### weak

`weak` 非常类似于 `Rc`，但是与 `Rc` 持有所有权不同，`Weak` 不持有所有权

weak是一种弱引用，没有所有权，也不增加引用计数，无法阻止drop;

仅保存一份指向数据的弱引用：返回**Option<Rc<T>>** ，如果引用关系存在，就返回Some；如果不存，就返回None

#### Weak 与 Rc 对比

`Weak`

`Rc`

不计数

引用计数

不拥有所有权

拥有值的所有权

不阻止值被释放(drop)

所有权计数归零，才能 drop

引用的值存在返回 `Some`，不存在返回 `None`

引用的值必定存在

通过 `upgrade` 取到 `Option<Rc<T>>`，然后再取值

通过 `Deref` 自动解引用，取值无需任何操作

-   可访问，但没有所有权，不增加引用计数，因此不会影响被引用值的释放回收
    
-   可由 `Rc<T>` 调用 `downgrade` 方法转换成 `Weak<T>`
    
-   `Weak<T>` 可使用 `upgrade` 方法转换成 `Option<Rc<T>>`，如果资源已经被释放，则 `Option` 的值是 `None`
    
-   常用于解决循环引用的问题
    

#### Weak解决循环引用

对于父子引用关系，可以让父节点通过 `Rc` 来引用子节点，然后让子节点通过 `Weak` 来引用父节点

use std::rc::Rc;  use std::rc::Weak;  use std::cell::RefCell;

// 主人  struct Owner {  name: String,  gadgets: RefCell<Vec<Weak<Gadget>>>,  }

// 工具  struct Gadget {  id: i32,  owner: Rc<Owner>,  }

fn main() {  // 创建一个 Owner  // 需要注意，该 Owner 也拥有多个 `gadgets`  let gadget_owner : Rc<Owner> = Rc::new(  Owner {  name: "Gadget Man".to_string(),  gadgets: RefCell::new(Vec::new()),  }  );

// 创建工具，同时与主人进行关联：创建两个 gadget，他们分别持有 gadget_owner 的一个引用。
let gadget1 = Rc::new(Gadget{id: 1, owner: gadget_owner.clone()});
let gadget2 = Rc::new(Gadget{id: 2, owner: gadget_owner.clone()});

// 为主人更新它所拥有的工具
// 因为之前使用了 `Rc`，现在必须要使用 `Weak`，否则就会循环引用
gadget_owner.gadgets.borrow_mut().push(Rc::downgrade(&gadget1));
gadget_owner.gadgets.borrow_mut().push(Rc::downgrade(&gadget2));

// 遍历 gadget_owner 的 gadgets 字段
for gadget_opt in gadget_owner.gadgets.borrow().iter() {

    // gadget_opt 是一个 Weak<Gadget> 。 因为 weak 指针不能保证他所引用的对象
    // 仍然存在。所以我们需要显式的调用 upgrade() 来通过其返回值(Option<_>)来判
    // 断其所指向的对象是否存在。
    // 当然，Option 为 None 的时候这个引用原对象就不存在了。
    let gadget = gadget_opt.upgrade().unwrap();
    println!("Gadget {} owned by {}", gadget.id, gadget.owner.name);
}

// 在 main 函数的最后，gadget_owner，gadget1 和 gadget2 都被销毁。
// 具体是，因为这几个结构体之间没有了强引用（`Rc<T>`），所以，当他们销毁的时候。
// 首先 gadget2 和 gadget1 被销毁。
// 然后因为 gadget_owner 的引用数量为 0，所以这个对象可以被销毁了。
// 循环引用问题也就避免了

}

## 结构体自引用

struct SelfRef<'a> {  value: String,

// 该引用指向上面的value
pointer_to_value: &'a str,

}

解决途径：Pin、第三方库、Option、Unsafe

# 无畏并发

### 概述

#### 并行和并发

并发：轮流处理，多个队列使用同一个处理器

并行：同时处理，每个队列使用一个处理器

并发乍看之下，仍是一次处理一个任务，单线程串行没有区别；

但是，当某个队列运行时间过长时（假死），调度器可以终止这个任务，切换到另外的队列

![image-20220811160823653](file://D:/Note/Rust/image/image-20220811160823653.png?lastModify=1673405731?lastModify=1687513606)

#### 多线程编程的风险

由于多线程的代码是同时运行的，因此我们无法保证线程间的执行顺序，这会导致一些问题：

-   竞态条件(race conditions)，多个线程以非一致性的顺序同时访问数据资源
    
-   死锁(deadlocks)，两个线程都想使用某个资源，但是又都在等待对方释放资源后才能使用，结果最终都无法继续执行
    
-   一些因为多线程导致的很隐晦的 BUG，难以复现和解决
    

### 多线程的使用

#### 创建线程

thread::spawn

use std::thread;  use std::time::Duration;

fn main() {  thread::spawn(|| {  for i in 1..10 {  println!("hi number {} from the spawned thread!", i);  thread::sleep(Duration::from_millis(1));  }  });

for i in 1..5 {
    println!("hi number {} from the main thread!", i);
    thread::sleep(Duration::from_millis(1));
}

}

-   线程内部的代码使用闭包来执行
    
-   `main` 线程一旦结束，程序就立刻结束，因此需要保持它的存活，直到其它子线程完成自己的任务
    
-   `thread::sleep` 会让当前线程休眠指定的时间，随后其它线程会被调度运行，因此就算你的电脑只有一个 CPU 核心，该程序也会表现的如同多 CPU 核心一般，这就是并发！
    

如果多运行几次，你会发现好像每次输出会不太一样，因为：虽说线程往往是轮流执行的，但是这一点无法被保证！线程调度的方式往往取决于你使用的操作系统。总之，**千万不要依赖线程的执行顺序**

#### 等待子线程的结束

通过调用 `handle.join`，可以让当前线程阻塞，直到它等待的子线程的结束

use std::thread;  use std::time::Duration;

fn main() {  let handle = thread::spawn(|| {  for i in 1..5 {  println!("hi number {} from the spawned thread!", i);  thread::sleep(Duration::from_millis(1));  }  });

handle.join().unwrap();

for i in 1..5 {
    println!("hi number {} from the main thread!", i);
    thread::sleep(Duration::from_millis(1));
}

}

如果你将 `handle.join` 放置在 `main` 线程中的 `for` 循环后面，那就是另外一个结果：两个线程交替输出。

#### 在线程闭包中使用 move.

在一个子线程中直接使用另一个线程的数据

use std::thread;

fn main() {  let v = vec![1, 2, 3];

let handle = thread::spawn(|| {
    println!("Here's a vector: {:?}", v);
});

handle.join().unwrap();

}

error：主线程释放后，无法确定子线程对v的使用是否合法（v可能被释放，甚至在子线程创建前）

使用Move来转移所有权到子线程

let handle = thread::spawn(	move	|| {
    println!("Here's a vector: {:?}", v);
});

#### 线程结束

Rust为保证安全性，不提供主动杀死线程的方法，当线程执行完任务后，才结束线程

#### 多线程性能

线程的创建需要开销：约0.24ms

多线程的性能并非与线程数成正相关

![image-20220811164923073](file://D:/Note/Rust/image/image-20220811164923073.png?lastModify=1673405731?lastModify=1687513606)

多线程的开销往往是在锁、数据竞争、缓存失效上，这些限制了现代化软件系统随着 CPU 核心的增多性能也线性增加的野心

#### 线程屏障(Barrier)

在 Rust 中，可以使用 `Barrier` 让多个线程都执行到某个点后，才继续一起往后执行：

#### 用条件控制线程的挂起和执行

条件变量(Condition Variables)经常和 `Mutex` 一起使用，可以让线程挂起，直到某个条件发生后再继续执行：

use std::thread;  use std::sync::{Arc, Mutex, Condvar};

fn main() {  let pair = Arc::new((Mutex::new(false), Condvar::new()));  let pair2 = pair.clone();

thread::spawn(move|| {
    let &(ref lock, ref cvar) = &*pair2;
    let mut started = lock.lock().unwrap();
    println!("changing started");
    *started = true;
    cvar.notify_one();
});

let &(ref lock, ref cvar) = &*pair;
let mut started = lock.lock().unwrap();
while !*started {
    started = cvar.wait(started).unwrap();
}

println!("started changed");

}

上述代码流程如下：

1.  `main` 线程首先进入 `while` 循环，调用 `wait` 方法挂起等待子线程的通知，并释放了锁 `started`
    
2.  子线程获取到锁，并将其修改为 `true`，然后调用条件变量的 `notify_one` 方法来通知主线程继续执行
    

#### 只被调用一次的函数

有时，我们会需要某个函数在多线程环境下只被调用一次，例如初始化全局变量，无论是哪个线程先调用函数来初始化，都会保证全局变量只会被初始化一次，随后的其它线程调用就会忽略该函数：

use std::thread;  use std::sync::Once;

static mut VAL: usize = 0;  static INIT: Once = Once::new();

fn main() {  let handle1 = thread::spawn(move || {  INIT.call_once(|| {  unsafe {  VAL = 1;  }  });  });

let handle2 = thread::spawn(move || {
    INIT.call_once(|| {
        unsafe {
            VAL = 2;
        }
    });
});

handle1.join().unwrap();
handle2.join().unwrap();

println!("{}", unsafe { VAL });

}

代码运行的结果取决于哪个线程先调用 `INIT.call_once` （虽然代码具有先后顺序，但是线程的初始化顺序并无法被保证！因为线程初始化是异步的，且耗时较久），若 `handle1` 先，则输出 `1`，否则输出 `2`。

**call_once 方法**

执行初始化过程一次，并且只执行一次。

如果当前有另一个初始化过程正在运行，线程将阻止该方法被调用。

当这个函数返回时，保证一些初始化已经运行并完成，它还保证由执行的闭包所执行的任何内存写入都能被其他线程在这时可靠地观察到。

### 线程同步：消息传递

#### 消息通道

Rust 是在标准库里提供了消息通道(`channel`)

一个通道应该支持多个发送者和接收者

#### 多发送者、单接收者

标准库提供了通道`std::sync::mpsc`，其中`mpsc`（multiple producer, single consumer）

多发送者包括单发送者

##### **单发送者、单接收者**

use std::sync::mpsc;  use std::thread;

fn main() {  // 创建一个消息通道, 返回一个元组：(发送者，接收者)  let (tx, rx) = mpsc::channel();

// 创建线程，并发送消息
thread::spawn(move || {
    // 发送一个数字1, send方法返回Result<T,E>，通过unwrap进行快速错误处理
    tx.send(1).unwrap();

    // 下面代码将报错，因为编译器自动推导出通道传递的值是i32类型，那么Option<i32>类型将产生不匹配错误
    // tx.send(Some(1)).unwrap()
});

// 在主线程中接收子线程发送的消息并输出
println!("receive {}", rx.recv().unwrap());

}

-   接收消息的操作`rx.recv()`会**阻塞当前线程**，直到读取到值，或者通道被关闭
    

**不阻塞的 try_recv 方法**

use std::sync::mpsc;  use std::thread;

fn main() {  let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    tx.send(1).unwrap();
});

println!("receive {:?}", rx.try_recv());

}

try_recv：不会阻塞当前线程，如果没有收到消息，返回Err(Empty)

**传输具有所有权的数据**

如果实现了copy特征，会传递一份copy

如果没有实现copy特征，会传出所有权

##### 多发送者

子线程会move所有权，因此需要对发送者clone

use std::sync::mpsc;  use std::thread;

fn main() {  let (tx, rx) = mpsc::channel();  let tx1 = tx.clone();  thread::spawn(move || {  tx.send(String::from("hi from raw tx")).unwrap();  });

thread::spawn(move || {
    tx1.send(String::from("hi from cloned tx")).unwrap();
});

for received in rx {
    println!("Got: {}", received);
}

}

子线程的创建顺序未知，无法知道消息传递的顺序

但是通道内的消息是有序的，先进先出

#### 同步和异步通道

Rust 标准库的`mpsc`通道其实分为两种类型：同步和异步。

##### 异步通道

异步通道：无论接收者是否正在接收消息，消息发送者在发送消息时都不会阻塞:

use std::sync::mpsc;  use std::thread;  use std::time::Duration;  fn main() {  let (tx, rx)= mpsc::channel();

let handle = thread::spawn(move || {
    println!("发送之前");
    tx.send(1).unwrap();
    println!("发送之后");
});

println!("睡眠之前");
thread::sleep(Duration::from_secs(3));
println!("睡眠之后");

println!("receive {}", rx.recv().unwrap());
handle.join().unwrap();

}

运行后输出如下:

睡眠之前  发送之前  发送之后  //···睡眠3秒  睡眠之后  receive 1

主线程因为睡眠阻塞了 3 秒，因此并没有进行消息接收，而子线程却在此期间轻松完成了消息的发送。

等主线程睡眠结束后，才姗姗来迟的从通道中接收了子线程老早之前发送的消息。

从输出还可以看出，`发送之前`和`发送之后`是连续输出的，没有受到接收端主线程的任何影响，因此通过`mpsc::channel`创建的通道是异步通道。

##### 同步通道

同步通道：**发送消息是阻塞的，只有在消息被接收后才解除阻塞**，例如：

use std::sync::mpsc;  use std::thread;  use std::time::Duration;  fn main() {  let (tx, rx)= mpsc::sync_channel(0);

let handle = thread::spawn(move || {
    println!("发送之前");
    tx.send(1).unwrap();
    println!("发送之后");
});

println!("睡眠之前");
thread::sleep(Duration::from_secs(3));
println!("睡眠之后");

println!("receive {}", rx.recv().unwrap());
handle.join().unwrap();

}

运行后输出如下：

睡眠之前  发送之前  //···睡眠3秒  睡眠之后  receive 1  发送之后

可以看出，主线程由于睡眠被阻塞导致无法接收消息，因此子线程的发送也一直被阻塞，直到主线程结束睡眠并成功接收消息后，发送才成功：**发送之后**的输出是在**receive 1**之后，说明**只有接收消息彻底成功后，发送消息才算完成**。

##### 消息缓存

细心的读者可能已经发现在创建同步通道时，我们传递了一个参数`0`: `mpsc::sync_channel(0);`，这是什么意思呢？

该值可以用来指定同步通道的消息缓存条数，当你设定为`N`时，发送者就可以无阻塞的往通道中发送`N`条消息，当消息缓冲队列满了后，新的消息发送将被阻塞(如果没有接收者消费缓冲队列中的消息，那么第`N+1`条消息就将触发发送阻塞)。

#### 传输多种类型数据

一个消息通道只能传输一种类型的数据，如果你想要传输多种类型的数据，可以为每个类型创建一个通道，你也可以使用枚举类型来实现

注：枚举类型具有内存对齐，按照最大的那个成员内存

#### 新手容易遇到的坑

`mpsc`虽然相当简洁明了，但是在使用起来还是可能存在坑：

use std::sync::mpsc;  fn main() {

use std::thread;

let (send, recv) = mpsc::channel();
let num_threads = 3;
for i in 0..num_threads {
    let thread_send = send.clone();
    thread::spawn(move || {
        thread_send.send(i).unwrap();
        println!("thread {:?} finished", i);
    });
}

// 在这里drop send...

for x in recv {
    println!("Got: {}", x);
}
println!("finished iterating");

}

以上代码看起来非常正常，但是运行后主线程会一直阻塞，最后一行打印输出也不会被执行，原因在于： 子线程拿走的是复制后的`send`的所有权，这些拷贝会在子线程结束后被`drop`，因此无需担心，但是`send`本身却直到`main`函数的结束才会被`drop`。

之前提到，通道关闭的两个条件：发送者全部`drop`或接收者被`drop`，要结束`for`循环显然是要求发送者全部`drop`，但是由于`send`自身没有被`drop`，会导致该循环永远无法结束，最终主线程会一直阻塞。

解决办法很简单，`drop`掉`send`即可：在代码中的注释下面添加一行`drop(send);`。

#### mpmc(多发送者，多接收者)

使用第三方库、性能更佳

-   [**crossbeam-channel**](https://github.com/crossbeam-rs/crossbeam/tree/master/crossbeam-channel), 老牌强库，功能较全，性能较强，之前是独立的库，但是后面合并到了`crossbeam`主仓库中
    
-   [**flume**](https://github.com/zesterer/flume), 官方给出的性能数据某些场景要比 crossbeam 更好些
    

### 线程同步：锁、Condvar、信号量

#### 共享内存

消息传递类似一个**单所有权**的系统：一个值同时只能有一个所有者，如果另一个线程需要该值的所有权，需要将所有权通过消息传递进行转移。

而**共享内存**类似于一个多所有权的系统：多个线程可以同时访问同一个值

#### 互斥锁（Mutex）

同一时间，只允许一个线程`A`访问该值，其它线程需要等待`A`访问完成后才能继续

##### 单线程

use std::sync::Mutex;

fn main() {  // 使用`Mutex`结构体的关联函数创建新的互斥锁实例  let m = Mutex::new(5);

{
    // 获取锁，然后deref为`m`的引用
    // lock返回的是Result
    let mut num = m.lock().unwrap();
    *num = 6;
    // 锁自动被drop
}

println!("m = {:?}", m);

}

数据访问：m.lock()申请访问。并**阻塞当前线程，直至访问到数据**

返回一个智能指针`MutexGuard<T>`，实现了Deref特征和Drop特征

Deref特征使其自动解引用，暴露T

##### 单线程死锁

use std::sync::Mutex;

fn main() {  let m = Mutex::new(5);

let mut num = m.lock().unwrap();
*num = 6;
// 锁还没有被 drop 就尝试申请下一个锁，导致主线程阻塞
// drop(num); // 手动 drop num ，可以让 num1 申请到下个锁
let mut num1 = m.lock().unwrap();
*num1 = 7;
// drop(num1); // 手动 drop num1 ，观察打印结果的不同

println!("m = {:?}", m);

}

##### 多线程

use std::sync::{Arc, Mutex};  use std::thread;

fn main() {  let counter = Arc::new(Mutex::new(0));  let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();

        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());

}

子线程需要通过`move`拿走锁的所有权，因此我们需要使用**多所有权**来保证每个线程都拿到数据的独立所有权，智能指针Rc<T>和Arc<T>都可以做到，但是**多线程只能使用Arc<T>**,Rc没有实现**Send**特性

##### 多线程死锁

常见场景：线程1访问锁1，线程2访问锁2；此时，线程1又申请访问锁2，线程2申请访问锁1；两个线程都阻塞，没有释放锁1锁2，开始套娃，进入死锁

ps:这种场景是偶发死锁，因为多线程的执行顺序每次都不同，例如线程1在线程2访问锁2之前，就完成了对锁1、锁2的访问

use std::{sync::{Mutex, MutexGuard}, thread};  use std::thread::sleep;  use std::time::Duration;

use lazy_static::lazy_static;  lazy_static! {  static ref MUTEX1: Mutex<i64> = Mutex::new(0);  static ref MUTEX2: Mutex<i64> = Mutex::new(0);  }

fn main() {  // 存放子线程的句柄  let mut children = vec![];  for i_thread in 0..2 {  children.push(thread::spawn(move || {  for _ in 0..1 {  // 线程1  if i_thread % 2 == 0 {  // 锁住MUTEX1  let guard: MutexGuard<i64> = MUTEX1.lock().unwrap();

                println!("线程 {} 锁住了MUTEX1，接着准备去锁MUTEX2 !", i_thread);

                // 当前线程睡眠一小会儿，等待线程2锁住MUTEX2
                sleep(Duration::from_millis(10));

                // 去锁MUTEX2
                let guard = MUTEX2.lock().unwrap();
            // 线程2
            } else {
                // 锁住MUTEX2
                let _guard = MUTEX2.lock().unwrap();

                println!("线程 {} 锁住了MUTEX2, 准备去锁MUTEX1", i_thread);

                let _guard = MUTEX1.lock().unwrap();
            }
        }
    }));
}

// 等子线程完成
for child in children {
    let _ = child.join();
}

println!("死锁没有发生");

}

##### try_lock

与lock不同，try_lock不会阻塞线程；只是进行一次尝试访问，访问失败就返回一个错误

#### 读写锁 RwLock

同时允许多个线程进行读取，但是写操作只能有一个

读写操作不能同时存在

性能低于Mutex

### 基于Send和Sync的线程安全

#### Send

`Send` 标记 trait 表明类型的所有权可以在线程间传递。

几乎所有的 Rust 类型都是`Send` 的，除了 `Rc<T>`、裸指针；

#### Sync

# 面向对象编程

Rust中类似其他语言的面向对象编程概念

### 对象

对象定义：一个 **对象** 包含数据和操作这些数据的过程。这些过程通常被称为 **方法** 或 **操作**

Rsut：枚举和结构体包含数据，impl提供关联的方法

### 封装

封装定义：对象的**实现细节**、**内部数据**不想被使用对象的代码获取；仅提供对外接口，用以对象交互

Rust：用pub来暴露对外接口，内部私有数据通过对外接口来获取操作

### 继承

继承定义：一个对象可以定义为继承另一个对象的定义，这使其可以获得父对象的数据和行为，而无需重新定义

严格来说，rust并不符合继承定义，但是rust可通过trait、trait对象这两个手段来达到类似的作用

①trait：通过trait标注，使得结构体可以复用trait的默认实现方法，以达到对**代码的复用**

②trait对象：子类可以用于父类被使用的地方 ——**多态**

### trait对象

#### WHY

vector<>中只能存放同一类型的数据

-   可以通过存放枚举值，来存放不同类型
    
-   引入**特征对象**的概念，将实现相同**trait**但不同**类型**的对象存在在一个动态数组中
    

**Box<dyn trait>**

#### trait对象定义

pub trait Draw {  //定义trait  fn draw(&self);  }

pub struct Screen {  pub components: Vec<Box<dyn Draw>>,  //定义trait对象  }

impl Screen {  pub fn run(&self) {  for component in self.components.iter() {  component.draw();  }  }  }

pub struct Button {  pub width: u32,  pub height: u32,  pub label: String,  }  impl Draw for Button {  //Button实现了triat特征  fn draw(&self) {  // 实际绘制按钮的代码  }  }  }

struct SelectBox {  width: u32,  height: u32,  options: Vec<String>,  }

impl Draw for SelectBox {  //SelectBox也实现了triat特征  fn draw(&self) {  // code to actually draw a select box  }  }

let screen = Screen {	//实例化screen、并存入button和selectbox
    components: vec![
        Box::new(SelectBox {
            width: 75,
            height: 10,
            options: vec![
                String::from("Yes"),
                String::from("Maybe"),
                String::from("No")
            ],
        }),
        Box::new(Button {
            width: 50,
            height: 10,
            label: String::from("OK"),
        }),
    ],
};

screen.run();	//调用trait对象各自的特征方法

##### trait & 泛型T

pub trait Draw {  fn draw(&self);  }

pub struct Screen<T: Draw> {  pub components: Vec<T>,  }

impl<T> Screen<T>  where T: Draw {  pub fn run(&self) {  for component in self.components.iter() {  component.draw();  }  }  }

泛型也可以传入多种实现了trait的类型

BUT!!!

泛型的实例，只能有一种类型：实例screen中，要么全是A，要么全是B

trait对象的实例，可以包含多种类型

#### 动态分发

trait对象在编译期无法确定具体类型，需要通过指针在运行期判断

trait对象的**大小不是固定**的，因此总是使用trait对象的**引用形式**，引用大小固定

trait对象的引用形式包括一个**ptr指针**、一个**vptr指针**

-   ptr指针：指向实现trait的具体实例
    
-   vptr指针：指向一个虚表，里面存放了实现trait方法的函数指针，调用trait对象特征方法时，需要在表中查找对应实现
    

#### 对象安全

self：指代实例对象

Self：指代方法或类型的别名

当一个特征的所有方法都有如下属性时，它的对象才是安全的：

-   特征的方法不能返回**Self**
    
-   特征的方法不能有泛型参数
    

对于trait对象的特征方法如果返回Self，无法知道其具体类型是什么

pub trait Clone {  fn clone(&self) -> Self;  }

# 模式 & 匹配

模式是 Rust 中的特殊语法，它用来匹配类型中的结构和数据，它往往和 `match` 表达式联用，以实现强大的模式匹配能力。模式一般由以下内容组合而成：

-   字面值
    
-   解构的数组、枚举、结构体或者元组
    
-   变量
    
-   通配符
    
-   占位符
    

**let、for、match都要求穷尽**

## 模式适用场景

### match

用来对模型进行匹配，类似switch和if，但是match的判断条件可以是不同类型，而不局限于bool

enum Coin {  Penny,  Nickel,  Dime,  Quarter,  }

fn value_in_cents(coin: Coin) -> u8 {  match coin {  Coin::Penny => 1,  Coin::Nickel => {  println!("Lucky");  5,  }  Coin::Dime => 10,  Coin::Quarter => 25,  }  }

match本身为**表达式**，可用作赋值（**模式匹配**）

**穷尽匹配**

枚举中所有类别，都需要在match中检测匹配

通配符：_、涵括其余类别

#### let

let Some(x) = some_option_value;

let是穷尽匹配的，some_option_value可能是some，也可能是none；此时，没有对应none的可能

#### if let

`if let` 往往用于匹配一个模式，而忽略剩下的所有模式的场景：

if let是match的语法糖，用来简化match

if let PATTERN = SOME_VALUE {

}

#### while let

// Vec是动态数组  let mut stack = Vec::new();

// 向数组尾部插入元素  stack.push(1);  stack.push(2);  stack.push(3);

// stack.pop从数组尾部弹出元素  while let Some(top) = stack.pop() {  println!("{}", top);  }

只要stack.pop()返回some<T>,while就会执行，直到stack.pop()返回None、与Some(top)不匹配

#### for循环

let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {  println!("{} is at index {}", value, index);  }

enumerate（）每次返回一个元组（索引,值）

#### 函数参数

fn foo(x: i32) {  // 代码  }

x就是一个**模式**

fn print_coordinates(&(x, y): &(i32, i32)) {  println!("Current location: ({}, {})", x, y);  }

fn main() {  let point = (3, 5);  print_coordinates(&point);  }

&(3,5) 会匹配 &（x,y）

## Refutability（可反驳性）

模式有两种形式：refutable、irrefutable

-   irrefutable：能够匹配任何模式（需要覆盖所有可能性）
    
-   refutable：可能存在匹配失败（无需穷尽）
    

#### irrefutable

let、for、match都是irrefutable

因此，irrefutable用来解构的变量，必须覆盖右边所有的可能性

some_option_value可能是Some<T>、也可能是None；但是左边只给出了Some(X),无法覆盖

let Some(x) = some_option_value  //error

#### refutable

if let、while let都是refutable

if let Some(x) = some_option_value {  println!("{}", x);  }

## 全模式列表

列举所有模式的使用方式，自行查找[全模式列表](https://course.rs/basic/match-pattern/all-patterns.html)

# 高级特征

## Unsafe Rust

Rust的编译器检查功能**强大**且**保守**，因此需要unsafe来突破编译器的限制

BUT！！！，需要自己来负责检查的职责，**确保代码的正确性**

-   解引用裸指针，就如上例所示
    
-   调用一个 `unsafe` 或外部的函数
    
-   访问或修改一个可变的[静态变量](https://course.rs/advance/global-variable.html#%E9%9D%99%E6%80%81%E5%8F%98%E9%87%8F)
    
-   实现一个 `unsafe` 特征
    
-   访问 `union` 中的字段
    

### 解引用裸指针（raw pointer）

#### 裸指针

可变裸指针:*** mut T**

不可变裸指针：*** const T**

裸指针相较于引用和智能指针

-   绕过Rust的借用规则，拥有两个可变指针，同时拥有可变、不可变指针
    
-   不能保证指向内存的合法性
    
-   没有实现自动回收机制Drop
    
-   可以是null
    

#### 裸指针的创建

//基于引用来创建裸指针  let mut num = 5;

let r1 = &num as *const i32;  let r2 = &mut num as *mut i32;

//基于地址来创建  let address = 0x012345usize;  let r = address as *const i32;

//基于智能指针来创建  let a: Box<i32> = Box::new(10);  // 需要先解引用a  let b: _const i32 = &_a;  // 使用 into_raw 来创建  let c: *const i32 = Box::into_raw(a);

#### 裸指针的解引用

裸指针的创建是safe的，但是对裸指针解引用是unsafe，因为此时不会检查内存的合法性、借用规则等等

只有一件事，给定了我一个地址，又给定了长度(类型字节数)，那我就解析 ptr,ptr+len 这段内存

unsafe{ *a };

### FFI

`FFI`（Foreign Function Interface），使得Rust可以与其他语言进行交互，扩展生态

extern "C"：定义了**应用二进制接口**`ABI` (Application Binary Interface),按照C的形式

#[no_mangle]：用于告诉 Rust 编译器：不要乱改函数的名称

#### 导入

extern "C" {  fn abs(input: i32) -> i32;  }

fn main() {  unsafe {  println!("Absolute value of -3 according to C: {}", abs(-3));  }  }

#### 导出

#[no_mangle]  pub extern "C" fn call_from_c() {  println!("Just called a Rust function from C!");  }

## 高级类型

### newtype 和 类型别名

#### newtype

使用[元组结构体](https://course.rs/basic/compound-type/struct.html#%E5%85%83%E7%BB%84%E7%BB%93%E6%9E%84%E4%BD%93tuple-struct)的方式将已有的类型包裹起来：

`struct Meters(u32);`

此处 `Meters` 就是一个 `newtype`

##### **为外部类型实现外部特征**

该种情况，不满足孤儿原则，需要借助New type来实现

struct Wrapper(Vec<String>);  //利用newtype 将 Vec封装成Wrapper

impl fmt::Display for Wrapper {  //为wrapper  实现特征  Display  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {  write!(f, "[{}]", self.0.join(", "))  }  }

##### 更好的可读性

struct Meters(u32)

Meter比u32的可读性更强，力量的代价就是代码较繁琐，你需要为Meter单独实现一些方法

##### 类型异化

Meter和u32并不是同一个类型

fn test(meter:Meter){};  //无法将u32传入

##### **隐藏内部类型的细节**

struct Meters(u32);

fn main() {  let i: u32 = 2;  assert_eq!(i.pow(2), 4);

let n = Meters(i);
// 下面的代码将报错，因为`Meters`类型上没有`pow`方法
// assert_eq!(n.pow(2), 4);

}

newtpye无法直接调用内部类型的方法，但是可通过 `n.0.pow(2)`的方式，取出内部元素再调用

#### 类型别名（Type Alias）

类型别名只是一个别名，仍然是同一个类型；类似 **typedef**

tyoe Meter = u32;

let a:Meter =1;  let b:u32 =2;  let c= a+b;  //Meter、u32是同一种类型

-   简化类型名
    

type Thunk = Box<dyn Fn() + Send + 'static>;  //后续Box<dyn Fn() + Send + 'static>都用Thunk代替

-   使类型名更有意义
    

tyoe Meter = u32;

### 动态大小类型（DST）

指：编译器无法在编译阶段，确定类型的大小，只有到程序运行时，才能确定

注：Vec、String、Hashmap等不是DST；

因为，**编译器关心的是栈上的空间分配**；这些集合类型，动态底层数据存放在堆上，栈上存放引用（地址、大小、容量），栈上的引用类型是固定大小的

**DST无法通过编译**

#### 常见DST

##### **创建动态大小的数组**

fn my_function(n: usize) {  let array = [123; n];  }

##### **str**

s1：在栈上占据12个字节

s2: 在栈上占据15个字节

str在栈上的类型大小是动态的，因此 常用 **&str**

// error  let s1: str = "Hello there!";  let s2: str = "How's it going?";

// ok  let s3: &str = "on?"  }

##### **特征对象**

特征对象的大小是不确定的，由具体实现trait的类型所决定

trati MyThing{}

fn foobar_1(thing: &dyn MyThing) {} // OK  fn foobar_2(thing: Box<dyn MyThing>) {} // OK  fn foobar_3(thing: MyThing) {} // ERROR!  fn foobar_3(thing: dyn MyThing) {} // ERROR!

##### 小结

DST无法通过编译，只能利用&、Box<T>的方式间接使用

#### Sized特征

sized特征用来约束泛型，过滤DST

常用的数据类型 编译器都默认实现了sized特征，并隐式添加

### 类型转换

[From 和 Into - 通过例子学 Rust 中文版 (rustwiki.org)](https://rustwiki.org/zh-CN/rust-by-example/conversion/from_into.html)

注：Rust的类型转化，感觉要比C++要严格

let a:32 =1;  //error not match  let a:32 =1.0  //ok

#### as运算符

存在数值溢出的情况

let b = 300_i32 as i8; //300超过了i8的表示范围  assert!(b,44)  //b = 44

#### From & Into

**trati From**：标准库中提供大量类型实现，用于将入参类型int，转化为目标类型Number

use std::convert::From;

#[derive(Debug)]  struct Number {  value: i32,  }

impl From<i32> for Number {  fn from(item: i32) -> Self {  Number { value: item }  }  }

fn main() {  let num = Number::from(30);  println!("My number is {:?}", num);  }

**trait Into**:与From互生，实现From后可直接使用Into，需要**显示指定类型**：Number

use std::convert::From;

#[derive(Debug)]  struct Number {  value: i32,  }

impl From<i32> for Number {  fn from(item: i32) -> Self {  Number { value: item }  }  }

fn main() {  let int = 5;  // 试试删除类型说明  let num: Number = int.into();  println!("My number is {:?}", num);  }

#### TryFrom & TryInto

类似From、Into，区别为**返回一个Result<T,E>**用来提示类型转化是否失败，常用来易出错的类型转化

use std::convert::TryInto;  let a: u8 = 10;  let b: u16 = 1500;

let b_: u8 = match b.try_into() {
    Ok(b1) => b1,
    Err(e) => {
        println!("{:?}", e.to_string());
        0
    }
};

TryInto会自动捕捉 大数值类型 转换 小数值类型时的**溢出错误**

**返回结果为枚举类型**Result

enum Result<T, E> {  Ok(T),  Err(E),  }

返回结果用match模式匹配

#### ToString & FromStr

trait ToString：将类型转化为String，需要为类型实现ToString trait，或者直接实现 trait fmt::Display，Display提供ToString

use std::fmt;

struct Circle {  radius: i32  }

impl fmt::Display for Circle {  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {  write!(f, "Circle of radius {}", self.radius)  }  }

fn main() {  let circle = Circle { radius: 6 };  println!("{}", circle.to_string());  }

trait FromStr：为类型实现trait FromStr，用parse方法将str转化为类型

fn main() {  let parsed: i32 = "5".parse().unwrap();  let turbo_parsed = "10".parse::<i32>().unwrap();

let sum = parsed + turbo_parsed;
println!{"Sum: {:?}", sum};

}

#### transmute

unsafe的特性

`mem::transmute<T, U>` 将类型 `T` 直接转成类型 `U`，唯一的要求就是，这两个类型占用同样大小的字节数

`mem::transmute_copy<T, U>` 从 `T` 类型中拷贝出 `U` 类型所需的字节数，然后转换成 `U`

危险！！！

注：尤其遇到FFI的时候，不同语言的内存对齐规则是不同的

## 全局变量

用来简化状态共享的代码，例如全局ID，全局数据等

### 编译期初始化

全局变量的初始化，发生在编译期

#### 静态常量

const MAX_ID: usize = usize::MAX / 2;  fn main() {  println!("用户ID允许的最大值是{}",MAX_ID);  }

const声明

显示指定类型

大写命名

可在任意作用域定义，其生命周期贯穿整个程序的生命周期 ps：这是保证静态常量全局有效，不会在脱离作用域后，被drop;想访问静态常量，需要可见

静态常量不能被遮蔽

静态常量的赋值只能是常量，表达式等，需要在编译期就能计算出来

静态常量被编译器内联到代码中？？？， 引起对静态常量的引用指向的地址可能不同

#### 静态变量

static mut REQUEST_RECV: usize = 0;  fn main() {  unsafe {  REQUEST_RECV += 1;  assert_eq!(REQUEST_RECV, 1);  }  }

这是unsafe的，在多线程中容易造成脏数据

**尽量只在单线程下，使用静态变量**

#### 原子类型

完美解决多线程的脏数据

用到再看[线程同步：Atomic 原子操作与内存顺序 - Rust语言圣经(Rust Course)](https://course.rs/advance/concurrency-with-threads/sync2.html)

### 运行期初始化

## Result、Option

本节更多的探讨对Result、Option的快速处理

更精确的处理，可参考：[错误处理 - Rust语言圣经(Rust Course)](https://course.rs/advance/errors.html)

#### or()、and()

or：比较两个表达式，第一个不为None时，返回第一个表达式；第一个为None时，返回第二个表达式

and:第一个表达式不为None,返回第二个表达式；否则，返回None

pub const fn or(self, optb: Option<T>) -> Option<T>
where
    T: ~const Destruct,
{
    match self {
        Some(x) => Some(x),
        None => optb,
    }
}

  pub const fn or<F>(self, res: Result<T, F>) -> Result<T, F>
where
    T: ~const Destruct,
    E: ~const Destruct,
    F: ~const Destruct,
{
    match self {
        Ok(v) => Ok(v),
        // FIXME: ~const Drop doesn't quite work right yet
        #[allow(unused_variables)]
        Err(e) => res,
    }
}

#### or_else()、and_then()

与or、and类似，只是第二个参数是闭包

#### filter()

对Option进行过滤，None直接返回None,Some(x) 且 满足predicate(x)过滤，返回some(x)

pub const fn filter<P>(self, predicate: P) -> Self
where
    T: ~const Destruct,
    P: ~const FnOnce(&T) -> bool,
    P: ~const Destruct,
{
    if let Some(x) = self {
        if predicate(&x) {
            return Some(x);
        }
    }
    None
}

#### map()、map_err()

map()：用来将Some(x)、Ok(x)映射成Some(y),Ok(y)

pub const fn map<U, F>(self, f: F) -> Option<U>
where
    F: ~const FnOnce(T) -> U,
    F: ~const Destruct,
{
    match self {
        Some(x) => Some(f(x)),
        None => None,
    }
}

map_err()：用来映射Err(x) to Err(y)

#### map_or()、map_or_else()

map_or在map的基础上，提供一个default

pub fn map_or<U, F: FnOnce(T) -> U>(self, default: U, f: F) -> U {
    match self {
        Ok(t) => f(t),
        Err(_) => default,
    }
}

map_or_else提供的default不是一个值，而是一个闭包

#### ok_or()、ok_or_else()

用来将Option转化为Result,

fn main() {  const ERR_DEFAULT: &str = "error message";

let s = Some("abcde");
let n: Option<&str> = None;

let o: Result<&str, &str> = Ok("abcde");
let e: Result<&str, &str> = Err(ERR_DEFAULT);

assert_eq!(s.ok_or(ERR_DEFAULT), o); // Some(T) -> Ok(T)
assert_eq!(n.ok_or(ERR_DEFAULT), e); // None -> Err(default)

}

## Macro宏编程

[【译】Rust宏：教程与示例（一） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/353421021)

[paste：结合驼峰、蛇形，可对函数名进行改写等)](https://docs.rs/paste/latest/paste/)

#### 小结

Rust的宏：对**不同类型**但是具有**相同行为**的函数进行简化

-   可以接受不同数量的参
    
    vec![1]  vec![1,2,3]
    
-   可以接受不同类型的参数
    
    let a:f32 =0.;  let b:usize =0;  println!("{}",a);  println!("{}",b);
    
-   可以重载，根据传入不同的参数，执行不同的函数分支，类似match
    

macro_rules! test {  (​left),  stringify!(​left && 每个分支都必须以分号结束。​left:expr, ​left),  stringify!(​left || $right)  );  }

宏并不在运行期直接执行代码，

**在编译器，根据调用传入的参数，match相应的代码分支，然后在调用处展开代码**

rust宏的强大且复杂，强大之处来源于各种各样的token

#### 实践

cargo expand：用来展开宏和#[drive]

cargo expand > expand.rs 展开到expand.rs中

**paste库 提供[<>]拼接函数名、类型等**

通过**（$name:ident)**来解析，灵活且强大

**$ $parm)*：重复语法**

$()：重复符号，

**$parm：重复的内容，可以用来接受参数，也能用来重复执行**

，：注意，的位置

*：重复标志， *代表出现0次或多次；+代表出现1次或多次；

macro_rules! geometry_create{  ($name:ident)=>{ paste!{ pub extern "C" fn [<VreCreate $name:caml>](config:[<Vre$name:caml MeshConfig>])->VreGeometry{ //caml驼峰写法 let config:[<$name:caml MeshConfig>] = unsafe{std::mem::transmute(config)}; //low小写 return_handle!([<create_geometry_ $name:low>](config))  }  }  }  }

宏展开前  geometry_create!(box)  编译期展开后  pub extern "C" fn VreCreateBox(config:VreBoxMeshConfig)-<VreGeometry{  let config:BoxMeshConfig = unsafe{std::mem::transmute(config)};  return_handle!(create_geometry_box(config))  }

# Rust拾遗

## mod、use

mod ：将module打包复制

use：引入module到当前路径

//将mod vre_api 重命名为 VreApi 并导出VreApi  pub mod vre_api;  pub use vre_pi as VreApi;  pub use VreAPI::*;

## Arena

支持插入删除，**不保序**，支持遍历。通过Handle进行寻址，需要上层对Handle进行管理。

删除的位置会进入到freelist中，下次有插入就会重用这个位置

**内存复用率高，但是不保序**

Struct Arena{
	items:Vec<Entry<T>>,//存储数据的地方
	free_list_head:Option<usize>,	//当前你存中空余的index
	len：usize
}
enum Entry<T>{
	Free{next_free:Option<usize>}	//当前位置free，且指向下一个free
	Occupied{value}					//当前位置被占用，且存放数据
}
Struct Handle<T>{
	handle:usize	//index 指向Arena中的items
}

①insert(value)：先寻找free_list尝试插入,没有空位的话 就对vec扩容；插入成功返回handle给上层

②remove(handle)：通过handle拿到index,释放内存，并更新

items[index]=Free(nex_free:free_list)
free_list = Some(handle)

## 数据类型的内存布局

[Rustt/[2022-05-04] 可视化 Rust 各数据类型的内存布局.md at main · rustlang-cn/Rustt (github.com)](https://github.com/rustlang-cn/Rustt/blob/main/Articles/%5B2022-05-04%5D%20%E5%8F%AF%E8%A7%86%E5%8C%96%20Rust%20%E5%90%84%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%86%85%E5%AD%98%E5%B8%83%E5%B1%80.md)

## FFI

想要从Rust端，传出一个Vec<T>，可以通过传出ptr和len,外部接口拿到ptr和len来读取数据

难点：Vec如何释放

如果在Rust端释放，那么外部拿到的ptr访问内存已经释放

tips：**这种情况下，可能仍能读取到正确数据，但是并不稳定，这块内存已经被释放，如果有新的数据写入这块数据，那么之前的数据就被覆盖了**

如果不在Rust端释放，那么需要抑制Rust端的drop，在外部读取完数据后，再调用Rust端来drop

[How to expose a Rust `Vec` to FFI? - Stack Overflow](https://stackoverflow.com/questions/39224904/how-to-expose-a-rust-vect-to-ffi)

### 内存对齐

[Size, Stride, Alignment · YUI 的严肃文 (linkexin.github.io)](https://linkexin.github.io/notes/Size,-Stride,-Alignment#:~:text=1%20Size%20%E6%98%AF%E8%AF%BB%E5%8F%96%E5%88%B0%E5%BD%93%E5%89%8D%E5%AE%9E%E4%BE%8B%E6%89%80%E6%9C%89%E6%95%B0%E6%8D%AE%E7%9A%84%E5%AD%97%E8%8A%82%E6%95%B0%202,Stride%20%E6%98%AF%E4%BB%8E%E5%BD%93%E5%89%8D%E5%AE%9E%E4%BE%8B%E5%89%8D%E8%BF%9B%E5%88%B0%E4%B8%8B%E4%B8%80%E4%B8%AA%E5%AE%9E%E4%BE%8B%E7%9A%84%E5%AD%97%E8%8A%82%E6%95%B0%203%20Alignment%20%E6%98%AF%E6%AF%8F%E4%B8%AA%E5%AE%9E%E4%BE%8B%E5%BF%85%E9%A1%BB%E4%BD%8D%E4%BA%8E%E7%9A%84%E3%80%8C%E5%9D%87%E5%8C%80%E5%8F%AF%E9%99%A4%E6%95%B0%E3%80%8D%E6%95%B0%E5%AD%97%EF%BC%8C%E5%A6%82%E6%9E%9C%E8%A6%81%E4%B8%BA%E5%A4%8D%E5%88%B6%E6%95%B0%E6%8D%AE%E8%80%8C%E5%88%86%E9%85%8D%E5%86%85%E5%AD%98%EF%BC%8C%E5%88%99%E9%9C%80%E8%A6%81%E6%8C%87%E5%AE%9A%E6%AD%A3%E7%A1%AE%E7%9A%84%E5%AF%B9%E9%BD%90%E6%96%B9%E5%BC%8F%EF%BC%88%E4%BE%8B%E5%A6%82%EF%BC%8Callocate%EF%BC%88byteCount%EF%BC%9A100%EF%BC%8Calignment%EF%BC%9A4%EF%BC%89%EF%BC%89)

#### 内存对齐的本质

通过牺牲部分空间，来加快数据的读取效率

如果数据紧密排布的话，读取int b时，需要读取两块内存，再拼接到一起

#### repr

**无repr**

默认为Rust的对齐格式，可能每次排布的方式都不相同

**#[repr(C)]**

保证struct的对齐格式，与C一致，FFI通用格式

**#[repr(packed)]**

紧密排布，去掉对齐填充

**#[repr(u * 、i*)]**

指定无成员枚举的大小，如果超过则编译报错

**#[repr(alig(n)]**

强制按照2^n来排布

# 实用工具

## 第三方库

paste：用来拼接宏

[粘贴 - 生锈 (docs.rs)](https://docs.rs/paste/latest/paste/)

## 插件

Code Spell Checker：检查单词拼写

fmt：自动格式化

rust-analyzer

Even Better TOML，支持 .toml 文件完整特性

Error Lens, 更好的获得错误展示

CodeLLDB or C/C++, Debugger 程序

注：IDE需要开启断点功能 文件->首选项->设置
