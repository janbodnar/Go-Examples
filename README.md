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
