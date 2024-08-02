# 库兼容

在介绍整体架构那个章节，我们大致看过了RUST在linux内核的库和普通程序有一些不同

本章我们讲尝试解析这些区别的由来

### 库的问题
无论如何，内核由于某种原因，可能需要对某些基础库进行修改，
但是这种修改，会引入上游兼容的问题: 

 - 上游软件库的版本更新，本地怎么同步？ 
 - 在同步时可能有冲突 
 - 本地版本的开发可能会和上游版本差异越来越大

内核对于上述问题给出了统一的解决建议: 

 - 本地版本修改，应该尽快合入上游分支
 - 上游分支应该提供对于内核的兼容机制
 - 最终，内核不应该保留外部库的副本，应该采用上游库


## 从core开始

`core crate`是RUST中第一个最底层的包，core在RUST中，总是会被作为一个预导入包导入，参考[预导入包](https://rustwiki.org/zh-CN/reference/names/preludes.html?highlight=core#%E5%A4%96%E9%83%A8%E9%A2%84%E5%AF%BC%E5%85%A5%E5%8C%85) 

`core crate` 设计的初衷就是把一些可以跨平台的、最基础的类型抽象，单独作为一个crate，从而达到跨平台使用的目的

### core兼容性

经过刚才的讨论，可以看到，`core crate` 从设计上讲，几乎不会存在兼容性问题,但是鉴于Linux内核的特殊性, 还存在
以下兼容性问题

 - core的库中提供了浮点数的计算(Linux内核中为了硬件兼容 一般不允许浮点计算)
 - i128/u128类型，在内核中也不应该使用

基于上述原因，rust 需要对core进行修改和兼容性适配，但是万幸，
我们可以看到，目前兼容问题只是`disable`掉部分功能，LINUX 采用了
一个比较hack的方案来解决 

 - Linux不需要修改core的源码，因此不需要在内核维护一个core的版本分支
 - Linux需要重新编译core 
 - Linux通过重定向core中内核不支持的函数为自己实现的函数，实现方法为`Makefile:redirect-intrinsics`
 - 重定向函数定义在: `linux/rust/compiler_builtins.rs` 
 - 重定向函数默认行为都为`panic`
 


## alloc
引用官方说明[alloc](https://rust-for-linux.com/unstable-features)

alloc 是 Rust 标准库的一部分，它的实现使用了许多不稳定的特性。通常情况下，
该库（以及 core 和其他库）由编译器提供，因此这些不稳定特性不会破坏用户的代码。

目前，内核包含一个 alloc 的分叉（与内核支持的 Rust 版本相匹配），并在上面添加了一些功能。
由于这些不稳定的特性，使用不同版本的编译器编译内核会变得复杂，但这个分叉只是暂时的：
与上游 Rust（及其他）讨论的 alloc 原始计划已在 rust/alloc/README.md 文件中记录。

目前正在与上游 Rust 公司进行讨论，以确定最终的分配解决方案。


总结: 
  - 当前内核迫于无奈 维护了一个alloc分支代码，因为现在必须要修改代码 才可以兼容内核(无法像core一样简单) 
  - 关于内核的兼容性修改，目前正在和RUST社区进行讨论
  - 未来会把内核的alloc分支给移除，alloc可以通过简单的配置即可在内核使用

### alloc兼容性
allock 关闭的功能:  
```
alloc-cfgs = \               
      --cfg no_borrow \        
      --cfg no_fmt \           
      --cfg no_global_oom_handling \
      --cfg no_macros \
      --cfg no_rc \
      --cfg no_str \
      --cfg no_string \
      --cfg no_sync \
      --cfg no_thin 
```



