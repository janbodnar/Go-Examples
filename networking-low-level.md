# Low-Level Networking in Go

Low-level networking is the foundation of modern distributed systems, enabling  
direct control over network communication at the socket and protocol level.  
Unlike high-level HTTP or RPC frameworks, low-level networking provides complete  
control over data transmission, protocol design, and network behavior. This  
makes it essential for building custom protocols, implementing performance-  
critical systems, and understanding how network communication works under  
the hood.

Go's networking capabilities are built on a robust foundation provided by the  
`net` package, which offers both high-level and low-level primitives for  
network programming. The package abstracts platform-specific details while  
still providing access to raw sockets, TCP/UDP operations, and custom protocol  
implementations. This design allows developers to write portable network code  
that works consistently across different operating systems.

Socket programming forms the core of low-level networking. A socket is an  
endpoint for network communication, providing a bidirectional channel for  
data exchange between processes on the same machine or across networks. Go  
supports various socket types including TCP for reliable, connection-oriented  
communication, UDP for connectionless datagram transmission, and Unix domain  
sockets for efficient local inter-process communication.

TCP (Transmission Control Protocol) provides reliable, ordered, and error-  
checked delivery of data streams. It handles connection establishment through  
three-way handshake, flow control, congestion control, and ensures data  
arrives in the correct order. TCP is ideal for applications where data  
integrity and reliability are more important than speed.

UDP (User Datagram Protocol) offers a simpler, connectionless communication  
model without reliability guarantees. It's faster than TCP but doesn't ensure  
delivery, ordering, or error correction. UDP is perfect for real-time  
applications like video streaming, gaming, or DNS where speed matters more  
than guaranteed delivery.

Raw socket programming allows direct access to network layer protocols,  
enabling custom packet construction and protocol implementation. While more  
complex, raw sockets provide complete control over network communication,  
making them essential for network tools, security applications, and custom  
protocol development.

Stream manipulation is crucial for efficient network programming. It involves  
buffering strategies, data transformation, chunking, and handling partial  
reads and writes. Proper stream handling ensures optimal performance and  
correct protocol implementation.

This comprehensive guide contains 99 practical Go code examples demonstrating  
low-level networking operations. Each example builds upon previous concepts,  
progressing from basic socket setup to advanced custom protocol implementation,  
following Go best practices and idiomatic conventions.

## Basic TCP Server

A TCP server listens for incoming connections and handles client requests.  
This example demonstrates the fundamental pattern for creating a TCP server.  

```go
package main

import (
    "fmt"
    "log"
    "net"
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Failed to create listener: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("TCP server listening on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Failed to accept connection: %v", err)
            continue
        }
        
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    defer conn.Close()
    
    addr := conn.RemoteAddr()
    fmt.Printf("New connection from %s\n", addr)
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Printf("Error reading from %s: %v", addr, err)
        return
    }
    
    fmt.Printf("Received %d bytes: %s\n", n, string(buffer[:n]))
    
    _, err = conn.Write([]byte("Message received\n"))
    if err != nil {
        log.Printf("Error writing to %s: %v", addr, err)
    }
}
```

The TCP server uses `net.Listen` to create a listener on port 8080. It accepts  
connections in a loop, spawning a new goroutine for each client to handle  
concurrent connections efficiently. The server reads data from the client,  
logs it, and sends a response back.  

## Basic TCP Client

A TCP client connects to a server and exchanges data. This example shows  
how to establish a connection, send data, and receive responses.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
)

func main() {
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Failed to connect: %v", err)
    }
    defer conn.Close()
    
    fmt.Printf("Connected to %s\n", conn.RemoteAddr())
    
    message := "Hello from TCP client\n"
    _, err = conn.Write([]byte(message))
    if err != nil {
        log.Fatalf("Failed to send data: %v", err)
    }
    
    fmt.Printf("Sent: %s", message)
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil && err != io.EOF {
        log.Fatalf("Failed to read response: %v", err)
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
}
```

The client uses `net.Dial` to establish a TCP connection to the server. It  
sends a message using `conn.Write`, then reads the server's response. Proper  
error handling and connection cleanup with `defer` ensure reliable operation.  

## TCP Server with Multiple Handlers

Concurrent connection handling allows a server to process multiple clients  
simultaneously without blocking.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "sync"
    "time"
)

var (
    connCount int
    mu        sync.Mutex
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Failed to create listener: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Multi-client TCP server listening on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        mu.Lock()
        connCount++
        clientID := connCount
        mu.Unlock()
        
        go handleClient(conn, clientID)
    }
}

func handleClient(conn net.Conn, clientID int) {
    defer conn.Close()
    
    fmt.Printf("Client %d connected from %s\n", clientID, conn.RemoteAddr())
    
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        line := scanner.Text()
        fmt.Printf("Client %d: %s\n", clientID, line)
        
        response := fmt.Sprintf("Echo from server: %s\n", line)
        _, err := conn.Write([]byte(response))
        if err != nil {
            log.Printf("Client %d write error: %v", clientID, err)
            break
        }
    }
    
    if err := scanner.Err(); err != nil {
        log.Printf("Client %d scanner error: %v", clientID, err)
    }
    
    fmt.Printf("Client %d disconnected\n", clientID)
}
```

This server handles multiple simultaneous connections by launching a goroutine  
for each client. A mutex protects the connection counter to safely assign  
unique IDs. The `bufio.Scanner` simplifies line-based protocol handling.  

## TCP Connection with Timeout

Setting timeouts prevents indefinite blocking and ensures responsive network  
applications.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    timeout := 5 * time.Second
    
    conn, err := net.DialTimeout("tcp", "localhost:8080", timeout)
    if err != nil {
        log.Fatalf("Connection failed: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("Connected with timeout")
    
    err = conn.SetDeadline(time.Now().Add(timeout))
    if err != nil {
        log.Fatalf("Failed to set deadline: %v", err)
    }
    
    _, err = conn.Write([]byte("Quick message\n"))
    if err != nil {
        log.Fatalf("Write failed: %v", err)
    }
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
            log.Printf("Read timeout: %v", err)
            return
        }
        log.Fatalf("Read failed: %v", err)
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
}
```

`net.DialTimeout` establishes connections with a timeout. `SetDeadline` applies  
timeouts to both read and write operations. Checking for timeout errors allows  
appropriate handling of network delays.  

## TCP Server with Read/Write Timeouts

Server-side timeouts prevent slow clients from consuming resources indefinitely.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Server with timeouts listening on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go handleWithTimeout(conn)
    }
}

func handleWithTimeout(conn net.Conn) {
    defer conn.Close()
    
    readTimeout := 10 * time.Second
    writeTimeout := 5 * time.Second
    
    fmt.Printf("Client connected: %s\n", conn.RemoteAddr())
    
    reader := bufio.NewReader(conn)
    
    for {
        conn.SetReadDeadline(time.Now().Add(readTimeout))
        
        line, err := reader.ReadString('\n')
        if err != nil {
            if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
                log.Printf("Read timeout for %s", conn.RemoteAddr())
            } else {
                log.Printf("Read error: %v", err)
            }
            break
        }
        
        fmt.Printf("Received: %s", line)
        
        conn.SetWriteDeadline(time.Now().Add(writeTimeout))
        
        response := fmt.Sprintf("Echo: %s", line)
        _, err = conn.Write([]byte(response))
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
    }
    
    fmt.Printf("Client disconnected: %s\n", conn.RemoteAddr())
}
```

This server sets separate read and write timeouts for each operation. Timeouts  
are reset before each I/O operation to ensure responsive handling while  
preventing resource exhaustion from slow or stalled clients.  

## Basic UDP Server

UDP servers receive datagrams without establishing connections, making them  
suitable for stateless communication.  

```go
package main

import (
    "fmt"
    "log"
    "net"
)

func main() {
    addr, err := net.ResolveUDPAddr("udp", ":8080")
    if err != nil {
        log.Fatalf("Failed to resolve address: %v", err)
    }
    
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("UDP server listening on :8080")
    
    buffer := make([]byte, 1024)
    
    for {
        n, clientAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            log.Printf("Read error: %v", err)
            continue
        }
        
        fmt.Printf("Received %d bytes from %s: %s\n", 
            n, clientAddr, string(buffer[:n]))
        
        response := []byte("UDP response\n")
        _, err = conn.WriteToUDP(response, clientAddr)
        if err != nil {
            log.Printf("Write error: %v", err)
        }
    }
}
```

UDP servers use `ListenUDP` to create a connection and `ReadFromUDP` to receive  
datagrams. Each message includes the sender's address, allowing the server to  
reply using `WriteToUDP`. No connection state is maintained between messages.  

## Basic UDP Client

UDP clients send datagrams without establishing a connection, suitable for  
fire-and-forget or query-response patterns.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    serverAddr, err := net.ResolveUDPAddr("udp", "localhost:8080")
    if err != nil {
        log.Fatalf("Failed to resolve address: %v", err)
    }
    
    conn, err := net.DialUDP("udp", nil, serverAddr)
    if err != nil {
        log.Fatalf("Failed to dial: %v", err)
    }
    defer conn.Close()
    
    fmt.Printf("Sending to %s\n", serverAddr)
    
    message := []byte("Hello UDP server\n")
    _, err = conn.Write(message)
    if err != nil {
        log.Fatalf("Failed to send: %v", err)
    }
    
    fmt.Printf("Sent: %s", message)
    
    conn.SetReadDeadline(time.Now().Add(5 * time.Second))
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Fatalf("Failed to receive: %v", err)
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
}
```

The UDP client uses `DialUDP` to associate with a server address, then sends  
datagrams with `Write`. A read timeout prevents indefinite waiting. UDP  
doesn't guarantee delivery or ordering of messages.  

## TCP Connection Pooling

Connection pools reuse TCP connections to reduce overhead and improve  
performance for frequent client-server communication.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "sync"
    "time"
)

type ConnectionPool struct {
    address string
    pool    chan net.Conn
    maxSize int
    mu      sync.Mutex
    count   int
}

func NewConnectionPool(address string, maxSize int) *ConnectionPool {
    return &ConnectionPool{
        address: address,
        pool:    make(chan net.Conn, maxSize),
        maxSize: maxSize,
    }
}

func (p *ConnectionPool) Get() (net.Conn, error) {
    select {
    case conn := <-p.pool:
        if conn == nil {
            return p.createConnection()
        }
        return conn, nil
    default:
        return p.createConnection()
    }
}

func (p *ConnectionPool) createConnection() (net.Conn, error) {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if p.count >= p.maxSize {
        return nil, fmt.Errorf("connection pool exhausted")
    }
    
    conn, err := net.DialTimeout("tcp", p.address, 5*time.Second)
    if err != nil {
        return nil, err
    }
    
    p.count++
    fmt.Printf("Created new connection (total: %d)\n", p.count)
    return conn, nil
}

func (p *ConnectionPool) Put(conn net.Conn) {
    if conn == nil {
        return
    }
    
    select {
    case p.pool <- conn:
    default:
        conn.Close()
        p.mu.Lock()
        p.count--
        p.mu.Unlock()
    }
}

func (p *ConnectionPool) Close() {
    close(p.pool)
    for conn := range p.pool {
        if conn != nil {
            conn.Close()
        }
    }
}

func main() {
    pool := NewConnectionPool("localhost:8080", 5)
    defer pool.Close()
    
    var wg sync.WaitGroup
    
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            conn, err := pool.Get()
            if err != nil {
                log.Printf("Worker %d: Failed to get connection: %v", id, err)
                return
            }
            defer pool.Put(conn)
            
            message := fmt.Sprintf("Request from worker %d\n", id)
            _, err = conn.Write([]byte(message))
            if err != nil {
                log.Printf("Worker %d: Write error: %v", id, err)
                return
            }
            
            buffer := make([]byte, 1024)
            n, err := conn.Read(buffer)
            if err != nil {
                log.Printf("Worker %d: Read error: %v", id, err)
                return
            }
            
            fmt.Printf("Worker %d received: %s", id, string(buffer[:n]))
            
            time.Sleep(100 * time.Millisecond)
        }(i)
    }
    
    wg.Wait()
}
```

The connection pool maintains a pool of reusable connections, creating new ones  
as needed up to a maximum limit. Connections are borrowed with `Get()` and  
returned with `Put()`. This reduces connection establishment overhead for  
applications making frequent requests.  

## TCP Keep-Alive Configuration

TCP keep-alive detects broken connections and prevents resource waste from  
dead connections.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Server with keep-alive on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go handleWithKeepAlive(conn)
    }
}

func handleWithKeepAlive(conn net.Conn) {
    defer conn.Close()
    
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        log.Println("Not a TCP connection")
        return
    }
    
    err := tcpConn.SetKeepAlive(true)
    if err != nil {
        log.Printf("Failed to enable keep-alive: %v", err)
        return
    }
    
    err = tcpConn.SetKeepAlivePeriod(30 * time.Second)
    if err != nil {
        log.Printf("Failed to set keep-alive period: %v", err)
        return
    }
    
    fmt.Printf("Connection from %s with keep-alive enabled\n", conn.RemoteAddr())
    
    buffer := make([]byte, 1024)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            log.Printf("Read error: %v", err)
            break
        }
        
        fmt.Printf("Received: %s", string(buffer[:n]))
        
        _, err = conn.Write(buffer[:n])
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
    }
}
```

Keep-alive probes detect dead connections by periodically sending packets.  
`SetKeepAlive` enables the feature, and `SetKeepAlivePeriod` configures the  
interval. This prevents accumulation of zombie connections.  

## Graceful Server Shutdown

Graceful shutdown ensures existing connections complete before the server  
stops, preventing data loss and connection errors.  

```go
package main

import (
    "context"
    "fmt"
    "log"
    "net"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)

type Server struct {
    listener net.Listener
    wg       sync.WaitGroup
    quit     chan struct{}
}

func NewServer(addr string) (*Server, error) {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        return nil, err
    }
    
    return &Server{
        listener: listener,
        quit:     make(chan struct{}),
    }, nil
}

func (s *Server) Start() {
    fmt.Printf("Server starting on %s\n", s.listener.Addr())
    
    go func() {
        for {
            select {
            case <-s.quit:
                return
            default:
                conn, err := s.listener.Accept()
                if err != nil {
                    select {
                    case <-s.quit:
                        return
                    default:
                        log.Printf("Accept error: %v", err)
                        continue
                    }
                }
                
                s.wg.Add(1)
                go s.handleConnection(conn)
            }
        }
    }()
}

func (s *Server) handleConnection(conn net.Conn) {
    defer s.wg.Done()
    defer conn.Close()
    
    fmt.Printf("New connection: %s\n", conn.RemoteAddr())
    
    buffer := make([]byte, 1024)
    for {
        conn.SetReadDeadline(time.Now().Add(5 * time.Second))
        
        n, err := conn.Read(buffer)
        if err != nil {
            if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
                continue
            }
            break
        }
        
        fmt.Printf("Received: %s", string(buffer[:n]))
        
        _, err = conn.Write(buffer[:n])
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
    }
    
    fmt.Printf("Connection closed: %s\n", conn.RemoteAddr())
}

func (s *Server) Shutdown(ctx context.Context) error {
    fmt.Println("Shutting down server...")
    
    close(s.quit)
    s.listener.Close()
    
    done := make(chan struct{})
    go func() {
        s.wg.Wait()
        close(done)
    }()
    
    select {
    case <-done:
        fmt.Println("All connections closed gracefully")
        return nil
    case <-ctx.Done():
        fmt.Println("Shutdown timeout exceeded")
        return ctx.Err()
    }
}

func main() {
    server, err := NewServer(":8080")
    if err != nil {
        log.Fatalf("Failed to create server: %v", err)
    }
    
    server.Start()
    
    sigChan := make(chan os.Signal, 1)
    signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
    
    <-sigChan
    
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()
    
    if err := server.Shutdown(ctx); err != nil {
        log.Printf("Shutdown error: %v", err)
    }
}
```

This server tracks active connections using a `WaitGroup` and provides graceful  
shutdown. On receiving a signal, it stops accepting new connections and waits  
for existing ones to finish, with a timeout to prevent indefinite waiting.  

## Buffered Reading and Writing

Buffered I/O reduces system calls and improves performance for network  
operations involving frequent small reads or writes.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Buffered I/O server on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go handleBuffered(conn)
    }
}

func handleBuffered(conn net.Conn) {
    defer conn.Close()
    
    reader := bufio.NewReader(conn)
    writer := bufio.NewWriter(conn)
    
    fmt.Printf("Client connected: %s\n", conn.RemoteAddr())
    
    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            log.Printf("Read error: %v", err)
            break
        }
        
        fmt.Printf("Received: %s", line)
        
        response := fmt.Sprintf("Echo: %s", line)
        _, err = writer.WriteString(response)
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        err = writer.Flush()
        if err != nil {
            log.Printf("Flush error: %v", err)
            break
        }
    }
}
```

`bufio.Reader` and `bufio.Writer` buffer network I/O operations, reducing the  
number of system calls. This is particularly beneficial for line-based  
protocols. Remember to call `Flush()` to ensure buffered data is sent.  

## Binary Protocol Implementation

Binary protocols offer compact data representation and efficient parsing  
compared to text-based protocols.  

```go
package main

import (
    "encoding/binary"
    "fmt"
    "io"
    "log"
    "net"
)

type Message struct {
    Length  uint32
    Type    uint8
    Payload []byte
}

func (m *Message) Encode(conn net.Conn) error {
    if err := binary.Write(conn, binary.BigEndian, m.Length); err != nil {
        return err
    }
    if err := binary.Write(conn, binary.BigEndian, m.Type); err != nil {
        return err
    }
    _, err := conn.Write(m.Payload)
    return err
}

func DecodeMessage(conn net.Conn) (*Message, error) {
    msg := &Message{}
    
    if err := binary.Read(conn, binary.BigEndian, &msg.Length); err != nil {
        return nil, err
    }
    
    if err := binary.Read(conn, binary.BigEndian, &msg.Type); err != nil {
        return nil, err
    }
    
    if msg.Length > 0 {
        msg.Payload = make([]byte, msg.Length)
        _, err := io.ReadFull(conn, msg.Payload)
        if err != nil {
            return nil, err
        }
    }
    
    return msg, nil
}

func main() {
    go startBinaryServer()
    
    // Wait for server to start
    waitForConnection(":8080")
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    payload := []byte("Binary message data")
    msg := &Message{
        Length:  uint32(len(payload)),
        Type:    1,
        Payload: payload,
    }
    
    fmt.Printf("Sending message: Type=%d, Length=%d\n", msg.Type, msg.Length)
    
    err = msg.Encode(conn)
    if err != nil {
        log.Fatalf("Encode failed: %v", err)
    }
    
    response, err := DecodeMessage(conn)
    if err != nil {
        log.Fatalf("Decode failed: %v", err)
    }
    
    fmt.Printf("Received: Type=%d, Payload=%s\n", 
        response.Type, string(response.Payload))
}

func startBinaryServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go func(c net.Conn) {
            defer c.Close()
            
            msg, err := DecodeMessage(c)
            if err != nil {
                return
            }
            
            response := &Message{
                Length:  uint32(len(msg.Payload)),
                Type:    msg.Type + 1,
                Payload: msg.Payload,
            }
            
            response.Encode(c)
        }(conn)
    }
}

func waitForConnection(addr string) {
    for i := 0; i < 10; i++ {
        conn, err := net.Dial("tcp", addr)
        if err == nil {
            conn.Close()
            return
        }
    }
}
```

Binary protocols use fixed-size headers with length and type fields, followed  
by variable payload data. The `encoding/binary` package handles byte order  
conversion, ensuring consistent interpretation across platforms.  

## Stream Chunking

Chunking breaks large data streams into manageable pieces for efficient  
transmission and processing.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
)

const ChunkSize = 1024

func sendChunked(conn net.Conn, data []byte) error {
    totalSize := len(data)
    chunks := (totalSize + ChunkSize - 1) / ChunkSize
    
    fmt.Printf("Sending %d bytes in %d chunks\n", totalSize, chunks)
    
    for i := 0; i < totalSize; i += ChunkSize {
        end := i + ChunkSize
        if end > totalSize {
            end = totalSize
        }
        
        chunk := data[i:end]
        _, err := conn.Write(chunk)
        if err != nil {
            return err
        }
        
        fmt.Printf("Sent chunk %d: %d bytes\n", i/ChunkSize+1, len(chunk))
    }
    
    return nil
}

func receiveChunked(conn net.Conn, expectedSize int) ([]byte, error) {
    received := make([]byte, 0, expectedSize)
    buffer := make([]byte, ChunkSize)
    
    for len(received) < expectedSize {
        n, err := conn.Read(buffer)
        if err != nil {
            if err == io.EOF && len(received) == expectedSize {
                break
            }
            return nil, err
        }
        
        received = append(received, buffer[:n]...)
        fmt.Printf("Received chunk: %d bytes (total: %d/%d)\n", 
            n, len(received), expectedSize)
    }
    
    return received, nil
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        data, err := receiveChunked(conn, 5000)
        if err != nil {
            log.Printf("Receive error: %v", err)
            return
        }
        
        fmt.Printf("Server received %d bytes total\n", len(data))
    }()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    largeData := make([]byte, 5000)
    for i := range largeData {
        largeData[i] = byte(i % 256)
    }
    
    err = sendChunked(conn, largeData)
    if err != nil {
        log.Fatalf("Send failed: %v", err)
    }
}
```

Chunking allows controlled data transmission, enabling progress tracking,  
memory-efficient handling of large transfers, and resumable uploads. The  
chunk size balances memory usage and transmission efficiency.  

## Connection State Management

Maintaining connection state enables stateful protocols and session management.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strings"
    "sync"
    "time"
)

type ClientState struct {
    ID          int
    Conn        net.Conn
    Username    string
    Authenticated bool
    LastActivity time.Time
}

type StateManager struct {
    clients map[int]*ClientState
    mu      sync.RWMutex
    nextID  int
}

func NewStateManager() *StateManager {
    return &StateManager{
        clients: make(map[int]*ClientState),
    }
}

func (sm *StateManager) AddClient(conn net.Conn) *ClientState {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    sm.nextID++
    state := &ClientState{
        ID:           sm.nextID,
        Conn:         conn,
        LastActivity: time.Now(),
    }
    
    sm.clients[sm.nextID] = state
    return state
}

func (sm *StateManager) RemoveClient(id int) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    
    delete(sm.clients, id)
}

func (sm *StateManager) GetClient(id int) *ClientState {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    
    return sm.clients[id]
}

func main() {
    manager := NewStateManager()
    
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Stateful server on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        state := manager.AddClient(conn)
        go handleStateful(state, manager)
    }
}

func handleStateful(state *ClientState, manager *StateManager) {
    defer state.Conn.Close()
    defer manager.RemoveClient(state.ID)
    
    fmt.Printf("Client %d connected from %s\n", state.ID, state.Conn.RemoteAddr())
    
    scanner := bufio.NewScanner(state.Conn)
    
    for scanner.Scan() {
        line := strings.TrimSpace(scanner.Text())
        state.LastActivity = time.Now()
        
        parts := strings.Fields(line)
        if len(parts) == 0 {
            continue
        }
        
        command := parts[0]
        
        switch command {
        case "LOGIN":
            if len(parts) < 2 {
                state.Conn.Write([]byte("ERROR: Username required\n"))
                continue
            }
            state.Username = parts[1]
            state.Authenticated = true
            state.Conn.Write([]byte(fmt.Sprintf("OK: Logged in as %s\n", 
                state.Username)))
            
        case "WHOAMI":
            if !state.Authenticated {
                state.Conn.Write([]byte("ERROR: Not authenticated\n"))
                continue
            }
            state.Conn.Write([]byte(fmt.Sprintf("You are %s\n", state.Username)))
            
        case "QUIT":
            state.Conn.Write([]byte("Goodbye\n"))
            return
            
        default:
            state.Conn.Write([]byte("ERROR: Unknown command\n"))
        }
    }
}
```

The state manager tracks client sessions with authentication status, user  
information, and activity timestamps. This enables implementing stateful  
protocols with login sessions, permissions, and idle timeout handling.  

## UDP Broadcast

Broadcasting sends messages to all hosts on a network segment, useful for  
service discovery and announcements.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    go startBroadcastReceiver()
    
    time.Sleep(100 * time.Millisecond)
    
    sendBroadcast()
}

func sendBroadcast() {
    addr, err := net.ResolveUDPAddr("udp", "255.255.255.255:9999")
    if err != nil {
        log.Fatalf("Resolve failed: %v", err)
    }
    
    conn, err := net.DialUDP("udp", nil, addr)
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    message := []byte("Broadcast message from server")
    
    for i := 0; i < 3; i++ {
        _, err = conn.Write(message)
        if err != nil {
            log.Printf("Write error: %v", err)
            return
        }
        
        fmt.Printf("Broadcast %d: %s\n", i+1, message)
        time.Sleep(1 * time.Second)
    }
}

func startBroadcastReceiver() {
    addr, err := net.ResolveUDPAddr("udp", ":9999")
    if err != nil {
        log.Fatalf("Resolve failed: %v", err)
    }
    
    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("Broadcast receiver listening on :9999")
    
    buffer := make([]byte, 1024)
    
    for {
        n, remoteAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            log.Printf("Read error: %v", err)
            continue
        }
        
        fmt.Printf("Received broadcast from %s: %s\n", 
            remoteAddr, string(buffer[:n]))
    }
}
```

Broadcast uses the special address 255.255.255.255 to send packets to all  
hosts on the local network. The receiver listens on a specific port and  
receives broadcasts from any sender. This is commonly used for service  
discovery protocols.  

## UDP Multicast

Multicasting sends data to a group of interested receivers, more efficient  
than broadcasting for group communication.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

const (
    MulticastAddr = "224.0.0.1:9999"
    MaxDatagramSize = 8192
)

func main() {
    go startMulticastReceiver()
    
    time.Sleep(100 * time.Millisecond)
    
    sendMulticast()
}

func sendMulticast() {
    addr, err := net.ResolveUDPAddr("udp", MulticastAddr)
    if err != nil {
        log.Fatalf("Resolve failed: %v", err)
    }
    
    conn, err := net.DialUDP("udp", nil, addr)
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("Sending multicast messages")
    
    for i := 0; i < 5; i++ {
        message := fmt.Sprintf("Multicast message %d", i+1)
        _, err = conn.Write([]byte(message))
        if err != nil {
            log.Printf("Write error: %v", err)
            return
        }
        
        fmt.Printf("Sent: %s\n", message)
        time.Sleep(1 * time.Second)
    }
}

func startMulticastReceiver() {
    addr, err := net.ResolveUDPAddr("udp", MulticastAddr)
    if err != nil {
        log.Fatalf("Resolve failed: %v", err)
    }
    
    conn, err := net.ListenMulticastUDP("udp", nil, addr)
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("Multicast receiver listening on", MulticastAddr)
    
    conn.SetReadBuffer(MaxDatagramSize)
    
    buffer := make([]byte, MaxDatagramSize)
    
    for {
        n, remoteAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            log.Printf("Read error: %v", err)
            continue
        }
        
        fmt.Printf("Received from %s: %s\n", remoteAddr, string(buffer[:n]))
    }
}
```

Multicast uses special IP addresses (224.0.0.0 to 239.255.255.255) to deliver  
packets to multiple subscribers. Receivers join multicast groups to receive  
messages. This is efficient for streaming data to multiple clients.  

## Non-blocking I/O with Select

Non-blocking operations prevent goroutines from blocking indefinitely on  
network I/O.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "time"
)

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Non-blocking server on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go handleNonBlocking(conn)
    }
}

func handleNonBlocking(conn net.Conn) {
    defer conn.Close()
    
    fmt.Printf("Client connected: %s\n", conn.RemoteAddr())
    
    dataChan := make(chan []byte)
    errChan := make(chan error)
    
    go func() {
        buffer := make([]byte, 1024)
        for {
            n, err := conn.Read(buffer)
            if err != nil {
                errChan <- err
                return
            }
            
            data := make([]byte, n)
            copy(data, buffer[:n])
            dataChan <- data
        }
    }()
    
    timeout := time.After(10 * time.Second)
    
    for {
        select {
        case data := <-dataChan:
            fmt.Printf("Received: %s", string(data))
            
            _, err := conn.Write(data)
            if err != nil {
                log.Printf("Write error: %v", err)
                return
            }
            
        case err := <-errChan:
            if err != io.EOF {
                log.Printf("Read error: %v", err)
            }
            return
            
        case <-timeout:
            fmt.Println("Connection timeout")
            return
        }
    }
}
```

Using goroutines with channels enables non-blocking network I/O. The select  
statement multiplexes between data reception, error handling, and timeout  
conditions, preventing indefinite blocking.  

## Request-Response Pattern

The request-response pattern implements synchronous communication where  
clients wait for server responses.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strings"
    "time"
)

type Request struct {
    ID      string
    Command string
    Args    []string
}

type Response struct {
    ID     string
    Status string
    Data   string
}

func main() {
    go startRequestServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    requests := []Request{
        {ID: "1", Command: "ECHO", Args: []string{"hello"}},
        {ID: "2", Command: "UPPER", Args: []string{"make", "this", "uppercase"}},
        {ID: "3", Command: "REVERSE", Args: []string{"reverse"}},
    }
    
    reader := bufio.NewReader(conn)
    
    for _, req := range requests {
        line := fmt.Sprintf("%s:%s:%s\n", 
            req.ID, req.Command, strings.Join(req.Args, " "))
        
        _, err := conn.Write([]byte(line))
        if err != nil {
            log.Printf("Write error: %v", err)
            continue
        }
        
        response, err := reader.ReadString('\n')
        if err != nil {
            log.Printf("Read error: %v", err)
            continue
        }
        
        fmt.Printf("Request %s: %s", req.ID, response)
    }
}

func startRequestServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go handleRequests(conn)
    }
}

func handleRequests(conn net.Conn) {
    defer conn.Close()
    
    scanner := bufio.NewScanner(conn)
    
    for scanner.Scan() {
        line := scanner.Text()
        parts := strings.SplitN(line, ":", 3)
        
        if len(parts) != 3 {
            continue
        }
        
        id, command, args := parts[0], parts[1], parts[2]
        
        var result string
        
        switch command {
        case "ECHO":
            result = args
        case "UPPER":
            result = strings.ToUpper(args)
        case "REVERSE":
            runes := []rune(args)
            for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
                runes[i], runes[j] = runes[j], runes[i]
            }
            result = string(runes)
        default:
            result = "Unknown command"
        }
        
        response := fmt.Sprintf("%s:OK:%s\n", id, result)
        conn.Write([]byte(response))
    }
}
```

This pattern assigns unique IDs to requests, allowing clients to match  
responses. The server processes each request and returns a corresponding  
response, enabling synchronous communication over asynchronous transport.  

## Pipeline Processing

Pipeline processing chains multiple processing stages for efficient data  
transformation in network applications.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strings"
    "time"
)

func main() {
    go startPipelineServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    messages := []string{
        "hello world",
        "go programming",
        "network pipeline",
    }
    
    writer := bufio.NewWriter(conn)
    reader := bufio.NewReader(conn)
    
    for _, msg := range messages {
        _, err := writer.WriteString(msg + "\n")
        if err != nil {
            log.Printf("Write error: %v", err)
            continue
        }
        writer.Flush()
        
        response, err := reader.ReadString('\n')
        if err != nil {
            log.Printf("Read error: %v", err)
            continue
        }
        
        fmt.Printf("Input: %s => Output: %s", msg, response)
    }
}

func startPipelineServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go processPipeline(conn)
    }
}

func processPipeline(conn net.Conn) {
    defer conn.Close()
    
    scanner := bufio.NewScanner(conn)
    
    for scanner.Scan() {
        input := scanner.Text()
        
        stage1 := make(chan string)
        stage2 := make(chan string)
        stage3 := make(chan string)
        
        go func() {
            trimmed := strings.TrimSpace(input)
            stage1 <- trimmed
            close(stage1)
        }()
        
        go func() {
            text := <-stage1
            upper := strings.ToUpper(text)
            stage2 <- upper
            close(stage2)
        }()
        
        go func() {
            text := <-stage2
            reversed := reverseWords(text)
            stage3 <- reversed
            close(stage3)
        }()
        
        result := <-stage3
        
        conn.Write([]byte(result + "\n"))
    }
}

func reverseWords(s string) string {
    words := strings.Fields(s)
    for i, j := 0, len(words)-1; i < j; i, j = i+1, j-1 {
        words[i], words[j] = words[j], words[i]
    }
    return strings.Join(words, " ")
}
```

Pipeline processing splits data transformation into stages connected by  
channels. Each stage runs concurrently, enabling parallel processing and  
efficient resource utilization for complex data transformations.  

## Half-Duplex Communication

Half-duplex communication allows data transmission in both directions but  
not simultaneously.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    go startHalfDuplexServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    reader := bufio.NewReader(conn)
    
    for i := 1; i <= 3; i++ {
        message := fmt.Sprintf("Message %d\n", i)
        
        fmt.Printf("Sending: %s", message)
        _, err := conn.Write([]byte(message))
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        response, err := reader.ReadString('\n')
        if err != nil {
            log.Printf("Read error: %v", err)
            break
        }
        
        fmt.Printf("Received: %s", response)
        
        time.Sleep(500 * time.Millisecond)
    }
}

func startHalfDuplexServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go handleHalfDuplex(conn)
    }
}

func handleHalfDuplex(conn net.Conn) {
    defer conn.Close()
    
    scanner := bufio.NewScanner(conn)
    
    for scanner.Scan() {
        received := scanner.Text()
        fmt.Printf("Server received: %s\n", received)
        
        response := fmt.Sprintf("ACK: %s\n", received)
        
        time.Sleep(200 * time.Millisecond)
        
        _, err := conn.Write([]byte(response))
        if err != nil {
            log.Printf("Write error: %v", err)
            return
        }
    }
}
```

Half-duplex requires coordinated turn-taking between sender and receiver.  
The client sends a message and waits for response before sending the next  
message, ensuring orderly communication.  

## Full-Duplex Communication

Full-duplex enables simultaneous bidirectional communication for efficient  
interactive applications.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "time"
)

func main() {
    go startFullDuplexServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    go clientReceive(conn)
    clientSend(conn)
}

func clientSend(conn net.Conn) {
    for i := 1; i <= 5; i++ {
        message := fmt.Sprintf("Client message %d\n", i)
        _, err := conn.Write([]byte(message))
        if err != nil {
            log.Printf("Send error: %v", err)
            return
        }
        fmt.Printf("Sent: %s", message)
        time.Sleep(1 * time.Second)
    }
}

func clientReceive(conn net.Conn) {
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        fmt.Printf("Received: %s\n", scanner.Text())
    }
}

func startFullDuplexServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go handleFullDuplex(conn)
    }
}

func handleFullDuplex(conn net.Conn) {
    defer conn.Close()
    
    go serverReceive(conn)
    serverSend(conn)
}

func serverSend(conn net.Conn) {
    for i := 1; i <= 5; i++ {
        message := fmt.Sprintf("Server message %d\n", i)
        _, err := conn.Write([]byte(message))
        if err != nil {
            return
        }
        time.Sleep(1500 * time.Millisecond)
    }
}

func serverReceive(conn net.Conn) {
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        fmt.Printf("Server received: %s\n", scanner.Text())
    }
}
```

Full-duplex uses separate goroutines for sending and receiving, allowing  
simultaneous communication in both directions. This pattern is essential for  
interactive protocols like chat or real-time collaboration.  

## Custom Protocol with Header

Implementing custom protocols with headers enables structured communication  
with metadata and payload separation.  

```go
package main

import (
    "bytes"
    "encoding/binary"
    "fmt"
    "io"
    "log"
    "net"
)

const (
    ProtocolVersion = 1
    MagicNumber     = 0xDEADBEEF
)

type PacketHeader struct {
    Magic   uint32
    Version uint8
    Type    uint8
    Length  uint16
}

type Packet struct {
    Header  PacketHeader
    Payload []byte
}

func (p *Packet) Encode() ([]byte, error) {
    buf := new(bytes.Buffer)
    
    if err := binary.Write(buf, binary.BigEndian, p.Header); err != nil {
        return nil, err
    }
    
    if _, err := buf.Write(p.Payload); err != nil {
        return nil, err
    }
    
    return buf.Bytes(), nil
}

func DecodePacket(conn net.Conn) (*Packet, error) {
    var header PacketHeader
    
    if err := binary.Read(conn, binary.BigEndian, &header); err != nil {
        return nil, err
    }
    
    if header.Magic != MagicNumber {
        return nil, fmt.Errorf("invalid magic number: 0x%X", header.Magic)
    }
    
    if header.Version != ProtocolVersion {
        return nil, fmt.Errorf("unsupported version: %d", header.Version)
    }
    
    payload := make([]byte, header.Length)
    if _, err := io.ReadFull(conn, payload); err != nil {
        return nil, err
    }
    
    return &Packet{Header: header, Payload: payload}, nil
}

func main() {
    go startProtocolServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    payload := []byte("Custom protocol message")
    packet := &Packet{
        Header: PacketHeader{
            Magic:   MagicNumber,
            Version: ProtocolVersion,
            Type:    1,
            Length:  uint16(len(payload)),
        },
        Payload: payload,
    }
    
    data, err := packet.Encode()
    if err != nil {
        log.Fatalf("Encode failed: %v", err)
    }
    
    _, err = conn.Write(data)
    if err != nil {
        log.Fatalf("Write failed: %v", err)
    }
    
    response, err := DecodePacket(conn)
    if err != nil {
        log.Fatalf("Decode failed: %v", err)
    }
    
    fmt.Printf("Response: %s\n", string(response.Payload))
}

func startProtocolServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go func(c net.Conn) {
            defer c.Close()
            
            packet, err := DecodePacket(c)
            if err != nil {
                return
            }
            
            response := &Packet{
                Header: PacketHeader{
                    Magic:   MagicNumber,
                    Version: ProtocolVersion,
                    Type:    packet.Header.Type + 1,
                    Length:  uint16(len(packet.Payload)),
                },
                Payload: packet.Payload,
            }
            
            data, _ := response.Encode()
            c.Write(data)
        }(conn)
    }
}
```

Custom protocols with headers include magic numbers for validation, version  
for compatibility, type for message classification, and length for payload  
size. This structure enables robust protocol implementation.  

## Connection Multiplexing

Multiplexing allows multiple logical channels over a single physical  
connection, improving resource efficiency.  

```go
package main

import (
    "encoding/binary"
    "fmt"
    "io"
    "log"
    "net"
    "sync"
)

type MultiplexedConn struct {
    conn     net.Conn
    channels map[uint16]chan []byte
    mu       sync.RWMutex
}

func NewMultiplexedConn(conn net.Conn) *MultiplexedConn {
    mc := &MultiplexedConn{
        conn:     conn,
        channels: make(map[uint16]chan []byte),
    }
    
    go mc.readLoop()
    
    return mc
}

func (mc *MultiplexedConn) OpenChannel(id uint16) chan []byte {
    mc.mu.Lock()
    defer mc.mu.Unlock()
    
    ch := make(chan []byte, 10)
    mc.channels[id] = ch
    return ch
}

func (mc *MultiplexedConn) Send(channelID uint16, data []byte) error {
    header := make([]byte, 4)
    binary.BigEndian.PutUint16(header[0:2], channelID)
    binary.BigEndian.PutUint16(header[2:4], uint16(len(data)))
    
    mc.mu.Lock()
    defer mc.mu.Unlock()
    
    if _, err := mc.conn.Write(header); err != nil {
        return err
    }
    
    _, err := mc.conn.Write(data)
    return err
}

func (mc *MultiplexedConn) readLoop() {
    for {
        header := make([]byte, 4)
        if _, err := io.ReadFull(mc.conn, header); err != nil {
            return
        }
        
        channelID := binary.BigEndian.Uint16(header[0:2])
        length := binary.BigEndian.Uint16(header[2:4])
        
        data := make([]byte, length)
        if _, err := io.ReadFull(mc.conn, data); err != nil {
            return
        }
        
        mc.mu.RLock()
        ch, exists := mc.channels[channelID]
        mc.mu.RUnlock()
        
        if exists {
            ch <- data
        }
    }
}

func main() {
    go startMuxServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    mux := NewMultiplexedConn(conn)
    
    ch1 := mux.OpenChannel(1)
    ch2 := mux.OpenChannel(2)
    
    go func() {
        for data := range ch1 {
            fmt.Printf("Channel 1: %s\n", string(data))
        }
    }()
    
    go func() {
        for data := range ch2 {
            fmt.Printf("Channel 2: %s\n", string(data))
        }
    }()
    
    mux.Send(1, []byte("Message on channel 1"))
    mux.Send(2, []byte("Message on channel 2"))
    mux.Send(1, []byte("Another message on channel 1"))
}

func startMuxServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    mux := NewMultiplexedConn(conn)
    
    ch1 := mux.OpenChannel(1)
    ch2 := mux.OpenChannel(2)
    
    go func() {
        for data := range ch1 {
            mux.Send(1, append([]byte("Echo 1: "), data...))
        }
    }()
    
    go func() {
        for data := range ch2 {
            mux.Send(2, append([]byte("Echo 2: "), data...))
        }
    }()
    
    select {}
}
```

Multiplexing splits a connection into virtual channels identified by IDs.  
Each message includes a channel ID and length in its header. This enables  
concurrent streams over a single connection.  

## Rate Limiting

Rate limiting controls the speed of data transmission to prevent overwhelming  
receivers or network infrastructure.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

type RateLimiter struct {
    rate     int
    interval time.Duration
    tokens   chan struct{}
}

func NewRateLimiter(rate int, interval time.Duration) *RateLimiter {
    rl := &RateLimiter{
        rate:     rate,
        interval: interval,
        tokens:   make(chan struct{}, rate),
    }
    
    for i := 0; i < rate; i++ {
        rl.tokens <- struct{}{}
    }
    
    go rl.refill()
    
    return rl
}

func (rl *RateLimiter) refill() {
    ticker := time.NewTicker(rl.interval)
    defer ticker.Stop()
    
    for range ticker.C {
        for i := 0; i < rl.rate; i++ {
            select {
            case rl.tokens <- struct{}{}:
            default:
            }
        }
    }
}

func (rl *RateLimiter) Wait() {
    <-rl.tokens
}

func main() {
    go startRateLimitedServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    limiter := NewRateLimiter(5, time.Second)
    
    for i := 1; i <= 15; i++ {
        limiter.Wait()
        
        message := fmt.Sprintf("Message %d\n", i)
        _, err := conn.Write([]byte(message))
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        fmt.Printf("Sent: %s", message)
    }
}

func startRateLimitedServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            return
        }
        fmt.Printf("Server received: %s", string(buffer[:n]))
    }
}
```

Rate limiting uses a token bucket algorithm to control request rates. Tokens  
are consumed for each operation and refilled periodically. This prevents  
overwhelming servers or exceeding API quotas.  

## Connection Health Checks

Health checks detect and handle connection failures proactively to maintain  
service reliability.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "sync"
    "time"
)

type HealthCheckConn struct {
    conn          net.Conn
    healthy       bool
    mu            sync.RWMutex
    pingInterval  time.Duration
    pongTimeout   time.Duration
    stopChan      chan struct{}
}

func NewHealthCheckConn(conn net.Conn) *HealthCheckConn {
    hc := &HealthCheckConn{
        conn:         conn,
        healthy:      true,
        pingInterval: 5 * time.Second,
        pongTimeout:  2 * time.Second,
        stopChan:     make(chan struct{}),
    }
    
    go hc.healthCheck()
    
    return hc
}

func (hc *HealthCheckConn) healthCheck() {
    ticker := time.NewTicker(hc.pingInterval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            if !hc.sendPing() {
                hc.markUnhealthy()
                return
            }
        case <-hc.stopChan:
            return
        }
    }
}

func (hc *HealthCheckConn) sendPing() bool {
    hc.conn.SetWriteDeadline(time.Now().Add(hc.pongTimeout))
    
    _, err := hc.conn.Write([]byte("PING\n"))
    if err != nil {
        return false
    }
    
    hc.conn.SetReadDeadline(time.Now().Add(hc.pongTimeout))
    
    buffer := make([]byte, 5)
    _, err = hc.conn.Read(buffer)
    
    return err == nil && string(buffer) == "PONG\n"
}

func (hc *HealthCheckConn) IsHealthy() bool {
    hc.mu.RLock()
    defer hc.mu.RUnlock()
    return hc.healthy
}

func (hc *HealthCheckConn) markUnhealthy() {
    hc.mu.Lock()
    defer hc.mu.Unlock()
    hc.healthy = false
    fmt.Println("Connection marked as unhealthy")
}

func (hc *HealthCheckConn) Close() {
    close(hc.stopChan)
    hc.conn.Close()
}

func main() {
    go startHealthCheckServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    
    hcConn := NewHealthCheckConn(conn)
    defer hcConn.Close()
    
    for i := 0; i < 20; i++ {
        time.Sleep(1 * time.Second)
        
        if hcConn.IsHealthy() {
            fmt.Printf("Connection healthy at t=%ds\n", i+1)
        } else {
            fmt.Println("Connection unhealthy, exiting")
            break
        }
    }
}

func startHealthCheckServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 5)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            return
        }
        
        if string(buffer[:n]) == "PING\n" {
            conn.Write([]byte("PONG\n"))
        }
    }
}
```

Health checks send periodic PING messages and expect PONG responses. Failed  
checks mark connections as unhealthy, allowing applications to reconnect or  
failover. This ensures connection reliability.  

## Zero-Copy Transfer

Zero-copy techniques minimize data copying for efficient large file transfers.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "os"
)

func sendFile(conn net.Conn, filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    fileInfo, err := file.Stat()
    if err != nil {
        return err
    }
    
    fmt.Printf("Sending file: %s (%d bytes)\n", filename, fileInfo.Size())
    
    written, err := io.Copy(conn, file)
    if err != nil {
        return err
    }
    
    fmt.Printf("Sent %d bytes\n", written)
    return nil
}

func receiveFile(conn net.Conn, filename string) error {
    file, err := os.Create(filename)
    if err != nil {
        return err
    }
    defer file.Close()
    
    written, err := io.Copy(file, conn)
    if err != nil {
        return err
    }
    
    fmt.Printf("Received %d bytes to %s\n", written, filename)
    return nil
}

func main() {
    err := os.WriteFile("/tmp/test.txt", []byte("Zero-copy test data"), 0644)
    if err != nil {
        log.Fatalf("Failed to create test file: %v", err)
    }
    
    go startFileServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    err = sendFile(conn, "/tmp/test.txt")
    if err != nil {
        log.Fatalf("Send failed: %v", err)
    }
}

func startFileServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    receiveFile(conn, "/tmp/received.txt")
}
```

`io.Copy` performs efficient data transfer without intermediate buffering in  
user space. The kernel can use zero-copy techniques like sendfile() on  
supported platforms, significantly improving performance for large transfers.  

## Socket Options Configuration

Socket options control low-level networking behavior for performance tuning  
and protocol requirements.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "syscall"
)

func configureSocket(conn net.Conn) error {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fmt.Errorf("not a TCP connection")
    }
    
    rawConn, err := tcpConn.SyscallConn()
    if err != nil {
        return err
    }
    
    var sockOptErr error
    err = rawConn.Control(func(fd uintptr) {
        sockOptErr = syscall.SetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_REUSEADDR, 1)
        if sockOptErr != nil {
            return
        }
        
        sockOptErr = syscall.SetsockoptInt(int(fd), 
            syscall.IPPROTO_TCP, syscall.TCP_NODELAY, 1)
        if sockOptErr != nil {
            return
        }
        
        sockOptErr = syscall.SetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_RCVBUF, 65536)
        if sockOptErr != nil {
            return
        }
        
        sockOptErr = syscall.SetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_SNDBUF, 65536)
    })
    
    if err != nil {
        return err
    }
    
    return sockOptErr
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Socket options server on :8080")
    
    conn, err := listener.Accept()
    if err != nil {
        log.Fatalf("Accept failed: %v", err)
    }
    defer conn.Close()
    
    err = configureSocket(conn)
    if err != nil {
        log.Printf("Socket configuration failed: %v", err)
    } else {
        fmt.Println("Socket configured successfully")
    }
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Printf("Read error: %v", err)
        return
    }
    
    fmt.Printf("Received: %s\n", string(buffer[:n]))
}
```

Socket options like SO_REUSEADDR (address reuse), TCP_NODELAY (disable Nagle),  
SO_RCVBUF (receive buffer size), and SO_SNDBUF (send buffer size) tune socket  
behavior for specific requirements.  

## Scatter-Gather I/O

Scatter-gather I/O enables efficient handling of non-contiguous data buffers  
in a single system call.  

```go
package main

import (
    "fmt"
    "log"
    "net"
)

func writeVectored(conn net.Conn, buffers [][]byte) (int, error) {
    total := 0
    
    for _, buf := range buffers {
        n, err := conn.Write(buf)
        if err != nil {
            return total, err
        }
        total += n
    }
    
    return total, nil
}

func readVectored(conn net.Conn, buffers [][]byte) (int, error) {
    total := 0
    
    for _, buf := range buffers {
        n, err := conn.Read(buf)
        if err != nil {
            return total, err
        }
        total += n
        
        if n < len(buf) {
            break
        }
    }
    
    return total, nil
}

func main() {
    go startScatterGatherServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    buffers := [][]byte{
        []byte("First "),
        []byte("Second "),
        []byte("Third\n"),
    }
    
    n, err := writeVectored(conn, buffers)
    if err != nil {
        log.Fatalf("Write failed: %v", err)
    }
    
    fmt.Printf("Sent %d bytes in %d buffers\n", n, len(buffers))
    
    recvBuffers := [][]byte{
        make([]byte, 10),
        make([]byte, 10),
    }
    
    n, err = readVectored(conn, recvBuffers)
    if err != nil {
        log.Printf("Read error: %v", err)
    }
    
    fmt.Printf("Received %d bytes\n", n)
}

func startScatterGatherServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, _ := conn.Read(buffer)
    
    conn.Write([]byte("Echo: "))
    conn.Write(buffer[:n])
}
```

Scatter-gather I/O writes multiple buffers in one operation (gather) or reads  
into multiple buffers (scatter). This reduces system call overhead when  
working with fragmented data structures.  

## Bandwidth Monitoring

Monitoring network bandwidth helps identify performance bottlenecks and  
optimize resource usage.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "sync/atomic"
    "time"
)

type BandwidthMonitor struct {
    bytesRead    uint64
    bytesWritten uint64
    startTime    time.Time
}

func NewBandwidthMonitor() *BandwidthMonitor {
    return &BandwidthMonitor{
        startTime: time.Now(),
    }
}

func (bm *BandwidthMonitor) MonitoredRead(conn net.Conn, buf []byte) (int, error) {
    n, err := conn.Read(buf)
    atomic.AddUint64(&bm.bytesRead, uint64(n))
    return n, err
}

func (bm *BandwidthMonitor) MonitoredWrite(conn net.Conn, buf []byte) (int, error) {
    n, err := conn.Write(buf)
    atomic.AddUint64(&bm.bytesWritten, uint64(n))
    return n, err
}

func (bm *BandwidthMonitor) GetStats() (readBps, writeBps float64) {
    elapsed := time.Since(bm.startTime).Seconds()
    
    bytesRead := atomic.LoadUint64(&bm.bytesRead)
    bytesWritten := atomic.LoadUint64(&bm.bytesWritten)
    
    readBps = float64(bytesRead) / elapsed
    writeBps = float64(bytesWritten) / elapsed
    
    return
}

func (bm *BandwidthMonitor) PrintStats() {
    readBps, writeBps := bm.GetStats()
    
    fmt.Printf("Read: %.2f bytes/sec (%.2f KB/s)\n", readBps, readBps/1024)
    fmt.Printf("Write: %.2f bytes/sec (%.2f KB/s)\n", writeBps, writeBps/1024)
}

func main() {
    go startMonitoredServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    monitor := NewBandwidthMonitor()
    
    go func() {
        ticker := time.NewTicker(2 * time.Second)
        defer ticker.Stop()
        
        for range ticker.C {
            monitor.PrintStats()
        }
    }()
    
    data := make([]byte, 1024)
    for i := 0; i < 100; i++ {
        monitor.MonitoredWrite(conn, data)
        time.Sleep(50 * time.Millisecond)
    }
    
    time.Sleep(1 * time.Second)
    monitor.PrintStats()
}

func startMonitoredServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    io.Copy(io.Discard, conn)
}
```

Bandwidth monitoring tracks bytes transferred over time to calculate throughput.  
Atomic operations ensure thread-safe counter updates. This helps identify  
network performance characteristics.  

## Connection Backpressure

Backpressure prevents fast senders from overwhelming slow receivers by  
controlling data flow.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "time"
)

type BackpressureWriter struct {
    conn          net.Conn
    maxBufferSize int
    buffer        []byte
    flushInterval time.Duration
}

func NewBackpressureWriter(conn net.Conn, maxSize int, interval time.Duration) *BackpressureWriter {
    bw := &BackpressureWriter{
        conn:          conn,
        maxBufferSize: maxSize,
        buffer:        make([]byte, 0, maxSize),
        flushInterval: interval,
    }
    
    go bw.autoFlush()
    
    return bw
}

func (bw *BackpressureWriter) Write(data []byte) error {
    for len(bw.buffer)+len(data) > bw.maxBufferSize {
        if err := bw.Flush(); err != nil {
            return err
        }
        time.Sleep(10 * time.Millisecond)
    }
    
    bw.buffer = append(bw.buffer, data...)
    
    return nil
}

func (bw *BackpressureWriter) Flush() error {
    if len(bw.buffer) == 0 {
        return nil
    }
    
    _, err := bw.conn.Write(bw.buffer)
    if err != nil {
        return err
    }
    
    bw.buffer = bw.buffer[:0]
    return nil
}

func (bw *BackpressureWriter) autoFlush() {
    ticker := time.NewTicker(bw.flushInterval)
    defer ticker.Stop()
    
    for range ticker.C {
        bw.Flush()
    }
}

func main() {
    go startBackpressureServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    writer := NewBackpressureWriter(conn, 4096, 500*time.Millisecond)
    
    data := make([]byte, 512)
    for i := 0; i < 20; i++ {
        err := writer.Write(data)
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        fmt.Printf("Queued write %d\n", i+1)
        time.Sleep(100 * time.Millisecond)
    }
    
    writer.Flush()
}

func startBackpressureServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    io.Copy(io.Discard, conn)
}
```

Backpressure buffers writes and flushes when the buffer is full or after a  
timeout. This prevents overwhelming slow receivers while maintaining efficient  
batching of small writes.  

## Custom Framing Protocol

Framing separates message boundaries in continuous byte streams, essential  
for message-oriented protocols.  

```go
package main

import (
    "bytes"
    "encoding/binary"
    "fmt"
    "io"
    "log"
    "net"
)

type FrameReader struct {
    conn net.Conn
}

type FrameWriter struct {
    conn net.Conn
}

func NewFrameReader(conn net.Conn) *FrameReader {
    return &FrameReader{conn: conn}
}

func NewFrameWriter(conn net.Conn) *FrameWriter {
    return &FrameWriter{conn: conn}
}

func (fw *FrameWriter) WriteFrame(data []byte) error {
    length := uint32(len(data))
    
    if err := binary.Write(fw.conn, binary.BigEndian, length); err != nil {
        return err
    }
    
    _, err := fw.conn.Write(data)
    return err
}

func (fr *FrameReader) ReadFrame() ([]byte, error) {
    var length uint32
    
    if err := binary.Read(fr.conn, binary.BigEndian, &length); err != nil {
        return nil, err
    }
    
    if length > 1024*1024 {
        return nil, fmt.Errorf("frame too large: %d bytes", length)
    }
    
    data := make([]byte, length)
    _, err := io.ReadFull(fr.conn, data)
    
    return data, err
}

func main() {
    go startFramingServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    writer := NewFrameWriter(conn)
    reader := NewFrameReader(conn)
    
    messages := []string{
        "First frame",
        "Second frame with more data",
        "Third frame",
    }
    
    for _, msg := range messages {
        err := writer.WriteFrame([]byte(msg))
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        fmt.Printf("Sent: %s\n", msg)
        
        response, err := reader.ReadFrame()
        if err != nil {
            log.Printf("Read error: %v", err)
            break
        }
        
        fmt.Printf("Received: %s\n", string(response))
    }
}

func startFramingServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    reader := NewFrameReader(conn)
    writer := NewFrameWriter(conn)
    
    for {
        frame, err := reader.ReadFrame()
        if err != nil {
            return
        }
        
        response := bytes.ToUpper(frame)
        writer.WriteFrame(response)
    }
}
```

Length-prefixed framing writes a 4-byte length header before each message.  
The reader first reads the length, then reads exactly that many bytes. This  
enables reliable message boundary detection in byte streams.  

