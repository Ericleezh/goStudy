### 基础语法/特性
---

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





