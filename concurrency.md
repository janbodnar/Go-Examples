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
