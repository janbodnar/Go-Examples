# Generics

Go generics, introduced in Go 1.18, represent one of the most significant  
additions to the language since its inception. Generics enable developers to  
write type-safe, reusable code that works with multiple types while maintaining  
compile-time type checking and runtime performance. This comprehensive guide  
explores every aspect of Go generics through detailed examples and explanations.

Generics solve the long-standing challenge of writing functions and data  
structures that work with different types without sacrificing type safety or  
performance. Before generics, developers had to choose between code duplication,  
runtime type assertions with interface{}, or complex reflection-based solutions.  
Generics provide a clean, efficient alternative that enables truly polymorphic  
code while preserving Go's emphasis on simplicity and performance.

The design of Go generics emphasizes readability and gradual adoption. Type  
parameters use square bracket syntax [T], making generic code visually distinct  
from regular Go code. Type constraints, expressed through interfaces, provide  
precise control over what operations are allowed on generic types. This design  
enables powerful abstractions while maintaining Go's commitment to explicit,  
understandable code.

Understanding generics is crucial for modern Go development. They enable more  
expressive APIs, reduce code duplication, improve type safety, and provide  
better performance than interface{}-based alternatives. Generic programming  
patterns are becoming standard in Go libraries and applications, making mastery  
of this feature essential for effective Go programming.

This guide progresses from fundamental concepts to advanced patterns, covering  
type parameters, constraints, generic functions, generic types, interface  
constraints, error handling patterns, concurrency integration, testing  
strategies, and real-world applications. Each example builds understanding  
through practical, executable code that demonstrates best practices and common  
patterns in generic Go programming.


## Basic type parameters

Type parameters form the foundation of Go generics, allowing functions and  
types to work with placeholder types that are specified when the generic  
code is instantiated.

```go
package main

import "fmt"

// Generic function with single type parameter
func Identity[T any](value T) T {
    return value
}

// Generic function with multiple type parameters
func Pair[T, U any](first T, second U) (T, U) {
    return first, second
}

// Generic function with same type constraint
func Swap[T any](a, b T) (T, T) {
    return b, a
}

func main() {
    // Type inference - Go infers the type from arguments
    intResult := Identity(42)
    stringResult := Identity("hello there")
    
    fmt.Printf("Identity(42): %v (type: %T)\n", intResult, intResult)
    fmt.Printf("Identity(\"hello there\"): %v (type: %T)\n", stringResult, stringResult)
    
    // Explicit type specification
    floatResult := Identity[float64](3.14)
    fmt.Printf("Identity[float64](3.14): %v (type: %T)\n", floatResult, floatResult)
    
    // Multiple type parameters
    name, age := Pair("Alice", 30)
    fmt.Printf("Pair: %s is %d years old\n", name, age)
    
    // Type inference with multiple parameters
    flag, message := Pair(true, "success")
    fmt.Printf("Pair: %v - %s\n", flag, message)
    
    // Swapping values
    x, y := Swap(10, 20)
    fmt.Printf("Swap(10, 20): %d, %d\n", x, y)
    
    a, b := Swap("first", "second")
    fmt.Printf("Swap(\"first\", \"second\"): %s, %s\n", a, b)
}
```

Type parameters enable writing functions that work with any type while  
maintaining compile-time type safety. The `any` constraint (alias for  
`interface{}`) allows the most general type parameter, while type inference  
reduces verbosity by automatically determining types from function arguments.  

## Generic functions with constraints

Type constraints restrict generic types to specific sets of types or require  
certain methods, enabling more sophisticated generic programming while  
maintaining type safety.

```go
package main

import "fmt"

// Numeric constraint for arithmetic operations
type Numeric interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64
}

// Ordered constraint for comparison operations
type Ordered interface {
    int | int8 | int16 | int32 | int64 |
    uint | uint8 | uint16 | uint32 | uint64 |
    float32 | float64 | string
}

// Stringer constraint for types with String method
type Stringer interface {
    String() string
}

// Generic arithmetic functions
func Add[T Numeric](a, b T) T {
    return a + b
}

func Multiply[T Numeric](a, b T) T {
    return a * b
}

// Generic comparison function
func Max[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func Min[T Ordered](slice []T) T {
    if len(slice) == 0 {
        var zero T
        return zero
    }
    
    min := slice[0]
    for _, v := range slice[1:] {
        if v < min {
            min = v
        }
    }
    return min
}

// Generic function with method constraint
func PrintFormatted[T Stringer](items []T) {
    for i, item := range items {
        fmt.Printf("Item %d: %s\n", i+1, item.String())
    }
}

// Custom type implementing Stringer
type Product struct {
    Name  string
    Price float64
}

func (p Product) String() string {
    return fmt.Sprintf("%s ($%.2f)", p.Name, p.Price)
}

func main() {
    // Numeric constraints
    fmt.Println("=== Numeric Operations ===")
    fmt.Printf("Add(10, 20): %d\n", Add(10, 20))
    fmt.Printf("Add(3.14, 2.86): %.2f\n", Add(3.14, 2.86))
    fmt.Printf("Multiply(5, 7): %d\n", Multiply(5, 7))
    fmt.Printf("Multiply(2.5, 4.0): %.2f\n", Multiply(2.5, 4.0))
    
    // Ordered constraints
    fmt.Println("\n=== Comparison Operations ===")
    fmt.Printf("Max(15, 25): %d\n", Max(15, 25))
    fmt.Printf("Max(\"apple\", \"banana\"): %s\n", Max("apple", "banana"))
    fmt.Printf("Max(3.14, 2.71): %.2f\n", Max(3.14, 2.71))
    
    intSlice := []int{5, 2, 8, 1, 9, 3}
    stringSlice := []string{"zebra", "apple", "banana", "cherry"}
    fmt.Printf("Min(%v): %d\n", intSlice, Min(intSlice))
    fmt.Printf("Min(%v): %s\n", stringSlice, Min(stringSlice))
    
    // Method constraints
    fmt.Println("\n=== Method Constraints ===")
    products := []Product{
        {"Laptop", 999.99},
        {"Mouse", 29.99},
        {"Keyboard", 79.99},
    }
    PrintFormatted(products)
}
```

Type constraints provide precise control over generic type parameters,  
enabling arithmetic operations, comparisons, and method calls while ensuring  
compile-time safety. Union constraints (using `|`) specify multiple allowed  
types, while interface constraints require specific method implementations.  

## Generic types and data structures

Generic types enable creating reusable data structures that work with any  
type while maintaining type safety and performance characteristics of  
statically typed code.

```go
package main

import "fmt"

// Generic stack data structure
type Stack[T any] struct {
    items []T
}

func NewStack[T any]() *Stack[T] {
    return &Stack[T]{
        items: make([]T, 0),
    }
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    
    index := len(s.items) - 1
    item := s.items[index]
    s.items = s.items[:index]
    return item, true
}

func (s *Stack[T]) Peek() (T, bool) {
    if len(s.items) == 0 {
        var zero T
        return zero, false
    }
    return s.items[len(s.items)-1], true
}

func (s *Stack[T]) Size() int {
    return len(s.items)
}

func (s *Stack[T]) IsEmpty() bool {
    return len(s.items) == 0
}

// Generic queue data structure
type Queue[T any] struct {
    items []T
}

func NewQueue[T any]() *Queue[T] {
    return &Queue[T]{
        items: make([]T, 0),
    }
}

func (q *Queue[T]) Enqueue(item T) {
    q.items = append(q.items, item)
}

func (q *Queue[T]) Dequeue() (T, bool) {
    if len(q.items) == 0 {
        var zero T
        return zero, false
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item, true
}

func (q *Queue[T]) Front() (T, bool) {
    if len(q.items) == 0 {
        var zero T
        return zero, false
    }
    return q.items[0], true
}

func (q *Queue[T]) Size() int {
    return len(q.items)
}

// Generic linked list node
type Node[T any] struct {
    Value T
    Next  *Node[T]
}

type LinkedList[T any] struct {
    head *Node[T]
    size int
}

func NewLinkedList[T any]() *LinkedList[T] {
    return &LinkedList[T]{}
}

func (ll *LinkedList[T]) Prepend(value T) {
    newNode := &Node[T]{Value: value, Next: ll.head}
    ll.head = newNode
    ll.size++
}

func (ll *LinkedList[T]) Append(value T) {
    newNode := &Node[T]{Value: value}
    
    if ll.head == nil {
        ll.head = newNode
    } else {
        current := ll.head
        for current.Next != nil {
            current = current.Next
        }
        current.Next = newNode
    }
    ll.size++
}

func (ll *LinkedList[T]) Remove(value T) bool {
    if ll.head == nil {
        return false
    }
    
    // Handle removal of head node
    if isEqual(ll.head.Value, value) {
        ll.head = ll.head.Next
        ll.size--
        return true
    }
    
    current := ll.head
    for current.Next != nil {
        if isEqual(current.Next.Value, value) {
            current.Next = current.Next.Next
            ll.size--
            return true
        }
        current = current.Next
    }
    return false
}

func (ll *LinkedList[T]) ToSlice() []T {
    result := make([]T, 0, ll.size)
    current := ll.head
    for current != nil {
        result = append(result, current.Value)
        current = current.Next
    }
    return result
}

func (ll *LinkedList[T]) Size() int {
    return ll.size
}

// Helper function for comparison (simplified for demo)
func isEqual[T any](a, b T) bool {
    return fmt.Sprintf("%v", a) == fmt.Sprintf("%v", b)
}

func main() {
    // Generic stack usage
    fmt.Println("=== Generic Stack ===")
    intStack := NewStack[int]()
    intStack.Push(10)
    intStack.Push(20)
    intStack.Push(30)
    
    fmt.Printf("Stack size: %d\n", intStack.Size())
    
    if value, ok := intStack.Peek(); ok {
        fmt.Printf("Top item: %d\n", value)
    }
    
    for !intStack.IsEmpty() {
        if value, ok := intStack.Pop(); ok {
            fmt.Printf("Popped: %d\n", value)
        }
    }
    
    // String stack
    stringStack := NewStack[string]()
    stringStack.Push("first")
    stringStack.Push("second")
    stringStack.Push("third")
    
    fmt.Println("\nString stack contents:")
    for !stringStack.IsEmpty() {
        if value, ok := stringStack.Pop(); ok {
            fmt.Printf("Popped: %s\n", value)
        }
    }
    
    // Generic queue usage
    fmt.Println("\n=== Generic Queue ===")
    stringQueue := NewQueue[string]()
    stringQueue.Enqueue("Alice")
    stringQueue.Enqueue("Bob")
    stringQueue.Enqueue("Charlie")
    
    fmt.Printf("Queue size: %d\n", stringQueue.Size())
    
    for stringQueue.Size() > 0 {
        if value, ok := stringQueue.Dequeue(); ok {
            fmt.Printf("Dequeued: %s\n", value)
        }
    }
    
    // Generic linked list usage
    fmt.Println("\n=== Generic Linked List ===")
    intList := NewLinkedList[int]()
    intList.Append(1)
    intList.Append(2)
    intList.Append(3)
    intList.Prepend(0)
    
    fmt.Printf("List contents: %v\n", intList.ToSlice())
    fmt.Printf("List size: %d\n", intList.Size())
    
    intList.Remove(2)
    fmt.Printf("After removing 2: %v\n", intList.ToSlice())
}
```

Generic data structures provide type-safe containers that work with any type  
while maintaining performance. The zero value handling for generic types  
ensures safe operations even when containers are empty, and type parameters  
enable compile-time verification of element types throughout the data  
structure operations.  

## Type constraints with interfaces

Interface-based type constraints enable sophisticated generic programming by  
combining type unions with method requirements, providing precise control  
over generic type capabilities.

```go
package main

import (
    "fmt"
    "strings"
)

// Basic interface constraints
type Stringer interface {
    String() string
}

type Comparable interface {
    comparable
}

// Combined constraints using type unions and methods
type NumericStringer interface {
    ~int | ~int64 | ~float32 | ~float64
    String() string
}

// Interface constraint with multiple methods
type Serializable interface {
    String() string
    Bytes() []byte
}

// Advanced constraint combining unions and methods
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

type OrderedStringer interface {
    Ordered
    String() string
}

// Generic functions with interface constraints
func FormatAll[T Stringer](items []T) []string {
    result := make([]string, len(items))
    for i, item := range items {
        result[i] = item.String()
    }
    return result
}

func FindMax[T Ordered](slice []T) T {
    if len(slice) == 0 {
        var zero T
        return zero
    }
    
    max := slice[0]
    for _, v := range slice[1:] {
        if v > max {
            max = v
        }
    }
    return max
}

func CompareItems[T Comparable](a, b T) bool {
    return a == b
}

func SerializeAll[T Serializable](items []T) ([]string, [][]byte) {
    strings := make([]string, len(items))
    bytes := make([][]byte, len(items))
    
    for i, item := range items {
        strings[i] = item.String()
        bytes[i] = item.Bytes()
    }
    
    return strings, bytes
}

// Custom types implementing various interfaces
type Temperature int

func (t Temperature) String() string {
    return fmt.Sprintf("%d°C", int(t))
}

type Distance float64

func (d Distance) String() string {
    return fmt.Sprintf("%.2f km", float64(d))
}

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s (%d)", p.Name, p.Age)
}

func (p Person) Bytes() []byte {
    return []byte(fmt.Sprintf("Person{Name:%s,Age:%d}", p.Name, p.Age))
}

type Document struct {
    Title   string
    Content string
}

func (d Document) String() string {
    return fmt.Sprintf("Document: %s", d.Title)
}

func (d Document) Bytes() []byte {
    return []byte(fmt.Sprintf("%s\n%s", d.Title, d.Content))
}

// Constraint with type parameter bounds
func ProcessOrderedStringers[T OrderedStringer](items []T) {
    // Find maximum using Ordered constraint
    if len(items) > 0 {
        max := FindMax(items)
        fmt.Printf("Maximum item: %s\n", max.String())
    }
    
    // Format all using Stringer constraint
    formatted := FormatAll(items)
    fmt.Printf("All items: %s\n", strings.Join(formatted, ", "))
}

// Generic type with interface constraint
type Container[T Stringer] struct {
    items []T
}

func NewContainer[T Stringer]() *Container[T] {
    return &Container[T]{
        items: make([]T, 0),
    }
}

func (c *Container[T]) Add(item T) {
    c.items = append(c.items, item)
}

func (c *Container[T]) String() string {
    var parts []string
    for _, item := range c.items {
        parts = append(parts, item.String())
    }
    return "[" + strings.Join(parts, ", ") + "]"
}

func main() {
    // Using basic interface constraints
    fmt.Println("=== Basic Interface Constraints ===")
    
    temps := []Temperature{20, 25, 18, 30}
    distances := []Distance{5.5, 10.2, 3.8, 15.0}
    
    fmt.Printf("Temperatures: %s\n", strings.Join(FormatAll(temps), ", "))
    fmt.Printf("Distances: %s\n", strings.Join(FormatAll(distances), ", "))
    
    // Using Ordered constraint
    fmt.Println("\n=== Ordered Constraints ===")
    numbers := []int{10, 5, 8, 3, 12}
    words := []string{"apple", "zebra", "banana", "cherry"}
    
    fmt.Printf("Max number: %d\n", FindMax(numbers))
    fmt.Printf("Max word: %s\n", FindMax(words))
    fmt.Printf("Max temperature: %s\n", FindMax(temps))
    
    // Using Comparable constraint
    fmt.Println("\n=== Comparable Constraints ===")
    fmt.Printf("Compare 10 and 10: %v\n", CompareItems(10, 10))
    fmt.Printf("Compare 'hello' and 'world': %v\n", CompareItems("hello", "world"))
    
    // Using Serializable constraint
    fmt.Println("\n=== Serializable Constraints ===")
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
    }
    
    docs := []Document{
        {"README", "This is a README file"},
        {"LICENSE", "MIT License"},
    }
    
    peopleStrings, peopleBytes := SerializeAll(people)
    fmt.Printf("People strings: %v\n", peopleStrings)
    fmt.Printf("People bytes lengths: %v\n", 
        func() []int {
            lengths := make([]int, len(peopleBytes))
            for i, b := range peopleBytes {
                lengths[i] = len(b)
            }
            return lengths
        }())
    
    docStrings, docBytes := SerializeAll(docs)
    fmt.Printf("Document strings: %v\n", docStrings)
    fmt.Printf("Document bytes lengths: %v\n",
        func() []int {
            lengths := make([]int, len(docBytes))
            for i, b := range docBytes {
                lengths[i] = len(b)
            }
            return lengths
        }())
    
    // Using generic container with constraints
    fmt.Println("\n=== Generic Container ===")
    personContainer := NewContainer[Person]()
    personContainer.Add(Person{"Charlie", 35})
    personContainer.Add(Person{"Diana", 28})
    
    fmt.Printf("Person container: %s\n", personContainer.String())
    
    tempContainer := NewContainer[Temperature]()
    tempContainer.Add(Temperature(22))
    tempContainer.Add(Temperature(28))
    
    fmt.Printf("Temperature container: %s\n", tempContainer.String())
}
```

Interface constraints enable sophisticated generic programming by combining  
type unions, method requirements, and type bounds. The `comparable` built-in  
constraint allows equality operations, while custom interfaces can specify  
complex method requirements. Type approximation with `~` allows constraints  
on underlying types, enabling more flexible generic code.  

## Higher-order generic functions

Higher-order functions combined with generics create powerful abstractions  
for functional programming patterns while maintaining type safety and  
performance in Go.

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

// Generic map function
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Generic filter function
func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Generic reduce function
func Reduce[T, U any](slice []T, initial U, fn func(U, T) U) U {
    result := initial
    for _, v := range slice {
        result = fn(result, v)
    }
    return result
}

// Generic find function
func Find[T any](slice []T, predicate func(T) bool) (T, bool) {
    for _, v := range slice {
        if predicate(v) {
            return v, true
        }
    }
    var zero T
    return zero, false
}

// Generic any/all functions
func Any[T any](slice []T, predicate func(T) bool) bool {
    for _, v := range slice {
        if predicate(v) {
            return true
        }
    }
    return false
}

func All[T any](slice []T, predicate func(T) bool) bool {
    for _, v := range slice {
        if !predicate(v) {
            return false
        }
    }
    return true
}

// Generic partition function
func Partition[T any](slice []T, predicate func(T) bool) ([]T, []T) {
    var truthy, falsy []T
    for _, v := range slice {
        if predicate(v) {
            truthy = append(truthy, v)
        } else {
            falsy = append(falsy, v)
        }
    }
    return truthy, falsy
}

// Generic chain operations
func Chain[T any](slice []T) *ChainOps[T] {
    return &ChainOps[T]{data: slice}
}

type ChainOps[T any] struct {
    data []T
}

func (c *ChainOps[T]) Map[U any](fn func(T) U) *ChainOps[U] {
    return &ChainOps[U]{data: Map(c.data, fn)}
}

func (c *ChainOps[T]) Filter(predicate func(T) bool) *ChainOps[T] {
    return &ChainOps[T]{data: Filter(c.data, predicate)}
}

func (c *ChainOps[T]) Reduce[U any](initial U, fn func(U, T) U) U {
    return Reduce(c.data, initial, fn)
}

func (c *ChainOps[T]) Value() []T {
    return c.data
}

// Generic curry functions
func Curry2[T, U, V any](fn func(T, U) V) func(T) func(U) V {
    return func(t T) func(U) V {
        return func(u U) V {
            return fn(t, u)
        }
    }
}

func Curry3[T, U, V, W any](fn func(T, U, V) W) func(T) func(U) func(V) W {
    return func(t T) func(U) func(V) W {
        return func(u U) func(V) W {
            return func(v V) W {
                return fn(t, u, v)
            }
        }
    }
}

// Generic compose functions
func Compose2[T, U, V any](f func(U) V, g func(T) U) func(T) V {
    return func(t T) V {
        return f(g(t))
    }
}

// Generic memoization
func Memoize[T comparable, U any](fn func(T) U) func(T) U {
    cache := make(map[T]U)
    return func(t T) U {
        if v, ok := cache[t]; ok {
            return v
        }
        result := fn(t)
        cache[t] = result
        return result
    }
}

// Custom types for demonstration
type User struct {
    ID   int
    Name string
    Age  int
}

func (u User) String() string {
    return fmt.Sprintf("User{ID:%d, Name:%s, Age:%d}", u.ID, u.Name, u.Age)
}

func main() {
    // Basic higher-order functions
    fmt.Println("=== Basic Higher-Order Functions ===")
    
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    
    // Map examples
    doubled := Map(numbers, func(n int) int { return n * 2 })
    fmt.Printf("Doubled: %v\n", doubled)
    
    stringified := Map(numbers, func(n int) string { return strconv.Itoa(n) })
    fmt.Printf("Stringified: %v\n", stringified)
    
    // Filter examples
    evens := Filter(numbers, func(n int) bool { return n%2 == 0 })
    fmt.Printf("Even numbers: %v\n", evens)
    
    // Reduce examples
    sum := Reduce(numbers, 0, func(acc, n int) int { return acc + n })
    fmt.Printf("Sum: %d\n", sum)
    
    product := Reduce(numbers[:5], 1, func(acc, n int) int { return acc * n })
    fmt.Printf("Product of first 5: %d\n", product)
    
    // Find examples
    if found, ok := Find(numbers, func(n int) bool { return n > 7 }); ok {
        fmt.Printf("First number > 7: %d\n", found)
    }
    
    // Any/All examples
    hasEven := Any(numbers, func(n int) bool { return n%2 == 0 })
    allPositive := All(numbers, func(n int) bool { return n > 0 })
    fmt.Printf("Has even number: %v\n", hasEven)
    fmt.Printf("All positive: %v\n", allPositive)
    
    // Partition example
    evens2, odds := Partition(numbers, func(n int) bool { return n%2 == 0 })
    fmt.Printf("Evens: %v, Odds: %v\n", evens2, odds)
    
    // Working with custom types
    fmt.Println("\n=== Working with Custom Types ===")
    
    users := []User{
        {1, "Alice", 30},
        {2, "Bob", 25},
        {3, "Charlie", 35},
        {4, "Diana", 28},
    }
    
    // Extract names
    names := Map(users, func(u User) string { return u.Name })
    fmt.Printf("Names: %v\n", names)
    
    // Filter by age
    youngUsers := Filter(users, func(u User) bool { return u.Age < 30 })
    fmt.Printf("Young users: %v\n", youngUsers)
    
    // Calculate average age
    totalAge := Reduce(users, 0, func(acc int, u User) int { return acc + u.Age })
    avgAge := float64(totalAge) / float64(len(users))
    fmt.Printf("Average age: %.1f\n", avgAge)
    
    // Method chaining
    fmt.Println("\n=== Method Chaining ===")
    
    result := Chain(numbers).
        Filter(func(n int) bool { return n%2 == 0 }).
        Map(func(n int) int { return n * n }).
        Value()
    fmt.Printf("Even numbers squared: %v\n", result)
    
    // Chain with type transformation
    stringResult := Chain(numbers).
        Filter(func(n int) bool { return n <= 5 }).
        Map(func(n int) string { return fmt.Sprintf("num_%d", n) }).
        Value()
    fmt.Printf("Filtered and stringified: %v\n", stringResult)
    
    // Currying examples
    fmt.Println("\n=== Currying ===")
    
    add := func(a, b int) int { return a + b }
    curriedAdd := Curry2(add)
    add5 := curriedAdd(5)
    
    fmt.Printf("add5(3): %d\n", add5(3))
    fmt.Printf("add5(7): %d\n", add5(7))
    
    multiply := func(a, b, c int) int { return a * b * c }
    curriedMultiply := Curry3(multiply)
    multiplyBy2Then3 := curriedMultiply(2)(3)
    
    fmt.Printf("2*3*4: %d\n", multiplyBy2Then3(4))
    fmt.Printf("2*3*5: %d\n", multiplyBy2Then3(5))
    
    // Function composition
    fmt.Println("\n=== Function Composition ===")
    
    square := func(n int) int { return n * n }
    addOne := func(n int) int { return n + 1 }
    
    squareThenAddOne := Compose2(addOne, square)
    fmt.Printf("square(3) then add 1: %d\n", squareThenAddOne(3))
    
    // String operations composition
    toUpper := strings.ToUpper
    addExclamation := func(s string) string { return s + "!" }
    
    upperAndExclaim := Compose2(addExclamation, toUpper)
    fmt.Printf("'hello' -> %s\n", upperAndExclaim("hello"))
    
    // Memoization
    fmt.Println("\n=== Memoization ===")
    
    expensiveFunc := func(n int) int {
        fmt.Printf("Computing for %d...\n", n)
        result := n * n * n // Simulate expensive computation
        return result
    }
    
    memoizedFunc := Memoize(expensiveFunc)
    
    fmt.Printf("First call memoizedFunc(5): %d\n", memoizedFunc(5))
    fmt.Printf("Second call memoizedFunc(5): %d\n", memoizedFunc(5)) // Cached
    fmt.Printf("memoizedFunc(3): %d\n", memoizedFunc(3))
}
```

Higher-order generic functions enable powerful functional programming  
patterns in Go while maintaining type safety and performance. Function  
composition, currying, and memoization become type-safe operations that  
work with any types. Method chaining provides fluent APIs for data  
transformation pipelines, making complex operations readable and efficient.  

## Generic error handling patterns

Generic error handling patterns provide type-safe alternatives to traditional  
error handling, enabling functional approaches like Result types and  
Optional types while maintaining Go's explicit error handling philosophy.

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

// Generic Result type for error handling
type Result[T any] struct {
    value T
    err   error
}

// Result constructors
func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, err: nil}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{value: zero, err: err}
}

// Result methods
func (r Result[T]) IsOk() bool {
    return r.err == nil
}

func (r Result[T]) IsErr() bool {
    return r.err != nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.value, r.err
}

func (r Result[T]) UnwrapOr(defaultValue T) T {
    if r.IsErr() {
        return defaultValue
    }
    return r.value
}

func (r Result[T]) UnwrapOrElse(fn func(error) T) T {
    if r.IsErr() {
        return fn(r.err)
    }
    return r.value
}

// Result transformations
func (r Result[T]) Map[U any](fn func(T) U) Result[U] {
    if r.IsErr() {
        return Err[U](r.err)
    }
    return Ok(fn(r.value))
}

func (r Result[T]) MapErr(fn func(error) error) Result[T] {
    if r.IsErr() {
        return Err[T](fn(r.err))
    }
    return r
}

func (r Result[T]) FlatMap[U any](fn func(T) Result[U]) Result[U] {
    if r.IsErr() {
        return Err[U](r.err)
    }
    return fn(r.value)
}

func (r Result[T]) Filter(predicate func(T) bool, err error) Result[T] {
    if r.IsErr() {
        return r
    }
    if predicate(r.value) {
        return r
    }
    return Err[T](err)
}

// Generic Optional type
type Optional[T any] struct {
    value   T
    present bool
}

// Optional constructors
func Some[T any](value T) Optional[T] {
    return Optional[T]{value: value, present: true}
}

func None[T any]() Optional[T] {
    var zero T
    return Optional[T]{value: zero, present: false}
}

// Optional methods
func (o Optional[T]) IsSome() bool {
    return o.present
}

func (o Optional[T]) IsNone() bool {
    return !o.present
}

func (o Optional[T]) Unwrap() T {
    if !o.present {
        panic("called Unwrap on None value")
    }
    return o.value
}

func (o Optional[T]) UnwrapOr(defaultValue T) T {
    if !o.present {
        return defaultValue
    }
    return o.value
}

func (o Optional[T]) Map[U any](fn func(T) U) Optional[U] {
    if !o.present {
        return None[U]()
    }
    return Some(fn(o.value))
}

func (o Optional[T]) FlatMap[U any](fn func(T) Optional[U]) Optional[U] {
    if !o.present {
        return None[U]()
    }
    return fn(o.value)
}

func (o Optional[T]) Filter(predicate func(T) bool) Optional[T] {
    if !o.present || !predicate(o.value) {
        return None[T]()
    }
    return o
}

// Generic error combinators
func TryMap[T, U any](slice []T, fn func(T) (U, error)) Result[[]U] {
    result := make([]U, 0, len(slice))
    for _, item := range slice {
        if value, err := fn(item); err != nil {
            return Err[[]U](err)
        } else {
            result = append(result, value)
        }
    }
    return Ok(result)
}

func Collect[T any](results []Result[T]) Result[[]T] {
    values := make([]T, 0, len(results))
    for _, result := range results {
        if result.IsErr() {
            return Err[[]T](result.err)
        }
        values = append(values, result.value)
    }
    return Ok(values)
}

func CollectErrors[T any](results []Result[T]) ([]T, []error) {
    var values []T
    var errors []error
    
    for _, result := range results {
        if result.IsErr() {
            errors = append(errors, result.err)
        } else {
            values = append(values, result.value)
        }
    }
    
    return values, errors
}

// Generic retry pattern
func Retry[T any](attempts int, fn func() (T, error)) Result[T] {
    var lastErr error
    
    for i := 0; i < attempts; i++ {
        if value, err := fn(); err == nil {
            return Ok(value)
        } else {
            lastErr = err
        }
    }
    
    return Err[T](fmt.Errorf("failed after %d attempts: %w", attempts, lastErr))
}

// Validation patterns
type ValidationError struct {
    Field   string
    Message string
}

func (ve ValidationError) Error() string {
    return fmt.Sprintf("validation error on field '%s': %s", ve.Field, ve.Message)
}

func ValidateString(field, value string, minLen, maxLen int) Result[string] {
    if len(value) < minLen {
        return Err[string](ValidationError{
            Field:   field,
            Message: fmt.Sprintf("must be at least %d characters", minLen),
        })
    }
    if len(value) > maxLen {
        return Err[string](ValidationError{
            Field:   field,
            Message: fmt.Sprintf("must be at most %d characters", maxLen),
        })
    }
    return Ok(value)
}

func ValidateRange[T Numeric](field string, value T, min, max T) Result[T] {
    if value < min {
        return Err[T](ValidationError{
            Field:   field,
            Message: fmt.Sprintf("must be at least %v", min),
        })
    }
    if value > max {
        return Err[T](ValidationError{
            Field:   field,
            Message: fmt.Sprintf("must be at most %v", max),
        })
    }
    return Ok(value)
}

type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

// Example application types
type User struct {
    Name string
    Age  int
    Email string
}

func NewUser(name string, age int, email string) Result[User] {
    nameResult := ValidateString("name", name, 2, 50)
    ageResult := ValidateRange("age", age, 0, 150)
    emailResult := ValidateString("email", email, 5, 100).
        Filter(func(e string) bool { return strings.Contains(e, "@") },
               ValidationError{Field: "email", Message: "must contain @"})
    
    if nameResult.IsErr() {
        return Err[User](nameResult.err)
    }
    if ageResult.IsErr() {
        return Err[User](ageResult.err)
    }
    if emailResult.IsErr() {
        return Err[User](emailResult.err)
    }
    
    return Ok(User{
        Name:  nameResult.UnwrapOr(""),
        Age:   ageResult.UnwrapOr(0),
        Email: emailResult.UnwrapOr(""),
    })
}

func main() {
    fmt.Println("=== Result Type Patterns ===")
    
    // Basic Result usage
    parseNumber := func(s string) Result[int] {
        if value, err := strconv.Atoi(s); err != nil {
            return Err[int](err)
        } else {
            return Ok(value)
        }
    }
    
    testStrings := []string{"42", "invalid", "100", "not_a_number", "7"}
    
    for _, s := range testStrings {
        result := parseNumber(s)
        if result.IsOk() {
            fmt.Printf("'%s' -> %d\n", s, result.UnwrapOr(0))
        } else {
            fmt.Printf("'%s' -> Error: %v\n", s, result.err)
        }
    }
    
    // Result transformations
    fmt.Println("\n=== Result Transformations ===")
    
    // Map and chain operations
    doubleResult := parseNumber("21").Map(func(n int) int { return n * 2 })
    fmt.Printf("Double of 21: %v\n", doubleResult.UnwrapOr(-1))
    
    // FlatMap for chaining operations that might fail
    chainedResult := parseNumber("10").FlatMap(func(n int) Result[string] {
        if n > 5 {
            return Ok(fmt.Sprintf("Large number: %d", n))
        }
        return Err[string](fmt.Errorf("number too small: %d", n))
    })
    
    fmt.Printf("Chained result: %s\n", chainedResult.UnwrapOr("default"))
    
    // Working with collections
    fmt.Println("\n=== Collection Operations ===")
    
    // TryMap - transform slice with potentially failing function
    strings := []string{"1", "2", "invalid", "4", "5"}
    numbersResult := TryMap(strings, func(s string) (int, error) {
        return strconv.Atoi(s)
    })
    
    if numbersResult.IsOk() {
        fmt.Printf("All numbers: %v\n", numbersResult.UnwrapOr(nil))
    } else {
        fmt.Printf("Failed to parse all: %v\n", numbersResult.err)
    }
    
    // Collect multiple results
    results := []Result[int]{
        Ok(1), Ok(2), Err[int](fmt.Errorf("error")), Ok(4),
    }
    
    collected := Collect(results)
    if collected.IsOk() {
        fmt.Printf("Collected: %v\n", collected.UnwrapOr(nil))
    } else {
        fmt.Printf("Collection failed: %v\n", collected.err)
    }
    
    // Separate successes and errors
    values, errors := CollectErrors(results)
    fmt.Printf("Successful values: %v\n", values)
    fmt.Printf("Errors: %v\n", errors)
    
    // Optional type usage
    fmt.Println("\n=== Optional Type Patterns ===")
    
    findFirst := func(slice []int, predicate func(int) bool) Optional[int] {
        for _, v := range slice {
            if predicate(v) {
                return Some(v)
            }
        }
        return None[int]()
    }
    
    numbers := []int{1, 3, 5, 8, 9, 12}
    
    firstEven := findFirst(numbers, func(n int) bool { return n%2 == 0 })
    if firstEven.IsSome() {
        fmt.Printf("First even number: %d\n", firstEven.Unwrap())
    } else {
        fmt.Println("No even numbers found")
    }
    
    // Optional chaining
    doubled := firstEven.Map(func(n int) int { return n * 2 })
    fmt.Printf("Doubled first even: %d\n", doubled.UnwrapOr(-1))
    
    // Retry pattern
    fmt.Println("\n=== Retry Pattern ===")
    
    unreliableOperation := func() (string, error) {
        // Simulate unreliable operation
        if fmt.Sprintf("%d", len("test")) == "4" { // Always true for demo
            return "success", nil
        }
        return "", fmt.Errorf("operation failed")
    }
    
    retryResult := Retry(3, unreliableOperation)
    fmt.Printf("Retry result: %s\n", retryResult.UnwrapOr("failed"))
    
    // Validation patterns
    fmt.Println("\n=== Validation Patterns ===")
    
    testUsers := []struct {
        name  string
        age   int
        email string
    }{
        {"Alice", 30, "alice@example.com"},
        {"B", 25, "bob@example.com"},        // Name too short
        {"Charlie", 200, "charlie@example.com"}, // Age too high
        {"Diana", 28, "invalid-email"},       // Invalid email
        {"Eve", 35, "eve@example.com"},
    }
    
    for _, userData := range testUsers {
        userResult := NewUser(userData.name, userData.age, userData.email)
        if userResult.IsOk() {
            user := userResult.UnwrapOr(User{})
            fmt.Printf("✓ Created user: %+v\n", user)
        } else {
            fmt.Printf("✗ Validation failed: %v\n", userResult.err)
        }
    }
}
```

Generic error handling patterns provide type-safe alternatives to traditional  
Go error handling while maintaining the language's explicit error philosophy.  
Result and Optional types enable functional error handling patterns, validation  
becomes composable, and error collection patterns help manage multiple  
potential failures in a type-safe manner.  

## Generic interfaces and composition

Generic interfaces enable powerful composition patterns by allowing interfaces  
to work with type parameters, creating flexible and reusable abstractions  
that maintain type safety across different implementations.

```go
package main

import (
    "fmt"
    "sort"
    "time"
)

// Basic generic interface
type Container[T any] interface {
    Add(item T)
    Remove(item T) bool
    Contains(item T) bool
    Size() int
    Clear()
}

// Generic iterator interface
type Iterator[T any] interface {
    HasNext() bool
    Next() T
}

// Generic collection interface combining multiple interfaces
type Collection[T any] interface {
    Container[T]
    Iterable[T]
}

type Iterable[T any] interface {
    Iterator() Iterator[T]
}

// Generic comparator interface
type Comparator[T any] interface {
    Compare(a, b T) int
}

// Generic transformer interface
type Transformer[T, U any] interface {
    Transform(T) U
}

// Generic predicate interface
type Predicate[T any] interface {
    Test(T) bool
}

// Implementations

// Generic slice-based container
type SliceContainer[T comparable] struct {
    items []T
}

func NewSliceContainer[T comparable]() *SliceContainer[T] {
    return &SliceContainer[T]{
        items: make([]T, 0),
    }
}

func (sc *SliceContainer[T]) Add(item T) {
    sc.items = append(sc.items, item)
}

func (sc *SliceContainer[T]) Remove(item T) bool {
    for i, v := range sc.items {
        if v == item {
            sc.items = append(sc.items[:i], sc.items[i+1:]...)
            return true
        }
    }
    return false
}

func (sc *SliceContainer[T]) Contains(item T) bool {
    for _, v := range sc.items {
        if v == item {
            return true
        }
    }
    return false
}

func (sc *SliceContainer[T]) Size() int {
    return len(sc.items)
}

func (sc *SliceContainer[T]) Clear() {
    sc.items = sc.items[:0]
}

func (sc *SliceContainer[T]) Iterator() Iterator[T] {
    return &SliceIterator[T]{
        items: sc.items,
        index: 0,
    }
}

// Generic slice iterator
type SliceIterator[T any] struct {
    items []T
    index int
}

func (si *SliceIterator[T]) HasNext() bool {
    return si.index < len(si.items)
}

func (si *SliceIterator[T]) Next() T {
    if !si.HasNext() {
        var zero T
        return zero
    }
    item := si.items[si.index]
    si.index++
    return item
}

// Generic map-based container
type MapContainer[K comparable, V any] struct {
    items map[K]V
}

func NewMapContainer[K comparable, V any]() *MapContainer[K, V] {
    return &MapContainer[K, V]{
        items: make(map[K]V),
    }
}

func (mc *MapContainer[K, V]) Set(key K, value V) {
    mc.items[key] = value
}

func (mc *MapContainer[K, V]) Get(key K) (V, bool) {
    value, exists := mc.items[key]
    return value, exists
}

func (mc *MapContainer[K, V]) Delete(key K) bool {
    if _, exists := mc.items[key]; exists {
        delete(mc.items, key)
        return true
    }
    return false
}

func (mc *MapContainer[K, V]) Keys() []K {
    keys := make([]K, 0, len(mc.items))
    for k := range mc.items {
        keys = append(keys, k)
    }
    return keys
}

func (mc *MapContainer[K, V]) Values() []V {
    values := make([]V, 0, len(mc.items))
    for _, v := range mc.items {
        values = append(values, v)
    }
    return values
}

// Comparator implementations
type IntComparator struct{}

func (ic IntComparator) Compare(a, b int) int {
    if a < b {
        return -1
    } else if a > b {
        return 1
    }
    return 0
}

type StringComparator struct{}

func (sc StringComparator) Compare(a, b string) int {
    if a < b {
        return -1
    } else if a > b {
        return 1
    }
    return 0
}

// Generic sorting with comparator
func Sort[T any](slice []T, comp Comparator[T]) {
    sort.Slice(slice, func(i, j int) bool {
        return comp.Compare(slice[i], slice[j]) < 0
    })
}

// Transformer implementations
type StringToUpperTransformer struct{}

func (sut StringToUpperTransformer) Transform(s string) string {
    return fmt.Sprintf("UPPER_%s", s)
}

type IntToStringTransformer struct{}

func (ist IntToStringTransformer) Transform(i int) string {
    return fmt.Sprintf("num_%d", i)
}

// Predicate implementations
type EvenPredicate struct{}

func (ep EvenPredicate) Test(n int) bool {
    return n%2 == 0
}

type LengthPredicate struct {
    MinLength int
}

func (lp LengthPredicate) Test(s string) bool {
    return len(s) >= lp.MinLength
}

// Generic processing functions using interfaces
func Transform[T, U any](items []T, transformer Transformer[T, U]) []U {
    result := make([]U, len(items))
    for i, item := range items {
        result[i] = transformer.Transform(item)
    }
    return result
}

func Filter[T any](items []T, predicate Predicate[T]) []T {
    var result []T
    for _, item := range items {
        if predicate.Test(item) {
            result = append(result, item)
        }
    }
    return result
}

// Generic repository pattern
type Repository[T any, K comparable] interface {
    Save(entity T) K
    FindByID(id K) (T, bool)
    FindAll() []T
    Update(id K, entity T) bool
    Delete(id K) bool
}

// In-memory repository implementation
type InMemoryRepository[T any, K comparable] struct {
    data    map[K]T
    nextID  K
    idGen   func(K) K
}

func NewInMemoryRepository[T any, K comparable](idGen func(K) K, startID K) *InMemoryRepository[T, K] {
    return &InMemoryRepository[T, K]{
        data:   make(map[K]T),
        nextID: startID,
        idGen:  idGen,
    }
}

func (repo *InMemoryRepository[T, K]) Save(entity T) K {
    id := repo.nextID
    repo.data[id] = entity
    repo.nextID = repo.idGen(repo.nextID)
    return id
}

func (repo *InMemoryRepository[T, K]) FindByID(id K) (T, bool) {
    entity, exists := repo.data[id]
    return entity, exists
}

func (repo *InMemoryRepository[T, K]) FindAll() []T {
    entities := make([]T, 0, len(repo.data))
    for _, entity := range repo.data {
        entities = append(entities, entity)
    }
    return entities
}

func (repo *InMemoryRepository[T, K]) Update(id K, entity T) bool {
    if _, exists := repo.data[id]; exists {
        repo.data[id] = entity
        return true
    }
    return false
}

func (repo *InMemoryRepository[T, K]) Delete(id K) bool {
    if _, exists := repo.data[id]; exists {
        delete(repo.data, id)
        return true
    }
    return false
}

// Example entities
type User struct {
    Name  string
    Email string
    Age   int
}

type Product struct {
    Name  string
    Price float64
    Category string
}

func main() {
    fmt.Println("=== Generic Interface Patterns ===")
    
    // Container usage
    fmt.Println("Container Operations:")
    stringContainer := NewSliceContainer[string]()
    stringContainer.Add("apple")
    stringContainer.Add("banana")
    stringContainer.Add("cherry")
    
    fmt.Printf("Container size: %d\n", stringContainer.Size())
    fmt.Printf("Contains 'banana': %v\n", stringContainer.Contains("banana"))
    
    // Iterator usage
    fmt.Println("\nIterating through container:")
    iter := stringContainer.Iterator()
    for iter.HasNext() {
        item := iter.Next()
        fmt.Printf("Item: %s\n", item)
    }
    
    stringContainer.Remove("banana")
    fmt.Printf("After removing 'banana', size: %d\n", stringContainer.Size())
    
    // Map container usage
    fmt.Println("\n=== Map Container ===")
    userMap := NewMapContainer[int, User]()
    userMap.Set(1, User{"Alice", "alice@example.com", 30})
    userMap.Set(2, User{"Bob", "bob@example.com", 25})
    
    if user, found := userMap.Get(1); found {
        fmt.Printf("User 1: %+v\n", user)
    }
    
    fmt.Printf("All user IDs: %v\n", userMap.Keys())
    
    // Comparator usage
    fmt.Println("\n=== Comparator Patterns ===")
    
    numbers := []int{5, 2, 8, 1, 9, 3}
    fmt.Printf("Original numbers: %v\n", numbers)
    
    intComp := IntComparator{}
    Sort(numbers, intComp)
    fmt.Printf("Sorted numbers: %v\n", numbers)
    
    words := []string{"zebra", "apple", "banana", "cherry"}
    fmt.Printf("Original words: %v\n", words)
    
    stringComp := StringComparator{}
    Sort(words, stringComp)
    fmt.Printf("Sorted words: %v\n", words)
    
    // Transformer usage
    fmt.Println("\n=== Transformer Patterns ===")
    
    strings := []string{"hello", "world", "go"}
    upperTransformer := StringToUpperTransformer{}
    upperStrings := Transform(strings, upperTransformer)
    fmt.Printf("Transformed strings: %v\n", upperStrings)
    
    nums := []int{1, 2, 3, 4, 5}
    intToStringTransformer := IntToStringTransformer{}
    stringNums := Transform(nums, intToStringTransformer)
    fmt.Printf("Transformed numbers: %v\n", stringNums)
    
    // Predicate usage
    fmt.Println("\n=== Predicate Patterns ===")
    
    evenPredicate := EvenPredicate{}
    evenNumbers := Filter(nums, evenPredicate)
    fmt.Printf("Even numbers: %v\n", evenNumbers)
    
    lengthPredicate := LengthPredicate{MinLength: 4}
    longWords := Filter(words, lengthPredicate)
    fmt.Printf("Words with length >= 4: %v\n", longWords)
    
    // Repository pattern usage
    fmt.Println("\n=== Repository Pattern ===")
    
    // User repository with int IDs
    userRepo := NewInMemoryRepository[User, int](
        func(id int) int { return id + 1 }, // ID generator
        1, // Starting ID
    )
    
    // Save some users
    id1 := userRepo.Save(User{"Alice", "alice@example.com", 30})
    id2 := userRepo.Save(User{"Bob", "bob@example.com", 25})
    id3 := userRepo.Save(User{"Charlie", "charlie@example.com", 35})
    
    fmt.Printf("Saved user with ID: %d\n", id1)
    fmt.Printf("Saved user with ID: %d\n", id2)
    fmt.Printf("Saved user with ID: %d\n", id3)
    
    // Find user by ID
    if user, found := userRepo.FindByID(2); found {
        fmt.Printf("Found user 2: %+v\n", user)
    }
    
    // Update user
    updated := userRepo.Update(2, User{"Bob Smith", "bob.smith@example.com", 26})
    fmt.Printf("User 2 updated: %v\n", updated)
    
    // Find all users
    allUsers := userRepo.FindAll()
    fmt.Printf("All users (%d):\n", len(allUsers))
    for i, user := range allUsers {
        fmt.Printf("  %d: %+v\n", i+1, user)
    }
    
    // Delete user
    deleted := userRepo.Delete(1)
    fmt.Printf("User 1 deleted: %v\n", deleted)
    fmt.Printf("Remaining users: %d\n", len(userRepo.FindAll()))
    
    // Product repository with string IDs
    productRepo := NewInMemoryRepository[Product, string](
        func(id string) string { return fmt.Sprintf("PROD-%d", time.Now().UnixNano()%10000) },
        "PROD-0001",
    )
    
    prodID1 := productRepo.Save(Product{"Laptop", 999.99, "Electronics"})
    prodID2 := productRepo.Save(Product{"Book", 29.99, "Education"})
    
    fmt.Printf("\nProduct IDs: %s, %s\n", prodID1, prodID2)
    
    if product, found := productRepo.FindByID(prodID1); found {
        fmt.Printf("Found product: %+v\n", product)
    }
}
```

Generic interfaces enable sophisticated composition patterns by parameterizing  
interfaces with types. This creates flexible abstractions for containers,  
iterators, transformers, and business logic patterns like repositories.  
Interface composition with generics provides type-safe building blocks for  
complex systems while maintaining Go's interface-based design philosophy.  

## Generic algorithms and utilities

Generic algorithms provide reusable implementations of common computational  
patterns, from basic sorting and searching to advanced algorithmic operations,  
all while maintaining type safety and performance.

```go
package main

import (
    "fmt"
    "math/rand"
    "sort"
    "time"
)

// Constraint interfaces for algorithms
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

// Searching algorithms
func LinearSearch[T comparable](slice []T, target T) int {
    for i, v := range slice {
        if v == target {
            return i
        }
    }
    return -1
}

func BinarySearch[T Ordered](slice []T, target T) int {
    left, right := 0, len(slice)-1
    
    for left <= right {
        mid := left + (right-left)/2
        
        if slice[mid] == target {
            return mid
        } else if slice[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}

func FindIf[T any](slice []T, predicate func(T) bool) int {
    for i, v := range slice {
        if predicate(v) {
            return i
        }
    }
    return -1
}

func FindAll[T any](slice []T, predicate func(T) bool) []int {
    var indices []int
    for i, v := range slice {
        if predicate(v) {
            indices = append(indices, i)
        }
    }
    return indices
}

// Sorting algorithms
func BubbleSort[T Ordered](slice []T) {
    n := len(slice)
    for i := 0; i < n-1; i++ {
        for j := 0; j < n-i-1; j++ {
            if slice[j] > slice[j+1] {
                slice[j], slice[j+1] = slice[j+1], slice[j]
            }
        }
    }
}

func QuickSort[T Ordered](slice []T) {
    if len(slice) <= 1 {
        return
    }
    
    quickSortHelper(slice, 0, len(slice)-1)
}

func quickSortHelper[T Ordered](slice []T, low, high int) {
    if low < high {
        pi := partition(slice, low, high)
        quickSortHelper(slice, low, pi-1)
        quickSortHelper(slice, pi+1, high)
    }
}

func partition[T Ordered](slice []T, low, high int) int {
    pivot := slice[high]
    i := low - 1
    
    for j := low; j < high; j++ {
        if slice[j] <= pivot {
            i++
            slice[i], slice[j] = slice[j], slice[i]
        }
    }
    slice[i+1], slice[high] = slice[high], slice[i+1]
    return i + 1
}

func MergeSort[T Ordered](slice []T) []T {
    if len(slice) <= 1 {
        return slice
    }
    
    mid := len(slice) / 2
    left := MergeSort(slice[:mid])
    right := MergeSort(slice[mid:])
    
    return merge(left, right)
}

func merge[T Ordered](left, right []T) []T {
    result := make([]T, 0, len(left)+len(right))
    i, j := 0, 0
    
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    
    return result
}

// Mathematical algorithms
func Min[T Ordered](slice []T) T {
    if len(slice) == 0 {
        var zero T
        return zero
    }
    
    min := slice[0]
    for _, v := range slice[1:] {
        if v < min {
            min = v
        }
    }
    return min
}

func Max[T Ordered](slice []T) T {
    if len(slice) == 0 {
        var zero T
        return zero
    }
    
    max := slice[0]
    for _, v := range slice[1:] {
        if v > max {
            max = v
        }
    }
    return max
}

func Sum[T Numeric](slice []T) T {
    var sum T
    for _, v := range slice {
        sum += v
    }
    return sum
}

func Average[T Numeric](slice []T) float64 {
    if len(slice) == 0 {
        return 0
    }
    return float64(Sum(slice)) / float64(len(slice))
}

func Product[T Numeric](slice []T) T {
    if len(slice) == 0 {
        return 0
    }
    
    var product T = 1
    for _, v := range slice {
        product *= v
    }
    return product
}

// Set operations
func Union[T comparable](set1, set2 []T) []T {
    seen := make(map[T]bool)
    var result []T
    
    for _, v := range set1 {
        if !seen[v] {
            seen[v] = true
            result = append(result, v)
        }
    }
    
    for _, v := range set2 {
        if !seen[v] {
            seen[v] = true
            result = append(result, v)
        }
    }
    
    return result
}

func Intersection[T comparable](set1, set2 []T) []T {
    seen := make(map[T]bool)
    var result []T
    
    for _, v := range set1 {
        seen[v] = true
    }
    
    visited := make(map[T]bool)
    for _, v := range set2 {
        if seen[v] && !visited[v] {
            visited[v] = true
            result = append(result, v)
        }
    }
    
    return result
}

func Difference[T comparable](set1, set2 []T) []T {
    seen := make(map[T]bool)
    var result []T
    
    for _, v := range set2 {
        seen[v] = true
    }
    
    for _, v := range set1 {
        if !seen[v] {
            result = append(result, v)
        }
    }
    
    return result
}

func Unique[T comparable](slice []T) []T {
    seen := make(map[T]bool)
    var result []T
    
    for _, v := range slice {
        if !seen[v] {
            seen[v] = true
            result = append(result, v)
        }
    }
    
    return result
}

// Utility algorithms
func Reverse[T any](slice []T) {
    for i, j := 0, len(slice)-1; i < j; i, j = i+1, j-1 {
        slice[i], slice[j] = slice[j], slice[i]
    }
}

func Rotate[T any](slice []T, k int) {
    if len(slice) == 0 {
        return
    }
    
    k = k % len(slice)
    if k < 0 {
        k += len(slice)
    }
    
    Reverse(slice)
    Reverse(slice[:k])
    Reverse(slice[k:])
}

func Shuffle[T any](slice []T) {
    rand.Seed(time.Now().UnixNano())
    for i := len(slice) - 1; i > 0; i-- {
        j := rand.Intn(i + 1)
        slice[i], slice[j] = slice[j], slice[i]
    }
}

func Chunk[T any](slice []T, size int) [][]T {
    if size <= 0 {
        return nil
    }
    
    var chunks [][]T
    for i := 0; i < len(slice); i += size {
        end := i + size
        if end > len(slice) {
            end = len(slice)
        }
        chunks = append(chunks, slice[i:end])
    }
    
    return chunks
}

func Flatten[T any](slices [][]T) []T {
    var result []T
    for _, slice := range slices {
        result = append(result, slice...)
    }
    return result
}

// Combinatorial algorithms
func Permutations[T any](slice []T) [][]T {
    if len(slice) <= 1 {
        return [][]T{slice}
    }
    
    var result [][]T
    for i, v := range slice {
        rest := make([]T, 0, len(slice)-1)
        rest = append(rest, slice[:i]...)
        rest = append(rest, slice[i+1:]...)
        
        subPerms := Permutations(rest)
        for _, perm := range subPerms {
            newPerm := append([]T{v}, perm...)
            result = append(result, newPerm)
        }
    }
    
    return result
}

func Combinations[T any](slice []T, r int) [][]T {
    if r == 0 {
        return [][]T{{}}
    }
    if len(slice) == 0 {
        return nil
    }
    
    var result [][]T
    
    // Include first element
    first := slice[0]
    rest := slice[1:]
    
    subCombos := Combinations(rest, r-1)
    for _, combo := range subCombos {
        newCombo := append([]T{first}, combo...)
        result = append(result, newCombo)
    }
    
    // Exclude first element
    result = append(result, Combinations(rest, r)...)
    
    return result
}

// Comparison function type
type CompareFunc[T any] func(T, T) int

func SortBy[T any](slice []T, compare CompareFunc[T]) {
    sort.Slice(slice, func(i, j int) bool {
        return compare(slice[i], slice[j]) < 0
    })
}

// Example custom types
type Point struct {
    X, Y int
}

func (p Point) String() string {
    return fmt.Sprintf("(%d,%d)", p.X, p.Y)
}

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s(%d)", p.Name, p.Age)
}

func main() {
    fmt.Println("=== Generic Algorithm Examples ===")
    
    // Searching algorithms
    fmt.Println("Search Algorithms:")
    numbers := []int{1, 3, 5, 7, 9, 11, 13, 15}
    target := 7
    
    linearIndex := LinearSearch(numbers, target)
    binaryIndex := BinarySearch(numbers, target)
    
    fmt.Printf("Linear search for %d: index %d\n", target, linearIndex)
    fmt.Printf("Binary search for %d: index %d\n", target, binaryIndex)
    
    evenIndex := FindIf(numbers, func(n int) bool { return n%2 == 0 })
    fmt.Printf("First even number index: %d\n", evenIndex)
    
    oddIndices := FindAll(numbers, func(n int) bool { return n%2 == 1 })
    fmt.Printf("All odd number indices: %v\n", oddIndices)
    
    // Sorting algorithms
    fmt.Println("\nSorting Algorithms:")
    
    // Test data for sorting
    testData1 := []int{64, 34, 25, 12, 22, 11, 90}
    testData2 := make([]int, len(testData1))
    testData3 := make([]int, len(testData1))
    copy(testData2, testData1)
    copy(testData3, testData1)
    
    fmt.Printf("Original: %v\n", testData1)
    
    BubbleSort(testData1)
    fmt.Printf("Bubble sort: %v\n", testData1)
    
    QuickSort(testData2)
    fmt.Printf("Quick sort: %v\n", testData2)
    
    sortedData3 := MergeSort(testData3)
    fmt.Printf("Merge sort: %v\n", sortedData3)
    
    // Mathematical operations
    fmt.Println("\nMathematical Operations:")
    
    values := []float64{3.14, 2.71, 1.41, 1.73, 2.23}
    fmt.Printf("Values: %v\n", values)
    fmt.Printf("Min: %.2f\n", Min(values))
    fmt.Printf("Max: %.2f\n", Max(values))
    fmt.Printf("Sum: %.2f\n", Sum(values))
    fmt.Printf("Average: %.2f\n", Average(values))
    fmt.Printf("Product: %.2f\n", Product(values))
    
    // Set operations
    fmt.Println("\nSet Operations:")
    
    set1 := []int{1, 2, 3, 4, 5}
    set2 := []int{4, 5, 6, 7, 8}
    
    fmt.Printf("Set 1: %v\n", set1)
    fmt.Printf("Set 2: %v\n", set2)
    fmt.Printf("Union: %v\n", Union(set1, set2))
    fmt.Printf("Intersection: %v\n", Intersection(set1, set2))
    fmt.Printf("Difference (1-2): %v\n", Difference(set1, set2))
    fmt.Printf("Difference (2-1): %v\n", Difference(set2, set1))
    
    duplicates := []int{1, 2, 2, 3, 3, 3, 4, 4, 5}
    fmt.Printf("With duplicates: %v\n", duplicates)
    fmt.Printf("Unique: %v\n", Unique(duplicates))
    
    // Utility operations
    fmt.Println("\nUtility Operations:")
    
    letters := []string{"a", "b", "c", "d", "e"}
    fmt.Printf("Original: %v\n", letters)
    
    // Make copies for different operations
    reversed := make([]string, len(letters))
    copy(reversed, letters)
    Reverse(reversed)
    fmt.Printf("Reversed: %v\n", reversed)
    
    rotated := make([]string, len(letters))
    copy(rotated, letters)
    Rotate(rotated, 2)
    fmt.Printf("Rotated by 2: %v\n", rotated)
    
    shuffled := make([]string, len(letters))
    copy(shuffled, letters)
    Shuffle(shuffled)
    fmt.Printf("Shuffled: %v\n", shuffled)
    
    // Chunking
    nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    chunks := Chunk(nums, 3)
    fmt.Printf("Chunked by 3: %v\n", chunks)
    
    flattened := Flatten(chunks)
    fmt.Printf("Flattened: %v\n", flattened)
    
    // Combinatorial operations
    fmt.Println("\nCombinatorial Operations:")
    
    smallSet := []string{"A", "B", "C"}
    fmt.Printf("Set: %v\n", smallSet)
    
    perms := Permutations(smallSet)
    fmt.Printf("Permutations (%d):\n", len(perms))
    for i, perm := range perms {
        fmt.Printf("  %d: %v\n", i+1, perm)
    }
    
    combos := Combinations(smallSet, 2)
    fmt.Printf("Combinations of 2 (%d):\n", len(combos))
    for i, combo := range combos {
        fmt.Printf("  %d: %v\n", i+1, combo)
    }
    
    // Custom sorting
    fmt.Println("\nCustom Sorting:")
    
    people := []Person{
        {"Alice", 30},
        {"Bob", 25},
        {"Charlie", 35},
        {"Diana", 28},
    }
    
    fmt.Printf("People: %v\n", people)
    
    // Sort by age
    SortBy(people, func(a, b Person) int {
        if a.Age < b.Age {
            return -1
        } else if a.Age > b.Age {
            return 1
        }
        return 0
    })
    fmt.Printf("Sorted by age: %v\n", people)
    
    // Sort by name
    SortBy(people, func(a, b Person) int {
        if a.Name < b.Name {
            return -1
        } else if a.Name > b.Name {
            return 1
        }
        return 0
    })
    fmt.Printf("Sorted by name: %v\n", people)
    
    // Points sorting by distance from origin
    points := []Point{
        {3, 4}, {1, 1}, {5, 0}, {2, 3},
    }
    
    fmt.Printf("Points: %v\n", points)
    
    SortBy(points, func(a, b Point) int {
        distA := a.X*a.X + a.Y*a.Y
        distB := b.X*b.X + b.Y*b.Y
        if distA < distB {
            return -1
        } else if distA > distB {
            return 1
        }
        return 0
    })
    fmt.Printf("Sorted by distance from origin: %v\n", points)
}
```

Generic algorithms provide type-safe implementations of fundamental  
computational patterns. From searching and sorting to mathematical operations  
and combinatorial algorithms, generics enable writing efficient, reusable  
code that works with any appropriate type. These building blocks form the  
foundation for more complex algorithmic solutions in real-world applications.  

## Concurrent generic patterns

Generic concurrency patterns combine Go's powerful concurrency primitives  
with type safety, enabling reusable concurrent data structures and  
communication patterns that work with any type.

```go
package main

import (
    "context"
    "fmt"
    "math/rand"
    "sync"
    "time"
)

// Generic channel patterns

// Fan-out: distribute work to multiple workers
func FanOut[T any](input <-chan T, workers int) []<-chan T {
    outputs := make([]<-chan T, workers)
    
    for i := 0; i < workers; i++ {
        output := make(chan T)
        outputs[i] = output
        
        go func(out chan<- T) {
            defer close(out)
            for value := range input {
                out <- value
            }
        }(output)
    }
    
    return outputs
}

// Fan-in: collect results from multiple workers
func FanIn[T any](inputs ...<-chan T) <-chan T {
    output := make(chan T)
    var wg sync.WaitGroup
    
    for _, input := range inputs {
        wg.Add(1)
        go func(in <-chan T) {
            defer wg.Done()
            for value := range in {
                output <- value
            }
        }(input)
    }
    
    go func() {
        wg.Wait()
        close(output)
    }()
    
    return output
}

// Generic pipeline stage
func Pipeline[T, U any](input <-chan T, transform func(T) U) <-chan U {
    output := make(chan U)
    
    go func() {
        defer close(output)
        for value := range input {
            output <- transform(value)
        }
    }()
    
    return output
}

// Generic worker pool
type WorkerPool[T, U any] struct {
    workers int
    process func(T) U
}

func NewWorkerPool[T, U any](workers int, process func(T) U) *WorkerPool[T, U] {
    return &WorkerPool[T, U]{
        workers: workers,
        process: process,
    }
}

func (wp *WorkerPool[T, U]) Process(input <-chan T) <-chan U {
    output := make(chan U)
    
    var wg sync.WaitGroup
    
    // Start workers
    for i := 0; i < wp.workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for task := range input {
                result := wp.process(task)
                output <- result
            }
        }()
    }
    
    // Close output when all workers are done
    go func() {
        wg.Wait()
        close(output)
    }()
    
    return output
}

// Generic concurrent data structures

// Thread-safe generic map
type SafeMap[K comparable, V any] struct {
    mu   sync.RWMutex
    data map[K]V
}

func NewSafeMap[K comparable, V any]() *SafeMap[K, V] {
    return &SafeMap[K, V]{
        data: make(map[K]V),
    }
}

func (sm *SafeMap[K, V]) Set(key K, value V) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    sm.data[key] = value
}

func (sm *SafeMap[K, V]) Get(key K) (V, bool) {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    value, exists := sm.data[key]
    return value, exists
}

func (sm *SafeMap[K, V]) Delete(key K) {
    sm.mu.Lock()
    defer sm.mu.Unlock()
    delete(sm.data, key)
}

func (sm *SafeMap[K, V]) Len() int {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    return len(sm.data)
}

func (sm *SafeMap[K, V]) Keys() []K {
    sm.mu.RLock()
    defer sm.mu.RUnlock()
    
    keys := make([]K, 0, len(sm.data))
    for k := range sm.data {
        keys = append(keys, k)
    }
    return keys
}

// Thread-safe generic queue
type SafeQueue[T any] struct {
    mu    sync.Mutex
    cond  *sync.Cond
    items []T
}

func NewSafeQueue[T any]() *SafeQueue[T] {
    sq := &SafeQueue[T]{
        items: make([]T, 0),
    }
    sq.cond = sync.NewCond(&sq.mu)
    return sq
}

func (sq *SafeQueue[T]) Enqueue(item T) {
    sq.mu.Lock()
    defer sq.mu.Unlock()
    
    sq.items = append(sq.items, item)
    sq.cond.Signal()
}

func (sq *SafeQueue[T]) Dequeue() T {
    sq.mu.Lock()
    defer sq.mu.Unlock()
    
    for len(sq.items) == 0 {
        sq.cond.Wait()
    }
    
    item := sq.items[0]
    sq.items = sq.items[1:]
    return item
}

func (sq *SafeQueue[T]) TryDequeue() (T, bool) {
    sq.mu.Lock()
    defer sq.mu.Unlock()
    
    if len(sq.items) == 0 {
        var zero T
        return zero, false
    }
    
    item := sq.items[0]
    sq.items = sq.items[1:]
    return item, true
}

func (sq *SafeQueue[T]) Size() int {
    sq.mu.Lock()
    defer sq.mu.Unlock()
    return len(sq.items)
}

// Generic channel-based cache
type Cache[K comparable, V any] struct {
    requests chan cacheRequest[K, V]
    data     map[K]cacheItem[V]
    ttl      time.Duration
}

type cacheRequest[K comparable, V any] struct {
    operation string
    key       K
    value     V
    response  chan cacheResponse[V]
}

type cacheResponse[V any] struct {
    value  V
    exists bool
}

type cacheItem[V any] struct {
    value  V
    expiry time.Time
}

func NewCache[K comparable, V any](ttl time.Duration) *Cache[K, V] {
    cache := &Cache[K, V]{
        requests: make(chan cacheRequest[K, V]),
        data:     make(map[K]cacheItem[V]),
        ttl:      ttl,
    }
    
    go cache.run()
    return cache
}

func (c *Cache[K, V]) run() {
    ticker := time.NewTicker(c.ttl)
    defer ticker.Stop()
    
    for {
        select {
        case req := <-c.requests:
            switch req.operation {
            case "get":
                item, exists := c.data[req.key]
                if exists && time.Now().Before(item.expiry) {
                    req.response <- cacheResponse[V]{value: item.value, exists: true}
                } else {
                    delete(c.data, req.key)
                    var zero V
                    req.response <- cacheResponse[V]{value: zero, exists: false}
                }
            case "set":
                c.data[req.key] = cacheItem[V]{
                    value:  req.value,
                    expiry: time.Now().Add(c.ttl),
                }
                req.response <- cacheResponse[V]{exists: true}
            case "delete":
                delete(c.data, req.key)
                req.response <- cacheResponse[V]{exists: true}
            }
        case <-ticker.C:
            // Clean up expired items
            now := time.Now()
            for key, item := range c.data {
                if now.After(item.expiry) {
                    delete(c.data, key)
                }
            }
        }
    }
}

func (c *Cache[K, V]) Get(key K) (V, bool) {
    response := make(chan cacheResponse[V])
    c.requests <- cacheRequest[K, V]{
        operation: "get",
        key:       key,
        response:  response,
    }
    
    resp := <-response
    return resp.value, resp.exists
}

func (c *Cache[K, V]) Set(key K, value V) {
    response := make(chan cacheResponse[V])
    c.requests <- cacheRequest[K, V]{
        operation: "set",
        key:       key,
        value:     value,
        response:  response,
    }
    
    <-response
}

func (c *Cache[K, V]) Delete(key K) {
    response := make(chan cacheResponse[V])
    c.requests <- cacheRequest[K, V]{
        operation: "delete",
        key:       key,
        response:  response,
    }
    
    <-response
}

// Generic future/promise pattern
type Future[T any] struct {
    result chan T
    err    chan error
    once   sync.Once
}

func NewFuture[T any]() *Future[T] {
    return &Future[T]{
        result: make(chan T, 1),
        err:    make(chan error, 1),
    }
}

func (f *Future[T]) Complete(value T) {
    f.once.Do(func() {
        f.result <- value
    })
}

func (f *Future[T]) Fail(err error) {
    f.once.Do(func() {
        f.err <- err
    })
}

func (f *Future[T]) Get() (T, error) {
    select {
    case value := <-f.result:
        return value, nil
    case err := <-f.err:
        var zero T
        return zero, err
    }
}

func (f *Future[T]) GetWithTimeout(timeout time.Duration) (T, error) {
    select {
    case value := <-f.result:
        return value, nil
    case err := <-f.err:
        var zero T
        return zero, err
    case <-time.After(timeout):
        var zero T
        return zero, fmt.Errorf("timeout after %v", timeout)
    }
}

// Async processing function
func Async[T any](fn func() (T, error)) *Future[T] {
    future := NewFuture[T]()
    
    go func() {
        if value, err := fn(); err != nil {
            future.Fail(err)
        } else {
            future.Complete(value)
        }
    }()
    
    return future
}

// Generic context-aware processing
func ProcessWithContext[T, U any](ctx context.Context, input <-chan T, process func(T) U) <-chan U {
    output := make(chan U)
    
    go func() {
        defer close(output)
        for {
            select {
            case <-ctx.Done():
                return
            case value, ok := <-input:
                if !ok {
                    return
                }
                select {
                case <-ctx.Done():
                    return
                case output <- process(value):
                }
            }
        }
    }()
    
    return output
}

// Example types for demonstration
type Task struct {
    ID   int
    Data string
}

type Result struct {
    TaskID int
    Output string
    Duration time.Duration
}

func processTask(task Task) Result {
    start := time.Now()
    
    // Simulate work
    time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
    
    return Result{
        TaskID:   task.ID,
        Output:   fmt.Sprintf("Processed: %s", task.Data),
        Duration: time.Since(start),
    }
}

func main() {
    fmt.Println("=== Generic Concurrent Patterns ===")
    
    // Fan-out and Fan-in example
    fmt.Println("Fan-out/Fan-in Pattern:")
    
    input := make(chan int, 10)
    for i := 1; i <= 10; i++ {
        input <- i
    }
    close(input)
    
    // Fan out to 3 workers
    outputs := FanOut(input, 3)
    
    // Process each stream
    processed := make([]<-chan string, len(outputs))
    for i, output := range outputs {
        processed[i] = Pipeline(output, func(n int) string {
            return fmt.Sprintf("Worker %d processed %d", i, n)
        })
    }
    
    // Fan in the results
    results := FanIn(processed...)
    
    fmt.Println("Results:")
    for result := range results {
        fmt.Printf("  %s\n", result)
    }
    
    // Worker pool example
    fmt.Println("\nWorker Pool Pattern:")
    
    tasks := make(chan Task, 5)
    for i := 1; i <= 5; i++ {
        tasks <- Task{ID: i, Data: fmt.Sprintf("Task-%d", i)}
    }
    close(tasks)
    
    pool := NewWorkerPool(3, processTask)
    taskResults := pool.Process(tasks)
    
    fmt.Println("Task Results:")
    for result := range taskResults {
        fmt.Printf("  Task %d: %s (took %v)\n", result.TaskID, result.Output, result.Duration)
    }
    
    // Safe map example
    fmt.Println("\nSafe Map Pattern:")
    
    safeMap := NewSafeMap[string, int]()
    
    var wg sync.WaitGroup
    
    // Multiple goroutines writing
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            for j := 0; j < 3; j++ {
                key := fmt.Sprintf("key-%d-%d", id, j)
                safeMap.Set(key, id*10+j)
            }
        }(i)
    }
    
    wg.Wait()
    
    fmt.Printf("Safe map size: %d\n", safeMap.Len())
    fmt.Printf("Keys: %v\n", safeMap.Keys())
    
    if value, exists := safeMap.Get("key-2-1"); exists {
        fmt.Printf("key-2-1 = %d\n", value)
    }
    
    // Safe queue example
    fmt.Println("\nSafe Queue Pattern:")
    
    queue := NewSafeQueue[string]()
    
    // Producer
    go func() {
        for i := 0; i < 5; i++ {
            message := fmt.Sprintf("Message-%d", i)
            queue.Enqueue(message)
            fmt.Printf("Enqueued: %s\n", message)
            time.Sleep(100 * time.Millisecond)
        }
    }()
    
    // Consumer
    time.Sleep(50 * time.Millisecond) // Let producer start first
    for i := 0; i < 5; i++ {
        message := queue.Dequeue()
        fmt.Printf("Dequeued: %s\n", message)
    }
    
    // Cache example
    fmt.Println("\nCache Pattern:")
    
    cache := NewCache[string, string](2 * time.Second)
    
    // Set some values
    cache.Set("user:1", "Alice")
    cache.Set("user:2", "Bob")
    
    // Get values
    if value, exists := cache.Get("user:1"); exists {
        fmt.Printf("Cache hit: user:1 = %s\n", value)
    }
    
    if value, exists := cache.Get("user:3"); !exists {
        fmt.Printf("Cache miss: user:3\n")
        _ = value // Suppress unused variable warning
    }
    
    // Wait for expiration
    fmt.Println("Waiting for cache expiration...")
    time.Sleep(2100 * time.Millisecond)
    
    if value, exists := cache.Get("user:1"); !exists {
        fmt.Printf("Cache expired: user:1\n")
        _ = value // Suppress unused variable warning
    }
    
    // Future/Promise example
    fmt.Println("\nFuture/Promise Pattern:")
    
    // Async computation
    future := Async(func() (int, error) {
        time.Sleep(200 * time.Millisecond)
        return 42, nil
    })
    
    // Do other work...
    fmt.Println("Doing other work...")
    time.Sleep(100 * time.Millisecond)
    
    // Get the result
    if result, err := future.Get(); err == nil {
        fmt.Printf("Async result: %d\n", result)
    }
    
    // Future with timeout
    slowFuture := Async(func() (string, error) {
        time.Sleep(300 * time.Millisecond)
        return "slow result", nil
    })
    
    if result, err := slowFuture.GetWithTimeout(100 * time.Millisecond); err != nil {
        fmt.Printf("Future timeout: %v\n", err)
    } else {
        fmt.Printf("Fast result: %s\n", result)
    }
    
    // Context-aware processing
    fmt.Println("\nContext-aware Processing:")
    
    ctx, cancel := context.WithTimeout(context.Background(), 150*time.Millisecond)
    defer cancel()
    
    numbers := make(chan int, 5)
    for i := 1; i <= 5; i++ {
        numbers <- i
    }
    close(numbers)
    
    processed2 := ProcessWithContext(ctx, numbers, func(n int) string {
        time.Sleep(50 * time.Millisecond) // Simulate work
        return fmt.Sprintf("Processed: %d", n)
    })
    
    for result := range processed2 {
        fmt.Printf("Context result: %s\n", result)
    }
    
    fmt.Println("Context processing completed")
}
```

Generic concurrent patterns enable type-safe concurrent programming by  
combining Go's concurrency primitives with generic type parameters. These  
patterns provide reusable building blocks for concurrent applications,  
including worker pools, thread-safe data structures, caching systems,  
and async programming models, all while maintaining type safety and  
performance characteristics.  

## Generic builders and fluent APIs

Generic builders provide type-safe construction patterns with fluent APIs,  
enabling flexible object creation while maintaining compile-time type  
checking and method chaining across different types.

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

// Generic builder interface
type Builder[T any] interface {
    Build() T
}

// Generic query builder
type QueryBuilder[T any] struct {
    table   string
    fields  []string
    joins   []string
    wheres  []string
    orders  []string
    limit   int
    offset  int
}

func NewQueryBuilder[T any]() *QueryBuilder[T] {
    return &QueryBuilder[T]{
        fields: make([]string, 0),
        joins:  make([]string, 0),
        wheres: make([]string, 0),
        orders: make([]string, 0),
    }
}

func (qb *QueryBuilder[T]) Select(fields ...string) *QueryBuilder[T] {
    qb.fields = append(qb.fields, fields...)
    return qb
}

func (qb *QueryBuilder[T]) From(table string) *QueryBuilder[T] {
    qb.table = table
    return qb
}

func (qb *QueryBuilder[T]) Join(table, condition string) *QueryBuilder[T] {
    qb.joins = append(qb.joins, fmt.Sprintf("JOIN %s ON %s", table, condition))
    return qb
}

func (qb *QueryBuilder[T]) Where(condition string) *QueryBuilder[T] {
    qb.wheres = append(qb.wheres, condition)
    return qb
}

func (qb *QueryBuilder[T]) OrderBy(field, direction string) *QueryBuilder[T] {
    qb.orders = append(qb.orders, fmt.Sprintf("%s %s", field, direction))
    return qb
}

func (qb *QueryBuilder[T]) Limit(limit int) *QueryBuilder[T] {
    qb.limit = limit
    return qb
}

func (qb *QueryBuilder[T]) Offset(offset int) *QueryBuilder[T] {
    qb.offset = offset
    return qb
}

func (qb *QueryBuilder[T]) Build() string {
    var parts []string
    
    // SELECT clause
    if len(qb.fields) > 0 {
        parts = append(parts, fmt.Sprintf("SELECT %s", strings.Join(qb.fields, ", ")))
    }
    
    // FROM clause
    if qb.table != "" {
        parts = append(parts, fmt.Sprintf("FROM %s", qb.table))
    }
    
    // JOIN clauses
    for _, join := range qb.joins {
        parts = append(parts, join)
    }
    
    // WHERE clause
    if len(qb.wheres) > 0 {
        parts = append(parts, fmt.Sprintf("WHERE %s", strings.Join(qb.wheres, " AND ")))
    }
    
    // ORDER BY clause
    if len(qb.orders) > 0 {
        parts = append(parts, fmt.Sprintf("ORDER BY %s", strings.Join(qb.orders, ", ")))
    }
    
    // LIMIT clause
    if qb.limit > 0 {
        parts = append(parts, fmt.Sprintf("LIMIT %d", qb.limit))
    }
    
    // OFFSET clause
    if qb.offset > 0 {
        parts = append(parts, fmt.Sprintf("OFFSET %d", qb.offset))
    }
    
    return strings.Join(parts, " ")
}

// Generic HTTP request builder
type HTTPRequestBuilder[T any] struct {
    method  string
    url     string
    headers map[string]string
    body    T
    timeout time.Duration
}

func NewHTTPRequestBuilder[T any]() *HTTPRequestBuilder[T] {
    return &HTTPRequestBuilder[T]{
        headers: make(map[string]string),
        timeout: 30 * time.Second,
    }
}

func (hrb *HTTPRequestBuilder[T]) Method(method string) *HTTPRequestBuilder[T] {
    hrb.method = method
    return hrb
}

func (hrb *HTTPRequestBuilder[T]) URL(url string) *HTTPRequestBuilder[T] {
    hrb.url = url
    return hrb
}

func (hrb *HTTPRequestBuilder[T]) Header(key, value string) *HTTPRequestBuilder[T] {
    hrb.headers[key] = value
    return hrb
}

func (hrb *HTTPRequestBuilder[T]) Body(body T) *HTTPRequestBuilder[T] {
    hrb.body = body
    return hrb
}

func (hrb *HTTPRequestBuilder[T]) Timeout(timeout time.Duration) *HTTPRequestBuilder[T] {
    hrb.timeout = timeout
    return hrb
}

func (hrb *HTTPRequestBuilder[T]) Build() HTTPRequest[T] {
    return HTTPRequest[T]{
        Method:  hrb.method,
        URL:     hrb.url,
        Headers: hrb.headers,
        Body:    hrb.body,
        Timeout: hrb.timeout,
    }
}

type HTTPRequest[T any] struct {
    Method  string
    URL     string
    Headers map[string]string
    Body    T
    Timeout time.Duration
}

func (hr HTTPRequest[T]) String() string {
    var parts []string
    parts = append(parts, fmt.Sprintf("Method: %s", hr.Method))
    parts = append(parts, fmt.Sprintf("URL: %s", hr.URL))
    
    if len(hr.Headers) > 0 {
        parts = append(parts, "Headers:")
        for k, v := range hr.Headers {
            parts = append(parts, fmt.Sprintf("  %s: %s", k, v))
        }
    }
    
    parts = append(parts, fmt.Sprintf("Body: %+v", hr.Body))
    parts = append(parts, fmt.Sprintf("Timeout: %v", hr.Timeout))
    
    return strings.Join(parts, "\n")
}

// Generic configuration builder
type ConfigBuilder[T any] struct {
    config T
    validators []func(T) error
}

func NewConfigBuilder[T any](initial T) *ConfigBuilder[T] {
    return &ConfigBuilder[T]{
        config: initial,
        validators: make([]func(T) error, 0),
    }
}

func (cb *ConfigBuilder[T]) Set(setter func(T) T) *ConfigBuilder[T] {
    cb.config = setter(cb.config)
    return cb
}

func (cb *ConfigBuilder[T]) Validate(validator func(T) error) *ConfigBuilder[T] {
    cb.validators = append(cb.validators, validator)
    return cb
}

func (cb *ConfigBuilder[T]) Build() (T, error) {
    for _, validator := range cb.validators {
        if err := validator(cb.config); err != nil {
            var zero T
            return zero, err
        }
    }
    return cb.config, nil
}

// Generic data structure builder
type DataStructureBuilder[T any, K comparable] struct {
    data   map[K]T
    keyGen func() K
}

func NewDataStructureBuilder[T any, K comparable](keyGen func() K) *DataStructureBuilder[T, K] {
    return &DataStructureBuilder[T, K]{
        data:   make(map[K]T),
        keyGen: keyGen,
    }
}

func (dsb *DataStructureBuilder[T, K]) Add(item T) *DataStructureBuilder[T, K] {
    key := dsb.keyGen()
    dsb.data[key] = item
    return dsb
}

func (dsb *DataStructureBuilder[T, K]) AddWithKey(key K, item T) *DataStructureBuilder[T, K] {
    dsb.data[key] = item
    return dsb
}

func (dsb *DataStructureBuilder[T, K]) Remove(key K) *DataStructureBuilder[T, K] {
    delete(dsb.data, key)
    return dsb
}

func (dsb *DataStructureBuilder[T, K]) Build() map[K]T {
    result := make(map[K]T)
    for k, v := range dsb.data {
        result[k] = v
    }
    return result
}

// Generic chain builder for transformations
type ChainBuilder[T any] struct {
    value T
}

func Chain[T any](value T) *ChainBuilder[T] {
    return &ChainBuilder[T]{value: value}
}

func (cb *ChainBuilder[T]) Transform(fn func(T) T) *ChainBuilder[T] {
    cb.value = fn(cb.value)
    return cb
}

func (cb *ChainBuilder[T]) Map[U any](fn func(T) U) *ChainBuilder[U] {
    return &ChainBuilder[U]{value: fn(cb.value)}
}

func (cb *ChainBuilder[T]) Filter(predicate func(T) bool, defaultValue T) *ChainBuilder[T] {
    if predicate(cb.value) {
        return cb
    }
    return &ChainBuilder[T]{value: defaultValue}
}

func (cb *ChainBuilder[T]) Build() T {
    return cb.value
}

// Generic accumulator builder
type AccumulatorBuilder[T, U any] struct {
    initial U
    items   []T
    reducer func(U, T) U
}

func NewAccumulatorBuilder[T, U any](initial U, reducer func(U, T) U) *AccumulatorBuilder[T, U] {
    return &AccumulatorBuilder[T, U]{
        initial: initial,
        items:   make([]T, 0),
        reducer: reducer,
    }
}

func (ab *AccumulatorBuilder[T, U]) Add(item T) *AccumulatorBuilder[T, U] {
    ab.items = append(ab.items, item)
    return ab
}

func (ab *AccumulatorBuilder[T, U]) AddAll(items ...T) *AccumulatorBuilder[T, U] {
    ab.items = append(ab.items, items...)
    return ab
}

func (ab *AccumulatorBuilder[T, U]) Build() U {
    result := ab.initial
    for _, item := range ab.items {
        result = ab.reducer(result, item)
    }
    return result
}

// Example types for demonstration
type User struct {
    ID    int
    Name  string
    Email string
    Age   int
}

type DatabaseConfig struct {
    Host     string
    Port     int
    Database string
    Username string
    Password string
    MaxConns int
}

type APIRequest struct {
    Endpoint string
    Method   string
    Headers  map[string]string
    Payload  interface{}
}

func main() {
    fmt.Println("=== Generic Builder Patterns ===")
    
    // Query builder example
    fmt.Println("Query Builder:")
    
    userQuery := NewQueryBuilder[User]().
        Select("id", "name", "email").
        From("users").
        Join("profiles", "users.id = profiles.user_id").
        Where("users.active = true").
        Where("users.age >= 18").
        OrderBy("users.name", "ASC").
        Limit(10).
        Offset(20).
        Build()
    
    fmt.Printf("Generated SQL:\n%s\n\n", userQuery)
    
    // HTTP request builder example
    fmt.Println("HTTP Request Builder:")
    
    // JSON request
    jsonRequest := NewHTTPRequestBuilder[map[string]interface{}]().
        Method("POST").
        URL("https://api.example.com/users").
        Header("Content-Type", "application/json").
        Header("Authorization", "Bearer token123").
        Body(map[string]interface{}{
            "name":  "John Doe",
            "email": "john@example.com",
            "age":   30,
        }).
        Timeout(15 * time.Second).
        Build()
    
    fmt.Printf("JSON Request:\n%s\n\n", jsonRequest)
    
    // String request
    stringRequest := NewHTTPRequestBuilder[string]().
        Method("POST").
        URL("https://api.example.com/webhook").
        Header("Content-Type", "text/plain").
        Body("Hello from webhook!").
        Timeout(5 * time.Second).
        Build()
    
    fmt.Printf("String Request:\n%s\n\n", stringRequest)
    
    // Configuration builder example
    fmt.Println("Configuration Builder:")
    
    config, err := NewConfigBuilder(DatabaseConfig{}).
        Set(func(c DatabaseConfig) DatabaseConfig {
            c.Host = "localhost"
            return c
        }).
        Set(func(c DatabaseConfig) DatabaseConfig {
            c.Port = 5432
            return c
        }).
        Set(func(c DatabaseConfig) DatabaseConfig {
            c.Database = "myapp"
            c.Username = "admin"
            c.Password = "secret"
            c.MaxConns = 10
            return c
        }).
        Validate(func(c DatabaseConfig) error {
            if c.Host == "" {
                return fmt.Errorf("host is required")
            }
            return nil
        }).
        Validate(func(c DatabaseConfig) error {
            if c.Port <= 0 {
                return fmt.Errorf("port must be positive")
            }
            return nil
        }).
        Validate(func(c DatabaseConfig) error {
            if c.MaxConns <= 0 {
                return fmt.Errorf("max connections must be positive")
            }
            return nil
        }).
        Build()
    
    if err != nil {
        fmt.Printf("Configuration error: %v\n", err)
    } else {
        fmt.Printf("Database Config: %+v\n\n", config)
    }
    
    // Data structure builder example
    fmt.Println("Data Structure Builder:")
    
    userID := 1
    userMap := NewDataStructureBuilder[User, int](func() int {
        id := userID
        userID++
        return id
    }).
        Add(User{Name: "Alice", Email: "alice@example.com", Age: 30}).
        Add(User{Name: "Bob", Email: "bob@example.com", Age: 25}).
        AddWithKey(100, User{Name: "Charlie", Email: "charlie@example.com", Age: 35}).
        Add(User{Name: "Diana", Email: "diana@example.com", Age: 28}).
        Remove(2). // Remove Bob
        Build()
    
    fmt.Println("User Map:")
    for k, v := range userMap {
        fmt.Printf("  %d: %+v\n", k, v)
    }
    fmt.Println()
    
    // Chain builder example
    fmt.Println("Chain Builder:")
    
    // Transform string
    result1 := Chain("hello world").
        Transform(strings.ToUpper).
        Transform(func(s string) string {
            return strings.ReplaceAll(s, " ", "_")
        }).
        Build()
    
    fmt.Printf("String transformation: %s\n", result1)
    
    // Transform and map to different type
    result2 := Chain("12345").
        Map(func(s string) int {
            return len(s)
        }).
        Transform(func(n int) int {
            return n * 2
        }).
        Build()
    
    fmt.Printf("String to int transformation: %d\n", result2)
    
    // Chain with filter
    result3 := Chain(42).
        Filter(func(n int) bool { return n > 50 }, 0).
        Transform(func(n int) int { return n * 2 }).
        Build()
    
    fmt.Printf("Filtered result: %d\n", result3)
    
    // Accumulator builder example
    fmt.Println("\nAccumulator Builder:")
    
    // Sum accumulator
    sum := NewAccumulatorBuilder(0, func(acc int, item int) int {
        return acc + item
    }).
        Add(1).
        Add(2).
        Add(3).
        AddAll(4, 5, 6).
        Build()
    
    fmt.Printf("Sum: %d\n", sum)
    
    // String concatenation accumulator
    concatenated := NewAccumulatorBuilder("", func(acc string, item string) string {
        if acc == "" {
            return item
        }
        return acc + ", " + item
    }).
        Add("apple").
        Add("banana").
        AddAll("cherry", "date", "elderberry").
        Build()
    
    fmt.Printf("Concatenated: %s\n", concatenated)
    
    // User accumulator (building a slice)
    users := NewAccumulatorBuilder([]User{}, func(acc []User, item User) []User {
        return append(acc, item)
    }).
        Add(User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}).
        Add(User{ID: 2, Name: "Bob", Email: "bob@example.com", Age: 25}).
        Build()
    
    fmt.Printf("Users: %+v\n", users)
    
    // Complex accumulator (statistics)
    type Stats struct {
        Count int
        Sum   int
        Min   int
        Max   int
    }
    
    stats := NewAccumulatorBuilder(Stats{Min: int(^uint(0) >> 1)}, func(acc Stats, item int) Stats {
        if acc.Count == 0 {
            return Stats{Count: 1, Sum: item, Min: item, Max: item}
        }
        
        newStats := Stats{
            Count: acc.Count + 1,
            Sum:   acc.Sum + item,
            Min:   acc.Min,
            Max:   acc.Max,
        }
        
        if item < newStats.Min {
            newStats.Min = item
        }
        if item > newStats.Max {
            newStats.Max = item
        }
        
        return newStats
    }).
        AddAll(5, 2, 8, 1, 9, 3, 7).
        Build()
    
    avg := float64(stats.Sum) / float64(stats.Count)
    fmt.Printf("Statistics: Count=%d, Sum=%d, Min=%d, Max=%d, Avg=%.2f\n", 
               stats.Count, stats.Sum, stats.Min, stats.Max, avg)
}
```

Generic builders enable type-safe construction patterns with fluent APIs  
that maintain compile-time type checking across method chains. These patterns  
provide flexible object creation, configuration building, and data  
transformation while ensuring type safety and enabling expressive,  
readable code through method chaining.  

## Advanced generic patterns

Advanced generic patterns demonstrate sophisticated uses of Go's type system,  
including type inference, constraint composition, and complex generic  
abstractions that push the boundaries of what's possible with generics.

```go
package main

import (
    "fmt"
    "reflect"
    "sort"
    "strconv"
    "strings"
)

// Multi-constraint interfaces
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

type SignedNumeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~float32 | ~float64
}

// Complex constraint composition
type Addable[T any] interface {
    Add(T) T
}

type Multipliable[T any] interface {
    Multiply(T) T
}

type Arithmetic[T any] interface {
    Addable[T]
    Multipliable[T]
}

// Generic type with multiple parameters and constraints
type Matrix[T Numeric] struct {
    rows, cols int
    data       [][]T
}

func NewMatrix[T Numeric](rows, cols int) *Matrix[T] {
    matrix := &Matrix[T]{
        rows: rows,
        cols: cols,
        data: make([][]T, rows),
    }
    
    for i := range matrix.data {
        matrix.data[i] = make([]T, cols)
    }
    
    return matrix
}

func (m *Matrix[T]) Set(row, col int, value T) {
    if row >= 0 && row < m.rows && col >= 0 && col < m.cols {
        m.data[row][col] = value
    }
}

func (m *Matrix[T]) Get(row, col int) T {
    if row >= 0 && row < m.rows && col >= 0 && col < m.cols {
        return m.data[row][col]
    }
    var zero T
    return zero
}

func (m *Matrix[T]) Add(other *Matrix[T]) *Matrix[T] {
    if m.rows != other.rows || m.cols != other.cols {
        return nil
    }
    
    result := NewMatrix[T](m.rows, m.cols)
    for i := 0; i < m.rows; i++ {
        for j := 0; j < m.cols; j++ {
            result.Set(i, j, m.Get(i, j)+other.Get(i, j))
        }
    }
    
    return result
}

func (m *Matrix[T]) Multiply(other *Matrix[T]) *Matrix[T] {
    if m.cols != other.rows {
        return nil
    }
    
    result := NewMatrix[T](m.rows, other.cols)
    for i := 0; i < m.rows; i++ {
        for j := 0; j < other.cols; j++ {
            var sum T
            for k := 0; k < m.cols; k++ {
                sum += m.Get(i, k) * other.Get(k, j)
            }
            result.Set(i, j, sum)
        }
    }
    
    return result
}

func (m *Matrix[T]) String() string {
    var sb strings.Builder
    sb.WriteString("[")
    for i := 0; i < m.rows; i++ {
        if i > 0 {
            sb.WriteString(" ")
        }
        sb.WriteString("[")
        for j := 0; j < m.cols; j++ {
            if j > 0 {
                sb.WriteString(" ")
            }
            sb.WriteString(fmt.Sprintf("%v", m.Get(i, j)))
        }
        sb.WriteString("]")
        if i < m.rows-1 {
            sb.WriteString("\n")
        }
    }
    sb.WriteString("]")
    return sb.String()
}

// Generic visitor pattern
type Visitor[T any] interface {
    Visit(T) error
}

type Visitable[T any] interface {
    Accept(Visitor[T]) error
}

// Generic tree structure with visitor
type TreeNode[T any] struct {
    Value    T
    Children []*TreeNode[T]
}

func NewTreeNode[T any](value T) *TreeNode[T] {
    return &TreeNode[T]{
        Value:    value,
        Children: make([]*TreeNode[T], 0),
    }
}

func (tn *TreeNode[T]) AddChild(child *TreeNode[T]) {
    tn.Children = append(tn.Children, child)
}

func (tn *TreeNode[T]) Accept(visitor Visitor[T]) error {
    if err := visitor.Visit(tn.Value); err != nil {
        return err
    }
    
    for _, child := range tn.Children {
        if err := child.Accept(visitor); err != nil {
            return err
        }
    }
    
    return nil
}

// Concrete visitor implementations
type PrintVisitor[T any] struct {
    depth int
}

func NewPrintVisitor[T any]() *PrintVisitor[T] {
    return &PrintVisitor[T]{depth: 0}
}

func (pv *PrintVisitor[T]) Visit(value T) error {
    indent := strings.Repeat("  ", pv.depth)
    fmt.Printf("%s%v\n", indent, value)
    pv.depth++
    return nil
}

type CollectVisitor[T any] struct {
    items []T
}

func NewCollectVisitor[T any]() *CollectVisitor[T] {
    return &CollectVisitor[T]{
        items: make([]T, 0),
    }
}

func (cv *CollectVisitor[T]) Visit(value T) error {
    cv.items = append(cv.items, value)
    return nil
}

func (cv *CollectVisitor[T]) GetItems() []T {
    return cv.items
}

// Generic strategy pattern
type Strategy[T, U any] interface {
    Execute(T) U
}

type Context[T, U any] struct {
    strategy Strategy[T, U]
}

func NewContext[T, U any](strategy Strategy[T, U]) *Context[T, U] {
    return &Context[T, U]{strategy: strategy}
}

func (c *Context[T, U]) SetStrategy(strategy Strategy[T, U]) {
    c.strategy = strategy
}

func (c *Context[T, U]) ExecuteStrategy(input T) U {
    return c.strategy.Execute(input)
}

// Concrete strategies
type UpperCaseStrategy struct{}

func (ucs UpperCaseStrategy) Execute(input string) string {
    return strings.ToUpper(input)
}

type LowerCaseStrategy struct{}

func (lcs LowerCaseStrategy) Execute(input string) string {
    return strings.ToLower(input)
}

type ReverseStrategy struct{}

func (rs ReverseStrategy) Execute(input string) string {
    runes := []rune(input)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

// Generic state machine
type State[T any] interface {
    Handle(T) State[T]
    String() string
}

type StateMachine[T any] struct {
    currentState State[T]
}

func NewStateMachine[T any](initialState State[T]) *StateMachine[T] {
    return &StateMachine[T]{currentState: initialState}
}

func (sm *StateMachine[T]) ProcessEvent(event T) {
    sm.currentState = sm.currentState.Handle(event)
}

func (sm *StateMachine[T]) GetCurrentState() State[T] {
    return sm.currentState
}

// Example states for a traffic light
type TrafficEvent string

const (
    TimerExpired TrafficEvent = "timer_expired"
    Emergency    TrafficEvent = "emergency"
    Reset        TrafficEvent = "reset"
)

type RedState struct{}
type YellowState struct{}
type GreenState struct{}

func (rs RedState) Handle(event TrafficEvent) State[TrafficEvent] {
    switch event {
    case TimerExpired:
        return GreenState{}
    case Emergency:
        return RedState{}
    default:
        return rs
    }
}

func (rs RedState) String() string {
    return "RED"
}

func (ys YellowState) Handle(event TrafficEvent) State[TrafficEvent] {
    switch event {
    case TimerExpired:
        return RedState{}
    case Emergency:
        return RedState{}
    default:
        return ys
    }
}

func (ys YellowState) String() string {
    return "YELLOW"
}

func (gs GreenState) Handle(event TrafficEvent) State[TrafficEvent] {
    switch event {
    case TimerExpired:
        return YellowState{}
    case Emergency:
        return RedState{}
    default:
        return gs
    }
}

func (gs GreenState) String() string {
    return "GREEN"
}

// Generic command pattern
type Command[T any] interface {
    Execute(T) error
    Undo(T) error
}

type CommandInvoker[T any] struct {
    history []Command[T]
    current int
}

func NewCommandInvoker[T any]() *CommandInvoker[T] {
    return &CommandInvoker[T]{
        history: make([]Command[T], 0),
        current: -1,
    }
}

func (ci *CommandInvoker[T]) Execute(command Command[T], target T) error {
    if err := command.Execute(target); err != nil {
        return err
    }
    
    // Remove any commands after current position
    ci.history = ci.history[:ci.current+1]
    
    // Add new command
    ci.history = append(ci.history, command)
    ci.current++
    
    return nil
}

func (ci *CommandInvoker[T]) Undo(target T) error {
    if ci.current < 0 {
        return fmt.Errorf("nothing to undo")
    }
    
    command := ci.history[ci.current]
    if err := command.Undo(target); err != nil {
        return err
    }
    
    ci.current--
    return nil
}

func (ci *CommandInvoker[T]) Redo(target T) error {
    if ci.current >= len(ci.history)-1 {
        return fmt.Errorf("nothing to redo")
    }
    
    ci.current++
    command := ci.history[ci.current]
    return command.Execute(target)
}

// Example command for string manipulation
type StringBuffer struct {
    content string
}

type AppendCommand struct {
    text string
}

func (ac AppendCommand) Execute(sb *StringBuffer) error {
    sb.content += ac.text
    return nil
}

func (ac AppendCommand) Undo(sb *StringBuffer) error {
    if len(sb.content) >= len(ac.text) {
        sb.content = sb.content[:len(sb.content)-len(ac.text)]
    }
    return nil
}

type DeleteCommand struct {
    count int
    deleted string
}

func NewDeleteCommand(count int) *DeleteCommand {
    return &DeleteCommand{count: count}
}

func (dc *DeleteCommand) Execute(sb *StringBuffer) error {
    if len(sb.content) >= dc.count {
        dc.deleted = sb.content[len(sb.content)-dc.count:]
        sb.content = sb.content[:len(sb.content)-dc.count]
    }
    return nil
}

func (dc *DeleteCommand) Undo(sb *StringBuffer) error {
    sb.content += dc.deleted
    return nil
}

// Generic factory pattern with type constraints
type Factory[T any] interface {
    Create() T
}

type FactoryRegistry[K comparable, T any] struct {
    factories map[K]Factory[T]
}

func NewFactoryRegistry[K comparable, T any]() *FactoryRegistry[K, T] {
    return &FactoryRegistry[K, T]{
        factories: make(map[K]Factory[T]),
    }
}

func (fr *FactoryRegistry[K, T]) Register(key K, factory Factory[T]) {
    fr.factories[key] = factory
}

func (fr *FactoryRegistry[K, T]) Create(key K) (T, error) {
    factory, exists := fr.factories[key]
    if !exists {
        var zero T
        return zero, fmt.Errorf("no factory registered for key: %v", key)
    }
    
    return factory.Create(), nil
}

func (fr *FactoryRegistry[K, T]) GetRegisteredKeys() []K {
    keys := make([]K, 0, len(fr.factories))
    for k := range fr.factories {
        keys = append(keys, k)
    }
    return keys
}

// Example factories
type Shape interface {
    Area() float64
    String() string
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) String() string {
    return fmt.Sprintf("Circle(radius=%.2f)", c.Radius)
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) String() string {
    return fmt.Sprintf("Rectangle(width=%.2f, height=%.2f)", r.Width, r.Height)
}

type CircleFactory struct{}

func (cf CircleFactory) Create() Shape {
    return Circle{Radius: 1.0}
}

type RectangleFactory struct{}

func (rf RectangleFactory) Create() Shape {
    return Rectangle{Width: 1.0, Height: 1.0}
}

func main() {
    fmt.Println("=== Advanced Generic Patterns ===")
    
    // Matrix operations
    fmt.Println("Matrix Operations:")
    
    m1 := NewMatrix[int](2, 2)
    m1.Set(0, 0, 1)
    m1.Set(0, 1, 2)
    m1.Set(1, 0, 3)
    m1.Set(1, 1, 4)
    
    m2 := NewMatrix[int](2, 2)
    m2.Set(0, 0, 5)
    m2.Set(0, 1, 6)
    m2.Set(1, 0, 7)
    m2.Set(1, 1, 8)
    
    fmt.Printf("Matrix 1:\n%s\n\n", m1)
    fmt.Printf("Matrix 2:\n%s\n\n", m2)
    
    sum := m1.Add(m2)
    fmt.Printf("Sum:\n%s\n\n", sum)
    
    product := m1.Multiply(m2)
    fmt.Printf("Product:\n%s\n\n", product)
    
    // Visitor pattern
    fmt.Println("Visitor Pattern:")
    
    root := NewTreeNode("root")
    child1 := NewTreeNode("child1")
    child2 := NewTreeNode("child2")
    grandchild1 := NewTreeNode("grandchild1")
    grandchild2 := NewTreeNode("grandchild2")
    
    root.AddChild(child1)
    root.AddChild(child2)
    child1.AddChild(grandchild1)
    child1.AddChild(grandchild2)
    
    fmt.Println("Tree structure:")
    printVisitor := NewPrintVisitor[string]()
    root.Accept(printVisitor)
    
    fmt.Println("\nCollected items:")
    collectVisitor := NewCollectVisitor[string]()
    root.Accept(collectVisitor)
    fmt.Printf("Items: %v\n\n", collectVisitor.GetItems())
    
    // Strategy pattern
    fmt.Println("Strategy Pattern:")
    
    context := NewContext[string, string](UpperCaseStrategy{})
    
    text := "Hello World"
    fmt.Printf("Original: %s\n", text)
    
    result := context.ExecuteStrategy(text)
    fmt.Printf("Upper case: %s\n", result)
    
    context.SetStrategy(LowerCaseStrategy{})
    result = context.ExecuteStrategy(text)
    fmt.Printf("Lower case: %s\n", result)
    
    context.SetStrategy(ReverseStrategy{})
    result = context.ExecuteStrategy(text)
    fmt.Printf("Reversed: %s\n\n", result)
    
    // State machine
    fmt.Println("State Machine Pattern:")
    
    trafficLight := NewStateMachine[TrafficEvent](RedState{})
    
    fmt.Printf("Initial state: %s\n", trafficLight.GetCurrentState())
    
    trafficLight.ProcessEvent(TimerExpired)
    fmt.Printf("After timer expired: %s\n", trafficLight.GetCurrentState())
    
    trafficLight.ProcessEvent(TimerExpired)
    fmt.Printf("After timer expired: %s\n", trafficLight.GetCurrentState())
    
    trafficLight.ProcessEvent(Emergency)
    fmt.Printf("After emergency: %s\n\n", trafficLight.GetCurrentState())
    
    // Command pattern
    fmt.Println("Command Pattern:")
    
    buffer := &StringBuffer{}
    invoker := NewCommandInvoker[*StringBuffer]()
    
    fmt.Printf("Initial buffer: '%s'\n", buffer.content)
    
    invoker.Execute(AppendCommand{"Hello"}, buffer)
    fmt.Printf("After append 'Hello': '%s'\n", buffer.content)
    
    invoker.Execute(AppendCommand{" World"}, buffer)
    fmt.Printf("After append ' World': '%s'\n", buffer.content)
    
    deleteCmd := NewDeleteCommand(6)
    invoker.Execute(deleteCmd, buffer)
    fmt.Printf("After delete 6 chars: '%s'\n", buffer.content)
    
    invoker.Undo(buffer)
    fmt.Printf("After undo: '%s'\n", buffer.content)
    
    invoker.Undo(buffer)
    fmt.Printf("After undo: '%s'\n", buffer.content)
    
    invoker.Redo(buffer)
    fmt.Printf("After redo: '%s'\n\n", buffer.content)
    
    // Factory pattern
    fmt.Println("Factory Pattern:")
    
    shapeRegistry := NewFactoryRegistry[string, Shape]()
    shapeRegistry.Register("circle", CircleFactory{})
    shapeRegistry.Register("rectangle", RectangleFactory{})
    
    fmt.Printf("Registered factories: %v\n", shapeRegistry.GetRegisteredKeys())
    
    if circle, err := shapeRegistry.Create("circle"); err == nil {
        fmt.Printf("Created circle: %s, Area: %.2f\n", circle, circle.Area())
    }
    
    if rectangle, err := shapeRegistry.Create("rectangle"); err == nil {
        fmt.Printf("Created rectangle: %s, Area: %.2f\n", rectangle, rectangle.Area())
    }
    
    if _, err := shapeRegistry.Create("triangle"); err != nil {
        fmt.Printf("Factory error: %v\n", err)
    }
}
```

Advanced generic patterns demonstrate sophisticated applications of Go's type  
system, enabling complex design patterns like visitor, strategy, state machine,  
command, and factory patterns to be implemented in a type-safe manner. These  
patterns leverage constraint composition, type inference, and generic  
interfaces to create flexible, reusable abstractions while maintaining  
compile-time type safety and runtime performance.  

## Generic testing and mocking

Generic testing patterns enable type-safe test utilities and mocking  
frameworks that work with any type while providing compile-time guarantees  
about test correctness and mock behavior.

```go
package main

import (
    "fmt"
    "reflect"
    "testing"
    "time"
)

// Generic test assertions
type TestAssertion[T any] struct {
    t *testing.T
    actual T
}

func Assert[T any](t *testing.T, actual T) *TestAssertion[T] {
    return &TestAssertion[T]{t: t, actual: actual}
}

func (ta *TestAssertion[T]) Equals(expected T) *TestAssertion[T] {
    if !reflect.DeepEqual(ta.actual, expected) {
        ta.t.Errorf("Expected %v, but got %v", expected, ta.actual)
    }
    return ta
}

func (ta *TestAssertion[T]) NotEquals(notExpected T) *TestAssertion[T] {
    if reflect.DeepEqual(ta.actual, notExpected) {
        ta.t.Errorf("Expected not to equal %v, but got %v", notExpected, ta.actual)
    }
    return ta
}

func (ta *TestAssertion[T]) IsNil() *TestAssertion[T] {
    if !reflect.ValueOf(ta.actual).IsNil() {
        ta.t.Errorf("Expected nil, but got %v", ta.actual)
    }
    return ta
}

func (ta *TestAssertion[T]) IsNotNil() *TestAssertion[T] {
    if reflect.ValueOf(ta.actual).IsNil() {
        ta.t.Errorf("Expected not nil, but got nil")
    }
    return ta
}

// Generic mock framework
type MockCall[T any] struct {
    Args     []interface{}
    Returns  []interface{}
    CallCount int
}

type Mock[T any] struct {
    calls map[string]*MockCall[T]
}

func NewMock[T any]() *Mock[T] {
    return &Mock[T]{
        calls: make(map[string]*MockCall[T]),
    }
}

func (m *Mock[T]) On(method string, args ...interface{}) *MockCall[T] {
    call := &MockCall[T]{
        Args:    args,
        Returns: make([]interface{}, 0),
    }
    m.calls[method] = call
    return call
}

func (call *MockCall[T]) Return(values ...interface{}) *MockCall[T] {
    call.Returns = values
    return call
}

func (m *Mock[T]) Call(method string, args ...interface{}) []interface{} {
    if call, exists := m.calls[method]; exists {
        call.CallCount++
        return call.Returns
    }
    return nil
}

func (m *Mock[T]) AssertCalled(t *testing.T, method string, times int) {
    if call, exists := m.calls[method]; exists {
        if call.CallCount != times {
            t.Errorf("Expected method %s to be called %d times, but was called %d times", 
                    method, times, call.CallCount)
        }
    } else {
        t.Errorf("Expected method %s to be called, but was never called", method)
    }
}

// Generic test data builders
type TestDataBuilder[T any] struct {
    data []T
    generators []func() T
}

func NewTestDataBuilder[T any]() *TestDataBuilder[T] {
    return &TestDataBuilder[T]{
        data: make([]T, 0),
        generators: make([]func() T, 0),
    }
}

func (tdb *TestDataBuilder[T]) Add(item T) *TestDataBuilder[T] {
    tdb.data = append(tdb.data, item)
    return tdb
}

func (tdb *TestDataBuilder[T]) AddGenerator(gen func() T) *TestDataBuilder[T] {
    tdb.generators = append(tdb.generators, gen)
    return tdb
}

func (tdb *TestDataBuilder[T]) Generate(count int) []T {
    result := make([]T, 0, len(tdb.data)+count*len(tdb.generators))
    
    // Add static data
    result = append(result, tdb.data...)
    
    // Generate dynamic data
    for i := 0; i < count; i++ {
        for _, gen := range tdb.generators {
            result = append(result, gen())
        }
    }
    
    return result
}

// Generic table-driven test framework
type TestCase[Input, Expected any] struct {
    Name     string
    Input    Input
    Expected Expected
    ShouldError bool
}

type TableTest[Input, Expected any] struct {
    cases []TestCase[Input, Expected]
}

func NewTableTest[Input, Expected any]() *TableTest[Input, Expected] {
    return &TableTest[Input, Expected]{
        cases: make([]TestCase[Input, Expected], 0),
    }
}

func (tt *TableTest[Input, Expected]) AddCase(name string, input Input, expected Expected) *TableTest[Input, Expected] {
    tt.cases = append(tt.cases, TestCase[Input, Expected]{
        Name:     name,
        Input:    input,
        Expected: expected,
    })
    return tt
}

func (tt *TableTest[Input, Expected]) AddErrorCase(name string, input Input) *TableTest[Input, Expected] {
    var zero Expected
    tt.cases = append(tt.cases, TestCase[Input, Expected]{
        Name:        name,
        Input:       input,
        Expected:    zero,
        ShouldError: true,
    })
    return tt
}

func (tt *TableTest[Input, Expected]) Run(t *testing.T, testFunc func(Input) (Expected, error)) {
    for _, tc := range tt.cases {
        t.Run(tc.Name, func(t *testing.T) {
            result, err := testFunc(tc.Input)
            
            if tc.ShouldError {
                if err == nil {
                    t.Errorf("Expected error, but got none")
                }
                return
            }
            
            if err != nil {
                t.Errorf("Unexpected error: %v", err)
                return
            }
            
            Assert(t, result).Equals(tc.Expected)
        })
    }
}

// Generic benchmark framework
type BenchmarkSuite[T any] struct {
    name string
    data []T
    setupFunc func() T
    teardownFunc func(T)
}

func NewBenchmarkSuite[T any](name string) *BenchmarkSuite[T] {
    return &BenchmarkSuite[T]{
        name: name,
        data: make([]T, 0),
    }
}

func (bs *BenchmarkSuite[T]) WithData(data ...T) *BenchmarkSuite[T] {
    bs.data = append(bs.data, data...)
    return bs
}

func (bs *BenchmarkSuite[T]) WithSetup(setupFunc func() T) *BenchmarkSuite[T] {
    bs.setupFunc = setupFunc
    return bs
}

func (bs *BenchmarkSuite[T]) WithTeardown(teardownFunc func(T)) *BenchmarkSuite[T] {
    bs.teardownFunc = teardownFunc
    return bs
}

func (bs *BenchmarkSuite[T]) Benchmark(b *testing.B, benchFunc func(T)) {
    for _, data := range bs.data {
        b.Run(fmt.Sprintf("%s-%v", bs.name, data), func(b *testing.B) {
            var testData T
            if bs.setupFunc != nil {
                testData = bs.setupFunc()
            } else {
                testData = data
            }
            
            if bs.teardownFunc != nil {
                defer bs.teardownFunc(testData)
            }
            
            b.ResetTimer()
            for i := 0; i < b.N; i++ {
                benchFunc(testData)
            }
        })
    }
}

// Generic property-based testing
type PropertyTest[T any] struct {
    generator func() T
    property  func(T) bool
    samples   int
}

func NewPropertyTest[T any](generator func() T, property func(T) bool) *PropertyTest[T] {
    return &PropertyTest[T]{
        generator: generator,
        property:  property,
        samples:   100, // Default sample size
    }
}

func (pt *PropertyTest[T]) WithSamples(samples int) *PropertyTest[T] {
    pt.samples = samples
    return pt
}

func (pt *PropertyTest[T]) Run(t *testing.T) {
    for i := 0; i < pt.samples; i++ {
        testData := pt.generator()
        if !pt.property(testData) {
            t.Errorf("Property failed for input: %v", testData)
            return
        }
    }
}

// Generic test doubles
type Spy[T any] struct {
    calls []SpyCall[T]
}

type SpyCall[T any] struct {
    Method string
    Args   []interface{}
    Result T
    Time   time.Time
}

func NewSpy[T any]() *Spy[T] {
    return &Spy[T]{
        calls: make([]SpyCall[T], 0),
    }
}

func (s *Spy[T]) Record(method string, args []interface{}, result T) {
    s.calls = append(s.calls, SpyCall[T]{
        Method: method,
        Args:   args,
        Result: result,
        Time:   time.Now(),
    })
}

func (s *Spy[T]) GetCalls() []SpyCall[T] {
    return s.calls
}

func (s *Spy[T]) GetCallsForMethod(method string) []SpyCall[T] {
    var methodCalls []SpyCall[T]
    for _, call := range s.calls {
        if call.Method == method {
            methodCalls = append(methodCalls, call)
        }
    }
    return methodCalls
}

func (s *Spy[T]) WasCalled(method string) bool {
    return len(s.GetCallsForMethod(method)) > 0
}

func (s *Spy[T]) CallCount(method string) int {
    return len(s.GetCallsForMethod(method))
}

// Generic test fixtures
type Fixture[T any] struct {
    setup    func() T
    teardown func(T)
    data     T
}

func NewFixture[T any](setup func() T, teardown func(T)) *Fixture[T] {
    return &Fixture[T]{
        setup:    setup,
        teardown: teardown,
    }
}

func (f *Fixture[T]) Setup() T {
    if f.setup != nil {
        f.data = f.setup()
    }
    return f.data
}

func (f *Fixture[T]) Teardown() {
    if f.teardown != nil {
        f.teardown(f.data)
    }
}

func (f *Fixture[T]) GetData() T {
    return f.data
}

// Example usage types
type User struct {
    ID    int
    Name  string
    Email string
    Age   int
}

type UserService struct {
    users map[int]User
    spy   *Spy[User]
}

func NewUserService() *UserService {
    return &UserService{
        users: make(map[int]User),
        spy:   NewSpy[User](),
    }
}

func (us *UserService) CreateUser(user User) (User, error) {
    if user.Name == "" {
        return User{}, fmt.Errorf("name is required")
    }
    
    user.ID = len(us.users) + 1
    us.users[user.ID] = user
    us.spy.Record("CreateUser", []interface{}{user}, user)
    
    return user, nil
}

func (us *UserService) GetUser(id int) (User, error) {
    user, exists := us.users[id]
    if !exists {
        return User{}, fmt.Errorf("user not found")
    }
    
    us.spy.Record("GetUser", []interface{}{id}, user)
    return user, nil
}

func (us *UserService) GetSpy() *Spy[User] {
    return us.spy
}

// Demonstration functions (normally these would be in *_test.go files)
func demonstrateAssertions() {
    fmt.Println("=== Generic Test Assertions ===")
    
    // This would normally use *testing.T, but for demo we'll simulate
    mockT := &testing.T{}
    
    // Basic assertions
    Assert(mockT, 42).Equals(42)
    Assert(mockT, "hello").NotEquals("world")
    
    fmt.Println("Assertions would pass in real tests")
}

func demonstrateTableTesting() {
    fmt.Println("\n=== Table-Driven Testing ===")
    
    // Function to test
    add := func(inputs [2]int) (int, error) {
        return inputs[0] + inputs[1], nil
    }
    
    // Build test table
    tableTest := NewTableTest[[2]int, int]().
        AddCase("positive numbers", [2]int{2, 3}, 5).
        AddCase("negative numbers", [2]int{-2, -3}, -5).
        AddCase("mixed numbers", [2]int{5, -3}, 2).
        AddCase("zeros", [2]int{0, 0}, 0)
    
    fmt.Printf("Table test created with %d cases\n", len(tableTest.cases))
    
    // In real tests, you would call: tableTest.Run(t, add)
    for _, tc := range tableTest.cases {
        result, err := add(tc.Input)
        if err == nil && result == tc.Expected {
            fmt.Printf("✓ %s: %v + %v = %v\n", tc.Name, tc.Input[0], tc.Input[1], result)
        }
    }
}

func demonstrateTestDataBuilder() {
    fmt.Println("\n=== Test Data Builder ===")
    
    builder := NewTestDataBuilder[User]().
        Add(User{Name: "Alice", Email: "alice@example.com", Age: 30}).
        Add(User{Name: "Bob", Email: "bob@example.com", Age: 25}).
        AddGenerator(func() User {
            return User{
                Name:  fmt.Sprintf("User%d", time.Now().UnixNano()%1000),
                Email: fmt.Sprintf("user%d@example.com", time.Now().UnixNano()%1000),
                Age:   20 + int(time.Now().UnixNano()%30),
            }
        })
    
    testData := builder.Generate(2)
    fmt.Printf("Generated %d test users:\n", len(testData))
    for i, user := range testData {
        fmt.Printf("  %d: %+v\n", i+1, user)
    }
}

func demonstratePropertyTesting() {
    fmt.Println("\n=== Property-Based Testing ===")
    
    // Property: reverse of reverse should equal original
    reverseProperty := func(s string) bool {
        reversed := reverseString(s)
        doubleReversed := reverseString(reversed)
        return s == doubleReversed
    }
    
    // Generator for random strings
    stringGenerator := func() string {
        chars := "abcdefghijklmnopqrstuvwxyz"
        length := 1 + int(time.Now().UnixNano()%10)
        result := make([]byte, length)
        for i := range result {
            result[i] = chars[time.Now().UnixNano()%int64(len(chars))]
        }
        return string(result)
    }
    
    // Test property
    propertyTest := NewPropertyTest(stringGenerator, reverseProperty).WithSamples(10)
    
    // Simulate property testing
    passed := 0
    for i := 0; i < 10; i++ {
        testStr := stringGenerator()
        if reverseProperty(testStr) {
            passed++
            fmt.Printf("✓ Property holds for: '%s'\n", testStr)
        }
    }
    fmt.Printf("Property test passed: %d/10\n", passed)
}

func demonstrateSpyTesting() {
    fmt.Println("\n=== Spy Testing ===")
    
    userService := NewUserService()
    
    // Perform some operations
    user1, _ := userService.CreateUser(User{Name: "Alice", Email: "alice@example.com", Age: 30})
    user2, _ := userService.CreateUser(User{Name: "Bob", Email: "bob@example.com", Age: 25})
    
    userService.GetUser(user1.ID)
    userService.GetUser(user2.ID)
    userService.GetUser(999) // Non-existent user
    
    // Check spy records
    spy := userService.GetSpy()
    fmt.Printf("Total calls recorded: %d\n", len(spy.GetCalls()))
    fmt.Printf("CreateUser calls: %d\n", spy.CallCount("CreateUser"))
    fmt.Printf("GetUser calls: %d\n", spy.CallCount("GetUser"))
    
    // Show call details
    createCalls := spy.GetCallsForMethod("CreateUser")
    for i, call := range createCalls {
        fmt.Printf("CreateUser call %d: %+v\n", i+1, call.Result)
    }
}

func demonstrateFixtures() {
    fmt.Println("\n=== Test Fixtures ===")
    
    // Database-like fixture
    fixture := NewFixture(
        func() map[string]User {
            fmt.Println("Setting up in-memory database")
            db := make(map[string]User)
            db["alice"] = User{ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30}
            db["bob"] = User{ID: 2, Name: "Bob", Email: "bob@example.com", Age: 25}
            return db
        },
        func(db map[string]User) {
            fmt.Println("Cleaning up in-memory database")
            for k := range db {
                delete(db, k)
            }
        },
    )
    
    // Use fixture
    db := fixture.Setup()
    fmt.Printf("Database initialized with %d users\n", len(db))
    
    // Simulate test operations
    if user, exists := db["alice"]; exists {
        fmt.Printf("Found user: %+v\n", user)
    }
    
    fixture.Teardown()
    fmt.Println("Fixture cleanup completed")
}

func reverseString(s string) string {
    runes := []rune(s)
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

func main() {
    demonstrateAssertions()
    demonstrateTableTesting()
    demonstrateTestDataBuilder()
    demonstratePropertyTesting()
    demonstrateSpyTesting()
    demonstrateFixtures()
}
```

Generic testing patterns provide type-safe testing utilities that work with  
any type while maintaining compile-time guarantees. These patterns enable  
fluent assertions, table-driven tests, property-based testing, and test  
doubles (mocks, spies, fixtures) that are both flexible and type-safe,  
making tests more reliable and maintainable.  

## Migration from interface{} to generics

Migration patterns demonstrate how to systematically convert existing  
interface{}-based code to use generics, improving type safety and  
performance while maintaining backward compatibility.

```go
package main

import (
    "fmt"
    "reflect"
    "sort"
    "strconv"
    "sync"
)

// Before: interface{} based implementations
// After: Generic implementations

// =================================
// Example 1: Collection Migration
// =================================

// BEFORE: interface{} based collection
type OldContainer struct {
    items []interface{}
    mu    sync.RWMutex
}

func NewOldContainer() *OldContainer {
    return &OldContainer{
        items: make([]interface{}, 0),
    }
}

func (oc *OldContainer) Add(item interface{}) {
    oc.mu.Lock()
    defer oc.mu.Unlock()
    oc.items = append(oc.items, item)
}

func (oc *OldContainer) Get(index int) (interface{}, bool) {
    oc.mu.RLock()
    defer oc.mu.RUnlock()
    if index >= 0 && index < len(oc.items) {
        return oc.items[index], true
    }
    return nil, false
}

func (oc *OldContainer) Size() int {
    oc.mu.RLock()
    defer oc.mu.RUnlock()
    return len(oc.items)
}

// Issues with old approach:
// 1. Type assertions needed everywhere
// 2. Runtime panics possible
// 3. No compile-time type checking
// 4. Performance overhead

func demonstrateOldContainerIssues() {
    container := NewOldContainer()
    container.Add("hello")
    container.Add(42)
    container.Add(true)
    
    // Type assertions needed - error prone
    if item, ok := container.Get(0); ok {
        if str, ok := item.(string); ok {
            fmt.Printf("String: %s\n", str)
        }
    }
    
    // Runtime panic risk
    if item, ok := container.Get(1); ok {
        // This would panic if item is not an int
        // num := item.(int) // Dangerous!
        if num, ok := item.(int); ok {
            fmt.Printf("Number: %d\n", num)
        }
    }
}

// AFTER: Generic implementation
type NewContainer[T any] struct {
    items []T
    mu    sync.RWMutex
}

func NewGenericContainer[T any]() *NewContainer[T] {
    return &NewContainer[T]{
        items: make([]T, 0),
    }
}

func (nc *NewContainer[T]) Add(item T) {
    nc.mu.Lock()
    defer nc.mu.Unlock()
    nc.items = append(nc.items, item)
}

func (nc *NewContainer[T]) Get(index int) (T, bool) {
    nc.mu.RLock()
    defer nc.mu.RUnlock()
    if index >= 0 && index < len(nc.items) {
        return nc.items[index], true
    }
    var zero T
    return zero, false
}

func (nc *NewContainer[T]) Size() int {
    nc.mu.RLock()
    defer nc.mu.RUnlock()
    return len(nc.items)
}

// Benefits of generic approach:
// 1. No type assertions needed
// 2. Compile-time type safety
// 3. Better performance
// 4. Self-documenting code

func demonstrateNewContainerBenefits() {
    stringContainer := NewGenericContainer[string]()
    stringContainer.Add("hello")
    stringContainer.Add("world")
    
    // No type assertions needed!
    if item, ok := stringContainer.Get(0); ok {
        fmt.Printf("String: %s\n", item) // item is guaranteed to be string
    }
    
    intContainer := NewGenericContainer[int]()
    intContainer.Add(42)
    intContainer.Add(24)
    
    if item, ok := intContainer.Get(0); ok {
        fmt.Printf("Number: %d\n", item) // item is guaranteed to be int
    }
}

// =================================
// Example 2: Function Migration
// =================================

// BEFORE: interface{} based utility functions
func OldMax(a, b interface{}) interface{} {
    va := reflect.ValueOf(a)
    vb := reflect.ValueOf(b)
    
    if va.Type() != vb.Type() {
        panic("types must match")
    }
    
    switch va.Kind() {
    case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
        if va.Int() > vb.Int() {
            return a
        }
        return b
    case reflect.Float32, reflect.Float64:
        if va.Float() > vb.Float() {
            return a
        }
        return b
    case reflect.String:
        if va.String() > vb.String() {
            return a
        }
        return b
    default:
        panic("unsupported type")
    }
}

func OldSort(slice interface{}) {
    v := reflect.ValueOf(slice)
    if v.Kind() != reflect.Slice {
        panic("not a slice")
    }
    
    sort.Slice(slice, func(i, j int) bool {
        vi := v.Index(i).Interface()
        vj := v.Index(j).Interface()
        
        // This is complex and error-prone
        switch vi.(type) {
        case int:
            return vi.(int) < vj.(int)
        case string:
            return vi.(string) < vj.(string)
        case float64:
            return vi.(float64) < vj.(float64)
        default:
            panic("unsupported type for sorting")
        }
    })
}

// AFTER: Generic implementations
type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

func NewMax[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func NewSort[T Ordered](slice []T) {
    sort.Slice(slice, func(i, j int) bool {
        return slice[i] < slice[j]
    })
}

// =================================
// Example 3: Map Migration
// =================================

// BEFORE: interface{} based cache
type OldCache struct {
    data map[string]interface{}
    mu   sync.RWMutex
}

func NewOldCache() *OldCache {
    return &OldCache{
        data: make(map[string]interface{}),
    }
}

func (oc *OldCache) Set(key string, value interface{}) {
    oc.mu.Lock()
    defer oc.mu.Unlock()
    oc.data[key] = value
}

func (oc *OldCache) Get(key string) (interface{}, bool) {
    oc.mu.RLock()
    defer oc.mu.RUnlock()
    value, exists := oc.data[key]
    return value, exists
}

// AFTER: Generic cache
type NewCache[T any] struct {
    data map[string]T
    mu   sync.RWMutex
}

func NewGenericCache[T any]() *NewCache[T] {
    return &NewCache[T]{
        data: make(map[string]T),
    }
}

func (nc *NewCache[T]) Set(key string, value T) {
    nc.mu.Lock()
    defer nc.mu.Unlock()
    nc.data[key] = value
}

func (nc *NewCache[T]) Get(key string) (T, bool) {
    nc.mu.RLock()
    defer nc.mu.RUnlock()
    value, exists := nc.data[key]
    return value, exists
}

// =================================
// Example 4: Migration Strategy
// =================================

// Migration wrapper for gradual transition
type MigrationWrapper[T any] struct {
    oldAPI *OldContainer
    newAPI *NewContainer[T]
    useNew bool
}

func NewMigrationWrapper[T any]() *MigrationWrapper[T] {
    return &MigrationWrapper[T]{
        oldAPI: NewOldContainer(),
        newAPI: NewGenericContainer[T](),
        useNew: false, // Start with old API
    }
}

func (mw *MigrationWrapper[T]) EnableNewAPI() {
    mw.useNew = true
}

func (mw *MigrationWrapper[T]) Add(item T) {
    if mw.useNew {
        mw.newAPI.Add(item)
    } else {
        mw.oldAPI.Add(item)
    }
}

func (mw *MigrationWrapper[T]) Get(index int) (T, bool) {
    if mw.useNew {
        return mw.newAPI.Get(index)
    } else {
        if item, ok := mw.oldAPI.Get(index); ok {
            if typedItem, ok := item.(T); ok {
                return typedItem, true
            }
        }
        var zero T
        return zero, false
    }
}

// =================================
// Example 5: Backward Compatibility
// =================================

// Adapter to maintain backward compatibility
type BackwardCompatibleContainer[T any] struct {
    generic *NewContainer[T]
}

func NewBackwardCompatibleContainer[T any]() *BackwardCompatibleContainer[T] {
    return &BackwardCompatibleContainer[T]{
        generic: NewGenericContainer[T](),
    }
}

// New generic methods
func (bc *BackwardCompatibleContainer[T]) AddTyped(item T) {
    bc.generic.Add(item)
}

func (bc *BackwardCompatibleContainer[T]) GetTyped(index int) (T, bool) {
    return bc.generic.Get(index)
}

// Old interface{} methods for compatibility
func (bc *BackwardCompatibleContainer[T]) Add(item interface{}) error {
    if typedItem, ok := item.(T); ok {
        bc.generic.Add(typedItem)
        return nil
    }
    return fmt.Errorf("item is not of expected type")
}

func (bc *BackwardCompatibleContainer[T]) Get(index int) (interface{}, bool) {
    if item, ok := bc.generic.Get(index); ok {
        return item, true
    }
    return nil, false
}

// =================================
// Example 6: Performance Comparison
// =================================

type User struct {
    ID   int
    Name string
}

func benchmarkOldVsNew() {
    fmt.Println("=== Performance Comparison ===")
    
    // Old approach with interface{}
    oldContainer := NewOldContainer()
    
    // New approach with generics
    newContainer := NewGenericContainer[User]()
    
    users := []User{
        {1, "Alice"},
        {2, "Bob"},
        {3, "Charlie"},
    }
    
    // Populate containers
    for _, user := range users {
        oldContainer.Add(user)
        newContainer.Add(user)
    }
    
    // Old approach requires type assertions
    fmt.Println("Old approach (with type assertions):")
    for i := 0; i < oldContainer.Size(); i++ {
        if item, ok := oldContainer.Get(i); ok {
            if user, ok := item.(User); ok {
                fmt.Printf("  User: %+v\n", user)
            }
        }
    }
    
    // New approach is type-safe
    fmt.Println("New approach (type-safe):")
    for i := 0; i < newContainer.Size(); i++ {
        if user, ok := newContainer.Get(i); ok {
            fmt.Printf("  User: %+v\n", user)
        }
    }
}

// =================================
// Example 7: Migration Checklist
// =================================

func migrationChecklist() {
    fmt.Println("=== Migration Checklist ===")
    
    checklist := []string{
        "✓ Identify interface{} usage patterns",
        "✓ Define appropriate type constraints",
        "✓ Create generic versions of functions/types",
        "✓ Add backward compatibility layers",
        "✓ Write migration tests",
        "✓ Update documentation",
        "✓ Plan gradual rollout strategy",
        "✓ Monitor performance improvements",
        "✓ Remove old code after migration",
    }
    
    for _, item := range checklist {
        fmt.Printf("  %s\n", item)
    }
}

// =================================
// Example 8: Common Migration Patterns
// =================================

// Pattern 1: Type switch to constraint
func oldTypeSwitch(value interface{}) string {
    switch v := value.(type) {
    case int:
        return strconv.Itoa(v)
    case float64:
        return strconv.FormatFloat(v, 'f', 2, 64)
    case string:
        return v
    default:
        return "unknown"
    }
}

// Generic version with constraint
type Stringable interface {
    ~int | ~float64 | ~string
}

func newGenericToString[T Stringable](value T) string {
    switch v := any(value).(type) {
    case int:
        return strconv.Itoa(v)
    case float64:
        return strconv.FormatFloat(v, 'f', 2, 64)
    case string:
        return v
    default:
        return fmt.Sprintf("%v", value)
    }
}

// Pattern 2: Slice operations migration
func oldFilter(slice []interface{}, predicate func(interface{}) bool) []interface{} {
    var result []interface{}
    for _, item := range slice {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

func newFilter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, item := range slice {
        if predicate(item) {
            result = append(result, item)
        }
    }
    return result
}

func main() {
    fmt.Println("=== Migration from interface{} to Generics ===")
    
    fmt.Println("1. Old Container Issues:")
    demonstrateOldContainerIssues()
    
    fmt.Println("\n2. New Container Benefits:")
    demonstrateNewContainerBenefits()
    
    fmt.Println("\n3. Function Migration:")
    // Old way
    oldResult := OldMax(10, 20)
    fmt.Printf("Old Max result: %v (type: %T)\n", oldResult, oldResult)
    
    // New way
    newResult := NewMax(10, 20)
    fmt.Printf("New Max result: %v (type: %T)\n", newResult, newResult)
    
    fmt.Println("\n4. Sorting Migration:")
    oldSlice := []interface{}{3, 1, 4, 1, 5}
    newSlice := []int{3, 1, 4, 1, 5}
    
    fmt.Printf("Before old sort: %v\n", oldSlice)
    OldSort(oldSlice)
    fmt.Printf("After old sort: %v\n", oldSlice)
    
    fmt.Printf("Before new sort: %v\n", newSlice)
    NewSort(newSlice)
    fmt.Printf("After new sort: %v\n", newSlice)
    
    fmt.Println("\n5. Cache Migration:")
    oldCache := NewOldCache()
    oldCache.Set("user1", User{1, "Alice"})
    
    newCache := NewGenericCache[User]()
    newCache.Set("user1", User{1, "Alice"})
    
    // Old way requires type assertion
    if value, ok := oldCache.Get("user1"); ok {
        if user, ok := value.(User); ok {
            fmt.Printf("Old cache user: %+v\n", user)
        }
    }
    
    // New way is type-safe
    if user, ok := newCache.Get("user1"); ok {
        fmt.Printf("New cache user: %+v\n", user)
    }
    
    fmt.Println("\n6. Migration Wrapper Demo:")
    wrapper := NewMigrationWrapper[string]()
    wrapper.Add("hello")
    wrapper.Add("world")
    
    fmt.Println("Using old API:")
    if item, ok := wrapper.Get(0); ok {
        fmt.Printf("Item: %s\n", item)
    }
    
    wrapper.EnableNewAPI()
    wrapper.Add("generics")
    
    fmt.Println("Using new API:")
    if item, ok := wrapper.Get(2); ok {
        fmt.Printf("Item: %s\n", item)
    }
    
    benchmarkOldVsNew()
    migrationChecklist()
    
    fmt.Println("\n7. Pattern Migration:")
    // Old pattern
    oldFilterResult := oldFilter(
        []interface{}{1, 2, 3, 4, 5},
        func(item interface{}) bool {
            if num, ok := item.(int); ok {
                return num%2 == 0
            }
            return false
        },
    )
    fmt.Printf("Old filter result: %v\n", oldFilterResult)
    
    // New pattern
    newFilterResult := newFilter(
        []int{1, 2, 3, 4, 5},
        func(item int) bool {
            return item%2 == 0
        },
    )
    fmt.Printf("New filter result: %v\n", newFilterResult)
    
    // Type conversion examples
    fmt.Printf("Old string conversion: %s\n", oldTypeSwitch(42))
    fmt.Printf("New string conversion: %s\n", newGenericToString(42))
}
```

Migration from interface{} to generics provides significant benefits in type  
safety, performance, and code maintainability. The key is to migrate gradually  
using compatibility layers, wrapper types, and systematic replacement of  
interface{} patterns with appropriate generic constraints. This approach  
minimizes risk while maximizing the benefits of Go's type system improvements.  

## Performance optimizations with generics

Generic performance patterns demonstrate how generics can improve runtime  
performance by eliminating type assertions, reducing allocations, and  
enabling more efficient algorithms through compile-time specialization.

```go
package main

import (
    "fmt"
    "runtime"
    "time"
    "unsafe"
)

// Benchmark framework for performance testing
type BenchmarkResult struct {
    Name        string
    Operations  int
    Duration    time.Duration
    NsPerOp     int64
    AllocBytes  uint64
    AllocCount  uint64
}

func benchmark(name string, operations int, fn func()) BenchmarkResult {
    // Force garbage collection before benchmark
    runtime.GC()
    
    var memBefore, memAfter runtime.MemStats
    runtime.ReadMemStats(&memBefore)
    
    start := time.Now()
    fn()
    duration := time.Since(start)
    
    runtime.ReadMemStats(&memAfter)
    
    return BenchmarkResult{
        Name:        name,
        Operations:  operations,
        Duration:    duration,
        NsPerOp:     duration.Nanoseconds() / int64(operations),
        AllocBytes:  memAfter.TotalAlloc - memBefore.TotalAlloc,
        AllocCount:  memAfter.Mallocs - memBefore.Mallocs,
    }
}

// =================================
// Example 1: Container Performance
// =================================

// interface{} based container (slower)
type SlowContainer struct {
    items []interface{}
}

func (sc *SlowContainer) Add(item interface{}) {
    sc.items = append(sc.items, item)
}

func (sc *SlowContainer) Get(index int) interface{} {
    return sc.items[index]
}

func (sc *SlowContainer) Process() int {
    sum := 0
    for _, item := range sc.items {
        if num, ok := item.(int); ok {
            sum += num
        }
    }
    return sum
}

// Generic container (faster)
type FastContainer[T any] struct {
    items []T
}

func (fc *FastContainer[T]) Add(item T) {
    fc.items = append(fc.items, item)
}

func (fc *FastContainer[T]) Get(index int) T {
    return fc.items[index]
}

func (fc *FastContainer[T]) Process() int {
    sum := 0
    for _, item := range fc.items {
        // Type assertion to interface{} then to int for demonstration
        if num, ok := interface{}(item).(int); ok {
            sum += num
        }
    }
    return sum
}

// Specialized int container (fastest)
func (fc *FastContainer[int]) ProcessInts() int {
    sum := 0
    for _, item := range fc.items {
        sum += item // No type assertions needed!
    }
    return sum
}

// =================================
// Example 2: Slice Operation Performance
// =================================

// Old approach with interface{} and reflection
func oldMap(slice []interface{}, fn func(interface{}) interface{}) []interface{} {
    result := make([]interface{}, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Generic approach (much faster)
func newMap[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// Specialized approach for common types
func intMap(slice []int, fn func(int) int) []int {
    result := make([]int, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// =================================
// Example 3: Zero-Allocation Patterns
// =================================

// Pool pattern with generics for zero allocation
type Pool[T any] struct {
    items chan T
    new   func() T
}

func NewPool[T any](size int, newFn func() T) *Pool[T] {
    return &Pool[T]{
        items: make(chan T, size),
        new:   newFn,
    }
}

func (p *Pool[T]) Get() T {
    select {
    case item := <-p.items:
        return item
    default:
        return p.new()
    }
}

func (p *Pool[T]) Put(item T) {
    select {
    case p.items <- item:
    default:
        // Pool is full, discard item
    }
}

// Zero-allocation buffer operations
type Buffer[T any] struct {
    data []T
    len  int
}

func NewBuffer[T any](capacity int) *Buffer[T] {
    return &Buffer[T]{
        data: make([]T, capacity),
        len:  0,
    }
}

func (b *Buffer[T]) Append(item T) bool {
    if b.len >= len(b.data) {
        return false // Buffer full
    }
    b.data[b.len] = item
    b.len++
    return true
}

func (b *Buffer[T]) Get(index int) (T, bool) {
    if index >= 0 && index < b.len {
        return b.data[index], true
    }
    var zero T
    return zero, false
}

func (b *Buffer[T]) Reset() {
    b.len = 0
    // Clear references for GC
    for i := 0; i < len(b.data); i++ {
        var zero T
        b.data[i] = zero
    }
}

func (b *Buffer[T]) Slice() []T {
    return b.data[:b.len]
}

// =================================
// Example 4: Memory Layout Optimization
// =================================

// Struct of arrays (better cache locality)
type SOA[T any] struct {
    data   []T
    active []bool
    count  int
}

func NewSOA[T any](capacity int) *SOA[T] {
    return &SOA[T]{
        data:   make([]T, capacity),
        active: make([]bool, capacity),
        count:  0,
    }
}

func (soa *SOA[T]) Add(item T) int {
    if soa.count >= len(soa.data) {
        return -1
    }
    
    index := soa.count
    soa.data[index] = item
    soa.active[index] = true
    soa.count++
    return index
}

func (soa *SOA[T]) Get(index int) (T, bool) {
    if index >= 0 && index < soa.count && soa.active[index] {
        return soa.data[index], true
    }
    var zero T
    return zero, false
}

func (soa *SOA[T]) Remove(index int) bool {
    if index >= 0 && index < soa.count && soa.active[index] {
        soa.active[index] = false
        var zero T
        soa.data[index] = zero // Clear for GC
        return true
    }
    return false
}

// Iterate only over active items (cache-friendly)
func (soa *SOA[T]) ForEach(fn func(T)) {
    for i := 0; i < soa.count; i++ {
        if soa.active[i] {
            fn(soa.data[i])
        }
    }
}

// =================================
// Example 5: Inline Optimization
// =================================

// Generic functions that can be inlined
func min[T Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

func max[T Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

// Specialized versions for hot paths
func minInt(a, b int) int {
    if a < b {
        return a
    }
    return b
}

func maxInt(a, b int) int {
    if a > b {
        return a
    }
    return b
}

// =================================
// Example 6: Avoiding Boxing/Unboxing
// =================================

// Interface{} causes boxing (slower)
func oldSum(values []interface{}) float64 {
    sum := 0.0
    for _, v := range values {
        switch num := v.(type) {
        case int:
            sum += float64(num)
        case float64:
            sum += num
        case float32:
            sum += float64(num)
        }
    }
    return sum
}

// Generics avoid boxing (faster)
func newSum[T Numeric](values []T) T {
    var sum T
    for _, v := range values {
        sum += v
    }
    return sum
}

type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

// =================================
// Example 7: Compile-time Specialization
// =================================

// Generic sorting with compile-time specialization
func genericSort[T Ordered](slice []T) {
    // Compiler can specialize this for each type
    for i := 0; i < len(slice)-1; i++ {
        for j := 0; j < len(slice)-i-1; j++ {
            if slice[j] > slice[j+1] {
                slice[j], slice[j+1] = slice[j+1], slice[j]
            }
        }
    }
}

// Compare with interface{} version that uses reflection
func oldSort(slice []interface{}) {
    for i := 0; i < len(slice)-1; i++ {
        for j := 0; j < len(slice)-i-1; j++ {
            // This requires runtime type checking
            if compareInterface(slice[j], slice[j+1]) > 0 {
                slice[j], slice[j+1] = slice[j+1], slice[j]
            }
        }
    }
}

func compareInterface(a, b interface{}) int {
    switch va := a.(type) {
    case int:
        if vb, ok := b.(int); ok {
            if va > vb {
                return 1
            } else if va < vb {
                return -1
            }
            return 0
        }
    case string:
        if vb, ok := b.(string); ok {
            if va > vb {
                return 1
            } else if va < vb {
                return -1
            }
            return 0
        }
    }
    return 0
}

// =================================
// Performance Test Functions
// =================================

func testContainerPerformance() {
    const operations = 100000
    
    // Test slow container
    slowResult := benchmark("Slow Container", operations, func() {
        container := &SlowContainer{}
        for i := 0; i < operations; i++ {
            container.Add(i)
        }
        container.Process()
    })
    
    // Test fast container
    fastResult := benchmark("Fast Container", operations, func() {
        container := &FastContainer[int]{}
        for i := 0; i < operations; i++ {
            container.Add(i)
        }
        container.ProcessInts()
    })
    
    fmt.Printf("%-20s | %10s | %10s | %10s\n", "Test", "Ns/Op", "Allocs", "Bytes")
    fmt.Printf("%-20s | %10d | %10d | %10d\n", slowResult.Name, slowResult.NsPerOp, slowResult.AllocCount, slowResult.AllocBytes)
    fmt.Printf("%-20s | %10d | %10d | %10d\n", fastResult.Name, fastResult.NsPerOp, fastResult.AllocCount, fastResult.AllocBytes)
    
    if slowResult.NsPerOp > 0 {
        speedup := float64(slowResult.NsPerOp) / float64(fastResult.NsPerOp)
        fmt.Printf("Speedup: %.2fx\n", speedup)
    }
}

func testMapPerformance() {
    const size = 10000
    data := make([]interface{}, size)
    intData := make([]int, size)
    
    for i := 0; i < size; i++ {
        data[i] = i
        intData[i] = i
    }
    
    // Old map with interface{}
    oldResult := benchmark("Old Map", 1, func() {
        oldMap(data, func(v interface{}) interface{} {
            if num, ok := v.(int); ok {
                return num * 2
            }
            return v
        })
    })
    
    // Generic map
    newResult := benchmark("Generic Map", 1, func() {
        newMap(intData, func(v int) int {
            return v * 2
        })
    })
    
    // Specialized map
    specializedResult := benchmark("Specialized Map", 1, func() {
        intMap(intData, func(v int) int {
            return v * 2
        })
    })
    
    fmt.Printf("\nMap Performance:\n")
    fmt.Printf("%-20s | %10s | %10s | %10s\n", "Test", "Ns/Op", "Allocs", "Bytes")
    fmt.Printf("%-20s | %10d | %10d | %10d\n", oldResult.Name, oldResult.NsPerOp, oldResult.AllocCount, oldResult.AllocBytes)
    fmt.Printf("%-20s | %10d | %10d | %10d\n", newResult.Name, newResult.NsPerOp, newResult.AllocCount, newResult.AllocBytes)
    fmt.Printf("%-20s | %10d | %10d | %10d\n", specializedResult.Name, specializedResult.NsPerOp, specializedResult.AllocCount, specializedResult.AllocBytes)
}

func testZeroAllocationPattern() {
    fmt.Printf("\nZero-Allocation Pattern:\n")
    
    pool := NewPool(10, func() *Buffer[int] {
        return NewBuffer[int](1000)
    })
    
    // Use buffer from pool
    buffer := pool.Get()
    
    // Fill buffer
    for i := 0; i < 100; i++ {
        buffer.Append(i)
    }
    
    // Process buffer
    sum := 0
    buffer.ForEach(func(item int) {
        sum += item
    })
    
    fmt.Printf("Buffer sum: %d\n", sum)
    fmt.Printf("Buffer size: %d bytes\n", unsafe.Sizeof(*buffer))
    
    // Reset and return to pool
    buffer.Reset()
    pool.Put(buffer)
}

func testMemoryLayoutOptimization() {
    fmt.Printf("\nMemory Layout Optimization:\n")
    
    const size = 10000
    
    // Test SOA structure
    soa := NewSOA[int](size)
    
    // Add items
    for i := 0; i < size; i++ {
        soa.Add(i)
    }
    
    // Benchmark iteration
    result := benchmark("SOA Iteration", 1000, func() {
        sum := 0
        soa.ForEach(func(item int) {
            sum += item
        })
    })
    
    fmt.Printf("SOA iteration: %d ns/op\n", result.NsPerOp)
    fmt.Printf("SOA memory usage: %d bytes\n", 
              unsafe.Sizeof(*soa)+uintptr(len(soa.data))*unsafe.Sizeof(soa.data[0])+
              uintptr(len(soa.active))*unsafe.Sizeof(soa.active[0]))
}

func (b *Buffer[T]) ForEach(fn func(T)) {
    for i := 0; i < b.len; i++ {
        fn(b.data[i])
    }
}

func main() {
    fmt.Println("=== Generic Performance Optimizations ===")
    
    fmt.Println("1. Container Performance Comparison:")
    testContainerPerformance()
    
    fmt.Println("\n2. Map Operation Performance:")
    testMapPerformance()
    
    fmt.Println("\n3. Zero-Allocation Patterns:")
    testZeroAllocationPattern()
    
    fmt.Println("\n4. Memory Layout Optimization:")
    testMemoryLayoutOptimization()
    
    fmt.Println("\n5. Inline Optimization Examples:")
    a, b := 10, 20
    fmt.Printf("Generic min(%d, %d): %d\n", a, b, min(a, b))
    fmt.Printf("Specialized minInt(%d, %d): %d\n", a, b, minInt(a, b))
    
    fmt.Println("\n6. Boxing/Unboxing Avoidance:")
    
    // Old approach with boxing
    interfaceSlice := []interface{}{1, 2.5, 3, 4.7, 5}
    oldSumResult := oldSum(interfaceSlice)
    fmt.Printf("Old sum (with boxing): %.2f\n", oldSumResult)
    
    // New approach without boxing
    intSlice := []int{1, 2, 3, 4, 5}
    floatSlice := []float64{1.0, 2.5, 3.0, 4.7, 5.0}
    
    intSumResult := newSum(intSlice)
    floatSumResult := newSum(floatSlice)
    
    fmt.Printf("New sum (int, no boxing): %d\n", intSumResult)
    fmt.Printf("New sum (float64, no boxing): %.2f\n", floatSumResult)
    
    fmt.Println("\n7. Compile-time Specialization:")
    
    // Generic sort (compiler specializes for each type)
    intTestSlice := []int{5, 2, 8, 1, 9}
    stringTestSlice := []string{"banana", "apple", "cherry"}
    
    fmt.Printf("Before sort: %v\n", intTestSlice)
    genericSort(intTestSlice)
    fmt.Printf("After sort: %v\n", intTestSlice)
    
    fmt.Printf("Before sort: %v\n", stringTestSlice)
    genericSort(stringTestSlice)
    fmt.Printf("After sort: %v\n", stringTestSlice)
    
    fmt.Println("\n=== Performance Summary ===")
    fmt.Println("✓ Generics eliminate type assertions and boxing overhead")
    fmt.Println("✓ Compile-time specialization enables better optimization")
    fmt.Println("✓ Zero-allocation patterns reduce GC pressure")
    fmt.Println("✓ Better cache locality with optimized memory layouts")
    fmt.Println("✓ Inlining opportunities for small generic functions")
}
```

Generic performance optimizations demonstrate how generics can significantly  
improve runtime performance by eliminating type assertions, reducing memory  
allocations, enabling compile-time specialization, and providing better  
cache locality. These patterns are essential for high-performance applications  
where every nanosecond matters while maintaining type safety and code clarity.  

## Best practices and guidelines

Best practices for generic programming in Go ensure code remains readable,  
maintainable, and performant while leveraging the full power of the type  
system. These guidelines help teams adopt generics effectively.

```go
package main

import (
    "fmt"
    "strings"
)

// =================================
// Guideline 1: Use meaningful constraint names
// =================================

// ❌ Bad: Generic, unclear constraints
type Comparable interface {
    comparable
}

type Any interface {
    any
}

// ✅ Good: Descriptive, purpose-driven constraints
type Numeric interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64
}

type Ordered interface {
    ~int | ~int8 | ~int16 | ~int32 | ~int64 |
    ~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 |
    ~float32 | ~float64 | ~string
}

type Stringer interface {
    String() string
}

// =================================
// Guideline 2: Start simple, add constraints as needed
// =================================

// ❌ Bad: Over-constrained from the start
type OverConstrainedContainer[T Numeric & Stringer & comparable] struct {
    items []T
}

// ✅ Good: Start with minimal constraints
type SimpleContainer[T any] struct {
    items []T
}

func (sc *SimpleContainer[T]) Add(item T) {
    sc.items = append(sc.items, item)
}

// Add constraints only when needed
func (sc *SimpleContainer[T]) Sum() T where T Numeric {
    var sum T
    for _, item := range sc.items {
        sum += item
    }
    return sum
}

// Go doesn't have "where" clauses, so use separate functions
func SumContainer[T Numeric](sc *SimpleContainer[T]) T {
    var sum T
    for _, item := range sc.items {
        sum += item
    }
    return sum
}

// =================================
// Guideline 3: Prefer type parameters over interface{}
// =================================

// ❌ Bad: Using interface{} when generics would be better
type BadProcessor struct{}

func (bp *BadProcessor) Process(items []interface{}) []interface{} {
    result := make([]interface{}, len(items))
    for i, item := range items {
        // Type assertions required
        if str, ok := item.(string); ok {
            result[i] = strings.ToUpper(str)
        } else {
            result[i] = item
        }
    }
    return result
}

// ✅ Good: Using generics for type safety
type GoodProcessor[T any] struct{}

func (gp *GoodProcessor[T]) Process(items []T, transform func(T) T) []T {
    result := make([]T, len(items))
    for i, item := range items {
        result[i] = transform(item)
    }
    return result
}

// =================================
// Guideline 4: Don't over-generalize
// =================================

// ❌ Bad: Unnecessary generics for simple cases
type BadAdder[T Numeric] struct{}

func (ba *BadAdder[T]) Add(a, b T) T {
    return a + b
}

// ✅ Good: Simple function when generics add no value
func Add[T Numeric](a, b T) T {
    return a + b
}

// ✅ Better: Only use generics when you need multiple types
func ProcessNumbers[T Numeric](numbers []T, operation func(T, T) T) T {
    if len(numbers) == 0 {
        var zero T
        return zero
    }
    
    result := numbers[0]
    for _, num := range numbers[1:] {
        result = operation(result, num)
    }
    return result
}

// =================================
// Guideline 5: Use type inference when possible
// =================================

// ✅ Good: Functions that support type inference
func Map[T, U any](slice []T, fn func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Usage relies on type inference
func demonstrateTypeInference() {
    numbers := []int{1, 2, 3, 4, 5}
    
    // Type inference works
    doubled := Map(numbers, func(n int) int { return n * 2 })
    evens := Filter(numbers, func(n int) bool { return n%2 == 0 })
    
    fmt.Printf("Doubled: %v\n", doubled)
    fmt.Printf("Evens: %v\n", evens)
}

// =================================
// Guideline 6: Composition over inheritance
// =================================

// ✅ Good: Composable generic interfaces
type Reader[T any] interface {
    Read() (T, error)
}

type Writer[T any] interface {
    Write(T) error
}

type ReadWriter[T any] interface {
    Reader[T]
    Writer[T]
}

type Processor[T any] interface {
    Process(T) T
}

// Compose interfaces for specific needs
type ProcessingPipeline[T any] interface {
    Reader[T]
    Processor[T]
    Writer[T]
}

// =================================
// Guideline 7: Error handling with generics
// =================================

// ✅ Good: Generic error handling patterns
type Result[T any] struct {
    value T
    err   error
}

func Ok[T any](value T) Result[T] {
    return Result[T]{value: value, err: nil}
}

func Err[T any](err error) Result[T] {
    var zero T
    return Result[T]{value: zero, err: err}
}

func (r Result[T]) IsOk() bool {
    return r.err == nil
}

func (r Result[T]) Unwrap() (T, error) {
    return r.value, r.err
}

func (r Result[T]) Map[U any](fn func(T) U) Result[U] {
    if r.err != nil {
        return Err[U](r.err)
    }
    return Ok(fn(r.value))
}

// =================================
// Guideline 8: Testing generic code
// =================================

// ✅ Good: Test with multiple types
func TestGenericFunction() {
    // Test with different types
    intResult := Add(5, 3)
    floatResult := Add(5.5, 3.2)
    
    fmt.Printf("Int add: %d\n", intResult)
    fmt.Printf("Float add: %.2f\n", floatResult)
    
    // Test edge cases
    var zero int
    zeroResult := Add(zero, 0)
    fmt.Printf("Zero add: %d\n", zeroResult)
}

// =================================
// Guideline 9: Documentation for generics
// =================================

// ✅ Good: Well-documented generic function
// Transform applies the given function to each element in the slice,
// returning a new slice containing the transformed elements.
//
// Type parameters:
//   T: the input element type
//   U: the output element type
//
// The function fn must not be nil.
//
// Example:
//   numbers := []int{1, 2, 3}
//   strings := Transform(numbers, strconv.Itoa)
//   // strings is []string{"1", "2", "3"}
func Transform[T, U any](slice []T, fn func(T) U) []U {
    if fn == nil {
        panic("transform function cannot be nil")
    }
    
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = fn(v)
    }
    return result
}

// =================================
// Guideline 10: Performance considerations
// =================================

// ✅ Good: Consider allocation patterns
func EfficientMap[T, U any](slice []T, fn func(T) U) []U {
    // Pre-allocate with known capacity
    result := make([]U, 0, len(slice))
    for _, v := range slice {
        result = append(result, fn(v))
    }
    return result
}

// ✅ Good: Provide specialized versions for hot paths
func FastIntSum(numbers []int) int {
    sum := 0
    for _, n := range numbers {
        sum += n
    }
    return sum
}

// Generic version for other types
func GenericSum[T Numeric](numbers []T) T {
    var sum T
    for _, n := range numbers {
        sum += n
    }
    return sum
}

// =================================
// Guideline 11: API design with generics
// =================================

// ✅ Good: Progressive API design
type BasicContainer[T any] struct {
    items []T
}

// Level 1: Basic operations
func (bc *BasicContainer[T]) Add(item T) {
    bc.items = append(bc.items, item)
}

func (bc *BasicContainer[T]) Get(index int) (T, bool) {
    if index >= 0 && index < len(bc.items) {
        return bc.items[index], true
    }
    var zero T
    return zero, false
}

// Level 2: Operations requiring specific constraints
func ForEach[T any](bc *BasicContainer[T], fn func(T)) {
    for _, item := range bc.items {
        fn(item)
    }
}

func FindIf[T any](bc *BasicContainer[T], predicate func(T) bool) (T, bool) {
    for _, item := range bc.items {
        if predicate(item) {
            return item, true
        }
    }
    var zero T
    return zero, false
}

// Level 3: Advanced operations with constraints
func SortContainer[T Ordered](bc *BasicContainer[T]) {
    items := bc.items
    for i := 0; i < len(items)-1; i++ {
        for j := i + 1; j < len(items); j++ {
            if items[i] > items[j] {
                items[i], items[j] = items[j], items[i]
            }
        }
    }
}

// =================================
// Anti-patterns to avoid
// =================================

// ❌ Bad: Generic type with no constraints when you need them
type BadMath[T any] struct{}

func (bm *BadMath[T]) Add(a, b T) T {
    // This won't compile - no + operator for arbitrary T
    // return a + b
    panic("cannot add arbitrary types")
}

// ❌ Bad: Too many type parameters
type OverParameterized[T, U, V, W, X, Y, Z any] struct {
    // This is getting out of hand
}

// ❌ Bad: Pointless generic wrapper
type PointlessWrapper[T any] struct {
    Value T
}

// This adds no value over just using T directly

// =================================
// Best Practice Checklist
// =================================

func bestPracticeChecklist() {
    checklist := []string{
        "✓ Use descriptive constraint names",
        "✓ Start simple, add constraints as needed",
        "✓ Prefer generics over interface{} for type safety",
        "✓ Don't over-generalize simple cases",
        "✓ Leverage type inference when possible",
        "✓ Use composition over inheritance",
        "✓ Handle errors appropriately in generic code",
        "✓ Test with multiple types",
        "✓ Document type parameters and constraints",
        "✓ Consider performance implications",
        "✓ Design APIs progressively",
        "✓ Avoid unnecessary type parameters",
        "✓ Keep constraint complexity reasonable",
        "✓ Use specialized versions for hot paths",
        "✓ Follow Go naming conventions",
    }
    
    fmt.Println("=== Generic Programming Best Practices ===")
    for _, item := range checklist {
        fmt.Printf("  %s\n", item)
    }
}

func main() {
    fmt.Println("=== Go Generics Best Practices ===")
    
    fmt.Println("\n1. Type Inference Demo:")
    demonstrateTypeInference()
    
    fmt.Println("\n2. Generic Error Handling:")
    result := Ok(42)
    doubled := result.Map(func(n int) int { return n * 2 })
    if value, err := doubled.Unwrap(); err == nil {
        fmt.Printf("Doubled result: %d\n", value)
    }
    
    fmt.Println("\n3. Testing Generic Code:")
    TestGenericFunction()
    
    fmt.Println("\n4. API Design Example:")
    container := &BasicContainer[string]{}
    container.Add("apple")
    container.Add("banana")
    container.Add("cherry")
    
    ForEach(container, func(item string) {
        fmt.Printf("Item: %s\n", item)
    })
    
    if found, ok := FindIf(container, func(item string) bool {
        return strings.HasPrefix(item, "b")
    }); ok {
        fmt.Printf("Found item starting with 'b': %s\n", found)
    }
    
    fmt.Println("\n5. Performance Considerations:")
    numbers := []int{5, 2, 8, 1, 9}
    
    // Use specialized version for better performance
    fastSum := FastIntSum(numbers)
    fmt.Printf("Fast int sum: %d\n", fastSum)
    
    // Generic version for other types
    floats := []float64{5.5, 2.2, 8.8, 1.1, 9.9}
    genericSum := GenericSum(floats)
    fmt.Printf("Generic float sum: %.2f\n", genericSum)
    
    bestPracticeChecklist()
    
    fmt.Println("\n=== Summary ===")
    fmt.Println("Go generics enable type-safe, performant, and reusable code.")
    fmt.Println("Follow these best practices to write effective generic code:")
    fmt.Println("• Start simple and add complexity only when needed")
    fmt.Println("• Use descriptive names for type parameters and constraints")
    fmt.Println("• Test thoroughly with multiple types")
    fmt.Println("• Document your generic APIs clearly")
    fmt.Println("• Consider performance implications")
    fmt.Println("• Avoid over-engineering with unnecessary generics")
}
```

These best practices ensure that generic code remains readable, maintainable,  
and performant. The key is to use generics judiciously - they are a powerful  
tool that should enhance code safety and reusability without adding unnecessary  
complexity. Start simple, iterate based on real needs, and always prioritize  
clarity and performance in your generic designs.  

## Conclusion

Go generics represent a fundamental advancement in the language's type system,  
providing developers with powerful tools for writing type-safe, reusable, and  
performant code. Through these 60 comprehensive examples, we have explored  
every aspect of generic programming in Go, from basic type parameters to  
advanced design patterns, performance optimizations, and migration strategies.

The journey through Go generics reveals several key insights. First, generics  
solve real problems that have long challenged Go developers: the choice between  
code duplication and runtime type assertions with interface{}. By providing  
compile-time type checking with runtime performance, generics enable elegant  
solutions that were previously impossible or impractical.

Second, the design of Go generics emphasizes gradual adoption and simplicity.  
The syntax is clean and consistent with Go's philosophy, making generic code  
readable and maintainable. Type constraints using interfaces provide precise  
control over generic behavior while maintaining the language's commitment to  
explicit, understandable code.

Third, performance improvements with generics are substantial. By eliminating  
type assertions, reducing boxing overhead, and enabling compile-time  
specialization, generic code can significantly outperform interface{}-based  
alternatives. These improvements become critical in high-performance scenarios  
where every allocation and type assertion matters.

The examples demonstrate that generics enable powerful abstractions across all  
areas of Go programming. From basic data structures and algorithms to  
sophisticated patterns like builders, visitors, and state machines, generics  
provide type-safe building blocks that work with any appropriate type. This  
capability transforms how we think about library design and code reusability.

Error handling with generics introduces functional programming patterns that  
complement Go's traditional error handling approach. Result types, Optional  
types, and generic validation patterns provide additional tools for managing  
errors in a type-safe manner while maintaining Go's explicit error philosophy.

Concurrency patterns with generics showcase how type safety can extend to  
Go's powerful concurrency primitives. Generic channels, worker pools, and  
concurrent data structures provide reusable building blocks for parallel  
programming that maintain type safety across goroutines.

The migration strategies from interface{} to generics offer practical guidance  
for adopting generics in existing codebases. Gradual migration using  
compatibility layers and wrapper types minimizes risk while enabling teams  
to capture the benefits of improved type safety and performance.

Testing patterns for generic code ensure that the benefits of compile-time  
type checking extend to the testing phase. Generic test utilities, property-based  
testing, and type-safe mocking frameworks provide robust tools for validating  
generic code across multiple type instantiations.

Performance optimization techniques demonstrate that generics not only improve  
type safety but can also enhance runtime performance. Zero-allocation patterns,  
memory layout optimizations, and compile-time specialization showcase how  
generic code can achieve better performance than traditional approaches.

The best practices and guidelines provide a roadmap for effective generic  
programming in Go. These practices emphasize starting simple, using descriptive  
constraints, leveraging type inference, and avoiding over-engineering. The goal  
is to use generics where they add genuine value while maintaining code clarity  
and performance.

Looking forward, Go generics open new possibilities for library design and  
application architecture. They enable more expressive APIs, better performance,  
and improved type safety without sacrificing Go's core values of simplicity  
and clarity. As the ecosystem matures, we can expect to see generic patterns  
become standard practice in Go development.

The most important lesson from this comprehensive exploration is that generics  
are not just about avoiding code duplication or type assertions. They represent  
a new way of thinking about type relationships, code reusability, and API design  
in Go. When used thoughtfully, they enable developers to write more expressive,  
safer, and more performant code.

For developers beginning their journey with Go generics, start with simple  
patterns and gradually explore more advanced techniques. Focus on real problems  
that generics solve rather than using them for their own sake. Test thoroughly  
with multiple types, document your generic APIs clearly, and always consider  
the performance implications of your design choices.

Go generics mark the beginning of a new era in Go programming. They provide  
powerful tools while maintaining the language's commitment to simplicity,  
performance, and clarity. By mastering these patterns and techniques, developers  
can write more robust, reusable, and efficient Go code that fully leverages  
the capabilities of the modern Go type system.

The 60 examples in this guide provide a comprehensive foundation for generic  
programming in Go. From basic concepts to advanced patterns, from performance  
optimizations to best practices, these examples demonstrate the full potential  
of Go generics. Use them as a reference, a learning tool, and inspiration for  
your own generic programming adventures in Go.  
