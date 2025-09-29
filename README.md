# Go examples

Go is a statically typed, compiled programming language designed at Google by  
Robert Griesemer, Rob Pike, and Ken Thompson. First announced in 2009 and  
reaching version 1.0 in 2012, Go was created to address the challenges of  
modern software development at scale. The language combines the performance  
and safety of statically typed languages with the ease of programming of  
dynamically typed languages.  

The primary motivation behind Go's creation was to solve real-world problems  
that Google's engineers faced daily. As software systems became increasingly  
complex and distributed, traditional languages like C++ showed their age with  
slow compilation times, complex dependency management, and difficult  
concurrent programming models. Java and C# brought garbage collection and  
memory safety but came with heavy runtime environments and verbose syntax.  
Dynamic languages like Python and JavaScript offered development speed but  
lacked the performance and type safety needed for large-scale systems.  

Go's design philosophy centers around simplicity, clarity, and efficiency. The  
language deliberately omits many features found in other modern languages,  
such as inheritance, generics (until Go 1.18), exceptions, and operator  
overloading. This minimalist approach reduces cognitive load and makes code  
more predictable and maintainable. The designers believed that complexity in  
programming languages often hinders rather than helps developers, leading to  
bugs and reduced productivity.  

The language excels in several key areas that make it particularly attractive  
for modern software development. Go provides built-in support for concurrent  
programming through goroutines and channels, making it natural to write  
efficient concurrent code without the complexity of traditional threading  
models. The garbage collector is optimized for low-latency applications,  
making Go suitable for server applications that require consistent response  
times. Fast compilation speeds enable rapid development cycles, with most  
programs compiling in seconds rather than minutes.  

When compared to other programming languages, Go occupies a unique position  
in the ecosystem. Against C and C++, Go offers memory safety through garbage  
collection while maintaining similar performance characteristics. The language  
eliminates entire classes of bugs related to memory management, buffer  
overflows, and dangling pointers that plague C/C++ development. Compared to  
Java and C#, Go produces smaller, self-contained binaries without requiring  
a runtime environment, simplifying deployment and reducing memory overhead.  

In contrast to dynamic languages like Python, Ruby, and JavaScript, Go  
provides compile-time type checking that catches errors before runtime while  
maintaining readability and development speed. The static typing system helps  
with code documentation, IDE support, and refactoring capabilities. Unlike  
Node.js, which uses an event loop for concurrency, Go's goroutines provide  
a more intuitive model for concurrent programming that scales naturally with  
available CPU cores.  

Go's approach to object-oriented programming differs significantly from  
traditional OOP languages. Instead of classes and inheritance, Go uses struct  
types and composition. Interfaces are satisfied implicitly, meaning types  
implement interfaces automatically when they have the required methods. This  
design promotes loose coupling and makes code more flexible and testable.  

The language's standard library is comprehensive and well-designed, covering  
everything from basic data structures to HTTP servers, JSON processing, and  
cryptography. The library follows consistent conventions and error handling  
patterns, making it easy to learn and use effectively. Package management  
through the module system ensures reproducible builds and dependency  
resolution.  

Go has found particular success in cloud computing, microservices, DevOps  
tools, and network programming. Major companies including Google, Uber,  
Dropbox, Docker, and Kubernetes rely heavily on Go for their infrastructure.  
The language's efficiency in resource utilization makes it cost-effective  
for cloud deployments, while its simplicity reduces the barrier to entry  
for new team members.  

## Installation and Setup  

To begin developing with Go, you need to install the Go toolchain on your  
system. The installation process varies by operating system but is generally  
straightforward and well-documented.  

### Installing Go  

For most users, the easiest installation method is downloading the official  
binary distribution from the Go website at https://golang.org/dl/. The site  
provides installers for Windows, macOS, and Linux that handle the installation  
process automatically.  

On macOS, you can also install Go using Homebrew:  
```
$ brew install go
```

On Ubuntu or Debian-based Linux systems, you can use the package manager:  
```
$ sudo apt update
$ sudo apt install golang-go
```

On Windows, download the MSI installer and follow the installation wizard.  
The installer sets up the necessary environment variables automatically.  

### Configuring Your Environment  

After installation, verify that Go is properly installed by checking the  
version:  
```
$ go version
go version go1.21.0 linux/amd64
```

Go uses several environment variables to control its behavior. The most  
important ones are automatically configured by the installer, but it's useful  
to understand them:  

- `GOROOT`: Points to the Go installation directory  
- `GOPATH`: Workspace directory for Go code (less important with modules)  
- `GOPROXY`: Proxy server for downloading modules  
- `GO111MODULE`: Controls module behavior (auto by default)  

You can view all Go environment variables with:  
```
$ go env
```

### Setting Up Your Development Environment  

Go works well with many text editors and IDEs. Popular choices include:  

**Visual Studio Code** with the Go extension provides excellent language  
support, debugging capabilities, and integration with Go tools. The extension  
offers features like code completion, error highlighting, and automated  
refactoring.  

**GoLand** by JetBrains is a full-featured IDE specifically designed for Go  
development. It provides advanced debugging, testing tools, and code analysis  
features.  

**Vim** and **Neovim** users can use the vim-go plugin for comprehensive Go  
support including syntax highlighting, code completion, and tool integration.  

For beginners, Visual Studio Code with the Go extension provides the best  
balance of functionality and ease of use.  

### Creating Your First Go Program  

Let's create a simple program to verify your setup works correctly. First,  
create a new directory for your project and initialize a Go module:  

```
$ mkdir hello-go
$ cd hello-go
$ go mod init example.com/hello
```

The `go mod init` command creates a `go.mod` file that defines your module  
and tracks dependencies. The module path should be a unique identifier,  
typically based on a domain you control.  

## Basic Language Concepts  

Go's syntax is clean and readable, drawing inspiration from C while  
eliminating much of its complexity. The language uses a package-based  
organization system where every Go source file belongs to a package.  

### Hello There Example  

Here's a traditional first program that demonstrates basic Go syntax:  

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello there, Go programmer!")
    
    name := "Alice"
    age := 28
    
    fmt.Printf("%s is %d years old\n", name, age)
}
```

This program introduces several fundamental concepts. The `package main`  
declaration indicates this is an executable program rather than a library.  
The `import "fmt"` statement makes the formatting functions available. The  
`main` function serves as the entry point for execution.  

The variable declarations show Go's type inference capability where the  
compiler determines types automatically. The `:=` operator declares and  
initializes variables in one step, while `Printf` provides formatted output  
similar to C's printf function.  

### Variables and Types  

Go provides several ways to declare variables, each appropriate for different  
situations. Understanding these patterns is essential for writing idiomatic  
Go code.  

```go
package main

import "fmt"

func main() {
    // Explicit type declaration
    var message string = "Hello there"
    var count int = 42
    var price float64 = 19.99
    var active bool = true
    
    // Type inference with var
    var name = "Bob"
    var number = 100
    
    // Short declaration (only inside functions)
    city := "New York"
    population := 8_336_817
    
    fmt.Println(message, count, price, active)
    fmt.Println(name, number)
    fmt.Println(city, population)
}
```

The example demonstrates three declaration styles. Explicit type declarations  
are useful when you need a specific type that can't be inferred. Type  
inference with `var` allows the compiler to determine appropriate types.  
Short declaration with `:=` provides concise syntax for local variables.  

Note the use of underscores in the number literal `8_336_817`, which improves  
readability for large numbers without affecting the value.  

### Basic Data Types  

Go provides a rich set of built-in types that cover most programming needs  
while maintaining simplicity and performance.  

```go
package main

import "fmt"

func main() {
    // Integer types
    var smallInt int8 = 127
    var bigInt int64 = 9223372036854775807
    var unsignedInt uint32 = 4294967295
    
    // Floating point types
    var smallFloat float32 = 3.14159
    var bigFloat float64 = 2.718281828459045
    
    // String and boolean
    var text string = "Hello there, Go!"
    var flag bool = true
    
    // Complex numbers
    var complex64Num complex64 = 1 + 2i
    var complex128Num complex128 = 3 + 4i
    
    fmt.Printf("Integer types: %d, %d, %d\n", smallInt, bigInt, unsignedInt)
    fmt.Printf("Float types: %.2f, %.6f\n", smallFloat, bigFloat)
    fmt.Printf("String and bool: %s, %t\n", text, flag)
    fmt.Printf("Complex types: %v, %v\n", complex64Num, complex128Num)
}
```

Go's type system is designed for clarity and performance. Integer types come  
in various sizes (8, 16, 32, 64 bits) and can be signed or unsigned. The  
plain `int` type uses the most efficient size for the target platform,  
typically 64 bits on modern systems.  

Floating-point types follow IEEE 754 standards with `float32` providing  
single precision and `float64` providing double precision. For most  
applications, `float64` is the preferred choice due to its higher precision.  

## Version

```
$ go version
go version go1.22.2 linux/amd64
```

## Create new module 

```
$ go mod init com.zetcode/simple
```

We create a new module with `go mod init` command. It produces a `go.mod`  
file. The module path serves as a unique identifier and import path for  
your project. Modern Go development relies on modules for dependency  
management and versioning, replacing the older GOPATH-based workflow.  

## Shorthand variable declaration 

The short variable declaration operator `:=` provides a concise way to declare  
and initialize variables within functions. This syntax reduces boilerplate  
while maintaining type safety through Go's type inference system.  

```go
package main

import "fmt"

func main() {

    name := "John Doe"
    age := 34

    fmt.Printf("%s is %d years old\n", name, age)
}
```

This example demonstrates the most common variable declaration pattern in Go.  
The compiler automatically infers that `name` is a string and `age` is an  
integer based on the assigned values. The short declaration syntax is only  
available within function bodies and is preferred for local variables.

## Constants

Constants are created with the `const` keyword. They cannot be modified.  
Constants provide compile-time guarantees and eliminate magic numbers from  
your code, improving readability and maintainability.  

```go
package main

import "fmt"

func main() {

    var age int = 34
    const WIDTH = 100

    age = 35
    age = 36

    // WIDTH = 101

    fmt.Println(age, WIDTH)
}
```

This example shows the difference between variables and constants. The `age`  
variable can be reassigned multiple times, while the `WIDTH` constant cannot  
be changed after declaration. Attempting to modify a constant results in a  
compilation error, as shown by the commented line.

## Type inference

Go's type inference allows the compiler to automatically determine variable  
types based on assigned values. This feature reduces code verbosity while  
maintaining the benefits of static typing.  

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {

    var name = "John Doe"
    var age = 34

    fmt.Println(reflect.TypeOf(name))
    fmt.Println(reflect.TypeOf(age))

    fmt.Printf("%s is %d years old\n", name, age)
}
```

This example demonstrates how Go infers types automatically. The `reflect`  
package allows us to examine the actual types at runtime, showing that `name`  
is inferred as `string` and `age` as `int`. Type inference works with any  
expression, not just literals.


## Swapping values in function

We use pointers for this. Pointers allow functions to modify variables from  
the calling scope by passing memory addresses instead of copying values.  
This technique is essential for efficient data manipulation and avoiding  
unnecessary memory allocations.  

```go
package main

import (
    "fmt"
)

func main() {

    var x int = 5
    var y int = 8
	
    fmt.Println(x, y)
    swap(&x, &y)
    fmt.Println(x, y)
}

func swap(x, y *int) {
    var temp int = *x
    *x = *y
    *y = temp
}
```

This example demonstrates pointer usage for modifying function arguments. The  
`&` operator gets the address of a variable, while the `*` operator  
dereferences a pointer to access its value. The swap function receives  
pointers and modifies the original variables through pointer dereferencing.

## Builder pattern 

The builder pattern provides a flexible way to construct complex objects  
step by step. In Go, this pattern uses method chaining to configure object  
properties before creating the final instance.  

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

type httpClientBuilder struct {
    timeout time.Duration
}

func NewBuilder() *httpClientBuilder {
    return &httpClientBuilder{timeout: 5 * time.Second}
}

func (b *httpClientBuilder) Build() *http.Client {
    return &http.Client{Timeout: b.timeout}
}

func (b *httpClientBuilder) Timeout(t time.Duration) *httpClientBuilder {
    b.timeout = t
    return b
}

func main() {

    client := NewBuilder().Timeout(5 * time.Second).Build()

    url := "https://webcode.me"
    res, err := client.Head(url)

    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(res.Status)
}
```

This example creates an HTTP client using the builder pattern. The pattern  
allows for fluent, readable configuration while maintaining immutability and  
type safety. Each configuration method returns the builder, enabling method  
chaining for concise object construction.

## The any type

The `any` type is a built-in alias for the interface{} type, which can hold  
any value. Introduced in Go 1.18, `any` provides a more intuitive and readable  
way to indicate that a variable can be of `any` type.  

```go
package main

import "fmt"

func main() {

    var val any

    val = "hello"
    fmt.Printf("%T\n", val)

    val = 3
    fmt.Printf("%T\n", val)

    val = 4.5
    fmt.Printf("%T\n", val)

    PrintValues(42, "hello", 3.14, true)
}

func PrintValues(values ...any) {
    for _, value := range values {
        fmt.Println(value)
    }
}
```

This example demonstrates the flexibility of the `any` type. The same variable  
can hold different types throughout its lifetime. The `PrintValues` function  
accepts a variadic parameter of `any` type, allowing it to process arguments  
of different types in a single call.

When dealing with values of type any, you often use type assertions or  
reflection to handle the specific underlying type.  

```go
package main

import "fmt"

func main() {

    PrintStrings(42, "hello", 3.14, "book", true, "falcon")
}

func PrintStrings(values ...any) {

    for _, val := range values {

        if str, ok := val.(string); ok {
            fmt.Println(str)
        }
    }
}
```

Type assertions allow you to extract the concrete value from an interface.  
The comma ok idiom safely checks whether the assertion succeeds. This example  
filters and prints only string values from a mixed-type argument list,  
demonstrating safe type checking at runtime.

Using generics:  

Generics, introduced in Go 1.18, provide type-safe alternatives to using  
`any` for operations that work with multiple types. They offer better  
performance and compile-time type checking.  

```go
package main

import (
    "fmt"   
)

// PrintAll is a generic function that prints all elements in a slice
func PrintAll[T any](items []T) {
    for _, item := range items {
        fmt.Println(item)
    }
}

func main() {
    // Create slices of different types
    intSlice := []int{1, 2, 3, 4, 5}
    stringSlice := []string{"apple", "banana", "cherry"}
    floatSlice := []float64{1.1, 2.2, 3.3}

    // Use the generic function to print slices of different types
    fmt.Println("Integers:")
    PrintAll(intSlice)

    fmt.Println("\nStrings:")
    PrintAll(stringSlice)

    fmt.Println("\nFloats:")
    PrintAll(floatSlice)
}
```

This example shows how generics eliminate the need for type assertions while  
maintaining type safety. The `PrintAll` function works with any type `T`,  
providing better performance than interface{}-based solutions and catching  
type errors at compile time rather than runtime.

## DeepSeek via API

This example demonstrates HTTP client usage for API interactions, JSON  
marshaling, and error handling in a real-world scenario. It shows how Go's  
standard library provides comprehensive tools for web service integration.  

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

type ChatRequest struct {
    Model    string    `json:"model"`
    Messages []Message `json:"messages"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type ChatResponse struct {
    Choices []Choice `json:"choices"`
}

type Choice struct {
    Message Message `json:"message"`
}

func main() {
    apiKey := os.Getenv("DEEPSEEK_API_KEY")
    if apiKey == "" {
        fmt.Println("Please set the DEEPSEEK_API_KEY environment variable")
        return
    }

    prompt := "What is NetBSD in a sentence?"
    request := ChatRequest{
        Model: "deepseek-chat",
        Messages: []Message{
            {Role: "user", Content: prompt},
        },
    }

    jsonData, err := json.Marshal(request)
    if err != nil {
        fmt.Printf("Error marshaling request: %v\n", err)
        return
    }

    req, err := http.NewRequest("POST", "https://api.deepseek.com/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error sending request: %v\n", err)
        return
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response: %v\n", err)
        return
    }

    var chatResp ChatResponse
    if err := json.Unmarshal(body, &chatResp); err != nil {
        fmt.Printf("Error unmarshaling response: %v\n", err)
        return
    }

    if len(chatResp.Choices) > 0 {
        fmt.Printf("Response: %s\n", chatResp.Choices[0].Message.Content)
    } else {
        fmt.Println("No content in response")
    }
}
```

This example showcases several important Go concepts: struct tags for JSON  
serialization, environment variable access, HTTP client configuration, and  
comprehensive error handling. The code demonstrates how Go's explicit error  
handling prevents silent failures while maintaining clean, readable structure.

## Tool call with DeepSeek API

This advanced example extends the previous API integration to demonstrate  
complex data structures, function definitions, and conditional processing.  
It illustrates how Go handles sophisticated API interactions with nested  
JSON structures and dynamic content.  

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
    "time"
)

type ChatRequest struct {
    Model    string    `json:"model"`
    Messages []Message `json:"messages"`
    Tools    []Tool    `json:"tools,omitempty"`
}

type Message struct {
    Role      string     `json:"role"`
    Content   string     `json:"content"`
    ToolCalls []ToolCall `json:"tool_calls,omitempty"`
}

type ChatResponse struct {
    Choices []Choice `json:"choices"`
}

type Choice struct {
    Message Message `json:"message"`
}

type Tool struct {
    Type     string   `json:"type"`
    Function Function `json:"function"`
}

type Function struct {
    Name        string                 `json:"name"`
    Description string                 `json:"description"`
    Parameters  map[string]interface{} `json:"parameters"`
}

type ToolCall struct {
    ID       string   `json:"id"`
    Type     string   `json:"type"`
    Function Function `json:"function"`
}

func getCurrentDateTime() string {
    return time.Now().Format("2006-01-02 15:04:05")
}

func main() {
    apiKey := os.Getenv("DEEPSEEK_API_KEY")
    if apiKey == "" {
        fmt.Println("Please set the DEEPSEEK_API_KEY environment variable")
        return
    }

    prompt := "Who was Napolen. What time is it?"
    tool := Tool{
        Type: "function",
        Function: Function{
            Name:        "get_current_datetime",
            Description: "Get the current date and time",
            Parameters: map[string]interface{}{
                "type":       "object",
                "properties": map[string]interface{}{},
            },
        },
    }
    request := ChatRequest{
        Model: "deepseek-chat",
        Messages: []Message{
            {Role: "user", Content: prompt},
        },
        Tools: []Tool{tool},
    }

    jsonData, err := json.Marshal(request)
    if err != nil {
        fmt.Printf("Error marshaling request: %v\n", err)
        return
    }

    req, err := http.NewRequest("POST", "https://api.deepseek.com/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error sending request: %v\n", err)
        return
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response: %v\n", err)
        return
    }

    var chatResp ChatResponse
    if err := json.Unmarshal(body, &chatResp); err != nil {
        fmt.Printf("Error unmarshaling response: %v\n", err)
        return
    }

    if len(chatResp.Choices) > 0 {
        message := chatResp.Choices[0].Message
        if message.Content != "" {
            fmt.Printf("Response: %s\n", message.Content)
        }
        if len(message.ToolCalls) > 0 {
            for _, toolCall := range message.ToolCalls {
                if toolCall.Function.Name == "get_current_datetime" {
                    result := getCurrentDateTime()
                    fmt.Printf("Tool call result: %s\n", result)
                }
            }
        }
    } else {
        fmt.Println("No content in response")
    }

    fmt.Println("Request completed at:", time.Now().Format(time.RFC1123))
}
```

This comprehensive example demonstrates advanced Go features including  
complex nested structures, optional JSON fields using the `omitempty` tag,  
function calls based on dynamic content, and time formatting using Go's  
unique reference time format. It showcases how Go's type system and standard  
library enable robust API integration with minimal external dependencies.
