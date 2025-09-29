# Functions

Functions are the fundamental building blocks of Go programs, providing a way to  
organize code into reusable, modular components. In Go, functions are first-class  
citizens, meaning they can be assigned to variables, passed as arguments to other  
functions, and returned as values from functions. This flexibility makes functions  
essential for creating clean, maintainable, and efficient code.

Go functions follow a clear and consistent syntax that emphasizes readability and  
simplicity. Every Go program must have at least one function - the `main` function  
that serves as the entry point. Functions in Go can accept zero or more parameters,  
return zero or more values, and can be defined with named or unnamed return values.  
This design promotes explicit error handling and makes function contracts clear.

The Go function system supports various advanced features including variadic  
parameters, closures, anonymous functions, and higher-order functions. Functions  
can be methods associated with types, enabling object-oriented programming  
patterns through receiver functions. Go's approach to functions emphasizes  
composition over inheritance, leading to more flexible and testable code.

Function types in Go are determined by their parameter list and return values,  
allowing for powerful abstraction through function interfaces. This type system  
enables advanced patterns like decorators, middleware, and functional programming  
techniques. Understanding functions deeply is crucial for mastering Go's  
concurrent programming model, as functions work seamlessly with goroutines and  
channels for building concurrent applications.

Go's function design also includes unique features like named return values,  
defer statements for cleanup operations, and panic/recover mechanisms for error  
handling. These features, combined with Go's garbage collector and memory  
management, make functions both powerful and safe to use in production systems.


## Basic function declaration

Functions in Go are declared using the `func` keyword followed by the function  
name, parameter list, return type, and function body.  

```go
package main

import "fmt"

func greet() {
    fmt.Println("Hello there!")
}

func add(a, b int) int {
    return a + b
}

func main() {
    greet()
    result := add(5, 3)
    fmt.Printf("5 + 3 = %d\n", result)
}
```

This example shows two basic function types: `greet()` which takes no parameters  
and returns nothing, and `add()` which takes two integer parameters and returns  
an integer. The function signature clearly defines the contract for each function.  


## Functions with multiple parameters

Functions can accept multiple parameters of different types, making them versatile  
for various computation needs.  

```go
package main

import "fmt"

func introduce(name string, age int, city string) {
    fmt.Printf("Name: %s, Age: %d, City: %s\n", name, age, city)
}

func calculate(a, b int, operation string) float64 {
    switch operation {
    case "add":
        return float64(a + b)
    case "subtract":
        return float64(a - b)
    case "multiply":
        return float64(a * b)
    case "divide":
        if b != 0 {
            return float64(a) / float64(b)
        }
        return 0
    default:
        return 0
    }
}

func main() {
    introduce("Alice", 30, "New York")
    
    fmt.Printf("10 + 5 = %.2f\n", calculate(10, 5, "add"))
    fmt.Printf("10 / 3 = %.2f\n", calculate(10, 3, "divide"))
}
```

Multiple parameters allow functions to be more expressive and handle complex  
operations. Parameter types can be mixed, and when consecutive parameters share  
the same type, the type can be specified once at the end.  


## Multiple return values

Go functions can return multiple values, which is particularly useful for error  
handling and returning computed results along with status information.  

```go
package main

import (
    "fmt"
    "errors"
)

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func getNameAndAge() (string, int) {
    return "Bob", 25
}

func main() {
    result, err := divide(10.0, 2.0)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Result: %.2f\n", result)
    }
    
    name, age := getNameAndAge()
    fmt.Printf("Name: %s, Age: %d\n", name, age)
}
```

Multiple return values eliminate the need for output parameters and make error  
handling explicit. The common pattern is to return the result and an error,  
allowing callers to handle both success and failure cases appropriately.  


## Named return values

Go allows naming return values, which creates variables that can be used within  
the function and are automatically returned when the function ends.  

```go
package main

import "fmt"

func rectangle(length, width float64) (area, perimeter float64) {
    area = length * width
    perimeter = 2 * (length + width)
    return // naked return
}

func parseCoordinates(input string) (x, y int, valid bool) {
    if input == "origin" {
        x, y, valid = 0, 0, true
        return
    }
    if input == "unit" {
        x, y, valid = 1, 1, true
        return
    }
    return 0, 0, false
}

func main() {
    a, p := rectangle(5.0, 3.0)
    fmt.Printf("Rectangle: Area=%.2f, Perimeter=%.2f\n", a, p)
    
    x, y, ok := parseCoordinates("origin")
    if ok {
        fmt.Printf("Coordinates: (%d, %d)\n", x, y)
    }
}
```

Named return values improve readability by documenting what the function returns.  
They're initialized to their zero values and can be modified throughout the  
function. The naked return statement returns the current values of named returns.  


## Variadic functions

Variadic functions accept a variable number of arguments of the same type using  
the ellipsis (...) syntax.  

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

func printMessages(prefix string, messages ...string) {
    for i, msg := range messages {
        fmt.Printf("%s[%d]: %s\n", prefix, i+1, msg)
    }
}

func main() {
    fmt.Printf("Sum of 1,2,3: %d\n", sum(1, 2, 3))
    fmt.Printf("Sum of 1,2,3,4,5: %d\n", sum(1, 2, 3, 4, 5))
    
    numbers := []int{10, 20, 30}
    fmt.Printf("Sum of slice: %d\n", sum(numbers...))
    
    printMessages("LOG", "Starting application", "Loading config", "Ready")
}
```

Variadic parameters are treated as slices within the function. You can pass  
individual arguments or expand a slice using the ellipsis operator when calling  
the function. This pattern is useful for functions with flexible argument counts.  


## Anonymous functions

Anonymous functions are functions without names that can be defined inline and  
assigned to variables or called immediately.  

```go
package main

import "fmt"

func main() {
    // Anonymous function assigned to variable
    square := func(x int) int {
        return x * x
    }
    
    // Immediately invoked function expression
    result := func(a, b int) int {
        return a * b
    }(4, 5)
    
    fmt.Printf("Square of 7: %d\n", square(7))
    fmt.Printf("Immediate result: %d\n", result)
    
    // Anonymous function with closure
    multiplier := 3
    multiply := func(x int) int {
        return x * multiplier
    }
    
    fmt.Printf("5 * %d = %d\n", multiplier, multiply(5))
}
```

Anonymous functions are useful for short-lived operations, event handlers, and  
closures. They can capture variables from their surrounding scope, creating  
powerful patterns for callbacks and functional programming.  


## Function closures

Closures are functions that capture and maintain references to variables from  
their surrounding scope, even after the outer function returns.  

```go
package main

import "fmt"

func createCounter() func() int {
    count := 0
    return func() int {
        count++
        return count
    }
}

func createMultiplier(factor int) func(int) int {
    return func(value int) int {
        return value * factor
    }
}

func main() {
    counter := createCounter()
    fmt.Printf("Count: %d\n", counter()) // 1
    fmt.Printf("Count: %d\n", counter()) // 2
    fmt.Printf("Count: %d\n", counter()) // 3
    
    double := createMultiplier(2)
    triple := createMultiplier(3)
    
    fmt.Printf("Double 5: %d\n", double(5))
    fmt.Printf("Triple 5: %d\n", triple(5))
}
```

Closures maintain state between calls and enable factory patterns. Each closure  
instance maintains its own copy of captured variables, allowing for independent  
state management across multiple function instances.  


## Higher-order functions

Higher-order functions accept other functions as parameters or return functions  
as results, enabling powerful abstraction and code reuse patterns.  

```go
package main

import "fmt"

func apply(numbers []int, fn func(int) int) []int {
    result := make([]int, len(numbers))
    for i, num := range numbers {
        result[i] = fn(num)
    }
    return result
}

func filter(numbers []int, predicate func(int) bool) []int {
    var result []int
    for _, num := range numbers {
        if predicate(num) {
            result = append(result, num)
        }
    }
    return result
}

func main() {
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    
    squares := apply(numbers, func(x int) int {
        return x * x
    })
    
    evens := filter(numbers, func(x int) bool {
        return x%2 == 0
    })
    
    fmt.Printf("Original: %v\n", numbers)
    fmt.Printf("Squares: %v\n", squares)
    fmt.Printf("Evens: %v\n", evens)
}
```

Higher-order functions promote functional programming patterns and code reuse.  
They allow you to write generic algorithms that can work with different  
behaviors supplied through function parameters.  


## Recursive functions

Recursive functions call themselves to solve problems by breaking them down  
into smaller, similar subproblems.  

```go
package main

import "fmt"

func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}

func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

func power(base, exponent int) int {
    if exponent == 0 {
        return 1
    }
    if exponent == 1 {
        return base
    }
    return base * power(base, exponent-1)
}

func main() {
    fmt.Printf("Factorial of 5: %d\n", factorial(5))
    fmt.Printf("Fibonacci of 8: %d\n", fibonacci(8))
    fmt.Printf("2^5 = %d\n", power(2, 5))
}
```

Recursive functions require a base case to prevent infinite recursion. They're  
particularly useful for problems with self-similar structure like mathematical  
sequences, tree traversal, and divide-and-conquer algorithms.  


## Function types and variables

Functions in Go have types based on their signature, and can be assigned to  
variables, enabling dynamic function selection and polymorphism.  

```go
package main

import "fmt"

type Operation func(int, int) int
type Formatter func(string, ...interface{}) string

func add(a, b int) int {
    return a + b
}

func multiply(a, b int) int {
    return a * b
}

func createFormatter(prefix string) Formatter {
    return func(format string, args ...interface{}) string {
        return prefix + fmt.Sprintf(format, args...)
    }
}

func main() {
    var op Operation
    
    op = add
    fmt.Printf("Addition: %d\n", op(5, 3))
    
    op = multiply
    fmt.Printf("Multiplication: %d\n", op(5, 3))
    
    logger := createFormatter("[LOG] ")
    errorLogger := createFormatter("[ERROR] ")
    
    fmt.Println(logger("User %s logged in", "Alice"))
    fmt.Println(errorLogger("Connection failed: %v", "timeout"))
}
```

Function types enable polymorphism and help create flexible APIs. Functions  
with the same signature can be used interchangeably, allowing for strategy  
patterns and plugin architectures.  


## Methods with receivers

Methods are functions with a receiver argument that associates the function  
with a specific type, enabling object-oriented programming patterns.  

```go
package main

import "fmt"

type Rectangle struct {
    width, height float64
}

type Circle struct {
    radius float64
}

func (r Rectangle) area() float64 {
    return r.width * r.height
}

func (r Rectangle) perimeter() float64 {
    return 2 * (r.width + r.height)
}

func (c Circle) area() float64 {
    return 3.14159 * c.radius * c.radius
}

func (r *Rectangle) scale(factor float64) {
    r.width *= factor
    r.height *= factor
}

func main() {
    rect := Rectangle{width: 5, height: 3}
    circle := Circle{radius: 2}
    
    fmt.Printf("Rectangle area: %.2f\n", rect.area())
    fmt.Printf("Rectangle perimeter: %.2f\n", rect.perimeter())
    fmt.Printf("Circle area: %.2f\n", circle.area())
    
    rect.scale(2)
    fmt.Printf("Scaled rectangle area: %.2f\n", rect.area())
}
```

Methods with value receivers operate on copies, while pointer receivers can  
modify the original value. This distinction is crucial for understanding  
method behavior and performance implications.  


## Interface implementations

Interfaces define method signatures that types can implement, enabling  
polymorphism and abstraction without explicit inheritance.  

```go
package main

import "fmt"

type Shape interface {
    area() float64
    perimeter() float64
}

type Drawable interface {
    draw()
}

type Rectangle struct {
    width, height float64
}

type Circle struct {
    radius float64
}

func (r Rectangle) area() float64 {
    return r.width * r.height
}

func (r Rectangle) perimeter() float64 {
    return 2 * (r.width + r.height)
}

func (r Rectangle) draw() {
    fmt.Printf("Drawing rectangle %.1fx%.1f\n", r.width, r.height)
}

func (c Circle) area() float64 {
    return 3.14159 * c.radius * c.radius
}

func (c Circle) perimeter() float64 {
    return 2 * 3.14159 * c.radius
}

func (c Circle) draw() {
    fmt.Printf("Drawing circle with radius %.1f\n", c.radius)
}

func printShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.area(), s.perimeter())
}

func main() {
    rect := Rectangle{width: 4, height: 3}
    circle := Circle{radius: 2}
    
    printShapeInfo(rect)
    printShapeInfo(circle)
    
    shapes := []Drawable{rect, circle}
    for _, shape := range shapes {
        shape.draw()
    }
}
```

Go's interface system is based on method sets rather than explicit declarations.  
Any type that implements all methods of an interface automatically satisfies  
that interface, promoting loose coupling and flexible design.  


## Function with defer statements

Defer statements schedule function calls to execute when the surrounding  
function returns, useful for cleanup operations and resource management.  

```go
package main

import (
    "fmt"
    "os"
)

func processFile(filename string) error {
    fmt.Printf("Opening file: %s\n", filename)
    
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer func() {
        fmt.Println("Closing file")
        file.Close()
        os.Remove(filename) // cleanup for demo
    }()
    
    defer fmt.Println("Function ending")
    
    fmt.Println("Writing to file")
    _, err = file.WriteString("Hello there!\n")
    if err != nil {
        return err
    }
    
    fmt.Println("Processing complete")
    return nil
}

func demonstrateDefer() {
    defer fmt.Println("Third")
    defer fmt.Println("Second")
    defer fmt.Println("First")
    fmt.Println("Function body")
}

func main() {
    fmt.Println("=== Defer Stack Demo ===")
    demonstrateDefer()
    
    fmt.Println("\n=== File Processing Demo ===")
    if err := processFile("demo.txt"); err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

Deferred function calls are executed in LIFO (Last In, First Out) order.  
Defer is commonly used for resource cleanup, ensuring that cleanup code  
runs regardless of how the function exits (normal return or panic).  


## Error handling patterns

Go's explicit error handling using multiple return values creates robust  
and predictable error management patterns.  

```go
package main

import (
    "fmt"
    "errors"
    "strconv"
)

type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation error - %s: %s", e.Field, e.Message)
}

func validateAge(age string) (int, error) {
    if age == "" {
        return 0, ValidationError{Field: "age", Message: "cannot be empty"}
    }
    
    ageInt, err := strconv.Atoi(age)
    if err != nil {
        return 0, ValidationError{Field: "age", Message: "must be a number"}
    }
    
    if ageInt < 0 || ageInt > 150 {
        return 0, ValidationError{Field: "age", Message: "must be between 0 and 150"}
    }
    
    return ageInt, nil
}

func processUser(name, ageStr string) error {
    if name == "" {
        return errors.New("name cannot be empty")
    }
    
    age, err := validateAge(ageStr)
    if err != nil {
        return fmt.Errorf("failed to validate user: %w", err)
    }
    
    fmt.Printf("User processed: %s (age: %d)\n", name, age)
    return nil
}

func main() {
    users := [][]string{
        {"Alice", "30"},
        {"", "25"},
        {"Bob", "invalid"},
        {"Charlie", "200"},
        {"Diana", "28"},
    }
    
    for _, user := range users {
        name, age := user[0], user[1]
        if err := processUser(name, age); err != nil {
            fmt.Printf("Error processing user: %v\n", err)
        }
    }
}
```

Go's error handling emphasizes explicit error checking and propagation.  
Custom error types provide structured error information, and error wrapping  
preserves the error chain for better debugging.  


## Panic and recover

Panic and recover provide a mechanism for handling exceptional situations  
that shouldn't occur in normal program flow.  

```go
package main

import (
    "fmt"
    "runtime"
)

func safeExecute(fn func()) (recovered bool) {
    defer func() {
        if r := recover(); r != nil {
            recovered = true
            fmt.Printf("Recovered from panic: %v\n", r)
            
            // Print stack trace
            buf := make([]byte, 1024)
            runtime.Stack(buf, false)
            fmt.Printf("Stack trace:\n%s\n", buf)
        }
    }()
    
    fn()
    return false
}

func riskyOperation(value int) {
    if value < 0 {
        panic("negative values not allowed")
    }
    
    data := []int{1, 2, 3}
    if value >= len(data) {
        panic(fmt.Sprintf("index %d out of bounds", value))
    }
    
    fmt.Printf("Success: data[%d] = %d\n", value, data[value])
}

func main() {
    testValues := []int{1, -1, 5, 2}
    
    for _, val := range testValues {
        fmt.Printf("\nTesting value: %d\n", val)
        wasRecovered := safeExecute(func() {
            riskyOperation(val)
        })
        
        if !wasRecovered {
            fmt.Println("Operation completed successfully")
        }
    }
}
```

Panic should be used for unrecoverable errors or programming mistakes.  
Recover can only catch panics from the current goroutine and must be called  
from within a deferred function to be effective.  


## Function composition

Function composition allows combining simple functions to create more complex  
operations, promoting code reuse and modularity.  

```go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

type StringProcessor func(string) string

func compose(functions ...StringProcessor) StringProcessor {
    return func(input string) string {
        result := input
        for _, fn := range functions {
            result = fn(result)
        }
        return result
    }
}

func toLowerCase(s string) string {
    return strings.ToLower(s)
}

func removeSpaces(s string) string {
    return strings.ReplaceAll(s, " ", "")
}

func keepAlphanumeric(s string) string {
    var result strings.Builder
    for _, r := range s {
        if unicode.IsLetter(r) || unicode.IsDigit(r) {
            result.WriteRune(r)
        }
    }
    return result.String()
}

func addPrefix(prefix string) StringProcessor {
    return func(s string) string {
        return prefix + s
    }
}

func main() {
    input := "Hello World! 123"
    
    // Individual operations
    fmt.Printf("Original: '%s'\n", input)
    fmt.Printf("Lower: '%s'\n", toLowerCase(input))
    fmt.Printf("No spaces: '%s'\n", removeSpaces(input))
    fmt.Printf("Alphanumeric: '%s'\n", keepAlphanumeric(input))
    
    // Composed operations
    processor := compose(
        toLowerCase,
        keepAlphanumeric,
        addPrefix("processed_"),
    )
    
    result := processor(input)
    fmt.Printf("Composed result: '%s'\n", result)
    
    // Another composition
    cleaner := compose(toLowerCase, removeSpaces)
    fmt.Printf("Cleaned: '%s'\n", cleaner("  Hello WORLD  "))
}
```

Function composition creates pipelines of transformations that are easy to  
understand, test, and modify. This pattern is fundamental to functional  
programming and creates highly reusable code components.  


## Callback functions

Callback functions are passed as arguments to other functions and called  
at specific points during execution, enabling event-driven programming.  

```go
package main

import (
    "fmt"
    "time"
)

type EventCallback func(string, interface{})

func processData(data []int, onProgress EventCallback, onComplete EventCallback) {
    total := len(data)
    
    onProgress("started", map[string]interface{}{
        "total": total,
        "time":  time.Now(),
    })
    
    var results []int
    for i, item := range data {
        // Simulate processing time
        time.Sleep(10 * time.Millisecond)
        
        processed := item * item
        results = append(results, processed)
        
        onProgress("progress", map[string]interface{}{
            "completed": i + 1,
            "total":     total,
            "percent":   float64(i+1) / float64(total) * 100,
        })
    }
    
    onComplete("finished", map[string]interface{}{
        "results": results,
        "time":    time.Now(),
    })
}

func main() {
    data := []int{1, 2, 3, 4, 5}
    
    progressCallback := func(event string, data interface{}) {
        switch event {
        case "started":
            info := data.(map[string]interface{})
            fmt.Printf("Processing started: %d items\n", info["total"])
        case "progress":
            info := data.(map[string]interface{})
            fmt.Printf("Progress: %d/%d (%.1f%%)\n", 
                       info["completed"], info["total"], info["percent"])
        }
    }
    
    completeCallback := func(event string, data interface{}) {
        if event == "finished" {
            info := data.(map[string]interface{})
            results := info["results"].([]int)
            fmt.Printf("Processing complete! Results: %v\n", results)
        }
    }
    
    fmt.Println("Starting data processing with callbacks...")
    processData(data, progressCallback, completeCallback)
}
```

Callbacks enable loose coupling between components and allow for customizable  
behavior in frameworks and libraries. They're essential for asynchronous  
programming and event handling systems.  


## Decorator pattern with functions

The decorator pattern uses function composition to add behavior to existing  
functions without modifying their implementation.  

```go
package main

import (
    "fmt"
    "time"
)

type HandlerFunc func(string) string

func withLogging(fn HandlerFunc, name string) HandlerFunc {
    return func(input string) string {
        fmt.Printf("[LOG] %s called with input: %s\n", name, input)
        result := fn(input)
        fmt.Printf("[LOG] %s returned: %s\n", name, result)
        return result
    }
}

func withTiming(fn HandlerFunc) HandlerFunc {
    return func(input string) string {
        start := time.Now()
        result := fn(input)
        duration := time.Since(start)
        fmt.Printf("[TIMING] Function took: %v\n", duration)
        return result
    }
}

func withRetry(fn HandlerFunc, maxAttempts int) HandlerFunc {
    return func(input string) string {
        var result string
        for attempt := 1; attempt <= maxAttempts; attempt++ {
            result = fn(input)
            if result != "error" {
                break
            }
            fmt.Printf("[RETRY] Attempt %d failed, retrying...\n", attempt)
            time.Sleep(100 * time.Millisecond)
        }
        return result
    }
}

func greetUser(name string) string {
    if name == "error" {
        return "error"
    }
    return fmt.Sprintf("Hello there, %s!", name)
}

func processText(text string) string {
    time.Sleep(50 * time.Millisecond) // simulate work
    return fmt.Sprintf("Processed: %s", text)
}

func main() {
    // Basic functions
    fmt.Println("=== Basic Functions ===")
    fmt.Println(greetUser("Alice"))
    fmt.Println(processText("data"))
    
    // Decorated functions
    fmt.Println("\n=== Decorated Functions ===")
    
    decoratedGreet := withTiming(
        withLogging(greetUser, "greetUser"),
    )
    fmt.Println(decoratedGreet("Bob"))
    
    robustProcess := withRetry(
        withTiming(
            withLogging(processText, "processText"),
        ), 3,
    )
    fmt.Println(robustProcess("important data"))
    
    // Test retry mechanism
    fmt.Println("\n=== Testing Retry ===")
    flakyGreet := withRetry(greetUser, 3)
    fmt.Println(flakyGreet("error"))
}
```

Decorators provide a clean way to add cross-cutting concerns like logging,  
timing, authentication, and retry logic to functions without modifying the  
original implementation. This pattern promotes separation of concerns.  


## Function factories

Function factories are functions that create and return other functions,  
often with customized behavior based on parameters.  

```go
package main

import (
    "fmt"
    "math"
)

func createValidator(min, max int, fieldName string) func(int) error {
    return func(value int) error {
        if value < min {
            return fmt.Errorf("%s must be at least %d, got %d", fieldName, min, value)
        }
        if value > max {
            return fmt.Errorf("%s must be at most %d, got %d", fieldName, max, value)
        }
        return nil
    }
}

func createMathOperation(operation string) func(float64, float64) (float64, error) {
    switch operation {
    case "add":
        return func(a, b float64) (float64, error) {
            return a + b, nil
        }
    case "multiply":
        return func(a, b float64) (float64, error) {
            return a * b, nil
        }
    case "power":
        return func(base, exponent float64) (float64, error) {
            return math.Pow(base, exponent), nil
        }
    case "divide":
        return func(a, b float64) (float64, error) {
            if b == 0 {
                return 0, fmt.Errorf("cannot divide by zero")
            }
            return a / b, nil
        }
    default:
        return func(a, b float64) (float64, error) {
            return 0, fmt.Errorf("unknown operation: %s", operation)
        }
    }
}

func createFormatter(template string) func(...interface{}) string {
    return func(args ...interface{}) string {
        return fmt.Sprintf(template, args...)
    }
}

func main() {
    // Validator factories
    ageValidator := createValidator(0, 120, "age")
    scoreValidator := createValidator(0, 100, "score")
    
    fmt.Println("=== Validators ===")
    ages := []int{25, -5, 150}
    for _, age := range ages {
        if err := ageValidator(age); err != nil {
            fmt.Printf("Invalid age: %v\n", err)
        } else {
            fmt.Printf("Valid age: %d\n", age)
        }
    }
    
    // Math operation factories
    fmt.Println("\n=== Math Operations ===")
    add := createMathOperation("add")
    divide := createMathOperation("divide")
    power := createMathOperation("power")
    
    if result, err := add(10, 5); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("10 + 5 = %.2f\n", result)
    }
    
    if result, err := power(2, 8); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("2^8 = %.2f\n", result)
    }
    
    // Formatter factories
    fmt.Println("\n=== Formatters ===")
    logFormatter := createFormatter("[%s] %s: %s")
    userFormatter := createFormatter("User{name: %s, age: %d, email: %s}")
    
    fmt.Println(logFormatter("INFO", "2023-12-01", "Application started"))
    fmt.Println(userFormatter("Alice", 30, "alice@example.com"))
}
```

Function factories enable runtime customization of behavior and promote  
code reuse by parameterizing function creation. They're useful for creating  
configuration-driven systems and plugin architectures.  


## Memoization with closures

Memoization caches function results to improve performance for expensive  
computations with repeated inputs.  

```go
package main

import (
    "fmt"
    "time"
)

func memoize(fn func(int) int) func(int) int {
    cache := make(map[int]int)
    return func(n int) int {
        if result, exists := cache[n]; exists {
            fmt.Printf("Cache hit for %d\n", n)
            return result
        }
        fmt.Printf("Computing for %d\n", n)
        result := fn(n)
        cache[n] = result
        return result
    }
}

func expensiveFibonacci(n int) int {
    time.Sleep(100 * time.Millisecond) // Simulate expensive computation
    if n <= 1 {
        return n
    }
    return expensiveFibonacci(n-1) + expensiveFibonacci(n-2)
}

func createMemoizedFunction[T comparable, R any](fn func(T) R) func(T) R {
    cache := make(map[T]R)
    return func(input T) R {
        if result, exists := cache[input]; exists {
            return result
        }
        result := fn(input)
        cache[input] = result
        return result
    }
}

func expensiveStringProcess(s string) string {
    time.Sleep(200 * time.Millisecond)
    return fmt.Sprintf("processed_%s", s)
}

func main() {
    fmt.Println("=== Memoized Fibonacci ===")
    
    // Create memoized version
    memoizedFib := memoize(expensiveFibonacci)
    
    start := time.Now()
    fmt.Printf("fib(5) = %d\n", memoizedFib(5))
    fmt.Printf("First call took: %v\n", time.Since(start))
    
    start = time.Now()
    fmt.Printf("fib(5) = %d\n", memoizedFib(5))
    fmt.Printf("Second call took: %v\n", time.Since(start))
    
    start = time.Now()
    fmt.Printf("fib(6) = %d\n", memoizedFib(6))
    fmt.Printf("fib(6) call took: %v\n", time.Since(start))
    
    // Generic memoized function
    fmt.Println("\n=== Generic Memoized Function ===")
    memoizedStringProcess := createMemoizedFunction(expensiveStringProcess)
    
    inputs := []string{"data1", "data2", "data1", "data3", "data2"}
    for _, input := range inputs {
        start := time.Now()
        result := memoizedStringProcess(input)
        duration := time.Since(start)
        fmt.Printf("Input: %s, Result: %s, Time: %v\n", input, result, duration)
    }
}
```

Memoization trades memory for speed by caching computed results. It's  
particularly effective for recursive algorithms and functions with expensive  
computations that are called repeatedly with the same inputs.  


## Function pipelines

Function pipelines chain operations together, allowing data to flow through  
a series of transformations in a clean and readable manner.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

type Pipeline[T any] struct {
    value T
}

func NewPipeline[T any](value T) *Pipeline[T] {
    return &Pipeline[T]{value: value}
}

func (p *Pipeline[T]) Apply(fn func(T) T) *Pipeline[T] {
    p.value = fn(p.value)
    return p
}

func (p *Pipeline[T]) Get() T {
    return p.value
}

func numbersFilter(predicate func(int) bool) func([]int) []int {
    return func(numbers []int) []int {
        var result []int
        for _, num := range numbers {
            if predicate(num) {
                result = append(result, num)
            }
        }
        return result
    }
}

func numbersTransform(fn func(int) int) func([]int) []int {
    return func(numbers []int) []int {
        result := make([]int, len(numbers))
        for i, num := range numbers {
            result[i] = fn(num)
        }
        return result
    }
}

func main() {
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    
    result := NewPipeline(numbers).
        Apply(numbersFilter(func(n int) bool { return n%2 == 0 })).
        Apply(numbersTransform(func(n int) int { return n * n })).
        Get()
    
    fmt.Printf("Even squares: %v\n", result)
}
```

Pipelines provide a declarative way to express data transformations and  
make complex processing chains easy to understand and modify.  


## Functional options pattern

The functional options pattern provides a flexible way to configure functions  
and constructors with optional parameters while maintaining clean APIs.  

```go
package main

import (
    "fmt"
    "time"
)

type Server struct {
    host    string
    port    int
    timeout time.Duration
}

type ServerOption func(*Server)

func WithHost(host string) ServerOption {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) ServerOption {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) ServerOption {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func NewServer(options ...ServerOption) *Server {
    server := &Server{
        host:    "localhost",
        port:    8080,
        timeout: 30 * time.Second,
    }
    
    for _, option := range options {
        option(server)
    }
    
    return server
}

func main() {
    server := NewServer(
        WithHost("0.0.0.0"),
        WithPort(9090),
        WithTimeout(60*time.Second),
    )
    
    fmt.Printf("Server: %+v\n", server)
}
```

The functional options pattern provides excellent flexibility and extensibility  
while maintaining backward compatibility.  


## Context-aware functions

Context-aware functions accept context parameters for cancellation, timeouts,  
and carrying request-scoped values across function boundaries.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func simulateWork(ctx context.Context, name string, duration time.Duration) error {
    fmt.Printf("%s: Starting work\n", name)
    
    select {
    case <-time.After(duration):
        fmt.Printf("%s: Work completed\n", name)
        return nil
    case <-ctx.Done():
        fmt.Printf("%s: Work cancelled: %v\n", name, ctx.Err())
        return ctx.Err()
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    err := simulateWork(ctx, "Task1", 1*time.Second)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    err = simulateWork(ctx, "Task2", 3*time.Second)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

Context-aware functions enable proper cancellation, timeout handling, and  
request-scoped data propagation.  


## Generic functions

Generic functions work with multiple types using type parameters, providing  
type safety while maintaining flexibility.  

```go
package main

import "fmt"

func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

func main() {
    numbers := []int{1, 2, 3, 4, 5}
    
    doubled := Map(numbers, func(n int) int { return n * 2 })
    fmt.Printf("Doubled: %v\n", doubled)
    
    evens := Filter(numbers, func(n int) bool { return n%2 == 0 })
    fmt.Printf("Evens: %v\n", evens)
    
    sum := Reduce(numbers, 0, func(acc, n int) int { return acc + n })
    fmt.Printf("Sum: %d\n", sum)
}
```

Generic functions eliminate code duplication while maintaining type safety,  
enabling reusable algorithms that work with any type.  


## Function middleware

Middleware functions wrap other functions to provide cross-cutting concerns  
like authentication, logging, and request processing.  

```go
package main

import (
    "fmt"
    "time"
)

type Handler func(string) (string, error)

func authMiddleware(next Handler) Handler {
    return func(input string) (string, error) {
        if input == "unauthorized" {
            return "", fmt.Errorf("authentication failed")
        }
        fmt.Println("Auth: Authenticated successfully")
        return next(input)
    }
}

func loggingMiddleware(next Handler) Handler {
    return func(input string) (string, error) {
        start := time.Now()
        fmt.Printf("Log: Processing request with input: %s\n", input)
        
        result, err := next(input)
        
        duration := time.Since(start)
        if err != nil {
            fmt.Printf("Log: Request failed after %v: %v\n", duration, err)
        } else {
            fmt.Printf("Log: Request completed in %v\n", duration)
        }
        
        return result, err
    }
}

func businessHandler(input string) (string, error) {
    time.Sleep(100 * time.Millisecond) // Simulate processing
    return fmt.Sprintf("processed_%s", input), nil
}

func main() {
    // Chain middleware
    handler := loggingMiddleware(
        authMiddleware(
            businessHandler,
        ),
    )
    
    testInputs := []string{"valid_data", "unauthorized", "another_request"}
    
    for _, input := range testInputs {
        fmt.Printf("\n=== Processing: %s ===\n", input)
        result, err := handler(input)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Printf("Result: %s\n", result)
        }
    }
}
```

Middleware patterns enable clean separation of concerns and reusable  
cross-cutting functionality that can be applied to multiple handlers.  


## Function benchmarking

Benchmarking functions measure and compare performance characteristics  
of different implementations.  

```go
package main

import (
    "fmt"
    "time"
)

func benchmark(name string, fn func(), iterations int) {
    fmt.Printf("\nBenchmarking %s with %d iterations...\n", name, iterations)
    
    start := time.Now()
    for i := 0; i < iterations; i++ {
        fn()
    }
    duration := time.Since(start)
    
    avgTime := duration / time.Duration(iterations)
    fmt.Printf("%s: Total=%v, Average=%v, Ops/sec=%.0f\n", 
               name, duration, avgTime, float64(iterations)/duration.Seconds())
}

func fibonacciRecursive(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacciRecursive(n-1) + fibonacciRecursive(n-2)
}

func fibonacciIterative(n int) int {
    if n <= 1 {
        return n
    }
    a, b := 0, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}

func main() {
    const n = 20
    const iterations = 1000
    
    benchmark("Recursive Fibonacci", func() {
        fibonacciRecursive(n)
    }, iterations)
    
    benchmark("Iterative Fibonacci", func() {
        fibonacciIterative(n)
    }, iterations)
    
    // String operations benchmark
    benchmark("String Concatenation", func() {
        result := ""
        for i := 0; i < 100; i++ {
            result += "x"
        }
    }, 10000)
    
    benchmark("String Builder", func() {
        var builder fmt.Sprintf("")
        for i := 0; i < 100; i++ {
            builder += "x"
        }
    }, 10000)
}
```

Benchmarking functions help identify performance bottlenecks and compare  
the efficiency of different algorithms and implementations.  


## Function testing helpers

Testing helper functions create reusable test utilities and assertion  
functions for comprehensive testing.  

```go
package main

import (
    "fmt"
    "reflect"
    "testing"
)

func assertEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}

func assertNotEqual[T comparable](t *testing.T, got, want T) {
    t.Helper()
    if got == want {
        t.Errorf("got %v, expected it to be different", got)
    }
}

func assertError(t *testing.T, err error, wantErr bool) {
    t.Helper()
    if (err != nil) != wantErr {
        t.Errorf("error = %v, wantErr %v", err, wantErr)
    }
}

func assertSliceEqual[T comparable](t *testing.T, got, want []T) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}

func testWithTable[T any](t *testing.T, testCases []struct {
    name string
    test func(*testing.T) T
    want T
}) {
    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            got := tc.test(t)
            if !reflect.DeepEqual(got, tc.want) {
                t.Errorf("got %v, want %v", got, tc.want)
            }
        })
    }
}

// Example usage (this is for demonstration)
func add(a, b int) int {
    return a + b
}

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}

func main() {
    // This would typically be in a _test.go file
    fmt.Println("Testing helper functions demonstrated")
    fmt.Println("These would be used in actual test functions with *testing.T")
    
    // Example of how these would be used:
    // func TestAdd(t *testing.T) {
    //     assertEqual(t, add(2, 3), 5)
    //     assertNotEqual(t, add(2, 3), 6)
    // }
    
    // func TestDivide(t *testing.T) {
    //     result, err := divide(10, 2)
    //     assertError(t, err, false)
    //     assertEqual(t, result, 5.0)
    //     
    //     _, err = divide(10, 0)
    //     assertError(t, err, true)
    // }
}
```

Testing helper functions reduce boilerplate code in tests and provide  
consistent assertion patterns across test suites.  


## Function serialization

Functions for serializing and deserializing data structures provide  
consistent data transformation patterns.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strconv"
    "strings"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func serializeJSON[T any](data T) (string, error) {
    bytes, err := json.Marshal(data)
    if err != nil {
        return "", fmt.Errorf("failed to serialize: %w", err)
    }
    return string(bytes), nil
}

func deserializeJSON[T any](data string, target *T) error {
    err := json.Unmarshal([]byte(data), target)
    if err != nil {
        return fmt.Errorf("failed to deserialize: %w", err)
    }
    return nil
}

func serializeCSV(users []User) string {
    var lines []string
    lines = append(lines, "id,name,email") // header
    
    for _, user := range users {
        line := fmt.Sprintf("%d,%s,%s", user.ID, user.Name, user.Email)
        lines = append(lines, line)
    }
    
    return strings.Join(lines, "\n")
}

func deserializeCSV(data string) ([]User, error) {
    lines := strings.Split(data, "\n")
    if len(lines) < 2 { // header + at least one data line
        return nil, fmt.Errorf("invalid CSV format")
    }
    
    var users []User
    for i, line := range lines[1:] { // skip header
        fields := strings.Split(line, ",")
        if len(fields) != 3 {
            return nil, fmt.Errorf("invalid CSV line %d: %s", i+2, line)
        }
        
        id, err := strconv.Atoi(fields[0])
        if err != nil {
            return nil, fmt.Errorf("invalid ID in line %d: %s", i+2, fields[0])
        }
        
        users = append(users, User{
            ID:    id,
            Name:  fields[1],
            Email: fields[2],
        })
    }
    
    return users, nil
}

func main() {
    users := []User{
        {ID: 1, Name: "Alice", Email: "alice@example.com"},
        {ID: 2, Name: "Bob", Email: "bob@example.com"},
        {ID: 3, Name: "Charlie", Email: "charlie@example.com"},
    }
    
    // JSON serialization
    jsonData, err := serializeJSON(users)
    if err != nil {
        fmt.Printf("JSON serialization error: %v\n", err)
        return
    }
    fmt.Printf("JSON: %s\n", jsonData)
    
    var deserializedUsers []User
    err = deserializeJSON(jsonData, &deserializedUsers)
    if err != nil {
        fmt.Printf("JSON deserialization error: %v\n", err)
        return
    }
    fmt.Printf("Deserialized: %+v\n", deserializedUsers)
    
    // CSV serialization
    csvData := serializeCSV(users)
    fmt.Printf("\nCSV:\n%s\n", csvData)
    
    csvUsers, err := deserializeCSV(csvData)
    if err != nil {
        fmt.Printf("CSV deserialization error: %v\n", err)
        return
    }
    fmt.Printf("CSV Users: %+v\n", csvUsers)
}
```

Serialization functions provide consistent data format conversion and  
enable interoperability between different systems and storage formats.  


## Function pooling

Function pooling manages reusable function instances and resources  
for improved performance in high-throughput scenarios.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type WorkerPool struct {
    workers    int
    jobs       chan func()
    wg         sync.WaitGroup
    quit       chan struct{}
    workerFunc func(int, <-chan func(), <-chan struct{})
}

func NewWorkerPool(workers int) *WorkerPool {
    return &WorkerPool{
        workers: workers,
        jobs:    make(chan func(), 100),
        quit:    make(chan struct{}),
        workerFunc: func(id int, jobs <-chan func(), quit <-chan struct{}) {
            fmt.Printf("Worker %d started\n", id)
            for {
                select {
                case job := <-jobs:
                    if job != nil {
                        job()
                    }
                case <-quit:
                    fmt.Printf("Worker %d stopping\n", id)
                    return
                }
            }
        },
    }
}

func (p *WorkerPool) Start() {
    for i := 1; i <= p.workers; i++ {
        p.wg.Add(1)
        go func(workerID int) {
            defer p.wg.Done()
            p.workerFunc(workerID, p.jobs, p.quit)
        }(i)
    }
}

func (p *WorkerPool) Submit(job func()) {
    select {
    case p.jobs <- job:
    case <-time.After(1 * time.Second):
        fmt.Println("Job submission timeout")
    }
}

func (p *WorkerPool) Stop() {
    close(p.quit)
    close(p.jobs)
    p.wg.Wait()
}

func createJob(id int) func() {
    return func() {
        fmt.Printf("Processing job %d\n", id)
        time.Sleep(100 * time.Millisecond) // Simulate work
        fmt.Printf("Job %d completed\n", id)
    }
}

func main() {
    pool := NewWorkerPool(3)
    pool.Start()
    
    // Submit jobs
    for i := 1; i <= 10; i++ {
        job := createJob(i)
        pool.Submit(job)
    }
    
    // Wait a bit for jobs to complete
    time.Sleep(2 * time.Second)
    
    // Shutdown pool
    pool.Stop()
    fmt.Println("All workers stopped")
}
```

Function pooling enables efficient resource utilization and controlled  
concurrency for CPU-intensive or I/O-bound operations.  


## Function caching strategies

Advanced caching strategies for functions improve performance through  
intelligent cache management and eviction policies.  

```go
package main

import (
    "fmt"
    "time"
)

type CacheEntry[T any] struct {
    value      T
    expiration time.Time
    accessTime time.Time
}

type Cache[K comparable, V any] struct {
    data       map[K]*CacheEntry[V]
    maxSize    int
    defaultTTL time.Duration
}

func NewCache[K comparable, V any](maxSize int, defaultTTL time.Duration) *Cache[K, V] {
    return &Cache[K, V]{
        data:       make(map[K]*CacheEntry[V]),
        maxSize:    maxSize,
        defaultTTL: defaultTTL,
    }
}

func (c *Cache[K, V]) Get(key K) (V, bool) {
    entry, exists := c.data[key]
    if !exists {
        var zero V
        return zero, false
    }
    
    if time.Now().After(entry.expiration) {
        delete(c.data, key)
        var zero V
        return zero, false
    }
    
    entry.accessTime = time.Now()
    return entry.value, true
}

func (c *Cache[K, V]) Set(key K, value V) {
    if len(c.data) >= c.maxSize {
        c.evictLRU()
    }
    
    c.data[key] = &CacheEntry[V]{
        value:      value,
        expiration: time.Now().Add(c.defaultTTL),
        accessTime: time.Now(),
    }
}

func (c *Cache[K, V]) evictLRU() {
    var oldestKey K
    var oldestTime time.Time
    first := true
    
    for key, entry := range c.data {
        if first || entry.accessTime.Before(oldestTime) {
            oldestKey = key
            oldestTime = entry.accessTime
            first = false
        }
    }
    
    delete(c.data, oldestKey)
    fmt.Printf("Evicted key from cache\n")
}

func WithCache[K comparable, V any](fn func(K) V, cache *Cache[K, V]) func(K) V {
    return func(key K) V {
        if value, hit := cache.Get(key); hit {
            fmt.Printf("Cache hit for key: %v\n", key)
            return value
        }
        
        fmt.Printf("Cache miss for key: %v, computing...\n", key)
        value := fn(key)
        cache.Set(key, value)
        return value
    }
}

func expensiveComputation(n int) int {
    time.Sleep(200 * time.Millisecond)
    return n * n * n
}

func main() {
    cache := NewCache[int, int](3, 5*time.Second)
    cachedCompute := WithCache(expensiveComputation, cache)
    
    testValues := []int{1, 2, 3, 1, 4, 2, 5, 1}
    
    for _, val := range testValues {
        start := time.Now()
        result := cachedCompute(val)
        duration := time.Since(start)
        fmt.Printf("f(%d) = %d, took %v\n", val, result, duration)
    }
}
```

Advanced caching strategies optimize memory usage and cache hit rates  
through intelligent eviction policies and expiration management.  


## Function composition with error handling

Error-aware function composition chains operations while gracefully  
handling and propagating errors through the pipeline.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

type Result[T any] struct {
    value T
    err   error
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, err: nil}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{value: zero, err: err}
}

func (r Result[T]) IsOk() bool {
    return r.err == nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.value, r.err
}

func (r Result[T]) Map[U any](fn func(T) U) Result[U] {
    if r.err != nil {
        return Err[U](r.err)
    }
    return Ok(fn(r.value))
}

func (r Result[T]) FlatMap[U any](fn func(T) Result[U]) Result[U] {
    if r.err != nil {
        return Err[U](r.err)
    }
    return fn(r.value)
}

func parseNumber(s string) Result[int] {
    num, err := strconv.Atoi(strings.TrimSpace(s))
    if err != nil {
        return Err[int](fmt.Errorf("invalid number: %s", s))
    }
    return Ok(num)
}

func validatePositive(n int) Result[int] {
    if n <= 0 {
        return Err[int](fmt.Errorf("number must be positive: %d", n))
    }
    return Ok(n)
}

func computeSquareRoot(n int) Result[float64] {
    if n < 0 {
        return Err[float64](fmt.Errorf("cannot compute square root of negative number"))
    }
    
    // Simple approximation
    x := float64(n)
    for i := 0; i < 10; i++ {
        x = (x + float64(n)/x) / 2
    }
    return Ok(x)
}

func processInput(input string) Result[string] {
    return parseNumber(input).
        FlatMap(validatePositive).
        FlatMap(computeSquareRoot).
        Map(func(sqrt float64) string {
            return fmt.Sprintf("√%s = %.6f", input, sqrt)
        })
}

func main() {
    testInputs := []string{"16", "25", "0", "-4", "invalid", "100"}
    
    for _, input := range testInputs {
        result := processInput(input)
        if result.IsOk() {
            value, _ := result.Unwrap()
            fmt.Printf("Success: %s\n", value)
        } else {
            _, err := result.Unwrap()
            fmt.Printf("Error processing '%s': %v\n", input, err)
        }
    }
}
```

Error-aware composition provides clean error handling while maintaining  
the benefits of functional composition and pipeline processing.  
