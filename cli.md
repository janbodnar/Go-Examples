# Go Command-line Arguments and Flags

Command-line argument and flag processing is a fundamental aspect of building  
command-line interface (CLI) applications in Go. The ability to accept user  
input through command-line arguments allows programs to be flexible, configurable,  
and scriptable. Go provides excellent built-in support for handling command-line  
arguments through the `os` package and more sophisticated flag processing through  
the `flag` package.

Command-line arguments are the values passed to a program when it's executed.  
In Go, these arguments are automatically captured and made available through  
`os.Args`, which is a slice of strings containing the program name and all  
arguments passed to it. The first element (`os.Args[0]`) is always the program  
name, while subsequent elements contain the actual arguments provided by the user.

The `flag` package provides a more structured approach to command-line argument  
processing by implementing POSIX-style flags with support for various data types,  
default values, and automatic help generation. Flags can be boolean switches,  
string values, integers, floating-point numbers, or custom types. The flag  
package automatically handles parsing, validation, and provides built-in help  
functionality that makes CLI applications user-friendly.

Modern CLI development in Go often involves creating applications with subcommands,  
complex flag hierarchies, and rich user interactions. Advanced patterns include  
flag validation, custom flag types, configuration file integration, and  
environment variable support. Understanding these concepts enables developers  
to build professional-grade command-line tools that follow Unix conventions  
and provide excellent user experience.

Go's approach to CLI development emphasizes simplicity and convention while  
providing powerful features for complex use cases. The standard library's  
flag package covers most common scenarios, while third-party libraries like  
Cobra and Kingpin extend capabilities for building sophisticated CLI applications  
with features like nested subcommands, shell completion, and advanced validation.


## Basic command-line arguments

The `os.Args` slice provides access to raw command-line arguments passed to  
your Go program.  

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    fmt.Printf("Program name: %s\n", os.Args[0])
    fmt.Printf("Number of arguments: %d\n", len(os.Args)-1)
    
    if len(os.Args) > 1 {
        fmt.Println("Arguments provided:")
        for i, arg := range os.Args[1:] {
            fmt.Printf("  [%d]: %s\n", i+1, arg)
        }
    } else {
        fmt.Println("No arguments provided")
    }
    
    // Demonstrate argument processing
    if len(os.Args) >= 3 {
        first := os.Args[1]
        second := os.Args[2] 
        fmt.Printf("First two args: '%s' and '%s'\n", first, second)
    }
}
```

This example shows how to access and iterate through command-line arguments.  
The program displays the executable name, counts arguments, and processes them  
individually. Note that `os.Args[0]` contains the program name, so actual user  
arguments start from index 1.  

## Argument validation and processing

Raw arguments require validation and type conversion for safe processing in  
real applications.  

```go
package main

import (
    "fmt"
    "os"
    "strconv"
    "strings"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Usage: program <command> [options...]")
        fmt.Println("Commands: add, multiply, greet")
        os.Exit(1)
    }
    
    command := strings.ToLower(os.Args[1])
    
    switch command {
    case "add":
        if len(os.Args) < 4 {
            fmt.Println("Usage: program add <num1> <num2>")
            os.Exit(1)
        }
        
        num1, err1 := strconv.ParseFloat(os.Args[2], 64)
        num2, err2 := strconv.ParseFloat(os.Args[3], 64)
        
        if err1 != nil || err2 != nil {
            fmt.Println("Error: Both arguments must be valid numbers")
            os.Exit(1)
        }
        
        result := num1 + num2
        fmt.Printf("%.2f + %.2f = %.2f\n", num1, num2, result)
        
    case "multiply":
        if len(os.Args) < 4 {
            fmt.Println("Usage: program multiply <num1> <num2>")
            os.Exit(1)
        }
        
        num1, err1 := strconv.ParseFloat(os.Args[2], 64)
        num2, err2 := strconv.ParseFloat(os.Args[3], 64)
        
        if err1 != nil || err2 != nil {
            fmt.Println("Error: Both arguments must be valid numbers")
            os.Exit(1)
        }
        
        result := num1 * num2
        fmt.Printf("%.2f × %.2f = %.2f\n", num1, num2, result)
        
    case "greet":
        name := "there"
        if len(os.Args) >= 3 {
            name = strings.Join(os.Args[2:], " ")
        }
        fmt.Printf("Hello %s!\n", name)
        
    default:
        fmt.Printf("Unknown command: %s\n", command)
        fmt.Println("Available commands: add, multiply, greet")
        os.Exit(1)
    }
}
```

This example demonstrates argument validation, type conversion, and command  
processing. It shows how to handle different argument counts, convert strings  
to numbers, and provide meaningful error messages. The program implements a  
simple calculator with multiple operations.  

## Basic flags with flag package

The `flag` package provides structured command-line flag processing with  
automatic parsing and help generation.  

```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    // Define flags
    var verbose bool
    var output string
    var count int
    var threshold float64
    
    flag.BoolVar(&verbose, "verbose", false, "enable verbose output")
    flag.BoolVar(&verbose, "v", false, "enable verbose output (short)")
    flag.StringVar(&output, "output", "result.txt", "output file name")
    flag.StringVar(&output, "o", "result.txt", "output file name (short)")
    flag.IntVar(&count, "count", 10, "number of items to process")
    flag.Float64Var(&threshold, "threshold", 0.5, "threshold value")
    
    // Parse flags
    flag.Parse()
    
    // Display parsed values
    fmt.Printf("Verbose mode: %t\n", verbose)
    fmt.Printf("Output file: %s\n", output)
    fmt.Printf("Count: %d\n", count)
    fmt.Printf("Threshold: %.2f\n", threshold)
    
    // Get remaining non-flag arguments
    args := flag.Args()
    fmt.Printf("Remaining arguments: %v\n", args)
    
    if verbose {
        fmt.Println("Detailed processing information:")
        for i, arg := range args {
            fmt.Printf("  Processing argument %d: %s\n", i+1, arg)
        }
    }
}
```

The flag package automatically handles flag parsing, type conversion, and  
provides both long and short flag variants. The `flag.Parse()` function processes  
all flags, while `flag.Args()` returns remaining non-flag arguments. Default  
values and help text are automatically managed.  

## Custom flag usage and help

The flag package allows customization of usage messages and help output  
formatting.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "time"
)

var (
    input     = flag.String("input", "", "input file path (required)")
    format    = flag.String("format", "json", "output format: json, xml, csv")
    compress  = flag.Bool("compress", false, "compress output file") 
    workers   = flag.Int("workers", 4, "number of worker threads")
    timeout   = flag.Duration("timeout", 30*time.Second, "processing timeout")
)

func init() {
    flag.Usage = func() {
        fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
        fmt.Fprintf(os.Stderr, "\nFile Processing Tool\n")
        fmt.Fprintf(os.Stderr, "Processes input files and converts them to different formats.\n\n")
        fmt.Fprintf(os.Stderr, "Example:\n")
        fmt.Fprintf(os.Stderr, "  %s -input data.txt -format json -compress\n\n", os.Args[0])
        fmt.Fprintf(os.Stderr, "Options:\n")
        flag.PrintDefaults()
    }
}

func main() {
    flag.Parse()
    
    // Validate required flags
    if *input == "" {
        fmt.Fprintf(os.Stderr, "Error: -input flag is required\n\n")
        flag.Usage()
        os.Exit(1)
    }
    
    // Validate format option
    validFormats := map[string]bool{
        "json": true,
        "xml":  true, 
        "csv":  true,
    }
    
    if !validFormats[*format] {
        fmt.Fprintf(os.Stderr, "Error: invalid format '%s'\n", *format)
        fmt.Fprintf(os.Stderr, "Valid formats: json, xml, csv\n\n")
        os.Exit(1)
    }
    
    fmt.Printf("Processing file: %s\n", *input)
    fmt.Printf("Output format: %s\n", *format)
    fmt.Printf("Compression: %t\n", *compress)
    fmt.Printf("Workers: %d\n", *workers)
    fmt.Printf("Timeout: %v\n", *timeout)
}
```

This example shows how to create custom usage messages, validate required flags,  
and implement input validation. The `init()` function sets a custom usage  
function that provides detailed help information including examples and  
descriptions.  

## Flag pointers and direct access

Flags can be defined using pointer-returning functions for more flexible  
access patterns.  

```go
package main

import (
    "flag"
    "fmt"
    "strings"
    "time"
)

func main() {
    // Using pointer-returning flag functions
    host := flag.String("host", "localhost", "server hostname")
    port := flag.Int("port", 8080, "server port")
    debug := flag.Bool("debug", false, "enable debug mode")
    tags := flag.String("tags", "", "comma-separated tags")
    timeout := flag.Duration("timeout", 5*time.Second, "connection timeout")
    
    flag.Parse()
    
    fmt.Printf("Server configuration:\n")
    fmt.Printf("  Host: %s\n", *host)
    fmt.Printf("  Port: %d\n", *port)
    fmt.Printf("  Debug: %t\n", *debug)
    fmt.Printf("  Timeout: %v\n", *timeout)
    
    // Process tags if provided
    if *tags != "" {
        tagList := strings.Split(*tags, ",")
        fmt.Printf("  Tags: %v\n", tagList)
        
        if *debug {
            fmt.Println("Debug: Processing tags:")
            for i, tag := range tagList {
                fmt.Printf("    %d: %s\n", i+1, strings.TrimSpace(tag))
            }
        }
    }
    
    // Construct server address
    address := fmt.Sprintf("%s:%d", *host, *port)
    fmt.Printf("  Full address: %s\n", address)
    
    // Simulate server start
    if *debug {
        fmt.Printf("Debug: Starting server with %v timeout\n", *timeout)
    }
}
```

Using pointer-returning functions (`flag.String`, `flag.Int`, etc.) provides  
direct access to parsed values without needing separate variable declarations.  
This pattern is common in Go CLI applications for its conciseness and clarity.  

## Boolean flags and variations

Boolean flags in Go support multiple formats and can be used as switches  
or with explicit values.  

```go
package main

import (
    "flag"
    "fmt"
)

func main() {
    var (
        verbose  = flag.Bool("verbose", false, "enable verbose logging")
        quiet    = flag.Bool("quiet", false, "suppress all output") 
        force    = flag.Bool("force", false, "force operation without confirmation")
        dryRun   = flag.Bool("dry-run", false, "show what would be done without executing")
        colored  = flag.Bool("color", true, "enable colored output")
        autoYes  = flag.Bool("yes", false, "automatically answer yes to prompts")
    )
    
    flag.Parse()
    
    fmt.Printf("Configuration flags:\n")
    fmt.Printf("  Verbose: %t\n", *verbose)
    fmt.Printf("  Quiet: %t\n", *quiet)
    fmt.Printf("  Force: %t\n", *force) 
    fmt.Printf("  Dry run: %t\n", *dryRun)
    fmt.Printf("  Colored output: %t\n", *colored)
    fmt.Printf("  Auto-yes: %t\n", *autoYes)
    
    // Validate conflicting flags
    if *verbose && *quiet {
        fmt.Println("Error: cannot use both -verbose and -quiet flags")
        return
    }
    
    // Demonstrate flag-based behavior
    if *dryRun {
        fmt.Println("\n[DRY RUN MODE] Operations that would be performed:")
    }
    
    if !*quiet {
        message := "Performing operation..."
        if *colored {
            message = "\033[32m" + message + "\033[0m" // Green color
        }
        fmt.Println(message)
    }
    
    if *verbose {
        fmt.Println("Verbose: Detailed operation steps would be shown here")
    }
    
    if *force {
        fmt.Println("Force mode: Skipping safety checks")
    } else if !*autoYes {
        fmt.Println("Would prompt for confirmation (use -yes to skip)")
    }
}
```

Boolean flags can be specified as `-flag`, `-flag=true`, or `-flag=false`.  
This example shows common boolean flag patterns and demonstrates handling  
conflicting flags and conditional behavior based on flag combinations.  

## String flags with validation

String flags often require validation to ensure they contain acceptable  
values or meet specific format requirements.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "regexp"
    "strings"
)

func main() {
    var (
        logLevel  = flag.String("log-level", "info", "log level: debug, info, warn, error")
        format    = flag.String("format", "table", "output format: table, json, csv, xml")
        email     = flag.String("email", "", "email address for notifications")
        url       = flag.String("url", "", "target URL")
        output    = flag.String("output", "", "output directory path")
    )
    
    flag.Parse()
    
    // Validate log level
    validLogLevels := []string{"debug", "info", "warn", "error"}
    if !contains(validLogLevels, *logLevel) {
        fmt.Fprintf(os.Stderr, "Error: invalid log level '%s'\n", *logLevel)
        fmt.Fprintf(os.Stderr, "Valid levels: %s\n", strings.Join(validLogLevels, ", "))
        os.Exit(1)
    }
    
    // Validate format
    validFormats := []string{"table", "json", "csv", "xml"}
    if !contains(validFormats, *format) {
        fmt.Fprintf(os.Stderr, "Error: invalid format '%s'\n", *format)
        fmt.Fprintf(os.Stderr, "Valid formats: %s\n", strings.Join(validFormats, ", "))
        os.Exit(1)
    }
    
    // Validate email format if provided
    if *email != "" {
        emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
        if !emailRegex.MatchString(*email) {
            fmt.Fprintf(os.Stderr, "Error: invalid email format '%s'\n", *email)
            os.Exit(1)
        }
    }
    
    // Validate URL format if provided
    if *url != "" {
        urlRegex := regexp.MustCompile(`^https?://[^\s/$.?#].[^\s]*$`)
        if !urlRegex.MatchString(*url) {
            fmt.Fprintf(os.Stderr, "Error: invalid URL format '%s'\n", *url)
            os.Exit(1)
        }
    }
    
    fmt.Printf("Validated configuration:\n")
    fmt.Printf("  Log Level: %s\n", *logLevel)
    fmt.Printf("  Format: %s\n", *format)
    if *email != "" {
        fmt.Printf("  Email: %s\n", *email)
    }
    if *url != "" {
        fmt.Printf("  URL: %s\n", *url)
    }
    if *output != "" {
        fmt.Printf("  Output: %s\n", *output)
    }
}

func contains(slice []string, item string) bool {
    for _, s := range slice {
        if s == item {
            return true
        }
    }
    return false
}
```

String flag validation ensures data integrity and provides clear error messages  
for invalid inputs. This example demonstrates validation for enumerated values,  
email addresses, and URLs using regular expressions and helper functions.  

## Numeric flags with ranges

Numeric flags often need range validation to ensure values fall within  
acceptable bounds for the application logic.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    var (
        threads     = flag.Int("threads", 4, "number of threads (1-16)")
        memory      = flag.Int("memory", 512, "memory limit in MB (128-4096)")
        port        = flag.Int("port", 8080, "server port (1024-65535)")
        percentage  = flag.Float64("percentage", 0.5, "completion percentage (0.0-1.0)")
        temperature = flag.Float64("temp", 20.0, "temperature in Celsius (-50.0-50.0)")
        timeout     = flag.Int("timeout", 30, "timeout in seconds (1-300)")
    )
    
    flag.Parse()
    
    // Validate thread count
    if *threads < 1 || *threads > 16 {
        fmt.Fprintf(os.Stderr, "Error: threads must be between 1 and 16 (got %d)\n", *threads)
        os.Exit(1)
    }
    
    // Validate memory limit
    if *memory < 128 || *memory > 4096 {
        fmt.Fprintf(os.Stderr, "Error: memory must be between 128 and 4096 MB (got %d)\n", *memory)
        os.Exit(1)
    }
    
    // Validate port range
    if *port < 1024 || *port > 65535 {
        fmt.Fprintf(os.Stderr, "Error: port must be between 1024 and 65535 (got %d)\n", *port)
        os.Exit(1)
    }
    
    // Validate percentage
    if *percentage < 0.0 || *percentage > 1.0 {
        fmt.Fprintf(os.Stderr, "Error: percentage must be between 0.0 and 1.0 (got %.2f)\n", *percentage)
        os.Exit(1)
    }
    
    // Validate temperature
    if *temperature < -50.0 || *temperature > 50.0 {
        fmt.Fprintf(os.Stderr, "Error: temperature must be between -50.0 and 50.0°C (got %.1f)\n", *temperature)
        os.Exit(1)
    }
    
    // Validate timeout
    if *timeout < 1 || *timeout > 300 {
        fmt.Fprintf(os.Stderr, "Error: timeout must be between 1 and 300 seconds (got %d)\n", *timeout)
        os.Exit(1)
    }
    
    fmt.Printf("System configuration:\n")
    fmt.Printf("  Threads: %d\n", *threads)
    fmt.Printf("  Memory: %d MB\n", *memory)
    fmt.Printf("  Port: %d\n", *port)
    fmt.Printf("  Percentage: %.2f%% \n", *percentage*100)
    fmt.Printf("  Temperature: %.1f°C\n", *temperature)
    fmt.Printf("  Timeout: %d seconds\n", *timeout)
}
```

Numeric validation prevents runtime errors and ensures system resources are  
used within safe limits. This example shows validation for different numeric  
types with appropriate error messages that include the valid ranges and  
received values.  

## Duration flags

Go's flag package has built-in support for duration flags that accept  
time values in various formats.  

```go
package main

import (
    "flag"
    "fmt"
    "time"
)

func main() {
    var (
        connectTimeout = flag.Duration("connect-timeout", 5*time.Second, "connection timeout")
        readTimeout    = flag.Duration("read-timeout", 30*time.Second, "read timeout")
        writeTimeout   = flag.Duration("write-timeout", 30*time.Second, "write timeout")
        retryInterval  = flag.Duration("retry-interval", 1*time.Minute, "retry interval")
        maxDuration    = flag.Duration("max-duration", 1*time.Hour, "maximum processing duration")
        pollInterval   = flag.Duration("poll-interval", 100*time.Millisecond, "polling interval")
    )
    
    flag.Parse()
    
    fmt.Printf("Timeout configuration:\n")
    fmt.Printf("  Connect timeout: %v\n", *connectTimeout)
    fmt.Printf("  Read timeout: %v\n", *readTimeout)
    fmt.Printf("  Write timeout: %v\n", *writeTimeout)
    fmt.Printf("  Retry interval: %v\n", *retryInterval)
    fmt.Printf("  Max duration: %v\n", *maxDuration)
    fmt.Printf("  Poll interval: %v\n", *pollInterval)
    
    // Display in different units
    fmt.Printf("\nDuration details:\n")
    fmt.Printf("  Connect timeout: %.0f ms\n", connectTimeout.Seconds()*1000)
    fmt.Printf("  Read timeout: %.0f seconds\n", readTimeout.Seconds())
    fmt.Printf("  Max duration: %.2f hours\n", maxDuration.Hours())
    fmt.Printf("  Poll interval: %.2f microseconds\n", float64(pollInterval.Nanoseconds())/1000)
    
    // Validate duration relationships
    if *connectTimeout > *readTimeout {
        fmt.Println("Warning: connect timeout is greater than read timeout")
    }
    
    if *retryInterval > *maxDuration {
        fmt.Println("Warning: retry interval is greater than max duration")
    }
    
    // Demonstrate duration arithmetic
    totalTimeout := *connectTimeout + *readTimeout + *writeTimeout
    fmt.Printf("  Total operation timeout: %v\n", totalTimeout)
    
    // Show duration parsing examples
    fmt.Printf("\nDuration format examples:\n")
    fmt.Printf("  Milliseconds: 500ms\n")
    fmt.Printf("  Seconds: 30s\n") 
    fmt.Printf("  Minutes: 5m\n")
    fmt.Printf("  Hours: 2h\n")
    fmt.Printf("  Combined: 1h30m45s\n")
}
```

Duration flags accept various time formats including nanoseconds (ns),  
microseconds (us), milliseconds (ms), seconds (s), minutes (m), and hours (h).  
They can be combined like "1h30m" and automatically parse to time.Duration  
values for precise time calculations.  

## Custom flag types

Go allows you to implement custom flag types by satisfying the flag.Value  
interface for complex data structures.  

```go
package main

import (
    "flag"
    "fmt"
    "net"
    "strconv"
    "strings"
)

// IPAddress implements flag.Value for IP address validation
type IPAddress struct {
    IP net.IP
}

func (ip *IPAddress) String() string {
    if ip.IP == nil {
        return ""
    }
    return ip.IP.String()
}

func (ip *IPAddress) Set(value string) error {
    parsedIP := net.ParseIP(value)
    if parsedIP == nil {
        return fmt.Errorf("invalid IP address: %s", value)
    }
    ip.IP = parsedIP
    return nil
}

// PortRange represents a range of ports
type PortRange struct {
    Start, End int
}

func (pr *PortRange) String() string {
    if pr.Start == 0 && pr.End == 0 {
        return ""
    }
    return fmt.Sprintf("%d-%d", pr.Start, pr.End)
}

func (pr *PortRange) Set(value string) error {
    if !strings.Contains(value, "-") {
        port, err := strconv.Atoi(value)
        if err != nil {
            return fmt.Errorf("invalid port: %s", value)
        }
        if port < 1 || port > 65535 {
            return fmt.Errorf("port must be between 1 and 65535: %d", port)
        }
        pr.Start = port
        pr.End = port
        return nil
    }
    
    parts := strings.Split(value, "-")
    if len(parts) != 2 {
        return fmt.Errorf("invalid port range format: %s", value)
    }
    
    start, err1 := strconv.Atoi(parts[0])
    end, err2 := strconv.Atoi(parts[1])
    
    if err1 != nil || err2 != nil {
        return fmt.Errorf("invalid port range: %s", value)
    }
    
    if start < 1 || start > 65535 || end < 1 || end > 65535 {
        return fmt.Errorf("ports must be between 1 and 65535")
    }
    
    if start > end {
        return fmt.Errorf("start port cannot be greater than end port")
    }
    
    pr.Start = start
    pr.End = end
    return nil
}

// StringList represents a list of strings
type StringList []string

func (sl *StringList) String() string {
    return strings.Join(*sl, ",")
}

func (sl *StringList) Set(value string) error {
    *sl = strings.Split(value, ",")
    // Trim whitespace from each item
    for i, item := range *sl {
        (*sl)[i] = strings.TrimSpace(item)
    }
    return nil
}

func main() {
    var (
        serverIP   IPAddress
        portRange  PortRange
        tags       StringList
        configPath = flag.String("config", "app.conf", "configuration file path")
    )
    
    flag.Var(&serverIP, "ip", "server IP address")
    flag.Var(&portRange, "ports", "port or port range (e.g., 8080 or 8080-8090)")
    flag.Var(&tags, "tags", "comma-separated list of tags")
    
    flag.Parse()
    
    fmt.Printf("Custom flag configuration:\n")
    fmt.Printf("  Server IP: %s\n", serverIP.String())
    fmt.Printf("  Port range: %s\n", portRange.String())
    fmt.Printf("  Tags: %v\n", []string(tags))
    fmt.Printf("  Config path: %s\n", *configPath)
    
    // Demonstrate usage of parsed custom types
    if serverIP.IP != nil {
        if serverIP.IP.IsLoopback() {
            fmt.Println("  Note: Using loopback address")
        } else if serverIP.IP.IsPrivate() {
            fmt.Println("  Note: Using private network address")
        }
    }
    
    if portRange.Start != 0 {
        if portRange.Start == portRange.End {
            fmt.Printf("  Single port configuration: %d\n", portRange.Start)
        } else {
            portCount := portRange.End - portRange.Start + 1
            fmt.Printf("  Port range covers %d ports\n", portCount)
        }
    }
    
    if len(tags) > 0 {
        fmt.Printf("  Tag count: %d\n", len(tags))
        for i, tag := range tags {
            fmt.Printf("    %d: %s\n", i+1, tag)
        }
    }
}
```

Custom flag types enable sophisticated command-line argument parsing for  
complex data structures. The `flag.Value` interface requires `String()` and  
`Set()` methods, allowing custom validation and parsing logic for specialized  
types like IP addresses, port ranges, and string lists.  

## Environment variable integration

Combining command-line flags with environment variables provides flexible  
configuration options for different deployment scenarios.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "strconv"
    "strings"
)

func main() {
    var (
        dbHost     = flag.String("db-host", getEnv("DB_HOST", "localhost"), "database host")
        dbPort     = flag.Int("db-port", getEnvInt("DB_PORT", 5432), "database port")
        dbUser     = flag.String("db-user", getEnv("DB_USER", "postgres"), "database user")
        dbPassword = flag.String("db-password", getEnv("DB_PASSWORD", ""), "database password")
        dbName     = flag.String("db-name", getEnv("DB_NAME", "myapp"), "database name")
        logLevel   = flag.String("log-level", getEnv("LOG_LEVEL", "info"), "log level")
        debug      = flag.Bool("debug", getEnvBool("DEBUG", false), "enable debug mode")
        serverPort = flag.Int("port", getEnvInt("PORT", 8080), "server port")
    )
    
    flag.Parse()
    
    fmt.Printf("Configuration (flags override environment variables):\n")
    fmt.Printf("  Database Host: %s\n", *dbHost)
    fmt.Printf("  Database Port: %d\n", *dbPort)
    fmt.Printf("  Database User: %s\n", *dbUser)
    fmt.Printf("  Database Name: %s\n", *dbName)
    fmt.Printf("  Log Level: %s\n", *logLevel)
    fmt.Printf("  Debug Mode: %t\n", *debug)
    fmt.Printf("  Server Port: %d\n", *serverPort)
    
    // Show password status without revealing it
    if *dbPassword != "" {
        fmt.Printf("  Database Password: [SET]\n")
    } else {
        fmt.Printf("  Database Password: [NOT SET]\n")
    }
    
    // Construct database connection string
    if *dbPassword != "" {
        fmt.Printf("  Connection: postgresql://%s:***@%s:%d/%s\n", 
            *dbUser, *dbHost, *dbPort, *dbName)
    } else {
        fmt.Printf("  Connection: postgresql://%s@%s:%d/%s\n", 
            *dbUser, *dbHost, *dbPort, *dbName)
    }
    
    // Show environment variable sources
    fmt.Printf("\nEnvironment variable sources:\n")
    showEnvSource("DB_HOST", *dbHost)
    showEnvSource("DB_PORT", strconv.Itoa(*dbPort))
    showEnvSource("LOG_LEVEL", *logLevel)
    showEnvSource("DEBUG", strconv.FormatBool(*debug))
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getEnvInt(key string, defaultValue int) int {
    if value := os.Getenv(key); value != "" {
        if intValue, err := strconv.Atoi(value); err == nil {
            return intValue
        }
    }
    return defaultValue
}

func getEnvBool(key string, defaultValue bool) bool {
    if value := os.Getenv(key); value != "" {
        if boolValue, err := strconv.ParseBool(value); err == nil {
            return boolValue
        }
    }
    return defaultValue
}

func showEnvSource(key, currentValue string) {
    envValue := os.Getenv(key)
    if envValue != "" && envValue == currentValue {
        fmt.Printf("  %s: %s (from environment)\n", key, envValue)
    } else {
        fmt.Printf("  %s: %s (default or flag override)\n", key, currentValue)
    }
}
```

Environment variable integration allows configuration through multiple sources  
with a clear precedence hierarchy: command-line flags override environment  
variables, which override default values. This pattern is common in  
containerized applications and cloud deployments.  

## Flag sets for subcommands

Flag sets enable creating applications with multiple subcommands, each  
with their own set of flags and options.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    if len(os.Args) < 2 {
        fmt.Println("Usage: program <command> [options]")
        fmt.Println("Commands:")
        fmt.Println("  server    - start HTTP server")
        fmt.Println("  client    - run client operations")
        fmt.Println("  migrate   - run database migrations")
        os.Exit(1)
    }
    
    command := os.Args[1]
    
    switch command {
    case "server":
        serverCommand()
    case "client":
        clientCommand()
    case "migrate":
        migrateCommand()
    default:
        fmt.Printf("Unknown command: %s\n", command)
        os.Exit(1)
    }
}

func serverCommand() {
    serverFlags := flag.NewFlagSet("server", flag.ExitOnError)
    port := serverFlags.Int("port", 8080, "server port")
    host := serverFlags.String("host", "localhost", "server host")
    tls := serverFlags.Bool("tls", false, "enable TLS")
    certFile := serverFlags.String("cert", "", "TLS certificate file")
    keyFile := serverFlags.String("key", "", "TLS key file")
    
    serverFlags.Parse(os.Args[2:])
    
    fmt.Printf("Starting HTTP server:\n")
    fmt.Printf("  Host: %s\n", *host)
    fmt.Printf("  Port: %d\n", *port)
    fmt.Printf("  TLS: %t\n", *tls)
    
    if *tls {
        if *certFile == "" || *keyFile == "" {
            fmt.Println("Error: TLS requires both -cert and -key files")
            os.Exit(1)
        }
        fmt.Printf("  Certificate: %s\n", *certFile)
        fmt.Printf("  Key: %s\n", *keyFile)
        fmt.Printf("  Server URL: https://%s:%d\n", *host, *port)
    } else {
        fmt.Printf("  Server URL: http://%s:%d\n", *host, *port)
    }
}

func clientCommand() {
    clientFlags := flag.NewFlagSet("client", flag.ExitOnError)
    serverURL := clientFlags.String("url", "http://localhost:8080", "server URL")
    timeout := clientFlags.Duration("timeout", 30*time.Second, "request timeout")
    retries := clientFlags.Int("retries", 3, "number of retries")
    output := clientFlags.String("output", "", "output file")
    verbose := clientFlags.Bool("verbose", false, "verbose output")
    
    clientFlags.Parse(os.Args[2:])
    
    fmt.Printf("Client configuration:\n")
    fmt.Printf("  Server URL: %s\n", *serverURL)
    fmt.Printf("  Timeout: %v\n", *timeout)
    fmt.Printf("  Retries: %d\n", *retries)
    fmt.Printf("  Verbose: %t\n", *verbose)
    
    if *output != "" {
        fmt.Printf("  Output file: %s\n", *output)
    }
    
    // Process remaining arguments as API endpoints
    endpoints := clientFlags.Args()
    if len(endpoints) > 0 {
        fmt.Printf("  Endpoints to call: %v\n", endpoints)
    }
}

func migrateCommand() {
    migrateFlags := flag.NewFlagSet("migrate", flag.ExitOnError)
    direction := migrateFlags.String("direction", "up", "migration direction: up, down")
    steps := migrateFlags.Int("steps", 0, "number of migration steps (0 = all)")
    dryRun := migrateFlags.Bool("dry-run", false, "show migrations without executing")
    dbURL := migrateFlags.String("database-url", "", "database connection URL")
    
    migrateFlags.Parse(os.Args[2:])
    
    if *dbURL == "" {
        fmt.Println("Error: -database-url is required for migrations")
        os.Exit(1)
    }
    
    fmt.Printf("Database migration:\n")
    fmt.Printf("  Direction: %s\n", *direction)
    fmt.Printf("  Steps: %d\n", *steps)
    fmt.Printf("  Dry run: %t\n", *dryRun)
    fmt.Printf("  Database: %s\n", maskDBURL(*dbURL))
    
    if *dryRun {
        fmt.Println("  [DRY RUN] Would execute migrations:")
    } else {
        fmt.Println("  Executing migrations:")
    }
}

func maskDBURL(url string) string {
    // Simple masking for demonstration
    if strings.Contains(url, "@") {
        parts := strings.Split(url, "@")
        if len(parts) == 2 {
            return "postgresql://***@" + parts[1]
        }
    }
    return url
}
```

Flag sets provide isolated flag parsing for subcommands, enabling complex  
CLI applications with different options for each command. Each flag set  
maintains its own state and can have different flag definitions and  
validation logic.  

## Configuration file integration

Combining flags with configuration files provides a flexible configuration  
hierarchy for complex applications.  

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "io/ioutil"
    "os"
    "time"
)

// Config represents the application configuration
type Config struct {
    Server struct {
        Host         string        `json:"host"`
        Port         int           `json:"port"`
        ReadTimeout  time.Duration `json:"read_timeout"`
        WriteTimeout time.Duration `json:"write_timeout"`
    } `json:"server"`
    
    Database struct {
        Host     string `json:"host"`
        Port     int    `json:"port"`
        User     string `json:"user"`
        Password string `json:"password"`
        Name     string `json:"name"`
    } `json:"database"`
    
    Logging struct {
        Level  string `json:"level"`
        Format string `json:"format"`
        File   string `json:"file"`
    } `json:"logging"`
    
    Features struct {
        EnableMetrics bool `json:"enable_metrics"`
        EnableTracing bool `json:"enable_tracing"`
        MaxWorkers    int  `json:"max_workers"`
    } `json:"features"`
}

func main() {
    // Command-line flags
    var (
        configFile    = flag.String("config", "config.json", "configuration file path")
        serverHost    = flag.String("host", "", "server host (overrides config)")
        serverPort    = flag.Int("port", 0, "server port (overrides config)")
        dbHost        = flag.String("db-host", "", "database host (overrides config)")
        logLevel      = flag.String("log-level", "", "log level (overrides config)")
        enableMetrics = flag.Bool("metrics", false, "enable metrics (overrides config)")
        showConfig    = flag.Bool("show-config", false, "show final configuration and exit")
    )
    
    flag.Parse()
    
    // Load configuration file
    config := loadConfig(*configFile)
    
    // Apply command-line flag overrides
    if *serverHost != "" {
        config.Server.Host = *serverHost
    }
    if *serverPort != 0 {
        config.Server.Port = *serverPort
    }
    if *dbHost != "" {
        config.Database.Host = *dbHost
    }
    if *logLevel != "" {
        config.Logging.Level = *logLevel
    }
    if *enableMetrics {
        config.Features.EnableMetrics = true
    }
    
    if *showConfig {
        printConfig(config)
        return
    }
    
    fmt.Printf("Application starting with configuration:\n")
    fmt.Printf("  Server: %s:%d\n", config.Server.Host, config.Server.Port)
    fmt.Printf("  Database: %s:%d/%s\n", config.Database.Host, 
        config.Database.Port, config.Database.Name)
    fmt.Printf("  Log level: %s\n", config.Logging.Level)
    fmt.Printf("  Metrics: %t\n", config.Features.EnableMetrics)
    
    // Validate configuration
    if err := validateConfig(config); err != nil {
        fmt.Fprintf(os.Stderr, "Configuration error: %v\n", err)
        os.Exit(1)
    }
    
    fmt.Println("Configuration validated successfully")
}

func loadConfig(filename string) *Config {
    // Set defaults
    config := &Config{}
    config.Server.Host = "localhost"
    config.Server.Port = 8080
    config.Server.ReadTimeout = 30 * time.Second
    config.Server.WriteTimeout = 30 * time.Second
    config.Database.Host = "localhost"
    config.Database.Port = 5432
    config.Database.User = "postgres"
    config.Database.Name = "myapp"
    config.Logging.Level = "info"
    config.Logging.Format = "json"
    config.Features.MaxWorkers = 4
    
    // Try to load configuration file
    if _, err := os.Stat(filename); err == nil {
        data, err := ioutil.ReadFile(filename)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Warning: could not read config file %s: %v\n", filename, err)
            return config
        }
        
        if err := json.Unmarshal(data, config); err != nil {
            fmt.Fprintf(os.Stderr, "Warning: could not parse config file %s: %v\n", filename, err)
            return config
        }
        
        fmt.Printf("Loaded configuration from %s\n", filename)
    } else {
        fmt.Printf("Config file %s not found, using defaults\n", filename)
    }
    
    return config
}

func printConfig(config *Config) {
    data, err := json.MarshalIndent(config, "", "  ")
    if err != nil {
        fmt.Printf("Error marshaling config: %v\n", err)
        return
    }
    fmt.Printf("Final configuration:\n%s\n", string(data))
}

func validateConfig(config *Config) error {
    if config.Server.Port < 1 || config.Server.Port > 65535 {
        return fmt.Errorf("invalid server port: %d", config.Server.Port)
    }
    
    if config.Database.Port < 1 || config.Database.Port > 65535 {
        return fmt.Errorf("invalid database port: %d", config.Database.Port)
    }
    
    validLogLevels := []string{"debug", "info", "warn", "error"}
    found := false
    for _, level := range validLogLevels {
        if config.Logging.Level == level {
            found = true
            break
        }
    }
    if !found {
        return fmt.Errorf("invalid log level: %s", config.Logging.Level)
    }
    
    if config.Features.MaxWorkers < 1 || config.Features.MaxWorkers > 100 {
        return fmt.Errorf("invalid max workers: %d", config.Features.MaxWorkers)
    }
    
    return nil
}
```

Configuration file integration provides a three-tier configuration system:  
defaults, configuration file values, and command-line flag overrides. This  
pattern is essential for production applications that need flexible deployment  
configurations while maintaining sensible defaults.  

## Interactive prompts with flags

Combining command-line flags with interactive prompts provides a user-friendly  
experience for CLI applications.  

```go
package main

import (
    "bufio"
    "flag"
    "fmt"
    "os"
    "strings"
    "syscall"
    
    "golang.org/x/term"
)

func main() {
    var (
        username = flag.String("username", "", "username for authentication")
        password = flag.String("password", "", "password for authentication")
        server   = flag.String("server", "", "server address")
        batch    = flag.Bool("batch", false, "batch mode (no interactive prompts)")
        force    = flag.Bool("force", false, "skip confirmation prompts")
    )
    
    flag.Parse()
    
    // Interactive prompts for missing required values
    if !*batch {
        if *username == "" {
            *username = promptString("Username: ")
        }
        
        if *password == "" {
            *password = promptPassword("Password: ")
        }
        
        if *server == "" {
            *server = promptStringWithDefault("Server address", "localhost:8080")
        }
        
        // Confirmation prompt
        if !*force {
            fmt.Printf("\nConnect to %s as %s? (y/N): ", *server, *username)
            if !promptConfirm() {
                fmt.Println("Operation cancelled")
                return
            }
        }
    } else {
        // Batch mode validation
        if *username == "" || *password == "" || *server == "" {
            fmt.Println("Error: username, password, and server are required in batch mode")
            os.Exit(1)
        }
    }
    
    fmt.Printf("Connecting to %s as %s...\n", *server, *username)
    fmt.Println("Connection successful!")
}

func promptString(prompt string) string {
    fmt.Print(prompt)
    reader := bufio.NewReader(os.Stdin)
    input, _ := reader.ReadString('\n')
    return strings.TrimSpace(input)
}

func promptStringWithDefault(prompt, defaultValue string) string {
    fmt.Printf("%s [%s]: ", prompt, defaultValue)
    reader := bufio.NewReader(os.Stdin)
    input, _ := reader.ReadString('\n')
    input = strings.TrimSpace(input)
    if input == "" {
        return defaultValue
    }
    return input
}

func promptPassword(prompt string) string {
    fmt.Print(prompt)
    password, err := term.ReadPassword(int(syscall.Stdin))
    if err != nil {
        fmt.Printf("Error reading password: %v\n", err)
        os.Exit(1)
    }
    fmt.Println() // New line after password input
    return string(password)
}

func promptConfirm() bool {
    reader := bufio.NewReader(os.Stdin)
    input, _ := reader.ReadString('\n')
    input = strings.ToLower(strings.TrimSpace(input))
    return input == "y" || input == "yes"
}
```

Interactive prompts enhance user experience by allowing missing values to be  
entered at runtime. This pattern supports both interactive and batch modes,  
making CLI tools suitable for both human users and automated systems.  

## Progress tracking with flags

CLI applications often need to display progress information during long-running  
operations, with flags controlling the display format.  

```go
package main

import (
    "flag"
    "fmt"
    "math/rand"
    "time"
)

type ProgressTracker struct {
    total       int
    current     int
    showBar     bool
    showPercent bool
    showETA     bool
    startTime   time.Time
}

func NewProgressTracker(total int, showBar, showPercent, showETA bool) *ProgressTracker {
    return &ProgressTracker{
        total:       total,
        showBar:     showBar,
        showPercent: showPercent,
        showETA:     showETA,
        startTime:   time.Now(),
    }
}

func (pt *ProgressTracker) Update(current int) {
    pt.current = current
    pt.Display()
}

func (pt *ProgressTracker) Display() {
    fmt.Print("\r") // Return to start of line
    
    if pt.showBar {
        pt.displayProgressBar()
    }
    
    fmt.Printf(" %d/%d", pt.current, pt.total)
    
    if pt.showPercent {
        percentage := float64(pt.current) / float64(pt.total) * 100
        fmt.Printf(" (%.1f%%)", percentage)
    }
    
    if pt.showETA && pt.current > 0 {
        elapsed := time.Since(pt.startTime)
        rate := float64(pt.current) / elapsed.Seconds()
        remaining := float64(pt.total-pt.current) / rate
        eta := time.Duration(remaining) * time.Second
        fmt.Printf(" ETA: %v", eta.Round(time.Second))
    }
}

func (pt *ProgressTracker) displayProgressBar() {
    barWidth := 30
    progress := float64(pt.current) / float64(pt.total)
    filled := int(progress * float64(barWidth))
    
    fmt.Print("[")
    for i := 0; i < barWidth; i++ {
        if i < filled {
            fmt.Print("=")
        } else if i == filled && pt.current < pt.total {
            fmt.Print(">")
        } else {
            fmt.Print(" ")
        }
    }
    fmt.Print("]")
}

func main() {
    var (
        items       = flag.Int("items", 100, "number of items to process")
        showBar     = flag.Bool("progress-bar", true, "show progress bar")
        showPercent = flag.Bool("percent", true, "show percentage")
        showETA     = flag.Bool("eta", true, "show estimated time remaining")
        delay       = flag.Duration("delay", 100*time.Millisecond, "delay between items")
        quiet       = flag.Bool("quiet", false, "suppress progress output")
        verbose     = flag.Bool("verbose", false, "show detailed item information")
    )
    
    flag.Parse()
    
    fmt.Printf("Processing %d items...\n", *items)
    
    var tracker *ProgressTracker
    if !*quiet {
        tracker = NewProgressTracker(*items, *showBar, *showPercent, *showETA)
    }
    
    for i := 1; i <= *items; i++ {
        // Simulate work
        time.Sleep(*delay)
        
        // Simulate occasional longer operations
        if rand.Intn(10) == 0 {
            time.Sleep(*delay * 2)
        }
        
        if *verbose {
            fmt.Printf("\nProcessed item %d\n", i)
        }
        
        if tracker != nil {
            tracker.Update(i)
        }
    }
    
    if !*quiet {
        fmt.Printf("\nCompleted processing %d items in %v\n", 
            *items, time.Since(tracker.startTime).Round(time.Millisecond))
    }
    
    fmt.Println("Operation completed successfully!")
}
```

Progress tracking flags allow users to control how much feedback they receive  
during long operations. This example demonstrates configurable progress bars,  
percentage indicators, and ETA calculations controlled by command-line flags.  

## Signal handling with flags

CLI applications should handle system signals gracefully, with flags controlling  
shutdown behavior and cleanup operations.  

```go
package main

import (
    "context"
    "flag"
    "fmt"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)

type Application struct {
    gracefulShutdown bool
    shutdownTimeout  time.Duration
    workers          int
    verbose          bool
    wg               sync.WaitGroup
    ctx              context.Context
    cancel           context.CancelFunc
}

func main() {
    var (
        workers         = flag.Int("workers", 3, "number of worker goroutines")
        graceful        = flag.Bool("graceful-shutdown", true, "enable graceful shutdown")
        shutdownTimeout = flag.Duration("shutdown-timeout", 10*time.Second, "graceful shutdown timeout")
        verbose         = flag.Bool("verbose", false, "verbose logging")
        workDuration    = flag.Duration("work-duration", 2*time.Second, "simulated work duration")
    )
    
    flag.Parse()
    
    app := &Application{
        gracefulShutdown: *graceful,
        shutdownTimeout:  *shutdownTimeout,
        workers:          *workers,
        verbose:          *verbose,
    }
    
    app.ctx, app.cancel = context.WithCancel(context.Background())
    
    // Set up signal handling
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    
    // Start workers
    fmt.Printf("Starting %d workers...\n", *workers)
    for i := 1; i <= *workers; i++ {
        app.wg.Add(1)
        go app.worker(i, *workDuration)
    }
    
    // Wait for signal
    go func() {
        sig := <-sigChan
        fmt.Printf("\nReceived signal: %v\n", sig)
        
        if app.gracefulShutdown {
            fmt.Printf("Initiating graceful shutdown (timeout: %v)...\n", app.shutdownTimeout)
            app.gracefulShutdownHandler()
        } else {
            fmt.Println("Forcing immediate shutdown...")
            os.Exit(1)
        }
    }()
    
    // Wait for all workers to complete
    app.wg.Wait()
    fmt.Println("All workers completed. Application shutdown.")
}

func (app *Application) worker(id int, workDuration time.Duration) {
    defer app.wg.Done()
    
    if app.verbose {
        fmt.Printf("Worker %d started\n", id)
    }
    
    for {
        select {
        case <-app.ctx.Done():
            if app.verbose {
                fmt.Printf("Worker %d received shutdown signal\n", id)
            }
            return
        default:
            // Simulate work
            if app.verbose {
                fmt.Printf("Worker %d processing...\n", id)
            }
            
            // Use a select with timeout to make work interruptible
            timer := time.NewTimer(workDuration)
            select {
            case <-timer.C:
                // Work completed normally
            case <-app.ctx.Done():
                timer.Stop()
                if app.verbose {
                    fmt.Printf("Worker %d interrupted during work\n", id)
                }
                return
            }
        }
    }
}

func (app *Application) gracefulShutdownHandler() {
    // Signal all workers to stop
    app.cancel()
    
    // Wait for graceful shutdown with timeout
    done := make(chan struct{})
    go func() {
        app.wg.Wait()
        close(done)
    }()
    
    select {
    case <-done:
        fmt.Println("Graceful shutdown completed")
    case <-time.After(app.shutdownTimeout):
        fmt.Printf("Graceful shutdown timeout after %v, forcing exit\n", app.shutdownTimeout)
        os.Exit(1)
    }
}
```

Signal handling flags control how applications respond to system signals  
like SIGINT and SIGTERM. This example shows configurable graceful shutdown  
with timeout handling, allowing clean resource cleanup and proper goroutine  
termination in CLI applications.  

## Logging configuration flags

Comprehensive logging configuration through command-line flags enables  
flexible debugging and monitoring in CLI applications.  

```go
package main

import (
    "flag"
    "fmt"
    "log"
    "os"
    "path/filepath"
    "strings"
    "time"
)

type Logger struct {
    level      string
    format     string
    output     string
    timestamped bool
    colored    bool
    file       *os.File
}

func NewLogger(level, format, output string, timestamped, colored bool) (*Logger, error) {
    logger := &Logger{
        level:       level,
        format:      format,
        timestamped: timestamped,
        colored:     colored,
    }
    
    if output == "stdout" || output == "" {
        logger.output = "stdout"
    } else if output == "stderr" {
        logger.output = "stderr"
    } else {
        // Ensure log directory exists
        dir := filepath.Dir(output)
        if err := os.MkdirAll(dir, 0755); err != nil {
            return nil, fmt.Errorf("failed to create log directory: %v", err)
        }
        
        file, err := os.OpenFile(output, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        if err != nil {
            return nil, fmt.Errorf("failed to open log file: %v", err)
        }
        logger.file = file
        logger.output = output
    }
    
    return logger, nil
}

func (l *Logger) Log(level, message string) {
    if !l.shouldLog(level) {
        return
    }
    
    var output strings.Builder
    
    // Add timestamp
    if l.timestamped {
        timestamp := time.Now().Format("2006-01-02 15:04:05")
        output.WriteString(fmt.Sprintf("[%s] ", timestamp))
    }
    
    // Add level with optional coloring
    levelStr := strings.ToUpper(level)
    if l.colored && (l.output == "stdout" || l.output == "stderr") {
        switch level {
        case "debug":
            levelStr = fmt.Sprintf("\033[36m%s\033[0m", levelStr) // Cyan
        case "info":
            levelStr = fmt.Sprintf("\033[32m%s\033[0m", levelStr)  // Green
        case "warn":
            levelStr = fmt.Sprintf("\033[33m%s\033[0m", levelStr)  // Yellow
        case "error":
            levelStr = fmt.Sprintf("\033[31m%s\033[0m", levelStr) // Red
        }
    }
    
    if l.format == "json" {
        output.WriteString(fmt.Sprintf(`{"timestamp": "%s", "level": "%s", "message": "%s"}`,
            time.Now().Format(time.RFC3339), strings.ToUpper(level), message))
    } else {
        output.WriteString(fmt.Sprintf("[%s] %s", levelStr, message))
    }
    
    output.WriteString("\n")
    
    // Write to appropriate output
    switch l.output {
    case "stdout":
        fmt.Print(output.String())
    case "stderr":
        fmt.Fprint(os.Stderr, output.String())
    default:
        if l.file != nil {
            l.file.WriteString(output.String())
        }
    }
}

func (l *Logger) shouldLog(level string) bool {
    levels := []string{"debug", "info", "warn", "error"}
    currentIndex := -1
    levelIndex := -1
    
    for i, l := range levels {
        if l == l.level {
            currentIndex = i
        }
        if l == level {
            levelIndex = i
        }
    }
    
    return levelIndex >= currentIndex
}

func (l *Logger) Close() {
    if l.file != nil {
        l.file.Close()
    }
}

func main() {
    var (
        logLevel    = flag.String("log-level", "info", "log level: debug, info, warn, error")
        logFormat   = flag.String("log-format", "text", "log format: text, json")
        logOutput   = flag.String("log-output", "stdout", "log output: stdout, stderr, or file path")
        logColor    = flag.Bool("log-color", true, "enable colored log output")
        timestamp   = flag.Bool("timestamp", true, "include timestamps in logs")
        verbose     = flag.Bool("verbose", false, "enable verbose logging (sets level to debug)")
        quiet       = flag.Bool("quiet", false, "quiet mode (sets level to error)")
    )
    
    flag.Parse()
    
    // Handle verbose and quiet flags
    if *verbose {
        *logLevel = "debug"
    } else if *quiet {
        *logLevel = "error"
    }
    
    // Validate log level
    validLevels := []string{"debug", "info", "warn", "error"}
    if !contains(validLevels, *logLevel) {
        fmt.Fprintf(os.Stderr, "Invalid log level: %s\n", *logLevel)
        os.Exit(1)
    }
    
    // Validate log format
    validFormats := []string{"text", "json"}
    if !contains(validFormats, *logFormat) {
        fmt.Fprintf(os.Stderr, "Invalid log format: %s\n", *logFormat)
        os.Exit(1)
    }
    
    // Create logger
    logger, err := NewLogger(*logLevel, *logFormat, *logOutput, *timestamp, *logColor)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Failed to create logger: %v\n", err)
        os.Exit(1)
    }
    defer logger.Close()
    
    fmt.Printf("Logger configuration:\n")
    fmt.Printf("  Level: %s\n", *logLevel)
    fmt.Printf("  Format: %s\n", *logFormat)
    fmt.Printf("  Output: %s\n", *logOutput)
    fmt.Printf("  Colored: %t\n", *logColor)
    fmt.Printf("  Timestamps: %t\n", *timestamp)
    
    // Demonstrate logging at different levels
    logger.Log("debug", "This is a debug message")
    logger.Log("info", "Application started successfully")
    logger.Log("warn", "This is a warning message")
    logger.Log("error", "This is an error message")
    
    // Simulate some application work
    logger.Log("info", "Processing data...")
    time.Sleep(1 * time.Second)
    logger.Log("info", "Data processing completed")
}

func contains(slice []string, item string) bool {
    for _, s := range slice {
        if s == item {
            return true
        }
    }
    return false
}
```

Logging configuration flags provide comprehensive control over application  
output, including log levels, formats, destinations, and styling. This enables  
effective debugging in development and proper monitoring in production  
environments.  

## File processing with flags

CLI applications often process files with various options controlled by  
command-line flags for input validation and output formatting.  

```go
package main

import (
    "bufio"
    "flag"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "regexp"
    "strings"
)

type FileProcessor struct {
    caseSensitive bool
    recursive     bool
    extensions    []string
    pattern       *regexp.Regexp
    outputFormat  string
    includeLineNo bool
    maxResults    int
}

func main() {
    var (
        input         = flag.String("input", ".", "input file or directory")
        pattern       = flag.String("pattern", "", "search pattern (regex)")
        caseSensitive = flag.Bool("case-sensitive", false, "case sensitive matching")
        recursive     = flag.Bool("recursive", false, "search recursively in directories")
        extensions    = flag.String("extensions", "", "comma-separated file extensions (e.g., go,txt)")
        outputFormat  = flag.String("format", "text", "output format: text, json, csv")
        includeLineNo = flag.Bool("line-numbers", false, "include line numbers in output")
        maxResults    = flag.Int("max-results", 100, "maximum number of results")
        outputFile    = flag.String("output", "", "output file (default: stdout)")
        count         = flag.Bool("count", false, "show only count of matches")
        quiet         = flag.Bool("quiet", false, "suppress file processing messages")
    )
    
    flag.Parse()
    
    if *pattern == "" {
        fmt.Println("Error: pattern is required")
        flag.Usage()
        os.Exit(1)
    }
    
    // Compile regex pattern
    var regexFlags int
    if !*caseSensitive {
        regexFlags = regexp.IgnoreCase
    }
    
    regex, err := regexp.Compile(fmt.Sprintf("(?%s)%s", 
        map[bool]string{true: "", false: "i"}[*caseSensitive], *pattern))
    if err != nil {
        fmt.Printf("Error: invalid regex pattern: %v\n", err)
        os.Exit(1)
    }
    
    processor := &FileProcessor{
        caseSensitive: *caseSensitive,
        recursive:     *recursive,
        pattern:       regex,
        outputFormat:  *outputFormat,
        includeLineNo: *includeLineNo,
        maxResults:    *maxResults,
    }
    
    // Parse extensions
    if *extensions != "" {
        processor.extensions = strings.Split(*extensions, ",")
        for i, ext := range processor.extensions {
            processor.extensions[i] = strings.TrimSpace(ext)
            if !strings.HasPrefix(processor.extensions[i], ".") {
                processor.extensions[i] = "." + processor.extensions[i]
            }
        }
    }
    
    // Set up output
    var output io.Writer = os.Stdout
    if *outputFile != "" {
        file, err := os.Create(*outputFile)
        if err != nil {
            fmt.Printf("Error creating output file: %v\n", err)
            os.Exit(1)
        }
        defer file.Close()
        output = file
    }
    
    // Process files
    matches, err := processor.ProcessPath(*input, output, *count, *quiet)
    if err != nil {
        fmt.Printf("Error processing files: %v\n", err)
        os.Exit(1)
    }
    
    if *count {
        fmt.Fprintf(output, "Total matches: %d\n", matches)
    } else if !*quiet {
        fmt.Printf("Processing completed. Found %d matches.\n", matches)
    }
}

func (fp *FileProcessor) ProcessPath(path string, output io.Writer, countOnly, quiet bool) (int, error) {
    info, err := os.Stat(path)
    if err != nil {
        return 0, err
    }
    
    totalMatches := 0
    
    if info.IsDir() {
        err := filepath.Walk(path, func(filePath string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            
            if info.IsDir() {
                if !fp.recursive && filePath != path {
                    return filepath.SkipDir
                }
                return nil
            }
            
            if fp.shouldProcessFile(filePath) {
                matches, err := fp.ProcessFile(filePath, output, countOnly, quiet)
                if err != nil && !quiet {
                    fmt.Printf("Warning: error processing %s: %v\n", filePath, err)
                }
                totalMatches += matches
                
                if totalMatches >= fp.maxResults {
                    return fmt.Errorf("max results reached")
                }
            }
            
            return nil
        })
        
        if err != nil && !strings.Contains(err.Error(), "max results") {
            return totalMatches, err
        }
    } else {
        matches, err := fp.ProcessFile(path, output, countOnly, quiet)
        totalMatches = matches
        if err != nil {
            return totalMatches, err
        }
    }
    
    return totalMatches, nil
}

func (fp *FileProcessor) shouldProcessFile(filePath string) bool {
    if len(fp.extensions) == 0 {
        return true
    }
    
    ext := filepath.Ext(filePath)
    for _, allowedExt := range fp.extensions {
        if ext == allowedExt {
            return true
        }
    }
    return false
}

func (fp *FileProcessor) ProcessFile(filePath string, output io.Writer, countOnly, quiet bool) (int, error) {
    file, err := os.Open(filePath)
    if err != nil {
        return 0, err
    }
    defer file.Close()
    
    if !quiet {
        fmt.Printf("Processing: %s\n", filePath)
    }
    
    scanner := bufio.NewScanner(file)
    lineNumber := 0
    matches := 0
    
    for scanner.Scan() {
        lineNumber++
        line := scanner.Text()
        
        if fp.pattern.MatchString(line) {
            matches++
            
            if !countOnly {
                fp.outputMatch(output, filePath, lineNumber, line)
            }
            
            if matches >= fp.maxResults {
                break
            }
        }
    }
    
    return matches, scanner.Err()
}

func (fp *FileProcessor) outputMatch(output io.Writer, filePath string, lineNumber int, line string) {
    switch fp.outputFormat {
    case "json":
        fmt.Fprintf(output, `{"file": "%s", "line": %d, "content": "%s"}`+"\n", 
            filePath, lineNumber, strings.ReplaceAll(line, `"`, `\"`))
    case "csv":
        fmt.Fprintf(output, `"%s",%d,"%s"`+"\n", 
            filePath, lineNumber, strings.ReplaceAll(line, `"`, `""`))
    default: // text
        if fp.includeLineNo {
            fmt.Fprintf(output, "%s:%d:%s\n", filePath, lineNumber, line)
        } else {
            fmt.Fprintf(output, "%s:%s\n", filePath, line)
        }
    }
}
```

File processing flags enable sophisticated file search and processing  
capabilities with configurable input validation, output formatting, and  
performance controls. This pattern is common in text processing tools  
and file analysis utilities.  

## Network client configuration

Network clients require extensive configuration options for connections,  
timeouts, authentication, and protocol-specific settings.  

```go
package main

import (
    "crypto/tls"
    "flag"
    "fmt"
    "io"
    "net/http"
    "net/url"
    "strings"
    "time"
)

type HTTPClientConfig struct {
    BaseURL           string
    Timeout           time.Duration
    ConnectTimeout    time.Duration
    KeepAlive         time.Duration
    MaxIdleConns      int
    MaxIdleConnsPerHost int
    InsecureSkipVerify bool
    UserAgent         string
    Headers           map[string]string
    ProxyURL          string
    Retries           int
    RetryDelay        time.Duration
}

func main() {
    var (
        baseURL         = flag.String("url", "", "base URL for requests (required)")
        timeout         = flag.Duration("timeout", 30*time.Second, "request timeout")
        connectTimeout  = flag.Duration("connect-timeout", 10*time.Second, "connection timeout")
        keepAlive       = flag.Duration("keep-alive", 30*time.Second, "keep-alive duration")
        maxIdle         = flag.Int("max-idle", 10, "maximum idle connections")
        maxIdlePerHost  = flag.Int("max-idle-per-host", 2, "maximum idle connections per host")
        insecure        = flag.Bool("insecure", false, "skip TLS certificate verification")
        userAgent       = flag.String("user-agent", "go-http-client/1.0", "User-Agent header")
        headers         = flag.String("headers", "", "additional headers (key:value,key:value)")
        proxyURL        = flag.String("proxy", "", "proxy URL")
        retries         = flag.Int("retries", 3, "number of retry attempts")
        retryDelay      = flag.Duration("retry-delay", 1*time.Second, "delay between retries")
        method          = flag.String("method", "GET", "HTTP method")
        body            = flag.String("body", "", "request body")
        output          = flag.String("output", "", "output file for response")
        verbose         = flag.Bool("verbose", false, "verbose output")
        showHeaders     = flag.Bool("show-headers", false, "show response headers")
    )
    
    flag.Parse()
    
    if *baseURL == "" {
        fmt.Println("Error: URL is required")
        flag.Usage()
        os.Exit(1)
    }
    
    // Parse additional headers
    headerMap := make(map[string]string)
    if *headers != "" {
        headerPairs := strings.Split(*headers, ",")
        for _, pair := range headerPairs {
            parts := strings.SplitN(strings.TrimSpace(pair), ":", 2)
            if len(parts) == 2 {
                key := strings.TrimSpace(parts[0])
                value := strings.TrimSpace(parts[1])
                headerMap[key] = value
            }
        }
    }
    
    // Create HTTP client configuration
    config := &HTTPClientConfig{
        BaseURL:             *baseURL,
        Timeout:             *timeout,
        ConnectTimeout:      *connectTimeout,
        KeepAlive:           *keepAlive,
        MaxIdleConns:        *maxIdle,
        MaxIdleConnsPerHost: *maxIdlePerHost,
        InsecureSkipVerify:  *insecure,
        UserAgent:           *userAgent,
        Headers:             headerMap,
        ProxyURL:            *proxyURL,
        Retries:             *retries,
        RetryDelay:          *retryDelay,
    }
    
    client, err := createHTTPClient(config)
    if err != nil {
        fmt.Printf("Error creating HTTP client: %v\n", err)
        os.Exit(1)
    }
    
    // Make HTTP request with retries
    response, err := makeRequestWithRetries(client, config, *method, *body, *verbose)
    if err != nil {
        fmt.Printf("Request failed: %v\n", err)
        os.Exit(1)
    }
    defer response.Body.Close()
    
    // Display response information
    fmt.Printf("Response Status: %s\n", response.Status)
    fmt.Printf("Response Status Code: %d\n", response.StatusCode)
    fmt.Printf("Content Length: %d\n", response.ContentLength)
    
    if *showHeaders {
        fmt.Println("Response Headers:")
        for key, values := range response.Header {
            for _, value := range values {
                fmt.Printf("  %s: %s\n", key, value)
            }
        }
    }
    
    // Handle response body
    var output io.Writer = os.Stdout
    if *output != "" {
        file, err := os.Create(*output)
        if err != nil {
            fmt.Printf("Error creating output file: %v\n", err)
            os.Exit(1)
        }
        defer file.Close()
        output = file
        fmt.Printf("Response body saved to: %s\n", *output)
    } else {
        fmt.Println("Response Body:")
    }
    
    io.Copy(output, response.Body)
}

func createHTTPClient(config *HTTPClientConfig) (*http.Client, error) {
    transport := &http.Transport{
        MaxIdleConns:        config.MaxIdleConns,
        MaxIdleConnsPerHost: config.MaxIdleConnsPerHost,
        IdleConnTimeout:     config.KeepAlive,
        DisableCompression:  false,
    }
    
    // Configure TLS
    if config.InsecureSkipVerify {
        transport.TLSClientConfig = &tls.Config{
            InsecureSkipVerify: true,
        }
    }
    
    // Configure proxy
    if config.ProxyURL != "" {
        proxyURL, err := url.Parse(config.ProxyURL)
        if err != nil {
            return nil, fmt.Errorf("invalid proxy URL: %v", err)
        }
        transport.Proxy = http.ProxyURL(proxyURL)
    }
    
    client := &http.Client{
        Transport: transport,
        Timeout:   config.Timeout,
    }
    
    return client, nil
}

func makeRequestWithRetries(client *http.Client, config *HTTPClientConfig, method, body string, verbose bool) (*http.Response, error) {
    var response *http.Response
    var err error
    
    for attempt := 1; attempt <= config.Retries; attempt++ {
        if verbose {
            fmt.Printf("Attempt %d/%d: %s %s\n", attempt, config.Retries, method, config.BaseURL)
        }
        
        // Create request
        var bodyReader io.Reader
        if body != "" {
            bodyReader = strings.NewReader(body)
        }
        
        req, err := http.NewRequest(method, config.BaseURL, bodyReader)
        if err != nil {
            return nil, fmt.Errorf("error creating request: %v", err)
        }
        
        // Set headers
        req.Header.Set("User-Agent", config.UserAgent)
        for key, value := range config.Headers {
            req.Header.Set(key, value)
        }
        
        // Make request
        response, err = client.Do(req)
        if err == nil && response.StatusCode < 500 {
            // Success or client error (don't retry client errors)
            break
        }
        
        if response != nil {
            response.Body.Close()
        }
        
        if attempt < config.Retries {
            if verbose {
                fmt.Printf("Request failed (attempt %d): %v, retrying in %v\n", 
                    attempt, err, config.RetryDelay)
            }
            time.Sleep(config.RetryDelay)
        }
    }
    
    return response, err
}
```

Network client configuration flags provide comprehensive control over HTTP  
client behavior including timeouts, connection pooling, TLS settings, proxy  
configuration, and retry logic. This pattern is essential for robust network  
clients in production environments.  

## Plugin system with flags

CLI applications can support plugin systems where flags control plugin  
loading, configuration, and execution behavior.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "path/filepath"
    "plugin"
    "sort"
    "strings"
)

// Plugin interface that all plugins must implement
type Plugin interface {
    Name() string
    Version() string
    Description() string
    Execute(args []string) error
}

// PluginManager manages plugin loading and execution
type PluginManager struct {
    plugins     map[string]Plugin
    pluginPaths []string
    enabled     map[string]bool
    verbose     bool
}

func NewPluginManager(pluginPaths []string, enabled map[string]bool, verbose bool) *PluginManager {
    return &PluginManager{
        plugins:     make(map[string]Plugin),
        pluginPaths: pluginPaths,
        enabled:     enabled,
        verbose:     verbose,
    }
}

func (pm *PluginManager) LoadPlugins() error {
    for _, pluginPath := range pm.pluginPaths {
        if pm.verbose {
            fmt.Printf("Scanning plugin directory: %s\n", pluginPath)
        }
        
        err := filepath.Walk(pluginPath, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            
            if strings.HasSuffix(path, ".so") {
                return pm.loadPlugin(path)
            }
            return nil
        })
        
        if err != nil && pm.verbose {
            fmt.Printf("Warning: error scanning %s: %v\n", pluginPath, err)
        }
    }
    
    return nil
}

func (pm *PluginManager) loadPlugin(path string) error {
    if pm.verbose {
        fmt.Printf("Loading plugin: %s\n", path)
    }
    
    plug, err := plugin.Open(path)
    if err != nil {
        return fmt.Errorf("failed to open plugin %s: %v", path, err)
    }
    
    // Look for the NewPlugin symbol
    newPluginFunc, err := plug.Lookup("NewPlugin")
    if err != nil {
        return fmt.Errorf("plugin %s does not export NewPlugin function: %v", path, err)
    }
    
    // Call the NewPlugin function
    newPlugin, ok := newPluginFunc.(func() Plugin)
    if !ok {
        return fmt.Errorf("plugin %s has invalid NewPlugin function signature", path)
    }
    
    pluginInstance := newPlugin()
    name := pluginInstance.Name()
    
    // Check if plugin is enabled
    if pm.enabled != nil {
        if enabled, exists := pm.enabled[name]; exists && !enabled {
            if pm.verbose {
                fmt.Printf("Plugin %s is disabled, skipping\n", name)
            }
            return nil
        }
    }
    
    pm.plugins[name] = pluginInstance
    
    if pm.verbose {
        fmt.Printf("Loaded plugin: %s v%s - %s\n", 
            pluginInstance.Name(), 
            pluginInstance.Version(), 
            pluginInstance.Description())
    }
    
    return nil
}

func (pm *PluginManager) ListPlugins() {
    if len(pm.plugins) == 0 {
        fmt.Println("No plugins loaded")
        return
    }
    
    fmt.Println("Available plugins:")
    
    // Sort plugin names for consistent output
    var names []string
    for name := range pm.plugins {
        names = append(names, name)
    }
    sort.Strings(names)
    
    for _, name := range names {
        plugin := pm.plugins[name]
        status := "enabled"
        if pm.enabled != nil {
            if enabled, exists := pm.enabled[name]; exists && !enabled {
                status = "disabled"
            }
        }
        
        fmt.Printf("  %s v%s [%s]\n    %s\n", 
            plugin.Name(), 
            plugin.Version(), 
            status,
            plugin.Description())
    }
}

func (pm *PluginManager) ExecutePlugin(name string, args []string) error {
    plugin, exists := pm.plugins[name]
    if !exists {
        return fmt.Errorf("plugin %s not found", name)
    }
    
    return plugin.Execute(args)
}

func main() {
    var (
        pluginPaths = flag.String("plugin-paths", "./plugins", "comma-separated plugin directory paths")
        enabledList = flag.String("enabled-plugins", "", "comma-separated list of enabled plugins")
        disabledList = flag.String("disabled-plugins", "", "comma-separated list of disabled plugins")
        listPlugins = flag.Bool("list-plugins", false, "list available plugins")
        verbose     = flag.Bool("verbose", false, "verbose plugin loading")
        pluginName  = flag.String("plugin", "", "plugin to execute")
    )
    
    flag.Parse()
    
    // Parse plugin paths
    paths := strings.Split(*pluginPaths, ",")
    for i, path := range paths {
        paths[i] = strings.TrimSpace(path)
    }
    
    // Parse enabled/disabled plugins
    enabled := make(map[string]bool)
    
    if *enabledList != "" {
        plugins := strings.Split(*enabledList, ",")
        // If enabled list is provided, default all to disabled
        for _, plugin := range plugins {
            enabled[strings.TrimSpace(plugin)] = true
        }
    }
    
    if *disabledList != "" {
        plugins := strings.Split(*disabledList, ",")
        for _, plugin := range plugins {
            enabled[strings.TrimSpace(plugin)] = false
        }
    }
    
    // Create plugin manager
    var enabledMap map[string]bool
    if len(enabled) > 0 {
        enabledMap = enabled
    }
    
    manager := NewPluginManager(paths, enabledMap, *verbose)
    
    // Load plugins
    if err := manager.LoadPlugins(); err != nil {
        fmt.Printf("Error loading plugins: %v\n", err)
        os.Exit(1)
    }
    
    // Handle list request
    if *listPlugins {
        manager.ListPlugins()
        return
    }
    
    // Execute specific plugin
    if *pluginName != "" {
        args := flag.Args()
        if err := manager.ExecutePlugin(*pluginName, args); err != nil {
            fmt.Printf("Plugin execution failed: %v\n", err)
            os.Exit(1)
        }
        return
    }
    
    // Show usage if no specific action requested
    fmt.Println("Plugin System")
    fmt.Println("Use -list-plugins to see available plugins")
    fmt.Println("Use -plugin <name> to execute a specific plugin")
    manager.ListPlugins()
}
```

Plugin system flags enable dynamic loading and configuration of plugins  
with fine-grained control over which plugins are enabled, where they're  
loaded from, and how they're executed. This pattern is common in extensible  
CLI applications and build systems.  

## Database connection flags

CLI applications that interact with databases require comprehensive  
configuration for connections, query parameters, and output formatting.  

```go
package main

import (
    "database/sql"
    "flag"
    "fmt"
    "net/url"
    "os"
    "strconv"
    "strings"
    "time"
    
    _ "github.com/lib/pq" // PostgreSQL driver (example)
)

type DatabaseConfig struct {
    Driver          string
    Host            string
    Port            int
    Database        string
    Username        string
    Password        string
    SSLMode         string
    ConnectTimeout  time.Duration
    MaxOpenConns    int
    MaxIdleConns    int
    ConnMaxLifetime time.Duration
    QueryTimeout    time.Duration
}

func main() {
    var (
        driver          = flag.String("driver", "postgres", "database driver")
        host            = flag.String("host", "localhost", "database host")
        port            = flag.Int("port", 5432, "database port")
        database        = flag.String("database", "", "database name (required)")
        username        = flag.String("username", "postgres", "database username")
        password        = flag.String("password", "", "database password")
        sslMode         = flag.String("ssl-mode", "disable", "SSL mode: disable, require, verify-ca, verify-full")
        connectTimeout  = flag.Duration("connect-timeout", 10*time.Second, "connection timeout")
        queryTimeout    = flag.Duration("query-timeout", 30*time.Second, "query timeout")
        maxOpenConns    = flag.Int("max-open-conns", 10, "maximum open connections")
        maxIdleConns    = flag.Int("max-idle-conns", 5, "maximum idle connections")
        connMaxLifetime = flag.Duration("conn-max-lifetime", 1*time.Hour, "connection maximum lifetime")
        query           = flag.String("query", "", "SQL query to execute")
        queryFile       = flag.String("query-file", "", "file containing SQL query")
        output          = flag.String("output", "table", "output format: table, json, csv")
        headers         = flag.Bool("headers", true, "include column headers in output")
        delimiter       = flag.String("delimiter", ",", "CSV delimiter")
        verbose         = flag.Bool("verbose", false, "verbose output")
    )
    
    flag.Parse()
    
    if *database == "" {
        fmt.Println("Error: database name is required")
        flag.Usage()
        os.Exit(1)
    }
    
    // Validate SSL mode
    validSSLModes := []string{"disable", "require", "verify-ca", "verify-full"}
    if !contains(validSSLModes, *sslMode) {
        fmt.Printf("Error: invalid SSL mode '%s'\n", *sslMode)
        fmt.Printf("Valid modes: %s\n", strings.Join(validSSLModes, ", "))
        os.Exit(1)
    }
    
    // Create database config
    config := &DatabaseConfig{
        Driver:          *driver,
        Host:            *host,
        Port:            *port,
        Database:        *database,
        Username:        *username,
        Password:        *password,
        SSLMode:         *sslMode,
        ConnectTimeout:  *connectTimeout,
        MaxOpenConns:    *maxOpenConns,
        MaxIdleConns:    *maxIdleConns,
        ConnMaxLifetime: *connMaxLifetime,
        QueryTimeout:    *queryTimeout,
    }
    
    // Get password from environment if not provided
    if config.Password == "" {
        if envPassword := os.Getenv("DB_PASSWORD"); envPassword != "" {
            config.Password = envPassword
        }
    }
    
    if *verbose {
        fmt.Printf("Database configuration:\n")
        fmt.Printf("  Driver: %s\n", config.Driver)
        fmt.Printf("  Host: %s\n", config.Host)
        fmt.Printf("  Port: %d\n", config.Port)
        fmt.Printf("  Database: %s\n", config.Database)
        fmt.Printf("  Username: %s\n", config.Username)
        fmt.Printf("  SSL Mode: %s\n", config.SSLMode)
        fmt.Printf("  Connect Timeout: %v\n", config.ConnectTimeout)
        fmt.Printf("  Query Timeout: %v\n", config.QueryTimeout)
        fmt.Printf("  Max Open Connections: %d\n", config.MaxOpenConns)
        fmt.Printf("  Max Idle Connections: %d\n", config.MaxIdleConns)
        fmt.Printf("  Connection Max Lifetime: %v\n", config.ConnMaxLifetime)
    }
    
    // Connect to database
    db, err := connectToDatabase(config)
    if err != nil {
        fmt.Printf("Error connecting to database: %v\n", err)
        os.Exit(1)
    }
    defer db.Close()
    
    if *verbose {
        fmt.Println("Successfully connected to database")
    }
    
    // Get query from file or command line
    var sqlQuery string
    if *queryFile != "" {
        queryBytes, err := os.ReadFile(*queryFile)
        if err != nil {
            fmt.Printf("Error reading query file: %v\n", err)
            os.Exit(1)
        }
        sqlQuery = string(queryBytes)
    } else if *query != "" {
        sqlQuery = *query
    } else {
        fmt.Println("Error: either -query or -query-file must be provided")
        os.Exit(1)
    }
    
    // Execute query and output results
    if err := executeQuery(db, sqlQuery, *output, *headers, *delimiter, *verbose); err != nil {
        fmt.Printf("Error executing query: %v\n", err)
        os.Exit(1)
    }
}

func connectToDatabase(config *DatabaseConfig) (*sql.DB, error) {
    // Build connection string based on driver
    var connectionString string
    
    switch config.Driver {
    case "postgres":
        connectionString = buildPostgresConnectionString(config)
    default:
        return nil, fmt.Errorf("unsupported database driver: %s", config.Driver)
    }
    
    db, err := sql.Open(config.Driver, connectionString)
    if err != nil {
        return nil, fmt.Errorf("failed to open database connection: %v", err)
    }
    
    // Configure connection pool
    db.SetMaxOpenConns(config.MaxOpenConns)
    db.SetMaxIdleConns(config.MaxIdleConns)
    db.SetConnMaxLifetime(config.ConnMaxLifetime)
    
    // Test connection
    if err := db.Ping(); err != nil {
        return nil, fmt.Errorf("failed to ping database: %v", err)
    }
    
    return db, nil
}

func buildPostgresConnectionString(config *DatabaseConfig) string {
    values := url.Values{}
    values.Set("host", config.Host)
    values.Set("port", strconv.Itoa(config.Port))
    values.Set("dbname", config.Database)
    values.Set("user", config.Username)
    values.Set("sslmode", config.SSLMode)
    values.Set("connect_timeout", strconv.Itoa(int(config.ConnectTimeout.Seconds())))
    
    if config.Password != "" {
        values.Set("password", config.Password)
    }
    
    return values.Encode()
}

func executeQuery(db *sql.DB, query, outputFormat string, includeHeaders bool, delimiter string, verbose bool) error {
    if verbose {
        fmt.Printf("Executing query: %s\n", strings.TrimSpace(query))
    }
    
    rows, err := db.Query(query)
    if err != nil {
        return fmt.Errorf("query execution failed: %v", err)
    }
    defer rows.Close()
    
    // Get column information
    columns, err := rows.Columns()
    if err != nil {
        return fmt.Errorf("failed to get column information: %v", err)
    }
    
    // Output headers
    if includeHeaders {
        switch outputFormat {
        case "table":
            fmt.Printf("| %s |\n", strings.Join(columns, " | "))
            fmt.Printf("|%s|\n", strings.Repeat("-", len(strings.Join(columns, " | "))+2))
        case "csv":
            fmt.Printf("%s\n", strings.Join(columns, delimiter))
        case "json":
            fmt.Println("[")
        }
    } else if outputFormat == "json" {
        fmt.Println("[")
    }
    
    // Process rows
    values := make([]interface{}, len(columns))
    valuePtrs := make([]interface{}, len(columns))
    for i := range values {
        valuePtrs[i] = &values[i]
    }
    
    rowCount := 0
    for rows.Next() {
        if err := rows.Scan(valuePtrs...); err != nil {
            return fmt.Errorf("error scanning row: %v", err)
        }
        
        // Convert values to strings
        stringValues := make([]string, len(values))
        for i, val := range values {
            if val != nil {
                stringValues[i] = fmt.Sprintf("%v", val)
            } else {
                stringValues[i] = "NULL"
            }
        }
        
        // Output row
        switch outputFormat {
        case "table":
            fmt.Printf("| %s |\n", strings.Join(stringValues, " | "))
        case "csv":
            fmt.Printf("%s\n", strings.Join(stringValues, delimiter))
        case "json":
            if rowCount > 0 {
                fmt.Print(",")
            }
            fmt.Print("\n  {")
            for i, col := range columns {
                if i > 0 {
                    fmt.Print(",")
                }
                fmt.Printf("\"%s\": \"%s\"", col, stringValues[i])
            }
            fmt.Print("}")
        }
        
        rowCount++
    }
    
    if outputFormat == "json" {
        if rowCount > 0 {
            fmt.Print("\n")
        }
        fmt.Println("]")
    }
    
    if verbose {
        fmt.Printf("Query completed. Rows returned: %d\n", rowCount)
    }
    
    return rows.Err()
}

func contains(slice []string, item string) bool {
    for _, s := range slice {
        if s == item {
            return true
        }
    }
    return false
}
```

Database connection flags provide comprehensive configuration for database  
clients including connection parameters, pool settings, query execution  
options, and output formatting. This pattern is essential for CLI database  
tools and data processing applications.  

## Performance monitoring flags

CLI applications often need performance monitoring capabilities with  
configurable metrics collection and reporting options.  

```go
package main

import (
    "flag"
    "fmt"
    "runtime"
    "sync"
    "time"
)

type PerformanceMonitor struct {
    enabled         bool
    interval        time.Duration
    showMemory      bool
    showGoroutines  bool
    showGC          bool
    showCPU         bool
    outputFile      string
    metrics         []MetricSnapshot
    mutex           sync.RWMutex
    stopChan        chan bool
}

type MetricSnapshot struct {
    Timestamp      time.Time
    MemoryUsage    uint64
    GoroutineCount int
    GCCycles       uint32
    CPUPercent     float64
}

func NewPerformanceMonitor(enabled bool, interval time.Duration, showMemory, showGoroutines, showGC, showCPU bool, outputFile string) *PerformanceMonitor {
    return &PerformanceMonitor{
        enabled:        enabled,
        interval:       interval,
        showMemory:     showMemory,
        showGoroutines: showGoroutines,
        showGC:         showGC,
        showCPU:        showCPU,
        outputFile:     outputFile,
        metrics:        make([]MetricSnapshot, 0),
        stopChan:       make(chan bool),
    }
}

func (pm *PerformanceMonitor) Start() {
    if !pm.enabled {
        return
    }
    
    go pm.monitorLoop()
}

func (pm *PerformanceMonitor) Stop() {
    if !pm.enabled {
        return
    }
    
    close(pm.stopChan)
}

func (pm *PerformanceMonitor) monitorLoop() {
    ticker := time.NewTicker(pm.interval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            pm.collectMetrics()
        case <-pm.stopChan:
            return
        }
    }
}

func (pm *PerformanceMonitor) collectMetrics() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    snapshot := MetricSnapshot{
        Timestamp:      time.Now(),
        MemoryUsage:    m.Alloc,
        GoroutineCount: runtime.NumGoroutine(),
        GCCycles:       m.NumGC,
    }
    
    pm.mutex.Lock()
    pm.metrics = append(pm.metrics, snapshot)
    pm.mutex.Unlock()
    
    pm.displayMetrics(snapshot)
}

func (pm *PerformanceMonitor) displayMetrics(snapshot MetricSnapshot) {
    fmt.Printf("[%s] ", snapshot.Timestamp.Format("15:04:05"))
    
    if pm.showMemory {
        fmt.Printf("Memory: %s ", formatBytes(snapshot.MemoryUsage))
    }
    
    if pm.showGoroutines {
        fmt.Printf("Goroutines: %d ", snapshot.GoroutineCount)
    }
    
    if pm.showGC {
        fmt.Printf("GC Cycles: %d ", snapshot.GCCycles)
    }
    
    fmt.Println()
}

func (pm *PerformanceMonitor) GetSummary() {
    pm.mutex.RLock()
    defer pm.mutex.RUnlock()
    
    if len(pm.metrics) == 0 {
        fmt.Println("No performance metrics collected")
        return
    }
    
    fmt.Println("\n=== Performance Summary ===")
    
    first := pm.metrics[0]
    last := pm.metrics[len(pm.metrics)-1]
    duration := last.Timestamp.Sub(first.Timestamp)
    
    fmt.Printf("Monitoring Duration: %v\n", duration)
    fmt.Printf("Samples Collected: %d\n", len(pm.metrics))
    
    if pm.showMemory {
        minMemory := pm.metrics[0].MemoryUsage
        maxMemory := pm.metrics[0].MemoryUsage
        var totalMemory uint64
        
        for _, metric := range pm.metrics {
            if metric.MemoryUsage < minMemory {
                minMemory = metric.MemoryUsage
            }
            if metric.MemoryUsage > maxMemory {
                maxMemory = metric.MemoryUsage
            }
            totalMemory += metric.MemoryUsage
        }
        
        avgMemory := totalMemory / uint64(len(pm.metrics))
        
        fmt.Printf("Memory Usage:\n")
        fmt.Printf("  Min: %s\n", formatBytes(minMemory))
        fmt.Printf("  Max: %s\n", formatBytes(maxMemory))
        fmt.Printf("  Avg: %s\n", formatBytes(avgMemory))
    }
    
    if pm.showGoroutines {
        minGoroutines := pm.metrics[0].GoroutineCount
        maxGoroutines := pm.metrics[0].GoroutineCount
        totalGoroutines := 0
        
        for _, metric := range pm.metrics {
            if metric.GoroutineCount < minGoroutines {
                minGoroutines = metric.GoroutineCount
            }
            if metric.GoroutineCount > maxGoroutines {
                maxGoroutines = metric.GoroutineCount
            }
            totalGoroutines += metric.GoroutineCount
        }
        
        avgGoroutines := float64(totalGoroutines) / float64(len(pm.metrics))
        
        fmt.Printf("Goroutines:\n")
        fmt.Printf("  Min: %d\n", minGoroutines)
        fmt.Printf("  Max: %d\n", maxGoroutines)
        fmt.Printf("  Avg: %.1f\n", avgGoroutines)
    }
    
    if pm.showGC {
        gcCycles := last.GCCycles - first.GCCycles
        fmt.Printf("GC Cycles: %d\n", gcCycles)
    }
}

func formatBytes(bytes uint64) string {
    const (
        KB = 1024
        MB = KB * 1024
        GB = MB * 1024
    )
    
    switch {
    case bytes >= GB:
        return fmt.Sprintf("%.2f GB", float64(bytes)/GB)
    case bytes >= MB:
        return fmt.Sprintf("%.2f MB", float64(bytes)/MB)
    case bytes >= KB:
        return fmt.Sprintf("%.2f KB", float64(bytes)/KB)
    default:
        return fmt.Sprintf("%d B", bytes)
    }
}

// Simulate some work to demonstrate monitoring
func simulateWork(duration time.Duration, workers int) {
    var wg sync.WaitGroup
    
    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            endTime := time.Now().Add(duration)
            for time.Now().Before(endTime) {
                // Allocate some memory
                data := make([]byte, 1024*1024) // 1MB
                _ = data
                
                // Do some CPU work
                for j := 0; j < 10000; j++ {
                    _ = j * j
                }
                
                time.Sleep(100 * time.Millisecond)
            }
        }(i)
    }
    
    wg.Wait()
}

func main() {
    var (
        enableMonitoring = flag.Bool("monitor", false, "enable performance monitoring")
        monitorInterval  = flag.Duration("monitor-interval", 1*time.Second, "monitoring interval")
        showMemory       = flag.Bool("show-memory", true, "show memory usage")
        showGoroutines   = flag.Bool("show-goroutines", true, "show goroutine count")
        showGC           = flag.Bool("show-gc", true, "show garbage collection stats")
        showCPU          = flag.Bool("show-cpu", false, "show CPU usage (not implemented)")
        outputFile       = flag.String("monitor-output", "", "output file for metrics")
        workDuration     = flag.Duration("work-duration", 10*time.Second, "duration of simulated work")
        workers          = flag.Int("workers", 3, "number of worker goroutines")
        summary          = flag.Bool("summary", true, "show performance summary at end")
    )
    
    flag.Parse()
    
    fmt.Printf("Performance Monitoring Demo\n")
    fmt.Printf("Work duration: %v\n", *workDuration)
    fmt.Printf("Workers: %d\n", *workers)
    fmt.Printf("Monitoring enabled: %t\n", *enableMonitoring)
    
    if *enableMonitoring {
        fmt.Printf("Monitor interval: %v\n", *monitorInterval)
    }
    
    // Create and start performance monitor
    monitor := NewPerformanceMonitor(
        *enableMonitoring,
        *monitorInterval,
        *showMemory,
        *showGoroutines,
        *showGC,
        *showCPU,
        *outputFile,
    )
    
    monitor.Start()
    
    fmt.Printf("\nStarting work simulation...\n")
    simulateWork(*workDuration, *workers)
    fmt.Printf("Work simulation completed.\n")
    
    monitor.Stop()
    
    if *enableMonitoring && *summary {
        monitor.GetSummary()
    }
}
```

Performance monitoring flags enable comprehensive runtime monitoring of  
CLI applications with configurable metrics collection, display options,  
and reporting. This pattern is valuable for performance analysis and  
optimization of long-running CLI processes.  

## Third-party CLI library integration

Modern Go CLI development often uses third-party libraries like Cobra  
for enhanced command-line interfaces with subcommands and rich features.  

```go
package main

import (
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
    
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var (
    cfgFile string
    verbose bool
    output  string
)

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
    Use:   "myapp",
    Short: "A comprehensive CLI application built with Cobra",
    Long: `MyApp is a CLI application that demonstrates advanced command-line
interface patterns using the Cobra library. It supports multiple subcommands,
configuration files, environment variables, and rich flag handling.`,
    
    PersistentPreRun: func(cmd *cobra.Command, args []string) {
        if verbose {
            fmt.Printf("Executing command: %s\n", cmd.Name())
            fmt.Printf("Arguments: %v\n", args)
        }
    },
}

// serverCmd represents the server command
var serverCmd = &cobra.Command{
    Use:   "server",
    Short: "Start the application server",
    Long: `Start the application server with the specified configuration.
The server will listen for incoming connections and process requests.`,
    
    Run: func(cmd *cobra.Command, args []string) {
        host := viper.GetString("server.host")
        port := viper.GetInt("server.port")
        tls := viper.GetBool("server.tls")
        
        fmt.Printf("Starting server on %s:%d\n", host, port)
        fmt.Printf("TLS enabled: %t\n", tls)
        fmt.Printf("Output format: %s\n", output)
        
        // Simulate server startup
        fmt.Println("Server started successfully!")
    },
}

// clientCmd represents the client command
var clientCmd = &cobra.Command{
    Use:   "client",
    Short: "Run client operations",
    Long: `Execute client operations against a remote server.
Supports various operations like get, post, and delete.`,
}

// clientGetCmd represents the client get command
var clientGetCmd = &cobra.Command{
    Use:   "get [resource]",
    Short: "Get a resource from the server",
    Args:  cobra.ExactArgs(1),
    
    Run: func(cmd *cobra.Command, args []string) {
        resource := args[0]
        serverURL := viper.GetString("client.server_url")
        timeout := viper.GetDuration("client.timeout")
        
        fmt.Printf("Getting resource: %s\n", resource)
        fmt.Printf("Server URL: %s\n", serverURL)
        fmt.Printf("Timeout: %v\n", timeout)
        fmt.Printf("Output format: %s\n", output)
        
        // Simulate client operation
        fmt.Printf("Successfully retrieved %s\n", resource)
    },
}

// clientPostCmd represents the client post command
var clientPostCmd = &cobra.Command{
    Use:   "post [resource]",
    Short: "Create a new resource on the server",
    Args:  cobra.ExactArgs(1),
    
    Run: func(cmd *cobra.Command, args []string) {
        resource := args[0]
        data, _ := cmd.Flags().GetString("data")
        
        fmt.Printf("Creating resource: %s\n", resource)
        if data != "" {
            fmt.Printf("Data: %s\n", data)
        }
        
        // Simulate client operation
        fmt.Printf("Successfully created %s\n", resource)
    },
}

// configCmd represents the config command
var configCmd = &cobra.Command{
    Use:   "config",
    Short: "Manage application configuration",
    Long: `Manage application configuration including viewing current settings,
updating values, and validating configuration files.`,
}

// configShowCmd shows current configuration
var configShowCmd = &cobra.Command{
    Use:   "show",
    Short: "Show current configuration",
    
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("Current configuration:")
        
        allSettings := viper.AllSettings()
        for key, value := range allSettings {
            fmt.Printf("  %s: %v\n", key, value)
        }
    },
}

// configSetCmd sets a configuration value
var configSetCmd = &cobra.Command{
    Use:   "set [key] [value]",
    Short: "Set a configuration value",
    Args:  cobra.ExactArgs(2),
    
    Run: func(cmd *cobra.Command, args []string) {
        key := args[0]
        value := args[1]
        
        // Try to convert value to appropriate type
        if intValue, err := strconv.Atoi(value); err == nil {
            viper.Set(key, intValue)
        } else if boolValue, err := strconv.ParseBool(value); err == nil {
            viper.Set(key, boolValue)
        } else if durValue, err := time.ParseDuration(value); err == nil {
            viper.Set(key, durValue)
        } else {
            viper.Set(key, value)
        }
        
        fmt.Printf("Set %s = %v\n", key, viper.Get(key))
        
        // Save to config file if specified
        if cfgFile != "" {
            if err := viper.WriteConfig(); err != nil {
                fmt.Printf("Error saving configuration: %v\n", err)
            } else {
                fmt.Printf("Configuration saved to %s\n", cfgFile)
            }
        }
    },
}

func init() {
    cobra.OnInitialize(initConfig)
    
    // Global flags
    rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.myapp.yaml)")
    rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false, "verbose output")
    rootCmd.PersistentFlags().StringVarP(&output, "output", "o", "text", "output format (text, json, yaml)")
    
    // Server command flags
    serverCmd.Flags().String("host", "localhost", "server host")
    serverCmd.Flags().IntP("port", "p", 8080, "server port")
    serverCmd.Flags().Bool("tls", false, "enable TLS")
    serverCmd.Flags().Duration("read-timeout", 30*time.Second, "read timeout")
    serverCmd.Flags().Duration("write-timeout", 30*time.Second, "write timeout")
    
    // Bind server flags to viper
    viper.BindPFlag("server.host", serverCmd.Flags().Lookup("host"))
    viper.BindPFlag("server.port", serverCmd.Flags().Lookup("port"))
    viper.BindPFlag("server.tls", serverCmd.Flags().Lookup("tls"))
    viper.BindPFlag("server.read_timeout", serverCmd.Flags().Lookup("read-timeout"))
    viper.BindPFlag("server.write_timeout", serverCmd.Flags().Lookup("write-timeout"))
    
    // Client command flags
    clientCmd.PersistentFlags().String("server-url", "http://localhost:8080", "server URL")
    clientCmd.PersistentFlags().Duration("timeout", 30*time.Second, "request timeout")
    clientCmd.PersistentFlags().Int("retries", 3, "number of retries")
    
    // Bind client flags to viper
    viper.BindPFlag("client.server_url", clientCmd.PersistentFlags().Lookup("server-url"))
    viper.BindPFlag("client.timeout", clientCmd.PersistentFlags().Lookup("timeout"))
    viper.BindPFlag("client.retries", clientCmd.PersistentFlags().Lookup("retries"))
    
    // Client post specific flags
    clientPostCmd.Flags().StringP("data", "d", "", "data to send with request")
    clientPostCmd.Flags().String("content-type", "application/json", "content type header")
    
    // Add subcommands
    rootCmd.AddCommand(serverCmd)
    rootCmd.AddCommand(clientCmd)
    rootCmd.AddCommand(configCmd)
    
    clientCmd.AddCommand(clientGetCmd)
    clientCmd.AddCommand(clientPostCmd)
    
    configCmd.AddCommand(configShowCmd)
    configCmd.AddCommand(configSetCmd)
}

func initConfig() {
    if cfgFile != "" {
        viper.SetConfigFile(cfgFile)
    } else {
        home, err := os.UserHomeDir()
        cobra.CheckErr(err)
        
        viper.AddConfigPath(home)
        viper.AddConfigPath(".")
        viper.SetConfigType("yaml")
        viper.SetConfigName(".myapp")
    }
    
    // Environment variable support
    viper.SetEnvPrefix("MYAPP")
    viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
    viper.AutomaticEnv()
    
    // Read configuration file
    if err := viper.ReadInConfig(); err == nil && verbose {
        fmt.Printf("Using config file: %s\n", viper.ConfigFileUsed())
    }
    
    // Set defaults
    viper.SetDefault("server.host", "localhost")
    viper.SetDefault("server.port", 8080)
    viper.SetDefault("server.tls", false)
    viper.SetDefault("client.server_url", "http://localhost:8080")
    viper.SetDefault("client.timeout", "30s")
    viper.SetDefault("client.retries", 3)
}

func main() {
    if err := rootCmd.Execute(); err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
}
```

Third-party CLI libraries like Cobra provide powerful features including  
subcommands, flag inheritance, configuration file integration, environment  
variable support, and automatic help generation. This pattern enables  
building sophisticated CLI applications with rich user interfaces and  
flexible configuration options.  

## Testing CLI applications

CLI applications require comprehensive testing strategies that cover  
flag parsing, command execution, and output validation.  

```go
package main

import (
    "bytes"
    "flag"
    "fmt"
    "io"
    "os"
    "strings"
    "testing"
)

// CLIApp represents our CLI application for testing
type CLIApp struct {
    output io.Writer
    input  io.Reader
    args   []string
}

func NewCLIApp(output io.Writer, input io.Reader, args []string) *CLIApp {
    return &CLIApp{
        output: output,
        input:  input,
        args:   args,
    }
}

func (app *CLIApp) Run() error {
    flagSet := flag.NewFlagSet("testapp", flag.ContinueOnError)
    flagSet.SetOutput(app.output)
    
    var (
        verbose = flagSet.Bool("verbose", false, "verbose output")
        name    = flagSet.String("name", "World", "name to greet")
        count   = flagSet.Int("count", 1, "number of greetings")
        format  = flagSet.String("format", "text", "output format")
    )
    
    if err := flagSet.Parse(app.args); err != nil {
        return err
    }
    
    if *verbose {
        fmt.Fprintf(app.output, "Generating %d greeting(s) for %s\n", *count, *name)
    }
    
    for i := 0; i < *count; i++ {
        switch *format {
        case "json":
            fmt.Fprintf(app.output, `{"greeting": "Hello %s!", "number": %d}`+"\n", *name, i+1)
        case "xml":
            fmt.Fprintf(app.output, `<greeting number="%d">Hello %s!</greeting>`+"\n", i+1, *name)
        default:
            fmt.Fprintf(app.output, "Hello %s!\n", *name)
        }
    }
    
    return nil
}

// Test functions
func TestCLIAppBasicGreeting(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{})
    
    err := app.Run()
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    expected := "Hello World!\n"
    if output.String() != expected {
        t.Errorf("Expected %q, got %q", expected, output.String())
    }
}

func TestCLIAppCustomName(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{"-name", "Alice"})
    
    err := app.Run()
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    expected := "Hello Alice!\n"
    if output.String() != expected {
        t.Errorf("Expected %q, got %q", expected, output.String())
    }
}

func TestCLIAppMultipleGreetings(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{"-name", "Bob", "-count", "3"})
    
    err := app.Run()
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    lines := strings.Split(strings.TrimSpace(output.String()), "\n")
    if len(lines) != 3 {
        t.Errorf("Expected 3 lines, got %d", len(lines))
    }
    
    for _, line := range lines {
        if line != "Hello Bob!" {
            t.Errorf("Expected 'Hello Bob!', got %q", line)
        }
    }
}

func TestCLIAppJSONFormat(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{"-name", "Charlie", "-format", "json"})
    
    err := app.Run()
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    expected := `{"greeting": "Hello Charlie!", "number": 1}` + "\n"
    if output.String() != expected {
        t.Errorf("Expected %q, got %q", expected, output.String())
    }
}

func TestCLIAppVerboseMode(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{"-verbose", "-name", "Dave"})
    
    err := app.Run()
    if err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
    
    outputStr := output.String()
    if !strings.Contains(outputStr, "Generating 1 greeting(s) for Dave") {
        t.Errorf("Expected verbose output, got %q", outputStr)
    }
    if !strings.Contains(outputStr, "Hello Dave!") {
        t.Errorf("Expected greeting output, got %q", outputStr)
    }
}

func TestCLIAppInvalidFlag(t *testing.T) {
    var output bytes.Buffer
    app := NewCLIApp(&output, nil, []string{"-invalid-flag"})
    
    err := app.Run()
    if err == nil {
        t.Error("Expected error for invalid flag, got nil")
    }
    
    outputStr := output.String()
    if !strings.Contains(outputStr, "flag provided but not defined") {
        t.Errorf("Expected error message about undefined flag, got %q", outputStr)
    }
}

// Example of testing with temporary files
func TestCLIAppWithFile(t *testing.T) {
    // Create temporary file
    tmpFile, err := os.CreateTemp("", "cli-test-*.txt")
    if err != nil {
        t.Fatalf("Failed to create temp file: %v", err)
    }
    defer os.Remove(tmpFile.Name())
    defer tmpFile.Close()
    
    // Write test data
    testData := "Alice\nBob\nCharlie\n"
    if _, err := tmpFile.WriteString(testData); err != nil {
        t.Fatalf("Failed to write test data: %v", err)
    }
    
    // Reset file pointer
    tmpFile.Seek(0, 0)
    
    // Test reading from file
    var output bytes.Buffer
    app := NewCLIApp(&output, tmpFile, []string{"-format", "json"})
    
    // This would be implemented to read names from input
    // For demonstration, we'll just test the concept
    if err := app.Run(); err != nil {
        t.Fatalf("Expected no error, got: %v", err)
    }
}

func main() {
    // Run the CLI application
    app := NewCLIApp(os.Stdout, os.Stdin, os.Args[1:])
    if err := app.Run(); err != nil {
        fmt.Fprintf(os.Stderr, "Error: %v\n", err)
        os.Exit(1)
    }
}
```

Testing CLI applications requires isolating input/output, creating testable  
interfaces, and validating both successful operations and error conditions.  
This example demonstrates comprehensive testing patterns including output  
capture, flag validation, and file-based testing scenarios.  

## Internationalization and localization

CLI applications often need to support multiple languages and locales  
with configurable message formatting and cultural preferences.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
)

// MessageCatalog stores localized messages
type MessageCatalog map[string]map[string]string

// Localizer provides localization services
type Localizer struct {
    catalog    MessageCatalog
    locale     string
    fallback   string
    timeFormat string
    dateFormat string
}

func NewLocalizer(catalog MessageCatalog, locale, fallback string) *Localizer {
    l := &Localizer{
        catalog:  catalog,
        locale:   locale,
        fallback: fallback,
    }
    
    // Set locale-specific formats
    switch locale {
    case "en-US":
        l.timeFormat = "3:04 PM"
        l.dateFormat = "1/2/2006"
    case "en-GB":
        l.timeFormat = "15:04"
        l.dateFormat = "2/1/2006"
    case "de-DE":
        l.timeFormat = "15:04"
        l.dateFormat = "2.1.2006"
    case "fr-FR":
        l.timeFormat = "15:04"
        l.dateFormat = "2/1/2006"
    case "ja-JP":
        l.timeFormat = "15:04"
        l.dateFormat = "2006/1/2"
    default:
        l.timeFormat = "15:04:05"
        l.dateFormat = "2006-01-02"
    }
    
    return l
}

func (l *Localizer) GetMessage(key string, args ...interface{}) string {
    // Try to get message in current locale
    if messages, exists := l.catalog[l.locale]; exists {
        if message, exists := messages[key]; exists {
            if len(args) > 0 {
                return fmt.Sprintf(message, args...)
            }
            return message
        }
    }
    
    // Fallback to default locale
    if l.fallback != l.locale {
        if messages, exists := l.catalog[l.fallback]; exists {
            if message, exists := messages[key]; exists {
                if len(args) > 0 {
                    return fmt.Sprintf(message, args...)
                }
                return message
            }
        }
    }
    
    // Return key if message not found
    return fmt.Sprintf("[%s]", key)
}

func (l *Localizer) FormatTime(t time.Time) string {
    return t.Format(l.timeFormat)
}

func (l *Localizer) FormatDate(t time.Time) string {
    return t.Format(l.dateFormat)
}

func (l *Localizer) FormatNumber(n float64) string {
    switch l.locale {
    case "de-DE", "fr-FR":
        // European format: 1.234,56
        str := fmt.Sprintf("%.2f", n)
        parts := strings.Split(str, ".")
        if len(parts) == 2 {
            return parts[0] + "," + parts[1]
        }
        return str
    case "en-IN":
        // Indian format: 1,23,456.78
        str := fmt.Sprintf("%.2f", n)
        parts := strings.Split(str, ".")
        integer := parts[0]
        decimal := ""
        if len(parts) == 2 {
            decimal = "." + parts[1]
        }
        
        // Add commas in Indian style
        if len(integer) > 3 {
            result := integer[len(integer)-3:]
            remaining := integer[:len(integer)-3]
            
            for len(remaining) > 2 {
                result = remaining[len(remaining)-2:] + "," + result
                remaining = remaining[:len(remaining)-2]
            }
            
            if len(remaining) > 0 {
                result = remaining + "," + result
            }
            
            return result + decimal
        }
        return str
    default:
        // US/UK format: 1,234.56
        return fmt.Sprintf("%.2f", n)
    }
}

func initializeMessageCatalog() MessageCatalog {
    return MessageCatalog{
        "en-US": {
            "welcome":        "Welcome to the application!",
            "processing":     "Processing %d items...",
            "completed":      "Processing completed successfully.",
            "error":          "An error occurred: %s",
            "current_time":   "Current time: %s",
            "file_not_found": "File not found: %s",
            "invalid_input":  "Invalid input provided",
            "goodbye":        "Thank you for using our application!",
        },
        "en-GB": {
            "welcome":        "Welcome to the programme!",
            "processing":     "Processing %d items...",
            "completed":      "Processing completed successfully.",
            "error":          "An error occurred: %s",
            "current_time":   "Current time: %s",
            "file_not_found": "File not found: %s",
            "invalid_input":  "Invalid input provided",
            "goodbye":        "Cheers! Thanks for using our programme!",
        },
        "de-DE": {
            "welcome":        "Willkommen zur Anwendung!",
            "processing":     "Verarbeite %d Elemente...",
            "completed":      "Verarbeitung erfolgreich abgeschlossen.",
            "error":          "Ein Fehler ist aufgetreten: %s",
            "current_time":   "Aktuelle Zeit: %s",
            "file_not_found": "Datei nicht gefunden: %s",
            "invalid_input":  "Ungültige Eingabe",
            "goodbye":        "Vielen Dank für die Nutzung unserer Anwendung!",
        },
        "fr-FR": {
            "welcome":        "Bienvenue dans l'application !",
            "processing":     "Traitement de %d éléments...",
            "completed":      "Traitement terminé avec succès.",
            "error":          "Une erreur s'est produite : %s",
            "current_time":   "Heure actuelle : %s",
            "file_not_found": "Fichier non trouvé : %s",
            "invalid_input":  "Saisie invalide",
            "goodbye":        "Merci d'utiliser notre application !",
        },
        "ja-JP": {
            "welcome":        "アプリケーションへようこそ！",
            "processing":     "%d個のアイテムを処理中...",
            "completed":      "処理が正常に完了しました。",
            "error":          "エラーが発生しました：%s",
            "current_time":   "現在時刻：%s",
            "file_not_found": "ファイルが見つかりません：%s",
            "invalid_input":  "無効な入力です",
            "goodbye":        "ご利用ありがとうございました！",
        },
    }
}

func main() {
    var (
        locale     = flag.String("locale", "en-US", "locale (en-US, en-GB, de-DE, fr-FR, ja-JP)")
        count      = flag.Int("count", 5, "number of items to process")
        showTime   = flag.Bool("show-time", true, "show current time")
        showNumber = flag.Bool("show-number", true, "show formatted number")
        testError  = flag.Bool("test-error", false, "simulate an error")
    )
    
    flag.Parse()
    
    // Initialize localization
    catalog := initializeMessageCatalog()
    localizer := NewLocalizer(catalog, *locale, "en-US")
    
    // Welcome message
    fmt.Println(localizer.GetMessage("welcome"))
    
    // Show current time if requested
    if *showTime {
        now := time.Now()
        timeStr := localizer.FormatTime(now)
        dateStr := localizer.FormatDate(now)
        fmt.Printf("%s - %s %s\n", 
            localizer.GetMessage("current_time", timeStr), 
            dateStr, timeStr)
    }
    
    // Show formatted number
    if *showNumber {
        number := 123456.78
        formatted := localizer.FormatNumber(number)
        fmt.Printf("Sample number: %s\n", formatted)
    }
    
    // Simulate error if requested
    if *testError {
        fmt.Println(localizer.GetMessage("error", "simulated error"))
        return
    }
    
    // Processing simulation
    fmt.Println(localizer.GetMessage("processing", *count))
    time.Sleep(1 * time.Second) // Simulate work
    
    fmt.Println(localizer.GetMessage("completed"))
    fmt.Println(localizer.GetMessage("goodbye"))
}
```

Internationalization flags enable CLI applications to support multiple  
languages, locales, and cultural formatting preferences. This pattern  
includes message catalogs, locale-specific number and date formatting,  
and fallback mechanisms for missing translations.  

## Security and authentication flags

CLI applications handling sensitive operations require robust security  
configuration through command-line flags for authentication and authorization.  

```go
package main

import (
    "crypto/rand"
    "crypto/sha256"
    "encoding/base64"
    "encoding/hex"
    "flag"
    "fmt"
    "os"
    "strings"
    "time"
    
    "golang.org/x/crypto/bcrypt"
)

type SecurityConfig struct {
    AuthMethod      string
    TokenFile       string
    PasswordHash    string
    APIKey          string
    RequireSSL      bool
    SessionTimeout  time.Duration
    MaxLoginAttempts int
    LockoutDuration time.Duration
}

type AuthProvider struct {
    config       *SecurityConfig
    failedAttempts map[string]int
    lockedUntil   map[string]time.Time
}

func NewAuthProvider(config *SecurityConfig) *AuthProvider {
    return &AuthProvider{
        config:         config,
        failedAttempts: make(map[string]int),
        lockedUntil:    make(map[string]time.Time),
    }
}

func (ap *AuthProvider) Authenticate(identifier, credential string) error {
    // Check if user is locked out
    if lockTime, exists := ap.lockedUntil[identifier]; exists {
        if time.Now().Before(lockTime) {
            remaining := lockTime.Sub(time.Now())
            return fmt.Errorf("account locked, try again in %v", remaining.Round(time.Second))
        }
        // Lockout period expired
        delete(ap.lockedUntil, identifier)
        delete(ap.failedAttempts, identifier)
    }
    
    var authSuccess bool
    var err error
    
    switch ap.config.AuthMethod {
    case "password":
        authSuccess, err = ap.authenticatePassword(credential)
    case "token":
        authSuccess, err = ap.authenticateToken(credential)
    case "apikey":
        authSuccess, err = ap.authenticateAPIKey(credential)
    default:
        return fmt.Errorf("unsupported authentication method: %s", ap.config.AuthMethod)
    }
    
    if err != nil {
        return err
    }
    
    if !authSuccess {
        ap.handleFailedAttempt(identifier)
        return fmt.Errorf("authentication failed")
    }
    
    // Reset failed attempts on successful authentication
    delete(ap.failedAttempts, identifier)
    return nil
}

func (ap *AuthProvider) authenticatePassword(password string) (bool, error) {
    if ap.config.PasswordHash == "" {
        return false, fmt.Errorf("no password hash configured")
    }
    
    hashBytes, err := hex.DecodeString(ap.config.PasswordHash)
    if err != nil {
        return false, fmt.Errorf("invalid password hash format")
    }
    
    err = bcrypt.CompareHashAndPassword(hashBytes, []byte(password))
    return err == nil, nil
}

func (ap *AuthProvider) authenticateToken(token string) (bool, error) {
    if ap.config.TokenFile == "" {
        return false, fmt.Errorf("no token file configured")
    }
    
    tokenBytes, err := os.ReadFile(ap.config.TokenFile)
    if err != nil {
        return false, fmt.Errorf("failed to read token file: %v", err)
    }
    
    validToken := strings.TrimSpace(string(tokenBytes))
    return token == validToken, nil
}

func (ap *AuthProvider) authenticateAPIKey(apiKey string) (bool, error) {
    if ap.config.APIKey == "" {
        return false, fmt.Errorf("no API key configured")
    }
    
    return apiKey == ap.config.APIKey, nil
}

func (ap *AuthProvider) handleFailedAttempt(identifier string) {
    ap.failedAttempts[identifier]++
    
    if ap.failedAttempts[identifier] >= ap.config.MaxLoginAttempts {
        ap.lockedUntil[identifier] = time.Now().Add(ap.config.LockoutDuration)
        fmt.Printf("Too many failed attempts. Account locked for %v\n", ap.config.LockoutDuration)
    }
}

func generateSecureToken(length int) (string, error) {
    bytes := make([]byte, length)
    if _, err := rand.Read(bytes); err != nil {
        return "", err
    }
    return base64.URLEncoding.EncodeToString(bytes), nil
}

func hashPassword(password string) (string, error) {
    hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
    if err != nil {
        return "", err
    }
    return hex.EncodeToString(hash), nil
}

func generateAPIKey() string {
    hash := sha256.New()
    hash.Write([]byte(fmt.Sprintf("%d", time.Now().UnixNano())))
    return hex.EncodeToString(hash.Sum(nil))[:32]
}

func main() {
    var (
        authMethod      = flag.String("auth-method", "password", "authentication method: password, token, apikey")
        username        = flag.String("username", "", "username for authentication")
        password        = flag.String("password", "", "password for authentication")
        passwordHash    = flag.String("password-hash", "", "bcrypt password hash (hex encoded)")
        token           = flag.String("token", "", "authentication token")
        tokenFile       = flag.String("token-file", "", "file containing authentication token")
        apiKey          = flag.String("api-key", "", "API key for authentication")
        requireSSL      = flag.Bool("require-ssl", true, "require SSL/TLS connections")
        sessionTimeout  = flag.Duration("session-timeout", 30*time.Minute, "session timeout duration")
        maxAttempts     = flag.Int("max-login-attempts", 3, "maximum login attempts before lockout")
        lockoutDuration = flag.Duration("lockout-duration", 15*time.Minute, "account lockout duration")
        
        // Utility flags
        genToken     = flag.Bool("generate-token", false, "generate a secure token")
        genAPIKey    = flag.Bool("generate-api-key", false, "generate an API key")
        hashPassword = flag.String("hash-password", "", "generate password hash for given password")
        
        // Operation flags
        operation = flag.String("operation", "login", "operation to perform: login, status, logout")
        verbose   = flag.Bool("verbose", false, "verbose security logging")
    )
    
    flag.Parse()
    
    // Handle utility operations
    if *genToken {
        token, err := generateSecureToken(32)
        if err != nil {
            fmt.Printf("Error generating token: %v\n", err)
            os.Exit(1)
        }
        fmt.Printf("Generated token: %s\n", token)
        return
    }
    
    if *genAPIKey {
        apiKeyValue := generateAPIKey()
        fmt.Printf("Generated API key: %s\n", apiKeyValue)
        return
    }
    
    if *hashPassword != "" {
        hash, err := hashPassword(*hashPassword)
        if err != nil {
            fmt.Printf("Error hashing password: %v\n", err)
            os.Exit(1)
        }
        fmt.Printf("Password hash: %s\n", hash)
        return
    }
    
    // Security configuration
    config := &SecurityConfig{
        AuthMethod:       *authMethod,
        TokenFile:        *tokenFile,
        PasswordHash:     *passwordHash,
        APIKey:           *apiKey,
        RequireSSL:       *requireSSL,
        SessionTimeout:   *sessionTimeout,
        MaxLoginAttempts: *maxAttempts,
        LockoutDuration:  *lockoutDuration,
    }
    
    if *verbose {
        fmt.Printf("Security configuration:\n")
        fmt.Printf("  Auth method: %s\n", config.AuthMethod)
        fmt.Printf("  Require SSL: %t\n", config.RequireSSL)
        fmt.Printf("  Session timeout: %v\n", config.SessionTimeout)
        fmt.Printf("  Max login attempts: %d\n", config.MaxLoginAttempts)
        fmt.Printf("  Lockout duration: %v\n", config.LockoutDuration)
        
        if config.TokenFile != "" {
            fmt.Printf("  Token file: %s\n", config.TokenFile)
        }
    }
    
    authProvider := NewAuthProvider(config)
    
    switch *operation {
    case "login":
        if *username == "" {
            fmt.Println("Error: username is required for login")
            os.Exit(1)
        }
        
        var credential string
        switch *authMethod {
        case "password":
            credential = *password
        case "token":
            credential = *token
        case "apikey":
            credential = *apiKey
        }
        
        if credential == "" {
            fmt.Printf("Error: %s is required for authentication\n", *authMethod)
            os.Exit(1)
        }
        
        if *verbose {
            fmt.Printf("Attempting authentication for user: %s\n", *username)
        }
        
        if err := authProvider.Authenticate(*username, credential); err != nil {
            fmt.Printf("Authentication failed: %v\n", err)
            os.Exit(1)
        }
        
        fmt.Printf("Authentication successful for user: %s\n", *username)
        fmt.Printf("Session valid until: %s\n", time.Now().Add(config.SessionTimeout).Format("2006-01-02 15:04:05"))
        
    case "status":
        fmt.Println("Security Status:")
        fmt.Printf("  SSL/TLS required: %t\n", config.RequireSSL)
        fmt.Printf("  Session timeout: %v\n", config.SessionTimeout)
        fmt.Printf("  Authentication method: %s\n", config.AuthMethod)
        
    case "logout":
        fmt.Println("Logout completed successfully")
        
    default:
        fmt.Printf("Unknown operation: %s\n", *operation)
        os.Exit(1)
    }
}
```

Security flags provide comprehensive authentication and authorization  
configuration including multiple authentication methods, session management,  
rate limiting, and security utilities. This pattern is essential for CLI  
applications handling sensitive data or privileged operations.  

## Backup and restore operations

CLI applications often need backup and restore capabilities with extensive  
configuration options for data protection and recovery scenarios.  

```go
package main

import (
    "archive/tar"
    "compress/gzip"
    "crypto/md5"
    "flag"
    "fmt"
    "io"
    "os"
    "path/filepath"
    "strings"
    "time"
)

type BackupConfig struct {
    SourcePaths     []string
    DestinationPath string
    Compression     bool
    Encryption      bool
    Incremental     bool
    ExcludePatterns []string
    MaxBackups      int
    Verify          bool
    Verbose         bool
}

type BackupManager struct {
    config *BackupConfig
}

func NewBackupManager(config *BackupConfig) *BackupManager {
    return &BackupManager{config: config}
}

func (bm *BackupManager) CreateBackup() error {
    timestamp := time.Now().Format("20060102-150405")
    backupName := fmt.Sprintf("backup-%s", timestamp)
    
    if bm.config.Compression {
        backupName += ".tar.gz"
    } else {
        backupName += ".tar"
    }
    
    backupPath := filepath.Join(bm.config.DestinationPath, backupName)
    
    if bm.config.Verbose {
        fmt.Printf("Creating backup: %s\n", backupPath)
    }
    
    // Ensure destination directory exists
    if err := os.MkdirAll(bm.config.DestinationPath, 0755); err != nil {
        return fmt.Errorf("failed to create destination directory: %v", err)
    }
    
    // Create backup file
    file, err := os.Create(backupPath)
    if err != nil {
        return fmt.Errorf("failed to create backup file: %v", err)
    }
    defer file.Close()
    
    var writer io.Writer = file
    
    // Add compression if enabled
    if bm.config.Compression {
        gzWriter := gzip.NewWriter(file)
        defer gzWriter.Close()
        writer = gzWriter
    }
    
    // Create tar writer
    tarWriter := tar.NewWriter(writer)
    defer tarWriter.Close()
    
    // Add files to backup
    for _, sourcePath := range bm.config.SourcePaths {
        if err := bm.addToBackup(tarWriter, sourcePath, ""); err != nil {
            return fmt.Errorf("failed to backup %s: %v", sourcePath, err)
        }
    }
    
    if bm.config.Verbose {
        fmt.Printf("Backup created successfully: %s\n", backupPath)
    }
    
    // Verify backup if requested
    if bm.config.Verify {
        if err := bm.verifyBackup(backupPath); err != nil {
            return fmt.Errorf("backup verification failed: %v", err)
        }
        fmt.Println("Backup verification completed successfully")
    }
    
    // Clean up old backups if max backups is set
    if bm.config.MaxBackups > 0 {
        if err := bm.cleanupOldBackups(); err != nil {
            fmt.Printf("Warning: failed to cleanup old backups: %v\n", err)
        }
    }
    
    return nil
}

func (bm *BackupManager) addToBackup(tarWriter *tar.Writer, sourcePath, basePath string) error {
    return filepath.Walk(sourcePath, func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        // Check exclusion patterns
        if bm.shouldExclude(path) {
            if info.IsDir() {
                return filepath.SkipDir
            }
            return nil
        }
        
        // Create archive path
        var archivePath string
        if basePath != "" {
            relPath, err := filepath.Rel(basePath, path)
            if err != nil {
                return err
            }
            archivePath = relPath
        } else {
            archivePath = path
        }
        
        // Create tar header
        header, err := tar.FileInfoHeader(info, "")
        if err != nil {
            return err
        }
        header.Name = archivePath
        
        if bm.config.Verbose {
            fmt.Printf("Adding to backup: %s\n", archivePath)
        }
        
        // Write header
        if err := tarWriter.WriteHeader(header); err != nil {
            return err
        }
        
        // Write file content for regular files
        if info.Mode().IsRegular() {
            file, err := os.Open(path)
            if err != nil {
                return err
            }
            defer file.Close()
            
            if _, err := io.Copy(tarWriter, file); err != nil {
                return err
            }
        }
        
        return nil
    })
}

func (bm *BackupManager) shouldExclude(path string) bool {
    for _, pattern := range bm.config.ExcludePatterns {
        if matched, _ := filepath.Match(pattern, filepath.Base(path)); matched {
            return true
        }
        if strings.Contains(path, pattern) {
            return true
        }
    }
    return false
}

func (bm *BackupManager) verifyBackup(backupPath string) error {
    if bm.config.Verbose {
        fmt.Printf("Verifying backup: %s\n", backupPath)
    }
    
    file, err := os.Open(backupPath)
    if err != nil {
        return err
    }
    defer file.Close()
    
    var reader io.Reader = file
    
    // Handle compression
    if bm.config.Compression {
        gzReader, err := gzip.NewReader(file)
        if err != nil {
            return err
        }
        defer gzReader.Close()
        reader = gzReader
    }
    
    tarReader := tar.NewReader(reader)
    
    fileCount := 0
    for {
        header, err := tarReader.Next()
        if err == io.EOF {
            break
        }
        if err != nil {
            return err
        }
        
        fileCount++
        
        // Calculate checksum for regular files
        if header.Typeflag == tar.TypeReg {
            hasher := md5.New()
            if _, err := io.Copy(hasher, tarReader); err != nil {
                return err
            }
            checksum := fmt.Sprintf("%x", hasher.Sum(nil))
            
            if bm.config.Verbose {
                fmt.Printf("Verified: %s (checksum: %s)\n", header.Name, checksum[:8])
            }
        }
    }
    
    fmt.Printf("Verified %d files in backup\n", fileCount)
    return nil
}

func (bm *BackupManager) cleanupOldBackups() error {
    entries, err := os.ReadDir(bm.config.DestinationPath)
    if err != nil {
        return err
    }
    
    var backupFiles []os.DirEntry
    for _, entry := range entries {
        if strings.HasPrefix(entry.Name(), "backup-") {
            backupFiles = append(backupFiles, entry)
        }
    }
    
    if len(backupFiles) <= bm.config.MaxBackups {
        return nil
    }
    
    // Remove oldest backups
    toRemove := len(backupFiles) - bm.config.MaxBackups
    for i := 0; i < toRemove; i++ {
        backupPath := filepath.Join(bm.config.DestinationPath, backupFiles[i].Name())
        if err := os.Remove(backupPath); err != nil {
            return err
        }
        if bm.config.Verbose {
            fmt.Printf("Removed old backup: %s\n", backupFiles[i].Name())
        }
    }
    
    return nil
}

func main() {
    var (
        sources        = flag.String("sources", ".", "comma-separated list of source paths to backup")
        destination    = flag.String("destination", "./backups", "destination directory for backups")
        compress       = flag.Bool("compress", true, "enable gzip compression")
        encrypt        = flag.Bool("encrypt", false, "enable encryption (not implemented)")
        incremental    = flag.Bool("incremental", false, "create incremental backup (not implemented)")
        excludePattern = flag.String("exclude", "", "comma-separated exclude patterns")
        maxBackups     = flag.Int("max-backups", 10, "maximum number of backups to keep (0 = unlimited)")
        verify         = flag.Bool("verify", true, "verify backup integrity after creation")
        verbose        = flag.Bool("verbose", false, "verbose output")
        
        // Operations
        operation = flag.String("operation", "backup", "operation: backup, restore, list")
        restoreFrom = flag.String("restore-from", "", "backup file to restore from")
        restoreTo   = flag.String("restore-to", "./restore", "directory to restore to")
    )
    
    flag.Parse()
    
    // Parse source paths
    sourcePaths := strings.Split(*sources, ",")
    for i, path := range sourcePaths {
        sourcePaths[i] = strings.TrimSpace(path)
    }
    
    // Parse exclude patterns
    var excludePatterns []string
    if *excludePattern != "" {
        excludePatterns = strings.Split(*excludePattern, ",")
        for i, pattern := range excludePatterns {
            excludePatterns[i] = strings.TrimSpace(pattern)
        }
    }
    
    config := &BackupConfig{
        SourcePaths:     sourcePaths,
        DestinationPath: *destination,
        Compression:     *compress,
        Encryption:      *encrypt,
        Incremental:     *incremental,
        ExcludePatterns: excludePatterns,
        MaxBackups:      *maxBackups,
        Verify:          *verify,
        Verbose:         *verbose,
    }
    
    manager := NewBackupManager(config)
    
    switch *operation {
    case "backup":
        if *verbose {
            fmt.Printf("Starting backup operation...\n")
            fmt.Printf("Sources: %v\n", sourcePaths)
            fmt.Printf("Destination: %s\n", *destination)
            fmt.Printf("Compression: %t\n", *compress)
            if len(excludePatterns) > 0 {
                fmt.Printf("Exclude patterns: %v\n", excludePatterns)
            }
        }
        
        if err := manager.CreateBackup(); err != nil {
            fmt.Printf("Backup failed: %v\n", err)
            os.Exit(1)
        }
        
    case "list":
        entries, err := os.ReadDir(*destination)
        if err != nil {
            fmt.Printf("Error listing backups: %v\n", err)
            os.Exit(1)
        }
        
        fmt.Println("Available backups:")
        for _, entry := range entries {
            if strings.HasPrefix(entry.Name(), "backup-") {
                info, _ := entry.Info()
                fmt.Printf("  %s (%s)\n", entry.Name(), 
                    info.ModTime().Format("2006-01-02 15:04:05"))
            }
        }
        
    case "restore":
        if *restoreFrom == "" {
            fmt.Println("Error: -restore-from is required for restore operation")
            os.Exit(1)
        }
        
        fmt.Printf("Restore operation not implemented yet\n")
        fmt.Printf("Would restore from: %s\n", *restoreFrom)
        fmt.Printf("Would restore to: %s\n", *restoreTo)
        
    default:
        fmt.Printf("Unknown operation: %s\n", *operation)
        os.Exit(1)
    }
}
```

Backup and restore flags provide comprehensive data protection capabilities  
including compression options, exclusion patterns, verification, and retention  
policies. This pattern is essential for CLI tools that manage critical data  
and need reliable backup and recovery functionality.  

## Deployment and containerization flags

CLI applications often need deployment-specific configuration for  
containerization, orchestration, and cloud deployment scenarios.  

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "os"
    "path/filepath"
    "strconv"
    "strings"
    "time"
)

type DeploymentConfig struct {
    Environment       string
    ContainerRegistry string
    ImageName         string
    ImageTag          string
    Namespace         string
    Replicas          int
    Resources         ResourceConfig
    HealthCheck       HealthCheckConfig
    Secrets           []string
    ConfigMaps        []string
    Volumes           []VolumeConfig
    NetworkPolicies   []string
    Labels            map[string]string
    Annotations       map[string]string
}

type ResourceConfig struct {
    CPURequest    string
    CPULimit      string
    MemoryRequest string
    MemoryLimit   string
}

type HealthCheckConfig struct {
    Enabled         bool
    Path            string
    Port            int
    InitialDelay    time.Duration
    Period          time.Duration
    Timeout         time.Duration
    FailureThreshold int
}

type VolumeConfig struct {
    Name      string
    Type      string
    MountPath string
    Size      string
}

type DeploymentManager struct {
    config *DeploymentConfig
}

func NewDeploymentManager(config *DeploymentConfig) *DeploymentManager {
    return &DeploymentManager{config: config}
}

func (dm *DeploymentManager) GenerateKubernetesManifest() (string, error) {
    manifest := fmt.Sprintf(`apiVersion: apps/v1
kind: Deployment
metadata:
  name: %s
  namespace: %s
  labels:
%s
  annotations:
%s
spec:
  replicas: %d
  selector:
    matchLabels:
      app: %s
  template:
    metadata:
      labels:
        app: %s
%s
    spec:
      containers:
      - name: %s
        image: %s/%s:%s
        resources:
          requests:
            cpu: %s
            memory: %s
          limits:
            cpu: %s
            memory: %s
        ports:
        - containerPort: %d
%s
%s
%s
`,
        dm.config.ImageName,
        dm.config.Namespace,
        dm.formatLabels(dm.config.Labels, 4),
        dm.formatAnnotations(dm.config.Annotations, 4),
        dm.config.Replicas,
        dm.config.ImageName,
        dm.config.ImageName,
        dm.formatLabels(dm.config.Labels, 8),
        dm.config.ImageName,
        dm.config.ContainerRegistry,
        dm.config.ImageName,
        dm.config.ImageTag,
        dm.config.Resources.CPURequest,
        dm.config.Resources.MemoryRequest,
        dm.config.Resources.CPULimit,
        dm.config.Resources.MemoryLimit,
        dm.config.HealthCheck.Port,
        dm.generateHealthCheckProbes(),
        dm.generateVolumeMounts(),
        dm.generateVolumes())
    
    return manifest, nil
}

func (dm *DeploymentManager) formatLabels(labels map[string]string, indent int) string {
    if len(labels) == 0 {
        return ""
    }
    
    var result strings.Builder
    spaces := strings.Repeat(" ", indent)
    
    for key, value := range labels {
        result.WriteString(fmt.Sprintf("%s%s: %s\n", spaces, key, value))
    }
    
    return strings.TrimSuffix(result.String(), "\n")
}

func (dm *DeploymentManager) formatAnnotations(annotations map[string]string, indent int) string {
    if len(annotations) == 0 {
        return ""
    }
    
    var result strings.Builder
    spaces := strings.Repeat(" ", indent)
    
    for key, value := range annotations {
        result.WriteString(fmt.Sprintf("%s%s: \"%s\"\n", spaces, key, value))
    }
    
    return strings.TrimSuffix(result.String(), "\n")
}

func (dm *DeploymentManager) generateHealthCheckProbes() string {
    if !dm.config.HealthCheck.Enabled {
        return ""
    }
    
    return fmt.Sprintf(`        livenessProbe:
          httpGet:
            path: %s
            port: %d
          initialDelaySeconds: %d
          periodSeconds: %d
          timeoutSeconds: %d
          failureThreshold: %d
        readinessProbe:
          httpGet:
            path: %s
            port: %d
          initialDelaySeconds: %d
          periodSeconds: %d
          timeoutSeconds: %d
          failureThreshold: %d`,
        dm.config.HealthCheck.Path,
        dm.config.HealthCheck.Port,
        int(dm.config.HealthCheck.InitialDelay.Seconds()),
        int(dm.config.HealthCheck.Period.Seconds()),
        int(dm.config.HealthCheck.Timeout.Seconds()),
        dm.config.HealthCheck.FailureThreshold,
        dm.config.HealthCheck.Path,
        dm.config.HealthCheck.Port,
        int(dm.config.HealthCheck.InitialDelay.Seconds()),
        int(dm.config.HealthCheck.Period.Seconds()),
        int(dm.config.HealthCheck.Timeout.Seconds()),
        dm.config.HealthCheck.FailureThreshold)
}

func (dm *DeploymentManager) generateVolumeMounts() string {
    if len(dm.config.Volumes) == 0 {
        return ""
    }
    
    var result strings.Builder
    result.WriteString("        volumeMounts:\n")
    
    for _, volume := range dm.config.Volumes {
        result.WriteString(fmt.Sprintf("        - name: %s\n", volume.Name))
        result.WriteString(fmt.Sprintf("          mountPath: %s\n", volume.MountPath))
    }
    
    return result.String()
}

func (dm *DeploymentManager) generateVolumes() string {
    if len(dm.config.Volumes) == 0 {
        return ""
    }
    
    var result strings.Builder
    result.WriteString("      volumes:\n")
    
    for _, volume := range dm.config.Volumes {
        result.WriteString(fmt.Sprintf("      - name: %s\n", volume.Name))
        switch volume.Type {
        case "configMap":
            result.WriteString(fmt.Sprintf("        configMap:\n"))
            result.WriteString(fmt.Sprintf("          name: %s\n", volume.Name))
        case "secret":
            result.WriteString(fmt.Sprintf("        secret:\n"))
            result.WriteString(fmt.Sprintf("          secretName: %s\n", volume.Name))
        case "persistentVolumeClaim":
            result.WriteString(fmt.Sprintf("        persistentVolumeClaim:\n"))
            result.WriteString(fmt.Sprintf("          claimName: %s\n", volume.Name))
        }
    }
    
    return result.String()
}

func (dm *DeploymentManager) GenerateDockerfile(baseImage, workdir string, commands []string) string {
    dockerfile := fmt.Sprintf("FROM %s\n\n", baseImage)
    
    if workdir != "" {
        dockerfile += fmt.Sprintf("WORKDIR %s\n\n", workdir)
    }
    
    // Add labels
    if len(dm.config.Labels) > 0 {
        dockerfile += "# Labels\n"
        for key, value := range dm.config.Labels {
            dockerfile += fmt.Sprintf("LABEL %s=\"%s\"\n", key, value)
        }
        dockerfile += "\n"
    }
    
    // Add commands
    for _, cmd := range commands {
        dockerfile += fmt.Sprintf("RUN %s\n", cmd)
    }
    
    dockerfile += "\n"
    
    // Health check
    if dm.config.HealthCheck.Enabled {
        dockerfile += fmt.Sprintf("HEALTHCHECK --interval=%ds --timeout=%ds --start-period=%ds --retries=%d \\\n",
            int(dm.config.HealthCheck.Period.Seconds()),
            int(dm.config.HealthCheck.Timeout.Seconds()),
            int(dm.config.HealthCheck.InitialDelay.Seconds()),
            dm.config.HealthCheck.FailureThreshold)
        dockerfile += fmt.Sprintf("  CMD curl -f http://localhost:%d%s || exit 1\n\n",
            dm.config.HealthCheck.Port,
            dm.config.HealthCheck.Path)
    }
    
    dockerfile += fmt.Sprintf("EXPOSE %d\n\n", dm.config.HealthCheck.Port)
    dockerfile += "CMD [\"./app\"]\n"
    
    return dockerfile
}

func parseKeyValuePairs(input string) map[string]string {
    result := make(map[string]string)
    if input == "" {
        return result
    }
    
    pairs := strings.Split(input, ",")
    for _, pair := range pairs {
        parts := strings.SplitN(strings.TrimSpace(pair), "=", 2)
        if len(parts) == 2 {
            result[parts[0]] = parts[1]
        }
    }
    
    return result
}

func parseVolumes(input string) []VolumeConfig {
    if input == "" {
        return nil
    }
    
    var volumes []VolumeConfig
    volumeSpecs := strings.Split(input, ",")
    
    for _, spec := range volumeSpecs {
        parts := strings.Split(strings.TrimSpace(spec), ":")
        if len(parts) >= 3 {
            volume := VolumeConfig{
                Name:      parts[0],
                Type:      parts[1],
                MountPath: parts[2],
            }
            if len(parts) > 3 {
                volume.Size = parts[3]
            }
            volumes = append(volumes, volume)
        }
    }
    
    return volumes
}

func main() {
    var (
        environment    = flag.String("environment", "production", "deployment environment")
        registry       = flag.String("registry", "docker.io", "container registry")
        imageName      = flag.String("image-name", "myapp", "container image name")
        imageTag       = flag.String("image-tag", "latest", "container image tag")
        namespace      = flag.String("namespace", "default", "Kubernetes namespace")
        replicas       = flag.Int("replicas", 3, "number of replicas")
        
        // Resource configuration
        cpuRequest    = flag.String("cpu-request", "100m", "CPU request")
        cpuLimit      = flag.String("cpu-limit", "500m", "CPU limit")
        memoryRequest = flag.String("memory-request", "128Mi", "memory request")
        memoryLimit   = flag.String("memory-limit", "512Mi", "memory limit")
        
        // Health check configuration
        enableHealthCheck = flag.Bool("health-check", true, "enable health checks")
        healthPath       = flag.String("health-path", "/health", "health check path")
        healthPort       = flag.Int("health-port", 8080, "health check port")
        healthDelay      = flag.Duration("health-delay", 30*time.Second, "initial delay for health checks")
        healthPeriod     = flag.Duration("health-period", 10*time.Second, "health check period")
        healthTimeout    = flag.Duration("health-timeout", 5*time.Second, "health check timeout")
        healthThreshold  = flag.Int("health-threshold", 3, "health check failure threshold")
        
        // Additional configuration
        labels      = flag.String("labels", "", "labels as key=value pairs (comma-separated)")
        annotations = flag.String("annotations", "", "annotations as key=value pairs (comma-separated)")
        secrets     = flag.String("secrets", "", "comma-separated list of secrets")
        configMaps  = flag.String("config-maps", "", "comma-separated list of config maps")
        volumes     = flag.String("volumes", "", "volumes as name:type:mountPath:size (comma-separated)")
        
        // Operations
        operation    = flag.String("operation", "manifest", "operation: manifest, dockerfile, deploy")
        outputFile   = flag.String("output", "", "output file (default: stdout)")
        baseImage    = flag.String("base-image", "alpine:latest", "base Docker image")
        workdir      = flag.String("workdir", "/app", "Docker working directory")
        commands     = flag.String("commands", "", "comma-separated Docker RUN commands")
        verbose      = flag.Bool("verbose", false, "verbose output")
    )
    
    flag.Parse()
    
    // Parse labels and annotations
    labelMap := parseKeyValuePairs(*labels)
    annotationMap := parseKeyValuePairs(*annotations)
    
    // Set default labels
    if labelMap["app"] == "" {
        labelMap["app"] = *imageName
    }
    if labelMap["version"] == "" {
        labelMap["version"] = *imageTag
    }
    if labelMap["environment"] == "" {
        labelMap["environment"] = *environment
    }
    
    // Parse secrets and config maps
    var secretList, configMapList []string
    if *secrets != "" {
        secretList = strings.Split(*secrets, ",")
    }
    if *configMaps != "" {
        configMapList = strings.Split(*configMaps, ",")
    }
    
    // Parse volumes
    volumeList := parseVolumes(*volumes)
    
    config := &DeploymentConfig{
        Environment:       *environment,
        ContainerRegistry: *registry,
        ImageName:         *imageName,
        ImageTag:          *imageTag,
        Namespace:         *namespace,
        Replicas:          *replicas,
        Resources: ResourceConfig{
            CPURequest:    *cpuRequest,
            CPULimit:      *cpuLimit,
            MemoryRequest: *memoryRequest,
            MemoryLimit:   *memoryLimit,
        },
        HealthCheck: HealthCheckConfig{
            Enabled:          *enableHealthCheck,
            Path:             *healthPath,
            Port:             *healthPort,
            InitialDelay:     *healthDelay,
            Period:           *healthPeriod,
            Timeout:          *healthTimeout,
            FailureThreshold: *healthThreshold,
        },
        Secrets:     secretList,
        ConfigMaps:  configMapList,
        Volumes:     volumeList,
        Labels:      labelMap,
        Annotations: annotationMap,
    }
    
    manager := NewDeploymentManager(config)
    
    if *verbose {
        fmt.Printf("Deployment configuration:\n")
        configJSON, _ := json.MarshalIndent(config, "", "  ")
        fmt.Printf("%s\n\n", configJSON)
    }
    
    var output string
    var err error
    
    switch *operation {
    case "manifest":
        output, err = manager.GenerateKubernetesManifest()
        if err != nil {
            fmt.Printf("Error generating manifest: %v\n", err)
            os.Exit(1)
        }
        
    case "dockerfile":
        var commandList []string
        if *commands != "" {
            commandList = strings.Split(*commands, ",")
        }
        output = manager.GenerateDockerfile(*baseImage, *workdir, commandList)
        
    case "deploy":
        fmt.Println("Deploy operation would execute:")
        fmt.Printf("  kubectl apply -f deployment.yaml\n")
        fmt.Printf("  kubectl set image deployment/%s %s=%s/%s:%s\n", 
            *imageName, *imageName, *registry, *imageName, *imageTag)
        return
        
    default:
        fmt.Printf("Unknown operation: %s\n", *operation)
        os.Exit(1)
    }
    
    // Write output
    if *outputFile != "" {
        if err := os.WriteFile(*outputFile, []byte(output), 0644); err != nil {
            fmt.Printf("Error writing output file: %v\n", err)
            os.Exit(1)
        }
        fmt.Printf("Output written to: %s\n", *outputFile)
    } else {
        fmt.Print(output)
    }
}
```

Deployment and containerization flags provide comprehensive configuration  
for modern cloud-native applications including Kubernetes manifests,  
Docker containers, resource management, health checks, and orchestration  
settings. This pattern is essential for CLI tools that manage application  
deployment pipelines and infrastructure as code.  

## API client with authentication

CLI applications frequently need to interact with REST APIs using various  
authentication methods and request configurations controlled by flags.  

```go
package main

import (
    "bytes"
    "encoding/json"
    "flag"
    "fmt"
    "io"
    "net/http"
    "net/url"
    "os"
    "strings"
    "time"
)

type APIClient struct {
    BaseURL    string
    HTTPClient *http.Client
    AuthType   string
    AuthToken  string
    APIKey     string
    Username   string
    Password   string
    Headers    map[string]string
    Verbose    bool
}

type APIResponse struct {
    StatusCode int
    Headers    http.Header
    Body       []byte
    Duration   time.Duration
}

func NewAPIClient(baseURL, authType string) *APIClient {
    return &APIClient{
        BaseURL:    baseURL,
        AuthType:   authType,
        Headers:    make(map[string]string),
        HTTPClient: &http.Client{Timeout: 30 * time.Second},
    }
}

func (client *APIClient) SetTimeout(timeout time.Duration) {
    client.HTTPClient.Timeout = timeout
}

func (client *APIClient) AddHeader(key, value string) {
    client.Headers[key] = value
}

func (client *APIClient) SetAuth(token, apiKey, username, password string) {
    client.AuthToken = token
    client.APIKey = apiKey
    client.Username = username
    client.Password = password
}

func (client *APIClient) buildRequest(method, endpoint string, body interface{}) (*http.Request, error) {
    fullURL := client.BaseURL + endpoint
    
    var reqBody io.Reader
    if body != nil {
        switch v := body.(type) {
        case string:
            reqBody = strings.NewReader(v)
        case []byte:
            reqBody = bytes.NewReader(v)
        default:
            jsonData, err := json.Marshal(body)
            if err != nil {
                return nil, fmt.Errorf("failed to marshal request body: %v", err)
            }
            reqBody = bytes.NewReader(jsonData)
            client.Headers["Content-Type"] = "application/json"
        }
    }
    
    req, err := http.NewRequest(method, fullURL, reqBody)
    if err != nil {
        return nil, err
    }
    
    // Add custom headers
    for key, value := range client.Headers {
        req.Header.Set(key, value)
    }
    
    // Add authentication
    switch client.AuthType {
    case "bearer":
        if client.AuthToken != "" {
            req.Header.Set("Authorization", "Bearer "+client.AuthToken)
        }
    case "apikey":
        if client.APIKey != "" {
            req.Header.Set("X-API-Key", client.APIKey)
        }
    case "basic":
        if client.Username != "" && client.Password != "" {
            req.SetBasicAuth(client.Username, client.Password)
        }
    }
    
    return req, nil
}

func (client *APIClient) makeRequest(method, endpoint string, body interface{}) (*APIResponse, error) {
    req, err := client.buildRequest(method, endpoint, body)
    if err != nil {
        return nil, err
    }
    
    if client.Verbose {
        fmt.Printf("→ %s %s\n", method, req.URL.String())
        for key, values := range req.Header {
            for _, value := range values {
                if key == "Authorization" {
                    fmt.Printf("  %s: %s\n", key, maskAuth(value))
                } else {
                    fmt.Printf("  %s: %s\n", key, value)
                }
            }
        }
    }
    
    startTime := time.Now()
    resp, err := client.HTTPClient.Do(req)
    duration := time.Since(startTime)
    
    if err != nil {
        return nil, fmt.Errorf("request failed: %v", err)
    }
    defer resp.Body.Close()
    
    respBody, err := io.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed to read response body: %v", err)
    }
    
    response := &APIResponse{
        StatusCode: resp.StatusCode,
        Headers:    resp.Header,
        Body:       respBody,
        Duration:   duration,
    }
    
    if client.Verbose {
        fmt.Printf("← %d %s (%v)\n", resp.StatusCode, http.StatusText(resp.StatusCode), duration)
        fmt.Printf("  Content-Length: %d\n", len(respBody))
    }
    
    return response, nil
}

func (client *APIClient) Get(endpoint string) (*APIResponse, error) {
    return client.makeRequest("GET", endpoint, nil)
}

func (client *APIClient) Post(endpoint string, body interface{}) (*APIResponse, error) {
    return client.makeRequest("POST", endpoint, body)
}

func (client *APIClient) Put(endpoint string, body interface{}) (*APIResponse, error) {
    return client.makeRequest("PUT", endpoint, body)
}

func (client *APIClient) Delete(endpoint string) (*APIResponse, error) {
    return client.makeRequest("DELETE", endpoint, nil)
}

func maskAuth(auth string) string {
    if len(auth) <= 10 {
        return "***"
    }
    return auth[:7] + "***"
}

func parseQueryParams(params string) string {
    if params == "" {
        return ""
    }
    
    values := url.Values{}
    pairs := strings.Split(params, ",")
    
    for _, pair := range pairs {
        parts := strings.SplitN(strings.TrimSpace(pair), "=", 2)
        if len(parts) == 2 {
            values.Add(parts[0], parts[1])
        }
    }
    
    if len(values) > 0 {
        return "?" + values.Encode()
    }
    return ""
}

func formatResponse(response *APIResponse, format string) string {
    switch format {
    case "json":
        var prettyJSON bytes.Buffer
        if err := json.Indent(&prettyJSON, response.Body, "", "  "); err == nil {
            return prettyJSON.String()
        }
        return string(response.Body)
        
    case "headers":
        var result strings.Builder
        result.WriteString(fmt.Sprintf("Status: %d %s\n", response.StatusCode, http.StatusText(response.StatusCode)))
        result.WriteString(fmt.Sprintf("Duration: %v\n", response.Duration))
        result.WriteString("Headers:\n")
        for key, values := range response.Headers {
            for _, value := range values {
                result.WriteString(fmt.Sprintf("  %s: %s\n", key, value))
            }
        }
        return result.String()
        
    case "raw":
        return string(response.Body)
        
    default:
        var result strings.Builder
        result.WriteString(fmt.Sprintf("Status: %d %s\n", response.StatusCode, http.StatusText(response.StatusCode)))
        result.WriteString(fmt.Sprintf("Duration: %v\n", response.Duration))
        result.WriteString(fmt.Sprintf("Content-Length: %d\n", len(response.Body)))
        result.WriteString("\nResponse:\n")
        result.WriteString(string(response.Body))
        return result.String()
    }
}

func main() {
    var (
        baseURL      = flag.String("url", "", "base API URL (required)")
        method       = flag.String("method", "GET", "HTTP method (GET, POST, PUT, DELETE)")
        endpoint     = flag.String("endpoint", "/", "API endpoint path")
        authType     = flag.String("auth-type", "none", "authentication type: none, bearer, apikey, basic")
        authToken    = flag.String("token", "", "bearer token for authentication")
        apiKey       = flag.String("api-key", "", "API key for authentication")
        username     = flag.String("username", "", "username for basic authentication")
        password     = flag.String("password", "", "password for basic authentication")
        headers      = flag.String("headers", "", "additional headers (key=value,key=value)")
        queryParams  = flag.String("params", "", "query parameters (key=value,key=value)")
        body         = flag.String("body", "", "request body (JSON string or @filename)")
        bodyFile     = flag.String("body-file", "", "file containing request body")
        timeout      = flag.Duration("timeout", 30*time.Second, "request timeout")
        format       = flag.String("format", "default", "response format: default, json, headers, raw")
        outputFile   = flag.String("output", "", "output file for response")
        verbose      = flag.Bool("verbose", false, "verbose request/response logging")
        insecure     = flag.Bool("insecure", false, "skip TLS certificate verification")
    )
    
    flag.Parse()
    
    if *baseURL == "" {
        fmt.Println("Error: -url is required")
        flag.Usage()
        os.Exit(1)
    }
    
    // Create API client
    client := NewAPIClient(*baseURL, *authType)
    client.SetTimeout(*timeout)
    client.Verbose = *verbose
    
    // Set authentication
    client.SetAuth(*authToken, *apiKey, *username, *password)
    
    // Add custom headers
    if *headers != "" {
        headerPairs := strings.Split(*headers, ",")
        for _, pair := range headerPairs {
            parts := strings.SplitN(strings.TrimSpace(pair), "=", 2)
            if len(parts) == 2 {
                client.AddHeader(parts[0], parts[1])
            }
        }
    }
    
    // Build endpoint with query parameters
    fullEndpoint := *endpoint + parseQueryParams(*queryParams)
    
    // Prepare request body
    var requestBody interface{}
    if *bodyFile != "" {
        bodyBytes, err := os.ReadFile(*bodyFile)
        if err != nil {
            fmt.Printf("Error reading body file: %v\n", err)
            os.Exit(1)
        }
        requestBody = bodyBytes
    } else if *body != "" {
        if strings.HasPrefix(*body, "@") {
            filename := strings.TrimPrefix(*body, "@")
            bodyBytes, err := os.ReadFile(filename)
            if err != nil {
                fmt.Printf("Error reading body file %s: %v\n", filename, err)
                os.Exit(1)
            }
            requestBody = bodyBytes
        } else {
            requestBody = *body
        }
    }
    
    // Make API request
    var response *APIResponse
    var err error
    
    switch strings.ToUpper(*method) {
    case "GET":
        response, err = client.Get(fullEndpoint)
    case "POST":
        response, err = client.Post(fullEndpoint, requestBody)
    case "PUT":
        response, err = client.Put(fullEndpoint, requestBody)
    case "DELETE":
        response, err = client.Delete(fullEndpoint)
    default:
        fmt.Printf("Unsupported HTTP method: %s\n", *method)
        os.Exit(1)
    }
    
    if err != nil {
        fmt.Printf("API request failed: %v\n", err)
        os.Exit(1)
    }
    
    // Format and output response
    output := formatResponse(response, *format)
    
    if *outputFile != "" {
        if err := os.WriteFile(*outputFile, []byte(output), 0644); err != nil {
            fmt.Printf("Error writing output file: %v\n", err)
            os.Exit(1)
        }
        fmt.Printf("Response saved to: %s\n", *outputFile)
    } else {
        fmt.Print(output)
    }
    
    // Exit with non-zero code for HTTP errors
    if response.StatusCode >= 400 {
        os.Exit(1)
    }
}
```

API client flags provide comprehensive configuration for HTTP requests  
including multiple authentication methods, custom headers, request bodies,  
timeout settings, and response formatting. This pattern is essential for  
CLI tools that integrate with REST APIs and web services.  

## Comprehensive CLI application template

This final example demonstrates a complete CLI application template that  
incorporates many of the patterns and techniques shown in previous examples.  

```go
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "log"
    "os"
    "path/filepath"
    "strings"
    "time"
)

// Application represents the main CLI application
type Application struct {
    Name        string
    Version     string
    Description string
    Config      *Config
    Logger      *Logger
    Commands    map[string]Command
}

// Config holds application configuration
type Config struct {
    LogLevel    string        `json:"log_level"`
    LogFormat   string        `json:"log_format"`
    Timeout     time.Duration `json:"timeout"`
    MaxRetries  int           `json:"max_retries"`
    OutputDir   string        `json:"output_dir"`
    TempDir     string        `json:"temp_dir"`
    Concurrent  bool          `json:"concurrent"`
    MaxWorkers  int           `json:"max_workers"`
    Verbose     bool          `json:"verbose"`
    DryRun      bool          `json:"dry_run"`
}

// Command interface for subcommands
type Command interface {
    Name() string
    Description() string
    Execute(args []string, config *Config) error
}

// Logger provides structured logging
type Logger struct {
    level  string
    format string
}

func (l *Logger) Info(msg string, args ...interface{}) {
    if l.shouldLog("info") {
        l.log("INFO", msg, args...)
    }
}

func (l *Logger) Error(msg string, args ...interface{}) {
    if l.shouldLog("error") {
        l.log("ERROR", msg, args...)
    }
}

func (l *Logger) Debug(msg string, args ...interface{}) {
    if l.shouldLog("debug") {
        l.log("DEBUG", msg, args...)
    }
}

func (l *Logger) shouldLog(level string) bool {
    levels := map[string]int{"debug": 0, "info": 1, "warn": 2, "error": 3}
    return levels[level] >= levels[l.level]
}

func (l *Logger) log(level, msg string, args ...interface{}) {
    timestamp := time.Now().Format("2006-01-02T15:04:05Z07:00")
    message := fmt.Sprintf(msg, args...)
    
    switch l.format {
    case "json":
        logEntry := map[string]interface{}{
            "timestamp": timestamp,
            "level":     level,
            "message":   message,
        }
        jsonData, _ := json.Marshal(logEntry)
        fmt.Println(string(jsonData))
    default:
        fmt.Printf("[%s] %s: %s\n", timestamp, level, message)
    }
}

// ProcessCommand implements file processing
type ProcessCommand struct{}

func (cmd *ProcessCommand) Name() string {
    return "process"
}

func (cmd *ProcessCommand) Description() string {
    return "Process files with various options"
}

func (cmd *ProcessCommand) Execute(args []string, config *Config) error {
    fmt.Printf("Processing files with config: %+v\n", config)
    fmt.Printf("Arguments: %v\n", args)
    
    if config.DryRun {
        fmt.Println("[DRY RUN] Would process files here")
        return nil
    }
    
    // Simulate processing
    fmt.Println("Processing completed successfully")
    return nil
}

// StatusCommand shows application status
type StatusCommand struct{}

func (cmd *StatusCommand) Name() string {
    return "status"
}

func (cmd *StatusCommand) Description() string {
    return "Show application status and configuration"
}

func (cmd *StatusCommand) Execute(args []string, config *Config) error {
    fmt.Printf("Application Status:\n")
    fmt.Printf("  Log Level: %s\n", config.LogLevel)
    fmt.Printf("  Output Directory: %s\n", config.OutputDir)
    fmt.Printf("  Max Workers: %d\n", config.MaxWorkers)
    fmt.Printf("  Concurrent: %t\n", config.Concurrent)
    fmt.Printf("  Dry Run: %t\n", config.DryRun)
    return nil
}

// NewApplication creates a new CLI application
func NewApplication() *Application {
    app := &Application{
        Name:        "myapp",
        Version:     "1.0.0",
        Description: "A comprehensive CLI application demonstrating Go flag patterns",
        Commands:    make(map[string]Command),
    }
    
    // Register commands
    app.Commands["process"] = &ProcessCommand{}
    app.Commands["status"] = &StatusCommand{}
    
    return app
}

func (app *Application) LoadConfig(configFile string) error {
    // Set defaults
    app.Config = &Config{
        LogLevel:   "info",
        LogFormat:  "text",
        Timeout:    30 * time.Second,
        MaxRetries: 3,
        OutputDir:  "./output",
        TempDir:    "/tmp",
        MaxWorkers: 4,
    }
    
    // Try to load from file
    if configFile != "" {
        if data, err := os.ReadFile(configFile); err == nil {
            if err := json.Unmarshal(data, app.Config); err != nil {
                return fmt.Errorf("invalid config file format: %v", err)
            }
        }
    }
    
    return nil
}

func (app *Application) InitializeLogger() {
    app.Logger = &Logger{
        level:  app.Config.LogLevel,
        format: app.Config.LogFormat,
    }
}

func (app *Application) ShowHelp() {
    fmt.Printf("%s v%s\n", app.Name, app.Version)
    fmt.Printf("%s\n\n", app.Description)
    fmt.Printf("Usage: %s [options] <command> [command-options]\n\n", os.Args[0])
    
    fmt.Printf("Available commands:\n")
    for name, cmd := range app.Commands {
        fmt.Printf("  %-12s %s\n", name, cmd.Description())
    }
    
    fmt.Printf("\nGlobal options:\n")
    flag.PrintDefaults()
}

func (app *Application) Run(args []string) error {
    if len(args) == 0 {
        app.ShowHelp()
        return nil
    }
    
    commandName := args[0]
    command, exists := app.Commands[commandName]
    if !exists {
        return fmt.Errorf("unknown command: %s", commandName)
    }
    
    app.Logger.Info("Executing command: %s", commandName)
    return command.Execute(args[1:], app.Config)
}

func main() {
    var (
        // Configuration flags
        configFile  = flag.String("config", "", "configuration file path")
        logLevel    = flag.String("log-level", "info", "log level (debug, info, warn, error)")
        logFormat   = flag.String("log-format", "text", "log format (text, json)")
        timeout     = flag.Duration("timeout", 30*time.Second, "operation timeout")
        maxRetries  = flag.Int("max-retries", 3, "maximum retry attempts")
        outputDir   = flag.String("output-dir", "./output", "output directory")
        tempDir     = flag.String("temp-dir", "/tmp", "temporary directory")
        concurrent  = flag.Bool("concurrent", true, "enable concurrent processing")
        maxWorkers  = flag.Int("max-workers", 4, "maximum worker goroutines")
        verbose     = flag.Bool("verbose", false, "verbose output")
        dryRun      = flag.Bool("dry-run", false, "dry run mode")
        
        // Utility flags
        version = flag.Bool("version", false, "show version information")
        help    = flag.Bool("help", false, "show help information")
    )
    
    // Custom usage function
    flag.Usage = func() {
        fmt.Printf("Usage: %s [options] <command> [command-options]\n\n", os.Args[0])
        fmt.Printf("A comprehensive CLI application demonstrating Go flag patterns.\n\n")
        fmt.Printf("Options:\n")
        flag.PrintDefaults()
        fmt.Printf("\nCommands:\n")
        fmt.Printf("  process      Process files with various options\n")
        fmt.Printf("  status       Show application status\n")
        fmt.Printf("\nExamples:\n")
        fmt.Printf("  %s -verbose process file1.txt file2.txt\n", os.Args[0])
        fmt.Printf("  %s -dry-run -max-workers 8 process *.txt\n", os.Args[0])
        fmt.Printf("  %s -config myapp.json status\n", os.Args[0])
    }
    
    flag.Parse()
    
    app := NewApplication()
    
    // Handle version flag
    if *version {
        fmt.Printf("%s version %s\n", app.Name, app.Version)
        return
    }
    
    // Handle help flag
    if *help {
        flag.Usage()
        return
    }
    
    // Load configuration
    if err := app.LoadConfig(*configFile); err != nil {
        fmt.Printf("Configuration error: %v\n", err)
        os.Exit(1)
    }
    
    // Apply command-line flag overrides
    if flag.Lookup("log-level").Value.String() != "info" {
        app.Config.LogLevel = *logLevel
    }
    if flag.Lookup("log-format").Value.String() != "text" {
        app.Config.LogFormat = *logFormat
    }
    app.Config.Timeout = *timeout
    app.Config.MaxRetries = *maxRetries
    app.Config.OutputDir = *outputDir
    app.Config.TempDir = *tempDir
    app.Config.Concurrent = *concurrent
    app.Config.MaxWorkers = *maxWorkers
    app.Config.Verbose = *verbose
    app.Config.DryRun = *dryRun
    
    // Initialize logger
    app.InitializeLogger()
    
    // Ensure output directory exists
    if err := os.MkdirAll(app.Config.OutputDir, 0755); err != nil {
        app.Logger.Error("Failed to create output directory: %v", err)
        os.Exit(1)
    }
    
    if app.Config.Verbose {
        app.Logger.Info("Application started with config: %+v", app.Config)
    }
    
    // Execute command
    args := flag.Args()
    if err := app.Run(args); err != nil {
        app.Logger.Error("Command execution failed: %v", err)
        os.Exit(1)
    }
    
    app.Logger.Info("Application completed successfully")
}
```

This comprehensive CLI application template demonstrates integration of  
multiple flag patterns including configuration files, logging, subcommands,  
validation, and structured application architecture. It provides a solid  
foundation for building sophisticated command-line tools with professional  
features and maintainable code organization.  