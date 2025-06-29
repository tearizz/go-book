# 类型、方法与接口（Types, Methods, and Interfaces）
如前几章所述，GoLang是静态类型语言，既包括基本类型，又包括自定义类型。Go允许你为**任何你定义的类型（包括基本类型的别名、结构体、甚至其他包的类型别名）** 定义方法，因为Go没有类（class）的概念，方法直接绑定到类型本身，而非通过类来组织，这提供了面向对象编程中“行为”与“数据”的关联能力，使代码组织更加清晰，任何类型都可以拥有其自己的方法。

Go同样满足类型抽象，即**接口（Interface）** 的实现，接口是Go实现**类型抽象**和**多态**的核心机制。接口只定义一组方法（方法名，参数列表，返回值列表），不包含任何实现代码或数据，它规定了**一个类型必须实现哪些方法才能满足该接口**。其中**隐式实现/鸭子类型**是Go接口最强大的特性之一，即一个类型**不需要显式声明**它实现了某个接口（如Java中的`implements`），只要**一个类型拥有接口所声明的所有方法**，那么它就**自动**满足并实现了该接口。是一种“如果它走起来像鸭子，叫起来像鸭子，那么它就是鸭子”的哲学。


> 静态类型语言(Statically type languages) vs 动态类型语言(Dynamically type languages)
> 
> 静态类型语言：C， C++， C#， Java， Go， Rust， Swift， Kotlin
>
> 动态类型语言：Python， JavaScript， Ruby， PHP， Perl， Lua
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
> - 类型推断：很多现代静态类型语言（Go，Kotlin，Swift，C#`var`， C++`auto`）支持**类型推断**，编译器根据上下文自动推断类型变量，程序员无需显式写出所有类型，但类型仍然是静态确定且在编译器检查的。如 Go: `var a = 10`
> - 类型提示/注释：动态类型语言（如Python，JavaScript，PHP等）引入了**类型提示/注释** 的概念，这允许程序员在代码中（函数参数、返回值、变量）添加可选的类型信息。这些类型提示**不改变语言的动态特性**，在运行时解释器仍然进行动态类型检查，类型提示是给开发者和外部工具用的，而非对运行时引擎的强制约束。
> 
> | 方面 | 静态类型语言 | 动态类型语言|
> | ---- |  ----       | --------  |
> | 类型错误| 编译器发现错误，阻止错误程序运行| 运行时发现错误，导致程序运行崩溃|
> | 性能 | 通常更高，编译器提前知道类型，能进行大量优化（内联，特定指令等）| 通常较低，运行时需要不断检查和解析类型信息。但现代JIT(V8，PyPy)改善了性能|
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
- 声明方式：方法的声需要接收器 *the receiver*，receiver 位于`func`与方法名称之间，接收器的名称一般是类型名的小写首字母，如`(p Person)`，`(a Animal)`，而非`(this Person)`或`(self Animal)`
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
> 关键在于: 值接收者方法根本没有机会"处理"`nil`，因为在方法体执行之前，解引用`nil`指针的操作就已经发生并导致panic了，nil 指针根本传递不进方法内部
>
> （3）为什么指针接收器可以处理`nil`实例： 因为指针接收器方法在内部接收到的参数就是nil，方法体有机会在执行任何操作之前检查`p==nil`，并采取不同的处理逻辑（比如记录日志、返回错误、执行默认行为、安全返回等），从而避免panic
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

doUpdateRight中的参数`*Counter`是一个指针实例，你可以对其调用`Increment`和`String`方法。Go将指针和值接收器方法都视为指针变量可调用的方法。对于值变量，只有值接收器方法才包含在其**方法集（Method Set）**中。

这里提到的**方法集**：一个类型（或该类型的指针）的"方法集"是指所有可以合法地在该类型（或指针）的值上调用的方法的集合。方法集决定了哪些方法可以被调用，更重要的是，它决定了该类型（或指针）**实现了哪些接口**，因为接口要求其所有方法都在类型的方法集中
> [关于Go方法集的探讨--Alexy Gronskiy](https://gronskiy.com/posts/2020-04-golang-pointer-vs-value-methods/)

别用`Getter()` 与 `Setter()`方法，Go 建议直接访问结构体中的属性，方法用于结构体的业务逻辑。但有两个例外情况：（1）将多个字段通过单个操作更新；（2）更新而非赋值；前面的`Increment`方法就体现了这个两个例外。

### nil 实例调用方法 Code Your Methods for nil Instances
如上文所述，nil调用值方法时，会报panic；而nil调用指针方法时，如果方法有正确处理nil，则可以正常执行；
```go
type IntTree struct{
    val         int 
    left, right *IntTree
}
func (it *IntTree) Insert(val int) *IntTree{
    if it == nil{
        return &IntTree{val:val}
    }
    if val < it.val{
        it.left = it.left.Insert(val)
    } else if val > it.val{
        it.right = it.right.Insert(val)
    }

    return it 
}

func (it *IntTree) Contains(val int) bool{
    switch {
        case it == nil:
            return false
        case val < it.val:
            return it.left.Contains(val)
        case val > it.val:
            return it.right.Contains(val)
        default:
            return true
    }
}
```

`Contains()`虽然没有修改`IntTree`，但是需要检查空指针，根据前文的规则，应该使用指针方法

虽然Go 调用 nil 接收者的方法非常巧妙，在某些情况下很有用，比如前面的树节点示例。然而，大多数情况下它用处不大。指针接收器的工作方式类似于指针函数参数；它是传递给方法的指针的副本*copy*。就像传递给函数的 nil 参数一样，如果你更改了指针的副本，你并**没有更改原始指针**。这意味着你不能编写一个指针接收者方法来处理 nil 并使原始指针变为非 nil。

但是如果方法使用指针作为接收器，且并非用于nil指针，此时有两个方法处理nil接收器：（1）将其视为fatal，不执行任何操作，在运行时panic，但需要确保[测试](Chapter15)；（2）如果空接收器时可以恢复的，需要检查是否为空并返回[错误](Chapter9)

### 方法代替函数 Methods Are Functions Too
Go 中的方法与函数非常相似，只要有函数类型的变量或参数，就可以使用方法来替代函数。
```go
type Addr struct{
    start int
}

func (a Addr) AddTo(val int) int {
    return a.start+val
}
```
此时可以用创建实例并调用方法
```go
myAddr := Addr{start:10}
fmt.Println(myAddr.AddTo(5)) // prints 15
```
也可以将方法赋值给一个变量，或者将其传递给`func(int)int`类型的参数。这被称为方法值*method value*。 方法值有点类似于闭包，可以访问到实例中的字段
```go
f1 := myAddr.AddTo
fmt.Println(f1(10)) // prints 20
```
还可以根据类型本身创建函数，叫做方法表达式*method expression*:
```go
f2 := Addr.AddTo
fmt.Println(f2(myaddr,20))
```
在方法表达式*method expression* 中，第一个参数是方法的**接收器**，函数签名*function signature*是`func(Adder,int)int`。
 
> 函数签名 Function Signature 定义了函数的“函数名、参数列表、返回值列表”
> 
> 即函数签名只用于描述函数，而不关心函数内部如何实现

后续在[Implicit Interfaces Make Dependency Injection Easier](p174)中还可以了解到更多方法值与方法表达式的用法

### 方法对比函数 Functions Versus Methods
虽然可以用方法代替函数，但具体何时使用方法，何时使用函数呢?


区别在于**你的函数是否依赖于其他数据**，正如前文提到的，包级别状态应该是不可变的。
- 如果你的逻辑依赖于**启动时配置的值或在程序运行时更改的值**，那么这些值应该存储在结构体中，并且通过**方法**实现。
- 如果逻辑只依赖于**输入参数**，那么它应该是一个**函数**。

类型声明并非继承
除了基于 Go 内置类型和结构体字面量声明类型之外，你还可以基于另一个用户定义类型声明一个用户定义类型：
类型 HighScore Score
类型 Employee Person
许多概念都可以被认为是“面向对象”的，但有一个概念尤为突出：继承。通过继承，父类型的状态和方法被声明为在子类型上可用，并且子类型的值可以替换父类型的值。


### 类型的声明不是"继承" Type Declarations Aren't Inheritance 
Go除了内置类型与结构体的声明之外，还可以基于一个用户类型声明另一个用户类型，比如:
```go
type HighScore Score
type Employee  Person
```
基于一个类型声明另一个类型，看似是继承，但实际上并不是。两种类型仅仅是底层类型相同，没有其他的层次结构。在支持继承的语言中，子实例可以在任何使用父实例的地方使用，子实例还拥有父实例的所有方法与数据结构，但在Golang中并非如此。如果不进行类型转换，就无法将`HighScore`类型的实例赋值给`int`类型的变量。同时，此处涉及到Go 类型系统中的一个核心概念：

**类型标识Type Identity** 与 **方法集 Method Set** 的分离

（1）关于共享底层类型 *（Share an Underlying Type）*
：Go 允许基于现有类型定义新的命名类型，比如
```go
type Celsius float64
type Fahrenheit float64
```
- `Celsius` 与 `Fahrenheit` 都是不同的类型，但共享相同的底层类型 *Underlying Type*
- 底层类型在决定了值在内存中如何表示（占多少字节、如何解释这些字节）

（2）关于类型转换 *（Type Conversion）*
- 因为`Celsius` 与 `Fahrenheit`共享底层`float64`，Go允许二者间进行**显式**类型转换
```go
var c Celsius = 100.0
f := Fahrenheit(c)
```
- 这种类型转换是合法的，因为编译器知道两种类型在内存中的表示方式完全相同（都是`float64`）

（3）**保持相同的底层存储 *（Keeps the Same Underlying Storage）***

当执行`Fahrenheit(c)`时，不创建新的内存，不复制`c`的值，只是**编译器改变了对内存中现有值（100.0）的类型解释方式**
- 变量`c`所在的内存地址存储的二进制位模式（表示100.0的IEEE754格式）**完全没有改变**。转换后的`f`与`c`**指向同一块内存**，或者更精确地说，`f`的值就是`c`的值在内存的表示，只是现在用`Fahrenheit`类型去解读它，因此转换是零成本的（没有运行时开销）。另一个例子是：
```go
	var i int = 10
	var s Score = 20
	var hs HighScore = 30
	// hs = s	        // cannot use i (type int) as type Score in assignment
	// s = i	        // s = hs // cannot use hs (type HighScore) as type Score in assignment
	s = Score(i)        // explicit conversion
	hs = HighScore(s)   // explicit conversion
```

（4）**关联不同的方法 *（Associates Different Methods）***

由于方法是绑定在特定的命名类型上的，虽然Celsius 和 Fahrenheit 共享底层`float64`，但他们是完全独立的类型，可以为各自定义独立的方法集。
```go
func (c Celsius) ToFahrenheit() Fahrenheit {
    return Fahrenheit((c * 9 / 5) + 32)
}

func (f Fahrenheit) ToCelsius() Celsius {
    return Celsius((f - 32) * 5 / 9)
}

func (c Celsius) String() string {
    return fmt.Sprintf("%.2f°C", c)
}

func (f Fahrenheit) String() string {
    return fmt.Sprintf("%.2f°F", f)
}
```
所以当一个Celsius的值显式转换为 Fahrenheit 时：
```go
c := Celsius(100.0)
f := Fahrenheit(c)
```
变量`c`的类型是`Celsius`，拥有`ToFahrenheit()`与`String()`方法；
变量`f`的类型是`Fahrenheit`，拥有`ToCelsius()`与`String()`方法

即 **转换操作值改变了值的类型标签，并没有改变值本身。但这个新的类型标签决定了现在可以调用哪些方法**。转换后的f不再拥有Clesius类型的方法`ToFahrenheit()`，反而拥有了`Fahrenheit`类型的方法`ToCelsius`，即使在内存中表现相同，编译器也会根据变量的当前类型来决定哪些方法可用。

在下述例子中，s虽然是Score类型，但底层存储的仍是整数值`50`,而无类型常量的隐式转换满足将`100`转换为`Score`类型（因为操作数`s`是`Score`类型），同时当命名类型与无类型常量运算时，结果保持命名类型，下述的表达式等价于`s+Score(100)`；但是，当两个不同命名类型运算，即使底层类型相同也会报错，比如`Score+HighScore`，会报错类型不匹配，这两个例子就很好的体现了Go类型系统：**通过命名类型提供抽象和安全，同时利用底层类型和无类型常量保持灵活性**
```go
var s Score = 50
scoreWithBounds := s + 100  // type of scoreWithBouns is Score

var hs HighScore = 100
scoreTotal := s + hs        // invalid operation: s + hs (mismatched types Score and HighScore) 
```

### 类型即可执行的“文档” *Types Are Executable Documentation*
类型可视作代码的“自文档化”工具，比如`type Percentage int` 比单纯`int`更能明确表达“百分比”的概念。
同时，`Calculation(p Percentage)` 作为函数签名能够清晰地表达参数含义。此外，编译器还能阻止传递普通`int`，减少`CalculateDiscount(150)`这类错误

## "iota" 有时用于枚举 *iota Is for Enumerations-Sometimes*
Go 虽然没有枚举类型*enumeration type*，但可以通过`iota`机制，为**一组常量赋一个递增的值**。
> iota 的概念源自编程语言 APL（A Programming Language，即“一种编程语言”）。在 APL 中，要生成包含前三个正整数的列表，可以写成 ι3，其中 ι 是小写希腊字母 iota
> 
> APL 因其对自身自定义符号的依赖而闻名，以至于它需要配备特殊键盘的计算机。例如，(~R∊R∘.×R)/R←1↓ιR 是一个 APL 程序，用于查找所有不超过变量 R 值的素数
> 
> 像 Go 这样注重可读性的语言，竟然会从一种简洁到极致的语言中借用一个概念，这似乎有些讽刺，但这正是你应该学习多种编程语言的原因：你可以在任何地方找到灵感。

使用`iota`时，最好定义基于`int`的类型，可以表示所有有效值
```go
type MailCategory int
```
接下来，使用`const`为该类型定义一组值:
```go
const(
    Uncategorized MailCategory = iota
    Personal
    Spam
    Social
    Advertisements
)
```
`const block`中的第一个常量指定了类型`MailCategory`,并设置为`iota`，后续的每行均无需指定类型和赋值。Go 编译器会重复指定类型，并将iota赋给块中所有后续常量。`iota`的值会随着`const`中的常量递增，从0开始，即第一个常量为0，第二个为1，第三个为2。。。当创建新的`const-iota`时，`iota`会被重新设置为`0`

同时，`iota`无论是否用于定义常量的值，其在`const block`中都会从第一项开始递增，如下例所示：
```go
const (
	f1 = 0
	f2 = 1 + iota
	f3 = 30
	f4
	f5 = iota
)

func main() {
	fmt.Println(f1, f2, f3, f4, f5) // 0 2 30 30 4
}
```
f2的值为2是因为iota在第二行的值为1，f4的值为20是因为其没有被特殊设置，因此与上一个之相同。最后f5为4，是因为在const第5行，从0开始递增的值就是4，同理，可以设置带有偏移量的`iota`
```go
const(
    _ Status = iota //保留0作为无效状态

    // From 100
    StatusA = iota+99   // 100
    StatusB             // 101
    StatusC             // 102
)
```

关于`iota`“最好”的建议：
> Don’t use iota for defining constants where its values are explicitly defined (elsewhere).For example, when implementing parts of a specification and the specification says which values are assigned to which constants, you should explicitly write the constant values. Use iota for “internal” purposes only. That is, where the constants are referred to by name rather than by value. That way, you can optimally enjoy iota by inserting
new constants at any moment in time / location in the list without the risk of breaking everything. --- [Danny van Heumen](https://www.dannyvanheumen.nl/post/when-to-use-go-iota/)

Danny所述的本质是：**「控制常量值的稳定性边界」**，在保证外部兼容性的前提下，最大化内部代码的自由。

大意是在**值由外部定义时，千万不要使用`iota`**（如http的返回值，200、201、202、404等，或错地址、规范、错误码等）；
只有**纯内部使用时，可以使用`iota`** 以享受便利，即插入新常量不影响现有逻辑，比如前文的`MailCategory`。

必须使用`iota`时的安全插入策略：
```go
const(
    _ = iota    // 跳过0
    Start       // 1

    // 预留空间
    _
    _

    Middle      // 4
    End         // 5
)

// 后续插入时
const(
    _ = iota    // 跳过0
    Start       // 1

    // 预留空间
    NewPoint    // 2
    _

    Middle      // 4
    End         // 5
)
```

同时还可以将**字面量表达式**通过`iota`赋值给常量：
```go
type BitField int
const(
    Field1 BitField = 1 << iota // 1*2^0 = 1 
    Field2                      // 1*2^1 = 2
    Field3                      // 1*2^2 = 4
    Field4                      // 1*2^3 = 8
)
```
虽然很帅，但还是需要慎用。同时注意iota是从0开始的，如果不方便赋值，可以将0设为状态值，如前文的`Status`

## 以“嵌入”的方式“组合” *（Use Embedding for Composition）*
软件工程中“对象的组合优于继承”的说法可以追溯到1994年由Erich Gamma, Richard Helm, Ralph Johnson与John Vlissides “四人组”的著作 ***Design Patterns*** 。虽然Go 不支持继承，但是可以通过组合以实现代码的重用
```go
type Empolyee struct{
    Name    string 
    ID      string
}

func (e Employee) Description() string{
    return fmt.Sprintf("%s(%s)",e.Name,e.ID)
}

type Manager struct{
    Employee
    Reports []Employee
}

func(m Manager) FindNewEmployees() []Employee{
}
```
`Manager`虽然没有为包含`Employee`类型的字段赋值，但已经成功潜入了`Employee`类型，`Employee`的任何字段与方法都可以在`manager`变量上直接调用，如：
```go
m := Manager{
    Employee: Employee{
        Name: "Bob",
        ID:   "12345",
    },
    Reports: []Employee{},
}
fmt.Println(m.ID)   // 12345
fmt.Println(m.Description())    // Bob(12345)
```
如果嵌入的变量名或类型方法重名，则在调用时需具体指出具体嵌入的字段名，比如：
```go
type Inner struct{
    X int
}
type Outer struct{
    Inner 
    X int
}
o := Outer{
    Inner: Inner{
        X:10,
    }
    X: 20,
}
fmt.Println(o.X)        // 20
fmt.Println(o.Inner.X)  // 10
```

## 嵌入不是继承！Embedding Is Not Inheritance 
嵌入似乎是Go 特有的，但绝不是Go用嵌入代替继承，嵌入/组合与继承是完完全全两个领域的概念。嵌入是语法糖层面的组合，不是类型系统层面的继承，被嵌入类型与容器类型毫无关联。同时，Go要求使用显式路径访问被嵌入的字段，以避免多重嵌入时的命名冲突、隐式行为导致的歧义、破坏代码的可行性等。Go的嵌入机制绝非“is-a”的继承思维，而是“has-a”的组合思维！

```go
func (i Inner) IntPrinter(val int) string {
    return fmt.Sprintf("Inner: %d", val)
}
func (i Inner) Double() string{
    return i.IntPrinter(i.X *2)
}
func (o Outer) IntPrinter(val int) string {
    return fmt.Sprintf("Outer: %d",o.X)
}
o := Outer{
    Inner:Inner{
        X:10,
    },
    X:30
}
fmt.Println(o.Double())         // Inner: 20
fmt.Println(o.IntPrinter(o.X))  // Outer: 30
```
虽然将一个具体类型嵌入到另一个具体类型中并不能将外部类型视为内部类型，但嵌入字段的**方法**确实会被计入外部结构体的方法集，这意味着外部结构体可以满足接口，例子如下：
```go
type SpeakIf interface {
	speak()
}

type Name string

type people struct {
	Name
	Age int
}

func speak(s SpeakIf) {
	s.speak()
}
func (n Name) speak() {
	fmt.Printf("My name is %s\n", n)
}

func main() {
	p := people{
		Name: Name("John"),
		Age:  26,
	}
	speak(p)
}
```

## A Quick Lesson on Interfaces
虽然Golang的[并发模型](chapter12)设计十分优秀，但Go设计的真正亮点在于他的 **隐式接口 *Implicit Interface*** ，这是Go中唯一的抽象类型。

对于接口的声明同样使用关键字`type`，那么目前已知`type`可用于声明结构体、自定义类型与接口。接口内的方法构成了接口的方法集。同时，如前文所述，指针实例的方法集包括指针接收器与值接收器方法，而变量实例的方法集只包括值接收器方法：
```go
type Stringer interface{
    String()    string 
}
type Incrementer interface{
    Increment()
}

type Counter struct{
    total int
    lastUpdated time.Time
}

func (c *Counter) Increment() {
    c.total++
    c.lastUpdated = time.Now()
} 

func (c Counter) String() string {
    return fmt.Sprintf("total: %d, last updated: %v",c.total,c.lastUpdated)
}

var myStringer fmt.Stringer
var myIncrementer Incrementer

pointerCounter := &Conunter{}
valueCounter := Counter{}

myStringer = pointerCounter     // ok
myStringer = valueCounter       // ok 
myIncrementer = pointerCounter  // ok 
myIncrementer = valueCounter    // cannot use valueCounter (variable of type Counter) as Incrementer value in assignment: Counter does not implement Incrementer (method Increment has pointer receiver)
```
与其他类型相同，接口也可以被声明在任何级别的块中，接口通常以"-er"结尾，比如`fmt.Stringer`、`io.Reader`、`io.Closer`、`io.ReadCloser`、`json.Marshaler`、`http.Handler`等。

## 接口时类型安全与鸭子类型 *Interfaces Are Type-Safe Duck Typing*
目前为止，Go 语言的接口机制与其他语言相比并无很大区别，真正不同的是Go语言接口的 **隐式实现 *implemented implicitly***，上一节中的`Counter`结构体并没有显式地声明实现了`Incrementer`接口，即在Go中：*一个具体的类型实现了接口中的全部方法，就认为该类型实现了接口*。

这种隐式行为使得接口成为 Go 语言类型系统中最精妙的设计，因为它同时实现了**类型安全与解耦**，既保留了静态语言的类型检查安全性，又结合了动态语言灵活适配类型的灵活性。
- 类型安全： Go 是静态类型语言，编译器在编译过程中进行类型检查，当你将一个具体类型(Counter)赋值给接口类型(Incrementer)时，编译器会严格检查这个类型是否真的包含了接口要求的所有方法。 若没有满足全部方法，编译器会立即报错，这保证了在运行时通过接口变量调用的方法是一定存在的，并且参数与返回值类型均匹配，避免了动态语言中常见的“方法未找到”或“类型错误”的运行时异常，这就是编译时的**类型安全**
- 解耦： 接口定义了的是契约，即一组方法签名，声明了“需要什么能力”，而非“如何提供能力”或“由谁提供这个能力”。这种“按行为匹配类型”的方式非常灵活，允许你编写通用的代码来处理不同的类型，只要它们行为一致（方法相同）。你不需要复杂的继承体系或提前声明类型关系。

前文提到过 *Design Patterns* 中“优先使用组合而非继承”的观点，他的另一个观点是 **“针对接口编程，而非针对实现”**。这样做可以依赖于行为，而非具体的实现动作，可以根据行为切换实现，使带个能够根据需求不可避免的变化而不断演进。

像Python，Ruby与JavaScript等动态类型语言并没有接口机制，但他们可以使用“鸭子类型” *duck typing*，即“如果它走起来像鸭子，叫起来像鸭子，那么它就是鸭子”。其概念是，将类型的实例作为参数传递给函数，只要该函数可以找到它期望调用的方法：
```py
class Logic:
    def process(self,data):
        # business logic
def program(logic):
    # get data from somewhere
    logic.process(data)
    
logicToUse = Logic()
program(logicToUse)
```
鸭子类型乍一听可能有点奇怪，但它已被用于构建大型且成功的系统。如果你使用静态类型语言编程，这听起来会非常混乱。如果没有明确指定类型，就很难确切知道应该实现哪些功能。当新开发人员加入项目，或者现有开发人员忘记代码的功能时，他们必须追溯代码才能确定实际的依赖关系。

Java 开发者使用不同的模式。他们定义一个接口，创建该接口的实现，但只在客户端代码中引用该接口：
```java
public interface Logic {
    String process(String data);
}
public class LogicImpl implements Logic {
    public String process(String data){
        // business logic
    }
}

public class Client {
    private final Logic logic;

    public Client(Logic logic) {
        this.logic = logic
    }

    public void program() {
        // get data from somewhere
        this.logic.process(data);
    }
}

public static void main(String[] args){
    Logic logic = new LogicImpl();
    Client client = new Client(logic);
    client.program();
}
```
Go 的开发者认为这两组人都是对的。如果你的应用程序会随着时间的推移而增长和变化，你需要灵活地更改实现。然而，为了让人们理解你的代码在做什么（因为随着时间的推移，会有新的人使用相同的代码），你还需要指定代码依赖的内容。这就是隐式接口的用武之地。Go 代码是前两种风格的融合。
```go
type LogicProvider struct{}

func (lp LogicProvider) Process(data string) string{
    //business logic
}

type Logic interface{
    Process(data string) string 
}

type Client struct{
    L Logic
}

func (c Client) Program(){
    // get data from somewhere
    c.L.Process(data)
}

func main(){
    c := Client{
        L: LogicProvider{},
    }
    c.Program()
}
```
Go 代码提供的`Logic` 接口，只有调用者（`client`）了解，虽然`LogicProvider`上没有任何声明，但他允许创建一个新的logic provider，同时保证任何传入客户端的类型都满足client的需求 ==> **接口指定调用者需要什么，客户端通过接口指定所需功能**

Go 的标准库中有些用于输入输出的接口，使用标准接口最好使用**装饰器模式**。在Go 中，***编写工厂函数接收一个接口的实例，并返回另一个实现相同接口的类型使很常见的***，比如：
```go
func process(r io.Reader) error

func getError() error {
    r,err := os.Open(fileName)
    if err != nil{
        return err 
    }
    defer r.Close()

    return process(r)
}
```
这个例子中，`op.Open`返回值满足`io.Reader`的接口，并可以读取任何代码中的data，如果文件值gzip压缩的，你可以在一个`io.Reader`中包装另一个`io.Reader`
```go
r,err := os.Open(fileName)
if err != nil{
    return err
}
defer r.CLose()

gz,err := gzip.NewReader(r)
if err != nil{
    return err 
}
defer gz.Close()

return process(gz)
```
上述代码实现了从未压缩文件中读取的代码，再从压缩文件中读取一遍，`os.Open()`的返回值与`gzip.NewReader()`的入参均满足`io.Reader`的接口。如果标准库中的接口描述了你的代码所需的功能，**尽情使用它吧！** 常用的接口包括 io.Reader、io.Writer 和 io.Closer。

对于满足接口的类型来说，指定不属于该接口的额外方法是完全没问题的。一部分客户端代码可能不关心这些方法，但其他代码则关心。例如，io.File 类型也满足 io.Writer 接口。如果您的代码只关心从文件中读取数据，请使用 io.Reader 接口来引用文件实例，并忽略其他方法。haoel分享过一个接口的**完整性检查**方法：将变量的空指针赋值给接口，若变量已经满足接口则不会报错，若不满足则在编译时就会报错。
```go
var _ Interface = (*StructType)(nil)
```
##  嵌入与接口 Embedding and Interfaces
不仅结构体可以相互嵌入，接口同样可以相互嵌入，比如`io.ReadCloser`接口就基于`io.Reader`接口与`io.Closer`接口
```go
type Reader interface{
    Read(p []byte) (n int,err error)
}

type Closer interface {
    Closer() error
}

type ReadCloser interface{
    Reader
    Closer
}
```
接口甚至还可以嵌入到结构体中，具体见[Using Stubs in Go](Page397)

## 接口作参数，结构体作返回值 *Accept Interfaces, Return Structs*
"Accept interfaces, return structs"这句话最早出自于Jack Lindamood在2016年的博客[Premptive Interface Anti-Pattern in Go](https://medium.com/@cep21/preemptive-interface-anti-pattern-in-go-54c18ac0668a)
。他的大致意思是**依赖抽象接口进行调用，但返回具体的实现**。
- 函数内部调用的业务逻辑应该通过接口进行：当函数内部需要调用其他组件（如服务、存储、外部API等）来完成工作时，不应该直接以来具体的实现类型，而是应该依赖这些组件所实现的接口
    - 函数只需要关心接口定义的行为，而无需关心具体哪个结构体实现或如何实现。这使得可以轻松更换底层实现（就像`cryptoalgo`可以是`ecdsaImpl`实现也可以是`rsaImpl`实现）
    - 在测试时，可以为这些接口创建`Mock`或`Stub`实现，从而隔离被测函数，只测试自身逻辑。 **Mock，Stub？？**
    ```go
    // 依赖接口 UserRepository，而不是具体的 *MySQLUserReporisitory 或 *InMemoryUserRepository
    func GetUserByID(repo UserRepository, id int) (*User, error) {
        // 业务通过接口调用
        return repo.GetByID(id)
    }

    type UserRepository interface {
        GetByID(id int) (*User, error)
        Save(user *User) (error)
        // ... other methods
    }
    ```
- 函数的输出应该是一个具体类型：当函数需要返回数据或结果时，应该返回的具体的结构体类型或基础类型，而非接口
    - 调用者只需要依赖函数返回的具体类型，不需要满足这个类型实现的接口定义，如果返回接口，那么所有的调用者都必须满足接口，即便可能只需要具体结构体中的某个字段
    - 避免接口污染，不需要为每个返回的对象都定义一个专门的接口，让API更清晰、直接，减少了代码的抽象和复杂性
    ```go
    type User struct{
        ID      int
        Name    string
        Email   string
    }

    func CreateUser(repo UserRepository, name string, email string) (*User, error) {
        newUser := &User{Name: name, Email: email}
        err := repo.Save(newUser)
        if err != nil{
            return nil,err
        }
        return newUser, nil
    }
    ```
*Accept Interfaces, Return Structs* 是Go 社区广泛认可的最佳实践，它平衡了抽象和解耦的需求（通过接口参数）与 API 的清晰性和使用便利性（通过返回具体类型）。同时充分利用了 Go 隐式接口的特性：调用方可以轻松地将函数返回的具体类型赋值给自己需要的接口（如果类型匹配），而函数本身不需要关心或定义这些接口。它还让模块间的边界更清晰：函数声明了它需要的外部依赖（接口入参），并提供了明确的产出（具体类型返回值）。

但是在极少数情况下，*接口中的方法不得已返回另一个接口*，比如标准库中`database/sql/driver`包定义的一组公共接口（如`Driver`、`Conn`、`Stmt`、`Result`、`Rows`等）中的方法，其返回值通常是其他接口，这种设计主要是为了：
- 解耦具体实现：数据库驱动开发者只需要实现接口，而不需要依赖具体的实现类型
- 隐藏底层细节：调用方（如`database/sql`包）仅通过接口与驱动交互，无需关心驱动内部的具体类型
> 数据库驱动开发者：
> 指的是为特定数据库系统（如MySQL、PostgreSQL、SQLite、Oracle 等）编写Go语言连接驱动程序的开发者或团队，他们是连接GO 标准数据库抽象层（database/sql）和具体数据库系统（MySQL、PG, etc.）的桥梁，通过精确实现`databse/sql/driver`定义的接口，是的用户可以使用统一的标准库API操作不同的数据库。他们虽然不是标准库的核心团队，但是作为第三方开发者，维护如`github.com/go-sql-driver/mysql`或`github.com/lib/pq`这些流行的数据库驱动库。

但是，Go在后续发展（1.8）中，需要为数据库驱动增加新功能，但为了保证**向后兼容**，既不能修改现有接口，又不能修改现有方法，此时唯一的解决方案就是：
1. 保留旧接口，不修改原有接口及其方法（如`driver.Conn`接口）
2. 新增接口，在新接口中实现新功能的方法（如`driver.ConnPrepareContext`接口）
3. 驱动作者同时实现新旧接口，具体类型需同时实现新旧接口（如`driver.Conn`与`driver.ConnPrepareContext`接口）
```go
// Go1.0 旧接口
type Conn interface {
    Prepare(query string) (Stmt, error)
}
// Go1.8 新增接口
type ConnPrepareContext interface {
    PrepareContext(ctx context.Context, query string) (Stmt,error)
}

// 驱动实现
type MyConn struct {}
func (m *MyConn) Prepare(query string) (Stmt, error) {/**/}
func (m *MyConn) PrepareContext(ctx context.Context, query string) (Stmt, error) {/**/}
```

这就引出了一个问题：如何检查这些新方法是否存在？以及如果存在，如何访问它们？在[Type Assertions and Type Switches](P167)中又说明

> deepseek 偷学的**类型断言检查**：
> ```go
> if connCtx, ok := conn.(driver.ConnPrepareContext);ok{
>   // 使用新接口    
> } else{
>   // 回退到旧接口    
> }
> ```
> 既满足了向后兼容，使旧驱动无须修改，又可以通过扩展而非修改实现新功能；
> 
> 就是可能驱动开发者需要实现更多接口，且调用者需处理类型断言和回退逻辑

**工厂函数的返回值选择：**

工厂设计模式中，在设计工厂函数时，为每种具体类型提供独立的工厂函数（如`NewFileLogger() *FileLogger`和 `NewDBLogger() *DBLogger`），而非统一返回Logger接口的`NewLogger(config) Logger`，避免编写一个返回接口类型的“万能工厂”（根据参数返回多种具体类型的接口）。

但例外情况是：当**函数必须返回多种无法预知的异构类型**（如解析器返回多种Token类型）时，且这些类型**天然符合同意接口**时，返回接口是**唯一的可行方案**，如：
```go
type Token interface {
    Type() string 
}
type NumberToken struct{}
type StringToken struct{}

func ParseToken(input string) Token {
    if isNumber(input){
        return &NumberToken{/**/}
    }
    return &StringToken{/**/}
}
```

**错误处理的例外规则：**

[`error`](chap9) 同样是这个返回值规则的一个例外，Go 强制规定**错误必须通过`error`接口类型返回**。因为错误的来源十分多样（文件 I/O 错误、网络错误、业务逻辑错误等），每种错误由不同具体类型实现（如 `os.PathError`, `net.OpError`, 自定义错误）。而接口时Go唯一支持的类型抽象机制，只有返回`error`接口才能统一处理所有错误类型


**接口参数的性能权衡：**

这种模式有一个潜在的缺点，如[Reducing the Garbage Collector's Workload](P136)中提到的：减少堆分配可以通过减少垃圾收集器的工作量来提高性能。虽然返回结构体可以避免堆分配，但是以接口作为参数的函数在被调用时，每个接口都会触发一次堆分配，这会影响程序的性能。因此在程序的整个生命周期中，一定要权衡好性能与可读/可维护性。如果因为接口参数的堆分配导致程序运行过慢，可以尝试重写该函数，以具体的类型作为参数，否则如果以接口的多个实现传递给函数，意味着需要创建多个包含重复逻辑的函数。

拥有C++或Rust背景的开发者或许会通过泛型产生专门的函数，但在Go1.21中或许不会生成更快的代码，详情见 ["Idiomatic Go and Generics"](Page199)

## 接口与nil *Interfaces and nil*

在讨论[指针](chap6)时，讨论过指针类型的`nil`与零值。在接口中，同样可以使用`nil`表示零值，但与具体的类型又有一些不同：
> 这有一篇关于iface与eface的详述：[理解Go interface的两种底层实现:iface和eface](https://blog.frognew.com/2018/11/go-interface-iface-eface.html)，深入后会有更多内容，不在此赘述

Go的运行时*runtime*通过包含两个指针字段的结构体实现接口，一个指向值，一个指向值的类型。只要指向类型的指针非空，那么接口就是非空的。如果需要接口为空，那么两个指针均需要为空，如下所示：
```go
var pointerCounter *Counter
fmt.Println(pointerCounter == nil)  // prints true

var incrementer Incrementer
fmt.Println(incrementer == nil)     //  prints true 

incrementer = pointerCounter
fmt.Println(incrementer == nil)     // prints false
```

## 接口是可比的 *Interfaces Are Comparable*
第三章提到的[可比类型](chap3)可以通过`==`进行比较。你可能会惊讶于接口也是可比的，就像接口为空需要两个指针均为空一样，两个接口相等也需要两个指针相等。但是这产生了一个问题，如果指针的类型是不可比的呢？例子如下：
```go
type Double interface {
    Double()
}
```


## The Empty Interface Says Nothing 
## Type Assertions and TypeSwitches
## Use TypesAssertions and TypeSwitches Sparingly
## Function Types Are a Bridge to Interfaces
## Implicit Interfaces Make Dependency Injection Easier
## Wire
## Go Isn't Particularly Object-Oriented( and That's Great)
## Exercises
## Wrapping Up
