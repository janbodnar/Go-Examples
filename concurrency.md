# Concurrency

Go's concurrency model is built around the concept of communicating sequential processes  
(CSP), which emphasizes the use of goroutines and channels for concurrent programming.  
This approach makes it easier to write correct concurrent programs by avoiding the  
complexities of shared memory and locks that plague other languages.

Goroutines are lightweight threads managed by the Go runtime. They are much cheaper  
than operating system threads, allowing you to create thousands or even millions of  
them without significant overhead. Goroutines are multiplexed onto a smaller number  
of OS threads, and the Go scheduler handles the coordination automatically.

Channels are the primary mechanism for communication between goroutines. They provide  
a type-safe way to pass data between concurrent operations while ensuring proper  
synchronization. Channels can be buffered or unbuffered, and they follow the principle  
of "Don't communicate by sharing memory; share memory by communicating."

The select statement is Go's multiplexer for channel operations. It allows a goroutine  
to wait on multiple channel operations simultaneously, making it possible to implement  
complex coordination patterns, timeouts, and non-blocking operations with elegant code.

Go also provides traditional synchronization primitives through the sync package,  
including mutexes, wait groups, and atomic operations. These tools complement channels  
and are often used for protecting shared state or coordinating groups of goroutines.

Context is another crucial component for managing goroutine lifecycles, especially  
in server applications. The context package provides a way to carry deadlines,  
cancellation signals, and request-scoped values across API boundaries and between  
processes, making it easier to build robust concurrent applications.

## Basic goroutine

Goroutines are launched with the `go` keyword and enable concurrent execution  
of functions alongside the main program execution.  

```go
package main

import (
    "fmt"
    "time"
)

func sayHello(name string) {
    for i := 0; i < 3; i++ {
        fmt.Printf("Hello %s! (%d)\n", name, i+1)
        time.Sleep(100 * time.Millisecond)
    }
}

func main() {
    // Launch a goroutine
    go sayHello("Alice")
    
    // Main goroutine continues
    for i := 0; i < 3; i++ {
        fmt.Printf("Main function (%d)\n", i+1)
        time.Sleep(150 * time.Millisecond)
    }
    
    // Wait for goroutine to potentially complete
    time.Sleep(500 * time.Millisecond)
}
```

This example demonstrates concurrent execution where the `sayHello` function runs  
alongside the main function. Both execute simultaneously, showing how goroutines  
enable parallel processing of independent tasks.  

## Anonymous goroutines

Anonymous functions can be launched as goroutines directly, providing a compact  
way to create concurrent operations without defining separate functions.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Anonymous goroutine with closure
    message := "Hello from goroutine"
    
    go func() {
        for i := 0; i < 3; i++ {
            fmt.Printf("%s: %d\n", message, i)
            time.Sleep(100 * time.Millisecond)
        }
    }()
    
    // Another anonymous goroutine with parameters
    go func(name string, count int) {
        for i := 0; i < count; i++ {
            fmt.Printf("Greetings %s! (%d/%d)\n", name, i+1, count)
            time.Sleep(80 * time.Millisecond)
        }
    }("Bob", 4)
    
    // Wait for goroutines to complete
    time.Sleep(500 * time.Millisecond)
}
```

Anonymous goroutines are useful for short-lived concurrent tasks and can capture  
variables from their enclosing scope. Parameters prevent common closure pitfalls  
by ensuring values are captured at goroutine creation time.  

## Multiple goroutines

Creating multiple goroutines demonstrates how Go can handle many concurrent  
operations efficiently with minimal resource overhead.  

```go
package main

import (
    "fmt"
    "time"
)

func worker(id int, duration time.Duration) {
    fmt.Printf("Worker %d starting\n", id)
    time.Sleep(duration)
    fmt.Printf("Worker %d finished\n", id)
}

func main() {
    fmt.Println("Launching multiple goroutines...")
    
    // Launch 5 goroutines with different durations
    durations := []time.Duration{
        100 * time.Millisecond,
        200 * time.Millisecond,
        50 * time.Millisecond,
        300 * time.Millisecond,
        150 * time.Millisecond,
    }
    
    for i, duration := range durations {
        go worker(i+1, duration)
    }
    
    // Also launch some immediate workers
    for i := 0; i < 3; i++ {
        go func(workerID int) {
            fmt.Printf("Quick worker %d executing\n", workerID)
        }(i + 6)
    }
    
    // Wait for all goroutines to complete
    time.Sleep(400 * time.Millisecond)
    fmt.Println("All workers completed")
}
```

Multiple goroutines execute concurrently, and their completion order depends on  
their individual execution times. The Go runtime efficiently schedules all  
goroutines across available processor cores.  

## Goroutine with return values problem

Goroutines cannot return values directly, demonstrating a common challenge  
that leads to the need for channels for inter-goroutine communication.  

```go
package main

import (
    "fmt"
    "time"
)

func calculate(n int) int {
    time.Sleep(100 * time.Millisecond) // Simulate work
    return n * n
}

func main() {
    fmt.Println("Problem: Goroutines cannot return values directly")
    
    // This won't work - goroutines don't return values
    // result := go calculate(5)  // Compilation error
    
    // We can launch the goroutine, but lose the result
    go func() {
        result := calculate(5)
        // This result is lost - no way to get it back to main
        fmt.Printf("Calculation result (lost): %d\n", result)
    }()
    
    // Main function continues without access to the result
    fmt.Println("Main function cannot access goroutine results")
    
    // The need for synchronization is demonstrated here
    var results []int // We want to collect results here
    
    for i := 1; i <= 3; i++ {
        go func(n int) {
            result := calculate(n)
            // Problem: concurrent access to results slice is unsafe
            // results = append(results, result) // Race condition!
            fmt.Printf("Calculated %d^2 = %d (not safely stored)\n", n, result)
        }(i)
    }
    
    time.Sleep(500 * time.Millisecond)
    fmt.Printf("Results collected: %v (empty - problem demonstrated)\n", results)
    fmt.Println("Solution: Use channels for goroutine communication")
}
```

This example illustrates why channels are necessary in Go concurrency. Goroutines  
run independently and cannot return values directly to the calling code,  
requiring communication mechanisms for data exchange.  

## WaitGroup for synchronization

WaitGroup provides a way to wait for multiple goroutines to complete their  
execution before the main function exits.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done() // Signal completion when function returns
    
    fmt.Printf("Worker %d starting work\n", id)
    
    // Simulate work with variable duration
    workTime := time.Duration(100+(id*50)) * time.Millisecond
    time.Sleep(workTime)
    
    fmt.Printf("Worker %d completed work\n", id)
}

func main() {
    var wg sync.WaitGroup
    
    fmt.Println("Starting workers with WaitGroup synchronization")
    
    // Launch 5 workers
    for i := 1; i <= 5; i++ {
        wg.Add(1) // Increment the WaitGroup counter
        go worker(i, &wg)
    }
    
    // Launch additional workers in a batch
    wg.Add(3)
    for i := 6; i <= 8; i++ {
        go func(workerID int) {
            defer wg.Done()
            fmt.Printf("Batch worker %d executing\n", workerID)
            time.Sleep(75 * time.Millisecond)
        }(i)
    }
    
    fmt.Println("Waiting for all workers to complete...")
    wg.Wait() // Block until all workers call Done()
    
    fmt.Println("All workers have finished. Main function exiting.")
}
```

WaitGroup ensures the main function waits for all goroutines to complete.  
The Add() method increments a counter, Done() decrements it, and Wait()  
blocks until the counter reaches zero.  

## Basic unbuffered channel

Unbuffered channels provide synchronous communication between goroutines,  
blocking the sender until the receiver is ready to accept the value.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create an unbuffered channel
    ch := make(chan string)
    
    fmt.Println("Creating unbuffered channel")
    
    // Launch a goroutine to send data
    go func() {
        fmt.Println("Goroutine: About to send data")
        ch <- "Hello from goroutine!"
        fmt.Println("Goroutine: Data sent successfully")
    }()
    
    // Small delay to show timing
    time.Sleep(100 * time.Millisecond)
    
    // Receive data from the channel
    fmt.Println("Main: About to receive data")
    message := <-ch
    fmt.Println("Main: Received:", message)
    
    // Demonstrate synchronous nature
    go func() {
        for i := 1; i <= 3; i++ {
            fmt.Printf("Sending: Message %d\n", i)
            ch <- fmt.Sprintf("Message %d", i)
            fmt.Printf("Sent: Message %d\n", i)
        }
    }()
    
    // Receive all messages
    for i := 1; i <= 3; i++ {
        time.Sleep(200 * time.Millisecond) // Delay to show blocking
        msg := <-ch
        fmt.Printf("Received: %s\n", msg)
    }
}
```

Unbuffered channels enforce synchronization - the sender blocks until the  
receiver is ready, ensuring handshake-style communication between goroutines.  
This guarantees that data transfer occurs precisely when both parties are ready.  

## Basic buffered channel

Buffered channels allow asynchronous communication by storing values in an  
internal buffer, enabling senders to continue without immediate receivers.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a buffered channel with capacity of 3
    ch := make(chan int, 3)
    
    fmt.Println("Creating buffered channel with capacity 3")
    
    // Send values without blocking (up to buffer capacity)
    fmt.Println("Sending values to buffered channel...")
    ch <- 1
    fmt.Println("Sent: 1")
    ch <- 2
    fmt.Println("Sent: 2")
    ch <- 3
    fmt.Println("Sent: 3")
    fmt.Printf("Channel length: %d, capacity: %d\n", len(ch), cap(ch))
    
    // Try to send one more - this would block since buffer is full
    go func() {
        fmt.Println("Goroutine: Trying to send 4 (will block)")
        ch <- 4
        fmt.Println("Goroutine: Sent 4 (unblocked)")
    }()
    
    time.Sleep(100 * time.Millisecond)
    fmt.Println("Buffer is full, sender is blocked")
    
    // Start receiving values
    fmt.Println("Starting to receive values...")
    for i := 0; i < 4; i++ {
        val := <-ch
        fmt.Printf("Received: %d\n", val)
        fmt.Printf("Channel length: %d\n", len(ch))
        time.Sleep(200 * time.Millisecond)
    }
    
    // Demonstrate non-blocking sends when buffer has space
    fmt.Println("Sending more values...")
    ch <- 5
    ch <- 6
    fmt.Printf("Quickly sent 5 and 6. Channel length: %d\n", len(ch))
    
    // Clean up
    fmt.Printf("Final receives: %d, %d\n", <-ch, <-ch)
}
```

Buffered channels decouple senders and receivers in time. Senders can proceed  
as long as the buffer isn't full, and receivers can consume values when  
available, making them ideal for producer-consumer scenarios.  

## Channel closing

Closing channels signals that no more values will be sent and enables  
receivers to detect completion using range loops or the comma ok idiom.  

```go
package main

import (
    "fmt"
    "time"
)

func producer(ch chan<- int) {
    fmt.Println("Producer: Starting to send values")
    
    for i := 1; i <= 5; i++ {
        fmt.Printf("Producer: Sending %d\n", i)
        ch <- i
        time.Sleep(100 * time.Millisecond)
    }
    
    fmt.Println("Producer: Closing channel")
    close(ch) // Signal that no more values will be sent
}

func main() {
    ch := make(chan int, 2)
    
    // Start producer in a goroutine
    go producer(ch)
    
    // Method 1: Range loop automatically handles channel closing
    fmt.Println("Receiving with range loop:")
    for value := range ch {
        fmt.Printf("Received: %d\n", value)
    }
    fmt.Println("Range loop finished - channel was closed")
    
    // Start another producer for second demonstration
    ch2 := make(chan string, 1)
    go func() {
        ch2 <- "first"
        ch2 <- "second"
        ch2 <- "third"
        close(ch2)
    }()
    
    // Method 2: Manual checking with comma ok idiom
    fmt.Println("\nReceiving with comma ok idiom:")
    for {
        value, ok := <-ch2
        if !ok {
            fmt.Println("Channel closed, no more values")
            break
        }
        fmt.Printf("Received: %s\n", value)
    }
    
    // Demonstrate receiving from closed channel
    fmt.Println("\nReceiving from closed channel:")
    val, ok := <-ch
    fmt.Printf("Value: %d, Channel open: %t\n", val, ok)
}
```

Closing channels is essential for graceful shutdown of goroutines. Range loops  
automatically terminate when a channel is closed, and the comma ok idiom  
allows explicit checking for channel closure status.  

## Send-only and receive-only channels

Channel direction restrictions enforce communication patterns and prevent  
misuse by restricting channels to either sending or receiving operations.  

```go
package main

import (
    "fmt"
    "time"
)

// Function that only sends to a channel
func sender(ch chan<- string, id int) {
    for i := 1; i <= 3; i++ {
        message := fmt.Sprintf("Message %d from sender %d", i, id)
        fmt.Printf("Sending: %s\n", message)
        ch <- message
        time.Sleep(100 * time.Millisecond)
    }
    fmt.Printf("Sender %d finished\n", id)
}

// Function that only receives from a channel
func receiver(ch <-chan string, id int) {
    fmt.Printf("Receiver %d starting\n", id)
    
    for message := range ch {
        fmt.Printf("Receiver %d got: %s\n", id, message)
        time.Sleep(50 * time.Millisecond)
    }
    
    fmt.Printf("Receiver %d finished\n", id)
}

// Pipeline function demonstrating channel direction flow
func pipeline(input <-chan int, output chan<- int) {
    for num := range input {
        result := num * num
        fmt.Printf("Pipeline: %d -> %d\n", num, result)
        output <- result
    }
    close(output)
}

func main() {
    // Bidirectional channel that can be restricted
    ch := make(chan string, 5)
    
    fmt.Println("Starting sender and receiver with directional channels")
    
    // Pass channel as send-only to sender
    go sender(ch, 1)
    go sender(ch, 2)
    
    // Close channel when all senders are done
    go func() {
        time.Sleep(500 * time.Millisecond)
        close(ch)
    }()
    
    // Pass channel as receive-only to receiver
    receiver(ch, 1)
    
    // Demonstrate pipeline pattern with directional channels
    fmt.Println("\nDemonstrating pipeline with directional channels:")
    
    input := make(chan int, 3)
    output := make(chan int, 3)
    
    // Start pipeline
    go pipeline(input, output)
    
    // Send data to pipeline
    go func() {
        for i := 1; i <= 4; i++ {
            input <- i
        }
        close(input)
    }()
    
    // Receive processed data
    for result := range output {
        fmt.Printf("Pipeline result: %d\n", result)
    }
    
    fmt.Println("Pipeline processing complete")
}
```

Directional channels provide compile-time safety by preventing inappropriate  
operations. Send-only channels (chan<-) can only send, receive-only channels  
(<-chan) can only receive, enforcing clear communication contracts.  

## Channel communication patterns

Channels enable various communication patterns between goroutines, including  
fan-in, fan-out, and broadcast patterns for complex coordination.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Fan-out: One producer, multiple consumers
func fanOut(input <-chan int, workers int) []<-chan int {
    outputs := make([]<-chan int, workers)
    
    for i := 0; i < workers; i++ {
        output := make(chan int)
        outputs[i] = output
        
        go func(workerID int, out chan<- int) {
            defer close(out)
            
            for num := range input {
                result := num * (workerID + 1) // Different processing per worker
                fmt.Printf("Worker %d: %d -> %d\n", workerID, num, result)
                out <- result
                time.Sleep(100 * time.Millisecond)
            }
            
            fmt.Printf("Worker %d finished\n", workerID)
        }(i, output)
    }
    
    return outputs
}

// Fan-in: Multiple producers, one consumer
func fanIn(inputs ...<-chan int) <-chan int {
    output := make(chan int)
    var wg sync.WaitGroup
    
    // Start a goroutine for each input channel
    for i, input := range inputs {
        wg.Add(1)
        go func(id int, ch <-chan int) {
            defer wg.Done()
            for value := range ch {
                fmt.Printf("Fan-in from source %d: %d\n", id, value)
                output <- value
            }
        }(i, input)
    }
    
    // Close output when all inputs are drained
    go func() {
        wg.Wait()
        close(output)
    }()
    
    return output
}

func main() {
    // Create input channel for fan-out
    input := make(chan int, 5)
    
    // Send some data
    go func() {
        for i := 1; i <= 6; i++ {
            input <- i
        }
        close(input)
    }()
    
    fmt.Println("Demonstrating Fan-out pattern:")
    
    // Fan-out to 3 workers
    outputs := fanOut(input, 3)
    
    // Collect all results
    var allResults []int
    var wg sync.WaitGroup
    
    for i, output := range outputs {
        wg.Add(1)
        go func(workerID int, ch <-chan int) {
            defer wg.Done()
            for result := range ch {
                allResults = append(allResults, result)
            }
        }(i, output)
    }
    
    wg.Wait()
    fmt.Printf("All fan-out results: %v\n", allResults)
    
    fmt.Println("\nDemonstrating Fan-in pattern:")
    
    // Create multiple input sources
    source1 := make(chan int, 3)
    source2 := make(chan int, 3)
    source3 := make(chan int, 3)
    
    // Send data to sources
    go func() {
        for i := 100; i <= 102; i++ {
            source1 <- i
        }
        close(source1)
    }()
    
    go func() {
        for i := 200; i <= 202; i++ {
            source2 <- i
        }
        close(source2)
    }()
    
    go func() {
        for i := 300; i <= 302; i++ {
            source3 <- i
        }
        close(source3)
    }()
    
    // Fan-in all sources
    merged := fanIn(source1, source2, source3)
    
    // Collect merged results
    var mergedResults []int
    for result := range merged {
        mergedResults = append(mergedResults, result)
    }
    
    fmt.Printf("Fan-in results: %v\n", mergedResults)
}
```

Fan-out distributes work among multiple goroutines for parallel processing,  
while fan-in consolidates results from multiple sources. These patterns enable  
scalable concurrent architectures and efficient resource utilization.  

## Basic select statement

The select statement enables non-blocking channel operations and allows  
a goroutine to wait on multiple channel operations simultaneously.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    // Goroutine sending to ch1
    go func() {
        time.Sleep(200 * time.Millisecond)
        ch1 <- "Message from channel 1"
    }()
    
    // Goroutine sending to ch2
    go func() {
        time.Sleep(100 * time.Millisecond)
        ch2 <- "Message from channel 2"
    }()
    
    // Select waits for the first available channel operation
    fmt.Println("Waiting for first message...")
    select {
    case msg1 := <-ch1:
        fmt.Println("Received from ch1:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received from ch2:", msg2)
    }
    
    // Demonstrate multiple select operations
    fmt.Println("Waiting for second message...")
    select {
    case msg1 := <-ch1:
        fmt.Println("Received from ch1:", msg1)
    case msg2 := <-ch2:
        fmt.Println("Received from ch2:", msg2)
    case <-time.After(500 * time.Millisecond):
        fmt.Println("Timeout: No message received")
    }
    
    // Select with default case (non-blocking)
    fmt.Println("Non-blocking select attempts:")
    for i := 0; i < 3; i++ {
        select {
        case msg := <-ch1:
            fmt.Println("Got message:", msg)
        case msg := <-ch2:
            fmt.Println("Got message:", msg)
        default:
            fmt.Printf("No messages available (attempt %d)\n", i+1)
        }
        time.Sleep(50 * time.Millisecond)
    }
}
```

The select statement chooses the first channel operation that becomes ready.  
If multiple operations are ready simultaneously, it selects one at random.  
The default case makes select non-blocking for polling scenarios.  

## Select with timeout

Select statements combined with time.After enable timeout patterns for  
preventing goroutines from blocking indefinitely on channel operations.  

```go
package main

import (
    "fmt"
    "time"
)

func slowOperation(result chan<- string, duration time.Duration) {
    fmt.Printf("Starting operation (will take %v)\n", duration)
    time.Sleep(duration)
    result <- fmt.Sprintf("Operation completed after %v", duration)
}

func main() {
    fmt.Println("Demonstrating select with timeout patterns")
    
    // Example 1: Simple timeout
    ch := make(chan string, 1)
    go slowOperation(ch, 300*time.Millisecond)
    
    select {
    case result := <-ch:
        fmt.Println("Success:", result)
    case <-time.After(200 * time.Millisecond):
        fmt.Println("Timeout: Operation took too long")
    }
    
    fmt.Println()
    
    // Example 2: Multiple operations with different timeouts
    fastCh := make(chan string, 1)
    slowCh := make(chan string, 1)
    
    go slowOperation(fastCh, 100*time.Millisecond)
    go slowOperation(slowCh, 600*time.Millisecond)
    
    // Wait for fast operation or timeout
    select {
    case result := <-fastCh:
        fmt.Println("Fast operation:", result)
    case <-time.After(150 * time.Millisecond):
        fmt.Println("Fast operation timed out")
    }
    
    // Wait for slow operation with longer timeout
    select {
    case result := <-slowCh:
        fmt.Println("Slow operation:", result)
    case <-time.After(800 * time.Millisecond):
        fmt.Println("Slow operation timed out")
    }
    
    fmt.Println()
    
    // Example 3: Periodic timeout checking
    workCh := make(chan int, 5)
    
    // Send some work
    go func() {
        for i := 1; i <= 8; i++ {
            time.Sleep(time.Duration(i*50) * time.Millisecond)
            workCh <- i
        }
        close(workCh)
    }()
    
    fmt.Println("Processing with periodic timeouts:")
    timeout := 200 * time.Millisecond
    
    for {
        select {
        case work, ok := <-workCh:
            if !ok {
                fmt.Println("Work channel closed")
                return
            }
            fmt.Printf("Processing work item: %d\n", work)
            
        case <-time.After(timeout):
            fmt.Printf("No work received within %v, checking again...\n", timeout)
            // Could implement timeout handling logic here
        }
    }
}
```

Timeout patterns prevent goroutines from waiting indefinitely. The time.After  
function creates a channel that sends a value after the specified duration,  
enabling elegant timeout handling in concurrent operations.  

## Select with multiple channels

Select statements can monitor multiple channels simultaneously, making them  
ideal for coordinating complex multi-channel communication patterns.  

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func randomProducer(ch chan<- string, name string, count int) {
    defer close(ch)
    
    for i := 1; i <= count; i++ {
        // Random delay to simulate varying production rates
        delay := time.Duration(rand.Intn(300)+100) * time.Millisecond
        time.Sleep(delay)
        
        message := fmt.Sprintf("%s-message-%d", name, i)
        ch <- message
        fmt.Printf("Produced: %s\n", message)
    }
    
    fmt.Printf("Producer %s finished\n", name)
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    // Create multiple channels
    ch1 := make(chan string)
    ch2 := make(chan string)
    ch3 := make(chan string)
    
    // Start producers
    go randomProducer(ch1, "Alpha", 3)
    go randomProducer(ch2, "Beta", 4)
    go randomProducer(ch3, "Gamma", 2)
    
    // Track which channels are still active
    ch1Active, ch2Active, ch3Active := true, true, true
    messageCount := 0
    
    fmt.Println("Starting multi-channel select loop:")
    
    for ch1Active || ch2Active || ch3Active {
        select {
        case msg, ok := <-ch1:
            if ok {
                fmt.Printf("From ch1: %s\n", msg)
                messageCount++
            } else {
                fmt.Println("Channel 1 closed")
                ch1Active = false
            }
            
        case msg, ok := <-ch2:
            if ok {
                fmt.Printf("From ch2: %s\n", msg)
                messageCount++
            } else {
                fmt.Println("Channel 2 closed")
                ch2Active = false
            }
            
        case msg, ok := <-ch3:
            if ok {
                fmt.Printf("From ch3: %s\n", msg)
                messageCount++
            } else {
                fmt.Println("Channel 3 closed")
                ch3Active = false
            }
            
        case <-time.After(1 * time.Second):
            fmt.Println("Warning: No messages received for 1 second")
            // In real applications, might implement cleanup logic here
        }
    }
    
    fmt.Printf("All channels closed. Total messages received: %d\n", messageCount)
    
    // Demonstrate priority selection using nested selects
    fmt.Println("\nDemonstrating priority selection:")
    priorityCh := make(chan string, 3)
    normalCh := make(chan string, 3)
    
    // Fill both channels
    priorityCh <- "HIGH PRIORITY"
    normalCh <- "normal message"
    priorityCh <- "URGENT"
    normalCh <- "another normal message"
    
    // Priority select: always check priority channel first
    for i := 0; i < 4; i++ {
        select {
        case msg := <-priorityCh:
            fmt.Printf("Priority: %s\n", msg)
        default:
            select {
            case msg := <-priorityCh:
                fmt.Printf("Priority: %s\n", msg)
            case msg := <-normalCh:
                fmt.Printf("Normal: %s\n", msg)
            default:
                fmt.Println("No messages available")
            }
        }
    }
}
```

Multi-channel select enables sophisticated coordination patterns. Channel  
closing detection and priority selection demonstrate advanced techniques  
for managing complex concurrent communication requirements.  

## Non-blocking channel operations

Select with default cases enables non-blocking channel operations, allowing  
goroutines to continue processing instead of waiting for channel availability.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int, 2) // Small buffer for demonstration
    
    fmt.Println("Demonstrating non-blocking channel operations")
    
    // Non-blocking sends
    fmt.Println("Non-blocking send attempts:")
    for i := 1; i <= 5; i++ {
        select {
        case ch <- i:
            fmt.Printf("Successfully sent %d (buffer space available)\n", i)
        default:
            fmt.Printf("Cannot send %d (channel full), doing other work\n", i)
            // Could implement alternative logic here
            time.Sleep(50 * time.Millisecond)
        }
    }
    
    fmt.Printf("Channel length: %d, capacity: %d\n", len(ch), cap(ch))
    
    // Non-blocking receives
    fmt.Println("\nNon-blocking receive attempts:")
    for i := 0; i < 5; i++ {
        select {
        case value := <-ch:
            fmt.Printf("Successfully received %d\n", value)
        default:
            fmt.Printf("No value available (attempt %d), continuing...\n", i+1)
            // Could implement polling logic or alternative work
        }
        time.Sleep(100 * time.Millisecond)
    }
    
    // Practical example: Non-blocking producer-consumer
    fmt.Println("\nNon-blocking producer-consumer pattern:")
    
    workCh := make(chan string, 3)
    resultsCh := make(chan string, 3)
    
    // Producer that doesn't block when buffer is full
    go func() {
        tasks := []string{"task1", "task2", "task3", "task4", "task5"}
        
        for _, task := range tasks {
            select {
            case workCh <- task:
                fmt.Printf("Queued: %s\n", task)
            default:
                fmt.Printf("Queue full, deferring: %s\n", task)
                // Could implement queue overflow handling
                time.Sleep(100 * time.Millisecond)
                
                // Retry after delay
                select {
                case workCh <- task:
                    fmt.Printf("Queued after retry: %s\n", task)
                default:
                    fmt.Printf("Still cannot queue: %s\n", task)
                }
            }
        }
        close(workCh)
    }()
    
    // Consumer that processes available work
    go func() {
        for {
            select {
            case task, ok := <-workCh:
                if !ok {
                    close(resultsCh)
                    return
                }
                result := fmt.Sprintf("processed-%s", task)
                fmt.Printf("Processing: %s -> %s\n", task, result)
                
                // Non-blocking result delivery
                select {
                case resultsCh <- result:
                    // Result delivered
                default:
                    fmt.Printf("Results buffer full, result lost: %s\n", result)
                }
                
            default:
                fmt.Println("No work available, doing maintenance...")
                time.Sleep(150 * time.Millisecond)
            }
        }
    }()
    
    // Collect results non-blockingly
    timeout := time.After(2 * time.Second)
    resultsCollected := 0
    
processResults:
    for {
        select {
        case result, ok := <-resultsCh:
            if !ok {
                fmt.Println("Results channel closed")
                break processResults
            }
            fmt.Printf("Collected result: %s\n", result)
            resultsCollected++
            
        case <-timeout:
            fmt.Println("Timeout reached, stopping collection")
            break processResults
            
        default:
            // Do other work while waiting for results
            time.Sleep(50 * time.Millisecond)
        }
    }
    
    fmt.Printf("Total results collected: %d\n", resultsCollected)
}
```

Non-blocking operations prevent goroutines from stalling on full or empty  
channels. This pattern enables resilient systems that can adapt to varying  
load conditions and implement sophisticated flow control mechanisms.  

## Channel range and close

Range loops over channels provide elegant iteration patterns, while proper  
channel closing ensures graceful shutdown and resource cleanup.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func numberGenerator(ch chan<- int, start, count int) {
    defer close(ch)
    
    fmt.Printf("Generator starting: %d numbers from %d\n", count, start)
    
    for i := 0; i < count; i++ {
        value := start + i
        fmt.Printf("Generating: %d\n", value)
        ch <- value
        time.Sleep(100 * time.Millisecond)
    }
    
    fmt.Printf("Generator finished (started from %d)\n", start)
}

func processor(input <-chan int, output chan<- string, id int) {
    defer close(output)
    
    fmt.Printf("Processor %d starting\n", id)
    
    // Range automatically handles channel closing
    for number := range input {
        result := fmt.Sprintf("P%d-processed-%d", id, number*number)
        fmt.Printf("Processor %d: %d -> %s\n", id, number, result)
        output <- result
        time.Sleep(150 * time.Millisecond)
    }
    
    fmt.Printf("Processor %d finished (input channel closed)\n", id)
}

func multiplexedGenerator(outputs []chan<- int, count int) {
    var wg sync.WaitGroup
    
    // Start generators for each output channel
    for i, ch := range outputs {
        wg.Add(1)
        go func(channel chan<- int, generatorID int) {
            defer wg.Done()
            defer close(channel)
            
            for j := 1; j <= count; j++ {
                value := generatorID*100 + j
                channel <- value
                fmt.Printf("Gen%d: %d\n", generatorID, value)
                time.Sleep(time.Duration(50+generatorID*30) * time.Millisecond)
            }
        }(ch, i+1)
    }
    
    wg.Wait()
    fmt.Println("All generators finished")
}

func main() {
    // Example 1: Simple range over channel
    fmt.Println("Example 1: Basic range over channel")
    
    ch1 := make(chan int, 5)
    go numberGenerator(ch1, 1, 5)
    
    // Range loop automatically detects channel closure
    fmt.Println("Consuming with range:")
    for value := range ch1 {
        fmt.Printf("Consumed: %d\n", value)
    }
    fmt.Println("Range loop completed\n")
    
    // Example 2: Channel pipeline with range
    fmt.Println("Example 2: Pipeline with range")
    
    numbers := make(chan int, 3)
    processed := make(chan string, 3)
    
    go numberGenerator(numbers, 10, 4)
    go processor(numbers, processed, 1)
    
    fmt.Println("Pipeline results:")
    for result := range processed {
        fmt.Printf("Final result: %s\n", result)
    }
    fmt.Println("Pipeline completed\n")
    
    // Example 3: Multiple channels with synchronized closing
    fmt.Println("Example 3: Multiple channels with coordinated closing")
    
    channels := make([]chan int, 3)
    for i := range channels {
        channels[i] = make(chan int, 2)
    }
    
    // Convert to send-only for the generator
    sendChannels := make([]chan<- int, len(channels))
    for i, ch := range channels {
        sendChannels[i] = ch
    }
    
    go multiplexedGenerator(sendChannels, 3)
    
    // Collect from all channels using range
    var wg sync.WaitGroup
    results := make(chan string, 20)
    
    for i, ch := range channels {
        wg.Add(1)
        go func(input <-chan int, collectorID int) {
            defer wg.Done()
            
            for value := range input {
                result := fmt.Sprintf("Collector%d: received %d", collectorID, value)
                results <- result
            }
            fmt.Printf("Collector %d finished\n", collectorID)
        }(ch, i+1)
    }
    
    // Close results channel when all collectors finish
    go func() {
        wg.Wait()
        close(results)
    }()
    
    // Collect all results
    fmt.Println("All results:")
    for result := range results {
        fmt.Println(result)
    }
    
    fmt.Println("All operations completed successfully")
}
```

Range loops provide the idiomatic way to consume channels until closure.  
Proper channel closing propagates through pipeline stages, enabling clean  
shutdown of complex concurrent systems with multiple interconnected goroutines.  

## Mutex and shared state

Mutexes provide traditional synchronization for protecting shared state when  
channels are not suitable for the coordination pattern required.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Safe counter using mutex
type SafeCounter struct {
    mu    sync.Mutex
    value int
}

func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *SafeCounter) Decrement() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value--
}

func (c *SafeCounter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

// Shared resource with readers-writer pattern
type SafeResource struct {
    rwMu sync.RWMutex
    data map[string]int
}

func NewSafeResource() *SafeResource {
    return &SafeResource{
        data: make(map[string]int),
    }
}

func (r *SafeResource) Read(key string) (int, bool) {
    r.rwMu.RLock()
    defer r.rwMu.RUnlock()
    
    value, exists := r.data[key]
    fmt.Printf("Read %s: %d (exists: %t)\n", key, value, exists)
    return value, exists
}

func (r *SafeResource) Write(key string, value int) {
    r.rwMu.Lock()
    defer r.rwMu.Unlock()
    
    r.data[key] = value
    fmt.Printf("Write %s: %d\n", key, value)
}

func (r *SafeResource) Keys() []string {
    r.rwMu.RLock()
    defer r.rwMu.RUnlock()
    
    keys := make([]string, 0, len(r.data))
    for k := range r.data {
        keys = append(keys, k)
    }
    return keys
}

func main() {
    // Example 1: Basic mutex usage with counter
    fmt.Println("Example 1: Mutex-protected counter")
    
    counter := &SafeCounter{}
    var wg sync.WaitGroup
    
    // Launch incrementing goroutines
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(goroutineID int) {
            defer wg.Done()
            
            for j := 0; j < 100; j++ {
                counter.Increment()
            }
            fmt.Printf("Goroutine %d: finished incrementing\n", goroutineID)
        }(i + 1)
    }
    
    // Launch decrementing goroutines
    for i := 0; i < 2; i++ {
        wg.Add(1)
        go func(goroutineID int) {
            defer wg.Done()
            
            for j := 0; j < 150; j++ {
                counter.Decrement()
            }
            fmt.Printf("Decrementer %d: finished\n", goroutineID)
        }(i + 1)
    }
    
    // Monitor progress
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        for i := 0; i < 5; i++ {
            time.Sleep(200 * time.Millisecond)
            fmt.Printf("Current counter value: %d\n", counter.Value())
        }
    }()
    
    wg.Wait()
    fmt.Printf("Final counter value: %d\n", counter.Value())
    
    fmt.Println()
    
    // Example 2: RWMutex for readers-writers
    fmt.Println("Example 2: RWMutex for concurrent reads and writes")
    
    resource := NewSafeResource()
    wg = sync.WaitGroup{}
    
    // Writers
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(writerID int) {
            defer wg.Done()
            
            for j := 0; j < 3; j++ {
                key := fmt.Sprintf("key-%d-%d", writerID, j)
                value := writerID*100 + j
                
                resource.Write(key, value)
                time.Sleep(100 * time.Millisecond)
            }
            fmt.Printf("Writer %d finished\n", writerID)
        }(i + 1)
    }
    
    // Readers (can run concurrently with each other)
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(readerID int) {
            defer wg.Done()
            
            for j := 0; j < 10; j++ {
                // Try to read various keys
                key := fmt.Sprintf("key-%d-%d", (j%3)+1, j%3)
                resource.Read(key)
                time.Sleep(50 * time.Millisecond)
            }
            fmt.Printf("Reader %d finished\n", readerID)
        }(i + 1)
    }
    
    // Key lister
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        time.Sleep(300 * time.Millisecond)
        keys := resource.Keys()
        fmt.Printf("Available keys: %v\n", keys)
        
        time.Sleep(500 * time.Millisecond)
        keys = resource.Keys()
        fmt.Printf("Final keys: %v\n", keys)
    }()
    
    wg.Wait()
    
    fmt.Println("Mutex examples completed")
}
```

Mutexes provide essential protection for shared state when channel communication  
is not appropriate. RWMutex enables concurrent reads while ensuring exclusive  
writes, optimizing performance for read-heavy workloads.  

## Atomic operations

Atomic operations provide lock-free synchronization for simple shared variables,  
offering better performance than mutexes for basic counter and flag operations.  

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
    "time"
)

type AtomicCounter struct {
    value int64
}

func (c *AtomicCounter) Increment() int64 {
    return atomic.AddInt64(&c.value, 1)
}

func (c *AtomicCounter) Decrement() int64 {
    return atomic.AddInt64(&c.value, -1)
}

func (c *AtomicCounter) Load() int64 {
    return atomic.LoadInt64(&c.value)
}

func (c *AtomicCounter) Store(value int64) {
    atomic.StoreInt64(&c.value, value)
}

func (c *AtomicCounter) CompareAndSwap(old, new int64) bool {
    return atomic.CompareAndSwapInt64(&c.value, old, new)
}

// Configuration with atomic flag
type ServiceConfig struct {
    enabled int32 // 0 = disabled, 1 = enabled
    timeout int64 // timeout in milliseconds
}

func (c *ServiceConfig) Enable() {
    atomic.StoreInt32(&c.enabled, 1)
}

func (c *ServiceConfig) Disable() {
    atomic.StoreInt32(&c.enabled, 0)
}

func (c *ServiceConfig) IsEnabled() bool {
    return atomic.LoadInt32(&c.enabled) == 1
}

func (c *ServiceConfig) SetTimeout(ms int64) {
    atomic.StoreInt64(&c.timeout, ms)
}

func (c *ServiceConfig) GetTimeout() int64 {
    return atomic.LoadInt64(&c.timeout)
}

func main() {
    // Example 1: Atomic counter operations
    fmt.Println("Example 1: Atomic counter")
    
    counter := &AtomicCounter{}
    var wg sync.WaitGroup
    
    // Multiple goroutines incrementing
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(goroutineID int) {
            defer wg.Done()
            
            for j := 0; j < 1000; j++ {
                newValue := counter.Increment()
                if j%250 == 0 {
                    fmt.Printf("Goroutine %d: incremented to %d\n", goroutineID, newValue)
                }
            }
        }(i)
    }
    
    // Progress monitor
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        for i := 0; i < 10; i++ {
            time.Sleep(100 * time.Millisecond)
            value := counter.Load()
            fmt.Printf("Progress check: %d\n", value)
        }
    }()
    
    wg.Wait()
    fmt.Printf("Final counter value: %d\n", counter.Load())
    
    fmt.Println()
    
    // Example 2: Compare-and-swap operations
    fmt.Println("Example 2: Compare-and-swap")
    
    counter.Store(100)
    
    // Multiple goroutines trying to update the counter
    wg = sync.WaitGroup{}
    successCount := int64(0)
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(goroutineID int) {
            defer wg.Done()
            
            for attempt := 0; attempt < 10; attempt++ {
                current := counter.Load()
                newValue := current + int64(goroutineID+1)*10
                
                if counter.CompareAndSwap(current, newValue) {
                    atomic.AddInt64(&successCount, 1)
                    fmt.Printf("Goroutine %d: successfully updated %d -> %d\n", 
                              goroutineID, current, newValue)
                    time.Sleep(50 * time.Millisecond)
                } else {
                    fmt.Printf("Goroutine %d: failed CAS attempt %d (value changed)\n", 
                              goroutineID, attempt)
                }
                
                time.Sleep(10 * time.Millisecond)
            }
        }(i)
    }
    
    wg.Wait()
    fmt.Printf("Final value after CAS: %d, successful operations: %d\n", 
              counter.Load(), successCount)
    
    fmt.Println()
    
    // Example 3: Atomic configuration management
    fmt.Println("Example 3: Atomic configuration")
    
    config := &ServiceConfig{}
    config.Enable()
    config.SetTimeout(5000)
    
    // Service workers checking configuration
    wg = sync.WaitGroup{}
    
    for i := 0; i < 3; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            
            for j := 0; j < 8; j++ {
                if config.IsEnabled() {
                    timeout := config.GetTimeout()
                    fmt.Printf("Worker %d: service enabled, timeout=%dms\n", 
                              workerID, timeout)
                } else {
                    fmt.Printf("Worker %d: service disabled, skipping work\n", workerID)
                }
                time.Sleep(200 * time.Millisecond)
            }
        }(i + 1)
    }
    
    // Configuration manager
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        time.Sleep(800 * time.Millisecond)
        fmt.Println("Config manager: disabling service")
        config.Disable()
        
        time.Sleep(600 * time.Millisecond)
        fmt.Println("Config manager: re-enabling with new timeout")
        config.SetTimeout(10000)
        config.Enable()
    }()
    
    wg.Wait()
    
    fmt.Printf("Final config: enabled=%t, timeout=%dms\n", 
              config.IsEnabled(), config.GetTimeout())
}
```

Atomic operations enable lock-free programming for simple shared variables.  
They're faster than mutexes for basic operations and prevent the overhead  
and potential deadlocks associated with traditional locking mechanisms.  

## Once and initialization

sync.Once ensures that initialization code runs exactly once, even when  
called from multiple goroutines, providing thread-safe lazy initialization.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Global resource that needs initialization
var (
    globalResource *Resource
    initOnce       sync.Once
)

type Resource struct {
    Name string
    Data map[string]interface{}
}

func initializeResource() {
    fmt.Println("Initializing expensive resource...")
    time.Sleep(200 * time.Millisecond) // Simulate expensive initialization
    
    globalResource = &Resource{
        Name: "GlobalResource",
        Data: make(map[string]interface{}),
    }
    
    globalResource.Data["initialized"] = time.Now()
    globalResource.Data["version"] = "1.0"
    
    fmt.Println("Resource initialization completed")
}

func getResource() *Resource {
    initOnce.Do(initializeResource) // This will only run once
    return globalResource
}

// Service with lazy initialization
type DatabaseConnection struct {
    connectionString string
    connected        bool
    initOnce         sync.Once
}

func (db *DatabaseConnection) connect() {
    fmt.Printf("Connecting to database: %s\n", db.connectionString)
    time.Sleep(150 * time.Millisecond) // Simulate connection establishment
    db.connected = true
    fmt.Println("Database connection established")
}

func (db *DatabaseConnection) Query(query string) string {
    db.initOnce.Do(db.connect) // Ensure connection is established
    
    if !db.connected {
        return "ERROR: Not connected"
    }
    
    fmt.Printf("Executing query: %s\n", query)
    return fmt.Sprintf("Result for: %s", query)
}

func main() {
    fmt.Println("Demonstrating sync.Once")
    
    var wg sync.WaitGroup
    
    // Multiple goroutines trying to access the global resource
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(goroutineID int) {
            defer wg.Done()
            
            fmt.Printf("Goroutine %d: requesting resource\n", goroutineID)
            resource := getResource()
            
            fmt.Printf("Goroutine %d: got resource %s\n", goroutineID, resource.Name)
            fmt.Printf("Goroutine %d: initialization time: %v\n", 
                      goroutineID, resource.Data["initialized"])
        }(i + 1)
    }
    
    wg.Wait()
    
    fmt.Println()
    
    // Example 2: Multiple instances with individual Once
    fmt.Println("Example 2: Individual Once per instance")
    
    // Create multiple database connections
    databases := []*DatabaseConnection{
        {connectionString: "postgres://db1"},
        {connectionString: "mysql://db2"},
        {connectionString: "sqlite://db3"},
    }
    
    // Multiple goroutines using different databases
    for i, db := range databases {
        for j := 0; j < 3; j++ {
            wg.Add(1)
            go func(database *DatabaseConnection, dbID, queryID int) {
                defer wg.Done()
                
                query := fmt.Sprintf("SELECT * FROM table%d", queryID)
                result := database.Query(query)
                fmt.Printf("DB%d Query%d: %s\n", dbID, queryID, result)
            }(db, i+1, j+1)
        }
    }
    
    wg.Wait()
    
    fmt.Println()
    
    // Example 3: Configuration loading with Once
    fmt.Println("Example 3: Configuration loading")
    
    type Config struct {
        once     sync.Once
        settings map[string]string
    }
    
    config := &Config{
        settings: make(map[string]string),
    }
    
    loadConfig := func() {
        fmt.Println("Loading configuration from file...")
        time.Sleep(100 * time.Millisecond)
        
        config.settings["api_endpoint"] = "https://api.example.com"
        config.settings["timeout"] = "30s"
        config.settings["retry_count"] = "3"
        
        fmt.Println("Configuration loaded successfully")
    }
    
    getConfig := func(key string) string {
        config.once.Do(loadConfig)
        return config.settings[key]
    }
    
    // Multiple services requesting configuration
    services := []string{"auth", "payment", "notification", "logging"}
    
    for _, service := range services {
        wg.Add(1)
        go func(serviceName string) {
            defer wg.Done()
            
            endpoint := getConfig("api_endpoint")
            timeout := getConfig("timeout")
            
            fmt.Printf("Service %s: endpoint=%s, timeout=%s\n", 
                      serviceName, endpoint, timeout)
        }(service)
    }
    
    wg.Wait()
    
    fmt.Println("All services configured successfully")
}
```

sync.Once guarantees that initialization code executes exactly once across  
all goroutines. This pattern is essential for lazy initialization, singleton  
creation, and ensuring expensive setup operations occur only when needed.  

## Worker pool pattern

Worker pools manage a fixed number of goroutines to process jobs from a queue,  
providing controlled concurrency and efficient resource utilization.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Job struct {
    ID       int
    Data     string
    Priority int
}

type Result struct {
    Job    Job
    Output string
    Error  error
}

func worker(id int, jobs <-chan Job, results chan<- Result, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("Worker %d: starting\n", id)
    
    for job := range jobs {
        fmt.Printf("Worker %d: processing job %d\n", id, job.ID)
        
        // Simulate work based on job priority
        workTime := time.Duration(100+job.Priority*50) * time.Millisecond
        time.Sleep(workTime)
        
        // Simulate occasional errors
        var err error
        if job.ID%7 == 0 {
            err = fmt.Errorf("simulated error for job %d", job.ID)
        }
        
        result := Result{
            Job:    job,
            Output: fmt.Sprintf("Processed: %s (worker %d)", job.Data, id),
            Error:  err,
        }
        
        results <- result
        fmt.Printf("Worker %d: completed job %d\n", id, job.ID)
    }
    
    fmt.Printf("Worker %d: shutting down\n", id)
}

func createWorkerPool(numWorkers int, jobs <-chan Job, results chan<- Result) {
    var wg sync.WaitGroup
    
    // Start workers
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go worker(i, jobs, results, &wg)
    }
    
    // Close results channel when all workers finish
    go func() {
        wg.Wait()
        close(results)
        fmt.Println("All workers have finished")
    }()
}

func main() {
    const numJobs = 15
    const numWorkers = 4
    
    fmt.Printf("Starting worker pool with %d workers for %d jobs\n", numWorkers, numJobs)
    
    // Create channels
    jobs := make(chan Job, 10)       // Buffered job queue
    results := make(chan Result, 10) // Buffered results
    
    // Start worker pool
    createWorkerPool(numWorkers, jobs, results)
    
    // Send jobs
    go func() {
        defer close(jobs)
        
        for i := 1; i <= numJobs; i++ {
            job := Job{
                ID:       i,
                Data:     fmt.Sprintf("task-%d", i),
                Priority: i % 3, // 0=low, 1=medium, 2=high priority
            }
            
            fmt.Printf("Submitting job %d (priority %d)\n", job.ID, job.Priority)
            jobs <- job
        }
        
        fmt.Println("All jobs submitted")
    }()
    
    // Collect results
    var successCount, errorCount int
    var wg sync.WaitGroup
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        for result := range results {
            if result.Error != nil {
                fmt.Printf("ERROR - Job %d: %v\n", result.Job.ID, result.Error)
                errorCount++
            } else {
                fmt.Printf("SUCCESS - Job %d: %s\n", result.Job.ID, result.Output)
                successCount++
            }
        }
    }()
    
    // Monitor progress
    wg.Add(1)
    go func() {
        defer wg.Done()
        
        ticker := time.NewTicker(500 * time.Millisecond)
        defer ticker.Stop()
        
        for i := 0; i < 8; i++ {
            select {
            case <-ticker.C:
                fmt.Printf("Progress: %d jobs queued, %d results ready\n", 
                          len(jobs), len(results))
            }
        }
    }()
    
    wg.Wait()
    
    fmt.Printf("\nWorker Pool Summary:\n")
    fmt.Printf("Total jobs: %d\n", numJobs)
    fmt.Printf("Successful: %d\n", successCount)
    fmt.Printf("Failed: %d\n", errorCount)
    
    // Advanced example: Dynamic worker pool
    fmt.Println("\nAdvanced: Dynamic worker scaling")
    dynamicWorkerExample()
}

func dynamicWorkerExample() {
    jobs := make(chan Job, 20)
    results := make(chan Result, 20)
    
    // Start with 2 workers
    var workerWG sync.WaitGroup
    currentWorkers := 2
    
    startWorkers := func(count int) {
        for i := 0; i < count; i++ {
            workerWG.Add(1)
            go worker(currentWorkers+i+1, jobs, results, &workerWG)
        }
        currentWorkers += count
    }
    
    // Initial workers
    startWorkers(2)
    
    // Job generator with burst pattern
    go func() {
        defer close(jobs)
        
        // Initial batch
        for i := 1; i <= 5; i++ {
            jobs <- Job{ID: i, Data: fmt.Sprintf("batch1-task%d", i), Priority: 1}
        }
        
        time.Sleep(300 * time.Millisecond)
        
        // Detect load and scale up
        if len(jobs) > 3 {
            fmt.Println("High load detected, scaling up workers")
            startWorkers(2) // Add 2 more workers
        }
        
        // Heavy load batch
        for i := 6; i <= 12; i++ {
            jobs <- Job{ID: i, Data: fmt.Sprintf("batch2-task%d", i), Priority: 2}
        }
        
        fmt.Printf("All dynamic jobs submitted. Active workers: %d\n", currentWorkers)
    }()
    
    // Results collector
    go func() {
        workerWG.Wait()
        close(results)
    }()
    
    // Collect dynamic results
    dynamicResults := 0
    for result := range results {
        fmt.Printf("Dynamic result: Job %d completed\n", result.Job.ID)
        dynamicResults++
    }
    
    fmt.Printf("Dynamic worker pool processed %d jobs with %d workers\n", 
              dynamicResults, currentWorkers)
}
```

Worker pools provide controlled concurrency by limiting the number of active  
goroutines. They're essential for managing resource usage, preventing system  
overload, and ensuring predictable performance under varying load conditions.  

## Pipeline pattern

Pipelines connect multiple processing stages through channels, enabling  
data to flow through transformation steps in a concurrent, streaming manner.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Pipeline stage: number generation
func generateNumbers(count int) <-chan int {
    output := make(chan int)
    
    go func() {
        defer close(output)
        
        fmt.Println("Generator: starting number generation")
        
        for i := 1; i <= count; i++ {
            fmt.Printf("Generator: producing %d\n", i)
            output <- i
            time.Sleep(100 * time.Millisecond)
        }
        
        fmt.Println("Generator: finished")
    }()
    
    return output
}

// Pipeline stage: number transformation
func transform(input <-chan int, operation string) <-chan int {
    output := make(chan int)
    
    go func() {
        defer close(output)
        
        fmt.Printf("Transformer (%s): starting\n", operation)
        
        for number := range input {
            var result int
            
            switch operation {
            case "square":
                result = number * number
            case "double":
                result = number * 2
            case "increment":
                result = number + 1
            default:
                result = number
            }
            
            fmt.Printf("Transformer (%s): %d -> %d\n", operation, number, result)
            output <- result
            time.Sleep(80 * time.Millisecond)
        }
        
        fmt.Printf("Transformer (%s): finished\n", operation)
    }()
    
    return output
}

// Pipeline stage: filtering
func filter(input <-chan int, predicate func(int) bool, name string) <-chan int {
    output := make(chan int)
    
    go func() {
        defer close(output)
        
        fmt.Printf("Filter (%s): starting\n", name)
        
        for number := range input {
            if predicate(number) {
                fmt.Printf("Filter (%s): %d passed\n", name, number)
                output <- number
            } else {
                fmt.Printf("Filter (%s): %d filtered out\n", name, number)
            }
            time.Sleep(50 * time.Millisecond)
        }
        
        fmt.Printf("Filter (%s): finished\n", name)
    }()
    
    return output
}

// Pipeline stage: aggregation
func aggregate(input <-chan int, operation string) <-chan int {
    output := make(chan int, 1) // Single result
    
    go func() {
        defer close(output)
        
        fmt.Printf("Aggregator (%s): starting\n", operation)
        
        var result int
        count := 0
        
        for number := range input {
            count++
            switch operation {
            case "sum":
                result += number
            case "max":
                if count == 1 || number > result {
                    result = number
                }
            case "min":
                if count == 1 || number < result {
                    result = number
                }
            }
            fmt.Printf("Aggregator (%s): processing %d, current result: %d\n", 
                      operation, number, result)
        }
        
        if count > 0 {
            fmt.Printf("Aggregator (%s): final result: %d\n", operation, result)
            output <- result
        }
        
        fmt.Printf("Aggregator (%s): finished\n", operation)
    }()
    
    return output
}

// Fan-out to multiple parallel pipelines
func parallelPipelines(input <-chan int, numPipelines int) <-chan int {
    outputs := make([]<-chan int, numPipelines)
    
    // Create parallel processing pipelines
    for i := 0; i < numPipelines; i++ {
        // Each pipeline gets its own subset of data
        pipelineInput := make(chan int)
        
        // Split input across pipelines
        go func(id int, out chan<- int) {
            defer close(out)
            
            for value := range input {
                if value%numPipelines == id {
                    fmt.Printf("Pipeline %d: received %d\n", id, value)
                    out <- value
                }
            }
        }(i, pipelineInput)
        
        // Process through individual pipeline
        outputs[i] = transform(pipelineInput, "square")
    }
    
    // Fan-in: merge results from parallel pipelines
    merged := make(chan int)
    var wg sync.WaitGroup
    
    for i, output := range outputs {
        wg.Add(1)
        go func(id int, ch <-chan int) {
            defer wg.Done()
            
            for result := range ch {
                fmt.Printf("Merge: pipeline %d result: %d\n", id, result)
                merged <- result
            }
        }(i, output)
    }
    
    go func() {
        wg.Wait()
        close(merged)
        fmt.Println("All parallel pipelines completed")
    }()
    
    return merged
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    fmt.Println("Demonstrating Pipeline Pattern")
    
    // Example 1: Simple linear pipeline
    fmt.Println("\nExample 1: Linear Pipeline")
    
    // Build pipeline: generate -> square -> filter even -> sum
    numbers := generateNumbers(5)
    squared := transform(numbers, "square")
    evenOnly := filter(squared, func(n int) bool { return n%2 == 0 }, "even-only")
    sum := aggregate(evenOnly, "sum")
    
    // Get final result
    for result := range sum {
        fmt.Printf("Final pipeline result: %d\n", result)
    }
    
    fmt.Println()
    
    // Example 2: Multi-stage pipeline with branching
    fmt.Println("Example 2: Multi-stage branching pipeline")
    
    numbers2 := generateNumbers(6)
    
    // Branch 1: square -> filter large -> max
    branch1 := transform(numbers2, "square")
    largeBranch := filter(branch1, func(n int) bool { return n > 10 }, "large-numbers")
    maxResult := aggregate(largeBranch, "max")
    
    // Get branching result
    for result := range maxResult {
        fmt.Printf("Branching pipeline result: %d\n", result)
    }
    
    fmt.Println()
    
    // Example 3: Parallel processing pipeline
    fmt.Println("Example 3: Parallel processing pipelines")
    
    numbers3 := generateNumbers(8)
    parallelResults := parallelPipelines(numbers3, 3)
    
    fmt.Println("Parallel pipeline results:")
    var allResults []int
    for result := range parallelResults {
        allResults = append(allResults, result)
        fmt.Printf("Collected: %d\n", result)
    }
    
    fmt.Printf("All parallel results: %v\n", allResults)
    
    fmt.Println()
    
    // Example 4: Error-handling pipeline
    fmt.Println("Example 4: Error-handling pipeline")
    errorHandlingPipeline()
}

func errorHandlingPipeline() {
    type Result struct {
        Value int
        Error error
    }
    
    // Generator with potential errors
    generateWithErrors := func() <-chan Result {
        output := make(chan Result)
        
        go func() {
            defer close(output)
            
            for i := 1; i <= 6; i++ {
                var err error
                if i == 3 {
                    err = fmt.Errorf("simulated error at %d", i)
                }
                
                output <- Result{Value: i, Error: err}
                time.Sleep(100 * time.Millisecond)
            }
        }()
        
        return output
    }
    
    // Error-aware processor
    processWithErrors := func(input <-chan Result) <-chan Result {
        output := make(chan Result)
        
        go func() {
            defer close(output)
            
            for result := range input {
                if result.Error != nil {
                    fmt.Printf("Error processor: passing through error: %v\n", result.Error)
                    output <- result
                    continue
                }
                
                // Process successful values
                newValue := result.Value * 10
                fmt.Printf("Error processor: %d -> %d\n", result.Value, newValue)
                output <- Result{Value: newValue, Error: nil}
                time.Sleep(50 * time.Millisecond)
            }
        }()
        
        return output
    }
    
    // Build error-handling pipeline
    input := generateWithErrors()
    processed := processWithErrors(input)
    
    // Collect results and errors
    var successResults []int
    var errors []error
    
    for result := range processed {
        if result.Error != nil {
            errors = append(errors, result.Error)
        } else {
            successResults = append(successResults, result.Value)
        }
    }
    
    fmt.Printf("Error-handling pipeline completed:\n")
    fmt.Printf("Successful results: %v\n", successResults)
    fmt.Printf("Errors encountered: %d\n", len(errors))
    for _, err := range errors {
        fmt.Printf("  - %v\n", err)
    }
}
```

Pipeline patterns enable efficient streaming data processing through multiple  
stages. Each stage runs concurrently, allowing for high throughput and  
natural parallelization of complex data transformation workflows.  

## Context and cancellation

Context provides a way to carry cancellation signals, deadlines, and values  
across goroutine boundaries, enabling graceful shutdown and resource cleanup.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func longRunningTask(ctx context.Context, taskID int) {
    fmt.Printf("Task %d: starting\n", taskID)
    
    for i := 1; i <= 10; i++ {
        // Check for cancellation
        select {
        case <-ctx.Done():
            fmt.Printf("Task %d: cancelled at step %d, reason: %v\n", 
                      taskID, i, ctx.Err())
            return
        default:
            // Continue with work
        }
        
        fmt.Printf("Task %d: step %d\n", taskID, i)
        time.Sleep(200 * time.Millisecond)
    }
    
    fmt.Printf("Task %d: completed successfully\n", taskID)
}

func networkOperation(ctx context.Context, url string) error {
    fmt.Printf("Starting network operation: %s\n", url)
    
    // Simulate network call with context timeout
    select {
    case <-time.After(800 * time.Millisecond):
        fmt.Printf("Network operation completed: %s\n", url)
        return nil
    case <-ctx.Done():
        fmt.Printf("Network operation cancelled: %s, reason: %v\n", url, ctx.Err())
        return ctx.Err()
    }
}

func main() {
    // Example 1: Basic cancellation
    fmt.Println("Example 1: Basic context cancellation")
    
    ctx, cancel := context.WithCancel(context.Background())
    
    go longRunningTask(ctx, 1)
    go longRunningTask(ctx, 2)
    
    // Let tasks run for a while, then cancel
    time.Sleep(1 * time.Second)
    fmt.Println("Main: cancelling all tasks")
    cancel()
    
    // Wait for tasks to handle cancellation
    time.Sleep(500 * time.Millisecond)
    
    fmt.Println()
    
    // Example 2: Timeout context
    fmt.Println("Example 2: Context with timeout")
    
    ctx2, cancel2 := context.WithTimeout(context.Background(), 1500*time.Millisecond)
    defer cancel2()
    
    go longRunningTask(ctx2, 3)
    
    // This task will timeout before completion
    time.Sleep(2 * time.Second)
    
    fmt.Println()
    
    // Example 3: Deadline context
    fmt.Println("Example 3: Context with deadline")
    
    deadline := time.Now().Add(1200 * time.Millisecond)
    ctx3, cancel3 := context.WithDeadline(context.Background(), deadline)
    defer cancel3()
    
    err := networkOperation(ctx3, "https://api.example.com/data")
    if err != nil {
        fmt.Printf("Network operation failed: %v\n", err)
    }
    
    // This will timeout
    err = networkOperation(ctx3, "https://slow-api.example.com/data")
    if err != nil {
        fmt.Printf("Slow network operation failed: %v\n", err)
    }
    
    fmt.Println()
    
    // Example 4: Context value passing
    fmt.Println("Example 4: Context with values")
    
    type contextKey string
    const (
        userIDKey    contextKey = "userID"
        requestIDKey contextKey = "requestID"
    )
    
    processRequest := func(ctx context.Context, operation string) {
        userID := ctx.Value(userIDKey)
        requestID := ctx.Value(requestIDKey)
        
        fmt.Printf("Processing %s - User: %v, Request: %v\n", 
                  operation, userID, requestID)
        
        // Simulate work
        select {
        case <-time.After(300 * time.Millisecond):
            fmt.Printf("Completed %s\n", operation)
        case <-ctx.Done():
            fmt.Printf("Cancelled %s: %v\n", operation, ctx.Err())
        }
    }
    
    // Create context with values
    ctx4 := context.WithValue(context.Background(), userIDKey, "user123")
    ctx4 = context.WithValue(ctx4, requestIDKey, "req456")
    
    // Add timeout to the context chain
    ctx4, cancel4 := context.WithTimeout(ctx4, 2*time.Second)
    defer cancel4()
    
    go processRequest(ctx4, "authenticate")
    go processRequest(ctx4, "fetch-data")
    go processRequest(ctx4, "process-payment")
    
    time.Sleep(1 * time.Second)
    
    fmt.Println("Context examples completed")
}
```

Context enables clean cancellation propagation and resource management  
across goroutine boundaries. It's essential for building robust services  
that can handle timeouts, user cancellations, and graceful shutdowns.  

## Context with HTTP-like server

Context patterns are commonly used in server applications for request  
handling, timeout management, and graceful service shutdown.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Simulated HTTP request
type Request struct {
    ID      string
    Path    string
    UserID  string
    Timeout time.Duration
}

// Simulated HTTP response
type Response struct {
    RequestID  string
    StatusCode int
    Body       string
    Error      error
}

// Database simulation
func queryDatabase(ctx context.Context, query string) (string, error) {
    fmt.Printf("Database: executing query: %s\n", query)
    
    // Simulate database work
    select {
    case <-time.After(400 * time.Millisecond):
        return fmt.Sprintf("result for: %s", query), nil
    case <-ctx.Done():
        fmt.Printf("Database: query cancelled: %s\n", query)
        return "", ctx.Err()
    }
}

// External API simulation
func callExternalAPI(ctx context.Context, endpoint string) (string, error) {
    fmt.Printf("API: calling %s\n", endpoint)
    
    select {
    case <-time.After(600 * time.Millisecond):
        return fmt.Sprintf("response from %s", endpoint), nil
    case <-ctx.Done():
        fmt.Printf("API: call cancelled: %s\n", endpoint)
        return "", ctx.Err()
    }
}

// Request handler with context
func handleRequest(ctx context.Context, req Request) Response {
    fmt.Printf("Handler: processing request %s (%s)\n", req.ID, req.Path)
    
    // Add request-specific timeout
    reqCtx, cancel := context.WithTimeout(ctx, req.Timeout)
    defer cancel()
    
    // Add request ID to context
    type contextKey string
    const requestIDKey contextKey = "requestID"
    reqCtx = context.WithValue(reqCtx, requestIDKey, req.ID)
    
    response := Response{RequestID: req.ID}
    
    switch req.Path {
    case "/user":
        // Database query for user data
        dbResult, err := queryDatabase(reqCtx, fmt.Sprintf("SELECT * FROM users WHERE id='%s'", req.UserID))
        if err != nil {
            response.StatusCode = 500
            response.Error = err
            return response
        }
        
        response.StatusCode = 200
        response.Body = dbResult
        
    case "/profile":
        // Multiple concurrent operations
        var wg sync.WaitGroup
        var mu sync.Mutex
        var dbResult, apiResult string
        var dbErr, apiErr error
        
        wg.Add(2)
        
        // Concurrent database query
        go func() {
            defer wg.Done()
            result, err := queryDatabase(reqCtx, "SELECT profile FROM users")
            mu.Lock()
            dbResult, dbErr = result, err
            mu.Unlock()
        }()
        
        // Concurrent API call
        go func() {
            defer wg.Done()
            result, err := callExternalAPI(reqCtx, "/external/preferences")
            mu.Lock()
            apiResult, apiErr = result, err
            mu.Unlock()
        }()
        
        // Wait for both operations
        wg.Wait()
        
        if dbErr != nil || apiErr != nil {
            response.StatusCode = 500
            if dbErr != nil {
                response.Error = dbErr
            } else {
                response.Error = apiErr
            }
            return response
        }
        
        response.StatusCode = 200
        response.Body = fmt.Sprintf("Combined: %s + %s", dbResult, apiResult)
        
    case "/slow":
        // Intentionally slow operation to demonstrate timeout
        select {
        case <-time.After(2 * time.Second):
            response.StatusCode = 200
            response.Body = "slow operation completed"
        case <-reqCtx.Done():
            response.StatusCode = 408
            response.Error = reqCtx.Err()
        }
        
    default:
        response.StatusCode = 404
        response.Body = "not found"
    }
    
    fmt.Printf("Handler: completed request %s (status: %d)\n", req.ID, response.StatusCode)
    return response
}

// Server simulation
func runServer(ctx context.Context, requests []Request) []Response {
    fmt.Println("Server: starting request processing")
    
    responses := make([]Response, len(requests))
    var wg sync.WaitGroup
    
    for i, req := range requests {
        wg.Add(1)
        go func(index int, request Request) {
            defer wg.Done()
            
            // Check if server context is still active
            select {
            case <-ctx.Done():
                responses[index] = Response{
                    RequestID:  request.ID,
                    StatusCode: 503,
                    Error:      fmt.Errorf("server shutting down"),
                }
                return
            default:
            }
            
            responses[index] = handleRequest(ctx, request)
        }(i, req)
    }
    
    wg.Wait()
    fmt.Println("Server: all requests processed")
    return responses
}

func main() {
    fmt.Println("Simulating HTTP-like server with context")
    
    // Create requests with different characteristics
    requests := []Request{
        {ID: "req1", Path: "/user", UserID: "user123", Timeout: 1 * time.Second},
        {ID: "req2", Path: "/profile", UserID: "user456", Timeout: 2 * time.Second},
        {ID: "req3", Path: "/slow", UserID: "user789", Timeout: 500 * time.Millisecond}, // Will timeout
        {ID: "req4", Path: "/nonexistent", UserID: "user000", Timeout: 1 * time.Second},
    }
    
    // Server context with overall timeout
    serverCtx, serverCancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer serverCancel()
    
    // Process requests
    responses := runServer(serverCtx, requests)
    
    // Display results
    fmt.Println("\nServer Response Summary:")
    for _, resp := range responses {
        fmt.Printf("Request %s: Status %d", resp.RequestID, resp.StatusCode)
        if resp.Error != nil {
            fmt.Printf(", Error: %v", resp.Error)
        } else if resp.Body != "" {
            fmt.Printf(", Body: %s", resp.Body)
        }
        fmt.Println()
    }
    
    fmt.Println()
    
    // Demonstrate server shutdown
    fmt.Println("Demonstrating graceful server shutdown")
    
    // New server context with short timeout
    shutdownCtx, shutdownCancel := context.WithTimeout(context.Background(), 800*time.Millisecond)
    defer shutdownCancel()
    
    slowRequests := []Request{
        {ID: "slow1", Path: "/slow", UserID: "user1", Timeout: 3 * time.Second},
        {ID: "slow2", Path: "/slow", UserID: "user2", Timeout: 3 * time.Second},
    }
    
    shutdownResponses := runServer(shutdownCtx, slowRequests)
    
    fmt.Println("\nShutdown Response Summary:")
    for _, resp := range shutdownResponses {
        fmt.Printf("Request %s: Status %d", resp.RequestID, resp.StatusCode)
        if resp.Error != nil {
            fmt.Printf(", Error: %v", resp.Error)
        }
        fmt.Println()
    }
}
```

Context-aware servers can gracefully handle timeouts, cancellations, and  
shutdowns. This pattern ensures proper resource cleanup and provides  
consistent behavior under various failure and load conditions.  

## Context best practices

Context usage patterns and best practices for building robust concurrent  
applications that properly handle cancellation and resource management.  

```go
package main

import (
    "context"
    "errors"
    "fmt"
    "sync"
    "time"
)

// Custom errors for better error handling
var (
    ErrServiceUnavailable = errors.New("service unavailable")
    ErrTimeout           = errors.New("operation timeout")
)

// Service interface demonstrating context best practices
type DataService struct {
    name string
}

// Good practice: Accept context as first parameter
func (s *DataService) FetchData(ctx context.Context, id string) (string, error) {
    // Always check context at the beginning
    if err := ctx.Err(); err != nil {
        return "", err
    }
    
    fmt.Printf("%s: starting fetch for ID: %s\n", s.name, id)
    
    // Simulate work with context checking
    for i := 0; i < 5; i++ {
        select {
        case <-ctx.Done():
            fmt.Printf("%s: fetch cancelled for ID: %s\n", s.name, id)
            return "", ctx.Err()
        case <-time.After(200 * time.Millisecond):
            // Continue processing
            fmt.Printf("%s: processing step %d for ID: %s\n", s.name, i+1, id)
        }
    }
    
    fmt.Printf("%s: fetch completed for ID: %s\n", s.name, id)
    return fmt.Sprintf("data-%s", id), nil
}

// Good practice: Propagate context through call chain
func (s *DataService) ProcessBatch(ctx context.Context, ids []string) (map[string]string, error) {
    results := make(map[string]string)
    var mu sync.Mutex
    var wg sync.WaitGroup
    
    // Use errgroup pattern for better error handling
    errChan := make(chan error, len(ids))
    
    for _, id := range ids {
        wg.Add(1)
        go func(itemID string) {
            defer wg.Done()
            
            // Create child context for individual operations
            childCtx, cancel := context.WithTimeout(ctx, 1500*time.Millisecond)
            defer cancel()
            
            data, err := s.FetchData(childCtx, itemID)
            if err != nil {
                errChan <- fmt.Errorf("failed to fetch %s: %w", itemID, err)
                return
            }
            
            mu.Lock()
            results[itemID] = data
            mu.Unlock()
        }(id)
    }
    
    wg.Wait()
    close(errChan)
    
    // Check for any errors
    var errs []error
    for err := range errChan {
        errs = append(errs, err)
    }
    
    if len(errs) > 0 {
        return results, fmt.Errorf("batch processing errors: %v", errs)
    }
    
    return results, nil
}

// Advanced context pattern: Context with custom values and keys
type contextKey string

const (
    TraceIDKey   contextKey = "traceID"
    UserIDKey    contextKey = "userID"
    OperationKey contextKey = "operation"
)

func withTracing(ctx context.Context, traceID string) context.Context {
    return context.WithValue(ctx, TraceIDKey, traceID)
}

func withUser(ctx context.Context, userID string) context.Context {
    return context.WithValue(ctx, UserIDKey, userID)
}

func withOperation(ctx context.Context, operation string) context.Context {
    return context.WithValue(ctx, OperationKey, operation)
}

func getTraceID(ctx context.Context) string {
    if traceID, ok := ctx.Value(TraceIDKey).(string); ok {
        return traceID
    }
    return "no-trace"
}

func tracedOperation(ctx context.Context, name string, duration time.Duration) error {
    traceID := getTraceID(ctx)
    userID := ctx.Value(UserIDKey)
    
    fmt.Printf("[%s] Starting %s for user %v\n", traceID, name, userID)
    
    select {
    case <-time.After(duration):
        fmt.Printf("[%s] Completed %s\n", traceID, name)
        return nil
    case <-ctx.Done():
        fmt.Printf("[%s] Cancelled %s: %v\n", traceID, name, ctx.Err())
        return ctx.Err()
    }
}

// Pattern: Context middleware for common functionality
func withLogging(ctx context.Context, operation string) (context.Context, func()) {
    start := time.Now()
    traceID := getTraceID(ctx)
    
    fmt.Printf("[%s] Operation started: %s\n", traceID, operation)
    
    cleanup := func() {
        duration := time.Since(start)
        if ctx.Err() != nil {
            fmt.Printf("[%s] Operation cancelled: %s (after %v)\n", traceID, operation, duration)
        } else {
            fmt.Printf("[%s] Operation completed: %s (took %v)\n", traceID, operation, duration)
        }
    }
    
    return withOperation(ctx, operation), cleanup
}

func main() {
    fmt.Println("Demonstrating Context Best Practices")
    
    // Example 1: Basic service usage with proper context handling
    fmt.Println("\nExample 1: Proper context propagation")
    
    service := &DataService{name: "DataService"}
    
    ctx1, cancel1 := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel1()
    
    data, err := service.FetchData(ctx1, "item1")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Success: %s\n", data)
    }
    
    // Example 2: Batch processing with individual timeouts
    fmt.Println("\nExample 2: Batch processing")
    
    ctx2, cancel2 := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel2()
    
    ids := []string{"batch1", "batch2", "batch3"}
    results, err := service.ProcessBatch(ctx2, ids)
    if err != nil {
        fmt.Printf("Batch error: %v\n", err)
    }
    
    fmt.Printf("Batch results: %v\n", results)
    
    // Example 3: Context with values and tracing
    fmt.Println("\nExample 3: Context values and tracing")
    
    baseCtx := context.Background()
    tracedCtx := withTracing(baseCtx, "trace-123")
    userCtx := withUser(tracedCtx, "user-456")
    
    // Add timeout to the context chain
    timedCtx, cancel3 := context.WithTimeout(userCtx, 3*time.Second)
    defer cancel3()
    
    var wg sync.WaitGroup
    
    operations := []struct {
        name     string
        duration time.Duration
    }{
        {"validate-user", 300 * time.Millisecond},
        {"load-preferences", 500 * time.Millisecond},
        {"update-cache", 200 * time.Millisecond},
    }
    
    for _, op := range operations {
        wg.Add(1)
        go func(name string, dur time.Duration) {
            defer wg.Done()
            
            // Use middleware pattern for consistent logging
            opCtx, cleanup := withLogging(timedCtx, name)
            defer cleanup()
            
            err := tracedOperation(opCtx, name, dur)
            if err != nil {
                fmt.Printf("Operation %s failed: %v\n", name, err)
            }
        }(op.name, op.duration)
    }
    
    wg.Wait()
    
    // Example 4: Context cancellation cascading
    fmt.Println("\nExample 4: Cancellation cascading")
    
    parentCtx, parentCancel := context.WithCancel(context.Background())
    childCtx1, child1Cancel := context.WithTimeout(parentCtx, 2*time.Second)
    childCtx2, child2Cancel := context.WithTimeout(parentCtx, 3*time.Second)
    
    defer parentCancel()
    defer child1Cancel()
    defer child2Cancel()
    
    // Start operations with different contexts
    wg.Add(2)
    
    go func() {
        defer wg.Done()
        err := tracedOperation(withTracing(childCtx1, "child1-trace"), "child1-op", 4*time.Second)
        fmt.Printf("Child1 result: %v\n", err)
    }()
    
    go func() {
        defer wg.Done()
        err := tracedOperation(withTracing(childCtx2, "child2-trace"), "child2-op", 4*time.Second)
        fmt.Printf("Child2 result: %v\n", err)
    }()
    
    // Cancel parent after 1 second - this should cascade to all children
    time.Sleep(1 * time.Second)
    fmt.Println("Cancelling parent context...")
    parentCancel()
    
    wg.Wait()
    
    fmt.Println("Context best practices demonstration completed")
}
```

Context best practices include accepting context as the first parameter,  
checking cancellation regularly, propagating context through call chains,  
and using context values judiciously for request-scoped data like trace IDs.  

## Fan-in fan-out pattern

Fan-in and fan-out patterns distribute work across multiple goroutines  
and consolidate results, enabling scalable parallel processing architectures.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Work item for processing
type WorkItem struct {
    ID       int
    Data     string
    Priority int
}

// Result from processing
type Result struct {
    WorkerID int
    Item     WorkItem
    Output   string
    Duration time.Duration
}

// Fan-out: Distribute work to multiple workers
func fanOut(input <-chan WorkItem, numWorkers int) []<-chan Result {
    outputs := make([]<-chan Result, numWorkers)
    
    for i := 0; i < numWorkers; i++ {
        output := make(chan Result)
        outputs[i] = output
        
        go func(workerID int, workChan <-chan WorkItem, resultChan chan<- Result) {
            defer close(resultChan)
            
            fmt.Printf("Worker %d: starting\n", workerID)
            
            for item := range workChan {
                start := time.Now()
                
                fmt.Printf("Worker %d: processing item %d\n", workerID, item.ID)
                
                // Simulate work based on priority
                processingTime := time.Duration(100+item.Priority*50) * time.Millisecond
                time.Sleep(processingTime)
                
                result := Result{
                    WorkerID: workerID,
                    Item:     item,
                    Output:   fmt.Sprintf("Processed-%s-by-worker-%d", item.Data, workerID),
                    Duration: time.Since(start),
                }
                
                resultChan <- result
                fmt.Printf("Worker %d: completed item %d in %v\n", 
                          workerID, item.ID, result.Duration)
            }
            
            fmt.Printf("Worker %d: shutting down\n", workerID)
        }(i+1, input, output)
    }
    
    return outputs
}

// Fan-in: Combine results from multiple workers
func fanIn(inputs ...<-chan Result) <-chan Result {
    output := make(chan Result)
    var wg sync.WaitGroup
    
    // Start a goroutine for each input channel
    for i, input := range inputs {
        wg.Add(1)
        go func(workerOutput int, ch <-chan Result) {
            defer wg.Done()
            
            for result := range ch {
                fmt.Printf("Fan-in: collecting result from worker %d (item %d)\n", 
                          result.WorkerID, result.Item.ID)
                output <- result
            }
            
            fmt.Printf("Fan-in: worker %d output drained\n", workerOutput+1)
        }(i, input)
    }
    
    // Close output when all inputs are drained
    go func() {
        wg.Wait()
        close(output)
        fmt.Println("Fan-in: all worker outputs collected")
    }()
    
    return output
}

// Priority-based fan-out
func priorityFanOut(input <-chan WorkItem, highPriorityWorkers, normalWorkers int) (high, normal <-chan Result) {
    highPriorityChan := make(chan WorkItem, 10)
    normalPriorityChan := make(chan WorkItem, 10)
    
    // Distribute work based on priority
    go func() {
        defer close(highPriorityChan)
        defer close(normalPriorityChan)
        
        for item := range input {
            if item.Priority >= 2 {
                fmt.Printf("Router: sending high priority item %d\n", item.ID)
                highPriorityChan <- item
            } else {
                fmt.Printf("Router: sending normal priority item %d\n", item.ID)
                normalPriorityChan <- item
            }
        }
        
        fmt.Println("Router: finished distributing work")
    }()
    
    // Create worker pools for each priority
    highWorkers := fanOut(highPriorityChan, highPriorityWorkers)
    normalWorkers := fanOut(normalPriorityChan, normalWorkers)
    
    // Fan-in results from each priority pool
    high = fanIn(highWorkers...)
    normal = fanIn(normalWorkers...)
    
    return high, normal
}

// Load balancing fan-out
func loadBalancingFanOut(input <-chan WorkItem, numWorkers int) <-chan Result {
    // Create individual channels for each worker
    workerChannels := make([]chan WorkItem, numWorkers)
    for i := range workerChannels {
        workerChannels[i] = make(chan WorkItem, 5) // Small buffer per worker
    }
    
    // Load balancer goroutine
    go func() {
        defer func() {
            for _, ch := range workerChannels {
                close(ch)
            }
        }()
        
        currentWorker := 0
        for item := range input {
            // Round-robin distribution
            fmt.Printf("Load balancer: assigning item %d to worker %d\n", 
                      item.ID, currentWorker+1)
            workerChannels[currentWorker] <- item
            currentWorker = (currentWorker + 1) % numWorkers
        }
        
        fmt.Println("Load balancer: finished distributing work")
    }()
    
    // Create workers for each channel
    outputs := make([]<-chan Result, numWorkers)
    for i, workChan := range workerChannels {
        output := make(chan Result)
        outputs[i] = output
        
        go func(workerID int, work <-chan WorkItem, results chan<- Result) {
            defer close(results)
            
            for item := range work {
                start := time.Now()
                
                // Variable processing time to show load balancing
                processingTime := time.Duration(rand.Intn(200)+100) * time.Millisecond
                time.Sleep(processingTime)
                
                results <- Result{
                    WorkerID: workerID,
                    Item:     item,
                    Output:   fmt.Sprintf("LoadBalanced-%s", item.Data),
                    Duration: time.Since(start),
                }
            }
        }(i+1, workChan, output)
    }
    
    return fanIn(outputs...)
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    fmt.Println("Demonstrating Fan-in Fan-out Pattern")
    
    // Example 1: Basic fan-out fan-in
    fmt.Println("\nExample 1: Basic fan-out fan-in")
    
    workItems := make(chan WorkItem, 10)
    
    // Generate work items
    go func() {
        defer close(workItems)
        
        for i := 1; i <= 8; i++ {
            item := WorkItem{
                ID:       i,
                Data:     fmt.Sprintf("task-%d", i),
                Priority: rand.Intn(3),
            }
            
            fmt.Printf("Generator: created item %d (priority %d)\n", item.ID, item.Priority)
            workItems <- item
        }
        
        fmt.Println("Generator: finished creating work items")
    }()
    
    // Fan-out to 3 workers, then fan-in
    workerOutputs := fanOut(workItems, 3)
    results := fanIn(workerOutputs...)
    
    // Collect all results
    var allResults []Result
    for result := range results {
        allResults = append(allResults, result)
        fmt.Printf("Main: collected result from worker %d for item %d\n", 
                  result.WorkerID, result.Item.ID)
    }
    
    fmt.Printf("Basic fan-out fan-in completed: %d results\n", len(allResults))
    
    fmt.Println()
    
    // Example 2: Priority-based processing
    fmt.Println("Example 2: Priority-based fan-out")
    
    priorityWork := make(chan WorkItem, 15)
    
    go func() {
        defer close(priorityWork)
        
        priorities := []int{0, 1, 2, 0, 2, 1, 2, 0, 2, 1}
        for i, priority := range priorities {
            item := WorkItem{
                ID:       i + 1,
                Data:     fmt.Sprintf("priority-task-%d", i+1),
                Priority: priority,
            }
            priorityWork <- item
        }
    }()
    
    // Use priority-based fan-out
    highPriorityResults, normalPriorityResults := priorityFanOut(priorityWork, 2, 3)
    
    // Collect results from both priority streams
    var wg sync.WaitGroup
    var highResults, normalResults []Result
    var mu sync.Mutex
    
    wg.Add(2)
    
    go func() {
        defer wg.Done()
        for result := range highPriorityResults {
            mu.Lock()
            highResults = append(highResults, result)
            mu.Unlock()
            fmt.Printf("High priority result: item %d\n", result.Item.ID)
        }
    }()
    
    go func() {
        defer wg.Done()
        for result := range normalPriorityResults {
            mu.Lock()
            normalResults = append(normalResults, result)
            mu.Unlock()
            fmt.Printf("Normal priority result: item %d\n", result.Item.ID)
        }
    }()
    
    wg.Wait()
    
    fmt.Printf("Priority processing completed: %d high priority, %d normal priority\n", 
              len(highResults), len(normalResults))
    
    fmt.Println()
    
    // Example 3: Load-balanced processing
    fmt.Println("Example 3: Load-balanced fan-out")
    
    balancedWork := make(chan WorkItem, 12)
    
    go func() {
        defer close(balancedWork)
        
        for i := 1; i <= 12; i++ {
            item := WorkItem{
                ID:       i,
                Data:     fmt.Sprintf("balanced-task-%d", i),
                Priority: 1,
            }
            balancedWork <- item
        }
    }()
    
    balancedResults := loadBalancingFanOut(balancedWork, 4)
    
    // Collect and analyze load distribution
    workerLoad := make(map[int]int)
    totalProcessingTime := time.Duration(0)
    
    for result := range balancedResults {
        workerLoad[result.WorkerID]++
        totalProcessingTime += result.Duration
        fmt.Printf("Load balanced result: item %d processed by worker %d in %v\n", 
                  result.Item.ID, result.WorkerID, result.Duration)
    }
    
    fmt.Printf("Load balancing completed:\n")
    for workerID, count := range workerLoad {
        fmt.Printf("  Worker %d: processed %d items\n", workerID, count)
    }
    fmt.Printf("Total processing time across all workers: %v\n", totalProcessingTime)
    
    fmt.Println("Fan-in fan-out demonstration completed")
}
```

Fan-in and fan-out patterns enable scalable parallel processing by distributing  
work across multiple goroutines and efficiently collecting results. These  
patterns are fundamental for building high-throughput concurrent systems.  

## Semaphore pattern

Semaphores limit the number of concurrent operations, providing flow control  
and preventing resource exhaustion in high-load scenarios.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Semaphore implementation using buffered channel
type Semaphore chan struct{}

func NewSemaphore(capacity int) Semaphore {
    return make(chan struct{}, capacity)
}

func (s Semaphore) Acquire() {
    s <- struct{}{}
}

func (s Semaphore) Release() {
    <-s
}

func (s Semaphore) TryAcquire() bool {
    select {
    case s <- struct{}{}:
        return true
    default:
        return false
    }
}

// Resource-intensive operation
func processFile(filename string, sem Semaphore, wg *sync.WaitGroup) {
    defer wg.Done()
    
    // Try to acquire semaphore
    fmt.Printf("File %s: waiting for available slot...\n", filename)
    sem.Acquire()
    defer sem.Release()
    
    fmt.Printf("File %s: processing started\n", filename)
    
    // Simulate file processing
    processingTime := time.Duration(300+len(filename)*10) * time.Millisecond
    time.Sleep(processingTime)
    
    fmt.Printf("File %s: processing completed (%v)\n", filename, processingTime)
}

// Database connection pool simulation
type ConnectionPool struct {
    semaphore Semaphore
    poolSize  int
    activeOps int
    mu        sync.Mutex
}

func NewConnectionPool(size int) *ConnectionPool {
    return &ConnectionPool{
        semaphore: NewSemaphore(size),
        poolSize:  size,
    }
}

func (pool *ConnectionPool) Execute(query string) error {
    fmt.Printf("Query '%s': requesting connection...\n", query)
    
    pool.semaphore.Acquire()
    defer pool.semaphore.Release()
    
    // Track active operations
    pool.mu.Lock()
    pool.activeOps++
    currentActive := pool.activeOps
    pool.mu.Unlock()
    
    defer func() {
        pool.mu.Lock()
        pool.activeOps--
        pool.mu.Unlock()
    }()
    
    fmt.Printf("Query '%s': connection acquired (active: %d/%d)\n", 
              query, currentActive, pool.poolSize)
    
    // Simulate database operation
    time.Sleep(time.Duration(200+len(query)*5) * time.Millisecond)
    
    fmt.Printf("Query '%s': executed successfully\n", query)
    return nil
}

// Rate limiter using semaphore with time-based refill
type RateLimiter struct {
    semaphore Semaphore
    refillRate time.Duration
    capacity   int
}

func NewRateLimiter(capacity int, refillRate time.Duration) *RateLimiter {
    rl := &RateLimiter{
        semaphore:  NewSemaphore(capacity),
        refillRate: refillRate,
        capacity:   capacity,
    }
    
    // Fill initial capacity
    for i := 0; i < capacity; i++ {
        rl.semaphore.Release()
    }
    
    // Start refill goroutine
    go rl.refill()
    
    return rl
}

func (rl *RateLimiter) refill() {
    ticker := time.NewTicker(rl.refillRate)
    defer ticker.Stop()
    
    for range ticker.C {
        select {
        case rl.semaphore <- struct{}{}:
            // Token added
        default:
            // Semaphore full, skip
        }
    }
}

func (rl *RateLimiter) Allow() bool {
    select {
    case <-rl.semaphore:
        return true
    default:
        return false
    }
}

func (rl *RateLimiter) Wait() {
    <-rl.semaphore
}

func main() {
    fmt.Println("Demonstrating Semaphore Pattern")
    
    // Example 1: File processing with limited concurrency
    fmt.Println("\nExample 1: File processing semaphore")
    
    // Allow only 3 concurrent file processing operations
    fileSemaphore := NewSemaphore(3)
    var wg sync.WaitGroup
    
    files := []string{
        "document1.pdf", "image2.jpg", "video3.mp4", "archive4.zip",
        "database5.db", "config6.yaml", "log7.txt", "backup8.tar.gz",
    }
    
    fmt.Printf("Processing %d files with max 3 concurrent operations\n", len(files))
    
    for _, filename := range files {
        wg.Add(1)
        go processFile(filename, fileSemaphore, &wg)
    }
    
    wg.Wait()
    fmt.Println("All files processed")
    
    fmt.Println()
    
    // Example 2: Database connection pool
    fmt.Println("Example 2: Database connection pool")
    
    pool := NewConnectionPool(2) // Only 2 concurrent connections
    
    queries := []string{
        "SELECT * FROM users",
        "UPDATE products SET price = 100",
        "INSERT INTO orders (user_id) VALUES (1)",
        "DELETE FROM temp_data",
        "SELECT COUNT(*) FROM logs",
    }
    
    wg = sync.WaitGroup{}
    for _, query := range queries {
        wg.Add(1)
        go func(q string) {
            defer wg.Done()
            pool.Execute(q)
        }(query)
    }
    
    wg.Wait()
    fmt.Println("All database queries completed")
    
    fmt.Println()
    
    // Example 3: Rate limiter
    fmt.Println("Example 3: Rate limiter")
    
    // Allow 1 operation every 500ms, with burst capacity of 3
    rateLimiter := NewRateLimiter(3, 500*time.Millisecond)
    
    // Simulate API requests
    for i := 1; i <= 8; i++ {
        go func(requestID int) {
            if rateLimiter.Allow() {
                fmt.Printf("Request %d: allowed immediately\n", requestID)
            } else {
                fmt.Printf("Request %d: rate limited, waiting...\n", requestID)
                rateLimiter.Wait()
                fmt.Printf("Request %d: allowed after waiting\n", requestID)
            }
            
            // Simulate API processing
            time.Sleep(100 * time.Millisecond)
            fmt.Printf("Request %d: completed\n", requestID)
        }(i)
    }
    
    time.Sleep(6 * time.Second) // Let rate limiter demo complete
    
    fmt.Println()
    
    // Example 4: Advanced semaphore with priority
    fmt.Println("Example 4: Priority semaphore")
    prioritySemaphoreDemo()
}

func prioritySemaphoreDemo() {
    type PriorityTask struct {
        ID       int
        Priority int // Higher number = higher priority
        Name     string
    }
    
    // Priority-aware semaphore using channels
    highPriority := make(chan PriorityTask, 10)
    normalPriority := make(chan PriorityTask, 10)
    semaphore := NewSemaphore(2) // Allow 2 concurrent operations
    
    // Task dispatcher
    go func() {
        tasks := []PriorityTask{
            {1, 1, "normal-task-1"},
            {2, 3, "high-task-1"},
            {3, 1, "normal-task-2"},
            {4, 3, "high-task-2"},
            {5, 2, "medium-task-1"},
            {6, 1, "normal-task-3"},
        }
        
        for _, task := range tasks {
            if task.Priority >= 3 {
                highPriority <- task
            } else {
                normalPriority <- task
            }
        }
        
        close(highPriority)
        close(normalPriority)
    }()
    
    // Priority processor
    processTask := func(task PriorityTask) {
        semaphore.Acquire()
        defer semaphore.Release()
        
        fmt.Printf("Priority task %s (ID: %d, Priority: %d): started\n", 
                  task.Name, task.ID, task.Priority)
        
        time.Sleep(time.Duration(200+task.Priority*50) * time.Millisecond)
        
        fmt.Printf("Priority task %s: completed\n", task.Name)
    }
    
    var wg sync.WaitGroup
    
    // High priority worker
    wg.Add(1)
    go func() {
        defer wg.Done()
        for task := range highPriority {
            wg.Add(1)
            go func(t PriorityTask) {
                defer wg.Done()
                processTask(t)
            }(task)
        }
    }()
    
    // Normal priority worker (only processes when no high priority tasks)
    wg.Add(1)
    go func() {
        defer wg.Done()
        for task := range normalPriority {
            // Small delay to let high priority tasks be processed first
            time.Sleep(50 * time.Millisecond)
            
            wg.Add(1)
            go func(t PriorityTask) {
                defer wg.Done()
                processTask(t)
            }(task)
        }
    }()
    
    wg.Wait()
    fmt.Println("Priority semaphore demo completed")
}
```

Semaphores provide essential flow control by limiting concurrent operations.  
They're crucial for preventing resource exhaustion, implementing connection  
pools, and managing system load under high-concurrency scenarios.  

## Barrier synchronization

Barriers synchronize multiple goroutines to wait for all participants  
to reach a certain point before any can proceed further.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Barrier implementation
type Barrier struct {
    n         int           // number of participants
    count     int           // current count
    mutex     sync.Mutex    // protects count
    cond      *sync.Cond    // condition variable for waiting
    generation int           // barrier generation (for reuse)
}

func NewBarrier(n int) *Barrier {
    b := &Barrier{n: n}
    b.cond = sync.NewCond(&b.mutex)
    return b
}

func (b *Barrier) Wait() {
    b.mutex.Lock()
    defer b.mutex.Unlock()
    
    generation := b.generation
    b.count++
    
    if b.count == b.n {
        // Last goroutine reaches barrier - wake up all others
        b.count = 0
        b.generation++
        b.cond.Broadcast()
    } else {
        // Wait until all goroutines reach the barrier
        for b.generation == generation {
            b.cond.Wait()
        }
    }
}

// Phased computation example
func phasedWorker(workerID int, phases []string, barrier *Barrier, results chan<- string) {
    for phaseNum, phase := range phases {
        // Do work for this phase
        workTime := time.Duration(rand.Intn(300)+100) * time.Millisecond
        
        fmt.Printf("Worker %d: starting phase %d (%s)\n", workerID, phaseNum+1, phase)
        time.Sleep(workTime)
        
        result := fmt.Sprintf("Worker%d-Phase%d-Result", workerID, phaseNum+1)
        results <- result
        
        fmt.Printf("Worker %d: completed phase %d, waiting at barrier\n", workerID, phaseNum+1)
        
        // Wait for all workers to complete this phase
        barrier.Wait()
        
        fmt.Printf("Worker %d: barrier released, proceeding to next phase\n", workerID)
    }
    
    fmt.Printf("Worker %d: all phases completed\n", workerID)
}

// Simulation phase synchronization
type SimulationWorker struct {
    ID       int
    Position [2]float64 // x, y coordinates
    Velocity [2]float64 // vx, vy
}

func (w *SimulationWorker) updatePosition(deltaTime float64) {
    w.Position[0] += w.Velocity[0] * deltaTime
    w.Position[1] += w.Velocity[1] * deltaTime
}

func (w *SimulationWorker) computeForces(others []*SimulationWorker) {
    // Simple attraction/repulsion simulation
    for _, other := range others {
        if other.ID == w.ID {
            continue
        }
        
        dx := other.Position[0] - w.Position[0]
        dy := other.Position[1] - w.Position[1]
        
        // Simple force calculation (artificial)
        force := 0.01
        w.Velocity[0] += dx * force
        w.Velocity[1] += dy * force
    }
}

func simulationStep(worker *SimulationWorker, others []*SimulationWorker, 
                   stepBarrier, forceBarrier *Barrier, step int) {
    
    fmt.Printf("Sim Worker %d: step %d - computing forces\n", worker.ID, step)
    
    // Phase 1: Compute forces based on current positions
    worker.computeForces(others)
    
    // Wait for all workers to finish force computation
    fmt.Printf("Sim Worker %d: waiting at force barrier (step %d)\n", worker.ID, step)
    forceBarrier.Wait()
    
    fmt.Printf("Sim Worker %d: step %d - updating position\n", worker.ID, step)
    
    // Phase 2: Update positions based on computed forces
    worker.updatePosition(0.1) // deltaTime = 0.1
    
    // Wait for all workers to finish position update
    fmt.Printf("Sim Worker %d: waiting at step barrier (step %d)\n", worker.ID, step)
    stepBarrier.Wait()
    
    fmt.Printf("Sim Worker %d: step %d completed - pos(%.2f, %.2f)\n", 
              worker.ID, step, worker.Position[0], worker.Position[1])
}

// Tournament-style computation
func tournamentWorker(workerID int, data []int, barrier *Barrier, 
                     results chan<- int, round int) int {
    
    fmt.Printf("Tournament Worker %d: round %d processing data %v\n", 
              workerID, round, data)
    
    // Compute local result
    localResult := 0
    for _, value := range data {
        localResult += value * value // Sum of squares
    }
    
    processingTime := time.Duration(rand.Intn(200)+100) * time.Millisecond
    time.Sleep(processingTime)
    
    fmt.Printf("Tournament Worker %d: round %d result = %d\n", 
              workerID, round, localResult)
    
    results <- localResult
    
    // Wait for all workers in this round
    fmt.Printf("Tournament Worker %d: waiting at round %d barrier\n", workerID, round)
    barrier.Wait()
    
    return localResult
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    fmt.Println("Demonstrating Barrier Synchronization")
    
    // Example 1: Phased computation
    fmt.Println("\nExample 1: Phased computation with barriers")
    
    numWorkers := 4
    phases := []string{"initialize", "compute", "validate", "finalize"}
    barrier := NewBarrier(numWorkers)
    results := make(chan string, numWorkers*len(phases))
    
    var wg sync.WaitGroup
    
    for i := 1; i <= numWorkers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            phasedWorker(id, phases, barrier, results)
        }(i)
    }
    
    // Collect results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    var phaseResults []string
    for result := range results {
        phaseResults = append(phaseResults, result)
    }
    
    fmt.Printf("Phased computation completed: %d results collected\n", len(phaseResults))
    
    fmt.Println()
    
    // Example 2: Parallel simulation with synchronized steps
    fmt.Println("Example 2: Physics simulation with barrier synchronization")
    
    numSimWorkers := 3
    numSteps := 4
    
    // Create simulation workers
    simWorkers := make([]*SimulationWorker, numSimWorkers)
    for i := 0; i < numSimWorkers; i++ {
        simWorkers[i] = &SimulationWorker{
            ID:       i + 1,
            Position: [2]float64{rand.Float64() * 10, rand.Float64() * 10},
            Velocity: [2]float64{rand.Float64() - 0.5, rand.Float64() - 0.5},
        }
    }
    
    stepBarrier := NewBarrier(numSimWorkers)
    forceBarrier := NewBarrier(numSimWorkers)
    
    wg = sync.WaitGroup{}
    
    for _, worker := range simWorkers {
        wg.Add(1)
        go func(w *SimulationWorker) {
            defer wg.Done()
            
            for step := 1; step <= numSteps; step++ {
                simulationStep(w, simWorkers, stepBarrier, forceBarrier, step)
            }
        }(worker)
    }
    
    wg.Wait()
    
    fmt.Println("Final simulation state:")
    for _, worker := range simWorkers {
        fmt.Printf("Worker %d: position(%.2f, %.2f), velocity(%.2f, %.2f)\n",
                  worker.ID, worker.Position[0], worker.Position[1],
                  worker.Velocity[0], worker.Velocity[1])
    }
    
    fmt.Println()
    
    // Example 3: Tournament-style reduction
    fmt.Println("Example 3: Tournament reduction with barriers")
    
    // Initial data for each worker
    initialData := [][]int{
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12},
        {13, 14, 15, 16},
    }
    
    currentWorkers := len(initialData)
    round := 1
    results = make(chan int, currentWorkers)
    
    for currentWorkers > 1 {
        fmt.Printf("Tournament round %d: %d workers\n", round, currentWorkers)
        
        roundBarrier := NewBarrier(currentWorkers)
        wg = sync.WaitGroup{}
        
        for i := 0; i < currentWorkers; i++ {
            wg.Add(1)
            go func(workerID int, data []int) {
                defer wg.Done()
                tournamentWorker(workerID+1, data, roundBarrier, results, round)
            }(i, initialData[i])
        }
        
        // Wait for round to complete and collect results
        wg.Wait()
        
        // Collect results for next round
        var nextRoundData [][]int
        for i := 0; i < currentWorkers; i++ {
            result := <-results
            nextRoundData = append(nextRoundData, []int{result})
        }
        
        // Combine adjacent results for next round
        initialData = nil
        for i := 0; i < len(nextRoundData); i += 2 {
            if i+1 < len(nextRoundData) {
                combined := append(nextRoundData[i], nextRoundData[i+1]...)
                initialData = append(initialData, combined)
            } else {
                initialData = append(initialData, nextRoundData[i])
            }
        }
        
        currentWorkers = len(initialData)
        round++
    }
    
    if len(initialData) > 0 {
        fmt.Printf("Tournament final result: %v\n", initialData[0])
    }
    
    fmt.Println("Barrier synchronization demonstration completed")
}
```

Barriers ensure that all participants in a parallel computation reach  
synchronization points together. They're essential for phased algorithms,  
simulations, and any scenario requiring lockstep execution coordination.  

## Publisher-Subscriber pattern

Pub-Sub enables decoupled communication where publishers send messages  
to topics and subscribers receive messages based on their interests.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Message represents a published message
type Message struct {
    Topic     string
    Content   interface{}
    Timestamp time.Time
    Publisher string
}

// Subscriber interface
type Subscriber interface {
    OnMessage(msg Message)
    ID() string
}

// Simple subscriber implementation
type SimpleSubscriber struct {
    id       string
    msgCount int
    mu       sync.Mutex
}

func NewSimpleSubscriber(id string) *SimpleSubscriber {
    return &SimpleSubscriber{id: id}
}

func (s *SimpleSubscriber) OnMessage(msg Message) {
    s.mu.Lock()
    s.msgCount++
    count := s.msgCount
    s.mu.Unlock()
    
    fmt.Printf("[%s] Received message #%d on topic '%s': %v (from %s)\n",
               s.id, count, msg.Topic, msg.Content, msg.Publisher)
}

func (s *SimpleSubscriber) ID() string {
    return s.id
}

func (s *SimpleSubscriber) MessageCount() int {
    s.mu.Lock()
    defer s.mu.Unlock()
    return s.msgCount
}

// Topic-specific subscriber
type TopicSubscriber struct {
    id            string
    interestedTopics []string
    messageHistory   []Message
    mu            sync.RWMutex
}

func NewTopicSubscriber(id string, topics ...string) *TopicSubscriber {
    return &TopicSubscriber{
        id:               id,
        interestedTopics: topics,
    }
}

func (ts *TopicSubscriber) OnMessage(msg Message) {
    // Check if interested in this topic
    interested := false
    for _, topic := range ts.interestedTopics {
        if topic == msg.Topic {
            interested = true
            break
        }
    }
    
    if !interested {
        return
    }
    
    ts.mu.Lock()
    ts.messageHistory = append(ts.messageHistory, msg)
    ts.mu.Unlock()
    
    fmt.Printf("[%s] Topic subscriber received '%s': %v\n", 
               ts.id, msg.Topic, msg.Content)
}

func (ts *TopicSubscriber) ID() string {
    return ts.id
}

func (ts *TopicSubscriber) GetHistory() []Message {
    ts.mu.RLock()
    defer ts.mu.RUnlock()
    
    history := make([]Message, len(ts.messageHistory))
    copy(history, ts.messageHistory)
    return history
}

// PubSub broker
type PubSub struct {
    subscribers map[string]map[string]Subscriber // topic -> subscriber_id -> subscriber
    mu          sync.RWMutex
}

func NewPubSub() *PubSub {
    return &PubSub{
        subscribers: make(map[string]map[string]Subscriber),
    }
}

func (ps *PubSub) Subscribe(topic string, subscriber Subscriber) {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    if ps.subscribers[topic] == nil {
        ps.subscribers[topic] = make(map[string]Subscriber)
    }
    
    ps.subscribers[topic][subscriber.ID()] = subscriber
    fmt.Printf("Subscriber '%s' subscribed to topic '%s'\n", subscriber.ID(), topic)
}

func (ps *PubSub) Unsubscribe(topic string, subscriberID string) {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    if topicSubs, exists := ps.subscribers[topic]; exists {
        delete(topicSubs, subscriberID)
        if len(topicSubs) == 0 {
            delete(ps.subscribers, topic)
        }
        fmt.Printf("Subscriber '%s' unsubscribed from topic '%s'\n", subscriberID, topic)
    }
}

func (ps *PubSub) Publish(topic, publisher string, content interface{}) {
    ps.mu.RLock()
    topicSubs := ps.subscribers[topic]
    ps.mu.RUnlock()
    
    if len(topicSubs) == 0 {
        fmt.Printf("No subscribers for topic '%s'\n", topic)
        return
    }
    
    msg := Message{
        Topic:     topic,
        Content:   content,
        Timestamp: time.Now(),
        Publisher: publisher,
    }
    
    fmt.Printf("Publishing to topic '%s': %v (publisher: %s)\n", 
               topic, content, publisher)
    
    // Send message to all subscribers concurrently
    var wg sync.WaitGroup
    for _, subscriber := range topicSubs {
        wg.Add(1)
        go func(sub Subscriber) {
            defer wg.Done()
            sub.OnMessage(msg)
        }(subscriber)
    }
    wg.Wait()
}

func (ps *PubSub) GetTopics() []string {
    ps.mu.RLock()
    defer ps.mu.RUnlock()
    
    topics := make([]string, 0, len(ps.subscribers))
    for topic := range ps.subscribers {
        topics = append(topics, topic)
    }
    return topics
}

func (ps *PubSub) GetSubscriberCount(topic string) int {
    ps.mu.RLock()
    defer ps.mu.RUnlock()
    
    if subs, exists := ps.subscribers[topic]; exists {
        return len(subs)
    }
    return 0
}

// Advanced filtered subscriber
type FilteredSubscriber struct {
    id       string
    filter   func(Message) bool
    messages []Message
    mu       sync.Mutex
}

func NewFilteredSubscriber(id string, filter func(Message) bool) *FilteredSubscriber {
    return &FilteredSubscriber{
        id:     id,
        filter: filter,
    }
}

func (fs *FilteredSubscriber) OnMessage(msg Message) {
    if !fs.filter(msg) {
        return
    }
    
    fs.mu.Lock()
    fs.messages = append(fs.messages, msg)
    count := len(fs.messages)
    fs.mu.Unlock()
    
    fmt.Printf("[%s] Filtered subscriber received message #%d: %v\n", 
               fs.id, count, msg.Content)
}

func (fs *FilteredSubscriber) ID() string {
    return fs.id
}

func (fs *FilteredSubscriber) GetMessages() []Message {
    fs.mu.Lock()
    defer fs.mu.Unlock()
    
    messages := make([]Message, len(fs.messages))
    copy(messages, fs.messages)
    return messages
}

func main() {
    fmt.Println("Demonstrating Publisher-Subscriber Pattern")
    
    pubsub := NewPubSub()
    
    // Example 1: Basic pub-sub
    fmt.Println("\nExample 1: Basic publish-subscribe")
    
    sub1 := NewSimpleSubscriber("subscriber-1")
    sub2 := NewSimpleSubscriber("subscriber-2")
    
    pubsub.Subscribe("news", sub1)
    pubsub.Subscribe("news", sub2)
    pubsub.Subscribe("sports", sub1)
    
    pubsub.Publish("news", "NewsBot", "Breaking: Go 1.23 released!")
    pubsub.Publish("sports", "SportsBot", "Team A wins championship!")
    pubsub.Publish("news", "NewsBot", "New concurrency patterns discovered")
    
    time.Sleep(100 * time.Millisecond)
    
    fmt.Printf("Subscriber 1 received %d messages\n", sub1.MessageCount())
    fmt.Printf("Subscriber 2 received %d messages\n", sub2.MessageCount())
    
    fmt.Println()
    
    // Example 2: Topic-specific subscribers
    fmt.Println("Example 2: Topic-specific subscribers")
    
    techSub := NewTopicSubscriber("tech-enthusiast", "technology", "programming")
    businessSub := NewTopicSubscriber("business-reader", "business", "finance")
    generalSub := NewTopicSubscriber("general-reader", "news", "technology", "business")
    
    pubsub.Subscribe("technology", techSub)
    pubsub.Subscribe("programming", techSub)
    pubsub.Subscribe("business", businessSub)
    pubsub.Subscribe("finance", businessSub)
    pubsub.Subscribe("news", generalSub)
    pubsub.Subscribe("technology", generalSub)
    pubsub.Subscribe("business", generalSub)
    
    // Publish to various topics
    topics := []struct {
        topic   string
        content string
    }{
        {"technology", "New AI breakthrough announced"},
        {"programming", "Go 1.23 features unveiled"},
        {"business", "Market reaches new highs"},
        {"finance", "Cryptocurrency trends analysis"},
        {"news", "Global summit concludes successfully"},
    }
    
    for _, item := range topics {
        pubsub.Publish(item.topic, "ContentBot", item.content)
        time.Sleep(50 * time.Millisecond)
    }
    
    fmt.Println("\nMessage histories:")
    for _, sub := range []*TopicSubscriber{techSub, businessSub, generalSub} {
        history := sub.GetHistory()
        fmt.Printf("%s received %d messages\n", sub.ID(), len(history))
    }
    
    fmt.Println()
    
    // Example 3: Filtered subscribers
    fmt.Println("Example 3: Filtered subscribers")
    
    // Filter for high-priority messages
    highPriorityFilter := func(msg Message) bool {
        if content, ok := msg.Content.(map[string]interface{}); ok {
            if priority, exists := content["priority"]; exists {
                if priorityStr, ok := priority.(string); ok {
                    return priorityStr == "high" || priorityStr == "critical"
                }
            }
        }
        return false
    }
    
    // Filter for specific publisher
    techPublisherFilter := func(msg Message) bool {
        return msg.Publisher == "TechNews"
    }
    
    prioritySub := NewFilteredSubscriber("priority-alerts", highPriorityFilter)
    techSub2 := NewFilteredSubscriber("tech-only", techPublisherFilter)
    
    pubsub.Subscribe("alerts", prioritySub)
    pubsub.Subscribe("technology", techSub2)
    pubsub.Subscribe("news", techSub2)
    
    // Publish messages with metadata
    messages := []struct {
        topic     string
        publisher string
        content   map[string]interface{}
    }{
        {"alerts", "SystemBot", map[string]interface{}{
            "message":  "System maintenance scheduled",
            "priority": "low",
        }},
        {"alerts", "SecurityBot", map[string]interface{}{
            "message":  "Security breach detected",
            "priority": "critical",
        }},
        {"technology", "TechNews", map[string]interface{}{
            "message":  "New framework released",
            "priority": "medium",
        }},
        {"news", "GeneralNews", map[string]interface{}{
            "message":  "Weather update",
            "priority": "low",
        }},
        {"news", "TechNews", map[string]interface{}{
            "message":  "Tech conference announced",
            "priority": "medium",
        }},
    }
    
    for _, msg := range messages {
        pubsub.Publish(msg.topic, msg.publisher, msg.content)
        time.Sleep(50 * time.Millisecond)
    }
    
    fmt.Println("\nFiltered results:")
    fmt.Printf("Priority subscriber received %d high-priority messages\n", 
              len(prioritySub.GetMessages()))
    fmt.Printf("Tech publisher subscriber received %d messages from TechNews\n", 
              len(techSub2.GetMessages()))
    
    fmt.Println()
    
    // Example 4: Dynamic subscription management
    fmt.Println("Example 4: Dynamic subscription management")
    
    dynamicSub := NewSimpleSubscriber("dynamic-subscriber")
    
    // Subscribe to multiple topics
    topics2 := []string{"updates", "notifications", "alerts"}
    for _, topic := range topics2 {
        pubsub.Subscribe(topic, dynamicSub)
    }
    
    // Publish some messages
    pubsub.Publish("updates", "UpdateBot", "System updated")
    pubsub.Publish("notifications", "NotifyBot", "New message received")
    
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("Dynamic subscriber received %d messages\n", dynamicSub.MessageCount())
    
    // Unsubscribe from some topics
    pubsub.Unsubscribe("updates", dynamicSub.ID())
    pubsub.Unsubscribe("alerts", dynamicSub.ID())
    
    // Publish more messages
    pubsub.Publish("updates", "UpdateBot", "Another update")
    pubsub.Publish("notifications", "NotifyBot", "Another notification")
    pubsub.Publish("alerts", "AlertBot", "System alert")
    
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("After unsubscribing, dynamic subscriber received %d total messages\n", 
              dynamicSub.MessageCount())
    
    // Show current topic statistics
    fmt.Println("\nCurrent topic statistics:")
    for _, topic := range pubsub.GetTopics() {
        count := pubsub.GetSubscriberCount(topic)
        fmt.Printf("Topic '%s': %d subscribers\n", topic, count)
    }
    
    fmt.Println("Publisher-Subscriber pattern demonstration completed")
}
```

The Publisher-Subscriber pattern enables loose coupling between message  
producers and consumers. It's essential for event-driven architectures,  
notification systems, and building scalable, decoupled applications.  

## Circuit breaker pattern

Circuit breakers prevent cascading failures by temporarily stopping calls  
to failing services, allowing them time to recover before retrying.  

```go
package main

import (
    "errors"
    "fmt"
    "sync"
    "time"
)

// Circuit breaker states
type State int

const (
    Closed State = iota
    Open
    HalfOpen
)

func (s State) String() string {
    switch s {
    case Closed:
        return "CLOSED"
    case Open:
        return "OPEN"
    case HalfOpen:
        return "HALF_OPEN"
    default:
        return "UNKNOWN"
    }
}

// Circuit breaker configuration
type Config struct {
    FailureThreshold int           // Number of failures to open circuit
    RecoveryTimeout  time.Duration // Time to wait before trying half-open
    SuccessThreshold int           // Consecutive successes to close circuit
    Timeout          time.Duration // Request timeout
}

// Circuit breaker implementation
type CircuitBreaker struct {
    config       Config
    state        State
    failures     int
    successes    int
    lastFailTime time.Time
    mu           sync.RWMutex
}

func NewCircuitBreaker(config Config) *CircuitBreaker {
    return &CircuitBreaker{
        config: config,
        state:  Closed,
    }
}

// Execute function with circuit breaker protection
func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    fmt.Printf("Circuit breaker: state=%s, failures=%d, successes=%d\n", 
              cb.state, cb.failures, cb.successes)
    
    if cb.state == Open {
        // Check if recovery timeout has passed
        if time.Since(cb.lastFailTime) > cb.config.RecoveryTimeout {
            fmt.Println("Circuit breaker: transitioning to HALF_OPEN")
            cb.state = HalfOpen
            cb.successes = 0
        } else {
            return errors.New("circuit breaker is OPEN")
        }
    }
    
    // Execute the function with timeout
    resultChan := make(chan error, 1)
    
    go func() {
        resultChan <- fn()
    }()
    
    select {
    case err := <-resultChan:
        return cb.handleResult(err)
    case <-time.After(cb.config.Timeout):
        return cb.handleResult(errors.New("request timeout"))
    }
}

func (cb *CircuitBreaker) handleResult(err error) error {
    if err != nil {
        cb.onFailure()
        return err
    }
    
    cb.onSuccess()
    return nil
}

func (cb *CircuitBreaker) onFailure() {
    cb.failures++
    cb.lastFailTime = time.Now()
    
    fmt.Printf("Circuit breaker: failure recorded (total: %d)\n", cb.failures)
    
    if cb.state == HalfOpen || cb.failures >= cb.config.FailureThreshold {
        fmt.Printf("Circuit breaker: opening circuit (threshold: %d)\n", 
                  cb.config.FailureThreshold)
        cb.state = Open
        cb.successes = 0
    }
}

func (cb *CircuitBreaker) onSuccess() {
    cb.failures = 0
    
    if cb.state == HalfOpen {
        cb.successes++
        fmt.Printf("Circuit breaker: success in HALF_OPEN (count: %d)\n", cb.successes)
        
        if cb.successes >= cb.config.SuccessThreshold {
            fmt.Println("Circuit breaker: closing circuit")
            cb.state = Closed
            cb.successes = 0
        }
    } else {
        fmt.Println("Circuit breaker: success recorded")
    }
}

func (cb *CircuitBreaker) GetState() State {
    cb.mu.RLock()
    defer cb.mu.RUnlock()
    return cb.state
}

// Simulated service that can fail
type UnreliableService struct {
    name         string
    failureRate  float64 // 0.0 to 1.0
    responseTime time.Duration
}

func (s *UnreliableService) Call() error {
    fmt.Printf("Service %s: processing request\n", s.name)
    
    // Simulate processing time
    time.Sleep(s.responseTime)
    
    // Simulate random failures
    if time.Now().UnixNano()%100 < int64(s.failureRate*100) {
        return fmt.Errorf("service %s failed", s.name)
    }
    
    fmt.Printf("Service %s: request successful\n", s.name)
    return nil
}

// Multi-service circuit breaker manager
type ServiceManager struct {
    breakers map[string]*CircuitBreaker
    mu       sync.RWMutex
}

func NewServiceManager() *ServiceManager {
    return &ServiceManager{
        breakers: make(map[string]*CircuitBreaker),
    }
}

func (sm *ServiceManager) RegisterService(name string, config Config) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    sm.breakers[name] = NewCircuitBreaker(config)
    fmt.Printf("Service manager: registered service '%s'\n", name)
}

func (sm *ServiceManager) CallService(serviceName string, fn func() error) error {
    sm.mu.RLock()
    breaker, exists := sm.breakers[serviceName]
    sm.mu.RUnlock()
    
    if !exists {
        return fmt.Errorf("service '%s' not registered", serviceName)
    }
    
    return breaker.Execute(fn)
}

func (sm *ServiceManager) GetServiceStates() map[string]State {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    
    states := make(map[string]State)
    for name, breaker := range sm.breakers {
        states[name] = breaker.GetState()
    }
    return states
}

func main() {
    fmt.Println("Demonstrating Circuit Breaker Pattern")
    
    // Example 1: Basic circuit breaker
    fmt.Println("\nExample 1: Basic circuit breaker behavior")
    
    config := Config{
        FailureThreshold: 3,
        RecoveryTimeout:  2 * time.Second,
        SuccessThreshold: 2,
        Timeout:          500 * time.Millisecond,
    }
    
    breaker := NewCircuitBreaker(config)
    service := &UnreliableService{
        name:         "TestService",
        failureRate:  0.7, // 70% failure rate
        responseTime: 100 * time.Millisecond,
    }
    
    // Test circuit breaker behavior
    fmt.Println("Testing circuit breaker with unreliable service (70% failure rate)")
    
    for i := 1; i <= 12; i++ {
        fmt.Printf("\n--- Request %d ---\n", i)
        
        err := breaker.Execute(func() error {
            return service.Call()
        })
        
        if err != nil {
            fmt.Printf("Request %d failed: %v\n", i, err)
        } else {
            fmt.Printf("Request %d succeeded\n", i)
        }
        
        fmt.Printf("Circuit breaker state: %s\n", breaker.GetState())
        time.Sleep(300 * time.Millisecond)
    }
    
    fmt.Println()
    
    // Example 2: Service recovery simulation
    fmt.Println("Example 2: Service recovery simulation")
    
    // Create a service that will recover after some time
    recoveringService := &UnreliableService{
        name:         "RecoveringService",
        failureRate:  0.9, // Start with high failure rate
        responseTime: 150 * time.Millisecond,
    }
    
    config2 := Config{
        FailureThreshold: 2,
        RecoveryTimeout:  1 * time.Second,
        SuccessThreshold: 3,
        Timeout:          400 * time.Millisecond,
    }
    
    breaker2 := NewCircuitBreaker(config2)
    
    // Goroutine to simulate service recovery
    go func() {
        time.Sleep(3 * time.Second)
        fmt.Println("\n*** Service is recovering... ***")
        recoveringService.failureRate = 0.1 // Improve to 90% success rate
    }()
    
    // Test service with recovery
    for i := 1; i <= 15; i++ {
        fmt.Printf("\n--- Recovery Test %d ---\n", i)
        
        err := breaker2.Execute(func() error {
            return recoveringService.Call()
        })
        
        if err != nil {
            fmt.Printf("Recovery test %d failed: %v\n", i, err)
        } else {
            fmt.Printf("Recovery test %d succeeded\n", i)
        }
        
        time.Sleep(500 * time.Millisecond)
    }
    
    fmt.Println()
    
    // Example 3: Multiple services with service manager
    fmt.Println("Example 3: Multiple services with circuit breakers")
    
    manager := NewServiceManager()
    
    // Register different services with different configurations
    services := map[string]Config{
        "database": {
            FailureThreshold: 5,
            RecoveryTimeout:  3 * time.Second,
            SuccessThreshold: 2,
            Timeout:          1 * time.Second,
        },
        "cache": {
            FailureThreshold: 2,
            RecoveryTimeout:  1 * time.Second,
            SuccessThreshold: 1,
            Timeout:          200 * time.Millisecond,
        },
        "external_api": {
            FailureThreshold: 3,
            RecoveryTimeout:  5 * time.Second,
            SuccessThreshold: 3,
            Timeout:          800 * time.Millisecond,
        },
    }
    
    for serviceName, config := range services {
        manager.RegisterService(serviceName, config)
    }
    
    // Create service implementations
    serviceImpls := map[string]*UnreliableService{
        "database": {
            name:         "Database",
            failureRate:  0.3,
            responseTime: 200 * time.Millisecond,
        },
        "cache": {
            name:         "Cache",
            failureRate:  0.1,
            responseTime: 50 * time.Millisecond,
        },
        "external_api": {
            name:         "ExternalAPI",
            failureRate:  0.6,
            responseTime: 400 * time.Millisecond,
        },
    }
    
    // Simulate concurrent requests to different services
    var wg sync.WaitGroup
    
    for serviceName, serviceImpl := range serviceImpls {
        wg.Add(1)
        go func(svcName string, svc *UnreliableService) {
            defer wg.Done()
            
            for i := 1; i <= 8; i++ {
                err := manager.CallService(svcName, func() error {
                    return svc.Call()
                })
                
                if err != nil {
                    fmt.Printf("Service %s request %d failed: %v\n", svcName, i, err)
                } else {
                    fmt.Printf("Service %s request %d succeeded\n", svcName, i)
                }
                
                time.Sleep(400 * time.Millisecond)
            }
        }(serviceName, serviceImpl)
    }
    
    wg.Wait()
    
    // Display final service states
    fmt.Println("\nFinal service circuit breaker states:")
    states := manager.GetServiceStates()
    for serviceName, state := range states {
        fmt.Printf("Service '%s': %s\n", serviceName, state)
    }
    
    fmt.Println("Circuit breaker pattern demonstration completed")
}
```

Circuit breakers protect distributed systems from cascading failures.  
They monitor service health and prevent calls to failing services,  
allowing systems to gracefully degrade and recover automatically.  

## Real-world web crawler

A concurrent web crawler demonstrates practical application of goroutines,  
channels, rate limiting, and coordination in a realistic scenario.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// URL represents a web page to crawl
type URL struct {
    Address string
    Depth   int
}

// Page represents crawled content
type Page struct {
    URL       string
    Content   string
    Links     []string
    Error     error
    Timestamp time.Time
}

// Crawler configuration
type CrawlerConfig struct {
    MaxWorkers    int
    MaxDepth      int
    MaxPages      int
    RateLimit     time.Duration
    RequestTimeout time.Duration
}

// Web crawler implementation
type WebCrawler struct {
    config      CrawlerConfig
    urlQueue    chan URL
    results     chan Page
    visited     map[string]bool
    visitedMu   sync.RWMutex
    semaphore   chan struct{}
    pageCount   int
    pageCountMu sync.Mutex
    wg          sync.WaitGroup
    done        chan struct{}
}

func NewWebCrawler(config CrawlerConfig) *WebCrawler {
    return &WebCrawler{
        config:    config,
        urlQueue:  make(chan URL, config.MaxWorkers*2),
        results:   make(chan Page, config.MaxWorkers),
        visited:   make(map[string]bool),
        semaphore: make(chan struct{}, config.MaxWorkers),
        done:      make(chan struct{}),
    }
}

// Simulate fetching a web page
func (wc *WebCrawler) fetchPage(url string) Page {
    fmt.Printf("Crawler: fetching %s\n", url)
    
    // Simulate network delay
    time.Sleep(wc.config.RateLimit)
    
    // Simulate random failures
    if rand.Float64() < 0.1 { // 10% failure rate
        return Page{
            URL:       url,
            Error:     fmt.Errorf("failed to fetch %s", url),
            Timestamp: time.Now(),
        }
    }
    
    // Generate mock content and links
    content := fmt.Sprintf("Content from %s - fetched at %v", 
                           url, time.Now().Format("15:04:05"))
    
    var links []string
    numLinks := rand.Intn(5) + 1 // 1-5 links per page
    for i := 0; i < numLinks; i++ {
        link := fmt.Sprintf("%s/page-%d", url, rand.Intn(10)+1)
        links = append(links, link)
    }
    
    return Page{
        URL:       url,
        Content:   content,
        Links:     links,
        Timestamp: time.Now(),
    }
}

// Worker goroutine
func (wc *WebCrawler) worker(workerID int) {
    defer wc.wg.Done()
    
    fmt.Printf("Worker %d: starting\n", workerID)
    
    for {
        select {
        case url := <-wc.urlQueue:
            // Check if we should stop
            select {
            case <-wc.done:
                return
            default:
            }
            
            // Check if already visited
            wc.visitedMu.Lock()
            if wc.visited[url.Address] {
                wc.visitedMu.Unlock()
                continue
            }
            wc.visited[url.Address] = true
            wc.visitedMu.Unlock()
            
            // Check page limit
            wc.pageCountMu.Lock()
            if wc.pageCount >= wc.config.MaxPages {
                wc.pageCountMu.Unlock()
                return
            }
            wc.pageCount++
            currentCount := wc.pageCount
            wc.pageCountMu.Unlock()
            
            fmt.Printf("Worker %d: processing %s (page %d/%d, depth %d)\n", 
                      workerID, url.Address, currentCount, 
                      wc.config.MaxPages, url.Depth)
            
            // Fetch the page
            page := wc.fetchPage(url.Address)
            
            // Send result
            wc.results <- page
            
            // Add new URLs if within depth limit
            if url.Depth < wc.config.MaxDepth && page.Error == nil {
                for _, link := range page.Links {
                    select {
                    case wc.urlQueue <- URL{Address: link, Depth: url.Depth + 1}:
                    case <-wc.done:
                        return
                    default:
                        // Queue full, skip this link
                    }
                }
            }
            
        case <-wc.done:
            fmt.Printf("Worker %d: stopping\n", workerID)
            return
        }
    }
}

// Start crawling
func (wc *WebCrawler) Start(startURLs []string) <-chan Page {
    // Add initial URLs
    for _, url := range startURLs {
        wc.urlQueue <- URL{Address: url, Depth: 0}
    }
    
    // Start workers
    for i := 1; i <= wc.config.MaxWorkers; i++ {
        wc.wg.Add(1)
        go wc.worker(i)
    }
    
    // Monitor completion
    go func() {
        defer close(wc.results)
        
        ticker := time.NewTicker(2 * time.Second)
        defer ticker.Stop()
        
        for {
            select {
            case <-ticker.C:
                wc.pageCountMu.Lock()
                count := wc.pageCount
                wc.pageCountMu.Unlock()
                
                if count >= wc.config.MaxPages {
                    fmt.Printf("Crawler: reached page limit (%d), stopping...\n", count)
                    close(wc.done)
                    wc.wg.Wait()
                    return
                }
                
                // Check if queue is empty and no more work
                if len(wc.urlQueue) == 0 {
                    fmt.Println("Crawler: no more URLs to process, stopping...")
                    close(wc.done)
                    wc.wg.Wait()
                    return
                }
                
            case <-time.After(10 * time.Second):
                fmt.Println("Crawler: timeout reached, stopping...")
                close(wc.done)
                wc.wg.Wait()
                return
            }
        }
    }()
    
    return wc.results
}

// Crawler statistics
type CrawlStats struct {
    TotalPages      int
    SuccessfulPages int
    FailedPages     int
    UniqueURLs      int
    CrawlDuration   time.Duration
    PagesPerSecond  float64
}

// Result collector
func collectResults(results <-chan Page) CrawlStats {
    startTime := time.Now()
    stats := CrawlStats{}
    
    urlSet := make(map[string]bool)
    
    for page := range results {
        stats.TotalPages++
        urlSet[page.URL] = true
        
        if page.Error != nil {
            stats.FailedPages++
            fmt.Printf("Results: ERROR - %s: %v\n", page.URL, page.Error)
        } else {
            stats.SuccessfulPages++
            fmt.Printf("Results: SUCCESS - %s (%d links found)\n", 
                      page.URL, len(page.Links))
        }
    }
    
    stats.UniqueURLs = len(urlSet)
    stats.CrawlDuration = time.Since(startTime)
    stats.PagesPerSecond = float64(stats.TotalPages) / stats.CrawlDuration.Seconds()
    
    return stats
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    fmt.Println("Demonstrating Real-world Web Crawler")
    
    // Example 1: Basic web crawling
    fmt.Println("\nExample 1: Basic web crawler")
    
    config := CrawlerConfig{
        MaxWorkers:     3,
        MaxDepth:       2,
        MaxPages:       15,
        RateLimit:      200 * time.Millisecond,
        RequestTimeout: 2 * time.Second,
    }
    
    crawler := NewWebCrawler(config)
    
    startURLs := []string{
        "https://example.com",
        "https://test.com",
    }
    
    fmt.Printf("Starting crawl with %d workers, max depth %d, max pages %d\n",
              config.MaxWorkers, config.MaxDepth, config.MaxPages)
    
    results := crawler.Start(startURLs)
    stats := collectResults(results)
    
    fmt.Printf("\nCrawl completed:\n")
    fmt.Printf("  Total pages: %d\n", stats.TotalPages)
    fmt.Printf("  Successful: %d\n", stats.SuccessfulPages)
    fmt.Printf("  Failed: %d\n", stats.FailedPages)
    fmt.Printf("  Unique URLs: %d\n", stats.UniqueURLs)
    fmt.Printf("  Duration: %v\n", stats.CrawlDuration)
    fmt.Printf("  Pages/second: %.2f\n", stats.PagesPerSecond)
    
    fmt.Println()
    
    // Example 2: High-throughput crawling
    fmt.Println("Example 2: High-throughput web crawler")
    
    config2 := CrawlerConfig{
        MaxWorkers:     8,
        MaxDepth:       3,
        MaxPages:       25,
        RateLimit:      100 * time.Millisecond,
        RequestTimeout: 1 * time.Second,
    }
    
    crawler2 := NewWebCrawler(config2)
    
    startURLs2 := []string{
        "https://site1.com",
        "https://site2.com",
        "https://site3.com",
    }
    
    fmt.Printf("High-throughput crawl: %d workers, max depth %d, max pages %d\n",
              config2.MaxWorkers, config2.MaxDepth, config2.MaxPages)
    
    results2 := crawler2.Start(startURLs2)
    stats2 := collectResults(results2)
    
    fmt.Printf("\nHigh-throughput crawl completed:\n")
    fmt.Printf("  Total pages: %d\n", stats2.TotalPages)
    fmt.Printf("  Successful: %d\n", stats2.SuccessfulPages)
    fmt.Printf("  Failed: %d\n", stats2.FailedPages)
    fmt.Printf("  Duration: %v\n", stats2.CrawlDuration)
    fmt.Printf("  Pages/second: %.2f\n", stats2.PagesPerSecond)
    
    fmt.Println()
    
    // Example 3: Comparison of different configurations
    fmt.Println("Example 3: Configuration comparison")
    
    configs := []struct {
        name   string
        config CrawlerConfig
    }{
        {
            "Conservative",
            CrawlerConfig{MaxWorkers: 2, MaxDepth: 2, MaxPages: 10, RateLimit: 500 * time.Millisecond},
        },
        {
            "Balanced",
            CrawlerConfig{MaxWorkers: 4, MaxDepth: 2, MaxPages: 10, RateLimit: 250 * time.Millisecond},
        },
        {
            "Aggressive",
            CrawlerConfig{MaxWorkers: 6, MaxDepth: 2, MaxPages: 10, RateLimit: 100 * time.Millisecond},
        },
    }
    
    for _, cfg := range configs {
        fmt.Printf("\nTesting %s configuration:\n", cfg.name)
        
        crawler := NewWebCrawler(cfg.config)
        startTime := time.Now()
        
        results := crawler.Start([]string{"https://benchmark.com"})
        stats := collectResults(results)
        
        fmt.Printf("%s results: %d pages in %v (%.2f pages/sec)\n",
                  cfg.name, stats.TotalPages, stats.CrawlDuration, stats.PagesPerSecond)
    }
    
    fmt.Println("Web crawler demonstration completed")
}
```

This web crawler demonstrates real-world concurrency patterns including  
worker pools, rate limiting, graceful shutdown, and coordination between  
multiple goroutines processing a shared queue of work items.  

## Performance monitoring system

A monitoring system showcases advanced concurrency patterns for collecting,  
aggregating, and reporting metrics from multiple sources concurrently.  

```go
package main

import (
    "fmt"
    "math/rand"
    "sort"
    "sync"
    "sync/atomic"
    "time"
)

// Metric represents a single measurement
type Metric struct {
    Name      string
    Value     float64
    Tags      map[string]string
    Timestamp time.Time
    Source    string
}

// MetricCollector interface for different metric sources
type MetricCollector interface {
    Name() string
    Collect() []Metric
    Start(chan<- []Metric)
    Stop()
}

// CPU usage collector
type CPUCollector struct {
    name     string
    interval time.Duration
    stop     chan struct{}
    wg       sync.WaitGroup
}

func NewCPUCollector(interval time.Duration) *CPUCollector {
    return &CPUCollector{
        name:     "cpu",
        interval: interval,
        stop:     make(chan struct{}),
    }
}

func (c *CPUCollector) Name() string {
    return c.name
}

func (c *CPUCollector) Collect() []Metric {
    // Simulate CPU metrics collection
    return []Metric{
        {
            Name:      "cpu.usage.percent",
            Value:     rand.Float64() * 100,
            Tags:      map[string]string{"core": "0"},
            Timestamp: time.Now(),
            Source:    c.name,
        },
        {
            Name:      "cpu.usage.percent", 
            Value:     rand.Float64() * 100,
            Tags:      map[string]string{"core": "1"},
            Timestamp: time.Now(),
            Source:    c.name,
        },
    }
}

func (c *CPUCollector) Start(output chan<- []Metric) {
    c.wg.Add(1)
    go func() {
        defer c.wg.Done()
        
        ticker := time.NewTicker(c.interval)
        defer ticker.Stop()
        
        fmt.Printf("CPUCollector: started (interval: %v)\n", c.interval)
        
        for {
            select {
            case <-ticker.C:
                metrics := c.Collect()
                select {
                case output <- metrics:
                    fmt.Printf("CPUCollector: sent %d metrics\n", len(metrics))
                case <-c.stop:
                    return
                }
            case <-c.stop:
                fmt.Println("CPUCollector: stopping")
                return
            }
        }
    }()
}

func (c *CPUCollector) Stop() {
    close(c.stop)
    c.wg.Wait()
}

// Memory usage collector
type MemoryCollector struct {
    name     string
    interval time.Duration
    stop     chan struct{}
    wg       sync.WaitGroup
}

func NewMemoryCollector(interval time.Duration) *MemoryCollector {
    return &MemoryCollector{
        name:     "memory",
        interval: interval,
        stop:     make(chan struct{}),
    }
}

func (m *MemoryCollector) Name() string {
    return m.name
}

func (m *MemoryCollector) Collect() []Metric {
    // Simulate memory metrics
    totalMem := 8 * 1024 * 1024 * 1024 // 8GB
    usedMem := rand.Float64() * float64(totalMem) * 0.8 // Up to 80% usage
    
    return []Metric{
        {
            Name:      "memory.used.bytes",
            Value:     usedMem,
            Tags:      map[string]string{"type": "physical"},
            Timestamp: time.Now(),
            Source:    m.name,
        },
        {
            Name:      "memory.usage.percent",
            Value:     (usedMem / float64(totalMem)) * 100,
            Tags:      map[string]string{"type": "physical"},
            Timestamp: time.Now(),
            Source:    m.name,
        },
    }
}

func (m *MemoryCollector) Start(output chan<- []Metric) {
    m.wg.Add(1)
    go func() {
        defer m.wg.Done()
        
        ticker := time.NewTicker(m.interval)
        defer ticker.Stop()
        
        fmt.Printf("MemoryCollector: started (interval: %v)\n", m.interval)
        
        for {
            select {
            case <-ticker.C:
                metrics := m.Collect()
                select {
                case output <- metrics:
                    fmt.Printf("MemoryCollector: sent %d metrics\n", len(metrics))
                case <-m.stop:
                    return
                }
            case <-m.stop:
                fmt.Println("MemoryCollector: stopping")
                return
            }
        }
    }()
}

func (m *MemoryCollector) Stop() {
    close(m.stop)
    m.wg.Wait()
}

// Disk I/O collector
type DiskCollector struct {
    name     string
    interval time.Duration
    stop     chan struct{}
    wg       sync.WaitGroup
}

func NewDiskCollector(interval time.Duration) *DiskCollector {
    return &DiskCollector{
        name:     "disk",
        interval: interval,
        stop:     make(chan struct{}),
    }
}

func (d *DiskCollector) Name() string {
    return d.name
}

func (d *DiskCollector) Collect() []Metric {
    // Simulate disk metrics
    return []Metric{
        {
            Name:      "disk.read.bytes_per_sec",
            Value:     rand.Float64() * 1000000, // Up to 1MB/s
            Tags:      map[string]string{"device": "sda1"},
            Timestamp: time.Now(),
            Source:    d.name,
        },
        {
            Name:      "disk.write.bytes_per_sec",
            Value:     rand.Float64() * 500000, // Up to 500KB/s
            Tags:      map[string]string{"device": "sda1"},
            Timestamp: time.Now(),
            Source:    d.name,
        },
    }
}

func (d *DiskCollector) Start(output chan<- []Metric) {
    d.wg.Add(1)
    go func() {
        defer d.wg.Done()
        
        ticker := time.NewTicker(d.interval)
        defer ticker.Stop()
        
        fmt.Printf("DiskCollector: started (interval: %v)\n", d.interval)
        
        for {
            select {
            case <-ticker.C:
                metrics := d.Collect()
                select {
                case output <- metrics:
                    fmt.Printf("DiskCollector: sent %d metrics\n", len(metrics))
                case <-d.stop:
                    return
                }
            case <-d.stop:
                fmt.Println("DiskCollector: stopping")
                return
            }
        }
    }()
}

func (d *DiskCollector) Stop() {
    close(d.stop)
    d.wg.Wait()
}

// Metric aggregator
type MetricAggregator struct {
    metrics    []Metric
    mu         sync.RWMutex
    totalCount int64
}

func NewMetricAggregator() *MetricAggregator {
    return &MetricAggregator{}
}

func (ma *MetricAggregator) AddMetrics(metrics []Metric) {
    ma.mu.Lock()
    defer ma.mu.Unlock()
    
    ma.metrics = append(ma.metrics, metrics...)
    atomic.AddInt64(&ma.totalCount, int64(len(metrics)))
    
    fmt.Printf("Aggregator: added %d metrics (total: %d)\n", 
              len(metrics), atomic.LoadInt64(&ma.totalCount))
}

func (ma *MetricAggregator) GetMetricsByName(name string) []Metric {
    ma.mu.RLock()
    defer ma.mu.RUnlock()
    
    var result []Metric
    for _, metric := range ma.metrics {
        if metric.Name == name {
            result = append(result, metric)
        }
    }
    
    return result
}

func (ma *MetricAggregator) GetSummary() map[string]interface{} {
    ma.mu.RLock()
    defer ma.mu.RUnlock()
    
    metricCounts := make(map[string]int)
    sourceCounts := make(map[string]int)
    
    for _, metric := range ma.metrics {
        metricCounts[metric.Name]++
        sourceCounts[metric.Source]++
    }
    
    return map[string]interface{}{
        "total_metrics": len(ma.metrics),
        "metric_types":  metricCounts,
        "source_counts": sourceCounts,
    }
}

// Performance monitoring system
type MonitoringSystem struct {
    collectors []MetricCollector
    aggregator *MetricAggregator
    input      chan []Metric
    stop       chan struct{}
    wg         sync.WaitGroup
}

func NewMonitoringSystem() *MonitoringSystem {
    return &MonitoringSystem{
        aggregator: NewMetricAggregator(),
        input:      make(chan []Metric, 100),
        stop:       make(chan struct{}),
    }
}

func (ms *MonitoringSystem) AddCollector(collector MetricCollector) {
    ms.collectors = append(ms.collectors, collector)
    fmt.Printf("MonitoringSystem: added collector '%s'\n", collector.Name())
}

func (ms *MonitoringSystem) Start() {
    fmt.Println("MonitoringSystem: starting all collectors")
    
    // Start metric aggregation goroutine
    ms.wg.Add(1)
    go func() {
        defer ms.wg.Done()
        
        for {
            select {
            case metrics := <-ms.input:
                ms.aggregator.AddMetrics(metrics)
            case <-ms.stop:
                fmt.Println("MonitoringSystem: stopping aggregation")
                return
            }
        }
    }()
    
    // Start all collectors
    for _, collector := range ms.collectors {
        collector.Start(ms.input)
    }
    
    // Start reporting goroutine
    ms.wg.Add(1)
    go ms.reportingLoop()
}

func (ms *MonitoringSystem) reportingLoop() {
    defer ms.wg.Done()
    
    ticker := time.NewTicker(3 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            ms.generateReport()
        case <-ms.stop:
            fmt.Println("MonitoringSystem: stopping reporting")
            return
        }
    }
}

func (ms *MonitoringSystem) generateReport() {
    summary := ms.aggregator.GetSummary()
    
    fmt.Println("\n=== MONITORING REPORT ===")
    fmt.Printf("Total metrics collected: %v\n", summary["total_metrics"])
    
    if metricTypes, ok := summary["metric_types"].(map[string]int); ok {
        fmt.Println("Metric types:")
        
        // Sort metric names for consistent output
        var names []string
        for name := range metricTypes {
            names = append(names, name)
        }
        sort.Strings(names)
        
        for _, name := range names {
            count := metricTypes[name]
            
            // Get recent values for this metric
            metrics := ms.aggregator.GetMetricsByName(name)
            if len(metrics) > 0 {
                latest := metrics[len(metrics)-1]
                fmt.Printf("  %s: %d samples, latest=%.2f\n", 
                          name, count, latest.Value)
            }
        }
    }
    
    if sourceCounts, ok := summary["source_counts"].(map[string]int); ok {
        fmt.Println("Sources:")
        for source, count := range sourceCounts {
            fmt.Printf("  %s: %d metrics\n", source, count)
        }
    }
    
    fmt.Println("========================\n")
}

func (ms *MonitoringSystem) Stop() {
    fmt.Println("MonitoringSystem: initiating shutdown")
    
    // Stop all collectors
    for _, collector := range ms.collectors {
        collector.Stop()
    }
    
    // Stop internal goroutines
    close(ms.stop)
    ms.wg.Wait()
    
    fmt.Println("MonitoringSystem: shutdown complete")
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    fmt.Println("Demonstrating Performance Monitoring System")
    
    // Create monitoring system
    system := NewMonitoringSystem()
    
    // Add collectors with different intervals
    system.AddCollector(NewCPUCollector(1 * time.Second))
    system.AddCollector(NewMemoryCollector(2 * time.Second))
    system.AddCollector(NewDiskCollector(1500 * time.Millisecond))
    
    // Start the monitoring system
    system.Start()
    
    fmt.Println("Monitoring system running... (will run for 15 seconds)")
    
    // Let the system run
    time.Sleep(15 * time.Second)
    
    // Stop the system
    system.Stop()
    
    // Final report
    fmt.Println("Generating final summary report...")
    system.generateReport()
    
    fmt.Println("Performance monitoring system demonstration completed")
}
```

This monitoring system demonstrates enterprise-grade concurrency patterns  
including multiple data collectors, metric aggregation, concurrent processing,  
and coordinated shutdown - essential patterns for production systems.  

## Conclusion

Go's concurrency model provides powerful primitives for building scalable,  
efficient concurrent applications. The examples demonstrate fundamental  
patterns from basic goroutines to sophisticated real-world systems.  

Key takeaways:
- Goroutines enable lightweight concurrent execution
- Channels provide safe communication between goroutines  
- Select statements enable sophisticated coordination patterns
- Synchronization primitives complement channel-based communication
- Context enables cancellation and deadline management
- Real-world systems combine multiple patterns effectively

These patterns form the foundation for building robust, concurrent Go  
applications that can handle high loads and complex coordination requirements  
while maintaining code clarity and correctness.  
