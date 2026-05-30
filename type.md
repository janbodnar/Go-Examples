# The Go Type System

## Introduction

Go is a statically typed language: every variable, constant, function parameter, and return value has a type that is known at compile time. The compiler uses type information to catch entire classes of bugs before the program ever runs.

### Named vs. Unnamed Types

A **named type** is introduced with the `type` keyword and has a distinct name (`int`, `string`, user-defined `type Celsius float64`). An **unnamed (composite) type** is written as a type literal—`[]int`, `map[string]bool`, `*User`—and has no name of its own.

### Underlying Types and Type Identity

Every type has an **underlying type**. For predeclared types such as `int` or `string`, the underlying type is the type itself. For a declared type `type Celsius float64`, the underlying type is `float64`. Two types are **identical** only if they are the same named type, or both are unnamed types with identical structure and element types.

### Zero Values

Every type has a **zero value**—the default when a variable is declared without an explicit initializer. Numeric types zero to `0`, `bool` to `false`, `string` to `""`, and all pointer/slice/map/channel/function/interface types to `nil`.

### Comparison with Other Languages

| Feature | Go | C | Java | Python |
|---|---|---|---|---|
| Type checking | Static, compile-time | Static, compile-time | Static, compile-time | Dynamic, runtime |
| Typing style | Structural (interfaces) | Nominal | Nominal | Duck typing |
| Implicit casting | No | Yes (arithmetic) | Widening only | Yes |
| Generics | Yes (1.18+) | No (macros) | Yes | Yes |

Go interfaces are satisfied **structurally**: a type need not declare that it implements an interface—it just needs to have the right methods. This is unlike Java's `implements` keyword.

```go
package main

import "fmt"

type Celsius float64
type Fahrenheit float64

func main() {
    var c Celsius    // zero value
    var f Fahrenheit // zero value
    fmt.Println(c, f)
    // Output: 0 0
}
```

```go
package main

import "fmt"

func main() {
    var i int
    var s string
    var b bool
    var p *int
    fmt.Printf("int=%d string=%q bool=%v pointer=%v\n", i, s, b, p)
    // Output: int=0 string="" bool=false pointer=<nil>
}
```

---

## Basic Types

Go's predeclared basic types cover booleans, all numeric sizes, and strings.

| Category | Types |
|---|---|
| Boolean | `bool` |
| Integer | `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` |
| Float | `float32`, `float64` |
| Complex | `complex64`, `complex128` |
| String | `string` |
| Aliases | `byte` = `uint8`, `rune` = `int32` |

### byte and rune

`byte` is an alias for `uint8` and represents a raw octet of data. `rune` is an alias for `int32` and represents a Unicode code point. Using `rune` makes intent explicit when working with text.

```go
package main

import "fmt"

func main() {
    var b byte = 'A'
    var r rune = '⌘' // U+2318 PLACE OF INTEREST SIGN

    fmt.Printf("byte value: %d  char: %c\n", b, b)
    fmt.Printf("rune value: %d  char: %c\n", r, r)
    // Output:
    // byte value: 65  char: A
    // rune value: 8984  char: ⌘
}
```

### Zero Values for Basic Types

```go
package main

import "fmt"

func main() {
    var i int
    var f float64
    var b bool
    var s string
    var by byte
    var r rune

    fmt.Printf("int:%d float64:%f bool:%v string:%q byte:%d rune:%d\n",
        i, f, b, s, by, r)
    // Output: int:0 float64:0.000000 bool:false string:"" byte:0 rune:0
}
```

---

## Composite Types

### Arrays vs. Slices

An **array** has a fixed length that is part of its type: `[3]int` and `[4]int` are different types. Arrays are value types—assigning copies all elements.

A **slice** is a lightweight descriptor containing three fields: a pointer to an underlying array, a length (`len`), and a capacity (`cap`). Slices are reference types; assigning a slice copies the header, not the data.

```go
package main

import "fmt"

func main() {
    arr := [3]int{1, 2, 3} // array – fixed size
    sl := []int{4, 5, 6}   // slice – dynamic

    fmt.Printf("array: %v  len=%d\n", arr, len(arr))
    fmt.Printf("slice: %v  len=%d cap=%d\n", sl, len(sl), cap(sl))

    sl = append(sl, 7)
    fmt.Printf("after append: %v  len=%d cap=%d\n", sl, len(sl), cap(sl))
    // Output:
    // array: [1 2 3]  len=3
    // slice: [4 5 6]  len=3 cap=3
    // after append: [4 5 6 7]  len=4 cap=6
}
```

### Maps

A map is an unordered collection of key-value pairs. Keys must be **comparable** (support `==`): booleans, numbers, strings, pointers, arrays, and structs with only comparable fields qualify. Slices, maps, and functions are not comparable and cannot be keys.

```go
package main

import "fmt"

func main() {
    scores := map[string]int{
        "Alice": 95,
        "Bob":   87,
    }
    scores["Carol"] = 92

    for name, score := range scores {
        fmt.Printf("%s: %d\n", name, score)
    }

    val, ok := scores["Dave"]
    fmt.Printf("Dave: %d, found: %v\n", val, ok)
    // Output (order may vary):
    // Alice: 95
    // Bob: 87
    // Carol: 92
    // Dave: 0, found: false
}
```

### Structs

A struct groups named fields of arbitrary types into a single composite type.

```go
package main

import "fmt"

type Point struct {
    X, Y float64
}

func main() {
    p := Point{X: 3.0, Y: 4.0}
    fmt.Printf("Point: %+v\n", p)
    fmt.Printf("X=%.1f Y=%.1f\n", p.X, p.Y)
    // Output:
    // Point: {X:3 Y:4}
    // X=3.0 Y=4.0
}
```

### Pointers

A pointer holds the memory address of a value. `*T` is the type "pointer to T". The zero value of any pointer is `nil`. Use `&` to take the address of a value and `*` to dereference it.

```go
package main

import "fmt"

func increment(n *int) {
    *n++
}

func main() {
    x := 10
    increment(&x)
    fmt.Println(x) // Output: 11
}
```

### Functions as Types

Functions are first-class values in Go. A function's type is determined by its parameter and return types, not its name.

```go
package main

import "fmt"

type BinaryOp func(int, int) int

func apply(op BinaryOp, a, b int) int {
    return op(a, b)
}

func main() {
    add := func(a, b int) int { return a + b }
    mul := func(a, b int) int { return a * b }

    fmt.Println(apply(add, 3, 4)) // Output: 7
    fmt.Println(apply(mul, 3, 4)) // Output: 12
}
```

### Channels

A channel is a typed conduit for communication between goroutines. Its type encodes the element type and direction: `chan T` (bidirectional), `<-chan T` (receive-only), `chan<- T` (send-only). The zero value of a channel is `nil`.

```go
package main

import "fmt"

func main() {
    ch := make(chan int, 1) // buffered channel of int
    ch <- 42
    v := <-ch
    fmt.Println(v) // Output: 42
}
```

### Interfaces

An interface type specifies a **method set**. Any type that implements all listed methods satisfies the interface—no explicit declaration is needed.

```go
package main

import (
    "fmt"
    "math"
)

type Shape interface {
    Area() float64
}

type Circle struct{ Radius float64 }
type Rectangle struct{ Width, Height float64 }

func (c Circle) Area() float64    { return math.Pi * c.Radius * c.Radius }
func (r Rectangle) Area() float64 { return r.Width * r.Height }

func printArea(s Shape) {
    fmt.Printf("%.2f\n", s.Area())
}

func main() {
    printArea(Circle{Radius: 5})
    printArea(Rectangle{Width: 4, Height: 3})
    // Output:
    // 78.54
    // 12.00
}
```

---

## Type Definitions & Aliases

### Type Definitions: `type X Y`

`type X Y` creates a **new named type** `X` with the same underlying type as `Y`. `X` and `Y` are distinct types—values cannot be mixed without explicit conversion. This enables domain modeling and attaching methods.

```go
package main

import "fmt"

type UserID int64
type Email string
type Money float64

func (m Money) String() string {
    return fmt.Sprintf("$%.2f", float64(m))
}

func main() {
    var id UserID = 42
    var addr Email = "alice@example.com"
    price := Money(9.99)

    fmt.Println(id, addr, price)
    // Output: 42 alice@example.com $9.99
}
```

### Type Aliases: `type X = Y`

`type X = Y` introduces an **alias**: `X` and `Y` are exactly the same type. Aliases are primarily used for code migration and to expose a type under a different package name.

```go
package main

import "fmt"

type MyString = string // alias – identical to string

func greet(s MyString) string {
    return "Hello, " + s
}

func main() {
    var name string = "Go"
    fmt.Println(greet(name)) // no conversion needed
    // Output: Hello, Go
}
```

### When to Use Each

| Use case | Mechanism |
|---|---|
| Domain modeling, adding methods, type safety | `type X Y` (definition) |
| Refactoring: rename a type across packages | `type X = Y` (alias) |
| Provide a shorter name for a complex type | `type X = Y` (alias) |

---

## Method Sets

The **method set** of a type determines which methods can be called on values of that type and, consequently, which interfaces those values satisfy.

| Type | Method set |
|---|---|
| `T` | Methods with value receiver `(t T)` |
| `*T` | Methods with value receiver `(t T)` **and** pointer receiver `(t *T)` |

A value of type `T` can only call value-receiver methods. A pointer `*T` can call both. This rule determines whether a value or a pointer satisfies an interface.

```go
package main

import "fmt"

type Counter struct{ n int }

func (c Counter) Value() int  { return c.n }       // value receiver
func (c *Counter) Inc()       { c.n++ }             // pointer receiver

func main() {
    c := Counter{}
    c.Inc()   // Go auto-takes address: (&c).Inc()
    c.Inc()
    fmt.Println(c.Value()) // Output: 2
}
```

```go
package main

import "fmt"

type Stringer interface {
    String() string
}

type Name struct{ First, Last string }

// Only *Name has this method (pointer receiver).
func (n *Name) String() string {
    return n.First + " " + n.Last
}

func print(s Stringer) { fmt.Println(s.String()) }

func main() {
    n := &Name{First: "Ada", Last: "Lovelace"}
    print(n) // *Name satisfies Stringer
    // Output: Ada Lovelace

    // print(Name{...}) would not compile:
    // Name does not implement Stringer (String method has pointer receiver)
}
```

---

## Interface Satisfaction

### Implicit Implementation

Go uses **structural typing**: a type satisfies an interface automatically when it provides all required methods. There is no `implements` keyword.

```go
package main

import "fmt"

type Animal interface {
    Sound() string
}

type Dog struct{}
type Cat struct{}

func (d Dog) Sound() string { return "Woof" }
func (c Cat) Sound() string { return "Meow" }

func speak(a Animal) { fmt.Println(a.Sound()) }

func main() {
    speak(Dog{})
    speak(Cat{})
    // Output:
    // Woof
    // Meow
}
```

### The Empty Interface and `any`

`interface{}` (aliased as `any` since Go 1.18) has an empty method set and is satisfied by every type. Use it only when you genuinely need to hold values of unknown type.

### Type Assertions and Type Switches

A **type assertion** `x.(T)` extracts the concrete value stored in an interface. If `x` does not hold a `T`, the single-return form panics; the two-return form sets `ok` to `false`.

A **type switch** selects a case based on the dynamic type of an interface value.

```go
package main

import "fmt"

func describe(i any) {
    switch v := i.(type) {
    case int:
        fmt.Printf("int: %d\n", v)
    case string:
        fmt.Printf("string: %q\n", v)
    case bool:
        fmt.Printf("bool: %v\n", v)
    default:
        fmt.Printf("unknown: %T\n", v)
    }
}

func main() {
    describe(42)
    describe("hello")
    describe(true)
    describe(3.14)
    // Output:
    // int: 42
    // string: "hello"
    // bool: true
    // unknown: float64
}
```

```go
package main

import "fmt"

func main() {
    var i any = "Go"

    s, ok := i.(string) // safe two-value assertion
    fmt.Printf("value=%q ok=%v\n", s, ok)

    n, ok := i.(int)
    fmt.Printf("value=%d ok=%v\n", n, ok)
    // Output:
    // value="Go" ok=true
    // value=0 ok=false
}
```

---

## Type Conversions

Go **never performs implicit conversions**. Every conversion between distinct types must be written explicitly as `T(x)`. This avoids silent precision loss and makes data flow obvious.

### Numeric Conversions

```go
package main

import "fmt"

func main() {
    i := 42
    f := float64(i)  // int → float64
    u := uint(f)     // float64 → uint (truncates)

    fmt.Printf("int=%d float64=%f uint=%d\n", i, f, u)
    // Output: int=42 float64=42.000000 uint=42
}
```

### Converting Between Types with the Same Underlying Type

Two named types that share the same underlying type can be converted into each other explicitly.

```go
package main

import "fmt"

type Meters float64
type Feet float64

const feetPerMeter = 3.28084

func toFeet(m Meters) Feet {
    return Feet(float64(m) * feetPerMeter)
}

func main() {
    dist := Meters(10)
    fmt.Printf("%.2f m = %.2f ft\n", dist, toFeet(dist))
    // Output: 10.00 m = 32.81 ft
}
```

### Untyped Constants and Default Types

An **untyped constant** has a default type that is applied when the constant is assigned to a variable without an explicit type. For example, `42` defaults to `int`, `3.14` to `float64`, `'A'` to `rune`, and `"hi"` to `string`.

```go
package main

import "fmt"

func main() {
    const n = 100       // untyped integer constant
    const pi = 3.14159  // untyped floating-point constant

    var x float64 = n   // n adapts to float64
    var y int = n       // n adapts to int

    fmt.Printf("float64: %f  int: %d  pi: %f\n", x, y, pi)
    // Output: float64: 100.000000  int: 100  pi: 3.141590
}
```

---

## Generics & Constraints (Type Parameters)

Go 1.18 introduced **type parameters**, enabling functions and types to operate on a range of types while retaining full static type safety.

### Syntax

```
func FuncName[T Constraint](param T) T { ... }
```

### Built-in Constraints

| Constraint | Meaning |
|---|---|
| `any` | Any type (`interface{}`) |
| `comparable` | Types that support `==` and `!=` |

### Custom Constraints

A constraint is an interface. It may include a **type element** (a union of concrete types) to restrict the permitted set.

```go
package main

import "fmt"

type Number interface {
    ~int | ~int32 | ~int64 | ~float32 | ~float64
}

func Sum[T Number](values []T) T {
    var total T
    for _, v := range values {
        total += v
    }
    return total
}

func main() {
    ints := []int{1, 2, 3, 4, 5}
    floats := []float64{1.1, 2.2, 3.3}

    fmt.Println(Sum(ints))   // Output: 15
    fmt.Println(Sum(floats)) // Output: 6.6
}
```

### Generic Functions

```go
package main

import "fmt"

func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

func Filter[T any](s []T, f func(T) bool) []T {
    var result []T
    for _, v := range s {
        if f(v) {
            result = append(result, v)
        }
    }
    return result
}

func main() {
    nums := []int{1, 2, 3, 4, 5}

    doubled := Map(nums, func(n int) int { return n * 2 })
    fmt.Println(doubled) // Output: [2 4 6 8 10]

    evens := Filter(nums, func(n int) bool { return n%2 == 0 })
    fmt.Println(evens) // Output: [2 4]

    strs := Map(nums, func(n int) string { return fmt.Sprintf("%d!", n) })
    fmt.Println(strs) // Output: [1! 2! 3! 4! 5!]
}
```

### Min with `comparable` + ordered constraint

```go
package main

import "fmt"

type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
        ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
        ~float32 | ~float64 | ~string
}

func Min[T Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

func main() {
    fmt.Println(Min(3, 7))         // Output: 3
    fmt.Println(Min(3.14, 2.71))   // Output: 2.71
    fmt.Println(Min("apple", "banana")) // Output: apple
}
```

---

## Reflection (Brief Overview)

The `reflect` package lets programs inspect and manipulate types and values at runtime. It is powerful but costly—reflection bypasses compile-time checks and is slower than direct code.

### `reflect.Type` and `reflect.Kind`

`reflect.TypeOf(x)` returns the `reflect.Type` of `x`—its full named type (e.g., `main.Celsius`). `reflect.Kind` is a coarser classification: the underlying category such as `reflect.Float64`, `reflect.Struct`, or `reflect.Slice`.

```go
package main

import (
    "fmt"
    "reflect"
)

type Celsius float64

type Point struct {
    X, Y float64
}

func main() {
    c := Celsius(100)
    p := Point{1, 2}
    sl := []int{1, 2, 3}

    for _, v := range []any{c, p, sl} {
        t := reflect.TypeOf(v)
        fmt.Printf("Type: %-20s Kind: %s\n", t, t.Kind())
    }
    // Output:
    // Type: main.Celsius        Kind: float64
    // Type: main.Point          Kind: struct
    // Type: []int               Kind: slice
}
```

### Inspecting Struct Fields

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name  string
    Email string
    Age   int
}

func main() {
    u := User{Name: "Alice", Email: "alice@example.com", Age: 30}
    t := reflect.TypeOf(u)
    v := reflect.ValueOf(u)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("%-8s = %v\n", field.Name, value)
    }
    // Output:
    // Name     = Alice
    // Email    = alice@example.com
    // Age      = 30
}
```

### When to Use Reflection

Use reflection sparingly. Prefer it only when the type is genuinely unknown at compile time—for example, when writing serialization libraries, test helpers, or framework-level code. For all other cases, use generics or interface-based polymorphism: they are faster, safer, and more readable.
