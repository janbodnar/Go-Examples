# Go Logging

Logging is a crucial aspect of application development that provides visibility  
into application behavior, performance monitoring, debugging capabilities, and  
operational insights. In Go, logging encompasses various approaches from simple  
console output to sophisticated structured logging systems with multiple output  
destinations, log levels, and formatting options.

Go's approach to logging emphasizes simplicity and performance while providing  
flexibility for different use cases. The standard library includes the `log`  
package for basic logging functionality, while the newer `log/slog` package  
introduced in Go 1.21 provides structured logging capabilities. Additionally,  
the ecosystem offers powerful third-party logging libraries like Logrus and Zap  
that extend functionality with features like structured logging, multiple output  
formats, and high-performance implementations.

Effective logging involves several key considerations: choosing appropriate log  
levels to balance information and noise, implementing structured logging for  
machine readability, managing log output destinations and rotation, ensuring  
thread safety in concurrent applications, and optimizing performance to minimize  
logging overhead. Modern applications often require contextual logging that  
includes request IDs, user information, and other metadata to correlate log  
entries across distributed systems.

Log levels provide a hierarchy for categorizing log messages by importance:  
DEBUG for detailed diagnostic information, INFO for general application flow,  
WARN for potentially problematic situations, ERROR for error conditions that  
don't stop execution, and FATAL for critical errors that terminate the program.  
Proper level usage enables filtering and routing of log messages based on  
deployment environment and operational requirements.

Structured logging formats logs as key-value pairs or JSON objects rather than  
plain text, enabling automated log processing, searching, and analysis. This  
approach is essential for modern observability practices where logs are ingested  
by centralized logging systems, analyzed for patterns, and correlated with  
metrics and traces to provide comprehensive application insights.

## Basic console logging

The `fmt` package provides the simplest logging approach using print functions  
that output directly to the console.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("Application starting...")
    
    // Simulate application work
    fmt.Printf("Processing started at: %v\n", time.Now())
    
    // Simple progress logging
    for i := 1; i <= 3; i++ {
        fmt.Printf("Step %d completed\n", i)
        time.Sleep(500 * time.Millisecond)
    }
    
    fmt.Println("Application finished successfully")
}
```

This example demonstrates basic console output using `fmt.Println` and  
`fmt.Printf`. While simple, this approach lacks log levels, timestamps, and  
structured formatting that production applications require.  

## Standard log package usage

The `log` package provides enhanced logging capabilities with automatic  
timestamps and customizable output formatting.  

```go
package main

import (
    "fmt"
    "log"
    "time"
)

func processData(data string) error {
    log.Printf("Processing data: %s", data)
    
    // Simulate processing time
    time.Sleep(100 * time.Millisecond)
    
    if data == "invalid" {
        log.Printf("Error: invalid data encountered")
        return fmt.Errorf("invalid data: %s", data)
    }
    
    log.Printf("Successfully processed: %s", data)
    return nil
}

func main() {
    log.Println("Application started")
    
    datasets := []string{"user_data", "config", "invalid", "metrics"}
    
    for _, data := range datasets {
        if err := processData(data); err != nil {
            log.Printf("Failed to process %s: %v", data, err)
        }
    }
    
    log.Println("Processing complete")
}
```

The `log` package automatically adds timestamps and provides a consistent format  
for log messages. Functions like `log.Printf` support formatted output, while  
`log.Println` handles simple string messages with automatic newlines.  

## Custom logger with levels

Creating custom loggers allows fine-grained control over log formatting, levels,  
and output destinations.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
    "time"
)

type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
    FATAL
)

func (l LogLevel) String() string {
    return []string{"DEBUG", "INFO", "WARN", "ERROR", "FATAL"}[l]
}

type Logger struct {
    level  LogLevel
    logger *log.Logger
}

func NewLogger(level LogLevel, output io.Writer) *Logger {
    return &Logger{
        level:  level,
        logger: log.New(output, "", 0),
    }
}

func (l *Logger) log(level LogLevel, message string) {
    if level < l.level {
        return
    }
    
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    levelStr := level.String()
    
    l.logger.Printf("[%s] %s: %s", timestamp, levelStr, message)
}

func (l *Logger) Debug(message string) { l.log(DEBUG, message) }
func (l *Logger) Info(message string)  { l.log(INFO, message) }
func (l *Logger) Warn(message string)  { l.log(WARN, message) }
func (l *Logger) Error(message string) { l.log(ERROR, message) }
func (l *Logger) Fatal(message string) {
    l.log(FATAL, message)
    os.Exit(1)
}

func (l *Logger) Debugf(format string, args ...interface{}) {
    l.log(DEBUG, fmt.Sprintf(format, args...))
}

func (l *Logger) Infof(format string, args ...interface{}) {
    l.log(INFO, fmt.Sprintf(format, args...))
}

func (l *Logger) Warnf(format string, args ...interface{}) {
    l.log(WARN, fmt.Sprintf(format, args...))
}

func (l *Logger) Errorf(format string, args ...interface{}) {
    l.log(ERROR, fmt.Sprintf(format, args...))
}

func main() {
    logger := NewLogger(INFO, os.Stdout)
    
    logger.Debug("This debug message won't appear")
    logger.Info("Application starting")
    logger.Warnf("Configuration file not found, using defaults")
    logger.Errorf("Failed to connect to database: %s", "connection timeout")
    
    // Demonstrate level filtering
    fmt.Println("\nChanging log level to DEBUG:")
    debugLogger := NewLogger(DEBUG, os.Stdout)
    debugLogger.Debug("Now debug messages appear")
    debugLogger.Info("Detailed logging enabled")
}
```

This custom logger implementation provides log level filtering, consistent  
formatting, and both simple and formatted logging methods. The level filtering  
ensures that only messages at or above the configured level are output.  

## File-based logging

Logging to files enables persistent storage and analysis of application logs  
beyond console output.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "path/filepath"
    "time"
)

func setupFileLogger(filename string) (*log.Logger, *os.File, error) {
    // Ensure log directory exists
    logDir := filepath.Dir(filename)
    if err := os.MkdirAll(logDir, 0755); err != nil {
        return nil, nil, fmt.Errorf("failed to create log directory: %v", err)
    }
    
    // Open log file with append mode
    logFile, err := os.OpenFile(filename, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        return nil, nil, fmt.Errorf("failed to open log file: %v", err)
    }
    
    // Create logger with custom prefix and flags
    logger := log.New(logFile, "[APP] ", log.Ldate|log.Ltime|log.Lmicroseconds)
    
    return logger, logFile, nil
}

func simulateApplicationWork(logger *log.Logger) {
    logger.Println("Starting data processing")
    
    for i := 1; i <= 5; i++ {
        logger.Printf("Processing batch %d/5", i)
        
        // Simulate varying processing times
        processingTime := time.Duration(i*100) * time.Millisecond
        time.Sleep(processingTime)
        
        if i == 3 {
            logger.Printf("Warning: Batch %d took longer than expected", i)
        }
    }
    
    logger.Println("Data processing completed successfully")
}

func main() {
    logFile := "/tmp/app.log"
    logger, file, err := setupFileLogger(logFile)
    if err != nil {
        fmt.Printf("Error setting up file logger: %v\n", err)
        return
    }
    defer file.Close()
    
    // Log to both file and console
    logger.Println("Application started - logging to file")
    fmt.Println("Application started - check log file at:", logFile)
    
    simulateApplicationWork(logger)
    
    logger.Println("Application finished")
    fmt.Println("Application finished - logs saved to:", logFile)
    
    // Display log file contents
    fmt.Println("\nLog file contents:")
    content, err := os.ReadFile(logFile)
    if err != nil {
        fmt.Printf("Error reading log file: %v\n", err)
        return
    }
    fmt.Print(string(content))
}
```

File-based logging provides persistent log storage with automatic directory  
creation and proper file handling. The example demonstrates log file creation,  
custom formatting flags, and safe file closure using defer.  

## Multiple output destinations

Logging to multiple destinations simultaneously enables both real-time monitoring  
and persistent storage.  

```go
package main

import (
    "io"
    "log"
    "os"
    "strings"
    "time"
)

type MultiWriter struct {
    writers []io.Writer
}

func NewMultiWriter(writers ...io.Writer) *MultiWriter {
    return &MultiWriter{writers: writers}
}

func (mw *MultiWriter) Write(p []byte) (n int, err error) {
    for _, writer := range mw.writers {
        if _, err := writer.Write(p); err != nil {
            return 0, err
        }
    }
    return len(p), nil
}

func simulateWebServerLogs(logger *log.Logger) {
    requests := []struct {
        method string
        path   string
        status int
        time   time.Duration
    }{
        {"GET", "/api/users", 200, 45 * time.Millisecond},
        {"POST", "/api/login", 200, 120 * time.Millisecond},
        {"GET", "/api/invalid", 404, 15 * time.Millisecond},
        {"GET", "/api/data", 500, 200 * time.Millisecond},
        {"PUT", "/api/users/123", 200, 80 * time.Millisecond},
    }
    
    for _, req := range requests {
        logger.Printf("%s %s - %d - %v", req.method, req.path, req.status, req.time)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // Create log file
    logFile, err := os.OpenFile("/tmp/server.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        log.Fatalf("Failed to open log file: %v", err)
    }
    defer logFile.Close()
    
    // Create multi-writer for console and file output
    multiWriter := NewMultiWriter(os.Stdout, logFile)
    
    // Create logger with multiple outputs
    logger := log.New(multiWriter, "[SERVER] ", log.Ldate|log.Ltime)
    
    logger.Println("Web server starting...")
    logger.Println("Logging to both console and file: /tmp/server.log")
    
    simulateWebServerLogs(logger)
    
    logger.Println("Server shutdown complete")
    
    // Show what was written to file
    fmt.Println("\n--- File contents ---")
    content, err := os.ReadFile("/tmp/server.log")
    if err != nil {
        log.Printf("Error reading log file: %v", err)
        return
    }
    
    lines := strings.Split(string(content), "\n")
    for _, line := range lines {
        if strings.TrimSpace(line) != "" {
            fmt.Println("FILE:", line)
        }
    }
}
```

The MultiWriter implementation enables simultaneous logging to multiple  
destinations. This pattern is valuable for applications that need real-time  
console output during development and persistent file storage in production.  

## Structured JSON logging

JSON-formatted logs enable automated parsing and analysis by log management  
systems and monitoring tools.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "time"
)

type LogEntry struct {
    Timestamp string                 `json:"timestamp"`
    Level     string                 `json:"level"`
    Message   string                 `json:"message"`
    Fields    map[string]interface{} `json:"fields,omitempty"`
}

type JSONLogger struct {
    output *os.File
}

func NewJSONLogger(output *os.File) *JSONLogger {
    return &JSONLogger{output: output}
}

func (jl *JSONLogger) log(level, message string, fields map[string]interface{}) {
    entry := LogEntry{
        Timestamp: time.Now().UTC().Format(time.RFC3339),
        Level:     level,
        Message:   message,
        Fields:    fields,
    }
    
    jsonData, err := json.Marshal(entry)
    if err != nil {
        fmt.Fprintf(os.Stderr, "Failed to marshal log entry: %v\n", err)
        return
    }
    
    jl.output.Write(jsonData)
    jl.output.WriteString("\n")
}

func (jl *JSONLogger) Info(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    jl.log("INFO", message, f)
}

func (jl *JSONLogger) Warn(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    jl.log("WARN", message, f)
}

func (jl *JSONLogger) Error(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    jl.log("ERROR", message, f)
}

func simulateUserOperations(logger *JSONLogger) {
    operations := []struct {
        user   string
        action string
        result string
        data   map[string]interface{}
    }{
        {"alice", "login", "success", map[string]interface{}{"ip": "192.168.1.100", "duration_ms": 45}},
        {"bob", "create_post", "success", map[string]interface{}{"post_id": "12345", "title": "Hello there"}},
        {"charlie", "login", "failed", map[string]interface{}{"ip": "192.168.1.200", "reason": "invalid_password"}},
        {"alice", "update_profile", "success", map[string]interface{}{"fields_changed": []string{"email", "phone"}}},
        {"bob", "delete_post", "failed", map[string]interface{}{"post_id": "99999", "reason": "not_found"}},
    }
    
    for _, op := range operations {
        fields := map[string]interface{}{
            "user":   op.user,
            "action": op.action,
            "result": op.result,
        }
        
        // Merge operation-specific data
        for k, v := range op.data {
            fields[k] = v
        }
        
        switch op.result {
        case "success":
            logger.Info("User operation completed", fields)
        case "failed":
            logger.Error("User operation failed", fields)
        default:
            logger.Warn("User operation with unknown result", fields)
        }
        
        time.Sleep(200 * time.Millisecond)
    }
}

func main() {
    logger := NewJSONLogger(os.Stdout)
    
    logger.Info("Application started", map[string]interface{}{
        "version": "1.0.0",
        "env":     "development",
    })
    
    simulateUserOperations(logger)
    
    logger.Info("Application shutdown initiated", map[string]interface{}{
        "uptime_seconds": 10,
        "active_users":   3,
    })
}
```

Structured JSON logging provides machine-readable output with consistent field  
naming and typed values. Each log entry includes a timestamp, level, message,  
and optional fields containing contextual information for analysis and debugging.  

## Contextual logging with fields

Adding contextual information to log entries helps trace application flow and  
correlate related events across distributed systems.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "math/rand"
    "strings"
    "time"
)

type ContextLogger struct {
    prefix string
    fields map[string]interface{}
}

func NewContextLogger() *ContextLogger {
    return &ContextLogger{
        fields: make(map[string]interface{}),
    }
}

func (cl *ContextLogger) WithField(key string, value interface{}) *ContextLogger {
    newLogger := &ContextLogger{
        prefix: cl.prefix,
        fields: make(map[string]interface{}),
    }
    
    // Copy existing fields
    for k, v := range cl.fields {
        newLogger.fields[k] = v
    }
    
    // Add new field
    newLogger.fields[key] = value
    
    return newLogger
}

func (cl *ContextLogger) WithFields(fields map[string]interface{}) *ContextLogger {
    newLogger := &ContextLogger{
        prefix: cl.prefix,
        fields: make(map[string]interface{}),
    }
    
    // Copy existing fields
    for k, v := range cl.fields {
        newLogger.fields[k] = v
    }
    
    // Add new fields
    for k, v := range fields {
        newLogger.fields[k] = v
    }
    
    return newLogger
}

func (cl *ContextLogger) log(level, message string) {
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    
    var fieldStr strings.Builder
    if len(cl.fields) > 0 {
        fieldStr.WriteString(" [")
        first := true
        for k, v := range cl.fields {
            if !first {
                fieldStr.WriteString(", ")
            }
            fieldStr.WriteString(fmt.Sprintf("%s=%v", k, v))
            first = false
        }
        fieldStr.WriteString("]")
    }
    
    log.Printf("[%s] %s: %s%s", timestamp, level, message, fieldStr.String())
}

func (cl *ContextLogger) Info(message string)  { cl.log("INFO", message) }
func (cl *ContextLogger) Warn(message string)  { cl.log("WARN", message) }
func (cl *ContextLogger) Error(message string) { cl.log("ERROR", message) }

func processOrder(ctx context.Context, orderID string, userID string) {
    // Create request-scoped logger
    logger := NewContextLogger().WithFields(map[string]interface{}{
        "order_id": orderID,
        "user_id":  userID,
        "trace_id": fmt.Sprintf("trace-%d", rand.Intn(10000)),
    })
    
    logger.Info("Starting order processing")
    
    // Simulate order validation
    time.Sleep(100 * time.Millisecond)
    logger.WithField("step", "validation").Info("Order validation completed")
    
    // Simulate payment processing
    time.Sleep(200 * time.Millisecond)
    paymentLogger := logger.WithField("step", "payment")
    
    if rand.Float32() < 0.8 { // 80% success rate
        paymentLogger.WithField("amount", "$29.99").Info("Payment processed successfully")
        
        // Simulate inventory update
        time.Sleep(150 * time.Millisecond)
        logger.WithFields(map[string]interface{}{
            "step":     "inventory",
            "product":  "Go Programming Book",
            "quantity": 1,
        }).Info("Inventory updated")
        
        logger.Info("Order processing completed successfully")
    } else {
        paymentLogger.WithField("reason", "insufficient_funds").Error("Payment processing failed")
        logger.Error("Order processing failed")
    }
}

func main() {
    baseLogger := NewContextLogger()
    baseLogger.WithField("service", "order-api").Info("Service starting")
    
    // Simulate multiple concurrent orders
    orders := []struct {
        orderID string
        userID  string
    }{
        {"order-001", "user-123"},
        {"order-002", "user-456"},
        {"order-003", "user-789"},
    }
    
    for _, order := range orders {
        ctx := context.Background()
        processOrder(ctx, order.orderID, order.userID)
        fmt.Println() // Add spacing between orders
    }
    
    baseLogger.WithField("service", "order-api").Info("Service shutdown")
}
```

Contextual logging enables adding structured fields to log entries, making it  
easier to trace requests, correlate events, and filter logs by specific criteria.  
This approach is essential for debugging complex applications and distributed  
systems.  

## Performance-optimized logging

High-performance applications require logging implementations that minimize  
overhead while maintaining functionality.  

```go
package main

import (
    "fmt"
    "runtime"
    "sync"
    "time"
)

type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
)

type FastLogger struct {
    level   LogLevel
    pool    sync.Pool
    channel chan *LogEntry
    done    chan struct{}
    wg      sync.WaitGroup
}

type LogEntry struct {
    Level     LogLevel
    Message   string
    Timestamp time.Time
}

func NewFastLogger(level LogLevel, bufferSize int) *FastLogger {
    fl := &FastLogger{
        level:   level,
        channel: make(chan *LogEntry, bufferSize),
        done:    make(chan struct{}),
    }
    
    // Object pool for log entries to reduce allocations
    fl.pool = sync.Pool{
        New: func() interface{} {
            return &LogEntry{}
        },
    }
    
    // Start background goroutine for log processing
    fl.wg.Add(1)
    go fl.processLogs()
    
    return fl
}

func (fl *FastLogger) processLogs() {
    defer fl.wg.Done()
    
    for {
        select {
        case entry := <-fl.channel:
            // Process log entry
            levelStr := []string{"DEBUG", "INFO", "WARN", "ERROR"}[entry.Level]
            fmt.Printf("[%s] %s: %s\n",
                entry.Timestamp.Format("15:04:05.000"),
                levelStr,
                entry.Message)
            
            // Return entry to pool
            fl.pool.Put(entry)
            
        case <-fl.done:
            // Process remaining log entries
            for len(fl.channel) > 0 {
                entry := <-fl.channel
                levelStr := []string{"DEBUG", "INFO", "WARN", "ERROR"}[entry.Level]
                fmt.Printf("[%s] %s: %s\n",
                    entry.Timestamp.Format("15:04:05.000"),
                    levelStr,
                    entry.Message)
                fl.pool.Put(entry)
            }
            return
        }
    }
}

func (fl *FastLogger) log(level LogLevel, message string) {
    if level < fl.level {
        return
    }
    
    // Get entry from pool
    entry := fl.pool.Get().(*LogEntry)
    entry.Level = level
    entry.Message = message
    entry.Timestamp = time.Now()
    
    // Send to processing channel (non-blocking)
    select {
    case fl.channel <- entry:
        // Entry sent successfully
    default:
        // Channel full, drop log entry to prevent blocking
        fl.pool.Put(entry)
    }
}

func (fl *FastLogger) Debug(message string) { fl.log(DEBUG, message) }
func (fl *FastLogger) Info(message string)  { fl.log(INFO, message) }
func (fl *FastLogger) Warn(message string)  { fl.log(WARN, message) }
func (fl *FastLogger) Error(message string) { fl.log(ERROR, message) }

func (fl *FastLogger) Close() {
    close(fl.done)
    fl.wg.Wait()
    close(fl.channel)
}

func benchmarkLogging(logger *FastLogger, messages int) time.Duration {
    start := time.Now()
    
    for i := 0; i < messages; i++ {
        logger.Info(fmt.Sprintf("Benchmark message %d", i))
    }
    
    return time.Since(start)
}

func memoryUsage() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("Memory: Alloc=%d KB, Sys=%d KB, NumGC=%d\n",
        m.Alloc/1024, m.Sys/1024, m.NumGC)
}

func main() {
    logger := NewFastLogger(INFO, 1000)
    defer logger.Close()
    
    fmt.Println("=== Performance-Optimized Logging Demo ===")
    memoryUsage()
    
    // Benchmark logging performance
    messageCount := 10000
    fmt.Printf("\nLogging %d messages...\n", messageCount)
    
    duration := benchmarkLogging(logger, messageCount)
    fmt.Printf("Completed in: %v\n", duration)
    fmt.Printf("Messages per second: %.0f\n", float64(messageCount)/duration.Seconds())
    
    // Allow background processing to complete
    time.Sleep(100 * time.Millisecond)
    memoryUsage()
    
    // Demonstrate different log levels
    fmt.Println("\n=== Log Level Examples ===")
    logger.Debug("Debug message (filtered out)")
    logger.Info("Application processing data")
    logger.Warn("Memory usage approaching limit")
    logger.Error("Failed to process request")
    
    // Brief delay to see async log output
    time.Sleep(50 * time.Millisecond)
}
```

This high-performance logger uses object pooling to reduce garbage collection  
pressure, asynchronous processing to prevent blocking the main thread, and  
buffered channels to handle burst logging scenarios efficiently.  

## Colored console output

Adding colors to console logs improves readability and helps quickly identify  
different log levels during development.  

```go
package main

import (
    "fmt"
    "os"
    "runtime"
    "time"
)

type ColorCode string

const (
    ColorReset  ColorCode = "\033[0m"
    ColorRed    ColorCode = "\033[31m"
    ColorYellow ColorCode = "\033[33m"
    ColorBlue   ColorCode = "\033[34m"
    ColorGreen  ColorCode = "\033[32m"
    ColorCyan   ColorCode = "\033[36m"
    ColorWhite  ColorCode = "\033[37m"
    ColorGray   ColorCode = "\033[90m"
)

type ColoredLogger struct {
    enableColors bool
}

func NewColoredLogger() *ColoredLogger {
    return &ColoredLogger{
        enableColors: isTerminal(),
    }
}

func isTerminal() bool {
    // Simple terminal detection - in production you might use a more robust method
    return os.Getenv("TERM") != "" && runtime.GOOS != "windows"
}

func (cl *ColoredLogger) colorize(color ColorCode, text string) string {
    if !cl.enableColors {
        return text
    }
    return string(color) + text + string(ColorReset)
}

func (cl *ColoredLogger) log(level, message string, color ColorCode) {
    timestamp := time.Now().Format("15:04:05")
    coloredLevel := cl.colorize(color, level)
    
    fmt.Printf("[%s] %s: %s\n", 
        cl.colorize(ColorGray, timestamp), 
        coloredLevel, 
        message)
}

func (cl *ColoredLogger) Debug(message string) {
    cl.log("DEBUG", message, ColorCyan)
}

func (cl *ColoredLogger) Info(message string) {
    cl.log("INFO ", message, ColorGreen)
}

func (cl *ColoredLogger) Warn(message string) {
    cl.log("WARN ", message, ColorYellow)
}

func (cl *ColoredLogger) Error(message string) {
    cl.log("ERROR", message, ColorRed)
}

func (cl *ColoredLogger) Fatal(message string) {
    cl.log("FATAL", message, ColorRed)
    os.Exit(1)
}

func simulateApplicationFlow(logger *ColoredLogger) {
    logger.Info("Application initialization started")
    
    // Simulate startup sequence
    logger.Debug("Loading configuration files")
    time.Sleep(100 * time.Millisecond)
    
    logger.Info("Database connection established")
    time.Sleep(150 * time.Millisecond)
    
    logger.Debug("Setting up HTTP routes")
    logger.Info("Web server listening on port 8080")
    
    // Simulate some operations
    logger.Info("Processing incoming request")
    logger.Debug("Validating user authentication")
    
    logger.Warn("High memory usage detected")
    time.Sleep(200 * time.Millisecond)
    
    logger.Error("Failed to connect to external API")
    logger.Info("Falling back to cached data")
    
    logger.Debug("Request processing completed")
    logger.Info("Response sent successfully")
}

func demonstrateColorCodes() {
    fmt.Println("\n=== Color Code Demonstration ===")
    fmt.Printf("%sRed Text%s\n", ColorRed, ColorReset)
    fmt.Printf("%sYellow Text%s\n", ColorYellow, ColorReset)
    fmt.Printf("%sGreen Text%s\n", ColorGreen, ColorReset)
    fmt.Printf("%sBlue Text%s\n", ColorBlue, ColorReset)
    fmt.Printf("%sCyan Text%s\n", ColorCyan, ColorReset)
    fmt.Printf("%sGray Text%s\n", ColorGray, ColorReset)
}

func main() {
    logger := NewColoredLogger()
    
    fmt.Printf("Terminal colors enabled: %t\n", logger.enableColors)
    
    demonstrateColorCodes()
    
    fmt.Println("\n=== Application Log Flow ===")
    simulateApplicationFlow(logger)
    
    // Show difference with colors disabled
    fmt.Println("\n=== Same Logs Without Colors ===")
    logger.enableColors = false
    logger.Info("Application starting (no colors)")
    logger.Warn("This is a warning (no colors)")
    logger.Error("This is an error (no colors)")
}
```

Colored logging enhances the developer experience by making different log levels  
visually distinct. The implementation includes terminal detection and graceful  
fallback to uncolored output when colors aren't supported.  

## Log rotation and file management

Production applications require log rotation to prevent log files from consuming  
excessive disk space while maintaining historical data.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "path/filepath"
    "strings"
    "time"
)

type RotatingLogger struct {
    baseFilename string
    maxSize      int64  // bytes
    maxAge       int    // days
    maxBackups   int    // number of old log files to retain
    currentSize  int64
    logger       *log.Logger
    file         *os.File
}

func NewRotatingLogger(baseFilename string, maxSize int64, maxAge, maxBackups int) (*RotatingLogger, error) {
    rl := &RotatingLogger{
        baseFilename: baseFilename,
        maxSize:      maxSize,
        maxAge:       maxAge,
        maxBackups:   maxBackups,
    }
    
    if err := rl.openLogFile(); err != nil {
        return nil, err
    }
    
    return rl, nil
}

func (rl *RotatingLogger) openLogFile() error {
    // Ensure directory exists
    dir := filepath.Dir(rl.baseFilename)
    if err := os.MkdirAll(dir, 0755); err != nil {
        return fmt.Errorf("failed to create log directory: %v", err)
    }
    
    // Get current file size if it exists
    if info, err := os.Stat(rl.baseFilename); err == nil {
        rl.currentSize = info.Size()
    }
    
    // Open log file
    file, err := os.OpenFile(rl.baseFilename, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        return fmt.Errorf("failed to open log file: %v", err)
    }
    
    rl.file = file
    rl.logger = log.New(file, "", log.Ldate|log.Ltime|log.Lmicroseconds)
    
    return nil
}

func (rl *RotatingLogger) Write(p []byte) (n int, err error) {
    // Check if rotation is needed
    if rl.currentSize+int64(len(p)) > rl.maxSize {
        if err := rl.rotate(); err != nil {
            return 0, err
        }
    }
    
    // Write to current file
    n, err = rl.file.Write(p)
    rl.currentSize += int64(n)
    
    return n, err
}

func (rl *RotatingLogger) rotate() error {
    // Close current file
    if rl.file != nil {
        rl.file.Close()
    }
    
    // Rotate existing files
    for i := rl.maxBackups - 1; i >= 1; i-- {
        oldName := fmt.Sprintf("%s.%d", rl.baseFilename, i)
        newName := fmt.Sprintf("%s.%d", rl.baseFilename, i+1)
        
        if _, err := os.Stat(oldName); err == nil {
            os.Rename(oldName, newName)
        }
    }
    
    // Move current file to .1
    if _, err := os.Stat(rl.baseFilename); err == nil {
        backupName := fmt.Sprintf("%s.1", rl.baseFilename)
        if err := os.Rename(rl.baseFilename, backupName); err != nil {
            return fmt.Errorf("failed to rotate log file: %v", err)
        }
    }
    
    // Clean old files based on maxBackups
    rl.cleanOldFiles()
    
    // Open new file
    rl.currentSize = 0
    return rl.openLogFile()
}

func (rl *RotatingLogger) cleanOldFiles() {
    for i := rl.maxBackups + 1; i <= rl.maxBackups + 10; i++ {
        oldFile := fmt.Sprintf("%s.%d", rl.baseFilename, i)
        os.Remove(oldFile)
    }
    
    // Clean files older than maxAge
    if rl.maxAge > 0 {
        cutoff := time.Now().AddDate(0, 0, -rl.maxAge)
        pattern := rl.baseFilename + ".*"
        
        matches, _ := filepath.Glob(pattern)
        for _, match := range matches {
            if info, err := os.Stat(match); err == nil {
                if info.ModTime().Before(cutoff) {
                    os.Remove(match)
                }
            }
        }
    }
}

func (rl *RotatingLogger) Printf(format string, v ...interface{}) {
    rl.logger.Printf(format, v...)
}

func (rl *RotatingLogger) Println(v ...interface{}) {
    rl.logger.Println(v...)
}

func (rl *RotatingLogger) Close() error {
    if rl.file != nil {
        return rl.file.Close()
    }
    return nil
}

func generateLogData(logger *RotatingLogger) {
    for i := 1; i <= 100; i++ {
        message := fmt.Sprintf("Log entry %d - This is a test message with some data: %s", 
            i, strings.Repeat("data", 10))
        logger.Println(message)
        
        if i%10 == 0 {
            logger.Printf("Batch %d completed at %v", i/10, time.Now())
        }
        
        time.Sleep(10 * time.Millisecond)
    }
}

func listLogFiles(baseFilename string) {
    fmt.Println("\n=== Log Files ===")
    dir := filepath.Dir(baseFilename)
    baseName := filepath.Base(baseFilename)
    
    files, err := os.ReadDir(dir)
    if err != nil {
        fmt.Printf("Error reading directory: %v\n", err)
        return
    }
    
    for _, file := range files {
        if strings.HasPrefix(file.Name(), baseName) {
            info, _ := file.Info()
            fmt.Printf("File: %s, Size: %d bytes, Modified: %v\n", 
                file.Name(), info.Size(), info.ModTime().Format("2006-01-02 15:04:05"))
        }
    }
}

func main() {
    logFile := "/tmp/rotating.log"
    
    // Create rotating logger with small max size to trigger rotation
    logger, err := NewRotatingLogger(logFile, 1024, 7, 3) // 1KB max, 7 days, 3 backups
    if err != nil {
        log.Fatalf("Failed to create rotating logger: %v", err)
    }
    defer logger.Close()
    
    fmt.Printf("Starting log rotation demo with file: %s\n", logFile)
    fmt.Println("Max size: 1KB, Max age: 7 days, Max backups: 3")
    
    logger.Println("=== Log Rotation Demo Started ===")
    generateLogData(logger)
    logger.Println("=== Log Rotation Demo Completed ===")
    
    listLogFiles(logFile)
    
    // Show content of main log file
    fmt.Println("\n=== Current Log File Content ===")
    content, err := os.ReadFile(logFile)
    if err != nil {
        fmt.Printf("Error reading log file: %v\n", err)
        return
    }
    
    lines := strings.Split(string(content), "\n")
    for i, line := range lines {
        if strings.TrimSpace(line) != "" && i < 10 { // Show first 10 lines
            fmt.Println(line)
        }
    }
    
    if len(lines) > 10 {
        fmt.Printf("... and %d more lines\n", len(lines)-10)
    }
}
```

Log rotation prevents disk space issues by automatically creating backup files  
when the current log reaches a size limit. The implementation includes cleanup  
of old files based on age and backup count limits.  

## Concurrent logging patterns

Thread-safe logging is essential for applications with multiple goroutines  
writing log messages simultaneously.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "sync"
    "time"
)

type SafeLogger struct {
    mu     sync.RWMutex
    logger *log.Logger
    stats  LogStats
}

type LogStats struct {
    TotalMessages int64
    DebugCount    int64
    InfoCount     int64
    WarnCount     int64
    ErrorCount    int64
}

func NewSafeLogger(output *os.File) *SafeLogger {
    return &SafeLogger{
        logger: log.New(output, "", log.Ldate|log.Ltime|log.Lmicroseconds),
    }
}

func (sl *SafeLogger) log(level string, message string) {
    sl.mu.Lock()
    defer sl.mu.Unlock()
    
    sl.logger.Printf("[%s] %s", level, message)
    sl.stats.TotalMessages++
    
    // Update level-specific counters
    switch level {
    case "DEBUG":
        sl.stats.DebugCount++
    case "INFO":
        sl.stats.InfoCount++
    case "WARN":
        sl.stats.WarnCount++
    case "ERROR":
        sl.stats.ErrorCount++
    }
}

func (sl *SafeLogger) Debug(message string) { sl.log("DEBUG", message) }
func (sl *SafeLogger) Info(message string)  { sl.log("INFO", message) }
func (sl *SafeLogger) Warn(message string)  { sl.log("WARN", message) }
func (sl *SafeLogger) Error(message string) { sl.log("ERROR", message) }

func (sl *SafeLogger) GetStats() LogStats {
    sl.mu.RLock()
    defer sl.mu.RUnlock()
    return sl.stats
}

func worker(id int, logger *SafeLogger, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for i := 1; i <= 10; i++ {
        switch i % 4 {
        case 0:
            logger.Debug(fmt.Sprintf("Worker %d: Debug message %d", id, i))
        case 1:
            logger.Info(fmt.Sprintf("Worker %d: Processing task %d", id, i))
        case 2:
            logger.Warn(fmt.Sprintf("Worker %d: Task %d took longer than expected", id, i))
        case 3:
            logger.Error(fmt.Sprintf("Worker %d: Error in task %d", id, i))
        }
        
        // Simulate work with varying delays
        time.Sleep(time.Duration(10+i*5) * time.Millisecond)
    }
    
    logger.Info(fmt.Sprintf("Worker %d completed all tasks", id))
}

func main() {
    fmt.Println("=== Concurrent Logging Patterns Demo ===")
    
    safeLogger := NewSafeLogger(os.Stdout)
    
    var wg sync.WaitGroup
    numWorkers := 5
    
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, safeLogger, &wg)
    }
    
    wg.Wait()
    
    stats := safeLogger.GetStats()
    fmt.Printf("\nLogging Statistics:\n")
    fmt.Printf("Total Messages: %d\n", stats.TotalMessages)
    fmt.Printf("Debug: %d, Info: %d, Warn: %d, Error: %d\n", 
        stats.DebugCount, stats.InfoCount, stats.WarnCount, stats.ErrorCount)
}
```

Concurrent logging requires careful consideration of thread safety. This example  
demonstrates mutex-based synchronization for safe concurrent access to logging  
resources while maintaining performance and collecting statistics.  

## Using the slog package

Go 1.21 introduced the `log/slog` package for structured logging with improved  
performance and flexibility compared to the traditional log package.  

```go
package main

import (
    "context"
    "fmt"
    "log/slog"
    "os"
    "time"
)

func demonstrateBasicSlog() {
    // Default text handler
    logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
    
    logger.Info("Application started")
    logger.Warn("This is a warning message")
    logger.Error("An error occurred", "error", "file not found")
    
    // With attributes
    logger.Info("User login", 
        "user", "alice", 
        "ip", "192.168.1.100",
        "timestamp", time.Now())
}

func demonstrateJSONHandler() {
    // JSON handler for structured output
    opts := &slog.HandlerOptions{
        Level: slog.LevelDebug,
    }
    
    logger := slog.New(slog.NewJSONHandler(os.Stdout, opts))
    
    logger.Debug("Debug information", "module", "auth")
    logger.Info("Processing request", 
        "method", "POST", 
        "path", "/api/users",
        "duration", 45*time.Millisecond)
    
    logger.Warn("Rate limit approaching", 
        "current", 95, 
        "limit", 100,
        "user", "bob")
    
    logger.Error("Database connection failed", 
        "error", "connection timeout",
        "retry_count", 3,
        "will_retry", true)
}

func demonstrateContextualLogging() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    
    // Create context with request ID
    ctx := context.Background()
    
    // Using context for request-scoped logging
    requestLogger := logger.With("request_id", "req-12345")
    
    requestLogger.InfoContext(ctx, "Request started", "endpoint", "/api/data")
    
    // Simulate processing steps
    requestLogger.InfoContext(ctx, "Validating input", "step", "validation")
    time.Sleep(50 * time.Millisecond)
    
    requestLogger.InfoContext(ctx, "Querying database", 
        "step", "database",
        "query", "SELECT * FROM users")
    time.Sleep(100 * time.Millisecond)
    
    requestLogger.InfoContext(ctx, "Request completed", 
        "status", "success",
        "response_time", 150*time.Millisecond)
}

func demonstrateGroupedAttributes() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    
    // Grouping related attributes
    logger.Info("Database operation",
        slog.Group("db",
            "host", "localhost",
            "port", 5432,
            "database", "myapp"),
        slog.Group("operation",
            "type", "SELECT",
            "table", "users",
            "duration", 25*time.Millisecond))
    
    // Nested groups
    logger.Info("HTTP request processed",
        slog.Group("request",
            "method", "GET",
            "path", "/api/users/123",
            slog.Group("headers",
                "user-agent", "Go-Client/1.0",
                "content-type", "application/json")),
        slog.Group("response",
            "status", 200,
            "size", 1024))
}

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

func (u User) LogValue() slog.Value {
    return slog.GroupValue(
        slog.Int("id", u.ID),
        slog.String("name", u.Name),
        slog.String("email", u.Email),
    )
}

func demonstrateCustomTypes() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
    
    user := User{
        ID:    123,
        Name:  "Alice Johnson",
        Email: "alice@example.com",
    }
    
    // Custom type implementing LogValuer
    logger.Info("User operation", "user", user, "action", "profile_update")
    
    // Using slog.Any for automatic handling
    logger.Info("User created", slog.Any("user_data", user))
}

func main() {
    fmt.Println("=== Go slog Package Demo ===\n")
    
    fmt.Println("--- Basic slog Usage ---")
    demonstrateBasicSlog()
    
    fmt.Println("\n--- JSON Handler ---")
    demonstrateJSONHandler()
    
    fmt.Println("\n--- Contextual Logging ---")
    demonstrateContextualLogging()
    
    fmt.Println("\n--- Grouped Attributes ---")
    demonstrateGroupedAttributes()
    
    fmt.Println("\n--- Custom Types ---")
    demonstrateCustomTypes()
}
```

The `log/slog` package provides modern structured logging capabilities with  
excellent performance, flexible handlers, and built-in support for contextual  
logging and attribute grouping. It's the recommended approach for new Go  
applications requiring structured logging.  

## Conditional logging and guards

Implementing conditional logging helps prevent expensive operations when log  
levels are disabled, improving application performance.  

```go
package main

import (
    "fmt"
    "runtime"
    "strings"
    "time"
)

type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
)

type ConditionalLogger struct {
    level LogLevel
}

func NewConditionalLogger(level LogLevel) *ConditionalLogger {
    return &ConditionalLogger{level: level}
}

func (cl *ConditionalLogger) IsDebugEnabled() bool { return cl.level <= DEBUG }
func (cl *ConditionalLogger) IsInfoEnabled() bool  { return cl.level <= INFO }
func (cl *ConditionalLogger) IsWarnEnabled() bool  { return cl.level <= WARN }
func (cl *ConditionalLogger) IsErrorEnabled() bool { return cl.level <= ERROR }

func (cl *ConditionalLogger) log(level LogLevel, message string) {
    if level < cl.level {
        return
    }
    
    levelStr := []string{"DEBUG", "INFO", "WARN", "ERROR"}[level]
    timestamp := time.Now().Format("15:04:05.000")
    
    fmt.Printf("[%s] %s: %s\n", timestamp, levelStr, message)
}

func (cl *ConditionalLogger) Debug(message string) { cl.log(DEBUG, message) }
func (cl *ConditionalLogger) Info(message string)  { cl.log(INFO, message) }
func (cl *ConditionalLogger) Warn(message string)  { cl.log(WARN, message) }
func (cl *ConditionalLogger) Error(message string) { cl.log(ERROR, message) }

// Expensive debug function that should only run when debug is enabled
func expensiveDebugInfo() string {
    fmt.Println("Computing expensive debug information...")
    
    // Simulate expensive computation
    var result strings.Builder
    for i := 0; i < 1000; i++ {
        result.WriteString(fmt.Sprintf("Item %d ", i))
    }
    
    return result.String()
}

func getCaller() string {
    _, file, line, ok := runtime.Caller(2)
    if !ok {
        return "unknown"
    }
    
    // Extract just the filename
    parts := strings.Split(file, "/")
    filename := parts[len(parts)-1]
    
    return fmt.Sprintf("%s:%d", filename, line)
}

func businessLogic(logger *ConditionalLogger) {
    logger.Info("Starting business logic")
    
    // Inefficient - always computes expensive debug info
    logger.Debug("Processing items: " + expensiveDebugInfo())
    
    // Efficient - only computes when debug is enabled
    if logger.IsDebugEnabled() {
        logger.Debug("Processing items: " + expensiveDebugInfo())
    }
    
    // Complex debug information with caller context
    if logger.IsDebugEnabled() {
        caller := getCaller()
        logger.Debug(fmt.Sprintf("Called from %s with detailed state information", caller))
    }
    
    logger.Info("Processing 100 items")
    for i := 0; i < 100; i++ {
        // Conditional logging to avoid string formatting
        if logger.IsDebugEnabled() {
            logger.Debug(fmt.Sprintf("Processing item %d with data: %s", i, generateItemData(i)))
        }
        
        if i%10 == 0 && logger.IsInfoEnabled() {
            logger.Info(fmt.Sprintf("Progress: %d%%", i))
        }
        
        // Simulate work
        time.Sleep(10 * time.Millisecond)
    }
    
    logger.Info("Business logic completed")
}

func generateItemData(i int) string {
    // Simulate expensive data generation that should only happen for debug
    return strings.Repeat(fmt.Sprintf("data-%d-", i), 10)
}

func demonstratePerformanceImpact() {
    fmt.Println("=== Performance Impact Demo ===")
    
    // Test with DEBUG level (expensive operations run)
    debugLogger := NewConditionalLogger(DEBUG)
    fmt.Println("\nWith DEBUG level (expensive operations enabled):")
    start := time.Now()
    businessLogic(debugLogger)
    debugDuration := time.Since(start)
    
    // Test with INFO level (expensive operations skipped)
    infoLogger := NewConditionalLogger(INFO)
    fmt.Println("\nWith INFO level (expensive operations skipped):")
    start = time.Now()
    businessLogic(infoLogger)
    infoDuration := time.Since(start)
    
    fmt.Printf("\nPerformance comparison:\n")
    fmt.Printf("DEBUG level duration: %v\n", debugDuration)
    fmt.Printf("INFO level duration: %v\n", infoDuration)
    fmt.Printf("Performance improvement: %.2fx faster\n", 
        float64(debugDuration)/float64(infoDuration))
}

func main() {
    logger := NewConditionalLogger(INFO)
    
    fmt.Println("=== Conditional Logging Demo ===")
    fmt.Printf("Logger levels enabled - Debug: %t, Info: %t, Warn: %t, Error: %t\n",
        logger.IsDebugEnabled(), logger.IsInfoEnabled(), 
        logger.IsWarnEnabled(), logger.IsErrorEnabled())
    
    // Demonstrate conditional guard usage
    logger.Info("Application starting")
    
    if logger.IsDebugEnabled() {
        logger.Debug("Debug mode enabled - detailed logging active")
    }
    
    logger.Warn("This is a warning message")
    logger.Error("This is an error message")
    
    demonstratePerformanceImpact()
}
```

Conditional logging guards prevent expensive operations from running when the  
corresponding log level is disabled. This pattern is crucial for maintaining  
performance in production while enabling detailed debugging in development.  

## Error integration with logging

Combining error handling with logging provides comprehensive application  
monitoring and debugging capabilities.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "runtime"
    "time"
)

type ErrorLogger struct {
    logger *log.Logger
}

func NewErrorLogger() *ErrorLogger {
    return &ErrorLogger{
        logger: log.New(os.Stdout, "", log.Ldate|log.Ltime|log.Lmicroseconds),
    }
}

func (el *ErrorLogger) LogError(err error, context string) {
    if err == nil {
        return
    }
    
    // Get caller information
    _, file, line, ok := runtime.Caller(1)
    var caller string
    if ok {
        caller = fmt.Sprintf("%s:%d", file, line)
    }
    
    el.logger.Printf("[ERROR] %s: %v (at %s)", context, err, caller)
}

func (el *ErrorLogger) LogErrorf(err error, format string, args ...interface{}) {
    if err == nil {
        return
    }
    
    context := fmt.Sprintf(format, args...)
    el.LogError(err, context)
}

func (el *ErrorLogger) WrapError(err error, context string) error {
    if err == nil {
        return nil
    }
    
    el.LogError(err, context)
    return fmt.Errorf("%s: %w", context, err)
}

// Custom error types with additional context
type DatabaseError struct {
    Operation string
    Table     string
    Err       error
    Timestamp time.Time
}

func (de *DatabaseError) Error() string {
    return fmt.Sprintf("database %s error on table %s: %v", de.Operation, de.Table, de.Err)
}

func (de *DatabaseError) Unwrap() error {
    return de.Err
}

type ValidationError struct {
    Field   string
    Value   interface{}
    Rule    string
    Message string
}

func (ve *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field '%s' with value '%v': %s", 
        ve.Field, ve.Value, ve.Message)
}

func (el *ErrorLogger) LogStructuredError(err error) {
    if err == nil {
        return
    }
    
    switch e := err.(type) {
    case *DatabaseError:
        el.logger.Printf("[ERROR] Database operation failed - Operation: %s, Table: %s, Time: %v, Error: %v",
            e.Operation, e.Table, e.Timestamp, e.Err)
    case *ValidationError:
        el.logger.Printf("[ERROR] Validation failed - Field: %s, Value: %v, Rule: %s, Message: %s",
            e.Field, e.Value, e.Rule, e.Message)
    default:
        el.logger.Printf("[ERROR] %v", err)
    }
}

func simulateDatabase(operation, table string) error {
    // Simulate random database errors
    time.Sleep(50 * time.Millisecond)
    
    if operation == "delete" && table == "users" {
        return &DatabaseError{
            Operation: operation,
            Table:     table,
            Err:       fmt.Errorf("foreign key constraint violation"),
            Timestamp: time.Now(),
        }
    }
    
    return nil
}

func validateUser(name string, age int) error {
    if name == "" {
        return &ValidationError{
            Field:   "name",
            Value:   name,
            Rule:    "required",
            Message: "name cannot be empty",
        }
    }
    
    if age < 0 || age > 150 {
        return &ValidationError{
            Field:   "age",
            Value:   age,
            Rule:    "range(0-150)",
            Message: "age must be between 0 and 150",
        }
    }
    
    return nil
}

func processUser(logger *ErrorLogger, name string, age int) error {
    logger.logger.Printf("[INFO] Processing user: %s, age: %d", name, age)
    
    // Validate input
    if err := validateUser(name, age); err != nil {
        logger.LogStructuredError(err)
        return logger.WrapError(err, "user validation failed")
    }
    
    // Simulate database operations
    if err := simulateDatabase("insert", "users"); err != nil {
        logger.LogStructuredError(err)
        return logger.WrapError(err, "failed to insert user")
    }
    
    // Simulate potential delete operation
    if name == "test_delete" {
        if err := simulateDatabase("delete", "users"); err != nil {
            logger.LogStructuredError(err)
            return logger.WrapError(err, "failed to delete user")
        }
    }
    
    logger.logger.Printf("[INFO] User processed successfully: %s", name)
    return nil
}

func main() {
    logger := NewErrorLogger()
    
    fmt.Println("=== Error Integration with Logging Demo ===")
    
    // Test cases with various error scenarios
    testCases := []struct {
        name string
        age  int
        desc string
    }{
        {"Alice", 25, "Valid user"},
        {"", 30, "Empty name validation error"},
        {"Bob", -5, "Invalid age validation error"},
        {"Charlie", 200, "Age out of range validation error"},
        {"test_delete", 35, "Database constraint error"},
        {"Diana", 28, "Valid user"},
    }
    
    for _, tc := range testCases {
        fmt.Printf("\n--- Test: %s ---\n", tc.desc)
        
        if err := processUser(logger, tc.name, tc.age); err != nil {
            logger.logger.Printf("[ERROR] Final error: %v", err)
        }
    }
    
    // Demonstrate error wrapping and unwrapping
    fmt.Println("\n=== Error Wrapping Example ===")
    
    originalErr := fmt.Errorf("connection timeout")
    wrappedErr := logger.WrapError(originalErr, "database connection failed")
    doubleWrappedErr := logger.WrapError(wrappedErr, "user service unavailable")
    
    logger.logger.Printf("[INFO] Original error: %v", originalErr)
    logger.logger.Printf("[INFO] Final wrapped error: %v", doubleWrappedErr)
}
```

Error integration with logging creates comprehensive error tracking with context  
preservation. This approach enables effective debugging by capturing both the  
error details and the application state when errors occur.  

## Sampling and rate limiting

High-volume applications need log sampling and rate limiting to prevent log  
floods while maintaining observability.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type SamplingLogger struct {
    sampleRate   float64 // 0.0 to 1.0
    rateLimiter  *RateLimiter
    droppedCount int64
    mu           sync.Mutex
}

type RateLimiter struct {
    maxMessages int
    interval    time.Duration
    messageCount int
    lastReset   time.Time
    mu          sync.Mutex
}

func NewRateLimiter(maxMessages int, interval time.Duration) *RateLimiter {
    return &RateLimiter{
        maxMessages: maxMessages,
        interval:    interval,
        lastReset:   time.Now(),
    }
}

func (rl *RateLimiter) Allow() bool {
    rl.mu.Lock()
    defer rl.mu.Unlock()
    
    now := time.Now()
    if now.Sub(rl.lastReset) >= rl.interval {
        rl.messageCount = 0
        rl.lastReset = now
    }
    
    if rl.messageCount < rl.maxMessages {
        rl.messageCount++
        return true
    }
    
    return false
}

func NewSamplingLogger(sampleRate float64, maxMessages int, interval time.Duration) *SamplingLogger {
    return &SamplingLogger{
        sampleRate:  sampleRate,
        rateLimiter: NewRateLimiter(maxMessages, interval),
    }
}

func (sl *SamplingLogger) shouldLog() bool {
    // Check sampling first
    if rand.Float64() > sl.sampleRate {
        sl.mu.Lock()
        sl.droppedCount++
        sl.mu.Unlock()
        return false
    }
    
    // Check rate limiting
    if !sl.rateLimiter.Allow() {
        sl.mu.Lock()
        sl.droppedCount++
        sl.mu.Unlock()
        return false
    }
    
    return true
}

func (sl *SamplingLogger) Log(level, message string) {
    if !sl.shouldLog() {
        return
    }
    
    timestamp := time.Now().Format("15:04:05.000")
    fmt.Printf("[%s] %s: %s\n", timestamp, level, message)
}

func (sl *SamplingLogger) GetDroppedCount() int64 {
    sl.mu.Lock()
    defer sl.mu.Unlock()
    return sl.droppedCount
}

func (sl *SamplingLogger) ResetDroppedCount() {
    sl.mu.Lock()
    sl.droppedCount = 0
    sl.mu.Unlock()
}

// Adaptive sampling that adjusts based on log volume
type AdaptiveSamplingLogger struct {
    baseSampleRate    float64
    currentSampleRate float64
    messageCount      int64
    lastAdjustment    time.Time
    adjustmentInterval time.Duration
    mu                sync.Mutex
}

func NewAdaptiveSamplingLogger(baseSampleRate float64) *AdaptiveSamplingLogger {
    return &AdaptiveSamplingLogger{
        baseSampleRate:     baseSampleRate,
        currentSampleRate:  baseSampleRate,
        lastAdjustment:     time.Now(),
        adjustmentInterval: 10 * time.Second,
    }
}

func (asl *AdaptiveSamplingLogger) adjustSampleRate() {
    asl.mu.Lock()
    defer asl.mu.Unlock()
    
    now := time.Now()
    if now.Sub(asl.lastAdjustment) < asl.adjustmentInterval {
        return
    }
    
    // Calculate messages per second
    duration := now.Sub(asl.lastAdjustment).Seconds()
    messagesPerSecond := float64(asl.messageCount) / duration
    
    // Adjust sample rate based on volume
    if messagesPerSecond > 1000 {
        // High volume - reduce sample rate
        asl.currentSampleRate = asl.baseSampleRate * 0.1
    } else if messagesPerSecond > 100 {
        // Medium volume - moderate reduction
        asl.currentSampleRate = asl.baseSampleRate * 0.5
    } else {
        // Low volume - use base rate
        asl.currentSampleRate = asl.baseSampleRate
    }
    
    fmt.Printf("[SAMPLER] Adjusted sample rate to %.3f (%.0f msg/s)\n", 
        asl.currentSampleRate, messagesPerSecond)
    
    asl.messageCount = 0
    asl.lastAdjustment = now
}

func (asl *AdaptiveSamplingLogger) Log(level, message string) {
    asl.mu.Lock()
    asl.messageCount++
    currentRate := asl.currentSampleRate
    asl.mu.Unlock()
    
    asl.adjustSampleRate()
    
    if rand.Float64() <= currentRate {
        timestamp := time.Now().Format("15:04:05.000")
        fmt.Printf("[%s] %s: %s\n", timestamp, level, message)
    }
}

func simulateHighVolumeLogging(logger interface{}) {
    switch l := logger.(type) {
    case *SamplingLogger:
        for i := 0; i < 1000; i++ {
            l.Log("INFO", fmt.Sprintf("High volume message %d", i))
            if i%100 == 0 {
                time.Sleep(10 * time.Millisecond) // Brief pause
            }
        }
    case *AdaptiveSamplingLogger:
        for i := 0; i < 1000; i++ {
            l.Log("INFO", fmt.Sprintf("Adaptive message %d", i))
            if i%100 == 0 {
                time.Sleep(100 * time.Millisecond) // Brief pause
            }
        }
    }
}

func main() {
    fmt.Println("=== Sampling and Rate Limiting Demo ===")
    
    // Demo 1: Fixed sampling with rate limiting
    fmt.Println("\n--- Fixed Sampling Logger (50% sample, 10 msg/sec) ---")
    samplingLogger := NewSamplingLogger(0.5, 10, 1*time.Second)
    
    start := time.Now()
    simulateHighVolumeLogging(samplingLogger)
    duration := time.Since(start)
    
    fmt.Printf("\nStats - Duration: %v, Dropped messages: %d\n", 
        duration, samplingLogger.GetDroppedCount())
    
    // Demo 2: Adaptive sampling
    fmt.Println("\n--- Adaptive Sampling Logger (base 100% sample) ---")
    adaptiveLogger := NewAdaptiveSamplingLogger(1.0)
    
    // Simulate burst of messages
    fmt.Println("Simulating message burst...")
    start = time.Now()
    simulateHighVolumeLogging(adaptiveLogger)
    duration = time.Since(start)
    
    fmt.Printf("\nAdaptive logging completed in: %v\n", duration)
    
    // Demo 3: Different sampling rates
    fmt.Println("\n--- Sampling Rate Comparison ---")
    rates := []float64{1.0, 0.5, 0.1, 0.01}
    
    for _, rate := range rates {
        fmt.Printf("\nSample rate %.1f%%:\n", rate*100)
        logger := NewSamplingLogger(rate, 1000, 1*time.Second)
        
        messageCount := 100
        for i := 0; i < messageCount; i++ {
            logger.Log("INFO", fmt.Sprintf("Test message %d", i))
        }
        
        fmt.Printf("Dropped: %d/%d messages\n", logger.GetDroppedCount(), messageCount)
    }
}
```

Sampling and rate limiting prevent log floods in high-volume systems while  
maintaining observability. Adaptive sampling adjusts automatically based on  
message volume, while fixed sampling provides predictable behavior.  

## HTTP request logging middleware

Web applications require comprehensive request logging for monitoring,  
debugging, and security analysis.  

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "os"
    "strings"
    "time"
)

type ResponseWriter struct {
    http.ResponseWriter
    statusCode int
    size       int
}

func (rw *ResponseWriter) WriteHeader(statusCode int) {
    rw.statusCode = statusCode
    rw.ResponseWriter.WriteHeader(statusCode)
}

func (rw *ResponseWriter) Write(data []byte) (int, error) {
    size, err := rw.ResponseWriter.Write(data)
    rw.size += size
    return size, err
}

func LoggingMiddleware(logger *log.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Wrap response writer to capture status and size
            wrapped := &ResponseWriter{
                ResponseWriter: w,
                statusCode:     200, // Default status
            }
            
            // Log request start
            logger.Printf("Started %s %s from %s", r.Method, r.URL.Path, r.RemoteAddr)
            
            // Process request
            next.ServeHTTP(wrapped, r)
            
            // Log request completion
            duration := time.Since(start)
            logger.Printf("Completed %s %s - %d (%d bytes) in %v", 
                r.Method, r.URL.Path, wrapped.statusCode, wrapped.size, duration)
            
            // Log slow requests
            if duration > 1*time.Second {
                logger.Printf("SLOW REQUEST: %s %s took %v", r.Method, r.URL.Path, duration)
            }
        })
    }
}

func DetailedLoggingMiddleware(logger *log.Logger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Log detailed request information
            var requestInfo strings.Builder
            requestInfo.WriteString(fmt.Sprintf("Request: %s %s", r.Method, r.URL.String()))
            requestInfo.WriteString(fmt.Sprintf(" | Remote: %s", r.RemoteAddr))
            requestInfo.WriteString(fmt.Sprintf(" | User-Agent: %s", r.UserAgent()))
            
            if r.Referer() != "" {
                requestInfo.WriteString(fmt.Sprintf(" | Referer: %s", r.Referer()))
            }
            
            // Log request headers (selective)
            importantHeaders := []string{"Authorization", "Content-Type", "Accept"}
            for _, header := range importantHeaders {
                if value := r.Header.Get(header); value != "" {
                    // Mask sensitive headers
                    if header == "Authorization" {
                        value = maskSensitiveValue(value)
                    }
                    requestInfo.WriteString(fmt.Sprintf(" | %s: %s", header, value))
                }
            }
            
            logger.Println(requestInfo.String())
            
            wrapped := &ResponseWriter{ResponseWriter: w, statusCode: 200}
            
            next.ServeHTTP(wrapped, r)
            
            // Log response details
            duration := time.Since(start)
            responseInfo := fmt.Sprintf("Response: %d (%d bytes) in %v", 
                wrapped.statusCode, wrapped.size, duration)
            
            // Add performance warning for slow requests
            if duration > 500*time.Millisecond {
                responseInfo += " [SLOW]"
            }
            
            // Add error indication for 4xx/5xx responses
            if wrapped.statusCode >= 400 {
                responseInfo += " [ERROR]"
            }
            
            logger.Println(responseInfo)
        })
    }
}

func maskSensitiveValue(value string) string {
    if len(value) <= 8 {
        return "***"
    }
    return value[:4] + "***" + value[len(value)-4:]
}

// Sample HTTP handlers
func homeHandler(w http.ResponseWriter, r *http.Request) {
    time.Sleep(100 * time.Millisecond) // Simulate processing
    fmt.Fprintf(w, "Welcome to the home page!")
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    time.Sleep(1200 * time.Millisecond) // Simulate slow operation
    fmt.Fprintf(w, "This was a slow operation")
}

func errorHandler(w http.ResponseWriter, r *http.Request) {
    time.Sleep(50 * time.Millisecond)
    http.Error(w, "Something went wrong", http.StatusInternalServerError)
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    time.Sleep(200 * time.Millisecond)
    w.Header().Set("Content-Type", "application/json")
    fmt.Fprintf(w, `{"message": "Hello there", "timestamp": "%s"}`, time.Now().Format(time.RFC3339))
}

func simulateRequests() {
    client := &http.Client{Timeout: 5 * time.Second}
    
    requests := []struct {
        method string
        url    string
        desc   string
    }{
        {"GET", "http://localhost:8080/", "Home page"},
        {"GET", "http://localhost:8080/api", "API endpoint"},
        {"GET", "http://localhost:8080/slow", "Slow endpoint"},
        {"GET", "http://localhost:8080/error", "Error endpoint"},
        {"POST", "http://localhost:8080/api", "API POST"},
    }
    
    for _, req := range requests {
        fmt.Printf("Making request: %s\n", req.desc)
        
        httpReq, _ := http.NewRequest(req.method, req.url, nil)
        httpReq.Header.Set("User-Agent", "Go-Client/1.0")
        httpReq.Header.Set("Authorization", "Bearer secret-token-12345")
        
        resp, err := client.Do(httpReq)
        if err != nil {
            fmt.Printf("Request failed: %v\n", err)
            continue
        }
        resp.Body.Close()
        
        time.Sleep(500 * time.Millisecond) // Brief pause between requests
    }
}

func main() {
    // Create logger for HTTP requests
    httpLogger := log.New(os.Stdout, "[HTTP] ", log.Ldate|log.Ltime|log.Lmicroseconds)
    
    // Setup routes with logging middleware
    mux := http.NewServeMux()
    mux.HandleFunc("/", homeHandler)
    mux.HandleFunc("/api", apiHandler)
    mux.HandleFunc("/slow", slowHandler)
    mux.HandleFunc("/error", errorHandler)
    
    // Apply detailed logging middleware
    handler := DetailedLoggingMiddleware(httpLogger)(mux)
    
    // Start server in background
    server := &http.Server{
        Addr:    ":8080",
        Handler: handler,
    }
    
    go func() {
        fmt.Println("Starting server on :8080...")
        if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Printf("Server error: %v", err)
        }
    }()
    
    // Give server time to start
    time.Sleep(100 * time.Millisecond)
    
    // Simulate requests
    fmt.Println("\n=== Simulating HTTP Requests ===")
    simulateRequests()
    
    fmt.Println("\nServer will continue running...")
    time.Sleep(1 * time.Second)
}
```

HTTP request logging middleware provides comprehensive request/response tracking  
with performance monitoring, error detection, and security-conscious header  
logging. This pattern is essential for web application observability.  

## Database query logging

Database interactions require detailed logging for performance optimization,  
debugging, and security auditing.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "strings"
    "time"
)

type QueryLogger struct {
    logger *log.Logger
}

func NewQueryLogger() *QueryLogger {
    return &QueryLogger{
        logger: log.New(os.Stdout, "[DB] ", log.Ldate|log.Ltime|log.Lmicroseconds),
    }
}

func (ql *QueryLogger) LogQuery(ctx context.Context, query string, args []interface{}, duration time.Duration, rowsAffected int64, err error) {
    // Clean up query for logging
    cleanQuery := strings.ReplaceAll(strings.TrimSpace(query), "\n", " ")
    cleanQuery = strings.Join(strings.Fields(cleanQuery), " ")
    
    // Build log entry
    var logEntry strings.Builder
    logEntry.WriteString(fmt.Sprintf("Query: %s", cleanQuery))
    
    // Add parameters (with sensitive data masking)
    if len(args) > 0 {
        logEntry.WriteString(" | Args: [")
        for i, arg := range args {
            if i > 0 {
                logEntry.WriteString(", ")
            }
            logEntry.WriteString(ql.formatArgument(arg))
        }
        logEntry.WriteString("]")
    }
    
    // Add execution details
    logEntry.WriteString(fmt.Sprintf(" | Duration: %v", duration))
    logEntry.WriteString(fmt.Sprintf(" | Rows: %d", rowsAffected))
    
    // Add context if available
    if requestID := ctx.Value("request_id"); requestID != nil {
        logEntry.WriteString(fmt.Sprintf(" | RequestID: %v", requestID))
    }
    
    if err != nil {
        logEntry.WriteString(fmt.Sprintf(" | Error: %v", err))
        ql.logger.Printf("ERROR %s", logEntry.String())
    } else {
        // Log performance warnings
        if duration > 1*time.Second {
            logEntry.WriteString(" | [SLOW QUERY]")
            ql.logger.Printf("WARN %s", logEntry.String())
        } else {
            ql.logger.Printf("INFO %s", logEntry.String())
        }
    }
}

func (ql *QueryLogger) formatArgument(arg interface{}) string {
    switch v := arg.(type) {
    case string:
        // Mask potential sensitive strings
        if ql.isSensitiveString(v) {
            return "***"
        }
        if len(v) > 50 {
            return fmt.Sprintf("'%s...' (len=%d)", v[:47], len(v))
        }
        return fmt.Sprintf("'%s'", v)
    case []byte:
        if len(v) > 50 {
            return fmt.Sprintf("[]byte{...} (len=%d)", len(v))
        }
        return fmt.Sprintf("[]byte{%x}", v)
    default:
        return fmt.Sprintf("%v", v)
    }
}

func (ql *QueryLogger) isSensitiveString(s string) bool {
    sensitiveKeywords := []string{"password", "token", "secret", "key", "credential"}
    lowerS := strings.ToLower(s)
    
    for _, keyword := range sensitiveKeywords {
        if strings.Contains(lowerS, keyword) {
            return true
        }
    }
    
    return false
}

// Mock database operations for demonstration
type MockDB struct {
    queryLogger *QueryLogger
}

func NewMockDB(logger *QueryLogger) *MockDB {
    return &MockDB{queryLogger: logger}
}

func (db *MockDB) Query(ctx context.Context, query string, args ...interface{}) ([]map[string]interface{}, error) {
    start := time.Now()
    
    // Simulate query execution
    var duration time.Duration
    var rowsAffected int64
    var err error
    
    // Simulate different scenarios based on query
    if strings.Contains(strings.ToLower(query), "select") {
        duration = time.Duration(50+len(query)) * time.Millisecond
        rowsAffected = 5
    } else if strings.Contains(strings.ToLower(query), "insert") {
        duration = 100 * time.Millisecond
        rowsAffected = 1
    } else if strings.Contains(strings.ToLower(query), "slow") {
        duration = 1500 * time.Millisecond // Slow query
        rowsAffected = 1
    } else if strings.Contains(strings.ToLower(query), "error") {
        duration = 200 * time.Millisecond
        err = fmt.Errorf("table does not exist")
    } else {
        duration = 75 * time.Millisecond
        rowsAffected = 3
    }
    
    // Simulate actual processing time
    time.Sleep(duration)
    
    // Log the query
    db.queryLogger.LogQuery(ctx, query, args, duration, rowsAffected, err)
    
    if err != nil {
        return nil, err
    }
    
    // Return mock results
    return []map[string]interface{}{
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"},
    }, nil
}

func simulateDatabaseOperations() {
    logger := NewQueryLogger()
    db := NewMockDB(logger)
    
    fmt.Println("=== Database Query Logging Demo ===")
    
    // Create context with request ID
    ctx := context.WithValue(context.Background(), "request_id", "req-12345")
    
    // Test different types of queries
    queries := []struct {
        query string
        args  []interface{}
        desc  string
    }{
        {
            "SELECT id, name, email FROM users WHERE active = ? AND created_at > ?",
            []interface{}{true, time.Now().AddDate(0, -1, 0)},
            "Standard SELECT query",
        },
        {
            "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
            []interface{}{"John Doe", "john@example.com", "secret_password_123"},
            "INSERT with sensitive data",
        },
        {
            "UPDATE users SET last_login = ? WHERE id = ?",
            []interface{}{time.Now(), 42},
            "UPDATE query",
        },
        {
            "SELECT * FROM users WHERE name LIKE ? ORDER BY created_at DESC LIMIT ?",
            []interface{}{"%Alice%", 10},
            "Complex SELECT with patterns",
        },
        {
            "SELECT COUNT(*) FROM very_large_table WHERE status = 'slow' AND processed = false",
            []interface{}{},
            "Slow query simulation",
        },
        {
            "SELECT * FROM non_existent_error_table",
            []interface{}{},
            "Query that causes error",
        },
        {
            `SELECT u.id, u.name, p.title, c.content 
             FROM users u 
             JOIN posts p ON u.id = p.user_id 
             JOIN comments c ON p.id = c.post_id 
             WHERE u.active = ? AND p.published = ?`,
            []interface{}{true, true},
            "Multi-line complex JOIN query",
        },
    }
    
    for i, q := range queries {
        fmt.Printf("\n--- Test %d: %s ---\n", i+1, q.desc)
        
        results, err := db.Query(ctx, q.query, q.args...)
        if err != nil {
            fmt.Printf("Query failed: %v\n", err)
        } else {
            fmt.Printf("Query succeeded, returned %d rows\n", len(results))
        }
        
        time.Sleep(100 * time.Millisecond) // Brief pause between queries
    }
}

func main() {
    simulateDatabaseOperations()
    
    fmt.Println("\n=== Query Performance Summary ===")
    fmt.Println("Check the logs above for:")
    fmt.Println("- Query execution times")
    fmt.Println("- Slow query warnings (>1s)")
    fmt.Println("- Masked sensitive parameters")
    fmt.Println("- Error details")
    fmt.Println("- Request correlation IDs")
}
```

Database query logging provides detailed insights into query performance,  
parameter values (with sensitive data masking), and execution context. This  
information is crucial for database optimization and debugging.  

## Application lifecycle logging

Comprehensive application lifecycle logging tracks startup, shutdown, and  
critical state changes throughout application execution.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)

type LifecycleLogger struct {
    logger    *log.Logger
    startTime time.Time
    events    []LifecycleEvent
    mu        sync.Mutex
}

type LifecycleEvent struct {
    Timestamp time.Time
    Phase     string
    Component string
    Event     string
    Duration  time.Duration
    Data      map[string]interface{}
}

func NewLifecycleLogger() *LifecycleLogger {
    return &LifecycleLogger{
        logger:    log.New(os.Stdout, "[LIFECYCLE] ", log.Ldate|log.Ltime|log.Lmicroseconds),
        startTime: time.Now(),
        events:    make([]LifecycleEvent, 0),
    }
}

func (ll *LifecycleLogger) LogEvent(phase, component, event string, duration time.Duration, data map[string]interface{}) {
    ll.mu.Lock()
    defer ll.mu.Unlock()
    
    lifecycleEvent := LifecycleEvent{
        Timestamp: time.Now(),
        Phase:     phase,
        Component: component,
        Event:     event,
        Duration:  duration,
        Data:      data,
    }
    
    ll.events = append(ll.events, lifecycleEvent)
    
    // Build log message
    uptime := time.Since(ll.startTime)
    message := fmt.Sprintf("[%s] %s.%s", phase, component, event)
    
    if duration > 0 {
        message += fmt.Sprintf(" (took %v)", duration)
    }
    
    message += fmt.Sprintf(" | uptime: %v", uptime)
    
    if len(data) > 0 {
        message += " | data: {"
        first := true
        for k, v := range data {
            if !first {
                message += ", "
            }
            message += fmt.Sprintf("%s: %v", k, v)
            first = false
        }
        message += "}"
    }
    
    ll.logger.Println(message)
}

func (ll *LifecycleLogger) LogStartup(component string, duration time.Duration, data map[string]interface{}) {
    ll.LogEvent("STARTUP", component, "initialized", duration, data)
}

func (ll *LifecycleLogger) LogShutdown(component string, duration time.Duration, data map[string]interface{}) {
    ll.LogEvent("SHUTDOWN", component, "terminated", duration, data)
}

func (ll *LifecycleLogger) LogRuntime(component, event string, data map[string]interface{}) {
    ll.LogEvent("RUNTIME", component, event, 0, data)
}

func (ll *LifecycleLogger) GetSummary() {
    ll.mu.Lock()
    defer ll.mu.Unlock()
    
    totalUptime := time.Since(ll.startTime)
    
    ll.logger.Printf("=== Application Lifecycle Summary ===")
    ll.logger.Printf("Total uptime: %v", totalUptime)
    ll.logger.Printf("Total events: %d", len(ll.events))
    
    // Group events by phase
    phaseStats := make(map[string]int)
    componentStats := make(map[string]int)
    
    for _, event := range ll.events {
        phaseStats[event.Phase]++
        componentStats[event.Component]++
    }
    
    ll.logger.Printf("Events by phase:")
    for phase, count := range phaseStats {
        ll.logger.Printf("  %s: %d", phase, count)
    }
    
    ll.logger.Printf("Events by component:")
    for component, count := range componentStats {
        ll.logger.Printf("  %s: %d", component, count)
    }
}

// Application components
type DatabaseComponent struct {
    name   string
    logger *LifecycleLogger
}

func NewDatabaseComponent(name string, logger *LifecycleLogger) *DatabaseComponent {
    return &DatabaseComponent{name: name, logger: logger}
}

func (db *DatabaseComponent) Start() error {
    start := time.Now()
    
    // Simulate database initialization
    db.logger.LogEvent("STARTUP", db.name, "connecting", 0, map[string]interface{}{
        "host": "localhost",
        "port": 5432,
    })
    
    time.Sleep(200 * time.Millisecond) // Simulate connection time
    
    duration := time.Since(start)
    db.logger.LogStartup(db.name, duration, map[string]interface{}{
        "status":      "connected",
        "connections": 10,
    })
    
    return nil
}

func (db *DatabaseComponent) Stop() error {
    start := time.Now()
    
    db.logger.LogEvent("SHUTDOWN", db.name, "disconnecting", 0, nil)
    
    time.Sleep(100 * time.Millisecond) // Simulate cleanup time
    
    duration := time.Since(start)
    db.logger.LogShutdown(db.name, duration, map[string]interface{}{
        "status": "disconnected",
    })
    
    return nil
}

type HTTPServerComponent struct {
    name   string
    logger *LifecycleLogger
}

func NewHTTPServerComponent(name string, logger *LifecycleLogger) *HTTPServerComponent {
    return &HTTPServerComponent{name: name, logger: logger}
}

func (hs *HTTPServerComponent) Start() error {
    start := time.Now()
    
    hs.logger.LogEvent("STARTUP", hs.name, "binding", 0, map[string]interface{}{
        "port": 8080,
    })
    
    time.Sleep(150 * time.Millisecond) // Simulate server startup
    
    duration := time.Since(start)
    hs.logger.LogStartup(hs.name, duration, map[string]interface{}{
        "status": "listening",
        "addr":   ":8080",
    })
    
    return nil
}

func (hs *HTTPServerComponent) Stop() error {
    start := time.Now()
    
    hs.logger.LogEvent("SHUTDOWN", hs.name, "stopping", 0, nil)
    
    time.Sleep(300 * time.Millisecond) // Simulate graceful shutdown
    
    duration := time.Since(start)
    hs.logger.LogShutdown(hs.name, duration, map[string]interface{}{
        "status": "stopped",
    })
    
    return nil
}

type CacheComponent struct {
    name   string
    logger *LifecycleLogger
}

func NewCacheComponent(name string, logger *LifecycleLogger) *CacheComponent {
    return &CacheComponent{name: name, logger: logger}
}

func (c *CacheComponent) Start() error {
    start := time.Now()
    
    time.Sleep(50 * time.Millisecond) // Simulate cache initialization
    
    duration := time.Since(start)
    c.logger.LogStartup(c.name, duration, map[string]interface{}{
        "status":   "ready",
        "maxSize":  "100MB",
        "eviction": "LRU",
    })
    
    return nil
}

func (c *CacheComponent) Stop() error {
    start := time.Now()
    
    c.logger.LogEvent("SHUTDOWN", c.name, "flushing", 0, map[string]interface{}{
        "entries": 1500,
    })
    
    time.Sleep(80 * time.Millisecond) // Simulate cache flush
    
    duration := time.Since(start)
    c.logger.LogShutdown(c.name, duration, map[string]interface{}{
        "status": "flushed",
    })
    
    return nil
}

func main() {
    logger := NewLifecycleLogger()
    
    logger.LogEvent("STARTUP", "application", "starting", 0, map[string]interface{}{
        "version": "1.0.0",
        "env":     "development",
        "pid":     os.Getpid(),
    })
    
    // Initialize components
    components := []interface {
        Start() error
        Stop() error
    }{
        NewCacheComponent("cache", logger),
        NewDatabaseComponent("database", logger),
        NewHTTPServerComponent("http-server", logger),
    }
    
    // Start all components
    for _, component := range components {
        if err := component.Start(); err != nil {
            logger.LogEvent("STARTUP", "application", "failed", 0, map[string]interface{}{
                "error": err.Error(),
            })
            return
        }
    }
    
    logger.LogEvent("STARTUP", "application", "ready", time.Since(logger.startTime), map[string]interface{}{
        "components": len(components),
    })
    
    // Simulate runtime events
    go func() {
        for i := 0; i < 5; i++ {
            time.Sleep(1 * time.Second)
            logger.LogRuntime("application", "health-check", map[string]interface{}{
                "status":      "healthy",
                "check_count": i + 1,
            })
        }
    }()
    
    // Setup graceful shutdown
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    
    fmt.Println("Application running... Press Ctrl+C to shutdown")
    
    // Wait for shutdown signal
    <-sigChan
    
    logger.LogEvent("SHUTDOWN", "application", "signal-received", 0, map[string]interface{}{
        "signal": "SIGINT",
    })
    
    // Shutdown components in reverse order
    for i := len(components) - 1; i >= 0; i-- {
        if err := components[i].Stop(); err != nil {
            logger.LogEvent("SHUTDOWN", "application", "component-failed", 0, map[string]interface{}{
                "error": err.Error(),
            })
        }
    }
    
    shutdownDuration := time.Since(logger.startTime)
    logger.LogEvent("SHUTDOWN", "application", "completed", shutdownDuration, map[string]interface{}{
        "graceful": true,
    })
    
    // Print summary
    logger.GetSummary()
}
```

Application lifecycle logging provides complete visibility into application  
startup, runtime events, and shutdown procedures. This comprehensive tracking  
helps diagnose initialization issues and ensures proper resource cleanup.  

## Log aggregation patterns

Centralized log collection and processing patterns enable effective monitoring  
across distributed systems and multiple application instances.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "sync"
    "time"
)

type LogEntry struct {
    Timestamp string                 `json:"timestamp"`
    Level     string                 `json:"level"`
    Service   string                 `json:"service"`
    Host      string                 `json:"host"`
    Message   string                 `json:"message"`
    Fields    map[string]interface{} `json:"fields,omitempty"`
    TraceID   string                 `json:"trace_id,omitempty"`
}

type LogAggregator struct {
    entries []LogEntry
    mu      sync.RWMutex
    stats   AggregationStats
}

type AggregationStats struct {
    TotalEntries   int
    EntriesByLevel map[string]int
    EntriesByHost  map[string]int
    StartTime      time.Time
}

func NewLogAggregator() *LogAggregator {
    return &LogAggregator{
        entries: make([]LogEntry, 0),
        stats: AggregationStats{
            EntriesByLevel: make(map[string]int),
            EntriesByHost:  make(map[string]int),
            StartTime:      time.Now(),
        },
    }
}

func (la *LogAggregator) AddEntry(entry LogEntry) {
    la.mu.Lock()
    defer la.mu.Unlock()
    
    la.entries = append(la.entries, entry)
    la.stats.TotalEntries++
    la.stats.EntriesByLevel[entry.Level]++
    la.stats.EntriesByHost[entry.Host]++
}

func (la *LogAggregator) GetEntries() []LogEntry {
    la.mu.RLock()
    defer la.mu.RUnlock()
    
    // Return a copy to avoid concurrent access issues
    entries := make([]LogEntry, len(la.entries))
    copy(entries, la.entries)
    return entries
}

func (la *LogAggregator) FilterByLevel(level string) []LogEntry {
    la.mu.RLock()
    defer la.mu.RUnlock()
    
    var filtered []LogEntry
    for _, entry := range la.entries {
        if entry.Level == level {
            filtered = append(filtered, entry)
        }
    }
    return filtered
}

func (la *LogAggregator) FilterByService(service string) []LogEntry {
    la.mu.RLock()
    defer la.mu.RUnlock()
    
    var filtered []LogEntry
    for _, entry := range la.entries {
        if entry.Service == service {
            filtered = append(filtered, entry)
        }
    }
    return filtered
}

func (la *LogAggregator) FilterByTraceID(traceID string) []LogEntry {
    la.mu.RLock()
    defer la.mu.RUnlock()
    
    var filtered []LogEntry
    for _, entry := range la.entries {
        if entry.TraceID == traceID {
            filtered = append(filtered, entry)
        }
    }
    return filtered
}

func (la *LogAggregator) GetStats() AggregationStats {
    la.mu.RLock()
    defer la.mu.RUnlock()
    return la.stats
}

func (la *LogAggregator) PrintSummary() {
    stats := la.GetStats()
    
    fmt.Println("=== Log Aggregation Summary ===")
    fmt.Printf("Total entries: %d\n", stats.TotalEntries)
    fmt.Printf("Collection time: %v\n", time.Since(stats.StartTime))
    
    fmt.Println("\nEntries by level:")
    for level, count := range stats.EntriesByLevel {
        fmt.Printf("  %s: %d\n", level, count)
    }
    
    fmt.Println("\nEntries by host:")
    for host, count := range stats.EntriesByHost {
        fmt.Printf("  %s: %d\n", host, count)
    }
}

func (la *LogAggregator) ExportJSON() (string, error) {
    entries := la.GetEntries()
    
    jsonData, err := json.MarshalIndent(entries, "", "  ")
    if err != nil {
        return "", err
    }
    
    return string(jsonData), nil
}

// Simulate multiple services generating logs
func simulateService(aggregator *LogAggregator, serviceName, hostName string, wg *sync.WaitGroup) {
    defer wg.Done()
    
    traceID := fmt.Sprintf("trace-%s-%d", serviceName, time.Now().Unix())
    
    logs := []struct {
        level   string
        message string
        fields  map[string]interface{}
    }{
        {"INFO", "Service starting", map[string]interface{}{"version": "1.0.0"}},
        {"DEBUG", "Loading configuration", map[string]interface{}{"config_file": "/etc/app.conf"}},
        {"INFO", "Database connection established", map[string]interface{}{"db_host": "db.example.com"}},
        {"WARN", "High memory usage detected", map[string]interface{}{"memory_percent": 85}},
        {"ERROR", "Failed to process request", map[string]interface{}{"error": "timeout", "request_id": "req-123"}},
        {"INFO", "Service shutting down", map[string]interface{}{"graceful": true}},
    }
    
    for i, logData := range logs {
        entry := LogEntry{
            Timestamp: time.Now().Format(time.RFC3339),
            Level:     logData.level,
            Service:   serviceName,
            Host:      hostName,
            Message:   logData.message,
            Fields:    logData.fields,
            TraceID:   traceID,
        }
        
        aggregator.AddEntry(entry)
        
        // Simulate processing time
        time.Sleep(time.Duration(100+i*50) * time.Millisecond)
    }
}

func demonstrateFiltering(aggregator *LogAggregator) {
    fmt.Println("\n=== Filtering Examples ===")
    
    // Filter by level
    errorEntries := aggregator.FilterByLevel("ERROR")
    fmt.Printf("Error entries: %d\n", len(errorEntries))
    for _, entry := range errorEntries {
        fmt.Printf("  [%s] %s: %s\n", entry.Service, entry.Level, entry.Message)
    }
    
    // Filter by service
    userServiceEntries := aggregator.FilterByService("user-service")
    fmt.Printf("\nUser service entries: %d\n", len(userServiceEntries))
    for _, entry := range userServiceEntries {
        fmt.Printf("  [%s] %s: %s\n", entry.Timestamp, entry.Level, entry.Message)
    }
    
    // Show trace correlation
    if len(aggregator.GetEntries()) > 0 {
        firstTraceID := aggregator.GetEntries()[0].TraceID
        traceEntries := aggregator.FilterByTraceID(firstTraceID)
        fmt.Printf("\nTrace %s entries: %d\n", firstTraceID, len(traceEntries))
        for _, entry := range traceEntries {
            fmt.Printf("  [%s] %s: %s\n", entry.Timestamp, entry.Level, entry.Message)
        }
    }
}

func main() {
    aggregator := NewLogAggregator()
    
    fmt.Println("=== Log Aggregation Demo ===")
    fmt.Println("Simulating multiple services...")
    
    // Simulate multiple services on different hosts
    var wg sync.WaitGroup
    
    services := []struct {
        name string
        host string
    }{
        {"user-service", "web-01"},
        {"auth-service", "web-02"},
        {"payment-service", "web-03"},
        {"notification-service", "web-01"},
    }
    
    for _, service := range services {
        wg.Add(1)
        go simulateService(aggregator, service.name, service.host, &wg)
    }
    
    // Wait for all services to complete
    wg.Wait()
    
    // Print aggregation summary
    aggregator.PrintSummary()
    
    // Demonstrate filtering capabilities
    demonstrateFiltering(aggregator)
    
    // Export to JSON
    fmt.Println("\n=== JSON Export Sample ===")
    jsonData, err := aggregator.ExportJSON()
    if err != nil {
        fmt.Printf("Export error: %v\n", err)
        return
    }
    
    // Show first few entries of JSON export
    entries := aggregator.GetEntries()
    if len(entries) > 0 {
        firstEntry, _ := json.MarshalIndent(entries[0], "", "  ")
        fmt.Printf("Sample JSON entry:\n%s\n", string(firstEntry))
    }
    
    fmt.Printf("\nTotal JSON size: %d bytes\n", len(jsonData))
    fmt.Println("Complete JSON export available for analysis")
}
```

Log aggregation patterns enable centralized collection, filtering, and analysis  
of logs from distributed services. This approach provides comprehensive system  
visibility and correlation capabilities essential for modern applications.  

## Logging best practices summary

This comprehensive example demonstrates all the key logging best practices  
integrated into a cohesive logging strategy.  

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "os"
    "runtime"
    "strings"
    "sync"
    "time"
)

// Best practices logger that combines multiple techniques
type BestPracticesLogger struct {
    level          LogLevel
    output         *os.File
    enableSampling bool
    sampleRate     float64
    enableColors   bool
    droppedCount   int64
    mu             sync.RWMutex
}

type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
    FATAL
)

func (l LogLevel) String() string {
    return []string{"DEBUG", "INFO", "WARN", "ERROR", "FATAL"}[l]
}

type LogMessage struct {
    Timestamp string                 `json:"timestamp"`
    Level     string                 `json:"level"`
    Message   string                 `json:"message"`
    Fields    map[string]interface{} `json:"fields,omitempty"`
    Caller    string                 `json:"caller,omitempty"`
    TraceID   string                 `json:"trace_id,omitempty"`
}

func NewBestPracticesLogger(level LogLevel, outputFile string) (*BestPracticesLogger, error) {
    var output *os.File
    var err error
    
    if outputFile == "" {
        output = os.Stdout
    } else {
        output, err = os.OpenFile(outputFile, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        if err != nil {
            return nil, fmt.Errorf("failed to open log file: %v", err)
        }
    }
    
    return &BestPracticesLogger{
        level:          level,
        output:         output,
        enableSampling: false,
        sampleRate:     1.0,
        enableColors:   output == os.Stdout,
    }, nil
}

func (bpl *BestPracticesLogger) WithSampling(rate float64) *BestPracticesLogger {
    bpl.enableSampling = true
    bpl.sampleRate = rate
    return bpl
}

func (bpl *BestPracticesLogger) shouldLog(level LogLevel) bool {
    // Check log level first
    if level < bpl.level {
        return false
    }
    
    // Check sampling if enabled
    if bpl.enableSampling && level < ERROR {
        // Never sample ERROR and FATAL messages
        if rand.Float64() > bpl.sampleRate {
            bpl.mu.Lock()
            bpl.droppedCount++
            bpl.mu.Unlock()
            return false
        }
    }
    
    return true
}

func (bpl *BestPracticesLogger) getCaller() string {
    _, file, line, ok := runtime.Caller(3)
    if !ok {
        return "unknown"
    }
    
    // Extract just the filename for brevity
    parts := strings.Split(file, "/")
    filename := parts[len(parts)-1]
    
    return fmt.Sprintf("%s:%d", filename, line)
}

func (bpl *BestPracticesLogger) log(level LogLevel, message string, fields map[string]interface{}) {
    if !bpl.shouldLog(level) {
        return
    }
    
    // Create log message
    logMsg := LogMessage{
        Timestamp: time.Now().UTC().Format(time.RFC3339),
        Level:     level.String(),
        Message:   message,
        Fields:    fields,
        Caller:    bpl.getCaller(),
    }
    
    // Add trace ID from context if available
    if fields != nil {
        if traceID, ok := fields["trace_id"]; ok {
            logMsg.TraceID = fmt.Sprintf("%v", traceID)
        }
    }
    
    // Format output based on destination
    var output string
    if bpl.output == os.Stdout {
        // Human-readable format for console
        output = bpl.formatConsole(logMsg)
    } else {
        // JSON format for files
        jsonData, _ := json.Marshal(logMsg)
        output = string(jsonData)
    }
    
    // Write to output
    bpl.mu.Lock()
    fmt.Fprintln(bpl.output, output)
    bpl.mu.Unlock()
}

func (bpl *BestPracticesLogger) formatConsole(msg LogMessage) string {
    var result strings.Builder
    
    // Timestamp
    timestamp := msg.Timestamp
    if len(timestamp) > 19 {
        timestamp = timestamp[:19] // Trim to YYYY-MM-DDTHH:MM:SS
    }
    
    // Level with color if enabled
    levelStr := msg.Level
    if bpl.enableColors {
        switch msg.Level {
        case "DEBUG":
            levelStr = fmt.Sprintf("\033[36m%s\033[0m", levelStr)
        case "INFO":
            levelStr = fmt.Sprintf("\033[32m%s\033[0m", levelStr)
        case "WARN":
            levelStr = fmt.Sprintf("\033[33m%s\033[0m", levelStr)
        case "ERROR", "FATAL":
            levelStr = fmt.Sprintf("\033[31m%s\033[0m", levelStr)
        }
    }
    
    result.WriteString(fmt.Sprintf("[%s] %s: %s", timestamp, levelStr, msg.Message))
    
    // Add caller information
    if msg.Caller != "" {
        result.WriteString(fmt.Sprintf(" (%s)", msg.Caller))
    }
    
    // Add fields
    if len(msg.Fields) > 0 {
        result.WriteString(" |")
        for k, v := range msg.Fields {
            result.WriteString(fmt.Sprintf(" %s=%v", k, v))
        }
    }
    
    // Add trace ID
    if msg.TraceID != "" {
        result.WriteString(fmt.Sprintf(" | trace=%s", msg.TraceID))
    }
    
    return result.String()
}

// Logging methods with best practices
func (bpl *BestPracticesLogger) Debug(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    bpl.log(DEBUG, message, f)
}

func (bpl *BestPracticesLogger) Info(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    bpl.log(INFO, message, f)
}

func (bpl *BestPracticesLogger) Warn(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    bpl.log(WARN, message, f)
}

func (bpl *BestPracticesLogger) Error(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    bpl.log(ERROR, message, f)
}

func (bpl *BestPracticesLogger) Fatal(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    bpl.log(FATAL, message, f)
    os.Exit(1)
}

func (bpl *BestPracticesLogger) WithContext(ctx context.Context) *BestPracticesLogger {
    // Create a copy with context-specific fields
    // In a real implementation, this would extract values from context
    return bpl
}

func (bpl *BestPracticesLogger) Close() error {
    if bpl.output != os.Stdout && bpl.output != os.Stderr {
        return bpl.output.Close()
    }
    return nil
}

func (bpl *BestPracticesLogger) GetDroppedCount() int64 {
    bpl.mu.RLock()
    defer bpl.mu.RUnlock()
    return bpl.droppedCount
}

func demonstrateBestPractices() {
    fmt.Println("=== Logging Best Practices Demo ===")
    
    // Create console logger
    consoleLogger, err := NewBestPracticesLogger(DEBUG, "")
    if err != nil {
        log.Fatalf("Failed to create console logger: %v", err)
    }
    
    // Create file logger with sampling
    fileLogger, err := NewBestPracticesLogger(INFO, "/tmp/best_practices.log")
    if err != nil {
        log.Fatalf("Failed to create file logger: %v", err)
    }
    defer fileLogger.Close()
    
    fileLogger.WithSampling(0.5) // 50% sampling rate
    
    fmt.Println("\n--- Console Logging (with colors) ---")
    
    // Demonstrate structured logging
    consoleLogger.Info("Application starting", map[string]interface{}{
        "version":    "1.0.0",
        "env":        "development",
        "trace_id":   "trace-12345",
        "start_time": time.Now(),
    })
    
    consoleLogger.Debug("Loading configuration", map[string]interface{}{
        "config_file": "/etc/app.conf",
        "trace_id":    "trace-12345",
    })
    
    consoleLogger.Warn("High memory usage", map[string]interface{}{
        "memory_percent": 85,
        "threshold":      80,
        "trace_id":       "trace-12345",
    })
    
    consoleLogger.Error("Database connection failed", map[string]interface{}{
        "error":      "connection timeout",
        "retry_count": 3,
        "trace_id":    "trace-12345",
    })
    
    fmt.Println("\n--- File Logging (JSON format with sampling) ---")
    
    // Demonstrate high-volume logging with sampling
    for i := 0; i < 20; i++ {
        fileLogger.Info(fmt.Sprintf("Processing item %d", i), map[string]interface{}{
            "item_id":  i,
            "batch":    i / 5,
            "trace_id": "trace-batch-001",
        })
    }
    
    fmt.Printf("File logger dropped %d messages due to sampling\n", fileLogger.GetDroppedCount())
    
    // Show best practices summary
    fmt.Println("\n=== Best Practices Applied ===")
    fmt.Println("✓ Structured logging with consistent fields")
    fmt.Println("✓ Appropriate log levels")
    fmt.Println("✓ Caller information for debugging")
    fmt.Println("✓ Trace ID correlation")
    fmt.Println("✓ Performance optimization with sampling")
    fmt.Println("✓ Different output formats (console vs file)")
    fmt.Println("✓ Thread-safe logging")
    fmt.Println("✓ Graceful resource management")
    fmt.Println("✓ Color coding for improved readability")
    fmt.Println("✓ JSON format for machine processing")
    
    // Read back file contents to show JSON format
    fmt.Println("\n--- File Contents (JSON format) ---")
    content, err := os.ReadFile("/tmp/best_practices.log")
    if err != nil {
        fmt.Printf("Error reading log file: %v\n", err)
        return
    }
    
    lines := strings.Split(string(content), "\n")
    for i, line := range lines {
        if strings.TrimSpace(line) != "" && i < 3 { // Show first 3 entries
            var logEntry LogMessage
            if err := json.Unmarshal([]byte(line), &logEntry); err == nil {
                fmt.Printf("Entry %d: %s - %s\n", i+1, logEntry.Level, logEntry.Message)
            }
        }
    }
    
    if len(lines) > 3 {
        fmt.Printf("... and %d more entries\n", len(lines)-3)
    }
}

func main() {
    demonstrateBestPractices()
}
```

This comprehensive example demonstrates all key logging best practices including  
structured logging, appropriate levels, performance optimization, output  
formatting, thread safety, and proper resource management. These patterns  
provide a solid foundation for production logging systems.  

## Environment-based logging configuration

Different environments require different logging configurations for optimal  
development experience and production monitoring.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "strconv"
    "strings"
)

type Environment string

const (
    Development Environment = "development"
    Staging     Environment = "staging"
    Production  Environment = "production"
)

type EnvironmentLogger struct {
    environment Environment
    logger      *log.Logger
    debugMode   bool
    verbose     bool
}

func NewEnvironmentLogger() *EnvironmentLogger {
    env := Environment(os.Getenv("GO_ENV"))
    if env == "" {
        env = Development
    }
    
    debugMode, _ := strconv.ParseBool(os.Getenv("DEBUG"))
    verbose, _ := strconv.ParseBool(os.Getenv("VERBOSE"))
    
    var logger *log.Logger
    
    switch env {
    case Development:
        // Development: detailed, colorful logs to console
        logger = log.New(os.Stdout, "[DEV] ", log.Ldate|log.Ltime|log.Lshortfile)
    case Staging:
        // Staging: JSON format for testing log aggregation
        logger = log.New(os.Stdout, "", 0)
    case Production:
        // Production: structured logs to file
        logFile := os.Getenv("LOG_FILE")
        if logFile == "" {
            logFile = "/var/log/app.log"
        }
        
        file, err := os.OpenFile(logFile, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        if err != nil {
            // Fallback to stdout if file creation fails
            logger = log.New(os.Stdout, "[PROD] ", log.Ldate|log.Ltime)
        } else {
            logger = log.New(file, "", 0)
        }
    default:
        logger = log.New(os.Stdout, "[UNKNOWN] ", log.Ldate|log.Ltime)
    }
    
    return &EnvironmentLogger{
        environment: env,
        logger:      logger,
        debugMode:   debugMode,
        verbose:     verbose,
    }
}

func (el *EnvironmentLogger) Debug(message string) {
    if !el.debugMode {
        return
    }
    
    switch el.environment {
    case Development:
        el.logger.Printf("DEBUG: %s", message)
    case Staging, Production:
        el.logger.Printf(`{"level":"debug","message":"%s","env":"%s"}`, message, el.environment)
    }
}

func (el *EnvironmentLogger) Info(message string) {
    switch el.environment {
    case Development:
        el.logger.Printf("INFO: %s", message)
    case Staging, Production:
        el.logger.Printf(`{"level":"info","message":"%s","env":"%s"}`, message, el.environment)
    }
}

func (el *EnvironmentLogger) Warn(message string) {
    switch el.environment {
    case Development:
        el.logger.Printf("WARN: %s", message)
    case Staging, Production:
        el.logger.Printf(`{"level":"warn","message":"%s","env":"%s"}`, message, el.environment)
    }
}

func (el *EnvironmentLogger) Error(message string) {
    switch el.environment {
    case Development:
        el.logger.Printf("ERROR: %s", message)
    case Staging, Production:
        el.logger.Printf(`{"level":"error","message":"%s","env":"%s"}`, message, el.environment)
    }
}

func (el *EnvironmentLogger) Verbose(message string) {
    if !el.verbose {
        return
    }
    
    switch el.environment {
    case Development:
        el.logger.Printf("VERBOSE: %s", message)
    case Staging, Production:
        el.logger.Printf(`{"level":"verbose","message":"%s","env":"%s"}`, message, el.environment)
    }
}

func simulateEnvironments() {
    envs := []struct {
        name   string
        env    string
        debug  string
        verbose string
    }{
        {"Development", "development", "true", "true"},
        {"Staging", "staging", "false", "true"},
        {"Production", "production", "false", "false"},
    }
    
    for _, config := range envs {
        fmt.Printf("\n=== %s Environment ===\n", config.name)
        
        // Set environment variables
        os.Setenv("GO_ENV", config.env)
        os.Setenv("DEBUG", config.debug)
        os.Setenv("VERBOSE", config.verbose)
        
        logger := NewEnvironmentLogger()
        
        logger.Debug("This is a debug message")
        logger.Verbose("This is a verbose message")
        logger.Info("Application started successfully")
        logger.Warn("Memory usage is at 75%")
        logger.Error("Failed to connect to external service")
    }
}

func main() {
    fmt.Println("=== Environment-based Logging Configuration ===")
    simulateEnvironments()
    
    fmt.Println("\n=== Configuration Summary ===")
    fmt.Println("Development: Detailed console logs with file info")
    fmt.Println("Staging: JSON format for testing log aggregation")
    fmt.Println("Production: Structured logs to file (fallback to stdout)")
    fmt.Println("Debug mode: Controlled by DEBUG environment variable")
    fmt.Println("Verbose mode: Controlled by VERBOSE environment variable")
}
```

Environment-based logging adapts log format and verbosity based on deployment  
environment, providing optimal configuration for development, testing, and  
production scenarios.  

## Asynchronous logging with channels

Asynchronous logging prevents I/O operations from blocking application threads,  
improving performance in high-throughput scenarios.  

```go
package main

import (
    "fmt"
    "log"
    "os"
    "sync"
    "time"
)

type AsyncLogger struct {
    logChan chan LogMessage
    done    chan struct{}
    wg      sync.WaitGroup
    logger  *log.Logger
}

type LogMessage struct {
    Level     string
    Message   string
    Timestamp time.Time
    Fields    map[string]interface{}
}

func NewAsyncLogger(bufferSize int) *AsyncLogger {
    al := &AsyncLogger{
        logChan: make(chan LogMessage, bufferSize),
        done:    make(chan struct{}),
        logger:  log.New(os.Stdout, "", 0),
    }
    
    // Start background goroutine for processing logs
    al.wg.Add(1)
    go al.processLogs()
    
    return al
}

func (al *AsyncLogger) processLogs() {
    defer al.wg.Done()
    
    for {
        select {
        case msg := <-al.logChan:
            al.writeLog(msg)
        case <-al.done:
            // Process remaining messages
            for len(al.logChan) > 0 {
                msg := <-al.logChan
                al.writeLog(msg)
            }
            return
        }
    }
}

func (al *AsyncLogger) writeLog(msg LogMessage) {
    timestamp := msg.Timestamp.Format("2006-01-02 15:04:05.000")
    
    if len(msg.Fields) > 0 {
        fieldStr := ""
        for k, v := range msg.Fields {
            fieldStr += fmt.Sprintf(" %s=%v", k, v)
        }
        al.logger.Printf("[%s] %s: %s |%s", timestamp, msg.Level, msg.Message, fieldStr)
    } else {
        al.logger.Printf("[%s] %s: %s", timestamp, msg.Level, msg.Message)
    }
}

func (al *AsyncLogger) log(level, message string, fields map[string]interface{}) {
    msg := LogMessage{
        Level:     level,
        Message:   message,
        Timestamp: time.Now(),
        Fields:    fields,
    }
    
    select {
    case al.logChan <- msg:
        // Message queued successfully
    default:
        // Channel full - could implement overflow handling
        fmt.Fprintf(os.Stderr, "WARN: Log buffer full, dropping message: %s\n", message)
    }
}

func (al *AsyncLogger) Debug(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    al.log("DEBUG", message, f)
}

func (al *AsyncLogger) Info(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    al.log("INFO", message, f)
}

func (al *AsyncLogger) Warn(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    al.log("WARN", message, f)
}

func (al *AsyncLogger) Error(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    al.log("ERROR", message, f)
}

func (al *AsyncLogger) Close() {
    close(al.done)
    al.wg.Wait()
    close(al.logChan)
}

func benchmarkSyncVsAsync() {
    const messageCount = 10000
    
    // Benchmark synchronous logging
    syncLogger := log.New(os.Stdout, "[SYNC] ", log.Ltime)
    start := time.Now()
    
    for i := 0; i < messageCount; i++ {
        syncLogger.Printf("Sync message %d", i)
    }
    
    syncDuration := time.Since(start)
    
    // Benchmark asynchronous logging
    asyncLogger := NewAsyncLogger(1000)
    defer asyncLogger.Close()
    
    start = time.Now()
    
    for i := 0; i < messageCount; i++ {
        asyncLogger.Info(fmt.Sprintf("Async message %d", i))
    }
    
    // Wait for all messages to be processed
    time.Sleep(100 * time.Millisecond)
    asyncLogger.Close()
    
    asyncDuration := time.Since(start)
    
    fmt.Printf("\nPerformance Comparison:\n")
    fmt.Printf("Synchronous logging: %v\n", syncDuration)
    fmt.Printf("Asynchronous logging: %v\n", asyncDuration)
    fmt.Printf("Speedup: %.2fx\n", float64(syncDuration)/float64(asyncDuration))
}

func main() {
    fmt.Println("=== Asynchronous Logging Demo ===")
    
    logger := NewAsyncLogger(100)
    defer logger.Close()
    
    // Demonstrate non-blocking logging
    fmt.Println("Logging messages asynchronously...")
    
    logger.Info("Application started")
    logger.Debug("Loading configuration", map[string]interface{}{
        "config_file": "/etc/app.conf",
    })
    
    // Simulate high-volume logging
    for i := 0; i < 20; i++ {
        logger.Info(fmt.Sprintf("Processing item %d", i), map[string]interface{}{
            "item_id": i,
            "batch":   i / 5,
        })
        
        if i%5 == 0 {
            logger.Warn("Checkpoint reached", map[string]interface{}{
                "checkpoint": i,
            })
        }
    }
    
    logger.Error("Simulated error occurred", map[string]interface{}{
        "error_code": "E001",
        "component":  "data_processor",
    })
    
    logger.Info("Application shutting down")
    
    // Give async logger time to process all messages
    time.Sleep(200 * time.Millisecond)
    
    fmt.Println("\nAll messages processed by background goroutine")
}
```

Asynchronous logging prevents I/O operations from blocking the main application  
thread, significantly improving performance in high-throughput scenarios while  
maintaining log message ordering.  

## Log filtering and transformation

Advanced filtering and transformation capabilities enable dynamic log processing  
based on content, source, and runtime conditions.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "time"
)

type LogFilter interface {
    ShouldLog(entry LogEntry) bool
}

type LogTransformer interface {
    Transform(entry LogEntry) LogEntry
}

type LogEntry struct {
    Timestamp time.Time
    Level     string
    Message   string
    Source    string
    Fields    map[string]interface{}
}

// Filters
type LevelFilter struct {
    allowedLevels map[string]bool
}

func NewLevelFilter(levels ...string) *LevelFilter {
    allowed := make(map[string]bool)
    for _, level := range levels {
        allowed[strings.ToUpper(level)] = true
    }
    return &LevelFilter{allowedLevels: allowed}
}

func (lf *LevelFilter) ShouldLog(entry LogEntry) bool {
    return lf.allowedLevels[strings.ToUpper(entry.Level)]
}

type SourceFilter struct {
    allowedSources map[string]bool
}

func NewSourceFilter(sources ...string) *SourceFilter {
    allowed := make(map[string]bool)
    for _, source := range sources {
        allowed[source] = true
    }
    return &SourceFilter{allowedSources: allowed}
}

func (sf *SourceFilter) ShouldLog(entry LogEntry) bool {
    return sf.allowedSources[entry.Source]
}

type RegexFilter struct {
    pattern *regexp.Regexp
    exclude bool
}

func NewRegexFilter(pattern string, exclude bool) (*RegexFilter, error) {
    regex, err := regexp.Compile(pattern)
    if err != nil {
        return nil, err
    }
    
    return &RegexFilter{
        pattern: regex,
        exclude: exclude,
    }, nil
}

func (rf *RegexFilter) ShouldLog(entry LogEntry) bool {
    matches := rf.pattern.MatchString(entry.Message)
    if rf.exclude {
        return !matches
    }
    return matches
}

type FrequencyFilter struct {
    messageCount map[string]int
    maxCount     int
    resetInterval time.Duration
    lastReset     time.Time
}

func NewFrequencyFilter(maxCount int, resetInterval time.Duration) *FrequencyFilter {
    return &FrequencyFilter{
        messageCount:  make(map[string]int),
        maxCount:      maxCount,
        resetInterval: resetInterval,
        lastReset:     time.Now(),
    }
}

func (ff *FrequencyFilter) ShouldLog(entry LogEntry) bool {
    now := time.Now()
    if now.Sub(ff.lastReset) >= ff.resetInterval {
        ff.messageCount = make(map[string]int)
        ff.lastReset = now
    }
    
    key := fmt.Sprintf("%s:%s", entry.Source, entry.Message)
    ff.messageCount[key]++
    
    return ff.messageCount[key] <= ff.maxCount
}

// Transformers
type FieldMasker struct {
    sensitiveFields []string
}

func NewFieldMasker(fields ...string) *FieldMasker {
    return &FieldMasker{sensitiveFields: fields}
}

func (fm *FieldMasker) Transform(entry LogEntry) LogEntry {
    if entry.Fields == nil {
        return entry
    }
    
    transformed := LogEntry{
        Timestamp: entry.Timestamp,
        Level:     entry.Level,
        Message:   entry.Message,
        Source:    entry.Source,
        Fields:    make(map[string]interface{}),
    }
    
    // Copy fields, masking sensitive ones
    for k, v := range entry.Fields {
        masked := false
        for _, sensitive := range fm.sensitiveFields {
            if strings.Contains(strings.ToLower(k), strings.ToLower(sensitive)) {
                transformed.Fields[k] = "***MASKED***"
                masked = true
                break
            }
        }
        if !masked {
            transformed.Fields[k] = v
        }
    }
    
    return transformed
}

type MessageEnricher struct {
    enrichments map[string]interface{}
}

func NewMessageEnricher(enrichments map[string]interface{}) *MessageEnricher {
    return &MessageEnricher{enrichments: enrichments}
}

func (me *MessageEnricher) Transform(entry LogEntry) LogEntry {
    if entry.Fields == nil {
        entry.Fields = make(map[string]interface{})
    }
    
    // Add enrichment fields
    for k, v := range me.enrichments {
        if _, exists := entry.Fields[k]; !exists {
            entry.Fields[k] = v
        }
    }
    
    return entry
}

type PrefixTransformer struct {
    prefix string
}

func NewPrefixTransformer(prefix string) *PrefixTransformer {
    return &PrefixTransformer{prefix: prefix}
}

func (pt *PrefixTransformer) Transform(entry LogEntry) LogEntry {
    entry.Message = pt.prefix + entry.Message
    return entry
}

// Filtering Logger
type FilteringLogger struct {
    filters      []LogFilter
    transformers []LogTransformer
}

func NewFilteringLogger() *FilteringLogger {
    return &FilteringLogger{
        filters:      make([]LogFilter, 0),
        transformers: make([]LogTransformer, 0),
    }
}

func (fl *FilteringLogger) AddFilter(filter LogFilter) {
    fl.filters = append(fl.filters, filter)
}

func (fl *FilteringLogger) AddTransformer(transformer LogTransformer) {
    fl.transformers = append(fl.transformers, transformer)
}

func (fl *FilteringLogger) Log(entry LogEntry) {
    // Apply filters
    for _, filter := range fl.filters {
        if !filter.ShouldLog(entry) {
            return // Message filtered out
        }
    }
    
    // Apply transformers
    for _, transformer := range fl.transformers {
        entry = transformer.Transform(entry)
    }
    
    // Output the log entry
    fl.outputLog(entry)
}

func (fl *FilteringLogger) outputLog(entry LogEntry) {
    timestamp := entry.Timestamp.Format("15:04:05.000")
    
    output := fmt.Sprintf("[%s] %s [%s]: %s", 
        timestamp, entry.Level, entry.Source, entry.Message)
    
    if len(entry.Fields) > 0 {
        output += " |"
        for k, v := range entry.Fields {
            output += fmt.Sprintf(" %s=%v", k, v)
        }
    }
    
    fmt.Println(output)
}

func demonstrateFiltering() {
    logger := NewFilteringLogger()
    
    // Add filters
    levelFilter := NewLevelFilter("INFO", "WARN", "ERROR")
    logger.AddFilter(levelFilter)
    
    sourceFilter := NewSourceFilter("auth", "api", "database")
    logger.AddFilter(sourceFilter)
    
    regexFilter, _ := NewRegexFilter("password|secret", true) // Exclude sensitive messages
    logger.AddFilter(regexFilter)
    
    frequencyFilter := NewFrequencyFilter(2, 5*time.Second) // Max 2 occurrences per 5 seconds
    logger.AddFilter(frequencyFilter)
    
    // Add transformers
    masker := NewFieldMasker("password", "token", "key")
    logger.AddTransformer(masker)
    
    enricher := NewMessageEnricher(map[string]interface{}{
        "app_version": "1.0.0",
        "environment": "production",
    })
    logger.AddTransformer(enricher)
    
    prefixer := NewPrefixTransformer("[FILTERED] ")
    logger.AddTransformer(prefixer)
    
    fmt.Println("=== Filtered and Transformed Logging ===")
    
    // Test various log entries
    testEntries := []LogEntry{
        {time.Now(), "DEBUG", "Debug message", "auth", nil}, // Filtered out by level
        {time.Now(), "INFO", "User logged in", "auth", map[string]interface{}{"user": "alice"}},
        {time.Now(), "ERROR", "Database connection failed", "database", map[string]interface{}{"error": "timeout"}},
        {time.Now(), "WARN", "Invalid password attempt", "auth", nil}, // Filtered out by regex
        {time.Now(), "INFO", "API request processed", "cache", nil}, // Filtered out by source
        {time.Now(), "ERROR", "Payment failed", "payment", map[string]interface{}{"user": "bob", "card_token": "secret123"}},
        {time.Now(), "INFO", "User logged in", "auth", map[string]interface{}{"user": "charlie"}}, // Same message
        {time.Now(), "INFO", "User logged in", "auth", map[string]interface{}{"user": "david"}}, // Same message
        {time.Now(), "INFO", "User logged in", "auth", map[string]interface{}{"user": "eve"}}, // Filtered by frequency
    }
    
    for i, entry := range testEntries {
        fmt.Printf("\nEntry %d: ", i+1)
        logger.Log(entry)
    }
    
    fmt.Println("\n=== Filter Summary ===")
    fmt.Println("✓ Level filter: Only INFO, WARN, ERROR")
    fmt.Println("✓ Source filter: Only auth, api, database")
    fmt.Println("✓ Regex filter: Exclude messages containing 'password' or 'secret'")
    fmt.Println("✓ Frequency filter: Max 2 identical messages per 5 seconds")
    fmt.Println("✓ Field masker: Mask sensitive fields")
    fmt.Println("✓ Message enricher: Add app metadata")
    fmt.Println("✓ Prefix transformer: Add [FILTERED] prefix")
}

func main() {
    demonstrateFiltering()
}
```

Log filtering and transformation provide sophisticated control over log output,  
enabling dynamic content filtering, sensitive data masking, message enrichment,  
and rate limiting to optimize log quality and security.  

## Logger factory pattern

The factory pattern enables centralized logger creation with consistent  
configuration and different logger types for various use cases.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
    "path/filepath"
    "strings"
    "time"
)

type LoggerType int

const (
    ConsoleLogger LoggerType = iota
    FileLogger
    JSONLogger
    SyslogLogger
    DebugLogger
)

type LoggerConfig struct {
    Type        LoggerType
    Level       string
    Output      string
    Format      string
    Prefix      string
    EnableColor bool
    MaxSize     int64
    Fields      map[string]interface{}
}

type Logger interface {
    Debug(msg string)
    Info(msg string)
    Warn(msg string)
    Error(msg string)
    Close() error
}

type LoggerFactory struct {
    defaultConfig LoggerConfig
}

func NewLoggerFactory() *LoggerFactory {
    return &LoggerFactory{
        defaultConfig: LoggerConfig{
            Type:        ConsoleLogger,
            Level:       "INFO",
            Format:      "text",
            EnableColor: true,
            MaxSize:     10 * 1024 * 1024, // 10MB
            Fields:      make(map[string]interface{}),
        },
    }
}

func (lf *LoggerFactory) CreateLogger(config LoggerConfig) (Logger, error) {
    // Merge with defaults
    if config.Level == "" {
        config.Level = lf.defaultConfig.Level
    }
    if config.Format == "" {
        config.Format = lf.defaultConfig.Format
    }
    
    switch config.Type {
    case ConsoleLogger:
        return lf.createConsoleLogger(config)
    case FileLogger:
        return lf.createFileLogger(config)
    case JSONLogger:
        return lf.createJSONLogger(config)
    case DebugLogger:
        return lf.createDebugLogger(config)
    default:
        return nil, fmt.Errorf("unsupported logger type: %v", config.Type)
    }
}

func (lf *LoggerFactory) createConsoleLogger(config LoggerConfig) (Logger, error) {
    return &ConsoleLoggerImpl{
        logger:      log.New(os.Stdout, config.Prefix, log.Ldate|log.Ltime),
        level:       config.Level,
        enableColor: config.EnableColor,
    }, nil
}

func (lf *LoggerFactory) createFileLogger(config LoggerConfig) (Logger, error) {
    if config.Output == "" {
        return nil, fmt.Errorf("file output path required for file logger")
    }
    
    // Ensure directory exists
    dir := filepath.Dir(config.Output)
    if err := os.MkdirAll(dir, 0755); err != nil {
        return nil, fmt.Errorf("failed to create log directory: %v", err)
    }
    
    file, err := os.OpenFile(config.Output, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        return nil, fmt.Errorf("failed to open log file: %v", err)
    }
    
    return &FileLoggerImpl{
        logger: log.New(file, config.Prefix, log.Ldate|log.Ltime|log.Lmicroseconds),
        file:   file,
        level:  config.Level,
    }, nil
}

func (lf *LoggerFactory) createJSONLogger(config LoggerConfig) (Logger, error) {
    var output io.Writer = os.Stdout
    
    if config.Output != "" {
        file, err := os.OpenFile(config.Output, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        if err != nil {
            return nil, fmt.Errorf("failed to open JSON log file: %v", err)
        }
        output = file
    }
    
    return &JSONLoggerImpl{
        output: output,
        level:  config.Level,
        fields: config.Fields,
    }, nil
}

func (lf *LoggerFactory) createDebugLogger(config LoggerConfig) (Logger, error) {
    return &DebugLoggerImpl{
        logger: log.New(os.Stderr, "[DEBUG] "+config.Prefix, log.Ldate|log.Ltime|log.Lshortfile),
        level:  "DEBUG",
    }, nil
}

// Logger implementations
type ConsoleLoggerImpl struct {
    logger      *log.Logger
    level       string
    enableColor bool
}

func (cl *ConsoleLoggerImpl) shouldLog(level string) bool {
    levels := map[string]int{"DEBUG": 0, "INFO": 1, "WARN": 2, "ERROR": 3}
    return levels[level] >= levels[cl.level]
}

func (cl *ConsoleLoggerImpl) colorize(level, message string) string {
    if !cl.enableColor {
        return fmt.Sprintf("[%s] %s", level, message)
    }
    
    var color string
    switch level {
    case "DEBUG":
        color = "\033[36m" // Cyan
    case "INFO":
        color = "\033[32m" // Green
    case "WARN":
        color = "\033[33m" // Yellow
    case "ERROR":
        color = "\033[31m" // Red
    default:
        color = "\033[0m" // Reset
    }
    
    return fmt.Sprintf("%s[%s]\033[0m %s", color, level, message)
}

func (cl *ConsoleLoggerImpl) Debug(msg string) {
    if cl.shouldLog("DEBUG") {
        cl.logger.Print(cl.colorize("DEBUG", msg))
    }
}

func (cl *ConsoleLoggerImpl) Info(msg string) {
    if cl.shouldLog("INFO") {
        cl.logger.Print(cl.colorize("INFO", msg))
    }
}

func (cl *ConsoleLoggerImpl) Warn(msg string) {
    if cl.shouldLog("WARN") {
        cl.logger.Print(cl.colorize("WARN", msg))
    }
}

func (cl *ConsoleLoggerImpl) Error(msg string) {
    if cl.shouldLog("ERROR") {
        cl.logger.Print(cl.colorize("ERROR", msg))
    }
}

func (cl *ConsoleLoggerImpl) Close() error {
    return nil
}

type FileLoggerImpl struct {
    logger *log.Logger
    file   *os.File
    level  string
}

func (fl *FileLoggerImpl) shouldLog(level string) bool {
    levels := map[string]int{"DEBUG": 0, "INFO": 1, "WARN": 2, "ERROR": 3}
    return levels[level] >= levels[fl.level]
}

func (fl *FileLoggerImpl) Debug(msg string) {
    if fl.shouldLog("DEBUG") {
        fl.logger.Printf("[DEBUG] %s", msg)
    }
}

func (fl *FileLoggerImpl) Info(msg string) {
    if fl.shouldLog("INFO") {
        fl.logger.Printf("[INFO] %s", msg)
    }
}

func (fl *FileLoggerImpl) Warn(msg string) {
    if fl.shouldLog("WARN") {
        fl.logger.Printf("[WARN] %s", msg)
    }
}

func (fl *FileLoggerImpl) Error(msg string) {
    if fl.shouldLog("ERROR") {
        fl.logger.Printf("[ERROR] %s", msg)
    }
}

func (fl *FileLoggerImpl) Close() error {
    return fl.file.Close()
}

type JSONLoggerImpl struct {
    output io.Writer
    level  string
    fields map[string]interface{}
}

func (jl *JSONLoggerImpl) shouldLog(level string) bool {
    levels := map[string]int{"DEBUG": 0, "INFO": 1, "WARN": 2, "ERROR": 3}
    return levels[level] >= levels[jl.level]
}

func (jl *JSONLoggerImpl) logJSON(level, message string) {
    entry := map[string]interface{}{
        "timestamp": time.Now().UTC().Format(time.RFC3339),
        "level":     level,
        "message":   message,
    }
    
    // Add default fields
    for k, v := range jl.fields {
        entry[k] = v
    }
    
    // Simple JSON formatting
    var jsonStr strings.Builder
    jsonStr.WriteString("{")
    first := true
    for k, v := range entry {
        if !first {
            jsonStr.WriteString(",")
        }
        jsonStr.WriteString(fmt.Sprintf(`"%s":"%v"`, k, v))
        first = false
    }
    jsonStr.WriteString("}\n")
    
    fmt.Fprint(jl.output, jsonStr.String())
}

func (jl *JSONLoggerImpl) Debug(msg string) {
    if jl.shouldLog("DEBUG") {
        jl.logJSON("DEBUG", msg)
    }
}

func (jl *JSONLoggerImpl) Info(msg string) {
    if jl.shouldLog("INFO") {
        jl.logJSON("INFO", msg)
    }
}

func (jl *JSONLoggerImpl) Warn(msg string) {
    if jl.shouldLog("WARN") {
        jl.logJSON("WARN", msg)
    }
}

func (jl *JSONLoggerImpl) Error(msg string) {
    if jl.shouldLog("ERROR") {
        jl.logJSON("ERROR", msg)
    }
}

func (jl *JSONLoggerImpl) Close() error {
    if closer, ok := jl.output.(io.Closer); ok {
        return closer.Close()
    }
    return nil
}

type DebugLoggerImpl struct {
    logger *log.Logger
    level  string
}

func (dl *DebugLoggerImpl) Debug(msg string) { dl.logger.Printf("DEBUG: %s", msg) }
func (dl *DebugLoggerImpl) Info(msg string)  { dl.logger.Printf("INFO: %s", msg) }
func (dl *DebugLoggerImpl) Warn(msg string)  { dl.logger.Printf("WARN: %s", msg) }
func (dl *DebugLoggerImpl) Error(msg string) { dl.logger.Printf("ERROR: %s", msg) }
func (dl *DebugLoggerImpl) Close() error     { return nil }

func demonstrateLoggerFactory() {
    factory := NewLoggerFactory()
    
    fmt.Println("=== Logger Factory Pattern Demo ===")
    
    // Create different types of loggers
    configs := []struct {
        name   string
        config LoggerConfig
    }{
        {
            "Console Logger",
            LoggerConfig{
                Type:        ConsoleLogger,
                Level:       "DEBUG",
                Prefix:      "[CONSOLE] ",
                EnableColor: true,
            },
        },
        {
            "File Logger",
            LoggerConfig{
                Type:   FileLogger,
                Level:  "INFO",
                Output: "/tmp/factory_demo.log",
                Prefix: "[FILE] ",
            },
        },
        {
            "JSON Logger",
            LoggerConfig{
                Type:   JSONLogger,
                Level:  "WARN",
                Fields: map[string]interface{}{
                    "service": "demo-app",
                    "version": "1.0.0",
                },
            },
        },
        {
            "Debug Logger",
            LoggerConfig{
                Type:   DebugLogger,
                Level:  "DEBUG",
                Prefix: "APP ",
            },
        },
    }
    
    for _, config := range configs {
        fmt.Printf("\n--- %s ---\n", config.name)
        
        logger, err := factory.CreateLogger(config.config)
        if err != nil {
            fmt.Printf("Failed to create logger: %v\n", err)
            continue
        }
        
        // Test all log levels
        logger.Debug("This is a debug message")
        logger.Info("This is an info message")
        logger.Warn("This is a warning message")
        logger.Error("This is an error message")
        
        logger.Close()
    }
    
    // Show file logger output
    fmt.Println("\n--- File Logger Output ---")
    if content, err := os.ReadFile("/tmp/factory_demo.log"); err == nil {
        fmt.Print(string(content))
    }
}

func main() {
    demonstrateLoggerFactory()
    
    fmt.Println("\n=== Factory Pattern Benefits ===")
    fmt.Println("✓ Centralized logger creation")
    fmt.Println("✓ Consistent configuration")
    fmt.Println("✓ Easy to add new logger types")
    fmt.Println("✓ Default value management")
    fmt.Println("✓ Type safety with interfaces")
    fmt.Println("✓ Simplified logger selection")
}
```

The logger factory pattern provides centralized, consistent logger creation with  
support for multiple logger types, configuration management, and easy extension  
for new logging requirements.  

## Testing and mocking loggers

Effective testing requires logger mocking and verification to ensure logging  
behavior without producing actual log output during tests.  

```go
package main

import (
    "fmt"
    "strings"
    "testing"
)

// Logger interface for dependency injection
type TestableLogger interface {
    Debug(msg string)
    Info(msg string)
    Warn(msg string)
    Error(msg string)
}

// Mock logger for testing
type MockLogger struct {
    entries []LogEntry
}

type LogEntry struct {
    Level   string
    Message string
}

func NewMockLogger() *MockLogger {
    return &MockLogger{entries: make([]LogEntry, 0)}
}

func (ml *MockLogger) Debug(msg string) { ml.log("DEBUG", msg) }
func (ml *MockLogger) Info(msg string)  { ml.log("INFO", msg) }
func (ml *MockLogger) Warn(msg string)  { ml.log("WARN", msg) }
func (ml *MockLogger) Error(msg string) { ml.log("ERROR", msg) }

func (ml *MockLogger) log(level, message string) {
    ml.entries = append(ml.entries, LogEntry{Level: level, Message: message})
}

func (ml *MockLogger) GetEntries() []LogEntry {
    return ml.entries
}

func (ml *MockLogger) GetEntriesByLevel(level string) []LogEntry {
    var filtered []LogEntry
    for _, entry := range ml.entries {
        if entry.Level == level {
            filtered = append(filtered, entry)
        }
    }
    return filtered
}

func (ml *MockLogger) ContainsMessage(message string) bool {
    for _, entry := range ml.entries {
        if strings.Contains(entry.Message, message) {
            return true
        }
    }
    return false
}

func (ml *MockLogger) Clear() {
    ml.entries = ml.entries[:0]
}

// Business logic that uses logging
type UserService struct {
    logger TestableLogger
}

func NewUserService(logger TestableLogger) *UserService {
    return &UserService{logger: logger}
}

func (us *UserService) CreateUser(username, email string) error {
    us.logger.Info(fmt.Sprintf("Creating user: %s", username))
    
    if username == "" {
        us.logger.Error("Username cannot be empty")
        return fmt.Errorf("invalid username")
    }
    
    if !strings.Contains(email, "@") {
        us.logger.Warn(fmt.Sprintf("Invalid email format: %s", email))
        return fmt.Errorf("invalid email")
    }
    
    // Simulate user creation
    us.logger.Debug(fmt.Sprintf("Saving user to database: %s", username))
    us.logger.Info(fmt.Sprintf("User created successfully: %s", username))
    
    return nil
}

func (us *UserService) DeleteUser(userID int) error {
    us.logger.Info(fmt.Sprintf("Deleting user ID: %d", userID))
    
    if userID <= 0 {
        us.logger.Error("Invalid user ID for deletion")
        return fmt.Errorf("invalid user ID")
    }
    
    // Simulate deletion
    us.logger.Debug(fmt.Sprintf("Removing user %d from database", userID))
    us.logger.Info(fmt.Sprintf("User %d deleted successfully", userID))
    
    return nil
}

// Test functions (would normally use testing.T)
func TestUserCreation() {
    mockLogger := NewMockLogger()
    userService := NewUserService(mockLogger)
    
    fmt.Println("=== Test: Valid User Creation ===")
    
    err := userService.CreateUser("alice", "alice@example.com")
    if err != nil {
        fmt.Printf("Unexpected error: %v\n", err)
        return
    }
    
    // Verify logging behavior
    entries := mockLogger.GetEntries()
    fmt.Printf("Total log entries: %d\n", len(entries))
    
    if !mockLogger.ContainsMessage("Creating user: alice") {
        fmt.Println("FAIL: Missing 'Creating user' log entry")
    } else {
        fmt.Println("PASS: Found 'Creating user' log entry")
    }
    
    if !mockLogger.ContainsMessage("User created successfully: alice") {
        fmt.Println("FAIL: Missing success log entry")
    } else {
        fmt.Println("PASS: Found success log entry")
    }
    
    infoEntries := mockLogger.GetEntriesByLevel("INFO")
    if len(infoEntries) != 2 {
        fmt.Printf("FAIL: Expected 2 INFO entries, got %d\n", len(infoEntries))
    } else {
        fmt.Println("PASS: Correct number of INFO entries")
    }
}

func TestUserCreationErrors() {
    fmt.Println("\n=== Test: User Creation Errors ===")
    
    // Test empty username
    mockLogger := NewMockLogger()
    userService := NewUserService(mockLogger)
    
    err := userService.CreateUser("", "test@example.com")
    if err == nil {
        fmt.Println("FAIL: Expected error for empty username")
    } else {
        fmt.Println("PASS: Got expected error for empty username")
    }
    
    if !mockLogger.ContainsMessage("Username cannot be empty") {
        fmt.Println("FAIL: Missing error log for empty username")
    } else {
        fmt.Println("PASS: Found error log for empty username")
    }
    
    // Test invalid email
    mockLogger.Clear()
    err = userService.CreateUser("bob", "invalid-email")
    if err == nil {
        fmt.Println("FAIL: Expected error for invalid email")
    } else {
        fmt.Println("PASS: Got expected error for invalid email")
    }
    
    if !mockLogger.ContainsMessage("Invalid email format") {
        fmt.Println("FAIL: Missing warning log for invalid email")
    } else {
        fmt.Println("PASS: Found warning log for invalid email")
    }
    
    warnEntries := mockLogger.GetEntriesByLevel("WARN")
    if len(warnEntries) != 1 {
        fmt.Printf("FAIL: Expected 1 WARN entry, got %d\n", len(warnEntries))
    } else {
        fmt.Println("PASS: Correct number of WARN entries")
    }
}

func TestUserDeletion() {
    fmt.Println("\n=== Test: User Deletion ===")
    
    mockLogger := NewMockLogger()
    userService := NewUserService(mockLogger)
    
    // Test valid deletion
    err := userService.DeleteUser(123)
    if err != nil {
        fmt.Printf("Unexpected error: %v\n", err)
        return
    }
    
    if !mockLogger.ContainsMessage("Deleting user ID: 123") {
        fmt.Println("FAIL: Missing deletion start log")
    } else {
        fmt.Println("PASS: Found deletion start log")
    }
    
    // Test invalid deletion
    mockLogger.Clear()
    err = userService.DeleteUser(-1)
    if err == nil {
        fmt.Println("FAIL: Expected error for invalid user ID")
    } else {
        fmt.Println("PASS: Got expected error for invalid user ID")
    }
    
    errorEntries := mockLogger.GetEntriesByLevel("ERROR")
    if len(errorEntries) != 1 {
        fmt.Printf("FAIL: Expected 1 ERROR entry, got %d\n", len(errorEntries))
    } else {
        fmt.Println("PASS: Correct number of ERROR entries")
    }
}

// Spy logger that tracks calls
type SpyLogger struct {
    debugCalls int
    infoCalls  int
    warnCalls  int
    errorCalls int
    lastMessage string
}

func NewSpyLogger() *SpyLogger {
    return &SpyLogger{}
}

func (sl *SpyLogger) Debug(msg string) { sl.debugCalls++; sl.lastMessage = msg }
func (sl *SpyLogger) Info(msg string)  { sl.infoCalls++; sl.lastMessage = msg }
func (sl *SpyLogger) Warn(msg string)  { sl.warnCalls++; sl.lastMessage = msg }
func (sl *SpyLogger) Error(msg string) { sl.errorCalls++; sl.lastMessage = msg }

func (sl *SpyLogger) GetCallCounts() (int, int, int, int) {
    return sl.debugCalls, sl.infoCalls, sl.warnCalls, sl.errorCalls
}

func TestLoggingFrequency() {
    fmt.Println("\n=== Test: Logging Frequency ===")
    
    spyLogger := NewSpyLogger()
    userService := NewUserService(spyLogger)
    
    // Perform multiple operations
    userService.CreateUser("user1", "user1@example.com")
    userService.CreateUser("user2", "user2@example.com")
    userService.DeleteUser(1)
    userService.DeleteUser(2)
    
    debug, info, warn, error := spyLogger.GetCallCounts()
    
    fmt.Printf("Call counts - Debug: %d, Info: %d, Warn: %d, Error: %d\n", 
        debug, info, warn, error)
    
    expectedInfo := 6 // 2 create start + 2 create success + 2 delete start + 2 delete success
    if info != expectedInfo {
        fmt.Printf("FAIL: Expected %d INFO calls, got %d\n", expectedInfo, info)
    } else {
        fmt.Println("PASS: Correct number of INFO calls")
    }
}

func main() {
    fmt.Println("=== Logger Testing and Mocking Demo ===")
    
    TestUserCreation()
    TestUserCreationErrors()
    TestUserDeletion()
    TestLoggingFrequency()
    
    fmt.Println("\n=== Testing Benefits ===")
    fmt.Println("✓ Verify logging behavior without output")
    fmt.Println("✓ Test different log levels separately")
    fmt.Println("✓ Ensure proper error logging")
    fmt.Println("✓ Validate log message content")
    fmt.Println("✓ Count logging frequency")
    fmt.Println("✓ Test logging in isolation")
}
```

Testing loggers through mocking enables verification of logging behavior without  
producing actual output, ensuring proper error handling and message content  
validation in unit tests.  

## Metrics and monitoring integration

Integrating logging with metrics collection provides comprehensive observability  
for application performance and health monitoring.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type MetricsLogger struct {
    metrics   *LogMetrics
    logger    Logger
    mu        sync.RWMutex
}

type LogMetrics struct {
    TotalLogs     int64
    LogsByLevel   map[string]int64
    ErrorCount    int64
    WarnCount     int64
    LastLogTime   time.Time
    LogRate       float64 // logs per second
    StartTime     time.Time
    
    // Performance metrics
    AverageLogTime time.Duration
    TotalLogTime   time.Duration
    
    // Error tracking
    RecentErrors []ErrorInfo
    MaxErrors    int
}

type ErrorInfo struct {
    Timestamp time.Time
    Message   string
    Level     string
}

type Logger interface {
    Log(level, message string)
}

type SimpleLogger struct{}

func (sl *SimpleLogger) Log(level, message string) {
    fmt.Printf("[%s] %s: %s\n", time.Now().Format("15:04:05"), level, message)
}

func NewMetricsLogger(logger Logger) *MetricsLogger {
    return &MetricsLogger{
        logger: logger,
        metrics: &LogMetrics{
            LogsByLevel:  make(map[string]int64),
            StartTime:    time.Now(),
            RecentErrors: make([]ErrorInfo, 0),
            MaxErrors:    10,
        },
    }
}

func (ml *MetricsLogger) Log(level, message string) {
    start := time.Now()
    
    // Log the message
    ml.logger.Log(level, message)
    
    // Update metrics
    ml.updateMetrics(level, message, time.Since(start))
}

func (ml *MetricsLogger) updateMetrics(level, message string, logTime time.Duration) {
    ml.mu.Lock()
    defer ml.mu.Unlock()
    
    // Update counters
    ml.metrics.TotalLogs++
    ml.metrics.LogsByLevel[level]++
    ml.metrics.LastLogTime = time.Now()
    
    // Update performance metrics
    ml.metrics.TotalLogTime += logTime
    ml.metrics.AverageLogTime = time.Duration(int64(ml.metrics.TotalLogTime) / ml.metrics.TotalLogs)
    
    // Calculate log rate
    elapsed := time.Since(ml.metrics.StartTime).Seconds()
    if elapsed > 0 {
        ml.metrics.LogRate = float64(ml.metrics.TotalLogs) / elapsed
    }
    
    // Track errors and warnings
    if level == "ERROR" {
        ml.metrics.ErrorCount++
        ml.addRecentError(level, message)
    } else if level == "WARN" {
        ml.metrics.WarnCount++
        ml.addRecentError(level, message)
    }
}

func (ml *MetricsLogger) addRecentError(level, message string) {
    errorInfo := ErrorInfo{
        Timestamp: time.Now(),
        Message:   message,
        Level:     level,
    }
    
    ml.metrics.RecentErrors = append(ml.metrics.RecentErrors, errorInfo)
    
    // Keep only recent errors
    if len(ml.metrics.RecentErrors) > ml.metrics.MaxErrors {
        ml.metrics.RecentErrors = ml.metrics.RecentErrors[1:]
    }
}

func (ml *MetricsLogger) GetMetrics() LogMetrics {
    ml.mu.RLock()
    defer ml.mu.RUnlock()
    
    // Return a copy
    metrics := *ml.metrics
    metrics.LogsByLevel = make(map[string]int64)
    for k, v := range ml.metrics.LogsByLevel {
        metrics.LogsByLevel[k] = v
    }
    
    metrics.RecentErrors = make([]ErrorInfo, len(ml.metrics.RecentErrors))
    copy(metrics.RecentErrors, ml.metrics.RecentErrors)
    
    return metrics
}

func (ml *MetricsLogger) PrintMetrics() {
    metrics := ml.GetMetrics()
    
    fmt.Println("\n=== Logging Metrics ===")
    fmt.Printf("Total Logs: %d\n", metrics.TotalLogs)
    fmt.Printf("Uptime: %v\n", time.Since(metrics.StartTime))
    fmt.Printf("Log Rate: %.2f logs/second\n", metrics.LogRate)
    fmt.Printf("Average Log Time: %v\n", metrics.AverageLogTime)
    fmt.Printf("Last Log: %v ago\n", time.Since(metrics.LastLogTime))
    
    fmt.Println("\nLogs by Level:")
    for level, count := range metrics.LogsByLevel {
        percentage := float64(count) / float64(metrics.TotalLogs) * 100
        fmt.Printf("  %s: %d (%.1f%%)\n", level, count, percentage)
    }
    
    fmt.Printf("\nError/Warning Summary:\n")
    fmt.Printf("  Total Errors: %d\n", metrics.ErrorCount)
    fmt.Printf("  Total Warnings: %d\n", metrics.WarnCount)
    
    if len(metrics.RecentErrors) > 0 {
        fmt.Printf("\nRecent Errors/Warnings:\n")
        for i, err := range metrics.RecentErrors {
            fmt.Printf("  %d. [%s] %s (%v ago)\n", 
                i+1, err.Level, err.Message, time.Since(err.Timestamp))
        }
    }
}

// Health checker that uses metrics
type HealthChecker struct {
    metricsLogger *MetricsLogger
    thresholds    HealthThresholds
}

type HealthThresholds struct {
    MaxErrorRate     float64       // errors per second
    MaxLogRate       float64       // logs per second
    MaxAvgLogTime    time.Duration // average log processing time
    MaxTimeSinceLog  time.Duration // time since last log
}

type HealthStatus struct {
    Healthy  bool
    Issues   []string
    Metrics  LogMetrics
}

func NewHealthChecker(ml *MetricsLogger) *HealthChecker {
    return &HealthChecker{
        metricsLogger: ml,
        thresholds: HealthThresholds{
            MaxErrorRate:     5.0,
            MaxLogRate:       1000.0,
            MaxAvgLogTime:    10 * time.Millisecond,
            MaxTimeSinceLog:  5 * time.Minute,
        },
    }
}

func (hc *HealthChecker) CheckHealth() HealthStatus {
    metrics := hc.metricsLogger.GetMetrics()
    status := HealthStatus{
        Healthy: true,
        Issues:  make([]string, 0),
        Metrics: metrics,
    }
    
    // Check error rate
    elapsed := time.Since(metrics.StartTime).Seconds()
    if elapsed > 0 {
        errorRate := float64(metrics.ErrorCount) / elapsed
        if errorRate > hc.thresholds.MaxErrorRate {
            status.Healthy = false
            status.Issues = append(status.Issues, 
                fmt.Sprintf("High error rate: %.2f/sec (threshold: %.2f/sec)", 
                    errorRate, hc.thresholds.MaxErrorRate))
        }
    }
    
    // Check log rate
    if metrics.LogRate > hc.thresholds.MaxLogRate {
        status.Healthy = false
        status.Issues = append(status.Issues, 
            fmt.Sprintf("High log rate: %.2f/sec (threshold: %.2f/sec)", 
                metrics.LogRate, hc.thresholds.MaxLogRate))
    }
    
    // Check average log time
    if metrics.AverageLogTime > hc.thresholds.MaxAvgLogTime {
        status.Healthy = false
        status.Issues = append(status.Issues, 
            fmt.Sprintf("Slow logging: %v (threshold: %v)", 
                metrics.AverageLogTime, hc.thresholds.MaxAvgLogTime))
    }
    
    // Check time since last log
    if time.Since(metrics.LastLogTime) > hc.thresholds.MaxTimeSinceLog {
        status.Healthy = false
        status.Issues = append(status.Issues, 
            fmt.Sprintf("No recent logs: %v ago (threshold: %v)", 
                time.Since(metrics.LastLogTime), hc.thresholds.MaxTimeSinceLog))
    }
    
    return status
}

func simulateApplicationActivity(logger *MetricsLogger) {
    fmt.Println("Simulating application activity...")
    
    activities := []struct {
        level   string
        message string
        delay   time.Duration
    }{
        {"INFO", "Application started", 10 * time.Millisecond},
        {"DEBUG", "Loading configuration", 5 * time.Millisecond},
        {"INFO", "Database connected", 20 * time.Millisecond},
        {"INFO", "Server listening on port 8080", 5 * time.Millisecond},
        {"DEBUG", "Processing request 1", 2 * time.Millisecond},
        {"INFO", "Request completed successfully", 3 * time.Millisecond},
        {"WARN", "High memory usage: 85%", 8 * time.Millisecond},
        {"DEBUG", "Garbage collection triggered", 15 * time.Millisecond},
        {"ERROR", "Database connection timeout", 25 * time.Millisecond},
        {"INFO", "Retrying database connection", 10 * time.Millisecond},
        {"INFO", "Database reconnected", 18 * time.Millisecond},
        {"ERROR", "Failed to process payment", 12 * time.Millisecond},
        {"WARN", "API rate limit exceeded", 7 * time.Millisecond},
        {"INFO", "Scheduled task completed", 4 * time.Millisecond},
        {"DEBUG", "Cache cleanup performed", 6 * time.Millisecond},
    }
    
    for _, activity := range activities {
        time.Sleep(activity.delay)
        logger.Log(activity.level, activity.message)
        time.Sleep(50 * time.Millisecond) // Simulate work between logs
    }
}

func main() {
    fmt.Println("=== Metrics and Monitoring Integration Demo ===")
    
    // Create metrics logger
    baseLogger := &SimpleLogger{}
    metricsLogger := NewMetricsLogger(baseLogger)
    
    // Create health checker
    healthChecker := NewHealthChecker(metricsLogger)
    
    // Simulate application activity
    simulateApplicationActivity(metricsLogger)
    
    // Print metrics
    metricsLogger.PrintMetrics()
    
    // Check health
    fmt.Println("\n=== Health Check ===")
    health := healthChecker.CheckHealth()
    
    if health.Healthy {
        fmt.Println("✓ System is healthy")
    } else {
        fmt.Println("✗ System health issues detected:")
        for _, issue := range health.Issues {
            fmt.Printf("  - %s\n", issue)
        }
    }
    
    // Demonstrate real-time monitoring
    fmt.Println("\n=== Real-time Monitoring ===")
    fmt.Println("Generating high-frequency logs...")
    
    start := time.Now()
    for i := 0; i < 100; i++ {
        metricsLogger.Log("INFO", fmt.Sprintf("High-frequency log %d", i))
        if i%10 == 0 && i > 0 {
            time.Sleep(1 * time.Millisecond) // Brief pause
        }
    }
    duration := time.Since(start)
    
    fmt.Printf("Generated 100 logs in %v\n", duration)
    
    // Final metrics
    metricsLogger.PrintMetrics()
    
    fmt.Println("\n=== Monitoring Benefits ===")
    fmt.Println("✓ Real-time performance tracking")
    fmt.Println("✓ Error rate monitoring")
    fmt.Println("✓ Log volume analysis")
    fmt.Println("✓ Health status indicators")
    fmt.Println("✓ Historical error tracking")
    fmt.Println("✓ Performance bottleneck detection")
}
```

Metrics integration with logging provides comprehensive observability through  
performance tracking, error rate monitoring, and health status indicators  
essential for production system management.  

## Configuration-driven logging

Dynamic logging configuration enables runtime adjustments without application  
restarts, supporting different environments and debugging scenarios.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "strings"
    "sync"
    "time"
)

type LoggingConfig struct {
    Level       string                 `json:"level"`
    Format      string                 `json:"format"`
    Output      string                 `json:"output"`
    MaxSize     int64                  `json:"max_size"`
    MaxAge      int                    `json:"max_age"`
    MaxBackups  int                    `json:"max_backups"`
    Compress    bool                   `json:"compress"`
    EnableColor bool                   `json:"enable_color"`
    Fields      map[string]interface{} `json:"fields"`
    Sampling    SamplingConfig         `json:"sampling"`
}

type SamplingConfig struct {
    Enabled bool    `json:"enabled"`
    Rate    float64 `json:"rate"`
}

type ConfigurableLogger struct {
    config     LoggingConfig
    configFile string
    mu         sync.RWMutex
    lastReload time.Time
}

func NewConfigurableLogger(configFile string) (*ConfigurableLogger, error) {
    cl := &ConfigurableLogger{
        configFile: configFile,
        lastReload: time.Now(),
    }
    
    if err := cl.LoadConfig(); err != nil {
        return nil, err
    }
    
    return cl, nil
}

func (cl *ConfigurableLogger) LoadConfig() error {
    cl.mu.Lock()
    defer cl.mu.Unlock()
    
    // Default configuration
    defaultConfig := LoggingConfig{
        Level:       "INFO",
        Format:      "text",
        Output:      "stdout",
        MaxSize:     100 * 1024 * 1024, // 100MB
        MaxAge:      30,                 // 30 days
        MaxBackups:  5,
        Compress:    true,
        EnableColor: true,
        Fields:      make(map[string]interface{}),
        Sampling: SamplingConfig{
            Enabled: false,
            Rate:    1.0,
        },
    }
    
    // Try to load from file
    if cl.configFile != "" {
        data, err := os.ReadFile(cl.configFile)
        if err != nil {
            fmt.Printf("Warning: Could not read config file %s: %v\n", cl.configFile, err)
            cl.config = defaultConfig
            return nil
        }
        
        if err := json.Unmarshal(data, &cl.config); err != nil {
            fmt.Printf("Warning: Could not parse config file %s: %v\n", cl.configFile, err)
            cl.config = defaultConfig
            return nil
        }
        
        fmt.Printf("Loaded logging configuration from %s\n", cl.configFile)
    } else {
        cl.config = defaultConfig
    }
    
    cl.lastReload = time.Now()
    return nil
}

func (cl *ConfigurableLogger) shouldLog(level string) bool {
    cl.mu.RLock()
    defer cl.mu.RUnlock()
    
    levels := map[string]int{
        "DEBUG": 0,
        "INFO":  1,
        "WARN":  2,
        "ERROR": 3,
    }
    
    return levels[level] >= levels[cl.config.Level]
}

func (cl *ConfigurableLogger) log(level, message string) {
    if !cl.shouldLog(level) {
        return
    }
    
    cl.mu.RLock()
    config := cl.config
    cl.mu.RUnlock()
    
    // Apply sampling if enabled
    if config.Sampling.Enabled && level != "ERROR" {
        // Simple sampling - in production use proper random sampling
        if time.Now().UnixNano()%100 >= int64(config.Sampling.Rate*100) {
            return
        }
    }
    
    timestamp := time.Now()
    
    var output string
    
    switch config.Format {
    case "json":
        output = cl.formatJSON(level, message, timestamp, config)
    default:
        output = cl.formatText(level, message, timestamp, config)
    }
    
    // Output to destination
    switch config.Output {
    case "stdout":
        fmt.Print(output)
    case "stderr":
        fmt.Fprint(os.Stderr, output)
    default:
        // File output
        file, err := os.OpenFile(config.Output, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        if err != nil {
            fmt.Fprintf(os.Stderr, "Failed to open log file: %v\n", err)
            fmt.Print(output) // Fallback to stdout
        } else {
            file.WriteString(output)
            file.Close()
        }
    }
}

func (cl *ConfigurableLogger) formatJSON(level, message string, timestamp time.Time, config LoggingConfig) string {
    entry := map[string]interface{}{
        "timestamp": timestamp.Format(time.RFC3339),
        "level":     level,
        "message":   message,
    }
    
    // Add configured fields
    for k, v := range config.Fields {
        entry[k] = v
    }
    
    jsonData, _ := json.Marshal(entry)
    return string(jsonData) + "\n"
}

func (cl *ConfigurableLogger) formatText(level, message string, timestamp time.Time, config LoggingConfig) string {
    timestampStr := timestamp.Format("2006-01-02 15:04:05")
    
    levelStr := level
    if config.EnableColor {
        switch level {
        case "DEBUG":
            levelStr = fmt.Sprintf("\033[36m%s\033[0m", level)
        case "INFO":
            levelStr = fmt.Sprintf("\033[32m%s\033[0m", level)
        case "WARN":
            levelStr = fmt.Sprintf("\033[33m%s\033[0m", level)
        case "ERROR":
            levelStr = fmt.Sprintf("\033[31m%s\033[0m", level)
        }
    }
    
    result := fmt.Sprintf("[%s] %s: %s", timestampStr, levelStr, message)
    
    // Add configured fields
    if len(config.Fields) > 0 {
        result += " |"
        for k, v := range config.Fields {
            result += fmt.Sprintf(" %s=%v", k, v)
        }
    }
    
    return result + "\n"
}

func (cl *ConfigurableLogger) Debug(message string) { cl.log("DEBUG", message) }
func (cl *ConfigurableLogger) Info(message string)  { cl.log("INFO", message) }
func (cl *ConfigurableLogger) Warn(message string)  { cl.log("WARN", message) }
func (cl *ConfigurableLogger) Error(message string) { cl.log("ERROR", message) }

func (cl *ConfigurableLogger) ReloadConfig() error {
    fmt.Println("Reloading logging configuration...")
    return cl.LoadConfig()
}

func (cl *ConfigurableLogger) GetConfig() LoggingConfig {
    cl.mu.RLock()
    defer cl.mu.RUnlock()
    return cl.config
}

func (cl *ConfigurableLogger) SetLevel(level string) {
    cl.mu.Lock()
    defer cl.mu.Unlock()
    cl.config.Level = level
    fmt.Printf("Log level changed to: %s\n", level)
}

func (cl *ConfigurableLogger) PrintConfig() {
    config := cl.GetConfig()
    
    fmt.Println("\n=== Current Logging Configuration ===")
    fmt.Printf("Level: %s\n", config.Level)
    fmt.Printf("Format: %s\n", config.Format)
    fmt.Printf("Output: %s\n", config.Output)
    fmt.Printf("Max Size: %d bytes\n", config.MaxSize)
    fmt.Printf("Max Age: %d days\n", config.MaxAge)
    fmt.Printf("Max Backups: %d\n", config.MaxBackups)
    fmt.Printf("Compress: %t\n", config.Compress)
    fmt.Printf("Enable Color: %t\n", config.EnableColor)
    fmt.Printf("Sampling: enabled=%t, rate=%.2f\n", config.Sampling.Enabled, config.Sampling.Rate)
    
    if len(config.Fields) > 0 {
        fmt.Println("Fields:")
        for k, v := range config.Fields {
            fmt.Printf("  %s: %v\n", k, v)
        }
    }
    
    fmt.Printf("Last Reload: %v\n", cl.lastReload)
}

func createSampleConfig(filename string) error {
    config := LoggingConfig{
        Level:       "DEBUG",
        Format:      "json",
        Output:      "/tmp/configurable.log",
        MaxSize:     50 * 1024 * 1024, // 50MB
        MaxAge:      7,                 // 7 days
        MaxBackups:  3,
        Compress:    true,
        EnableColor: false,
        Fields: map[string]interface{}{
            "service":     "demo-app",
            "version":     "1.0.0",
            "environment": "development",
        },
        Sampling: SamplingConfig{
            Enabled: true,
            Rate:    0.8,
        },
    }
    
    data, err := json.MarshalIndent(config, "", "  ")
    if err != nil {
        return err
    }
    
    return os.WriteFile(filename, data, 0644)
}

func demonstrateConfigChanges(logger *ConfigurableLogger) {
    fmt.Println("\n=== Demonstrating Configuration Changes ===")
    
    // Initial logging
    logger.Debug("Initial debug message")
    logger.Info("Initial info message")
    logger.Warn("Initial warning message")
    logger.Error("Initial error message")
    
    // Change log level dynamically
    fmt.Println("\nChanging log level to ERROR...")
    logger.SetLevel("ERROR")
    
    logger.Debug("This debug message should not appear")
    logger.Info("This info message should not appear")
    logger.Warn("This warning message should not appear")
    logger.Error("This error message should appear")
    
    // Reset level
    fmt.Println("\nChanging log level back to DEBUG...")
    logger.SetLevel("DEBUG")
    
    logger.Debug("Debug is back")
    logger.Info("Info is back")
    
    // Reload configuration
    fmt.Println("\nReloading configuration from file...")
    if err := logger.ReloadConfig(); err != nil {
        fmt.Printf("Failed to reload config: %v\n", err)
    } else {
        logger.Info("Configuration reloaded successfully")
        logger.Warn("This message uses reloaded config")
    }
}

func main() {
    fmt.Println("=== Configuration-driven Logging Demo ===")
    
    // Create sample configuration file
    configFile := "/tmp/logging-config.json"
    if err := createSampleConfig(configFile); err != nil {
        fmt.Printf("Failed to create config file: %v\n", err)
        return
    }
    
    fmt.Printf("Created sample configuration file: %s\n", configFile)
    
    // Create configurable logger
    logger, err := NewConfigurableLogger(configFile)
    if err != nil {
        fmt.Printf("Failed to create logger: %v\n", err)
        return
    }
    
    // Display initial configuration
    logger.PrintConfig()
    
    // Demonstrate configuration-driven behavior
    demonstrateConfigChanges(logger)
    
    // Show final configuration
    logger.PrintConfig()
    
    fmt.Println("\n=== Configuration Benefits ===")
    fmt.Println("✓ Runtime configuration changes")
    fmt.Println("✓ Environment-specific settings")
    fmt.Println("✓ Dynamic log level adjustment")
    fmt.Println("✓ Hot configuration reloading")
    fmt.Println("✓ Centralized configuration management")
    fmt.Println("✓ No application restart required")
    
    // Show configuration file content
    fmt.Println("\n=== Sample Configuration File ===")
    content, _ := os.ReadFile(configFile)
    fmt.Print(string(content))
}
```

Configuration-driven logging enables dynamic runtime adjustments, environment-  
specific settings, and centralized configuration management without requiring  
application restarts or code changes.  

## Custom log formatters

Creating custom log formatters allows precise control over log output format  
for specific requirements and integration with external systems.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strings"
    "time"
)

type LogFormatter interface {
    Format(entry LogEntry) string
}

type LogEntry struct {
    Timestamp time.Time
    Level     string
    Message   string
    Fields    map[string]interface{}
    Caller    string
    TraceID   string
}

// Simple text formatter
type TextFormatter struct {
    TimestampFormat string
    EnableCaller    bool
    FieldSeparator  string
}

func NewTextFormatter() *TextFormatter {
    return &TextFormatter{
        TimestampFormat: "2006-01-02 15:04:05",
        EnableCaller:    true,
        FieldSeparator:  " | ",
    }
}

func (tf *TextFormatter) Format(entry LogEntry) string {
    var result strings.Builder
    
    // Timestamp
    result.WriteString(fmt.Sprintf("[%s]", entry.Timestamp.Format(tf.TimestampFormat)))
    
    // Level
    result.WriteString(fmt.Sprintf(" %s:", entry.Level))
    
    // Message
    result.WriteString(fmt.Sprintf(" %s", entry.Message))
    
    // Caller information
    if tf.EnableCaller && entry.Caller != "" {
        result.WriteString(fmt.Sprintf(" (%s)", entry.Caller))
    }
    
    // Trace ID
    if entry.TraceID != "" {
        result.WriteString(fmt.Sprintf(" [trace:%s]", entry.TraceID))
    }
    
    // Fields
    if len(entry.Fields) > 0 {
        result.WriteString(tf.FieldSeparator)
        first := true
        for k, v := range entry.Fields {
            if !first {
                result.WriteString(", ")
            }
            result.WriteString(fmt.Sprintf("%s=%v", k, v))
            first = false
        }
    }
    
    result.WriteString("\n")
    return result.String()
}

// JSON formatter
type JSONFormatter struct {
    PrettyPrint bool
    ExcludeEmpty bool
}

func NewJSONFormatter() *JSONFormatter {
    return &JSONFormatter{
        PrettyPrint: false,
        ExcludeEmpty: true,
    }
}

func (jf *JSONFormatter) Format(entry LogEntry) string {
    logData := map[string]interface{}{
        "timestamp": entry.Timestamp.Format(time.RFC3339),
        "level":     entry.Level,
        "message":   entry.Message,
    }
    
    // Add optional fields
    if entry.Caller != "" || !jf.ExcludeEmpty {
        logData["caller"] = entry.Caller
    }
    
    if entry.TraceID != "" || !jf.ExcludeEmpty {
        logData["trace_id"] = entry.TraceID
    }
    
    // Add custom fields
    for k, v := range entry.Fields {
        logData[k] = v
    }
    
    var jsonData []byte
    var err error
    
    if jf.PrettyPrint {
        jsonData, err = json.MarshalIndent(logData, "", "  ")
    } else {
        jsonData, err = json.Marshal(logData)
    }
    
    if err != nil {
        return fmt.Sprintf("JSON formatting error: %v\n", err)
    }
    
    return string(jsonData) + "\n"
}

// Syslog formatter (RFC3164)
type SyslogFormatter struct {
    Facility int
    Hostname string
    AppName  string
}

func NewSyslogFormatter(facility int, hostname, appName string) *SyslogFormatter {
    return &SyslogFormatter{
        Facility: facility,
        Hostname: hostname,
        AppName:  appName,
    }
}

func (sf *SyslogFormatter) Format(entry LogEntry) string {
    // Map log levels to syslog severity
    severity := map[string]int{
        "DEBUG": 7,
        "INFO":  6,
        "WARN":  4,
        "ERROR": 3,
    }
    
    priority := sf.Facility*8 + severity[entry.Level]
    
    timestamp := entry.Timestamp.Format("Jan 2 15:04:05")
    
    // Basic syslog format: <priority>timestamp hostname appname: message
    result := fmt.Sprintf("<%d>%s %s %s: %s",
        priority, timestamp, sf.Hostname, sf.AppName, entry.Message)
    
    // Add structured fields as key=value pairs
    if len(entry.Fields) > 0 {
        result += " |"
        for k, v := range entry.Fields {
            result += fmt.Sprintf(" %s=%v", k, v)
        }
    }
    
    return result + "\n"
}

// Custom CSV formatter
type CSVFormatter struct {
    Delimiter   string
    Headers     []string
    QuoteValues bool
}

func NewCSVFormatter() *CSVFormatter {
    return &CSVFormatter{
        Delimiter:   ",",
        Headers:     []string{"timestamp", "level", "message", "caller", "trace_id"},
        QuoteValues: true,
    }
}

func (cf *CSVFormatter) GetHeaders() string {
    return strings.Join(cf.Headers, cf.Delimiter) + "\n"
}

func (cf *CSVFormatter) Format(entry LogEntry) string {
    values := make([]string, len(cf.Headers))
    
    for i, header := range cf.Headers {
        var value string
        
        switch header {
        case "timestamp":
            value = entry.Timestamp.Format(time.RFC3339)
        case "level":
            value = entry.Level
        case "message":
            value = entry.Message
        case "caller":
            value = entry.Caller
        case "trace_id":
            value = entry.TraceID
        default:
            // Check in fields
            if v, exists := entry.Fields[header]; exists {
                value = fmt.Sprintf("%v", v)
            }
        }
        
        // Quote values if needed
        if cf.QuoteValues && strings.Contains(value, cf.Delimiter) {
            value = fmt.Sprintf(`"%s"`, strings.ReplaceAll(value, `"`, `""`))
        }
        
        values[i] = value
    }
    
    return strings.Join(values, cf.Delimiter) + "\n"
}

// Compact formatter for high-volume logging
type CompactFormatter struct {
    UseShortLevel bool
}

func NewCompactFormatter() *CompactFormatter {
    return &CompactFormatter{
        UseShortLevel: true,
    }
}

func (cf *CompactFormatter) Format(entry LogEntry) string {
    timestamp := entry.Timestamp.Format("15:04:05.000")
    
    level := entry.Level
    if cf.UseShortLevel {
        levelMap := map[string]string{
            "DEBUG": "D",
            "INFO":  "I",
            "WARN":  "W",
            "ERROR": "E",
        }
        if short, exists := levelMap[entry.Level]; exists {
            level = short
        }
    }
    
    result := fmt.Sprintf("%s %s %s", timestamp, level, entry.Message)
    
    // Add essential fields only
    essentialFields := []string{"user_id", "request_id", "error"}
    for _, field := range essentialFields {
        if value, exists := entry.Fields[field]; exists {
            result += fmt.Sprintf(" %s=%v", field, value)
        }
    }
    
    return result + "\n"
}

// Custom logger using formatters
type FormatterLogger struct {
    formatter LogFormatter
}

func NewFormatterLogger(formatter LogFormatter) *FormatterLogger {
    return &FormatterLogger{formatter: formatter}
}

func (fl *FormatterLogger) Log(level, message string, fields map[string]interface{}) {
    entry := LogEntry{
        Timestamp: time.Now(),
        Level:     level,
        Message:   message,
        Fields:    fields,
        Caller:    "main.go:123", // Simplified for demo
        TraceID:   "trace-12345",
    }
    
    output := fl.formatter.Format(entry)
    fmt.Print(output)
}

func demonstrateFormatters() {
    fmt.Println("=== Custom Log Formatters Demo ===")
    
    // Sample log data
    sampleFields := map[string]interface{}{
        "user_id":    12345,
        "request_id": "req-abcd-1234",
        "duration":   "150ms",
        "status":     "success",
    }
    
    formatters := []struct {
        name      string
        formatter LogFormatter
    }{
        {"Text Formatter", NewTextFormatter()},
        {"JSON Formatter", NewJSONFormatter()},
        {"Syslog Formatter", NewSyslogFormatter(16, "web-server-01", "myapp")},
        {"CSV Formatter", NewCSVFormatter()},
        {"Compact Formatter", NewCompactFormatter()},
    }
    
    for _, f := range formatters {
        fmt.Printf("\n--- %s ---\n", f.name)
        
        logger := NewFormatterLogger(f.formatter)
        
        // Special handling for CSV headers
        if csvFormatter, ok := f.formatter.(*CSVFormatter); ok {
            fmt.Print(csvFormatter.GetHeaders())
        }
        
        logger.Log("INFO", "User logged in successfully", sampleFields)
        logger.Log("WARN", "Memory usage high", map[string]interface{}{
            "memory_percent": 85,
            "threshold":      80,
        })
        logger.Log("ERROR", "Database connection failed", map[string]interface{}{
            "error":        "connection timeout",
            "retry_count":  3,
            "database":     "users_db",
        })
    }
}

func demonstrateCustomFormatter() {
    fmt.Println("\n=== Custom Formatter Example ===")
    
    // Create a custom key-value formatter
    type KeyValueFormatter struct {
        Separator string
    }
    
    keyValueFormatter := &KeyValueFormatter{Separator: " :: "}
    
    // Implement the LogFormatter interface
    format := func(entry LogEntry) string {
        var result strings.Builder
        
        result.WriteString(fmt.Sprintf("time=%s", entry.Timestamp.Format("15:04:05")))
        result.WriteString(keyValueFormatter.Separator)
        result.WriteString(fmt.Sprintf("level=%s", entry.Level))
        result.WriteString(keyValueFormatter.Separator)
        result.WriteString(fmt.Sprintf("msg=\"%s\"", entry.Message))
        
        for k, v := range entry.Fields {
            result.WriteString(keyValueFormatter.Separator)
            result.WriteString(fmt.Sprintf("%s=%v", k, v))
        }
        
        return result.String() + "\n"
    }
    
    // Anonymous struct implementing LogFormatter
    customFormatter := struct{ LogFormatter }{
        LogFormatter: LogFormatterFunc(format),
    }
    
    logger := NewFormatterLogger(customFormatter)
    logger.Log("INFO", "Custom formatter test", map[string]interface{}{
        "component": "auth",
        "action":    "login",
        "success":   true,
    })
}

// Adapter to convert function to LogFormatter interface
type LogFormatterFunc func(LogEntry) string

func (f LogFormatterFunc) Format(entry LogEntry) string {
    return f(entry)
}

func main() {
    demonstrateFormatters()
    demonstrateCustomFormatter()
    
    fmt.Println("\n=== Formatter Benefits ===")
    fmt.Println("✓ Precise output control")
    fmt.Println("✓ Integration with external systems")
    fmt.Println("✓ Multiple output formats from same data")
    fmt.Println("✓ Performance optimization for specific use cases")
    fmt.Println("✓ Standards compliance (syslog, CSV, etc.)")
    fmt.Println("✓ Easy customization and extension")
}
```

Custom log formatters provide precise control over output format, enabling  
integration with various systems and standards while maintaining flexibility  
for specific requirements and performance optimization.  

## Log streaming and real-time processing

Real-time log streaming enables live monitoring, alerting, and immediate  
response to application events.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "math/rand"
    "os"
    "strings"
    "sync"
    "time"
)

type LogStream struct {
    subscribers []chan LogEvent
    mu          sync.RWMutex
    buffer      []LogEvent
    maxBuffer   int
}

type LogEvent struct {
    Timestamp time.Time
    Level     string
    Source    string
    Message   string
    Fields    map[string]interface{}
}

func NewLogStream() *LogStream {
    return &LogStream{
        subscribers: make([]chan LogEvent, 0),
        buffer:      make([]LogEvent, 0),
        maxBuffer:   1000,
    }
}

func (ls *LogStream) Subscribe() <-chan LogEvent {
    ls.mu.Lock()
    defer ls.mu.Unlock()
    
    ch := make(chan LogEvent, 100) // Buffered channel
    ls.subscribers = append(ls.subscribers, ch)
    
    // Send recent events to new subscriber
    for _, event := range ls.buffer {
        select {
        case ch <- event:
        default:
            // Channel full, skip old events
        }
    }
    
    return ch
}

func (ls *LogStream) Unsubscribe(ch <-chan LogEvent) {
    ls.mu.Lock()
    defer ls.mu.Unlock()
    
    for i, subscriber := range ls.subscribers {
        if subscriber == ch {
            close(subscriber)
            ls.subscribers = append(ls.subscribers[:i], ls.subscribers[i+1:]...)
            break
        }
    }
}

func (ls *LogStream) Publish(event LogEvent) {
    ls.mu.Lock()
    defer ls.mu.Unlock()
    
    // Add to buffer
    ls.buffer = append(ls.buffer, event)
    if len(ls.buffer) > ls.maxBuffer {
        ls.buffer = ls.buffer[1:] // Remove oldest event
    }
    
    // Send to all subscribers
    for i := len(ls.subscribers) - 1; i >= 0; i-- {
        select {
        case ls.subscribers[i] <- event:
            // Event sent successfully
        default:
            // Subscriber channel full, remove it
            close(ls.subscribers[i])
            ls.subscribers = append(ls.subscribers[:i], ls.subscribers[i+1:]...)
        }
    }
}

func (ls *LogStream) GetBufferedEvents() []LogEvent {
    ls.mu.RLock()
    defer ls.mu.RUnlock()
    
    events := make([]LogEvent, len(ls.buffer))
    copy(events, ls.buffer)
    return events
}

// Streaming logger
type StreamingLogger struct {
    stream *LogStream
    source string
}

func NewStreamingLogger(stream *LogStream, source string) *StreamingLogger {
    return &StreamingLogger{
        stream: stream,
        source: source,
    }
}

func (sl *StreamingLogger) log(level, message string, fields map[string]interface{}) {
    event := LogEvent{
        Timestamp: time.Now(),
        Level:     level,
        Source:    sl.source,
        Message:   message,
        Fields:    fields,
    }
    
    sl.stream.Publish(event)
}

func (sl *StreamingLogger) Debug(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log("DEBUG", message, f)
}

func (sl *StreamingLogger) Info(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log("INFO", message, f)
}

func (sl *StreamingLogger) Warn(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log("WARN", message, f)
}

func (sl *StreamingLogger) Error(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log("ERROR", message, f)
}

// Real-time processors
type LogProcessor interface {
    Process(event LogEvent)
    Name() string
}

type ConsoleProcessor struct {
    name string
}

func NewConsoleProcessor(name string) *ConsoleProcessor {
    return &ConsoleProcessor{name: name}
}

func (cp *ConsoleProcessor) Name() string {
    return cp.name
}

func (cp *ConsoleProcessor) Process(event LogEvent) {
    timestamp := event.Timestamp.Format("15:04:05.000")
    fmt.Printf("[%s] %s [%s] %s: %s\n", 
        cp.name, timestamp, event.Source, event.Level, event.Message)
    
    if len(event.Fields) > 0 {
        for k, v := range event.Fields {
            fmt.Printf("  %s: %v\n", k, v)
        }
    }
}

type AlertProcessor struct {
    name      string
    threshold map[string]int
    counts    map[string]int
    window    time.Duration
    lastReset time.Time
}

func NewAlertProcessor(name string, window time.Duration) *AlertProcessor {
    return &AlertProcessor{
        name:      name,
        threshold: map[string]int{"ERROR": 5, "WARN": 10},
        counts:    make(map[string]int),
        window:    window,
        lastReset: time.Now(),
    }
}

func (ap *AlertProcessor) Name() string {
    return ap.name
}

func (ap *AlertProcessor) Process(event LogEvent) {
    now := time.Now()
    
    // Reset counts if window expired
    if now.Sub(ap.lastReset) >= ap.window {
        ap.counts = make(map[string]int)
        ap.lastReset = now
    }
    
    // Increment count for this level
    ap.counts[event.Level]++
    
    // Check thresholds
    if threshold, exists := ap.threshold[event.Level]; exists {
        if ap.counts[event.Level] >= threshold {
            fmt.Printf("🚨 ALERT [%s]: %s threshold exceeded (%d events in %v)\n",
                ap.name, event.Level, ap.counts[event.Level], ap.window)
            fmt.Printf("   Latest: %s from %s\n", event.Message, event.Source)
        }
    }
}

type FileProcessor struct {
    name     string
    filename string
    file     *os.File
}

func NewFileProcessor(name, filename string) (*FileProcessor, error) {
    file, err := os.OpenFile(filename, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        return nil, err
    }
    
    return &FileProcessor{
        name:     name,
        filename: filename,
        file:     file,
    }, nil
}

func (fp *FileProcessor) Name() string {
    return fp.name
}

func (fp *FileProcessor) Process(event LogEvent) {
    timestamp := event.Timestamp.Format(time.RFC3339)
    line := fmt.Sprintf("%s [%s] %s: %s\n", 
        timestamp, event.Source, event.Level, event.Message)
    
    fp.file.WriteString(line)
    fp.file.Sync() // Force write to disk
}

func (fp *FileProcessor) Close() error {
    return fp.file.Close()
}

// Stream manager
type StreamManager struct {
    stream     *LogStream
    processors []LogProcessor
    done       chan struct{}
    wg         sync.WaitGroup
}

func NewStreamManager(stream *LogStream) *StreamManager {
    return &StreamManager{
        stream:     stream,
        processors: make([]LogProcessor, 0),
        done:       make(chan struct{}),
    }
}

func (sm *StreamManager) AddProcessor(processor LogProcessor) {
    sm.processors = append(sm.processors, processor)
}

func (sm *StreamManager) Start() {
    subscription := sm.stream.Subscribe()
    
    sm.wg.Add(1)
    go func() {
        defer sm.wg.Done()
        
        for {
            select {
            case event := <-subscription:
                // Process event with all processors
                for _, processor := range sm.processors {
                    processor.Process(event)
                }
            case <-sm.done:
                sm.stream.Unsubscribe(subscription)
                return
            }
        }
    }()
}

func (sm *StreamManager) Stop() {
    close(sm.done)
    sm.wg.Wait()
}

// Simulate application components
func simulateWebServer(logger *StreamingLogger) {
    requests := []struct {
        path   string
        status int
        delay  time.Duration
    }{
        {"/api/users", 200, 50 * time.Millisecond},
        {"/api/login", 200, 120 * time.Millisecond},
        {"/api/data", 404, 20 * time.Millisecond},
        {"/api/payment", 500, 200 * time.Millisecond},
        {"/health", 200, 10 * time.Millisecond},
    }
    
    for i := 0; i < 10; i++ {
        req := requests[rand.Intn(len(requests))]
        
        logger.Info("HTTP request", map[string]interface{}{
            "method":   "GET",
            "path":     req.path,
            "status":   req.status,
            "duration": req.delay,
            "request_id": fmt.Sprintf("req-%d", i+1),
        })
        
        if req.status >= 400 {
            if req.status >= 500 {
                logger.Error("Server error", map[string]interface{}{
                    "path":   req.path,
                    "status": req.status,
                    "error":  "internal server error",
                })
            } else {
                logger.Warn("Client error", map[string]interface{}{
                    "path":   req.path,
                    "status": req.status,
                })
            }
        }
        
        time.Sleep(time.Duration(rand.Intn(300)) * time.Millisecond)
    }
}

func simulateDatabase(logger *StreamingLogger) {
    operations := []string{"SELECT", "INSERT", "UPDATE", "DELETE"}
    
    for i := 0; i < 8; i++ {
        op := operations[rand.Intn(len(operations))]
        duration := time.Duration(rand.Intn(100)+10) * time.Millisecond
        
        logger.Debug("Database operation", map[string]interface{}{
            "operation": op,
            "table":     "users",
            "duration":  duration,
            "rows":      rand.Intn(10) + 1,
        })
        
        // Simulate occasional database errors
        if rand.Float32() < 0.1 {
            logger.Error("Database timeout", map[string]interface{}{
                "operation": op,
                "timeout":   "5s",
                "retry":     true,
            })
        }
        
        time.Sleep(time.Duration(rand.Intn(500)) * time.Millisecond)
    }
}

func main() {
    fmt.Println("=== Log Streaming and Real-time Processing Demo ===")
    
    // Create log stream
    stream := NewLogStream()
    
    // Create processors
    consoleProcessor := NewConsoleProcessor("Console")
    alertProcessor := NewAlertProcessor("AlertManager", 10*time.Second)
    
    fileProcessor, err := NewFileProcessor("FileWriter", "/tmp/stream.log")
    if err != nil {
        log.Fatalf("Failed to create file processor: %v", err)
    }
    defer fileProcessor.Close()
    
    // Create stream manager
    manager := NewStreamManager(stream)
    manager.AddProcessor(consoleProcessor)
    manager.AddProcessor(alertProcessor)
    manager.AddProcessor(fileProcessor)
    
    // Start processing
    manager.Start()
    defer manager.Stop()
    
    // Create loggers for different components
    webLogger := NewStreamingLogger(stream, "web-server")
    dbLogger := NewStreamingLogger(stream, "database")
    
    fmt.Println("Starting real-time log streaming...")
    fmt.Println("Processors: Console, AlertManager, FileWriter")
    
    // Simulate concurrent activity
    var wg sync.WaitGroup
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        simulateWebServer(webLogger)
    }()
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        simulateDatabase(dbLogger)
    }()
    
    // Wait for simulation to complete
    wg.Wait()
    
    // Give processors time to handle remaining events
    time.Sleep(1 * time.Second)
    
    // Show stream statistics
    fmt.Println("\n=== Stream Statistics ===")
    bufferedEvents := stream.GetBufferedEvents()
    fmt.Printf("Buffered events: %d\n", len(bufferedEvents))
    
    levelCounts := make(map[string]int)
    sourceCounts := make(map[string]int)
    
    for _, event := range bufferedEvents {
        levelCounts[event.Level]++
        sourceCounts[event.Source]++
    }
    
    fmt.Println("Events by level:")
    for level, count := range levelCounts {
        fmt.Printf("  %s: %d\n", level, count)
    }
    
    fmt.Println("Events by source:")
    for source, count := range sourceCounts {
        fmt.Printf("  %s: %d\n", source, count)
    }
    
    fmt.Println("\n=== File Output ===")
    if content, err := os.ReadFile("/tmp/stream.log"); err == nil {
        lines := strings.Split(string(content), "\n")
        fmt.Printf("File contains %d lines\n", len(lines)-1)
        
        // Show first few lines
        for i, line := range lines {
            if i < 3 && strings.TrimSpace(line) != "" {
                fmt.Printf("  %s\n", line)
            }
        }
        if len(lines) > 4 {
            fmt.Printf("  ... and %d more lines\n", len(lines)-4)
        }
    }
    
    fmt.Println("\n=== Streaming Benefits ===")
    fmt.Println("✓ Real-time log processing")
    fmt.Println("✓ Multiple concurrent subscribers")
    fmt.Println("✓ Automatic alerting and notifications")
    fmt.Println("✓ Live monitoring and dashboards")
    fmt.Println("✓ Immediate response to critical events")
    fmt.Println("✓ Event buffering for new subscribers")
}
```

Log streaming enables real-time processing, monitoring, and alerting by  
providing live event feeds to multiple subscribers with automatic buffering  
and concurrent processing capabilities.  

## Logging middleware and interceptors

Middleware patterns enable transparent logging integration across application  
layers without modifying core business logic.  

```go
package main

import (
    "context"
    "fmt"
    "reflect"
    "runtime"
    "strings"
    "time"
)

type Logger interface {
    Log(level, message string, fields map[string]interface{})
}

type SimpleLogger struct{}

func (sl *SimpleLogger) Log(level, message string, fields map[string]interface{}) {
    timestamp := time.Now().Format("15:04:05.000")
    
    output := fmt.Sprintf("[%s] %s: %s", timestamp, level, message)
    if len(fields) > 0 {
        output += " |"
        for k, v := range fields {
            output += fmt.Sprintf(" %s=%v", k, v)
        }
    }
    fmt.Println(output)
}

// Function interceptor
type FunctionInterceptor struct {
    logger Logger
}

func NewFunctionInterceptor(logger Logger) *FunctionInterceptor {
    return &FunctionInterceptor{logger: logger}
}

func (fi *FunctionInterceptor) Intercept(fn interface{}, args ...interface{}) []reflect.Value {
    fnValue := reflect.ValueOf(fn)
    fnType := fnValue.Type()
    
    // Get function name
    fnName := runtime.FuncForPC(fnValue.Pointer()).Name()
    parts := strings.Split(fnName, ".")
    simpleName := parts[len(parts)-1]
    
    start := time.Now()
    
    // Log function entry
    fi.logger.Log("DEBUG", "Function entry", map[string]interface{}{
        "function": simpleName,
        "args":     fmt.Sprintf("%v", args),
    })
    
    // Convert arguments to reflect.Value slice
    reflectArgs := make([]reflect.Value, len(args))
    for i, arg := range args {
        reflectArgs[i] = reflect.ValueOf(arg)
    }
    
    // Call the function
    var results []reflect.Value
    var err interface{}
    
    func() {
        defer func() {
            if r := recover(); r != nil {
                err = r
            }
        }()
        results = fnValue.Call(reflectArgs)
    }()
    
    duration := time.Since(start)
    
    // Log function exit
    if err != nil {
        fi.logger.Log("ERROR", "Function panic", map[string]interface{}{
            "function": simpleName,
            "duration": duration,
            "error":    err,
        })
        panic(err) // Re-panic
    } else {
        level := "DEBUG"
        if duration > 100*time.Millisecond {
            level = "WARN"
        }
        
        fi.logger.Log(level, "Function exit", map[string]interface{}{
            "function": simpleName,
            "duration": duration,
            "results":  fmt.Sprintf("%v", results),
        })
    }
    
    return results
}

// Method interceptor for structs
type MethodInterceptor struct {
    logger Logger
    target interface{}
}

func NewMethodInterceptor(logger Logger, target interface{}) *MethodInterceptor {
    return &MethodInterceptor{
        logger: logger,
        target: target,
    }
}

func (mi *MethodInterceptor) Call(methodName string, args ...interface{}) []reflect.Value {
    targetValue := reflect.ValueOf(mi.target)
    method := targetValue.MethodByName(methodName)
    
    if !method.IsValid() {
        mi.logger.Log("ERROR", "Method not found", map[string]interface{}{
            "method": methodName,
            "type":   reflect.TypeOf(mi.target).String(),
        })
        return nil
    }
    
    start := time.Now()
    
    // Log method entry
    mi.logger.Log("DEBUG", "Method entry", map[string]interface{}{
        "method": methodName,
        "type":   reflect.TypeOf(mi.target).String(),
        "args":   fmt.Sprintf("%v", args),
    })
    
    // Convert arguments
    reflectArgs := make([]reflect.Value, len(args))
    for i, arg := range args {
        reflectArgs[i] = reflect.ValueOf(arg)
    }
    
    // Call method
    results := method.Call(reflectArgs)
    duration := time.Since(start)
    
    // Log method exit
    mi.logger.Log("DEBUG", "Method exit", map[string]interface{}{
        "method":   methodName,
        "duration": duration,
        "results":  fmt.Sprintf("%v", results),
    })
    
    return results
}

// Context-aware middleware
type ContextMiddleware struct {
    logger Logger
    next   func(ctx context.Context, req interface{}) (interface{}, error)
}

func NewContextMiddleware(logger Logger, next func(context.Context, interface{}) (interface{}, error)) *ContextMiddleware {
    return &ContextMiddleware{
        logger: logger,
        next:   next,
    }
}

func (cm *ContextMiddleware) Execute(ctx context.Context, req interface{}) (interface{}, error) {
    start := time.Now()
    
    // Extract context values for logging
    fields := map[string]interface{}{
        "request_type": reflect.TypeOf(req).String(),
    }
    
    if requestID := ctx.Value("request_id"); requestID != nil {
        fields["request_id"] = requestID
    }
    
    if userID := ctx.Value("user_id"); userID != nil {
        fields["user_id"] = userID
    }
    
    cm.logger.Log("INFO", "Request started", fields)
    
    // Execute next middleware/handler
    result, err := cm.next(ctx, req)
    
    duration := time.Since(start)
    fields["duration"] = duration
    
    if err != nil {
        fields["error"] = err.Error()
        cm.logger.Log("ERROR", "Request failed", fields)
    } else {
        level := "INFO"
        if duration > 500*time.Millisecond {
            level = "WARN"
            fields["slow_request"] = true
        }
        cm.logger.Log(level, "Request completed", fields)
    }
    
    return result, err
}

// Database interceptor
type DatabaseInterceptor struct {
    logger Logger
    db     DatabaseInterface
}

type DatabaseInterface interface {
    Query(sql string, args ...interface{}) (interface{}, error)
    Execute(sql string, args ...interface{}) error
}

type MockDatabase struct{}

func (md *MockDatabase) Query(sql string, args ...interface{}) (interface{}, error) {
    // Simulate query execution
    time.Sleep(time.Duration(len(sql)) * time.Millisecond)
    
    if strings.Contains(strings.ToLower(sql), "error") {
        return nil, fmt.Errorf("database error: table not found")
    }
    
    return map[string]interface{}{
        "rows": []map[string]interface{}{
            {"id": 1, "name": "Alice"},
            {"id": 2, "name": "Bob"},
        },
    }, nil
}

func (md *MockDatabase) Execute(sql string, args ...interface{}) error {
    time.Sleep(time.Duration(len(sql)) * time.Millisecond)
    
    if strings.Contains(strings.ToLower(sql), "error") {
        return fmt.Errorf("database error: constraint violation")
    }
    
    return nil
}

func NewDatabaseInterceptor(logger Logger, db DatabaseInterface) *DatabaseInterceptor {
    return &DatabaseInterceptor{
        logger: logger,
        db:     db,
    }
}

func (di *DatabaseInterceptor) Query(sql string, args ...interface{}) (interface{}, error) {
    start := time.Now()
    
    // Clean and log SQL
    cleanSQL := strings.ReplaceAll(strings.TrimSpace(sql), "\n", " ")
    cleanSQL = strings.Join(strings.Fields(cleanSQL), " ")
    
    fields := map[string]interface{}{
        "sql":  cleanSQL,
        "type": "query",
    }
    
    if len(args) > 0 {
        fields["args"] = fmt.Sprintf("%v", args)
    }
    
    di.logger.Log("DEBUG", "Database query", fields)
    
    result, err := di.db.Query(sql, args...)
    
    duration := time.Since(start)
    fields["duration"] = duration
    
    if err != nil {
        fields["error"] = err.Error()
        di.logger.Log("ERROR", "Query failed", fields)
    } else {
        level := "DEBUG"
        if duration > 100*time.Millisecond {
            level = "WARN"
            fields["slow_query"] = true
        }
        di.logger.Log(level, "Query completed", fields)
    }
    
    return result, err
}

func (di *DatabaseInterceptor) Execute(sql string, args ...interface{}) error {
    start := time.Now()
    
    cleanSQL := strings.ReplaceAll(strings.TrimSpace(sql), "\n", " ")
    cleanSQL = strings.Join(strings.Fields(cleanSQL), " ")
    
    fields := map[string]interface{}{
        "sql":  cleanSQL,
        "type": "execute",
    }
    
    if len(args) > 0 {
        fields["args"] = fmt.Sprintf("%v", args)
    }
    
    di.logger.Log("DEBUG", "Database execute", fields)
    
    err := di.db.Execute(sql, args...)
    
    duration := time.Since(start)
    fields["duration"] = duration
    
    if err != nil {
        fields["error"] = err.Error()
        di.logger.Log("ERROR", "Execute failed", fields)
    } else {
        di.logger.Log("DEBUG", "Execute completed", fields)
    }
    
    return err
}

// Business logic examples
type UserService struct {
    Name string
}

func (us *UserService) GetUser(id int) (map[string]interface{}, error) {
    if id <= 0 {
        return nil, fmt.Errorf("invalid user ID: %d", id)
    }
    
    // Simulate processing time
    time.Sleep(50 * time.Millisecond)
    
    return map[string]interface{}{
        "id":    id,
        "name":  fmt.Sprintf("User %d", id),
        "email": fmt.Sprintf("user%d@example.com", id),
    }, nil
}

func (us *UserService) CreateUser(name, email string) error {
    if name == "" {
        return fmt.Errorf("name cannot be empty")
    }
    
    // Simulate processing time
    time.Sleep(100 * time.Millisecond)
    
    return nil
}

func businessFunction(a, b int) (int, error) {
    time.Sleep(20 * time.Millisecond)
    
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    
    return a / b, nil
}

func demonstrateMiddleware() {
    logger := &SimpleLogger{}
    
    fmt.Println("=== Logging Middleware and Interceptors Demo ===")
    
    // Function interception
    fmt.Println("\n--- Function Interception ---")
    interceptor := NewFunctionInterceptor(logger)
    
    results := interceptor.Intercept(businessFunction, 10, 2)
    fmt.Printf("Function result: %v\n", results[0].Int())
    
    // Method interception
    fmt.Println("\n--- Method Interception ---")
    userService := &UserService{Name: "UserService"}
    methodInterceptor := NewMethodInterceptor(logger, userService)
    
    results = methodInterceptor.Call("GetUser", 123)
    if len(results) > 0 && results[0].Kind() == reflect.Map {
        user := results[0].Interface().(map[string]interface{})
        fmt.Printf("Method result: %v\n", user)
    }
    
    // Context middleware
    fmt.Println("\n--- Context Middleware ---")
    
    businessHandler := func(ctx context.Context, req interface{}) (interface{}, error) {
        time.Sleep(150 * time.Millisecond)
        
        if req == "error" {
            return nil, fmt.Errorf("simulated error")
        }
        
        return fmt.Sprintf("Processed: %v", req), nil
    }
    
    contextMiddleware := NewContextMiddleware(logger, businessHandler)
    
    ctx := context.Background()
    ctx = context.WithValue(ctx, "request_id", "req-12345")
    ctx = context.WithValue(ctx, "user_id", "user-789")
    
    result, err := contextMiddleware.Execute(ctx, "test request")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Result: %v\n", result)
    }
    
    // Database interception
    fmt.Println("\n--- Database Interception ---")
    mockDB := &MockDatabase{}
    dbInterceptor := NewDatabaseInterceptor(logger, mockDB)
    
    // Test queries
    queries := []struct {
        sql  string
        args []interface{}
    }{
        {"SELECT id, name FROM users WHERE active = ?", []interface{}{true}},
        {"INSERT INTO users (name, email) VALUES (?, ?)", []interface{}{"Alice", "alice@example.com"}},
        {"SELECT * FROM error_table", nil}, // This will cause an error
    }
    
    for _, q := range queries {
        if strings.Contains(strings.ToLower(q.sql), "select") {
            result, err := dbInterceptor.Query(q.sql, q.args...)
            if err != nil {
                fmt.Printf("Query error: %v\n", err)
            } else {
                fmt.Printf("Query result: %v\n", result)
            }
        } else {
            err := dbInterceptor.Execute(q.sql, q.args...)
            if err != nil {
                fmt.Printf("Execute error: %v\n", err)
            } else {
                fmt.Println("Execute succeeded")
            }
        }
    }
}

func main() {
    demonstrateMiddleware()
    
    fmt.Println("\n=== Middleware Benefits ===")
    fmt.Println("✓ Transparent logging integration")
    fmt.Println("✓ No modification of business logic")
    fmt.Println("✓ Consistent logging across layers")
    fmt.Println("✓ Performance monitoring")
    fmt.Println("✓ Error tracking and debugging")
    fmt.Println("✓ Context propagation")
    fmt.Println("✓ Centralized logging concerns")
}
```

Logging middleware and interceptors provide transparent integration across  
application layers, enabling consistent logging without modifying business  
logic while maintaining performance monitoring and error tracking capabilities.  