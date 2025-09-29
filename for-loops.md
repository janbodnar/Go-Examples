# For loops

The `for` loop is Go's fundamental control structure for iteration and one of the most  
versatile constructs in the language. Unlike many other programming languages that  
provide multiple loop types (while, do-while, foreach), Go simplifies this by offering  
only the `for` loop, which can handle all iteration scenarios through various syntaxes.  

A `for` loop in Go can take several forms: the classic C-style loop with initialization,  
condition, and post-statement; a while-style loop with only a condition; and the  
range-based loop for iterating over collections. This design philosophy reflects Go's  
emphasis on simplicity and readability while maintaining powerful functionality.  

The `range` keyword is used with `for` keyword to define loops, which are used to iterate over elements  
in various data structures:  

- arrays
- slices
- maps
- strings
- channels

`range` automatically iterates through each element in the data structure.  

- for arrays and slices, it returns the index (integer) and the element value.  
- for maps, it returns the key and the corresponding value.  
- for strings, it returns the byte index (integer) and the Unicode character (rune).  
- for channels, it returns the value received from the channel (and no second value).  

The `for` loop's flexibility extends beyond basic iteration. It supports advanced  
features like labeled breaks and continues for nested loops, can be used to create  
infinite loops for server applications, and works seamlessly with goroutines for  
concurrent programming. Understanding these patterns is crucial for effective Go  
programming, as loops are fundamental to data processing, algorithm implementation,  
and system control flow.  


## C style

The classic C-style for loop includes initialization, condition, and post-statement.  
This form provides complete control over the loop's execution and is ideal when you  
need precise iteration control.  

```go
package main

import "fmt"

func main() {

    sum := 0

    for i := 0; i < 10; i++ {
    
        sum += i
    }
    
    fmt.Println(sum)
}
```

The example demonstrates a traditional counter loop that sums integers from 0 to 9.  
The loop initializes `i` to 0, continues while `i` is less than 10, and increments  
`i` after each iteration. This pattern is commonly used for precise counting and  
array indexing scenarios.  

## Single condition

The single condition loop is equivalent to a while loop in other languages.  
It continues executing as long as the condition remains true.  

```go
package main

import "fmt"

func main() {

    sum := 0
    i := 9

    for i > 0 {
        
        sum += i
        i--
    }

    fmt.Println(sum)
}
```

This example shows a countdown loop that sums values from 9 down to 1. The loop  
continues while `i` is greater than 0, making it useful for reverse iterations  
and conditions that don't follow a simple counter pattern.  

## Range clause

The range-based loop provides an elegant way to iterate over collections without  
manual index management. It automatically handles bounds checking and is the  
preferred method for processing slices, arrays, and other iterable types.  

```go
package main

import "fmt"

func main() {
    
    nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}

    sum := 0
    
    for _, num := range nums {
    
        sum += num
    }

    fmt.Println(sum)
}
```

This example uses the blank identifier `_` to ignore the index, focusing only on  
the values. The range keyword automatically iterates through each element in the  
slice, making the code more readable and less error-prone than manual indexing.  

## Using index

The first argument is index, the second is the element.  

```go
package main 

import "fmt"

func main() {

    words := []string{"sky", "cup", "cloud", "news", "water"}

    for idx, word := range words {

        fmt.Printf("%s has index %d\n", word, idx)
    }
}
```

## Range over integers

Go 1.22 introduced the ability to range directly over integers, providing a clean  
syntax for simple counting loops without creating intermediate slices.  

```go
package main

import "fmt"

func main() {

    for i := range 5 {
        fmt.Println(i)
    }

    for range 6 {
        fmt.Println("falcon")
    }
}
```

The first loop iterates from 0 to 4, providing access to the counter variable `i`.  
The second loop simply executes 6 times without needing the counter value, using  
the blank identifier to ignore it. This modern syntax reduces boilerplate code  
for simple repetitive operations.  

## Infinite loop

Using `for` without a condition creates an infinite loop. We must use `break` to  
terminate the loop.  

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {

    for {

        r := rand.Intn(30)

        fmt.Printf("%d ", r)

        if r == 22 {
            break
        }
    }
}
```

This infinite loop generates random numbers until it encounters the value 22.  
The `break` statement provides the only way to exit the loop. Infinite loops are  
commonly used in server applications, event handlers, and background processing  
where the termination condition is complex or external.  

## Basic counter

A simple counter loop demonstrates fundamental iteration patterns and is often  
used for initialization and basic repetitive tasks.  

```go
package main

import "fmt"

func main() {

    fmt.Println("Counting up:")
    for i := 1; i <= 5; i++ {
        fmt.Printf("Count: %d\n", i)
    }

    fmt.Println("Counting down:")
    for i := 5; i >= 1; i-- {
        fmt.Printf("Count: %d\n", i)
    }
}
```

This example shows both ascending and descending counter patterns. The first loop  
counts from 1 to 5, while the second counts backward from 5 to 1, demonstrating  
how the post-statement can be modified to control direction.  

## Range over strings

When ranging over strings, Go returns the byte index and the rune (Unicode  
character), which is essential for proper Unicode text processing.  

```go
package main

import "fmt"

func main() {

    message := "hello there"

    fmt.Println("Character by character:")
    for i, char := range message {
        fmt.Printf("Index %d: %c\n", i, char)
    }

    fmt.Println("Unicode example:")
    text := "café"
    for i, r := range text {
        fmt.Printf("Byte %d: %c (Unicode: %U)\n", i, r, r)
    }
}
```

The example demonstrates string iteration handling both ASCII and Unicode  
characters. Note how the byte index may not increment by 1 for multi-byte  
Unicode characters, making range the preferred method for string processing.  

## Range over maps

Maps provide key-value iteration where the order is not guaranteed, making  
them ideal for association lookups and data organization.  

```go
package main

import "fmt"

func main() {

    colors := map[string]string{
        "red":   "#FF0000",
        "green": "#00FF00",
        "blue":  "#0000FF",
        "black": "#000000",
    }

    fmt.Println("Color codes:")
    for name, code := range colors {
        fmt.Printf("%s: %s\n", name, code)
    }

    fmt.Println("Just the names:")
    for name := range colors {
        fmt.Println(name)
    }
}
```

This example shows both full key-value iteration and key-only iteration over  
a map. The iteration order is randomized in Go to prevent dependency on  
insertion order, encouraging robust programming practices.  

## Range over channels

Channel iteration automatically receives values until the channel is closed,  
providing elegant producer-consumer communication patterns.  

```go
package main

import "fmt"

func main() {

    ch := make(chan int, 5)

    // Send values to channel
    go func() {
        for i := 1; i <= 5; i++ {
            ch <- i * i
        }
        close(ch)
    }()

    // Receive values from channel
    fmt.Println("Squares received:")
    for square := range ch {
        fmt.Printf("Received: %d\n", square)
    }
}
```

The goroutine sends squared values to the channel and closes it when done.  
The range loop receives all values until the channel is closed, demonstrating  
safe concurrent communication without explicit synchronization.  

## Nested loops

Nested loops enable complex iteration patterns for multi-dimensional data  
processing and algorithmic operations.  

```go
package main

import "fmt"

func main() {

    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }

    fmt.Println("Matrix elements:")
    for i, row := range matrix {
        for j, value := range row {
            fmt.Printf("matrix[%d][%d] = %d\n", i, j, value)
        }
    }

    fmt.Println("Multiplication table:")
    for i := 1; i <= 3; i++ {
        for j := 1; j <= 3; j++ {
            fmt.Printf("%d x %d = %d\t", i, j, i*j)
        }
        fmt.Println()
    }
}
```

The first nested loop iterates over a 2D slice, accessing both indices and  
values. The second creates a multiplication table, showing how nested loops  
can generate structured output patterns.  

## Loop with continue

The `continue` statement skips the current iteration and proceeds to the next,  
allowing selective processing of loop elements.  

```go
package main

import "fmt"

func main() {

    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    fmt.Println("Even numbers only:")
    for _, num := range numbers {
        if num%2 != 0 {
            continue // Skip odd numbers
        }
        fmt.Printf("%d ", num)
    }
    fmt.Println()

    fmt.Println("Processing with conditions:")
    for i := 0; i < 10; i++ {
        if i < 3 {
            continue // Skip first 3
        }
        if i > 7 {
            continue // Skip after 7
        }
        fmt.Printf("Processing: %d\n", i)
    }
}
```

The first loop uses continue to skip odd numbers, printing only even values.  
The second demonstrates complex conditional skipping, processing only values  
between 3 and 7 inclusive.  

## Loop with break and labels

Labeled breaks provide precise control over nested loop termination, enabling  
complex flow control scenarios.  

```go
package main

import "fmt"

func main() {

    fmt.Println("Finding target in matrix:")

    matrix := [][]int{
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9},
    }

    target := 5

OuterLoop:
    for i, row := range matrix {
        for j, value := range row {
            fmt.Printf("Checking [%d][%d] = %d\n", i, j, value)
            if value == target {
                fmt.Printf("Found %d at position [%d][%d]\n", target, i, j)
                break OuterLoop
            }
        }
    }

    fmt.Println("Search completed")
}
```

The labeled break `OuterLoop` terminates both the inner and outer loops when  
the target value is found. This technique is essential for early termination  
in nested iteration structures without using function returns.  

## Range with blank identifier

The blank identifier `_` ignores unwanted values from range operations,  
improving code clarity and preventing unused variable warnings.  

```go
package main

import "fmt"

func main() {

    fruits := []string{"apple", "banana", "cherry", "date"}

    fmt.Println("Fruits with positions:")
    for idx, _ := range fruits {
        fmt.Printf("Position %d: %s\n", idx, fruits[idx])
    }

    fmt.Println("Just iterating over indices:")
    for idx := range fruits {
        fmt.Printf("Index: %d\n", idx)
    }

    fmt.Println("Executing 4 times:")
    for range fruits {
        fmt.Println("Action performed")
    }
}
```

The examples show different uses of the blank identifier: ignoring values while  
keeping indices, using range for index-only access, and using range purely  
for counting iterations without accessing any values.  

## Reverse iteration

Reverse iteration requires manual index control since range always goes forward,  
but provides efficient backward processing capabilities.  

```go
package main

import "fmt"

func main() {

    numbers := []int{10, 20, 30, 40, 50}

    fmt.Println("Forward iteration:")
    for i, num := range numbers {
        fmt.Printf("Index %d: %d\n", i, num)
    }

    fmt.Println("Reverse iteration:")
    for i := len(numbers) - 1; i >= 0; i-- {
        fmt.Printf("Index %d: %d\n", i, numbers[i])
    }

    fmt.Println("Reverse with range and slice:")
    for i := range numbers {
        idx := len(numbers) - 1 - i
        fmt.Printf("Position %d: %d\n", idx, numbers[idx])
    }
}
```

The first approach uses a traditional counter loop for reverse iteration.  
The second technique combines range with index calculation, maintaining the  
convenience of range while achieving reverse order processing.  

## Loop over array pointers

Working with array pointers in loops requires dereferencing to access the  
underlying array elements safely.  

```go
package main

import "fmt"

func main() {

    arr := [5]int{1, 2, 3, 4, 5}
    ptr := &arr

    fmt.Println("Iterating over array via pointer:")
    for i, value := range *ptr {
        fmt.Printf("Index %d: %d\n", i, value)
    }

    fmt.Println("Modifying through pointer:")
    for i := range *ptr {
        (*ptr)[i] *= 2
    }

    fmt.Println("Modified array:")
    for i, value := range arr {
        fmt.Printf("Index %d: %d\n", i, value)
    }
}
```

The example demonstrates dereferencing array pointers for both reading and  
writing operations. The asterisk operator `*` is required to access the  
array through the pointer in range expressions.  

## Range over slice of structs

Struct slices are common in Go applications, and range provides clean  
iteration patterns for processing structured data.  

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
    City string
}

func main() {

    people := []Person{
        {"Alice", 30, "New York"},
        {"Bob", 25, "London"},
        {"Charlie", 35, "Tokyo"},
    }

    fmt.Println("People information:")
    for i, person := range people {
        fmt.Printf("%d. %s (%d years) from %s\n", 
            i+1, person.Name, person.Age, person.City)
    }

    fmt.Println("Adults only:")
    for _, person := range people {
        if person.Age >= 30 {
            fmt.Printf("%s is an adult\n", person.Name)
        }
    }
}
```

This example shows struct iteration with both index access and value-only  
processing. The range operation provides direct access to struct fields  
without requiring pointer dereferencing or complex indexing.  

## Loop with defer

Combining loops with defer statements enables cleanup operations for each  
iteration, useful for resource management and debugging.  

```go
package main

import "fmt"

func processFile(filename string) {
    fmt.Printf("Opening file: %s\n", filename)
    defer fmt.Printf("Closing file: %s\n", filename)
    
    fmt.Printf("Processing file: %s\n", filename)
}

func main() {

    files := []string{"config.txt", "data.csv", "log.txt"}

    fmt.Println("Processing files:")
    for _, filename := range files {
        func() {
            defer fmt.Printf("Finished processing: %s\n", filename)
            processFile(filename)
        }()
    }

    fmt.Println("Cleanup with numbered operations:")
    for i := range 3 {
        func(id int) {
            defer fmt.Printf("Operation %d completed\n", id)
            fmt.Printf("Starting operation %d\n", id)
        }(i)
    }
}
```

The anonymous functions create separate defer contexts for each iteration,  
ensuring proper cleanup ordering. This pattern is essential when dealing  
with resources that need per-iteration cleanup.  

## Parallel processing with goroutines

Loops can launch goroutines for concurrent processing, enabling parallel  
execution of independent operations.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    
    fmt.Printf("Worker %d started\n", id)
    time.Sleep(time.Millisecond * 100)
    fmt.Printf("Worker %d finished\n", id)
}

func main() {

    var wg sync.WaitGroup

    fmt.Println("Starting parallel workers:")
    for i := range 5 {
        wg.Add(1)
        go worker(i+1, &wg)
    }

    wg.Wait()
    fmt.Println("All workers completed")

    fmt.Println("Processing data in parallel:")
    data := []int{1, 2, 3, 4, 5}
    results := make(chan int, len(data))

    for _, value := range data {
        go func(v int) {
            results <- v * v
        }(value)
    }

    for range data {
        result := <-results
        fmt.Printf("Result received: %d\n", result)
    }
    close(results)
}
```

The first loop launches concurrent workers using goroutines and WaitGroup  
for synchronization. The second example processes data in parallel using  
channels for result collection.  

## Range over map keys only

Iterating over map keys without values is useful for set operations and  
key-based processing scenarios.  

```go
package main

import (
    "fmt"
    "sort"
)

func main() {

    inventory := map[string]int{
        "apples":  50,
        "bananas": 30,
        "oranges": 25,
        "grapes":  40,
    }

    fmt.Println("Available items:")
    for item := range inventory {
        fmt.Printf("- %s\n", item)
    }

    // Collecting keys for sorting
    var items []string
    for item := range inventory {
        items = append(items, item)
    }

    sort.Strings(items)
    fmt.Println("Sorted inventory:")
    for _, item := range items {
        fmt.Printf("%s: %d units\n", item, inventory[item])
    }
}
```

The first loop iterates over keys only, useful for checking item existence.  
The second section collects keys into a slice for sorting, demonstrating  
how key-only iteration facilitates ordered processing of map data.  

## Range over map values only

Focusing on map values enables statistical operations and value-based  
processing without key considerations.  

```go
package main

import "fmt"

func main() {

    scores := map[string]int{
        "Alice":   95,
        "Bob":     87,
        "Charlie": 92,
        "Diana":   88,
    }

    fmt.Println("All scores:")
    total := 0
    count := 0
    for _, score := range scores {
        fmt.Printf("Score: %d\n", score)
        total += score
        count++
    }

    average := float64(total) / float64(count)
    fmt.Printf("Average score: %.2f\n", average)

    fmt.Println("Above average performers:")
    for name, score := range scores {
        if float64(score) > average {
            fmt.Printf("%s: %d (above average)\n", name, score)
        }
    }
}
```

The first loop processes values only for statistical calculation. The second  
loop combines key-value access to identify performers above the calculated  
average, showing practical value-focused processing patterns.  

## Custom iteration with functions

Function-based iteration provides flexible control over traversal patterns  
and enables custom iteration logic for specialized data structures.  

```go
package main

import "fmt"

func iterateEvens(max int, fn func(int)) {
    for i := 0; i <= max; i += 2 {
        fn(i)
    }
}

func fibonacci(n int, fn func(int, int)) {
    a, b := 0, 1
    for i := 0; i < n; i++ {
        fn(i, a)
        a, b = b, a+b
    }
}

func main() {

    fmt.Println("Even numbers up to 10:")
    iterateEvens(10, func(num int) {
        fmt.Printf("%d ", num)
    })
    fmt.Println()

    fmt.Println("First 8 Fibonacci numbers:")
    fibonacci(8, func(pos, value int) {
        fmt.Printf("F(%d) = %d\n", pos, value)
    })

    fmt.Println("Custom processing:")
    numbers := []int{1, 2, 3, 4, 5}
    process := func(nums []int, fn func(int, int)) {
        for i, num := range nums {
            fn(i, num*num)
        }
    }

    process(numbers, func(idx, square int) {
        fmt.Printf("Index %d: %d squared = %d\n", idx, numbers[idx], square)
    })
}
```

These examples show custom iteration functions that accept callback functions  
for processing. This pattern provides reusable iteration logic while allowing  
flexible processing behavior through function parameters.  

## Loop with error handling

Error handling in loops requires careful consideration of when to continue,  
break, or accumulate errors for batch processing.  

```go
package main

import (
    "errors"
    "fmt"
)

func processItem(item string) error {
    if item == "" {
        return errors.New("empty item not allowed")
    }
    if item == "error" {
        return errors.New("processing failed for item")
    }
    fmt.Printf("Successfully processed: %s\n", item)
    return nil
}

func main() {

    items := []string{"apple", "", "banana", "error", "cherry"}

    fmt.Println("Processing with continue on error:")
    for i, item := range items {
        if err := processItem(item); err != nil {
            fmt.Printf("Error at index %d: %v\n", i, err)
            continue
        }
    }

    fmt.Println("Collecting all errors:")
    var errors []error
    for i, item := range items {
        if err := processItem(item); err != nil {
            errors = append(errors, fmt.Errorf("item %d: %w", i, err))
        }
    }

    if len(errors) > 0 {
        fmt.Printf("Total errors encountered: %d\n", len(errors))
        for _, err := range errors {
            fmt.Printf("- %v\n", err)
        }
    }
}
```

The first approach handles errors individually with continue statements.  
The second collects all errors for batch reporting, useful in validation  
scenarios where all errors should be reported simultaneously.  

## Range over byte slice

Byte slice iteration is essential for binary data processing and low-level  
operations requiring byte-by-byte analysis.  

```go
package main

import "fmt"

func main() {

    text := "hello there"
    data := []byte(text)

    fmt.Println("Byte representation:")
    for i, b := range data {
        fmt.Printf("data[%d] = %d ('%c')\n", i, b, b)
    }

    fmt.Println("Hexadecimal view:")
    for i, b := range data {
        fmt.Printf("%02X ", b)
        if (i+1)%8 == 0 {
            fmt.Println()
        }
    }
    fmt.Println()

    fmt.Println("Finding specific bytes:")
    target := byte('e')
    positions := []int{}
    for i, b := range data {
        if b == target {
            positions = append(positions, i)
        }
    }
    fmt.Printf("Byte '%c' found at positions: %v\n", target, positions)
}
```

The example shows different approaches to byte slice processing: character  
representation, hexadecimal formatting, and pattern searching. This is  
fundamental for file processing and protocol implementation.  

## Loop with timer and ticker

Time-based loops enable periodic operations and timeout handling, essential  
for scheduled tasks and real-time applications.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {

    fmt.Println("Timer-based processing:")
    timer := time.NewTimer(2 * time.Second)
    counter := 0

    for {
        select {
        case <-timer.C:
            fmt.Println("Timer expired, stopping loop")
            return
        default:
            counter++
            fmt.Printf("Work iteration: %d\n", counter)
            time.Sleep(300 * time.Millisecond)
        }
    }
}

func demonstrateTicker() {
    fmt.Println("Ticker-based periodic task:")
    ticker := time.NewTicker(500 * time.Millisecond)
    defer ticker.Stop()

    count := 0
    for {
        select {
        case <-ticker.C:
            count++
            fmt.Printf("Tick %d at %s\n", count, time.Now().Format("15:04:05"))
            if count >= 5 {
                fmt.Println("Completed 5 ticks, stopping")
                return
            }
        }
    }
}
```

The timer example creates a timeout-controlled loop that performs work until  
the timer expires. The ticker function (called separately) demonstrates  
periodic task execution with precise timing intervals.  

## Complex nested loops with data processing

Complex nested iterations handle multi-dimensional data processing scenarios  
common in algorithms, data analysis, and matrix operations.  

```go
package main

import "fmt"

type Student struct {
    Name    string
    Grades  map[string]int
    Average float64
}

func main() {

    students := []Student{
        {"Alice", map[string]int{"Math": 95, "Physics": 87, "Chemistry": 92}, 0},
        {"Bob", map[string]int{"Math": 78, "Physics": 82, "Chemistry": 79}, 0},
        {"Charlie", map[string]int{"Math": 88, "Physics": 91, "Chemistry": 85}, 0},
    }

    fmt.Println("Computing student averages:")
    for i := range students {
        total := 0
        count := 0
        
        for subject, grade := range students[i].Grades {
            fmt.Printf("%s - %s: %d\n", students[i].Name, subject, grade)
            total += grade
            count++
        }
        
        students[i].Average = float64(total) / float64(count)
        fmt.Printf("%s's average: %.2f\n\n", students[i].Name, students[i].Average)
    }

    fmt.Println("Subject-wise analysis:")
    subjects := make(map[string][]int)
    
    for _, student := range students {
        for subject, grade := range student.Grades {
            subjects[subject] = append(subjects[subject], grade)
        }
    }

    for subject, grades := range subjects {
        total := 0
        for _, grade := range grades {
            total += grade
        }
        average := float64(total) / float64(len(grades))
        fmt.Printf("%s average across all students: %.2f\n", subject, average)
    }
}
```

This complex example demonstrates nested loops processing student data with  
multiple dimensions: students, subjects, and grades. It calculates individual  
averages and subject-wise statistics, showing practical multi-level iteration  
patterns used in data processing applications.  
