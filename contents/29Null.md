## 欢迎回来，Null！

在本章开始时，我们回顾了在《Writing An Interpreter In Go》中实现条件语句的方法，现在我们已经实现了大部分相同的行为。但还有一件事我们尚未解决：当条件语句的条件不是真值，并且该条件语句本身没有替代分支时会发生什么？在之前的书中，这个问题的答案是 `*object.Null` ，即 Monkey 的 `null` 值。

这合乎逻辑，因为条件语句是表达式，而表达式按定义会产生值。那么，一个未产生任何值的表达式会评估为什么呢？Null。等等，让我重新说一遍，这次想象背景中有宏大的管风琴声，头顶上有乌鸦飞过，尖叫着，雷电交加：它们……闪电劈下……评估为……警报声……null。

看，null 和我，我们并不是最好的朋友。我对它的好坏并不完全确定。它是许多诅咒的原因，但我确实理解有些语言中某些东西评估为“无”，而这个“无”必须以某种方式表示。在 Monkey 中，条件为假且没有替代分支的条件语句就是其中之一，“无”由 `*object.Null` 表示。长话短说：是时候把 `*object.Null` 引入我们的编译器和 VM，使这种类型的条件语句能够正常工作了。

我们需要做的第一件事是在我们的 VM 中定义 `*object.Null` 。由于它的值是常量，我们可以将其定义为全局变量，就像我们之前对 `vm.True` 和 `vm.False` 的定义一样：

```Go
// vm/vm.go

var Null = &object.Null{}
```

这与 `vm.True` 和 `vm.False` 类似，因为在比较 Monkey 对象时它可以为我们节省大量工作。我们只需检查一个 `object.Object` 是否等于 `vm.Null` ，而无需解开它查看其值。

我们之所以先定义 `vm.Null` ，而不是像往常那样先编写编译器测试，是因为这次我们想先编写一个 VM 测试。这是因为 VM 测试允许我们如此简洁地表达我们的需求：

```Go
// vm/vm_test.go

func TestConditionals(t *testing.T) {
    tests := []vmTestCase {
        // [...]
        {"if (1 > 2) { 10 }", Null},
        {"if (false) { 10 }", Null},
    }
    runVmTests(t, tests)
}

func testExpectedObject(t *testing.T, expected interface{}, actual object.Object) {
    t.Helper()
    switch expected := expected.(type) {
    // [...]
    case *object.Null:
        if actual != Null {
            t.Errorf("object is not Null: %T (%+v)", actual, actual)
        }
    }
}
```

这里我们为现有的 TestConditionals 函数添加了两个新的测试用例，其中条件不是 Monkey 的真值，以强制评估替代分支。但由于没有替代分支，我们期望 Null 最终出现在栈上。为了正确测试这一点，我们在 testExpectedObject 中扩展了一个新的 `*object.Null` 处理分支。

表达得很整洁，不是吗？好吧，错误信息却不是这样的：

```
$ go test ./vm
--- FAIL: TestConditionals (0.00s)
 panic: runtime error: index out of range [recovered]
 panic: runtime error: index out of range
 ...
FAIL    monkey/vm   0.012s
```

导致这次崩溃的原因是在条件语句之后我们发出了 OpPop 指令。由于这些条件语句没有生成任何值，VM 在尝试从栈中弹出数据时就会崩溃。现在是时候改变这一点了，把 `vm.Null` 放到栈上。

为了达到这个目的，我们要做两件事。首先，我们要定义一个操作码，告诉 VM 将 `vm.Null` 放到栈上。然后，我们要修改编译器，当条件语句没有替代分支时插入一个替代分支。而这个替代分支中唯一包含的就是新的操作码，它负责将 `vm.Null` 推送到栈上。

我们首先定义操作码，这样我们就可以在更新的编译器测试中使用它：

```Go
// code/code.go

const (
    // [...]
    OpNull
)

var definitions = map[Opcode]*Definition {
    // [...]
    OpNull: {"OpNull", []int{}},
}
```

这与布尔值的对应操作码 OpTrue 和 OpFalse 也类似。OpNull 没有任何操作数，仅指示 VM 将一个值推送到栈上。

现在，我们不是编写一个新的编译器测试，而是更新 TestConditionals 中的一个现有测试用例，期望在生成的指令中找到 OpNull。请注意，我们需要更改第一个测试用例，即条件语句没有替代分支的那个；其他测试用例保持不变：

```Go
// compiler/compiler_test.go

func TestConditionals(t *testing.T) {
    tests := []compilerTestCase {
        {
            input: `
            if (true) { 10 }; 3333;
            `,
            expectedConstants: []interface{}{10, 3333},
            expectedInstructions: []code.Instructions {
                // 0000
                code.Make(code.OpTrue),
                // 0001
                code.Make(code.OpJumpNotTruthy, 10),
                // 0004
                code.Make(code.OpConstant, 0),
                // 0007
                code.Make(code.OpJump, 11),
                // 0010
                code.Make(code.OpNull),
                // 0011
                code.Make(code.OpPop),
                // 0012
                code.Make(code.OpConstant, 1),
                // 0015
                code.Make(code.OpPop),
            },
        },
        // [...]
    }
    runCompilerTests(t, tests)
}
```

新增的是中间的两条指令：OpJump 和 OpNull。请记住，OpJump 存在是为了跳过替代分支，而现在 OpNull 就是那个替代分支。由于添加了这两条指令改变了现有指令的索引，因此 OpJumpNotTruthy 的操作数也需要从 `7` 更改为 `10` 。其余部分保持不变。

运行更新后的测试确认了编译器还没有学会自动为条件语句插入人工替代分支：

```
$ go test ./compiler
--- FAIL: TestConditionals (0.00s)
compiler_test.go:288: testInstructions failed: wrong instructions length.
want="0000 OpTrue\n0001 OpJumpNotTruthy 10\n0004 OpConstant 0\n\
0007 OpJump 11\n0010 OpNull\n\
0011 OpPop\n0012 OpConstant 1\n0015 OpPop\n"
got ="0000 OpTrue\n0001 OpJumpNotTruthy 7\n0004 OpConstant 0\n\
0007 OpPop\n0008 OpConstant 1\n0011 OpPop\n"
FAIL
FAIL    monkey/compiler 0.008s
```

修复这一点的最佳之处在于可以使我们编译器中的代码变得更简单、更容易理解。我们不再需要检查是否要发出 OpJump，因为现在我们总是想要这样做。只不过有时候我们想要跳过一个“真实的”替代分支，而有时候则是跳过一个 OpNull 指令。因此，这里是更新后的 Compile 方法中 `*ast.IfExpression` 分支的代码：

```Go
// compiler/compiler.go

func (c *Compiler) Compile(node ast.Node) error {
    switch node := node.(type) {
    // [...]
    case *ast.IfExpression:
        err := c.Compile(node.Condition)
        if err != nil {
            return err
        }
        // Emit an `OpJumpNotTruthy` with a bogus value
        jumpNotTruthyPos := c.emit(code.OpJumpNotTruthy, 9999)
        err = c.Compile(node.Consequence)
        if err != nil {
            return err
        }
        if c.lastInstructionIsPop() {
            c.removeLastPop()
        }
        // Emit an `OpJump` with a bogus value
        jumpPos := c.emit(code.OpJump, 9999)
        afterConsequencePos := len(c.instructions)
        c.changeOperand(jumpNotTruthyPos, afterConsequencePos)
        if node.Alternative == nil {
            c.emit(code.OpNull)
        } else {
            err := c.Compile(node.Alternative)
            if err != nil {
                return err
            }
            if c.lastInstructionIsPop() {
                c.removeLastPop()
            }
        }
        afterAlternativePos := len(c.instructions)
        c.changeOperand(jumpPos, afterAlternativePos)
        // [...]
    }
    // [...]
}
```

这是完整的分支，但只有其后半部分进行了更改：重复修补 OpJumpNotTruthy 指令的部分已经被移除，取而代之的是对可能存在的 `node.Alternative` 的新、可读的编译。

我们首先发出一个 OpJump 指令，并更新 OpJumpNotTruthy 指令的操作数。无论是否有 `node.Alternative` ，这一步都会执行。然后我们检查 `node.Alternative` 是否为 `nil` ，如果是，我们就发出新的 OpNull 操作码。如果不是 `nil` ，我们就按照之前的流程继续：编译 `node.Alternative` ，然后尝试移除一个可能存在的 OpPop 指令。

之后，我们将 OpJump 指令的操作数更改为跳过刚编译的替代分支——无论是只是一个 OpNull 还是更多内容。

这段代码不仅比之前的版本干净得多，而且也能正常工作：

```
$ go test ./compiler
ok      monkey/compiler 0.009s
```

现在我们可以转向我们的 VM，那里我们的测试仍然失败，我们需要在那里实现新的 OpNull 操作码：

```Go
// vm/vm.go

func (vm *VM) Run() error {
    // [...]
    switch op {
    // [...]
    case code.OpNull:
        err := vm.push(Null)
        if err != nil {
            return err
        }
        // [...]
    }
    // [...]
}
```

这样一来，崩溃问题解决了，我们的测试也通过了：

```
$ go test ./vm
ok      monkey/vm 0.009s
```

现在，我们可以转向 VM 的实现，那里我们的测试仍然失败，我们需要在那里实现新的 OpNull 操作码。下面是实现 OpNull 操作码的步骤：

```Go
// vm/vm.go

func (vm *VM) Run() error {
    // [...]
    switch op {
    // [...]
    case code.OpNull:
        err := vm.push(Null)
        if err != nil {
            return err
        }
        // [...]
    }
    // [...]
}
```

随着 OpNull 操作码的实现，崩溃问题得以解决，我们的测试也顺利通过了：

```
$ go test ./vm
ok      monkey/vm 0.009s
```

这意味着我们成功地使条件为非真值的条件语句能够在栈上放置 Null。我们现在实现了与《Writing An Interpreter In Go》中描述的条件语句相同的所有行为！

但是，很遗憾地说，还有最后一件事要做。随着最后一个测试的通过，我们正式进入了一个新的世界。由于条件语句是一个表达式，而表达式可以互换使用，因此可以推断出任何表达式现在都可以在我们的 VM 中生成 Null。这是一个令人畏惧的世界，没错。

对我们来说，实际影响是现在我们必须在处理表达式产生的值的每一个地方都处理 Null。幸运的是，我们 VM 中的大多数这些地方——比如 `vm.executeBinaryOperation` ——如果遇到未预期的值会抛出错误。但有一些函数和方法现在必须显式地处理 Null。

其中一个就是 `vm.executeBangOperator` 。我们可以添加一个测试来确保它在遇到 Null 时不会崩溃：

```Go
// vm/vm_test.go

func TestBooleanExpressions(t *testing.T) {
    tests := []vmTestCase {
        // [...]
        {"!(if (false) { 5; })", true},
    }
    runVmTests(t, tests)
}
```

通过这个测试用例，我们隐式地确保了条件为非真值且没有替代分支的条件语句结果为 Null，并且通过使用 ! 操作符对其求反后结果为 True。在底层，这涉及到 `vm.executeBangOperator` 方法，为了使测试通过，我们需要对其进行修改：

```Go
// vm/vm.go
func (vm *VM) executeBangOperator() error {
    operand := vm.pop()
    switch operand {
    case True:
        return vm.push(False)
    case False:
        return vm.push(True)
    case Null:
        return vm.push(True)
    default:
        return vm.push(False)
    }
}
```

现在，Null 的否定结果为 True——这与《Writing An Interpreter In Go》中的行为完全一致。测试已经通过：

```
$ go test ./vm
ok      monkey/vm 0.009s
```

这里来了奇怪的部分。既然条件语句本身是一个表达式，而其条件也是一个表达式，那么可以推断出我们可以将一个条件语句用作另一个条件语句的条件。虽然我相信你和我在实际编写代码时不会这样做，但无论如何，它在我们的 VM 中必须能够工作——即使内部的条件语句生成了 Null:

```Go
// vm/vm_test.go

func TestConditionals(t *testing.T) {
    tests := []vmTestCase{
        // [...]
        {"if ((if (false) { 10 })) { 10 } else { 20 }", 20},
    }
    runVmTests(t, tests)
}
```

这看起来可能是一团糟，但既然我们的代码非常干净且维护得很好，所以只需要在一个地方做出改变；这是一个相当明显的变化。我们需要告诉 VM， `*object.Null` 不是 isTruthy ：

```Go
// vm/vm.go

func isTruthy(obj object.Object) bool {
    switch obj := obj.(type) {
    case *object.Boolean:
        return obj.Value
    case *object.Null:
        return false
    default:
        return true
    }
}
```

这就是两行新的代码，我们就可以完成了：

```
$ go test ./vm
ok      monkey/vm 0.009s
```

现在完成就是真的完成了。我们的条件语句实现现在已经功能完整，并且我们有了一个 Null 安全的 VM：

```
$ go build -o monkey . && ./monkey
Hello mrnugget! This is the Monkey programming language!
Feel free to type in commands
>> if (false) { 10 }
null
```

是时候再次播放 "鼓掌声.wav" 了，这次我们知道没有遗漏任何东西。

|[⬅ 执行跳转](./28执行跳转.md)|[跟踪名称 ➡](./30跟踪名称.md)|
| --- | --- |
