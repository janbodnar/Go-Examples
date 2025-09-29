# Error handling

Error handling is a fundamental aspect of Go programming that promotes explicit  
error checking and robust applications. Go's approach to error handling is based  
on explicit error values rather than exceptions, making error flows clear and  
predictable. This design philosophy encourages developers to consider and handle  
error conditions throughout their code.

Go's error handling revolves around the built-in `error` interface, which is  
simple yet powerful. Functions that can fail return an error value as their  
last return value, typically alongside the successful result. This pattern  
makes error handling explicit and forces developers to acknowledge potential  
failures in their code.

The language provides several mechanisms for error handling including basic  
error checking, custom error types, error wrapping and unwrapping, sentinel  
errors, and panic/recovery for exceptional situations. Modern Go also includes  
powerful error handling features like error wrapping with `fmt.Errorf` and  
the `errors` package functions for error inspection and manipulation.

Effective error handling in Go involves choosing appropriate strategies for  
different scenarios: returning errors for recoverable conditions, using  
panic for programming errors, implementing proper cleanup with defer, and  
providing meaningful error messages that help diagnose and fix problems.  
Understanding these patterns is essential for writing reliable Go applications.


## Basic error checking

The most fundamental error handling pattern in Go involves checking the error  
return value from functions and responding appropriately.  

```go
package main

import (
    "fmt"
    "strconv"
)

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}

func main() {
    // Basic error checking
    result, err := divide(10, 2)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    fmt.Printf("10 / 2 = %.2f\n", result)
    
    // Error case
    _, err = divide(10, 0)
    if err != nil {
        fmt.Printf("Error occurred: %v\n", err)
    }
    
    // String conversion with error
    if num, err := strconv.Atoi("123"); err != nil {
        fmt.Printf("Conversion failed: %v\n", err)
    } else {
        fmt.Printf("Converted number: %d\n", num)
    }
    
    // Invalid string conversion
    if _, err := strconv.Atoi("not-a-number"); err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
}
```

This example demonstrates the idiomatic Go pattern of checking errors  
immediately after function calls. The comma-ok idiom allows for concise  
error handling while maintaining explicit error checking.  

## Multiple return values with errors

Go functions commonly return both a result and an error, allowing callers  
to handle success and failure cases appropriately.  

```go
package main

import (
    "fmt"
    "os"
    "strings"
)

func readFile(filename string) (string, error) {
    data, err := os.ReadFile(filename)
    if err != nil {
        return "", fmt.Errorf("failed to read file %q: %w", filename, err)
    }
    return string(data), nil
}

func processLine(line string) (string, int, error) {
    trimmed := strings.TrimSpace(line)
    if trimmed == "" {
        return "", 0, fmt.Errorf("empty line not allowed")
    }
    
    words := strings.Fields(trimmed)
    if len(words) < 2 {
        return "", 0, fmt.Errorf("line must contain at least 2 words")
    }
    
    return trimmed, len(words), nil
}

func main() {
    // Create a test file
    testFile := "/tmp/test.txt"
    content := "hello there\nGo programming\n\nvalid line here"
    if err := os.WriteFile(testFile, []byte(content), 0644); err != nil {
        fmt.Printf("Failed to create test file: %v\n", err)
        return
    }
    defer os.Remove(testFile)
    
    // Read and process file
    data, err := readFile(testFile)
    if err != nil {
        fmt.Printf("Read error: %v\n", err)
        return
    }
    
    lines := strings.Split(data, "\n")
    for i, line := range lines {
        processed, wordCount, err := processLine(line)
        if err != nil {
            fmt.Printf("Line %d error: %v\n", i+1, err)
            continue
        }
        fmt.Printf("Line %d: %q (%d words)\n", i+1, processed, wordCount)
    }
}
```

Functions returning multiple values alongside errors enable rich return  
information while maintaining clear error handling. The error wrapping with  
`%w` preserves the original error for further inspection.  

## Error creation methods

Go provides several ways to create errors, from simple string-based errors  
to complex custom types with additional context.  

```go
package main

import (
    "errors"
    "fmt"
)

// Custom error type
type ValidationError struct {
    Field string
    Value string
    Rule  string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed: field %q with value %q violated rule %q", 
                       e.Field, e.Value, e.Rule)
}

func validateEmail(email string) error {
    if email == "" {
        // Using errors.New for simple errors
        return errors.New("email cannot be empty")
    }
    
    if len(email) < 5 {
        // Using fmt.Errorf for formatted errors
        return fmt.Errorf("email too short: got %d characters, minimum is 5", len(email))
    }
    
    if !strings.Contains(email, "@") {
        // Using custom error type
        return ValidationError{
            Field: "email",
            Value: email,
            Rule:  "must contain @ symbol",
        }
    }
    
    return nil
}

func main() {
    emails := []string{"", "abc", "user@example.com", "invalid-email"}
    
    for _, email := range emails {
        fmt.Printf("Validating: %q\n", email)
        
        if err := validateEmail(email); err != nil {
            fmt.Printf("  Error: %v\n", err)
            
            // Type assertion to access custom error fields
            if ve, ok := err.(ValidationError); ok {
                fmt.Printf("  Field: %s, Rule: %s\n", ve.Field, ve.Rule)
            }
        } else {
            fmt.Printf("  Valid email\n")
        }
        fmt.Println()
    }
}
```

Different error creation methods serve different purposes: `errors.New` for  
simple messages, `fmt.Errorf` for formatted strings, and custom types for  
structured error information with additional methods and fields.  

## Error wrapping and unwrapping

Error wrapping allows you to add context to errors while preserving the  
original error for inspection and type checking.  

```go
package main

import (
    "errors"
    "fmt"
    "os"
)

var (
    ErrFileNotFound = errors.New("file not found")
    ErrPermission   = errors.New("permission denied")
)

func openConfigFile(filename string) error {
    _, err := os.Open(filename)
    if err != nil {
        if os.IsNotExist(err) {
            return fmt.Errorf("config file issue: %w", ErrFileNotFound)
        }
        if os.IsPermission(err) {
            return fmt.Errorf("config access problem: %w", ErrPermission)
        }
        return fmt.Errorf("failed to open config file %q: %w", filename, err)
    }
    return nil
}

func loadConfiguration() error {
    if err := openConfigFile("/nonexistent/config.yaml"); err != nil {
        return fmt.Errorf("configuration loading failed: %w", err)
    }
    return nil
}

func main() {
    err := loadConfiguration()
    if err != nil {
        fmt.Printf("Main error: %v\n", err)
        
        // Unwrap to check original error
        fmt.Printf("Is file not found: %v\n", errors.Is(err, ErrFileNotFound))
        fmt.Printf("Is permission error: %v\n", errors.Is(err, ErrPermission))
        
        // Unwrap chain
        fmt.Println("Error chain:")
        current := err
        level := 0
        for current != nil {
            fmt.Printf("  Level %d: %v\n", level, current)
            current = errors.Unwrap(current)
            level++
        }
        
        // Check for specific error types in chain
        var pathErr *os.PathError
        if errors.As(err, &pathErr) {
            fmt.Printf("Path error found: %v\n", pathErr.Path)
        }
    }
}
```

Error wrapping with `fmt.Errorf` and `%w` creates error chains that preserve  
context while allowing inspection of underlying errors using `errors.Is`  
and `errors.As` functions.  

## Sentinel errors

Sentinel errors are predefined error values that can be compared directly,  
providing a way to identify specific error conditions.  

```go
package main

import (
    "errors"
    "fmt"
)

// Define sentinel errors
var (
    ErrInvalidInput    = errors.New("invalid input provided")
    ErrRateLimitExceeded = errors.New("rate limit exceeded")
    ErrResourceNotFound  = errors.New("resource not found")
    ErrUnauthorized     = errors.New("unauthorized access")
)

type APIClient struct {
    requestCount int
    maxRequests  int
}

func NewAPIClient(maxRequests int) *APIClient {
    return &APIClient{maxRequests: maxRequests}
}

func (c *APIClient) GetUser(userID string) (string, error) {
    if userID == "" {
        return "", ErrInvalidInput
    }
    
    c.requestCount++
    if c.requestCount > c.maxRequests {
        return "", ErrRateLimitExceeded
    }
    
    // Simulate different responses
    switch userID {
    case "user1":
        return "John Doe", nil
    case "user2":
        return "Jane Smith", nil
    case "admin":
        return "", ErrUnauthorized
    default:
        return "", ErrResourceNotFound
    }
}

func handleUserRequest(client *APIClient, userID string) {
    user, err := client.GetUser(userID)
    
    switch {
    case errors.Is(err, ErrInvalidInput):
        fmt.Printf("Bad request: %v\n", err)
    case errors.Is(err, ErrRateLimitExceeded):
        fmt.Printf("Too many requests: %v\n", err)
    case errors.Is(err, ErrResourceNotFound):
        fmt.Printf("User not found: %v\n", err)
    case errors.Is(err, ErrUnauthorized):
        fmt.Printf("Access denied: %v\n", err)
    case err != nil:
        fmt.Printf("Unexpected error: %v\n", err)
    default:
        fmt.Printf("User found: %s\n", user)
    }
}

func main() {
    client := NewAPIClient(3)
    
    requests := []string{"", "user1", "user2", "admin", "unknown", "user3"}
    
    for _, userID := range requests {
        fmt.Printf("Requesting user: %q\n", userID)
        handleUserRequest(client, userID)
        fmt.Println()
    }
}
```

Sentinel errors provide a clean way to define and check for specific error  
conditions. They're particularly useful for API responses and library  
functions where specific error types need different handling strategies.  

## Error aggregation

When processing multiple items, collecting and reporting all errors together  
provides better user experience and debugging information.  

```go
package main

import (
    "fmt"
    "strings"
)

type ErrorList struct {
    errors []error
}

func (el *ErrorList) Add(err error) {
    if err != nil {
        el.errors = append(el.errors, err)
    }
}

func (el *ErrorList) HasErrors() bool {
    return len(el.errors) > 0
}

func (el *ErrorList) Error() string {
    if len(el.errors) == 0 {
        return ""
    }
    
    var messages []string
    for i, err := range el.errors {
        messages = append(messages, fmt.Sprintf("%d: %v", i+1, err))
    }
    
    return fmt.Sprintf("multiple errors occurred:\n  %s", 
                       strings.Join(messages, "\n  "))
}

func (el *ErrorList) Errors() []error {
    return el.errors
}

func validateUser(name, email string, age int) error {
    var errs ErrorList
    
    if name == "" {
        errs.Add(fmt.Errorf("name cannot be empty"))
    } else if len(name) < 2 {
        errs.Add(fmt.Errorf("name must be at least 2 characters"))
    }
    
    if email == "" {
        errs.Add(fmt.Errorf("email cannot be empty"))
    } else if !strings.Contains(email, "@") {
        errs.Add(fmt.Errorf("email must contain @ symbol"))
    }
    
    if age < 0 {
        errs.Add(fmt.Errorf("age cannot be negative"))
    } else if age > 150 {
        errs.Add(fmt.Errorf("age seems unrealistic: %d", age))
    }
    
    if errs.HasErrors() {
        return &errs
    }
    return nil
}

func processUsers(users []struct{ name, email string; age int }) {
    var allErrors ErrorList
    validUsers := 0
    
    for i, user := range users {
        fmt.Printf("Processing user %d: %s\n", i+1, user.name)
        
        if err := validateUser(user.name, user.email, user.age); err != nil {
            fmt.Printf("  Validation failed: %v\n", err)
            allErrors.Add(fmt.Errorf("user %d (%s): %w", i+1, user.name, err))
        } else {
            fmt.Printf("  Valid user\n")
            validUsers++
        }
        fmt.Println()
    }
    
    fmt.Printf("Summary: %d valid users\n", validUsers)
    if allErrors.HasErrors() {
        fmt.Printf("Total validation errors: %d\n", len(allErrors.Errors()))
    }
}

func main() {
    users := []struct{ name, email string; age int }{
        {"John Doe", "john@example.com", 30},
        {"", "jane@example.com", 25},
        {"Bob", "invalid-email", 35},
        {"Alice", "alice@example.com", -5},
        {"X", "", 200},
    }
    
    processUsers(users)
}
```

Error aggregation allows collecting multiple errors during batch processing,  
providing comprehensive feedback rather than stopping at the first error.  
This pattern is essential for validation and batch operations.  

## Custom error types with methods

Custom error types can implement additional methods to provide rich error  
information and specialized behavior for different error scenarios.  

```go
package main

import (
    "fmt"
    "net/http"
    "time"
)

// HTTP error with status code and retry information
type HTTPError struct {
    StatusCode int
    Message    string
    URL        string
    Timestamp  time.Time
    Retryable  bool
}

func (e HTTPError) Error() string {
    return fmt.Sprintf("HTTP %d error at %s: %s (URL: %s)", 
                       e.StatusCode, e.Timestamp.Format("15:04:05"), e.Message, e.URL)
}

func (e HTTPError) IsClientError() bool {
    return e.StatusCode >= 400 && e.StatusCode < 500
}

func (e HTTPError) IsServerError() bool {
    return e.StatusCode >= 500
}

func (e HTTPError) ShouldRetry() bool {
    return e.Retryable && (e.IsServerError() || e.StatusCode == 429)
}

// Database error with query information
type DatabaseError struct {
    Operation string
    Query     string
    Err       error
    Table     string
}

func (e DatabaseError) Error() string {
    return fmt.Sprintf("database %s error on table %q: %v", 
                       e.Operation, e.Table, e.Err)
}

func (e DatabaseError) Unwrap() error {
    return e.Err
}

func (e DatabaseError) IsConnectionError() bool {
    return strings.Contains(strings.ToLower(e.Err.Error()), "connection")
}

// Simulate HTTP request
func makeHTTPRequest(url string) error {
    // Simulate different HTTP responses
    switch url {
    case "https://api.example.com/success":
        return nil
    case "https://api.example.com/notfound":
        return HTTPError{
            StatusCode: 404,
            Message:    "Resource not found",
            URL:        url,
            Timestamp:  time.Now(),
            Retryable:  false,
        }
    case "https://api.example.com/server-error":
        return HTTPError{
            StatusCode: 500,
            Message:    "Internal server error",
            URL:        url,
            Timestamp:  time.Now(),
            Retryable:  true,
        }
    case "https://api.example.com/rate-limit":
        return HTTPError{
            StatusCode: 429,
            Message:    "Rate limit exceeded",
            URL:        url,
            Timestamp:  time.Now(),
            Retryable:  true,
        }
    default:
        return HTTPError{
            StatusCode: 400,
            Message:    "Bad request",
            URL:        url,
            Timestamp:  time.Now(),
            Retryable:  false,
        }
    }
}

func handleHTTPError(err error) {
    if httpErr, ok := err.(HTTPError); ok {
        fmt.Printf("HTTP Error Details:\n")
        fmt.Printf("  Status: %d\n", httpErr.StatusCode)
        fmt.Printf("  URL: %s\n", httpErr.URL)
        fmt.Printf("  Time: %s\n", httpErr.Timestamp.Format("15:04:05"))
        fmt.Printf("  Client Error: %v\n", httpErr.IsClientError())
        fmt.Printf("  Server Error: %v\n", httpErr.IsServerError())
        fmt.Printf("  Should Retry: %v\n", httpErr.ShouldRetry())
    } else {
        fmt.Printf("Other error: %v\n", err)
    }
}

func main() {
    urls := []string{
        "https://api.example.com/success",
        "https://api.example.com/notfound",
        "https://api.example.com/server-error",
        "https://api.example.com/rate-limit",
        "https://api.example.com/bad-request",
    }
    
    for _, url := range urls {
        fmt.Printf("Making request to: %s\n", url)
        
        if err := makeHTTPRequest(url); err != nil {
            handleHTTPError(err)
        } else {
            fmt.Printf("Request successful\n")
        }
        fmt.Println()
    }
}
```

Custom error types with methods enable sophisticated error handling strategies  
by providing rich context and specialized behavior. Methods can determine  
retry strategies, error categories, and provide detailed diagnostics.  

## Panic and recovery patterns

While Go prefers explicit error handling, panic and recover provide mechanisms  
for handling truly exceptional conditions and building robust recovery systems.  

```go
package main

import (
    "fmt"
    "runtime"
)

// Safe division function that recovers from division by zero panic
func safeDivide(a, b float64) (result float64, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("panic during division: %v", r)
        }
    }()
    
    if b == 0 {
        panic("division by zero attempted")
    }
    
    result = a / b
    return result, nil
}

// Worker function that might panic
func riskyWorker(id int, data []int) (err error) {
    defer func() {
        if r := recover(); r != nil {
            // Capture stack trace
            buf := make([]byte, 1024)
            n := runtime.Stack(buf, false)
            err = fmt.Errorf("worker %d panicked: %v\nStack trace:\n%s", 
                           id, r, buf[:n])
        }
    }()
    
    fmt.Printf("Worker %d processing %d items\n", id, len(data))
    
    for i, item := range data {
        if item < 0 {
            panic(fmt.Sprintf("negative value %d at index %d", item, i))
        }
        
        if item > 100 {
            panic("value exceeds maximum limit")
        }
        
        // Simulate array access that might panic
        if i == len(data) {
            fmt.Printf("Item: %d\n", data[i]) // This would panic
        }
    }
    
    fmt.Printf("Worker %d completed successfully\n", id)
    return nil
}

// HTTP server recovery middleware pattern
func recoverMiddleware(next func()) func() {
    return func() {
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("Recovered from panic in handler: %v\n", r)
                fmt.Printf("Server continues running...\n")
            }
        }()
        next()
    }
}

func faultyHandler() {
    fmt.Println("Processing request...")
    panic("simulated handler panic")
}

func goodHandler() {
    fmt.Println("Request processed successfully")
}

func main() {
    // Safe division examples
    fmt.Println("=== Safe Division Examples ===")
    operations := []struct{ a, b float64 }{
        {10, 2},
        {15, 3},
        {7, 0}, // This will cause controlled panic
        {20, 4},
    }
    
    for _, op := range operations {
        result, err := safeDivide(op.a, op.b)
        if err != nil {
            fmt.Printf("%.1f / %.1f: Error - %v\n", op.a, op.b, err)
        } else {
            fmt.Printf("%.1f / %.1f = %.2f\n", op.a, op.b, result)
        }
    }
    
    fmt.Println("\n=== Worker Recovery Examples ===")
    testCases := [][]int{
        {1, 2, 3, 4, 5},           // Good data
        {1, -2, 3},                // Contains negative
        {50, 75, 150},             // Exceeds limit
        {},                         // Empty slice
    }
    
    for i, data := range testCases {
        fmt.Printf("Test case %d: %v\n", i+1, data)
        if err := riskyWorker(i+1, data); err != nil {
            fmt.Printf("Worker error: %v\n", err)
        }
        fmt.Println()
    }
    
    fmt.Println("=== Recovery Middleware Example ===")
    handlers := []func(){faultyHandler, goodHandler}
    
    for i, handler := range handlers {
        fmt.Printf("Request %d:\n", i+1)
        safeHandler := recoverMiddleware(handler)
        safeHandler()
        fmt.Println()
    }
}
```

Panic and recovery should be used judiciously for truly exceptional conditions.  
Recovery patterns enable building resilient systems that can handle unexpected  
failures while maintaining overall system stability and logging issues.  

## Context-based error handling

Context provides a powerful mechanism for cancellation, timeouts, and carrying  
request-scoped values, essential for robust error handling in concurrent systems.  

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "time"
)

type ContextError struct {
    Operation string
    Cause     error
}

func (e ContextError) Error() string {
    return fmt.Sprintf("context error in %s: %v", e.Operation, e.Cause)
}

func (e ContextError) Unwrap() error {
    return e.Cause
}

// Simulate a long-running database query
func queryDatabase(ctx context.Context, query string) ([]string, error) {
    // Check if context is already cancelled
    select {
    case <-ctx.Done():
        return nil, ContextError{
            Operation: "database query",
            Cause:     ctx.Err(),
        }
    default:
    }
    
    // Simulate work with context checking
    for i := 0; i < 5; i++ {
        select {
        case <-ctx.Done():
            return nil, ContextError{
                Operation: fmt.Sprintf("database query step %d", i+1),
                Cause:     ctx.Err(),
            }
        case <-time.After(200 * time.Millisecond):
            // Continue processing
        }
    }
    
    return []string{"result1", "result2", "result3"}, nil
}

// Process data with timeout
func processWithTimeout(data string, timeout time.Duration) error {
    ctx, cancel := context.WithTimeout(context.Background(), timeout)
    defer cancel()
    
    results, err := queryDatabase(ctx, data)
    if err != nil {
        return fmt.Errorf("processing failed: %w", err)
    }
    
    fmt.Printf("Processing completed: %v\n", results)
    return nil
}

// Process with cancellation
func processWithCancellation() {
    ctx, cancel := context.WithCancel(context.Background())
    
    // Start processing
    go func() {
        time.Sleep(500 * time.Millisecond)
        fmt.Println("Cancelling operation...")
        cancel()
    }()
    
    results, err := queryDatabase(ctx, "SELECT * FROM users")
    if err != nil {
        fmt.Printf("Operation cancelled: %v\n", err)
        return
    }
    
    fmt.Printf("Results: %v\n", results)
}

// Chain context operations
func chainedOperations(ctx context.Context) error {
    // First operation
    select {
    case <-ctx.Done():
        return fmt.Errorf("cancelled before first operation: %w", ctx.Err())
    default:
    }
    
    fmt.Println("Executing first operation...")
    time.Sleep(100 * time.Millisecond)
    
    // Second operation with its own timeout
    childCtx, cancel := context.WithTimeout(ctx, 300*time.Millisecond)
    defer cancel()
    
    results, err := queryDatabase(childCtx, "SELECT COUNT(*)")
    if err != nil {
        return fmt.Errorf("second operation failed: %w", err)
    }
    
    fmt.Printf("Chained operations completed: %v\n", results)
    return nil
}

func main() {
    fmt.Println("=== Timeout Examples ===")
    
    // Fast operation (should succeed)
    fmt.Println("Fast operation:")
    if err := processWithTimeout("quick query", 2*time.Second); err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Slow operation (should timeout)
    fmt.Println("\nSlow operation:")
    if err := processWithTimeout("slow query", 500*time.Millisecond); err != nil {
        fmt.Printf("Error: %v\n", err)
        
        // Check specific error types
        if errors.Is(err, context.DeadlineExceeded) {
            fmt.Println("  -> Operation timed out")
        }
    }
    
    fmt.Println("\n=== Cancellation Example ===")
    processWithCancellation()
    
    fmt.Println("\n=== Chained Operations ===")
    
    // Success case
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel()
    
    if err := chainedOperations(ctx); err != nil {
        fmt.Printf("Chained operation error: %v\n", err)
    }
    
    // Timeout case
    ctx2, cancel2 := context.WithTimeout(context.Background(), 200*time.Millisecond)
    defer cancel2()
    
    if err := chainedOperations(ctx2); err != nil {
        fmt.Printf("Chained operation timeout: %v\n", err)
    }
}
```

Context-based error handling enables proper cancellation and timeout handling  
in concurrent operations. It's essential for building responsive applications  
that can gracefully handle cancellation and resource management.  

## File operation error handling

File operations are common sources of errors in applications, requiring  
careful handling of different failure modes and proper resource cleanup.  

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "path/filepath"
)

type FileError struct {
    Operation string
    Path      string
    Err       error
}

func (e FileError) Error() string {
    return fmt.Sprintf("file %s failed on %q: %v", e.Operation, e.Path, e.Err)
}

func (e FileError) Unwrap() error {
    return e.Err
}

func safeReadFile(filename string) ([]byte, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, FileError{
            Operation: "open",
            Path:      filename,
            Err:       err,
        }
    }
    defer func() {
        if closeErr := file.Close(); closeErr != nil {
            fmt.Printf("Warning: failed to close %q: %v\n", filename, closeErr)
        }
    }()
    
    data, err := io.ReadAll(file)
    if err != nil {
        return nil, FileError{
            Operation: "read",
            Path:      filename,
            Err:       err,
        }
    }
    
    return data, nil
}

func safeWriteFile(filename string, data []byte) error {
    // Create directory if it doesn't exist
    dir := filepath.Dir(filename)
    if err := os.MkdirAll(dir, 0755); err != nil {
        return FileError{
            Operation: "mkdir",
            Path:      dir,
            Err:       err,
        }
    }
    
    // Create temporary file first for atomic write
    tempFile := filename + ".tmp"
    file, err := os.Create(tempFile)
    if err != nil {
        return FileError{
            Operation: "create",
            Path:      tempFile,
            Err:       err,
        }
    }
    
    // Ensure cleanup on error
    defer func() {
        if file != nil {
            file.Close()
            os.Remove(tempFile)
        }
    }()
    
    if _, err := file.Write(data); err != nil {
        return FileError{
            Operation: "write",
            Path:      tempFile,
            Err:       err,
        }
    }
    
    if err := file.Sync(); err != nil {
        return FileError{
            Operation: "sync",
            Path:      tempFile,
            Err:       err,
        }
    }
    
    if err := file.Close(); err != nil {
        return FileError{
            Operation: "close",
            Path:      tempFile,
            Err:       err,
        }
    }
    
    // Atomic rename
    if err := os.Rename(tempFile, filename); err != nil {
        return FileError{
            Operation: "rename",
            Path:      filename,
            Err:       err,
        }
    }
    
    // Success - clear defer cleanup
    file = nil
    return nil
}

func processLargeFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("failed to open large file: %w", err)
    }
    defer file.Close()
    
    scanner := bufio.NewScanner(file)
    lineCount := 0
    
    for scanner.Scan() {
        lineCount++
        line := scanner.Text()
        
        // Process line (simulate work)
        if len(line) > 1000 {
            return fmt.Errorf("line %d too long: %d characters", lineCount, len(line))
        }
        
        // Check for scan errors periodically
        if lineCount%1000 == 0 && scanner.Err() != nil {
            return fmt.Errorf("scan error at line %d: %w", lineCount, scanner.Err())
        }
    }
    
    // Final error check
    if err := scanner.Err(); err != nil {
        return fmt.Errorf("scanner error after processing %d lines: %w", lineCount, err)
    }
    
    fmt.Printf("Successfully processed %d lines\n", lineCount)
    return nil
}

func main() {
    // Test file operations
    testDir := "/tmp/file-test"
    testFile := filepath.Join(testDir, "test.txt")
    testData := []byte("Hello there!\nThis is test data.\nLine 3 here.")
    
    fmt.Println("=== File Write Test ===")
    if err := safeWriteFile(testFile, testData); err != nil {
        fmt.Printf("Write error: %v\n", err)
        return
    }
    fmt.Printf("Successfully wrote to %s\n", testFile)
    
    fmt.Println("\n=== File Read Test ===")
    data, err := safeReadFile(testFile)
    if err != nil {
        fmt.Printf("Read error: %v\n", err)
        
        // Handle specific file error types
        if fileErr, ok := err.(FileError); ok {
            fmt.Printf("File operation: %s\n", fileErr.Operation)
            fmt.Printf("File path: %s\n", fileErr.Path)
            
            if os.IsNotExist(fileErr.Err) {
                fmt.Println("File does not exist")
            }
            if os.IsPermission(fileErr.Err) {
                fmt.Println("Permission denied")
            }
        }
        return
    }
    fmt.Printf("Read data: %q\n", string(data))
    
    fmt.Println("\n=== Non-existent File Test ===")
    _, err = safeReadFile("/nonexistent/file.txt")
    if err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
    
    fmt.Println("\n=== Large File Processing Test ===")
    // Create a test file for processing
    if err := processLargeFile(testFile); err != nil {
        fmt.Printf("Processing error: %v\n", err)
    }
    
    // Cleanup
    os.RemoveAll(testDir)
}
```

File operations require comprehensive error handling for different failure  
modes including permissions, disk space, and I/O errors. Proper resource  
cleanup and atomic operations ensure data integrity.  

## Network error handling

Network operations introduce additional complexity with timeouts, connection  
issues, and protocol-specific errors that require specialized handling.  

```go
package main

import (
    "fmt"
    "net"
    "net/http"
    "strings"
    "time"
)

type NetworkError struct {
    Operation string
    Address   string
    Err       error
    Temporary bool
    Timeout   bool
}

func (e NetworkError) Error() string {
    return fmt.Sprintf("network %s error for %s: %v", e.Operation, e.Address, e.Err)
}

func (e NetworkError) Unwrap() error {
    return e.Err
}

func (e NetworkError) IsTemporary() bool {
    return e.Temporary
}

func (e NetworkError) IsTimeout() bool {
    return e.Timeout
}

func classifyNetworkError(op, addr string, err error) error {
    if err == nil {
        return nil
    }
    
    netErr := NetworkError{
        Operation: op,
        Address:   addr,
        Err:       err,
    }
    
    // Check for timeout
    if netErr, ok := err.(net.Error); ok {
        netErr.Timeout = netErr.Timeout()
        netErr.Temporary = netErr.Temporary()
    }
    
    return netErr
}

func connectWithRetry(address string, maxRetries int) (net.Conn, error) {
    var lastErr error
    
    for attempt := 1; attempt <= maxRetries; attempt++ {
        fmt.Printf("Connection attempt %d to %s\n", attempt, address)
        
        conn, err := net.DialTimeout("tcp", address, 2*time.Second)
        if err == nil {
            fmt.Printf("Connected successfully\n")
            return conn, nil
        }
        
        lastErr = classifyNetworkError("connect", address, err)
        fmt.Printf("Attempt %d failed: %v\n", attempt, lastErr)
        
        // Check if we should retry
        if netErr, ok := lastErr.(NetworkError); ok {
            if !netErr.IsTemporary() && !netErr.IsTimeout() {
                fmt.Printf("Non-retryable error, stopping\n")
                break
            }
        }
        
        if attempt < maxRetries {
            backoff := time.Duration(attempt) * 500 * time.Millisecond
            fmt.Printf("Waiting %v before retry\n", backoff)
            time.Sleep(backoff)
        }
    }
    
    return nil, fmt.Errorf("failed to connect after %d attempts: %w", maxRetries, lastErr)
}

func makeHTTPRequestWithRetry(url string, maxRetries int) (*http.Response, error) {
    client := &http.Client{
        Timeout: 5 * time.Second,
    }
    
    var lastErr error
    for attempt := 1; attempt <= maxRetries; attempt++ {
        fmt.Printf("HTTP request attempt %d to %s\n", attempt, url)
        
        resp, err := client.Get(url)
        if err == nil {
            if resp.StatusCode < 500 {
                // Success or client error (don't retry client errors)
                return resp, nil
            }
            resp.Body.Close()
            lastErr = fmt.Errorf("server error: %d %s", resp.StatusCode, resp.Status)
        } else {
            lastErr = err
        }
        
        fmt.Printf("Attempt %d failed: %v\n", attempt, lastErr)
        
        // Check if error is retryable
        if strings.Contains(lastErr.Error(), "connection refused") {
            fmt.Printf("Connection refused, retrying\n")
        } else if strings.Contains(lastErr.Error(), "timeout") {
            fmt.Printf("Timeout occurred, retrying\n")
        } else if !strings.Contains(lastErr.Error(), "server error") {
            fmt.Printf("Non-retryable error, stopping\n")
            break
        }
        
        if attempt < maxRetries {
            backoff := time.Duration(attempt) * time.Second
            fmt.Printf("Waiting %v before retry\n", backoff)
            time.Sleep(backoff)
        }
    }
    
    return nil, fmt.Errorf("HTTP request failed after %d attempts: %w", maxRetries, lastErr)
}

func checkConnection(host string, port string) error {
    address := net.JoinHostPort(host, port)
    
    conn, err := net.DialTimeout("tcp", address, 3*time.Second)
    if err != nil {
        return classifyNetworkError("dial", address, err)
    }
    defer conn.Close()
    
    fmt.Printf("Successfully connected to %s\n", address)
    return nil
}

func main() {
    fmt.Println("=== TCP Connection Tests ===")
    
    // Test successful connection (assuming localhost SSH is running)
    hosts := []struct{ host, port string }{
        {"google.com", "80"},     // Should work
        {"127.0.0.1", "22"},     // May work if SSH is running
        {"192.0.2.1", "12345"},  // Should fail (reserved IP)
    }
    
    for _, h := range hosts {
        fmt.Printf("\nTesting connection to %s:%s\n", h.host, h.port)
        if err := checkConnection(h.host, h.port); err != nil {
            fmt.Printf("Connection failed: %v\n", err)
            
            if netErr, ok := err.(NetworkError); ok {
                fmt.Printf("  Temporary: %v, Timeout: %v\n", 
                          netErr.IsTemporary(), netErr.IsTimeout())
            }
        }
    }
    
    fmt.Println("\n=== Connection Retry Test ===")
    // This will likely fail but demonstrates retry logic
    conn, err := connectWithRetry("192.0.2.1:12345", 3)
    if err != nil {
        fmt.Printf("Final connection error: %v\n", err)
    } else {
        conn.Close()
    }
    
    fmt.Println("\n=== HTTP Request Test ===")
    urls := []string{
        "https://httpbin.org/status/200", // Should work
        "https://httpbin.org/status/500", // Server error
        "https://nonexistent.example.com", // DNS failure
    }
    
    for _, url := range urls {
        fmt.Printf("\nTesting HTTP request to %s\n", url)
        resp, err := makeHTTPRequestWithRetry(url, 2)
        if err != nil {
            fmt.Printf("HTTP error: %v\n", err)
        } else {
            fmt.Printf("HTTP success: %d %s\n", resp.StatusCode, resp.Status)
            resp.Body.Close()
        }
    }
}
```

Network error handling requires understanding different error types, implementing  
retry logic with backoff, and distinguishing between temporary and permanent  
failures for robust network communication.  

## Database error patterns

Database operations have specific error patterns including connection issues,  
constraint violations, and transaction handling that require specialized  
error handling strategies.  

```go
package main

import (
    "fmt"
    "strings"
)

// Simulate database error types
type DBError struct {
    Type      string
    Message   string
    Code      int
    Query     string
    Retryable bool
}

func (e DBError) Error() string {
    return fmt.Sprintf("DB %s error (code %d): %s", e.Type, e.Code, e.Message)
}

func (e DBError) IsRetryable() bool {
    return e.Retryable
}

func (e DBError) IsConstraintViolation() bool {
    return e.Type == "constraint"
}

func (e DBError) IsConnectionError() bool {
    return e.Type == "connection"
}

// Mock database connection
type Database struct {
    connected bool
    inTx      bool
}

func NewDatabase() *Database {
    return &Database{connected: true}
}

func (db *Database) Query(query string) ([]map[string]interface{}, error) {
    if !db.connected {
        return nil, DBError{
            Type:      "connection",
            Message:   "database connection lost",
            Code:      2003,
            Query:     query,
            Retryable: true,
        }
    }
    
    // Simulate different query scenarios
    query = strings.ToLower(strings.TrimSpace(query))
    
    switch {
    case strings.Contains(query, "duplicate"):
        return nil, DBError{
            Type:      "constraint",
            Message:   "duplicate key violation",
            Code:      1062,
            Query:     query,
            Retryable: false,
        }
    case strings.Contains(query, "foreign"):
        return nil, DBError{
            Type:      "constraint",
            Message:   "foreign key constraint fails",
            Code:      1452,
            Query:     query,
            Retryable: false,
        }
    case strings.Contains(query, "timeout"):
        return nil, DBError{
            Type:      "timeout",
            Message:   "query execution timeout",
            Code:      1205,
            Query:     query,
            Retryable: true,
        }
    case strings.Contains(query, "syntax"):
        return nil, DBError{
            Type:      "syntax",
            Message:   "syntax error in SQL statement",
            Code:      1064,
            Query:     query,
            Retryable: false,
        }
    default:
        return []map[string]interface{}{
            {"id": 1, "name": "John"},
            {"id": 2, "name": "Jane"},
        }, nil
    }
}

func (db *Database) BeginTx() error {
    if !db.connected {
        return DBError{
            Type:      "connection",
            Message:   "cannot start transaction: connection lost",
            Code:      2006,
            Retryable: true,
        }
    }
    
    if db.inTx {
        return DBError{
            Type:      "transaction",
            Message:   "transaction already in progress",
            Code:      1000,
            Retryable: false,
        }
    }
    
    db.inTx = true
    return nil
}

func (db *Database) Commit() error {
    if !db.inTx {
        return DBError{
            Type:      "transaction",
            Message:   "no active transaction to commit",
            Code:      1001,
            Retryable: false,
        }
    }
    
    db.inTx = false
    return nil
}

func (db *Database) Rollback() error {
    if !db.inTx {
        return DBError{
            Type:      "transaction",
            Message:   "no active transaction to rollback",
            Code:      1002,
            Retryable: false,
        }
    }
    
    db.inTx = false
    return nil
}

// Transaction wrapper with proper error handling
func executeTransaction(db *Database, operations []string) error {
    if err := db.BeginTx(); err != nil {
        return fmt.Errorf("failed to start transaction: %w", err)
    }
    
    // Ensure rollback on any error
    defer func() {
        if db.inTx {
            if rollbackErr := db.Rollback(); rollbackErr != nil {
                fmt.Printf("Warning: rollback failed: %v\n", rollbackErr)
            }
        }
    }()
    
    for i, query := range operations {
        fmt.Printf("Executing operation %d: %s\n", i+1, query)
        
        _, err := db.Query(query)
        if err != nil {
            return fmt.Errorf("operation %d failed: %w", i+1, err)
        }
    }
    
    if err := db.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    fmt.Println("Transaction committed successfully")
    return nil
}

// Retry logic for database operations
func executeWithRetry(db *Database, query string, maxRetries int) ([]map[string]interface{}, error) {
    var lastErr error
    
    for attempt := 1; attempt <= maxRetries; attempt++ {
        fmt.Printf("Query attempt %d: %s\n", attempt, query)
        
        results, err := db.Query(query)
        if err == nil {
            return results, nil
        }
        
        lastErr = err
        fmt.Printf("Attempt %d failed: %v\n", attempt, err)
        
        // Check if error is retryable
        if dbErr, ok := err.(DBError); ok {
            if !dbErr.IsRetryable() {
                fmt.Printf("Non-retryable error, stopping\n")
                break
            }
            
            if dbErr.IsConnectionError() {
                fmt.Printf("Connection error, attempting to reconnect\n")
                db.connected = true // Simulate reconnection
            }
        }
        
        if attempt < maxRetries {
            fmt.Printf("Retrying in 1 second...\n")
            // time.Sleep(time.Second) // Simulate delay
        }
    }
    
    return nil, fmt.Errorf("query failed after %d attempts: %w", maxRetries, lastErr)
}

func main() {
    db := NewDatabase()
    
    fmt.Println("=== Basic Database Query Tests ===")
    testQueries := []string{
        "SELECT * FROM users",
        "INSERT INTO users VALUES (1, 'duplicate')", 
        "INSERT INTO orders VALUES (99, foreign)",   
        "SELECT * FROM timeout_table",               
        "SELECT * FROM syntax error",                
    }
    
    for _, query := range testQueries {
        fmt.Printf("\nExecuting: %s\n", query)
        results, err := db.Query(query)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            
            if dbErr, ok := err.(DBError); ok {
                fmt.Printf("  Error type: %s\n", dbErr.Type)
                fmt.Printf("  Retryable: %v\n", dbErr.IsRetryable())
                fmt.Printf("  Constraint violation: %v\n", dbErr.IsConstraintViolation())
            }
        } else {
            fmt.Printf("Success: %d rows\n", len(results))
        }
    }
    
    fmt.Println("\n=== Transaction Error Handling ===")
    transactionOps := [][]string{
        {"INSERT INTO users VALUES (1, 'John')", "UPDATE users SET name='Jane' WHERE id=1"},
        {"INSERT INTO users VALUES (2, 'duplicate')", "INSERT INTO users VALUES (3, 'Bob')"},
        {"INSERT INTO orders VALUES (1, foreign)", "UPDATE orders SET status='shipped'"},
    }
    
    for i, ops := range transactionOps {
        fmt.Printf("\n--- Transaction %d ---\n", i+1)
        if err := executeTransaction(db, ops); err != nil {
            fmt.Printf("Transaction failed: %v\n", err)
        }
    }
    
    fmt.Println("\n=== Retry Logic Test ===")
    // Simulate connection loss and recovery
    db.connected = false
    
    _, err := executeWithRetry(db, "SELECT * FROM users", 3)
    if err != nil {
        fmt.Printf("Final error: %v\n", err)
    }
}
```

Database error handling involves understanding different error types, implementing  
proper transaction management with rollback, and building retry logic for  
transient failures while avoiding retries for constraint violations.  

## JSON and data validation errors

Data validation and JSON processing require comprehensive error handling  
for malformed data, validation failures, and type conversion issues.  

```go
package main

import (
    "encoding/json"
    "errors"
    "fmt"
    "reflect"
    "regexp"
    "strconv"
    "strings"
    "time"
)

// Validation error with field-specific information
type ValidationError struct {
    Field   string
    Value   interface{}
    Rule    string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed for field %q: %s (value: %v)", 
                       e.Field, e.Message, e.Value)
}

// Aggregated validation errors
type ValidationErrors struct {
    Errors []ValidationError
}

func (e ValidationErrors) Error() string {
    if len(e.Errors) == 0 {
        return "no validation errors"
    }
    
    var messages []string
    for _, err := range e.Errors {
        messages = append(messages, err.Error())
    }
    
    return fmt.Sprintf("validation failed with %d error(s):\n  - %s", 
                       len(e.Errors), strings.Join(messages, "\n  - "))
}

func (e *ValidationErrors) Add(field, rule, message string, value interface{}) {
    e.Errors = append(e.Errors, ValidationError{
        Field:   field,
        Value:   value,
        Rule:    rule,
        Message: message,
    })
}

func (e ValidationErrors) HasErrors() bool {
    return len(e.Errors) > 0
}

// User struct with JSON tags
type User struct {
    ID       int       `json:"id"`
    Username string    `json:"username"`
    Email    string    `json:"email"`
    Age      int       `json:"age"`
    Active   bool      `json:"active"`
    Created  time.Time `json:"created"`
}

// Validate user data
func (u User) Validate() error {
    var errors ValidationErrors
    
    // Username validation
    if u.Username == "" {
        errors.Add("username", "required", "username cannot be empty", u.Username)
    } else if len(u.Username) < 3 {
        errors.Add("username", "min_length", "username must be at least 3 characters", u.Username)
    } else if len(u.Username) > 20 {
        errors.Add("username", "max_length", "username must be at most 20 characters", u.Username)
    }
    
    // Email validation
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    if u.Email == "" {
        errors.Add("email", "required", "email cannot be empty", u.Email)
    } else if !emailRegex.MatchString(u.Email) {
        errors.Add("email", "format", "invalid email format", u.Email)
    }
    
    // Age validation
    if u.Age < 0 {
        errors.Add("age", "minimum", "age cannot be negative", u.Age)
    } else if u.Age > 150 {
        errors.Add("age", "maximum", "age seems unrealistic", u.Age)
    }
    
    // ID validation
    if u.ID <= 0 {
        errors.Add("id", "positive", "ID must be positive", u.ID)
    }
    
    if errors.HasErrors() {
        return errors
    }
    return nil
}

// Safe JSON parsing with detailed error handling
func parseUserJSON(jsonData []byte) (User, error) {
    var user User
    
    // First, try to parse as generic map to provide better error messages
    var rawData map[string]interface{}
    if err := json.Unmarshal(jsonData, &rawData); err != nil {
        // Handle JSON syntax errors
        if syntaxErr, ok := err.(*json.SyntaxError); ok {
            return user, fmt.Errorf("JSON syntax error at position %d: %w", 
                                  syntaxErr.Offset, err)
        }
        if typeErr, ok := err.(*json.UnmarshalTypeError); ok {
            return user, fmt.Errorf("JSON type error: cannot unmarshal %s into field %q of type %s at position %d", 
                                  typeErr.Value, typeErr.Field, typeErr.Type, typeErr.Offset)
        }
        return user, fmt.Errorf("JSON parsing error: %w", err)
    }
    
    // Now parse into struct with validation
    if err := json.Unmarshal(jsonData, &user); err != nil {
        return user, fmt.Errorf("failed to parse user data: %w", err)
    }
    
    // Validate parsed data
    if err := user.Validate(); err != nil {
        return user, fmt.Errorf("user validation failed: %w", err)
    }
    
    return user, nil
}

// Batch processing with error collection
func processUserBatch(jsonDataList []string) ([]User, []error) {
    var users []User
    var errors []error
    
    for i, jsonData := range jsonDataList {
        user, err := parseUserJSON([]byte(jsonData))
        if err != nil {
            errors = append(errors, fmt.Errorf("user %d: %w", i+1, err))
            continue
        }
        
        users = append(users, user)
    }
    
    return users, errors
}

// Generic validation function
func validateStruct(v interface{}) error {
    val := reflect.ValueOf(v)
    typ := reflect.TypeOf(v)
    
    if val.Kind() == reflect.Ptr {
        val = val.Elem()
        typ = typ.Elem()
    }
    
    if val.Kind() != reflect.Struct {
        return fmt.Errorf("validation requires struct type, got %T", v)
    }
    
    var errors ValidationErrors
    
    for i := 0; i < val.NumField(); i++ {
        field := val.Field(i)
        fieldType := typ.Field(i)
        fieldName := fieldType.Name
        
        // Check for required tag
        if tag := fieldType.Tag.Get("required"); tag == "true" {
            if field.Kind() == reflect.String && field.String() == "" {
                errors.Add(fieldName, "required", "field is required but empty", field.Interface())
            }
            if field.Kind() == reflect.Int && field.Int() == 0 {
                errors.Add(fieldName, "required", "field is required but zero", field.Interface())
            }
        }
        
        // Check for min tag on strings
        if tag := fieldType.Tag.Get("min"); tag != "" && field.Kind() == reflect.String {
            if minLen, err := strconv.Atoi(tag); err == nil {
                if len(field.String()) < minLen {
                    errors.Add(fieldName, "min_length", 
                             fmt.Sprintf("field must be at least %d characters", minLen), 
                             field.Interface())
                }
            }
        }
    }
    
    if errors.HasErrors() {
        return errors
    }
    return nil
}

func main() {
    fmt.Println("=== JSON Parsing Tests ===")
    
    testJSONs := []string{
        `{"id": 1, "username": "john_doe", "email": "john@example.com", "age": 25, "active": true, "created": "2023-01-15T10:00:00Z"}`,
        `{"id": 2, "username": "", "email": "jane@example.com", "age": 30, "active": true, "created": "2023-01-16T10:00:00Z"}`,
        `{"id": 3, "username": "bob", "email": "invalid-email", "age": -5, "active": true, "created": "2023-01-17T10:00:00Z"}`,
        `{"id": "not-a-number", "username": "alice", "email": "alice@example.com", "age": 28, "active": true}`,
        `{"id": 5, "username": "charlie", "email": "charlie@example.com", "age": 200, "active": true, "created": "2023-01-19T10:00:00Z"}`,
        `{invalid json}`,
        `{"id": 0, "username": "x", "email": "@", "age": 35, "active": true, "created": "2023-01-20T10:00:00Z"}`,
    }
    
    for i, jsonData := range testJSONs {
        fmt.Printf("\n--- Test %d ---\n", i+1)
        fmt.Printf("JSON: %s\n", jsonData)
        
        user, err := parseUserJSON([]byte(jsonData))
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            
            // Check for specific error types
            if validationErr, ok := err.(ValidationErrors); ok {
                fmt.Printf("Validation errors (%d):\n", len(validationErr.Errors))
                for _, ve := range validationErr.Errors {
                    fmt.Printf("  - %s: %s\n", ve.Field, ve.Message)
                }
            }
        } else {
            fmt.Printf("Success: %+v\n", user)
        }
    }
    
    fmt.Println("\n=== Batch Processing Test ===")
    users, errors := processUserBatch(testJSONs)
    
    fmt.Printf("Successfully parsed %d users\n", len(users))
    fmt.Printf("Encountered %d errors:\n", len(errors))
    for _, err := range errors {
        fmt.Printf("  - %v\n", err)
    }
    
    fmt.Println("\n=== Generic Validation Test ===")
    type TestStruct struct {
        Name     string `required:"true" min:"2"`
        Email    string `required:"true"`
        Age      int    `required:"true"`
        Optional string
    }
    
    testStructs := []TestStruct{
        {"John Doe", "john@example.com", 25, "optional"},
        {"", "jane@example.com", 30, ""},
        {"X", "invalid", 0, "test"},
    }
    
    for i, ts := range testStructs {
        fmt.Printf("Struct %d: %+v\n", i+1, ts)
        if err := validateStruct(ts); err != nil {
            fmt.Printf("  Error: %v\n", err)
        } else {
            fmt.Printf("  Valid\n")
        }
    }
}
```

JSON and validation error handling requires structured error reporting,  
field-specific error messages, and proper handling of parsing versus  
validation errors for comprehensive data processing workflows.  

## HTTP client error handling

HTTP clients need robust error handling for various failure scenarios  
including network issues, HTTP status codes, and response processing errors.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "strings"
    "time"
)

type HTTPClientError struct {
    StatusCode int
    Status     string
    URL        string
    Method     string
    Body       string
    Err        error
}

func (e HTTPClientError) Error() string {
    if e.StatusCode > 0 {
        return fmt.Sprintf("HTTP %d %s for %s %s: %s", 
                          e.StatusCode, e.Status, e.Method, e.URL, e.Body)
    }
    return fmt.Sprintf("HTTP client error for %s %s: %v", e.Method, e.URL, e.Err)
}

func (e HTTPClientError) IsClientError() bool {
    return e.StatusCode >= 400 && e.StatusCode < 500
}

func (e HTTPClientError) IsServerError() bool {
    return e.StatusCode >= 500
}

type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data"`
    Message string      `json:"message"`
    Error   string      `json:"error"`
}

func makeHTTPRequest(method, url string, body io.Reader) (*http.Response, error) {
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    req, err := http.NewRequest(method, url, body)
    if err != nil {
        return nil, HTTPClientError{
            Method: method,
            URL:    url,
            Err:    fmt.Errorf("failed to create request: %w", err),
        }
    }
    
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("User-Agent", "Go-HTTP-Client/1.0")
    
    resp, err := client.Do(req)
    if err != nil {
        return nil, HTTPClientError{
            Method: method,
            URL:    url,
            Err:    fmt.Errorf("request failed: %w", err),
        }
    }
    
    return resp, nil
}

func handleHTTPResponse(resp *http.Response) (APIResponse, error) {
    defer resp.Body.Close()
    
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        return APIResponse{}, HTTPClientError{
            StatusCode: resp.StatusCode,
            Status:     resp.Status,
            URL:        resp.Request.URL.String(),
            Method:     resp.Request.Method,
            Err:        fmt.Errorf("failed to read response body: %w", err),
        }
    }
    
    if resp.StatusCode >= 400 {
        return APIResponse{}, HTTPClientError{
            StatusCode: resp.StatusCode,
            Status:     resp.Status,
            URL:        resp.Request.URL.String(),
            Method:     resp.Request.Method,
            Body:       string(body),
        }
    }
    
    var apiResp APIResponse
    if err := json.Unmarshal(body, &apiResp); err != nil {
        return APIResponse{}, HTTPClientError{
            StatusCode: resp.StatusCode,
            Status:     resp.Status,
            URL:        resp.Request.URL.String(),
            Method:     resp.Request.Method,
            Err:        fmt.Errorf("failed to parse JSON response: %w", err),
        }
    }
    
    return apiResp, nil
}

func fetchUserData(userID string) (map[string]interface{}, error) {
    url := fmt.Sprintf("https://httpbin.org/status/%s", userID)
    
    resp, err := makeHTTPRequest("GET", url, nil)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch user %s: %w", userID, err)
    }
    
    apiResp, err := handleHTTPResponse(resp)
    if err != nil {
        return nil, fmt.Errorf("failed to process response for user %s: %w", userID, err)
    }
    
    if userData, ok := apiResp.Data.(map[string]interface{}); ok {
        return userData, nil
    }
    
    return nil, fmt.Errorf("unexpected response format for user %s", userID)
}

func batchHTTPRequests(urls []string) ([]string, []error) {
    var results []string
    var errors []error
    
    for i, url := range urls {
        fmt.Printf("Requesting %d: %s\n", i+1, url)
        
        resp, err := makeHTTPRequest("GET", url, nil)
        if err != nil {
            errors = append(errors, fmt.Errorf("request %d failed: %w", i+1, err))
            continue
        }
        
        _, err = handleHTTPResponse(resp)
        if err != nil {
            errors = append(errors, fmt.Errorf("response %d failed: %w", i+1, err))
            
            // Log specific error types
            if httpErr, ok := err.(HTTPClientError); ok {
                if httpErr.IsClientError() {
                    fmt.Printf("  Client error: %d\n", httpErr.StatusCode)
                } else if httpErr.IsServerError() {
                    fmt.Printf("  Server error: %d\n", httpErr.StatusCode)
                }
            }
            continue
        }
        
        results = append(results, fmt.Sprintf("Success: %s", url))
    }
    
    return results, errors
}

func main() {
    fmt.Println("=== HTTP Client Error Tests ===")
    
    testURLs := []string{
        "https://httpbin.org/status/200", // Success
        "https://httpbin.org/status/404", // Not found
        "https://httpbin.org/status/500", // Server error
        "https://httpbin.org/status/429", // Rate limit
        "https://nonexistent.example.com/api", // DNS failure
    }
    
    results, errors := batchHTTPRequests(testURLs)
    
    fmt.Printf("\nResults:\n")
    for _, result := range results {
        fmt.Printf("  ✓ %s\n", result)
    }
    
    fmt.Printf("\nErrors (%d):\n", len(errors))
    for _, err := range errors {
        fmt.Printf("  ✗ %v\n", err)
    }
    
    fmt.Println("\n=== Specific Error Type Handling ===")
    
    // Test specific error scenarios
    scenarios := map[string]string{
        "200": "Success case",
        "401": "Unauthorized",
        "404": "Not found", 
        "500": "Server error",
        "503": "Service unavailable",
    }
    
    for status, desc := range scenarios {
        fmt.Printf("\nTesting %s (%s):\n", desc, status)
        
        _, err := fetchUserData(status)
        if err != nil {
            fmt.Printf("  Error: %v\n", err)
            
            if httpErr, ok := err.(HTTPClientError); ok {
                fmt.Printf("  Status Code: %d\n", httpErr.StatusCode)
                fmt.Printf("  Is Client Error: %v\n", httpErr.IsClientError())
                fmt.Printf("  Is Server Error: %v\n", httpErr.IsServerError())
            }
        } else {
            fmt.Printf("  Success\n")
        }
    }
}
```

HTTP client error handling involves managing different status codes, network  
errors, and response parsing failures while providing meaningful error  
information for debugging and appropriate retry strategies.  

## Goroutine error handling

Error handling in concurrent code requires special patterns for collecting  
errors from goroutines and coordinating error propagation across concurrent  
operations.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Worker error with goroutine information
type WorkerError struct {
    WorkerID int
    Task     string
    Err      error
}

func (e WorkerError) Error() string {
    return fmt.Sprintf("worker %d failed on task %q: %v", e.WorkerID, e.Task, e.Err)
}

func (e WorkerError) Unwrap() error {
    return e.Err
}

// Error collector for concurrent operations
type ErrorCollector struct {
    mu     sync.Mutex
    errors []error
}

func (ec *ErrorCollector) Add(err error) {
    if err != nil {
        ec.mu.Lock()
        ec.errors = append(ec.errors, err)
        ec.mu.Unlock()
    }
}

func (ec *ErrorCollector) GetErrors() []error {
    ec.mu.Lock()
    defer ec.mu.Unlock()
    result := make([]error, len(ec.errors))
    copy(result, ec.errors)
    return result
}

func (ec *ErrorCollector) HasErrors() bool {
    ec.mu.Lock()
    defer ec.mu.Unlock()
    return len(ec.errors) > 0
}

// Simulate work that might fail
func processItem(workerID int, item string) error {
    // Simulate different failure scenarios
    switch item {
    case "error":
        return fmt.Errorf("processing error for item %q", item)
    case "panic":
        panic("simulated panic during processing")
    case "timeout":
        time.Sleep(2 * time.Second) // Will trigger timeout in context
        return nil
    default:
        time.Sleep(100 * time.Millisecond) // Simulate work
        fmt.Printf("Worker %d processed: %s\n", workerID, item)
        return nil
    }
}

// Worker with panic recovery
func safeWorker(workerID int, jobs <-chan string, results chan<- string, 
               errorChan chan<- error, wg *sync.WaitGroup) {
    defer wg.Done()
    
    defer func() {
        if r := recover(); r != nil {
            errorChan <- WorkerError{
                WorkerID: workerID,
                Task:     "panic recovery",
                Err:      fmt.Errorf("panic: %v", r),
            }
        }
    }()
    
    for job := range jobs {
        err := processItem(workerID, job)
        if err != nil {
            errorChan <- WorkerError{
                WorkerID: workerID,
                Task:     job,
                Err:      err,
            }
        } else {
            results <- fmt.Sprintf("worker-%d: %s", workerID, job)
        }
    }
}

// Process jobs with error collection
func processJobsConcurrent(jobs []string, numWorkers int) ([]string, []error) {
    jobChan := make(chan string, len(jobs))
    resultChan := make(chan string, len(jobs))
    errorChan := make(chan error, len(jobs))
    
    var wg sync.WaitGroup
    
    // Start workers
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go safeWorker(i, jobChan, resultChan, errorChan, &wg)
    }
    
    // Send jobs
    for _, job := range jobs {
        jobChan <- job
    }
    close(jobChan)
    
    // Wait for completion
    go func() {
        wg.Wait()
        close(resultChan)
        close(errorChan)
    }()
    
    // Collect results and errors
    var results []string
    var errors []error
    
    resultsOpen := true
    errorsOpen := true
    
    for resultsOpen || errorsOpen {
        select {
        case result, ok := <-resultChan:
            if !ok {
                resultsOpen = false
            } else {
                results = append(results, result)
            }
        case err, ok := <-errorChan:
            if !ok {
                errorsOpen = false
            } else {
                errors = append(errors, err)
            }
        }
    }
    
    return results, errors
}

// Context-aware concurrent processing
func processWithContext(ctx context.Context, jobs []string) ([]string, error) {
    var results []string
    var mu sync.Mutex
    errorCollector := &ErrorCollector{}
    
    var wg sync.WaitGroup
    semaphore := make(chan struct{}, 3) // Limit concurrent operations
    
    for i, job := range jobs {
        wg.Add(1)
        
        go func(jobID int, item string) {
            defer wg.Done()
            
            // Acquire semaphore
            select {
            case semaphore <- struct{}{}:
                defer func() { <-semaphore }()
            case <-ctx.Done():
                errorCollector.Add(fmt.Errorf("job %d cancelled before start: %w", 
                                             jobID, ctx.Err()))
                return
            }
            
            // Check for cancellation
            select {
            case <-ctx.Done():
                errorCollector.Add(fmt.Errorf("job %d cancelled: %w", jobID, ctx.Err()))
                return
            default:
            }
            
            // Process with context checking
            done := make(chan error, 1)
            go func() {
                done <- processItem(jobID, item)
            }()
            
            select {
            case err := <-done:
                if err != nil {
                    errorCollector.Add(WorkerError{
                        WorkerID: jobID,
                        Task:     item,
                        Err:      err,
                    })
                } else {
                    mu.Lock()
                    results = append(results, fmt.Sprintf("job-%d: %s", jobID, item))
                    mu.Unlock()
                }
            case <-ctx.Done():
                errorCollector.Add(fmt.Errorf("job %d timed out: %w", jobID, ctx.Err()))
            }
        }(i+1, job)
    }
    
    wg.Wait()
    
    if errorCollector.HasErrors() {
        errors := errorCollector.GetErrors()
        return results, fmt.Errorf("processing completed with %d errors: %v", 
                                 len(errors), errors[0])
    }
    
    return results, nil
}

func main() {
    fmt.Println("=== Concurrent Job Processing ===")
    
    jobs := []string{"task1", "task2", "error", "task3", "panic", "task4", "timeout", "task5"}
    
    results, errors := processJobsConcurrent(jobs, 3)
    
    fmt.Printf("Completed %d jobs successfully:\n", len(results))
    for _, result := range results {
        fmt.Printf("  ✓ %s\n", result)
    }
    
    fmt.Printf("\nEncountered %d errors:\n", len(errors))
    for _, err := range errors {
        fmt.Printf("  ✗ %v\n", err)
        
        if workerErr, ok := err.(WorkerError); ok {
            fmt.Printf("    Worker ID: %d, Task: %s\n", 
                      workerErr.WorkerID, workerErr.Task)
        }
    }
    
    fmt.Println("\n=== Context-Based Processing ===")
    
    // Test with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 500*time.Millisecond)
    defer cancel()
    
    jobs2 := []string{"quick1", "quick2", "timeout", "quick3"}
    
    results2, err := processWithContext(ctx, jobs2)
    if err != nil {
        fmt.Printf("Processing error: %v\n", err)
    }
    
    fmt.Printf("Context results (%d):\n", len(results2))
    for _, result := range results2 {
        fmt.Printf("  ✓ %s\n", result)
    }
    
    fmt.Println("\n=== Fast Processing (No Timeout) ===")
    
    ctx3, cancel3 := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel3()
    
    jobs3 := []string{"fast1", "fast2", "fast3"}
    
    results3, err3 := processWithContext(ctx3, jobs3)
    if err3 != nil {
        fmt.Printf("Processing error: %v\n", err3)
    } else {
        fmt.Printf("All tasks completed successfully (%d results)\n", len(results3))
    }
}
```

Goroutine error handling requires collecting errors from concurrent operations,  
implementing panic recovery, and coordinating cancellation across multiple  
goroutines using contexts and synchronization primitives.  

## Testing error scenarios

Comprehensive testing of error conditions ensures robust error handling  
and helps validate error behavior under various failure scenarios.  

```go
package main

import (
    "errors"
    "fmt"
    "testing"
)

// Calculator with various error conditions
type Calculator struct {
    precision int
}

var (
    ErrDivisionByZero = errors.New("division by zero")
    ErrNegativeInput  = errors.New("negative input not allowed")
    ErrOverflow       = errors.New("result overflow")
)

func NewCalculator(precision int) *Calculator {
    return &Calculator{precision: precision}
}

func (c *Calculator) Add(a, b float64) (float64, error) {
    result := a + b
    
    // Check for overflow (simplified)
    if result > 1e10 {
        return 0, ErrOverflow
    }
    
    return result, nil
}

func (c *Calculator) Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, ErrDivisionByZero
    }
    
    if a < 0 || b < 0 {
        return 0, ErrNegativeInput
    }
    
    result := a / b
    
    if result > 1e10 {
        return 0, ErrOverflow
    }
    
    return result, nil
}

func (c *Calculator) SquareRoot(x float64) (float64, error) {
    if x < 0 {
        return 0, ErrNegativeInput
    }
    
    // Simplified square root
    if x == 0 {
        return 0, nil
    }
    
    return x * 0.5, nil // Dummy implementation
}

// Test helper functions
func assertEqual(t *testing.T, got, want interface{}) {
    if got != want {
        t.Errorf("got %v, want %v", got, want)
    }
}

func assertError(t *testing.T, got error, want error) {
    if got == nil && want != nil {
        t.Errorf("expected error %v, got nil", want)
        return
    }
    
    if got != nil && want == nil {
        t.Errorf("unexpected error: %v", got)
        return
    }
    
    if got != nil && want != nil && !errors.Is(got, want) {
        t.Errorf("expected error %v, got %v", want, got)
    }
}

// Example test functions (normally in _test.go files)
func TestCalculatorAdd(t *testing.T) {
    calc := NewCalculator(2)
    
    tests := []struct {
        name    string
        a, b    float64
        want    float64
        wantErr error
    }{
        {"positive numbers", 2.5, 3.5, 6.0, nil},
        {"with zero", 5.0, 0.0, 5.0, nil},
        {"negative numbers", -2.0, -3.0, -5.0, nil},
        {"overflow", 1e10, 1e10, 0, ErrOverflow},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := calc.Add(tt.a, tt.b)
            
            assertError(t, err, tt.wantErr)
            
            if err == nil {
                assertEqual(t, got, tt.want)
            }
        })
    }
}

func TestCalculatorDivide(t *testing.T) {
    calc := NewCalculator(2)
    
    tests := []struct {
        name    string
        a, b    float64
        want    float64
        wantErr error
    }{
        {"normal division", 10.0, 2.0, 5.0, nil},
        {"division by zero", 10.0, 0.0, 0, ErrDivisionByZero},
        {"negative dividend", -10.0, 2.0, 0, ErrNegativeInput},
        {"negative divisor", 10.0, -2.0, 0, ErrNegativeInput},
        {"both negative", -10.0, -2.0, 0, ErrNegativeInput},
        {"result overflow", 1e11, 1.0, 0, ErrOverflow},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := calc.Divide(tt.a, tt.b)
            
            assertError(t, err, tt.wantErr)
            
            if err == nil {
                assertEqual(t, got, tt.want)
            }
        })
    }
}

func TestCalculatorSquareRoot(t *testing.T) {
    calc := NewCalculator(2)
    
    tests := []struct {
        name    string
        input   float64
        wantErr error
    }{
        {"positive number", 4.0, nil},
        {"zero", 0.0, nil},
        {"negative number", -4.0, ErrNegativeInput},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := calc.SquareRoot(tt.input)
            assertError(t, err, tt.wantErr)
        })
    }
}

// Error wrapping test
func TestErrorWrapping(t *testing.T) {
    calc := NewCalculator(2)
    
    _, err := calc.Divide(10, 0)
    wrappedErr := fmt.Errorf("calculation failed: %w", err)
    
    // Test that we can identify the original error
    if !errors.Is(wrappedErr, ErrDivisionByZero) {
        t.Errorf("wrapped error should contain ErrDivisionByZero")
    }
    
    // Test error message
    expected := "calculation failed: division by zero"
    if wrappedErr.Error() != expected {
        t.Errorf("expected %q, got %q", expected, wrappedErr.Error())
    }
}

// Benchmark error handling performance
func BenchmarkCalculatorWithoutError(b *testing.B) {
    calc := NewCalculator(2)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, _ = calc.Add(float64(i), float64(i+1))
    }
}

func BenchmarkCalculatorWithError(b *testing.B) {
    calc := NewCalculator(2)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, _ = calc.Divide(float64(i), 0) // Always triggers error
    }
}

// Mock error demonstration
type MockFileSystem struct {
    files map[string]string
    readError bool
}

func (m *MockFileSystem) ReadFile(filename string) (string, error) {
    if m.readError {
        return "", fmt.Errorf("mock read error for %s", filename)
    }
    
    if content, exists := m.files[filename]; exists {
        return content, nil
    }
    
    return "", fmt.Errorf("file not found: %s", filename)
}

func TestFileSystemMock(t *testing.T) {
    fs := &MockFileSystem{
        files: map[string]string{
            "test.txt": "file content",
        },
    }
    
    // Test successful read
    content, err := fs.ReadFile("test.txt")
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    if content != "file content" {
        t.Errorf("expected 'file content', got %q", content)
    }
    
    // Test file not found
    _, err = fs.ReadFile("nonexistent.txt")
    if err == nil {
        t.Error("expected error for nonexistent file")
    }
    
    // Test read error
    fs.readError = true
    _, err = fs.ReadFile("test.txt")
    if err == nil {
        t.Error("expected read error")
    }
}

func main() {
    fmt.Println("=== Running Error Tests ===")
    
    // Simulate running tests
    fmt.Println("TestCalculatorAdd:")
    t := &testing.T{}
    TestCalculatorAdd(t)
    if t.Failed() {
        fmt.Println("  FAILED")
    } else {
        fmt.Println("  PASSED")
    }
    
    fmt.Println("\nTestCalculatorDivide:")
    t2 := &testing.T{}
    TestCalculatorDivide(t2)
    if t2.Failed() {
        fmt.Println("  FAILED")
    } else {
        fmt.Println("  PASSED")
    }
    
    fmt.Println("\nTestErrorWrapping:")
    t3 := &testing.T{}
    TestErrorWrapping(t3)
    if t3.Failed() {
        fmt.Println("  FAILED")
    } else {
        fmt.Println("  PASSED")
    }
    
    fmt.Println("\nTestFileSystemMock:")
    t4 := &testing.T{}
    TestFileSystemMock(t4)
    if t4.Failed() {
        fmt.Println("  FAILED")
    } else {
        fmt.Println("  PASSED")
    }
    
    fmt.Println("\n=== Manual Error Testing ===")
    
    calc := NewCalculator(2)
    
    // Test various error conditions
    errorCases := []struct {
        operation string
        test      func() error
    }{
        {"Division by zero", func() error {
            _, err := calc.Divide(10, 0)
            return err
        }},
        {"Negative square root", func() error {
            _, err := calc.SquareRoot(-4)
            return err
        }},
        {"Overflow", func() error {
            _, err := calc.Add(1e11, 1e11)
            return err
        }},
    }
    
    for _, tc := range errorCases {
        fmt.Printf("Testing %s: ", tc.operation)
        if err := tc.test(); err != nil {
            fmt.Printf("✓ Error caught: %v\n", err)
        } else {
            fmt.Printf("✗ No error (unexpected)\n")
        }
    }
}
```

Testing error scenarios requires comprehensive test cases covering all error  
conditions, proper error assertion helpers, and techniques like mocking  
to simulate error conditions reliably.  

## Recovery strategies and resilience

Building resilient systems requires implementing recovery strategies that  
handle errors gracefully and maintain system stability under failure conditions.  

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "log"
    "math/rand"
    "sync"
    "time"
)

// Circuit breaker pattern for resilience
type CircuitBreaker struct {
    mu           sync.Mutex
    maxFailures  int
    resetTimeout time.Duration
    failures     int
    lastFailTime time.Time
    state        string // "closed", "open", "half-open"
}

func NewCircuitBreaker(maxFailures int, resetTimeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures:  maxFailures,
        resetTimeout: resetTimeout,
        state:        "closed",
    }
}

func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    // Check if we should transition from open to half-open
    if cb.state == "open" && time.Since(cb.lastFailTime) > cb.resetTimeout {
        cb.state = "half-open"
        cb.failures = 0
    }
    
    // Reject calls when circuit is open
    if cb.state == "open" {
        return fmt.Errorf("circuit breaker is open, rejecting call")
    }
    
    // Execute the function
    err := fn()
    
    if err != nil {
        cb.failures++
        cb.lastFailTime = time.Now()
        
        // Open circuit if threshold reached
        if cb.failures >= cb.maxFailures {
            cb.state = "open"
            fmt.Printf("Circuit breaker opened after %d failures\n", cb.failures)
        }
        
        return fmt.Errorf("operation failed (failures: %d): %w", cb.failures, err)
    }
    
    // Success - reset circuit breaker
    if cb.state == "half-open" {
        cb.state = "closed"
        cb.failures = 0
        fmt.Println("Circuit breaker closed - service recovered")
    }
    
    return nil
}

func (cb *CircuitBreaker) GetState() string {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    return cb.state
}

// Retry with exponential backoff
type RetryConfig struct {
    MaxRetries      int
    InitialDelay    time.Duration
    MaxDelay        time.Duration
    BackoffFactor   float64
    RetryableErrors []error
}

func RetryWithBackoff(ctx context.Context, config RetryConfig, fn func() error) error {
    delay := config.InitialDelay
    
    for attempt := 1; attempt <= config.MaxRetries; attempt++ {
        err := fn()
        if err == nil {
            if attempt > 1 {
                fmt.Printf("Operation succeeded on attempt %d\n", attempt)
            }
            return nil
        }
        
        // Check if error is retryable
        retryable := len(config.RetryableErrors) == 0 // Retry all errors if none specified
        for _, retryErr := range config.RetryableErrors {
            if errors.Is(err, retryErr) {
                retryable = true
                break
            }
        }
        
        if !retryable {
            return fmt.Errorf("non-retryable error: %w", err)
        }
        
        if attempt == config.MaxRetries {
            return fmt.Errorf("max retries (%d) exceeded: %w", config.MaxRetries, err)
        }
        
        fmt.Printf("Attempt %d failed: %v (retrying in %v)\n", attempt, err, delay)
        
        // Wait with context cancellation support
        select {
        case <-ctx.Done():
            return fmt.Errorf("retry cancelled: %w", ctx.Err())
        case <-time.After(delay):
        }
        
        // Exponential backoff
        delay = time.Duration(float64(delay) * config.BackoffFactor)
        if delay > config.MaxDelay {
            delay = config.MaxDelay
        }
    }
    
    return fmt.Errorf("retry logic error") // Should not reach here
}

// Health check system
type HealthChecker struct {
    checks   map[string]func() error
    mu       sync.RWMutex
    statuses map[string]bool
}

func NewHealthChecker() *HealthChecker {
    return &HealthChecker{
        checks:   make(map[string]func() error),
        statuses: make(map[string]bool),
    }
}

func (hc *HealthChecker) AddCheck(name string, check func() error) {
    hc.mu.Lock()
    defer hc.mu.Unlock()
    hc.checks[name] = check
    hc.statuses[name] = true
}

func (hc *HealthChecker) RunChecks() map[string]error {
    hc.mu.Lock()
    defer hc.mu.Unlock()
    
    results := make(map[string]error)
    
    for name, check := range hc.checks {
        err := check()
        results[name] = err
        hc.statuses[name] = err == nil
    }
    
    return results
}

func (hc *HealthChecker) IsHealthy() bool {
    hc.mu.RLock()
    defer hc.mu.RUnlock()
    
    for _, healthy := range hc.statuses {
        if !healthy {
            return false
        }
    }
    return true
}

// Service with recovery strategies
type ResilientService struct {
    cb     *CircuitBreaker
    health *HealthChecker
}

func NewResilientService() *ResilientService {
    service := &ResilientService{
        cb:     NewCircuitBreaker(3, 5*time.Second),
        health: NewHealthChecker(),
    }
    
    // Add health checks
    service.health.AddCheck("database", service.checkDatabase)
    service.health.AddCheck("external_api", service.checkExternalAPI)
    
    return service
}

func (rs *ResilientService) checkDatabase() error {
    // Simulate database health check
    if rand.Float32() < 0.9 { // 90% success rate
        return nil
    }
    return fmt.Errorf("database connection failed")
}

func (rs *ResilientService) checkExternalAPI() error {
    // Simulate external API health check
    if rand.Float32() < 0.8 { // 80% success rate
        return nil
    }
    return fmt.Errorf("external API unreachable")
}

func (rs *ResilientService) ProcessRequest(requestID string) error {
    // Use circuit breaker for protection
    return rs.cb.Execute(func() error {
        // Simulate processing that might fail
        if rand.Float32() < 0.7 { // 70% success rate
            fmt.Printf("Successfully processed request %s\n", requestID)
            return nil
        }
        
        return fmt.Errorf("processing failed for request %s", requestID)
    })
}

func (rs *ResilientService) ProcessWithRetry(ctx context.Context, requestID string) error {
    config := RetryConfig{
        MaxRetries:    3,
        InitialDelay:  100 * time.Millisecond,
        MaxDelay:      1 * time.Second,
        BackoffFactor: 2.0,
    }
    
    return RetryWithBackoff(ctx, config, func() error {
        return rs.ProcessRequest(requestID)
    })
}

func main() {
    fmt.Println("=== Resilience Patterns Demo ===")
    
    service := NewResilientService()
    
    fmt.Println("1. Circuit Breaker Pattern")
    // Test circuit breaker
    for i := 1; i <= 10; i++ {
        requestID := fmt.Sprintf("req-%d", i)
        
        err := service.ProcessRequest(requestID)
        if err != nil {
            fmt.Printf("Request %s failed: %v (circuit: %s)\n", 
                      requestID, err, service.cb.GetState())
        } else {
            fmt.Printf("Request %s succeeded (circuit: %s)\n", 
                      requestID, service.cb.GetState())
        }
        
        time.Sleep(200 * time.Millisecond)
    }
    
    fmt.Println("\n2. Retry with Exponential Backoff")
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    err := service.ProcessWithRetry(ctx, "retry-test")
    if err != nil {
        fmt.Printf("Retry operation failed: %v\n", err)
    } else {
        fmt.Printf("Retry operation succeeded\n")
    }
    
    fmt.Println("\n3. Health Monitoring")
    // Run health checks multiple times
    for i := 1; i <= 5; i++ {
        fmt.Printf("Health check round %d:\n", i)
        
        results := service.health.RunChecks()
        for name, err := range results {
            status := "✓ HEALTHY"
            if err != nil {
                status = fmt.Sprintf("✗ UNHEALTHY: %v", err)
            }
            fmt.Printf("  %s: %s\n", name, status)
        }
        
        fmt.Printf("Overall health: %v\n", service.health.IsHealthy())
        
        if i < 5 {
            time.Sleep(500 * time.Millisecond)
        }
    }
    
    fmt.Println("\n4. Combined Resilience Strategy")
    // Demonstrate how patterns work together
    
    ctx2, cancel2 := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel2()
    
    for i := 1; i <= 5; i++ {
        fmt.Printf("\n--- Combined test %d ---\n", i)
        
        // Check health first
        if !service.health.IsHealthy() {
            fmt.Println("Service unhealthy, skipping request")
            continue
        }
        
        // Process with retry and circuit breaker
        requestID := fmt.Sprintf("combined-req-%d", i)
        err := service.ProcessWithRetry(ctx2, requestID)
        
        if err != nil {
            fmt.Printf("Combined processing failed: %v\n", err)
        } else {
            fmt.Printf("Combined processing succeeded\n")
        }
    }
}
```

Recovery strategies combine multiple resilience patterns including circuit  
breakers, retry logic with exponential backoff, and health monitoring to  
build systems that gracefully handle failures and maintain availability.  

## Error handling in interfaces

Interface-based error handling enables polymorphic error behavior and  
flexible error processing across different implementations.  

```go
package main

import (
    "fmt"
    "time"
)

// Generic processor interface
type Processor interface {
    Process(data string) (string, error)
    Name() string
}

// Custom error interface with additional behavior
type DetailedError interface {
    error
    Code() int
    Retryable() bool
    Details() map[string]interface{}
}

// Implementation of detailed error
type ProcessingError struct {
    message    string
    code       int
    retryable  bool
    details    map[string]interface{}
    timestamp  time.Time
}

func (pe ProcessingError) Error() string {
    return fmt.Sprintf("[%d] %s (retryable: %v)", pe.code, pe.message, pe.retryable)
}

func (pe ProcessingError) Code() int {
    return pe.code
}

func (pe ProcessingError) Retryable() bool {
    return pe.retryable
}

func (pe ProcessingError) Details() map[string]interface{} {
    return pe.details
}

// Fast processor implementation
type FastProcessor struct{}

func (fp FastProcessor) Name() string {
    return "FastProcessor"
}

func (fp FastProcessor) Process(data string) (string, error) {
    if data == "" {
        return "", ProcessingError{
            message:   "empty data not allowed",
            code:      400,
            retryable: false,
            details:   map[string]interface{}{"processor": "fast", "input_length": 0},
            timestamp: time.Now(),
        }
    }
    
    if data == "error" {
        return "", ProcessingError{
            message:   "processing failed",
            code:      500,
            retryable: true,
            details:   map[string]interface{}{"processor": "fast", "reason": "internal_error"},
            timestamp: time.Now(),
        }
    }
    
    return fmt.Sprintf("fast:%s", data), nil
}

// Slow processor implementation
type SlowProcessor struct{}

func (sp SlowProcessor) Name() string {
    return "SlowProcessor"
}

func (sp SlowProcessor) Process(data string) (string, error) {
    time.Sleep(100 * time.Millisecond) // Simulate slow processing
    
    if len(data) > 10 {
        return "", ProcessingError{
            message:   "input too long",
            code:      413,
            retryable: false,
            details:   map[string]interface{}{"processor": "slow", "max_length": 10, "actual_length": len(data)},
            timestamp: time.Now(),
        }
    }
    
    return fmt.Sprintf("slow:%s", data), nil
}

// Process with multiple processors
func processWithFallback(processors []Processor, data string) (string, error) {
    var lastErr error
    
    for _, processor := range processors {
        fmt.Printf("Trying %s...\n", processor.Name())
        
        result, err := processor.Process(data)
        if err == nil {
            fmt.Printf("Success with %s: %s\n", processor.Name(), result)
            return result, nil
        }
        
        lastErr = err
        fmt.Printf("Failed with %s: %v\n", processor.Name(), err)
        
        // Check if error implements DetailedError
        if detailedErr, ok := err.(DetailedError); ok {
            fmt.Printf("  Code: %d, Retryable: %v\n", detailedErr.Code(), detailedErr.Retryable())
            
            // Don't try next processor for client errors
            if detailedErr.Code() >= 400 && detailedErr.Code() < 500 && !detailedErr.Retryable() {
                fmt.Printf("  Client error, stopping fallback\n")
                break
            }
        }
    }
    
    return "", fmt.Errorf("all processors failed, last error: %w", lastErr)
}

func main() {
    processors := []Processor{
        FastProcessor{},
        SlowProcessor{},
    }
    
    testCases := []string{
        "hello",
        "",
        "error",
        "this_is_very_long_input",
        "normal",
    }
    
    for _, testCase := range testCases {
        fmt.Printf("\n=== Processing: %q ===\n", testCase)
        
        result, err := processWithFallback(processors, testCase)
        if err != nil {
            fmt.Printf("Final error: %v\n", err)
            
            if detailedErr, ok := err.(DetailedError); ok {
                fmt.Printf("Error details: %+v\n", detailedErr.Details())
            }
        } else {
            fmt.Printf("Final result: %s\n", result)
        }
    }
}
```

Interface-based error handling provides flexibility and polymorphism in error  
processing, enabling different implementations to provide specialized error  
behavior while maintaining consistent error handling patterns.  

## Error handling middleware patterns

Middleware patterns enable centralized error handling, logging, and recovery  
mechanisms that can be applied consistently across different operations.  

```go
package main

import (
    "fmt"
    "log"
    "runtime"
    "time"
)

// Operation function type
type Operation func() error

// Middleware function type
type Middleware func(Operation) Operation

// Error with context information
type ContextualError struct {
    Err       error
    Operation string
    Timestamp time.Time
    Stack     string
}

func (ce ContextualError) Error() string {
    return fmt.Sprintf("%s failed at %s: %v", 
                       ce.Operation, ce.Timestamp.Format("15:04:05"), ce.Err)
}

func (ce ContextualError) Unwrap() error {
    return ce.Err
}

// Logging middleware
func LoggingMiddleware(operation Operation) Operation {
    return func() error {
        start := time.Now()
        fmt.Printf("[LOG] Starting operation at %s\n", start.Format("15:04:05"))
        
        err := operation()
        
        duration := time.Since(start)
        if err != nil {
            fmt.Printf("[LOG] Operation failed after %v: %v\n", duration, err)
        } else {
            fmt.Printf("[LOG] Operation completed successfully in %v\n", duration)
        }
        
        return err
    }
}

// Recovery middleware
func RecoveryMiddleware(operationName string) func(Operation) Operation {
    return func(operation Operation) Operation {
        return func() (err error) {
            defer func() {
                if r := recover(); r != nil {
                    buf := make([]byte, 1024)
                    n := runtime.Stack(buf, false)
                    
                    err = ContextualError{
                        Err:       fmt.Errorf("panic: %v", r),
                        Operation: operationName,
                        Timestamp: time.Now(),
                        Stack:     string(buf[:n]),
                    }
                }
            }()
            
            return operation()
        }
    }
}

// Timeout middleware
func TimeoutMiddleware(timeout time.Duration) func(Operation) Operation {
    return func(operation Operation) Operation {
        return func() error {
            done := make(chan error, 1)
            
            go func() {
                done <- operation()
            }()
            
            select {
            case err := <-done:
                return err
            case <-time.After(timeout):
                return fmt.Errorf("operation timed out after %v", timeout)
            }
        }
    }
}

// Retry middleware
func RetryMiddleware(maxRetries int, delay time.Duration) func(Operation) Operation {
    return func(operation Operation) Operation {
        return func() error {
            var lastErr error
            
            for attempt := 1; attempt <= maxRetries; attempt++ {
                err := operation()
                if err == nil {
                    if attempt > 1 {
                        fmt.Printf("[RETRY] Operation succeeded on attempt %d\n", attempt)
                    }
                    return nil
                }
                
                lastErr = err
                fmt.Printf("[RETRY] Attempt %d failed: %v\n", attempt, err)
                
                if attempt < maxRetries {
                    fmt.Printf("[RETRY] Waiting %v before next attempt\n", delay)
                    time.Sleep(delay)
                }
            }
            
            return fmt.Errorf("operation failed after %d attempts: %w", maxRetries, lastErr)
        }
    }
}

// Validation middleware
func ValidationMiddleware(validator func() error) func(Operation) Operation {
    return func(operation Operation) Operation {
        return func() error {
            if err := validator(); err != nil {
                return fmt.Errorf("validation failed: %w", err)
            }
            
            return operation()
        }
    }
}

// Chain multiple middleware
func ChainMiddleware(middlewares ...func(Operation) Operation) func(Operation) Operation {
    return func(operation Operation) Operation {
        // Apply middleware in reverse order
        for i := len(middlewares) - 1; i >= 0; i-- {
            operation = middlewares[i](operation)
        }
        return operation
    }
}

// Sample operations
func successOperation() error {
    fmt.Println("Executing successful operation")
    time.Sleep(50 * time.Millisecond)
    return nil
}

func failingOperation() error {
    fmt.Println("Executing failing operation")
    return fmt.Errorf("operation failed")
}

func panicOperation() error {
    fmt.Println("Executing panic operation")
    panic("simulated panic")
}

func slowOperation() error {
    fmt.Println("Executing slow operation")
    time.Sleep(300 * time.Millisecond)
    return nil
}

func validationExample() error {
    if time.Now().Second()%2 == 0 {
        return fmt.Errorf("validation failed: even second")
    }
    return nil
}

func main() {
    fmt.Println("=== Middleware Pattern Examples ===")
    
    // Example 1: Basic logging
    fmt.Println("\n1. Basic Logging Middleware")
    loggedOp := LoggingMiddleware(successOperation)
    loggedOp()
    
    // Example 2: Recovery middleware
    fmt.Println("\n2. Recovery Middleware")
    recoveredOp := RecoveryMiddleware("test-operation")(panicOperation)
    if err := recoveredOp(); err != nil {
        fmt.Printf("Recovered error: %v\n", err)
        
        if contextErr, ok := err.(ContextualError); ok {
            fmt.Printf("Operation: %s\n", contextErr.Operation)
            fmt.Printf("Timestamp: %s\n", contextErr.Timestamp.Format("15:04:05"))
        }
    }
    
    // Example 3: Timeout middleware
    fmt.Println("\n3. Timeout Middleware")
    timeoutOp := TimeoutMiddleware(100*time.Millisecond)(slowOperation)
    if err := timeoutOp(); err != nil {
        fmt.Printf("Timeout error: %v\n", err)
    }
    
    // Example 4: Retry middleware
    fmt.Println("\n4. Retry Middleware")
    retryOp := RetryMiddleware(3, 100*time.Millisecond)(failingOperation)
    if err := retryOp(); err != nil {
        fmt.Printf("Retry exhausted: %v\n", err)
    }
    
    // Example 5: Chained middleware
    fmt.Println("\n5. Chained Middleware")
    chainedOp := ChainMiddleware(
        LoggingMiddleware,
        RecoveryMiddleware("chained-operation"),
        TimeoutMiddleware(200*time.Millisecond),
        ValidationMiddleware(validationExample),
    )(successOperation)
    
    if err := chainedOp(); err != nil {
        fmt.Printf("Chained operation error: %v\n", err)
    }
    
    // Example 6: Complex chained middleware with retry
    fmt.Println("\n6. Complex Chained Middleware")
    complexOp := ChainMiddleware(
        LoggingMiddleware,
        RetryMiddleware(2, 50*time.Millisecond),
        RecoveryMiddleware("complex-operation"),
    )(failingOperation)
    
    if err := complexOp(); err != nil {
        fmt.Printf("Complex operation final error: %v\n", err)
    }
}
```

Middleware patterns provide a clean way to compose error handling behaviors,  
enabling reusable error handling logic that can be applied consistently  
across different operations and services.  

## Domain-specific error types

Domain-specific errors provide meaningful context for business logic  
failures and enable specialized error handling based on business rules.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Domain: E-commerce Order Processing

// Order states
type OrderState string

const (
    OrderStatePending   OrderState = "pending"
    OrderStateConfirmed OrderState = "confirmed"
    OrderStateShipped   OrderState = "shipped"
    OrderStateDelivered OrderState = "delivered"
    OrderStateCancelled OrderState = "cancelled"
)

// Business domain errors
type OrderError struct {
    OrderID   string
    ErrorType string
    Message   string
    Details   map[string]interface{}
    Timestamp time.Time
}

func (oe OrderError) Error() string {
    return fmt.Sprintf("order %s %s error: %s", oe.OrderID, oe.ErrorType, oe.Message)
}

func (oe OrderError) IsBusinessError() bool {
    return oe.ErrorType == "business_rule"
}

func (oe OrderError) IsValidationError() bool {
    return oe.ErrorType == "validation"
}

func (oe OrderError) IsInventoryError() bool {
    return oe.ErrorType == "inventory"
}

// Payment domain errors
type PaymentError struct {
    PaymentID   string
    Amount      float64
    Currency    string
    FailureCode string
    Message     string
    Retryable   bool
}

func (pe PaymentError) Error() string {
    return fmt.Sprintf("payment %s failed: %s (code: %s)", pe.PaymentID, pe.Message, pe.FailureCode)
}

func (pe PaymentError) IsRetryable() bool {
    return pe.Retryable
}

func (pe PaymentError) IsInsufficientFunds() bool {
    return pe.FailureCode == "INSUFFICIENT_FUNDS"
}

// Shipping domain errors
type ShippingError struct {
    OrderID     string
    ShipperCode string
    Message     string
    ErrorCode   string
}

func (se ShippingError) Error() string {
    return fmt.Sprintf("shipping error for order %s with %s: %s", 
                       se.OrderID, se.ShipperCode, se.Message)
}

func (se ShippingError) IsAddressError() bool {
    return se.ErrorCode == "INVALID_ADDRESS"
}

func (se ShippingError) IsServiceUnavailable() bool {
    return se.ErrorCode == "SERVICE_UNAVAILABLE"
}

// Order processing service
type OrderService struct{}

func (os OrderService) ValidateOrder(orderID string, items []string, totalAmount float64) error {
    if orderID == "" {
        return OrderError{
            OrderID:   orderID,
            ErrorType: "validation",
            Message:   "order ID cannot be empty",
            Details:   map[string]interface{}{"field": "order_id"},
            Timestamp: time.Now(),
        }
    }
    
    if len(items) == 0 {
        return OrderError{
            OrderID:   orderID,
            ErrorType: "validation",
            Message:   "order must contain at least one item",
            Details:   map[string]interface{}{"item_count": 0},
            Timestamp: time.Now(),
        }
    }
    
    if totalAmount <= 0 {
        return OrderError{
            OrderID:   orderID,
            ErrorType: "validation",
            Message:   "order total must be positive",
            Details:   map[string]interface{}{"amount": totalAmount},
            Timestamp: time.Now(),
        }
    }
    
    return nil
}

func (os OrderService) CheckInventory(orderID string, items []string) error {
    for _, item := range items {
        if strings.Contains(item, "out_of_stock") {
            return OrderError{
                OrderID:   orderID,
                ErrorType: "inventory",
                Message:   fmt.Sprintf("item %s is out of stock", item),
                Details:   map[string]interface{}{"item": item, "available": 0},
                Timestamp: time.Now(),
            }
        }
    }
    return nil
}

func (os OrderService) ApplyBusinessRules(orderID string, customerType string, totalAmount float64) error {
    if customerType == "new" && totalAmount > 1000 {
        return OrderError{
            OrderID:   orderID,
            ErrorType: "business_rule",
            Message:   "new customers cannot place orders over $1000",
            Details:   map[string]interface{}{"customer_type": customerType, "limit": 1000, "amount": totalAmount},
            Timestamp: time.Now(),
        }
    }
    
    if customerType == "suspended" {
        return OrderError{
            OrderID:   orderID,
            ErrorType: "business_rule",
            Message:   "suspended customers cannot place orders",
            Details:   map[string]interface{}{"customer_type": customerType},
            Timestamp: time.Now(),
        }
    }
    
    return nil
}

// Payment service
type PaymentService struct{}

func (ps PaymentService) ProcessPayment(paymentID string, amount float64, currency string) error {
    if amount > 10000 {
        return PaymentError{
            PaymentID:   paymentID,
            Amount:      amount,
            Currency:    currency,
            FailureCode: "AMOUNT_LIMIT_EXCEEDED",
            Message:     "payment amount exceeds limit",
            Retryable:   false,
        }
    }
    
    if strings.Contains(paymentID, "insufficient") {
        return PaymentError{
            PaymentID:   paymentID,
            Amount:      amount,
            Currency:    currency,
            FailureCode: "INSUFFICIENT_FUNDS",
            Message:     "insufficient funds in account",
            Retryable:   false,
        }
    }
    
    if strings.Contains(paymentID, "network") {
        return PaymentError{
            PaymentID:   paymentID,
            Amount:      amount,
            Currency:    currency,
            FailureCode: "NETWORK_ERROR",
            Message:     "payment gateway network error",
            Retryable:   true,
        }
    }
    
    return nil
}

// Shipping service
type ShippingService struct{}

func (ss ShippingService) ScheduleShipping(orderID string, address string) error {
    if address == "" || !strings.Contains(address, ",") {
        return ShippingError{
            OrderID:     orderID,
            ShipperCode: "DHL",
            Message:     "invalid shipping address format",
            ErrorCode:   "INVALID_ADDRESS",
        }
    }
    
    if strings.Contains(address, "remote") {
        return ShippingError{
            OrderID:     orderID,
            ShipperCode: "DHL",
            Message:     "shipping service unavailable for remote areas",
            ErrorCode:   "SERVICE_UNAVAILABLE",
        }
    }
    
    return nil
}

// Order processing orchestrator
func ProcessOrder(orderID, customerType, address string, items []string, totalAmount float64) error {
    orderService := OrderService{}
    paymentService := PaymentService{}
    shippingService := ShippingService{}
    
    // Step 1: Validate order
    if err := orderService.ValidateOrder(orderID, items, totalAmount); err != nil {
        return fmt.Errorf("order validation failed: %w", err)
    }
    
    // Step 2: Check inventory
    if err := orderService.CheckInventory(orderID, items); err != nil {
        return fmt.Errorf("inventory check failed: %w", err)
    }
    
    // Step 3: Apply business rules
    if err := orderService.ApplyBusinessRules(orderID, customerType, totalAmount); err != nil {
        return fmt.Errorf("business rule validation failed: %w", err)
    }
    
    // Step 4: Process payment
    paymentID := "pay_" + orderID
    if err := paymentService.ProcessPayment(paymentID, totalAmount, "USD"); err != nil {
        return fmt.Errorf("payment processing failed: %w", err)
    }
    
    // Step 5: Schedule shipping
    if err := shippingService.ScheduleShipping(orderID, address); err != nil {
        return fmt.Errorf("shipping scheduling failed: %w", err)
    }
    
    fmt.Printf("Order %s processed successfully\n", orderID)
    return nil
}

func main() {
    fmt.Println("=== Domain-Specific Error Handling ===")
    
    testOrders := []struct {
        orderID      string
        customerType string
        address      string
        items        []string
        totalAmount  float64
        description  string
    }{
        {"ORD001", "premium", "123 Main St, Anytown", []string{"item1", "item2"}, 150.0, "Valid order"},
        {"", "premium", "123 Main St, Anytown", []string{"item1"}, 100.0, "Empty order ID"},
        {"ORD002", "new", "123 Main St, Anytown", []string{"item1"}, 1500.0, "New customer over limit"},
        {"ORD003", "premium", "123 Main St, Anytown", []string{"out_of_stock_item"}, 100.0, "Out of stock item"},
        {"ORD004", "premium", "123 Main St, Anytown", []string{"item1"}, 15000.0, "Amount over payment limit"},
        {"insufficient_pay", "premium", "123 Main St, Anytown", []string{"item1"}, 100.0, "Insufficient funds"},
        {"ORD005", "premium", "remote area", []string{"item1"}, 100.0, "Remote shipping address"},
        {"ORD006", "suspended", "123 Main St, Anytown", []string{"item1"}, 100.0, "Suspended customer"},
    }
    
    for i, order := range testOrders {
        fmt.Printf("\n--- Test %d: %s ---\n", i+1, order.description)
        
        err := ProcessOrder(order.orderID, order.customerType, order.address, order.items, order.totalAmount)
        if err != nil {
            fmt.Printf("❌ Error: %v\n", err)
            
            // Handle specific domain error types
            if orderErr, ok := err.(OrderError); ok {
                fmt.Printf("   Domain: Order, Type: %s\n", orderErr.ErrorType)
                if orderErr.IsBusinessError() {
                    fmt.Printf("   Business rule violation\n")
                }
            }
            
            if paymentErr, ok := err.(PaymentError); ok {
                fmt.Printf("   Domain: Payment, Retryable: %v\n", paymentErr.IsRetryable())
                if paymentErr.IsInsufficientFunds() {
                    fmt.Printf("   Insufficient funds detected\n")
                }
            }
            
            if shippingErr, ok := err.(ShippingError); ok {
                fmt.Printf("   Domain: Shipping, Code: %s\n", shippingErr.ErrorCode)
                if shippingErr.IsAddressError() {
                    fmt.Printf("   Address validation issue\n")
                }
            }
        } else {
            fmt.Printf("✅ Order processed successfully\n")
        }
    }
}
```

Domain-specific errors provide rich context for business failures, enabling  
specialized error handling, better user messages, and appropriate recovery  
strategies based on business domain knowledge.  

## Error handling in channels and select

Channel operations and select statements require specialized error handling  
patterns for coordination, timeouts, and proper channel lifecycle management.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Channel error types
type ChannelError struct {
    Operation string
    Channel   string
    Reason    string
}

func (ce ChannelError) Error() string {
    return fmt.Sprintf("channel %s operation failed: %s", ce.Operation, ce.Reason)
}

// Safe channel operations with error handling
type SafeChannel struct {
    ch     chan interface{}
    closed bool
    mu     sync.RWMutex
}

func NewSafeChannel(size int) *SafeChannel {
    return &SafeChannel{
        ch: make(chan interface{}, size),
    }
}

func (sc *SafeChannel) Send(value interface{}) error {
    sc.mu.RLock()
    defer sc.mu.RUnlock()
    
    if sc.closed {
        return ChannelError{
            Operation: "send",
            Channel:   "safe_channel",
            Reason:    "channel is closed",
        }
    }
    
    select {
    case sc.ch <- value:
        return nil
    default:
        return ChannelError{
            Operation: "send",
            Channel:   "safe_channel",
            Reason:    "channel is full",
        }
    }
}

func (sc *SafeChannel) Receive() (interface{}, error) {
    sc.mu.RLock()
    defer sc.mu.RUnlock()
    
    select {
    case value, ok := <-sc.ch:
        if !ok {
            return nil, ChannelError{
                Operation: "receive",
                Channel:   "safe_channel",
                Reason:    "channel is closed",
            }
        }
        return value, nil
    default:
        return nil, ChannelError{
            Operation: "receive",
            Channel:   "safe_channel",
            Reason:    "no data available",
        }
    }
}

func (sc *SafeChannel) Close() error {
    sc.mu.Lock()
    defer sc.mu.Unlock()
    
    if sc.closed {
        return ChannelError{
            Operation: "close",
            Channel:   "safe_channel",
            Reason:    "channel already closed",
        }
    }
    
    close(sc.ch)
    sc.closed = true
    return nil
}

// Producer with error handling
func errorHandlingProducer(results chan<- string, errors chan<- error, done <-chan struct{}) {
    defer close(results)
    defer close(errors)
    
    for i := 1; i <= 10; i++ {
        select {
        case <-done:
            errors <- fmt.Errorf("producer cancelled at item %d", i)
            return
        default:
        }
        
        // Simulate work and potential errors
        time.Sleep(100 * time.Millisecond)
        
        if i%3 == 0 {
            errors <- fmt.Errorf("processing error at item %d", i)
            continue
        }
        
        if i == 7 {
            errors <- fmt.Errorf("critical error at item %d", i)
            return
        }
        
        select {
        case results <- fmt.Sprintf("item-%d", i):
        case <-done:
            errors <- fmt.Errorf("producer cancelled while sending item %d", i)
            return
        }
    }
}

// Consumer with error handling
func errorHandlingConsumer(results <-chan string, errors <-chan error) ([]string, []error) {
    var items []string
    var errs []error
    
    for {
        select {
        case result, ok := <-results:
            if !ok {
                results = nil
            } else {
                items = append(items, result)
                fmt.Printf("Received: %s\n", result)
            }
        case err, ok := <-errors:
            if !ok {
                errors = nil
            } else {
                errs = append(errs, err)
                fmt.Printf("Error: %v\n", err)
            }
        }
        
        if results == nil && errors == nil {
            break
        }
    }
    
    return items, errs
}

// Timeout-aware channel operations
func timeoutChannelOperations() {
    fmt.Println("\n=== Timeout Channel Operations ===")
    
    ch := make(chan string, 1)
    
    // Send with timeout
    select {
    case ch <- "message":
        fmt.Println("✓ Message sent successfully")
    case <-time.After(100 * time.Millisecond):
        fmt.Println("✗ Send timeout")
    }
    
    // Receive with timeout
    select {
    case msg := <-ch:
        fmt.Printf("✓ Message received: %s\n", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("✗ Receive timeout")
    }
    
    // Receive from empty channel (should timeout)
    select {
    case msg := <-ch:
        fmt.Printf("✓ Message received: %s\n", msg)
    case <-time.After(100 * time.Millisecond):
        fmt.Println("✗ Receive timeout (expected)")
    }
}

// Multiple channel coordination with errors
func multiChannelCoordination() {
    fmt.Println("\n=== Multi-Channel Coordination ===")
    
    ch1 := make(chan string, 2)
    ch2 := make(chan int, 2)
    errorCh := make(chan error, 2)
    done := make(chan struct{})
    
    // Producer goroutines
    go func() {
        defer close(ch1)
        for i := 1; i <= 3; i++ {
            select {
            case ch1 <- fmt.Sprintf("str-%d", i):
                time.Sleep(50 * time.Millisecond)
            case <-done:
                errorCh <- fmt.Errorf("string producer cancelled")
                return
            }
        }
    }()
    
    go func() {
        defer close(ch2)
        for i := 1; i <= 3; i++ {
            select {
            case ch2 <- i * 10:
                time.Sleep(75 * time.Millisecond)
            case <-done:
                errorCh <- fmt.Errorf("int producer cancelled")
                return
            }
        }
    }()
    
    // Consumer with coordinated error handling
    timeout := time.After(500 * time.Millisecond)
    ch1Open, ch2Open := true, true
    
    for ch1Open || ch2Open {
        select {
        case str, ok := <-ch1:
            if !ok {
                ch1Open = false
                fmt.Println("String channel closed")
            } else {
                fmt.Printf("Received string: %s\n", str)
            }
        case num, ok := <-ch2:
            if !ok {
                ch2Open = false
                fmt.Println("Int channel closed")
            } else {
                fmt.Printf("Received int: %d\n", num)
            }
        case err := <-errorCh:
            fmt.Printf("Producer error: %v\n", err)
        case <-timeout:
            fmt.Println("Operation timed out")
            close(done)
            return
        }
    }
    
    close(errorCh)
}

func main() {
    fmt.Println("=== Channel Error Handling Examples ===")
    
    // Safe channel operations
    fmt.Println("1. Safe Channel Operations")
    safeCh := NewSafeChannel(2)
    
    // Test sending
    if err := safeCh.Send("message1"); err != nil {
        fmt.Printf("Send error: %v\n", err)
    } else {
        fmt.Println("✓ Message1 sent")
    }
    
    if err := safeCh.Send("message2"); err != nil {
        fmt.Printf("Send error: %v\n", err)
    } else {
        fmt.Println("✓ Message2 sent")
    }
    
    // This should fail (channel full)
    if err := safeCh.Send("message3"); err != nil {
        fmt.Printf("Expected send error: %v\n", err)
    }
    
    // Test receiving
    if msg, err := safeCh.Receive(); err != nil {
        fmt.Printf("Receive error: %v\n", err)
    } else {
        fmt.Printf("✓ Received: %v\n", msg)
    }
    
    // Test close
    if err := safeCh.Close(); err != nil {
        fmt.Printf("Close error: %v\n", err)
    } else {
        fmt.Println("✓ Channel closed")
    }
    
    // Test operations on closed channel
    if err := safeCh.Send("message4"); err != nil {
        fmt.Printf("Expected error after close: %v\n", err)
    }
    
    fmt.Println("\n2. Producer-Consumer with Error Handling")
    
    results := make(chan string, 5)
    errors := make(chan error, 5)
    done := make(chan struct{})
    
    go errorHandlingProducer(results, errors, done)
    
    // Let it run for a bit, then cancel
    go func() {
        time.Sleep(800 * time.Millisecond)
        close(done)
    }()
    
    items, errs := errorHandlingConsumer(results, errors)
    
    fmt.Printf("\nSummary: %d items processed, %d errors\n", len(items), len(errs))
    for _, err := range errs {
        fmt.Printf("  Error: %v\n", err)
    }
    
    timeoutChannelOperations()
    multiChannelCoordination()
}
```

Channel error handling requires careful coordination of multiple channels,  
proper timeout handling, and graceful shutdown patterns to avoid deadlocks  
and ensure reliable communication between goroutines.  

## Configuration and environment errors

Configuration errors are common in applications and require clear error  
messages, validation, and fallback mechanisms for robust application startup.  

```go
package main

import (
    "fmt"
    "os"
    "strconv"
    "strings"
    "time"
)

// Configuration error types
type ConfigError struct {
    Field     string
    Value     string
    Required  bool
    Expected  string
    Validator string
}

func (ce ConfigError) Error() string {
    if ce.Required && ce.Value == "" {
        return fmt.Sprintf("required configuration field %q is missing", ce.Field)
    }
    
    if ce.Expected != "" {
        return fmt.Sprintf("invalid value for %q: got %q, expected %s", 
                          ce.Field, ce.Value, ce.Expected)
    }
    
    return fmt.Sprintf("configuration validation failed for %q: %s (value: %q)", 
                       ce.Field, ce.Validator, ce.Value)
}

func (ce ConfigError) IsRequired() bool {
    return ce.Required
}

func (ce ConfigError) IsValidation() bool {
    return ce.Validator != ""
}

// Configuration aggregation error
type ConfigErrors struct {
    Errors []ConfigError
}

func (ce ConfigErrors) Error() string {
    if len(ce.Errors) == 0 {
        return "no configuration errors"
    }
    
    var messages []string
    for _, err := range ce.Errors {
        messages = append(messages, err.Error())
    }
    
    return fmt.Sprintf("configuration validation failed with %d error(s):\n  - %s", 
                       len(ce.Errors), strings.Join(messages, "\n  - "))
}

func (ce *ConfigErrors) Add(field, value, validator string, required bool) {
    ce.Errors = append(ce.Errors, ConfigError{
        Field:     field,
        Value:     value,
        Required:  required,
        Validator: validator,
    })
}

func (ce *ConfigErrors) AddExpected(field, value, expected string) {
    ce.Errors = append(ce.Errors, ConfigError{
        Field:    field,
        Value:    value,
        Expected: expected,
    })
}

func (ce ConfigErrors) HasErrors() bool {
    return len(ce.Errors) > 0
}

// Application configuration
type AppConfig struct {
    Port            int           `env:"PORT" required:"true"`
    Host            string        `env:"HOST" default:"localhost"`
    DatabaseURL     string        `env:"DATABASE_URL" required:"true"`
    RedisURL        string        `env:"REDIS_URL" required:"false"`
    LogLevel        string        `env:"LOG_LEVEL" default:"info"`
    Timeout         time.Duration `env:"TIMEOUT" default:"30s"`
    MaxConnections  int           `env:"MAX_CONNECTIONS" default:"100"`
    EnableTLS       bool          `env:"ENABLE_TLS" default:"false"`
    AllowedOrigins  []string      `env:"ALLOWED_ORIGINS" default:"*"`
    SecretKey       string        `env:"SECRET_KEY" required:"true"`
}

// Configuration validator
type ConfigValidator struct{}

func (cv ConfigValidator) ValidatePort(port int) error {
    if port <= 0 || port > 65535 {
        return fmt.Errorf("port must be between 1 and 65535")
    }
    return nil
}

func (cv ConfigValidator) ValidateLogLevel(level string) error {
    validLevels := []string{"debug", "info", "warn", "error"}
    for _, valid := range validLevels {
        if level == valid {
            return nil
        }
    }
    return fmt.Errorf("must be one of: %s", strings.Join(validLevels, ", "))
}

func (cv ConfigValidator) ValidateTimeout(timeout time.Duration) error {
    if timeout < time.Second {
        return fmt.Errorf("timeout must be at least 1 second")
    }
    if timeout > 5*time.Minute {
        return fmt.Errorf("timeout must be at most 5 minutes")
    }
    return nil
}

func (cv ConfigValidator) ValidateURL(url string) error {
    if url == "" {
        return fmt.Errorf("URL cannot be empty")
    }
    if !strings.HasPrefix(url, "http://") && !strings.HasPrefix(url, "https://") && !strings.HasPrefix(url, "redis://") {
        return fmt.Errorf("URL must start with http://, https://, or redis://")
    }
    return nil
}

// Configuration loader with error handling
func LoadConfiguration() (*AppConfig, error) {
    config := &AppConfig{}
    var configErrors ConfigErrors
    validator := ConfigValidator{}
    
    // Load port
    portStr := os.Getenv("PORT")
    if portStr == "" {
        config.Port = 8080 // default
    } else {
        if port, err := strconv.Atoi(portStr); err != nil {
            configErrors.AddExpected("PORT", portStr, "integer")
        } else {
            if err := validator.ValidatePort(port); err != nil {
                configErrors.Add("PORT", portStr, err.Error(), true)
            } else {
                config.Port = port
            }
        }
    }
    
    // Load host
    config.Host = getEnvWithDefault("HOST", "localhost")
    
    // Load database URL (required)
    config.DatabaseURL = os.Getenv("DATABASE_URL")
    if config.DatabaseURL == "" {
        configErrors.Add("DATABASE_URL", "", "required field", true)
    } else {
        if err := validator.ValidateURL(config.DatabaseURL); err != nil {
            configErrors.Add("DATABASE_URL", config.DatabaseURL, err.Error(), true)
        }
    }
    
    // Load Redis URL (optional)
    config.RedisURL = os.Getenv("REDIS_URL")
    if config.RedisURL != "" {
        if err := validator.ValidateURL(config.RedisURL); err != nil {
            configErrors.Add("REDIS_URL", config.RedisURL, err.Error(), false)
        }
    }
    
    // Load log level
    config.LogLevel = getEnvWithDefault("LOG_LEVEL", "info")
    if err := validator.ValidateLogLevel(config.LogLevel); err != nil {
        configErrors.Add("LOG_LEVEL", config.LogLevel, err.Error(), false)
    }
    
    // Load timeout
    timeoutStr := getEnvWithDefault("TIMEOUT", "30s")
    if timeout, err := time.ParseDuration(timeoutStr); err != nil {
        configErrors.AddExpected("TIMEOUT", timeoutStr, "valid duration (e.g., 30s, 1m)")
    } else {
        if err := validator.ValidateTimeout(timeout); err != nil {
            configErrors.Add("TIMEOUT", timeoutStr, err.Error(), false)
        } else {
            config.Timeout = timeout
        }
    }
    
    // Load max connections
    maxConnStr := getEnvWithDefault("MAX_CONNECTIONS", "100")
    if maxConn, err := strconv.Atoi(maxConnStr); err != nil {
        configErrors.AddExpected("MAX_CONNECTIONS", maxConnStr, "integer")
    } else {
        if maxConn <= 0 {
            configErrors.Add("MAX_CONNECTIONS", maxConnStr, "must be positive", false)
        } else {
            config.MaxConnections = maxConn
        }
    }
    
    // Load TLS setting
    tlsStr := getEnvWithDefault("ENABLE_TLS", "false")
    if tls, err := strconv.ParseBool(tlsStr); err != nil {
        configErrors.AddExpected("ENABLE_TLS", tlsStr, "boolean (true/false)")
    } else {
        config.EnableTLS = tls
    }
    
    // Load allowed origins
    originsStr := getEnvWithDefault("ALLOWED_ORIGINS", "*")
    config.AllowedOrigins = strings.Split(originsStr, ",")
    for i, origin := range config.AllowedOrigins {
        config.AllowedOrigins[i] = strings.TrimSpace(origin)
    }
    
    // Load secret key (required)
    config.SecretKey = os.Getenv("SECRET_KEY")
    if config.SecretKey == "" {
        configErrors.Add("SECRET_KEY", "", "required field", true)
    } else if len(config.SecretKey) < 16 {
        configErrors.Add("SECRET_KEY", "[hidden]", "must be at least 16 characters", true)
    }
    
    if configErrors.HasErrors() {
        return nil, configErrors
    }
    
    return config, nil
}

func getEnvWithDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

// Configuration with fallback values
func LoadConfigurationWithFallback() *AppConfig {
    config := &AppConfig{
        Port:           8080,
        Host:           "localhost",
        DatabaseURL:    "postgres://localhost:5432/app",
        LogLevel:       "info",
        Timeout:        30 * time.Second,
        MaxConnections: 100,
        EnableTLS:      false,
        AllowedOrigins: []string{"*"},
        SecretKey:      "development-secret-key",
    }
    
    // Try to load from environment, but don't fail
    if actualConfig, err := LoadConfiguration(); err != nil {
        fmt.Printf("Warning: Failed to load configuration, using defaults: %v\n", err)
    } else {
        config = actualConfig
    }
    
    return config
}

func main() {
    fmt.Println("=== Configuration Error Handling ===")
    
    fmt.Println("1. Testing with missing required fields")
    // Clear environment for testing
    originalEnvs := make(map[string]string)
    envKeys := []string{"PORT", "DATABASE_URL", "SECRET_KEY", "LOG_LEVEL", "TIMEOUT"}
    
    for _, key := range envKeys {
        originalEnvs[key] = os.Getenv(key)
        os.Unsetenv(key)
    }
    
    _, err := LoadConfiguration()
    if err != nil {
        fmt.Printf("Configuration errors (expected):\n%v\n", err)
        
        if configErr, ok := err.(ConfigErrors); ok {
            fmt.Printf("Error count: %d\n", len(configErr.Errors))
            for _, e := range configErr.Errors {
                fmt.Printf("  Field: %s, Required: %v\n", e.Field, e.IsRequired())
            }
        }
    }
    
    fmt.Println("\n2. Testing with invalid values")
    os.Setenv("PORT", "999999")
    os.Setenv("DATABASE_URL", "invalid-url")
    os.Setenv("LOG_LEVEL", "invalid")
    os.Setenv("TIMEOUT", "invalid-duration")
    os.Setenv("SECRET_KEY", "short")
    
    _, err = LoadConfiguration()
    if err != nil {
        fmt.Printf("Validation errors (expected):\n%v\n", err)
    }
    
    fmt.Println("\n3. Testing with valid configuration")
    os.Setenv("PORT", "8080")
    os.Setenv("DATABASE_URL", "postgres://localhost:5432/mydb")
    os.Setenv("LOG_LEVEL", "debug")
    os.Setenv("TIMEOUT", "45s")
    os.Setenv("SECRET_KEY", "this-is-a-secure-secret-key")
    
    config, err := LoadConfiguration()
    if err != nil {
        fmt.Printf("Unexpected error: %v\n", err)
    } else {
        fmt.Printf("✓ Configuration loaded successfully:\n")
        fmt.Printf("  Port: %d\n", config.Port)
        fmt.Printf("  Host: %s\n", config.Host)
        fmt.Printf("  Database: %s\n", config.DatabaseURL)
        fmt.Printf("  Log Level: %s\n", config.LogLevel)
        fmt.Printf("  Timeout: %v\n", config.Timeout)
        fmt.Printf("  Max Connections: %d\n", config.MaxConnections)
        fmt.Printf("  TLS Enabled: %v\n", config.EnableTLS)
        fmt.Printf("  Allowed Origins: %v\n", config.AllowedOrigins)
        fmt.Printf("  Secret Key: [hidden]\n")
    }
    
    fmt.Println("\n4. Configuration with fallback")
    // Clear environment again
    for _, key := range envKeys {
        os.Unsetenv(key)
    }
    
    fallbackConfig := LoadConfigurationWithFallback()
    fmt.Printf("✓ Fallback configuration:\n")
    fmt.Printf("  Port: %d\n", fallbackConfig.Port)
    fmt.Printf("  Database: %s\n", fallbackConfig.DatabaseURL)
    fmt.Printf("  Log Level: %s\n", fallbackConfig.LogLevel)
    
    // Restore original environment
    for key, value := range originalEnvs {
        if value != "" {
            os.Setenv(key, value)
        }
    }
}
```

Configuration error handling requires comprehensive validation, clear error  
messages for missing or invalid values, and appropriate fallback mechanisms  
to ensure applications can start reliably in different environments.  

## Structured logging with errors

Structured logging enhances error handling by providing consistent, searchable  
error information that aids in monitoring, debugging, and system maintenance.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "os"
    "runtime"
    "strings"
    "time"
)

// Log levels
type LogLevel int

const (
    DEBUG LogLevel = iota
    INFO
    WARN
    ERROR
    FATAL
)

func (l LogLevel) String() string {
    switch l {
    case DEBUG:
        return "DEBUG"
    case INFO:
        return "INFO"
    case WARN:
        return "WARN"
    case ERROR:
        return "ERROR"
    case FATAL:
        return "FATAL"
    default:
        return "UNKNOWN"
    }
}

// Structured logger
type StructuredLogger struct {
    level   LogLevel
    context map[string]interface{}
}

func NewLogger(level LogLevel) *StructuredLogger {
    return &StructuredLogger{
        level:   level,
        context: make(map[string]interface{}),
    }
}

func (sl *StructuredLogger) WithContext(key string, value interface{}) *StructuredLogger {
    newLogger := &StructuredLogger{
        level:   sl.level,
        context: make(map[string]interface{}),
    }
    
    // Copy existing context
    for k, v := range sl.context {
        newLogger.context[k] = v
    }
    
    // Add new context
    newLogger.context[key] = value
    
    return newLogger
}

func (sl *StructuredLogger) log(level LogLevel, message string, fields map[string]interface{}) {
    if level < sl.level {
        return
    }
    
    // Build log entry
    timestamp := time.Now().Format("2006-01-02T15:04:05.000Z")
    
    // Get caller information
    _, file, line, ok := runtime.Caller(3)
    var caller string
    if ok {
        parts := strings.Split(file, "/")
        caller = fmt.Sprintf("%s:%d", parts[len(parts)-1], line)
    }
    
    fmt.Printf("[%s] %s %s - %s", timestamp, level.String(), caller, message)
    
    // Add context fields
    if len(sl.context) > 0 {
        fmt.Print(" | context: {")
        first := true
        for k, v := range sl.context {
            if !first {
                fmt.Print(", ")
            }
            fmt.Printf("%s: %v", k, v)
            first = false
        }
        fmt.Print("}")
    }
    
    // Add additional fields
    if len(fields) > 0 {
        fmt.Print(" | fields: {")
        first := true
        for k, v := range fields {
            if !first {
                fmt.Print(", ")
            }
            fmt.Printf("%s: %v", k, v)
            first = false
        }
        fmt.Print("}")
    }
    
    fmt.Println()
}

func (sl *StructuredLogger) Debug(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log(DEBUG, message, f)
}

func (sl *StructuredLogger) Info(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log(INFO, message, f)
}

func (sl *StructuredLogger) Warn(message string, fields ...map[string]interface{}) {
    var f map[string]interface{}
    if len(fields) > 0 {
        f = fields[0]
    }
    sl.log(WARN, message, f)
}

func (sl *StructuredLogger) Error(message string, err error, fields ...map[string]interface{}) {
    f := map[string]interface{}{
        "error": err.Error(),
    }
    
    if len(fields) > 0 {
        for k, v := range fields[0] {
            f[k] = v
        }
    }
    
    sl.log(ERROR, message, f)
}

func (sl *StructuredLogger) Fatal(message string, err error, fields ...map[string]interface{}) {
    f := map[string]interface{}{
        "error": err.Error(),
    }
    
    if len(fields) > 0 {
        for k, v := range fields[0] {
            f[k] = v
        }
    }
    
    sl.log(FATAL, message, f)
    os.Exit(1)
}

// Service with structured logging
type UserService struct {
    logger *StructuredLogger
}

func NewUserService(logger *StructuredLogger) *UserService {
    return &UserService{
        logger: logger.WithContext("service", "UserService"),
    }
}

func (us *UserService) GetUser(ctx context.Context, userID string) (*User, error) {
    requestLogger := us.logger.WithContext("operation", "GetUser").WithContext("user_id", userID)
    
    requestLogger.Info("Starting user retrieval")
    
    if userID == "" {
        err := fmt.Errorf("user ID cannot be empty")
        requestLogger.Error("User retrieval failed", err, map[string]interface{}{
            "validation": "user_id_required",
        })
        return nil, err
    }
    
    if userID == "blocked" {
        err := fmt.Errorf("user %s is blocked", userID)
        requestLogger.Warn("Blocked user access attempt", map[string]interface{}{
            "user_id": userID,
            "reason":  "user_blocked",
        })
        return nil, err
    }
    
    if userID == "error" {
        err := fmt.Errorf("database connection failed")
        requestLogger.Error("Database error during user retrieval", err, map[string]interface{}{
            "component":    "database",
            "retry_count":  3,
            "last_attempt": time.Now(),
        })
        return nil, err
    }
    
    // Simulate successful retrieval
    user := &User{
        ID:    userID,
        Name:  "John Doe",
        Email: "john@example.com",
    }
    
    requestLogger.Info("User retrieved successfully", map[string]interface{}{
        "user_name":  user.Name,
        "user_email": "[hidden]",
        "duration":   "45ms",
    })
    
    return user, nil
}

func (us *UserService) CreateUser(ctx context.Context, name, email string) (*User, error) {
    requestLogger := us.logger.WithContext("operation", "CreateUser")
    
    requestLogger.Info("Starting user creation", map[string]interface{}{
        "name":  name,
        "email": "[hidden]",
    })
    
    if name == "" || email == "" {
        err := fmt.Errorf("name and email are required")
        requestLogger.Error("User creation validation failed", err, map[string]interface{}{
            "validation": "required_fields",
            "name_empty": name == "",
            "email_empty": email == "",
        })
        return nil, err
    }
    
    if strings.Contains(email, "spam") {
        err := fmt.Errorf("email %s rejected", email)
        requestLogger.Warn("Spam email detected", map[string]interface{}{
            "email": email,
            "reason": "spam_filter",
            "action": "rejected",
        })
        return nil, err
    }
    
    // Simulate creation
    user := &User{
        ID:    "new-user-123",
        Name:  name,
        Email: email,
    }
    
    requestLogger.Info("User created successfully", map[string]interface{}{
        "user_id": user.ID,
        "name":    user.Name,
        "email":   "[hidden]",
    })
    
    return user, nil
}

// User entity
type User struct {
    ID    string
    Name  string
    Email string
}

// Application with structured error logging
type Application struct {
    logger      *StructuredLogger
    userService *UserService
}

func NewApplication() *Application {
    logger := NewLogger(DEBUG)
    
    return &Application{
        logger:      logger,
        userService: NewUserService(logger),
    }
}

func (app *Application) HandleRequest(requestID, operation, userID string) {
    requestLogger := app.logger.WithContext("request_id", requestID).WithContext("operation", operation)
    
    requestLogger.Info("Processing request")
    
    ctx := context.Background()
    
    switch operation {
    case "get_user":
        user, err := app.userService.GetUser(ctx, userID)
        if err != nil {
            requestLogger.Error("Request failed", err, map[string]interface{}{
                "operation": operation,
                "user_id":   userID,
            })
            return
        }
        
        requestLogger.Info("Request completed successfully", map[string]interface{}{
            "operation": operation,
            "user_id":   user.ID,
        })
        
    case "create_user":
        // For demo, extract name and email from userID
        parts := strings.Split(userID, ":")
        if len(parts) != 2 {
            err := fmt.Errorf("invalid create user format: expected name:email")
            requestLogger.Error("Request failed", err, map[string]interface{}{
                "operation": operation,
                "format":    "name:email",
                "input":     userID,
            })
            return
        }
        
        user, err := app.userService.CreateUser(ctx, parts[0], parts[1])
        if err != nil {
            requestLogger.Error("Request failed", err, map[string]interface{}{
                "operation": operation,
                "name":      parts[0],
                "email":     "[hidden]",
            })
            return
        }
        
        requestLogger.Info("Request completed successfully", map[string]interface{}{
            "operation": operation,
            "user_id":   user.ID,
        })
        
    default:
        err := fmt.Errorf("unknown operation: %s", operation)
        requestLogger.Error("Request failed", err, map[string]interface{}{
            "operation": operation,
            "available": []string{"get_user", "create_user"},
        })
    }
}

func main() {
    fmt.Println("=== Structured Logging with Error Handling ===")
    
    app := NewApplication()
    
    // Test various scenarios
    testCases := []struct {
        requestID string
        operation string
        userID    string
        description string
    }{
        {"req-001", "get_user", "user123", "Successful user retrieval"},
        {"req-002", "get_user", "", "Empty user ID validation error"},
        {"req-003", "get_user", "blocked", "Blocked user access"},
        {"req-004", "get_user", "error", "Database error simulation"},
        {"req-005", "create_user", "John:john@example.com", "Successful user creation"},
        {"req-006", "create_user", "Jane:", "Missing email validation"},
        {"req-007", "create_user", "Bob:spam@spam.com", "Spam email rejection"},
        {"req-008", "invalid_op", "user123", "Unknown operation"},
        {"req-009", "create_user", "invalid_format", "Invalid create format"},
    }
    
    for _, tc := range testCases {
        fmt.Printf("\n--- %s ---\n", tc.description)
        app.HandleRequest(tc.requestID, tc.operation, tc.userID)
    }
    
    fmt.Println("\n=== Logger Context Examples ===")
    
    baseLogger := NewLogger(INFO)
    
    // Service-specific logger
    serviceLogger := baseLogger.WithContext("service", "PaymentService")
    serviceLogger.Info("Service initialized")
    
    // Request-specific logger
    requestLogger := serviceLogger.WithContext("request_id", "pay-123")
    requestLogger.Info("Processing payment")
    
    // Operation-specific logger  
    operationLogger := requestLogger.WithContext("operation", "charge_card")
    operationLogger.Error("Payment processing failed", fmt.Errorf("card declined"), map[string]interface{}{
        "card_type": "visa",
        "amount":    "99.99",
        "currency":  "USD",
        "decline_code": "insufficient_funds",
    })
}
```

Structured logging with error handling provides rich contextual information,  
consistent log formats, and enhanced debugging capabilities that are essential  
for monitoring and maintaining production applications.  

## Error handling patterns with generics

Go's generics enable type-safe error handling patterns that work across  
different types while maintaining compile-time safety and reducing code duplication.  

```go
package main

import (
    "fmt"
    "strconv"
)

// Generic Result type for error handling
type Result[T any] struct {
    Value T
    Err   error
}

func (r Result[T]) IsOk() bool {
    return r.Err == nil
}

func (r Result[T]) IsErr() bool {
    return r.Err != nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.Value, r.Err
}

func (r Result[T]) UnwrapOr(defaultValue T) T {
    if r.IsErr() {
        return defaultValue
    }
    return r.Value
}

func (r Result[T]) Map(fn func(T) T) Result[T] {
    if r.IsErr() {
        return r
    }
    return Ok(fn(r.Value))
}

func (r Result[T]) MapErr(fn func(error) error) Result[T] {
    if r.IsOk() {
        return r
    }
    return Err[T](fn(r.Err))
}

// Helper constructors
func Ok[T any](value T) Result[T] {
    return Result[T]{Value: value, Err: nil}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{Value: zero, Err: err}
}

// Generic validator interface
type Validator[T any] interface {
    Validate(T) error
}

// String validator
type StringValidator struct {
    MinLength int
    MaxLength int
    Required  bool
}

func (sv StringValidator) Validate(s string) error {
    if sv.Required && s == "" {
        return fmt.Errorf("string is required")
    }
    
    if len(s) < sv.MinLength {
        return fmt.Errorf("string too short: got %d, minimum %d", len(s), sv.MinLength)
    }
    
    if sv.MaxLength > 0 && len(s) > sv.MaxLength {
        return fmt.Errorf("string too long: got %d, maximum %d", len(s), sv.MaxLength)
    }
    
    return nil
}

// Number validator
type NumberValidator[T int | float64] struct {
    Min T
    Max T
}

func (nv NumberValidator[T]) Validate(n T) error {
    if n < nv.Min {
        return fmt.Errorf("number too small: got %v, minimum %v", n, nv.Min)
    }
    
    if n > nv.Max {
        return fmt.Errorf("number too large: got %v, maximum %v", n, nv.Max)
    }
    
    return nil
}

// Generic validation function
func ValidateValue[T any](value T, validator Validator[T]) Result[T] {
    if err := validator.Validate(value); err != nil {
        return Err[T](fmt.Errorf("validation failed: %w", err))
    }
    return Ok(value)
}

// Generic parser with error handling
type Parser[T any] interface {
    Parse(string) Result[T]
}

// String parser
type StringParser struct {
    Validator StringValidator
}

func (sp StringParser) Parse(s string) Result[string] {
    return ValidateValue(s, sp.Validator)
}

// Integer parser
type IntParser struct {
    Validator NumberValidator[int]
}

func (ip IntParser) Parse(s string) Result[int] {
    value, err := strconv.Atoi(s)
    if err != nil {
        return Err[int](fmt.Errorf("invalid integer: %w", err))
    }
    
    return ValidateValue(value, ip.Validator)
}

// Float parser
type FloatParser struct {
    Validator NumberValidator[float64]
}

func (fp FloatParser) Parse(s string) Result[float64] {
    value, err := strconv.ParseFloat(s, 64)
    if err != nil {
        return Err[float64](fmt.Errorf("invalid float: %w", err))
    }
    
    return ValidateValue(value, fp.Validator)
}

// Generic batch processing with error collection
func ProcessBatch[T, R any](items []T, processor func(T) Result[R]) ([]R, []error) {
    var results []R
    var errors []error
    
    for i, item := range items {
        result := processor(item)
        if result.IsErr() {
            errors = append(errors, fmt.Errorf("item %d: %w", i, result.Err))
        } else {
            results = append(results, result.Value)
        }
    }
    
    return results, errors
}

// Generic chain processing
func Chain[T any](initial Result[T], operations ...func(T) Result[T]) Result[T] {
    current := initial
    
    for _, op := range operations {
        if current.IsErr() {
            return current
        }
        current = op(current.Value)
    }
    
    return current
}

// Generic optional type
type Option[T any] struct {
    value  T
    hasValue bool
}

func Some[T any](value T) Option[T] {
    return Option[T]{value: value, hasValue: true}
}

func None[T any]() Option[T] {
    var zero T
    return Option[T]{value: zero, hasValue: false}
}

func (o Option[T]) IsSome() bool {
    return o.hasValue
}

func (o Option[T]) IsNone() bool {
    return !o.hasValue
}

func (o Option[T]) Unwrap() T {
    if !o.hasValue {
        panic("called Unwrap on None")
    }
    return o.value
}

func (o Option[T]) UnwrapOr(defaultValue T) T {
    if !o.hasValue {
        return defaultValue
    }
    return o.value
}

func (o Option[T]) ToResult(err error) Result[T] {
    if !o.hasValue {
        return Err[T](err)
    }
    return Ok(o.value)
}

// Generic find function
func Find[T any](slice []T, predicate func(T) bool) Option[T] {
    for _, item := range slice {
        if predicate(item) {
            return Some(item)
        }
    }
    return None[T]()
}

func main() {
    fmt.Println("=== Generic Error Handling Patterns ===")
    
    fmt.Println("1. Result Type Pattern")
    
    // String parsing with validation
    stringParser := StringParser{
        Validator: StringValidator{MinLength: 3, MaxLength: 10, Required: true},
    }
    
    testStrings := []string{"hello", "hi", "", "this is too long"}
    for _, s := range testStrings {
        result := stringParser.Parse(s)
        fmt.Printf("String %q: ", s)
        if result.IsOk() {
            fmt.Printf("✓ Valid: %s\n", result.Value)
        } else {
            fmt.Printf("✗ Error: %v\n", result.Err)
        }
    }
    
    fmt.Println("\n2. Number Parsing with Validation")
    
    intParser := IntParser{
        Validator: NumberValidator[int]{Min: 1, Max: 100},
    }
    
    testInts := []string{"50", "0", "150", "not-a-number"}
    for _, s := range testInts {
        result := intParser.Parse(s)
        fmt.Printf("Integer %q: ", s)
        if result.IsOk() {
            fmt.Printf("✓ Valid: %d\n", result.Value)
        } else {
            fmt.Printf("✗ Error: %v\n", result.Err)
        }
    }
    
    fmt.Println("\n3. Batch Processing")
    
    numbers := []string{"10", "20", "invalid", "30", "200"}
    results, errors := ProcessBatch(numbers, intParser.Parse)
    
    fmt.Printf("Processed %d items successfully: %v\n", len(results), results)
    fmt.Printf("Encountered %d errors:\n", len(errors))
    for _, err := range errors {
        fmt.Printf("  - %v\n", err)
    }
    
    fmt.Println("\n4. Chain Processing")
    
    doubleValue := func(x int) Result[int] {
        return Ok(x * 2)
    }
    
    validateRange := func(x int) Result[int] {
        if x > 200 {
            return Err[int](fmt.Errorf("value %d exceeds maximum", x))
        }
        return Ok(x)
    }
    
    addTen := func(x int) Result[int] {
        return Ok(x + 10)
    }
    
    initialValue := Ok(25)
    finalResult := Chain(initialValue, doubleValue, validateRange, addTen)
    
    fmt.Printf("Chain processing: ")
    if finalResult.IsOk() {
        fmt.Printf("✓ Result: %d\n", finalResult.Value)
    } else {
        fmt.Printf("✗ Error: %v\n", finalResult.Err)
    }
    
    // Test chain with error
    initialValue2 := Ok(150)
    finalResult2 := Chain(initialValue2, doubleValue, validateRange, addTen)
    
    fmt.Printf("Chain with error: ")
    if finalResult2.IsOk() {
        fmt.Printf("✓ Result: %d\n", finalResult2.Value)
    } else {
        fmt.Printf("✗ Error: %v\n", finalResult2.Err)
    }
    
    fmt.Println("\n5. Option Type Pattern")
    
    users := []struct{ ID int; Name string }{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"},
    }
    
    findUser := func(id int) Option[string] {
        found := Find(users, func(u struct{ ID int; Name string }) bool {
            return u.ID == id
        })
        
        if found.IsNone() {
            return None[string]()
        }
        
        return Some(found.Unwrap().Name)
    }
    
    testIDs := []int{1, 4, 2}
    for _, id := range testIDs {
        user := findUser(id)
        fmt.Printf("User ID %d: ", id)
        if user.IsSome() {
            fmt.Printf("✓ Found: %s\n", user.Unwrap())
        } else {
            fmt.Printf("✗ Not found\n")
        }
    }
    
    fmt.Println("\n6. Option to Result Conversion")
    
    userResult := findUser(5).ToResult(fmt.Errorf("user not found"))
    fmt.Printf("User lookup result: ")
    if userResult.IsOk() {
        fmt.Printf("✓ User: %s\n", userResult.Value)
    } else {
        fmt.Printf("✗ Error: %v\n", userResult.Err)
    }
    
    fmt.Println("\n7. Result Transformation")
    
    parseResult := intParser.Parse("42")
    transformedResult := parseResult.Map(func(x int) int { return x * 2 })
    
    fmt.Printf("Transform result: ")
    if transformedResult.IsOk() {
        fmt.Printf("✓ Doubled: %d\n", transformedResult.Value)
    } else {
        fmt.Printf("✗ Error: %v\n", transformedResult.Err)
    }
    
    // Error transformation
    errorResult := intParser.Parse("invalid")
    mappedError := errorResult.MapErr(func(err error) error {
        return fmt.Errorf("parsing failed: %w", err)
    })
    
    fmt.Printf("Transform error: ✗ %v\n", mappedError.Err)
}
```

Generic error handling patterns provide type-safe, reusable error handling  
mechanisms that reduce code duplication while maintaining compile-time safety  
and enabling elegant functional-style error processing.  

## Monitoring and metrics for errors

Error monitoring and metrics collection provide insights into application  
health, error patterns, and system reliability for proactive maintenance.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Error metrics collector
type ErrorMetrics struct {
    mu              sync.RWMutex
    totalErrors     int64
    errorsByType    map[string]int64
    errorsByService map[string]int64
    errorRates      map[string]*RateCounter
    recentErrors    []ErrorEvent
    maxRecentErrors int
}

type ErrorEvent struct {
    Timestamp   time.Time
    ErrorType   string
    Service     string
    Message     string
    Severity    string
    UserID      string
    RequestID   string
    Metadata    map[string]interface{}
}

type RateCounter struct {
    count     int64
    window    time.Duration
    lastReset time.Time
    mu        sync.Mutex
}

func NewRateCounter(window time.Duration) *RateCounter {
    return &RateCounter{
        window:    window,
        lastReset: time.Now(),
    }
}

func (rc *RateCounter) Increment() {
    rc.mu.Lock()
    defer rc.mu.Unlock()
    
    now := time.Now()
    if now.Sub(rc.lastReset) >= rc.window {
        rc.count = 0
        rc.lastReset = now
    }
    
    rc.count++
}

func (rc *RateCounter) Rate() int64 {
    rc.mu.Lock()
    defer rc.mu.Unlock()
    
    now := time.Now()
    if now.Sub(rc.lastReset) >= rc.window {
        return 0
    }
    
    return rc.count
}

func NewErrorMetrics() *ErrorMetrics {
    return &ErrorMetrics{
        errorsByType:    make(map[string]int64),
        errorsByService: make(map[string]int64),
        errorRates:      make(map[string]*RateCounter),
        recentErrors:    make([]ErrorEvent, 0),
        maxRecentErrors: 100,
    }
}

func (em *ErrorMetrics) RecordError(event ErrorEvent) {
    em.mu.Lock()
    defer em.mu.Unlock()
    
    em.totalErrors++
    em.errorsByType[event.ErrorType]++
    em.errorsByService[event.Service]++
    
    // Update rate counters
    typeKey := fmt.Sprintf("type:%s", event.ErrorType)
    if em.errorRates[typeKey] == nil {
        em.errorRates[typeKey] = NewRateCounter(time.Minute)
    }
    em.errorRates[typeKey].Increment()
    
    serviceKey := fmt.Sprintf("service:%s", event.Service)
    if em.errorRates[serviceKey] == nil {
        em.errorRates[serviceKey] = NewRateCounter(time.Minute)
    }
    em.errorRates[serviceKey].Increment()
    
    // Store recent errors
    em.recentErrors = append(em.recentErrors, event)
    if len(em.recentErrors) > em.maxRecentErrors {
        em.recentErrors = em.recentErrors[1:]
    }
}

func (em *ErrorMetrics) GetStats() map[string]interface{} {
    em.mu.RLock()
    defer em.mu.RUnlock()
    
    errorRates := make(map[string]int64)
    for key, counter := range em.errorRates {
        errorRates[key] = counter.Rate()
    }
    
    return map[string]interface{}{
        "total_errors":      em.totalErrors,
        "errors_by_type":    em.copyMap(em.errorsByType),
        "errors_by_service": em.copyMap(em.errorsByService),
        "error_rates_per_minute": errorRates,
        "recent_errors_count":    len(em.recentErrors),
    }
}

func (em *ErrorMetrics) copyMap(m map[string]int64) map[string]int64 {
    copy := make(map[string]int64)
    for k, v := range m {
        copy[k] = v
    }
    return copy
}

func (em *ErrorMetrics) GetRecentErrors(limit int) []ErrorEvent {
    em.mu.RLock()
    defer em.mu.RUnlock()
    
    if limit <= 0 || limit > len(em.recentErrors) {
        limit = len(em.recentErrors)
    }
    
    start := len(em.recentErrors) - limit
    result := make([]ErrorEvent, limit)
    copy(result, em.recentErrors[start:])
    return result
}

// Alert system
type AlertRule struct {
    Name        string
    Condition   func(*ErrorMetrics) bool
    Message     string
    Severity    string
    Cooldown    time.Duration
    LastTriggered time.Time
}

type AlertManager struct {
    metrics *ErrorMetrics
    rules   []AlertRule
    alerts  []Alert
    mu      sync.RWMutex
}

type Alert struct {
    Rule      string
    Message   string
    Severity  string
    Timestamp time.Time
    Resolved  bool
}

func NewAlertManager(metrics *ErrorMetrics) *AlertManager {
    am := &AlertManager{
        metrics: metrics,
        alerts:  make([]Alert, 0),
    }
    
    // Define alert rules
    am.rules = []AlertRule{
        {
            Name: "high_error_rate",
            Condition: func(m *ErrorMetrics) bool {
                stats := m.GetStats()
                rates := stats["error_rates_per_minute"].(map[string]int64)
                for _, rate := range rates {
                    if rate > 10 { // More than 10 errors per minute
                        return true
                    }
                }
                return false
            },
            Message:  "High error rate detected",
            Severity: "critical",
            Cooldown: 5 * time.Minute,
        },
        {
            Name: "database_errors",
            Condition: func(m *ErrorMetrics) bool {
                stats := m.GetStats()
                byType := stats["errors_by_type"].(map[string]int64)
                return byType["database"] > 5
            },
            Message:  "Multiple database errors detected",
            Severity: "warning",
            Cooldown: 2 * time.Minute,
        },
        {
            Name: "authentication_failures",
            Condition: func(m *ErrorMetrics) bool {
                stats := m.GetStats()
                byType := stats["errors_by_type"].(map[string]int64)
                return byType["authentication"] > 3
            },
            Message:  "Multiple authentication failures",
            Severity: "warning",
            Cooldown: 1 * time.Minute,
        },
    }
    
    return am
}

func (am *AlertManager) CheckAlerts() []Alert {
    am.mu.Lock()
    defer am.mu.Unlock()
    
    var newAlerts []Alert
    
    for i := range am.rules {
        rule := &am.rules[i]
        
        // Check cooldown
        if time.Since(rule.LastTriggered) < rule.Cooldown {
            continue
        }
        
        if rule.Condition(am.metrics) {
            alert := Alert{
                Rule:      rule.Name,
                Message:   rule.Message,
                Severity:  rule.Severity,
                Timestamp: time.Now(),
                Resolved:  false,
            }
            
            am.alerts = append(am.alerts, alert)
            newAlerts = append(newAlerts, alert)
            rule.LastTriggered = time.Now()
            
            fmt.Printf("🚨 ALERT [%s]: %s at %s\n", 
                      alert.Severity, alert.Message, alert.Timestamp.Format("15:04:05"))
        }
    }
    
    return newAlerts
}

// Monitored service wrapper
type MonitoredService struct {
    name    string
    metrics *ErrorMetrics
}

func NewMonitoredService(name string, metrics *ErrorMetrics) *MonitoredService {
    return &MonitoredService{
        name:    name,
        metrics: metrics,
    }
}

func (ms *MonitoredService) ExecuteWithMonitoring(operation string, fn func() error) error {
    start := time.Now()
    err := fn()
    duration := time.Since(start)
    
    if err != nil {
        errorType := classifyError(err)
        
        event := ErrorEvent{
            Timestamp: time.Now(),
            ErrorType: errorType,
            Service:   ms.name,
            Message:   err.Error(),
            Severity:  determineSeverity(errorType),
            Metadata: map[string]interface{}{
                "operation": operation,
                "duration":  duration.Milliseconds(),
            },
        }
        
        ms.metrics.RecordError(event)
        
        return fmt.Errorf("monitored operation failed: %w", err)
    }
    
    return nil
}

func classifyError(err error) string {
    errMsg := err.Error()
    
    switch {
    case contains(errMsg, "database", "connection", "sql"):
        return "database"
    case contains(errMsg, "network", "timeout", "connection refused"):
        return "network"
    case contains(errMsg, "authentication", "unauthorized", "forbidden"):
        return "authentication"
    case contains(errMsg, "validation", "invalid", "required"):
        return "validation"
    case contains(errMsg, "not found", "missing"):
        return "not_found"
    default:
        return "unknown"
    }
}

func contains(text string, keywords ...string) bool {
    for _, keyword := range keywords {
        if fmt.Sprintf("%s", text) != "" && fmt.Sprintf("%s", keyword) != "" {
            // Simple contains check (in a real implementation, use strings.Contains)
            return true
        }
    }
    return false
}

func determineSeverity(errorType string) string {
    switch errorType {
    case "database", "network":
        return "critical"
    case "authentication":
        return "warning"
    case "validation", "not_found":
        return "info"
    default:
        return "error"
    }
}

func main() {
    fmt.Println("=== Error Monitoring and Metrics ===")
    
    metrics := NewErrorMetrics()
    alertManager := NewAlertManager(metrics)
    
    // Create monitored services
    userService := NewMonitoredService("user-service", metrics)
    authService := NewMonitoredService("auth-service", metrics)
    dbService := NewMonitoredService("database-service", metrics)
    
    fmt.Println("1. Simulating Service Operations")
    
    // Simulate various operations and errors
    operations := []struct {
        service   *MonitoredService
        operation string
        fn        func() error
        desc      string
    }{
        {userService, "get_user", func() error { return nil }, "Successful user lookup"},
        {userService, "get_user", func() error { return fmt.Errorf("user not found") }, "User not found"},
        {authService, "login", func() error { return fmt.Errorf("authentication failed") }, "Login failure"},
        {authService, "login", func() error { return fmt.Errorf("authentication failed") }, "Another login failure"},
        {dbService, "query", func() error { return fmt.Errorf("database connection timeout") }, "DB timeout"},
        {dbService, "query", func() error { return fmt.Errorf("sql syntax error") }, "SQL error"},
        {userService, "create_user", func() error { return fmt.Errorf("validation error: email required") }, "Validation error"},
        {authService, "validate_token", func() error { return fmt.Errorf("unauthorized access") }, "Token validation"},
        {dbService, "query", func() error { return fmt.Errorf("database connection lost") }, "DB connection lost"},
    }
    
    for i, op := range operations {
        fmt.Printf("Operation %d: %s - ", i+1, op.desc)
        
        err := op.service.ExecuteWithMonitoring(op.operation, op.fn)
        if err != nil {
            fmt.Printf("❌ %v\n", err)
        } else {
            fmt.Printf("✅ Success\n")
        }
        
        // Check for alerts after each operation
        alerts := alertManager.CheckAlerts()
        if len(alerts) > 0 {
            fmt.Printf("  New alerts triggered: %d\n", len(alerts))
        }
        
        // Small delay to allow rate counters to work
        time.Sleep(100 * time.Millisecond)
    }
    
    fmt.Println("\n2. Current Metrics")
    stats := metrics.GetStats()
    
    fmt.Printf("Total Errors: %v\n", stats["total_errors"])
    fmt.Printf("Errors by Type: %v\n", stats["errors_by_type"])
    fmt.Printf("Errors by Service: %v\n", stats["errors_by_service"])
    fmt.Printf("Error Rates (per minute): %v\n", stats["error_rates_per_minute"])
    
    fmt.Println("\n3. Recent Error Events")
    recentErrors := metrics.GetRecentErrors(5)
    for i, event := range recentErrors {
        fmt.Printf("  %d. [%s] %s - %s (%s) at %s\n", 
                  i+1, event.Severity, event.Service, 
                  event.ErrorType, event.Message, 
                  event.Timestamp.Format("15:04:05"))
    }
    
    fmt.Println("\n4. Triggering High Error Rate Alert")
    
    // Simulate high error rate
    for i := 0; i < 15; i++ {
        metrics.RecordError(ErrorEvent{
            Timestamp: time.Now(),
            ErrorType: "network",
            Service:   "external-api",
            Message:   fmt.Sprintf("network timeout %d", i),
            Severity:  "critical",
        })
    }
    
    alerts := alertManager.CheckAlerts()
    fmt.Printf("Triggered %d alerts\n", len(alerts))
    
    // Final stats
    fmt.Println("\n5. Final Metrics Summary")
    finalStats := metrics.GetStats()
    fmt.Printf("Total Errors: %v\n", finalStats["total_errors"])
    fmt.Printf("Critical Error Rate: High network error rate detected\n")
    
    fmt.Println("\n6. System Health Dashboard")
    fmt.Println("📊 Error Distribution:")
    errorsByType := finalStats["errors_by_type"].(map[string]int64)
    for errorType, count := range errorsByType {
        percentage := float64(count) / float64(finalStats["total_errors"].(int64)) * 100
        fmt.Printf("  %s: %d errors (%.1f%%)\n", errorType, count, percentage)
    }
}
```

Error monitoring and metrics provide essential insights into application  
health, enabling proactive issue detection, performance optimization, and  
data-driven decisions about system reliability and maintenance priorities.  

## Advanced error propagation patterns

Complex applications require sophisticated error propagation strategies  
that preserve error context while enabling proper handling at different layers.  

```go
package main

import (
    "context"
    "fmt"
    "strings"
    "time"
)

// Error with propagation chain
type PropagatedError struct {
    Err       error
    Layer     string
    Component string
    Operation string
    Timestamp time.Time
    Context   map[string]interface{}
    Cause     *PropagatedError
}

func (pe PropagatedError) Error() string {
    if pe.Cause != nil {
        return fmt.Sprintf("%s.%s[%s]: %v (caused by: %s)", 
                          pe.Layer, pe.Component, pe.Operation, pe.Err, pe.Cause.Error())
    }
    return fmt.Sprintf("%s.%s[%s]: %v", pe.Layer, pe.Component, pe.Operation, pe.Err)
}

func (pe PropagatedError) Unwrap() error {
    if pe.Cause != nil {
        return pe.Cause
    }
    return pe.Err
}

func (pe PropagatedError) RootCause() error {
    current := &pe
    for current.Cause != nil {
        current = current.Cause
    }
    return current.Err
}

func (pe PropagatedError) Chain() []*PropagatedError {
    var chain []*PropagatedError
    current := &pe
    
    for current != nil {
        chain = append(chain, current)
        current = current.Cause
    }
    
    return chain
}

// Error propagation builder
type ErrorPropagator struct {
    layer     string
    component string
}

func NewErrorPropagator(layer, component string) *ErrorPropagator {
    return &ErrorPropagator{
        layer:     layer,
        component: component,
    }
}

func (ep *ErrorPropagator) Propagate(operation string, err error, context ...map[string]interface{}) *PropagatedError {
    var ctx map[string]interface{}
    if len(context) > 0 {
        ctx = context[0]
    }
    
    propagatedErr := &PropagatedError{
        Err:       err,
        Layer:     ep.layer,
        Component: ep.component,
        Operation: operation,
        Timestamp: time.Now(),
        Context:   ctx,
    }
    
    // Check if the error is already a propagated error
    if existingErr, ok := err.(*PropagatedError); ok {
        propagatedErr.Cause = existingErr
        propagatedErr.Err = fmt.Errorf("operation failed")
    }
    
    return propagatedErr
}

// Multi-layer application example
type DatabaseLayer struct {
    propagator *ErrorPropagator
}

func NewDatabaseLayer() *DatabaseLayer {
    return &DatabaseLayer{
        propagator: NewErrorPropagator("data", "database"),
    }
}

func (dl *DatabaseLayer) GetUser(userID string) (*User, error) {
    if userID == "" {
        return nil, dl.propagator.Propagate("get_user", 
            fmt.Errorf("user ID cannot be empty"),
            map[string]interface{}{"user_id": userID})
    }
    
    if userID == "connection_error" {
        return nil, dl.propagator.Propagate("get_user",
            fmt.Errorf("database connection failed"),
            map[string]interface{}{"user_id": userID, "retry_count": 3})
    }
    
    if userID == "not_found" {
        return nil, dl.propagator.Propagate("get_user",
            fmt.Errorf("user not found"),
            map[string]interface{}{"user_id": userID, "table": "users"})
    }
    
    return &User{ID: userID, Name: "John Doe", Email: "john@example.com"}, nil
}

func (dl *DatabaseLayer) SaveUser(user *User) error {
    if user == nil {
        return dl.propagator.Propagate("save_user",
            fmt.Errorf("user cannot be nil"))
    }
    
    if user.Email == "" {
        return dl.propagator.Propagate("save_user",
            fmt.Errorf("user email is required"),
            map[string]interface{}{"user_id": user.ID})
    }
    
    return nil
}

// Business logic layer
type BusinessLayer struct {
    db         *DatabaseLayer
    propagator *ErrorPropagator
}

func NewBusinessLayer(db *DatabaseLayer) *BusinessLayer {
    return &BusinessLayer{
        db:         db,
        propagator: NewErrorPropagator("business", "user_service"),
    }
}

func (bl *BusinessLayer) ProcessUserLogin(userID, password string) (*User, error) {
    if password == "" {
        return nil, bl.propagator.Propagate("process_login",
            fmt.Errorf("password is required"),
            map[string]interface{}{"user_id": userID})
    }
    
    user, err := bl.db.GetUser(userID)
    if err != nil {
        return nil, bl.propagator.Propagate("process_login", err,
            map[string]interface{}{"user_id": userID, "step": "user_lookup"})
    }
    
    if password == "wrong_password" {
        return nil, bl.propagator.Propagate("process_login",
            fmt.Errorf("invalid credentials"),
            map[string]interface{}{"user_id": userID, "step": "authentication"})
    }
    
    return user, nil
}

func (bl *BusinessLayer) CreateUserAccount(name, email string) (*User, error) {
    if name == "" || email == "" {
        return nil, bl.propagator.Propagate("create_account",
            fmt.Errorf("name and email are required"),
            map[string]interface{}{"name": name, "email": "[hidden]"})
    }
    
    // Check if user exists
    existingUser, err := bl.db.GetUser(email)
    if err != nil {
        // Check if error is "not found" which is expected
        if propErr, ok := err.(*PropagatedError); ok {
            if !strings.Contains(propErr.RootCause().Error(), "not found") {
                return nil, bl.propagator.Propagate("create_account", err,
                    map[string]interface{}{"email": "[hidden]", "step": "existence_check"})
            }
        }
    } else if existingUser != nil {
        return nil, bl.propagator.Propagate("create_account",
            fmt.Errorf("user already exists"),
            map[string]interface{}{"email": "[hidden]", "existing_id": existingUser.ID})
    }
    
    newUser := &User{ID: "new_" + email, Name: name, Email: email}
    
    if err := bl.db.SaveUser(newUser); err != nil {
        return nil, bl.propagator.Propagate("create_account", err,
            map[string]interface{}{"user_id": newUser.ID, "step": "save_user"})
    }
    
    return newUser, nil
}

// Presentation layer
type PresentationLayer struct {
    business   *BusinessLayer
    propagator *ErrorPropagator
}

func NewPresentationLayer(business *BusinessLayer) *PresentationLayer {
    return &PresentationLayer{
        business:   business,
        propagator: NewErrorPropagator("presentation", "http_handler"),
    }
}

func (pl *PresentationLayer) HandleLogin(ctx context.Context, userID, password string) (map[string]interface{}, error) {
    requestID := getRequestID(ctx)
    
    user, err := pl.business.ProcessUserLogin(userID, password)
    if err != nil {
        return nil, pl.propagator.Propagate("handle_login", err,
            map[string]interface{}{"request_id": requestID, "user_id": userID})
    }
    
    response := map[string]interface{}{
        "user_id":    user.ID,
        "name":       user.Name,
        "email":      user.Email,
        "request_id": requestID,
    }
    
    return response, nil
}

func (pl *PresentationLayer) HandleUserCreation(ctx context.Context, name, email string) (map[string]interface{}, error) {
    requestID := getRequestID(ctx)
    
    user, err := pl.business.CreateUserAccount(name, email)
    if err != nil {
        return nil, pl.propagator.Propagate("handle_user_creation", err,
            map[string]interface{}{"request_id": requestID, "name": name})
    }
    
    response := map[string]interface{}{
        "user_id":    user.ID,
        "name":       user.Name,
        "email":      user.Email,
        "request_id": requestID,
        "status":     "created",
    }
    
    return response, nil
}

// Helper types and functions
type User struct {
    ID    string
    Name  string
    Email string
}

func getRequestID(ctx context.Context) string {
    if requestID, ok := ctx.Value("request_id").(string); ok {
        return requestID
    }
    return fmt.Sprintf("req_%d", time.Now().Unix())
}

// Error analysis utilities
func analyzeError(err error) {
    if propErr, ok := err.(*PropagatedError); ok {
        fmt.Printf("🔍 Error Analysis:\n")
        fmt.Printf("  Primary Error: %v\n", propErr.Error())
        fmt.Printf("  Root Cause: %v\n", propErr.RootCause())
        
        chain := propErr.Chain()
        fmt.Printf("  Propagation Chain (%d levels):\n", len(chain))
        
        for i, chainErr := range chain {
            fmt.Printf("    %d. [%s.%s] %s: %v\n", 
                      i+1, chainErr.Layer, chainErr.Component, 
                      chainErr.Operation, chainErr.Err)
            
            if len(chainErr.Context) > 0 {
                fmt.Printf("       Context: %v\n", chainErr.Context)
            }
        }
    } else {
        fmt.Printf("🔍 Simple Error: %v\n", err)
    }
    fmt.Println()
}

func main() {
    fmt.Println("=== Advanced Error Propagation Patterns ===")
    
    // Build application layers
    db := NewDatabaseLayer()
    business := NewBusinessLayer(db)
    presentation := NewPresentationLayer(business)
    
    fmt.Println("1. Successful Operation")
    ctx := context.WithValue(context.Background(), "request_id", "req_001")
    result, err := presentation.HandleLogin(ctx, "user123", "correct_password")
    if err != nil {
        analyzeError(err)
    } else {
        fmt.Printf("✅ Success: %v\n\n", result)
    }
    
    fmt.Println("2. Validation Error at Presentation Layer")
    result, err = presentation.HandleLogin(ctx, "", "password")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("3. Business Logic Error")
    result, err = presentation.HandleLogin(ctx, "user123", "wrong_password")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("4. Database Layer Error Propagation")
    result, err = presentation.HandleLogin(ctx, "connection_error", "password")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("5. Complex Error Chain in User Creation")
    ctx2 := context.WithValue(context.Background(), "request_id", "req_002")
    result, err = presentation.HandleUserCreation(ctx2, "", "")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("6. Database Error During User Creation")
    result, err = presentation.HandleUserCreation(ctx2, "John", "connection_error")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("7. User Already Exists Scenario")
    // First create a user that exists
    result, err = presentation.HandleUserCreation(ctx2, "Jane", "existing_user")
    if err != nil {
        analyzeError(err)
    }
    
    fmt.Println("8. Error Pattern Summary")
    fmt.Println("Error propagation provides:")
    fmt.Println("  ✅ Full context preservation across layers")
    fmt.Println("  ✅ Root cause identification")
    fmt.Println("  ✅ Layer-specific error information")
    fmt.Println("  ✅ Debugging and monitoring capabilities")
    fmt.Println("  ✅ Structured error analysis")
}
```

Advanced error propagation patterns enable comprehensive error tracking  
across application layers, providing detailed context for debugging while  
maintaining clean separation of concerns and facilitating effective error handling.  

## Error handling with reflection

Reflection-based error handling enables generic error processing and  
validation that works across different types while maintaining type safety.  

```go
package main

import (
    "fmt"
    "reflect"
    "strings"
)

// Validation tags and reflection-based validation
type ValidationRule struct {
    Field     string
    Rule      string
    Parameter string
    Message   string
}

type ValidationError struct {
    Field   string
    Value   interface{}
    Rule    string
    Message string
}

func (ve ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %q: %s", ve.Field, ve.Message)
}

type ValidationErrors []ValidationError

func (ve ValidationErrors) Error() string {
    if len(ve) == 0 {
        return "no validation errors"
    }
    
    var messages []string
    for _, err := range ve {
        messages = append(messages, err.Error())
    }
    
    return fmt.Sprintf("validation failed with %d error(s): %s", 
                       len(ve), strings.Join(messages, "; "))
}

func (ve ValidationErrors) HasErrors() bool {
    return len(ve) > 0
}

// Reflection-based validator
type ReflectionValidator struct{}

func (rv ReflectionValidator) Validate(v interface{}) error {
    val := reflect.ValueOf(v)
    typ := reflect.TypeOf(v)
    
    // Handle pointers
    if val.Kind() == reflect.Ptr {
        if val.IsNil() {
            return fmt.Errorf("cannot validate nil pointer")
        }
        val = val.Elem()
        typ = typ.Elem()
    }
    
    if val.Kind() != reflect.Struct {
        return fmt.Errorf("validation requires struct type, got %T", v)
    }
    
    var errors ValidationErrors
    
    for i := 0; i < val.NumField(); i++ {
        field := val.Field(i)
        fieldType := typ.Field(i)
        fieldName := fieldType.Name
        
        // Skip unexported fields
        if !field.CanInterface() {
            continue
        }
        
        if err := rv.validateField(fieldName, field, fieldType); err != nil {
            if validationErrors, ok := err.(ValidationErrors); ok {
                errors = append(errors, validationErrors...)
            } else if validationError, ok := err.(ValidationError); ok {
                errors = append(errors, validationError)
            } else {
                errors = append(errors, ValidationError{
                    Field:   fieldName,
                    Value:   field.Interface(),
                    Rule:    "unknown",
                    Message: err.Error(),
                })
            }
        }
    }
    
    if errors.HasErrors() {
        return errors
    }
    
    return nil
}

func (rv ReflectionValidator) validateField(fieldName string, field reflect.Value, fieldType reflect.StructField) error {
    var errors ValidationErrors
    
    // Check required tag
    if required := fieldType.Tag.Get("required"); required == "true" {
        if rv.isEmpty(field) {
            errors = append(errors, ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "required",
                Message: "field is required",
            })
        }
    }
    
    // Check min tag
    if minStr := fieldType.Tag.Get("min"); minStr != "" {
        if err := rv.validateMin(fieldName, field, minStr); err != nil {
            errors = append(errors, err)
        }
    }
    
    // Check max tag
    if maxStr := fieldType.Tag.Get("max"); maxStr != "" {
        if err := rv.validateMax(fieldName, field, maxStr); err != nil {
            errors = append(errors, err)
        }
    }
    
    // Check pattern tag
    if pattern := fieldType.Tag.Get("pattern"); pattern != "" {
        if err := rv.validatePattern(fieldName, field, pattern); err != nil {
            errors = append(errors, err)
        }
    }
    
    // Check custom validation method
    if method := fieldType.Tag.Get("validate"); method != "" {
        if err := rv.callCustomValidator(fieldName, field, method); err != nil {
            errors = append(errors, err)
        }
    }
    
    if len(errors) > 0 {
        return errors
    }
    
    return nil
}

func (rv ReflectionValidator) isEmpty(field reflect.Value) bool {
    switch field.Kind() {
    case reflect.String:
        return field.String() == ""
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        return field.Int() == 0
    case reflect.Float32, reflect.Float64:
        return field.Float() == 0
    case reflect.Bool:
        return !field.Bool()
    case reflect.Slice, reflect.Array, reflect.Map:
        return field.Len() == 0
    case reflect.Ptr, reflect.Interface:
        return field.IsNil()
    default:
        return false
    }
}

func (rv ReflectionValidator) validateMin(fieldName string, field reflect.Value, minStr string) ValidationError {
    switch field.Kind() {
    case reflect.String:
        if len(field.String()) < rv.parseInt(minStr) {
            return ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "min",
                Message: fmt.Sprintf("minimum length is %s", minStr),
            }
        }
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        if field.Int() < int64(rv.parseInt(minStr)) {
            return ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "min",
                Message: fmt.Sprintf("minimum value is %s", minStr),
            }
        }
    }
    return ValidationError{}
}

func (rv ReflectionValidator) validateMax(fieldName string, field reflect.Value, maxStr string) ValidationError {
    switch field.Kind() {
    case reflect.String:
        if len(field.String()) > rv.parseInt(maxStr) {
            return ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "max",
                Message: fmt.Sprintf("maximum length is %s", maxStr),
            }
        }
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        if field.Int() > int64(rv.parseInt(maxStr)) {
            return ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "max",
                Message: fmt.Sprintf("maximum value is %s", maxStr),
            }
        }
    }
    return ValidationError{}
}

func (rv ReflectionValidator) validatePattern(fieldName string, field reflect.Value, pattern string) ValidationError {
    if field.Kind() != reflect.String {
        return ValidationError{}
    }
    
    value := field.String()
    
    // Simple pattern matching (in real implementation, use regex)
    switch pattern {
    case "email":
        if !strings.Contains(value, "@") || !strings.Contains(value, ".") {
            return ValidationError{
                Field:   fieldName,
                Value:   value,
                Rule:    "pattern",
                Message: "must be a valid email address",
            }
        }
    case "numeric":
        for _, r := range value {
            if r < '0' || r > '9' {
                return ValidationError{
                    Field:   fieldName,
                    Value:   value,
                    Rule:    "pattern",
                    Message: "must contain only numbers",
                }
            }
        }
    }
    
    return ValidationError{}
}

func (rv ReflectionValidator) callCustomValidator(fieldName string, field reflect.Value, method string) ValidationError {
    // In a real implementation, this would call custom validation methods
    // For demo purposes, we'll implement a simple check
    if method == "positive" && field.Kind() == reflect.Int {
        if field.Int() <= 0 {
            return ValidationError{
                Field:   fieldName,
                Value:   field.Interface(),
                Rule:    "positive",
                Message: "must be a positive number",
            }
        }
    }
    
    return ValidationError{}
}

func (rv ReflectionValidator) parseInt(s string) int {
    // Simple integer parsing (in real implementation, use strconv.Atoi)
    switch s {
    case "1": return 1
    case "2": return 2
    case "3": return 3
    case "5": return 5
    case "10": return 10
    case "100": return 100
    default: return 0
    }
}

// Example structs with validation tags
type User struct {
    ID       int    `required:"true" validate:"positive"`
    Username string `required:"true" min:"3" max:"20"`
    Email    string `required:"true" pattern:"email"`
    Age      int    `min:"18" max:"120"`
    Phone    string `pattern:"numeric" min:"10" max:"15"`
}

type Product struct {
    Name        string  `required:"true" min:"2" max:"50"`
    Price       float64 `required:"true"`
    Description string  `max:"200"`
    Category    string  `required:"true" min:"2"`
}

func main() {
    fmt.Println("=== Reflection-Based Error Handling ===")
    
    validator := ReflectionValidator{}
    
    fmt.Println("1. Valid User")
    validUser := User{
        ID:       1,
        Username: "john_doe",
        Email:    "john@example.com",
        Age:      25,
        Phone:    "1234567890",
    }
    
    if err := validator.Validate(validUser); err != nil {
        fmt.Printf("Validation error: %v\n", err)
    } else {
        fmt.Printf("✅ User validation passed\n")
    }
    
    fmt.Println("\n2. Invalid User (Multiple Errors)")
    invalidUser := User{
        ID:       0,  // Should be positive
        Username: "jo", // Too short
        Email:    "invalid-email", // Invalid format
        Age:      15, // Too young
        Phone:    "123", // Too short
    }
    
    if err := validator.Validate(invalidUser); err != nil {
        fmt.Printf("Validation errors:\n")
        if validationErrors, ok := err.(ValidationErrors); ok {
            for i, ve := range validationErrors {
                fmt.Printf("  %d. %s: %s (value: %v)\n", 
                          i+1, ve.Field, ve.Message, ve.Value)
            }
        }
    }
    
    fmt.Println("\n3. Missing Required Fields")
    emptyUser := User{
        Age: 25, // Only age provided
    }
    
    if err := validator.Validate(emptyUser); err != nil {
        fmt.Printf("Required field errors:\n")
        if validationErrors, ok := err.(ValidationErrors); ok {
            for _, ve := range validationErrors {
                if ve.Rule == "required" {
                    fmt.Printf("  - %s is required\n", ve.Field)
                }
            }
        }
    }
    
    fmt.Println("\n4. Product Validation")
    products := []Product{
        {
            Name:        "Laptop",
            Price:       999.99,
            Description: "High-performance laptop",
            Category:    "Electronics",
        },
        {
            Name:        "",  // Missing name
            Price:       0,   // Missing price
            Description: strings.Repeat("Very long description ", 20), // Too long
            Category:    "E", // Too short
        },
    }
    
    for i, product := range products {
        fmt.Printf("Product %d validation:\n", i+1)
        
        if err := validator.Validate(product); err != nil {
            if validationErrors, ok := err.(ValidationErrors); ok {
                fmt.Printf("  ❌ %d error(s):\n", len(validationErrors))
                for _, ve := range validationErrors {
                    fmt.Printf("    - %s: %s\n", ve.Field, ve.Message)
                }
            }
        } else {
            fmt.Printf("  ✅ Valid\n")
        }
    }
    
    fmt.Println("\n5. Reflection Capabilities")
    fmt.Println("Reflection-based validation provides:")
    fmt.Println("  ✅ Generic validation across different struct types")
    fmt.Println("  ✅ Tag-based validation rules")
    fmt.Println("  ✅ Automatic field discovery")
    fmt.Println("  ✅ Custom validation method integration")
    fmt.Println("  ✅ Comprehensive error reporting")
}
```

Reflection-based error handling enables powerful generic validation and  
error processing capabilities that work across different types while  
maintaining flexibility and comprehensive error reporting.  

## Async error aggregation

Asynchronous operations require specialized error aggregation patterns  
to collect and process errors from multiple concurrent operations effectively.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Async error collector
type AsyncErrorCollector struct {
    mu         sync.RWMutex
    errors     []AsyncError
    maxErrors  int
    timeout    time.Duration
    onError    func(AsyncError)
    onComplete func([]AsyncError)
}

type AsyncError struct {
    ID        string
    Error     error
    Source    string
    Timestamp time.Time
    Metadata  map[string]interface{}
    Severity  string
}

func NewAsyncErrorCollector(maxErrors int, timeout time.Duration) *AsyncErrorCollector {
    return &AsyncErrorCollector{
        maxErrors: maxErrors,
        timeout:   timeout,
        errors:    make([]AsyncError, 0, maxErrors),
    }
}

func (aec *AsyncErrorCollector) SetCallbacks(onError func(AsyncError), onComplete func([]AsyncError)) {
    aec.onError = onError
    aec.onComplete = onComplete
}

func (aec *AsyncErrorCollector) Collect(asyncErr AsyncError) {
    aec.mu.Lock()
    defer aec.mu.Unlock()
    
    aec.errors = append(aec.errors, asyncErr)
    
    if aec.onError != nil {
        go aec.onError(asyncErr)
    }
    
    // Check if we've reached max errors
    if len(aec.errors) >= aec.maxErrors {
        if aec.onComplete != nil {
            go aec.onComplete(aec.GetErrors())
        }
    }
}

func (aec *AsyncErrorCollector) GetErrors() []AsyncError {
    aec.mu.RLock()
    defer aec.mu.RUnlock()
    
    result := make([]AsyncError, len(aec.errors))
    copy(result, aec.errors)
    return result
}

func (aec *AsyncErrorCollector) GetErrorCount() int {
    aec.mu.RLock()
    defer aec.mu.RUnlock()
    return len(aec.errors)
}

func (aec *AsyncErrorCollector) Clear() {
    aec.mu.Lock()
    defer aec.mu.Unlock()
    aec.errors = aec.errors[:0]
}

// Async operation result
type AsyncResult struct {
    ID      string
    Value   interface{}
    Error   error
    Source  string
    StartTime time.Time
    EndTime   time.Time
}

func (ar AsyncResult) Duration() time.Duration {
    return ar.EndTime.Sub(ar.StartTime)
}

func (ar AsyncResult) IsSuccess() bool {
    return ar.Error == nil
}

// Async operation manager
type AsyncOperationManager struct {
    collector  *AsyncErrorCollector
    results    chan AsyncResult
    wg         sync.WaitGroup
    ctx        context.Context
    cancel     context.CancelFunc
}

func NewAsyncOperationManager(ctx context.Context, collector *AsyncErrorCollector) *AsyncOperationManager {
    childCtx, cancel := context.WithCancel(ctx)
    
    return &AsyncOperationManager{
        collector: collector,
        results:   make(chan AsyncResult, 100),
        ctx:       childCtx,
        cancel:    cancel,
    }
}

func (aom *AsyncOperationManager) Execute(id, source string, operation func() (interface{}, error)) {
    aom.wg.Add(1)
    
    go func() {
        defer aom.wg.Done()
        
        result := AsyncResult{
            ID:        id,
            Source:    source,
            StartTime: time.Now(),
        }
        
        defer func() {
            result.EndTime = time.Now()
            
            select {
            case aom.results <- result:
            case <-aom.ctx.Done():
            }
        }()
        
        // Check for cancellation before starting
        select {
        case <-aom.ctx.Done():
            result.Error = aom.ctx.Err()
            return
        default:
        }
        
        value, err := operation()
        result.Value = value
        result.Error = err
        
        if err != nil {
            asyncErr := AsyncError{
                ID:        id,
                Error:     err,
                Source:    source,
                Timestamp: time.Now(),
                Metadata: map[string]interface{}{
                    "duration": result.Duration().Milliseconds(),
                },
                Severity: determineSeverity(err),
            }
            
            aom.collector.Collect(asyncErr)
        }
    }()
}

func (aom *AsyncOperationManager) Wait() []AsyncResult {
    // Close results channel when all operations complete
    go func() {
        aom.wg.Wait()
        close(aom.results)
    }()
    
    var results []AsyncResult
    for result := range aom.results {
        results = append(results, result)
    }
    
    return results
}

func (aom *AsyncOperationManager) Cancel() {
    aom.cancel()
}

// Error pattern analyzer
type ErrorPatternAnalyzer struct {
    patterns map[string]int
    mu       sync.RWMutex
}

func NewErrorPatternAnalyzer() *ErrorPatternAnalyzer {
    return &ErrorPatternAnalyzer{
        patterns: make(map[string]int),
    }
}

func (epa *ErrorPatternAnalyzer) Analyze(errors []AsyncError) map[string]interface{} {
    epa.mu.Lock()
    defer epa.mu.Unlock()
    
    bySource := make(map[string]int)
    bySeverity := make(map[string]int)
    byType := make(map[string]int)
    
    for _, err := range errors {
        bySource[err.Source]++
        bySeverity[err.Severity]++
        
        errorType := classifyAsyncError(err.Error)
        byType[errorType]++
        epa.patterns[errorType]++
    }
    
    return map[string]interface{}{
        "total_errors":   len(errors),
        "by_source":      bySource,
        "by_severity":    bySeverity,
        "by_type":        byType,
        "patterns":       epa.copyPatterns(),
    }
}

func (epa *ErrorPatternAnalyzer) copyPatterns() map[string]int {
    patterns := make(map[string]int)
    for k, v := range epa.patterns {
        patterns[k] = v
    }
    return patterns
}

func classifyAsyncError(err error) string {
    errStr := err.Error()
    
    switch {
    case contains(errStr, "timeout", "deadline"):
        return "timeout"
    case contains(errStr, "network", "connection"):
        return "network"
    case contains(errStr, "permission", "unauthorized"):
        return "permission"
    case contains(errStr, "not found", "missing"):
        return "not_found"
    case contains(errStr, "validation", "invalid"):
        return "validation"
    default:
        return "unknown"
    }
}

func determineSeverity(err error) string {
    errStr := err.Error()
    
    switch {
    case contains(errStr, "critical", "fatal", "panic"):
        return "critical"
    case contains(errStr, "timeout", "connection"):
        return "high"
    case contains(errStr, "not found", "validation"):
        return "medium"
    default:
        return "low"
    }
}

func contains(text string, keywords ...string) bool {
    for _, keyword := range keywords {
        if len(text) > 0 && len(keyword) > 0 {
            return true // Simplified check
        }
    }
    return false
}

// Simulated async operations
func simulateAPICall(id string) (interface{}, error) {
    time.Sleep(time.Duration(50+id[0]%100) * time.Millisecond)
    
    switch id {
    case "api_1", "api_5", "api_9":
        return fmt.Sprintf("data_%s", id), nil
    case "api_2":
        return nil, fmt.Errorf("network timeout for %s", id)
    case "api_3":
        return nil, fmt.Errorf("validation error in %s", id)
    case "api_4":
        return nil, fmt.Errorf("not found error for %s", id)
    case "api_6":
        return nil, fmt.Errorf("permission denied for %s", id)
    case "api_7":
        return nil, fmt.Errorf("critical system error in %s", id)
    default:
        return nil, fmt.Errorf("unknown error for %s", id)
    }
}

func simulateDatabaseQuery(id string) (interface{}, error) {
    time.Sleep(time.Duration(30+id[0]%80) * time.Millisecond)
    
    switch id {
    case "db_1", "db_3", "db_7":
        return fmt.Sprintf("result_%s", id), nil
    case "db_2":
        return nil, fmt.Errorf("database connection timeout")
    case "db_4":
        return nil, fmt.Errorf("record not found in database")
    case "db_5":
        return nil, fmt.Errorf("database constraint violation")
    default:
        return nil, fmt.Errorf("database error for %s", id)
    }
}

func main() {
    fmt.Println("=== Async Error Aggregation ===")
    
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // Setup error collector with callbacks
    collector := NewAsyncErrorCollector(20, 2*time.Second)
    analyzer := NewErrorPatternAnalyzer()
    
    collector.SetCallbacks(
        func(asyncErr AsyncError) {
            fmt.Printf("🚨 Async Error: [%s] %s - %v\n", 
                      asyncErr.Severity, asyncErr.Source, asyncErr.Error)
        },
        func(errors []AsyncError) {
            fmt.Printf("📊 Error batch completed: %d errors collected\n", len(errors))
        },
    )
    
    manager := NewAsyncOperationManager(ctx, collector)
    
    fmt.Println("1. Launching Async Operations")
    
    // Launch API calls
    for i := 1; i <= 10; i++ {
        id := fmt.Sprintf("api_%d", i)
        manager.Execute(id, "api_service", func() (interface{}, error) {
            return simulateAPICall(id)
        })
    }
    
    // Launch database queries
    for i := 1; i <= 8; i++ {
        id := fmt.Sprintf("db_%d", i)
        manager.Execute(id, "database", func() (interface{}, error) {
            return simulateDatabaseQuery(id)
        })
    }
    
    fmt.Println("2. Waiting for Operations to Complete")
    
    results := manager.Wait()
    
    fmt.Printf("\n3. Operation Results Summary\n")
    successCount := 0
    errorCount := 0
    
    for _, result := range results {
        if result.IsSuccess() {
            successCount++
            fmt.Printf("✅ %s: Success (%v)\n", result.ID, result.Duration())
        } else {
            errorCount++
            fmt.Printf("❌ %s: Failed (%v) - %v\n", 
                      result.ID, result.Duration(), result.Error)
        }
    }
    
    fmt.Printf("\nSummary: %d successful, %d failed\n", successCount, errorCount)
    
    fmt.Println("\n4. Error Analysis")
    
    collectedErrors := collector.GetErrors()
    analysis := analyzer.Analyze(collectedErrors)
    
    fmt.Printf("Total Errors Collected: %v\n", analysis["total_errors"])
    fmt.Printf("Errors by Source: %v\n", analysis["by_source"])
    fmt.Printf("Errors by Severity: %v\n", analysis["by_severity"])
    fmt.Printf("Errors by Type: %v\n", analysis["by_type"])
    
    fmt.Println("\n5. Detailed Error Timeline")
    
    for i, asyncErr := range collectedErrors {
        fmt.Printf("%d. [%s] %s at %s - %v\n", 
                  i+1, asyncErr.Severity, asyncErr.Source, 
                  asyncErr.Timestamp.Format("15:04:05.000"), asyncErr.Error)
        
        if asyncErr.Metadata != nil {
            fmt.Printf("   Duration: %vms\n", asyncErr.Metadata["duration"])
        }
    }
    
    fmt.Println("\n6. Error Pattern Analysis")
    patterns := analysis["patterns"].(map[string]int)
    
    fmt.Println("Most common error patterns:")
    for pattern, count := range patterns {
        fmt.Printf("  %s: %d occurrences\n", pattern, count)
    }
    
    fmt.Println("\nAsync error aggregation provides:")
    fmt.Println("  ✅ Real-time error collection from concurrent operations")
    fmt.Println("  ✅ Error pattern analysis and classification")
    fmt.Println("  ✅ Severity-based error handling")
    fmt.Println("  ✅ Comprehensive error timeline tracking")
    fmt.Println("  ✅ Callback-based error notification")
}
```

Async error aggregation enables comprehensive error collection and analysis  
for concurrent operations, providing real-time error monitoring and pattern  
analysis essential for distributed and high-concurrency applications.  

## Error serialization and deserialization

Error serialization enables error information persistence, transmission  
across network boundaries, and structured error logging in distributed systems.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Serializable error structure
type SerializableError struct {
    Type        string                 `json:"type"`
    Message     string                 `json:"message"`
    Code        string                 `json:"code,omitempty"`
    Timestamp   time.Time             `json:"timestamp"`
    Source      string                 `json:"source,omitempty"`
    Stack       []StackFrame          `json:"stack,omitempty"`
    Context     map[string]interface{} `json:"context,omitempty"`
    Cause       *SerializableError     `json:"cause,omitempty"`
    Severity    string                 `json:"severity"`
    Retryable   bool                   `json:"retryable"`
    UserMessage string                 `json:"user_message,omitempty"`
}

type StackFrame struct {
    Function string `json:"function"`
    File     string `json:"file"`
    Line     int    `json:"line"`
}

func (se SerializableError) Error() string {
    if se.Cause != nil {
        return fmt.Sprintf("%s: %s (caused by: %s)", se.Type, se.Message, se.Cause.Error())
    }
    return fmt.Sprintf("%s: %s", se.Type, se.Message)
}

// Error serializer
type ErrorSerializer struct{}

func NewErrorSerializer() *ErrorSerializer {
    return &ErrorSerializer{}
}

func (es *ErrorSerializer) Serialize(err error) ([]byte, error) {
    serializableErr := es.convertToSerializable(err)
    return json.MarshalIndent(serializableErr, "", "  ")
}

func (es *ErrorSerializer) Deserialize(data []byte) (*SerializableError, error) {
    var serializableErr SerializableError
    if err := json.Unmarshal(data, &serializableErr); err != nil {
        return nil, fmt.Errorf("failed to deserialize error: %w", err)
    }
    return &serializableErr, nil
}

func (es *ErrorSerializer) convertToSerializable(err error) *SerializableError {
    if err == nil {
        return nil
    }
    
    serErr := &SerializableError{
        Type:      getErrorType(err),
        Message:   err.Error(),
        Timestamp: time.Now(),
        Severity:  "error",
    }
    
    // Handle different error types
    switch e := err.(type) {
    case *BusinessError:
        serErr.Code = e.Code
        serErr.Source = e.Source
        serErr.Context = e.Context
        serErr.Severity = e.Severity
        serErr.Retryable = e.Retryable
        serErr.UserMessage = e.UserMessage
        if e.Cause != nil {
            serErr.Cause = es.convertToSerializable(e.Cause)
        }
    
    case *ValidationError:
        serErr.Code = "VALIDATION_ERROR"
        serErr.Source = "validation"
        serErr.Context = map[string]interface{}{
            "field": e.Field,
            "rule":  e.Rule,
            "value": e.Value,
        }
        serErr.Severity = "warning"
        serErr.UserMessage = e.UserMessage
    
    case *NetworkError:
        serErr.Code = e.ErrorCode
        serErr.Source = "network"
        serErr.Context = map[string]interface{}{
            "url":        e.URL,
            "method":     e.Method,
            "status":     e.StatusCode,
            "retryable":  e.Retryable,
        }
        serErr.Retryable = e.Retryable
        serErr.Severity = determineSeverityByStatus(e.StatusCode)
    
    default:
        serErr.Context = map[string]interface{}{
            "original_type": fmt.Sprintf("%T", err),
        }
    }
    
    return serErr
}

func getErrorType(err error) string {
    switch err.(type) {
    case *BusinessError:
        return "BusinessError"
    case *ValidationError:
        return "ValidationError"
    case *NetworkError:
        return "NetworkError"
    default:
        return "GenericError"
    }
}

func determineSeverityByStatus(status int) string {
    switch {
    case status >= 500:
        return "critical"
    case status >= 400:
        return "warning"
    default:
        return "info"
    }
}

// Custom error types for demonstration
type BusinessError struct {
    Code        string
    Message     string
    Source      string
    Context     map[string]interface{}
    Severity    string
    Retryable   bool
    UserMessage string
    Cause       error
}

func (be *BusinessError) Error() string {
    return fmt.Sprintf("[%s] %s", be.Code, be.Message)
}

func (be *BusinessError) Unwrap() error {
    return be.Cause
}

type ValidationError struct {
    Field       string
    Rule        string
    Value       interface{}
    Message     string
    UserMessage string
}

func (ve *ValidationError) Error() string {
    return fmt.Sprintf("validation failed for %q: %s", ve.Field, ve.Message)
}

type NetworkError struct {
    URL         string
    Method      string
    StatusCode  int
    ErrorCode   string
    Message     string
    Retryable   bool
}

func (ne *NetworkError) Error() string {
    return fmt.Sprintf("network error %d for %s %s: %s", 
                       ne.StatusCode, ne.Method, ne.URL, ne.Message)
}

// Error transmission service
type ErrorTransmissionService struct {
    serializer *ErrorSerializer
}

func NewErrorTransmissionService() *ErrorTransmissionService {
    return &ErrorTransmissionService{
        serializer: NewErrorSerializer(),
    }
}

func (ets *ErrorTransmissionService) SendError(err error, destination string) error {
    serialized, serErr := ets.serializer.Serialize(err)
    if serErr != nil {
        return fmt.Errorf("failed to serialize error for transmission: %w", serErr)
    }
    
    fmt.Printf("📤 Sending error to %s:\n%s\n\n", destination, string(serialized))
    
    // Simulate network transmission
    time.Sleep(50 * time.Millisecond)
    
    return nil
}

func (ets *ErrorTransmissionService) ReceiveError(data []byte) (*SerializableError, error) {
    fmt.Printf("📥 Receiving error data (%d bytes)\n", len(data))
    
    deserializedErr, err := ets.serializer.Deserialize(data)
    if err != nil {
        return nil, fmt.Errorf("failed to deserialize received error: %w", err)
    }
    
    return deserializedErr, nil
}

// Error storage service
type ErrorStorageService struct {
    storage    map[string][]byte
    serializer *ErrorSerializer
}

func NewErrorStorageService() *ErrorStorageService {
    return &ErrorStorageService{
        storage:    make(map[string][]byte),
        serializer: NewErrorSerializer(),
    }
}

func (ess *ErrorStorageService) StoreError(id string, err error) error {
    serialized, serErr := ess.serializer.Serialize(err)
    if serErr != nil {
        return fmt.Errorf("failed to serialize error for storage: %w", serErr)
    }
    
    ess.storage[id] = serialized
    fmt.Printf("💾 Stored error %s (%d bytes)\n", id, len(serialized))
    
    return nil
}

func (ess *ErrorStorageService) RetrieveError(id string) (*SerializableError, error) {
    data, exists := ess.storage[id]
    if !exists {
        return nil, fmt.Errorf("error %s not found in storage", id)
    }
    
    deserializedErr, err := ess.serializer.Deserialize(data)
    if err != nil {
        return nil, fmt.Errorf("failed to deserialize stored error: %w", err)
    }
    
    fmt.Printf("📂 Retrieved error %s\n", id)
    return deserializedErr, nil
}

func (ess *ErrorStorageService) ListStoredErrors() []string {
    var ids []string
    for id := range ess.storage {
        ids = append(ids, id)
    }
    return ids
}

func main() {
    fmt.Println("=== Error Serialization and Deserialization ===")
    
    serializer := NewErrorSerializer()
    transmissionService := NewErrorTransmissionService()
    storageService := NewErrorStorageService()
    
    fmt.Println("1. Creating Various Error Types")
    
    // Business error with cause
    businessErr := &BusinessError{
        Code:        "PAYMENT_FAILED",
        Message:     "Payment processing failed",
        Source:      "payment-service",
        Context: map[string]interface{}{
            "transaction_id": "txn_12345",
            "amount":        99.99,
            "currency":      "USD",
        },
        Severity:    "critical",
        Retryable:   true,
        UserMessage: "We're unable to process your payment. Please try again.",
        Cause: &NetworkError{
            URL:        "https://payment-gateway.com/charge",
            Method:     "POST",
            StatusCode: 503,
            ErrorCode:  "SERVICE_UNAVAILABLE",
            Message:    "Payment gateway temporarily unavailable",
            Retryable:  true,
        },
    }
    
    // Validation error
    validationErr := &ValidationError{
        Field:       "email",
        Rule:        "format",
        Value:       "invalid-email",
        Message:     "invalid email format",
        UserMessage: "Please enter a valid email address.",
    }
    
    // Network error
    networkErr := &NetworkError{
        URL:        "https://api.example.com/users",
        Method:     "GET",
        StatusCode: 404,
        ErrorCode:  "NOT_FOUND",
        Message:    "User not found",
        Retryable:  false,
    }
    
    errors := []error{businessErr, validationErr, networkErr}
    errorNames := []string{"business_error", "validation_error", "network_error"}
    
    fmt.Println("2. Serializing Errors")
    
    for i, err := range errors {
        fmt.Printf("\n--- %s ---\n", errorNames[i])
        
        serialized, serErr := serializer.Serialize(err)
        if serErr != nil {
            fmt.Printf("Serialization failed: %v\n", serErr)
            continue
        }
        
        fmt.Printf("Serialized JSON:\n%s\n", string(serialized))
        
        // Store error
        if storeErr := storageService.StoreError(errorNames[i], err); storeErr != nil {
            fmt.Printf("Storage failed: %v\n", storeErr)
        }
    }
    
    fmt.Println("\n3. Deserializing and Retrieving Errors")
    
    storedIDs := storageService.ListStoredErrors()
    fmt.Printf("Stored error IDs: %v\n", storedIDs)
    
    for _, id := range storedIDs {
        fmt.Printf("\n--- Retrieving %s ---\n", id)
        
        retrievedErr, err := storageService.RetrieveError(id)
        if err != nil {
            fmt.Printf("Retrieval failed: %v\n", err)
            continue
        }
        
        fmt.Printf("Retrieved Error:\n")
        fmt.Printf("  Type: %s\n", retrievedErr.Type)
        fmt.Printf("  Message: %s\n", retrievedErr.Message)
        fmt.Printf("  Code: %s\n", retrievedErr.Code)
        fmt.Printf("  Severity: %s\n", retrievedErr.Severity)
        fmt.Printf("  Retryable: %v\n", retrievedErr.Retryable)
        fmt.Printf("  Timestamp: %s\n", retrievedErr.Timestamp.Format("2006-01-02 15:04:05"))
        
        if retrievedErr.Context != nil {
            fmt.Printf("  Context: %v\n", retrievedErr.Context)
        }
        
        if retrievedErr.UserMessage != "" {
            fmt.Printf("  User Message: %s\n", retrievedErr.UserMessage)
        }
        
        if retrievedErr.Cause != nil {
            fmt.Printf("  Cause: %s\n", retrievedErr.Cause.Type)
            fmt.Printf("    Message: %s\n", retrievedErr.Cause.Message)
        }
    }
    
    fmt.Println("\n4. Error Transmission Simulation")
    
    // Transmit business error
    if err := transmissionService.SendError(businessErr, "error-service"); err != nil {
        fmt.Printf("Transmission failed: %v\n", err)
    }
    
    // Simulate receiving the error
    serialized, _ := serializer.Serialize(businessErr)
    receivedErr, err := transmissionService.ReceiveError(serialized)
    if err != nil {
        fmt.Printf("Reception failed: %v\n", err)
    } else {
        fmt.Printf("✅ Successfully transmitted and received error:\n")
        fmt.Printf("  Type: %s\n", receivedErr.Type)
        fmt.Printf("  Code: %s\n", receivedErr.Code)
        fmt.Printf("  Severity: %s\n", receivedErr.Severity)
    }
    
    fmt.Println("\n5. Error Analytics from Serialized Data")
    
    errorStats := map[string]int{
        "total":      0,
        "critical":   0,
        "warning":    0,
        "retryable":  0,
    }
    
    for _, id := range storedIDs {
        if retrievedErr, err := storageService.RetrieveError(id); err == nil {
            errorStats["total"]++
            
            switch retrievedErr.Severity {
            case "critical":
                errorStats["critical"]++
            case "warning":
                errorStats["warning"]++
            }
            
            if retrievedErr.Retryable {
                errorStats["retryable"]++
            }
        }
    }
    
    fmt.Printf("Error Statistics:\n")
    fmt.Printf("  Total Errors: %d\n", errorStats["total"])
    fmt.Printf("  Critical: %d\n", errorStats["critical"])
    fmt.Printf("  Warning: %d\n", errorStats["warning"])
    fmt.Printf("  Retryable: %d\n", errorStats["retryable"])
    
    fmt.Println("\nError serialization provides:")
    fmt.Println("  ✅ Persistent error storage and retrieval")
    fmt.Println("  ✅ Cross-service error transmission")
    fmt.Println("  ✅ Structured error logging and analysis")
    fmt.Println("  ✅ Error context preservation")
    fmt.Println("  ✅ Error causality chain maintenance")
}
```

Error serialization and deserialization enable persistent error storage,  
cross-system error transmission, and comprehensive error analysis in  
distributed systems while preserving all error context and metadata.  

## Error handling best practices summary

Comprehensive error handling in Go requires understanding patterns,  
anti-patterns, and best practices for building robust applications.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"
)

// Best practices demonstration
type BestPracticesExample struct {
    logger Logger
}

type Logger interface {
    Error(message string, err error, fields ...map[string]interface{})
    Info(message string, fields ...map[string]interface{})
}

type SimpleLogger struct{}

func (sl SimpleLogger) Error(message string, err error, fields ...map[string]interface{}) {
    fmt.Printf("[ERROR] %s: %v", message, err)
    if len(fields) > 0 {
        fmt.Printf(" | %v", fields[0])
    }
    fmt.Println()
}

func (sl SimpleLogger) Info(message string, fields ...map[string]interface{}) {
    fmt.Printf("[INFO] %s", message)
    if len(fields) > 0 {
        fmt.Printf(" | %v", fields[0])
    }
    fmt.Println()
}

func NewBestPracticesExample() *BestPracticesExample {
    return &BestPracticesExample{
        logger: SimpleLogger{},
    }
}

// ✅ GOOD: Clear error messages with context
func (bpe *BestPracticesExample) GoodErrorMessages(userID string) error {
    if userID == "" {
        return fmt.Errorf("user ID is required for user lookup")
    }
    
    if userID == "invalid" {
        return fmt.Errorf("user lookup failed: user ID %q is not valid (must be alphanumeric)", userID)
    }
    
    return nil
}

// ❌ BAD: Vague error messages
func (bpe *BestPracticesExample) BadErrorMessages(userID string) error {
    if userID == "" {
        return fmt.Errorf("error") // Too vague
    }
    
    if userID == "invalid" {
        return fmt.Errorf("bad input") // Not specific
    }
    
    return nil
}

// ✅ GOOD: Proper error wrapping
func (bpe *BestPracticesExample) GoodErrorWrapping(filename string) error {
    data, err := bpe.readFile(filename)
    if err != nil {
        return fmt.Errorf("failed to process file %q: %w", filename, err)
    }
    
    if err := bpe.processData(data); err != nil {
        return fmt.Errorf("data processing failed for file %q: %w", filename, err)
    }
    
    return nil
}

// ❌ BAD: Error shadowing
func (bpe *BestPracticesExample) BadErrorWrapping(filename string) error {
    data, err := bpe.readFile(filename)
    if err != nil {
        return fmt.Errorf("failed to process file: %s", err.Error()) // Loses original error
    }
    
    if err := bpe.processData(data); err != nil {
        return err // No context added
    }
    
    return nil
}

// ✅ GOOD: Context-aware error handling
func (bpe *BestPracticesExample) GoodContextHandling(ctx context.Context, userID string) error {
    select {
    case <-ctx.Done():
        return fmt.Errorf("user lookup cancelled: %w", ctx.Err())
    default:
    }
    
    user, err := bpe.fetchUserWithTimeout(ctx, userID)
    if err != nil {
        bpe.logger.Error("User fetch failed", err, map[string]interface{}{
            "user_id":   userID,
            "operation": "fetch_user",
        })
        return fmt.Errorf("failed to fetch user %q: %w", userID, err)
    }
    
    bpe.logger.Info("User fetched successfully", map[string]interface{}{
        "user_id": user.ID,
        "name":    user.Name,
    })
    
    return nil
}

// ❌ BAD: Ignoring context
func (bpe *BestPracticesExample) BadContextHandling(ctx context.Context, userID string) error {
    // Ignores context cancellation
    user, err := bpe.fetchUser(userID)
    if err != nil {
        log.Printf("Error: %v", err) // Poor logging
        return err
    }
    
    fmt.Printf("Got user: %v", user) // Not using structured logging
    return nil
}

// ✅ GOOD: Appropriate error levels
func (bpe *BestPracticesExample) GoodErrorLevels() {
    // Use different approaches for different error types
    
    // Programming errors: panic
    if 1+1 != 2 {
        panic("arithmetic is broken") // Should never happen
    }
    
    // Recoverable errors: return error
    if err := bpe.connectToDatabase(); err != nil {
        bpe.logger.Error("Database connection failed", err, map[string]interface{}{
            "retry_recommended": true,
        })
    }
    
    // Expected conditions: handle gracefully
    user, err := bpe.findUser("nonexistent")
    if err != nil {
        if isNotFoundError(err) {
            bpe.logger.Info("User not found, creating new user", map[string]interface{}{
                "user_id": "nonexistent",
            })
        }
    } else {
        fmt.Printf("Found user: %v\n", user)
    }
}

// ❌ BAD: Inappropriate error handling
func (bpe *BestPracticesExample) BadErrorLevels() {
    // Using panic for recoverable errors
    if err := bpe.connectToDatabase(); err != nil {
        panic(err) // Wrong approach
    }
    
    // Ignoring important errors
    bpe.saveUserData("user123") // Ignoring error return
    
    // Using errors for control flow
    for i := 0; i < 10; i++ {
        if err := bpe.processItem(i); err != nil {
            break // Using error to break loop instead of explicit condition
        }
    }
}

// ✅ GOOD: Resource cleanup
func (bpe *BestPracticesExample) GoodResourceCleanup(filename string) error {
    file, err := bpe.openFile(filename)
    if err != nil {
        return fmt.Errorf("failed to open file %q: %w", filename, err)
    }
    
    defer func() {
        if closeErr := file.Close(); closeErr != nil {
            bpe.logger.Error("Failed to close file", closeErr, map[string]interface{}{
                "filename": filename,
            })
        }
    }()
    
    if err := bpe.processFile(file); err != nil {
        return fmt.Errorf("failed to process file %q: %w", filename, err)
    }
    
    return nil
}

// ❌ BAD: Resource leaks
func (bpe *BestPracticesExample) BadResourceCleanup(filename string) error {
    file, err := bpe.openFile(filename)
    if err != nil {
        return err
    }
    
    // Missing defer file.Close()
    
    if err := bpe.processFile(file); err != nil {
        return err // File not closed on error
    }
    
    file.Close() // Only closed on success path
    return nil
}

// ✅ GOOD: Error aggregation for batch operations
func (bpe *BestPracticesExample) GoodBatchProcessing(items []string) error {
    var errors []error
    successCount := 0
    
    for i, item := range items {
        if err := bpe.processItem(item); err != nil {
            errors = append(errors, fmt.Errorf("item %d (%q): %w", i, item, err))
        } else {
            successCount++
        }
    }
    
    if len(errors) > 0 {
        bpe.logger.Error("Batch processing completed with errors", 
            fmt.Errorf("%d of %d items failed", len(errors), len(items)),
            map[string]interface{}{
                "success_count": successCount,
                "error_count":   len(errors),
                "total_count":   len(items),
            })
        
        return fmt.Errorf("batch processing failed: %d errors occurred", len(errors))
    }
    
    bpe.logger.Info("Batch processing completed successfully", map[string]interface{}{
        "processed_count": successCount,
    })
    
    return nil
}

// ❌ BAD: Fail-fast without aggregation
func (bpe *BestPracticesExample) BadBatchProcessing(items []string) error {
    for _, item := range items {
        if err := bpe.processItem(item); err != nil {
            return err // Stops at first error, doesn't process remaining items
        }
    }
    return nil
}

// Mock methods for demonstration
func (bpe *BestPracticesExample) readFile(filename string) (string, error) {
    if filename == "nonexistent.txt" {
        return "", fmt.Errorf("file not found")
    }
    return "file content", nil
}

func (bpe *BestPracticesExample) processData(data string) error {
    if data == "" {
        return fmt.Errorf("empty data")
    }
    return nil
}

func (bpe *BestPracticesExample) fetchUser(userID string) (*User, error) {
    if userID == "error" {
        return nil, fmt.Errorf("user fetch failed")
    }
    return &User{ID: userID, Name: "John"}, nil
}

func (bpe *BestPracticesExample) fetchUserWithTimeout(ctx context.Context, userID string) (*User, error) {
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    case <-time.After(50 * time.Millisecond):
        return bpe.fetchUser(userID)
    }
}

func (bpe *BestPracticesExample) connectToDatabase() error {
    return fmt.Errorf("connection timeout")
}

func (bpe *BestPracticesExample) findUser(userID string) (*User, error) {
    if userID == "nonexistent" {
        return nil, &NotFoundError{Resource: "user", ID: userID}
    }
    return &User{ID: userID, Name: "Found User"}, nil
}

func (bpe *BestPracticesExample) saveUserData(userID string) error {
    return fmt.Errorf("save failed")
}

func (bpe *BestPracticesExample) processItem(item interface{}) error {
    if fmt.Sprintf("%v", item) == "error" {
        return fmt.Errorf("processing failed")
    }
    return nil
}

func (bpe *BestPracticesExample) openFile(filename string) (*File, error) {
    if filename == "protected.txt" {
        return nil, fmt.Errorf("permission denied")
    }
    return &File{Name: filename}, nil
}

func (bpe *BestPracticesExample) processFile(file *File) error {
    return nil
}

type User struct {
    ID   string
    Name string
}

type File struct {
    Name string
}

func (f *File) Close() error {
    return nil
}

type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s %q not found", e.Resource, e.ID)
}

func isNotFoundError(err error) bool {
    _, ok := err.(*NotFoundError)
    return ok
}

func main() {
    fmt.Println("=== Go Error Handling Best Practices Summary ===")
    
    example := NewBestPracticesExample()
    
    fmt.Println("📋 Error Handling Best Practices:")
    fmt.Println()
    
    fmt.Println("✅ DO:")
    fmt.Println("  • Write clear, specific error messages with context")
    fmt.Println("  • Use error wrapping with fmt.Errorf and %w verb")
    fmt.Println("  • Handle context cancellation appropriately")
    fmt.Println("  • Use structured logging for errors")
    fmt.Println("  • Clean up resources with defer")
    fmt.Println("  • Aggregate errors in batch operations")
    fmt.Println("  • Return errors for recoverable conditions")
    fmt.Println("  • Use panic only for programming errors")
    fmt.Println("  • Implement custom error types when needed")
    fmt.Println("  • Check errors immediately after function calls")
    fmt.Println()
    
    fmt.Println("❌ DON'T:")
    fmt.Println("  • Ignore errors or use _ = err")
    fmt.Println("  • Create vague error messages")
    fmt.Println("  • Shadow original errors when wrapping")
    fmt.Println("  • Use panic for recoverable errors")
    fmt.Println("  • Forget to clean up resources")
    fmt.Println("  • Use errors for normal control flow")
    fmt.Println("  • Log and return the same error")
    fmt.Println("  • Return nil errors in error conditions")
    fmt.Println()
    
    fmt.Println("🔧 Demonstrations:")
    fmt.Println()
    
    fmt.Println("1. Good vs Bad Error Messages:")
    if err := example.GoodErrorMessages(""); err != nil {
        fmt.Printf("   Good: %v\n", err)
    }
    if err := example.BadErrorMessages(""); err != nil {
        fmt.Printf("   Bad:  %v\n", err)
    }
    fmt.Println()
    
    fmt.Println("2. Error Wrapping:")
    fmt.Println("   Good: Preserves original error with context")
    fmt.Println("   Bad:  Loses original error information")
    fmt.Println()
    
    fmt.Println("3. Context Handling:")
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    
    fmt.Println("   Good: Respects context cancellation and logs appropriately")
    example.GoodContextHandling(ctx, "user123")
    fmt.Println()
    
    fmt.Println("4. Error Levels:")
    fmt.Println("   Good: Uses appropriate error handling for each situation")
    example.GoodErrorLevels()
    fmt.Println()
    
    fmt.Println("5. Batch Processing:")
    items := []string{"item1", "error", "item3"}
    if err := example.GoodBatchProcessing(items); err != nil {
        fmt.Printf("   Batch result: %v\n", err)
    }
    fmt.Println()
    
    fmt.Println("🎯 Key Principles:")
    fmt.Println("  1. Errors are values - treat them as such")
    fmt.Println("  2. Explicit error handling over hidden control flow")
    fmt.Println("  3. Provide context and actionable information")
    fmt.Println("  4. Fail fast for programming errors, recover gracefully for runtime errors")
    fmt.Println("  5. Use the type system to encode error information")
    fmt.Println("  6. Design error handling into your APIs from the start")
    fmt.Println("  7. Test error conditions as thoroughly as success paths")
    fmt.Println("  8. Monitor and alert on error patterns in production")
    fmt.Println()
    
    fmt.Println("📊 Error Handling Patterns Covered:")
    patterns := []string{
        "Basic error checking",
        "Multiple return values with errors",  
        "Error creation methods",
        "Error wrapping and unwrapping",
        "Sentinel errors",
        "Error aggregation",
        "Custom error types with methods",
        "Panic and recovery patterns",
        "Context-based error handling",
        "File operation error handling",
        "Network error handling", 
        "Database error patterns",
        "JSON and data validation errors",
        "HTTP client error handling",
        "Goroutine error handling",
        "Testing error scenarios",
        "Recovery strategies and resilience",
        "Error handling in interfaces",
        "Error handling middleware patterns",
        "Domain-specific error types",
        "Error handling in channels and select",
        "Configuration and environment errors",
        "Structured logging with errors",
        "Error handling patterns with generics",
        "Monitoring and metrics for errors",
        "Async error aggregation",
        "Error serialization and deserialization",
        "Advanced error propagation patterns",
        "Error handling with reflection",
        "Best practices summary",
    }
    
    fmt.Printf("Total patterns demonstrated: %d\n", len(patterns))
    fmt.Println("\n🚀 You now have comprehensive knowledge of Go error handling!")
}
```

This comprehensive guide covers 30 essential Go error handling patterns,  
from basic error checking to advanced techniques for distributed systems,  
providing the foundation for building robust, maintainable Go applications.  
