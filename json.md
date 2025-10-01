# JSON

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange  
format that is easy for humans to read and write, and easy for machines to parse  
and generate. In Go, the `encoding/json` package provides powerful and efficient  
tools for working with JSON data, making it the de facto standard for data  
serialization in web APIs, configuration files, and data storage.

Go's JSON support is built around the concepts of marshaling (encoding Go data  
structures to JSON) and unmarshaling (decoding JSON to Go data structures). The  
package uses reflection to automatically map between Go structs and JSON objects,  
with struct tags providing fine-grained control over the serialization process.  
This approach allows developers to work with strongly-typed data structures while  
seamlessly converting to and from JSON format.

The `encoding/json` package supports all standard JSON types: objects (mapped to  
structs or maps), arrays (mapped to slices or arrays), strings, numbers, booleans,  
and null values. Go provides several ways to handle JSON: the `Marshal` and  
`Unmarshal` functions for working with byte slices, the `Encoder` and `Decoder`  
types for streaming JSON data, and custom `MarshalJSON` and `UnmarshalJSON`  
methods for implementing specialized serialization logic. Understanding these  
tools and patterns is essential for building robust Go applications.

## Basic JSON marshaling

Marshaling converts Go data structures into JSON format using struct tags.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name  string `json:"name"`
    Age   int    `json:"age"`
    Email string `json:"email"`
}

func main() {
    person := Person{
        Name:  "Alice Johnson",
        Age:   28,
        Email: "alice@example.com",
    }
    
    jsonData, err := json.Marshal(person)
    if err != nil {
        fmt.Printf("Error marshaling: %v\n", err)
        return
    }
    
    fmt.Printf("JSON: %s\n", string(jsonData))
}
```

The `json.Marshal` function converts Go values to JSON-encoded byte slices. Struct  
tags like `json:"name"` specify how field names appear in the JSON output. Only  
exported (capitalized) struct fields are included in the JSON output.  

## Pretty-print JSON

Pretty-printing formats JSON with indentation for better readability.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Product struct {
    ID          int     `json:"id"`
    Name        string  `json:"name"`
    Price       float64 `json:"price"`
    InStock     bool    `json:"in_stock"`
    Description string  `json:"description"`
}

func main() {
    product := Product{
        ID:          101,
        Name:        "Laptop",
        Price:       999.99,
        InStock:     true,
        Description: "High-performance laptop",
    }
    
    prettyJSON, err := json.MarshalIndent(product, "", "  ")
    if err != nil {
        fmt.Printf("Error marshaling: %v\n", err)
        return
    }
    
    fmt.Println(string(prettyJSON))
}
```

The `MarshalIndent` function adds newlines and indentation to JSON output. The  
first string parameter is the prefix for each line, and the second is the  
indentation string. This is useful for debugging and human-readable output.  

## Basic JSON unmarshaling

Unmarshaling converts JSON data back into Go data structures.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    Username string `json:"username"`
    Email    string `json:"email"`
    Active   bool   `json:"active"`
}

func main() {
    jsonString := `{"username":"john_doe","email":"john@example.com","active":true}`
    
    var user User
    err := json.Unmarshal([]byte(jsonString), &user)
    if err != nil {
        fmt.Printf("Error unmarshaling: %v\n", err)
        return
    }
    
    fmt.Printf("Username: %s\n", user.Username)
    fmt.Printf("Email: %s\n", user.Email)
    fmt.Printf("Active: %t\n", user.Active)
}
```

The `json.Unmarshal` function parses JSON data and stores the result in the value  
pointed to by the second parameter. The JSON field names are matched to struct  
field tags, or to field names if no tag is present.  

## Nested structs with JSON

Complex data structures with nested objects can be serialized naturally.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Address struct {
    Street  string `json:"street"`
    City    string `json:"city"`
    ZipCode string `json:"zip_code"`
}

type Employee struct {
    ID       int     `json:"id"`
    Name     string  `json:"name"`
    Position string  `json:"position"`
    Address  Address `json:"address"`
}

func main() {
    employee := Employee{
        ID:       123,
        Name:     "Sarah Connor",
        Position: "Senior Developer",
        Address: Address{
            Street:  "456 Tech Street",
            City:    "San Francisco",
            ZipCode: "94102",
        },
    }
    
    jsonData, _ := json.MarshalIndent(employee, "", "  ")
    fmt.Println("Marshaled:")
    fmt.Println(string(jsonData))
    
    var decoded Employee
    json.Unmarshal(jsonData, &decoded)
    fmt.Printf("\nDecoded: %+v\n", decoded)
}
```

Nested structs are automatically handled by the JSON encoder and decoder. Each  
struct maintains its own field tags, allowing for complete control over the JSON  
structure. This pattern is ideal for representing complex hierarchical data.  

## Slices and arrays in JSON

Go slices and arrays map to JSON arrays seamlessly.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Team struct {
    Name    string   `json:"name"`
    Members []string `json:"members"`
    Scores  []int    `json:"scores"`
}

func main() {
    team := Team{
        Name:    "Development Team",
        Members: []string{"Alice", "Bob", "Charlie"},
        Scores:  []int{95, 87, 92},
    }
    
    jsonData, _ := json.MarshalIndent(team, "", "  ")
    fmt.Println("JSON with slices:")
    fmt.Println(string(jsonData))
    
    jsonString := `{
        "name": "Sales Team",
        "members": ["David", "Emma", "Frank"],
        "scores": [88, 91, 85]
    }`
    
    var decodedTeam Team
    json.Unmarshal([]byte(jsonString), &decodedTeam)
    fmt.Printf("\nDecoded team: %+v\n", decodedTeam)
}
```

Slices in Go structs are encoded as JSON arrays. When unmarshaling, the JSON  
decoder automatically creates slices of the appropriate size. Empty slices are  
encoded as empty arrays, while nil slices are encoded as `null`.  

## Maps with JSON

Go maps can be marshaled to JSON objects with string keys.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    // Simple map
    config := map[string]string{
        "host":     "localhost",
        "port":     "8080",
        "protocol": "https",
    }
    
    jsonData, _ := json.MarshalIndent(config, "", "  ")
    fmt.Println("Config map:")
    fmt.Println(string(jsonData))
    
    // Map with interface{} values
    settings := map[string]interface{}{
        "timeout":    30,
        "enabled":    true,
        "server":     "api.example.com",
        "maxRetries": 3,
    }
    
    jsonData, _ = json.MarshalIndent(settings, "", "  ")
    fmt.Println("\nSettings map:")
    fmt.Println(string(jsonData))
    
    // Unmarshal to map
    var decoded map[string]interface{}
    json.Unmarshal(jsonData, &decoded)
    fmt.Printf("\nDecoded: %+v\n", decoded)
}
```

Maps with string keys marshal to JSON objects. When the value type is  
`interface{}`, the encoder determines the JSON type based on the actual value.  
Numbers are decoded as float64, so type assertions may be needed.  

## Omitempty tag

The `omitempty` tag excludes zero values from JSON output.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type OptionalFields struct {
    Name     string  `json:"name"`
    Age      int     `json:"age,omitempty"`
    Email    string  `json:"email,omitempty"`
    Phone    string  `json:"phone,omitempty"`
    Salary   float64 `json:"salary,omitempty"`
    IsActive bool    `json:"is_active,omitempty"`
}

func main() {
    // All fields populated
    full := OptionalFields{
        Name:     "John Smith",
        Age:      35,
        Email:    "john@example.com",
        Phone:    "555-0123",
        Salary:   75000.00,
        IsActive: true,
    }
    
    jsonData, _ := json.MarshalIndent(full, "", "  ")
    fmt.Println("Full object:")
    fmt.Println(string(jsonData))
    
    // Partial fields (zero values omitted)
    partial := OptionalFields{
        Name:  "Jane Doe",
        Email: "jane@example.com",
    }
    
    jsonData, _ = json.MarshalIndent(partial, "", "  ")
    fmt.Println("\nPartial object (with omitempty):")
    fmt.Println(string(jsonData))
}
```

The `omitempty` tag instructs the encoder to skip fields with zero values (empty  
strings, 0 for numbers, false for booleans, nil for pointers/slices/maps). This  
is useful for creating compact JSON output and representing optional fields.  

## Pointer fields for nullable values

Pointers allow distinguishing between zero values and missing values.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Account struct {
    Username string   `json:"username"`
    Email    *string  `json:"email,omitempty"`
    Age      *int     `json:"age,omitempty"`
    Verified *bool    `json:"verified,omitempty"`
    Balance  *float64 `json:"balance,omitempty"`
}

func main() {
    email := "user@example.com"
    age := 0 // Zero age is valid
    verified := false
    
    account := Account{
        Username: "testuser",
        Email:    &email,
        Age:      &age,      // Will be included even though it's 0
        Verified: &verified, // Will be included even though it's false
        Balance:  nil,       // Will be omitted
    }
    
    jsonData, _ := json.MarshalIndent(account, "", "  ")
    fmt.Println("Account with pointer fields:")
    fmt.Println(string(jsonData))
    
    // Unmarshal back
    var decoded Account
    json.Unmarshal(jsonData, &decoded)
    
    if decoded.Age != nil {
        fmt.Printf("\nAge is present: %d\n", *decoded.Age)
    }
    if decoded.Balance == nil {
        fmt.Println("Balance is not present (nil)")
    }
}
```

Pointer fields enable true nullable values in JSON. A nil pointer marshals to  
`null` or is omitted with `omitempty`. This pattern distinguishes between "not  
provided" (nil) and "provided as zero value" (pointer to zero).  

## Renaming and ignoring fields

Struct tags control field names and visibility in JSON.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Secret struct {
    PublicID     string `json:"id"`
    UserName     string `json:"username"`
    PasswordHash string `json:"-"`              // Completely ignored
    APIKey       string `json:"api_key"`
    internal     string // Unexported, ignored
    Metadata     string `json:"meta,omitempty"`
}

func main() {
    secret := Secret{
        PublicID:     "user-123",
        UserName:     "alice",
        PasswordHash: "secret-hash-never-serialized",
        APIKey:       "key-abc-xyz",
        internal:     "not exported",
        Metadata:     "",
    }
    
    jsonData, _ := json.MarshalIndent(secret, "", "  ")
    fmt.Println("Serialized (note missing fields):")
    fmt.Println(string(jsonData))
}
```

The `json:"-"` tag completely excludes a field from JSON processing. Field names  
in tags can differ from struct field names. Unexported fields (lowercase) are  
automatically ignored. This provides security and flexibility in data exposure.  

## Custom types with JSON

Custom types can be serialized by implementing standard interfaces.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strings"
)

type Status int

const (
    StatusPending Status = iota
    StatusActive
    StatusInactive
    StatusSuspended
)

func (s Status) String() string {
    switch s {
    case StatusPending:
        return "pending"
    case StatusActive:
        return "active"
    case StatusInactive:
        return "inactive"
    case StatusSuspended:
        return "suspended"
    default:
        return "unknown"
    }
}

func (s Status) MarshalJSON() ([]byte, error) {
    return json.Marshal(s.String())
}

func (s *Status) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    
    switch strings.ToLower(str) {
    case "pending":
        *s = StatusPending
    case "active":
        *s = StatusActive
    case "inactive":
        *s = StatusInactive
    case "suspended":
        *s = StatusSuspended
    default:
        *s = StatusPending
    }
    return nil
}

type UserAccount struct {
    ID     int    `json:"id"`
    Name   string `json:"name"`
    Status Status `json:"status"`
}

func main() {
    user := UserAccount{
        ID:     42,
        Name:   "Bob Wilson",
        Status: StatusActive,
    }
    
    jsonData, _ := json.MarshalIndent(user, "", "  ")
    fmt.Println("Custom type serialization:")
    fmt.Println(string(jsonData))
    
    jsonString := `{"id":43,"name":"Carol White","status":"inactive"}`
    var decoded UserAccount
    json.Unmarshal([]byte(jsonString), &decoded)
    fmt.Printf("\nDecoded: %+v\n", decoded)
    fmt.Printf("Status enum value: %d (%s)\n", decoded.Status, decoded.Status)
}
```

Custom types implement `MarshalJSON` and `UnmarshalJSON` to control their JSON  
representation. This is useful for enums, special formatting, or converting  
between different representations during serialization.  

## Working with time.Time

Time values require special handling in JSON serialization.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type Event struct {
    Name      string    `json:"name"`
    Timestamp time.Time `json:"timestamp"`
    Duration  int64     `json:"duration_seconds"`
}

func main() {
    event := Event{
        Name:      "Conference",
        Timestamp: time.Now(),
        Duration:  3600,
    }
    
    jsonData, _ := json.MarshalIndent(event, "", "  ")
    fmt.Println("Event with time:")
    fmt.Println(string(jsonData))
    
    var decoded Event
    json.Unmarshal(jsonData, &decoded)
    fmt.Printf("\nDecoded time: %v\n", decoded.Timestamp)
    fmt.Printf("Formatted: %s\n", decoded.Timestamp.Format(time.RFC3339))
}
```

Go's `time.Time` implements `MarshalJSON` and `UnmarshalJSON`, serializing to  
RFC3339 format by default. This format is widely compatible and includes timezone  
information. Custom time formats require implementing custom marshaling methods.  

## Custom time format

Custom marshaling methods enable specific time formats.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

type CustomTime struct {
    time.Time
}

func (ct CustomTime) MarshalJSON() ([]byte, error) {
    formatted := ct.Format("2006-01-02 15:04:05")
    return json.Marshal(formatted)
}

func (ct *CustomTime) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    
    parsed, err := time.Parse("2006-01-02 15:04:05", str)
    if err != nil {
        return err
    }
    
    ct.Time = parsed
    return nil
}

type LogEntry struct {
    Message   string     `json:"message"`
    Timestamp CustomTime `json:"timestamp"`
    Level     string     `json:"level"`
}

func main() {
    entry := LogEntry{
        Message:   "Application started",
        Timestamp: CustomTime{time.Now()},
        Level:     "info",
    }
    
    jsonData, _ := json.MarshalIndent(entry, "", "  ")
    fmt.Println("Custom time format:")
    fmt.Println(string(jsonData))
    
    var decoded LogEntry
    json.Unmarshal(jsonData, &decoded)
    fmt.Printf("\nDecoded timestamp: %v\n", decoded.Timestamp.Time)
}
```

Creating a wrapper type around `time.Time` allows custom formatting while  
preserving the underlying time functionality. This pattern is useful when working  
with APIs that require specific date/time formats.  

## JSON with embedded structs

Embedded structs can flatten or nest JSON structure.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Timestamps struct {
    CreatedAt string `json:"created_at"`
    UpdatedAt string `json:"updated_at"`
}

type Metadata struct {
    Version string `json:"version"`
    Author  string `json:"author"`
}

// Flattened structure (embedded without tag)
type Article struct {
    ID      int    `json:"id"`
    Title   string `json:"title"`
    Content string `json:"content"`
    Timestamps      // Fields are flattened
}

// Nested structure (embedded with tag)
type Document struct {
    ID       int      `json:"id"`
    Name     string   `json:"name"`
    Metadata Metadata `json:"metadata"` // Fields are nested
}

func main() {
    article := Article{
        ID:      1,
        Title:   "Go JSON Guide",
        Content: "Comprehensive examples",
        Timestamps: Timestamps{
            CreatedAt: "2024-01-15T10:00:00Z",
            UpdatedAt: "2024-01-16T12:30:00Z",
        },
    }
    
    jsonData, _ := json.MarshalIndent(article, "", "  ")
    fmt.Println("Flattened embedded struct:")
    fmt.Println(string(jsonData))
    
    doc := Document{
        ID:   2,
        Name: "Project Spec",
        Metadata: Metadata{
            Version: "1.0",
            Author:  "Team",
        },
    }
    
    jsonData, _ = json.MarshalIndent(doc, "", "  ")
    fmt.Println("\nNested embedded struct:")
    fmt.Println(string(jsonData))
}
```

Embedded structs without JSON tags have their fields promoted to the parent  
level, creating a flat JSON structure. Embedded structs with JSON tags create  
nested objects. This provides flexibility in JSON structure design.  

## JSON streaming with encoder

JSON encoders write directly to io.Writer for efficient streaming.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "strings"
)

type Record struct {
    ID    int    `json:"id"`
    Value string `json:"value"`
}

func main() {
    var buffer strings.Builder
    encoder := json.NewEncoder(&buffer)
    
    records := []Record{
        {ID: 1, Value: "First"},
        {ID: 2, Value: "Second"},
        {ID: 3, Value: "Third"},
    }
    
    for _, record := range records {
        err := encoder.Encode(record)
        if err != nil {
            fmt.Printf("Error encoding: %v\n", err)
            continue
        }
    }
    
    fmt.Println("Encoded stream:")
    fmt.Print(buffer.String())
    
    // Encode to stdout
    fmt.Println("\nEncoding to stdout:")
    stdEncoder := json.NewEncoder(os.Stdout)
    stdEncoder.SetIndent("", "  ")
    stdEncoder.Encode(Record{ID: 4, Value: "Fourth"})
}
```

The `json.Encoder` type writes JSON values directly to an `io.Writer`, avoiding  
intermediate byte slices. This is more efficient for large datasets or streaming  
scenarios. Each `Encode` call writes a complete JSON value.  

## JSON streaming with decoder

JSON decoders read directly from io.Reader for efficient parsing.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strings"
)

type Item struct {
    Name     string  `json:"name"`
    Quantity int     `json:"quantity"`
    Price    float64 `json:"price"`
}

func main() {
    jsonStream := `
    {"name":"Widget","quantity":10,"price":9.99}
    {"name":"Gadget","quantity":5,"price":19.99}
    {"name":"Doohickey","quantity":15,"price":4.99}
    `
    
    decoder := json.NewDecoder(strings.NewReader(jsonStream))
    
    fmt.Println("Decoding stream:")
    count := 0
    for decoder.More() {
        var item Item
        err := decoder.Decode(&item)
        if err != nil {
            fmt.Printf("Error decoding: %v\n", err)
            break
        }
        
        count++
        fmt.Printf("Item %d: %s (qty: %d, price: $%.2f)\n",
            count, item.Name, item.Quantity, item.Price)
    }
}
```

The `json.Decoder` type reads and parses JSON values from an `io.Reader`. The  
`More` method checks if there's more data to decode. This is ideal for processing  
large JSON files or streaming data without loading everything into memory.  

## Unknown fields with map

Maps can capture arbitrary JSON structures when the schema is unknown.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

func main() {
    jsonString := `{
        "name": "Dynamic Data",
        "type": "example",
        "count": 42,
        "active": true,
        "tags": ["go", "json", "dynamic"],
        "nested": {
            "key1": "value1",
            "key2": 123
        }
    }`
    
    var data map[string]interface{}
    err := json.Unmarshal([]byte(jsonString), &data)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Println("Parsed dynamic JSON:")
    for key, value := range data {
        fmt.Printf("  %s: %v (type: %T)\n", key, value, value)
    }
    
    // Accessing nested data requires type assertions
    if nested, ok := data["nested"].(map[string]interface{}); ok {
        fmt.Println("\nNested object:")
        for k, v := range nested {
            fmt.Printf("  %s: %v\n", k, v)
        }
    }
}
```

Using `map[string]interface{}` allows unmarshaling arbitrary JSON when the  
structure isn't known at compile time. Numbers unmarshal as float64, requiring  
type assertions for specific numeric types.  

## JSON RawMessage

RawMessage defers JSON decoding for flexible processing.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Envelope struct {
    Type    string          `json:"type"`
    Payload json.RawMessage `json:"payload"`
}

type UserPayload struct {
    Username string `json:"username"`
    Email    string `json:"email"`
}

type ProductPayload struct {
    SKU   string  `json:"sku"`
    Price float64 `json:"price"`
}

func main() {
    messages := []string{
        `{"type":"user","payload":{"username":"alice","email":"alice@example.com"}}`,
        `{"type":"product","payload":{"sku":"WIDGET-001","price":29.99}}`,
    }
    
    for i, msg := range messages {
        var env Envelope
        err := json.Unmarshal([]byte(msg), &env)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            continue
        }
        
        fmt.Printf("\nMessage %d (type: %s):\n", i+1, env.Type)
        
        switch env.Type {
        case "user":
            var user UserPayload
            json.Unmarshal(env.Payload, &user)
            fmt.Printf("  User: %s (%s)\n", user.Username, user.Email)
        case "product":
            var product ProductPayload
            json.Unmarshal(env.Payload, &product)
            fmt.Printf("  Product: %s ($%.2f)\n", product.SKU, product.Price)
        }
    }
}
```

`json.RawMessage` stores raw JSON bytes without parsing, allowing deferred or  
conditional decoding. This is useful for polymorphic messages or when you need  
to inspect part of the JSON before deciding how to parse the rest.  

## Handling JSON arrays at root

JSON arrays at the root level are decoded into slices.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Task struct {
    ID       int    `json:"id"`
    Title    string `json:"title"`
    Complete bool   `json:"complete"`
}

func main() {
    jsonArray := `[
        {"id":1,"title":"Write code","complete":true},
        {"id":2,"title":"Test code","complete":false},
        {"id":3,"title":"Deploy code","complete":false}
    ]`
    
    var tasks []Task
    err := json.Unmarshal([]byte(jsonArray), &tasks)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Println("Tasks:")
    for _, task := range tasks {
        status := "incomplete"
        if task.Complete {
            status = "complete"
        }
        fmt.Printf("  [%d] %s (%s)\n", task.ID, task.Title, status)
    }
    
    // Marshal back to JSON array
    jsonData, _ := json.MarshalIndent(tasks, "", "  ")
    fmt.Println("\nMarshaled array:")
    fmt.Println(string(jsonData))
}
```

When the root JSON element is an array, unmarshal directly into a slice. The  
decoder creates the necessary slice capacity automatically. This pattern is  
common in REST APIs that return lists of resources.  

## Error handling patterns

Robust error handling is essential for reliable JSON processing.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Config struct {
    Port    int    `json:"port"`
    Host    string `json:"host"`
    Timeout int    `json:"timeout"`
}

func parseConfig(data []byte) (*Config, error) {
    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return nil, fmt.Errorf("failed to parse config: %w", err)
    }
    
    // Validation after parsing
    if config.Port <= 0 || config.Port > 65535 {
        return nil, fmt.Errorf("invalid port: %d", config.Port)
    }
    if config.Host == "" {
        return nil, fmt.Errorf("host is required")
    }
    
    return &config, nil
}

func main() {
    // Valid JSON
    validJSON := `{"port":8080,"host":"localhost","timeout":30}`
    if config, err := parseConfig([]byte(validJSON)); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Valid config: %+v\n", config)
    }
    
    // Invalid JSON syntax
    invalidJSON := `{"port":8080,"host":"localhost"`
    if _, err := parseConfig([]byte(invalidJSON)); err != nil {
        fmt.Printf("\nError with invalid JSON: %v\n", err)
    }
    
    // Invalid values
    invalidValues := `{"port":99999,"host":"localhost","timeout":30}`
    if _, err := parseConfig([]byte(invalidValues)); err != nil {
        fmt.Printf("\nError with invalid values: %v\n", err)
    }
}
```

Always check errors from JSON operations. Syntax errors occur when JSON is  
malformed, while type errors occur when JSON types don't match Go types. Adding  
validation after unmarshaling ensures data integrity beyond just JSON syntax.  

## JSON file operations

Reading and writing JSON files is common for configuration and data storage.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type Database struct {
    Host     string   `json:"host"`
    Port     int      `json:"port"`
    Name     string   `json:"name"`
    Replicas []string `json:"replicas"`
}

func saveToFile(filename string, data interface{}) error {
    file, err := os.Create(filename)
    if err != nil {
        return fmt.Errorf("create file: %w", err)
    }
    defer file.Close()
    
    encoder := json.NewEncoder(file)
    encoder.SetIndent("", "  ")
    
    if err := encoder.Encode(data); err != nil {
        return fmt.Errorf("encode JSON: %w", err)
    }
    
    return nil
}

func loadFromFile(filename string, target interface{}) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("open file: %w", err)
    }
    defer file.Close()
    
    decoder := json.NewDecoder(file)
    if err := decoder.Decode(target); err != nil {
        return fmt.Errorf("decode JSON: %w", err)
    }
    
    return nil
}

func main() {
    db := Database{
        Host:     "db.example.com",
        Port:     5432,
        Name:     "production",
        Replicas: []string{"replica1.example.com", "replica2.example.com"},
    }
    
    filename := "/tmp/database.json"
    
    // Save to file
    if err := saveToFile(filename, db); err != nil {
        fmt.Printf("Error saving: %v\n", err)
        return
    }
    fmt.Printf("Saved to %s\n", filename)
    
    // Load from file
    var loaded Database
    if err := loadFromFile(filename, &loaded); err != nil {
        fmt.Printf("Error loading: %v\n", err)
        return
    }
    fmt.Printf("Loaded: %+v\n", loaded)
    
    // Cleanup
    os.Remove(filename)
}
```

Using `json.Encoder` and `json.Decoder` with files is more efficient than loading  
entire files into memory. Always use defer to ensure files are closed, and handle  
errors appropriately for production code.  

## JSON with validation

Validation ensures data meets business requirements beyond type checking.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "regexp"
)

type Registration struct {
    Username string `json:"username"`
    Email    string `json:"email"`
    Age      int    `json:"age"`
    Password string `json:"password"`
}

func (r *Registration) Validate() error {
    if len(r.Username) < 3 {
        return fmt.Errorf("username must be at least 3 characters")
    }
    
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    if !emailRegex.MatchString(r.Email) {
        return fmt.Errorf("invalid email format")
    }
    
    if r.Age < 18 || r.Age > 120 {
        return fmt.Errorf("age must be between 18 and 120")
    }
    
    if len(r.Password) < 8 {
        return fmt.Errorf("password must be at least 8 characters")
    }
    
    return nil
}

func main() {
    validJSON := `{
        "username": "john_doe",
        "email": "john@example.com",
        "age": 25,
        "password": "securepass123"
    }`
    
    var reg Registration
    if err := json.Unmarshal([]byte(validJSON), &reg); err != nil {
        fmt.Printf("JSON error: %v\n", err)
        return
    }
    
    if err := reg.Validate(); err != nil {
        fmt.Printf("Validation error: %v\n", err)
    } else {
        fmt.Println("Valid registration")
    }
    
    // Test invalid data
    invalidJSON := `{
        "username": "ab",
        "email": "not-an-email",
        "age": 15,
        "password": "short"
    }`
    
    var invalidReg Registration
    json.Unmarshal([]byte(invalidJSON), &invalidReg)
    if err := invalidReg.Validate(); err != nil {
        fmt.Printf("\nExpected validation error: %v\n", err)
    }
}
```

Implementing a `Validate` method on structs provides centralized validation logic.  
Call validation after unmarshaling to ensure data meets all requirements. This  
separates JSON parsing from business logic validation.  

## Complex nested structures

Deeply nested JSON structures can be handled with proper type definitions.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Company struct {
    Name         string       `json:"name"`
    Headquarters Location     `json:"headquarters"`
    Departments  []Department `json:"departments"`
}

type Location struct {
    Address Address `json:"address"`
    Phone   string  `json:"phone"`
}

type Address struct {
    Street  string `json:"street"`
    City    string `json:"city"`
    Country string `json:"country"`
}

type Department struct {
    Name      string     `json:"name"`
    Manager   string     `json:"manager"`
    Employees []Employee `json:"employees"`
}

type Employee struct {
    Name     string   `json:"name"`
    Position string   `json:"position"`
    Skills   []string `json:"skills"`
}

func main() {
    company := Company{
        Name: "Tech Corp",
        Headquarters: Location{
            Address: Address{
                Street:  "100 Main St",
                City:    "San Francisco",
                Country: "USA",
            },
            Phone: "555-0100",
        },
        Departments: []Department{
            {
                Name:    "Engineering",
                Manager: "Alice Johnson",
                Employees: []Employee{
                    {
                        Name:     "Bob Smith",
                        Position: "Senior Developer",
                        Skills:   []string{"Go", "Python", "Docker"},
                    },
                    {
                        Name:     "Carol White",
                        Position: "DevOps Engineer",
                        Skills:   []string{"Kubernetes", "AWS", "Terraform"},
                    },
                },
            },
        },
    }
    
    jsonData, _ := json.MarshalIndent(company, "", "  ")
    fmt.Println("Complex nested structure:")
    fmt.Println(string(jsonData))
    
    var decoded Company
    json.Unmarshal(jsonData, &decoded)
    fmt.Printf("\nDecoded company: %s\n", decoded.Name)
    fmt.Printf("Departments: %d\n", len(decoded.Departments))
    fmt.Printf("First department has %d employees\n",
        len(decoded.Departments[0].Employees))
}
```

Complex nested structures are handled naturally by defining appropriate struct  
types. The JSON encoder and decoder recursively process nested structures,  
maintaining type safety throughout the hierarchy.  

## Custom marshaling for special cases

Custom marshaling handles non-standard JSON requirements.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strings"
)

type Coordinates struct {
    Lat float64
    Lng float64
}

func (c Coordinates) MarshalJSON() ([]byte, error) {
    // Serialize as array instead of object
    return json.Marshal([]float64{c.Lat, c.Lng})
}

func (c *Coordinates) UnmarshalJSON(data []byte) error {
    var coords []float64
    if err := json.Unmarshal(data, &coords); err != nil {
        return err
    }
    
    if len(coords) != 2 {
        return fmt.Errorf("coordinates must have exactly 2 elements")
    }
    
    c.Lat = coords[0]
    c.Lng = coords[1]
    return nil
}

type Place struct {
    Name     string      `json:"name"`
    Location Coordinates `json:"location"`
}

type Tags []string

func (t Tags) MarshalJSON() ([]byte, error) {
    // Serialize as comma-separated string
    return json.Marshal(strings.Join(t, ","))
}

func (t *Tags) UnmarshalJSON(data []byte) error {
    var str string
    if err := json.Unmarshal(data, &str); err != nil {
        return err
    }
    
    *t = strings.Split(str, ",")
    return nil
}

type Article struct {
    Title string `json:"title"`
    Tags  Tags   `json:"tags"`
}

func main() {
    place := Place{
        Name:     "Golden Gate Bridge",
        Location: Coordinates{Lat: 37.8199, Lng: -122.4783},
    }
    
    jsonData, _ := json.MarshalIndent(place, "", "  ")
    fmt.Println("Coordinates as array:")
    fmt.Println(string(jsonData))
    
    article := Article{
        Title: "Go Programming",
        Tags:  Tags{"go", "programming", "tutorial"},
    }
    
    jsonData, _ = json.MarshalIndent(article, "", "  ")
    fmt.Println("\nTags as string:")
    fmt.Println(string(jsonData))
    
    // Unmarshal back
    var decodedPlace Place
    jsonStr := `{"name":"Eiffel Tower","location":[48.8584,2.2945]}`
    json.Unmarshal([]byte(jsonStr), &decodedPlace)
    fmt.Printf("\nDecoded: %s at [%.4f, %.4f]\n",
        decodedPlace.Name, decodedPlace.Location.Lat, decodedPlace.Location.Lng)
}
```

Custom marshaling methods allow complete control over JSON representation. This  
is useful for handling legacy formats, optimizing payload size, or conforming  
to external API requirements that differ from Go conventions.  

## Working with JSON APIs

Practical patterns for consuming JSON APIs with error handling.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type APIResponse struct {
    Success bool            `json:"success"`
    Data    json.RawMessage `json:"data,omitempty"`
    Error   string          `json:"error,omitempty"`
}

type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Username string `json:"username"`
}

func fetchUsers() ([]User, error) {
    // Simulated API response
    mockResponse := `{
        "success": true,
        "data": [
            {"id":1,"name":"Alice","username":"alice123"},
            {"id":2,"name":"Bob","username":"bob456"}
        ]
    }`
    
    var response APIResponse
    if err := json.Unmarshal([]byte(mockResponse), &response); err != nil {
        return nil, fmt.Errorf("parse response: %w", err)
    }
    
    if !response.Success {
        return nil, fmt.Errorf("API error: %s", response.Error)
    }
    
    var users []User
    if err := json.Unmarshal(response.Data, &users); err != nil {
        return nil, fmt.Errorf("parse data: %w", err)
    }
    
    return users, nil
}

func postData(endpoint string, data interface{}) error {
    jsonData, err := json.Marshal(data)
    if err != nil {
        return fmt.Errorf("marshal data: %w", err)
    }
    
    // In real code, use http.Post
    fmt.Printf("Would POST to %s:\n%s\n", endpoint, string(jsonData))
    return nil
}

func main() {
    // Fetch data
    users, err := fetchUsers()
    if err != nil {
        fmt.Printf("Error fetching users: %v\n", err)
        return
    }
    
    fmt.Println("Fetched users:")
    for _, user := range users {
        fmt.Printf("  [%d] %s (@%s)\n", user.ID, user.Name, user.Username)
    }
    
    // Post data
    newUser := User{
        Name:     "Charlie",
        Username: "charlie789",
    }
    
    fmt.Println()
    postData("https://api.example.com/users", newUser)
}
```

When working with APIs, wrap responses in a standard envelope structure for  
consistent error handling. Use `json.RawMessage` for flexible data parsing and  
always validate the response before processing the data payload.  

## Handling version compatibility

Managing different JSON schemas across versions requires careful design.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UserV1 struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

type UserV2 struct {
    ID        int    `json:"id"`
    FirstName string `json:"first_name"`
    LastName  string `json:"last_name"`
    Email     string `json:"email,omitempty"`
}

type VersionedUser struct {
    Version int             `json:"version"`
    Data    json.RawMessage `json:"data"`
}

func parseUser(data []byte) (interface{}, error) {
    var versioned VersionedUser
    if err := json.Unmarshal(data, &versioned); err != nil {
        return nil, err
    }
    
    switch versioned.Version {
    case 1:
        var user UserV1
        if err := json.Unmarshal(versioned.Data, &user); err != nil {
            return nil, err
        }
        return user, nil
    case 2:
        var user UserV2
        if err := json.Unmarshal(versioned.Data, &user); err != nil {
            return nil, err
        }
        return user, nil
    default:
        return nil, fmt.Errorf("unsupported version: %d", versioned.Version)
    }
}

func main() {
    v1JSON := `{
        "version": 1,
        "data": {"id": 1, "name": "Alice Johnson"}
    }`
    
    v2JSON := `{
        "version": 2,
        "data": {
            "id": 2,
            "first_name": "Bob",
            "last_name": "Smith",
            "email": "bob@example.com"
        }
    }`
    
    user1, _ := parseUser([]byte(v1JSON))
    fmt.Printf("V1 User: %+v\n", user1)
    
    user2, _ := parseUser([]byte(v2JSON))
    fmt.Printf("V2 User: %+v\n", user2)
}
```

Version fields and polymorphic parsing allow handling multiple schema versions.  
Use `json.RawMessage` to defer parsing until the version is known, then unmarshal  
into the appropriate struct type based on version information.  

## Performance optimization

Optimizing JSON processing for large datasets and high-throughput scenarios.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "sync"
    "time"
)

type DataPoint struct {
    Timestamp int64   `json:"ts"`
    Value     float64 `json:"val"`
    Tag       string  `json:"tag"`
}

// Pool for reusing byte buffers
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 0, 1024)
    },
}

func marshalWithPool(data interface{}) ([]byte, error) {
    buf := bufferPool.Get().([]byte)[:0]
    defer bufferPool.Put(buf)
    
    return json.Marshal(data)
}

func benchmarkMarshal(count int) time.Duration {
    data := make([]DataPoint, count)
    for i := 0; i < count; i++ {
        data[i] = DataPoint{
            Timestamp: time.Now().Unix(),
            Value:     float64(i) * 1.5,
            Tag:       "sensor",
        }
    }
    
    start := time.Now()
    for i := 0; i < 100; i++ {
        json.Marshal(data)
    }
    return time.Since(start)
}

func main() {
    // Single marshal operation
    point := DataPoint{
        Timestamp: time.Now().Unix(),
        Value:     42.5,
        Tag:       "temperature",
    }
    
    jsonData, _ := json.Marshal(point)
    fmt.Printf("Single point: %s\n", string(jsonData))
    
    // Benchmark
    fmt.Println("\nPerformance test:")
    for _, size := range []int{10, 100, 1000} {
        duration := benchmarkMarshal(size)
        fmt.Printf("  %d items x 100 iterations: %v\n", size, duration)
    }
    
    fmt.Println("\nOptimization tips:")
    fmt.Println("  - Reuse buffers with sync.Pool")
    fmt.Println("  - Use streaming for large datasets")
    fmt.Println("  - Profile before optimizing")
    fmt.Println("  - Consider alternative formats for extreme cases")
}
```

For performance-critical applications, reuse buffers, use streaming APIs, and  
profile to identify bottlenecks. JSON is human-readable but not the most  
efficient format; consider Protocol Buffers or MessagePack for extreme cases.  

## JSON with interfaces and type switching

Handling polymorphic data with interfaces and type assertions.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type Shape interface {
    Area() float64
}

type Circle struct {
    Type   string  `json:"type"`
    Radius float64 `json:"radius"`
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

type Rectangle struct {
    Type   string  `json:"type"`
    Width  float64 `json:"width"`
    Height float64 `json:"height"`
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

type ShapeWrapper struct {
    Type string          `json:"type"`
    Data json.RawMessage `json:"data"`
}

func parseShape(data []byte) (Shape, error) {
    var wrapper map[string]interface{}
    if err := json.Unmarshal(data, &wrapper); err != nil {
        return nil, err
    }
    
    shapeType, ok := wrapper["type"].(string)
    if !ok {
        return nil, fmt.Errorf("missing or invalid type field")
    }
    
    switch shapeType {
    case "circle":
        var circle Circle
        if err := json.Unmarshal(data, &circle); err != nil {
            return nil, err
        }
        return circle, nil
    case "rectangle":
        var rect Rectangle
        if err := json.Unmarshal(data, &rect); err != nil {
            return nil, err
        }
        return rect, nil
    default:
        return nil, fmt.Errorf("unknown shape type: %s", shapeType)
    }
}

func main() {
    shapes := []string{
        `{"type":"circle","radius":5.0}`,
        `{"type":"rectangle","width":4.0,"height":6.0}`,
    }
    
    for i, shapeJSON := range shapes {
        shape, err := parseShape([]byte(shapeJSON))
        if err != nil {
            fmt.Printf("Error parsing shape %d: %v\n", i+1, err)
            continue
        }
        
        fmt.Printf("Shape %d area: %.2f\n", i+1, shape.Area())
    }
}
```

Polymorphic JSON requires examining a type field before unmarshaling into the  
correct struct type. This pattern is common when working with heterogeneous  
collections where different items have different structures but share a  
common interface.  

## Working with null values

Properly handling JSON null values requires careful type selection.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type UserProfile struct {
    Username    string   `json:"username"`
    Email       *string  `json:"email"`       // Can be null
    PhoneNumber *string  `json:"phone"`       // Can be null
    Age         *int     `json:"age"`         // Can be null
    Preferences *Prefs   `json:"preferences"` // Can be null
}

type Prefs struct {
    Theme         string `json:"theme"`
    Notifications bool   `json:"notifications"`
}

func main() {
    // JSON with null values
    jsonWithNulls := `{
        "username": "testuser",
        "email": "user@example.com",
        "phone": null,
        "age": null,
        "preferences": null
    }`
    
    var profile UserProfile
    json.Unmarshal([]byte(jsonWithNulls), &profile)
    
    fmt.Println("Profile with nulls:")
    fmt.Printf("  Username: %s\n", profile.Username)
    
    if profile.Email != nil {
        fmt.Printf("  Email: %s\n", *profile.Email)
    } else {
        fmt.Println("  Email: not provided")
    }
    
    if profile.PhoneNumber != nil {
        fmt.Printf("  Phone: %s\n", *profile.PhoneNumber)
    } else {
        fmt.Println("  Phone: not provided")
    }
    
    if profile.Age != nil {
        fmt.Printf("  Age: %d\n", *profile.Age)
    } else {
        fmt.Println("  Age: not provided")
    }
    
    // Marshal back - null fields are serialized as null
    jsonData, _ := json.MarshalIndent(profile, "", "  ")
    fmt.Println("\nRe-marshaled:")
    fmt.Println(string(jsonData))
}
```

Pointer fields properly represent JSON null values. A nil pointer marshals to  
null, while a pointer to a zero value marshals to that zero value. This  
distinction is crucial for APIs where null and zero are semantically different.  

## JSON tags reference

Comprehensive guide to available JSON struct tags.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type TagsDemo struct {
    // Basic renaming
    PublicName string `json:"public_name"`
    
    // Omit empty values
    Optional string `json:"optional,omitempty"`
    
    // Completely ignore field
    Secret string `json:"-"`
    
    // Use field name with dash as JSON key
    DashField string `json:"-,"`
    
    // String encoded number
    StringNumber int `json:"string_number,string"`
    
    // Multiple options
    MultiOption *int `json:"multi_opt,omitempty"`
}

func main() {
    demo := TagsDemo{
        PublicName:   "visible",
        Optional:     "",           // Will be omitted
        Secret:       "hidden",     // Will be ignored
        DashField:    "dash value",
        StringNumber: 42,
        MultiOption:  nil, // Will be omitted
    }
    
    jsonData, _ := json.MarshalIndent(demo, "", "  ")
    fmt.Println("JSON with various tags:")
    fmt.Println(string(jsonData))
    
    fmt.Println("\nTag options:")
    fmt.Println("  json:\"name\"         - Use custom field name")
    fmt.Println("  json:\",omitempty\"   - Omit if zero value")
    fmt.Println("  json:\"-\"            - Always ignore field")
    fmt.Println("  json:\"-,\"           - Use literal \"-\" as name")
    fmt.Println("  json:\"name,string\"  - Encode as string")
}
```

JSON struct tags control field behavior during marshaling and unmarshaling. The  
`omitempty` option is most commonly used for optional fields, while `-` completely  
excludes fields from JSON processing. The `string` option is useful when working  
with APIs that require numbers as strings.  

## Best practices and common pitfalls

Summary of important patterns and mistakes to avoid in JSON handling.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// ✅ GOOD: Exported fields, proper tags
type GoodExample struct {
    ID        int       `json:"id"`
    Name      string    `json:"name"`
    CreatedAt time.Time `json:"created_at"`
}

// ❌ BAD: Unexported fields won't be serialized
type BadExample struct {
    id   int    // Won't be marshaled
    name string // Won't be marshaled
}

// ✅ GOOD: Use pointers for optional fields
type OptionalGood struct {
    Required string  `json:"required"`
    Optional *string `json:"optional,omitempty"`
}

// ❌ BAD: Can't distinguish between zero value and missing
type OptionalBad struct {
    Required string `json:"required"`
    Optional string `json:"optional,omitempty"` // Empty string is omitted
}

func main() {
    fmt.Println("Best Practices:")
    fmt.Println("✅ Use exported (capitalized) field names")
    fmt.Println("✅ Add json tags for all fields")
    fmt.Println("✅ Use pointers for truly optional fields")
    fmt.Println("✅ Validate data after unmarshaling")
    fmt.Println("✅ Handle errors from all JSON operations")
    fmt.Println("✅ Use streaming for large datasets")
    fmt.Println("✅ Consider using json.RawMessage for flexibility")
    
    fmt.Println("\nCommon Pitfalls:")
    fmt.Println("❌ Forgetting to export struct fields")
    fmt.Println("❌ Not checking errors")
    fmt.Println("❌ Assuming number types (all decode to float64 in maps)")
    fmt.Println("❌ Not validating unmarshaled data")
    fmt.Println("❌ Using value receivers for UnmarshalJSON")
    fmt.Println("❌ Loading entire large files into memory")
    
    // Demonstrate good vs bad
    good := GoodExample{ID: 1, Name: "Test", CreatedAt: time.Now()}
    goodJSON, _ := json.Marshal(good)
    fmt.Printf("\nGood example JSON: %s\n", string(goodJSON))
    
    bad := BadExample{id: 1, name: "Test"}
    badJSON, _ := json.Marshal(bad)
    fmt.Printf("Bad example JSON: %s (empty!)\n", string(badJSON))
}
```

Always export fields intended for JSON serialization, use appropriate types for  
optional fields, validate unmarshaled data, and handle errors properly. Use  
streaming APIs for large datasets and profile before optimizing. Following these  
practices ensures reliable and maintainable JSON handling in Go applications.  
