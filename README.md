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



