# Go Packages and Imports

Go packages are the fundamental unit of code organization and modularization in  
the Go programming language. A package is a collection of Go source files in  
the same directory that are compiled together. Packages serve as namespaces  
for identifiers and provide encapsulation, code reuse, and modularity to Go  
programs. Every Go program is made up of packages, with program execution  
beginning in the main package.

The package system in Go is designed around simplicity and efficiency. Each  
source file belongs to exactly one package, declared at the top of the file  
with a package statement. The package name should match the directory name  
containing the package files, though this is not strictly enforced for the  
main package. Go's convention encourages lowercase package names that are  
short and clear.

Imports in Go allow you to use code from other packages, including the  
standard library and external dependencies. The import system is declarative  
and explicit - you must import exactly what you use, and unused imports  
result in compilation errors. This design prevents code bloat and makes  
dependencies clear and manageable. Go supports various import styles  
including regular imports, aliased imports, and blank imports for  
side effects.

Package visibility in Go is controlled by naming conventions rather than  
explicit access modifiers. Identifiers that begin with uppercase letters  
are exported (public) and can be accessed from other packages, while those  
beginning with lowercase letters are unexported (private) and only accessible  
within the same package. This simple yet effective system provides clear  
encapsulation boundaries and encourages well-designed APIs.

The Go standard library is organized into packages that cover everything from  
basic I/O operations to cryptographic functions, networking, and web services.  
Understanding how to effectively use and organize packages is crucial for  
writing maintainable, scalable Go applications. Modern Go development also  
involves working with modules, which provide versioned dependency management  
and enable reliable, reproducible builds across different environments.


## Basic package declaration

Every Go source file must begin with a package declaration that specifies  
which package the file belongs to.  

```go
// calculator.go
package calculator

import "fmt"

// Add performs addition of two integers
func Add(a, b int) int {
    return a + b
}

// Subtract performs subtraction of two integers
func Subtract(a, b int) int {
    return a - b
}

// PrintResult displays the calculation result
func PrintResult(operation string, a, b, result int) {
    fmt.Printf("%s: %d and %d = %d\n", operation, a, b, result)
}
```

The package declaration tells the Go compiler which package this file belongs  
to. All files in the same directory must declare the same package name.  
Functions starting with uppercase letters are exported and can be used by  
other packages.  

## Import statements

Import statements allow you to use code from other packages in your program.  

```go
package main

import "fmt"
import "math"
import "time"

func main() {
    // Using fmt package
    fmt.Println("Current calculations:")
    
    // Using math package
    result := math.Sqrt(16)
    fmt.Printf("Square root of 16: %.2f\n", result)
    
    // Using time package
    now := time.Now()
    fmt.Printf("Current time: %s\n", now.Format("2006-01-02 15:04:05"))
    
    // Using math constants
    fmt.Printf("Value of Pi: %.6f\n", math.Pi)
    fmt.Printf("Value of E: %.6f\n", math.E)
}
```

Each import statement specifies a single package to import. The imported  
package can then be used by referencing its name followed by a dot and  
the identifier you want to access from that package.  

## Multiple imports

You can group multiple import statements together using parentheses for  
better organization and readability.  

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strconv"
    "strings"
)

func main() {
    reader := bufio.NewReader(os.Stdin)
    
    fmt.Print("Enter your name: ")
    name, _ := reader.ReadString('\n')
    name = strings.TrimSpace(name)
    
    fmt.Print("Enter your age: ")
    ageStr, _ := reader.ReadString('\n')
    ageStr = strings.TrimSpace(ageStr)
    
    age, err := strconv.Atoi(ageStr)
    if err != nil {
        fmt.Println("Invalid age entered")
        return
    }
    
    fmt.Printf("Hello there %s, you are %d years old\n", name, age)
    
    if age >= 18 {
        fmt.Println("You are an adult")
    } else {
        fmt.Println("You are a minor")
    }
}
```

Grouping imports in parentheses is the preferred Go style. The imports are  
automatically sorted alphabetically by most Go formatters. This example  
shows how different packages work together for user input processing.  

## Package aliases

Package aliases allow you to rename imported packages to avoid naming  
conflicts or provide shorter names.  

```go
package main

import (
    "fmt"
    crand "crypto/rand"
    mrand "math/rand"
    "time"
)

func main() {
    // Using math/rand with alias
    mrand.Seed(time.Now().UnixNano())
    fmt.Printf("Math random number: %d\n", mrand.Intn(100))
    
    // Using crypto/rand with alias
    bytes := make([]byte, 4)
    crand.Read(bytes)
    fmt.Printf("Crypto random bytes: %v\n", bytes)
    
    // Convert crypto random bytes to integer
    cryptoNum := int(bytes[0])
    fmt.Printf("Crypto random as int: %d\n", cryptoNum)
    
    // Generate multiple random values
    fmt.Println("Multiple math random values:")
    for i := 0; i < 5; i++ {
        fmt.Printf("  %d: %d\n", i+1, mrand.Intn(50))
    }
}
```

Package aliases are especially useful when you need to import packages with  
the same name but from different paths, such as crypto/rand and math/rand.  
The alias becomes the new name used to reference the package.  

## Dot imports

Dot imports allow you to access exported identifiers from a package without  
using the package name as a qualifier.  

```go
package main

import (
    "fmt"
    . "math"
    . "strings"
)

func main() {
    // Using math functions without package name
    radius := 5.0
    area := Pi * Pow(radius, 2)
    fmt.Printf("Circle area with radius %.1f: %.2f\n", radius, area)
    
    // Using strings functions without package name
    text := "Go Programming Language"
    fmt.Printf("Original: %s\n", text)
    fmt.Printf("Uppercase: %s\n", ToUpper(text))
    fmt.Printf("Lowercase: %s\n", ToLower(text))
    fmt.Printf("Title case: %s\n", ToTitle(text))
    
    // Using more math functions
    angle := 45.0
    radians := angle * Pi / 180
    fmt.Printf("Sin(%.0f°): %.4f\n", angle, Sin(radians))
    fmt.Printf("Cos(%.0f°): %.4f\n", angle, Cos(radians))
    fmt.Printf("Tan(%.0f°): %.4f\n", angle, Tan(radians))
}
```

Dot imports should be used sparingly as they can make code less readable  
by obscuring where identifiers come from. They're mainly useful in tests  
and when working with packages designed for this usage pattern.  

## fmt package usage

The fmt package provides formatted I/O functions similar to C's printf  
and scanf, essential for most Go programs.  

```go
package main

import "fmt"

type Product struct {
    Name     string
    Price    float64
    Quantity int
}

func main() {
    product := Product{
        Name:     "Laptop Computer",
        Price:    1299.99,
        Quantity: 5,
    }
    
    // Different printing functions
    fmt.Print("Product: ")
    fmt.Println(product.Name)
    fmt.Printf("Price: $%.2f\n", product.Price)
    
    // Formatting with different verbs
    fmt.Printf("Product details:\n")
    fmt.Printf("  Name: %s\n", product.Name)
    fmt.Printf("  Price: $%v\n", product.Price)
    fmt.Printf("  Quantity: %d\n", product.Quantity)
    fmt.Printf("  In stock: %t\n", product.Quantity > 0)
    
    // String formatting
    description := fmt.Sprintf("%s costs $%.2f (qty: %d)", 
        product.Name, product.Price, product.Quantity)
    fmt.Println("Description:", description)
    
    // Different number formats
    fmt.Printf("Price in different formats:\n")
    fmt.Printf("  Decimal: %.2f\n", product.Price)
    fmt.Printf("  Scientific: %.2e\n", product.Price)
    fmt.Printf("  Hexadecimal: %x\n", int(product.Price))
}
```

The fmt package is one of the most commonly used packages in Go. It provides  
various formatting verbs for different data types and output styles. The  
Printf family of functions offers precise control over output formatting.  

## strings package

The strings package provides utilities for manipulating UTF-8 encoded  
strings with various operations.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "  Go Programming Language  "
    
    // Basic string operations
    fmt.Printf("Original: '%s'\n", text)
    fmt.Printf("Trimmed: '%s'\n", strings.TrimSpace(text))
    fmt.Printf("Uppercase: %s\n", strings.ToUpper(text))
    fmt.Printf("Lowercase: %s\n", strings.ToLower(text))
    
    // String searching and checking
    target := "Programming"
    fmt.Printf("Contains '%s': %t\n", target, strings.Contains(text, target))
    fmt.Printf("Starts with '  Go': %t\n", strings.HasPrefix(text, "  Go"))
    fmt.Printf("Ends with 'age  ': %t\n", strings.HasSuffix(text, "age  "))
    fmt.Printf("Index of '%s': %d\n", target, strings.Index(text, target))
    
    // String splitting and joining
    sentence := "apple,banana,cherry,date"
    fruits := strings.Split(sentence, ",")
    fmt.Printf("Split fruits: %v\n", fruits)
    
    joined := strings.Join(fruits, " | ")
    fmt.Printf("Joined: %s\n", joined)
    
    // String replacement
    original := "I like apples and apples are great"
    replaced := strings.Replace(original, "apples", "oranges", 1)
    fmt.Printf("Replace once: %s\n", replaced)
    
    replacedAll := strings.ReplaceAll(original, "apples", "oranges")
    fmt.Printf("Replace all: %s\n", replacedAll)
    
    // String repetition and building
    repeated := strings.Repeat("Go! ", 3)
    fmt.Printf("Repeated: %s\n", repeated)
}
```

The strings package is essential for text processing in Go. It provides  
efficient implementations of common string operations and is designed to  
work seamlessly with UTF-8 encoded text data.  

## math package

The math package provides basic constants and mathematical functions for  
floating-point operations.  

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    // Mathematical constants
    fmt.Printf("Mathematical constants:\n")
    fmt.Printf("Pi: %.6f\n", math.Pi)
    fmt.Printf("E: %.6f\n", math.E)
    fmt.Printf("Phi: %.6f\n", math.Phi)
    fmt.Printf("Sqrt(2): %.6f\n", math.Sqrt2)
    
    // Basic arithmetic functions
    x, y := 16.0, 3.0
    fmt.Printf("\nBasic operations on %.1f and %.1f:\n", x, y)
    fmt.Printf("Power: %.2f\n", math.Pow(x, y))
    fmt.Printf("Square root: %.2f\n", math.Sqrt(x))
    fmt.Printf("Absolute value: %.2f\n", math.Abs(-x))
    fmt.Printf("Ceiling: %.0f\n", math.Ceil(x/y))
    fmt.Printf("Floor: %.0f\n", math.Floor(x/y))
    fmt.Printf("Round: %.0f\n", math.Round(x/y))
    
    // Trigonometric functions (in radians)
    angle := 45.0 * math.Pi / 180 // Convert degrees to radians
    fmt.Printf("\nTrigonometric functions for 45°:\n")
    fmt.Printf("Sin: %.4f\n", math.Sin(angle))
    fmt.Printf("Cos: %.4f\n", math.Cos(angle))
    fmt.Printf("Tan: %.4f\n", math.Tan(angle))
    
    // Logarithmic functions
    value := 100.0
    fmt.Printf("\nLogarithmic functions for %.1f:\n", value)
    fmt.Printf("Natural log: %.4f\n", math.Log(value))
    fmt.Printf("Base 10 log: %.4f\n", math.Log10(value))
    fmt.Printf("Base 2 log: %.4f\n", math.Log2(value))
    
    // Min and Max
    a, b := 15.7, 23.2
    fmt.Printf("\nMin/Max of %.1f and %.1f:\n", a, b)
    fmt.Printf("Minimum: %.1f\n", math.Min(a, b))
    fmt.Printf("Maximum: %.1f\n", math.Max(a, b))
}
```

The math package provides comprehensive mathematical functions for scientific  
computing and general mathematical operations. All functions work with  
float64 values and follow IEEE 754 standards for floating-point arithmetic.  

## time package

The time package provides functionality for measuring and displaying time,  
including parsing and formatting timestamps.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Current time
    now := time.Now()
    fmt.Printf("Current time: %s\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("Unix timestamp: %d\n", now.Unix())
    
    // Different time formats
    fmt.Printf("Different formats:\n")
    fmt.Printf("  RFC3339: %s\n", now.Format(time.RFC3339))
    fmt.Printf("  Kitchen: %s\n", now.Format(time.Kitchen))
    fmt.Printf("  Stamp: %s\n", now.Format(time.Stamp))
    fmt.Printf("  Custom: %s\n", now.Format("Jan 2, 2006 at 3:04 PM"))
    
    // Creating specific times
    birthday := time.Date(1990, time.May, 15, 14, 30, 0, 0, time.UTC)
    fmt.Printf("Birthday: %s\n", birthday.Format("January 2, 2006"))
    
    // Time calculations
    duration := now.Sub(birthday)
    fmt.Printf("Age: %.0f days\n", duration.Hours()/24)
    
    // Adding and subtracting time
    future := now.Add(24 * time.Hour)
    fmt.Printf("Tomorrow: %s\n", future.Format("2006-01-02 15:04:05"))
    
    past := now.Add(-7 * 24 * time.Hour)
    fmt.Printf("Week ago: %s\n", past.Format("2006-01-02 15:04:05"))
    
    // Duration parsing and formatting
    duration1, _ := time.ParseDuration("2h30m15s")
    fmt.Printf("Parsed duration: %v\n", duration1)
    fmt.Printf("Duration in minutes: %.2f\n", duration1.Minutes())
    
    // Timer and sleeping
    fmt.Println("Waiting for 2 seconds...")
    time.Sleep(2 * time.Second)
    fmt.Println("Done waiting!")
    
    // Time zones
    loc, _ := time.LoadLocation("America/New_York")
    nyTime := now.In(loc)
    fmt.Printf("Time in New York: %s\n", nyTime.Format("2006-01-02 15:04:05 MST"))
}
```

The time package is essential for any application that needs to work with  
dates, times, durations, or scheduling. It provides comprehensive support  
for time zones, parsing, formatting, and time arithmetic operations.  

## os package

The os package provides a platform-independent interface to operating  
system functionality including file operations and environment variables.  

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    // Environment variables
    fmt.Println("Environment information:")
    fmt.Printf("HOME directory: %s\n", os.Getenv("HOME"))
    fmt.Printf("PATH: %s\n", os.Getenv("PATH")[:50]+"...")
    
    // Set and get custom environment variable
    os.Setenv("MY_APP_MODE", "development")
    fmt.Printf("Custom env var: %s\n", os.Getenv("MY_APP_MODE"))
    
    // Command line arguments
    fmt.Printf("\nCommand line arguments:\n")
    fmt.Printf("Program name: %s\n", os.Args[0])
    fmt.Printf("Number of arguments: %d\n", len(os.Args))
    
    for i, arg := range os.Args {
        fmt.Printf("  Arg %d: %s\n", i, arg)
    }
    
    // Working directory
    wd, err := os.Getwd()
    if err != nil {
        fmt.Printf("Error getting working directory: %v\n", err)
    } else {
        fmt.Printf("\nCurrent working directory: %s\n", wd)
    }
    
    // User information
    hostname, _ := os.Hostname()
    fmt.Printf("Hostname: %s\n", hostname)
    
    // Process information
    fmt.Printf("Process ID: %d\n", os.Getpid())
    fmt.Printf("Parent Process ID: %d\n", os.Getppid())
    
    // File operations (creating and checking)
    filename := "test_file.txt"
    
    // Create a temporary file
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
    } else {
        fmt.Printf("Created file: %s\n", filename)
        file.WriteString("This is a test file created by Go\n")
        file.Close()
        
        // Check if file exists
        if _, err := os.Stat(filename); err == nil {
            fmt.Printf("File %s exists\n", filename)
        }
        
        // Clean up
        os.Remove(filename)
        fmt.Printf("Cleaned up file: %s\n", filename)
    }
}
```

The os package provides essential functionality for interacting with the  
operating system. It abstracts platform-specific operations and provides  
a consistent interface for file operations, process management, and system  
information retrieval.  

## Multiple files in same package

A single package can span multiple files in the same directory, allowing  
for better code organization and separation of concerns.  

```go
// types.go
package calculator

type Operation struct {
    Name   string
    Symbol string
    Func   func(float64, float64) float64
}

type Calculator struct {
    operations map[string]Operation
    history    []string
}

// operations.go  
package calculator

import "fmt"

func (c *Calculator) AddOperation(name, symbol string, fn func(float64, float64) float64) {
    if c.operations == nil {
        c.operations = make(map[string]Operation)
    }
    c.operations[name] = Operation{Name: name, Symbol: symbol, Func: fn}
}

func (c *Calculator) Calculate(operation string, a, b float64) (float64, error) {
    op, exists := c.operations[operation]
    if !exists {
        return 0, fmt.Errorf("operation %s not found", operation)
    }
    
    result := op.Func(a, b)
    c.addToHistory(fmt.Sprintf("%.2f %s %.2f = %.2f", a, op.Symbol, b, result))
    return result, nil
}

func (c *Calculator) addToHistory(entry string) {
    c.history = append(c.history, entry)
}

// history.go
package calculator

import "fmt"

func (c *Calculator) GetHistory() []string {
    return c.history
}

func (c *Calculator) ClearHistory() {
    c.history = nil
}

func (c *Calculator) PrintHistory() {
    fmt.Println("Calculator History:")
    for i, entry := range c.history {
        fmt.Printf("%d: %s\n", i+1, entry)
    }
}

// main.go
package main

import (
    "fmt"
    "your-module/calculator"
)

func main() {
    calc := &calculator.Calculator{}
    
    // Add basic operations
    calc.AddOperation("add", "+", func(a, b float64) float64 { return a + b })
    calc.AddOperation("subtract", "-", func(a, b float64) float64 { return a - b })
    calc.AddOperation("multiply", "*", func(a, b float64) float64 { return a * b })
    calc.AddOperation("divide", "/", func(a, b float64) float64 { return a / b })
    
    // Perform calculations
    result1, _ := calc.Calculate("add", 10, 5)
    fmt.Printf("10 + 5 = %.2f\n", result1)
    
    result2, _ := calc.Calculate("multiply", 3, 7)
    fmt.Printf("3 * 7 = %.2f\n", result2)
    
    // Show history
    calc.PrintHistory()
}
```

When multiple files belong to the same package, they share the same namespace  
and can access each other's unexported identifiers. This allows for logical  
separation of functionality while maintaining cohesive package behavior.  

## Package initialization

Packages can have initialization functions that run automatically when  
the package is first imported.  

```go
// config.go
package config

import (
    "fmt"
    "os"
)

var (
    AppName    string
    Version    string
    Debug      bool
    ConfigFile string
)

func init() {
    fmt.Println("Initializing config package...")
    
    // Set default values
    AppName = "MyApplication"
    Version = "1.0.0"
    Debug = false
    ConfigFile = "app.conf"
    
    // Override with environment variables if present
    if name := os.Getenv("APP_NAME"); name != "" {
        AppName = name
    }
    
    if version := os.Getenv("APP_VERSION"); version != "" {
        Version = version
    }
    
    if debug := os.Getenv("DEBUG"); debug == "true" {
        Debug = true
    }
    
    fmt.Printf("Config initialized: %s v%s (debug=%t)\n", AppName, Version, Debug)
}

func PrintConfig() {
    fmt.Printf("Application Configuration:\n")
    fmt.Printf("  Name: %s\n", AppName)
    fmt.Printf("  Version: %s\n", Version)
    fmt.Printf("  Debug Mode: %t\n", Debug)
    fmt.Printf("  Config File: %s\n", ConfigFile)
}

// database.go
package database

import (
    "fmt"
    "log"
)

type Connection struct {
    Host string
    Port int
    Name string
}

var defaultConnection *Connection

func init() {
    fmt.Println("Initializing database package...")
    
    // Initialize default connection
    defaultConnection = &Connection{
        Host: "localhost",
        Port: 5432,
        Name: "myapp",
    }
    
    // Simulate connection setup
    fmt.Printf("Default database connection configured: %s:%d/%s\n", 
        defaultConnection.Host, defaultConnection.Port, defaultConnection.Name)
}

func GetConnection() *Connection {
    if defaultConnection == nil {
        log.Fatal("Database not initialized")
    }
    return defaultConnection
}

// main.go
package main

import (
    "fmt"
    "your-module/config"
    "your-module/database"
)

func main() {
    fmt.Println("Starting application...")
    
    // Package init functions have already run
    config.PrintConfig()
    
    // Use initialized database connection
    db := database.GetConnection()
    fmt.Printf("Using database: %s:%d/%s\n", db.Host, db.Port, db.Name)
    
    fmt.Println("Application started successfully!")
}
```

The init function is called automatically when a package is imported. If  
a package has multiple files with init functions, they execute in the  
order the files are presented to the compiler. Init functions are useful  
for one-time setup tasks and package configuration.  

## Init function usage

Init functions can be used for complex initialization logic, registering  
handlers, and setting up package-level state.  

```go
// registry.go
package registry

import (
    "fmt"
    "sort"
)

type Handler func(string) string

var (
    handlers map[string]Handler
    plugins  []string
)

func init() {
    fmt.Println("Registry initialization phase 1: Creating maps")
    handlers = make(map[string]Handler)
    plugins = make([]string, 0)
}

func init() {
    fmt.Println("Registry initialization phase 2: Registering default handlers")
    
    // Register built-in handlers
    RegisterHandler("uppercase", func(input string) string {
        return fmt.Sprintf("UPPER: %s", input)
    })
    
    RegisterHandler("lowercase", func(input string) string {
        return fmt.Sprintf("lower: %s", input)
    })
    
    RegisterHandler("reverse", func(input string) string {
        runes := []rune(input)
        for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
            runes[i], runes[j] = runes[j], runes[i]
        }
        return fmt.Sprintf("REVERSED: %s", string(runes))
    })
}

func init() {
    fmt.Println("Registry initialization phase 3: Setup complete")
    fmt.Printf("Registered %d handlers\n", len(handlers))
}

func RegisterHandler(name string, handler Handler) {
    handlers[name] = handler
    plugins = append(plugins, name)
    sort.Strings(plugins)
    fmt.Printf("Registered handler: %s\n", name)
}

func Execute(handlerName, input string) (string, error) {
    handler, exists := handlers[handlerName]
    if !exists {
        return "", fmt.Errorf("handler '%s' not found", handlerName)
    }
    return handler(input), nil
}

func ListHandlers() []string {
    return plugins
}

// logger.go
package logger

import (
    "fmt"
    "os"
    "time"
)

type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
)

var (
    logFile   *os.File
    logLevel  LogLevel
    logFormat string
)

func init() {
    fmt.Println("Logger initialization: Setting up logging")
    
    // Set default log level
    logLevel = INFO
    logFormat = "2006-01-02 15:04:05"
    
    // Try to create log file
    var err error
    logFile, err = os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        fmt.Printf("Warning: Could not open log file: %v\n", err)
        logFile = nil
    } else {
        fmt.Println("Log file opened successfully")
    }
    
    // Log initialization
    Log(INFO, "Logger initialized successfully")
}

func Log(level LogLevel, message string) {
    if level < logLevel {
        return
    }
    
    levelStr := []string{"DEBUG", "INFO", "WARN", "ERROR"}[level]
    logEntry := fmt.Sprintf("[%s] %s: %s\n", 
        time.Now().Format(logFormat), levelStr, message)
    
    // Write to console
    fmt.Print(logEntry)
    
    // Write to file if available
    if logFile != nil {
        logFile.WriteString(logEntry)
    }
}

func Close() {
    if logFile != nil {
        Log(INFO, "Closing log file")
        logFile.Close()
    }
}

// main.go
package main

import (
    "fmt"
    "your-module/logger"
    "your-module/registry"
)

func main() {
    fmt.Println("\n--- Application Starting ---")
    
    // Use initialized registry
    handlers := registry.ListHandlers()
    fmt.Printf("Available handlers: %v\n", handlers)
    
    // Test handlers
    testInput := "Hello there"
    for _, handlerName := range handlers {
        result, err := registry.Execute(handlerName, testInput)
        if err != nil {
            logger.Log(logger.ERROR, fmt.Sprintf("Handler error: %v", err))
        } else {
            logger.Log(logger.INFO, fmt.Sprintf("Handler '%s' result: %s", handlerName, result))
        }
    }
    
    // Register a custom handler
    registry.RegisterHandler("prefix", func(input string) string {
        return fmt.Sprintf("PREFIX: %s", input)
    })
    
    // Test new handler
    result, _ := registry.Execute("prefix", testInput)
    logger.Log(logger.INFO, fmt.Sprintf("Custom handler result: %s", result))
    
    // Cleanup
    logger.Close()
}
```

Multiple init functions in the same package execute in the order they appear  
in the source code. This allows for complex initialization sequences where  
later init functions can depend on earlier ones completing successfully.  

## Package visibility (exported/unexported)

Go uses naming conventions to determine visibility - identifiers starting  
with uppercase letters are exported (public), while lowercase are unexported  
(private).  

```go
// mathutils.go
package mathutils

import "fmt"

// Exported constants (public)
const (
    Pi     = 3.14159265359
    E      = 2.71828182846
    Golden = 1.61803398875
)

// Unexported constants (private)
const (
    precision = 10
    maxValue  = 1000000
)

// Exported variable (public)
var Debug bool

// Unexported variables (private)
var (
    calculations int
    lastResult   float64
)

// Exported struct (public)
type Calculator struct {
    Name    string
    Version string
    // unexported field (private)
    precision int
}

// unexported struct (private)  
type result struct {
    value float64
    error string
}

// Exported function (public)
func Add(a, b float64) float64 {
    incrementCalculations()
    result := a + b
    lastResult = result
    if Debug {
        fmt.Printf("Add(%.2f, %.2f) = %.2f\n", a, b, result)
    }
    return result
}

// Exported function (public)
func NewCalculator(name string) *Calculator {
    return &Calculator{
        Name:      name,
        Version:   "1.0",
        precision: precision, // using unexported constant
    }
}

// Exported method (public)
func (c *Calculator) GetInfo() string {
    return fmt.Sprintf("%s v%s (precision: %d)", c.Name, c.Version, c.precision)
}

// unexported function (private)
func incrementCalculations() {
    calculations++
}

// unexported method (private)
func (c *Calculator) validate() bool {
    return c.Name != ""
}

// Exported function that uses unexported functionality
func GetStats() (int, float64) {
    return calculations, lastResult
}

// main.go
package main

import (
    "fmt"
    "your-module/mathutils"
)

func main() {
    // Can access exported identifiers
    mathutils.Debug = true
    
    // Can use exported constants
    fmt.Printf("Pi: %.6f\n", mathutils.Pi)
    fmt.Printf("E: %.6f\n", mathutils.E)
    
    // Can call exported functions
    sum := mathutils.Add(10.5, 20.3)
    fmt.Printf("Sum: %.2f\n", sum)
    
    // Can create exported structs
    calc := mathutils.NewCalculator("Scientific Calculator")
    fmt.Println(calc.GetInfo())
    
    // Can access exported fields
    calc.Name = "Advanced Calculator"
    calc.Version = "2.0"
    
    // Cannot access unexported identifiers (would cause compile errors):
    // fmt.Println(mathutils.precision)      // Error: cannot access
    // mathutils.incrementCalculations()     // Error: cannot access  
    // calc.precision = 15                   // Error: cannot access
    // calc.validate()                       // Error: cannot access
    
    // Can use exported functions that internally use unexported functionality
    count, last := mathutils.GetStats()
    fmt.Printf("Calculations: %d, Last result: %.2f\n", count, last)
}
```

The visibility system in Go provides clear encapsulation boundaries. Exported  
identifiers form the public API of a package, while unexported ones remain  
implementation details that can change without affecting package users.  

## Package documentation

Package documentation is created using comments that appear before the  
package declaration and exported identifiers.  

```go
// Package textprocessor provides utilities for advanced text processing and analysis.
//
// This package offers a comprehensive set of functions for text manipulation,
// including word counting, text transformation, and pattern matching.
// It is designed to work efficiently with UTF-8 encoded text.
//
// Basic usage:
//
//     processor := textprocessor.New()
//     result := processor.Process("Hello there, world!")
//     fmt.Println(result.WordCount)
//
// For more advanced features, see the TextAnalyzer type and its methods.
package textprocessor

import (
    "fmt"
    "regexp"
    "strings"
    "unicode"
)

// TextProcessor represents a text processing engine with configurable options.
// It maintains internal state for processing operations and provides
// methods for various text analysis tasks.
type TextProcessor struct {
    CaseSensitive bool
    IgnoreSpaces  bool
    MaxLength     int
}

// Result holds the results of text processing operations.
// It provides detailed information about the processed text
// including statistics and transformed content.
type Result struct {
    Original    string
    Processed   string
    WordCount   int
    CharCount   int
    LineCount   int
    Errors      []string
}

// New creates a new TextProcessor with default settings.
// The processor is configured with case-sensitive processing
// enabled and no length restrictions.
//
// Example:
//     processor := New()
//     processor.CaseSensitive = false
func New() *TextProcessor {
    return &TextProcessor{
        CaseSensitive: true,
        IgnoreSpaces:  false,
        MaxLength:     10000,
    }
}

// Process analyzes the given text and returns detailed results.
// It performs word counting, character analysis, and applies
// any configured transformations.
//
// The input text should be valid UTF-8. Processing will stop
// if the text exceeds MaxLength characters.
//
// Example:
//     processor := New()
//     result := processor.Process("Hello there!")
//     fmt.Printf("Words: %d\n", result.WordCount)
func (tp *TextProcessor) Process(text string) *Result {
    if len(text) > tp.MaxLength {
        return &Result{
            Original: text,
            Errors:   []string{fmt.Sprintf("text exceeds maximum length of %d", tp.MaxLength)},
        }
    }
    
    processed := text
    if !tp.CaseSensitive {
        processed = strings.ToLower(processed)
    }
    
    if tp.IgnoreSpaces {
        processed = strings.ReplaceAll(processed, " ", "")
    }
    
    return &Result{
        Original:  text,
        Processed: processed,
        WordCount: countWords(text),
        CharCount: len([]rune(text)),
        LineCount: strings.Count(text, "\n") + 1,
    }
}

// Transform applies various text transformations based on the specified mode.
// Supported modes include "upper", "lower", "title", and "reverse".
//
// Returns an error if the mode is not supported or if the text
// cannot be processed.
func (tp *TextProcessor) Transform(text, mode string) (string, error) {
    switch mode {
    case "upper":
        return strings.ToUpper(text), nil
    case "lower":
        return strings.ToLower(text), nil
    case "title":
        return strings.ToTitle(text), nil
    case "reverse":
        runes := []rune(text)
        for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
            runes[i], runes[j] = runes[j], runes[i]
        }
        return string(runes), nil
    default:
        return "", fmt.Errorf("unsupported transformation mode: %s", mode)
    }
}

// countWords counts the number of words in the given text.
// Words are separated by whitespace and punctuation.
func countWords(text string) int {
    if text == "" {
        return 0
    }
    
    // Use regex to split on word boundaries
    re := regexp.MustCompile(`\s+`)
    words := re.Split(strings.TrimSpace(text), -1)
    
    // Filter out empty strings
    count := 0
    for _, word := range words {
        if word != "" {
            count++
        }
    }
    
    return count
}

// IsAlpha checks if the given text contains only alphabetic characters.
// It returns true if all runes in the text are letters, false otherwise.
//
// Empty strings return false.
func IsAlpha(text string) bool {
    if text == "" {
        return false
    }
    
    for _, r := range text {
        if !unicode.IsLetter(r) {
            return false
        }
    }
    return true
}

// WordFrequency analyzes text and returns word frequency counts.
// The analysis is case-insensitive and excludes common stop words.
//
// Returns a map where keys are words and values are occurrence counts.
func WordFrequency(text string) map[string]int {
    frequency := make(map[string]int)
    words := strings.FieldsFunc(strings.ToLower(text), func(c rune) bool {
        return !unicode.IsLetter(c)
    })
    
    stopWords := map[string]bool{
        "the": true, "a": true, "an": true, "and": true, "or": true,
        "but": true, "in": true, "on": true, "at": true, "to": true,
        "for": true, "of": true, "with": true, "by": true,
    }
    
    for _, word := range words {
        if len(word) > 0 && !stopWords[word] {
            frequency[word]++
        }
    }
    
    return frequency
}
```

Good package documentation includes a package comment, detailed function and  
type comments, and usage examples. The go doc tool extracts this documentation  
automatically. Comments should explain the purpose, behavior, and usage  
patterns of exported identifiers.  

## Blank identifier imports

Blank identifier imports are used to import packages for their side effects  
without directly using their exported identifiers.  

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    _ "net/http/pprof"  // Import for side effects only
    "os"
    "time"
    
    _ "image/jpeg"  // Registers JPEG decoder
    _ "image/png"   // Registers PNG decoder
)

func main() {
    // Start HTTP server with pprof endpoints automatically available
    // The pprof package registers its handlers in the default ServeMux
    go func() {
        log.Println("Starting profiling server on :6060")
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // Create a simple HTTP server
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Server started at %s\n", time.Now().Format(time.RFC3339))
        fmt.Fprintf(w, "Process ID: %d\n", os.Getpid())
        
        // Simulate some work
        start := time.Now()
        time.Sleep(100 * time.Millisecond)
        fmt.Fprintf(w, "Request processed in %v\n", time.Since(start))
    })
    
    http.HandleFunc("/status", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Status: OK\n")
        fmt.Fprintf(w, "Time: %s\n", time.Now().Format(time.Kitchen))
    })
    
    fmt.Println("Server starting on :8080")
    fmt.Println("Profiling available at http://localhost:6060/debug/pprof/")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// Example with image processing that benefits from blank imports
// imageprocessor.go
package main

import (
    "fmt"
    "image"
    _ "image/gif"   // Import for GIF support
    _ "image/jpeg"  // Import for JPEG support  
    _ "image/png"   // Import for PNG support
    "os"
)

func processImage(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("failed to open file: %w", err)
    }
    defer file.Close()
    
    // image.Decode automatically uses the appropriate decoder
    // based on the registered formats from the blank imports
    img, format, err := image.Decode(file)
    if err != nil {
        return fmt.Errorf("failed to decode image: %w", err)
    }
    
    bounds := img.Bounds()
    fmt.Printf("Image format: %s\n", format)
    fmt.Printf("Dimensions: %dx%d\n", bounds.Dx(), bounds.Dy())
    fmt.Printf("Color model: %T\n", img.ColorModel())
    
    return nil
}

func main() {
    // The blank imports enable support for multiple image formats
    fmt.Println("Supported image formats through blank imports:")
    
    // This would work with JPEG, PNG, or GIF files
    testFiles := []string{"test.jpg", "test.png", "test.gif"}
    
    for _, file := range testFiles {
        fmt.Printf("Attempting to process: %s\n", file)
        if err := processImage(file); err != nil {
            fmt.Printf("Error: %v\n", err)
        }
        fmt.Println()
    }
}
```

Blank imports execute the init functions of imported packages without making  
their exported identifiers available. This is commonly used for registering  
drivers, decoders, or handlers that integrate with larger systems through  
package-level registration mechanisms.  

## Conditional imports with build tags

Build tags allow you to conditionally include packages based on build  
constraints, enabling platform-specific or feature-specific code.  

```go
// logger_dev.go
//go:build dev
// +build dev

package logger

import (
    "fmt"
    "log"
    "os"
)

var debugMode = true

func init() {
    log.SetFlags(log.LstdFlags | log.Lshortfile)
    log.SetOutput(os.Stdout)
    fmt.Println("Development logger initialized with debug mode")
}

func Debug(msg string, args ...interface{}) {
    if debugMode {
        log.Printf("[DEBUG] "+msg, args...)
    }
}

func Info(msg string, args ...interface{}) {
    log.Printf("[INFO] "+msg, args...)
}

// logger_prod.go
//go:build prod
// +build prod

package logger

import (
    "io/ioutil"
    "log"
    "os"
)

var debugMode = false

func init() {
    // Production logger configuration
    file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalln("Failed to open log file:", err)
    }
    log.SetOutput(file)
    log.SetFlags(log.LstdFlags)
}

func Debug(msg string, args ...interface{}) {
    // Debug messages are silently discarded in production
}

func Info(msg string, args ...interface{}) {
    log.Printf("[INFO] "+msg, args...)
}

// database_sqlite.go
//go:build sqlite
// +build sqlite

package database

import (
    "database/sql"
    _ "github.com/mattn/go-sqlite3"  // SQLite driver
)

type Config struct {
    DataSourceName string
}

func Connect(config Config) (*sql.DB, error) {
    return sql.Open("sqlite3", config.DataSourceName)
}

func DefaultConfig() Config {
    return Config{
        DataSourceName: "app.db",
    }
}

// database_postgres.go
//go:build postgres
// +build postgres

package database

import (
    "database/sql"
    _ "github.com/lib/pq"  // PostgreSQL driver
)

type Config struct {
    Host     string
    Port     int
    User     string
    Password string
    DBName   string
}

func Connect(config Config) (*sql.DB, error) {
    dsn := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=disable",
        config.Host, config.Port, config.User, config.Password, config.DBName)
    return sql.Open("postgres", dsn)
}

func DefaultConfig() Config {
    return Config{
        Host:     "localhost",
        Port:     5432,
        User:     "postgres",
        Password: "password",
        DBName:   "myapp",
    }
}

// main.go
package main

import (
    "fmt"
    "your-module/database"
    "your-module/logger"
)

func main() {
    logger.Info("Application starting...")
    logger.Debug("This is a debug message")
    
    // Database connection using build-tag specific implementation
    config := database.DefaultConfig()
    db, err := database.Connect(config)
    if err != nil {
        logger.Info("Database connection failed: %v", err)
        return
    }
    defer db.Close()
    
    logger.Info("Database connected successfully")
    logger.Debug("Using configuration: %+v", config)
    
    // Application logic here
    fmt.Println("Application running...")
}

/*
To build with different configurations:

Development with SQLite:
go build -tags "dev sqlite" -o myapp-dev

Production with PostgreSQL:
go build -tags "prod postgres" -o myapp-prod

Development with PostgreSQL:
go build -tags "dev postgres" -o myapp-dev-pg
*/
```

Build tags provide powerful conditional compilation capabilities. They enable  
the same codebase to support multiple environments, platforms, or feature  
sets while maintaining clean separation of concerns and avoiding runtime  
overhead for unused functionality.  

## Internal packages

Internal packages are a Go feature that restricts package imports to code  
within the same module or parent directory tree.  

```go
// Directory structure:
// myapp/
//   ├── main.go
//   ├── internal/
//   │   ├── auth/
//   │   │   └── auth.go
//   │   ├── config/
//   │   │   └── config.go
//   │   └── database/
//   │       └── db.go
//   ├── api/
//   │   └── handlers.go
//   └── cmd/
//       └── cli.go

// internal/auth/auth.go
package auth

import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "time"
)

type User struct {
    ID       int
    Username string
    Email    string
    Role     string
}

type Session struct {
    Token   string
    UserID  int
    Expires time.Time
}

var sessions = make(map[string]*Session)

func GenerateToken() string {
    bytes := make([]byte, 16)
    rand.Read(bytes)
    return hex.EncodeToString(bytes)
}

func CreateSession(user *User) *Session {
    session := &Session{
        Token:   GenerateToken(),
        UserID:  user.ID,
        Expires: time.Now().Add(24 * time.Hour),
    }
    sessions[session.Token] = session
    return session
}

func ValidateSession(token string) (*Session, error) {
    session, exists := sessions[token]
    if !exists {
        return nil, fmt.Errorf("session not found")
    }
    
    if time.Now().After(session.Expires) {
        delete(sessions, token)
        return nil, fmt.Errorf("session expired")
    }
    
    return session, nil
}

func RevokeSession(token string) {
    delete(sessions, token)
}

// internal/config/config.go
package config

import (
    "encoding/json"
    "fmt"
    "os"
)

type AppConfig struct {
    Port         int    `json:"port"`
    DatabaseURL  string `json:"database_url"`
    JWTSecret    string `json:"jwt_secret"`
    Environment  string `json:"environment"`
    LogLevel     string `json:"log_level"`
}

var globalConfig *AppConfig

func Load(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("failed to open config file: %w", err)
    }
    defer file.Close()
    
    decoder := json.NewDecoder(file)
    config := &AppConfig{}
    if err := decoder.Decode(config); err != nil {
        return fmt.Errorf("failed to decode config: %w", err)
    }
    
    // Set defaults
    if config.Port == 0 {
        config.Port = 8080
    }
    if config.Environment == "" {
        config.Environment = "development"
    }
    if config.LogLevel == "" {
        config.LogLevel = "info"
    }
    
    globalConfig = config
    return nil
}

func Get() *AppConfig {
    if globalConfig == nil {
        // Return default config if none loaded
        return &AppConfig{
            Port:        8080,
            Environment: "development",
            LogLevel:    "info",
        }
    }
    return globalConfig
}

func IsDevelopment() bool {
    return Get().Environment == "development"
}

func IsProduction() bool {
    return Get().Environment == "production"
}

// internal/database/db.go
package database

import (
    "database/sql"
    "fmt"
    "myapp/internal/config"
    
    _ "github.com/lib/pq"
)

type DB struct {
    conn *sql.DB
}

var instance *DB

func Initialize() error {
    cfg := config.Get()
    db, err := sql.Open("postgres", cfg.DatabaseURL)
    if err != nil {
        return fmt.Errorf("failed to connect to database: %w", err)
    }
    
    if err := db.Ping(); err != nil {
        return fmt.Errorf("failed to ping database: %w", err)
    }
    
    instance = &DB{conn: db}
    return nil
}

func GetInstance() *DB {
    if instance == nil {
        panic("database not initialized")
    }
    return instance
}

func (db *DB) Query(query string, args ...interface{}) (*sql.Rows, error) {
    return db.conn.Query(query, args...)
}

func (db *DB) Exec(query string, args ...interface{}) (sql.Result, error) {
    return db.conn.Exec(query, args...)
}

func (db *DB) Close() error {
    if db.conn != nil {
        return db.conn.Close()
    }
    return nil
}

// api/handlers.go
package api

import (
    "encoding/json"
    "fmt"
    "net/http"
    "myapp/internal/auth"  // Can import internal packages from parent
    "myapp/internal/config"
)

func LoginHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse login credentials
    var credentials struct {
        Username string `json:"username"`
        Password string `json:"password"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&credentials); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Simulate user authentication
    if credentials.Username == "admin" && credentials.Password == "password" {
        user := &auth.User{
            ID:       1,
            Username: credentials.Username,
            Email:    "admin@example.com",
            Role:     "admin",
        }
        
        session := auth.CreateSession(user)
        
        response := map[string]interface{}{
            "token":   session.Token,
            "expires": session.Expires,
            "user":    user,
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
    } else {
        http.Error(w, "Invalid credentials", http.StatusUnauthorized)
    }
}

func StatusHandler(w http.ResponseWriter, r *http.Request) {
    cfg := config.Get()
    status := map[string]interface{}{
        "status":      "ok",
        "environment": cfg.Environment,
        "port":        cfg.Port,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(status)
}

// main.go
package main

import (
    "fmt"
    "log"
    "net/http"
    "myapp/api"
    "myapp/internal/config"  // Can import internal packages
    "myapp/internal/database"
)

func main() {
    // Load configuration
    if err := config.Load("config.json"); err != nil {
        log.Printf("Warning: %v, using defaults", err)
    }
    
    cfg := config.Get()
    fmt.Printf("Starting server in %s mode on port %d\n", cfg.Environment, cfg.Port)
    
    // Initialize database
    if err := database.Initialize(); err != nil {
        log.Fatal("Database initialization failed:", err)
    }
    defer database.GetInstance().Close()
    
    // Setup routes
    http.HandleFunc("/login", api.LoginHandler)
    http.HandleFunc("/status", api.StatusHandler)
    
    // Start server
    addr := fmt.Sprintf(":%d", cfg.Port)
    log.Printf("Server listening on %s", addr)
    log.Fatal(http.ListenAndServe(addr, nil))
}
```

Internal packages are only importable by code within the same module or  
parent directory tree. This provides strong encapsulation for implementation  
details and prevents external dependencies on internal APIs. The "internal"  
directory name has special meaning to the Go toolchain.  

## Vendor directory usage

The vendor directory allows you to include dependencies directly in your  
project for reproducible builds and offline development.  

```go
// Project structure with vendoring:
// myproject/
//   ├── go.mod
//   ├── go.sum
//   ├── main.go
//   ├── vendor/
//   │   ├── modules.txt
//   │   ├── github.com/
//   │   │   ├── gorilla/
//   │   │   │   └── mux/
//   │   │   └── pkg/
//   │   │       └── errors/
//   │   └── gopkg.in/
//   │       └── yaml.v3/
//   └── internal/
//       └── config/
//           └── config.go

// go.mod
module myproject

go 1.21

require (
    github.com/gorilla/mux v1.8.0
    github.com/pkg/errors v0.9.1
    gopkg.in/yaml.v3 v3.0.1
)

// internal/config/config.go
package config

import (
    "fmt"
    "io/ioutil"
    "os"
    
    "github.com/pkg/errors"  // Vendored dependency
    "gopkg.in/yaml.v3"       // Vendored dependency
)

type ServerConfig struct {
    Host string `yaml:"host"`
    Port int    `yaml:"port"`
    TLS  struct {
        Enabled  bool   `yaml:"enabled"`
        CertFile string `yaml:"cert_file"`
        KeyFile  string `yaml:"key_file"`
    } `yaml:"tls"`
}

type DatabaseConfig struct {
    Driver   string `yaml:"driver"`
    Host     string `yaml:"host"`
    Port     int    `yaml:"port"`
    Name     string `yaml:"name"`
    User     string `yaml:"user"`
    Password string `yaml:"password"`
}

type Config struct {
    App      string         `yaml:"app_name"`
    Version  string         `yaml:"version"`
    Debug    bool           `yaml:"debug"`
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
    Features map[string]bool `yaml:"features"`
}

func LoadFromFile(filename string) (*Config, error) {
    if _, err := os.Stat(filename); os.IsNotExist(err) {
        return nil, errors.Wrap(err, "config file does not exist")
    }
    
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return nil, errors.Wrap(err, "failed to read config file")
    }
    
    var config Config
    if err := yaml.Unmarshal(data, &config); err != nil {
        return nil, errors.Wrap(err, "failed to parse YAML config")
    }
    
    // Set defaults
    if config.Server.Host == "" {
        config.Server.Host = "localhost"
    }
    if config.Server.Port == 0 {
        config.Server.Port = 8080
    }
    if config.Database.Driver == "" {
        config.Database.Driver = "postgres"
    }
    
    return &config, nil
}

func (c *Config) Validate() error {
    if c.App == "" {
        return errors.New("app_name is required")
    }
    if c.Version == "" {
        return errors.New("version is required")
    }
    if c.Server.Port < 1 || c.Server.Port > 65535 {
        return errors.New("server port must be between 1 and 65535")
    }
    if c.Server.TLS.Enabled {
        if c.Server.TLS.CertFile == "" {
            return errors.New("TLS cert_file is required when TLS is enabled")
        }
        if c.Server.TLS.KeyFile == "" {
            return errors.New("TLS key_file is required when TLS is enabled")
        }
    }
    return nil
}

func (c *Config) GetDatabaseURL() string {
    return fmt.Sprintf("%s://%s:%s@%s:%d/%s",
        c.Database.Driver,
        c.Database.User,
        c.Database.Password,
        c.Database.Host,
        c.Database.Port,
        c.Database.Name,
    )
}

// main.go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
    
    "github.com/gorilla/mux"      // Vendored dependency
    "github.com/pkg/errors"       // Vendored dependency
    "myproject/internal/config"
)

func main() {
    // Load configuration
    cfg, err := config.LoadFromFile("config.yaml")
    if err != nil {
        log.Fatal("Failed to load config:", err)
    }
    
    if err := cfg.Validate(); err != nil {
        log.Fatal("Invalid configuration:", err)
    }
    
    fmt.Printf("Starting %s v%s\n", cfg.App, cfg.Version)
    
    // Create router using vendored gorilla/mux
    router := mux.NewRouter()
    
    // API routes
    api := router.PathPrefix("/api/v1").Subrouter()
    api.HandleFunc("/health", healthHandler(cfg)).Methods("GET")
    api.HandleFunc("/config", configHandler(cfg)).Methods("GET")
    api.HandleFunc("/features", featuresHandler(cfg)).Methods("GET")
    
    // Static routes
    router.HandleFunc("/", indexHandler).Methods("GET")
    
    // Create HTTP server
    addr := fmt.Sprintf("%s:%d", cfg.Server.Host, cfg.Server.Port)
    server := &http.Server{
        Addr:         addr,
        Handler:      router,
        ReadTimeout:  30 * time.Second,
        WriteTimeout: 30 * time.Second,
        IdleTimeout:  60 * time.Second,
    }
    
    // Start server
    go func() {
        log.Printf("Server listening on %s", addr)
        if cfg.Server.TLS.Enabled {
            if err := server.ListenAndServeTLS(cfg.Server.TLS.CertFile, cfg.Server.TLS.KeyFile); err != nil && err != http.ErrServerClosed {
                log.Fatal("Server failed:", err)
            }
        } else {
            if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
                log.Fatal("Server failed:", err)
            }
        }
    }()
    
    // Wait for interrupt signal to gracefully shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    log.Println("Shutting down server...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Fatal("Server forced to shutdown:", err)
    }
    
    log.Println("Server exited")
}

func healthHandler(cfg *config.Config) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{"status": "ok", "app": "%s", "version": "%s"}`, cfg.App, cfg.Version)
    }
}

func configHandler(cfg *config.Config) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{"debug": %t, "server": {"host": "%s", "port": %d}}`, 
            cfg.Debug, cfg.Server.Host, cfg.Server.Port)
    }
}

func featuresHandler(cfg *config.Config) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.Write([]byte("{\"features\": {"))
        first := true
        for feature, enabled := range cfg.Features {
            if !first {
                w.Write([]byte(","))
            }
            fmt.Fprintf(w, `"%s": %t`, feature, enabled)
            first = false
        }
        w.Write([]byte("}}")
    }
}

func indexHandler(w http.ResponseWriter, r *http.Request) {
    html := `<!DOCTYPE html>
<html>
<head><title>My Project</title></head>
<body>
    <h1>Welcome to My Project</h1>
    <p>API endpoints:</p>
    <ul>
        <li><a href="/api/v1/health">Health Check</a></li>
        <li><a href="/api/v1/config">Configuration</a></li>
        <li><a href="/api/v1/features">Features</a></li>
    </ul>
</body>
</html>`
    w.Header().Set("Content-Type", "text/html")
    w.Write([]byte(html))
}

/*
To use vendoring:

1. Enable module mode:
   go mod init myproject

2. Add dependencies:
   go get github.com/gorilla/mux@v1.8.0
   go get github.com/pkg/errors@v0.9.1
   go get gopkg.in/yaml.v3@v3.0.1

3. Create vendor directory:
   go mod vendor

4. Build using vendor directory:
   go build -mod=vendor

5. Verify vendoring:
   go mod verify
*/
```

Vendoring creates a local copy of all dependencies in the vendor directory.  
This ensures reproducible builds, enables offline development, and provides  
protection against dependency changes. The go mod vendor command populates  
the vendor directory with exact versions from go.mod and go.sum.  

## Module-based imports

Modern Go uses modules for dependency management, allowing for versioned  
imports and semantic import versioning.  

```go
// go.mod - Module declaration and dependencies
module github.com/myorg/myapp

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/golang-jwt/jwt/v5 v5.0.0
    github.com/google/uuid v1.3.0
    github.com/redis/go-redis/v9 v9.1.0
    gorm.io/driver/postgres v1.5.2
    gorm.io/gorm v1.25.4
)

require (
    // Indirect dependencies managed automatically
    github.com/bytedance/sonic v1.9.1 // indirect
    github.com/chenzhuoyu/base64x v0.0.0-20221115062448-fe3a3abad311 // indirect
    github.com/gabriel-vasile/mimetype v1.4.2 // indirect
    // ... more indirect dependencies
)

replace (
    // Replace directive for local development
    github.com/myorg/shared => ../shared
    
    // Replace specific version with fork
    github.com/problematic/lib v1.2.3 => github.com/myorg/lib v1.2.4-fix
)

// auth/jwt.go - Using versioned imports
package auth

import (
    "fmt"
    "time"
    
    "github.com/golang-jwt/jwt/v5"  // v5 import path
    "github.com/google/uuid"
)

type Claims struct {
    UserID   string    `json:"user_id"`
    Username string    `json:"username"`
    Role     string    `json:"role"`
    IssuedAt time.Time `json:"issued_at"`
    jwt.RegisteredClaims
}

type JWTManager struct {
    secretKey     string
    tokenDuration time.Duration
}

func NewJWTManager(secretKey string, duration time.Duration) *JWTManager {
    return &JWTManager{
        secretKey:     secretKey,
        tokenDuration: duration,
    }
}

func (manager *JWTManager) Generate(userID, username, role string) (string, error) {
    claims := Claims{
        UserID:   userID,
        Username: username,
        Role:     role,
        IssuedAt: time.Now(),
        RegisteredClaims: jwt.RegisteredClaims{
            ID:        uuid.New().String(),
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(manager.tokenDuration)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            NotBefore: jwt.NewNumericDate(time.Now()),
            Issuer:    "myapp",
            Subject:   userID,
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString([]byte(manager.secretKey))
}

func (manager *JWTManager) Verify(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(
        tokenString,
        &Claims{},
        func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, fmt.Errorf("unexpected token signing method")
            }
            return []byte(manager.secretKey), nil
        },
    )
    
    if err != nil {
        return nil, fmt.Errorf("invalid token: %w", err)
    }
    
    claims, ok := token.Claims.(*Claims)
    if !ok {
        return nil, fmt.Errorf("invalid token claims")
    }
    
    return claims, nil
}

// cache/redis.go - Redis client with context support
package cache

import (
    "context"
    "encoding/json"
    "fmt"
    "time"
    
    "github.com/redis/go-redis/v9"  // v9 with context support
)

type RedisCache struct {
    client *redis.Client
    ctx    context.Context
}

func NewRedisCache(addr, password string, db int) *RedisCache {
    rdb := redis.NewClient(&redis.Options{
        Addr:     addr,
        Password: password,
        DB:       db,
    })
    
    return &RedisCache{
        client: rdb,
        ctx:    context.Background(),
    }
}

func (r *RedisCache) Set(key string, value interface{}, expiration time.Duration) error {
    data, err := json.Marshal(value)
    if err != nil {
        return fmt.Errorf("failed to marshal value: %w", err)
    }
    
    return r.client.Set(r.ctx, key, data, expiration).Err()
}

func (r *RedisCache) Get(key string, dest interface{}) error {
    val, err := r.client.Get(r.ctx, key).Result()
    if err != nil {
        if err == redis.Nil {
            return fmt.Errorf("key not found: %s", key)
        }
        return fmt.Errorf("failed to get key: %w", err)
    }
    
    return json.Unmarshal([]byte(val), dest)
}

func (r *RedisCache) Delete(key string) error {
    return r.client.Del(r.ctx, key).Err()
}

func (r *RedisCache) Exists(key string) bool {
    count, err := r.client.Exists(r.ctx, key).Result()
    return err == nil && count > 0
}

func (r *RedisCache) Close() error {
    return r.client.Close()
}

// models/user.go - GORM with PostgreSQL driver
package models

import (
    "time"
    
    "github.com/google/uuid"
    "gorm.io/gorm"
)

type User struct {
    ID        uuid.UUID      `gorm:"type:uuid;primary_key" json:"id"`
    Username  string         `gorm:"uniqueIndex;not null" json:"username"`
    Email     string         `gorm:"uniqueIndex;not null" json:"email"`
    Password  string         `gorm:"not null" json:"-"`
    Role      string         `gorm:"default:'user'" json:"role"`
    IsActive  bool           `gorm:"default:true" json:"is_active"`
    CreatedAt time.Time      `json:"created_at"`
    UpdatedAt time.Time      `json:"updated_at"`
    DeletedAt gorm.DeletedAt `gorm:"index" json:"-"`
}

func (u *User) BeforeCreate(tx *gorm.DB) error {
    if u.ID == uuid.Nil {
        u.ID = uuid.New()
    }
    return nil
}

type UserRepository struct {
    db *gorm.DB
}

func NewUserRepository(db *gorm.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) Create(user *User) error {
    return r.db.Create(user).Error
}

func (r *UserRepository) GetByID(id uuid.UUID) (*User, error) {
    var user User
    err := r.db.First(&user, "id = ?", id).Error
    return &user, err
}

func (r *UserRepository) GetByUsername(username string) (*User, error) {
    var user User
    err := r.db.First(&user, "username = ?", username).Error
    return &user, err
}

func (r *UserRepository) Update(user *User) error {
    return r.db.Save(user).Error
}

func (r *UserRepository) Delete(id uuid.UUID) error {
    return r.db.Delete(&User{}, "id = ?", id).Error
}

// main.go - Main application using all imports
package main

import (
    "log"
    "net/http"
    "time"
    
    "github.com/gin-gonic/gin"
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    
    "github.com/myorg/myapp/auth"
    "github.com/myorg/myapp/cache"
    "github.com/myorg/myapp/models"
)

func main() {
    // Initialize database
    dsn := "host=localhost user=postgres password=password dbname=myapp port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }
    
    // Auto-migrate models
    if err := db.AutoMigrate(&models.User{}); err != nil {
        log.Fatal("Failed to migrate database:", err)
    }
    
    // Initialize Redis cache
    redisCache := cache.NewRedisCache("localhost:6379", "", 0)
    defer redisCache.Close()
    
    // Initialize JWT manager
    jwtManager := auth.NewJWTManager("secret-key", 24*time.Hour)
    
    // Initialize repositories
    userRepo := models.NewUserRepository(db)
    
    // Create Gin router
    r := gin.Default()
    
    // Routes
    api := r.Group("/api/v1")
    {
        api.POST("/register", registerHandler(userRepo))
        api.POST("/login", loginHandler(userRepo, jwtManager, redisCache))
        api.GET("/profile", authMiddleware(jwtManager), profileHandler(userRepo))
    }
    
    // Start server
    log.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", r))
}

func registerHandler(repo *models.UserRepository) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Registration logic
        c.JSON(http.StatusOK, gin.H{"message": "User registered"})
    }
}

func loginHandler(repo *models.UserRepository, jwt *auth.JWTManager, cache *cache.RedisCache) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Login logic with JWT and Redis caching
        c.JSON(http.StatusOK, gin.H{"message": "Login successful"})
    }
}

func profileHandler(repo *models.UserRepository) gin.HandlerFunc {
    return func(c *gin.Context) {
        // Profile retrieval logic
        c.JSON(http.StatusOK, gin.H{"message": "Profile data"})
    }
}

func authMiddleware(jwt *auth.JWTManager) gin.HandlerFunc {
    return func(c *gin.Context) {
        // JWT authentication middleware
        c.Next()
    }
}

/*
Module management commands:

1. Initialize module:
   go mod init github.com/myorg/myapp

2. Add dependencies:
   go get github.com/gin-gonic/gin@latest
   go get github.com/golang-jwt/jwt/v5@latest

3. Update dependencies:
   go get -u ./...

4. Clean up unused dependencies:
   go mod tidy

5. Verify dependencies:
   go mod verify

6. Download dependencies:
   go mod download
*/
```

Module-based imports provide version management, semantic versioning, and  
reliable dependency resolution. Major version changes require different  
import paths (semantic import versioning), enabling multiple versions of  
the same package to coexist in a single build.  

## Package interfaces

Interfaces defined at the package level provide contract-based design  
and enable polymorphism across different implementations.  

```go
// storage/interface.go
package storage

import (
    "context"
    "io"
)

// Storage defines the interface for data storage operations
type Storage interface {
    Store(ctx context.Context, key string, data io.Reader) error
    Retrieve(ctx context.Context, key string) (io.ReadCloser, error)
    Delete(ctx context.Context, key string) error
    Exists(ctx context.Context, key string) (bool, error)
    List(ctx context.Context, prefix string) ([]string, error)
}

// Metadata provides additional information about stored objects
type Metadata interface {
    GetMetadata(ctx context.Context, key string) (map[string]string, error)
    SetMetadata(ctx context.Context, key string, metadata map[string]string) error
}

// StorageWithMetadata combines storage and metadata capabilities
type StorageWithMetadata interface {
    Storage
    Metadata
}

// storage/filesystem.go
package storage

import (
    "context"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
)

type FileSystemStorage struct {
    basePath string
}

func NewFileSystemStorage(basePath string) (*FileSystemStorage, error) {
    if err := os.MkdirAll(basePath, 0755); err != nil {
        return nil, fmt.Errorf("failed to create base directory: %w", err)
    }
    
    return &FileSystemStorage{basePath: basePath}, nil
}

func (fs *FileSystemStorage) Store(ctx context.Context, key string, data io.Reader) error {
    filePath := filepath.Join(fs.basePath, key)
    
    // Create directory if it doesn't exist
    if err := os.MkdirAll(filepath.Dir(filePath), 0755); err != nil {
        return fmt.Errorf("failed to create directory: %w", err)
    }
    
    file, err := os.Create(filePath)
    if err != nil {
        return fmt.Errorf("failed to create file: %w", err)
    }
    defer file.Close()
    
    _, err = io.Copy(file, data)
    return err
}

func (fs *FileSystemStorage) Retrieve(ctx context.Context, key string) (io.ReadCloser, error) {
    filePath := filepath.Join(fs.basePath, key)
    return os.Open(filePath)
}

func (fs *FileSystemStorage) Delete(ctx context.Context, key string) error {
    filePath := filepath.Join(fs.basePath, key)
    return os.Remove(filePath)
}

func (fs *FileSystemStorage) Exists(ctx context.Context, key string) (bool, error) {
    filePath := filepath.Join(fs.basePath, key)
    _, err := os.Stat(filePath)
    if err == nil {
        return true, nil
    }
    if os.IsNotExist(err) {
        return false, nil
    }
    return false, err
}

func (fs *FileSystemStorage) List(ctx context.Context, prefix string) ([]string, error) {
    var keys []string
    
    err := filepath.Walk(fs.basePath, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        if !info.IsDir() {
            relPath, err := filepath.Rel(fs.basePath, path)
            if err != nil {
                return err
            }
            
            if strings.HasPrefix(relPath, prefix) {
                keys = append(keys, relPath)
            }
        }
        return nil
    })
    
    return keys, err
}

// storage/memory.go
package storage

import (
    "bytes"
    "context"
    "fmt"
    "io"
    "strings"
    "sync"
)

type MemoryStorage struct {
    data     map[string][]byte
    metadata map[string]map[string]string
    mutex    sync.RWMutex
}

func NewMemoryStorage() *MemoryStorage {
    return &MemoryStorage{
        data:     make(map[string][]byte),
        metadata: make(map[string]map[string]string),
    }
}

func (ms *MemoryStorage) Store(ctx context.Context, key string, data io.Reader) error {
    content, err := io.ReadAll(data)
    if err != nil {
        return fmt.Errorf("failed to read data: %w", err)
    }
    
    ms.mutex.Lock()
    defer ms.mutex.Unlock()
    
    ms.data[key] = content
    return nil
}

func (ms *MemoryStorage) Retrieve(ctx context.Context, key string) (io.ReadCloser, error) {
    ms.mutex.RLock()
    defer ms.mutex.RUnlock()
    
    content, exists := ms.data[key]
    if !exists {
        return nil, fmt.Errorf("key not found: %s", key)
    }
    
    return io.NopCloser(bytes.NewReader(content)), nil
}

func (ms *MemoryStorage) Delete(ctx context.Context, key string) error {
    ms.mutex.Lock()
    defer ms.mutex.Unlock()
    
    delete(ms.data, key)
    delete(ms.metadata, key)
    return nil
}

func (ms *MemoryStorage) Exists(ctx context.Context, key string) (bool, error) {
    ms.mutex.RLock()
    defer ms.mutex.RUnlock()
    
    _, exists := ms.data[key]
    return exists, nil
}

func (ms *MemoryStorage) List(ctx context.Context, prefix string) ([]string, error) {
    ms.mutex.RLock()
    defer ms.mutex.Unlock()
    
    var keys []string
    for key := range ms.data {
        if strings.HasPrefix(key, prefix) {
            keys = append(keys, key)
        }
    }
    return keys, nil
}

func (ms *MemoryStorage) GetMetadata(ctx context.Context, key string) (map[string]string, error) {
    ms.mutex.RLock()
    defer ms.mutex.RUnlock()
    
    metadata, exists := ms.metadata[key]
    if !exists {
        return make(map[string]string), nil
    }
    
    // Return a copy to prevent external modification
    result := make(map[string]string)
    for k, v := range metadata {
        result[k] = v
    }
    return result, nil
}

func (ms *MemoryStorage) SetMetadata(ctx context.Context, key string, metadata map[string]string) error {
    ms.mutex.Lock()
    defer ms.mutex.Unlock()
    
    if ms.metadata[key] == nil {
        ms.metadata[key] = make(map[string]string)
    }
    
    for k, v := range metadata {
        ms.metadata[key][k] = v
    }
    return nil
}

// main.go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "strings"
    "your-module/storage"
)

func demonstrateStorage(s storage.Storage, name string) {
    ctx := context.Background()
    
    fmt.Printf("\n=== Demonstrating %s ===\n", name)
    
    // Store data
    data := strings.NewReader("Hello there, this is test data!")
    if err := s.Store(ctx, "test/file1.txt", data); err != nil {
        log.Printf("Store error: %v", err)
        return
    }
    
    // Check if exists
    exists, err := s.Exists(ctx, "test/file1.txt")
    if err != nil {
        log.Printf("Exists error: %v", err)
        return
    }
    fmt.Printf("File exists: %t\n", exists)
    
    // Retrieve data
    reader, err := s.Retrieve(ctx, "test/file1.txt")
    if err != nil {
        log.Printf("Retrieve error: %v", err)
        return
    }
    defer reader.Close()
    
    content, err := io.ReadAll(reader)
    if err != nil {
        log.Printf("Read error: %v", err)
        return
    }
    fmt.Printf("Retrieved content: %s\n", string(content))
    
    // Store more files for listing
    s.Store(ctx, "test/file2.txt", strings.NewReader("Second file"))
    s.Store(ctx, "other/file3.txt", strings.NewReader("Third file"))
    
    // List files with prefix
    keys, err := s.List(ctx, "test/")
    if err != nil {
        log.Printf("List error: %v", err)
        return
    }
    fmt.Printf("Files with 'test/' prefix: %v\n", keys)
    
    // Test metadata if supported
    if metaStorage, ok := s.(storage.StorageWithMetadata); ok {
        metadata := map[string]string{
            "content-type": "text/plain",
            "created-by":   "demo-app",
        }
        if err := metaStorage.SetMetadata(ctx, "test/file1.txt", metadata); err != nil {
            log.Printf("SetMetadata error: %v", err)
        } else {
            retrievedMeta, err := metaStorage.GetMetadata(ctx, "test/file1.txt")
            if err != nil {
                log.Printf("GetMetadata error: %v", err)
            } else {
                fmt.Printf("Metadata: %v\n", retrievedMeta)
            }
        }
    }
}

func main() {
    // Create file system storage
    fsStorage, err := storage.NewFileSystemStorage("./data")
    if err != nil {
        log.Fatal("Failed to create filesystem storage:", err)
    }
    
    // Create memory storage
    memStorage := storage.NewMemoryStorage()
    
    // Demonstrate both implementations using the same interface
    storages := []struct {
        impl storage.Storage
        name string
    }{
        {fsStorage, "FileSystem Storage"},
        {memStorage, "Memory Storage"},
    }
    
    for _, s := range storages {
        demonstrateStorage(s.impl, s.name)
    }
    
    // Cleanup
    ctx := context.Background()
    fsStorage.Delete(ctx, "test/file1.txt")
    fsStorage.Delete(ctx, "test/file2.txt")
    fsStorage.Delete(ctx, "other/file3.txt")
}
```

Package-level interfaces enable polymorphism and clean architecture patterns.  
Different implementations can be swapped without changing client code,  
promoting testability and flexibility. Interface segregation allows  
for focused contracts that are easier to implement and understand.  

## Package constants and variables

Package-level constants and variables provide shared state and configuration  
for package functionality.  

```go
// http/constants.go
package http

import "time"

// HTTP status code constants
const (
    StatusContinue           = 100
    StatusSwitchingProtocols = 101
    StatusProcessing         = 102
    StatusEarlyHints         = 103
    
    StatusOK                   = 200
    StatusCreated              = 201
    StatusAccepted             = 202
    StatusNonAuthoritativeInfo = 203
    StatusNoContent            = 204
    StatusResetContent         = 205
    StatusPartialContent       = 206
    
    StatusMultipleChoices   = 300
    StatusMovedPermanently  = 301
    StatusFound             = 302
    StatusSeeOther          = 303
    StatusNotModified       = 304
    StatusUseProxy          = 305
    StatusTemporaryRedirect = 307
    StatusPermanentRedirect = 308
    
    StatusBadRequest                   = 400
    StatusUnauthorized                 = 401
    StatusPaymentRequired              = 402
    StatusForbidden                    = 403
    StatusNotFound                     = 404
    StatusMethodNotAllowed             = 405
    StatusNotAcceptable                = 406
    StatusProxyAuthRequired            = 407
    StatusRequestTimeout               = 408
    StatusConflict                     = 409
    StatusGone                         = 410
    StatusLengthRequired               = 411
    StatusPreconditionFailed           = 412
    StatusRequestEntityTooLarge        = 413
    StatusRequestURITooLong            = 414
    StatusUnsupportedMediaType         = 415
    StatusRequestedRangeNotSatisfiable = 416
    StatusExpectationFailed            = 417
    StatusTeapot                       = 418
    StatusMisdirectedRequest           = 421
    StatusUnprocessableEntity          = 422
    StatusLocked                       = 423
    StatusFailedDependency             = 424
    StatusTooEarly                     = 425
    StatusUpgradeRequired              = 426
    StatusPreconditionRequired         = 428
    StatusTooManyRequests              = 429
    StatusRequestHeaderFieldsTooLarge  = 431
    StatusUnavailableForLegalReasons   = 451
    
    StatusInternalServerError           = 500
    StatusNotImplemented                = 501
    StatusBadGateway                    = 502
    StatusServiceUnavailable            = 503
    StatusGatewayTimeout                = 504
    StatusHTTPVersionNotSupported       = 505
    StatusVariantAlsoNegotiates         = 506
    StatusInsufficientStorage           = 507
    StatusLoopDetected                  = 508
    StatusNotExtended                   = 510
    StatusNetworkAuthenticationRequired = 511
)

// HTTP method constants
const (
    MethodGet     = "GET"
    MethodHead    = "HEAD"
    MethodPost    = "POST"
    MethodPut     = "PUT"
    MethodPatch   = "PATCH"
    MethodDelete  = "DELETE"
    MethodConnect = "CONNECT"
    MethodOptions = "OPTIONS"
    MethodTrace   = "TRACE"
)

// Header name constants
const (
    HeaderAccept                          = "Accept"
    HeaderAcceptCharset                   = "Accept-Charset"
    HeaderAcceptEncoding                  = "Accept-Encoding"
    HeaderAcceptLanguage                  = "Accept-Language"
    HeaderAcceptRanges                    = "Accept-Ranges"
    HeaderAccessControlAllowCredentials   = "Access-Control-Allow-Credentials"
    HeaderAccessControlAllowHeaders       = "Access-Control-Allow-Headers"
    HeaderAccessControlAllowMethods       = "Access-Control-Allow-Methods"
    HeaderAccessControlAllowOrigin        = "Access-Control-Allow-Origin"
    HeaderAccessControlExposeHeaders      = "Access-Control-Expose-Headers"
    HeaderAccessControlMaxAge             = "Access-Control-Max-Age"
    HeaderAccessControlRequestHeaders     = "Access-Control-Request-Headers"
    HeaderAccessControlRequestMethod      = "Access-Control-Request-Method"
    HeaderAge                             = "Age"
    HeaderAllow                           = "Allow"
    HeaderAuthorization                   = "Authorization"
    HeaderCacheControl                    = "Cache-Control"
    HeaderConnection                      = "Connection"
    HeaderContentDisposition              = "Content-Disposition"
    HeaderContentEncoding                 = "Content-Encoding"
    HeaderContentLanguage                 = "Content-Language"
    HeaderContentLength                   = "Content-Length"
    HeaderContentLocation                 = "Content-Location"
    HeaderContentMD5                      = "Content-MD5"
    HeaderContentRange                    = "Content-Range"
    HeaderContentType                     = "Content-Type"
    HeaderCookie                          = "Cookie"
    HeaderDate                            = "Date"
    HeaderEtag                            = "Etag"
    HeaderExpect                          = "Expect"
    HeaderExpires                         = "Expires"
    HeaderFrom                            = "From"
    HeaderHost                            = "Host"
    HeaderIfMatch                         = "If-Match"
    HeaderIfModifiedSince                 = "If-Modified-Since"
    HeaderIfNoneMatch                     = "If-None-Match"
    HeaderIfRange                         = "If-Range"
    HeaderIfUnmodifiedSince               = "If-Unmodified-Since"
    HeaderLastModified                    = "Last-Modified"
    HeaderLocation                        = "Location"
    HeaderMaxForwards                     = "Max-Forwards"
    HeaderOrigin                          = "Origin"
    HeaderPragma                          = "Pragma"
    HeaderProxyAuthenticate               = "Proxy-Authenticate"
    HeaderProxyAuthorization              = "Proxy-Authorization"
    HeaderRange                           = "Range"
    HeaderReferer                         = "Referer"
    HeaderRetryAfter                      = "Retry-After"
    HeaderServer                          = "Server"
    HeaderSetCookie                       = "Set-Cookie"
    HeaderTE                              = "Te"
    HeaderTrailer                         = "Trailer"
    HeaderTransferEncoding                = "Transfer-Encoding"
    HeaderUpgrade                         = "Upgrade"
    HeaderUserAgent                       = "User-Agent"
    HeaderVary                            = "Vary"
    HeaderVia                             = "Via"
    HeaderWarning                         = "Warning"
    HeaderWWWAuthenticate                 = "WWW-Authenticate"
)

// Timeout constants
const (
    DefaultTimeout         = 30 * time.Second
    DefaultKeepAliveTimeout = 90 * time.Second
    DefaultReadTimeout     = 15 * time.Second
    DefaultWriteTimeout    = 15 * time.Second
    DefaultIdleTimeout     = 60 * time.Second
)

// http/variables.go
package http

import (
    "fmt"
    "net/http"
    "sync"
)

// Package-level variables for shared state
var (
    // DefaultClient provides a default HTTP client with sensible timeouts
    DefaultClient *http.Client
    
    // RequestCounter tracks the number of requests made
    RequestCounter int64
    
    // ActiveConnections tracks current active connections
    ActiveConnections int32
    
    // StatusMessages maps status codes to human-readable messages
    StatusMessages map[int]string
    
    // GlobalHeaders are headers added to all requests
    GlobalHeaders map[string]string
    
    // Mutex for thread-safe operations
    mu sync.RWMutex
    
    // Debug mode flag
    DebugMode bool
    
    // Default user agent string
    DefaultUserAgent string
)

// Package initialization
func init() {
    // Initialize default client
    DefaultClient = &http.Client{
        Timeout: DefaultTimeout,
        Transport: &http.Transport{
            MaxIdleConns:        100,
            IdleConnTimeout:     DefaultIdleTimeout,
            DisableCompression:  false,
            DisableKeepAlives:   false,
        },
    }
    
    // Initialize status messages
    StatusMessages = map[int]string{
        StatusOK:                    "OK",
        StatusCreated:               "Created",
        StatusAccepted:              "Accepted",
        StatusNoContent:             "No Content",
        StatusBadRequest:            "Bad Request",
        StatusUnauthorized:          "Unauthorized",
        StatusForbidden:             "Forbidden",
        StatusNotFound:              "Not Found",
        StatusMethodNotAllowed:      "Method Not Allowed",
        StatusConflict:              "Conflict",
        StatusInternalServerError:   "Internal Server Error",
        StatusNotImplemented:        "Not Implemented",
        StatusBadGateway:           "Bad Gateway",
        StatusServiceUnavailable:   "Service Unavailable",
        StatusGatewayTimeout:       "Gateway Timeout",
    }
    
    // Initialize global headers
    GlobalHeaders = make(map[string]string)
    GlobalHeaders[HeaderUserAgent] = "MyApp/1.0"
    
    // Set default user agent
    DefaultUserAgent = "MyApp/1.0 (Go HTTP Client)"
    
    fmt.Println("HTTP package initialized")
}

// Exported functions to work with package variables
func SetDebugMode(enabled bool) {
    mu.Lock()
    defer mu.Unlock()
    DebugMode = enabled
}

func IsDebugMode() bool {
    mu.RLock()
    defer mu.RUnlock()
    return DebugMode
}

func IncrementRequestCounter() int64 {
    mu.Lock()
    defer mu.Unlock()
    RequestCounter++
    return RequestCounter
}

func GetRequestCount() int64 {
    mu.RLock()
    defer mu.RUnlock()
    return RequestCounter
}

func SetGlobalHeader(key, value string) {
    mu.Lock()
    defer mu.Unlock()
    GlobalHeaders[key] = value
}

func GetGlobalHeaders() map[string]string {
    mu.RLock()
    defer mu.RUnlock()
    
    // Return a copy to prevent external modification
    headers := make(map[string]string)
    for k, v := range GlobalHeaders {
        headers[k] = v
    }
    return headers
}

func GetStatusMessage(code int) string {
    mu.RLock()
    defer mu.RUnlock()
    
    if message, exists := StatusMessages[code]; exists {
        return message
    }
    return fmt.Sprintf("Unknown Status Code: %d", code)
}

func SetDefaultTimeout(timeout time.Duration) {
    mu.Lock()
    defer mu.Unlock()
    DefaultClient.Timeout = timeout
}

// http/client.go
package http

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "sync/atomic"
)

// Request represents an HTTP request with additional metadata
type Request struct {
    Method  string
    URL     string
    Headers map[string]string
    Body    interface{}
}

// Response represents an HTTP response with parsed data
type Response struct {
    StatusCode int
    Status     string
    Headers    map[string]string
    Body       []byte
    Request    *Request
}

// Client wraps the default HTTP client with additional functionality
type Client struct {
    httpClient *http.Client
    baseURL    string
    headers    map[string]string
}

func NewClient(baseURL string) *Client {
    return &Client{
        httpClient: DefaultClient,
        baseURL:    baseURL,
        headers:    GetGlobalHeaders(),
    }
}

func (c *Client) SetHeader(key, value string) {
    if c.headers == nil {
        c.headers = make(map[string]string)
    }
    c.headers[key] = value
}

func (c *Client) Do(req *Request) (*Response, error) {
    // Increment request counter
    count := IncrementRequestCounter()
    
    if IsDebugMode() {
        fmt.Printf("Making request #%d: %s %s\n", count, req.Method, req.URL)
    }
    
    // Prepare request body
    var bodyReader io.Reader
    if req.Body != nil {
        switch body := req.Body.(type) {
        case string:
            bodyReader = bytes.NewReader([]byte(body))
        case []byte:
            bodyReader = bytes.NewReader(body)
        default:
            jsonBody, err := json.Marshal(body)
            if err != nil {
                return nil, fmt.Errorf("failed to marshal request body: %w", err)
            }
            bodyReader = bytes.NewReader(jsonBody)
        }
    }
    
    // Create HTTP request
    fullURL := c.baseURL + req.URL
    httpReq, err := http.NewRequest(req.Method, fullURL, bodyReader)
    if err != nil {
        return nil, fmt.Errorf("failed to create request: %w", err)
    }
    
    // Add headers
    for key, value := range c.headers {
        httpReq.Header.Set(key, value)
    }
    for key, value := range req.Headers {
        httpReq.Header.Set(key, value)
    }
    
    // Track active connection
    atomic.AddInt32(&ActiveConnections, 1)
    defer atomic.AddInt32(&ActiveConnections, -1)
    
    // Make request
    httpResp, err := c.httpClient.Do(httpReq)
    if err != nil {
        return nil, fmt.Errorf("request failed: %w", err)
    }
    defer httpResp.Body.Close()
    
    // Read response body
    body, err := io.ReadAll(httpResp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read response body: %w", err)
    }
    
    // Parse response headers
    headers := make(map[string]string)
    for key, values := range httpResp.Header {
        if len(values) > 0 {
            headers[key] = values[0]
        }
    }
    
    response := &Response{
        StatusCode: httpResp.StatusCode,
        Status:     GetStatusMessage(httpResp.StatusCode),
        Headers:    headers,
        Body:       body,
        Request:    req,
    }
    
    if IsDebugMode() {
        fmt.Printf("Response #%d: %d %s\n", count, response.StatusCode, response.Status)
    }
    
    return response, nil
}

func (c *Client) Get(url string) (*Response, error) {
    return c.Do(&Request{Method: MethodGet, URL: url})
}

func (c *Client) Post(url string, body interface{}) (*Response, error) {
    return c.Do(&Request{Method: MethodPost, URL: url, Body: body})
}

func (c *Client) Put(url string, body interface{}) (*Response, error) {
    return c.Do(&Request{Method: MethodPut, URL: url, Body: body})
}

func (c *Client) Delete(url string) (*Response, error) {
    return c.Do(&Request{Method: MethodDelete, URL: url})
}

// Package-level convenience functions
func Get(url string) (*Response, error) {
    client := NewClient("")
    return client.Get(url)
}

func Post(url string, body interface{}) (*Response, error) {
    client := NewClient("")
    return client.Post(url, body)
}

func GetStats() (requestCount int64, activeConnections int32) {
    return GetRequestCount(), atomic.LoadInt32(&ActiveConnections)
}

// main.go
package main

import (
    "fmt"
    "log"
    "time"
    "your-module/http"
)

func main() {
    // Configure package-level settings
    http.SetDebugMode(true)
    http.SetGlobalHeader("X-API-Version", "v1")
    http.SetDefaultTimeout(10 * time.Second)
    
    // Create HTTP client
    client := http.NewClient("https://httpbin.org")
    client.SetHeader(http.HeaderContentType, "application/json")
    
    // Make requests using package constants
    response, err := client.Get("/get")
    if err != nil {
        log.Fatal("GET request failed:", err)
    }
    
    fmt.Printf("Status: %d (%s)\n", response.StatusCode, response.Status)
    fmt.Printf("Response body length: %d bytes\n", len(response.Body))
    
    // Make POST request
    postData := map[string]string{
        "message": "Hello there!",
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    response, err = client.Post("/post", postData)
    if err != nil {
        log.Fatal("POST request failed:", err)
    }
    
    fmt.Printf("POST Status: %d\n", response.StatusCode)
    
    // Check package statistics
    requestCount, activeConns := http.GetStats()
    fmt.Printf("Total requests made: %d\n", requestCount)
    fmt.Printf("Active connections: %d\n", activeConns)
    
    // Test status code constants
    fmt.Printf("HTTP Status Messages:\n")
    codes := []int{
        http.StatusOK, 
        http.StatusCreated, 
        http.StatusBadRequest, 
        http.StatusNotFound, 
        http.StatusInternalServerError,
    }
    
    for _, code := range codes {
        fmt.Printf("  %d: %s\n", code, http.GetStatusMessage(code))
    }
    
    // Display global headers
    fmt.Printf("Global headers: %v\n", http.GetGlobalHeaders())
}
```

Package constants and variables provide shared configuration and state  
management. Constants offer compile-time guarantees and eliminate magic  
numbers, while package variables enable shared state with proper  
synchronization. Initialization functions set up default values and  
prepare the package for use.  

## Package level functions

Package-level functions provide the primary API and utility functions  
that operate on package types and data structures.  

```go
// mathutils/functions.go
package mathutils

import (
    "errors"
    "math"
)

// Basic arithmetic functions
func Add(a, b float64) float64 {
    return a + b
}

func Subtract(a, b float64) float64 {
    return a - b
}

func Multiply(a, b float64) float64 {
    return a * b
}

func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Advanced mathematical functions
func Power(base, exponent float64) float64 {
    return math.Pow(base, exponent)
}

func Sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, errors.New("square root of negative number")
    }
    return math.Sqrt(x), nil
}

func Factorial(n int) (int64, error) {
    if n < 0 {
        return 0, errors.New("factorial of negative number")
    }
    if n == 0 || n == 1 {
        return 1, nil
    }
    
    result := int64(1)
    for i := 2; i <= n; i++ {
        result *= int64(i)
    }
    return result, nil
}

// IsPrime checks if a number is prime
func IsPrime(n int) bool {
    if n < 2 {
        return false
    }
    if n == 2 {
        return true
    }
    if n%2 == 0 {
        return false
    }
    
    limit := int(math.Sqrt(float64(n)))
    for i := 3; i <= limit; i += 2 {
        if n%i == 0 {
            return false
        }
    }
    return true
}

// GCD calculates the greatest common divisor
func GCD(a, b int) int {
    for b != 0 {
        a, b = b, a%b
    }
    return a
}

// LCM calculates the least common multiple
func LCM(a, b int) int {
    return (a * b) / GCD(a, b)
}

// Fibonacci generates Fibonacci sequence up to n terms
func Fibonacci(n int) []int {
    if n <= 0 {
        return []int{}
    }
    if n == 1 {
        return []int{0}
    }
    if n == 2 {
        return []int{0, 1}
    }
    
    fib := make([]int, n)
    fib[0], fib[1] = 0, 1
    
    for i := 2; i < n; i++ {
        fib[i] = fib[i-1] + fib[i-2]
    }
    return fib
}

// Statistical functions
func Mean(numbers []float64) float64 {
    if len(numbers) == 0 {
        return 0
    }
    
    sum := 0.0
    for _, num := range numbers {
        sum += num
    }
    return sum / float64(len(numbers))
}

func Median(numbers []float64) float64 {
    if len(numbers) == 0 {
        return 0
    }
    
    // Create a copy and sort it
    sorted := make([]float64, len(numbers))
    copy(sorted, numbers)
    
    // Simple bubble sort for demonstration
    for i := 0; i < len(sorted); i++ {
        for j := 0; j < len(sorted)-i-1; j++ {
            if sorted[j] > sorted[j+1] {
                sorted[j], sorted[j+1] = sorted[j+1], sorted[j]
            }
        }
    }
    
    mid := len(sorted) / 2
    if len(sorted)%2 == 0 {
        return (sorted[mid-1] + sorted[mid]) / 2
    }
    return sorted[mid]
}

func StandardDeviation(numbers []float64) float64 {
    if len(numbers) == 0 {
        return 0
    }
    
    mean := Mean(numbers)
    sumSquaredDiffs := 0.0
    
    for _, num := range numbers {
        diff := num - mean
        sumSquaredDiffs += diff * diff
    }
    
    variance := sumSquaredDiffs / float64(len(numbers))
    return math.Sqrt(variance)
}

// Trigonometric functions with degree conversion
func SinDegrees(degrees float64) float64 {
    radians := degrees * math.Pi / 180
    return math.Sin(radians)
}

func CosDegrees(degrees float64) float64 {
    radians := degrees * math.Pi / 180
    return math.Cos(radians)
}

func TanDegrees(degrees float64) float64 {
    radians := degrees * math.Pi / 180
    return math.Tan(radians)
}

// Conversion functions
func DegreesToRadians(degrees float64) float64 {
    return degrees * math.Pi / 180
}

func RadiansToDegrees(radians float64) float64 {
    return radians * 180 / math.Pi
}

// Utility functions
func Min(numbers ...float64) float64 {
    if len(numbers) == 0 {
        return 0
    }
    
    min := numbers[0]
    for _, num := range numbers[1:] {
        if num < min {
            min = num
        }
    }
    return min
}

func Max(numbers ...float64) float64 {
    if len(numbers) == 0 {
        return 0
    }
    
    max := numbers[0]
    for _, num := range numbers[1:] {
        if num > max {
            max = num
        }
    }
    return max
}

func Abs(x float64) float64 {
    if x < 0 {
        return -x
    }
    return x
}

func Round(x float64, precision int) float64 {
    multiplier := math.Pow(10, float64(precision))
    return math.Round(x*multiplier) / multiplier
}

func Clamp(value, min, max float64) float64 {
    if value < min {
        return min
    }
    if value > max {
        return max
    }
    return value
}

// main.go
package main

import (
    "fmt"
    "log"
    "your-module/mathutils"
)

func main() {
    fmt.Println("=== Basic Arithmetic ===")
    fmt.Printf("10 + 5 = %.2f\n", mathutils.Add(10, 5))
    fmt.Printf("10 - 5 = %.2f\n", mathutils.Subtract(10, 5))
    fmt.Printf("10 * 5 = %.2f\n", mathutils.Multiply(10, 5))
    
    if result, err := mathutils.Divide(10, 5); err != nil {
        log.Printf("Division error: %v", err)
    } else {
        fmt.Printf("10 / 5 = %.2f\n", result)
    }
    
    fmt.Println("\n=== Advanced Functions ===")
    fmt.Printf("2^8 = %.0f\n", mathutils.Power(2, 8))
    
    if sqrt, err := mathutils.Sqrt(25); err != nil {
        log.Printf("Square root error: %v", err)
    } else {
        fmt.Printf("√25 = %.2f\n", sqrt)
    }
    
    if fact, err := mathutils.Factorial(5); err != nil {
        log.Printf("Factorial error: %v", err)
    } else {
        fmt.Printf("5! = %d\n", fact)
    }
    
    fmt.Println("\n=== Number Theory ===")
    fmt.Printf("Is 17 prime? %t\n", mathutils.IsPrime(17))
    fmt.Printf("Is 21 prime? %t\n", mathutils.IsPrime(21))
    fmt.Printf("GCD(48, 18) = %d\n", mathutils.GCD(48, 18))
    fmt.Printf("LCM(12, 8) = %d\n", mathutils.LCM(12, 8))
    
    fmt.Println("\n=== Fibonacci Sequence ===")
    fib := mathutils.Fibonacci(10)
    fmt.Printf("First 10 Fibonacci numbers: %v\n", fib)
    
    fmt.Println("\n=== Statistics ===")
    data := []float64{10, 20, 30, 40, 50, 25, 35, 45}
    fmt.Printf("Data: %v\n", data)
    fmt.Printf("Mean: %.2f\n", mathutils.Mean(data))
    fmt.Printf("Median: %.2f\n", mathutils.Median(data))
    fmt.Printf("Standard Deviation: %.2f\n", mathutils.StandardDeviation(data))
    
    fmt.Println("\n=== Trigonometry ===")
    angle := 45.0
    fmt.Printf("sin(45°) = %.4f\n", mathutils.SinDegrees(angle))
    fmt.Printf("cos(45°) = %.4f\n", mathutils.CosDegrees(angle))
    fmt.Printf("tan(45°) = %.4f\n", mathutils.TanDegrees(angle))
    
    fmt.Println("\n=== Conversions ===")
    degrees := 180.0
    radians := mathutils.DegreesToRadians(degrees)
    fmt.Printf("180° = %.4f radians\n", radians)
    fmt.Printf("π radians = %.2f degrees\n", mathutils.RadiansToDegrees(radians))
    
    fmt.Println("\n=== Utility Functions ===")
    values := []float64{3.7, 1.2, 8.9, 2.4, 6.1}
    fmt.Printf("Values: %v\n", values)
    fmt.Printf("Min: %.2f\n", mathutils.Min(values...))
    fmt.Printf("Max: %.2f\n", mathutils.Max(values...))
    fmt.Printf("Absolute value of -7.3: %.2f\n", mathutils.Abs(-7.3))
    fmt.Printf("Round 3.14159 to 2 decimals: %.2f\n", mathutils.Round(3.14159, 2))
    fmt.Printf("Clamp 15 between 5 and 10: %.0f\n", mathutils.Clamp(15, 5, 10))
}
```

Package-level functions form the primary API surface of a package. They  
should be well-designed, documented, and handle edge cases appropriately.  
Function names should be clear and follow Go naming conventions, with  
exported functions providing the public interface.  

## Package factory patterns

Factory patterns in packages provide controlled object creation with  
configuration, validation, and initialization logic.  

```go
// database/factory.go
package database

import (
    "database/sql"
    "fmt"
    "time"
    
    _ "github.com/lib/pq"           // PostgreSQL driver
    _ "github.com/go-sql-driver/mysql" // MySQL driver
    _ "github.com/mattn/go-sqlite3"    // SQLite driver
)

// DatabaseType represents supported database types
type DatabaseType string

const (
    PostgreSQL DatabaseType = "postgres"
    MySQL      DatabaseType = "mysql" 
    SQLite     DatabaseType = "sqlite3"
)

// Config holds database configuration
type Config struct {
    Type            DatabaseType  `json:"type"`
    Host            string       `json:"host"`
    Port            int          `json:"port"`
    Database        string       `json:"database"`
    Username        string       `json:"username"`
    Password        string       `json:"password"`
    SSLMode         string       `json:"ssl_mode"`
    MaxConnections  int          `json:"max_connections"`
    MaxIdleTime     time.Duration `json:"max_idle_time"`
    MaxLifetime     time.Duration `json:"max_lifetime"`
    ConnectTimeout  time.Duration `json:"connect_timeout"`
    FilePath        string       `json:"file_path"` // For SQLite
}

// Connection wraps sql.DB with additional functionality
type Connection struct {
    db     *sql.DB
    config Config
    stats  ConnectionStats
}

type ConnectionStats struct {
    TotalConnections int64
    ActiveConnections int32
    CreatedAt       time.Time
}

// ConnectionFactory provides methods for creating database connections
type ConnectionFactory struct {
    defaultConfig Config
}

// NewConnectionFactory creates a new connection factory with default settings
func NewConnectionFactory() *ConnectionFactory {
    return &ConnectionFactory{
        defaultConfig: Config{
            Type:           PostgreSQL,
            Host:           "localhost",
            Port:           5432,
            SSLMode:        "disable",
            MaxConnections: 25,
            MaxIdleTime:    5 * time.Minute,
            MaxLifetime:    1 * time.Hour,
            ConnectTimeout: 10 * time.Second,
        },
    }
}

// SetDefaults updates the default configuration
func (f *ConnectionFactory) SetDefaults(config Config) {
    f.defaultConfig = config
}

// CreateConnection creates a new database connection with the given configuration
func (f *ConnectionFactory) CreateConnection(config Config) (*Connection, error) {
    // Merge with defaults
    finalConfig := f.mergeWithDefaults(config)
    
    // Validate configuration
    if err := f.validateConfig(finalConfig); err != nil {
        return nil, fmt.Errorf("invalid configuration: %w", err)
    }
    
    // Build connection string
    dsn, err := f.buildDSN(finalConfig)
    if err != nil {
        return nil, fmt.Errorf("failed to build DSN: %w", err)
    }
    
    // Open database connection
    db, err := sql.Open(string(finalConfig.Type), dsn)
    if err != nil {
        return nil, fmt.Errorf("failed to open database: %w", err)
    }
    
    // Configure connection pool
    db.SetMaxOpenConns(finalConfig.MaxConnections)
    db.SetMaxIdleConns(finalConfig.MaxConnections / 2)
    db.SetConnMaxIdleTime(finalConfig.MaxIdleTime)
    db.SetConnMaxLifetime(finalConfig.MaxLifetime)
    
    // Test connection
    if err := db.Ping(); err != nil {
        db.Close()
        return nil, fmt.Errorf("failed to ping database: %w", err)
    }
    
    connection := &Connection{
        db:     db,
        config: finalConfig,
        stats: ConnectionStats{
            CreatedAt: time.Now(),
        },
    }
    
    return connection, nil
}

// CreatePostgreSQLConnection creates a PostgreSQL connection
func (f *ConnectionFactory) CreatePostgreSQLConnection(host string, port int, database, username, password string) (*Connection, error) {
    config := Config{
        Type:     PostgreSQL,
        Host:     host,
        Port:     port,
        Database: database,
        Username: username,
        Password: password,
        SSLMode:  "disable",
    }
    return f.CreateConnection(config)
}

// CreateMySQLConnection creates a MySQL connection
func (f *ConnectionFactory) CreateMySQLConnection(host string, port int, database, username, password string) (*Connection, error) {
    config := Config{
        Type:     MySQL,
        Host:     host,
        Port:     port,
        Database: database,
        Username: username,
        Password: password,
    }
    return f.CreateConnection(config)
}

// CreateSQLiteConnection creates a SQLite connection
func (f *ConnectionFactory) CreateSQLiteConnection(filePath string) (*Connection, error) {
    config := Config{
        Type:     SQLite,
        FilePath: filePath,
    }
    return f.CreateConnection(config)
}

func (f *ConnectionFactory) mergeWithDefaults(config Config) Config {
    if config.Type == "" {
        config.Type = f.defaultConfig.Type
    }
    if config.Host == "" {
        config.Host = f.defaultConfig.Host
    }
    if config.Port == 0 {
        config.Port = f.defaultConfig.Port
    }
    if config.SSLMode == "" {
        config.SSLMode = f.defaultConfig.SSLMode
    }
    if config.MaxConnections == 0 {
        config.MaxConnections = f.defaultConfig.MaxConnections
    }
    if config.MaxIdleTime == 0 {
        config.MaxIdleTime = f.defaultConfig.MaxIdleTime
    }
    if config.MaxLifetime == 0 {
        config.MaxLifetime = f.defaultConfig.MaxLifetime
    }
    if config.ConnectTimeout == 0 {
        config.ConnectTimeout = f.defaultConfig.ConnectTimeout
    }
    return config
}

func (f *ConnectionFactory) validateConfig(config Config) error {
    switch config.Type {
    case PostgreSQL, MySQL:
        if config.Host == "" {
            return fmt.Errorf("host is required for %s", config.Type)
        }
        if config.Port <= 0 || config.Port > 65535 {
            return fmt.Errorf("invalid port: %d", config.Port)
        }
        if config.Database == "" {
            return fmt.Errorf("database name is required")
        }
        if config.Username == "" {
            return fmt.Errorf("username is required")
        }
    case SQLite:
        if config.FilePath == "" {
            return fmt.Errorf("file path is required for SQLite")
        }
    default:
        return fmt.Errorf("unsupported database type: %s", config.Type)
    }
    
    if config.MaxConnections <= 0 {
        return fmt.Errorf("max connections must be positive")
    }
    
    return nil
}

func (f *ConnectionFactory) buildDSN(config Config) (string, error) {
    switch config.Type {
    case PostgreSQL:
        return fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s connect_timeout=%d",
            config.Host, config.Port, config.Username, config.Password, 
            config.Database, config.SSLMode, int(config.ConnectTimeout.Seconds())), nil
    case MySQL:
        return fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?parseTime=true&timeout=%s",
            config.Username, config.Password, config.Host, config.Port, 
            config.Database, config.ConnectTimeout), nil
    case SQLite:
        return config.FilePath, nil
    default:
        return "", fmt.Errorf("unsupported database type: %s", config.Type)
    }
}

// Connection methods
func (c *Connection) DB() *sql.DB {
    return c.db
}

func (c *Connection) Config() Config {
    return c.config
}

func (c *Connection) Stats() ConnectionStats {
    dbStats := c.db.Stats()
    c.stats.TotalConnections = int64(dbStats.OpenConnections)
    c.stats.ActiveConnections = int32(dbStats.InUse)
    return c.stats
}

func (c *Connection) Close() error {
    return c.db.Close()
}

func (c *Connection) Ping() error {
    return c.db.Ping()
}

// Package-level factory functions for convenience
var defaultFactory = NewConnectionFactory()

func NewConnection(config Config) (*Connection, error) {
    return defaultFactory.CreateConnection(config)
}

func NewPostgreSQLConnection(host string, port int, database, username, password string) (*Connection, error) {
    return defaultFactory.CreatePostgreSQLConnection(host, port, database, username, password)
}

func NewMySQLConnection(host string, port int, database, username, password string) (*Connection, error) {
    return defaultFactory.CreateMySQLConnection(host, port, database, username, password)
}

func NewSQLiteConnection(filePath string) (*Connection, error) {
    return defaultFactory.CreateSQLiteConnection(filePath)
}

// Builder pattern for complex configurations
type ConfigBuilder struct {
    config Config
}

func NewConfigBuilder() *ConfigBuilder {
    return &ConfigBuilder{
        config: Config{
            Type:           PostgreSQL,
            Host:           "localhost",
            Port:           5432,
            SSLMode:        "disable",
            MaxConnections: 25,
            MaxIdleTime:    5 * time.Minute,
            MaxLifetime:    1 * time.Hour,
            ConnectTimeout: 10 * time.Second,
        },
    }
}

func (b *ConfigBuilder) WithType(dbType DatabaseType) *ConfigBuilder {
    b.config.Type = dbType
    return b
}

func (b *ConfigBuilder) WithHost(host string) *ConfigBuilder {
    b.config.Host = host
    return b
}

func (b *ConfigBuilder) WithPort(port int) *ConfigBuilder {
    b.config.Port = port
    return b
}

func (b *ConfigBuilder) WithDatabase(database string) *ConfigBuilder {
    b.config.Database = database
    return b
}

func (b *ConfigBuilder) WithCredentials(username, password string) *ConfigBuilder {
    b.config.Username = username
    b.config.Password = password
    return b
}

func (b *ConfigBuilder) WithSSLMode(sslMode string) *ConfigBuilder {
    b.config.SSLMode = sslMode
    return b
}

func (b *ConfigBuilder) WithPoolSettings(maxConns int, maxIdleTime, maxLifetime time.Duration) *ConfigBuilder {
    b.config.MaxConnections = maxConns
    b.config.MaxIdleTime = maxIdleTime
    b.config.MaxLifetime = maxLifetime
    return b
}

func (b *ConfigBuilder) WithTimeout(timeout time.Duration) *ConfigBuilder {
    b.config.ConnectTimeout = timeout
    return b
}

func (b *ConfigBuilder) WithFilePath(filePath string) *ConfigBuilder {
    b.config.FilePath = filePath
    return b
}

func (b *ConfigBuilder) Build() Config {
    return b.config
}

func (b *ConfigBuilder) Connect() (*Connection, error) {
    return NewConnection(b.config)
}

// main.go
package main

import (
    "fmt"
    "log"
    "time"
    "your-module/database"
)

func main() {
    fmt.Println("=== Database Factory Pattern Demo ===")
    
    // Method 1: Using factory directly
    factory := database.NewConnectionFactory()
    
    // Create PostgreSQL connection
    pgConfig := database.Config{
        Type:     database.PostgreSQL,
        Host:     "localhost",
        Port:     5432,
        Database: "testdb",
        Username: "postgres",
        Password: "password",
    }
    
    pgConn, err := factory.CreateConnection(pgConfig)
    if err != nil {
        log.Printf("PostgreSQL connection error: %v", err)
    } else {
        fmt.Println("PostgreSQL connection created successfully")
        fmt.Printf("Config: %+v\n", pgConn.Config())
        pgConn.Close()
    }
    
    // Method 2: Using convenience functions
    sqliteConn, err := database.NewSQLiteConnection("./test.db")
    if err != nil {
        log.Printf("SQLite connection error: %v", err)
    } else {
        fmt.Println("SQLite connection created successfully")
        fmt.Printf("Stats: %+v\n", sqliteConn.Stats())
        sqliteConn.Close()
    }
    
    // Method 3: Using builder pattern
    config := database.NewConfigBuilder().
        WithType(database.MySQL).
        WithHost("localhost").
        WithPort(3306).
        WithDatabase("myapp").
        WithCredentials("user", "password").
        WithPoolSettings(50, 10*time.Minute, 2*time.Hour).
        WithTimeout(15*time.Second).
        Build()
    
    fmt.Printf("Built config: %+v\n", config)
    
    // Method 4: Builder with direct connection
    conn, err := database.NewConfigBuilder().
        WithType(database.SQLite).
        WithFilePath("./app.db").
        WithPoolSettings(10, 5*time.Minute, 1*time.Hour).
        Connect()
    
    if err != nil {
        log.Printf("Builder connection error: %v", err)
    } else {
        fmt.Println("Builder connection created successfully")
        conn.Close()
    }
    
    // Demonstrate different database types
    fmt.Println("\n=== Testing Different Database Types ===")
    
    configs := []struct {
        name   string
        config database.Config
    }{
        {
            name: "PostgreSQL",
            config: database.Config{
                Type: database.PostgreSQL,
                Host: "localhost", Port: 5432,
                Database: "postgres", Username: "postgres", Password: "password",
            },
        },
        {
            name: "MySQL",
            config: database.Config{
                Type: database.MySQL,
                Host: "localhost", Port: 3306,
                Database: "mysql", Username: "root", Password: "password",
            },
        },
        {
            name: "SQLite",
            config: database.Config{
                Type:     database.SQLite,
                FilePath: "./test.db",
            },
        },
    }
    
    for _, tc := range configs {
        conn, err := database.NewConnection(tc.config)
        if err != nil {
            fmt.Printf("%s: Connection failed - %v\n", tc.name, err)
        } else {
            fmt.Printf("%s: Connection successful\n", tc.name)
            conn.Close()
        }
    }
}
```

Package factory patterns provide controlled object creation with validation,  
configuration management, and multiple creation strategies. Factories can  
combine with builder patterns for complex configurations and offer both  
convenience methods and full customization options for different use cases.  

## Package configuration patterns

Configuration patterns in packages provide structured ways to manage  
application settings, environment variables, and runtime parameters.  

```go
// config/types.go
package config

import (
    "encoding/json"
    "fmt"
    "time"
)

// Environment represents deployment environments
type Environment string

const (
    Development Environment = "development"
    Staging     Environment = "staging"
    Production  Environment = "production"
    Testing     Environment = "testing"
)

// LogLevel represents logging levels
type LogLevel string

const (
    DEBUG LogLevel = "debug"
    INFO  LogLevel = "info"
    WARN  LogLevel = "warn"
    ERROR LogLevel = "error"
)

// DatabaseConfig holds database configuration
type DatabaseConfig struct {
    Driver          string        `json:"driver" yaml:"driver" env:"DB_DRIVER"`
    Host            string        `json:"host" yaml:"host" env:"DB_HOST"`
    Port            int           `json:"port" yaml:"port" env:"DB_PORT"`
    Name            string        `json:"name" yaml:"name" env:"DB_NAME"`
    Username        string        `json:"username" yaml:"username" env:"DB_USERNAME"`
    Password        string        `json:"password" yaml:"password" env:"DB_PASSWORD"`
    SSLMode         string        `json:"ssl_mode" yaml:"ssl_mode" env:"DB_SSL_MODE"`
    MaxConnections  int           `json:"max_connections" yaml:"max_connections" env:"DB_MAX_CONNECTIONS"`
    MaxIdleTime     time.Duration `json:"max_idle_time" yaml:"max_idle_time" env:"DB_MAX_IDLE_TIME"`
    ConnectTimeout  time.Duration `json:"connect_timeout" yaml:"connect_timeout" env:"DB_CONNECT_TIMEOUT"`
    MigrationsPath  string        `json:"migrations_path" yaml:"migrations_path" env:"DB_MIGRATIONS_PATH"`
}

// ServerConfig holds HTTP server configuration
type ServerConfig struct {
    Host               string        `json:"host" yaml:"host" env:"SERVER_HOST"`
    Port               int           `json:"port" yaml:"port" env:"SERVER_PORT"`
    ReadTimeout        time.Duration `json:"read_timeout" yaml:"read_timeout" env:"SERVER_READ_TIMEOUT"`
    WriteTimeout       time.Duration `json:"write_timeout" yaml:"write_timeout" env:"SERVER_WRITE_TIMEOUT"`
    IdleTimeout        time.Duration `json:"idle_timeout" yaml:"idle_timeout" env:"SERVER_IDLE_TIMEOUT"`
    ShutdownTimeout    time.Duration `json:"shutdown_timeout" yaml:"shutdown_timeout" env:"SERVER_SHUTDOWN_TIMEOUT"`
    MaxHeaderBytes     int           `json:"max_header_bytes" yaml:"max_header_bytes" env:"SERVER_MAX_HEADER_BYTES"`
    EnableTLS          bool          `json:"enable_tls" yaml:"enable_tls" env:"SERVER_ENABLE_TLS"`
    TLSCertFile        string        `json:"tls_cert_file" yaml:"tls_cert_file" env:"SERVER_TLS_CERT_FILE"`
    TLSKeyFile         string        `json:"tls_key_file" yaml:"tls_key_file" env:"SERVER_TLS_KEY_FILE"`
    EnableProfiling    bool          `json:"enable_profiling" yaml:"enable_profiling" env:"SERVER_ENABLE_PROFILING"`
    EnableMetrics      bool          `json:"enable_metrics" yaml:"enable_metrics" env:"SERVER_ENABLE_METRICS"`
}

// RedisConfig holds Redis configuration
type RedisConfig struct {
    Host               string        `json:"host" yaml:"host" env:"REDIS_HOST"`
    Port               int           `json:"port" yaml:"port" env:"REDIS_PORT"`
    Password           string        `json:"password" yaml:"password" env:"REDIS_PASSWORD"`
    Database           int           `json:"database" yaml:"database" env:"REDIS_DATABASE"`
    MaxRetries         int           `json:"max_retries" yaml:"max_retries" env:"REDIS_MAX_RETRIES"`
    MinRetryBackoff    time.Duration `json:"min_retry_backoff" yaml:"min_retry_backoff" env:"REDIS_MIN_RETRY_BACKOFF"`
    MaxRetryBackoff    time.Duration `json:"max_retry_backoff" yaml:"max_retry_backoff" env:"REDIS_MAX_RETRY_BACKOFF"`
    DialTimeout        time.Duration `json:"dial_timeout" yaml:"dial_timeout" env:"REDIS_DIAL_TIMEOUT"`
    ReadTimeout        time.Duration `json:"read_timeout" yaml:"read_timeout" env:"REDIS_READ_TIMEOUT"`
    WriteTimeout       time.Duration `json:"write_timeout" yaml:"write_timeout" env:"REDIS_WRITE_TIMEOUT"`
    PoolSize           int           `json:"pool_size" yaml:"pool_size" env:"REDIS_POOL_SIZE"`
    MinIdleConns       int           `json:"min_idle_conns" yaml:"min_idle_conns" env:"REDIS_MIN_IDLE_CONNS"`
    MaxConnAge         time.Duration `json:"max_conn_age" yaml:"max_conn_age" env:"REDIS_MAX_CONN_AGE"`
    PoolTimeout        time.Duration `json:"pool_timeout" yaml:"pool_timeout" env:"REDIS_POOL_TIMEOUT"`
    IdleTimeout        time.Duration `json:"idle_timeout" yaml:"idle_timeout" env:"REDIS_IDLE_TIMEOUT"`
    IdleCheckFrequency time.Duration `json:"idle_check_frequency" yaml:"idle_check_frequency" env:"REDIS_IDLE_CHECK_FREQUENCY"`
}

// LoggingConfig holds logging configuration
type LoggingConfig struct {
    Level       LogLevel `json:"level" yaml:"level" env:"LOG_LEVEL"`
    Format      string   `json:"format" yaml:"format" env:"LOG_FORMAT"`
    Output      string   `json:"output" yaml:"output" env:"LOG_OUTPUT"`
    File        string   `json:"file" yaml:"file" env:"LOG_FILE"`
    MaxSize     int      `json:"max_size" yaml:"max_size" env:"LOG_MAX_SIZE"`
    MaxAge      int      `json:"max_age" yaml:"max_age" env:"LOG_MAX_AGE"`
    MaxBackups  int      `json:"max_backups" yaml:"max_backups" env:"LOG_MAX_BACKUPS"`
    Compress    bool     `json:"compress" yaml:"compress" env:"LOG_COMPRESS"`
    EnableCaller bool    `json:"enable_caller" yaml:"enable_caller" env:"LOG_ENABLE_CALLER"`
    EnableStacktrace bool `json:"enable_stacktrace" yaml:"enable_stacktrace" env:"LOG_ENABLE_STACKTRACE"`
}

// AuthConfig holds authentication configuration
type AuthConfig struct {
    JWTSecret           string        `json:"jwt_secret" yaml:"jwt_secret" env:"AUTH_JWT_SECRET"`
    JWTExpiration       time.Duration `json:"jwt_expiration" yaml:"jwt_expiration" env:"AUTH_JWT_EXPIRATION"`
    RefreshTokenExpiration time.Duration `json:"refresh_token_expiration" yaml:"refresh_token_expiration" env:"AUTH_REFRESH_TOKEN_EXPIRATION"`
    PasswordMinLength   int           `json:"password_min_length" yaml:"password_min_length" env:"AUTH_PASSWORD_MIN_LENGTH"`
    PasswordMaxLength   int           `json:"password_max_length" yaml:"password_max_length" env:"AUTH_PASSWORD_MAX_LENGTH"`
    PasswordRequireUppercase bool     `json:"password_require_uppercase" yaml:"password_require_uppercase" env:"AUTH_PASSWORD_REQUIRE_UPPERCASE"`
    PasswordRequireLowercase bool     `json:"password_require_lowercase" yaml:"password_require_lowercase" env:"AUTH_PASSWORD_REQUIRE_LOWERCASE"`
    PasswordRequireNumbers   bool     `json:"password_require_numbers" yaml:"password_require_numbers" env:"AUTH_PASSWORD_REQUIRE_NUMBERS"`
    PasswordRequireSymbols   bool     `json:"password_require_symbols" yaml:"password_require_symbols" env:"AUTH_PASSWORD_REQUIRE_SYMBOLS"`
    MaxLoginAttempts    int           `json:"max_login_attempts" yaml:"max_login_attempts" env:"AUTH_MAX_LOGIN_ATTEMPTS"`
    LockoutDuration     time.Duration `json:"lockout_duration" yaml:"lockout_duration" env:"AUTH_LOCKOUT_DURATION"`
    EnableTwoFactor     bool          `json:"enable_two_factor" yaml:"enable_two_factor" env:"AUTH_ENABLE_TWO_FACTOR"`
}

// ApplicationConfig is the main configuration structure
type ApplicationConfig struct {
    Name        string          `json:"name" yaml:"name" env:"APP_NAME"`
    Version     string          `json:"version" yaml:"version" env:"APP_VERSION"`
    Environment Environment     `json:"environment" yaml:"environment" env:"APP_ENVIRONMENT"`
    Debug       bool            `json:"debug" yaml:"debug" env:"APP_DEBUG"`
    Timezone    string          `json:"timezone" yaml:"timezone" env:"APP_TIMEZONE"`
    
    Database    DatabaseConfig  `json:"database" yaml:"database"`
    Server      ServerConfig    `json:"server" yaml:"server"`
    Redis       RedisConfig     `json:"redis" yaml:"redis"`
    Logging     LoggingConfig   `json:"logging" yaml:"logging"`
    Auth        AuthConfig      `json:"auth" yaml:"auth"`
    
    Features    map[string]bool `json:"features" yaml:"features"`
    Secrets     map[string]string `json:"-" yaml:"-"` // Never serialize secrets
}

// String returns a string representation hiding sensitive data
func (c *ApplicationConfig) String() string {
    // Create a copy for safe printing
    safe := *c
    safe.Database.Password = "***"
    safe.Redis.Password = "***"
    safe.Auth.JWTSecret = "***"
    safe.Secrets = map[string]string{"hidden": "***"}
    
    data, _ := json.MarshalIndent(safe, "", "  ")
    return string(data)
}

// GetDatabaseURL returns the database connection URL
func (c *ApplicationConfig) GetDatabaseURL() string {
    db := c.Database
    switch db.Driver {
    case "postgres":
        return fmt.Sprintf("postgres://%s:%s@%s:%d/%s?sslmode=%s",
            db.Username, db.Password, db.Host, db.Port, db.Name, db.SSLMode)
    case "mysql":
        return fmt.Sprintf("%s:%s@tcp(%s:%d)/%s",
            db.Username, db.Password, db.Host, db.Port, db.Name)
    case "sqlite":
        return db.Name
    default:
        return ""
    }
}

// GetRedisURL returns the Redis connection URL
func (c *ApplicationConfig) GetRedisURL() string {
    r := c.Redis
    if r.Password != "" {
        return fmt.Sprintf("redis://:%s@%s:%d/%d", r.Password, r.Host, r.Port, r.Database)
    }
    return fmt.Sprintf("redis://%s:%d/%d", r.Host, r.Port, r.Database)
}

// IsProduction returns true if running in production environment
func (c *ApplicationConfig) IsProduction() bool {
    return c.Environment == Production
}

// IsDevelopment returns true if running in development environment
func (c *ApplicationConfig) IsDevelopment() bool {
    return c.Environment == Development
}

// IsDebugEnabled returns true if debug mode is enabled
func (c *ApplicationConfig) IsDebugEnabled() bool {
    return c.Debug || c.Environment == Development
}

// IsFeatureEnabled returns true if a feature is enabled
func (c *ApplicationConfig) IsFeatureEnabled(feature string) bool {
    if c.Features == nil {
        return false
    }
    return c.Features[feature]
}

// SetFeature enables or disables a feature
func (c *ApplicationConfig) SetFeature(feature string, enabled bool) {
    if c.Features == nil {
        c.Features = make(map[string]bool)
    }
    c.Features[feature] = enabled
}

// GetSecret returns a secret value
func (c *ApplicationConfig) GetSecret(key string) string {
    if c.Secrets == nil {
        return ""
    }
    return c.Secrets[key]
}

// SetSecret sets a secret value
func (c *ApplicationConfig) SetSecret(key, value string) {
    if c.Secrets == nil {
        c.Secrets = make(map[string]string)
    }
    c.Secrets[key] = value
}

// Validate validates the configuration
func (c *ApplicationConfig) Validate() error {
    if c.Name == "" {
        return fmt.Errorf("application name is required")
    }
    if c.Version == "" {
        return fmt.Errorf("application version is required")
    }
    
    // Validate database config
    if c.Database.Driver == "" {
        return fmt.Errorf("database driver is required")
    }
    if c.Database.Driver != "sqlite" {
        if c.Database.Host == "" {
            return fmt.Errorf("database host is required")
        }
        if c.Database.Port <= 0 {
            return fmt.Errorf("database port must be positive")
        }
        if c.Database.Name == "" {
            return fmt.Errorf("database name is required")
        }
    }
    
    // Validate server config
    if c.Server.Port <= 0 || c.Server.Port > 65535 {
        return fmt.Errorf("server port must be between 1 and 65535")
    }
    if c.Server.ReadTimeout <= 0 {
        return fmt.Errorf("server read timeout must be positive")
    }
    if c.Server.WriteTimeout <= 0 {
        return fmt.Errorf("server write timeout must be positive")
    }
    
    // Validate auth config
    if c.Auth.JWTSecret == "" {
        return fmt.Errorf("JWT secret is required")
    }
    if len(c.Auth.JWTSecret) < 32 {
        return fmt.Errorf("JWT secret must be at least 32 characters")
    }
    if c.Auth.JWTExpiration <= 0 {
        return fmt.Errorf("JWT expiration must be positive")
    }
    
    return nil
}
```

Configuration patterns provide structured approaches to application settings  
management. They support multiple formats (JSON, YAML, environment variables),  
validation, type safety, and secure handling of sensitive data. Proper  
configuration design improves maintainability and deployment flexibility.  

## Package testing patterns

Package testing patterns provide comprehensive testing strategies including  
unit tests, integration tests, benchmarks, and test utilities.  

```go
// calculator/calculator.go
package calculator

import (
    "errors"
    "math"
)

// Calculator provides mathematical operations
type Calculator struct {
    precision int
    history   []Operation
}

// Operation represents a calculation operation
type Operation struct {
    Type   string
    A      float64
    B      float64
    Result float64
}

// New creates a new calculator with default precision
func New() *Calculator {
    return &Calculator{
        precision: 2,
        history:   make([]Operation, 0),
    }
}

// NewWithPrecision creates a calculator with custom precision
func NewWithPrecision(precision int) *Calculator {
    if precision < 0 {
        precision = 0
    }
    if precision > 10 {
        precision = 10
    }
    return &Calculator{
        precision: precision,
        history:   make([]Operation, 0),
    }
}

func (c *Calculator) addToHistory(op Operation) {
    c.history = append(c.history, op)
}

func (c *Calculator) roundResult(result float64) float64 {
    multiplier := math.Pow(10, float64(c.precision))
    return math.Round(result*multiplier) / multiplier
}

// Add performs addition
func (c *Calculator) Add(a, b float64) float64 {
    result := c.roundResult(a + b)
    c.addToHistory(Operation{Type: "add", A: a, B: b, Result: result})
    return result
}

// Subtract performs subtraction
func (c *Calculator) Subtract(a, b float64) float64 {
    result := c.roundResult(a - b)
    c.addToHistory(Operation{Type: "subtract", A: a, B: b, Result: result})
    return result
}

// Multiply performs multiplication
func (c *Calculator) Multiply(a, b float64) float64 {
    result := c.roundResult(a * b)
    c.addToHistory(Operation{Type: "multiply", A: a, B: b, Result: result})
    return result
}

// Divide performs division
func (c *Calculator) Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    result := c.roundResult(a / b)
    c.addToHistory(Operation{Type: "divide", A: a, B: b, Result: result})
    return result, nil
}

// Power raises a to the power of b
func (c *Calculator) Power(a, b float64) float64 {
    result := c.roundResult(math.Pow(a, b))
    c.addToHistory(Operation{Type: "power", A: a, B: b, Result: result})
    return result
}

// Sqrt calculates square root
func (c *Calculator) Sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, errors.New("square root of negative number")
    }
    result := c.roundResult(math.Sqrt(x))
    c.addToHistory(Operation{Type: "sqrt", A: x, B: 0, Result: result})
    return result, nil
}

// GetHistory returns calculation history
func (c *Calculator) GetHistory() []Operation {
    history := make([]Operation, len(c.history))
    copy(history, c.history)
    return history
}

// ClearHistory clears calculation history
func (c *Calculator) ClearHistory() {
    c.history = c.history[:0]
}

// GetPrecision returns current precision setting
func (c *Calculator) GetPrecision() int {
    return c.precision
}

// SetPrecision updates precision setting
func (c *Calculator) SetPrecision(precision int) {
    if precision >= 0 && precision <= 10 {
        c.precision = precision
    }
}

// calculator/calculator_test.go
package calculator

import (
    "math"
    "testing"
)

func TestNew(t *testing.T) {
    calc := New()
    if calc == nil {
        t.Fatal("New() returned nil")
    }
    if calc.precision != 2 {
        t.Errorf("Expected precision 2, got %d", calc.precision)
    }
    if len(calc.history) != 0 {
        t.Errorf("Expected empty history, got %d items", len(calc.history))
    }
}

func TestNewWithPrecision(t *testing.T) {
    tests := []struct {
        name      string
        precision int
        expected  int
    }{
        {"Valid precision", 4, 4},
        {"Negative precision", -1, 0},
        {"Too high precision", 15, 10},
        {"Zero precision", 0, 0},
        {"Max precision", 10, 10},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            calc := NewWithPrecision(tt.precision)
            if calc.precision != tt.expected {
                t.Errorf("Expected precision %d, got %d", tt.expected, calc.precision)
            }
        })
    }
}

func TestAdd(t *testing.T) {
    calc := New()
    
    tests := []struct {
        name     string
        a, b     float64
        expected float64
    }{
        {"Positive numbers", 5.5, 2.3, 7.8},
        {"Negative numbers", -3.2, -1.8, -5.0},
        {"Mixed signs", 10.0, -3.0, 7.0},
        {"Zeros", 0.0, 0.0, 0.0},
        {"Large numbers", 1e6, 1e6, 2e6},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := calc.Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%f, %f) = %f, expected %f", tt.a, tt.b, result, tt.expected)
            }
        })
    }
    
    // Verify history
    history := calc.GetHistory()
    if len(history) != len(tests) {
        t.Errorf("Expected %d operations in history, got %d", len(tests), len(history))
    }
}

func TestDivide(t *testing.T) {
    calc := New()
    
    // Test normal division
    result, err := calc.Divide(10.0, 2.0)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 5.0 {
        t.Errorf("Expected 5.0, got %f", result)
    }
    
    // Test division by zero
    _, err = calc.Divide(10.0, 0.0)
    if err == nil {
        t.Error("Expected error for division by zero")
    }
    if err.Error() != "division by zero" {
        t.Errorf("Expected 'division by zero', got '%s'", err.Error())
    }
}

func TestSqrt(t *testing.T) {
    calc := New()
    
    // Test positive number
    result, err := calc.Sqrt(25.0)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 5.0 {
        t.Errorf("Expected 5.0, got %f", result)
    }
    
    // Test negative number
    _, err = calc.Sqrt(-25.0)
    if err == nil {
        t.Error("Expected error for square root of negative number")
    }
    
    // Test zero
    result, err = calc.Sqrt(0.0)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 0.0 {
        t.Errorf("Expected 0.0, got %f", result)
    }
}

func TestPrecision(t *testing.T) {
    calc := NewWithPrecision(3)
    
    result := calc.Add(1.2345, 2.6789)
    expected := 3.913 // Rounded to 3 decimal places
    
    if result != expected {
        t.Errorf("Expected %f, got %f", expected, result)
    }
}

func TestHistory(t *testing.T) {
    calc := New()
    
    // Perform operations
    calc.Add(5, 3)
    calc.Multiply(2, 4)
    calc.Subtract(10, 6)
    
    history := calc.GetHistory()
    if len(history) != 3 {
        t.Errorf("Expected 3 operations, got %d", len(history))
    }
    
    // Check first operation
    if history[0].Type != "add" || history[0].A != 5 || history[0].B != 3 || history[0].Result != 8 {
        t.Errorf("First operation incorrect: %+v", history[0])
    }
    
    // Clear history
    calc.ClearHistory()
    history = calc.GetHistory()
    if len(history) != 0 {
        t.Errorf("Expected empty history after clear, got %d items", len(history))
    }
}

// Test helper functions
func assertFloat64Equal(t *testing.T, expected, actual float64) {
    t.Helper()
    if math.Abs(expected-actual) > 1e-10 {
        t.Errorf("Expected %f, got %f", expected, actual)
    }
}

func assertError(t *testing.T, err error, expectedMsg string) {
    t.Helper()
    if err == nil {
        t.Errorf("Expected error with message '%s', got nil", expectedMsg)
        return
    }
    if err.Error() != expectedMsg {
        t.Errorf("Expected error message '%s', got '%s'", expectedMsg, err.Error())
    }
}

func TestWithHelpers(t *testing.T) {
    calc := New()
    
    result := calc.Add(0.1, 0.2)
    assertFloat64Equal(t, 0.3, result)
    
    _, err := calc.Divide(1, 0)
    assertError(t, err, "division by zero")
}

// Benchmark tests
func BenchmarkAdd(b *testing.B) {
    calc := New()
    for i := 0; i < b.N; i++ {
        calc.Add(float64(i), float64(i+1))
    }
}

func BenchmarkMultiply(b *testing.B) {
    calc := New()
    for i := 0; i < b.N; i++ {
        calc.Multiply(float64(i), 2.5)
    }
}

func BenchmarkPower(b *testing.B) {
    calc := New()
    for i := 0; i < b.N; i++ {
        calc.Power(2.0, float64(i%10))
    }
}

// Example tests (documentation examples)
func ExampleCalculator_Add() {
    calc := New()
    result := calc.Add(5.5, 2.3)
    fmt.Printf("%.1f", result)
    // Output: 7.8
}

func ExampleCalculator_Divide() {
    calc := New()
    result, err := calc.Divide(10.0, 2.0)
    if err != nil {
        fmt.Printf("Error: %v", err)
    } else {
        fmt.Printf("%.1f", result)
    }
    // Output: 5.0
}

func ExampleNew() {
    calc := New()
    fmt.Printf("Precision: %d", calc.GetPrecision())
    // Output: Precision: 2
}

// Table-driven tests
func TestCalculatorOperations(t *testing.T) {
    calc := New()
    
    tests := []struct {
        name      string
        operation func() (float64, error)
        expected  float64
        shouldErr bool
    }{
        {
            name:      "Add positive",
            operation: func() (float64, error) { return calc.Add(5, 3), nil },
            expected:  8.0,
            shouldErr: false,
        },
        {
            name:      "Subtract",
            operation: func() (float64, error) { return calc.Subtract(10, 4), nil },
            expected:  6.0,
            shouldErr: false,
        },
        {
            name:      "Multiply",
            operation: func() (float64, error) { return calc.Multiply(3, 4), nil },
            expected:  12.0,
            shouldErr: false,
        },
        {
            name:      "Divide valid",
            operation: func() (float64, error) { return calc.Divide(15, 3) },
            expected:  5.0,
            shouldErr: false,
        },
        {
            name:      "Divide by zero",
            operation: func() (float64, error) { return calc.Divide(5, 0) },
            expected:  0.0,
            shouldErr: true,
        },
        {
            name:      "Square root positive",
            operation: func() (float64, error) { return calc.Sqrt(16) },
            expected:  4.0,
            shouldErr: false,
        },
        {
            name:      "Square root negative",
            operation: func() (float64, error) { return calc.Sqrt(-4) },
            expected:  0.0,
            shouldErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := tt.operation()
            
            if tt.shouldErr && err == nil {
                t.Error("Expected error but got none")
            }
            if !tt.shouldErr && err != nil {
                t.Errorf("Unexpected error: %v", err)
            }
            if !tt.shouldErr && result != tt.expected {
                t.Errorf("Expected %f, got %f", tt.expected, result)
            }
        })
    }
}

// Property-based testing example
func TestCalculatorProperties(t *testing.T) {
    calc := New()
    
    // Test commutativity of addition
    for i := 0; i < 100; i++ {
        a := float64(i)
        b := float64(i + 1)
        
        calc.ClearHistory()
        result1 := calc.Add(a, b)
        
        calc.ClearHistory()
        result2 := calc.Add(b, a)
        
        if result1 != result2 {
            t.Errorf("Addition not commutative: %f + %f = %f, but %f + %f = %f",
                a, b, result1, b, a, result2)
        }
    }
    
    // Test associativity of addition
    for i := 0; i < 50; i++ {
        a := float64(i)
        b := float64(i + 1)
        c := float64(i + 2)
        
        calc.ClearHistory()
        temp1 := calc.Add(a, b)
        result1 := calc.Add(temp1, c)
        
        calc.ClearHistory()
        temp2 := calc.Add(b, c)
        result2 := calc.Add(a, temp2)
        
        if math.Abs(result1-result2) > 1e-10 {
            t.Errorf("Addition not associative: (%f + %f) + %f = %f, but %f + (%f + %f) = %f",
                a, b, c, result1, a, b, c, result2)
        }
    }
}

// Fuzz testing example (Go 1.18+)
/*
func FuzzCalculatorAdd(f *testing.F) {
    calc := New()
    
    // Add seed values
    f.Add(1.0, 2.0)
    f.Add(-1.0, 1.0)
    f.Add(0.0, 0.0)
    
    f.Fuzz(func(t *testing.T, a, b float64) {
        // Skip infinite and NaN values
        if math.IsInf(a, 0) || math.IsInf(b, 0) || math.IsNaN(a) || math.IsNaN(b) {
            return
        }
        
        result := calc.Add(a, b)
        
        // Basic property: result should be finite
        if math.IsInf(result, 0) || math.IsNaN(result) {
            t.Errorf("Add(%f, %f) produced invalid result: %f", a, b, result)
        }
        
        // Property: adding zero should return the other operand (with precision rounding)
        if b == 0.0 {
            expected := calc.roundResult(a)
            if result != expected {
                t.Errorf("Add(%f, 0) = %f, expected %f", a, result, expected)
            }
        }
    })
}
*/
```

Package testing patterns encompass unit tests, table-driven tests, benchmarks,  
example tests, and property-based testing. Comprehensive testing ensures  
reliability, documents behavior through examples, and measures performance  
through benchmarks. Test helpers and utilities improve test maintainability.  

## Package benchmarking

Benchmarking provides performance measurement and optimization guidance  
for package functions and methods.  

```go
// sort/algorithms.go
package sort

import (
    "math/rand"
    "time"
)

// BubbleSort implements bubble sort algorithm
func BubbleSort(arr []int) []int {
    result := make([]int, len(arr))
    copy(result, arr)
    
    n := len(result)
    for i := 0; i < n; i++ {
        for j := 0; j < n-i-1; j++ {
            if result[j] > result[j+1] {
                result[j], result[j+1] = result[j+1], result[j]
            }
        }
    }
    return result
}

// SelectionSort implements selection sort algorithm
func SelectionSort(arr []int) []int {
    result := make([]int, len(arr))
    copy(result, arr)
    
    n := len(result)
    for i := 0; i < n-1; i++ {
        minIdx := i
        for j := i + 1; j < n; j++ {
            if result[j] < result[minIdx] {
                minIdx = j
            }
        }
        result[i], result[minIdx] = result[minIdx], result[i]
    }
    return result
}

// InsertionSort implements insertion sort algorithm
func InsertionSort(arr []int) []int {
    result := make([]int, len(arr))
    copy(result, arr)
    
    for i := 1; i < len(result); i++ {
        key := result[i]
        j := i - 1
        
        for j >= 0 && result[j] > key {
            result[j+1] = result[j]
            j--
        }
        result[j+1] = key
    }
    return result
}

// QuickSort implements quicksort algorithm
func QuickSort(arr []int) []int {
    result := make([]int, len(arr))
    copy(result, arr)
    quickSortRecursive(result, 0, len(result)-1)
    return result
}

func quickSortRecursive(arr []int, low, high int) {
    if low < high {
        pi := partition(arr, low, high)
        quickSortRecursive(arr, low, pi-1)
        quickSortRecursive(arr, pi+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high]
    i := low - 1
    
    for j := low; j < high; j++ {
        if arr[j] < pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}

// MergeSort implements merge sort algorithm
func MergeSort(arr []int) []int {
    if len(arr) <= 1 {
        result := make([]int, len(arr))
        copy(result, arr)
        return result
    }
    
    mid := len(arr) / 2
    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])
    
    return merge(left, right)
}

func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    
    return result
}

// HeapSort implements heap sort algorithm
func HeapSort(arr []int) []int {
    result := make([]int, len(arr))
    copy(result, arr)
    
    n := len(result)
    
    // Build max heap
    for i := n/2 - 1; i >= 0; i-- {
        heapify(result, n, i)
    }
    
    // Extract elements from heap
    for i := n - 1; i > 0; i-- {
        result[0], result[i] = result[i], result[0]
        heapify(result, i, 0)
    }
    
    return result
}

func heapify(arr []int, n, i int) {
    largest := i
    left := 2*i + 1
    right := 2*i + 2
    
    if left < n && arr[left] > arr[largest] {
        largest = left
    }
    
    if right < n && arr[right] > arr[largest] {
        largest = right
    }
    
    if largest != i {
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
    }
}

// Helper functions for benchmarking
func GenerateRandomArray(size int) []int {
    rand.Seed(time.Now().UnixNano())
    arr := make([]int, size)
    for i := range arr {
        arr[i] = rand.Intn(1000)
    }
    return arr
}

func GenerateSortedArray(size int) []int {
    arr := make([]int, size)
    for i := range arr {
        arr[i] = i
    }
    return arr
}

func GenerateReverseSortedArray(size int) []int {
    arr := make([]int, size)
    for i := range arr {
        arr[i] = size - i
    }
    return arr
}

func IsSorted(arr []int) bool {
    for i := 1; i < len(arr); i++ {
        if arr[i] < arr[i-1] {
            return false
        }
    }
    return true
}

// sort/algorithms_bench_test.go
package sort

import (
    "math/rand"
    "testing"
    "time"
)

// Basic benchmarks for each algorithm
func BenchmarkBubbleSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                BubbleSort(arr)
            }
        })
    }
}

func BenchmarkSelectionSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                SelectionSort(arr)
            }
        })
    }
}

func BenchmarkInsertionSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500, 1000}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                InsertionSort(arr)
            }
        })
    }
}

func BenchmarkQuickSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500, 1000, 5000}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                QuickSort(arr)
            }
        })
    }
}

func BenchmarkMergeSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500, 1000, 5000}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                MergeSort(arr)
            }
        })
    }
}

func BenchmarkHeapSort(b *testing.B) {
    sizes := []int{10, 50, 100, 500, 1000, 5000}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("Size_%d", size), func(b *testing.B) {
            arr := GenerateRandomArray(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                HeapSort(arr)
            }
        })
    }
}

// Benchmarks with different data patterns
func BenchmarkSortAlgorithms(b *testing.B) {
    algorithms := map[string]func([]int) []int{
        "BubbleSort":    BubbleSort,
        "SelectionSort": SelectionSort,
        "InsertionSort": InsertionSort,
        "QuickSort":     QuickSort,
        "MergeSort":     MergeSort,
        "HeapSort":      HeapSort,
    }
    
    dataTypes := map[string]func(int) []int{
        "Random":       GenerateRandomArray,
        "Sorted":       GenerateSortedArray,
        "ReverseSorted": GenerateReverseSortedArray,
    }
    
    sizes := []int{100, 1000}
    
    for algName, algFunc := range algorithms {
        for dataName, dataFunc := range dataTypes {
            for _, size := range sizes {
                name := fmt.Sprintf("%s_%s_%d", algName, dataName, size)
                b.Run(name, func(b *testing.B) {
                    arr := dataFunc(size)
                    b.ResetTimer()
                    
                    for i := 0; i < b.N; i++ {
                        b.StopTimer()
                        testArr := make([]int, len(arr))
                        copy(testArr, arr)
                        b.StartTimer()
                        
                        algFunc(testArr)
                    }
                })
            }
        }
    }
}

// Memory allocation benchmarks
func BenchmarkSortMemoryUsage(b *testing.B) {
    size := 1000
    arr := GenerateRandomArray(size)
    
    b.Run("QuickSort", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            QuickSort(arr)
        }
    })
    
    b.Run("MergeSort", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            MergeSort(arr)
        }
    })
    
    b.Run("HeapSort", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            HeapSort(arr)
        }
    })
}

// Parallel benchmarks
func BenchmarkParallelQuickSort(b *testing.B) {
    size := 10000
    arr := GenerateRandomArray(size)
    
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            QuickSort(arr)
        }
    })
}

func BenchmarkParallelMergeSort(b *testing.B) {
    size := 10000
    arr := GenerateRandomArray(size)
    
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            MergeSort(arr)
        }
    })
}

// Custom benchmark with setup and teardown
func BenchmarkWithSetup(b *testing.B) {
    // Setup phase (not measured)
    rand.Seed(42) // Fixed seed for reproducible results
    
    for i := 0; i < b.N; i++ {
        b.StopTimer()
        // Setup for this iteration
        arr := GenerateRandomArray(1000)
        b.StartTimer()
        
        // The actual operation being measured
        QuickSort(arr)
    }
}

// Benchmark with custom metrics
func BenchmarkSortWithCustomMetrics(b *testing.B) {
    size := 1000
    arr := GenerateRandomArray(size)
    
    b.Run("QuickSort", func(b *testing.B) {
        var totalComparisons int64
        var totalSwaps int64
        
        for i := 0; i < b.N; i++ {
            testArr := make([]int, len(arr))
            copy(testArr, arr)
            
            // In a real implementation, you'd modify the sort function
            // to count comparisons and swaps
            QuickSort(testArr)
            
            // Simulated metrics
            totalComparisons += int64(size * 10)
            totalSwaps += int64(size * 5)
        }
        
        b.ReportMetric(float64(totalComparisons)/float64(b.N), "comparisons/op")
        b.ReportMetric(float64(totalSwaps)/float64(b.N), "swaps/op")
    })
}

// Benchmark helper functions
func benchmarkSort(b *testing.B, sortFunc func([]int) []int, size int) {
    arr := GenerateRandomArray(size)
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        sortFunc(arr)
    }
}

func BenchmarkAllAlgorithms(b *testing.B) {
    algorithms := []struct {
        name string
        fn   func([]int) []int
    }{
        {"BubbleSort", BubbleSort},
        {"SelectionSort", SelectionSort},
        {"InsertionSort", InsertionSort},
        {"QuickSort", QuickSort},
        {"MergeSort", MergeSort},
        {"HeapSort", HeapSort},
    }
    
    for _, alg := range algorithms {
        b.Run(alg.name, func(b *testing.B) {
            benchmarkSort(b, alg.fn, 1000)
        })
    }
}

// Comparative benchmark
func BenchmarkSortComparison(b *testing.B) {
    size := 1000
    
    b.Run("Go_builtin_sort", func(b *testing.B) {
        arr := GenerateRandomArray(size)
        b.ResetTimer()
        
        for i := 0; i < b.N; i++ {
            testArr := make([]int, len(arr))
            copy(testArr, arr)
            sort.Ints(testArr)
        }
    })
    
    b.Run("Custom_QuickSort", func(b *testing.B) {
        benchmarkSort(b, QuickSort, size)
    })
    
    b.Run("Custom_MergeSort", func(b *testing.B) {
        benchmarkSort(b, MergeSort, size)
    })
}

/*
Running benchmarks:

Basic benchmark:
go test -bench=.

Specific benchmark:
go test -bench=BenchmarkQuickSort

With memory profiling:
go test -bench=. -benchmem

Save results for comparison:
go test -bench=. > old.txt
# Make changes
go test -bench=. > new.txt
benchcmp old.txt new.txt

CPU profiling:
go test -bench=BenchmarkQuickSort -cpuprofile=cpu.prof

Memory profiling:
go test -bench=BenchmarkQuickSort -memprofile=mem.prof

View profiles:
go tool pprof cpu.prof
go tool pprof mem.prof

Run benchmarks multiple times for statistical significance:
go test -bench=BenchmarkQuickSort -count=5

Set benchmark time:
go test -bench=BenchmarkQuickSort -benchtime=10s
*/
```

Package benchmarking provides performance measurement, comparison, and  
optimization guidance. Benchmarks should cover different input sizes,  
data patterns, and usage scenarios. Memory profiling and custom metrics  
help identify bottlenecks and guide performance improvements.  

## Package profiling integration

Profiling integration enables runtime performance analysis and  
optimization through pprof and other profiling tools.  

```go
// profiler/profiler.go
package profiler

import (
    "context"
    "fmt"
    "log"
    "net/http"
    _ "net/http/pprof"
    "os"
    "runtime"
    "runtime/pprof"
    "sync"
    "time"
)

// ProfileConfig holds profiling configuration
type ProfileConfig struct {
    CPUProfile    string        `json:"cpu_profile"`
    MemProfile    string        `json:"mem_profile"`
    BlockProfile  string        `json:"block_profile"`
    MutexProfile  string        `json:"mutex_profile"`
    TraceFile     string        `json:"trace_file"`
    HTTPAddr      string        `json:"http_addr"`
    Duration      time.Duration `json:"duration"`
    EnableGC      bool          `json:"enable_gc"`
}

// Profiler manages application profiling
type Profiler struct {
    config       ProfileConfig
    cpuFile      *os.File
    memFile      *os.File
    blockFile    *os.File
    mutexFile    *os.File
    traceFile    *os.File
    httpServer   *http.Server
    isRunning    bool
    stopChan     chan struct{}
    mu           sync.RWMutex
}

// NewProfiler creates a new profiler instance
func NewProfiler(config ProfileConfig) *Profiler {
    return &Profiler{
        config:   config,
        stopChan: make(chan struct{}),
    }
}

// Start begins profiling with the configured options
func (p *Profiler) Start() error {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if p.isRunning {
        return fmt.Errorf("profiler is already running")
    }
    
    // Start CPU profiling
    if p.config.CPUProfile != "" {
        if err := p.startCPUProfiling(); err != nil {
            return fmt.Errorf("failed to start CPU profiling: %w", err)
        }
    }
    
    // Enable memory profiling
    if p.config.MemProfile != "" {
        runtime.MemProfileRate = 1
    }
    
    // Enable block profiling
    if p.config.BlockProfile != "" {
        runtime.SetBlockProfileRate(1)
    }
    
    // Enable mutex profiling
    if p.config.MutexProfile != "" {
        runtime.SetMutexProfileFraction(1)
    }
    
    // Start HTTP profiling server
    if p.config.HTTPAddr != "" {
        if err := p.startHTTPProfiling(); err != nil {
            return fmt.Errorf("failed to start HTTP profiling: %w", err)
        }
    }
    
    p.isRunning = true
    
    // Auto-stop after duration if specified
    if p.config.Duration > 0 {
        go func() {
            select {
            case <-time.After(p.config.Duration):
                p.Stop()
            case <-p.stopChan:
                return
            }
        }()
    }
    
    log.Println("Profiling started")
    return nil
}

// Stop ends profiling and saves profiles
func (p *Profiler) Stop() error {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if !p.isRunning {
        return fmt.Errorf("profiler is not running")
    }
    
    close(p.stopChan)
    p.isRunning = false
    
    // Stop CPU profiling
    if p.cpuFile != nil {
        pprof.StopCPUProfile()
        p.cpuFile.Close()
        log.Printf("CPU profile saved to %s", p.config.CPUProfile)
    }
    
    // Save memory profile
    if p.config.MemProfile != "" {
        if err := p.saveMemProfile(); err != nil {
            log.Printf("Failed to save memory profile: %v", err)
        }
    }
    
    // Save block profile
    if p.config.BlockProfile != "" {
        if err := p.saveBlockProfile(); err != nil {
            log.Printf("Failed to save block profile: %v", err)
        }
    }
    
    // Save mutex profile
    if p.config.MutexProfile != "" {
        if err := p.saveMutexProfile(); err != nil {
            log.Printf("Failed to save mutex profile: %v", err)
        }
    }
    
    // Stop HTTP server
    if p.httpServer != nil {
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        if err := p.httpServer.Shutdown(ctx); err != nil {
            log.Printf("HTTP profiling server shutdown error: %v", err)
        }
    }
    
    log.Println("Profiling stopped")
    return nil
}

// IsRunning returns true if profiling is active
func (p *Profiler) IsRunning() bool {
    p.mu.RLock()
    defer p.mu.RUnlock()
    return p.isRunning
}

// GetRuntimeStats returns current runtime statistics
func (p *Profiler) GetRuntimeStats() RuntimeStats {
    var stats RuntimeStats
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    stats.Goroutines = runtime.NumGoroutine()
    stats.CGOCalls = runtime.NumCgoCall()
    stats.Alloc = m.Alloc
    stats.TotalAlloc = m.TotalAlloc
    stats.Sys = m.Sys
    stats.Lookups = m.Lookups
    stats.Mallocs = m.Mallocs
    stats.Frees = m.Frees
    stats.HeapAlloc = m.HeapAlloc
    stats.HeapSys = m.HeapSys
    stats.HeapIdle = m.HeapIdle
    stats.HeapInuse = m.HeapInuse
    stats.StackInuse = m.StackInuse
    stats.StackSys = m.StackSys
    stats.MSpanInuse = m.MSpanInuse
    stats.MSpanSys = m.MSpanSys
    stats.MCacheInuse = m.MCacheInuse
    stats.MCacheSys = m.MCacheSys
    stats.GCSys = m.GCSys
    stats.NextGC = m.NextGC
    stats.LastGC = m.LastGC
    stats.PauseTotalNs = m.PauseTotalNs
    stats.NumGC = m.NumGC
    stats.NumForcedGC = m.NumForcedGC
    stats.GCCPUFraction = m.GCCPUFraction
    
    return stats
}

// PrintRuntimeStats prints current runtime statistics
func (p *Profiler) PrintRuntimeStats() {
    stats := p.GetRuntimeStats()
    fmt.Printf("Runtime Statistics:\n")
    fmt.Printf("  Goroutines: %d\n", stats.Goroutines)
    fmt.Printf("  CGO Calls: %d\n", stats.CGOCalls)
    fmt.Printf("  Memory Allocated: %s\n", formatBytes(stats.Alloc))
    fmt.Printf("  Total Allocated: %s\n", formatBytes(stats.TotalAlloc))
    fmt.Printf("  System Memory: %s\n", formatBytes(stats.Sys))
    fmt.Printf("  Heap Allocated: %s\n", formatBytes(stats.HeapAlloc))
    fmt.Printf("  Heap System: %s\n", formatBytes(stats.HeapSys))
    fmt.Printf("  Heap In Use: %s\n", formatBytes(stats.HeapInuse))
    fmt.Printf("  Stack In Use: %s\n", formatBytes(stats.StackInuse))
    fmt.Printf("  GC Cycles: %d\n", stats.NumGC)
    fmt.Printf("  GC CPU Fraction: %.4f\n", stats.GCCPUFraction)
    fmt.Printf("  Last GC: %v\n", time.Unix(0, int64(stats.LastGC)))
}

// RuntimeStats holds runtime statistics
type RuntimeStats struct {
    Goroutines     int
    CGOCalls       int64
    Alloc          uint64
    TotalAlloc     uint64
    Sys            uint64
    Lookups        uint64
    Mallocs        uint64
    Frees          uint64
    HeapAlloc      uint64
    HeapSys        uint64
    HeapIdle       uint64
    HeapInuse      uint64
    StackInuse     uint64
    StackSys       uint64
    MSpanInuse     uint64
    MSpanSys       uint64
    MCacheInuse    uint64
    MCacheSys      uint64
    GCSys          uint64
    NextGC         uint64
    LastGC         uint64
    PauseTotalNs   uint64
    NumGC          uint32
    NumForcedGC    uint32
    GCCPUFraction  float64
}

func (p *Profiler) startCPUProfiling() error {
    f, err := os.Create(p.config.CPUProfile)
    if err != nil {
        return err
    }
    p.cpuFile = f
    
    if err := pprof.StartCPUProfile(f); err != nil {
        f.Close()
        return err
    }
    
    return nil
}

func (p *Profiler) saveMemProfile() error {
    if p.config.EnableGC {
        runtime.GC()
    }
    
    f, err := os.Create(p.config.MemProfile)
    if err != nil {
        return err
    }
    defer f.Close()
    
    return pprof.WriteHeapProfile(f)
}

func (p *Profiler) saveBlockProfile() error {
    f, err := os.Create(p.config.BlockProfile)
    if err != nil {
        return err
    }
    defer f.Close()
    
    profile := pprof.Lookup("block")
    return profile.WriteTo(f, 0)
}

func (p *Profiler) saveMutexProfile() error {
    f, err := os.Create(p.config.MutexProfile)
    if err != nil {
        return err
    }
    defer f.Close()
    
    profile := pprof.Lookup("mutex")
    return profile.WriteTo(f, 0)
}

func (p *Profiler) startHTTPProfiling() error {
    mux := http.NewServeMux()
    
    // Add custom profiling endpoints
    mux.HandleFunc("/debug/pprof/", http.HandlerFunc(pprof.Index))
    mux.HandleFunc("/debug/pprof/cmdline", http.HandlerFunc(pprof.Cmdline))
    mux.HandleFunc("/debug/pprof/profile", http.HandlerFunc(pprof.Profile))
    mux.HandleFunc("/debug/pprof/symbol", http.HandlerFunc(pprof.Symbol))
    mux.HandleFunc("/debug/pprof/trace", http.HandlerFunc(pprof.Trace))
    
    // Add custom runtime stats endpoint
    mux.HandleFunc("/debug/stats", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        stats := p.GetRuntimeStats()
        fmt.Fprintf(w, `{
            "goroutines": %d,
            "cgo_calls": %d,
            "memory_allocated": %d,
            "total_allocated": %d,
            "system_memory": %d,
            "heap_allocated": %d,
            "heap_in_use": %d,
            "stack_in_use": %d,
            "gc_cycles": %d,
            "gc_cpu_fraction": %.6f
        }`, stats.Goroutines, stats.CGOCalls, stats.Alloc, stats.TotalAlloc,
            stats.Sys, stats.HeapAlloc, stats.HeapInuse, stats.StackInuse,
            stats.NumGC, stats.GCCPUFraction)
    })
    
    p.httpServer = &http.Server{
        Addr:    p.config.HTTPAddr,
        Handler: mux,
    }
    
    go func() {
        log.Printf("HTTP profiling server starting on %s", p.config.HTTPAddr)
        if err := p.httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Printf("HTTP profiling server error: %v", err)
        }
    }()
    
    return nil
}

func formatBytes(b uint64) string {
    const unit = 1024
    if b < unit {
        return fmt.Sprintf("%d B", b)
    }
    div, exp := int64(unit), 0
    for n := b / unit; n >= unit; n /= unit {
        div *= unit
        exp++
    }
    return fmt.Sprintf("%.1f %ciB",
        float64(b)/float64(div), "KMGTPE"[exp])
}

// Package-level convenience functions
var defaultProfiler *Profiler

// StartProfiling starts profiling with the given configuration
func StartProfiling(config ProfileConfig) error {
    defaultProfiler = NewProfiler(config)
    return defaultProfiler.Start()
}

// StopProfiling stops the default profiler
func StopProfiling() error {
    if defaultProfiler == nil {
        return fmt.Errorf("no active profiler")
    }
    return defaultProfiler.Stop()
}

// PrintStats prints runtime statistics
func PrintStats() {
    if defaultProfiler == nil {
        defaultProfiler = NewProfiler(ProfileConfig{})
    }
    defaultProfiler.PrintRuntimeStats()
}

// Example usage in main application
func main() {
    // Configure profiling
    config := ProfileConfig{
        CPUProfile:   "cpu.prof",
        MemProfile:   "mem.prof",
        BlockProfile: "block.prof",
        HTTPAddr:     ":6060",
        Duration:     30 * time.Second,
        EnableGC:     true,
    }
    
    // Start profiling
    profiler := NewProfiler(config)
    if err := profiler.Start(); err != nil {
        log.Fatal("Failed to start profiling:", err)
    }
    
    // Your application code here
    simulateWork()
    
    // Print stats
    profiler.PrintRuntimeStats()
    
    // Stop profiling
    if err := profiler.Stop(); err != nil {
        log.Printf("Failed to stop profiling: %v", err)
    }
}

func simulateWork() {
    // Simulate CPU-intensive work
    for i := 0; i < 1000000; i++ {
        _ = fibonacci(30)
    }
    
    // Simulate memory allocations
    data := make([][]byte, 1000)
    for i := range data {
        data[i] = make([]byte, 1024)
        for j := range data[i] {
            data[i][j] = byte(i + j)
        }
    }
    
    // Simulate goroutine blocking
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            time.Sleep(100 * time.Millisecond)
        }()
    }
    wg.Wait()
}

func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

/*
Using the profiler:

1. CPU Profile:
   go tool pprof cpu.prof
   (pprof) top10
   (pprof) web
   (pprof) list functionName

2. Memory Profile:
   go tool pprof mem.prof
   (pprof) top10 -alloc_space
   (pprof) top10 -inuse_space
   (pprof) web

3. Block Profile:
   go tool pprof block.prof
   (pprof) top10
   (pprof) web

4. HTTP Interface:
   http://localhost:6060/debug/pprof/
   go tool pprof http://localhost:6060/debug/pprof/profile
   go tool pprof http://localhost:6060/debug/pprof/heap

5. Trace Analysis:
   go tool trace trace.out
*/
```

Package profiling integration enables comprehensive performance analysis  
through CPU profiling, memory profiling, blocking analysis, and HTTP  
endpoints. Profiling helps identify bottlenecks, memory leaks, and  
performance issues in production applications.  

## Package plugin architecture

Plugin architecture enables dynamic loading and extensibility through  
interfaces and registration patterns.  

```go
// plugin/interface.go
package plugin

import (
    "context"
    "fmt"
    "sync"
)

// Plugin defines the basic plugin interface
type Plugin interface {
    Name() string
    Version() string
    Description() string
    Initialize(config map[string]interface{}) error
    Shutdown() error
}

// ProcessorPlugin defines plugins that can process data
type ProcessorPlugin interface {
    Plugin
    Process(ctx context.Context, input []byte) ([]byte, error)
    CanProcess(contentType string) bool
}

// FilterPlugin defines plugins that can filter data
type FilterPlugin interface {
    Plugin
    Filter(ctx context.Context, data interface{}) (bool, error)
    GetCriteria() map[string]interface{}
}

// HandlerPlugin defines plugins that can handle requests
type HandlerPlugin interface {
    Plugin
    Handle(ctx context.Context, request interface{}) (interface{}, error)
    GetRoutes() []string
}

// ValidatorPlugin defines plugins that can validate data
type ValidatorPlugin interface {
    Plugin
    Validate(ctx context.Context, data interface{}) []ValidationError
    GetSchema() interface{}
}

// ValidationError represents a validation error
type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
    Code    string `json:"code"`
}

// PluginInfo holds metadata about a plugin
type PluginInfo struct {
    Name        string                 `json:"name"`
    Version     string                 `json:"version"`
    Description string                 `json:"description"`
    Type        string                 `json:"type"`
    Config      map[string]interface{} `json:"config"`
    Enabled     bool                   `json:"enabled"`
    LoadedAt    time.Time              `json:"loaded_at"`
}

// Registry manages plugin registration and lookup
type Registry struct {
    plugins     map[string]Plugin
    pluginTypes map[string][]Plugin
    info        map[string]PluginInfo
    mu          sync.RWMutex
}

// NewRegistry creates a new plugin registry
func NewRegistry() *Registry {
    return &Registry{
        plugins:     make(map[string]Plugin),
        pluginTypes: make(map[string][]Plugin),
        info:        make(map[string]PluginInfo),
    }
}

// Register adds a plugin to the registry
func (r *Registry) Register(plugin Plugin, config map[string]interface{}) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    name := plugin.Name()
    if _, exists := r.plugins[name]; exists {
        return fmt.Errorf("plugin %s is already registered", name)
    }
    
    // Initialize the plugin
    if err := plugin.Initialize(config); err != nil {
        return fmt.Errorf("failed to initialize plugin %s: %w", name, err)
    }
    
    // Register the plugin
    r.plugins[name] = plugin
    
    // Categorize by type
    pluginType := r.getPluginType(plugin)
    r.pluginTypes[pluginType] = append(r.pluginTypes[pluginType], plugin)
    
    // Store plugin info
    r.info[name] = PluginInfo{
        Name:        name,
        Version:     plugin.Version(),
        Description: plugin.Description(),
        Type:        pluginType,
        Config:      config,
        Enabled:     true,
        LoadedAt:    time.Now(),
    }
    
    return nil
}

// Unregister removes a plugin from the registry
func (r *Registry) Unregister(name string) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    plugin, exists := r.plugins[name]
    if !exists {
        return fmt.Errorf("plugin %s is not registered", name)
    }
    
    // Shutdown the plugin
    if err := plugin.Shutdown(); err != nil {
        return fmt.Errorf("failed to shutdown plugin %s: %w", name, err)
    }
    
    // Remove from registry
    delete(r.plugins, name)
    
    // Remove from type categorization
    pluginType := r.getPluginType(plugin)
    plugins := r.pluginTypes[pluginType]
    for i, p := range plugins {
        if p.Name() == name {
            r.pluginTypes[pluginType] = append(plugins[:i], plugins[i+1:]...)
            break
        }
    }
    
    delete(r.info, name)
    
    return nil
}

// Get retrieves a plugin by name
func (r *Registry) Get(name string) (Plugin, error) {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    plugin, exists := r.plugins[name]
    if !exists {
        return nil, fmt.Errorf("plugin %s not found", name)
    }
    
    return plugin, nil
}

// GetByType retrieves all plugins of a specific type
func (r *Registry) GetByType(pluginType string) []Plugin {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    plugins := r.pluginTypes[pluginType]
    result := make([]Plugin, len(plugins))
    copy(result, plugins)
    return result
}

// GetProcessors retrieves all processor plugins
func (r *Registry) GetProcessors() []ProcessorPlugin {
    plugins := r.GetByType("processor")
    result := make([]ProcessorPlugin, len(plugins))
    for i, p := range plugins {
        result[i] = p.(ProcessorPlugin)
    }
    return result
}

// GetFilters retrieves all filter plugins
func (r *Registry) GetFilters() []FilterPlugin {
    plugins := r.GetByType("filter")
    result := make([]FilterPlugin, len(plugins))
    for i, p := range plugins {
        result[i] = p.(FilterPlugin)
    }
    return result
}

// GetHandlers retrieves all handler plugins
func (r *Registry) GetHandlers() []HandlerPlugin {
    plugins := r.GetByType("handler")
    result := make([]HandlerPlugin, len(plugins))
    for i, p := range plugins {
        result[i] = p.(HandlerPlugin)
    }
    return result
}

// GetValidators retrieves all validator plugins
func (r *Registry) GetValidators() []ValidatorPlugin {
    plugins := r.GetByType("validator")
    result := make([]ValidatorPlugin, len(plugins))
    for i, p := range plugins {
        result[i] = p.(ValidatorPlugin)
    }
    return result
}

// ListPlugins returns information about all registered plugins
func (r *Registry) ListPlugins() []PluginInfo {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    result := make([]PluginInfo, 0, len(r.info))
    for _, info := range r.info {
        result = append(result, info)
    }
    return result
}

// EnablePlugin enables a plugin
func (r *Registry) EnablePlugin(name string) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    info, exists := r.info[name]
    if !exists {
        return fmt.Errorf("plugin %s not found", name)
    }
    
    info.Enabled = true
    r.info[name] = info
    return nil
}

// DisablePlugin disables a plugin
func (r *Registry) DisablePlugin(name string) error {
    r.mu.Lock()
    defer r.mu.Unlock()
    
    info, exists := r.info[name]
    if !exists {
        return fmt.Errorf("plugin %s not found", name)
    }
    
    info.Enabled = false
    r.info[name] = info
    return nil
}

// IsEnabled checks if a plugin is enabled
func (r *Registry) IsEnabled(name string) bool {
    r.mu.RLock()
    defer r.mu.RUnlock()
    
    info, exists := r.info[name]
    return exists && info.Enabled
}

func (r *Registry) getPluginType(plugin Plugin) string {
    switch plugin.(type) {
    case ProcessorPlugin:
        return "processor"
    case FilterPlugin:
        return "filter"
    case HandlerPlugin:
        return "handler"
    case ValidatorPlugin:
        return "validator"
    default:
        return "generic"
    }
}

// Example processor plugins
type TextProcessorPlugin struct {
    name    string
    version string
    config  map[string]interface{}
}

func NewTextProcessorPlugin() *TextProcessorPlugin {
    return &TextProcessorPlugin{
        name:    "text-processor",
        version: "1.0.0",
    }
}

func (p *TextProcessorPlugin) Name() string { return p.name }
func (p *TextProcessorPlugin) Version() string { return p.version }
func (p *TextProcessorPlugin) Description() string { 
    return "Processes text content with various transformations"
}

func (p *TextProcessorPlugin) Initialize(config map[string]interface{}) error {
    p.config = config
    return nil
}

func (p *TextProcessorPlugin) Shutdown() error {
    return nil
}

func (p *TextProcessorPlugin) Process(ctx context.Context, input []byte) ([]byte, error) {
    text := string(input)
    
    // Apply transformations based on config
    if transform, ok := p.config["transform"].(string); ok {
        switch transform {
        case "uppercase":
            text = strings.ToUpper(text)
        case "lowercase":
            text = strings.ToLower(text)
        case "reverse":
            runes := []rune(text)
            for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
                runes[i], runes[j] = runes[j], runes[i]
            }
            text = string(runes)
        }
    }
    
    return []byte(text), nil
}

func (p *TextProcessorPlugin) CanProcess(contentType string) bool {
    return strings.HasPrefix(contentType, "text/")
}

// Example filter plugin
type LengthFilterPlugin struct {
    name      string
    version   string
    minLength int
    maxLength int
}

func NewLengthFilterPlugin(minLen, maxLen int) *LengthFilterPlugin {
    return &LengthFilterPlugin{
        name:      "length-filter",
        version:   "1.0.0",
        minLength: minLen,
        maxLength: maxLen,
    }
}

func (p *LengthFilterPlugin) Name() string { return p.name }
func (p *LengthFilterPlugin) Version() string { return p.version }
func (p *LengthFilterPlugin) Description() string {
    return fmt.Sprintf("Filters content by length (%d-%d characters)", p.minLength, p.maxLength)
}

func (p *LengthFilterPlugin) Initialize(config map[string]interface{}) error {
    if minLen, ok := config["min_length"].(float64); ok {
        p.minLength = int(minLen)
    }
    if maxLen, ok := config["max_length"].(float64); ok {
        p.maxLength = int(maxLen)
    }
    return nil
}

func (p *LengthFilterPlugin) Shutdown() error {
    return nil
}

func (p *LengthFilterPlugin) Filter(ctx context.Context, data interface{}) (bool, error) {
    var length int
    
    switch v := data.(type) {
    case string:
        length = len(v)
    case []byte:
        length = len(v)
    default:
        return false, fmt.Errorf("unsupported data type for length filter")
    }
    
    return length >= p.minLength && length <= p.maxLength, nil
}

func (p *LengthFilterPlugin) GetCriteria() map[string]interface{} {
    return map[string]interface{}{
        "min_length": p.minLength,
        "max_length": p.maxLength,
    }
}

// Global registry
var globalRegistry = NewRegistry()

// Package-level convenience functions
func Register(plugin Plugin, config map[string]interface{}) error {
    return globalRegistry.Register(plugin, config)
}

func Get(name string) (Plugin, error) {
    return globalRegistry.Get(name)
}

func GetProcessors() []ProcessorPlugin {
    return globalRegistry.GetProcessors()
}

func GetFilters() []FilterPlugin {
    return globalRegistry.GetFilters()
}

func ListPlugins() []PluginInfo {
    return globalRegistry.ListPlugins()
}

// Usage example
func main() {
    // Register plugins
    textProcessor := NewTextProcessorPlugin()
    err := Register(textProcessor, map[string]interface{}{
        "transform": "uppercase",
    })
    if err != nil {
        log.Fatal("Failed to register text processor:", err)
    }
    
    lengthFilter := NewLengthFilterPlugin(5, 100)
    err = Register(lengthFilter, map[string]interface{}{
        "min_length": 10,
        "max_length": 200,
    })
    if err != nil {
        log.Fatal("Failed to register length filter:", err)
    }
    
    // Use plugins
    ctx := context.Background()
    
    // Process data with processor plugins
    processors := GetProcessors()
    for _, processor := range processors {
        if processor.CanProcess("text/plain") {
            input := []byte("hello there, world!")
            output, err := processor.Process(ctx, input)
            if err != nil {
                log.Printf("Processing error: %v", err)
                continue
            }
            fmt.Printf("Processed: %s -> %s\n", string(input), string(output))
        }
    }
    
    // Filter data with filter plugins
    filters := GetFilters()
    testData := "This is a test string for filtering"
    
    for _, filter := range filters {
        passed, err := filter.Filter(ctx, testData)
        if err != nil {
            log.Printf("Filter error: %v", err)
            continue
        }
        fmt.Printf("Filter %s: %s -> %t\n", filter.Name(), testData, passed)
    }
    
    // List all plugins
    fmt.Println("\nRegistered plugins:")
    for _, info := range ListPlugins() {
        fmt.Printf("- %s v%s (%s): %s\n", 
            info.Name, info.Version, info.Type, info.Description)
    }
}
```

Package plugin architecture enables extensibility through interface-based  
design and dynamic registration. Plugins provide modularity, allow third-party  
extensions, and support runtime configuration. The registry pattern manages  
plugin lifecycle and provides type-safe access to different plugin categories.  

## Package versioning strategies

Versioning strategies enable backward compatibility, semantic versioning,  
and migration paths for package evolution.  

```go
// version/version.go
package version

import (
    "fmt"
    "regexp"
    "strconv"
    "strings"
    "time"
)

// Version represents a semantic version
type Version struct {
    Major      int    `json:"major"`
    Minor      int    `json:"minor"`
    Patch      int    `json:"patch"`
    PreRelease string `json:"pre_release,omitempty"`
    BuildMeta  string `json:"build_meta,omitempty"`
}

// Parse parses a version string into a Version struct
func Parse(v string) (*Version, error) {
    // Semantic version regex
    re := regexp.MustCompile(`^v?(\d+)\.(\d+)\.(\d+)(?:-([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?(?:\+([0-9A-Za-z-]+(?:\.[0-9A-Za-z-]+)*))?$`)
    
    matches := re.FindStringSubmatch(v)
    if matches == nil {
        return nil, fmt.Errorf("invalid version format: %s", v)
    }
    
    major, _ := strconv.Atoi(matches[1])
    minor, _ := strconv.Atoi(matches[2])
    patch, _ := strconv.Atoi(matches[3])
    
    return &Version{
        Major:      major,
        Minor:      minor,
        Patch:      patch,
        PreRelease: matches[4],
        BuildMeta:  matches[5],
    }, nil
}

// New creates a new version
func New(major, minor, patch int) *Version {
    return &Version{
        Major: major,
        Minor: minor,
        Patch: patch,
    }
}

// String returns the string representation of the version
func (v *Version) String() string {
    version := fmt.Sprintf("%d.%d.%d", v.Major, v.Minor, v.Patch)
    
    if v.PreRelease != "" {
        version += "-" + v.PreRelease
    }
    
    if v.BuildMeta != "" {
        version += "+" + v.BuildMeta
    }
    
    return version
}

// WithPreRelease adds a pre-release identifier
func (v *Version) WithPreRelease(preRelease string) *Version {
    newVersion := *v
    newVersion.PreRelease = preRelease
    return &newVersion
}

// WithBuildMeta adds build metadata
func (v *Version) WithBuildMeta(buildMeta string) *Version {
    newVersion := *v
    newVersion.BuildMeta = buildMeta
    return &newVersion
}

// Compare compares two versions (-1, 0, 1 for less, equal, greater)
func (v *Version) Compare(other *Version) int {
    // Compare major, minor, patch
    if v.Major != other.Major {
        if v.Major < other.Major {
            return -1
        }
        return 1
    }
    
    if v.Minor != other.Minor {
        if v.Minor < other.Minor {
            return -1
        }
        return 1
    }
    
    if v.Patch != other.Patch {
        if v.Patch < other.Patch {
            return -1
        }
        return 1
    }
    
    // Compare pre-release
    if v.PreRelease == "" && other.PreRelease != "" {
        return 1 // No pre-release is higher than with pre-release
    }
    if v.PreRelease != "" && other.PreRelease == "" {
        return -1
    }
    if v.PreRelease != "" && other.PreRelease != "" {
        return strings.Compare(v.PreRelease, other.PreRelease)
    }
    
    return 0
}

// IsCompatible checks if this version is compatible with another version
func (v *Version) IsCompatible(other *Version) bool {
    // Same major version for backward compatibility
    return v.Major == other.Major && v.Compare(other) >= 0
}

// IsBreakingChange checks if updating to another version would be breaking
func (v *Version) IsBreakingChange(other *Version) bool {
    return other.Major > v.Major
}

// NextMajor returns the next major version
func (v *Version) NextMajor() *Version {
    return &Version{Major: v.Major + 1, Minor: 0, Patch: 0}
}

// NextMinor returns the next minor version
func (v *Version) NextMinor() *Version {
    return &Version{Major: v.Major, Minor: v.Minor + 1, Patch: 0}
}

// NextPatch returns the next patch version
func (v *Version) NextPatch() *Version {
    return &Version{Major: v.Major, Minor: v.Minor, Patch: v.Patch + 1}
}

// APIVersion represents API versioning information
type APIVersion struct {
    Version     *Version           `json:"version"`
    APILevel    int                `json:"api_level"`
    Deprecated  bool               `json:"deprecated"`
    EOL         *time.Time         `json:"eol,omitempty"`
    Features    map[string]bool    `json:"features"`
    Changes     []ChangeEntry      `json:"changes"`
}

// ChangeEntry represents a version change
type ChangeEntry struct {
    Version     string    `json:"version"`
    Date        time.Time `json:"date"`
    Type        string    `json:"type"` // "added", "changed", "deprecated", "removed", "fixed", "security"
    Description string    `json:"description"`
    Breaking    bool      `json:"breaking"`
}

// VersionManager manages API versions and compatibility
type VersionManager struct {
    current     *Version
    supported   []*APIVersion
    deprecated  []*APIVersion
    migrations  map[string]MigrationFunc
}

// MigrationFunc represents a function that migrates data between versions
type MigrationFunc func(from, to *Version, data interface{}) (interface{}, error)

// NewVersionManager creates a new version manager
func NewVersionManager(current *Version) *VersionManager {
    return &VersionManager{
        current:    current,
        supported:  make([]*APIVersion, 0),
        deprecated: make([]*APIVersion, 0),
        migrations: make(map[string]MigrationFunc),
    }
}

// AddSupportedVersion adds a supported API version
func (vm *VersionManager) AddSupportedVersion(apiVersion *APIVersion) {
    vm.supported = append(vm.supported, apiVersion)
}

// AddDeprecatedVersion adds a deprecated API version
func (vm *VersionManager) AddDeprecatedVersion(apiVersion *APIVersion) {
    vm.deprecated = append(vm.deprecated, apiVersion)
}

// AddMigration adds a migration function between versions
func (vm *VersionManager) AddMigration(fromVersion, toVersion string, migrationFunc MigrationFunc) {
    key := fmt.Sprintf("%s->%s", fromVersion, toVersion)
    vm.migrations[key] = migrationFunc
}

// GetCurrent returns the current version
func (vm *VersionManager) GetCurrent() *Version {
    return vm.current
}

// IsSupported checks if a version is supported
func (vm *VersionManager) IsSupported(version *Version) bool {
    for _, supported := range vm.supported {
        if supported.Version.Compare(version) == 0 {
            return true
        }
    }
    return false
}

// IsDeprecated checks if a version is deprecated
func (vm *VersionManager) IsDeprecated(version *Version) bool {
    for _, deprecated := range vm.deprecated {
        if deprecated.Version.Compare(version) == 0 {
            return true
        }
    }
    return false
}

// GetCompatibleVersions returns versions compatible with the given version
func (vm *VersionManager) GetCompatibleVersions(version *Version) []*Version {
    var compatible []*Version
    
    for _, supported := range vm.supported {
        if version.IsCompatible(supported.Version) {
            compatible = append(compatible, supported.Version)
        }
    }
    
    return compatible
}

// Migrate migrates data from one version to another
func (vm *VersionManager) Migrate(from, to *Version, data interface{}) (interface{}, error) {
    if from.Compare(to) == 0 {
        return data, nil // No migration needed
    }
    
    key := fmt.Sprintf("%s->%s", from.String(), to.String())
    if migration, exists := vm.migrations[key]; exists {
        return migration(from, to, data)
    }
    
    return nil, fmt.Errorf("no migration path from %s to %s", from.String(), to.String())
}

// GetChangelog returns the changelog for all versions
func (vm *VersionManager) GetChangelog() []ChangeEntry {
    var allChanges []ChangeEntry
    
    for _, apiVersion := range vm.supported {
        allChanges = append(allChanges, apiVersion.Changes...)
    }
    
    for _, apiVersion := range vm.deprecated {
        allChanges = append(allChanges, apiVersion.Changes...)
    }
    
    return allChanges
}

// Feature represents a feature flag with versioning
type Feature struct {
    Name            string     `json:"name"`
    Description     string     `json:"description"`
    IntroducedIn    *Version   `json:"introduced_in"`
    DeprecatedIn    *Version   `json:"deprecated_in,omitempty"`
    RemovedIn       *Version   `json:"removed_in,omitempty"`
    Enabled         bool       `json:"enabled"`
    DefaultEnabled  bool       `json:"default_enabled"`
}

// IsAvailable checks if a feature is available in a specific version
func (f *Feature) IsAvailable(version *Version) bool {
    if version.Compare(f.IntroducedIn) < 0 {
        return false
    }
    
    if f.RemovedIn != nil && version.Compare(f.RemovedIn) >= 0 {
        return false
    }
    
    return true
}

// IsDeprecated checks if a feature is deprecated in a specific version
func (f *Feature) IsDeprecated(version *Version) bool {
    if f.DeprecatedIn == nil {
        return false
    }
    
    return version.Compare(f.DeprecatedIn) >= 0 && 
           (f.RemovedIn == nil || version.Compare(f.RemovedIn) < 0)
}

// Package-level version information
var (
    PackageVersion = New(2, 1, 0)
    APILevel       = 3
    BuildTime      = time.Now()
    GitCommit      = "abc123def456"
)

// GetPackageInfo returns comprehensive package version information
func GetPackageInfo() map[string]interface{} {
    return map[string]interface{}{
        "version":    PackageVersion.String(),
        "api_level":  APILevel,
        "build_time": BuildTime.Format(time.RFC3339),
        "git_commit": GitCommit,
        "go_version": runtime.Version(),
    }
}

// Example usage
func main() {
    // Version parsing and comparison
    v1, _ := Parse("1.2.3")
    v2, _ := Parse("1.2.4-beta.1+build.123")
    v3, _ := Parse("2.0.0")
    
    fmt.Printf("v1: %s\n", v1)
    fmt.Printf("v2: %s\n", v2)
    fmt.Printf("v3: %s\n", v3)
    
    fmt.Printf("v1 vs v2: %d\n", v1.Compare(v2))
    fmt.Printf("v1 compatible with v2: %t\n", v1.IsCompatible(v2))
    fmt.Printf("v1 to v3 breaking: %t\n", v1.IsBreakingChange(v3))
    
    // Version manager
    vm := NewVersionManager(New(2, 1, 0))
    
    // Add supported versions
    vm.AddSupportedVersion(&APIVersion{
        Version:  New(2, 1, 0),
        APILevel: 3,
        Features: map[string]bool{
            "feature_a": true,
            "feature_b": true,
        },
        Changes: []ChangeEntry{
            {
                Version:     "2.1.0",
                Date:        time.Now(),
                Type:        "added",
                Description: "Added new authentication method",
                Breaking:    false,
            },
        },
    })
    
    vm.AddDeprecatedVersion(&APIVersion{
        Version:    New(1, 5, 0),
        APILevel:   2,
        Deprecated: true,
        EOL:        &time.Time{},
    })
    
    // Add migration
    vm.AddMigration("1.5.0", "2.1.0", func(from, to *Version, data interface{}) (interface{}, error) {
        // Migration logic here
        fmt.Printf("Migrating from %s to %s\n", from, to)
        return data, nil
    })
    
    // Feature management
    feature := &Feature{
        Name:         "advanced_search",
        Description:  "Advanced search capabilities",
        IntroducedIn: New(2, 0, 0),
        DeprecatedIn: New(3, 0, 0),
        Enabled:      true,
    }
    
    currentVersion := New(2, 1, 0)
    fmt.Printf("Feature available in %s: %t\n", currentVersion, feature.IsAvailable(currentVersion))
    fmt.Printf("Feature deprecated in %s: %t\n", currentVersion, feature.IsDeprecated(currentVersion))
    
    // Package info
    info := GetPackageInfo()
    fmt.Printf("Package info: %+v\n", info)
}
```

Package versioning strategies provide structured approaches to managing API  
evolution, backward compatibility, and feature lifecycle. Semantic versioning,  
migration paths, and feature flags enable controlled package evolution while  
maintaining stability for existing users. Version managers help coordinate  
complex upgrade scenarios and deprecation timelines.  