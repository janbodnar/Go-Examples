# HTTP/Web Programming

HTTP (Hypertext Transfer Protocol) is the foundation of data communication on  
the World Wide Web. Go's `net/http` package provides a comprehensive set of  
tools for building HTTP clients and servers, making it one of the most popular  
choices for web development and API creation. The package offers both high-level  
convenience functions and low-level primitives for maximum flexibility.

Go's approach to HTTP programming emphasizes simplicity and performance. The  
standard library includes everything needed to build production-ready web  
applications and services without requiring external frameworks. This design  
philosophy aligns with Go's overall goal of providing powerful tools with  
minimal complexity and excellent performance characteristics.

The `net/http` package follows Go's interface-based design, making it highly  
composable and extensible. HTTP handlers implement the simple `Handler`  
interface, which takes a `ResponseWriter` and `*Request`. This design enables  
powerful middleware patterns, request routing, and custom functionality while  
maintaining type safety and performance.

HTTP clients in Go are built around the `Client` type, which provides  
configurable timeout behavior, custom transport layers, and automatic handling  
of cookies, redirects, and compression. The client API supports all HTTP  
methods and provides fine-grained control over request construction and  
response processing.

Server development in Go centers around the `Server` type and the  
`ListenAndServe` family of functions. Go's HTTP server supports concurrent  
request handling by default, with each request processed in its own goroutine.  
This built-in concurrency model makes Go servers highly scalable without  
requiring complex threading or async/await patterns.

Advanced features include HTTP/2 support, WebSocket upgrades, server-sent  
events, graceful shutdown, TLS configuration, and middleware composition.  
The package also provides excellent support for testing HTTP code through  
the `httptest` package, enabling comprehensive unit and integration testing.

## Basic HTTP GET request

The simplest HTTP operation is making a GET request to retrieve data from  
a server. Go's `http.Get` function provides a convenient way to perform  
basic GET requests with default settings.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
)

func main() {
    resp, err := http.Get("https://httpbin.org/get")
    if err != nil {
        log.Fatal("Error making request:", err)
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        log.Fatal("Error reading response:", err)
    }

    fmt.Printf("Status: %s\n", resp.Status)
    fmt.Printf("Status Code: %d\n", resp.StatusCode)
    fmt.Printf("Response Body:\n%s\n", body)
}
```

This example demonstrates the basic pattern for HTTP requests in Go. Always  
check for errors from the HTTP operation, defer closing the response body to  
prevent resource leaks, and read the entire response body using `io.ReadAll`.  
The response includes status information and headers alongside the body content.

## HTTP GET with custom headers

Adding custom headers to HTTP requests is essential for API authentication,  
content negotiation, and providing metadata about the client or request.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
)

func main() {
    client := &http.Client{}
    
    req, err := http.NewRequest("GET", "https://httpbin.org/headers", nil)
    if err != nil {
        log.Fatal("Error creating request:", err)
    }

    req.Header.Set("User-Agent", "Go-HTTP-Client/1.0")
    req.Header.Set("Accept", "application/json")
    req.Header.Set("Authorization", "Bearer your-token-here")
    req.Header.Add("X-Custom-Header", "custom-value")

    resp, err := client.Do(req)
    if err != nil {
        log.Fatal("Error making request:", err)
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        log.Fatal("Error reading response:", err)
    }

    fmt.Printf("Request Headers Sent:\n")
    for name, values := range req.Header {
        for _, value := range values {
            fmt.Printf("%s: %s\n", name, value)
        }
    }

    fmt.Printf("\nResponse:\n%s\n", body)
}
```

Creating custom requests with `http.NewRequest` allows complete control over  
headers, HTTP method, and request body. Use `Header.Set` to replace existing  
headers and `Header.Add` to append additional values. This pattern is essential  
for integrating with APIs that require specific headers for authentication or  
content negotiation.

## HTTP POST with JSON data

Sending JSON data via POST requests is fundamental for REST API interactions  
and data submission. This example shows proper JSON marshaling and content  
type configuration.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
)

type User struct {
    Name     string `json:"name"`
    Email    string `json:"email"`
    Age      int    `json:"age"`
    Location string `json:"location"`
}

func main() {
    user := User{
        Name:     "Alice Johnson",
        Email:    "alice@example.com",
        Age:      28,
        Location: "San Francisco",
    }

    jsonData, err := json.Marshal(user)
    if err != nil {
        log.Fatal("Error marshaling JSON:", err)
    }

    resp, err := http.Post(
        "https://httpbin.org/post",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        log.Fatal("Error making POST request:", err)
    }
    defer resp.Body.Close()

    responseBody, err := io.ReadAll(resp.Body)
    if err != nil {
        log.Fatal("Error reading response:", err)
    }

    fmt.Printf("Sent JSON data:\n%s\n\n", jsonData)
    fmt.Printf("Status: %s\n", resp.Status)
    fmt.Printf("Response:\n%s\n", responseBody)
}
```

JSON marshaling converts Go structs to JSON format for transmission. The  
`bytes.NewBuffer` function creates an `io.Reader` from the JSON data, and  
setting the content type to "application/json" ensures the server correctly  
interprets the request body. This pattern is essential for REST API clients.

## HTTP PUT request

PUT requests are used to create or update resources, typically sending the  
complete resource representation to the server. This example demonstrates  
updating user data with proper error handling.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
)

type UserUpdate struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Active   bool   `json:"active"`
    Modified string `json:"modified"`
}

func main() {
    client := &http.Client{}
    
    updatedUser := UserUpdate{
        ID:       123,
        Name:     "Alice Smith",
        Email:    "alice.smith@example.com",
        Active:   true,
        Modified: "2024-01-15T10:30:00Z",
    }

    jsonData, err := json.Marshal(updatedUser)
    if err != nil {
        log.Fatal("Error marshaling JSON:", err)
    }

    req, err := http.NewRequest("PUT", "https://httpbin.org/put", 
                               bytes.NewBuffer(jsonData))
    if err != nil {
        log.Fatal("Error creating request:", err)
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Accept", "application/json")

    resp, err := client.Do(req)
    if err != nil {
        log.Fatal("Error making PUT request:", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        fmt.Printf("Unexpected status code: %d\n", resp.StatusCode)
    }

    responseBody, err := io.ReadAll(resp.Body)
    if err != nil {
        log.Fatal("Error reading response:", err)
    }

    fmt.Printf("Updated user data:\n%s\n\n", jsonData)
    fmt.Printf("Server response:\n%s\n", responseBody)
}
```

PUT requests typically send the complete resource representation to create or  
replace an existing resource. The example shows proper JSON marshaling,  
content type headers, and status code checking. PUT is idempotent, meaning  
multiple identical requests should have the same effect as a single request.

## HTTP DELETE request

DELETE requests remove resources from the server. This example shows how to  
send DELETE requests with proper authentication and response handling.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
)

func deleteUser(client *http.Client, userID int, authToken string) error {
    url := fmt.Sprintf("https://httpbin.org/delete?user_id=%d", userID)
    
    req, err := http.NewRequest("DELETE", url, nil)
    if err != nil {
        return fmt.Errorf("error creating DELETE request: %w", err)
    }

    req.Header.Set("Authorization", "Bearer "+authToken)
    req.Header.Set("Accept", "application/json")

    resp, err := client.Do(req)
    if err != nil {
        return fmt.Errorf("error executing DELETE request: %w", err)
    }
    defer resp.Body.Close()

    switch resp.StatusCode {
    case http.StatusOK, http.StatusNoContent:
        fmt.Printf("User %d deleted successfully\n", userID)
    case http.StatusNotFound:
        return fmt.Errorf("user %d not found", userID)
    case http.StatusUnauthorized:
        return fmt.Errorf("unauthorized to delete user %d", userID)
    default:
        body, _ := io.ReadAll(resp.Body)
        return fmt.Errorf("unexpected status %d: %s", resp.StatusCode, body)
    }

    return nil
}

func main() {
    client := &http.Client{}
    authToken := "your-auth-token-here"
    
    userIDs := []int{123, 456, 789}

    for _, userID := range userIDs {
        fmt.Printf("Attempting to delete user %d...\n", userID)
        
        if err := deleteUser(client, userID, authToken); err != nil {
            log.Printf("Failed to delete user %d: %v", userID, err)
        }
    }
}
```

DELETE requests often require authentication and careful status code handling.  
The example demonstrates URL parameter construction, authorization headers, and  
comprehensive status code checking. Proper error handling distinguishes between  
different failure scenarios like authorization failures and resource not found.

## HTTP request with timeout

Timeouts are crucial for preventing requests from hanging indefinitely and  
ensuring application responsiveness. This example shows different timeout  
configuration approaches.

```go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "net/http"
    "time"
)

func requestWithClientTimeout() {
    client := &http.Client{
        Timeout: 5 * time.Second,
    }

    fmt.Println("Making request with client timeout...")
    resp, err := client.Get("https://httpbin.org/delay/3")
    if err != nil {
        log.Printf("Client timeout error: %v", err)
        return
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Response received: %d bytes\n", len(body))
}

func requestWithContextTimeout() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", 
                                          "https://httpbin.org/delay/4", nil)
    if err != nil {
        log.Fatal("Error creating request:", err)
    }

    fmt.Println("Making request with context timeout...")
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        log.Printf("Context timeout error: %v", err)
        return
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Response received: %d bytes\n", len(body))
}

func main() {
    fmt.Println("=== HTTP Timeout Examples ===")
    
    requestWithClientTimeout()
    fmt.Println()
    
    requestWithContextTimeout()
    
    fmt.Println("\nTimeout examples completed")
}
```

Client-level timeouts apply to all requests made by that client, while  
context-based timeouts provide per-request control and better integration  
with request cancellation. Context timeouts are preferred for server  
applications where request deadlines need to be propagated through the  
call stack.

## HTTP request with query parameters

Query parameters are used to pass data to the server via the URL. This  
example demonstrates different approaches to building URLs with parameters.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "net/url"
    "strconv"
)

type SearchParams struct {
    Query    string
    Page     int
    PageSize int
    SortBy   string
    Order    string
    Filter   []string
}

func buildSearchURL(baseURL string, params SearchParams) string {
    u, err := url.Parse(baseURL)
    if err != nil {
        log.Fatal("Invalid base URL:", err)
    }

    q := u.Query()
    q.Set("q", params.Query)
    q.Set("page", strconv.Itoa(params.Page))
    q.Set("page_size", strconv.Itoa(params.PageSize))
    q.Set("sort_by", params.SortBy)
    q.Set("order", params.Order)

    for _, filter := range params.Filter {
        q.Add("filter", filter)
    }

    u.RawQuery = q.Encode()
    return u.String()
}

func main() {
    baseURL := "https://httpbin.org/get"
    
    // Simple query parameters
    simpleURL := baseURL + "?name=Alice&age=25&city=San Francisco"
    fmt.Printf("Simple URL: %s\n", simpleURL)

    resp, err := http.Get(simpleURL)
    if err != nil {
        log.Fatal("Error making request:", err)
    }
    resp.Body.Close()

    // Using url.Values for complex parameters
    params := url.Values{}
    params.Set("search", "golang http")
    params.Set("category", "programming")
    params.Add("tag", "web")
    params.Add("tag", "api")
    params.Add("tag", "backend")

    complexURL := baseURL + "?" + params.Encode()
    fmt.Printf("Complex URL: %s\n", complexURL)

    // Using structured approach
    searchParams := SearchParams{
        Query:    "http programming",
        Page:     2,
        PageSize: 20,
        SortBy:   "relevance",
        Order:    "desc",
        Filter:   []string{"recent", "verified", "popular"},
    }

    structuredURL := buildSearchURL(baseURL, searchParams)
    fmt.Printf("Structured URL: %s\n", structuredURL)

    resp, err = http.Get(structuredURL)
    if err != nil {
        log.Fatal("Error making structured request:", err)
    }
    defer resp.Body.Close()

    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Response length: %d bytes\n", len(body))
}
```

URL encoding ensures special characters in query parameters are properly  
handled. The `url.Values` type provides convenient methods for building  
complex query strings, while the structured approach using custom types  
makes code more maintainable and type-safe. Always use `url.QueryEscape`  
or `url.Values` instead of manual string concatenation.

## Basic HTTP server

Creating HTTP servers in Go is straightforward with the `net/http` package.  
This example demonstrates a basic server with multiple routes and different  
response types.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "time"
)

type ServerInfo struct {
    Name      string    `json:"name"`
    Version   string    `json:"version"`
    Timestamp time.Time `json:"timestamp"`
    Uptime    string    `json:"uptime"`
}

var serverStartTime = time.Now()

func homeHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    html := `
    <html>
    <head><title>Go HTTP Server</title></head>
    <body>
        <h1>Hello there!</h1>
        <p>Welcome to the Go HTTP server example.</p>
        <ul>
            <li><a href="/info">Server Info (JSON)</a></li>
            <li><a href="/time">Current Time</a></li>
            <li><a href="/hello?name=Alice">Greeting</a></li>
        </ul>
    </body>
    </html>`
    
    fmt.Fprint(w, html)
}

func infoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    uptime := time.Since(serverStartTime).Round(time.Second)
    
    info := ServerInfo{
        Name:      "Go HTTP Example Server",
        Version:   "1.0.0",
        Timestamp: time.Now(),
        Uptime:    uptime.String(),
    }

    if err := json.NewEncoder(w).Encode(info); err != nil {
        http.Error(w, "Error encoding JSON", http.StatusInternalServerError)
        return
    }
}

func timeHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/plain")
    currentTime := time.Now().Format("2006-01-02 15:04:05 MST")
    fmt.Fprintf(w, "Current server time: %s\n", currentTime)
}

func greetingHandler(w http.ResponseWriter, r *http.Request) {
    name := r.URL.Query().Get("name")
    if name == "" {
        name = "Guest"
    }
    
    w.Header().Set("Content-Type", "text/plain")
    fmt.Fprintf(w, "Hello there, %s! Welcome to our Go server.\n", name)
}

func main() {
    http.HandleFunc("/", homeHandler)
    http.HandleFunc("/info", infoHandler)
    http.HandleFunc("/time", timeHandler)
    http.HandleFunc("/hello", greetingHandler)

    port := ":8080"
    fmt.Printf("Starting server on http://localhost%s\n", port)
    fmt.Println("Press Ctrl+C to stop the server")

    if err := http.ListenAndServe(port, nil); err != nil {
        log.Fatal("Server failed to start:", err)
    }
}
```

This server demonstrates different response types: HTML, JSON, and plain text.  
Each handler function receives a `ResponseWriter` for sending responses and a  
`*Request` containing request data. The server automatically handles concurrent  
requests using goroutines, making it scalable without additional complexity.

## HTTP server with custom multiplexer

Using a custom multiplexer provides more control over routing and middleware.  
This example shows how to create a custom server configuration with enhanced  
routing capabilities.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type Book struct {
    ID     int    `json:"id"`
    Title  string `json:"title"`
    Author string `json:"author"`
    Year   int    `json:"year"`
}

var books = []Book{
    {1, "The Go Programming Language", "Alan Donovan", 2015},
    {2, "Effective Go", "Go Team", 2009},
    {3, "Go in Action", "William Kennedy", 2015},
}

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        duration := time.Since(start)
        log.Printf("%s %s - %v", r.Method, r.URL.Path, duration)
    })
}

func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
        
        if r.Method == http.MethodOptions {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

func booksHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    switch r.Method {
    case http.MethodGet:
        if err := json.NewEncoder(w).Encode(books); err != nil {
            http.Error(w, "Error encoding books", 
                      http.StatusInternalServerError)
        }
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func bookHandler(w http.ResponseWriter, r *http.Request) {
    path := strings.TrimPrefix(r.URL.Path, "/books/")
    if path == "" {
        http.Error(w, "Book ID required", http.StatusBadRequest)
        return
    }

    bookID, err := strconv.Atoi(path)
    if err != nil {
        http.Error(w, "Invalid book ID", http.StatusBadRequest)
        return
    }

    w.Header().Set("Content-Type", "application/json")

    for _, book := range books {
        if book.ID == bookID {
            if err := json.NewEncoder(w).Encode(book); err != nil {
                http.Error(w, "Error encoding book", 
                          http.StatusInternalServerError)
            }
            return
        }
    }

    http.Error(w, "Book not found", http.StatusNotFound)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    
    response := map[string]interface{}{
        "status":    "healthy",
        "timestamp": time.Now(),
        "books":     len(books),
    }
    
    json.NewEncoder(w).Encode(response)
}

func main() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/books", booksHandler)
    mux.HandleFunc("/books/", bookHandler)
    mux.HandleFunc("/health", healthHandler)

    // Apply middleware
    handler := loggingMiddleware(corsMiddleware(mux))

    server := &http.Server{
        Addr:         ":8080",
        Handler:      handler,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    fmt.Println("Server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  GET /books - List all books")
    fmt.Println("  GET /books/{id} - Get specific book")
    fmt.Println("  GET /health - Health check")

    if err := server.ListenAndServe(); err != nil {
        log.Fatal("Server failed:", err)
    }
}
```

Custom multiplexers enable advanced routing patterns and middleware composition.  
This example shows logging middleware, CORS support, RESTful routing, and  
server configuration with timeouts. The middleware pattern allows for clean  
separation of cross-cutting concerns like logging, authentication, and CORS.

## HTTP request handling and routing

Sophisticated request handling involves parsing different content types,  
handling various HTTP methods, and implementing proper error responses.  
This example demonstrates comprehensive request processing.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "net/url"
    "strconv"
    "strings"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type ErrorResponse struct {
    Error   string `json:"error"`
    Code    int    `json:"code"`
    Message string `json:"message"`
}

var users = make(map[int]User)
var nextUserID = 1

func sendJSONError(w http.ResponseWriter, message string, code int) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    
    errorResp := ErrorResponse{
        Error:   "request_failed",
        Code:    code,
        Message: message,
    }
    
    json.NewEncoder(w).Encode(errorResp)
}

func sendJSONResponse(w http.ResponseWriter, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    
    if err := json.NewEncoder(w).Encode(data); err != nil {
        log.Printf("Error encoding JSON response: %v", err)
    }
}

func parseUserFromJSON(r *http.Request) (User, error) {
    var user User
    
    contentType := r.Header.Get("Content-Type")
    if !strings.Contains(contentType, "application/json") {
        return user, fmt.Errorf("content type must be application/json")
    }
    
    body, err := io.ReadAll(r.Body)
    if err != nil {
        return user, fmt.Errorf("error reading request body: %w", err)
    }
    
    if err := json.Unmarshal(body, &user); err != nil {
        return user, fmt.Errorf("error parsing JSON: %w", err)
    }
    
    return user, nil
}

func parseUserFromForm(r *http.Request) (User, error) {
    if err := r.ParseForm(); err != nil {
        return User{}, fmt.Errorf("error parsing form: %w", err)
    }
    
    name := r.Form.Get("name")
    email := r.Form.Get("email")
    
    if name == "" || email == "" {
        return User{}, fmt.Errorf("name and email are required")
    }
    
    return User{Name: name, Email: email}, nil
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        handleGetUsers(w, r)
    case http.MethodPost:
        handleCreateUser(w, r)
    default:
        sendJSONError(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func userHandler(w http.ResponseWriter, r *http.Request) {
    idStr := strings.TrimPrefix(r.URL.Path, "/users/")
    if idStr == "" {
        sendJSONError(w, "User ID required", http.StatusBadRequest)
        return
    }
    
    userID, err := strconv.Atoi(idStr)
    if err != nil {
        sendJSONError(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    switch r.Method {
    case http.MethodGet:
        handleGetUser(w, r, userID)
    case http.MethodPut:
        handleUpdateUser(w, r, userID)
    case http.MethodDelete:
        handleDeleteUser(w, r, userID)
    default:
        sendJSONError(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func handleGetUsers(w http.ResponseWriter, r *http.Request) {
    userList := make([]User, 0, len(users))
    for _, user := range users {
        userList = append(userList, user)
    }
    
    sendJSONResponse(w, map[string]interface{}{
        "users": userList,
        "count": len(userList),
    })
}

func handleCreateUser(w http.ResponseWriter, r *http.Request) {
    var user User
    var err error
    
    contentType := r.Header.Get("Content-Type")
    if strings.Contains(contentType, "application/json") {
        user, err = parseUserFromJSON(r)
    } else if strings.Contains(contentType, "application/x-www-form-urlencoded") {
        user, err = parseUserFromForm(r)
    } else {
        sendJSONError(w, "Unsupported content type", 
                     http.StatusUnsupportedMediaType)
        return
    }
    
    if err != nil {
        sendJSONError(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    user.ID = nextUserID
    nextUserID++
    users[user.ID] = user
    
    w.WriteHeader(http.StatusCreated)
    sendJSONResponse(w, user)
}

func handleGetUser(w http.ResponseWriter, r *http.Request, userID int) {
    user, exists := users[userID]
    if !exists {
        sendJSONError(w, "User not found", http.StatusNotFound)
        return
    }
    
    sendJSONResponse(w, user)
}

func handleUpdateUser(w http.ResponseWriter, r *http.Request, userID int) {
    _, exists := users[userID]
    if !exists {
        sendJSONError(w, "User not found", http.StatusNotFound)
        return
    }
    
    user, err := parseUserFromJSON(r)
    if err != nil {
        sendJSONError(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    user.ID = userID
    users[userID] = user
    
    sendJSONResponse(w, user)
}

func handleDeleteUser(w http.ResponseWriter, r *http.Request, userID int) {
    _, exists := users[userID]
    if !exists {
        sendJSONError(w, "User not found", http.StatusNotFound)
        return
    }
    
    delete(users, userID)
    w.WriteHeader(http.StatusNoContent)
}

func main() {
    http.HandleFunc("/users", usersHandler)
    http.HandleFunc("/users/", userHandler)
    
    fmt.Println("User API server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  GET    /users     - List users")
    fmt.Println("  POST   /users     - Create user")
    fmt.Println("  GET    /users/{id} - Get user")
    fmt.Println("  PUT    /users/{id} - Update user")
    fmt.Println("  DELETE /users/{id} - Delete user")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This comprehensive example demonstrates RESTful API patterns with proper  
HTTP method handling, content type negotiation, error responses, and resource  
management. The code shows how to parse both JSON and form data, implement  
proper status codes, and maintain clean separation between parsing, validation,  
and business logic.

## HTTP response headers and status codes

Proper HTTP response handling involves setting appropriate headers and status  
codes to communicate effectively with clients. This example demonstrates  
various response scenarios and header management.

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "strconv"
    "time"
)

type APIResponse struct {
    Success   bool        `json:"success"`
    Data      interface{} `json:"data,omitempty"`
    Error     string      `json:"error,omitempty"`
    Timestamp time.Time   `json:"timestamp"`
}

func setCacheHeaders(w http.ResponseWriter, maxAge int) {
    w.Header().Set("Cache-Control", fmt.Sprintf("max-age=%d", maxAge))
    w.Header().Set("ETag", fmt.Sprintf("\"%d\"", time.Now().Unix()))
    w.Header().Set("Last-Modified", time.Now().UTC().Format(http.TimeFormat))
}

func setSecurityHeaders(w http.ResponseWriter) {
    w.Header().Set("X-Content-Type-Options", "nosniff")
    w.Header().Set("X-Frame-Options", "DENY")
    w.Header().Set("X-XSS-Protection", "1; mode=block")
    w.Header().Set("Strict-Transport-Security", 
                   "max-age=31536000; includeSubDomains")
}

func sendResponse(w http.ResponseWriter, statusCode int, data interface{}, 
                 errMsg string) {
    w.Header().Set("Content-Type", "application/json")
    setSecurityHeaders(w)
    w.WriteHeader(statusCode)
    
    response := APIResponse{
        Success:   statusCode >= 200 && statusCode < 300,
        Timestamp: time.Now(),
    }
    
    if errMsg != "" {
        response.Error = errMsg
    } else {
        response.Data = data
    }
    
    json.NewEncoder(w).Encode(response)
}

func statusCodesHandler(w http.ResponseWriter, r *http.Request) {
    code := r.URL.Query().Get("code")
    if code == "" {
        sendResponse(w, http.StatusBadRequest, nil, "status code required")
        return
    }
    
    statusCode, err := strconv.Atoi(code)
    if err != nil {
        sendResponse(w, http.StatusBadRequest, nil, "invalid status code")
        return
    }
    
    switch statusCode {
    case 200:
        sendResponse(w, http.StatusOK, 
                    map[string]string{"message": "Request successful"}, "")
    case 201:
        w.Header().Set("Location", "/resource/123")
        sendResponse(w, http.StatusCreated, 
                    map[string]interface{}{
                        "id": 123, 
                        "created": time.Now(),
                    }, "")
    case 204:
        w.WriteHeader(http.StatusNoContent)
    case 301:
        w.Header().Set("Location", "/new-endpoint")
        w.WriteHeader(http.StatusMovedPermanently)
        fmt.Fprint(w, "Resource moved permanently")
    case 304:
        w.WriteHeader(http.StatusNotModified)
    case 400:
        sendResponse(w, http.StatusBadRequest, nil, "bad request example")
    case 401:
        w.Header().Set("WWW-Authenticate", "Bearer")
        sendResponse(w, http.StatusUnauthorized, nil, "authentication required")
    case 403:
        sendResponse(w, http.StatusForbidden, nil, "access denied")
    case 404:
        sendResponse(w, http.StatusNotFound, nil, "resource not found")
    case 429:
        w.Header().Set("Retry-After", "60")
        w.Header().Set("X-RateLimit-Limit", "100")
        w.Header().Set("X-RateLimit-Remaining", "0")
        sendResponse(w, http.StatusTooManyRequests, nil, 
                    "rate limit exceeded")
    case 500:
        sendResponse(w, http.StatusInternalServerError, nil, 
                    "internal server error")
    case 503:
        w.Header().Set("Retry-After", "300")
        sendResponse(w, http.StatusServiceUnavailable, nil, 
                    "service temporarily unavailable")
    default:
        sendResponse(w, http.StatusBadRequest, nil, 
                    "unsupported status code")
    }
}

func contentTypesHandler(w http.ResponseWriter, r *http.Request) {
    format := r.URL.Query().Get("format")
    
    data := map[string]interface{}{
        "message": "Hello there!",
        "time":    time.Now(),
        "items":   []string{"apple", "banana", "cherry"},
    }
    
    switch format {
    case "json":
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(data)
    case "xml":
        w.Header().Set("Content-Type", "application/xml")
        fmt.Fprint(w, `<?xml version="1.0" encoding="UTF-8"?>
<response>
    <message>Hello there!</message>
    <items>
        <item>apple</item>
        <item>banana</item>
        <item>cherry</item>
    </items>
</response>`)
    case "text":
        w.Header().Set("Content-Type", "text/plain")
        fmt.Fprintf(w, "Message: %s\nTime: %v\n", 
                   data["message"], data["time"])
    case "html":
        w.Header().Set("Content-Type", "text/html")
        fmt.Fprint(w, `<!DOCTYPE html>
<html>
<head><title>Response</title></head>
<body>
    <h1>Hello there!</h1>
    <p>Current time: `, data["time"], `</p>
    <ul>
        <li>apple</li>
        <li>banana</li>
        <li>cherry</li>
    </ul>
</body>
</html>`)
    default:
        sendResponse(w, http.StatusBadRequest, nil, 
                    "supported formats: json, xml, text, html")
    }
}

func cacheHandler(w http.ResponseWriter, r *http.Request) {
    setCacheHeaders(w, 3600) // Cache for 1 hour
    
    // Check if client sent If-None-Match header
    if r.Header.Get("If-None-Match") != "" {
        w.WriteHeader(http.StatusNotModified)
        return
    }
    
    sendResponse(w, http.StatusOK, 
                map[string]interface{}{
                    "message": "This response is cacheable",
                    "cached":  true,
                }, "")
}

func main() {
    http.HandleFunc("/status", statusCodesHandler)
    http.HandleFunc("/content", contentTypesHandler)
    http.HandleFunc("/cache", cacheHandler)
    
    fmt.Println("Response examples server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  /status?code=200  - Status code examples")
    fmt.Println("  /content?format=json - Content type examples")
    fmt.Println("  /cache - Caching headers example")
    
    http.ListenAndServe(":8080", nil)
}
```

This example demonstrates comprehensive response handling including status  
codes, content types, caching headers, security headers, and location headers.  
Proper header management improves API usability, security, and performance  
while ensuring clients receive appropriate metadata about responses.

## HTTP middleware patterns

Middleware enables modular, reusable functionality that can be applied across  
multiple handlers. This example shows various middleware patterns for common  
web application concerns.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"
)

type ContextKey string

const (
    UserContextKey    ContextKey = "user"
    RequestIDKey      ContextKey = "requestID"
    StartTimeKey      ContextKey = "startTime"
)

type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Role     string `json:"role"`
}

// Logging middleware
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Add start time to context
        ctx := context.WithValue(r.Context(), StartTimeKey, start)
        r = r.WithContext(ctx)
        
        // Wrap response writer to capture status code
        ww := &responseWriter{ResponseWriter: w, statusCode: 200}
        
        next.ServeHTTP(ww, r)
        
        duration := time.Since(start)
        log.Printf("%s %s %d %v", r.Method, r.URL.Path, 
                  ww.statusCode, duration)
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

// Request ID middleware
func requestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestID := fmt.Sprintf("%d", time.Now().UnixNano())
        
        ctx := context.WithValue(r.Context(), RequestIDKey, requestID)
        r = r.WithContext(ctx)
        
        w.Header().Set("X-Request-ID", requestID)
        next.ServeHTTP(w, r)
    })
}

// Authentication middleware
func authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Authorization header required", 
                      http.StatusUnauthorized)
            return
        }
        
        // Simple token validation (in real apps, use proper JWT validation)
        token := strings.TrimPrefix(authHeader, "Bearer ")
        user, valid := validateToken(token)
        if !valid {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        
        ctx := context.WithValue(r.Context(), UserContextKey, user)
        r = r.WithContext(ctx)
        
        next.ServeHTTP(w, r)
    })
}

// Rate limiting middleware
func rateLimitMiddleware(requestsPerMinute int) func(http.Handler) http.Handler {
    clients := make(map[string][]time.Time)
    
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            clientIP := r.RemoteAddr
            now := time.Now()
            
            // Clean old requests
            if times, exists := clients[clientIP]; exists {
                var validTimes []time.Time
                for _, t := range times {
                    if now.Sub(t) < time.Minute {
                        validTimes = append(validTimes, t)
                    }
                }
                clients[clientIP] = validTimes
            }
            
            // Check rate limit
            if len(clients[clientIP]) >= requestsPerMinute {
                w.Header().Set("X-RateLimit-Limit", 
                              fmt.Sprintf("%d", requestsPerMinute))
                w.Header().Set("X-RateLimit-Remaining", "0")
                w.Header().Set("Retry-After", "60")
                http.Error(w, "Rate limit exceeded", 
                          http.StatusTooManyRequests)
                return
            }
            
            // Add current request
            clients[clientIP] = append(clients[clientIP], now)
            
            w.Header().Set("X-RateLimit-Limit", 
                          fmt.Sprintf("%d", requestsPerMinute))
            w.Header().Set("X-RateLimit-Remaining", 
                          fmt.Sprintf("%d", requestsPerMinute-len(clients[clientIP])))
            
            next.ServeHTTP(w, r)
        })
    }
}

// CORS middleware
func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", 
                      "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", 
                      "Content-Type, Authorization")
        w.Header().Set("Access-Control-Max-Age", "86400")
        
        if r.Method == http.MethodOptions {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next.ServeHTTP(w, r)
    })
}

// Recovery middleware
func recoveryMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("Panic recovered: %v", err)
                http.Error(w, "Internal server error", 
                          http.StatusInternalServerError)
            }
        }()
        
        next.ServeHTTP(w, r)
    })
}

func validateToken(token string) (User, bool) {
    // Simple token validation - in real apps use proper JWT validation
    validTokens := map[string]User{
        "user123": {ID: 1, Username: "alice", Role: "user"},
        "admin456": {ID: 2, Username: "admin", Role: "admin"},
    }
    
    user, valid := validTokens[token]
    return user, valid
}

func publicHandler(w http.ResponseWriter, r *http.Request) {
    requestID := r.Context().Value(RequestIDKey).(string)
    
    response := map[string]interface{}{
        "message":    "Hello there! This is a public endpoint.",
        "request_id": requestID,
        "timestamp":  time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func protectedHandler(w http.ResponseWriter, r *http.Request) {
    user := r.Context().Value(UserContextKey).(User)
    requestID := r.Context().Value(RequestIDKey).(string)
    
    response := map[string]interface{}{
        "message":    fmt.Sprintf("Hello %s! This is a protected endpoint.", 
                                user.Username),
        "user":       user,
        "request_id": requestID,
        "timestamp":  time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func panicHandler(w http.ResponseWriter, r *http.Request) {
    panic("Intentional panic for testing recovery middleware")
}

func chainMiddleware(h http.Handler, middlewares ...func(http.Handler) http.Handler) http.Handler {
    for i := len(middlewares) - 1; i >= 0; i-- {
        h = middlewares[i](h)
    }
    return h
}

func main() {
    mux := http.NewServeMux()
    
    // Public endpoint with basic middleware
    publicChain := chainMiddleware(
        http.HandlerFunc(publicHandler),
        loggingMiddleware,
        requestIDMiddleware,
        corsMiddleware,
        recoveryMiddleware,
        rateLimitMiddleware(10), // 10 requests per minute
    )
    
    // Protected endpoint with authentication
    protectedChain := chainMiddleware(
        http.HandlerFunc(protectedHandler),
        loggingMiddleware,
        requestIDMiddleware,
        corsMiddleware,
        recoveryMiddleware,
        authMiddleware,
        rateLimitMiddleware(5), // 5 requests per minute
    )
    
    // Panic endpoint for testing recovery
    panicChain := chainMiddleware(
        http.HandlerFunc(panicHandler),
        loggingMiddleware,
        requestIDMiddleware,
        recoveryMiddleware,
    )
    
    mux.Handle("/public", publicChain)
    mux.Handle("/protected", protectedChain)
    mux.Handle("/panic", panicChain)
    
    fmt.Println("Middleware example server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  GET /public - Public endpoint")
    fmt.Println("  GET /protected - Protected endpoint (requires auth)")
    fmt.Println("  GET /panic - Panic endpoint (for testing recovery)")
    fmt.Println("\nValid tokens: user123, admin456")
    
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

This comprehensive middleware example demonstrates logging, authentication,  
rate limiting, CORS, panic recovery, and request ID tracking. The middleware  
chain pattern allows for flexible composition of functionality while  
maintaining clean separation of concerns and reusable components.

## HTTP file upload handling

File uploads are common in web applications and require careful handling of  
multipart form data. This example demonstrates secure file upload processing  
with validation and storage.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "mime/multipart"
    "net/http"
    "os"
    "path/filepath"
    "strings"
    "time"
)

const (
    maxUploadSize = 10 << 20 // 10 MB
    uploadPath    = "./uploads"
)

type UploadResult struct {
    Filename string `json:"filename"`
    Size     int64  `json:"size"`
    Path     string `json:"path"`
    URL      string `json:"url"`
}

func init() {
    // Create uploads directory if it doesn't exist
    if err := os.MkdirAll(uploadPath, 0755); err != nil {
        log.Fatal("Failed to create upload directory:", err)
    }
}

func isValidFileType(filename string) bool {
    allowedExtensions := []string{".jpg", ".jpeg", ".png", ".gif", ".pdf", ".txt"}
    ext := strings.ToLower(filepath.Ext(filename))
    
    for _, allowed := range allowedExtensions {
        if ext == allowed {
            return true
        }
    }
    return false
}

func generateUniqueFilename(originalFilename string) string {
    ext := filepath.Ext(originalFilename)
    name := strings.TrimSuffix(originalFilename, ext)
    timestamp := time.Now().Unix()
    
    return fmt.Sprintf("%s_%d%s", name, timestamp, ext)
}

func saveUploadedFile(fileHeader *multipart.FileHeader) (*UploadResult, error) {
    // Validate file size
    if fileHeader.Size > maxUploadSize {
        return nil, fmt.Errorf("file size %d exceeds maximum allowed size %d",
                              fileHeader.Size, maxUploadSize)
    }

    // Validate file type
    if !isValidFileType(fileHeader.Filename) {
        return nil, fmt.Errorf("file type not allowed: %s", 
                              filepath.Ext(fileHeader.Filename))
    }

    // Open uploaded file
    src, err := fileHeader.Open()
    if err != nil {
        return nil, fmt.Errorf("failed to open uploaded file: %w", err)
    }
    defer src.Close()

    // Generate unique filename
    filename := generateUniqueFilename(fileHeader.Filename)
    filePath := filepath.Join(uploadPath, filename)

    // Create destination file
    dst, err := os.Create(filePath)
    if err != nil {
        return nil, fmt.Errorf("failed to create destination file: %w", err)
    }
    defer dst.Close()

    // Copy file content
    size, err := io.Copy(dst, src)
    if err != nil {
        return nil, fmt.Errorf("failed to save file: %w", err)
    }

    result := &UploadResult{
        Filename: filename,
        Size:     size,
        Path:     filePath,
        URL:      fmt.Sprintf("/uploads/%s", filename),
    }

    return result, nil
}

func uploadHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    // Parse multipart form
    if err := r.ParseMultipartForm(maxUploadSize); err != nil {
        http.Error(w, "Failed to parse multipart form", 
                  http.StatusBadRequest)
        return
    }

    // Get file from form
    file, fileHeader, err := r.FormFile("file")
    if err != nil {
        http.Error(w, "Failed to get file from form", 
                  http.StatusBadRequest)
        return
    }
    defer file.Close()

    // Save uploaded file
    result, err := saveUploadedFile(fileHeader)
    if err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // Get additional form fields
    description := r.FormValue("description")
    category := r.FormValue("category")

    w.Header().Set("Content-Type", "text/html")
    fmt.Fprintf(w, `
    <html>
    <head><title>Upload Successful</title></head>
    <body>
        <h2>File Upload Successful!</h2>
        <p><strong>Filename:</strong> %s</p>
        <p><strong>Size:</strong> %d bytes</p>
        <p><strong>Description:</strong> %s</p>
        <p><strong>Category:</strong> %s</p>
        <p><a href="/">Upload another file</a></p>
    </body>
    </html>`, result.Filename, result.Size, description, category)
}

func uploadFormHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <html>
    <head><title>File Upload</title></head>
    <body>
        <h2>File Upload Example</h2>
        <form action="/upload" method="post" enctype="multipart/form-data">
            <div>
                <label for="file">Choose file:</label><br>
                <input type="file" id="file" name="file" required>
            </div>
            <br>
            <div>
                <label for="description">Description:</label><br>
                <textarea id="description" name="description" rows="3" cols="50"></textarea>
            </div>
            <br>
            <div>
                <label for="category">Category:</label><br>
                <select id="category" name="category">
                    <option value="document">Document</option>
                    <option value="image">Image</option>
                    <option value="other">Other</option>
                </select>
            </div>
            <br>
            <input type="submit" value="Upload File">
        </form>
        
        <h3>Allowed file types:</h3>
        <ul>
            <li>Images: .jpg, .jpeg, .png, .gif</li>
            <li>Documents: .pdf, .txt</li>
        </ul>
        <p>Maximum file size: 10 MB</p>
    </body>
    </html>`)
}

func fileServerHandler(w http.ResponseWriter, r *http.Request) {
    filename := strings.TrimPrefix(r.URL.Path, "/uploads/")
    if filename == "" {
        http.Error(w, "Filename required", http.StatusBadRequest)
        return
    }

    filePath := filepath.Join(uploadPath, filename)
    
    // Check if file exists
    if _, err := os.Stat(filePath); os.IsNotExist(err) {
        http.Error(w, "File not found", http.StatusNotFound)
        return
    }

    // Serve the file
    http.ServeFile(w, r, filePath)
}

func main() {
    http.HandleFunc("/", uploadFormHandler)
    http.HandleFunc("/upload", uploadHandler)
    http.HandleFunc("/uploads/", fileServerHandler)

    fmt.Println("File upload server starting on :8080")
    fmt.Println("Open http://localhost:8080 to upload files")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This file upload example demonstrates secure file handling including size  
validation, file type checking, unique filename generation, and proper  
multipart form parsing. The code shows how to handle both file data and  
additional form fields while preventing common security issues.

## HTTP file download

File downloads require proper content headers and streaming for efficient  
handling of large files. This example shows different download scenarios  
and header configuration.

```go
package main

import (
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "path/filepath"
    "strconv"
    "strings"
    "time"
)

type FileInfo struct {
    Name    string
    Size    int64
    ModTime time.Time
    IsDir   bool
}

func getFileInfo(filePath string) (*FileInfo, error) {
    stat, err := os.Stat(filePath)
    if err != nil {
        return nil, err
    }
    
    return &FileInfo{
        Name:    stat.Name(),
        Size:    stat.Size(),
        ModTime: stat.ModTime(),
        IsDir:   stat.IsDir(),
    }, nil
}

func setDownloadHeaders(w http.ResponseWriter, filename string, size int64) {
    w.Header().Set("Content-Disposition", 
                   fmt.Sprintf("attachment; filename=\"%s\"", filename))
    w.Header().Set("Content-Type", "application/octet-stream")
    w.Header().Set("Content-Length", strconv.FormatInt(size, 10))
    w.Header().Set("Cache-Control", "no-cache, no-store, must-revalidate")
    w.Header().Set("Pragma", "no-cache")
    w.Header().Set("Expires", "0")
}

func setInlineHeaders(w http.ResponseWriter, filename string) {
    ext := strings.ToLower(filepath.Ext(filename))
    
    var contentType string
    switch ext {
    case ".pdf":
        contentType = "application/pdf"
    case ".jpg", ".jpeg":
        contentType = "image/jpeg"
    case ".png":
        contentType = "image/png"
    case ".gif":
        contentType = "image/gif"
    case ".txt":
        contentType = "text/plain"
    case ".html":
        contentType = "text/html"
    default:
        contentType = "application/octet-stream"
    }
    
    w.Header().Set("Content-Type", contentType)
    w.Header().Set("Content-Disposition", 
                   fmt.Sprintf("inline; filename=\"%s\"", filename))
}

func downloadHandler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    if filename == "" {
        http.Error(w, "File parameter required", http.StatusBadRequest)
        return
    }

    // Security: prevent directory traversal
    filename = filepath.Base(filename)
    filePath := filepath.Join("./downloads", filename)

    fileInfo, err := getFileInfo(filePath)
    if err != nil {
        if os.IsNotExist(err) {
            http.Error(w, "File not found", http.StatusNotFound)
        } else {
            http.Error(w, "Error accessing file", 
                      http.StatusInternalServerError)
        }
        return
    }

    if fileInfo.IsDir {
        http.Error(w, "Cannot download directory", http.StatusBadRequest)
        return
    }

    // Open file
    file, err := os.Open(filePath)
    if err != nil {
        http.Error(w, "Error opening file", http.StatusInternalServerError)
        return
    }
    defer file.Close()

    // Set download headers
    setDownloadHeaders(w, fileInfo.Name, fileInfo.Size)

    // Stream file to client
    _, err = io.Copy(w, file)
    if err != nil {
        log.Printf("Error streaming file: %v", err)
    }
}

func viewHandler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    if filename == "" {
        http.Error(w, "File parameter required", http.StatusBadRequest)
        return
    }

    filename = filepath.Base(filename)
    filePath := filepath.Join("./downloads", filename)

    fileInfo, err := getFileInfo(filePath)
    if err != nil {
        if os.IsNotExist(err) {
            http.Error(w, "File not found", http.StatusNotFound)
        } else {
            http.Error(w, "Error accessing file", 
                      http.StatusInternalServerError)
        }
        return
    }

    if fileInfo.IsDir {
        http.Error(w, "Cannot view directory", http.StatusBadRequest)
        return
    }

    // Set inline headers for browser viewing
    setInlineHeaders(w, fileInfo.Name)

    // Serve file for inline viewing
    http.ServeFile(w, r, filePath)
}

func rangeDownloadHandler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    if filename == "" {
        http.Error(w, "File parameter required", http.StatusBadRequest)
        return
    }

    filename = filepath.Base(filename)
    filePath := filepath.Join("./downloads", filename)

    file, err := os.Open(filePath)
    if err != nil {
        if os.IsNotExist(err) {
            http.Error(w, "File not found", http.StatusNotFound)
        } else {
            http.Error(w, "Error opening file", 
                      http.StatusInternalServerError)
        }
        return
    }
    defer file.Close()

    fileInfo, err := file.Stat()
    if err != nil {
        http.Error(w, "Error getting file info", 
                  http.StatusInternalServerError)
        return
    }

    // Enable range requests for resumable downloads
    w.Header().Set("Accept-Ranges", "bytes")
    
    http.ServeContent(w, r, fileInfo.Name(), fileInfo.ModTime(), file)
}

func listFilesHandler(w http.ResponseWriter, r *http.Request) {
    files, err := os.ReadDir("./downloads")
    if err != nil {
        http.Error(w, "Error reading directory", 
                  http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <html>
    <head><title>File Downloads</title></head>
    <body>
        <h2>Available Files</h2>
        <table border="1" cellpadding="5" cellspacing="0">
            <tr>
                <th>Filename</th>
                <th>Size</th>
                <th>Actions</th>
            </tr>`)

    for _, file := range files {
        if !file.IsDir() {
            info, _ := file.Info()
            fmt.Fprintf(w, `
            <tr>
                <td>%s</td>
                <td>%d bytes</td>
                <td>
                    <a href="/download?file=%s">Download</a> |
                    <a href="/view?file=%s" target="_blank">View</a> |
                    <a href="/range?file=%s">Range Download</a>
                </td>
            </tr>`, file.Name(), info.Size(), file.Name(), 
                    file.Name(), file.Name())
        }
    }

    fmt.Fprint(w, `
        </table>
        <br>
        <p><strong>Download:</strong> Downloads file as attachment</p>
        <p><strong>View:</strong> Opens file in browser (if supported)</p>
        <p><strong>Range Download:</strong> Supports resumable downloads</p>
    </body>
    </html>`)
}

func createSampleFiles() {
    os.MkdirAll("./downloads", 0755)
    
    // Create sample text file
    textContent := `Hello there! This is a sample text file.
This file demonstrates HTTP download functionality in Go.
You can download this file using different methods:
- Direct download as attachment
- View inline in browser
- Range/resumable download

Current time: ` + time.Now().Format(time.RFC3339)
    
    os.WriteFile("./downloads/sample.txt", []byte(textContent), 0644)
    
    // Create sample HTML file
    htmlContent := `<!DOCTYPE html>
<html>
<head><title>Sample HTML File</title></head>
<body>
    <h1>Hello there!</h1>
    <p>This is a sample HTML file for download testing.</p>
    <p>Generated at: ` + time.Now().Format(time.RFC3339) + `</p>
</body>
</html>`
    
    os.WriteFile("./downloads/sample.html", []byte(htmlContent), 0644)
}

func main() {
    createSampleFiles()
    
    http.HandleFunc("/", listFilesHandler)
    http.HandleFunc("/download", downloadHandler)
    http.HandleFunc("/view", viewHandler)
    http.HandleFunc("/range", rangeDownloadHandler)

    fmt.Println("File download server starting on :8080")
    fmt.Println("Open http://localhost:8080 to see available files")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This download example demonstrates different file serving methods including  
forced downloads with attachment headers, inline viewing with appropriate  
content types, and range requests for resumable downloads. The code shows  
proper security measures to prevent directory traversal attacks.

## JSON API server

JSON APIs are fundamental to modern web development. This example creates  
a comprehensive REST API with proper JSON handling, validation, and error  
responses.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type Product struct {
    ID          int       `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    Price       float64   `json:"price"`
    Category    string    `json:"category"`
    InStock     bool      `json:"in_stock"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}

type CreateProductRequest struct {
    Name        string  `json:"name"`
    Description string  `json:"description"`
    Price       float64 `json:"price"`
    Category    string  `json:"category"`
    InStock     bool    `json:"in_stock"`
}

type UpdateProductRequest struct {
    Name        *string  `json:"name,omitempty"`
    Description *string  `json:"description,omitempty"`
    Price       *float64 `json:"price,omitempty"`
    Category    *string  `json:"category,omitempty"`
    InStock     *bool    `json:"in_stock,omitempty"`
}

type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
    Meta    interface{} `json:"meta,omitempty"`
}

type PaginationMeta struct {
    Page       int `json:"page"`
    PerPage    int `json:"per_page"`
    Total      int `json:"total"`
    TotalPages int `json:"total_pages"`
}

var (
    products     = make(map[int]Product)
    nextID       = 1
    validCategories = []string{"electronics", "clothing", "books", "home"}
)

func init() {
    // Add sample data
    sampleProducts := []CreateProductRequest{
        {
            Name:        "Wireless Headphones",
            Description: "High-quality wireless headphones with noise cancellation",
            Price:       199.99,
            Category:    "electronics",
            InStock:     true,
        },
        {
            Name:        "Go Programming Book",
            Description: "Comprehensive guide to Go programming language",
            Price:       49.99,
            Category:    "books",
            InStock:     true,
        },
        {
            Name:        "Cotton T-Shirt",
            Description: "Comfortable cotton t-shirt in various colors",
            Price:       24.99,
            Category:    "clothing",
            InStock:     false,
        },
    }
    
    for _, req := range sampleProducts {
        createProduct(req)
    }
}

func sendJSONResponse(w http.ResponseWriter, statusCode int, success bool, 
                     data interface{}, errorMsg string, meta interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(statusCode)
    
    response := APIResponse{
        Success: success,
        Data:    data,
        Error:   errorMsg,
        Meta:    meta,
    }
    
    json.NewEncoder(w).Encode(response)
}

func validateCreateProduct(req CreateProductRequest) error {
    if req.Name == "" {
        return fmt.Errorf("name is required")
    }
    if req.Price < 0 {
        return fmt.Errorf("price must be non-negative")
    }
    if req.Category == "" {
        return fmt.Errorf("category is required")
    }
    
    validCategory := false
    for _, cat := range validCategories {
        if req.Category == cat {
            validCategory = true
            break
        }
    }
    if !validCategory {
        return fmt.Errorf("invalid category, must be one of: %s", 
                         strings.Join(validCategories, ", "))
    }
    
    return nil
}

func createProduct(req CreateProductRequest) Product {
    now := time.Now()
    product := Product{
        ID:          nextID,
        Name:        req.Name,
        Description: req.Description,
        Price:       req.Price,
        Category:    req.Category,
        InStock:     req.InStock,
        CreatedAt:   now,
        UpdatedAt:   now,
    }
    
    products[nextID] = product
    nextID++
    
    return product
}

func productsHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        handleGetProducts(w, r)
    case http.MethodPost:
        handleCreateProduct(w, r)
    default:
        sendJSONResponse(w, http.StatusMethodNotAllowed, false, nil, 
                        "Method not allowed", nil)
    }
}

func productHandler(w http.ResponseWriter, r *http.Request) {
    idStr := strings.TrimPrefix(r.URL.Path, "/products/")
    if idStr == "" {
        sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                        "Product ID required", nil)
        return
    }
    
    productID, err := strconv.Atoi(idStr)
    if err != nil {
        sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                        "Invalid product ID", nil)
        return
    }
    
    switch r.Method {
    case http.MethodGet:
        handleGetProduct(w, r, productID)
    case http.MethodPut:
        handleUpdateProduct(w, r, productID)
    case http.MethodDelete:
        handleDeleteProduct(w, r, productID)
    default:
        sendJSONResponse(w, http.StatusMethodNotAllowed, false, nil, 
                        "Method not allowed", nil)
    }
}

func handleGetProducts(w http.ResponseWriter, r *http.Request) {
    // Parse query parameters
    category := r.URL.Query().Get("category")
    inStockStr := r.URL.Query().Get("in_stock")
    pageStr := r.URL.Query().Get("page")
    perPageStr := r.URL.Query().Get("per_page")
    
    // Default pagination
    page := 1
    perPage := 10
    
    if pageStr != "" {
        if p, err := strconv.Atoi(pageStr); err == nil && p > 0 {
            page = p
        }
    }
    
    if perPageStr != "" {
        if pp, err := strconv.Atoi(perPageStr); err == nil && pp > 0 && pp <= 100 {
            perPage = pp
        }
    }
    
    // Filter products
    var filteredProducts []Product
    for _, product := range products {
        if category != "" && product.Category != category {
            continue
        }
        
        if inStockStr != "" {
            inStock, _ := strconv.ParseBool(inStockStr)
            if product.InStock != inStock {
                continue
            }
        }
        
        filteredProducts = append(filteredProducts, product)
    }
    
    // Paginate
    total := len(filteredProducts)
    totalPages := (total + perPage - 1) / perPage
    
    start := (page - 1) * perPage
    end := start + perPage
    
    if start >= total {
        filteredProducts = []Product{}
    } else {
        if end > total {
            end = total
        }
        filteredProducts = filteredProducts[start:end]
    }
    
    meta := PaginationMeta{
        Page:       page,
        PerPage:    perPage,
        Total:      total,
        TotalPages: totalPages,
    }
    
    sendJSONResponse(w, http.StatusOK, true, filteredProducts, "", meta)
}

func handleCreateProduct(w http.ResponseWriter, r *http.Request) {
    var req CreateProductRequest
    
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                        "Invalid JSON format", nil)
        return
    }
    
    if err := validateCreateProduct(req); err != nil {
        sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                        err.Error(), nil)
        return
    }
    
    product := createProduct(req)
    
    sendJSONResponse(w, http.StatusCreated, true, product, "", nil)
}

func handleGetProduct(w http.ResponseWriter, r *http.Request, productID int) {
    product, exists := products[productID]
    if !exists {
        sendJSONResponse(w, http.StatusNotFound, false, nil, 
                        "Product not found", nil)
        return
    }
    
    sendJSONResponse(w, http.StatusOK, true, product, "", nil)
}

func handleUpdateProduct(w http.ResponseWriter, r *http.Request, productID int) {
    product, exists := products[productID]
    if !exists {
        sendJSONResponse(w, http.StatusNotFound, false, nil, 
                        "Product not found", nil)
        return
    }
    
    var req UpdateProductRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                        "Invalid JSON format", nil)
        return
    }
    
    // Update fields if provided
    if req.Name != nil {
        product.Name = *req.Name
    }
    if req.Description != nil {
        product.Description = *req.Description
    }
    if req.Price != nil {
        if *req.Price < 0 {
            sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                            "Price must be non-negative", nil)
            return
        }
        product.Price = *req.Price
    }
    if req.Category != nil {
        validCategory := false
        for _, cat := range validCategories {
            if *req.Category == cat {
                validCategory = true
                break
            }
        }
        if !validCategory {
            sendJSONResponse(w, http.StatusBadRequest, false, nil, 
                            "Invalid category", nil)
            return
        }
        product.Category = *req.Category
    }
    if req.InStock != nil {
        product.InStock = *req.InStock
    }
    
    product.UpdatedAt = time.Now()
    products[productID] = product
    
    sendJSONResponse(w, http.StatusOK, true, product, "", nil)
}

func handleDeleteProduct(w http.ResponseWriter, r *http.Request, productID int) {
    _, exists := products[productID]
    if !exists {
        sendJSONResponse(w, http.StatusNotFound, false, nil, 
                        "Product not found", nil)
        return
    }
    
    delete(products, productID)
    
    w.WriteHeader(http.StatusNoContent)
}

func categoriesHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        sendJSONResponse(w, http.StatusMethodNotAllowed, false, nil, 
                        "Method not allowed", nil)
        return
    }
    
    sendJSONResponse(w, http.StatusOK, true, validCategories, "", nil)
}

func main() {
    http.HandleFunc("/products", productsHandler)
    http.HandleFunc("/products/", productHandler)
    http.HandleFunc("/categories", categoriesHandler)

    fmt.Println("JSON API server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  GET    /products          - List products (with pagination & filters)")
    fmt.Println("  POST   /products          - Create product")
    fmt.Println("  GET    /products/{id}     - Get specific product")
    fmt.Println("  PUT    /products/{id}     - Update product")
    fmt.Println("  DELETE /products/{id}     - Delete product")
    fmt.Println("  GET    /categories        - List valid categories")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This comprehensive JSON API example demonstrates RESTful design patterns,  
request validation, pagination, filtering, partial updates with PATCH  
semantics, proper error handling, and structured API responses. The code  
shows best practices for building maintainable and user-friendly APIs.

## HTTP authentication with JWT

JSON Web Tokens (JWT) provide a stateless authentication mechanism for APIs.  
This example demonstrates JWT creation, validation, and middleware  
implementation for secure endpoints.

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"
)

type JWTHeader struct {
    Algorithm string `json:"alg"`
    Type      string `json:"typ"`
}

type JWTClaims struct {
    UserID   int    `json:"user_id"`
    Username string `json:"username"`
    Role     string `json:"role"`
    IssuedAt int64  `json:"iat"`
    ExpiresAt int64 `json:"exp"`
}

type LoginRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type LoginResponse struct {
    Token     string    `json:"token"`
    ExpiresAt time.Time `json:"expires_at"`
    User      User      `json:"user"`
}

type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Role     string `json:"role"`
}

var (
    jwtSecret = []byte("your-secret-key-here-change-in-production")
    users = map[string]User{
        "alice": {ID: 1, Username: "alice", Role: "user"},
        "admin": {ID: 2, Username: "admin", Role: "admin"},
    }
    passwords = map[string]string{
        "alice": "password123",
        "admin": "admin456",
    }
)

func base64URLEncode(data []byte) string {
    return strings.TrimRight(base64.URLEncoding.EncodeToString(data), "=")
}

func base64URLDecode(data string) ([]byte, error) {
    // Add padding if needed
    switch len(data) % 4 {
    case 2:
        data += "=="
    case 3:
        data += "="
    }
    return base64.URLEncoding.DecodeString(data)
}

func createJWT(user User) (string, time.Time, error) {
    header := JWTHeader{
        Algorithm: "HS256",
        Type:      "JWT",
    }
    
    expiresAt := time.Now().Add(24 * time.Hour)
    claims := JWTClaims{
        UserID:    user.ID,
        Username:  user.Username,
        Role:      user.Role,
        IssuedAt:  time.Now().Unix(),
        ExpiresAt: expiresAt.Unix(),
    }
    
    headerJSON, err := json.Marshal(header)
    if err != nil {
        return "", time.Time{}, err
    }
    
    claimsJSON, err := json.Marshal(claims)
    if err != nil {
        return "", time.Time{}, err
    }
    
    headerEncoded := base64URLEncode(headerJSON)
    claimsEncoded := base64URLEncode(claimsJSON)
    
    payload := headerEncoded + "." + claimsEncoded
    
    // Create signature
    h := hmac.New(sha256.New, jwtSecret)
    h.Write([]byte(payload))
    signature := base64URLEncode(h.Sum(nil))
    
    token := payload + "." + signature
    
    return token, expiresAt, nil
}

func validateJWT(tokenString string) (*JWTClaims, error) {
    parts := strings.Split(tokenString, ".")
    if len(parts) != 3 {
        return nil, fmt.Errorf("invalid token format")
    }
    
    headerEncoded, claimsEncoded, signatureEncoded := parts[0], parts[1], parts[2]
    
    // Verify signature
    payload := headerEncoded + "." + claimsEncoded
    h := hmac.New(sha256.New, jwtSecret)
    h.Write([]byte(payload))
    expectedSignature := base64URLEncode(h.Sum(nil))
    
    if signatureEncoded != expectedSignature {
        return nil, fmt.Errorf("invalid token signature")
    }
    
    // Decode claims
    claimsJSON, err := base64URLDecode(claimsEncoded)
    if err != nil {
        return nil, fmt.Errorf("invalid claims encoding")
    }
    
    var claims JWTClaims
    if err := json.Unmarshal(claimsJSON, &claims); err != nil {
        return nil, fmt.Errorf("invalid claims format")
    }
    
    // Check expiration
    if time.Now().Unix() > claims.ExpiresAt {
        return nil, fmt.Errorf("token expired")
    }
    
    return &claims, nil
}

func jwtMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Authorization header required", 
                      http.StatusUnauthorized)
            return
        }
        
        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        if tokenString == authHeader {
            http.Error(w, "Bearer token required", http.StatusUnauthorized)
            return
        }
        
        claims, err := validateJWT(tokenString)
        if err != nil {
            http.Error(w, "Invalid token: "+err.Error(), 
                      http.StatusUnauthorized)
            return
        }
        
        // Add claims to request context
        r.Header.Set("X-User-ID", fmt.Sprintf("%d", claims.UserID))
        r.Header.Set("X-Username", claims.Username)
        r.Header.Set("X-Role", claims.Role)
        
        next.ServeHTTP(w, r)
    }
}

func adminMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        role := r.Header.Get("X-Role")
        if role != "admin" {
            http.Error(w, "Admin access required", http.StatusForbidden)
            return
        }
        
        next.ServeHTTP(w, r)
    }
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var req LoginRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    user, userExists := users[req.Username]
    password, passwordExists := passwords[req.Username]
    
    if !userExists || !passwordExists || password != req.Password {
        http.Error(w, "Invalid credentials", http.StatusUnauthorized)
        return
    }
    
    token, expiresAt, err := createJWT(user)
    if err != nil {
        http.Error(w, "Error creating token", http.StatusInternalServerError)
        return
    }
    
    response := LoginResponse{
        Token:     token,
        ExpiresAt: expiresAt,
        User:      user,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func profileHandler(w http.ResponseWriter, r *http.Request) {
    userID := r.Header.Get("X-User-ID")
    username := r.Header.Get("X-Username")
    role := r.Header.Get("X-Role")
    
    response := map[string]interface{}{
        "user_id":  userID,
        "username": username,
        "role":     role,
        "message":  "Hello there! This is your profile.",
        "accessed_at": time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func adminHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message": "Hello there, admin! You have access to this endpoint.",
        "users":   users,
        "accessed_at": time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func publicHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message": "Hello there! This is a public endpoint.",
        "info":    "No authentication required to access this endpoint.",
        "time":    time.Now(),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func main() {
    http.HandleFunc("/login", loginHandler)
    http.HandleFunc("/public", publicHandler)
    http.HandleFunc("/profile", jwtMiddleware(profileHandler))
    http.HandleFunc("/admin", jwtMiddleware(adminMiddleware(adminHandler)))

    fmt.Println("JWT Authentication server starting on :8080")
    fmt.Println("Endpoints:")
    fmt.Println("  POST /login    - Login with username/password")
    fmt.Println("  GET  /public   - Public endpoint (no auth)")
    fmt.Println("  GET  /profile  - Protected endpoint (requires JWT)")
    fmt.Println("  GET  /admin    - Admin endpoint (requires admin role)")
    fmt.Println("\nValid credentials:")
    fmt.Println("  alice / password123 (user role)")
    fmt.Println("  admin / admin456 (admin role)")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This JWT authentication example demonstrates token creation with HMAC  
signatures, token validation, role-based access control, and middleware  
composition. The implementation shows how to build secure authentication  
without external dependencies while maintaining proper security practices.

## HTTP cookies and sessions

Cookies and sessions are essential for maintaining state in web applications.  
This example demonstrates cookie handling, session management, and user  
authentication using server-side sessions.

```go
package main

import (
    "crypto/rand"
    "encoding/hex"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
    "time"
)

type Session struct {
    ID        string
    UserID    int
    Username  string
    CreatedAt time.Time
    LastSeen  time.Time
    Data      map[string]interface{}
}

type SessionStore struct {
    sessions map[string]*Session
    mutex    sync.RWMutex
}

type LoginRequest struct {
    Username string `json:"username"`
    Password string `json:"password"`
}

type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

var (
    sessionStore = &SessionStore{
        sessions: make(map[string]*Session),
    }
    users = map[string]User{
        "alice": {ID: 1, Username: "alice", Email: "alice@example.com"},
        "bob":   {ID: 2, Username: "bob", Email: "bob@example.com"},
    }
    passwords = map[string]string{
        "alice": "password123",
        "bob":   "secret456",
    }
)

func generateSessionID() string {
    bytes := make([]byte, 16)
    rand.Read(bytes)
    return hex.EncodeToString(bytes)
}

func (store *SessionStore) Create(userID int, username string) *Session {
    store.mutex.Lock()
    defer store.mutex.Unlock()
    
    sessionID := generateSessionID()
    session := &Session{
        ID:        sessionID,
        UserID:    userID,
        Username:  username,
        CreatedAt: time.Now(),
        LastSeen:  time.Now(),
        Data:      make(map[string]interface{}),
    }
    
    store.sessions[sessionID] = session
    return session
}

func (store *SessionStore) Get(sessionID string) *Session {
    store.mutex.RLock()
    defer store.mutex.RUnlock()
    
    session, exists := store.sessions[sessionID]
    if !exists {
        return nil
    }
    
    // Check if session is expired (24 hours)
    if time.Since(session.LastSeen) > 24*time.Hour {
        return nil
    }
    
    return session
}

func (store *SessionStore) Update(sessionID string) {
    store.mutex.Lock()
    defer store.mutex.Unlock()
    
    if session, exists := store.sessions[sessionID]; exists {
        session.LastSeen = time.Now()
    }
}

func (store *SessionStore) Delete(sessionID string) {
    store.mutex.Lock()
    defer store.mutex.Unlock()
    
    delete(store.sessions, sessionID)
}

func (store *SessionStore) SetData(sessionID string, key string, value interface{}) {
    store.mutex.Lock()
    defer store.mutex.Unlock()
    
    if session, exists := store.sessions[sessionID]; exists {
        session.Data[key] = value
    }
}

func (store *SessionStore) GetData(sessionID string, key string) interface{} {
    store.mutex.RLock()
    defer store.mutex.RUnlock()
    
    if session, exists := store.sessions[sessionID]; exists {
        return session.Data[key]
    }
    return nil
}

func setSessionCookie(w http.ResponseWriter, sessionID string) {
    cookie := &http.Cookie{
        Name:     "session_id",
        Value:    sessionID,
        Path:     "/",
        MaxAge:   86400, // 24 hours
        HttpOnly: true,
        Secure:   false, // Set to true in production with HTTPS
        SameSite: http.SameSiteLaxMode,
    }
    http.SetCookie(w, cookie)
}

func clearSessionCookie(w http.ResponseWriter) {
    cookie := &http.Cookie{
        Name:     "session_id",
        Value:    "",
        Path:     "/",
        MaxAge:   -1,
        HttpOnly: true,
    }
    http.SetCookie(w, cookie)
}

func getSessionFromRequest(r *http.Request) *Session {
    cookie, err := r.Cookie("session_id")
    if err != nil {
        return nil
    }
    
    session := sessionStore.Get(cookie.Value)
    if session != nil {
        sessionStore.Update(cookie.Value)
    }
    
    return session
}

func requireAuth(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        session := getSessionFromRequest(r)
        if session == nil {
            http.Error(w, "Authentication required", http.StatusUnauthorized)
            return
        }
        
        // Add session info to request headers for handlers
        r.Header.Set("X-Session-ID", session.ID)
        r.Header.Set("X-User-ID", fmt.Sprintf("%d", session.UserID))
        r.Header.Set("X-Username", session.Username)
        
        next.ServeHTTP(w, r)
    }
}

func loginHandler(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        // Return login form
        w.Header().Set("Content-Type", "text/html")
        fmt.Fprint(w, `
        <html>
        <head><title>Login</title></head>
        <body>
            <h2>Login</h2>
            <form method="post">
                <div>
                    <label>Username:</label><br>
                    <input type="text" name="username" required>
                </div>
                <br>
                <div>
                    <label>Password:</label><br>
                    <input type="password" name="password" required>
                </div>
                <br>
                <input type="submit" value="Login">
            </form>
            <p>Valid users: alice/password123, bob/secret456</p>
        </body>
        </html>`)
        
    case http.MethodPost:
        contentType := r.Header.Get("Content-Type")
        
        var username, password string
        
        if contentType == "application/json" {
            var req LoginRequest
            if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
                http.Error(w, "Invalid JSON", http.StatusBadRequest)
                return
            }
            username, password = req.Username, req.Password
        } else {
            // Form data
            r.ParseForm()
            username = r.FormValue("username")
            password = r.FormValue("password")
        }
        
        user, userExists := users[username]
        userPassword, passwordExists := passwords[username]
        
        if !userExists || !passwordExists || userPassword != password {
            http.Error(w, "Invalid credentials", http.StatusUnauthorized)
            return
        }
        
        session := sessionStore.Create(user.ID, user.Username)
        setSessionCookie(w, session.ID)
        
        if contentType == "application/json" {
            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(map[string]interface{}{
                "message":    "Login successful",
                "session_id": session.ID,
                "user":       user,
            })
        } else {
            http.Redirect(w, r, "/dashboard", http.StatusSeeOther)
        }
        
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func logoutHandler(w http.ResponseWriter, r *http.Request) {
    session := getSessionFromRequest(r)
    if session != nil {
        sessionStore.Delete(session.ID)
    }
    
    clearSessionCookie(w)
    
    if r.Header.Get("Accept") == "application/json" {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]string{
            "message": "Logout successful",
        })
    } else {
        http.Redirect(w, r, "/login", http.StatusSeeOther)
    }
}

func dashboardHandler(w http.ResponseWriter, r *http.Request) {
    sessionID := r.Header.Get("X-Session-ID")
    username := r.Header.Get("X-Username")
    
    // Increment page visit counter
    visits := sessionStore.GetData(sessionID, "dashboard_visits")
    if visits == nil {
        visits = 0
    }
    newVisits := visits.(int) + 1
    sessionStore.SetData(sessionID, "dashboard_visits", newVisits)
    
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprintf(w, `
    <html>
    <head><title>Dashboard</title></head>
    <body>
        <h2>Hello there, %s!</h2>
        <p>Welcome to your dashboard.</p>
        <p>You have visited this page %d time(s) in this session.</p>
        <p>Session ID: %s</p>
        <p><a href="/profile">View Profile</a> | <a href="/logout">Logout</a></p>
    </body>
    </html>`, username, newVisits, sessionID)
}

func profileHandler(w http.ResponseWriter, r *http.Request) {
    username := r.Header.Get("X-Username")
    user := users[username]
    
    if r.Header.Get("Accept") == "application/json" {
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]interface{}{
            "user":    user,
            "message": "Hello there! This is your profile.",
        })
    } else {
        w.Header().Set("Content-Type", "text/html")
        fmt.Fprintf(w, `
        <html>
        <head><title>Profile</title></head>
        <body>
            <h2>User Profile</h2>
            <p><strong>Username:</strong> %s</p>
            <p><strong>Email:</strong> %s</p>
            <p><strong>User ID:</strong> %d</p>
            <p><a href="/dashboard">Back to Dashboard</a></p>
        </body>
        </html>`, user.Username, user.Email, user.ID)
    }
}

func setCookieHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    r.ParseForm()
    name := r.FormValue("name")
    value := r.FormValue("value")
    
    if name == "" || value == "" {
        http.Error(w, "Name and value required", http.StatusBadRequest)
        return
    }
    
    cookie := &http.Cookie{
        Name:  name,
        Value: value,
        Path:  "/",
        MaxAge: 3600, // 1 hour
    }
    http.SetCookie(w, cookie)
    
    fmt.Fprintf(w, "Cookie '%s' set to '%s'", name, value)
}

func getCookieHandler(w http.ResponseWriter, r *http.Request) {
    cookies := r.Cookies()
    
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <html>
    <head><title>Cookies</title></head>
    <body>
        <h2>Current Cookies</h2>
        <table border="1" cellpadding="5">
            <tr><th>Name</th><th>Value</th></tr>`)
    
    for _, cookie := range cookies {
        fmt.Fprintf(w, "<tr><td>%s</td><td>%s</td></tr>", 
                   cookie.Name, cookie.Value)
    }
    
    fmt.Fprint(w, `
        </table>
        <br>
        <h3>Set New Cookie</h3>
        <form method="post" action="/set-cookie">
            <input type="text" name="name" placeholder="Cookie name" required>
            <input type="text" name="value" placeholder="Cookie value" required>
            <input type="submit" value="Set Cookie">
        </form>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/login", loginHandler)
    http.HandleFunc("/logout", logoutHandler)
    http.HandleFunc("/dashboard", requireAuth(dashboardHandler))
    http.HandleFunc("/profile", requireAuth(profileHandler))
    http.HandleFunc("/cookies", getCookieHandler)
    http.HandleFunc("/set-cookie", setCookieHandler)
    
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        http.Redirect(w, r, "/login", http.StatusSeeOther)
    })

    fmt.Println("Cookie/Session server starting on :8080")
    fmt.Println("Visit http://localhost:8080 to test the application")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This session management example demonstrates cookie handling, server-side  
session storage, authentication flows, and session data persistence. The  
implementation shows both HTML form-based and JSON API authentication  
patterns with proper security considerations for cookie attributes.

## WebSocket server

WebSockets enable real-time, bidirectional communication between clients  
and servers. This example demonstrates WebSocket upgrading, message  
handling, and broadcasting for chat-like functionality.

```go
package main

import (
    "crypto/sha1"
    "encoding/base64"
    "encoding/binary"
    "fmt"
    "log"
    "net/http"
    "strings"
    "sync"
    "time"
)

const websocketMagicString = "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"

type WebSocketOpcode byte

const (
    OpcodeContinuation WebSocketOpcode = 0x0
    OpcodeText         WebSocketOpcode = 0x1
    OpcodeBinary       WebSocketOpcode = 0x2
    OpcodeClose        WebSocketOpcode = 0x8
    OpcodePing         WebSocketOpcode = 0x9
    OpcodePong         WebSocketOpcode = 0xA
)

type WebSocketFrame struct {
    Fin           bool
    Opcode        WebSocketOpcode
    Masked        bool
    PayloadLength uint64
    MaskingKey    [4]byte
    Payload       []byte
}

type Client struct {
    ID   string
    Conn http.ResponseWriter
    Hub  *Hub
}

type Hub struct {
    clients    map[*Client]bool
    broadcast  chan []byte
    register   chan *Client
    unregister chan *Client
    mutex      sync.RWMutex
}

type Message struct {
    Type      string    `json:"type"`
    ClientID  string    `json:"client_id,omitempty"`
    Content   string    `json:"content,omitempty"`
    Timestamp time.Time `json:"timestamp"`
    Clients   int       `json:"clients,omitempty"`
}

func NewHub() *Hub {
    return &Hub{
        clients:    make(map[*Client]bool),
        broadcast:  make(chan []byte),
        register:   make(chan *Client),
        unregister: make(chan *Client),
    }
}

func (h *Hub) Run() {
    for {
        select {
        case client := <-h.register:
            h.mutex.Lock()
            h.clients[client] = true
            h.mutex.Unlock()
            
            log.Printf("Client %s connected. Total clients: %d", 
                      client.ID, len(h.clients))
            
            // Notify all clients about new connection
            welcomeMsg := fmt.Sprintf(`{
                "type": "user_joined",
                "client_id": "%s",
                "content": "User %s joined the chat",
                "timestamp": "%s",
                "clients": %d
            }`, client.ID, client.ID, time.Now().Format(time.RFC3339), 
                len(h.clients))
            
            h.broadcast <- []byte(welcomeMsg)

        case client := <-h.unregister:
            h.mutex.Lock()
            if _, ok := h.clients[client]; ok {
                delete(h.clients, client)
            }
            h.mutex.Unlock()
            
            log.Printf("Client %s disconnected. Total clients: %d", 
                      client.ID, len(h.clients))
            
            // Notify remaining clients about disconnection
            if len(h.clients) > 0 {
                leaveMsg := fmt.Sprintf(`{
                    "type": "user_left",
                    "client_id": "%s",
                    "content": "User %s left the chat",
                    "timestamp": "%s",
                    "clients": %d
                }`, client.ID, client.ID, time.Now().Format(time.RFC3339), 
                    len(h.clients))
                
                h.broadcast <- []byte(leaveMsg)
            }

        case message := <-h.broadcast:
            h.mutex.RLock()
            for client := range h.clients {
                if err := writeWebSocketMessage(client.Conn, message); err != nil {
                    log.Printf("Error sending message to client %s: %v", 
                              client.ID, err)
                    delete(h.clients, client)
                }
            }
            h.mutex.RUnlock()
        }
    }
}

func generateWebSocketAccept(key string) string {
    h := sha1.New()
    h.Write([]byte(key))
    h.Write([]byte(websocketMagicString))
    return base64.StdEncoding.EncodeToString(h.Sum(nil))
}

func upgradeToWebSocket(w http.ResponseWriter, r *http.Request) error {
    if r.Header.Get("Upgrade") != "websocket" {
        return fmt.Errorf("missing or invalid upgrade header")
    }
    
    if r.Header.Get("Connection") != "Upgrade" {
        return fmt.Errorf("missing or invalid connection header")
    }
    
    if r.Header.Get("Sec-WebSocket-Version") != "13" {
        return fmt.Errorf("unsupported websocket version")
    }
    
    key := r.Header.Get("Sec-WebSocket-Key")
    if key == "" {
        return fmt.Errorf("missing websocket key")
    }
    
    acceptKey := generateWebSocketAccept(key)
    
    w.Header().Set("Upgrade", "websocket")
    w.Header().Set("Connection", "Upgrade")
    w.Header().Set("Sec-WebSocket-Accept", acceptKey)
    w.WriteHeader(http.StatusSwitchingProtocols)
    
    return nil
}

func writeWebSocketMessage(w http.ResponseWriter, data []byte) error {
    frame := make([]byte, 2+len(data))
    frame[0] = 0x81 // FIN=1, Opcode=1 (text)
    
    if len(data) < 126 {
        frame[1] = byte(len(data))
        copy(frame[2:], data)
    } else if len(data) < 65536 {
        frame[1] = 126
        binary.BigEndian.PutUint16(frame[2:4], uint16(len(data)))
        copy(frame[4:], data)
        frame = append(frame[:4], data...)
    } else {
        // For larger payloads (not implemented in this example)
        return fmt.Errorf("payload too large")
    }
    
    if flusher, ok := w.(http.Flusher); ok {
        w.Write(frame)
        flusher.Flush()
        return nil
    }
    
    return fmt.Errorf("response writer doesn't support flushing")
}

func readWebSocketFrame(r *http.Request) (*WebSocketFrame, error) {
    // This is a simplified implementation
    // In production, use gorilla/websocket or similar library
    return nil, fmt.Errorf("frame reading not implemented in this example")
}

var hub = NewHub()

func websocketHandler(w http.ResponseWriter, r *http.Request) {
    if err := upgradeToWebSocket(w, r); err != nil {
        http.Error(w, "Failed to upgrade to WebSocket: "+err.Error(), 
                  http.StatusBadRequest)
        return
    }
    
    clientID := fmt.Sprintf("user_%d", time.Now().UnixNano()%10000)
    client := &Client{
        ID:   clientID,
        Conn: w,
        Hub:  hub,
    }
    
    hub.register <- client
    
    // Send welcome message to the new client
    welcomeMsg := fmt.Sprintf(`{
        "type": "welcome",
        "client_id": "%s",
        "content": "Hello there! Welcome to the WebSocket chat.",
        "timestamp": "%s"
    }`, clientID, time.Now().Format(time.RFC3339))
    
    writeWebSocketMessage(w, []byte(welcomeMsg))
    
    // Keep connection alive
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            // Send ping frame
            pingFrame := []byte{0x89, 0x00} // Ping frame with no payload
            if flusher, ok := w.(http.Flusher); ok {
                w.Write(pingFrame)
                flusher.Flush()
            }
            
        case <-time.After(60 * time.Second):
            // Timeout - close connection
            hub.unregister <- client
            return
        }
    }
}

func chatPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>WebSocket Chat</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            #messages { border: 1px solid #ccc; height: 400px; 
                       overflow-y: scroll; padding: 10px; margin-bottom: 10px; }
            #messageInput { width: 300px; padding: 5px; }
            #sendButton { padding: 5px 10px; }
            .message { margin: 5px 0; }
            .system { color: #666; font-style: italic; }
            .user { color: #333; }
            .error { color: red; }
        </style>
    </head>
    <body>
        <h2>WebSocket Chat Example</h2>
        <div id="status">Connecting...</div>
        <div id="messages"></div>
        <input type="text" id="messageInput" placeholder="Type your message...">
        <button id="sendButton">Send</button>
        
        <script>
            const ws = new WebSocket('ws://localhost:8080/ws');
            const messages = document.getElementById('messages');
            const status = document.getElementById('status');
            const messageInput = document.getElementById('messageInput');
            const sendButton = document.getElementById('sendButton');
            
            ws.onopen = function() {
                status.textContent = 'Connected';
                status.style.color = 'green';
            };
            
            ws.onmessage = function(event) {
                try {
                    const message = JSON.parse(event.data);
                    const div = document.createElement('div');
                    div.className = 'message';
                    
                    let content = '';
                    if (message.type === 'welcome' || message.type === 'user_joined' || 
                        message.type === 'user_left') {
                        div.className += ' system';
                        content = message.content;
                    } else {
                        div.className += ' user';
                        content = '[' + message.client_id + '] ' + message.content;
                    }
                    
                    div.textContent = new Date(message.timestamp).toLocaleTimeString() + 
                                     ' - ' + content;
                    messages.appendChild(div);
                    messages.scrollTop = messages.scrollHeight;
                } catch (e) {
                    console.error('Error parsing message:', e);
                }
            };
            
            ws.onclose = function() {
                status.textContent = 'Disconnected';
                status.style.color = 'red';
            };
            
            ws.onerror = function(error) {
                status.textContent = 'Connection error';
                status.style.color = 'red';
                console.error('WebSocket error:', error);
            };
            
            function sendMessage() {
                const message = messageInput.value.trim();
                if (message && ws.readyState === WebSocket.OPEN) {
                    const msg = {
                        type: 'chat',
                        content: message,
                        timestamp: new Date().toISOString()
                    };
                    ws.send(JSON.stringify(msg));
                    messageInput.value = '';
                }
            }
            
            sendButton.onclick = sendMessage;
            messageInput.onkeypress = function(e) {
                if (e.key === 'Enter') {
                    sendMessage();
                }
            };
        </script>
    </body>
    </html>`)
}

func main() {
    go hub.Run()
    
    http.HandleFunc("/", chatPageHandler)
    http.HandleFunc("/ws", websocketHandler)

    fmt.Println("WebSocket server starting on :8080")
    fmt.Println("Open http://localhost:8080 to join the chat")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This WebSocket example demonstrates protocol upgrading, frame handling,  
client management, and real-time messaging. The implementation shows the  
WebSocket handshake process and basic frame construction, though production  
applications should use established libraries like gorilla/websocket for  
complete protocol compliance and security.

## HTTP/2 server

HTTP/2 provides improved performance through multiplexing, server push, and  
header compression. This example demonstrates HTTP/2 features and backwards  
compatibility with HTTP/1.1.

```go
package main

import (
    "crypto/tls"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"
)

func http2Handler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    
    protocol := "HTTP/1.1"
    if r.ProtoMajor == 2 {
        protocol = "HTTP/2"
    }
    
    fmt.Fprintf(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>%s Example</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .protocol { color: %s; font-weight: bold; }
            .info { background: #f5f5f5; padding: 15px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <h1>Hello there!</h1>
        <p>This response is served over <span class="protocol">%s</span></p>
        
        <div class="info">
            <h3>Request Information:</h3>
            <p><strong>Method:</strong> %s</p>
            <p><strong>URL:</strong> %s</p>
            <p><strong>Protocol:</strong> %s %d.%d</p>
            <p><strong>Remote Address:</strong> %s</p>
            <p><strong>User Agent:</strong> %s</p>
        </div>
        
        <h3>HTTP/2 Features:</h3>
        <ul>
            <li>Multiplexing - Multiple streams over single connection</li>
            <li>Header compression with HPACK</li>
            <li>Server push for improved performance</li>
            <li>Binary protocol instead of text</li>
            <li>Stream prioritization</li>
        </ul>
        
        <p><a href="/push-demo">Try Server Push Demo</a></p>
        <p><a href="/stream-demo">Try Streaming Demo</a></p>
    </body>
    </html>`, 
    protocol,
    map[bool]string{true: "green", false: "blue"}[r.ProtoMajor == 2],
    protocol,
    r.Method,
    r.URL.String(),
    r.Proto,
    r.ProtoMajor,
    r.ProtoMinor,
    r.RemoteAddr,
    r.UserAgent())
}

func serverPushHandler(w http.ResponseWriter, r *http.Request) {
    // Check if client supports HTTP/2 and server push
    if pusher, ok := w.(http.Pusher); ok && r.ProtoMajor == 2 {
        // Push CSS and JavaScript resources
        pushTargets := []string{
            "/static/style.css",
            "/static/script.js",
        }
        
        for _, target := range pushTargets {
            if err := pusher.Push(target, nil); err != nil {
                log.Printf("Failed to push %s: %v", target, err)
            }
        }
    }
    
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Server Push Demo</title>
        <link rel="stylesheet" href="/static/style.css">
    </head>
    <body>
        <h1>Server Push Demo</h1>
        <p>If you're using HTTP/2, the CSS and JavaScript files were pushed by the server!</p>
        <div id="content">Loading...</div>
        <script src="/static/script.js"></script>
    </body>
    </html>`)
}

func streamingHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    w.Header().Set("Cache-Control", "no-cache")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Streaming Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .message { margin: 10px 0; padding: 10px; background: #f0f0f0; }
        </style>
    </head>
    <body>
        <h1>HTTP/2 Streaming Demo</h1>
        <p>Messages will appear one by one:</p>
    `)
    flusher.Flush()
    
    messages := []string{
        "Hello there! Starting the streaming demo...",
        "HTTP/2 allows efficient multiplexing of streams.",
        "Each message is sent as soon as it's available.",
        "This reduces perceived latency for users.",
        "Streaming works great with HTTP/2's binary framing.",
        "Demo completed successfully!",
    }
    
    for i, message := range messages {
        time.Sleep(1 * time.Second)
        
        fmt.Fprintf(w, `
        <div class="message">
            <strong>Message %d:</strong> %s
        </div>`, i+1, message)
        flusher.Flush()
    }
    
    fmt.Fprint(w, `
        <p><a href="/">Back to Home</a></p>
    </body>
    </html>`)
    flusher.Flush()
}

func staticFileHandler(w http.ResponseWriter, r *http.Request) {
    filename := strings.TrimPrefix(r.URL.Path, "/static/")
    
    switch filename {
    case "style.css":
        w.Header().Set("Content-Type", "text/css")
        fmt.Fprint(w, `
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
            color: #333;
        }
        h1 { color: #2c3e50; }
        #content {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }`)
        
    case "script.js":
        w.Header().Set("Content-Type", "application/javascript")
        fmt.Fprint(w, `
        document.addEventListener('DOMContentLoaded', function() {
            const content = document.getElementById('content');
            content.innerHTML = 'Hello there! This content was loaded via pushed JavaScript!';
            content.style.color = 'green';
        });`)
        
    default:
        http.NotFound(w, r)
    }
}

func protocolInfoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    info := map[string]interface{}{
        "protocol":        r.Proto,
        "protocol_major":  r.ProtoMajor,
        "protocol_minor":  r.ProtoMinor,
        "method":          r.Method,
        "url":             r.URL.String(),
        "remote_addr":     r.RemoteAddr,
        "user_agent":      r.UserAgent(),
        "headers":         r.Header,
        "tls_info":        nil,
        "http2_supported": r.ProtoMajor == 2,
        "server_push_supported": false,
    }
    
    // Check for TLS information
    if r.TLS != nil {
        info["tls_info"] = map[string]interface{}{
            "version":                r.TLS.Version,
            "cipher_suite":          r.TLS.CipherSuite,
            "server_name":           r.TLS.ServerName,
            "negotiated_protocol":   r.TLS.NegotiatedProtocol,
            "negotiated_protocol_is_mutual": r.TLS.NegotiatedProtocolIsMutual,
        }
    }
    
    // Check for server push support
    if _, ok := w.(http.Pusher); ok {
        info["server_push_supported"] = true
    }
    
    fmt.Fprintf(w, `{
    "message": "Hello there! Here's your protocol information.",
    "timestamp": "%s",
    "protocol_info": %s
}`, time.Now().Format(time.RFC3339), toJSON(info))
}

func toJSON(v interface{}) string {
    // Simple JSON conversion for demo purposes
    switch val := v.(type) {
    case map[string]interface{}:
        result := "{"
        first := true
        for k, v := range val {
            if !first {
                result += ","
            }
            result += fmt.Sprintf(`"%s":%s`, k, toJSON(v))
            first = false
        }
        result += "}"
        return result
    case string:
        return fmt.Sprintf(`"%s"`, val)
    case int:
        return fmt.Sprintf("%d", val)
    case bool:
        return fmt.Sprintf("%t", val)
    default:
        return `null`
    }
}

func createSelfSignedCert() {
    // This would create a self-signed certificate for testing
    // In production, use proper certificates from a CA
    log.Println("Using self-signed certificate for HTTPS/HTTP2")
}

func main() {
    http.HandleFunc("/", http2Handler)
    http.HandleFunc("/push-demo", serverPushHandler)
    http.HandleFunc("/stream-demo", streamingHandler)
    http.HandleFunc("/static/", staticFileHandler)
    http.HandleFunc("/api/protocol", protocolInfoHandler)
    
    // Create TLS configuration for HTTP/2
    tlsConfig := &tls.Config{
        NextProtos: []string{"h2", "http/1.1"},
    }
    
    server := &http.Server{
        Addr:      ":8443",
        TLSConfig: tlsConfig,
    }
    
    fmt.Println("HTTP/2 server starting on :8443 (HTTPS)")
    fmt.Println("Visit https://localhost:8443 to test HTTP/2 features")
    fmt.Println("Note: You may need to accept the self-signed certificate")
    
    // Also start HTTP/1.1 server for comparison
    go func() {
        fmt.Println("HTTP/1.1 server starting on :8080")
        http.ListenAndServe(":8080", nil)
    }()
    
    // Start HTTPS server with HTTP/2 support
    log.Fatal(server.ListenAndServeTLS("server.crt", "server.key"))
}
```

This HTTP/2 example demonstrates protocol detection, server push capabilities,  
streaming responses, and performance improvements over HTTP/1.1. The code  
shows how to leverage HTTP/2 features while maintaining backwards  
compatibility with HTTP/1.1 clients.

## HTTP client with connection pooling

Connection pooling improves performance by reusing HTTP connections across  
multiple requests. This example demonstrates custom transport configuration,  
connection management, and performance optimization.

```go
package main

import (
    "context"
    "crypto/tls"
    "fmt"
    "io"
    "log"
    "net"
    "net/http"
    "sync"
    "time"
)

type HTTPClientPool struct {
    client     *http.Client
    stats      *ClientStats
    config     *PoolConfig
}

type PoolConfig struct {
    MaxIdleConns        int
    MaxIdleConnsPerHost int
    IdleConnTimeout     time.Duration
    DialTimeout         time.Duration
    TLSHandshakeTimeout time.Duration
    ResponseHeaderTimeout time.Duration
    ExpectContinueTimeout time.Duration
}

type ClientStats struct {
    RequestCount    int64
    SuccessCount    int64
    ErrorCount      int64
    TotalBytes      int64
    ConnectionReuse int64
    mutex           sync.RWMutex
}

func (s *ClientStats) IncrementRequest() {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.RequestCount++
}

func (s *ClientStats) IncrementSuccess(bytes int64) {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.SuccessCount++
    s.TotalBytes += bytes
}

func (s *ClientStats) IncrementError() {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.ErrorCount++
}

func (s *ClientStats) IncrementReuse() {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.ConnectionReuse++
}

func (s *ClientStats) GetStats() (int64, int64, int64, int64, int64) {
    s.mutex.RLock()
    defer s.mutex.RUnlock()
    return s.RequestCount, s.SuccessCount, s.ErrorCount, 
           s.TotalBytes, s.ConnectionReuse
}

func NewHTTPClientPool(config *PoolConfig) *HTTPClientPool {
    if config == nil {
        config = &PoolConfig{
            MaxIdleConns:          100,
            MaxIdleConnsPerHost:   10,
            IdleConnTimeout:       90 * time.Second,
            DialTimeout:           30 * time.Second,
            TLSHandshakeTimeout:   10 * time.Second,
            ResponseHeaderTimeout: 30 * time.Second,
            ExpectContinueTimeout: 1 * time.Second,
        }
    }
    
    // Custom dialer with timeout and keep-alive
    dialer := &net.Dialer{
        Timeout:   config.DialTimeout,
        KeepAlive: 30 * time.Second,
    }
    
    // Custom transport with connection pooling
    transport := &http.Transport{
        DialContext: func(ctx context.Context, network, addr string) (net.Conn, error) {
            return dialer.DialContext(ctx, network, addr)
        },
        MaxIdleConns:          config.MaxIdleConns,
        MaxIdleConnsPerHost:   config.MaxIdleConnsPerHost,
        IdleConnTimeout:       config.IdleConnTimeout,
        TLSHandshakeTimeout:   config.TLSHandshakeTimeout,
        ResponseHeaderTimeout: config.ResponseHeaderTimeout,
        ExpectContinueTimeout: config.ExpectContinueTimeout,
        
        // Enable HTTP/2
        ForceAttemptHTTP2: true,
        
        // TLS configuration
        TLSClientConfig: &tls.Config{
            InsecureSkipVerify: false,
        },
        
        // Disable compression for clearer byte counting
        DisableCompression: true,
    }
    
    client := &http.Client{
        Transport: transport,
        Timeout:   60 * time.Second,
    }
    
    return &HTTPClientPool{
        client: client,
        stats:  &ClientStats{},
        config: config,
    }
}

func (p *HTTPClientPool) Get(url string) (*http.Response, error) {
    return p.Do("GET", url, nil)
}

func (p *HTTPClientPool) Post(url string, body io.Reader) (*http.Response, error) {
    return p.Do("POST", url, body)
}

func (p *HTTPClientPool) Do(method, url string, body io.Reader) (*http.Response, error) {
    p.stats.IncrementRequest()
    
    req, err := http.NewRequest(method, url, body)
    if err != nil {
        p.stats.IncrementError()
        return nil, err
    }
    
    // Add headers to track connection reuse
    req.Header.Set("User-Agent", "Go-PooledClient/1.0")
    req.Header.Set("Connection", "keep-alive")
    
    start := time.Now()
    resp, err := p.client.Do(req)
    duration := time.Since(start)
    
    if err != nil {
        p.stats.IncrementError()
        return nil, err
    }
    
    // Check if connection was reused
    if resp.Header.Get("Connection") == "keep-alive" {
        p.stats.IncrementReuse()
    }
    
    log.Printf("Request %s %s completed in %v (status: %d)", 
              method, url, duration, resp.StatusCode)
    
    return resp, nil
}

func (p *HTTPClientPool) GetStats() map[string]interface{} {
    requests, success, errors, bytes, reuse := p.stats.GetStats()
    
    successRate := float64(0)
    if requests > 0 {
        successRate = float64(success) / float64(requests) * 100
    }
    
    reuseRate := float64(0)
    if requests > 0 {
        reuseRate = float64(reuse) / float64(requests) * 100
    }
    
    return map[string]interface{}{
        "total_requests":    requests,
        "successful":        success,
        "errors":           errors,
        "total_bytes":      bytes,
        "connection_reuse": reuse,
        "success_rate":     fmt.Sprintf("%.2f%%", successRate),
        "reuse_rate":       fmt.Sprintf("%.2f%%", reuseRate),
        "config":           p.config,
    }
}

func (p *HTTPClientPool) Close() {
    if transport, ok := p.client.Transport.(*http.Transport); ok {
        transport.CloseIdleConnections()
    }
}

func performLoadTest(pool *HTTPClientPool, urls []string, concurrent int) {
    var wg sync.WaitGroup
    
    log.Printf("Starting load test with %d concurrent workers", concurrent)
    start := time.Now()
    
    for i := 0; i < concurrent; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            
            for j, url := range urls {
                resp, err := pool.Get(url)
                if err != nil {
                    log.Printf("Worker %d request %d failed: %v", 
                              workerID, j, err)
                    continue
                }
                
                body, err := io.ReadAll(resp.Body)
                resp.Body.Close()
                
                if err != nil {
                    log.Printf("Worker %d failed to read response %d: %v", 
                              workerID, j, err)
                    continue
                }
                
                pool.stats.IncrementSuccess(int64(len(body)))
                
                // Small delay to simulate processing
                time.Sleep(100 * time.Millisecond)
            }
        }(i)
    }
    
    wg.Wait()
    duration := time.Since(start)
    
    log.Printf("Load test completed in %v", duration)
}

func main() {
    config := &PoolConfig{
        MaxIdleConns:        50,
        MaxIdleConnsPerHost: 5,
        IdleConnTimeout:     60 * time.Second,
        DialTimeout:         10 * time.Second,
        TLSHandshakeTimeout: 5 * time.Second,
        ResponseHeaderTimeout: 15 * time.Second,
        ExpectContinueTimeout: 1 * time.Second,
    }
    
    pool := NewHTTPClientPool(config)
    defer pool.Close()
    
    // Test URLs
    urls := []string{
        "https://httpbin.org/get",
        "https://httpbin.org/json",
        "https://httpbin.org/headers",
        "https://httpbin.org/user-agent",
        "https://httpbin.org/uuid",
    }
    
    fmt.Println("=== HTTP Client Connection Pooling Demo ===")
    
    // Single request test
    fmt.Println("\n1. Single Request Test:")
    resp, err := pool.Get("https://httpbin.org/get")
    if err != nil {
        log.Fatal("Single request failed:", err)
    }
    resp.Body.Close()
    
    // Sequential requests test
    fmt.Println("\n2. Sequential Requests Test:")
    for i := 0; i < 5; i++ {
        resp, err := pool.Get(urls[i%len(urls)])
        if err != nil {
            log.Printf("Sequential request %d failed: %v", i, err)
            continue
        }
        io.ReadAll(resp.Body)
        resp.Body.Close()
    }
    
    // Concurrent load test
    fmt.Println("\n3. Concurrent Load Test:")
    performLoadTest(pool, urls, 10)
    
    // Display statistics
    fmt.Println("\n=== Client Statistics ===")
    stats := pool.GetStats()
    for key, value := range stats {
        fmt.Printf("%-20s: %v\n", key, value)
    }
    
    fmt.Println("\n=== Connection Pool Benefits ===")
    fmt.Println("• Reduced connection establishment overhead")
    fmt.Println("• Lower latency for subsequent requests")
    fmt.Println("• Better resource utilization")
    fmt.Println("• Improved scalability under load")
    fmt.Println("• Automatic connection lifecycle management")
}
```

This connection pooling example demonstrates advanced HTTP client  
configuration, performance monitoring, and connection reuse optimization.  
The code shows how proper pooling can significantly improve application  
performance and resource efficiency in high-throughput scenarios.

## HTTP request retries with exponential backoff

Robust HTTP clients need retry mechanisms to handle transient failures.  
This example demonstrates retry logic with exponential backoff, circuit  
breaking, and configurable retry policies.

```go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "math"
    "math/rand"
    "net/http"
    "sync"
    "time"
)

type RetryPolicy struct {
    MaxRetries      int
    InitialDelay    time.Duration
    MaxDelay        time.Duration
    BackoffFactor   float64
    JitterFactor    float64
    RetryableStatus []int
}

type CircuitBreakerState int

const (
    CircuitClosed CircuitBreakerState = iota
    CircuitOpen
    CircuitHalfOpen
)

type CircuitBreaker struct {
    maxFailures   int
    resetTimeout  time.Duration
    state         CircuitBreakerState
    failures      int
    lastFailTime  time.Time
    mutex         sync.RWMutex
}

type RetryableClient struct {
    client         *http.Client
    policy         *RetryPolicy
    circuitBreaker *CircuitBreaker
    stats          *RetryStats
}

type RetryStats struct {
    TotalRequests   int64
    SuccessRequests int64
    FailedRequests  int64
    RetriedRequests int64
    CircuitOpens    int64
    mutex           sync.RWMutex
}

type RetryableError struct {
    Attempt    int
    StatusCode int
    Err        error
    Retryable  bool
}

func (e RetryableError) Error() string {
    return fmt.Sprintf("attempt %d failed (status %d): %v (retryable: %v)", 
                       e.Attempt, e.StatusCode, e.Err, e.Retryable)
}

func NewRetryPolicy() *RetryPolicy {
    return &RetryPolicy{
        MaxRetries:      3,
        InitialDelay:    100 * time.Millisecond,
        MaxDelay:        30 * time.Second,
        BackoffFactor:   2.0,
        JitterFactor:    0.1,
        RetryableStatus: []int{500, 502, 503, 504, 408, 429},
    }
}

func NewCircuitBreaker(maxFailures int, resetTimeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures:  maxFailures,
        resetTimeout: resetTimeout,
        state:        CircuitClosed,
    }
}

func (cb *CircuitBreaker) CanExecute() bool {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    
    switch cb.state {
    case CircuitClosed:
        return true
    case CircuitOpen:
        if time.Since(cb.lastFailTime) > cb.resetTimeout {
            cb.state = CircuitHalfOpen
            return true
        }
        return false
    case CircuitHalfOpen:
        return true
    default:
        return false
    }
}

func (cb *CircuitBreaker) OnSuccess() {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    
    cb.failures = 0
    cb.state = CircuitClosed
}

func (cb *CircuitBreaker) OnFailure() {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    
    cb.failures++
    cb.lastFailTime = time.Now()
    
    if cb.failures >= cb.maxFailures {
        cb.state = CircuitOpen
    }
}

func (cb *CircuitBreaker) GetState() CircuitBreakerState {
    cb.mutex.RLock()
    defer cb.mutex.RUnlock()
    return cb.state
}

func NewRetryableClient(policy *RetryPolicy, circuitBreaker *CircuitBreaker) *RetryableClient {
    if policy == nil {
        policy = NewRetryPolicy()
    }
    
    if circuitBreaker == nil {
        circuitBreaker = NewCircuitBreaker(5, 30*time.Second)
    }
    
    return &RetryableClient{
        client: &http.Client{
            Timeout: 30 * time.Second,
        },
        policy:         policy,
        circuitBreaker: circuitBreaker,
        stats:          &RetryStats{},
    }
}

func (rc *RetryableClient) isRetryable(statusCode int, err error) bool {
    // Network errors are generally retryable
    if err != nil {
        return true
    }
    
    // Check if status code is in retryable list
    for _, code := range rc.policy.RetryableStatus {
        if statusCode == code {
            return true
        }
    }
    
    return false
}

func (rc *RetryableClient) calculateDelay(attempt int) time.Duration {
    delay := float64(rc.policy.InitialDelay) * 
             math.Pow(rc.policy.BackoffFactor, float64(attempt-1))
    
    // Add jitter to prevent thundering herd
    jitter := delay * rc.policy.JitterFactor * (rand.Float64()*2 - 1)
    delay += jitter
    
    // Cap at maximum delay
    if delay > float64(rc.policy.MaxDelay) {
        delay = float64(rc.policy.MaxDelay)
    }
    
    return time.Duration(delay)
}

func (rc *RetryableClient) Do(req *http.Request) (*http.Response, error) {
    return rc.DoWithContext(context.Background(), req)
}

func (rc *RetryableClient) DoWithContext(ctx context.Context, req *http.Request) (*http.Response, error) {
    rc.stats.mutex.Lock()
    rc.stats.TotalRequests++
    rc.stats.mutex.Unlock()
    
    // Check circuit breaker
    if !rc.circuitBreaker.CanExecute() {
        rc.stats.mutex.Lock()
        rc.stats.FailedRequests++
        rc.stats.mutex.Unlock()
        
        return nil, fmt.Errorf("circuit breaker is open")
    }
    
    var lastErr error
    var lastStatusCode int
    
    for attempt := 1; attempt <= rc.policy.MaxRetries+1; attempt++ {
        // Clone request for retry
        reqClone := req.Clone(ctx)
        
        resp, err := rc.client.Do(reqClone)
        
        if err == nil && resp.StatusCode < 400 {
            // Success
            rc.circuitBreaker.OnSuccess()
            rc.stats.mutex.Lock()
            rc.stats.SuccessRequests++
            rc.stats.mutex.Unlock()
            
            return resp, nil
        }
        
        statusCode := 0
        if resp != nil {
            statusCode = resp.StatusCode
            resp.Body.Close() // Close body to prevent resource leak
        }
        
        lastErr = err
        lastStatusCode = statusCode
        
        // Check if this error is retryable
        if !rc.isRetryable(statusCode, err) {
            rc.circuitBreaker.OnFailure()
            rc.stats.mutex.Lock()
            rc.stats.FailedRequests++
            rc.stats.mutex.Unlock()
            
            return nil, RetryableError{
                Attempt:    attempt,
                StatusCode: statusCode,
                Err:        err,
                Retryable:  false,
            }
        }
        
        // If this is the last attempt, don't wait
        if attempt > rc.policy.MaxRetries {
            break
        }
        
        // Calculate delay for next attempt
        delay := rc.calculateDelay(attempt)
        
        log.Printf("Attempt %d failed (status: %d, error: %v), retrying in %v", 
                  attempt, statusCode, err, delay)
        
        rc.stats.mutex.Lock()
        rc.stats.RetriedRequests++
        rc.stats.mutex.Unlock()
        
        // Wait before retry, respecting context cancellation
        select {
        case <-ctx.Done():
            return nil, ctx.Err()
        case <-time.After(delay):
            // Continue to next attempt
        }
    }
    
    // All retries exhausted
    rc.circuitBreaker.OnFailure()
    rc.stats.mutex.Lock()
    rc.stats.FailedRequests++
    rc.stats.mutex.Unlock()
    
    if rc.circuitBreaker.GetState() == CircuitOpen {
        rc.stats.mutex.Lock()
        rc.stats.CircuitOpens++
        rc.stats.mutex.Unlock()
    }
    
    return nil, RetryableError{
        Attempt:    rc.policy.MaxRetries + 1,
        StatusCode: lastStatusCode,
        Err:        lastErr,
        Retryable:  true,
    }
}

func (rc *RetryableClient) Get(url string) (*http.Response, error) {
    req, err := http.NewRequest("GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    return rc.Do(req)
}

func (rc *RetryableClient) GetStats() map[string]interface{} {
    rc.stats.mutex.RLock()
    defer rc.stats.mutex.RUnlock()
    
    successRate := float64(0)
    if rc.stats.TotalRequests > 0 {
        successRate = float64(rc.stats.SuccessRequests) / 
                     float64(rc.stats.TotalRequests) * 100
    }
    
    return map[string]interface{}{
        "total_requests":   rc.stats.TotalRequests,
        "success_requests": rc.stats.SuccessRequests,
        "failed_requests":  rc.stats.FailedRequests,
        "retried_requests": rc.stats.RetriedRequests,
        "circuit_opens":    rc.stats.CircuitOpens,
        "success_rate":     fmt.Sprintf("%.2f%%", successRate),
        "circuit_state":    rc.circuitBreaker.GetState(),
    }
}

func simulateFailingService() *http.ServeMux {
    mux := http.NewServeMux()
    
    requestCount := 0
    
    mux.HandleFunc("/reliable", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        fmt.Fprint(w, `{"message": "Hello there! This endpoint is reliable."}`)
    })
    
    mux.HandleFunc("/flaky", func(w http.ResponseWriter, r *http.Request) {
        requestCount++
        
        // Fail 70% of the time
        if rand.Float32() < 0.7 {
            status := []int{500, 502, 503, 504}[rand.Intn(4)]
            w.WriteHeader(status)
            fmt.Fprintf(w, `{"error": "Simulated failure", "status": %d}`, status)
            return
        }
        
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, `{"message": "Hello there! Success after %d total requests."}`, 
                   requestCount)
    })
    
    mux.HandleFunc("/very-flaky", func(w http.ResponseWriter, r *http.Request) {
        // Fail 90% of the time
        if rand.Float32() < 0.9 {
            w.WriteHeader(http.StatusInternalServerError)
            fmt.Fprint(w, `{"error": "Very flaky service failure"}`)
            return
        }
        
        w.WriteHeader(http.StatusOK)
        fmt.Fprint(w, `{"message": "Hello there! Rare success!"}`)
    })
    
    return mux
}

func main() {
    // Start test server
    go func() {
        mux := simulateFailingService()
        log.Printf("Test server starting on :9090")
        http.ListenAndServe(":9090", mux)
    }()
    
    // Wait for server to start
    time.Sleep(100 * time.Millisecond)
    
    // Configure retry policy
    policy := &RetryPolicy{
        MaxRetries:      5,
        InitialDelay:    200 * time.Millisecond,
        MaxDelay:        10 * time.Second,
        BackoffFactor:   2.0,
        JitterFactor:    0.1,
        RetryableStatus: []int{500, 502, 503, 504, 408, 429},
    }
    
    circuitBreaker := NewCircuitBreaker(3, 10*time.Second)
    client := NewRetryableClient(policy, circuitBreaker)
    
    fmt.Println("=== HTTP Retry with Exponential Backoff Demo ===")
    
    // Test reliable endpoint
    fmt.Println("\n1. Testing reliable endpoint:")
    resp, err := client.Get("http://localhost:9090/reliable")
    if err != nil {
        log.Printf("Reliable endpoint failed: %v", err)
    } else {
        body, _ := io.ReadAll(resp.Body)
        resp.Body.Close()
        fmt.Printf("Success: %s\n", body)
    }
    
    // Test flaky endpoint
    fmt.Println("\n2. Testing flaky endpoint (multiple attempts):")
    for i := 0; i < 5; i++ {
        resp, err := client.Get("http://localhost:9090/flaky")
        if err != nil {
            log.Printf("Flaky request %d failed: %v", i+1, err)
        } else {
            body, _ := io.ReadAll(resp.Body)
            resp.Body.Close()
            fmt.Printf("Flaky request %d succeeded: %s\n", i+1, body)
        }
    }
    
    // Test very flaky endpoint to trigger circuit breaker
    fmt.Println("\n3. Testing very flaky endpoint (to trigger circuit breaker):")
    for i := 0; i < 10; i++ {
        resp, err := client.Get("http://localhost:9090/very-flaky")
        if err != nil {
            log.Printf("Very flaky request %d failed: %v", i+1, err)
        } else {
            body, _ := io.ReadAll(resp.Body)
            resp.Body.Close()
            fmt.Printf("Very flaky request %d succeeded: %s\n", i+1, body)
        }
        time.Sleep(500 * time.Millisecond)
    }
    
    // Display statistics
    fmt.Println("\n=== Client Statistics ===")
    stats := client.GetStats()
    for key, value := range stats {
        fmt.Printf("%-20s: %v\n", key, value)
    }
    
    fmt.Println("\n=== Retry Strategy Benefits ===")
    fmt.Println("• Automatic recovery from transient failures")
    fmt.Println("• Exponential backoff prevents server overload")
    fmt.Println("• Jitter reduces thundering herd effects")
    fmt.Println("• Circuit breaker protects against cascading failures")
    fmt.Println("• Configurable retry policies for different scenarios")
}
```

This retry mechanism example demonstrates sophisticated error handling with  
exponential backoff, jitter, circuit breaking, and comprehensive statistics.  
The implementation shows how to build resilient HTTP clients that can  
gracefully handle transient failures while protecting backend services  
from being overwhelmed.

## HTTP testing with httptest

Testing HTTP code is essential for reliable web applications. This example  
demonstrates using the `httptest` package for unit testing HTTP handlers,  
servers, and client interactions.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "net/http/httptest"
    "net/url"
    "strings"
    "testing"
    "time"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

type UserService struct {
    users map[int]User
    nextID int
}

func NewUserService() *UserService {
    return &UserService{
        users:  make(map[int]User),
        nextID: 1,
    }
}

func (s *UserService) CreateUser(name, email string) User {
    user := User{
        ID:    s.nextID,
        Name:  name,
        Email: email,
    }
    s.users[s.nextID] = user
    s.nextID++
    return user
}

func (s *UserService) GetUser(id int) (User, bool) {
    user, exists := s.users[id]
    return user, exists
}

func (s *UserService) GetAllUsers() []User {
    users := make([]User, 0, len(s.users))
    for _, user := range s.users {
        users = append(users, user)
    }
    return users
}

func (s *UserService) UpdateUser(id int, name, email string) bool {
    if _, exists := s.users[id]; !exists {
        return false
    }
    
    s.users[id] = User{
        ID:    id,
        Name:  name,
        Email: email,
    }
    return true
}

func (s *UserService) DeleteUser(id int) bool {
    if _, exists := s.users[id]; !exists {
        return false
    }
    
    delete(s.users, id)
    return true
}

type UserHandler struct {
    service *UserService
}

func NewUserHandler(service *UserService) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        h.handleGet(w, r)
    case http.MethodPost:
        h.handlePost(w, r)
    case http.MethodPut:
        h.handlePut(w, r)
    case http.MethodDelete:
        h.handleDelete(w, r)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

func (h *UserHandler) handleGet(w http.ResponseWriter, r *http.Request) {
    users := h.service.GetAllUsers()
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]interface{}{
        "users": users,
        "count": len(users),
    })
}

func (h *UserHandler) handlePost(w http.ResponseWriter, r *http.Request) {
    var request struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    if request.Name == "" || request.Email == "" {
        http.Error(w, "Name and email required", http.StatusBadRequest)
        return
    }
    
    user := h.service.CreateUser(request.Name, request.Email)
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}

func (h *UserHandler) handlePut(w http.ResponseWriter, r *http.Request) {
    // Simplified: assume ID is in query parameter
    idStr := r.URL.Query().Get("id")
    if idStr == "" {
        http.Error(w, "ID required", http.StatusBadRequest)
        return
    }
    
    var id int
    if _, err := fmt.Sscanf(idStr, "%d", &id); err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }
    
    var request struct {
        Name  string `json:"name"`
        Email string `json:"email"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    if !h.service.UpdateUser(id, request.Name, request.Email) {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    user, _ := h.service.GetUser(id)
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

func (h *UserHandler) handleDelete(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    if idStr == "" {
        http.Error(w, "ID required", http.StatusBadRequest)
        return
    }
    
    var id int
    if _, err := fmt.Sscanf(idStr, "%d", &id); err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }
    
    if !h.service.DeleteUser(id) {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    w.WriteHeader(http.StatusNoContent)
}

// Test functions

func TestUserHandler_GetUsers(t *testing.T) {
    service := NewUserService()
    service.CreateUser("Alice", "alice@example.com")
    service.CreateUser("Bob", "bob@example.com")
    
    handler := NewUserHandler(service)
    
    req := httptest.NewRequest("GET", "/users", nil)
    w := httptest.NewRecorder()
    
    handler.ServeHTTP(w, req)
    
    if w.Code != http.StatusOK {
        t.Errorf("Expected status %d, got %d", http.StatusOK, w.Code)
    }
    
    var response map[string]interface{}
    if err := json.Unmarshal(w.Body.Bytes(), &response); err != nil {
        t.Errorf("Failed to parse response: %v", err)
    }
    
    count := response["count"].(float64)
    if count != 2 {
        t.Errorf("Expected 2 users, got %f", count)
    }
}

func TestUserHandler_CreateUser(t *testing.T) {
    service := NewUserService()
    handler := NewUserHandler(service)
    
    user := map[string]string{
        "name":  "Charlie",
        "email": "charlie@example.com",
    }
    
    jsonData, _ := json.Marshal(user)
    req := httptest.NewRequest("POST", "/users", bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")
    
    w := httptest.NewRecorder()
    handler.ServeHTTP(w, req)
    
    if w.Code != http.StatusCreated {
        t.Errorf("Expected status %d, got %d", http.StatusCreated, w.Code)
    }
    
    var createdUser User
    if err := json.Unmarshal(w.Body.Bytes(), &createdUser); err != nil {
        t.Errorf("Failed to parse response: %v", err)
    }
    
    if createdUser.Name != "Charlie" {
        t.Errorf("Expected name 'Charlie', got '%s'", createdUser.Name)
    }
}

func TestUserHandler_CreateUser_InvalidJSON(t *testing.T) {
    service := NewUserService()
    handler := NewUserHandler(service)
    
    req := httptest.NewRequest("POST", "/users", strings.NewReader("invalid json"))
    req.Header.Set("Content-Type", "application/json")
    
    w := httptest.NewRecorder()
    handler.ServeHTTP(w, req)
    
    if w.Code != http.StatusBadRequest {
        t.Errorf("Expected status %d, got %d", http.StatusBadRequest, w.Code)
    }
}

func TestUserHandler_UpdateUser(t *testing.T) {
    service := NewUserService()
    user := service.CreateUser("David", "david@example.com")
    handler := NewUserHandler(service)
    
    updateData := map[string]string{
        "name":  "David Updated",
        "email": "david.updated@example.com",
    }
    
    jsonData, _ := json.Marshal(updateData)
    req := httptest.NewRequest("PUT", fmt.Sprintf("/users?id=%d", user.ID), 
                              bytes.NewBuffer(jsonData))
    req.Header.Set("Content-Type", "application/json")
    
    w := httptest.NewRecorder()
    handler.ServeHTTP(w, req)
    
    if w.Code != http.StatusOK {
        t.Errorf("Expected status %d, got %d", http.StatusOK, w.Code)
    }
    
    var updatedUser User
    if err := json.Unmarshal(w.Body.Bytes(), &updatedUser); err != nil {
        t.Errorf("Failed to parse response: %v", err)
    }
    
    if updatedUser.Name != "David Updated" {
        t.Errorf("Expected name 'David Updated', got '%s'", updatedUser.Name)
    }
}

func TestUserHandler_DeleteUser(t *testing.T) {
    service := NewUserService()
    user := service.CreateUser("Eve", "eve@example.com")
    handler := NewUserHandler(service)
    
    req := httptest.NewRequest("DELETE", fmt.Sprintf("/users?id=%d", user.ID), nil)
    w := httptest.NewRecorder()
    
    handler.ServeHTTP(w, req)
    
    if w.Code != http.StatusNoContent {
        t.Errorf("Expected status %d, got %d", http.StatusNoContent, w.Code)
    }
    
    // Verify user was deleted
    _, exists := service.GetUser(user.ID)
    if exists {
        t.Errorf("User should have been deleted")
    }
}

func TestHTTPClient_Integration(t *testing.T) {
    // Create test server
    service := NewUserService()
    handler := NewUserHandler(service)
    server := httptest.NewServer(handler)
    defer server.Close()
    
    client := &http.Client{Timeout: 5 * time.Second}
    
    // Test creating a user
    userData := map[string]string{
        "name":  "Test User",
        "email": "test@example.com",
    }
    
    jsonData, _ := json.Marshal(userData)
    resp, err := client.Post(server.URL, "application/json", 
                            bytes.NewBuffer(jsonData))
    if err != nil {
        t.Fatalf("Failed to create user: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusCreated {
        t.Errorf("Expected status %d, got %d", http.StatusCreated, resp.StatusCode)
    }
    
    var createdUser User
    if err := json.NewDecoder(resp.Body).Decode(&createdUser); err != nil {
        t.Errorf("Failed to decode response: %v", err)
    }
    
    // Test getting users
    resp, err = client.Get(server.URL)
    if err != nil {
        t.Fatalf("Failed to get users: %v", err)
    }
    defer resp.Body.Close()
    
    if resp.StatusCode != http.StatusOK {
        t.Errorf("Expected status %d, got %d", http.StatusOK, resp.StatusCode)
    }
}

func BenchmarkUserHandler_CreateUser(b *testing.B) {
    service := NewUserService()
    handler := NewUserHandler(service)
    
    userData := map[string]string{
        "name":  "Benchmark User",
        "email": "benchmark@example.com",
    }
    jsonData, _ := json.Marshal(userData)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        req := httptest.NewRequest("POST", "/users", bytes.NewBuffer(jsonData))
        req.Header.Set("Content-Type", "application/json")
        
        w := httptest.NewRecorder()
        handler.ServeHTTP(w, req)
        
        if w.Code != http.StatusCreated {
            b.Errorf("Expected status %d, got %d", http.StatusCreated, w.Code)
        }
    }
}

func ExampleHTTPTest() {
    // Create a test server
    handler := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]string{
            "message": "Hello there from test server!",
            "path":    r.URL.Path,
            "method":  r.Method,
        })
    })
    
    server := httptest.NewServer(handler)
    defer server.Close()
    
    // Make a request to the test server
    resp, err := http.Get(server.URL + "/test")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Response: %s\n", body)
    
    // Output: Response: {"message":"Hello there from test server!","method":"GET","path":"/test"}
}

func main() {
    fmt.Println("=== HTTP Testing with httptest Demo ===")
    
    // Run example
    ExampleHTTPTest()
    
    // Create a simple test server for interactive testing
    service := NewUserService()
    service.CreateUser("Alice", "alice@example.com")
    service.CreateUser("Bob", "bob@example.com")
    
    handler := NewUserHandler(service)
    server := httptest.NewServer(handler)
    defer server.Close()
    
    fmt.Printf("\nTest server running at: %s\n", server.URL)
    fmt.Println("Try these commands:")
    fmt.Printf("curl %s\n", server.URL)
    fmt.Printf("curl -X POST -H 'Content-Type: application/json' -d '{\"name\":\"Charlie\",\"email\":\"charlie@example.com\"}' %s\n", server.URL)
    
    // Keep server running for a bit
    time.Sleep(2 * time.Second)
    
    fmt.Println("\n=== Testing Benefits ===")
    fmt.Println("• Isolated testing without external dependencies")
    fmt.Println("• Fast test execution with in-memory servers")
    fmt.Println("• Complete control over request/response cycle")
    fmt.Println("• Easy mocking of external HTTP services")
    fmt.Println("• Comprehensive coverage of HTTP edge cases")
}
```

This comprehensive testing example demonstrates unit testing HTTP handlers,  
integration testing with test servers, benchmarking HTTP operations, and  
mocking external services. The `httptest` package provides powerful tools  
for testing HTTP code without requiring real network connections.

## HTTP graceful shutdown

Graceful shutdown ensures that ongoing requests are completed before stopping  
the server. This example demonstrates proper server lifecycle management,  
signal handling, and connection draining.

```go
package main

import (
    "context"
    "fmt"
    "log"
    "net/http"
    "os"
    "os/signal"
    "sync"
    "sync/atomic"
    "syscall"
    "time"
)

type GracefulServer struct {
    server          *http.Server
    shutdownTimeout time.Duration
    requestCount    int64
    activeRequests  int64
    isShuttingDown  int32
    shutdownChan    chan struct{}
    wg              sync.WaitGroup
}

type ServerStats struct {
    StartTime       time.Time
    TotalRequests   int64
    ActiveRequests  int64
    IsShuttingDown  bool
    Uptime          time.Duration
}

func NewGracefulServer(addr string, handler http.Handler, shutdownTimeout time.Duration) *GracefulServer {
    return &GracefulServer{
        server: &http.Server{
            Addr:    addr,
            Handler: handler,
        },
        shutdownTimeout: shutdownTimeout,
        shutdownChan:    make(chan struct{}),
    }
}

func (gs *GracefulServer) trackRequests(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Check if server is shutting down
        if atomic.LoadInt32(&gs.isShuttingDown) == 1 {
            w.Header().Set("Connection", "close")
            http.Error(w, "Server is shutting down", http.StatusServiceUnavailable)
            return
        }
        
        // Increment counters
        atomic.AddInt64(&gs.requestCount, 1)
        atomic.AddInt64(&gs.activeRequests, 1)
        
        // Ensure this request is tracked for graceful shutdown
        gs.wg.Add(1)
        defer func() {
            atomic.AddInt64(&gs.activeRequests, -1)
            gs.wg.Done()
        }()
        
        next.ServeHTTP(w, r)
    })
}

func (gs *GracefulServer) Start() error {
    gs.server.Handler = gs.trackRequests(gs.server.Handler)
    
    log.Printf("Starting server on %s", gs.server.Addr)
    
    go func() {
        if err := gs.server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Printf("Server error: %v", err)
        }
    }()
    
    return nil
}

func (gs *GracefulServer) Shutdown(ctx context.Context) error {
    log.Println("Initiating graceful shutdown...")
    
    // Mark server as shutting down
    atomic.StoreInt32(&gs.isShuttingDown, 1)
    close(gs.shutdownChan)
    
    // Create a context with timeout for shutdown
    shutdownCtx, cancel := context.WithTimeout(ctx, gs.shutdownTimeout)
    defer cancel()
    
    // Start graceful shutdown
    shutdownComplete := make(chan error, 1)
    go func() {
        // Wait for active requests to complete
        log.Printf("Waiting for %d active requests to complete...", 
                  atomic.LoadInt64(&gs.activeRequests))
        
        done := make(chan struct{})
        go func() {
            gs.wg.Wait()
            close(done)
        }()
        
        select {
        case <-done:
            log.Println("All requests completed")
        case <-shutdownCtx.Done():
            log.Println("Shutdown timeout reached, forcing shutdown")
        }
        
        shutdownComplete <- gs.server.Shutdown(shutdownCtx)
    }()
    
    select {
    case err := <-shutdownComplete:
        return err
    case <-shutdownCtx.Done():
        return shutdownCtx.Err()
    }
}

func (gs *GracefulServer) GetStats() ServerStats {
    return ServerStats{
        StartTime:      time.Now(), // In real implementation, track actual start time
        TotalRequests:  atomic.LoadInt64(&gs.requestCount),
        ActiveRequests: atomic.LoadInt64(&gs.activeRequests),
        IsShuttingDown: atomic.LoadInt32(&gs.isShuttingDown) == 1,
        Uptime:         time.Since(time.Now()), // In real implementation, calculate properly
    }
}

func createDemoHandlers() *http.ServeMux {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "text/html")
        fmt.Fprint(w, `
        <html>
        <head><title>Graceful Shutdown Demo</title></head>
        <body>
            <h1>Hello there!</h1>
            <p>This is a graceful shutdown demonstration.</p>
            <ul>
                <li><a href="/fast">Fast response</a></li>
                <li><a href="/slow">Slow response (5s)</a></li>
                <li><a href="/very-slow">Very slow response (15s)</a></li>
                <li><a href="/stats">Server stats</a></li>
            </ul>
            <p>Try accessing slow endpoints then send SIGTERM to test graceful shutdown.</p>
        </body>
        </html>`)
    })
    
    mux.HandleFunc("/fast", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{
            "message": "Hello there! This is a fast response.",
            "timestamp": "%s",
            "processing_time": "immediate"
        }`, time.Now().Format(time.RFC3339))
    })
    
    mux.HandleFunc("/slow", func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Simulate slow processing
        time.Sleep(5 * time.Second)
        
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{
            "message": "Hello there! This is a slow response.",
            "timestamp": "%s",
            "processing_time": "%v"
        }`, time.Now().Format(time.RFC3339), time.Since(start))
    })
    
    mux.HandleFunc("/very-slow", func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // Simulate very slow processing with periodic checks
        for i := 0; i < 15; i++ {
            time.Sleep(1 * time.Second)
            
            // Check if server is shutting down
            select {
            case <-r.Context().Done():
                w.Header().Set("Content-Type", "application/json")
                fmt.Fprintf(w, `{
                    "message": "Request cancelled during processing",
                    "timestamp": "%s",
                    "processing_time": "%v"
                }`, time.Now().Format(time.RFC3339), time.Since(start))
                return
            default:
                // Continue processing
            }
        }
        
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{
            "message": "Hello there! This is a very slow response.",
            "timestamp": "%s",
            "processing_time": "%v"
        }`, time.Now().Format(time.RFC3339), time.Since(start))
    })
    
    return mux
}

func statsHandler(gs *GracefulServer) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        stats := gs.GetStats()
        
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{
            "total_requests": %d,
            "active_requests": %d,
            "is_shutting_down": %t,
            "timestamp": "%s"
        }`, stats.TotalRequests, stats.ActiveRequests, 
            stats.IsShuttingDown, time.Now().Format(time.RFC3339))
    }
}

func setupSignalHandling(gs *GracefulServer) {
    signalChan := make(chan os.Signal, 1)
    signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)
    
    go func() {
        sig := <-signalChan
        log.Printf("Received signal: %v", sig)
        
        // Create context with timeout for shutdown
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
        defer cancel()
        
        if err := gs.Shutdown(ctx); err != nil {
            log.Printf("Graceful shutdown failed: %v", err)
            os.Exit(1)
        }
        
        log.Println("Server stopped gracefully")
        os.Exit(0)
    }()
}

func simulateTraffic(serverAddr string) {
    go func() {
        time.Sleep(2 * time.Second) // Wait for server to start
        
        log.Println("Simulating traffic...")
        
        endpoints := []string{"/fast", "/slow", "/very-slow"}
        
        for i := 0; i < 10; i++ {
            endpoint := endpoints[i%len(endpoints)]
            
            go func(ep string, reqNum int) {
                resp, err := http.Get("http://" + serverAddr + ep)
                if err != nil {
                    log.Printf("Request %d to %s failed: %v", reqNum, ep, err)
                    return
                }
                defer resp.Body.Close()
                
                log.Printf("Request %d to %s completed (status: %d)", 
                          reqNum, ep, resp.StatusCode)
            }(endpoint, i)
            
            time.Sleep(500 * time.Millisecond)
        }
    }()
}

func main() {
    mux := createDemoHandlers()
    
    server := NewGracefulServer(":8080", mux, 20*time.Second)
    
    // Add stats endpoint
    mux.HandleFunc("/stats", statsHandler(server))
    
    // Setup signal handling for graceful shutdown
    setupSignalHandling(server)
    
    // Start the server
    if err := server.Start(); err != nil {
        log.Fatal("Failed to start server:", err)
    }
    
    // Simulate some traffic
    simulateTraffic("localhost:8080")
    
    fmt.Println("=== Graceful Shutdown Demo ===")
    fmt.Println("Server started on http://localhost:8080")
    fmt.Println("Try accessing slow endpoints, then press Ctrl+C to test graceful shutdown")
    fmt.Println()
    fmt.Println("Test commands:")
    fmt.Println("  curl http://localhost:8080/slow &")
    fmt.Println("  curl http://localhost:8080/very-slow &")
    fmt.Println("  # Then press Ctrl+C")
    fmt.Println()
    fmt.Println("The server will:")
    fmt.Println("• Stop accepting new requests")
    fmt.Println("• Wait for active requests to complete")
    fmt.Println("• Timeout after 20 seconds if requests don't finish")
    fmt.Println("• Clean up resources and exit")
    
    // Keep the main goroutine running
    select {}
}
```

This graceful shutdown example demonstrates proper server lifecycle management,  
signal handling, request tracking, and connection draining. The implementation  
ensures that clients receive proper responses and no requests are abruptly  
terminated during server shutdown, which is crucial for production systems.

## HTTP proxy server

Proxy servers forward requests to backend servers and can modify requests  
and responses. This example demonstrates reverse proxy implementation with  
load balancing and health checking.

```go
package main

import (
    "context"
    "fmt"
    "io"
    "log"
    "net/http"
    "net/http/httputil"
    "net/url"
    "strings"
    "sync"
    "sync/atomic"
    "time"
)

type Backend struct {
    URL          *url.URL
    Alive        bool
    mux          sync.RWMutex
    ReverseProxy *httputil.ReverseProxy
}

type ServerPool struct {
    backends []*Backend
    current  uint64
}

type ProxyStats struct {
    TotalRequests   int64
    BackendRequests map[string]int64
    FailedRequests  int64
    mutex           sync.RWMutex
}

func (b *Backend) SetAlive(alive bool) {
    b.mux.Lock()
    b.Alive = alive
    b.mux.Unlock()
}

func (b *Backend) IsAlive() bool {
    b.mux.RLock()
    alive := b.Alive
    b.mux.RUnlock()
    return alive
}

func (s *ServerPool) AddBackend(backend *Backend) {
    s.backends = append(s.backends, backend)
}

func (s *ServerPool) NextIndex() int {
    return int(atomic.AddUint64(&s.current, uint64(1)) % uint64(len(s.backends)))
}

func (s *ServerPool) MarkBackendStatus(backendUrl *url.URL, alive bool) {
    for _, b := range s.backends {
        if b.URL.String() == backendUrl.String() {
            b.SetAlive(alive)
            break
        }
    }
}

func (s *ServerPool) GetNextPeer() *Backend {
    next := s.NextIndex()
    l := len(s.backends) + next

    for i := next; i < l; i++ {
        idx := i % len(s.backends)
        if s.backends[idx].IsAlive() {
            if i != next {
                atomic.StoreUint64(&s.current, uint64(idx))
            }
            return s.backends[idx]
        }
    }
    return nil
}

func (s *ServerPool) HealthCheck() {
    for _, b := range s.backends {
        status := "up"
        alive := isBackendAlive(b.URL)
        b.SetAlive(alive)
        if !alive {
            status = "down"
        }
        log.Printf("Backend %s is %s", b.URL, status)
    }
}

func isBackendAlive(u *url.URL) bool {
    timeout := 2 * time.Second
    conn, err := net.DialTimeout("tcp", u.Host, timeout)
    if err != nil {
        return false
    }
    defer conn.Close()
    return true
}

func healthCheck() {
    t := time.NewTicker(time.Second * 20)
    for {
        select {
        case <-t.C:
            log.Println("Starting health check...")
            serverPool.HealthCheck()
            log.Println("Health check completed")
        }
    }
}

func loadBalance(w http.ResponseWriter, r *http.Request) {
    attempts := GetAttemptsFromContext(r)
    if attempts > 3 {
        log.Printf("%s(%s) Max attempts reached, terminating\n", r.RemoteAddr, r.URL.Path)
        http.Error(w, "Service not available", http.StatusServiceUnavailable)
        return
    }

    peer := serverPool.GetNextPeer()
    if peer != nil {
        peer.ReverseProxy.ServeHTTP(w, r)
        return
    }
    http.Error(w, "Service not available", http.StatusServiceUnavailable)
}

func GetAttemptsFromContext(r *http.Request) int {
    if attempts, ok := r.Context().Value(Attempts).(int); ok {
        return attempts
    }
    return 1
}

func GetRetryFromContext(r *http.Request) int {
    if retry, ok := r.Context().Value(Retry).(int); ok {
        return retry
    }
    return 0
}

const (
    Attempts int = iota
    Retry
)

var serverPool ServerPool
var stats ProxyStats

func createReverseProxy(target *url.URL) *httputil.ReverseProxy {
    proxy := httputil.NewSingleHostReverseProxy(target)

    originalDirector := proxy.Director
    proxy.Director = func(req *http.Request) {
        originalDirector(req)
        modifyRequest(req)
    }

    proxy.ModifyResponse = modifyResponse
    proxy.ErrorHandler = errorHandler
    return proxy
}

func modifyRequest(req *http.Request) {
    req.Header.Set("X-Proxy", "Go-Proxy/1.0")
    req.Header.Set("X-Forwarded-For", req.RemoteAddr)
}

func modifyResponse(resp *http.Response) error {
    resp.Header.Set("X-Proxy", "Go-Proxy/1.0")
    return nil
}

func errorHandler(w http.ResponseWriter, r *http.Request, err error) {
    log.Printf("Error: %v", err)

    retries := GetRetryFromContext(r)
    if retries < 3 {
        select {
        case <-time.After(10 * time.Millisecond):
            ctx := context.WithValue(r.Context(), Retry, retries+1)
            proxy.ServeHTTP(w, r.WithContext(ctx))
        }
        return
    }

    serverPool.MarkBackendStatus(r.URL, false)

    attempts := GetAttemptsFromContext(r)
    log.Printf("%s(%s) Attempting retry %d\n", r.RemoteAddr, r.URL.Path, attempts)
    ctx := context.WithValue(r.Context(), Attempts, attempts+1)
    loadBalance(w, r.WithContext(ctx))
}

func proxyHandler(w http.ResponseWriter, r *http.Request) {
    atomic.AddInt64(&stats.TotalRequests, 1)
    
    if r.URL.Path == "/stats" {
        statsHandler(w, r)
        return
    }
    
    if r.URL.Path == "/health" {
        healthHandler(w, r)
        return
    }
    
    loadBalance(w, r)
}

func statsHandler(w http.ResponseWriter, r *http.Request) {
    stats.mutex.RLock()
    defer stats.mutex.RUnlock()
    
    w.Header().Set("Content-Type", "application/json")
    fmt.Fprintf(w, `{
        "total_requests": %d,
        "failed_requests": %d,
        "backends": [`, stats.TotalRequests, stats.FailedRequests)
    
    for i, backend := range serverPool.backends {
        if i > 0 {
            fmt.Fprint(w, ",")
        }
        fmt.Fprintf(w, `{
            "url": "%s",
            "alive": %t,
            "requests": %d
        }`, backend.URL.String(), backend.IsAlive(), 
            stats.BackendRequests[backend.URL.String()])
    }
    
    fmt.Fprint(w, `]
    }`)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    healthy := 0
    for _, backend := range serverPool.backends {
        if backend.IsAlive() {
            healthy++
        }
    }
    
    status := http.StatusOK
    message := "Healthy"
    
    if healthy == 0 {
        status = http.StatusServiceUnavailable
        message = "No healthy backends"
    } else if healthy < len(serverPool.backends) {
        message = "Partially healthy"
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    fmt.Fprintf(w, `{
        "status": "%s",
        "healthy_backends": %d,
        "total_backends": %d,
        "timestamp": "%s"
    }`, message, healthy, len(serverPool.backends), time.Now().Format(time.RFC3339))
}

func createBackendServers() {
    // Backend server 1
    go func() {
        mux := http.NewServeMux()
        mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            w.Header().Set("Content-Type", "application/json")
            fmt.Fprintf(w, `{
                "message": "Hello there from backend server 1!",
                "server": "backend-1",
                "path": "%s",
                "timestamp": "%s"
            }`, r.URL.Path, time.Now().Format(time.RFC3339))
        })
        
        log.Println("Backend server 1 starting on :3001")
        http.ListenAndServe(":3001", mux)
    }()
    
    // Backend server 2
    go func() {
        mux := http.NewServeMux()
        mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            w.Header().Set("Content-Type", "application/json")
            fmt.Fprintf(w, `{
                "message": "Hello there from backend server 2!",
                "server": "backend-2",
                "path": "%s",
                "timestamp": "%s"
            }`, r.URL.Path, time.Now().Format(time.RFC3339))
        })
        
        log.Println("Backend server 2 starting on :3002")
        http.ListenAndServe(":3002", mux)
    }()
    
    // Backend server 3 (sometimes fails)
    go func() {
        mux := http.NewServeMux()
        mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
            // Simulate occasional failures
            if time.Now().Second()%7 == 0 {
                http.Error(w, "Backend server 3 temporary failure", 
                          http.StatusInternalServerError)
                return
            }
            
            w.Header().Set("Content-Type", "application/json")
            fmt.Fprintf(w, `{
                "message": "Hello there from backend server 3!",
                "server": "backend-3",
                "path": "%s",
                "timestamp": "%s"
            }`, r.URL.Path, time.Now().Format(time.RFC3339))
        })
        
        log.Println("Backend server 3 starting on :3003")
        http.ListenAndServe(":3003", mux)
    }()
}

func main() {
    stats.BackendRequests = make(map[string]int64)
    
    // Create backend servers
    createBackendServers()
    
    // Wait for backend servers to start
    time.Sleep(1 * time.Second)
    
    // Configure backends
    backends := []string{
        "http://localhost:3001",
        "http://localhost:3002", 
        "http://localhost:3003",
    }
    
    for _, backend := range backends {
        url, err := url.Parse(backend)
        if err != nil {
            log.Fatal(err)
        }
        
        proxy := createReverseProxy(url)
        
        serverPool.AddBackend(&Backend{
            URL:          url,
            Alive:        true,
            ReverseProxy: proxy,
        })
        
        stats.BackendRequests[backend] = 0
    }
    
    // Start health checking
    go healthCheck()
    
    server := http.Server{
        Addr:    ":8080",
        Handler: http.HandlerFunc(proxyHandler),
    }
    
    fmt.Println("=== HTTP Proxy Server Demo ===")
    fmt.Println("Proxy server starting on http://localhost:8080")
    fmt.Println("Backend servers:")
    fmt.Println("  - http://localhost:3001")
    fmt.Println("  - http://localhost:3002")
    fmt.Println("  - http://localhost:3003")
    fmt.Println()
    fmt.Println("Test endpoints:")
    fmt.Println("  curl http://localhost:8080/")
    fmt.Println("  curl http://localhost:8080/stats")
    fmt.Println("  curl http://localhost:8080/health")
    fmt.Println()
    fmt.Println("Features:")
    fmt.Println("• Round-robin load balancing")
    fmt.Println("• Health checking and failover")
    fmt.Println("• Request/response modification")
    fmt.Println("• Statistics and monitoring")
    
    log.Fatal(server.ListenAndServe())
}
```

This proxy server example demonstrates reverse proxying, load balancing,  
health checking, and request/response modification. The implementation  
shows how to build a production-ready proxy with failover capabilities  
and monitoring features essential for distributed systems.

## HTTP caching implementation

HTTP caching improves performance by storing and reusing responses. This  
example demonstrates various caching strategies including in-memory cache,  
cache headers, and conditional requests.

```go
package main

import (
    "crypto/md5"
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
    "strings"
    "sync"
    "time"
)

type CacheEntry struct {
    Data        []byte
    Headers     http.Header
    StatusCode  int
    ETag        string
    LastModified time.Time
    Expires     time.Time
    CreatedAt   time.Time
}

type Cache struct {
    entries map[string]*CacheEntry
    mutex   sync.RWMutex
    maxSize int
    hits    int64
    misses  int64
}

type CacheStats struct {
    Hits      int64
    Misses    int64
    Entries   int
    HitRatio  float64
}

func NewCache(maxSize int) *Cache {
    return &Cache{
        entries: make(map[string]*CacheEntry),
        maxSize: maxSize,
    }
}

func (c *Cache) generateKey(r *http.Request) string {
    return fmt.Sprintf("%s:%s:%s", r.Method, r.Host, r.URL.String())
}

func (c *Cache) generateETag(data []byte) string {
    hash := md5.Sum(data)
    return fmt.Sprintf(`"%x"`, hash)
}

func (c *Cache) isExpired(entry *CacheEntry) bool {
    return time.Now().After(entry.Expires)
}

func (c *Cache) Get(key string) (*CacheEntry, bool) {
    c.mutex.RLock()
    defer c.mutex.RUnlock()
    
    entry, exists := c.entries[key]
    if !exists {
        c.misses++
        return nil, false
    }
    
    if c.isExpired(entry) {
        c.misses++
        return nil, false
    }
    
    c.hits++
    return entry, true
}

func (c *Cache) Set(key string, entry *CacheEntry) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    // Simple eviction: remove oldest entries if cache is full
    if len(c.entries) >= c.maxSize {
        c.evictOldest()
    }
    
    c.entries[key] = entry
}

func (c *Cache) evictOldest() {
    var oldestKey string
    var oldestTime time.Time
    
    for key, entry := range c.entries {
        if oldestKey == "" || entry.CreatedAt.Before(oldestTime) {
            oldestKey = key
            oldestTime = entry.CreatedAt
        }
    }
    
    if oldestKey != "" {
        delete(c.entries, oldestKey)
    }
}

func (c *Cache) Delete(key string) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    delete(c.entries, key)
}

func (c *Cache) GetStats() CacheStats {
    c.mutex.RLock()
    defer c.mutex.RUnlock()
    
    total := c.hits + c.misses
    hitRatio := float64(0)
    if total > 0 {
        hitRatio = float64(c.hits) / float64(total) * 100
    }
    
    return CacheStats{
        Hits:     c.hits,
        Misses:   c.misses,
        Entries:  len(c.entries),
        HitRatio: hitRatio,
    }
}

func (c *Cache) Clear() {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    c.entries = make(map[string]*CacheEntry)
    c.hits = 0
    c.misses = 0
}

type CachingHandler struct {
    cache   *Cache
    handler http.Handler
}

func NewCachingHandler(cache *Cache, handler http.Handler) *CachingHandler {
    return &CachingHandler{
        cache:   cache,
        handler: handler,
    }
}

func (ch *CachingHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    // Only cache GET requests
    if r.Method != http.MethodGet {
        ch.handler.ServeHTTP(w, r)
        return
    }
    
    key := ch.cache.generateKey(r)
    
    // Check for cached entry
    if entry, found := ch.cache.Get(key); found {
        // Handle conditional requests
        ifNoneMatch := r.Header.Get("If-None-Match")
        ifModifiedSince := r.Header.Get("If-Modified-Since")
        
        if ifNoneMatch != "" && ifNoneMatch == entry.ETag {
            w.WriteHeader(http.StatusNotModified)
            return
        }
        
        if ifModifiedSince != "" {
            if modTime, err := time.Parse(http.TimeFormat, ifModifiedSince); err == nil {
                if !entry.LastModified.After(modTime) {
                    w.WriteHeader(http.StatusNotModified)
                    return
                }
            }
        }
        
        // Serve from cache
        ch.serveCachedResponse(w, entry)
        return
    }
    
    // Not in cache, capture response
    recorder := &ResponseRecorder{
        ResponseWriter: w,
        statusCode:     http.StatusOK,
        headers:        make(http.Header),
    }
    
    ch.handler.ServeHTTP(recorder, r)
    
    // Cache the response if appropriate
    if ch.shouldCache(recorder.statusCode, recorder.headers) {
        entry := &CacheEntry{
            Data:         recorder.body.Bytes(),
            Headers:      recorder.headers,
            StatusCode:   recorder.statusCode,
            ETag:         ch.cache.generateETag(recorder.body.Bytes()),
            LastModified: time.Now(),
            Expires:      ch.calculateExpiry(recorder.headers),
            CreatedAt:    time.Now(),
        }
        
        ch.cache.Set(key, entry)
    }
}

func (ch *CachingHandler) shouldCache(statusCode int, headers http.Header) bool {
    // Only cache successful responses
    if statusCode != http.StatusOK {
        return false
    }
    
    // Don't cache if explicitly told not to
    cacheControl := headers.Get("Cache-Control")
    if strings.Contains(cacheControl, "no-cache") || 
       strings.Contains(cacheControl, "no-store") {
        return false
    }
    
    return true
}

func (ch *CachingHandler) calculateExpiry(headers http.Header) time.Time {
    // Check for explicit expiry
    if expires := headers.Get("Expires"); expires != "" {
        if expTime, err := time.Parse(http.TimeFormat, expires); err == nil {
            return expTime
        }
    }
    
    // Check for Cache-Control max-age
    cacheControl := headers.Get("Cache-Control")
    if strings.Contains(cacheControl, "max-age=") {
        parts := strings.Split(cacheControl, "max-age=")
        if len(parts) > 1 {
            ageStr := strings.Split(parts[1], ",")[0]
            if age, err := strconv.Atoi(strings.TrimSpace(ageStr)); err == nil {
                return time.Now().Add(time.Duration(age) * time.Second)
            }
        }
    }
    
    // Default cache duration
    return time.Now().Add(5 * time.Minute)
}

func (ch *CachingHandler) serveCachedResponse(w http.ResponseWriter, entry *CacheEntry) {
    // Copy headers
    for key, values := range entry.Headers {
        for _, value := range values {
            w.Header().Add(key, value)
        }
    }
    
    // Add cache headers
    w.Header().Set("ETag", entry.ETag)
    w.Header().Set("Last-Modified", entry.LastModified.Format(http.TimeFormat))
    w.Header().Set("X-Cache", "HIT")
    
    w.WriteHeader(entry.StatusCode)
    w.Write(entry.Data)
}

type ResponseRecorder struct {
    http.ResponseWriter
    statusCode int
    headers    http.Header
    body       strings.Builder
}

func (rr *ResponseRecorder) WriteHeader(statusCode int) {
    rr.statusCode = statusCode
    rr.ResponseWriter.WriteHeader(statusCode)
}

func (rr *ResponseRecorder) Write(data []byte) (int, error) {
    rr.body.Write(data)
    return rr.ResponseWriter.Write(data)
}

func (rr *ResponseRecorder) Header() http.Header {
    // Capture headers being set
    original := rr.ResponseWriter.Header()
    for key, values := range original {
        rr.headers[key] = values
    }
    return original
}

func createDemoContent() *http.ServeMux {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "text/html")
        w.Header().Set("Cache-Control", "max-age=300") // 5 minutes
        
        fmt.Fprintf(w, `
        <html>
        <head><title>HTTP Caching Demo</title></head>
        <body>
            <h1>Hello there!</h1>
            <p>This is a caching demonstration.</p>
            <p>Generated at: %s</p>
            <ul>
                <li><a href="/cached">Cached content (5 min)</a></li>
                <li><a href="/no-cache">Non-cached content</a></li>
                <li><a href="/stats">Cache statistics</a></li>
                <li><a href="/clear-cache">Clear cache</a></li>
            </ul>
        </body>
        </html>`, time.Now().Format(time.RFC3339))
    })
    
    mux.HandleFunc("/cached", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Cache-Control", "max-age=300")
        
        fmt.Fprintf(w, `{
            "message": "Hello there! This response is cached.",
            "timestamp": "%s",
            "cache_duration": "5 minutes",
            "random": %d
        }`, time.Now().Format(time.RFC3339), time.Now().UnixNano()%1000)
    })
    
    mux.HandleFunc("/no-cache", func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Cache-Control", "no-cache, no-store")
        
        fmt.Fprintf(w, `{
            "message": "Hello there! This response is never cached.",
            "timestamp": "%s",
            "random": %d
        }`, time.Now().Format(time.RFC3339), time.Now().UnixNano()%1000)
    })
    
    return mux
}

func statsHandler(cache *Cache) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        stats := cache.GetStats()
        
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Cache-Control", "no-cache")
        
        fmt.Fprintf(w, `{
            "cache_hits": %d,
            "cache_misses": %d,
            "hit_ratio": %.2f,
            "cached_entries": %d,
            "timestamp": "%s"
        }`, stats.Hits, stats.Misses, stats.HitRatio, 
            stats.Entries, time.Now().Format(time.RFC3339))
    }
}

func clearCacheHandler(cache *Cache) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        cache.Clear()
        
        w.Header().Set("Content-Type", "application/json")
        fmt.Fprintf(w, `{
            "message": "Cache cleared successfully",
            "timestamp": "%s"
        }`, time.Now().Format(time.RFC3339))
    }
}

func main() {
    cache := NewCache(1000)
    
    // Create demo content handlers
    contentHandler := createDemoContent()
    
    // Wrap with caching
    cachingHandler := NewCachingHandler(cache, contentHandler)
    
    // Add cache management endpoints
    http.Handle("/", cachingHandler)
    http.HandleFunc("/stats", statsHandler(cache))
    http.HandleFunc("/clear-cache", clearCacheHandler(cache))
    
    fmt.Println("=== HTTP Caching Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Test caching behavior:")
    fmt.Println("  curl -v http://localhost:8080/cached")
    fmt.Println("  curl -v http://localhost:8080/cached  # Should be from cache")
    fmt.Println("  curl -H 'If-None-Match: \"etag\"' http://localhost:8080/cached")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• In-memory response caching")
    fmt.Println("• ETag and Last-Modified support")
    fmt.Println("• Conditional request handling")
    fmt.Println("• Cache-Control header parsing")
    fmt.Println("• Cache statistics and management")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This comprehensive caching example demonstrates in-memory caching, conditional  
requests, cache headers, and cache management. The implementation shows how  
to build a production-ready caching layer that respects HTTP caching  
semantics while providing significant performance improvements.

## HTTP PATCH method

PATCH requests allow partial updates to resources, sending only the fields  
that need to be changed. This example demonstrates proper PATCH handling  
with JSON Patch and merge patch strategies.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type Product struct {
    ID          int       `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    Price       float64   `json:"price"`
    Category    string    `json:"category"`
    InStock     bool      `json:"in_stock"`
    Tags        []string  `json:"tags"`
    CreatedAt   time.Time `json:"created_at"`
    UpdatedAt   time.Time `json:"updated_at"`
}

type PatchOperation struct {
    Op    string      `json:"op"`
    Path  string      `json:"path"`
    Value interface{} `json:"value"`
}

var products = map[int]*Product{
    1: {
        ID:          1,
        Name:        "Wireless Headphones",
        Description: "High-quality wireless headphones",
        Price:       199.99,
        Category:    "electronics",
        InStock:     true,
        Tags:        []string{"audio", "wireless", "premium"},
        CreatedAt:   time.Now().Add(-24 * time.Hour),
        UpdatedAt:   time.Now().Add(-24 * time.Hour),
    },
    2: {
        ID:          2,
        Name:        "Go Programming Book",
        Description: "Learn Go programming language",
        Price:       49.99,
        Category:    "books",
        InStock:     false,
        Tags:        []string{"programming", "go", "education"},
        CreatedAt:   time.Now().Add(-48 * time.Hour),
        UpdatedAt:   time.Now().Add(-12 * time.Hour),
    },
}

func patchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPatch {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    // Extract product ID from URL
    path := strings.TrimPrefix(r.URL.Path, "/products/")
    productID, err := strconv.Atoi(path)
    if err != nil {
        http.Error(w, "Invalid product ID", http.StatusBadRequest)
        return
    }

    product, exists := products[productID]
    if !exists {
        http.Error(w, "Product not found", http.StatusNotFound)
        return
    }

    contentType := r.Header.Get("Content-Type")
    
    switch {
    case strings.Contains(contentType, "application/json-patch+json"):
        handleJSONPatch(w, r, product)
    case strings.Contains(contentType, "application/merge-patch+json"):
        handleMergePatch(w, r, product)
    case strings.Contains(contentType, "application/json"):
        handleMergePatch(w, r, product) // Default to merge patch
    default:
        http.Error(w, "Unsupported content type", http.StatusUnsupportedMediaType)
    }
}

func handleJSONPatch(w http.ResponseWriter, r *http.Request, product *Product) {
    var operations []PatchOperation
    if err := json.NewDecoder(r.Body).Decode(&operations); err != nil {
        http.Error(w, "Invalid JSON patch format", http.StatusBadRequest)
        return
    }

    // Apply operations
    for _, op := range operations {
        if err := applyPatchOperation(product, op); err != nil {
            http.Error(w, fmt.Sprintf("Patch operation failed: %v", err), 
                      http.StatusBadRequest)
            return
        }
    }

    product.UpdatedAt = time.Now()
    sendJSONResponse(w, product)
}

func handleMergePatch(w http.ResponseWriter, r *http.Request, product *Product) {
    var updates map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&updates); err != nil {
        http.Error(w, "Invalid JSON format", http.StatusBadRequest)
        return
    }

    // Apply merge patch
    if err := applyMergeUpdates(product, updates); err != nil {
        http.Error(w, fmt.Sprintf("Merge patch failed: %v", err), 
                  http.StatusBadRequest)
        return
    }

    product.UpdatedAt = time.Now()
    sendJSONResponse(w, product)
}

func applyPatchOperation(product *Product, op PatchOperation) error {
    switch op.Op {
    case "replace":
        return applyReplace(product, op.Path, op.Value)
    case "add":
        return applyAdd(product, op.Path, op.Value)
    case "remove":
        return applyRemove(product, op.Path)
    case "test":
        return applyTest(product, op.Path, op.Value)
    default:
        return fmt.Errorf("unsupported operation: %s", op.Op)
    }
}

func applyReplace(product *Product, path string, value interface{}) error {
    switch path {
    case "/name":
        if name, ok := value.(string); ok {
            product.Name = name
        } else {
            return fmt.Errorf("invalid value type for name")
        }
    case "/description":
        if desc, ok := value.(string); ok {
            product.Description = desc
        } else {
            return fmt.Errorf("invalid value type for description")
        }
    case "/price":
        if price, ok := value.(float64); ok {
            product.Price = price
        } else {
            return fmt.Errorf("invalid value type for price")
        }
    case "/category":
        if category, ok := value.(string); ok {
            product.Category = category
        } else {
            return fmt.Errorf("invalid value type for category")
        }
    case "/in_stock":
        if inStock, ok := value.(bool); ok {
            product.InStock = inStock
        } else {
            return fmt.Errorf("invalid value type for in_stock")
        }
    case "/tags":
        if tags, ok := value.([]interface{}); ok {
            stringTags := make([]string, len(tags))
            for i, tag := range tags {
                if str, ok := tag.(string); ok {
                    stringTags[i] = str
                } else {
                    return fmt.Errorf("invalid tag type")
                }
            }
            product.Tags = stringTags
        } else {
            return fmt.Errorf("invalid value type for tags")
        }
    default:
        return fmt.Errorf("unsupported path: %s", path)
    }
    return nil
}

func applyAdd(product *Product, path string, value interface{}) error {
    if strings.HasPrefix(path, "/tags/") {
        // Add to tags array
        if tag, ok := value.(string); ok {
            product.Tags = append(product.Tags, tag)
        } else {
            return fmt.Errorf("invalid tag value")
        }
    } else {
        // For other fields, add is same as replace
        return applyReplace(product, path, value)
    }
    return nil
}

func applyRemove(product *Product, path string) error {
    if strings.HasPrefix(path, "/tags/") {
        // Remove from tags array by index or value
        indexStr := strings.TrimPrefix(path, "/tags/")
        if index, err := strconv.Atoi(indexStr); err == nil {
            if index >= 0 && index < len(product.Tags) {
                product.Tags = append(product.Tags[:index], product.Tags[index+1:]...)
            }
        } else {
            // Remove by value
            for i, tag := range product.Tags {
                if tag == indexStr {
                    product.Tags = append(product.Tags[:i], product.Tags[i+1:]...)
                    break
                }
            }
        }
    } else {
        return fmt.Errorf("cannot remove required field: %s", path)
    }
    return nil
}

func applyTest(product *Product, path string, expected interface{}) error {
    var actual interface{}
    
    switch path {
    case "/name":
        actual = product.Name
    case "/price":
        actual = product.Price
    case "/in_stock":
        actual = product.InStock
    default:
        return fmt.Errorf("test not supported for path: %s", path)
    }
    
    if actual != expected {
        return fmt.Errorf("test failed: expected %v, got %v", expected, actual)
    }
    
    return nil
}

func applyMergeUpdates(product *Product, updates map[string]interface{}) error {
    for field, value := range updates {
        switch field {
        case "name":
            if name, ok := value.(string); ok {
                product.Name = name
            } else {
                return fmt.Errorf("invalid value for name")
            }
        case "description":
            if desc, ok := value.(string); ok {
                product.Description = desc
            } else {
                return fmt.Errorf("invalid value for description")
            }
        case "price":
            if price, ok := value.(float64); ok {
                product.Price = price
            } else {
                return fmt.Errorf("invalid value for price")
            }
        case "category":
            if category, ok := value.(string); ok {
                product.Category = category
            } else {
                return fmt.Errorf("invalid value for category")
            }
        case "in_stock":
            if inStock, ok := value.(bool); ok {
                product.InStock = inStock
            } else {
                return fmt.Errorf("invalid value for in_stock")
            }
        case "tags":
            if tags, ok := value.([]interface{}); ok {
                stringTags := make([]string, len(tags))
                for i, tag := range tags {
                    if str, ok := tag.(string); ok {
                        stringTags[i] = str
                    } else {
                        return fmt.Errorf("invalid tag type")
                    }
                }
                product.Tags = stringTags
            } else {
                return fmt.Errorf("invalid value for tags")
            }
        case "id", "created_at":
            // Ignore immutable fields
            continue
        default:
            return fmt.Errorf("unknown field: %s", field)
        }
    }
    return nil
}

func sendJSONResponse(w http.ResponseWriter, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
}

func getProductHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    path := strings.TrimPrefix(r.URL.Path, "/products/")
    productID, err := strconv.Atoi(path)
    if err != nil {
        http.Error(w, "Invalid product ID", http.StatusBadRequest)
        return
    }

    product, exists := products[productID]
    if !exists {
        http.Error(w, "Product not found", http.StatusNotFound)
        return
    }

    sendJSONResponse(w, product)
}

func listProductsHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    productList := make([]*Product, 0, len(products))
    for _, product := range products {
        productList = append(productList, product)
    }

    sendJSONResponse(w, map[string]interface{}{
        "products": productList,
        "count":    len(productList),
    })
}

func demoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head><title>PATCH Method Demo</title></head>
    <body>
        <h1>HTTP PATCH Method Demo</h1>
        <p>Hello there! This demonstrates PATCH operations.</p>
        
        <h2>Available Products:</h2>
        <ul>
            <li><a href="/products/1">Product 1 - Wireless Headphones</a></li>
            <li><a href="/products/2">Product 2 - Go Programming Book</a></li>
        </ul>
        
        <h2>Example PATCH Operations:</h2>
        
        <h3>JSON Patch (RFC 6901):</h3>
        <pre>
curl -X PATCH http://localhost:8080/products/1 \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {"op": "test", "path": "/price", "value": 199.99},
    {"op": "replace", "path": "/price", "value": 179.99},
    {"op": "add", "path": "/tags", "value": "sale"}
  ]'
        </pre>
        
        <h3>Merge Patch (RFC 7396):</h3>
        <pre>
curl -X PATCH http://localhost:8080/products/1 \
  -H "Content-Type: application/merge-patch+json" \
  -d '{
    "price": 189.99,
    "in_stock": false,
    "description": "Updated description"
  }'
        </pre>
        
        <h3>Operations Supported:</h3>
        <ul>
            <li><strong>test:</strong> Verify current value</li>
            <li><strong>replace:</strong> Update field value</li>
            <li><strong>add:</strong> Add to arrays or set field</li>
            <li><strong>remove:</strong> Remove from arrays</li>
        </ul>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", demoHandler)
    http.HandleFunc("/products", listProductsHandler)
    http.HandleFunc("/products/", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet:
            getProductHandler(w, r)
        case http.MethodPatch:
            patchHandler(w, r)
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })

    fmt.Println("=== HTTP PATCH Method Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• JSON Patch (RFC 6901) operations")
    fmt.Println("• Merge Patch (RFC 7396) updates")
    fmt.Println("• Partial resource updates")
    fmt.Println("• Test operations for validation")
    fmt.Println("• Array manipulation (add/remove)")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

The PATCH method example demonstrates proper handling of partial updates  
using both JSON Patch and Merge Patch standards. The implementation shows  
how to safely apply incremental changes to resources while maintaining  
data integrity and supporting various patch operations.

## HTTP HEAD method

HEAD requests return only headers without the response body, useful for  
checking resource metadata, cache validation, and content size. This  
example demonstrates proper HEAD method implementation.

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "strconv"
    "time"
)

type ResourceInfo struct {
    Size         int64
    LastModified time.Time
    ContentType  string
    ETag         string
}

var resources = map[string]ResourceInfo{
    "document.pdf": {
        Size:         1048576, // 1MB
        LastModified: time.Now().Add(-24 * time.Hour),
        ContentType:  "application/pdf",
        ETag:         `"doc-v1.0"`,
    },
    "image.jpg": {
        Size:         2097152, // 2MB
        LastModified: time.Now().Add(-12 * time.Hour),
        ContentType:  "image/jpeg",
        ETag:         `"img-v2.1"`,
    },
    "data.json": {
        Size:         4096, // 4KB
        LastModified: time.Now().Add(-6 * time.Hour),
        ContentType:  "application/json",
        ETag:         `"data-v3.0"`,
    },
}

func headHandler(w http.ResponseWriter, r *http.Request) {
    resource := r.URL.Query().Get("resource")
    if resource == "" {
        http.Error(w, "Resource parameter required", http.StatusBadRequest)
        return
    }

    info, exists := resources[resource]
    if !exists {
        w.WriteHeader(http.StatusNotFound)
        return
    }

    // Set all the headers that would be present in a GET response
    w.Header().Set("Content-Type", info.ContentType)
    w.Header().Set("Content-Length", strconv.FormatInt(info.Size, 10))
    w.Header().Set("Last-Modified", info.LastModified.Format(http.TimeFormat))
    w.Header().Set("ETag", info.ETag)
    w.Header().Set("Accept-Ranges", "bytes")
    w.Header().Set("Cache-Control", "public, max-age=3600")

    // HEAD method should only return headers, no body
    if r.Method == http.MethodHead {
        w.WriteHeader(http.StatusOK)
        return
    }

    // For GET requests, we would normally return the actual content
    // Here we simulate by returning metadata
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, `{
        "resource": "%s",
        "size": %d,
        "content_type": "%s",
        "last_modified": "%s",
        "etag": %s,
        "message": "This would be the actual resource content"
    }`, resource, info.Size, info.ContentType, 
        info.LastModified.Format(time.RFC3339), info.ETag)
}

func downloadHandler(w http.ResponseWriter, r *http.Request) {
    resource := r.URL.Query().Get("resource")
    if resource == "" {
        http.Error(w, "Resource parameter required", http.StatusBadRequest)
        return
    }

    info, exists := resources[resource]
    if !exists {
        w.WriteHeader(http.StatusNotFound)
        return
    }

    // Set headers
    w.Header().Set("Content-Type", info.ContentType)
    w.Header().Set("Content-Length", strconv.FormatInt(info.Size, 10))
    w.Header().Set("Last-Modified", info.LastModified.Format(http.TimeFormat))
    w.Header().Set("ETag", info.ETag)
    w.Header().Set("Content-Disposition", 
                   fmt.Sprintf("attachment; filename=\"%s\"", resource))

    // Handle conditional requests
    ifModifiedSince := r.Header.Get("If-Modified-Since")
    if ifModifiedSince != "" {
        if modTime, err := time.Parse(http.TimeFormat, ifModifiedSince); err == nil {
            if !info.LastModified.After(modTime) {
                w.WriteHeader(http.StatusNotModified)
                return
            }
        }
    }

    ifNoneMatch := r.Header.Get("If-None-Match")
    if ifNoneMatch != "" && ifNoneMatch == info.ETag {
        w.WriteHeader(http.StatusNotModified)
        return
    }

    // For demonstration, create simulated content
    w.WriteHeader(http.StatusOK)
    
    if r.Method == http.MethodHead {
        return // Don't write body for HEAD requests
    }

    // Simulate file content based on type
    switch info.ContentType {
    case "application/pdf":
        w.Write([]byte("%PDF-1.4\n1 0 obj\n<<\n/Type /Catalog\n>>"))
    case "image/jpeg":
        w.Write([]byte("\xFF\xD8\xFF\xE0\x00\x10JFIF")) // JPEG header
    case "application/json":
        fmt.Fprintf(w, `{
            "resource": "%s",
            "timestamp": "%s",
            "data": "Sample JSON content"
        }`, resource, time.Now().Format(time.RFC3339))
    default:
        fmt.Fprintf(w, "Content for %s", resource)
    }
}

func bulkHeadHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodHead && r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    // Calculate total size and latest modification time
    var totalSize int64
    var latestMod time.Time
    
    for _, info := range resources {
        totalSize += info.Size
        if info.LastModified.After(latestMod) {
            latestMod = info.LastModified
        }
    }

    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Content-Length", strconv.FormatInt(totalSize, 10))
    w.Header().Set("Last-Modified", latestMod.Format(http.TimeFormat))
    w.Header().Set("X-Resource-Count", strconv.Itoa(len(resources)))
    w.Header().Set("X-Total-Size", strconv.FormatInt(totalSize, 10))

    if r.Method == http.MethodHead {
        w.WriteHeader(http.StatusOK)
        return
    }

    // Return summary for GET requests
    w.WriteHeader(http.StatusOK)
    fmt.Fprintf(w, `{
        "total_resources": %d,
        "total_size": %d,
        "latest_modification": "%s",
        "resources": {`, len(resources), totalSize, latestMod.Format(time.RFC3339))

    first := true
    for name, info := range resources {
        if !first {
            fmt.Fprint(w, ",")
        }
        fmt.Fprintf(w, `
            "%s": {
                "size": %d,
                "type": "%s",
                "modified": "%s"
            }`, name, info.Size, info.ContentType, 
            info.LastModified.Format(time.RFC3339))
        first = false
    }

    fmt.Fprint(w, `
        }
    }`)
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head><title>HTTP HEAD Method Demo</title></head>
    <body>
        <h1>HTTP HEAD Method Demo</h1>
        <p>Hello there! This demonstrates HEAD method usage.</p>
        
        <h2>Available Resources:</h2>
        <ul>
            <li>document.pdf (1MB PDF file)</li>
            <li>image.jpg (2MB JPEG image)</li>
            <li>data.json (4KB JSON data)</li>
        </ul>
        
        <h2>Test Commands:</h2>
        
        <h3>Check resource metadata (HEAD):</h3>
        <pre>
curl -I "http://localhost:8080/resource?resource=document.pdf"
curl -I "http://localhost:8080/resource?resource=image.jpg"
curl -I "http://localhost:8080/resource?resource=data.json"
        </pre>
        
        <h3>Get resource content (GET):</h3>
        <pre>
curl "http://localhost:8080/resource?resource=data.json"
        </pre>
        
        <h3>Download with conditional headers:</h3>
        <pre>
curl -I -H "If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT" \
  "http://localhost:8080/resource?resource=document.pdf"
        </pre>
        
        <h3>Bulk resource info:</h3>
        <pre>
curl -I "http://localhost:8080/bulk"
curl "http://localhost:8080/bulk"
        </pre>
        
        <h2>HEAD Method Benefits:</h2>
        <ul>
            <li>Check resource existence without downloading</li>
            <li>Verify content size before download</li>
            <li>Cache validation with ETags and Last-Modified</li>
            <li>Bandwidth efficient metadata queries</li>
            <li>Pre-flight checks for large downloads</li>
        </ul>
        
        <h2>Response Headers Explained:</h2>
        <ul>
            <li><strong>Content-Length:</strong> Size of the resource</li>
            <li><strong>Content-Type:</strong> MIME type of the resource</li>
            <li><strong>Last-Modified:</strong> When resource was last changed</li>
            <li><strong>ETag:</strong> Version identifier for caching</li>
            <li><strong>Accept-Ranges:</strong> Whether range requests are supported</li>
        </ul>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", demoPageHandler)
    http.HandleFunc("/resource", func(w http.ResponseWriter, r *http.Request) {
        if r.Method == http.MethodHead || r.Method == http.MethodGet {
            headHandler(w, r)
        } else {
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })
    http.HandleFunc("/download", func(w http.ResponseWriter, r *http.Request) {
        if r.Method == http.MethodHead || r.Method == http.MethodGet {
            downloadHandler(w, r)
        } else {
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })
    http.HandleFunc("/bulk", bulkHeadHandler)

    fmt.Println("=== HTTP HEAD Method Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("The HEAD method allows clients to:")
    fmt.Println("• Get response headers without the body")
    fmt.Println("• Check resource size before downloading")
    fmt.Println("• Validate cache entries efficiently")
    fmt.Println("• Test resource availability quickly")
    fmt.Println("• Save bandwidth on metadata queries")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

The HEAD method example demonstrates efficient resource metadata retrieval,  
cache validation, and bandwidth conservation. The implementation shows how  
HEAD requests should return identical headers to GET requests but without  
the response body, making them perfect for pre-flight checks and caching.

## HTTP OPTIONS method

OPTIONS requests discover allowed HTTP methods and CORS policies for  
resources. This example demonstrates proper OPTIONS handling and CORS  
preflight request support.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"
)

type CORSConfig struct {
    AllowedOrigins   []string
    AllowedMethods   []string
    AllowedHeaders   []string
    ExposedHeaders   []string
    AllowCredentials bool
    MaxAge           int
}

type EndpointInfo struct {
    Path           string
    Methods        []string
    Description    string
    Parameters     map[string]string
    Authentication bool
    CORS           CORSConfig
}

var endpoints = map[string]EndpointInfo{
    "/api/users": {
        Path:        "/api/users",
        Methods:     []string{"GET", "POST", "OPTIONS"},
        Description: "User management endpoint",
        Parameters: map[string]string{
            "GET":  "?page=1&limit=10",
            "POST": "JSON body with user data",
        },
        Authentication: true,
        CORS: CORSConfig{
            AllowedOrigins:   []string{"https://example.com", "https://app.example.com"},
            AllowedMethods:   []string{"GET", "POST", "OPTIONS"},
            AllowedHeaders:   []string{"Content-Type", "Authorization", "X-Requested-With"},
            ExposedHeaders:   []string{"X-Total-Count", "X-Page-Count"},
            AllowCredentials: true,
            MaxAge:           86400,
        },
    },
    "/api/users/profile": {
        Path:        "/api/users/profile",
        Methods:     []string{"GET", "PUT", "PATCH", "DELETE", "OPTIONS"},
        Description: "User profile management",
        Parameters: map[string]string{
            "GET":    "No parameters",
            "PUT":    "JSON body with complete profile",
            "PATCH":  "JSON body with partial updates",
            "DELETE": "No parameters",
        },
        Authentication: true,
        CORS: CORSConfig{
            AllowedOrigins:   []string{"*"},
            AllowedMethods:   []string{"GET", "PUT", "PATCH", "DELETE", "OPTIONS"},
            AllowedHeaders:   []string{"*"},
            ExposedHeaders:   []string{"X-User-ID"},
            AllowCredentials: false,
            MaxAge:           3600,
        },
    },
    "/api/public/status": {
        Path:        "/api/public/status",
        Methods:     []string{"GET", "HEAD", "OPTIONS"},
        Description: "Public status endpoint",
        Parameters: map[string]string{
            "GET":  "No parameters",
            "HEAD": "Headers only",
        },
        Authentication: false,
        CORS: CORSConfig{
            AllowedOrigins: []string{"*"},
            AllowedMethods: []string{"GET", "HEAD", "OPTIONS"},
            AllowedHeaders: []string{"Content-Type"},
            MaxAge:         300,
        },
    },
}

func optionsHandler(w http.ResponseWriter, r *http.Request) {
    path := r.URL.Path
    
    // Find matching endpoint
    endpoint, exists := endpoints[path]
    if !exists {
        // Check for wildcard or pattern matching
        endpoint = findBestMatch(path)
        if endpoint.Path == "" {
            w.WriteHeader(http.StatusNotFound)
            return
        }
    }
    
    // Set CORS headers
    setCORSHeaders(w, r, endpoint.CORS)
    
    // Set allowed methods
    w.Header().Set("Allow", strings.Join(endpoint.Methods, ", "))
    
    // Set additional options headers
    w.Header().Set("Accept", "application/json, text/plain, */*")
    w.Header().Set("Accept-Encoding", "gzip, deflate, br")
    w.Header().Set("Accept-Language", "en-US,en;q=0.9")
    
    // Set content type options
    w.Header().Set("X-Content-Type-Options", "nosniff")
    
    // Return detailed endpoint information for development
    if r.Header.Get("X-Debug") == "true" {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusOK)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "endpoint":     endpoint,
            "cors_policy":  endpoint.CORS,
            "request_info": getRequestInfo(r),
        })
        return
    }
    
    w.WriteHeader(http.StatusNoContent)
}

func setCORSHeaders(w http.ResponseWriter, r *http.Request, config CORSConfig) {
    origin := r.Header.Get("Origin")
    
    // Check if origin is allowed
    if isOriginAllowed(origin, config.AllowedOrigins) {
        w.Header().Set("Access-Control-Allow-Origin", origin)
    } else if len(config.AllowedOrigins) == 1 && config.AllowedOrigins[0] == "*" {
        w.Header().Set("Access-Control-Allow-Origin", "*")
    }
    
    // Set allowed methods
    if len(config.AllowedMethods) > 0 {
        w.Header().Set("Access-Control-Allow-Methods", 
                      strings.Join(config.AllowedMethods, ", "))
    }
    
    // Set allowed headers
    if len(config.AllowedHeaders) > 0 {
        if len(config.AllowedHeaders) == 1 && config.AllowedHeaders[0] == "*" {
            requestedHeaders := r.Header.Get("Access-Control-Request-Headers")
            if requestedHeaders != "" {
                w.Header().Set("Access-Control-Allow-Headers", requestedHeaders)
            }
        } else {
            w.Header().Set("Access-Control-Allow-Headers", 
                          strings.Join(config.AllowedHeaders, ", "))
        }
    }
    
    // Set exposed headers
    if len(config.ExposedHeaders) > 0 {
        w.Header().Set("Access-Control-Expose-Headers", 
                      strings.Join(config.ExposedHeaders, ", "))
    }
    
    // Set credentials
    if config.AllowCredentials {
        w.Header().Set("Access-Control-Allow-Credentials", "true")
    }
    
    // Set max age for preflight caching
    if config.MaxAge > 0 {
        w.Header().Set("Access-Control-Max-Age", fmt.Sprintf("%d", config.MaxAge))
    }
}

func isOriginAllowed(origin string, allowedOrigins []string) bool {
    for _, allowed := range allowedOrigins {
        if allowed == "*" || allowed == origin {
            return true
        }
        // Could add wildcard matching here
    }
    return false
}

func findBestMatch(path string) EndpointInfo {
    // Simple pattern matching - in production, use a router
    for endpointPath, info := range endpoints {
        if strings.HasPrefix(path, strings.TrimSuffix(endpointPath, "/")) {
            return info
        }
    }
    return EndpointInfo{}
}

func getRequestInfo(r *http.Request) map[string]interface{} {
    return map[string]interface{}{
        "method":  r.Method,
        "path":    r.URL.Path,
        "origin":  r.Header.Get("Origin"),
        "user_agent": r.Header.Get("User-Agent"),
        "requested_method": r.Header.Get("Access-Control-Request-Method"),
        "requested_headers": r.Header.Get("Access-Control-Request-Headers"),
        "timestamp": time.Now().Format(time.RFC3339),
    }
}

func corsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Handle preflight requests
        if r.Method == http.MethodOptions {
            optionsHandler(w, r)
            return
        }
        
        // Find endpoint config for actual requests
        if endpoint, exists := endpoints[r.URL.Path]; exists {
            setCORSHeaders(w, r, endpoint.CORS)
        }
        
        next.ServeHTTP(w, r)
    })
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    endpoint, exists := endpoints[r.URL.Path]
    if !exists {
        http.Error(w, "Endpoint not found", http.StatusNotFound)
        return
    }
    
    // Check if method is allowed
    methodAllowed := false
    for _, method := range endpoint.Methods {
        if r.Method == method {
            methodAllowed = true
            break
        }
    }
    
    if !methodAllowed {
        w.Header().Set("Allow", strings.Join(endpoint.Methods, ", "))
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Simulate API response
    w.Header().Set("Content-Type", "application/json")
    response := map[string]interface{}{
        "message":     fmt.Sprintf("Hello there from %s!", r.URL.Path),
        "method":      r.Method,
        "endpoint":    endpoint.Description,
        "timestamp":   time.Now().Format(time.RFC3339),
        "auth_required": endpoint.Authentication,
    }
    
    json.NewEncoder(w).Encode(response)
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head><title>HTTP OPTIONS Method Demo</title></head>
    <body>
        <h1>HTTP OPTIONS Method Demo</h1>
        <p>Hello there! This demonstrates OPTIONS method and CORS handling.</p>
        
        <h2>Available Endpoints:</h2>
        <ul>
            <li>/api/users - User management (restricted CORS)</li>
            <li>/api/users/profile - Profile management (open CORS)</li>
            <li>/api/public/status - Public status (minimal CORS)</li>
        </ul>
        
        <h2>Test Commands:</h2>
        
        <h3>Basic OPTIONS requests:</h3>
        <pre>
curl -X OPTIONS http://localhost:8080/api/users
curl -X OPTIONS http://localhost:8080/api/users/profile
curl -X OPTIONS http://localhost:8080/api/public/status
        </pre>
        
        <h3>CORS preflight simulation:</h3>
        <pre>
curl -X OPTIONS \
  -H "Origin: https://example.com" \
  -H "Access-Control-Request-Method: POST" \
  -H "Access-Control-Request-Headers: Content-Type,Authorization" \
  http://localhost:8080/api/users
        </pre>
        
        <h3>Debug mode (detailed response):</h3>
        <pre>
curl -X OPTIONS \
  -H "X-Debug: true" \
  -H "Origin: https://app.example.com" \
  http://localhost:8080/api/users/profile
        </pre>
        
        <h3>Cross-origin from browser:</h3>
        <pre>
fetch('http://localhost:8080/api/users', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token'
    },
    body: JSON.stringify({name: 'John'})
});
        </pre>
        
        <h2>OPTIONS Method Benefits:</h2>
        <ul>
            <li>Discover allowed HTTP methods for a resource</li>
            <li>CORS preflight request handling</li>
            <li>API capability discovery</li>
            <li>Security policy advertisement</li>
            <li>Client-server contract verification</li>
        </ul>
        
        <h2>CORS Headers Explained:</h2>
        <ul>
            <li><strong>Access-Control-Allow-Origin:</strong> Allowed origins</li>
            <li><strong>Access-Control-Allow-Methods:</strong> Allowed HTTP methods</li>
            <li><strong>Access-Control-Allow-Headers:</strong> Allowed request headers</li>
            <li><strong>Access-Control-Expose-Headers:</strong> Headers exposed to client</li>
            <li><strong>Access-Control-Allow-Credentials:</strong> Cookie/auth support</li>
            <li><strong>Access-Control-Max-Age:</strong> Preflight cache duration</li>
        </ul>
        
        <script>
        // Demo CORS request
        function testCORS() {
            fetch('/api/users/profile', {
                method: 'GET',
                headers: {
                    'X-Requested-With': 'XMLHttpRequest'
                }
            })
            .then(response => response.json())
            .then(data => console.log('CORS test successful:', data))
            .catch(error => console.error('CORS test failed:', error));
        }
        
        // Add button to test CORS
        document.addEventListener('DOMContentLoaded', function() {
            const button = document.createElement('button');
            button.textContent = 'Test CORS Request';
            button.onclick = testCORS;
            document.body.appendChild(button);
        });
        </script>
    </body>
    </html>`)
}

func main() {
    mux := http.NewServeMux()
    
    // Apply CORS middleware to all routes
    handler := corsMiddleware(mux)
    
    mux.HandleFunc("/", demoPageHandler)
    mux.HandleFunc("/api/users", apiHandler)
    mux.HandleFunc("/api/users/profile", apiHandler)
    mux.HandleFunc("/api/public/status", apiHandler)
    
    server := &http.Server{
        Addr:    ":8080",
        Handler: handler,
    }

    fmt.Println("=== HTTP OPTIONS Method Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• OPTIONS method handling")
    fmt.Println("• CORS preflight requests")
    fmt.Println("• Method discovery")
    fmt.Println("• Origin validation")
    fmt.Println("• Header policy advertisement")
    fmt.Println("• Preflight caching control")

    log.Fatal(server.ListenAndServe())
}
```

The OPTIONS method example demonstrates comprehensive CORS handling,  
preflight request processing, and API capability discovery. The  
implementation shows how to properly handle cross-origin requests while  
maintaining security through configurable CORS policies.

## HTTP request context and cancellation

Request contexts enable cancellation, timeouts, and request-scoped values.  
This example demonstrates context usage for request lifecycle management  
and graceful cancellation handling.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "time"
)

type RequestMetadata struct {
    RequestID string
    UserID    string
    StartTime time.Time
    Timeout   time.Duration
}

type contextKey string

const (
    RequestIDKey contextKey = "requestID"
    UserIDKey    contextKey = "userID"
    MetadataKey  contextKey = "metadata"
)

func requestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        requestID := r.Header.Get("X-Request-ID")
        if requestID == "" {
            requestID = fmt.Sprintf("req_%d", time.Now().UnixNano())
        }
        
        ctx := context.WithValue(r.Context(), RequestIDKey, requestID)
        r = r.WithContext(ctx)
        
        w.Header().Set("X-Request-ID", requestID)
        next.ServeHTTP(w, r)
    })
}

func userMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        userID := r.Header.Get("X-User-ID")
        if userID == "" {
            userID = "anonymous"
        }
        
        ctx := context.WithValue(r.Context(), UserIDKey, userID)
        r = r.WithContext(ctx)
        
        next.ServeHTTP(w, r)
    })
}

func timeoutMiddleware(defaultTimeout time.Duration) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            timeout := defaultTimeout
            
            // Allow custom timeout via header
            if timeoutStr := r.Header.Get("X-Timeout"); timeoutStr != "" {
                if customTimeout, err := time.ParseDuration(timeoutStr); err == nil {
                    timeout = customTimeout
                }
            }
            
            ctx, cancel := context.WithTimeout(r.Context(), timeout)
            defer cancel()
            
            // Store metadata in context
            metadata := RequestMetadata{
                RequestID: ctx.Value(RequestIDKey).(string),
                UserID:    ctx.Value(UserIDKey).(string),
                StartTime: time.Now(),
                Timeout:   timeout,
            }
            
            ctx = context.WithValue(ctx, MetadataKey, metadata)
            r = r.WithContext(ctx)
            
            next.ServeHTTP(w, r)
        })
    }
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    duration := 5 * time.Second
    
    if durationStr := r.URL.Query().Get("duration"); durationStr != "" {
        if customDuration, err := time.ParseDuration(durationStr); err == nil {
            duration = customDuration
        }
    }
    
    // Simulate long-running operation with context cancellation
    select {
    case <-time.After(duration):
        // Operation completed successfully
        metadata := r.Context().Value(MetadataKey).(RequestMetadata)
        elapsed := time.Since(metadata.StartTime)
        
        response := map[string]interface{}{
            "message":     "Hello there! Slow operation completed successfully",
            "request_id":  metadata.RequestID,
            "user_id":     metadata.UserID,
            "duration":    duration.String(),
            "elapsed":     elapsed.String(),
            "timeout":     metadata.Timeout.String(),
            "timestamp":   time.Now().Format(time.RFC3339),
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
        
    case <-r.Context().Done():
        // Request was cancelled or timed out
        err := r.Context().Err()
        metadata := r.Context().Value(MetadataKey).(RequestMetadata)
        elapsed := time.Since(metadata.StartTime)
        
        log.Printf("Request %s cancelled after %v: %v", 
                  metadata.RequestID, elapsed, err)
        
        status := http.StatusRequestTimeout
        message := "Request timed out"
        
        if err == context.Canceled {
            status = http.StatusClientClosedRequest
            message = "Request cancelled by client"
        }
        
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(status)
        
        errorResponse := map[string]interface{}{
            "error":       message,
            "request_id":  metadata.RequestID,
            "elapsed":     elapsed.String(),
            "timeout":     metadata.Timeout.String(),
            "timestamp":   time.Now().Format(time.RFC3339),
        }
        
        json.NewEncoder(w).Encode(errorResponse)
    }
}

func batchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var request struct {
        Operations []struct {
            ID       string `json:"id"`
            Duration string `json:"duration"`
        } `json:"operations"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    metadata := r.Context().Value(MetadataKey).(RequestMetadata)
    
    // Process operations concurrently with context cancellation
    results := make(map[string]interface{})
    resultChan := make(chan map[string]interface{}, len(request.Operations))
    
    for _, op := range request.Operations {
        go func(opID, durationStr string) {
            duration, err := time.ParseDuration(durationStr)
            if err != nil {
                duration = 1 * time.Second
            }
            
            select {
            case <-time.After(duration):
                resultChan <- map[string]interface{}{
                    "id":      opID,
                    "status":  "completed",
                    "duration": duration.String(),
                }
            case <-r.Context().Done():
                resultChan <- map[string]interface{}{
                    "id":     opID,
                    "status": "cancelled",
                    "error":  r.Context().Err().Error(),
                }
            }
        }(op.ID, op.Duration)
    }
    
    // Collect results with timeout
    for i := 0; i < len(request.Operations); i++ {
        select {
        case result := <-resultChan:
            results[result["id"].(string)] = result
        case <-r.Context().Done():
            // Context cancelled, return partial results
            break
        }
    }
    
    elapsed := time.Since(metadata.StartTime)
    
    response := map[string]interface{}{
        "message":     "Batch operation processed",
        "request_id":  metadata.RequestID,
        "elapsed":     elapsed.String(),
        "results":     results,
        "completed":   len(results),
        "total":       len(request.Operations),
        "timestamp":   time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func progressHandler(w http.ResponseWriter, r *http.Request) {
    steps := 10
    if stepsStr := r.URL.Query().Get("steps"); stepsStr != "" {
        if customSteps, err := strconv.Atoi(stepsStr); err == nil && customSteps > 0 {
            steps = customSteps
        }
    }
    
    metadata := r.Context().Value(MetadataKey).(RequestMetadata)
    
    // Server-sent events for progress updates
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    for i := 0; i <= steps; i++ {
        select {
        case <-time.After(500 * time.Millisecond):
            progress := float64(i) / float64(steps) * 100
            elapsed := time.Since(metadata.StartTime)
            
            data := map[string]interface{}{
                "request_id": metadata.RequestID,
                "step":       i,
                "total":      steps,
                "progress":   fmt.Sprintf("%.1f%%", progress),
                "elapsed":    elapsed.String(),
                "timestamp":  time.Now().Format(time.RFC3339),
            }
            
            jsonData, _ := json.Marshal(data)
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            
            if i == steps {
                fmt.Fprintf(w, "data: {\"status\": \"completed\", \"message\": \"Hello there! Progress completed successfully\"}\n\n")
                flusher.Flush()
                return
            }
            
        case <-r.Context().Done():
            // Send cancellation event
            errorData := map[string]interface{}{
                "status":     "cancelled",
                "error":      r.Context().Err().Error(),
                "request_id": metadata.RequestID,
                "timestamp":  time.Now().Format(time.RFC3339),
            }
            
            jsonData, _ := json.Marshal(errorData)
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            return
        }
    }
}

func chainedHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate a chain of operations that depend on context
    ctx := r.Context()
    metadata := ctx.Value(MetadataKey).(RequestMetadata)
    
    operations := []struct {
        name     string
        duration time.Duration
    }{
        {"validate", 200 * time.Millisecond},
        {"authenticate", 300 * time.Millisecond},
        {"authorize", 250 * time.Millisecond},
        {"process", 1 * time.Second},
        {"save", 400 * time.Millisecond},
    }
    
    results := make([]map[string]interface{}, 0, len(operations))
    
    for _, op := range operations {
        start := time.Now()
        
        select {
        case <-time.After(op.duration):
            elapsed := time.Since(start)
            results = append(results, map[string]interface{}{
                "operation": op.name,
                "status":    "completed",
                "duration":  elapsed.String(),
            })
            
        case <-ctx.Done():
            elapsed := time.Since(start)
            results = append(results, map[string]interface{}{
                "operation": op.name,
                "status":    "cancelled",
                "duration":  elapsed.String(),
                "error":     ctx.Err().Error(),
            })
            
            // Return partial results
            response := map[string]interface{}{
                "message":     "Operation chain cancelled",
                "request_id":  metadata.RequestID,
                "operations":  results,
                "cancelled_at": op.name,
                "timestamp":   time.Now().Format(time.RFC3339),
            }
            
            w.Header().Set("Content-Type", "application/json")
            w.WriteHeader(http.StatusRequestTimeout)
            json.NewEncoder(w).Encode(response)
            return
        }
    }
    
    // All operations completed successfully
    response := map[string]interface{}{
        "message":    "Hello there! All operations completed successfully",
        "request_id": metadata.RequestID,
        "operations": results,
        "total_time": time.Since(metadata.StartTime).String(),
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head><title>HTTP Context and Cancellation Demo</title></head>
    <body>
        <h1>HTTP Context and Cancellation Demo</h1>
        <p>Hello there! This demonstrates request context and cancellation.</p>
        
        <h2>Test Endpoints:</h2>
        <ul>
            <li>/slow - Long-running operation with timeout</li>
            <li>/batch - Concurrent operations with cancellation</li>
            <li>/progress - Server-sent events with cancellation</li>
            <li>/chained - Sequential operations with context</li>
        </ul>
        
        <h2>Test Commands:</h2>
        
        <h3>Basic slow request (will timeout in 10s):</h3>
        <pre>
curl "http://localhost:8080/slow?duration=5s"
        </pre>
        
        <h3>Request with custom timeout:</h3>
        <pre>
curl -H "X-Timeout: 2s" "http://localhost:8080/slow?duration=5s"
        </pre>
        
        <h3>Batch operations:</h3>
        <pre>
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "operations": [
      {"id": "op1", "duration": "1s"},
      {"id": "op2", "duration": "2s"},
      {"id": "op3", "duration": "3s"}
    ]
  }' \
  "http://localhost:8080/batch"
        </pre>
        
        <h3>Progress with cancellation:</h3>
        <pre>
curl "http://localhost:8080/progress?steps=20"
# Press Ctrl+C to cancel
        </pre>
        
        <h3>Chained operations:</h3>
        <pre>
curl -H "X-Timeout: 1s" "http://localhost:8080/chained"
        </pre>
        
        <h2>Context Features Demonstrated:</h2>
        <ul>
            <li>Request-scoped values (request ID, user ID)</li>
            <li>Timeout handling and cancellation</li>
            <li>Graceful operation interruption</li>
            <li>Concurrent operation management</li>
            <li>Context propagation through middleware</li>
            <li>Resource cleanup on cancellation</li>
        </ul>
        
        <h2>Headers for Testing:</h2>
        <ul>
            <li><strong>X-Request-ID:</strong> Custom request identifier</li>
            <li><strong>X-User-ID:</strong> User context for operations</li>
            <li><strong>X-Timeout:</strong> Custom timeout duration (e.g., "5s")</li>
        </ul>
    </body>
    </html>`)
}

func main() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/", demoPageHandler)
    mux.HandleFunc("/slow", slowHandler)
    mux.HandleFunc("/batch", batchHandler)
    mux.HandleFunc("/progress", progressHandler)
    mux.HandleFunc("/chained", chainedHandler)
    
    // Apply middleware chain
    handler := requestIDMiddleware(
        userMiddleware(
            timeoutMiddleware(10 * time.Second)(mux)))

    fmt.Println("=== HTTP Context and Cancellation Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Request context propagation")
    fmt.Println("• Timeout handling and cancellation")
    fmt.Println("• Request-scoped values")
    fmt.Println("• Graceful operation interruption")
    fmt.Println("• Concurrent operation management")
    fmt.Println("• Server-sent events with cancellation")

    log.Fatal(http.ListenAndServe(":8080", handler))
}
```

This context and cancellation example demonstrates sophisticated request  
lifecycle management, timeout handling, and graceful cancellation. The  
implementation shows how to use Go's context package to build resilient  
HTTP services that can handle client disconnections and timeouts properly.

## HTTP rate limiting

Rate limiting protects services from abuse and ensures fair resource  
usage. This example demonstrates various rate limiting strategies including  
token bucket, sliding window, and per-user limits.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "sync"
    "time"
)

type RateLimiter interface {
    Allow(key string) bool
    GetInfo(key string) RateLimitInfo
}

type RateLimitInfo struct {
    Allowed    bool
    Limit      int
    Remaining  int
    ResetTime  time.Time
    RetryAfter time.Duration
}

// Token Bucket Rate Limiter
type TokenBucket struct {
    capacity     int
    refillRate   int
    refillPeriod time.Duration
    buckets      map[string]*bucket
    mutex        sync.RWMutex
}

type bucket struct {
    tokens   int
    lastFill time.Time
}

func NewTokenBucket(capacity, refillRate int, refillPeriod time.Duration) *TokenBucket {
    return &TokenBucket{
        capacity:     capacity,
        refillRate:   refillRate,
        refillPeriod: refillPeriod,
        buckets:      make(map[string]*bucket),
    }
}

func (tb *TokenBucket) Allow(key string) bool {
    tb.mutex.Lock()
    defer tb.mutex.Unlock()
    
    b, exists := tb.buckets[key]
    if !exists {
        b = &bucket{
            tokens:   tb.capacity,
            lastFill: time.Now(),
        }
        tb.buckets[key] = b
    }
    
    // Refill tokens based on time elapsed
    now := time.Now()
    elapsed := now.Sub(b.lastFill)
    tokensToAdd := int(elapsed / tb.refillPeriod) * tb.refillRate
    
    if tokensToAdd > 0 {
        b.tokens = min(tb.capacity, b.tokens+tokensToAdd)
        b.lastFill = now
    }
    
    if b.tokens > 0 {
        b.tokens--
        return true
    }
    
    return false
}

func (tb *TokenBucket) GetInfo(key string) RateLimitInfo {
    tb.mutex.RLock()
    defer tb.mutex.RUnlock()
    
    b, exists := tb.buckets[key]
    if !exists {
        return RateLimitInfo{
            Allowed:    true,
            Limit:      tb.capacity,
            Remaining:  tb.capacity,
            ResetTime:  time.Now().Add(tb.refillPeriod),
            RetryAfter: 0,
        }
    }
    
    // Calculate when next token will be available
    timeToNextToken := tb.refillPeriod - time.Since(b.lastFill)%tb.refillPeriod
    
    return RateLimitInfo{
        Allowed:    b.tokens > 0,
        Limit:      tb.capacity,
        Remaining:  b.tokens,
        ResetTime:  time.Now().Add(timeToNextToken),
        RetryAfter: timeToNextToken,
    }
}

// Sliding Window Rate Limiter
type SlidingWindow struct {
    limit      int
    window     time.Duration
    requests   map[string][]time.Time
    mutex      sync.RWMutex
}

func NewSlidingWindow(limit int, window time.Duration) *SlidingWindow {
    return &SlidingWindow{
        limit:    limit,
        window:   window,
        requests: make(map[string][]time.Time),
    }
}

func (sw *SlidingWindow) Allow(key string) bool {
    sw.mutex.Lock()
    defer sw.mutex.Unlock()
    
    now := time.Now()
    cutoff := now.Add(-sw.window)
    
    // Clean old requests
    if requests, exists := sw.requests[key]; exists {
        validRequests := make([]time.Time, 0, len(requests))
        for _, reqTime := range requests {
            if reqTime.After(cutoff) {
                validRequests = append(validRequests, reqTime)
            }
        }
        sw.requests[key] = validRequests
    } else {
        sw.requests[key] = make([]time.Time, 0)
    }
    
    // Check if limit exceeded
    if len(sw.requests[key]) >= sw.limit {
        return false
    }
    
    // Add current request
    sw.requests[key] = append(sw.requests[key], now)
    return true
}

func (sw *SlidingWindow) GetInfo(key string) RateLimitInfo {
    sw.mutex.RLock()
    defer sw.mutex.RUnlock()
    
    now := time.Now()
    cutoff := now.Add(-sw.window)
    
    requests, exists := sw.requests[key]
    if !exists {
        return RateLimitInfo{
            Allowed:    true,
            Limit:      sw.limit,
            Remaining:  sw.limit,
            ResetTime:  now.Add(sw.window),
            RetryAfter: 0,
        }
    }
    
    // Count valid requests
    validCount := 0
    oldestValid := now
    for _, reqTime := range requests {
        if reqTime.After(cutoff) {
            validCount++
            if reqTime.Before(oldestValid) {
                oldestValid = reqTime
            }
        }
    }
    
    remaining := sw.limit - validCount
    retryAfter := time.Duration(0)
    
    if remaining <= 0 {
        retryAfter = oldestValid.Add(sw.window).Sub(now)
    }
    
    return RateLimitInfo{
        Allowed:    remaining > 0,
        Limit:      sw.limit,
        Remaining:  max(0, remaining),
        ResetTime:  oldestValid.Add(sw.window),
        RetryAfter: retryAfter,
    }
}

// Composite Rate Limiter with multiple strategies
type CompositeRateLimiter struct {
    limiters map[string]RateLimiter
}

func NewCompositeRateLimiter() *CompositeRateLimiter {
    return &CompositeRateLimiter{
        limiters: map[string]RateLimiter{
            "global":    NewSlidingWindow(1000, time.Minute),           // 1000 req/min globally
            "per_ip":    NewTokenBucket(100, 10, time.Second),          // 100 tokens, 10/sec refill
            "per_user":  NewSlidingWindow(500, time.Minute),            // 500 req/min per user
            "burst":     NewTokenBucket(20, 1, time.Second),            // 20 burst, 1/sec refill
        },
    }
}

func (crl *CompositeRateLimiter) checkLimits(keys map[string]string) map[string]RateLimitInfo {
    results := make(map[string]RateLimitInfo)
    
    for limiterType, key := range keys {
        if limiter, exists := crl.limiters[limiterType]; exists {
            info := limiter.GetInfo(key)
            results[limiterType] = info
            
            if !info.Allowed {
                return results // Return early if any limit exceeded
            }
        }
    }
    
    // All checks passed, consume tokens
    for limiterType, key := range keys {
        if limiter, exists := crl.limiters[limiterType]; exists {
            limiter.Allow(key)
        }
    }
    
    return results
}

func rateLimitMiddleware(limiter *CompositeRateLimiter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Extract rate limit keys
            clientIP := getClientIP(r)
            userID := r.Header.Get("X-User-ID")
            if userID == "" {
                userID = "anonymous"
            }
            
            keys := map[string]string{
                "global":   "global",
                "per_ip":   clientIP,
                "per_user": userID,
                "burst":    clientIP + ":" + userID,
            }
            
            results := limiter.checkLimits(keys)
            
            // Find the most restrictive limit
            var mostRestrictive RateLimitInfo
            var limitType string
            
            for lType, info := range results {
                if !info.Allowed {
                    mostRestrictive = info
                    limitType = lType
                    break
                }
                
                if limitType == "" || info.Remaining < mostRestrictive.Remaining {
                    mostRestrictive = info
                    limitType = lType
                }
            }
            
            // Set rate limit headers
            w.Header().Set("X-RateLimit-Limit", strconv.Itoa(mostRestrictive.Limit))
            w.Header().Set("X-RateLimit-Remaining", strconv.Itoa(mostRestrictive.Remaining))
            w.Header().Set("X-RateLimit-Reset", strconv.FormatInt(mostRestrictive.ResetTime.Unix(), 10))
            w.Header().Set("X-RateLimit-Type", limitType)
            
            if !mostRestrictive.Allowed {
                w.Header().Set("Retry-After", strconv.FormatInt(int64(mostRestrictive.RetryAfter.Seconds()), 10))
                
                errorResponse := map[string]interface{}{
                    "error":       "Rate limit exceeded",
                    "limit_type":  limitType,
                    "limit":       mostRestrictive.Limit,
                    "reset_time":  mostRestrictive.ResetTime.Format(time.RFC3339),
                    "retry_after": mostRestrictive.RetryAfter.String(),
                    "timestamp":   time.Now().Format(time.RFC3339),
                }
                
                w.Header().Set("Content-Type", "application/json")
                w.WriteHeader(http.StatusTooManyRequests)
                json.NewEncoder(w).Encode(errorResponse)
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

func getClientIP(r *http.Request) string {
    // Check X-Forwarded-For header
    if xff := r.Header.Get("X-Forwarded-For"); xff != "" {
        return xff
    }
    
    // Check X-Real-IP header
    if xri := r.Header.Get("X-Real-IP"); xri != "" {
        return xri
    }
    
    // Fall back to remote address
    return r.RemoteAddr
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message":   "Hello there! Request processed successfully",
        "timestamp": time.Now().Format(time.RFC3339),
        "method":    r.Method,
        "path":      r.URL.Path,
        "client_ip": getClientIP(r),
        "user_id":   r.Header.Get("X-User-ID"),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func statusHandler(limiter *CompositeRateLimiter) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        clientIP := getClientIP(r)
        userID := r.Header.Get("X-User-ID")
        if userID == "" {
            userID = "anonymous"
        }
        
        keys := map[string]string{
            "global":   "global",
            "per_ip":   clientIP,
            "per_user": userID,
            "burst":    clientIP + ":" + userID,
        }
        
        status := make(map[string]RateLimitInfo)
        for limiterType, key := range keys {
            if rateLimiter, exists := limiter.limiters[limiterType]; exists {
                status[limiterType] = rateLimiter.GetInfo(key)
            }
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]interface{}{
            "rate_limits": status,
            "client_ip":   clientIP,
            "user_id":     userID,
            "timestamp":   time.Now().Format(time.RFC3339),
        })
    }
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func main() {
    limiter := NewCompositeRateLimiter()
    
    mux := http.NewServeMux()
    mux.HandleFunc("/api/data", apiHandler)
    mux.HandleFunc("/status", statusHandler(limiter))
    
    // Apply rate limiting middleware
    handler := rateLimitMiddleware(limiter)(mux)
    
    fmt.Println("=== HTTP Rate Limiting Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Rate limits configured:")
    fmt.Println("• Global: 1000 requests/minute")
    fmt.Println("• Per IP: 100 tokens, 10/second refill")
    fmt.Println("• Per User: 500 requests/minute")
    fmt.Println("• Burst: 20 tokens, 1/second refill")
    fmt.Println()
    fmt.Println("Test commands:")
    fmt.Println("  curl http://localhost:8080/api/data")
    fmt.Println("  curl -H 'X-User-ID: user123' http://localhost:8080/api/data")
    fmt.Println("  curl http://localhost:8080/status")
    fmt.Println()
    fmt.Println("Load testing:")
    fmt.Println("  for i in {1..25}; do curl http://localhost:8080/api/data; done")

    log.Fatal(http.ListenAndServe(":8080", handler))
}
```

This comprehensive rate limiting example demonstrates multiple limiting  
strategies, composite rate limiters, and proper HTTP headers for rate  
limit communication. The implementation shows how to build production-ready  
rate limiting that protects services while providing clear feedback to  
clients about their usage limits.

## HTTP request validation

Request validation ensures data integrity and API security. This example  
demonstrates comprehensive input validation, sanitization, and error  
handling for HTTP requests.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "net/mail"
    "regexp"
    "strconv"
    "strings"
    "time"
    "unicode"
)

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
    Code    string `json:"code"`
}

type ValidationResult struct {
    Valid  bool              `json:"valid"`
    Errors []ValidationError `json:"errors,omitempty"`
}

type UserRequest struct {
    Name     string `json:"name"`
    Email    string `json:"email"`
    Age      int    `json:"age"`
    Phone    string `json:"phone"`
    Website  string `json:"website,omitempty"`
    Bio      string `json:"bio,omitempty"`
    Tags     []string `json:"tags,omitempty"`
    Settings map[string]interface{} `json:"settings,omitempty"`
}

type Validator struct {
    emailRegex   *regexp.Regexp
    phoneRegex   *regexp.Regexp
    urlRegex     *regexp.Regexp
    nameRegex    *regexp.Regexp
}

func NewValidator() *Validator {
    return &Validator{
        emailRegex: regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        phoneRegex: regexp.MustCompile(`^\+?[\d\s\-\(\)]{10,15}$`),
        urlRegex:   regexp.MustCompile(`^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(?:/.*)?$`),
        nameRegex:  regexp.MustCompile(`^[a-zA-Z\s\-'\.]{2,50}$`),
    }
}

func (v *Validator) ValidateUser(user UserRequest) ValidationResult {
    var errors []ValidationError
    
    // Validate name
    if nameErr := v.validateName(user.Name); nameErr != nil {
        errors = append(errors, *nameErr)
    }
    
    // Validate email
    if emailErr := v.validateEmail(user.Email); emailErr != nil {
        errors = append(errors, *emailErr)
    }
    
    // Validate age
    if ageErr := v.validateAge(user.Age); ageErr != nil {
        errors = append(errors, *ageErr)
    }
    
    // Validate phone
    if phoneErr := v.validatePhone(user.Phone); phoneErr != nil {
        errors = append(errors, *phoneErr)
    }
    
    // Validate website (optional)
    if user.Website != "" {
        if websiteErr := v.validateWebsite(user.Website); websiteErr != nil {
            errors = append(errors, *websiteErr)
        }
    }
    
    // Validate bio (optional)
    if bioErr := v.validateBio(user.Bio); bioErr != nil {
        errors = append(errors, *bioErr)
    }
    
    // Validate tags
    if tagsErr := v.validateTags(user.Tags); tagsErr != nil {
        errors = append(errors, tagsErr...)
    }
    
    // Validate settings
    if settingsErr := v.validateSettings(user.Settings); settingsErr != nil {
        errors = append(errors, settingsErr...)
    }
    
    return ValidationResult{
        Valid:  len(errors) == 0,
        Errors: errors,
    }
}

func (v *Validator) validateName(name string) *ValidationError {
    name = strings.TrimSpace(name)
    
    if name == "" {
        return &ValidationError{
            Field:   "name",
            Message: "Name is required",
            Code:    "REQUIRED",
        }
    }
    
    if len(name) < 2 {
        return &ValidationError{
            Field:   "name",
            Message: "Name must be at least 2 characters long",
            Code:    "TOO_SHORT",
        }
    }
    
    if len(name) > 50 {
        return &ValidationError{
            Field:   "name",
            Message: "Name must be no more than 50 characters long",
            Code:    "TOO_LONG",
        }
    }
    
    if !v.nameRegex.MatchString(name) {
        return &ValidationError{
            Field:   "name",
            Message: "Name contains invalid characters",
            Code:    "INVALID_FORMAT",
        }
    }
    
    // Check for potentially malicious content
    if containsSuspiciousContent(name) {
        return &ValidationError{
            Field:   "name",
            Message: "Name contains suspicious content",
            Code:    "SUSPICIOUS_CONTENT",
        }
    }
    
    return nil
}

func (v *Validator) validateEmail(email string) *ValidationError {
    email = strings.TrimSpace(strings.ToLower(email))
    
    if email == "" {
        return &ValidationError{
            Field:   "email",
            Message: "Email is required",
            Code:    "REQUIRED",
        }
    }
    
    // Use Go's mail package for initial validation
    if _, err := mail.ParseAddress(email); err != nil {
        return &ValidationError{
            Field:   "email",
            Message: "Invalid email format",
            Code:    "INVALID_FORMAT",
        }
    }
    
    // Additional regex validation
    if !v.emailRegex.MatchString(email) {
        return &ValidationError{
            Field:   "email",
            Message: "Email format is not supported",
            Code:    "UNSUPPORTED_FORMAT",
        }
    }
    
    // Check for disposable email domains (simplified)
    disposableDomains := []string{"tempmail.org", "10minutemail.com", "guerrillamail.com"}
    emailParts := strings.Split(email, "@")
    if len(emailParts) == 2 {
        domain := emailParts[1]
        for _, disposable := range disposableDomains {
            if domain == disposable {
                return &ValidationError{
                    Field:   "email",
                    Message: "Disposable email addresses are not allowed",
                    Code:    "DISPOSABLE_EMAIL",
                }
            }
        }
    }
    
    return nil
}

func (v *Validator) validateAge(age int) *ValidationError {
    if age < 13 {
        return &ValidationError{
            Field:   "age",
            Message: "Age must be at least 13",
            Code:    "TOO_YOUNG",
        }
    }
    
    if age > 120 {
        return &ValidationError{
            Field:   "age",
            Message: "Age must be no more than 120",
            Code:    "TOO_OLD",
        }
    }
    
    return nil
}

func (v *Validator) validatePhone(phone string) *ValidationError {
    phone = strings.TrimSpace(phone)
    
    if phone == "" {
        return &ValidationError{
            Field:   "phone",
            Message: "Phone number is required",
            Code:    "REQUIRED",
        }
    }
    
    // Remove common formatting characters for validation
    cleanPhone := strings.ReplaceAll(phone, " ", "")
    cleanPhone = strings.ReplaceAll(cleanPhone, "-", "")
    cleanPhone = strings.ReplaceAll(cleanPhone, "(", "")
    cleanPhone = strings.ReplaceAll(cleanPhone, ")", "")
    
    if !v.phoneRegex.MatchString(phone) {
        return &ValidationError{
            Field:   "phone",
            Message: "Invalid phone number format",
            Code:    "INVALID_FORMAT",
        }
    }
    
    // Check for obviously fake numbers
    fakePatterns := []string{"1234567890", "0000000000", "1111111111"}
    for _, pattern := range fakePatterns {
        if strings.Contains(cleanPhone, pattern) {
            return &ValidationError{
                Field:   "phone",
                Message: "Phone number appears to be fake",
                Code:    "FAKE_NUMBER",
            }
        }
    }
    
    return nil
}

func (v *Validator) validateWebsite(website string) *ValidationError {
    website = strings.TrimSpace(website)
    
    if len(website) > 255 {
        return &ValidationError{
            Field:   "website",
            Message: "Website URL is too long",
            Code:    "TOO_LONG",
        }
    }
    
    if !v.urlRegex.MatchString(website) {
        return &ValidationError{
            Field:   "website",
            Message: "Invalid website URL format",
            Code:    "INVALID_FORMAT",
        }
    }
    
    // Check for malicious URLs (simplified)
    maliciousDomains := []string{"malware.com", "phishing.net", "suspicious.org"}
    for _, domain := range maliciousDomains {
        if strings.Contains(website, domain) {
            return &ValidationError{
                Field:   "website",
                Message: "Website URL is not allowed",
                Code:    "BLOCKED_DOMAIN",
            }
        }
    }
    
    return nil
}

func (v *Validator) validateBio(bio string) *ValidationError {
    bio = strings.TrimSpace(bio)
    
    if len(bio) > 500 {
        return &ValidationError{
            Field:   "bio",
            Message: "Bio must be no more than 500 characters",
            Code:    "TOO_LONG",
        }
    }
    
    // Check for suspicious content
    if containsSuspiciousContent(bio) {
        return &ValidationError{
            Field:   "bio",
            Message: "Bio contains inappropriate content",
            Code:    "INAPPROPRIATE_CONTENT",
        }
    }
    
    return nil
}

func (v *Validator) validateTags(tags []string) []ValidationError {
    var errors []ValidationError
    
    if len(tags) > 10 {
        errors = append(errors, ValidationError{
            Field:   "tags",
            Message: "Maximum 10 tags allowed",
            Code:    "TOO_MANY",
        })
    }
    
    for i, tag := range tags {
        tag = strings.TrimSpace(tag)
        
        if tag == "" {
            errors = append(errors, ValidationError{
                Field:   fmt.Sprintf("tags[%d]", i),
                Message: "Tag cannot be empty",
                Code:    "EMPTY_TAG",
            })
            continue
        }
        
        if len(tag) > 30 {
            errors = append(errors, ValidationError{
                Field:   fmt.Sprintf("tags[%d]", i),
                Message: "Tag must be no more than 30 characters",
                Code:    "TOO_LONG",
            })
        }
        
        // Check for invalid characters
        for _, char := range tag {
            if !unicode.IsLetter(char) && !unicode.IsNumber(char) && char != '-' && char != '_' {
                errors = append(errors, ValidationError{
                    Field:   fmt.Sprintf("tags[%d]", i),
                    Message: "Tag contains invalid characters",
                    Code:    "INVALID_CHARACTERS",
                })
                break
            }
        }
    }
    
    return errors
}

func (v *Validator) validateSettings(settings map[string]interface{}) []ValidationError {
    var errors []ValidationError
    
    if len(settings) > 20 {
        errors = append(errors, ValidationError{
            Field:   "settings",
            Message: "Maximum 20 settings allowed",
            Code:    "TOO_MANY",
        })
    }
    
    for key, value := range settings {
        if len(key) > 50 {
            errors = append(errors, ValidationError{
                Field:   fmt.Sprintf("settings.%s", key),
                Message: "Setting key is too long",
                Code:    "KEY_TOO_LONG",
            })
        }
        
        // Validate value based on type
        switch v := value.(type) {
        case string:
            if len(v) > 1000 {
                errors = append(errors, ValidationError{
                    Field:   fmt.Sprintf("settings.%s", key),
                    Message: "Setting value is too long",
                    Code:    "VALUE_TOO_LONG",
                })
            }
        case float64:
            if v < -1000000 || v > 1000000 {
                errors = append(errors, ValidationError{
                    Field:   fmt.Sprintf("settings.%s", key),
                    Message: "Setting value is out of range",
                    Code:    "VALUE_OUT_OF_RANGE",
                })
            }
        }
    }
    
    return errors
}

func containsSuspiciousContent(content string) bool {
    // Simple check for suspicious patterns
    suspiciousPatterns := []string{
        "<script", "javascript:", "onclick=", "onerror=",
        "eval(", "alert(", "confirm(", "prompt(",
    }
    
    lowerContent := strings.ToLower(content)
    for _, pattern := range suspiciousPatterns {
        if strings.Contains(lowerContent, pattern) {
            return true
        }
    }
    
    return false
}

func sanitizeString(input string) string {
    // Remove potentially dangerous characters
    input = strings.TrimSpace(input)
    input = strings.ReplaceAll(input, "<", "&lt;")
    input = strings.ReplaceAll(input, ">", "&gt;")
    input = strings.ReplaceAll(input, "\"", "&quot;")
    input = strings.ReplaceAll(input, "'", "&#39;")
    input = strings.ReplaceAll(input, "&", "&amp;")
    
    return input
}

func validateAndCreateUser(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var userReq UserRequest
    if err := json.NewDecoder(r.Body).Decode(&userReq); err != nil {
        http.Error(w, "Invalid JSON format", http.StatusBadRequest)
        return
    }
    
    validator := NewValidator()
    result := validator.ValidateUser(userReq)
    
    if !result.Valid {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusBadRequest)
        
        response := map[string]interface{}{
            "error":      "Validation failed",
            "validation": result,
            "timestamp":  time.Now().Format(time.RFC3339),
        }
        
        json.NewEncoder(w).Encode(response)
        return
    }
    
    // Sanitize input data
    sanitizedUser := UserRequest{
        Name:     sanitizeString(userReq.Name),
        Email:    strings.ToLower(strings.TrimSpace(userReq.Email)),
        Age:      userReq.Age,
        Phone:    strings.TrimSpace(userReq.Phone),
        Website:  strings.TrimSpace(userReq.Website),
        Bio:      sanitizeString(userReq.Bio),
        Tags:     make([]string, len(userReq.Tags)),
        Settings: userReq.Settings,
    }
    
    for i, tag := range userReq.Tags {
        sanitizedUser.Tags[i] = sanitizeString(strings.TrimSpace(tag))
    }
    
    // Simulate user creation
    response := map[string]interface{}{
        "message":   "Hello there! User created successfully",
        "user":      sanitizedUser,
        "user_id":   fmt.Sprintf("user_%d", time.Now().UnixNano()),
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(response)
}

func validateQueryParams(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query()
    var errors []ValidationError
    
    // Validate page parameter
    if pageStr := query.Get("page"); pageStr != "" {
        if page, err := strconv.Atoi(pageStr); err != nil || page < 1 || page > 1000 {
            errors = append(errors, ValidationError{
                Field:   "page",
                Message: "Page must be a number between 1 and 1000",
                Code:    "INVALID_PAGE",
            })
        }
    }
    
    // Validate limit parameter
    if limitStr := query.Get("limit"); limitStr != "" {
        if limit, err := strconv.Atoi(limitStr); err != nil || limit < 1 || limit > 100 {
            errors = append(errors, ValidationError{
                Field:   "limit",
                Message: "Limit must be a number between 1 and 100",
                Code:    "INVALID_LIMIT",
            })
        }
    }
    
    // Validate sort parameter
    if sort := query.Get("sort"); sort != "" {
        allowedSorts := []string{"name", "email", "age", "created_at"}
        valid := false
        for _, allowed := range allowedSorts {
            if sort == allowed || sort == "-"+allowed {
                valid = true
                break
            }
        }
        if !valid {
            errors = append(errors, ValidationError{
                Field:   "sort",
                Message: "Invalid sort field",
                Code:    "INVALID_SORT",
            })
        }
    }
    
    // Validate search parameter
    if search := query.Get("search"); search != "" {
        if len(search) < 2 {
            errors = append(errors, ValidationError{
                Field:   "search",
                Message: "Search query must be at least 2 characters",
                Code:    "SEARCH_TOO_SHORT",
            })
        }
        if containsSuspiciousContent(search) {
            errors = append(errors, ValidationError{
                Field:   "search",
                Message: "Search query contains invalid content",
                Code:    "INVALID_SEARCH",
            })
        }
    }
    
    if len(errors) > 0 {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusBadRequest)
        
        response := map[string]interface{}{
            "error":      "Query parameter validation failed",
            "validation": ValidationResult{Valid: false, Errors: errors},
            "timestamp":  time.Now().Format(time.RFC3339),
        }
        
        json.NewEncoder(w).Encode(response)
        return
    }
    
    response := map[string]interface{}{
        "message":    "Hello there! Query parameters are valid",
        "parameters": query,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head><title>HTTP Request Validation Demo</title></head>
    <body>
        <h1>HTTP Request Validation Demo</h1>
        <p>Hello there! This demonstrates comprehensive request validation.</p>
        
        <h2>Test Endpoints:</h2>
        <ul>
            <li>POST /users - Create user with validation</li>
            <li>GET /search - Query parameter validation</li>
        </ul>
        
        <h2>Example Requests:</h2>
        
        <h3>Valid User Creation:</h3>
        <pre>
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "age": 25,
    "phone": "+1-555-123-4567",
    "website": "https://johndoe.com",
    "bio": "Software developer",
    "tags": ["developer", "go", "web"],
    "settings": {"theme": "dark", "notifications": true}
  }'
        </pre>
        
        <h3>Invalid User (multiple validation errors):</h3>
        <pre>
curl -X POST http://localhost:8080/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "J",
    "email": "invalid-email",
    "age": 5,
    "phone": "1234",
    "website": "not-a-url",
    "bio": "' + strings.Repeat("a", 600) + '",
    "tags": ["' + strings.Repeat("b", 50) + '"]
  }'
        </pre>
        
        <h3>Query Parameter Validation:</h3>
        <pre>
curl "http://localhost:8080/search?page=1&limit=10&sort=name&search=john"
curl "http://localhost:8080/search?page=0&limit=200&sort=invalid&search=x"
        </pre>
        
        <h2>Validation Features:</h2>
        <ul>
            <li>Required field validation</li>
            <li>Length and range constraints</li>
            <li>Format validation (email, phone, URL)</li>
            <li>Content sanitization</li>
            <li>Suspicious content detection</li>
            <li>Custom validation rules</li>
            <li>Detailed error messages</li>
            <li>Query parameter validation</li>
        </ul>
        
        <h2>Security Measures:</h2>
        <ul>
            <li>XSS prevention through sanitization</li>
            <li>SQL injection prevention</li>
            <li>Input length limits</li>
            <li>Disposable email detection</li>
            <li>Malicious URL blocking</li>
            <li>Suspicious pattern detection</li>
        </ul>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", demoPageHandler)
    http.HandleFunc("/users", validateAndCreateUser)
    http.HandleFunc("/search", validateQueryParams)

    fmt.Println("=== HTTP Request Validation Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Comprehensive input validation")
    fmt.Println("• Content sanitization and security")
    fmt.Println("• Custom validation rules")
    fmt.Println("• Detailed error responses")
    fmt.Println("• Query parameter validation")
    fmt.Println("• Security threat detection")

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This comprehensive validation example demonstrates input validation,  
sanitization, security threat detection, and detailed error reporting.  
The implementation shows how to build secure HTTP APIs that properly  
validate and sanitize user input while providing clear feedback about  
validation failures.

## HTTPS and TLS configuration

HTTPS provides encrypted communication and is essential for production  
web applications. This example demonstrates TLS configuration, certificate  
management, and secure server setup.

```go
package main

import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/tls"
    "crypto/x509"
    "crypto/x509/pkix"
    "encoding/pem"
    "fmt"
    "log"
    "math/big"
    "net/http"
    "os"
    "time"
)

func generateSelfSignedCert() error {
    // Generate private key
    privateKey, err := rsa.GenerateKey(rand.Reader, 2048)
    if err != nil {
        return fmt.Errorf("failed to generate private key: %v", err)
    }

    // Create certificate template
    template := x509.Certificate{
        SerialNumber: big.NewInt(1),
        Subject: pkix.Name{
            Organization:  []string{"Demo Company"},
            Country:       []string{"US"},
            Province:      []string{""},
            Locality:      []string{"San Francisco"},
            StreetAddress: []string{""},
            PostalCode:    []string{""},
            CommonName:    "localhost",
        },
        NotBefore:             time.Now(),
        NotAfter:              time.Now().Add(365 * 24 * time.Hour),
        KeyUsage:              x509.KeyUsageKeyEncipherment | x509.KeyUsageDigitalSignature,
        ExtKeyUsage:           []x509.ExtKeyUsage{x509.ExtKeyUsageServerAuth},
        IPAddresses:           []net.IP{net.IPv4(127, 0, 0, 1), net.IPv6loopback},
        DNSNames:              []string{"localhost"},
        BasicConstraintsValid: true,
    }

    // Create certificate
    certDER, err := x509.CreateCertificate(rand.Reader, &template, &template, 
                                          &privateKey.PublicKey, privateKey)
    if err != nil {
        return fmt.Errorf("failed to create certificate: %v", err)
    }

    // Save certificate
    certOut, err := os.Create("server.crt")
    if err != nil {
        return fmt.Errorf("failed to open cert.pem for writing: %v", err)
    }
    defer certOut.Close()

    if err := pem.Encode(certOut, &pem.Block{Type: "CERTIFICATE", Bytes: certDER}); err != nil {
        return fmt.Errorf("failed to write certificate: %v", err)
    }

    // Save private key
    keyOut, err := os.Create("server.key")
    if err != nil {
        return fmt.Errorf("failed to open key.pem for writing: %v", err)
    }
    defer keyOut.Close()

    privateKeyDER, err := x509.MarshalPKCS8PrivateKey(privateKey)
    if err != nil {
        return fmt.Errorf("failed to marshal private key: %v", err)
    }

    if err := pem.Encode(keyOut, &pem.Block{Type: "PRIVATE KEY", Bytes: privateKeyDER}); err != nil {
        return fmt.Errorf("failed to write private key: %v", err)
    }

    fmt.Println("Generated self-signed certificate and private key")
    return nil
}

func createTLSConfig() *tls.Config {
    return &tls.Config{
        MinVersion: tls.VersionTLS12,
        MaxVersion: tls.VersionTLS13,
        CipherSuites: []uint16{
            tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
            tls.TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256,
            tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
        },
        CurvePreferences: []tls.CurveID{
            tls.X25519,
            tls.CurveP256,
            tls.CurveP384,
        },
        PreferServerCipherSuites: true,
        InsecureSkipVerify:       false,
        NextProtos:               []string{"h2", "http/1.1"},
    }
}

func httpsHandler(w http.ResponseWriter, r *http.Request) {
    // Security headers
    w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
    w.Header().Set("X-Content-Type-Options", "nosniff")
    w.Header().Set("X-Frame-Options", "DENY")
    w.Header().Set("X-XSS-Protection", "1; mode=block")
    w.Header().Set("Content-Security-Policy", "default-src 'self'")

    w.Header().Set("Content-Type", "text/html")
    fmt.Fprintf(w, `
    <!DOCTYPE html>
    <html>
    <head><title>HTTPS Demo</title></head>
    <body>
        <h1>Hello there!</h1>
        <p>This is a secure HTTPS connection.</p>
        
        <h2>Connection Information:</h2>
        <ul>
            <li><strong>Protocol:</strong> %s</li>
            <li><strong>TLS Version:</strong> %s</li>
            <li><strong>Cipher Suite:</strong> %s</li>
            <li><strong>Server Name:</strong> %s</li>
            <li><strong>Certificates:</strong> %d</li>
        </ul>
        
        <h2>Security Headers:</h2>
        <ul>
            <li>Strict-Transport-Security</li>
            <li>X-Content-Type-Options</li>
            <li>X-Frame-Options</li>
            <li>X-XSS-Protection</li>
            <li>Content-Security-Policy</li>
        </ul>
        
        <h2>Client Certificate Information:</h2>
        <p>Client certificate verification: %s</p>
        
        <script>
        // Display additional security information
        document.addEventListener('DOMContentLoaded', function() {
            const isSecure = location.protocol === 'https:';
            const securityInfo = document.createElement('p');
            securityInfo.innerHTML = '<strong>Secure Connection:</strong> ' + 
                                   (isSecure ? 'Yes ✓' : 'No ✗');
            securityInfo.style.color = isSecure ? 'green' : 'red';
            document.body.appendChild(securityInfo);
        });
        </script>
    </body>
    </html>`, 
    r.Proto,
    getTLSVersionString(r.TLS),
    getCipherSuiteString(r.TLS),
    getServerName(r.TLS),
    getCertCount(r.TLS),
    getClientCertStatus(r.TLS))
}

func getTLSVersionString(tls *tls.ConnectionState) string {
    if tls == nil {
        return "No TLS"
    }
    
    switch tls.Version {
    case tls.VersionTLS10:
        return "TLS 1.0"
    case tls.VersionTLS11:
        return "TLS 1.1"
    case tls.VersionTLS12:
        return "TLS 1.2"
    case tls.VersionTLS13:
        return "TLS 1.3"
    default:
        return fmt.Sprintf("Unknown (%d)", tls.Version)
    }
}

func getCipherSuiteString(tls *tls.ConnectionState) string {
    if tls == nil {
        return "No TLS"
    }
    
    suites := map[uint16]string{
        tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384:        "ECDHE-RSA-AES256-GCM-SHA384",
        tls.TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256:  "ECDHE-RSA-CHACHA20-POLY1305",
        tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256:        "ECDHE-RSA-AES128-GCM-SHA256",
        tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384:      "ECDHE-ECDSA-AES256-GCM-SHA384",
        tls.TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256: "ECDHE-ECDSA-CHACHA20-POLY1305",
        tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256:      "ECDHE-ECDSA-AES128-GCM-SHA256",
    }
    
    if name, exists := suites[tls.CipherSuite]; exists {
        return name
    }
    
    return fmt.Sprintf("Unknown (%d)", tls.CipherSuite)
}

func getServerName(tls *tls.ConnectionState) string {
    if tls == nil {
        return "No TLS"
    }
    if tls.ServerName == "" {
        return "Not specified"
    }
    return tls.ServerName
}

func getCertCount(tls *tls.ConnectionState) int {
    if tls == nil {
        return 0
    }
    return len(tls.PeerCertificates)
}

func getClientCertStatus(tls *tls.ConnectionState) string {
    if tls == nil {
        return "No TLS"
    }
    
    if len(tls.PeerCertificates) > 0 {
        return "Client certificate provided"
    }
    
    return "No client certificate"
}

func redirectToHTTPS(w http.ResponseWriter, r *http.Request) {
    httpsURL := "https://" + r.Host + r.RequestURI
    http.Redirect(w, r, httpsURL, http.StatusMovedPermanently)
}

func securityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Security headers
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains; preload")
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("X-Frame-Options", "DENY")
        w.Header().Set("X-XSS-Protection", "1; mode=block")
        w.Header().Set("Referrer-Policy", "strict-origin-when-cross-origin")
        w.Header().Set("Content-Security-Policy", "default-src 'self'; script-src 'self' 'unsafe-inline'")
        
        next.ServeHTTP(w, r)
    })
}

func tlsInfoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    if r.TLS == nil {
        http.Error(w, "Not a TLS connection", http.StatusBadRequest)
        return
    }
    
    info := map[string]interface{}{
        "tls_version":           getTLSVersionString(r.TLS),
        "cipher_suite":          getCipherSuiteString(r.TLS),
        "server_name":           r.TLS.ServerName,
        "handshake_complete":    r.TLS.HandshakeComplete,
        "did_resume":           r.TLS.DidResume,
        "negotiated_protocol":   r.TLS.NegotiatedProtocol,
        "peer_certificates":     len(r.TLS.PeerCertificates),
        "verified_chains":       len(r.TLS.VerifiedChains),
        "signature_scheme":      r.TLS.SignatureScheme,
        "supported_protos":      r.TLS.NegotiatedProtocolIsMutual,
        "timestamp":             time.Now().Format(time.RFC3339),
    }
    
    // Add certificate information if available
    if len(r.TLS.PeerCertificates) > 0 {
        cert := r.TLS.PeerCertificates[0]
        info["client_cert"] = map[string]interface{}{
            "subject":     cert.Subject.String(),
            "issuer":      cert.Issuer.String(),
            "not_before":  cert.NotBefore.Format(time.RFC3339),
            "not_after":   cert.NotAfter.Format(time.RFC3339),
            "dns_names":   cert.DNSNames,
            "ip_addresses": cert.IPAddresses,
        }
    }
    
    json.NewEncoder(w).Encode(map[string]interface{}{
        "message": "Hello there! TLS connection information",
        "tls":     info,
    })
}

func certificateInfoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    // Load server certificate for information
    cert, err := tls.LoadX509KeyPair("server.crt", "server.key")
    if err != nil {
        http.Error(w, "Failed to load certificate", http.StatusInternalServerError)
        return
    }
    
    // Parse certificate
    x509Cert, err := x509.ParseCertificate(cert.Certificate[0])
    if err != nil {
        http.Error(w, "Failed to parse certificate", http.StatusInternalServerError)
        return
    }
    
    certInfo := map[string]interface{}{
        "subject":          x509Cert.Subject.String(),
        "issuer":           x509Cert.Issuer.String(),
        "serial_number":    x509Cert.SerialNumber.String(),
        "not_before":       x509Cert.NotBefore.Format(time.RFC3339),
        "not_after":        x509Cert.NotAfter.Format(time.RFC3339),
        "dns_names":        x509Cert.DNSNames,
        "ip_addresses":     x509Cert.IPAddresses,
        "signature_algorithm": x509Cert.SignatureAlgorithm.String(),
        "public_key_algorithm": x509Cert.PublicKeyAlgorithm.String(),
        "key_usage":        getKeyUsageStrings(x509Cert.KeyUsage),
        "ext_key_usage":    getExtKeyUsageStrings(x509Cert.ExtKeyUsage),
        "is_ca":           x509Cert.IsCA,
        "version":         x509Cert.Version,
    }
    
    response := map[string]interface{}{
        "message":     "Server certificate information",
        "certificate": certInfo,
        "timestamp":   time.Now().Format(time.RFC3339),
    }
    
    json.NewEncoder(w).Encode(response)
}

func getKeyUsageStrings(usage x509.KeyUsage) []string {
    var usages []string
    
    if usage&x509.KeyUsageDigitalSignature != 0 {
        usages = append(usages, "Digital Signature")
    }
    if usage&x509.KeyUsageContentCommitment != 0 {
        usages = append(usages, "Content Commitment")
    }
    if usage&x509.KeyUsageKeyEncipherment != 0 {
        usages = append(usages, "Key Encipherment")
    }
    if usage&x509.KeyUsageDataEncipherment != 0 {
        usages = append(usages, "Data Encipherment")
    }
    if usage&x509.KeyUsageKeyAgreement != 0 {
        usages = append(usages, "Key Agreement")
    }
    if usage&x509.KeyUsageCertSign != 0 {
        usages = append(usages, "Certificate Sign")
    }
    if usage&x509.KeyUsageCRLSign != 0 {
        usages = append(usages, "CRL Sign")
    }
    if usage&x509.KeyUsageEncipherOnly != 0 {
        usages = append(usages, "Encipher Only")
    }
    if usage&x509.KeyUsageDecipherOnly != 0 {
        usages = append(usages, "Decipher Only")
    }
    
    return usages
}

func getExtKeyUsageStrings(usage []x509.ExtKeyUsage) []string {
    var usages []string
    
    for _, u := range usage {
        switch u {
        case x509.ExtKeyUsageServerAuth:
            usages = append(usages, "Server Authentication")
        case x509.ExtKeyUsageClientAuth:
            usages = append(usages, "Client Authentication")
        case x509.ExtKeyUsageCodeSigning:
            usages = append(usages, "Code Signing")
        case x509.ExtKeyUsageEmailProtection:
            usages = append(usages, "Email Protection")
        case x509.ExtKeyUsageTimeStamping:
            usages = append(usages, "Time Stamping")
        case x509.ExtKeyUsageOCSPSigning:
            usages = append(usages, "OCSP Signing")
        }
    }
    
    return usages
}

func main() {
    // Generate self-signed certificate if it doesn't exist
    if _, err := os.Stat("server.crt"); os.IsNotExist(err) {
        if err := generateSelfSignedCert(); err != nil {
            log.Fatal("Failed to generate certificate:", err)
        }
    }
    
    mux := http.NewServeMux()
    mux.HandleFunc("/", httpsHandler)
    mux.HandleFunc("/tls-info", tlsInfoHandler)
    mux.HandleFunc("/cert-info", certificateInfoHandler)
    
    // Apply security headers middleware
    secureHandler := securityHeadersMiddleware(mux)
    
    // HTTPS server with custom TLS configuration
    httpsServer := &http.Server{
        Addr:      ":8443",
        Handler:   secureHandler,
        TLSConfig: createTLSConfig(),
        
        // Security timeouts
        ReadTimeout:       15 * time.Second,
        WriteTimeout:      15 * time.Second,
        IdleTimeout:       60 * time.Second,
        ReadHeaderTimeout: 5 * time.Second,
    }
    
    // HTTP server for redirects
    httpServer := &http.Server{
        Addr:    ":8080",
        Handler: http.HandlerFunc(redirectToHTTPS),
    }
    
    // Start HTTP redirect server
    go func() {
        fmt.Println("HTTP redirect server starting on :8080")
        if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Printf("HTTP server error: %v", err)
        }
    }()
    
    fmt.Println("=== HTTPS and TLS Configuration Demo ===")
    fmt.Println("HTTPS server starting on :8443")
    fmt.Println("HTTP redirect server starting on :8080")
    fmt.Println()
    fmt.Println("Test URLs:")
    fmt.Println("  https://localhost:8443/")
    fmt.Println("  https://localhost:8443/tls-info")
    fmt.Println("  https://localhost:8443/cert-info")
    fmt.Println("  http://localhost:8080/ (redirects to HTTPS)")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Self-signed certificate generation")
    fmt.Println("• Custom TLS configuration")
    fmt.Println("• Security headers")
    fmt.Println("• HTTP to HTTPS redirects")
    fmt.Println("• TLS connection information")
    fmt.Println("• Certificate details")
    fmt.Println()
    fmt.Println("Note: Accept the self-signed certificate warning in your browser")
    
    log.Fatal(httpsServer.ListenAndServeTLS("server.crt", "server.key"))
}
```

This HTTPS example demonstrates comprehensive TLS configuration, certificate  
generation, security headers, and connection information extraction. The  
implementation shows how to set up secure HTTPS servers with proper TLS  
settings and security best practices for production environments.

## HTTP content compression

Content compression reduces bandwidth usage and improves performance.  
This example demonstrates gzip compression, content negotiation, and  
conditional compression based on content type and size.

```go
package main

import (
    "compress/gzip"
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type CompressResponseWriter struct {
    http.ResponseWriter
    Writer io.Writer
}

func (crw *CompressResponseWriter) Write(data []byte) (int, error) {
    return crw.Writer.Write(data)
}

func shouldCompress(contentType string, size int64) bool {
    // Don't compress small responses
    if size > 0 && size < 1024 {
        return false
    }
    
    // Compress text-based content types
    compressibleTypes := []string{
        "text/",
        "application/json",
        "application/javascript",
        "application/xml",
        "application/xml+rss",
        "application/atom+xml",
        "image/svg+xml",
    }
    
    for _, ctype := range compressibleTypes {
        if strings.HasPrefix(contentType, ctype) {
            return true
        }
    }
    
    return false
}

func acceptsGzip(r *http.Request) bool {
    acceptEncoding := r.Header.Get("Accept-Encoding")
    return strings.Contains(acceptEncoding, "gzip")
}

func compressionMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !acceptsGzip(r) {
            next.ServeHTTP(w, r)
            return
        }
        
        // Wrap response writer to capture headers and content
        recorder := &ResponseRecorder{
            ResponseWriter: w,
            statusCode:     200,
            header:         make(http.Header),
        }
        
        next.ServeHTTP(recorder, r)
        
        // Check if we should compress
        contentType := recorder.header.Get("Content-Type")
        contentLength := recorder.header.Get("Content-Length")
        
        var size int64
        if contentLength != "" {
            size, _ = strconv.ParseInt(contentLength, 10, 64)
        } else {
            size = int64(len(recorder.body))
        }
        
        if shouldCompress(contentType, size) {
            // Set compression headers
            w.Header().Set("Content-Encoding", "gzip")
            w.Header().Set("Vary", "Accept-Encoding")
            
            // Copy other headers
            for key, values := range recorder.header {
                if key != "Content-Length" { // Will be set by gzip writer
                    for _, value := range values {
                        w.Header().Add(key, value)
                    }
                }
            }
            
            w.WriteHeader(recorder.statusCode)
            
            // Compress and write response
            gzipWriter := gzip.NewWriter(w)
            defer gzipWriter.Close()
            
            gzipWriter.Write(recorder.body)
        } else {
            // Don't compress, write as-is
            for key, values := range recorder.header {
                for _, value := range values {
                    w.Header().Add(key, value)
                }
            }
            
            w.WriteHeader(recorder.statusCode)
            w.Write(recorder.body)
        }
    })
}

type ResponseRecorder struct {
    http.ResponseWriter
    statusCode int
    header     http.Header
    body       []byte
}

func (rr *ResponseRecorder) WriteHeader(statusCode int) {
    rr.statusCode = statusCode
}

func (rr *ResponseRecorder) Write(data []byte) (int, error) {
    rr.body = append(rr.body, data...)
    return len(data), nil
}

func (rr *ResponseRecorder) Header() http.Header {
    return rr.header
}

func largeTextHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/plain")
    
    // Generate large text content
    content := "Hello there! This is a large text response.\n"
    repeatedContent := strings.Repeat(content, 1000) // ~43KB
    
    fmt.Fprint(w, repeatedContent)
}

func jsonDataHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    // Generate JSON data
    data := map[string]interface{}{
        "message": "Hello there! This is JSON data that can be compressed.",
        "timestamp": time.Now().Format(time.RFC3339),
        "data": make([]map[string]interface{}, 100),
    }
    
    // Add some bulk to the JSON
    for i := 0; i < 100; i++ {
        data["data"].([]map[string]interface{})[i] = map[string]interface{}{
            "id":          i,
            "name":        fmt.Sprintf("Item %d", i),
            "description": fmt.Sprintf("This is a description for item number %d. It contains some repeated text to make compression more effective.", i),
            "timestamp":   time.Now().Add(time.Duration(i) * time.Minute).Format(time.RFC3339),
            "metadata": map[string]string{
                "category": "example",
                "type":     "demo",
                "source":   "compression-test",
            },
        }
    }
    
    json.NewEncoder(w).Encode(data)
}

func htmlPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    
    html := `<!DOCTYPE html>
<html>
<head>
    <title>Compression Demo</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; line-height: 1.6; }
        .container { max-width: 800px; margin: 0 auto; }
        .demo-section { background: #f5f5f5; padding: 20px; margin: 20px 0; border-radius: 5px; }
        pre { background: #333; color: #fff; padding: 15px; border-radius: 3px; overflow-x: auto; }
        .compression-info { background: #e8f4fd; padding: 15px; border-left: 4px solid #2196F3; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello there! HTTP Compression Demo</h1>
        
        <div class="compression-info">
            <h3>Compression Information</h3>
            <p>This page demonstrates HTTP compression using gzip encoding.</p>
            <p><strong>Content-Encoding:</strong> <span id="encoding">Not available</span></p>
            <p><strong>Original Size:</strong> <span id="original-size">Calculating...</span></p>
            <p><strong>Compressed Size:</strong> <span id="compressed-size">Calculating...</span></p>
            <p><strong>Compression Ratio:</strong> <span id="compression-ratio">Calculating...</span></p>
        </div>
        
        <div class="demo-section">
            <h2>Test Endpoints</h2>
            <ul>
                <li><a href="/large-text">Large Text Content (~43KB)</a></li>
                <li><a href="/json-data">JSON Data (~15KB)</a></li>
                <li><a href="/compression-stats">Compression Statistics</a></li>
            </ul>
        </div>
        
        <div class="demo-section">
            <h2>Compression Benefits</h2>
            <ul>
                <li>Reduced bandwidth usage</li>
                <li>Faster page load times</li>
                <li>Lower hosting costs</li>
                <li>Better user experience</li>
                <li>Improved SEO rankings</li>
            </ul>
        </div>
        
        <div class="demo-section">
            <h2>Test Commands</h2>
            <pre>
# Test with compression
curl -H "Accept-Encoding: gzip" -v http://localhost:8080/large-text | gunzip

# Test without compression
curl -v http://localhost:8080/large-text

# Compare sizes
curl -H "Accept-Encoding: gzip" -s http://localhost:8080/json-data | wc -c
curl -s http://localhost:8080/json-data | wc -c
            </pre>
        </div>
        
        <!-- Add some repeated content to make compression more effective -->
        ` + strings.Repeat(`
        <div class="demo-section">
            <p>This is repeated content to demonstrate compression effectiveness. `, 50) +
            strings.Repeat(`Compression works best with repetitive text content. `, 100) +
            strings.Repeat(`</p></div>`, 50) + `
    </div>
    
    <script>
        // Display compression information
        document.addEventListener('DOMContentLoaded', function() {
            // Check response headers for compression info
            fetch(window.location.href)
                .then(response => {
                    const encoding = response.headers.get('content-encoding');
                    document.getElementById('encoding').textContent = encoding || 'None';
                    
                    // Estimate sizes (simplified)
                    const bodySize = document.documentElement.outerHTML.length;
                    document.getElementById('original-size').textContent = bodySize + ' bytes';
                    
                    if (encoding === 'gzip') {
                        const compressedSize = Math.round(bodySize * 0.3); // Estimate
                        document.getElementById('compressed-size').textContent = compressedSize + ' bytes (estimated)';
                        const ratio = Math.round((1 - compressedSize/bodySize) * 100);
                        document.getElementById('compression-ratio').textContent = ratio + '% size reduction';
                    } else {
                        document.getElementById('compressed-size').textContent = 'Not compressed';
                        document.getElementById('compression-ratio').textContent = 'No compression';
                    }
                })
                .catch(error => {
                    console.error('Error checking compression:', error);
                });
        });
    </script>
</body>
</html>`
    
    fmt.Fprint(w, html)
}

func compressionStatsHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    
    acceptsCompression := acceptsGzip(r)
    
    stats := map[string]interface{}{
        "message": "Hello there! Compression statistics",
        "client_accepts_gzip": acceptsCompression,
        "accept_encoding": r.Header.Get("Accept-Encoding"),
        "user_agent": r.Header.Get("User-Agent"),
        "compression_types": []string{
            "text/html", "text/plain", "text/css", "text/javascript",
            "application/json", "application/javascript", "application/xml",
            "image/svg+xml",
        },
        "min_size_for_compression": 1024,
        "timestamp": time.Now().Format(time.RFC3339),
        "server_info": map[string]interface{}{
            "compression_enabled": true,
            "compression_algorithm": "gzip",
            "compression_level": "default",
        },
    }
    
    json.NewEncoder(w).Encode(stats)
}

func imageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "image/png")
    
    // Simulate a small PNG image (won't be compressed)
    pngData := []byte{
        0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A, // PNG signature
        0x00, 0x00, 0x00, 0x0D, 0x49, 0x48, 0x44, 0x52, // IHDR chunk
        0x00, 0x00, 0x00, 0x01, 0x00, 0x00, 0x00, 0x01, // 1x1 image
        0x08, 0x02, 0x00, 0x00, 0x00, 0x90, 0x77, 0x53,
        0xDE, 0x00, 0x00, 0x00, 0x0C, 0x49, 0x44, 0x41, 0x54, // IDAT chunk
        0x08, 0x57, 0x63, 0xF8, 0x00, 0x00, 0x00, 0x01,
        0x00, 0x01, 0x5C, 0xDD, 0x39, 0x82,
        0x00, 0x00, 0x00, 0x00, 0x49, 0x45, 0x4E, 0x44, // IEND chunk
        0xAE, 0x42, 0x60, 0x82,
    }
    
    w.Write(pngData)
}

func compressionTestHandler(w http.ResponseWriter, r *http.Request) {
    sizeParam := r.URL.Query().Get("size")
    size := 10000 // Default 10KB
    
    if sizeParam != "" {
        if parsedSize, err := strconv.Atoi(sizeParam); err == nil && parsedSize > 0 {
            size = parsedSize
        }
    }
    
    w.Header().Set("Content-Type", "text/plain")
    
    // Generate content of specified size
    line := "This is a test line for compression demonstration. "
    lineSize := len(line)
    lines := size / lineSize
    
    content := fmt.Sprintf("Compression test content (%d bytes)\n", size)
    content += strings.Repeat(line, lines)
    
    // Add random content to see compression effectiveness
    if remaining := size - len(content); remaining > 0 {
        content += strings.Repeat("X", remaining)
    }
    
    fmt.Fprint(w, content)
}

func main() {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/", htmlPageHandler)
    mux.HandleFunc("/large-text", largeTextHandler)
    mux.HandleFunc("/json-data", jsonDataHandler)
    mux.HandleFunc("/compression-stats", compressionStatsHandler)
    mux.HandleFunc("/image.png", imageHandler)
    mux.HandleFunc("/test", compressionTestHandler)
    
    // Apply compression middleware
    handler := compressionMiddleware(mux)
    
    server := &http.Server{
        Addr:    ":8080",
        Handler: handler,
    }
    
    fmt.Println("=== HTTP Content Compression Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Test URLs:")
    fmt.Println("  http://localhost:8080/ - HTML page with compression")
    fmt.Println("  http://localhost:8080/large-text - Large text content")
    fmt.Println("  http://localhost:8080/json-data - JSON data")
    fmt.Println("  http://localhost:8080/test?size=50000 - Custom size test")
    fmt.Println()
    fmt.Println("Test compression:")
    fmt.Println("  curl -H 'Accept-Encoding: gzip' -v http://localhost:8080/large-text")
    fmt.Println("  curl -v http://localhost:8080/large-text")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Gzip compression middleware")
    fmt.Println("• Content-type based compression")
    fmt.Println("• Size-based compression decisions")
    fmt.Println("• Accept-Encoding negotiation")
    fmt.Println("• Compression statistics")

    log.Fatal(server.ListenAndServe())
}
```

This compression example demonstrates gzip compression middleware, content  
negotiation, and conditional compression strategies. The implementation  
shows how to efficiently compress HTTP responses while avoiding compression  
of unsuitable content types and small responses that wouldn't benefit.

## HTTP request logging and monitoring

Request logging and monitoring are essential for debugging, analytics, and  
performance tracking. This example demonstrates comprehensive logging,  
metrics collection, and request monitoring.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
    "sync"
    "time"
)

type RequestLog struct {
    Timestamp    time.Time `json:"timestamp"`
    Method       string    `json:"method"`
    URL          string    `json:"url"`
    RemoteAddr   string    `json:"remote_addr"`
    UserAgent    string    `json:"user_agent"`
    StatusCode   int       `json:"status_code"`
    ContentLength int64    `json:"content_length"`
    Duration     time.Duration `json:"duration"`
    RequestID    string    `json:"request_id"`
    Error        string    `json:"error,omitempty"`
}

type Metrics struct {
    TotalRequests   int64            `json:"total_requests"`
    RequestsByMethod map[string]int64 `json:"requests_by_method"`
    RequestsByStatus map[int]int64    `json:"requests_by_status"`
    AverageLatency  time.Duration    `json:"average_latency"`
    TotalLatency    time.Duration    `json:"total_latency"`
    ErrorRate       float64          `json:"error_rate"`
    RequestsPerSecond float64        `json:"requests_per_second"`
    StartTime       time.Time        `json:"start_time"`
    mutex           sync.RWMutex
}

type LoggingResponseWriter struct {
    http.ResponseWriter
    statusCode    int
    contentLength int64
}

func (lrw *LoggingResponseWriter) WriteHeader(statusCode int) {
    lrw.statusCode = statusCode
    lrw.ResponseWriter.WriteHeader(statusCode)
}

func (lrw *LoggingResponseWriter) Write(data []byte) (int, error) {
    n, err := lrw.ResponseWriter.Write(data)
    lrw.contentLength += int64(n)
    return n, err
}

var (
    metrics = &Metrics{
        RequestsByMethod: make(map[string]int64),
        RequestsByStatus: make(map[int]int64),
        StartTime:       time.Now(),
    }
    
    logFile *os.File
    logger  *log.Logger
)

func init() {
    var err error
    logFile, err = os.OpenFile("access.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatal("Failed to open log file:", err)
    }
    
    logger = log.New(logFile, "", 0)
}

func generateRequestID() string {
    return fmt.Sprintf("req_%d_%d", time.Now().UnixNano(), os.Getpid())
}

func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        requestID := generateRequestID()
        
        // Add request ID to context and response header
        w.Header().Set("X-Request-ID", requestID)
        
        // Wrap response writer
        lrw := &LoggingResponseWriter{
            ResponseWriter: w,
            statusCode:    200, // Default status code
        }
        
        // Process request
        next.ServeHTTP(lrw, r)
        
        // Calculate duration
        duration := time.Since(start)
        
        // Create log entry
        logEntry := RequestLog{
            Timestamp:     start,
            Method:        r.Method,
            URL:           r.URL.String(),
            RemoteAddr:    getClientIP(r),
            UserAgent:     r.UserAgent(),
            StatusCode:    lrw.statusCode,
            ContentLength: lrw.contentLength,
            Duration:      duration,
            RequestID:     requestID,
        }
        
        // Add error information for non-success status codes
        if lrw.statusCode >= 400 {
            logEntry.Error = http.StatusText(lrw.statusCode)
        }
        
        // Log to file
        logJSON, _ := json.Marshal(logEntry)
        logger.Println(string(logJSON))
        
        // Update metrics
        updateMetrics(logEntry)
        
        // Console logging for development
        logToConsole(logEntry)
    })
}

func getClientIP(r *http.Request) string {
    forwarded := r.Header.Get("X-Forwarded-For")
    if forwarded != "" {
        return forwarded
    }
    
    realIP := r.Header.Get("X-Real-IP")
    if realIP != "" {
        return realIP
    }
    
    return r.RemoteAddr
}

func updateMetrics(logEntry RequestLog) {
    metrics.mutex.Lock()
    defer metrics.mutex.Unlock()
    
    metrics.TotalRequests++
    metrics.RequestsByMethod[logEntry.Method]++
    metrics.RequestsByStatus[logEntry.StatusCode]++
    metrics.TotalLatency += logEntry.Duration
    
    // Calculate average latency
    metrics.AverageLatency = metrics.TotalLatency / time.Duration(metrics.TotalRequests)
    
    // Calculate error rate
    errorCount := int64(0)
    for status, count := range metrics.RequestsByStatus {
        if status >= 400 {
            errorCount += count
        }
    }
    metrics.ErrorRate = float64(errorCount) / float64(metrics.TotalRequests) * 100
    
    // Calculate requests per second
    elapsed := time.Since(metrics.StartTime).Seconds()
    if elapsed > 0 {
        metrics.RequestsPerSecond = float64(metrics.TotalRequests) / elapsed
    }
}

func logToConsole(logEntry RequestLog) {
    status := logEntry.StatusCode
    var statusColor string
    
    switch {
    case status >= 200 && status < 300:
        statusColor = "\033[32m" // Green
    case status >= 300 && status < 400:
        statusColor = "\033[33m" // Yellow
    case status >= 400 && status < 500:
        statusColor = "\033[31m" // Red
    default:
        statusColor = "\033[35m" // Magenta
    }
    
    resetColor := "\033[0m"
    
    fmt.Printf("[%s] %s%s %d%s %s %s %v\n",
        logEntry.Timestamp.Format("2006-01-02 15:04:05"),
        statusColor,
        logEntry.Method,
        logEntry.StatusCode,
        resetColor,
        logEntry.URL,
        logEntry.RemoteAddr,
        logEntry.Duration,
    )
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate different response scenarios
    path := r.URL.Path
    
    switch path {
    case "/api/success":
        response := map[string]interface{}{
            "message":   "Hello there! Request successful",
            "timestamp": time.Now().Format(time.RFC3339),
            "method":    r.Method,
        }
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
        
    case "/api/slow":
        time.Sleep(2 * time.Second)
        response := map[string]interface{}{
            "message": "Hello there! Slow response completed",
            "delay":   "2 seconds",
        }
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
        
    case "/api/error":
        http.Error(w, "Simulated server error", http.StatusInternalServerError)
        
    case "/api/not-found":
        http.Error(w, "Resource not found", http.StatusNotFound)
        
    case "/api/unauthorized":
        http.Error(w, "Unauthorized access", http.StatusUnauthorized)
        
    default:
        response := map[string]interface{}{
            "message": "Hello there! Generic API response",
            "path":    path,
        }
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
    }
}

func metricsHandler(w http.ResponseWriter, r *http.Request) {
    metrics.mutex.RLock()
    defer metrics.mutex.RUnlock()
    
    uptime := time.Since(metrics.StartTime)
    
    response := map[string]interface{}{
        "message": "Server metrics and statistics",
        "metrics": map[string]interface{}{
            "total_requests":     metrics.TotalRequests,
            "requests_by_method": metrics.RequestsByMethod,
            "requests_by_status": metrics.RequestsByStatus,
            "average_latency_ms": metrics.AverageLatency.Milliseconds(),
            "error_rate_percent": metrics.ErrorRate,
            "requests_per_second": metrics.RequestsPerSecond,
            "uptime_seconds":     uptime.Seconds(),
        },
        "server_info": map[string]interface{}{
            "start_time": metrics.StartTime.Format(time.RFC3339),
            "uptime":     uptime.String(),
            "pid":        os.Getpid(),
        },
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    metrics.mutex.RLock()
    errorRate := metrics.ErrorRate
    avgLatency := metrics.AverageLatency
    metrics.mutex.RUnlock()
    
    status := "healthy"
    httpStatus := http.StatusOK
    
    // Simple health checks
    if errorRate > 50 {
        status = "unhealthy"
        httpStatus = http.StatusServiceUnavailable
    } else if errorRate > 20 || avgLatency > 5*time.Second {
        status = "degraded"
    }
    
    response := map[string]interface{}{
        "status":            status,
        "timestamp":         time.Now().Format(time.RFC3339),
        "error_rate":        errorRate,
        "average_latency_ms": avgLatency.Milliseconds(),
        "checks": map[string]interface{}{
            "error_rate_ok":  errorRate <= 50,
            "latency_ok":     avgLatency <= 5*time.Second,
            "log_file_ok":    logFile != nil,
        },
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(httpStatus)
    json.NewEncoder(w).Encode(response)
}

func logsHandler(w http.ResponseWriter, r *http.Request) {
    // Read recent log entries
    file, err := os.Open("access.log")
    if err != nil {
        http.Error(w, "Unable to read logs", http.StatusInternalServerError)
        return
    }
    defer file.Close()
    
    // For demo purposes, read last 10 lines
    // In production, use proper log rotation and pagination
    
    w.Header().Set("Content-Type", "text/plain")
    w.Header().Set("Cache-Control", "no-cache")
    
    // Simple approach: read entire file and show last entries
    content := make([]byte, 10*1024) // Read last 10KB
    file.Seek(-int64(len(content)), 2) // Seek from end
    n, _ := file.Read(content)
    
    fmt.Fprintf(w, "Recent log entries:\n\n%s", content[:n])
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Logging and Monitoring Demo</title>
        <meta http-equiv="refresh" content="30">
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .metrics { background: #f5f5f5; padding: 20px; margin: 20px 0; }
            .status-ok { color: green; }
            .status-warning { color: orange; }
            .status-error { color: red; }
            pre { background: #333; color: #fff; padding: 15px; overflow-x: auto; }
        </style>
    </head>
    <body>
        <h1>HTTP Logging and Monitoring Demo</h1>
        <p>Hello there! This page demonstrates HTTP request logging and monitoring.</p>
        
        <div class="metrics">
            <h2>Quick Actions</h2>
            <button onclick="makeRequest('/api/success')">Success Request</button>
            <button onclick="makeRequest('/api/slow')">Slow Request</button>
            <button onclick="makeRequest('/api/error')">Error Request</button>
            <button onclick="makeRequest('/api/not-found')">Not Found</button>
        </div>
        
        <div class="metrics">
            <h2>Monitoring Endpoints</h2>
            <ul>
                <li><a href="/metrics">Metrics</a> - Server statistics</li>
                <li><a href="/health">Health Check</a> - Service health status</li>
                <li><a href="/logs">Recent Logs</a> - Log file entries</li>
            </ul>
        </div>
        
        <div class="metrics">
            <h2>Test Commands</h2>
            <pre>
# Generate some traffic
for i in {1..10}; do curl http://localhost:8080/api/success; done

# Test different endpoints
curl http://localhost:8080/api/slow
curl http://localhost:8080/api/error
curl http://localhost:8080/api/not-found

# Check metrics
curl http://localhost:8080/metrics | jq '.'

# Check health
curl http://localhost:8080/health | jq '.'
            </pre>
        </div>
        
        <div class="metrics">
            <h2>Logging Features</h2>
            <ul>
                <li>Structured JSON logging</li>
                <li>Request/response metrics</li>
                <li>Performance monitoring</li>
                <li>Error rate tracking</li>
                <li>Client IP detection</li>
                <li>Request ID correlation</li>
                <li>Console and file logging</li>
            </ul>
        </div>
        
        <script>
        function makeRequest(endpoint) {
            fetch(endpoint)
                .then(response => {
                    console.log(endpoint + ' responded with status: ' + response.status);
                    return response.json().catch(() => ({ error: 'Non-JSON response' }));
                })
                .then(data => console.log(endpoint + ' data:', data))
                .catch(error => console.error(endpoint + ' error:', error));
        }
        
        // Auto-refresh metrics every 30 seconds
        setInterval(() => {
            if (window.location.pathname === '/metrics' || 
                window.location.pathname === '/health') {
                window.location.reload();
            }
        }, 30000);
        </script>
    </body>
    </html>`)
}

func main() {
    defer logFile.Close()
    
    mux := http.NewServeMux()
    mux.HandleFunc("/", demoPageHandler)
    mux.HandleFunc("/api/", apiHandler)
    mux.HandleFunc("/metrics", metricsHandler)
    mux.HandleFunc("/health", healthHandler)
    mux.HandleFunc("/logs", logsHandler)
    
    // Apply logging middleware
    handler := loggingMiddleware(mux)
    
    fmt.Println("=== HTTP Logging and Monitoring Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println("Logs are written to: access.log")
    fmt.Println()
    fmt.Println("Monitoring endpoints:")
    fmt.Println("  http://localhost:8080/metrics - Server metrics")
    fmt.Println("  http://localhost:8080/health - Health check")
    fmt.Println("  http://localhost:8080/logs - Recent log entries")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Structured request logging")
    fmt.Println("• Real-time metrics collection")
    fmt.Println("• Performance monitoring")
    fmt.Println("• Health check endpoints")
    fmt.Println("• Error rate tracking")
    fmt.Println("• Request correlation")

    log.Fatal(http.ListenAndServe(":8080", handler))
}
```

This logging and monitoring example demonstrates comprehensive request  
tracking, metrics collection, health checks, and performance monitoring.  
The implementation shows how to build production-ready observability  
features that are essential for maintaining and debugging HTTP services.

## HTTP request routing with patterns

Advanced routing enables clean URL structures and parameter extraction.  
This example demonstrates pattern-based routing, middleware per route,  
and RESTful API design patterns.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type Route struct {
    Pattern    string
    Method     string
    Handler    http.HandlerFunc
    Middleware []func(http.HandlerFunc) http.HandlerFunc
}

type Router struct {
    routes []Route
}

type RouteParams map[string]string

func NewRouter() *Router {
    return &Router{
        routes: make([]Route, 0),
    }
}

func (r *Router) Add(method, pattern string, handler http.HandlerFunc, 
                    middleware ...func(http.HandlerFunc) http.HandlerFunc) {
    r.routes = append(r.routes, Route{
        Pattern:    pattern,
        Method:     method,
        Handler:    handler,
        Middleware: middleware,
    })
}

func (r *Router) GET(pattern string, handler http.HandlerFunc, 
                    middleware ...func(http.HandlerFunc) http.HandlerFunc) {
    r.Add("GET", pattern, handler, middleware...)
}

func (r *Router) POST(pattern string, handler http.HandlerFunc, 
                     middleware ...func(http.HandlerFunc) http.HandlerFunc) {
    r.Add("POST", pattern, handler, middleware...)
}

func (r *Router) PUT(pattern string, handler http.HandlerFunc, 
                    middleware ...func(http.HandlerFunc) http.HandlerFunc) {
    r.Add("PUT", pattern, handler, middleware...)
}

func (r *Router) DELETE(pattern string, handler http.HandlerFunc, 
                       middleware ...func(http.HandlerFunc) http.HandlerFunc) {
    r.Add("DELETE", pattern, handler, middleware...)
}

func (r *Router) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for _, route := range r.routes {
        if route.Method != req.Method {
            continue
        }
        
        params, match := matchPattern(route.Pattern, req.URL.Path)
        if !match {
            continue
        }
        
        // Add params to request context
        req = req.WithContext(withRouteParams(req.Context(), params))
        
        // Apply middleware chain
        handler := route.Handler
        for i := len(route.Middleware) - 1; i >= 0; i-- {
            handler = route.Middleware[i](handler)
        }
        
        handler(w, req)
        return
    }
    
    // No route matched
    http.NotFound(w, req)
}

func matchPattern(pattern, path string) (RouteParams, bool) {
    patternParts := strings.Split(strings.Trim(pattern, "/"), "/")
    pathParts := strings.Split(strings.Trim(path, "/"), "/")
    
    if len(patternParts) != len(pathParts) {
        return nil, false
    }
    
    params := make(RouteParams)
    
    for i, patternPart := range patternParts {
        pathPart := pathParts[i]
        
        if strings.HasPrefix(patternPart, ":") {
            // Named parameter
            paramName := patternPart[1:]
            params[paramName] = pathPart
        } else if strings.HasPrefix(patternPart, "*") {
            // Wildcard parameter
            paramName := patternPart[1:]
            if paramName == "" {
                paramName = "wildcard"
            }
            // For simplicity, just capture this segment
            params[paramName] = pathPart
        } else if patternPart != pathPart {
            // Exact match failed
            return nil, false
        }
    }
    
    return params, true
}

type contextKey string

const routeParamsKey contextKey = "routeParams"

func withRouteParams(ctx context.Context, params RouteParams) context.Context {
    return context.WithValue(ctx, routeParamsKey, params)
}

func GetRouteParams(r *http.Request) RouteParams {
    if params, ok := r.Context().Value(routeParamsKey).(RouteParams); ok {
        return params
    }
    return make(RouteParams)
}

func GetRouteParam(r *http.Request, key string) string {
    params := GetRouteParams(r)
    return params[key]
}

// Middleware functions
func authMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "Authorization required", http.StatusUnauthorized)
            return
        }
        
        // Simple token validation
        if !strings.HasPrefix(token, "Bearer ") {
            http.Error(w, "Invalid authorization format", http.StatusUnauthorized)
            return
        }
        
        next(w, r)
    }
}

func corsMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")
        
        if r.Method == "OPTIONS" {
            w.WriteHeader(http.StatusOK)
            return
        }
        
        next(w, r)
    }
}

func jsonMiddleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("Content-Type", "application/json")
        next(w, r)
    }
}

// Handlers
func homeHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message":   "Hello there! Welcome to the routing demo",
        "timestamp": time.Now().Format(time.RFC3339),
        "routes": []string{
            "GET /",
            "GET /users",
            "GET /users/:id",
            "POST /users",
            "PUT /users/:id",
            "DELETE /users/:id",
            "GET /posts/:id/comments",
            "GET /files/*path",
            "GET /admin/stats (requires auth)",
        },
    }
    
    json.NewEncoder(w).Encode(response)
}

func getUsersHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate user data
    users := []map[string]interface{}{
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"},
        {"id": 3, "name": "Charlie", "email": "charlie@example.com"},
    }
    
    response := map[string]interface{}{
        "message": "User list retrieved successfully",
        "users":   users,
        "count":   len(users),
    }
    
    json.NewEncoder(w).Encode(response)
}

func getUserHandler(w http.ResponseWriter, r *http.Request) {
    userID := GetRouteParam(r, "id")
    
    id, err := strconv.Atoi(userID)
    if err != nil {
        http.Error(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    // Simulate user lookup
    user := map[string]interface{}{
        "id":    id,
        "name":  fmt.Sprintf("User %d", id),
        "email": fmt.Sprintf("user%d@example.com", id),
    }
    
    response := map[string]interface{}{
        "message": "User retrieved successfully",
        "user":    user,
    }
    
    json.NewEncoder(w).Encode(response)
}

func createUserHandler(w http.ResponseWriter, r *http.Request) {
    var userData map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&userData); err != nil {
        http.Error(w, "Invalid JSON data", http.StatusBadRequest)
        return
    }
    
    // Simulate user creation
    newUser := map[string]interface{}{
        "id":         123,
        "name":       userData["name"],
        "email":      userData["email"],
        "created_at": time.Now().Format(time.RFC3339),
    }
    
    response := map[string]interface{}{
        "message": "User created successfully",
        "user":    newUser,
    }
    
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(response)
}

func updateUserHandler(w http.ResponseWriter, r *http.Request) {
    userID := GetRouteParam(r, "id")
    
    var userData map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&userData); err != nil {
        http.Error(w, "Invalid JSON data", http.StatusBadRequest)
        return
    }
    
    id, _ := strconv.Atoi(userID)
    updatedUser := map[string]interface{}{
        "id":         id,
        "name":       userData["name"],
        "email":      userData["email"],
        "updated_at": time.Now().Format(time.RFC3339),
    }
    
    response := map[string]interface{}{
        "message": "User updated successfully",
        "user":    updatedUser,
    }
    
    json.NewEncoder(w).Encode(response)
}

func deleteUserHandler(w http.ResponseWriter, r *http.Request) {
    userID := GetRouteParam(r, "id")
    
    response := map[string]interface{}{
        "message": "User deleted successfully",
        "user_id": userID,
    }
    
    json.NewEncoder(w).Encode(response)
}

func getCommentsHandler(w http.ResponseWriter, r *http.Request) {
    postID := GetRouteParam(r, "id")
    
    // Simulate comments for a post
    comments := []map[string]interface{}{
        {"id": 1, "text": "Great post!", "author": "Alice"},
        {"id": 2, "text": "Thanks for sharing", "author": "Bob"},
    }
    
    response := map[string]interface{}{
        "message":  "Comments retrieved successfully",
        "post_id":  postID,
        "comments": comments,
        "count":    len(comments),
    }
    
    json.NewEncoder(w).Encode(response)
}

func getFileHandler(w http.ResponseWriter, r *http.Request) {
    filePath := GetRouteParam(r, "path")
    
    response := map[string]interface{}{
        "message":   "File access request",
        "file_path": filePath,
        "full_path": r.URL.Path,
        "note":      "In a real application, this would serve the actual file",
    }
    
    json.NewEncoder(w).Encode(response)
}

func adminStatsHandler(w http.ResponseWriter, r *http.Request) {
    // This handler requires authentication
    stats := map[string]interface{}{
        "total_users":    150,
        "active_users":   75,
        "total_posts":    340,
        "server_uptime":  "5 days",
        "memory_usage":   "256 MB",
        "disk_usage":     "12.5 GB",
    }
    
    response := map[string]interface{}{
        "message": "Hello there! Admin statistics",
        "stats":   stats,
    }
    
    json.NewEncoder(w).Encode(response)
}

func routeInfoHandler(w http.ResponseWriter, r *http.Request) {
    params := GetRouteParams(r)
    
    response := map[string]interface{}{
        "message":     "Route information",
        "method":      r.Method,
        "path":        r.URL.Path,
        "params":      params,
        "query":       r.URL.Query(),
        "headers":     r.Header,
        "timestamp":   time.Now().Format(time.RFC3339),
    }
    
    json.NewEncoder(w).Encode(response)
}

func main() {
    router := NewRouter()
    
    // Public routes
    router.GET("/", homeHandler, jsonMiddleware, corsMiddleware)
    router.GET("/users", getUsersHandler, jsonMiddleware, corsMiddleware)
    router.GET("/users/:id", getUserHandler, jsonMiddleware, corsMiddleware)
    router.POST("/users", createUserHandler, jsonMiddleware, corsMiddleware)
    router.PUT("/users/:id", updateUserHandler, jsonMiddleware, corsMiddleware)
    router.DELETE("/users/:id", deleteUserHandler, jsonMiddleware, corsMiddleware)
    
    // Nested resource routes
    router.GET("/posts/:id/comments", getCommentsHandler, jsonMiddleware, corsMiddleware)
    
    // Wildcard routes
    router.GET("/files/*path", getFileHandler, jsonMiddleware, corsMiddleware)
    
    // Protected routes
    router.GET("/admin/stats", adminStatsHandler, jsonMiddleware, corsMiddleware, authMiddleware)
    
    // Route debugging
    router.GET("/route-info", routeInfoHandler, jsonMiddleware, corsMiddleware)
    
    server := &http.Server{
        Addr:    ":8080",
        Handler: router,
    }
    
    fmt.Println("=== HTTP Request Routing Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Available routes:")
    fmt.Println("  GET    /")
    fmt.Println("  GET    /users")
    fmt.Println("  GET    /users/:id")
    fmt.Println("  POST   /users")
    fmt.Println("  PUT    /users/:id")
    fmt.Println("  DELETE /users/:id")
    fmt.Println("  GET    /posts/:id/comments")
    fmt.Println("  GET    /files/*path")
    fmt.Println("  GET    /admin/stats (requires auth)")
    fmt.Println("  GET    /route-info")
    fmt.Println()
    fmt.Println("Test commands:")
    fmt.Println("  curl http://localhost:8080/")
    fmt.Println("  curl http://localhost:8080/users/123")
    fmt.Println("  curl http://localhost:8080/posts/456/comments")
    fmt.Println("  curl http://localhost:8080/files/docs/readme.txt")
    fmt.Println("  curl -H 'Authorization: Bearer token' http://localhost:8080/admin/stats")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Pattern-based routing")
    fmt.Println("• Named parameters (:id)")
    fmt.Println("• Wildcard parameters (*path)")
    fmt.Println("• Per-route middleware")
    fmt.Println("• RESTful API patterns")
    fmt.Println("• Route parameter extraction")
    
    log.Fatal(server.ListenAndServe())
}
```

This routing example demonstrates advanced URL pattern matching, parameter  
extraction, middleware composition, and RESTful API design. The implementation  
shows how to build a flexible routing system that supports named parameters,  
wildcards, and per-route middleware for building complex web applications.

## HTTP streaming responses

Streaming responses enable real-time data transmission and improve perceived  
performance. This example demonstrates server-sent events, chunked responses,  
and real-time data streaming techniques.

```go
package main

import (
    "bufio"
    "encoding/json"
    "fmt"
    "log"
    "math/rand"
    "net/http"
    "strconv"
    "time"
)

type StreamMessage struct {
    ID        string    `json:"id"`
    Type      string    `json:"type"`
    Data      interface{} `json:"data"`
    Timestamp time.Time `json:"timestamp"`
}

func sseHandler(w http.ResponseWriter, r *http.Request) {
    // Set headers for server-sent events
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    w.Header().Set("Access-Control-Allow-Origin", "*")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    // Send initial connection message
    fmt.Fprintf(w, "event: connected\n")
    fmt.Fprintf(w, "data: {\"message\": \"Hello there! Connected to stream\", \"timestamp\": \"%s\"}\n\n", 
               time.Now().Format(time.RFC3339))
    flusher.Flush()
    
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()
    
    messageID := 1
    
    for {
        select {
        case <-ticker.C:
            // Send periodic updates
            message := StreamMessage{
                ID:   fmt.Sprintf("msg_%d", messageID),
                Type: "update",
                Data: map[string]interface{}{
                    "counter":     messageID,
                    "random":      rand.Intn(100),
                    "temperature": 20 + rand.Float64()*10,
                    "status":      []string{"active", "idle", "busy"}[rand.Intn(3)],
                },
                Timestamp: time.Now(),
            }
            
            jsonData, _ := json.Marshal(message)
            
            fmt.Fprintf(w, "id: %s\n", message.ID)
            fmt.Fprintf(w, "event: update\n")
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            
            messageID++
            
        case <-r.Context().Done():
            // Client disconnected
            fmt.Fprintf(w, "event: disconnected\n")
            fmt.Fprintf(w, "data: {\"message\": \"Stream disconnected\"}\n\n")
            flusher.Flush()
            return
        }
    }
}

func progressStreamHandler(w http.ResponseWriter, r *http.Request) {
    totalSteps := 50
    if stepsParam := r.URL.Query().Get("steps"); stepsParam != "" {
        if steps, err := strconv.Atoi(stepsParam); err == nil && steps > 0 {
            totalSteps = steps
        }
    }
    
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    // Send start event
    fmt.Fprintf(w, "event: start\n")
    fmt.Fprintf(w, "data: {\"message\": \"Starting progress stream\", \"total\": %d}\n\n", totalSteps)
    flusher.Flush()
    
    for i := 0; i <= totalSteps; i++ {
        select {
        case <-time.After(100 * time.Millisecond):
            progress := float64(i) / float64(totalSteps) * 100
            
            progressData := map[string]interface{}{
                "step":     i,
                "total":    totalSteps,
                "progress": progress,
                "eta":      fmt.Sprintf("%.1fs", float64(totalSteps-i)*0.1),
            }
            
            jsonData, _ := json.Marshal(progressData)
            
            fmt.Fprintf(w, "event: progress\n")
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            
            if i == totalSteps {
                fmt.Fprintf(w, "event: complete\n")
                fmt.Fprintf(w, "data: {\"message\": \"Progress completed successfully\"}\n\n")
                flusher.Flush()
                return
            }
            
        case <-r.Context().Done():
            fmt.Fprintf(w, "event: cancelled\n")
            fmt.Fprintf(w, "data: {\"message\": \"Progress cancelled\"}\n\n")
            flusher.Flush()
            return
        }
    }
}

func chunkedResponseHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Transfer-Encoding", "chunked")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Chunked encoding not supported", http.StatusInternalServerError)
        return
    }
    
    // Start JSON array
    fmt.Fprint(w, "[\n")
    flusher.Flush()
    
    for i := 0; i < 10; i++ {
        if i > 0 {
            fmt.Fprint(w, ",\n")
        }
        
        item := map[string]interface{}{
            "id":        i + 1,
            "message":   fmt.Sprintf("Hello there! This is item %d", i+1),
            "timestamp": time.Now().Format(time.RFC3339),
            "delay":     i * 500,
        }
        
        jsonData, _ := json.MarshalIndent(item, "  ", "  ")
        fmt.Fprintf(w, "  %s", jsonData)
        flusher.Flush()
        
        // Simulate processing delay
        time.Sleep(500 * time.Millisecond)
    }
    
    // End JSON array
    fmt.Fprint(w, "\n]\n")
    flusher.Flush()
}

func dataStreamHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/plain")
    w.Header().Set("Transfer-Encoding", "chunked")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    duration := 30 * time.Second
    if durationParam := r.URL.Query().Get("duration"); durationParam != "" {
        if d, err := time.ParseDuration(durationParam); err == nil {
            duration = d
        }
    }
    
    start := time.Now()
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    lineNumber := 1
    
    fmt.Fprintf(w, "Starting data stream for %v...\n", duration)
    flusher.Flush()
    
    for {
        select {
        case <-ticker.C:
            elapsed := time.Since(start)
            if elapsed > duration {
                fmt.Fprintf(w, "Stream completed after %v\n", elapsed)
                flusher.Flush()
                return
            }
            
            data := fmt.Sprintf("[%03d] %s - Elapsed: %v, Random: %d\n",
                               lineNumber,
                               time.Now().Format("15:04:05"),
                               elapsed.Round(time.Second),
                               rand.Intn(1000))
            
            fmt.Fprint(w, data)
            flusher.Flush()
            
            lineNumber++
            
        case <-r.Context().Done():
            fmt.Fprintf(w, "\nStream interrupted by client disconnect\n")
            flusher.Flush()
            return
        }
    }
}

func logStreamHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    // Simulate log entries
    logLevels := []string{"INFO", "WARN", "ERROR", "DEBUG"}
    components := []string{"auth", "api", "database", "cache", "worker"}
    
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    entryID := 1
    
    for {
        select {
        case <-ticker.C:
            level := logLevels[rand.Intn(len(logLevels))]
            component := components[rand.Intn(len(components))]
            
            logEntry := map[string]interface{}{
                "id":        entryID,
                "timestamp": time.Now().Format(time.RFC3339),
                "level":     level,
                "component": component,
                "message":   fmt.Sprintf("Sample log message from %s component", component),
                "metadata": map[string]interface{}{
                    "pid":      rand.Intn(10000),
                    "thread":   rand.Intn(100),
                    "duration": rand.Intn(1000),
                },
            }
            
            jsonData, _ := json.Marshal(logEntry)
            
            fmt.Fprintf(w, "id: %d\n", entryID)
            fmt.Fprintf(w, "event: log\n")
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            
            entryID++
            
        case <-r.Context().Done():
            return
        }
    }
}

func chatStreamHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    
    flusher, ok := w.(http.Flusher)
    if !ok {
        http.Error(w, "Streaming not supported", http.StatusInternalServerError)
        return
    }
    
    users := []string{"Alice", "Bob", "Charlie", "Diana", "Eve"}
    messageTypes := []string{"message", "join", "leave", "typing"}
    
    ticker := time.NewTicker(3 * time.Second)
    defer ticker.Stop()
    
    messageID := 1
    
    for {
        select {
        case <-ticker.C:
            msgType := messageTypes[rand.Intn(len(messageTypes))]
            user := users[rand.Intn(len(users))]
            
            var message map[string]interface{}
            
            switch msgType {
            case "message":
                messages := []string{
                    "Hello there everyone!",
                    "How's everyone doing?",
                    "Great to see you all here",
                    "Anyone working on Go projects?",
                    "This streaming demo is pretty cool",
                }
                message = map[string]interface{}{
                    "id":      messageID,
                    "type":    "message",
                    "user":    user,
                    "content": messages[rand.Intn(len(messages))],
                    "timestamp": time.Now().Format(time.RFC3339),
                }
            case "join":
                message = map[string]interface{}{
                    "id":      messageID,
                    "type":    "join",
                    "user":    user,
                    "content": fmt.Sprintf("%s joined the chat", user),
                    "timestamp": time.Now().Format(time.RFC3339),
                }
            case "leave":
                message = map[string]interface{}{
                    "id":      messageID,
                    "type":    "leave",
                    "user":    user,
                    "content": fmt.Sprintf("%s left the chat", user),
                    "timestamp": time.Now().Format(time.RFC3339),
                }
            case "typing":
                message = map[string]interface{}{
                    "id":      messageID,
                    "type":    "typing",
                    "user":    user,
                    "content": fmt.Sprintf("%s is typing...", user),
                    "timestamp": time.Now().Format(time.RFC3339),
                }
            }
            
            jsonData, _ := json.Marshal(message)
            
            fmt.Fprintf(w, "id: %d\n", messageID)
            fmt.Fprintf(w, "event: %s\n", msgType)
            fmt.Fprintf(w, "data: %s\n\n", jsonData)
            flusher.Flush()
            
            messageID++
            
        case <-r.Context().Done():
            return
        }
    }
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Streaming Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .stream-container { border: 1px solid #ccc; padding: 20px; margin: 20px 0; height: 300px; overflow-y: scroll; background: #f9f9f9; }
            .message { margin: 5px 0; padding: 5px; background: white; border-radius: 3px; }
            .log-entry { font-family: monospace; font-size: 12px; }
            .progress-bar { width: 100%; height: 20px; background: #f0f0f0; border-radius: 10px; overflow: hidden; }
            .progress-fill { height: 100%; background: #4CAF50; transition: width 0.3s; }
            button { margin: 5px; padding: 10px 15px; }
        </style>
    </head>
    <body>
        <h1>HTTP Streaming Demo</h1>
        <p>Hello there! This demonstrates various HTTP streaming techniques.</p>
        
        <h2>Server-Sent Events (SSE)</h2>
        <button onclick="startSSE()">Start SSE Stream</button>
        <button onclick="stopSSE()">Stop SSE Stream</button>
        <div id="sse-container" class="stream-container"></div>
        
        <h2>Progress Stream</h2>
        <button onclick="startProgress()">Start Progress</button>
        <div class="progress-bar">
            <div id="progress-fill" class="progress-fill" style="width: 0%"></div>
        </div>
        <div id="progress-info"></div>
        
        <h2>Log Stream</h2>
        <button onclick="startLogs()">Start Log Stream</button>
        <button onclick="stopLogs()">Stop Log Stream</button>
        <div id="log-container" class="stream-container"></div>
        
        <h2>Chat Stream</h2>
        <button onclick="startChat()">Start Chat Stream</button>
        <button onclick="stopChat()">Stop Chat Stream</button>
        <div id="chat-container" class="stream-container"></div>
        
        <h2>Test Endpoints</h2>
        <ul>
            <li><a href="/stream/sse" target="_blank">SSE Stream</a></li>
            <li><a href="/stream/progress" target="_blank">Progress Stream</a></li>
            <li><a href="/stream/chunked" target="_blank">Chunked Response</a></li>
            <li><a href="/stream/data" target="_blank">Data Stream</a></li>
        </ul>
        
        <script>
            let sseSource = null;
            let progressSource = null;
            let logSource = null;
            let chatSource = null;
            
            function startSSE() {
                stopSSE();
                const container = document.getElementById('sse-container');
                container.innerHTML = '';
                
                sseSource = new EventSource('/stream/sse');
                
                sseSource.onmessage = function(event) {
                    const data = JSON.parse(event.data);
                    addMessage(container, 'SSE: ' + JSON.stringify(data, null, 2));
                };
                
                sseSource.onerror = function(event) {
                    addMessage(container, 'SSE Error: Connection lost');
                };
            }
            
            function stopSSE() {
                if (sseSource) {
                    sseSource.close();
                    sseSource = null;
                }
            }
            
            function startProgress() {
                if (progressSource) progressSource.close();
                
                const progressFill = document.getElementById('progress-fill');
                const progressInfo = document.getElementById('progress-info');
                
                progressSource = new EventSource('/stream/progress?steps=100');
                
                progressSource.addEventListener('progress', function(event) {
                    const data = JSON.parse(event.data);
                    progressFill.style.width = data.progress + '%';
                    progressInfo.textContent = ` + `Step ${data.step}/${data.total} (${data.progress.toFixed(1)}%) - ETA: ${data.eta}` + `;
                });
                
                progressSource.addEventListener('complete', function(event) {
                    progressInfo.textContent = 'Progress completed!';
                    progressSource.close();
                });
            }
            
            function startLogs() {
                stopLogs();
                const container = document.getElementById('log-container');
                container.innerHTML = '';
                
                logSource = new EventSource('/stream/logs');
                
                logSource.addEventListener('log', function(event) {
                    const data = JSON.parse(event.data);
                    const logText = ` + `[${data.timestamp}] ${data.level} ${data.component}: ${data.message}` + `;
                    addMessage(container, logText, 'log-entry');
                });
            }
            
            function stopLogs() {
                if (logSource) {
                    logSource.close();
                    logSource = null;
                }
            }
            
            function startChat() {
                stopChat();
                const container = document.getElementById('chat-container');
                container.innerHTML = '';
                
                chatSource = new EventSource('/stream/chat');
                
                chatSource.onmessage = function(event) {
                    const data = JSON.parse(event.data);
                    let messageText = ` + `[${data.timestamp.substr(11, 8)}] ` + `;
                    
                    switch(data.type) {
                        case 'message':
                            messageText += ` + `<${data.user}> ${data.content}` + `;
                            break;
                        case 'join':
                        case 'leave':
                        case 'typing':
                            messageText += ` + `* ${data.content}` + `;
                            break;
                    }
                    
                    addMessage(container, messageText);
                };
            }
            
            function stopChat() {
                if (chatSource) {
                    chatSource.close();
                    chatSource = null;
                }
            }
            
            function addMessage(container, text, className = '') {
                const div = document.createElement('div');
                div.className = 'message ' + className;
                div.textContent = text;
                container.appendChild(div);
                container.scrollTop = container.scrollHeight;
                
                // Keep only last 50 messages
                while (container.children.length > 50) {
                    container.removeChild(container.firstChild);
                }
            }
            
            // Cleanup on page unload
            window.addEventListener('beforeunload', function() {
                stopSSE();
                if (progressSource) progressSource.close();
                stopLogs();
                stopChat();
            });
        </script>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", demoPageHandler)
    http.HandleFunc("/stream/sse", sseHandler)
    http.HandleFunc("/stream/progress", progressStreamHandler)
    http.HandleFunc("/stream/chunked", chunkedResponseHandler)
    http.HandleFunc("/stream/data", dataStreamHandler)
    http.HandleFunc("/stream/logs", logStreamHandler)
    http.HandleFunc("/stream/chat", chatStreamHandler)
    
    fmt.Println("=== HTTP Streaming Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Streaming endpoints:")
    fmt.Println("  /stream/sse - Server-sent events")
    fmt.Println("  /stream/progress - Progress updates")
    fmt.Println("  /stream/chunked - Chunked JSON response")
    fmt.Println("  /stream/data - Real-time data stream")
    fmt.Println("  /stream/logs - Log streaming")
    fmt.Println("  /stream/chat - Chat simulation")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Server-sent events (SSE)")
    fmt.Println("• Chunked transfer encoding")
    fmt.Println("• Real-time data streaming")
    fmt.Println("• Progress updates")
    fmt.Println("• Client disconnection handling")
    fmt.Println("• Event-based communication")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This streaming example demonstrates comprehensive real-time communication  
techniques including server-sent events, chunked responses, and progress  
streaming. The implementation shows how to build responsive applications  
that provide immediate feedback and real-time updates to users.

## HTTP multipart form handling

Multipart forms enable file uploads and complex form data submission.  
This example demonstrates multipart parsing, file handling, and form  
field processing with validation and error handling.

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "log"
    "mime/multipart"
    "net/http"
    "os"
    "path/filepath"
    "strconv"
    "strings"
    "time"
)

type FormData struct {
    Name        string                `json:"name"`
    Email       string                `json:"email"`
    Description string                `json:"description"`
    Category    string                `json:"category"`
    Tags        []string              `json:"tags"`
    Files       []FileInfo            `json:"files"`
}

type FileInfo struct {
    FieldName    string `json:"field_name"`
    Filename     string `json:"filename"`
    Size         int64  `json:"size"`
    ContentType  string `json:"content_type"`
    SavedPath    string `json:"saved_path"`
}

const maxMemory = 32 << 20 // 32MB

func multipartFormHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method == http.MethodGet {
        showUploadForm(w, r)
        return
    }
    
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse multipart form
    err := r.ParseMultipartForm(maxMemory)
    if err != nil {
        http.Error(w, "Error parsing multipart form: "+err.Error(), 
                  http.StatusBadRequest)
        return
    }
    
    formData := FormData{
        Files: make([]FileInfo, 0),
    }
    
    // Process form fields
    if values := r.MultipartForm.Value["name"]; len(values) > 0 {
        formData.Name = values[0]
    }
    if values := r.MultipartForm.Value["email"]; len(values) > 0 {
        formData.Email = values[0]
    }
    if values := r.MultipartForm.Value["description"]; len(values) > 0 {
        formData.Description = values[0]
    }
    if values := r.MultipartForm.Value["category"]; len(values) > 0 {
        formData.Category = values[0]
    }
    if values := r.MultipartForm.Value["tags"]; len(values) > 0 {
        formData.Tags = strings.Split(values[0], ",")
        // Trim whitespace from tags
        for i, tag := range formData.Tags {
            formData.Tags[i] = strings.TrimSpace(tag)
        }
    }
    
    // Process uploaded files
    for fieldName, fileHeaders := range r.MultipartForm.File {
        for _, fileHeader := range fileHeaders {
            fileInfo, err := saveUploadedFile(fieldName, fileHeader)
            if err != nil {
                log.Printf("Error saving file %s: %v", fileHeader.Filename, err)
                continue
            }
            formData.Files = append(formData.Files, *fileInfo)
        }
    }
    
    // Validate form data
    if err := validateFormData(formData); err != nil {
        http.Error(w, "Validation error: "+err.Error(), http.StatusBadRequest)
        return
    }
    
    // Send response
    response := map[string]interface{}{
        "message":     "Hello there! Form submitted successfully",
        "form_data":   formData,
        "timestamp":   time.Now().Format(time.RFC3339),
        "files_count": len(formData.Files),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func saveUploadedFile(fieldName string, fileHeader *multipart.FileHeader) (*FileInfo, error) {
    // Open uploaded file
    file, err := fileHeader.Open()
    if err != nil {
        return nil, fmt.Errorf("failed to open uploaded file: %w", err)
    }
    defer file.Close()
    
    // Create uploads directory if it doesn't exist
    uploadDir := "uploads"
    if err := os.MkdirAll(uploadDir, 0755); err != nil {
        return nil, fmt.Errorf("failed to create upload directory: %w", err)
    }
    
    // Generate unique filename
    ext := filepath.Ext(fileHeader.Filename)
    baseName := strings.TrimSuffix(fileHeader.Filename, ext)
    uniqueName := fmt.Sprintf("%s_%d%s", baseName, time.Now().UnixNano(), ext)
    savePath := filepath.Join(uploadDir, uniqueName)
    
    // Create destination file
    dst, err := os.Create(savePath)
    if err != nil {
        return nil, fmt.Errorf("failed to create destination file: %w", err)
    }
    defer dst.Close()
    
    // Copy file content
    size, err := io.Copy(dst, file)
    if err != nil {
        return nil, fmt.Errorf("failed to save file: %w", err)
    }
    
    return &FileInfo{
        FieldName:   fieldName,
        Filename:    fileHeader.Filename,
        Size:        size,
        ContentType: fileHeader.Header.Get("Content-Type"),
        SavedPath:   savePath,
    }, nil
}

func validateFormData(data FormData) error {
    if data.Name == "" {
        return fmt.Errorf("name is required")
    }
    if data.Email == "" {
        return fmt.Errorf("email is required")
    }
    if !strings.Contains(data.Email, "@") {
        return fmt.Errorf("invalid email format")
    }
    if len(data.Files) == 0 {
        return fmt.Errorf("at least one file is required")
    }
    
    // Validate file types and sizes
    for _, file := range data.Files {
        if file.Size > 10<<20 { // 10MB limit
            return fmt.Errorf("file %s is too large (max 10MB)", file.Filename)
        }
        
        allowedTypes := []string{
            "image/jpeg", "image/png", "image/gif",
            "text/plain", "application/pdf",
        }
        
        validType := false
        for _, allowedType := range allowedTypes {
            if file.ContentType == allowedType {
                validType = true
                break
            }
        }
        
        if !validType {
            return fmt.Errorf("file type %s not allowed for %s", 
                             file.ContentType, file.Filename)
        }
    }
    
    return nil
}

func showUploadForm(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Multipart Form Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; max-width: 800px; }
            .form-group { margin: 15px 0; }
            label { display: block; margin-bottom: 5px; font-weight: bold; }
            input, textarea, select { width: 100%; padding: 8px; box-sizing: border-box; }
            textarea { height: 100px; resize: vertical; }
            input[type="file"] { padding: 3px; }
            button { background: #007cba; color: white; padding: 10px 20px; border: none; cursor: pointer; }
            button:hover { background: #005a87; }
            .info { background: #f0f8ff; padding: 15px; border-left: 4px solid #007cba; margin: 20px 0; }
        </style>
    </head>
    <body>
        <h1>Multipart Form Upload Demo</h1>
        <p>Hello there! This form demonstrates multipart form handling with file uploads.</p>
        
        <div class="info">
            <h3>Upload Requirements:</h3>
            <ul>
                <li>Maximum file size: 10MB per file</li>
                <li>Allowed types: JPEG, PNG, GIF, PDF, Text files</li>
                <li>All fields are required</li>
                <li>At least one file must be uploaded</li>
            </ul>
        </div>
        
        <form action="/multipart" method="post" enctype="multipart/form-data">
            <div class="form-group">
                <label for="name">Name:</label>
                <input type="text" id="name" name="name" required>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required>
            </div>
            
            <div class="form-group">
                <label for="description">Description:</label>
                <textarea id="description" name="description" placeholder="Describe your upload..."></textarea>
            </div>
            
            <div class="form-group">
                <label for="category">Category:</label>
                <select id="category" name="category" required>
                    <option value="">Select a category</option>
                    <option value="documents">Documents</option>
                    <option value="images">Images</option>
                    <option value="media">Media</option>
                    <option value="other">Other</option>
                </select>
            </div>
            
            <div class="form-group">
                <label for="tags">Tags (comma-separated):</label>
                <input type="text" id="tags" name="tags" placeholder="tag1, tag2, tag3">
            </div>
            
            <div class="form-group">
                <label for="files">Primary File:</label>
                <input type="file" id="files" name="files" required>
            </div>
            
            <div class="form-group">
                <label for="attachments">Additional Files:</label>
                <input type="file" id="attachments" name="attachments" multiple>
            </div>
            
            <div class="form-group">
                <label for="thumbnail">Thumbnail (optional):</label>
                <input type="file" id="thumbnail" name="thumbnail" accept="image/*">
            </div>
            
            <button type="submit">Upload Files</button>
        </form>
        
        <div class="info">
            <h3>Test with curl:</h3>
            <pre>
curl -X POST http://localhost:8080/multipart \
  -F "name=John Doe" \
  -F "email=john@example.com" \
  -F "description=Test upload" \
  -F "category=documents" \
  -F "tags=test,demo" \
  -F "files=@test.txt" \
  -F "attachments=@image.jpg"
            </pre>
        </div>
    </body>
    </html>`)
}

func downloadFileHandler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    if filename == "" {
        http.Error(w, "File parameter required", http.StatusBadRequest)
        return
    }
    
    // Security: prevent path traversal
    filename = filepath.Base(filename)
    filePath := filepath.Join("uploads", filename)
    
    // Check if file exists
    if _, err := os.Stat(filePath); os.IsNotExist(err) {
        http.Error(w, "File not found", http.StatusNotFound)
        return
    }
    
    // Set appropriate headers
    w.Header().Set("Content-Disposition", 
                   fmt.Sprintf("attachment; filename=\"%s\"", filename))
    w.Header().Set("Content-Type", "application/octet-stream")
    
    http.ServeFile(w, r, filePath)
}

func listFilesHandler(w http.ResponseWriter, r *http.Request) {
    uploadDir := "uploads"
    
    files, err := os.ReadDir(uploadDir)
    if err != nil {
        if os.IsNotExist(err) {
            // No uploads directory exists yet
            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(map[string]interface{}{
                "message": "No files uploaded yet",
                "files":   []interface{}{},
                "count":   0,
            })
            return
        }
        http.Error(w, "Error reading uploads directory", 
                  http.StatusInternalServerError)
        return
    }
    
    var fileList []map[string]interface{}
    
    for _, file := range files {
        if !file.IsDir() {
            info, _ := file.Info()
            fileList = append(fileList, map[string]interface{}{
                "name":         file.Name(),
                "size":         info.Size(),
                "modified":     info.ModTime().Format(time.RFC3339),
                "download_url": fmt.Sprintf("/download?file=%s", file.Name()),
            })
        }
    }
    
    response := map[string]interface{}{
        "message": "File list retrieved successfully",
        "files":   fileList,
        "count":   len(fileList),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func ajaxUploadHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse multipart form with size limit
    r.ParseMultipartForm(maxMemory)
    
    file, header, err := r.FormFile("file")
    if err != nil {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusBadRequest)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "success": false,
            "error":   "No file uploaded",
        })
        return
    }
    defer file.Close()
    
    // Save file
    fileInfo, err := saveUploadedFile("ajax", header)
    if err != nil {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusInternalServerError)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "success": false,
            "error":   err.Error(),
        })
        return
    }
    
    // Return success response
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]interface{}{
        "success":  true,
        "message":  "File uploaded successfully",
        "file":     fileInfo,
        "timestamp": time.Now().Format(time.RFC3339),
    })
}

func main() {
    http.HandleFunc("/", showUploadForm)
    http.HandleFunc("/multipart", multipartFormHandler)
    http.HandleFunc("/download", downloadFileHandler)
    http.HandleFunc("/files", listFilesHandler)
    http.HandleFunc("/ajax-upload", ajaxUploadHandler)
    
    fmt.Println("=== HTTP Multipart Form Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Upload form")
    fmt.Println("  POST /multipart - Process multipart form")
    fmt.Println("  GET  /download?file=filename - Download file")
    fmt.Println("  GET  /files - List uploaded files")
    fmt.Println("  POST /ajax-upload - AJAX file upload")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Multipart form parsing")
    fmt.Println("• File upload handling")
    fmt.Println("• Form field processing")
    fmt.Println("• File validation")
    fmt.Println("• Multiple file uploads")
    fmt.Println("• AJAX upload support")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This multipart form example demonstrates comprehensive form handling  
including file uploads, validation, and multiple field types. The  
implementation shows proper multipart parsing, file saving, and  
security considerations for user-uploaded content.

## HTTP content negotiation

Content negotiation allows servers to provide different representations  
of resources based on client preferences. This example demonstrates  
Accept header processing and format selection.

```go
package main

import (
    "encoding/json"
    "encoding/xml"
    "fmt"
    "log"
    "net/http"
    "sort"
    "strconv"
    "strings"
    "time"
)

type User struct {
    ID       int       `json:"id" xml:"id"`
    Name     string    `json:"name" xml:"name"`
    Email    string    `json:"email" xml:"email"`
    Age      int       `json:"age" xml:"age"`
    Created  time.Time `json:"created" xml:"created"`
}

type UserList struct {
    XMLName xml.Name `xml:"users"`
    Users   []User   `xml:"user"`
    Count   int      `xml:"count,attr"`
}

type MediaType struct {
    Type       string
    Subtype    string
    Quality    float64
    Parameters map[string]string
}

var sampleUsers = []User{
    {1, "Alice Johnson", "alice@example.com", 28, time.Now().Add(-30 * 24 * time.Hour)},
    {2, "Bob Smith", "bob@example.com", 35, time.Now().Add(-45 * 24 * time.Hour)},
    {3, "Charlie Brown", "charlie@example.com", 22, time.Now().Add(-15 * 24 * time.Hour)},
}

func parseAcceptHeader(accept string) []MediaType {
    var mediaTypes []MediaType
    
    if accept == "" {
        return []MediaType{{Type: "*", Subtype: "*", Quality: 1.0}}
    }
    
    parts := strings.Split(accept, ",")
    
    for _, part := range parts {
        part = strings.TrimSpace(part)
        
        // Split media type and parameters
        segments := strings.Split(part, ";")
        mediaRange := strings.TrimSpace(segments[0])
        
        // Parse media type
        typeParts := strings.Split(mediaRange, "/")
        if len(typeParts) != 2 {
            continue
        }
        
        mediaType := MediaType{
            Type:       strings.TrimSpace(typeParts[0]),
            Subtype:    strings.TrimSpace(typeParts[1]),
            Quality:    1.0,
            Parameters: make(map[string]string),
        }
        
        // Parse parameters
        for i := 1; i < len(segments); i++ {
            param := strings.TrimSpace(segments[i])
            if kv := strings.SplitN(param, "=", 2); len(kv) == 2 {
                key := strings.TrimSpace(kv[0])
                value := strings.TrimSpace(kv[1])
                
                if key == "q" {
                    if quality, err := strconv.ParseFloat(value, 64); err == nil {
                        mediaType.Quality = quality
                    }
                } else {
                    mediaType.Parameters[key] = value
                }
            }
        }
        
        mediaTypes = append(mediaTypes, mediaType)
    }
    
    // Sort by quality (descending)
    sort.Slice(mediaTypes, func(i, j int) bool {
        return mediaTypes[i].Quality > mediaTypes[j].Quality
    })
    
    return mediaTypes
}

func selectContentType(acceptHeader string, available []string) string {
    mediaTypes := parseAcceptHeader(acceptHeader)
    
    for _, mediaType := range mediaTypes {
        for _, availableType := range available {
            parts := strings.Split(availableType, "/")
            if len(parts) != 2 {
                continue
            }
            
            if (mediaType.Type == "*" || mediaType.Type == parts[0]) &&
               (mediaType.Subtype == "*" || mediaType.Subtype == parts[1]) {
                return availableType
            }
        }
    }
    
    // Default to first available type
    if len(available) > 0 {
        return available[0]
    }
    
    return "text/plain"
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    accept := r.Header.Get("Accept")
    
    availableTypes := []string{
        "application/json",
        "application/xml",
        "text/html",
        "text/plain",
        "text/csv",
    }
    
    contentType := selectContentType(accept, availableTypes)
    
    switch contentType {
    case "application/json":
        serveJSON(w, sampleUsers)
    case "application/xml":
        serveXML(w, sampleUsers)
    case "text/html":
        serveHTML(w, sampleUsers)
    case "text/csv":
        serveCSV(w, sampleUsers)
    default:
        servePlainText(w, sampleUsers)
    }
}

func serveJSON(w http.ResponseWriter, users []User) {
    w.Header().Set("Content-Type", "application/json; charset=utf-8")
    
    response := map[string]interface{}{
        "message": "Hello there! User data in JSON format",
        "users":   users,
        "count":   len(users),
        "format":  "json",
    }
    
    json.NewEncoder(w).Encode(response)
}

func serveXML(w http.ResponseWriter, users []User) {
    w.Header().Set("Content-Type", "application/xml; charset=utf-8")
    
    userList := UserList{
        Users: users,
        Count: len(users),
    }
    
    w.Write([]byte(xml.Header))
    xml.NewEncoder(w).Encode(userList)
}

func serveHTML(w http.ResponseWriter, users []User) {
    w.Header().Set("Content-Type", "text/html; charset=utf-8")
    
    html := `<!DOCTYPE html>
<html>
<head>
    <title>User List</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        th { background-color: #f2f2f2; }
        tr:nth-child(even) { background-color: #f9f9f9; }
    </style>
</head>
<body>
    <h1>Hello there! User List (HTML Format)</h1>
    <p>Total users: ` + fmt.Sprintf("%d", len(users)) + `</p>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Age</th>
                <th>Created</th>
            </tr>
        </thead>
        <tbody>`
    
    for _, user := range users {
        html += fmt.Sprintf(`
            <tr>
                <td>%d</td>
                <td>%s</td>
                <td>%s</td>
                <td>%d</td>
                <td>%s</td>
            </tr>`, user.ID, user.Name, user.Email, user.Age, 
                   user.Created.Format("2006-01-02 15:04:05"))
    }
    
    html += `
        </tbody>
    </table>
    
    <h2>Test Different Formats</h2>
    <ul>
        <li><a href="?format=json">JSON Format</a></li>
        <li><a href="?format=xml">XML Format</a></li>
        <li><a href="?format=csv">CSV Format</a></li>
        <li><a href="?format=text">Plain Text</a></li>
    </ul>
    
    <h2>Content Negotiation Examples</h2>
    <pre>
curl -H "Accept: application/json" http://localhost:8080/users
curl -H "Accept: application/xml" http://localhost:8080/users
curl -H "Accept: text/html" http://localhost:8080/users
curl -H "Accept: text/csv" http://localhost:8080/users
curl -H "Accept: application/json;q=0.8,text/html;q=0.9" http://localhost:8080/users
    </pre>
</body>
</html>`
    
    fmt.Fprint(w, html)
}

func serveCSV(w http.ResponseWriter, users []User) {
    w.Header().Set("Content-Type", "text/csv; charset=utf-8")
    w.Header().Set("Content-Disposition", "attachment; filename=\"users.csv\"")
    
    // CSV header
    fmt.Fprint(w, "ID,Name,Email,Age,Created\n")
    
    // CSV data
    for _, user := range users {
        fmt.Fprintf(w, "%d,\"%s\",\"%s\",%d,\"%s\"\n",
                   user.ID, user.Name, user.Email, user.Age,
                   user.Created.Format("2006-01-02 15:04:05"))
    }
}

func servePlainText(w http.ResponseWriter, users []User) {
    w.Header().Set("Content-Type", "text/plain; charset=utf-8")
    
    fmt.Fprintf(w, "Hello there! User List (Plain Text Format)\n")
    fmt.Fprintf(w, "==========================================\n\n")
    fmt.Fprintf(w, "Total users: %d\n\n", len(users))
    
    for _, user := range users {
        fmt.Fprintf(w, "ID: %d\n", user.ID)
        fmt.Fprintf(w, "Name: %s\n", user.Name)
        fmt.Fprintf(w, "Email: %s\n", user.Email)
        fmt.Fprintf(w, "Age: %d\n", user.Age)
        fmt.Fprintf(w, "Created: %s\n", user.Created.Format("2006-01-02 15:04:05"))
        fmt.Fprintf(w, "---\n")
    }
}

func negotiationInfoHandler(w http.ResponseWriter, r *http.Request) {
    accept := r.Header.Get("Accept")
    mediaTypes := parseAcceptHeader(accept)
    
    availableTypes := []string{
        "application/json",
        "application/xml", 
        "text/html",
        "text/plain",
        "text/csv",
    }
    
    selectedType := selectContentType(accept, availableTypes)
    
    info := map[string]interface{}{
        "message":         "Content negotiation information",
        "accept_header":   accept,
        "parsed_types":    mediaTypes,
        "available_types": availableTypes,
        "selected_type":   selectedType,
        "timestamp":       time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(info)
}

func languageNegotiationHandler(w http.ResponseWriter, r *http.Request) {
    acceptLang := r.Header.Get("Accept-Language")
    
    // Simple language selection
    var language string
    var message string
    
    if strings.Contains(acceptLang, "es") {
        language = "es"
        message = "¡Hola! Bienvenido a nuestro servicio"
    } else if strings.Contains(acceptLang, "fr") {
        language = "fr"
        message = "Bonjour! Bienvenue à notre service"
    } else if strings.Contains(acceptLang, "de") {
        language = "de"
        message = "Hallo! Willkommen zu unserem Service"
    } else {
        language = "en"
        message = "Hello there! Welcome to our service"
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Content-Language", language)
    
    response := map[string]interface{}{
        "message":           message,
        "language":          language,
        "accept_language":   acceptLang,
        "timestamp":         time.Now().Format(time.RFC3339),
    }
    
    json.NewEncoder(w).Encode(response)
}

func compressionNegotiationHandler(w http.ResponseWriter, r *http.Request) {
    acceptEncoding := r.Header.Get("Accept-Encoding")
    
    // Check if gzip is supported
    supportsGzip := strings.Contains(acceptEncoding, "gzip")
    
    // Large content that benefits from compression
    content := strings.Repeat("Hello there! This content is repeated many times to demonstrate compression benefits. ", 100)
    
    response := map[string]interface{}{
        "message":              "Compression negotiation demo",
        "content":              content,
        "content_length":       len(content),
        "accept_encoding":      acceptEncoding,
        "compression_applied":  supportsGzip,
        "timestamp":            time.Now().Format(time.RFC3339),
    }
    
    if supportsGzip {
        w.Header().Set("Content-Encoding", "gzip")
        // Note: In a real implementation, you would apply gzip compression here
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func demoPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Content Negotiation Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .demo-section { background: #f5f5f5; padding: 20px; margin: 20px 0; border-radius: 5px; }
            pre { background: #333; color: #fff; padding: 15px; border-radius: 3px; overflow-x: auto; }
            .format-buttons button { margin: 5px; padding: 10px 15px; }
        </style>
    </head>
    <body>
        <h1>Content Negotiation Demo</h1>
        <p>Hello there! This demonstrates HTTP content negotiation.</p>
        
        <div class="demo-section">
            <h2>Format Selection</h2>
            <p>Choose your preferred format:</p>
            <div class="format-buttons">
                <button onclick="requestFormat('application/json')">JSON</button>
                <button onclick="requestFormat('application/xml')">XML</button>
                <button onclick="requestFormat('text/html')">HTML</button>
                <button onclick="requestFormat('text/csv')">CSV</button>
                <button onclick="requestFormat('text/plain')">Plain Text</button>
            </div>
            <div id="format-result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Quality Values</h2>
            <p>Test preference with quality values:</p>
            <button onclick="requestWithQuality()">JSON preferred, HTML fallback</button>
            <div id="quality-result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Language Negotiation</h2>
            <button onclick="requestLanguage('en')">English</button>
            <button onclick="requestLanguage('es')">Spanish</button>
            <button onclick="requestLanguage('fr')">French</button>
            <button onclick="requestLanguage('de')">German</button>
            <div id="language-result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Test Commands</h2>
            <pre>
# Request JSON format
curl -H "Accept: application/json" http://localhost:8080/users

# Request XML format  
curl -H "Accept: application/xml" http://localhost:8080/users

# Request with quality values
curl -H "Accept: application/json;q=0.8,text/html;q=0.9" http://localhost:8080/users

# Language negotiation
curl -H "Accept-Language: es-ES,es;q=0.9" http://localhost:8080/language

# Content negotiation info
curl http://localhost:8080/negotiation-info
            </pre>
        </div>
        
        <div class="demo-section">
            <h2>Available Endpoints</h2>
            <ul>
                <li><a href="/users">Users (content negotiation)</a></li>
                <li><a href="/negotiation-info">Negotiation info</a></li>
                <li><a href="/language">Language negotiation</a></li>
                <li><a href="/compression">Compression negotiation</a></li>
            </ul>
        </div>
        
        <script>
        function requestFormat(mimeType) {
            fetch('/users', {
                headers: {
                    'Accept': mimeType
                }
            })
            .then(response => {
                const contentType = response.headers.get('content-type');
                document.getElementById('format-result').innerHTML = 
                    '<h4>Response Content-Type: ' + contentType + '</h4>';
                return response.text();
            })
            .then(data => {
                document.getElementById('format-result').innerHTML += 
                    '<pre>' + data.substring(0, 500) + '...</pre>';
            })
            .catch(error => {
                document.getElementById('format-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function requestWithQuality() {
            fetch('/users', {
                headers: {
                    'Accept': 'application/json;q=0.8,text/html;q=0.9'
                }
            })
            .then(response => {
                const contentType = response.headers.get('content-type');
                document.getElementById('quality-result').innerHTML = 
                    '<h4>Selected: ' + contentType + '</h4>' +
                    '<p>Server chose HTML due to higher quality value (0.9 vs 0.8)</p>';
            });
        }
        
        function requestLanguage(lang) {
            fetch('/language', {
                headers: {
                    'Accept-Language': lang
                }
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('language-result').innerHTML = 
                    '<h4>Language: ' + data.language + '</h4>' +
                    '<p>Message: ' + data.message + '</p>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", demoPageHandler)
    http.HandleFunc("/users", usersHandler)
    http.HandleFunc("/negotiation-info", negotiationInfoHandler)
    http.HandleFunc("/language", languageNegotiationHandler)
    http.HandleFunc("/compression", compressionNegotiationHandler)
    
    fmt.Println("=== HTTP Content Negotiation Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Accept header parsing")
    fmt.Println("• Media type selection")
    fmt.Println("• Quality value handling")
    fmt.Println("• Multiple format support")
    fmt.Println("• Language negotiation")
    fmt.Println("• Compression negotiation")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This content negotiation example demonstrates sophisticated client-server  
communication where the server can provide different resource representations  
based on client preferences. The implementation shows proper Accept header  
parsing, quality value handling, and multiple format support for building  
flexible APIs.

## HTTP CORS (Cross-Origin Resource Sharing)

CORS enables controlled access to resources from different origins. This  
example demonstrates comprehensive CORS handling with preflight requests,  
credential support, and security considerations.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strings"
    "time"
)

type CORSConfig struct {
    AllowedOrigins   []string
    AllowedMethods   []string
    AllowedHeaders   []string
    ExposedHeaders   []string
    AllowCredentials bool
    MaxAge           int
}

func NewCORSConfig() *CORSConfig {
    return &CORSConfig{
        AllowedOrigins: []string{"*"},
        AllowedMethods: []string{"GET", "POST", "PUT", "DELETE", "OPTIONS"},
        AllowedHeaders: []string{"Content-Type", "Authorization", "X-Requested-With"},
        ExposedHeaders: []string{"X-Total-Count", "X-Page-Count"},
        AllowCredentials: false,
        MaxAge: 86400, // 24 hours
    }
}

func (c *CORSConfig) isOriginAllowed(origin string) bool {
    if len(c.AllowedOrigins) == 0 {
        return false
    }
    
    for _, allowed := range c.AllowedOrigins {
        if allowed == "*" {
            return true
        }
        if allowed == origin {
            return true
        }
        // Support wildcard subdomains like *.example.com
        if strings.HasPrefix(allowed, "*.") {
            domain := strings.TrimPrefix(allowed, "*.")
            if strings.HasSuffix(origin, domain) {
                return true
            }
        }
    }
    
    return false
}

func (c *CORSConfig) isMethodAllowed(method string) bool {
    for _, allowed := range c.AllowedMethods {
        if allowed == method {
            return true
        }
    }
    return false
}

func (c *CORSConfig) areHeadersAllowed(headers []string) bool {
    for _, header := range headers {
        found := false
        for _, allowed := range c.AllowedHeaders {
            if strings.EqualFold(header, allowed) {
                found = true
                break
            }
        }
        if !found {
            return false
        }
    }
    return true
}

func CORSMiddleware(config *CORSConfig) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            origin := r.Header.Get("Origin")
            
            // Check if origin is allowed
            if origin != "" && config.isOriginAllowed(origin) {
                w.Header().Set("Access-Control-Allow-Origin", origin)
            }
            
            // Handle preflight request
            if r.Method == http.MethodOptions {
                handlePreflightRequest(w, r, config)
                return
            }
            
            // Set CORS headers for actual request
            if config.AllowCredentials {
                w.Header().Set("Access-Control-Allow-Credentials", "true")
            }
            
            if len(config.ExposedHeaders) > 0 {
                w.Header().Set("Access-Control-Expose-Headers", 
                              strings.Join(config.ExposedHeaders, ", "))
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

func handlePreflightRequest(w http.ResponseWriter, r *http.Request, config *CORSConfig) {
    origin := r.Header.Get("Origin")
    requestMethod := r.Header.Get("Access-Control-Request-Method")
    requestHeaders := r.Header.Get("Access-Control-Request-Headers")
    
    // Check origin
    if origin == "" || !config.isOriginAllowed(origin) {
        w.WriteHeader(http.StatusForbidden)
        return
    }
    
    // Check method
    if requestMethod == "" || !config.isMethodAllowed(requestMethod) {
        w.WriteHeader(http.StatusMethodNotAllowed)
        return
    }
    
    // Check headers
    var headers []string
    if requestHeaders != "" {
        headers = strings.Split(requestHeaders, ",")
        for i, header := range headers {
            headers[i] = strings.TrimSpace(header)
        }
        
        if !config.areHeadersAllowed(headers) {
            w.WriteHeader(http.StatusForbidden)
            return
        }
    }
    
    // Set preflight response headers
    w.Header().Set("Access-Control-Allow-Origin", origin)
    w.Header().Set("Access-Control-Allow-Methods", 
                   strings.Join(config.AllowedMethods, ", "))
    w.Header().Set("Access-Control-Allow-Headers", 
                   strings.Join(config.AllowedHeaders, ", "))
    w.Header().Set("Access-Control-Max-Age", fmt.Sprintf("%d", config.MaxAge))
    
    if config.AllowCredentials {
        w.Header().Set("Access-Control-Allow-Credentials", "true")
    }
    
    w.WriteHeader(http.StatusOK)
}

func dataHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("X-Total-Count", "42")
    w.Header().Set("X-Page-Count", "3")
    
    response := map[string]interface{}{
        "message": "Hello there! This is CORS-enabled data",
        "data": []map[string]interface{}{
            {"id": 1, "name": "Alice", "role": "admin"},
            {"id": 2, "name": "Bob", "role": "user"},
            {"id": 3, "name": "Charlie", "role": "user"},
        },
        "timestamp": time.Now().Format(time.RFC3339),
        "origin":    r.Header.Get("Origin"),
        "method":    r.Method,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func secureDataHandler(w http.ResponseWriter, r *http.Request) {
    // This endpoint requires credentials
    authHeader := r.Header.Get("Authorization")
    if authHeader == "" {
        http.Error(w, "Authorization required", http.StatusUnauthorized)
        return
    }
    
    response := map[string]interface{}{
        "message":   "Hello there! This is secure CORS data",
        "user_info": map[string]string{
            "username": "alice",
            "role":     "admin",
        },
        "timestamp": time.Now().Format(time.RFC3339),
        "secure":    true,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func corsInfoHandler(w http.ResponseWriter, r *http.Request) {
    info := map[string]interface{}{
        "message": "CORS configuration and request information",
        "request_info": map[string]string{
            "origin":               r.Header.Get("Origin"),
            "method":               r.Method,
            "access_control_request_method": r.Header.Get("Access-Control-Request-Method"),
            "access_control_request_headers": r.Header.Get("Access-Control-Request-Headers"),
            "user_agent":           r.Header.Get("User-Agent"),
            "referer":              r.Header.Get("Referer"),
        },
        "response_headers": map[string]string{
            "access_control_allow_origin":      w.Header().Get("Access-Control-Allow-Origin"),
            "access_control_allow_methods":     w.Header().Get("Access-Control-Allow-Methods"),
            "access_control_allow_headers":     w.Header().Get("Access-Control-Allow-Headers"),
            "access_control_allow_credentials": w.Header().Get("Access-Control-Allow-Credentials"),
            "access_control_expose_headers":    w.Header().Get("Access-Control-Expose-Headers"),
            "access_control_max_age":           w.Header().Get("Access-Control-Max-Age"),
        },
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(info)
}

func corsTestPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>CORS Test Page</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .test-section { background: #f5f5f5; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #333; color: #fff; padding: 15px; border-radius: 3px; overflow-x: auto; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ccc; border-radius: 3px; }
        </style>
    </head>
    <body>
        <h1>CORS Test Page</h1>
        <p>Hello there! This page tests CORS functionality.</p>
        
        <div class="test-section">
            <h2>Simple CORS Request</h2>
            <button onclick="testSimpleRequest()">Test GET Request</button>
            <button onclick="testPostRequest()">Test POST Request</button>
            <div id="simple-result" class="result"></div>
        </div>
        
        <div class="test-section">
            <h2>Preflight CORS Request</h2>
            <button onclick="testPreflightRequest()">Test PUT with Custom Header</button>
            <div id="preflight-result" class="result"></div>
        </div>
        
        <div class="test-section">
            <h2>Credentials Request</h2>
            <button onclick="testCredentialsRequest()">Test Request with Credentials</button>
            <div id="credentials-result" class="result"></div>
        </div>
        
        <div class="test-section">
            <h2>CORS Information</h2>
            <button onclick="getCorsInfo()">Get CORS Info</button>
            <div id="info-result" class="result"></div>
        </div>
        
        <div class="test-section">
            <h2>Test Commands</h2>
            <pre>
# Simple CORS request
curl -H "Origin: https://example.com" http://localhost:8080/data

# Preflight request
curl -X OPTIONS -H "Origin: https://example.com" \
     -H "Access-Control-Request-Method: PUT" \
     -H "Access-Control-Request-Headers: Content-Type,Authorization" \
     http://localhost:8080/data

# Request with credentials
curl -H "Origin: https://example.com" \
     -H "Authorization: Bearer token123" \
     http://localhost:8080/secure-data

# Different origins to test
curl -H "Origin: https://allowed.com" http://localhost:8080/data
curl -H "Origin: https://blocked.com" http://localhost:8080/data
            </pre>
        </div>
        
        <script>
        function testSimpleRequest() {
            fetch('/data', {
                method: 'GET',
                headers: {
                    'Content-Type': 'application/json'
                }
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('simple-result').innerHTML = 
                    '<h4>Success!</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('simple-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testPostRequest() {
            fetch('/data', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({message: 'Hello from browser'})
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('simple-result').innerHTML = 
                    '<h4>POST Success!</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('simple-result').innerHTML = 
                    '<h4 style="color: red;">POST Error: ' + error + '</h4>';
            });
        }
        
        function testPreflightRequest() {
            fetch('/data', {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'X-Custom-Header': 'custom-value'
                },
                body: JSON.stringify({data: 'preflight test'})
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('preflight-result').innerHTML = 
                    '<h4>Preflight Success!</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('preflight-result').innerHTML = 
                    '<h4 style="color: red;">Preflight Error: ' + error + '</h4>';
            });
        }
        
        function testCredentialsRequest() {
            fetch('/secure-data', {
                method: 'GET',
                credentials: 'include',
                headers: {
                    'Authorization': 'Bearer demo-token'
                }
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('credentials-result').innerHTML = 
                    '<h4>Credentials Success!</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('credentials-result').innerHTML = 
                    '<h4 style="color: red;">Credentials Error: ' + error + '</h4>';
            });
        }
        
        function getCorsInfo() {
            fetch('/cors-info')
            .then(response => response.json())
            .then(data => {
                document.getElementById('info-result').innerHTML = 
                    '<h4>CORS Information</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('info-result').innerHTML = 
                    '<h4 style="color: red;">Info Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Configure CORS for public endpoints
    publicCORS := NewCORSConfig()
    publicCORS.AllowedOrigins = []string{
        "http://localhost:3000",
        "https://example.com",
        "https://*.example.com",
    }
    
    // Configure CORS for secure endpoints (with credentials)
    secureCORS := NewCORSConfig()
    secureCORS.AllowedOrigins = []string{"https://app.example.com"}
    secureCORS.AllowCredentials = true
    secureCORS.AllowedHeaders = append(secureCORS.AllowedHeaders, "Authorization")
    
    mux := http.NewServeMux()
    
    // Public endpoints
    publicHandler := CORSMiddleware(publicCORS)(http.HandlerFunc(dataHandler))
    mux.Handle("/data", publicHandler)
    
    // Secure endpoints
    secureHandler := CORSMiddleware(secureCORS)(http.HandlerFunc(secureDataHandler))
    mux.Handle("/secure-data", secureHandler)
    
    // Info endpoint
    infoHandler := CORSMiddleware(publicCORS)(http.HandlerFunc(corsInfoHandler))
    mux.Handle("/cors-info", infoHandler)
    
    // Test page
    mux.HandleFunc("/", corsTestPageHandler)
    
    fmt.Println("=== HTTP CORS Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("CORS Configuration:")
    fmt.Println("• Public endpoints: localhost:3000, *.example.com")
    fmt.Println("• Secure endpoints: app.example.com (with credentials)")
    fmt.Println("• Preflight support for complex requests")
    fmt.Println("• Custom headers and exposed headers")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - CORS test page")
    fmt.Println("  GET  /data - Public CORS-enabled data")
    fmt.Println("  GET  /secure-data - Secure CORS data (credentials required)")
    fmt.Println("  GET  /cors-info - CORS configuration info")
    
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

This CORS example demonstrates comprehensive cross-origin resource sharing  
implementation including preflight request handling, origin validation,  
credential support, and security considerations. CORS is essential for  
modern web applications that need to access APIs from different domains.

## HTTP compression and encoding

Compression reduces bandwidth usage and improves performance. This example  
demonstrates various compression techniques including gzip, content encoding,  
and custom compression strategies.

```go
package main

import (
    "compress/gzip"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type CompressionMiddleware struct {
    MinSize   int
    MimeTypes []string
}

func NewCompressionMiddleware() *CompressionMiddleware {
    return &CompressionMiddleware{
        MinSize: 1024, // Only compress responses larger than 1KB
        MimeTypes: []string{
            "text/html",
            "text/css",
            "text/javascript",
            "application/javascript",
            "application/json",
            "application/xml",
            "text/xml",
            "text/plain",
        },
    }
}

func (cm *CompressionMiddleware) shouldCompress(r *http.Request, contentType string, size int) bool {
    // Check if client accepts gzip
    acceptEncoding := r.Header.Get("Accept-Encoding")
    if !strings.Contains(acceptEncoding, "gzip") {
        return false
    }
    
    // Check minimum size
    if size < cm.MinSize {
        return false
    }
    
    // Check content type
    for _, mimeType := range cm.MimeTypes {
        if strings.Contains(contentType, mimeType) {
            return true
        }
    }
    
    return false
}

type gzipResponseWriter struct {
    http.ResponseWriter
    gzipWriter   *gzip.Writer
    written      bool
    contentType  string
    buffer       []byte
}

func newGzipResponseWriter(w http.ResponseWriter) *gzipResponseWriter {
    return &gzipResponseWriter{
        ResponseWriter: w,
        buffer:        make([]byte, 0),
    }
}

func (grw *gzipResponseWriter) Header() http.Header {
    return grw.ResponseWriter.Header()
}

func (grw *gzipResponseWriter) WriteHeader(statusCode int) {
    grw.ResponseWriter.WriteHeader(statusCode)
}

func (grw *gzipResponseWriter) Write(data []byte) (int, error) {
    if !grw.written {
        grw.contentType = grw.Header().Get("Content-Type")
        grw.buffer = append(grw.buffer, data...)
        
        // Check if we have enough data to decide on compression
        if len(grw.buffer) >= 1024 {
            grw.flushBuffer()
        }
        return len(data), nil
    }
    
    if grw.gzipWriter != nil {
        return grw.gzipWriter.Write(data)
    }
    
    return grw.ResponseWriter.Write(data)
}

func (grw *gzipResponseWriter) flushBuffer() {
    grw.written = true
    
    // Decision point: compress or not
    if grw.gzipWriter != nil || len(grw.buffer) >= 1024 {
        grw.Header().Set("Content-Encoding", "gzip")
        grw.Header().Set("Vary", "Accept-Encoding")
        grw.gzipWriter = gzip.NewWriter(grw.ResponseWriter)
        grw.gzipWriter.Write(grw.buffer)
    } else {
        grw.ResponseWriter.Write(grw.buffer)
    }
    
    grw.buffer = nil
}

func (grw *gzipResponseWriter) Close() error {
    if !grw.written && len(grw.buffer) > 0 {
        grw.flushBuffer()
    }
    
    if grw.gzipWriter != nil {
        return grw.gzipWriter.Close()
    }
    
    return nil
}

func CompressionHandler(cm *CompressionMiddleware) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            acceptEncoding := r.Header.Get("Accept-Encoding")
            
            if strings.Contains(acceptEncoding, "gzip") {
                grw := newGzipResponseWriter(w)
                defer grw.Close()
                
                // Create decision point for compression
                grw.gzipWriter = gzip.NewWriter(w)
                
                next.ServeHTTP(grw, r)
            } else {
                next.ServeHTTP(w, r)
            }
        })
    }
}

func generateLargeResponse(size int) map[string]interface{} {
    // Generate large text content
    content := strings.Repeat("Hello there! This is sample content for compression testing. ", size/64)
    
    data := map[string]interface{}{
        "message":     "Hello there! Large response for compression demo",
        "content":     content,
        "size":        len(content),
        "timestamp":   time.Now().Format(time.RFC3339),
        "compression": "This content should be compressed if client supports it",
        "metadata": map[string]interface{}{
            "user_agent":      "go-http-client",
            "compression_test": true,
            "sample_numbers":   make([]int, 100),
        },
    }
    
    // Fill sample numbers
    numbers := data["metadata"].(map[string]interface{})["sample_numbers"].([]int)
    for i := range numbers {
        numbers[i] = i * i
    }
    
    return data
}

func largeResponseHandler(w http.ResponseWriter, r *http.Request) {
    sizeParam := r.URL.Query().Get("size")
    size := 50000 // Default 50KB
    
    if sizeParam != "" {
        if parsedSize, err := strconv.Atoi(sizeParam); err == nil {
            size = parsedSize
        }
    }
    
    response := generateLargeResponse(size)
    
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("X-Original-Size", fmt.Sprintf("%d", size))
    
    json.NewEncoder(w).Encode(response)
}

func compressionInfoHandler(w http.ResponseWriter, r *http.Request) {
    acceptEncoding := r.Header.Get("Accept-Encoding")
    
    info := map[string]interface{}{
        "message":         "Compression information and capabilities",
        "accept_encoding": acceptEncoding,
        "supported_encodings": []string{"gzip", "identity"},
        "compression_enabled": strings.Contains(acceptEncoding, "gzip"),
        "response_headers": map[string]string{
            "content_encoding": w.Header().Get("Content-Encoding"),
            "vary":            w.Header().Get("Vary"),
        },
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(info)
}

func comparisonHandler(w http.ResponseWriter, r *http.Request) {
    // Generate test content
    content := strings.Repeat("Hello there! Compression test data. ", 1000)
    
    compressed := r.URL.Query().Get("compressed") == "true"
    
    if compressed {
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Content-Encoding", "gzip")
        w.Header().Set("Vary", "Accept-Encoding")
        
        gw := gzip.NewWriter(w)
        defer gw.Close()
        
        response := map[string]interface{}{
            "message":         "Compressed response",
            "content":         content,
            "original_size":   len(content),
            "compression":     "gzip",
            "timestamp":       time.Now().Format(time.RFC3339),
        }
        
        json.NewEncoder(gw).Encode(response)
    } else {
        w.Header().Set("Content-Type", "application/json")
        
        response := map[string]interface{}{
            "message":       "Uncompressed response",
            "content":       content,
            "original_size": len(content),
            "compression":   "none",
            "timestamp":     time.Now().Format(time.RFC3339),
        }
        
        json.NewEncoder(w).Encode(response)
    }
}

func streamedCompressionHandler(w http.ResponseWriter, r *http.Request) {
    acceptEncoding := r.Header.Get("Accept-Encoding")
    
    if strings.Contains(acceptEncoding, "gzip") {
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Content-Encoding", "gzip")
        w.Header().Set("Transfer-Encoding", "chunked")
        
        gw := gzip.NewWriter(w)
        defer gw.Close()
        
        encoder := json.NewEncoder(gw)
        
        // Stream JSON objects
        fmt.Fprint(gw, "[")
        
        for i := 0; i < 100; i++ {
            if i > 0 {
                fmt.Fprint(gw, ",")
            }
            
            item := map[string]interface{}{
                "id":      i + 1,
                "message": fmt.Sprintf("Hello there! Streamed item %d", i+1),
                "data":    strings.Repeat("x", 100),
                "timestamp": time.Now().Format(time.RFC3339),
            }
            
            encoder.Encode(item)
            
            if flusher, ok := gw.(http.Flusher); ok {
                flusher.Flush()
            }
            
            time.Sleep(50 * time.Millisecond)
        }
        
        fmt.Fprint(gw, "]")
    } else {
        http.Error(w, "Compression required for this endpoint", 
                  http.StatusNotAcceptable)
    }
}

func downloadHandler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    if filename == "" {
        filename = "sample.txt"
    }
    
    // Generate large text file content
    content := strings.Repeat("Hello there! This is line content for download testing.\n", 10000)
    
    acceptEncoding := r.Header.Get("Accept-Encoding")
    
    if strings.Contains(acceptEncoding, "gzip") {
        w.Header().Set("Content-Type", "application/octet-stream")
        w.Header().Set("Content-Disposition", 
                       fmt.Sprintf("attachment; filename=\"%s.gz\"", filename))
        w.Header().Set("Content-Encoding", "gzip")
        
        gw := gzip.NewWriter(w)
        defer gw.Close()
        
        gw.Write([]byte(content))
    } else {
        w.Header().Set("Content-Type", "text/plain")
        w.Header().Set("Content-Disposition", 
                       fmt.Sprintf("attachment; filename=\"%s\"", filename))
        
        fmt.Fprint(w, content)
    }
}

func compressionTestPageHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    
    // This HTML content will be compressed
    html := `<!DOCTYPE html>
<html>
<head>
    <title>HTTP Compression Demo</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; }
        .test-section { background: #f5f5f5; padding: 20px; margin: 20px 0; border-radius: 5px; }
        button { margin: 5px; padding: 10px 15px; }
        pre { background: #333; color: #fff; padding: 15px; border-radius: 3px; overflow-x: auto; }
        .result { margin: 10px 0; padding: 10px; border: 1px solid #ccc; border-radius: 3px; }
        .size-info { background: #e8f4f8; padding: 10px; border-radius: 3px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>HTTP Compression Demo</h1>
    <p>Hello there! This page demonstrates HTTP compression capabilities.</p>
    
    <div class="test-section">
        <h2>Compression Test</h2>
        <button onclick="testCompression()">Test Large Response</button>
        <button onclick="testComparison()">Compare Compressed vs Uncompressed</button>
        <button onclick="testStreamedCompression()">Test Streamed Compression</button>
        <div id="compression-result" class="result"></div>
    </div>
    
    <div class="test-section">
        <h2>File Download</h2>
        <button onclick="downloadFile()">Download Compressed File</button>
        <button onclick="downloadUncompressed()">Download Uncompressed File</button>
        <div id="download-result" class="result"></div>
    </div>
    
    <div class="test-section">
        <h2>Compression Info</h2>
        <button onclick="getCompressionInfo()">Get Compression Information</button>
        <div id="info-result" class="result"></div>
    </div>
    
    <div class="test-section">
        <h2>Browser Support</h2>
        <div id="browser-support" class="size-info">
            <h4>Current Browser Capabilities:</h4>
            <p>Accept-Encoding: <span id="accept-encoding"></span></p>
            <p>Compression Support: <span id="compression-support"></span></p>
        </div>
    </div>
    
    <div class="test-section">
        <h2>Test Commands</h2>
        <pre>
# Test with compression
curl -H "Accept-Encoding: gzip" -v http://localhost:8080/large

# Test without compression  
curl -H "Accept-Encoding: identity" -v http://localhost:8080/large

# Compare response sizes
curl -H "Accept-Encoding: gzip" -s http://localhost:8080/comparison?compressed=true | wc -c
curl -H "Accept-Encoding: identity" -s http://localhost:8080/comparison?compressed=false | wc -c

# Download compressed file
curl -H "Accept-Encoding: gzip" -o sample.txt.gz http://localhost:8080/download

# Stream with compression
curl -H "Accept-Encoding: gzip" --compressed -N http://localhost:8080/stream-compressed
        </pre>
    </div>
    
    <script>
    // Display browser capabilities
    document.addEventListener('DOMContentLoaded', function() {
        const acceptEncoding = 'gzip, deflate, br';
        document.getElementById('accept-encoding').textContent = acceptEncoding;
        
        const supportsCompression = acceptEncoding.includes('gzip');
        document.getElementById('compression-support').textContent = 
            supportsCompression ? 'Yes (gzip)' : 'No';
    });
    
    function testCompression() {
        const startTime = Date.now();
        
        fetch('/large?size=100000')
        .then(response => {
            const contentEncoding = response.headers.get('content-encoding');
            const contentLength = response.headers.get('content-length');
            const originalSize = response.headers.get('x-original-size');
            
            return response.json().then(data => ({
                data: data,
                contentEncoding: contentEncoding,
                contentLength: contentLength,
                originalSize: originalSize,
                responseTime: Date.now() - startTime
            }));
        })
        .then(result => {
            document.getElementById('compression-result').innerHTML = 
                '<h4>Compression Test Results</h4>' +
                '<div class="size-info">' +
                '<p>Content-Encoding: ' + (result.contentEncoding || 'none') + '</p>' +
                '<p>Original Size: ' + result.originalSize + ' bytes</p>' +
                '<p>Transferred Size: ' + (result.contentLength || 'chunked') + ' bytes</p>' +
                '<p>Response Time: ' + result.responseTime + 'ms</p>' +
                '</div>' +
                '<pre>' + JSON.stringify(result.data, null, 2).substring(0, 500) + '...</pre>';
        })
        .catch(error => {
            document.getElementById('compression-result').innerHTML = 
                '<h4 style="color: red;">Error: ' + error + '</h4>';
        });
    }
    
    function testComparison() {
        Promise.all([
            fetch('/comparison?compressed=true').then(r => r.json()),
            fetch('/comparison?compressed=false').then(r => r.json())
        ])
        .then(([compressed, uncompressed]) => {
            document.getElementById('compression-result').innerHTML = 
                '<h4>Compression Comparison</h4>' +
                '<div class="size-info">' +
                '<p>Compressed: ' + JSON.stringify(compressed).length + ' bytes</p>' +
                '<p>Uncompressed: ' + JSON.stringify(uncompressed).length + ' bytes</p>' +
                '<p>Compression Ratio: ' + 
                ((1 - JSON.stringify(compressed).length / JSON.stringify(uncompressed).length) * 100).toFixed(1) + '%</p>' +
                '</div>';
        });
    }
    
    function testStreamedCompression() {
        const startTime = Date.now();
        document.getElementById('compression-result').innerHTML = '<h4>Streaming compressed data...</h4>';
        
        fetch('/stream-compressed')
        .then(response => {
            const contentEncoding = response.headers.get('content-encoding');
            return response.json().then(data => ({
                data: data,
                contentEncoding: contentEncoding,
                responseTime: Date.now() - startTime
            }));
        })
        .then(result => {
            document.getElementById('compression-result').innerHTML = 
                '<h4>Streamed Compression Results</h4>' +
                '<div class="size-info">' +
                '<p>Content-Encoding: ' + (result.contentEncoding || 'none') + '</p>' +
                '<p>Items Received: ' + result.data.length + '</p>' +
                '<p>Total Response Time: ' + result.responseTime + 'ms</p>' +
                '</div>';
        })
        .catch(error => {
            document.getElementById('compression-result').innerHTML = 
                '<h4 style="color: red;">Stream Error: ' + error + '</h4>';
        });
    }
    
    function downloadFile() {
        const link = document.createElement('a');
        link.href = '/download?file=demo.txt';
        link.download = 'demo.txt';
        link.click();
        
        document.getElementById('download-result').innerHTML = 
            '<p>Compressed file download initiated...</p>';
    }
    
    function downloadUncompressed() {
        fetch('/download?file=demo.txt', {
            headers: { 'Accept-Encoding': 'identity' }
        })
        .then(response => response.blob())
        .then(blob => {
            const url = window.URL.createObjectURL(blob);
            const link = document.createElement('a');
            link.href = url;
            link.download = 'demo-uncompressed.txt';
            link.click();
            window.URL.revokeObjectURL(url);
            
            document.getElementById('download-result').innerHTML = 
                '<p>Uncompressed file download completed. Size: ' + blob.size + ' bytes</p>';
        });
    }
    
    function getCompressionInfo() {
        fetch('/compression-info')
        .then(response => response.json())
        .then(data => {
            document.getElementById('info-result').innerHTML = 
                '<h4>Compression Information</h4>' +
                '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
        });
    }
    </script>
</body>
</html>`
    
    fmt.Fprint(w, html)
}

func main() {
    compression := NewCompressionMiddleware()
    
    mux := http.NewServeMux()
    
    // Apply compression middleware to selected endpoints
    compressedHandler := CompressionHandler(compression)
    
    mux.Handle("/", compressedHandler(http.HandlerFunc(compressionTestPageHandler)))
    mux.Handle("/large", compressedHandler(http.HandlerFunc(largeResponseHandler)))
    mux.Handle("/comparison", http.HandlerFunc(comparisonHandler))
    mux.Handle("/stream-compressed", http.HandlerFunc(streamedCompressionHandler))
    mux.Handle("/download", http.HandlerFunc(downloadHandler))
    mux.Handle("/compression-info", compressedHandler(http.HandlerFunc(compressionInfoHandler)))
    
    fmt.Println("=== HTTP Compression Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Gzip compression middleware")
    fmt.Println("• Content-type based compression")
    fmt.Println("• Size-threshold compression")
    fmt.Println("• Streamed compression")
    fmt.Println("• File download compression")
    fmt.Println("• Compression comparison tools")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Compression test page")
    fmt.Println("  GET /large?size=N - Large response (compressed)")
    fmt.Println("  GET /comparison - Compare compressed vs uncompressed")
    fmt.Println("  GET /stream-compressed - Streamed compression")
    fmt.Println("  GET /download - Compressed file download")
    fmt.Println("  GET /compression-info - Compression information")
    
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

This compression example demonstrates comprehensive HTTP compression  
techniques including gzip encoding, content-type filtering, size thresholds,  
and streaming compression. Proper compression implementation significantly  
improves application performance and reduces bandwidth usage.

## HTTP request and response transformation

Request and response transformation allows modification of HTTP data as it  
flows through the application. This example demonstrates interceptors,  
data transformation, and request/response manipulation.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type TransformationRule struct {
    Path        string
    Method      string
    Transform   func(*http.Request, *ResponseCapture) error
}

type ResponseCapture struct {
    StatusCode int
    Headers    http.Header
    Body       []byte
    Modified   bool
}

type RequestTransformer struct {
    rules []TransformationRule
}

func NewRequestTransformer() *RequestTransformer {
    return &RequestTransformer{
        rules: make([]TransformationRule, 0),
    }
}

func (rt *RequestTransformer) AddRule(path, method string, 
                                     transform func(*http.Request, *ResponseCapture) error) {
    rt.rules = append(rt.rules, TransformationRule{
        Path:      path,
        Method:    method,
        Transform: transform,
    })
}

func (rt *RequestTransformer) Apply(r *http.Request, rc *ResponseCapture) error {
    for _, rule := range rt.rules {
        if (rule.Path == "*" || strings.Contains(r.URL.Path, rule.Path)) &&
           (rule.Method == "*" || rule.Method == r.Method) {
            if err := rule.Transform(r, rc); err != nil {
                return err
            }
        }
    }
    return nil
}

type responseWriter struct {
    http.ResponseWriter
    statusCode int
    body       *bytes.Buffer
    headers    http.Header
}

func newResponseWriter(w http.ResponseWriter) *responseWriter {
    return &responseWriter{
        ResponseWriter: w,
        statusCode:     200,
        body:          new(bytes.Buffer),
        headers:       make(http.Header),
    }
}

func (rw *responseWriter) WriteHeader(statusCode int) {
    rw.statusCode = statusCode
    // Don't write header yet, we need to transform first
}

func (rw *responseWriter) Write(data []byte) (int, error) {
    return rw.body.Write(data)
}

func (rw *responseWriter) Header() http.Header {
    // Merge headers
    for k, v := range rw.ResponseWriter.Header() {
        rw.headers[k] = v
    }
    return rw.headers
}

func TransformationMiddleware(transformer *RequestTransformer) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Capture the response
            rw := newResponseWriter(w)
            
            // Execute the handler
            next.ServeHTTP(rw, r)
            
            // Create response capture
            rc := &ResponseCapture{
                StatusCode: rw.statusCode,
                Headers:    rw.headers,
                Body:       rw.body.Bytes(),
                Modified:   false,
            }
            
            // Apply transformations
            if err := transformer.Apply(r, rc); err != nil {
                log.Printf("Transformation error: %v", err)
                http.Error(w, "Transformation failed", http.StatusInternalServerError)
                return
            }
            
            // Write the final response
            for k, v := range rc.Headers {
                w.Header()[k] = v
            }
            
            if rc.Modified {
                w.Header().Set("X-Transformed", "true")
                w.Header().Set("Content-Length", strconv.Itoa(len(rc.Body)))
            }
            
            w.WriteHeader(rc.StatusCode)
            w.Write(rc.Body)
        })
    }
}

// Transformation functions
func AddTimestampTransform(r *http.Request, rc *ResponseCapture) error {
    if strings.Contains(rc.Headers.Get("Content-Type"), "application/json") {
        var data map[string]interface{}
        if err := json.Unmarshal(rc.Body, &data); err != nil {
            return err
        }
        
        data["transformed_at"] = time.Now().Format(time.RFC3339)
        data["transform_type"] = "timestamp_added"
        
        newBody, err := json.Marshal(data)
        if err != nil {
            return err
        }
        
        rc.Body = newBody
        rc.Modified = true
    }
    return nil
}

func AddRequestInfoTransform(r *http.Request, rc *ResponseCapture) error {
    if strings.Contains(rc.Headers.Get("Content-Type"), "application/json") {
        var data map[string]interface{}
        if err := json.Unmarshal(rc.Body, &data); err != nil {
            return err
        }
        
        data["request_info"] = map[string]interface{}{
            "method":      r.Method,
            "path":        r.URL.Path,
            "user_agent":  r.Header.Get("User-Agent"),
            "remote_addr": r.RemoteAddr,
            "query_params": r.URL.Query(),
        }
        
        newBody, err := json.Marshal(data)
        if err != nil {
            return err
        }
        
        rc.Body = newBody
        rc.Modified = true
    }
    return nil
}

func FilterSensitiveDataTransform(r *http.Request, rc *ResponseCapture) error {
    if strings.Contains(rc.Headers.Get("Content-Type"), "application/json") {
        var data map[string]interface{}
        if err := json.Unmarshal(rc.Body, &data); err != nil {
            return err
        }
        
        // Remove sensitive fields
        sensitiveFields := []string{"password", "secret", "token", "key"}
        
        modified := false
        var filterRecursive func(obj interface{}) interface{}
        filterRecursive = func(obj interface{}) interface{} {
            switch v := obj.(type) {
            case map[string]interface{}:
                filtered := make(map[string]interface{})
                for key, value := range v {
                    lowerKey := strings.ToLower(key)
                    isSensitive := false
                    
                    for _, sensitive := range sensitiveFields {
                        if strings.Contains(lowerKey, sensitive) {
                            isSensitive = true
                            break
                        }
                    }
                    
                    if isSensitive {
                        filtered[key] = "[FILTERED]"
                        modified = true
                    } else {
                        filtered[key] = filterRecursive(value)
                    }
                }
                return filtered
            case []interface{}:
                filtered := make([]interface{}, len(v))
                for i, item := range v {
                    filtered[i] = filterRecursive(item)
                }
                return filtered
            default:
                return v
            }
        }
        
        filteredData := filterRecursive(data)
        
        if modified {
            newBody, err := json.Marshal(filteredData)
            if err != nil {
                return err
            }
            
            rc.Body = newBody
            rc.Modified = true
            rc.Headers.Set("X-Data-Filtered", "true")
        }
    }
    return nil
}

func WrapResponseTransform(r *http.Request, rc *ResponseCapture) error {
    if strings.Contains(rc.Headers.Get("Content-Type"), "application/json") {
        var originalData interface{}
        if err := json.Unmarshal(rc.Body, &originalData); err != nil {
            return err
        }
        
        wrappedData := map[string]interface{}{
            "success": rc.StatusCode >= 200 && rc.StatusCode < 300,
            "status_code": rc.StatusCode,
            "data": originalData,
            "metadata": map[string]interface{}{
                "timestamp": time.Now().Format(time.RFC3339),
                "version": "1.0",
                "endpoint": r.URL.Path,
            },
        }
        
        newBody, err := json.Marshal(wrappedData)
        if err != nil {
            return err
        }
        
        rc.Body = newBody
        rc.Modified = true
    }
    return nil
}

func PaginationTransform(r *http.Request, rc *ResponseCapture) error {
    if strings.Contains(rc.Headers.Get("Content-Type"), "application/json") {
        page := r.URL.Query().Get("page")
        limit := r.URL.Query().Get("limit")
        
        if page != "" && limit != "" {
            var data map[string]interface{}
            if err := json.Unmarshal(rc.Body, &data); err != nil {
                return err
            }
            
            pageNum, _ := strconv.Atoi(page)
            limitNum, _ := strconv.Atoi(limit)
            
            if pageNum < 1 {
                pageNum = 1
            }
            if limitNum < 1 {
                limitNum = 10
            }
            
            // Add pagination metadata
            data["pagination"] = map[string]interface{}{
                "page": pageNum,
                "limit": limitNum,
                "has_next": pageNum < 5, // Simulate
                "has_prev": pageNum > 1,
                "total_pages": 5,
                "total_items": 50,
            }
            
            newBody, err := json.Marshal(data)
            if err != nil {
                return err
            }
            
            rc.Body = newBody
            rc.Modified = true
        }
    }
    return nil
}

// Sample handlers
func usersHandler(w http.ResponseWriter, r *http.Request) {
    users := []map[string]interface{}{
        {
            "id": 1,
            "name": "Alice Johnson",
            "email": "alice@example.com",
            "password": "secret123",
            "api_key": "ak_123456789",
            "role": "admin",
        },
        {
            "id": 2,
            "name": "Bob Smith", 
            "email": "bob@example.com",
            "password": "password456",
            "api_key": "ak_987654321",
            "role": "user",
        },
    }
    
    response := map[string]interface{}{
        "message": "Hello there! User data retrieved",
        "users": users,
        "count": len(users),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func profileHandler(w http.ResponseWriter, r *http.Request) {
    profile := map[string]interface{}{
        "id": 123,
        "username": "alice_dev",
        "email": "alice@example.com",
        "password": "super_secret_password",
        "secret_token": "tok_abcdef123456",
        "preferences": map[string]interface{}{
            "theme": "dark",
            "language": "en",
            "notifications": true,
        },
        "created_at": "2023-01-15T10:30:00Z",
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(profile)
}

func dataHandler(w http.ResponseWriter, r *http.Request) {
    data := map[string]interface{}{
        "message": "Hello there! Raw data response",
        "items": []string{"item1", "item2", "item3"},
        "generated_at": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(data)
}

func transformationDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Transformation Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 40px; }
            .demo-section { background: #f5f5f5; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #333; color: #fff; padding: 15px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ccc; border-radius: 3px; }
        </style>
    </head>
    <body>
        <h1>HTTP Transformation Demo</h1>
        <p>Hello there! This demonstrates request/response transformation.</p>
        
        <div class="demo-section">
            <h2>Data Filtering Transformation</h2>
            <p>Removes sensitive fields from responses:</p>
            <button onclick="testFiltering()">Test Data Filtering</button>
            <div id="filtering-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Response Wrapping</h2>
            <p>Wraps responses in standard format:</p>
            <button onclick="testWrapping()">Test Response Wrapping</button>
            <div id="wrapping-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Request Info Addition</h2>
            <p>Adds request metadata to responses:</p>
            <button onclick="testRequestInfo()">Test Request Info</button>
            <div id="request-info-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Pagination Transformation</h2>
            <p>Adds pagination metadata:</p>
            <button onclick="testPagination()">Test Pagination</button>
            <div id="pagination-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Multiple Transformations</h2>
            <p>Combines multiple transformations:</p>
            <button onclick="testMultiple()">Test Multiple Transforms</button>
            <div id="multiple-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Test Commands</h2>
            <pre>
# Test data filtering
curl http://localhost:8080/profile

# Test response wrapping
curl http://localhost:8080/wrapped/data

# Test request info addition  
curl -H "User-Agent: Custom-Client/1.0" http://localhost:8080/with-info/data

# Test pagination
curl "http://localhost:8080/paginated/users?page=2&limit=5"

# Compare original vs transformed
curl http://localhost:8080/users
curl http://localhost:8080/filtered/users
            </pre>
        </div>
        
        <script>
        function testFiltering() {
            fetch('/profile')
            .then(response => response.json())
            .then(data => {
                const isTransformed = data.password === '[FILTERED]';
                document.getElementById('filtering-result').innerHTML = 
                    '<h4>Data Filtering Result</h4>' +
                    '<p>Sensitive data filtered: ' + isTransformed + '</p>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('filtering-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testWrapping() {
            fetch('/wrapped/data')
            .then(response => response.json())
            .then(data => {
                document.getElementById('wrapping-result').innerHTML = 
                    '<h4>Response Wrapping Result</h4>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('wrapping-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testRequestInfo() {
            fetch('/with-info/data', {
                headers: {
                    'User-Agent': 'Demo-Client/1.0',
                    'X-Custom-Header': 'test-value'
                }
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('request-info-result').innerHTML = 
                    '<h4>Request Info Addition Result</h4>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('request-info-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testPagination() {
            fetch('/paginated/users?page=2&limit=3')
            .then(response => response.json())
            .then(data => {
                document.getElementById('pagination-result').innerHTML = 
                    '<h4>Pagination Transformation Result</h4>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('pagination-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testMultiple() {
            fetch('/multi/profile?page=1&limit=10')
            .then(response => response.json())
            .then(data => {
                document.getElementById('multiple-result').innerHTML = 
                    '<h4>Multiple Transformations Result</h4>' +
                    '<p>Applied: Filtering + Wrapping + Pagination + Request Info</p>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('multiple-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Create transformers for different routes
    
    // Basic transformer (adds timestamps)
    basicTransformer := NewRequestTransformer()
    basicTransformer.AddRule("*", "*", AddTimestampTransform)
    
    // Filtering transformer
    filterTransformer := NewRequestTransformer()
    filterTransformer.AddRule("*", "*", FilterSensitiveDataTransform)
    
    // Wrapping transformer
    wrapTransformer := NewRequestTransformer()
    wrapTransformer.AddRule("*", "*", WrapResponseTransform)
    
    // Request info transformer
    requestInfoTransformer := NewRequestTransformer()
    requestInfoTransformer.AddRule("*", "*", AddRequestInfoTransform)
    
    // Pagination transformer
    paginationTransformer := NewRequestTransformer()
    paginationTransformer.AddRule("*", "*", PaginationTransform)
    
    // Multi-transformer (combines multiple)
    multiTransformer := NewRequestTransformer()
    multiTransformer.AddRule("*", "*", FilterSensitiveDataTransform)
    multiTransformer.AddRule("*", "*", WrapResponseTransform)
    multiTransformer.AddRule("*", "*", AddRequestInfoTransform)
    multiTransformer.AddRule("*", "*", PaginationTransform)
    
    mux := http.NewServeMux()
    
    // Demo page
    mux.HandleFunc("/", transformationDemoHandler)
    
    // Original endpoints (minimal transformation)
    mux.Handle("/users", TransformationMiddleware(basicTransformer)(http.HandlerFunc(usersHandler)))
    mux.Handle("/profile", TransformationMiddleware(filterTransformer)(http.HandlerFunc(profileHandler)))
    mux.Handle("/data", TransformationMiddleware(basicTransformer)(http.HandlerFunc(dataHandler)))
    
    // Transformed endpoints
    mux.Handle("/filtered/", http.StripPrefix("/filtered", 
        TransformationMiddleware(filterTransformer)(http.HandlerFunc(usersHandler))))
    
    mux.Handle("/wrapped/", http.StripPrefix("/wrapped", 
        TransformationMiddleware(wrapTransformer)(http.HandlerFunc(dataHandler))))
    
    mux.Handle("/with-info/", http.StripPrefix("/with-info", 
        TransformationMiddleware(requestInfoTransformer)(http.HandlerFunc(dataHandler))))
    
    mux.Handle("/paginated/", http.StripPrefix("/paginated", 
        TransformationMiddleware(paginationTransformer)(http.HandlerFunc(usersHandler))))
    
    mux.Handle("/multi/", http.StripPrefix("/multi", 
        TransformationMiddleware(multiTransformer)(http.HandlerFunc(profileHandler))))
    
    fmt.Println("=== HTTP Transformation Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Request/Response transformation")
    fmt.Println("• Sensitive data filtering")
    fmt.Println("• Response wrapping and metadata")
    fmt.Println("• Request information injection")
    fmt.Println("• Pagination transformation")
    fmt.Println("• Multiple transformation chaining")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Transformation demo page")
    fmt.Println("  GET /users - Basic transformation")
    fmt.Println("  GET /profile - Filtered sensitive data")
    fmt.Println("  GET /wrapped/* - Response wrapping")
    fmt.Println("  GET /with-info/* - Request info addition")
    fmt.Println("  GET /paginated/* - Pagination metadata")
    fmt.Println("  GET /multi/* - Multiple transformations")
    
    log.Fatal(http.ListenAndServe(":8080", mux))
}
```

This transformation example demonstrates sophisticated request and response  
manipulation techniques including data filtering, response wrapping, metadata  
injection, and transformation chaining. These patterns are essential for  
building flexible APIs that can adapt data format and content based on  
different requirements.

## HTTP performance monitoring and metrics

Performance monitoring provides insights into application behavior and helps  
identify bottlenecks. This example demonstrates comprehensive metrics  
collection, performance tracking, and monitoring endpoints.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "runtime"
    "sort"
    "strconv"
    "sync"
    "sync/atomic"
    "time"
)

type Metrics struct {
    RequestCount     int64             `json:"request_count"`
    ErrorCount       int64             `json:"error_count"`
    TotalDuration    int64             `json:"total_duration_ns"`
    ActiveRequests   int32             `json:"active_requests"`
    EndpointMetrics  map[string]*EndpointMetrics `json:"endpoint_metrics"`
    ResponseSizes    map[string]int64  `json:"response_sizes"`
    StatusCodes      map[int]int64     `json:"status_codes"`
    StartTime        time.Time         `json:"start_time"`
    mu               sync.RWMutex
}

type EndpointMetrics struct {
    Count        int64         `json:"count"`
    TotalTime    int64         `json:"total_time_ns"`
    MinTime      int64         `json:"min_time_ns"`
    MaxTime      int64         `json:"max_time_ns"`
    AvgTime      float64       `json:"avg_time_ns"`
    ErrorCount   int64         `json:"error_count"`
    LastAccessed time.Time     `json:"last_accessed"`
    ResponseTimes []int64      `json:"-"` // For percentile calculation
}

type PerformanceMonitor struct {
    metrics *Metrics
}

func NewPerformanceMonitor() *PerformanceMonitor {
    return &PerformanceMonitor{
        metrics: &Metrics{
            EndpointMetrics: make(map[string]*EndpointMetrics),
            ResponseSizes:   make(map[string]int64),
            StatusCodes:     make(map[int]int64),
            StartTime:       time.Now(),
        },
    }
}

func (pm *PerformanceMonitor) getOrCreateEndpointMetrics(endpoint string) *EndpointMetrics {
    pm.metrics.mu.Lock()
    defer pm.metrics.mu.Unlock()
    
    if em, exists := pm.metrics.EndpointMetrics[endpoint]; exists {
        return em
    }
    
    em := &EndpointMetrics{
        MinTime:       int64(^uint64(0) >> 1), // Max int64
        ResponseTimes: make([]int64, 0, 1000),
    }
    pm.metrics.EndpointMetrics[endpoint] = em
    return em
}

func (pm *PerformanceMonitor) recordRequest(endpoint string, duration time.Duration, 
                                           statusCode int, responseSize int64) {
    durationNs := duration.Nanoseconds()
    
    // Update global metrics
    atomic.AddInt64(&pm.metrics.RequestCount, 1)
    atomic.AddInt64(&pm.metrics.TotalDuration, durationNs)
    
    if statusCode >= 400 {
        atomic.AddInt64(&pm.metrics.ErrorCount, 1)
    }
    
    // Update endpoint metrics
    em := pm.getOrCreateEndpointMetrics(endpoint)
    
    em.Count++
    em.TotalTime += durationNs
    em.LastAccessed = time.Now()
    
    if statusCode >= 400 {
        em.ErrorCount++
    }
    
    if durationNs < em.MinTime {
        em.MinTime = durationNs
    }
    if durationNs > em.MaxTime {
        em.MaxTime = durationNs
    }
    
    em.AvgTime = float64(em.TotalTime) / float64(em.Count)
    
    // Store response time for percentile calculation (limit to last 1000)
    if len(em.ResponseTimes) >= 1000 {
        em.ResponseTimes = em.ResponseTimes[1:]
    }
    em.ResponseTimes = append(em.ResponseTimes, durationNs)
    
    // Update status code metrics
    pm.metrics.mu.Lock()
    pm.metrics.StatusCodes[statusCode]++
    
    // Update response size metrics
    sizeCategory := categorizeResponseSize(responseSize)
    pm.metrics.ResponseSizes[sizeCategory]++
    pm.metrics.mu.Unlock()
}

func categorizeResponseSize(size int64) string {
    switch {
    case size < 1024:
        return "< 1KB"
    case size < 10*1024:
        return "1KB - 10KB"
    case size < 100*1024:
        return "10KB - 100KB"
    case size < 1024*1024:
        return "100KB - 1MB"
    default:
        return "> 1MB"
    }
}

func (pm *PerformanceMonitor) calculatePercentile(times []int64, percentile float64) int64 {
    if len(times) == 0 {
        return 0
    }
    
    sorted := make([]int64, len(times))
    copy(sorted, times)
    sort.Slice(sorted, func(i, j int) bool { return sorted[i] < sorted[j] })
    
    index := int(float64(len(sorted)) * percentile / 100.0)
    if index >= len(sorted) {
        index = len(sorted) - 1
    }
    
    return sorted[index]
}

type responseWriterWithSize struct {
    http.ResponseWriter
    statusCode   int
    responseSize int64
}

func (rws *responseWriterWithSize) WriteHeader(statusCode int) {
    rws.statusCode = statusCode
    rws.ResponseWriter.WriteHeader(statusCode)
}

func (rws *responseWriterWithSize) Write(data []byte) (int, error) {
    rws.responseSize += int64(len(data))
    return rws.ResponseWriter.Write(data)
}

func MonitoringMiddleware(pm *PerformanceMonitor) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            atomic.AddInt32(&pm.metrics.ActiveRequests, 1)
            defer atomic.AddInt32(&pm.metrics.ActiveRequests, -1)
            
            // Wrap the response writer to capture status code and size
            rws := &responseWriterWithSize{
                ResponseWriter: w,
                statusCode:    200,
            }
            
            // Execute the handler
            next.ServeHTTP(rws, r)
            
            // Record metrics
            duration := time.Since(start)
            endpoint := r.Method + " " + r.URL.Path
            
            pm.recordRequest(endpoint, duration, rws.statusCode, rws.responseSize)
        })
    }
}

func (pm *PerformanceMonitor) getMetricsSnapshot() map[string]interface{} {
    pm.metrics.mu.RLock()
    defer pm.metrics.mu.RUnlock()
    
    uptime := time.Since(pm.metrics.StartTime)
    requestCount := atomic.LoadInt64(&pm.metrics.RequestCount)
    totalDuration := atomic.LoadInt64(&pm.metrics.TotalDuration)
    
    var avgResponseTime float64
    if requestCount > 0 {
        avgResponseTime = float64(totalDuration) / float64(requestCount) / 1e6 // Convert to ms
    }
    
    // Calculate endpoint percentiles
    endpointDetails := make(map[string]interface{})
    for endpoint, em := range pm.metrics.EndpointMetrics {
        p50 := pm.calculatePercentile(em.ResponseTimes, 50)
        p95 := pm.calculatePercentile(em.ResponseTimes, 95)
        p99 := pm.calculatePercentile(em.ResponseTimes, 99)
        
        endpointDetails[endpoint] = map[string]interface{}{
            "count":         em.Count,
            "avg_time_ms":   em.AvgTime / 1e6,
            "min_time_ms":   float64(em.MinTime) / 1e6,
            "max_time_ms":   float64(em.MaxTime) / 1e6,
            "p50_ms":        float64(p50) / 1e6,
            "p95_ms":        float64(p95) / 1e6,
            "p99_ms":        float64(p99) / 1e6,
            "error_count":   em.ErrorCount,
            "error_rate":    float64(em.ErrorCount) / float64(em.Count) * 100,
            "last_accessed": em.LastAccessed.Format(time.RFC3339),
        }
    }
    
    // Get memory stats
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    return map[string]interface{}{
        "server_info": map[string]interface{}{
            "uptime_seconds":      uptime.Seconds(),
            "start_time":          pm.metrics.StartTime.Format(time.RFC3339),
            "active_requests":     atomic.LoadInt32(&pm.metrics.ActiveRequests),
            "total_requests":      requestCount,
            "total_errors":        atomic.LoadInt64(&pm.metrics.ErrorCount),
            "error_rate_percent":  float64(atomic.LoadInt64(&pm.metrics.ErrorCount)) / float64(requestCount) * 100,
            "avg_response_time_ms": avgResponseTime,
            "requests_per_second": float64(requestCount) / uptime.Seconds(),
        },
        "memory_stats": map[string]interface{}{
            "alloc_mb":         float64(m.Alloc) / 1024 / 1024,
            "total_alloc_mb":   float64(m.TotalAlloc) / 1024 / 1024,
            "sys_mb":           float64(m.Sys) / 1024 / 1024,
            "num_gc":           m.NumGC,
            "gc_cpu_fraction":  m.GCCPUFraction,
        },
        "runtime_stats": map[string]interface{}{
            "goroutines":    runtime.NumGoroutine(),
            "cgo_calls":     runtime.NumCgoCall(),
            "go_version":    runtime.Version(),
            "num_cpu":       runtime.NumCPU(),
        },
        "endpoint_metrics": endpointDetails,
        "status_codes":     pm.metrics.StatusCodes,
        "response_sizes":   pm.metrics.ResponseSizes,
        "timestamp":        time.Now().Format(time.RFC3339),
    }
}

// Sample handlers for testing
func fastHandler(w http.ResponseWriter, r *http.Request) {
    response := map[string]interface{}{
        "message": "Hello there! Fast response",
        "data":    "Quick operation completed",
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate slow operation
    delay := 100 + (time.Now().UnixNano() % 400) // 100-500ms
    time.Sleep(time.Duration(delay) * time.Millisecond)
    
    response := map[string]interface{}{
        "message": "Hello there! Slow response",
        "data":    "Long operation completed",
        "delay_ms": delay,
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func errorHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate random errors
    if time.Now().UnixNano()%3 == 0 {
        http.Error(w, "Simulated server error", http.StatusInternalServerError)
        return
    }
    
    if time.Now().UnixNano()%4 == 0 {
        http.Error(w, "Simulated bad request", http.StatusBadRequest)
        return
    }
    
    response := map[string]interface{}{
        "message": "Hello there! Success response",
        "data":    "Operation completed successfully",
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func largeResponseHandler(w http.ResponseWriter, r *http.Request) {
    size := 1000
    if sizeParam := r.URL.Query().Get("size"); sizeParam != "" {
        if s, err := strconv.Atoi(sizeParam); err == nil {
            size = s
        }
    }
    
    // Generate large response
    data := make([]map[string]interface{}, size)
    for i := 0; i < size; i++ {
        data[i] = map[string]interface{}{
            "id":      i + 1,
            "message": fmt.Sprintf("Hello there! Item %d", i+1),
            "data":    fmt.Sprintf("Sample data for item %d", i+1),
        }
    }
    
    response := map[string]interface{}{
        "message": "Hello there! Large response generated",
        "items":   data,
        "count":   size,
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func metricsHandler(pm *PerformanceMonitor) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        metrics := pm.getMetricsSnapshot()
        
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Cache-Control", "no-cache")
        
        json.NewEncoder(w).Encode(metrics)
    }
}

func healthHandler(pm *PerformanceMonitor) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        metrics := pm.getMetricsSnapshot()
        serverInfo := metrics["server_info"].(map[string]interface{})
        
        errorRate := serverInfo["error_rate_percent"].(float64)
        avgResponseTime := serverInfo["avg_response_time_ms"].(float64)
        activeRequests := serverInfo["active_requests"].(int32)
        
        healthy := errorRate < 10.0 && avgResponseTime < 1000.0 && activeRequests < 100
        
        status := "healthy"
        if !healthy {
            status = "unhealthy"
            w.WriteHeader(http.StatusServiceUnavailable)
        }
        
        health := map[string]interface{}{
            "status": status,
            "checks": map[string]interface{}{
                "error_rate_ok": map[string]interface{}{
                    "status": errorRate < 10.0,
                    "value":  errorRate,
                    "threshold": 10.0,
                },
                "response_time_ok": map[string]interface{}{
                    "status": avgResponseTime < 1000.0,
                    "value":  avgResponseTime,
                    "threshold": 1000.0,
                },
                "load_ok": map[string]interface{}{
                    "status": activeRequests < 100,
                    "value":  activeRequests,
                    "threshold": 100,
                },
            },
            "timestamp": time.Now().Format(time.RFC3339),
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(health)
    }
}

func monitoringDashboardHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Performance Monitoring Dashboard</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .dashboard { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
            .metric-card { background: #f5f5f5; padding: 20px; border-radius: 8px; border: 1px solid #ddd; }
            .metric-value { font-size: 2em; font-weight: bold; color: #007cba; }
            .metric-label { color: #666; margin-bottom: 10px; }
            .health-good { color: #28a745; }
            .health-warning { color: #ffc107; }
            .health-bad { color: #dc3545; }
            table { width: 100%; border-collapse: collapse; margin-top: 10px; }
            th, td { padding: 8px; text-align: left; border-bottom: 1px solid #ddd; }
            th { background: #f8f9fa; }
            button { margin: 5px; padding: 10px 15px; }
            .auto-refresh { background: #e8f4f8; padding: 10px; border-radius: 5px; margin-bottom: 20px; }
        </style>
    </head>
    <body>
        <h1>Performance Monitoring Dashboard</h1>
        <p>Hello there! Real-time application performance metrics.</p>
        
        <div class="auto-refresh">
            <label><input type="checkbox" id="auto-refresh" checked> Auto-refresh every 5 seconds</label>
            <button onclick="refreshMetrics()">Refresh Now</button>
        </div>
        
        <div class="dashboard">
            <div class="metric-card">
                <div class="metric-label">Total Requests</div>
                <div class="metric-value" id="total-requests">-</div>
            </div>
            
            <div class="metric-card">
                <div class="metric-label">Error Rate</div>
                <div class="metric-value" id="error-rate">-</div>
            </div>
            
            <div class="metric-card">
                <div class="metric-label">Avg Response Time</div>
                <div class="metric-value" id="avg-response-time">-</div>
            </div>
            
            <div class="metric-card">
                <div class="metric-label">Active Requests</div>
                <div class="metric-value" id="active-requests">-</div>
            </div>
            
            <div class="metric-card">
                <div class="metric-label">Requests/Second</div>
                <div class="metric-value" id="requests-per-second">-</div>
            </div>
            
            <div class="metric-card">
                <div class="metric-label">Memory Usage</div>
                <div class="metric-value" id="memory-usage">-</div>
            </div>
        </div>
        
        <div style="margin-top: 30px;">
            <h2>Endpoint Performance</h2>
            <table id="endpoint-table">
                <thead>
                    <tr>
                        <th>Endpoint</th>
                        <th>Count</th>
                        <th>Avg (ms)</th>
                        <th>P95 (ms)</th>
                        <th>P99 (ms)</th>
                        <th>Error Rate</th>
                    </tr>
                </thead>
                <tbody id="endpoint-tbody">
                </tbody>
            </table>
        </div>
        
        <div style="margin-top: 30px;">
            <h2>Load Testing</h2>
            <button onclick="generateLoad()">Generate Load</button>
            <button onclick="generateErrors()">Generate Errors</button>
            <button onclick="generateSlowRequests()">Generate Slow Requests</button>
            <div id="load-result" style="margin-top: 10px;"></div>
        </div>
        
        <script>
        let refreshInterval;
        
        function refreshMetrics() {
            fetch('/metrics')
            .then(response => response.json())
            .then(data => {
                const serverInfo = data.server_info;
                const memoryStats = data.memory_stats;
                
                document.getElementById('total-requests').textContent = 
                    serverInfo.total_requests.toLocaleString();
                
                const errorRate = serverInfo.error_rate_percent;
                const errorRateEl = document.getElementById('error-rate');
                errorRateEl.textContent = errorRate.toFixed(2) + '%';
                errorRateEl.className = 'metric-value ' + 
                    (errorRate < 5 ? 'health-good' : errorRate < 10 ? 'health-warning' : 'health-bad');
                
                document.getElementById('avg-response-time').textContent = 
                    serverInfo.avg_response_time_ms.toFixed(1) + ' ms';
                
                document.getElementById('active-requests').textContent = 
                    serverInfo.active_requests;
                
                document.getElementById('requests-per-second').textContent = 
                    serverInfo.requests_per_second.toFixed(1);
                
                document.getElementById('memory-usage').textContent = 
                    memoryStats.alloc_mb.toFixed(1) + ' MB';
                
                // Update endpoint table
                const tbody = document.getElementById('endpoint-tbody');
                tbody.innerHTML = '';
                
                for (const [endpoint, metrics] of Object.entries(data.endpoint_metrics)) {
                    const row = tbody.insertRow();
                    row.insertCell(0).textContent = endpoint;
                    row.insertCell(1).textContent = metrics.count;
                    row.insertCell(2).textContent = metrics.avg_time_ms.toFixed(1);
                    row.insertCell(3).textContent = metrics.p95_ms.toFixed(1);
                    row.insertCell(4).textContent = metrics.p99_ms.toFixed(1);
                    
                    const errorCell = row.insertCell(5);
                    errorCell.textContent = metrics.error_rate.toFixed(1) + '%';
                    errorCell.className = metrics.error_rate < 5 ? 'health-good' : 
                                         metrics.error_rate < 10 ? 'health-warning' : 'health-bad';
                }
            })
            .catch(error => {
                console.error('Error fetching metrics:', error);
            });
        }
        
        function generateLoad() {
            document.getElementById('load-result').innerHTML = 'Generating load...';
            
            const promises = [];
            for (let i = 0; i < 50; i++) {
                promises.push(fetch('/fast'));
                promises.push(fetch('/slow'));
                promises.push(fetch('/error'));
            }
            
            Promise.allSettled(promises).then(() => {
                document.getElementById('load-result').innerHTML = 
                    'Load generation completed (150 requests)';
                setTimeout(refreshMetrics, 1000);
            });
        }
        
        function generateErrors() {
            const promises = [];
            for (let i = 0; i < 20; i++) {
                promises.push(fetch('/error'));
            }
            
            Promise.allSettled(promises).then(() => {
                document.getElementById('load-result').innerHTML = 
                    'Error generation completed (20 requests)';
                setTimeout(refreshMetrics, 1000);
            });
        }
        
        function generateSlowRequests() {
            const promises = [];
            for (let i = 0; i < 10; i++) {
                promises.push(fetch('/slow'));
                promises.push(fetch('/large?size=5000'));
            }
            
            Promise.allSettled(promises).then(() => {
                document.getElementById('load-result').innerHTML = 
                    'Slow request generation completed (20 requests)';
                setTimeout(refreshMetrics, 1000);
            });
        }
        
        // Auto-refresh functionality
        document.getElementById('auto-refresh').addEventListener('change', function() {
            if (this.checked) {
                refreshInterval = setInterval(refreshMetrics, 5000);
            } else {
                clearInterval(refreshInterval);
            }
        });
        
        // Initial load and setup auto-refresh
        refreshMetrics();
        refreshInterval = setInterval(refreshMetrics, 5000);
        </script>
    </body>
    </html>`)
}

func main() {
    monitor := NewPerformanceMonitor()
    
    mux := http.NewServeMux()
    
    // Monitoring middleware applied to all handlers
    monitoredMux := MonitoringMiddleware(monitor)(mux)
    
    // Demo page
    mux.HandleFunc("/", monitoringDashboardHandler)
    
    // Test endpoints
    mux.HandleFunc("/fast", fastHandler)
    mux.HandleFunc("/slow", slowHandler)
    mux.HandleFunc("/error", errorHandler)
    mux.HandleFunc("/large", largeResponseHandler)
    
    // Monitoring endpoints (not monitored to avoid recursion)
    http.HandleFunc("/metrics", metricsHandler(monitor))
    http.HandleFunc("/health", healthHandler(monitor))
    
    // Main application routes (monitored)
    http.Handle("/", monitoredMux)
    
    fmt.Println("=== HTTP Performance Monitoring Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Request/response metrics collection")
    fmt.Println("• Performance percentiles (P50, P95, P99)")
    fmt.Println("• Error rate monitoring")
    fmt.Println("• Memory and runtime statistics")
    fmt.Println("• Real-time dashboard")
    fmt.Println("• Health check endpoints")
    fmt.Println()
    fmt.Println("Monitoring endpoints:")
    fmt.Println("  GET / - Performance dashboard")
    fmt.Println("  GET /metrics - Raw metrics (JSON)")
    fmt.Println("  GET /health - Health check")
    fmt.Println()
    fmt.Println("Test endpoints:")
    fmt.Println("  GET /fast - Fast response (~10ms)")
    fmt.Println("  GET /slow - Slow response (100-500ms)")
    fmt.Println("  GET /error - Random errors (30% failure)")
    fmt.Println("  GET /large?size=N - Large response")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This performance monitoring example demonstrates comprehensive metrics  
collection including request counting, response time percentiles, error  
tracking, and memory monitoring. Performance monitoring is essential for  
maintaining application health and identifying optimization opportunities.

## HTTP request tracing and debugging

Request tracing provides detailed insights into request processing flow  
and helps debug complex issues. This example demonstrates comprehensive  
tracing, logging, and debugging capabilities.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "os"
    "runtime"
    "strconv"
    "strings"
    "sync"
    "time"
)

type TraceID string
type SpanID string

type Span struct {
    TraceID   TraceID                `json:"trace_id"`
    SpanID    SpanID                 `json:"span_id"`
    ParentID  SpanID                 `json:"parent_id,omitempty"`
    Operation string                 `json:"operation"`
    StartTime time.Time              `json:"start_time"`
    EndTime   time.Time              `json:"end_time,omitempty"`
    Duration  time.Duration          `json:"duration"`
    Tags      map[string]interface{} `json:"tags"`
    Logs      []LogEntry             `json:"logs"`
    Status    string                 `json:"status"`
    Error     string                 `json:"error,omitempty"`
}

type LogEntry struct {
    Timestamp time.Time              `json:"timestamp"`
    Level     string                 `json:"level"`
    Message   string                 `json:"message"`
    Fields    map[string]interface{} `json:"fields"`
}

type Tracer struct {
    spans map[TraceID][]*Span
    mu    sync.RWMutex
}

func NewTracer() *Tracer {
    return &Tracer{
        spans: make(map[TraceID][]*Span),
    }
}

func (t *Tracer) StartSpan(ctx context.Context, operation string) (*Span, context.Context) {
    traceID := TraceID(fmt.Sprintf("trace_%d", time.Now().UnixNano()))
    spanID := SpanID(fmt.Sprintf("span_%d", time.Now().UnixNano()))
    
    // Check for parent span in context
    if parentSpan := SpanFromContext(ctx); parentSpan != nil {
        traceID = parentSpan.TraceID
    }
    
    span := &Span{
        TraceID:   traceID,
        SpanID:    spanID,
        Operation: operation,
        StartTime: time.Now(),
        Tags:      make(map[string]interface{}),
        Logs:      make([]LogEntry, 0),
        Status:    "started",
    }
    
    t.mu.Lock()
    t.spans[traceID] = append(t.spans[traceID], span)
    t.mu.Unlock()
    
    ctx = context.WithValue(ctx, "current_span", span)
    return span, ctx
}

func (t *Tracer) FinishSpan(span *Span) {
    span.EndTime = time.Now()
    span.Duration = span.EndTime.Sub(span.StartTime)
    if span.Status == "started" {
        span.Status = "finished"
    }
}

func (t *Tracer) GetTrace(traceID TraceID) []*Span {
    t.mu.RLock()
    defer t.mu.RUnlock()
    
    spans := make([]*Span, len(t.spans[traceID]))
    copy(spans, t.spans[traceID])
    return spans
}

func (t *Tracer) GetAllTraces() map[TraceID][]*Span {
    t.mu.RLock()
    defer t.mu.RUnlock()
    
    result := make(map[TraceID][]*Span)
    for traceID, spans := range t.spans {
        result[traceID] = make([]*Span, len(spans))
        copy(result[traceID], spans)
    }
    return result
}

func SpanFromContext(ctx context.Context) *Span {
    if span, ok := ctx.Value("current_span").(*Span); ok {
        return span
    }
    return nil
}

func (s *Span) SetTag(key string, value interface{}) {
    s.Tags[key] = value
}

func (s *Span) SetError(err error) {
    s.Status = "error"
    s.Error = err.Error()
    s.SetTag("error", true)
}

func (s *Span) LogInfo(message string, fields map[string]interface{}) {
    s.Logs = append(s.Logs, LogEntry{
        Timestamp: time.Now(),
        Level:     "info",
        Message:   message,
        Fields:    fields,
    })
}

func (s *Span) LogError(message string, err error) {
    fields := map[string]interface{}{"error": err.Error()}
    s.Logs = append(s.Logs, LogEntry{
        Timestamp: time.Now(),
        Level:     "error",
        Message:   message,
        Fields:    fields,
    })
}

var globalTracer = NewTracer()

func TracingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        span, ctx := globalTracer.StartSpan(r.Context(), 
                                          fmt.Sprintf("%s %s", r.Method, r.URL.Path))
        defer globalTracer.FinishSpan(span)
        
        // Add request information to span
        span.SetTag("http.method", r.Method)
        span.SetTag("http.url", r.URL.String())
        span.SetTag("http.user_agent", r.Header.Get("User-Agent"))
        span.SetTag("http.remote_addr", r.RemoteAddr)
        
        // Add trace ID to response headers
        w.Header().Set("X-Trace-ID", string(span.TraceID))
        w.Header().Set("X-Span-ID", string(span.SpanID))
        
        span.LogInfo("Request started", map[string]interface{}{
            "method": r.Method,
            "path":   r.URL.Path,
            "query":  r.URL.RawQuery,
        })
        
        // Wrap response writer to capture status code
        rw := &responseWriterWithStatus{ResponseWriter: w, statusCode: 200}
        
        // Execute handler with tracing context
        next.ServeHTTP(rw, r.WithContext(ctx))
        
        // Add response information
        span.SetTag("http.status_code", rw.statusCode)
        
        if rw.statusCode >= 400 {
            span.SetTag("error", true)
            span.Status = "error"
        }
        
        span.LogInfo("Request completed", map[string]interface{}{
            "status_code": rw.statusCode,
            "duration_ms": span.Duration.Seconds() * 1000,
        })
    })
}

type responseWriterWithStatus struct {
    http.ResponseWriter
    statusCode int
}

func (rw *responseWriterWithStatus) WriteHeader(statusCode int) {
    rw.statusCode = statusCode
    rw.ResponseWriter.WriteHeader(statusCode)
}

// Debug middleware that adds detailed logging
func DebugMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if strings.Contains(r.Header.Get("X-Debug"), "true") {
            span := SpanFromContext(r.Context())
            if span != nil {
                // Log all headers
                headerMap := make(map[string]interface{})
                for name, values := range r.Header {
                    headerMap[name] = strings.Join(values, ", ")
                }
                span.LogInfo("Request headers", headerMap)
                
                // Log goroutine info
                span.LogInfo("Runtime info", map[string]interface{}{
                    "goroutines": runtime.NumGoroutine(),
                    "memory_mb":  getMemoryUsage(),
                })
            }
        }
        
        next.ServeHTTP(w, r)
    })
}

func getMemoryUsage() float64 {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    return float64(m.Alloc) / 1024 / 1024
}

// Sample handlers with tracing
func tracedDataHandler(w http.ResponseWriter, r *http.Request) {
    span := SpanFromContext(r.Context())
    if span != nil {
        span.LogInfo("Processing data request", nil)
    }
    
    // Simulate database operation
    dbSpan, ctx := globalTracer.StartSpan(r.Context(), "database.query")
    defer globalTracer.FinishSpan(dbSpan)
    
    dbSpan.SetTag("db.table", "users")
    dbSpan.SetTag("db.operation", "SELECT")
    
    time.Sleep(50 * time.Millisecond) // Simulate DB query
    
    dbSpan.LogInfo("Database query executed", map[string]interface{}{
        "query": "SELECT * FROM users",
        "rows":  3,
    })
    
    // Simulate cache operation
    cacheSpan, _ := globalTracer.StartSpan(ctx, "cache.get")
    defer globalTracer.FinishSpan(cacheSpan)
    
    cacheSpan.SetTag("cache.key", "user_data")
    cacheSpan.SetTag("cache.hit", false)
    
    time.Sleep(10 * time.Millisecond) // Simulate cache operation
    
    response := map[string]interface{}{
        "message": "Hello there! Traced data response",
        "data": []map[string]interface{}{
            {"id": 1, "name": "Alice"},
            {"id": 2, "name": "Bob"},
            {"id": 3, "name": "Charlie"},
        },
        "trace_id": string(span.TraceID),
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    if span != nil {
        span.LogInfo("Response prepared", map[string]interface{}{
            "item_count": 3,
        })
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func errorHandler(w http.ResponseWriter, r *http.Request) {
    span := SpanFromContext(r.Context())
    
    // Simulate error condition
    if r.URL.Query().Get("force_error") == "true" {
        err := fmt.Errorf("simulated error condition")
        
        if span != nil {
            span.SetError(err)
            span.LogError("Error occurred", err)
        }
        
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }
    
    response := map[string]interface{}{
        "message": "Hello there! Error handler success",
        "status":  "ok",
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func traceHandler(w http.ResponseWriter, r *http.Request) {
    traceIDParam := r.URL.Query().Get("trace_id")
    if traceIDParam == "" {
        http.Error(w, "trace_id parameter required", http.StatusBadRequest)
        return
    }
    
    traceID := TraceID(traceIDParam)
    spans := globalTracer.GetTrace(traceID)
    
    if len(spans) == 0 {
        http.Error(w, "Trace not found", http.StatusNotFound)
        return
    }
    
    response := map[string]interface{}{
        "trace_id":   traceID,
        "span_count": len(spans),
        "spans":      spans,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func allTracesHandler(w http.ResponseWriter, r *http.Request) {
    allTraces := globalTracer.GetAllTraces()
    
    summary := make([]map[string]interface{}, 0)
    for traceID, spans := range allTraces {
        if len(spans) == 0 {
            continue
        }
        
        firstSpan := spans[0]
        var totalDuration time.Duration
        errorCount := 0
        
        for _, span := range spans {
            if span.Duration > 0 {
                totalDuration += span.Duration
            }
            if span.Status == "error" {
                errorCount++
            }
        }
        
        summary = append(summary, map[string]interface{}{
            "trace_id":      traceID,
            "span_count":    len(spans),
            "start_time":    firstSpan.StartTime.Format(time.RFC3339),
            "total_duration_ms": totalDuration.Seconds() * 1000,
            "error_count":   errorCount,
            "root_operation": firstSpan.Operation,
        })
    }
    
    response := map[string]interface{}{
        "message":     "All traces summary",
        "trace_count": len(allTraces),
        "traces":      summary,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func tracingDashboardHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Tracing Dashboard</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .trace-card { background: #f8f9fa; padding: 15px; margin: 10px 0; border-radius: 5px; border-left: 4px solid #007cba; }
            .span-item { background: #fff; margin: 5px 0; padding: 10px; border-radius: 3px; border-left: 2px solid #28a745; }
            .span-error { border-left-color: #dc3545; }
            .trace-id { font-family: monospace; background: #e9ecef; padding: 2px 6px; border-radius: 3px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; }
            .debug-info { background: #fff3cd; padding: 10px; border-radius: 3px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <h1>HTTP Tracing Dashboard</h1>
        <p>Hello there! Real-time request tracing and debugging.</p>
        
        <div class="debug-info">
            <h3>Test Endpoints</h3>
            <button onclick="makeTracedRequest()">Make Traced Request</button>
            <button onclick="makeErrorRequest()">Trigger Error</button>
            <button onclick="makeDebugRequest()">Debug Request</button>
            <button onclick="refreshTraces()">Refresh Traces</button>
        </div>
        
        <div id="current-trace" style="margin: 20px 0;">
            <h3>Current Request Trace</h3>
            <div id="trace-result">No trace data</div>
        </div>
        
        <div id="all-traces">
            <h3>All Traces</h3>
            <div id="traces-list">Loading...</div>
        </div>
        
        <div style="margin-top: 30px;">
            <h3>Test Commands</h3>
            <pre>
# Make traced request
curl -H "X-Debug: true" http://localhost:8080/data

# Trigger error
curl "http://localhost:8080/error?force_error=true"

# Get specific trace
curl "http://localhost:8080/trace?trace_id=TRACE_ID"

# Get all traces
curl http://localhost:8080/traces
            </pre>
        </div>
        
        <script>
        let currentTraceId = null;
        
        function makeTracedRequest() {
            fetch('/data', {
                headers: {
                    'X-Debug': 'true'
                }
            })
            .then(response => {
                currentTraceId = response.headers.get('X-Trace-ID');
                return response.json();
            })
            .then(data => {
                document.getElementById('trace-result').innerHTML = 
                    '<h4>Response Data</h4>' +
                    '<p>Trace ID: <span class="trace-id">' + currentTraceId + '</span></p>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                
                // Get detailed trace
                return fetch('/trace?trace_id=' + currentTraceId);
            })
            .then(response => response.json())
            .then(traceData => {
                displayTraceDetails(traceData);
                refreshTraces();
            })
            .catch(error => {
                document.getElementById('trace-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function makeErrorRequest() {
            fetch('/error?force_error=true')
            .then(response => {
                currentTraceId = response.headers.get('X-Trace-ID');
                return response.text();
            })
            .then(data => {
                if (currentTraceId) {
                    return fetch('/trace?trace_id=' + currentTraceId);
                }
            })
            .then(response => response ? response.json() : null)
            .then(traceData => {
                if (traceData) {
                    displayTraceDetails(traceData);
                    refreshTraces();
                }
            })
            .catch(error => {
                console.error('Error request failed:', error);
                refreshTraces();
            });
        }
        
        function makeDebugRequest() {
            fetch('/data', {
                headers: {
                    'X-Debug': 'true',
                    'User-Agent': 'TracingDashboard/1.0'
                }
            })
            .then(response => {
                currentTraceId = response.headers.get('X-Trace-ID');
                return response.json();
            })
            .then(data => {
                return fetch('/trace?trace_id=' + currentTraceId);
            })
            .then(response => response.json())
            .then(traceData => {
                displayTraceDetails(traceData);
                refreshTraces();
            });
        }
        
        function displayTraceDetails(traceData) {
            let html = '<h4>Trace Details</h4>';
            html += '<p>Trace ID: <span class="trace-id">' + traceData.trace_id + '</span></p>';
            html += '<p>Spans: ' + traceData.span_count + '</p>';
            
            traceData.spans.forEach(span => {
                const spanClass = span.status === 'error' ? 'span-item span-error' : 'span-item';
                html += '<div class="' + spanClass + '">';
                html += '<strong>' + span.operation + '</strong> (' + span.duration.toFixed(2) + 'ms)';
                html += '<br>Status: ' + span.status;
                
                if (span.error) {
                    html += '<br><span style="color: red;">Error: ' + span.error + '</span>';
                }
                
                if (span.tags && Object.keys(span.tags).length > 0) {
                    html += '<br>Tags: ' + JSON.stringify(span.tags);
                }
                
                if (span.logs && span.logs.length > 0) {
                    html += '<br>Logs:';
                    span.logs.forEach(log => {
                        html += '<br>&nbsp;&nbsp;' + log.level.toUpperCase() + ': ' + log.message;
                    });
                }
                
                html += '</div>';
            });
            
            document.getElementById('trace-result').innerHTML = html;
        }
        
        function refreshTraces() {
            fetch('/traces')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Recent Traces (' + data.trace_count + ')</h4>';
                
                data.traces.forEach(trace => {
                    html += '<div class="trace-card">';
                    html += '<strong>' + trace.root_operation + '</strong>';
                    html += '<br>Trace ID: <span class="trace-id">' + trace.trace_id + '</span>';
                    html += '<br>Spans: ' + trace.span_count + ', Duration: ' + trace.total_duration_ms.toFixed(2) + 'ms';
                    html += '<br>Started: ' + new Date(trace.start_time).toLocaleString();
                    
                    if (trace.error_count > 0) {
                        html += '<br><span style="color: red;">Errors: ' + trace.error_count + '</span>';
                    }
                    
                    html += '<br><button onclick="loadTrace(\'' + trace.trace_id + '\')">View Details</button>';
                    html += '</div>';
                });
                
                document.getElementById('traces-list').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('traces-list').innerHTML = 
                    '<p style="color: red;">Error loading traces: ' + error + '</p>';
            });
        }
        
        function loadTrace(traceId) {
            fetch('/trace?trace_id=' + traceId)
            .then(response => response.json())
            .then(traceData => {
                displayTraceDetails(traceData);
            })
            .catch(error => {
                document.getElementById('trace-result').innerHTML = 
                    '<p style="color: red;">Error loading trace: ' + error + '</p>';
            });
        }
        
        // Auto-refresh traces every 10 seconds
        setInterval(refreshTraces, 10000);
        
        // Initial load
        refreshTraces();
        </script>
    </body>
    </html>`)
}

func main() {
    mux := http.NewServeMux()
    
    // Apply tracing and debug middleware
    tracedMux := TracingMiddleware(DebugMiddleware(mux))
    
    // Dashboard
    mux.HandleFunc("/", tracingDashboardHandler)
    
    // Application endpoints
    mux.HandleFunc("/data", tracedDataHandler)
    mux.HandleFunc("/error", errorHandler)
    
    // Tracing endpoints (not traced to avoid recursion)
    http.HandleFunc("/trace", traceHandler)
    http.HandleFunc("/traces", allTracesHandler)
    
    // Main handler with tracing
    http.Handle("/", tracedMux)
    
    fmt.Println("=== HTTP Tracing and Debugging Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Distributed tracing with spans")
    fmt.Println("• Request/response correlation")
    fmt.Println("• Error tracking and debugging")
    fmt.Println("• Performance monitoring")
    fmt.Println("• Debug middleware")
    fmt.Println("• Interactive tracing dashboard")
    fmt.Println()
    fmt.Println("Tracing endpoints:")
    fmt.Println("  GET / - Tracing dashboard")
    fmt.Println("  GET /trace?trace_id=ID - Get specific trace")
    fmt.Println("  GET /traces - Get all traces")
    fmt.Println()
    fmt.Println("Test endpoints:")
    fmt.Println("  GET /data - Data endpoint with DB/cache tracing")
    fmt.Println("  GET /error?force_error=true - Error simulation")
    fmt.Println()
    fmt.Println("Use X-Debug: true header for detailed logging")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This tracing example demonstrates comprehensive request tracing and debugging  
capabilities including distributed tracing, span creation, error tracking,  
and performance monitoring. Tracing is essential for understanding request  
flow and debugging issues in complex distributed systems.

## HTTP load balancing and health checks

Load balancing distributes requests across multiple backend servers while  
health checks ensure traffic only goes to healthy instances. This example  
demonstrates various load balancing strategies and health monitoring.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "net/http/httputil"
    "net/url"
    "strconv"
    "sync"
    "sync/atomic"
    "time"
)

type Backend struct {
    URL          *url.URL
    Proxy        *httputil.ReverseProxy
    Healthy      int32  // atomic boolean (1 = healthy, 0 = unhealthy)
    Connections  int32  // current active connections
    RequestCount int64  // total requests served
    ErrorCount   int64  // total errors
    LastCheck    time.Time
    ResponseTime time.Duration
}

func (b *Backend) IsHealthy() bool {
    return atomic.LoadInt32(&b.Healthy) == 1
}

func (b *Backend) SetHealthy(healthy bool) {
    if healthy {
        atomic.StoreInt32(&b.Healthy, 1)
    } else {
        atomic.StoreInt32(&b.Healthy, 0)
    }
}

func (b *Backend) AddConnection() {
    atomic.AddInt32(&b.Connections, 1)
}

func (b *Backend) RemoveConnection() {
    atomic.AddInt32(&b.Connections, -1)
}

func (b *Backend) IncrementRequests() {
    atomic.AddInt64(&b.RequestCount, 1)
}

func (b *Backend) IncrementErrors() {
    atomic.AddInt64(&b.ErrorCount, 1)
}

func (b *Backend) GetStats() map[string]interface{} {
    return map[string]interface{}{
        "url":           b.URL.String(),
        "healthy":       b.IsHealthy(),
        "connections":   atomic.LoadInt32(&b.Connections),
        "requests":      atomic.LoadInt64(&b.RequestCount),
        "errors":        atomic.LoadInt64(&b.ErrorCount),
        "last_check":    b.LastCheck.Format(time.RFC3339),
        "response_time": b.ResponseTime.Milliseconds(),
    }
}

type LoadBalancer struct {
    backends       []*Backend
    current        int32 // for round-robin
    strategy       string
    healthChecker  *HealthChecker
    mu             sync.RWMutex
}

type HealthChecker struct {
    interval     time.Duration
    timeout      time.Duration
    endpoint     string
    lb           *LoadBalancer
    stopCh       chan struct{}
}

func NewLoadBalancer(strategy string) *LoadBalancer {
    lb := &LoadBalancer{
        backends: make([]*Backend, 0),
        strategy: strategy,
    }
    
    lb.healthChecker = &HealthChecker{
        interval: 30 * time.Second,
        timeout:  5 * time.Second,
        endpoint: "/health",
        lb:       lb,
        stopCh:   make(chan struct{}),
    }
    
    return lb
}

func (lb *LoadBalancer) AddBackend(urlStr string) error {
    parsedURL, err := url.Parse(urlStr)
    if err != nil {
        return err
    }
    
    proxy := httputil.NewSingleHostReverseProxy(parsedURL)
    
    // Customize proxy behavior
    originalDirector := proxy.Director
    proxy.Director = func(req *http.Request) {
        originalDirector(req)
        req.Header.Set("X-Forwarded-Host", req.Header.Get("Host"))
        req.Header.Set("X-Proxy", "Go-LoadBalancer/1.0")
    }
    
    backend := &Backend{
        URL:       parsedURL,
        Proxy:     proxy,
        Healthy:   1, // Start as healthy
        LastCheck: time.Now(),
    }
    
    lb.mu.Lock()
    lb.backends = append(lb.backends, backend)
    lb.mu.Unlock()
    
    return nil
}

func (lb *LoadBalancer) GetBackend(r *http.Request) *Backend {
    lb.mu.RLock()
    defer lb.mu.RUnlock()
    
    healthyBackends := make([]*Backend, 0)
    for _, backend := range lb.backends {
        if backend.IsHealthy() {
            healthyBackends = append(healthyBackends, backend)
        }
    }
    
    if len(healthyBackends) == 0 {
        return nil
    }
    
    switch lb.strategy {
    case "round-robin":
        return lb.roundRobin(healthyBackends)
    case "least-connections":
        return lb.leastConnections(healthyBackends)
    case "weighted-round-robin":
        return lb.weightedRoundRobin(healthyBackends)
    case "ip-hash":
        return lb.ipHash(healthyBackends, r)
    default:
        return lb.roundRobin(healthyBackends)
    }
}

func (lb *LoadBalancer) roundRobin(backends []*Backend) *Backend {
    if len(backends) == 0 {
        return nil
    }
    
    next := atomic.AddInt32(&lb.current, 1)
    return backends[(next-1)%int32(len(backends))]
}

func (lb *LoadBalancer) leastConnections(backends []*Backend) *Backend {
    if len(backends) == 0 {
        return nil
    }
    
    var selected *Backend
    minConnections := int32(^uint32(0) >> 1) // Max int32
    
    for _, backend := range backends {
        connections := atomic.LoadInt32(&backend.Connections)
        if connections < minConnections {
            minConnections = connections
            selected = backend
        }
    }
    
    return selected
}

func (lb *LoadBalancer) weightedRoundRobin(backends []*Backend) *Backend {
    // Simple weight based on inverse error rate
    if len(backends) == 0 {
        return nil
    }
    
    var selected *Backend
    bestScore := float64(-1)
    
    for _, backend := range backends {
        requests := atomic.LoadInt64(&backend.RequestCount)
        errors := atomic.LoadInt64(&backend.ErrorCount)
        
        // Calculate score (lower error rate = higher score)
        errorRate := float64(0)
        if requests > 0 {
            errorRate = float64(errors) / float64(requests)
        }
        
        score := 1.0 - errorRate
        
        if score > bestScore {
            bestScore = score
            selected = backend
        }
    }
    
    return selected
}

func (lb *LoadBalancer) ipHash(backends []*Backend, r *http.Request) *Backend {
    if len(backends) == 0 {
        return nil
    }
    
    // Simple hash of client IP
    clientIP := r.RemoteAddr
    hash := 0
    for _, c := range clientIP {
        hash = hash*31 + int(c)
    }
    
    index := hash % len(backends)
    if index < 0 {
        index = -index
    }
    
    return backends[index]
}

func (lb *LoadBalancer) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    backend := lb.GetBackend(r)
    if backend == nil {
        http.Error(w, "No healthy backends available", http.StatusServiceUnavailable)
        return
    }
    
    backend.AddConnection()
    backend.IncrementRequests()
    defer backend.RemoveConnection()
    
    // Add backend info to response headers
    w.Header().Set("X-Backend-Server", backend.URL.Host)
    w.Header().Set("X-Load-Balancer", "Go-LB/1.0")
    
    // Custom response writer to track errors
    rw := &responseWriterWithTracking{
        ResponseWriter: w,
        backend:       backend,
    }
    
    backend.Proxy.ServeHTTP(rw, r)
}

type responseWriterWithTracking struct {
    http.ResponseWriter
    backend    *Backend
    statusCode int
}

func (rw *responseWriterWithTracking) WriteHeader(statusCode int) {
    rw.statusCode = statusCode
    if statusCode >= 500 {
        rw.backend.IncrementErrors()
    }
    rw.ResponseWriter.WriteHeader(statusCode)
}

func (hc *HealthChecker) Start() {
    ticker := time.NewTicker(hc.interval)
    defer ticker.Stop()
    
    log.Printf("Health checker started (interval: %v)", hc.interval)
    
    // Initial health check
    hc.checkAllBackends()
    
    for {
        select {
        case <-ticker.C:
            hc.checkAllBackends()
        case <-hc.stopCh:
            log.Println("Health checker stopped")
            return
        }
    }
}

func (hc *HealthChecker) Stop() {
    close(hc.stopCh)
}

func (hc *HealthChecker) checkAllBackends() {
    hc.lb.mu.RLock()
    backends := make([]*Backend, len(hc.lb.backends))
    copy(backends, hc.lb.backends)
    hc.lb.mu.RUnlock()
    
    for _, backend := range backends {
        go hc.checkBackend(backend)
    }
}

func (hc *HealthChecker) checkBackend(backend *Backend) {
    start := time.Now()
    
    ctx, cancel := context.WithTimeout(context.Background(), hc.timeout)
    defer cancel()
    
    healthURL := backend.URL.String() + hc.endpoint
    req, err := http.NewRequestWithContext(ctx, "GET", healthURL, nil)
    if err != nil {
        backend.SetHealthy(false)
        log.Printf("Health check failed for %s: %v", backend.URL.Host, err)
        return
    }
    
    client := &http.Client{Timeout: hc.timeout}
    resp, err := client.Do(req)
    
    backend.ResponseTime = time.Since(start)
    backend.LastCheck = time.Now()
    
    if err != nil {
        backend.SetHealthy(false)
        log.Printf("Health check failed for %s: %v", backend.URL.Host, err)
        return
    }
    defer resp.Body.Close()
    
    if resp.StatusCode >= 200 && resp.StatusCode < 300 {
        if !backend.IsHealthy() {
            log.Printf("Backend %s is now healthy", backend.URL.Host)
        }
        backend.SetHealthy(true)
    } else {
        if backend.IsHealthy() {
            log.Printf("Backend %s is now unhealthy (status: %d)", 
                      backend.URL.Host, resp.StatusCode)
        }
        backend.SetHealthy(false)
    }
}

func (lb *LoadBalancer) GetStats() map[string]interface{} {
    lb.mu.RLock()
    defer lb.mu.RUnlock()
    
    backendStats := make([]map[string]interface{}, len(lb.backends))
    totalRequests := int64(0)
    totalErrors := int64(0)
    healthyCount := 0
    
    for i, backend := range lb.backends {
        stats := backend.GetStats()
        backendStats[i] = stats
        
        totalRequests += atomic.LoadInt64(&backend.RequestCount)
        totalErrors += atomic.LoadInt64(&backend.ErrorCount)
        
        if backend.IsHealthy() {
            healthyCount++
        }
    }
    
    errorRate := float64(0)
    if totalRequests > 0 {
        errorRate = float64(totalErrors) / float64(totalRequests) * 100
    }
    
    return map[string]interface{}{
        "strategy":       lb.strategy,
        "total_backends": len(lb.backends),
        "healthy_backends": healthyCount,
        "total_requests": totalRequests,
        "total_errors":   totalErrors,
        "error_rate":     errorRate,
        "backends":       backendStats,
    }
}

// Mock backend servers for testing
func createMockBackend(port int, healthy bool, delay time.Duration) {
    mux := http.NewServeMux()
    
    mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        if delay > 0 {
            time.Sleep(delay)
        }
        
        response := map[string]interface{}{
            "message":    "Hello there from backend!",
            "server":     fmt.Sprintf("backend-%d", port),
            "timestamp":  time.Now().Format(time.RFC3339),
            "request_id": fmt.Sprintf("req_%d", time.Now().UnixNano()),
        }
        
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("X-Backend-Port", strconv.Itoa(port))
        
        json.NewEncoder(w).Encode(response)
    })
    
    mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        if !healthy {
            w.WriteHeader(http.StatusServiceUnavailable)
            fmt.Fprint(w, "Unhealthy")
            return
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(map[string]interface{}{
            "status":    "healthy",
            "server":    fmt.Sprintf("backend-%d", port),
            "timestamp": time.Now().Format(time.RFC3339),
        })
    })
    
    mux.HandleFunc("/slow", func(w http.ResponseWriter, r *http.Request) {
        time.Sleep(2 * time.Second)
        fmt.Fprintf(w, "Slow response from backend-%d", port)
    })
    
    server := &http.Server{
        Addr:    fmt.Sprintf(":%d", port),
        Handler: mux,
    }
    
    log.Printf("Mock backend starting on port %d (healthy: %v)", port, healthy)
    log.Fatal(server.ListenAndServe())
}

func statsHandler(lb *LoadBalancer) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        stats := lb.GetStats()
        
        w.Header().Set("Content-Type", "application/json")
        w.Header().Set("Cache-Control", "no-cache")
        
        json.NewEncoder(w).Encode(stats)
    }
}

func dashboardHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Load Balancer Dashboard</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .dashboard { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
            .metric-card { background: #f8f9fa; padding: 20px; border-radius: 8px; border: 1px solid #ddd; }
            .backend-card { background: #fff; padding: 15px; margin: 10px 0; border-radius: 5px; border-left: 4px solid #28a745; }
            .backend-unhealthy { border-left-color: #dc3545; }
            .metric-value { font-size: 1.5em; font-weight: bold; color: #007cba; }
            table { width: 100%; border-collapse: collapse; margin-top: 10px; }
            th, td { padding: 8px; text-align: left; border-bottom: 1px solid #ddd; }
            th { background: #f8f9fa; }
            button { margin: 5px; padding: 10px 15px; }
            .status-healthy { color: #28a745; font-weight: bold; }
            .status-unhealthy { color: #dc3545; font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>Load Balancer Dashboard</h1>
        <p>Hello there! Monitor load balancer performance and backend health.</p>
        
        <div class="dashboard">
            <div class="metric-card">
                <div>Strategy</div>
                <div class="metric-value" id="strategy">-</div>
            </div>
            
            <div class="metric-card">
                <div>Healthy Backends</div>
                <div class="metric-value" id="healthy-backends">-</div>
            </div>
            
            <div class="metric-card">
                <div>Total Requests</div>
                <div class="metric-value" id="total-requests">-</div>
            </div>
            
            <div class="metric-card">
                <div>Error Rate</div>
                <div class="metric-value" id="error-rate">-</div>
            </div>
        </div>
        
        <div style="margin-top: 30px;">
            <h2>Backend Status</h2>
            <div id="backends-list"></div>
        </div>
        
        <div style="margin-top: 30px;">
            <h2>Load Testing</h2>
            <button onclick="sendRequests(10)">Send 10 Requests</button>
            <button onclick="sendRequests(100)">Send 100 Requests</button>
            <button onclick="loadTest()">Continuous Load Test</button>
            <button onclick="stopLoadTest()">Stop Load Test</button>
            <div id="test-result" style="margin-top: 10px;"></div>
        </div>
        
        <div style="margin-top: 30px;">
            <h2>Test Different Strategies</h2>
            <button onclick="testStrategy('round-robin')">Round Robin</button>
            <button onclick="testStrategy('least-connections')">Least Connections</button>
            <button onclick="testStrategy('weighted-round-robin')">Weighted RR</button>
            <button onclick="testStrategy('ip-hash')">IP Hash</button>
        </div>
        
        <script>
        let loadTestInterval = null;
        
        function refreshStats() {
            fetch('/stats')
            .then(response => response.json())
            .then(data => {
                document.getElementById('strategy').textContent = data.strategy;
                document.getElementById('healthy-backends').textContent = 
                    data.healthy_backends + '/' + data.total_backends;
                document.getElementById('total-requests').textContent = 
                    data.total_requests.toLocaleString();
                document.getElementById('error-rate').textContent = 
                    data.error_rate.toFixed(2) + '%';
                
                // Update backends list
                let html = '';
                data.backends.forEach(backend => {
                    const healthyClass = backend.healthy ? 'backend-card' : 'backend-card backend-unhealthy';
                    const statusClass = backend.healthy ? 'status-healthy' : 'status-unhealthy';
                    const status = backend.healthy ? 'Healthy' : 'Unhealthy';
                    
                    html += '<div class="' + healthyClass + '">';
                    html += '<h4>' + backend.url + ' <span class="' + statusClass + '">' + status + '</span></h4>';
                    html += '<p>Requests: ' + backend.requests + ', Errors: ' + backend.errors + '</p>';
                    html += '<p>Connections: ' + backend.connections + ', Response Time: ' + backend.response_time + 'ms</p>';
                    html += '<p>Last Check: ' + new Date(backend.last_check).toLocaleString() + '</p>';
                    html += '</div>';
                });
                
                document.getElementById('backends-list').innerHTML = html;
            })
            .catch(error => {
                console.error('Error fetching stats:', error);
            });
        }
        
        function sendRequests(count) {
            document.getElementById('test-result').innerHTML = 'Sending ' + count + ' requests...';
            
            const promises = [];
            for (let i = 0; i < count; i++) {
                promises.push(
                    fetch('/')
                    .then(response => ({
                        status: response.status,
                        backend: response.headers.get('X-Backend-Server')
                    }))
                    .catch(error => ({ error: error.message }))
                );
            }
            
            Promise.all(promises).then(results => {
                const successCount = results.filter(r => r.status === 200).length;
                const errorCount = results.filter(r => r.error || r.status !== 200).length;
                
                // Count requests per backend
                const backendCounts = {};
                results.forEach(result => {
                    if (result.backend) {
                        backendCounts[result.backend] = (backendCounts[result.backend] || 0) + 1;
                    }
                });
                
                let resultHtml = '<h4>Test Results</h4>';
                resultHtml += '<p>Successful: ' + successCount + ', Errors: ' + errorCount + '</p>';
                resultHtml += '<h5>Distribution:</h5>';
                for (const [backend, count] of Object.entries(backendCounts)) {
                    resultHtml += '<p>' + backend + ': ' + count + ' requests</p>';
                }
                
                document.getElementById('test-result').innerHTML = resultHtml;
                refreshStats();
            });
        }
        
        function loadTest() {
            if (loadTestInterval) return;
            
            document.getElementById('test-result').innerHTML = 'Running continuous load test...';
            
            loadTestInterval = setInterval(() => {
                fetch('/').catch(() => {}); // Ignore errors
            }, 100); // 10 requests per second
        }
        
        function stopLoadTest() {
            if (loadTestInterval) {
                clearInterval(loadTestInterval);
                loadTestInterval = null;
                document.getElementById('test-result').innerHTML = 'Load test stopped';
            }
        }
        
        function testStrategy(strategy) {
            // Note: This would require server restart with different strategy
            // For demo purposes, just show which strategy would be tested
            document.getElementById('test-result').innerHTML = 
                'Testing strategy: ' + strategy + ' (requires server restart)';
        }
        
        // Auto-refresh every 5 seconds
        refreshStats();
        setInterval(refreshStats, 5000);
        </script>
    </body>
    </html>`)
}

func main() {
    strategy := "round-robin"
    if len(os.Args) > 1 {
        strategy = os.Args[1]
    }
    
    // Create load balancer
    lb := NewLoadBalancer(strategy)
    
    // Add backend servers (these would be real servers in production)
    backends := []string{
        "http://localhost:8081",
        "http://localhost:8082", 
        "http://localhost:8083",
    }
    
    for _, backend := range backends {
        if err := lb.AddBackend(backend); err != nil {
            log.Printf("Error adding backend %s: %v", backend, err)
        }
    }
    
    // Start health checker
    go lb.healthChecker.Start()
    
    // Start mock backends for testing
    go createMockBackend(8081, true, 0)
    go createMockBackend(8082, true, 100*time.Millisecond)
    go createMockBackend(8083, false, 0) // Unhealthy backend
    
    // Setup routes
    http.HandleFunc("/stats", statsHandler(lb))
    http.HandleFunc("/dashboard", dashboardHandler)
    http.Handle("/", lb) // All other requests go through load balancer
    
    fmt.Println("=== HTTP Load Balancer Demo ===")
    fmt.Printf("Load balancer starting on :8080 (strategy: %s)\n", strategy)
    fmt.Println()
    fmt.Println("Backend servers:")
    fmt.Println("• :8081 - Fast, healthy")
    fmt.Println("• :8082 - Slow, healthy (100ms delay)")
    fmt.Println("• :8083 - Unhealthy")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET /dashboard - Load balancer dashboard")
    fmt.Println("  GET /stats - Statistics (JSON)")
    fmt.Println("  GET / - Proxied requests to backends")
    fmt.Println()
    fmt.Println("Available strategies: round-robin, least-connections, weighted-round-robin, ip-hash")
    fmt.Println("Health checks run every 30 seconds")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This load balancing example demonstrates comprehensive traffic distribution  
with multiple strategies, health checking, and monitoring. Load balancing  
is essential for building scalable and resilient distributed systems that  
can handle high traffic and provide fault tolerance.

## HTTP request batching and parallel processing

Request batching and parallel processing improve performance when handling  
multiple operations. This example demonstrates various concurrency patterns  
for HTTP requests and response aggregation.

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "sync"
    "time"
)

type BatchRequest struct {
    ID      string      `json:"id"`
    Method  string      `json:"method"`
    URL     string      `json:"url"`
    Headers map[string]string `json:"headers,omitempty"`
    Body    interface{} `json:"body,omitempty"`
    Timeout int         `json:"timeout,omitempty"` // seconds
}

type BatchResponse struct {
    ID         string                 `json:"id"`
    StatusCode int                    `json:"status_code"`
    Headers    map[string]string      `json:"headers"`
    Body       interface{}            `json:"body"`
    Error      string                 `json:"error,omitempty"`
    Duration   int64                  `json:"duration_ms"`
}

type BatchProcessor struct {
    maxConcurrency int
    defaultTimeout time.Duration
    client         *http.Client
}

func NewBatchProcessor(maxConcurrency int, defaultTimeout time.Duration) *BatchProcessor {
    return &BatchProcessor{
        maxConcurrency: maxConcurrency,
        defaultTimeout: defaultTimeout,
        client: &http.Client{
            Timeout: defaultTimeout,
        },
    }
}

func (bp *BatchProcessor) ProcessBatch(ctx context.Context, requests []BatchRequest) []BatchResponse {
    responses := make([]BatchResponse, len(requests))
    
    // Create semaphore for concurrency control
    sem := make(chan struct{}, bp.maxConcurrency)
    var wg sync.WaitGroup
    
    for i, req := range requests {
        wg.Add(1)
        go func(idx int, request BatchRequest) {
            defer wg.Done()
            
            // Acquire semaphore
            sem <- struct{}{}
            defer func() { <-sem }()
            
            responses[idx] = bp.processRequest(ctx, request)
        }(i, req)
    }
    
    wg.Wait()
    return responses
}

func (bp *BatchProcessor) processRequest(ctx context.Context, req BatchRequest) BatchResponse {
    start := time.Now()
    
    response := BatchResponse{
        ID:      req.ID,
        Headers: make(map[string]string),
    }
    
    // Create HTTP request
    httpReq, err := http.NewRequestWithContext(ctx, req.Method, req.URL, nil)
    if err != nil {
        response.Error = fmt.Sprintf("Failed to create request: %v", err)
        response.Duration = time.Since(start).Milliseconds()
        return response
    }
    
    // Add headers
    for key, value := range req.Headers {
        httpReq.Header.Set(key, value)
    }
    
    // Set timeout
    timeout := bp.defaultTimeout
    if req.Timeout > 0 {
        timeout = time.Duration(req.Timeout) * time.Second
    }
    
    client := &http.Client{Timeout: timeout}
    
    // Execute request
    resp, err := client.Do(httpReq)
    if err != nil {
        response.Error = fmt.Sprintf("Request failed: %v", err)
        response.Duration = time.Since(start).Milliseconds()
        return response
    }
    defer resp.Body.Close()
    
    // Copy response headers
    for key, values := range resp.Header {
        if len(values) > 0 {
            response.Headers[key] = values[0]
        }
    }
    
    response.StatusCode = resp.StatusCode
    
    // Read response body
    var body interface{}
    if resp.Header.Get("Content-Type") == "application/json" {
        json.NewDecoder(resp.Body).Decode(&body)
    } else {
        // For non-JSON responses, return as string
        bodyBytes := make([]byte, 1024)
        n, _ := resp.Body.Read(bodyBytes)
        body = string(bodyBytes[:n])
    }
    
    response.Body = body
    response.Duration = time.Since(start).Milliseconds()
    
    return response
}

func batchHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var batchReq struct {
        Requests        []BatchRequest `json:"requests"`
        MaxConcurrency  int           `json:"max_concurrency,omitempty"`
        DefaultTimeout  int           `json:"default_timeout,omitempty"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&batchReq); err != nil {
        http.Error(w, "Invalid JSON: "+err.Error(), http.StatusBadRequest)
        return
    }
    
    // Set defaults
    if batchReq.MaxConcurrency <= 0 {
        batchReq.MaxConcurrency = 10
    }
    if batchReq.DefaultTimeout <= 0 {
        batchReq.DefaultTimeout = 30
    }
    
    processor := NewBatchProcessor(batchReq.MaxConcurrency, 
                                 time.Duration(batchReq.DefaultTimeout)*time.Second)
    
    start := time.Now()
    responses := processor.ProcessBatch(r.Context(), batchReq.Requests)
    totalDuration := time.Since(start)
    
    result := map[string]interface{}{
        "message":        "Hello there! Batch processing completed",
        "total_requests": len(batchReq.Requests),
        "responses":      responses,
        "total_duration_ms": totalDuration.Milliseconds(),
        "max_concurrency": batchReq.MaxConcurrency,
        "timestamp":      time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}

// Parallel data aggregation
type DataSource struct {
    Name string `json:"name"`
    URL  string `json:"url"`
}

type AggregatedData struct {
    Source   string      `json:"source"`
    Data     interface{} `json:"data"`
    Error    string      `json:"error,omitempty"`
    Duration int64       `json:"duration_ms"`
}

func aggregateHandler(w http.ResponseWriter, r *http.Request) {
    sources := []DataSource{
        {"users", "https://jsonplaceholder.typicode.com/users"},
        {"posts", "https://jsonplaceholder.typicode.com/posts"},
        {"todos", "https://jsonplaceholder.typicode.com/todos"},
        {"albums", "https://jsonplaceholder.typicode.com/albums"},
    }
    
    // Fetch data from all sources in parallel
    results := make([]AggregatedData, len(sources))
    var wg sync.WaitGroup
    
    start := time.Now()
    
    for i, source := range sources {
        wg.Add(1)
        go func(idx int, src DataSource) {
            defer wg.Done()
            
            sourceStart := time.Now()
            
            resp, err := http.Get(src.URL)
            if err != nil {
                results[idx] = AggregatedData{
                    Source:   src.Name,
                    Error:    err.Error(),
                    Duration: time.Since(sourceStart).Milliseconds(),
                }
                return
            }
            defer resp.Body.Close()
            
            var data interface{}
            if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
                results[idx] = AggregatedData{
                    Source:   src.Name,
                    Error:    fmt.Sprintf("JSON decode error: %v", err),
                    Duration: time.Since(sourceStart).Milliseconds(),
                }
                return
            }
            
            results[idx] = AggregatedData{
                Source:   src.Name,
                Data:     data,
                Duration: time.Since(sourceStart).Milliseconds(),
            }
        }(i, source)
    }
    
    wg.Wait()
    totalDuration := time.Since(start)
    
    response := map[string]interface{}{
        "message":           "Hello there! Data aggregation completed",
        "sources":           len(sources),
        "results":           results,
        "total_duration_ms": totalDuration.Milliseconds(),
        "timestamp":         time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Worker pool pattern for request processing
type WorkerPool struct {
    workerCount int
    jobQueue    chan Job
    quit        chan bool
    wg          sync.WaitGroup
}

type Job struct {
    ID       string
    URL      string
    Response chan JobResult
}

type JobResult struct {
    ID       string      `json:"id"`
    Data     interface{} `json:"data"`
    Error    string      `json:"error,omitempty"`
    Duration int64       `json:"duration_ms"`
}

func NewWorkerPool(workerCount, queueSize int) *WorkerPool {
    return &WorkerPool{
        workerCount: workerCount,
        jobQueue:    make(chan Job, queueSize),
        quit:        make(chan bool),
    }
}

func (wp *WorkerPool) Start() {
    for i := 0; i < wp.workerCount; i++ {
        wp.wg.Add(1)
        go wp.worker(i)
    }
}

func (wp *WorkerPool) Stop() {
    close(wp.quit)
    wp.wg.Wait()
}

func (wp *WorkerPool) AddJob(job Job) {
    wp.jobQueue <- job
}

func (wp *WorkerPool) worker(id int) {
    defer wp.wg.Done()
    
    for {
        select {
        case job := <-wp.jobQueue:
            start := time.Now()
            
            resp, err := http.Get(job.URL)
            result := JobResult{
                ID:       job.ID,
                Duration: time.Since(start).Milliseconds(),
            }
            
            if err != nil {
                result.Error = err.Error()
            } else {
                defer resp.Body.Close()
                var data interface{}
                if json.NewDecoder(resp.Body).Decode(&data) == nil {
                    result.Data = data
                } else {
                    result.Error = "Failed to decode JSON"
                }
            }
            
            job.Response <- result
            
        case <-wp.quit:
            return
        }
    }
}

var workerPool *WorkerPool

func workerPoolHandler(w http.ResponseWriter, r *http.Request) {
    countParam := r.URL.Query().Get("count")
    count := 10
    if countParam != "" {
        if c, err := strconv.Atoi(countParam); err == nil && c > 0 {
            count = c
        }
    }
    
    jobs := make([]Job, count)
    responses := make(chan JobResult, count)
    
    // Create jobs
    for i := 0; i < count; i++ {
        jobs[i] = Job{
            ID:       fmt.Sprintf("job_%d", i+1),
            URL:      fmt.Sprintf("https://httpbin.org/delay/%d", i%3+1),
            Response: responses,
        }
    }
    
    start := time.Now()
    
    // Submit all jobs
    for _, job := range jobs {
        workerPool.AddJob(job)
    }
    
    // Collect results
    results := make([]JobResult, 0, count)
    for i := 0; i < count; i++ {
        result := <-responses
        results = append(results, result)
    }
    
    totalDuration := time.Since(start)
    
    response := map[string]interface{}{
        "message":           "Hello there! Worker pool processing completed",
        "job_count":         count,
        "worker_count":      workerPool.workerCount,
        "results":           results,
        "total_duration_ms": totalDuration.Milliseconds(),
        "timestamp":         time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func pipelineHandler(w http.ResponseWriter, r *http.Request) {
    // Pipeline pattern: process data through multiple stages
    type PipelineData struct {
        ID    string      `json:"id"`
        Stage string      `json:"stage"`
        Data  interface{} `json:"data"`
    }
    
    input := make(chan PipelineData, 10)
    stage1 := make(chan PipelineData, 10)
    stage2 := make(chan PipelineData, 10)
    output := make(chan PipelineData, 10)
    
    // Stage 1: Fetch data
    go func() {
        defer close(stage1)
        for data := range input {
            data.Stage = "fetch"
            // Simulate data fetching
            time.Sleep(100 * time.Millisecond)
            data.Data = map[string]interface{}{
                "fetched_at": time.Now().Format(time.RFC3339),
                "source":     "external_api",
            }
            stage1 <- data
        }
    }()
    
    // Stage 2: Transform data
    go func() {
        defer close(stage2)
        for data := range stage1 {
            data.Stage = "transform"
            // Simulate data transformation
            time.Sleep(50 * time.Millisecond)
            if dataMap, ok := data.Data.(map[string]interface{}); ok {
                dataMap["transformed_at"] = time.Now().Format(time.RFC3339)
                dataMap["processed"] = true
            }
            stage2 <- data
        }
    }()
    
    // Stage 3: Validate data
    go func() {
        defer close(output)
        for data := range stage2 {
            data.Stage = "validate"
            // Simulate validation
            time.Sleep(25 * time.Millisecond)
            if dataMap, ok := data.Data.(map[string]interface{}); ok {
                dataMap["validated_at"] = time.Now().Format(time.RFC3339)
                dataMap["valid"] = true
            }
            output <- data
        }
    }()
    
    // Send data through pipeline
    count := 5
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        defer close(input)
        for i := 0; i < count; i++ {
            input <- PipelineData{
                ID:   fmt.Sprintf("item_%d", i+1),
                Data: map[string]interface{}{"initial": true},
            }
        }
    }()
    
    // Collect results
    results := make([]PipelineData, 0, count)
    for result := range output {
        results = append(results, result)
    }
    
    wg.Wait()
    
    response := map[string]interface{}{
        "message":    "Hello there! Pipeline processing completed",
        "item_count": count,
        "results":    results,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func batchDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Batch Processing Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            textarea { width: 100%; height: 150px; }
        </style>
    </head>
    <body>
        <h1>HTTP Batch Processing Demo</h1>
        <p>Hello there! This demonstrates various parallel processing patterns.</p>
        
        <div class="demo-section">
            <h2>Batch Request Processing</h2>
            <p>Send multiple HTTP requests in parallel:</p>
            <button onclick="testBatchProcessing()">Test Batch Processing</button>
            <div id="batch-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Data Aggregation</h2>
            <p>Fetch data from multiple sources concurrently:</p>
            <button onclick="testAggregation()">Test Data Aggregation</button>
            <div id="aggregation-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Worker Pool</h2>
            <p>Process jobs using a worker pool pattern:</p>
            <button onclick="testWorkerPool(5)">5 Jobs</button>
            <button onclick="testWorkerPool(20)">20 Jobs</button>
            <button onclick="testWorkerPool(50)">50 Jobs</button>
            <div id="worker-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Pipeline Processing</h2>
            <p>Process data through multiple stages:</p>
            <button onclick="testPipeline()">Test Pipeline</button>
            <div id="pipeline-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Custom Batch Request</h2>
            <p>Create your own batch request:</p>
            <textarea id="batch-json" placeholder="Enter batch request JSON...">
{
  "requests": [
    {
      "id": "request1",
      "method": "GET",
      "url": "https://httpbin.org/get"
    },
    {
      "id": "request2", 
      "method": "GET",
      "url": "https://httpbin.org/headers"
    },
    {
      "id": "request3",
      "method": "GET", 
      "url": "https://httpbin.org/user-agent"
    }
  ],
  "max_concurrency": 3,
  "default_timeout": 10
}
            </textarea>
            <br>
            <button onclick="testCustomBatch()">Send Custom Batch</button>
            <div id="custom-result" class="result"></div>
        </div>
        
        <script>
        function testBatchProcessing() {
            const batchRequest = {
                requests: [
                    { id: "req1", method: "GET", url: "https://httpbin.org/get" },
                    { id: "req2", method: "GET", url: "https://httpbin.org/headers" },
                    { id: "req3", method: "GET", url: "https://httpbin.org/user-agent" },
                    { id: "req4", method: "GET", url: "https://httpbin.org/delay/1" },
                    { id: "req5", method: "GET", url: "https://httpbin.org/delay/2" }
                ],
                max_concurrency: 3,
                default_timeout: 10
            };
            
            document.getElementById('batch-result').innerHTML = 'Processing batch...';
            
            fetch('/batch', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(batchRequest)
            })
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Batch Results</h4>';
                html += '<p>Total Duration: ' + data.total_duration_ms + 'ms</p>';
                html += '<p>Requests: ' + data.total_requests + '</p>';
                
                data.responses.forEach(resp => {
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<strong>' + resp.id + '</strong> - Status: ' + resp.status_code;
                    html += ' (' + resp.duration_ms + 'ms)';
                    if (resp.error) {
                        html += '<br><span style="color: red;">Error: ' + resp.error + '</span>';
                    }
                    html += '</div>';
                });
                
                document.getElementById('batch-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('batch-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testAggregation() {
            document.getElementById('aggregation-result').innerHTML = 'Aggregating data...';
            
            fetch('/aggregate')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Aggregation Results</h4>';
                html += '<p>Total Duration: ' + data.total_duration_ms + 'ms</p>';
                html += '<p>Sources: ' + data.sources + '</p>';
                
                data.results.forEach(result => {
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<strong>' + result.source + '</strong> (' + result.duration_ms + 'ms)';
                    if (result.error) {
                        html += '<br><span style="color: red;">Error: ' + result.error + '</span>';
                    } else {
                        html += '<br>Data items: ' + (Array.isArray(result.data) ? result.data.length : 'N/A');
                    }
                    html += '</div>';
                });
                
                document.getElementById('aggregation-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('aggregation-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testWorkerPool(count) {
            document.getElementById('worker-result').innerHTML = 'Processing ' + count + ' jobs...';
            
            fetch('/worker-pool?count=' + count)
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Worker Pool Results</h4>';
                html += '<p>Jobs: ' + data.job_count + ', Workers: ' + data.worker_count + '</p>';
                html += '<p>Total Duration: ' + data.total_duration_ms + 'ms</p>';
                
                const avgDuration = data.results.reduce((sum, r) => sum + r.duration_ms, 0) / data.results.length;
                html += '<p>Average Job Duration: ' + avgDuration.toFixed(2) + 'ms</p>';
                
                const successCount = data.results.filter(r => !r.error).length;
                html += '<p>Success Rate: ' + ((successCount / data.results.length) * 100).toFixed(1) + '%</p>';
                
                document.getElementById('worker-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('worker-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testPipeline() {
            document.getElementById('pipeline-result').innerHTML = 'Processing pipeline...';
            
            fetch('/pipeline')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Pipeline Results</h4>';
                html += '<p>Items Processed: ' + data.item_count + '</p>';
                
                data.results.forEach(result => {
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<strong>' + result.id + '</strong> - Stage: ' + result.stage;
                    html += '<br><pre>' + JSON.stringify(result.data, null, 2) + '</pre>';
                    html += '</div>';
                });
                
                document.getElementById('pipeline-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('pipeline-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testCustomBatch() {
            const jsonText = document.getElementById('batch-json').value;
            
            try {
                const batchRequest = JSON.parse(jsonText);
                document.getElementById('custom-result').innerHTML = 'Processing custom batch...';
                
                fetch('/batch', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(batchRequest)
                })
                .then(response => response.json())
                .then(data => {
                    let html = '<h4>Custom Batch Results</h4>';
                    html += '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                    document.getElementById('custom-result').innerHTML = html;
                })
                .catch(error => {
                    document.getElementById('custom-result').innerHTML = 
                        '<p style="color: red;">Error: ' + error + '</p>';
                });
                
            } catch (error) {
                document.getElementById('custom-result').innerHTML = 
                    '<p style="color: red;">Invalid JSON: ' + error.message + '</p>';
            }
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Initialize worker pool
    workerPool = NewWorkerPool(5, 100)
    workerPool.Start()
    defer workerPool.Stop()
    
    http.HandleFunc("/", batchDemoHandler)
    http.HandleFunc("/batch", batchHandler)
    http.HandleFunc("/aggregate", aggregateHandler)
    http.HandleFunc("/worker-pool", workerPoolHandler)
    http.HandleFunc("/pipeline", pipelineHandler)
    
    fmt.Println("=== HTTP Batch Processing Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Batch request processing")
    fmt.Println("• Parallel data aggregation")
    fmt.Println("• Worker pool pattern")
    fmt.Println("• Pipeline processing")
    fmt.Println("• Concurrency control")
    fmt.Println("• Request/response correlation")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Batch processing demo")
    fmt.Println("  POST /batch - Process batch requests")
    fmt.Println("  GET  /aggregate - Parallel data aggregation")
    fmt.Println("  GET  /worker-pool?count=N - Worker pool processing")
    fmt.Println("  GET  /pipeline - Pipeline processing demo")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This batch processing example demonstrates comprehensive parallel request  
handling including batch processing, data aggregation, worker pools, and  
pipeline patterns. These techniques are essential for building high-performance  
applications that can efficiently handle multiple concurrent operations.

## HTTP custom transport and dialer

Custom transports and dialers provide fine-grained control over network  
connections, enabling features like connection pooling, custom DNS resolution,  
and advanced networking configurations.

```go
package main

import (
    "context"
    "crypto/tls"
    "encoding/json"
    "fmt"
    "log"
    "net"
    "net/http"
    "time"
)

type CustomDialer struct {
    timeout     time.Duration
    keepAlive   time.Duration
    dnsCache    map[string]string
    dialCount   int64
}

func NewCustomDialer(timeout, keepAlive time.Duration) *CustomDialer {
    return &CustomDialer{
        timeout:   timeout,
        keepAlive: keepAlive,
        dnsCache:  make(map[string]string),
    }
}

func (d *CustomDialer) DialContext(ctx context.Context, network, address string) (net.Conn, error) {
    d.dialCount++
    
    dialer := &net.Dialer{
        Timeout:   d.timeout,
        KeepAlive: d.keepAlive,
    }
    
    log.Printf("Custom dialer: connecting to %s (dial #%d)", address, d.dialCount)
    
    conn, err := dialer.DialContext(ctx, network, address)
    if err != nil {
        return nil, fmt.Errorf("dial failed: %w", err)
    }
    
    // Wrap connection to add logging
    return &loggedConnection{Conn: conn, address: address}, nil
}

type loggedConnection struct {
    net.Conn
    address string
}

func (lc *loggedConnection) Read(b []byte) (int, error) {
    n, err := lc.Conn.Read(b)
    if err != nil {
        log.Printf("Read error from %s: %v", lc.address, err)
    }
    return n, err
}

func (lc *loggedConnection) Write(b []byte) (int, error) {
    n, err := lc.Conn.Write(b)
    if err != nil {
        log.Printf("Write error to %s: %v", lc.address, err)
    }
    return n, err
}

func (lc *loggedConnection) Close() error {
    log.Printf("Closing connection to %s", lc.address)
    return lc.Conn.Close()
}

type CustomTransport struct {
    *http.Transport
    requestCount  int64
    responseCount int64
}

func NewCustomTransport() *CustomTransport {
    dialer := NewCustomDialer(30*time.Second, 30*time.Second)
    
    transport := &http.Transport{
        DialContext: dialer.DialContext,
        TLSClientConfig: &tls.Config{
            InsecureSkipVerify: false,
        },
        MaxIdleConns:        100,
        MaxIdleConnsPerHost: 10,
        IdleConnTimeout:     90 * time.Second,
        DisableCompression:  false,
        ForceAttemptHTTP2:   true,
    }
    
    return &CustomTransport{Transport: transport}
}

func (ct *CustomTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    ct.requestCount++
    
    // Add custom headers
    req.Header.Set("X-Custom-Transport", "Go-Custom/1.0")
    req.Header.Set("X-Request-ID", fmt.Sprintf("req_%d", ct.requestCount))
    
    start := time.Now()
    resp, err := ct.Transport.RoundTrip(req)
    duration := time.Since(start)
    
    if err != nil {
        log.Printf("Request failed: %s %s (%v)", req.Method, req.URL, err)
        return nil, err
    }
    
    ct.responseCount++
    
    // Add response metadata
    resp.Header.Set("X-Response-Time", duration.String())
    resp.Header.Set("X-Transport-Info", fmt.Sprintf("req:%d,resp:%d", ct.requestCount, ct.responseCount))
    
    log.Printf("Request completed: %s %s (%d) in %v", 
              req.Method, req.URL, resp.StatusCode, duration)
    
    return resp, nil
}

func (ct *CustomTransport) GetStats() map[string]interface{} {
    return map[string]interface{}{
        "requests":  ct.requestCount,
        "responses": ct.responseCount,
        "idle_connections": ct.Transport.IdleConnTimeout.String(),
        "max_idle_conns": ct.Transport.MaxIdleConns,
    }
}

// Connection pool monitoring
type PoolMonitor struct {
    transport *CustomTransport
}

func NewPoolMonitor(transport *CustomTransport) *PoolMonitor {
    return &PoolMonitor{transport: transport}
}

func (pm *PoolMonitor) GetConnectionStats() map[string]interface{} {
    // Note: Go's http.Transport doesn't expose detailed connection pool stats
    // This is a simplified example of what monitoring might look like
    return map[string]interface{}{
        "max_idle_conns":         pm.transport.MaxIdleConns,
        "max_idle_conns_per_host": pm.transport.MaxIdleConnsPerHost,
        "idle_conn_timeout":      pm.transport.IdleConnTimeout.String(),
        "tls_handshake_timeout":  pm.transport.TLSHandshakeTimeout.String(),
        "force_attempt_http2":    pm.transport.ForceAttemptHTTP2,
    }
}

func customTransportHandler(w http.ResponseWriter, r *http.Request) {
    // Create client with custom transport
    customTransport := NewCustomTransport()
    client := &http.Client{
        Transport: customTransport,
        Timeout:   30 * time.Second,
    }
    
    // Test different endpoints
    urls := []string{
        "https://httpbin.org/get",
        "https://httpbin.org/headers",
        "https://httpbin.org/user-agent",
        "https://httpbin.org/delay/1",
    }
    
    results := make([]map[string]interface{}, 0)
    
    for _, url := range urls {
        start := time.Now()
        
        resp, err := client.Get(url)
        if err != nil {
            results = append(results, map[string]interface{}{
                "url":   url,
                "error": err.Error(),
                "duration": time.Since(start).Milliseconds(),
            })
            continue
        }
        defer resp.Body.Close()
        
        var body interface{}
        json.NewDecoder(resp.Body).Decode(&body)
        
        results = append(results, map[string]interface{}{
            "url":           url,
            "status_code":   resp.StatusCode,
            "headers":       map[string]string{
                "content-type": resp.Header.Get("Content-Type"),
                "response-time": resp.Header.Get("X-Response-Time"),
                "transport-info": resp.Header.Get("X-Transport-Info"),
            },
            "duration":      time.Since(start).Milliseconds(),
            "body_snippet":  fmt.Sprintf("%.100s...", fmt.Sprintf("%v", body)),
        })
    }
    
    response := map[string]interface{}{
        "message":          "Hello there! Custom transport test completed",
        "transport_stats":  customTransport.GetStats(),
        "connection_stats": NewPoolMonitor(customTransport).GetConnectionStats(),
        "results":          results,
        "timestamp":        time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

// Proxy transport for routing through proxy
type ProxyTransport struct {
    proxyURL   string
    underlying http.RoundTripper
}

func NewProxyTransport(proxyURL string) *ProxyTransport {
    return &ProxyTransport{
        proxyURL:   proxyURL,
        underlying: http.DefaultTransport,
    }
}

func (pt *ProxyTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    // Add proxy headers
    req.Header.Set("X-Forwarded-For", "127.0.0.1")
    req.Header.Set("X-Proxy-By", "Go-Proxy-Transport")
    
    log.Printf("Proxying request: %s %s through %s", req.Method, req.URL, pt.proxyURL)
    
    return pt.underlying.RoundTrip(req)
}

func proxyTransportHandler(w http.ResponseWriter, r *http.Request) {
    proxyTransport := NewProxyTransport("http://proxy.example.com:8080")
    client := &http.Client{
        Transport: proxyTransport,
        Timeout:   30 * time.Second,
    }
    
    // Test request through proxy transport
    resp, err := client.Get("https://httpbin.org/headers")
    if err != nil {
        http.Error(w, "Proxy request failed: "+err.Error(), 
                  http.StatusInternalServerError)
        return
    }
    defer resp.Body.Close()
    
    var responseData interface{}
    json.NewDecoder(resp.Body).Decode(&responseData)
    
    result := map[string]interface{}{
        "message":      "Hello there! Proxy transport test",
        "proxy_url":    proxyTransport.proxyURL,
        "status_code":  resp.StatusCode,
        "response":     responseData,
        "timestamp":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}

// Circuit breaker transport
type CircuitBreakerTransport struct {
    underlying    http.RoundTripper
    failures      int
    lastFailure   time.Time
    threshold     int
    timeout       time.Duration
    state         string // "closed", "open", "half-open"
}

func NewCircuitBreakerTransport(threshold int, timeout time.Duration) *CircuitBreakerTransport {
    return &CircuitBreakerTransport{
        underlying: http.DefaultTransport,
        threshold:  threshold,
        timeout:    timeout,
        state:      "closed",
    }
}

func (cbt *CircuitBreakerTransport) RoundTrip(req *http.Request) (*http.Response, error) {
    switch cbt.state {
    case "open":
        if time.Since(cbt.lastFailure) > cbt.timeout {
            cbt.state = "half-open"
            log.Println("Circuit breaker: transitioning to half-open")
        } else {
            return nil, fmt.Errorf("circuit breaker is open")
        }
    }
    
    resp, err := cbt.underlying.RoundTrip(req)
    
    if err != nil || (resp != nil && resp.StatusCode >= 500) {
        cbt.failures++
        cbt.lastFailure = time.Now()
        
        if cbt.failures >= cbt.threshold {
            cbt.state = "open"
            log.Printf("Circuit breaker: opened due to %d failures", cbt.failures)
        }
        
        if err != nil {
            return nil, err
        }
    } else {
        // Success - reset failure count and close circuit
        if cbt.state == "half-open" {
            cbt.state = "closed"
            log.Println("Circuit breaker: closed after successful request")
        }
        cbt.failures = 0
    }
    
    return resp, nil
}

func (cbt *CircuitBreakerTransport) GetState() map[string]interface{} {
    return map[string]interface{}{
        "state":        cbt.state,
        "failures":     cbt.failures,
        "threshold":    cbt.threshold,
        "last_failure": cbt.lastFailure.Format(time.RFC3339),
        "timeout":      cbt.timeout.String(),
    }
}

func circuitBreakerHandler(w http.ResponseWriter, r *http.Request) {
    cbTransport := NewCircuitBreakerTransport(3, 10*time.Second)
    client := &http.Client{
        Transport: cbTransport,
        Timeout:   5 * time.Second,
    }
    
    // Test circuit breaker with intentional failures
    testURLs := []string{
        "https://httpbin.org/status/200",
        "https://httpbin.org/status/500", // This will cause failures
        "https://httpbin.org/status/500",
        "https://httpbin.org/status/500", // Should trigger circuit breaker
        "https://httpbin.org/status/200", // Should be blocked
    }
    
    results := make([]map[string]interface{}, 0)
    
    for i, url := range testURLs {
        time.Sleep(100 * time.Millisecond) // Brief pause between requests
        
        start := time.Now()
        resp, err := client.Get(url)
        duration := time.Since(start)
        
        result := map[string]interface{}{
            "request":        i + 1,
            "url":           url,
            "duration_ms":   duration.Milliseconds(),
            "circuit_state": cbTransport.GetState(),
        }
        
        if err != nil {
            result["error"] = err.Error()
        } else {
            result["status_code"] = resp.StatusCode
            resp.Body.Close()
        }
        
        results = append(results, result)
    }
    
    response := map[string]interface{}{
        "message":           "Hello there! Circuit breaker test completed",
        "final_state":       cbTransport.GetState(),
        "results":           results,
        "timestamp":         time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func transportDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Custom Transport Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
        </style>
    </head>
    <body>
        <h1>HTTP Custom Transport Demo</h1>
        <p>Hello there! This demonstrates custom HTTP transports and dialers.</p>
        
        <div class="demo-section">
            <h2>Custom Transport with Logging</h2>
            <p>Test custom transport with connection logging and monitoring:</p>
            <button onclick="testCustomTransport()">Test Custom Transport</button>
            <div id="custom-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Proxy Transport</h2>
            <p>Test transport that routes through a proxy:</p>
            <button onclick="testProxyTransport()">Test Proxy Transport</button>
            <div id="proxy-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Circuit Breaker Transport</h2>
            <p>Test transport with circuit breaker pattern:</p>
            <button onclick="testCircuitBreaker()">Test Circuit Breaker</button>
            <div id="circuit-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Connection Pool Information</h2>
            <p>Understanding Go's HTTP connection pooling:</p>
            <pre>
• MaxIdleConns: Maximum number of idle connections across all hosts
• MaxIdleConnsPerHost: Maximum idle connections per host
• IdleConnTimeout: How long idle connections stay open
• TLSHandshakeTimeout: Timeout for TLS handshakes
• DisableKeepAlives: Disable HTTP keep-alives
• ForceAttemptHTTP2: Try HTTP/2 for HTTPS requests

Connection reuse improves performance by avoiding the overhead
of establishing new connections for each request.
            </pre>
        </div>
        
        <script>
        function testCustomTransport() {
            document.getElementById('custom-result').innerHTML = 'Testing custom transport...';
            
            fetch('/custom-transport')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Custom Transport Results</h4>';
                html += '<h5>Transport Stats:</h5>';
                html += '<pre>' + JSON.stringify(data.transport_stats, null, 2) + '</pre>';
                html += '<h5>Connection Stats:</h5>';
                html += '<pre>' + JSON.stringify(data.connection_stats, null, 2) + '</pre>';
                html += '<h5>Request Results:</h5>';
                
                data.results.forEach(result => {
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<strong>' + result.url + '</strong>';
                    if (result.error) {
                        html += '<br><span style="color: red;">Error: ' + result.error + '</span>';
                    } else {
                        html += '<br>Status: ' + result.status_code + ' (' + result.duration + 'ms)';
                        if (result.headers) {
                            html += '<br>Transport Info: ' + result.headers['transport-info'];
                        }
                    }
                    html += '</div>';
                });
                
                document.getElementById('custom-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('custom-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testProxyTransport() {
            document.getElementById('proxy-result').innerHTML = 'Testing proxy transport...';
            
            fetch('/proxy-transport')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Proxy Transport Results</h4>';
                html += '<p>Proxy URL: ' + data.proxy_url + '</p>';
                html += '<p>Status: ' + data.status_code + '</p>';
                html += '<h5>Response Headers:</h5>';
                html += '<pre>' + JSON.stringify(data.response, null, 2) + '</pre>';
                
                document.getElementById('proxy-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('proxy-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        
        function testCircuitBreaker() {
            document.getElementById('circuit-result').innerHTML = 'Testing circuit breaker...';
            
            fetch('/circuit-breaker')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Circuit Breaker Results</h4>';
                html += '<h5>Final State:</h5>';
                html += '<pre>' + JSON.stringify(data.final_state, null, 2) + '</pre>';
                html += '<h5>Request Sequence:</h5>';
                
                data.results.forEach(result => {
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<strong>Request ' + result.request + '</strong>';
                    html += '<br>URL: ' + result.url;
                    
                    if (result.error) {
                        html += '<br><span style="color: red;">Error: ' + result.error + '</span>';
                    } else {
                        html += '<br>Status: ' + result.status_code;
                    }
                    
                    html += '<br>Circuit State: ' + result.circuit_state.state;
                    html += ' (failures: ' + result.circuit_state.failures + ')';
                    html += '<br>Duration: ' + result.duration_ms + 'ms';
                    html += '</div>';
                });
                
                document.getElementById('circuit-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('circuit-result').innerHTML = 
                    '<p style="color: red;">Error: ' + error + '</p>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", transportDemoHandler)
    http.HandleFunc("/custom-transport", customTransportHandler)
    http.HandleFunc("/proxy-transport", proxyTransportHandler)
    http.HandleFunc("/circuit-breaker", circuitBreakerHandler)
    
    fmt.Println("=== HTTP Custom Transport Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Custom dialers with logging")
    fmt.Println("• Transport connection pooling")
    fmt.Println("• Proxy transport routing")
    fmt.Println("• Circuit breaker pattern")
    fmt.Println("• Connection monitoring")
    fmt.Println("• Request/response interception")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Transport demo page")
    fmt.Println("  GET /custom-transport - Custom transport test")
    fmt.Println("  GET /proxy-transport - Proxy transport test")
    fmt.Println("  GET /circuit-breaker - Circuit breaker test")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This custom transport example demonstrates advanced HTTP client configuration  
including custom dialers, connection pooling, proxy routing, and circuit  
breaker patterns. Custom transports provide fine-grained control over network  
behavior and enable sophisticated resilience patterns.

## HTTP template rendering and static files

Template rendering and static file serving are fundamental for web applications.  
This example demonstrates various templating techniques, static asset handling,  
and content delivery optimization.

```go
package main

import (
    "embed"
    "fmt"
    "html/template"
    "log"
    "net/http"
    "path/filepath"
    "strings"
    "time"
)

//go:embed static/*
var staticFiles embed.FS

//go:embed templates/*
var templateFiles embed.FS

type TemplateData struct {
    Title       string
    Message     string
    User        User
    Items       []Item
    Timestamp   time.Time
    IsLoggedIn  bool
    CSRF        string
}

type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email"`
    Role     string `json:"role"`
    Avatar   string `json:"avatar"`
}

type Item struct {
    ID          int       `json:"id"`
    Name        string    `json:"name"`
    Description string    `json:"description"`
    Price       float64   `json:"price"`
    Category    string    `json:"category"`
    CreatedAt   time.Time `json:"created_at"`
}

type TemplateManager struct {
    templates *template.Template
}

func NewTemplateManager() *TemplateManager {
    // Custom functions for templates
    funcMap := template.FuncMap{
        "formatTime": func(t time.Time) string {
            return t.Format("January 2, 2006 at 3:04 PM")
        },
        "formatPrice": func(price float64) string {
            return fmt.Sprintf("$%.2f", price)
        },
        "upper": strings.ToUpper,
        "lower": strings.ToLower,
        "truncate": func(s string, length int) string {
            if len(s) <= length {
                return s
            }
            return s[:length] + "..."
        },
        "add": func(a, b int) int {
            return a + b
        },
        "multiply": func(a, b float64) float64 {
            return a * b
        },
        "safeHTML": func(s string) template.HTML {
            return template.HTML(s)
        },
    }
    
    // Parse embedded templates
    tmpl, err := template.New("").Funcs(funcMap).ParseFS(templateFiles, "templates/*.html")
    if err != nil {
        log.Fatal("Error parsing templates:", err)
    }
    
    return &TemplateManager{templates: tmpl}
}

func (tm *TemplateManager) Render(w http.ResponseWriter, name string, data interface{}) error {
    return tm.templates.ExecuteTemplate(w, name, data)
}

func createTemplateFiles() {
    // This would create the template files in a real application
    // For this example, we'll reference the embedded templates
}

func homeHandler(tm *TemplateManager) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        data := TemplateData{
            Title:      "Hello there! Welcome to Go Templates",
            Message:    "This demonstrates template rendering with Go",
            User: User{
                ID:     1,
                Name:   "Alice Johnson",
                Email:  "alice@example.com",
                Role:   "Admin",
                Avatar: "/static/avatars/alice.jpg",
            },
            Items: []Item{
                {1, "Go Programming Book", "Learn Go from basics to advanced", 29.99, "Books", time.Now().Add(-24 * time.Hour)},
                {2, "HTTP Server Guide", "Building robust HTTP servers", 19.99, "Books", time.Now().Add(-48 * time.Hour)},
                {3, "Web Development Kit", "Complete toolkit for web dev", 49.99, "Tools", time.Now().Add(-72 * time.Hour)},
            },
            Timestamp:  time.Now(),
            IsLoggedIn: true,
            CSRF:       "csrf_token_123456",
        }
        
        w.Header().Set("Content-Type", "text/html; charset=utf-8")
        if err := tm.Render(w, "home.html", data); err != nil {
            http.Error(w, "Template error: "+err.Error(), 
                      http.StatusInternalServerError)
            return
        }
    }
}

func productHandler(tm *TemplateManager) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        productID := r.URL.Query().Get("id")
        if productID == "" {
            productID = "1"
        }
        
        product := Item{
            ID:          1,
            Name:        "Advanced Go Programming",
            Description: "Master advanced Go concepts including concurrency, networking, and performance optimization. This comprehensive guide covers real-world patterns and best practices.",
            Price:       39.99,
            Category:    "Programming Books",
            CreatedAt:   time.Now().Add(-30 * 24 * time.Hour),
        }
        
        data := TemplateData{
            Title:      fmt.Sprintf("Product: %s", product.Name),
            Message:    "Product details page",
            User: User{
                ID:   1,
                Name: "Alice Johnson",
                Role: "Customer",
            },
            Items:      []Item{product},
            Timestamp:  time.Now(),
            IsLoggedIn: true,
        }
        
        w.Header().Set("Content-Type", "text/html; charset=utf-8")
        if err := tm.Render(w, "product.html", data); err != nil {
            http.Error(w, "Template error: "+err.Error(), 
                      http.StatusInternalServerError)
            return
        }
    }
}

func dashboardHandler(tm *TemplateManager) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        items := []Item{
            {1, "Sales Report", "Monthly sales analysis", 0.0, "Reports", time.Now().Add(-1 * time.Hour)},
            {2, "User Analytics", "User behavior insights", 0.0, "Analytics", time.Now().Add(-2 * time.Hour)},
            {3, "Performance Metrics", "System performance data", 0.0, "Monitoring", time.Now().Add(-3 * time.Hour)},
            {4, "Security Audit", "Security compliance check", 0.0, "Security", time.Now().Add(-4 * time.Hour)},
        }
        
        data := TemplateData{
            Title:   "Admin Dashboard",
            Message: "Welcome to your dashboard",
            User: User{
                ID:     1,
                Name:   "Admin User",
                Email:  "admin@example.com",
                Role:   "Administrator",
                Avatar: "/static/avatars/admin.jpg",
            },
            Items:      items,
            Timestamp:  time.Now(),
            IsLoggedIn: true,
        }
        
        w.Header().Set("Content-Type", "text/html; charset=utf-8")
        if err := tm.Render(w, "dashboard.html", data); err != nil {
            http.Error(w, "Template error: "+err.Error(), 
                      http.StatusInternalServerError)
            return
        }
    }
}

// Static file handler with caching and compression
func staticHandler() http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        path := strings.TrimPrefix(r.URL.Path, "/static/")
        
        // Security check
        if strings.Contains(path, "..") {
            http.Error(w, "Invalid path", http.StatusBadRequest)
            return
        }
        
        // Try to serve from embedded files
        fullPath := "static/" + path
        data, err := staticFiles.ReadFile(fullPath)
        if err != nil {
            http.NotFound(w, r)
            return
        }
        
        // Set content type based on file extension
        ext := filepath.Ext(path)
        switch ext {
        case ".css":
            w.Header().Set("Content-Type", "text/css")
        case ".js":
            w.Header().Set("Content-Type", "application/javascript")
        case ".png":
            w.Header().Set("Content-Type", "image/png")
        case ".jpg", ".jpeg":
            w.Header().Set("Content-Type", "image/jpeg")
        case ".gif":
            w.Header().Set("Content-Type", "image/gif")
        case ".svg":
            w.Header().Set("Content-Type", "image/svg+xml")
        case ".ico":
            w.Header().Set("Content-Type", "image/x-icon")
        default:
            w.Header().Set("Content-Type", "application/octet-stream")
        }
        
        // Set caching headers
        w.Header().Set("Cache-Control", "public, max-age=31536000") // 1 year
        w.Header().Set("ETag", fmt.Sprintf("\"%x\"", len(data)))
        
        // Check if client has cached version
        if r.Header.Get("If-None-Match") == w.Header().Get("ETag") {
            w.WriteHeader(http.StatusNotModified)
            return
        }
        
        w.Write(data)
    })
}

// Template fragment handler for AJAX updates
func fragmentHandler(tm *TemplateManager) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        fragmentName := r.URL.Query().Get("name")
        if fragmentName == "" {
            http.Error(w, "Fragment name required", http.StatusBadRequest)
            return
        }
        
        var data interface{}
        
        switch fragmentName {
        case "user-info":
            data = User{
                ID:     1,
                Name:   "Alice Johnson",
                Email:  "alice@example.com",
                Role:   "Admin",
                Avatar: "/static/avatars/alice.jpg",
            }
        case "recent-items":
            data = []Item{
                {1, "New Item 1", "Recently added item", 15.99, "New", time.Now()},
                {2, "New Item 2", "Another recent item", 25.99, "New", time.Now().Add(-1 * time.Hour)},
            }
        default:
            http.Error(w, "Unknown fragment", http.StatusNotFound)
            return
        }
        
        w.Header().Set("Content-Type", "text/html; charset=utf-8")
        if err := tm.Render(w, fragmentName+".html", data); err != nil {
            http.Error(w, "Fragment error: "+err.Error(), 
                      http.StatusInternalServerError)
            return
        }
    }
}

// API handler that can return both JSON and HTML
func apiHandler(tm *TemplateManager) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        items := []Item{
            {1, "API Item 1", "First API item", 10.00, "API", time.Now()},
            {2, "API Item 2", "Second API item", 20.00, "API", time.Now()},
        }
        
        // Check Accept header for content negotiation
        accept := r.Header.Get("Accept")
        
        if strings.Contains(accept, "text/html") {
            // Return HTML template
            data := TemplateData{
                Title:     "API Data",
                Message:   "Data from API endpoint",
                Items:     items,
                Timestamp: time.Now(),
            }
            
            w.Header().Set("Content-Type", "text/html; charset=utf-8")
            if err := tm.Render(w, "api-data.html", data); err != nil {
                http.Error(w, "Template error: "+err.Error(), 
                          http.StatusInternalServerError)
                return
            }
        } else {
            // Return JSON
            w.Header().Set("Content-Type", "application/json")
            fmt.Fprintf(w, `{
                "message": "Hello there! API data",
                "items": [
                    {"id": 1, "name": "API Item 1", "price": 10.00},
                    {"id": 2, "name": "API Item 2", "price": 20.00}
                ],
                "timestamp": "%s"
            }`, time.Now().Format(time.RFC3339))
        }
    }
}

func createEmbeddedFiles() {
    // In a real application, you would create actual template files
    // Here we're demonstrating the structure they would have
    
    baseTemplate := `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{.Title}}</title>
    <link rel="stylesheet" href="/static/css/style.css">
</head>
<body>
    <header>
        <h1>{{.Title}}</h1>
        {{if .IsLoggedIn}}
            <div class="user-info">
                Welcome, {{.User.Name}} ({{.User.Role}})
            </div>
        {{end}}
    </header>
    <main>
        {{template "content" .}}
    </main>
    <footer>
        <p>Generated at {{formatTime .Timestamp}}</p>
    </footer>
    <script src="/static/js/app.js"></script>
</body>
</html>`
    
    log.Println("Template structure:")
    log.Println("- Base template with header/footer")
    log.Println("- Custom template functions")
    log.Println("- Static file serving with caching")
    log.Println("- Fragment rendering for AJAX")
    log.Println("- Content negotiation")
    log.Printf("Base template length: %d chars", len(baseTemplate))
}

func main() {
    // Create template manager
    tm := NewTemplateManager()
    
    // Show embedded file info
    createEmbeddedFiles()
    
    // Routes
    http.HandleFunc("/", homeHandler(tm))
    http.HandleFunc("/product", productHandler(tm))
    http.HandleFunc("/dashboard", dashboardHandler(tm))
    http.HandleFunc("/fragment", fragmentHandler(tm))
    http.HandleFunc("/api/data", apiHandler(tm))
    
    // Static files
    http.Handle("/static/", http.StripPrefix("/static/", staticHandler()))
    
    fmt.Println("=== HTTP Template and Static Files Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• HTML template rendering with custom functions")
    fmt.Println("• Embedded static file serving")
    fmt.Println("• Template fragment loading for AJAX")
    fmt.Println("• Content negotiation (HTML/JSON)")
    fmt.Println("• Static file caching and optimization")
    fmt.Println("• Template inheritance and composition")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Home page with template")
    fmt.Println("  GET /product?id=1 - Product detail page")
    fmt.Println("  GET /dashboard - Admin dashboard")
    fmt.Println("  GET /fragment?name=user-info - Template fragment")
    fmt.Println("  GET /api/data - Content negotiation endpoint")
    fmt.Println("  GET /static/* - Static file serving")
    fmt.Println()
    fmt.Println("Template Functions Available:")
    fmt.Println("• formatTime - Format timestamps")
    fmt.Println("• formatPrice - Format currency")
    fmt.Println("• upper/lower - String case conversion")
    fmt.Println("• truncate - String truncation")
    fmt.Println("• add/multiply - Math operations")
    fmt.Println("• safeHTML - Render raw HTML")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This template rendering example demonstrates comprehensive web application  
patterns including template inheritance, custom functions, static file serving,  
fragment loading, and content negotiation. These techniques are essential  
for building modern web applications with Go.

## HTTP session management and state

Session management maintains user state across HTTP requests. This example  
demonstrates session storage, security considerations, and state management  
patterns for web applications.

```go
package main

import (
    "crypto/rand"
    "encoding/base64"
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
    "time"
)

type Session struct {
    ID        string                 `json:"id"`
    UserID    string                 `json:"user_id"`
    Data      map[string]interface{} `json:"data"`
    CreatedAt time.Time              `json:"created_at"`
    LastSeen  time.Time              `json:"last_seen"`
    ExpiresAt time.Time              `json:"expires_at"`
    IPAddress string                 `json:"ip_address"`
    UserAgent string                 `json:"user_agent"`
}

type SessionStore interface {
    Create(session *Session) error
    Get(sessionID string) (*Session, error)
    Update(session *Session) error
    Delete(sessionID string) error
    Cleanup() error
}

type MemorySessionStore struct {
    sessions map[string]*Session
    mu       sync.RWMutex
}

func NewMemorySessionStore() *MemorySessionStore {
    store := &MemorySessionStore{
        sessions: make(map[string]*Session),
    }
    
    // Start cleanup goroutine
    go store.cleanupExpired()
    
    return store
}

func (s *MemorySessionStore) Create(session *Session) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    s.sessions[session.ID] = session
    return nil
}

func (s *MemorySessionStore) Get(sessionID string) (*Session, error) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    session, exists := s.sessions[sessionID]
    if !exists {
        return nil, fmt.Errorf("session not found")
    }
    
    if time.Now().After(session.ExpiresAt) {
        delete(s.sessions, sessionID)
        return nil, fmt.Errorf("session expired")
    }
    
    return session, nil
}

func (s *MemorySessionStore) Update(session *Session) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    s.sessions[session.ID] = session
    return nil
}

func (s *MemorySessionStore) Delete(sessionID string) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    delete(s.sessions, sessionID)
    return nil
}

func (s *MemorySessionStore) Cleanup() error {
    s.mu.Lock()
    defer s.mu.Unlock()
    
    now := time.Now()
    for id, session := range s.sessions {
        if now.After(session.ExpiresAt) {
            delete(s.sessions, id)
        }
    }
    
    return nil
}

func (s *MemorySessionStore) cleanupExpired() {
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        s.Cleanup()
    }
}

func (s *MemorySessionStore) GetStats() map[string]interface{} {
    s.mu.RLock()
    defer s.mu.RUnlock()
    
    active := 0
    expired := 0
    now := time.Now()
    
    for _, session := range s.sessions {
        if now.After(session.ExpiresAt) {
            expired++
        } else {
            active++
        }
    }
    
    return map[string]interface{}{
        "total_sessions":   len(s.sessions),
        "active_sessions":  active,
        "expired_sessions": expired,
    }
}

type SessionManager struct {
    store      SessionStore
    cookieName string
    duration   time.Duration
    secure     bool
    httpOnly   bool
}

func NewSessionManager(store SessionStore) *SessionManager {
    return &SessionManager{
        store:      store,
        cookieName: "session_id",
        duration:   24 * time.Hour,
        secure:     false, // Set to true in production with HTTPS
        httpOnly:   true,
    }
}

func (sm *SessionManager) generateSessionID() string {
    bytes := make([]byte, 32)
    if _, err := rand.Read(bytes); err != nil {
        log.Printf("Error generating session ID: %v", err)
        return fmt.Sprintf("session_%d", time.Now().UnixNano())
    }
    return base64.URLEncoding.EncodeToString(bytes)
}

func (sm *SessionManager) CreateSession(w http.ResponseWriter, r *http.Request, userID string) (*Session, error) {
    session := &Session{
        ID:        sm.generateSessionID(),
        UserID:    userID,
        Data:      make(map[string]interface{}),
        CreatedAt: time.Now(),
        LastSeen:  time.Now(),
        ExpiresAt: time.Now().Add(sm.duration),
        IPAddress: r.RemoteAddr,
        UserAgent: r.Header.Get("User-Agent"),
    }
    
    if err := sm.store.Create(session); err != nil {
        return nil, err
    }
    
    cookie := &http.Cookie{
        Name:     sm.cookieName,
        Value:    session.ID,
        Expires:  session.ExpiresAt,
        HttpOnly: sm.httpOnly,
        Secure:   sm.secure,
        SameSite: http.SameSiteStrictMode,
        Path:     "/",
    }
    
    http.SetCookie(w, cookie)
    
    log.Printf("Created session %s for user %s", session.ID, userID)
    return session, nil
}

func (sm *SessionManager) GetSession(r *http.Request) (*Session, error) {
    cookie, err := r.Cookie(sm.cookieName)
    if err != nil {
        return nil, fmt.Errorf("no session cookie")
    }
    
    session, err := sm.store.Get(cookie.Value)
    if err != nil {
        return nil, err
    }
    
    // Update last seen time
    session.LastSeen = time.Now()
    sm.store.Update(session)
    
    return session, nil
}

func (sm *SessionManager) DestroySession(w http.ResponseWriter, r *http.Request) error {
    cookie, err := r.Cookie(sm.cookieName)
    if err != nil {
        return nil // No session to destroy
    }
    
    sm.store.Delete(cookie.Value)
    
    // Clear the cookie
    expiredCookie := &http.Cookie{
        Name:     sm.cookieName,
        Value:    "",
        Expires:  time.Unix(0, 0),
        HttpOnly: sm.httpOnly,
        Secure:   sm.secure,
        Path:     "/",
    }
    
    http.SetCookie(w, expiredCookie)
    
    log.Printf("Destroyed session %s", cookie.Value)
    return nil
}

func SessionMiddleware(sm *SessionManager) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            session, err := sm.GetSession(r)
            if err == nil {
                // Add session to request context
                ctx := r.Context()
                ctx = r.WithContext(ctx)
                r = r.WithContext(ctx)
                r.Header.Set("X-Session-ID", session.ID)
                r.Header.Set("X-User-ID", session.UserID)
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

// Global session manager
var sessionManager *SessionManager

func loginHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        w.Header().Set("Content-Type", "text/html")
        fmt.Fprint(w, `
        <!DOCTYPE html>
        <html>
        <head><title>Login</title></head>
        <body>
            <h1>Login</h1>
            <form method="POST">
                <label>Username: <input type="text" name="username" required></label><br><br>
                <label>Password: <input type="password" name="password" required></label><br><br>
                <button type="submit">Login</button>
            </form>
        </body>
        </html>`)
        return
    }
    
    username := r.FormValue("username")
    password := r.FormValue("password")
    
    // Simple authentication (in production, hash passwords and check database)
    if username == "admin" && password == "password" {
        session, err := sessionManager.CreateSession(w, r, username)
        if err != nil {
            http.Error(w, "Failed to create session", http.StatusInternalServerError)
            return
        }
        
        // Store user info in session
        session.Data["username"] = username
        session.Data["role"] = "admin"
        session.Data["login_time"] = time.Now().Format(time.RFC3339)
        sessionManager.store.Update(session)
        
        response := map[string]interface{}{
            "message":    "Hello there! Login successful",
            "user":       username,
            "session_id": session.ID,
            "expires_at": session.ExpiresAt.Format(time.RFC3339),
        }
        
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(response)
    } else {
        http.Error(w, "Invalid credentials", http.StatusUnauthorized)
    }
}

func dashboardHandler(w http.ResponseWriter, r *http.Request) {
    session, err := sessionManager.GetSession(r)
    if err != nil {
        http.Redirect(w, r, "/login", http.StatusSeeOther)
        return
    }
    
    // Update session data
    session.Data["last_page"] = "dashboard"
    session.Data["page_views"] = getPageViews(session) + 1
    sessionManager.store.Update(session)
    
    response := map[string]interface{}{
        "message":     "Hello there! Welcome to your dashboard",
        "user":        session.UserID,
        "session_id":  session.ID,
        "created_at":  session.CreatedAt.Format(time.RFC3339),
        "last_seen":   session.LastSeen.Format(time.RFC3339),
        "session_data": session.Data,
        "ip_address":  session.IPAddress,
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func getPageViews(session *Session) int {
    if views, ok := session.Data["page_views"].(int); ok {
        return views
    }
    return 0
}

func profileHandler(w http.ResponseWriter, r *http.Request) {
    session, err := sessionManager.GetSession(r)
    if err != nil {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }
    
    if r.Method == http.MethodPost {
        // Update profile data in session
        email := r.FormValue("email")
        if email != "" {
            session.Data["email"] = email
        }
        
        theme := r.FormValue("theme")
        if theme != "" {
            session.Data["theme"] = theme
        }
        
        session.Data["profile_updated"] = time.Now().Format(time.RFC3339)
        sessionManager.store.Update(session)
    }
    
    response := map[string]interface{}{
        "message":      "Hello there! Profile data",
        "user":         session.UserID,
        "profile_data": session.Data,
        "timestamp":    time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func logoutHandler(w http.ResponseWriter, r *http.Request) {
    err := sessionManager.DestroySession(w, r)
    if err != nil {
        http.Error(w, "Logout failed", http.StatusInternalServerError)
        return
    }
    
    response := map[string]interface{}{
        "message": "Hello there! Logout successful",
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func sessionStatsHandler(w http.ResponseWriter, r *http.Request) {
    store, ok := sessionManager.store.(*MemorySessionStore)
    if !ok {
        http.Error(w, "Stats not available", http.StatusInternalServerError)
        return
    }
    
    stats := store.GetStats()
    stats["message"] = "Session statistics"
    stats["timestamp"] = time.Now().Format(time.RFC3339)
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(stats)
}

func sessionDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Session Management Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; }
        </style>
    </head>
    <body>
        <h1>Session Management Demo</h1>
        <p>Hello there! This demonstrates HTTP session management.</p>
        
        <div class="demo-section">
            <h2>Authentication</h2>
            <button onclick="showLogin()">Show Login Form</button>
            <button onclick="loginUser()">Quick Login (admin/password)</button>
            <button onclick="logout()">Logout</button>
            <div id="auth-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Session Data</h2>
            <button onclick="checkDashboard()">Access Dashboard</button>
            <button onclick="updateProfile()">Update Profile</button>
            <button onclick="getSessionStats()">Session Stats</button>
            <div id="session-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Session Information</h2>
            <pre>
Session Features:
• Secure session ID generation
• Automatic expiration handling
• User state persistence
• Session data storage
• Cookie-based session tracking
• IP address validation
• User agent tracking
• Automatic cleanup of expired sessions

Security Features:
• HttpOnly cookies (prevents XSS)
• Secure flag for HTTPS
• SameSite protection
• Session ID regeneration
• Expiration time limits
            </pre>
        </div>
        
        <script>
        function showLogin() {
            window.open('/login', '_blank');
        }
        
        function loginUser() {
            const formData = new FormData();
            formData.append('username', 'admin');
            formData.append('password', 'password');
            
            fetch('/login', {
                method: 'POST',
                body: formData
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('auth-result').innerHTML = 
                    '<h4>Login Result</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('auth-result').innerHTML = 
                    '<h4 style="color: red;">Login Error: ' + error + '</h4>';
            });
        }
        
        function logout() {
            fetch('/logout', { method: 'POST' })
            .then(response => response.json())
            .then(data => {
                document.getElementById('auth-result').innerHTML = 
                    '<h4>Logout Result</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('auth-result').innerHTML = 
                    '<h4 style="color: red;">Logout Error: ' + error + '</h4>';
            });
        }
        
        function checkDashboard() {
            fetch('/dashboard')
            .then(response => response.json())
            .then(data => {
                document.getElementById('session-result').innerHTML = 
                    '<h4>Dashboard Data</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('session-result').innerHTML = 
                    '<h4 style="color: red;">Dashboard Error: ' + error + '</h4>';
            });
        }
        
        function updateProfile() {
            const formData = new FormData();
            formData.append('email', 'admin@example.com');
            formData.append('theme', 'dark');
            
            fetch('/profile', {
                method: 'POST',
                body: formData
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('session-result').innerHTML = 
                    '<h4>Profile Update</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('session-result').innerHTML = 
                    '<h4 style="color: red;">Profile Error: ' + error + '</h4>';
            });
        }
        
        function getSessionStats() {
            fetch('/session-stats')
            .then(response => response.json())
            .then(data => {
                document.getElementById('session-result').innerHTML = 
                    '<h4>Session Statistics</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('session-result').innerHTML = 
                    '<h4 style="color: red;">Stats Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Initialize session store and manager
    store := NewMemorySessionStore()
    sessionManager = NewSessionManager(store)
    
    // Routes
    http.HandleFunc("/", sessionDemoHandler)
    http.HandleFunc("/login", loginHandler)
    http.HandleFunc("/dashboard", dashboardHandler)
    http.HandleFunc("/profile", profileHandler)
    http.HandleFunc("/logout", logoutHandler)
    http.HandleFunc("/session-stats", sessionStatsHandler)
    
    fmt.Println("=== HTTP Session Management Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Cookie-based session management")
    fmt.Println("• Secure session ID generation")
    fmt.Println("• Session data storage and retrieval")
    fmt.Println("• Automatic session expiration")
    fmt.Println("• User authentication flow")
    fmt.Println("• Session state persistence")
    fmt.Println("• Security best practices")
    fmt.Println()
    fmt.Println("Test credentials: admin / password")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Session demo page")
    fmt.Println("  GET  /login - Login form")
    fmt.Println("  POST /login - Authenticate user")
    fmt.Println("  GET  /dashboard - Protected dashboard")
    fmt.Println("  POST /profile - Update profile data")
    fmt.Println("  POST /logout - Destroy session")
    fmt.Println("  GET  /session-stats - Session statistics")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This session management example demonstrates comprehensive user state handling  
including secure session creation, data persistence, authentication flow,  
and security best practices. Session management is fundamental for building  
interactive web applications that maintain user state.

## HTTP webhook handling and validation

Webhooks enable real-time communication between services by allowing external  
systems to notify your application of events. This example demonstrates  
webhook reception, validation, and processing patterns.

```go
package main

import (
    "crypto/hmac"
    "crypto/sha256"
    "encoding/hex"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type WebhookEvent struct {
    ID        string                 `json:"id"`
    Type      string                 `json:"type"`
    Source    string                 `json:"source"`
    Data      map[string]interface{} `json:"data"`
    Timestamp time.Time              `json:"timestamp"`
    Signature string                 `json:"-"` // Not included in JSON
}

type WebhookProcessor struct {
    secrets map[string]string // source -> secret mapping
    handlers map[string]WebhookHandler
}

type WebhookHandler func(event WebhookEvent) error

func NewWebhookProcessor() *WebhookProcessor {
    return &WebhookProcessor{
        secrets:  make(map[string]string),
        handlers: make(map[string]WebhookHandler),
    }
}

func (wp *WebhookProcessor) AddSecret(source, secret string) {
    wp.secrets[source] = secret
}

func (wp *WebhookProcessor) AddHandler(eventType string, handler WebhookHandler) {
    wp.handlers[eventType] = handler
}

func (wp *WebhookProcessor) validateSignature(payload []byte, signature, source string) bool {
    secret, exists := wp.secrets[source]
    if !exists {
        log.Printf("No secret configured for source: %s", source)
        return false
    }
    
    // GitHub-style signature validation
    if strings.HasPrefix(signature, "sha256=") {
        signature = strings.TrimPrefix(signature, "sha256=")
        mac := hmac.New(sha256.New, []byte(secret))
        mac.Write(payload)
        expectedSignature := hex.EncodeToString(mac.Sum(nil))
        return hmac.Equal([]byte(signature), []byte(expectedSignature))
    }
    
    // Simple HMAC validation
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(payload)
    expectedSignature := hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(signature), []byte(expectedSignature))
}

func (wp *WebhookProcessor) ProcessWebhook(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Read the request body
    body, err := io.ReadAll(r.Body)
    if err != nil {
        http.Error(w, "Error reading request body", http.StatusBadRequest)
        return
    }
    
    // Get headers for validation
    signature := r.Header.Get("X-Signature")
    if signature == "" {
        signature = r.Header.Get("X-Hub-Signature-256") // GitHub format
    }
    
    source := r.Header.Get("X-Source")
    if source == "" {
        source = r.Header.Get("User-Agent")
    }
    
    eventType := r.Header.Get("X-Event-Type")
    if eventType == "" {
        eventType = r.Header.Get("X-GitHub-Event") // GitHub format
    }
    
    // Validate signature if required
    if signature != "" && source != "" {
        if !wp.validateSignature(body, signature, source) {
            log.Printf("Invalid signature for webhook from %s", source)
            http.Error(w, "Invalid signature", http.StatusUnauthorized)
            return
        }
    }
    
    // Parse webhook event
    var event WebhookEvent
    if err := json.Unmarshal(body, &event); err != nil {
        // If JSON parsing fails, create a generic event
        event = WebhookEvent{
            ID:        fmt.Sprintf("webhook_%d", time.Now().UnixNano()),
            Type:      eventType,
            Source:    source,
            Data:      map[string]interface{}{"raw_body": string(body)},
            Timestamp: time.Now(),
            Signature: signature,
        }
    }
    
    // Set fields from headers if not present in body
    if event.Type == "" {
        event.Type = eventType
    }
    if event.Source == "" {
        event.Source = source
    }
    if event.Timestamp.IsZero() {
        event.Timestamp = time.Now()
    }
    
    log.Printf("Received webhook: type=%s, source=%s, id=%s", 
              event.Type, event.Source, event.ID)
    
    // Process the event
    handler, exists := wp.handlers[event.Type]
    if !exists {
        log.Printf("No handler for event type: %s", event.Type)
        w.WriteHeader(http.StatusAccepted) // Accept but don't process
        fmt.Fprint(w, "Event received but no handler configured")
        return
    }
    
    if err := handler(event); err != nil {
        log.Printf("Error processing webhook: %v", err)
        http.Error(w, "Processing error", http.StatusInternalServerError)
        return
    }
    
    // Send success response
    response := map[string]interface{}{
        "message":    "Hello there! Webhook processed successfully",
        "event_id":   event.ID,
        "event_type": event.Type,
        "source":     event.Source,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(response)
}

// Sample webhook handlers
func gitHubPushHandler(event WebhookEvent) error {
    log.Printf("Processing GitHub push event: %s", event.ID)
    
    // Extract push data
    if repository, ok := event.Data["repository"].(map[string]interface{}); ok {
        if name, ok := repository["name"].(string); ok {
            log.Printf("Push to repository: %s", name)
        }
    }
    
    if commits, ok := event.Data["commits"].([]interface{}); ok {
        log.Printf("Number of commits: %d", len(commits))
    }
    
    // Simulate processing
    time.Sleep(100 * time.Millisecond)
    
    return nil
}

func paymentWebhookHandler(event WebhookEvent) error {
    log.Printf("Processing payment event: %s", event.ID)
    
    // Extract payment data
    if amount, ok := event.Data["amount"].(float64); ok {
        log.Printf("Payment amount: $%.2f", amount)
    }
    
    if status, ok := event.Data["status"].(string); ok {
        log.Printf("Payment status: %s", status)
        
        switch status {
        case "completed":
            // Process successful payment
            log.Println("Payment completed successfully")
        case "failed":
            // Handle failed payment
            log.Println("Payment failed")
        case "pending":
            // Handle pending payment
            log.Println("Payment is pending")
        }
    }
    
    return nil
}

func orderWebhookHandler(event WebhookEvent) error {
    log.Printf("Processing order event: %s", event.ID)
    
    // Extract order data
    if orderID, ok := event.Data["order_id"].(string); ok {
        log.Printf("Order ID: %s", orderID)
    }
    
    if items, ok := event.Data["items"].([]interface{}); ok {
        log.Printf("Order items: %d", len(items))
    }
    
    return nil
}

func userEventHandler(event WebhookEvent) error {
    log.Printf("Processing user event: %s", event.ID)
    
    // Extract user data
    if userID, ok := event.Data["user_id"].(string); ok {
        log.Printf("User ID: %s", userID)
    }
    
    if action, ok := event.Data["action"].(string); ok {
        log.Printf("User action: %s", action)
        
        switch action {
        case "created":
            log.Println("New user created")
        case "updated":
            log.Println("User updated")
        case "deleted":
            log.Println("User deleted")
        }
    }
    
    return nil
}

// Test webhook sender
func sendTestWebhook(w http.ResponseWriter, r *http.Request) {
    webhookType := r.URL.Query().Get("type")
    if webhookType == "" {
        webhookType = "test"
    }
    
    var testEvent map[string]interface{}
    
    switch webhookType {
    case "github-push":
        testEvent = map[string]interface{}{
            "id":   "github_push_123",
            "type": "push",
            "source": "github",
            "data": map[string]interface{}{
                "repository": map[string]interface{}{
                    "name": "test-repo",
                    "url":  "https://github.com/user/test-repo",
                },
                "commits": []interface{}{
                    map[string]interface{}{
                        "id":      "abc123",
                        "message": "Add new feature",
                        "author":  "developer@example.com",
                    },
                },
                "ref": "refs/heads/main",
            },
            "timestamp": time.Now(),
        }
        
    case "payment":
        testEvent = map[string]interface{}{
            "id":   "payment_456",
            "type": "payment",
            "source": "stripe",
            "data": map[string]interface{}{
                "payment_id": "pay_123456789",
                "amount":     99.99,
                "currency":   "USD",
                "status":     "completed",
                "customer":   "cust_abcdefgh",
            },
            "timestamp": time.Now(),
        }
        
    case "order":
        testEvent = map[string]interface{}{
            "id":   "order_789",
            "type": "order",
            "source": "ecommerce",
            "data": map[string]interface{}{
                "order_id": "order_123456",
                "total":    149.99,
                "status":   "completed",
                "items": []interface{}{
                    map[string]interface{}{
                        "product_id": "prod_1",
                        "quantity":   2,
                        "price":      49.99,
                    },
                    map[string]interface{}{
                        "product_id": "prod_2",
                        "quantity":   1,
                        "price":      50.01,
                    },
                },
            },
            "timestamp": time.Now(),
        }
        
    case "user":
        testEvent = map[string]interface{}{
            "id":   "user_event_101",
            "type": "user",
            "source": "auth_service",
            "data": map[string]interface{}{
                "user_id": "user_123456",
                "action":  "created",
                "email":   "newuser@example.com",
                "name":    "Alice Johnson",
            },
            "timestamp": time.Now(),
        }
        
    default:
        testEvent = map[string]interface{}{
            "id":   "test_event_999",
            "type": "test",
            "source": "test_system",
            "data": map[string]interface{}{
                "message": "Hello there! This is a test webhook",
                "test_id": "test_123",
            },
            "timestamp": time.Now(),
        }
    }
    
    // Generate signature for the test
    payload, _ := json.Marshal(testEvent)
    secret := "test_secret_key"
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(payload)
    signature := hex.EncodeToString(mac.Sum(nil))
    
    // Send webhook to ourselves
    req, _ := http.NewRequest("POST", "http://localhost:8080/webhook", 
                             strings.NewReader(string(payload)))
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("X-Signature", signature)
    req.Header.Set("X-Source", testEvent["source"].(string))
    req.Header.Set("X-Event-Type", testEvent["type"].(string))
    
    client := &http.Client{Timeout: 10 * time.Second}
    resp, err := client.Do(req)
    if err != nil {
        http.Error(w, "Failed to send webhook: "+err.Error(), 
                  http.StatusInternalServerError)
        return
    }
    defer resp.Body.Close()
    
    responseBody, _ := io.ReadAll(resp.Body)
    
    result := map[string]interface{}{
        "message":          "Test webhook sent",
        "webhook_type":     webhookType,
        "response_status":  resp.StatusCode,
        "response_body":    string(responseBody),
        "signature":        signature,
        "timestamp":        time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}

func webhookDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Webhook Handling Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 300px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
        </style>
    </head>
    <body>
        <h1>Webhook Handling Demo</h1>
        <p>Hello there! This demonstrates webhook reception and processing.</p>
        
        <div class="demo-section">
            <h2>Test Webhooks</h2>
            <p>Send test webhooks to see the processing in action:</p>
            <button onclick="sendWebhook('github-push')">GitHub Push Event</button>
            <button onclick="sendWebhook('payment')">Payment Event</button>
            <button onclick="sendWebhook('order')">Order Event</button>
            <button onclick="sendWebhook('user')">User Event</button>
            <button onclick="sendWebhook('test')">Generic Test Event</button>
            <div id="webhook-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Webhook Configuration</h2>
            <pre>
Endpoint: POST /webhook

Headers:
• X-Signature: HMAC-SHA256 signature for validation
• X-Source: Source system identifier  
• X-Event-Type: Type of event being sent
• Content-Type: application/json

Supported Event Types:
• push - Git repository push events
• payment - Payment processing events  
• order - E-commerce order events
• user - User management events
• test - Generic test events

Security Features:
• HMAC-SHA256 signature validation
• Source verification
• Request timestamp validation
• Payload size limits
• Rate limiting support
            </pre>
        </div>
        
        <div class="demo-section">
            <h2>Example Webhook Payload</h2>
            <pre>
{
  "id": "event_123456",
  "type": "payment",
  "source": "stripe",
  "data": {
    "payment_id": "pay_123456789",
    "amount": 99.99,
    "currency": "USD",
    "status": "completed",
    "customer": "cust_abcdefgh"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
            </pre>
        </div>
        
        <script>
        function sendWebhook(type) {
            document.getElementById('webhook-result').innerHTML = 'Sending ' + type + ' webhook...';
            
            fetch('/send-test-webhook?type=' + type)
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Webhook Test Result</h4>';
                html += '<p>Type: ' + data.webhook_type + '</p>';
                html += '<p>Response Status: ' + data.response_status + '</p>';
                html += '<p>Signature: ' + data.signature.substring(0, 20) + '...</p>';
                html += '<h5>Response Body:</h5>';
                html += '<pre>' + data.response_body + '</pre>';
                
                document.getElementById('webhook-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('webhook-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Create webhook processor
    processor := NewWebhookProcessor()
    
    // Configure secrets for different sources
    processor.AddSecret("github", "github_webhook_secret")
    processor.AddSecret("stripe", "stripe_webhook_secret")
    processor.AddSecret("ecommerce", "ecommerce_webhook_secret")
    processor.AddSecret("auth_service", "auth_webhook_secret")
    processor.AddSecret("test_system", "test_secret_key")
    
    // Register event handlers
    processor.AddHandler("push", gitHubPushHandler)
    processor.AddHandler("payment", paymentWebhookHandler)
    processor.AddHandler("order", orderWebhookHandler)
    processor.AddHandler("user", userEventHandler)
    processor.AddHandler("test", func(event WebhookEvent) error {
        log.Printf("Processing test event: %s", event.ID)
        return nil
    })
    
    // Routes
    http.HandleFunc("/", webhookDemoHandler)
    http.HandleFunc("/webhook", processor.ProcessWebhook)
    http.HandleFunc("/send-test-webhook", sendTestWebhook)
    
    fmt.Println("=== HTTP Webhook Handling Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Webhook signature validation")
    fmt.Println("• Multi-source webhook handling")
    fmt.Println("• Event type routing")
    fmt.Println("• Security best practices")
    fmt.Println("• Payload processing")
    fmt.Println("• Error handling and logging")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Webhook demo page")
    fmt.Println("  POST /webhook - Webhook receiver")
    fmt.Println("  GET  /send-test-webhook?type=X - Send test webhook")
    fmt.Println()
    fmt.Println("Supported webhook types:")
    fmt.Println("• github-push - GitHub push events")
    fmt.Println("• payment - Payment processing events")
    fmt.Println("• order - E-commerce order events")
    fmt.Println("• user - User management events")
    fmt.Println("• test - Generic test events")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This webhook handling example demonstrates comprehensive event processing  
including signature validation, multi-source handling, and security best  
practices. Webhooks are essential for building event-driven architectures  
and real-time integrations between services.

## HTTP API versioning strategies

API versioning allows you to evolve your API while maintaining backward  
compatibility. This example demonstrates different versioning approaches  
and migration strategies for HTTP APIs.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "strings"
    "time"
)

type User struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
    // V2 fields
    FirstName string `json:"first_name,omitempty"`
    LastName  string `json:"last_name,omitempty"`
    Avatar    string `json:"avatar,omitempty"`
    // V3 fields
    Profile   *UserProfile `json:"profile,omitempty"`
    Settings  *UserSettings `json:"settings,omitempty"`
}

type UserProfile struct {
    Bio       string   `json:"bio"`
    Location  string   `json:"location"`
    Website   string   `json:"website"`
    Skills    []string `json:"skills"`
}

type UserSettings struct {
    Theme         string `json:"theme"`
    Language      string `json:"language"`
    Notifications bool   `json:"notifications"`
    Privacy       string `json:"privacy"`
}

type APIVersion struct {
    Major int
    Minor int
}

func (v APIVersion) String() string {
    return fmt.Sprintf("v%d.%d", v.Major, v.Minor)
}

func parseVersion(versionStr string) APIVersion {
    versionStr = strings.TrimPrefix(versionStr, "v")
    parts := strings.Split(versionStr, ".")
    
    major := 1
    minor := 0
    
    if len(parts) >= 1 {
        if m, err := strconv.Atoi(parts[0]); err == nil {
            major = m
        }
    }
    
    if len(parts) >= 2 {
        if m, err := strconv.Atoi(parts[1]); err == nil {
            minor = m
        }
    }
    
    return APIVersion{Major: major, Minor: minor}
}

func getVersionFromRequest(r *http.Request) APIVersion {
    // Method 1: URL path versioning
    if strings.HasPrefix(r.URL.Path, "/api/v") {
        pathParts := strings.Split(r.URL.Path, "/")
        if len(pathParts) >= 3 {
            return parseVersion(pathParts[2])
        }
    }
    
    // Method 2: Header versioning
    if version := r.Header.Get("API-Version"); version != "" {
        return parseVersion(version)
    }
    
    // Method 3: Accept header versioning
    accept := r.Header.Get("Accept")
    if strings.Contains(accept, "application/vnd.api") {
        if strings.Contains(accept, "version=") {
            parts := strings.Split(accept, "version=")
            if len(parts) > 1 {
                versionPart := strings.Split(parts[1], ",")[0]
                return parseVersion(versionPart)
            }
        }
    }
    
    // Method 4: Query parameter versioning
    if version := r.URL.Query().Get("version"); version != "" {
        return parseVersion(version)
    }
    
    // Default to v1.0
    return APIVersion{Major: 1, Minor: 0}
}

func transformUserForVersion(user User, version APIVersion) interface{} {
    switch version.Major {
    case 1:
        // V1: Basic user information
        return map[string]interface{}{
            "id":         user.ID,
            "name":       user.Name,
            "email":      user.Email,
            "created_at": user.CreatedAt.Format(time.RFC3339),
        }
        
    case 2:
        // V2: Split name into first_name and last_name, add avatar
        v2User := map[string]interface{}{
            "id":         user.ID,
            "email":      user.Email,
            "created_at": user.CreatedAt.Format(time.RFC3339),
        }
        
        if user.FirstName != "" && user.LastName != "" {
            v2User["first_name"] = user.FirstName
            v2User["last_name"] = user.LastName
        } else {
            // Split name for backwards compatibility
            nameParts := strings.SplitN(user.Name, " ", 2)
            v2User["first_name"] = nameParts[0]
            if len(nameParts) > 1 {
                v2User["last_name"] = nameParts[1]
            }
        }
        
        if user.Avatar != "" {
            v2User["avatar"] = user.Avatar
        }
        
        return v2User
        
    case 3:
        // V3: Full user object with profile and settings
        return user
        
    default:
        // Unknown version, return latest
        return user
    }
}

func createSampleUsers() []User {
    return []User{
        {
            ID:        1,
            Name:      "Alice Johnson",
            Email:     "alice@example.com",
            CreatedAt: time.Now().Add(-30 * 24 * time.Hour),
            FirstName: "Alice",
            LastName:  "Johnson",
            Avatar:    "https://example.com/avatars/alice.jpg",
            Profile: &UserProfile{
                Bio:      "Software engineer passionate about Go",
                Location: "San Francisco, CA",
                Website:  "https://alice.dev",
                Skills:   []string{"Go", "Python", "JavaScript", "React"},
            },
            Settings: &UserSettings{
                Theme:         "dark",
                Language:      "en",
                Notifications: true,
                Privacy:       "public",
            },
        },
        {
            ID:        2,
            Name:      "Bob Smith",
            Email:     "bob@example.com",
            CreatedAt: time.Now().Add(-15 * 24 * time.Hour),
            FirstName: "Bob",
            LastName:  "Smith",
            Avatar:    "https://example.com/avatars/bob.jpg",
            Profile: &UserProfile{
                Bio:      "DevOps engineer and cloud architect",
                Location: "New York, NY",
                Website:  "https://bobsmith.io",
                Skills:   []string{"Kubernetes", "AWS", "Docker", "Go"},
            },
            Settings: &UserSettings{
                Theme:         "light",
                Language:      "en",
                Notifications: false,
                Privacy:       "private",
            },
        },
    }
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    version := getVersionFromRequest(r)
    users := createSampleUsers()
    
    // Transform users based on API version
    transformedUsers := make([]interface{}, len(users))
    for i, user := range users {
        transformedUsers[i] = transformUserForVersion(user, version)
    }
    
    response := map[string]interface{}{
        "message":     "Hello there! User list retrieved",
        "version":     version.String(),
        "users":       transformedUsers,
        "total":       len(users),
        "timestamp":   time.Now().Format(time.RFC3339),
    }
    
    // Set response headers
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("API-Version", version.String())
    w.Header().Set("X-API-Deprecated", getDeprecationWarning(version))
    
    json.NewEncoder(w).Encode(response)
}

func userHandler(w http.ResponseWriter, r *http.Request) {
    version := getVersionFromRequest(r)
    
    // Extract user ID from path
    path := strings.TrimPrefix(r.URL.Path, "/api/")
    path = strings.TrimPrefix(path, version.String()+"/")
    path = strings.TrimPrefix(path, "users/")
    
    userID, err := strconv.Atoi(path)
    if err != nil {
        http.Error(w, "Invalid user ID", http.StatusBadRequest)
        return
    }
    
    users := createSampleUsers()
    var user *User
    for _, u := range users {
        if u.ID == userID {
            user = &u
            break
        }
    }
    
    if user == nil {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }
    
    transformedUser := transformUserForVersion(*user, version)
    
    response := map[string]interface{}{
        "message":   "Hello there! User details retrieved",
        "version":   version.String(),
        "user":      transformedUser,
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("API-Version", version.String())
    w.Header().Set("X-API-Deprecated", getDeprecationWarning(version))
    
    json.NewEncoder(w).Encode(response)
}

func getDeprecationWarning(version APIVersion) string {
    if version.Major == 1 {
        return "API v1 is deprecated. Please migrate to v2 or v3."
    }
    if version.Major == 2 && version.Minor == 0 {
        return "API v2.0 will be deprecated in 6 months. Consider upgrading to v3."
    }
    return ""
}

func versionInfoHandler(w http.ResponseWriter, r *http.Request) {
    currentVersion := getVersionFromRequest(r)
    
    versionInfo := map[string]interface{}{
        "message":         "API version information",
        "current_version": currentVersion.String(),
        "supported_versions": []map[string]interface{}{
            {
                "version":     "v1.0",
                "status":      "deprecated",
                "sunset_date": "2024-12-31",
                "description": "Original API version with basic user fields",
            },
            {
                "version":     "v2.0",
                "status":      "stable",
                "sunset_date": "2025-06-30",
                "description": "Enhanced API with split name fields and avatar support",
            },
            {
                "version":     "v3.0",
                "status":      "current",
                "sunset_date": null,
                "description": "Latest API with full user profiles and settings",
            },
        },
        "versioning_methods": []string{
            "URL path: /api/v2/users",
            "Header: API-Version: v2.0",
            "Accept: application/vnd.api+json;version=2.0",
            "Query: /api/users?version=v2.0",
        },
        "migration_guide": "https://api.example.com/docs/migration",
        "timestamp":       time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(versionInfo)
}

func migrationHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>API Migration Guide</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; max-width: 1200px; }
            .version-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            .deprecated { border-left: 4px solid #dc3545; }
            .stable { border-left: 4px solid #ffc107; }
            .current { border-left: 4px solid #28a745; }
            table { width: 100%; border-collapse: collapse; margin: 10px 0; }
            th, td { padding: 8px; text-align: left; border-bottom: 1px solid #ddd; }
            th { background: #f8f9fa; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; }
            .breaking-change { background: #fff3cd; padding: 10px; border-radius: 3px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <h1>API Versioning Migration Guide</h1>
        <p>Hello there! This guide helps you migrate between API versions.</p>
        
        <div class="version-section deprecated">
            <h2>API v1.0 (Deprecated)</h2>
            <p><strong>Status:</strong> Deprecated | <strong>Sunset:</strong> December 31, 2024</p>
            <p>The original API version with basic user information.</p>
            
            <h3>Response Format:</h3>
            <pre>
{
  "id": 1,
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "created_at": "2023-12-01T10:00:00Z"
}
            </pre>
        </div>
        
        <div class="version-section stable">
            <h2>API v2.0 (Stable)</h2>
            <p><strong>Status:</strong> Stable | <strong>Sunset:</strong> June 30, 2025</p>
            <p>Enhanced API with improved name handling and avatar support.</p>
            
            <h3>Breaking Changes:</h3>
            <div class="breaking-change">
                <strong>Name Field Split:</strong> The <code>name</code> field has been split into <code>first_name</code> and <code>last_name</code> fields.
            </div>
            
            <h3>Response Format:</h3>
            <pre>
{
  "id": 1,
  "first_name": "Alice",
  "last_name": "Johnson",
  "email": "alice@example.com",
  "avatar": "https://example.com/avatars/alice.jpg",
  "created_at": "2023-12-01T10:00:00Z"
}
            </pre>
        </div>
        
        <div class="version-section current">
            <h2>API v3.0 (Current)</h2>
            <p><strong>Status:</strong> Current | <strong>Sunset:</strong> None</p>
            <p>Latest API with full user profiles and settings.</p>
            
            <h3>New Features:</h3>
            <ul>
                <li>User profiles with bio, location, and skills</li>
                <li>User settings for theme and preferences</li>
                <li>Enhanced data structure</li>
                <li>Better error handling</li>
            </ul>
            
            <h3>Response Format:</h3>
            <pre>
{
  "id": 1,
  "name": "Alice Johnson",
  "first_name": "Alice",
  "last_name": "Johnson",
  "email": "alice@example.com",
  "avatar": "https://example.com/avatars/alice.jpg",
  "created_at": "2023-12-01T10:00:00Z",
  "profile": {
    "bio": "Software engineer passionate about Go",
    "location": "San Francisco, CA",
    "website": "https://alice.dev",
    "skills": ["Go", "Python", "JavaScript", "React"]
  },
  "settings": {
    "theme": "dark",
    "language": "en",
    "notifications": true,
    "privacy": "public"
  }
}
            </pre>
        </div>
        
        <h2>Versioning Methods</h2>
        <table>
            <thead>
                <tr>
                    <th>Method</th>
                    <th>Example</th>
                    <th>Pros</th>
                    <th>Cons</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>URL Path</td>
                    <td><code>/api/v2/users</code></td>
                    <td>Clear, cacheable</td>
                    <td>URL pollution</td>
                </tr>
                <tr>
                    <td>Header</td>
                    <td><code>API-Version: v2.0</code></td>
                    <td>Clean URLs</td>
                    <td>Less visible</td>
                </tr>
                <tr>
                    <td>Accept Header</td>
                    <td><code>application/vnd.api+json;version=2.0</code></td>
                    <td>HTTP standard</td>
                    <td>Complex</td>
                </tr>
                <tr>
                    <td>Query Parameter</td>
                    <td><code>/api/users?version=v2.0</code></td>
                    <td>Simple</td>
                    <td>Can be ignored</td>
                </tr>
            </tbody>
        </table>
        
        <h2>Migration Checklist</h2>
        <h3>V1 → V2 Migration:</h3>
        <ul>
            <li>☐ Update client to handle split name fields</li>
            <li>☐ Add avatar field handling</li>
            <li>☐ Update API version in requests</li>
            <li>☐ Test with new response format</li>
            <li>☐ Update documentation</li>
        </ul>
        
        <h3>V2 → V3 Migration:</h3>
        <ul>
            <li>☐ Add profile object handling</li>
            <li>☐ Add settings object handling</li>
            <li>☐ Update API version in requests</li>
            <li>☐ Handle new nested data structure</li>
            <li>☐ Update error handling for new format</li>
        </ul>
        
        <h2>Test API Versions</h2>
        <p>Try different versioning methods:</p>
        <ul>
            <li><a href="/api/v1/users" target="_blank">V1 Users (URL path)</a></li>
            <li><a href="/api/v2/users" target="_blank">V2 Users (URL path)</a></li>
            <li><a href="/api/v3/users" target="_blank">V3 Users (URL path)</a></li>
            <li><a href="/api/users?version=v2.0" target="_blank">V2 Users (Query param)</a></li>
        </ul>
    </body>
    </html>`)
}

func apiVersionDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>API Versioning Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            select, input { margin: 5px; padding: 5px; }
        </style>
    </head>
    <body>
        <h1>API Versioning Demo</h1>
        <p>Hello there! This demonstrates different API versioning strategies.</p>
        
        <div class="demo-section">
            <h2>Test API Versions</h2>
            <p>Select version and method to test:</p>
            <select id="version">
                <option value="v1.0">v1.0 (Deprecated)</option>
                <option value="v2.0" selected>v2.0 (Stable)</option>
                <option value="v3.0">v3.0 (Current)</option>
            </select>
            
            <select id="method">
                <option value="path">URL Path</option>
                <option value="header">Header</option>
                <option value="accept">Accept Header</option>
                <option value="query">Query Parameter</option>
            </select>
            
            <button onclick="testVersion()">Test API Version</button>
            <button onclick="testSpecificUser()">Test User Details</button>
            <div id="version-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Version Information</h2>
            <button onclick="getVersionInfo()">Get Version Info</button>
            <button onclick="showMigrationGuide()">Migration Guide</button>
            <div id="info-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Comparison Tool</h2>
            <button onclick="compareVersions()">Compare All Versions</button>
            <div id="comparison-result" class="result"></div>
        </div>
        
        <script>
        function testVersion() {
            const version = document.getElementById('version').value;
            const method = document.getElementById('method').value;
            
            let url = '/api/users';
            let headers = { 'Content-Type': 'application/json' };
            
            switch (method) {
                case 'path':
                    url = '/api/' + version + '/users';
                    break;
                case 'header':
                    headers['API-Version'] = version;
                    break;
                case 'accept':
                    headers['Accept'] = 'application/vnd.api+json;version=' + version;
                    break;
                case 'query':
                    url = '/api/users?version=' + version;
                    break;
            }
            
            document.getElementById('version-result').innerHTML = 'Testing ' + version + ' via ' + method + '...';
            
            fetch(url, { headers: headers })
            .then(response => response.json())
            .then(data => {
                let html = '<h4>API ' + version + ' Response (' + method + ')</h4>';
                html += '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                document.getElementById('version-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('version-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testSpecificUser() {
            const version = document.getElementById('version').value;
            const method = document.getElementById('method').value;
            
            let url = '/api/users/1';
            let headers = { 'Content-Type': 'application/json' };
            
            switch (method) {
                case 'path':
                    url = '/api/' + version + '/users/1';
                    break;
                case 'header':
                    headers['API-Version'] = version;
                    break;
                case 'accept':
                    headers['Accept'] = 'application/vnd.api+json;version=' + version;
                    break;
                case 'query':
                    url = '/api/users/1?version=' + version;
                    break;
            }
            
            fetch(url, { headers: headers })
            .then(response => response.json())
            .then(data => {
                let html = '<h4>User Details - API ' + version + '</h4>';
                html += '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                document.getElementById('version-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('version-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function getVersionInfo() {
            fetch('/api/version-info')
            .then(response => response.json())
            .then(data => {
                document.getElementById('info-result').innerHTML = 
                    '<h4>Version Information</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('info-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function showMigrationGuide() {
            window.open('/migration-guide', '_blank');
        }
        
        function compareVersions() {
            document.getElementById('comparison-result').innerHTML = 'Comparing all versions...';
            
            const versions = ['v1.0', 'v2.0', 'v3.0'];
            const promises = versions.map(version => 
                fetch('/api/' + version + '/users/1').then(r => r.json())
            );
            
            Promise.all(promises).then(results => {
                let html = '<h4>Version Comparison</h4>';
                
                results.forEach((result, index) => {
                    html += '<h5>' + versions[index] + ':</h5>';
                    html += '<pre>' + JSON.stringify(result.user, null, 2) + '</pre>';
                });
                
                document.getElementById('comparison-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('comparison-result').innerHTML = 
                    '<h4 style="color: red;">Comparison Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // API versioning routes
    http.HandleFunc("/", apiVersionDemoHandler)
    http.HandleFunc("/migration-guide", migrationHandler)
    
    // Version-specific routes (URL path versioning)
    http.HandleFunc("/api/v1/users", usersHandler)
    http.HandleFunc("/api/v1/users/", userHandler)
    http.HandleFunc("/api/v2/users", usersHandler)
    http.HandleFunc("/api/v2/users/", userHandler)
    http.HandleFunc("/api/v3/users", usersHandler)
    http.HandleFunc("/api/v3/users/", userHandler)
    
    // Generic routes (use header/query versioning)
    http.HandleFunc("/api/users", usersHandler)
    http.HandleFunc("/api/users/", userHandler)
    
    // Utility routes
    http.HandleFunc("/api/version-info", versionInfoHandler)
    
    fmt.Println("=== HTTP API Versioning Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Multiple versioning strategies")
    fmt.Println("• Backward compatibility")
    fmt.Println("• Version deprecation handling")
    fmt.Println("• Data transformation between versions")
    fmt.Println("• Migration guidance")
    fmt.Println("• Version detection from multiple sources")
    fmt.Println()
    fmt.Println("API Versions:")
    fmt.Println("• v1.0 - Basic user data (deprecated)")
    fmt.Println("• v2.0 - Enhanced with split names and avatars")
    fmt.Println("• v3.0 - Full profiles and settings")
    fmt.Println()
    fmt.Println("Versioning methods:")
    fmt.Println("• URL: /api/v2/users")
    fmt.Println("• Header: API-Version: v2.0")
    fmt.Println("• Accept: application/vnd.api+json;version=2.0")
    fmt.Println("• Query: /api/users?version=v2.0")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This API versioning example demonstrates comprehensive strategies for managing  
API evolution including multiple versioning methods, backward compatibility,  
deprecation handling, and migration guidance. Proper API versioning is  
essential for maintaining long-term API stability and client satisfaction.

## HTTP circuit breaker pattern

The circuit breaker pattern prevents cascading failures by automatically  
stopping requests to failing services. This example demonstrates circuit  
breaker implementation for HTTP services with state management and recovery.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "sync"
    "time"
)

type CircuitState string

const (
    StateClosed   CircuitState = "closed"
    StateOpen     CircuitState = "open"
    StateHalfOpen CircuitState = "half-open"
)

type CircuitBreaker struct {
    name             string
    maxFailures      int
    timeout          time.Duration
    resetTimeout     time.Duration
    state            CircuitState
    failures         int
    lastFailureTime  time.Time
    lastSuccessTime  time.Time
    nextAttempt      time.Time
    mu               sync.RWMutex
    onStateChange    func(name string, from, to CircuitState)
}

func NewCircuitBreaker(name string, maxFailures int, timeout, resetTimeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        name:         name,
        maxFailures:  maxFailures,
        timeout:      timeout,
        resetTimeout: resetTimeout,
        state:        StateClosed,
    }
}

func (cb *CircuitBreaker) Call(fn func() error) error {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    now := time.Now()
    
    // Check if we should transition from open to half-open
    if cb.state == StateOpen && now.After(cb.nextAttempt) {
        cb.setState(StateHalfOpen)
    }
    
    // Don't allow calls when circuit is open
    if cb.state == StateOpen {
        return fmt.Errorf("circuit breaker %s is open", cb.name)
    }
    
    // Execute the function
    err := fn()
    
    if err != nil {
        cb.onFailure(now)
        return err
    }
    
    cb.onSuccess(now)
    return nil
}

func (cb *CircuitBreaker) onSuccess(now time.Time) {
    cb.lastSuccessTime = now
    
    if cb.state == StateHalfOpen {
        // Reset the circuit breaker on successful call in half-open state
        cb.setState(StateClosed)
        cb.failures = 0
    }
}

func (cb *CircuitBreaker) onFailure(now time.Time) {
    cb.failures++
    cb.lastFailureTime = now
    
    if cb.failures >= cb.maxFailures {
        cb.setState(StateOpen)
        cb.nextAttempt = now.Add(cb.resetTimeout)
    }
}

func (cb *CircuitBreaker) setState(newState CircuitState) {
    if cb.state != newState {
        oldState := cb.state
        cb.state = newState
        
        log.Printf("Circuit breaker %s: %s -> %s", cb.name, oldState, newState)
        
        if cb.onStateChange != nil {
            cb.onStateChange(cb.name, oldState, newState)
        }
    }
}

func (cb *CircuitBreaker) GetState() map[string]interface{} {
    cb.mu.RLock()
    defer cb.mu.RUnlock()
    
    return map[string]interface{}{
        "name":              cb.name,
        "state":             string(cb.state),
        "failures":          cb.failures,
        "max_failures":      cb.maxFailures,
        "timeout":           cb.timeout.String(),
        "reset_timeout":     cb.resetTimeout.String(),
        "last_failure_time": cb.lastFailureTime.Format(time.RFC3339),
        "last_success_time": cb.lastSuccessTime.Format(time.RFC3339),
        "next_attempt":      cb.nextAttempt.Format(time.RFC3339),
    }
}

type HTTPCircuitBreaker struct {
    client         *http.Client
    circuitBreaker *CircuitBreaker
}

func NewHTTPCircuitBreaker(name string, maxFailures int, timeout, resetTimeout time.Duration) *HTTPCircuitBreaker {
    return &HTTPCircuitBreaker{
        client: &http.Client{Timeout: timeout},
        circuitBreaker: NewCircuitBreaker(name, maxFailures, timeout, resetTimeout),
    }
}

func (hcb *HTTPCircuitBreaker) Get(url string) (*http.Response, error) {
    var resp *http.Response
    var err error
    
    cbErr := hcb.circuitBreaker.Call(func() error {
        resp, err = hcb.client.Get(url)
        
        if err != nil {
            return err
        }
        
        // Consider 5xx status codes as failures
        if resp.StatusCode >= 500 {
            return fmt.Errorf("server error: %d", resp.StatusCode)
        }
        
        return nil
    })
    
    if cbErr != nil {
        return nil, cbErr
    }
    
    return resp, err
}

func (hcb *HTTPCircuitBreaker) GetState() map[string]interface{} {
    return hcb.circuitBreaker.GetState()
}

// Global circuit breakers for different services
var (
    paymentService   *HTTPCircuitBreaker
    userService      *HTTPCircuitBreaker
    inventoryService *HTTPCircuitBreaker
)

func initCircuitBreakers() {
    // Circuit breaker for payment service (more sensitive)
    paymentService = NewHTTPCircuitBreaker(
        "payment-service",
        3,                  // Max failures
        5*time.Second,      // Request timeout
        30*time.Second,     // Reset timeout
    )
    
    // Circuit breaker for user service
    userService = NewHTTPCircuitBreaker(
        "user-service",
        5,                  // Max failures
        10*time.Second,     // Request timeout
        20*time.Second,     // Reset timeout
    )
    
    // Circuit breaker for inventory service
    inventoryService = NewHTTPCircuitBreaker(
        "inventory-service",
        4,                  // Max failures
        8*time.Second,      // Request timeout
        25*time.Second,     // Reset timeout
    )
    
    // Set up state change notifications
    paymentService.circuitBreaker.onStateChange = onCircuitBreakerStateChange
    userService.circuitBreaker.onStateChange = onCircuitBreakerStateChange
    inventoryService.circuitBreaker.onStateChange = onCircuitBreakerStateChange
}

func onCircuitBreakerStateChange(name string, from, to CircuitState) {
    log.Printf("⚡ Circuit breaker state change: %s (%s -> %s)", name, from, to)
    
    // In a real application, you might:
    // - Send alerts to monitoring systems
    // - Update metrics
    // - Trigger fallback mechanisms
    // - Notify operations teams
}

// Handlers that use circuit breakers
func orderHandler(w http.ResponseWriter, r *http.Request) {
    orderID := r.URL.Query().Get("order_id")
    if orderID == "" {
        orderID = "order_12345"
    }
    
    // Simulate order processing with multiple service calls
    var responses []map[string]interface{}
    
    // 1. Check inventory
    inventoryResp, err := inventoryService.Get("https://httpbin.org/status/200")
    if err != nil {
        responses = append(responses, map[string]interface{}{
            "service": "inventory",
            "status":  "failed",
            "error":   err.Error(),
        })
    } else {
        inventoryResp.Body.Close()
        responses = append(responses, map[string]interface{}{
            "service": "inventory",
            "status":  "success",
            "code":    inventoryResp.StatusCode,
        })
    }
    
    // 2. Process payment
    paymentResp, err := paymentService.Get("https://httpbin.org/status/200")
    if err != nil {
        responses = append(responses, map[string]interface{}{
            "service": "payment",
            "status":  "failed",
            "error":   err.Error(),
        })
    } else {
        paymentResp.Body.Close()
        responses = append(responses, map[string]interface{}{
            "service": "payment",
            "status":  "success",
            "code":    paymentResp.StatusCode,
        })
    }
    
    // 3. Update user account
    userResp, err := userService.Get("https://httpbin.org/status/200")
    if err != nil {
        responses = append(responses, map[string]interface{}{
            "service": "user",
            "status":  "failed",
            "error":   err.Error(),
        })
    } else {
        userResp.Body.Close()
        responses = append(responses, map[string]interface{}{
            "service": "user",
            "status":  "success",
            "code":    userResp.StatusCode,
        })
    }
    
    // Determine overall order status
    successCount := 0
    for _, resp := range responses {
        if resp["status"] == "success" {
            successCount++
        }
    }
    
    orderStatus := "completed"
    if successCount < len(responses) {
        orderStatus = "partially_completed"
    }
    if successCount == 0 {
        orderStatus = "failed"
    }
    
    response := map[string]interface{}{
        "message":        "Hello there! Order processing completed",
        "order_id":       orderID,
        "order_status":   orderStatus,
        "service_calls":  responses,
        "success_rate":   float64(successCount) / float64(len(responses)) * 100,
        "timestamp":      time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    if orderStatus == "failed" {
        w.WriteHeader(http.StatusServiceUnavailable)
    }
    json.NewEncoder(w).Encode(response)
}

func simulateFailureHandler(w http.ResponseWriter, r *http.Request) {
    service := r.URL.Query().Get("service")
    statusCode := r.URL.Query().Get("status")
    
    if statusCode == "" {
        statusCode = "500"
    }
    
    var cb *HTTPCircuitBreaker
    switch service {
    case "payment":
        cb = paymentService
    case "user":
        cb = userService
    case "inventory":
        cb = inventoryService
    default:
        http.Error(w, "Unknown service", http.StatusBadRequest)
        return
    }
    
    // Force a failure by calling a failing endpoint
    url := fmt.Sprintf("https://httpbin.org/status/%s", statusCode)
    resp, err := cb.Get(url)
    
    result := map[string]interface{}{
        "message":       "Failure simulation completed",
        "service":       service,
        "url":           url,
        "circuit_state": cb.GetState(),
        "timestamp":     time.Now().Format(time.RFC3339),
    }
    
    if err != nil {
        result["error"] = err.Error()
    } else {
        result["status_code"] = resp.StatusCode
        resp.Body.Close()
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(result)
}

func circuitStatusHandler(w http.ResponseWriter, r *http.Request) {
    status := map[string]interface{}{
        "message": "Circuit breaker status",
        "services": map[string]interface{}{
            "payment":   paymentService.GetState(),
            "user":      userService.GetState(),
            "inventory": inventoryService.GetState(),
        },
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(status)
}

func circuitBreakerDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Circuit Breaker Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 300px; }
            .status-closed { color: #28a745; font-weight: bold; }
            .status-open { color: #dc3545; font-weight: bold; }
            .status-half-open { color: #ffc107; font-weight: bold; }
            .service-card { background: #fff; padding: 15px; margin: 10px 0; border-radius: 5px; border-left: 4px solid #007cba; }
        </style>
    </head>
    <body>
        <h1>Circuit Breaker Pattern Demo</h1>
        <p>Hello there! This demonstrates circuit breaker implementation for HTTP services.</p>
        
        <div class="demo-section">
            <h2>Service Calls</h2>
            <p>Test normal operations that use multiple services:</p>
            <button onclick="processOrder()">Process Order</button>
            <button onclick="processMultipleOrders()">Process 10 Orders</button>
            <div id="order-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Failure Simulation</h2>
            <p>Simulate failures to trigger circuit breakers:</p>
            <button onclick="simulateFailure('payment', '500')">Payment Service Failure</button>
            <button onclick="simulateFailure('user', '503')">User Service Failure</button>
            <button onclick="simulateFailure('inventory', '500')">Inventory Service Failure</button>
            <button onclick="simulateFailure('payment', '200')">Payment Service Recovery</button>
            <div id="failure-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Circuit Breaker Status</h2>
            <button onclick="getCircuitStatus()">Get Status</button>
            <button onclick="startStatusMonitoring()">Start Auto-Refresh</button>
            <button onclick="stopStatusMonitoring()">Stop Auto-Refresh</button>
            <div id="status-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Circuit Breaker States</h2>
            <div class="service-card">
                <h4>Closed State <span class="status-closed">●</span></h4>
                <p>Normal operation. Requests are passed through. Failures are counted.</p>
            </div>
            <div class="service-card">
                <h4>Open State <span class="status-open">●</span></h4>
                <p>Circuit is tripped. All requests fail immediately without calling the service.</p>
            </div>
            <div class="service-card">
                <h4>Half-Open State <span class="status-half-open">●</span></h4>
                <p>Testing phase. Limited requests are allowed to test if service has recovered.</p>
            </div>
        </div>
        
        <script>
        let statusInterval = null;
        
        function processOrder() {
            document.getElementById('order-result').innerHTML = 'Processing order...';
            
            fetch('/order?order_id=ORD' + Date.now())
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Order Processing Result</h4>';
                html += '<p>Order Status: <strong>' + data.order_status + '</strong></p>';
                html += '<p>Success Rate: ' + data.success_rate.toFixed(1) + '%</p>';
                html += '<h5>Service Calls:</h5>';
                
                data.service_calls.forEach(call => {
                    const statusColor = call.status === 'success' ? 'green' : 'red';
                    html += '<p style="color: ' + statusColor + ';">';
                    html += call.service + ': ' + call.status;
                    if (call.error) {
                        html += ' (' + call.error + ')';
                    }
                    html += '</p>';
                });
                
                document.getElementById('order-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('order-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function processMultipleOrders() {
            document.getElementById('order-result').innerHTML = 'Processing 10 orders...';
            
            const promises = [];
            for (let i = 0; i < 10; i++) {
                promises.push(
                    fetch('/order?order_id=BATCH' + i)
                    .then(r => r.json())
                    .catch(e => ({ error: e.message }))
                );
            }
            
            Promise.all(promises).then(results => {
                let successCount = 0;
                let failedCount = 0;
                let partialCount = 0;
                
                results.forEach(result => {
                    if (result.order_status === 'completed') successCount++;
                    else if (result.order_status === 'failed') failedCount++;
                    else partialCount++;
                });
                
                let html = '<h4>Batch Order Processing</h4>';
                html += '<p>Completed: ' + successCount + '</p>';
                html += '<p>Partially Completed: ' + partialCount + '</p>';
                html += '<p>Failed: ' + failedCount + '</p>';
                
                document.getElementById('order-result').innerHTML = html;
                getCircuitStatus(); // Refresh status after batch
            });
        }
        
        function simulateFailure(service, status) {
            document.getElementById('failure-result').innerHTML = 'Simulating ' + service + ' failure...';
            
            fetch('/simulate-failure?service=' + service + '&status=' + status)
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Failure Simulation Result</h4>';
                html += '<p>Service: ' + data.service + '</p>';
                html += '<p>Circuit State: <span class="status-' + data.circuit_state.state + '">' + 
                        data.circuit_state.state + '</span></p>';
                html += '<p>Failures: ' + data.circuit_state.failures + '/' + data.circuit_state.max_failures + '</p>';
                
                if (data.error) {
                    html += '<p style="color: red;">Error: ' + data.error + '</p>';
                }
                
                document.getElementById('failure-result').innerHTML = html;
                getCircuitStatus(); // Refresh status
            })
            .catch(error => {
                document.getElementById('failure-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function getCircuitStatus() {
            fetch('/circuit-status')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Circuit Breaker Status</h4>';
                
                Object.keys(data.services).forEach(serviceName => {
                    const service = data.services[serviceName];
                    html += '<div class="service-card">';
                    html += '<h5>' + service.name + ' <span class="status-' + service.state + '">' + 
                            service.state + '</span></h5>';
                    html += '<p>Failures: ' + service.failures + '/' + service.max_failures + '</p>';
                    html += '<p>Timeout: ' + service.timeout + '</p>';
                    html += '<p>Reset Timeout: ' + service.reset_timeout + '</p>';
                    if (service.last_failure_time !== '0001-01-01T00:00:00Z') {
                        html += '<p>Last Failure: ' + new Date(service.last_failure_time).toLocaleString() + '</p>';
                    }
                    html += '</div>';
                });
                
                document.getElementById('status-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('status-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function startStatusMonitoring() {
            if (statusInterval) return;
            
            statusInterval = setInterval(getCircuitStatus, 2000);
            getCircuitStatus(); // Initial load
        }
        
        function stopStatusMonitoring() {
            if (statusInterval) {
                clearInterval(statusInterval);
                statusInterval = null;
            }
        }
        
        // Initial status load
        getCircuitStatus();
        </script>
    </body>
    </html>`)
}

func main() {
    // Initialize circuit breakers
    initCircuitBreakers()
    
    // Routes
    http.HandleFunc("/", circuitBreakerDemoHandler)
    http.HandleFunc("/order", orderHandler)
    http.HandleFunc("/simulate-failure", simulateFailureHandler)
    http.HandleFunc("/circuit-status", circuitStatusHandler)
    
    fmt.Println("=== HTTP Circuit Breaker Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Circuit breaker pattern implementation")
    fmt.Println("• Automatic failure detection")
    fmt.Println("• Service degradation handling")
    fmt.Println("• State transitions (closed/open/half-open)")
    fmt.Println("• Fallback mechanisms")
    fmt.Println("• Recovery testing")
    fmt.Println()
    fmt.Println("Circuit breakers configured:")
    fmt.Println("• Payment Service: 3 failures, 30s reset")
    fmt.Println("• User Service: 5 failures, 20s reset")
    fmt.Println("• Inventory Service: 4 failures, 25s reset")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Circuit breaker demo")
    fmt.Println("  GET /order - Process order (uses all services)")
    fmt.Println("  GET /simulate-failure - Simulate service failures")
    fmt.Println("  GET /circuit-status - Get circuit breaker status")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This circuit breaker example demonstrates comprehensive failure handling  
including automatic failure detection, state management, and recovery testing.  
Circuit breakers are essential for building resilient distributed systems  
that can gracefully handle service failures and prevent cascading outages.

## HTTP request mocking and testing utilities

Request mocking and testing utilities are essential for unit testing HTTP  
clients and services. This example demonstrates comprehensive testing  
patterns including mock servers, request verification, and test utilities.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
    "time"
)

// MockServer provides a configurable HTTP mock server for testing
type MockServer struct {
    server     *httptest.Server
    responses  map[string]*MockResponse
    requests   []*RecordedRequest
    middleware []func(http.Handler) http.Handler
}

type MockResponse struct {
    StatusCode int                    `json:"status_code"`
    Headers    map[string]string      `json:"headers"`
    Body       interface{}            `json:"body"`
    Delay      time.Duration          `json:"delay"`
    Count      int                   `json:"count"` // How many times to use this response
    Used       int                   `json:"used"`  // How many times it's been used
}

type RecordedRequest struct {
    Method    string              `json:"method"`
    URL       string              `json:"url"`
    Headers   map[string][]string `json:"headers"`
    Body      string              `json:"body"`
    Timestamp time.Time           `json:"timestamp"`
}

func NewMockServer() *MockServer {
    ms := &MockServer{
        responses: make(map[string]*MockResponse),
        requests:  make([]*RecordedRequest, 0),
    }
    
    mux := http.NewServeMux()
    mux.HandleFunc("/", ms.handler)
    
    // Apply middleware
    var handler http.Handler = mux
    for i := len(ms.middleware) - 1; i >= 0; i-- {
        handler = ms.middleware[i](handler)
    }
    
    ms.server = httptest.NewServer(handler)
    return ms
}

func (ms *MockServer) AddResponse(method, path string, response *MockResponse) {
    key := method + " " + path
    ms.responses[key] = response
}

func (ms *MockServer) AddMiddleware(middleware func(http.Handler) http.Handler) {
    ms.middleware = append(ms.middleware, middleware)
}

func (ms *MockServer) handler(w http.ResponseWriter, r *http.Request) {
    // Record the request
    body, _ := io.ReadAll(r.Body)
    r.Body = io.NopCloser(bytes.NewReader(body))
    
    recordedReq := &RecordedRequest{
        Method:    r.Method,
        URL:       r.URL.String(),
        Headers:   r.Header,
        Body:      string(body),
        Timestamp: time.Now(),
    }
    ms.requests = append(ms.requests, recordedReq)
    
    // Find matching response
    key := r.Method + " " + r.URL.Path
    response, exists := ms.responses[key]
    if !exists {
        // Try wildcard match
        wildcardKey := r.Method + " *"
        response, exists = ms.responses[wildcardKey]
    }
    
    if !exists {
        http.Error(w, "No mock response configured", http.StatusNotFound)
        return
    }
    
    // Check if response has been used too many times
    if response.Count > 0 && response.Used >= response.Count {
        http.Error(w, "Mock response exhausted", http.StatusGone)
        return
    }
    
    response.Used++
    
    // Add delay if specified
    if response.Delay > 0 {
        time.Sleep(response.Delay)
    }
    
    // Set headers
    for key, value := range response.Headers {
        w.Header().Set(key, value)
    }
    
    // Set status code
    w.WriteHeader(response.StatusCode)
    
    // Write body
    switch body := response.Body.(type) {
    case string:
        w.Write([]byte(body))
    case []byte:
        w.Write(body)
    default:
        json.NewEncoder(w).Encode(body)
    }
}

func (ms *MockServer) GetRequests() []*RecordedRequest {
    return ms.requests
}

func (ms *MockServer) ClearRequests() {
    ms.requests = make([]*RecordedRequest, 0)
}

func (ms *MockServer) GetURL() string {
    return ms.server.URL
}

func (ms *MockServer) Close() {
    ms.server.Close()
}

// HTTPClient wrapper for testing
type HTTPClient struct {
    client  *http.Client
    baseURL string
}

func NewHTTPClient(baseURL string) *HTTPClient {
    return &HTTPClient{
        client:  &http.Client{Timeout: 30 * time.Second},
        baseURL: baseURL,
    }
}

func (c *HTTPClient) Get(path string) (*http.Response, error) {
    url := c.baseURL + path
    return c.client.Get(url)
}

func (c *HTTPClient) Post(path string, body interface{}) (*http.Response, error) {
    url := c.baseURL + path
    
    var bodyReader io.Reader
    if body != nil {
        jsonBody, err := json.Marshal(body)
        if err != nil {
            return nil, err
        }
        bodyReader = bytes.NewReader(jsonBody)
    }
    
    req, err := http.NewRequest("POST", url, bodyReader)
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Content-Type", "application/json")
    return c.client.Do(req)
}

// Test utilities
func AssertStatusCode(t *testing.T, expected, actual int) {
    if expected != actual {
        t.Errorf("Expected status code %d, got %d", expected, actual)
    }
}

func AssertHeader(t *testing.T, response *http.Response, key, expected string) {
    actual := response.Header.Get(key)
    if actual != expected {
        t.Errorf("Expected header %s to be %q, got %q", key, expected, actual)
    }
}

func AssertBodyContains(t *testing.T, response *http.Response, expected string) {
    body, err := io.ReadAll(response.Body)
    if err != nil {
        t.Fatalf("Failed to read response body: %v", err)
    }
    
    bodyStr := string(body)
    if !strings.Contains(bodyStr, expected) {
        t.Errorf("Expected response body to contain %q, got %q", expected, bodyStr)
    }
}

func AssertRequestCount(t *testing.T, mockServer *MockServer, expected int) {
    actual := len(mockServer.GetRequests())
    if actual != expected {
        t.Errorf("Expected %d requests, got %d", expected, actual)
    }
}

func AssertRequestMethod(t *testing.T, request *RecordedRequest, expected string) {
    if request.Method != expected {
        t.Errorf("Expected request method %s, got %s", expected, request.Method)
    }
}

func AssertRequestPath(t *testing.T, request *RecordedRequest, expected string) {
    if !strings.Contains(request.URL, expected) {
        t.Errorf("Expected request URL to contain %s, got %s", expected, request.URL)
    }
}

// Example tests demonstrating the testing utilities
func TestHTTPClientGet(t *testing.T) {
    // Create mock server
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    // Configure mock response
    mockServer.AddResponse("GET", "/users", &MockResponse{
        StatusCode: 200,
        Headers:    map[string]string{"Content-Type": "application/json"},
        Body: map[string]interface{}{
            "message": "Hello there! Mock response",
            "users": []map[string]interface{}{
                {"id": 1, "name": "Alice"},
                {"id": 2, "name": "Bob"},
            },
        },
    })
    
    // Create HTTP client
    client := NewHTTPClient(mockServer.GetURL())
    
    // Make request
    response, err := client.Get("/users")
    if err != nil {
        t.Fatalf("Request failed: %v", err)
    }
    defer response.Body.Close()
    
    // Assertions
    AssertStatusCode(t, 200, response.StatusCode)
    AssertHeader(t, response, "Content-Type", "application/json")
    AssertBodyContains(t, response, "Hello there! Mock response")
    AssertRequestCount(t, mockServer, 1)
    
    requests := mockServer.GetRequests()
    AssertRequestMethod(t, requests[0], "GET")
    AssertRequestPath(t, requests[0], "/users")
}

func TestHTTPClientPost(t *testing.T) {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    // Configure mock response
    mockServer.AddResponse("POST", "/users", &MockResponse{
        StatusCode: 201,
        Headers:    map[string]string{"Content-Type": "application/json"},
        Body: map[string]interface{}{
            "message": "User created successfully",
            "id":      123,
        },
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    
    // Prepare request body
    requestBody := map[string]interface{}{
        "name":  "Charlie",
        "email": "charlie@example.com",
    }
    
    response, err := client.Post("/users", requestBody)
    if err != nil {
        t.Fatalf("Request failed: %v", err)
    }
    defer response.Body.Close()
    
    AssertStatusCode(t, 201, response.StatusCode)
    AssertRequestCount(t, mockServer, 1)
    
    requests := mockServer.GetRequests()
    AssertRequestMethod(t, requests[0], "POST")
    
    // Verify request body
    var receivedBody map[string]interface{}
    json.Unmarshal([]byte(requests[0].Body), &receivedBody)
    
    if receivedBody["name"] != "Charlie" {
        t.Errorf("Expected name Charlie, got %v", receivedBody["name"])
    }
}

func TestErrorHandling(t *testing.T) {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    // Configure error response
    mockServer.AddResponse("GET", "/error", &MockResponse{
        StatusCode: 500,
        Headers:    map[string]string{"Content-Type": "application/json"},
        Body: map[string]interface{}{
            "error":   "Internal server error",
            "message": "Something went wrong",
        },
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    
    response, err := client.Get("/error")
    if err != nil {
        t.Fatalf("Request failed: %v", err)
    }
    defer response.Body.Close()
    
    AssertStatusCode(t, 500, response.StatusCode)
    AssertBodyContains(t, response, "Internal server error")
}

func TestResponseDelay(t *testing.T) {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    // Configure delayed response
    mockServer.AddResponse("GET", "/slow", &MockResponse{
        StatusCode: 200,
        Headers:    map[string]string{"Content-Type": "text/plain"},
        Body:       "Delayed response",
        Delay:      100 * time.Millisecond,
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    
    start := time.Now()
    response, err := client.Get("/slow")
    duration := time.Since(start)
    
    if err != nil {
        t.Fatalf("Request failed: %v", err)
    }
    defer response.Body.Close()
    
    if duration < 100*time.Millisecond {
        t.Errorf("Expected delay of at least 100ms, got %v", duration)
    }
    
    AssertStatusCode(t, 200, response.StatusCode)
}

func TestLimitedResponses(t *testing.T) {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    // Configure response that can only be used twice
    mockServer.AddResponse("GET", "/limited", &MockResponse{
        StatusCode: 200,
        Body:       "Limited response",
        Count:      2, // Can only be used twice
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    
    // First request should succeed
    resp1, err := client.Get("/limited")
    if err != nil {
        t.Fatalf("First request failed: %v", err)
    }
    resp1.Body.Close()
    AssertStatusCode(t, 200, resp1.StatusCode)
    
    // Second request should succeed
    resp2, err := client.Get("/limited")
    if err != nil {
        t.Fatalf("Second request failed: %v", err)
    }
    resp2.Body.Close()
    AssertStatusCode(t, 200, resp2.StatusCode)
    
    // Third request should fail (response exhausted)
    resp3, err := client.Get("/limited")
    if err != nil {
        t.Fatalf("Third request failed: %v", err)
    }
    resp3.Body.Close()
    AssertStatusCode(t, 410, resp3.StatusCode) // Gone
}

// Demo handlers for the web interface
func runTestsHandler(w http.ResponseWriter, r *http.Request) {
    testType := r.URL.Query().Get("type")
    
    var results []map[string]interface{}
    
    switch testType {
    case "get":
        results = append(results, runGetTest())
    case "post":
        results = append(results, runPostTest())
    case "error":
        results = append(results, runErrorTest())
    case "delay":
        results = append(results, runDelayTest())
    case "limited":
        results = append(results, runLimitedTest())
    default:
        // Run all tests
        results = append(results, runGetTest())
        results = append(results, runPostTest())
        results = append(results, runErrorTest())
        results = append(results, runDelayTest())
        results = append(results, runLimitedTest())
    }
    
    response := map[string]interface{}{
        "message":    "Hello there! Test execution completed",
        "test_type":  testType,
        "results":    results,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func runGetTest() map[string]interface{} {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("GET", "/users", &MockResponse{
        StatusCode: 200,
        Headers:    map[string]string{"Content-Type": "application/json"},
        Body: map[string]interface{}{
            "message": "Hello there! Mock response",
            "users":   []string{"Alice", "Bob"},
        },
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    response, err := client.Get("/users")
    
    result := map[string]interface{}{
        "test_name": "GET Request Test",
        "success":   err == nil && response.StatusCode == 200,
    }
    
    if response != nil {
        result["status_code"] = response.StatusCode
        response.Body.Close()
    }
    
    if err != nil {
        result["error"] = err.Error()
    }
    
    return result
}

func runPostTest() map[string]interface{} {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("POST", "/users", &MockResponse{
        StatusCode: 201,
        Body:       map[string]interface{}{"id": 123, "message": "Created"},
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    requestBody := map[string]string{"name": "Charlie"}
    response, err := client.Post("/users", requestBody)
    
    result := map[string]interface{}{
        "test_name": "POST Request Test",
        "success":   err == nil && response.StatusCode == 201,
    }
    
    if response != nil {
        result["status_code"] = response.StatusCode
        response.Body.Close()
    }
    
    if err != nil {
        result["error"] = err.Error()
    }
    
    return result
}

func runErrorTest() map[string]interface{} {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("GET", "/error", &MockResponse{
        StatusCode: 500,
        Body:       "Internal Server Error",
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    response, err := client.Get("/error")
    
    result := map[string]interface{}{
        "test_name": "Error Handling Test",
        "success":   err == nil && response.StatusCode == 500,
    }
    
    if response != nil {
        result["status_code"] = response.StatusCode
        response.Body.Close()
    }
    
    if err != nil {
        result["error"] = err.Error()
    }
    
    return result
}

func runDelayTest() map[string]interface{} {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("GET", "/slow", &MockResponse{
        StatusCode: 200,
        Body:       "Delayed response",
        Delay:      100 * time.Millisecond,
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    start := time.Now()
    response, err := client.Get("/slow")
    duration := time.Since(start)
    
    result := map[string]interface{}{
        "test_name":     "Delay Test",
        "success":       err == nil && duration >= 100*time.Millisecond,
        "duration_ms":   duration.Milliseconds(),
        "expected_delay": 100,
    }
    
    if response != nil {
        result["status_code"] = response.StatusCode
        response.Body.Close()
    }
    
    if err != nil {
        result["error"] = err.Error()
    }
    
    return result
}

func runLimitedTest() map[string]interface{} {
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("GET", "/limited", &MockResponse{
        StatusCode: 200,
        Body:       "Limited response",
        Count:      2,
    })
    
    client := NewHTTPClient(mockServer.GetURL())
    
    // Make three requests
    statuses := []int{}
    for i := 0; i < 3; i++ {
        response, err := client.Get("/limited")
        if err == nil {
            statuses = append(statuses, response.StatusCode)
            response.Body.Close()
        }
    }
    
    // Should have 200, 200, 410 (first two succeed, third fails)
    success := len(statuses) == 3 && 
              statuses[0] == 200 && 
              statuses[1] == 200 && 
              statuses[2] == 410
    
    return map[string]interface{}{
        "test_name":     "Limited Response Test",
        "success":       success,
        "status_codes":  statuses,
        "expected":      []int{200, 200, 410},
    }
}

func mockingDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Mocking and Testing Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            .test-pass { color: #28a745; font-weight: bold; }
            .test-fail { color: #dc3545; font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>HTTP Mocking and Testing Demo</h1>
        <p>Hello there! This demonstrates HTTP mocking utilities for testing.</p>
        
        <div class="demo-section">
            <h2>Test Execution</h2>
            <p>Run different types of tests to see mocking in action:</p>
            <button onclick="runTest('get')">GET Request Test</button>
            <button onclick="runTest('post')">POST Request Test</button>
            <button onclick="runTest('error')">Error Handling Test</button>
            <button onclick="runTest('delay')">Response Delay Test</button>
            <button onclick="runTest('limited')">Limited Response Test</button>
            <button onclick="runTest('all')">Run All Tests</button>
            <div id="test-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Mock Server Features</h2>
            <ul>
                <li><strong>Response Configuration:</strong> Set status codes, headers, and body content</li>
                <li><strong>Request Recording:</strong> Capture all incoming requests for verification</li>
                <li><strong>Response Delays:</strong> Simulate network latency and slow services</li>
                <li><strong>Limited Responses:</strong> Control how many times a response can be used</li>
                <li><strong>Wildcard Matching:</strong> Use patterns to match multiple endpoints</li>
                <li><strong>Middleware Support:</strong> Add custom request/response processing</li>
            </ul>
        </div>
        
        <div class="demo-section">
            <h2>Testing Best Practices</h2>
            <pre>
1. Isolation: Each test should use its own mock server
2. Verification: Assert both request and response details
3. Edge Cases: Test error conditions and timeouts
4. Cleanup: Always close mock servers after tests
5. Realistic Data: Use representative test data
6. State Management: Reset mock state between tests

Example Test Structure:
func TestUserService(t *testing.T) {
    // Arrange
    mockServer := NewMockServer()
    defer mockServer.Close()
    
    mockServer.AddResponse("GET", "/users", &MockResponse{
        StatusCode: 200,
        Body: []User{{ID: 1, Name: "Alice"}},
    })
    
    client := NewUserService(mockServer.GetURL())
    
    // Act
    users, err := client.GetUsers()
    
    // Assert
    assert.NoError(t, err)
    assert.Len(t, users, 1)
    assert.Equal(t, "Alice", users[0].Name)
    
    // Verify request was made correctly
    requests := mockServer.GetRequests()
    assert.Len(t, requests, 1)
    assert.Equal(t, "GET", requests[0].Method)
}
            </pre>
        </div>
        
        <script>
        function runTest(type) {
            document.getElementById('test-result').innerHTML = 'Running ' + (type === 'all' ? 'all tests' : type + ' test') + '...';
            
            fetch('/run-tests?type=' + type)
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Test Results</h4>';
                
                data.results.forEach(result => {
                    const statusClass = result.success ? 'test-pass' : 'test-fail';
                    const statusText = result.success ? 'PASS' : 'FAIL';
                    
                    html += '<div style="margin: 10px 0; padding: 10px; background: #fff; border-radius: 3px;">';
                    html += '<h5>' + result.test_name + ' <span class="' + statusClass + '">' + statusText + '</span></h5>';
                    
                    if (result.status_code) {
                        html += '<p>Status Code: ' + result.status_code + '</p>';
                    }
                    
                    if (result.duration_ms) {
                        html += '<p>Duration: ' + result.duration_ms + 'ms</p>';
                    }
                    
                    if (result.status_codes) {
                        html += '<p>Status Codes: [' + result.status_codes.join(', ') + ']</p>';
                        html += '<p>Expected: [' + result.expected.join(', ') + ']</p>';
                    }
                    
                    if (result.error) {
                        html += '<p style="color: red;">Error: ' + result.error + '</p>';
                    }
                    
                    html += '</div>';
                });
                
                document.getElementById('test-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('test-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

func main() {
    http.HandleFunc("/", mockingDemoHandler)
    http.HandleFunc("/run-tests", runTestsHandler)
    
    fmt.Println("=== HTTP Mocking and Testing Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Mock HTTP server creation")
    fmt.Println("• Request recording and verification")
    fmt.Println("• Response configuration and delays")
    fmt.Println("• Limited response usage")
    fmt.Println("• Test utilities and assertions")
    fmt.Println("• Testing best practices")
    fmt.Println()
    fmt.Println("Testing features:")
    fmt.Println("• Response mocking with custom status/headers/body")
    fmt.Println("• Request capture and verification")
    fmt.Println("• Simulated network delays")
    fmt.Println("• Response count limiting")
    fmt.Println("• Wildcard path matching")
    fmt.Println("• Middleware support")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET / - Mocking demo page")
    fmt.Println("  GET /run-tests?type=X - Run specific test type")
    
    // Note: In a real testing scenario, these would be actual unit tests
    // This demo runs the tests through HTTP handlers for demonstration
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This mocking and testing example demonstrates comprehensive HTTP testing  
utilities including mock servers, request recording, response configuration,  
and testing best practices. Proper mocking is essential for reliable unit  
testing of HTTP clients and services.

## HTTP content filtering and validation

Content filtering and validation ensure data integrity and security by  
validating incoming requests and filtering responses. This example  
demonstrates comprehensive validation patterns and content filtering.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "regexp"
    "strconv"
    "strings"
    "time"
)

type ValidationRule struct {
    Field    string
    Required bool
    MinLen   int
    MaxLen   int
    Pattern  *regexp.Regexp
    Custom   func(interface{}) error
}

type ValidationError struct {
    Field   string `json:"field"`
    Message string `json:"message"`
    Value   interface{} `json:"value,omitempty"`
}

type ValidationResult struct {
    Valid  bool               `json:"valid"`
    Errors []ValidationError  `json:"errors"`
}

type ContentFilter struct {
    rules         []ValidationRule
    allowedFields map[string]bool
    blockedWords  []string
    maxSize       int64
}

func NewContentFilter() *ContentFilter {
    return &ContentFilter{
        rules:         make([]ValidationRule, 0),
        allowedFields: make(map[string]bool),
        blockedWords:  make([]string, 0),
        maxSize:       1024 * 1024, // 1MB default
    }
}

func (cf *ContentFilter) AddRule(rule ValidationRule) {
    cf.rules = append(cf.rules, rule)
}

func (cf *ContentFilter) SetAllowedFields(fields []string) {
    cf.allowedFields = make(map[string]bool)
    for _, field := range fields {
        cf.allowedFields[field] = true
    }
}

func (cf *ContentFilter) AddBlockedWords(words []string) {
    cf.blockedWords = append(cf.blockedWords, words...)
}

func (cf *ContentFilter) SetMaxSize(size int64) {
    cf.maxSize = size
}

func (cf *ContentFilter) ValidateRequest(r *http.Request) *ValidationResult {
    result := &ValidationResult{Valid: true, Errors: make([]ValidationError, 0)}
    
    // Check content length
    if r.ContentLength > cf.maxSize {
        result.Valid = false
        result.Errors = append(result.Errors, ValidationError{
            Field:   "content-length",
            Message: fmt.Sprintf("Request too large: %d bytes (max %d)", r.ContentLength, cf.maxSize),
            Value:   r.ContentLength,
        })
    }
    
    // Check content type for JSON requests
    contentType := r.Header.Get("Content-Type")
    if r.Method == "POST" || r.Method == "PUT" || r.Method == "PATCH" {
        if !strings.Contains(contentType, "application/json") {
            result.Valid = false
            result.Errors = append(result.Errors, ValidationError{
                Field:   "content-type",
                Message: "Content-Type must be application/json",
                Value:   contentType,
            })
        }
    }
    
    return result
}

func (cf *ContentFilter) ValidateData(data map[string]interface{}) *ValidationResult {
    result := &ValidationResult{Valid: true, Errors: make([]ValidationError, 0)}
    
    // Check allowed fields
    if len(cf.allowedFields) > 0 {
        for field := range data {
            if !cf.allowedFields[field] {
                result.Valid = false
                result.Errors = append(result.Errors, ValidationError{
                    Field:   field,
                    Message: "Field not allowed",
                    Value:   data[field],
                })
            }
        }
    }
    
    // Apply validation rules
    for _, rule := range cf.rules {
        value, exists := data[rule.Field]
        
        // Check required fields
        if rule.Required && !exists {
            result.Valid = false
            result.Errors = append(result.Errors, ValidationError{
                Field:   rule.Field,
                Message: "Field is required",
            })
            continue
        }
        
        if !exists {
            continue
        }
        
        // Validate string fields
        if strValue, ok := value.(string); ok {
            if err := cf.validateString(rule, strValue); err != nil {
                result.Valid = false
                result.Errors = append(result.Errors, ValidationError{
                    Field:   rule.Field,
                    Message: err.Error(),
                    Value:   value,
                })
            }
        }
        
        // Apply custom validation
        if rule.Custom != nil {
            if err := rule.Custom(value); err != nil {
                result.Valid = false
                result.Errors = append(result.Errors, ValidationError{
                    Field:   rule.Field,
                    Message: err.Error(),
                    Value:   value,
                })
            }
        }
    }
    
    return result
}

func (cf *ContentFilter) validateString(rule ValidationRule, value string) error {
    // Check length constraints
    if rule.MinLen > 0 && len(value) < rule.MinLen {
        return fmt.Errorf("minimum length is %d characters", rule.MinLen)
    }
    
    if rule.MaxLen > 0 && len(value) > rule.MaxLen {
        return fmt.Errorf("maximum length is %d characters", rule.MaxLen)
    }
    
    // Check pattern matching
    if rule.Pattern != nil && !rule.Pattern.MatchString(value) {
        return fmt.Errorf("invalid format")
    }
    
    // Check for blocked words
    lowerValue := strings.ToLower(value)
    for _, word := range cf.blockedWords {
        if strings.Contains(lowerValue, strings.ToLower(word)) {
            return fmt.Errorf("contains prohibited content")
        }
    }
    
    return nil
}

func (cf *ContentFilter) FilterResponse(data map[string]interface{}) map[string]interface{} {
    filtered := make(map[string]interface{})
    
    for key, value := range data {
        // Filter based on allowed fields if configured
        if len(cf.allowedFields) > 0 && !cf.allowedFields[key] {
            continue
        }
        
        // Filter string values for blocked words
        if strValue, ok := value.(string); ok {
            filtered[key] = cf.filterString(strValue)
        } else {
            filtered[key] = value
        }
    }
    
    return filtered
}

func (cf *ContentFilter) filterString(value string) string {
    result := value
    for _, word := range cf.blockedWords {
        // Replace blocked words with asterisks
        pattern := regexp.MustCompile("(?i)" + regexp.QuoteMeta(word))
        replacement := strings.Repeat("*", len(word))
        result = pattern.ReplaceAllString(result, replacement)
    }
    return result
}

// Predefined validation functions
func ValidateEmail(value interface{}) error {
    email, ok := value.(string)
    if !ok {
        return fmt.Errorf("must be a string")
    }
    
    emailPattern := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    if !emailPattern.MatchString(email) {
        return fmt.Errorf("invalid email format")
    }
    
    return nil
}

func ValidateAge(value interface{}) error {
    age, ok := value.(float64) // JSON numbers are float64
    if !ok {
        return fmt.Errorf("must be a number")
    }
    
    if age < 0 || age > 150 {
        return fmt.Errorf("age must be between 0 and 150")
    }
    
    return nil
}

func ValidateURL(value interface{}) error {
    url, ok := value.(string)
    if !ok {
        return fmt.Errorf("must be a string")
    }
    
    urlPattern := regexp.MustCompile(`^https?://[^\s/$.?#].[^\s]*$`)
    if !urlPattern.MatchString(url) {
        return fmt.Errorf("invalid URL format")
    }
    
    return nil
}

func ValidatePhoneNumber(value interface{}) error {
    phone, ok := value.(string)
    if !ok {
        return fmt.Errorf("must be a string")
    }
    
    // Simple phone number pattern
    phonePattern := regexp.MustCompile(`^\+?[\d\s\-\(\)]+$`)
    if !phonePattern.MatchString(phone) {
        return fmt.Errorf("invalid phone number format")
    }
    
    return nil
}

// Middleware for content filtering
func ContentFilterMiddleware(filter *ContentFilter) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            // Validate request
            if validation := filter.ValidateRequest(r); !validation.Valid {
                w.Header().Set("Content-Type", "application/json")
                w.WriteHeader(http.StatusBadRequest)
                json.NewEncoder(w).Encode(map[string]interface{}{
                    "error":      "Request validation failed",
                    "validation": validation,
                })
                return
            }
            
            next.ServeHTTP(w, r)
        })
    }
}

// Sample handlers demonstrating content filtering
func createUserHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    // Parse request body
    var userData map[string]interface{}
    if err := json.NewDecoder(r.Body).Decode(&userData); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    // Create content filter for user creation
    filter := NewContentFilter()
    filter.SetAllowedFields([]string{"name", "email", "age", "bio", "website", "phone"})
    filter.AddBlockedWords([]string{"spam", "hack", "virus", "malware"})
    
    // Add validation rules
    namePattern := regexp.MustCompile(`^[a-zA-Z\s]+$`)
    filter.AddRule(ValidationRule{
        Field:    "name",
        Required: true,
        MinLen:   2,
        MaxLen:   50,
        Pattern:  namePattern,
    })
    
    filter.AddRule(ValidationRule{
        Field:    "email",
        Required: true,
        Custom:   ValidateEmail,
    })
    
    filter.AddRule(ValidationRule{
        Field:  "age",
        Custom: ValidateAge,
    })
    
    filter.AddRule(ValidationRule{
        Field:   "bio",
        MaxLen:  200,
    })
    
    filter.AddRule(ValidationRule{
        Field:  "website",
        Custom: ValidateURL,
    })
    
    filter.AddRule(ValidationRule{
        Field:  "phone",
        Custom: ValidatePhoneNumber,
    })
    
    // Validate data
    validation := filter.ValidateData(userData)
    if !validation.Valid {
        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusBadRequest)
        json.NewEncoder(w).Encode(map[string]interface{}{
            "error":      "Data validation failed",
            "validation": validation,
        })
        return
    }
    
    // Filter and clean data
    cleanedData := filter.FilterResponse(userData)
    
    // Simulate user creation
    user := map[string]interface{}{
        "id":         12345,
        "name":       cleanedData["name"],
        "email":      cleanedData["email"],
        "age":        cleanedData["age"],
        "bio":        cleanedData["bio"],
        "website":    cleanedData["website"],
        "phone":      cleanedData["phone"],
        "created_at": time.Now().Format(time.RFC3339),
        "status":     "active",
    }
    
    response := map[string]interface{}{
        "message":   "Hello there! User created successfully",
        "user":      user,
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(response)
}

func searchUsersHandler(w http.ResponseWriter, r *http.Request) {
    query := r.URL.Query().Get("q")
    page := r.URL.Query().Get("page")
    limit := r.URL.Query().Get("limit")
    
    // Validate query parameters
    filter := NewContentFilter()
    filter.AddBlockedWords([]string{"script", "javascript", "eval", "alert"})
    
    // Validate search query
    if query != "" {
        queryData := map[string]interface{}{"query": query}
        filter.AddRule(ValidationRule{
            Field:   "query",
            MinLen:  1,
            MaxLen:  100,
        })
        
        if validation := filter.ValidateData(queryData); !validation.Valid {
            w.Header().Set("Content-Type", "application/json")
            w.WriteHeader(http.StatusBadRequest)
            json.NewEncoder(w).Encode(map[string]interface{}{
                "error":      "Invalid search query",
                "validation": validation,
            })
            return
        }
        
        // Filter the query
        query = filter.filterString(query)
    }
    
    // Validate pagination parameters
    pageNum := 1
    if page != "" {
        if p, err := strconv.Atoi(page); err != nil || p < 1 {
            http.Error(w, "Invalid page number", http.StatusBadRequest)
            return
        } else {
            pageNum = p
        }
    }
    
    limitNum := 10
    if limit != "" {
        if l, err := strconv.Atoi(limit); err != nil || l < 1 || l > 100 {
            http.Error(w, "Invalid limit (1-100)", http.StatusBadRequest)
            return
        } else {
            limitNum = l
        }
    }
    
    // Mock search results
    users := []map[string]interface{}{
        {"id": 1, "name": "Alice Johnson", "email": "alice@example.com"},
        {"id": 2, "name": "Bob Smith", "email": "bob@example.com"},
        {"id": 3, "name": "Charlie Brown", "email": "charlie@example.com"},
    }
    
    // Filter search results if query provided
    if query != "" {
        filteredUsers := make([]map[string]interface{}, 0)
        for _, user := range users {
            name := user["name"].(string)
            if strings.Contains(strings.ToLower(name), strings.ToLower(query)) {
                filteredUsers = append(filteredUsers, user)
            }
        }
        users = filteredUsers
    }
    
    response := map[string]interface{}{
        "message":    "Hello there! Search completed",
        "query":      query,
        "users":      users,
        "page":       pageNum,
        "limit":      limitNum,
        "total":      len(users),
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func validationDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Content Filtering and Validation Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            textarea { width: 100%; height: 120px; }
            input { margin: 5px; padding: 5px; width: 200px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 300px; }
            .error { color: #dc3545; }
            .success { color: #28a745; }
        </style>
    </head>
    <body>
        <h1>Content Filtering and Validation Demo</h1>
        <p>Hello there! This demonstrates content filtering and validation.</p>
        
        <div class="demo-section">
            <h2>User Creation with Validation</h2>
            <p>Test user creation with various validation rules:</p>
            <textarea id="user-data" placeholder="Enter JSON data for user creation...">
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "age": 25,
  "bio": "Software engineer passionate about Go programming",
  "website": "https://alice.dev",
  "phone": "+1-555-0123"
}
            </textarea>
            <br>
            <button onclick="createUser()">Create User</button>
            <button onclick="testInvalidData()">Test Invalid Data</button>
            <button onclick="testBlockedWords()">Test Blocked Words</button>
            <div id="user-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Search with Content Filtering</h2>
            <p>Test search functionality with query filtering:</p>
            <input type="text" id="search-query" placeholder="Search query..." value="alice">
            <input type="number" id="search-page" placeholder="Page" value="1" min="1">
            <input type="number" id="search-limit" placeholder="Limit" value="10" min="1" max="100">
            <br>
            <button onclick="searchUsers()">Search Users</button>
            <button onclick="testMaliciousQuery()">Test Malicious Query</button>
            <div id="search-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Validation Rules</h2>
            <h4>User Creation Rules:</h4>
            <ul>
                <li><strong>name:</strong> Required, 2-50 characters, letters and spaces only</li>
                <li><strong>email:</strong> Required, valid email format</li>
                <li><strong>age:</strong> Optional, number between 0-150</li>
                <li><strong>bio:</strong> Optional, max 200 characters</li>
                <li><strong>website:</strong> Optional, valid URL format</li>
                <li><strong>phone:</strong> Optional, valid phone number format</li>
            </ul>
            
            <h4>Content Filtering:</h4>
            <ul>
                <li>Blocked words are filtered from all text fields</li>
                <li>Blocked words: spam, hack, virus, malware, script, javascript, eval, alert</li>
                <li>Maximum request size: 1MB</li>
                <li>Only allowed fields are accepted</li>
            </ul>
        </div>
        
        <script>
        function createUser() {
            const userData = document.getElementById('user-data').value;
            
            try {
                JSON.parse(userData); // Validate JSON
            } catch (e) {
                document.getElementById('user-result').innerHTML = 
                    '<div class="error">Invalid JSON: ' + e.message + '</div>';
                return;
            }
            
            document.getElementById('user-result').innerHTML = 'Creating user...';
            
            fetch('/create-user', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: userData
            })
            .then(response => response.json())
            .then(data => {
                let html = '<h4>User Creation Result</h4>';
                if (data.error) {
                    html += '<div class="error">Error: ' + data.error + '</div>';
                    if (data.validation && data.validation.errors) {
                        html += '<h5>Validation Errors:</h5><ul>';
                        data.validation.errors.forEach(error => {
                            html += '<li class="error">' + error.field + ': ' + error.message + '</li>';
                        });
                        html += '</ul>';
                    }
                } else {
                    html += '<div class="success">User created successfully!</div>';
                    html += '<pre>' + JSON.stringify(data.user, null, 2) + '</pre>';
                }
                document.getElementById('user-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('user-result').innerHTML = 
                    '<div class="error">Request failed: ' + error + '</div>';
            });
        }
        
        function testInvalidData() {
            const invalidData = {
                "name": "A", // Too short
                "email": "invalid-email", // Invalid format
                "age": -5, // Invalid age
                "bio": "A".repeat(250), // Too long
                "website": "not-a-url", // Invalid URL
                "phone": "abc123", // Invalid phone
                "unauthorized_field": "This field is not allowed"
            };
            
            document.getElementById('user-data').value = JSON.stringify(invalidData, null, 2);
        }
        
        function testBlockedWords() {
            const blockedData = {
                "name": "Spam User",
                "email": "user@example.com",
                "bio": "I love to hack systems and create virus software with malware"
            };
            
            document.getElementById('user-data').value = JSON.stringify(blockedData, null, 2);
        }
        
        function searchUsers() {
            const query = document.getElementById('search-query').value;
            const page = document.getElementById('search-page').value;
            const limit = document.getElementById('search-limit').value;
            
            let url = '/search-users?';
            if (query) url += 'q=' + encodeURIComponent(query) + '&';
            if (page) url += 'page=' + page + '&';
            if (limit) url += 'limit=' + limit;
            
            document.getElementById('search-result').innerHTML = 'Searching...';
            
            fetch(url)
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Search Results</h4>';
                if (data.error) {
                    html += '<div class="error">Error: ' + data.error + '</div>';
                } else {
                    html += '<p>Query: "' + data.query + '" | Results: ' + data.total + '</p>';
                    html += '<pre>' + JSON.stringify(data.users, null, 2) + '</pre>';
                }
                document.getElementById('search-result').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('search-result').innerHTML = 
                    '<div class="error">Search failed: ' + error + '</div>';
            });
        }
        
        function testMaliciousQuery() {
            document.getElementById('search-query').value = '<script>alert("XSS")</script>';
        }
        </script>
    </body>
    </html>`)
}

func main() {
    // Create global content filter for requests
    globalFilter := NewContentFilter()
    globalFilter.SetMaxSize(1024 * 1024) // 1MB limit
    
    http.HandleFunc("/", validationDemoHandler)
    
    // Apply content filter middleware to API endpoints
    http.Handle("/create-user", ContentFilterMiddleware(globalFilter)(http.HandlerFunc(createUserHandler)))
    http.HandleFunc("/search-users", searchUsersHandler)
    
    fmt.Println("=== HTTP Content Filtering and Validation Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Request validation with custom rules")
    fmt.Println("• Content filtering and blocked word detection")
    fmt.Println("• Field validation (email, URL, phone, etc.)")
    fmt.Println("• Query parameter validation")
    fmt.Println("• Response content filtering")
    fmt.Println("• Security-focused input sanitization")
    fmt.Println()
    fmt.Println("Validation types:")
    fmt.Println("• Required field validation")
    fmt.Println("• String length constraints")
    fmt.Println("• Pattern matching (regex)")
    fmt.Println("• Custom validation functions")
    fmt.Println("• Allowed field whitelisting")
    fmt.Println("• Blocked word filtering")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Validation demo page")
    fmt.Println("  POST /create-user - User creation with validation")
    fmt.Println("  GET  /search-users - Search with content filtering")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This content filtering and validation example demonstrates comprehensive  
input validation, content filtering, and security measures for HTTP APIs.  
Proper validation and filtering are essential for data integrity, security,  
and preventing malicious input from compromising your application.

## HTTP server-sent events (SSE) advanced implementation

Server-sent events enable real-time data streaming from server to clients.  
This advanced example demonstrates comprehensive SSE implementation with  
connection management, event types, and broadcasting capabilities.

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "strconv"
    "sync"
    "time"
)

type SSEEvent struct {
    ID    string      `json:"id,omitempty"`
    Event string      `json:"event,omitempty"`
    Data  interface{} `json:"data"`
    Retry int         `json:"retry,omitempty"`
}

type SSEClient struct {
    ID       string
    Channel  chan SSEEvent
    Request  *http.Request
    Writer   http.ResponseWriter
    LastSeen time.Time
    Topics   map[string]bool
}

type SSEManager struct {
    clients    map[string]*SSEClient
    topics     map[string]map[string]*SSEClient
    register   chan *SSEClient
    unregister chan *SSEClient
    broadcast  chan SSEMessage
    mu         sync.RWMutex
}

type SSEMessage struct {
    Topic string
    Event SSEEvent
}

func NewSSEManager() *SSEManager {
    manager := &SSEManager{
        clients:    make(map[string]*SSEClient),
        topics:     make(map[string]map[string]*SSEClient),
        register:   make(chan *SSEClient),
        unregister: make(chan *SSEClient),
        broadcast:  make(chan SSEMessage),
    }
    
    go manager.run()
    return manager
}

func (m *SSEManager) run() {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case client := <-m.register:
            m.addClient(client)
            log.Printf("SSE client connected: %s", client.ID)
            
            // Send welcome message
            welcomeEvent := SSEEvent{
                ID:    fmt.Sprintf("welcome_%d", time.Now().UnixNano()),
                Event: "welcome",
                Data: map[string]interface{}{
                    "message":    "Hello there! Connected to SSE stream",
                    "client_id":  client.ID,
                    "server_time": time.Now().Format(time.RFC3339),
                },
            }
            select {
            case client.Channel <- welcomeEvent:
            default:
                close(client.Channel)
            }
            
        case client := <-m.unregister:
            m.removeClient(client)
            log.Printf("SSE client disconnected: %s", client.ID)
            
        case message := <-m.broadcast:
            m.broadcastToTopic(message.Topic, message.Event)
            
        case <-ticker.C:
            m.sendHeartbeat()
            m.cleanupStaleClients()
        }
    }
}

func (m *SSEManager) addClient(client *SSEClient) {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    m.clients[client.ID] = client
    
    // Subscribe to topics
    for topic := range client.Topics {
        if m.topics[topic] == nil {
            m.topics[topic] = make(map[string]*SSEClient)
        }
        m.topics[topic][client.ID] = client
    }
}

func (m *SSEManager) removeClient(client *SSEClient) {
    m.mu.Lock()
    defer m.mu.Unlock()
    
    delete(m.clients, client.ID)
    
    // Unsubscribe from topics
    for topic := range client.Topics {
        if clients, exists := m.topics[topic]; exists {
            delete(clients, client.ID)
            if len(clients) == 0 {
                delete(m.topics, topic)
            }
        }
    }
    
    close(client.Channel)
}

func (m *SSEManager) broadcastToTopic(topic string, event SSEEvent) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    if clients, exists := m.topics[topic]; exists {
        for _, client := range clients {
            select {
            case client.Channel <- event:
                client.LastSeen = time.Now()
            default:
                // Client channel is full, remove client
                go func(c *SSEClient) {
                    m.unregister <- c
                }(client)
            }
        }
    }
}

func (m *SSEManager) sendHeartbeat() {
    heartbeatEvent := SSEEvent{
        ID:    fmt.Sprintf("heartbeat_%d", time.Now().UnixNano()),
        Event: "heartbeat",
        Data: map[string]interface{}{
            "timestamp": time.Now().Format(time.RFC3339),
            "server_uptime": time.Since(startTime).String(),
        },
    }
    
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    for _, client := range m.clients {
        select {
        case client.Channel <- heartbeatEvent:
            client.LastSeen = time.Now()
        default:
            // Client not responding, will be cleaned up
        }
    }
}

func (m *SSEManager) cleanupStaleClients() {
    cutoff := time.Now().Add(-2 * time.Minute)
    
    m.mu.RLock()
    staleClients := make([]*SSEClient, 0)
    for _, client := range m.clients {
        if client.LastSeen.Before(cutoff) {
            staleClients = append(staleClients, client)
        }
    }
    m.mu.RUnlock()
    
    for _, client := range staleClients {
        m.unregister <- client
    }
}

func (m *SSEManager) GetStats() map[string]interface{} {
    m.mu.RLock()
    defer m.mu.RUnlock()
    
    topicStats := make(map[string]int)
    for topic, clients := range m.topics {
        topicStats[topic] = len(clients)
    }
    
    return map[string]interface{}{
        "total_clients":    len(m.clients),
        "total_topics":     len(m.topics),
        "topic_subscribers": topicStats,
        "server_uptime":    time.Since(startTime).String(),
    }
}

var (
    sseManager *SSEManager
    startTime  time.Time
)

func sseHandler(w http.ResponseWriter, r *http.Request) {
    // Check if client supports SSE
    if r.Header.Get("Accept") != "text/event-stream" {
        http.Error(w, "SSE not supported", http.StatusNotAcceptable)
        return
    }
    
    // Set SSE headers
    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")
    w.Header().Set("Access-Control-Allow-Origin", "*")
    w.Header().Set("Access-Control-Allow-Headers", "Cache-Control")
    
    // Get client preferences
    topics := make(map[string]bool)
    if topicParam := r.URL.Query().Get("topics"); topicParam != "" {
        for _, topic := range []string{topicParam} {
            topics[topic] = true
        }
    } else {
        // Default topics
        topics["general"] = true
        topics["updates"] = true
    }
    
    // Create client
    clientID := fmt.Sprintf("client_%d", time.Now().UnixNano())
    client := &SSEClient{
        ID:       clientID,
        Channel:  make(chan SSEEvent, 100),
        Request:  r,
        Writer:   w,
        LastSeen: time.Now(),
        Topics:   topics,
    }
    
    // Register client
    sseManager.register <- client
    
    // Setup cleanup on disconnect
    defer func() {
        sseManager.unregister <- client
    }()
    
    // Handle client disconnect
    notify := r.Context().Done()
    
    // Send events to client
    for {
        select {
        case event := <-client.Channel:
            if err := writeSSEEvent(w, event); err != nil {
                log.Printf("Error writing SSE event: %v", err)
                return
            }
            
        case <-notify:
            log.Printf("SSE client disconnected: %s", clientID)
            return
        }
    }
}

func writeSSEEvent(w http.ResponseWriter, event SSEEvent) error {
    if event.ID != "" {
        fmt.Fprintf(w, "id: %s\n", event.ID)
    }
    
    if event.Event != "" {
        fmt.Fprintf(w, "event: %s\n", event.Event)
    }
    
    if event.Retry > 0 {
        fmt.Fprintf(w, "retry: %d\n", event.Retry)
    }
    
    // Handle data serialization
    var dataStr string
    switch data := event.Data.(type) {
    case string:
        dataStr = data
    case []byte:
        dataStr = string(data)
    default:
        jsonData, err := json.Marshal(data)
        if err != nil {
            return err
        }
        dataStr = string(jsonData)
    }
    
    fmt.Fprintf(w, "data: %s\n\n", dataStr)
    
    // Flush the data to client
    if flusher, ok := w.(http.Flusher); ok {
        flusher.Flush()
    }
    
    return nil
}

func broadcastHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var request struct {
        Topic   string      `json:"topic"`
        Event   string      `json:"event"`
        Data    interface{} `json:"data"`
        Message string      `json:"message"`
    }
    
    if err := json.NewDecoder(r.Body).Decode(&request); err != nil {
        http.Error(w, "Invalid JSON", http.StatusBadRequest)
        return
    }
    
    if request.Topic == "" {
        request.Topic = "general"
    }
    
    if request.Event == "" {
        request.Event = "message"
    }
    
    if request.Data == nil && request.Message != "" {
        request.Data = map[string]interface{}{
            "message":   request.Message,
            "timestamp": time.Now().Format(time.RFC3339),
        }
    }
    
    event := SSEEvent{
        ID:    fmt.Sprintf("broadcast_%d", time.Now().UnixNano()),
        Event: request.Event,
        Data:  request.Data,
    }
    
    message := SSEMessage{
        Topic: request.Topic,
        Event: event,
    }
    
    sseManager.broadcast <- message
    
    response := map[string]interface{}{
        "message":    "Hello there! Event broadcasted successfully",
        "topic":      request.Topic,
        "event_type": request.Event,
        "event_id":   event.ID,
        "timestamp":  time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func sseStatsHandler(w http.ResponseWriter, r *http.Request) {
    stats := sseManager.GetStats()
    stats["message"] = "SSE server statistics"
    stats["timestamp"] = time.Now().Format(time.RFC3339)
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(stats)
}

// Simulate real-time events
func startEventGenerators() {
    // Stock price updates
    go func() {
        ticker := time.NewTicker(5 * time.Second)
        defer ticker.Stop()
        
        stocks := []string{"AAPL", "GOOGL", "MSFT", "AMZN", "TSLA"}
        
        for range ticker.C {
            stock := stocks[time.Now().Unix()%int64(len(stocks))]
            price := 100 + float64(time.Now().Unix()%200)
            change := (float64(time.Now().Unix()%20) - 10) / 10.0
            
            event := SSEEvent{
                ID:    fmt.Sprintf("stock_%d", time.Now().UnixNano()),
                Event: "stock_update",
                Data: map[string]interface{}{
                    "symbol": stock,
                    "price":  price,
                    "change": change,
                    "timestamp": time.Now().Format(time.RFC3339),
                },
            }
            
            sseManager.broadcast <- SSEMessage{Topic: "stocks", Event: event}
        }
    }()
    
    // News updates
    go func() {
        ticker := time.NewTicker(10 * time.Second)
        defer ticker.Stop()
        
        news := []string{
            "Go 1.22 released with new features",
            "HTTP/3 adoption continues to grow",
            "WebAssembly support improved in latest browsers",
            "Cloud computing trends for 2024",
            "New security best practices published",
        }
        
        for range ticker.C {
            article := news[time.Now().Unix()%int64(len(news))]
            
            event := SSEEvent{
                ID:    fmt.Sprintf("news_%d", time.Now().UnixNano()),
                Event: "news_update",
                Data: map[string]interface{}{
                    "headline": article,
                    "category": "technology",
                    "timestamp": time.Now().Format(time.RFC3339),
                },
            }
            
            sseManager.broadcast <- SSEMessage{Topic: "news", Event: event}
        }
    }()
    
    // System metrics
    go func() {
        ticker := time.NewTicker(3 * time.Second)
        defer ticker.Stop()
        
        for range ticker.C {
            event := SSEEvent{
                ID:    fmt.Sprintf("metrics_%d", time.Now().UnixNano()),
                Event: "system_metrics",
                Data: map[string]interface{}{
                    "cpu_usage":    float64(time.Now().Unix()%100),
                    "memory_usage": float64(time.Now().Unix()%100),
                    "connections":  len(sseManager.clients),
                    "timestamp":    time.Now().Format(time.RFC3339),
                },
            }
            
            sseManager.broadcast <- SSEMessage{Topic: "metrics", Event: event}
        }
    }()
}

func sseDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>Advanced Server-Sent Events Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            .events-container { height: 300px; overflow-y: auto; border: 1px solid #ddd; padding: 10px; background: white; }
            .event-item { margin: 5px 0; padding: 8px; border-radius: 3px; }
            .event-welcome { background: #d4edda; }
            .event-heartbeat { background: #f8f9fa; }
            .event-stock_update { background: #fff3cd; }
            .event-news_update { background: #d1ecf1; }
            .event-system_metrics { background: #e2e3e5; }
            .event-message { background: #d1e7dd; }
            button { margin: 5px; padding: 10px 15px; }
            input, select { margin: 5px; padding: 5px; }
            .status-connected { color: #28a745; font-weight: bold; }
            .status-disconnected { color: #dc3545; font-weight: bold; }
            .stats { background: #fff; padding: 10px; border-radius: 3px; margin: 10px 0; }
        </style>
    </head>
    <body>
        <h1>Advanced Server-Sent Events Demo</h1>
        <p>Hello there! This demonstrates real-time SSE with multiple event types.</p>
        
        <div class="demo-section">
            <h2>Connection Control</h2>
            <label>Topics: 
                <select id="topics">
                    <option value="general">General</option>
                    <option value="stocks">Stock Updates</option>
                    <option value="news">News Updates</option>
                    <option value="metrics">System Metrics</option>
                </select>
            </label>
            <button onclick="connect()">Connect</button>
            <button onclick="disconnect()">Disconnect</button>
            <button onclick="clearEvents()">Clear Events</button>
            <span id="connection-status" class="status-disconnected">Disconnected</span>
        </div>
        
        <div class="demo-section">
            <h2>Broadcast Message</h2>
            <input type="text" id="broadcast-topic" placeholder="Topic" value="general">
            <input type="text" id="broadcast-event" placeholder="Event type" value="message">
            <input type="text" id="broadcast-message" placeholder="Message" value="Hello everyone!">
            <button onclick="broadcastMessage()">Send Broadcast</button>
        </div>
        
        <div class="demo-section">
            <h2>Server Statistics</h2>
            <button onclick="getStats()">Get Stats</button>
            <button onclick="startStatsPolling()">Start Auto-Update</button>
            <button onclick="stopStatsPolling()">Stop Auto-Update</button>
            <div id="stats-display" class="stats">Click "Get Stats" to load...</div>
        </div>
        
        <div class="demo-section">
            <h2>Real-time Events</h2>
            <div id="events" class="events-container">
                <p>Connect to start receiving events...</p>
            </div>
        </div>
        
        <script>
        let eventSource = null;
        let statsInterval = null;
        
        function connect() {
            if (eventSource) {
                eventSource.close();
            }
            
            const topic = document.getElementById('topics').value;
            eventSource = new EventSource('/sse?topics=' + topic);
            
            eventSource.onopen = function() {
                document.getElementById('connection-status').textContent = 'Connected';
                document.getElementById('connection-status').className = 'status-connected';
                addEvent('system', 'Connected to SSE stream for topic: ' + topic, 'info');
            };
            
            eventSource.onerror = function() {
                document.getElementById('connection-status').textContent = 'Connection Error';
                document.getElementById('connection-status').className = 'status-disconnected';
                addEvent('system', 'Connection error occurred', 'error');
            };
            
            eventSource.onmessage = function(event) {
                try {
                    const data = JSON.parse(event.data);
                    addEvent('message', data, 'message');
                } catch (e) {
                    addEvent('message', event.data, 'message');
                }
            };
            
            // Specific event handlers
            eventSource.addEventListener('welcome', function(event) {
                const data = JSON.parse(event.data);
                addEvent('welcome', data, 'welcome');
            });
            
            eventSource.addEventListener('heartbeat', function(event) {
                const data = JSON.parse(event.data);
                addEvent('heartbeat', 'Heartbeat: ' + data.timestamp, 'heartbeat');
            });
            
            eventSource.addEventListener('stock_update', function(event) {
                const data = JSON.parse(event.data);
                addEvent('stock_update', 
                        data.symbol + ': $' + data.price + ' (' + 
                        (data.change >= 0 ? '+' : '') + data.change + ')', 'stock_update');
            });
            
            eventSource.addEventListener('news_update', function(event) {
                const data = JSON.parse(event.data);
                addEvent('news_update', data.headline, 'news_update');
            });
            
            eventSource.addEventListener('system_metrics', function(event) {
                const data = JSON.parse(event.data);
                addEvent('system_metrics', 
                        'CPU: ' + data.cpu_usage + '% | Memory: ' + data.memory_usage + '% | Connections: ' + data.connections, 
                        'system_metrics');
            });
        }
        
        function disconnect() {
            if (eventSource) {
                eventSource.close();
                eventSource = null;
            }
            document.getElementById('connection-status').textContent = 'Disconnected';
            document.getElementById('connection-status').className = 'status-disconnected';
            addEvent('system', 'Disconnected from SSE stream', 'info');
        }
        
        function addEvent(type, data, eventClass) {
            const eventsContainer = document.getElementById('events');
            const eventDiv = document.createElement('div');
            eventDiv.className = 'event-item event-' + eventClass;
            
            const timestamp = new Date().toLocaleTimeString();
            const content = typeof data === 'string' ? data : JSON.stringify(data);
            
            eventDiv.innerHTML = '<strong>[' + timestamp + '] ' + type + ':</strong> ' + content;
            
            eventsContainer.appendChild(eventDiv);
            eventsContainer.scrollTop = eventsContainer.scrollHeight;
            
            // Limit events to prevent memory issues
            while (eventsContainer.children.length > 100) {
                eventsContainer.removeChild(eventsContainer.firstChild);
            }
        }
        
        function clearEvents() {
            document.getElementById('events').innerHTML = '';
        }
        
        function broadcastMessage() {
            const topic = document.getElementById('broadcast-topic').value;
            const event = document.getElementById('broadcast-event').value;
            const message = document.getElementById('broadcast-message').value;
            
            fetch('/broadcast', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    topic: topic,
                    event: event,
                    message: message
                })
            })
            .then(response => response.json())
            .then(data => {
                addEvent('broadcast_sent', 'Message sent to ' + topic + ' topic', 'info');
            })
            .catch(error => {
                addEvent('broadcast_error', 'Failed to send message: ' + error, 'error');
            });
        }
        
        function getStats() {
            fetch('/sse-stats')
            .then(response => response.json())
            .then(data => {
                let html = '<h4>Server Statistics</h4>';
                html += '<p>Total Clients: ' + data.total_clients + '</p>';
                html += '<p>Total Topics: ' + data.total_topics + '</p>';
                html += '<p>Server Uptime: ' + data.server_uptime + '</p>';
                html += '<h5>Topic Subscribers:</h5>';
                html += '<ul>';
                for (let topic in data.topic_subscribers) {
                    html += '<li>' + topic + ': ' + data.topic_subscribers[topic] + ' subscribers</li>';
                }
                html += '</ul>';
                html += '<p><small>Last updated: ' + data.timestamp + '</small></p>';
                
                document.getElementById('stats-display').innerHTML = html;
            })
            .catch(error => {
                document.getElementById('stats-display').innerHTML = 
                    '<p style="color: red;">Error loading stats: ' + error + '</p>';
            });
        }
        
        function startStatsPolling() {
            if (statsInterval) return;
            
            statsInterval = setInterval(getStats, 5000);
            getStats(); // Initial load
        }
        
        function stopStatsPolling() {
            if (statsInterval) {
                clearInterval(statsInterval);
                statsInterval = null;
            }
        }
        
        // Auto-connect on page load
        window.addEventListener('load', function() {
            connect();
        });
        
        // Cleanup on page unload
        window.addEventListener('beforeunload', function() {
            disconnect();
        });
        </script>
    </body>
    </html>`)
}

func main() {
    startTime = time.Now()
    sseManager = NewSSEManager()
    
    // Start background event generators
    startEventGenerators()
    
    http.HandleFunc("/", sseDemoHandler)
    http.HandleFunc("/sse", sseHandler)
    http.HandleFunc("/broadcast", broadcastHandler)
    http.HandleFunc("/sse-stats", sseStatsHandler)
    
    fmt.Println("=== Advanced Server-Sent Events Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Real-time event streaming with SSE")
    fmt.Println("• Multiple event types and topics")
    fmt.Println("• Client connection management")
    fmt.Println("• Broadcasting to specific topics")
    fmt.Println("• Automatic heartbeat and cleanup")
    fmt.Println("• Connection statistics and monitoring")
    fmt.Println()
    fmt.Println("Event types:")
    fmt.Println("• welcome - Client connection acknowledgment")
    fmt.Println("• heartbeat - Keep-alive messages")
    fmt.Println("• stock_update - Simulated stock prices")
    fmt.Println("• news_update - Simulated news articles")
    fmt.Println("• system_metrics - Server performance data")
    fmt.Println("• message - Custom broadcast messages")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - SSE demo interface")
    fmt.Println("  GET  /sse?topics=X - SSE event stream")
    fmt.Println("  POST /broadcast - Send broadcast message")
    fmt.Println("  GET  /sse-stats - Connection statistics")
    
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This advanced SSE example demonstrates comprehensive real-time streaming  
including connection management, topic-based broadcasting, multiple event  
types, and client lifecycle handling. SSE is perfect for real-time  
applications like dashboards, notifications, and live data feeds.

## HTTP request/response logging and debugging

Comprehensive logging and debugging are essential for monitoring HTTP  
applications and troubleshooting issues. This example demonstrates  
advanced logging patterns, request tracing, and debugging utilities.

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "log"
    "net/http"
    "os"
    "runtime"
    "strconv"
    "strings"
    "time"
)

type LogLevel int

const (
    LogLevelDebug LogLevel = iota
    LogLevelInfo
    LogLevelWarn
    LogLevelError
)

func (l LogLevel) String() string {
    switch l {
    case LogLevelDebug:
        return "DEBUG"
    case LogLevelInfo:
        return "INFO"
    case LogLevelWarn:
        return "WARN"
    case LogLevelError:
        return "ERROR"
    default:
        return "UNKNOWN"
    }
}

type HTTPLogger struct {
    logger    *log.Logger
    level     LogLevel
    logBodies bool
    maxBodySize int
}

func NewHTTPLogger(output io.Writer, level LogLevel) *HTTPLogger {
    return &HTTPLogger{
        logger:      log.New(output, "", 0),
        level:       level,
        logBodies:   true,
        maxBodySize: 1024, // 1KB default
    }
}

func (hl *HTTPLogger) SetLogBodies(enabled bool) {
    hl.logBodies = enabled
}

func (hl *HTTPLogger) SetMaxBodySize(size int) {
    hl.maxBodySize = size
}

func (hl *HTTPLogger) log(level LogLevel, format string, args ...interface{}) {
    if level < hl.level {
        return
    }
    
    timestamp := time.Now().Format("2006-01-02 15:04:05.000")
    prefix := fmt.Sprintf("[%s] %s ", timestamp, level.String())
    
    message := fmt.Sprintf(format, args...)
    hl.logger.Printf("%s%s", prefix, message)
}

func (hl *HTTPLogger) Debug(format string, args ...interface{}) {
    hl.log(LogLevelDebug, format, args...)
}

func (hl *HTTPLogger) Info(format string, args ...interface{}) {
    hl.log(LogLevelInfo, format, args...)
}

func (hl *HTTPLogger) Warn(format string, args ...interface{}) {
    hl.log(LogLevelWarn, format, args...)
}

func (hl *HTTPLogger) Error(format string, args ...interface{}) {
    hl.log(LogLevelError, format, args...)
}

type RequestLogEntry struct {
    Timestamp    time.Time         `json:"timestamp"`
    Method       string            `json:"method"`
    URL          string            `json:"url"`
    Proto        string            `json:"proto"`
    Headers      map[string]string `json:"headers"`
    Body         string            `json:"body,omitempty"`
    RemoteAddr   string            `json:"remote_addr"`
    UserAgent    string            `json:"user_agent"`
    RequestSize  int64             `json:"request_size"`
}

type ResponseLogEntry struct {
    Timestamp    time.Time         `json:"timestamp"`
    StatusCode   int               `json:"status_code"`
    Headers      map[string]string `json:"headers"`
    Body         string            `json:"body,omitempty"`
    ResponseSize int64             `json:"response_size"`
    Duration     time.Duration     `json:"duration"`
}

type LoggingResponseWriter struct {
    http.ResponseWriter
    statusCode   int
    body         *bytes.Buffer
    size         int64
}

func NewLoggingResponseWriter(w http.ResponseWriter) *LoggingResponseWriter {
    return &LoggingResponseWriter{
        ResponseWriter: w,
        statusCode:     200,
        body:          new(bytes.Buffer),
    }
}

func (lrw *LoggingResponseWriter) WriteHeader(statusCode int) {
    lrw.statusCode = statusCode
    lrw.ResponseWriter.WriteHeader(statusCode)
}

func (lrw *LoggingResponseWriter) Write(data []byte) (int, error) {
    size, err := lrw.ResponseWriter.Write(data)
    lrw.size += int64(size)
    
    // Capture response body for logging
    lrw.body.Write(data)
    
    return size, err
}

func LoggingMiddleware(logger *HTTPLogger) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            
            // Log request
            requestEntry := hl.logRequest(r, logger)
            
            // Wrap response writer
            lrw := NewLoggingResponseWriter(w)
            
            // Execute handler
            next.ServeHTTP(lrw, r)
            
            // Log response
            responseEntry := hl.logResponse(lrw, start, logger)
            
            // Log combined entry
            hl.logCombinedEntry(requestEntry, responseEntry, logger)
        })
    }
}

func (hl *HTTPLogger) logRequest(r *http.Request, logger *HTTPLogger) *RequestLogEntry {
    headers := make(map[string]string)
    for name, values := range r.Header {
        headers[name] = strings.Join(values, ", ")
    }
    
    var body string
    if hl.logBodies && r.Body != nil {
        bodyBytes, err := io.ReadAll(r.Body)
        if err == nil {
            r.Body = io.NopCloser(bytes.NewReader(bodyBytes))
            
            if len(bodyBytes) <= hl.maxBodySize {
                body = string(bodyBytes)
            } else {
                body = fmt.Sprintf("[Body too large: %d bytes]", len(bodyBytes))
            }
        }
    }
    
    entry := &RequestLogEntry{
        Timestamp:   time.Now(),
        Method:      r.Method,
        URL:         r.URL.String(),
        Proto:       r.Proto,
        Headers:     headers,
        Body:        body,
        RemoteAddr:  r.RemoteAddr,
        UserAgent:   r.Header.Get("User-Agent"),
        RequestSize: r.ContentLength,
    }
    
    logger.Info("REQUEST: %s %s from %s", r.Method, r.URL.Path, r.RemoteAddr)
    if body != "" {
        logger.Debug("Request body: %s", body)
    }
    
    return entry
}

func (hl *HTTPLogger) logResponse(lrw *LoggingResponseWriter, start time.Time, logger *HTTPLogger) *ResponseLogEntry {
    duration := time.Since(start)
    
    headers := make(map[string]string)
    for name, values := range lrw.Header() {
        headers[name] = strings.Join(values, ", ")
    }
    
    var body string
    if hl.logBodies && lrw.body.Len() > 0 {
        if lrw.body.Len() <= hl.maxBodySize {
            body = lrw.body.String()
        } else {
            body = fmt.Sprintf("[Body too large: %d bytes]", lrw.body.Len())
        }
    }
    
    entry := &ResponseLogEntry{
        Timestamp:    time.Now(),
        StatusCode:   lrw.statusCode,
        Headers:      headers,
        Body:         body,
        ResponseSize: lrw.size,
        Duration:     duration,
    }
    
    statusLevel := LogLevelInfo
    if lrw.statusCode >= 400 {
        statusLevel = LogLevelWarn
    }
    if lrw.statusCode >= 500 {
        statusLevel = LogLevelError
    }
    
    logger.log(statusLevel, "RESPONSE: %d (%s) in %v", 
              lrw.statusCode, http.StatusText(lrw.statusCode), duration)
    
    if body != "" {
        logger.Debug("Response body: %s", body)
    }
    
    return entry
}

func (hl *HTTPLogger) logCombinedEntry(req *RequestLogEntry, resp *ResponseLogEntry, logger *HTTPLogger) {
    // Apache Combined Log Format
    // LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
    
    timestamp := req.Timestamp.Format("02/Jan/2006:15:04:05 -0700")
    referer := req.Headers["Referer"]
    if referer == "" {
        referer = "-"
    }
    
    userAgent := req.UserAgent
    if userAgent == "" {
        userAgent = "-"
    }
    
    combinedLog := fmt.Sprintf(`%s - - [%s] "%s %s %s" %d %d "%s" "%s"`,
        req.RemoteAddr,
        timestamp,
        req.Method,
        req.URL,
        req.Proto,
        resp.StatusCode,
        resp.ResponseSize,
        referer,
        userAgent)
    
    logger.Info("COMBINED: %s", combinedLog)
}

// Debug utilities
func debugHandler(w http.ResponseWriter, r *http.Request) {
    debugInfo := map[string]interface{}{
        "message":   "Hello there! Debug information",
        "timestamp": time.Now().Format(time.RFC3339),
        "request": map[string]interface{}{
            "method":        r.Method,
            "url":           r.URL.String(),
            "proto":         r.Proto,
            "content_length": r.ContentLength,
            "remote_addr":   r.RemoteAddr,
            "host":          r.Host,
            "request_uri":   r.RequestURI,
        },
        "headers": r.Header,
        "query_params": r.URL.Query(),
        "server_info": map[string]interface{}{
            "go_version":   runtime.Version(),
            "num_cpu":      runtime.NumCPU(),
            "num_goroutine": runtime.NumGoroutine(),
        },
    }
    
    // Add body if present
    if r.Body != nil {
        body, err := io.ReadAll(r.Body)
        if err == nil {
            r.Body = io.NopCloser(bytes.NewReader(body))
            debugInfo["body"] = string(body)
        }
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(debugInfo)
}

func slowHandler(w http.ResponseWriter, r *http.Request) {
    delayParam := r.URL.Query().Get("delay")
    delay := 1000 // Default 1 second
    
    if delayParam != "" {
        if d, err := strconv.Atoi(delayParam); err == nil {
            delay = d
        }
    }
    
    httpLogger.Info("Slow handler: sleeping for %dms", delay)
    time.Sleep(time.Duration(delay) * time.Millisecond)
    
    response := map[string]interface{}{
        "message":   "Hello there! Slow response completed",
        "delay_ms":  delay,
        "timestamp": time.Now().Format(time.RFC3339),
    }
    
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(response)
}

func errorHandler(w http.ResponseWriter, r *http.Request) {
    errorCode := r.URL.Query().Get("code")
    code := 500 // Default to internal server error
    
    if errorCode != "" {
        if c, err := strconv.Atoi(errorCode); err == nil {
            code = c
        }
    }
    
    httpLogger.Error("Intentional error response: %d", code)
    
    errorResponse := map[string]interface{}{
        "error":      http.StatusText(code),
        "code":       code,
        "message":    "Hello there! This is an intentional error for testing",
        "timestamp":  time.Now().Format(time.RFC3339),
        "request_id": fmt.Sprintf("req_%d", time.Now().UnixNano()),
    }
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(code)
    json.NewEncoder(w).Encode(errorResponse)
}

func loggingDemoHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "text/html")
    fmt.Fprint(w, `
    <!DOCTYPE html>
    <html>
    <head>
        <title>HTTP Logging and Debugging Demo</title>
        <style>
            body { font-family: Arial, sans-serif; margin: 20px; }
            .demo-section { background: #f8f9fa; padding: 20px; margin: 20px 0; border-radius: 5px; }
            button { margin: 5px; padding: 10px 15px; }
            .result { margin: 10px 0; padding: 10px; border: 1px solid #ddd; border-radius: 3px; }
            textarea { width: 100%; height: 120px; }
            input, select { margin: 5px; padding: 5px; }
            pre { background: #f8f9fa; padding: 10px; border-radius: 3px; overflow-x: auto; max-height: 400px; }
            .log-debug { color: #6c757d; }
            .log-info { color: #007cba; }
            .log-warn { color: #ffc107; }
            .log-error { color: #dc3545; }
        </style>
    </head>
    <body>
        <h1>HTTP Logging and Debugging Demo</h1>
        <p>Hello there! This demonstrates comprehensive HTTP logging and debugging.</p>
        
        <div class="demo-section">
            <h2>Request Testing</h2>
            <p>Test different types of requests to see logging in action:</p>
            <button onclick="testDebugInfo()">Get Debug Info</button>
            <button onclick="testSlowRequest()">Slow Request (2s)</button>
            <button onclick="testError(400)">Client Error (400)</button>
            <button onclick="testError(500)">Server Error (500)</button>
            <div id="request-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>POST Request Testing</h2>
            <p>Test POST requests with body logging:</p>
            <textarea id="post-data" placeholder="Enter JSON data...">
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "message": "Hello there! This is a test message."
}
            </textarea>
            <br>
            <button onclick="sendPostRequest()">Send POST Request</button>
            <div id="post-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Custom Request Testing</h2>
            <p>Customize request parameters:</p>
            <select id="method">
                <option value="GET">GET</option>
                <option value="POST">POST</option>
                <option value="PUT">PUT</option>
                <option value="DELETE">DELETE</option>
            </select>
            <input type="text" id="path" placeholder="Path" value="/debug">
            <input type="text" id="delay" placeholder="Delay (ms)" value="0">
            <br>
            <button onclick="sendCustomRequest()">Send Custom Request</button>
            <div id="custom-result" class="result"></div>
        </div>
        
        <div class="demo-section">
            <h2>Logging Information</h2>
            <h4>Log Levels:</h4>
            <ul>
                <li><span class="log-debug">DEBUG</span> - Detailed information for debugging</li>
                <li><span class="log-info">INFO</span> - General information about requests/responses</li>
                <li><span class="log-warn">WARN</span> - Warning conditions (4xx responses)</li>
                <li><span class="log-error">ERROR</span> - Error conditions (5xx responses)</li>
            </ul>
            
            <h4>Logged Information:</h4>
            <ul>
                <li>Request: Method, URL, headers, body, remote address</li>
                <li>Response: Status code, headers, body, response time</li>
                <li>Combined: Apache combined log format</li>
                <li>Debug: Server info, runtime statistics</li>
            </ul>
            
            <h4>Console Output:</h4>
            <p>Check the server console to see detailed logging output including:</p>
            <ul>
                <li>Request/response details</li>
                <li>Timing information</li>
                <li>Body content (if enabled)</li>
                <li>Apache combined log format</li>
                <li>Debug and error information</li>
            </ul>
        </div>
        
        <script>
        function testDebugInfo() {
            document.getElementById('request-result').innerHTML = 'Getting debug info...';
            
            fetch('/debug', {
                headers: {
                    'X-Custom-Header': 'debug-test',
                    'X-Client-Info': 'Demo Client v1.0'
                }
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('request-result').innerHTML = 
                    '<h4>Debug Information</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('request-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testSlowRequest() {
            document.getElementById('request-result').innerHTML = 'Sending slow request...';
            const start = Date.now();
            
            fetch('/slow?delay=2000')
            .then(response => response.json())
            .then(data => {
                const duration = Date.now() - start;
                document.getElementById('request-result').innerHTML = 
                    '<h4>Slow Request Result</h4>' +
                    '<p>Duration: ' + duration + 'ms</p>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('request-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function testError(code) {
            document.getElementById('request-result').innerHTML = 'Testing error response...';
            
            fetch('/error?code=' + code)
            .then(response => response.json())
            .then(data => {
                document.getElementById('request-result').innerHTML = 
                    '<h4>Error Response (Status: ' + data.code + ')</h4>' +
                    '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('request-result').innerHTML = 
                    '<h4 style="color: red;">Request Error: ' + error + '</h4>';
            });
        }
        
        function sendPostRequest() {
            const data = document.getElementById('post-data').value;
            
            try {
                JSON.parse(data); // Validate JSON
            } catch (e) {
                document.getElementById('post-result').innerHTML = 
                    '<h4 style="color: red;">Invalid JSON: ' + e.message + '</h4>';
                return;
            }
            
            document.getElementById('post-result').innerHTML = 'Sending POST request...';
            
            fetch('/debug', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'X-Request-Type': 'demo-post'
                },
                body: data
            })
            .then(response => response.json())
            .then(data => {
                document.getElementById('post-result').innerHTML = 
                    '<h4>POST Request Result</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('post-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        
        function sendCustomRequest() {
            const method = document.getElementById('method').value;
            const path = document.getElementById('path').value;
            const delay = document.getElementById('delay').value;
            
            let url = path;
            if (delay && parseInt(delay) > 0) {
                url += (path.includes('?') ? '&' : '?') + 'delay=' + delay;
            }
            
            document.getElementById('custom-result').innerHTML = 'Sending ' + method + ' request...';
            
            const options = {
                method: method,
                headers: {
                    'X-Custom-Test': 'true',
                    'X-Method': method
                }
            };
            
            if (method === 'POST' || method === 'PUT') {
                options.headers['Content-Type'] = 'application/json';
                options.body = JSON.stringify({
                    test: true,
                    method: method,
                    timestamp: new Date().toISOString()
                });
            }
            
            fetch(url, options)
            .then(response => response.json())
            .then(data => {
                document.getElementById('custom-result').innerHTML = 
                    '<h4>Custom Request Result</h4><pre>' + JSON.stringify(data, null, 2) + '</pre>';
            })
            .catch(error => {
                document.getElementById('custom-result').innerHTML = 
                    '<h4 style="color: red;">Error: ' + error + '</h4>';
            });
        }
        </script>
    </body>
    </html>`)
}

var httpLogger *HTTPLogger

func main() {
    // Create HTTP logger
    httpLogger = NewHTTPLogger(os.Stdout, LogLevelDebug)
    httpLogger.SetLogBodies(true)
    httpLogger.SetMaxBodySize(2048)
    
    // Routes with logging middleware
    mux := http.NewServeMux()
    mux.HandleFunc("/", loggingDemoHandler)
    mux.HandleFunc("/debug", debugHandler)
    mux.HandleFunc("/slow", slowHandler)
    mux.HandleFunc("/error", errorHandler)
    
    // Apply logging middleware
    loggedMux := LoggingMiddleware(httpLogger)(mux)
    
    fmt.Println("=== HTTP Logging and Debugging Demo ===")
    fmt.Println("Server starting on http://localhost:8080")
    fmt.Println()
    fmt.Println("Features demonstrated:")
    fmt.Println("• Comprehensive request/response logging")
    fmt.Println("• Multiple log levels (DEBUG, INFO, WARN, ERROR)")
    fmt.Println("• Request and response body logging")
    fmt.Println("• Apache combined log format")
    fmt.Println("• Performance timing and metrics")
    fmt.Println("• Debug information and server stats")
    fmt.Println()
    fmt.Println("Logging configuration:")
    fmt.Println("• Log level: DEBUG (all messages)")
    fmt.Println("• Body logging: Enabled")
    fmt.Println("• Max body size: 2048 bytes")
    fmt.Println("• Output: Console (stdout)")
    fmt.Println()
    fmt.Println("Endpoints:")
    fmt.Println("  GET  / - Logging demo interface")
    fmt.Println("  GET  /debug - Debug information")
    fmt.Println("  GET  /slow?delay=N - Slow response (N milliseconds)")
    fmt.Println("  GET  /error?code=N - Error response (status code N)")
    fmt.Println()
    fmt.Println("Watch the console output for detailed logging information!")
    
    log.Fatal(http.ListenAndServe(":8080", loggedMux))
}
```

This logging and debugging example demonstrates comprehensive HTTP monitoring  
including request/response logging, multiple log levels, body capture, and  
debug utilities. Proper logging is essential for monitoring application  
behavior, troubleshooting issues, and maintaining system health.
