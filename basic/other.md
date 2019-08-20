### Other

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