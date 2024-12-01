## Monkey的进展

### 过去与现在

在《用 Go 编写解释器》一书中，我们构建了编程语言 Monkey 的解释器。Monkey 是为一个特定目的而创造的：在《用 Go 编写解释器》这本书中，由读者们从零开始中构建并实现。它的唯一官方实现包含在《用 Go 编写解释器》这本书中，尽管许多非官方的实现是由读者使用各种语言编写的，并在网络上传播。

如果你忘记了 Monkey 的样子，这里有一个小代码片段，它尽可能地将 Monkey 的特性压缩到了最少的行数内：

```javascript
let name = "Monkey";
let age = 1;
let inspirations = ["Scheme", "Lisp", "JavaScript", "Clojure"];
let book = {
    "title": "Writing A Compiler In Go",
    "author": "Thorsten Ball",
    "prequel": "Writing An Interpreter In Go"
};
let printBookName = fn(book) {
    let title = book["title"];
    let author = book["author"];
    puts(author + " - " + title);
};
printBookName(book);
// => prints: "Thorsten Ball - Writing A Compiler In Go"
let fibonacci = fn(x) {
    if (x == 0) {
        0
    } else {
        if (x == 1) {
            return 1;
        } else {
            fibonacci(x - 1) + fibonacci(x - 2);
        }
    }
};
let map = fn(arr, f) {
    let iter = fn(arr, accumulated) {
        if (len(arr) == 0) {
            accumulated
        } else {
            iter(rest(arr), push(accumulated, f(first(arr))));
        }
    };
    iter(arr, []);
};

let numbers = [1, 1 + 1, 4 - 1, 2 * 2, 2 + 3, 12 / 2];
map(numbers, fibonacci);
// => returns: [1, 1, 2, 3, 5, 8]
```

Monkey支持以下特性：
* 整数
* 布尔值
* 字符串
* 数组
* 哈希表
* 前缀、中缀和索引运算符
* 条件语句
* 全局和局部绑定
* 一等函数
* 返回语句
* 闭包

好长的一份清单，对吧？我们自己把这些都构建到了我们的 Monkey 解释器中，并且——最重要的是！——我们是从零开始构建的，没有使用任何第三方工具或库。

我们首先构建了词法分析器，它将输入到 REPL 中的字符串转换成记号。词法分析器定义在 lexer 包中，它生成的记号可以在 token 包中找到。

接下来，我们构建了解析器，一个自顶向下的递归下降解析器（通常称为Pratt解析器），它将这些记号转换为抽象语法树，简称 AST。 AST 的节点定义在 ast 包中，而解析器本身则位于 parser 包中。

经过解析器处理后，Monkey 程序就会以树的形式存储在内存中，下一步是对其进行求值。为此，我们构建了一个求值器。这是 evaluator 包中定义的一个名为 Eval 的函数的另一个称呼。Eval 会递归地遍历 AST 并对其进行求值，使用我们在object包中定义的对象系统来产生值。

例如，它会将表示 `1 + 2` 的AST节点转换成 `object.Integer{Value: 3}` .

这样，Monkey代码的生命周期就完成了，结果会被打印到REPL中。

这一系列的转换——从字符串到记号，从记号到树，再到 `object.Object` ——在整个我们构建的Monkey REPL 主循环中从头到尾都是可见的:

```javascript
// repl/repl.go

package repl

func Start( in io.Reader, out io.Writer) {
    scanner: = bufio.NewScanner( in )
    env: = object.NewEnvironment()
    for {
        fmt.Fprintf(out, PROMPT)
        scanned: = scanner.Scan()
        if !scanned {
            return
        }
        line: = scanner.Text()
        l: = lexer.New(line)
        p: = parser.New(l)
        program: = p.ParseProgram()
        if len(p.Errors()) != 0 {
            printParserErrors(out, p.Errors())
            continue
        }
        evaluated: = evaluator.Eval(program, env)
        if evaluated != nil {
            io.WriteString(out, evaluated.Inspect())
            io.WriteString(out, "\n")
        }
    }
}
```

这就是我们在上一本书结束时Monkey的状态。半年后，《遗失的章节：Monkey的宏系统》重新出现，向读者展示了如何让Monkey使用宏来自我编程。然而，在这本书中，《遗失的章节》及其宏系统将不会登场。事实上，情况就像是《遗失的章节》从未被发现过一样，我们回到了《用Go语言编写解释器》一书的结尾。这其实是一件好事，因为我们出色地实现了我们的解释器。

Monkey完全按照我们期望的方式工作，并且它的实现既容易理解又易于扩展。所以在第二本书开始时，一个自然的问题就浮现了：为什么要改变它？为什么不保持Monkey原样呢？

因为我们的目标是学习，而Monkey还有很多可以教给我们的东西。《用Go语言编写解释器》的一个目标是更深入地了解我们日常使用的编程语言的实现细节。确实，许多“现实世界”的语言最初都有与Monkey非常相似的实现方式。通过构建Monkey，我们能够更好地理解这些语言实现的基础和起源。

但是，随着语言的成长和发展，面对生产环境的工作负载以及对性能和语言特性的更高要求，语言的实现和架构通常会发生变化。这种变化的一个副作用是，语言的实现不再像Monkey那样简单，因为Monkey的设计并没有考虑到性能和实际生产使用。

成熟的语言与Monkey之间的差距是我们Monkey实现的最大缺点之一：它的架构与真实世界语言的架构相比，就像一辆肥皂箱赛车与一级方程式赛车之间的区别。虽然它有四个轮子和一个座位，可以帮助我们学习转向的基本原理，但缺少引擎这一事实很难忽视。

在这本书中，我们将缩小Monkey与真实语言之间的差距。我们要为我们的Monkey肥皂箱赛车装上真正的引擎。

### 未来

我们将把我们的遍历树结构并即时评估的解释器转变为一个字节码编译器和一个执行该字节码的虚拟机。这不仅非常有趣，而且也是目前最常用的解释器架构之一。Ruby、Lua、Python、Perl、Guile、不同的JavaScript实现以及许多其他编程语言都是以这种方式构建的。即使是强大的Java虚拟机也解释执行字节码。字节码编译器和虚拟机无处不在——这是有充分理由的。

除了提供一个新的抽象层——从编译器传递到虚拟机的字节码——使得系统更加模块化之外，这种架构的主要吸引力在于其性能。字节码解释器速度很快。想要具体数字吗？在本书结束时，我们将拥有一个比第一本书中的前一版本快三倍的Monkey实现。

```JavaScript
$. / monkey - fibonacci - engine = eval
engine = eval, result = 9227465, duration = 27.204277379 s
$. / monkey - fibonacci - engine = vm
engine = vm, result = 9227465, duration = 8.876222455 s
```

听起来不错！如果您准备好了开始编写代码，我们可以先确定您想要实现的具体项目或练习。

<div style="width: 100%; border: 1px solid #000; padding: 10px; display: flex; justify-content: space-between; ">
  <a href="./2引言.md" style="flex: 1; text-align: left; ">⬅ 引言</a>
  <a href="./4本书的使用.md" style="flex: 1; text-align: right; ">本书的使用 ➡</a>
</div>
