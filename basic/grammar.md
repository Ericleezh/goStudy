### 基础语法/特性

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









