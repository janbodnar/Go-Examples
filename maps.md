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



