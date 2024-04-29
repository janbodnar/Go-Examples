# Go examples


## For loop

### C style

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

### Single condition

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

### Range clause

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

### Using index

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
