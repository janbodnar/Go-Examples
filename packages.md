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