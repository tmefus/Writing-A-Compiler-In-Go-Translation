## Writing-A-Compiler-In-Go-Translation

### 感谢作者 Thorsten Ball

* 本书原版: [Writing A Compiler In Go](https://compilerbook.com/)

## 目录

01. **[致谢](./contents/01致谢.md)**
02. **[引言](./contents/02引言.md)**
    - **[Monkey的进展](./contents/03Monkey的进展.md)**
    - **[本书的使用](./contents/04本书的使用.md)**
03. **[编译器和虚拟机](./contents/05编译器和虚拟机.md)**
    - **[编译器](./contents/06编译器.md)**
    - **[虚拟机和真实机](./contents/07虚拟机和真实机.md)**
    - **[什么是虚拟机](./contents/08什么是虚拟机.md)**
    - **[为什么构建一个？](./contents/09为什么构建一个.md)**
    - **[字节码](./contents/10字节码.md)**
04. **[Hello 字节码](./contents/11hello字节码.md)**
    - **[第一个指令](./contents/12第一个指令.md)**
    - **[从字节开始](./contents/13从字节开始.md)**
    - **[最小的编译器](./contents/14最小的编译器.md)**
    - **[字节码反汇编](./contents/15字节码反汇编.md)**
    - **[回到手头的任务](./contents/16回到手头的任务.md)**
    - **[开机](./contents/17开机.md)**
    - **[在栈上添加元素](./contents/18在栈上添加元素.md)**
    - **[连接REPL](./contents/19连接REPL.md)**
05. **[编译表达式](./contents/20编译表达式.md)**
    - **[中缀表达式](./contents/21中缀表达式.md)**
    - **[布尔值](./contents/22布尔值.md)**
    - **[比较运算符](./contents/23比较运算符.md)**
    - **[前缀表达式](./contents/24前缀表达式.md)**
06. **[条件语句](./contents/25条件语句.md)**
    - **[跳转](./contents/26跳转.md)**
    - **[编译条件](./contents/27编译条件.md)**
    - **[执行跳转](./contents/28执行跳转.md)**
    - **[欢迎回来，Null！](./contents/29Null.md)**
07. **[符号追踪](./contents/30符号追踪.md)**
    - **[编译绑定](./contents/31编译绑定.md)**
    - **[向虚拟机添加全局变量](./contents/32向虚拟机添加全局变量.md)**
08. **[字符串、数组和散列](./contents/33字符串、数组和散列.md)**
    - **[数组](./contents/34数组.md)**
    - **[散列](./contents/35散列.md)**
    - **[添加索引操作符](./contents/36添加索引操作符.md)**
09. **[函数](./contents/37函数.md)**
    - **[编译函数字面量](./contents/38编译函数字面量.md)**
    - **[添加作用域](./contents/39添加作用域.md)**
    - **[编译函数调用](./contents/40编译函数调用.md)**
    - **[执行函数调用](./contents/41执行函数调用.md)**
    - **[局部绑定](./contents/42局部绑定.md)**
    - **[编译作用域](./contents/43编译作用域.md)**
    - **[在虚拟机中实现局部绑定](./contents/44在虚拟机中实现局部绑定.md)**
    - **[参数](./contents/45参数.md)**
    - **[解析对参数的引用](./contents/46解析对参数的引用.md)**
10. **[内置函数](./contents/47内置函数.md)**
    - **[实施更改：计划](./contents/48实施更改：计划.md)**
    - **[执行内置函数](./contents/49执行内置函数.md)**
11. **[闭包](./contents/50闭包.md)**
    - **[都是闭包](./contents/51都是闭包.md)**
    - **[编译和解析自由变量](./contents/52编译和解析自由变量.md)**
    - **[在运行时创建真正的闭包](./contents/53在运行时创建真正的闭包.md)**
    - **[递归闭包](./contents/54递归闭包.md)**
12. **[花点儿时间](./contents/55花点儿时间.md)**

**结束**

## 阅读以及PDF下载

* [在github上阅读本书](contents/01致谢.md)
* [英文版PDF下载](writing-a-compiler-in-go.pdf)
* **本书在翻译时没有严格按照原本的分章目录，而是为方便翻译有所调整。**
* **同系列前本的翻译版 \[[github/LixvYang](https://github.com/LixvYang)\] [Writing a Interpreter in Go](https://github.com/LixvYang/Writing-a-Interpreter-in-Go-Translation)**

## 免责声明

本人纯属兴趣翻译此书。本人承诺绝不以此译文以任何形式牟利。也坚决拒绝其他任何人以此牟利。本译文只做学习与交流用途。

**[@tmefus](https://github.com/tmefus)** 保留对译文的署名权以及其他权力，若有人以此译文进行侵权或者违反知识产权行为与本人无关。

## MIT License

Copyright (c) 2024 tmefus
