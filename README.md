# Go examples



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
