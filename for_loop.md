# For loop

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


## C style

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

## Single condition

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

## Range clause

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

## Using index

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

## Infinite loop

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
