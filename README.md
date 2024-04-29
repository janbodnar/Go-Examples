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
