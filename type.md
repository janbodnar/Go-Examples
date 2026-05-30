# The Go Type System

## Introduction

Go is a statically typed language: every variable, constant, function parameter, and return value has a type that is known at compile time. The compiler uses type information to catch entire classes of bugs before the program ever runs, without the runtime overhead of dynamic type checking.

### Named vs. Unnamed Types

A **named type** is introduced with the `type` keyword and has a distinct name: the predeclared types `int`, `string`, `bool`, and user-defined types like `type Celsius float64`. An **unnamed (composite) type** is written as a type literal—`[]int`, `map[string]bool`, `*User`—and has no name of its own.

### Underlying Types and Type Identity

Every type has an **underlying type**. For predeclared types such as `int` or `string`, the underlying type is the type itself. For a declared type `type Celsius float64`, the underlying type is `float64`. Two types are **identical** only if they are the same named type, or both are unnamed types with identical structure and element types. Two types are **assignable** when they share the same underlying type and at least one of them is unnamed.

### Zero Values

Every type has a **zero value**—the default when a variable is declared without an explicit initializer. Numeric types zero to `0`, `bool` to `false`, `string` to `""`, and all pointer/slice/map/channel/function/interface types to `nil`. Structs and arrays zero-initialize each field/element recursively.

### Comparison with Other Languages

| Feature | Go | C | Java | Python |
|---|---|---|---|---|
| Type checking | Static, compile-time | Static, compile-time | Static, compile-time | Dynamic, runtime |
| Typing style | Structural (interfaces) | Nominal | Nominal | Duck typing |
| Implicit casting | No | Yes (arithmetic) | Widening only | Yes |
| Generics | Yes (1.18+) | No (macros) | Yes | Yes |

Go interfaces are satisfied **structurally**: a type need not declare that it implements an interface—it just needs to have the right methods. This is unlike Java's `implements` keyword or Rust's `impl Trait for Type`.

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

## Basic Types

Go's predeclared basic types cover booleans, all numeric widths, and strings.

| Category | Types |
|---|---|
| Boolean | `bool` |
| Integer | `int`, `int8`, `int16`, `int32`, `int64`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr` |
| Float | `float32`, `float64` |
| Complex | `complex64`, `complex128` |
| String | `string` |
| Aliases | `byte` = `uint8`, `rune` = `int32` |

### Platform-Dependent Integer Types

`int`, `uint`, and `uintptr` are sized to match the platform's native word: 32 bits on 32-bit systems and 64 bits on 64-bit systems. Use `int` as the default integer type for general-purpose values; use the explicitly sized variants (`int32`, `int64`) when a specific bit width is required—for example, when encoding binary data or interfacing with C code via `cgo`.

### byte and rune

`byte` is an alias for `uint8` and represents a raw octet of data. `rune` is an alias for `int32` and represents a Unicode code point. Go source code is always UTF-8; iterating over a string with `range` yields `rune` values, not bytes.

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

### Strings, Bytes, and Runes

A `string` in Go is an **immutable sequence of bytes**. The `len` built-in returns the number of bytes, not the number of Unicode characters. To work with characters, convert to `[]rune`. To work with raw data, convert to `[]byte`.

```go
package main

import "fmt"

func main() {
    s := "café"

    fmt.Println("bytes:", len(s))             // Output: bytes: 5 (é is 2 bytes in UTF-8)
    fmt.Println("runes:", len([]rune(s)))      // Output: runes: 4

    // Iterating by byte index
    for i := 0; i < len(s); i++ {
        fmt.Printf("byte[%d] = %d\n", i, s[i])
    }

    // Iterating by rune (Unicode code point)
    for i, r := range s {
        fmt.Printf("rune[%d] = %c (%d)\n", i, r, r)
    }
}
```

Converting between strings, byte slices, and rune slices always copies the data, since strings are immutable and slices are mutable:

```go
package main

import "fmt"

func main() {
    s := "hello"

    bs := []byte(s)   // string → []byte
    rs := []rune(s)   // string → []rune

    bs[0] = 'H'
    s2 := string(bs)  // []byte → string

    fmt.Println(s2)          // Output: Hello
    fmt.Println(len(rs))     // Output: 5
    fmt.Println(string(rs))  // Output: hello (original unchanged)
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

## Constants and iota

Constants are values that are fixed at compile time. They can be **typed** or **untyped**.

### Typed vs. Untyped Constants

A **typed constant** has an explicit type. An **untyped constant** has a *kind* (integer, float, string, etc.) but no concrete type until it is used in a context that demands one. Untyped constants are more flexible: the literal `1` can be assigned to any numeric variable without conversion.

```go
package main

import "fmt"

const typedPi float64 = 3.14159 // typed: always float64
const untypedPi = 3.14159       // untyped: adapts to the context

func main() {
    var f32 float32 = untypedPi // OK – adapts
    var f64 float64 = untypedPi // OK – adapts
    // var f32b float32 = typedPi // ERROR: cannot use float64 as float32

    fmt.Println(f32, f64)
}
```

### iota

`iota` is a predeclared identifier that is reset to `0` at the start of each `const` block and increments by 1 for each constant in that block. It is the idiomatic way to define sequential integer constants in Go.

```go
package main

import "fmt"

type Weekday int

const (
    Sunday Weekday = iota // 0
    Monday                // 1
    Tuesday               // 2
    Wednesday             // 3
    Thursday              // 4
    Friday                // 5
    Saturday              // 6
)

func (d Weekday) String() string {
    names := [...]string{"Sunday", "Monday", "Tuesday", "Wednesday",
        "Thursday", "Friday", "Saturday"}
    if d < Sunday || d > Saturday {
        return fmt.Sprintf("Weekday(%d)", int(d))
    }
    return names[d]
}

func main() {
    fmt.Println(Monday, Friday) // Output: Monday Friday
}
```

`iota` can be used in expressions to create non-sequential or bitmask constants:

```go
package main

import "fmt"

type Permission uint

const (
    Read    Permission = 1 << iota // 1
    Write                          // 2
    Execute                        // 4
)

func main() {
    perm := Read | Write
    fmt.Printf("perm=%b  read=%v  execute=%v\n",
        perm, perm&Read != 0, perm&Execute != 0)
    // Output: perm=11  read=true  execute=false
}
```

## Composite Types

### Arrays

An **array** has a fixed length that is part of its type: `[3]int` and `[4]int` are different, incompatible types. Arrays are **value types**—assigning or passing an array copies every element. This makes arrays predictable but expensive for large sizes; prefer slices in practice.

```go
package main

import "fmt"

func main() {
    a := [3]int{1, 2, 3}
    b := a        // full copy
    b[0] = 99
    fmt.Println(a) // Output: [1 2 3]  (a is unchanged)
    fmt.Println(b) // Output: [99 2 3]
}
```

### Slices

A **slice** is a lightweight descriptor of three fields: a pointer to an underlying array, a length (`len`), and a capacity (`cap`). Slices are **reference types**; assigning a slice copies only the header—both variables share the same backing array.

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

A **nil slice** (`var s []int`) is distinct from an **empty slice** (`s := []int{}`). Both have length 0 and both work with `append`, but only a nil slice compares equal to `nil`. Prefer nil slices as zero values; create empty slices when you need to marshal to JSON as `[]` rather than `null`.

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    var nilSlice []int
    emptySlice := []int{}

    n, _ := json.Marshal(nilSlice)
    e, _ := json.Marshal(emptySlice)

    fmt.Println(string(n)) // Output: null
    fmt.Println(string(e)) // Output: []
}
```

### Maps

A map is an unordered collection of key-value pairs backed by a hash table. Keys must be **comparable** (support `==`): booleans, numbers, strings, pointers, arrays, and structs composed entirely of comparable fields qualify. Slices, maps, and functions cannot be keys.

Reading a missing key returns the element type's zero value. The two-value lookup form distinguishes "absent" from "present with zero value":

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

A **nil map** reads safely (returns zero values) but panics on write. Always initialize a map with `make` or a composite literal before inserting keys.

```go
var m map[string]int  // nil map – reads OK, write panics
m = make(map[string]int)
m["key"] = 1          // safe after make
```

### Structs

A struct groups named fields of arbitrary types into a single composite value. Structs are value types; assigning copies all fields.

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

### Struct Tags

Fields can carry **struct tags**: raw string literals that follow the field declaration. Tags are metadata used at runtime by packages such as `encoding/json`, `encoding/xml`, and `database/sql` to control serialization.

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age,omitempty"` // omitted when zero
    pass  string // unexported – never serialized
}

func main() {
    u := User{Name: "Alice", Email: "alice@example.com"}
    data, _ := json.Marshal(u)
    fmt.Println(string(data))
    // Output: {"name":"Alice","email":"alice@example.com"}
}
```

### Struct Embedding

Go achieves **composition** through embedding: placing a type inside a struct without giving it an explicit field name. The embedded type's fields and methods are **promoted** to the outer struct.

```go
package main

import "fmt"

type Animal struct {
    Name string
}

func (a Animal) Speak() string {
    return a.Name + " makes a sound"
}

type Dog struct {
    Animal        // embedded – fields and methods promoted
    Breed  string
}

func (d Dog) Speak() string { // overrides Animal.Speak
    return d.Name + " barks"
}

func main() {
    d := Dog{Animal: Animal{Name: "Rex"}, Breed: "Labrador"}
    fmt.Println(d.Name)    // promoted field
    fmt.Println(d.Speak()) // Dog's method shadows Animal's
    fmt.Println(d.Animal.Speak()) // explicit access to Animal's method
    // Output:
    // Rex
    // Rex barks
    // Rex makes a sound
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

Go has no pointer arithmetic. Unlike C, you cannot add or subtract integers from a pointer—this prevents an entire class of memory safety bugs.

### Functions as Types

Functions are first-class values in Go. A function's type is determined solely by its parameter and return types, not its name. This enables higher-order functions, callbacks, and the strategy pattern.

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

A channel is a typed conduit for goroutine communication. Its type encodes the element type and direction: `chan T` (bidirectional), `<-chan T` (receive-only), `chan<- T` (send-only). The zero value of a channel is `nil`; a receive from a nil channel blocks forever.

```go
package main

import "fmt"

func main() {
    ch := make(chan int, 1) // buffered channel of int, capacity 1
    ch <- 42
    v := <-ch
    fmt.Println(v) // Output: 42
}
```

Directional channel types are used in function signatures to constrain how a goroutine may use a channel—providing type-safe communication contracts:

```go
func produce(out chan<- int) { out <- 1 }  // may only send
func consume(in <-chan int)  { <-in }      // may only receive
```

## Interfaces

An interface type specifies a **method set**. Any concrete type that implements all listed methods satisfies the interface—no explicit declaration is needed (structural typing).

```go
package main

import (
    "fmt"
    "math"
)

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Circle struct{ Radius float64 }
type Rectangle struct{ Width, Height float64 }

func (c Circle) Area() float64      { return math.Pi * c.Radius * c.Radius }
func (c Circle) Perimeter() float64 { return 2 * math.Pi * c.Radius }

func (r Rectangle) Area() float64      { return r.Width * r.Height }
func (r Rectangle) Perimeter() float64 { return 2 * (r.Width + r.Height) }

func describe(s Shape) {
    fmt.Printf("area=%.2f  perimeter=%.2f\n", s.Area(), s.Perimeter())
}

func main() {
    describe(Circle{Radius: 5})
    describe(Rectangle{Width: 4, Height: 3})
    // Output:
    // area=78.54  perimeter=31.42
    // area=12.00  perimeter=14.00
}
```

### Interface Composition

Interfaces can be composed from other interfaces using embedding. This encourages building small, focused interfaces and combining them when needed.

```go
package main

import "fmt"

type Reader interface {
    Read() string
}

type Writer interface {
    Write(s string)
}

type ReadWriter interface {
    Reader
    Writer
}

type Buffer struct{ data string }

func (b *Buffer) Read() string      { return b.data }
func (b *Buffer) Write(s string)    { b.data = s }

func roundtrip(rw ReadWriter) {
    rw.Write("hello")
    fmt.Println(rw.Read())
}

func main() {
    buf := &Buffer{}
    roundtrip(buf) // Output: hello
}
```

### The empty interface and `any`

`interface{}` (aliased as `any` since Go 1.18) has an empty method set and is satisfied by every type. It is used when the type is genuinely unknown—for example, in JSON unmarshaling or fmt functions. Prefer concrete types or generics over `any` where possible.

### The `error` Interface

`error` is a built-in interface with a single method:

```go
type error interface {
    Error() string
}
```

Any type with an `Error() string` method satisfies `error`. This allows custom error types that carry additional context:

```go
package main

import "fmt"

type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on %s: %s", e.Field, e.Message)
}

func validate(name string) error {
    if name == "" {
        return &ValidationError{Field: "name", Message: "must not be empty"}
    }
    return nil
}

func main() {
    if err := validate(""); err != nil {
        fmt.Println(err)
        // Output: validation error on name: must not be empty

        var ve *ValidationError
        if ok := false; !ok {
            // Type assertion to access the concrete type:
            if v, ok := err.(*ValidationError); ok {
                fmt.Printf("field=%s\n", v.Field)
                // Output: field=name
            }
        }
    }
}
```

## Type Definitions & Aliases

### Type Definitions: `type X Y`

`type X Y` creates a **new named type** `X` with the same underlying type as `Y`. `X` and `Y` are **distinct types**—values cannot be mixed without explicit conversion. This enables domain modeling and attaching methods to any type.

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

### Enum-Like Patterns with `iota`

The combination of a type definition and `iota` is Go's idiomatic substitute for enums:

```go
package main

import "fmt"

type Direction int

const (
    North Direction = iota
    East
    South
    West
)

func (d Direction) String() string {
    return [...]string{"North", "East", "South", "West"}[d]
}

func main() {
    d := North
    fmt.Println(d)       // Output: North
    fmt.Println(d == 0)  // Output: true
}
```

### Type Aliases: `type X = Y`

`type X = Y` introduces an **alias**: `X` and `Y` are exactly the same type. Aliases are primarily used for large-scale refactoring (moving a type to a new package while keeping the old name available) and for re-exporting types from a package.

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
| Expose a type under a different package name | `type X = Y` (alias) |
| Enum-like constants | `type X Y` + `iota` |

## Method Sets

The **method set** of a type determines which methods can be called on a value of that type and, consequently, which interfaces it satisfies.

| Type | Method set |
|---|---|
| `T` | Methods with value receiver `(t T)` |
| `*T` | Methods with value receiver `(t T)` **and** pointer receiver `(t *T)` |

A value of type `T` can only call value-receiver methods. A pointer `*T` can call both. This asymmetry matters for interface satisfaction: if any method has a pointer receiver, only `*T` satisfies the interface, not `T`.

```go
package main

import "fmt"

type Counter struct{ n int }

func (c Counter) Value() int { return c.n }  // value receiver
func (c *Counter) Inc()      { c.n++ }       // pointer receiver

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

func (n *Name) String() string { // pointer receiver
    return n.First + " " + n.Last
}

func print(s Stringer) { fmt.Println(s.String()) }

func main() {
    n := &Name{First: "Ada", Last: "Lovelace"}
    print(n) // *Name satisfies Stringer
    // Output: Ada Lovelace

    // print(Name{...}) does not compile:
    // Name does not implement Stringer (String method has pointer receiver)
}
```

### Promoted Methods from Embedding

When a type is embedded, its method set is promoted to the outer struct. The outer type can then satisfy interfaces through its embedded fields:

```go
package main

import (
    "fmt"
    "strings"
)

type Logger struct{ prefix string }

func (l Logger) Log(msg string) {
    fmt.Printf("[%s] %s\n", strings.ToUpper(l.prefix), msg)
}

type Server struct {
    Logger        // promotes Log method
    addr   string
}

func main() {
    s := Server{Logger: Logger{prefix: "server"}, addr: ":8080"}
    s.Log("starting") // calls s.Logger.Log("starting")
    // Output: [SERVER] starting
}
```

## Interface Satisfaction

### Implicit Implementation

Go uses **structural typing**: a type satisfies an interface automatically when it provides all required methods. There is no `implements` keyword, and no coupling between the type and the interface's package.

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

### Type Assertions

A **type assertion** `x.(T)` extracts the concrete value stored in an interface. If `x` does not hold a `T`, the single-return form panics; the two-return form sets `ok` to `false` and does not panic.

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

### Type Switches

A **type switch** selects a branch based on the dynamic type of an interface value. It is cleaner than chaining type assertions.

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

### The Nil Interface Pitfall

An interface value has two internal components: a dynamic type and a dynamic value. An interface is only `nil` when **both** are nil. This is a frequent source of bugs: a non-nil pointer stored in an interface produces a non-nil interface, even if the pointer itself is nil.

```go
package main

import "fmt"

type MyError struct{ msg string }
func (e *MyError) Error() string { return e.msg }

func badError() error {
    var e *MyError = nil // typed nil pointer
    return e             // wraps (*MyError)(nil) in an error interface
}

func goodError() error {
    return nil // truly nil interface
}

func main() {
    err1 := badError()
    err2 := goodError()

    fmt.Println(err1 == nil) // Output: false — surprise!
    fmt.Println(err2 == nil) // Output: true

    // Fix: return error directly, not a concrete nil pointer.
}
```

## Type Conversions

Go **never performs implicit conversions**. Every conversion between distinct types must be written explicitly as `T(x)`. This avoids silent precision loss and makes data flow obvious.

### Numeric Conversions

Conversions between numeric types may truncate or lose precision; the programmer is responsible for ensuring the conversion is valid.

```go
package main

import "fmt"

func main() {
    i := 42
    f := float64(i)  // int → float64
    u := uint(f)     // float64 → uint (truncates fractional part)

    big := int64(1_000_000)
    small := int8(big) // narrowing: wraps around

    fmt.Printf("int=%d float64=%f uint=%d\n", i, f, u)
    fmt.Printf("int64=%d int8=%d\n", big, small)
    // Output:
    // int=42 float64=42.000000 uint=42
    // int64=1000000 int8=64
}
```

### Converting Between Named Types

Two named types that share the same underlying type can be converted explicitly. Arithmetic is performed using the underlying type, never the named type directly.

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

An **untyped constant** has a default type applied when it is assigned to a variable without an explicit type annotation. For example, `42` defaults to `int`, `3.14` to `float64`, `'A'` to `rune`, and `"hi"` to `string`. Untyped constants can be assigned to any compatible type without explicit conversion.

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

## Generics & Type Parameters

Go 1.18 introduced **type parameters**, enabling functions and types to operate on a range of types while retaining full static type safety. Generics eliminate the need for code duplication or `any`-based workarounds.

### Syntax

```
func FuncName[T Constraint](param T) T { ... }
type TypeName[T Constraint] struct { ... }
```

### Built-in Constraints

| Constraint | Meaning |
|---|---|
| `any` | Any type (`interface{}`) |
| `comparable` | Types that support `==` and `!=` |

### The Tilde Operator `~`

In a constraint's type set, `~T` means "any type whose underlying type is T". This includes type definitions built on top of `T`.

```go
type MyInt int
// ~int matches both int and MyInt; plain int matches only int.
```

### Custom Constraints

A constraint is an interface. It may include a **type element**—a union of types using `|`—to restrict what type arguments are permitted.

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

func Reduce[T, U any](s []T, init U, f func(U, T) U) U {
    acc := init
    for _, v := range s {
        acc = f(acc, v)
    }
    return acc
}

func main() {
    nums := []int{1, 2, 3, 4, 5}

    doubled := Map(nums, func(n int) int { return n * 2 })
    fmt.Println(doubled) // Output: [2 4 6 8 10]

    evens := Filter(nums, func(n int) bool { return n%2 == 0 })
    fmt.Println(evens) // Output: [2 4]

    sum := Reduce(nums, 0, func(acc, n int) int { return acc + n })
    fmt.Println(sum) // Output: 15

    strs := Map(nums, func(n int) string { return fmt.Sprintf("%d!", n) })
    fmt.Println(strs) // Output: [1! 2! 3! 4! 5!]
}
```

### Generic Types

Type parameters apply to type definitions too, not just functions. This is how standard-library types like `sync.Map` would be expressed with generics today.

```go
package main

import "fmt"

type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(v T)     { s.items = append(s.items, v) }
func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    last := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return last, true
}
func (s *Stack[T]) Len() int { return len(s.items) }

func main() {
    var s Stack[int]
    s.Push(1)
    s.Push(2)
    s.Push(3)

    for s.Len() > 0 {
        v, _ := s.Pop()
        fmt.Println(v)
    }
    // Output:
    // 3
    // 2
    // 1
}
```

### Type Inference

Go infers type arguments from the function's actual arguments when the type parameters are used in the parameter list. You rarely need to write them explicitly.

```go
// Inferred:  Map(nums, f)        — compiler deduces T=int, U=string
// Explicit:  Map[int, string](nums, f)
```

### `cmp.Ordered` (Go 1.21)

Go 1.21 added the `cmp` package with the `cmp.Ordered` constraint, covering all ordered built-in types. The `cmp.Compare` function and `slices.Min` / `slices.Max` use it.

```go
package main

import (
    "cmp"
    "fmt"
    "slices"
)

func Min[T cmp.Ordered](a, b T) T {
    if cmp.Compare(a, b) < 0 {
        return a
    }
    return b
}

func main() {
    fmt.Println(Min(3, 7))              // Output: 3
    fmt.Println(Min(3.14, 2.71))        // Output: 2.71
    fmt.Println(Min("apple", "banana")) // Output: apple

    nums := []int{5, 2, 8, 1, 9}
    fmt.Println(slices.Min(nums)) // Output: 1
    fmt.Println(slices.Max(nums)) // Output: 9
}
```

## Reflection

The `reflect` package lets programs inspect and manipulate types and values at runtime. It is powerful but costly—reflection bypasses compile-time checks and is significantly slower than direct code. Prefer generics or interfaces whenever the type can be known at compile time.

### `reflect.Type` and `reflect.Kind`

`reflect.TypeOf(x)` returns the `reflect.Type` of `x`—its full named type (e.g., `main.Celsius`). `reflect.Kind` is a coarser category: the underlying kind such as `reflect.Float64`, `reflect.Struct`, or `reflect.Slice`.

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

Reflection can iterate over struct fields and their tags—commonly used in serialization libraries:

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
    Age   int    `json:"age,omitempty"`
}

func main() {
    u := User{Name: "Alice", Email: "alice@example.com", Age: 30}
    t := reflect.TypeOf(u)
    v := reflect.ValueOf(u)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("%-8s = %-25v tag=%s\n", field.Name, value, field.Tag.Get("json"))
    }
    // Output:
    // Name     = Alice                     tag=name
    // Email    = alice@example.com         tag=email
    // Age      = 30                        tag=age,omitempty
}
```

### When to Use Reflection

Use reflection sparingly—only when the type is genuinely unknown at compile time, such as when writing serialization libraries, test helpers, dependency injection containers, or ORMs. For all other cases, use generics or interface-based polymorphism: they are faster, safer, and more readable.
