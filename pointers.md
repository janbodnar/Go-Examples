# Pointers

Pointers in Go are variables that store memory addresses of other variables,  
enabling indirect access and manipulation of data. They provide powerful  
mechanisms for efficient memory usage, parameter passing, and building complex  
data structures like linked lists and trees. Understanding pointers is essential  
for effective Go programming, particularly when working with large data  
structures, implementing algorithms, or building performance-critical applications.  

A pointer holds the memory address where a value is stored, rather than the  
value itself. The ampersand operator (`&`) retrieves the address of a variable,  
creating a pointer to that variable. The asterisk operator (`*`) dereferences  
a pointer, allowing access to the value stored at the memory address the  
pointer contains.  

Go pointers are safer than pointers in languages like C or C++. Go doesn't  
support pointer arithmetic, which prevents many common programming errors  
such as buffer overflows and dangling pointer references. The Go runtime's  
garbage collector automatically manages memory, reducing the risk of memory  
leaks and invalid memory access.  

The zero value of a pointer is `nil`, representing a pointer that doesn't  
point to any valid memory address. Dereferencing a nil pointer causes a  
runtime panic, so proper nil checking is important for robust programs.  
Pointers enable efficient parameter passing by reference, allowing functions  
to modify the original data without copying large values.  

Pointers are particularly useful with structs, slices, maps, and channels,  
as these types are often passed by reference to avoid expensive copying  
operations. Method receivers can use pointers to modify the receiver's state  
or to handle large receiver types efficiently. Understanding when and how  
to use pointers effectively is crucial for writing idiomatic and performant  
Go code.  


## Basic pointer declaration

Basic pointer declarations demonstrate fundamental pointer syntax and the  
relationship between variables and their memory addresses.  

```go
package main

import "fmt"

func main() {
    var x int = 42
    var p *int = &x
    
    fmt.Printf("x = %d\n", x)
    fmt.Printf("Address of x = %p\n", &x)
    fmt.Printf("p = %p\n", p)
    fmt.Printf("Value at p = %d\n", *p)
    
    // Short declaration syntax
    y := 100
    q := &y
    
    fmt.Printf("y = %d\n", y)
    fmt.Printf("q points to = %d\n", *q)
}
```

This example shows how to declare pointers using both explicit and short  
declaration syntax. The `*int` type indicates a pointer to an integer,  
while `&x` creates a pointer to variable `x`. The `*p` syntax dereferences  
the pointer to access the underlying value.  

## Modifying values through pointers

Pointers enable indirect modification of variables, allowing functions to  
change values without returning new ones.  

```go
package main

import "fmt"

func main() {
    x := 10
    p := &x
    
    fmt.Printf("Before: x = %d\n", x)
    
    *p = 25  // Modify x through pointer p
    fmt.Printf("After: x = %d\n", x)
    
    // Multiple pointers to same variable
    q := &x
    *q = 50
    
    fmt.Printf("Modified via q: x = %d\n", x)
    fmt.Printf("Both pointers: *p = %d, *q = %d\n", *p, *q)
}
```

The example demonstrates how modifying a value through one pointer affects  
all pointers pointing to the same memory location. This behavior is  
fundamental to understanding how pointers share references to data.  

## Nil pointers and safety

Nil pointers represent uninitialized or empty pointer values and require  
careful handling to prevent runtime panics.  

```go
package main

import "fmt"

func main() {
    var p *int
    
    fmt.Printf("Uninitialized pointer: %v\n", p)
    fmt.Printf("Is nil? %t\n", p == nil)
    
    // Safe dereferencing with nil check
    if p != nil {
        fmt.Printf("Value: %d\n", *p)
    } else {
        fmt.Println("Cannot dereference nil pointer")
    }
    
    // Initialize the pointer
    x := 42
    p = &x
    
    if p != nil {
        fmt.Printf("Now points to: %d\n", *p)
    }
    
    // Setting pointer back to nil
    p = nil
    fmt.Printf("Reset to nil: %v\n", p)
}
```

Always check for nil before dereferencing pointers to avoid runtime panics.  
This example shows proper nil checking patterns and how to safely work  
with potentially uninitialized pointers.  

## Pointers to structs

Struct pointers enable efficient manipulation of complex data structures  
without expensive copying operations.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    person := Person{Name: "Alice", Age: 30}
    p := &person
    
    fmt.Printf("Original: %+v\n", person)
    fmt.Printf("Via pointer: %+v\n", *p)
    
    // Modify through pointer
    p.Name = "Bob"  // Equivalent to (*p).Name = "Bob"
    p.Age = 25
    
    fmt.Printf("Modified: %+v\n", person)
    
    // Creating pointer directly
    q := &Person{Name: "Charlie", Age: 35}
    fmt.Printf("Direct creation: %+v\n", *q)
}
```

Go automatically dereferences struct pointers when accessing fields, making  
`p.Name` equivalent to `(*p).Name`. This syntactic sugar makes struct  
pointer usage more readable and intuitive.  

## Function parameters with pointers

Pointers as function parameters enable functions to modify caller variables  
and avoid expensive copying of large data structures.  

```go
package main

import "fmt"

func increment(x *int) {
    *x++
}

func swap(a, b *int) {
    *a, *b = *b, *a
}

func updatePerson(p *Person) {
    p.Age++
    p.Name = "Updated " + p.Name
}

type Person struct {
    Name string
    Age  int
}

func main() {
    num := 5
    fmt.Printf("Before increment: %d\n", num)
    increment(&num)
    fmt.Printf("After increment: %d\n", num)
    
    x, y := 10, 20
    fmt.Printf("Before swap: x=%d, y=%d\n", x, y)
    swap(&x, &y)
    fmt.Printf("After swap: x=%d, y=%d\n", x, y)
    
    person := Person{Name: "Alice", Age: 30}
    fmt.Printf("Before update: %+v\n", person)
    updatePerson(&person)
    fmt.Printf("After update: %+v\n", person)
}
```

Functions receiving pointers can modify the original variables, enabling  
efficient in-place updates. This pattern is essential for performance  
when working with large structs or when multiple modifications are needed.  

## Pointer receivers in methods

Method receivers can use pointers to modify the receiver or handle large  
receiver types efficiently.  

```go
package main

import (
    "fmt"
    "math"
)

type Circle struct {
    Radius float64
}

// Value receiver - works with copy
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

// Pointer receiver - works with original
func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}

func (c *Circle) Reset() {
    c.Radius = 0
}

func main() {
    circle := Circle{Radius: 5}
    fmt.Printf("Original: %+v\n", circle)
    fmt.Printf("Area: %.2f\n", circle.Area())
    
    // Methods with pointer receivers
    circle.Scale(2)
    fmt.Printf("After scaling: %+v\n", circle)
    
    circle.Reset()
    fmt.Printf("After reset: %+v\n", circle)
    
    // Works with pointer to struct too
    p := &Circle{Radius: 10}
    p.Scale(1.5)
    fmt.Printf("Pointer method: %+v\n", *p)
}
```

Pointer receivers enable methods to modify the receiver's state and are  
more efficient for large structs. Go automatically handles the conversion  
between values and pointers when calling methods.  

## Returning pointers from functions

Functions can return pointers to enable caller access to locally created  
data, though this requires careful consideration of memory management.  

```go
package main

import "fmt"

type Point struct {
    X, Y int
}

func createPoint(x, y int) *Point {
    p := Point{X: x, Y: y}  // Local variable
    return &p               // Return pointer to local variable
}

func newPoint(x, y int) *Point {
    return &Point{X: x, Y: y}  // Direct allocation
}

func main() {
    // Both patterns work - Go handles memory allocation
    p1 := createPoint(1, 2)
    p2 := newPoint(3, 4)
    
    fmt.Printf("p1: %+v\n", *p1)
    fmt.Printf("p2: %+v\n", *p2)
    
    // Modify through returned pointers
    p1.X = 10
    p2.Y = 20
    
    fmt.Printf("Modified p1: %+v\n", *p1)
    fmt.Printf("Modified p2: %+v\n", *p2)
}
```

Go's garbage collector ensures that memory referenced by returned pointers  
remains valid, even when pointing to variables that would be out of scope  
in other languages. This enables flexible memory management patterns.  

## Pointers to arrays

Array pointers provide direct access to array memory, enabling efficient  
manipulation of fixed-size collections.  

```go
package main

import "fmt"

func modifyArray(arr *[5]int) {
    for i := range *arr {
        (*arr)[i] *= 2
    }
}

func printArray(arr *[5]int) {
    for i, v := range *arr {
        fmt.Printf("Index %d: %d\n", i, v)
    }
}

func main() {
    numbers := [5]int{1, 2, 3, 4, 5}
    fmt.Println("Original array:")
    printArray(&numbers)
    
    modifyArray(&numbers)
    fmt.Println("Modified array:")
    printArray(&numbers)
    
    // Pointer to array literal
    ptr := &[3]string{"hello", "there", "world"}
    fmt.Printf("Array via pointer: %v\n", *ptr)
}
```

Array pointers require dereferencing to access elements, using `(*arr)[i]`  
syntax. This pattern is less common than slice usage but useful when  
fixed-size arrays are specifically required.  

## Pointers to slices

While slices are reference types, pointers to slices enable modification  
of slice headers including length and capacity.  

```go
package main

import "fmt"

func appendToSlice(s *[]int, value int) {
    *s = append(*s, value)
}

func modifySlice(s *[]int) {
    for i := range *s {
        (*s)[i] *= 10
    }
}

func resetSlice(s *[]int) {
    *s = (*s)[:0]  // Keep capacity, reset length
}

func main() {
    numbers := []int{1, 2, 3}
    fmt.Printf("Original: %v (len=%d, cap=%d)\n", 
               numbers, len(numbers), cap(numbers))
    
    appendToSlice(&numbers, 4)
    appendToSlice(&numbers, 5)
    fmt.Printf("After append: %v (len=%d, cap=%d)\n", 
               numbers, len(numbers), cap(numbers))
    
    modifySlice(&numbers)
    fmt.Printf("After modify: %v\n", numbers)
    
    resetSlice(&numbers)
    fmt.Printf("After reset: %v (len=%d, cap=%d)\n", 
               numbers, len(numbers), cap(numbers))
}
```

Pointers to slices are useful when you need to modify the slice header  
itself, such as when growing or shrinking slices within functions.  
Regular slice parameters allow modification of elements but not the header.  

## Pointer to pointer (double pointers)

Double pointers store addresses of other pointers, enabling indirect  
pointer manipulation useful in advanced data structures.  

```go
package main

import "fmt"

func changePointer(pp **int) {
    y := 100
    *pp = &y  // Change what the pointer points to
}

func modifyThroughDouble(pp **int) {
    **pp = 200  // Modify the value indirectly
}

func main() {
    x := 42
    p := &x   // Pointer to x
    pp := &p  // Pointer to pointer
    
    fmt.Printf("x = %d\n", x)
    fmt.Printf("*p = %d\n", *p)
    fmt.Printf("**pp = %d\n", **pp)
    
    // Modify value through double pointer
    **pp = 75
    fmt.Printf("After **pp = 75: x = %d\n", x)
    
    // Change what p points to
    z := 99
    *pp = &z
    fmt.Printf("After redirect: *p = %d\n", *p)
    fmt.Printf("Original x still = %d\n", x)
    
    // Function modifying pointer target
    a := 10
    ptr := &a
    ptrToPtr := &ptr
    changePointer(ptrToPtr)
    fmt.Printf("After changePointer: *ptr = %d\n", *ptr)
}
```

Double pointers are particularly useful in linked list implementations  
and when functions need to modify pointer variables themselves, not  
just the values they point to.  

## Pointers in linked lists

Linked list implementation demonstrates practical pointer usage for  
building dynamic data structures with pointer-based connections.  

```go
package main

import "fmt"

type Node struct {
    Data int
    Next *Node
}

type LinkedList struct {
    Head *Node
}

func (ll *LinkedList) Insert(data int) {
    newNode := &Node{Data: data, Next: ll.Head}
    ll.Head = newNode
}

func (ll *LinkedList) Display() {
    current := ll.Head
    for current != nil {
        fmt.Printf("%d -> ", current.Data)
        current = current.Next
    }
    fmt.Println("nil")
}

func (ll *LinkedList) Delete(data int) {
    if ll.Head == nil {
        return
    }
    
    if ll.Head.Data == data {
        ll.Head = ll.Head.Next
        return
    }
    
    current := ll.Head
    for current.Next != nil && current.Next.Data != data {
        current = current.Next
    }
    
    if current.Next != nil {
        current.Next = current.Next.Next
    }
}

func main() {
    ll := &LinkedList{}
    
    ll.Insert(3)
    ll.Insert(2)
    ll.Insert(1)
    
    fmt.Print("List: ")
    ll.Display()
    
    ll.Delete(2)
    fmt.Print("After deleting 2: ")
    ll.Display()
}
```

Linked lists showcase fundamental pointer patterns: connecting nodes  
through pointer fields, traversing using pointer following, and  
manipulating connections by updating pointer values.  

## Pointers with maps

Map pointers enable functions to modify maps and provide efficient  
access to large map structures without copying.  

```go
package main

import "fmt"

func addToMap(m *map[string]int, key string, value int) {
    (*m)[key] = value
}

func deleteFromMap(m *map[string]int, key string) {
    delete(*m, key)
}

func processMap(m *map[string]int) {
    for k, v := range *m {
        (*m)[k] = v * 2
    }
}

func main() {
    scores := make(map[string]int)
    
    addToMap(&scores, "Alice", 85)
    addToMap(&scores, "Bob", 90)
    addToMap(&scores, "Charlie", 78)
    
    fmt.Printf("Original scores: %v\n", scores)
    
    processMap(&scores)
    fmt.Printf("Doubled scores: %v\n", scores)
    
    deleteFromMap(&scores, "Bob")
    fmt.Printf("After deletion: %v\n", scores)
    
    // Pointer to map literal
    ptr := &map[string]string{
        "name": "John",
        "city": "New York",
    }
    fmt.Printf("Map via pointer: %v\n", *ptr)
}
```

While maps are reference types, using pointers to maps can be useful  
when you need to reassign the entire map or when working with functions  
that expect consistent pointer-based interfaces.  

## Pointers with channels

Channel pointers enable functions to modify channel variables and  
work with channels as first-class values in data structures.  

```go
package main

import (
    "fmt"
    "time"
)

func sendData(ch *chan int, data int) {
    *ch <- data
}

func closeChannel(ch *chan int) {
    close(*ch)
}

func processChannel(ch *chan int) {
    for value := range *ch {
        fmt.Printf("Received: %d\n", value)
    }
}

func main() {
    ch := make(chan int, 3)
    
    // Send data through pointer
    sendData(&ch, 10)
    sendData(&ch, 20)
    sendData(&ch, 30)
    
    // Process channel through pointer
    go processChannel(&ch)
    
    time.Sleep(100 * time.Millisecond)
    closeChannel(&ch)
    
    time.Sleep(100 * time.Millisecond)
    
    // Array of channel pointers
    channels := []*chan string{
        func() *chan string { c := make(chan string, 1); return &c }(),
        func() *chan string { c := make(chan string, 1); return &c }(),
    }
    
    *channels[0] <- "hello"
    *channels[1] <- "there"
    
    fmt.Printf("From channel 0: %s\n", <-*channels[0])
    fmt.Printf("From channel 1: %s\n", <-*channels[1])
}
```

Channel pointers are useful when building complex communication patterns  
or when channels need to be stored in data structures and modified  
through function calls.  

## Pointer type assertions

Type assertions with pointers enable safe conversion between interface  
types and specific pointer types.  

```go
package main

import "fmt"

type Shape interface {
    Area() float64
}

type Rectangle struct {
    Width, Height float64
}

func (r *Rectangle) Area() float64 {
    return r.Width * r.Height
}

type Circle struct {
    Radius float64
}

func (c *Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func processShape(s Shape) {
    fmt.Printf("Area: %.2f\n", s.Area())
    
    // Type assertion to pointer type
    if rect, ok := s.(*Rectangle); ok {
        fmt.Printf("Rectangle dimensions: %.2f x %.2f\n", 
                   rect.Width, rect.Height)
        rect.Width *= 2  // Modify through pointer
        fmt.Printf("Modified width: %.2f\n", rect.Width)
    }
    
    if circle, ok := s.(*Circle); ok {
        fmt.Printf("Circle radius: %.2f\n", circle.Radius)
        circle.Radius *= 1.5  // Modify through pointer
        fmt.Printf("Modified radius: %.2f\n", circle.Radius)
    }
}

func main() {
    rect := &Rectangle{Width: 5, Height: 3}
    circle := &Circle{Radius: 4}
    
    shapes := []Shape{rect, circle}
    
    for _, shape := range shapes {
        processShape(shape)
        fmt.Println()
    }
}
```

Type assertions with pointers allow safe access to underlying concrete  
types while maintaining the ability to modify the original objects  
through their pointer interfaces.  

## Memory allocation with new

The `new` function allocates memory and returns a pointer to the newly  
allocated zero value of the specified type.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    // new with basic types
    p := new(int)
    fmt.Printf("new(int): %p, value: %d\n", p, *p)
    
    *p = 42
    fmt.Printf("After assignment: %d\n", *p)
    
    // new with struct
    person := new(Person)
    fmt.Printf("new(Person): %+v\n", *person)
    
    person.Name = "Alice"
    person.Age = 30
    fmt.Printf("Modified: %+v\n", *person)
    
    // Compare with literal allocation
    person2 := &Person{}
    person3 := &Person{Name: "Bob", Age: 25}
    
    fmt.Printf("Empty literal: %+v\n", *person2)
    fmt.Printf("Initialized literal: %+v\n", *person3)
    
    // new with slices creates pointer to nil slice
    s := new([]int)
    fmt.Printf("new([]int): %v, is nil: %t\n", *s, *s == nil)
    
    *s = append(*s, 1, 2, 3)
    fmt.Printf("After append: %v\n", *s)
}
```

The `new` function is less commonly used than literal allocation with `&`,  
but it's useful when you need a pointer to a zero value or when the  
type is determined dynamically.  

## Pointer arithmetic limitations

Go restricts pointer arithmetic to prevent unsafe memory access,  
demonstrating the language's safety-first approach.  

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    arr := [5]int{1, 2, 3, 4, 5}
    
    // This would NOT compile - pointer arithmetic not allowed
    // p := &arr[0]
    // p++  // Error: invalid operation
    
    // Safe alternative: use slice indexing
    p := &arr[0]
    fmt.Printf("First element: %d\n", *p)
    
    // Access other elements safely
    for i := range arr {
        ptr := &arr[i]
        fmt.Printf("arr[%d] = %d (addr: %p)\n", i, *ptr, ptr)
    }
    
    // unsafe package allows pointer arithmetic (not recommended)
    unsafePtr := unsafe.Pointer(&arr[0])
    
    // Convert to uintptr for arithmetic
    addr := uintptr(unsafePtr)
    fmt.Printf("Base address: %x\n", addr)
    
    // This is dangerous and not recommended
    secondElementAddr := addr + unsafe.Sizeof(arr[0])
    secondPtr := (*int)(unsafe.Pointer(secondElementAddr))
    fmt.Printf("Second element via unsafe: %d\n", *secondPtr)
    
    fmt.Println("Note: unsafe operations should be avoided in normal code")
}
```

Go's restriction on pointer arithmetic prevents buffer overflows and  
other memory safety issues. Use slice indexing or other safe patterns  
instead of attempting pointer arithmetic.  

## Function pointers and callbacks

Function pointers enable callback patterns and higher-order programming  
constructs in Go applications.  

```go
package main

import "fmt"

type Operation func(int, int) int

func add(a, b int) int {
    return a + b
}

func multiply(a, b int) int {
    return a * b
}

func applyOperation(op Operation, x, y int) int {
    return op(x, y)
}

type Calculator struct {
    Operation Operation
}

func (c *Calculator) SetOperation(op Operation) {
    c.Operation = op
}

func (c *Calculator) Calculate(a, b int) int {
    if c.Operation != nil {
        return c.Operation(a, b)
    }
    return 0
}

func main() {
    // Function variables
    var op Operation = add
    result := applyOperation(op, 5, 3)
    fmt.Printf("5 + 3 = %d\n", result)
    
    // Change function pointer
    op = multiply
    result = applyOperation(op, 5, 3)
    fmt.Printf("5 * 3 = %d\n", result)
    
    // Calculator with function pointer
    calc := &Calculator{}
    calc.SetOperation(add)
    fmt.Printf("Calculator add: %d\n", calc.Calculate(10, 20))
    
    calc.SetOperation(multiply)
    fmt.Printf("Calculator multiply: %d\n", calc.Calculate(10, 20))
    
    // Anonymous function
    calc.SetOperation(func(a, b int) int {
        return a - b
    })
    fmt.Printf("Calculator subtract: %d\n", calc.Calculate(20, 5))
    
    // Array of function pointers
    operations := []Operation{add, multiply}
    for i, op := range operations {
        result := op(4, 6)
        fmt.Printf("Operation %d: %d\n", i, result)
    }
}
```

Function pointers provide flexibility for implementing strategy patterns,  
callbacks, and configurable behavior in Go programs while maintaining  
type safety.  

## Pointers in goroutines

Sharing pointers between goroutines requires careful synchronization  
to prevent race conditions and ensure thread safety.  

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

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Get() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

func worker(id int, counter *Counter, wg *sync.WaitGroup) {
    defer wg.Done()
    
    for i := 0; i < 1000; i++ {
        counter.Increment()
    }
    fmt.Printf("Worker %d completed\n", id)
}

func main() {
    counter := &Counter{}
    var wg sync.WaitGroup
    
    // Start multiple goroutines sharing counter pointer
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go worker(i, counter, &wg)
    }
    
    wg.Wait()
    fmt.Printf("Final counter value: %d\n", counter.Get())
    
    // Channel-based pointer sharing
    type Message struct {
        ID   int
        Data string
    }
    
    msgChan := make(chan *Message, 5)
    
    // Producer
    go func() {
        for i := 0; i < 3; i++ {
            msg := &Message{ID: i, Data: fmt.Sprintf("Message %d", i)}
            msgChan <- msg
        }
        close(msgChan)
    }()
    
    // Consumer
    for msg := range msgChan {
        fmt.Printf("Received: %+v\n", *msg)
        time.Sleep(100 * time.Millisecond)
    }
}
```

When sharing pointers between goroutines, use synchronization primitives  
like mutexes or communicate through channels to ensure safe concurrent  
access and prevent data races.  

## Circular references and cleanup

Circular references can create memory management challenges, requiring  
careful design to prevent resource leaks.  

```go
package main

import (
    "fmt"
    "runtime"
)

type Node struct {
    ID       int
    Children []*Node
    Parent   *Node
}

func (n *Node) AddChild(child *Node) {
    child.Parent = n
    n.Children = append(n.Children, child)
}

func (n *Node) RemoveChild(childID int) {
    for i, child := range n.Children {
        if child.ID == childID {
            // Break circular reference
            child.Parent = nil
            // Remove from slice
            n.Children = append(n.Children[:i], n.Children[i+1:]...)
            break
        }
    }
}

func (n *Node) Cleanup() {
    // Break all circular references
    for _, child := range n.Children {
        child.Parent = nil
        child.Cleanup() // Recursively cleanup
    }
    n.Children = nil
    n.Parent = nil
}

func createTree() *Node {
    root := &Node{ID: 1}
    child1 := &Node{ID: 2}
    child2 := &Node{ID: 3}
    grandchild := &Node{ID: 4}
    
    root.AddChild(child1)
    root.AddChild(child2)
    child1.AddChild(grandchild)
    
    return root
}

func printMemStats(label string) {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    fmt.Printf("%s - Alloc: %d KB, NumGC: %d\n", 
               label, m.Alloc/1024, m.NumGC)
}

func main() {
    printMemStats("Initial")
    
    // Create tree structure
    root := createTree()
    fmt.Printf("Root has %d children\n", len(root.Children))
    
    printMemStats("After creation")
    
    // Navigate tree
    if len(root.Children) > 0 {
        child := root.Children[0]
        fmt.Printf("Child %d has parent %d\n", child.ID, child.Parent.ID)
        
        if len(child.Children) > 0 {
            grandchild := child.Children[0]
            fmt.Printf("Grandchild %d has parent %d\n", 
                       grandchild.ID, grandchild.Parent.ID)
        }
    }
    
    // Cleanup circular references
    root.Cleanup()
    root = nil
    
    runtime.GC()
    printMemStats("After cleanup")
    
    fmt.Println("Tree cleaned up successfully")
}
```

Go's garbage collector can handle circular references, but explicit cleanup  
can be beneficial for large data structures or when deterministic cleanup  
timing is important.  

## Pointer performance considerations

Understanding pointer performance characteristics helps optimize memory  
usage and access patterns in Go programs.  

```go
package main

import (
    "fmt"
    "time"
)

type LargeStruct struct {
    Data [1000]int
    Name string
    ID   int
}

func processValueCopy(ls LargeStruct) int {
    sum := 0
    for _, v := range ls.Data {
        sum += v
    }
    return sum
}

func processValuePointer(ls *LargeStruct) int {
    sum := 0
    for _, v := range ls.Data {
        sum += v
    }
    return sum
}

func benchmark(name string, fn func()) {
    start := time.Now()
    fn()
    duration := time.Since(start)
    fmt.Printf("%s took: %v\n", name, duration)
}

func main() {
    // Initialize large struct
    ls := LargeStruct{Name: "Test", ID: 1}
    for i := range ls.Data {
        ls.Data[i] = i
    }
    
    iterations := 100000
    
    // Benchmark value copying
    benchmark("Value copy", func() {
        for i := 0; i < iterations; i++ {
            processValueCopy(ls)
        }
    })
    
    // Benchmark pointer passing
    benchmark("Pointer passing", func() {
        for i := 0; i < iterations; i++ {
            processValuePointer(&ls)
        }
    })
    
    // Memory allocation patterns
    start := time.Now()
    var structs []*LargeStruct
    for i := 0; i < 1000; i++ {
        s := &LargeStruct{ID: i}
        structs = append(structs, s)
    }
    fmt.Printf("Pointer allocation took: %v\n", time.Since(start))
    
    start = time.Now()
    var values []LargeStruct
    for i := 0; i < 1000; i++ {
        s := LargeStruct{ID: i}
        values = append(values, s)
    }
    fmt.Printf("Value allocation took: %v\n", time.Since(start))
    
    fmt.Printf("Created %d pointer structs and %d value structs\n", 
               len(structs), len(values))
}
```

Pointers provide significant performance benefits when working with large  
structs by avoiding expensive copy operations. However, consider memory  
locality and garbage collection overhead when designing data structures.  

## Advanced pointer patterns

Advanced pointer patterns demonstrate sophisticated usage in complex  
data structures and algorithms.  

```go
package main

import "fmt"

// Generic node structure for various data structures
type TreeNode struct {
    Value int
    Left  *TreeNode
    Right *TreeNode
}

// Binary search tree operations
func (root *TreeNode) Insert(value int) *TreeNode {
    if root == nil {
        return &TreeNode{Value: value}
    }
    
    if value < root.Value {
        root.Left = root.Left.Insert(value)
    } else if value > root.Value {
        root.Right = root.Right.Insert(value)
    }
    return root
}

func (root *TreeNode) Search(value int) *TreeNode {
    if root == nil || root.Value == value {
        return root
    }
    
    if value < root.Value {
        return root.Left.Search(value)
    }
    return root.Right.Search(value)
}

func (root *TreeNode) InorderTraversal() {
    if root != nil {
        root.Left.InorderTraversal()
        fmt.Printf("%d ", root.Value)
        root.Right.InorderTraversal()
    }
}

// Self-referential structure with weak references
type Cache struct {
    data map[string]*CacheEntry
    head *CacheEntry
    tail *CacheEntry
}

type CacheEntry struct {
    key   string
    value interface{}
    prev  *CacheEntry
    next  *CacheEntry
}

func NewCache() *Cache {
    head := &CacheEntry{}
    tail := &CacheEntry{}
    head.next = tail
    tail.prev = head
    
    return &Cache{
        data: make(map[string]*CacheEntry),
        head: head,
        tail: tail,
    }
}

func (c *Cache) addToFront(entry *CacheEntry) {
    entry.next = c.head.next
    entry.prev = c.head
    c.head.next.prev = entry
    c.head.next = entry
}

func (c *Cache) removeEntry(entry *CacheEntry) {
    entry.prev.next = entry.next
    entry.next.prev = entry.prev
}

func (c *Cache) Put(key string, value interface{}) {
    if entry, exists := c.data[key]; exists {
        entry.value = value
        c.removeEntry(entry)
        c.addToFront(entry)
    } else {
        entry := &CacheEntry{key: key, value: value}
        c.data[key] = entry
        c.addToFront(entry)
    }
}

func (c *Cache) Get(key string) (interface{}, bool) {
    if entry, exists := c.data[key]; exists {
        c.removeEntry(entry)
        c.addToFront(entry)
        return entry.value, true
    }
    return nil, false
}

func main() {
    // Binary search tree example
    var root *TreeNode
    values := []int{50, 30, 70, 20, 40, 60, 80}
    
    for _, v := range values {
        root = root.Insert(v)
    }
    
    fmt.Print("BST Inorder traversal: ")
    root.InorderTraversal()
    fmt.Println()
    
    // Search operations
    if node := root.Search(40); node != nil {
        fmt.Printf("Found value: %d\n", node.Value)
    }
    
    if node := root.Search(100); node == nil {
        fmt.Println("Value 100 not found")
    }
    
    // LRU Cache example
    cache := NewCache()
    cache.Put("a", 1)
    cache.Put("b", 2)
    cache.Put("c", 3)
    
    if value, found := cache.Get("a"); found {
        fmt.Printf("Cache hit: %v\n", value)
    }
    
    cache.Put("d", 4)
    fmt.Println("Advanced pointer patterns demonstrated successfully")
}
```

Advanced pointer patterns enable implementation of sophisticated data  
structures like binary search trees, doubly-linked lists, and LRU caches.  
These patterns showcase the power and flexibility of Go's pointer system.  

## Pointer best practices and safety

Following established best practices ensures safe and effective pointer  
usage while avoiding common pitfalls and security issues.  

```go
package main

import (
    "errors"
    "fmt"
)

type SafePointer struct {
    ptr   *int
    valid bool
}

func NewSafePointer(value int) *SafePointer {
    return &SafePointer{
        ptr:   &value,
        valid: true,
    }
}

func (sp *SafePointer) Get() (int, error) {
    if !sp.valid || sp.ptr == nil {
        return 0, errors.New("invalid pointer")
    }
    return *sp.ptr, nil
}

func (sp *SafePointer) Set(value int) error {
    if !sp.valid || sp.ptr == nil {
        return errors.New("invalid pointer")
    }
    *sp.ptr = value
    return nil
}

func (sp *SafePointer) Invalidate() {
    sp.valid = false
    sp.ptr = nil
}

// Defensive programming with pointers
func safeProcessData(data *[]int) error {
    // Always validate input pointers
    if data == nil {
        return errors.New("data pointer is nil")
    }
    
    if *data == nil {
        return errors.New("data slice is nil")
    }
    
    // Safe operations on validated pointer
    for i := range *data {
        (*data)[i] *= 2
    }
    
    return nil
}

// Resource management with pointers
type Resource struct {
    id     int
    active bool
}

func NewResource(id int) *Resource {
    r := &Resource{id: id, active: true}
    fmt.Printf("Resource %d created\n", id)
    return r
}

func (r *Resource) Close() {
    if r != nil && r.active {
        r.active = false
        fmt.Printf("Resource %d closed\n", r.id)
    }
}

func (r *Resource) Use() error {
    if r == nil {
        return errors.New("resource is nil")
    }
    if !r.active {
        return errors.New("resource is closed")
    }
    
    fmt.Printf("Using resource %d\n", r.id)
    return nil
}

// RAII-like pattern with defer
func useResource(id int) error {
    resource := NewResource(id)
    if resource == nil {
        return errors.New("failed to create resource")
    }
    defer resource.Close()
    
    return resource.Use()
}

func main() {
    // Safe pointer usage
    sp := NewSafePointer(42)
    
    if value, err := sp.Get(); err == nil {
        fmt.Printf("Safe value: %d\n", value)
    }
    
    sp.Set(100)
    if value, err := sp.Get(); err == nil {
        fmt.Printf("Updated safe value: %d\n", value)
    }
    
    sp.Invalidate()
    if _, err := sp.Get(); err != nil {
        fmt.Printf("Expected error: %v\n", err)
    }
    
    // Defensive programming
    data := []int{1, 2, 3, 4, 5}
    if err := safeProcessData(&data); err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Processed data: %v\n", data)
    }
    
    // Nil pointer safety
    var nilData *[]int
    if err := safeProcessData(nilData); err != nil {
        fmt.Printf("Nil check caught: %v\n", err)
    }
    
    // Resource management
    if err := useResource(1); err != nil {
        fmt.Printf("Resource error: %v\n", err)
    }
    
    fmt.Println("\nBest practices:")
    fmt.Println("1. Always check for nil before dereferencing")
    fmt.Println("2. Use defensive programming patterns")
    fmt.Println("3. Implement proper resource cleanup")
    fmt.Println("4. Consider using safe wrapper types")
    fmt.Println("5. Document pointer ownership and lifetime")
}
```

Following pointer best practices prevents common errors like nil pointer  
dereferences, resource leaks, and unsafe memory access. Always validate  
pointers, implement proper cleanup, and use defensive programming patterns  
for robust applications.  

## Pointer slicing and subslices

Pointer-based slice operations demonstrate advanced memory management  
and efficient sub-slice creation without data copying.  

```go
package main

import "fmt"

func createSubSlicePointer(data *[]int, start, end int) *[]int {
    if data == nil || *data == nil {
        return nil
    }
    
    length := len(*data)
    if start < 0 || end > length || start > end {
        return nil
    }
    
    subSlice := (*data)[start:end]
    return &subSlice
}

func modifyThroughPointer(slicePtr *[]int, multiplier int) {
    if slicePtr == nil || *slicePtr == nil {
        return
    }
    
    for i := range *slicePtr {
        (*slicePtr)[i] *= multiplier
    }
}

func appendSafe(slicePtr *[]int, values ...int) {
    if slicePtr == nil {
        return
    }
    
    *slicePtr = append(*slicePtr, values...)
}

func main() {
    original := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    fmt.Printf("Original: %v\n", original)
    
    // Create sub-slice through pointer
    subPtr := createSubSlicePointer(&original, 2, 6)
    if subPtr != nil {
        fmt.Printf("Sub-slice: %v\n", *subPtr)
        
        // Modify through pointer affects original
        modifyThroughPointer(subPtr, 100)
        fmt.Printf("After modification: %v\n", *subPtr)
        fmt.Printf("Original affected: %v\n", original)
    }
    
    // Safe append operations
    newSlice := []int{1, 2, 3}
    fmt.Printf("Before append: %v\n", newSlice)
    
    appendSafe(&newSlice, 4, 5, 6)
    fmt.Printf("After append: %v\n", newSlice)
    
    // Working with slice of pointers
    values := []int{10, 20, 30}
    pointerSlice := make([]*int, len(values))
    
    for i := range values {
        pointerSlice[i] = &values[i]
    }
    
    fmt.Print("Slice of pointers: ")
    for _, ptr := range pointerSlice {
        fmt.Printf("%d ", *ptr)
    }
    fmt.Println()
    
    // Modify through pointer slice
    for _, ptr := range pointerSlice {
        *ptr *= 2
    }
    
    fmt.Printf("Modified original values: %v\n", values)
}
```

Pointer-based slice operations enable sophisticated memory management  
patterns while maintaining efficiency and avoiding unnecessary data  
copying in performance-critical applications.  

## Interface pointers and polymorphism

Interface pointers enable polymorphic behavior while maintaining the  
ability to modify underlying concrete types through their interfaces.  

```go
package main

import (
    "fmt"
    "math"
)

type Drawable interface {
    Draw()
    Area() float64
    Scale(factor float64)
}

type Rectangle struct {
    Width, Height float64
    X, Y          float64
}

func (r *Rectangle) Draw() {
    fmt.Printf("Drawing Rectangle at (%.1f,%.1f) with dimensions %.1fx%.1f\n",
               r.X, r.Y, r.Width, r.Height)
}

func (r *Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

type Circle struct {
    Radius float64
    X, Y   float64
}

func (c *Circle) Draw() {
    fmt.Printf("Drawing Circle at (%.1f,%.1f) with radius %.1f\n",
               c.X, c.Y, c.Radius)
}

func (c *Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c *Circle) Scale(factor float64) {
    c.Radius *= factor
}

type Canvas struct {
    shapes []Drawable
}

func (canvas *Canvas) AddShape(shape Drawable) {
    canvas.shapes = append(canvas.shapes, shape)
}

func (canvas *Canvas) DrawAll() {
    for _, shape := range canvas.shapes {
        shape.Draw()
    }
}

func (canvas *Canvas) ScaleAll(factor float64) {
    for _, shape := range canvas.shapes {
        shape.Scale(factor)
    }
}

func (canvas *Canvas) TotalArea() float64 {
    total := 0.0
    for _, shape := range canvas.shapes {
        total += shape.Area()
    }
    return total
}

func processDrawable(d *Drawable) {
    if d == nil || *d == nil {
        fmt.Println("Invalid drawable")
        return
    }
    
    fmt.Printf("Processing drawable with area: %.2f\n", (*d).Area())
    (*d).Draw()
}

func main() {
    // Create shapes using pointers
    rect := &Rectangle{Width: 10, Height: 5, X: 0, Y: 0}
    circle := &Circle{Radius: 3, X: 15, Y: 10}
    
    // Interface pointers
    var drawable Drawable = rect
    fmt.Printf("Rectangle area: %.2f\n", drawable.Area())
    drawable.Draw()
    
    drawable = circle
    fmt.Printf("Circle area: %.2f\n", drawable.Area())
    drawable.Draw()
    
    // Canvas with polymorphic shapes
    canvas := &Canvas{}
    canvas.AddShape(rect)
    canvas.AddShape(circle)
    canvas.AddShape(&Rectangle{Width: 8, Height: 3, X: 20, Y: 5})
    
    fmt.Println("\nOriginal canvas:")
    canvas.DrawAll()
    fmt.Printf("Total area: %.2f\n", canvas.TotalArea())
    
    // Scale all shapes
    canvas.ScaleAll(1.5)
    fmt.Println("\nScaled canvas:")
    canvas.DrawAll()
    fmt.Printf("Total area after scaling: %.2f\n", canvas.TotalArea())
    
    // Process drawable through pointer
    var drawablePtr Drawable = circle
    processDrawable(&drawablePtr)
}
```

Interface pointers combine Go's interface system with pointer semantics  
to enable powerful polymorphic patterns while maintaining efficiency  
and the ability to modify objects through their interface types.  

## Custom pointer types and method sets

Custom pointer types enable creation of specialized pointer behaviors  
and method sets for domain-specific applications.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Custom pointer type with reference counting
type RefPtr struct {
    ptr   *interface{}
    count *int
    mutex *sync.Mutex
}

func NewRefPtr(value interface{}) *RefPtr {
    count := 1
    return &RefPtr{
        ptr:   &value,
        count: &count,
        mutex: &sync.Mutex{},
    }
}

func (rp *RefPtr) Clone() *RefPtr {
    if rp == nil || rp.ptr == nil {
        return nil
    }
    
    rp.mutex.Lock()
    defer rp.mutex.Unlock()
    
    (*rp.count)++
    return &RefPtr{
        ptr:   rp.ptr,
        count: rp.count,
        mutex: rp.mutex,
    }
}

func (rp *RefPtr) Get() (interface{}, bool) {
    if rp == nil || rp.ptr == nil {
        return nil, false
    }
    
    rp.mutex.Lock()
    defer rp.mutex.Unlock()
    
    return *rp.ptr, true
}

func (rp *RefPtr) RefCount() int {
    if rp == nil || rp.count == nil {
        return 0
    }
    
    rp.mutex.Lock()
    defer rp.mutex.Unlock()
    
    return *rp.count
}

func (rp *RefPtr) Release() {
    if rp == nil || rp.count == nil {
        return
    }
    
    rp.mutex.Lock()
    defer rp.mutex.Unlock()
    
    (*rp.count)--
    if *rp.count <= 0 {
        fmt.Printf("Releasing shared resource (refcount: %d)\n", *rp.count)
        rp.ptr = nil
        rp.count = nil
    }
}

// Smart pointer for automatic cleanup
type SmartPtr struct {
    ptr     *interface{}
    cleanup func(interface{})
}

func NewSmartPtr(value interface{}, cleanup func(interface{})) *SmartPtr {
    return &SmartPtr{
        ptr:     &value,
        cleanup: cleanup,
    }
}

func (sp *SmartPtr) Get() (interface{}, bool) {
    if sp == nil || sp.ptr == nil {
        return nil, false
    }
    return *sp.ptr, true
}

func (sp *SmartPtr) Reset(value interface{}) {
    if sp == nil {
        return
    }
    
    // Cleanup old value
    if sp.ptr != nil && sp.cleanup != nil {
        sp.cleanup(*sp.ptr)
    }
    
    // Set new value
    sp.ptr = &value
}

func (sp *SmartPtr) Release() {
    if sp == nil || sp.ptr == nil {
        return
    }
    
    if sp.cleanup != nil {
        sp.cleanup(*sp.ptr)
    }
    sp.ptr = nil
    sp.cleanup = nil
}

// Weak pointer that doesn't own the resource
type WeakPtr struct {
    ptr   *interface{}
    valid *bool
}

func NewWeakPtr(refPtr *RefPtr) *WeakPtr {
    if refPtr == nil || refPtr.ptr == nil {
        return nil
    }
    
    valid := true
    return &WeakPtr{
        ptr:   refPtr.ptr,
        valid: &valid,
    }
}

func (wp *WeakPtr) IsValid() bool {
    return wp != nil && wp.valid != nil && *wp.valid && wp.ptr != nil
}

func (wp *WeakPtr) Get() (interface{}, bool) {
    if !wp.IsValid() {
        return nil, false
    }
    return *wp.ptr, true
}

func (wp *WeakPtr) Invalidate() {
    if wp != nil && wp.valid != nil {
        *wp.valid = false
    }
}

func main() {
    // Reference counted pointer
    fmt.Println("=== Reference Counted Pointer ===")
    refPtr1 := NewRefPtr("Shared Resource")
    fmt.Printf("Initial refcount: %d\n", refPtr1.RefCount())
    
    refPtr2 := refPtr1.Clone()
    refPtr3 := refPtr1.Clone()
    fmt.Printf("After cloning: %d\n", refPtr1.RefCount())
    
    if value, ok := refPtr2.Get(); ok {
        fmt.Printf("Value: %v\n", value)
    }
    
    refPtr2.Release()
    fmt.Printf("After first release: %d\n", refPtr1.RefCount())
    
    refPtr3.Release()
    refPtr1.Release()
    
    // Smart pointer with cleanup
    fmt.Println("\n=== Smart Pointer ===")
    cleanup := func(value interface{}) {
        fmt.Printf("Cleaning up: %v\n", value)
    }
    
    smartPtr := NewSmartPtr("Resource 1", cleanup)
    if value, ok := smartPtr.Get(); ok {
        fmt.Printf("Smart pointer value: %v\n", value)
    }
    
    smartPtr.Reset("Resource 2")
    smartPtr.Release()
    
    // Weak pointer
    fmt.Println("\n=== Weak Pointer ===")
    refPtr := NewRefPtr("Weak Test")
    weakPtr := NewWeakPtr(refPtr)
    
    fmt.Printf("Weak pointer valid: %t\n", weakPtr.IsValid())
    if value, ok := weakPtr.Get(); ok {
        fmt.Printf("Weak pointer value: %v\n", value)
    }
    
    refPtr.Release()
    weakPtr.Invalidate()
    fmt.Printf("After invalidation: %t\n", weakPtr.IsValid())
    
    fmt.Println("\nCustom pointer types provide specialized memory management")
}
```

Custom pointer types enable sophisticated memory management patterns  
including reference counting, automatic cleanup, and weak references,  
providing building blocks for complex resource management systems.