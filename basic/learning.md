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

