## 连接REPL

在我们继续之前，我们可以将编译器和虚拟机(VM)连接到我们的 REPL 上。这样，当我们想要尝试 Monkey 语言时，就可以获得即时反馈。我们只需要从 REPL 的 `Start` 函数中移除评估器和环境设置，并用我们已经从测试中熟悉的编译器和虚拟机的调用来替换它们：

```Go
// repl/repl.go

import(
    "bufio"
    "fmt"
    "io"
    "monkey/compiler"
    "monkey/lexer"
    "monkey/parser"
    "monkey/vm"
)

func Start(in io.Reader, out io.Writer) {
    scanner := bufio.NewScanner( in )
    for {
        fmt.Fprintf(out, PROMPT)
        scanned := scanner.Scan()
        if !scanned {
            return
        }
        line := scanner.Text()
        l := lexer.New(line)
        p := parser.New(l)
        program := p.ParseProgram()
        if len(p.Errors()) != 0 {
            printParserErrors(out, p.Errors())
            continue
        }
        comp := compiler.New()
        err := comp.Compile(program)
        if err != nil {
            fmt.Fprintf(out, "Woops! Compilation failed:\n %s\n", err)
            continue
        }
        machine := vm.New(comp.Bytecode())
        err = machine.Run()
        if err != nil {
            fmt.Fprintf(out, "Woops! Executing bytecode failed:\n %s\n", err)
            continue
        }
        stackTop := machine.StackTop()
        io.WriteString(out, stackTop.Inspect())
        io.WriteString(out, "\n")
    }
}
```

首先我们对输入进行标记化，然后我们解析它，接着我们编译并执行程序。我们还用打印虚拟机栈顶的对象来替换之前打印 Eval 的返回值的操作。

现在我们可以启动 REPL，看到我们的编译器和虚拟机在后台工作：

```
$ go build -o monkey . && ./monkey
Hello mrnugget! This is the Monkey programming language!
Feel free to type in commands
>> 1
1
>> 1 + 2
3
>> 1 + 2 + 3
6
>> 1000 + 555
1555
```

很好。但是，当然，一旦我们想要做的不仅仅是加两个数字，问题就来了：

```
>> 99 - 1
Woops! Compilation failed:
 unknown operator -
>> 80 / 2
Woops! Compilation failed:
 unknown operator /
```

我们仍然有工作要做。让我们开始吧。

|[⬅ 在栈上添加元素](./18在栈上添加元素.md)|[编译表达式 ➡](./20编译表达式.md)|
| --- | --- |
