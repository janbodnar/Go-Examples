# Slices

```go
package main

import (
    "fmt"
    "slices"
)

func main() {

    vals := []int{1, 2, 3, 4, 5, 6, 7, 8}

    for val := range slices.Backward(vals) {
        fmt.Println(val)
    }

    fmt.Println("---------------------------")

    for val := range slices.Chunk(vals, 2) {
        fmt.Println(val)
    }

    fmt.Println("---------------------------")

    words := []string{"sky", "ten", "water", "forest", "cup"}
    for i, v := range slices.All(words) {
        fmt.Println(i, ":", v)
    }

    fmt.Println("---------------------------")

    it := slices.Values(words)
    sorted_words := slices.Sorted(it)

    for _, word := range sorted_words {
        fmt.Println(word)
    }
}
```
