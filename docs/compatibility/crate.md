# 库兼容

在介绍整体架构那个章节，我们大致看过了RUST在linux内核的库和普通程序有一些不同

本章我们讲尝试解析这些区别的由来

## core

在正式进入后续了解之前，先补充一些基础知识点(RUST课程中未涉及) 

以现内存参考 [RUST 参考文档](https://rustwiki.org/zh-CN/reference/conditional-compilation.html)

条件编译本质作用，就是为了可以做到选择性的编译某些代码，也就是说，该部分代码可能被编译在包里面，也可能没有被编译进去

C里面的条件编译，我们已经很熟悉了
```
if defined(xxxx) 

#endif
```

看一下RUST 中宏选项的几种基本形式 

```
#[cfg

```












