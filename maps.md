# Maps

## Access/modify/delete

```go
package main

import "fmt"

func main() {
    // Declare and initialize a map
    capitals := map[string]string{
        "France":   "Paris",
        "Japan":    "Tokyo",
        "Slovakia": "Bratislava",
    }

    // Access elements
    fmt.Println("The capital of France is:", capitals["France"])

    // Add a new key-value pair
    capitals["Germany"] = "Berlin"

    // Modify an existing value
    capitals["Japan"] = "Kyoto"

    // Delete a key-value pair
    delete(capitals, "Slovakia")

    // Iterate over the map
    for country, capital := range capitals {
        fmt.Printf("The capital of %s is %s\n", country, capital)
    }
}
```


## All/Keys/Values/Collect

```go
package main

import (
    "fmt"
    "maps"
)

func main() {

    countries := map[string]string{
        "sk": "Slovakia",
        "ru": "Russia",
        "de": "Germany",
        "no": "Norway",
    }

    for k := range maps.Keys(countries) {

        fmt.Println(k, ": ", countries[k])
    }

    for v := range maps.Values(countries) {

        fmt.Println(v)
    }

    fmt.Println("------------------------")

    m1 := map[string]string{"po": "Poland"}
    m2 := map[string]string{"ro": "Romania"}

    it := maps.All(countries)
    maps.Insert(m1, it)
    maps.Insert(m2, it)

    countries2 := maps.Collect(it)

    for k, v := range maps.All(countries2) {
        fmt.Println(k, v)
    }
}
```
