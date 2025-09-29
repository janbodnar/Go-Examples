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