# Maps 

In Go (Golang), a map is a data structure that associates keys with values, allowing for efficient  
lookups, insertions, and deletions. Maps are defined using the `map` keyword followed by the types  
of the keys and values. For instance, `map[string]int` is a map with string keys and integer values.  
They are often used for tasks such as counting occurrences of items, grouping data, or implementing  
lookup tables. Maps in Go are implemented using hash tables, which provide average-case constant time  
complexity for these operations.

To create and initialize a map, you can use the `make` function or map literals. The `make` function  
is useful when you want to create an empty map and add entries dynamically, like `myMap := make(map[string]int)`.   
Map literals allow you to define a map with initial key-value pairs, like `myMap := map[string]int{"one": 1, "two": 2, "three": 3}`.   

Maps are extremely flexible and powerful, but it's important to handle them carefully since accessing  
or modifying elements in a nil map will cause a runtime panic. The `delete` function can be used to  
remove key-value pairs, and you can check for the presence of a key using the  
comma-ok idiom: `value, ok := myMap["key"]`. This powerful combination of features makes maps an essential  
tool in Go programming.


## Empty map

```go
package main

import "fmt"

func main() {
    fruits := map[string]int{}

    fruits["apple"] = 5
    fruits["banana"] = 10

    fmt.Println(fruits)

    capitals := make(map[string]string)
    capitals["Slovakia"] = "Bratislava"
    capitals["Germany"] = "Berlin"

    fmt.Println(capitals)
}
```

## Loops 

```go
package main

import "fmt"

func main() {

    countries := map[string]string{
        "sk": "Slovakia",
        "ru": "Russia",
        "de": "Germany",
        "no": "Norway",
    }

    for country := range countries {
        fmt.Println(country, "=>", countries[country])
    }

    for key, value := range countries {
        fmt.Printf("countries[%s] = %s\n", key, value)
    }
}
```

## Map size

```go
package main

import "fmt"

func main() {

    countries := map[string]string{
        "sk": "Slovakia",
        "ru": "Russia",
        "de": "Germany",
        "no": "Norway",
    }

    fmt.Printf("There are %d pairs in the map\n", len(countries))
}
```


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

## Check if exists

```go
package main

import "fmt"

func main() {
    ages := map[string]int{
        "Alice": 30,
        "Bob":   25,
        "Eve":   35,
    }

    name := "Alice"
    age, exists := ages[name]
    if exists {
        fmt.Printf("%s's age is %d\n", name, age)
    } else {
        fmt.Printf("%s's age is not available\n", name)
    }
}
```

## Map of structs
 
```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    people := map[int]Person{
        1: {"Alice", 30},
        2: {"Bob", 25},
    }
    
    // Accessing struct fields
    fmt.Println("Alice's age:", people[1].Age)
    
    // Adding a new person
    people[3] = Person{"Eve", 35}
    
    // Iterating over the map
    for key, person := range people {
        fmt.Printf("%d: %s is %d years old\n", key, person.Name, person.Age)
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


## Nested maps

```go
package main

import "fmt"

func main() {
    students := map[string]map[string]int{
        "Alice": {
            "Math":    90,
            "Science": 85,
        },
        "Bob": {
            "Math":    80,
            "Science": 70,
        },
    }

    // Accessing nested maps
    fmt.Println("Alice's Math score:", students["Alice"]["Math"])

    // Adding a new subject for a student
    students["Alice"]["History"] = 75

    // Iterating over nested maps
    for student, subjects := range students {
        fmt.Printf("%s's scores:\n", student)
        for subject, score := range subjects {
            fmt.Printf("  %s: %d\n", subject, score)
        }
    }
}
```



