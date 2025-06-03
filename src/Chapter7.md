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

## A Qyucj Lesson on Interfaces
## Interfaces Are Type-Safe Duck Typing
## Embedding and Interfaces
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
