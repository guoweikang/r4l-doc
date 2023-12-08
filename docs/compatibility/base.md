# 基础知识

在正式进入后续了解兼容性之前，先补充一些基础知识点(RUST课程中未涉及) 


## 条件编译

本节内容参考来自
[RUST 参考文档](https://rustwiki.org/zh-CN/reference/conditional-compilation.html)

条件编译本质作用，就是为了可以做到选择性的编译某些代码，也就是说，该部分代码可能被编译在包里面，也可能没有被编译进去


### 基本语法

Rust条件编译一共有三种基本使用语法结构，分别对应不同的使用场景

第一种: 是否编译某段代码

```
#[cfg(Condition)]
fn cfg_bool(); 
```

第二种: 是否使用某个属性

```
#[cfg(Condition, attribute)]
fn cfg_bool(); 

```
上述代码等于：当Condition为真时，会使用`,`后面的的属性 
```
#[attribute]
fn cfg_bool(); 
```

第三种: 控制代码逻辑

```
let machine_kind = if cfg!(Condition) {
  "Condition is true"
} else {
  "Condition is false"
};
```

已经知道，cfg后面主要跟一个`Condition`，作为条件，那就应该支持条件语法,条件支持多个条件进行逻辑运算

```
#[cfg(all(conditon1,conditon2,conditon3))] // 仅当三个条件都为真 才成立 
#[cfg(any(conditon1,conditon2,conditon3))] // 三个条件任意一个为真，则成立
#[cfg(not(cond1)] // 当条件为假时 才成立
``` 
`not`、`all`以及 `not` 可以配合使用

```
[cfg(all(not(cond1), cond2))] //当 cond1 不满足且 cond2 满足时，才会进行条件编译。
```

cfg_attr中的conditon表达式和上面类似

```
[cfg(all(not(cond1), cond2))，attr1，attr2] //当 cond1 不满足且 cond2 满足时，才会标记attr1 attr2 属性
```

### 支持键值对

条件表达式中也支持 `a = b` 的语法 

```
[cfg(feature1=abcd)] //当 feature1=abcd时 条件才成立
```

### 条件的设置
我们已经熟悉了条件表达的基本语法，那么条件是谁设置的？ 条件来源也有几种情况

rustc 通过使用 `--cfg` 选项指定编译条件

```
rustc --cfg cond1 --cfg "feature1=abcd" // 编译器设置cond1=true 设置feature1=abcd  
```

rustc 通过使用 `--test` 选项指定编译条件 
```
#[cfg(test)] //当编译器指定了test选项 该选项为真
```

编译器自带有一些编译选项 关于平台的 通过命令`rustc --print cfg` 查看
```
$ rustc --print cfg
debug_assertions
panic="unwind"
target_arch="x86_64"
target_endian="little"
target_env="gnu"
target_family="unix"
target_feature="fxsr"
target_feature="sse"
target_feature="sse2"
target_has_atomic="16"
target_has_atomic="32"
target_has_atomic="64"
target_has_atomic="8"
target_has_atomic="ptr"
target_os="linux"
target_pointer_width="64"
target_vendor="unknown"
unix
```


## (不)稳定特性

关于特性: RUST 给标准库以及编译器规定了一个术语，叫做特性 `feature` 

我们可以在基础库(比如`core` `alloc`) 中的`lib.rs` 的开头 定义了很多特性`#![feature(xxxx)]`

引入特性的目的，是为了可以让版本具备在开发中迭代的能力，打个比方:  我们在开发某个功能时，是否会一次把该功能开发完成？
往往都是一边开发，一边演进，随着需求不断变化，会一直演进，直到趋于稳定

RUST中的特性就是为了完成上述工作 因此特性有两种分类: 稳定版本特性 和 不稳定特性 

稳定特性，默认外部包都可以使用， 不稳定特性，需要外部包**明确引入** 












