# Slices

In Go, a slice is a dynamically-sized, flexible view into the elements of an  
array. Slices are more common than arrays in Go programming because they  
provide a more powerful interface to sequences of data. A slice is formed by  
specifying two indices, a low and high bound, separated by a colon: `a[low:high]`.  
This selects a half-open range which includes the first element, but excludes  
the last one.

Slices are reference types, meaning they don't store data themselves but  
instead point to an underlying array. When you modify elements through a  
slice, you're modifying the underlying array. Multiple slices can share the  
same underlying array, which makes slices both powerful and memory-efficient.  
The zero value of a slice is `nil`.

A slice has three components: a pointer to the array, length, and capacity.  
Length is the number of elements the slice contains, while capacity is the  
number of elements in the underlying array, counting from the first element  
in the slice. You can check these with the built-in `len()` and `cap()`  
functions respectively.

Go provides a rich `slices` package (introduced in Go 1.21) that offers many  
utility functions for working with slices efficiently. These functions include  
searching, sorting, comparing, and manipulating slices in various ways. The  
package follows generic programming principles, making it type-safe and  
efficient for any comparable types.

## Basic slice creation

The most straightforward way to create slices in Go.  

```go
package main

import "fmt"

func main() {

    // Slice literal
    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println("Numbers:", numbers)
    
    // Empty slice
    var empty []string
    fmt.Println("Empty slice:", empty)
    fmt.Println("Is nil:", empty == nil)
    
    // Using make
    made := make([]int, 5)
    fmt.Println("Made slice:", made)
    
    // Make with capacity
    withCap := make([]int, 3, 10)
    fmt.Printf("Length: %d, Capacity: %d\n", len(withCap), cap(withCap))
}
```

This example demonstrates the various ways to create slices. Slice literals  
provide initial values, while `make()` creates slices with specified length  
and optional capacity. Empty slices created with `var` are `nil`.  

## Slice from array

Creating slices from existing arrays using slice expressions.  

```go
package main

import "fmt"

func main() {

    arr := [6]string{"red", "green", "blue", "yellow", "orange", "purple"}
    fmt.Println("Original array:", arr)
    
    // Slice entire array
    all := arr[:]
    fmt.Println("All elements:", all)
    
    // Slice with start index
    fromIndex := arr[2:]
    fmt.Println("From index 2:", fromIndex)
    
    // Slice with end index
    toIndex := arr[:4]
    fmt.Println("To index 4:", toIndex)
    
    // Slice with both indices
    middle := arr[1:4]
    fmt.Println("Middle slice:", middle)
}
```

Slice expressions create views into arrays or other slices. The syntax  
`[low:high]` creates a slice including elements from index `low` up to but  
not including index `high`. Omitting indices defaults to 0 and length.  

## Append operation

Growing slices dynamically using the append function.  

```go
package main

import "fmt"

func main() {

    fruits := []string{"apple", "banana"}
    fmt.Println("Initial fruits:", fruits)
    fmt.Printf("Length: %d, Capacity: %d\n", len(fruits), cap(fruits))
    
    // Append single element
    fruits = append(fruits, "orange")
    fmt.Println("After appending orange:", fruits)
    
    // Append multiple elements
    fruits = append(fruits, "grape", "mango")
    fmt.Println("After appending multiple:", fruits)
    fmt.Printf("Length: %d, Capacity: %d\n", len(fruits), cap(fruits))
    
    // Append another slice
    more := []string{"peach", "pear"}
    fruits = append(fruits, more...)
    fmt.Println("After appending slice:", fruits)
}
```

The `append()` function adds elements to the end of a slice and returns the  
updated slice. When the slice's capacity is exceeded, Go automatically  
allocates a new underlying array with increased capacity, typically doubling it.  

## Copying slices

Creating independent copies of slices to avoid shared references.  

```go
package main

import "fmt"

func main() {

    original := []int{1, 2, 3, 4, 5}
    fmt.Println("Original:", original)
    
    // Create copy using make and copy
    copied := make([]int, len(original))
    n := copy(copied, original)
    fmt.Printf("Copied %d elements: %v\n", n, copied)
    
    // Modify original
    original[0] = 99
    fmt.Println("Modified original:", original)
    fmt.Println("Copy unchanged:", copied)
    
    // Partial copy
    partial := make([]int, 3)
    copy(partial, original)
    fmt.Println("Partial copy:", partial)
}
```

The `copy()` function creates independent copies of slice data. This prevents  
modifications to one slice from affecting another. The function returns the  
number of elements copied, which is the minimum of source and destination lengths.  

## Slice length and capacity

Understanding the difference between slice length and capacity.  

```go
package main

import "fmt"

func main() {

    s := make([]int, 3, 8)
    fmt.Printf("Initial - len: %d, cap: %d, slice: %v\n", len(s), cap(s), s)
    
    // Add elements
    s = append(s, 10, 20, 30)
    fmt.Printf("After append - len: %d, cap: %d, slice: %v\n", len(s), cap(s), s)
    
    // Demonstrate capacity growth
    for i := 0; i < 5; i++ {
        s = append(s, i)
        fmt.Printf("len: %d, cap: %d\n", len(s), cap(s))
    }
    
    // Slice reduces length but not capacity
    shortened := s[:3]
    fmt.Printf("Shortened - len: %d, cap: %d\n", len(shortened), cap(shortened))
}
```

Length represents the number of elements currently in the slice, while capacity  
is the maximum number of elements the slice can hold without reallocation.  
When capacity is exceeded, Go allocates a new underlying array.  

## Slice iteration

Different patterns for iterating over slice elements.  

```go
package main

import "fmt"

func main() {

    colors := []string{"red", "green", "blue", "yellow"}
    
    // Traditional for loop
    fmt.Println("Traditional for loop:")
    for i := 0; i < len(colors); i++ {
        fmt.Printf("Index %d: %s\n", i, colors[i])
    }
    
    fmt.Println("\nRange with index and value:")
    for i, color := range colors {
        fmt.Printf("Index %d: %s\n", i, color)
    }
    
    fmt.Println("\nRange with value only:")
    for _, color := range colors {
        fmt.Printf("Color: %s\n", color)
    }
    
    fmt.Println("\nRange with index only:")
    for i := range colors {
        fmt.Printf("Index: %d\n", i)
    }
}
```

Go provides flexible iteration patterns for slices. The `range` keyword offers  
convenient access to both indices and values, with the blank identifier `_`  
used to ignore unwanted values during iteration.  

## Slice contains check

Checking if a slice contains specific elements using the slices package.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{10, 20, 30, 40, 50}
    fmt.Println("Numbers:", numbers)
    
    // Check if slice contains value
    fmt.Println("Contains 30:", slices.Contains(numbers, 30))
    fmt.Println("Contains 60:", slices.Contains(numbers, 60))
    
    // Check with strings
    words := []string{"apple", "banana", "cherry"}
    fmt.Println("Words contain 'banana':", slices.Contains(words, "banana"))
    
    // Check multiple values
    targets := []int{20, 30, 60}
    for _, target := range targets {
        found := slices.Contains(numbers, target)
        fmt.Printf("Contains %d: %t\n", target, found)
    }
}
```

The `slices.Contains()` function provides an efficient way to check if a slice  
contains a specific value. It returns a boolean and works with any comparable  
type, making it more convenient than manual iteration.  

## Finding indices

Locating elements within slices using index-based searches.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    fruits := []string{"apple", "banana", "cherry", "banana", "date"}
    fmt.Println("Fruits:", fruits)
    
    // Find first occurrence
    firstBanana := slices.Index(fruits, "banana")
    fmt.Printf("First 'banana' at index: %d\n", firstBanana)
    
    // Element not found returns -1
    notFound := slices.Index(fruits, "grape")
    fmt.Printf("'grape' index: %d\n", notFound)
    
    // Find using custom comparison
    numbers := []int{5, 10, 15, 20, 25}
    greaterThan12 := slices.IndexFunc(numbers, func(n int) bool {
        return n > 12
    })
    fmt.Printf("First number > 12 at index: %d\n", greaterThan12)
}
```

Index functions help locate specific elements or elements meeting certain  
criteria. `slices.Index()` finds exact matches, while `slices.IndexFunc()`  
allows custom matching logic through predicates.  

## Minimum and maximum

Finding extreme values in slices using built-in functions.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    scores := []int{85, 92, 78, 96, 88, 74, 91}
    fmt.Println("Scores:", scores)
    
    // Find minimum and maximum
    minScore := slices.Min(scores)
    maxScore := slices.Max(scores)
    fmt.Printf("Minimum score: %d\n", minScore)
    fmt.Printf("Maximum score: %d\n", maxScore)
    
    // With floating point numbers
    prices := []float64{12.99, 8.50, 15.25, 9.75, 11.00}
    fmt.Println("Prices:", prices)
    fmt.Printf("Cheapest: $%.2f\n", slices.Min(prices))
    fmt.Printf("Most expensive: $%.2f\n", slices.Max(prices))
    
    // With strings (lexicographic order)
    names := []string{"Alice", "Bob", "Charlie", "David"}
    fmt.Printf("First name: %s\n", slices.Min(names))
    fmt.Printf("Last name: %s\n", slices.Max(names))
}
```

The `slices.Min()` and `slices.Max()` functions work with any ordered type,  
including numbers and strings. For strings, comparison follows lexicographic  
(dictionary) order, making these functions versatile for many use cases.  

## Sorting slices

Arranging slice elements in ascending order using sort functions.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{42, 15, 8, 23, 4, 16}
    fmt.Println("Original numbers:", numbers)
    
    // Sort in-place
    slices.Sort(numbers)
    fmt.Println("Sorted numbers:", numbers)
    
    // Sort strings
    words := []string{"zebra", "apple", "cherry", "banana"}
    fmt.Println("Original words:", words)
    slices.Sort(words)
    fmt.Println("Sorted words:", words)
    
    // Check if sorted
    values := []int{1, 3, 5, 7, 9}
    fmt.Printf("Is %v sorted: %t\n", values, slices.IsSorted(values))
    
    unsorted := []int{1, 5, 3, 7}
    fmt.Printf("Is %v sorted: %t\n", unsorted, slices.IsSorted(unsorted))
}
```

The `slices.Sort()` function modifies the slice in-place, arranging elements  
in ascending order. `slices.IsSorted()` efficiently checks if a slice is  
already sorted, which can be useful for optimization decisions.  

## Custom sorting

Sorting slices with custom comparison functions for complex ordering.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    // Sort by absolute value
    numbers := []int{-5, 3, -8, 1, -2, 7}
    fmt.Println("Original numbers:", numbers)
    
    slices.SortFunc(numbers, func(a, b int) int {
        absA := a
        if absA < 0 {
            absA = -absA
        }
        absB := b
        if absB < 0 {
            absB = -absB
        }
        if absA < absB {
            return -1
        }
        if absA > absB {
            return 1
        }
        return 0
    })
    fmt.Println("Sorted by absolute value:", numbers)
    
    // Sort strings by length
    words := []string{"elephant", "cat", "hippopotamus", "dog", "bird"}
    fmt.Println("Original words:", words)
    
    slices.SortFunc(words, func(a, b string) int {
        return len(a) - len(b)
    })
    fmt.Println("Sorted by length:", words)
}
```

`slices.SortFunc()` accepts a comparison function that returns negative, zero,  
or positive values for less-than, equal, or greater-than relationships. This  
enables complex sorting criteria beyond natural ordering.  

## Reversing slices

Reversing the order of elements in slices using different approaches.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    original := []string{"first", "second", "third", "fourth", "fifth"}
    fmt.Println("Original:", original)
    
    // Using slices.Reverse
    reversed := slices.Clone(original)
    slices.Reverse(reversed)
    fmt.Println("Reversed:", reversed)
    
    // Manual reverse (in-place)
    manual := []int{1, 2, 3, 4, 5, 6}
    fmt.Println("Before manual reverse:", manual)
    
    for i, j := 0, len(manual)-1; i < j; i, j = i+1, j-1 {
        manual[i], manual[j] = manual[j], manual[i]
    }
    fmt.Println("After manual reverse:", manual)
    
    // Reverse slice of runes
    chars := []rune("hello")
    slices.Reverse(chars)
    fmt.Println("Reversed string:", string(chars))
}
```

The `slices.Reverse()` function efficiently reverses slices in-place by  
swapping elements from both ends. Manual reversal demonstrates the underlying  
algorithm and provides insight into slice manipulation techniques.  

## Comparing slices

Checking equality between slices using various comparison methods.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    slice1 := []int{1, 2, 3, 4, 5}
    slice2 := []int{1, 2, 3, 4, 5}
    slice3 := []int{1, 2, 3, 5, 4}
    
    fmt.Println("Slice1:", slice1)
    fmt.Println("Slice2:", slice2)
    fmt.Println("Slice3:", slice3)
    
    // Equal slices
    fmt.Printf("slice1 == slice2: %t\n", slices.Equal(slice1, slice2))
    fmt.Printf("slice1 == slice3: %t\n", slices.Equal(slice1, slice3))
    
    // Compare function for custom comparison
    result := slices.Compare(slice1, slice2)
    fmt.Printf("Compare slice1 to slice2: %d\n", result)
    
    result = slices.Compare(slice1, slice3)
    fmt.Printf("Compare slice1 to slice3: %d\n", result)
    
    // String comparison
    words1 := []string{"apple", "banana", "cherry"}
    words2 := []string{"apple", "banana", "cherry"}
    fmt.Printf("String slices equal: %t\n", slices.Equal(words1, words2))
}
```

`slices.Equal()` checks if two slices have the same length and all elements  
are equal. `slices.Compare()` returns -1, 0, or 1 for lexicographic ordering,  
useful for implementing sorting or search algorithms.  

## Cloning slices

Creating independent copies of slices with identical content.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    original := []string{"red", "green", "blue"}
    fmt.Println("Original:", original)
    fmt.Printf("Original address: %p\n", original)
    
    // Clone creates independent copy
    cloned := slices.Clone(original)
    fmt.Println("Cloned:", cloned)
    fmt.Printf("Cloned address: %p\n", cloned)
    
    // Modify original
    original[0] = "yellow"
    fmt.Println("After modifying original:")
    fmt.Println("Original:", original)
    fmt.Println("Cloned (unchanged):", cloned)
    
    // Clone with numbers
    numbers := []int{10, 20, 30}
    numbersCopy := slices.Clone(numbers)
    
    // Append to copy doesn't affect original
    numbersCopy = append(numbersCopy, 40, 50)
    fmt.Println("Original numbers:", numbers)
    fmt.Println("Extended copy:", numbersCopy)
}
```

`slices.Clone()` creates a shallow copy with new underlying storage, ensuring  
complete independence between original and cloned slices. Changes to one slice  
don't affect the other, including length modifications.  

## Deleting elements

Removing elements from slices using the delete function.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    colors := []string{"red", "green", "blue", "yellow", "orange"}
    fmt.Println("Original colors:", colors)
    
    // Delete single element at index 2
    colors = slices.Delete(colors, 2, 3)
    fmt.Println("After deleting index 2:", colors)
    
    // Delete range of elements
    numbers := []int{10, 20, 30, 40, 50, 60, 70}
    fmt.Println("Original numbers:", numbers)
    
    // Delete elements from index 1 to 4 (exclusive)
    numbers = slices.Delete(numbers, 1, 4)
    fmt.Println("After deleting range [1:4]:", numbers)
    
    // Delete from end
    fruits := []string{"apple", "banana", "cherry", "date", "elderberry"}
    fruits = slices.Delete(fruits, len(fruits)-2, len(fruits))
    fmt.Println("After deleting last 2:", fruits)
}
```

`slices.Delete()` removes elements between two indices and returns a new slice.  
The function efficiently shifts remaining elements to fill the gap, maintaining  
slice continuity without leaving empty spaces.  

## Inserting elements

Adding elements at specific positions within slices.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    animals := []string{"cat", "dog", "rabbit"}
    fmt.Println("Original animals:", animals)
    
    // Insert single element at index 1
    animals = slices.Insert(animals, 1, "bird")
    fmt.Println("After inserting 'bird' at index 1:", animals)
    
    // Insert multiple elements
    numbers := []int{1, 2, 5, 6}
    fmt.Println("Original numbers:", numbers)
    
    numbers = slices.Insert(numbers, 2, 3, 4)
    fmt.Println("After inserting 3,4 at index 2:", numbers)
    
    // Insert at beginning
    fruits := []string{"banana", "cherry"}
    fruits = slices.Insert(fruits, 0, "apple")
    fmt.Println("After inserting at beginning:", fruits)
    
    // Insert at end (equivalent to append)
    vegetables := []string{"carrot", "broccoli"}
    vegetables = slices.Insert(vegetables, len(vegetables), "spinach")
    fmt.Println("After inserting at end:", vegetables)
}
```

`slices.Insert()` adds one or more elements at a specified position, shifting  
existing elements to make room. This operation can insert at any valid position  
including the beginning, middle, or end of the slice.  

## Replacing elements

Substituting slice segments with new values using replace operations.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    letters := []string{"a", "b", "c", "d", "e", "f"}
    fmt.Println("Original letters:", letters)
    
    // Replace single element
    letters = slices.Replace(letters, 2, 3, "X")
    fmt.Println("After replacing index 2:", letters)
    
    // Replace range with multiple elements
    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println("Original numbers:", numbers)
    
    numbers = slices.Replace(numbers, 1, 4, 10, 20, 30)
    fmt.Println("After replacing indices 1-3:", numbers)
    
    // Replace with fewer elements
    colors := []string{"red", "green", "blue", "yellow", "orange"}
    colors = slices.Replace(colors, 1, 4, "purple")
    fmt.Println("After replacing with fewer elements:", colors)
    
    // Replace with more elements
    days := []string{"Mon", "Tue", "Fri"}
    days = slices.Replace(days, 2, 3, "Wed", "Thu", "Fri")
    fmt.Println("After expanding replacement:", days)
}
```

`slices.Replace()` removes elements in a range and inserts new elements in  
their place. The replacement can be shorter, longer, or the same length as  
the original range, making it very flexible for slice modifications.  

## Growing slices

Increasing slice capacity efficiently using the grow function.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{1, 2, 3}
    fmt.Printf("Initial - len: %d, cap: %d\n", len(numbers), cap(numbers))
    
    // Grow to accommodate more elements efficiently
    numbers = slices.Grow(numbers, 10)
    fmt.Printf("After growing by 10 - len: %d, cap: %d\n", len(numbers), cap(numbers))
    
    // Demonstrate efficiency
    start := []string{"apple"}
    fmt.Printf("Start - len: %d, cap: %d\n", len(start), cap(start))
    
    // Grow before adding many elements
    start = slices.Grow(start, 1000)
    fmt.Printf("After growing - len: %d, cap: %d\n", len(start), cap(start))
    
    // Now append operations won't cause reallocations
    for i := 0; i < 100; i++ {
        start = append(start, fmt.Sprintf("item%d", i))
    }
    fmt.Printf("After appending 100 items - len: %d, cap: %d\n", len(start), cap(start))
}
```

`slices.Grow()` pre-allocates capacity to prevent multiple reallocations during  
subsequent append operations. This optimization is particularly useful when you  
know approximately how many elements will be added to a slice.  

## Compacting slices

Removing consecutive duplicate elements to create compact slices.  

```go
package main

import (
    "fmt"
    "slices"
    "strings"
)

func main() {

    // Compact removes consecutive duplicates
    numbers := []int{1, 1, 2, 2, 2, 3, 1, 1, 4, 4}
    fmt.Println("Original numbers:", numbers)
    
    compacted := slices.Compact(numbers)
    fmt.Println("After compacting:", compacted)
    
    // Compact with strings
    words := []string{"apple", "apple", "banana", "banana", "cherry", "cherry", "cherry"}
    fmt.Println("Original words:", words)
    
    compactedWords := slices.Compact(words)
    fmt.Println("After compacting words:", compactedWords)
    
    // CompactFunc with custom comparison
    items := []string{"Apple", "APPLE", "apple", "Banana", "BANANA", "Cherry"}
    fmt.Println("Original items:", items)
    
    compactedItems := slices.CompactFunc(items, func(a, b string) bool {
        // Compare case-insensitive
        lowerA := strings.ToLower(a)
        lowerB := strings.ToLower(b)
        return lowerA == lowerB
    })
    fmt.Println("After case-insensitive compact:", compactedItems)
}
```

`slices.Compact()` removes consecutive duplicate elements, creating a more  
concise representation. `slices.CompactFunc()` allows custom equality testing,  
useful for case-insensitive comparisons or complex object comparisons.  

## Binary search

Efficiently searching in sorted slices using binary search algorithms.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    // Binary search requires sorted slice
    sortedNumbers := []int{2, 4, 6, 8, 10, 12, 14, 16, 18, 20}
    fmt.Println("Sorted numbers:", sortedNumbers)
    
    // Find exact value
    target := 12
    index, found := slices.BinarySearch(sortedNumbers, target)
    fmt.Printf("Search for %d: index=%d, found=%t\n", target, index, found)
    
    // Search for non-existent value
    target = 13
    index, found = slices.BinarySearch(sortedNumbers, target)
    fmt.Printf("Search for %d: index=%d, found=%t\n", target, index, found)
    
    // BinarySearchFunc with custom comparison
    names := []string{"Alice", "Bob", "Charlie", "David", "Eve"}
    fmt.Println("Names:", names)
    
    searchName := "Charlie"
    index, found = slices.BinarySearch(names, searchName)
    fmt.Printf("Search for '%s': index=%d, found=%t\n", searchName, index, found)
    
    // Search in larger slice
    bigSlice := make([]int, 1000)
    for i := range bigSlice {
        bigSlice[i] = i * 2
    }
    
    target = 500
    index, found = slices.BinarySearch(bigSlice, target)
    fmt.Printf("Search for %d in big slice: index=%d, found=%t\n", target, index, found)
}
```

Binary search provides O(log n) search time in sorted slices, much faster than  
linear search for large datasets. The function returns both the index and a  
boolean indicating whether the exact value was found.  

## Iterating backwards

Traversing slices in reverse order using the backward iterator.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    vals := []int{1, 2, 3, 4, 5, 6, 7, 8}
    fmt.Println("Original slice:", vals)
    
    fmt.Println("Backward iteration:")
    for val := range slices.Backward(vals) {
        fmt.Println(val)
    }
    
    fmt.Println("\nBackward with strings:")
    words := []string{"first", "second", "third", "fourth"}
    for word := range slices.Backward(words) {
        fmt.Printf("Word: %s\n", word)
    }
    
    fmt.Println("\nCollecting backward values:")
    reversed := []int{}
    for val := range slices.Backward(vals) {
        reversed = append(reversed, val)
    }
    fmt.Println("Collected in reverse:", reversed)
}
```

`slices.Backward()` returns an iterator that traverses the slice from end to  
beginning. This is useful for reverse processing without modifying the  
original slice or creating temporary reversed copies.  

## Chunking slices

Dividing slices into smaller segments of fixed size.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    vals := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    fmt.Println("Original vals:", vals)
    
    fmt.Println("\nChunking by 3:")
    for chunk := range slices.Chunk(vals, 3) {
        fmt.Println("Chunk:", chunk)
    }
    
    fmt.Println("\nChunking strings by 2:")
    words := []string{"apple", "banana", "cherry", "date", "elderberry", "fig", "grape"}
    
    for chunk := range slices.Chunk(words, 2) {
        fmt.Printf("Word pair: %v\n", chunk)
    }
    
    fmt.Println("\nProcessing chunks:")
    numbers := []int{10, 20, 30, 40, 50, 60, 70}
    
    for chunk := range slices.Chunk(numbers, 4) {
        sum := 0
        for _, num := range chunk {
            sum += num
        }
        fmt.Printf("Chunk %v sum: %d\n", chunk, sum)
    }
}
```

`slices.Chunk()` creates an iterator that yields successive slices of the  
specified size. The final chunk may be smaller if the slice length isn't  
evenly divisible by the chunk size, making it useful for batch processing.  

## All iterator

Creating iterators that yield both index and value pairs.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    colors := []string{"red", "green", "blue", "yellow", "purple"}
    fmt.Println("Colors:", colors)
    
    fmt.Println("\nUsing All iterator:")
    for i, color := range slices.All(colors) {
        fmt.Printf("Index %d: %s\n", i, color)
    }
    
    fmt.Println("\nFiltering with All iterator:")
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    
    for i, num := range slices.All(numbers) {
        if num%2 == 0 {
            fmt.Printf("Even number at index %d: %d\n", i, num)
        }
    }
    
    fmt.Println("\nCollecting indices:")
    evenIndices := []int{}
    for i, num := range slices.All(numbers) {
        if num%2 == 0 {
            evenIndices = append(evenIndices, i)
        }
    }
    fmt.Println("Indices of even numbers:", evenIndices)
}
```

`slices.All()` provides access to both index and value during iteration,  
similar to the standard range operator but as an explicit iterator. This is  
useful for functional programming patterns and iterator composition.  

## Values iterator

Iterating over slice values without indices using values iterator.  

```go
package main

import (
    "fmt"
    "slices"
    "strings"
)

func main() {

    words := []string{"sky", "ten", "water", "forest", "cup"}
    fmt.Println("Words:", words)
    
    fmt.Println("\nUsing Values iterator:")
    for word := range slices.Values(words) {
        fmt.Printf("Word: %s (length: %d)\n", word, len(word))
    }
    
    fmt.Println("\nFiltering with Values:")
    numbers := []int{15, 8, 23, 42, 7, 31, 19}
    
    fmt.Println("Numbers greater than 20:")
    for num := range slices.Values(numbers) {
        if num > 20 {
            fmt.Printf("  %d\n", num)
        }
    }
    
    fmt.Println("\nTransforming values:")
    names := []string{"alice", "bob", "charlie"}
    uppercased := []string{}
    
    for name := range slices.Values(names) {
        uppercased = append(uppercased, strings.ToUpper(name))
    }
    fmt.Println("Uppercased names:", uppercased)
}
```

`slices.Values()` creates an iterator that yields only values, ignoring indices.  
This is particularly useful when you only need the values for processing or  
transformation and want to avoid the overhead of unused index variables.  

## Sorted iterator

Creating sorted views of slices without modifying the original.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    words := []string{"zebra", "apple", "cherry", "banana", "date"}
    fmt.Println("Original words:", words)
    
    // Create iterator for values and sort them
    it := slices.Values(words)
    sorted_words := slices.Sorted(it)
    
    fmt.Println("Sorted words:")
    for _, word := range sorted_words {
        fmt.Println("  ", word)
    }
    
    fmt.Println("Original unchanged:", words)
    
    // Sort numbers
    numbers := []int{64, 23, 1, 42, 78, 9, 34}
    fmt.Println("\nOriginal numbers:", numbers)
    
    numIterator := slices.Values(numbers)
    sortedNumbers := slices.Sorted(numIterator)
    fmt.Println("Sorted numbers:", sortedNumbers)
    
    // Sort with transformation
    mixed := []string{"Apple", "banana", "Cherry", "date"}
    fmt.Println("\nMixed case:", mixed)
    
    // Note: This creates a new slice, original is unchanged
    sortedMixed := slices.Sorted(slices.Values(mixed))
    fmt.Println("Sorted mixed:", sortedMixed)
}
```

`slices.Sorted()` takes an iterator and returns a new sorted slice without  
modifying the original. This is useful when you need a sorted view of data  
while preserving the original order for other operations.  

## Collecting from iterators

Converting iterators back into slices using collect operations.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println("Original numbers:", numbers)
    
    // Collect all values
    it := slices.Values(numbers)
    collected := slices.Collect(it)
    fmt.Println("Collected values:", collected)
    
    // Collect from filtered iterator
    evenNumbers := []int{}
    for num := range slices.Values(numbers) {
        if num%2 == 0 {
            evenNumbers = append(evenNumbers, num)
        }
    }
    fmt.Println("Even numbers:", evenNumbers)
    
    // Collect from backward iterator
    reversed := slices.Collect(slices.Backward(numbers))
    fmt.Println("Reversed using collect:", reversed)
    
    // Collect chunks into 2D slice
    data := []int{1, 2, 3, 4, 5, 6, 7, 8}
    chunks := [][]int{}
    
    for chunk := range slices.Chunk(data, 3) {
        // Clone chunk since it's reused by the iterator
        chunkCopy := slices.Clone(chunk)
        chunks = append(chunks, chunkCopy)
    }
    fmt.Println("Collected chunks:", chunks)
}
```

`slices.Collect()` converts iterators back into concrete slices. This is  
essential when you need to store iterator results or pass them to functions  
that expect slice types rather than iterator interfaces.  

## Nil slice handling

Working safely with nil slices and understanding their behavior.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    var nilSlice []int
    fmt.Printf("Nil slice: %v\n", nilSlice)
    fmt.Printf("Length: %d, Capacity: %d\n", len(nilSlice), cap(nilSlice))
    fmt.Printf("Is nil: %t\n", nilSlice == nil)
    
    // Safe operations on nil slices
    fmt.Println("Contains 0:", slices.Contains(nilSlice, 0))
    fmt.Println("Index of 0:", slices.Index(nilSlice, 0))
    
    // Append to nil slice creates new slice
    nilSlice = append(nilSlice, 10, 20, 30)
    fmt.Printf("After append: %v (no longer nil: %t)\n", nilSlice, nilSlice != nil)
    
    // Clone of nil slice
    var anotherNil []string
    cloned := slices.Clone(anotherNil)
    fmt.Printf("Cloned nil slice: %v, is nil: %t\n", cloned, cloned == nil)
    
    // Empty slice vs nil slice
    emptySlice := []int{}
    fmt.Printf("Empty slice: %v, is nil: %t\n", emptySlice, emptySlice == nil)
    fmt.Printf("Equal to nil slice: %t\n", slices.Equal(emptySlice, nilSlice))
}
```

Nil slices are valid in Go and most operations handle them gracefully. They  
have zero length and capacity but aren't the same as empty slices. Understanding  
the distinction is important for proper error handling and memory management.  

## Memory considerations

Understanding slice memory usage and optimization strategies.  

```go
package main

import (
    "fmt"
    "slices"
    "runtime"
)

func main() {

    // Demonstrate memory sharing
    bigArray := make([]int, 1000000)
    for i := range bigArray {
        bigArray[i] = i
    }
    
    // Small slice referencing big array
    smallSlice := bigArray[999995:999999]
    fmt.Printf("Small slice: %v\n", smallSlice)
    fmt.Printf("Small slice length: %d, capacity: %d\n", len(smallSlice), cap(smallSlice))
    
    // The entire array is kept in memory!
    runtime.GC()
    runtime.GC()
    
    // Create independent small slice
    independent := slices.Clone(smallSlice)
    fmt.Printf("Independent slice: %v\n", independent)
    fmt.Printf("Independent length: %d, capacity: %d\n", len(independent), cap(independent))
    
    // Now bigArray can be garbage collected
    bigArray = nil
    smallSlice = nil
    runtime.GC()
    
    // Demonstrate capacity growth
    s := make([]int, 0, 1)
    fmt.Println("\nCapacity growth pattern:")
    for i := 0; i < 10; i++ {
        oldCap := cap(s)
        s = append(s, i)
        if cap(s) != oldCap {
            fmt.Printf("Capacity changed from %d to %d\n", oldCap, cap(s))
        }
    }
}
```

Slices can hold references to large underlying arrays, preventing garbage  
collection. Cloning creates independent storage, breaking the reference and  
allowing memory reclamation. Understanding capacity growth helps optimize  
memory allocation patterns.  

## Performance patterns

Optimizing slice operations for better performance in critical code paths.  

```go
package main

import (
    "fmt"
    "slices"
    "time"
)

func main() {

    const size = 100000
    
    // Pre-allocate vs growing
    start := time.Now()
    var growingSlice []int
    for i := 0; i < size; i++ {
        growingSlice = append(growingSlice, i)
    }
    growTime := time.Since(start)
    
    start = time.Now()
    preallocated := make([]int, 0, size)
    for i := 0; i < size; i++ {
        preallocated = append(preallocated, i)
    }
    preallocTime := time.Since(start)
    
    fmt.Printf("Growing slice time: %v\n", growTime)
    fmt.Printf("Preallocated time: %v\n", preallocTime)
    fmt.Printf("Preallocated is %.2fx faster\n", float64(growTime)/float64(preallocTime))
    
    // Efficient filtering
    numbers := make([]int, size)
    for i := range numbers {
        numbers[i] = i
    }
    
    // Inefficient: creating new slice
    start = time.Now()
    var evens1 []int
    for _, num := range numbers {
        if num%2 == 0 {
            evens1 = append(evens1, num)
        }
    }
    inefficientTime := time.Since(start)
    
    // Efficient: preallocated slice
    start = time.Now()
    evens2 := make([]int, 0, size/2)
    for _, num := range numbers {
        if num%2 == 0 {
            evens2 = append(evens2, num)
        }
    }
    efficientTime := time.Since(start)
    
    fmt.Printf("\nFiltering - inefficient: %v\n", inefficientTime)
    fmt.Printf("Filtering - efficient: %v\n", efficientTime)
}
```

Pre-allocating slices with known or estimated sizes significantly improves  
performance by avoiding multiple reallocations. Understanding these patterns  
helps write efficient Go code for performance-critical applications.  

## Filtering slices

Creating new slices containing only elements that meet specific criteria.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    fmt.Println("Original numbers:", numbers)
    
    // Filter even numbers
    evens := []int{}
    for _, num := range numbers {
        if num%2 == 0 {
            evens = append(evens, num)
        }
    }
    fmt.Println("Even numbers:", evens)
    
    // Filter using DeleteFunc (removes elements that match)
    odds := slices.Clone(numbers)
    odds = slices.DeleteFunc(odds, func(n int) bool {
        return n%2 == 0 // Remove even numbers
    })
    fmt.Println("Odd numbers:", odds)
    
    // Filter strings by length
    words := []string{"go", "programming", "is", "awesome", "and", "fun"}
    
    longWords := []string{}
    for _, word := range words {
        if len(word) > 3 {
            longWords = append(longWords, word)
        }
    }
    fmt.Println("Long words (>3 chars):", longWords)
    
    // Remove short words using DeleteFunc
    filtered := slices.Clone(words)
    filtered = slices.DeleteFunc(filtered, func(s string) bool {
        return len(s) <= 3
    })
    fmt.Println("Filtered words:", filtered)
}
```

Filtering creates new slices with elements meeting specific criteria.  
`slices.DeleteFunc()` provides an efficient way to remove unwanted elements  
based on custom predicates, which is equivalent to keeping desired elements.  

## Slice concatenation

Joining multiple slices into a single slice using various approaches.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    slice1 := []int{1, 2, 3}
    slice2 := []int{4, 5, 6}
    slice3 := []int{7, 8, 9}
    
    fmt.Println("Slice1:", slice1)
    fmt.Println("Slice2:", slice2)
    fmt.Println("Slice3:", slice3)
    
    // Concatenate using append
    combined := append(slice1, slice2...)
    combined = append(combined, slice3...)
    fmt.Println("Combined using append:", combined)
    
    // Concatenate using Concat (Go 1.22+)
    concatenated := slices.Concat(slice1, slice2, slice3)
    fmt.Println("Combined using Concat:", concatenated)
    
    // String slice concatenation
    fruits := []string{"apple", "banana"}
    vegetables := []string{"carrot", "broccoli"}
    grains := []string{"rice", "wheat"}
    
    food := slices.Concat(fruits, vegetables, grains)
    fmt.Println("All food:", food)
    
    // Build slice progressively
    var result []string
    sources := [][]string{fruits, vegetables, grains}
    
    for _, source := range sources {
        result = append(result, source...)
    }
    fmt.Println("Progressive build:", result)
}
```

Slice concatenation joins multiple slices into one. The `append()` function  
works for two slices, while `slices.Concat()` efficiently handles multiple  
slices in a single operation without intermediate allocations.  

## Unique elements

Removing duplicate elements from slices to create unique collections.  

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    numbers := []int{1, 2, 2, 3, 3, 3, 4, 4, 5}
    fmt.Println("Original numbers:", numbers)
    
    // Sort first, then compact for unique elements
    unique := slices.Clone(numbers)
    slices.Sort(unique)
    unique = slices.Compact(unique)
    fmt.Println("Unique numbers:", unique)
    
    // Manual unique using map
    seen := make(map[int]bool)
    manual := []int{}
    
    for _, num := range numbers {
        if !seen[num] {
            seen[num] = true
            manual = append(manual, num)
        }
    }
    fmt.Println("Manual unique (preserves order):", manual)
    
    // Unique strings
    words := []string{"apple", "banana", "apple", "cherry", "banana", "date"}
    fmt.Println("Original words:", words)
    
    wordSet := make(map[string]bool)
    uniqueWords := []string{}
    
    for _, word := range words {
        if !wordSet[word] {
            wordSet[word] = true
            uniqueWords = append(uniqueWords, word)
        }
    }
    fmt.Println("Unique words:", uniqueWords)
    
    // Using sort + compact for strings
    sortedWords := slices.Clone(words)
    slices.Sort(sortedWords)
    uniqueSorted := slices.Compact(sortedWords)
    fmt.Println("Unique sorted words:", uniqueSorted)
}
```

Creating unique slices removes duplicate elements. The sort-and-compact approach  
is efficient but changes order, while the map-based approach preserves original  
order but uses more memory. Choose based on your requirements.  

## Shuffling slices

Randomizing slice element order using different shuffling algorithms.  

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {

    // Initialize random seed
    rand.Seed(time.Now().UnixNano())
    
    cards := []string{"A", "K", "Q", "J", "10", "9", "8", "7", "6", "5", "4", "3", "2"}
    fmt.Println("Original cards:", cards)
    
    // Fisher-Yates shuffle
    shuffled := make([]string, len(cards))
    copy(shuffled, cards)
    
    for i := len(shuffled) - 1; i > 0; i-- {
        j := rand.Intn(i + 1)
        shuffled[i], shuffled[j] = shuffled[j], shuffled[i]
    }
    fmt.Println("Shuffled cards:", shuffled)
    
    // Shuffle numbers
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    fmt.Println("Original numbers:", numbers)
    
    // Simple shuffle
    for i := range numbers {
        j := rand.Intn(len(numbers))
        numbers[i], numbers[j] = numbers[j], numbers[i]
    }
    fmt.Println("Shuffled numbers:", numbers)
    
    // Multiple shuffles
    colors := []string{"red", "green", "blue", "yellow", "purple"}
    fmt.Println("Original colors:", colors)
    
    for round := 1; round <= 3; round++ {
        // Create copy for each shuffle
        shuffledColors := make([]string, len(colors))
        copy(shuffledColors, colors)
        
        // Fisher-Yates shuffle
        for i := len(shuffledColors) - 1; i > 0; i-- {
            j := rand.Intn(i + 1)
            shuffledColors[i], shuffledColors[j] = shuffledColors[j], shuffledColors[i]
        }
        fmt.Printf("Shuffle %d: %v\n", round, shuffledColors)
    }
}
```

The Fisher-Yates shuffle algorithm ensures each permutation has equal probability.  
Shuffling is useful for randomizing data, creating test scenarios, or implementing  
games and simulations where random order is required.  

## Partitioning slices

Dividing slices into multiple groups based on specific criteria.  

```go
package main

import (
    "fmt"
)

func main() {

    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12}
    fmt.Println("Original numbers:", numbers)
    
    // Partition into even and odd
    evens := []int{}
    odds := []int{}
    
    for _, num := range numbers {
        if num%2 == 0 {
            evens = append(evens, num)
        } else {
            odds = append(odds, num)
        }
    }
    fmt.Println("Even numbers:", evens)
    fmt.Println("Odd numbers:", odds)
    
    // Partition strings by length
    words := []string{"cat", "elephant", "dog", "hippopotamus", "ant", "bird"}
    fmt.Println("Original words:", words)
    
    short := []string{}   // <= 3 characters
    medium := []string{}  // 4-6 characters  
    long := []string{}    // > 6 characters
    
    for _, word := range words {
        switch {
        case len(word) <= 3:
            short = append(short, word)
        case len(word) <= 6:
            medium = append(medium, word)
        default:
            long = append(long, word)
        }
    }
    
    fmt.Println("Short words:", short)
    fmt.Println("Medium words:", medium)
    fmt.Println("Long words:", long)
    
    // Partition numbers into ranges
    scores := []int{45, 78, 92, 56, 89, 34, 67, 83, 91, 72}
    fmt.Println("Scores:", scores)
    
    failing := []int{}    // < 60
    passing := []int{}    // 60-79
    excellent := []int{}  // >= 80
    
    for _, score := range scores {
        switch {
        case score < 60:
            failing = append(failing, score)
        case score < 80:
            passing = append(passing, score)
        default:
            excellent = append(excellent, score)
        }
    }
    
    fmt.Printf("Failing (<%d): %v\n", 60, failing)
    fmt.Printf("Passing (60-79): %v\n", passing)
    fmt.Printf("Excellent (>=80): %v\n", excellent)
}
```

Partitioning divides slices into multiple groups based on conditions. This  
pattern is useful for categorizing data, implementing bucket sort algorithms,  
or organizing information for further processing based on different criteria.  

## Slice transformation

Converting slice elements to different types or applying functions to all elements.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
)

func main() {

    // Transform numbers to strings
    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println("Original numbers:", numbers)
    
    strings_from_nums := make([]string, len(numbers))
    for i, num := range numbers {
        strings_from_nums[i] = strconv.Itoa(num)
    }
    fmt.Println("Numbers as strings:", strings_from_nums)
    
    // Transform strings to uppercase
    words := []string{"hello", "world", "golang", "programming"}
    fmt.Println("Original words:", words)
    
    uppercase := make([]string, len(words))
    for i, word := range words {
        uppercase[i] = strings.ToUpper(word)
    }
    fmt.Println("Uppercase words:", uppercase)
    
    // Mathematical transformations
    values := []float64{1.0, 2.0, 3.0, 4.0, 5.0}
    fmt.Println("Original values:", values)
    
    // Square each value
    squared := make([]float64, len(values))
    for i, val := range values {
        squared[i] = val * val
    }
    fmt.Println("Squared values:", squared)
    
    // Parse strings to integers
    stringNumbers := []string{"10", "20", "30", "40", "50"}
    fmt.Println("String numbers:", stringNumbers)
    
    parsed := []int{}
    for _, str := range stringNumbers {
        if num, err := strconv.Atoi(str); err == nil {
            parsed = append(parsed, num)
        } else {
            fmt.Printf("Error parsing %s: %v\n", str, err)
        }
    }
    fmt.Println("Parsed integers:", parsed)
    
    // Complex transformation: words to lengths
    sentences := []string{"Go is fast", "Python is readable", "Rust is safe"}
    fmt.Println("Sentences:", sentences)
    
    wordCounts := make([]int, len(sentences))
    for i, sentence := range sentences {
        wordCounts[i] = len(strings.Fields(sentence))
    }
    fmt.Println("Word counts:", wordCounts)
}
```

Slice transformation applies operations to all elements, creating new slices  
with converted values. This functional programming pattern is essential for  
data processing, format conversion, and applying business logic to collections.
