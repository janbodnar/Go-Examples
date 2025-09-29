# Context

The Context package in Go provides a standardized way to carry cancellation  
signals, deadlines, timeouts, and request-scoped values across API boundaries  
and between processes. Introduced as a third-party package and later added to  
the standard library in Go 1.7, context has become an essential tool for  
building robust, scalable applications that need to handle cancellation and  
timeouts gracefully.

Context addresses several critical challenges in concurrent programming,  
particularly in server applications and distributed systems. Before context,  
managing cancellation and timeouts across multiple goroutines was complex and  
error-prone. Applications often struggled with resource leaks, hanging  
goroutines, and cascading failures when parts of the system needed to be  
cancelled or timed out.

The Context interface defines four methods: Deadline() returns any deadline  
set for the context, Done() returns a channel that closes when the context  
should be cancelled, Err() returns the cancellation reason, and Value()  
retrieves values associated with the context. This simple interface enables  
powerful patterns for request lifecycle management, resource cleanup, and  
graceful shutdown scenarios.

Context follows a tree-like hierarchy where child contexts inherit from parent  
contexts. When a parent context is cancelled, all its children are  
automatically cancelled, enabling cascading cancellation that prevents  
resource leaks and ensures consistent cleanup behavior. This design makes it  
natural to compose complex operations from simpler ones while maintaining  
proper cancellation semantics.

The package provides several functions for creating contexts: Background()  
creates a root context typically used in main functions and tests,  
WithCancel() creates a cancellable context, WithTimeout() and WithDeadline()  
add time-based cancellation, and WithValue() attaches request-scoped data.  
These building blocks enable sophisticated patterns for managing concurrent  
operations with proper timeout and cancellation handling.

Context has become a cornerstone of idiomatic Go programming, especially in  
server applications, HTTP handlers, database operations, and any code that  
performs I/O or long-running computations. Most Go libraries and frameworks  
now accept context as the first parameter in their APIs, establishing a  
consistent pattern for cancellation and timeout handling across the ecosystem.

Understanding context deeply is essential for writing production-ready Go  
applications. It enables proper resource management, prevents goroutine leaks,  
improves application responsiveness, and provides a foundation for building  
resilient distributed systems. The context package exemplifies Go's philosophy  
of providing simple, composable primitives that enable powerful patterns  
through composition rather than complexity.

Context also plays a crucial role in observability and distributed tracing.  
Request-scoped values can carry trace IDs, user information, and other  
metadata across service boundaries, enabling comprehensive monitoring and  
debugging capabilities. This makes context indispensable for microservices  
architectures and cloud-native applications.

## Basic context creation

Context creation starts with the Background() function, which returns an empty  
context that's never cancelled and has no deadline or values.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func basicOperation(ctx context.Context, name string) {
    fmt.Printf("Starting operation: %s\n", name)
    
    // Simulate some work
    select {
    case <-time.After(500 * time.Millisecond):
        fmt.Printf("Operation %s completed\n", name)
    case <-ctx.Done():
        fmt.Printf("Operation %s cancelled: %v\n", name, ctx.Err())
    }
}

func main() {
    // Create a background context
    ctx := context.Background()
    
    fmt.Println("Running operation with background context")
    basicOperation(ctx, "task-1")
    
    // Background context is never cancelled
    fmt.Printf("Context deadline: %v\n", ctx.Deadline())
    fmt.Printf("Context done channel: %v\n", ctx.Done())
    fmt.Printf("Context error: %v\n", ctx.Err())
}
```

Background context serves as the root of all context hierarchies and never  
expires or gets cancelled. It's typically used in main functions, tests,  
and when you need a context but don't have one available from caller.  

## Context with cancellation

WithCancel creates a cancellable context that can be manually cancelled using  
the returned cancel function.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func cancellableWork(ctx context.Context, workID int) {
    fmt.Printf("Worker %d starting\n", workID)
    
    for i := 1; i <= 10; i++ {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d cancelled at step %d: %v\n", 
                      workID, i, ctx.Err())
            return
        default:
            fmt.Printf("Worker %d: step %d\n", workID, i)
            time.Sleep(200 * time.Millisecond)
        }
    }
    
    fmt.Printf("Worker %d completed successfully\n", workID)
}

func main() {
    // Create cancellable context
    ctx, cancel := context.WithCancel(context.Background())
    
    // Start multiple workers
    go cancellableWork(ctx, 1)
    go cancellableWork(ctx, 2)
    go cancellableWork(ctx, 3)
    
    // Let them work for a while
    time.Sleep(1 * time.Second)
    
    // Cancel all workers
    fmt.Println("Cancelling all workers")
    cancel()
    
    // Give time for cancellation to propagate
    time.Sleep(500 * time.Millisecond)
    fmt.Println("All operations cancelled")
}
```

Manual cancellation is useful for stopping operations based on external  
events, user actions, or application logic. The cancel function can be  
called multiple times safely and will signal all goroutines watching the  
context's Done channel.  

## Context with timeout

WithTimeout creates a context that automatically cancels after a specified  
duration, providing time-based operation limits.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func timeoutSensitiveWork(ctx context.Context, taskName string, duration time.Duration) error {
    fmt.Printf("Starting %s (duration: %v)\n", taskName, duration)
    
    // Simulate work with the given duration
    select {
    case <-time.After(duration):
        fmt.Printf("Task %s completed successfully\n", taskName)
        return nil
    case <-ctx.Done():
        fmt.Printf("Task %s timed out: %v\n", taskName, ctx.Err())
        return ctx.Err()
    }
}

func main() {
    // Create context with 1-second timeout
    ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel() // Always call cancel to free resources
    
    // Start multiple tasks with different durations
    tasks := []struct {
        name     string
        duration time.Duration
    }{
        {"quick-task", 500 * time.Millisecond},
        {"medium-task", 800 * time.Millisecond},
        {"slow-task", 1500 * time.Millisecond},
    }
    
    for _, task := range tasks {
        if err := timeoutSensitiveWork(ctx, task.name, task.duration); err != nil {
            fmt.Printf("Task failed: %v\n", err)
        }
    }
    
    // Check deadline
    deadline, hasDeadline := ctx.Deadline()
    if hasDeadline {
        fmt.Printf("Context deadline: %v\n", deadline)
        fmt.Printf("Time remaining: %v\n", time.Until(deadline))
    }
}
```

Timeout contexts automatically handle time-based cancellation, making them  
ideal for operations that shouldn't run indefinitely. They help prevent  
resource exhaustion and improve application responsiveness.  

## Context with deadline

WithDeadline creates a context that cancels at a specific time, offering  
precise control over when operations should terminate.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func deadlineAwareOperation(ctx context.Context, operationName string) {
    deadline, hasDeadline := ctx.Deadline()
    if hasDeadline {
        fmt.Printf("Operation %s has deadline: %v\n", 
                  operationName, deadline.Format("15:04:05.000"))
    }
    
    // Simulate work in chunks
    for i := 1; i <= 5; i++ {
        select {
        case <-ctx.Done():
            fmt.Printf("Operation %s cancelled at iteration %d: %v\n", 
                      operationName, i, ctx.Err())
            return
        default:
            fmt.Printf("Operation %s: iteration %d\n", operationName, i)
            time.Sleep(300 * time.Millisecond)
        }
    }
    
    fmt.Printf("Operation %s completed successfully\n", operationName)
}

func main() {
    // Set deadline 1.2 seconds from now
    deadline := time.Now().Add(1200 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    fmt.Printf("Current time: %v\n", time.Now().Format("15:04:05.000"))
    fmt.Printf("Deadline set for: %v\n", deadline.Format("15:04:05.000"))
    
    // Run operations that may or may not complete before deadline
    go deadlineAwareOperation(ctx, "operation-A")
    go deadlineAwareOperation(ctx, "operation-B")
    
    // Wait for deadline to pass
    time.Sleep(2 * time.Second)
    
    fmt.Printf("Current time: %v\n", time.Now().Format("15:04:05.000"))
    fmt.Println("All operations should be complete or cancelled")
}
```

Deadline contexts provide absolute time-based cancellation, useful for  
operations that must complete before a specific deadline. They work well  
for scheduling tasks and managing time-sensitive operations.  

## Context with values

WithValue creates a context that carries request-scoped data, enabling  
information to flow through call chains without explicit parameter passing.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

type contextKey string

const (
    userIDKey     contextKey = "userID"
    requestIDKey  contextKey = "requestID"
    sessionIDKey  contextKey = "sessionID"
)

func authenticateUser(ctx context.Context) context.Context {
    // Simulate authentication and add user ID to context
    userID := "user_12345"
    fmt.Printf("Authenticated user: %s\n", userID)
    return context.WithValue(ctx, userIDKey, userID)
}

func processRequest(ctx context.Context, operation string) {
    userID := ctx.Value(userIDKey)
    requestID := ctx.Value(requestIDKey)
    sessionID := ctx.Value(sessionIDKey)
    
    fmt.Printf("Processing %s:\n", operation)
    fmt.Printf("  User ID: %v\n", userID)
    fmt.Printf("  Request ID: %v\n", requestID)
    fmt.Printf("  Session ID: %v\n", sessionID)
    
    // Simulate work
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("Completed %s\n\n", operation)
}

func main() {
    // Start with background context
    ctx := context.Background()
    
    // Add request ID
    ctx = context.WithValue(ctx, requestIDKey, "req_67890")
    
    // Add session ID
    ctx = context.WithValue(ctx, sessionIDKey, "session_abcdef")
    
    // Authenticate user (adds user ID)
    ctx = authenticateUser(ctx)
    
    // Process multiple operations with the enriched context
    processRequest(ctx, "fetch_profile")
    processRequest(ctx, "update_preferences")
    processRequest(ctx, "log_activity")
    
    // Demonstrate value retrieval patterns
    if userID, ok := ctx.Value(userIDKey).(string); ok {
        fmt.Printf("Final user ID: %s\n", userID)
    }
}
```

Context values should be used for request-scoped data like trace IDs,  
authentication tokens, and user information. Use typed keys to avoid  
collisions and always check type assertions when retrieving values.  

## Context hierarchy and inheritance

Child contexts inherit properties from their parents and can add additional  
constraints like timeouts or cancellation.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func operationWithTimeout(ctx context.Context, name string, duration time.Duration) {
    // Create child context with operation-specific timeout
    opCtx, cancel := context.WithTimeout(ctx, duration)
    defer cancel()
    
    fmt.Printf("Starting %s with %v timeout\n", name, duration)
    
    // Check parent context values
    if userID := ctx.Value("userID"); userID != nil {
        fmt.Printf("  User ID from parent: %v\n", userID)
    }
    
    select {
    case <-time.After(duration + 100*time.Millisecond):
        fmt.Printf("%s would complete\n", name)
    case <-opCtx.Done():
        fmt.Printf("%s cancelled: %v\n", name, opCtx.Err())
    }
}

func main() {
    // Create root context with values
    rootCtx := context.WithValue(context.Background(), "userID", "user123")
    
    // Create parent context with cancellation
    parentCtx, parentCancel := context.WithCancel(rootCtx)
    
    // Create child context with timeout
    childCtx, childCancel := context.WithTimeout(parentCtx, 2*time.Second)
    defer childCancel()
    
    // Start operations with different timeout levels
    go operationWithTimeout(childCtx, "quick-op", 500*time.Millisecond)
    go operationWithTimeout(childCtx, "medium-op", 1500*time.Millisecond)
    go operationWithTimeout(childCtx, "slow-op", 3000*time.Millisecond)
    
    // Let operations run
    time.Sleep(1 * time.Second)
    
    // Cancel parent - this should cascade to all children
    fmt.Println("Cancelling parent context...")
    parentCancel()
    
    // Wait to see cancellation effects
    time.Sleep(500 * time.Millisecond)
    
    // Show context hierarchy information
    fmt.Printf("Child context deadline: %v\n", childCtx.Deadline())
    fmt.Printf("Child context error: %v\n", childCtx.Err())
}
```

Context hierarchies enable fine-grained control over operation lifecycles.  
Child contexts can add additional constraints but cannot remove parent  
constraints. Cancelling a parent automatically cancels all children.  

## HTTP server with context

HTTP servers commonly use context for request lifecycle management,  
enabling graceful handling of client disconnections and timeouts.  

```go
package main

import (
    "context"
    "fmt"
    "net/http"
    "time"
)

func slowHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    fmt.Printf("Request started: %s %s\n", r.Method, r.URL.Path)
    
    // Simulate slow operation
    select {
    case <-time.After(5 * time.Second):
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, "Operation completed successfully")
        fmt.Println("Request completed normally")
    case <-ctx.Done():
        fmt.Printf("Request cancelled: %v\n", ctx.Err())
        w.WriteHeader(http.StatusRequestTimeout)
        fmt.Fprintf(w, "Request cancelled")
    }
}

func timeoutMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Create context with timeout
        ctx, cancel := context.WithTimeout(r.Context(), 3*time.Second)
        defer cancel()
        
        // Replace request context
        r = r.WithContext(ctx)
        next(w, r)
    }
}

func main() {
    http.HandleFunc("/slow", timeoutMiddleware(slowHandler))
    
    fmt.Println("Server starting on :8080")
    fmt.Println("Try: curl http://localhost:8080/slow")
    
    if err := http.ListenAndServe(":8080", nil); err != nil {
        fmt.Printf("Server error: %v\n", err)
    }
}
```

HTTP contexts automatically handle client disconnections and can be enhanced  
with timeouts through middleware. This prevents server resources from being  
tied up by abandoned requests.  

## Database operations with context

Database operations benefit from context for query timeouts and cancellation,  
preventing long-running queries from blocking application resources.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

type Database struct {
    connected bool
}

func (db *Database) QueryWithContext(ctx context.Context, query string) ([]string, error) {
    fmt.Printf("Executing query: %s\n", query)
    
    // Simulate database query time
    queryDuration := 2 * time.Second
    if query == "SELECT * FROM large_table" {
        queryDuration = 5 * time.Second
    }
    
    select {
    case <-time.After(queryDuration):
        results := []string{"row1", "row2", "row3"}
        fmt.Printf("Query completed: %d rows\n", len(results))
        return results, nil
    case <-ctx.Done():
        fmt.Printf("Query cancelled: %v\n", ctx.Err())
        return nil, ctx.Err()
    }
}

func (db *Database) TransactionWithContext(ctx context.Context) error {
    fmt.Println("Starting transaction")
    
    queries := []string{
        "INSERT INTO users VALUES (...)",
        "UPDATE accounts SET balance = ...",
        "INSERT INTO audit_log VALUES (...)",
    }
    
    for i, query := range queries {
        select {
        case <-ctx.Done():
            fmt.Printf("Transaction cancelled at step %d: %v\n", i+1, ctx.Err())
            return ctx.Err()
        default:
            fmt.Printf("Executing transaction step %d\n", i+1)
            time.Sleep(500 * time.Millisecond)
        }
    }
    
    fmt.Println("Transaction committed successfully")
    return nil
}

func main() {
    db := &Database{connected: true}
    
    // Query with timeout
    fmt.Println("=== Query with timeout ===")
    ctx1, cancel1 := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel1()
    
    results, err := db.QueryWithContext(ctx1, "SELECT * FROM users")
    if err != nil {
        fmt.Printf("Query error: %v\n", err)
    } else {
        fmt.Printf("Query results: %v\n", results)
    }
    
    // Long query that will timeout
    fmt.Println("\n=== Long query with timeout ===")
    ctx2, cancel2 := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel2()
    
    _, err = db.QueryWithContext(ctx2, "SELECT * FROM large_table")
    if err != nil {
        fmt.Printf("Long query error: %v\n", err)
    }
    
    // Transaction with cancellation
    fmt.Println("\n=== Transaction with cancellation ===")
    ctx3, cancel3 := context.WithCancel(context.Background())
    
    go func() {
        time.Sleep(1200 * time.Millisecond)
        fmt.Println("Cancelling transaction...")
        cancel3()
    }()
    
    err = db.TransactionWithContext(ctx3)
    if err != nil {
        fmt.Printf("Transaction error: %v\n", err)
    }
}
```

Database contexts enable query timeouts, transaction cancellation, and  
connection pool management. Most database drivers support context-aware  
operations for robust data access patterns.  

## File operations with context

File operations can be enhanced with context for timeouts and cancellation,  
especially useful for network file systems or large file processing.  

```go
package main

import (
    "context"
    "fmt"
    "io"
    "os"
    "strings"
    "time"
)

func readFileWithContext(ctx context.Context, filename string) ([]byte, error) {
    fmt.Printf("Reading file: %s\n", filename)
    
    // Check for cancellation before starting
    select {
    case <-ctx.Done():
        return nil, ctx.Err()
    default:
    }
    
    // Simulate file opening delay
    select {
    case <-time.After(100 * time.Millisecond):
    case <-ctx.Done():
        return nil, fmt.Errorf("file open cancelled: %w", ctx.Err())
    }
    
    // Create a temporary file for demonstration
    content := "Hello there from file content!\nLine 2\nLine 3\n"
    file := strings.NewReader(content)
    
    // Read in chunks with context checking
    var result []byte
    buffer := make([]byte, 10)
    
    for {
        select {
        case <-ctx.Done():
            return nil, fmt.Errorf("file read cancelled: %w", ctx.Err())
        default:
        }
        
        n, err := file.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            return nil, err
        }
        
        result = append(result, buffer[:n]...)
        
        // Simulate slow reading
        time.Sleep(50 * time.Millisecond)
    }
    
    fmt.Printf("File read completed: %d bytes\n", len(result))
    return result, nil
}

func writeFileWithContext(ctx context.Context, filename string, data []byte) error {
    fmt.Printf("Writing file: %s (%d bytes)\n", filename, len(data))
    
    // Create context with deadline for write operation
    writeCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()
    
    // Simulate writing in chunks
    chunkSize := 5
    for i := 0; i < len(data); i += chunkSize {
        select {
        case <-writeCtx.Done():
            return fmt.Errorf("write cancelled at byte %d: %w", i, writeCtx.Err())
        default:
        }
        
        end := i + chunkSize
        if end > len(data) {
            end = len(data)
        }
        
        // Simulate chunk write delay
        time.Sleep(100 * time.Millisecond)
        fmt.Printf("Written chunk: bytes %d-%d\n", i, end-1)
    }
    
    fmt.Printf("File write completed: %s\n", filename)
    return nil
}

func main() {
    // Read file with timeout
    fmt.Println("=== Reading file with timeout ===")
    ctx1, cancel1 := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel1()
    
    data, err := readFileWithContext(ctx1, "example.txt")
    if err != nil {
        fmt.Printf("Read error: %v\n", err)
    } else {
        fmt.Printf("File content: %s\n", string(data))
    }
    
    // Write file with cancellation
    fmt.Println("\n=== Writing file with cancellation ===")
    ctx2, cancel2 := context.WithCancel(context.Background())
    
    // Cancel after a short delay
    go func() {
        time.Sleep(250 * time.Millisecond)
        fmt.Println("Cancelling write operation...")
        cancel2()
    }()
    
    writeData := []byte("This is a longer file content that will be written in chunks.")
    err = writeFileWithContext(ctx2, "output.txt", writeData)
    if err != nil {
        fmt.Printf("Write error: %v\n", err)
    }
    
    // Clean file operation
    fmt.Println("\n=== Successful file operation ===")
    ctx3, cancel3 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel3()
    
    data, err = readFileWithContext(ctx3, "quick.txt")
    if err != nil {
        fmt.Printf("Read error: %v\n", err)
    } else {
        fmt.Printf("Quick read successful: %d bytes\n", len(data))
    }
}
```

File operations with context enable cancellable I/O operations, timeout  
handling for network filesystems, and graceful interruption of large  
file processing tasks.  

## Worker pool with context

Worker pools benefit from context for graceful shutdown, load balancing,  
and coordinated cancellation of background tasks.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

type Job struct {
    ID       int
    Data     string
    Duration time.Duration
}

type WorkerPool struct {
    workerCount int
    jobQueue    chan Job
    wg          sync.WaitGroup
}

func NewWorkerPool(workerCount int, queueSize int) *WorkerPool {
    return &WorkerPool{
        workerCount: workerCount,
        jobQueue:    make(chan Job, queueSize),
    }
}

func (wp *WorkerPool) Start(ctx context.Context) {
    fmt.Printf("Starting worker pool with %d workers\n", wp.workerCount)
    
    for i := 1; i <= wp.workerCount; i++ {
        wp.wg.Add(1)
        go wp.worker(ctx, i)
    }
}

func (wp *WorkerPool) worker(ctx context.Context, workerID int) {
    defer wp.wg.Done()
    fmt.Printf("Worker %d started\n", workerID)
    
    for {
        select {
        case job, ok := <-wp.jobQueue:
            if !ok {
                fmt.Printf("Worker %d: job queue closed\n", workerID)
                return
            }
            
            fmt.Printf("Worker %d: processing job %d\n", workerID, job.ID)
            
            // Process job with context awareness
            jobCtx, cancel := context.WithTimeout(ctx, job.Duration+1*time.Second)
            wp.processJob(jobCtx, workerID, job)
            cancel()
            
        case <-ctx.Done():
            fmt.Printf("Worker %d: shutting down due to context cancellation\n", workerID)
            return
        }
    }
}

func (wp *WorkerPool) processJob(ctx context.Context, workerID int, job Job) {
    select {
    case <-time.After(job.Duration):
        fmt.Printf("Worker %d: completed job %d (%s)\n", workerID, job.ID, job.Data)
    case <-ctx.Done():
        fmt.Printf("Worker %d: job %d cancelled (%v)\n", workerID, job.ID, ctx.Err())
    }
}

func (wp *WorkerPool) Submit(job Job) {
    select {
    case wp.jobQueue <- job:
        fmt.Printf("Job %d submitted\n", job.ID)
    default:
        fmt.Printf("Job %d dropped - queue full\n", job.ID)
    }
}

func (wp *WorkerPool) Shutdown() {
    close(wp.jobQueue)
    wp.wg.Wait()
    fmt.Println("All workers shut down")
}

func main() {
    // Create worker pool
    pool := NewWorkerPool(3, 10)
    
    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()
    
    // Start worker pool
    pool.Start(ctx)
    
    // Submit various jobs
    jobs := []Job{
        {ID: 1, Data: "quick task", Duration: 500 * time.Millisecond},
        {ID: 2, Data: "medium task", Duration: 1 * time.Second},
        {ID: 3, Data: "slow task", Duration: 2 * time.Second},
        {ID: 4, Data: "very slow task", Duration: 4 * time.Second},
        {ID: 5, Data: "another quick task", Duration: 300 * time.Millisecond},
    }
    
    for _, job := range jobs {
        pool.Submit(job)
        time.Sleep(100 * time.Millisecond) // Stagger submissions
    }
    
    // Let workers process for a while
    time.Sleep(3 * time.Second)
    
    // Shutdown worker pool
    fmt.Println("Initiating worker pool shutdown...")
    pool.Shutdown()
}
```

Worker pools with context enable graceful shutdown, job cancellation, and  
coordinated resource cleanup. Context propagation ensures that all workers  
respond appropriately to cancellation signals.  

## Pipeline processing with context

Context enables cancellable pipeline stages, allowing complex data processing  
workflows to be interrupted gracefully at any stage.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

func generateNumbers(ctx context.Context, max int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 1; i <= max; i++ {
            select {
            case out <- i:
                time.Sleep(100 * time.Millisecond)
            case <-ctx.Done():
                fmt.Printf("Number generator cancelled at %d\n", i)
                return
            }
        }
        fmt.Println("Number generation completed")
    }()
    return out
}

func processStage(ctx context.Context, stageName string, input <-chan int, processor func(int) int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for {
            select {
            case num, ok := <-input:
                if !ok {
                    fmt.Printf("Stage %s: input channel closed\n", stageName)
                    return
                }
                
                result := processor(num)
                fmt.Printf("Stage %s: %d -> %d\n", stageName, num, result)
                
                select {
                case out <- result:
                case <-ctx.Done():
                    fmt.Printf("Stage %s cancelled during output\n", stageName)
                    return
                }
                
            case <-ctx.Done():
                fmt.Printf("Stage %s cancelled\n", stageName)
                return
            }
        }
    }()
    return out
}

func collectResults(ctx context.Context, input <-chan int) []int {
    var results []int
    for {
        select {
        case num, ok := <-input:
            if !ok {
                fmt.Println("Result collection completed")
                return results
            }
            results = append(results, num)
            fmt.Printf("Collected result: %d\n", num)
            
        case <-ctx.Done():
            fmt.Printf("Result collection cancelled, got %d results\n", len(results))
            return results
        }
    }
}

func main() {
    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
    defer cancel()
    
    // Set up processing pipeline
    numbers := generateNumbers(ctx, 10)
    
    // Stage 1: Square numbers
    squared := processStage(ctx, "square", numbers, func(n int) int {
        time.Sleep(50 * time.Millisecond) // Simulate processing time
        return n * n
    })
    
    // Stage 2: Add 10
    added := processStage(ctx, "add10", squared, func(n int) int {
        time.Sleep(50 * time.Millisecond) // Simulate processing time
        return n + 10
    })
    
    // Stage 3: Filter even numbers
    filtered := processStage(ctx, "filter-even", added, func(n int) int {
        time.Sleep(50 * time.Millisecond) // Simulate processing time
        if n%2 == 0 {
            return n
        }
        return -1 // Mark for filtering
    })
    
    // Collect results
    results := collectResults(ctx, filtered)
    
    // Filter out marked values
    var finalResults []int
    for _, r := range results {
        if r != -1 {
            finalResults = append(finalResults, r)
        }
    }
    
    fmt.Printf("Final results: %v\n", finalResults)
    fmt.Printf("Pipeline processed %d numbers\n", len(finalResults))
}
```

Pipeline processing with context allows complex data transformations to be  
cancelled at any stage, preventing resource waste and enabling responsive  
applications that can adapt to changing conditions.  

## Service discovery with context

Service discovery operations benefit from context for timeout handling,  
retries, and graceful degradation when services are unavailable.  

```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

type Service struct {
    Name     string
    Address  string
    Healthy  bool
    LastSeen time.Time
}

type ServiceRegistry struct {
    services map[string][]*Service
    mu       sync.RWMutex
}

func NewServiceRegistry() *ServiceRegistry {
    return &ServiceRegistry{
        services: make(map[string][]*Service),
    }
}

func (sr *ServiceRegistry) Register(service *Service) {
    sr.mu.Lock()
    defer sr.mu.Unlock()
    
    sr.services[service.Name] = append(sr.services[service.Name], service)
    fmt.Printf("Registered service: %s at %s\n", service.Name, service.Address)
}

func (sr *ServiceRegistry) DiscoverWithContext(ctx context.Context, serviceName string) ([]*Service, error) {
    fmt.Printf("Discovering services for: %s\n", serviceName)
    
    // Simulate discovery latency
    discoveryTime := time.Duration(rand.Intn(1000)) * time.Millisecond
    
    select {
    case <-time.After(discoveryTime):
        sr.mu.RLock()
        services := sr.services[serviceName]
        sr.mu.RUnlock()
        
        // Filter healthy services
        var healthyServices []*Service
        for _, service := range services {
            if sr.isHealthy(ctx, service) {
                healthyServices = append(healthyServices, service)
            }
        }
        
        fmt.Printf("Discovered %d healthy services for %s\n", len(healthyServices), serviceName)
        return healthyServices, nil
        
    case <-ctx.Done():
        fmt.Printf("Service discovery cancelled for %s: %v\n", serviceName, ctx.Err())
        return nil, ctx.Err()
    }
}

func (sr *ServiceRegistry) isHealthy(ctx context.Context, service *Service) bool {
    // Simulate health check with context
    healthCheckCtx, cancel := context.WithTimeout(ctx, 200*time.Millisecond)
    defer cancel()
    
    select {
    case <-time.After(time.Duration(rand.Intn(300)) * time.Millisecond):
        // Random health status
        healthy := rand.Float32() > 0.3
        if healthy {
            service.LastSeen = time.Now()
        }
        return healthy
        
    case <-healthCheckCtx.Done():
        fmt.Printf("Health check timeout for %s\n", service.Address)
        return false
    }
}

func (sr *ServiceRegistry) DiscoverWithRetry(ctx context.Context, serviceName string, maxRetries int) ([]*Service, error) {
    var lastErr error
    
    for attempt := 1; attempt <= maxRetries; attempt++ {
        // Create context for this attempt
        attemptCtx, cancel := context.WithTimeout(ctx, 1*time.Second)
        
        services, err := sr.DiscoverWithContext(attemptCtx, serviceName)
        cancel()
        
        if err == nil && len(services) > 0 {
            fmt.Printf("Discovery successful on attempt %d\n", attempt)
            return services, nil
        }
        
        lastErr = err
        fmt.Printf("Discovery attempt %d failed: %v\n", attempt, err)
        
        if attempt < maxRetries {
            // Wait before retry, but respect context cancellation
            select {
            case <-time.After(time.Duration(attempt*500) * time.Millisecond):
            case <-ctx.Done():
                return nil, ctx.Err()
            }
        }
    }
    
    return nil, fmt.Errorf("discovery failed after %d attempts: %w", maxRetries, lastErr)
}

func main() {
    registry := NewServiceRegistry()
    
    // Register some services
    services := []*Service{
        {Name: "user-service", Address: "10.0.0.1:8080", Healthy: true},
        {Name: "user-service", Address: "10.0.0.2:8080", Healthy: true},
        {Name: "order-service", Address: "10.0.0.3:8080", Healthy: true},
        {Name: "payment-service", Address: "10.0.0.4:8080", Healthy: true},
    }
    
    for _, service := range services {
        registry.Register(service)
    }
    
    // Discover services with timeout
    fmt.Println("=== Simple discovery with timeout ===")
    ctx1, cancel1 := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel1()
    
    userServices, err := registry.DiscoverWithContext(ctx1, "user-service")
    if err != nil {
        fmt.Printf("Discovery error: %v\n", err)
    } else {
        fmt.Printf("Found user services: %d\n", len(userServices))
    }
    
    // Discover with retry
    fmt.Println("\n=== Discovery with retry ===")
    ctx2, cancel2 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel2()
    
    orderServices, err := registry.DiscoverWithRetry(ctx2, "order-service", 3)
    if err != nil {
        fmt.Printf("Discovery with retry error: %v\n", err)
    } else {
        fmt.Printf("Found order services: %d\n", len(orderServices))
    }
    
    // Cancelled discovery
    fmt.Println("\n=== Cancelled discovery ===")
    ctx3, cancel3 := context.WithCancel(context.Background())
    
    // Cancel immediately
    cancel3()
    
    _, err = registry.DiscoverWithContext(ctx3, "payment-service")
    if err != nil {
        fmt.Printf("Expected cancellation error: %v\n", err)
    }
}
```

Service discovery with context enables resilient distributed systems by  
handling timeouts, retries, and cancellation gracefully. Context  
propagation ensures consistent behavior across service boundaries.  

## Cache operations with context

Caching operations can use context for timeout control, invalidation  
coordination, and background refresh operations.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

type CacheItem struct {
    Value     interface{}
    ExpiresAt time.Time
    Hits      int
}

type Cache struct {
    items map[string]*CacheItem
    mu    sync.RWMutex
}

func NewCache() *Cache {
    return &Cache{
        items: make(map[string]*CacheItem),
    }
}

func (c *Cache) GetWithContext(ctx context.Context, key string) (interface{}, bool) {
    // Simulate cache lookup latency
    select {
    case <-time.After(10 * time.Millisecond):
    case <-ctx.Done():
        fmt.Printf("Cache get cancelled for key: %s\n", key)
        return nil, false
    }
    
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    item, exists := c.items[key]
    if !exists {
        fmt.Printf("Cache miss for key: %s\n", key)
        return nil, false
    }
    
    if time.Now().After(item.ExpiresAt) {
        fmt.Printf("Cache expired for key: %s\n", key)
        return nil, false
    }
    
    item.Hits++
    fmt.Printf("Cache hit for key: %s (hits: %d)\n", key, item.Hits)
    return item.Value, true
}

func (c *Cache) SetWithContext(ctx context.Context, key string, value interface{}, ttl time.Duration) error {
    select {
    case <-time.After(5 * time.Millisecond):
    case <-ctx.Done():
        return fmt.Errorf("cache set cancelled for key %s: %w", key, ctx.Err())
    }
    
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.items[key] = &CacheItem{
        Value:     value,
        ExpiresAt: time.Now().Add(ttl),
        Hits:      0,
    }
    
    fmt.Printf("Cache set for key: %s (TTL: %v)\n", key, ttl)
    return nil
}

func (c *Cache) GetOrFetchWithContext(ctx context.Context, key string, fetcher func(context.Context) (interface{}, error)) (interface{}, error) {
    // Try to get from cache first
    if value, found := c.GetWithContext(ctx, key); found {
        return value, nil
    }
    
    fmt.Printf("Cache miss, fetching for key: %s\n", key)
    
    // Create timeout context for fetching
    fetchCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()
    
    // Fetch the value
    value, err := fetcher(fetchCtx)
    if err != nil {
        return nil, fmt.Errorf("failed to fetch value for key %s: %w", key, err)
    }
    
    // Cache the result
    if err := c.SetWithContext(ctx, key, value, 5*time.Minute); err != nil {
        fmt.Printf("Failed to cache value for key %s: %v\n", key, err)
    }
    
    return value, nil
}

func (c *Cache) RefreshInBackground(ctx context.Context, key string, fetcher func(context.Context) (interface{}, error)) {
    go func() {
        refreshCtx, cancel := context.WithTimeout(ctx, 3*time.Second)
        defer cancel()
        
        fmt.Printf("Background refresh started for key: %s\n", key)
        
        value, err := fetcher(refreshCtx)
        if err != nil {
            fmt.Printf("Background refresh failed for key %s: %v\n", key, err)
            return
        }
        
        if err := c.SetWithContext(refreshCtx, key, value, 5*time.Minute); err != nil {
            fmt.Printf("Background refresh cache set failed for key %s: %v\n", key, err)
            return
        }
        
        fmt.Printf("Background refresh completed for key: %s\n", key)
    }()
}

func (c *Cache) CleanupExpired(ctx context.Context) {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            c.mu.Lock()
            now := time.Now()
            expired := 0
            
            for key, item := range c.items {
                if now.After(item.ExpiresAt) {
                    delete(c.items, key)
                    expired++
                }
            }
            c.mu.Unlock()
            
            if expired > 0 {
                fmt.Printf("Cleaned up %d expired cache items\n", expired)
            }
            
        case <-ctx.Done():
            fmt.Println("Cache cleanup stopped")
            return
        }
    }
}

// Simulated data fetcher
func fetchUserData(ctx context.Context, userID string) (interface{}, error) {
    // Simulate network call
    select {
    case <-time.After(500 * time.Millisecond):
        return map[string]interface{}{
            "id":   userID,
            "name": "User " + userID,
            "age":  25,
        }, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

func main() {
    cache := NewCache()
    
    // Start background cleanup
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    go cache.CleanupExpired(ctx)
    
    // Test cache operations
    fmt.Println("=== Cache operations with context ===")
    
    // Fetch and cache user data
    userData1, err := cache.GetOrFetchWithContext(ctx, "user:123", func(ctx context.Context) (interface{}, error) {
        return fetchUserData(ctx, "123")
    })
    if err != nil {
        fmt.Printf("Error fetching user data: %v\n", err)
    } else {
        fmt.Printf("User data 1: %v\n", userData1)
    }
    
    // Second fetch should hit cache
    userData2, err := cache.GetOrFetchWithContext(ctx, "user:123", func(ctx context.Context) (interface{}, error) {
        return fetchUserData(ctx, "123")
    })
    if err != nil {
        fmt.Printf("Error fetching user data: %v\n", err)
    } else {
        fmt.Printf("User data 2: %v\n", userData2)
    }
    
    // Background refresh
    fmt.Println("\n=== Background refresh ===")
    cache.RefreshInBackground(ctx, "user:456", func(ctx context.Context) (interface{}, error) {
        return fetchUserData(ctx, "456")
    })
    
    // Wait for background operations
    time.Sleep(1 * time.Second)
    
    // Test with cancelled context
    fmt.Println("\n=== Cancelled operations ===")
    cancelledCtx, cancelFunc := context.WithCancel(context.Background())
    cancelFunc() // Cancel immediately
    
    _, err = cache.GetOrFetchWithContext(cancelledCtx, "user:789", func(ctx context.Context) (interface{}, error) {
        return fetchUserData(ctx, "789")
    })
    if err != nil {
        fmt.Printf("Expected cancellation error: %v\n", err)
    }
}
```

Cache operations with context enable timeout-aware data access, background  
refresh coordination, and graceful degradation when caching operations fail  
or are cancelled.  

