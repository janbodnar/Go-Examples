# Types

Go is a statically typed programming language, meaning every variable has a specific  
type that is determined at compile time. The Go type system includes both basic built-in  
types and composite types that can be constructed from other types. Understanding Go's  
type system is fundamental to writing effective Go programs.

Go's basic types include boolean, numeric (integers, floating-point numbers, and complex  
numbers), and string types. These form the foundation for more complex data structures.  
Numeric types are further divided into signed and unsigned integers of various sizes,  
floating-point numbers, and complex numbers for mathematical computations.

Go also provides powerful composite types including arrays, slices, maps, structs,  
pointers, functions, interfaces, and channels. These types allow you to build sophisticated  
data structures and implement complex algorithms. The language's type system promotes  
both safety and performance through static type checking and efficient memory layout.

One of Go's distinctive features is its approach to interfaces, which enable polymorphism  
through method sets rather than explicit inheritance. This design philosophy, combined  
with features like type embedding and type assertions, makes Go's type system both  
powerful and accessible. The `any` type (alias for `interface{}`) provides flexibility  
when working with values of unknown types at compile time.


## Boolean type

The boolean type represents truth values and can be either `true` or `false`. Boolean  
variables are often used in conditional statements and logical operations.  

```go
package main

import "fmt"

func main() {
    var active bool = true
    var finished bool = false
    
    fmt.Println("Active:", active)
    fmt.Println("Finished:", finished)
    fmt.Println("Both true:", active && finished)
    fmt.Println("At least one true:", active || finished)
}
```

This example demonstrates basic boolean operations including logical AND (`&&`) and  
logical OR (`||`). Boolean values are essential for program flow control and decision  
making in Go applications.  

## Integer types

Go provides several integer types of different sizes, both signed and unsigned. The  
choice of integer type depends on the range of values you need to store.  

```go
package main

import "fmt"

func main() {
    var smallInt int8 = 127
    var mediumInt int16 = 32767
    var largeInt int32 = 2147483647
    var hugeLInt int64 = 9223372036854775807
    
    var smallUint uint8 = 255
    var defaultInt int = 42
    
    fmt.Printf("int8: %d, int16: %d\n", smallInt, mediumInt)
    fmt.Printf("int32: %d, int64: %d\n", largeInt, hugeLInt)
    fmt.Printf("uint8: %d, int: %d\n", smallUint, defaultInt)
}
```

The `int` type size depends on the platform (32 or 64 bits). Unsigned integers can  
store larger positive values but cannot represent negative numbers. Choose the  
appropriate size based on your data requirements and memory constraints.  

## Floating-point types

Go supports two floating-point types: `float32` and `float64`. These are used for  
representing real numbers with decimal precision.  

```go
package main

import "fmt"

func main() {
    var price float32 = 19.99
    var distance float64 = 384400.0
    var pi float64 = 3.141592653589793
    
    fmt.Printf("Price: %.2f\n", price)
    fmt.Printf("Distance to moon: %.1f km\n", distance)
    fmt.Printf("Pi: %.15f\n", pi)
    
    // Mathematical operations
    var result float64 = distance * 2
    fmt.Printf("Round trip: %.1f km\n", result)
}
```

`float64` provides higher precision than `float32` and is generally preferred unless  
memory usage is a critical concern. Floating-point arithmetic follows IEEE 754  
standards and may have precision limitations for very large or very small numbers.  

## String type

Strings in Go are immutable sequences of bytes, typically containing UTF-8 encoded  
text. They are a fundamental type used extensively in Go programs.  

```go
package main

import "fmt"

func main() {
    var greeting string = "hello there"
    var name string = "Go programmer"
    
    message := greeting + ", " + name + "!"
    
    fmt.Println("Message:", message)
    fmt.Println("Length:", len(message))
    fmt.Println("First character:", message[0])
    
    // String literals
    multiline := `This is a
    multiline string
    using backticks`
    fmt.Println(multiline)
}
```

Go strings are UTF-8 encoded by default and can contain any Unicode character. The  
backtick syntax creates raw string literals that can span multiple lines and don't  
interpret escape sequences.  

## Rune type

The `rune` type is an alias for `int32` and represents a Unicode code point. It's  
used when you need to work with individual characters in strings.  

```go
package main

import "fmt"

func main() {
    var letter rune = 'A'
    var emoji rune = '😀'
    var unicode rune = 'π'
    
    fmt.Printf("Letter: %c (code: %d)\n", letter, letter)
    fmt.Printf("Emoji: %c (code: %d)\n", emoji, emoji)
    fmt.Printf("Pi symbol: %c (code: %d)\n", unicode, unicode)
    
    // Working with string runes
    text := "hello"
    for i, r := range text {
        fmt.Printf("Position %d: %c\n", i, r)
    }
}
```

Runes are essential when processing international text or when you need to work with  
individual characters rather than bytes. The `range` loop over strings automatically  
decodes UTF-8 sequences into runes.  

## Byte type

The `byte` type is an alias for `uint8` and is used to work with raw binary data  
or ASCII text processing.  

```go
package main

import "fmt"

func main() {
    var data byte = 65
    var ascii byte = 'Z'
    
    fmt.Printf("Byte value: %d, character: %c\n", data, data)
    fmt.Printf("ASCII 'Z': %d, character: %c\n", ascii, ascii)
    
    // Working with byte slices
    text := "Go bytes"
    bytes := []byte(text)
    
    fmt.Printf("String: %s\n", text)
    fmt.Printf("Bytes: %v\n", bytes)
    fmt.Printf("Back to string: %s\n", string(bytes))
}
```

Bytes are commonly used for low-level operations, network programming, and file I/O  
where you need to work with raw data. Converting between strings and byte slices  
is a common operation in Go programs.  

## Arrays

Arrays are fixed-size sequences of elements of the same type. The size is part of  
the array's type and cannot be changed after declaration.  

```go
package main

import "fmt"

func main() {
    var numbers [5]int
    var colors [3]string = [3]string{"red", "green", "blue"}
    
    // Initialize with values
    scores := [4]int{95, 87, 92, 78}
    
    // Let Go count the elements
    days := [...]string{"Mon", "Tue", "Wed", "Thu", "Fri"}
    
    numbers[0] = 10
    numbers[1] = 20
    
    fmt.Println("Numbers:", numbers)
    fmt.Println("Colors:", colors)
    fmt.Println("Scores:", scores)
    fmt.Println("Days:", days)
    fmt.Println("Array length:", len(days))
}
```

Arrays provide predictable memory layout and bounds checking. However, their fixed  
size makes them less flexible than slices for many use cases. The `...` syntax  
lets the compiler count array elements automatically.  

## Slices

Slices are dynamic arrays that can grow and shrink. They're more commonly used than  
arrays due to their flexibility and built-in functions.  

```go
package main

import "fmt"

func main() {
    // Creating slices
    var numbers []int
    colors := []string{"red", "green", "blue"}
    
    // Using make
    scores := make([]int, 3, 5) // length 3, capacity 5
    
    // Adding elements
    numbers = append(numbers, 1, 2, 3, 4)
    colors = append(colors, "yellow")
    
    fmt.Println("Numbers:", numbers)
    fmt.Println("Colors:", colors)
    fmt.Printf("Scores: %v (len: %d, cap: %d)\n", 
               scores, len(scores), cap(scores))
    
    // Slicing
    subset := numbers[1:3]
    fmt.Println("Subset:", subset)
}
```

Slices are reference types that point to an underlying array. The `append` function  
automatically handles memory allocation when the slice grows beyond its capacity.  
Slicing operations create new views of the same underlying data.  

## Maps

Maps are key-value pairs similar to hash tables or dictionaries in other languages.  
They provide efficient lookups, insertions, and deletions.  

```go
package main

import "fmt"

func main() {
    // Creating maps
    ages := make(map[string]int)
    ages["Alice"] = 30
    ages["Bob"] = 25
    
    // Map literals
    capitals := map[string]string{
        "France": "Paris",
        "Japan":  "Tokyo",
        "Italy":  "Rome",
    }
    
    fmt.Println("Ages:", ages)
    fmt.Println("Capitals:", capitals)
    
    // Check if key exists
    age, exists := ages["Charlie"]
    if exists {
        fmt.Printf("Charlie is %d years old\n", age)
    } else {
        fmt.Println("Charlie's age is unknown")
    }
}
```

Maps are reference types and the zero value is `nil`. The comma-ok idiom allows  
you to distinguish between a key with a zero value and a missing key. Maps are  
not ordered, so iteration order is not guaranteed.  

## Structs

Structs are user-defined types that group together variables of different types.  
They're used to create more complex data structures and model real-world entities.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
    City string
}

func main() {
    // Creating struct instances
    var p1 Person
    p1.Name = "Alice"
    p1.Age = 30
    p1.City = "New York"
    
    // Struct literals
    p2 := Person{"Bob", 25, "London"}
    p3 := Person{
        Name: "Charlie",
        Age:  35,
        City: "Paris",
    }
    
    fmt.Printf("Person 1: %+v\n", p1)
    fmt.Printf("Person 2: %+v\n", p2)
    fmt.Printf("Person 3: %+v\n", p3)
}
```

Structs can be initialized using various syntaxes. The `%+v` format verb prints  
struct fields with their names. Structs are value types, meaning they're copied  
when assigned or passed to functions.  

## Pointers

Pointers hold memory addresses of variables, allowing indirect access and  
modification of values. They're essential for efficient memory usage and  
sharing data between functions.  

```go
package main

import "fmt"

func main() {
    x := 42
    p := &x // p points to x
    
    fmt.Printf("x = %d\n", x)
    fmt.Printf("Address of x = %p\n", &x)
    fmt.Printf("p = %p\n", p)
    fmt.Printf("Value at p = %d\n", *p)
    
    // Modify through pointer
    *p = 100
    fmt.Printf("After *p = 100, x = %d\n", x)
    
    // Pointer to struct
    type Point struct{ X, Y int }
    point := Point{10, 20}
    ptr := &point
    ptr.X = 30 // equivalent to (*ptr).X = 30
    fmt.Printf("Point: %+v\n", point)
}
```

The `&` operator gets the address of a variable, while `*` dereferences a pointer  
to access the value it points to. Go automatically dereferences pointers when  
accessing struct fields, making the syntax cleaner.  

## Interfaces

Interfaces define method signatures that types must implement. They enable  
polymorphism and loose coupling between different parts of your program.  

```go
package main

import "fmt"

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

func main() {
    var s Shape
    
    s = Rectangle{Width: 10, Height: 5}
    fmt.Printf("Rectangle: Area=%.2f, Perimeter=%.2f\n", 
               s.Area(), s.Perimeter())
    
    s = Circle{Radius: 3}
    fmt.Printf("Circle: Area=%.2f, Perimeter=%.2f\n", 
               s.Area(), s.Perimeter())
}
```

Interfaces are implemented implicitly - any type that has the required methods  
automatically satisfies the interface. This promotes composition over inheritance  
and makes code more flexible and testable.  

## Channels

Channels are typed conduits for communication between goroutines. They're  
fundamental to Go's concurrency model and enable safe data sharing.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unbuffered channel
    ch := make(chan string)
    
    // Send data in a goroutine
    go func() {
        time.Sleep(1 * time.Second)
        ch <- "hello from goroutine"
    }()
    
    // Receive data
    message := <-ch
    fmt.Println("Received:", message)
    
    // Buffered channel
    numbers := make(chan int, 3)
    numbers <- 1
    numbers <- 2
    numbers <- 3
    
    fmt.Println("Numbers:", <-numbers, <-numbers, <-numbers)
}
```

Channels can be unbuffered (synchronous) or buffered (asynchronous). The `<-`  
operator is used for both sending to and receiving from channels. Channels  
provide a safe way to communicate between concurrent goroutines.  

## Function types

Functions are first-class values in Go and can be assigned to variables,  
passed as arguments, and returned from other functions.  

```go
package main

import "fmt"

type Calculator func(int, int) int

func add(a, b int) int {
    return a + b
}

func multiply(a, b int) int {
    return a * b
}

func apply(calc Calculator, x, y int) int {
    return calc(x, y)
}

func main() {
    // Function variables
    var operation Calculator = add
    result := operation(5, 3)
    fmt.Printf("Addition: %d\n", result)
    
    // Passing functions as arguments
    fmt.Printf("Multiply result: %d\n", apply(multiply, 4, 6))
    
    // Anonymous functions
    square := func(n int) int {
        return n * n
    }
    fmt.Printf("Square of 7: %d\n", square(7))
}
```

Function types specify the parameter and return types. Anonymous functions  
(closures) can capture variables from their surrounding scope, making them  
useful for callbacks and event handling.  

## Type aliases

Type aliases create alternative names for existing types, improving code  
readability and providing abstraction without creating new types.  

```go
package main

import "fmt"

type UserID = int
type Email = string
type Status = string

const (
    StatusActive   Status = "active"
    StatusInactive Status = "inactive"
)

func main() {
    var id UserID = 12345
    var email Email = "user@example.com"
    var status Status = StatusActive
    
    fmt.Printf("User ID: %d\n", id)
    fmt.Printf("Email: %s\n", email)
    fmt.Printf("Status: %s\n", status)
    
    // Type aliases are identical to their underlying type
    var regularInt int = id
    fmt.Printf("Regular int: %d\n", regularInt)
}
```

Type aliases using `=` create true aliases that are identical to the original  
type. They're useful for creating more descriptive names for existing types  
and for gradual code migration.  

## Custom types

Custom types create new types based on existing ones, allowing you to define  
methods and add behavior to simple types.  

```go
package main

import "fmt"

type Temperature float64
type Distance int

func (t Temperature) Celsius() float64 {
    return float64(t)
}

func (t Temperature) Fahrenheit() float64 {
    return float64(t)*9/5 + 32
}

func (d Distance) Meters() int {
    return int(d)
}

func (d Distance) Kilometers() float64 {
    return float64(d) / 1000
}

func main() {
    var temp Temperature = 25.5
    var dist Distance = 1500
    
    fmt.Printf("Temperature: %.1f°C, %.1f°F\n", 
               temp.Celsius(), temp.Fahrenheit())
    fmt.Printf("Distance: %dm, %.2fkm\n", 
               dist.Meters(), dist.Kilometers())
    
    // Custom types are distinct from their underlying types
    // var regularFloat float64 = temp // This would cause a compile error
    var regularFloat float64 = float64(temp) // Explicit conversion required
    fmt.Printf("As float64: %.1f\n", regularFloat)
}
```

Custom types (defined with `type Name UnderlyingType`) create distinct types  
that require explicit conversion. They allow you to add methods and create  
more type-safe APIs.  

## Empty interface

The empty interface `interface{}` (or `any` in Go 1.18+) can hold values of  
any type, providing maximum flexibility when the type is unknown at compile time.  

```go
package main

import "fmt"

func printValue(value any) {
    fmt.Printf("Value: %v, Type: %T\n", value, value)
}

func main() {
    var anything any
    
    anything = 42
    printValue(anything)
    
    anything = "hello there"
    printValue(anything)
    
    anything = []int{1, 2, 3}
    printValue(anything)
    
    anything = map[string]int{"age": 25}
    printValue(anything)
    
    // Working with mixed types
    values := []any{42, "text", 3.14, true}
    for i, v := range values {
        fmt.Printf("values[%d]: %v (%T)\n", i, v, v)
    }
}
```

The `any` type is useful for generic programming, JSON parsing, and interfacing  
with dynamic systems. However, you often need type assertions to work with  
the actual values.  

## Type assertions

Type assertions provide access to the underlying concrete value of an interface.  
They're essential when working with `any` or other interface types.  

```go
package main

import "fmt"

func processValue(value any) {
    // Type assertion with check
    if str, ok := value.(string); ok {
        fmt.Printf("String value: '%s' (length: %d)\n", str, len(str))
    } else if num, ok := value.(int); ok {
        fmt.Printf("Integer value: %d (doubled: %d)\n", num, num*2)
    } else if slice, ok := value.([]int); ok {
        fmt.Printf("Slice value: %v (sum: %d)\n", slice, sum(slice))
    } else {
        fmt.Printf("Unknown type: %T\n", value)
    }
}

func sum(nums []int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

func main() {
    values := []any{
        "hello there",
        42,
        []int{1, 2, 3, 4, 5},
        3.14,
    }
    
    for _, v := range values {
        processValue(v)
    }
}
```

The comma-ok idiom (`value, ok := interface.(Type)`) safely checks if a type  
assertion is valid. Without the boolean check, a failed assertion would panic  
the program.  

## Type switches

Type switches provide a clean way to handle multiple types when working with  
interface values, offering better readability than multiple type assertions.  

```go
package main

import "fmt"

func describe(value any) {
    switch v := value.(type) {
    case string:
        fmt.Printf("String: '%s' has %d characters\n", v, len(v))
    case int:
        if v < 0 {
            fmt.Printf("Negative integer: %d\n", v)
        } else {
            fmt.Printf("Positive integer: %d\n", v)
        }
    case float64:
        fmt.Printf("Float: %.2f\n", v)
    case bool:
        fmt.Printf("Boolean: %t\n", v)
    case []int:
        fmt.Printf("Integer slice with %d elements: %v\n", len(v), v)
    case nil:
        fmt.Println("Nil value")
    default:
        fmt.Printf("Unknown type: %T with value %v\n", v, v)
    }
}

func main() {
    values := []any{
        "hello there",
        42,
        -10,
        3.14159,
        true,
        []int{1, 2, 3},
        nil,
        complex(1, 2),
    }
    
    for _, v := range values {
        describe(v)
    }
}
```

Type switches are more efficient and readable than chains of type assertions.  
The `v` variable in each case is automatically converted to the matched type,  
eliminating the need for explicit casting.  

## Complex numbers

Go has built-in support for complex numbers with `complex64` and `complex128`  
types, useful for mathematical and scientific computations.  

```go
package main

import "fmt"

func main() {
    // Creating complex numbers
    var c1 complex128 = complex(3, 4)
    var c2 complex128 = 1 + 2i
    c3 := 2.5 + 1.5i
    
    fmt.Printf("c1: %.1f\n", c1)
    fmt.Printf("c2: %.1f\n", c2)
    fmt.Printf("c3: %.1f\n", c3)
    
    // Operations
    sum := c1 + c2
    product := c1 * c2
    
    fmt.Printf("c1 + c2 = %.1f\n", sum)
    fmt.Printf("c1 * c2 = %.1f\n", product)
    
    // Extract real and imaginary parts
    fmt.Printf("Real part of c1: %.1f\n", real(c1))
    fmt.Printf("Imaginary part of c1: %.1f\n", imag(c1))
}
```

Complex numbers support all standard arithmetic operations. The `real()` and  
`imag()` functions extract the real and imaginary components. Complex literals  
use the `i` suffix for the imaginary unit.  

## Constants and iota

Constants are compile-time values that cannot be changed. The `iota` identifier  
is used to create sequences of related constants automatically.  

```go
package main

import "fmt"

const (
    // Basic constants
    MaxRetries = 3
    Timeout    = 30
    APIKey     = "secret-key"
)

const (
    // Using iota for enumeration
    StatusPending = iota
    StatusRunning
    StatusComplete
    StatusFailed
)

const (
    // Custom iota expressions
    KB = 1 << (10 * iota)
    MB
    GB
    TB
)

func main() {
    fmt.Printf("Max retries: %d\n", MaxRetries)
    fmt.Printf("Timeout: %d seconds\n", Timeout)
    
    fmt.Printf("Status values: %d, %d, %d, %d\n", 
               StatusPending, StatusRunning, StatusComplete, StatusFailed)
    
    fmt.Printf("Storage sizes: %d KB, %d MB, %d GB, %d TB\n", 
               KB, MB/KB, GB/MB, TB/GB)
    
    // Constants can be untyped
    const untypedInt = 42
    var int32Val int32 = untypedInt
    var int64Val int64 = untypedInt
    fmt.Printf("Untyped constant used as: int32(%d), int64(%d)\n", 
               int32Val, int64Val)
}
```

Constants must be determinable at compile time. `iota` starts at 0 and  
increments by 1 for each item in the constant block. Untyped constants  
can be used wherever a typed value of compatible type is expected.  

## Zero values

Every type in Go has a zero value - the default value assigned to variables  
that are declared but not explicitly initialized.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    // Basic types zero values
    var b bool
    var i int
    var f float64
    var s string
    
    fmt.Printf("bool: %t\n", b)
    fmt.Printf("int: %d\n", i)
    fmt.Printf("float64: %f\n", f)
    fmt.Printf("string: '%s'\n", s)
    
    // Composite types zero values
    var slice []int
    var m map[string]int
    var ptr *int
    var ch chan int
    var person Person
    
    fmt.Printf("slice: %v (nil: %t)\n", slice, slice == nil)
    fmt.Printf("map: %v (nil: %t)\n", m, m == nil)
    fmt.Printf("pointer: %v (nil: %t)\n", ptr, ptr == nil)
    fmt.Printf("channel: %v (nil: %t)\n", ch, ch == nil)
    fmt.Printf("struct: %+v\n", person)
}
```

Zero values make Go variables predictable and safe. Reference types (slices,  
maps, pointers, channels, interfaces, functions) have `nil` as their zero  
value. Structs are initialized with zero values for all their fields.  

## Type embedding

Go supports composition through type embedding, allowing you to include one  
type within another and automatically promote the embedded type's methods.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func (p Person) Greet() string {
    return fmt.Sprintf("Hi, I'm %s", p.Name)
}

type Employee struct {
    Person    // Embedded type
    JobTitle  string
    Salary    int
}

func (e Employee) Work() string {
    return fmt.Sprintf("%s is working as %s", e.Name, e.JobTitle)
}

type Manager struct {
    Employee    // Embedded type
    TeamSize    int
    Department  string
}

func (m Manager) Manage() string {
    return fmt.Sprintf("%s manages %d people in %s", 
                       m.Name, m.TeamSize, m.Department)
}

func main() {
    emp := Employee{
        Person:   Person{Name: "Alice", Age: 30},
        JobTitle: "Developer",
        Salary:   75000,
    }
    
    mgr := Manager{
        Employee: Employee{
            Person:   Person{Name: "Bob", Age: 40},
            JobTitle: "Engineering Manager",
            Salary:   95000,
        },
        TeamSize:   5,
        Department: "Engineering",
    }
    
    // Embedded methods are promoted
    fmt.Println(emp.Greet())  // From Person
    fmt.Println(emp.Work())   // From Employee
    
    fmt.Println(mgr.Greet())  // From Person
    fmt.Println(mgr.Work())   // From Employee
    fmt.Println(mgr.Manage()) // From Manager
    
    // Direct field access
    fmt.Printf("Manager's age: %d\n", mgr.Age)
}
```

Type embedding creates an "is-a" relationship where the embedding type gains  
all the methods and fields of the embedded type. This enables powerful  
composition patterns without explicit inheritance.  

## Method types and receivers

Methods in Go are functions with special receiver arguments. You can define  
methods on any type you define in your package.  

```go
package main

import (
    "fmt"
    "math"
)

type Point struct {
    X, Y float64
}

// Value receiver
func (p Point) Distance() float64 {
    return math.Sqrt(p.X*p.X + p.Y*p.Y)
}

// Pointer receiver
func (p *Point) Scale(factor float64) {
    p.X *= factor
    p.Y *= factor
}

// Method on custom type
type Counter int

func (c Counter) String() string {
    return fmt.Sprintf("Count: %d", int(c))
}

func (c *Counter) Increment() {
    *c++
}

func (c *Counter) Add(n int) {
    *c += Counter(n)
}

func main() {
    p := Point{X: 3, Y: 4}
    fmt.Printf("Point: %+v\n", p)
    fmt.Printf("Distance from origin: %.2f\n", p.Distance())
    
    p.Scale(2)
    fmt.Printf("After scaling: %+v\n", p)
    fmt.Printf("New distance: %.2f\n", p.Distance())
    
    var counter Counter = 5
    fmt.Println(counter)
    
    counter.Increment()
    fmt.Println(counter)
    
    counter.Add(10)
    fmt.Println(counter)
}
```

Value receivers work with copies of the value, while pointer receivers work  
with the original value. Use pointer receivers when you need to modify the  
receiver or when the receiver is large and copying would be expensive.  

## Generic types

Go 1.18 introduced generics, allowing you to write functions and types that  
work with multiple types while maintaining type safety.  

```go
package main

import "fmt"

// Define a type constraint for ordered types
type Ordered interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64 | string
}

// Generic function
func Max[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

// Generic type
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    
    index := len(s.items) - 1
    item := s.items[index]
    s.items = s.items[:index]
    return item, true
}

func (s *Stack[T]) Size() int {
    return len(s.items)
}

func main() {
    // Using generic function
    fmt.Printf("Max of 10 and 20: %d\n", Max(10, 20))
    fmt.Printf("Max of 'apple' and 'banana': %s\n", Max("apple", "banana"))
    
    // Using generic type
    intStack := Stack[int]{}
    intStack.Push(10)
    intStack.Push(20)
    intStack.Push(30)
    
    fmt.Printf("Stack size: %d\n", intStack.Size())
    
    for intStack.Size() > 0 {
        if item, ok := intStack.Pop(); ok {
            fmt.Printf("Popped: %d\n", item)
        }
    }
    
    // String stack
    stringStack := Stack[string]{}
    stringStack.Push("hello")
    stringStack.Push("world")
    
    if item, ok := stringStack.Pop(); ok {
        fmt.Printf("Popped string: %s\n", item)
    }
}
```

Generics allow you to write reusable code while maintaining type safety. Type  
constraints like `comparable` and `any` specify what operations are allowed  
on the generic type parameters.  

## Type constraints and interfaces

Generic type constraints use interfaces to specify what methods or operations  
generic types must support, enabling more sophisticated generic programming.  

```go
package main

import "fmt"

// Numeric constraint interface
type Numeric interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64
}

// Generic function with numeric constraint
func Sum[T Numeric](numbers []T) T {
    var total T
    for _, num := range numbers {
        total += num
    }
    return total
}

// Interface constraint for types with String method
type Stringer interface {
    String() string
}

func PrintAll[T Stringer](items []T) {
    for i, item := range items {
        fmt.Printf("Item %d: %s\n", i, item.String())
    }
}

// Custom type that implements String method
type Product struct {
    Name  string
    Price float64
}

func (p Product) String() string {
    return fmt.Sprintf("%s ($%.2f)", p.Name, p.Price)
}

func main() {
    // Using numeric constraint
    intNumbers := []int{1, 2, 3, 4, 5}
    floatNumbers := []float64{1.1, 2.2, 3.3, 4.4, 5.5}
    
    fmt.Printf("Sum of ints: %d\n", Sum(intNumbers))
    fmt.Printf("Sum of floats: %.1f\n", Sum(floatNumbers))
    
    // Using interface constraint
    products := []Product{
        {"Laptop", 999.99},
        {"Mouse", 29.99},
        {"Keyboard", 79.99},
    }
    
    PrintAll(products)
}
```

Type constraints enable you to write generic code that operates on specific  
sets of types. Union constraints (using `|`) allow multiple types, while  
interface constraints require specific methods to be implemented.  

## Reflection and runtime type information

Go's `reflect` package provides runtime type introspection, allowing programs  
to examine types and values at runtime.  

```go
package main

import (
    "fmt"
    "reflect"
)

type Student struct {
    Name  string `json:"name"`
    Age   int    `json:"age"`
    Grade string `json:"grade"`
}

func analyzeType(v interface{}) {
    t := reflect.TypeOf(v)
    value := reflect.ValueOf(v)
    
    fmt.Printf("Type: %s\n", t.Name())
    fmt.Printf("Kind: %s\n", t.Kind())
    fmt.Printf("Size: %d bytes\n", t.Size())
    
    if t.Kind() == reflect.Struct {
        fmt.Printf("Struct with %d fields:\n", t.NumField())
        
        for i := 0; i < t.NumField(); i++ {
            field := t.Field(i)
            fieldValue := value.Field(i)
            
            fmt.Printf("  Field %d: %s (%s) = %v", 
                       i, field.Name, field.Type, fieldValue.Interface())
            
            if tag := field.Tag.Get("json"); tag != "" {
                fmt.Printf(" [json:\"%s\"]", tag)
            }
            fmt.Println()
        }
    }
    fmt.Println()
}

func main() {
    student := Student{
        Name:  "Alice Johnson",
        Age:   20,
        Grade: "A",
    }
    
    analyzeType(42)
    analyzeType("hello there")
    analyzeType([]int{1, 2, 3})
    analyzeType(student)
    analyzeType(&student)
    
    // Type checking
    var x interface{} = 42
    if reflect.TypeOf(x).Kind() == reflect.Int {
        fmt.Printf("x is an integer with value %d\n", x)
    }
}
```

Reflection is powerful but should be used sparingly as it reduces performance  
and type safety. It's commonly used in serialization libraries, ORMs, and  
testing frameworks where runtime type information is essential.  

## Type conversions and casting

Go requires explicit conversions between different types, even when they have  
the same underlying representation. This prevents accidental type mixing.  

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // Numeric conversions
    var i int = 42
    var f float64 = float64(i)
    var u uint = uint(i)
    
    fmt.Printf("int: %d, float64: %.1f, uint: %d\n", i, f, u)
    
    // String conversions
    var b byte = 65
    var r rune = 'A'
    
    fmt.Printf("byte %d as string: %s\n", b, string(b))
    fmt.Printf("rune %c as string: %s\n", r, string(r))
    
    // String to numeric conversions
    str := "123"
    if num, err := strconv.Atoi(str); err == nil {
        fmt.Printf("String '%s' as int: %d\n", str, num)
    }
    
    if f64, err := strconv.ParseFloat("3.14", 64); err == nil {
        fmt.Printf("String as float64: %.2f\n", f64)
    }
    
    // Numeric to string conversions
    fmt.Printf("Int to string: %s\n", strconv.Itoa(42))
    fmt.Printf("Float to string: %s\n", strconv.FormatFloat(3.14159, 'f', 2, 64))
    
    // Slice conversions
    bytes := []byte("hello")
    text := string(bytes)
    backToBytes := []byte(text)
    
    fmt.Printf("Bytes: %v\n", bytes)
    fmt.Printf("String: %s\n", text)
    fmt.Printf("Back to bytes: %v\n", backToBytes)
}
```

Go's explicit conversion requirement prevents bugs and makes code more readable.  
The `strconv` package provides functions for converting between strings and  
other basic types with proper error handling.  

## Slices of different types

Slices can hold elements of any type, including other slices, creating  
multi-dimensional structures useful for matrices and nested data.  

```go
package main

import "fmt"

func main() {
    // Slice of slices (2D)
    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }
    
    // Slice of structs
    type Person struct {
        Name string
        Age  int
    }
    
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35},
    }
    
    // Slice of functions
    operations := []func(int, int) int{
        func(a, b int) int { return a + b },
        func(a, b int) int { return a - b },
        func(a, b int) int { return a * b },
    }
    
    fmt.Println("Matrix:", matrix)
    fmt.Println("People:", people)
    
    for i, op := range operations {
        result := op(10, 5)
        fmt.Printf("Operation %d result: %d\n", i, result)
    }
}
```

Slices provide flexible containers for homogeneous data. Multi-dimensional  
slices are useful for mathematical computations and data processing applications.  

## Maps with complex keys and values

Maps can use any comparable type as keys and any type as values, enabling  
sophisticated data structures for complex applications.  

```go
package main

import "fmt"

type Coordinate struct {
    X, Y int
}

type Stats struct {
    Visits int
    Score  float64
}

func main() {
    // Map with struct key
    locationStats := map[Coordinate]Stats{
        {0, 0}: {Visits: 10, Score: 8.5},
        {1, 1}: {Visits: 5, Score: 9.2},
        {2, 2}: {Visits: 15, Score: 7.8},
    }
    
    // Map with slice value
    categories := map[string][]string{
        "fruits":     {"apple", "banana", "orange"},
        "vegetables": {"carrot", "broccoli", "spinach"},
        "grains":     {"rice", "wheat", "oats"},
    }
    
    // Map of maps (nested)
    inventory := map[string]map[string]int{
        "warehouse1": {"apples": 100, "oranges": 50},
        "warehouse2": {"apples": 75, "bananas": 30},
    }
    
    fmt.Println("Location stats:")
    for coord, stats := range locationStats {
        fmt.Printf("  %+v: %+v\n", coord, stats)
    }
    
    fmt.Println("Categories:", categories)
    fmt.Println("Inventory:", inventory["warehouse1"]["apples"])
}
```

Complex map structures enable powerful data modeling. Struct keys must be  
comparable (all fields must be comparable types). Nested maps are useful  
for hierarchical data organization.  

## Function closures and captured variables

Closures are functions that capture variables from their surrounding scope,  
creating powerful patterns for callbacks and state management.  

```go
package main

import "fmt"

func createCounter(initial int) func() int {
    count := initial
    return func() int {
        count++
        return count
    }
}

func createAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}

func main() {
    // Counter closure
    counter1 := createCounter(0)
    counter2 := createCounter(100)
    
    fmt.Printf("Counter1: %d, %d, %d\n", 
               counter1(), counter1(), counter1())
    fmt.Printf("Counter2: %d, %d, %d\n", 
               counter2(), counter2(), counter2())
    
    // Adder closures
    add10 := createAdder(10)
    add5 := createAdder(5)
    
    fmt.Printf("add10(3) = %d\n", add10(3))
    fmt.Printf("add5(7) = %d\n", add5(7))
    
    // Closure in loop
    var funcs []func() int
    for i := 0; i < 3; i++ {
        j := i // Capture loop variable
        funcs = append(funcs, func() int {
            return j * j
        })
    }
    
    for i, f := range funcs {
        fmt.Printf("funcs[%d]() = %d\n", i, f())
    }
}
```

Closures maintain references to variables in their enclosing scope, even after  
the outer function returns. This enables powerful patterns like factory functions  
and event handlers with embedded state.  

## Error handling with custom error types

Go's error handling uses interfaces, allowing custom error types with  
additional context and methods for better error management.  

```go
package main

import (
    "fmt"
)

// Custom error type
type ValidationError struct {
    Field   string
    Value   interface{}
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field '%s' with value '%v': %s",
                       e.Field, e.Value, e.Message)
}

type NetworkError struct {
    Operation string
    URL       string
    Err       error
}

func (e NetworkError) Error() string {
    return fmt.Sprintf("network error during %s to %s: %v", 
                       e.Operation, e.URL, e.Err)
}

func (e NetworkError) Unwrap() error {
    return e.Err
}

func validateAge(age int) error {
    if age < 0 {
        return ValidationError{
            Field:   "age",
            Value:   age,
            Message: "must be non-negative",
        }
    }
    if age > 150 {
        return ValidationError{
            Field:   "age", 
            Value:   age,
            Message: "seems unrealistic",
        }
    }
    return nil
}

func main() {
    ages := []int{25, -5, 200, 45}
    
    for _, age := range ages {
        if err := validateAge(age); err != nil {
            fmt.Printf("Error: %v\n", err)
            
            // Type assertion to access custom fields
            if ve, ok := err.(ValidationError); ok {
                fmt.Printf("  Field: %s, Value: %v\n", ve.Field, ve.Value)
            }
        } else {
            fmt.Printf("Age %d is valid\n", age)
        }
    }
}
```

Custom error types provide structured error information and can implement  
additional methods. The `Unwrap()` method supports error chain unwrapping  
for better error analysis and debugging.  

## Variadic functions and parameter types

Variadic functions accept a variable number of arguments, providing flexibility  
for functions that need to handle different numbers of inputs.  

```go
package main

import "fmt"

func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

func printf(format string, args ...interface{}) {
    fmt.Printf("Custom: "+format, args...)
}

func processItems(required string, items ...string) {
    fmt.Printf("Required: %s\n", required)
    if len(items) == 0 {
        fmt.Println("No optional items provided")
        return
    }
    
    fmt.Printf("Optional items (%d): ", len(items))
    for i, item := range items {
        if i > 0 {
            fmt.Print(", ")
        }
        fmt.Print(item)
    }
    fmt.Println()
}

func main() {
    // Variadic with integers
    fmt.Printf("Sum of 1,2,3: %d\n", sum(1, 2, 3))
    fmt.Printf("Sum of 10,20: %d\n", sum(10, 20))
    fmt.Printf("Sum of nothing: %d\n", sum())
    
    // Using slice with variadic
    numbers := []int{5, 10, 15, 20}
    fmt.Printf("Sum from slice: %d\n", sum(numbers...))
    
    // Variadic with interface{}
    printf("Hello %s, you are %d years old\n", "Alice", 30)
    
    // Mixed required and variadic parameters
    processItems("config.yaml")
    processItems("data.json", "backup1.json", "backup2.json")
}
```

The `...` syntax defines variadic parameters, which are received as slices  
within the function. When calling variadic functions with slices, use `...`  
to expand the slice elements as individual arguments.  

## Defer, panic, and recover

Go provides `defer`, `panic`, and `recover` for resource cleanup and  
error handling in exceptional situations.  

```go
package main

import (
    "fmt"
    "os"
)

func cleanup() {
    fmt.Println("Cleaning up resources...")
}

func riskyOperation() {
    defer cleanup() // Always executed
    
    fmt.Println("Starting risky operation...")
    
    // Simulate a panic
    panic("something went wrong!")
}

func safeWrapper() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Printf("Recovered from panic: %v\n", r)
        }
    }()
    
    riskyOperation()
    fmt.Println("This line won't be reached")
}

func fileOperation(filename string) {
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Failed to create file: %v\n", err)
        return
    }
    
    // Defer ensures file is closed even if panic occurs
    defer func() {
        if err := file.Close(); err != nil {
            fmt.Printf("Error closing file: %v\n", err)
        }
        fmt.Printf("File %s closed\n", filename)
    }()
    
    fmt.Printf("Working with file %s\n", filename)
    
    // Write some data
    if _, err := file.WriteString("Hello, World!"); err != nil {
        panic(fmt.Sprintf("Write failed: %v", err))
    }
}

func main() {
    fmt.Println("=== Defer and Panic Example ===")
    safeWrapper()
    fmt.Println("Program continues after recovery")
    
    fmt.Println("\n=== File Operation with Defer ===")
    fileOperation("/tmp/example.txt")
    
    fmt.Println("\nProgram completed successfully")
}
```

`defer` schedules function calls to run when the surrounding function returns.  
`panic` stops normal execution, while `recover` can catch panics in deferred  
functions. Use sparingly - prefer explicit error handling over panic/recover.  

## Goroutines and channel communication

Goroutines are lightweight threads managed by the Go runtime. Channels  
provide safe communication between goroutines without shared memory.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job)
        time.Sleep(100 * time.Millisecond) // Simulate work
        results <- job * 2
    }
}

func pipeline() {
    // Stage 1: Generate numbers
    numbers := make(chan int, 5)
    go func() {
        defer close(numbers)
        for i := 1; i <= 5; i++ {
            numbers <- i
            time.Sleep(50 * time.Millisecond)
        }
    }()
    
    // Stage 2: Square numbers
    squares := make(chan int, 5)
    go func() {
        defer close(squares)
        for num := range numbers {
            squares <- num * num
            fmt.Printf("Squared %d = %d\n", num, num*num)
        }
    }()
    
    // Stage 3: Sum all squares
    total := 0
    for square := range squares {
        total += square
    }
    
    fmt.Printf("Sum of squares: %d\n", total)
}

func main() {
    fmt.Println("=== Worker Pool Pattern ===")
    
    jobs := make(chan int, 10)
    results := make(chan int, 10)
    
    var wg sync.WaitGroup
    
    // Start workers
    numWorkers := 3
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, jobs, results, &wg)
    }
    
    // Send jobs
    go func() {
        for i := 1; i <= 5; i++ {
            jobs <- i
        }
        close(jobs)
    }()
    
    // Close results when all workers done
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect results
    for result := range results {
        fmt.Printf("Result: %d\n", result)
    }
    
    fmt.Println("\n=== Pipeline Pattern ===")
    pipeline()
}
```

Goroutines enable concurrent programming with minimal overhead. Channels  
provide type-safe communication and synchronization. Use buffered channels  
for asynchronous communication and unbuffered for synchronous handoffs.  
