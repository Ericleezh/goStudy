## 基础语法/特性


1. 在Go中，无法获取复数以及常量的地址。

2. switch可以接受任意类型的表达式,并且不需要使用break跳出循环。

3. 左大括号 `{` 不能单独放在一行中。

4. Go中，为了尽可能的简化代码，若函数体内若有未使用的变量，则编译无法通过（全局变量可以）。

5. 未使用的import不允许存在，同样会导致无法编译（可以使用 `_` 下划线忽略导入的包）。

6. 简短声明的变量只可以在函数体内使用。
``` go
str := 1 //编译错误
func main() {
    
}
```


7. struct的变量字段不能使用 `:=` 来赋值，可以使用预定义的变量来解决。

8. 显示类型的变量无法通过`nil`初始化。`nil`是`interface,function,pointer,map,slice,channel`类型变量的默认初始值，但是在声明的时候没有指定类型的话，编译器同样无法推断出变量的具体类型。

9. Go暂不支持泛型的概念，但可以使用空接口/类型选择（type switch）/反射（reflection）来实现相似的功能。

10. 在函数调用时，`slice,map,interface,channel`这样的引用类型都是默认使用引用传递。

11. 如果一个函数返回多个返回值，则可以传递一个切片（返回值具有相同类型）/结构体（返回值具有不同返回值）。**传递一个指针允许直接修改变量的值，内存消耗也更少**。

12. 命名返回值，作为结果形参（result params）被初始化为相应的零值，需要使用`()`括起来，返回的时候只需要直接使用`return`即可（但仍可返回明确的值）。
```go
func getMultiReturn(input int) (a int, b int) {
    a = input * 2
    b = input * 3
    // return a, b
    return
}
```

13. 空白符(blank identifier)，用于匹配不需要的一些值。

14. 传递变长参数，函数的最后一个参数采用 `...type`的形式，但若变长参数的类型并不是相同的，则可以使用`结构体/空接口`.
    * 结构体
    ```go
    type Options struct {
        par1 type1,
        par2 type2,
        ...
    }
    ```
    若需要初始化，则可以使用`Func(a, b, Options{p1:v1, p2:v2})`
    * 空接口(可以使用`for-range`循环以及switch结构对每一个参数类型进行判断)
    ```go
    func typecheck(.., ..,values … interface{}) {
        for _, value := range values {
            switch v := value.(type) {
                case int: …
                case float: …
                case string: …
                case bool: …
                default: …
            }
        }
    }
    ```

15. 关键字`defer`：允许推迟到函数返回之前（或任意位置执行`return`之后）执行某个语句/函数（作用类似于Java中的 *finally* 关键字的作用，一般用于释放某些已分配的资源）
    * 关闭文件流
    * 解锁一个已加锁的资源
    * 打印最终报告
    * 关闭数据库连接
    * 代码追踪：进入和离开某个函数的时候打印相关的消息
    * 记录函数的参数和返回值

16. 切片：对数组的一个连续片段的引用（即引用类型）
    * 容量=切片的长度+数组除切片之外的长度 （切片的长度永远不会超过他的容量）
    * 优点： 切片由于是引用，不需要使用额外的内存并且比使用数组会更有效率
    * 组织方式：有3个域的结构体（指向相关数组的指针、切片长度和切片容量）
    * 注意：切片本身是一个引用类型，所以不可以用指针指向slice

17. new() 和 make()的区别
    * new(T) 给每一个新的类型T分配一片内存，返回一个指向类型为T，值为0的地址的指针，适用于值类型（如数组和结构体）,相当于`&T{}`
    * make(T) 返回一个类型为T的初始值，只适用于3种内建的引用类型：`切片、map和channel`

18. 通过buffer串联字符串，最后转换成string类型（类似于Java中的StringBuilder，比+=的方式更节省内存和CPU）

19. for-range结构
    ```go
    for ix, value := range slice1 {
        ...
    }
    ```
    * 该结构仅适用于数组和切片，并且其中的value只是值的拷贝，不能修改对应索引位置的值。

20. Go中的字符串是不可变的，若需要修改字符串，需要先将字符串转换成字节数组，再修改数组中对应的元素，最后转换回字符串格式。
21. Go中 `...` 的三种用法
    * 可变长参数
    ```go
    func demo(num ...int) int {
        xxxx
    }
    ```
    * 将slice打散后追加
    ```go
    func main() {
        fmt.Println(Sum([]int{1,2,3}...))
    }
    ```
    * 略写数组长度
    ```go
    func main() {
        a:= [...]int{1,2,3}
    }
    ```
22. append函数的常见用法
![](http://wanyixia.online/image/github/201908/20190812181024.png)

23. slice和垃圾回收
    * 切片的底层是指向一个数组，**只有在没有任何切片指向的时候**，底层的内存才会被释放。（可能导致程序占用多余的内存）

24. map
    * **数组、切片和结构体**不能作为key(只含内建类型的struct可以作为key)，但**指针和接口类型**可以
    * 使用key在map中寻找value速度较线性查找快得多，但任然比从数组和切片的index中获取要慢得多。
    * map也的值也可以是函数，这样通过不同的key调用不同的function。
    * 使用make构造map，而不是使用new
    * 创建map时可以指明容量大小 `make(map[keytype]valuetype, cap)`
    * 判断map中的key是否存在： `val, isPresent = map[key]`

25. 遍历map：和一般的遍历相同，不过如果只是需要获取key的话，直接`for key := range map`的方式进行遍历即可。
    ```go
    for key := range map {
        xxx
    } 
    ```
26. map的排序
    * map默认是无序的，无论是根据key还是value都不排序
    * 若需要为map排序，可以将key/value拷贝到一个切片，再对切片即可。

27. go 中的锁机制。
    * map类型不存在锁机制，即非线性安全的
    * go中的锁机制由`sync`包中的`Mutex`实现。 `sync.Mutex`是一个互斥锁，用于确保同一时间内只有一个线程进入临界区。
    * sync中有`RWMutex`锁，通过`RLock()`来允许多个线程对同一个变量进行读操作，但仅能允许一个线程对其进行写操作
    * 若导致程序变慢，应该考虑使用goroutines或者channels来解决。

28. 自定义包和可见性
    * 写自己包的时候，应该使用**短小的不含有_的小写单词**命名，
    * import "包的路径或URL路径"
    * `import . "./pack"` ： 可以直接使用包内的函数以及变量
    * `import _ "./pack`: 只导入包内的`副作用`,即只执行init函数并初始化**全局变量**

29. 包的初始化
    * 一个包可以有多个init函数，执行顺序是无序的
    * init函数是不能被调用的
    * 导入的包在包自身初始化前被初始化，一个包在程序执行中只初始化一次

30. 结构体
    * 定义结构体
    ```go
    type identifier struct {
        field1 type1
        field2 type2
        ...
    }
    ```
    * 初始化， 初始化时可根据字段名初始化，未指定字段名则按顺序初始
    ```go
    ms := &struct1{10, 15.5, "Chris"}
    // 此时ms的类型是 *struct1, 这种方法底层仍然会调用new(),即new(Type)和&Type{}是等价的
    ```

31. 结构体工厂
    * Go 语言不支持面向对象编程语言中那样的构造子方法，但是可以很容易的在 Go 中实现 “构造子工厂”方法。为了方便通常会为类型定义一个工厂，按惯例，工厂的名字以 new 或 New 开头。
    ```go
    type File struct {
        fd      int     // 文件描述符
        name    string  // 文件名
    }
    ```
    工厂方法
    ```go
    func NewFile(fd int, name string) *File {
        if fd < 0 {
            return nil
        }
        return &File{fd, name}
    }
    ```

32. make()和new()的区别:使用make()一个结构体变量时，会导致编译错误；当使用new()一个map并填充数据时，会引发运行时错误。**因为new()返回的是一个指向nil的指针，并未分配内存。**

33. 匿名字段和内嵌结构体
    * 结构体中可以包含一个或多个**匿名(内嵌)字段**,即字段的名字默认是类型名称
    * 内嵌结构体类似于**继承**的思想，外层结构体可以直接使用`.`来引用内嵌字段
    * 方法也可以被内嵌，类似与继承方法
    * 方法和字段可以在外层类型中使用相同名称来进行覆盖

34. **类型和作用在它上面定义的方法必须在同一个包里定义**，否则会报`cannot define new methods on non-local type xxx`的编译错误。

35. 函数与方法的区别
    * 函数将变量作为参数：**Function1(recv)**
    * 方法在变量上被调用：**recv.Method1()**
    * 接收者必须有一个显式的名字，这个名字必须在方法中被使用。
    * receiver_type 叫做 （接收者）基本类型，这个类型必须在和方法同样的包中被声明。
    * **在 Go 中，（接收者）类型关联的方法不写在类型结构里面，就像类那样；耦合更加宽松；类型和方法之间的关联由接收者来建立。**
    * 方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是**独立**的。

36. 在另一个程序中修改或者只是读取一个类型的字段（使用getter和setter方法）
    * setter方法使用Set前缀
    * getter方法直接使用成员名 

37. 自定义的String()方法：定义了有多个方法的类型时，使用自定义的String()方法格式化输入内容时，fmt.Print() 和 fmt.Println() 也会自动使用 String() 方法。

38. 垃圾回收和SetFinalizer
    * Go运行时中有独立的进程（GC）搜索不再使用的变量并释放内存，可以通过`runtime`包访问GC进程。
    * 获取当前的内存状态
    ```go
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("%d Kb\n", m.Alloc / 1024)
    ```
    * 在对象被从内存移除之前执行一些特殊操作(例如写进日志文件)
    ```go
    runtime.SetFinalizer(obj, func(obj * typeObj))
    ```

39. 接口
    * 接口的名字由方法名加[e]r组成，或者以'I'开头
    * 类型不需要显示的声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一接口
    * 实现某个接口的类型（除了实现接口方法外）可以有其他的方法。
    * 一个类型可以实现多个接口。
    * 接口类型可以包含一个实例的引用， 该实例的类型实现了此接口（接口是动态类型）。

40. type-switch:用于判断接口变量的类型
```go
switch t := areaIntf.(type) {
case *Square:
    fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
    fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
    fmt.Printf("nil value: nothing to check?\n")
default:
    fmt.Printf("Unexpected type %T\n", t)
}
```

41. Go 语言规范定义了接口方法集的调用规则：
    * 类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集(会自动解引用)
    * 类型 T 的可调用方法集包含接受者为 T 的所有方法
    * 类型 T 的可调用方法集不包含接受者为 *T 的方法

42. 空接口/最小接口： 即不包含方法的接口（类似与Java中的Object），可以给其赋任何类型的值

43. 由于空接口切片和其他类型的数据切片在内存中的布局是不相同的，所以在将切片中的数据复制到一个空接口切片中时，会报`cannot use dataSlice (type []myType) as type []interface { } in assignment`的编译错误。 **必须使用`for-range`语句来一个个显式复制**

44. 通过反射修改/设置值
    * v.SetXXX(value)必须是v在**可设置**的情况下使用，否则会产生`reflect.Value.SetFloat using unaddressable value`的错误。
    * 是否可设置是Value的一个属性，并且不是所有的反射值都有这个属性，可以使用CanSet方法测试是否可设置。
    * 通过`Elem()`让value变为可设置状态。

45. 与其他语言相比较，Go是唯一结合了接口值，静态类型检查（该类型是否实现了某个接口），运行时动态转换的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许在不改变已有代码的情况下定义和使用新接口。

46. 实现了某个接口的类型可以被传给任何以此接口为参数的函数。

47. 若方法调用作用于类似`interface{}`这样的泛型上，可以通过类型断言来判断变量是否实现了相应接口。
```go
type xmlWriter interface {
    WriteXML(w io.Writer) error
}

// Exported XML streaming function.
func StreamXML(v interface{}, w io.Writer) error {
    if xw, ok := v.(xmlWriter); ok {
        // It’s an  xmlWriter, use method of asserted type.
        return xw.WriteXML(w)
    }
    // No implementation, so we have to use our own function (with perhaps reflection):
    return encodeToXML(v, w)
}

// Internal XML encoding function.
func encodeToXML(v interface{}, w io.Writer) error {
    // ...
}
```

48. 接口的继承
    * 当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法

49. 读文件
    * ReadingString()
    * 将整个文件内容读取到一个字符串中--ioutil.ReadFile()
    * 带缓冲的读取--bufio.Reader()
    * 按列读取文件中的数据--使用`fmt`中提供的以`FScan`开头的函数读取
    
50. go中可被编码为Json的数据结构(`Marshal() Unmarshal()`)
    * json对象仅支持字符串类型的key;要转换go map结构，map必须是`map[string]T`(其中的T是json包中支持的任何类型)
    * Channel,复杂类型和函数类型不能被解析
    * 不支持循环的数据结构，会导致**序列化进入死循环**
    * 指针可以被编码，实际上是对指针指向的值进行编码(或者指针是nil)

51. Gob(Go binary,Go自己的以二进制形式序列化和反序列化数据的格式，类似于Python的"pickle",java的"Serialization")
    * 通常用于**RPC**中参数和结果的传输
    * Gob特定用于**纯Go**的环境中,例如俩个Go写的服务之间的通信。(高效/优化)
    * Gob不是可外部定义,语言无关的编码方式
    * 在encode 和 decode时用到了Go的反射
    * 只有可导出的字段会被编码，零值会被忽略

52. defer-panic-and-recover 机制
    * Go中没有类似Java的`try/catch`异常机制，无法执行抛异常操作
    * go中的"捕获"异常更为轻量,并且应作为处理错误的最后手段
    * 普通错误：在函数和方法中返回错误对象，若错误对象为nil则没有错误
    * `panic and recover`用来处理真正的异常

53. 运行时异常和panic
    * 发生类似数组下标越界或者类型断言失败等运行错误时，Go运行时会触发`panic`(runtime.Error)
    * panic:当错误条件很严苛且不可恢复，程序无法运行时，可以使用**panic**函数来产生一个终止程序的运行时错误
    * panic接收一个任何类型的参数，一般为字符串，在终止程序是给出提示

54. recover(用于从panic或错误场景中恢复)
    * recover只能用于在defer修饰的函数中，用于取得panic调用中传递过来的错误值，若正常执行，调用recover会返回nil。

55. gotest
    * 测试文件命名规范： `*_test`
    * `_test`相关的程序不会被普通的Go编译器编译
    * gotest会编译所有的程序，包括普通程序和测试程序
    * 测试函数命名规范： `TestXXXX`

56. 并行：**同一个程序在某个时间点同时运行在多核或多处理器上才是真正的并行**
    * 确定性的(明确定义排序)
    * 非确定性的(加锁/互斥从而未定义排序)

57. 竞态：使用多线程的引用需要注意内存中数据共享的问题，它们会被多线程以无法预知的方式进行操作，导致一些无法重现或者随机的结果

58. 协程：应用程序并发处理的部分称为协程
    * go使用channels来同步协程
    * 栈的管理是自动的，在协程退货后会自动释放
    * 协程可以在多个操作系统线程之间运行，也可以在线程之内运行
    * 协程的栈会根据需要进行伸缩，不出现栈溢出
    * runtime.Gosched() 可以使计算均匀分布，使通信不至于迟迟得不到响应


