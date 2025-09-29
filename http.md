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
