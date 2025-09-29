# Structs

In Go (Golang), a struct is a composite data type that groups together zero or more  
named fields of possibly different types. Structs are one of the most important and  
frequently used features in Go, serving as the foundation for object-oriented design  
patterns. Unlike classes in other languages, Go structs are simple data containers  
that can have methods associated with them through receivers, making them both  
flexible and powerful for organizing code.

Structs are value types by default, meaning when you assign a struct to another  
variable or pass it to a function, a copy is made. However, you can work with  
pointers to structs for reference semantics. This design choice gives developers  
explicit control over memory allocation and data sharing patterns. Structs support  
field tags, which are compile-time metadata that can be used by reflection and  
various libraries for serialization, validation, and other purposes.

Go's approach to structs emphasizes composition over inheritance. Rather than  
traditional class hierarchies, Go encourages embedding one struct within another  
to achieve code reuse and polymorphism through interfaces. This leads to more  
flexible and maintainable code architectures. Struct fields can be exported  
(capitalized names) or unexported (lowercase names), providing encapsulation at  
the package level.

## Basic struct definition

The simplest way to define a struct is by declaring its fields and their types.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    var person Person
    person.Name = "Alice"
    person.Age = 30
    
    fmt.Println("Name:", person.Name)
    fmt.Println("Age:", person.Age)
}
```

This example demonstrates the basic syntax for defining a struct type and  
accessing its fields using dot notation. The zero value for a struct has  
all fields set to their respective zero values.  

## Struct literals

Structs can be initialized using struct literals with or without field names.  

```go
package main

import "fmt"

type Book struct {
    Title  string
    Author string
    Pages  int
}

func main() {
    // Positional initialization
    book1 := Book{"1984", "George Orwell", 328}
    
    // Named field initialization
    book2 := Book{
        Title:  "To Kill a Mockingbird",
        Author: "Harper Lee",
        Pages:  376,
    }
    
    fmt.Printf("Book1: %+v\n", book1)
    fmt.Printf("Book2: %+v\n", book2)
}
```

Named field initialization is generally preferred as it's more readable and  
allows fields to be specified in any order. The `%+v` format verb prints  
field names along with values.  

## Pointer to struct

Working with pointers to structs is common for efficiency and reference semantics.  

```go
package main

import "fmt"

type Product struct {
    Name  string
    Price float64
}

func main() {
    // Create pointer to struct
    product := &Product{
        Name:  "Laptop",
        Price: 999.99,
    }
    
    // Access fields through pointer
    fmt.Println("Name:", product.Name)
    fmt.Println("Price:", product.Price)
    
    // Modify through pointer
    product.Price = 899.99
    fmt.Println("New price:", product.Price)
}
```

Go automatically dereferences pointers when accessing struct fields, so you  
can use the same dot notation whether working with values or pointers.  

## Anonymous structs

Structs can be defined inline without creating a named type.  

```go
package main

import "fmt"

func main() {
    // Define and initialize anonymous struct
    config := struct {
        Host string
        Port int
        SSL  bool
    }{
        Host: "localhost",
        Port: 8080,
        SSL:  false,
    }
    
    fmt.Printf("Server: %s:%d (SSL: %v)\n", config.Host, config.Port, config.SSL)
    
    // Anonymous struct in slice
    servers := []struct {
        Name string
        URL  string
    }{
        {"Primary", "https://api.primary.com"},
        {"Backup", "https://api.backup.com"},
    }
    
    for _, server := range servers {
        fmt.Printf("%s: %s\n", server.Name, server.URL)
    }
}
```

Anonymous structs are useful for temporary data structures or when you need  
a struct type only in a specific context without polluting the namespace.  

## Nested structs

Structs can contain other structs as fields, creating complex data structures.  

```go
package main

import "fmt"

type Address struct {
    Street  string
    City    string
    Country string
}

type Employee struct {
    Name    string
    ID      int
    Address Address
}

func main() {
    employee := Employee{
        Name: "Bob Smith",
        ID:   12345,
        Address: Address{
            Street:  "123 Main St",
            City:    "Springfield",
            Country: "USA",
        },
    }
    
    fmt.Println("Name:", employee.Name)
    fmt.Println("Street:", employee.Address.Street)
    fmt.Println("City:", employee.Address.City)
}
```

Nested structs allow you to model real-world relationships and create  
hierarchical data structures. Fields are accessed using multiple dots.  

## Struct embedding

Go supports struct embedding, which promotes fields from embedded structs.  

```go
package main

import "fmt"

type Engine struct {
    Power      int
    FuelType   string
}

type Car struct {
    Brand string
    Model string
    Engine
}

func main() {
    car := Car{
        Brand: "Toyota",
        Model: "Camry",
        Engine: Engine{
            Power:    200,
            FuelType: "Gasoline",
        },
    }
    
    // Access embedded fields directly
    fmt.Println("Brand:", car.Brand)
    fmt.Println("Power:", car.Power)      // Promoted field
    fmt.Println("Fuel:", car.FuelType)    // Promoted field
    
    // Or access through embedded struct
    fmt.Println("Engine power:", car.Engine.Power)
}
```

Embedding allows composition-based design where embedded struct fields  
are promoted to the containing struct, enabling a form of inheritance.  

## Methods on structs

Structs can have methods defined using receiver syntax.  

```go
package main

import "fmt"

type Rectangle struct {
    Width  float64
    Height float64
}

// Value receiver method
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Pointer receiver method
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    
    fmt.Println("Area:", rect.Area())
    
    rect.Scale(2)
    fmt.Println("After scaling:")
    fmt.Printf("Width: %.1f, Height: %.1f\n", rect.Width, rect.Height)
    fmt.Println("New area:", rect.Area())
}
```

Methods with value receivers work on copies, while pointer receivers work  
on the original struct. Use pointer receivers when you need to modify  
the struct or when the struct is large.  

## Struct tags

Struct fields can have tags that provide metadata for reflection and libraries.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type User struct {
    ID       int    `json:"id"`
    Name     string `json:"name"`
    Email    string `json:"email,omitempty"`
    Password string `json:"-"`
}

func main() {
    user := User{
        ID:       1,
        Name:     "Alice Johnson",
        Email:    "alice@example.com",
        Password: "secret123",
    }
    
    // Marshal to JSON
    jsonData, err := json.Marshal(user)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    
    fmt.Println("JSON:", string(jsonData))
    
    // Unmarshal from JSON
    var newUser User
    err = json.Unmarshal(jsonData, &newUser)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    
    fmt.Printf("Unmarshaled: %+v\n", newUser)
}
```

Struct tags control JSON marshaling behavior: field names, omitting empty  
values, and excluding fields entirely. They're widely used in web APIs  
and configuration management.  

## Struct comparison

Structs are comparable if all their fields are comparable types.  

```go
package main

import "fmt"

type Point struct {
    X int
    Y int
}

type Person struct {
    Name string
    Age  int
}

func main() {
    p1 := Point{X: 1, Y: 2}
    p2 := Point{X: 1, Y: 2}
    p3 := Point{X: 2, Y: 1}
    
    fmt.Println("p1 == p2:", p1 == p2)  // true
    fmt.Println("p1 == p3:", p1 == p3)  // false
    
    person1 := Person{Name: "Alice", Age: 30}
    person2 := Person{Name: "Alice", Age: 30}
    
    fmt.Println("Persons equal:", person1 == person2)  // true
}
```

Struct comparison performs field-by-field comparison. Two structs are equal  
if all corresponding fields are equal. This is useful for testing and  
data validation scenarios.  

## Struct copying

Structs are value types and are copied when assigned or passed to functions.  

```go
package main

import "fmt"

type Counter struct {
    Value int
}

func modifyValue(c Counter) {
    c.Value = 100
    fmt.Println("Inside function:", c.Value)
}

func modifyPointer(c *Counter) {
    c.Value = 200
    fmt.Println("Inside pointer function:", c.Value)
}

func main() {
    counter := Counter{Value: 10}
    
    fmt.Println("Original:", counter.Value)
    
    // Pass by value - creates a copy
    modifyValue(counter)
    fmt.Println("After value function:", counter.Value)
    
    // Pass by pointer - modifies original
    modifyPointer(&counter)
    fmt.Println("After pointer function:", counter.Value)
}
```

Understanding when structs are copied versus referenced is crucial for  
performance and correctness. Use pointers when you need to modify the  
original or avoid expensive copies.  

## Empty struct

The empty struct consumes zero bytes and is useful as a signal or placeholder.  

```go
package main

import "fmt"
import "unsafe"

type Signal struct{}

func main() {
    var s Signal
    fmt.Println("Size of empty struct:", unsafe.Sizeof(s))
    
    // Use as set
    visited := make(map[string]struct{})
    visited["page1"] = struct{}{}
    visited["page2"] = struct{}{}
    
    if _, exists := visited["page1"]; exists {
        fmt.Println("page1 was visited")
    }
    
    // Use as channel signal
    done := make(chan struct{})
    
    go func() {
        fmt.Println("Work completed")
        done <- struct{}{}
    }()
    
    <-done
    fmt.Println("Received completion signal")
}
```

Empty structs are memory-efficient and commonly used for sets (as map values)  
and signaling in concurrent programming since they carry no data.  

## Struct with slices and maps

Structs can contain complex types like slices and maps.  

```go
package main

import "fmt"

type Student struct {
    Name    string
    Grades  []int
    Courses map[string]int
}

func main() {
    student := Student{
        Name:   "Charlie Brown",
        Grades: []int{85, 90, 78, 92},
        Courses: map[string]int{
            "Math":    90,
            "Science": 85,
            "History": 88,
        },
    }
    
    fmt.Println("Student:", student.Name)
    fmt.Println("Grades:", student.Grades)
    
    // Add a grade
    student.Grades = append(student.Grades, 95)
    
    // Add a course
    student.Courses["Art"] = 93
    
    // Calculate average
    total := 0
    for _, grade := range student.Grades {
        total += grade
    }
    average := float64(total) / float64(len(student.Grades))
    fmt.Printf("Average grade: %.1f\n", average)
    
    fmt.Println("Courses:")
    for course, grade := range student.Courses {
        fmt.Printf("  %s: %d\n", course, grade)
    }
}
```

Structs containing slices and maps create flexible data structures. Remember  
that these fields hold references, so copying the struct shares the  
underlying data.  

## Constructor functions

Go doesn't have constructors, but you can create factory functions.  

```go
package main

import (
    "fmt"
    "time"
)

type Account struct {
    ID      string
    Balance float64
    Created time.Time
}

// Constructor function
func NewAccount(id string, initialBalance float64) *Account {
    return &Account{
        ID:      id,
        Balance: initialBalance,
        Created: time.Now(),
    }
}

// Constructor with validation
func NewValidAccount(id string, initialBalance float64) (*Account, error) {
    if id == "" {
        return nil, fmt.Errorf("account ID cannot be empty")
    }
    if initialBalance < 0 {
        return nil, fmt.Errorf("initial balance cannot be negative")
    }
    
    return &Account{
        ID:      id,
        Balance: initialBalance,
        Created: time.Now(),
    }, nil
}

func main() {
    // Using simple constructor
    account1 := NewAccount("ACC001", 1000.0)
    fmt.Printf("Account: %+v\n", account1)
    
    // Using validating constructor
    account2, err := NewValidAccount("ACC002", 500.0)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Printf("Valid account: %+v\n", account2)
    
    // Try invalid account
    _, err = NewValidAccount("", -100)
    if err != nil {
        fmt.Println("Validation error:", err)
    }
}
```

Constructor functions are a common pattern for initializing structs with  
default values, validation, or complex setup logic. They typically  
return pointers to avoid copying.  

## Struct interfaces

Structs can implement interfaces implicitly by having the required methods.  

```go
package main

import "fmt"

type Shape interface {
    Area() float64
    Perimeter() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

type Square struct {
    Side float64
}

func (s Square) Area() float64 {
    return s.Side * s.Side
}

func (s Square) Perimeter() float64 {
    return 4 * s.Side
}

func printShapeInfo(shape Shape) {
    fmt.Printf("Area: %.2f\n", shape.Area())
    fmt.Printf("Perimeter: %.2f\n", shape.Perimeter())
}

func main() {
    circle := Circle{Radius: 5}
    square := Square{Side: 4}
    
    fmt.Println("Circle:")
    printShapeInfo(circle)
    
    fmt.Println("\nSquare:")
    printShapeInfo(square)
}
```

Interfaces in Go are implemented implicitly. Any struct that has the required  
methods automatically satisfies the interface, enabling polymorphism and  
flexible code design.  

## Type assertion with structs

You can use type assertions to extract concrete types from interface values.  

```go
package main

import "fmt"

type Animal interface {
    Sound() string
}

type Dog struct {
    Name string
    Breed string
}

func (d Dog) Sound() string {
    return "Woof!"
}

type Cat struct {
    Name string
    Color string
}

func (c Cat) Sound() string {
    return "Meow!"
}

func describeAnimal(animal Animal) {
    fmt.Println("Sound:", animal.Sound())
    
    // Type assertion
    if dog, ok := animal.(Dog); ok {
        fmt.Printf("This is a %s named %s\n", dog.Breed, dog.Name)
    } else if cat, ok := animal.(Cat); ok {
        fmt.Printf("This is a %s cat named %s\n", cat.Color, cat.Name)
    }
}

func main() {
    animals := []Animal{
        Dog{Name: "Buddy", Breed: "Golden Retriever"},
        Cat{Name: "Whiskers", Color: "Orange"},
    }
    
    for _, animal := range animals {
        describeAnimal(animal)
        fmt.Println()
    }
}
```

Type assertions let you access the underlying concrete type and its specific  
fields when working with interface values. Always use the two-value form  
to avoid panics.  

## Struct composition patterns

Complex behaviors can be achieved through struct composition.  

```go
package main

import "fmt"

type Logger struct {
    Prefix string
}

func (l Logger) Log(message string) {
    fmt.Printf("[%s] %s\n", l.Prefix, message)
}

type Database struct {
    Logger
    Host string
    Port int
}

func (db *Database) Connect() {
    db.Log(fmt.Sprintf("Connecting to %s:%d", db.Host, db.Port))
}

func (db *Database) Query(sql string) {
    db.Log(fmt.Sprintf("Executing query: %s", sql))
}

type WebServer struct {
    Logger
    Port int
    Database *Database
}

func (ws *WebServer) Start() {
    ws.Log(fmt.Sprintf("Starting web server on port %d", ws.Port))
    ws.Database.Connect()
}

func main() {
    db := &Database{
        Logger: Logger{Prefix: "DB"},
        Host:   "localhost",
        Port:   5432,
    }
    
    server := &WebServer{
        Logger:   Logger{Prefix: "WEB"},
        Port:     8080,
        Database: db,
    }
    
    server.Start()
    db.Query("SELECT * FROM users")
}
```

Composition allows you to build complex types by combining simpler ones.  
Embedded structs provide functionality that can be reused across different  
struct types.  

## Builder pattern with structs

The builder pattern provides a fluent interface for constructing complex structs.  

```go
package main

import "fmt"

type HTTPClient struct {
    timeout     int
    retries     int
    userAgent   string
    headers     map[string]string
}

type HTTPClientBuilder struct {
    client HTTPClient
}

func NewHTTPClientBuilder() *HTTPClientBuilder {
    return &HTTPClientBuilder{
        client: HTTPClient{
            timeout:   30,
            retries:   3,
            userAgent: "DefaultAgent/1.0",
            headers:   make(map[string]string),
        },
    }
}

func (b *HTTPClientBuilder) Timeout(seconds int) *HTTPClientBuilder {
    b.client.timeout = seconds
    return b
}

func (b *HTTPClientBuilder) Retries(count int) *HTTPClientBuilder {
    b.client.retries = count
    return b
}

func (b *HTTPClientBuilder) UserAgent(agent string) *HTTPClientBuilder {
    b.client.userAgent = agent
    return b
}

func (b *HTTPClientBuilder) Header(key, value string) *HTTPClientBuilder {
    b.client.headers[key] = value
    return b
}

func (b *HTTPClientBuilder) Build() HTTPClient {
    return b.client
}

func main() {
    client := NewHTTPClientBuilder().
        Timeout(60).
        Retries(5).
        UserAgent("MyApp/2.0").
        Header("Accept", "application/json").
        Header("Authorization", "Bearer token123").
        Build()
    
    fmt.Printf("HTTP Client Configuration:\n")
    fmt.Printf("  Timeout: %d seconds\n", client.timeout)
    fmt.Printf("  Retries: %d\n", client.retries)
    fmt.Printf("  User-Agent: %s\n", client.userAgent)
    fmt.Printf("  Headers:\n")
    for key, value := range client.headers {
        fmt.Printf("    %s: %s\n", key, value)
    }
}
```

The builder pattern is useful for structs with many optional fields or  
complex initialization logic. It provides a clean, readable way to  
construct objects step by step.  

## Struct validation

You can add validation methods to ensure struct data integrity.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type User struct {
    Username string
    Email    string
    Age      int
}

func (u User) Validate() []string {
    var errors []string
    
    // Username validation
    if len(u.Username) < 3 {
        errors = append(errors, "username must be at least 3 characters")
    }
    if strings.Contains(u.Username, " ") {
        errors = append(errors, "username cannot contain spaces")
    }
    
    // Email validation
    emailRegex := regexp.MustCompile(`^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`)
    if !emailRegex.MatchString(u.Email) {
        errors = append(errors, "email format is invalid")
    }
    
    // Age validation
    if u.Age < 0 || u.Age > 150 {
        errors = append(errors, "age must be between 0 and 150")
    }
    
    return errors
}

func (u User) IsValid() bool {
    return len(u.Validate()) == 0
}

func main() {
    users := []User{
        {Username: "alice123", Email: "alice@example.com", Age: 25},
        {Username: "b", Email: "invalid-email", Age: -5},
        {Username: "john doe", Email: "john@test.com", Age: 30},
    }
    
    for i, user := range users {
        fmt.Printf("User %d: %+v\n", i+1, user)
        if user.IsValid() {
            fmt.Println("  Status: Valid")
        } else {
            fmt.Println("  Status: Invalid")
            for _, err := range user.Validate() {
                fmt.Printf("    - %s\n", err)
            }
        }
        fmt.Println()
    }
}
```

Validation methods help ensure data consistency and provide clear feedback  
about what's wrong with invalid data. They're essential for user input  
and API data validation.  

## Struct serialization

Structs can be serialized to various formats beyond JSON.  

```go
package main

import (
    "encoding/gob"
    "encoding/json"
    "encoding/xml"
    "fmt"
    "bytes"
)

type Product struct {
    ID          int     `json:"id" xml:"id"`
    Name        string  `json:"name" xml:"name"`
    Price       float64 `json:"price" xml:"price"`
    Description string  `json:"description" xml:"description"`
}

func main() {
    product := Product{
        ID:          1,
        Name:        "Laptop",
        Price:       999.99,
        Description: "High-performance laptop",
    }
    
    // JSON serialization
    jsonData, _ := json.MarshalIndent(product, "", "  ")
    fmt.Println("JSON:")
    fmt.Println(string(jsonData))
    fmt.Println()
    
    // XML serialization
    xmlData, _ := xml.MarshalIndent(product, "", "  ")
    fmt.Println("XML:")
    fmt.Printf("<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n%s\n", string(xmlData))
    fmt.Println()
    
    // GOB serialization (binary format)
    var gobBuffer bytes.Buffer
    encoder := gob.NewEncoder(&gobBuffer)
    encoder.Encode(product)
    
    fmt.Printf("GOB (binary) size: %d bytes\n", gobBuffer.Len())
    
    // Deserialize from GOB
    var decodedProduct Product
    decoder := gob.NewDecoder(&gobBuffer)
    decoder.Decode(&decodedProduct)
    
    fmt.Printf("Decoded from GOB: %+v\n", decodedProduct)
}
```

Go supports multiple serialization formats through struct tags and standard  
library packages. Choose the format based on your needs: JSON for APIs,  
XML for legacy systems, GOB for Go-to-Go communication.  

## Struct with channels

Structs can contain channels for concurrent communication.  

```go
package main

import (
    "fmt"
    "time"
)

type Worker struct {
    ID      int
    TaskCh  chan string
    ResultCh chan string
    QuitCh  chan bool
}

func NewWorker(id int) *Worker {
    return &Worker{
        ID:      id,
        TaskCh:  make(chan string),
        ResultCh: make(chan string),
        QuitCh:  make(chan bool),
    }
}

func (w *Worker) Start() {
    go func() {
        for {
            select {
            case task := <-w.TaskCh:
                result := fmt.Sprintf("Worker %d processed: %s", w.ID, task)
                time.Sleep(100 * time.Millisecond) // Simulate work
                w.ResultCh <- result
            case <-w.QuitCh:
                fmt.Printf("Worker %d stopping\n", w.ID)
                return
            }
        }
    }()
}

func (w *Worker) Stop() {
    w.QuitCh <- true
}

func main() {
    worker := NewWorker(1)
    worker.Start()
    
    // Send tasks
    go func() {
        tasks := []string{"task1", "task2", "task3"}
        for _, task := range tasks {
            worker.TaskCh <- task
        }
    }()
    
    // Receive results
    for i := 0; i < 3; i++ {
        result := <-worker.ResultCh
        fmt.Println(result)
    }
    
    worker.Stop()
    time.Sleep(100 * time.Millisecond) // Wait for worker to stop
}
```

Structs with channels enable powerful concurrent patterns. They can represent  
workers, services, or any component that needs to communicate asynchronously  
with other parts of the system.  

## Struct method chaining

Methods can return the struct itself to enable method chaining.  

```go
package main

import "fmt"

type QueryBuilder struct {
    table  string
    fields []string
    where  []string
    limit  int
}

func NewQueryBuilder() *QueryBuilder {
    return &QueryBuilder{}
}

func (qb *QueryBuilder) From(table string) *QueryBuilder {
    qb.table = table
    return qb
}

func (qb *QueryBuilder) Select(fields ...string) *QueryBuilder {
    qb.fields = append(qb.fields, fields...)
    return qb
}

func (qb *QueryBuilder) Where(condition string) *QueryBuilder {
    qb.where = append(qb.where, condition)
    return qb
}

func (qb *QueryBuilder) Limit(limit int) *QueryBuilder {
    qb.limit = limit
    return qb
}

func (qb *QueryBuilder) Build() string {
    query := "SELECT "
    
    if len(qb.fields) == 0 {
        query += "*"
    } else {
        for i, field := range qb.fields {
            if i > 0 {
                query += ", "
            }
            query += field
        }
    }
    
    query += " FROM " + qb.table
    
    if len(qb.where) > 0 {
        query += " WHERE "
        for i, condition := range qb.where {
            if i > 0 {
                query += " AND "
            }
            query += condition
        }
    }
    
    if qb.limit > 0 {
        query += fmt.Sprintf(" LIMIT %d", qb.limit)
    }
    
    return query
}

func main() {
    query := NewQueryBuilder().
        From("users").
        Select("id", "name", "email").
        Where("age > 18").
        Where("active = true").
        Limit(10).
        Build()
    
    fmt.Println("Generated query:")
    fmt.Println(query)
}
```

Method chaining creates fluent interfaces that are easy to read and use.  
Each method returns a pointer to the struct, allowing multiple method  
calls to be chained together.  

## Struct with mutex for concurrency

Structs can include mutexes to ensure thread-safe operations.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Counter struct {
    mu    sync.Mutex
    value int
}

func NewCounter() *Counter {
    return &Counter{}
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Decrement() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value--
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func main() {
    counter := NewCounter()
    var wg sync.WaitGroup
    
    // Start 10 goroutines that increment
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 100; j++ {
                counter.Increment()
                time.Sleep(time.Microsecond)
            }
        }()
    }
    
    // Start 5 goroutines that decrement
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for j := 0; j < 100; j++ {
                counter.Decrement()
                time.Sleep(time.Microsecond)
            }
        }()
    }
    
    wg.Wait()
    fmt.Printf("Final counter value: %d\n", counter.Value())
}
```

Including mutexes in structs ensures thread-safe access to shared data.  
Always use defer with Unlock() to ensure the mutex is released even  
if the function panics.  

## Struct inheritance simulation

Go doesn't have inheritance, but embedding can simulate it.  

```go
package main

import "fmt"

// Base "class"
type Vehicle struct {
    Brand string
    Model string
    Year  int
}

func (v Vehicle) Info() string {
    return fmt.Sprintf("%d %s %s", v.Year, v.Brand, v.Model)
}

func (v Vehicle) Start() {
    fmt.Printf("%s is starting...\n", v.Info())
}

// "Derived" classes
type Car struct {
    Vehicle
    Doors int
}

func (c Car) Honk() {
    fmt.Printf("%s honks: Beep beep!\n", c.Info())
}

type Motorcycle struct {
    Vehicle
    HasSidecar bool
}

func (m Motorcycle) Rev() {
    fmt.Printf("%s revs: Vroom vroom!\n", m.Info())
}

// Override method
func (m Motorcycle) Start() {
    fmt.Printf("%s kicks to start...\n", m.Info())
}

func startVehicle(v interface{ Start() }) {
    v.Start()
}

func main() {
    car := Car{
        Vehicle: Vehicle{
            Brand: "Toyota",
            Model: "Camry",
            Year:  2023,
        },
        Doors: 4,
    }
    
    bike := Motorcycle{
        Vehicle: Vehicle{
            Brand: "Harley Davidson",
            Model: "Street 750",
            Year:  2022,
        },
        HasSidecar: false,
    }
    
    // Use embedded methods
    fmt.Println("Car info:", car.Info())
    fmt.Printf("Car has %d doors\n", car.Doors)
    car.Honk()
    
    fmt.Println("\nBike info:", bike.Info())
    bike.Rev()
    
    fmt.Println("\nStarting vehicles:")
    startVehicle(car)
    startVehicle(bike)  // Uses overridden method
}
```

Embedding provides a composition-based alternative to inheritance. Embedded  
structs can have their methods overridden by the containing struct, and  
interfaces can be used for polymorphism.  

## Struct factory pattern

Factory functions can create different types based on parameters.  

```go
package main

import (
    "fmt"
    "strings"
)

type Notifier interface {
    Send(message string) error
}

type EmailNotifier struct {
    SMTPServer string
    Port       int
}

func (e EmailNotifier) Send(message string) error {
    fmt.Printf("Email sent via %s:%d: %s\n", e.SMTPServer, e.Port, message)
    return nil
}

type SMSNotifier struct {
    APIKey string
    Sender string
}

func (s SMSNotifier) Send(message string) error {
    fmt.Printf("SMS sent from %s: %s\n", s.Sender, message)
    return nil
}

type SlackNotifier struct {
    WebhookURL string
    Channel    string
}

func (sl SlackNotifier) Send(message string) error {
    fmt.Printf("Slack message to %s: %s\n", sl.Channel, message)
    return nil
}

// Factory function
func NewNotifier(notifierType string, config map[string]interface{}) (Notifier, error) {
    switch strings.ToLower(notifierType) {
    case "email":
        return EmailNotifier{
            SMTPServer: config["smtp_server"].(string),
            Port:       config["port"].(int),
        }, nil
    case "sms":
        return SMSNotifier{
            APIKey: config["api_key"].(string),
            Sender: config["sender"].(string),
        }, nil
    case "slack":
        return SlackNotifier{
            WebhookURL: config["webhook_url"].(string),
            Channel:    config["channel"].(string),
        }, nil
    default:
        return nil, fmt.Errorf("unknown notifier type: %s", notifierType)
    }
}

func main() {
    // Create different notifiers using factory
    emailNotifier, _ := NewNotifier("email", map[string]interface{}{
        "smtp_server": "smtp.gmail.com",
        "port":        587,
    })
    
    smsNotifier, _ := NewNotifier("sms", map[string]interface{}{
        "api_key": "abc123",
        "sender":  "+1234567890",
    })
    
    slackNotifier, _ := NewNotifier("slack", map[string]interface{}{
        "webhook_url": "https://hooks.slack.com/...",
        "channel":     "#alerts",
    })
    
    notifiers := []Notifier{emailNotifier, smsNotifier, slackNotifier}
    
    message := "System alert: High CPU usage detected"
    for _, notifier := range notifiers {
        notifier.Send(message)
    }
}
```

The factory pattern centralizes object creation logic and provides a clean  
way to create different implementations of an interface based on runtime  
parameters.  

## Struct with custom string representation

You can implement the Stringer interface for custom string formatting.  

```go
package main

import (
    "fmt"
    "strings"
)

type Person struct {
    FirstName string
    LastName  string
    Age       int
    Email     string
}

// Implement fmt.Stringer interface
func (p Person) String() string {
    return fmt.Sprintf("%s %s (age %d) <%s>", 
        p.FirstName, p.LastName, p.Age, p.Email)
}

type Team struct {
    Name    string
    Members []Person
}

func (t Team) String() string {
    var builder strings.Builder
    builder.WriteString(fmt.Sprintf("Team: %s\n", t.Name))
    builder.WriteString("Members:\n")
    
    for i, member := range t.Members {
        builder.WriteString(fmt.Sprintf("  %d. %s\n", i+1, member))
    }
    
    return builder.String()
}

func main() {
    team := Team{
        Name: "Development Team",
        Members: []Person{
            {"Alice", "Johnson", 30, "alice@company.com"},
            {"Bob", "Smith", 25, "bob@company.com"},
            {"Carol", "Davis", 35, "carol@company.com"},
        },
    }
    
    fmt.Println("Individual person:")
    fmt.Println(team.Members[0])
    
    fmt.Println("\nFull team:")
    fmt.Println(team)
}
```

Implementing the Stringer interface allows you to control how your structs  
are displayed when printed. This is useful for debugging, logging, and  
creating user-friendly representations.  

## Struct reflection

Reflection can be used to inspect struct fields and values at runtime.  

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID       int    `json:"id" validate:"required"`
    Name     string `json:"name" validate:"required,min=2"`
    Email    string `json:"email" validate:"email"`
    Age      int    `json:"age" validate:"min=0,max=150"`
    IsActive bool   `json:"is_active"`
}

func inspectStruct(v interface{}) {
    rv := reflect.ValueOf(v)
    rt := reflect.TypeOf(v)
    
    // Handle pointer to struct
    if rv.Kind() == reflect.Ptr {
        rv = rv.Elem()
        rt = rt.Elem()
    }
    
    fmt.Printf("Struct: %s\n", rt.Name())
    fmt.Printf("Package: %s\n", rt.PkgPath())
    fmt.Printf("Number of fields: %d\n\n", rt.NumField())
    
    for i := 0; i < rt.NumField(); i++ {
        field := rt.Field(i)
        value := rv.Field(i)
        
        fmt.Printf("Field: %s\n", field.Name)
        fmt.Printf("  Type: %s\n", field.Type)
        fmt.Printf("  Value: %v\n", value.Interface())
        fmt.Printf("  Exported: %v\n", field.IsExported())
        
        // Print tags
        if jsonTag := field.Tag.Get("json"); jsonTag != "" {
            fmt.Printf("  JSON tag: %s\n", jsonTag)
        }
        if validateTag := field.Tag.Get("validate"); validateTag != "" {
            fmt.Printf("  Validate tag: %s\n", validateTag)
        }
        
        fmt.Println()
    }
}

func main() {
    user := User{
        ID:       1,
        Name:     "Alice Johnson",
        Email:    "alice@example.com",
        Age:      30,
        IsActive: true,
    }
    
    inspectStruct(user)
}
```

Reflection allows runtime inspection of struct types, fields, and tags.  
It's powerful but should be used carefully as it can impact performance  
and type safety.  

## Struct pooling for performance

Object pools can reduce garbage collection pressure for frequently used structs.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type Message struct {
    ID        int
    Content   string
    Timestamp time.Time
    Processed bool
}

// Reset method to clean up the struct for reuse
func (m *Message) Reset() {
    m.ID = 0
    m.Content = ""
    m.Timestamp = time.Time{}
    m.Processed = false
}

var messagePool = sync.Pool{
    New: func() interface{} {
        return &Message{}
    },
}

func GetMessage() *Message {
    return messagePool.Get().(*Message)
}

func PutMessage(m *Message) {
    m.Reset()
    messagePool.Put(m)
}

func processMessage(id int, content string) {
    // Get message from pool
    msg := GetMessage()
    
    // Initialize
    msg.ID = id
    msg.Content = content
    msg.Timestamp = time.Now()
    
    // Simulate processing
    time.Sleep(10 * time.Millisecond)
    msg.Processed = true
    
    fmt.Printf("Processed message %d: %s\n", msg.ID, msg.Content)
    
    // Return to pool
    PutMessage(msg)
}

func main() {
    messages := []string{
        "Hello there",
        "How are you?",
        "Processing data...",
        "Task completed",
        "Goodbye",
    }
    
    var wg sync.WaitGroup
    
    for i, content := range messages {
        wg.Add(1)
        go func(id int, content string) {
            defer wg.Done()
            processMessage(id+1, content)
        }(i, content)
    }
    
    wg.Wait()
    fmt.Println("All messages processed")
}
```

Object pooling reduces memory allocations and garbage collection overhead  
for frequently created and destroyed structs. It's especially useful in  
high-throughput applications.  

## Struct with context

Structs can work with context for cancellation and timeouts.  

```go
package main

import (
    "context"
    "fmt"
    "time"
)

type TaskProcessor struct {
    Name    string
    Workers int
}

type Task struct {
    ID       int
    Data     string
    Duration time.Duration
}

func (tp *TaskProcessor) ProcessTask(ctx context.Context, task Task) error {
    fmt.Printf("[%s] Starting task %d: %s\n", tp.Name, task.ID, task.Data)
    
    // Create a timer for the task duration
    timer := time.NewTimer(task.Duration)
    defer timer.Stop()
    
    select {
    case <-timer.C:
        fmt.Printf("[%s] Completed task %d\n", tp.Name, task.ID)
        return nil
    case <-ctx.Done():
        fmt.Printf("[%s] Task %d cancelled: %v\n", tp.Name, task.ID, ctx.Err())
        return ctx.Err()
    }
}

func (tp *TaskProcessor) ProcessTasks(ctx context.Context, tasks []Task) {
    for _, task := range tasks {
        if err := tp.ProcessTask(ctx, task); err != nil {
            fmt.Printf("[%s] Stopping due to error: %v\n", tp.Name, err)
            break
        }
    }
}

func main() {
    processor := &TaskProcessor{
        Name:    "MainProcessor",
        Workers: 4,
    }
    
    tasks := []Task{
        {ID: 1, Data: "Process data A", Duration: 100 * time.Millisecond},
        {ID: 2, Data: "Process data B", Duration: 200 * time.Millisecond},
        {ID: 3, Data: "Process data C", Duration: 150 * time.Millisecond},
        {ID: 4, Data: "Process data D", Duration: 300 * time.Millisecond},
    }
    
    // Create context with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 400*time.Millisecond)
    defer cancel()
    
    processor.ProcessTasks(ctx, tasks)
}
```

Context integration allows structs to handle cancellation, timeouts, and  
deadline-based operations gracefully. This is essential for building  
robust concurrent applications.  

## Functional options pattern

The functional options pattern provides a flexible way to configure structs.  

```go
package main

import (
    "fmt"
    "time"
)

type Server struct {
    host         string
    port         int
    timeout      time.Duration
    maxConns     int
    readTimeout  time.Duration
    writeTimeout time.Duration
    enableTLS    bool
}

// Option function type
type ServerOption func(*Server)

// Option functions
func WithHost(host string) ServerOption {
    return func(s *Server) {
        s.host = host
    }
}

func WithPort(port int) ServerOption {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) ServerOption {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func WithMaxConnections(maxConns int) ServerOption {
    return func(s *Server) {
        s.maxConns = maxConns
    }
}

func WithReadTimeout(timeout time.Duration) ServerOption {
    return func(s *Server) {
        s.readTimeout = timeout
    }
}

func WithWriteTimeout(timeout time.Duration) ServerOption {
    return func(s *Server) {
        s.writeTimeout = timeout
    }
}

func WithTLS(enable bool) ServerOption {
    return func(s *Server) {
        s.enableTLS = enable
    }
}

// Constructor with functional options
func NewServer(options ...ServerOption) *Server {
    // Default configuration
    server := &Server{
        host:         "localhost",
        port:         8080,
        timeout:      30 * time.Second,
        maxConns:     100,
        readTimeout:  5 * time.Second,
        writeTimeout: 5 * time.Second,
        enableTLS:    false,
    }
    
    // Apply options
    for _, option := range options {
        option(server)
    }
    
    return server
}

func (s *Server) Start() {
    protocol := "HTTP"
    if s.enableTLS {
        protocol = "HTTPS"
    }
    
    fmt.Printf("Starting %s server on %s:%d\n", protocol, s.host, s.port)
    fmt.Printf("Configuration:\n")
    fmt.Printf("  Timeout: %v\n", s.timeout)
    fmt.Printf("  Max Connections: %d\n", s.maxConns)
    fmt.Printf("  Read Timeout: %v\n", s.readTimeout)
    fmt.Printf("  Write Timeout: %v\n", s.writeTimeout)
    fmt.Printf("  TLS Enabled: %v\n", s.enableTLS)
}

func main() {
    // Create server with default options
    server1 := NewServer()
    fmt.Println("Server 1 (defaults):")
    server1.Start()
    
    fmt.Println("\n" + "----------------------------------------" + "\n")
    
    // Create server with custom options
    server2 := NewServer(
        WithHost("0.0.0.0"),
        WithPort(443),
        WithTimeout(60*time.Second),
        WithMaxConnections(1000),
        WithReadTimeout(10*time.Second),
        WithWriteTimeout(10*time.Second),
        WithTLS(true),
    )
    
    fmt.Println("Server 2 (custom):")
    server2.Start()
}
```

The functional options pattern provides a clean, extensible way to configure  
structs with many optional parameters. It's more flexible than constructors  
with many parameters and avoids the need for multiple constructor variants.  

## Struct testing patterns

Structs benefit from structured testing approaches and helper functions.  

```go
package main

import (
    "fmt"
    "reflect"
    "testing"
    "time"
)

type Order struct {
    ID          string
    CustomerID  string
    Items       []OrderItem
    Total       float64
    CreatedAt   time.Time
    Status      string
}

type OrderItem struct {
    ProductID string
    Quantity  int
    Price     float64
}

func (o *Order) AddItem(item OrderItem) {
    o.Items = append(o.Items, item)
    o.Total += float64(item.Quantity) * item.Price
}

func (o *Order) CalculateTotal() float64 {
    var total float64
    for _, item := range o.Items {
        total += float64(item.Quantity) * item.Price
    }
    return total
}

// Test helpers
func NewTestOrder() *Order {
    return &Order{
        ID:         "ORD-001",
        CustomerID: "CUST-001",
        Items:      []OrderItem{},
        Total:      0,
        CreatedAt:  time.Now(),
        Status:     "pending",
    }
}

func AssertOrderEqual(t *testing.T, expected, actual *Order) {
    if expected.ID != actual.ID {
        t.Errorf("Expected ID %s, got %s", expected.ID, actual.ID)
    }
    if expected.CustomerID != actual.CustomerID {
        t.Errorf("Expected CustomerID %s, got %s", expected.CustomerID, actual.CustomerID)
    }
    if expected.Total != actual.Total {
        t.Errorf("Expected Total %.2f, got %.2f", expected.Total, actual.Total)
    }
    if !reflect.DeepEqual(expected.Items, actual.Items) {
        t.Errorf("Items don't match: expected %+v, got %+v", expected.Items, actual.Items)
    }
}

// Example test function (would normally be in _test.go file)
func TestOrderAddItem(t *testing.T) {
    order := NewTestOrder()
    
    item := OrderItem{
        ProductID: "PROD-001",
        Quantity:  2,
        Price:     19.99,
    }
    
    order.AddItem(item)
    
    expectedTotal := 39.98
    if order.Total != expectedTotal {
        t.Errorf("Expected total %.2f, got %.2f", expectedTotal, order.Total)
    }
    
    if len(order.Items) != 1 {
        t.Errorf("Expected 1 item, got %d", len(order.Items))
    }
}

func TestCalculateTotal(t *testing.T) {
    order := NewTestOrder()
    order.Items = []OrderItem{
        {ProductID: "PROD-001", Quantity: 2, Price: 10.50},
        {ProductID: "PROD-002", Quantity: 1, Price: 25.00},
    }
    
    expectedTotal := 46.00
    actualTotal := order.CalculateTotal()
    
    if actualTotal != expectedTotal {
        t.Errorf("Expected total %.2f, got %.2f", expectedTotal, actualTotal)
    }
}

// Demonstration main function
func main() {
    // Create a test order
    order := NewTestOrder()
    
    // Add some items
    order.AddItem(OrderItem{ProductID: "LAPTOP", Quantity: 1, Price: 999.99})
    order.AddItem(OrderItem{ProductID: "MOUSE", Quantity: 2, Price: 29.99})
    
    fmt.Printf("Order: %s\n", order.ID)
    fmt.Printf("Customer: %s\n", order.CustomerID)
    fmt.Printf("Items: %d\n", len(order.Items))
    fmt.Printf("Total: $%.2f\n", order.Total)
    fmt.Printf("Status: %s\n", order.Status)
    
    // Verify calculated total matches stored total
    calculatedTotal := order.CalculateTotal()
    if calculatedTotal == order.Total {
        fmt.Println("✓ Total calculation is correct")
    } else {
        fmt.Printf("✗ Total mismatch: stored %.2f, calculated %.2f\n", 
            order.Total, calculatedTotal)
    }
    
    // Simulate running tests
    fmt.Println("\nRunning tests...")
    
    // Mock testing.T for demonstration
    mockT := &testing.T{}
    TestOrderAddItem(mockT)
    TestCalculateTotal(mockT)
    
    fmt.Println("Tests completed (check test output for results)")
}
```

Structured testing of structs involves creating test helpers, assertion  
functions, and factory methods. This approach makes tests more maintainable  
and provides better error reporting when tests fail.  

## Struct field aliasing

You can create type aliases for struct fields to improve readability.  

```go
package main

import "fmt"

// Type aliases for better readability
type UserID int64
type Email string
type PhoneNumber string

type Contact struct {
    ID    UserID
    Name  string
    Email Email
    Phone PhoneNumber
}

// Methods on aliased types
func (e Email) IsValid() bool {
    return len(e) > 0 && len(e) < 100
}

func (p PhoneNumber) Format() string {
    if len(p) == 10 {
        return fmt.Sprintf("(%s) %s-%s", p[:3], p[3:6], p[6:])
    }
    return string(p)
}

func main() {
    contact := Contact{
        ID:    UserID(12345),
        Name:  "John Smith",
        Email: Email("john@example.com"),
        Phone: PhoneNumber("5551234567"),
    }
    
    fmt.Printf("Contact ID: %d\n", contact.ID)
    fmt.Printf("Name: %s\n", contact.Name)
    fmt.Printf("Email: %s (valid: %v)\n", contact.Email, contact.Email.IsValid())
    fmt.Printf("Phone: %s\n", contact.Phone.Format())
}
```

Type aliases make struct definitions more expressive and allow you to add  
methods to specific field types, improving code readability and  
encapsulation.  

## Struct with custom JSON marshaling

You can implement custom JSON marshaling for complex data transformations.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "strconv"
    "time"
)

type Event struct {
    Name      string
    Timestamp time.Time
    UserID    int64
    Metadata  map[string]interface{}
}

// Custom JSON marshaling
func (e Event) MarshalJSON() ([]byte, error) {
    return json.Marshal(&struct {
        Name      string                 `json:"name"`
        Timestamp int64                  `json:"timestamp"`
        UserID    string                 `json:"user_id"`
        Metadata  map[string]interface{} `json:"metadata"`
    }{
        Name:      e.Name,
        Timestamp: e.Timestamp.Unix(),
        UserID:    strconv.FormatInt(e.UserID, 10),
        Metadata:  e.Metadata,
    })
}

// Custom JSON unmarshaling
func (e *Event) UnmarshalJSON(data []byte) error {
    var temp struct {
        Name      string                 `json:"name"`
        Timestamp int64                  `json:"timestamp"`
        UserID    string                 `json:"user_id"`
        Metadata  map[string]interface{} `json:"metadata"`
    }
    
    if err := json.Unmarshal(data, &temp); err != nil {
        return err
    }
    
    userID, err := strconv.ParseInt(temp.UserID, 10, 64)
    if err != nil {
        return err
    }
    
    e.Name = temp.Name
    e.Timestamp = time.Unix(temp.Timestamp, 0)
    e.UserID = userID
    e.Metadata = temp.Metadata
    
    return nil
}

func main() {
    event := Event{
        Name:      "user_login",
        Timestamp: time.Now(),
        UserID:    12345,
        Metadata: map[string]interface{}{
            "ip_address": "192.168.1.1",
            "device":     "mobile",
        },
    }
    
    // Marshal to JSON
    jsonData, _ := json.MarshalIndent(event, "", "  ")
    fmt.Println("Marshaled JSON:")
    fmt.Println(string(jsonData))
    
    // Unmarshal back
    var parsedEvent Event
    json.Unmarshal(jsonData, &parsedEvent)
    
    fmt.Printf("\nUnmarshaled event:\n")
    fmt.Printf("Name: %s\n", parsedEvent.Name)
    fmt.Printf("Timestamp: %v\n", parsedEvent.Timestamp)
    fmt.Printf("UserID: %d\n", parsedEvent.UserID)
    fmt.Printf("Metadata: %+v\n", parsedEvent.Metadata)
}
```

Custom JSON marshaling gives you complete control over how structs are  
serialized and deserialized, enabling data transformation and format  
conversion during the process.  

## Struct with computed fields

Structs can have methods that calculate values from other fields.  

```go
package main

import (
    "fmt"
    "math"
    "time"
)

type ShoppingCart struct {
    Items     []CartItem
    TaxRate   float64
    Discount  float64
    CreatedAt time.Time
}

type CartItem struct {
    Name     string
    Price    float64
    Quantity int
}

func (sc *ShoppingCart) Subtotal() float64 {
    var total float64
    for _, item := range sc.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

func (sc *ShoppingCart) Tax() float64 {
    return sc.Subtotal() * sc.TaxRate
}

func (sc *ShoppingCart) DiscountAmount() float64 {
    return sc.Subtotal() * sc.Discount
}

func (sc *ShoppingCart) Total() float64 {
    return sc.Subtotal() + sc.Tax() - sc.DiscountAmount()
}

func (sc *ShoppingCart) ItemCount() int {
    var count int
    for _, item := range sc.Items {
        count += item.Quantity
    }
    return count
}

func (sc *ShoppingCart) AverageItemPrice() float64 {
    if len(sc.Items) == 0 {
        return 0
    }
    return sc.Subtotal() / float64(sc.ItemCount())
}

func (sc *ShoppingCart) TimeAlive() time.Duration {
    return time.Since(sc.CreatedAt)
}

func main() {
    cart := &ShoppingCart{
        Items: []CartItem{
            {"Laptop", 999.99, 1},
            {"Mouse", 29.99, 2},
            {"Keyboard", 79.99, 1},
        },
        TaxRate:   0.08,
        Discount:  0.10,
        CreatedAt: time.Now().Add(-2 * time.Hour),
    }
    
    fmt.Printf("Shopping Cart Summary:\n")
    fmt.Printf("Items: %d\n", len(cart.Items))
    fmt.Printf("Total quantity: %d\n", cart.ItemCount())
    fmt.Printf("Subtotal: $%.2f\n", cart.Subtotal())
    fmt.Printf("Tax (%.0f%%): $%.2f\n", cart.TaxRate*100, cart.Tax())
    fmt.Printf("Discount (%.0f%%): -$%.2f\n", cart.Discount*100, cart.DiscountAmount())
    fmt.Printf("Total: $%.2f\n", cart.Total())
    fmt.Printf("Average item price: $%.2f\n", cart.AverageItemPrice())
    fmt.Printf("Cart age: %v\n", cart.TimeAlive().Round(time.Minute))
}
```

Computed fields provide calculated values based on struct data without  
storing redundant information. They ensure calculations are always  
up-to-date when the underlying data changes.  

## Struct versioning

Structs can support versioning for backward compatibility.  

```go
package main

import (
    "encoding/json"
    "fmt"
)

type ConfigV1 struct {
    Version  int    `json:"version"`
    Host     string `json:"host"`
    Port     int    `json:"port"`
    Database string `json:"database"`
}

type ConfigV2 struct {
    Version  int               `json:"version"`
    Server   ServerConfig      `json:"server"`
    Database DatabaseConfig    `json:"database"`
    Features map[string]bool   `json:"features"`
}

type ServerConfig struct {
    Host string `json:"host"`
    Port int    `json:"port"`
    SSL  bool   `json:"ssl"`
}

type DatabaseConfig struct {
    Name     string `json:"name"`
    Host     string `json:"host"`
    Port     int    `json:"port"`
    Username string `json:"username"`
}

type Config struct {
    Version  int               `json:"version"`
    Server   ServerConfig      `json:"server,omitempty"`
    Database DatabaseConfig    `json:"database,omitempty"`
    Features map[string]bool   `json:"features,omitempty"`
    // Legacy fields for V1 compatibility
    Host     string            `json:"host,omitempty"`
    Port     int               `json:"port,omitempty"`
    DBName   string            `json:"database,omitempty"`
}

func (c *Config) MigrateToV2() {
    if c.Version == 1 {
        // Migrate V1 to V2
        c.Server = ServerConfig{
            Host: c.Host,
            Port: c.Port,
            SSL:  false,
        }
        c.Database = DatabaseConfig{
            Name: c.DBName,
            Host: "localhost",
            Port: 5432,
        }
        c.Features = map[string]bool{
            "logging": true,
            "metrics": false,
        }
        c.Version = 2
        
        // Clear legacy fields
        c.Host = ""
        c.Port = 0
        c.DBName = ""
    }
}

func LoadConfig(jsonData []byte) (*Config, error) {
    var config Config
    if err := json.Unmarshal(jsonData, &config); err != nil {
        return nil, err
    }
    
    // Auto-migrate to latest version
    config.MigrateToV2()
    
    return &config, nil
}

func main() {
    // V1 configuration
    v1Config := `{
        "version": 1,
        "host": "localhost",
        "port": 8080,
        "database": "myapp"
    }`
    
    // V2 configuration
    v2Config := `{
        "version": 2,
        "server": {
            "host": "0.0.0.0",
            "port": 443,
            "ssl": true
        },
        "database": {
            "name": "production",
            "host": "db.example.com",
            "port": 5432,
            "username": "app_user"
        },
        "features": {
            "logging": true,
            "metrics": true
        }
    }`
    
    fmt.Println("Loading V1 config:")
    config1, _ := LoadConfig([]byte(v1Config))
    fmt.Printf("Migrated config: %+v\n", config1)
    
    fmt.Println("\nLoading V2 config:")
    config2, _ := LoadConfig([]byte(v2Config))
    fmt.Printf("V2 config: %+v\n", config2)
}
```

Struct versioning allows you to maintain backward compatibility while  
evolving your data structures. Migration methods can automatically  
upgrade older versions to newer formats.  

## Struct state machine

Structs can implement state machines with validation and transitions.  

```go
package main

import (
    "fmt"
    "time"
)

type OrderState string

const (
    StatePending   OrderState = "pending"
    StateConfirmed OrderState = "confirmed"
    StateShipped   OrderState = "shipped"
    StateDelivered OrderState = "delivered"
    StateCancelled OrderState = "cancelled"
)

type Order struct {
    ID       string
    State    OrderState
    Amount   float64
    Created  time.Time
    Modified time.Time
}

func NewOrder(id string, amount float64) *Order {
    return &Order{
        ID:       id,
        State:    StatePending,
        Amount:   amount,
        Created:  time.Now(),
        Modified: time.Now(),
    }
}

func (o *Order) CanTransitionTo(newState OrderState) bool {
    validTransitions := map[OrderState][]OrderState{
        StatePending:   {StateConfirmed, StateCancelled},
        StateConfirmed: {StateShipped, StateCancelled},
        StateShipped:   {StateDelivered},
        StateDelivered: {},
        StateCancelled: {},
    }
    
    allowed, exists := validTransitions[o.State]
    if !exists {
        return false
    }
    
    for _, state := range allowed {
        if state == newState {
            return true
        }
    }
    return false
}

func (o *Order) TransitionTo(newState OrderState) error {
    if !o.CanTransitionTo(newState) {
        return fmt.Errorf("cannot transition from %s to %s", o.State, newState)
    }
    
    oldState := o.State
    o.State = newState
    o.Modified = time.Now()
    
    fmt.Printf("Order %s transitioned from %s to %s\n", o.ID, oldState, newState)
    return nil
}

func (o *Order) Confirm() error {
    return o.TransitionTo(StateConfirmed)
}

func (o *Order) Ship() error {
    return o.TransitionTo(StateShipped)
}

func (o *Order) Deliver() error {
    return o.TransitionTo(StateDelivered)
}

func (o *Order) Cancel() error {
    return o.TransitionTo(StateCancelled)
}

func (o *Order) IsActive() bool {
    return o.State != StateCancelled && o.State != StateDelivered
}

func main() {
    order := NewOrder("ORD-001", 99.99)
    
    fmt.Printf("Created order: %s (state: %s)\n", order.ID, order.State)
    
    // Valid transitions
    order.Confirm()
    order.Ship()
    order.Deliver()
    
    fmt.Printf("Final state: %s, Active: %v\n", order.State, order.IsActive())
    
    // Try invalid transition
    err := order.Ship()
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

State machine structs enforce valid state transitions and provide clear  
interfaces for state changes. This prevents invalid states and makes  
business logic more robust.  

## Struct with event sourcing

Structs can track changes through event sourcing patterns.  

```go
package main

import (
    "fmt"
    "time"
)

type EventType string

const (
    EventAccountCreated EventType = "account_created"
    EventMoneyDeposited EventType = "money_deposited"
    EventMoneyWithdrawn EventType = "money_withdrawn"
    EventAccountClosed  EventType = "account_closed"
)

type Event struct {
    ID        int
    Type      EventType
    Data      map[string]interface{}
    Timestamp time.Time
}

type Account struct {
    ID       string
    Balance  float64
    Status   string
    Events   []Event
    nextEventID int
}

func NewAccount(id string) *Account {
    account := &Account{
        ID:          id,
        Balance:     0,
        Status:      "open",
        Events:      make([]Event, 0),
        nextEventID: 1,
    }
    
    account.addEvent(EventAccountCreated, map[string]interface{}{
        "account_id": id,
    })
    
    return account
}

func (a *Account) addEvent(eventType EventType, data map[string]interface{}) {
    event := Event{
        ID:        a.nextEventID,
        Type:      eventType,
        Data:      data,
        Timestamp: time.Now(),
    }
    
    a.Events = append(a.Events, event)
    a.nextEventID++
    a.applyEvent(event)
}

func (a *Account) applyEvent(event Event) {
    switch event.Type {
    case EventMoneyDeposited:
        if amount, ok := event.Data["amount"].(float64); ok {
            a.Balance += amount
        }
    case EventMoneyWithdrawn:
        if amount, ok := event.Data["amount"].(float64); ok {
            a.Balance -= amount
        }
    case EventAccountClosed:
        a.Status = "closed"
    }
}

func (a *Account) Deposit(amount float64) error {
    if a.Status != "open" {
        return fmt.Errorf("account is closed")
    }
    if amount <= 0 {
        return fmt.Errorf("amount must be positive")
    }
    
    a.addEvent(EventMoneyDeposited, map[string]interface{}{
        "amount": amount,
    })
    
    return nil
}

func (a *Account) Withdraw(amount float64) error {
    if a.Status != "open" {
        return fmt.Errorf("account is closed")
    }
    if amount <= 0 {
        return fmt.Errorf("amount must be positive")
    }
    if amount > a.Balance {
        return fmt.Errorf("insufficient funds")
    }
    
    a.addEvent(EventMoneyWithdrawn, map[string]interface{}{
        "amount": amount,
    })
    
    return nil
}

func (a *Account) Close() error {
    if a.Status != "open" {
        return fmt.Errorf("account is already closed")
    }
    
    a.addEvent(EventAccountClosed, map[string]interface{}{})
    return nil
}

func (a *Account) GetHistory() []Event {
    return a.Events
}

func main() {
    account := NewAccount("ACC-001")
    
    // Perform operations
    account.Deposit(1000.0)
    account.Withdraw(250.0)
    account.Deposit(500.0)
    account.Withdraw(100.0)
    
    fmt.Printf("Account: %s\n", account.ID)
    fmt.Printf("Balance: $%.2f\n", account.Balance)
    fmt.Printf("Status: %s\n", account.Status)
    
    fmt.Println("\nEvent History:")
    for _, event := range account.GetHistory() {
        fmt.Printf("  [%d] %s at %s: %+v\n", 
            event.ID, event.Type, event.Timestamp.Format("15:04:05"), event.Data)
    }
}
```

Event sourcing captures all changes as events, providing a complete audit  
trail and enabling features like replay, debugging, and temporal queries.  
The current state is derived from applying all events in sequence.  

## Struct with caching

Structs can implement caching mechanisms for expensive operations.  

```go
package main

import (
    "fmt"
    "math"
    "sync"
    "time"
)

type ExpensiveCalculator struct {
    cache     map[string]CacheEntry
    cacheMux  sync.RWMutex
    ttl       time.Duration
    hitCount  int64
    missCount int64
}

type CacheEntry struct {
    Value     float64
    Timestamp time.Time
}

func NewExpensiveCalculator(ttl time.Duration) *ExpensiveCalculator {
    return &ExpensiveCalculator{
        cache: make(map[string]CacheEntry),
        ttl:   ttl,
    }
}

func (ec *ExpensiveCalculator) expensiveOperation(x float64) float64 {
    // Simulate expensive calculation
    time.Sleep(100 * time.Millisecond)
    return math.Pow(x, 3) + math.Sin(x) * 1000
}

func (ec *ExpensiveCalculator) Calculate(x float64) float64 {
    key := fmt.Sprintf("calc_%.6f", x)
    
    // Check cache first
    ec.cacheMux.RLock()
    if entry, exists := ec.cache[key]; exists {
        if time.Since(entry.Timestamp) < ec.ttl {
            ec.cacheMux.RUnlock()
            ec.hitCount++
            fmt.Printf("Cache HIT for x=%.2f\n", x)
            return entry.Value
        }
    }
    ec.cacheMux.RUnlock()
    
    // Cache miss - perform calculation
    ec.missCount++
    fmt.Printf("Cache MISS for x=%.2f - calculating...\n", x)
    result := ec.expensiveOperation(x)
    
    // Store in cache
    ec.cacheMux.Lock()
    ec.cache[key] = CacheEntry{
        Value:     result,
        Timestamp: time.Now(),
    }
    ec.cacheMux.Unlock()
    
    return result
}

func (ec *ExpensiveCalculator) ClearExpired() {
    ec.cacheMux.Lock()
    defer ec.cacheMux.Unlock()
    
    now := time.Now()
    for key, entry := range ec.cache {
        if now.Sub(entry.Timestamp) >= ec.ttl {
            delete(ec.cache, key)
        }
    }
}

func (ec *ExpensiveCalculator) Stats() (int64, int64, float64) {
    total := ec.hitCount + ec.missCount
    hitRate := float64(ec.hitCount) / float64(total) * 100
    return ec.hitCount, ec.missCount, hitRate
}

func main() {
    calculator := NewExpensiveCalculator(2 * time.Second)
    
    // Perform calculations
    values := []float64{1.5, 2.5, 1.5, 3.5, 2.5, 1.5}
    
    for _, val := range values {
        result := calculator.Calculate(val)
        fmt.Printf("  Result: %.2f\n", result)
        time.Sleep(500 * time.Millisecond)
    }
    
    // Show cache statistics
    hits, misses, hitRate := calculator.Stats()
    fmt.Printf("\nCache Statistics:\n")
    fmt.Printf("  Hits: %d\n", hits)
    fmt.Printf("  Misses: %d\n", misses)
    fmt.Printf("  Hit Rate: %.1f%%\n", hitRate)
    
    // Test cache expiration
    fmt.Println("\nWaiting for cache to expire...")
    time.Sleep(3 * time.Second)
    calculator.ClearExpired()
    
    result := calculator.Calculate(1.5)
    fmt.Printf("After expiration - Result: %.2f\n", result)
}
```

Cached structs improve performance by storing expensive computation results.  
The cache includes TTL (time-to-live) functionality and provides statistics  
for monitoring cache effectiveness.  

## Struct configuration management

Structs can manage application configuration with validation and defaults.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "strconv"
    "time"
)

type AppConfig struct {
    Server   ServerConfig   `json:"server"`
    Database DatabaseConfig `json:"database"`
    Logging  LoggingConfig  `json:"logging"`
    Features FeatureFlags   `json:"features"`
}

type ServerConfig struct {
    Host         string        `json:"host"`
    Port         int           `json:"port"`
    ReadTimeout  time.Duration `json:"read_timeout"`
    WriteTimeout time.Duration `json:"write_timeout"`
    MaxConns     int           `json:"max_connections"`
}

type DatabaseConfig struct {
    Host         string        `json:"host"`
    Port         int           `json:"port"`
    Name         string        `json:"name"`
    User         string        `json:"user"`
    Password     string        `json:"password"`
    MaxIdleConns int           `json:"max_idle_connections"`
    ConnTimeout  time.Duration `json:"connection_timeout"`
}

type LoggingConfig struct {
    Level      string `json:"level"`
    Format     string `json:"format"`
    Output     string `json:"output"`
    Structured bool   `json:"structured"`
}

type FeatureFlags struct {
    EnableMetrics bool `json:"enable_metrics"`
    EnableTracing bool `json:"enable_tracing"`
    EnableCache   bool `json:"enable_cache"`
    DebugMode     bool `json:"debug_mode"`
}

func DefaultConfig() *AppConfig {
    return &AppConfig{
        Server: ServerConfig{
            Host:         "localhost",
            Port:         8080,
            ReadTimeout:  30 * time.Second,
            WriteTimeout: 30 * time.Second,
            MaxConns:     100,
        },
        Database: DatabaseConfig{
            Host:         "localhost",
            Port:         5432,
            Name:         "myapp",
            User:         "user",
            MaxIdleConns: 10,
            ConnTimeout:  10 * time.Second,
        },
        Logging: LoggingConfig{
            Level:      "info",
            Format:     "text",
            Output:     "stdout",
            Structured: false,
        },
        Features: FeatureFlags{
            EnableMetrics: true,
            EnableTracing: false,
            EnableCache:   true,
            DebugMode:     false,
        },
    }
}

func (c *AppConfig) LoadFromFile(filename string) error {
    data, err := os.ReadFile(filename)
    if err != nil {
        return err
    }
    return json.Unmarshal(data, c)
}

func (c *AppConfig) LoadFromEnv() {
    if host := os.Getenv("SERVER_HOST"); host != "" {
        c.Server.Host = host
    }
    if port := os.Getenv("SERVER_PORT"); port != "" {
        if p, err := strconv.Atoi(port); err == nil {
            c.Server.Port = p
        }
    }
    if dbHost := os.Getenv("DB_HOST"); dbHost != "" {
        c.Database.Host = dbHost
    }
    if dbPass := os.Getenv("DB_PASSWORD"); dbPass != "" {
        c.Database.Password = dbPass
    }
    if logLevel := os.Getenv("LOG_LEVEL"); logLevel != "" {
        c.Logging.Level = logLevel
    }
    if debug := os.Getenv("DEBUG"); debug == "true" {
        c.Features.DebugMode = true
    }
}

func (c *AppConfig) Validate() error {
    if c.Server.Port < 1 || c.Server.Port > 65535 {
        return fmt.Errorf("invalid server port: %d", c.Server.Port)
    }
    if c.Database.Name == "" {
        return fmt.Errorf("database name is required")
    }
    validLevels := []string{"debug", "info", "warn", "error"}
    valid := false
    for _, level := range validLevels {
        if c.Logging.Level == level {
            valid = true
            break
        }
    }
    if !valid {
        return fmt.Errorf("invalid log level: %s", c.Logging.Level)
    }
    return nil
}

func (c *AppConfig) Print() {
    fmt.Printf("Application Configuration:\n")
    fmt.Printf("Server:\n")
    fmt.Printf("  Host: %s\n", c.Server.Host)
    fmt.Printf("  Port: %d\n", c.Server.Port)
    fmt.Printf("  Read Timeout: %v\n", c.Server.ReadTimeout)
    fmt.Printf("  Write Timeout: %v\n", c.Server.WriteTimeout)
    fmt.Printf("  Max Connections: %d\n", c.Server.MaxConns)
    
    fmt.Printf("Database:\n")
    fmt.Printf("  Host: %s\n", c.Database.Host)
    fmt.Printf("  Port: %d\n", c.Database.Port)
    fmt.Printf("  Name: %s\n", c.Database.Name)
    fmt.Printf("  User: %s\n", c.Database.User)
    fmt.Printf("  Password: %s\n", maskPassword(c.Database.Password))
    
    fmt.Printf("Logging:\n")
    fmt.Printf("  Level: %s\n", c.Logging.Level)
    fmt.Printf("  Format: %s\n", c.Logging.Format)
    fmt.Printf("  Output: %s\n", c.Logging.Output)
    
    fmt.Printf("Features:\n")
    fmt.Printf("  Metrics: %v\n", c.Features.EnableMetrics)
    fmt.Printf("  Tracing: %v\n", c.Features.EnableTracing)
    fmt.Printf("  Cache: %v\n", c.Features.EnableCache)
    fmt.Printf("  Debug: %v\n", c.Features.DebugMode)
}

func maskPassword(password string) string {
    if password == "" {
        return "(not set)"
    }
    return "****"
}

func main() {
    // Start with defaults
    config := DefaultConfig()
    
    // Override with environment variables
    config.LoadFromEnv()
    
    // Validate configuration
    if err := config.Validate(); err != nil {
        fmt.Printf("Configuration error: %v\n", err)
        return
    }
    
    config.Print()
}
```

Configuration management structs provide structured way to handle application  
settings with defaults, validation, and multiple sources (files, environment  
variables). This approach ensures consistent configuration handling.  

## Struct lifecycle management

Structs can manage their own lifecycle with initialization and cleanup.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type ResourceManager struct {
    name        string
    initialized bool
    running     bool
    resources   map[string]interface{}
    workers     []*Worker
    stopCh      chan bool
    doneCh      chan bool
    mu          sync.RWMutex
}

type Worker struct {
    ID      int
    manager *ResourceManager
    stopCh  chan bool
}

func NewResourceManager(name string) *ResourceManager {
    return &ResourceManager{
        name:      name,
        resources: make(map[string]interface{}),
        workers:   make([]*Worker, 0),
        stopCh:    make(chan bool, 1),
        doneCh:    make(chan bool, 1),
    }
}

func (rm *ResourceManager) Initialize() error {
    rm.mu.Lock()
    defer rm.mu.Unlock()
    
    if rm.initialized {
        return fmt.Errorf("resource manager already initialized")
    }
    
    fmt.Printf("[%s] Initializing...\n", rm.name)
    
    // Initialize resources
    rm.resources["database"] = "db_connection"
    rm.resources["cache"] = "redis_connection"
    rm.resources["logger"] = "log_instance"
    
    // Create workers
    for i := 0; i < 3; i++ {
        worker := &Worker{
            ID:      i + 1,
            manager: rm,
            stopCh:  make(chan bool, 1),
        }
        rm.workers = append(rm.workers, worker)
    }
    
    rm.initialized = true
    fmt.Printf("[%s] Initialization complete\n", rm.name)
    return nil
}

func (rm *ResourceManager) Start() error {
    rm.mu.Lock()
    defer rm.mu.Unlock()
    
    if !rm.initialized {
        return fmt.Errorf("resource manager not initialized")
    }
    
    if rm.running {
        return fmt.Errorf("resource manager already running")
    }
    
    fmt.Printf("[%s] Starting...\n", rm.name)
    
    // Start workers
    for _, worker := range rm.workers {
        go worker.run()
    }
    
    // Start main loop
    go rm.mainLoop()
    
    rm.running = true
    fmt.Printf("[%s] Started with %d workers\n", rm.name, len(rm.workers))
    return nil
}

func (rm *ResourceManager) Stop() error {
    rm.mu.Lock()
    defer rm.mu.Unlock()
    
    if !rm.running {
        return fmt.Errorf("resource manager not running")
    }
    
    fmt.Printf("[%s] Stopping...\n", rm.name)
    
    // Stop all workers
    for _, worker := range rm.workers {
        worker.stopCh <- true
    }
    
    // Stop main loop
    rm.stopCh <- true
    
    // Wait for shutdown
    <-rm.doneCh
    
    rm.running = false
    fmt.Printf("[%s] Stopped\n", rm.name)
    return nil
}

func (rm *ResourceManager) Cleanup() error {
    rm.mu.Lock()
    defer rm.mu.Unlock()
    
    if rm.running {
        return fmt.Errorf("cannot cleanup while running")
    }
    
    fmt.Printf("[%s] Cleaning up...\n", rm.name)
    
    // Clean up resources
    for name := range rm.resources {
        fmt.Printf("[%s] Releasing resource: %s\n", rm.name, name)
        delete(rm.resources, name)
    }
    
    // Clean up workers
    rm.workers = rm.workers[:0]
    
    rm.initialized = false
    fmt.Printf("[%s] Cleanup complete\n", rm.name)
    return nil
}

func (rm *ResourceManager) mainLoop() {
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            fmt.Printf("[%s] Heartbeat - managing resources\n", rm.name)
        case <-rm.stopCh:
            fmt.Printf("[%s] Main loop stopping\n", rm.name)
            rm.doneCh <- true
            return
        }
    }
}

func (w *Worker) run() {
    fmt.Printf("[%s] Worker %d started\n", w.manager.name, w.ID)
    
    ticker := time.NewTicker(3 * time.Second)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            fmt.Printf("[%s] Worker %d processing\n", w.manager.name, w.ID)
        case <-w.stopCh:
            fmt.Printf("[%s] Worker %d stopping\n", w.manager.name, w.ID)
            return
        }
    }
}

func (rm *ResourceManager) Status() string {
    rm.mu.RLock()
    defer rm.mu.RUnlock()
    
    return fmt.Sprintf("Manager: %s, Initialized: %v, Running: %v, Workers: %d, Resources: %d",
        rm.name, rm.initialized, rm.running, len(rm.workers), len(rm.resources))
}

func main() {
    manager := NewResourceManager("AppManager")
    
    fmt.Println("Status:", manager.Status())
    
    // Lifecycle: Initialize -> Start -> Stop -> Cleanup
    manager.Initialize()
    manager.Start()
    
    fmt.Println("Status:", manager.Status())
    
    // Let it run for a while
    time.Sleep(8 * time.Second)
    
    manager.Stop()
    manager.Cleanup()
    
    fmt.Println("Final Status:", manager.Status())
}
```

Lifecycle management structs provide controlled initialization, startup,  
shutdown, and cleanup processes. This pattern is essential for managing  
resources, connections, and background processes in long-running applications.  

## Struct with plugin architecture

Structs can implement plugin systems for extensible functionality.  

```go
package main

import (
    "fmt"
    "sort"
)

// Plugin interface
type Plugin interface {
    Name() string
    Version() string
    Execute(input map[string]interface{}) (map[string]interface{}, error)
    Initialize() error
    Cleanup() error
}

// Plugin registry
type PluginRegistry struct {
    plugins map[string]Plugin
    hooks   map[string][]Plugin
}

func NewPluginRegistry() *PluginRegistry {
    return &PluginRegistry{
        plugins: make(map[string]Plugin),
        hooks:   make(map[string][]Plugin),
    }
}

func (pr *PluginRegistry) Register(plugin Plugin) error {
    name := plugin.Name()
    if _, exists := pr.plugins[name]; exists {
        return fmt.Errorf("plugin %s already registered", name)
    }
    
    if err := plugin.Initialize(); err != nil {
        return fmt.Errorf("failed to initialize plugin %s: %v", name, err)
    }
    
    pr.plugins[name] = plugin
    fmt.Printf("Registered plugin: %s v%s\n", plugin.Name(), plugin.Version())
    return nil
}

func (pr *PluginRegistry) AddHook(hookName string, plugin Plugin) {
    if pr.hooks[hookName] == nil {
        pr.hooks[hookName] = make([]Plugin, 0)
    }
    pr.hooks[hookName] = append(pr.hooks[hookName], plugin)
}

func (pr *PluginRegistry) ExecuteHook(hookName string, data map[string]interface{}) map[string]interface{} {
    plugins, exists := pr.hooks[hookName]
    if !exists {
        return data
    }
    
    result := data
    for _, plugin := range plugins {
        var err error
        result, err = plugin.Execute(result)
        if err != nil {
            fmt.Printf("Error executing plugin %s: %v\n", plugin.Name(), err)
        }
    }
    return result
}

func (pr *PluginRegistry) ListPlugins() []string {
    names := make([]string, 0, len(pr.plugins))
    for name := range pr.plugins {
        names = append(names, name)
    }
    sort.Strings(names)
    return names
}

func (pr *PluginRegistry) Cleanup() {
    for name, plugin := range pr.plugins {
        if err := plugin.Cleanup(); err != nil {
            fmt.Printf("Error cleaning up plugin %s: %v\n", name, err)
        }
    }
}

// Example plugins
type LoggerPlugin struct {
    logLevel string
}

func (lp *LoggerPlugin) Name() string { return "logger" }
func (lp *LoggerPlugin) Version() string { return "1.0.0" }

func (lp *LoggerPlugin) Initialize() error {
    lp.logLevel = "info"
    return nil
}

func (lp *LoggerPlugin) Execute(input map[string]interface{}) (map[string]interface{}, error) {
    message, ok := input["message"].(string)
    if !ok {
        message = "unknown"
    }
    fmt.Printf("[LOG] %s\n", message)
    
    output := make(map[string]interface{})
    for k, v := range input {
        output[k] = v
    }
    output["logged"] = true
    return output, nil
}

func (lp *LoggerPlugin) Cleanup() error {
    fmt.Println("Logger plugin cleaned up")
    return nil
}

type ValidatorPlugin struct{}

func (vp *ValidatorPlugin) Name() string { return "validator" }
func (vp *ValidatorPlugin) Version() string { return "2.1.0" }
func (vp *ValidatorPlugin) Initialize() error { return nil }

func (vp *ValidatorPlugin) Execute(input map[string]interface{}) (map[string]interface{}, error) {
    // Simple validation: check if required fields exist
    requiredFields := []string{"user_id", "action"}
    
    for _, field := range requiredFields {
        if _, exists := input[field]; !exists {
            return input, fmt.Errorf("missing required field: %s", field)
        }
    }
    
    output := make(map[string]interface{})
    for k, v := range input {
        output[k] = v
    }
    output["validated"] = true
    return output, nil
}

func (vp *ValidatorPlugin) Cleanup() error {
    fmt.Println("Validator plugin cleaned up")
    return nil
}

type MetricsPlugin struct {
    eventCount int
}

func (mp *MetricsPlugin) Name() string { return "metrics" }
func (mp *MetricsPlugin) Version() string { return "1.2.0" }
func (mp *MetricsPlugin) Initialize() error { return nil }

func (mp *MetricsPlugin) Execute(input map[string]interface{}) (map[string]interface{}, error) {
    mp.eventCount++
    fmt.Printf("[METRICS] Event count: %d\n", mp.eventCount)
    
    output := make(map[string]interface{})
    for k, v := range input {
        output[k] = v
    }
    output["metrics_recorded"] = true
    return output, nil
}

func (mp *MetricsPlugin) Cleanup() error {
    fmt.Printf("Metrics plugin cleaned up (final count: %d)\n", mp.eventCount)
    return nil
}

// Application with plugin support
type Application struct {
    registry *PluginRegistry
}

func NewApplication() *Application {
    return &Application{
        registry: NewPluginRegistry(),
    }
}

func (app *Application) LoadPlugins() error {
    plugins := []Plugin{
        &ValidatorPlugin{},
        &LoggerPlugin{},
        &MetricsPlugin{},
    }
    
    for _, plugin := range plugins {
        if err := app.registry.Register(plugin); err != nil {
            return err
        }
        
        // Register hooks based on plugin type
        switch plugin.Name() {
        case "validator":
            app.registry.AddHook("pre_process", plugin)
        case "logger":
            app.registry.AddHook("post_process", plugin)
        case "metrics":
            app.registry.AddHook("post_process", plugin)
        }
    }
    
    return nil
}

func (app *Application) ProcessEvent(eventData map[string]interface{}) {
    fmt.Printf("Processing event: %+v\n", eventData)
    
    // Execute pre-processing hooks
    data := app.registry.ExecuteHook("pre_process", eventData)
    
    // Main processing (simulated)
    fmt.Println("  -> Main processing...")
    data["processed"] = true
    
    // Execute post-processing hooks
    data = app.registry.ExecuteHook("post_process", data)
    
    fmt.Printf("Final result: %+v\n\n", data)
}

func (app *Application) Shutdown() {
    app.registry.Cleanup()
}

func main() {
    app := NewApplication()
    
    // Load plugins
    if err := app.LoadPlugins(); err != nil {
        fmt.Printf("Error loading plugins: %v\n", err)
        return
    }
    
    fmt.Printf("Loaded plugins: %v\n\n", app.registry.ListPlugins())
    
    // Process some events
    events := []map[string]interface{}{
        {"user_id": 123, "action": "login", "message": "User logged in"},
        {"user_id": 456, "action": "purchase", "message": "User made purchase"},
        {"action": "logout", "message": "Missing user_id"}, // This will fail validation
    }
    
    for _, event := range events {
        app.ProcessEvent(event)
    }
    
    // Cleanup
    app.Shutdown()
}
```

Plugin architecture structs enable extensible applications where functionality  
can be added dynamically. This pattern includes plugin registration, hooks,  
lifecycle management, and error handling for robust plugin systems.  