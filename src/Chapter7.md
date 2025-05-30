# 类型、方法与接口（Types, Methods, and Interfaces）
如前几章所述，GoLang是静态类型语言，既包括基本类型，又包括自定义类型。Go允许你为**任何你定义的类型（包括基本类型的别名、结构体、甚至其他包的类型别名）** 定义方法，因为Go没有类（class）的概念，方法直接绑定到类型本身，而非通过类来组织，这提供了面向对象编程中“行为”与“数据”的关联能力，使代码组织更加清晰，任何类型都可以拥有其自己的方法。

Go同样满足类型抽象，即**接口（Interface）** 的实现，接口是Go实现**类型抽象**和**多态**的核心机制。接口只定义一组方法（方法名，参数列表，返回值列表），不包含任何实现代码或数据，它规定了**一个类型必须实现哪些方法才能满足该接口**。其中**隐式实现/鸭子类型**是Go接口最强大的特性之一，即一个类型**不需要显式声明**它实现了某个接口（如Java中的`implements`），只要**一个类型拥有接口所声明的所有方法**，那么它就**自动**满足并实现了该接口。是一种“如果它走起来像鸭子，叫起来像鸭子，那么它就是鸭子”的哲学。


> 静态类型语言(Statically type languages) vs 动态类型语言(Dynamically type languages)
> 
> 静态类型语言：C, C++, C#, Java, Go, Rust, Swift, Kotlin
>
> 动态类型语言：Python, JavaScript, Ruby, PHP, Perl, Lua
> 
> 1. 类型检查时机：
>   - 静态类型语言：在**编译阶段(Compile-time)** ，编译器进行类型检查
>   - 动态类型检查：在**运行阶段(Runtime)** ，解释器(或运行时环境)在程序执行过程中，当具体操作发生时（如赋值、函数调用等），才会检查涉及的类型
> 2. 变量与类型的绑定：
>   - 静态类型语言：变量在声明时需要**显式**指定类型（或使编译器能明确推断出变量类型），且变量的类型通常是**固定**的，编译器根据声明的类型分配内存 
>       - 如 Java： `int number = 10;` `String name ="Alice";` 
>   - 动态类型语言：变量在声明时 **不需要指定类型** ，变量的类型**由其当前所持有的值决定**，且可以在运行时随时改变，类型信息存储在值本身而非变量名
>       - 如 Python：  `x=42`  `x="hello"`  `x=3.14`
> 3. 趋势--界限在模糊：
> - 类型推断：很多现代静态类型语言（Go，Kotlin，Swift，C#`var`, C++`auto`）支持**类型推断**，编译器根据上下文自动推断类型变量，程序员无需显式写出所有类型，但类型仍然是静态确定且在编译器检查的。如 Go: `var a = 10`
> - 类型提示/注释：动态类型语言（如Python，JavaScript，PHP等）引入了**类型提示/注释** 的概念，这允许程序员在代码中（函数参数、返回值、变量）添加可选的类型信息。这些类型提示**不改变语言的动态特性**，在运行时解释器仍然进行动态类型检查，类型提示是给开发者和外部工具用的，而非对运行时引擎的强制约束。
> 
> | 方面 | 静态类型语言 | 动态类型语言|
> | ---- |  ----       | --------  |
> | 类型错误| 编译器发现错误，阻止错误程序运行| 运行时发现错误，导致程序运行崩溃|
> | 性能 | 通常更高，编译器提前知道类型，能进行大量优化（内联，特定指令等）| 通常较低，运行时需要不断检查和解析类型信息。但现代JIT(V8,PyPy)改善了性能|
> | 灵活程度| 较低，需要声明类型，重构时设计多处修改 | 较高，无需声明类型|
> | 运行时类型操作| 支持有限（主要通过反射机制）| 强大且自然，可以轻松检查、修改、创建类型（元编程）|

## 类型与 "type" 
在第61页中曾介绍过如何定义一个结构体类型：
```go
type Person struct{
    FirstName   string 
    LastName    string 
    Age         int
}
```
可以理解为通过`type`声明了一个名为Person的自定义类型，除此之外，还可以使用任何原始类型或复合类型来定义具体类型，比如
```go
type Score int
type Converter  func(string)Score
type TeamScores map[string]Score
```
Go 可以在包块package block下的任意块级别内声明变量，但作用域也仅限当前块内，唯一的例外是从其他包中导出的类型，在Chapter10中还会解释

## 方法（Methods）
Go 支持用户自定义类型的方法，但**方法只能定义在包块级别**
```go
type Person struct{
    FirstName string 
    LastName  string
    Age       int
}

func (p Person) String() string{
    return fmt.Sprintf("%s %s, age %d",p.FirstName, p.LastName, p.Age)
}

func main(){
    p := Person{
        FirstName: "Fred",
        LastName:  "Fredson",
        Age:       20,
    }
    output := p.String()
}
```
方法*Methods* vs 函数*Functions*
- 声明方式：方法的声需要接收器 *the receiver*，receiver 位于`func`与方法名称之间，接收器的名称一般是类型名的小写首字母，如`(p Person)`,`(a Animal)`，而非`(this Person)`或`(self Animal)`
- 声明位置：方法只能在包块被声明，而函数可以被声明在各种块中
- 重载：方法不能被重载，但是相同的方法名称可用于不同的变量，但相同的方法名不可用于同一类型的多个方法

在Chapter10 中还会讨论packages，其中，方法必须与类型在同一个包中声明，且最好是在同一文件中，便于跟踪实现。

### 指针接收器与值接收器 *(Pointer Receivers and Value Receivers)*
在Chapter6中提到过，指针类型可用于希望在函数中修改的变量，方法的接收器也遵循同样的规则。以下三条规则可以帮助更好理解两类接收器的选择：
- 方法修改接收器，必须使用指针接收器
- 如果需要处理 **`nil`实例**，必须使用指针接收器
- 方法不修改接收器，可以使用值接收器 
> **nil 及nil 实例**:
> 
> （1）Go中的nil表示一个指针、接口、切片、映射、通道或函数类型的**零值**，即它**没有指向任何有效的内存地址或数据结构**
> 
> （2）为什么值接收器无法"处理" `nil`: 当在nil指针上调用值接收器方法时，Go的行为是：**隐式地对`nil`指针进行解引用以获取一个零值**，例子如下：
> ```go
> type TheStruct struct{
>   Field int 
> }  
> 
> func (t TheStruct) ValueReceiver(){
>     fmt.Println(t.Field)
> }
> 
> var ptr *TheStruct = nil 
> ptr.ValueReceiver()
> ```
> Go 首先看到`ptr.ValueReceiver()`是一个值接收器方法，因此尝试执行`(*ptr).ValueReceiver()`，即先对`ptr`解引用以获取`TheStruct`的值；但ptr是`nil`!解引用一个`nil`指针会立即导致**运行时panic**(invalid memory address or nil pointer dereference)。
> 
> 关键在于: 值接收者方法根本没有机会"处理"`nil`，因为在方法体执行之前，解引用`nil`指针的操作就已经发生并导致panic了,nil 指针根本传递不进方法内部
>
> （3）为什么指针接收器可以处理`nil`实例： 因为指针接收器方法在内部接收到的参数就是nil，方法体有机会在执行任何操作之前检查`p==nil`,并采取不同的处理逻辑（比如记录日志、返回错误、执行默认行为、安全返回等），从而避免panic
>   
> （4）**处理`nil`实例**的含义：方法的逻辑预期到调用时可能会传递一个`nil`指针作为接收器，并且希望在方法中能够检测到这种情况并安全地执行下去，而不是panic。这种需求通常出现在某些API设计或特定模式中，允许`nil`有某种意义的行为，比如：
> - 树节点，`nil`子节点可能需要特殊处理
> - 配置对象指针，`nil`可能表示默认配置
> - 链表节点的`Next()`方法，在`nil`链尾上调用时可能返回`nil`或特定值

当使用值类型调用指针接收器的方法时，Go 会在调用方法时自动获取局部变量的地址，如`v.PtrMethod()` 会被转换为`(&v).PtrMethod()`；同样，指针调用值接收器方法时，Go 会自动将指针解引用，即`p.ValueMethod()` 与 `(*p).ValueMethod`，但如上文所述，nil调用值接收器方法会panic

但同时，向函数传递值的规则仍然适用。如果在值类型上调用指针接收者方法，则你实际上是在副本上调用该方法：
```go
type Counter struct{
    total int
    lastUpdated time.Time
}
func (c *Counter) Increment(){
    c.total++
    c.lastUpdated = time.Now()
}
func (c Counter) String() string {
    return fmt.Sprintf("total:%d, last updated: %v",c.total,c.lastUpdated)
}

func doUpdateWrong(c Counter){
    c.Increment()
    fmt.Println("in doUpdateWrong:",c.String())
}

func doUpdateRight(c *Counter){
    c.Increment()
    fmt.Println("in doUpdateRight:",c.String())
}

func main(){
    var c Counter
    doUpdateWrong(c)
    fmt.Println("in main:",c.String())
    doUpdateRight(&c)
    fmt.Println("in main:",c,String())
}
// in doUpdateWrong: total:1, last updated: 2025-05-30 17:57:32.879429595 +0800 CST m=+0.000049118
// in main: total:0, last updated: 0001-01-01 00:00:00 +0000 UTC
// in doUpdateRight: total:1, last updated: 2025-05-30 17:57:32.879640636 +0800 CST m=+0.000260157
// in main: total:1, last updated: 2025-05-30 17:57:32.879640636 +0800 CST m=+0.000260157
```

doUpdateRight中的参数`*Counter`是一个指针实例，你可以对其调用`Increment`和`String`方法。Go将指针和值接收器方法都视为指针变量可调用的方法的一部分。对于值变量，只有值接收器方法才包含在其**方法集（Method Set）**中。

这里提到的**方法集**：一个类型（或该类型的指针）的"方法集"是指所有可以合法地在该类型（或指针）的值上调用的方法的集合。方法集决定了哪些方法可以被调用，更重要的是，它决定了该类型（或指针）**实现了哪些接口**，因为接口要求其所有方法都在类型的方法集中


### Code Your Methods for nil Instances
### Methods Are Functions Too
### Functions Versus Methods
### Type Declarations Aren't Inheritance 
### Types Are Executable Documentation

## iota Is for Enumerations-Sometimes

## Use Embedding for Composition
## Embedding Is Not Inheritance 
## Accept Interfaces, Return Structs
## Interfaces and nil 
## Interfaces Are Comparable
## The Empty Interface Says Nothing 
## Type Assertions and TypeSwitches
## Use TypesAssertions and TypeSwitches Sparingly
## Function Types Are a Bridge to Interfaces
## Implicit Interfaces Make Dependency Injection Easier
## Wire
## Go Isn't Particularly Object-Oriented( and That's Great)
## Exercises
## Wrapping Up
