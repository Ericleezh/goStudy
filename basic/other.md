### Other
---

#### 关于递归时栈溢出问题的解决
* [懒惰求值法](https://zh.wikipedia.org/wiki/%E6%83%B0%E6%80%A7%E6%B1%82%E5%80%BC)
* channel 实现
* goroutine 实现

#### 将数组传递给函数造成大量内存消耗，解决方法：
* 传递数组的指针
* 使用数组的切片

#### 为自定义包使用godoc
* 注释必须以 `//` 开头并无空行放在声明前
* 命令行下进入目录使用命令： `godoc -http=:6060 -goroot="."` （.表示当前路径） 执行后进入 `localhost:6060`即可看到对应的文档
* 通过设置 `sync_minutes=n` 可以使文档每n分钟自动更新

#### 如何在类型中嵌入功能
* 聚合（组合）:包含一个所需功能类型的具名字段
* 内嵌：内嵌（匿名地）所需功能类型。（不需要指针）

#### 多重继承
* 多重继承指的是获得多个父亲行为的能力，传统的面向对象的语言中通常是不被实现的(C++和Python例外),在Go语言中，通过在类型中嵌入所必须的父类型，可以简单的实现多重继承。

#### 和其他面向对象语言比较 Go 的类型和方法
* 在大部分面向对象语言中，方法在类的上下文中被定义和继承，在调用方法时，运行时会检测类以及它的超类中是否有此方法的定义，若没有会导致异常发生；**而在Go中，只要方法在该类型中定义了，就可以调用他，与其他类型上是否有该方法没有关系。**

#### To sum up for go of 'class' (关于在Go中‘类’的总结) [goop](https://github.com/losalamos/goop)
> Go中，类型就是类（数据和关联的 方法）
> 代码复用通过**组合和委托**实现，多态通过接口的使用来实现，也叫组件编程

#### 若接口中的方法没有被实现，即使没有地方进行调用，同样会给出没有实现的编译错误。
* 那意思就是我定义了一定要实现咯，若只是需要先定义，实现暂未想好，岂不是很坑。

#### 类型断言：如何检测和转换接口变量的类型
* v 是 varI 转换到类型 T 的值，ok 会是 true；否则 v 是类型 T 的零值，ok 是 false
```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```

#### 测试一个值是否实现了某个接口
```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```
