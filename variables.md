# Variables and Constants

Variables and constants are fundamental building blocks in Go programming,  
providing mechanisms to store, manipulate, and reference data throughout your  
program's execution. Understanding how to properly declare, initialize, and  
use variables and constants is essential for writing effective Go code.

In Go, variables are memory locations that hold values which can change during  
program execution. They must be declared with a specific type, either  
explicitly or through type inference. Go's type system ensures memory safety  
and helps prevent common programming errors through compile-time type checking.  
Variables  
can be declared at package level, function level, or block level, each with  
different scoping rules and lifetime characteristics.

Constants, on the other hand, are immutable values that are determined at  
compile time and cannot be changed during program execution. Go constants are  
more flexible than constants in many other languages, as they can be untyped,  
allowing them to be used in various contexts without explicit type conversion.  
Constants can be simple literals, or complex expressions involving other  
constants and compile-time evaluable functions.

Go provides several ways to declare variables, including explicit type  
declaration, type inference with the `var` keyword, and shorthand declaration  
using the `:=` operator. Each method has its appropriate use cases and  
understanding when to use which approach is crucial for writing idiomatic Go  
code. The language also provides the `const` keyword for declaring constants,  
with special support for the `iota` identifier to create enumerated constants.

The scope and lifetime of variables in Go follow clear rules that determine  
where variables can be accessed and when they are garbage collected. Package-  
level variables have program-wide lifetime, while local variables exist only  
within their declaring block. Go also supports variable shadowing, where  
inner scope variables can hide outer scope variables with the same name.

Understanding Go's zero values concept is fundamental, as every type has a  
meaningful zero value that variables automatically receive when declared  
without explicit initialization. This eliminates entire classes of bugs  
related to uninitialized variables and makes Go programs more predictable  
and reliable.


## Basic variable declaration

Variables in Go can be declared using the `var` keyword with explicit type  
specification. This is the most explicit form of variable declaration.  

```go
package main

import "fmt"

func main() {
    var name string
    var age int
    var salary float64
    var isActive bool

    name = "Alice Johnson"
    age = 28
    salary = 75000.50
    isActive = true

    fmt.Printf("Name: %s\n", name)
    fmt.Printf("Age: %d\n", age)
    fmt.Printf("Salary: %.2f\n", salary)
    fmt.Printf("Active: %t\n", isActive)
}
```

This example shows basic variable declarations with explicit types. Variables  
are first declared and then assigned values separately. Each variable must  
have a type specified, and Go's type system ensures type safety at compile  
time.  


## Variable initialization with declaration

Variables can be declared and initialized in a single statement, making code  
more concise and readable when initial values are known.  

```go
package main

import "fmt"

func main() {
    var name string = "Bob Wilson"
    var age int = 32
    var salary float64 = 82000.75
    var isManager bool = true

    fmt.Printf("Employee: %s, Age: %d\n", name, age)
    fmt.Printf("Salary: $%.2f, Manager: %t\n", salary, isManager)

    // Multiple variables of same type
    var x, y, z int = 10, 20, 30
    fmt.Printf("Coordinates: (%d, %d, %d)\n", x, y, z)

    // Different types in one declaration
    var (
        city       string = "New York"
        population int    = 8419000
        area       float64 = 783.8
    )
    fmt.Printf("%s: %d people, %.1f km²\n", city, population, area)
}
```

Combined declaration and initialization is more efficient and clearer than  
separate steps. The parenthesized var declaration allows grouping related  
variables together, improving code organization and readability.  


## Type inference with var

Go can automatically infer the type of a variable from its initial value,  
eliminating the need for explicit type specification in many cases.  

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    var name = "Charlie Brown"
    var age = 25
    var height = 5.9
    var isStudent = false

    fmt.Printf("Name: %s (type: %s)\n", name, reflect.TypeOf(name))
    fmt.Printf("Age: %d (type: %s)\n", age, reflect.TypeOf(age))
    fmt.Printf("Height: %.1f (type: %s)\n", height, reflect.TypeOf(height))
    fmt.Printf("Student: %t (type: %s)\n", isStudent, reflect.TypeOf(isStudent))

    // Mixed initialization with inference
    var (
        language = "Go"
        version  = 1.22
        stable   = true
    )

    fmt.Printf("\nLanguage: %s %.2f, Stable: %t\n", language, version, stable)
}
```

Type inference makes code cleaner while maintaining type safety. Go infers  
the most appropriate type based on the literal value: strings for text,  
int for integers, float64 for decimals, and bool for true/false values.  


## Shorthand variable declaration

The `:=` operator provides a concise way to declare and initialize variables  
within functions, combining declaration, type inference, and assignment.  

```go
package main

import "fmt"

func main() {
    name := "Diana Prince"
    age := 29
    salary := 95000.00
    isActive := true

    fmt.Printf("Employee: %s\n", name)
    fmt.Printf("Details: %d years old, $%.2f salary\n", age, salary)
    fmt.Printf("Status: Active = %t\n", isActive)

    // Multiple assignment with shorthand
    firstName, lastName := "John", "Smith"
    x, y := 42, 3.14

    fmt.Printf("\nName: %s %s\n", firstName, lastName)
    fmt.Printf("Values: integer=%d, float=%.2f\n", x, y)

    // Mixed with existing variables
    count := 100
    count, message := 200, "Updated count"
    fmt.Printf("Count: %d, Message: %s\n", count, message)
}
```

The `:=` operator is only available inside functions and requires at least  
one new variable on the left side. It's the most commonly used variable  
declaration method in Go due to its brevity and clarity.  


## Zero values

Every type in Go has a zero value - the default value assigned to variables  
when they are declared but not explicitly initialized.  

```go
package main

import "fmt"

func main() {
    var text string
    var number int
    var decimal float64
    var flag bool
    var data []byte
    var mapping map[string]int
    var ptr *int

    fmt.Printf("string zero value: '%s' (empty)\n", text)
    fmt.Printf("int zero value: %d\n", number)
    fmt.Printf("float64 zero value: %f\n", decimal)
    fmt.Printf("bool zero value: %t\n", flag)
    fmt.Printf("slice zero value: %v (nil)\n", data)
    fmt.Printf("map zero value: %v (nil)\n", mapping)
    fmt.Printf("pointer zero value: %v (nil)\n", ptr)

    // Zero values are ready to use
    text += "Hello"
    number += 42
    decimal += 3.14
    flag = !flag

    fmt.Printf("\nAfter modification:\n")
    fmt.Printf("string: '%s'\n", text)
    fmt.Printf("int: %d\n", number)
    fmt.Printf("float64: %.2f\n", decimal)
    fmt.Printf("bool: %t\n", flag)
}
```

Zero values make Go programs predictable and eliminate uninitialized variable  
bugs. Each type has a meaningful zero value that allows immediate use without  
explicit initialization, promoting safer programming practices.  


## Multiple variable declarations

Go provides several ways to declare multiple variables efficiently in a  
single statement or block.  

```go
package main

import "fmt"

func main() {
    // Multiple variables of same type
    var a, b, c int
    var x, y, z = 1, 2, 3

    a, b, c = 10, 20, 30
    fmt.Printf("Same type: a=%d, b=%d, c=%d\n", a, b, c)
    fmt.Printf("With init: x=%d, y=%d, z=%d\n", x, y, z)

    // Mixed types with shorthand
    name, age, salary := "Emma Watson", 34, 120000.0
    fmt.Printf("Mixed types: %s, %d, %.2f\n", name, age, salary)

    // Variable block declaration
    var (
        server   string  = "localhost"
        port     int     = 8080
        timeout  float64 = 30.5
        ssl      bool    = true
        maxConn  int     = 100
    )

    fmt.Printf("\nServer config:\n")
    fmt.Printf("  Address: %s:%d\n", server, port)
    fmt.Printf("  Timeout: %.1fs\n", timeout)
    fmt.Printf("  SSL: %t, Max connections: %d\n", ssl, maxConn)
}
```

Multiple declarations improve code organization and reduce repetition. The  
block form with parentheses is particularly useful for grouping related  
variables and improving readability.  


## Basic constants

Constants are immutable values that are evaluated at compile time and cannot  
be changed during program execution.  

```go
package main

import "fmt"

const (
    AppName    = "MyApplication"
    Version    = "1.2.3"
    MaxRetries = 5
    Timeout    = 30.0
    DebugMode  = false
)

const Pi = 3.14159265359

func main() {
    fmt.Printf("Application: %s v%s\n", AppName, Version)
    fmt.Printf("Configuration: %d retries, %.1fs timeout\n", 
               MaxRetries, Timeout)
    fmt.Printf("Debug mode: %t\n", DebugMode)
    fmt.Printf("Pi constant: %.5f\n", Pi)

    // Constants can be used in expressions
    circumference := 2 * Pi * 10.0
    fmt.Printf("Circle circumference (r=10): %.2f\n", circumference)

    // Compile-time evaluation
    const hoursPerDay = 24
    const minutesPerHour = 60
    const minutesPerDay = hoursPerDay * minutesPerHour
    
    fmt.Printf("Minutes in a day: %d\n", minutesPerDay)
}
```

Constants provide compile-time guarantees and help eliminate magic numbers  
from code. They can be used in constant expressions, and the Go compiler  
evaluates constant expressions at compile time for efficiency.  


## Typed constants

Constants can have explicit types, which restricts their usage to compatible  
types and provides additional type safety.  

```go
package main

import "fmt"

const (
    // Typed constants
    MaxSize     int     = 1000
    Rate        float32 = 0.05
    ServiceName string  = "UserService"
    IsEnabled   bool    = true
)

const (
    // Type specified for group
    Red   int = 255
    Green int = 128
    Blue  int = 64
)

func main() {
    var size int = MaxSize
    var interestRate float32 = Rate
    var service string = ServiceName
    var enabled bool = IsEnabled

    fmt.Printf("Max size: %d (int)\n", size)
    fmt.Printf("Rate: %.3f (float32)\n", interestRate)
    fmt.Printf("Service: %s (string)\n", service)
    fmt.Printf("Enabled: %t (bool)\n", enabled)

    // RGB color values
    fmt.Printf("\nColor values:\n")
    fmt.Printf("Red: %d, Green: %d, Blue: %d\n", Red, Green, Blue)

    // Type enforcement prevents mixing
    var colorValue int = Red  // OK: both are int
    fmt.Printf("Selected color value: %d\n", colorValue)
}
```

Typed constants provide stronger type checking and prevent accidental type  
mixing. When you need precise control over constant types or want to ensure  
constants can only be used with specific types, typed constants are preferred.  


## Untyped constants

Untyped constants are more flexible and can be used with any compatible type,  
making them ideal for general-purpose values and mathematical constants.  

```go
package main

import "fmt"

const (
    // Untyped constants
    MaxCount = 100
    Pi       = 3.14159265359
    Message  = "hello there"
    Active   = true
)

func main() {
    // Untyped constants adapt to context
    var intValue int = MaxCount
    var int32Value int32 = MaxCount
    var int64Value int64 = MaxCount

    fmt.Printf("int: %d, int32: %d, int64: %d\n", 
               intValue, int32Value, int64Value)

    // Floating point flexibility
    var float32Pi float32 = Pi
    var float64Pi float64 = Pi

    fmt.Printf("float32 Pi: %.5f\n", float32Pi)
    fmt.Printf("float64 Pi: %.10f\n", float64Pi)

    // String and boolean constants
    var greeting string = Message
    var status bool = Active

    fmt.Printf("Greeting: %s, Status: %t\n", greeting, status)

    // Use in expressions
    area := Pi * 5 * 5  // Pi adapts to expression context
    fmt.Printf("Circle area (r=5): %.2f\n", area)
}
```

Untyped constants provide maximum flexibility by adapting to their usage  
context. They're particularly useful for mathematical constants, configuration  
values, and any constant that might be used with multiple numeric types.  


## Constants with iota

The `iota` identifier generates successive integer constants automatically,  
starting from 0 and incrementing by 1 for each item in a constant block.  

```go
package main

import "fmt"

const (
    // Basic iota enumeration
    Sunday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
)

const (
    // Custom iota expressions
    KB = 1 << (10 * iota)  // 1 << 0 = 1
    MB                     // 1 << 10 = 1024
    GB                     // 1 << 20 = 1048576
    TB                     // 1 << 30 = 1073741824
)

const (
    // Status enumeration
    StatusPending = iota + 1  // Start from 1
    StatusProcessing
    StatusComplete
    StatusError
)

func main() {
    fmt.Printf("Days of week:\n")
    fmt.Printf("Sunday=%d, Monday=%d, Tuesday=%d\n", Sunday, Monday, Tuesday)
    fmt.Printf("Wednesday=%d, Thursday=%d, Friday=%d, Saturday=%d\n", 
               Wednesday, Thursday, Friday, Saturday)

    fmt.Printf("\nStorage sizes:\n")
    fmt.Printf("KB=%d, MB=%d, GB=%d, TB=%d\n", KB, MB, GB, TB)

    fmt.Printf("\nStatus codes:\n")
    fmt.Printf("Pending=%d, Processing=%d, Complete=%d, Error=%d\n",
               StatusPending, StatusProcessing, StatusComplete, StatusError)
}
```

The `iota` identifier simplifies creating enumerated constants and reduces  
maintenance overhead. It resets to 0 in each const block and can be used  
in expressions to create more complex constant sequences.  


## Variable scope

Variables in Go have different scopes depending on where they are declared:  
package level, function level, or block level.  

```go
package main

import "fmt"

// Package-level variables (global scope)
var globalCounter int = 100
var packageName string = "ScopeDemo"

func main() {
    // Function-level variables
    var functionVar string = "function scope"
    localVar := 42

    fmt.Printf("Package variables: %s, counter=%d\n", 
               packageName, globalCounter)
    fmt.Printf("Function variables: %s, local=%d\n", functionVar, localVar)

    // Block scope
    if true {
        blockVar := "block scope"
        fmt.Printf("Block variable: %s\n", blockVar)
        
        // Can access outer scopes
        fmt.Printf("Accessing function var from block: %s\n", functionVar)
        fmt.Printf("Accessing global from block: %d\n", globalCounter)
    }
    
    // blockVar is not accessible here
    // fmt.Println(blockVar)  // Would cause compile error

    // For loop scope
    for i := 0; i < 3; i++ {
        loopVar := i * 10
        fmt.Printf("Loop iteration %d, loopVar=%d\n", i, loopVar)
    }
    
    // i and loopVar are not accessible here
    
    demonstrateScope()
}

func demonstrateScope() {
    fmt.Printf("Function can access package variables: %s\n", packageName)
    // Cannot access main function's local variables
}
```

Understanding scope is crucial for variable accessibility and lifetime  
management. Variables are only accessible within their declared scope and  
inner scopes, promoting encapsulation and preventing naming conflicts.  


## Variable shadowing

Inner scope variables can shadow (hide) outer scope variables with the same  
name, creating a new variable that masks the outer one.  

```go
package main

import "fmt"

var name string = "Global Alice"

func main() {
    fmt.Printf("Global name: %s\n", name)

    name := "Function Bob"  // Shadows global variable
    fmt.Printf("Function name: %s\n", name)

    {
        name := "Block Charlie"  // Shadows function variable
        fmt.Printf("Block name: %s\n", name)
        
        {
            name := "Inner Diana"  // Shadows block variable
            fmt.Printf("Inner block name: %s\n", name)
        }
        
        fmt.Printf("Back to block name: %s\n", name)
    }

    fmt.Printf("Back to function name: %s\n", name)

    // Demonstrate with different types
    value := 42
    fmt.Printf("Integer value: %d\n", value)

    if true {
        value := "string value"  // Different type, shadows outer
        fmt.Printf("String value: %s\n", value)
    }

    fmt.Printf("Back to integer value: %d\n", value)
}
```

Variable shadowing can make code confusing if overused, but it's sometimes  
useful for temporary variables or when you want to reuse a variable name  
with a different type in a limited scope.  


## Pointer variables

Pointer variables store memory addresses of other variables, enabling  
indirect access and efficient data passing.  

```go
package main

import "fmt"

func main() {
    // Basic pointer declaration and usage
    var number int = 42
    var ptr *int = &number

    fmt.Printf("Value: %d\n", number)
    fmt.Printf("Address: %p\n", ptr)
    fmt.Printf("Value via pointer: %d\n", *ptr)

    // Modify through pointer
    *ptr = 100
    fmt.Printf("After modification: %d\n", number)

    // Pointer to different types
    name := "Elena Rodriguez"
    namePtr := &name
    
    salary := 85000.50
    salaryPtr := &salary

    fmt.Printf("\nEmployee data:\n")
    fmt.Printf("Name: %s (via pointer: %s)\n", name, *namePtr)
    fmt.Printf("Salary: %.2f (via pointer: %.2f)\n", salary, *salaryPtr)

    // Nil pointer
    var nilPtr *int
    fmt.Printf("Nil pointer: %v\n", nilPtr)

    // Pointer assignment
    value1, value2 := 10, 20
    ptr1, ptr2 := &value1, &value2

    fmt.Printf("Before swap: *ptr1=%d, *ptr2=%d\n", *ptr1, *ptr2)
    ptr1, ptr2 = ptr2, ptr1  // Swap pointers
    fmt.Printf("After pointer swap: *ptr1=%d, *ptr2=%d\n", *ptr1, *ptr2)
}
```

Pointers are essential for efficient memory usage and enabling functions to  
modify variables from outer scopes. They're particularly important for  
working with large data structures and implementing data structures like  
linked lists.  


## Variable assignment and reassignment

Variables can be reassigned new values of the same type throughout their  
lifetime, while constants cannot be modified after declaration.  

```go
package main

import "fmt"

func main() {
    // Basic assignment and reassignment
    var counter int = 0
    fmt.Printf("Initial counter: %d\n", counter)

    counter = 10
    fmt.Printf("After assignment: %d\n", counter)

    counter = counter + 5
    fmt.Printf("After increment: %d\n", counter)

    counter += 20  // Compound assignment
    fmt.Printf("After compound assignment: %d\n", counter)

    // Multiple assignment
    var a, b, c int = 1, 2, 3
    fmt.Printf("Initial: a=%d, b=%d, c=%d\n", a, b, c)

    a, b, c = c, a, b  // Rotate values
    fmt.Printf("After rotation: a=%d, b=%d, c=%d\n", a, b, c)

    // Assignment with different expressions
    name := "Frank"
    fmt.Printf("Name: %s\n", name)

    name = name + " Miller"
    fmt.Printf("Full name: %s\n", name)

    // Compound assignments
    score := 100.0
    score *= 1.1   // Multiply by 1.1
    score -= 5.0   // Subtract 5
    score /= 2.0   // Divide by 2
    
    fmt.Printf("Final score: %.2f\n", score)

    // Constants cannot be reassigned
    const maxValue = 1000
    fmt.Printf("Constant value: %d\n", maxValue)
    // maxValue = 2000  // Would cause compile error
}
```

Variable reassignment is fundamental to program logic and state management.  
Go provides compound assignment operators for common operations, making code  
more concise and readable.  


## Blank identifier

The blank identifier `_` is used to ignore values that you don't need,  
particularly useful with functions that return multiple values.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

func getPersonInfo() (string, int, float64) {
    return "Grace Hopper", 85, 95000.0
}

func parseNumber(s string) (int, error) {
    return strconv.Atoi(s)
}

func main() {
    // Ignore specific return values
    name, _, salary := getPersonInfo()  // Ignore age
    fmt.Printf("Employee: %s, Salary: %.2f\n", name, salary)

    _, age, _ := getPersonInfo()  // Only get age
    fmt.Printf("Age: %d\n", age)

    // Ignore error return (not recommended in real code)
    number, _ := parseNumber("123")
    fmt.Printf("Parsed number: %d\n", number)

    // Ignore iteration index
    fruits := []string{"apple", "banana", "orange"}
    for _, fruit := range fruits {
        fmt.Printf("Fruit: %s\n", fruit)
    }

    // Ignore iteration value
    for index, _ := range fruits {
        fmt.Printf("Index: %d\n", index)
    }

    // Multiple blank identifiers
    text := "Go,is,awesome"
    parts := strings.Split(text, ",")
    first, _, third := parts[0], parts[1], parts[2]
    fmt.Printf("First: %s, Third: %s\n", first, third)

    // Blank identifier in variable declarations
    var _ int = 42  // Valid but unusual
    _ = "ignored string"
}
```

The blank identifier helps write cleaner code by explicitly ignoring unused  
values. It prevents compiler errors about unused variables while making your  
intent clear to other developers reading the code.  


## Variable naming conventions

Go has established conventions for variable naming that promote code  
readability and consistency across projects.  

```go
package main

import "fmt"

// Package-level variables use camelCase for unexported
var serverPort int = 8080
var debugMode bool = true

// Exported variables start with capital letter
var MaxConnections int = 1000
var DefaultTimeout float64 = 30.0

func main() {
    // Local variables use camelCase
    userName := "john_doe"
    userAge := 25
    isActive := true

    // Descriptive names for clarity
    totalOrderValue := 1250.75
    customerEmailAddress := "john@example.com"
    numberOfRetries := 3

    fmt.Printf("User: %s (%d years old)\n", userName, userAge)
    fmt.Printf("Status: %t\n", isActive)
    fmt.Printf("Order total: $%.2f\n", totalOrderValue)
    fmt.Printf("Email: %s\n", customerEmailAddress)
    fmt.Printf("Retries: %d\n", numberOfRetries)

    // Short names for short-lived variables
    for i := 0; i < 3; i++ {
        for j := 0; j < 2; j++ {
            fmt.Printf("(%d,%d) ", i, j)
        }
    }
    fmt.Println()

    // Common abbreviations
    var id int = 12345
    var url string = "https://example.com"
    var db interface{}  // Database connection
    var ctx interface{} // Context
    
    fmt.Printf("ID: %d, URL: %s\n", id, url)

    // Avoid single-letter names except for short loops
    temperature := 23.5  // Not just 't'
    pressure := 1013.25  // Not just 'p'
    
    fmt.Printf("Temp: %.1f°C, Pressure: %.1f hPa\n", temperature, pressure)
}
```

Good variable names make code self-documenting and easier to maintain.  
Use descriptive names that clearly indicate the variable's purpose, and  
follow Go's conventions for exported/unexported visibility.  


## Package-level variables

Package-level variables are declared outside functions and are accessible  
throughout the package, with initialization occurring before main().  

```go
package main

import (
    "fmt"
    "time"
)

// Simple package variables
var appName string = "DataProcessor"
var version string = "2.1.0"
var startTime time.Time = time.Now()

// Variables with initialization function calls
var homeDir string = getHomeDirectory()
var configFile string = homeDir + "/config.json"

// Variables without initialization (zero values)
var totalProcessed int
var errorCount int

// Variable block for related items
var (
    maxWorkers   int     = 10
    bufferSize   int     = 1000
    timeout      float64 = 30.0
    retryEnabled bool    = true
)

func getHomeDirectory() string {
    return "/home/user"  // Simplified for example
}

func init() {
    // Package initialization function
    fmt.Printf("Initializing %s v%s\n", appName, version)
    fmt.Printf("Start time: %s\n", startTime.Format("15:04:05"))
    
    totalProcessed = 0
    errorCount = 0
}

func main() {
    fmt.Printf("Application: %s v%s\n", appName, version)
    fmt.Printf("Configuration:\n")
    fmt.Printf("  Config file: %s\n", configFile)
    fmt.Printf("  Max workers: %d\n", maxWorkers)
    fmt.Printf("  Buffer size: %d\n", bufferSize)
    fmt.Printf("  Timeout: %.1fs\n", timeout)
    fmt.Printf("  Retry enabled: %t\n", retryEnabled)
    
    // Modify package variables
    processData()
    
    fmt.Printf("\nResults:\n")
    fmt.Printf("  Total processed: %d\n", totalProcessed)
    fmt.Printf("  Errors: %d\n", errorCount)
}

func processData() {
    // Simulate processing
    totalProcessed = 150
    errorCount = 2
}
```

Package-level variables provide global state and configuration for the  
package. They're initialized in declaration order and can use previously  
declared package variables in their initialization expressions.  


## Variable initialization patterns

Go provides various patterns for initializing variables efficiently and  
safely, depending on the use case and data requirements.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Direct initialization
    name := "Isabella Martinez"
    age := 31

    // Conditional initialization
    var status string
    if age >= 18 {
        status = "adult"
    } else {
        status = "minor"
    }

    // Initialization with function calls
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    
    // Complex initialization
    var config = struct {
        host    string
        port    int
        enabled bool
    }{
        host:    "localhost",
        port:    8080,
        enabled: true,
    }

    fmt.Printf("Person: %s (%d) - %s\n", name, age, status)
    fmt.Printf("Timestamp: %s\n", timestamp)
    fmt.Printf("Config: %+v\n", config)

    // Slice initialization patterns
    numbers := []int{1, 2, 3, 4, 5}
    scores := make([]float64, 3, 5)  // length 3, capacity 5
    names := []string{"Alice", "Bob", "Charlie"}

    fmt.Printf("Numbers: %v\n", numbers)
    fmt.Printf("Scores: %v (len=%d, cap=%d)\n", 
               scores, len(scores), cap(scores))
    fmt.Printf("Names: %v\n", names)

    // Map initialization patterns
    ages := map[string]int{
        "Alice":   25,
        "Bob":     30,
        "Charlie": 28,
    }
    
    grades := make(map[string]float64)
    grades["Math"] = 95.5
    grades["Science"] = 88.0

    fmt.Printf("Ages: %v\n", ages)
    fmt.Printf("Grades: %v\n", grades)
}
```

Proper initialization patterns prevent runtime errors and make code more  
reliable. Choose the initialization method that best fits your data structure  
and usage requirements.  


## Type conversion with variables

Variables often need to be converted between different types, requiring  
explicit conversion in Go's type-safe system.  

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // Numeric type conversions
    var intValue int = 42
    var floatValue float64 = float64(intValue)
    var int32Value int32 = int32(intValue)
    
    fmt.Printf("int: %d, float64: %.1f, int32: %d\n", 
               intValue, floatValue, int32Value)

    // Precision may be lost
    var largeFloat float64 = 123.456
    var convertedInt int = int(largeFloat)
    
    fmt.Printf("float64: %.3f, converted to int: %d\n", 
               largeFloat, convertedInt)

    // String conversions
    var number int = 456
    var text string = strconv.Itoa(number)
    
    fmt.Printf("int: %d, as string: %s\n", number, text)

    // String to number conversion with error handling
    var numberString string = "789"
    if convertedNumber, err := strconv.Atoi(numberString); err == nil {
        fmt.Printf("string: %s, as int: %d\n", numberString, convertedNumber)
    } else {
        fmt.Printf("Conversion error: %v\n", err)
    }

    // Byte and rune conversions
    var letter byte = 65
    var runeChar rune = 'A'
    
    fmt.Printf("byte %d as string: %s\n", letter, string(letter))
    fmt.Printf("rune %c as string: %s\n", runeChar, string(runeChar))

    // Boolean to string (custom conversion needed)
    var flag bool = true
    var boolString string = strconv.FormatBool(flag)
    
    fmt.Printf("bool: %t, as string: %s\n", flag, boolString)
}
```

Explicit type conversion prevents accidental data loss and makes type  
transformations clear and intentional. Always handle potential errors  
when converting from strings to numeric types.  


## Variable lifetime and garbage collection

Understanding variable lifetime helps write efficient Go programs and  
avoid memory leaks.  

```go
package main

import (
    "fmt"
    "runtime"
)

// Package variables live for entire program duration
var globalData []int = make([]int, 1000)

func main() {
    fmt.Printf("Initial goroutines: %d\n", runtime.NumGoroutine())
    printMemStats("Initial")

    createShortLivedVariables()
    printMemStats("After short-lived variables")

    createLongLivedVariables()
    printMemStats("After long-lived variables")

    // Force garbage collection
    runtime.GC()
    printMemStats("After GC")

    demonstrateEscape()
    printMemStats("After escape demonstration")
}

func createShortLivedVariables() {
    // These variables exist only within this function
    data := make([]int, 10000)
    temp := "temporary string data"
    counter := 0

    for i := range data {
        data[i] = i * 2
        counter++
    }

    fmt.Printf("Processed %d items, temp: %s\n", counter, temp[:9])
    // data, temp, and counter are eligible for GC when function returns
}

func createLongLivedVariables() {
    // These variables live longer due to references
    persistent := make([]int, 5000)
    for i := range persistent {
        persistent[i] = i
    }
    
    // Store reference in global variable (extends lifetime)
    globalData = persistent[:1000]  // Share slice header
}

func demonstrateEscape() {
    // Variable escapes to heap due to return
    data := makeData()
    fmt.Printf("Escaped data length: %d\n", len(data))
}

func makeData() []int {
    localData := make([]int, 100)  // Escapes to heap
    return localData               // Returned value extends lifetime
}

func printMemStats(label string) {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("%s - Heap: %d KB, GC cycles: %d\n", 
               label, m.Alloc/1024, m.NumGC)
}
```

Variables allocated on the stack are automatically cleaned up when their  
scope ends, while heap-allocated variables require garbage collection.  
Understanding escape analysis helps write more efficient Go programs.  


## Constant expressions and compile-time evaluation

Constants can involve complex expressions that are evaluated at compile  
time, providing both performance benefits and type safety.  

```go
package main

import "fmt"

const (
    // Basic arithmetic expressions
    SecondsPerMinute = 60
    MinutesPerHour   = 60
    HoursPerDay      = 24
    SecondsPerHour   = SecondsPerMinute * MinutesPerHour
    SecondsPerDay    = SecondsPerHour * HoursPerDay
)

const (
    // Bit manipulation expressions
    Flag1 = 1 << 0  // 1
    Flag2 = 1 << 1  // 2
    Flag3 = 1 << 2  // 4
    Flag4 = 1 << 3  // 8
    AllFlags = Flag1 | Flag2 | Flag3 | Flag4  // 15
)

const (
    // String expressions
    Prefix      = "app_"
    Suffix      = "_v1"
    ServiceName = Prefix + "service" + Suffix
    LogFile     = ServiceName + ".log"
)

const (
    // Mathematical expressions
    Pi           = 3.14159265358979323846
    E            = 2.71828182845904523536
    CircleArea   = Pi * 10 * 10        // For radius 10
    SphereVolume = (4.0 / 3.0) * Pi * 5 * 5 * 5  // For radius 5
)

const (
    // Boolean expressions
    DevMode     = true
    TestMode    = false
    DebugMode   = DevMode && !TestMode
    LogEnabled  = DevMode || TestMode
)

func main() {
    fmt.Printf("Time constants:\n")
    fmt.Printf("  Seconds per hour: %d\n", SecondsPerHour)
    fmt.Printf("  Seconds per day: %d\n", SecondsPerDay)

    fmt.Printf("\nBit flags:\n")
    fmt.Printf("  Flag1: %d, Flag2: %d, Flag3: %d, Flag4: %d\n", 
               Flag1, Flag2, Flag3, Flag4)
    fmt.Printf("  All flags: %d (binary: %b)\n", AllFlags, AllFlags)

    fmt.Printf("\nString constants:\n")
    fmt.Printf("  Service: %s\n", ServiceName)
    fmt.Printf("  Log file: %s\n", LogFile)

    fmt.Printf("\nMathematical constants:\n")
    fmt.Printf("  Circle area (r=10): %.2f\n", CircleArea)
    fmt.Printf("  Sphere volume (r=5): %.2f\n", SphereVolume)

    fmt.Printf("\nBoolean constants:\n")
    fmt.Printf("  Debug mode: %t\n", DebugMode)
    fmt.Printf("  Logging enabled: %t\n", LogEnabled)
}
```

Compile-time constant evaluation improves runtime performance by eliminating  
calculations during program execution. Complex constant expressions are  
evaluated once at compile time, not repeatedly at runtime.  


## Enumeration patterns with iota

The `iota` identifier enables sophisticated enumeration patterns for creating  
related constant sequences with custom logic.  

```go
package main

import "fmt"

const (
    // HTTP status codes
    StatusOK = iota + 200  // 200
    StatusCreated          // 201
    StatusAccepted         // 202
    StatusNoContent        // 204
)

const (
    // File permissions (octal)
    ReadOwner   = 1 << (iota + 2)  // 1 << 2 = 4
    WriteOwner                     // 1 << 3 = 8
    ExecOwner                      // 1 << 4 = 16
    ReadGroup                      // 1 << 5 = 32
    WriteGroup                     // 1 << 6 = 64
    ExecGroup                      // 1 << 7 = 128
)

const (
    // Log levels with custom values
    _ = iota  // Skip 0
    LogError
    LogWarn
    LogInfo
    LogDebug
    LogTrace
)

const (
    // Network protocols
    TCP = iota + 1
    UDP
    ICMP
    HTTP
    HTTPS
    FTP
    SSH
)

const (
    // Data sizes with expressions
    Byte = 1 << (iota * 10)  // 1 << 0 = 1
    KB                       // 1 << 10 = 1024
    MB                       // 1 << 20 = 1048576
    GB                       // 1 << 30 = 1073741824
    TB                       // 1 << 40 = 1099511627776
)

func main() {
    fmt.Printf("HTTP Status Codes:\n")
    fmt.Printf("  OK: %d, Created: %d, Accepted: %d, NoContent: %d\n",
               StatusOK, StatusCreated, StatusAccepted, StatusNoContent)

    fmt.Printf("\nFile Permissions:\n")
    fmt.Printf("  ReadOwner: %d, WriteOwner: %d, ExecOwner: %d\n",
               ReadOwner, WriteOwner, ExecOwner)
    fmt.Printf("  ReadGroup: %d, WriteGroup: %d, ExecGroup: %d\n",
               ReadGroup, WriteGroup, ExecGroup)

    fmt.Printf("\nLog Levels:\n")
    fmt.Printf("  Error: %d, Warn: %d, Info: %d, Debug: %d, Trace: %d\n",
               LogError, LogWarn, LogInfo, LogDebug, LogTrace)

    fmt.Printf("\nNetwork Protocols:\n")
    fmt.Printf("  TCP: %d, UDP: %d, ICMP: %d, HTTP: %d, HTTPS: %d\n",
               TCP, UDP, ICMP, HTTP, HTTPS)

    fmt.Printf("\nData Sizes:\n")
    fmt.Printf("  Byte: %d, KB: %d, MB: %d, GB: %d, TB: %d\n",
               Byte, KB, MB, GB, TB)
}
```

Advanced `iota` patterns create meaningful enumerations for various domains  
such as status codes, permissions, protocols, and data sizes. These patterns  
make code more maintainable and self-documenting.  


## Complex constant expressions

Constants can involve sophisticated expressions combining arithmetic,  
bitwise operations, and string manipulations for advanced use cases.  

```go
package main

import "fmt"

const (
    // Version information
    MajorVersion = 2
    MinorVersion = 1
    PatchVersion = 3
    Version = "v" + string(rune(MajorVersion + '0')) + "." + 
              string(rune(MinorVersion + '0')) + "." + 
              string(rune(PatchVersion + '0'))
)

const (
    // Configuration with calculations
    DefaultPort     = 8080
    AdminPortOffset = 100
    AdminPort       = DefaultPort + AdminPortOffset
    
    // Memory settings (in bytes)
    DefaultBufferSizeKB = 64
    DefaultBufferSize   = DefaultBufferSizeKB * 1024
    MaxBufferSize       = DefaultBufferSize * 8
)

const (
    // Feature flags using bit operations
    FeatureAuth     = 1 << 0   // 1
    FeatureLogging  = 1 << 1   // 2
    FeatureCaching  = 1 << 2   // 4
    FeatureMetrics  = 1 << 3   // 8
    FeatureDebug    = 1 << 4   // 16
    
    // Combined feature sets
    BasicFeatures = FeatureAuth | FeatureLogging  // 3
    AdvancedFeatures = BasicFeatures | FeatureCaching | FeatureMetrics  // 15
    AllFeatures = AdvancedFeatures | FeatureDebug  // 31
)

const (
    // Timeout calculations (in milliseconds)
    BaseTimeoutMS    = 1000
    NetworkTimeoutMS = BaseTimeoutMS * 3
    DatabaseTimeout  = BaseTimeoutMS * 5
    TotalTimeoutMS   = NetworkTimeoutMS + DatabaseTimeout
)

const (
    // Mathematical constants with precision
    Pi32 = 3.14159265358979323846
    E32  = 2.71828182845904523536
    
    // Derived mathematical constants
    TwoPi     = 2 * Pi32
    PiOver2   = Pi32 / 2
    PiSquared = Pi32 * Pi32
    SqrtE     = 1.6487212707  // Approximation of sqrt(e)
)

func main() {
    fmt.Printf("Version Information:\n")
    fmt.Printf("  Version: %s\n", Version)
    fmt.Printf("  Components: %d.%d.%d\n", 
               MajorVersion, MinorVersion, PatchVersion)

    fmt.Printf("\nPort Configuration:\n")
    fmt.Printf("  Default port: %d\n", DefaultPort)
    fmt.Printf("  Admin port: %d\n", AdminPort)

    fmt.Printf("\nMemory Configuration:\n")
    fmt.Printf("  Default buffer: %d bytes (%d KB)\n", 
               DefaultBufferSize, DefaultBufferSizeKB)
    fmt.Printf("  Max buffer: %d bytes\n", MaxBufferSize)

    fmt.Printf("\nFeature Flags:\n")
    fmt.Printf("  Basic features: %d (binary: %b)\n", 
               BasicFeatures, BasicFeatures)
    fmt.Printf("  Advanced features: %d (binary: %b)\n", 
               AdvancedFeatures, AdvancedFeatures)
    fmt.Printf("  All features: %d (binary: %b)\n", 
               AllFeatures, AllFeatures)

    fmt.Printf("\nTimeout Configuration:\n")
    fmt.Printf("  Network: %dms, Database: %dms, Total: %dms\n",
               NetworkTimeoutMS, DatabaseTimeout, TotalTimeoutMS)

    fmt.Printf("\nMathematical Constants:\n")
    fmt.Printf("  Pi: %.10f\n", Pi32)
    fmt.Printf("  2π: %.10f\n", TwoPi)
    fmt.Printf("  π/2: %.10f\n", PiOver2)
    fmt.Printf("  π²: %.10f\n", PiSquared)
}
```

Complex constant expressions enable sophisticated compile-time calculations  
for configuration, mathematical constants, and feature flags. These  
expressions are evaluated once during compilation, providing both performance  
and maintainability benefits.  


## Constant type safety and validation

Constants provide compile-time type safety and validation, preventing  
runtime errors and ensuring program correctness through static analysis.  

```go
package main

import "fmt"

// Type-safe constants prevent misuse
type Status int

const (
    StatusInactive Status = iota
    StatusActive
    StatusSuspended
    StatusDeleted
)

type UserRole string

const (
    RoleGuest     UserRole = "guest"
    RoleUser      UserRole = "user" 
    RoleModerator UserRole = "moderator"
    RoleAdmin     UserRole = "admin"
)

// Validation using constants
const (
    MinPasswordLength = 8
    MaxPasswordLength = 128
    MinUsernameLength = 3
    MaxUsernameLength = 50
)

func validateUser(username, password string, role UserRole, status Status) bool {
    // Length validation using constants
    if len(username) < MinUsernameLength || len(username) > MaxUsernameLength {
        fmt.Printf("Invalid username length: %d (must be %d-%d)\n", 
                   len(username), MinUsernameLength, MaxUsernameLength)
        return false
    }
    
    if len(password) < MinPasswordLength || len(password) > MaxPasswordLength {
        fmt.Printf("Invalid password length: %d (must be %d-%d)\n", 
                   len(password), MinPasswordLength, MaxPasswordLength)
        return false
    }
    
    // Type safety prevents invalid values
    fmt.Printf("User validation: %s, role=%s, status=%d\n", username, role, status)
    return true
}

func main() {
    // Valid usage
    fmt.Println("=== Valid User Creation ===")
    validUser := validateUser("john_doe", "securepass123", RoleUser, StatusActive)
    fmt.Printf("Valid user created: %t\n\n", validUser)
    
    // Invalid username (too short)
    fmt.Println("=== Invalid Username ===")
    invalidUser1 := validateUser("jo", "securepass123", RoleGuest, StatusActive)
    fmt.Printf("User created: %t\n\n", invalidUser1)
    
    // Invalid password (too short)
    fmt.Println("=== Invalid Password ===")
    invalidUser2 := validateUser("alice", "short", RoleModerator, StatusActive)
    fmt.Printf("User created: %t\n\n", invalidUser2)
    
    // Demonstrate type safety
    fmt.Println("=== Type Safety Demo ===")
    roles := []UserRole{RoleGuest, RoleUser, RoleModerator, RoleAdmin}
    statuses := []Status{StatusInactive, StatusActive, StatusSuspended, StatusDeleted}
    
    for _, role := range roles {
        for _, status := range statuses {
            fmt.Printf("Role: %s, Status: %d\n", role, status)
        }
    }
    
    // Constants in switch statements
    fmt.Println("\n=== Status Processing ===")
    processStatus(StatusActive)
    processStatus(StatusSuspended)
    processStatus(StatusDeleted)
}

func processStatus(status Status) {
    switch status {
    case StatusInactive:
        fmt.Println("Processing inactive user")
    case StatusActive:
        fmt.Println("Processing active user")
    case StatusSuspended:
        fmt.Println("Processing suspended user")
    case StatusDeleted:
        fmt.Println("Processing deleted user")
    default:
        fmt.Println("Unknown status")
    }
}
```

Typed constants provide strong compile-time guarantees that prevent invalid  
values and improve code reliability. They work seamlessly with Go's type  
system to catch errors early and make code more maintainable.  


## Variable best practices and memory considerations

Following established patterns for variable usage improves code quality,  
performance, and maintainability in Go applications.  

```go
package main

import (
    "fmt"
    "runtime"
    "strings"
)

// Good: Group related package variables
var (
    serverConfig = ServerConfig{
        Host:    "localhost",
        Port:    8080,
        Timeout: 30,
    }
    
    // Good: Clear, descriptive names
    maxConcurrentUsers    int = 1000
    defaultRetryAttempts  int = 3
    enableRequestLogging  bool = true
)

type ServerConfig struct {
    Host    string
    Port    int
    Timeout int
}

func main() {
    demonstrateNaming()
    demonstrateScoping()
    demonstrateMemoryEfficiency()
    demonstrateConstantUsage()
}

func demonstrateNaming() {
    fmt.Println("=== Variable Naming Best Practices ===")
    
    // Good: Descriptive names
    customerEmailAddress := "customer@example.com"
    totalOrderAmount := 1250.50
    isPaymentProcessed := false
    
    // Good: Short names for short-lived variables
    for i := 0; i < 3; i++ {
        for j := 0; j < 2; j++ {
            fmt.Printf("Processing item (%d,%d)\n", i, j)
        }
    }
    
    // Good: Use context-appropriate abbreviations
    userID := 12345
    apiURL := "https://api.example.com"
    httpClient := "client instance"  // Simulated
    
    fmt.Printf("Customer: %s\n", customerEmailAddress)
    fmt.Printf("Order: $%.2f, Processed: %t\n", 
               totalOrderAmount, isPaymentProcessed)
    fmt.Printf("User: %d, API: %s, Client: %s\n", 
               userID, apiURL, httpClient)
}

func demonstrateScoping() {
    fmt.Println("\n=== Variable Scoping Best Practices ===")
    
    // Good: Declare variables close to their usage
    processUserData()
    
    // Good: Limit scope to minimize lifetime
    if needsProcessing := checkProcessingRequired(); needsProcessing {
        fmt.Println("Processing required")
    }
    
    // Good: Use block scope for temporary variables
    {
        tempData := make([]string, 100)
        processTemporaryData(tempData)
        // tempData automatically cleaned up
    }
}

func processUserData() {
    users := []string{"Alice", "Bob", "Charlie"}
    
    for _, user := range users {
        // Good: Variables scoped to loop iteration
        userProfile := fmt.Sprintf("Profile for %s", user)
        isActive := len(user) > 3  // Simple example logic
        
        fmt.Printf("%s (active: %t)\n", userProfile, isActive)
    }
}

func processTemporaryData(data []string) {
    fmt.Printf("Processing %d temporary items\n", len(data))
}

func checkProcessingRequired() bool {
    return true  // Simplified example
}

func demonstrateMemoryEfficiency() {
    fmt.Println("\n=== Memory Efficiency Best Practices ===")
    
    // Good: Pre-allocate slices when size is known
    knownSize := 1000
    efficientSlice := make([]int, 0, knownSize)  // Zero length, known capacity
    
    for i := 0; i < knownSize; i++ {
        efficientSlice = append(efficientSlice, i*2)
    }
    
    fmt.Printf("Efficient slice: len=%d, cap=%d\n", 
               len(efficientSlice), cap(efficientSlice))
    
    // Good: Reuse string builder for concatenation
    var builder strings.Builder
    builder.Grow(200)  // Pre-allocate capacity
    
    for i := 0; i < 10; i++ {
        fmt.Fprintf(&builder, "Item %d ", i)
    }
    
    result := builder.String()
    fmt.Printf("Built string: %s\n", strings.TrimSpace(result))
    
    // Memory usage information
    var memStats runtime.MemStats
    runtime.ReadMemStats(&memStats)
    fmt.Printf("Current heap size: %d KB\n", memStats.Alloc/1024)
}

func demonstrateConstantUsage() {
    fmt.Println("\n=== Constant Usage Best Practices ===")
    
    // Good: Use constants for configuration values
    const (
        maxRetries = 5
        timeoutSeconds = 30
        bufferSize = 4096
    )
    
    // Good: Use constants for magic numbers
    const (
        httpOK = 200
        httpNotFound = 404
        httpInternalError = 500
    )
    
    // Good: Typed constants for type safety
    type Priority int
    const (
        PriorityLow Priority = iota + 1
        PriorityMedium
        PriorityHigh
        PriorityCritical
    )
    
    fmt.Printf("Configuration: retries=%d, timeout=%ds, buffer=%d bytes\n",
               maxRetries, timeoutSeconds, bufferSize)
    fmt.Printf("HTTP codes: OK=%d, NotFound=%d, Error=%d\n",
               httpOK, httpNotFound, httpInternalError)
    fmt.Printf("Priorities: Low=%d, Medium=%d, High=%d, Critical=%d\n",
               PriorityLow, PriorityMedium, PriorityHigh, PriorityCritical)
}
```

Following variable best practices leads to more maintainable, efficient, and  
readable code. Use descriptive names, appropriate scoping, memory-efficient  
patterns, and constants for configuration values to create robust Go  
applications.