# Interfaces

Interfaces in Go are one of the most powerful and distinctive features of the  
language, providing a way to specify behavior without implementing it. An interface  
defines a contract by specifying a set of method signatures that a type must  
implement to satisfy that interface. Unlike many other programming languages,  
Go uses implicit interface implementation - any type that has the required methods  
automatically satisfies the interface, without explicit declaration.

This implicit satisfaction mechanism promotes loose coupling and makes code more  
flexible and testable. Interfaces enable polymorphism in Go, allowing different  
types to be treated uniformly when they implement the same interface. This design  
philosophy encourages composition over inheritance and makes it easy to write  
modular, extensible code that follows the principle of programming to interfaces  
rather than concrete implementations.

Go's interface system is designed around small, focused interfaces that do one  
thing well. The standard library exemplifies this with interfaces like io.Reader,  
io.Writer, and fmt.Stringer, each defining just one or two methods. This approach  
makes interfaces easy to implement and compose, leading to more maintainable  
and flexible code architectures.

Interfaces can be empty (containing no methods), which allows them to hold values  
of any type. The empty interface interface{} (now aliased as 'any' in Go 1.18+)  
is commonly used when the type is unknown at compile time. Type assertions and  
type switches provide ways to extract the underlying concrete type from interface  
values when needed.

Go 1.18 introduced type parameters and constraints, which use interfaces to  
specify what methods generic types must support. This extends the power of  
interfaces beyond runtime polymorphism to compile-time generic programming,  
enabling more flexible and reusable code while maintaining type safety.

## Basic interface definition

The most fundamental concept is defining an interface and implementing it  
implicitly through methods.  

```go
package main

import "fmt"

type Greeter interface {
    Greet() string
}

type Person struct {
    Name string
}

func (p Person) Greet() string {
    return fmt.Sprintf("Hello there, I'm %s", p.Name)
}

type Robot struct {
    Model string
}

func (r Robot) Greet() string {
    return fmt.Sprintf("Greetings, I am %s unit", r.Model)
}

func main() {
    var greeter Greeter
    
    person := Person{Name: "Alice"}
    robot := Robot{Model: "R2D2"}
    
    greeter = person
    fmt.Println(greeter.Greet())
    
    greeter = robot
    fmt.Println(greeter.Greet())
}
```

This example demonstrates implicit interface implementation where both Person  
and Robot types automatically satisfy the Greeter interface by having a Greet  
method with the matching signature. No explicit declaration is needed.  

## Interface with multiple methods

Interfaces can define multiple methods that implementing types must provide.  

```go
package main

import "fmt"

type Shape interface {
    Area() float64
    Perimeter() float64
    String() string
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func (r Rectangle) String() string {
    return fmt.Sprintf("Rectangle(%.1fx%.1f)", r.Width, r.Height)
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

func (c Circle) String() string {
    return fmt.Sprintf("Circle(radius=%.1f)", c.Radius)
}

func printShapeInfo(s Shape) {
    fmt.Printf("%s: Area=%.2f, Perimeter=%.2f\n", 
               s.String(), s.Area(), s.Perimeter())
}

func main() {
    shapes := []Shape{
        Rectangle{Width: 5, Height: 3},
        Circle{Radius: 2.5},
    }
    
    for _, shape := range shapes {
        printShapeInfo(shape)
    }
}
```

Multi-method interfaces define a contract that requires all specified methods  
to be implemented. This enables rich polymorphic behavior where different types  
can provide their own implementations while being used uniformly.  

## Empty interface

The empty interface can hold values of any type, providing maximum flexibility  
when types are unknown at compile time.  

```go
package main

import "fmt"

func printAnything(value any) {
    fmt.Printf("Value: %v, Type: %T\n", value, value)
}

func processValue(value any) {
    switch v := value.(type) {
    case string:
        fmt.Printf("String: '%s' (length: %d)\n", v, len(v))
    case int:
        fmt.Printf("Integer: %d (doubled: %d)\n", v, v*2)
    case float64:
        fmt.Printf("Float: %.2f (squared: %.2f)\n", v, v*v)
    case bool:
        fmt.Printf("Boolean: %t (negated: %t)\n", v, !v)
    default:
        fmt.Printf("Unknown type: %T with value %v\n", v, v)
    }
}

func main() {
    var anything any
    
    values := []any{
        "hello there",
        42,
        3.14159,
        true,
        []int{1, 2, 3},
        map[string]int{"age": 25},
    }
    
    for _, v := range values {
        printAnything(v)
        processValue(v)
        fmt.Println()
    }
    
    // Demonstrating assignment flexibility
    anything = "text"
    printAnything(anything)
    
    anything = 100
    printAnything(anything)
}
```

The empty interface (any) provides ultimate flexibility for handling values of  
unknown types. Type switches and type assertions are commonly used to work  
with the concrete values stored in empty interfaces.  

## Type assertions

Type assertions provide a way to extract the underlying concrete value from  
an interface value safely.  

```go
package main

import (
    "fmt"
    "strings"
)

func examineValue(value any) {
    // Safe type assertion with ok idiom
    if str, ok := value.(string); ok {
        fmt.Printf("String value: '%s' (uppercase: %s)\n", 
                   str, strings.ToUpper(str))
        return
    }
    
    if num, ok := value.(int); ok {
        fmt.Printf("Integer value: %d (factorial: %d)\n", 
                   num, factorial(num))
        return
    }
    
    if slice, ok := value.([]int); ok {
        total := 0
        for _, n := range slice {
            total += n
        }
        fmt.Printf("Integer slice: %v (sum: %d)\n", slice, total)
        return
    }
    
    // Direct type assertion (can panic if wrong type)
    fmt.Printf("Unhandled type: %T with value %v\n", value, value)
}

func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}

func main() {
    values := []any{
        "hello there",
        5,
        []int{1, 2, 3, 4, 5},
        3.14,
        true,
    }
    
    for _, v := range values {
        examineValue(v)
    }
    
    // Demonstrating unsafe assertion (commented to avoid panic)
    // str := values[0].(int) // This would panic!
}
```

The comma-ok idiom (value, ok := interface.(Type)) provides safe type assertions  
that won't panic if the assertion fails. Always prefer this form unless you're  
absolutely certain of the underlying type.  

## Interface composition

Interfaces can be composed by embedding other interfaces, creating larger  
contracts from smaller, focused ones.  

```go
package main

import (
    "fmt"
    "strings"
)

// Small, focused interfaces
type Reader interface {
    Read() string
}

type Writer interface {
    Write(data string) error
}

type Closer interface {
    Close() error
}

// Composed interfaces
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

// Implementation
type FileHandler struct {
    filename string
    content  strings.Builder
    closed   bool
}

func (f *FileHandler) Read() string {
    if f.closed {
        return ""
    }
    return f.content.String()
}

func (f *FileHandler) Write(data string) error {
    if f.closed {
        return fmt.Errorf("file %s is closed", f.filename)
    }
    f.content.WriteString(data)
    return nil
}

func (f *FileHandler) Close() error {
    if f.closed {
        return fmt.Errorf("file %s already closed", f.filename)
    }
    f.closed = true
    fmt.Printf("File %s closed\n", f.filename)
    return nil
}

func processFile(rwc ReadWriteCloser) {
    err := rwc.Write("hello there\n")
    if err != nil {
        fmt.Printf("Write error: %v\n", err)
        return
    }
    
    content := rwc.Read()
    fmt.Printf("Content: %s", content)
    
    err = rwc.Close()
    if err != nil {
        fmt.Printf("Close error: %v\n", err)
    }
}

func main() {
    file := &FileHandler{filename: "test.txt"}
    processFile(file)
    
    // Try to write after closing
    err := file.Write("more data")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

Interface composition allows building complex contracts from simple, focused  
interfaces. This promotes the single responsibility principle and makes  
interfaces easier to implement and test.  

## Interface with generic type constraints

Go 1.18+ allows interfaces to be used as type constraints in generic functions,  
enabling more flexible and type-safe generic programming.  

```go
package main

import "fmt"

// Numeric constraint interface
type Numeric interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64
}

// Interface constraint with methods
type Stringer interface {
    String() string
}

// Interface constraint combining type unions and methods
type Ordered interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64 | string
}

func Sum[T Numeric](slice []T) T {
    var total T
    for _, v := range slice {
        total += v
    }
    return total
}

func PrintItems[T Stringer](items []T) {
    for i, item := range items {
        fmt.Printf("Item %d: %s\n", i, item.String())
    }
}

func Max[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

type Product struct {
    Name  string
    Price float64
}

func (p Product) String() string {
    return fmt.Sprintf("%s ($%.2f)", p.Name, p.Price)
}

func main() {
    // Using numeric constraint
    intNumbers := []int{1, 2, 3, 4, 5}
    floatNumbers := []float64{1.1, 2.2, 3.3}
    
    fmt.Printf("Sum of ints: %d\n", Sum(intNumbers))
    fmt.Printf("Sum of floats: %.2f\n", Sum(floatNumbers))
    
    // Using Stringer constraint
    products := []Product{
        {"Laptop", 999.99},
        {"Mouse", 29.99},
    }
    
    PrintItems(products)
    
    // Using Ordered constraint
    fmt.Printf("Max int: %d\n", Max(10, 20))
    fmt.Printf("Max string: %s\n", Max("apple", "banana"))
    fmt.Printf("Max float: %.2f\n", Max(3.14, 2.71))
}
```

Generic interfaces provide powerful type constraints that enable writing  
reusable, type-safe generic code. Union constraints allow multiple types,  
while method constraints require specific method implementations.  

## Using standard library interfaces

Go's standard library provides many useful interfaces that your types can  
implement to gain powerful functionality.  

```go
package main

import (
    "fmt"
    "io"
    "sort"
    "strings"
)

// Implementing fmt.Stringer
type Temperature struct {
    celsius float64
}

func (t Temperature) String() string {
    fahrenheit := t.celsius*9/5 + 32
    return fmt.Sprintf("%.1f°C (%.1f°F)", t.celsius, fahrenheit)
}

// Implementing sort.Interface
type People []Person

func (p People) Len() int           { return len(p) }
func (p People) Less(i, j int) bool { return p[i].Age < p[j].Age }
func (p People) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

type Person struct {
    Name string
    Age  int
}

// Implementing io.Writer
type Logger struct {
    prefix string
}

func (l Logger) Write(data []byte) (int, error) {
    message := fmt.Sprintf("[%s] %s", l.prefix, string(data))
    fmt.Print(message)
    return len(data), nil
}

func main() {
    // Using fmt.Stringer
    temp := Temperature{celsius: 25.5}
    fmt.Printf("Temperature: %s\n", temp)
    
    // Using sort.Interface
    people := People{
        {"Bob", 30},
        {"Alice", 25},
        {"Charlie", 35},
    }
    
    fmt.Printf("Before sorting: %+v\n", people)
    sort.Sort(people)
    fmt.Printf("After sorting: %+v\n", people)
    
    // Using io.Writer
    logger := Logger{prefix: "INFO"}
    io.WriteString(&logger, "hello there from logger\n")
    
    // Multiple writers
    multiWriter := io.MultiWriter(&logger, &strings.Builder{})
    io.WriteString(multiWriter, "Broadcasting message\n")
}
```

Standard library interfaces provide well-tested contracts that integrate  
seamlessly with Go's ecosystem. Implementing these interfaces gives your  
types access to powerful built-in functionality.  

## Interface for dependency injection

Interfaces enable clean dependency injection by defining contracts rather  
than depending on concrete implementations.  

```go
package main

import (
    "fmt"
    "time"
)

// Database interface
type Database interface {
    Save(key, value string) error
    Load(key string) (string, error)
}

// Email service interface
type EmailService interface {
    SendEmail(to, subject, body string) error
}

// Logger interface
type Logger interface {
    Log(level, message string)
}

// Concrete implementations
type InMemoryDB struct {
    data map[string]string
}

func (db *InMemoryDB) Save(key, value string) error {
    if db.data == nil {
        db.data = make(map[string]string)
    }
    db.data[key] = value
    return nil
}

func (db *InMemoryDB) Load(key string) (string, error) {
    if db.data == nil {
        return "", fmt.Errorf("key not found: %s", key)
    }
    value, exists := db.data[key]
    if !exists {
        return "", fmt.Errorf("key not found: %s", key)
    }
    return value, nil
}

type MockEmailService struct{}

func (e *MockEmailService) SendEmail(to, subject, body string) error {
    fmt.Printf("Mock email sent to %s: %s\n", to, subject)
    return nil
}

type ConsoleLogger struct{}

func (l *ConsoleLogger) Log(level, message string) {
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    fmt.Printf("[%s] %s: %s\n", timestamp, level, message)
}

// Service that depends on interfaces
type UserService struct {
    db     Database
    email  EmailService
    logger Logger
}

func NewUserService(db Database, email EmailService, logger Logger) *UserService {
    return &UserService{
        db:     db,
        email:  email,
        logger: logger,
    }
}

func (us *UserService) CreateUser(username, email string) error {
    us.logger.Log("INFO", fmt.Sprintf("Creating user: %s", username))
    
    err := us.db.Save(username, email)
    if err != nil {
        us.logger.Log("ERROR", fmt.Sprintf("Failed to save user: %v", err))
        return err
    }
    
    err = us.email.SendEmail(email, "Welcome!", "Welcome to our service!")
    if err != nil {
        us.logger.Log("ERROR", fmt.Sprintf("Failed to send email: %v", err))
        return err
    }
    
    us.logger.Log("INFO", fmt.Sprintf("User created successfully: %s", username))
    return nil
}

func main() {
    // Inject dependencies
    db := &InMemoryDB{}
    emailService := &MockEmailService{}
    logger := &ConsoleLogger{}
    
    userService := NewUserService(db, emailService, logger)
    
    err := userService.CreateUser("alice", "alice@example.com")
    if err != nil {
        fmt.Printf("Error creating user: %v\n", err)
    }
}
```

Dependency injection through interfaces promotes testability, flexibility,  
and loose coupling. Services depend on behavior contracts rather than  
concrete implementations, making code easier to test and modify.  

## Interface for strategy pattern

The strategy pattern uses interfaces to encapsulate algorithms and make  
them interchangeable at runtime.  

```go
package main

import (
    "fmt"
    "sort"
)

// Payment strategy interface
type PaymentStrategy interface {
    Pay(amount float64) error
    GetName() string
}

// Concrete payment strategies
type CreditCardPayment struct {
    cardNumber string
    cvv        string
}

func (cc *CreditCardPayment) Pay(amount float64) error {
    fmt.Printf("Paid $%.2f using Credit Card ****%s\n", 
               amount, cc.cardNumber[len(cc.cardNumber)-4:])
    return nil
}

func (cc *CreditCardPayment) GetName() string {
    return "Credit Card"
}

type PayPalPayment struct {
    email string
}

func (pp *PayPalPayment) Pay(amount float64) error {
    fmt.Printf("Paid $%.2f using PayPal account %s\n", amount, pp.email)
    return nil
}

func (pp *PayPalPayment) GetName() string {
    return "PayPal"
}

type BankTransferPayment struct {
    accountNumber string
}

func (bt *BankTransferPayment) Pay(amount float64) error {
    fmt.Printf("Paid $%.2f using Bank Transfer to ****%s\n", 
               amount, bt.accountNumber[len(bt.accountNumber)-4:])
    return nil
}

func (bt *BankTransferPayment) GetName() string {
    return "Bank Transfer"
}

// Context that uses strategy
type ShoppingCart struct {
    items    []Item
    strategy PaymentStrategy
}

type Item struct {
    name  string
    price float64
}

func (sc *ShoppingCart) AddItem(item Item) {
    sc.items = append(sc.items, item)
}

func (sc *ShoppingCart) SetPaymentStrategy(strategy PaymentStrategy) {
    sc.strategy = strategy
}

func (sc *ShoppingCart) GetTotal() float64 {
    total := 0.0
    for _, item := range sc.items {
        total += item.price
    }
    return total
}

func (sc *ShoppingCart) Checkout() error {
    if sc.strategy == nil {
        return fmt.Errorf("payment strategy not set")
    }
    
    total := sc.GetTotal()
    fmt.Printf("Processing payment of $%.2f using %s\n", 
               total, sc.strategy.GetName())
    
    return sc.strategy.Pay(total)
}

func main() {
    cart := &ShoppingCart{}
    cart.AddItem(Item{"Laptop", 999.99})
    cart.AddItem(Item{"Mouse", 29.99})
    
    // Try different payment strategies
    strategies := []PaymentStrategy{
        &CreditCardPayment{cardNumber: "1234567890123456", cvv: "123"},
        &PayPalPayment{email: "user@example.com"},
        &BankTransferPayment{accountNumber: "9876543210"},
    }
    
    for _, strategy := range strategies {
        cart.SetPaymentStrategy(strategy)
        err := cart.Checkout()
        if err != nil {
            fmt.Printf("Payment failed: %v\n", err)
        }
        fmt.Println()
    }
}
```

The strategy pattern with interfaces allows algorithms to be selected and  
swapped at runtime. This provides flexibility and makes code more  
maintainable by separating algorithm implementation from usage.  

## Interface for observer pattern

The observer pattern uses interfaces to define how objects can be notified  
of changes in other objects.  

```go
package main

import "fmt"

// Observer interface
type Observer interface {
    Update(event string, data any)
    GetID() string
}

// Subject interface
type Subject interface {
    Attach(observer Observer)
    Detach(observer Observer)
    Notify(event string, data any)
}

// Concrete observer implementations
type EmailNotifier struct {
    id    string
    email string
}

func (en *EmailNotifier) Update(event string, data any) {
    fmt.Printf("Email notification to %s: %s - %v\n", 
               en.email, event, data)
}

func (en *EmailNotifier) GetID() string {
    return en.id
}

type SMSNotifier struct {
    id    string
    phone string
}

func (sn *SMSNotifier) Update(event string, data any) {
    fmt.Printf("SMS notification to %s: %s - %v\n", 
               sn.phone, event, data)
}

func (sn *SMSNotifier) GetID() string {
    return sn.id
}

type LogNotifier struct {
    id string
}

func (ln *LogNotifier) Update(event string, data any) {
    fmt.Printf("Log entry: %s - %v\n", event, data)
}

func (ln *LogNotifier) GetID() string {
    return ln.id
}

// Concrete subject
type NewsPublisher struct {
    observers []Observer
    articles  []string
}

func (np *NewsPublisher) Attach(observer Observer) {
    np.observers = append(np.observers, observer)
    fmt.Printf("Observer %s attached\n", observer.GetID())
}

func (np *NewsPublisher) Detach(observer Observer) {
    for i, obs := range np.observers {
        if obs.GetID() == observer.GetID() {
            np.observers = append(np.observers[:i], np.observers[i+1:]...)
            fmt.Printf("Observer %s detached\n", observer.GetID())
            break
        }
    }
}

func (np *NewsPublisher) Notify(event string, data any) {
    fmt.Printf("Notifying %d observers about: %s\n", 
               len(np.observers), event)
    for _, observer := range np.observers {
        observer.Update(event, data)
    }
    fmt.Println()
}

func (np *NewsPublisher) PublishArticle(title, content string) {
    np.articles = append(np.articles, title)
    
    data := map[string]string{
        "title":   title,
        "content": content,
    }
    
    np.Notify("NEW_ARTICLE", data)
}

func (np *NewsPublisher) Breaking News(title string) {
    data := map[string]string{
        "title":    title,
        "priority": "HIGH",
    }
    
    np.Notify("BREAKING_NEWS", data)
}

func main() {
    publisher := &NewsPublisher{}
    
    // Create observers
    emailNotifier := &EmailNotifier{
        id:    "email1",
        email: "user@example.com",
    }
    smsNotifier := &SMSNotifier{
        id:    "sms1",
        phone: "+1234567890",
    }
    logNotifier := &LogNotifier{
        id: "log1",
    }
    
    // Attach observers
    publisher.Attach(emailNotifier)
    publisher.Attach(smsNotifier)
    publisher.Attach(logNotifier)
    
    // Publish content
    publisher.PublishArticle("Go Interfaces Guide", "Learn about Go interfaces...")
    
    // Detach one observer
    publisher.Detach(smsNotifier)
    
    // Publish breaking news
    publisher.BreakingNews("Important Update!")
}
```

The observer pattern with interfaces enables loose coupling between subjects  
and observers. Objects can be notified of changes without having direct  
references to each other, promoting maintainable and extensible designs.  

## Interface for command pattern

The command pattern encapsulates requests as objects, enabling parameterization  
of clients with different requests and support for undoable operations.  

```go
package main

import (
    "fmt"
    "strings"
)

// Command interface
type Command interface {
    Execute() error
    Undo() error
    GetDescription() string
}

// Receiver - text editor
type TextEditor struct {
    content strings.Builder
    history []string
}

func (te *TextEditor) GetContent() string {
    return te.content.String()
}

func (te *TextEditor) saveState() {
    te.history = append(te.history, te.content.String())
}

func (te *TextEditor) restoreState(state string) {
    te.content.Reset()
    te.content.WriteString(state)
}

// Concrete commands
type WriteCommand struct {
    editor *TextEditor
    text   string
    prevState string
}

func (wc *WriteCommand) Execute() error {
    wc.editor.saveState()
    if len(wc.editor.history) > 0 {
        wc.prevState = wc.editor.history[len(wc.editor.history)-1]
    }
    wc.editor.content.WriteString(wc.text)
    return nil
}

func (wc *WriteCommand) Undo() error {
    wc.editor.restoreState(wc.prevState)
    return nil
}

func (wc *WriteCommand) GetDescription() string {
    return fmt.Sprintf("Write: '%s'", wc.text)
}

type DeleteCommand struct {
    editor    *TextEditor
    numChars  int
    deleted   string
    prevState string
}

func (dc *DeleteCommand) Execute() error {
    dc.editor.saveState()
    if len(dc.editor.history) > 0 {
        dc.prevState = dc.editor.history[len(dc.editor.history)-1]
    }
    
    content := dc.editor.content.String()
    if len(content) >= dc.numChars {
        dc.deleted = content[len(content)-dc.numChars:]
        dc.editor.content.Reset()
        dc.editor.content.WriteString(content[:len(content)-dc.numChars])
    }
    return nil
}

func (dc *DeleteCommand) Undo() error {
    dc.editor.restoreState(dc.prevState)
    return nil
}

func (dc *DeleteCommand) GetDescription() string {
    return fmt.Sprintf("Delete %d chars", dc.numChars)
}

// Invoker - command manager
type CommandManager struct {
    history []Command
    current int
}

func (cm *CommandManager) Execute(cmd Command) error {
    // Remove any commands after current position
    cm.history = cm.history[:cm.current]
    
    err := cmd.Execute()
    if err != nil {
        return err
    }
    
    cm.history = append(cm.history, cmd)
    cm.current++
    
    fmt.Printf("Executed: %s\n", cmd.GetDescription())
    return nil
}

func (cm *CommandManager) Undo() error {
    if cm.current == 0 {
        return fmt.Errorf("nothing to undo")
    }
    
    cm.current--
    cmd := cm.history[cm.current]
    err := cmd.Undo()
    if err != nil {
        cm.current++
        return err
    }
    
    fmt.Printf("Undid: %s\n", cmd.GetDescription())
    return nil
}

func (cm *CommandManager) Redo() error {
    if cm.current >= len(cm.history) {
        return fmt.Errorf("nothing to redo")
    }
    
    cmd := cm.history[cm.current]
    err := cmd.Execute()
    if err != nil {
        return err
    }
    
    cm.current++
    fmt.Printf("Redid: %s\n", cmd.GetDescription())
    return nil
}

func main() {
    editor := &TextEditor{}
    manager := &CommandManager{}
    
    // Execute commands
    manager.Execute(&WriteCommand{editor: editor, text: "hello there "})
    manager.Execute(&WriteCommand{editor: editor, text: "world!"})
    fmt.Printf("Content: '%s'\n\n", editor.GetContent())
    
    manager.Execute(&DeleteCommand{editor: editor, numChars: 6})
    fmt.Printf("Content after delete: '%s'\n\n", editor.GetContent())
    
    // Undo operations
    manager.Undo()
    fmt.Printf("Content after undo: '%s'\n\n", editor.GetContent())
    
    manager.Undo()
    fmt.Printf("Content after second undo: '%s'\n\n", editor.GetContent())
    
    // Redo operation
    manager.Redo()
    fmt.Printf("Content after redo: '%s'\n", editor.GetContent())
}
```

The command pattern with interfaces enables undoable operations, macro  
recording, and queuing of requests. Commands encapsulate all information  
needed to perform an action, including the ability to reverse it.  

## Interface for factory pattern

Factories use interfaces to create objects without specifying their exact  
classes, promoting loose coupling and flexibility.  

```go
package main

import (
    "fmt"
    "strings"
)

// Product interface
type DatabaseConnection interface {
    Connect() error
    Query(sql string) ([]map[string]any, error)
    Close() error
    GetType() string
}

// Concrete products
type MySQLConnection struct {
    host     string
    username string
    database string
    connected bool
}

func (mc *MySQLConnection) Connect() error {
    fmt.Printf("Connecting to MySQL at %s\n", mc.host)
    mc.connected = true
    return nil
}

func (mc *MySQLConnection) Query(sql string) ([]map[string]any, error) {
    if !mc.connected {
        return nil, fmt.Errorf("not connected to database")
    }
    
    fmt.Printf("MySQL query: %s\n", sql)
    
    // Mock result
    result := []map[string]any{
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"},
    }
    
    return result, nil
}

func (mc *MySQLConnection) Close() error {
    fmt.Println("Closing MySQL connection")
    mc.connected = false
    return nil
}

func (mc *MySQLConnection) GetType() string {
    return "MySQL"
}

type PostgreSQLConnection struct {
    host     string
    username string
    database string
    connected bool
}

func (pc *PostgreSQLConnection) Connect() error {
    fmt.Printf("Connecting to PostgreSQL at %s\n", pc.host)
    pc.connected = true
    return nil
}

func (pc *PostgreSQLConnection) Query(sql string) ([]map[string]any, error) {
    if !pc.connected {
        return nil, fmt.Errorf("not connected to database")
    }
    
    fmt.Printf("PostgreSQL query: %s\n", sql)
    
    // Mock result
    result := []map[string]any{
        {"id": 1, "name": "Charlie"},
        {"id": 2, "name": "Diana"},
    }
    
    return result, nil
}

func (pc *PostgreSQLConnection) Close() error {
    fmt.Println("Closing PostgreSQL connection")
    pc.connected = false
    return nil
}

func (pc *PostgreSQLConnection) GetType() string {
    return "PostgreSQL"
}

// Factory interface
type DatabaseFactory interface {
    CreateConnection(config map[string]string) (DatabaseConnection, error)
    GetSupportedTypes() []string
}

// Concrete factories
type MySQLFactory struct{}

func (mf *MySQLFactory) CreateConnection(config map[string]string) (DatabaseConnection, error) {
    return &MySQLConnection{
        host:     config["host"],
        username: config["username"],
        database: config["database"],
    }, nil
}

func (mf *MySQLFactory) GetSupportedTypes() []string {
    return []string{"mysql"}
}

type PostgreSQLFactory struct{}

func (pf *PostgreSQLFactory) CreateConnection(config map[string]string) (DatabaseConnection, error) {
    return &PostgreSQLConnection{
        host:     config["host"],
        username: config["username"],
        database: config["database"],
    }, nil
}

func (pf *PostgreSQLFactory) GetSupportedTypes() []string {
    return []string{"postgresql", "postgres"}
}

// Factory registry
type DatabaseFactoryRegistry struct {
    factories map[string]DatabaseFactory
}

func NewDatabaseFactoryRegistry() *DatabaseFactoryRegistry {
    registry := &DatabaseFactoryRegistry{
        factories: make(map[string]DatabaseFactory),
    }
    
    // Register default factories
    registry.RegisterFactory("mysql", &MySQLFactory{})
    registry.RegisterFactory("postgresql", &PostgreSQLFactory{})
    registry.RegisterFactory("postgres", &PostgreSQLFactory{})
    
    return registry
}

func (dfr *DatabaseFactoryRegistry) RegisterFactory(dbType string, factory DatabaseFactory) {
    dfr.factories[strings.ToLower(dbType)] = factory
}

func (dfr *DatabaseFactoryRegistry) CreateConnection(dbType string, config map[string]string) (DatabaseConnection, error) {
    factory, exists := dfr.factories[strings.ToLower(dbType)]
    if !exists {
        return nil, fmt.Errorf("unsupported database type: %s", dbType)
    }
    
    return factory.CreateConnection(config)
}

func (dfr *DatabaseFactoryRegistry) GetSupportedTypes() []string {
    var types []string
    for dbType := range dfr.factories {
        types = append(types, dbType)
    }
    return types
}

func main() {
    registry := NewDatabaseFactoryRegistry()
    
    configs := []struct {
        dbType string
        config map[string]string
    }{
        {
            dbType: "mysql",
            config: map[string]string{
                "host":     "localhost:3306",
                "username": "root",
                "database": "testdb",
            },
        },
        {
            dbType: "postgresql",
            config: map[string]string{
                "host":     "localhost:5432",
                "username": "postgres",
                "database": "testdb",
            },
        },
    }
    
    for _, cfg := range configs {
        fmt.Printf("\n--- Testing %s ---\n", cfg.dbType)
        
        conn, err := registry.CreateConnection(cfg.dbType, cfg.config)
        if err != nil {
            fmt.Printf("Failed to create connection: %v\n", err)
            continue
        }
        
        err = conn.Connect()
        if err != nil {
            fmt.Printf("Failed to connect: %v\n", err)
            continue
        }
        
        results, err := conn.Query("SELECT * FROM users")
        if err != nil {
            fmt.Printf("Query failed: %v\n", err)
        } else {
            fmt.Printf("Query results: %v\n", results)
        }
        
        conn.Close()
    }
    
    fmt.Printf("\nSupported types: %v\n", registry.GetSupportedTypes())
}
```

The factory pattern with interfaces provides a way to create objects without  
specifying their exact classes. This promotes loose coupling and makes it  
easy to extend the system with new implementations.  

## Interface for adapter pattern

The adapter pattern uses interfaces to allow incompatible interfaces to  
work together by wrapping an existing class with a new interface.  

```go
package main

import (
    "encoding/json"
    "encoding/xml"
    "fmt"
)

// Target interface that client expects
type DataProcessor interface {
    ProcessData(data []byte) ([]User, error)
    GetFormat() string
}

type User struct {
    ID    int    `json:"id" xml:"id"`
    Name  string `json:"name" xml:"name"`
    Email string `json:"email" xml:"email"`
}

// Adaptee - existing JSON processor
type JSONProcessor struct{}

func (jp *JSONProcessor) ParseJSON(jsonData []byte) ([]User, error) {
    var users []User
    err := json.Unmarshal(jsonData, &users)
    return users, err
}

// Adaptee - existing XML processor  
type XMLProcessor struct{}

func (xp *XMLProcessor) ParseXML(xmlData []byte) ([]User, error) {
    type XMLRoot struct {
        Users []User `xml:"user"`
    }
    
    var root XMLRoot
    err := xml.Unmarshal(xmlData, &root)
    return root.Users, err
}

// Adapters that implement the target interface
type JSONAdapter struct {
    processor *JSONProcessor
}

func (ja *JSONAdapter) ProcessData(data []byte) ([]User, error) {
    return ja.processor.ParseJSON(data)
}

func (ja *JSONAdapter) GetFormat() string {
    return "JSON"
}

type XMLAdapter struct {
    processor *XMLProcessor
}

func (xa *XMLAdapter) ProcessData(data []byte) ([]User, error) {
    return xa.processor.ParseXML(data)
}

func (xa *XMLAdapter) GetFormat() string {
    return "XML"
}

// Client that uses the target interface
type DataManager struct {
    processor DataProcessor
}

func (dm *DataManager) SetProcessor(processor DataProcessor) {
    dm.processor = processor
}

func (dm *DataManager) ImportUsers(data []byte) error {
    if dm.processor == nil {
        return fmt.Errorf("no processor set")
    }
    
    fmt.Printf("Processing data using %s processor\n", 
               dm.processor.GetFormat())
    
    users, err := dm.processor.ProcessData(data)
    if err != nil {
        return fmt.Errorf("failed to process data: %v", err)
    }
    
    fmt.Printf("Successfully processed %d users:\n", len(users))
    for _, user := range users {
        fmt.Printf("- ID: %d, Name: %s, Email: %s\n", 
                   user.ID, user.Name, user.Email)
    }
    
    return nil
}

func main() {
    manager := &DataManager{}
    
    // JSON data
    jsonData := `[
        {"id": 1, "name": "Alice", "email": "alice@example.com"},
        {"id": 2, "name": "Bob", "email": "bob@example.com"}
    ]`
    
    // XML data
    xmlData := `
    <root>
        <user>
            <id>3</id>
            <name>Charlie</name>
            <email>charlie@example.com</email>
        </user>
        <user>
            <id>4</id>
            <name>Diana</name>
            <email>diana@example.com</email>
        </user>
    </root>`
    
    // Use JSON adapter
    jsonAdapter := &JSONAdapter{processor: &JSONProcessor{}}
    manager.SetProcessor(jsonAdapter)
    
    fmt.Println("--- Processing JSON Data ---")
    err := manager.ImportUsers([]byte(jsonData))
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Use XML adapter
    xmlAdapter := &XMLAdapter{processor: &XMLProcessor{}}
    manager.SetProcessor(xmlAdapter)
    
    fmt.Println("\n--- Processing XML Data ---")
    err = manager.ImportUsers([]byte(xmlData))
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

The adapter pattern allows existing classes with incompatible interfaces  
to work together. Adapters wrap the existing functionality and present  
it through a common interface that clients expect.  

## Interface for decorator pattern

The decorator pattern uses interfaces to add behavior to objects dynamically  
without altering their structure.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Component interface
type Coffee interface {
    GetDescription() string
    GetCost() float64
}

// Concrete component
type SimpleCoffee struct{}

func (sc *SimpleCoffee) GetDescription() string {
    return "Simple coffee"
}

func (sc *SimpleCoffee) GetCost() float64 {
    return 2.00
}

// Base decorator
type CoffeeDecorator struct {
    coffee Coffee
}

func (cd *CoffeeDecorator) GetDescription() string {
    return cd.coffee.GetDescription()
}

func (cd *CoffeeDecorator) GetCost() float64 {
    return cd.coffee.GetCost()
}

// Concrete decorators
type MilkDecorator struct {
    CoffeeDecorator
}

func NewMilkDecorator(coffee Coffee) *MilkDecorator {
    return &MilkDecorator{CoffeeDecorator{coffee: coffee}}
}

func (md *MilkDecorator) GetDescription() string {
    return md.coffee.GetDescription() + ", milk"
}

func (md *MilkDecorator) GetCost() float64 {
    return md.coffee.GetCost() + 0.50
}

type SugarDecorator struct {
    CoffeeDecorator
}

func NewSugarDecorator(coffee Coffee) *SugarDecorator {
    return &SugarDecorator{CoffeeDecorator{coffee: coffee}}
}

func (sd *SugarDecorator) GetDescription() string {
    return sd.coffee.GetDescription() + ", sugar"
}

func (sd *SugarDecorator) GetCost() float64 {
    return sd.coffee.GetCost() + 0.25
}

type VanillaDecorator struct {
    CoffeeDecorator
}

func NewVanillaDecorator(coffee Coffee) *VanillaDecorator {
    return &VanillaDecorator{CoffeeDecorator{coffee: coffee}}
}

func (vd *VanillaDecorator) GetDescription() string {
    return vd.coffee.GetDescription() + ", vanilla"
}

func (vd *VanillaDecorator) GetCost() float64 {
    return vd.coffee.GetCost() + 0.75
}

// Advanced decorator with timing
type TimedCoffeeDecorator struct {
    CoffeeDecorator
    orderTime time.Time
}

func NewTimedCoffeeDecorator(coffee Coffee) *TimedCoffeeDecorator {
    return &TimedCoffeeDecorator{
        CoffeeDecorator: CoffeeDecorator{coffee: coffee},
        orderTime:       time.Now(),
    }
}

func (tcd *TimedCoffeeDecorator) GetDescription() string {
    elapsed := time.Since(tcd.orderTime)
    return fmt.Sprintf("%s (ordered %v ago)", 
                       tcd.coffee.GetDescription(), 
                       elapsed.Round(time.Second))
}

func (tcd *TimedCoffeeDecorator) GetCost() float64 {
    return tcd.coffee.GetCost()
}

// Logging decorator
type LoggingCoffeeDecorator struct {
    CoffeeDecorator
}

func NewLoggingCoffeeDecorator(coffee Coffee) *LoggingCoffeeDecorator {
    return &LoggingCoffeeDecorator{CoffeeDecorator{coffee: coffee}}
}

func (lcd *LoggingCoffeeDecorator) GetDescription() string {
    description := lcd.coffee.GetDescription()
    fmt.Printf("LOG: Getting description for: %s\n", description)
    return description
}

func (lcd *LoggingCoffeeDecorator) GetCost() float64 {
    cost := lcd.coffee.GetCost()
    fmt.Printf("LOG: Calculating cost: $%.2f\n", cost)
    return cost
}

func printCoffeeInfo(coffee Coffee) {
    fmt.Printf("Coffee: %s\n", coffee.GetDescription())
    fmt.Printf("Cost: $%.2f\n", coffee.GetCost())
    fmt.Println(strings.Repeat("-", 40))
}

func main() {
    // Start with simple coffee
    coffee := &SimpleCoffee{}
    printCoffeeInfo(coffee)
    
    // Add milk
    coffeeWithMilk := NewMilkDecorator(coffee)
    printCoffeeInfo(coffeeWithMilk)
    
    // Add sugar to coffee with milk
    coffeeWithMilkAndSugar := NewSugarDecorator(coffeeWithMilk)
    printCoffeeInfo(coffeeWithMilkAndSugar)
    
    // Add vanilla to the mix
    fancyCoffee := NewVanillaDecorator(coffeeWithMilkAndSugar)
    printCoffeeInfo(fancyCoffee)
    
    // Add timing information
    timedCoffee := NewTimedCoffeeDecorator(fancyCoffee)
    time.Sleep(2 * time.Second) // Simulate time passing
    printCoffeeInfo(timedCoffee)
    
    // Add logging
    fmt.Println("Adding logging decorator:")
    loggedCoffee := NewLoggingCoffeeDecorator(timedCoffee)
    printCoffeeInfo(loggedCoffee)
}
```

The decorator pattern allows behavior to be added to objects dynamically  
while keeping the interface consistent. Multiple decorators can be chained  
together to create complex combinations of functionality.  

## Interface for template method pattern

The template method pattern uses interfaces to define the skeleton of an  
algorithm while letting subclasses override specific steps.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Algorithm interface
type DataProcessor interface {
    ProcessData(data []byte) error
    
    // Template method steps
    ValidateData(data []byte) error
    ParseData(data []byte) (any, error)
    TransformData(parsed any) (any, error)
    SaveData(transformed any) error
}

// Base implementation with template method
type BaseDataProcessor struct {
    name string
}

func (bdp *BaseDataProcessor) ProcessData(data []byte) error {
    fmt.Printf("Starting %s data processing...\n", bdp.name)
    
    // Step 1: Validate
    if err := bdp.ValidateData(data); err != nil {
        return fmt.Errorf("validation failed: %v", err)
    }
    fmt.Println("✓ Data validation passed")
    
    // Step 2: Parse  
    parsed, err := bdp.ParseData(data)
    if err != nil {
        return fmt.Errorf("parsing failed: %v", err)
    }
    fmt.Println("✓ Data parsing completed")
    
    // Step 3: Transform
    transformed, err := bdp.TransformData(parsed)
    if err != nil {
        return fmt.Errorf("transformation failed: %v", err)
    }
    fmt.Println("✓ Data transformation completed")
    
    // Step 4: Save
    if err := bdp.SaveData(transformed); err != nil {
        return fmt.Errorf("saving failed: %v", err)
    }
    fmt.Println("✓ Data saving completed")
    
    fmt.Printf("%s processing finished successfully\n", bdp.name)
    return nil
}

// Default implementations (can be overridden)
func (bdp *BaseDataProcessor) ValidateData(data []byte) error {
    if len(data) == 0 {
        return fmt.Errorf("empty data")
    }
    return nil
}

func (bdp *BaseDataProcessor) TransformData(parsed any) (any, error) {
    // Default: no transformation
    return parsed, nil
}

func (bdp *BaseDataProcessor) SaveData(transformed any) error {
    fmt.Printf("Saving data: %v\n", transformed)
    return nil
}

// Concrete implementation for CSV processing
type CSVProcessor struct {
    BaseDataProcessor
    delimiter string
}

func NewCSVProcessor(delimiter string) *CSVProcessor {
    return &CSVProcessor{
        BaseDataProcessor: BaseDataProcessor{name: "CSV"},
        delimiter:         delimiter,
    }
}

func (cp *CSVProcessor) ParseData(data []byte) (any, error) {
    lines := strings.Split(string(data), "\n")
    var records [][]string
    
    for _, line := range lines {
        line = strings.TrimSpace(line)
        if line == "" {
            continue
        }
        fields := strings.Split(line, cp.delimiter)
        records = append(records, fields)
    }
    
    return records, nil
}

func (cp *CSVProcessor) TransformData(parsed any) (any, error) {
    records := parsed.([][]string)
    
    // Transform to map with headers
    if len(records) < 2 {
        return records, nil
    }
    
    headers := records[0]
    var result []map[string]string
    
    for _, record := range records[1:] {
        row := make(map[string]string)
        for i, value := range record {
            if i < len(headers) {
                row[headers[i]] = value
            }
        }
        result = append(result, row)
    }
    
    return result, nil
}

// Concrete implementation for log processing
type LogProcessor struct {
    BaseDataProcessor
    logLevel string
}

func NewLogProcessor(logLevel string) *LogProcessor {
    return &LogProcessor{
        BaseDataProcessor: BaseDataProcessor{name: "Log"},
        logLevel:          logLevel,
    }
}

func (lp *LogProcessor) ValidateData(data []byte) error {
    if err := lp.BaseDataProcessor.ValidateData(data); err != nil {
        return err
    }
    
    // Additional validation for log format
    content := string(data)
    if !strings.Contains(content, "[") || !strings.Contains(content, "]") {
        return fmt.Errorf("invalid log format - missing brackets")
    }
    
    return nil
}

func (lp *LogProcessor) ParseData(data []byte) (any, error) {
    lines := strings.Split(string(data), "\n")
    var logEntries []map[string]string
    
    for _, line := range lines {
        line = strings.TrimSpace(line)
        if line == "" {
            continue
        }
        
        // Simple log parsing: [timestamp] [level] message
        entry := make(map[string]string)
        
        if start := strings.Index(line, "["); start != -1 {
            if end := strings.Index(line[start+1:], "]"); end != -1 {
                entry["timestamp"] = line[start+1 : start+1+end]
                remaining := line[start+1+end+1:]
                
                if start2 := strings.Index(remaining, "["); start2 != -1 {
                    if end2 := strings.Index(remaining[start2+1:], "]"); end2 != -1 {
                        entry["level"] = remaining[start2+1 : start2+1+end2]
                        entry["message"] = strings.TrimSpace(remaining[start2+1+end2+1:])
                    }
                }
            }
        }
        
        logEntries = append(logEntries, entry)
    }
    
    return logEntries, nil
}

func (lp *LogProcessor) TransformData(parsed any) (any, error) {
    entries := parsed.([]map[string]string)
    var filtered []map[string]string
    
    // Filter by log level
    for _, entry := range entries {
        if level, exists := entry["level"]; exists {
            if strings.ToUpper(level) == strings.ToUpper(lp.logLevel) {
                // Add processing timestamp
                entry["processed_at"] = time.Now().Format("2006-01-02 15:04:05")
                filtered = append(filtered, entry)
            }
        }
    }
    
    return filtered, nil
}

func (lp *LogProcessor) SaveData(transformed any) error {
    entries := transformed.([]map[string]string)
    fmt.Printf("Saving %d filtered log entries:\n", len(entries))
    for i, entry := range entries {
        fmt.Printf("  %d. %s [%s] %s\n", 
                   i+1, entry["timestamp"], entry["level"], entry["message"])
    }
    return nil
}

func main() {
    // Test CSV processing
    csvData := "name,age,city\nAlice,25,New York\nBob,30,San Francisco\nCharlie,35,Chicago"
    
    fmt.Println("=== CSV Processing ===")
    csvProcessor := NewCSVProcessor(",")
    err := csvProcessor.ProcessData([]byte(csvData))
    if err != nil {
        fmt.Printf("CSV processing error: %v\n", err)
    }
    
    fmt.Println("\n=== Log Processing ===")
    logData := `[2023-01-01 10:00:00] [INFO] Application started
[2023-01-01 10:00:01] [DEBUG] Loading configuration
[2023-01-01 10:00:02] [ERROR] Database connection failed
[2023-01-01 10:00:03] [INFO] Retrying connection
[2023-01-01 10:00:04] [ERROR] Connection timeout`
    
    logProcessor := NewLogProcessor("ERROR")
    err = logProcessor.ProcessData([]byte(logData))
    if err != nil {
        fmt.Printf("Log processing error: %v\n", err)
    }
}
```

The template method pattern defines the skeleton of an algorithm in an  
interface, with concrete implementations providing specific behavior  
for individual steps. This promotes code reuse while allowing customization.  

## Interface nil checks and safety

Interfaces in Go can be nil, and checking for nil interfaces requires  
understanding the difference between typed and untyped nil values.  

```go
package main

import "fmt"

type Writer interface {
    Write(data string) error
}

type FileWriter struct {
    filename string
    closed   bool
}

func (fw *FileWriter) Write(data string) error {
    if fw.closed {
        return fmt.Errorf("writer is closed")
    }
    fmt.Printf("Writing to %s: %s\n", fw.filename, data)
    return nil
}

func (fw *FileWriter) Close() {
    fw.closed = true
}

func isNilInterface(w Writer) bool {
    // This checks if the interface itself is nil
    return w == nil
}

func isNilValue(w Writer) bool {
    // This checks if the underlying value is nil
    // but the interface type information is still present
    if w == nil {
        return true
    }
    
    // Type assertion to check underlying value
    if fw, ok := w.(*FileWriter); ok {
        return fw == nil
    }
    
    return false
}

func safeWrite(w Writer, data string) error {
    // Always check for nil interface first
    if w == nil {
        return fmt.Errorf("writer is nil")
    }
    
    return w.Write(data)
}

func demonstrateNilInterface() {
    fmt.Println("=== Nil Interface Demonstration ===")
    
    var w Writer
    
    fmt.Printf("Uninitialized interface: w == nil is %t\n", w == nil)
    fmt.Printf("isNilInterface(w): %t\n", isNilInterface(w))
    
    err := safeWrite(w, "test data")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Assign nil pointer to interface
    var fw *FileWriter = nil
    w = fw // w is now typed nil (not untyped nil)
    
    fmt.Printf("\nTyped nil interface: w == nil is %t\n", w == nil)
    fmt.Printf("isNilInterface(w): %t\n", isNilInterface(w))
    fmt.Printf("isNilValue(w): %t\n", isNilValue(w))
    
    err = safeWrite(w, "test data")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Assign actual value
    w = &FileWriter{filename: "output.txt"}
    
    fmt.Printf("\nValid interface: w == nil is %t\n", w == nil)
    fmt.Printf("isNilInterface(w): %t\n", isNilInterface(w))
    
    err = safeWrite(w, "hello there")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}

func main() {
    demonstrateNilInterface()
}
```

Understanding nil interfaces is crucial for safe Go programming. Always  
check for nil before calling interface methods, and be aware of the  
difference between typed and untyped nil values.  

## Interface type switches for multiple types

Type switches provide a clean way to handle multiple types when working  
with interface values.  

```go
package main

import (
    "fmt"
    "reflect"
    "strconv"
    "time"
)

func processValue(value any) string {
    switch v := value.(type) {
    case nil:
        return "null value"
    
    case string:
        if v == "" {
            return "empty string"
        }
        return fmt.Sprintf("string: '%s' (len=%d)", v, len(v))
    
    case int:
        if v == 0 {
            return "zero integer"
        }
        return fmt.Sprintf("integer: %d (hex=0x%x)", v, v)
    
    case int64:
        return fmt.Sprintf("int64: %d", v)
    
    case float64:
        return fmt.Sprintf("float: %.2f", v)
    
    case bool:
        return fmt.Sprintf("boolean: %t", v)
    
    case time.Time:
        return fmt.Sprintf("time: %s", v.Format("2006-01-02 15:04:05"))
    
    case []int:
        sum := 0
        for _, n := range v {
            sum += n
        }
        return fmt.Sprintf("int slice: %v (sum=%d)", v, sum)
    
    case []string:
        total := 0
        for _, s := range v {
            total += len(s)
        }
        return fmt.Sprintf("string slice: %v (total_chars=%d)", v, total)
    
    case map[string]int:
        count := len(v)
        total := 0
        for _, val := range v {
            total += val
        }
        return fmt.Sprintf("string->int map: %v (keys=%d, sum=%d)", 
                           v, count, total)
    
    case fmt.Stringer:
        // This catches any type that implements String() method
        return fmt.Sprintf("stringer: %s (type=%T)", v.String(), v)
    
    case error:
        return fmt.Sprintf("error: %s", v.Error())
    
    default:
        // Use reflection for unknown types
        rv := reflect.ValueOf(v)
        rt := reflect.TypeOf(v)
        
        switch rt.Kind() {
        case reflect.Slice:
            return fmt.Sprintf("unknown slice type %s with %d elements", 
                               rt, rv.Len())
        case reflect.Map:
            return fmt.Sprintf("unknown map type %s with %d keys", 
                               rt, rv.Len())
        case reflect.Struct:
            return fmt.Sprintf("struct of type %s with %d fields", 
                               rt, rv.NumField())
        case reflect.Ptr:
            if rv.IsNil() {
                return fmt.Sprintf("nil pointer of type %s", rt)
            }
            return fmt.Sprintf("pointer to %s", rt.Elem())
        default:
            return fmt.Sprintf("unknown type %T with value %v", v, v)
        }
    }
}

type CustomType struct {
    Name  string
    Value int
}

func (ct CustomType) String() string {
    return fmt.Sprintf("CustomType{Name: %s, Value: %d}", ct.Name, ct.Value)
}

type AnotherType struct {
    Data []byte
}

func main() {
    values := []any{
        nil,
        "",
        "hello there",
        0,
        42,
        int64(1234567890),
        3.14159,
        true,
        false,
        time.Now(),
        []int{1, 2, 3, 4, 5},
        []string{"go", "lang", "interfaces"},
        map[string]int{"a": 1, "b": 2, "c": 3},
        CustomType{Name: "test", Value: 100},
        fmt.Errorf("sample error"),
        []float64{1.1, 2.2, 3.3},
        map[int]string{1: "one", 2: "two"},
        &CustomType{Name: "pointer", Value: 200},
        (*CustomType)(nil),
        AnotherType{Data: []byte("binary data")},
    }
    
    for i, value := range values {
        result := processValue(value)
        fmt.Printf("%2d. %s\n", i+1, result)
    }
}
```

Type switches provide powerful pattern matching for interface values,  
allowing different handling logic for each type. Combined with reflection,  
they can handle virtually any Go type gracefully.  

## Interface for plugin architecture

Interfaces enable plugin architectures where functionality can be extended  
dynamically without modifying the core system.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "time"
)

// Core plugin interface
type Plugin interface {
    GetName() string
    GetVersion() string
    GetDescription() string
    Initialize(config map[string]any) error
    Execute(input any) (any, error)
    Cleanup() error
}

// Specialized plugin interfaces
type TextProcessor interface {
    Plugin
    ProcessText(text string) (string, error)
}

type DataValidator interface {
    Plugin
    ValidateData(data any) (bool, []string)
}

type NotificationSender interface {
    Plugin
    SendNotification(message, recipient string) error
}

// Text processing plugins
type UpperCasePlugin struct {
    name        string
    version     string
    description string
    enabled     bool
}

func (ucp *UpperCasePlugin) GetName() string { return ucp.name }
func (ucp *UpperCasePlugin) GetVersion() string { return ucp.version }
func (ucp *UpperCasePlugin) GetDescription() string { return ucp.description }

func (ucp *UpperCasePlugin) Initialize(config map[string]any) error {
    ucp.name = "UpperCase"
    ucp.version = "1.0.0"
    ucp.description = "Converts text to uppercase"
    ucp.enabled = true
    
    if enabled, exists := config["enabled"]; exists {
        if e, ok := enabled.(bool); ok {
            ucp.enabled = e
        }
    }
    
    fmt.Printf("Initialized %s plugin v%s\n", ucp.name, ucp.version)
    return nil
}

func (ucp *UpperCasePlugin) Execute(input any) (any, error) {
    if !ucp.enabled {
        return input, nil
    }
    
    if text, ok := input.(string); ok {
        return strings.ToUpper(text), nil
    }
    
    return nil, fmt.Errorf("input must be string")
}

func (ucp *UpperCasePlugin) ProcessText(text string) (string, error) {
    result, err := ucp.Execute(text)
    if err != nil {
        return "", err
    }
    return result.(string), nil
}

func (ucp *UpperCasePlugin) Cleanup() error {
    fmt.Printf("Cleaning up %s plugin\n", ucp.name)
    return nil
}

type WordCountPlugin struct {
    name        string
    version     string
    description string
    minLength   int
}

func (wcp *WordCountPlugin) GetName() string { return wcp.name }
func (wcp *WordCountPlugin) GetVersion() string { return wcp.version }
func (wcp *WordCountPlugin) GetDescription() string { return wcp.description }

func (wcp *WordCountPlugin) Initialize(config map[string]any) error {
    wcp.name = "WordCount"
    wcp.version = "1.0.0"
    wcp.description = "Counts words in text"
    wcp.minLength = 1
    
    if minLen, exists := config["min_length"]; exists {
        if ml, ok := minLen.(int); ok {
            wcp.minLength = ml
        }
    }
    
    fmt.Printf("Initialized %s plugin v%s (min_length=%d)\n", 
               wcp.name, wcp.version, wcp.minLength)
    return nil
}

func (wcp *WordCountPlugin) Execute(input any) (any, error) {
    if text, ok := input.(string); ok {
        words := strings.Fields(text)
        validWords := 0
        
        for _, word := range words {
            // Clean word (remove punctuation)
            cleaned := regexp.MustCompile(`[^a-zA-Z0-9]`).ReplaceAllString(word, "")
            if len(cleaned) >= wcp.minLength {
                validWords++
            }
        }
        
        return map[string]int{
            "total_words": len(words),
            "valid_words": validWords,
            "min_length":  wcp.minLength,
        }, nil
    }
    
    return nil, fmt.Errorf("input must be string")
}

func (wcp *WordCountPlugin) ProcessText(text string) (string, error) {
    result, err := wcp.Execute(text)
    if err != nil {
        return "", err
    }
    
    counts := result.(map[string]int)
    return fmt.Sprintf("Total: %d, Valid: %d (min %d chars)", 
                       counts["total_words"], 
                       counts["valid_words"], 
                       counts["min_length"]), nil
}

func (wcp *WordCountPlugin) Cleanup() error {
    fmt.Printf("Cleaning up %s plugin\n", wcp.name)
    return nil
}

// Email validation plugin
type EmailValidatorPlugin struct {
    name        string
    version     string
    description string
    emailRegex  *regexp.Regexp
}

func (evp *EmailValidatorPlugin) GetName() string { return evp.name }
func (evp *EmailValidatorPlugin) GetVersion() string { return evp.version }
func (evp *EmailValidatorPlugin) GetDescription() string { return evp.description }

func (evp *EmailValidatorPlugin) Initialize(config map[string]any) error {
    evp.name = "EmailValidator"
    evp.version = "1.0.0"
    evp.description = "Validates email addresses"
    
    // Simple email regex
    evp.emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    
    fmt.Printf("Initialized %s plugin v%s\n", evp.name, evp.version)
    return nil
}

func (evp *EmailValidatorPlugin) Execute(input any) (any, error) {
    return evp.ValidateData(input)
}

func (evp *EmailValidatorPlugin) ValidateData(data any) (bool, []string) {
    var errors []string
    
    if email, ok := data.(string); ok {
        if !evp.emailRegex.MatchString(email) {
            errors = append(errors, "invalid email format")
        }
        
        if len(email) > 254 {
            errors = append(errors, "email too long (max 254 characters)")
        }
        
        return len(errors) == 0, errors
    }
    
    errors = append(errors, "input must be a string")
    return false, errors
}

func (evp *EmailValidatorPlugin) Cleanup() error {
    fmt.Printf("Cleaning up %s plugin\n", evp.name)
    return nil
}

// Plugin manager
type PluginManager struct {
    plugins map[string]Plugin
}

func NewPluginManager() *PluginManager {
    return &PluginManager{
        plugins: make(map[string]Plugin),
    }
}

func (pm *PluginManager) RegisterPlugin(plugin Plugin, config map[string]any) error {
    err := plugin.Initialize(config)
    if err != nil {
        return fmt.Errorf("failed to initialize plugin: %v", err)
    }
    
    pm.plugins[plugin.GetName()] = plugin
    return nil
}

func (pm *PluginManager) GetPlugin(name string) (Plugin, bool) {
    plugin, exists := pm.plugins[name]
    return plugin, exists
}

func (pm *PluginManager) ListPlugins() []Plugin {
    var plugins []Plugin
    for _, plugin := range pm.plugins {
        plugins = append(plugins, plugin)
    }
    return plugins
}

func (pm *PluginManager) ExecutePlugin(name string, input any) (any, error) {
    plugin, exists := pm.plugins[name]
    if !exists {
        return nil, fmt.Errorf("plugin not found: %s", name)
    }
    
    return plugin.Execute(input)
}

func (pm *PluginManager) Shutdown() {
    for _, plugin := range pm.plugins {
        plugin.Cleanup()
    }
}

func main() {
    manager := NewPluginManager()
    
    // Register plugins
    err := manager.RegisterPlugin(&UpperCasePlugin{}, map[string]any{
        "enabled": true,
    })
    if err != nil {
        fmt.Printf("Failed to register plugin: %v\n", err)
    }
    
    err = manager.RegisterPlugin(&WordCountPlugin{}, map[string]any{
        "min_length": 3,
    })
    if err != nil {
        fmt.Printf("Failed to register plugin: %v\n", err)
    }
    
    err = manager.RegisterPlugin(&EmailValidatorPlugin{}, map[string]any{})
    if err != nil {
        fmt.Printf("Failed to register plugin: %v\n", err)
    }
    
    fmt.Println("\n=== Plugin Execution ===")
    
    // Test text processing plugins
    testText := "hello there, this is a test message!"
    
    result, err := manager.ExecutePlugin("UpperCase", testText)
    if err != nil {
        fmt.Printf("UpperCase error: %v\n", err)
    } else {
        fmt.Printf("UpperCase result: %s\n", result)
    }
    
    result, err = manager.ExecutePlugin("WordCount", testText)
    if err != nil {
        fmt.Printf("WordCount error: %v\n", err)
    } else {
        fmt.Printf("WordCount result: %v\n", result)
    }
    
    // Test validation plugin
    emails := []string{
        "valid@example.com",
        "invalid.email",
        "another@test.org",
    }
    
    for _, email := range emails {
        if plugin, exists := manager.GetPlugin("EmailValidator"); exists {
            if validator, ok := plugin.(DataValidator); ok {
                valid, errors := validator.ValidateData(email)
                fmt.Printf("Email '%s': valid=%t", email, valid)
                if len(errors) > 0 {
                    fmt.Printf(", errors=%v", errors)
                }
                fmt.Println()
            }
        }
    }
    
    fmt.Println("\n=== Registered Plugins ===")
    for _, plugin := range manager.ListPlugins() {
        fmt.Printf("- %s v%s: %s\n", 
                   plugin.GetName(), 
                   plugin.GetVersion(), 
                   plugin.GetDescription())
    }
    
    manager.Shutdown()
}
```

Plugin architectures with interfaces provide powerful extensibility  
mechanisms. Plugins can implement multiple specialized interfaces,  
and the system can discover and use their capabilities dynamically.  

## Interface for middleware chain

Middleware patterns use interfaces to create chainable request/response  
processors that can be composed together.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Core interfaces
type Request struct {
    ID      string
    Method  string
    Path    string
    Headers map[string]string
    Body    string
}

type Response struct {
    Status  int
    Headers map[string]string
    Body    string
}

type Handler interface {
    Handle(req *Request) (*Response, error)
}

type Middleware interface {
    Process(req *Request, next Handler) (*Response, error)
}

// Handler function type that implements Handler interface
type HandlerFunc func(req *Request) (*Response, error)

func (hf HandlerFunc) Handle(req *Request) (*Response, error) {
    return hf(req)
}

// Middleware implementations
type LoggingMiddleware struct {
    name string
}

func (lm *LoggingMiddleware) Process(req *Request, next Handler) (*Response, error) {
    start := time.Now()
    
    fmt.Printf("[%s] -> %s %s (ID: %s)\n", 
               lm.name, req.Method, req.Path, req.ID)
    
    resp, err := next.Handle(req)
    
    duration := time.Since(start)
    status := 0
    if resp != nil {
        status = resp.Status
    }
    
    fmt.Printf("[%s] <- %s %s %d (%v)\n", 
               lm.name, req.Method, req.Path, status, duration)
    
    return resp, err
}

type AuthenticationMiddleware struct {
    validTokens map[string]string
}

func NewAuthenticationMiddleware() *AuthenticationMiddleware {
    return &AuthenticationMiddleware{
        validTokens: map[string]string{
            "token123": "user1",
            "token456": "user2",
            "admin789": "admin",
        },
    }
}

func (am *AuthenticationMiddleware) Process(req *Request, next Handler) (*Response, error) {
    authHeader, exists := req.Headers["Authorization"]
    if !exists {
        return &Response{
            Status: 401,
            Headers: make(map[string]string),
            Body:   "Missing Authorization header",
        }, nil
    }
    
    // Extract token (assuming "Bearer <token>" format)
    parts := strings.Split(authHeader, " ")
    if len(parts) != 2 || parts[0] != "Bearer" {
        return &Response{
            Status: 401,
            Headers: make(map[string]string),
            Body:   "Invalid Authorization format",
        }, nil
    }
    
    token := parts[1]
    user, valid := am.validTokens[token]
    if !valid {
        return &Response{
            Status: 401,
            Headers: make(map[string]string),
            Body:   "Invalid token",
        }, nil
    }
    
    // Add user to request headers
    if req.Headers == nil {
        req.Headers = make(map[string]string)
    }
    req.Headers["X-User"] = user
    
    return next.Handle(req)
}

type RateLimitMiddleware struct {
    requests map[string][]time.Time
    limit    int
    window   time.Duration
}

func NewRateLimitMiddleware(limit int, window time.Duration) *RateLimitMiddleware {
    return &RateLimitMiddleware{
        requests: make(map[string][]time.Time),
        limit:    limit,
        window:   window,
    }
}

func (rlm *RateLimitMiddleware) Process(req *Request, next Handler) (*Response, error) {
    clientID := req.Headers["X-User"]
    if clientID == "" {
        clientID = "anonymous"
    }
    
    now := time.Now()
    
    // Clean old requests
    if requests, exists := rlm.requests[clientID]; exists {
        var validRequests []time.Time
        for _, reqTime := range requests {
            if now.Sub(reqTime) < rlm.window {
                validRequests = append(validRequests, reqTime)
            }
        }
        rlm.requests[clientID] = validRequests
    }
    
    // Check rate limit
    if len(rlm.requests[clientID]) >= rlm.limit {
        return &Response{
            Status: 429,
            Headers: map[string]string{
                "X-RateLimit-Limit": fmt.Sprintf("%d", rlm.limit),
                "X-RateLimit-Window": rlm.window.String(),
            },
            Body: "Rate limit exceeded",
        }, nil
    }
    
    // Add current request
    rlm.requests[clientID] = append(rlm.requests[clientID], now)
    
    return next.Handle(req)
}

type ValidationMiddleware struct {
    requiredHeaders []string
}

func NewValidationMiddleware(headers []string) *ValidationMiddleware {
    return &ValidationMiddleware{
        requiredHeaders: headers,
    }
}

func (vm *ValidationMiddleware) Process(req *Request, next Handler) (*Response, error) {
    var missingHeaders []string
    
    for _, header := range vm.requiredHeaders {
        if _, exists := req.Headers[header]; !exists {
            missingHeaders = append(missingHeaders, header)
        }
    }
    
    if len(missingHeaders) > 0 {
        return &Response{
            Status: 400,
            Headers: make(map[string]string),
            Body:   fmt.Sprintf("Missing required headers: %v", missingHeaders),
        }, nil
    }
    
    return next.Handle(req)
}

// Middleware chain
type MiddlewareChain struct {
    middlewares []Middleware
    handler     Handler
}

func NewMiddlewareChain(handler Handler) *MiddlewareChain {
    return &MiddlewareChain{
        handler: handler,
    }
}

func (mc *MiddlewareChain) Use(middleware Middleware) *MiddlewareChain {
    mc.middlewares = append(mc.middlewares, middleware)
    return mc
}

func (mc *MiddlewareChain) Handle(req *Request) (*Response, error) {
    if len(mc.middlewares) == 0 {
        return mc.handler.Handle(req)
    }
    
    // Create a handler that wraps the next middleware or final handler
    var buildHandler func(int) Handler
    buildHandler = func(index int) Handler {
        if index >= len(mc.middlewares) {
            return mc.handler
        }
        
        middleware := mc.middlewares[index]
        nextHandler := buildHandler(index + 1)
        
        return HandlerFunc(func(req *Request) (*Response, error) {
            return middleware.Process(req, nextHandler)
        })
    }
    
    return buildHandler(0).Handle(req)
}

// Sample handlers
func helloHandler(req *Request) (*Response, error) {
    user := req.Headers["X-User"]
    if user == "" {
        user = "anonymous"
    }
    
    return &Response{
        Status:  200,
        Headers: map[string]string{"Content-Type": "text/plain"},
        Body:    fmt.Sprintf("Hello there, %s!", user),
    }, nil
}

func protectedHandler(req *Request) (*Response, error) {
    user := req.Headers["X-User"]
    
    return &Response{
        Status:  200,
        Headers: map[string]string{"Content-Type": "application/json"},
        Body:    fmt.Sprintf(`{"message": "Secret data", "user": "%s"}`, user),
    }, nil
}

func main() {
    // Create middleware chain for public endpoint
    publicChain := NewMiddlewareChain(HandlerFunc(helloHandler))
    publicChain.Use(&LoggingMiddleware{name: "PUBLIC"})
    publicChain.Use(NewRateLimitMiddleware(3, time.Minute))
    
    // Create middleware chain for protected endpoint
    protectedChain := NewMiddlewareChain(HandlerFunc(protectedHandler))
    protectedChain.Use(&LoggingMiddleware{name: "PROTECTED"})
    protectedChain.Use(NewAuthenticationMiddleware())
    protectedChain.Use(NewValidationMiddleware([]string{"X-API-Version"}))
    protectedChain.Use(NewRateLimitMiddleware(5, time.Minute))
    
    // Test requests
    requests := []*Request{
        {
            ID:     "req1",
            Method: "GET",
            Path:   "/hello",
            Headers: map[string]string{},
        },
        {
            ID:     "req2",
            Method: "GET",
            Path:   "/protected",
            Headers: map[string]string{
                "Authorization": "Bearer token123",
                "X-API-Version": "1.0",
            },
        },
        {
            ID:     "req3",
            Method: "GET",
            Path:   "/protected",
            Headers: map[string]string{
                "Authorization": "Bearer invalid",
                "X-API-Version": "1.0",
            },
        },
        {
            ID:     "req4",
            Method: "GET",
            Path:   "/protected",
            Headers: map[string]string{
                "Authorization": "Bearer admin789",
            },
        },
    }
    
    fmt.Println("=== Testing Public Endpoint ===")
    resp, err := publicChain.Handle(requests[0])
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Response: %d - %s\n", resp.Status, resp.Body)
    }
    
    fmt.Println("\n=== Testing Protected Endpoint ===")
    for i := 1; i < len(requests); i++ {
        fmt.Printf("\n--- Request %d ---\n", i)
        resp, err := protectedChain.Handle(requests[i])
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Printf("Response: %d - %s\n", resp.Status, resp.Body)
        }
    }
    
    fmt.Println("\n=== Rate Limiting Test ===")
    for i := 0; i < 5; i++ {
        resp, err := publicChain.Handle(requests[0])
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Printf("Attempt %d: %d - %s\n", i+1, resp.Status, resp.Body)
        }
    }
}
```

Middleware chains provide a powerful pattern for request/response processing.  
Each middleware can perform pre-processing, post-processing, or both,  
allowing complex behaviors to be composed from simple components.  

## Interface for event sourcing

Event sourcing uses interfaces to define events and handlers that can  
reconstruct application state from a sequence of events.  

```go
package main

import (
    "fmt"
    "time"
)

// Core event sourcing interfaces
type Event interface {
    GetEventType() string
    GetTimestamp() time.Time
    GetAggregateID() string
    GetEventData() map[string]any
}

type EventHandler interface {
    CanHandle(eventType string) bool
    Handle(event Event) error
}

type EventStore interface {
    SaveEvent(event Event) error
    GetEvents(aggregateID string) ([]Event, error)
    GetEventsByType(eventType string) ([]Event, error)
}

type Aggregate interface {
    GetID() string
    ApplyEvent(event Event) error
    GetUncommittedEvents() []Event
    MarkEventsAsCommitted()
}

// Base event implementation
type BaseEvent struct {
    EventType   string            `json:"event_type"`
    Timestamp   time.Time         `json:"timestamp"`
    AggregateID string            `json:"aggregate_id"`
    EventData   map[string]any    `json:"event_data"`
}

func (be BaseEvent) GetEventType() string {
    return be.EventType
}

func (be BaseEvent) GetTimestamp() time.Time {
    return be.Timestamp
}

func (be BaseEvent) GetAggregateID() string {
    return be.AggregateID
}

func (be BaseEvent) GetEventData() map[string]any {
    return be.EventData
}

// Specific events
type UserCreatedEvent struct {
    BaseEvent
}

func NewUserCreatedEvent(userID, name, email string) *UserCreatedEvent {
    return &UserCreatedEvent{
        BaseEvent: BaseEvent{
            EventType:   "UserCreated",
            Timestamp:   time.Now(),
            AggregateID: userID,
            EventData: map[string]any{
                "name":  name,
                "email": email,
            },
        },
    }
}

type UserEmailChangedEvent struct {
    BaseEvent
}

func NewUserEmailChangedEvent(userID, oldEmail, newEmail string) *UserEmailChangedEvent {
    return &UserEmailChangedEvent{
        BaseEvent: BaseEvent{
            EventType:   "UserEmailChanged",
            Timestamp:   time.Now(),
            AggregateID: userID,
            EventData: map[string]any{
                "old_email": oldEmail,
                "new_email": newEmail,
            },
        },
    }
}

type UserDeactivatedEvent struct {
    BaseEvent
}

func NewUserDeactivatedEvent(userID, reason string) *UserDeactivatedEvent {
    return &UserDeactivatedEvent{
        BaseEvent: BaseEvent{
            EventType:   "UserDeactivated",
            Timestamp:   time.Now(),
            AggregateID: userID,
            EventData: map[string]any{
                "reason": reason,
            },
        },
    }
}

// User aggregate
type User struct {
    ID               string
    Name             string
    Email            string
    IsActive         bool
    CreatedAt        time.Time
    LastModified     time.Time
    uncommittedEvents []Event
}

func NewUser() *User {
    return &User{
        IsActive:         true,
        uncommittedEvents: make([]Event, 0),
    }
}

func (u *User) GetID() string {
    return u.ID
}

func (u *User) CreateUser(id, name, email string) {
    event := NewUserCreatedEvent(id, name, email)
    u.ApplyEvent(event)
    u.uncommittedEvents = append(u.uncommittedEvents, event)
}

func (u *User) ChangeEmail(newEmail string) error {
    if !u.IsActive {
        return fmt.Errorf("cannot change email for inactive user")
    }
    
    if u.Email == newEmail {
        return fmt.Errorf("new email is the same as current email")
    }
    
    event := NewUserEmailChangedEvent(u.ID, u.Email, newEmail)
    u.ApplyEvent(event)
    u.uncommittedEvents = append(u.uncommittedEvents, event)
    return nil
}

func (u *User) DeactivateUser(reason string) error {
    if !u.IsActive {
        return fmt.Errorf("user is already inactive")
    }
    
    event := NewUserDeactivatedEvent(u.ID, reason)
    u.ApplyEvent(event)
    u.uncommittedEvents = append(u.uncommittedEvents, event)
    return nil
}

func (u *User) ApplyEvent(event Event) error {
    switch event.GetEventType() {
    case "UserCreated":
        data := event.GetEventData()
        u.ID = event.GetAggregateID()
        u.Name = data["name"].(string)
        u.Email = data["email"].(string)
        u.IsActive = true
        u.CreatedAt = event.GetTimestamp()
        u.LastModified = event.GetTimestamp()
        
    case "UserEmailChanged":
        data := event.GetEventData()
        u.Email = data["new_email"].(string)
        u.LastModified = event.GetTimestamp()
        
    case "UserDeactivated":
        u.IsActive = false
        u.LastModified = event.GetTimestamp()
        
    default:
        return fmt.Errorf("unknown event type: %s", event.GetEventType())
    }
    
    return nil
}

func (u *User) GetUncommittedEvents() []Event {
    return u.uncommittedEvents
}

func (u *User) MarkEventsAsCommitted() {
    u.uncommittedEvents = make([]Event, 0)
}

// In-memory event store
type InMemoryEventStore struct {
    events map[string][]Event // aggregateID -> events
}

func NewInMemoryEventStore() *InMemoryEventStore {
    return &InMemoryEventStore{
        events: make(map[string][]Event),
    }
}

func (es *InMemoryEventStore) SaveEvent(event Event) error {
    aggregateID := event.GetAggregateID()
    es.events[aggregateID] = append(es.events[aggregateID], event)
    fmt.Printf("Saved event: %s for aggregate %s\n", 
               event.GetEventType(), aggregateID)
    return nil
}

func (es *InMemoryEventStore) GetEvents(aggregateID string) ([]Event, error) {
    events, exists := es.events[aggregateID]
    if !exists {
        return []Event{}, nil
    }
    return events, nil
}

func (es *InMemoryEventStore) GetEventsByType(eventType string) ([]Event, error) {
    var matchingEvents []Event
    
    for _, aggregateEvents := range es.events {
        for _, event := range aggregateEvents {
            if event.GetEventType() == eventType {
                matchingEvents = append(matchingEvents, event)
            }
        }
    }
    
    return matchingEvents, nil
}

// Event handlers
type UserEventHandler struct {
    name string
}

func (ueh *UserEventHandler) CanHandle(eventType string) bool {
    userEvents := []string{"UserCreated", "UserEmailChanged", "UserDeactivated"}
    for _, et := range userEvents {
        if et == eventType {
            return true
        }
    }
    return false
}

func (ueh *UserEventHandler) Handle(event Event) error {
    fmt.Printf("[%s] Handling event: %s for user %s\n", 
               ueh.name, event.GetEventType(), event.GetAggregateID())
    
    switch event.GetEventType() {
    case "UserCreated":
        data := event.GetEventData()
        fmt.Printf("  New user created: %s (%s)\n", 
                   data["name"], data["email"])
        
    case "UserEmailChanged":
        data := event.GetEventData()
        fmt.Printf("  Email changed from %s to %s\n", 
                   data["old_email"], data["new_email"])
        
    case "UserDeactivated":
        data := event.GetEventData()
        fmt.Printf("  User deactivated. Reason: %s\n", 
                   data["reason"])
    }
    
    return nil
}

type EmailNotificationHandler struct{}

func (enh *EmailNotificationHandler) CanHandle(eventType string) bool {
    return eventType == "UserEmailChanged"
}

func (enh *EmailNotificationHandler) Handle(event Event) error {
    if event.GetEventType() == "UserEmailChanged" {
        data := event.GetEventData()
        fmt.Printf("[EMAIL] Sending notification about email change to %s\n", 
                   data["new_email"])
    }
    return nil
}

// Repository for reconstructing aggregates
type UserRepository struct {
    eventStore EventStore
}

func NewUserRepository(eventStore EventStore) *UserRepository {
    return &UserRepository{eventStore: eventStore}
}

func (ur *UserRepository) GetByID(id string) (*User, error) {
    events, err := ur.eventStore.GetEvents(id)
    if err != nil {
        return nil, err
    }
    
    if len(events) == 0 {
        return nil, fmt.Errorf("user not found: %s", id)
    }
    
    user := NewUser()
    for _, event := range events {
        err := user.ApplyEvent(event)
        if err != nil {
            return nil, fmt.Errorf("failed to apply event: %v", err)
        }
    }
    
    return user, nil
}

func (ur *UserRepository) Save(user *User) error {
    for _, event := range user.GetUncommittedEvents() {
        err := ur.eventStore.SaveEvent(event)
        if err != nil {
            return err
        }
    }
    
    user.MarkEventsAsCommitted()
    return nil
}

func main() {
    // Setup
    eventStore := NewInMemoryEventStore()
    repository := NewUserRepository(eventStore)
    
    handlers := []EventHandler{
        &UserEventHandler{name: "UserHandler"},
        &EmailNotificationHandler{},
    }
    
    // Create and save user
    user := NewUser()
    user.CreateUser("user1", "Alice Johnson", "alice@example.com")
    
    fmt.Println("=== Saving New User ===")
    err := repository.Save(user)
    if err != nil {
        fmt.Printf("Error saving user: %v\n", err)
        return
    }
    
    // Process events
    fmt.Println("\n=== Processing Events ===")
    for _, event := range eventStore.events["user1"] {
        for _, handler := range handlers {
            if handler.CanHandle(event.GetEventType()) {
                handler.Handle(event)
            }
        }
    }
    
    // Load user and make changes
    fmt.Println("\n=== Loading and Updating User ===")
    loadedUser, err := repository.GetByID("user1")
    if err != nil {
        fmt.Printf("Error loading user: %v\n", err)
        return
    }
    
    fmt.Printf("Loaded user: %s (%s) - Active: %t\n", 
               loadedUser.Name, loadedUser.Email, loadedUser.IsActive)
    
    // Change email
    err = loadedUser.ChangeEmail("alice.johnson@newdomain.com")
    if err != nil {
        fmt.Printf("Error changing email: %v\n", err)
        return
    }
    
    // Deactivate user
    err = loadedUser.DeactivateUser("Account migration")
    if err != nil {
        fmt.Printf("Error deactivating user: %v\n", err)
        return
    }
    
    // Save changes
    err = repository.Save(loadedUser)
    if err != nil {
        fmt.Printf("Error saving changes: %v\n", err)
        return
    }
    
    // Process new events
    fmt.Println("\n=== Processing New Events ===")
    allEvents, _ := eventStore.GetEvents("user1")
    newEvents := allEvents[1:] // Skip the first event (already processed)
    
    for _, event := range newEvents {
        for _, handler := range handlers {
            if handler.CanHandle(event.GetEventType()) {
                handler.Handle(event)
            }
        }
    }
    
    // Final state
    fmt.Println("\n=== Final User State ===")
    finalUser, _ := repository.GetByID("user1")
    fmt.Printf("Final user: %s (%s) - Active: %t\n", 
               finalUser.Name, finalUser.Email, finalUser.IsActive)
    
    fmt.Printf("Last modified: %s\n", 
               finalUser.LastModified.Format("2006-01-02 15:04:05"))
}
```

Event sourcing provides a powerful pattern for building systems that need  
complete audit trails and the ability to reconstruct state. Interfaces  
enable clean separation between events, handlers, and storage mechanisms.  

## Interface for circuit breaker pattern

The circuit breaker pattern uses interfaces to prevent cascading failures  
by monitoring service health and failing fast when services are down.  

```go
package main

import (
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

// Core interfaces
type Service interface {
    Call(request string) (string, error)
    GetName() string
}

type CircuitBreaker interface {
    Execute(fn func() (any, error)) (any, error)
    GetState() State
    GetStats() Stats
    Reset()
}

type Stats struct {
    State           State
    SuccessCount    int64
    FailureCount    int64
    TimeoutCount    int64
    ConsecutiveFails int64
    LastFailTime    time.Time
    LastSuccessTime time.Time
}

// Circuit breaker configuration
type Config struct {
    MaxFailures     int64
    Timeout         time.Duration
    ResetTimeout    time.Duration
    HalfOpenMaxCalls int64
}

// Circuit breaker implementation
type DefaultCircuitBreaker struct {
    config    Config
    state     State
    stats     Stats
    mutex     sync.RWMutex
    nextAttempt time.Time
}

func NewCircuitBreaker(config Config) *DefaultCircuitBreaker {
    return &DefaultCircuitBreaker{
        config: config,
        state:  Closed,
        stats:  Stats{State: Closed},
    }
}

func (cb *DefaultCircuitBreaker) Execute(fn func() (any, error)) (any, error) {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    
    now := time.Now()
    
    switch cb.state {
    case Open:
        if now.After(cb.nextAttempt) {
            cb.state = HalfOpen
            cb.stats.State = HalfOpen
            fmt.Printf("Circuit breaker transitioning to HALF_OPEN\n")
        } else {
            return nil, fmt.Errorf("circuit breaker is OPEN")
        }
        
    case HalfOpen:
        // Allow limited calls in half-open state
        if cb.stats.SuccessCount >= cb.config.HalfOpenMaxCalls {
            cb.state = Closed
            cb.stats.State = Closed
            cb.stats.ConsecutiveFails = 0
            fmt.Printf("Circuit breaker transitioning to CLOSED\n")
        }
    }
    
    // Execute the function
    start := time.Now()
    result, err := cb.executeWithTimeout(fn)
    duration := time.Since(start)
    
    if err != nil {
        cb.onFailure(now, duration)
    } else {
        cb.onSuccess(now)
    }
    
    return result, err
}

func (cb *DefaultCircuitBreaker) executeWithTimeout(fn func() (any, error)) (any, error) {
    if cb.config.Timeout <= 0 {
        return fn()
    }
    
    resultChan := make(chan struct {
        result any
        err    error
    }, 1)
    
    go func() {
        result, err := fn()
        resultChan <- struct {
            result any
            err    error
        }{result, err}
    }()
    
    select {
    case res := <-resultChan:
        return res.result, res.err
    case <-time.After(cb.config.Timeout):
        cb.stats.TimeoutCount++
        return nil, fmt.Errorf("operation timed out after %v", cb.config.Timeout)
    }
}

func (cb *DefaultCircuitBreaker) onFailure(now time.Time, duration time.Duration) {
    cb.stats.FailureCount++
    cb.stats.ConsecutiveFails++
    cb.stats.LastFailTime = now
    
    fmt.Printf("Circuit breaker failure (consecutive: %d, duration: %v)\n", 
               cb.stats.ConsecutiveFails, duration)
    
    if cb.state == Closed && cb.stats.ConsecutiveFails >= cb.config.MaxFailures {
        cb.state = Open
        cb.stats.State = Open
        cb.nextAttempt = now.Add(cb.config.ResetTimeout)
        fmt.Printf("Circuit breaker transitioning to OPEN (reset at %s)\n", 
                   cb.nextAttempt.Format("15:04:05"))
    } else if cb.state == HalfOpen {
        // Go back to open state
        cb.state = Open
        cb.stats.State = Open
        cb.nextAttempt = now.Add(cb.config.ResetTimeout)
        fmt.Printf("Circuit breaker back to OPEN from HALF_OPEN\n")
    }
}

func (cb *DefaultCircuitBreaker) onSuccess(now time.Time) {
    cb.stats.SuccessCount++
    cb.stats.ConsecutiveFails = 0
    cb.stats.LastSuccessTime = now
    
    if cb.state == HalfOpen {
        fmt.Printf("Circuit breaker success in HALF_OPEN state\n")
    }
}

func (cb *DefaultCircuitBreaker) GetState() State {
    cb.mutex.RLock()
    defer cb.mutex.RUnlock()
    return cb.state
}

func (cb *DefaultCircuitBreaker) GetStats() Stats {
    cb.mutex.RLock()
    defer cb.mutex.RUnlock()
    return cb.stats
}

func (cb *DefaultCircuitBreaker) Reset() {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    cb.state = Closed
    cb.stats = Stats{State: Closed}
    fmt.Printf("Circuit breaker manually reset\n")
}

// Protected service wrapper
type ProtectedService struct {
    service Service
    breaker CircuitBreaker
}

func NewProtectedService(service Service, breaker CircuitBreaker) *ProtectedService {
    return &ProtectedService{
        service: service,
        breaker: breaker,
    }
}

func (ps *ProtectedService) Call(request string) (string, error) {
    result, err := ps.breaker.Execute(func() (any, error) {
        return ps.service.Call(request)
    })
    
    if err != nil {
        return "", err
    }
    
    return result.(string), nil
}

func (ps *ProtectedService) GetName() string {
    return ps.service.GetName()
}

func (ps *ProtectedService) GetBreakerStats() Stats {
    return ps.breaker.GetStats()
}

// Mock services for testing
type ReliableService struct {
    name string
}

func (rs *ReliableService) Call(request string) (string, error) {
    // Simulate some processing time
    time.Sleep(50 * time.Millisecond)
    return fmt.Sprintf("Reliable response to: %s", request), nil
}

func (rs *ReliableService) GetName() string {
    return rs.name
}

type UnreliableService struct {
    name        string
    failureRate float64 // 0.0 to 1.0
    callCount   int
}

func (us *UnreliableService) Call(request string) (string, error) {
    us.callCount++
    
    // Simulate random failures
    failureProbability := us.failureRate
    
    // Increase failure rate as call count grows (simulating degrading service)
    if us.callCount > 10 {
        failureProbability = 0.8
    }
    
    if float64(us.callCount%10) < failureProbability*10 {
        time.Sleep(100 * time.Millisecond) // Simulate slow failure
        return "", fmt.Errorf("service unavailable (call %d)", us.callCount)
    }
    
    time.Sleep(50 * time.Millisecond) // Normal processing time
    return fmt.Sprintf("Unreliable response to: %s (call %d)", request, us.callCount), nil
}

func (us *UnreliableService) GetName() string {
    return us.name
}

type SlowService struct {
    name  string
    delay time.Duration
}

func (ss *SlowService) Call(request string) (string, error) {
    time.Sleep(ss.delay)
    return fmt.Sprintf("Slow response to: %s", request), nil
}

func (ss *SlowService) GetName() string {
    return ss.name
}

func testService(service *ProtectedService, requests []string) {
    fmt.Printf("\n=== Testing %s ===\n", service.GetName())
    
    for i, request := range requests {
        fmt.Printf("Request %d: %s\n", i+1, request)
        
        response, err := service.Call(request)
        stats := service.GetBreakerStats()
        
        if err != nil {
            fmt.Printf("  Error: %v\n", err)
        } else {
            fmt.Printf("  Response: %s\n", response)
        }
        
        fmt.Printf("  Breaker State: %s (Success: %d, Failures: %d, Consecutive Fails: %d)\n", 
                   stats.State, stats.SuccessCount, stats.FailureCount, stats.ConsecutiveFails)
        
        time.Sleep(200 * time.Millisecond)
    }
}

func main() {
    // Configure circuit breaker
    config := Config{
        MaxFailures:      3,
        Timeout:          500 * time.Millisecond,
        ResetTimeout:     2 * time.Second,
        HalfOpenMaxCalls: 2,
    }
    
    // Create services
    services := []*ProtectedService{
        NewProtectedService(
            &ReliableService{name: "ReliableService"},
            NewCircuitBreaker(config),
        ),
        NewProtectedService(
            &UnreliableService{name: "UnreliableService", failureRate: 0.5},
            NewCircuitBreaker(config),
        ),
        NewProtectedService(
            &SlowService{name: "SlowService", delay: 1 * time.Second},
            NewCircuitBreaker(config),
        ),
    }
    
    requests := []string{
        "hello there",
        "test request 1",
        "test request 2",
        "test request 3",
        "test request 4",
        "test request 5",
        "recovery test 1",
        "recovery test 2",
    }
    
    // Test each service
    for _, service := range services {
        testService(service, requests)
        
        // Wait for potential recovery
        if service.GetBreakerStats().State == Open {
            fmt.Printf("\nWaiting for circuit breaker reset...\n")
            time.Sleep(3 * time.Second)
            
            // Try one more request to test recovery
            fmt.Printf("Testing recovery:\n")
            response, err := service.Call("recovery test")
            if err != nil {
                fmt.Printf("  Recovery failed: %v\n", err)
            } else {
                fmt.Printf("  Recovery successful: %s\n", response)
            }
        }
    }
    
    fmt.Printf("\n=== Final Stats ===\n")
    for _, service := range services {
        stats := service.GetBreakerStats()
        fmt.Printf("%s: State=%s, Success=%d, Failures=%d, Timeouts=%d\n",
                   service.GetName(), stats.State, 
                   stats.SuccessCount, stats.FailureCount, stats.TimeoutCount)
    }
}
```

The circuit breaker pattern prevents cascading failures by monitoring  
service health and failing fast when services are unavailable. It provides  
automatic recovery mechanisms and protects downstream systems.  

## Interface for repository pattern

The repository pattern uses interfaces to abstract data access logic,  
enabling testability and separation of concerns.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Domain entity
type User struct {
    ID        string    `json:"id"`
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    Age       int       `json:"age"`
    CreatedAt time.Time `json:"created_at"`
    UpdatedAt time.Time `json:"updated_at"`
}

// Repository interfaces
type UserRepository interface {
    Create(user *User) error
    GetByID(id string) (*User, error)
    GetByEmail(email string) (*User, error)
    GetAll() ([]*User, error)
    Update(user *User) error
    Delete(id string) error
    FindByAge(minAge, maxAge int) ([]*User, error)
}

type CacheRepository interface {
    Get(key string) (any, bool)
    Set(key string, value any, ttl time.Duration)
    Delete(key string)
    Clear()
}

// In-memory repository implementation
type InMemoryUserRepository struct {
    users  map[string]*User
    emails map[string]string // email -> id mapping
    mutex  sync.RWMutex
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{
        users:  make(map[string]*User),
        emails: make(map[string]string),
    }
}

func (r *InMemoryUserRepository) Create(user *User) error {
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    if user.ID == "" {
        return fmt.Errorf("user ID cannot be empty")
    }
    
    if _, exists := r.users[user.ID]; exists {
        return fmt.Errorf("user with ID %s already exists", user.ID)
    }
    
    if _, exists := r.emails[user.Email]; exists {
        return fmt.Errorf("user with email %s already exists", user.Email)
    }
    
    now := time.Now()
    userCopy := *user
    userCopy.CreatedAt = now
    userCopy.UpdatedAt = now
    
    r.users[user.ID] = &userCopy
    r.emails[user.Email] = user.ID
    
    return nil
}

func (r *InMemoryUserRepository) GetByID(id string) (*User, error) {
    r.mutex.RLock()
    defer r.mutex.RUnlock()
    
    user, exists := r.users[id]
    if !exists {
        return nil, fmt.Errorf("user not found with ID: %s", id)
    }
    
    // Return a copy to prevent external modification
    userCopy := *user
    return &userCopy, nil
}

func (r *InMemoryUserRepository) GetByEmail(email string) (*User, error) {
    r.mutex.RLock()
    defer r.mutex.RUnlock()
    
    id, exists := r.emails[email]
    if !exists {
        return nil, fmt.Errorf("user not found with email: %s", email)
    }
    
    return r.GetByID(id)
}

func (r *InMemoryUserRepository) GetAll() ([]*User, error) {
    r.mutex.RLock()
    defer r.mutex.RUnlock()
    
    users := make([]*User, 0, len(r.users))
    for _, user := range r.users {
        userCopy := *user
        users = append(users, &userCopy)
    }
    
    return users, nil
}

func (r *InMemoryUserRepository) Update(user *User) error {
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    existing, exists := r.users[user.ID]
    if !exists {
        return fmt.Errorf("user not found with ID: %s", user.ID)
    }
    
    // Check if email is changing and if new email already exists
    if existing.Email != user.Email {
        if _, emailExists := r.emails[user.Email]; emailExists {
            return fmt.Errorf("user with email %s already exists", user.Email)
        }
        
        // Update email mapping
        delete(r.emails, existing.Email)
        r.emails[user.Email] = user.ID
    }
    
    userCopy := *user
    userCopy.CreatedAt = existing.CreatedAt // Preserve creation time
    userCopy.UpdatedAt = time.Now()
    
    r.users[user.ID] = &userCopy
    
    return nil
}

func (r *InMemoryUserRepository) Delete(id string) error {
    r.mutex.Lock()
    defer r.mutex.Unlock()
    
    user, exists := r.users[id]
    if !exists {
        return fmt.Errorf("user not found with ID: %s", id)
    }
    
    delete(r.users, id)
    delete(r.emails, user.Email)
    
    return nil
}

func (r *InMemoryUserRepository) FindByAge(minAge, maxAge int) ([]*User, error) {
    r.mutex.RLock()
    defer r.mutex.RUnlock()
    
    var matchingUsers []*User
    
    for _, user := range r.users {
        if user.Age >= minAge && user.Age <= maxAge {
            userCopy := *user
            matchingUsers = append(matchingUsers, &userCopy)
        }
    }
    
    return matchingUsers, nil
}

// Cached repository decorator
type CachedUserRepository struct {
    repository UserRepository
    cache      CacheRepository
    cacheTTL   time.Duration
}

func NewCachedUserRepository(repo UserRepository, cache CacheRepository, ttl time.Duration) *CachedUserRepository {
    return &CachedUserRepository{
        repository: repo,
        cache:      cache,
        cacheTTL:   ttl,
    }
}

func (cr *CachedUserRepository) Create(user *User) error {
    err := cr.repository.Create(user)
    if err != nil {
        return err
    }
    
    // Cache the created user
    cr.cache.Set("user:id:"+user.ID, user, cr.cacheTTL)
    cr.cache.Set("user:email:"+user.Email, user, cr.cacheTTL)
    
    return nil
}

func (cr *CachedUserRepository) GetByID(id string) (*User, error) {
    cacheKey := "user:id:" + id
    
    if cached, found := cr.cache.Get(cacheKey); found {
        fmt.Printf("Cache hit for user ID: %s\n", id)
        user := cached.(*User)
        userCopy := *user
        return &userCopy, nil
    }
    
    fmt.Printf("Cache miss for user ID: %s\n", id)
    user, err := cr.repository.GetByID(id)
    if err != nil {
        return nil, err
    }
    
    cr.cache.Set(cacheKey, user, cr.cacheTTL)
    return user, nil
}

func (cr *CachedUserRepository) GetByEmail(email string) (*User, error) {
    cacheKey := "user:email:" + email
    
    if cached, found := cr.cache.Get(cacheKey); found {
        fmt.Printf("Cache hit for user email: %s\n", email)
        user := cached.(*User)
        userCopy := *user
        return &userCopy, nil
    }
    
    fmt.Printf("Cache miss for user email: %s\n", email)
    user, err := cr.repository.GetByEmail(email)
    if err != nil {
        return nil, err
    }
    
    cr.cache.Set(cacheKey, user, cr.cacheTTL)
    return user, nil
}

func (cr *CachedUserRepository) GetAll() ([]*User, error) {
    // For GetAll, we'll skip caching to keep it simple
    return cr.repository.GetAll()
}

func (cr *CachedUserRepository) Update(user *User) error {
    // Get old user to clear old cache entries
    oldUser, err := cr.repository.GetByID(user.ID)
    if err != nil {
        return err
    }
    
    err = cr.repository.Update(user)
    if err != nil {
        return err
    }
    
    // Clear old cache entries
    cr.cache.Delete("user:id:" + user.ID)
    cr.cache.Delete("user:email:" + oldUser.Email)
    
    // Cache updated user
    cr.cache.Set("user:id:"+user.ID, user, cr.cacheTTL)
    cr.cache.Set("user:email:"+user.Email, user, cr.cacheTTL)
    
    return nil
}

func (cr *CachedUserRepository) Delete(id string) error {
    // Get user to clear cache entries
    user, err := cr.repository.GetByID(id)
    if err != nil {
        return err
    }
    
    err = cr.repository.Delete(id)
    if err != nil {
        return err
    }
    
    // Clear cache entries
    cr.cache.Delete("user:id:" + id)
    cr.cache.Delete("user:email:" + user.Email)
    
    return nil
}

func (cr *CachedUserRepository) FindByAge(minAge, maxAge int) ([]*User, error) {
    // For complex queries, we'll skip caching
    return cr.repository.FindByAge(minAge, maxAge)
}

// Simple in-memory cache
type InMemoryCache struct {
    data   map[string]cacheItem
    mutex  sync.RWMutex
}

type cacheItem struct {
    value      any
    expiration time.Time
}

func NewInMemoryCache() *InMemoryCache {
    cache := &InMemoryCache{
        data: make(map[string]cacheItem),
    }
    
    // Start cleanup goroutine
    go cache.cleanup()
    
    return cache
}

func (c *InMemoryCache) Get(key string) (any, bool) {
    c.mutex.RLock()
    defer c.mutex.RUnlock()
    
    item, exists := c.data[key]
    if !exists || time.Now().After(item.expiration) {
        return nil, false
    }
    
    return item.value, true
}

func (c *InMemoryCache) Set(key string, value any, ttl time.Duration) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    c.data[key] = cacheItem{
        value:      value,
        expiration: time.Now().Add(ttl),
    }
}

func (c *InMemoryCache) Delete(key string) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    delete(c.data, key)
}

func (c *InMemoryCache) Clear() {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    
    c.data = make(map[string]cacheItem)
}

func (c *InMemoryCache) cleanup() {
    ticker := time.NewTicker(1 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        c.mutex.Lock()
        now := time.Now()
        for key, item := range c.data {
            if now.After(item.expiration) {
                delete(c.data, key)
            }
        }
        c.mutex.Unlock()
    }
}

// User service that uses repository
type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (us *UserService) CreateUser(id, name, email string, age int) error {
    user := &User{
        ID:    id,
        Name:  name,
        Email: email,
        Age:   age,
    }
    
    return us.repo.Create(user)
}

func (us *UserService) GetUser(id string) (*User, error) {
    return us.repo.GetByID(id)
}

func (us *UserService) UpdateUserEmail(id, newEmail string) error {
    user, err := us.repo.GetByID(id)
    if err != nil {
        return err
    }
    
    user.Email = newEmail
    return us.repo.Update(user)
}

func (us *UserService) GetUsersByAgeRange(minAge, maxAge int) ([]*User, error) {
    return us.repo.FindByAge(minAge, maxAge)
}

func main() {
    // Create repositories
    memRepo := NewInMemoryUserRepository()
    cache := NewInMemoryCache()
    cachedRepo := NewCachedUserRepository(memRepo, cache, 5*time.Minute)
    
    // Create service
    service := NewUserService(cachedRepo)
    
    // Test the service
    fmt.Println("=== Creating Users ===")
    users := []struct {
        id    string
        name  string
        email string
        age   int
    }{
        {"1", "Alice Johnson", "alice@example.com", 28},
        {"2", "Bob Smith", "bob@example.com", 35},
        {"3", "Charlie Brown", "charlie@example.com", 22},
        {"4", "Diana Prince", "diana@example.com", 30},
    }
    
    for _, u := range users {
        err := service.CreateUser(u.id, u.name, u.email, u.age)
        if err != nil {
            fmt.Printf("Error creating user %s: %v\n", u.name, err)
        } else {
            fmt.Printf("Created user: %s\n", u.name)
        }
    }
    
    fmt.Println("\n=== Testing Cache (Multiple Gets) ===")
    for i := 0; i < 3; i++ {
        user, err := service.GetUser("1")
        if err != nil {
            fmt.Printf("Error: %v\n", err)
        } else {
            fmt.Printf("Retrieved user: %s (%s)\n", user.Name, user.Email)
        }
    }
    
    fmt.Println("\n=== Finding Users by Age ===")
    youngUsers, err := service.GetUsersByAgeRange(20, 30)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Users aged 20-30:\n")
        for _, user := range youngUsers {
            fmt.Printf("  - %s (%d years old)\n", user.Name, user.Age)
        }
    }
    
    fmt.Println("\n=== Updating User Email ===")
    err := service.UpdateUserEmail("1", "alice.johnson@newdomain.com")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Println("Updated Alice's email")
    }
    
    // Verify update with cache behavior
    user, err := service.GetUser("1")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Alice's new email: %s\n", user.Email)
    }
}
```

The repository pattern provides a clean abstraction for data access,  
enabling different storage implementations and decorators like caching.  
This promotes testability and separation of business logic from data  
access concerns.  

## Interface mocking for testing

Interfaces enable easy mocking for unit tests, allowing isolation of  
components and testing of different scenarios.  

```go
package main

import (
    "fmt"
    "testing"
    "time"
)

// Service interfaces
type PaymentGateway interface {
    ProcessPayment(amount float64, cardToken string) (*PaymentResult, error)
    RefundPayment(transactionID string, amount float64) (*RefundResult, error)
    GetTransactionStatus(transactionID string) (string, error)
}

type EmailService interface {
    SendEmail(to, subject, body string) error
    SendHTMLEmail(to, subject, htmlBody string) error
}

type Logger interface {
    Info(message string)
    Error(message string)
    Debug(message string)
}

// Data types
type PaymentResult struct {
    TransactionID string
    Status        string
    ProcessedAt   time.Time
    Amount        float64
}

type RefundResult struct {
    RefundID    string
    Status      string
    ProcessedAt time.Time
    Amount      float64
}

type Order struct {
    ID          string
    CustomerID  string
    Amount      float64
    Status      string
    PaymentID   string
}

// Business service that uses interfaces
type OrderService struct {
    paymentGateway PaymentGateway
    emailService   EmailService
    logger         Logger
}

func NewOrderService(pg PaymentGateway, es EmailService, l Logger) *OrderService {
    return &OrderService{
        paymentGateway: pg,
        emailService:   es,
        logger:         l,
    }
}

func (os *OrderService) ProcessOrder(order *Order, cardToken string) error {
    os.logger.Info(fmt.Sprintf("Processing order %s for customer %s", 
                               order.ID, order.CustomerID))
    
    // Process payment
    result, err := os.paymentGateway.ProcessPayment(order.Amount, cardToken)
    if err != nil {
        os.logger.Error(fmt.Sprintf("Payment failed for order %s: %v", 
                                    order.ID, err))
        order.Status = "failed"
        return fmt.Errorf("payment processing failed: %v", err)
    }
    
    // Update order with payment info
    order.PaymentID = result.TransactionID
    order.Status = "paid"
    
    os.logger.Info(fmt.Sprintf("Payment successful for order %s. Transaction: %s", 
                               order.ID, result.TransactionID))
    
    // Send confirmation email
    emailBody := fmt.Sprintf(`
        Dear Customer,
        
        Your order %s has been successfully processed.
        Payment Amount: $%.2f
        Transaction ID: %s
        
        Thank you for your business!
    `, order.ID, order.Amount, result.TransactionID)
    
    err = os.emailService.SendEmail("customer@example.com", 
                                    "Order Confirmation", emailBody)
    if err != nil {
        os.logger.Error(fmt.Sprintf("Failed to send confirmation email for order %s: %v", 
                                    order.ID, err))
        // Don't fail the order processing just because email failed
    }
    
    return nil
}

func (os *OrderService) RefundOrder(order *Order, amount float64) error {
    if order.Status != "paid" {
        return fmt.Errorf("cannot refund order with status: %s", order.Status)
    }
    
    os.logger.Info(fmt.Sprintf("Processing refund for order %s", order.ID))
    
    result, err := os.paymentGateway.RefundPayment(order.PaymentID, amount)
    if err != nil {
        os.logger.Error(fmt.Sprintf("Refund failed for order %s: %v", 
                                    order.ID, err))
        return fmt.Errorf("refund processing failed: %v", err)
    }
    
    order.Status = "refunded"
    
    os.logger.Info(fmt.Sprintf("Refund successful for order %s. Refund ID: %s", 
                               order.ID, result.RefundID))
    
    // Send refund notification email
    emailBody := fmt.Sprintf(`
        Dear Customer,
        
        Your refund for order %s has been processed.
        Refund Amount: $%.2f
        Refund ID: %s
        
        Please allow 3-5 business days for the refund to appear on your statement.
    `, order.ID, amount, result.RefundID)
    
    err = os.emailService.SendEmail("customer@example.com", 
                                    "Refund Processed", emailBody)
    if err != nil {
        os.logger.Error(fmt.Sprintf("Failed to send refund notification for order %s: %v", 
                                    order.ID, err))
    }
    
    return nil
}

// Mock implementations for testing
type MockPaymentGateway struct {
    shouldFailPayment bool
    shouldFailRefund  bool
    transactions      map[string]*PaymentResult
    refunds          map[string]*RefundResult
}

func NewMockPaymentGateway() *MockPaymentGateway {
    return &MockPaymentGateway{
        transactions: make(map[string]*PaymentResult),
        refunds:      make(map[string]*RefundResult),
    }
}

func (mpg *MockPaymentGateway) SetPaymentFailure(shouldFail bool) {
    mpg.shouldFailPayment = shouldFail
}

func (mpg *MockPaymentGateway) SetRefundFailure(shouldFail bool) {
    mpg.shouldFailRefund = shouldFail
}

func (mpg *MockPaymentGateway) ProcessPayment(amount float64, cardToken string) (*PaymentResult, error) {
    if mpg.shouldFailPayment {
        return nil, fmt.Errorf("payment gateway error: insufficient funds")
    }
    
    result := &PaymentResult{
        TransactionID: fmt.Sprintf("txn_%d", time.Now().UnixNano()),
        Status:        "completed",
        ProcessedAt:   time.Now(),
        Amount:        amount,
    }
    
    mpg.transactions[result.TransactionID] = result
    return result, nil
}

func (mpg *MockPaymentGateway) RefundPayment(transactionID string, amount float64) (*RefundResult, error) {
    if mpg.shouldFailRefund {
        return nil, fmt.Errorf("refund not allowed for this transaction")
    }
    
    // Check if transaction exists
    if _, exists := mpg.transactions[transactionID]; !exists {
        return nil, fmt.Errorf("transaction not found: %s", transactionID)
    }
    
    result := &RefundResult{
        RefundID:    fmt.Sprintf("ref_%d", time.Now().UnixNano()),
        Status:      "completed",
        ProcessedAt: time.Now(),
        Amount:      amount,
    }
    
    mpg.refunds[result.RefundID] = result
    return result, nil
}

func (mpg *MockPaymentGateway) GetTransactionStatus(transactionID string) (string, error) {
    if txn, exists := mpg.transactions[transactionID]; exists {
        return txn.Status, nil
    }
    return "", fmt.Errorf("transaction not found: %s", transactionID)
}

type MockEmailService struct {
    sentEmails    []EmailMessage
    shouldFail    bool
    failAfterCount int
    emailCount    int
}

type EmailMessage struct {
    To      string
    Subject string
    Body    string
    IsHTML  bool
    SentAt  time.Time
}

func NewMockEmailService() *MockEmailService {
    return &MockEmailService{
        sentEmails: make([]EmailMessage, 0),
    }
}

func (mes *MockEmailService) SetFailure(shouldFail bool) {
    mes.shouldFail = shouldFail
}

func (mes *MockEmailService) SetFailAfterCount(count int) {
    mes.failAfterCount = count
}

func (mes *MockEmailService) SendEmail(to, subject, body string) error {
    mes.emailCount++
    
    if mes.shouldFail || (mes.failAfterCount > 0 && mes.emailCount > mes.failAfterCount) {
        return fmt.Errorf("email service unavailable")
    }
    
    email := EmailMessage{
        To:      to,
        Subject: subject,
        Body:    body,
        IsHTML:  false,
        SentAt:  time.Now(),
    }
    
    mes.sentEmails = append(mes.sentEmails, email)
    return nil
}

func (mes *MockEmailService) SendHTMLEmail(to, subject, htmlBody string) error {
    mes.emailCount++
    
    if mes.shouldFail || (mes.failAfterCount > 0 && mes.emailCount > mes.failAfterCount) {
        return fmt.Errorf("email service unavailable")
    }
    
    email := EmailMessage{
        To:      to,
        Subject: subject,
        Body:    htmlBody,
        IsHTML:  true,
        SentAt:  time.Now(),
    }
    
    mes.sentEmails = append(mes.sentEmails, email)
    return nil
}

func (mes *MockEmailService) GetSentEmails() []EmailMessage {
    return mes.sentEmails
}

func (mes *MockEmailService) GetEmailCount() int {
    return len(mes.sentEmails)
}

type MockLogger struct {
    logs []LogEntry
}

type LogEntry struct {
    Level   string
    Message string
    Time    time.Time
}

func NewMockLogger() *MockLogger {
    return &MockLogger{
        logs: make([]LogEntry, 0),
    }
}

func (ml *MockLogger) Info(message string) {
    ml.logs = append(ml.logs, LogEntry{
        Level:   "INFO",
        Message: message,
        Time:    time.Now(),
    })
}

func (ml *MockLogger) Error(message string) {
    ml.logs = append(ml.logs, LogEntry{
        Level:   "ERROR",
        Message: message,
        Time:    time.Now(),
    })
}

func (ml *MockLogger) Debug(message string) {
    ml.logs = append(ml.logs, LogEntry{
        Level:   "DEBUG",
        Message: message,
        Time:    time.Now(),
    })
}

func (ml *MockLogger) GetLogs() []LogEntry {
    return ml.logs
}

func (ml *MockLogger) GetLogsByLevel(level string) []LogEntry {
    var filtered []LogEntry
    for _, log := range ml.logs {
        if log.Level == level {
            filtered = append(filtered, log)
        }
    }
    return filtered
}

// Test functions
func TestSuccessfulOrderProcessing() error {
    fmt.Println("=== Test: Successful Order Processing ===")
    
    // Setup mocks
    paymentGateway := NewMockPaymentGateway()
    emailService := NewMockEmailService()
    logger := NewMockLogger()
    
    // Create service
    orderService := NewOrderService(paymentGateway, emailService, logger)
    
    // Create test order
    order := &Order{
        ID:         "order123",
        CustomerID: "customer456",
        Amount:     99.99,
        Status:     "pending",
    }
    
    // Process order
    err := orderService.ProcessOrder(order, "card_token_123")
    if err != nil {
        return fmt.Errorf("expected successful processing, got error: %v", err)
    }
    
    // Verify order status
    if order.Status != "paid" {
        return fmt.Errorf("expected order status 'paid', got '%s'", order.Status)
    }
    
    if order.PaymentID == "" {
        return fmt.Errorf("expected payment ID to be set")
    }
    
    // Verify email was sent
    if emailService.GetEmailCount() != 1 {
        return fmt.Errorf("expected 1 email sent, got %d", emailService.GetEmailCount())
    }
    
    // Verify logs
    infoLogs := logger.GetLogsByLevel("INFO")
    if len(infoLogs) < 2 {
        return fmt.Errorf("expected at least 2 info logs, got %d", len(infoLogs))
    }
    
    fmt.Println("✓ Order processed successfully")
    fmt.Printf("✓ Payment ID: %s\n", order.PaymentID)
    fmt.Printf("✓ Emails sent: %d\n", emailService.GetEmailCount())
    fmt.Printf("✓ Info logs: %d\n", len(infoLogs))
    
    return nil
}

func TestPaymentFailure() error {
    fmt.Println("\n=== Test: Payment Failure ===")
    
    // Setup mocks with payment failure
    paymentGateway := NewMockPaymentGateway()
    paymentGateway.SetPaymentFailure(true)
    emailService := NewMockEmailService()
    logger := NewMockLogger()
    
    // Create service
    orderService := NewOrderService(paymentGateway, emailService, logger)
    
    // Create test order
    order := &Order{
        ID:         "order124",
        CustomerID: "customer457",
        Amount:     149.99,
        Status:     "pending",
    }
    
    // Process order (should fail)
    err := orderService.ProcessOrder(order, "card_token_456")
    if err == nil {
        return fmt.Errorf("expected payment failure, but processing succeeded")
    }
    
    // Verify order status
    if order.Status != "failed" {
        return fmt.Errorf("expected order status 'failed', got '%s'", order.Status)
    }
    
    // Verify no emails were sent
    if emailService.GetEmailCount() != 0 {
        return fmt.Errorf("expected 0 emails sent, got %d", emailService.GetEmailCount())
    }
    
    // Verify error logs
    errorLogs := logger.GetLogsByLevel("ERROR")
    if len(errorLogs) == 0 {
        return fmt.Errorf("expected at least 1 error log, got %d", len(errorLogs))
    }
    
    fmt.Println("✓ Payment failure handled correctly")
    fmt.Printf("✓ Order status: %s\n", order.Status)
    fmt.Printf("✓ Error logs: %d\n", len(errorLogs))
    
    return nil
}

func TestEmailFailure() error {
    fmt.Println("\n=== Test: Email Failure (Order Still Succeeds) ===")
    
    // Setup mocks with email failure
    paymentGateway := NewMockPaymentGateway()
    emailService := NewMockEmailService()
    emailService.SetFailure(true)
    logger := NewMockLogger()
    
    // Create service
    orderService := NewOrderService(paymentGateway, emailService, logger)
    
    // Create test order
    order := &Order{
        ID:         "order125",
        CustomerID: "customer458",
        Amount:     199.99,
        Status:     "pending",
    }
    
    // Process order (should succeed despite email failure)
    err := orderService.ProcessOrder(order, "card_token_789")
    if err != nil {
        return fmt.Errorf("expected successful processing despite email failure, got error: %v", err)
    }
    
    // Verify order status (should still be paid)
    if order.Status != "paid" {
        return fmt.Errorf("expected order status 'paid', got '%s'", order.Status)
    }
    
    // Verify no emails were sent
    if emailService.GetEmailCount() != 0 {
        return fmt.Errorf("expected 0 emails sent, got %d", emailService.GetEmailCount())
    }
    
    // Verify error logs about email failure
    errorLogs := logger.GetLogsByLevel("ERROR")
    emailErrorFound := false
    for _, log := range errorLogs {
        if contains(log.Message, "Failed to send confirmation email") {
            emailErrorFound = true
            break
        }
    }
    
    if !emailErrorFound {
        return fmt.Errorf("expected error log about email failure")
    }
    
    fmt.Println("✓ Order succeeded despite email failure")
    fmt.Printf("✓ Order status: %s\n", order.Status)
    fmt.Printf("✓ Email error logged correctly\n")
    
    return nil
}

func TestRefundProcess() error {
    fmt.Println("\n=== Test: Refund Process ===")
    
    // Setup mocks
    paymentGateway := NewMockPaymentGateway()
    emailService := NewMockEmailService()
    logger := NewMockLogger()
    
    // Create service
    orderService := NewOrderService(paymentGateway, emailService, logger)
    
    // Create and process order first
    order := &Order{
        ID:         "order126",
        CustomerID: "customer459",
        Amount:     299.99,
        Status:     "pending",
    }
    
    err := orderService.ProcessOrder(order, "card_token_abc")
    if err != nil {
        return fmt.Errorf("failed to process order: %v", err)
    }
    
    // Process refund
    refundAmount := 150.00
    err = orderService.RefundOrder(order, refundAmount)
    if err != nil {
        return fmt.Errorf("failed to process refund: %v", err)
    }
    
    // Verify order status
    if order.Status != "refunded" {
        return fmt.Errorf("expected order status 'refunded', got '%s'", order.Status)
    }
    
    // Verify emails (1 for order confirmation + 1 for refund)
    if emailService.GetEmailCount() != 2 {
        return fmt.Errorf("expected 2 emails sent, got %d", emailService.GetEmailCount())
    }
    
    fmt.Println("✓ Refund processed successfully")
    fmt.Printf("✓ Order status: %s\n", order.Status)
    fmt.Printf("✓ Total emails sent: %d\n", emailService.GetEmailCount())
    
    return nil
}

func contains(s, substr string) bool {
    return len(s) >= len(substr) && (s == substr || 
           (len(s) > len(substr) && 
            (s[:len(substr)] == substr || s[len(s)-len(substr):] == substr ||
             containsSubstring(s, substr))))
}

func containsSubstring(s, substr string) bool {
    for i := 0; i <= len(s)-len(substr); i++ {
        if s[i:i+len(substr)] == substr {
            return true
        }
    }
    return false
}

func main() {
    tests := []func() error{
        TestSuccessfulOrderProcessing,
        TestPaymentFailure,
        TestEmailFailure,
        TestRefundProcess,
    }
    
    passed := 0
    failed := 0
    
    for _, test := range tests {
        if err := test(); err != nil {
            fmt.Printf("❌ Test failed: %v\n", err)
            failed++
        } else {
            passed++
        }
    }
    
    fmt.Printf("\n=== Test Results ===\n")
    fmt.Printf("Passed: %d\n", passed)
    fmt.Printf("Failed: %d\n", failed)
    fmt.Printf("Total:  %d\n", passed+failed)
    
    if failed == 0 {
        fmt.Println("🎉 All tests passed!")
    }
}
```

Interface mocking enables comprehensive testing by allowing developers to  
control the behavior of dependencies. This facilitates testing of edge  
cases, error conditions, and various scenarios without external dependencies.  

## Interface for resource pooling

Resource pooling uses interfaces to manage expensive resources like  
database connections, with automatic lifecycle management.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Resource interface
type Resource interface {
    GetID() string
    IsHealthy() bool
    Reset() error
    Close() error
    LastUsed() time.Time
}

// Pool interface
type ResourcePool interface {
    Get() (Resource, error)
    Put(resource Resource) error
    Close() error
    Stats() PoolStats
}

type PoolStats struct {
    TotalResources    int
    ActiveResources   int
    IdleResources     int
    CreatedResources  int64
    ReclaimedResources int64
}

// Database connection resource
type DBConnection struct {
    id          string
    isConnected bool
    lastUsed    time.Time
    queryCount  int
    mutex       sync.Mutex
}

func NewDBConnection(id string) *DBConnection {
    return &DBConnection{
        id:          id,
        isConnected: true,
        lastUsed:    time.Now(),
        queryCount:  0,
    }
}

func (db *DBConnection) GetID() string {
    return db.id
}

func (db *DBConnection) IsHealthy() bool {
    db.mutex.Lock()
    defer db.mutex.Unlock()
    
    // Simulate health check
    if !db.isConnected {
        return false
    }
    
    // Connection is unhealthy if unused for too long
    if time.Since(db.lastUsed) > 30*time.Minute {
        return false
    }
    
    // Connection is unhealthy if used too much
    if db.queryCount > 10000 {
        return false
    }
    
    return true
}

func (db *DBConnection) Reset() error {
    db.mutex.Lock()
    defer db.mutex.Unlock()
    
    if !db.isConnected {
        return fmt.Errorf("cannot reset disconnected connection")
    }
    
    fmt.Printf("Resetting connection %s\n", db.id)
    db.queryCount = 0
    db.lastUsed = time.Now()
    
    return nil
}

func (db *DBConnection) Close() error {
    db.mutex.Lock()
    defer db.mutex.Unlock()
    
    if !db.isConnected {
        return nil
    }
    
    fmt.Printf("Closing connection %s\n", db.id)
    db.isConnected = false
    
    return nil
}

func (db *DBConnection) LastUsed() time.Time {
    db.mutex.Lock()
    defer db.mutex.Unlock()
    return db.lastUsed
}

func (db *DBConnection) ExecuteQuery(query string) (string, error) {
    db.mutex.Lock()
    defer db.mutex.Unlock()
    
    if !db.isConnected {
        return "", fmt.Errorf("connection is closed")
    }
    
    db.lastUsed = time.Now()
    db.queryCount++
    
    // Simulate query execution
    time.Sleep(10 * time.Millisecond)
    
    result := fmt.Sprintf("Result for query '%s' from connection %s (query #%d)", 
                          query, db.id, db.queryCount)
    
    return result, nil
}

// Resource factory interface
type ResourceFactory interface {
    CreateResource() (Resource, error)
    ValidateResource(resource Resource) bool
}

type DBConnectionFactory struct {
    connectionCount int64
    mutex          sync.Mutex
}

func NewDBConnectionFactory() *DBConnectionFactory {
    return &DBConnectionFactory{}
}

func (factory *DBConnectionFactory) CreateResource() (Resource, error) {
    factory.mutex.Lock()
    defer factory.mutex.Unlock()
    
    factory.connectionCount++
    id := fmt.Sprintf("conn_%d", factory.connectionCount)
    
    // Simulate connection establishment
    fmt.Printf("Creating new database connection: %s\n", id)
    time.Sleep(100 * time.Millisecond)
    
    return NewDBConnection(id), nil
}

func (factory *DBConnectionFactory) ValidateResource(resource Resource) bool {
    return resource != nil && resource.IsHealthy()
}

// Pool configuration
type PoolConfig struct {
    MinSize          int
    MaxSize          int
    MaxIdleTime      time.Duration
    ValidationInterval time.Duration
    AcquisitionTimeout time.Duration
}

// Default pool implementation
type DefaultResourcePool struct {
    factory    ResourceFactory
    config     PoolConfig
    resources  []Resource
    active     map[string]Resource
    stats      PoolStats
    mutex      sync.Mutex
    closed     bool
    stopCleanup chan bool
}

func NewDefaultResourcePool(factory ResourceFactory, config PoolConfig) *DefaultResourcePool {
    pool := &DefaultResourcePool{
        factory:     factory,
        config:      config,
        resources:   make([]Resource, 0),
        active:      make(map[string]Resource),
        stopCleanup: make(chan bool),
    }
    
    // Initialize minimum resources
    pool.initialize()
    
    // Start cleanup routine
    go pool.cleanupRoutine()
    
    return pool
}

func (p *DefaultResourcePool) initialize() error {
    for i := 0; i < p.config.MinSize; i++ {
        resource, err := p.factory.CreateResource()
        if err != nil {
            return fmt.Errorf("failed to create initial resource: %v", err)
        }
        
        p.resources = append(p.resources, resource)
        p.stats.CreatedResources++
    }
    
    return nil
}

func (p *DefaultResourcePool) Get() (Resource, error) {
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    if p.closed {
        return nil, fmt.Errorf("pool is closed")
    }
    
    // Try to get an existing resource
    for i, resource := range p.resources {
        if p.factory.ValidateResource(resource) {
            // Remove from idle pool
            p.resources = append(p.resources[:i], p.resources[i+1:]...)
            
            // Add to active pool
            p.active[resource.GetID()] = resource
            
            // Reset resource
            err := resource.Reset()
            if err != nil {
                resource.Close()
                delete(p.active, resource.GetID())
                return nil, fmt.Errorf("failed to reset resource: %v", err)
            }
            
            return resource, nil
        } else {
            // Remove unhealthy resource
            p.resources = append(p.resources[:i], p.resources[i+1:]...)
            resource.Close()
            p.stats.ReclaimedResources++
            break
        }
    }
    
    // Create new resource if under max size
    totalResources := len(p.resources) + len(p.active)
    if totalResources < p.config.MaxSize {
        resource, err := p.factory.CreateResource()
        if err != nil {
            return nil, fmt.Errorf("failed to create resource: %v", err)
        }
        
        p.active[resource.GetID()] = resource
        p.stats.CreatedResources++
        
        return resource, nil
    }
    
    return nil, fmt.Errorf("pool exhausted: %d resources in use", len(p.active))
}

func (p *DefaultResourcePool) Put(resource Resource) error {
    if resource == nil {
        return fmt.Errorf("cannot return nil resource to pool")
    }
    
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    if p.closed {
        resource.Close()
        return fmt.Errorf("pool is closed")
    }
    
    resourceID := resource.GetID()
    
    // Check if resource is from this pool
    if _, exists := p.active[resourceID]; !exists {
        resource.Close()
        return fmt.Errorf("resource does not belong to this pool")
    }
    
    // Remove from active pool
    delete(p.active, resourceID)
    
    // Validate resource health
    if !p.factory.ValidateResource(resource) {
        resource.Close()
        p.stats.ReclaimedResources++
        return nil
    }
    
    // Return to idle pool
    p.resources = append(p.resources, resource)
    
    return nil
}

func (p *DefaultResourcePool) Close() error {
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    if p.closed {
        return nil
    }
    
    p.closed = true
    close(p.stopCleanup)
    
    // Close all idle resources
    for _, resource := range p.resources {
        resource.Close()
    }
    
    // Close all active resources
    for _, resource := range p.active {
        resource.Close()
    }
    
    p.resources = nil
    p.active = make(map[string]Resource)
    
    fmt.Println("Resource pool closed")
    return nil
}

func (p *DefaultResourcePool) Stats() PoolStats {
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    p.stats.TotalResources = len(p.resources) + len(p.active)
    p.stats.ActiveResources = len(p.active)
    p.stats.IdleResources = len(p.resources)
    
    return p.stats
}

func (p *DefaultResourcePool) cleanupRoutine() {
    ticker := time.NewTicker(p.config.ValidationInterval)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            p.cleanup()
        case <-p.stopCleanup:
            return
        }
    }
}

func (p *DefaultResourcePool) cleanup() {
    p.mutex.Lock()
    defer p.mutex.Unlock()
    
    if p.closed {
        return
    }
    
    now := time.Now()
    var validResources []Resource
    reclaimedCount := 0
    
    // Check idle resources
    for _, resource := range p.resources {
        if !p.factory.ValidateResource(resource) || 
           now.Sub(resource.LastUsed()) > p.config.MaxIdleTime {
            resource.Close()
            reclaimedCount++
        } else {
            validResources = append(validResources, resource)
        }
    }
    
    p.resources = validResources
    p.stats.ReclaimedResources += int64(reclaimedCount)
    
    if reclaimedCount > 0 {
        fmt.Printf("Cleanup: reclaimed %d resources\n", reclaimedCount)
    }
}

// Database service using pool
type DatabaseService struct {
    pool ResourcePool
}

func NewDatabaseService(pool ResourcePool) *DatabaseService {
    return &DatabaseService{pool: pool}
}

func (ds *DatabaseService) ExecuteQuery(query string) (string, error) {
    // Get resource from pool
    resource, err := ds.pool.Get()
    if err != nil {
        return "", fmt.Errorf("failed to acquire database connection: %v", err)
    }
    
    // Ensure resource is returned to pool
    defer func() {
        putErr := ds.pool.Put(resource)
        if putErr != nil {
            fmt.Printf("Warning: failed to return resource to pool: %v\n", putErr)
        }
    }()
    
    // Cast to database connection
    dbConn, ok := resource.(*DBConnection)
    if !ok {
        return "", fmt.Errorf("invalid resource type")
    }
    
    // Execute query
    return dbConn.ExecuteQuery(query)
}

func (ds *DatabaseService) GetPoolStats() PoolStats {
    return ds.pool.Stats()
}

func main() {
    // Create pool configuration
    config := PoolConfig{
        MinSize:            2,
        MaxSize:            5,
        MaxIdleTime:        5 * time.Second,
        ValidationInterval: 2 * time.Second,
        AcquisitionTimeout: 1 * time.Second,
    }
    
    // Create factory and pool
    factory := NewDBConnectionFactory()
    pool := NewDefaultResourcePool(factory, config)
    defer pool.Close()
    
    // Create service
    service := NewDatabaseService(pool)
    
    fmt.Println("=== Initial Pool Stats ===")
    stats := service.GetPoolStats()
    fmt.Printf("Total: %d, Active: %d, Idle: %d, Created: %d\n",
               stats.TotalResources, stats.ActiveResources, 
               stats.IdleResources, stats.CreatedResources)
    
    fmt.Println("\n=== Executing Queries ===")
    queries := []string{
        "SELECT * FROM users",
        "SELECT * FROM orders",
        "SELECT * FROM products",
        "SELECT * FROM inventory",
        "SELECT * FROM reports",
    }
    
    // Execute queries concurrently
    var wg sync.WaitGroup
    for i, query := range queries {
        wg.Add(1)
        go func(queryNum int, q string) {
            defer wg.Done()
            
            result, err := service.ExecuteQuery(q)
            if err != nil {
                fmt.Printf("Query %d failed: %v\n", queryNum, err)
            } else {
                fmt.Printf("Query %d result: %s\n", queryNum, result)
            }
        }(i+1, query)
    }
    
    wg.Wait()
    
    fmt.Println("\n=== After Query Execution ===")
    stats = service.GetPoolStats()
    fmt.Printf("Total: %d, Active: %d, Idle: %d, Created: %d, Reclaimed: %d\n",
               stats.TotalResources, stats.ActiveResources, 
               stats.IdleResources, stats.CreatedResources, stats.ReclaimedResources)
    
    // Wait for cleanup to run
    fmt.Println("\n=== Waiting for cleanup ===")
    time.Sleep(7 * time.Second)
    
    fmt.Println("\n=== After Cleanup ===")
    stats = service.GetPoolStats()
    fmt.Printf("Total: %d, Active: %d, Idle: %d, Created: %d, Reclaimed: %d\n",
               stats.TotalResources, stats.ActiveResources, 
               stats.IdleResources, stats.CreatedResources, stats.ReclaimedResources)
    
    // Test pool exhaustion
    fmt.Println("\n=== Testing Pool Exhaustion ===")
    var resources []Resource
    
    for i := 0; i < 6; i++ {
        resource, err := pool.Get()
        if err != nil {
            fmt.Printf("Failed to get resource %d: %v\n", i+1, err)
            break
        }
        resources = append(resources, resource)
        fmt.Printf("Acquired resource %d: %s\n", i+1, resource.GetID())
    }
    
    // Return all resources
    for _, resource := range resources {
        pool.Put(resource)
    }
    
    fmt.Println("\n=== Final Pool Stats ===")
    stats = service.GetPoolStats()
    fmt.Printf("Total: %d, Active: %d, Idle: %d, Created: %d, Reclaimed: %d\n",
               stats.TotalResources, stats.ActiveResources, 
               stats.IdleResources, stats.CreatedResources, stats.ReclaimedResources)
}
```

Resource pooling provides efficient management of expensive resources like  
database connections. Interfaces enable different resource types and pool  
implementations while providing automatic lifecycle management and cleanup.  

## Interface for state machines

State machines use interfaces to define states and transitions,  
enabling clean modeling of stateful behavior.  

```go
package main

import (
    "fmt"
    "time"
)

// State interface
type State interface {
    GetName() string
    OnEnter(context *StateMachineContext) error
    OnExit(context *StateMachineContext) error
    Handle(event string, context *StateMachineContext) (State, error)
}

// State machine context
type StateMachineContext struct {
    Data       map[string]any
    StartTime  time.Time
    LastUpdate time.Time
    EventCount int
}

func NewStateMachineContext() *StateMachineContext {
    return &StateMachineContext{
        Data:       make(map[string]any),
        StartTime:  time.Now(),
        LastUpdate: time.Now(),
    }
}

// State machine
type StateMachine struct {
    currentState State
    context      *StateMachineContext
    history      []StateTransition
}

type StateTransition struct {
    FromState string
    ToState   string
    Event     string
    Timestamp time.Time
}

func NewStateMachine(initialState State, context *StateMachineContext) *StateMachine {
    sm := &StateMachine{
        currentState: initialState,
        context:      context,
        history:      make([]StateTransition, 0),
    }
    
    // Enter initial state
    initialState.OnEnter(context)
    
    return sm
}

func (sm *StateMachine) GetCurrentState() State {
    return sm.currentState
}

func (sm *StateMachine) GetContext() *StateMachineContext {
    return sm.context
}

func (sm *StateMachine) HandleEvent(event string) error {
    oldStateName := sm.currentState.GetName()
    
    newState, err := sm.currentState.Handle(event, sm.context)
    if err != nil {
        return fmt.Errorf("failed to handle event '%s' in state '%s': %v", 
                          event, oldStateName, err)
    }
    
    if newState != nil && newState.GetName() != oldStateName {
        // State transition
        err = sm.currentState.OnExit(sm.context)
        if err != nil {
            return fmt.Errorf("failed to exit state '%s': %v", oldStateName, err)
        }
        
        err = newState.OnEnter(sm.context)
        if err != nil {
            return fmt.Errorf("failed to enter state '%s': %v", newState.GetName(), err)
        }
        
        // Record transition
        transition := StateTransition{
            FromState: oldStateName,
            ToState:   newState.GetName(),
            Event:     event,
            Timestamp: time.Now(),
        }
        sm.history = append(sm.history, transition)
        
        sm.currentState = newState
        fmt.Printf("State transition: %s -> %s (event: %s)\n", 
                   oldStateName, newState.GetName(), event)
    }
    
    sm.context.EventCount++
    sm.context.LastUpdate = time.Now()
    
    return nil
}

func (sm *StateMachine) GetHistory() []StateTransition {
    return sm.history
}

// Order states
type PendingState struct{}

func (ps *PendingState) GetName() string {
    return "Pending"
}

func (ps *PendingState) OnEnter(context *StateMachineContext) error {
    fmt.Println("Order is now pending...")
    context.Data["status"] = "pending"
    context.Data["created_at"] = time.Now()
    return nil
}

func (ps *PendingState) OnExit(context *StateMachineContext) error {
    fmt.Println("Leaving pending state...")
    return nil
}

func (ps *PendingState) Handle(event string, context *StateMachineContext) (State, error) {
    switch event {
    case "pay":
        if context.Data["payment_method"] == nil {
            return nil, fmt.Errorf("payment method required")
        }
        return &PaidState{}, nil
    case "cancel":
        context.Data["cancel_reason"] = "cancelled by user"
        return &CancelledState{}, nil
    case "expire":
        context.Data["cancel_reason"] = "payment timeout"
        return &CancelledState{}, nil
    default:
        return nil, fmt.Errorf("invalid event '%s' for pending state", event)
    }
}

type PaidState struct{}

func (ps *PaidState) GetName() string {
    return "Paid"
}

func (ps *PaidState) OnEnter(context *StateMachineContext) error {
    fmt.Println("Payment received, processing order...")
    context.Data["status"] = "paid"
    context.Data["paid_at"] = time.Now()
    return nil
}

func (ps *PaidState) OnExit(context *StateMachineContext) error {
    fmt.Println("Order leaving paid state...")
    return nil
}

func (ps *PaidState) Handle(event string, context *StateMachineContext) (State, error) {
    switch event {
    case "process":
        return &ProcessingState{}, nil
    case "refund":
        context.Data["refund_reason"] = "refund requested"
        return &RefundedState{}, nil
    default:
        return nil, fmt.Errorf("invalid event '%s' for paid state", event)
    }
}

type ProcessingState struct{}

func (ps *ProcessingState) GetName() string {
    return "Processing"
}

func (ps *ProcessingState) OnEnter(context *StateMachineContext) error {
    fmt.Println("Processing order...")
    context.Data["status"] = "processing"
    context.Data["processing_started"] = time.Now()
    return nil
}

func (ps *ProcessingState) OnExit(context *StateMachineContext) error {
    fmt.Println("Order processing completed...")
    return nil
}

func (ps *ProcessingState) Handle(event string, context *StateMachineContext) (State, error) {
    switch event {
    case "ship":
        trackingNumber := fmt.Sprintf("TRK%d", time.Now().Unix())
        context.Data["tracking_number"] = trackingNumber
        return &ShippedState{}, nil
    case "fail":
        context.Data["failure_reason"] = context.Data["error"]
        return &FailedState{}, nil
    default:
        return nil, fmt.Errorf("invalid event '%s' for processing state", event)
    }
}

type ShippedState struct{}

func (ss *ShippedState) GetName() string {
    return "Shipped"
}

func (ss *ShippedState) OnEnter(context *StateMachineContext) error {
    trackingNumber := context.Data["tracking_number"]
    fmt.Printf("Order shipped with tracking number: %s\n", trackingNumber)
    context.Data["status"] = "shipped"
    context.Data["shipped_at"] = time.Now()
    return nil
}

func (ss *ShippedState) OnExit(context *StateMachineContext) error {
    fmt.Println("Order leaving shipped state...")
    return nil
}

func (ss *ShippedState) Handle(event string, context *StateMachineContext) (State, error) {
    switch event {
    case "deliver":
        return &DeliveredState{}, nil
    case "return":
        context.Data["return_reason"] = "damaged in transit"
        return &ReturnedState{}, nil
    default:
        return nil, fmt.Errorf("invalid event '%s' for shipped state", event)
    }
}

type DeliveredState struct{}

func (ds *DeliveredState) GetName() string {
    return "Delivered"
}

func (ds *DeliveredState) OnEnter(context *StateMachineContext) error {
    fmt.Println("Order delivered successfully!")
    context.Data["status"] = "delivered"
    context.Data["delivered_at"] = time.Now()
    return nil
}

func (ds *DeliveredState) OnExit(context *StateMachineContext) error {
    fmt.Println("Order leaving delivered state...")
    return nil
}

func (ds *DeliveredState) Handle(event string, context *StateMachineContext) (State, error) {
    switch event {
    case "return":
        context.Data["return_reason"] = "customer return"
        return &ReturnedState{}, nil
    default:
        return nil, fmt.Errorf("invalid event '%s' for delivered state", event)
    }
}

// Terminal states
type CancelledState struct{}

func (cs *CancelledState) GetName() string {
    return "Cancelled"
}

func (cs *CancelledState) OnEnter(context *StateMachineContext) error {
    reason := context.Data["cancel_reason"]
    fmt.Printf("Order cancelled: %s\n", reason)
    context.Data["status"] = "cancelled"
    context.Data["cancelled_at"] = time.Now()
    return nil
}

func (cs *CancelledState) OnExit(context *StateMachineContext) error {
    return fmt.Errorf("cannot exit cancelled state")
}

func (cs *CancelledState) Handle(event string, context *StateMachineContext) (State, error) {
    return nil, fmt.Errorf("order is cancelled, no further events allowed")
}

type RefundedState struct{}

func (rs *RefundedState) GetName() string {
    return "Refunded"
}

func (rs *RefundedState) OnEnter(context *StateMachineContext) error {
    reason := context.Data["refund_reason"]
    fmt.Printf("Order refunded: %s\n", reason)
    context.Data["status"] = "refunded"
    context.Data["refunded_at"] = time.Now()
    return nil
}

func (rs *RefundedState) OnExit(context *StateMachineContext) error {
    return fmt.Errorf("cannot exit refunded state")
}

func (rs *RefundedState) Handle(event string, context *StateMachineContext) (State, error) {
    return nil, fmt.Errorf("order is refunded, no further events allowed")
}

type FailedState struct{}

func (fs *FailedState) GetName() string {
    return "Failed"
}

func (fs *FailedState) OnEnter(context *StateMachineContext) error {
    reason := context.Data["failure_reason"]
    fmt.Printf("Order failed: %s\n", reason)
    context.Data["status"] = "failed"
    context.Data["failed_at"] = time.Now()
    return nil
}

func (fs *FailedState) OnExit(context *StateMachineContext) error {
    return fmt.Errorf("cannot exit failed state")
}

func (fs *FailedState) Handle(event string, context *StateMachineContext) (State, error) {
    return nil, fmt.Errorf("order has failed, no further events allowed")
}

type ReturnedState struct{}

func (rs *ReturnedState) GetName() string {
    return "Returned"
}

func (rs *ReturnedState) OnEnter(context *StateMachineContext) error {
    reason := context.Data["return_reason"]
    fmt.Printf("Order returned: %s\n", reason)
    context.Data["status"] = "returned"
    context.Data["returned_at"] = time.Now()
    return nil
}

func (rs *ReturnedState) OnExit(context *StateMachineContext) error {
    return fmt.Errorf("cannot exit returned state")
}

func (rs *ReturnedState) Handle(event string, context *StateMachineContext) (State, error) {
    return nil, fmt.Errorf("order is returned, no further events allowed")
}

func runOrderScenario(name string, events []string, context *StateMachineContext) {
    fmt.Printf("\n=== %s ===\n", name)
    
    // Create state machine with initial pending state
    sm := NewStateMachine(&PendingState{}, context)
    
    fmt.Printf("Initial state: %s\n", sm.GetCurrentState().GetName())
    
    // Process events
    for i, event := range events {
        fmt.Printf("\nEvent %d: %s\n", i+1, event)
        err := sm.HandleEvent(event)
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            break
        }
        fmt.Printf("Current state: %s\n", sm.GetCurrentState().GetName())
    }
    
    // Print final context
    fmt.Printf("\nFinal state: %s\n", sm.GetCurrentState().GetName())
    fmt.Printf("Total events processed: %d\n", sm.GetContext().EventCount)
    
    // Print transition history
    history := sm.GetHistory()
    if len(history) > 0 {
        fmt.Println("\nState transitions:")
        for i, transition := range history {
            fmt.Printf("  %d. %s -> %s (event: %s) at %s\n", 
                       i+1, transition.FromState, transition.ToState, 
                       transition.Event, transition.Timestamp.Format("15:04:05"))
        }
    }
}

func main() {
    // Scenario 1: Successful order
    context1 := NewStateMachineContext()
    context1.Data["payment_method"] = "credit_card"
    context1.Data["order_id"] = "ORD001"
    
    events1 := []string{"pay", "process", "ship", "deliver"}
    runOrderScenario("Successful Order", events1, context1)
    
    // Scenario 2: Cancelled order
    context2 := NewStateMachineContext()
    context2.Data["order_id"] = "ORD002"
    
    events2 := []string{"cancel"}
    runOrderScenario("Cancelled Order", events2, context2)
    
    // Scenario 3: Failed processing
    context3 := NewStateMachineContext()
    context3.Data["payment_method"] = "paypal"
    context3.Data["order_id"] = "ORD003"
    context3.Data["error"] = "inventory unavailable"
    
    events3 := []string{"pay", "process", "fail"}
    runOrderScenario("Failed Processing", events3, context3)
    
    // Scenario 4: Returned order
    context4 := NewStateMachineContext()
    context4.Data["payment_method"] = "debit_card"
    context4.Data["order_id"] = "ORD004"
    
    events4 := []string{"pay", "process", "ship", "deliver", "return"}
    runOrderScenario("Returned Order", events4, context4)
    
    // Scenario 5: Invalid event handling
    context5 := NewStateMachineContext()
    context5.Data["order_id"] = "ORD005"
    
    events5 := []string{"invalid_event", "pay"}
    runOrderScenario("Invalid Event Handling", events5, context5)
    
    // Scenario 6: Missing payment method
    context6 := NewStateMachineContext()
    context6.Data["order_id"] = "ORD006"
    
    events6 := []string{"pay"}
    runOrderScenario("Missing Payment Method", events6, context6)
}
```

State machines provide a clean way to model complex stateful behavior.  
Interfaces enable different state implementations while maintaining  
consistent transition logic and state management capabilities.  

## Interface for pub/sub messaging

Publish-subscribe patterns use interfaces to decouple message producers  
from consumers, enabling scalable event-driven architectures.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Message interface
type Message interface {
    GetTopic() string
    GetPayload() any
    GetTimestamp() time.Time
    GetID() string
    GetHeaders() map[string]string
}

// Subscriber interface
type Subscriber interface {
    GetID() string
    HandleMessage(message Message) error
    GetSubscriptions() []string
}

// Publisher interface
type Publisher interface {
    Publish(topic string, payload any) error
    PublishWithHeaders(topic string, payload any, headers map[string]string) error
}

// Message broker interface
type MessageBroker interface {
    Publisher
    Subscribe(topic string, subscriber Subscriber) error
    Unsubscribe(topic string, subscriberID string) error
    GetSubscribers(topic string) []Subscriber
    Close() error
}

// Default message implementation
type DefaultMessage struct {
    ID        string
    Topic     string
    Payload   any
    Timestamp time.Time
    Headers   map[string]string
}

func NewMessage(topic string, payload any) *DefaultMessage {
    return &DefaultMessage{
        ID:        fmt.Sprintf("msg_%d", time.Now().UnixNano()),
        Topic:     topic,
        Payload:   payload,
        Timestamp: time.Now(),
        Headers:   make(map[string]string),
    }
}

func (dm *DefaultMessage) GetTopic() string {
    return dm.Topic
}

func (dm *DefaultMessage) GetPayload() any {
    return dm.Payload
}

func (dm *DefaultMessage) GetTimestamp() time.Time {
    return dm.Timestamp
}

func (dm *DefaultMessage) GetID() string {
    return dm.ID
}

func (dm *DefaultMessage) GetHeaders() map[string]string {
    return dm.Headers
}

// In-memory message broker
type InMemoryBroker struct {
    subscribers map[string][]Subscriber // topic -> subscribers
    mutex       sync.RWMutex
    closed      bool
}

func NewInMemoryBroker() *InMemoryBroker {
    return &InMemoryBroker{
        subscribers: make(map[string][]Subscriber),
    }
}

func (imb *InMemoryBroker) Publish(topic string, payload any) error {
    return imb.PublishWithHeaders(topic, payload, nil)
}

func (imb *InMemoryBroker) PublishWithHeaders(topic string, payload any, headers map[string]string) error {
    imb.mutex.RLock()
    defer imb.mutex.RUnlock()
    
    if imb.closed {
        return fmt.Errorf("broker is closed")
    }
    
    message := NewMessage(topic, payload)
    if headers != nil {
        for k, v := range headers {
            message.Headers[k] = v
        }
    }
    
    subscribers, exists := imb.subscribers[topic]
    if !exists || len(subscribers) == 0 {
        fmt.Printf("No subscribers for topic '%s'\n", topic)
        return nil
    }
    
    fmt.Printf("Publishing message to topic '%s' for %d subscribers\n", 
               topic, len(subscribers))
    
    // Deliver message to all subscribers asynchronously
    var wg sync.WaitGroup
    for _, subscriber := range subscribers {
        wg.Add(1)
        go func(sub Subscriber) {
            defer wg.Done()
            err := sub.HandleMessage(message)
            if err != nil {
                fmt.Printf("Error delivering message to subscriber %s: %v\n", 
                           sub.GetID(), err)
            }
        }(subscriber)
    }
    
    wg.Wait()
    return nil
}

func (imb *InMemoryBroker) Subscribe(topic string, subscriber Subscriber) error {
    imb.mutex.Lock()
    defer imb.mutex.Unlock()
    
    if imb.closed {
        return fmt.Errorf("broker is closed")
    }
    
    // Check if subscriber is already subscribed
    if subscribers, exists := imb.subscribers[topic]; exists {
        for _, sub := range subscribers {
            if sub.GetID() == subscriber.GetID() {
                return fmt.Errorf("subscriber %s already subscribed to topic %s", 
                                  subscriber.GetID(), topic)
            }
        }
    }
    
    imb.subscribers[topic] = append(imb.subscribers[topic], subscriber)
    
    fmt.Printf("Subscriber %s subscribed to topic '%s'\n", 
               subscriber.GetID(), topic)
    
    return nil
}

func (imb *InMemoryBroker) Unsubscribe(topic string, subscriberID string) error {
    imb.mutex.Lock()
    defer imb.mutex.Unlock()
    
    subscribers, exists := imb.subscribers[topic]
    if !exists {
        return fmt.Errorf("topic '%s' not found", topic)
    }
    
    for i, subscriber := range subscribers {
        if subscriber.GetID() == subscriberID {
            // Remove subscriber
            imb.subscribers[topic] = append(subscribers[:i], subscribers[i+1:]...)
            fmt.Printf("Subscriber %s unsubscribed from topic '%s'\n", 
                       subscriberID, topic)
            return nil
        }
    }
    
    return fmt.Errorf("subscriber %s not found in topic %s", subscriberID, topic)
}

func (imb *InMemoryBroker) GetSubscribers(topic string) []Subscriber {
    imb.mutex.RLock()
    defer imb.mutex.RUnlock()
    
    if subscribers, exists := imb.subscribers[topic]; exists {
        // Return a copy
        result := make([]Subscriber, len(subscribers))
        copy(result, subscribers)
        return result
    }
    
    return []Subscriber{}
}

func (imb *InMemoryBroker) Close() error {
    imb.mutex.Lock()
    defer imb.mutex.Unlock()
    
    imb.closed = true
    imb.subscribers = make(map[string][]Subscriber)
    
    fmt.Println("Message broker closed")
    return nil
}

// Email notification subscriber
type EmailSubscriber struct {
    id          string
    email       string
    topics      []string
    processedMessages int
}

func NewEmailSubscriber(id, email string, topics []string) *EmailSubscriber {
    return &EmailSubscriber{
        id:     id,
        email:  email,
        topics: topics,
    }
}

func (es *EmailSubscriber) GetID() string {
    return es.id
}

func (es *EmailSubscriber) HandleMessage(message Message) error {
    es.processedMessages++
    
    payload := message.GetPayload()
    headers := message.GetHeaders()
    
    fmt.Printf("[EMAIL:%s] Sending email to %s about '%s': %v\n", 
               es.id, es.email, message.GetTopic(), payload)
    
    if priority, exists := headers["priority"]; exists {
        fmt.Printf("  Priority: %s\n", priority)
    }
    
    // Simulate email sending delay
    time.Sleep(50 * time.Millisecond)
    
    return nil
}

func (es *EmailSubscriber) GetSubscriptions() []string {
    return es.topics
}

func (es *EmailSubscriber) GetProcessedCount() int {
    return es.processedMessages
}

// SMS notification subscriber
type SMSSubscriber struct {
    id          string
    phoneNumber string
    topics      []string
    processedMessages int
}

func NewSMSSubscriber(id, phoneNumber string, topics []string) *SMSSubscriber {
    return &SMSSubscriber{
        id:          id,
        phoneNumber: phoneNumber,
        topics:      topics,
    }
}

func (ss *SMSSubscriber) GetID() string {
    return ss.id
}

func (ss *SMSSubscriber) HandleMessage(message Message) error {
    ss.processedMessages++
    
    payload := message.GetPayload()
    
    // SMS has character limit, so truncate long messages
    messageText := fmt.Sprintf("%v", payload)
    if len(messageText) > 160 {
        messageText = messageText[:157] + "..."
    }
    
    fmt.Printf("[SMS:%s] Sending SMS to %s: %s\n", 
               ss.id, ss.phoneNumber, messageText)
    
    // Simulate SMS sending delay
    time.Sleep(30 * time.Millisecond)
    
    return nil
}

func (ss *SMSSubscriber) GetSubscriptions() []string {
    return ss.topics
}

func (ss *SMSSubscriber) GetProcessedCount() int {
    return ss.processedMessages
}

// Logging subscriber
type LoggingSubscriber struct {
    id          string
    logLevel    string
    topics      []string
    processedMessages int
    logFile     string
}

func NewLoggingSubscriber(id, logLevel string, topics []string) *LoggingSubscriber {
    return &LoggingSubscriber{
        id:       id,
        logLevel: logLevel,
        topics:   topics,
        logFile:  fmt.Sprintf("/var/log/%s.log", id),
    }
}

func (ls *LoggingSubscriber) GetID() string {
    return ls.id
}

func (ls *LoggingSubscriber) HandleMessage(message Message) error {
    ls.processedMessages++
    
    timestamp := message.GetTimestamp().Format("2006-01-02 15:04:05")
    payload := message.GetPayload()
    
    logEntry := fmt.Sprintf("[%s] %s - Topic: %s, Message: %v", 
                           timestamp, ls.logLevel, message.GetTopic(), payload)
    
    fmt.Printf("[LOG:%s] Writing to %s: %s\n", ls.id, ls.logFile, logEntry)
    
    return nil
}

func (ls *LoggingSubscriber) GetSubscriptions() []string {
    return ls.topics
}

func (ls *LoggingSubscriber) GetProcessedCount() int {
    return ls.processedMessages
}

// Analytics subscriber
type AnalyticsSubscriber struct {
    id          string
    topics      []string
    eventCounts map[string]int
    processedMessages int
    mutex       sync.Mutex
}

func NewAnalyticsSubscriber(id string, topics []string) *AnalyticsSubscriber {
    return &AnalyticsSubscriber{
        id:          id,
        topics:      topics,
        eventCounts: make(map[string]int),
    }
}

func (as *AnalyticsSubscriber) GetID() string {
    return as.id
}

func (as *AnalyticsSubscriber) HandleMessage(message Message) error {
    as.mutex.Lock()
    defer as.mutex.Unlock()
    
    as.processedMessages++
    topic := message.GetTopic()
    as.eventCounts[topic]++
    
    fmt.Printf("[ANALYTICS:%s] Recorded event for topic '%s' (total: %d)\n", 
               as.id, topic, as.eventCounts[topic])
    
    return nil
}

func (as *AnalyticsSubscriber) GetSubscriptions() []string {
    return as.topics
}

func (as *AnalyticsSubscriber) GetProcessedCount() int {
    as.mutex.Lock()
    defer as.mutex.Unlock()
    return as.processedMessages
}

func (as *AnalyticsSubscriber) GetEventCounts() map[string]int {
    as.mutex.Lock()
    defer as.mutex.Unlock()
    
    result := make(map[string]int)
    for k, v := range as.eventCounts {
        result[k] = v
    }
    return result
}

// Event publisher service
type EventPublisher struct {
    broker MessageBroker
    name   string
}

func NewEventPublisher(broker MessageBroker, name string) *EventPublisher {
    return &EventPublisher{
        broker: broker,
        name:   name,
    }
}

func (ep *EventPublisher) PublishUserRegistration(userID, email string) error {
    payload := map[string]any{
        "user_id": userID,
        "email":   email,
        "action":  "registration",
    }
    
    headers := map[string]string{
        "priority": "high",
        "source":   ep.name,
    }
    
    return ep.broker.PublishWithHeaders("user.registered", payload, headers)
}

func (ep *EventPublisher) PublishOrderPlaced(orderID string, amount float64, customerID string) error {
    payload := map[string]any{
        "order_id":    orderID,
        "amount":      amount,
        "customer_id": customerID,
        "action":      "order_placed",
    }
    
    headers := map[string]string{
        "priority": "medium",
        "source":   ep.name,
    }
    
    return ep.broker.PublishWithHeaders("order.placed", payload, headers)
}

func (ep *EventPublisher) PublishPaymentProcessed(paymentID, orderID string, amount float64) error {
    payload := map[string]any{
        "payment_id": paymentID,
        "order_id":   orderID,
        "amount":     amount,
        "action":     "payment_processed",
    }
    
    return ep.broker.Publish("payment.processed", payload)
}

func (ep *EventPublisher) PublishSystemAlert(level, message string) error {
    payload := map[string]any{
        "level":   level,
        "message": message,
        "source":  ep.name,
    }
    
    headers := map[string]string{
        "priority": "critical",
        "alert":    "true",
    }
    
    return ep.broker.PublishWithHeaders("system.alert", payload, headers)
}

func main() {
    // Create broker
    broker := NewInMemoryBroker()
    defer broker.Close()
    
    // Create subscribers
    emailSub := NewEmailSubscriber("email1", "admin@company.com", 
                                   []string{"user.registered", "system.alert"})
    smsSub := NewSMSSubscriber("sms1", "+1234567890", 
                               []string{"system.alert", "order.placed"})
    logSub := NewLoggingSubscriber("logger1", "INFO", 
                                   []string{"user.registered", "order.placed", "payment.processed"})
    analyticsSub := NewAnalyticsSubscriber("analytics1", 
                                           []string{"user.registered", "order.placed", "payment.processed"})
    
    // Subscribe to topics
    subscribers := []struct {
        subscriber Subscriber
        topics     []string
    }{
        {emailSub, emailSub.GetSubscriptions()},
        {smsSub, smsSub.GetSubscriptions()},
        {logSub, logSub.GetSubscriptions()},
        {analyticsSub, analyticsSub.GetSubscriptions()},
    }
    
    fmt.Println("=== Setting up subscriptions ===")
    for _, sub := range subscribers {
        for _, topic := range sub.topics {
            err := broker.Subscribe(topic, sub.subscriber)
            if err != nil {
                fmt.Printf("Failed to subscribe %s to %s: %v\n", 
                           sub.subscriber.GetID(), topic, err)
            }
        }
    }
    
    // Create publisher
    publisher := NewEventPublisher(broker, "main-service")
    
    fmt.Println("\n=== Publishing events ===")
    
    // Publish some events
    events := []func() error{
        func() error {
            return publisher.PublishUserRegistration("user123", "user123@example.com")
        },
        func() error {
            return publisher.PublishOrderPlaced("order456", 99.99, "user123")
        },
        func() error {
            return publisher.PublishPaymentProcessed("pay789", "order456", 99.99)
        },
        func() error {
            return publisher.PublishSystemAlert("ERROR", "Database connection lost")
        },
        func() error {
            return publisher.PublishUserRegistration("user124", "user124@example.com")
        },
    }
    
    for i, eventFunc := range events {
        fmt.Printf("\n--- Publishing event %d ---\n", i+1)
        err := eventFunc()
        if err != nil {
            fmt.Printf("Failed to publish event: %v\n", err)
        }
        time.Sleep(200 * time.Millisecond) // Small delay between events
    }
    
    // Print statistics
    fmt.Println("\n=== Statistics ===")
    fmt.Printf("Email subscriber processed: %d messages\n", emailSub.GetProcessedCount())
    fmt.Printf("SMS subscriber processed: %d messages\n", smsSub.GetProcessedCount())
    fmt.Printf("Logging subscriber processed: %d messages\n", logSub.GetProcessedCount())
    fmt.Printf("Analytics subscriber processed: %d messages\n", analyticsSub.GetProcessedCount())
    
    fmt.Println("\n=== Analytics Event Counts ===")
    eventCounts := analyticsSub.GetEventCounts()
    for topic, count := range eventCounts {
        fmt.Printf("Topic '%s': %d events\n", topic, count)
    }
    
    // Test unsubscribe
    fmt.Println("\n=== Testing unsubscribe ===")
    err := broker.Unsubscribe("system.alert", "email1")
    if err != nil {
        fmt.Printf("Failed to unsubscribe: %v\n", err)
    }
    
    // Publish another system alert
    fmt.Println("\n--- Publishing system alert after unsubscribe ---")
    publisher.PublishSystemAlert("WARNING", "High memory usage detected")
    
    fmt.Println("\n=== Final Statistics ===")
    fmt.Printf("Email subscriber processed: %d messages\n", emailSub.GetProcessedCount())
    fmt.Printf("SMS subscriber processed: %d messages\n", smsSub.GetProcessedCount())
}
```

Publish-subscribe messaging enables decoupled, scalable event-driven  
architectures. Interfaces provide clean separation between publishers,  
subscribers, and message brokers, allowing easy extension and testing.  

## Interface for proxy pattern

The proxy pattern uses interfaces to provide a placeholder or surrogate  
for another object to control access to it.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Subject interface
type ImageService interface {
    LoadImage(filename string) ([]byte, error)
    GetImageInfo(filename string) (ImageInfo, error)
}

type ImageInfo struct {
    Filename string
    Width    int
    Height   int
    Format   string
    Size     int64
}

// Real subject - actual image service
type RealImageService struct {
    name string
}

func NewRealImageService() *RealImageService {
    return &RealImageService{name: "RealImageService"}
}

func (ris *RealImageService) LoadImage(filename string) ([]byte, error) {
    fmt.Printf("[%s] Loading image: %s\n", ris.name, filename)
    
    // Simulate expensive image loading operation
    time.Sleep(500 * time.Millisecond)
    
    // Mock image data
    imageData := []byte(fmt.Sprintf("Image data for %s", filename))
    
    fmt.Printf("[%s] Image loaded successfully: %s (%d bytes)\n", 
               ris.name, filename, len(imageData))
    
    return imageData, nil
}

func (ris *RealImageService) GetImageInfo(filename string) (ImageInfo, error) {
    fmt.Printf("[%s] Getting image info: %s\n", ris.name, filename)
    
    // Simulate metadata retrieval
    time.Sleep(100 * time.Millisecond)
    
    info := ImageInfo{
        Filename: filename,
        Width:    1920,
        Height:   1080,
        Format:   "JPEG",
        Size:     2048576, // 2MB
    }
    
    return info, nil
}

// Caching proxy
type CachingImageProxy struct {
    realService ImageService
    imageCache  map[string][]byte
    infoCache   map[string]ImageInfo
    mutex       sync.RWMutex
    cacheHits   int64
    cacheMisses int64
}

func NewCachingImageProxy(realService ImageService) *CachingImageProxy {
    return &CachingImageProxy{
        realService: realService,
        imageCache:  make(map[string][]byte),
        infoCache:   make(map[string]ImageInfo),
    }
}

func (cip *CachingImageProxy) LoadImage(filename string) ([]byte, error) {
    cip.mutex.RLock()
    if data, exists := cip.imageCache[filename]; exists {
        cip.cacheHits++
        cip.mutex.RUnlock()
        fmt.Printf("[CachingProxy] Cache HIT for image: %s\n", filename)
        return data, nil
    }
    cip.cacheMisses++
    cip.mutex.RUnlock()
    
    fmt.Printf("[CachingProxy] Cache MISS for image: %s\n", filename)
    
    // Load from real service
    data, err := cip.realService.LoadImage(filename)
    if err != nil {
        return nil, err
    }
    
    // Cache the result
    cip.mutex.Lock()
    cip.imageCache[filename] = data
    cip.mutex.Unlock()
    
    fmt.Printf("[CachingProxy] Cached image: %s\n", filename)
    return data, nil
}

func (cip *CachingImageProxy) GetImageInfo(filename string) (ImageInfo, error) {
    cip.mutex.RLock()
    if info, exists := cip.infoCache[filename]; exists {
        cip.cacheHits++
        cip.mutex.RUnlock()
        fmt.Printf("[CachingProxy] Cache HIT for info: %s\n", filename)
        return info, nil
    }
    cip.cacheMisses++
    cip.mutex.RUnlock()
    
    fmt.Printf("[CachingProxy] Cache MISS for info: %s\n", filename)
    
    // Get from real service
    info, err := cip.realService.GetImageInfo(filename)
    if err != nil {
        return ImageInfo{}, err
    }
    
    // Cache the result
    cip.mutex.Lock()
    cip.infoCache[filename] = info
    cip.mutex.Unlock()
    
    fmt.Printf("[CachingProxy] Cached info: %s\n", filename)
    return info, nil
}

func (cip *CachingImageProxy) GetCacheStats() (int64, int64) {
    cip.mutex.RLock()
    defer cip.mutex.RUnlock()
    return cip.cacheHits, cip.cacheMisses
}

// Access control proxy
type AccessControlImageProxy struct {
    realService   ImageService
    authorizedUsers map[string]bool
    currentUser   string
    mutex         sync.RWMutex
}

func NewAccessControlImageProxy(realService ImageService, authorizedUsers []string) *AccessControlImageProxy {
    authMap := make(map[string]bool)
    for _, user := range authorizedUsers {
        authMap[user] = true
    }
    
    return &AccessControlImageProxy{
        realService:     realService,
        authorizedUsers: authMap,
    }
}

func (acip *AccessControlImageProxy) SetCurrentUser(user string) {
    acip.mutex.Lock()
    defer acip.mutex.Unlock()
    acip.currentUser = user
}

func (acip *AccessControlImageProxy) isAuthorized() bool {
    acip.mutex.RLock()
    defer acip.mutex.RUnlock()
    return acip.authorizedUsers[acip.currentUser]
}

func (acip *AccessControlImageProxy) LoadImage(filename string) ([]byte, error) {
    if !acip.isAuthorized() {
        return nil, fmt.Errorf("access denied for user: %s", acip.currentUser)
    }
    
    fmt.Printf("[AccessControlProxy] Access granted for user: %s\n", acip.currentUser)
    return acip.realService.LoadImage(filename)
}

func (acip *AccessControlImageProxy) GetImageInfo(filename string) (ImageInfo, error) {
    if !acip.isAuthorized() {
        return ImageInfo{}, fmt.Errorf("access denied for user: %s", acip.currentUser)
    }
    
    fmt.Printf("[AccessControlProxy] Access granted for user: %s\n", acip.currentUser)
    return acip.realService.GetImageInfo(filename)
}

// Logging proxy
type LoggingImageProxy struct {
    realService ImageService
    requestLog  []LogEntry
    mutex       sync.Mutex
}

type LogEntry struct {
    Operation string
    Filename  string
    User      string
    Timestamp time.Time
    Duration  time.Duration
    Success   bool
    Error     string
}

func NewLoggingImageProxy(realService ImageService) *LoggingImageProxy {
    return &LoggingImageProxy{
        realService: realService,
        requestLog:  make([]LogEntry, 0),
    }
}

func (lip *LoggingImageProxy) LoadImage(filename string) ([]byte, error) {
    start := time.Now()
    
    fmt.Printf("[LoggingProxy] Starting LoadImage: %s\n", filename)
    
    data, err := lip.realService.LoadImage(filename)
    duration := time.Since(start)
    
    entry := LogEntry{
        Operation: "LoadImage",
        Filename:  filename,
        Timestamp: start,
        Duration:  duration,
        Success:   err == nil,
    }
    
    if err != nil {
        entry.Error = err.Error()
    }
    
    lip.mutex.Lock()
    lip.requestLog = append(lip.requestLog, entry)
    lip.mutex.Unlock()
    
    fmt.Printf("[LoggingProxy] Completed LoadImage: %s (duration: %v, success: %t)\n", 
               filename, duration, err == nil)
    
    return data, err
}

func (lip *LoggingImageProxy) GetImageInfo(filename string) (ImageInfo, error) {
    start := time.Now()
    
    fmt.Printf("[LoggingProxy] Starting GetImageInfo: %s\n", filename)
    
    info, err := lip.realService.GetImageInfo(filename)
    duration := time.Since(start)
    
    entry := LogEntry{
        Operation: "GetImageInfo",
        Filename:  filename,
        Timestamp: start,
        Duration:  duration,
        Success:   err == nil,
    }
    
    if err != nil {
        entry.Error = err.Error()
    }
    
    lip.mutex.Lock()
    lip.requestLog = append(lip.requestLog, entry)
    lip.mutex.Unlock()
    
    fmt.Printf("[LoggingProxy] Completed GetImageInfo: %s (duration: %v, success: %t)\n", 
               filename, duration, err == nil)
    
    return info, err
}

func (lip *LoggingImageProxy) GetRequestLog() []LogEntry {
    lip.mutex.Lock()
    defer lip.mutex.Unlock()
    
    // Return a copy
    result := make([]LogEntry, len(lip.requestLog))
    copy(result, lip.requestLog)
    return result
}

// Chain multiple proxies
func createProxyChain() ImageService {
    // Real service
    realService := NewRealImageService()
    
    // Add logging
    loggingProxy := NewLoggingImageProxy(realService)
    
    // Add caching
    cachingProxy := NewCachingImageProxy(loggingProxy)
    
    // Add access control
    accessProxy := NewAccessControlImageProxy(cachingProxy, []string{"admin", "user1", "user2"})
    
    return accessProxy
}

func testImageService(service ImageService, user string) {
    fmt.Printf("\n=== Testing with user: %s ===\n", user)
    
    // Set user if it's an access control proxy
    if accessProxy, ok := service.(*AccessControlImageProxy); ok {
        accessProxy.SetCurrentUser(user)
    }
    
    images := []string{"photo1.jpg", "logo.png", "photo1.jpg"} // Duplicate to test caching
    
    for _, filename := range images {
        fmt.Printf("\n--- Processing: %s ---\n", filename)
        
        // Get image info
        info, err := service.GetImageInfo(filename)
        if err != nil {
            fmt.Printf("Error getting info: %v\n", err)
            continue
        }
        
        fmt.Printf("Image info: %dx%d %s (%d bytes)\n", 
                   info.Width, info.Height, info.Format, info.Size)
        
        // Load image
        data, err := service.LoadImage(filename)
        if err != nil {
            fmt.Printf("Error loading image: %v\n", err)
            continue
        }
        
        fmt.Printf("Loaded %d bytes of image data\n", len(data))
    }
}

func main() {
    // Create proxy chain
    service := createProxyChain()
    
    // Test with authorized user
    testImageService(service, "admin")
    
    // Test with another authorized user
    testImageService(service, "user1")
    
    // Test with unauthorized user
    testImageService(service, "unauthorized")
    
    // Extract proxies from chain to show stats
    fmt.Println("\n=== Proxy Statistics ===")
    
    if accessProxy, ok := service.(*AccessControlImageProxy); ok {
        if cachingProxy, ok := accessProxy.realService.(*CachingImageProxy); ok {
            hits, misses := cachingProxy.GetCacheStats()
            fmt.Printf("Cache Statistics - Hits: %d, Misses: %d\n", hits, misses)
            
            if loggingProxy, ok := cachingProxy.realService.(*LoggingImageProxy); ok {
                requestLog := loggingProxy.GetRequestLog()
                fmt.Printf("\nRequest Log (%d entries):\n", len(requestLog))
                
                for i, entry := range requestLog {
                    status := "SUCCESS"
                    if !entry.Success {
                        status = "FAILED: " + entry.Error
                    }
                    
                    fmt.Printf("%d. %s %s - %s (duration: %v) - %s\n",
                               i+1, entry.Timestamp.Format("15:04:05"), 
                               entry.Operation, entry.Filename, 
                               entry.Duration, status)
                }
                
                // Calculate average duration
                if len(requestLog) > 0 {
                    var totalDuration time.Duration
                    successCount := 0
                    
                    for _, entry := range requestLog {
                        if entry.Success {
                            totalDuration += entry.Duration
                            successCount++
                        }
                    }
                    
                    if successCount > 0 {
                        avgDuration := totalDuration / time.Duration(successCount)
                        fmt.Printf("\nAverage successful request duration: %v\n", avgDuration)
                    }
                }
            }
        }
    }
}
```

The proxy pattern provides controlled access to objects through interfaces.  
Different proxy types can be chained together to add multiple concerns  
like caching, logging, access control, and performance monitoring.  

## Interface for chain of responsibility

The chain of responsibility pattern uses interfaces to pass requests along  
a chain of handlers until one handles it.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
    "time"
)

// Handler interface
type RequestHandler interface {
    SetNext(handler RequestHandler) RequestHandler
    Handle(request Request) error
}

// Request types
type Request struct {
    ID          string
    Type        string
    Data        map[string]any
    Timestamp   time.Time
    ProcessedBy []string
}

func NewRequest(requestType string, data map[string]any) *Request {
    return &Request{
        ID:          fmt.Sprintf("req_%d", time.Now().UnixNano()),
        Type:        requestType,
        Data:        data,
        Timestamp:   time.Now(),
        ProcessedBy: make([]string, 0),
    }
}

func (r *Request) AddProcessor(processor string) {
    r.ProcessedBy = append(r.ProcessedBy, processor)
}

// Base handler
type BaseHandler struct {
    nextHandler RequestHandler
}

func (bh *BaseHandler) SetNext(handler RequestHandler) RequestHandler {
    bh.nextHandler = handler
    return handler
}

func (bh *BaseHandler) HandleNext(request Request) error {
    if bh.nextHandler != nil {
        return bh.nextHandler.Handle(request)
    }
    return nil
}

// Authentication handler
type AuthenticationHandler struct {
    BaseHandler
    validTokens map[string]string
}

func NewAuthenticationHandler() *AuthenticationHandler {
    return &AuthenticationHandler{
        validTokens: map[string]string{
            "token123": "user1",
            "token456": "user2",
            "admin789": "admin",
        },
    }
}

func (ah *AuthenticationHandler) Handle(request Request) error {
    fmt.Printf("[AUTH] Processing request %s\n", request.ID)
    
    token, exists := request.Data["auth_token"]
    if !exists {
        return fmt.Errorf("authentication failed: missing auth token")
    }
    
    tokenStr, ok := token.(string)
    if !ok {
        return fmt.Errorf("authentication failed: invalid token format")
    }
    
    user, valid := ah.validTokens[tokenStr]
    if !valid {
        return fmt.Errorf("authentication failed: invalid token")
    }
    
    // Add user info to request
    request.Data["authenticated_user"] = user
    request.AddProcessor("AuthenticationHandler")
    
    fmt.Printf("[AUTH] Request %s authenticated for user: %s\n", request.ID, user)
    
    // Pass to next handler
    return ah.HandleNext(request)
}

// Authorization handler
type AuthorizationHandler struct {
    BaseHandler
    permissions map[string][]string // user -> allowed request types
}

func NewAuthorizationHandler() *AuthorizationHandler {
    return &AuthorizationHandler{
        permissions: map[string][]string{
            "user1": {"read", "create"},
            "user2": {"read"},
            "admin": {"read", "create", "update", "delete"},
        },
    }
}

func (ah *AuthorizationHandler) Handle(request Request) error {
    fmt.Printf("[AUTHZ] Processing request %s\n", request.ID)
    
    user, exists := request.Data["authenticated_user"]
    if !exists {
        return fmt.Errorf("authorization failed: user not authenticated")
    }
    
    userStr, ok := user.(string)
    if !ok {
        return fmt.Errorf("authorization failed: invalid user format")
    }
    
    userPermissions, hasPerms := ah.permissions[userStr]
    if !hasPerms {
        return fmt.Errorf("authorization failed: user %s has no permissions", userStr)
    }
    
    // Check if user can perform this request type
    allowed := false
    for _, permission := range userPermissions {
        if permission == request.Type {
            allowed = true
            break
        }
    }
    
    if !allowed {
        return fmt.Errorf("authorization failed: user %s cannot perform %s", 
                          userStr, request.Type)
    }
    
    request.AddProcessor("AuthorizationHandler")
    fmt.Printf("[AUTHZ] Request %s authorized for user: %s\n", request.ID, userStr)
    
    // Pass to next handler
    return ah.HandleNext(request)
}

// Rate limiting handler
type RateLimitingHandler struct {
    BaseHandler
    requestCounts map[string][]time.Time // user -> request timestamps
    maxRequests   int
    timeWindow    time.Duration
}

func NewRateLimitingHandler(maxRequests int, timeWindow time.Duration) *RateLimitingHandler {
    return &RateLimitingHandler{
        requestCounts: make(map[string][]time.Time),
        maxRequests:   maxRequests,
        timeWindow:    timeWindow,
    }
}

func (rlh *RateLimitingHandler) Handle(request Request) error {
    fmt.Printf("[RATE] Processing request %s\n", request.ID)
    
    user, exists := request.Data["authenticated_user"]
    if !exists {
        return fmt.Errorf("rate limiting failed: user not identified")
    }
    
    userStr := user.(string)
    now := time.Now()
    
    // Clean old timestamps
    if timestamps, exists := rlh.requestCounts[userStr]; exists {
        var validTimestamps []time.Time
        for _, timestamp := range timestamps {
            if now.Sub(timestamp) <= rlh.timeWindow {
                validTimestamps = append(validTimestamps, timestamp)
            }
        }
        rlh.requestCounts[userStr] = validTimestamps
    }
    
    // Check rate limit
    currentCount := len(rlh.requestCounts[userStr])
    if currentCount >= rlh.maxRequests {
        return fmt.Errorf("rate limit exceeded: user %s has made %d requests in the last %v", 
                          userStr, currentCount, rlh.timeWindow)
    }
    
    // Add current request
    rlh.requestCounts[userStr] = append(rlh.requestCounts[userStr], now)
    
    request.AddProcessor("RateLimitingHandler")
    fmt.Printf("[RATE] Request %s passed rate limiting (%d/%d)\n", 
               request.ID, currentCount+1, rlh.maxRequests)
    
    // Pass to next handler
    return ah.HandleNext(request)
}

// Validation handler
type ValidationHandler struct {
    BaseHandler
}

func NewValidationHandler() *ValidationHandler {
    return &ValidationHandler{}
}

func (vh *ValidationHandler) Handle(request Request) error {
    fmt.Printf("[VALID] Processing request %s\n", request.ID)
    
    // Validate based on request type
    switch request.Type {
    case "create":
        return vh.validateCreate(request)
    case "read":
        return vh.validateRead(request)
    case "update":
        return vh.validateUpdate(request)
    case "delete":
        return vh.validateDelete(request)
    default:
        return fmt.Errorf("validation failed: unknown request type %s", request.Type)
    }
}

func (vh *ValidationHandler) validateCreate(request Request) error {
    required := []string{"name", "email"}
    for _, field := range required {
        if _, exists := request.Data[field]; !exists {
            return fmt.Errorf("validation failed: missing required field '%s'", field)
        }
    }
    
    // Validate email format (simple check)
    email := request.Data["email"].(string)
    if !strings.Contains(email, "@") {
        return fmt.Errorf("validation failed: invalid email format")
    }
    
    request.AddProcessor("ValidationHandler")
    fmt.Printf("[VALID] Create request %s validation passed\n", request.ID)
    return vh.HandleNext(request)
}

func (vh *ValidationHandler) validateRead(request Request) error {
    if _, exists := request.Data["id"]; !exists {
        return fmt.Errorf("validation failed: missing required field 'id'")
    }
    
    request.AddProcessor("ValidationHandler")
    fmt.Printf("[VALID] Read request %s validation passed\n", request.ID)
    return vh.HandleNext(request)
}

func (vh *ValidationHandler) validateUpdate(request Request) error {
    if _, exists := request.Data["id"]; !exists {
        return fmt.Errorf("validation failed: missing required field 'id'")
    }
    
    // At least one field to update should be present
    updateFields := []string{"name", "email", "age"}
    hasUpdate := false
    for _, field := range updateFields {
        if _, exists := request.Data[field]; exists {
            hasUpdate = true
            break
        }
    }
    
    if !hasUpdate {
        return fmt.Errorf("validation failed: no fields to update")
    }
    
    request.AddProcessor("ValidationHandler")
    fmt.Printf("[VALID] Update request %s validation passed\n", request.ID)
    return vh.HandleNext(request)
}

func (vh *ValidationHandler) validateDelete(request Request) error {
    if _, exists := request.Data["id"]; !exists {
        return fmt.Errorf("validation failed: missing required field 'id'")
    }
    
    request.AddProcessor("ValidationHandler")
    fmt.Printf("[VALID] Delete request %s validation passed\n", request.ID)
    return vh.HandleNext(request)
}

// Business logic handler (final handler)
type BusinessLogicHandler struct {
    BaseHandler
}

func NewBusinessLogicHandler() *BusinessLogicHandler {
    return &BusinessLogicHandler{}
}

func (blh *BusinessLogicHandler) Handle(request Request) error {
    fmt.Printf("[BUSINESS] Processing request %s\n", request.ID)
    
    user := request.Data["authenticated_user"].(string)
    
    switch request.Type {
    case "create":
        return blh.handleCreate(request, user)
    case "read":
        return blh.handleRead(request, user)
    case "update":
        return blh.handleUpdate(request, user)
    case "delete":
        return blh.handleDelete(request, user)
    default:
        return fmt.Errorf("business logic error: unsupported operation %s", request.Type)
    }
}

func (blh *BusinessLogicHandler) handleCreate(request Request, user string) error {
    name := request.Data["name"].(string)
    email := request.Data["email"].(string)
    
    fmt.Printf("[BUSINESS] Creating user: name=%s, email=%s (by %s)\n", 
               name, email, user)
    
    // Simulate business logic
    time.Sleep(100 * time.Millisecond)
    
    request.Data["result"] = map[string]any{
        "id":      123,
        "message": "User created successfully",
    }
    
    request.AddProcessor("BusinessLogicHandler")
    fmt.Printf("[BUSINESS] User created successfully with ID: 123\n")
    
    return nil
}

func (blh *BusinessLogicHandler) handleRead(request Request, user string) error {
    id := request.Data["id"]
    
    fmt.Printf("[BUSINESS] Reading user with ID: %v (by %s)\n", id, user)
    
    // Simulate business logic
    time.Sleep(50 * time.Millisecond)
    
    request.Data["result"] = map[string]any{
        "id":    id,
        "name":  "John Doe",
        "email": "john@example.com",
        "age":   30,
    }
    
    request.AddProcessor("BusinessLogicHandler")
    fmt.Printf("[BUSINESS] User data retrieved successfully\n")
    
    return nil
}

func (blh *BusinessLogicHandler) handleUpdate(request Request, user string) error {
    id := request.Data["id"]
    
    fmt.Printf("[BUSINESS] Updating user with ID: %v (by %s)\n", id, user)
    
    // Simulate business logic
    time.Sleep(80 * time.Millisecond)
    
    request.Data["result"] = map[string]any{
        "id":      id,
        "message": "User updated successfully",
    }
    
    request.AddProcessor("BusinessLogicHandler")
    fmt.Printf("[BUSINESS] User updated successfully\n")
    
    return nil
}

func (blh *BusinessLogicHandler) handleDelete(request Request, user string) error {
    id := request.Data["id"]
    
    fmt.Printf("[BUSINESS] Deleting user with ID: %v (by %s)\n", id, user)
    
    // Simulate business logic
    time.Sleep(120 * time.Millisecond)
    
    request.Data["result"] = map[string]any{
        "id":      id,
        "message": "User deleted successfully",
    }
    
    request.AddProcessor("BusinessLogicHandler")
    fmt.Printf("[BUSINESS] User deleted successfully\n")
    
    return nil
}

// Request processor that uses the chain
type RequestProcessor struct {
    handlerChain RequestHandler
}

func NewRequestProcessor() *RequestProcessor {
    // Build the chain
    auth := NewAuthenticationHandler()
    authz := NewAuthorizationHandler()
    rateLimit := NewRateLimitingHandler(5, time.Minute) // 5 requests per minute
    validation := NewValidationHandler()
    business := NewBusinessLogicHandler()
    
    // Chain handlers: auth -> authz -> rate limit -> validation -> business
    auth.SetNext(authz).SetNext(rateLimit).SetNext(validation).SetNext(business)
    
    return &RequestProcessor{handlerChain: auth}
}

func (rp *RequestProcessor) ProcessRequest(request *Request) error {
    start := time.Now()
    
    fmt.Printf("\n=== Processing Request %s ===\n", request.ID)
    fmt.Printf("Type: %s, Timestamp: %s\n", 
               request.Type, request.Timestamp.Format("15:04:05"))
    
    err := rp.handlerChain.Handle(*request)
    
    duration := time.Since(start)
    
    if err != nil {
        fmt.Printf("❌ Request %s FAILED: %v (duration: %v)\n", 
                   request.ID, err, duration)
        return err
    }
    
    fmt.Printf("✅ Request %s SUCCESS (duration: %v)\n", request.ID, duration)
    fmt.Printf("Processed by: %v\n", request.ProcessedBy)
    
    if result, exists := request.Data["result"]; exists {
        fmt.Printf("Result: %v\n", result)
    }
    
    return nil
}

func main() {
    processor := NewRequestProcessor()
    
    // Test cases
    testCases := []struct {
        name    string
        reqType string
        data    map[string]any
    }{
        {
            name:    "Valid Create Request",
            reqType: "create",
            data: map[string]any{
                "auth_token": "token123",
                "name":       "Alice Johnson",
                "email":      "alice@example.com",
            },
        },
        {
            name:    "Valid Read Request",
            reqType: "read",
            data: map[string]any{
                "auth_token": "admin789",
                "id":         "123",
            },
        },
        {
            name:    "Unauthorized Request",
            reqType: "delete",
            data: map[string]any{
                "auth_token": "token456", // user2 can only read
                "id":         "123",
            },
        },
        {
            name:    "Invalid Token",
            reqType: "read",
            data: map[string]any{
                "auth_token": "invalid_token",
                "id":         "123",
            },
        },
        {
            name:    "Missing Authentication",
            reqType: "create",
            data: map[string]any{
                "name":  "Bob Smith",
                "email": "bob@example.com",
            },
        },
        {
            name:    "Validation Failed",
            reqType: "create",
            data: map[string]any{
                "auth_token": "admin789",
                "name":       "Charlie Brown",
                // missing email
            },
        },
        {
            name:    "Valid Update Request",
            reqType: "update",
            data: map[string]any{
                "auth_token": "admin789",
                "id":         "123",
                "name":       "Updated Name",
            },
        },
    }
    
    // Process all test cases
    for i, testCase := range testCases {
        fmt.Printf("\n" + strings.Repeat("=", 50))
        fmt.Printf("\nTest Case %d: %s\n", i+1, testCase.name)
        fmt.Printf(strings.Repeat("=", 50))
        
        request := NewRequest(testCase.reqType, testCase.data)
        processor.ProcessRequest(request)
        
        time.Sleep(100 * time.Millisecond) // Small delay between requests
    }
    
    // Test rate limiting
    fmt.Printf("\n" + strings.Repeat("=", 50))
    fmt.Printf("\nTesting Rate Limiting")
    fmt.Printf("\n" + strings.Repeat("=", 50))
    
    for i := 0; i < 7; i++ { // Try 7 requests (limit is 5)
        request := NewRequest("read", map[string]any{
            "auth_token": "admin789",
            "id":         strconv.Itoa(i + 1),
        })
        
        fmt.Printf("\nRate limit test - request %d:\n", i+1)
        processor.ProcessRequest(request)
    }
}
```

The chain of responsibility pattern enables flexible request processing  
through a series of handlers. Each handler can process, modify, or reject  
requests, providing clean separation of concerns and easy extensibility.  

## Interface for visitor pattern

The visitor pattern uses interfaces to separate algorithms from the object  
structure they operate on, enabling new operations without changing classes.  

```go
package main

import (
    "fmt"
    "strings"
)

// Visitor interface
type Visitor interface {
    VisitTextElement(element *TextElement) any
    VisitImageElement(element *ImageElement) any
    VisitLinkElement(element *LinkElement) any
    VisitContainerElement(element *ContainerElement) any
}

// Element interface
type Element interface {
    Accept(visitor Visitor) any
}

// Concrete elements
type TextElement struct {
    Content string
    Style   map[string]string
}

func (te *TextElement) Accept(visitor Visitor) any {
    return visitor.VisitTextElement(te)
}

type ImageElement struct {
    Src    string
    Alt    string
    Width  int
    Height int
}

func (ie *ImageElement) Accept(visitor Visitor) any {
    return visitor.VisitImageElement(ie)
}

type LinkElement struct {
    URL  string
    Text string
}

func (le *LinkElement) Accept(visitor Visitor) any {
    return visitor.VisitLinkElement(le)
}

type ContainerElement struct {
    Tag      string
    Children []Element
    Attributes map[string]string
}

func (ce *ContainerElement) Accept(visitor Visitor) any {
    return visitor.VisitContainerElement(ce)
}

// HTML rendering visitor
type HTMLRenderVisitor struct{}

func (hrv *HTMLRenderVisitor) VisitTextElement(element *TextElement) any {
    html := element.Content
    
    if element.Style != nil {
        var styles []string
        for prop, value := range element.Style {
            styles = append(styles, fmt.Sprintf("%s: %s", prop, value))
        }
        if len(styles) > 0 {
            html = fmt.Sprintf(`<span style="%s">%s</span>`, 
                               strings.Join(styles, "; "), html)
        }
    }
    
    return html
}

func (hrv *HTMLRenderVisitor) VisitImageElement(element *ImageElement) any {
    return fmt.Sprintf(`<img src="%s" alt="%s" width="%d" height="%d" />`, 
                       element.Src, element.Alt, element.Width, element.Height)
}

func (hrv *HTMLRenderVisitor) VisitLinkElement(element *LinkElement) any {
    return fmt.Sprintf(`<a href="%s">%s</a>`, element.URL, element.Text)
}

func (hrv *HTMLRenderVisitor) VisitContainerElement(element *ContainerElement) any {
    var childrenHTML []string
    
    for _, child := range element.Children {
        childHTML := child.Accept(hrv).(string)
        childrenHTML = append(childrenHTML, childHTML)
    }
    
    content := strings.Join(childrenHTML, "\n")
    
    // Build attributes string
    var attrs []string
    for name, value := range element.Attributes {
        attrs = append(attrs, fmt.Sprintf(`%s="%s"`, name, value))
    }
    
    attrString := ""
    if len(attrs) > 0 {
        attrString = " " + strings.Join(attrs, " ")
    }
    
    return fmt.Sprintf("<%s%s>\n%s\n</%s>", 
                       element.Tag, attrString, content, element.Tag)
}

// Markdown rendering visitor
type MarkdownRenderVisitor struct{}

func (mrv *MarkdownRenderVisitor) VisitTextElement(element *TextElement) any {
    text := element.Content
    
    // Apply basic markdown formatting based on styles
    if element.Style != nil {
        if weight, exists := element.Style["font-weight"]; exists && weight == "bold" {
            text = "**" + text + "**"
        }
        if style, exists := element.Style["font-style"]; exists && style == "italic" {
            text = "*" + text + "*"
        }
    }
    
    return text
}

func (mrv *MarkdownRenderVisitor) VisitImageElement(element *ImageElement) any {
    return fmt.Sprintf("![%s](%s)", element.Alt, element.Src)
}

func (mrv *MarkdownRenderVisitor) VisitLinkElement(element *LinkElement) any {
    return fmt.Sprintf("[%s](%s)", element.Text, element.URL)
}

func (mrv *MarkdownRenderVisitor) VisitContainerElement(element *ContainerElement) any {
    var childrenMD []string
    
    for _, child := range element.Children {
        childMD := child.Accept(mrv).(string)
        childrenMD = append(childrenMD, childMD)
    }
    
    content := strings.Join(childrenMD, " ")
    
    // Convert HTML tags to markdown equivalents
    switch element.Tag {
    case "h1":
        return "# " + content
    case "h2":
        return "## " + content
    case "h3":
        return "### " + content
    case "p":
        return content + "\n"
    case "div":
        return content
    default:
        return content
    }
}

// Word count visitor
type WordCountVisitor struct{}

func (wcv *WordCountVisitor) VisitTextElement(element *TextElement) any {
    words := strings.Fields(element.Content)
    return len(words)
}

func (wcv *WordCountVisitor) VisitImageElement(element *ImageElement) any {
    // Count alt text words
    words := strings.Fields(element.Alt)
    return len(words)
}

func (wcv *WordCountVisitor) VisitLinkElement(element *LinkElement) any {
    words := strings.Fields(element.Text)
    return len(words)
}

func (wcv *WordCountVisitor) VisitContainerElement(element *ContainerElement) any {
    total := 0
    for _, child := range element.Children {
        count := child.Accept(wcv).(int)
        total += count
    }
    return total
}

// Statistics visitor
type StatisticsVisitor struct {
    ElementCounts map[string]int
    TotalWords    int
    TotalImages   int
    TotalLinks    int
}

func NewStatisticsVisitor() *StatisticsVisitor {
    return &StatisticsVisitor{
        ElementCounts: make(map[string]int),
    }
}

func (sv *StatisticsVisitor) VisitTextElement(element *TextElement) any {
    sv.ElementCounts["text"]++
    words := strings.Fields(element.Content)
    sv.TotalWords += len(words)
    return nil
}

func (sv *StatisticsVisitor) VisitImageElement(element *ImageElement) any {
    sv.ElementCounts["image"]++
    sv.TotalImages++
    return nil
}

func (sv *StatisticsVisitor) VisitLinkElement(element *LinkElement) any {
    sv.ElementCounts["link"]++
    sv.TotalLinks++
    words := strings.Fields(element.Text)
    sv.TotalWords += len(words)
    return nil
}

func (sv *StatisticsVisitor) VisitContainerElement(element *ContainerElement) any {
    sv.ElementCounts["container"]++
    sv.ElementCounts[element.Tag]++
    
    for _, child := range element.Children {
        child.Accept(sv)
    }
    return nil
}

func (sv *StatisticsVisitor) GetSummary() string {
    var summary strings.Builder
    summary.WriteString("Document Statistics:\n")
    summary.WriteString(fmt.Sprintf("- Total words: %d\n", sv.TotalWords))
    summary.WriteString(fmt.Sprintf("- Total images: %d\n", sv.TotalImages))
    summary.WriteString(fmt.Sprintf("- Total links: %d\n", sv.TotalLinks))
    summary.WriteString("\nElement counts:\n")
    
    for elementType, count := range sv.ElementCounts {
        summary.WriteString(fmt.Sprintf("- %s: %d\n", elementType, count))
    }
    
    return summary.String()
}

// Link validation visitor
type LinkValidationVisitor struct {
    InvalidLinks []string
    ValidLinks   []string
}

func NewLinkValidationVisitor() *LinkValidationVisitor {
    return &LinkValidationVisitor{
        InvalidLinks: make([]string, 0),
        ValidLinks:   make([]string, 0),
    }
}

func (lvv *LinkValidationVisitor) VisitTextElement(element *TextElement) any {
    return nil // No links in text elements
}

func (lvv *LinkValidationVisitor) VisitImageElement(element *ImageElement) any {
    // Validate image URLs
    if lvv.isValidURL(element.Src) {
        lvv.ValidLinks = append(lvv.ValidLinks, element.Src)
    } else {
        lvv.InvalidLinks = append(lvv.InvalidLinks, element.Src)
    }
    return nil
}

func (lvv *LinkValidationVisitor) VisitLinkElement(element *LinkElement) any {
    if lvv.isValidURL(element.URL) {
        lvv.ValidLinks = append(lvv.ValidLinks, element.URL)
    } else {
        lvv.InvalidLinks = append(lvv.InvalidLinks, element.URL)
    }
    return nil
}

func (lvv *LinkValidationVisitor) VisitContainerElement(element *ContainerElement) any {
    for _, child := range element.Children {
        child.Accept(lvv)
    }
    return nil
}

func (lvv *LinkValidationVisitor) isValidURL(url string) bool {
    // Simple URL validation
    return strings.HasPrefix(url, "http://") || 
           strings.HasPrefix(url, "https://") ||
           strings.HasPrefix(url, "/")
}

func (lvv *LinkValidationVisitor) GetReport() string {
    var report strings.Builder
    report.WriteString("Link Validation Report:\n")
    
    report.WriteString(fmt.Sprintf("Valid links (%d):\n", len(lvv.ValidLinks)))
    for _, link := range lvv.ValidLinks {
        report.WriteString(fmt.Sprintf("  ✓ %s\n", link))
    }
    
    report.WriteString(fmt.Sprintf("Invalid links (%d):\n", len(lvv.InvalidLinks)))
    for _, link := range lvv.InvalidLinks {
        report.WriteString(fmt.Sprintf("  ✗ %s\n", link))
    }
    
    return report.String()
}

func main() {
    // Build a document structure
    document := &ContainerElement{
        Tag: "div",
        Attributes: map[string]string{"class": "document"},
        Children: []Element{
            &ContainerElement{
                Tag: "h1",
                Children: []Element{
                    &TextElement{
                        Content: "Welcome to Go Interfaces",
                        Style: map[string]string{
                            "font-weight": "bold",
                            "color":       "blue",
                        },
                    },
                },
            },
            &ContainerElement{
                Tag: "p",
                Children: []Element{
                    &TextElement{
                        Content: "This is a sample document demonstrating the ",
                    },
                    &TextElement{
                        Content: "visitor pattern",
                        Style: map[string]string{
                            "font-style": "italic",
                        },
                    },
                    &TextElement{
                        Content: " in Go programming.",
                    },
                },
            },
            &ImageElement{
                Src:    "https://golang.org/lib/godoc/images/go-logo-blue.svg",
                Alt:    "Go programming language logo",
                Width:  200,
                Height: 100,
            },
            &ContainerElement{
                Tag: "p",
                Children: []Element{
                    &TextElement{
                        Content: "Learn more about Go at ",
                    },
                    &LinkElement{
                        URL:  "https://golang.org",
                        Text: "the official Go website",
                    },
                    &TextElement{
                        Content: " or check out ",
                    },
                    &LinkElement{
                        URL:  "invalid-url",
                        Text: "this broken link",
                    },
                    &TextElement{
                        Content: ".",
                    },
                },
            },
            &ContainerElement{
                Tag: "h2",
                Children: []Element{
                    &TextElement{
                        Content: "Interface Benefits",
                        Style: map[string]string{
                            "font-weight": "bold",
                        },
                    },
                },
            },
            &ContainerElement{
                Tag: "p",
                Children: []Element{
                    &TextElement{
                        Content: "Interfaces provide flexibility and enable powerful design patterns.",
                    },
                },
            },
        },
    }
    
    fmt.Println("=== HTML Rendering ===")
    htmlVisitor := &HTMLRenderVisitor{}
    htmlOutput := document.Accept(htmlVisitor).(string)
    fmt.Println(htmlOutput)
    
    fmt.Println("\n=== Markdown Rendering ===")
    markdownVisitor := &MarkdownRenderVisitor{}
    markdownOutput := document.Accept(markdownVisitor).(string)
    fmt.Println(markdownOutput)
    
    fmt.Println("\n=== Word Count ===")
    wordCountVisitor := &WordCountVisitor{}
    wordCount := document.Accept(wordCountVisitor).(int)
    fmt.Printf("Total words in document: %d\n", wordCount)
    
    fmt.Println("\n=== Statistics ===")
    statsVisitor := NewStatisticsVisitor()
    document.Accept(statsVisitor)
    fmt.Println(statsVisitor.GetSummary())
    
    fmt.Println("=== Link Validation ===")
    linkVisitor := NewLinkValidationVisitor()
    document.Accept(linkVisitor)
    fmt.Println(linkVisitor.GetReport())
}
```

The visitor pattern separates algorithms from object structures through  
interfaces. This enables adding new operations to existing object  
hierarchies without modifying the classes themselves.  

## Interface embedding and composition

Interface embedding allows creating new interfaces by composing existing  
ones, promoting code reuse and clean abstraction hierarchies.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
    "time"
)

// Basic interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}

type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}

// Composed interfaces through embedding
type ReadWriter interface {
    Reader
    Writer
}

type ReadCloser interface {
    Reader
    Closer
}

type WriteCloser interface {
    Writer
    Closer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

type ReadWriteSeeker interface {
    Reader
    Writer
    Seeker
}

type ReadWriteSeekCloser interface {
    Reader
    Writer
    Seeker
    Closer
}

// Custom file-like interface
type File interface {
    ReadWriteSeekCloser
    Name() string
    Size() int64
    ModTime() time.Time
    Stat() (FileInfo, error)
}

type FileInfo struct {
    Name    string
    Size    int64
    Mode    uint32
    ModTime time.Time
    IsDir   bool
}

// Buffer implementation that satisfies ReadWriteSeeker
type Buffer struct {
    data     []byte
    position int64
    name     string
    modTime  time.Time
}

func NewBuffer(name string, data []byte) *Buffer {
    return &Buffer{
        data:    data,
        name:    name,
        modTime: time.Now(),
    }
}

func (b *Buffer) Read(p []byte) (int, error) {
    if b.position >= int64(len(b.data)) {
        return 0, io.EOF
    }
    
    n := copy(p, b.data[b.position:])
    b.position += int64(n)
    
    fmt.Printf("Buffer.Read: read %d bytes at position %d\n", n, b.position-int64(n))
    return n, nil
}

func (b *Buffer) Write(p []byte) (int, error) {
    // Expand buffer if necessary
    needed := int(b.position) + len(p)
    if needed > len(b.data) {
        newData := make([]byte, needed)
        copy(newData, b.data)
        b.data = newData
    }
    
    n := copy(b.data[b.position:], p)
    b.position += int64(n)
    b.modTime = time.Now()
    
    fmt.Printf("Buffer.Write: wrote %d bytes at position %d\n", n, b.position-int64(n))
    return n, nil
}

func (b *Buffer) Seek(offset int64, whence int) (int64, error) {
    var abs int64
    
    switch whence {
    case io.SeekStart:
        abs = offset
    case io.SeekCurrent:
        abs = b.position + offset
    case io.SeekEnd:
        abs = int64(len(b.data)) + offset
    default:
        return 0, fmt.Errorf("invalid whence")
    }
    
    if abs < 0 {
        return 0, fmt.Errorf("negative position")
    }
    
    b.position = abs
    fmt.Printf("Buffer.Seek: moved to position %d (whence=%d, offset=%d)\n", 
               abs, whence, offset)
    
    return abs, nil
}

func (b *Buffer) Close() error {
    fmt.Printf("Buffer.Close: closing buffer %s\n", b.name)
    return nil
}

func (b *Buffer) Name() string {
    return b.name
}

func (b *Buffer) Size() int64 {
    return int64(len(b.data))
}

func (b *Buffer) ModTime() time.Time {
    return b.modTime
}

func (b *Buffer) Stat() (FileInfo, error) {
    return FileInfo{
        Name:    b.name,
        Size:    int64(len(b.data)),
        Mode:    0644,
        ModTime: b.modTime,
        IsDir:   false,
    }, nil
}

// Utility functions that work with embedded interfaces
func copyData(dst Writer, src Reader) (int64, error) {
    buf := make([]byte, 1024)
    var total int64
    
    for {
        n, err := src.Read(buf)
        if n > 0 {
            written, writeErr := dst.Write(buf[:n])
            total += int64(written)
            if writeErr != nil {
                return total, writeErr
            }
        }
        
        if err == io.EOF {
            break
        }
        if err != nil {
            return total, err
        }
    }
    
    return total, nil
}

func closeIfCloser(v any) {
    if closer, ok := v.(Closer); ok {
        fmt.Printf("Closing resource...\n")
        closer.Close()
    } else {
        fmt.Printf("Resource doesn't implement Closer\n")
    }
}

func seekToBeginning(v any) error {
    if seeker, ok := v.(Seeker); ok {
        _, err := seeker.Seek(0, io.SeekStart)
        if err != nil {
            return fmt.Errorf("failed to seek: %v", err)
        }
        fmt.Printf("Seeked to beginning\n")
        return nil
    }
    
    return fmt.Errorf("resource doesn't implement Seeker")
}

func getFileInfo(v any) string {
    if file, ok := v.(File); ok {
        info, err := file.Stat()
        if err != nil {
            return fmt.Sprintf("Error getting file info: %v", err)
        }
        
        return fmt.Sprintf("File: %s, Size: %d bytes, Modified: %s", 
                          info.Name, info.Size, info.ModTime.Format("2006-01-02 15:04:05"))
    }
    
    return "Not a File interface"
}

// Demonstrate interface satisfaction
func demonstrateInterfaceSatisfaction(resource any) {
    fmt.Printf("\n--- Interface Satisfaction Test ---\n")
    fmt.Printf("Resource type: %T\n", resource)
    
    // Test individual interfaces
    interfaces := []struct {
        name      string
        satisfied bool
    }{
        {"Reader", isReader(resource)},
        {"Writer", isWriter(resource)},
        {"Seeker", isSeeker(resource)},
        {"Closer", isCloser(resource)},
        {"ReadWriter", isReadWriter(resource)},
        {"ReadCloser", isReadCloser(resource)},
        {"WriteCloser", isWriteCloser(resource)},
        {"ReadWriteCloser", isReadWriteCloser(resource)},
        {"ReadWriteSeeker", isReadWriteSeeker(resource)},
        {"ReadWriteSeekCloser", isReadWriteSeekCloser(resource)},
        {"File", isFile(resource)},
    }
    
    fmt.Printf("Satisfied interfaces:\n")
    for _, iface := range interfaces {
        status := "✗"
        if iface.satisfied {
            status = "✓"
        }
        fmt.Printf("  %s %s\n", status, iface.name)
    }
}

// Interface type checking functions
func isReader(v any) bool         { _, ok := v.(Reader); return ok }
func isWriter(v any) bool         { _, ok := v.(Writer); return ok }
func isSeeker(v any) bool         { _, ok := v.(Seeker); return ok }
func isCloser(v any) bool         { _, ok := v.(Closer); return ok }
func isReadWriter(v any) bool     { _, ok := v.(ReadWriter); return ok }
func isReadCloser(v any) bool     { _, ok := v.(ReadCloser); return ok }
func isWriteCloser(v any) bool    { _, ok := v.(WriteCloser); return ok }
func isReadWriteCloser(v any) bool { _, ok := v.(ReadWriteCloser); return ok }
func isReadWriteSeeker(v any) bool { _, ok := v.(ReadWriteSeeker); return ok }
func isReadWriteSeekCloser(v any) bool { _, ok := v.(ReadWriteSeekCloser); return ok }
func isFile(v any) bool           { _, ok := v.(File); return ok }

// Example of using embedded interfaces in function parameters
func processReadWriter(rw ReadWriter, data string) error {
    // Write data
    _, err := rw.Write([]byte(data))
    if err != nil {
        return fmt.Errorf("write failed: %v", err)
    }
    
    // If it's also a seeker, go back to beginning
    if seeker, ok := rw.(Seeker); ok {
        seeker.Seek(0, io.SeekStart)
    }
    
    // Read it back
    buf := make([]byte, len(data))
    n, err := rw.Read(buf)
    if err != nil && err != io.EOF {
        return fmt.Errorf("read failed: %v", err)
    }
    
    fmt.Printf("Processed ReadWriter: wrote %d bytes, read back %d bytes\n", 
               len(data), n)
    fmt.Printf("Data: %s\n", string(buf[:n]))
    
    return nil
}

func main() {
    // Create a buffer that implements the full File interface
    buffer := NewBuffer("test.txt", []byte("initial content"))
    
    // Demonstrate interface satisfaction
    demonstrateInterfaceSatisfaction(buffer)
    
    fmt.Printf("\n=== Working with embedded interfaces ===\n")
    
    // Use as ReadWriter
    fmt.Printf("\n--- Using as ReadWriter ---\n")
    err := seekToBeginning(buffer)
    if err != nil {
        fmt.Printf("Seek error: %v\n", err)
    }
    
    err = processReadWriter(buffer, "Hello there from Go interfaces!")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
    
    // Use file-specific methods
    fmt.Printf("\n--- Using File interface methods ---\n")
    fmt.Printf("%s\n", getFileInfo(buffer))
    
    // Create another buffer for copying
    fmt.Printf("\n--- Copying between buffers ---\n")
    dest := NewBuffer("dest.txt", make([]byte, 0))
    
    // Seek source to beginning
    buffer.Seek(0, io.SeekStart)
    
    // Copy data
    copied, err := copyData(dest, buffer)
    if err != nil {
        fmt.Printf("Copy error: %v\n", err)
    } else {
        fmt.Printf("Copied %d bytes\n", copied)
    }
    
    // Read from destination
    dest.Seek(0, io.SeekStart)
    result := make([]byte, int(copied))
    n, err := dest.Read(result)
    if err != nil && err != io.EOF {
        fmt.Printf("Read error: %v\n", err)
    } else {
        fmt.Printf("Read from destination: %s\n", string(result[:n]))
    }
    
    // Test with standard library types
    fmt.Printf("\n--- Standard library string reader ---\n")
    stringReader := strings.NewReader("Hello from strings.Reader")
    demonstrateInterfaceSatisfaction(stringReader)
    
    // Use string reader as ReadSeeker
    if rs, ok := stringReader.(ReadWriteSeeker); ok {
        fmt.Printf("strings.Reader satisfies ReadWriteSeeker: %t\n", true)
        rs.Seek(6, io.SeekStart)
        buf := make([]byte, 4)
        n, _ := rs.Read(buf)
        fmt.Printf("Read after seek: %s\n", string(buf[:n]))
    } else {
        fmt.Printf("strings.Reader does not satisfy ReadWriteSeeker\n")
        
        // But it does satisfy ReadSeeker
        if rs2, ok := stringReader.(io.ReadSeeker); ok {
            fmt.Printf("strings.Reader satisfies io.ReadSeeker: %t\n", true)
        }
    }
    
    // Clean up
    fmt.Printf("\n--- Cleanup ---\n")
    closeIfCloser(buffer)
    closeIfCloser(dest)
    closeIfCloser(stringReader) // This won't close anything since strings.Reader doesn't implement Closer
    
    fmt.Printf("\n=== Interface Composition Benefits ===\n")
    fmt.Printf("✓ Code reuse through interface embedding\n")
    fmt.Printf("✓ Flexible APIs that accept various interface combinations\n")
    fmt.Printf("✓ Type safety with compile-time interface checking\n")
    fmt.Printf("✓ Easy testing with minimal interface implementations\n")
    fmt.Printf("✓ Clean separation of concerns\n")
}
```

Interface embedding enables powerful composition patterns where larger  
interfaces are built from smaller, focused ones. This promotes code  
reuse, clean APIs, and flexible implementations.  

## Interface for fluent APIs

Fluent interfaces use method chaining to create readable, expressive APIs  
that guide users through a sequence of operations.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Query builder interface
type QueryBuilder interface {
    Select(fields ...string) QueryBuilder
    From(table string) QueryBuilder
    Where(condition string, args ...any) QueryBuilder
    Join(table, condition string) QueryBuilder
    GroupBy(fields ...string) QueryBuilder
    OrderBy(field, direction string) QueryBuilder
    Limit(count int) QueryBuilder
    Offset(count int) QueryBuilder
    Build() (string, []any)
}

// HTTP client fluent interface
type HTTPRequestBuilder interface {
    Method(method string) HTTPRequestBuilder
    URL(url string) HTTPRequestBuilder
    Header(name, value string) HTTPRequestBuilder
    Body(body string) HTTPRequestBuilder
    Timeout(timeout time.Duration) HTTPRequestBuilder
    JSON(data any) HTTPRequestBuilder
    Execute() HTTPResponse
}

type HTTPResponse struct {
    StatusCode int
    Body       string
    Headers    map[string]string
    Error      error
}

// Email builder interface
type EmailBuilder interface {
    From(email string) EmailBuilder
    To(emails ...string) EmailBuilder
    CC(emails ...string) EmailBuilder
    BCC(emails ...string) EmailBuilder
    Subject(subject string) EmailBuilder
    Body(body string) EmailBuilder
    HTML(html string) EmailBuilder
    Attachment(filename, path string) EmailBuilder
    Priority(level int) EmailBuilder
    Send() error
}

// SQL Query Builder Implementation
type SQLQueryBuilder struct {
    selectFields []string
    fromTable    string
    whereClause  []whereCondition
    joinClauses  []joinClause
    groupByFields []string
    orderByClause []orderClause
    limitCount   int
    offsetCount  int
    args         []any
}

type whereCondition struct {
    condition string
    args      []any
}

type joinClause struct {
    table     string
    condition string
}

type orderClause struct {
    field     string
    direction string
}

func NewQueryBuilder() QueryBuilder {
    return &SQLQueryBuilder{
        selectFields: make([]string, 0),
        whereClause:  make([]whereCondition, 0),
        joinClauses:  make([]joinClause, 0),
        groupByFields: make([]string, 0),
        orderByClause: make([]orderClause, 0),
        args:         make([]any, 0),
        limitCount:   -1,
        offsetCount:  -1,
    }
}

func (qb *SQLQueryBuilder) Select(fields ...string) QueryBuilder {
    qb.selectFields = append(qb.selectFields, fields...)
    return qb
}

func (qb *SQLQueryBuilder) From(table string) QueryBuilder {
    qb.fromTable = table
    return qb
}

func (qb *SQLQueryBuilder) Where(condition string, args ...any) QueryBuilder {
    qb.whereClause = append(qb.whereClause, whereCondition{
        condition: condition,
        args:      args,
    })
    qb.args = append(qb.args, args...)
    return qb
}

func (qb *SQLQueryBuilder) Join(table, condition string) QueryBuilder {
    qb.joinClauses = append(qb.joinClauses, joinClause{
        table:     table,
        condition: condition,
    })
    return qb
}

func (qb *SQLQueryBuilder) GroupBy(fields ...string) QueryBuilder {
    qb.groupByFields = append(qb.groupByFields, fields...)
    return qb
}

func (qb *SQLQueryBuilder) OrderBy(field, direction string) QueryBuilder {
    qb.orderByClause = append(qb.orderByClause, orderClause{
        field:     field,
        direction: direction,
    })
    return qb
}

func (qb *SQLQueryBuilder) Limit(count int) QueryBuilder {
    qb.limitCount = count
    return qb
}

func (qb *SQLQueryBuilder) Offset(count int) QueryBuilder {
    qb.offsetCount = count
    return qb
}

func (qb *SQLQueryBuilder) Build() (string, []any) {
    var query strings.Builder
    
    // SELECT clause
    query.WriteString("SELECT ")
    if len(qb.selectFields) > 0 {
        query.WriteString(strings.Join(qb.selectFields, ", "))
    } else {
        query.WriteString("*")
    }
    
    // FROM clause
    if qb.fromTable != "" {
        query.WriteString(" FROM ")
        query.WriteString(qb.fromTable)
    }
    
    // JOIN clauses
    for _, join := range qb.joinClauses {
        query.WriteString(" JOIN ")
        query.WriteString(join.table)
        query.WriteString(" ON ")
        query.WriteString(join.condition)
    }
    
    // WHERE clause
    if len(qb.whereClause) > 0 {
        query.WriteString(" WHERE ")
        conditions := make([]string, len(qb.whereClause))
        for i, where := range qb.whereClause {
            conditions[i] = where.condition
        }
        query.WriteString(strings.Join(conditions, " AND "))
    }
    
    // GROUP BY clause
    if len(qb.groupByFields) > 0 {
        query.WriteString(" GROUP BY ")
        query.WriteString(strings.Join(qb.groupByFields, ", "))
    }
    
    // ORDER BY clause
    if len(qb.orderByClause) > 0 {
        query.WriteString(" ORDER BY ")
        orderParts := make([]string, len(qb.orderByClause))
        for i, order := range qb.orderByClause {
            orderParts[i] = fmt.Sprintf("%s %s", order.field, order.direction)
        }
        query.WriteString(strings.Join(orderParts, ", "))
    }
    
    // LIMIT clause
    if qb.limitCount > 0 {
        query.WriteString(fmt.Sprintf(" LIMIT %d", qb.limitCount))
    }
    
    // OFFSET clause
    if qb.offsetCount > 0 {
        query.WriteString(fmt.Sprintf(" OFFSET %d", qb.offsetCount))
    }
    
    return query.String(), qb.args
}

// HTTP Request Builder Implementation
type HTTPClient struct {
    method  string
    url     string
    headers map[string]string
    body    string
    timeout time.Duration
}

func NewHTTPClient() HTTPRequestBuilder {
    return &HTTPClient{
        headers: make(map[string]string),
        timeout: 30 * time.Second,
    }
}

func (hc *HTTPClient) Method(method string) HTTPRequestBuilder {
    hc.method = method
    return hc
}

func (hc *HTTPClient) URL(url string) HTTPRequestBuilder {
    hc.url = url
    return hc
}

func (hc *HTTPClient) Header(name, value string) HTTPRequestBuilder {
    hc.headers[name] = value
    return hc
}

func (hc *HTTPClient) Body(body string) HTTPRequestBuilder {
    hc.body = body
    return hc
}

func (hc *HTTPClient) Timeout(timeout time.Duration) HTTPRequestBuilder {
    hc.timeout = timeout
    return hc
}

func (hc *HTTPClient) JSON(data any) HTTPRequestBuilder {
    hc.headers["Content-Type"] = "application/json"
    hc.body = fmt.Sprintf("%v", data) // Simplified JSON conversion
    return hc
}

func (hc *HTTPClient) Execute() HTTPResponse {
    // Simulate HTTP request execution
    fmt.Printf("Executing HTTP request:\n")
    fmt.Printf("  Method: %s\n", hc.method)
    fmt.Printf("  URL: %s\n", hc.url)
    fmt.Printf("  Headers: %v\n", hc.headers)
    fmt.Printf("  Body: %s\n", hc.body)
    fmt.Printf("  Timeout: %v\n", hc.timeout)
    
    // Simulate response
    return HTTPResponse{
        StatusCode: 200,
        Body:       `{"message": "success", "data": "sample response"}`,
        Headers: map[string]string{
            "Content-Type": "application/json",
        },
        Error: nil,
    }
}

// Email Builder Implementation
type Email struct {
    from        string
    to          []string
    cc          []string
    bcc         []string
    subject     string
    body        string
    htmlBody    string
    attachments map[string]string // filename -> path
    priority    int
}

func NewEmailBuilder() EmailBuilder {
    return &Email{
        to:          make([]string, 0),
        cc:          make([]string, 0),
        bcc:         make([]string, 0),
        attachments: make(map[string]string),
        priority:    3, // Normal priority
    }
}

func (e *Email) From(email string) EmailBuilder {
    e.from = email
    return e
}

func (e *Email) To(emails ...string) EmailBuilder {
    e.to = append(e.to, emails...)
    return e
}

func (e *Email) CC(emails ...string) EmailBuilder {
    e.cc = append(e.cc, emails...)
    return e
}

func (e *Email) BCC(emails ...string) EmailBuilder {
    e.bcc = append(e.bcc, emails...)
    return e
}

func (e *Email) Subject(subject string) EmailBuilder {
    e.subject = subject
    return e
}

func (e *Email) Body(body string) EmailBuilder {
    e.body = body
    return e
}

func (e *Email) HTML(html string) EmailBuilder {
    e.htmlBody = html
    return e
}

func (e *Email) Attachment(filename, path string) EmailBuilder {
    e.attachments[filename] = path
    return e
}

func (e *Email) Priority(level int) EmailBuilder {
    e.priority = level
    return e
}

func (e *Email) Send() error {
    fmt.Printf("Sending email:\n")
    fmt.Printf("  From: %s\n", e.from)
    fmt.Printf("  To: %v\n", e.to)
    if len(e.cc) > 0 {
        fmt.Printf("  CC: %v\n", e.cc)
    }
    if len(e.bcc) > 0 {
        fmt.Printf("  BCC: %v\n", e.bcc)
    }
    fmt.Printf("  Subject: %s\n", e.subject)
    fmt.Printf("  Body: %s\n", e.body)
    if e.htmlBody != "" {
        fmt.Printf("  HTML Body: %s\n", e.htmlBody)
    }
    if len(e.attachments) > 0 {
        fmt.Printf("  Attachments: %v\n", e.attachments)
    }
    fmt.Printf("  Priority: %d\n", e.priority)
    fmt.Printf("Email sent successfully!\n")
    
    return nil
}

func main() {
    fmt.Println("=== SQL Query Builder ===")
    
    // Build complex SQL query using fluent interface
    query, args := NewQueryBuilder().
        Select("u.id", "u.name", "u.email", "p.title").
        From("users u").
        Join("posts p", "u.id = p.user_id").
        Where("u.active = ?", true).
        Where("u.created_at > ?", "2023-01-01").
        Where("p.published = ?", true).
        GroupBy("u.id", "u.name", "u.email").
        OrderBy("u.name", "ASC").
        OrderBy("p.created_at", "DESC").
        Limit(10).
        Offset(20).
        Build()
    
    fmt.Printf("Generated SQL:\n%s\n", query)
    fmt.Printf("Arguments: %v\n", args)
    
    // Build simple query
    fmt.Println("\n--- Simple Query ---")
    simpleQuery, simpleArgs := NewQueryBuilder().
        Select("name", "email").
        From("users").
        Where("active = ?", true).
        OrderBy("name", "ASC").
        Build()
    
    fmt.Printf("Simple SQL:\n%s\n", simpleQuery)
    fmt.Printf("Arguments: %v\n", simpleArgs)
    
    fmt.Println("\n=== HTTP Request Builder ===")
    
    // Build and execute HTTP request
    response := NewHTTPClient().
        Method("POST").
        URL("https://api.example.com/users").
        Header("Authorization", "Bearer token123").
        Header("User-Agent", "MyApp/1.0").
        JSON(map[string]any{
            "name":  "John Doe",
            "email": "john@example.com",
        }).
        Timeout(15 * time.Second).
        Execute()
    
    fmt.Printf("\nResponse:\n")
    fmt.Printf("  Status: %d\n", response.StatusCode)
    fmt.Printf("  Body: %s\n", response.Body)
    
    // Build GET request
    fmt.Println("\n--- GET Request ---")
    getResponse := NewHTTPClient().
        Method("GET").
        URL("https://api.example.com/users/123").
        Header("Accept", "application/json").
        Timeout(10 * time.Second).
        Execute()
    
    fmt.Printf("GET Response Status: %d\n", getResponse.StatusCode)
    
    fmt.Println("\n=== Email Builder ===")
    
    // Build and send complex email
    err := NewEmailBuilder().
        From("sender@company.com").
        To("recipient1@example.com", "recipient2@example.com").
        CC("manager@company.com").
        BCC("audit@company.com").
        Subject("Monthly Report - Go Interfaces").
        Body("Please find attached the monthly report covering Go interface usage.").
        HTML(`<h1>Monthly Report</h1>
              <p>Please find attached the <strong>monthly report</strong> covering Go interface usage.</p>
              <p>Best regards,<br>Development Team</p>`).
        Attachment("report.pdf", "/path/to/report.pdf").
        Attachment("data.csv", "/path/to/data.csv").
        Priority(2). // High priority
        Send()
    
    if err != nil {
        fmt.Printf("Error sending email: %v\n", err)
    }
    
    // Build simple email
    fmt.Println("\n--- Simple Email ---")
    NewEmailBuilder().
        From("hello@company.com").
        To("user@example.com").
        Subject("Welcome!").
        Body("Welcome to our service!").
        Send()
    
    fmt.Println("\n=== Fluent API Benefits ===")
    fmt.Println("✓ Readable, expressive code")
    fmt.Println("✓ Method chaining reduces boilerplate")
    fmt.Println("✓ IDE autocompletion guides usage")
    fmt.Println("✓ Immutable or mutable implementations possible")
    fmt.Println("✓ Easy to extend with new methods")
    fmt.Println("✓ Self-documenting APIs")
}
```

Fluent interfaces provide expressive, chainable APIs that make code more  
readable and self-documenting. Method chaining guides users through  
the API while maintaining type safety and flexibility.  

## Interface for concurrent patterns

Interfaces enable clean concurrent programming patterns by defining  
contracts for goroutines, workers, and synchronization primitives.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "time"
)

// Worker interface for concurrent processing
type Worker interface {
    Start(ctx context.Context) error
    Stop() error
    Process(task Task) error
    GetStats() WorkerStats
}

// Task interface for work items
type Task interface {
    GetID() string
    Execute() (any, error)
    GetPriority() int
    GetCreatedAt() time.Time
}

// Worker pool interface
type WorkerPool interface {
    Submit(task Task) error
    Start() error
    Stop() error
    GetPoolStats() PoolStats
}

// Statistics interfaces
type WorkerStats struct {
    ID           string
    TasksProcessed int64
    TasksFailed    int64
    AverageTime    time.Duration
    IsRunning      bool
}

type PoolStats struct {
    ActiveWorkers  int
    QueueSize      int
    TotalProcessed int64
    TotalFailed    int64
}

// Simple task implementation
type SimpleTask struct {
    id        string
    priority  int
    createdAt time.Time
    work      func() (any, error)
}

func NewSimpleTask(id string, priority int, work func() (any, error)) *SimpleTask {
    return &SimpleTask{
        id:        id,
        priority:  priority,
        createdAt: time.Now(),
        work:      work,
    }
}

func (st *SimpleTask) GetID() string {
    return st.id
}

func (st *SimpleTask) Execute() (any, error) {
    return st.work()
}

func (st *SimpleTask) GetPriority() int {
    return st.priority
}

func (st *SimpleTask) GetCreatedAt() time.Time {
    return st.createdAt
}

// Worker implementation
type DefaultWorker struct {
    id           string
    taskChan     chan Task
    stopChan     chan bool
    stats        WorkerStats
    mutex        sync.RWMutex
    isRunning    bool
    totalTime    time.Duration
}

func NewDefaultWorker(id string) *DefaultWorker {
    return &DefaultWorker{
        id:       id,
        taskChan: make(chan Task, 10),
        stopChan: make(chan bool),
        stats: WorkerStats{
            ID: id,
        },
    }
}

func (dw *DefaultWorker) Start(ctx context.Context) error {
    dw.mutex.Lock()
    if dw.isRunning {
        dw.mutex.Unlock()
        return fmt.Errorf("worker %s is already running", dw.id)
    }
    dw.isRunning = true
    dw.stats.IsRunning = true
    dw.mutex.Unlock()
    
    fmt.Printf("Worker %s starting...\n", dw.id)
    
    go func() {
        defer func() {
            dw.mutex.Lock()
            dw.isRunning = false
            dw.stats.IsRunning = false
            dw.mutex.Unlock()
            fmt.Printf("Worker %s stopped\n", dw.id)
        }()
        
        for {
            select {
            case task := <-dw.taskChan:
                dw.processTask(task)
                
            case <-dw.stopChan:
                // Process remaining tasks
                for {
                    select {
                    case task := <-dw.taskChan:
                        dw.processTask(task)
                    default:
                        return
                    }
                }
                
            case <-ctx.Done():
                fmt.Printf("Worker %s stopped due to context cancellation\n", dw.id)
                return
            }
        }
    }()
    
    return nil
}

func (dw *DefaultWorker) Stop() error {
    dw.mutex.RLock()
    if !dw.isRunning {
        dw.mutex.RUnlock()
        return fmt.Errorf("worker %s is not running", dw.id)
    }
    dw.mutex.RUnlock()
    
    fmt.Printf("Stopping worker %s...\n", dw.id)
    dw.stopChan <- true
    return nil
}

func (dw *DefaultWorker) Process(task Task) error {
    select {
    case dw.taskChan <- task:
        return nil
    default:
        return fmt.Errorf("worker %s queue is full", dw.id)
    }
}

func (dw *DefaultWorker) processTask(task Task) {
    start := time.Now()
    
    fmt.Printf("Worker %s processing task %s (priority: %d)\n", 
               dw.id, task.GetID(), task.GetPriority())
    
    result, err := task.Execute()
    duration := time.Since(start)
    
    dw.mutex.Lock()
    dw.totalTime += duration
    if err != nil {
        dw.stats.TasksFailed++
        fmt.Printf("Worker %s: task %s failed: %v (duration: %v)\n", 
                   dw.id, task.GetID(), err, duration)
    } else {
        dw.stats.TasksProcessed++
        fmt.Printf("Worker %s: task %s completed: %v (duration: %v)\n", 
                   dw.id, task.GetID(), result, duration)
    }
    
    totalTasks := dw.stats.TasksProcessed + dw.stats.TasksFailed
    if totalTasks > 0 {
        dw.stats.AverageTime = dw.totalTime / time.Duration(totalTasks)
    }
    dw.mutex.Unlock()
}

func (dw *DefaultWorker) GetStats() WorkerStats {
    dw.mutex.RLock()
    defer dw.mutex.RUnlock()
    return dw.stats
}

// Worker pool implementation
type DefaultWorkerPool struct {
    workers   []Worker
    taskQueue chan Task
    wg        sync.WaitGroup
    isRunning bool
    mutex     sync.RWMutex
}

func NewDefaultWorkerPool(numWorkers int) *DefaultWorkerPool {
    pool := &DefaultWorkerPool{
        workers:   make([]Worker, numWorkers),
        taskQueue: make(chan Task, numWorkers*10),
    }
    
    for i := 0; i < numWorkers; i++ {
        pool.workers[i] = NewDefaultWorker(fmt.Sprintf("worker-%d", i+1))
    }
    
    return pool
}

func (dwp *DefaultWorkerPool) Submit(task Task) error {
    dwp.mutex.RLock()
    if !dwp.isRunning {
        dwp.mutex.RUnlock()
        return fmt.Errorf("worker pool is not running")
    }
    dwp.mutex.RUnlock()
    
    select {
    case dwp.taskQueue <- task:
        return nil
    default:
        return fmt.Errorf("task queue is full")
    }
}

func (dwp *DefaultWorkerPool) Start() error {
    dwp.mutex.Lock()
    if dwp.isRunning {
        dwp.mutex.Unlock()
        return fmt.Errorf("worker pool is already running")
    }
    dwp.isRunning = true
    dwp.mutex.Unlock()
    
    fmt.Printf("Starting worker pool with %d workers...\n", len(dwp.workers))
    
    ctx := context.Background()
    
    // Start all workers
    for _, worker := range dwp.workers {
        err := worker.Start(ctx)
        if err != nil {
            return fmt.Errorf("failed to start worker: %v", err)
        }
    }
    
    // Start task dispatcher
    dwp.wg.Add(1)
    go func() {
        defer dwp.wg.Done()
        dwp.dispatchTasks()
    }()
    
    return nil
}

func (dwp *DefaultWorkerPool) Stop() error {
    dwp.mutex.Lock()
    if !dwp.isRunning {
        dwp.mutex.Unlock()
        return fmt.Errorf("worker pool is not running")
    }
    dwp.isRunning = false
    dwp.mutex.Unlock()
    
    fmt.Printf("Stopping worker pool...\n")
    
    // Close task queue
    close(dwp.taskQueue)
    
    // Stop all workers
    for _, worker := range dwp.workers {
        worker.Stop()
    }
    
    // Wait for dispatcher to finish
    dwp.wg.Wait()
    
    fmt.Printf("Worker pool stopped\n")
    return nil
}

func (dwp *DefaultWorkerPool) dispatchTasks() {
    for task := range dwp.taskQueue {
        // Find available worker (simple round-robin)
        for _, worker := range dwp.workers {
            err := worker.Process(task)
            if err == nil {
                break // Task assigned successfully
            }
        }
    }
}

func (dwp *DefaultWorkerPool) GetPoolStats() PoolStats {
    dwp.mutex.RLock()
    activeWorkers := 0
    var totalProcessed, totalFailed int64
    
    for _, worker := range dwp.workers {
        stats := worker.GetStats()
        if stats.IsRunning {
            activeWorkers++
        }
        totalProcessed += stats.TasksProcessed
        totalFailed += stats.TasksFailed
    }
    
    queueSize := len(dwp.taskQueue)
    dwp.mutex.RUnlock()
    
    return PoolStats{
        ActiveWorkers:  activeWorkers,
        QueueSize:      queueSize,
        TotalProcessed: totalProcessed,
        TotalFailed:    totalFailed,
    }
}

// Rate limiter interface
type RateLimiter interface {
    Allow() bool
    Wait(ctx context.Context) error
    SetRate(rate int, per time.Duration)
}

// Token bucket rate limiter
type TokenBucketLimiter struct {
    tokens    chan struct{}
    rate      int
    per       time.Duration
    stopChan  chan bool
    isRunning bool
    mutex     sync.Mutex
}

func NewTokenBucketLimiter(rate int, per time.Duration) *TokenBucketLimiter {
    limiter := &TokenBucketLimiter{
        tokens:   make(chan struct{}, rate),
        rate:     rate,
        per:      per,
        stopChan: make(chan bool),
    }
    
    // Fill initial tokens
    for i := 0; i < rate; i++ {
        limiter.tokens <- struct{}{}
    }
    
    limiter.start()
    return limiter
}

func (tbl *TokenBucketLimiter) start() {
    tbl.mutex.Lock()
    if tbl.isRunning {
        tbl.mutex.Unlock()
        return
    }
    tbl.isRunning = true
    tbl.mutex.Unlock()
    
    go func() {
        ticker := time.NewTicker(tbl.per / time.Duration(tbl.rate))
        defer ticker.Stop()
        
        for {
            select {
            case <-ticker.C:
                select {
                case tbl.tokens <- struct{}{}:
                    // Token added
                default:
                    // Bucket is full
                }
                
            case <-tbl.stopChan:
                tbl.mutex.Lock()
                tbl.isRunning = false
                tbl.mutex.Unlock()
                return
            }
        }
    }()
}

func (tbl *TokenBucketLimiter) Allow() bool {
    select {
    case <-tbl.tokens:
        return true
    default:
        return false
    }
}

func (tbl *TokenBucketLimiter) Wait(ctx context.Context) error {
    select {
    case <-tbl.tokens:
        return nil
    case <-ctx.Done():
        return ctx.Err()
    }
}

func (tbl *TokenBucketLimiter) SetRate(rate int, per time.Duration) {
    tbl.mutex.Lock()
    defer tbl.mutex.Unlock()
    
    tbl.rate = rate
    tbl.per = per
    
    // Recreate token channel with new capacity
    newTokens := make(chan struct{}, rate)
    
    // Transfer existing tokens
    close(tbl.tokens)
    for range tbl.tokens {
        select {
        case newTokens <- struct{}{}:
        default:
            break
        }
    }
    
    tbl.tokens = newTokens
}

func main() {
    // Create worker pool
    pool := NewDefaultWorkerPool(3)
    
    // Start the pool
    err := pool.Start()
    if err != nil {
        fmt.Printf("Failed to start pool: %v\n", err)
        return
    }
    defer pool.Stop()
    
    // Create rate limiter (5 requests per second)
    limiter := NewTokenBucketLimiter(5, time.Second)
    
    fmt.Println("=== Submitting Tasks ===")
    
    // Submit tasks with different priorities
    tasks := []struct {
        id       string
        priority int
        work     func() (any, error)
    }{
        {"task-1", 1, func() (any, error) {
            time.Sleep(100 * time.Millisecond)
            return "Task 1 completed", nil
        }},
        {"task-2", 3, func() (any, error) {
            time.Sleep(200 * time.Millisecond)
            return "Task 2 completed", nil
        }},
        {"task-3", 2, func() (any, error) {
            time.Sleep(50 * time.Millisecond)
            return nil, fmt.Errorf("task 3 failed")
        }},
        {"task-4", 1, func() (any, error) {
            time.Sleep(150 * time.Millisecond)
            return "Task 4 completed", nil
        }},
        {"task-5", 3, func() (any, error) {
            time.Sleep(80 * time.Millisecond)
            return "Task 5 completed", nil
        }},
    }
    
    // Submit tasks with rate limiting
    ctx := context.Background()
    for _, taskSpec := range tasks {
        // Wait for rate limiter
        err := limiter.Wait(ctx)
        if err != nil {
            fmt.Printf("Rate limiter error: %v\n", err)
            continue
        }
        
        task := NewSimpleTask(taskSpec.id, taskSpec.priority, taskSpec.work)
        err = pool.Submit(task)
        if err != nil {
            fmt.Printf("Failed to submit task %s: %v\n", taskSpec.id, err)
        } else {
            fmt.Printf("Submitted task %s (priority: %d)\n", 
                       taskSpec.id, taskSpec.priority)
        }
    }
    
    // Wait for tasks to complete
    fmt.Println("\n=== Processing Tasks ===")
    time.Sleep(2 * time.Second)
    
    // Get statistics
    fmt.Println("\n=== Statistics ===")
    poolStats := pool.GetPoolStats()
    fmt.Printf("Pool Stats:\n")
    fmt.Printf("  Active Workers: %d\n", poolStats.ActiveWorkers)
    fmt.Printf("  Queue Size: %d\n", poolStats.QueueSize)
    fmt.Printf("  Total Processed: %d\n", poolStats.TotalProcessed)
    fmt.Printf("  Total Failed: %d\n", poolStats.TotalFailed)
    
    fmt.Printf("\nWorker Stats:\n")
    for i := 0; i < 3; i++ {
        worker := pool.workers[i]
        stats := worker.GetStats()
        fmt.Printf("  Worker %s:\n", stats.ID)
        fmt.Printf("    Processed: %d\n", stats.TasksProcessed)
        fmt.Printf("    Failed: %d\n", stats.TasksFailed)
        fmt.Printf("    Average Time: %v\n", stats.AverageTime)
        fmt.Printf("    Running: %t\n", stats.IsRunning)
    }
    
    // Test rate limiter
    fmt.Println("\n=== Rate Limiter Test ===")
    start := time.Now()
    attempts := 0
    allowed := 0
    
    for i := 0; i < 10; i++ {
        attempts++
        if limiter.Allow() {
            allowed++
            fmt.Printf("Request %d: ALLOWED\n", i+1)
        } else {
            fmt.Printf("Request %d: RATE LIMITED\n", i+1)
        }
        time.Sleep(100 * time.Millisecond)
    }
    
    duration := time.Since(start)
    fmt.Printf("Rate limiting results: %d/%d allowed in %v\n", 
               allowed, attempts, duration)
    
    fmt.Println("\n=== Concurrent Patterns Benefits ===")
    fmt.Println("✓ Worker pools provide controlled concurrency")
    fmt.Println("✓ Rate limiters prevent resource exhaustion")
    fmt.Println("✓ Interfaces enable testable concurrent code")
    fmt.Println("✓ Clean separation of concerns")
    fmt.Println("✓ Easy to swap implementations")
    fmt.Println("✓ Graceful shutdown and error handling")
}
```

Concurrent patterns with interfaces provide clean abstractions for  
worker pools, rate limiters, and task processing. This enables scalable,  
testable concurrent systems with proper resource management.  

## Interface for reflection and dynamic behavior

Interfaces work seamlessly with Go's reflection system to enable  
dynamic behavior and runtime type inspection.  

```go
package main

import (
    "fmt"
    "reflect"
    "strings"
)

// Serializer interface for different formats
type Serializer interface {
    Serialize(data any) ([]byte, error)
    Deserialize(data []byte, target any) error
    GetContentType() string
}

// Validator interface for data validation
type Validator interface {
    Validate(data any) error
}

// Transformer interface for data transformation
type Transformer interface {
    Transform(data any) (any, error)
    GetName() string
}

// JSON serializer implementation
type JSONSerializer struct{}

func (js *JSONSerializer) Serialize(data any) ([]byte, error) {
    // Simplified JSON serialization using reflection
    v := reflect.ValueOf(data)
    t := reflect.TypeOf(data)
    
    if t.Kind() == reflect.Ptr {
        v = v.Elem()
        t = t.Elem()
    }
    
    if t.Kind() != reflect.Struct {
        return nil, fmt.Errorf("can only serialize structs")
    }
    
    var parts []string
    parts = append(parts, "{")
    
    for i := 0; i < v.NumField(); i++ {
        field := v.Field(i)
        fieldType := t.Field(i)
        
        if !field.CanInterface() {
            continue
        }
        
        fieldName := fieldType.Name
        fieldValue := field.Interface()
        
        jsonPart := fmt.Sprintf(`"%s": %v`, 
                               strings.ToLower(fieldName), 
                               formatJSONValue(fieldValue))
        
        if i > 0 {
            parts = append(parts, ", ")
        }
        parts = append(parts, jsonPart)
    }
    
    parts = append(parts, "}")
    return []byte(strings.Join(parts, "")), nil
}

func formatJSONValue(value any) string {
    v := reflect.ValueOf(value)
    switch v.Kind() {
    case reflect.String:
        return fmt.Sprintf(`"%s"`, value)
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        return fmt.Sprintf("%d", value)
    case reflect.Float32, reflect.Float64:
        return fmt.Sprintf("%f", value)
    case reflect.Bool:
        return fmt.Sprintf("%t", value)
    default:
        return fmt.Sprintf(`"%v"`, value)
    }
}

func (js *JSONSerializer) Deserialize(data []byte, target any) error {
    // Simplified deserialization - in real implementation would parse JSON
    return fmt.Errorf("deserialization not implemented in this example")
}

func (js *JSONSerializer) GetContentType() string {
    return "application/json"
}

// Struct validator using reflection
type StructValidator struct {
    rules map[string][]ValidationRule
}

type ValidationRule interface {
    Validate(value any) error
    GetDescription() string
}

type RequiredRule struct{}

func (rr *RequiredRule) Validate(value any) error {
    v := reflect.ValueOf(value)
    if !v.IsValid() || v.IsZero() {
        return fmt.Errorf("field is required")
    }
    return nil
}

func (rr *RequiredRule) GetDescription() string {
    return "field is required"
}

type MinLengthRule struct {
    minLength int
}

func (mlr *MinLengthRule) Validate(value any) error {
    v := reflect.ValueOf(value)
    if v.Kind() != reflect.String {
        return fmt.Errorf("min length rule only applies to strings")
    }
    
    if v.Len() < mlr.minLength {
        return fmt.Errorf("minimum length is %d, got %d", mlr.minLength, v.Len())
    }
    
    return nil
}

func (mlr *MinLengthRule) GetDescription() string {
    return fmt.Sprintf("minimum length: %d", mlr.minLength)
}

func NewStructValidator() *StructValidator {
    return &StructValidator{
        rules: make(map[string][]ValidationRule),
    }
}

func (sv *StructValidator) AddRule(fieldName string, rule ValidationRule) {
    sv.rules[fieldName] = append(sv.rules[fieldName], rule)
}

func (sv *StructValidator) Validate(data any) error {
    v := reflect.ValueOf(data)
    t := reflect.TypeOf(data)
    
    if t.Kind() == reflect.Ptr {
        v = v.Elem()
        t = t.Elem()
    }
    
    if t.Kind() != reflect.Struct {
        return fmt.Errorf("can only validate structs")
    }
    
    var errors []string
    
    for i := 0; i < v.NumField(); i++ {
        field := v.Field(i)
        fieldType := t.Field(i)
        fieldName := fieldType.Name
        
        if !field.CanInterface() {
            continue
        }
        
        if rules, exists := sv.rules[fieldName]; exists {
            for _, rule := range rules {
                err := rule.Validate(field.Interface())
                if err != nil {
                    errors = append(errors, fmt.Sprintf("%s: %s", fieldName, err.Error()))
                }
            }
        }
    }
    
    if len(errors) > 0 {
        return fmt.Errorf("validation failed: %s", strings.Join(errors, "; "))
    }
    
    return nil
}

// Field transformer for data manipulation
type FieldTransformer struct {
    transformations map[string]TransformFunc
}

type TransformFunc func(value any) any

func NewFieldTransformer() *FieldTransformer {
    return &FieldTransformer{
        transformations: make(map[string]TransformFunc),
    }
}

func (ft *FieldTransformer) AddTransformation(fieldName string, transform TransformFunc) {
    ft.transformations[fieldName] = transform
}

func (ft *FieldTransformer) Transform(data any) (any, error) {
    v := reflect.ValueOf(data)
    t := reflect.TypeOf(data)
    
    if t.Kind() == reflect.Ptr {
        v = v.Elem()
        t = t.Elem()
    }
    
    if t.Kind() != reflect.Struct {
        return nil, fmt.Errorf("can only transform structs")
    }
    
    // Create new struct of same type
    newValue := reflect.New(t).Elem()
    
    for i := 0; i < v.NumField(); i++ {
        field := v.Field(i)
        fieldType := t.Field(i)
        fieldName := fieldType.Name
        
        if !field.CanInterface() {
            continue
        }
        
        originalValue := field.Interface()
        newFieldValue := newValue.Field(i)
        
        if transform, exists := ft.transformations[fieldName]; exists {
            transformedValue := transform(originalValue)
            
            // Set the transformed value
            if newFieldValue.CanSet() {
                newFieldValue.Set(reflect.ValueOf(transformedValue))
            }
        } else {
            // Copy original value
            if newFieldValue.CanSet() {
                newFieldValue.Set(field)
            }
        }
    }
    
    return newValue.Interface(), nil
}

func (ft *FieldTransformer) GetName() string {
    return "FieldTransformer"
}

// Interface detector for runtime interface checking
type InterfaceDetector struct{}

func (id *InterfaceDetector) GetImplementedInterfaces(value any) []string {
    var interfaces []string
    
    v := reflect.ValueOf(value)
    t := reflect.TypeOf(value)
    
    // Check for common interfaces
    interfaceChecks := map[string]reflect.Type{
        "fmt.Stringer":    reflect.TypeOf((*fmt.Stringer)(nil)).Elem(),
        "error":           reflect.TypeOf((*error)(nil)).Elem(),
        "Serializer":      reflect.TypeOf((*Serializer)(nil)).Elem(),
        "Validator":       reflect.TypeOf((*Validator)(nil)).Elem(),
        "Transformer":     reflect.TypeOf((*Transformer)(nil)).Elem(),
    }
    
    for interfaceName, interfaceType := range interfaceChecks {
        if t.Implements(interfaceType) {
            interfaces = append(interfaces, interfaceName)
        }
    }
    
    return interfaces
}

func (id *InterfaceDetector) CanConvertTo(value any, targetInterface reflect.Type) bool {
    t := reflect.TypeOf(value)
    return t.Implements(targetInterface)
}

func (id *InterfaceDetector) GetMethods(value any) []string {
    var methods []string
    
    t := reflect.TypeOf(value)
    
    for i := 0; i < t.NumMethod(); i++ {
        method := t.Method(i)
        methods = append(methods, method.Name)
    }
    
    return methods
}

// Test structs
type User struct {
    Name     string
    Email    string
    Age      int
    IsActive bool
}

type Product struct {
    Name  string
    Price float64
}

func (p Product) String() string {
    return fmt.Sprintf("%s ($%.2f)", p.Name, p.Price)
}

func main() {
    fmt.Println("=== Reflection with Interfaces ===")
    
    // Create test data
    user := User{
        Name:     "Alice Johnson",
        Email:    "alice@example.com", 
        Age:      28,
        IsActive: true,
    }
    
    product := Product{
        Name:  "Laptop",
        Price: 999.99,
    }
    
    // Test serialization
    fmt.Println("\n--- Serialization ---")
    serializer := &JSONSerializer{}
    
    userData, err := serializer.Serialize(user)
    if err != nil {
        fmt.Printf("Serialization error: %v\n", err)
    } else {
        fmt.Printf("User JSON: %s\n", string(userData))
    }
    
    productData, err := serializer.Serialize(product)
    if err != nil {
        fmt.Printf("Serialization error: %v\n", err)
    } else {
        fmt.Printf("Product JSON: %s\n", string(productData))
    }
    
    // Test validation
    fmt.Println("\n--- Validation ---")
    validator := NewStructValidator()
    validator.AddRule("Name", &RequiredRule{})
    validator.AddRule("Email", &RequiredRule{})
    validator.AddRule("Email", &MinLengthRule{minLength: 5})
    
    err = validator.Validate(user)
    if err != nil {
        fmt.Printf("Validation failed: %v\n", err)
    } else {
        fmt.Printf("User validation passed\n")
    }
    
    // Test with invalid data
    invalidUser := User{
        Name:  "",
        Email: "a@b",
        Age:   25,
    }
    
    err = validator.Validate(invalidUser)
    if err != nil {
        fmt.Printf("Invalid user validation failed (expected): %v\n", err)
    }
    
    // Test transformation
    fmt.Println("\n--- Transformation ---")
    transformer := NewFieldTransformer()
    transformer.AddTransformation("Name", func(value any) any {
        if str, ok := value.(string); ok {
            return strings.ToUpper(str)
        }
        return value
    })
    transformer.AddTransformation("Email", func(value any) any {
        if str, ok := value.(string); ok {
            return strings.ToLower(str)
        }
        return value
    })
    
    transformedData, err := transformer.Transform(user)
    if err != nil {
        fmt.Printf("Transformation error: %v\n", err)
    } else {
        fmt.Printf("Original user: %+v\n", user)
        fmt.Printf("Transformed user: %+v\n", transformedData)
    }
    
    // Test interface detection
    fmt.Println("\n--- Interface Detection ---")
    detector := &InterfaceDetector{}
    
    objects := []any{user, product, serializer, validator, transformer}
    objectNames := []string{"User", "Product", "JSONSerializer", "StructValidator", "FieldTransformer"}
    
    for i, obj := range objects {
        fmt.Printf("\n%s implements:\n", objectNames[i])
        
        interfaces := detector.GetImplementedInterfaces(obj)
        for _, iface := range interfaces {
            fmt.Printf("  ✓ %s\n", iface)
        }
        
        if len(interfaces) == 0 {
            fmt.Printf("  (no detected interfaces)\n")
        }
        
        methods := detector.GetMethods(obj)
        fmt.Printf("Methods: %v\n", methods)
    }
    
    // Test runtime interface conversion
    fmt.Println("\n--- Runtime Interface Conversion ---")
    
    var serializers []Serializer
    var validators []Validator
    var transformers []Transformer
    
    allObjects := []any{serializer, validator, transformer}
    
    for _, obj := range allObjects {
        if s, ok := obj.(Serializer); ok {
            serializers = append(serializers, s)
            fmt.Printf("Added %T as Serializer\n", obj)
        }
        
        if v, ok := obj.(Validator); ok {
            validators = append(validators, v)
            fmt.Printf("Added %T as Validator\n", obj)
        }
        
        if t, ok := obj.(Transformer); ok {
            transformers = append(transformers, t)
            fmt.Printf("Added %T as Transformer\n", obj)
        }
    }
    
    fmt.Printf("\nFound %d serializers, %d validators, %d transformers\n", 
               len(serializers), len(validators), len(transformers))
    
    fmt.Println("\n=== Reflection Benefits ===")
    fmt.Println("✓ Runtime type inspection and manipulation")
    fmt.Println("✓ Generic serialization/deserialization")  
    fmt.Println("✓ Dynamic validation systems")
    fmt.Println("✓ Interface detection and conversion")
    fmt.Println("✓ Plugin and extension systems")
    fmt.Println("✓ ORM and mapping frameworks")
}
```

Reflection enables powerful dynamic behavior with interfaces, allowing  
runtime type inspection, generic operations, and flexible plugin systems.  
This is fundamental for frameworks, ORMs, and serialization libraries.  

## Interface for error handling patterns

Go's error interface enables sophisticated error handling patterns  
with custom error types and error wrapping.  

```go
package main

import (
    "errors"
    "fmt"
    "time"
)

// Custom error interfaces
type DetailedError interface {
    error
    GetCode() string
    GetDetails() map[string]any
    GetTimestamp() time.Time
}

type RetryableError interface {
    error
    IsRetryable() bool
    GetRetryAfter() time.Duration
}

type TemporaryError interface {
    error
    Temporary() bool
}

type TimeoutError interface {
    error
    Timeout() bool
}

// Error implementations
type ValidationError struct {
    code      string
    message   string
    details   map[string]any
    timestamp time.Time
}

func NewValidationError(code, message string, details map[string]any) *ValidationError {
    return &ValidationError{
        code:      code,
        message:   message,
        details:   details,
        timestamp: time.Now(),
    }
}

func (ve *ValidationError) Error() string {
    return fmt.Sprintf("validation failed [%s]: %s", ve.code, ve.message)
}

func (ve *ValidationError) GetCode() string {
    return ve.code
}

func (ve *ValidationError) GetDetails() map[string]any {
    return ve.details
}

func (ve *ValidationError) GetTimestamp() time.Time {
    return ve.timestamp
}

type NetworkError struct {
    operation   string
    address     string
    err         error
    temporary   bool
    timeout     bool
    retryable   bool
    retryAfter  time.Duration
    timestamp   time.Time
}

func NewNetworkError(operation, address string, err error, temporary, timeout bool) *NetworkError {
    return &NetworkError{
        operation:  operation,
        address:    address,
        err:        err,
        temporary:  temporary,
        timeout:    timeout,
        retryable:  temporary || timeout,
        retryAfter: 5 * time.Second,
        timestamp:  time.Now(),
    }
}

func (ne *NetworkError) Error() string {
    return fmt.Sprintf("network error during %s to %s: %v", 
                       ne.operation, ne.address, ne.err)
}

func (ne *NetworkError) Unwrap() error {
    return ne.err
}

func (ne *NetworkError) Temporary() bool {
    return ne.temporary
}

func (ne *NetworkError) Timeout() bool {
    return ne.timeout
}

func (ne *NetworkError) IsRetryable() bool {
    return ne.retryable
}

func (ne *NetworkError) GetRetryAfter() time.Duration {
    return ne.retryAfter
}

func (ne *NetworkError) GetCode() string {
    if ne.timeout {
        return "NETWORK_TIMEOUT"
    }
    if ne.temporary {
        return "NETWORK_TEMPORARY"
    }
    return "NETWORK_ERROR"
}

func (ne *NetworkError) GetDetails() map[string]any {
    return map[string]any{
        "operation": ne.operation,
        "address":   ne.address,
        "temporary": ne.temporary,
        "timeout":   ne.timeout,
    }
}

func (ne *NetworkError) GetTimestamp() time.Time {
    return ne.timestamp
}

type DatabaseError struct {
    query     string
    operation string
    err       error
    code      string
    timestamp time.Time
}

func NewDatabaseError(query, operation string, err error, code string) *DatabaseError {
    return &DatabaseError{
        query:     query,
        operation: operation,
        err:       err,
        code:      code,
        timestamp: time.Now(),
    }
}

func (de *DatabaseError) Error() string {
    return fmt.Sprintf("database error during %s [%s]: %v", 
                       de.operation, de.code, de.err)
}

func (de *DatabaseError) Unwrap() error {
    return de.err
}

func (de *DatabaseError) GetCode() string {
    return de.code
}

func (de *DatabaseError) GetDetails() map[string]any {
    return map[string]any{
        "operation": de.operation,
        "query":     de.query,
    }
}

func (de *DatabaseError) GetTimestamp() time.Time {
    return de.timestamp
}

func (de *DatabaseError) IsRetryable() bool {
    // Some database errors are retryable
    retryableCodes := []string{"DEADLOCK", "TIMEOUT", "CONNECTION_LOST"}
    for _, code := range retryableCodes {
        if de.code == code {
            return true
        }
    }
    return false
}

func (de *DatabaseError) GetRetryAfter() time.Duration {
    switch de.code {
    case "DEADLOCK":
        return 100 * time.Millisecond
    case "TIMEOUT", "CONNECTION_LOST":
        return 1 * time.Second
    default:
        return 0
    }
}

// Error handler interface
type ErrorHandler interface {
    CanHandle(err error) bool
    Handle(err error) error
}

// Retry error handler
type RetryHandler struct {
    maxRetries int
    baseDelay  time.Duration
}

func NewRetryHandler(maxRetries int, baseDelay time.Duration) *RetryHandler {
    return &RetryHandler{
        maxRetries: maxRetries,
        baseDelay:  baseDelay,
    }
}

func (rh *RetryHandler) CanHandle(err error) bool {
    var retryable RetryableError
    return errors.As(err, &retryable) && retryable.IsRetryable()
}

func (rh *RetryHandler) Handle(err error) error {
    var retryable RetryableError
    if !errors.As(err, &retryable) || !retryable.IsRetryable() {
        return err
    }
    
    fmt.Printf("Retryable error detected: %v\n", err)
    fmt.Printf("Would retry after: %v\n", retryable.GetRetryAfter())
    
    // In real implementation, would actually retry the operation
    return fmt.Errorf("retry handler: %w", err)
}

// Logging error handler
type LoggingHandler struct {
    logLevel string
}

func NewLoggingHandler(logLevel string) *LoggingHandler {
    return &LoggingHandler{logLevel: logLevel}
}

func (lh *LoggingHandler) CanHandle(err error) bool {
    return err != nil // Can log any error
}

func (lh *LoggingHandler) Handle(err error) error {
    timestamp := time.Now().Format("2006-01-02 15:04:05")
    
    fmt.Printf("[%s] %s: %v\n", timestamp, lh.logLevel, err)
    
    // Add detailed logging for custom error types
    if detailed, ok := err.(DetailedError); ok {
        fmt.Printf("  Code: %s\n", detailed.GetCode())
        fmt.Printf("  Details: %v\n", detailed.GetDetails())
        fmt.Printf("  Timestamp: %s\n", detailed.GetTimestamp().Format("2006-01-02 15:04:05"))
    }
    
    if temporary, ok := err.(TemporaryError); ok {
        fmt.Printf("  Temporary: %t\n", temporary.Temporary())
    }
    
    if timeout, ok := err.(TimeoutError); ok {
        fmt.Printf("  Timeout: %t\n", timeout.Timeout())
    }
    
    return err // Return original error
}

// Error chain processor
type ErrorProcessor struct {
    handlers []ErrorHandler
}

func NewErrorProcessor() *ErrorProcessor {
    return &ErrorProcessor{
        handlers: make([]ErrorHandler, 0),
    }
}

func (ep *ErrorProcessor) AddHandler(handler ErrorHandler) {
    ep.handlers = append(ep.handlers, handler)
}

func (ep *ErrorProcessor) ProcessError(err error) error {
    if err == nil {
        return nil
    }
    
    currentErr := err
    
    for _, handler := range ep.handlers {
        if handler.CanHandle(currentErr) {
            currentErr = handler.Handle(currentErr)
        }
    }
    
    return currentErr
}

// Error aggregator for collecting multiple errors
type ErrorAggregator struct {
    errors []error
}

func NewErrorAggregator() *ErrorAggregator {
    return &ErrorAggregator{
        errors: make([]error, 0),
    }
}

func (ea *ErrorAggregator) Add(err error) {
    if err != nil {
        ea.errors = append(ea.errors, err)
    }
}

func (ea *ErrorAggregator) HasErrors() bool {
    return len(ea.errors) > 0
}

func (ea *ErrorAggregator) Error() string {
    if len(ea.errors) == 0 {
        return "no errors"
    }
    
    if len(ea.errors) == 1 {
        return ea.errors[0].Error()
    }
    
    var messages []string
    for i, err := range ea.errors {
        messages = append(messages, fmt.Sprintf("%d: %s", i+1, err.Error()))
    }
    
    return fmt.Sprintf("multiple errors occurred:\n  %s", 
                       fmt.Sprintf("\n  %s", messages))
}

func (ea *ErrorAggregator) GetErrors() []error {
    return ea.errors
}

// Simulate operations that can fail
func validateUser(name, email string) error {
    if name == "" {
        return NewValidationError("EMPTY_NAME", "name cannot be empty", 
                                  map[string]any{"field": "name"})
    }
    
    if email == "" || !contains(email, "@") {
        return NewValidationError("INVALID_EMAIL", "email is invalid", 
                                  map[string]any{"field": "email", "value": email})
    }
    
    return nil
}

func connectToDatabase() error {
    // Simulate database connection failure
    return NewDatabaseError("SELECT 1", "connect", 
                            errors.New("connection refused"), "CONNECTION_LOST")
}

func fetchFromAPI(url string) error {
    // Simulate network timeout
    return NewNetworkError("GET", url, errors.New("context deadline exceeded"), 
                           false, true)
}

func contains(s, substr string) bool {
    return len(s) >= len(substr) && 
           (len(s) == len(substr) && s == substr ||
            len(s) > len(substr) && (s[0:len(substr)] == substr || 
                                     s[len(s)-len(substr):] == substr ||
                                     containsSubstring(s, substr)))
}

func containsSubstring(s, substr string) bool {
    for i := 0; i <= len(s)-len(substr); i++ {
        if s[i:i+len(substr)] == substr {
            return true
        }
    }
    return false
}

func main() {
    fmt.Println("=== Error Handling Patterns ===")
    
    // Setup error processor
    processor := NewErrorProcessor()
    processor.AddHandler(NewLoggingHandler("ERROR"))
    processor.AddHandler(NewRetryHandler(3, 1*time.Second))
    
    // Test different error types
    fmt.Println("\n--- Validation Errors ---")
    err := validateUser("", "invalid-email")
    processor.ProcessError(err)
    
    err = validateUser("John Doe", "john@example.com")
    if err == nil {
        fmt.Println("User validation passed")
    }
    
    fmt.Println("\n--- Network Errors ---")
    err = fetchFromAPI("https://api.example.com/data")
    processor.ProcessError(err)
    
    fmt.Println("\n--- Database Errors ---")
    err = connectToDatabase()
    processor.ProcessError(err)
    
    // Test error aggregation
    fmt.Println("\n--- Error Aggregation ---")
    aggregator := NewErrorAggregator()
    
    operations := []func() error{
        func() error { return validateUser("", "") },
        func() error { return connectToDatabase() },
        func() error { return fetchFromAPI("https://timeout.example.com") },
    }
    
    for i, op := range operations {
        err := op()
        if err != nil {
            fmt.Printf("Operation %d failed: %v\n", i+1, err)
            aggregator.Add(err)
        }
    }
    
    if aggregator.HasErrors() {
        fmt.Printf("\nAggregated errors:\n%v\n", aggregator.Error())
    }
    
    // Test error type checking
    fmt.Println("\n--- Error Type Analysis ---")
    testErrors := []error{
        NewValidationError("TEST", "test validation error", nil),
        NewNetworkError("GET", "example.com", errors.New("timeout"), false, true),
        NewDatabaseError("SELECT", "query", errors.New("deadlock"), "DEADLOCK"),
    }
    
    for i, testErr := range testErrors {
        fmt.Printf("\nError %d: %v\n", i+1, testErr)
        
        // Check error interfaces
        if detailed, ok := testErr.(DetailedError); ok {
            fmt.Printf("  Implements DetailedError: code=%s\n", detailed.GetCode())
        }
        
        if retryable, ok := testErr.(RetryableError); ok {
            fmt.Printf("  Implements RetryableError: retryable=%t, after=%v\n", 
                       retryable.IsRetryable(), retryable.GetRetryAfter())
        }
        
        if temporary, ok := testErr.(TemporaryError); ok {
            fmt.Printf("  Implements TemporaryError: temporary=%t\n", 
                       temporary.Temporary())
        }
        
        if timeout, ok := testErr.(TimeoutError); ok {
            fmt.Printf("  Implements TimeoutError: timeout=%t\n", 
                       timeout.Timeout())
        }
    }
    
    // Test error unwrapping
    fmt.Println("\n--- Error Unwrapping ---")
    wrappedErr := fmt.Errorf("operation failed: %w", 
                            NewNetworkError("POST", "api.example.com", 
                                          errors.New("connection reset"), true, false))
    
    fmt.Printf("Wrapped error: %v\n", wrappedErr)
    
    // Unwrap to get original network error
    var netErr *NetworkError
    if errors.As(wrappedErr, &netErr) {
        fmt.Printf("Unwrapped NetworkError: %v\n", netErr)
        fmt.Printf("  Temporary: %t\n", netErr.Temporary())
        fmt.Printf("  Retryable: %t\n", netErr.IsRetryable())
    }
    
    fmt.Println("\n=== Error Pattern Benefits ===")
    fmt.Println("✓ Rich error information with custom types")
    fmt.Println("✓ Error categorization and specialized handling") 
    fmt.Println("✓ Retry logic based on error characteristics")
    fmt.Println("✓ Error aggregation for batch operations")
    fmt.Println("✓ Structured logging with error context")
    fmt.Println("✓ Error wrapping preserves original context")
}
```

Custom error interfaces enable sophisticated error handling with  
categorization, retry logic, and detailed error information. This  
promotes robust, maintainable error handling in complex applications.  

## Interface best practices and patterns

This final example demonstrates essential best practices and common  
patterns when working with Go interfaces.  

```go
package main

import (
    "fmt"
    "io"
    "time"
)

// Best Practice 1: Keep interfaces small and focused
// ✓ Good: Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// ✗ Avoid: Large interfaces with many methods
// type EverythingInterface interface {
//     Read(p []byte) (n int, err error)
//     Write(p []byte) (n int, err error)  
//     Close() error
//     Seek(offset int64, whence int) (int64, error)
//     Stat() (FileInfo, error)
//     // ... many more methods
// }

// Best Practice 2: Define interfaces at the point of use (consumer side)
// ✓ Good: Package defines interfaces for its dependencies
type UserRepository interface {
    GetUser(id string) (*User, error)
    SaveUser(user *User) error
}

type UserService struct {
    repo UserRepository // Depends on interface, not concrete type
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

// Best Practice 3: Return concrete types, accept interfaces
// ✓ Good: Function accepts interface parameter
func ProcessReader(r io.Reader) error {
    data, err := io.ReadAll(r)
    if err != nil {
        return err
    }
    fmt.Printf("Read %d bytes\n", len(data))
    return nil
}

// ✓ Good: Function returns concrete type
func CreateStringReader(content string) *StringReader {
    return &StringReader{content: content}
}

// Best Practice 4: Use composition over inheritance
type StringReader struct {
    content  string
    position int
}

func (sr *StringReader) Read(p []byte) (n int, err error) {
    if sr.position >= len(sr.content) {
        return 0, io.EOF
    }
    
    n = copy(p, sr.content[sr.position:])
    sr.position += n
    return n, nil
}

// Compose larger interfaces from smaller ones
type ReadCloser interface {
    Reader
    io.Closer
}

// Best Practice 5: Use empty interface (any) judiciously
// ✓ Good: Used when type is truly unknown
func PrintValue(value any) {
    fmt.Printf("Value: %v, Type: %T\n", value, value)
}

// ✗ Avoid: Overusing any when specific types would work
// func BadFunction(data any) any { ... }

// Best Practice 6: Implement fmt.Stringer for custom types
type User struct {
    ID    string
    Name  string
    Email string
}

func (u User) String() string {
    return fmt.Sprintf("User{ID: %s, Name: %s, Email: %s}", 
                       u.ID, u.Name, u.Email)
}

// Best Practice 7: Use type assertions carefully
func SafeTypeAssertion(value any) {
    // ✓ Good: Use comma ok idiom
    if str, ok := value.(string); ok {
        fmt.Printf("String value: %s\n", str)
    } else {
        fmt.Printf("Not a string: %T\n", value)
    }
    
    // ✗ Avoid: Direct assertion without check (can panic)
    // str := value.(string) // Dangerous!
}

// Best Practice 8: Design for testing with interfaces
type Clock interface {
    Now() time.Time
}

type RealClock struct{}

func (rc *RealClock) Now() time.Time {
    return time.Now()
}

type MockClock struct {
    currentTime time.Time
}

func (mc *MockClock) Now() time.Time {
    return mc.currentTime
}

func (mc *MockClock) SetTime(t time.Time) {
    mc.currentTime = t
}

type TimeService struct {
    clock Clock
}

func NewTimeService(clock Clock) *TimeService {
    return &TimeService{clock: clock}
}

func (ts *TimeService) GetFormattedTime() string {
    return ts.clock.Now().Format("2006-01-02 15:04:05")
}

// Best Practice 9: Use interface embedding for composition
type Validator interface {
    Validate(data any) error
}

type Transformer interface {
    Transform(data any) (any, error)
}

type DataProcessor interface {
    Validator
    Transformer
    Process(data any) (any, error)
}

// Implementation
type JSONProcessor struct{}

func (jp *JSONProcessor) Validate(data any) error {
    // Validation logic
    fmt.Printf("Validating JSON data: %T\n", data)
    return nil
}

func (jp *JSONProcessor) Transform(data any) (any, error) {
    // Transformation logic
    fmt.Printf("Transforming JSON data: %T\n", data)
    return fmt.Sprintf("transformed_%v", data), nil
}

func (jp *JSONProcessor) Process(data any) (any, error) {
    err := jp.Validate(data)
    if err != nil {
        return nil, err
    }
    
    return jp.Transform(data)
}

// Best Practice 10: Handle nil interfaces correctly
func HandleNilInterface(value any) {
    if value == nil {
        fmt.Println("Value is nil")
        return
    }
    
    // Check for typed nil
    if reader, ok := value.(io.Reader); ok {
        if reader == nil {
            fmt.Println("Reader interface is typed nil")
            return
        }
        fmt.Println("Reader interface has value")
    }
}

// Best Practice 11: Use interfaces for configuration
type Config interface {
    GetString(key string) string
    GetInt(key string) int
    GetBool(key string) bool
}

type MapConfig struct {
    data map[string]any
}

func (mc *MapConfig) GetString(key string) string {
    if val, exists := mc.data[key]; exists {
        if str, ok := val.(string); ok {
            return str
        }
    }
    return ""
}

func (mc *MapConfig) GetInt(key string) int {
    if val, exists := mc.data[key]; exists {
        if num, ok := val.(int); ok {
            return num
        }
    }
    return 0
}

func (mc *MapConfig) GetBool(key string) bool {
    if val, exists := mc.data[key]; exists {
        if b, ok := val.(bool); ok {
            return b
        }
    }
    return false
}

type DatabaseConnection struct {
    config Config
}

func NewDatabaseConnection(config Config) *DatabaseConnection {
    return &DatabaseConnection{config: config}
}

func (dc *DatabaseConnection) Connect() error {
    host := dc.config.GetString("db_host")
    port := dc.config.GetInt("db_port")
    ssl := dc.config.GetBool("db_ssl")
    
    fmt.Printf("Connecting to %s:%d (SSL: %t)\n", host, port, ssl)
    return nil
}

// Best Practice 12: Interface segregation principle
// ✓ Good: Separate interfaces for different responsibilities
type FileReader interface {
    Read(p []byte) (n int, err error)
}

type FileWriter interface {
    Write(p []byte) (n int, err error)
}

type FileCloser interface {
    Close() error
}

// Clients can depend only on what they need
func ReadOnly(reader FileReader) {
    // Only needs reading capability
    buffer := make([]byte, 1024)
    reader.Read(buffer)
}

func WriteOnly(writer FileWriter) {
    // Only needs writing capability
    writer.Write([]byte("data"))
}

// Best Practice 13: Error handling with interfaces
type ServiceError interface {
    error
    IsRetryable() bool
    GetCode() string
}

type HTTPError struct {
    Code    string
    Message string
    Status  int
}

func (he *HTTPError) Error() string {
    return fmt.Sprintf("HTTP %d [%s]: %s", he.Status, he.Code, he.Message)
}

func (he *HTTPError) IsRetryable() bool {
    return he.Status >= 500 // Server errors are retryable
}

func (he *HTTPError) GetCode() string {
    return he.Code
}

func CallAPI() error {
    return &HTTPError{
        Code:    "SERVER_ERROR",
        Message: "Internal server error",
        Status:  500,
    }
}

func main() {
    fmt.Println("=== Interface Best Practices Demo ===")
    
    // 1. Small, focused interfaces
    fmt.Println("\n--- Practice 1: Small Interfaces ---")
    reader := CreateStringReader("hello there")
    ProcessReader(reader)
    
    // 2. Consumer-defined interfaces
    fmt.Println("\n--- Practice 2: Consumer-defined Interfaces ---")
    // UserService depends on interface, not concrete implementation
    fmt.Println("✓ UserService uses UserRepository interface")
    
    // 3. Return concrete types, accept interfaces
    fmt.Println("\n--- Practice 3: Concrete Returns, Interface Parameters ---")
    stringReader := CreateStringReader("example content")
    ProcessReader(stringReader) // Accepts interface
    
    // 4. Composition over inheritance
    fmt.Println("\n--- Practice 4: Composition ---")
    processor := &JSONProcessor{}
    result, err := processor.Process("sample data")
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Result: %v\n", result)
    }
    
    // 5. Judicious use of any
    fmt.Println("\n--- Practice 5: Using 'any' Judiciously ---")
    PrintValue("string")
    PrintValue(42)
    PrintValue(true)
    
    // 6. Implement fmt.Stringer
    fmt.Println("\n--- Practice 6: fmt.Stringer Implementation ---")
    user := User{ID: "123", Name: "Alice", Email: "alice@example.com"}
    fmt.Printf("User: %s\n", user) // Uses String() method
    
    // 7. Safe type assertions
    fmt.Println("\n--- Practice 7: Safe Type Assertions ---")
    SafeTypeAssertion("hello")
    SafeTypeAssertion(42)
    
    // 8. Design for testing
    fmt.Println("\n--- Practice 8: Design for Testing ---")
    realClock := &RealClock{}
    timeService := NewTimeService(realClock)
    fmt.Printf("Real time: %s\n", timeService.GetFormattedTime())
    
    mockClock := &MockClock{}
    mockClock.SetTime(time.Date(2023, 12, 25, 12, 0, 0, 0, time.UTC))
    mockTimeService := NewTimeService(mockClock)
    fmt.Printf("Mock time: %s\n", mockTimeService.GetFormattedTime())
    
    // 9. Interface embedding
    fmt.Println("\n--- Practice 9: Interface Embedding ---")
    fmt.Println("✓ DataProcessor embeds Validator and Transformer")
    
    // 10. Handle nil interfaces
    fmt.Println("\n--- Practice 10: Nil Interface Handling ---")
    HandleNilInterface(nil)
    var nilReader io.Reader = nil
    HandleNilInterface(nilReader)
    
    // 11. Configuration with interfaces
    fmt.Println("\n--- Practice 11: Configuration Interfaces ---")
    config := &MapConfig{
        data: map[string]any{
            "db_host": "localhost",
            "db_port": 5432,
            "db_ssl":  true,
        },
    }
    
    db := NewDatabaseConnection(config)
    db.Connect()
    
    // 12. Interface segregation
    fmt.Println("\n--- Practice 12: Interface Segregation ---")
    ReadOnly(reader)
    fmt.Println("✓ Functions depend only on needed interfaces")
    
    // 13. Error handling with interfaces
    fmt.Println("\n--- Practice 13: Error Interfaces ---")
    err := CallAPI()
    if serviceErr, ok := err.(ServiceError); ok {
        fmt.Printf("Service error: %s\n", serviceErr.Error())
        fmt.Printf("Retryable: %t\n", serviceErr.IsRetryable())
        fmt.Printf("Code: %s\n", serviceErr.GetCode())
    }
    
    fmt.Println("\n=== Key Interface Principles ===")
    fmt.Println("✓ Keep interfaces small and focused")
    fmt.Println("✓ Define interfaces where they're used (consumer side)")
    fmt.Println("✓ Return concrete types, accept interface parameters")
    fmt.Println("✓ Use composition over inheritance")
    fmt.Println("✓ Use 'any' sparingly and with purpose")
    fmt.Println("✓ Implement fmt.Stringer for custom types")
    fmt.Println("✓ Always use comma-ok idiom for type assertions")
    fmt.Println("✓ Design with testing in mind")
    fmt.Println("✓ Use interface embedding for composition")
    fmt.Println("✓ Handle nil interfaces correctly")
    fmt.Println("✓ Use interfaces for flexible configuration")
    fmt.Println("✓ Follow interface segregation principle")
    fmt.Println("✓ Leverage interfaces for rich error handling")
    
    fmt.Println("\n=== Remember ===")
    fmt.Println("\"The bigger the interface, the weaker the abstraction.\"")
    fmt.Println("\"Don't design with interfaces, discover them.\"")
    fmt.Println("\"Interfaces should describe behavior, not data.\"")
}
```

Interface best practices ensure clean, maintainable, and flexible code.  
Following these patterns leads to better abstractions, easier testing,  
and more robust software architecture in Go applications.  
