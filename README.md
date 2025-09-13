# Go examples


## Version

```
$ go version
go version go1.22.2 linux/amd64
```

## Create new module 

```
$ go mod init com.zetcode/simple
```

We create a new module with `go mod init` command. It produces a `go.mod` file.

## Shorthand variable declaration 


```go
package main

import "fmt"

func main() {

    name := "John Doe"
    age := 34

    fmt.Printf("%s is %d years old\n", name, age)
}
```

## Constants

Constants are created with the `const` keyword. They cannot be modified.  

```go
package main

import "fmt"

func main() {

    var age int = 34
    const WIDTH = 100

    age = 35
    age = 36

    // WIDTH = 101

    fmt.Println(age, WIDTH)
}
```

## Type inference

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {

    var name = "John Doe"
    var age = 34

    fmt.Println(reflect.TypeOf(name))
    fmt.Println(reflect.TypeOf(age))

    fmt.Printf("%s is %d years old\n", name, age)
}
```


## Swapping values in function

We use pointers for this.  

```go
package main

import (
    "fmt"
)

func main() {

    var x int = 5
    var y int = 8
	
    fmt.Println(x, y)
    swap(&x, &y)
    fmt.Println(x, y)
}

func swap(x, y *int) {
    var temp int = *x
    *x = *y
    *y = temp
}
```

## Builder pattern 

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "time"
)

type httpClientBuilder struct {
    timeout time.Duration
}

func NewBuilder() *httpClientBuilder {
    return &httpClientBuilder{timeout: 5 * time.Second}
}

func (b *httpClientBuilder) Build() *http.Client {
    return &http.Client{Timeout: b.timeout}
}

func (b *httpClientBuilder) Timeout(t time.Duration) *httpClientBuilder {
    b.timeout = t
    return b
}

func main() {

    client := NewBuilder().Timeout(5 * time.Second).Build()

    url := "https://webcode.me"
    res, err := client.Head(url)

    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(res.Status)
}
```

## The any type

The `any` type is a built-in alias for the interface{} type, which can hold any value. Introduced in Go 1.18,  
`any` provides a more intuitive and readable way to indicate that a variable can be of `any` type.  

```go
package main

import "fmt"

func main() {

    var val any

    val = "hello"
    fmt.Printf("%T\n", val)

    val = 3
    fmt.Printf("%T\n", val)

    val = 4.5
    fmt.Printf("%T\n", val)

    PrintValues(42, "hello", 3.14, true)
}

func PrintValues(values ...any) {
    for _, value := range values {
        fmt.Println(value)
    }
}
```

When dealing with values of type any, you often use type assertions or reflection to  
handle the specific underlying type.  

```go
package main

import "fmt"

func main() {

    PrintStrings(42, "hello", 3.14, "book", true, "falcon")
}

func PrintStrings(values ...any) {

    for _, val := range values {

        if str, ok := val.(string); ok {
            fmt.Println(str)
        }
    }
}
```

Using generics:  

```go
package main

import (
    "fmt"   
)

// PrintAll is a generic function that prints all elements in a slice
func PrintAll[T any](items []T) {
    for _, item := range items {
        fmt.Println(item)
    }
}

func main() {
    // Create slices of different types
    intSlice := []int{1, 2, 3, 4, 5}
    stringSlice := []string{"apple", "banana", "cherry"}
    floatSlice := []float64{1.1, 2.2, 3.3}

    // Use the generic function to print slices of different types
    fmt.Println("Integers:")
    PrintAll(intSlice)

    fmt.Println("\nStrings:")
    PrintAll(stringSlice)

    fmt.Println("\nFloats:")
    PrintAll(floatSlice)
}
```

## DeepSeek via API

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
)

type ChatRequest struct {
    Model    string    `json:"model"`
    Messages []Message `json:"messages"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type ChatResponse struct {
    Choices []Choice `json:"choices"`
}

type Choice struct {
    Message Message `json:"message"`
}

func main() {
    apiKey := os.Getenv("DEEPSEEK_API_KEY")
    if apiKey == "" {
        fmt.Println("Please set the DEEPSEEK_API_KEY environment variable")
        return
    }

    prompt := "What is NetBSD in a sentence?"
    request := ChatRequest{
        Model: "deepseek-chat",
        Messages: []Message{
            {Role: "user", Content: prompt},
        },
    }

    jsonData, err := json.Marshal(request)
    if err != nil {
        fmt.Printf("Error marshaling request: %v\n", err)
        return
    }

    req, err := http.NewRequest("POST", "https://api.deepseek.com/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error sending request: %v\n", err)
        return
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response: %v\n", err)
        return
    }

    var chatResp ChatResponse
    if err := json.Unmarshal(body, &chatResp); err != nil {
        fmt.Printf("Error unmarshaling response: %v\n", err)
        return
    }

    if len(chatResp.Choices) > 0 {
        fmt.Printf("Response: %s\n", chatResp.Choices[0].Message.Content)
    } else {
        fmt.Println("No content in response")
    }
}
```

## Tool call with DeepSeek API

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "os"
    "time"
)

type ChatRequest struct {
    Model    string    `json:"model"`
    Messages []Message `json:"messages"`
    Tools    []Tool    `json:"tools,omitempty"`
}

type Message struct {
    Role      string     `json:"role"`
    Content   string     `json:"content"`
    ToolCalls []ToolCall `json:"tool_calls,omitempty"`
}

type ChatResponse struct {
    Choices []Choice `json:"choices"`
}

type Choice struct {
    Message Message `json:"message"`
}

type Tool struct {
    Type     string   `json:"type"`
    Function Function `json:"function"`
}

type Function struct {
    Name        string                 `json:"name"`
    Description string                 `json:"description"`
    Parameters  map[string]interface{} `json:"parameters"`
}

type ToolCall struct {
    ID       string   `json:"id"`
    Type     string   `json:"type"`
    Function Function `json:"function"`
}

func getCurrentDateTime() string {
    return time.Now().Format("2006-01-02 15:04:05")
}

func main() {
    apiKey := os.Getenv("DEEPSEEK_API_KEY")
    if apiKey == "" {
        fmt.Println("Please set the DEEPSEEK_API_KEY environment variable")
        return
    }

    prompt := "Who was Napolen. What time is it?"
    tool := Tool{
        Type: "function",
        Function: Function{
            Name:        "get_current_datetime",
            Description: "Get the current date and time",
            Parameters: map[string]interface{}{
                "type":       "object",
                "properties": map[string]interface{}{},
            },
        },
    }
    request := ChatRequest{
        Model: "deepseek-chat",
        Messages: []Message{
            {Role: "user", Content: prompt},
        },
        Tools: []Tool{tool},
    }

    jsonData, err := json.Marshal(request)
    if err != nil {
        fmt.Printf("Error marshaling request: %v\n", err)
        return
    }

    req, err := http.NewRequest("POST", "https://api.deepseek.com/v1/chat/completions", bytes.NewBuffer(jsonData))
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }

    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("Authorization", "Bearer "+apiKey)

    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error sending request: %v\n", err)
        return
    }
    defer resp.Body.Close()

    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response: %v\n", err)
        return
    }

    var chatResp ChatResponse
    if err := json.Unmarshal(body, &chatResp); err != nil {
        fmt.Printf("Error unmarshaling response: %v\n", err)
        return
    }

    if len(chatResp.Choices) > 0 {
        message := chatResp.Choices[0].Message
        if message.Content != "" {
            fmt.Printf("Response: %s\n", message.Content)
        }
        if len(message.ToolCalls) > 0 {
            for _, toolCall := range message.ToolCalls {
                if toolCall.Function.Name == "get_current_datetime" {
                    result := getCurrentDateTime()
                    fmt.Printf("Tool call result: %s\n", result)
                }
            }
        }
    } else {
        fmt.Println("No content in response")
    }

    fmt.Println("Request completed at:", time.Now().Format(time.RFC1123))
}
```
