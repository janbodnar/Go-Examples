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
