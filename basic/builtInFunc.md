### 常用内置函数
---

1. `close`
2. `len, cap` len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）
3. `new, make` 两者都用于分配内存，new用于 **值类型**和**用户定义的类型（自定义结构）**，make用于**内置引用类型**（切片，map，管道）。他们将类型作为参数：new(type),make(type), new(T)分配类型T的零值并返回其地址，即指向类型T的指针，也可以用于基本类型。make(T)返回类型T初始化之后的值。
4. `copy, append` 用于复制和连接切片
5. `panic, recover` 均用于错误处理机制
6. `print, println` 底层打印函数，部署的时候建议使用 **fmt** 包
7. `complex, real imag` 创建和操作复数
