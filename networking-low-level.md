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

## Delimited Message Protocol

Delimiter-based protocols use special characters to mark message boundaries,  
simpler than length-prefixed framing for text protocols.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strings"
)

const Delimiter = '\n'

func main() {
    go startDelimitedServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    writer := bufio.NewWriter(conn)
    reader := bufio.NewReader(conn)
    
    messages := []string{
        "Hello server",
        "How are you",
        "Goodbye",
    }
    
    for _, msg := range messages {
        _, err := writer.WriteString(msg + string(Delimiter))
        if err != nil {
            log.Printf("Write error: %v", err)
            break
        }
        
        err = writer.Flush()
        if err != nil {
            log.Printf("Flush error: %v", err)
            break
        }
        
        fmt.Printf("Sent: %s\n", msg)
        
        response, err := reader.ReadString(Delimiter)
        if err != nil {
            log.Printf("Read error: %v", err)
            break
        }
        
        fmt.Printf("Received: %s", response)
    }
}

func startDelimitedServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    reader := bufio.NewReader(conn)
    writer := bufio.NewWriter(conn)
    
    for {
        line, err := reader.ReadString(Delimiter)
        if err != nil {
            return
        }
        
        response := strings.ToUpper(line)
        writer.WriteString(response)
        writer.Flush()
    }
}
```

Delimiter-based protocols use newlines, nulls, or custom characters to separate  
messages. This is common in text protocols like HTTP and SMTP. Care must be  
taken to escape delimiters in message content.  

## Stream Compression

Compression reduces bandwidth usage for large data transfers over network  
connections.  

```go
package main

import (
    "compress/gzip"
    "fmt"
    "io"
    "log"
    "net"
)

func sendCompressed(conn net.Conn, data []byte) error {
    gzipWriter := gzip.NewWriter(conn)
    defer gzipWriter.Close()
    
    _, err := gzipWriter.Write(data)
    if err != nil {
        return err
    }
    
    return gzipWriter.Close()
}

func receiveCompressed(conn net.Conn) ([]byte, error) {
    gzipReader, err := gzip.NewReader(conn)
    if err != nil {
        return nil, err
    }
    defer gzipReader.Close()
    
    return io.ReadAll(gzipReader)
}

func main() {
    go startCompressionServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    data := make([]byte, 10000)
    for i := range data {
        data[i] = byte(i % 256)
    }
    
    fmt.Printf("Sending %d bytes (uncompressed)\n", len(data))
    
    err = sendCompressed(conn, data)
    if err != nil {
        log.Fatalf("Send failed: %v", err)
    }
    
    fmt.Println("Data sent with compression")
}

func startCompressionServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    data, err := receiveCompressed(conn)
    if err != nil {
        log.Printf("Receive error: %v", err)
        return
    }
    
    fmt.Printf("Server received %d bytes (decompressed)\n", len(data))
}
```

Compression wraps the connection with gzip writers/readers, transparently  
compressing data during transmission. This significantly reduces bandwidth  
for compressible data but adds CPU overhead.  

## Connection Retry Logic

Automatic retry with exponential backoff improves reliability in unreliable  
network environments.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

type RetryConfig struct {
    MaxRetries     int
    InitialBackoff time.Duration
    MaxBackoff     time.Duration
    BackoffFactor  float64
}

func DialWithRetry(network, address string, config RetryConfig) (net.Conn, error) {
    var lastErr error
    backoff := config.InitialBackoff
    
    for attempt := 0; attempt <= config.MaxRetries; attempt++ {
        if attempt > 0 {
            fmt.Printf("Retry attempt %d after %v\n", attempt, backoff)
            time.Sleep(backoff)
            
            backoff = time.Duration(float64(backoff) * config.BackoffFactor)
            if backoff > config.MaxBackoff {
                backoff = config.MaxBackoff
            }
        }
        
        conn, err := net.DialTimeout(network, address, 5*time.Second)
        if err == nil {
            fmt.Printf("Connected successfully on attempt %d\n", attempt+1)
            return conn, nil
        }
        
        lastErr = err
        fmt.Printf("Attempt %d failed: %v\n", attempt+1, err)
    }
    
    return nil, fmt.Errorf("failed after %d attempts: %w", 
        config.MaxRetries+1, lastErr)
}

func main() {
    config := RetryConfig{
        MaxRetries:     5,
        InitialBackoff: 100 * time.Millisecond,
        MaxBackoff:     5 * time.Second,
        BackoffFactor:  2.0,
    }
    
    go func() {
        time.Sleep(2 * time.Second)
        listener, _ := net.Listen("tcp", ":8080")
        defer listener.Close()
        
        conn, _ := listener.Accept()
        conn.Close()
    }()
    
    conn, err := DialWithRetry("tcp", "localhost:8080", config)
    if err != nil {
        log.Fatalf("Connection failed: %v", err)
    }
    defer conn.Close()
    
    fmt.Println("Connection established")
}
```

Exponential backoff gradually increases wait time between retries, preventing  
network flooding while allowing recovery from transient failures. The maximum  
backoff prevents excessive delays.  

## Echo Server with Line Protocol

Echo servers are fundamental testing tools that return received data,  
demonstrating basic request-response patterns.  

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

type EchoServer struct {
    listener net.Listener
    clients  int
}

func NewEchoServer(addr string) (*EchoServer, error) {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        return nil, err
    }
    
    return &EchoServer{listener: listener}, nil
}

func (es *EchoServer) Start() {
    fmt.Printf("Echo server listening on %s\n", es.listener.Addr())
    
    for {
        conn, err := es.listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        es.clients++
        clientID := es.clients
        
        go es.handleClient(conn, clientID)
    }
}

func (es *EchoServer) handleClient(conn net.Conn, id int) {
    defer conn.Close()
    
    fmt.Printf("Client %d connected: %s\n", id, conn.RemoteAddr())
    
    scanner := bufio.NewScanner(conn)
    
    for scanner.Scan() {
        line := scanner.Text()
        
        if strings.TrimSpace(line) == "QUIT" {
            conn.Write([]byte("Goodbye\n"))
            break
        }
        
        timestamp := time.Now().Format("15:04:05")
        response := fmt.Sprintf("[%s] Echo: %s\n", timestamp, line)
        
        _, err := conn.Write([]byte(response))
        if err != nil {
            log.Printf("Client %d write error: %v", id, err)
            break
        }
    }
    
    if err := scanner.Err(); err != nil {
        log.Printf("Client %d scanner error: %v", id, err)
    }
    
    fmt.Printf("Client %d disconnected\n", id)
}

func main() {
    server, err := NewEchoServer(":8080")
    if err != nil {
        log.Fatalf("Failed to create server: %v", err)
    }
    
    server.Start()
}
```

Echo servers demonstrate fundamental patterns: accepting connections, reading  
lines, processing commands, and sending responses. The QUIT command provides  
graceful disconnection.  

## Chat Server Implementation

Chat servers broadcast messages to multiple connected clients, demonstrating  
multi-client coordination.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "sync"
)

type ChatServer struct {
    listener net.Listener
    clients  map[net.Conn]string
    mu       sync.RWMutex
    messages chan string
}

func NewChatServer(addr string) (*ChatServer, error) {
    listener, err := net.Listen("tcp", addr)
    if err != nil {
        return nil, err
    }
    
    return &ChatServer{
        listener: listener,
        clients:  make(map[net.Conn]string),
        messages: make(chan string, 100),
    }, nil
}

func (cs *ChatServer) Start() {
    fmt.Printf("Chat server listening on %s\n", cs.listener.Addr())
    
    go cs.broadcast()
    
    for {
        conn, err := cs.listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go cs.handleClient(conn)
    }
}

func (cs *ChatServer) broadcast() {
    for msg := range cs.messages {
        cs.mu.RLock()
        for client := range cs.clients {
            client.Write([]byte(msg))
        }
        cs.mu.RUnlock()
    }
}

func (cs *ChatServer) handleClient(conn net.Conn) {
    defer conn.Close()
    
    conn.Write([]byte("Enter your name: "))
    
    scanner := bufio.NewScanner(conn)
    if !scanner.Scan() {
        return
    }
    
    name := scanner.Text()
    
    cs.mu.Lock()
    cs.clients[conn] = name
    cs.mu.Unlock()
    
    cs.messages <- fmt.Sprintf("%s joined the chat\n", name)
    
    for scanner.Scan() {
        msg := scanner.Text()
        cs.messages <- fmt.Sprintf("%s: %s\n", name, msg)
    }
    
    cs.mu.Lock()
    delete(cs.clients, conn)
    cs.mu.Unlock()
    
    cs.messages <- fmt.Sprintf("%s left the chat\n", name)
}

func main() {
    server, err := NewChatServer(":8080")
    if err != nil {
        log.Fatalf("Failed to create server: %v", err)
    }
    
    server.Start()
}
```

The chat server maintains a client registry and broadcasts messages through  
a channel. A dedicated goroutine distributes messages to all clients,  
demonstrating fan-out messaging patterns.  

## Connection Pooling with Health Checks

Advanced connection pools monitor connection health and automatically replace  
failed connections.  

```go
package main

import (
    "errors"
    "fmt"
    "net"
    "sync"
    "time"
)

type HealthyConn struct {
    conn      net.Conn
    lastCheck time.Time
    healthy   bool
}

type HealthCheckPool struct {
    address      string
    maxSize      int
    checkInterval time.Duration
    pool         chan *HealthyConn
    mu           sync.Mutex
    closed       bool
}

func NewHealthCheckPool(address string, size int) *HealthCheckPool {
    p := &HealthCheckPool{
        address:      address,
        maxSize:      size,
        checkInterval: 30 * time.Second,
        pool:         make(chan *HealthyConn, size),
    }
    
    go p.healthChecker()
    
    return p
}

func (p *HealthCheckPool) Get() (net.Conn, error) {
    p.mu.Lock()
    if p.closed {
        p.mu.Unlock()
        return nil, errors.New("pool closed")
    }
    p.mu.Unlock()
    
    select {
    case hc := <-p.pool:
        if hc.healthy {
            return hc.conn, nil
        }
        hc.conn.Close()
        return p.createConnection()
    default:
        return p.createConnection()
    }
}

func (p *HealthCheckPool) Put(conn net.Conn) {
    if conn == nil {
        return
    }
    
    p.mu.Lock()
    if p.closed {
        p.mu.Unlock()
        conn.Close()
        return
    }
    p.mu.Unlock()
    
    hc := &HealthyConn{
        conn:      conn,
        lastCheck: time.Now(),
        healthy:   true,
    }
    
    select {
    case p.pool <- hc:
    default:
        conn.Close()
    }
}

func (p *HealthCheckPool) createConnection() (net.Conn, error) {
    return net.DialTimeout("tcp", p.address, 5*time.Second)
}

func (p *HealthCheckPool) healthChecker() {
    ticker := time.NewTicker(p.checkInterval)
    defer ticker.Stop()
    
    for range ticker.C {
        p.mu.Lock()
        if p.closed {
            p.mu.Unlock()
            return
        }
        p.mu.Unlock()
        
        size := len(p.pool)
        for i := 0; i < size; i++ {
            select {
            case hc := <-p.pool:
                if p.checkConnection(hc.conn) {
                    hc.healthy = true
                    hc.lastCheck = time.Now()
                    p.pool <- hc
                } else {
                    hc.conn.Close()
                }
            default:
                return
            }
        }
    }
}

func (p *HealthCheckPool) checkConnection(conn net.Conn) bool {
    conn.SetWriteDeadline(time.Now().Add(1 * time.Second))
    _, err := conn.Write([]byte("PING\n"))
    return err == nil
}

func (p *HealthCheckPool) Close() {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    p.closed = true
    close(p.pool)
    
    for hc := range p.pool {
        hc.conn.Close()
    }
}

func main() {
    pool := NewHealthCheckPool("localhost:8080", 5)
    defer pool.Close()
    
    conn, err := pool.Get()
    if err != nil {
        fmt.Printf("Get failed: %v\n", err)
        return
    }
    
    pool.Put(conn)
    fmt.Println("Connection pool with health checks initialized")
}
```

The health check pool periodically verifies connection health using PING  
messages. Unhealthy connections are removed and replaced automatically,  
ensuring the pool contains only viable connections.  

## Streaming Data Transfer

Streaming efficiently transfers large datasets without loading everything  
into memory.  

```go
package main

import (
    "crypto/rand"
    "fmt"
    "io"
    "log"
    "net"
)

func streamLargeData(conn net.Conn, size int64) error {
    chunkSize := 8192
    buffer := make([]byte, chunkSize)
    
    var sent int64
    
    for sent < size {
        toSend := int64(chunkSize)
        if size-sent < toSend {
            toSend = size - sent
        }
        
        _, err := rand.Read(buffer[:toSend])
        if err != nil {
            return err
        }
        
        n, err := conn.Write(buffer[:toSend])
        if err != nil {
            return err
        }
        
        sent += int64(n)
        
        if sent%(1024*1024) == 0 {
            fmt.Printf("Sent: %d MB\n", sent/1024/1024)
        }
    }
    
    fmt.Printf("Transfer complete: %d bytes\n", sent)
    return nil
}

func receiveStream(conn net.Conn) error {
    buffer := make([]byte, 8192)
    var received int64
    
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            if err == io.EOF {
                break
            }
            return err
        }
        
        received += int64(n)
        
        if received%(1024*1024) == 0 {
            fmt.Printf("Received: %d MB\n", received/1024/1024)
        }
    }
    
    fmt.Printf("Receive complete: %d bytes\n", received)
    return nil
}

func main() {
    go startStreamServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    dataSize := int64(10 * 1024 * 1024)
    
    err = streamLargeData(conn, dataSize)
    if err != nil {
        log.Fatalf("Stream failed: %v", err)
    }
}

func startStreamServer() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    receiveStream(conn)
}
```

Streaming processes data in chunks, keeping memory usage constant regardless  
of transfer size. This is essential for handling large files or continuous  
data feeds efficiently.  

## Connection Load Balancing

Load balancing distributes connections across multiple backend servers for  
scalability and reliability.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "sync/atomic"
)

type LoadBalancer struct {
    backends []string
    current  uint32
}

func NewLoadBalancer(backends []string) *LoadBalancer {
    return &LoadBalancer{
        backends: backends,
    }
}

func (lb *LoadBalancer) NextBackend() string {
    n := atomic.AddUint32(&lb.current, 1)
    return lb.backends[int(n-1)%len(lb.backends)]
}

func (lb *LoadBalancer) Handle(clientConn net.Conn) {
    defer clientConn.Close()
    
    backend := lb.NextBackend()
    
    backendConn, err := net.Dial("tcp", backend)
    if err != nil {
        log.Printf("Backend connection failed: %v", err)
        return
    }
    defer backendConn.Close()
    
    fmt.Printf("Forwarding to %s\n", backend)
    
    done := make(chan struct{}, 2)
    
    go func() {
        io.Copy(backendConn, clientConn)
        done <- struct{}{}
    }()
    
    go func() {
        io.Copy(clientConn, backendConn)
        done <- struct{}{}
    }()
    
    <-done
}

func main() {
    go startBackend(":9001")
    go startBackend(":9002")
    go startBackend(":9003")
    
    backends := []string{
        "localhost:9001",
        "localhost:9002",
        "localhost:9003",
    }
    
    lb := NewLoadBalancer(backends)
    
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Load balancer listening on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go lb.Handle(conn)
    }
}

func startBackend(addr string) {
    listener, err := net.Listen("tcp", addr)
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
            
            buffer := make([]byte, 1024)
            n, _ := c.Read(buffer)
            
            response := fmt.Sprintf("Backend %s: %s", addr, string(buffer[:n]))
            c.Write([]byte(response))
        }(conn)
    }
}
```

Round-robin load balancing distributes connections evenly across backends.  
The balancer proxies data bidirectionally between clients and backends using  
`io.Copy` in separate goroutines.  

## Transparent Proxy

A transparent proxy forwards traffic while inspecting or modifying data  
in transit.  

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "log"
    "net"
    "strings"
)

type Proxy struct {
    listenAddr string
    targetAddr string
}

func NewProxy(listen, target string) *Proxy {
    return &Proxy{
        listenAddr: listen,
        targetAddr: target,
    }
}

func (p *Proxy) Start() error {
    listener, err := net.Listen("tcp", p.listenAddr)
    if err != nil {
        return err
    }
    defer listener.Close()
    
    fmt.Printf("Proxy listening on %s, forwarding to %s\n", 
        p.listenAddr, p.targetAddr)
    
    for {
        clientConn, err := listener.Accept()
        if err != nil {
            log.Printf("Accept error: %v", err)
            continue
        }
        
        go p.handleConnection(clientConn)
    }
}

func (p *Proxy) handleConnection(clientConn net.Conn) {
    defer clientConn.Close()
    
    targetConn, err := net.Dial("tcp", p.targetAddr)
    if err != nil {
        log.Printf("Target connection failed: %v", err)
        return
    }
    defer targetConn.Close()
    
    fmt.Printf("Client %s <-> Target %s\n", 
        clientConn.RemoteAddr(), p.targetAddr)
    
    done := make(chan struct{}, 2)
    
    go func() {
        p.forwardWithLogging(targetConn, clientConn, "Client->Target")
        done <- struct{}{}
    }()
    
    go func() {
        p.forwardWithLogging(clientConn, targetConn, "Target->Client")
        done <- struct{}{}
    }()
    
    <-done
}

func (p *Proxy) forwardWithLogging(dst, src net.Conn, direction string) {
    reader := bufio.NewReader(src)
    
    for {
        line, err := reader.ReadString('\n')
        if err != nil {
            if err != io.EOF {
                log.Printf("Read error (%s): %v", direction, err)
            }
            return
        }
        
        fmt.Printf("[%s] %s", direction, line)
        
        modified := strings.ToUpper(line)
        
        _, err = dst.Write([]byte(modified))
        if err != nil {
            log.Printf("Write error (%s): %v", direction, err)
            return
        }
    }
}

func main() {
    go startTargetServer()
    
    proxy := NewProxy(":8080", "localhost:9000")
    
    if err := proxy.Start(); err != nil {
        log.Fatalf("Proxy failed: %v", err)
    }
}

func startTargetServer() {
    listener, err := net.Listen("tcp", ":9000")
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
            io.Copy(c, c)
        }(conn)
    }
}
```

The transparent proxy intercepts and logs traffic while forwarding it to the  
target. It can modify data in transit, enabling protocol translation,  
content filtering, or debugging.  

## Unix Domain Sockets

Unix domain sockets provide efficient IPC on the same host without network  
overhead.  

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net"
    "os"
)

const SocketPath = "/tmp/go-socket.sock"

func main() {
    os.Remove(SocketPath)
    
    go startUnixServer()
    
    conn, err := net.Dial("unix", SocketPath)
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    message := "Hello via Unix socket\n"
    _, err = conn.Write([]byte(message))
    if err != nil {
        log.Fatalf("Write failed: %v", err)
    }
    
    fmt.Printf("Sent: %s", message)
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil && err != io.EOF {
        log.Fatalf("Read failed: %v", err)
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
    
    os.Remove(SocketPath)
}

func startUnixServer() {
    listener, err := net.Listen("unix", SocketPath)
    if err != nil {
        log.Fatalf("Listen failed: %v", err)
    }
    defer listener.Close()
    
    fmt.Println("Unix socket server listening")
    
    conn, err := listener.Accept()
    if err != nil {
        log.Printf("Accept error: %v", err)
        return
    }
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil {
        log.Printf("Read error: %v", err)
        return
    }
    
    fmt.Printf("Server received: %s", string(buffer[:n]))
    
    conn.Write([]byte("Unix socket response\n"))
}
```

Unix domain sockets use filesystem paths instead of IP addresses and ports.  
They're faster than TCP sockets for local communication and support passing  
file descriptors between processes.  

## TLS/SSL Secure Connections

TLS encryption secures network communication against eavesdropping and  
tampering.  

```go
package main

import (
    "crypto/tls"
    "crypto/x509"
    "fmt"
    "io"
    "log"
)

func main() {
    go startTLSServer()
    
    cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
    if err != nil {
        fmt.Printf("Note: TLS example requires certificates\n")
        fmt.Printf("Generate with: openssl req -x509 -newkey rsa:4096 " +
            "-keyout server.key -out server.crt -days 365 -nodes\n")
        return
    }
    
    config := &tls.Config{
        Certificates:       []tls.Certificate{cert},
        InsecureSkipVerify: true,
    }
    
    conn, err := tls.Dial("tcp", "localhost:8443", config)
    if err != nil {
        log.Printf("Dial failed: %v", err)
        return
    }
    defer conn.Close()
    
    fmt.Println("TLS connection established")
    
    _, err = conn.Write([]byte("Secure message\n"))
    if err != nil {
        log.Printf("Write failed: %v", err)
        return
    }
    
    buffer := make([]byte, 1024)
    n, err := conn.Read(buffer)
    if err != nil && err != io.EOF {
        log.Printf("Read failed: %v", err)
        return
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
}

func startTLSServer() {
    cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
    if err != nil {
        return
    }
    
    config := &tls.Config{
        Certificates: []tls.Certificate{cert},
    }
    
    listener, err := tls.Listen("tcp", ":8443", config)
    if err != nil {
        return
    }
    defer listener.Close()
    
    conn, err := listener.Accept()
    if err != nil {
        return
    }
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, _ := conn.Read(buffer)
    
    conn.Write([]byte(fmt.Sprintf("Echo: %s", string(buffer[:n]))))
}
```

TLS wraps TCP connections with encryption, providing confidentiality and  
authentication. Certificates verify server identity. For production, use  
proper certificate validation instead of InsecureSkipVerify.  

## Connection Timeout Handling

Comprehensive timeout handling prevents resource leaks from hanging  
connections.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

type TimeoutConn struct {
    conn         net.Conn
    readTimeout  time.Duration
    writeTimeout time.Duration
}

func NewTimeoutConn(conn net.Conn, read, write time.Duration) *TimeoutConn {
    return &TimeoutConn{
        conn:         conn,
        readTimeout:  read,
        writeTimeout: write,
    }
}

func (tc *TimeoutConn) Read(b []byte) (int, error) {
    if tc.readTimeout > 0 {
        tc.conn.SetReadDeadline(time.Now().Add(tc.readTimeout))
    }
    return tc.conn.Read(b)
}

func (tc *TimeoutConn) Write(b []byte) (int, error) {
    if tc.writeTimeout > 0 {
        tc.conn.SetWriteDeadline(time.Now().Add(tc.writeTimeout))
    }
    return tc.conn.Write(b)
}

func (tc *TimeoutConn) Close() error {
    return tc.conn.Close()
}

func main() {
    go startTimeoutServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    
    timeoutConn := NewTimeoutConn(conn, 2*time.Second, 2*time.Second)
    defer timeoutConn.Close()
    
    _, err = timeoutConn.Write([]byte("Test message\n"))
    if err != nil {
        log.Printf("Write error: %v", err)
        return
    }
    
    buffer := make([]byte, 1024)
    n, err := timeoutConn.Read(buffer)
    if err != nil {
        log.Printf("Read error: %v", err)
        return
    }
    
    fmt.Printf("Received: %s", string(buffer[:n]))
}

func startTimeoutServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, _ := conn.Read(buffer)
    conn.Write(buffer[:n])
}
```

The timeout wrapper automatically sets deadlines before each I/O operation,  
providing a consistent timeout interface. This simplifies timeout management  
throughout application code.  

## Heartbeat Protocol

Heartbeats detect connection failures and maintain connection liveness in  
long-running sessions.  

```go
package main

import (
    "fmt"
    "log"
    "net"
    "time"
)

type HeartbeatConn struct {
    conn     net.Conn
    interval time.Duration
    timeout  time.Duration
    stopChan chan struct{}
}

func NewHeartbeatConn(conn net.Conn, interval, timeout time.Duration) *HeartbeatConn {
    hc := &HeartbeatConn{
        conn:     conn,
        interval: interval,
        timeout:  timeout,
        stopChan: make(chan struct{}),
    }
    
    go hc.sender()
    go hc.receiver()
    
    return hc
}

func (hc *HeartbeatConn) sender() {
    ticker := time.NewTicker(hc.interval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            hc.conn.SetWriteDeadline(time.Now().Add(hc.timeout))
            _, err := hc.conn.Write([]byte("HEARTBEAT\n"))
            if err != nil {
                fmt.Printf("Heartbeat send failed: %v\n", err)
                return
            }
        case <-hc.stopChan:
            return
        }
    }
}

func (hc *HeartbeatConn) receiver() {
    buffer := make([]byte, 128)
    
    for {
        select {
        case <-hc.stopChan:
            return
        default:
            hc.conn.SetReadDeadline(time.Now().Add(hc.interval * 2))
            n, err := hc.conn.Read(buffer)
            if err != nil {
                if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
                    fmt.Println("Heartbeat timeout - connection lost")
                    return
                }
                return
            }
            
            if string(buffer[:n]) == "HEARTBEAT\n" {
                fmt.Println("Heartbeat received")
            }
        }
    }
}

func (hc *HeartbeatConn) Close() {
    close(hc.stopChan)
    hc.conn.Close()
}

func main() {
    go startHeartbeatServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    
    hbConn := NewHeartbeatConn(conn, 2*time.Second, 1*time.Second)
    defer hbConn.Close()
    
    time.Sleep(10 * time.Second)
}

func startHeartbeatServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    hbConn := NewHeartbeatConn(conn, 2*time.Second, 1*time.Second)
    defer hbConn.Close()
    
    time.Sleep(15 * time.Second)
}
```

Heartbeat protocols send periodic HEARTBEAT messages to verify connection  
liveness. Missing heartbeats beyond the timeout indicate connection failure,  
allowing prompt detection and recovery.  

## Message Acknowledgment Protocol

Acknowledgments ensure reliable message delivery in unreliable transports.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strconv"
    "strings"
    "time"
)

type AckMessage struct {
    ID      int
    Content string
}

func sendWithAck(conn net.Conn, msg AckMessage) error {
    line := fmt.Sprintf("%d:%s\n", msg.ID, msg.Content)
    
    _, err := conn.Write([]byte(line))
    if err != nil {
        return err
    }
    
    conn.SetReadDeadline(time.Now().Add(5 * time.Second))
    
    reader := bufio.NewReader(conn)
    ack, err := reader.ReadString('\n')
    if err != nil {
        return err
    }
    
    expectedAck := fmt.Sprintf("ACK:%d\n", msg.ID)
    if strings.TrimSpace(ack) != strings.TrimSpace(expectedAck) {
        return fmt.Errorf("invalid ack: got %s, want %s", ack, expectedAck)
    }
    
    return nil
}

func main() {
    go startAckServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    messages := []AckMessage{
        {ID: 1, Content: "First message"},
        {ID: 2, Content: "Second message"},
        {ID: 3, Content: "Third message"},
    }
    
    for _, msg := range messages {
        err := sendWithAck(conn, msg)
        if err != nil {
            log.Printf("Send failed for message %d: %v", msg.ID, err)
            continue
        }
        
        fmt.Printf("Message %d acknowledged\n", msg.ID)
    }
}

func startAckServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    scanner := bufio.NewScanner(conn)
    
    for scanner.Scan() {
        line := scanner.Text()
        parts := strings.SplitN(line, ":", 2)
        
        if len(parts) != 2 {
            continue
        }
        
        id, err := strconv.Atoi(parts[0])
        if err != nil {
            continue
        }
        
        fmt.Printf("Received message %d: %s\n", id, parts[1])
        
        ack := fmt.Sprintf("ACK:%d\n", id)
        conn.Write([]byte(ack))
    }
}
```

Acknowledgment protocols require receivers to confirm message receipt.  
Senders wait for ACKs before considering messages delivered successfully.  
This enables retransmission on timeout for reliability.  

## Connection Metrics Collection

Collecting metrics provides insights into connection behavior and performance.  

```go
package main

import (
    "fmt"
    "io"
    "net"
    "sync/atomic"
    "time"
)

type ConnectionMetrics struct {
    bytesRead      uint64
    bytesWritten   uint64
    messagesRead   uint64
    messagesWritten uint64
    startTime      time.Time
    lastActivity   time.Time
    errors         uint64
}

func NewConnectionMetrics() *ConnectionMetrics {
    now := time.Now()
    return &ConnectionMetrics{
        startTime:    now,
        lastActivity: now,
    }
}

func (cm *ConnectionMetrics) RecordRead(bytes int) {
    atomic.AddUint64(&cm.bytesRead, uint64(bytes))
    atomic.AddUint64(&cm.messagesRead, 1)
    cm.lastActivity = time.Now()
}

func (cm *ConnectionMetrics) RecordWrite(bytes int) {
    atomic.AddUint64(&cm.bytesWritten, uint64(bytes))
    atomic.AddUint64(&cm.messagesWritten, 1)
    cm.lastActivity = time.Now()
}

func (cm *ConnectionMetrics) RecordError() {
    atomic.AddUint64(&cm.errors, 1)
}

func (cm *ConnectionMetrics) Report() {
    duration := time.Since(cm.startTime).Seconds()
    
    bytesRead := atomic.LoadUint64(&cm.bytesRead)
    bytesWritten := atomic.LoadUint64(&cm.bytesWritten)
    messagesRead := atomic.LoadUint64(&cm.messagesRead)
    messagesWritten := atomic.LoadUint64(&cm.messagesWritten)
    errors := atomic.LoadUint64(&cm.errors)
    
    fmt.Println("=== Connection Metrics ===")
    fmt.Printf("Duration: %.2f seconds\n", duration)
    fmt.Printf("Bytes read: %d (%.2f KB/s)\n", bytesRead, float64(bytesRead)/duration/1024)
    fmt.Printf("Bytes written: %d (%.2f KB/s)\n", bytesWritten, float64(bytesWritten)/duration/1024)
    fmt.Printf("Messages read: %d (%.2f/s)\n", messagesRead, float64(messagesRead)/duration)
    fmt.Printf("Messages written: %d (%.2f/s)\n", messagesWritten, float64(messagesWritten)/duration)
    fmt.Printf("Errors: %d\n", errors)
    fmt.Printf("Last activity: %v ago\n", time.Since(cm.lastActivity))
}

func main() {
    go startMetricsServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    defer conn.Close()
    
    metrics := NewConnectionMetrics()
    
    data := make([]byte, 1024)
    for i := 0; i < 10; i++ {
        n, err := conn.Write(data)
        if err != nil {
            metrics.RecordError()
            break
        }
        metrics.RecordWrite(n)
        
        time.Sleep(100 * time.Millisecond)
    }
    
    time.Sleep(500 * time.Millisecond)
    metrics.Report()
}

func startMetricsServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    io.Copy(io.Discard, conn)
}
```

Connection metrics track throughput, message counts, error rates, and timing  
information. Regular metrics reporting enables performance monitoring and  
capacity planning.  

## Custom Connection Wrapper

Wrapping connections enables adding cross-cutting concerns like logging,  
metrics, or compression.  

```go
package main

import (
    "fmt"
    "net"
    "time"
)

type WrappedConn struct {
    conn   net.Conn
    logger func(string, ...interface{})
}

func WrapConnection(conn net.Conn, logger func(string, ...interface{})) *WrappedConn {
    return &WrappedConn{
        conn:   conn,
        logger: logger,
    }
}

func (wc *WrappedConn) Read(b []byte) (int, error) {
    wc.logger("Read called with buffer size %d", len(b))
    
    start := time.Now()
    n, err := wc.conn.Read(b)
    duration := time.Since(start)
    
    if err != nil {
        wc.logger("Read error: %v", err)
    } else {
        wc.logger("Read %d bytes in %v", n, duration)
    }
    
    return n, err
}

func (wc *WrappedConn) Write(b []byte) (int, error) {
    wc.logger("Write called with %d bytes", len(b))
    
    start := time.Now()
    n, err := wc.conn.Write(b)
    duration := time.Since(start)
    
    if err != nil {
        wc.logger("Write error: %v", err)
    } else {
        wc.logger("Wrote %d bytes in %v", n, duration)
    }
    
    return n, err
}

func (wc *WrappedConn) Close() error {
    wc.logger("Closing connection")
    return wc.conn.Close()
}

func main() {
    go startWrappedServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    
    logger := func(format string, args ...interface{}) {
        fmt.Printf("[LOG] "+format+"\n", args...)
    }
    
    wrapped := WrapConnection(conn, logger)
    defer wrapped.Close()
    
    wrapped.Write([]byte("Test message\n"))
    
    buffer := make([]byte, 1024)
    wrapped.Read(buffer)
}

func startWrappedServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, _ := conn.Read(buffer)
    conn.Write(buffer[:n])
}
```

Connection wrappers implement the `net.Conn` interface while adding behavior.  
This decorator pattern enables composing multiple wrappers for layered  
functionality like logging, encryption, and compression.  

## Protocol Negotiation

Protocol negotiation allows clients and servers to agree on communication  
format and features.  

```go
package main

import (
    "bufio"
    "fmt"
    "log"
    "net"
    "strings"
)

const (
    ProtocolV1 = "PROTO/1.0"
    ProtocolV2 = "PROTO/2.0"
)

type ProtocolHandler interface {
    HandleMessage(msg string) string
}

type V1Handler struct{}

func (h *V1Handler) HandleMessage(msg string) string {
    return strings.ToUpper(msg)
}

type V2Handler struct{}

func (h *V2Handler) HandleMessage(msg string) string {
    return fmt.Sprintf("[v2] %s", strings.ToUpper(msg))
}

func negotiateProtocol(conn net.Conn) (ProtocolHandler, error) {
    conn.Write([]byte("PROTO?\n"))
    
    reader := bufio.NewReader(conn)
    response, err := reader.ReadString('\n')
    if err != nil {
        return nil, err
    }
    
    protocol := strings.TrimSpace(response)
    
    switch protocol {
    case ProtocolV1:
        fmt.Println("Negotiated protocol v1")
        return &V1Handler{}, nil
    case ProtocolV2:
        fmt.Println("Negotiated protocol v2")
        return &V2Handler{}, nil
    default:
        return nil, fmt.Errorf("unsupported protocol: %s", protocol)
    }
}

func main() {
    go startNegotiationServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    reader := bufio.NewReader(conn)
    
    request, _ := reader.ReadString('\n')
    if strings.TrimSpace(request) == "PROTO?" {
        conn.Write([]byte(ProtocolV2 + "\n"))
    }
    
    conn.Write([]byte("test message\n"))
    
    response, _ := reader.ReadString('\n')
    fmt.Printf("Response: %s", response)
}

func startNegotiationServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    handler, err := negotiateProtocol(conn)
    if err != nil {
        log.Printf("Negotiation failed: %v", err)
        return
    }
    
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        msg := scanner.Text()
        response := handler.HandleMessage(msg)
        conn.Write([]byte(response + "\n"))
    }
}
```

Protocol negotiation exchanges version information at connection start,  
allowing both parties to agree on capabilities. This enables backward  
compatibility and feature detection.  

## Streaming JSON Protocol

Streaming JSON enables efficient transmission of structured data over  
network connections.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net"
)

type Message struct {
    Type    string      `json:"type"`
    ID      int         `json:"id"`
    Payload interface{} `json:"payload"`
}

func sendJSON(conn net.Conn, msg Message) error {
    encoder := json.NewEncoder(conn)
    return encoder.Encode(msg)
}

func receiveJSON(conn net.Conn) (*Message, error) {
    var msg Message
    decoder := json.NewDecoder(conn)
    err := decoder.Decode(&msg)
    return &msg, err
}

func main() {
    go startJSONServer()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    messages := []Message{
        {Type: "greeting", ID: 1, Payload: "Hello"},
        {Type: "data", ID: 2, Payload: map[string]int{"value": 42}},
        {Type: "close", ID: 3, Payload: nil},
    }
    
    for _, msg := range messages {
        err := sendJSON(conn, msg)
        if err != nil {
            log.Printf("Send error: %v", err)
            break
        }
        
        fmt.Printf("Sent: %+v\n", msg)
        
        response, err := receiveJSON(conn)
        if err != nil {
            log.Printf("Receive error: %v", err)
            break
        }
        
        fmt.Printf("Received: %+v\n", response)
    }
}

func startJSONServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    for {
        msg, err := receiveJSON(conn)
        if err != nil {
            return
        }
        
        response := Message{
            Type:    "ack",
            ID:      msg.ID,
            Payload: fmt.Sprintf("Processed %s", msg.Type),
        }
        
        sendJSON(conn, response)
        
        if msg.Type == "close" {
            break
        }
    }
}
```

Streaming JSON uses `json.Encoder` and `json.Decoder` for efficient  
serialization directly to network connections. This is cleaner than  
marshaling to bytes first and supports streaming of multiple objects.  

## Circuit Breaker Pattern

Circuit breakers prevent cascading failures by temporarily blocking requests  
to failing services.  

```go
package main

import (
    "errors"
    "fmt"
    "net"
    "sync"
    "time"
)

type CircuitState int

const (
    StateClosed CircuitState = iota
    StateOpen
    StateHalfOpen
)

type CircuitBreaker struct {
    maxFailures  int
    resetTimeout time.Duration
    
    mu           sync.RWMutex
    state        CircuitState
    failures     int
    lastFailTime time.Time
}

func NewCircuitBreaker(maxFailures int, resetTimeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures:  maxFailures,
        resetTimeout: resetTimeout,
        state:        StateClosed,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    
    if cb.state == StateOpen {
        if time.Since(cb.lastFailTime) > cb.resetTimeout {
            cb.state = StateHalfOpen
            fmt.Println("Circuit breaker: half-open")
        } else {
            cb.mu.Unlock()
            return errors.New("circuit breaker open")
        }
    }
    
    cb.mu.Unlock()
    
    err := fn()
    
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    if err != nil {
        cb.failures++
        cb.lastFailTime = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = StateOpen
            fmt.Printf("Circuit breaker: opened after %d failures\n", cb.failures)
        }
        
        return err
    }
    
    if cb.state == StateHalfOpen {
        cb.state = StateClosed
        cb.failures = 0
        fmt.Println("Circuit breaker: closed")
    }
    
    return nil
}

func main() {
    cb := NewCircuitBreaker(3, 5*time.Second)
    
    connect := func() error {
        conn, err := net.DialTimeout("tcp", "localhost:9999", 1*time.Second)
        if err != nil {
            return err
        }
        conn.Close()
        return nil
    }
    
    for i := 0; i < 10; i++ {
        err := cb.Call(connect)
        if err != nil {
            fmt.Printf("Attempt %d failed: %v\n", i+1, err)
        } else {
            fmt.Printf("Attempt %d succeeded\n", i+1)
        }
        
        time.Sleep(1 * time.Second)
    }
}
```

Circuit breakers track failure rates and open the circuit after exceeding  
thresholds. This prevents wasting resources on failing services and allows  
time for recovery.  

## Request Batching

Batching groups multiple requests into single network operations for  
efficiency.  

```go
package main

import (
    "fmt"
    "net"
    "strings"
    "sync"
    "time"
)

type Batcher struct {
    conn        net.Conn
    batchSize   int
    flushInterval time.Duration
    
    mu      sync.Mutex
    buffer  []string
    stopChan chan struct{}
}

func NewBatcher(conn net.Conn, batchSize int, interval time.Duration) *Batcher {
    b := &Batcher{
        conn:          conn,
        batchSize:     batchSize,
        flushInterval: interval,
        buffer:        make([]string, 0, batchSize),
        stopChan:      make(chan struct{}),
    }
    
    go b.autoFlush()
    
    return b
}

func (b *Batcher) Add(item string) error {
    b.mu.Lock()
    defer b.mu.Unlock()
    
    b.buffer = append(b.buffer, item)
    
    if len(b.buffer) >= b.batchSize {
        return b.flushLocked()
    }
    
    return nil
}

func (b *Batcher) Flush() error {
    b.mu.Lock()
    defer b.mu.Unlock()
    return b.flushLocked()
}

func (b *Batcher) flushLocked() error {
    if len(b.buffer) == 0 {
        return nil
    }
    
    batch := strings.Join(b.buffer, ",")
    message := fmt.Sprintf("BATCH:%s\n", batch)
    
    _, err := b.conn.Write([]byte(message))
    if err != nil {
        return err
    }
    
    fmt.Printf("Flushed batch of %d items\n", len(b.buffer))
    b.buffer = b.buffer[:0]
    
    return nil
}

func (b *Batcher) autoFlush() {
    ticker := time.NewTicker(b.flushInterval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            b.Flush()
        case <-b.stopChan:
            return
        }
    }
}

func (b *Batcher) Close() error {
    close(b.stopChan)
    b.Flush()
    return nil
}

func main() {
    go startBatchServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    defer conn.Close()
    
    batcher := NewBatcher(conn, 5, 2*time.Second)
    defer batcher.Close()
    
    for i := 1; i <= 12; i++ {
        batcher.Add(fmt.Sprintf("item%d", i))
        time.Sleep(300 * time.Millisecond)
    }
    
    time.Sleep(1 * time.Second)
}

func startBatchServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 4096)
    for {
        n, err := conn.Read(buffer)
        if err != nil {
            return
        }
        fmt.Printf("Server received: %s", string(buffer[:n]))
    }
}
```

Request batching accumulates items and sends them together, either when the  
batch is full or after a timeout. This amortizes network overhead across  
multiple operations, improving throughput.  

## Connection Pooling with Priority Queues

Priority-based connection pooling serves high-priority requests first for  
better resource allocation.  

```go
package main

import (
    "container/heap"
    "fmt"
    "net"
    "sync"
)

type PriorityRequest struct {
    priority int
    callback chan net.Conn
    index    int
}

type PriorityQueue []*PriorityRequest

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].priority > pq[j].priority
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*PriorityRequest)
    item.index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil
    item.index = -1
    *pq = old[0 : n-1]
    return item
}

type PriorityPool struct {
    address string
    maxSize int
    
    mu      sync.Mutex
    pool    chan net.Conn
    waiting PriorityQueue
}

func NewPriorityPool(address string, maxSize int) *PriorityPool {
    pp := &PriorityPool{
        address: address,
        maxSize: maxSize,
        pool:    make(chan net.Conn, maxSize),
        waiting: make(PriorityQueue, 0),
    }
    
    heap.Init(&pp.waiting)
    
    return pp
}

func (pp *PriorityPool) Get(priority int) net.Conn {
    select {
    case conn := <-pp.pool:
        return conn
    default:
        pp.mu.Lock()
        
        callback := make(chan net.Conn, 1)
        req := &PriorityRequest{
            priority: priority,
            callback: callback,
        }
        
        heap.Push(&pp.waiting, req)
        pp.mu.Unlock()
        
        return <-callback
    }
}

func (pp *PriorityPool) Put(conn net.Conn) {
    pp.mu.Lock()
    
    if pp.waiting.Len() > 0 {
        req := heap.Pop(&pp.waiting).(*PriorityRequest)
        pp.mu.Unlock()
        req.callback <- conn
        return
    }
    
    pp.mu.Unlock()
    
    select {
    case pp.pool <- conn:
    default:
        conn.Close()
    }
}

func main() {
    pool := NewPriorityPool("localhost:8080", 2)
    
    fmt.Println("Priority pool demonstration")
}
```

Priority queues ensure high-priority requests receive connections first when  
resources are limited. This improves responsiveness for critical operations  
while still serving lower-priority requests.  

## Data Encryption Over Sockets

Custom encryption layers protect data without requiring TLS infrastructure.  

```go
package main

import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "fmt"
    "io"
    "net"
)

type EncryptedConn struct {
    conn   net.Conn
    gcm    cipher.AEAD
    nonce  []byte
}

func NewEncryptedConn(conn net.Conn, key []byte) (*EncryptedConn, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    return &EncryptedConn{
        conn: conn,
        gcm:  gcm,
    }, nil
}

func (ec *EncryptedConn) Write(data []byte) (int, error) {
    nonce := make([]byte, ec.gcm.NonceSize())
    if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
        return 0, err
    }
    
    encrypted := ec.gcm.Seal(nonce, nonce, data, nil)
    
    return ec.conn.Write(encrypted)
}

func (ec *EncryptedConn) Read(data []byte) (int, error) {
    encrypted := make([]byte, len(data)+ec.gcm.Overhead()+ec.gcm.NonceSize())
    
    n, err := ec.conn.Read(encrypted)
    if err != nil {
        return 0, err
    }
    
    nonceSize := ec.gcm.NonceSize()
    if n < nonceSize {
        return 0, fmt.Errorf("ciphertext too short")
    }
    
    nonce, ciphertext := encrypted[:nonceSize], encrypted[nonceSize:n]
    
    plaintext, err := ec.gcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return 0, err
    }
    
    copy(data, plaintext)
    return len(plaintext), nil
}

func (ec *EncryptedConn) Close() error {
    return ec.conn.Close()
}

func main() {
    key := make([]byte, 32)
    rand.Read(key)
    
    fmt.Println("Encrypted connection using AES-GCM")
    fmt.Printf("Key: %x\n", key[:8])
}
```

AES-GCM encryption provides authenticated encryption at the application layer.  
Each message has a unique nonce for security. This is useful when TLS isn't  
available or custom encryption is required.  

## Async Message Processing

Asynchronous processing decouples message reception from handling for better  
throughput.  

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

type AsyncProcessor struct {
    workers   int
    queue     chan string
    wg        sync.WaitGroup
    processor func(string) string
}

func NewAsyncProcessor(workers int, processor func(string) string) *AsyncProcessor {
    ap := &AsyncProcessor{
        workers:   workers,
        queue:     make(chan string, 100),
        processor: processor,
    }
    
    for i := 0; i < workers; i++ {
        ap.wg.Add(1)
        go ap.worker(i)
    }
    
    return ap
}

func (ap *AsyncProcessor) worker(id int) {
    defer ap.wg.Done()
    
    for msg := range ap.queue {
        result := ap.processor(msg)
        fmt.Printf("Worker %d processed: %s -> %s\n", id, msg, result)
        time.Sleep(100 * time.Millisecond)
    }
}

func (ap *AsyncProcessor) Submit(msg string) {
    ap.queue <- msg
}

func (ap *AsyncProcessor) Close() {
    close(ap.queue)
    ap.wg.Wait()
}

func main() {
    go startAsyncServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        log.Fatalf("Dial failed: %v", err)
    }
    defer conn.Close()
    
    messages := []string{"msg1", "msg2", "msg3", "msg4", "msg5"}
    
    for _, msg := range messages {
        conn.Write([]byte(msg + "\n"))
        time.Sleep(50 * time.Millisecond)
    }
    
    time.Sleep(2 * time.Second)
}

func startAsyncServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    processor := NewAsyncProcessor(3, func(msg string) string {
        return fmt.Sprintf("processed_%s", msg)
    })
    defer processor.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    scanner := bufio.NewScanner(conn)
    for scanner.Scan() {
        processor.Submit(scanner.Text())
    }
}
```

Async processing uses worker pools to handle messages concurrently. This  
prevents slow message processing from blocking network I/O, improving overall  
throughput and responsiveness.  

## Connection Tagging and Routing

Tags enable routing connections to specialized handlers based on metadata.  

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "strings"
)

type TaggedConnection struct {
    conn net.Conn
    tags map[string]string
}

type Router struct {
    routes map[string]func(*TaggedConnection)
}

func NewRouter() *Router {
    return &Router{
        routes: make(map[string]func(*TaggedConnection)),
    }
}

func (r *Router) Register(tag string, handler func(*TaggedConnection)) {
    r.routes[tag] = handler
}

func (r *Router) Route(tc *TaggedConnection) {
    if handler, ok := r.routes[tc.tags["type"]]; ok {
        handler(tc)
    } else {
        defaultHandler(tc)
    }
}

func defaultHandler(tc *TaggedConnection) {
    tc.conn.Write([]byte("Unknown connection type\n"))
    tc.conn.Close()
}

func handleHTTPLike(tc *TaggedConnection) {
    fmt.Printf("Handling HTTP-like connection from %s\n", tc.conn.RemoteAddr())
    tc.conn.Write([]byte("HTTP/1.0 200 OK\n\nHello\n"))
    tc.conn.Close()
}

func handleCustom(tc *TaggedConnection) {
    fmt.Printf("Handling custom connection from %s\n", tc.conn.RemoteAddr())
    tc.conn.Write([]byte("CUSTOM_RESPONSE\n"))
    tc.conn.Close()
}

func main() {
    router := NewRouter()
    router.Register("http", handleHTTPLike)
    router.Register("custom", handleCustom)
    
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Listen failed: %v\n", err)
        return
    }
    defer listener.Close()
    
    fmt.Println("Tagged routing server on :8080")
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        go handleTaggedConnection(conn, router)
    }
}

func handleTaggedConnection(conn net.Conn, router *Router) {
    reader := bufio.NewReader(conn)
    
    firstLine, err := reader.ReadString('\n')
    if err != nil {
        conn.Close()
        return
    }
    
    tc := &TaggedConnection{
        conn: conn,
        tags: make(map[string]string),
    }
    
    if strings.HasPrefix(firstLine, "GET") || strings.HasPrefix(firstLine, "POST") {
        tc.tags["type"] = "http"
    } else {
        tc.tags["type"] = "custom"
    }
    
    router.Route(tc)
}
```

Connection routing inspects initial data to classify connections and route  
them to appropriate handlers. This enables protocol multiplexing on a single  
port.  

## Socket Buffer Tuning

Optimizing socket buffers improves performance for high-throughput  
applications.  

```go
package main

import (
    "fmt"
    "net"
    "syscall"
)

func tuneSocketBuffers(conn net.Conn, recvBuf, sendBuf int) error {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fmt.Errorf("not a TCP connection")
    }
    
    rawConn, err := tcpConn.SyscallConn()
    if err != nil {
        return err
    }
    
    var sockErr error
    err = rawConn.Control(func(fd uintptr) {
        if recvBuf > 0 {
            sockErr = syscall.SetsockoptInt(int(fd), 
                syscall.SOL_SOCKET, syscall.SO_RCVBUF, recvBuf)
            if sockErr != nil {
                return
            }
        }
        
        if sendBuf > 0 {
            sockErr = syscall.SetsockoptInt(int(fd), 
                syscall.SOL_SOCKET, syscall.SO_SNDBUF, sendBuf)
        }
    })
    
    if err != nil {
        return err
    }
    
    return sockErr
}

func getSocketBuffers(conn net.Conn) (int, int, error) {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return 0, 0, fmt.Errorf("not a TCP connection")
    }
    
    rawConn, err := tcpConn.SyscallConn()
    if err != nil {
        return 0, 0, err
    }
    
    var recvBuf, sendBuf int
    var sockErr error
    
    err = rawConn.Control(func(fd uintptr) {
        recvBuf, sockErr = syscall.GetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_RCVBUF)
        if sockErr != nil {
            return
        }
        
        sendBuf, sockErr = syscall.GetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_SNDBUF)
    })
    
    if err != nil {
        return 0, 0, err
    }
    
    return recvBuf, sendBuf, sockErr
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Listen failed: %v\n", err)
        return
    }
    defer listener.Close()
    
    fmt.Println("Buffer tuning server on :8080")
    
    conn, err := listener.Accept()
    if err != nil {
        fmt.Printf("Accept failed: %v\n", err)
        return
    }
    defer conn.Close()
    
    oldRecv, oldSend, err := getSocketBuffers(conn)
    if err != nil {
        fmt.Printf("Get buffers failed: %v\n", err)
        return
    }
    
    fmt.Printf("Default buffers: recv=%d, send=%d\n", oldRecv, oldSend)
    
    err = tuneSocketBuffers(conn, 131072, 131072)
    if err != nil {
        fmt.Printf("Tune failed: %v\n", err)
        return
    }
    
    newRecv, newSend, err := getSocketBuffers(conn)
    if err != nil {
        fmt.Printf("Get buffers failed: %v\n", err)
        return
    }
    
    fmt.Printf("Tuned buffers: recv=%d, send=%d\n", newRecv, newSend)
}
```

Socket buffer sizes affect throughput and latency. Larger buffers improve  
throughput for bulk transfers but increase latency. Tuning should match  
application needs and network conditions.  

## Connection Draining

Draining ensures all buffered data is transmitted before closing connections.  

```go
package main

import (
    "fmt"
    "io"
    "net"
    "time"
)

func drainConnection(conn net.Conn, timeout time.Duration) error {
    if tcpConn, ok := conn.(*net.TCPConn); ok {
        tcpConn.CloseWrite()
        
        conn.SetReadDeadline(time.Now().Add(timeout))
        
        buffer := make([]byte, 4096)
        for {
            _, err := conn.Read(buffer)
            if err != nil {
                if err == io.EOF {
                    return nil
                }
                if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
                    return fmt.Errorf("drain timeout")
                }
                return err
            }
        }
    }
    
    return nil
}

func main() {
    go startDrainServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    
    conn.Write([]byte("Test message\n"))
    
    fmt.Println("Draining connection...")
    err = drainConnection(conn, 5*time.Second)
    if err != nil {
        fmt.Printf("Drain error: %v\n", err)
    } else {
        fmt.Println("Connection drained successfully")
    }
    
    conn.Close()
}

func startDrainServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    buffer := make([]byte, 1024)
    n, _ := conn.Read(buffer)
    
    conn.Write([]byte("Response\n"))
    
    time.Sleep(500 * time.Millisecond)
    
    conn.(*net.TCPConn).CloseWrite()
}
```

Connection draining calls `CloseWrite` to signal no more data will be sent,  
then reads remaining data from the peer. This ensures graceful connection  
shutdown without data loss.  

## Message Sequencing

Sequence numbers ensure messages are processed in order despite network  
reordering.  

```go
package main

import (
    "fmt"
    "net"
    "sort"
    "strconv"
    "strings"
    "sync"
)

type SequencedMessage struct {
    Sequence int
    Data     string
}

type SequenceTracker struct {
    expected int
    buffer   map[int]string
    mu       sync.Mutex
}

func NewSequenceTracker() *SequenceTracker {
    return &SequenceTracker{
        expected: 0,
        buffer:   make(map[int]string),
    }
}

func (st *SequenceTracker) Process(msg SequencedMessage) []string {
    st.mu.Lock()
    defer st.mu.Unlock()
    
    var ready []string
    
    if msg.Sequence == st.expected {
        ready = append(ready, msg.Data)
        st.expected++
        
        for {
            if data, ok := st.buffer[st.expected]; ok {
                ready = append(ready, data)
                delete(st.buffer, st.expected)
                st.expected++
            } else {
                break
            }
        }
    } else if msg.Sequence > st.expected {
        st.buffer[msg.Sequence] = msg.Data
    }
    
    return ready
}

func main() {
    tracker := NewSequenceTracker()
    
    messages := []SequencedMessage{
        {Sequence: 2, Data: "Third"},
        {Sequence: 0, Data: "First"},
        {Sequence: 1, Data: "Second"},
        {Sequence: 3, Data: "Fourth"},
    }
    
    for _, msg := range messages {
        ready := tracker.Process(msg)
        
        fmt.Printf("Received seq %d: %s\n", msg.Sequence, msg.Data)
        
        if len(ready) > 0 {
            fmt.Printf("  Ready to process: %v\n", ready)
        }
    }
}
```

Sequence tracking buffers out-of-order messages and delivers them in sequence  
order. This compensates for network reordering while maintaining message  
ordering semantics.  

## Nagle's Algorithm Control

Controlling Nagle's algorithm trades latency for efficiency in specific  
scenarios.  

```go
package main

import (
    "fmt"
    "net"
    "time"
)

func setNoDelay(conn net.Conn, noDelay bool) error {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fmt.Errorf("not a TCP connection")
    }
    
    return tcpConn.SetNoDelay(noDelay)
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Listen failed: %v\n", err)
        return
    }
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        buffer := make([]byte, 1024)
        for {
            n, err := conn.Read(buffer)
            if err != nil {
                return
            }
            conn.Write(buffer[:n])
        }
    }()
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    defer conn.Close()
    
    fmt.Println("Testing with Nagle enabled (default)")
    testLatency(conn, 10)
    
    setNoDelay(conn, true)
    fmt.Println("\nTesting with Nagle disabled (TCP_NODELAY)")
    testLatency(conn, 10)
}

func testLatency(conn net.Conn, iterations int) {
    var total time.Duration
    
    for i := 0; i < iterations; i++ {
        start := time.Now()
        
        conn.Write([]byte("ping"))
        
        buffer := make([]byte, 4)
        conn.Read(buffer)
        
        latency := time.Since(start)
        total += latency
    }
    
    avg := total / time.Duration(iterations)
    fmt.Printf("Average latency: %v\n", avg)
}
```

Disabling Nagle's algorithm with TCP_NODELAY reduces latency for small  
messages but may increase bandwidth usage. Enable for interactive protocols,  
disable for bulk transfers.  

## Connection Linger Configuration

Linger control determines how connections close when data remains unsent.  

```go
package main

import (
    "fmt"
    "net"
    "syscall"
    "time"
)

func setLinger(conn net.Conn, linger int) error {
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fmt.Errorf("not a TCP connection")
    }
    
    rawConn, err := tcpConn.SyscallConn()
    if err != nil {
        return err
    }
    
    var sockErr error
    err = rawConn.Control(func(fd uintptr) {
        l := syscall.Linger{
            Onoff:  1,
            Linger: int32(linger),
        }
        sockErr = syscall.SetsockoptLinger(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_LINGER, &l)
    })
    
    if err != nil {
        return err
    }
    
    return sockErr
}

func main() {
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Listen failed: %v\n", err)
        return
    }
    defer listener.Close()
    
    fmt.Println("Linger configuration server on :8080")
    
    conn, err := listener.Accept()
    if err != nil {
        fmt.Printf("Accept failed: %v\n", err)
        return
    }
    
    err = setLinger(conn, 5)
    if err != nil {
        fmt.Printf("Set linger failed: %v\n", err)
    } else {
        fmt.Println("Linger set to 5 seconds")
    }
    
    conn.Write([]byte("Message with linger\n"))
    
    time.Sleep(1 * time.Second)
    
    conn.Close()
    fmt.Println("Connection closed (linger in effect)")
}
```

Linger timeout controls how long `Close()` blocks waiting for unsent data  
transmission. Setting linger to 0 causes immediate RST, while positive values  
wait for graceful closure.  

## Bandwidth Throttling

Throttling limits transmission rate to prevent network congestion or quota  
violations.  

```go
package main

import (
    "fmt"
    "io"
    "net"
    "time"
)

type ThrottledConn struct {
    conn      net.Conn
    rateLimit int
    bucket    chan struct{}
}

func NewThrottledConn(conn net.Conn, bytesPerSec int) *ThrottledConn {
    tc := &ThrottledConn{
        conn:      conn,
        rateLimit: bytesPerSec,
        bucket:    make(chan struct{}, bytesPerSec),
    }
    
    go tc.fillBucket()
    
    return tc
}

func (tc *ThrottledConn) fillBucket() {
    ticker := time.NewTicker(time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        for i := 0; i < tc.rateLimit; i++ {
            select {
            case tc.bucket <- struct{}{}:
            default:
            }
        }
    }
}

func (tc *ThrottledConn) Write(data []byte) (int, error) {
    written := 0
    
    for written < len(data) {
        <-tc.bucket
        
        n, err := tc.conn.Write(data[written : written+1])
        if err != nil {
            return written, err
        }
        
        written += n
    }
    
    return written, nil
}

func (tc *ThrottledConn) Read(data []byte) (int, error) {
    return tc.conn.Read(data)
}

func (tc *ThrottledConn) Close() error {
    return tc.conn.Close()
}

func main() {
    go startThrottleServer()
    
    time.Sleep(100 * time.Millisecond)
    
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Dial failed: %v\n", err)
        return
    }
    
    throttled := NewThrottledConn(conn, 1000)
    defer throttled.Close()
    
    data := make([]byte, 5000)
    
    start := time.Now()
    throttled.Write(data)
    duration := time.Since(start)
    
    fmt.Printf("Sent %d bytes in %v (throttled to 1000 B/s)\n", 
        len(data), duration)
}

func startThrottleServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    io.Copy(io.Discard, conn)
}
```

Token bucket throttling limits transmission rate by requiring tokens for each  
byte sent. Tokens are refilled at a constant rate, enforcing bandwidth limits  
smoothly.  

## Concurrent Request Limiting

Limiting concurrent requests prevents resource exhaustion under high load.  

```go
package main

import (
    "fmt"
    "io"
    "net"
    "sync"
    "time"
)

type ConcurrencyLimiter struct {
    semaphore chan struct{}
    active    int
    mu        sync.Mutex
}

func NewConcurrencyLimiter(max int) *ConcurrencyLimiter {
    return &ConcurrencyLimiter{
        semaphore: make(chan struct{}, max),
    }
}

func (cl *ConcurrencyLimiter) Acquire() {
    cl.semaphore <- struct{}{}
    
    cl.mu.Lock()
    cl.active++
    cl.mu.Unlock()
}

func (cl *ConcurrencyLimiter) Release() {
    <-cl.semaphore
    
    cl.mu.Lock()
    cl.active--
    cl.mu.Unlock()
}

func (cl *ConcurrencyLimiter) Active() int {
    cl.mu.Lock()
    defer cl.mu.Unlock()
    return cl.active
}

func main() {
    limiter := NewConcurrencyLimiter(3)
    
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Listen failed: %v\n", err)
        return
    }
    defer listener.Close()
    
    fmt.Println("Concurrency-limited server on :8080 (max 3)")
    
    go func() {
        ticker := time.NewTicker(1 * time.Second)
        defer ticker.Stop()
        
        for range ticker.C {
            fmt.Printf("Active connections: %d\n", limiter.Active())
        }
    }()
    
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        
        limiter.Acquire()
        
        go func(c net.Conn) {
            defer limiter.Release()
            defer c.Close()
            
            time.Sleep(5 * time.Second)
            
            c.Write([]byte("Processed\n"))
        }(conn)
    }
}
```

Semaphore-based limiting controls maximum concurrent connections. Requests  
block when the limit is reached, preventing server overload while maintaining  
fairness.  

## Message Deduplication

Deduplication prevents processing duplicate messages in unreliable networks.  

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "sync"
    "time"
)

type Deduplicator struct {
    seen   map[string]time.Time
    ttl    time.Duration
    mu     sync.RWMutex
}

func NewDeduplicator(ttl time.Duration) *Deduplicator {
    d := &Deduplicator{
        seen: make(map[string]time.Time),
        ttl:  ttl,
    }
    
    go d.cleanup()
    
    return d
}

func (d *Deduplicator) IsDuplicate(data []byte) bool {
    hash := d.hash(data)
    
    d.mu.RLock()
    _, exists := d.seen[hash]
    d.mu.RUnlock()
    
    if exists {
        return true
    }
    
    d.mu.Lock()
    d.seen[hash] = time.Now()
    d.mu.Unlock()
    
    return false
}

func (d *Deduplicator) hash(data []byte) string {
    h := sha256.Sum256(data)
    return hex.EncodeToString(h[:])
}

func (d *Deduplicator) cleanup() {
    ticker := time.NewTicker(d.ttl / 2)
    defer ticker.Stop()
    
    for range ticker.C {
        d.mu.Lock()
        
        cutoff := time.Now().Add(-d.ttl)
        for hash, timestamp := range d.seen {
            if timestamp.Before(cutoff) {
                delete(d.seen, hash)
            }
        }
        
        d.mu.Unlock()
    }
}

func main() {
    dedup := NewDeduplicator(10 * time.Second)
    
    messages := [][]byte{
        []byte("message 1"),
        []byte("message 2"),
        []byte("message 1"),
        []byte("message 3"),
        []byte("message 2"),
    }
    
    for i, msg := range messages {
        isDup := dedup.IsDuplicate(msg)
        
        if isDup {
            fmt.Printf("Message %d: DUPLICATE - %s\n", i+1, string(msg))
        } else {
            fmt.Printf("Message %d: NEW - %s\n", i+1, string(msg))
        }
    }
}
```

Hash-based deduplication tracks message fingerprints with time-based expiry.  
Duplicate messages are detected and discarded, preventing redundant processing  
in idempotent systems.  

## Connection State Machine

State machines model complex connection lifecycles with explicit state  
transitions.  

```go
package main

import (
    "fmt"
    "net"
)

type State int

const (
    StateDisconnected State = iota
    StateConnecting
    StateAuthenticating
    StateReady
    StateClosing
)

type StatefulConnection struct {
    conn  net.Conn
    state State
}

func NewStatefulConnection(conn net.Conn) *StatefulConnection {
    return &StatefulConnection{
        conn:  conn,
        state: StateConnecting,
    }
}

func (sc *StatefulConnection) Authenticate(password string) error {
    if sc.state != StateConnecting {
        return fmt.Errorf("invalid state for auth: %v", sc.state)
    }
    
    _, err := sc.conn.Write([]byte("AUTH:" + password + "\n"))
    if err != nil {
        return err
    }
    
    sc.state = StateAuthenticating
    
    buffer := make([]byte, 32)
    n, err := sc.conn.Read(buffer)
    if err != nil {
        return err
    }
    
    if string(buffer[:n]) == "OK\n" {
        sc.state = StateReady
        return nil
    }
    
    return fmt.Errorf("authentication failed")
}

func (sc *StatefulConnection) Send(data string) error {
    if sc.state != StateReady {
        return fmt.Errorf("not ready: state %v", sc.state)
    }
    
    _, err := sc.conn.Write([]byte(data + "\n"))
    return err
}

func (sc *StatefulConnection) Close() error {
    sc.state = StateClosing
    err := sc.conn.Close()
    sc.state = StateDisconnected
    return err
}

func main() {
    fmt.Println("Connection state machine example")
    
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        buffer := make([]byte, 1024)
        n, _ := conn.Read(buffer)
        
        if string(buffer[:n]) == "AUTH:secret\n" {
            conn.Write([]byte("OK\n"))
        }
        
        conn.Read(buffer)
    }()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    sc := NewStatefulConnection(conn)
    defer sc.Close()
    
    err := sc.Authenticate("secret")
    if err != nil {
        fmt.Printf("Auth failed: %v\n", err)
        return
    }
    
    fmt.Println("Authenticated successfully")
    
    sc.Send("Hello")
}
```

State machines enforce valid state transitions and prevent invalid operations.  
This improves code correctness for protocols with complex connection  
lifecycles.  

## Pipelining Requests

Pipelining sends multiple requests without waiting for responses, improving  
throughput.  

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "sync"
)

type PipelinedClient struct {
    conn     net.Conn
    pending  []string
    mu       sync.Mutex
}

func NewPipelinedClient(conn net.Conn) *PipelinedClient {
    return &PipelinedClient{
        conn:    conn,
        pending: make([]string, 0),
    }
}

func (pc *PipelinedClient) Send(request string) error {
    pc.mu.Lock()
    defer pc.mu.Unlock()
    
    _, err := pc.conn.Write([]byte(request + "\n"))
    if err != nil {
        return err
    }
    
    pc.pending = append(pc.pending, request)
    
    return nil
}

func (pc *PipelinedClient) Receive() (string, error) {
    reader := bufio.NewReader(pc.conn)
    response, err := reader.ReadString('\n')
    if err != nil {
        return "", err
    }
    
    pc.mu.Lock()
    if len(pc.pending) > 0 {
        pc.pending = pc.pending[1:]
    }
    pc.mu.Unlock()
    
    return response, nil
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        scanner := bufio.NewScanner(conn)
        for scanner.Scan() {
            response := fmt.Sprintf("ACK:%s\n", scanner.Text())
            conn.Write([]byte(response))
        }
    }()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    defer conn.Close()
    
    client := NewPipelinedClient(conn)
    
    requests := []string{"req1", "req2", "req3"}
    
    for _, req := range requests {
        client.Send(req)
    }
    
    for range requests {
        resp, _ := client.Receive()
        fmt.Printf("Response: %s", resp)
    }
}
```

Pipelining sends requests in batches before receiving responses, reducing  
round-trip latency. This is effective for protocols where multiple independent  
requests can be issued together.  

## Adaptive Buffer Sizing

Dynamic buffer sizing optimizes memory usage based on actual data patterns.  

```go
package main

import (
    "fmt"
    "io"
    "net"
)

type AdaptiveBuffer struct {
    conn       net.Conn
    buffer     []byte
    minSize    int
    maxSize    int
    growFactor float64
}

func NewAdaptiveBuffer(conn net.Conn) *AdaptiveBuffer {
    return &AdaptiveBuffer{
        conn:       conn,
        buffer:     make([]byte, 1024),
        minSize:    1024,
        maxSize:    65536,
        growFactor: 1.5,
    }
}

func (ab *AdaptiveBuffer) Read() ([]byte, error) {
    n, err := ab.conn.Read(ab.buffer)
    if err != nil {
        return nil, err
    }
    
    if n == len(ab.buffer) && len(ab.buffer) < ab.maxSize {
        newSize := int(float64(len(ab.buffer)) * ab.growFactor)
        if newSize > ab.maxSize {
            newSize = ab.maxSize
        }
        
        ab.buffer = make([]byte, newSize)
        fmt.Printf("Buffer grown to %d bytes\n", newSize)
    } else if n < len(ab.buffer)/4 && len(ab.buffer) > ab.minSize {
        newSize := len(ab.buffer) / 2
        if newSize < ab.minSize {
            newSize = ab.minSize
        }
        
        ab.buffer = make([]byte, newSize)
        fmt.Printf("Buffer shrunk to %d bytes\n", newSize)
    }
    
    return ab.buffer[:n], nil
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        sizes := []int{500, 2000, 8000, 200, 100}
        
        for _, size := range sizes {
            data := make([]byte, size)
            conn.Write(data)
        }
    }()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    defer conn.Close()
    
    adaptive := NewAdaptiveBuffer(conn)
    
    for {
        data, err := adaptive.Read()
        if err != nil {
            if err != io.EOF {
                fmt.Printf("Read error: %v\n", err)
            }
            break
        }
        
        fmt.Printf("Read %d bytes\n", len(data))
    }
}
```

Adaptive buffers grow when consistently full and shrink when underutilized.  
This balances memory efficiency with throughput for variable workloads.  

## Connection Warm-up

Warm-up prepares connections for production traffic to avoid cold-start  
latency.  

```go
package main

import (
    "fmt"
    "net"
    "time"
)

type WarmupPool struct {
    address string
    size    int
    ready   chan net.Conn
}

func NewWarmupPool(address string, size int) *WarmupPool {
    wp := &WarmupPool{
        address: address,
        size:    size,
        ready:   make(chan net.Conn, size),
    }
    
    go wp.warmup()
    
    return wp
}

func (wp *WarmupPool) warmup() {
    for i := 0; i < wp.size; i++ {
        conn, err := net.Dial("tcp", wp.address)
        if err != nil {
            fmt.Printf("Warmup connection %d failed: %v\n", i+1, err)
            continue
        }
        
        conn.Write([]byte("WARMUP\n"))
        
        buffer := make([]byte, 32)
        conn.Read(buffer)
        
        wp.ready <- conn
        
        fmt.Printf("Connection %d warmed up\n", i+1)
    }
}

func (wp *WarmupPool) Get() net.Conn {
    return <-wp.ready
}

func (wp *WarmupPool) Put(conn net.Conn) {
    select {
    case wp.ready <- conn:
    default:
        conn.Close()
    }
}

func main() {
    go startWarmupServer()
    
    time.Sleep(100 * time.Millisecond)
    
    pool := NewWarmupPool("localhost:8080", 3)
    
    time.Sleep(2 * time.Second)
    
    conn := pool.Get()
    fmt.Println("Got warmed connection")
    
    conn.Write([]byte("Real request\n"))
    
    pool.Put(conn)
}

func startWarmupServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    for {
        conn, _ := listener.Accept()
        
        go func(c net.Conn) {
            defer c.Close()
            
            buffer := make([]byte, 1024)
            for {
                n, err := c.Read(buffer)
                if err != nil {
                    return
                }
                
                c.Write([]byte("OK\n"))
                
                if string(buffer[:n]) != "WARMUP\n" {
                    fmt.Printf("Processing: %s", string(buffer[:n]))
                }
            }
        }(conn)
    }
}
```

Connection warm-up establishes and validates connections proactively,  
eliminating connection setup latency for first requests. This improves  
response time for latency-sensitive applications.  

## Scatter-Gather Message Assembly

Scatter-gather assembles fragmented messages from multiple network reads.  

```go
package main

import (
    "encoding/binary"
    "fmt"
    "io"
    "net"
)

type FragmentedMessage struct {
    totalSize int
    fragments [][]byte
    received  int
}

func NewFragmentedMessage(totalSize int) *FragmentedMessage {
    return &FragmentedMessage{
        totalSize: totalSize,
        fragments: make([][]byte, 0),
    }
}

func (fm *FragmentedMessage) AddFragment(data []byte) {
    fragment := make([]byte, len(data))
    copy(fragment, data)
    fm.fragments = append(fm.fragments, fragment)
    fm.received += len(data)
}

func (fm *FragmentedMessage) IsComplete() bool {
    return fm.received >= fm.totalSize
}

func (fm *FragmentedMessage) Assemble() []byte {
    result := make([]byte, 0, fm.totalSize)
    for _, fragment := range fm.fragments {
        result = append(result, fragment...)
    }
    return result[:fm.totalSize]
}

func receiveFragmented(conn net.Conn) ([]byte, error) {
    var totalSize uint32
    err := binary.Read(conn, binary.BigEndian, &totalSize)
    if err != nil {
        return nil, err
    }
    
    msg := NewFragmentedMessage(int(totalSize))
    
    buffer := make([]byte, 1024)
    for !msg.IsComplete() {
        n, err := conn.Read(buffer)
        if err != nil {
            if err == io.EOF && msg.IsComplete() {
                break
            }
            return nil, err
        }
        
        msg.AddFragment(buffer[:n])
        fmt.Printf("Received fragment: %d bytes (%d/%d total)\n", 
            n, msg.received, msg.totalSize)
    }
    
    return msg.Assemble(), nil
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        defer conn.Close()
        
        data, _ := receiveFragmented(conn)
        fmt.Printf("Assembled message: %d bytes\n", len(data))
    }()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    defer conn.Close()
    
    message := make([]byte, 5000)
    totalSize := uint32(len(message))
    
    binary.Write(conn, binary.BigEndian, totalSize)
    
    chunkSize := 500
    for i := 0; i < len(message); i += chunkSize {
        end := i + chunkSize
        if end > len(message) {
            end = len(message)
        }
        
        conn.Write(message[i:end])
    }
}
```

Scatter-gather handles message fragmentation by collecting fragments until  
complete. This is essential for protocols where message boundaries don't  
align with packet boundaries.  

## Connection Affinity

Connection affinity routes related requests to the same backend for session  
consistency.  

```go
package main

import (
    "fmt"
    "hash/fnv"
    "net"
    "sync"
)

type AffinityBalancer struct {
    backends []string
    sessions map[string]string
    mu       sync.RWMutex
}

func NewAffinityBalancer(backends []string) *AffinityBalancer {
    return &AffinityBalancer{
        backends: backends,
        sessions: make(map[string]string),
    }
}

func (ab *AffinityBalancer) GetBackend(sessionID string) string {
    ab.mu.RLock()
    backend, exists := ab.sessions[sessionID]
    ab.mu.RUnlock()
    
    if exists {
        return backend
    }
    
    backend = ab.hashBackend(sessionID)
    
    ab.mu.Lock()
    ab.sessions[sessionID] = backend
    ab.mu.Unlock()
    
    return backend
}

func (ab *AffinityBalancer) hashBackend(key string) string {
    h := fnv.New32a()
    h.Write([]byte(key))
    index := h.Sum32() % uint32(len(ab.backends))
    return ab.backends[index]
}

func main() {
    backends := []string{
        "backend1:9001",
        "backend2:9002",
        "backend3:9003",
    }
    
    balancer := NewAffinityBalancer(backends)
    
    sessions := []string{"user1", "user2", "user1", "user3", "user2"}
    
    for _, session := range sessions {
        backend := balancer.GetBackend(session)
        fmt.Printf("Session %s -> %s\n", session, backend)
    }
}
```

Affinity routing uses session IDs to consistently route requests to the same  
backend. This maintains session state without requiring shared storage or  
sticky load balancer configuration.  

## Protocol Version Negotiation

Version negotiation allows clients and servers with different capabilities  
to communicate.  

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "strconv"
    "strings"
)

type VersionNegotiator struct {
    supportedVersions []int
    negotiatedVersion int
}

func NewVersionNegotiator(versions []int) *VersionNegotiator {
    return &VersionNegotiator{
        supportedVersions: versions,
    }
}

func (vn *VersionNegotiator) ClientNegotiate(conn net.Conn) error {
    versions := make([]string, len(vn.supportedVersions))
    for i, v := range vn.supportedVersions {
        versions[i] = strconv.Itoa(v)
    }
    
    message := fmt.Sprintf("VERSIONS:%s\n", strings.Join(versions, ","))
    _, err := conn.Write([]byte(message))
    if err != nil {
        return err
    }
    
    reader := bufio.NewReader(conn)
    response, err := reader.ReadString('\n')
    if err != nil {
        return err
    }
    
    parts := strings.Split(strings.TrimSpace(response), ":")
    if len(parts) != 2 || parts[0] != "USE" {
        return fmt.Errorf("invalid response: %s", response)
    }
    
    version, err := strconv.Atoi(parts[1])
    if err != nil {
        return err
    }
    
    vn.negotiatedVersion = version
    fmt.Printf("Negotiated version: %d\n", version)
    
    return nil
}

func (vn *VersionNegotiator) ServerNegotiate(conn net.Conn) error {
    reader := bufio.NewReader(conn)
    message, err := reader.ReadString('\n')
    if err != nil {
        return err
    }
    
    parts := strings.Split(strings.TrimSpace(message), ":")
    if len(parts) != 2 || parts[0] != "VERSIONS" {
        return fmt.Errorf("invalid request: %s", message)
    }
    
    clientVersions := strings.Split(parts[1], ",")
    
    var selectedVersion int
    for _, cv := range clientVersions {
        v, err := strconv.Atoi(cv)
        if err != nil {
            continue
        }
        
        for _, sv := range vn.supportedVersions {
            if v == sv && v > selectedVersion {
                selectedVersion = v
            }
        }
    }
    
    if selectedVersion == 0 {
        return fmt.Errorf("no common version")
    }
    
    response := fmt.Sprintf("USE:%d\n", selectedVersion)
    _, err = conn.Write([]byte(response))
    if err != nil {
        return err
    }
    
    vn.negotiatedVersion = selectedVersion
    fmt.Printf("Server selected version: %d\n", selectedVersion)
    
    return nil
}

func main() {
    go startVersionServer()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    defer conn.Close()
    
    client := NewVersionNegotiator([]int{1, 2, 3})
    client.ClientNegotiate(conn)
}

func startVersionServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    server := NewVersionNegotiator([]int{2, 3, 4})
    server.ServerNegotiate(conn)
}
```

Version negotiation exchanges supported versions and selects the highest  
common version. This enables backward compatibility while allowing newer  
features for capable clients.  

## Flow Control Windows

Flow control prevents fast senders from overwhelming slow receivers using  
window-based mechanisms.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type FlowController struct {
    windowSize int
    available  int
    mu         sync.Mutex
    cond       *sync.Cond
}

func NewFlowController(windowSize int) *FlowController {
    fc := &FlowController{
        windowSize: windowSize,
        available:  windowSize,
    }
    fc.cond = sync.NewCond(&fc.mu)
    return fc
}

func (fc *FlowController) Acquire(size int) {
    fc.mu.Lock()
    defer fc.mu.Unlock()
    
    for fc.available < size {
        fc.cond.Wait()
    }
    
    fc.available -= size
}

func (fc *FlowController) Release(size int) {
    fc.mu.Lock()
    defer fc.mu.Unlock()
    
    fc.available += size
    if fc.available > fc.windowSize {
        fc.available = fc.windowSize
    }
    
    fc.cond.Broadcast()
}

func (fc *FlowController) Available() int {
    fc.mu.Lock()
    defer fc.mu.Unlock()
    return fc.available
}

func main() {
    fc := NewFlowController(1000)
    
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for j := 0; j < 3; j++ {
                size := 400
                
                fmt.Printf("Sender %d: requesting %d bytes\n", id, size)
                fc.Acquire(size)
                fmt.Printf("Sender %d: acquired %d bytes (available: %d)\n", 
                    id, size, fc.Available())
                
                time.Sleep(500 * time.Millisecond)
                
                fc.Release(size)
                fmt.Printf("Sender %d: released %d bytes (available: %d)\n", 
                    id, size, fc.Available())
            }
        }(i)
    }
    
    wg.Wait()
}
```

Flow control windows limit outstanding data, blocking senders when the window  
is exhausted. Receivers release window space as they process data, maintaining  
balanced flow.  

## Connection Preloading

Preloading establishes connections in advance based on predicted demand.  

```go
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

type ConnectionPreloader struct {
    address   string
    targetSize int
    pool      []net.Conn
    mu        sync.Mutex
}

func NewConnectionPreloader(address string, targetSize int) *ConnectionPreloader {
    cp := &ConnectionPreloader{
        address:    address,
        targetSize: targetSize,
        pool:       make([]net.Conn, 0, targetSize),
    }
    
    go cp.maintain()
    
    return cp
}

func (cp *ConnectionPreloader) maintain() {
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        cp.mu.Lock()
        current := len(cp.pool)
        needed := cp.targetSize - current
        cp.mu.Unlock()
        
        if needed > 0 {
            fmt.Printf("Preloading %d connections\n", needed)
            
            for i := 0; i < needed; i++ {
                conn, err := net.Dial("tcp", cp.address)
                if err != nil {
                    fmt.Printf("Preload failed: %v\n", err)
                    continue
                }
                
                cp.mu.Lock()
                cp.pool = append(cp.pool, conn)
                cp.mu.Unlock()
            }
        }
    }
}

func (cp *ConnectionPreloader) Get() net.Conn {
    cp.mu.Lock()
    defer cp.mu.Unlock()
    
    if len(cp.pool) > 0 {
        conn := cp.pool[0]
        cp.pool = cp.pool[1:]
        return conn
    }
    
    conn, _ := net.Dial("tcp", cp.address)
    return conn
}

func main() {
    go startPreloadServer()
    
    time.Sleep(100 * time.Millisecond)
    
    preloader := NewConnectionPreloader("localhost:8080", 5)
    
    time.Sleep(2 * time.Second)
    
    conn := preloader.Get()
    fmt.Println("Got preloaded connection")
    
    conn.Write([]byte("Request\n"))
    conn.Close()
}

func startPreloadServer() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    for {
        conn, _ := listener.Accept()
        go func(c net.Conn) {
            defer c.Close()
            
            buffer := make([]byte, 1024)
            c.Read(buffer)
        }(conn)
    }
}
```

Connection preloading maintains a target pool size by establishing connections  
proactively. This eliminates connection setup latency during traffic spikes  
by keeping connections ready.  

## Zero-Downtime Connection Migration

Connection migration transfers active connections to new servers without  
interruption.  

```go
package main

import (
    "fmt"
    "io"
    "net"
    "sync"
)

type MigratableConnection struct {
    mu       sync.Mutex
    conn     net.Conn
    migrating bool
}

func NewMigratableConnection(conn net.Conn) *MigratableConnection {
    return &MigratableConnection{
        conn: conn,
    }
}

func (mc *MigratableConnection) Migrate(newAddr string) error {
    mc.mu.Lock()
    if mc.migrating {
        mc.mu.Unlock()
        return fmt.Errorf("already migrating")
    }
    mc.migrating = true
    mc.mu.Unlock()
    
    newConn, err := net.Dial("tcp", newAddr)
    if err != nil {
        mc.mu.Lock()
        mc.migrating = false
        mc.mu.Unlock()
        return err
    }
    
    mc.mu.Lock()
    oldConn := mc.conn
    mc.conn = newConn
    mc.migrating = false
    mc.mu.Unlock()
    
    oldConn.Close()
    
    fmt.Printf("Migrated to %s\n", newAddr)
    return nil
}

func (mc *MigratableConnection) Write(data []byte) (int, error) {
    mc.mu.Lock()
    conn := mc.conn
    mc.mu.Unlock()
    
    return conn.Write(data)
}

func (mc *MigratableConnection) Read(data []byte) (int, error) {
    mc.mu.Lock()
    conn := mc.conn
    mc.mu.Unlock()
    
    return conn.Read(data)
}

func main() {
    go startMigrationServer(":8080")
    go startMigrationServer(":8081")
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    mc := NewMigratableConnection(conn)
    
    mc.Write([]byte("Before migration\n"))
    
    mc.Migrate("localhost:8081")
    
    mc.Write([]byte("After migration\n"))
}

func startMigrationServer(addr string) {
    listener, _ := net.Listen("tcp", addr)
    defer listener.Close()
    
    fmt.Printf("Migration server on %s\n", addr)
    
    for {
        conn, _ := listener.Accept()
        
        go func(c net.Conn) {
            defer c.Close()
            
            io.Copy(io.Discard, c)
        }(conn)
    }
}
```

Connection migration establishes new connections to replacement servers while  
maintaining application-level continuity. This enables zero-downtime server  
upgrades and maintenance.  

## Idle Connection Detection

Detecting and closing idle connections prevents resource waste.  

```go
package main

import (
    "fmt"
    "net"
    "sync"
    "time"
)

type IdleTracker struct {
    timeout     time.Duration
    connections map[net.Conn]time.Time
    mu          sync.RWMutex
}

func NewIdleTracker(timeout time.Duration) *IdleTracker {
    it := &IdleTracker{
        timeout:     timeout,
        connections: make(map[net.Conn]time.Time),
    }
    
    go it.cleanup()
    
    return it
}

func (it *IdleTracker) Touch(conn net.Conn) {
    it.mu.Lock()
    defer it.mu.Unlock()
    it.connections[conn] = time.Now()
}

func (it *IdleTracker) Remove(conn net.Conn) {
    it.mu.Lock()
    defer it.mu.Unlock()
    delete(it.connections, conn)
}

func (it *IdleTracker) cleanup() {
    ticker := time.NewTicker(it.timeout / 2)
    defer ticker.Stop()
    
    for range ticker.C {
        it.mu.Lock()
        
        cutoff := time.Now().Add(-it.timeout)
        for conn, lastActive := range it.connections {
            if lastActive.Before(cutoff) {
                fmt.Printf("Closing idle connection: %s\n", conn.RemoteAddr())
                conn.Close()
                delete(it.connections, conn)
            }
        }
        
        it.mu.Unlock()
    }
}

func main() {
    tracker := NewIdleTracker(3 * time.Second)
    
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        for {
            conn, _ := listener.Accept()
            tracker.Touch(conn)
            
            go func(c net.Conn) {
                defer tracker.Remove(c)
                defer c.Close()
                
                buffer := make([]byte, 1024)
                for {
                    n, err := c.Read(buffer)
                    if err != nil {
                        return
                    }
                    
                    tracker.Touch(c)
                    c.Write(buffer[:n])
                }
            }(conn)
        }
    }()
    
    fmt.Println("Idle tracker server running")
    select {}
}
```

Idle tracking monitors last activity timestamps and automatically closes  
connections exceeding the timeout. This frees resources consumed by inactive  
clients.  

## Request Coalescing

Coalescing merges duplicate concurrent requests to reduce backend load.  

```go
package main

import (
    "fmt"
    "sync"
)

type CoalescingCache struct {
    pending map[string]*pendingRequest
    mu      sync.Mutex
}

type pendingRequest struct {
    result chan interface{}
    err    chan error
}

func NewCoalescingCache() *CoalescingCache {
    return &CoalescingCache{
        pending: make(map[string]*pendingRequest),
    }
}

func (cc *CoalescingCache) Do(key string, fn func() (interface{}, error)) (interface{}, error) {
    cc.mu.Lock()
    
    if req, exists := cc.pending[key]; exists {
        cc.mu.Unlock()
        
        select {
        case result := <-req.result:
            return result, nil
        case err := <-req.err:
            return nil, err
        }
    }
    
    req := &pendingRequest{
        result: make(chan interface{}, 1),
        err:    make(chan error, 1),
    }
    cc.pending[key] = req
    cc.mu.Unlock()
    
    result, err := fn()
    
    cc.mu.Lock()
    delete(cc.pending, key)
    cc.mu.Unlock()
    
    if err != nil {
        req.err <- err
    } else {
        req.result <- result
    }
    
    return result, err
}

func main() {
    cache := NewCoalescingCache()
    
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            result, err := cache.Do("key1", func() (interface{}, error) {
                fmt.Printf("Goroutine %d: executing expensive operation\n", id)
                return "result", nil
            })
            
            fmt.Printf("Goroutine %d: got %v, %v\n", id, result, err)
        }(i)
    }
    
    wg.Wait()
}
```

Request coalescing groups identical concurrent requests, executing the  
operation once and sharing the result. This reduces load for expensive  
operations with duplicate requests.  

## Backoff Retry with Jitter

Adding jitter to retry backoff prevents thundering herd problems.  

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

type RetryWithJitter struct {
    maxRetries int
    baseDelay  time.Duration
    maxDelay   time.Duration
}

func NewRetryWithJitter(maxRetries int, baseDelay, maxDelay time.Duration) *RetryWithJitter {
    return &RetryWithJitter{
        maxRetries: maxRetries,
        baseDelay:  baseDelay,
        maxDelay:   maxDelay,
    }
}

func (r *RetryWithJitter) Do(fn func() error) error {
    var lastErr error
    
    for attempt := 0; attempt < r.maxRetries; attempt++ {
        err := fn()
        if err == nil {
            return nil
        }
        
        lastErr = err
        
        if attempt < r.maxRetries-1 {
            delay := r.calculateDelay(attempt)
            fmt.Printf("Attempt %d failed, retrying in %v\n", attempt+1, delay)
            time.Sleep(delay)
        }
    }
    
    return fmt.Errorf("failed after %d attempts: %w", r.maxRetries, lastErr)
}

func (r *RetryWithJitter) calculateDelay(attempt int) time.Duration {
    base := r.baseDelay * time.Duration(1<<uint(attempt))
    if base > r.maxDelay {
        base = r.maxDelay
    }
    
    jitter := time.Duration(rand.Int63n(int64(base / 2)))
    
    return base + jitter
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    retry := NewRetryWithJitter(5, 100*time.Millisecond, 2*time.Second)
    
    attempts := 0
    err := retry.Do(func() error {
        attempts++
        if attempts < 3 {
            return fmt.Errorf("simulated failure %d", attempts)
        }
        return nil
    })
    
    if err != nil {
        fmt.Printf("Failed: %v\n", err)
    } else {
        fmt.Println("Success!")
    }
}
```

Jittered backoff adds randomness to retry delays, preventing synchronized  
retry storms when many clients fail simultaneously. This improves system  
stability during outages.  

## Connection Reuse Detection

Detecting connection reuse prevents protocol violations and security issues.  

```go
package main

import (
    "fmt"
    "net"
    "sync"
)

type ReuseDetector struct {
    used map[net.Conn]bool
    mu   sync.RWMutex
}

func NewReuseDetector() *ReuseDetector {
    return &ReuseDetector{
        used: make(map[net.Conn]bool),
    }
}

func (rd *ReuseDetector) MarkUsed(conn net.Conn) bool {
    rd.mu.Lock()
    defer rd.mu.Unlock()
    
    if rd.used[conn] {
        return false
    }
    
    rd.used[conn] = true
    return true
}

func (rd *ReuseDetector) Reset(conn net.Conn) {
    rd.mu.Lock()
    defer rd.mu.Unlock()
    
    delete(rd.used, conn)
}

func main() {
    detector := NewReuseDetector()
    
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    for {
        conn, _ := listener.Accept()
        
        if !detector.MarkUsed(conn) {
            fmt.Printf("Warning: Connection reuse detected for %s\n", 
                conn.RemoteAddr())
            conn.Close()
            continue
        }
        
        go func(c net.Conn) {
            defer detector.Reset(c)
            defer c.Close()
            
            fmt.Printf("Processing connection: %s\n", c.RemoteAddr())
        }(conn)
    }
}
```

Reuse detection tracks active connections and prevents processing the same  
connection multiple times, which could indicate attacks or protocol errors.  

## Connection Backlog Monitoring

Monitoring accept backlog helps detect connection queue buildup.  

```go
package main

import (
    "fmt"
    "net"
    "sync/atomic"
    "time"
)

type BacklogMonitor struct {
    pendingAccepts int64
    completedAccepts int64
}

func NewBacklogMonitor() *BacklogMonitor {
    bm := &BacklogMonitor{}
    go bm.report()
    return bm
}

func (bm *BacklogMonitor) BeforeAccept() {
    atomic.AddInt64(&bm.pendingAccepts, 1)
}

func (bm *BacklogMonitor) AfterAccept() {
    atomic.AddInt64(&bm.pendingAccepts, -1)
    atomic.AddInt64(&bm.completedAccepts, 1)
}

func (bm *BacklogMonitor) report() {
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        pending := atomic.LoadInt64(&bm.pendingAccepts)
        completed := atomic.LoadInt64(&bm.completedAccepts)
        
        fmt.Printf("Backlog: pending=%d, completed=%d\n", pending, completed)
    }
}

func main() {
    monitor := NewBacklogMonitor()
    
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    fmt.Println("Backlog monitor server on :8080")
    
    for {
        monitor.BeforeAccept()
        conn, _ := listener.Accept()
        monitor.AfterAccept()
        
        go func(c net.Conn) {
            defer c.Close()
            time.Sleep(2 * time.Second)
        }(conn)
    }
}
```

Backlog monitoring tracks pending accept operations to detect connection  
buildup, which indicates insufficient worker capacity or slow processing.  

## Dynamic Protocol Selection

Protocol detection enables supporting multiple protocols on one port.  

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "strings"
)

type ProtocolDetector struct {
    detectors map[string]func([]byte) bool
}

func NewProtocolDetector() *ProtocolDetector {
    pd := &ProtocolDetector{
        detectors: make(map[string]func([]byte) bool),
    }
    
    pd.detectors["HTTP"] = func(data []byte) bool {
        s := string(data)
        return strings.HasPrefix(s, "GET ") || 
               strings.HasPrefix(s, "POST ") ||
               strings.HasPrefix(s, "PUT ")
    }
    
    pd.detectors["CUSTOM"] = func(data []byte) bool {
        return len(data) > 0 && data[0] == 0xFF
    }
    
    return pd
}

func (pd *ProtocolDetector) Detect(conn net.Conn) string {
    reader := bufio.NewReader(conn)
    
    peek, err := reader.Peek(16)
    if err != nil {
        return "UNKNOWN"
    }
    
    for name, detector := range pd.detectors {
        if detector(peek) {
            return name
        }
    }
    
    return "UNKNOWN"
}

func main() {
    detector := NewProtocolDetector()
    
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    fmt.Println("Protocol detector on :8080")
    
    for {
        conn, _ := listener.Accept()
        
        protocol := detector.Detect(conn)
        fmt.Printf("Detected protocol: %s from %s\n", 
            protocol, conn.RemoteAddr())
        
        conn.Close()
    }
}
```

Protocol detection examines initial bytes to identify the protocol, enabling  
protocol multiplexing on a single port for simplified deployment.  

## Sliding Window Flow Control

Sliding window maintains optimal throughput while preventing overflow.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type SlidingWindow struct {
    size      int
    used      int
    sequence  int
    acked     int
    mu        sync.Mutex
    available *sync.Cond
}

func NewSlidingWindow(size int) *SlidingWindow {
    sw := &SlidingWindow{
        size: size,
    }
    sw.available = sync.NewCond(&sw.mu)
    return sw
}

func (sw *SlidingWindow) Send() int {
    sw.mu.Lock()
    defer sw.mu.Unlock()
    
    for sw.used >= sw.size {
        sw.available.Wait()
    }
    
    sw.sequence++
    sw.used++
    
    return sw.sequence
}

func (sw *SlidingWindow) Acknowledge(seq int) {
    sw.mu.Lock()
    defer sw.mu.Unlock()
    
    if seq > sw.acked {
        sw.acked = seq
        sw.used--
        sw.available.Signal()
    }
}

func main() {
    window := NewSlidingWindow(5)
    
    var wg sync.WaitGroup
    
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            seq := window.Send()
            fmt.Printf("Sender %d: sent sequence %d\n", id, seq)
            
            go func() {
                time.Sleep(500 * time.Millisecond)
                window.Acknowledge(seq)
                fmt.Printf("Acknowledged sequence %d\n", seq)
            }()
        }(i)
        
        time.Sleep(100 * time.Millisecond)
    }
    
    wg.Wait()
}
```

Sliding windows allow multiple outstanding messages up to a window size,  
blocking when full. Acknowledgments slide the window forward, balancing  
throughput and buffering.  

## Connection Health Scoring

Health scoring ranks connections for intelligent routing decisions.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type HealthScore struct {
    latency       time.Duration
    errorRate     float64
    successCount  int
    errorCount    int
    lastUpdate    time.Time
}

type ScoredConnection struct {
    address string
    score   HealthScore
    mu      sync.RWMutex
}

func NewScoredConnection(address string) *ScoredConnection {
    return &ScoredConnection{
        address: address,
        score: HealthScore{
            lastUpdate: time.Now(),
        },
    }
}

func (sc *ScoredConnection) RecordSuccess(latency time.Duration) {
    sc.mu.Lock()
    defer sc.mu.Unlock()
    
    sc.score.successCount++
    sc.score.latency = latency
    sc.score.lastUpdate = time.Now()
    sc.updateErrorRate()
}

func (sc *ScoredConnection) RecordError() {
    sc.mu.Lock()
    defer sc.mu.Unlock()
    
    sc.score.errorCount++
    sc.score.lastUpdate = time.Now()
    sc.updateErrorRate()
}

func (sc *ScoredConnection) updateErrorRate() {
    total := sc.score.successCount + sc.score.errorCount
    if total > 0 {
        sc.score.errorRate = float64(sc.score.errorCount) / float64(total)
    }
}

func (sc *ScoredConnection) GetScore() float64 {
    sc.mu.RLock()
    defer sc.mu.RUnlock()
    
    latencyScore := 1.0 / (1.0 + sc.score.latency.Seconds())
    errorScore := 1.0 - sc.score.errorRate
    
    return (latencyScore + errorScore) / 2.0
}

func main() {
    conn := NewScoredConnection("backend:8080")
    
    conn.RecordSuccess(50 * time.Millisecond)
    conn.RecordSuccess(60 * time.Millisecond)
    conn.RecordError()
    conn.RecordSuccess(55 * time.Millisecond)
    
    score := conn.GetScore()
    fmt.Printf("Connection health score: %.2f\n", score)
}
```

Health scoring combines latency, error rates, and recency to rank connections.  
Load balancers use scores to prefer healthier backends.  

## Adaptive Timeout Calculation

Dynamic timeouts adjust based on observed latency patterns.  

```go
package main

import (
    "fmt"
    "time"
)

type AdaptiveTimeout struct {
    samples    []time.Duration
    maxSamples int
}

func NewAdaptiveTimeout(maxSamples int) *AdaptiveTimeout {
    return &AdaptiveTimeout{
        samples:    make([]time.Duration, 0, maxSamples),
        maxSamples: maxSamples,
    }
}

func (at *AdaptiveTimeout) Record(latency time.Duration) {
    at.samples = append(at.samples, latency)
    
    if len(at.samples) > at.maxSamples {
        at.samples = at.samples[1:]
    }
}

func (at *AdaptiveTimeout) Calculate() time.Duration {
    if len(at.samples) == 0 {
        return 5 * time.Second
    }
    
    var sum time.Duration
    var max time.Duration
    
    for _, sample := range at.samples {
        sum += sample
        if sample > max {
            max = sample
        }
    }
    
    avg := sum / time.Duration(len(at.samples))
    
    timeout := avg*2 + time.Second
    if timeout < max*3/2 {
        timeout = max * 3 / 2
    }
    
    return timeout
}

func main() {
    adaptive := NewAdaptiveTimeout(10)
    
    latencies := []time.Duration{
        100 * time.Millisecond,
        150 * time.Millisecond,
        120 * time.Millisecond,
        200 * time.Millisecond,
        110 * time.Millisecond,
    }
    
    for _, latency := range latencies {
        adaptive.Record(latency)
        timeout := adaptive.Calculate()
        
        fmt.Printf("Latency: %v, Timeout: %v\n", latency, timeout)
    }
}
```

Adaptive timeouts track recent latencies to calculate appropriate timeout  
values. This balances responsiveness with tolerance for varying network  
conditions.  

## Message Priority Queuing

Priority queues ensure high-priority messages are processed first.  

```go
package main

import (
    "container/heap"
    "fmt"
)

type PriorityMessage struct {
    priority int
    data     string
    index    int
}

type MessageQueue []*PriorityMessage

func (mq MessageQueue) Len() int { return len(mq) }

func (mq MessageQueue) Less(i, j int) bool {
    return mq[i].priority > mq[j].priority
}

func (mq MessageQueue) Swap(i, j int) {
    mq[i], mq[j] = mq[j], mq[i]
    mq[i].index = i
    mq[j].index = j
}

func (mq *MessageQueue) Push(x interface{}) {
    n := len(*mq)
    item := x.(*PriorityMessage)
    item.index = n
    *mq = append(*mq, item)
}

func (mq *MessageQueue) Pop() interface{} {
    old := *mq
    n := len(old)
    item := old[n-1]
    old[n-1] = nil
    item.index = -1
    *mq = old[0 : n-1]
    return item
}

func main() {
    mq := make(MessageQueue, 0)
    heap.Init(&mq)
    
    messages := []*PriorityMessage{
        {priority: 1, data: "Low priority"},
        {priority: 5, data: "High priority"},
        {priority: 3, data: "Medium priority"},
        {priority: 5, data: "Another high"},
    }
    
    for _, msg := range messages {
        heap.Push(&mq, msg)
    }
    
    fmt.Println("Processing by priority:")
    for mq.Len() > 0 {
        msg := heap.Pop(&mq).(*PriorityMessage)
        fmt.Printf("Priority %d: %s\n", msg.priority, msg.data)
    }
}
```

Priority queuing ensures critical messages are processed before lower-priority  
ones, improving responsiveness for important operations even under load.  

## Connection Fingerprinting

Fingerprinting identifies connection characteristics for security and  
analytics.  

```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "fmt"
    "net"
    "syscall"
)

type ConnectionFingerprint struct {
    RemoteAddr    string
    LocalAddr     string
    SocketOptions map[string]int
}

func FingerprintConnection(conn net.Conn) (*ConnectionFingerprint, error) {
    fp := &ConnectionFingerprint{
        RemoteAddr:    conn.RemoteAddr().String(),
        LocalAddr:     conn.LocalAddr().String(),
        SocketOptions: make(map[string]int),
    }
    
    tcpConn, ok := conn.(*net.TCPConn)
    if !ok {
        return fp, nil
    }
    
    rawConn, err := tcpConn.SyscallConn()
    if err != nil {
        return fp, err
    }
    
    rawConn.Control(func(fd uintptr) {
        val, err := syscall.GetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_RCVBUF)
        if err == nil {
            fp.SocketOptions["SO_RCVBUF"] = val
        }
        
        val, err = syscall.GetsockoptInt(int(fd), 
            syscall.SOL_SOCKET, syscall.SO_SNDBUF)
        if err == nil {
            fp.SocketOptions["SO_SNDBUF"] = val
        }
    })
    
    return fp, nil
}

func (fp *ConnectionFingerprint) Hash() string {
    data := fmt.Sprintf("%s:%s:%v", fp.RemoteAddr, fp.LocalAddr, 
        fp.SocketOptions)
    
    h := sha256.Sum256([]byte(data))
    return hex.EncodeToString(h[:8])
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    fmt.Println("Connection fingerprinting server on :8080")
    
    conn, _ := listener.Accept()
    defer conn.Close()
    
    fp, err := FingerprintConnection(conn)
    if err != nil {
        fmt.Printf("Fingerprint error: %v\n", err)
        return
    }
    
    fmt.Printf("Remote: %s\n", fp.RemoteAddr)
    fmt.Printf("Local: %s\n", fp.LocalAddr)
    fmt.Printf("Options: %v\n", fp.SocketOptions)
    fmt.Printf("Hash: %s\n", fp.Hash())
}
```

Connection fingerprinting captures unique connection characteristics for  
identification, tracking, and security analysis. Hashes enable efficient  
comparison.  

## Bandwidth Shaping

Traffic shaping enforces rate limits and smooths bursts for QoS.  

```go
package main

import (
    "fmt"
    "time"
)

type TrafficShaper struct {
    rate       int
    burstSize  int
    tokens     int
    lastUpdate time.Time
}

func NewTrafficShaper(rate, burstSize int) *TrafficShaper {
    return &TrafficShaper{
        rate:       rate,
        burstSize:  burstSize,
        tokens:     burstSize,
        lastUpdate: time.Now(),
    }
}

func (ts *TrafficShaper) Allow(size int) bool {
    now := time.Now()
    elapsed := now.Sub(ts.lastUpdate)
    
    tokensToAdd := int(elapsed.Seconds() * float64(ts.rate))
    ts.tokens += tokensToAdd
    
    if ts.tokens > ts.burstSize {
        ts.tokens = ts.burstSize
    }
    
    ts.lastUpdate = now
    
    if ts.tokens >= size {
        ts.tokens -= size
        return true
    }
    
    return false
}

func main() {
    shaper := NewTrafficShaper(1000, 2000)
    
    for i := 0; i < 5; i++ {
        size := 800
        
        if shaper.Allow(size) {
            fmt.Printf("Request %d: allowed (%d bytes)\n", i+1, size)
        } else {
            fmt.Printf("Request %d: rate limited\n", i+1)
        }
        
        time.Sleep(500 * time.Millisecond)
    }
}
```

## Connection Pooling with Priorities

Priority-aware pools serve high-priority requests preferentially.  

```go
package main

import (
    "fmt"
    "net"
)

type PriorityConnection struct {
    conn     net.Conn
    priority int
}

type PriorityConnectionPool struct {
    available []PriorityConnection
}

func (pcp *PriorityConnectionPool) Add(conn net.Conn, priority int) {
    pcp.available = append(pcp.available, PriorityConnection{
        conn:     conn,
        priority: priority,
    })
}

func (pcp *PriorityConnectionPool) GetBest() net.Conn {
    if len(pcp.available) == 0 {
        return nil
    }
    
    best := 0
    for i, pc := range pcp.available {
        if pc.priority > pcp.available[best].priority {
            best = i
        }
    }
    
    conn := pcp.available[best].conn
    pcp.available = append(pcp.available[:best], pcp.available[best+1:]...)
    
    return conn
}

func main() {
    pool := &PriorityConnectionPool{}
    
    fmt.Println("Priority connection pool example")
}
```

## Connection Burst Handling

Burst handling manages sudden traffic spikes without degradation.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type BurstHandler struct {
    normal int
    burst  int
    window time.Duration
    
    mu      sync.Mutex
    count   int
    reset   time.Time
}

func NewBurstHandler(normal, burst int, window time.Duration) *BurstHandler {
    return &BurstHandler{
        normal: normal,
        burst:  burst,
        window: window,
        reset:  time.Now().Add(window),
    }
}

func (bh *BurstHandler) Allow() bool {
    bh.mu.Lock()
    defer bh.mu.Unlock()
    
    now := time.Now()
    if now.After(bh.reset) {
        bh.count = 0
        bh.reset = now.Add(bh.window)
    }
    
    limit := bh.normal
    if bh.count < bh.normal {
        limit = bh.burst
    }
    
    if bh.count < limit {
        bh.count++
        return true
    }
    
    return false
}

func main() {
    handler := NewBurstHandler(5, 10, 10*time.Second)
    
    for i := 0; i < 15; i++ {
        if handler.Allow() {
            fmt.Printf("Request %d: allowed\n", i+1)
        } else {
            fmt.Printf("Request %d: rejected\n", i+1)
        }
    }
}
```

## Message Replay Detection

Replay detection prevents processing duplicate messages in security contexts.  

```go
package main

import (
    "crypto/rand"
    "encoding/hex"
    "fmt"
    "sync"
    "time"
)

type ReplayDetector struct {
    seen   map[string]time.Time
    window time.Duration
    mu     sync.RWMutex
}

func NewReplayDetector(window time.Duration) *ReplayDetector {
    rd := &ReplayDetector{
        seen:   make(map[string]time.Time),
        window: window,
    }
    
    go rd.cleanup()
    
    return rd
}

func (rd *ReplayDetector) IsReplay(nonce string) bool {
    rd.mu.Lock()
    defer rd.mu.Unlock()
    
    if timestamp, exists := rd.seen[nonce]; exists {
        if time.Since(timestamp) < rd.window {
            return true
        }
    }
    
    rd.seen[nonce] = time.Now()
    return false
}

func (rd *ReplayDetector) cleanup() {
    ticker := time.NewTicker(rd.window)
    defer ticker.Stop()
    
    for range ticker.C {
        rd.mu.Lock()
        
        cutoff := time.Now().Add(-rd.window)
        for nonce, timestamp := range rd.seen {
            if timestamp.Before(cutoff) {
                delete(rd.seen, nonce)
            }
        }
        
        rd.mu.Unlock()
    }
}

func generateNonce() string {
    b := make([]byte, 16)
    rand.Read(b)
    return hex.EncodeToString(b)
}

func main() {
    detector := NewReplayDetector(5 * time.Second)
    
    nonce1 := generateNonce()
    nonce2 := generateNonce()
    
    fmt.Printf("First use of nonce1: replay=%v\n", detector.IsReplay(nonce1))
    fmt.Printf("First use of nonce2: replay=%v\n", detector.IsReplay(nonce2))
    fmt.Printf("Second use of nonce1: replay=%v\n", detector.IsReplay(nonce1))
}
```

## Connection Telemetry

Telemetry collects detailed connection metrics for observability.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Telemetry struct {
    connectionsOpened  int64
    connectionsClosed  int64
    bytesReceived      int64
    bytesSent          int64
    messagesReceived   int64
    messagesSent       int64
    errors             int64
    startTime          time.Time
    mu                 sync.RWMutex
}

func NewTelemetry() *Telemetry {
    return &Telemetry{
        startTime: time.Now(),
    }
}

func (t *Telemetry) RecordOpen() {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.connectionsOpened++
}

func (t *Telemetry) RecordClose() {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.connectionsClosed++
}

func (t *Telemetry) RecordReceived(bytes, messages int64) {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.bytesReceived += bytes
    t.messagesReceived += messages
}

func (t *Telemetry) RecordSent(bytes, messages int64) {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.bytesSent += bytes
    t.messagesSent += messages
}

func (t *Telemetry) RecordError() {
    t.mu.Lock()
    defer t.mu.Unlock()
    t.errors++
}

func (t *Telemetry) Report() {
    t.mu.RLock()
    defer t.mu.RUnlock()
    
    uptime := time.Since(t.startTime)
    active := t.connectionsOpened - t.connectionsClosed
    
    fmt.Println("=== Telemetry Report ===")
    fmt.Printf("Uptime: %v\n", uptime)
    fmt.Printf("Connections: opened=%d, closed=%d, active=%d\n", 
        t.connectionsOpened, t.connectionsClosed, active)
    fmt.Printf("Bytes: received=%d, sent=%d\n", 
        t.bytesReceived, t.bytesSent)
    fmt.Printf("Messages: received=%d, sent=%d\n", 
        t.messagesReceived, t.messagesSent)
    fmt.Printf("Errors: %d\n", t.errors)
}

func main() {
    telem := NewTelemetry()
    
    telem.RecordOpen()
    telem.RecordReceived(1024, 5)
    telem.RecordSent(2048, 10)
    telem.RecordClose()
    
    time.Sleep(1 * time.Second)
    
    telem.Report()
}
```

## Request Correlation IDs

Correlation IDs track requests across distributed systems.  

```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "time"
)

type correlationKeyType string

const correlationKey correlationKeyType = "correlation-id"

func WithCorrelationID(ctx context.Context) context.Context {
    id := generateID()
    return context.WithValue(ctx, correlationKey, id)
}

func GetCorrelationID(ctx context.Context) string {
    if id, ok := ctx.Value(correlationKey).(string); ok {
        return id
    }
    return ""
}

func generateID() string {
    return fmt.Sprintf("%d-%d", time.Now().UnixNano(), rand.Int63())
}

func processRequest(ctx context.Context, data string) {
    id := GetCorrelationID(ctx)
    fmt.Printf("[%s] Processing: %s\n", id, data)
}

func main() {
    rand.Seed(time.Now().UnixNano())
    
    ctx1 := WithCorrelationID(context.Background())
    ctx2 := WithCorrelationID(context.Background())
    
    processRequest(ctx1, "Request 1")
    processRequest(ctx2, "Request 2")
    processRequest(ctx1, "Request 1 continued")
}
```

## Connection Load Shedding

Load shedding drops excess requests when capacity is exceeded.  

```go
package main

import (
    "fmt"
    "sync/atomic"
    "time"
)

type LoadShedder struct {
    maxLoad     int64
    currentLoad int64
}

func NewLoadShedder(maxLoad int64) *LoadShedder {
    return &LoadShedder{
        maxLoad: maxLoad,
    }
}

func (ls *LoadShedder) Acquire() bool {
    current := atomic.LoadInt64(&ls.currentLoad)
    if current >= ls.maxLoad {
        return false
    }
    
    atomic.AddInt64(&ls.currentLoad, 1)
    return true
}

func (ls *LoadShedder) Release() {
    atomic.AddInt64(&ls.currentLoad, -1)
}

func (ls *LoadShedder) CurrentLoad() int64 {
    return atomic.LoadInt64(&ls.currentLoad)
}

func main() {
    shedder := NewLoadShedder(3)
    
    for i := 0; i < 5; i++ {
        if shedder.Acquire() {
            fmt.Printf("Request %d: accepted (load: %d)\n", 
                i+1, shedder.CurrentLoad())
            
            go func(id int) {
                time.Sleep(1 * time.Second)
                shedder.Release()
                fmt.Printf("Request %d: completed\n", id)
            }(i + 1)
        } else {
            fmt.Printf("Request %d: shed (overload)\n", i+1)
        }
        
        time.Sleep(200 * time.Millisecond)
    }
    
    time.Sleep(2 * time.Second)
}
```

## Connection Draining Strategy

Graceful draining ensures clean shutdown without dropping in-flight requests.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Drainer struct {
    draining bool
    active   int
    mu       sync.RWMutex
    done     chan struct{}
}

func NewDrainer() *Drainer {
    return &Drainer{
        done: make(chan struct{}),
    }
}

func (d *Drainer) Enter() bool {
    d.mu.RLock()
    defer d.mu.RUnlock()
    
    if d.draining {
        return false
    }
    
    d.active++
    return true
}

func (d *Drainer) Leave() {
    d.mu.Lock()
    defer d.mu.Unlock()
    
    d.active--
    
    if d.draining && d.active == 0 {
        close(d.done)
    }
}

func (d *Drainer) Drain(timeout time.Duration) bool {
    d.mu.Lock()
    d.draining = true
    d.mu.Unlock()
    
    select {
    case <-d.done:
        return true
    case <-time.After(timeout):
        return false
    }
}

func main() {
    drainer := NewDrainer()
    
    for i := 0; i < 3; i++ {
        if drainer.Enter() {
            go func(id int) {
                defer drainer.Leave()
                
                fmt.Printf("Request %d: processing\n", id)
                time.Sleep(1 * time.Second)
                fmt.Printf("Request %d: complete\n", id)
            }(i + 1)
        }
    }
    
    time.Sleep(500 * time.Millisecond)
    
    fmt.Println("Starting drain...")
    success := drainer.Drain(3 * time.Second)
    
    if success {
        fmt.Println("Drain completed successfully")
    } else {
        fmt.Println("Drain timed out")
    }
}
```

## Service Discovery Integration

Service discovery enables dynamic backend discovery for resilience.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type ServiceRegistry struct {
    services map[string][]string
    mu       sync.RWMutex
}

func NewServiceRegistry() *ServiceRegistry {
    return &ServiceRegistry{
        services: make(map[string][]string),
    }
}

func (sr *ServiceRegistry) Register(service, endpoint string) {
    sr.mu.Lock()
    defer sr.mu.Unlock()
    
    sr.services[service] = append(sr.services[service], endpoint)
    fmt.Printf("Registered: %s -> %s\n", service, endpoint)
}

func (sr *ServiceRegistry) Discover(service string) []string {
    sr.mu.RLock()
    defer sr.mu.RUnlock()
    
    return sr.services[service]
}

func (sr *ServiceRegistry) Deregister(service, endpoint string) {
    sr.mu.Lock()
    defer sr.mu.Unlock()
    
    endpoints := sr.services[service]
    for i, ep := range endpoints {
        if ep == endpoint {
            sr.services[service] = append(endpoints[:i], endpoints[i+1:]...)
            fmt.Printf("Deregistered: %s -> %s\n", service, endpoint)
            break
        }
    }
}

func main() {
    registry := NewServiceRegistry()
    
    registry.Register("api", "host1:8080")
    registry.Register("api", "host2:8080")
    registry.Register("db", "host3:5432")
    
    endpoints := registry.Discover("api")
    fmt.Printf("API endpoints: %v\n", endpoints)
    
    registry.Deregister("api", "host1:8080")
    
    endpoints = registry.Discover("api")
    fmt.Printf("API endpoints after deregister: %v\n", endpoints)
}
```

## Connection Queueing Strategies

Different queuing strategies optimize for throughput or latency.  

```go
package main

import (
    "fmt"
    "time"
)

type QueueStrategy interface {
    Enqueue(item interface{}) bool
    Dequeue() interface{}
}

type FIFOQueue struct {
    items []interface{}
    max   int
}

func NewFIFOQueue(max int) *FIFOQueue {
    return &FIFOQueue{
        items: make([]interface{}, 0, max),
        max:   max,
    }
}

func (q *FIFOQueue) Enqueue(item interface{}) bool {
    if len(q.items) >= q.max {
        return false
    }
    
    q.items = append(q.items, item)
    return true
}

func (q *FIFOQueue) Dequeue() interface{} {
    if len(q.items) == 0 {
        return nil
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item
}

type LIFOQueue struct {
    items []interface{}
    max   int
}

func NewLIFOQueue(max int) *LIFOQueue {
    return &LIFOQueue{
        items: make([]interface{}, 0, max),
        max:   max,
    }
}

func (q *LIFOQueue) Enqueue(item interface{}) bool {
    if len(q.items) >= q.max {
        return false
    }
    
    q.items = append(q.items, item)
    return true
}

func (q *LIFOQueue) Dequeue() interface{} {
    if len(q.items) == 0 {
        return nil
    }
    
    item := q.items[len(q.items)-1]
    q.items = q.items[:len(q.items)-1]
    return item
}

func main() {
    fifo := NewFIFOQueue(10)
    lifo := NewLIFOQueue(10)
    
    for i := 1; i <= 3; i++ {
        fifo.Enqueue(i)
        lifo.Enqueue(i)
    }
    
    fmt.Println("FIFO order:")
    for i := 0; i < 3; i++ {
        fmt.Printf("  %v\n", fifo.Dequeue())
    }
    
    fmt.Println("LIFO order:")
    for i := 0; i < 3; i++ {
        fmt.Printf("  %v\n", lifo.Dequeue())
    }
}
```

## Connection Failure Recovery

Automatic recovery reconnects after failures with exponential backoff.  

```go
package main

import (
    "fmt"
    "net"
    "time"
)

type RecoverableConnection struct {
    address   string
    conn      net.Conn
    reconnect chan struct{}
}

func NewRecoverableConnection(address string) *RecoverableConnection {
    rc := &RecoverableConnection{
        address:   address,
        reconnect: make(chan struct{}, 1),
    }
    
    go rc.maintain()
    
    return rc
}

func (rc *RecoverableConnection) maintain() {
    backoff := 100 * time.Millisecond
    maxBackoff := 10 * time.Second
    
    for {
        conn, err := net.Dial("tcp", rc.address)
        if err != nil {
            fmt.Printf("Connection failed: %v, retry in %v\n", err, backoff)
            time.Sleep(backoff)
            
            backoff *= 2
            if backoff > maxBackoff {
                backoff = maxBackoff
            }
            
            continue
        }
        
        rc.conn = conn
        backoff = 100 * time.Millisecond
        
        fmt.Println("Connection established")
        
        <-rc.reconnect
    }
}

func (rc *RecoverableConnection) TriggerReconnect() {
    select {
    case rc.reconnect <- struct{}{}:
    default:
    }
}

func main() {
    rc := NewRecoverableConnection("localhost:8080")
    
    time.Sleep(5 * time.Second)
    
    fmt.Println("Triggering reconnect...")
    rc.TriggerReconnect()
    
    time.Sleep(2 * time.Second)
}
```

## Connection Metadata Tagging

Metadata tagging enriches connections with application context.  

```go
package main

import (
    "fmt"
    "net"
    "sync"
)

type TaggedConn struct {
    conn     net.Conn
    metadata map[string]string
    mu       sync.RWMutex
}

func NewTaggedConn(conn net.Conn) *TaggedConn {
    return &TaggedConn{
        conn:     conn,
        metadata: make(map[string]string),
    }
}

func (tc *TaggedConn) SetTag(key, value string) {
    tc.mu.Lock()
    defer tc.mu.Unlock()
    tc.metadata[key] = value
}

func (tc *TaggedConn) GetTag(key string) string {
    tc.mu.RLock()
    defer tc.mu.RUnlock()
    return tc.metadata[key]
}

func (tc *TaggedConn) GetAllTags() map[string]string {
    tc.mu.RLock()
    defer tc.mu.RUnlock()
    
    tags := make(map[string]string)
    for k, v := range tc.metadata {
        tags[k] = v
    }
    
    return tags
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        conn, _ := listener.Accept()
        
        tagged := NewTaggedConn(conn)
        tagged.SetTag("user", "alice")
        tagged.SetTag("priority", "high")
        tagged.SetTag("region", "us-east")
        
        fmt.Printf("Connection tags: %v\n", tagged.GetAllTags())
        
        conn.Close()
    }()
    
    conn, _ := net.Dial("tcp", "localhost:8080")
    conn.Close()
    
    time.Sleep(100 * time.Millisecond)
}
```

## Performance Benchmarking

Benchmarking measures network operation performance for optimization.  

```go
package main

import (
    "fmt"
    "net"
    "time"
)

type Benchmark struct {
    operations int
    duration   time.Duration
}

func (b *Benchmark) Run(fn func()) {
    start := time.Now()
    
    for i := 0; i < b.operations; i++ {
        fn()
    }
    
    b.duration = time.Since(start)
}

func (b *Benchmark) Report() {
    opsPerSec := float64(b.operations) / b.duration.Seconds()
    avgLatency := b.duration / time.Duration(b.operations)
    
    fmt.Printf("Operations: %d\n", b.operations)
    fmt.Printf("Total time: %v\n", b.duration)
    fmt.Printf("Ops/sec: %.2f\n", opsPerSec)
    fmt.Printf("Avg latency: %v\n", avgLatency)
}

func main() {
    listener, _ := net.Listen("tcp", ":8080")
    defer listener.Close()
    
    go func() {
        for {
            conn, _ := listener.Accept()
            go func(c net.Conn) {
                defer c.Close()
                
                buffer := make([]byte, 1024)
                n, _ := c.Read(buffer)
                c.Write(buffer[:n])
            }(conn)
        }
    }()
    
    time.Sleep(100 * time.Millisecond)
    
    bench := &Benchmark{operations: 1000}
    
    bench.Run(func() {
        conn, _ := net.Dial("tcp", "localhost:8080")
        defer conn.Close()
        
        conn.Write([]byte("ping"))
        
        buffer := make([]byte, 4)
        conn.Read(buffer)
    })
    
    bench.Report()
}
```

## Network Protocol State Transitions

State machines model protocol lifecycle for correctness.  

```go
package main

import (
    "fmt"
)

type ProtocolState int

const (
    StateInit ProtocolState = iota
    StateHandshake
    StateAuthenticated
    StateActive
    StateClosing
    StateClosed
)

type StateMachine struct {
    state      ProtocolState
    transitions map[ProtocolState][]ProtocolState
}

func NewStateMachine() *StateMachine {
    sm := &StateMachine{
        state: StateInit,
        transitions: map[ProtocolState][]ProtocolState{
            StateInit:          {StateHandshake},
            StateHandshake:     {StateAuthenticated, StateClosed},
            StateAuthenticated: {StateActive, StateClosed},
            StateActive:        {StateClosing},
            StateClosing:       {StateClosed},
        },
    }
    
    return sm
}

func (sm *StateMachine) Transition(to ProtocolState) error {
    allowed := sm.transitions[sm.state]
    
    for _, allowed := range allowed {
        if to == allowed {
            fmt.Printf("State transition: %v -> %v\n", sm.state, to)
            sm.state = to
            return nil
        }
    }
    
    return fmt.Errorf("invalid transition: %v -> %v", sm.state, to)
}

func (sm *StateMachine) CurrentState() ProtocolState {
    return sm.state
}

func main() {
    sm := NewStateMachine()
    
    sm.Transition(StateHandshake)
    sm.Transition(StateAuthenticated)
    sm.Transition(StateActive)
    sm.Transition(StateClosing)
    sm.Transition(StateClosed)
    
    fmt.Printf("Final state: %v\n", sm.CurrentState())
}
```

## Summary

This comprehensive guide covered 99 practical Go examples demonstrating  
low-level networking operations. The examples progressed from basic socket  
programming through advanced patterns like connection pooling, custom  
protocols, security, flow control, and performance optimization.  

Key concepts covered include TCP/UDP fundamentals, connection lifecycle  
management, binary protocols, stream handling, multiplexing, rate limiting,  
health monitoring, encryption, load balancing, and advanced patterns for  
production systems.  

Each example follows Go best practices with proper error handling, resource  
cleanup, and concurrent programming patterns. The examples demonstrate both  
client and server implementations where applicable, showing complete  
bidirectional communication patterns.  

Understanding these low-level networking concepts enables building robust,  
performant, and scalable network applications in Go. The standard library  
provides excellent primitives for networking, while the language's  
concurrency features make it ideal for high-performance network services.  

