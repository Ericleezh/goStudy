## Learning code

#### #1.结构体、集合以及高阶函数相关demo
```go

type Any interface{}
type Car struct {
    Model        string
    Manufacturer string
    BuildYear    int
    // ...
}
type Cars []*Car

func main() {
    // make some cars:
    ford := &Car{"Fiesta", "Ford", 2008}
    bmw := &Car{"XL 450", "BMW", 2011}
    merc := &Car{"D600", "Mercedes", 2009}
    bmw2 := &Car{"X 800", "BMW", 2008}
    // query:
    allCars := Cars([]*Car{ford, bmw, merc, bmw2})
    allNewBMWs := allCars.FindAll(func(car *Car) bool {
        return (car.Manufacturer == "BMW") && (car.BuildYear > 2010)
    })
    fmt.Println("AllCars: ", allCars)
    fmt.Println("New BMWs: ", allNewBMWs)
    //
    manufacturers := []string{"Ford", "Aston Martin", "Land Rover", "BMW", "Jaguar"}
    sortedAppender, sortedCars := MakeSortedAppender(manufacturers)
    allCars.Process(sortedAppender)
    fmt.Println("Map sortedCars: ", sortedCars)
    BMWCount := len(sortedCars["BMW"])
    fmt.Println("We have ", BMWCount, " BMWs")
}

// Process all cars with the given function f:
func (cs Cars) Process(f func(car *Car)) {
    for _, c := range cs {
        f(c)
    }
}

// Find all cars matching a given criteria.
func (cs Cars) FindAll(f func(car *Car) bool) Cars {
    cars := make([]*Car, 0)

    cs.Process(func(c *Car) {
        if f(c) {
            cars = append(cars, c)
        }
    })
    return cars
}

// Process cars and create new data.
func (cs Cars) Map(f func(car *Car) Any) []Any {
    result := make([]Any, len(cs))
    ix := 0
    cs.Process(func(c *Car) {
        result[ix] = f(c)
        ix++
    })
    return result
}

func MakeSortedAppender(manufacturers []string) (func(car *Car), map[string]Cars) {
    // Prepare maps of sorted cars.
    sortedCars := make(map[string]Cars)

    for _, m := range manufacturers {
        sortedCars[m] = make([]*Car, 0)
    }
    sortedCars["Default"] = make([]*Car, 0)

    // Prepare appender function:
    appender := func(c *Car) {
        if _, ok := sortedCars[c.Manufacturer]; ok {
            sortedCars[c.Manufacturer] = append(sortedCars[c.Manufacturer], c)
        } else {
            sortedCars["Default"] = append(sortedCars["Default"], c)
        }
    }
    return appender, sortedCars
}
```

---

#### #2.通过buffer读取文件并输出到控制台
```go
func cat(r *bufio.Reader) {
    for {
        buf, err := r.ReadBytes('\n')
        if err == io.EOF {
            break
        }
        fmt.Fprintf(os.Stdout, "%s", buf)
    }
    return
}

func main() {
    flag.Parse()
    if flag.NArg() == 0 {
        cat(bufio.NewReader(os.Stdin))
    }
    for i := 0; i < flag.NArg(); i++ {
        f, err := os.Open(flag.Arg(i))
        if err != nil {
            fmt.Fprintf(os.Stderr, "%s:error reading from %s: %s\n",os.Args[0], flag.Arg(i), err.Error())
            continue
        }
        cat(bufio.NewReader(f))
    }
}
```

---

#### #3.panic,defer和recover结合使用demo
```go

import (
    "fmt"
)

func badCall() {
    panic("bad end")
}

func test() {
    defer func() {
        if e := recover(); e != nil {
            fmt.Printf("Panicing %s\r\n", e)
        }
    }()
    badCall()
    fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
    fmt.Printf("Calling test\r\n")
    test()
    fmt.Printf("Test completed\r\n")
}
```

---

#### #4.自定义包中的错误处理和panicking
* 向包的调用者返回错误值（而不是 panic）
* 在包内部，总是应该从 panic 中 recover：不允许显式的超出包范围的 panic()。

```go
// parse.go
package parse

import (
    "fmt"
    "strings"
    "strconv"
)

// A ParseError indicates an error in converting a word into an integer.
type ParseError struct {
    Index int      // The index into the space-separated list of words.
    Word  string   // The word that generated the parse error.
    Err error // The raw error that precipitated this error, if any.
}

// String returns a human-readable error message.
func (e *ParseError) String() string {
    return fmt.Sprintf("pkg parse: error parsing %q as int", e.Word)
}

// Parse parses the space-separated words in in put as integers.
func Parse(input string) (numbers []int, err error) {
    defer func() {
        if r := recover(); r != nil {
            var ok bool
            err, ok = r.(error)
            if !ok {
                err = fmt.Errorf("pkg: %v", r)
            }
        }
    }()

    fields := strings.Fields(input)
    numbers = fields2numbers(fields)
    return
}

func fields2numbers(fields []string) (numbers []int) {
    if len(fields) == 0 {
        panic("no words to parse")
    }
    for idx, field := range fields {
        num, err := strconv.Atoi(field)
        if err != nil {
            panic(&ParseError{idx, field, err})
        }
        numbers = append(numbers, num)
    }
    return
}
```

```go
func main() {
    var examples = []string{
            "1 2 3 4 5",
            "100 50 25 12.5 6.25",
            "2 + 2 = 4",
            "1st class",
            "",
    }

    for _, ex := range examples {
        fmt.Printf("Parsing %q:\n  ", ex)
        nums, err := parse.Parse(ex)
        if err != nil {
            fmt.Println(err) // here String() method from ParseError is used
            continue
        }
        fmt.Println(nums)
    }
}
```

---

#### #5.打印素数(协程)
```go
// Send the sequence 2, 3, 4, ... to returned channel
func generate() chan int {
    ch := make(chan int)
    go func() {
        for i := 2; ; i++ {
            ch <- i
        }
    }()
    return ch
}

// Filter out input values divisible by 'prime', send rest to returned channel
func filter(in chan int, prime int) chan int {
    out := make(chan int)
    go func() {
        for {
            if i := <-in; i%prime != 0 {
                out <- i
            }
        }
    }()
    return out
}

func sieve() chan int {
    out := make(chan int)
    go func() {
        ch := generate()
        for {
            prime := <-ch
            ch = filter(ch, prime)
            out <- prime
        }
    }()
    return out
}

func main() {
    primes := sieve()
    for {
        fmt.Println(<-primes)
    }
}
```

---

#### #6.选择使用锁还是通道
* 使用锁的场景
    - 访问共享数据结构中的缓存信息
    - 保存引用程序上下文的状态信息数据
* 使用通道的场景
    - 与异步操作的结果进行交互
    - 分发任务
    - 传递数据所有权

---

#### #7.使用协程和通道实现惰性生成器
```go
type Any interface{}
type EvalFunc func(Any) (Any, Any)

func main() {
    evenFunc := func(state Any) (Any, Any) {
        os := state.(int)
        ns := os + 2
        return os, ns
    }
    
    even := BuildLazyIntEvaluator(evenFunc, 0)
    
    for i := 0; i < 10; i++ {
        fmt.Printf("%vth even: %v\n", i, even())
    }
}

func BuildLazyEvaluator(evalFunc EvalFunc, initState Any) func() Any {
    retValChan := make(chan Any)
    loopFunc := func() {
        var actState Any = initState
        var retVal Any
        for {
            retVal, actState = evalFunc(actState)
            retValChan <- retVal
        }
    }
    retFunc := func() Any {
        return <- retValChan
    }
    go loopFunc()
    return retFunc
}

function BuildLazyIntEvaluator(evalFunc EvalFunc, initState Any) func() int {
    ef := BuildLazyEvaluator(evalFunc, initState)
    return func() int {
        return ef().(int)
    }
}
```

---

#### #8.C/S模式下, goroutines and channels demo.
```go
package main

import (
    "fmt"
)

type Request struct {
    a, b   int
    replyc chan int // reply channel inside the Request
}

type binOp func(a, b int) int

func run(op binOp, req *Request) {
    req.replyc <- op(req.a, req.b)
}

func server(op binOp, service chan *Request, quit chan bool) {
    for {
        select {
        case req := <-service: // requests arrive here
            // start goroutine for request:
            go run(op, req) // don't wait for op
        case <-quit:
            return
        }
    }
}

func startServer(op binOp) (service chan *Request, quit chan bool) {
    service = make(chan *Request)
    quit = make(chan bool)
    go server(op, service, quit)
    return service, quit
}

func main() {
    adder, quit := startServer(func(a, b int) int { return a + b })
    const N = 10000
    var reqs [N]Request
    for i := 0; i < N; i++ {
        req := &reqs[i]
        req.a = i
        req.b = i + N
        req.replyc = make(chan int)
        adder <- req
    }
    // checks:
    for i := N - 1; i >= 0; i-- { // doesn't matter what order
        if <-reqs[i].replyc != N+2*i {
            fmt.Println("fail at", i)
        } else {
            fmt.Println("Request ", i, " is ok!")
        }
    }
    quit <- true
    fmt.Println("done")
}
```

---

#### #9.并行化大量数据的运算(多个处理步骤启用多个协程)
```go
func ParallelProcessData (in <-chan *Data, out chan<- *Data) {
    // make channels:
    preOut := make(chan *Data, 100)
    stepAOut := make(chan *Data, 100)
    stepBOut := make(chan *Data, 100)
    stepCOut := make(chan *Data, 100)
    // start parallel computations:
    go PreprocessData(in, preOut)
    go ProcessStepA(preOut,StepAOut)
    go ProcessStepB(StepAOut,StepBOut)
    go ProcessStepC(StepBOut,StepCOut)
    go PostProcessData(StepCOut,out)
} 
```
