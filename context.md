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

## Distributed tracing with context

Context values are commonly used for distributed tracing, carrying trace  
and span IDs across service boundaries for observability.  

```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "time"
)

type TraceID string
type SpanID string

type Span struct {
    TraceID   TraceID
    SpanID    SpanID
    ParentID  SpanID
    Operation string
    StartTime time.Time
    EndTime   time.Time
    Tags      map[string]string
}

type Tracer struct {
    spans []Span
}

func NewTracer() *Tracer {
    return &Tracer{
        spans: make([]Span, 0),
    }
}

func (t *Tracer) StartSpan(ctx context.Context, operation string) (context.Context, *Span) {
    traceID := getTraceID(ctx)
    parentSpanID := getSpanID(ctx)
    
    if traceID == "" {
        traceID = TraceID(fmt.Sprintf("trace-%d", rand.Int63()))
    }
    
    span := &Span{
        TraceID:   traceID,
        SpanID:    SpanID(fmt.Sprintf("span-%d", rand.Int63())),
        ParentID:  parentSpanID,
        Operation: operation,
        StartTime: time.Now(),
        Tags:      make(map[string]string),
    }
    
    fmt.Printf("Started span: %s (trace: %s, parent: %s)\n", 
              span.SpanID, span.TraceID, span.ParentID)
    
    // Add span to context
    ctx = context.WithValue(ctx, "traceID", span.TraceID)
    ctx = context.WithValue(ctx, "spanID", span.SpanID)
    
    return ctx, span
}

func (t *Tracer) FinishSpan(span *Span) {
    span.EndTime = time.Now()
    duration := span.EndTime.Sub(span.StartTime)
    
    fmt.Printf("Finished span: %s (duration: %v)\n", span.SpanID, duration)
    t.spans = append(t.spans, *span)
}

func (t *Tracer) PrintTrace(traceID TraceID) {
    fmt.Printf("\n=== Trace: %s ===\n", traceID)
    for _, span := range t.spans {
        if span.TraceID == traceID {
            duration := span.EndTime.Sub(span.StartTime)
            fmt.Printf("  Span: %s | Op: %s | Duration: %v | Parent: %s\n",
                      span.SpanID, span.Operation, duration, span.ParentID)
        }
    }
}

func getTraceID(ctx context.Context) TraceID {
    if traceID, ok := ctx.Value("traceID").(TraceID); ok {
        return traceID
    }
    return ""
}

func getSpanID(ctx context.Context) SpanID {
    if spanID, ok := ctx.Value("spanID").(SpanID); ok {
        return spanID
    }
    return ""
}

func authenticateUser(ctx context.Context, tracer *Tracer, userID string) error {
    ctx, span := tracer.StartSpan(ctx, "authenticate_user")
    defer tracer.FinishSpan(span)
    
    span.Tags["user.id"] = userID
    
    // Simulate authentication work
    time.Sleep(100 * time.Millisecond)
    
    if userID == "invalid" {
        span.Tags["error"] = "authentication_failed"
        return fmt.Errorf("authentication failed for user %s", userID)
    }
    
    span.Tags["result"] = "success"
    return nil
}

func fetchUserProfile(ctx context.Context, tracer *Tracer, userID string) (map[string]interface{}, error) {
    ctx, span := tracer.StartSpan(ctx, "fetch_user_profile")
    defer tracer.FinishSpan(span)
    
    span.Tags["user.id"] = userID
    span.Tags["service"] = "user-service"
    
    // Simulate database call with nested span
    profile, err := queryDatabase(ctx, tracer, "SELECT * FROM users WHERE id = ?", userID)
    if err != nil {
        span.Tags["error"] = err.Error()
        return nil, err
    }
    
    span.Tags["result"] = "success"
    return profile, nil
}

func queryDatabase(ctx context.Context, tracer *Tracer, query string, args ...interface{}) (map[string]interface{}, error) {
    ctx, span := tracer.StartSpan(ctx, "database_query")
    defer tracer.FinishSpan(span)
    
    span.Tags["db.statement"] = query
    span.Tags["db.type"] = "postgresql"
    
    // Check for context cancellation
    select {
    case <-time.After(50 * time.Millisecond):
        result := map[string]interface{}{
            "id":   args[0],
            "name": "User " + args[0].(string),
            "age":  25,
        }
        span.Tags["db.rows_affected"] = "1"
        return result, nil
    case <-ctx.Done():
        span.Tags["error"] = "cancelled"
        return nil, ctx.Err()
    }
}

func processRequest(ctx context.Context, tracer *Tracer, userID string) error {
    ctx, span := tracer.StartSpan(ctx, "process_request")
    defer tracer.FinishSpan(span)
    
    span.Tags["request.user_id"] = userID
    
    // Step 1: Authenticate
    if err := authenticateUser(ctx, tracer, userID); err != nil {
        span.Tags["error"] = "authentication_failed"
        return err
    }
    
    // Step 2: Fetch profile
    profile, err := fetchUserProfile(ctx, tracer, userID)
    if err != nil {
        span.Tags["error"] = "profile_fetch_failed"
        return err
    }
    
    span.Tags["profile.name"] = profile["name"].(string)
    span.Tags["result"] = "success"
    
    fmt.Printf("Request processed successfully for user: %s\n", profile["name"])
    return nil
}

func main() {
    tracer := NewTracer()
    
    // Create root context
    ctx := context.Background()
    
    // Process multiple requests
    requests := []string{"user123", "user456", "invalid"}
    
    for _, userID := range requests {
        fmt.Printf("\n--- Processing request for user: %s ---\n", userID)
        
        // Create context with timeout for each request
        requestCtx, cancel := context.WithTimeout(ctx, 2*time.Second)
        
        err := processRequest(requestCtx, tracer, userID)
        if err != nil {
            fmt.Printf("Request failed: %v\n", err)
        }
        
        cancel()
        
        // Print trace for this request
        if traceID := getTraceID(requestCtx); traceID != "" {
            tracer.PrintTrace(traceID)
        }
    }
    
    fmt.Printf("\nTotal spans recorded: %d\n", len(tracer.spans))
}
```

Distributed tracing with context enables request tracking across service  
boundaries, providing visibility into complex distributed operations and  
helping identify performance bottlenecks and errors.  

## Context middleware patterns

Middleware functions can enhance requests with context values, timeouts,  
and cancellation logic in a composable manner.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "time"
)

type Middleware func(http.Handler) http.Handler

// Request ID middleware
func RequestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestID := fmt.Sprintf("req_%d", time.Now().UnixNano())
        ctx := context.WithValue(r.Context(), "requestID", requestID)
        
        w.Header().Set("X-Request-ID", requestID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// Timeout middleware
func TimeoutMiddleware(timeout time.Duration) Middleware {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            ctx, cancel := context.WithTimeout(r.Context(), timeout)
            defer cancel()
            
            // Create a channel to signal completion
            done := make(chan struct{})
            
            go func() {
                defer close(done)
                next.ServeHTTP(w, r.WithContext(ctx))
            }()
            
            select {
            case <-done:
                // Request completed normally
            case <-ctx.Done():
                // Request timed out
                w.WriteHeader(http.StatusRequestTimeout)
                fmt.Fprintf(w, "Request timeout after %v", timeout)
                log.Printf("Request timeout: %s %s", r.Method, r.URL.Path)
            }
        })
    }
}

// Logging middleware
func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        requestID := r.Context().Value("requestID")
        
        log.Printf("Started %s %s (ID: %v)", r.Method, r.URL.Path, requestID)
        
        // Wrap ResponseWriter to capture status code
        wrapped := &responseWriter{ResponseWriter: w, statusCode: 200}
        next.ServeHTTP(wrapped, r)
        
        duration := time.Since(start)
        log.Printf("Completed %s %s (ID: %v) - Status: %d, Duration: %v", 
                  r.Method, r.URL.Path, requestID, wrapped.statusCode, duration)
    })
}

type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}

// Authentication middleware
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            w.WriteHeader(http.StatusUnauthorized)
            fmt.Fprintf(w, "Missing authorization header")
            return
        }
        
        // Simulate token validation
        userID := validateToken(r.Context(), token)
        if userID == "" {
            w.WriteHeader(http.StatusUnauthorized)
            fmt.Fprintf(w, "Invalid token")
            return
        }
        
        // Add user to context
        ctx := context.WithValue(r.Context(), "userID", userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

func validateToken(ctx context.Context, token string) string {
    // Simulate token validation with context timeout
    select {
    case <-time.After(100 * time.Millisecond):
        if token == "Bearer valid-token" {
            return "user123"
        }
        return ""
    case <-ctx.Done():
        return ""
    }
}

// Rate limiting middleware
func RateLimitMiddleware(requestsPerSecond int) Middleware {
    ticker := time.NewTicker(time.Second / time.Duration(requestsPerSecond))
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            select {
            case <-ticker.C:
                next.ServeHTTP(w, r)
            case <-r.Context().Done():
                w.WriteHeader(http.StatusRequestTimeout)
                fmt.Fprintf(w, "Request cancelled")
            default:
                w.WriteHeader(http.StatusTooManyRequests)
                fmt.Fprintf(w, "Rate limit exceeded")
            }
        })
    }
}

// Chain multiple middlewares
func ChainMiddleware(middlewares ...Middleware) Middleware {
    return func(next http.Handler) http.Handler {
        for i := len(middlewares) - 1; i >= 0; i-- {
            next = middlewares[i](next)
        }
        return next
    }
}

// Sample handlers
func healthHandler(w http.ResponseWriter, r *http.Request) {
    requestID := r.Context().Value("requestID")
    fmt.Fprintf(w, "OK (Request ID: %v)", requestID)
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    requestID := ctx.Value("requestID")
    userID := ctx.Value("userID")
    
    // Simulate slow operation
    select {
    case <-time.After(2 * time.Second):
        fmt.Fprintf(w, "Slow operation completed (Request ID: %v, User: %v)", requestID, userID)
    case <-ctx.Done():
        // This won't be reached due to timeout middleware handling
        fmt.Fprintf(w, "Operation cancelled")
    }
}

func userHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    userID := ctx.Value("userID")
    requestID := ctx.Value("requestID")
    
    if userID == nil {
        w.WriteHeader(http.StatusUnauthorized)
        fmt.Fprintf(w, "User not authenticated")
        return
    }
    
    fmt.Fprintf(w, "User profile for %v (Request ID: %v)", userID, requestID)
}

func main() {
    // Create middleware chain
    middleware := ChainMiddleware(
        RequestIDMiddleware,
        LoggingMiddleware,
        TimeoutMiddleware(1*time.Second),
        RateLimitMiddleware(2), // 2 requests per second
    )
    
    // Protected routes with authentication
    protectedMiddleware := ChainMiddleware(
        RequestIDMiddleware,
        LoggingMiddleware,
        AuthMiddleware,
        TimeoutMiddleware(3*time.Second),
    )
    
    // Set up routes
    http.Handle("/health", middleware(http.HandlerFunc(healthHandler)))
    http.Handle("/slow", middleware(http.HandlerFunc(slowHandler)))
    http.Handle("/user", protectedMiddleware(http.HandlerFunc(userHandler)))
    
    fmt.Println("Server starting on :8080")
    fmt.Println("Try these endpoints:")
    fmt.Println("  curl http://localhost:8080/health")
    fmt.Println("  curl http://localhost:8080/slow")
    fmt.Println("  curl -H 'Authorization: Bearer valid-token' http://localhost:8080/user")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

Context middleware patterns enable composable request processing with  
consistent timeout handling, logging, authentication, and cancellation  
behavior across all HTTP endpoints.  

## Testing with context

Context makes testing asynchronous and long-running operations more  
deterministic by enabling controlled cancellation and timeouts.  

```go
package main

import (
    "context"
    "fmt"
    "testing"
    "time"
)

// Service to test
type UserService struct {
    delay time.Duration
}

func (s *UserService) GetUser(ctx context.Context, userID string) (*User, error) {
    select {
    case <-time.After(s.delay):
        if userID == "notfound" {
            return nil, fmt.Errorf("user not found: %s", userID)
        }
        return &User{ID: userID, Name: "User " + userID}, nil
    case <-ctx.Done():
        return nil, ctx.Err()
    }
}

func (s *UserService) CreateUser(ctx context.Context, user *User) error {
    select {
    case <-time.After(s.delay):
        if user.Name == "invalid" {
            return fmt.Errorf("invalid user name")
        }
        fmt.Printf("Created user: %s\n", user.Name)
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

type User struct {
    ID   string
    Name string
}

// Test helper functions
func TestUserService_GetUser_Success(t *testing.T) {
    service := &UserService{delay: 100 * time.Millisecond}
    ctx := context.Background()
    
    user, err := service.GetUser(ctx, "123")
    if err != nil {
        t.Fatalf("Expected no error, got %v", err)
    }
    
    if user.ID != "123" {
        t.Errorf("Expected user ID '123', got '%s'", user.ID)
    }
    
    if user.Name != "User 123" {
        t.Errorf("Expected user name 'User 123', got '%s'", user.Name)
    }
}

func TestUserService_GetUser_NotFound(t *testing.T) {
    service := &UserService{delay: 50 * time.Millisecond}
    ctx := context.Background()
    
    user, err := service.GetUser(ctx, "notfound")
    if err == nil {
        t.Fatal("Expected error for non-existent user")
    }
    
    if user != nil {
        t.Error("Expected nil user for non-existent user")
    }
}

func TestUserService_GetUser_Timeout(t *testing.T) {
    service := &UserService{delay: 500 * time.Millisecond}
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()
    
    user, err := service.GetUser(ctx, "123")
    if err != context.DeadlineExceeded {
        t.Fatalf("Expected context.DeadlineExceeded, got %v", err)
    }
    
    if user != nil {
        t.Error("Expected nil user on timeout")
    }
}

func TestUserService_GetUser_Cancellation(t *testing.T) {
    service := &UserService{delay: 500 * time.Millisecond}
    ctx, cancel := context.WithCancel(context.Background())
    
    // Cancel after 100ms
    go func() {
        time.Sleep(100 * time.Millisecond)
        cancel()
    }()
    
    user, err := service.GetUser(ctx, "123")
    if err != context.Canceled {
        t.Fatalf("Expected context.Canceled, got %v", err)
    }
    
    if user != nil {
        t.Error("Expected nil user on cancellation")
    }
}

func TestUserService_CreateUser_WithValues(t *testing.T) {
    service := &UserService{delay: 50 * time.Millisecond}
    
    // Create context with test values
    ctx := context.WithValue(context.Background(), "testID", "test123")
    ctx = context.WithValue(ctx, "requestID", "req456")
    
    user := &User{ID: "789", Name: "Test User"}
    err := service.CreateUser(ctx, user)
    if err != nil {
        t.Fatalf("Expected no error, got %v", err)
    }
    
    // Verify context values are accessible
    testID := ctx.Value("testID")
    if testID != "test123" {
        t.Errorf("Expected testID 'test123', got '%v'", testID)
    }
}

// Benchmark with context
func BenchmarkUserService_GetUser(b *testing.B) {
    service := &UserService{delay: 10 * time.Millisecond}
    ctx := context.Background()
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, err := service.GetUser(ctx, "benchmark")
        if err != nil {
            b.Fatalf("Unexpected error: %v", err)
        }
    }
}

func BenchmarkUserService_GetUser_WithTimeout(b *testing.B) {
    service := &UserService{delay: 5 * time.Millisecond}
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
        _, err := service.GetUser(ctx, "benchmark")
        cancel()
        
        if err != nil {
            b.Fatalf("Unexpected error: %v", err)
        }
    }
}

// Table-driven tests with context
func TestUserService_GetUser_TableDriven(t *testing.T) {
    tests := []struct {
        name        string
        userID      string
        timeout     time.Duration
        serviceDelay time.Duration
        expectError bool
        errorType   error
    }{
        {
            name:        "success",
            userID:      "123",
            timeout:     200 * time.Millisecond,
            serviceDelay: 50 * time.Millisecond,
            expectError: false,
        },
        {
            name:        "timeout",
            userID:      "123",
            timeout:     50 * time.Millisecond,
            serviceDelay: 200 * time.Millisecond,
            expectError: true,
            errorType:   context.DeadlineExceeded,
        },
        {
            name:        "not found",
            userID:      "notfound",
            timeout:     200 * time.Millisecond,
            serviceDelay: 50 * time.Millisecond,
            expectError: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            service := &UserService{delay: tt.serviceDelay}
            ctx, cancel := context.WithTimeout(context.Background(), tt.timeout)
            defer cancel()
            
            user, err := service.GetUser(ctx, tt.userID)
            
            if tt.expectError {
                if err == nil {
                    t.Error("Expected error but got none")
                }
                if tt.errorType != nil && err != tt.errorType {
                    t.Errorf("Expected error %v, got %v", tt.errorType, err)
                }
            } else {
                if err != nil {
                    t.Errorf("Expected no error but got %v", err)
                }
                if user == nil {
                    t.Error("Expected user but got nil")
                }
            }
        })
    }
}

// Mock implementation for testing
type MockUserService struct {
    getUserFunc    func(context.Context, string) (*User, error)
    createUserFunc func(context.Context, *User) error
}

func (m *MockUserService) GetUser(ctx context.Context, userID string) (*User, error) {
    if m.getUserFunc != nil {
        return m.getUserFunc(ctx, userID)
    }
    return nil, fmt.Errorf("mock not implemented")
}

func (m *MockUserService) CreateUser(ctx context.Context, user *User) error {
    if m.createUserFunc != nil {
        return m.createUserFunc(ctx, user)
    }
    return fmt.Errorf("mock not implemented")
}

func TestWithMock(t *testing.T) {
    mock := &MockUserService{
        getUserFunc: func(ctx context.Context, userID string) (*User, error) {
            // Check context cancellation
            select {
            case <-ctx.Done():
                return nil, ctx.Err()
            default:
            }
            
            return &User{ID: userID, Name: "Mock User"}, nil
        },
    }
    
    ctx := context.Background()
    user, err := mock.GetUser(ctx, "test")
    if err != nil {
        t.Fatalf("Mock test failed: %v", err)
    }
    
    if user.Name != "Mock User" {
        t.Errorf("Expected 'Mock User', got '%s'", user.Name)
    }
}

func main() {
    // Run a simple test demonstration
    fmt.Println("Running context testing examples...")
    
    // Simulate test runner
    t := &testing.T{}
    
    fmt.Println("Running TestUserService_GetUser_Success...")
    TestUserService_GetUser_Success(t)
    
    fmt.Println("Running TestUserService_GetUser_Timeout...")
    TestUserService_GetUser_Timeout(t)
    
    fmt.Println("Running TestWithMock...")
    TestWithMock(t)
    
    fmt.Println("All tests completed!")
}
```

Testing with context enables deterministic testing of asynchronous  
operations, timeout scenarios, and cancellation behavior, making tests  
more reliable and comprehensive.  

## Context with channels

Combining context with channels creates powerful patterns for coordinated  
cancellation and synchronization in concurrent systems.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Producer with context and channels
func producer(ctx context.Context, name string, out chan<- int) {
    defer close(out)
    
    for i := 1; ; i++ {
        select {
        case out <- i:
            fmt.Printf("Producer %s: sent %d\n", name, i)
            time.Sleep(200 * time.Millisecond)
        case <-ctx.Done():
            fmt.Printf("Producer %s: stopped (%v)\n", name, ctx.Err())
            return
        }
    }
}

// Consumer with context
func consumer(ctx context.Context, name string, in <-chan int, results chan<- string) {
    defer close(results)
    
    for {
        select {
        case value, ok := <-in:
            if !ok {
                fmt.Printf("Consumer %s: input channel closed\n", name)
                return
            }
            
            result := fmt.Sprintf("%s processed %d", name, value)
            
            select {
            case results <- result:
                fmt.Printf("Consumer %s: processed %d\n", name, value)
            case <-ctx.Done():
                fmt.Printf("Consumer %s: cancelled while sending result\n", name)
                return
            }
            
        case <-ctx.Done():
            fmt.Printf("Consumer %s: cancelled (%v)\n", name, ctx.Err())
            return
        }
    }
}

// Fan-out pattern with context
func fanOut(ctx context.Context, input <-chan int, workerCount int) []<-chan string {
    outputs := make([]<-chan string, workerCount)
    
    for i := 0; i < workerCount; i++ {
        output := make(chan string, 10)
        outputs[i] = output
        
        go func(workerID int, out chan<- string) {
            defer close(out)
            
            for {
                select {
                case value, ok := <-input:
                    if !ok {
                        fmt.Printf("Worker %d: input closed\n", workerID)
                        return
                    }
                    
                    // Simulate processing
                    time.Sleep(100 * time.Millisecond)
                    result := fmt.Sprintf("worker-%d: %d^2 = %d", workerID, value, value*value)
                    
                    select {
                    case out <- result:
                    case <-ctx.Done():
                        fmt.Printf("Worker %d: cancelled\n", workerID)
                        return
                    }
                    
                case <-ctx.Done():
                    fmt.Printf("Worker %d: cancelled\n", workerID)
                    return
                }
            }
        }(i, output)
    }
    
    return outputs
}

// Fan-in pattern with context
func fanIn(ctx context.Context, inputs ...<-chan string) <-chan string {
    output := make(chan string)
    var wg sync.WaitGroup
    
    // Start a goroutine for each input channel
    wg.Add(len(inputs))
    for i, input := range inputs {
        go func(id int, in <-chan string) {
            defer wg.Done()
            
            for {
                select {
                case value, ok := <-in:
                    if !ok {
                        fmt.Printf("FanIn input %d: channel closed\n", id)
                        return
                    }
                    
                    select {
                    case output <- value:
                    case <-ctx.Done():
                        fmt.Printf("FanIn input %d: cancelled\n", id)
                        return
                    }
                    
                case <-ctx.Done():
                    fmt.Printf("FanIn input %d: cancelled\n", id)
                    return
                }
            }
        }(i, input)
    }
    
    // Close output when all inputs are done
    go func() {
        wg.Wait()
        close(output)
        fmt.Println("FanIn: all inputs completed")
    }()
    
    return output
}

// Select with context for multiple operations
func selectWithContext(ctx context.Context) {
    // Create multiple channels with different timing
    fastChan := make(chan string, 1)
    slowChan := make(chan string, 1)
    timeoutChan := make(chan string, 1)
    
    // Start producers
    go func() {
        time.Sleep(100 * time.Millisecond)
        fastChan <- "fast result"
    }()
    
    go func() {
        time.Sleep(500 * time.Millisecond)
        slowChan <- "slow result"
    }()
    
    go func() {
        time.Sleep(2 * time.Second)
        timeoutChan <- "timeout result"
    }()
    
    // Select with context timeout
    for i := 0; i < 3; i++ {
        select {
        case result := <-fastChan:
            fmt.Printf("Received: %s\n", result)
        case result := <-slowChan:
            fmt.Printf("Received: %s\n", result)
        case result := <-timeoutChan:
            fmt.Printf("Received: %s\n", result)
        case <-ctx.Done():
            fmt.Printf("Select cancelled: %v\n", ctx.Err())
            return
        }
    }
}

func main() {
    fmt.Println("=== Producer-Consumer with Context ===")
    
    ctx1, cancel1 := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel1()
    
    // Simple producer-consumer
    dataChannel := make(chan int, 5)
    resultChannel := make(chan string, 5)
    
    go producer(ctx1, "P1", dataChannel)
    go consumer(ctx1, "C1", dataChannel, resultChannel)
    
    // Collect results
    go func() {
        for result := range resultChannel {
            fmt.Printf("Result: %s\n", result)
        }
    }()
    
    time.Sleep(3 * time.Second)
    
    fmt.Println("\n=== Fan-Out/Fan-In Pattern ===")
    
    ctx2, cancel2 := context.WithTimeout(context.Background(), 4*time.Second)
    defer cancel2()
    
    // Create input channel
    numbers := make(chan int, 10)
    
    // Start number producer
    go func() {
        defer close(numbers)
        for i := 1; i <= 10; i++ {
            select {
            case numbers <- i:
                time.Sleep(50 * time.Millisecond)
            case <-ctx2.Done():
                return
            }
        }
    }()
    
    // Fan-out to multiple workers
    workerOutputs := fanOut(ctx2, numbers, 3)
    
    // Fan-in results
    finalOutput := fanIn(ctx2, workerOutputs...)
    
    // Collect final results
    results := make([]string, 0)
    for result := range finalOutput {
        results = append(results, result)
        fmt.Printf("Final result: %s\n", result)
    }
    
    fmt.Printf("Processed %d results\n", len(results))
    
    fmt.Println("\n=== Select with Context ===")
    
    ctx3, cancel3 := context.WithTimeout(context.Background(), 1*time.Second)
    defer cancel3()
    
    selectWithContext(ctx3)
}
```

Context with channels enables sophisticated coordination patterns like  
fan-out/fan-in, graceful shutdown of channel pipelines, and timeout-aware  
select operations for robust concurrent systems.  

## Context propagation patterns

Understanding how context flows through application layers and service  
boundaries is crucial for building robust distributed systems.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// Domain layer
type Order struct {
    ID       string
    UserID   string
    Amount   float64
    Status   string
    Items    []OrderItem
}

type OrderItem struct {
    ProductID string
    Quantity  int
    Price     float64
}

// Repository layer
type OrderRepository struct {
    orders map[string]*Order
}

func NewOrderRepository() *OrderRepository {
    return &OrderRepository{
        orders: make(map[string]*Order),
    }
}

func (r *OrderRepository) GetByID(ctx context.Context, orderID string) (*Order, error) {
    // Simulate database query latency
    select {
    case <-time.After(100 * time.Millisecond):
    case <-ctx.Done():
        return nil, fmt.Errorf("repository query cancelled: %w", ctx.Err())
    }
    
    if order, exists := r.orders[orderID]; exists {
        fmt.Printf("Repository: found order %s\n", orderID)
        return order, nil
    }
    
    return nil, fmt.Errorf("order not found: %s", orderID)
}

func (r *OrderRepository) Save(ctx context.Context, order *Order) error {
    select {
    case <-time.After(150 * time.Millisecond):
    case <-ctx.Done():
        return fmt.Errorf("repository save cancelled: %w", ctx.Err())
    }
    
    r.orders[order.ID] = order
    fmt.Printf("Repository: saved order %s\n", order.ID)
    return nil
}

// Service layer
type PaymentService struct {
    processingTime time.Duration
}

func (p *PaymentService) ProcessPayment(ctx context.Context, orderID string, amount float64) error {
    // Extract context values for logging/tracing
    if requestID := ctx.Value("requestID"); requestID != nil {
        fmt.Printf("Payment service processing (request: %v)\n", requestID)
    }
    
    // Create child context with payment-specific timeout
    paymentCtx, cancel := context.WithTimeout(ctx, p.processingTime)
    defer cancel()
    
    select {
    case <-time.After(p.processingTime):
        fmt.Printf("Payment processed for order %s: $%.2f\n", orderID, amount)
        return nil
    case <-paymentCtx.Done():
        return fmt.Errorf("payment timeout for order %s: %w", orderID, paymentCtx.Err())
    }
}

type NotificationService struct{}

func (n *NotificationService) SendOrderConfirmation(ctx context.Context, userID, orderID string) error {
    // Notification doesn't block main flow, but respects cancellation
    select {
    case <-time.After(50 * time.Millisecond):
        fmt.Printf("Notification sent to user %s for order %s\n", userID, orderID)
        return nil
    case <-ctx.Done():
        fmt.Printf("Notification cancelled for order %s\n", orderID)
        return ctx.Err()
    }
}

// Application service (orchestrates business logic)
type OrderService struct {
    repository        *OrderRepository
    paymentService    *PaymentService
    notificationService *NotificationService
}

func NewOrderService(repo *OrderRepository, payment *PaymentService, notification *NotificationService) *OrderService {
    return &OrderService{
        repository:        repo,
        paymentService:    payment,
        notificationService: notification,
    }
}

func (s *OrderService) ProcessOrder(ctx context.Context, orderID string) error {
    // Add operation context
    ctx = context.WithValue(ctx, "operation", "process_order")
    
    // Step 1: Get order from repository
    order, err := s.repository.GetByID(ctx, orderID)
    if err != nil {
        return fmt.Errorf("failed to get order: %w", err)
    }
    
    // Step 2: Process payment with child context
    err = s.paymentService.ProcessPayment(ctx, order.ID, order.Amount)
    if err != nil {
        return fmt.Errorf("payment failed: %w", err)
    }
    
    // Step 3: Update order status
    order.Status = "paid"
    err = s.repository.Save(ctx, order)
    if err != nil {
        return fmt.Errorf("failed to update order: %w", err)
    }
    
    // Step 4: Send notification (non-blocking)
    go func() {
        // Create detached context for notification
        notificationCtx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        
        // Propagate important values to detached context
        if requestID := ctx.Value("requestID"); requestID != nil {
            notificationCtx = context.WithValue(notificationCtx, "requestID", requestID)
        }
        
        s.notificationService.SendOrderConfirmation(notificationCtx, order.UserID, order.ID)
    }()
    
    fmt.Printf("Order %s processed successfully\n", orderID)
    return nil
}

// HTTP layer simulation
func handleOrderRequest(ctx context.Context, orderService *OrderService, orderID string) {
    // Add request-specific context values
    requestID := fmt.Sprintf("req_%d", time.Now().UnixNano())
    ctx = context.WithValue(ctx, "requestID", requestID)
    ctx = context.WithValue(ctx, "userAgent", "OrderApp/1.0")
    
    fmt.Printf("Handling request %s for order %s\n", requestID, orderID)
    
    // Create request timeout
    requestCtx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    err := orderService.ProcessOrder(requestCtx, orderID)
    if err != nil {
        fmt.Printf("Request %s failed: %v\n", requestID, err)
    } else {
        fmt.Printf("Request %s completed successfully\n", requestID)
    }
}

// Background job processing
func backgroundOrderProcessor(ctx context.Context, orderService *OrderService, orders []string) {
    for _, orderID := range orders {
        select {
        case <-ctx.Done():
            fmt.Printf("Background processing cancelled\n")
            return
        default:
        }
        
        // Create job-specific context
        jobCtx := context.WithValue(ctx, "jobType", "background_order_processing")
        jobCtx = context.WithValue(jobCtx, "orderID", orderID)
        
        // Process with timeout
        jobCtx, cancel := context.WithTimeout(jobCtx, 2*time.Second)
        
        fmt.Printf("Background processing order %s\n", orderID)
        err := orderService.ProcessOrder(jobCtx, orderID)
        if err != nil {
            fmt.Printf("Background job failed for order %s: %v\n", orderID, err)
        }
        
        cancel()
        time.Sleep(100 * time.Millisecond) // Small delay between jobs
    }
}

func main() {
    // Set up services
    repo := NewOrderRepository()
    payment := &PaymentService{processingTime: 200 * time.Millisecond}
    notification := &NotificationService{}
    orderService := NewOrderService(repo, payment, notification)
    
    // Pre-populate some orders
    testOrders := []*Order{
        {ID: "order1", UserID: "user123", Amount: 99.99, Status: "pending"},
        {ID: "order2", UserID: "user456", Amount: 149.50, Status: "pending"},
        {ID: "order3", UserID: "user789", Amount: 75.25, Status: "pending"},
    }
    
    for _, order := range testOrders {
        repo.orders[order.ID] = order
    }
    
    fmt.Println("=== HTTP Request Processing ===")
    
    // Simulate HTTP requests
    rootCtx := context.Background()
    handleOrderRequest(rootCtx, orderService, "order1")
    handleOrderRequest(rootCtx, orderService, "order2")
    
    fmt.Println("\n=== Background Job Processing ===")
    
    // Simulate background job processing
    bgCtx, bgCancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer bgCancel()
    
    backgroundOrderProcessor(bgCtx, orderService, []string{"order3", "order1"})
    
    fmt.Println("\n=== Cancelled Request ===")
    
    // Simulate cancelled request
    cancelledCtx, cancel := context.WithCancel(context.Background())
    
    go func() {
        time.Sleep(500 * time.Millisecond)
        fmt.Println("Cancelling request...")
        cancel()
    }()
    
    handleOrderRequest(cancelledCtx, orderService, "order2")
    
    // Wait for any remaining notifications
    time.Sleep(500 * time.Millisecond)
}
```

Context propagation enables request tracing, timeout coordination, and  
value passing across application layers while maintaining clean separation  
of concerns and proper cancellation semantics.  

## Custom context implementations

Creating custom context implementations enables specialized behavior for  
specific use cases while maintaining compatibility with the standard interface.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Custom context that tracks operation metrics
type MetricContext struct {
    context.Context
    mu         sync.RWMutex
    startTime  time.Time
    operations map[string]int
    errors     map[string]int
}

func NewMetricContext(parent context.Context) *MetricContext {
    return &MetricContext{
        Context:    parent,
        startTime:  time.Now(),
        operations: make(map[string]int),
        errors:     make(map[string]int),
    }
}

func (mc *MetricContext) RecordOperation(operation string) {
    mc.mu.Lock()
    defer mc.mu.Unlock()
    mc.operations[operation]++
}

func (mc *MetricContext) RecordError(operation string) {
    mc.mu.Lock()
    defer mc.mu.Unlock()
    mc.errors[operation]++
}

func (mc *MetricContext) GetMetrics() (map[string]int, map[string]int, time.Duration) {
    mc.mu.RLock()
    defer mc.mu.RUnlock()
    
    ops := make(map[string]int)
    errs := make(map[string]int)
    
    for k, v := range mc.operations {
        ops[k] = v
    }
    for k, v := range mc.errors {
        errs[k] = v
    }
    
    return ops, errs, time.Since(mc.startTime)
}

// Context with priority levels
type PriorityContext struct {
    context.Context
    priority int
}

func WithPriority(parent context.Context, priority int) *PriorityContext {
    return &PriorityContext{
        Context:  parent,
        priority: priority,
    }
}

func (pc *PriorityContext) Priority() int {
    return pc.priority
}

func GetPriority(ctx context.Context) int {
    if pc, ok := ctx.(*PriorityContext); ok {
        return pc.Priority()
    }
    return 0 // Default priority
}

// Context with retry configuration
type RetryContext struct {
    context.Context
    maxRetries    int
    currentRetry  int
    backoffFunc   func(int) time.Duration
}

func WithRetry(parent context.Context, maxRetries int, backoffFunc func(int) time.Duration) *RetryContext {
    return &RetryContext{
        Context:     parent,
        maxRetries:  maxRetries,
        currentRetry: 0,
        backoffFunc: backoffFunc,
    }
}

func (rc *RetryContext) CanRetry() bool {
    return rc.currentRetry < rc.maxRetries
}

func (rc *RetryContext) NextRetry() *RetryContext {
    return &RetryContext{
        Context:      rc.Context,
        maxRetries:   rc.maxRetries,
        currentRetry: rc.currentRetry + 1,
        backoffFunc:  rc.backoffFunc,
    }
}

func (rc *RetryContext) BackoffDuration() time.Duration {
    if rc.backoffFunc != nil {
        return rc.backoffFunc(rc.currentRetry)
    }
    return time.Duration(rc.currentRetry) * 100 * time.Millisecond
}

func (rc *RetryContext) RetryInfo() (int, int) {
    return rc.currentRetry, rc.maxRetries
}

// Context that aggregates multiple parent contexts
type MultiContext struct {
    contexts []context.Context
    done     chan struct{}
    err      error
    mu       sync.RWMutex
}

func WithMultiple(contexts ...context.Context) *MultiContext {
    mc := &MultiContext{
        contexts: contexts,
        done:     make(chan struct{}),
    }
    
    // Monitor all parent contexts
    for i, ctx := range contexts {
        go func(index int, c context.Context) {
            select {
            case <-c.Done():
                mc.mu.Lock()
                if mc.err == nil {
                    mc.err = fmt.Errorf("context %d cancelled: %w", index, c.Err())
                    close(mc.done)
                }
                mc.mu.Unlock()
            case <-mc.done:
                return
            }
        }(i, ctx)
    }
    
    return mc
}

func (mc *MultiContext) Deadline() (deadline time.Time, ok bool) {
    var earliest time.Time
    hasDeadline := false
    
    for _, ctx := range mc.contexts {
        if deadline, ok := ctx.Deadline(); ok {
            if !hasDeadline || deadline.Before(earliest) {
                earliest = deadline
                hasDeadline = true
            }
        }
    }
    
    return earliest, hasDeadline
}

func (mc *MultiContext) Done() <-chan struct{} {
    return mc.done
}

func (mc *MultiContext) Err() error {
    mc.mu.RLock()
    defer mc.mu.RUnlock()
    return mc.err
}

func (mc *MultiContext) Value(key interface{}) interface{} {
    // Check all parent contexts for the value
    for _, ctx := range mc.contexts {
        if value := ctx.Value(key); value != nil {
            return value
        }
    }
    return nil
}

// Operation functions using custom contexts
func simulateDataProcessing(ctx context.Context, dataSize int) error {
    // Record operation if using MetricContext
    if mc, ok := ctx.(*MetricContext); ok {
        mc.RecordOperation("data_processing")
        defer func() {
            if r := recover(); r != nil {
                mc.RecordError("data_processing")
            }
        }()
    }
    
    // Check priority
    priority := GetPriority(ctx)
    processingTime := time.Duration(dataSize) * time.Millisecond
    if priority > 5 {
        processingTime = processingTime / 2 // High priority processes faster
    }
    
    fmt.Printf("Processing %d units of data (priority: %d, time: %v)\n", 
              dataSize, priority, processingTime)
    
    select {
    case <-time.After(processingTime):
        fmt.Printf("Data processing completed\n")
        return nil
    case <-ctx.Done():
        fmt.Printf("Data processing cancelled: %v\n", ctx.Err())
        return ctx.Err()
    }
}

func reliableNetworkCall(ctx context.Context, url string) error {
    if rc, ok := ctx.(*RetryContext); ok {
        current, max := rc.RetryInfo()
        fmt.Printf("Network call attempt %d/%d to %s\n", current+1, max+1, url)
        
        // Simulate network call
        select {
        case <-time.After(200 * time.Millisecond):
            // Simulate failure for demonstration
            if current < 2 {
                if rc.CanRetry() {
                    time.Sleep(rc.BackoffDuration())
                    return reliableNetworkCall(rc.NextRetry(), url)
                }
                return fmt.Errorf("network call failed after %d retries", max)
            }
            fmt.Printf("Network call succeeded on attempt %d\n", current+1)
            return nil
        case <-ctx.Done():
            return ctx.Err()
        }
    }
    
    // Fallback to simple call without retry
    select {
    case <-time.After(200 * time.Millisecond):
        fmt.Printf("Simple network call to %s completed\n", url)
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

func main() {
    fmt.Println("=== Custom Context Examples ===")
    
    // Example 1: MetricContext
    fmt.Println("\n--- MetricContext ---")
    metricCtx := NewMetricContext(context.Background())
    
    // Perform operations
    simulateDataProcessing(metricCtx, 100)
    simulateDataProcessing(metricCtx, 200)
    
    // Get metrics
    ops, errs, duration := metricCtx.GetMetrics()
    fmt.Printf("Operations: %v\n", ops)
    fmt.Printf("Errors: %v\n", errs)
    fmt.Printf("Total duration: %v\n", duration)
    
    // Example 2: PriorityContext
    fmt.Println("\n--- PriorityContext ---")
    lowPriorityCtx := WithPriority(context.Background(), 3)
    highPriorityCtx := WithPriority(context.Background(), 8)
    
    simulateDataProcessing(lowPriorityCtx, 300)
    simulateDataProcessing(highPriorityCtx, 300)
    
    // Example 3: RetryContext
    fmt.Println("\n--- RetryContext ---")
    exponentialBackoff := func(attempt int) time.Duration {
        return time.Duration(1<<uint(attempt)) * 100 * time.Millisecond
    }
    
    retryCtx := WithRetry(context.Background(), 3, exponentialBackoff)
    err := reliableNetworkCall(retryCtx, "https://api.example.com")
    if err != nil {
        fmt.Printf("Network call failed: %v\n", err)
    }
    
    // Example 4: MultiContext
    fmt.Println("\n--- MultiContext ---")
    ctx1, cancel1 := context.WithTimeout(context.Background(), 2*time.Second)
    ctx2, cancel2 := context.WithCancel(context.Background())
    ctx3 := context.WithValue(context.Background(), "source", "multi-context")
    
    defer cancel1()
    defer cancel2()
    
    multiCtx := WithMultiple(ctx1, ctx2, ctx3)
    
    // Cancel one of the parent contexts after a delay
    go func() {
        time.Sleep(500 * time.Millisecond)
        fmt.Println("Cancelling parent context...")
        cancel2()
    }()
    
    // This operation will be cancelled when ctx2 is cancelled
    select {
    case <-time.After(1 * time.Second):
        fmt.Println("MultiContext operation completed")
    case <-multiCtx.Done():
        fmt.Printf("MultiContext cancelled: %v\n", multiCtx.Err())
    }
    
    // Check value propagation
    source := multiCtx.Value("source")
    fmt.Printf("Value from MultiContext: %v\n", source)
    
    // Example 5: Combining custom contexts
    fmt.Println("\n--- Combined Custom Contexts ---")
    combinedCtx := NewMetricContext(WithPriority(context.Background(), 7))
    
    // This will use both metric tracking and priority
    simulateDataProcessing(combinedCtx, 150)
    
    ops, errs, duration = combinedCtx.GetMetrics()
    fmt.Printf("Combined context - Operations: %v, Duration: %v\n", ops, duration)
}
```

Custom context implementations enable specialized behaviors like metrics  
collection, priority handling, retry logic, and multi-parent coordination  
while maintaining standard context interface compatibility.  

## Context cancellation patterns

Advanced cancellation patterns enable sophisticated control over concurrent  
operations with graceful degradation and resource cleanup.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Cancellation coordinator for managing multiple operations
type CancellationCoordinator struct {
    mu        sync.RWMutex
    contexts  map[string]context.CancelFunc
    groups    map[string][]string
}

func NewCancellationCoordinator() *CancellationCoordinator {
    return &CancellationCoordinator{
        contexts: make(map[string]context.CancelFunc),
        groups:   make(map[string][]string),
    }
}

func (cc *CancellationCoordinator) Register(id string, cancel context.CancelFunc) {
    cc.mu.Lock()
    defer cc.mu.Unlock()
    cc.contexts[id] = cancel
}

func (cc *CancellationCoordinator) AddToGroup(groupName, id string) {
    cc.mu.Lock()
    defer cc.mu.Unlock()
    cc.groups[groupName] = append(cc.groups[groupName], id)
}

func (cc *CancellationCoordinator) Cancel(id string) bool {
    cc.mu.RLock()
    cancel, exists := cc.contexts[id]
    cc.mu.RUnlock()
    
    if exists {
        cancel()
        return true
    }
    return false
}

func (cc *CancellationCoordinator) CancelGroup(groupName string) int {
    cc.mu.RLock()
    ids := cc.groups[groupName]
    cc.mu.RUnlock()
    
    cancelled := 0
    for _, id := range ids {
        if cc.Cancel(id) {
            cancelled++
        }
    }
    return cancelled
}

func (cc *CancellationCoordinator) CancelAll() int {
    cc.mu.RLock()
    allCancels := make([]context.CancelFunc, 0, len(cc.contexts))
    for _, cancel := range cc.contexts {
        allCancels = append(allCancels, cancel)
    }
    cc.mu.RUnlock()
    
    for _, cancel := range allCancels {
        cancel()
    }
    return len(allCancels)
}

// Graceful shutdown pattern
type GracefulShutdown struct {
    ctx        context.Context
    cancel     context.CancelFunc
    operations sync.WaitGroup
    shutdownCh chan struct{}
    once       sync.Once
}

func NewGracefulShutdown() *GracefulShutdown {
    ctx, cancel := context.WithCancel(context.Background())
    return &GracefulShutdown{
        ctx:        ctx,
        cancel:     cancel,
        shutdownCh: make(chan struct{}),
    }
}

func (gs *GracefulShutdown) Context() context.Context {
    return gs.ctx
}

func (gs *GracefulShutdown) AddOperation() context.Context {
    gs.operations.Add(1)
    return gs.ctx
}

func (gs *GracefulShutdown) OperationDone() {
    gs.operations.Done()
}

func (gs *GracefulShutdown) Shutdown() <-chan struct{} {
    gs.once.Do(func() {
        fmt.Println("Initiating graceful shutdown...")
        gs.cancel()
        
        go func() {
            gs.operations.Wait()
            close(gs.shutdownCh)
            fmt.Println("Graceful shutdown completed")
        }()
    })
    
    return gs.shutdownCh
}

// Timeout cascade pattern
func timeoutCascade(ctx context.Context, operations []func(context.Context) error, timeouts []time.Duration) error {
    if len(operations) != len(timeouts) {
        return fmt.Errorf("operations and timeouts length mismatch")
    }
    
    currentCtx := ctx
    
    for i, op := range operations {
        // Create context with specific timeout for this operation
        opCtx, cancel := context.WithTimeout(currentCtx, timeouts[i])
        
        fmt.Printf("Executing operation %d with timeout %v\n", i+1, timeouts[i])
        
        err := op(opCtx)
        cancel()
        
        if err != nil {
            return fmt.Errorf("operation %d failed: %w", i+1, err)
        }
        
        // Check if parent context is still valid
        if ctx.Err() != nil {
            return fmt.Errorf("parent context cancelled")
        }
    }
    
    return nil
}

// Circuit breaker with context
type CircuitBreaker struct {
    mu          sync.RWMutex
    failures    int
    threshold   int
    timeout     time.Duration
    lastFailure time.Time
    state       string // "closed", "open", "half-open"
}

func NewCircuitBreaker(threshold int, timeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        threshold: threshold,
        timeout:   timeout,
        state:     "closed",
    }
}

func (cb *CircuitBreaker) Execute(ctx context.Context, operation func(context.Context) error) error {
    cb.mu.RLock()
    state := cb.state
    cb.mu.RUnlock()
    
    switch state {
    case "open":
        cb.mu.RLock()
        canTry := time.Since(cb.lastFailure) > cb.timeout
        cb.mu.RUnlock()
        
        if !canTry {
            return fmt.Errorf("circuit breaker open")
        }
        
        cb.mu.Lock()
        cb.state = "half-open"
        cb.mu.Unlock()
        fallthrough
        
    case "half-open":
        err := operation(ctx)
        if err != nil {
            cb.recordFailure()
            return err
        }
        cb.recordSuccess()
        return nil
        
    case "closed":
        err := operation(ctx)
        if err != nil {
            cb.recordFailure()
            return err
        }
        return nil
    }
    
    return fmt.Errorf("unknown circuit breaker state")
}

func (cb *CircuitBreaker) recordFailure() {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    cb.failures++
    cb.lastFailure = time.Now()
    
    if cb.failures >= cb.threshold {
        cb.state = "open"
        fmt.Printf("Circuit breaker opened after %d failures\n", cb.failures)
    }
}

func (cb *CircuitBreaker) recordSuccess() {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    cb.failures = 0
    cb.state = "closed"
    fmt.Println("Circuit breaker closed - operation successful")
}

// Example operations
func simulateOperation(ctx context.Context, id string, duration time.Duration, shouldFail bool) error {
    fmt.Printf("Operation %s: starting (duration: %v)\n", id, duration)
    
    select {
    case <-time.After(duration):
        if shouldFail {
            fmt.Printf("Operation %s: failed\n", id)
            return fmt.Errorf("operation %s failed", id)
        }
        fmt.Printf("Operation %s: completed successfully\n", id)
        return nil
    case <-ctx.Done():
        fmt.Printf("Operation %s: cancelled (%v)\n", id, ctx.Err())
        return ctx.Err()
    }
}

func longRunningWorker(ctx context.Context, gs *GracefulShutdown, workerID int) {
    opCtx := gs.AddOperation()
    defer gs.OperationDone()
    
    fmt.Printf("Worker %d: started\n", workerID)
    
    for i := 1; ; i++ {
        select {
        case <-opCtx.Done():
            fmt.Printf("Worker %d: shutting down gracefully after %d iterations\n", workerID, i-1)
            return
        default:
            fmt.Printf("Worker %d: iteration %d\n", workerID, i)
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    fmt.Println("=== Advanced Cancellation Patterns ===")
    
    // Example 1: Cancellation Coordinator
    fmt.Println("\n--- Cancellation Coordinator ---")
    coordinator := NewCancellationCoordinator()
    
    // Create multiple operations
    for i := 1; i <= 5; i++ {
        ctx, cancel := context.WithCancel(context.Background())
        id := fmt.Sprintf("op%d", i)
        coordinator.Register(id, cancel)
        
        if i <= 2 {
            coordinator.AddToGroup("batch1", id)
        } else {
            coordinator.AddToGroup("batch2", id)
        }
        
        go func(opID string, c context.Context) {
            select {
            case <-time.After(5 * time.Second):
                fmt.Printf("Operation %s: completed\n", opID)
            case <-c.Done():
                fmt.Printf("Operation %s: cancelled\n", opID)
            }
        }(id, ctx)
    }
    
    // Cancel operations in groups
    time.Sleep(1 * time.Second)
    cancelled := coordinator.CancelGroup("batch1")
    fmt.Printf("Cancelled %d operations in batch1\n", cancelled)
    
    time.Sleep(1 * time.Second)
    cancelled = coordinator.CancelAll()
    fmt.Printf("Cancelled %d remaining operations\n", cancelled)
    
    // Example 2: Graceful Shutdown
    fmt.Println("\n--- Graceful Shutdown ---")
    gs := NewGracefulShutdown()
    
    // Start workers
    for i := 1; i <= 3; i++ {
        go longRunningWorker(gs.Context(), gs, i)
    }
    
    // Let workers run
    time.Sleep(2 * time.Second)
    
    // Initiate shutdown
    shutdownComplete := gs.Shutdown()
    
    // Wait for graceful shutdown
    <-shutdownComplete
    
    // Example 3: Timeout Cascade
    fmt.Println("\n--- Timeout Cascade ---")
    operations := []func(context.Context) error{
        func(ctx context.Context) error {
            return simulateOperation(ctx, "fast", 200*time.Millisecond, false)
        },
        func(ctx context.Context) error {
            return simulateOperation(ctx, "medium", 500*time.Millisecond, false)
        },
        func(ctx context.Context) error {
            return simulateOperation(ctx, "slow", 800*time.Millisecond, false)
        },
    }
    
    timeouts := []time.Duration{
        500 * time.Millisecond,
        1 * time.Second,
        1200 * time.Millisecond,
    }
    
    ctx := context.Background()
    err := timeoutCascade(ctx, operations, timeouts)
    if err != nil {
        fmt.Printf("Cascade failed: %v\n", err)
    } else {
        fmt.Println("Cascade completed successfully")
    }
    
    // Example 4: Circuit Breaker
    fmt.Println("\n--- Circuit Breaker ---")
    cb := NewCircuitBreaker(3, 2*time.Second)
    
    // Simulate operations with some failures
    for i := 1; i <= 8; i++ {
        ctx := context.Background()
        shouldFail := i >= 2 && i <= 4 // Fail operations 2, 3, 4
        
        err := cb.Execute(ctx, func(ctx context.Context) error {
            return simulateOperation(ctx, fmt.Sprintf("cb-op-%d", i), 100*time.Millisecond, shouldFail)
        })
        
        if err != nil {
            fmt.Printf("Circuit breaker execution %d failed: %v\n", i, err)
        }
        
        time.Sleep(300 * time.Millisecond)
    }
}
```

Advanced cancellation patterns provide sophisticated control over operation  
lifecycles, enabling graceful shutdowns, coordinated cancellation, timeout  
cascades, and circuit breaker patterns for robust system design.  

## Context performance optimization

Optimizing context usage for performance-critical applications requires  
understanding context overhead and implementing efficient patterns.  

```go
package main

import (
    "context"
    "fmt"
    "runtime"
    "sync"
    "time"
)

// Optimized context pool
type ContextPool struct {
    pool sync.Pool
}

func NewContextPool() *ContextPool {
    return &ContextPool{
        pool: sync.Pool{
            New: func() interface{} {
                return &pooledContext{
                    Context: context.Background(),
                    values:  make(map[interface{}]interface{}),
                }
            },
        },
    }
}

type pooledContext struct {
    context.Context
    values map[interface{}]interface{}
    mu     sync.RWMutex
}

func (pc *pooledContext) Value(key interface{}) interface{} {
    pc.mu.RLock()
    defer pc.mu.RUnlock()
    
    if value, exists := pc.values[key]; exists {
        return value
    }
    return pc.Context.Value(key)
}

func (pc *pooledContext) SetValue(key, value interface{}) {
    pc.mu.Lock()
    defer pc.mu.Unlock()
    pc.values[key] = value
}

func (pc *pooledContext) Reset() {
    pc.mu.Lock()
    defer pc.mu.Unlock()
    
    // Clear values map
    for k := range pc.values {
        delete(pc.values, k)
    }
    pc.Context = context.Background()
}

func (cp *ContextPool) Get() *pooledContext {
    return cp.pool.Get().(*pooledContext)
}

func (cp *ContextPool) Put(ctx *pooledContext) {
    ctx.Reset()
    cp.pool.Put(ctx)
}

// Batch context operations
type BatchContext struct {
    contexts []context.Context
    mu       sync.RWMutex
}

func NewBatchContext() *BatchContext {
    return &BatchContext{
        contexts: make([]context.Context, 0, 10),
    }
}

func (bc *BatchContext) Add(ctx context.Context) {
    bc.mu.Lock()
    defer bc.mu.Unlock()
    bc.contexts = append(bc.contexts, ctx)
}

func (bc *BatchContext) ProcessBatch(operation func(context.Context) error) []error {
    bc.mu.RLock()
    contexts := make([]context.Context, len(bc.contexts))
    copy(contexts, bc.contexts)
    bc.mu.RUnlock()
    
    errors := make([]error, len(contexts))
    var wg sync.WaitGroup
    
    wg.Add(len(contexts))
    for i, ctx := range contexts {
        go func(index int, c context.Context) {
            defer wg.Done()
            errors[index] = operation(c)
        }(i, ctx)
    }
    
    wg.Wait()
    return errors
}

// Lightweight context for high-frequency operations
type LightContext struct {
    deadline time.Time
    done     chan struct{}
    err      error
    values   map[interface{}]interface{}
    once     sync.Once
}

func NewLightContext(timeout time.Duration) *LightContext {
    ctx := &LightContext{
        deadline: time.Now().Add(timeout),
        done:     make(chan struct{}),
        values:   make(map[interface{}]interface{}),
    }
    
    // Start timeout timer
    go func() {
        time.Sleep(timeout)
        ctx.once.Do(func() {
            ctx.err = context.DeadlineExceeded
            close(ctx.done)
        })
    }()
    
    return ctx
}

func (lc *LightContext) Deadline() (deadline time.Time, ok bool) {
    return lc.deadline, true
}

func (lc *LightContext) Done() <-chan struct{} {
    return lc.done
}

func (lc *LightContext) Err() error {
    return lc.err
}

func (lc *LightContext) Value(key interface{}) interface{} {
    return lc.values[key]
}

func (lc *LightContext) SetValue(key, value interface{}) {
    lc.values[key] = value
}

// Performance benchmarking
func benchmarkContextCreation(iterations int) time.Duration {
    start := time.Now()
    
    for i := 0; i < iterations; i++ {
        ctx, cancel := context.WithTimeout(context.Background(), time.Second)
        _ = ctx
        cancel()
    }
    
    return time.Since(start)
}

func benchmarkPooledContext(pool *ContextPool, iterations int) time.Duration {
    start := time.Now()
    
    for i := 0; i < iterations; i++ {
        ctx := pool.Get()
        ctx.SetValue("iteration", i)
        _ = ctx.Value("iteration")
        pool.Put(ctx)
    }
    
    return time.Since(start)
}

func benchmarkLightContext(iterations int) time.Duration {
    start := time.Now()
    
    for i := 0; i < iterations; i++ {
        ctx := NewLightContext(time.Second)
        ctx.SetValue("iteration", i)
        _ = ctx.Value("iteration")
    }
    
    return time.Since(start)
}

// Memory-efficient context value storage
type CompactContext struct {
    context.Context
    keys   []interface{}
    values []interface{}
}

func WithCompactValue(parent context.Context, key, value interface{}) *CompactContext {
    if cc, ok := parent.(*CompactContext); ok {
        return &CompactContext{
            Context: cc.Context,
            keys:    append(cc.keys, key),
            values:  append(cc.values, value),
        }
    }
    
    return &CompactContext{
        Context: parent,
        keys:    []interface{}{key},
        values:  []interface{}{value},
    }
}

func (cc *CompactContext) Value(key interface{}) interface{} {
    for i, k := range cc.keys {
        if k == key {
            return cc.values[i]
        }
    }
    return cc.Context.Value(key)
}

func main() {
    fmt.Println("=== Context Performance Optimization ===")
    
    const iterations = 100000
    
    // Benchmark standard context creation
    fmt.Println("\n--- Standard Context Benchmark ---")
    standardTime := benchmarkContextCreation(iterations)
    fmt.Printf("Standard context creation (%d iterations): %v\n", iterations, standardTime)
    fmt.Printf("Average per operation: %v\n", standardTime/time.Duration(iterations))
    
    // Benchmark pooled context
    fmt.Println("\n--- Pooled Context Benchmark ---")
    pool := NewContextPool()
    pooledTime := benchmarkPooledContext(pool, iterations)
    fmt.Printf("Pooled context (%d iterations): %v\n", iterations, pooledTime)
    fmt.Printf("Average per operation: %v\n", pooledTime/time.Duration(iterations))
    
    // Benchmark lightweight context
    fmt.Println("\n--- Lightweight Context Benchmark ---")
    lightTime := benchmarkLightContext(iterations)
    fmt.Printf("Lightweight context (%d iterations): %v\n", iterations, lightTime)
    fmt.Printf("Average per operation: %v\n", lightTime/time.Duration(iterations))
    
    // Memory usage comparison
    fmt.Println("\n--- Memory Usage Comparison ---")
    var m1, m2 runtime.MemStats
    
    // Standard context memory usage
    runtime.GC()
    runtime.ReadMemStats(&m1)
    
    contexts := make([]context.Context, 1000)
    for i := range contexts {
        ctx := context.WithValue(context.Background(), "key"+fmt.Sprint(i), "value"+fmt.Sprint(i))
        contexts[i] = ctx
    }
    
    runtime.ReadMemStats(&m2)
    standardMem := m2.HeapAlloc - m1.HeapAlloc
    fmt.Printf("Standard contexts (1000): %d bytes\n", standardMem)
    
    // Compact context memory usage
    runtime.GC()
    runtime.ReadMemStats(&m1)
    
    compactContexts := make([]*CompactContext, 1000)
    base := context.Background()
    for i := range compactContexts {
        ctx := WithCompactValue(base, "key"+fmt.Sprint(i), "value"+fmt.Sprint(i))
        compactContexts[i] = ctx
    }
    
    runtime.ReadMemStats(&m2)
    compactMem := m2.HeapAlloc - m1.HeapAlloc
    fmt.Printf("Compact contexts (1000): %d bytes\n", compactMem)
    
    if standardMem > 0 {
        fmt.Printf("Memory savings: %.1f%%\n", float64(standardMem-compactMem)/float64(standardMem)*100)
    }
    
    // Batch processing demonstration
    fmt.Println("\n--- Batch Processing ---")
    batch := NewBatchContext()
    
    // Add contexts to batch
    for i := 0; i < 5; i++ {
        ctx, cancel := context.WithTimeout(context.Background(), time.Duration(i+1)*100*time.Millisecond)
        defer cancel()
        batch.Add(ctx)
    }
    
    // Process batch
    start := time.Now()
    errors := batch.ProcessBatch(func(ctx context.Context) error {
        select {
        case <-time.After(150 * time.Millisecond):
            return nil
        case <-ctx.Done():
            return ctx.Err()
        }
    })
    
    fmt.Printf("Batch processing completed in: %v\n", time.Since(start))
    for i, err := range errors {
        if err != nil {
            fmt.Printf("Context %d error: %v\n", i, err)
        } else {
            fmt.Printf("Context %d: success\n", i)
        }
    }
    
    // Performance comparison summary
    fmt.Println("\n--- Performance Summary ---")
    fmt.Printf("Standard context: %v per operation\n", standardTime/time.Duration(iterations))
    fmt.Printf("Pooled context:   %v per operation (%.1fx faster)\n", 
              pooledTime/time.Duration(iterations),
              float64(standardTime)/float64(pooledTime))
    fmt.Printf("Light context:    %v per operation (%.1fx faster)\n", 
              lightTime/time.Duration(iterations),
              float64(standardTime)/float64(lightTime))
}
```

Performance optimization techniques include context pooling, lightweight  
implementations, batch processing, and memory-efficient value storage  
for high-throughput applications.  

## Context debugging and monitoring

Debugging and monitoring context usage helps identify performance issues,  
cancellation problems, and resource leaks in complex applications.  

```go
package main

import (
    "context"
    "fmt"
    "runtime"
    "sync"
    "sync/atomic"
    "time"
)

// Context monitor for tracking usage statistics
type ContextMonitor struct {
    mu              sync.RWMutex
    activeContexts  int64
    totalCreated    int64
    totalCancelled  int64
    timeouts        int64
    deadlines       int64
    values          int64
    contexts        map[string]*ContextInfo
}

type ContextInfo struct {
    ID        string
    CreatedAt time.Time
    Type      string
    Parent    string
    Stack     string
    Active    bool
}

func NewContextMonitor() *ContextMonitor {
    return &ContextMonitor{
        contexts: make(map[string]*ContextInfo),
    }
}

func (cm *ContextMonitor) RegisterContext(id, contextType, parentID string) {
    atomic.AddInt64(&cm.totalCreated, 1)
    atomic.AddInt64(&cm.activeContexts, 1)
    
    cm.mu.Lock()
    defer cm.mu.Unlock()
    
    // Capture stack trace
    buf := make([]byte, 1024)
    n := runtime.Stack(buf, false)
    stack := string(buf[:n])
    
    cm.contexts[id] = &ContextInfo{
        ID:        id,
        CreatedAt: time.Now(),
        Type:      contextType,
        Parent:    parentID,
        Stack:     stack,
        Active:    true,
    }
    
    switch contextType {
    case "timeout":
        atomic.AddInt64(&cm.timeouts, 1)
    case "deadline":
        atomic.AddInt64(&cm.deadlines, 1)
    case "value":
        atomic.AddInt64(&cm.values, 1)
    }
}

func (cm *ContextMonitor) UnregisterContext(id string) {
    atomic.AddInt64(&cm.totalCancelled, 1)
    atomic.AddInt64(&cm.activeContexts, -1)
    
    cm.mu.Lock()
    defer cm.mu.Unlock()
    
    if info, exists := cm.contexts[id]; exists {
        info.Active = false
    }
}

func (cm *ContextMonitor) GetStats() map[string]int64 {
    return map[string]int64{
        "active":     atomic.LoadInt64(&cm.activeContexts),
        "created":    atomic.LoadInt64(&cm.totalCreated),
        "cancelled":  atomic.LoadInt64(&cm.totalCancelled),
        "timeouts":   atomic.LoadInt64(&cm.timeouts),
        "deadlines":  atomic.LoadInt64(&cm.deadlines),
        "values":     atomic.LoadInt64(&cm.values),
    }
}

func (cm *ContextMonitor) PrintReport() {
    stats := cm.GetStats()
    fmt.Println("\n=== Context Monitor Report ===")
    fmt.Printf("Active contexts:    %d\n", stats["active"])
    fmt.Printf("Total created:      %d\n", stats["created"])
    fmt.Printf("Total cancelled:    %d\n", stats["cancelled"])
    fmt.Printf("Timeout contexts:   %d\n", stats["timeouts"])
    fmt.Printf("Deadline contexts:  %d\n", stats["deadlines"])
    fmt.Printf("Value contexts:     %d\n", stats["values"])
    
    cm.mu.RLock()
    defer cm.mu.RUnlock()
    
    fmt.Println("\nLong-running contexts:")
    threshold := time.Now().Add(-5 * time.Second)
    for _, info := range cm.contexts {
        if info.Active && info.CreatedAt.Before(threshold) {
            fmt.Printf("  %s (%s) - Age: %v\n", 
                      info.ID, info.Type, time.Since(info.CreatedAt))
        }
    }
}

// Traced context for debugging
type TracedContext struct {
    context.Context
    id       string
    monitor  *ContextMonitor
    parentID string
    logger   *ContextLogger
}

type ContextLogger struct {
    mu   sync.Mutex
    logs []LogEntry
}

type LogEntry struct {
    Timestamp time.Time
    ContextID string
    Event     string
    Message   string
}

func NewContextLogger() *ContextLogger {
    return &ContextLogger{
        logs: make([]LogEntry, 0),
    }
}

func (cl *ContextLogger) Log(contextID, event, message string) {
    cl.mu.Lock()
    defer cl.mu.Unlock()
    
    cl.logs = append(cl.logs, LogEntry{
        Timestamp: time.Now(),
        ContextID: contextID,
        Event:     event,
        Message:   message,
    })
    
    fmt.Printf("[%s] Context %s: %s - %s\n", 
              time.Now().Format("15:04:05.000"), contextID, event, message)
}

func (cl *ContextLogger) GetLogs(contextID string) []LogEntry {
    cl.mu.Lock()
    defer cl.mu.Unlock()
    
    var logs []LogEntry
    for _, log := range cl.logs {
        if log.ContextID == contextID {
            logs = append(logs, log)
        }
    }
    return logs
}

func NewTracedContext(parent context.Context, id string, monitor *ContextMonitor, logger *ContextLogger) *TracedContext {
    parentID := ""
    if tc, ok := parent.(*TracedContext); ok {
        parentID = tc.id
    }
    
    monitor.RegisterContext(id, "traced", parentID)
    logger.Log(id, "CREATED", fmt.Sprintf("Parent: %s", parentID))
    
    return &TracedContext{
        Context:  parent,
        id:       id,
        monitor:  monitor,
        parentID: parentID,
        logger:   logger,
    }
}

func (tc *TracedContext) WithTimeout(timeout time.Duration) (*TracedContext, context.CancelFunc) {
    childID := tc.id + "-timeout"
    ctx, cancel := context.WithTimeout(tc.Context, timeout)
    
    traced := &TracedContext{
        Context:  ctx,
        id:       childID,
        monitor:  tc.monitor,
        parentID: tc.id,
        logger:   tc.logger,
    }
    
    tc.monitor.RegisterContext(childID, "timeout", tc.id)
    tc.logger.Log(childID, "TIMEOUT_SET", fmt.Sprintf("Duration: %v", timeout))
    
    return traced, func() {
        tc.logger.Log(childID, "CANCELLED", "Timeout context cancelled")
        tc.monitor.UnregisterContext(childID)
        cancel()
    }
}

func (tc *TracedContext) WithValue(key, value interface{}) *TracedContext {
    childID := tc.id + "-value"
    ctx := context.WithValue(tc.Context, key, value)
    
    traced := &TracedContext{
        Context:  ctx,
        id:       childID,
        monitor:  tc.monitor,
        parentID: tc.id,
        logger:   tc.logger,
    }
    
    tc.monitor.RegisterContext(childID, "value", tc.id)
    tc.logger.Log(childID, "VALUE_SET", fmt.Sprintf("Key: %v, Value: %v", key, value))
    
    return traced
}

func (tc *TracedContext) Done() <-chan struct{} {
    tc.logger.Log(tc.id, "DONE_ACCESSED", "Done channel accessed")
    return tc.Context.Done()
}

func (tc *TracedContext) Err() error {
    err := tc.Context.Err()
    if err != nil {
        tc.logger.Log(tc.id, "ERROR", fmt.Sprintf("Context error: %v", err))
    }
    return err
}

// Context leak detector
type LeakDetector struct {
    mu        sync.RWMutex
    contexts  map[string]time.Time
    threshold time.Duration
}

func NewLeakDetector(threshold time.Duration) *LeakDetector {
    return &LeakDetector{
        contexts:  make(map[string]time.Time),
        threshold: threshold,
    }
}

func (ld *LeakDetector) Register(id string) {
    ld.mu.Lock()
    defer ld.mu.Unlock()
    ld.contexts[id] = time.Now()
}

func (ld *LeakDetector) Unregister(id string) {
    ld.mu.Lock()
    defer ld.mu.Unlock()
    delete(ld.contexts, id)
}

func (ld *LeakDetector) CheckLeaks() []string {
    ld.mu.RLock()
    defer ld.mu.RUnlock()
    
    var leaks []string
    threshold := time.Now().Add(-ld.threshold)
    
    for id, created := range ld.contexts {
        if created.Before(threshold) {
            leaks = append(leaks, id)
        }
    }
    
    return leaks
}

// Context hierarchy visualizer
func visualizeContextHierarchy(monitor *ContextMonitor) {
    monitor.mu.RLock()
    defer monitor.mu.RUnlock()
    
    fmt.Println("\n=== Context Hierarchy ===")
    
    // Find root contexts (no parent)
    roots := make([]*ContextInfo, 0)
    children := make(map[string][]*ContextInfo)
    
    for _, info := range monitor.contexts {
        if info.Parent == "" {
            roots = append(roots, info)
        } else {
            children[info.Parent] = append(children[info.Parent], info)
        }
    }
    
    // Print hierarchy
    for _, root := range roots {
        printContextTree(root, children, 0)
    }
}

func printContextTree(info *ContextInfo, children map[string][]*ContextInfo, depth int) {
    indent := ""
    for i := 0; i < depth; i++ {
        indent += "  "
    }
    
    status := "ACTIVE"
    if !info.Active {
        status = "CANCELLED"
    }
    
    age := time.Since(info.CreatedAt)
    fmt.Printf("%s├─ %s [%s] (%s) Age: %v\n", 
              indent, info.ID, info.Type, status, age)
    
    for _, child := range children[info.ID] {
        printContextTree(child, children, depth+1)
    }
}

func main() {
    fmt.Println("=== Context Debugging and Monitoring ===")
    
    // Set up monitoring
    monitor := NewContextMonitor()
    logger := NewContextLogger()
    leakDetector := NewLeakDetector(3 * time.Second)
    
    // Create traced context hierarchy
    root := NewTracedContext(context.Background(), "root", monitor, logger)
    
    // Add timeout context
    timeoutCtx, cancel1 := root.WithTimeout(2 * time.Second)
    defer cancel1()
    
    // Add value context
    valueCtx := timeoutCtx.WithValue("userID", "user123")
    
    // Register with leak detector
    leakDetector.Register("root")
    leakDetector.Register("timeout")
    leakDetector.Register("value")
    
    // Simulate operations
    go func() {
        select {
        case <-timeoutCtx.Done():
            logger.Log(timeoutCtx.id, "TIMEOUT", "Context timed out")
        case <-time.After(1 * time.Second):
            logger.Log(timeoutCtx.id, "COMPLETED", "Operation completed before timeout")
        }
    }()
    
    // Create more contexts for demonstration
    for i := 0; i < 5; i++ {
        ctx, cancel := context.WithTimeout(context.Background(), time.Duration(i+1)*500*time.Millisecond)
        id := fmt.Sprintf("worker-%d", i)
        monitor.RegisterContext(id, "timeout", "")
        leakDetector.Register(id)
        
        go func(contextID string, c context.Context, cancelFunc context.CancelFunc) {
            defer func() {
                monitor.UnregisterContext(contextID)
                leakDetector.Unregister(contextID)
                cancelFunc()
            }()
            
            select {
            case <-time.After(2 * time.Second):
                logger.Log(contextID, "COMPLETED", "Work finished")
            case <-c.Done():
                logger.Log(contextID, "CANCELLED", "Work cancelled")
            }
        }(id, ctx, cancel)
    }
    
    // Monitoring loop
    for i := 0; i < 3; i++ {
        time.Sleep(1 * time.Second)
        
        // Print monitoring report
        monitor.PrintReport()
        
        // Check for leaks
        leaks := leakDetector.CheckLeaks()
        if len(leaks) > 0 {
            fmt.Printf("\nPotential context leaks detected: %v\n", leaks)
        } else {
            fmt.Println("\nNo context leaks detected")
        }
        
        // Visualize hierarchy
        if i == 1 {
            visualizeContextHierarchy(monitor)
        }
    }
    
    // Print final logs for root context
    fmt.Println("\n=== Root Context Log History ===")
    logs := logger.GetLogs("root")
    for _, log := range logs {
        fmt.Printf("%s: %s - %s\n", 
                  log.Timestamp.Format("15:04:05.000"), log.Event, log.Message)
    }
    
    // Final cleanup
    leakDetector.Unregister("root")
    leakDetector.Unregister("timeout")
    leakDetector.Unregister("value")
    monitor.UnregisterContext("root")
    
    // Final report
    fmt.Println("\n=== Final Statistics ===")
    monitor.PrintReport()
}
```

Context debugging and monitoring tools provide visibility into context  
usage patterns, help identify leaks and performance issues, and enable  
effective troubleshooting of complex concurrent applications.  

## Context best practices

Following established best practices ensures effective context usage and  
helps avoid common pitfalls in concurrent Go applications.  

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "net/http"
    "time"
)

// ✅ GOOD: Context as first parameter
func processUserRequest(ctx context.Context, userID string, action string) error {
    // Always accept context as the first parameter
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    
    fmt.Printf("Processing %s for user %s\n", action, userID)
    return nil
}

// ❌ BAD: Context not as first parameter
func badProcessUserRequest(userID string, ctx context.Context, action string) error {
    // Context should always be the first parameter
    return nil
}

// ✅ GOOD: Don't store context in structs
type UserService struct {
    // Don't store context in struct fields
    database Database
    cache    Cache
}

func (us *UserService) GetUser(ctx context.Context, userID string) (*User, error) {
    // Pass context through method parameters
    return us.database.FindUser(ctx, userID)
}

// ❌ BAD: Storing context in struct
type BadUserService struct {
    ctx      context.Context // Don't do this
    database Database
}

// ✅ GOOD: Use context.WithValue sparingly and with typed keys
type contextKey string

const (
    userIDContextKey    contextKey = "userID"
    requestIDContextKey contextKey = "requestID"
    traceIDContextKey   contextKey = "traceID"
)

func withUserID(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, userIDContextKey, userID)
}

func getUserID(ctx context.Context) (string, bool) {
    userID, ok := ctx.Value(userIDContextKey).(string)
    return userID, ok
}

// ❌ BAD: Using string keys directly
func badWithUserID(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, "userID", userID) // Prone to collisions
}

// ✅ GOOD: Check for cancellation regularly in loops
func processLargeDataset(ctx context.Context, data []int) error {
    for i, item := range data {
        // Check for cancellation every iteration or periodically
        select {
        case <-ctx.Done():
            return fmt.Errorf("processing cancelled at item %d: %w", i, ctx.Err())
        default:
        }
        
        // Simulate processing
        time.Sleep(10 * time.Millisecond)
        fmt.Printf("Processed item %d: %d\n", i, item)
    }
    return nil
}

// ❌ BAD: Not checking for cancellation
func badProcessLargeDataset(ctx context.Context, data []int) error {
    for _, item := range data {
        // This could run forever even if context is cancelled
        time.Sleep(10 * time.Millisecond)
        fmt.Printf("Processed item: %d\n", item)
    }
    return nil
}

// ✅ GOOD: Always call cancel to free resources
func goodResourceManagement(ctx context.Context) error {
    // Create child context with timeout
    childCtx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel() // Always call cancel to free resources
    
    return doWork(childCtx)
}

// ❌ BAD: Not calling cancel
func badResourceManagement(ctx context.Context) error {
    childCtx, _ := context.WithTimeout(ctx, 5*time.Second)
    // Missing defer cancel() - resource leak!
    
    return doWork(childCtx)
}

// ✅ GOOD: Use Background() only for top-level contexts
func main() {
    // Background context for main function
    ctx := context.Background()
    
    // Add request ID for tracing
    ctx = context.WithValue(ctx, requestIDContextKey, "req-123")
    
    handleRequest(ctx)
}

func handleRequest(ctx context.Context) {
    // Don't create new Background context, use the passed one
    processUserRequest(ctx, "user123", "update_profile")
}

// ❌ BAD: Creating Background in the middle of call chain
func badHandleRequest(ctx context.Context) {
    newCtx := context.Background() // Don't do this
    processUserRequest(newCtx, "user123", "update_profile")
}

// ✅ GOOD: Proper timeout handling
func httpClientExample(ctx context.Context, url string) (*http.Response, error) {
    // Create request with context
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    client := &http.Client{
        Timeout: 10 * time.Second, // Client-level timeout
    }
    
    return client.Do(req)
}

// ✅ GOOD: Context value validation
func getRequiredUserID(ctx context.Context) (string, error) {
    userID, ok := getUserID(ctx)
    if !ok {
        return "", errors.New("user ID not found in context")
    }
    
    if userID == "" {
        return "", errors.New("user ID is empty")
    }
    
    return userID, nil
}

// ✅ GOOD: Graceful degradation
func getCachedData(ctx context.Context, key string) ([]byte, error) {
    // Try cache first with shorter timeout
    cacheCtx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
    defer cancel()
    
    data, err := cache.Get(cacheCtx, key)
    if err == nil {
        return data, nil
    }
    
    // If cache fails or times out, fall back to database
    fmt.Printf("Cache miss for %s, falling back to database\n", key)
    return database.Get(ctx, key)
}

// ✅ GOOD: Error wrapping with context
func fetchUserProfile(ctx context.Context, userID string) (*UserProfile, error) {
    profile, err := database.GetProfile(ctx, userID)
    if err != nil {
        // Wrap errors with context for better debugging
        return nil, fmt.Errorf("failed to fetch profile for user %s: %w", userID, err)
    }
    
    return profile, nil
}

// ✅ GOOD: Proper context propagation in HTTP handlers
func userHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    
    // Add request-specific values
    requestID := generateRequestID()
    ctx = context.WithValue(ctx, requestIDContextKey, requestID)
    
    // Set response header
    w.Header().Set("X-Request-ID", requestID)
    
    // Add timeout for this request
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    
    // Process request
    user, err := fetchUser(ctx, "user123")
    if err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            http.Error(w, "Request timeout", http.StatusRequestTimeout)
        } else {
            http.Error(w, "Internal error", http.StatusInternalServerError)
        }
        return
    }
    
    // Return user data
    fmt.Fprintf(w, "User: %v", user)
}

// ✅ GOOD: Testing with context
func TestUserService_GetUser(t *testing.T) {
    // Use context in tests
    ctx := context.Background()
    
    service := &UserService{}
    
    // Test normal case
    user, err := service.GetUser(ctx, "test-user")
    if err != nil {
        t.Fatalf("Expected no error, got %v", err)
    }
    
    // Test timeout case
    timeoutCtx, cancel := context.WithTimeout(ctx, 1*time.Millisecond)
    defer cancel()
    
    _, err = service.GetUser(timeoutCtx, "test-user")
    if !errors.Is(err, context.DeadlineExceeded) {
        t.Errorf("Expected timeout error, got %v", err)
    }
}

// Supporting types and functions
type User struct{ ID, Name string }
type UserProfile struct{ UserID, Bio string }
type Database interface{ 
    FindUser(context.Context, string) (*User, error) 
    GetProfile(context.Context, string) (*UserProfile, error)
    Get(context.Context, string) ([]byte, error)
}
type Cache interface{ Get(context.Context, string) ([]byte, error) }

var (
    database Database
    cache    Cache
)

func doWork(ctx context.Context) error { return nil }
func fetchUser(ctx context.Context, userID string) (*User, error) { 
    return &User{ID: userID, Name: "Test User"}, nil 
}
func generateRequestID() string { return "req-123" }

// Testing framework mock
type T struct{}
func (t *T) Fatalf(format string, args ...interface{}) { fmt.Printf("FAIL: "+format+"\n", args...) }
func (t *T) Errorf(format string, args ...interface{}) { fmt.Printf("ERROR: "+format+"\n", args...) }

func main() {
    fmt.Println("=== Context Best Practices ===")
    
    // Demonstrate good practices
    ctx := context.Background()
    ctx = context.WithValue(ctx, requestIDContextKey, "demo-request")
    
    // Good resource management
    err := goodResourceManagement(ctx)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Good cancellation checking
    data := []int{1, 2, 3, 4, 5}
    err = processLargeDataset(ctx, data)
    if err != nil {
        fmt.Printf("Processing error: %v\n", err)
    }
    
    // Good value access
    if requestID, ok := ctx.Value(requestIDContextKey).(string); ok {
        fmt.Printf("Request ID: %s\n", requestID)
    }
    
    // Demonstrate timeout with proper cleanup
    timeoutCtx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
    defer cancel()
    
    select {
    case <-time.After(200 * time.Millisecond):
        fmt.Println("Operation completed")
    case <-timeoutCtx.Done():
        fmt.Printf("Operation timed out: %v\n", timeoutCtx.Err())
    }
    
    fmt.Println("\nBest practices demonstrated:")
    fmt.Println("✅ Context as first parameter")
    fmt.Println("✅ Typed context keys")
    fmt.Println("✅ Regular cancellation checks")
    fmt.Println("✅ Proper resource cleanup")
    fmt.Println("✅ Error wrapping")
    fmt.Println("✅ Value validation")
}
```

Context best practices ensure proper resource management, effective  
cancellation handling, and maintainable code structure in concurrent  
Go applications.  

## Context anti-patterns

Understanding common context anti-patterns helps avoid mistakes that can  
lead to bugs, resource leaks, and poor application performance.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// ❌ ANTI-PATTERN 1: Storing context in structs
type BadService struct {
    ctx context.Context // DON'T DO THIS
    mu  sync.Mutex
}

func (bs *BadService) DoWork() error {
    // Using stored context is problematic
    return someOperation(bs.ctx)
}

// ✅ CORRECT: Pass context as parameter
type GoodService struct {
    mu sync.Mutex
}

func (gs *GoodService) DoWork(ctx context.Context) error {
    return someOperation(ctx)
}

// ❌ ANTI-PATTERN 2: Ignoring context cancellation
func badLongRunningOperation(ctx context.Context) error {
    // This ignores context cancellation
    for i := 0; i < 1000000; i++ {
        // Expensive operation without checking ctx.Done()
        time.Sleep(1 * time.Millisecond)
    }
    return nil
}

// ✅ CORRECT: Check context regularly
func goodLongRunningOperation(ctx context.Context) error {
    for i := 0; i < 1000000; i++ {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
        }
        
        // Expensive operation
        time.Sleep(1 * time.Millisecond)
        
        // Check every 100 iterations for performance
        if i%100 == 0 {
            select {
            case <-ctx.Done():
                return ctx.Err()
            default:
            }
        }
    }
    return nil
}

// ❌ ANTI-PATTERN 3: Creating background context in chain
func badMiddlewarePattern(ctx context.Context) error {
    // DON'T create new background context
    newCtx := context.Background()
    return processRequest(newCtx)
}

// ✅ CORRECT: Propagate existing context
func goodMiddlewarePattern(ctx context.Context) error {
    // Use the existing context
    return processRequest(ctx)
}

// ❌ ANTI-PATTERN 4: Using context for optional parameters
func badFunctionWithOptions(ctx context.Context, required string) error {
    // DON'T use context for regular function parameters
    optional := ctx.Value("optional").(string)
    return fmt.Errorf("processing %s with %s", required, optional)
}

// ✅ CORRECT: Use proper function parameters or options pattern
type Options struct {
    Optional string
    Timeout  time.Duration
}

func goodFunctionWithOptions(ctx context.Context, required string, opts Options) error {
    return fmt.Errorf("processing %s with %s", required, opts.Optional)
}

// ❌ ANTI-PATTERN 5: Not calling cancel function
func badResourceLeak() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    // Missing defer cancel() - RESOURCE LEAK!
    
    go someOperation(ctx)
    // cancel is never called
}

// ✅ CORRECT: Always call cancel
func goodResourceManagement() {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel() // Always call cancel
    
    go someOperation(ctx)
}

// ❌ ANTI-PATTERN 6: Nil context
func badNilContext() {
    // DON'T pass nil context
    processRequest(nil)
}

// ✅ CORRECT: Use context.TODO() if you don't have a context
func goodContextUsage() {
    // Use TODO when you don't have a context yet
    processRequest(context.TODO())
}

// ❌ ANTI-PATTERN 7: Context in wrong parameter position
func badParameterOrder(userID string, ctx context.Context, action string) error {
    // Context should be first parameter
    return nil
}

// ✅ CORRECT: Context as first parameter
func goodParameterOrder(ctx context.Context, userID string, action string) error {
    return nil
}

// ❌ ANTI-PATTERN 8: Overusing context values
func badContextValues(ctx context.Context) error {
    // DON'T overload context with too many values
    ctx = context.WithValue(ctx, "param1", "value1")
    ctx = context.WithValue(ctx, "param2", "value2")
    ctx = context.WithValue(ctx, "param3", "value3")
    ctx = context.WithValue(ctx, "param4", "value4")
    ctx = context.WithValue(ctx, "param5", "value5")
    
    return someOperation(ctx)
}

// ✅ CORRECT: Use context values sparingly
func goodContextValues(ctx context.Context, params ProcessParams) error {
    // Use a struct for multiple parameters
    // Only use context for request-scoped data like trace IDs
    ctx = context.WithValue(ctx, "traceID", "trace-123")
    return someOperationWithParams(ctx, params)
}

type ProcessParams struct {
    Param1, Param2, Param3, Param4, Param5 string
}

// ❌ ANTI-PATTERN 9: Context timeout too long/short
func badTimeoutUsage() {
    // Too long - ties up resources
    ctx1, cancel1 := context.WithTimeout(context.Background(), 24*time.Hour)
    defer cancel1()
    quickOperation(ctx1) // Should use much shorter timeout
    
    // Too short - operation will likely fail
    ctx2, cancel2 := context.WithTimeout(context.Background(), 1*time.Nanosecond)
    defer cancel2()
    databaseQuery(ctx2) // Unrealistic timeout
}

// ✅ CORRECT: Appropriate timeouts
func goodTimeoutUsage() {
    // Reasonable timeout for quick operation
    ctx1, cancel1 := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel1()
    quickOperation(ctx1)
    
    // Appropriate timeout for database query
    ctx2, cancel2 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel2()
    databaseQuery(ctx2)
}

// ❌ ANTI-PATTERN 10: Modifying parent context
func badContextModification(ctx context.Context) context.Context {
    // DON'T try to "modify" the parent context
    // This doesn't actually modify anything
    context.WithValue(ctx, "key", "value")
    return ctx // Returns original context, not modified one
}

// ✅ CORRECT: Create child context
func goodContextModification(ctx context.Context) context.Context {
    // Create and return child context
    return context.WithValue(ctx, "key", "value")
}

// ❌ ANTI-PATTERN 11: Catching all errors as context errors
func badErrorHandling(ctx context.Context) error {
    err := someFailingOperation(ctx)
    if err != nil {
        // DON'T assume all errors are context errors
        if err == context.Canceled {
            return fmt.Errorf("operation was cancelled")
        }
        return err
    }
    return nil
}

// ✅ CORRECT: Proper error type checking
func goodErrorHandling(ctx context.Context) error {
    err := someFailingOperation(ctx)
    if err != nil {
        // Check specific error types
        switch {
        case ctx.Err() == context.Canceled:
            return fmt.Errorf("operation cancelled by user")
        case ctx.Err() == context.DeadlineExceeded:
            return fmt.Errorf("operation timed out")
        default:
            return fmt.Errorf("operation failed: %w", err)
        }
    }
    return nil
}

// ❌ ANTI-PATTERN 12: Context leak in goroutines
func badGoroutinePattern(ctx context.Context) {
    // This can leak goroutines
    for i := 0; i < 100; i++ {
        go func(id int) {
            // No context checking - goroutine may run forever
            for {
                time.Sleep(1 * time.Second)
                fmt.Printf("Worker %d working...\n", id)
            }
        }(i)
    }
}

// ✅ CORRECT: Goroutines that respect context
func goodGoroutinePattern(ctx context.Context) {
    var wg sync.WaitGroup
    
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for {
                select {
                case <-ctx.Done():
                    fmt.Printf("Worker %d stopping\n", id)
                    return
                default:
                    time.Sleep(1 * time.Second)
                    fmt.Printf("Worker %d working...\n", id)
                }
            }
        }(i)
    }
    
    wg.Wait()
}

// Supporting functions
func someOperation(ctx context.Context) error { return nil }
func someOperationWithParams(ctx context.Context, params ProcessParams) error { return nil }
func processRequest(ctx context.Context) error { return nil }
func quickOperation(ctx context.Context) error { 
    time.Sleep(10 * time.Millisecond)
    return nil 
}
func databaseQuery(ctx context.Context) error { 
    time.Sleep(100 * time.Millisecond)
    return nil 
}
func someFailingOperation(ctx context.Context) error { 
    return fmt.Errorf("simulated failure")
}

func demonstrateAntiPattern(name string, badFunc, goodFunc func()) {
    fmt.Printf("\n=== %s ===\n", name)
    
    fmt.Println("❌ BAD:")
    badFunc()
    
    fmt.Println("✅ GOOD:")
    goodFunc()
}

func main() {
    fmt.Println("=== Context Anti-Patterns ===")
    
    demonstrateAntiPattern("Resource Management", 
        func() {
            fmt.Println("Creating context without calling cancel - RESOURCE LEAK!")
            badResourceLeak()
        },
        func() {
            fmt.Println("Creating context with proper cleanup")
            goodResourceManagement()
        })
    
    demonstrateAntiPattern("Context Parameters", 
        func() {
            fmt.Println("Using nil context")
            badNilContext()
        },
        func() {
            fmt.Println("Using context.TODO() when no context available")
            goodContextUsage()
        })
    
    demonstrateAntiPattern("Timeout Usage", 
        func() {
            fmt.Println("Using inappropriate timeouts")
            badTimeoutUsage()
        },
        func() {
            fmt.Println("Using appropriate timeouts")
            goodTimeoutUsage()
        })
    
    demonstrateAntiPattern("Goroutine Management", 
        func() {
            fmt.Println("Starting goroutines without context checking (would leak)")
            // badGoroutinePattern() - commented out to avoid actual leak
        },
        func() {
            fmt.Println("Starting goroutines with proper context handling")
            ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
            defer cancel()
            goodGoroutinePattern(ctx)
        })
    
    fmt.Println("\n=== Summary of Anti-Patterns to Avoid ===")
    fmt.Println("❌ Don't store context in structs")
    fmt.Println("❌ Don't ignore context cancellation in loops")
    fmt.Println("❌ Don't create background context mid-chain")
    fmt.Println("❌ Don't use context for optional parameters")
    fmt.Println("❌ Don't forget to call cancel functions")
    fmt.Println("❌ Don't pass nil context")
    fmt.Println("❌ Don't put context in wrong parameter position")
    fmt.Println("❌ Don't overuse context values")
    fmt.Println("❌ Don't use inappropriate timeouts")
    fmt.Println("❌ Don't ignore proper error handling")
    fmt.Println("❌ Don't leak goroutines without context checking")
}
```

Context anti-patterns demonstrate common mistakes to avoid, helping  
developers write more robust and maintainable concurrent Go applications  
with proper context usage.  

## Cross-service context propagation

Context propagation across service boundaries enables distributed tracing,  
timeout coordination, and request correlation in microservices architectures.  

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "strconv"
    "time"
)

// Context metadata for cross-service propagation
type ContextMetadata struct {
    TraceID    string            `json:"trace_id"`
    SpanID     string            `json:"span_id"`
    ParentID   string            `json:"parent_id,omitempty"`
    Baggage    map[string]string `json:"baggage,omitempty"`
    Deadline   *time.Time        `json:"deadline,omitempty"`
    TimeoutMS  int64             `json:"timeout_ms,omitempty"`
}

// Context propagator handles context marshaling/unmarshaling
type ContextPropagator struct {
    traceHeader   string
    timeoutHeader string
    baggageHeader string
}

func NewContextPropagator() *ContextPropagator {
    return &ContextPropagator{
        traceHeader:   "X-Trace-Context",
        timeoutHeader: "X-Timeout-MS",
        baggageHeader: "X-Context-Baggage",
    }
}

func (cp *ContextPropagator) InjectHTTP(ctx context.Context, req *http.Request) error {
    metadata := cp.extractMetadata(ctx)
    
    // Inject trace context
    if metadata.TraceID != "" {
        traceData, err := json.Marshal(metadata)
        if err != nil {
            return fmt.Errorf("failed to marshal trace context: %w", err)
        }
        req.Header.Set(cp.traceHeader, string(traceData))
    }
    
    // Inject timeout
    if deadline, ok := ctx.Deadline(); ok {
        remaining := time.Until(deadline)
        if remaining > 0 {
            req.Header.Set(cp.timeoutHeader, strconv.FormatInt(remaining.Milliseconds(), 10))
        }
    }
    
    // Inject baggage
    if len(metadata.Baggage) > 0 {
        baggageData, err := json.Marshal(metadata.Baggage)
        if err != nil {
            return fmt.Errorf("failed to marshal baggage: %w", err)
        }
        req.Header.Set(cp.baggageHeader, string(baggageData))
    }
    
    return nil
}

func (cp *ContextPropagator) ExtractHTTP(ctx context.Context, req *http.Request) (context.Context, error) {
    // Extract trace context
    if traceData := req.Header.Get(cp.traceHeader); traceData != "" {
        var metadata ContextMetadata
        if err := json.Unmarshal([]byte(traceData), &metadata); err != nil {
            return ctx, fmt.Errorf("failed to unmarshal trace context: %w", err)
        }
        
        ctx = cp.injectMetadata(ctx, metadata)
    }
    
    // Extract timeout
    if timeoutStr := req.Header.Get(cp.timeoutHeader); timeoutStr != "" {
        timeoutMS, err := strconv.ParseInt(timeoutStr, 10, 64)
        if err == nil && timeoutMS > 0 {
            timeout := time.Duration(timeoutMS) * time.Millisecond
            ctx, _ = context.WithTimeout(ctx, timeout)
        }
    }
    
    // Extract baggage
    if baggageData := req.Header.Get(cp.baggageHeader); baggageData != "" {
        var baggage map[string]string
        if err := json.Unmarshal([]byte(baggageData), &baggage); err == nil {
            for key, value := range baggage {
                ctx = context.WithValue(ctx, key, value)
            }
        }
    }
    
    return ctx, nil
}

func (cp *ContextPropagator) extractMetadata(ctx context.Context) ContextMetadata {
    metadata := ContextMetadata{
        Baggage: make(map[string]string),
    }
    
    // Extract trace information
    if traceID, ok := ctx.Value("traceID").(string); ok {
        metadata.TraceID = traceID
    }
    if spanID, ok := ctx.Value("spanID").(string); ok {
        metadata.SpanID = spanID
    }
    if parentID, ok := ctx.Value("parentID").(string); ok {
        metadata.ParentID = parentID
    }
    
    // Extract deadline
    if deadline, ok := ctx.Deadline(); ok {
        metadata.Deadline = &deadline
    }
    
    // Extract baggage (common context values)
    baggageKeys := []string{"userID", "sessionID", "clientIP", "userAgent"}
    for _, key := range baggageKeys {
        if value, ok := ctx.Value(key).(string); ok {
            metadata.Baggage[key] = value
        }
    }
    
    return metadata
}

func (cp *ContextPropagator) injectMetadata(ctx context.Context, metadata ContextMetadata) context.Context {
    if metadata.TraceID != "" {
        ctx = context.WithValue(ctx, "traceID", metadata.TraceID)
    }
    if metadata.SpanID != "" {
        ctx = context.WithValue(ctx, "spanID", metadata.SpanID)
    }
    if metadata.ParentID != "" {
        ctx = context.WithValue(ctx, "parentID", metadata.ParentID)
    }
    
    for key, value := range metadata.Baggage {
        ctx = context.WithValue(ctx, key, value)
    }
    
    return ctx
}

// Service client with context propagation
type ServiceClient struct {
    baseURL    string
    httpClient *http.Client
    propagator *ContextPropagator
}

func NewServiceClient(baseURL string) *ServiceClient {
    return &ServiceClient{
        baseURL: baseURL,
        httpClient: &http.Client{
            Timeout: 30 * time.Second,
        },
        propagator: NewContextPropagator(),
    }
}

func (sc *ServiceClient) CallService(ctx context.Context, endpoint string, data interface{}) (map[string]interface{}, error) {
    // Create request
    req, err := http.NewRequestWithContext(ctx, "POST", sc.baseURL+endpoint, nil)
    if err != nil {
        return nil, err
    }
    
    // Inject context into request headers
    if err := sc.propagator.InjectHTTP(ctx, req); err != nil {
        return nil, fmt.Errorf("failed to inject context: %w", err)
    }
    
    fmt.Printf("Making request to %s with trace ID: %v\n", 
              endpoint, ctx.Value("traceID"))
    
    // Simulate HTTP call
    time.Sleep(100 * time.Millisecond)
    
    // Simulate response
    response := map[string]interface{}{
        "status": "success",
        "data":   data,
        "trace_id": ctx.Value("traceID"),
    }
    
    return response, nil
}

// HTTP handler with context extraction
func serviceHandler(propagator *ContextPropagator) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Extract context from incoming request
        ctx, err := propagator.ExtractHTTP(r.Context(), r)
        if err != nil {
            http.Error(w, fmt.Sprintf("Failed to extract context: %v", err), 
                      http.StatusBadRequest)
            return
        }
        
        // Log received context
        fmt.Printf("Service received request with trace ID: %v\n", 
                  ctx.Value("traceID"))
        
        // Process request with propagated context
        result := processServiceRequest(ctx, r.URL.Path)
        
        // Return response
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(result)
    }
}

func processServiceRequest(ctx context.Context, path string) map[string]interface{} {
    // Simulate processing time
    select {
    case <-time.After(50 * time.Millisecond):
        return map[string]interface{}{
            "path":     path,
            "trace_id": ctx.Value("traceID"),
            "user_id":  ctx.Value("userID"),
            "processed_at": time.Now().Format(time.RFC3339),
        }
    case <-ctx.Done():
        return map[string]interface{}{
            "error":    "request cancelled",
            "trace_id": ctx.Value("traceID"),
        }
    }
}

// Distributed trace context
type DistributedTrace struct {
    TraceID  string
    Services []ServiceCall
}

type ServiceCall struct {
    Service   string
    StartTime time.Time
    EndTime   time.Time
    Duration  time.Duration
    Error     error
}

func (dt *DistributedTrace) AddServiceCall(service string, start, end time.Time, err error) {
    call := ServiceCall{
        Service:   service,
        StartTime: start,
        EndTime:   end,
        Duration:  end.Sub(start),
        Error:     err,
    }
    dt.Services = append(dt.Services, call)
}

func (dt *DistributedTrace) PrintTrace() {
    fmt.Printf("\n=== Distributed Trace: %s ===\n", dt.TraceID)
    for i, call := range dt.Services {
        status := "SUCCESS"
        if call.Error != nil {
            status = fmt.Sprintf("ERROR: %v", call.Error)
        }
        fmt.Printf("%d. %s: %v (%s)\n", 
                  i+1, call.Service, call.Duration, status)
    }
    
    total := dt.Services[len(dt.Services)-1].EndTime.Sub(dt.Services[0].StartTime)
    fmt.Printf("Total duration: %v\n", total)
}

// Orchestrator service that calls multiple services
func orchestrateRequest(ctx context.Context, client *ServiceClient) (*DistributedTrace, error) {
    traceID := ctx.Value("traceID").(string)
    trace := &DistributedTrace{TraceID: traceID}
    
    services := []string{"/auth", "/user", "/billing", "/notification"}
    
    for _, service := range services {
        start := time.Now()
        
        // Create child span for this service call
        serviceCtx := context.WithValue(ctx, "spanID", fmt.Sprintf("span-%d", len(trace.Services)))
        serviceCtx = context.WithValue(serviceCtx, "parentID", ctx.Value("spanID"))
        
        _, err := client.CallService(serviceCtx, service, map[string]string{
            "request_id": fmt.Sprintf("req-%d", len(trace.Services)),
        })
        
        end := time.Now()
        trace.AddServiceCall(service, start, end, err)
        
        if err != nil {
            return trace, fmt.Errorf("service %s failed: %w", service, err)
        }
        
        // Check for cancellation between service calls
        select {
        case <-ctx.Done():
            return trace, ctx.Err()
        default:
        }
    }
    
    return trace, nil
}

func main() {
    fmt.Println("=== Cross-Service Context Propagation ===")
    
    // Create root context with trace information
    ctx := context.Background()
    ctx = context.WithValue(ctx, "traceID", "trace-12345")
    ctx = context.WithValue(ctx, "spanID", "span-root")
    ctx = context.WithValue(ctx, "userID", "user-999")
    ctx = context.WithValue(ctx, "sessionID", "session-abc")
    
    // Add timeout for the entire operation
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()
    
    // Create service client
    client := NewServiceClient("https://api.example.com")
    
    fmt.Println("Starting distributed request...")
    
    // Orchestrate cross-service call
    trace, err := orchestrateRequest(ctx, client)
    if err != nil {
        fmt.Printf("Orchestration failed: %v\n", err)
    }
    
    // Print distributed trace
    trace.PrintTrace()
    
    // Demonstrate context extraction/injection
    fmt.Println("\n=== Context Propagation Demo ===")
    
    propagator := NewContextPropagator()
    
    // Create mock HTTP request
    req, _ := http.NewRequest("GET", "/test", nil)
    
    // Inject context into request
    err = propagator.InjectHTTP(ctx, req)
    if err != nil {
        fmt.Printf("Injection error: %v\n", err)
    } else {
        fmt.Println("Context injected into HTTP headers:")
        for name, values := range req.Header {
            fmt.Printf("  %s: %s\n", name, values[0])
        }
    }
    
    // Extract context from request
    extractedCtx, err := propagator.ExtractHTTP(context.Background(), req)
    if err != nil {
        fmt.Printf("Extraction error: %v\n", err)
    } else {
        fmt.Println("\nExtracted context values:")
        fmt.Printf("  TraceID: %v\n", extractedCtx.Value("traceID"))
        fmt.Printf("  UserID: %v\n", extractedCtx.Value("userID"))
        fmt.Printf("  SessionID: %v\n", extractedCtx.Value("sessionID"))
    }
    
    fmt.Println("\nCross-service context propagation completed successfully!")
}
```

Cross-service context propagation enables distributed tracing, timeout  
coordination, and metadata flow across microservices boundaries, providing  
observability and consistency in distributed systems.  

## Context conclusion

The Context package represents one of Go's most powerful tools for building  
robust, scalable, and maintainable concurrent applications.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

// Comprehensive example demonstrating key context principles
func demonstrateContextPrinciples() {
    fmt.Println("=== Context Package - Key Principles ===")
    
    // 1. Cancellation propagation
    fmt.Println("\n1. Cancellation Propagation:")
    parentCtx, parentCancel := context.WithCancel(context.Background())
    childCtx, childCancel := context.WithTimeout(parentCtx, 2*time.Second)
    defer parentCancel()
    defer childCancel()
    
    go func() {
        select {
        case <-childCtx.Done():
            fmt.Printf("   Child cancelled: %v\n", childCtx.Err())
        case <-time.After(3*time.Second):
            fmt.Println("   Child completed normally")
        }
    }()
    
    time.Sleep(500*time.Millisecond)
    parentCancel() // This cancels child too
    time.Sleep(100*time.Millisecond)
    
    // 2. Deadline management
    fmt.Println("\n2. Deadline Management:")
    deadline := time.Now().Add(1*time.Second)
    deadlineCtx, cancel := context.WithDeadline(context.Background(), deadline)
    defer cancel()
    
    if d, ok := deadlineCtx.Deadline(); ok {
        fmt.Printf("   Deadline set for: %v\n", d.Format("15:04:05.000"))
        fmt.Printf("   Time remaining: %v\n", time.Until(d))
    }
    
    // 3. Value propagation
    fmt.Println("\n3. Value Propagation:")
    valueCtx := context.WithValue(context.Background(), "requestID", "req-123")
    valueCtx = context.WithValue(valueCtx, "userID", "user-456")
    
    if reqID := valueCtx.Value("requestID"); reqID != nil {
        fmt.Printf("   Request ID: %v\n", reqID)
    }
    if userID := valueCtx.Value("userID"); userID != nil {
        fmt.Printf("   User ID: %v\n", userID)
    }
}

// Best practices summary
func contextBestPracticesSummary() {
    fmt.Println("\n=== Context Best Practices Summary ===")
    
    practices := []string{
        "✅ Always pass context as the first parameter",
        "✅ Use context.Background() for main/top-level functions",
        "✅ Use context.TODO() when unsure which context to use",
        "✅ Call cancel() functions to free resources",
        "✅ Check ctx.Done() regularly in loops",
        "✅ Use typed keys for context values",
        "✅ Keep context values minimal and request-scoped",
        "✅ Propagate context through call chains",
        "✅ Set appropriate timeouts for operations",
        "✅ Handle context errors appropriately",
    }
    
    for _, practice := range practices {
        fmt.Printf("   %s\n", practice)
    }
}

// Common use cases
func contextUseCases() {
    fmt.Println("\n=== Common Context Use Cases ===")
    
    useCases := map[string]string{
        "HTTP Servers": "Request lifecycle, client disconnection handling",
        "Database Operations": "Query timeouts, transaction cancellation",
        "gRPC Services": "Request deadlines, metadata propagation",
        "Background Jobs": "Graceful shutdown, job cancellation",
        "Caching": "Cache operation timeouts, background refresh",
        "Microservices": "Distributed tracing, cross-service timeouts",
        "Worker Pools": "Worker lifecycle management, load balancing",
        "File Operations": "I/O timeouts, operation cancellation",
        "API Clients": "Request timeouts, retry logic",
        "Testing": "Test timeouts, resource cleanup",
    }
    
    for useCase, description := range useCases {
        fmt.Printf("   %-18s: %s\n", useCase, description)
    }
}

// Performance considerations
func contextPerformanceConsiderations() {
    fmt.Println("\n=== Performance Considerations ===")
    
    considerations := []string{
        "• Context creation has minimal overhead for most applications",
        "• Avoid excessive context value nesting",
        "• Use context pools for high-frequency operations if needed",
        "• Check cancellation appropriately (not too often, not too rarely)",
        "• Prefer struct parameters over many context values",
        "• Be mindful of goroutine lifecycle and context propagation",
        "• Use appropriate timeout values for different operations",
        "• Consider batch operations for high-throughput scenarios",
    }
    
    for _, consideration := range considerations {
        fmt.Printf("   %s\n", consideration)
    }
}

// Advanced patterns recap
func contextAdvancedPatterns() {
    fmt.Println("\n=== Advanced Context Patterns ===")
    
    patterns := map[string]string{
        "Custom Contexts": "Specialized context implementations",
        "Context Pools": "Reusable contexts for performance",
        "Circuit Breakers": "Failure handling with context",
        "Fan-out/Fan-in": "Concurrent processing coordination",
        "Pipeline Processing": "Multi-stage data processing",
        "Graceful Shutdown": "Coordinated service shutdown",
        "Distributed Tracing": "Cross-service request tracking",
        "Middleware Chains": "Request processing pipelines",
        "Retry Logic": "Robust operation patterns",
        "Resource Management": "Lifecycle and cleanup coordination",
    }
    
    for pattern, description := range patterns {
        fmt.Printf("   %-20s: %s\n", pattern, description)
    }
}

// Future of context
func contextFuture() {
    fmt.Println("\n=== Context Package Evolution ===")
    
    fmt.Println("   The Context package continues to be central to Go's")
    fmt.Println("   concurrency model and ecosystem:")
    fmt.Println()
    fmt.Println("   • Standard library integration continues to expand")
    fmt.Println("   • Third-party libraries increasingly adopt context patterns")
    fmt.Println("   • Observability tools leverage context for tracing")
    fmt.Println("   • Cloud-native applications rely on context for coordination")
    fmt.Println("   • Performance optimizations continue to be refined")
    fmt.Println("   • Best practices evolve with real-world usage")
}

// Final example combining multiple concepts
func comprehensiveContextExample() {
    fmt.Println("\n=== Comprehensive Context Example ===")
    
    // Create root context with request information
    ctx := context.Background()
    ctx = context.WithValue(ctx, "requestID", "final-demo")
    ctx = context.WithValue(ctx, "userID", "demo-user")
    
    // Add timeout for the entire operation
    ctx, cancel := context.WithTimeout(ctx, 3*time.Second)
    defer cancel()
    
    // Simulate multi-stage operation
    operations := []struct {
        name     string
        duration time.Duration
    }{
        {"Authentication", 200 * time.Millisecond},
        {"Authorization", 150 * time.Millisecond},
        {"Data Processing", 800 * time.Millisecond},
        {"Response Generation", 100 * time.Millisecond},
    }
    
    fmt.Printf("   Starting request: %v\n", ctx.Value("requestID"))
    
    for i, op := range operations {
        // Create operation-specific context
        opCtx, opCancel := context.WithTimeout(ctx, op.duration+100*time.Millisecond)
        
        select {
        case <-time.After(op.duration):
            fmt.Printf("   %d. %s: completed (%v)\n", i+1, op.name, op.duration)
        case <-opCtx.Done():
            fmt.Printf("   %d. %s: failed (%v)\n", i+1, op.name, opCtx.Err())
            opCancel()
            return
        }
        
        opCancel()
        
        // Check parent context between operations
        select {
        case <-ctx.Done():
            fmt.Printf("   Request cancelled after %s\n", op.name)
            return
        default:
        }
    }
    
    fmt.Printf("   Request completed successfully: %v\n", ctx.Value("requestID"))
}

func main() {
    fmt.Println("# Context Package - Comprehensive Overview")
    fmt.Println()
    fmt.Println("The Context package in Go provides a powerful and elegant solution")
    fmt.Println("for managing cancellation, timeouts, and request-scoped values in")
    fmt.Println("concurrent applications. Through 30 comprehensive examples, we've")
    fmt.Println("explored its capabilities from basic usage to advanced patterns.")
    
    demonstrateContextPrinciples()
    contextBestPracticesSummary()
    contextUseCases()
    contextPerformanceConsiderations()
    contextAdvancedPatterns()
    contextFuture()
    comprehensiveContextExample()
    
    fmt.Println("\n=== Final Thoughts ===")
    fmt.Println()
    fmt.Println("   Context is more than just a cancellation mechanism - it's a")
    fmt.Println("   fundamental building block for robust concurrent applications.")
    fmt.Println("   Mastering context patterns enables developers to build:")
    fmt.Println()
    fmt.Println("   • Responsive applications that handle timeouts gracefully")
    fmt.Println("   • Scalable services with proper resource management")
    fmt.Println("   • Observable systems with distributed tracing")
    fmt.Println("   • Resilient architectures with coordinated cancellation")
    fmt.Println("   • Maintainable codebases with clear separation of concerns")
    fmt.Println()
    fmt.Println("   The journey through these 30 examples demonstrates that context")
    fmt.Println("   is essential for any serious Go application. From simple HTTP")
    fmt.Println("   servers to complex distributed systems, context provides the")
    fmt.Println("   foundation for building software that can handle the challenges")
    fmt.Println("   of modern concurrent and distributed computing.")
    fmt.Println()
    fmt.Println("   Continue exploring, experimenting, and applying these patterns")
    fmt.Println("   in your own applications. The Context package will serve you")
    fmt.Println("   well in building the next generation of Go applications.")
}
```

The Context package conclusion demonstrates the comprehensive nature of  
context in Go, summarizing key principles, best practices, use cases, and  
advanced patterns covered in these 30 examples for building robust  
concurrent applications.  

