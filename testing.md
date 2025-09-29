# Unit Testing with Go Testing Package

Testing is a fundamental aspect of software development that ensures code  
reliability, correctness, and maintainability. Go provides a robust built-in  
testing framework through the `testing` package, making it straightforward  
to write comprehensive test suites for applications. The Go testing philosophy  
emphasizes simplicity, clarity, and efficiency, encouraging developers to  
write tests that are easy to understand and maintain.  

The Go testing package supports multiple testing paradigms including unit  
tests, benchmark tests, example tests, and fuzz tests. Unit tests verify  
that individual functions or methods behave correctly under various  
conditions. Benchmark tests measure performance characteristics and help  
identify optimization opportunities. Example tests serve as executable  
documentation, demonstrating how to use functions and packages. Fuzz tests,  
introduced in Go 1.18, automatically generate test inputs to discover edge  
cases and potential vulnerabilities.  

Go's approach to testing differs from many other languages by integrating  
testing tools directly into the language toolchain. The `go test` command  
automatically discovers and runs tests, provides coverage analysis, and  
supports parallel test execution. Test files follow a simple naming  
convention with `_test.go` suffix, and test functions must start with  
`Test`, `Benchmark`, or `Example` prefixes.  

The testing package provides a minimal but powerful API centered around  
the `testing.T` type for regular tests and `testing.B` for benchmarks.  
These types offer methods for reporting test failures, logging information,  
and controlling test execution. The framework encourages explicit error  
checking and clear failure messages, making it easy to diagnose issues  
when tests fail.  

One of Go's testing strengths is its support for table-driven tests, a  
pattern that allows testing multiple scenarios with a single test function.  
This approach reduces code duplication while improving test coverage and  
readability. The framework also supports test helpers, setup and teardown  
functions, and parallel test execution for improved performance.  

Go testing integrates seamlessly with the module system, coverage tools,  
and continuous integration pipelines. The `go test` command can generate  
coverage reports, run tests in verbose mode for detailed output, and  
execute specific test patterns or packages. This integration makes it  
easy to incorporate testing into development workflows and maintain  
high code quality standards.  

The testing package also supports advanced features like subtests for  
organizing related test cases, test main functions for custom setup  
and teardown, and testing utilities for common operations. These features  
enable sophisticated testing strategies while maintaining the simplicity  
and clarity that characterize Go's design philosophy.  

## Basic test function

The foundation of Go testing is the basic test function that verifies  
individual function behavior under specific conditions.  

```go
package main

import (
    "testing"
)

// Add returns the sum of two integers
func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(2, 3) = %d, expected %d", result, expected)
    }
}

func TestAddZero(t *testing.T) {
    result := Add(5, 0)
    expected := 5
    
    if result != expected {
        t.Errorf("Add(5, 0) = %d, expected %d", result, expected)
    }
}

func TestAddNegative(t *testing.T) {
    result := Add(-3, -7)
    expected := -10
    
    if result != expected {
        t.Errorf("Add(-3, -7) = %d, expected %d", result, expected)
    }
}
```

Basic test functions establish the fundamental pattern for Go testing.  
Each test function takes a `*testing.T` parameter and uses its methods  
to report failures. The `Errorf` method provides formatted error messages  
that help identify what went wrong and why the test failed.  

## String comparison testing

String operations require careful testing to handle various edge cases  
including empty strings, special characters, and unicode content.  

```go
package main

import (
    "strings"
    "testing"
)

// Greet returns a greeting message
func Greet(name string) string {
    if name == "" {
        return "Hello there, stranger!"
    }
    return "Hello there, " + name + "!"
}

// Contains checks if a string contains a substring
func Contains(text, substr string) bool {
    return strings.Contains(text, substr)
}

func TestGreet(t *testing.T) {
    result := Greet("Alice")
    expected := "Hello there, Alice!"
    
    if result != expected {
        t.Errorf("Greet(\"Alice\") = %q, expected %q", result, expected)
    }
}

func TestGreetEmpty(t *testing.T) {
    result := Greet("")
    expected := "Hello there, stranger!"
    
    if result != expected {
        t.Errorf("Greet(\"\") = %q, expected %q", result, expected)
    }
}

func TestContains(t *testing.T) {
    if !Contains("hello there", "there") {
        t.Error("Expected 'hello there' to contain 'there'")
    }
    
    if Contains("hello", "world") {
        t.Error("Expected 'hello' to not contain 'world'")
    }
}
```

String testing demonstrates proper handling of empty inputs and the use  
of quoted string formatting in error messages. The `%q` verb provides  
clear visualization of string content including quotes and escape sequences.  

## Error handling tests

Testing error conditions ensures functions properly handle invalid inputs  
and exceptional cases, maintaining robustness and predictable behavior.  

```go
package main

import (
    "errors"
    "testing"
)

// Divide performs division with error handling
func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// ParseAge converts string to age with validation
func ParseAge(s string) (int, error) {
    if s == "" {
        return 0, errors.New("age cannot be empty")
    }
    
    var age int
    for _, char := range s {
        if char < '0' || char > '9' {
            return 0, errors.New("age must be numeric")
        }
        age = age*10 + int(char-'0')
    }
    
    if age < 0 || age > 150 {
        return 0, errors.New("age must be between 0 and 150")
    }
    
    return age, nil
}

func TestDivideSuccess(t *testing.T) {
    result, err := Divide(10, 2)
    if err != nil {
        t.Fatalf("Unexpected error: %v", err)
    }
    
    expected := 5.0
    if result != expected {
        t.Errorf("Divide(10, 2) = %f, expected %f", result, expected)
    }
}

func TestDivideByZero(t *testing.T) {
    _, err := Divide(10, 0)
    if err == nil {
        t.Fatal("Expected error for division by zero")
    }
    
    expectedMsg := "division by zero"
    if err.Error() != expectedMsg {
        t.Errorf("Expected error message %q, got %q", expectedMsg, err.Error())
    }
}

func TestParseAge(t *testing.T) {
    age, err := ParseAge("25")
    if err != nil {
        t.Fatalf("Unexpected error: %v", err)
    }
    
    if age != 25 {
        t.Errorf("ParseAge(\"25\") = %d, expected 25", age)
    }
}

func TestParseAgeInvalid(t *testing.T) {
    _, err := ParseAge("abc")
    if err == nil {
        t.Error("Expected error for non-numeric age")
    }
    
    _, err = ParseAge("")
    if err == nil {
        t.Error("Expected error for empty age")
    }
    
    _, err = ParseAge("200")
    if err == nil {
        t.Error("Expected error for age over 150")
    }
}
```

Error testing validates both success and failure paths, ensuring functions  
handle edge cases appropriately. The `Fatal` methods stop test execution  
immediately when critical conditions aren't met, preventing cascading  
failures in subsequent assertions.  

## Slice operations testing

Slice manipulation requires testing various scenarios including empty  
slices, boundary conditions, and data integrity preservation.  

```go
package main

import (
    "reflect"
    "testing"
)

// Filter returns elements that satisfy the predicate function
func Filter(slice []int, predicate func(int) bool) []int {
    var result []int
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Reverse returns a new slice with elements in reverse order
func Reverse(slice []string) []string {
    result := make([]string, len(slice))
    for i, v := range slice {
        result[len(slice)-1-i] = v
    }
    return result
}

// Sum calculates the total of all integers in a slice
func Sum(numbers []int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

func TestFilter(t *testing.T) {
    numbers := []int{1, 2, 3, 4, 5, 6}
    even := Filter(numbers, func(n int) bool { return n%2 == 0 })
    expected := []int{2, 4, 6}
    
    if !reflect.DeepEqual(even, expected) {
        t.Errorf("Filter even numbers = %v, expected %v", even, expected)
    }
}

func TestFilterEmpty(t *testing.T) {
    empty := []int{}
    result := Filter(empty, func(n int) bool { return n > 0 })
    
    if len(result) != 0 {
        t.Errorf("Filter empty slice = %v, expected empty slice", result)
    }
}

func TestReverse(t *testing.T) {
    words := []string{"hello", "there", "friend"}
    reversed := Reverse(words)
    expected := []string{"friend", "there", "hello"}
    
    if !reflect.DeepEqual(reversed, expected) {
        t.Errorf("Reverse = %v, expected %v", reversed, expected)
    }
}

func TestReverseEmpty(t *testing.T) {
    empty := []string{}
    result := Reverse(empty)
    
    if len(result) != 0 {
        t.Errorf("Reverse empty slice = %v, expected empty slice", result)
    }
}

func TestSum(t *testing.T) {
    numbers := []int{1, 2, 3, 4, 5}
    result := Sum(numbers)
    expected := 15
    
    if result != expected {
        t.Errorf("Sum = %d, expected %d", result, expected)
    }
}

func TestSumEmpty(t *testing.T) {
    empty := []int{}
    result := Sum(empty)
    
    if result != 0 {
        t.Errorf("Sum empty slice = %d, expected 0", result)
    }
}
```

Slice testing demonstrates the use of `reflect.DeepEqual` for comparing  
complex data structures and the importance of testing empty input scenarios.  
Proper slice testing ensures functions handle boundary conditions correctly  
and maintain data integrity during operations.  

## Map operations testing

Map-based functions require testing key existence, value retrieval,  
and proper handling of missing keys and empty maps.  

```go
package main

import (
    "testing"
)

// UserStore manages user data
type UserStore struct {
    users map[string]string
}

// NewUserStore creates a new user store
func NewUserStore() *UserStore {
    return &UserStore{
        users: make(map[string]string),
    }
}

// Add adds a user to the store
func (us *UserStore) Add(username, email string) {
    us.users[username] = email
}

// Get retrieves a user's email
func (us *UserStore) Get(username string) (string, bool) {
    email, exists := us.users[username]
    return email, exists
}

// Count returns the number of users
func (us *UserStore) Count() int {
    return len(us.users)
}

// Delete removes a user from the store
func (us *UserStore) Delete(username string) bool {
    if _, exists := us.users[username]; exists {
        delete(us.users, username)
        return true
    }
    return false
}

func TestUserStoreAdd(t *testing.T) {
    store := NewUserStore()
    store.Add("alice", "alice@example.com")
    
    email, exists := store.Get("alice")
    if !exists {
        t.Error("Expected user 'alice' to exist")
    }
    
    expected := "alice@example.com"
    if email != expected {
        t.Errorf("Get('alice') = %q, expected %q", email, expected)
    }
}

func TestUserStoreGetNonExistent(t *testing.T) {
    store := NewUserStore()
    
    _, exists := store.Get("nonexistent")
    if exists {
        t.Error("Expected user 'nonexistent' to not exist")
    }
}

func TestUserStoreCount(t *testing.T) {
    store := NewUserStore()
    
    if store.Count() != 0 {
        t.Errorf("New store count = %d, expected 0", store.Count())
    }
    
    store.Add("alice", "alice@example.com")
    store.Add("bob", "bob@example.com")
    
    if store.Count() != 2 {
        t.Errorf("Store count = %d, expected 2", store.Count())
    }
}

func TestUserStoreDelete(t *testing.T) {
    store := NewUserStore()
    store.Add("alice", "alice@example.com")
    
    deleted := store.Delete("alice")
    if !deleted {
        t.Error("Expected deletion to succeed")
    }
    
    _, exists := store.Get("alice")
    if exists {
        t.Error("Expected user 'alice' to be deleted")
    }
    
    deletedAgain := store.Delete("alice")
    if deletedAgain {
        t.Error("Expected second deletion to fail")
    }
}
```

Map testing validates data storage, retrieval, and manipulation operations.  
Testing both success and failure cases ensures robust handling of missing  
keys and proper state management across operations.  

## Table-driven test basics

Table-driven tests provide an efficient way to test multiple input scenarios  
with a single test function, reducing code duplication and improving coverage.  

```go
package main

import (
    "testing"
)

// IsEven checks if a number is even
func IsEven(n int) bool {
    return n%2 == 0
}

// Factorial calculates the factorial of a number
func Factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * Factorial(n-1)
}

func TestIsEven(t *testing.T) {
    tests := []struct {
        name     string
        input    int
        expected bool
    }{
        {"Zero is even", 0, true},
        {"Positive even", 4, true},
        {"Positive odd", 5, false},
        {"Negative even", -6, true},
        {"Negative odd", -7, false},
        {"One is odd", 1, false},
        {"Two is even", 2, true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := IsEven(tt.input)
            if result != tt.expected {
                t.Errorf("IsEven(%d) = %t, expected %t", tt.input, result, tt.expected)
            }
        })
    }
}

func TestFactorial(t *testing.T) {
    tests := []struct {
        name     string
        input    int
        expected int
    }{
        {"Factorial of 0", 0, 1},
        {"Factorial of 1", 1, 1},
        {"Factorial of 3", 3, 6},
        {"Factorial of 5", 5, 120},
        {"Factorial of 7", 7, 5040},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Factorial(tt.input)
            if result != tt.expected {
                t.Errorf("Factorial(%d) = %d, expected %d", tt.input, result, tt.expected)
            }
        })
    }
}
```

Table-driven tests organize test cases in structured data, making it easy  
to add new scenarios and maintain comprehensive test coverage. The `t.Run`  
method creates subtests with descriptive names for better error reporting.  

## Advanced table-driven tests

Complex table-driven tests can handle multiple outputs, error conditions,  
and sophisticated validation scenarios with structured test data.  

```go
package main

import (
    "fmt"
    "strings"
    "testing"
)

// ValidateEmail performs basic email validation
func ValidateEmail(email string) (bool, string) {
    if email == "" {
        return false, "email cannot be empty"
    }
    
    if !strings.Contains(email, "@") {
        return false, "email must contain @ symbol"
    }
    
    parts := strings.Split(email, "@")
    if len(parts) != 2 {
        return false, "email must have exactly one @ symbol"
    }
    
    if parts[0] == "" {
        return false, "email local part cannot be empty"
    }
    
    if parts[1] == "" {
        return false, "email domain cannot be empty"
    }
    
    if !strings.Contains(parts[1], ".") {
        return false, "domain must contain at least one dot"
    }
    
    return true, ""
}

// FormatName standardizes name formatting
func FormatName(first, last string) (string, error) {
    first = strings.TrimSpace(first)
    last = strings.TrimSpace(last)
    
    if first == "" {
        return "", fmt.Errorf("first name cannot be empty")
    }
    
    if last == "" {
        return "", fmt.Errorf("last name cannot be empty")
    }
    
    return strings.Title(strings.ToLower(first)) + " " + strings.Title(strings.ToLower(last)), nil
}

func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name          string
        email         string
        expectedValid bool
        expectedMsg   string
    }{
        {"Valid email", "user@example.com", true, ""},
        {"Empty email", "", false, "email cannot be empty"},
        {"No @ symbol", "userexample.com", false, "email must contain @ symbol"},
        {"Multiple @ symbols", "user@@example.com", false, "email must have exactly one @ symbol"},
        {"Empty local part", "@example.com", false, "email local part cannot be empty"},
        {"Empty domain", "user@", false, "email domain cannot be empty"},
        {"Domain without dot", "user@example", false, "domain must contain at least one dot"},
        {"Valid complex email", "user.name+tag@example.co.uk", true, ""},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            valid, msg := ValidateEmail(tt.email)
            
            if valid != tt.expectedValid {
                t.Errorf("ValidateEmail(%q) valid = %t, expected %t", tt.email, valid, tt.expectedValid)
            }
            
            if msg != tt.expectedMsg {
                t.Errorf("ValidateEmail(%q) message = %q, expected %q", tt.email, msg, tt.expectedMsg)
            }
        })
    }
}

func TestFormatName(t *testing.T) {
    tests := []struct {
        name        string
        first       string
        last        string
        expected    string
        expectError bool
    }{
        {"Normal names", "john", "doe", "John Doe", false},
        {"Mixed case", "ALICE", "smith", "Alice Smith", false},
        {"With spaces", " bob ", " jones ", "Bob Jones", false},
        {"Empty first", "", "doe", "", true},
        {"Empty last", "john", "", "", true},
        {"Both empty", "", "", "", true},
        {"Single letter names", "a", "b", "A B", false},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := FormatName(tt.first, tt.last)
            
            if tt.expectError {
                if err == nil {
                    t.Errorf("FormatName(%q, %q) expected error, got nil", tt.first, tt.last)
                }
            } else {
                if err != nil {
                    t.Errorf("FormatName(%q, %q) unexpected error: %v", tt.first, tt.last, err)
                }
                
                if result != tt.expected {
                    t.Errorf("FormatName(%q, %q) = %q, expected %q", tt.first, tt.last, result, tt.expected)
                }
            }
        })
    }
}
```

Advanced table-driven tests handle complex scenarios with multiple outputs  
and error conditions. The structured approach makes it easy to validate  
both successful operations and various failure modes comprehensively.  

## Test helper functions

Test helpers reduce code duplication and provide reusable testing utilities  
that improve test maintainability and consistency across test suites.  

```go
package main

import (
    "fmt"
    "math"
    "reflect"
    "testing"
)

// Calculator for testing helpers demonstration
type Calculator struct {
    precision int
}

// NewCalculator creates a calculator with specified precision
func NewCalculator(precision int) *Calculator {
    return &Calculator{precision: precision}
}

// Add performs addition
func (c *Calculator) Add(a, b float64) float64 {
    result := a + b
    return c.round(result)
}

// Multiply performs multiplication
func (c *Calculator) Multiply(a, b float64) float64 {
    result := a * b
    return c.round(result)
}

// Divide performs division
func (c *Calculator) Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    result := a / b
    return c.round(result), nil
}

func (c *Calculator) round(value float64) float64 {
    precision := math.Pow(10, float64(c.precision))
    return math.Round(value*precision) / precision
}

// Test helper functions
func assertFloatEqual(t *testing.T, got, want float64) {
    t.Helper()
    if math.Abs(got-want) > 1e-9 {
        t.Errorf("got %f, want %f", got, want)
    }
}

func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("unexpected error: %v", err)
    }
}

func assertError(t *testing.T, err error, expectedMsg string) {
    t.Helper()
    if err == nil {
        t.Fatalf("expected error %q, got nil", expectedMsg)
    }
    if err.Error() != expectedMsg {
        t.Errorf("expected error %q, got %q", expectedMsg, err.Error())
    }
}

func assertSliceEqual[T comparable](t *testing.T, got, want []T) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}

func assertMapEqual[K, V comparable](t *testing.T, got, want map[K]V) {
    t.Helper()
    if !reflect.DeepEqual(got, want) {
        t.Errorf("got %v, want %v", got, want)
    }
}

func TestCalculatorWithHelpers(t *testing.T) {
    calc := NewCalculator(2)
    
    result := calc.Add(0.1, 0.2)
    assertFloatEqual(t, result, 0.30)
    
    result = calc.Multiply(3.333, 2)
    assertFloatEqual(t, result, 6.67)
    
    result, err := calc.Divide(10, 3)
    assertNoError(t, err)
    assertFloatEqual(t, result, 3.33)
    
    _, err = calc.Divide(5, 0)
    assertError(t, err, "division by zero")
}

func TestSliceHelperExample(t *testing.T) {
    numbers := []int{1, 2, 3, 4, 5}
    expected := []int{1, 2, 3, 4, 5}
    
    assertSliceEqual(t, numbers, expected)
}

func TestMapHelperExample(t *testing.T) {
    scores := map[string]int{"alice": 95, "bob": 87}
    expected := map[string]int{"alice": 95, "bob": 87}
    
    assertMapEqual(t, scores, expected)
}
```

Test helpers encapsulate common assertion patterns and provide consistent  
error reporting across test suites. The `t.Helper()` call ensures error  
messages point to the calling test function rather than the helper itself.  

## Subtest organization

Subtests provide hierarchical organization of related test cases, enabling  
fine-grained control over test execution and clearer result reporting.  

```go
package main

import (
    "fmt"
    "reflect"
    "strings"
    "testing"
)

// StringProcessor provides various string operations
type StringProcessor struct {
    caseSensitive bool
}

// NewStringProcessor creates a new string processor
func NewStringProcessor(caseSensitive bool) *StringProcessor {
    return &StringProcessor{caseSensitive: caseSensitive}
}

// Clean removes whitespace and normalizes case
func (sp *StringProcessor) Clean(s string) string {
    cleaned := strings.TrimSpace(s)
    if !sp.caseSensitive {
        cleaned = strings.ToLower(cleaned)
    }
    return cleaned
}

// Split divides string by delimiter
func (sp *StringProcessor) Split(s, delimiter string) []string {
    if !sp.caseSensitive {
        s = strings.ToLower(s)
        delimiter = strings.ToLower(delimiter)
    }
    return strings.Split(s, delimiter)
}

// Contains checks if string contains substring
func (sp *StringProcessor) Contains(s, substr string) bool {
    if !sp.caseSensitive {
        s = strings.ToLower(s)
        substr = strings.ToLower(substr)
    }
    return strings.Contains(s, substr)
}

func TestStringProcessor(t *testing.T) {
    t.Run("CaseSensitive", func(t *testing.T) {
        processor := NewStringProcessor(true)
        
        t.Run("Clean", func(t *testing.T) {
            t.Run("RemovesWhitespace", func(t *testing.T) {
                result := processor.Clean("  hello there  ")
                expected := "hello there"
                if result != expected {
                    t.Errorf("Clean() = %q, expected %q", result, expected)
                }
            })
            
            t.Run("PreservesCase", func(t *testing.T) {
                result := processor.Clean("  Hello THERE  ")
                expected := "Hello THERE"
                if result != expected {
                    t.Errorf("Clean() = %q, expected %q", result, expected)
                }
            })
        })
        
        t.Run("Contains", func(t *testing.T) {
            t.Run("ExactMatch", func(t *testing.T) {
                if !processor.Contains("Hello There", "There") {
                    t.Error("Expected to find 'There' in 'Hello There'")
                }
            })
            
            t.Run("CaseMismatch", func(t *testing.T) {
                if processor.Contains("Hello There", "there") {
                    t.Error("Should not find 'there' in 'Hello There' (case sensitive)")
                }
            })
        })
    })
    
    t.Run("CaseInsensitive", func(t *testing.T) {
        processor := NewStringProcessor(false)
        
        t.Run("Clean", func(t *testing.T) {
            t.Run("NormalizesCase", func(t *testing.T) {
                result := processor.Clean("  Hello THERE  ")
                expected := "hello there"
                if result != expected {
                    t.Errorf("Clean() = %q, expected %q", result, expected)
                }
            })
        })
        
        t.Run("Contains", func(t *testing.T) {
            t.Run("CaseInsensitiveMatch", func(t *testing.T) {
                if !processor.Contains("Hello There", "there") {
                    t.Error("Expected to find 'there' in 'Hello There' (case insensitive)")
                }
            })
            
            t.Run("MixedCaseMatch", func(t *testing.T) {
                if !processor.Contains("HELLO there", "Hello") {
                    t.Error("Expected to find 'Hello' in 'HELLO there' (case insensitive)")
                }
            })
        })
        
        t.Run("Split", func(t *testing.T) {
            t.Run("CaseInsensitiveDelimiter", func(t *testing.T) {
                result := processor.Split("Hello,THERE,friend", ",")
                expected := []string{"hello", "there", "friend"}
                if !reflect.DeepEqual(result, expected) {
                    t.Errorf("Split() = %v, expected %v", result, expected)
                }
            })
        })
    })
}
```

Subtests create logical groupings of related test cases, making it easier  
to run specific test categories and understand test results. The hierarchical  
structure improves test organization and maintenance for complex scenarios.  

## Test setup and teardown

Setup and teardown functions manage test resources and ensure clean test  
environments for consistent and reliable test execution.  

```go
package main

import (
    "io/ioutil"
    "os"
    "path/filepath"
    "strings"
    "testing"
)

// FileManager handles file operations for testing
type FileManager struct {
    baseDir string
}

// NewFileManager creates a file manager with base directory
func NewFileManager(baseDir string) *FileManager {
    return &FileManager{baseDir: baseDir}
}

// CreateFile creates a file with content
func (fm *FileManager) CreateFile(name, content string) error {
    path := filepath.Join(fm.baseDir, name)
    return ioutil.WriteFile(path, []byte(content), 0644)
}

// ReadFile reads file content
func (fm *FileManager) ReadFile(name string) (string, error) {
    path := filepath.Join(fm.baseDir, name)
    content, err := ioutil.ReadFile(path)
    if err != nil {
        return "", err
    }
    return string(content), nil
}

// DeleteFile removes a file
func (fm *FileManager) DeleteFile(name string) error {
    path := filepath.Join(fm.baseDir, name)
    return os.Remove(path)
}

// ListFiles returns all files in the directory
func (fm *FileManager) ListFiles() ([]string, error) {
    files, err := ioutil.ReadDir(fm.baseDir)
    if err != nil {
        return nil, err
    }
    
    var names []string
    for _, file := range files {
        if !file.IsDir() {
            names = append(names, file.Name())
        }
    }
    return names, nil
}

func setupTestDir(t *testing.T) (string, func()) {
    t.Helper()
    
    tempDir, err := ioutil.TempDir("", "filemanager_test")
    if err != nil {
        t.Fatalf("Failed to create temp dir: %v", err)
    }
    
    cleanup := func() {
        if err := os.RemoveAll(tempDir); err != nil {
            t.Errorf("Failed to cleanup temp dir: %v", err)
        }
    }
    
    return tempDir, cleanup
}

func TestFileManager(t *testing.T) {
    tempDir, cleanup := setupTestDir(t)
    defer cleanup()
    
    fm := NewFileManager(tempDir)
    
    t.Run("CreateAndRead", func(t *testing.T) {
        content := "Hello there, file content!"
        err := fm.CreateFile("test.txt", content)
        if err != nil {
            t.Fatalf("Failed to create file: %v", err)
        }
        
        readContent, err := fm.ReadFile("test.txt")
        if err != nil {
            t.Fatalf("Failed to read file: %v", err)
        }
        
        if readContent != content {
            t.Errorf("File content = %q, expected %q", readContent, content)
        }
    })
    
    t.Run("ListFiles", func(t *testing.T) {
        // Create multiple files
        files := map[string]string{
            "file1.txt": "content1",
            "file2.txt": "content2",
            "file3.txt": "content3",
        }
        
        for name, content := range files {
            if err := fm.CreateFile(name, content); err != nil {
                t.Fatalf("Failed to create file %s: %v", name, err)
            }
        }
        
        listedFiles, err := fm.ListFiles()
        if err != nil {
            t.Fatalf("Failed to list files: %v", err)
        }
        
        if len(listedFiles) != len(files) {
            t.Errorf("Listed files count = %d, expected %d", len(listedFiles), len(files))
        }
    })
    
    t.Run("DeleteFile", func(t *testing.T) {
        // Create a file to delete
        err := fm.CreateFile("todelete.txt", "temporary content")
        if err != nil {
            t.Fatalf("Failed to create file: %v", err)
        }
        
        // Delete the file
        err = fm.DeleteFile("todelete.txt")
        if err != nil {
            t.Fatalf("Failed to delete file: %v", err)
        }
        
        // Verify file is gone
        _, err = fm.ReadFile("todelete.txt")
        if err == nil {
            t.Error("Expected error reading deleted file")
        }
    })
}

func TestFileManagerWithSubtestSetup(t *testing.T) {
    tempDir, cleanup := setupTestDir(t)
    defer cleanup()
    
    fm := NewFileManager(tempDir)
    
    setupFile := func(t *testing.T, name, content string) {
        t.Helper()
        if err := fm.CreateFile(name, content); err != nil {
            t.Fatalf("Failed to setup file %s: %v", name, err)
        }
    }
    
    t.Run("ProcessSpecialFile", func(t *testing.T) {
        setupFile(t, "special.txt", "special content here")
        
        content, err := fm.ReadFile("special.txt")
        if err != nil {
            t.Fatalf("Failed to read special file: %v", err)
        }
        
        expected := "special content here"
        if content != expected {
            t.Errorf("Special file content = %q, expected %q", content, expected)
        }
    })
    
    t.Run("ProcessConfigFile", func(t *testing.T) {
        setupFile(t, "config.txt", "config=value")
        
        content, err := fm.ReadFile("config.txt")
        if err != nil {
            t.Fatalf("Failed to read config file: %v", err)
        }
        
        if !strings.Contains(content, "config=") {
            t.Error("Config file should contain 'config='")
        }
    })
}
```

Setup and teardown functions ensure tests start with clean environments  
and properly clean up resources after execution. This pattern prevents  
test interference and resource leaks in comprehensive test suites.  

## Basic benchmark tests

Benchmark tests measure function performance and help identify optimization  
opportunities by providing quantitative performance metrics.  

```go
package main

import (
    "fmt"
    "sort"
    "testing"
)

// BubbleSort implements bubble sort algorithm
func BubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}

// QuickSort implements quicksort algorithm
func QuickSort(arr []int) {
    if len(arr) < 2 {
        return
    }
    
    pivot := partition(arr)
    QuickSort(arr[:pivot])
    QuickSort(arr[pivot+1:])
}

func partition(arr []int) int {
    pivot := arr[len(arr)-1]
    i := -1
    
    for j := 0; j < len(arr)-1; j++ {
        if arr[j] <= pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    
    arr[i+1], arr[len(arr)-1] = arr[len(arr)-1], arr[i+1]
    return i + 1
}

// GenerateTestData creates a slice of test data
func GenerateTestData(size int) []int {
    data := make([]int, size)
    for i := 0; i < size; i++ {
        data[i] = size - i // Reverse order for worst case
    }
    return data
}

func BenchmarkBubbleSort(b *testing.B) {
    data := GenerateTestData(100)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        testData := make([]int, len(data))
        copy(testData, data)
        BubbleSort(testData)
    }
}

func BenchmarkQuickSort(b *testing.B) {
    data := GenerateTestData(100)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        testData := make([]int, len(data))
        copy(testData, data)
        QuickSort(testData)
    }
}

func BenchmarkStandardSort(b *testing.B) {
    data := GenerateTestData(100)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        testData := make([]int, len(data))
        copy(testData, data)
        sort.Ints(testData)
    }
}

func BenchmarkSortingSizes(b *testing.B) {
    sizes := []int{10, 100, 1000, 10000}
    
    for _, size := range sizes {
        b.Run(fmt.Sprintf("BubbleSort-%d", size), func(b *testing.B) {
            data := GenerateTestData(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                testData := make([]int, len(data))
                copy(testData, data)
                BubbleSort(testData)
            }
        })
        
        b.Run(fmt.Sprintf("QuickSort-%d", size), func(b *testing.B) {
            data := GenerateTestData(size)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                testData := make([]int, len(data))
                copy(testData, data)
                QuickSort(testData)
            }
        })
    }
}
```

Benchmark tests provide performance metrics and enable comparison between  
different algorithms or implementations. The `b.ResetTimer()` call excludes  
setup time from measurements, ensuring accurate performance data.  

## Memory allocation benchmarks

Memory benchmarks track allocation patterns and help optimize memory usage  
by measuring allocations per operation and total allocated bytes.  

```go
package main

import (
    "fmt"
    "strings"
    "testing"
)

// StringBuilderConcat uses strings.Builder for concatenation
func StringBuilderConcat(words []string) string {
    var builder strings.Builder
    for _, word := range words {
        builder.WriteString(word)
        builder.WriteString(" ")
    }
    return strings.TrimSpace(builder.String())
}

// StringConcat uses string concatenation
func StringConcat(words []string) string {
    result := ""
    for _, word := range words {
        result += word + " "
    }
    return strings.TrimSpace(result)
}

// StringJoin uses strings.Join
func StringJoin(words []string) string {
    return strings.Join(words, " ")
}

// SliceAppend demonstrates slice growth patterns
func SliceAppend(n int) []int {
    var slice []int
    for i := 0; i < n; i++ {
        slice = append(slice, i)
    }
    return slice
}

// SlicePrealloc uses preallocated slice
func SlicePrealloc(n int) []int {
    slice := make([]int, 0, n)
    for i := 0; i < n; i++ {
        slice = append(slice, i)
    }
    return slice
}

func generateWords(count int) []string {
    words := make([]string, count)
    for i := 0; i < count; i++ {
        words[i] = fmt.Sprintf("word%d", i)
    }
    return words
}

func BenchmarkStringConcatenation(b *testing.B) {
    words := generateWords(100)
    
    b.Run("StringBuilder", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            StringBuilderConcat(words)
        }
    })
    
    b.Run("StringConcat", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            StringConcat(words)
        }
    })
    
    b.Run("StringJoin", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            StringJoin(words)
        }
    })
}

func BenchmarkSliceGrowth(b *testing.B) {
    const size = 1000
    
    b.Run("Append", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            SliceAppend(size)
        }
    })
    
    b.Run("Prealloc", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            SlicePrealloc(size)
        }
    })
}

func BenchmarkMapOperations(b *testing.B) {
    const size = 1000
    
    b.Run("StringKeys", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            m := make(map[string]int)
            for j := 0; j < size; j++ {
                key := fmt.Sprintf("key%d", j)
                m[key] = j
            }
        }
    })
    
    b.Run("IntKeys", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            m := make(map[int]int)
            for j := 0; j < size; j++ {
                m[j] = j
            }
        }
    })
    
    b.Run("PreallocatedMap", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            m := make(map[string]int, size)
            for j := 0; j < size; j++ {
                key := fmt.Sprintf("key%d", j)
                m[key] = j
            }
        }
    })
}
```

Memory benchmarks reveal allocation patterns and help identify opportunities  
for optimization. The `b.ReportAllocs()` call enables memory allocation  
tracking, providing insights into memory efficiency of different approaches.  

## Example tests as documentation

Example tests serve as executable documentation, demonstrating proper usage  
of functions and packages while ensuring examples remain accurate.  

```go
package main

import (
    "fmt"
    "strings"
    "sort"
)

// TextProcessor provides text manipulation utilities
type TextProcessor struct {
    delimiter string
}

// NewTextProcessor creates a text processor with delimiter
func NewTextProcessor(delimiter string) *TextProcessor {
    return &TextProcessor{delimiter: delimiter}
}

// Join combines strings with the configured delimiter
func (tp *TextProcessor) Join(words ...string) string {
    return strings.Join(words, tp.delimiter)
}

// Split divides text using the configured delimiter
func (tp *TextProcessor) Split(text string) []string {
    return strings.Split(text, tp.delimiter)
}

// WordCount counts words in text
func (tp *TextProcessor) WordCount(text string) int {
    if text == "" {
        return 0
    }
    words := tp.Split(text)
    count := 0
    for _, word := range words {
        if strings.TrimSpace(word) != "" {
            count++
        }
    }
    return count
}

// SortWords sorts words alphabetically
func SortWords(words []string) []string {
    sorted := make([]string, len(words))
    copy(sorted, words)
    sort.Strings(sorted)
    return sorted
}

// ExampleTextProcessor_Join demonstrates basic text joining
func ExampleTextProcessor_Join() {
    processor := NewTextProcessor(" ")
    result := processor.Join("hello", "there", "friend")
    fmt.Println(result)
    // Output: hello there friend
}

// ExampleTextProcessor_Join_customDelimiter shows custom delimiter usage
func ExampleTextProcessor_Join_customDelimiter() {
    processor := NewTextProcessor(", ")
    result := processor.Join("apples", "oranges", "bananas")
    fmt.Println(result)
    // Output: apples, oranges, bananas
}

// ExampleTextProcessor_Split demonstrates text splitting
func ExampleTextProcessor_Split() {
    processor := NewTextProcessor(" ")
    words := processor.Split("hello there friend")
    for i, word := range words {
        fmt.Printf("Word %d: %s\n", i+1, word)
    }
    // Output:
    // Word 1: hello
    // Word 2: there
    // Word 3: friend
}

// ExampleTextProcessor_WordCount shows word counting functionality
func ExampleTextProcessor_WordCount() {
    processor := NewTextProcessor(" ")
    
    count1 := processor.WordCount("hello there friend")
    fmt.Printf("Count 1: %d\n", count1)
    
    count2 := processor.WordCount("")
    fmt.Printf("Count 2: %d\n", count2)
    
    count3 := processor.WordCount("   hello   there   ")
    fmt.Printf("Count 3: %d\n", count3)
    // Output:
    // Count 1: 3
    // Count 2: 0
    // Count 3: 2
}

// ExampleSortWords demonstrates word sorting
func ExampleSortWords() {
    words := []string{"zebra", "apple", "banana", "cherry"}
    sorted := SortWords(words)
    
    fmt.Println("Original:", words)
    fmt.Println("Sorted:", sorted)
    // Output:
    // Original: [zebra apple banana cherry]
    // Sorted: [apple banana cherry zebra]
}

// ExampleNewTextProcessor shows processor creation
func ExampleNewTextProcessor() {
    // Create processor with hyphen delimiter
    processor := NewTextProcessor("-")
    result := processor.Join("hello", "there", "world")
    fmt.Println(result)
    // Output: hello-there-world
}
```

Example tests provide living documentation that stays current with code  
changes. The output comments are verified during test execution, ensuring  
examples remain accurate and demonstrate proper usage patterns.  

## Parallel test execution

Parallel tests improve test suite performance by running independent tests  
concurrently, taking advantage of multiple CPU cores.  

```go
package main

import (
    "crypto/md5"
    "fmt"
    "math/rand"
    "testing"
    "time"
)

// HashGenerator provides various hashing functions
type HashGenerator struct {
    salt string
}

// NewHashGenerator creates a hash generator with salt
func NewHashGenerator(salt string) *HashGenerator {
    return &HashGenerator{salt: salt}
}

// MD5Hash generates MD5 hash of input with salt
func (hg *HashGenerator) MD5Hash(input string) string {
    data := input + hg.salt
    hash := md5.Sum([]byte(data))
    return fmt.Sprintf("%x", hash)
}

// SimulateWork performs CPU-intensive work
func (hg *HashGenerator) SimulateWork(input string, iterations int) string {
    result := input
    for i := 0; i < iterations; i++ {
        result = hg.MD5Hash(result)
    }
    return result
}

// ProcessBatch processes multiple inputs
func (hg *HashGenerator) ProcessBatch(inputs []string) []string {
    results := make([]string, len(inputs))
    for i, input := range inputs {
        results[i] = hg.MD5Hash(input)
    }
    return results
}

func generateTestInputs(count int) []string {
    inputs := make([]string, count)
    rand.Seed(42) // Fixed seed for reproducible tests
    for i := 0; i < count; i++ {
        inputs[i] = fmt.Sprintf("input_%d_%d", i, rand.Intn(10000))
    }
    return inputs
}

func TestHashGeneratorParallel(t *testing.T) {
    inputs := generateTestInputs(100)
    
    t.Run("MD5Hash", func(t *testing.T) {
        t.Parallel()
        
        hg := NewHashGenerator("salt1")
        for i, input := range inputs {
            t.Run(fmt.Sprintf("Input_%d", i), func(t *testing.T) {
                t.Parallel()
                
                hash := hg.MD5Hash(input)
                if len(hash) != 32 {
                    t.Errorf("Expected hash length 32, got %d", len(hash))
                }
                
                // Verify consistency
                hash2 := hg.MD5Hash(input)
                if hash != hash2 {
                    t.Error("Hash should be consistent for same input")
                }
            })
        }
    })
    
    t.Run("ProcessBatch", func(t *testing.T) {
        t.Parallel()
        
        hg := NewHashGenerator("salt2")
        batch := inputs[:10]
        results := hg.ProcessBatch(batch)
        
        if len(results) != len(batch) {
            t.Errorf("Expected %d results, got %d", len(batch), len(results))
        }
        
        for i, result := range results {
            if len(result) != 32 {
                t.Errorf("Result %d: expected hash length 32, got %d", i, len(result))
            }
        }
    })
    
    t.Run("DifferentSalts", func(t *testing.T) {
        t.Parallel()
        
        input := "test input"
        salts := []string{"salt1", "salt2", "salt3", "salt4", "salt5"}
        
        for _, salt := range salts {
            t.Run(fmt.Sprintf("Salt_%s", salt), func(t *testing.T) {
                t.Parallel()
                
                hg := NewHashGenerator(salt)
                hash := hg.MD5Hash(input)
                
                if len(hash) != 32 {
                    t.Errorf("Expected hash length 32, got %d", len(hash))
                }
            })
        }
    })
}

func TestSimulateWorkParallel(t *testing.T) {
    hg := NewHashGenerator("work_salt")
    
    workloads := []struct {
        name       string
        input      string
        iterations int
    }{
        {"Light", "input1", 5},
        {"Medium", "input2", 10},
        {"Heavy", "input3", 15},
        {"Intensive", "input4", 20},
    }
    
    for _, workload := range workloads {
        t.Run(workload.name, func(t *testing.T) {
            t.Parallel()
            
            start := time.Now()
            result := hg.SimulateWork(workload.input, workload.iterations)
            duration := time.Since(start)
            
            if len(result) != 32 {
                t.Errorf("Expected result length 32, got %d", len(result))
            }
            
            t.Logf("Workload %s completed in %v", workload.name, duration)
        })
    }
}

func TestConcurrentAccess(t *testing.T) {
    t.Parallel()
    
    hg := NewHashGenerator("concurrent_salt")
    input := "concurrent test input"
    
    // Run multiple goroutines to test concurrent access
    done := make(chan bool, 10)
    
    for i := 0; i < 10; i++ {
        go func(id int) {
            defer func() { done <- true }()
            
            for j := 0; j < 100; j++ {
                testInput := fmt.Sprintf("%s_%d_%d", input, id, j)
                hash := hg.MD5Hash(testInput)
                
                if len(hash) != 32 {
                    t.Errorf("Goroutine %d: expected hash length 32, got %d", id, len(hash))
                    return
                }
            }
        }(i)
    }
    
    // Wait for all goroutines to complete
    for i := 0; i < 10; i++ {
        <-done
    }
}
```

Parallel tests execute concurrently when marked with `t.Parallel()`, improving  
test suite performance for CPU-intensive operations. This approach is  
particularly beneficial for tests with significant computation or I/O operations.  

## Testing with interfaces and mocks

Interface-based testing enables isolation of dependencies through mocking,  
allowing focused testing of individual components without external dependencies.  

```go
package main

import (
    "errors"
    "fmt"
    "testing"
)

// Database interface for data operations
type Database interface {
    Store(key, value string) error
    Retrieve(key string) (string, error)
    Delete(key string) error
}

// Logger interface for logging operations
type Logger interface {
    Info(message string)
    Error(message string)
}

// UserService handles user operations
type UserService struct {
    db     Database
    logger Logger
}

// NewUserService creates a user service
func NewUserService(db Database, logger Logger) *UserService {
    return &UserService{db: db, logger: logger}
}

// SaveUser saves user data
func (us *UserService) SaveUser(username, email string) error {
    if username == "" {
        return errors.New("username cannot be empty")
    }
    
    if email == "" {
        return errors.New("email cannot be empty")
    }
    
    us.logger.Info(fmt.Sprintf("Saving user: %s", username))
    
    err := us.db.Store(username, email)
    if err != nil {
        us.logger.Error(fmt.Sprintf("Failed to save user %s: %v", username, err))
        return fmt.Errorf("failed to save user: %w", err)
    }
    
    us.logger.Info(fmt.Sprintf("User %s saved successfully", username))
    return nil
}

// GetUser retrieves user data
func (us *UserService) GetUser(username string) (string, error) {
    if username == "" {
        return "", errors.New("username cannot be empty")
    }
    
    us.logger.Info(fmt.Sprintf("Retrieving user: %s", username))
    
    email, err := us.db.Retrieve(username)
    if err != nil {
        us.logger.Error(fmt.Sprintf("Failed to retrieve user %s: %v", username, err))
        return "", fmt.Errorf("failed to retrieve user: %w", err)
    }
    
    return email, nil
}

// Mock implementations for testing
type MockDatabase struct {
    data        map[string]string
    shouldError bool
    errorMsg    string
}

func NewMockDatabase() *MockDatabase {
    return &MockDatabase{
        data: make(map[string]string),
    }
}

func (md *MockDatabase) Store(key, value string) error {
    if md.shouldError {
        return errors.New(md.errorMsg)
    }
    md.data[key] = value
    return nil
}

func (md *MockDatabase) Retrieve(key string) (string, error) {
    if md.shouldError {
        return "", errors.New(md.errorMsg)
    }
    
    value, exists := md.data[key]
    if !exists {
        return "", errors.New("key not found")
    }
    return value, nil
}

func (md *MockDatabase) Delete(key string) error {
    if md.shouldError {
        return errors.New(md.errorMsg)
    }
    delete(md.data, key)
    return nil
}

func (md *MockDatabase) SetError(shouldError bool, errorMsg string) {
    md.shouldError = shouldError
    md.errorMsg = errorMsg
}

type MockLogger struct {
    infoMessages  []string
    errorMessages []string
}

func NewMockLogger() *MockLogger {
    return &MockLogger{
        infoMessages:  make([]string, 0),
        errorMessages: make([]string, 0),
    }
}

func (ml *MockLogger) Info(message string) {
    ml.infoMessages = append(ml.infoMessages, message)
}

func (ml *MockLogger) Error(message string) {
    ml.errorMessages = append(ml.errorMessages, message)
}

func TestUserServiceSaveUser(t *testing.T) {
    db := NewMockDatabase()
    logger := NewMockLogger()
    service := NewUserService(db, logger)
    
    t.Run("ValidUser", func(t *testing.T) {
        err := service.SaveUser("alice", "alice@example.com")
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        // Verify data was stored
        email, exists := db.data["alice"]
        if !exists {
            t.Error("Expected user to be stored in database")
        }
        
        if email != "alice@example.com" {
            t.Errorf("Expected email 'alice@example.com', got %q", email)
        }
        
        // Verify logging
        if len(logger.infoMessages) != 2 {
            t.Errorf("Expected 2 info messages, got %d", len(logger.infoMessages))
        }
    })
    
    t.Run("EmptyUsername", func(t *testing.T) {
        err := service.SaveUser("", "email@example.com")
        if err == nil {
            t.Error("Expected error for empty username")
        }
        
        expectedMsg := "username cannot be empty"
        if err.Error() != expectedMsg {
            t.Errorf("Expected error %q, got %q", expectedMsg, err.Error())
        }
    })
    
    t.Run("DatabaseError", func(t *testing.T) {
        db.SetError(true, "database connection failed")
        
        err := service.SaveUser("bob", "bob@example.com")
        if err == nil {
            t.Error("Expected error when database fails")
        }
        
        // Verify error was logged
        if len(logger.errorMessages) == 0 {
            t.Error("Expected error to be logged")
        }
        
        db.SetError(false, "") // Reset for other tests
    })
}

func TestUserServiceGetUser(t *testing.T) {
    db := NewMockDatabase()
    logger := NewMockLogger()
    service := NewUserService(db, logger)
    
    // Setup test data
    db.data["alice"] = "alice@example.com"
    
    t.Run("ExistingUser", func(t *testing.T) {
        email, err := service.GetUser("alice")
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        expected := "alice@example.com"
        if email != expected {
            t.Errorf("Expected email %q, got %q", expected, email)
        }
        
        // Verify logging
        if len(logger.infoMessages) == 0 {
            t.Error("Expected info message to be logged")
        }
    })
    
    t.Run("NonExistentUser", func(t *testing.T) {
        _, err := service.GetUser("nonexistent")
        if err == nil {
            t.Error("Expected error for nonexistent user")
        }
        
        // Verify error was logged
        if len(logger.errorMessages) == 0 {
            t.Error("Expected error to be logged")
        }
    })
}
```

Interface-based testing with mocks provides complete control over dependencies,  
enabling comprehensive testing of both success and failure scenarios. Mock  
objects capture interactions and allow verification of behavior without  
external dependencies.  

## Test data generation and fixtures

Test fixtures and data generation provide consistent, reusable test data  
that supports comprehensive testing scenarios and edge case validation.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "math/rand"
    "testing"
    "time"
)

// User represents a user entity
type User struct {
    ID       int       `json:"id"`
    Name     string    `json:"name"`
    Email    string    `json:"email"`
    Age      int       `json:"age"`
    Created  time.Time `json:"created"`
    Active   bool      `json:"active"`
    Roles    []string  `json:"roles"`
    Metadata map[string]interface{} `json:"metadata"`
}

// UserBuilder helps construct test users
type UserBuilder struct {
    user User
}

// NewUserBuilder creates a new user builder with defaults
func NewUserBuilder() *UserBuilder {
    return &UserBuilder{
        user: User{
            ID:       1,
            Name:     "Test User",
            Email:    "test@example.com",
            Age:      25,
            Created:  time.Now(),
            Active:   true,
            Roles:    []string{"user"},
            Metadata: make(map[string]interface{}),
        },
    }
}

// WithID sets the user ID
func (ub *UserBuilder) WithID(id int) *UserBuilder {
    ub.user.ID = id
    return ub
}

// WithName sets the user name
func (ub *UserBuilder) WithName(name string) *UserBuilder {
    ub.user.Name = name
    return ub
}

// WithEmail sets the user email
func (ub *UserBuilder) WithEmail(email string) *UserBuilder {
    ub.user.Email = email
    return ub
}

// WithAge sets the user age
func (ub *UserBuilder) WithAge(age int) *UserBuilder {
    ub.user.Age = age
    return ub
}

// WithRoles sets the user roles
func (ub *UserBuilder) WithRoles(roles ...string) *UserBuilder {
    ub.user.Roles = roles
    return ub
}

// WithMetadata adds metadata
func (ub *UserBuilder) WithMetadata(key string, value interface{}) *UserBuilder {
    ub.user.Metadata[key] = value
    return ub
}

// Inactive makes the user inactive
func (ub *UserBuilder) Inactive() *UserBuilder {
    ub.user.Active = false
    return ub
}

// Build creates the user
func (ub *UserBuilder) Build() User {
    return ub.user
}

// UserValidator validates user data
type UserValidator struct{}

// Validate checks if user data is valid
func (uv *UserValidator) Validate(user User) []string {
    var errors []string
    
    if user.ID <= 0 {
        errors = append(errors, "ID must be positive")
    }
    
    if user.Name == "" {
        errors = append(errors, "name cannot be empty")
    }
    
    if user.Email == "" {
        errors = append(errors, "email cannot be empty")
    }
    
    if user.Age < 0 || user.Age > 150 {
        errors = append(errors, "age must be between 0 and 150")
    }
    
    if len(user.Roles) == 0 {
        errors = append(errors, "user must have at least one role")
    }
    
    return errors
}

// IsValid checks if user is valid
func (uv *UserValidator) IsValid(user User) bool {
    return len(uv.Validate(user)) == 0
}

// Test fixtures
func validUser() User {
    return NewUserBuilder().
        WithID(1).
        WithName("Alice Johnson").
        WithEmail("alice@example.com").
        WithAge(30).
        WithRoles("user", "admin").
        WithMetadata("department", "engineering").
        Build()
}

func invalidUser() User {
    return NewUserBuilder().
        WithID(-1).
        WithName("").
        WithEmail("").
        WithAge(-5).
        WithRoles().
        Build()
}

func generateRandomUsers(count int) []User {
    rand.Seed(42) // Fixed seed for reproducible tests
    users := make([]User, count)
    
    names := []string{"Alice", "Bob", "Charlie", "Diana", "Eve", "Frank"}
    domains := []string{"example.com", "test.org", "demo.net"}
    roles := []string{"user", "admin", "moderator", "guest"}
    
    for i := 0; i < count; i++ {
        name := names[rand.Intn(len(names))]
        domain := domains[rand.Intn(len(domains))]
        email := fmt.Sprintf("%s%d@%s", name, i, domain)
        
        roleCount := rand.Intn(3) + 1
        userRoles := make([]string, roleCount)
        for j := 0; j < roleCount; j++ {
            userRoles[j] = roles[rand.Intn(len(roles))]
        }
        
        users[i] = NewUserBuilder().
            WithID(i + 1).
            WithName(fmt.Sprintf("%s %d", name, i)).
            WithEmail(email).
            WithAge(rand.Intn(50) + 18).
            WithRoles(userRoles...).
            WithMetadata("generated", true).
            Build()
    }
    
    return users
}

func TestUserBuilder(t *testing.T) {
    t.Run("DefaultUser", func(t *testing.T) {
        user := NewUserBuilder().Build()
        
        if user.ID != 1 {
            t.Errorf("Expected ID 1, got %d", user.ID)
        }
        
        if user.Name != "Test User" {
            t.Errorf("Expected name 'Test User', got %q", user.Name)
        }
        
        if !user.Active {
            t.Error("Expected user to be active by default")
        }
    })
    
    t.Run("CustomUser", func(t *testing.T) {
        user := NewUserBuilder().
            WithID(42).
            WithName("Custom User").
            WithEmail("custom@example.com").
            WithAge(35).
            WithRoles("admin", "moderator").
            Inactive().
            Build()
        
        if user.ID != 42 {
            t.Errorf("Expected ID 42, got %d", user.ID)
        }
        
        if user.Active {
            t.Error("Expected user to be inactive")
        }
        
        expectedRoles := []string{"admin", "moderator"}
        if len(user.Roles) != len(expectedRoles) {
            t.Errorf("Expected %d roles, got %d", len(expectedRoles), len(user.Roles))
        }
    })
}

func TestUserValidator(t *testing.T) {
    validator := &UserValidator{}
    
    t.Run("ValidUser", func(t *testing.T) {
        user := validUser()
        
        if !validator.IsValid(user) {
            t.Error("Expected valid user to pass validation")
        }
        
        errors := validator.Validate(user)
        if len(errors) != 0 {
            t.Errorf("Expected no validation errors, got: %v", errors)
        }
    })
    
    t.Run("InvalidUser", func(t *testing.T) {
        user := invalidUser()
        
        if validator.IsValid(user) {
            t.Error("Expected invalid user to fail validation")
        }
        
        errors := validator.Validate(user)
        if len(errors) == 0 {
            t.Error("Expected validation errors for invalid user")
        }
        
        expectedErrors := 5 // ID, name, email, age, roles
        if len(errors) != expectedErrors {
            t.Errorf("Expected %d validation errors, got %d: %v", expectedErrors, len(errors), errors)
        }
    })
    
    t.Run("EdgeCases", func(t *testing.T) {
        testCases := []struct {
            name     string
            user     User
            expectValid bool
        }{
            {
                name: "MinAge",
                user: NewUserBuilder().WithAge(0).Build(),
                expectValid: true,
            },
            {
                name: "MaxAge",
                user: NewUserBuilder().WithAge(150).Build(),
                expectValid: true,
            },
            {
                name: "TooOld",
                user: NewUserBuilder().WithAge(151).Build(),
                expectValid: false,
            },
            {
                name: "SingleRole",
                user: NewUserBuilder().WithRoles("user").Build(),
                expectValid: true,
            },
        }
        
        for _, tc := range testCases {
            t.Run(tc.name, func(t *testing.T) {
                isValid := validator.IsValid(tc.user)
                if isValid != tc.expectValid {
                    t.Errorf("Expected valid=%t, got valid=%t", tc.expectValid, isValid)
                }
            })
        }
    })
}

func TestRandomUserGeneration(t *testing.T) {
    users := generateRandomUsers(50)
    validator := &UserValidator{}
    
    if len(users) != 50 {
        t.Errorf("Expected 50 users, got %d", len(users))
    }
    
    validCount := 0
    for _, user := range users {
        if validator.IsValid(user) {
            validCount++
        }
    }
    
    if validCount != 50 {
        t.Errorf("Expected all 50 users to be valid, got %d valid users", validCount)
    }
    
    // Verify unique IDs
    ids := make(map[int]bool)
    for _, user := range users {
        if ids[user.ID] {
            t.Errorf("Duplicate ID found: %d", user.ID)
        }
        ids[user.ID] = true
    }
}
```

Test fixtures and builders provide consistent, maintainable test data while  
supporting complex object construction. Data generation creates diverse test  
scenarios and helps discover edge cases through systematic variation.  

## HTTP handler testing

HTTP handler testing validates web endpoints by simulating requests and  
verifying responses, ensuring proper handling of various HTTP scenarios.  

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
)

// APIResponse represents a standard API response
type APIResponse struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
}

// UserHandler handles user-related HTTP requests
type UserHandler struct {
    users map[string]User
}

// NewUserHandler creates a new user handler
func NewUserHandler() *UserHandler {
    return &UserHandler{
        users: make(map[string]User),
    }
}

// GetUser handles GET /users/{username}
func (uh *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        uh.writeError(w, http.StatusMethodNotAllowed, "method not allowed")
        return
    }
    
    username := strings.TrimPrefix(r.URL.Path, "/users/")
    if username == "" {
        uh.writeError(w, http.StatusBadRequest, "username required")
        return
    }
    
    user, exists := uh.users[username]
    if !exists {
        uh.writeError(w, http.StatusNotFound, "user not found")
        return
    }
    
    uh.writeSuccess(w, http.StatusOK, user)
}

// CreateUser handles POST /users
func (uh *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        uh.writeError(w, http.StatusMethodNotAllowed, "method not allowed")
        return
    }
    
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        uh.writeError(w, http.StatusBadRequest, "invalid JSON")
        return
    }
    
    if user.Name == "" {
        uh.writeError(w, http.StatusBadRequest, "name is required")
        return
    }
    
    if user.Email == "" {
        uh.writeError(w, http.StatusBadRequest, "email is required")
        return
    }
    
    // Check if user already exists
    if _, exists := uh.users[user.Name]; exists {
        uh.writeError(w, http.StatusConflict, "user already exists")
        return
    }
    
    uh.users[user.Name] = user
    uh.writeSuccess(w, http.StatusCreated, user)
}

// ListUsers handles GET /users
func (uh *UserHandler) ListUsers(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        uh.writeError(w, http.StatusMethodNotAllowed, "method not allowed")
        return
    }
    
    users := make([]User, 0, len(uh.users))
    for _, user := range uh.users {
        users = append(users, user)
    }
    
    uh.writeSuccess(w, http.StatusOK, users)
}

func (uh *UserHandler) writeSuccess(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    
    response := APIResponse{
        Success: true,
        Data:    data,
    }
    
    json.NewEncoder(w).Encode(response)
}

func (uh *UserHandler) writeError(w http.ResponseWriter, status int, message string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    
    response := APIResponse{
        Success: false,
        Error:   message,
    }
    
    json.NewEncoder(w).Encode(response)
}

func TestUserHandler_GetUser(t *testing.T) {
    handler := NewUserHandler()
    
    // Setup test data
    testUser := User{
        ID:    1,
        Name:  "alice",
        Email: "alice@example.com",
        Age:   30,
    }
    handler.users["alice"] = testUser
    
    t.Run("ExistingUser", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodGet, "/users/alice", nil)
        w := httptest.NewRecorder()
        
        handler.GetUser(w, req)
        
        if w.Code != http.StatusOK {
            t.Errorf("Expected status %d, got %d", http.StatusOK, w.Code)
        }
        
        var response APIResponse
        if err := json.NewDecoder(w.Body).Decode(&response); err != nil {
            t.Fatalf("Failed to decode response: %v", err)
        }
        
        if !response.Success {
            t.Error("Expected successful response")
        }
        
        // Verify response data
        userData, ok := response.Data.(map[string]interface{})
        if !ok {
            t.Fatal("Expected user data in response")
        }
        
        if userData["name"] != "alice" {
            t.Errorf("Expected name 'alice', got %v", userData["name"])
        }
    })
    
    t.Run("NonExistentUser", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodGet, "/users/nonexistent", nil)
        w := httptest.NewRecorder()
        
        handler.GetUser(w, req)
        
        if w.Code != http.StatusNotFound {
            t.Errorf("Expected status %d, got %d", http.StatusNotFound, w.Code)
        }
        
        var response APIResponse
        if err := json.NewDecoder(w.Body).Decode(&response); err != nil {
            t.Fatalf("Failed to decode response: %v", err)
        }
        
        if response.Success {
            t.Error("Expected error response")
        }
        
        if response.Error != "user not found" {
            t.Errorf("Expected error 'user not found', got %q", response.Error)
        }
    })
    
    t.Run("EmptyUsername", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodGet, "/users/", nil)
        w := httptest.NewRecorder()
        
        handler.GetUser(w, req)
        
        if w.Code != http.StatusBadRequest {
            t.Errorf("Expected status %d, got %d", http.StatusBadRequest, w.Code)
        }
    })
    
    t.Run("WrongMethod", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodPost, "/users/alice", nil)
        w := httptest.NewRecorder()
        
        handler.GetUser(w, req)
        
        if w.Code != http.StatusMethodNotAllowed {
            t.Errorf("Expected status %d, got %d", http.StatusMethodNotAllowed, w.Code)
        }
    })
}

func TestUserHandler_CreateUser(t *testing.T) {
    handler := NewUserHandler()
    
    t.Run("ValidUser", func(t *testing.T) {
        user := User{
            ID:    1,
            Name:  "bob",
            Email: "bob@example.com",
            Age:   25,
        }
        
        jsonData, _ := json.Marshal(user)
        req := httptest.NewRequest(http.MethodPost, "/users", bytes.NewBuffer(jsonData))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()
        
        handler.CreateUser(w, req)
        
        if w.Code != http.StatusCreated {
            t.Errorf("Expected status %d, got %d", http.StatusCreated, w.Code)
        }
        
        // Verify user was created
        if _, exists := handler.users["bob"]; !exists {
            t.Error("Expected user to be created")
        }
    })
    
    t.Run("InvalidJSON", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodPost, "/users", strings.NewReader("invalid json"))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()
        
        handler.CreateUser(w, req)
        
        if w.Code != http.StatusBadRequest {
            t.Errorf("Expected status %d, got %d", http.StatusBadRequest, w.Code)
        }
    })
    
    t.Run("MissingName", func(t *testing.T) {
        user := User{Email: "test@example.com"}
        jsonData, _ := json.Marshal(user)
        
        req := httptest.NewRequest(http.MethodPost, "/users", bytes.NewBuffer(jsonData))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()
        
        handler.CreateUser(w, req)
        
        if w.Code != http.StatusBadRequest {
            t.Errorf("Expected status %d, got %d", http.StatusBadRequest, w.Code)
        }
    })
    
    t.Run("DuplicateUser", func(t *testing.T) {
        // First, create a user
        user := User{Name: "duplicate", Email: "dup@example.com"}
        handler.users["duplicate"] = user
        
        // Try to create the same user again
        jsonData, _ := json.Marshal(user)
        req := httptest.NewRequest(http.MethodPost, "/users", bytes.NewBuffer(jsonData))
        req.Header.Set("Content-Type", "application/json")
        w := httptest.NewRecorder()
        
        handler.CreateUser(w, req)
        
        if w.Code != http.StatusConflict {
            t.Errorf("Expected status %d, got %d", http.StatusConflict, w.Code)
        }
    })
}

func TestUserHandler_ListUsers(t *testing.T) {
    handler := NewUserHandler()
    
    t.Run("EmptyList", func(t *testing.T) {
        req := httptest.NewRequest(http.MethodGet, "/users", nil)
        w := httptest.NewRecorder()
        
        handler.ListUsers(w, req)
        
        if w.Code != http.StatusOK {
            t.Errorf("Expected status %d, got %d", http.StatusOK, w.Code)
        }
        
        var response APIResponse
        if err := json.NewDecoder(w.Body).Decode(&response); err != nil {
            t.Fatalf("Failed to decode response: %v", err)
        }
        
        users, ok := response.Data.([]interface{})
        if !ok {
            t.Fatal("Expected array of users")
        }
        
        if len(users) != 0 {
            t.Errorf("Expected empty user list, got %d users", len(users))
        }
    })
    
    t.Run("WithUsers", func(t *testing.T) {
        // Add test users
        handler.users["user1"] = User{Name: "user1", Email: "user1@example.com"}
        handler.users["user2"] = User{Name: "user2", Email: "user2@example.com"}
        
        req := httptest.NewRequest(http.MethodGet, "/users", nil)
        w := httptest.NewRecorder()
        
        handler.ListUsers(w, req)
        
        if w.Code != http.StatusOK {
            t.Errorf("Expected status %d, got %d", http.StatusOK, w.Code)
        }
        
        var response APIResponse
        if err := json.NewDecoder(w.Body).Decode(&response); err != nil {
            t.Fatalf("Failed to decode response: %v", err)
        }
        
        users, ok := response.Data.([]interface{})
        if !ok {
            t.Fatal("Expected array of users")
        }
        
        if len(users) != 2 {
            t.Errorf("Expected 2 users, got %d", len(users))
        }
    })
}
```

HTTP handler testing uses `httptest` package to simulate requests and verify  
responses without starting actual servers. This approach enables comprehensive  
testing of web endpoints, status codes, headers, and response content.  

## Test coverage and analysis

Test coverage analysis helps identify untested code paths and ensures  
comprehensive test coverage across the entire codebase.  

```go
package main

import (
    "errors"
    "fmt"
    "testing"
)

// Calculator provides mathematical operations with error handling
type Calculator struct {
    history []Operation
    debug   bool
}

// Operation represents a mathematical operation
type Operation struct {
    Type   string
    A, B   float64
    Result float64
    Error  string
}

// NewCalculator creates a new calculator
func NewCalculator() *Calculator {
    return &Calculator{
        history: make([]Operation, 0),
    }
}

// SetDebug enables or disables debug mode
func (c *Calculator) SetDebug(enabled bool) {
    c.debug = enabled
}

// Add performs addition
func (c *Calculator) Add(a, b float64) float64 {
    result := a + b
    op := Operation{Type: "add", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f + %f = %f\n", a, b, result)
    }
    
    return result
}

// Subtract performs subtraction
func (c *Calculator) Subtract(a, b float64) float64 {
    result := a - b
    op := Operation{Type: "subtract", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f - %f = %f\n", a, b, result)
    }
    
    return result
}

// Multiply performs multiplication
func (c *Calculator) Multiply(a, b float64) float64 {
    result := a * b
    op := Operation{Type: "multiply", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f * %f = %f\n", a, b, result)
    }
    
    return result
}

// Divide performs division with error handling
func (c *Calculator) Divide(a, b float64) (float64, error) {
    if b == 0 {
        op := Operation{Type: "divide", A: a, B: b, Error: "division by zero"}
        c.addToHistory(op)
        return 0, errors.New("division by zero")
    }
    
    result := a / b
    op := Operation{Type: "divide", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f / %f = %f\n", a, b, result)
    }
    
    return result, nil
}

// Modulo performs modulo operation
func (c *Calculator) Modulo(a, b float64) (float64, error) {
    if b == 0 {
        op := Operation{Type: "modulo", A: a, B: b, Error: "modulo by zero"}
        c.addToHistory(op)
        return 0, errors.New("modulo by zero")
    }
    
    // This is a simplified modulo for demonstration
    result := float64(int(a) % int(b))
    op := Operation{Type: "modulo", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f %% %f = %f\n", a, b, result)
    }
    
    return result, nil
}

// Power raises a to the power of b (simplified)
func (c *Calculator) Power(a, b float64) (float64, error) {
    if a == 0 && b < 0 {
        op := Operation{Type: "power", A: a, B: b, Error: "zero to negative power"}
        c.addToHistory(op)
        return 0, errors.New("zero to negative power")
    }
    
    // Simplified power calculation
    var result float64 = 1
    if b >= 0 {
        for i := 0; i < int(b); i++ {
            result *= a
        }
    } else {
        // Handle negative powers (simplified)
        for i := 0; i < int(-b); i++ {
            result /= a
        }
    }
    
    op := Operation{Type: "power", A: a, B: b, Result: result}
    c.addToHistory(op)
    
    if c.debug {
        fmt.Printf("Debug: %f ^ %f = %f\n", a, b, result)
    }
    
    return result, nil
}

// GetHistory returns operation history
func (c *Calculator) GetHistory() []Operation {
    history := make([]Operation, len(c.history))
    copy(history, c.history)
    return history
}

// ClearHistory clears operation history
func (c *Calculator) ClearHistory() {
    c.history = c.history[:0]
}

// GetStats returns calculation statistics
func (c *Calculator) GetStats() map[string]int {
    stats := make(map[string]int)
    
    for _, op := range c.history {
        stats[op.Type]++
    }
    
    return stats
}

func (c *Calculator) addToHistory(op Operation) {
    c.history = append(c.history, op)
}

// Comprehensive test coverage
func TestCalculatorAdd(t *testing.T) {
    calc := NewCalculator()
    
    result := calc.Add(5, 3)
    if result != 8 {
        t.Errorf("Add(5, 3) = %f, expected 8", result)
    }
    
    // Test with negative numbers
    result = calc.Add(-5, -3)
    if result != -8 {
        t.Errorf("Add(-5, -3) = %f, expected -8", result)
    }
    
    // Test with zero
    result = calc.Add(0, 5)
    if result != 5 {
        t.Errorf("Add(0, 5) = %f, expected 5", result)
    }
}

func TestCalculatorSubtract(t *testing.T) {
    calc := NewCalculator()
    
    result := calc.Subtract(10, 3)
    if result != 7 {
        t.Errorf("Subtract(10, 3) = %f, expected 7", result)
    }
    
    result = calc.Subtract(3, 10)
    if result != -7 {
        t.Errorf("Subtract(3, 10) = %f, expected -7", result)
    }
}

func TestCalculatorMultiply(t *testing.T) {
    calc := NewCalculator()
    
    result := calc.Multiply(4, 5)
    if result != 20 {
        t.Errorf("Multiply(4, 5) = %f, expected 20", result)
    }
    
    result = calc.Multiply(-2, 3)
    if result != -6 {
        t.Errorf("Multiply(-2, 3) = %f, expected -6", result)
    }
    
    result = calc.Multiply(0, 100)
    if result != 0 {
        t.Errorf("Multiply(0, 100) = %f, expected 0", result)
    }
}

func TestCalculatorDivide(t *testing.T) {
    calc := NewCalculator()
    
    result, err := calc.Divide(10, 2)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 5 {
        t.Errorf("Divide(10, 2) = %f, expected 5", result)
    }
    
    _, err = calc.Divide(10, 0)
    if err == nil {
        t.Error("Expected error for division by zero")
    }
}

func TestCalculatorModulo(t *testing.T) {
    calc := NewCalculator()
    
    result, err := calc.Modulo(10, 3)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 1 {
        t.Errorf("Modulo(10, 3) = %f, expected 1", result)
    }
    
    _, err = calc.Modulo(10, 0)
    if err == nil {
        t.Error("Expected error for modulo by zero")
    }
}

func TestCalculatorPower(t *testing.T) {
    calc := NewCalculator()
    
    result, err := calc.Power(2, 3)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 8 {
        t.Errorf("Power(2, 3) = %f, expected 8", result)
    }
    
    result, err = calc.Power(5, 0)
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != 1 {
        t.Errorf("Power(5, 0) = %f, expected 1", result)
    }
    
    _, err = calc.Power(0, -1)
    if err == nil {
        t.Error("Expected error for zero to negative power")
    }
}

func TestCalculatorDebugMode(t *testing.T) {
    calc := NewCalculator()
    calc.SetDebug(true)
    
    // These operations should work in debug mode
    calc.Add(1, 2)
    calc.Subtract(5, 3)
    calc.Multiply(2, 4)
    calc.Divide(10, 2)
    
    calc.SetDebug(false)
    // These operations should work in normal mode
    calc.Add(1, 1)
}

func TestCalculatorHistory(t *testing.T) {
    calc := NewCalculator()
    
    calc.Add(1, 2)
    calc.Subtract(5, 3)
    calc.Multiply(2, 4)
    
    history := calc.GetHistory()
    if len(history) != 3 {
        t.Errorf("Expected 3 operations in history, got %d", len(history))
    }
    
    calc.ClearHistory()
    history = calc.GetHistory()
    if len(history) != 0 {
        t.Errorf("Expected empty history after clear, got %d", len(history))
    }
}

func TestCalculatorStats(t *testing.T) {
    calc := NewCalculator()
    
    calc.Add(1, 2)
    calc.Add(3, 4)
    calc.Subtract(5, 3)
    calc.Multiply(2, 4)
    calc.Divide(10, 0) // This will create an error operation
    
    stats := calc.GetStats()
    
    if stats["add"] != 2 {
        t.Errorf("Expected 2 add operations, got %d", stats["add"])
    }
    
    if stats["subtract"] != 1 {
        t.Errorf("Expected 1 subtract operation, got %d", stats["subtract"])
    }
    
    if stats["multiply"] != 1 {
        t.Errorf("Expected 1 multiply operation, got %d", stats["multiply"])
    }
    
    if stats["divide"] != 1 {
        t.Errorf("Expected 1 divide operation, got %d", stats["divide"])
    }
}

func TestCalculatorErrorHandling(t *testing.T) {
    calc := NewCalculator()
    
    // Test all error conditions
    _, err := calc.Divide(1, 0)
    if err == nil {
        t.Error("Expected division by zero error")
    }
    
    _, err = calc.Modulo(1, 0)
    if err == nil {
        t.Error("Expected modulo by zero error")
    }
    
    _, err = calc.Power(0, -1)
    if err == nil {
        t.Error("Expected zero to negative power error")
    }
    
    // Verify error operations are recorded in history
    history := calc.GetHistory()
    errorCount := 0
    for _, op := range history {
        if op.Error != "" {
            errorCount++
        }
    }
    
    if errorCount != 3 {
        t.Errorf("Expected 3 error operations in history, got %d", errorCount)
    }
}
```

Test coverage analysis identifies untested code paths and ensures comprehensive  
validation of all functionality including error conditions, edge cases, and  
different execution paths. Coverage tools help maintain code quality standards.  

## Integration test patterns

Integration tests validate component interactions and end-to-end workflows,  
ensuring proper system behavior when multiple components work together.  

```go
package main

import (
    "fmt"
    "sync"
    "testing"
    "time"
)

// EventBus facilitates communication between components
type EventBus struct {
    subscribers map[string][]EventHandler
    mutex       sync.RWMutex
}

// EventHandler processes events
type EventHandler func(Event)

// Event represents an application event
type Event struct {
    Type      string
    Payload   interface{}
    Timestamp time.Time
}

// NewEventBus creates a new event bus
func NewEventBus() *EventBus {
    return &EventBus{
        subscribers: make(map[string][]EventHandler),
    }
}

// Subscribe adds an event handler for specific event type
func (eb *EventBus) Subscribe(eventType string, handler EventHandler) {
    eb.mutex.Lock()
    defer eb.mutex.Unlock()
    
    eb.subscribers[eventType] = append(eb.subscribers[eventType], handler)
}

// Publish sends an event to all subscribers
func (eb *EventBus) Publish(event Event) {
    eb.mutex.RLock()
    handlers := eb.subscribers[event.Type]
    eb.mutex.RUnlock()
    
    for _, handler := range handlers {
        go handler(event)
    }
}

// OrderProcessor handles order processing workflow
type OrderProcessor struct {
    eventBus    *EventBus
    orderRepo   OrderRepository
    paymentSvc  PaymentService
    inventorySvc InventoryService
    notificationSvc NotificationService
}

// OrderRepository manages order persistence
type OrderRepository interface {
    Save(order Order) error
    FindByID(id string) (Order, error)
    UpdateStatus(id string, status string) error
}

// PaymentService handles payment processing
type PaymentService interface {
    ProcessPayment(orderID string, amount float64) (string, error)
}

// InventoryService manages inventory
type InventoryService interface {
    ReserveItems(orderID string, items []OrderItem) error
    ReleaseItems(orderID string) error
}

// NotificationService sends notifications
type NotificationService interface {
    SendOrderConfirmation(orderID string) error
    SendPaymentConfirmation(orderID string, transactionID string) error
}

// Order represents a customer order
type Order struct {
    ID       string
    Customer string
    Items    []OrderItem
    Total    float64
    Status   string
    Created  time.Time
}

// OrderItem represents an item in an order
type OrderItem struct {
    ProductID string
    Quantity  int
    Price     float64
}

// NewOrderProcessor creates an order processor
func NewOrderProcessor(eventBus *EventBus, orderRepo OrderRepository, 
    paymentSvc PaymentService, inventorySvc InventoryService, 
    notificationSvc NotificationService) *OrderProcessor {
    
    processor := &OrderProcessor{
        eventBus:        eventBus,
        orderRepo:       orderRepo,
        paymentSvc:      paymentSvc,
        inventorySvc:    inventorySvc,
        notificationSvc: notificationSvc,
    }
    
    // Subscribe to events
    eventBus.Subscribe("order.created", processor.handleOrderCreated)
    eventBus.Subscribe("payment.completed", processor.handlePaymentCompleted)
    eventBus.Subscribe("payment.failed", processor.handlePaymentFailed)
    
    return processor
}

// ProcessOrder initiates order processing
func (op *OrderProcessor) ProcessOrder(order Order) error {
    order.Status = "pending"
    order.Created = time.Now()
    
    if err := op.orderRepo.Save(order); err != nil {
        return fmt.Errorf("failed to save order: %w", err)
    }
    
    // Publish order created event
    event := Event{
        Type:      "order.created",
        Payload:   order,
        Timestamp: time.Now(),
    }
    op.eventBus.Publish(event)
    
    return nil
}

func (op *OrderProcessor) handleOrderCreated(event Event) {
    order := event.Payload.(Order)
    
    // Reserve inventory
    if err := op.inventorySvc.ReserveItems(order.ID, order.Items); err != nil {
        op.orderRepo.UpdateStatus(order.ID, "failed")
        return
    }
    
    // Process payment
    transactionID, err := op.paymentSvc.ProcessPayment(order.ID, order.Total)
    if err != nil {
        op.inventorySvc.ReleaseItems(order.ID)
        op.orderRepo.UpdateStatus(order.ID, "failed")
        
        failedEvent := Event{
            Type:      "payment.failed",
            Payload:   order,
            Timestamp: time.Now(),
        }
        op.eventBus.Publish(failedEvent)
        return
    }
    
    // Payment successful
    completedEvent := Event{
        Type: "payment.completed",
        Payload: map[string]interface{}{
            "order":         order,
            "transactionID": transactionID,
        },
        Timestamp: time.Now(),
    }
    op.eventBus.Publish(completedEvent)
}

func (op *OrderProcessor) handlePaymentCompleted(event Event) {
    payload := event.Payload.(map[string]interface{})
    order := payload["order"].(Order)
    transactionID := payload["transactionID"].(string)
    
    op.orderRepo.UpdateStatus(order.ID, "completed")
    op.notificationSvc.SendOrderConfirmation(order.ID)
    op.notificationSvc.SendPaymentConfirmation(order.ID, transactionID)
}

func (op *OrderProcessor) handlePaymentFailed(event Event) {
    order := event.Payload.(Order)
    op.notificationSvc.SendOrderConfirmation(order.ID)
}

// Mock implementations for testing
type MockOrderRepository struct {
    orders map[string]Order
    mutex  sync.RWMutex
}

func NewMockOrderRepository() *MockOrderRepository {
    return &MockOrderRepository{orders: make(map[string]Order)}
}

func (mor *MockOrderRepository) Save(order Order) error {
    mor.mutex.Lock()
    defer mor.mutex.Unlock()
    mor.orders[order.ID] = order
    return nil
}

func (mor *MockOrderRepository) FindByID(id string) (Order, error) {
    mor.mutex.RLock()
    defer mor.mutex.RUnlock()
    
    order, exists := mor.orders[id]
    if !exists {
        return Order{}, fmt.Errorf("order not found")
    }
    return order, nil
}

func (mor *MockOrderRepository) UpdateStatus(id string, status string) error {
    mor.mutex.Lock()
    defer mor.mutex.Unlock()
    
    order, exists := mor.orders[id]
    if !exists {
        return fmt.Errorf("order not found")
    }
    
    order.Status = status
    mor.orders[id] = order
    return nil
}

type MockPaymentService struct {
    shouldFail bool
}

func NewMockPaymentService() *MockPaymentService {
    return &MockPaymentService{}
}

func (mps *MockPaymentService) ProcessPayment(orderID string, amount float64) (string, error) {
    if mps.shouldFail {
        return "", fmt.Errorf("payment failed")
    }
    return fmt.Sprintf("txn_%s", orderID), nil
}

func (mps *MockPaymentService) SetShouldFail(shouldFail bool) {
    mps.shouldFail = shouldFail
}

type MockInventoryService struct {
    reservations map[string][]OrderItem
    shouldFail   bool
    mutex        sync.RWMutex
}

func NewMockInventoryService() *MockInventoryService {
    return &MockInventoryService{reservations: make(map[string][]OrderItem)}
}

func (mis *MockInventoryService) ReserveItems(orderID string, items []OrderItem) error {
    if mis.shouldFail {
        return fmt.Errorf("insufficient inventory")
    }
    
    mis.mutex.Lock()
    defer mis.mutex.Unlock()
    mis.reservations[orderID] = items
    return nil
}

func (mis *MockInventoryService) ReleaseItems(orderID string) error {
    mis.mutex.Lock()
    defer mis.mutex.Unlock()
    delete(mis.reservations, orderID)
    return nil
}

func (mis *MockInventoryService) SetShouldFail(shouldFail bool) {
    mis.shouldFail = shouldFail
}

type MockNotificationService struct {
    orderConfirmations   []string
    paymentConfirmations map[string]string
    mutex                sync.RWMutex
}

func NewMockNotificationService() *MockNotificationService {
    return &MockNotificationService{
        paymentConfirmations: make(map[string]string),
    }
}

func (mns *MockNotificationService) SendOrderConfirmation(orderID string) error {
    mns.mutex.Lock()
    defer mns.mutex.Unlock()
    mns.orderConfirmations = append(mns.orderConfirmations, orderID)
    return nil
}

func (mns *MockNotificationService) SendPaymentConfirmation(orderID string, transactionID string) error {
    mns.mutex.Lock()
    defer mns.mutex.Unlock()
    mns.paymentConfirmations[orderID] = transactionID
    return nil
}

func TestOrderProcessingIntegration(t *testing.T) {
    // Setup components
    eventBus := NewEventBus()
    orderRepo := NewMockOrderRepository()
    paymentSvc := NewMockPaymentService()
    inventorySvc := NewMockInventoryService()
    notificationSvc := NewMockNotificationService()
    
    processor := NewOrderProcessor(eventBus, orderRepo, paymentSvc, inventorySvc, notificationSvc)
    
    t.Run("SuccessfulOrderProcessing", func(t *testing.T) {
        order := Order{
            ID:       "order-001",
            Customer: "alice",
            Items: []OrderItem{
                {ProductID: "prod-1", Quantity: 2, Price: 10.00},
                {ProductID: "prod-2", Quantity: 1, Price: 15.00},
            },
            Total: 35.00,
        }
        
        err := processor.ProcessOrder(order)
        if err != nil {
            t.Fatalf("Failed to process order: %v", err)
        }
        
        // Wait for async processing
        time.Sleep(100 * time.Millisecond)
        
        // Verify order status
        savedOrder, err := orderRepo.FindByID("order-001")
        if err != nil {
            t.Fatalf("Failed to find order: %v", err)
        }
        
        if savedOrder.Status != "completed" {
            t.Errorf("Expected order status 'completed', got %q", savedOrder.Status)
        }
        
        // Verify notifications were sent
        if len(notificationSvc.orderConfirmations) != 1 {
            t.Errorf("Expected 1 order confirmation, got %d", len(notificationSvc.orderConfirmations))
        }
        
        if len(notificationSvc.paymentConfirmations) != 1 {
            t.Errorf("Expected 1 payment confirmation, got %d", len(notificationSvc.paymentConfirmations))
        }
    })
    
    t.Run("PaymentFailure", func(t *testing.T) {
        paymentSvc.SetShouldFail(true)
        defer paymentSvc.SetShouldFail(false)
        
        order := Order{
            ID:       "order-002",
            Customer: "bob",
            Items: []OrderItem{
                {ProductID: "prod-3", Quantity: 1, Price: 20.00},
            },
            Total: 20.00,
        }
        
        err := processor.ProcessOrder(order)
        if err != nil {
            t.Fatalf("Failed to process order: %v", err)
        }
        
        // Wait for async processing
        time.Sleep(100 * time.Millisecond)
        
        // Verify order status is failed
        savedOrder, err := orderRepo.FindByID("order-002")
        if err != nil {
            t.Fatalf("Failed to find order: %v", err)
        }
        
        if savedOrder.Status != "failed" {
            t.Errorf("Expected order status 'failed', got %q", savedOrder.Status)
        }
    })
    
    t.Run("InventoryFailure", func(t *testing.T) {
        inventorySvc.SetShouldFail(true)
        defer inventorySvc.SetShouldFail(false)
        
        order := Order{
            ID:       "order-003",
            Customer: "charlie",
            Items: []OrderItem{
                {ProductID: "prod-4", Quantity: 10, Price: 5.00},
            },
            Total: 50.00,
        }
        
        err := processor.ProcessOrder(order)
        if err != nil {
            t.Fatalf("Failed to process order: %v", err)
        }
        
        // Wait for async processing
        time.Sleep(100 * time.Millisecond)
        
        // Verify order status is failed
        savedOrder, err := orderRepo.FindByID("order-003")
        if err != nil {
            t.Fatalf("Failed to find order: %v", err)
        }
        
        if savedOrder.Status != "failed" {
            t.Errorf("Expected order status 'failed', got %q", savedOrder.Status)
        }
    })
}
```

Integration tests validate complex workflows and component interactions,  
ensuring proper system behavior across multiple services. These tests verify  
end-to-end functionality including error handling and recovery scenarios.  

## Fuzz testing basics

Fuzz testing automatically generates test inputs to discover edge cases  
and potential vulnerabilities through systematic input variation.  

```go
package main

import (
    "fmt"
    "strings"
    "testing"
    "unicode"
)

// ParseEmail extracts username and domain from email address
func ParseEmail(email string) (string, string, error) {
    if email == "" {
        return "", "", fmt.Errorf("email cannot be empty")
    }
    
    parts := strings.Split(email, "@")
    if len(parts) != 2 {
        return "", "", fmt.Errorf("invalid email format")
    }
    
    username := strings.TrimSpace(parts[0])
    domain := strings.TrimSpace(parts[1])
    
    if username == "" {
        return "", "", fmt.Errorf("username cannot be empty")
    }
    
    if domain == "" {
        return "", "", fmt.Errorf("domain cannot be empty")
    }
    
    return username, domain, nil
}

// SanitizeInput removes dangerous characters from user input
func SanitizeInput(input string) string {
    var result strings.Builder
    
    for _, r := range input {
        if unicode.IsLetter(r) || unicode.IsDigit(r) || unicode.IsSpace(r) {
            result.WriteRune(r)
        } else if r == '.' || r == '-' || r == '_' {
            result.WriteRune(r)
        }
        // Skip other characters
    }
    
    return strings.TrimSpace(result.String())
}

// ValidatePassword checks password strength
func ValidatePassword(password string) []string {
    var issues []string
    
    if len(password) < 8 {
        issues = append(issues, "password must be at least 8 characters")
    }
    
    if len(password) > 128 {
        issues = append(issues, "password must be at most 128 characters")
    }
    
    hasUpper := false
    hasLower := false
    hasDigit := false
    hasSpecial := false
    
    for _, r := range password {
        if unicode.IsUpper(r) {
            hasUpper = true
        } else if unicode.IsLower(r) {
            hasLower = true
        } else if unicode.IsDigit(r) {
            hasDigit = true
        } else if unicode.IsPunct(r) || unicode.IsSymbol(r) {
            hasSpecial = true
        }
    }
    
    if !hasUpper {
        issues = append(issues, "password must contain uppercase letter")
    }
    if !hasLower {
        issues = append(issues, "password must contain lowercase letter")
    }
    if !hasDigit {
        issues = append(issues, "password must contain digit")
    }
    if !hasSpecial {
        issues = append(issues, "password must contain special character")
    }
    
    return issues
}

// Traditional tests
func TestParseEmail(t *testing.T) {
    tests := []struct {
        name           string
        email          string
        expectedUser   string
        expectedDomain string
        expectError    bool
    }{
        {"Valid email", "user@example.com", "user", "example.com", false},
        {"Empty email", "", "", "", true},
        {"No @ symbol", "userexample.com", "", "", true},
        {"Multiple @ symbols", "user@sub@example.com", "", "", true},
        {"Empty username", "@example.com", "", "", true},
        {"Empty domain", "user@", "", "", true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            user, domain, err := ParseEmail(tt.email)
            
            if tt.expectError {
                if err == nil {
                    t.Errorf("Expected error for email %q", tt.email)
                }
            } else {
                if err != nil {
                    t.Errorf("Unexpected error for email %q: %v", tt.email, err)
                }
                if user != tt.expectedUser {
                    t.Errorf("Expected user %q, got %q", tt.expectedUser, user)
                }
                if domain != tt.expectedDomain {
                    t.Errorf("Expected domain %q, got %q", tt.expectedDomain, domain)
                }
            }
        })
    }
}

// Fuzz tests (Go 1.18+)
func FuzzParseEmail(f *testing.F) {
    // Seed with known inputs
    f.Add("user@example.com")
    f.Add("test@test.org")
    f.Add("")
    f.Add("@")
    f.Add("user@")
    f.Add("@domain.com")
    
    f.Fuzz(func(t *testing.T, email string) {
        user, domain, err := ParseEmail(email)
        
        // Properties that should always hold
        if err == nil {
            // If no error, both user and domain should be non-empty
            if user == "" {
                t.Errorf("ParseEmail(%q) returned empty user with no error", email)
            }
            if domain == "" {
                t.Errorf("ParseEmail(%q) returned empty domain with no error", email)
            }
            
            // Reconstructed email should contain original components
            if !strings.Contains(email, user) {
                t.Errorf("ParseEmail(%q) returned user %q not found in original", email, user)
            }
            if !strings.Contains(email, domain) {
                t.Errorf("ParseEmail(%q) returned domain %q not found in original", email, domain)
            }
        }
        
        // Function should not panic
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("ParseEmail(%q) panicked: %v", email, r)
            }
        }()
    })
}

func FuzzSanitizeInput(f *testing.F) {
    // Seed with various inputs
    f.Add("hello world")
    f.Add("user@example.com")
    f.Add("test123!@#$%^&*()")
    f.Add("")
    f.Add("русский текст")
    f.Add("🎉🎊")
    
    f.Fuzz(func(t *testing.T, input string) {
        result := SanitizeInput(input)
        
        // Properties that should always hold
        // 1. Result should never be longer than input
        if len(result) > len(input) {
            t.Errorf("SanitizeInput(%q) result longer than input", input)
        }
        
        // 2. Result should only contain safe characters
        for _, r := range result {
            if !unicode.IsLetter(r) && !unicode.IsDigit(r) && !unicode.IsSpace(r) &&
                r != '.' && r != '-' && r != '_' {
                t.Errorf("SanitizeInput(%q) contains unsafe character %q", input, r)
            }
        }
        
        // 3. Function should not panic
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("SanitizeInput(%q) panicked: %v", input, r)
            }
        }()
    })
}

func FuzzValidatePassword(f *testing.F) {
    // Seed with various password patterns
    f.Add("Password123!")
    f.Add("weak")
    f.Add("")
    f.Add("UPPERCASE123!")
    f.Add("lowercase123!")
    f.Add("NoDigits!")
    f.Add("NoSpecialChars123")
    
    f.Fuzz(func(t *testing.T, password string) {
        issues := ValidatePassword(password)
        
        // Properties that should always hold
        // 1. Issues should be meaningful
        for _, issue := range issues {
            if issue == "" {
                t.Errorf("ValidatePassword(%q) returned empty issue", password)
            }
        }
        
        // 2. If password is very long, should have length issue
        if len(password) > 128 {
            found := false
            for _, issue := range issues {
                if strings.Contains(issue, "128 characters") {
                    found = true
                    break
                }
            }
            if !found {
                t.Errorf("ValidatePassword(%q) missing length issue for long password", password)
            }
        }
        
        // 3. Function should not panic
        defer func() {
            if r := recover(); r != nil {
                t.Errorf("ValidatePassword(%q) panicked: %v", password, r)
            }
        }()
    })
}
```

Fuzz testing discovers edge cases by generating random inputs and validating  
that functions maintain their invariants. This approach helps identify bugs  
that traditional testing might miss through systematic input exploration.  

## Property-based testing

Property-based testing validates that functions maintain specific properties  
across a wide range of inputs, focusing on behavior rather than specific values.  

```go
package main

import (
    "math/rand"
    "sort"
    "testing"
    "time"
)

// MathUtils provides mathematical utility functions
type MathUtils struct{}

// Reverse reverses a slice in place
func (mu *MathUtils) Reverse(slice []int) {
    for i, j := 0, len(slice)-1; i < j; i, j = i+1, j-1 {
        slice[i], slice[j] = slice[j], slice[i]
    }
}

// Unique removes duplicate elements from sorted slice
func (mu *MathUtils) Unique(slice []int) []int {
    if len(slice) <= 1 {
        return slice
    }
    
    result := make([]int, 0, len(slice))
    result = append(result, slice[0])
    
    for i := 1; i < len(slice); i++ {
        if slice[i] != slice[i-1] {
            result = append(result, slice[i])
        }
    }
    
    return result
}

// Max finds the maximum value in a slice
func (mu *MathUtils) Max(slice []int) (int, bool) {
    if len(slice) == 0 {
        return 0, false
    }
    
    max := slice[0]
    for _, v := range slice[1:] {
        if v > max {
            max = v
        }
    }
    
    return max, true
}

// Min finds the minimum value in a slice
func (mu *MathUtils) Min(slice []int) (int, bool) {
    if len(slice) == 0 {
        return 0, false
    }
    
    min := slice[0]
    for _, v := range slice[1:] {
        if v < min {
            min = v
        }
    }
    
    return min, true
}

// Sum calculates the sum of all elements
func (mu *MathUtils) Sum(slice []int) int {
    sum := 0
    for _, v := range slice {
        sum += v
    }
    return sum
}

// Property-based test helpers
func generateRandomSlice(size int, maxValue int) []int {
    slice := make([]int, size)
    for i := 0; i < size; i++ {
        slice[i] = rand.Intn(maxValue*2) - maxValue
    }
    return slice
}

func contains(slice []int, value int) bool {
    for _, v := range slice {
        if v == value {
            return true
        }
    }
    return false
}

func TestMathUtilsProperties(t *testing.T) {
    rand.Seed(time.Now().UnixNano())
    mu := &MathUtils{}
    
    t.Run("ReverseProperties", func(t *testing.T) {
        for i := 0; i < 100; i++ {
            size := rand.Intn(50)
            original := generateRandomSlice(size, 100)
            
            // Make a copy for testing
            slice := make([]int, len(original))
            copy(slice, original)
            
            // Property 1: Reversing twice should give original
            mu.Reverse(slice)
            mu.Reverse(slice)
            
            if len(slice) != len(original) {
                t.Errorf("Iteration %d: Length changed after double reverse", i)
                continue
            }
            
            for j := 0; j < len(original); j++ {
                if slice[j] != original[j] {
                    t.Errorf("Iteration %d: Double reverse failed at index %d", i, j)
                    break
                }
            }
            
            // Property 2: All elements should still be present
            copy(slice, original)
            mu.Reverse(slice)
            
            for _, orig := range original {
                if !contains(slice, orig) {
                    t.Errorf("Iteration %d: Element %d missing after reverse", i, orig)
                }
            }
            
            // Property 3: Length should remain the same
            if len(slice) != len(original) {
                t.Errorf("Iteration %d: Length changed after reverse", i)
            }
        }
    })
    
    t.Run("UniqueProperties", func(t *testing.T) {
        for i := 0; i < 100; i++ {
            size := rand.Intn(50)
            slice := generateRandomSlice(size, 20) // Smaller range for duplicates
            sort.Ints(slice)
            
            unique := mu.Unique(slice)
            
            // Property 1: Result should be sorted (input was sorted)
            for j := 1; j < len(unique); j++ {
                if unique[j] <= unique[j-1] {
                    t.Errorf("Iteration %d: Unique result not sorted", i)
                    break
                }
            }
            
            // Property 2: No duplicates in result
            seen := make(map[int]bool)
            for _, v := range unique {
                if seen[v] {
                    t.Errorf("Iteration %d: Duplicate %d in unique result", i, v)
                }
                seen[v] = true
            }
            
            // Property 3: All unique elements from original should be present
            originalUnique := make(map[int]bool)
            for _, v := range slice {
                originalUnique[v] = true
            }
            
            if len(unique) != len(originalUnique) {
                t.Errorf("Iteration %d: Unique count mismatch", i)
            }
            
            // Property 4: Result should not be longer than input
            if len(unique) > len(slice) {
                t.Errorf("Iteration %d: Unique result longer than input", i)
            }
        }
    })
    
    t.Run("MaxMinProperties", func(t *testing.T) {
        for i := 0; i < 100; i++ {
            size := rand.Intn(50) + 1 // At least 1 element
            slice := generateRandomSlice(size, 100)
            
            max, maxExists := mu.Max(slice)
            min, minExists := mu.Min(slice)
            
            // Property 1: Max and min should exist for non-empty slice
            if !maxExists || !minExists {
                t.Errorf("Iteration %d: Max or min doesn't exist for non-empty slice", i)
                continue
            }
            
            // Property 2: Max should be >= all elements
            for _, v := range slice {
                if v > max {
                    t.Errorf("Iteration %d: Found element %d > max %d", i, v, max)
                }
            }
            
            // Property 3: Min should be <= all elements
            for _, v := range slice {
                if v < min {
                    t.Errorf("Iteration %d: Found element %d < min %d", i, v, min)
                }
            }
            
            // Property 4: Max should be >= min
            if max < min {
                t.Errorf("Iteration %d: Max %d < min %d", i, max, min)
            }
            
            // Property 5: Max and min should be in the original slice
            if !contains(slice, max) {
                t.Errorf("Iteration %d: Max %d not in original slice", i, max)
            }
            if !contains(slice, min) {
                t.Errorf("Iteration %d: Min %d not in original slice", i, min)
            }
        }
    })
    
    t.Run("SumProperties", func(t *testing.T) {
        for i := 0; i < 100; i++ {
            size := rand.Intn(50)
            slice := generateRandomSlice(size, 10) // Small values to avoid overflow
            
            sum := mu.Sum(slice)
            
            // Property 1: Sum of empty slice should be 0
            if len(slice) == 0 && sum != 0 {
                t.Errorf("Iteration %d: Sum of empty slice should be 0, got %d", i, sum)
                continue
            }
            
            // Property 2: Adding an element should increase sum by that element
            if len(slice) > 0 {
                newElement := rand.Intn(20) - 10
                newSlice := append(slice, newElement)
                newSum := mu.Sum(newSlice)
                
                if newSum != sum+newElement {
                    t.Errorf("Iteration %d: Sum property violated: %d != %d + %d", 
                        i, newSum, sum, newElement)
                }
            }
            
            // Property 3: Sum should equal manual calculation
            manualSum := 0
            for _, v := range slice {
                manualSum += v
            }
            
            if sum != manualSum {
                t.Errorf("Iteration %d: Sum %d != manual sum %d", i, sum, manualSum)
            }
        }
    })
    
    t.Run("EmptySliceProperties", func(t *testing.T) {
        emptySlice := []int{}
        
        // Property: Max and Min should return false for empty slice
        _, maxExists := mu.Max(emptySlice)
        _, minExists := mu.Min(emptySlice)
        
        if maxExists {
            t.Error("Max should not exist for empty slice")
        }
        if minExists {
            t.Error("Min should not exist for empty slice")
        }
        
        // Property: Sum of empty slice should be 0
        sum := mu.Sum(emptySlice)
        if sum != 0 {
            t.Errorf("Sum of empty slice should be 0, got %d", sum)
        }
        
        // Property: Unique of empty slice should be empty
        unique := mu.Unique(emptySlice)
        if len(unique) != 0 {
            t.Errorf("Unique of empty slice should be empty, got %v", unique)
        }
        
        // Property: Reverse of empty slice should remain empty
        mu.Reverse(emptySlice)
        if len(emptySlice) != 0 {
            t.Errorf("Reverse of empty slice should remain empty, got %v", emptySlice)
        }
    })
}
```

Property-based testing validates that functions maintain mathematical and  
logical properties across diverse inputs. This approach discovers bugs by  
checking invariants rather than specific input-output pairs.  

## Performance regression testing

Performance regression testing identifies when code changes negatively impact  
performance, ensuring system performance remains within acceptable bounds.  

```go
package main

import (
    "crypto/md5"
    "crypto/sha256"
    "fmt"
    "hash"
    "testing"
    "time"
)

// HashingService provides various hashing algorithms
type HashingService struct {
    algorithm string
}

// NewHashingService creates a hashing service
func NewHashingService(algorithm string) *HashingService {
    return &HashingService{algorithm: algorithm}
}

// Hash computes hash using configured algorithm
func (hs *HashingService) Hash(data []byte) []byte {
    var hasher hash.Hash
    
    switch hs.algorithm {
    case "md5":
        hasher = md5.New()
    case "sha256":
        hasher = sha256.New()
    default:
        hasher = sha256.New()
    }
    
    hasher.Write(data)
    return hasher.Sum(nil)
}

// HashString computes hash of string
func (hs *HashingService) HashString(data string) string {
    hash := hs.Hash([]byte(data))
    return fmt.Sprintf("%x", hash)
}

// BatchHash processes multiple inputs
func (hs *HashingService) BatchHash(inputs []string) []string {
    results := make([]string, len(inputs))
    for i, input := range inputs {
        results[i] = hs.HashString(input)
    }
    return results
}

// ParallelBatchHash processes inputs concurrently
func (hs *HashingService) ParallelBatchHash(inputs []string) []string {
    results := make([]string, len(inputs))
    done := make(chan struct{})
    
    for i, input := range inputs {
        go func(index int, data string) {
            results[index] = hs.HashString(data)
            done <- struct{}{}
        }(i, input)
    }
    
    // Wait for all goroutines
    for i := 0; i < len(inputs); i++ {
        <-done
    }
    
    return results
}

// Performance baseline benchmarks
func BenchmarkHashingSingleOperation(b *testing.B) {
    service := NewHashingService("sha256")
    data := "hello there this is test data for hashing performance"
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        service.HashString(data)
    }
}

func BenchmarkHashingBatchOperation(b *testing.B) {
    service := NewHashingService("sha256")
    inputs := make([]string, 100)
    for i := 0; i < 100; i++ {
        inputs[i] = fmt.Sprintf("test data %d for batch hashing performance", i)
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        service.BatchHash(inputs)
    }
}

func BenchmarkHashingParallelBatch(b *testing.B) {
    service := NewHashingService("sha256")
    inputs := make([]string, 100)
    for i := 0; i < 100; i++ {
        inputs[i] = fmt.Sprintf("test data %d for parallel hashing performance", i)
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        service.ParallelBatchHash(inputs)
    }
}

// Algorithm comparison benchmarks
func BenchmarkHashingAlgorithms(b *testing.B) {
    data := "performance test data for algorithm comparison benchmarking"
    
    algorithms := []string{"md5", "sha256"}
    
    for _, algo := range algorithms {
        b.Run(algo, func(b *testing.B) {
            service := NewHashingService(algo)
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                service.HashString(data)
            }
        })
    }
}

// Data size impact benchmarks
func BenchmarkHashingDataSizes(b *testing.B) {
    service := NewHashingService("sha256")
    sizes := []int{100, 1000, 10000, 100000}
    
    for _, size := range sizes {
        data := make([]byte, size)
        for i := 0; i < size; i++ {
            data[i] = byte(i % 256)
        }
        
        b.Run(fmt.Sprintf("Size-%d", size), func(b *testing.B) {
            b.SetBytes(int64(size))
            b.ResetTimer()
            
            for i := 0; i < b.N; i++ {
                service.Hash(data)
            }
        })
    }
}

// Memory allocation benchmarks
func BenchmarkHashingMemoryUsage(b *testing.B) {
    service := NewHashingService("sha256")
    data := "memory usage test data for performance regression analysis"
    
    b.ReportAllocs()
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        service.HashString(data)
    }
}

func BenchmarkBatchMemoryUsage(b *testing.B) {
    service := NewHashingService("sha256")
    inputs := make([]string, 50)
    for i := 0; i < 50; i++ {
        inputs[i] = fmt.Sprintf("batch memory test data item %d", i)
    }
    
    b.ReportAllocs()
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        service.BatchHash(inputs)
    }
}

// Performance regression test with thresholds
func TestPerformanceRegression(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping performance regression tests in short mode")
    }
    
    service := NewHashingService("sha256")
    
    t.Run("SingleHashPerformance", func(t *testing.T) {
        data := "performance regression test data"
        iterations := 10000
        
        start := time.Now()
        for i := 0; i < iterations; i++ {
            service.HashString(data)
        }
        duration := time.Since(start)
        
        // Performance threshold: should complete 10k hashes in under 100ms
        threshold := 100 * time.Millisecond
        if duration > threshold {
            t.Errorf("Performance regression: %d hashes took %v, expected under %v", 
                iterations, duration, threshold)
        }
        
        t.Logf("Performance: %d hashes completed in %v", iterations, duration)
    })
    
    t.Run("BatchHashPerformance", func(t *testing.T) {
        inputs := make([]string, 1000)
        for i := 0; i < 1000; i++ {
            inputs[i] = fmt.Sprintf("batch test data %d", i)
        }
        
        start := time.Now()
        results := service.BatchHash(inputs)
        duration := time.Since(start)
        
        if len(results) != len(inputs) {
            t.Errorf("Expected %d results, got %d", len(inputs), len(results))
        }
        
        // Performance threshold: should complete 1k batch hashes in under 50ms
        threshold := 50 * time.Millisecond
        if duration > threshold {
            t.Errorf("Batch performance regression: took %v, expected under %v", 
                duration, threshold)
        }
        
        t.Logf("Batch performance: %d hashes completed in %v", len(inputs), duration)
    })
    
    t.Run("ParallelPerformanceComparison", func(t *testing.T) {
        inputs := make([]string, 200)
        for i := 0; i < 200; i++ {
            inputs[i] = fmt.Sprintf("parallel test data %d", i)
        }
        
        // Sequential timing
        start := time.Now()
        seqResults := service.BatchHash(inputs)
        seqDuration := time.Since(start)
        
        // Parallel timing
        start = time.Now()
        parResults := service.ParallelBatchHash(inputs)
        parDuration := time.Since(start)
        
        // Verify results are equivalent
        if len(seqResults) != len(parResults) {
            t.Errorf("Result length mismatch: seq=%d, par=%d", 
                len(seqResults), len(parResults))
        }
        
        // Parallel should be faster (with enough work)
        if parDuration > seqDuration {
            t.Logf("Warning: Parallel (%v) slower than sequential (%v)", 
                parDuration, seqDuration)
        } else {
            speedup := float64(seqDuration) / float64(parDuration)
            t.Logf("Parallel speedup: %.2fx (seq=%v, par=%v)", 
                speedup, seqDuration, parDuration)
        }
    })
    
    t.Run("MemoryUsageRegression", func(t *testing.T) {
        data := "memory regression test data"
        
        // Measure memory before
        var memBefore, memAfter uint64
        
        // Warm up
        for i := 0; i < 100; i++ {
            service.HashString(data)
        }
        
        // This is a simplified memory check
        // In practice, you might use runtime.ReadMemStats
        start := time.Now()
        for i := 0; i < 1000; i++ {
            service.HashString(data)
        }
        duration := time.Since(start)
        
        // Ensure operations complete within reasonable time
        // indicating no memory-related performance issues
        maxDuration := 10 * time.Millisecond
        if duration > maxDuration {
            t.Errorf("Memory regression suspected: 1000 ops took %v, expected under %v", 
                duration, maxDuration)
        }
        
        t.Logf("Memory usage test: 1000 ops in %v (before: %d, after: %d)", 
            duration, memBefore, memAfter)
    })
}

// Concurrent performance testing
func TestConcurrentPerformance(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping concurrent performance tests in short mode")
    }
    
    service := NewHashingService("sha256")
    
    t.Run("ConcurrentHashingLoad", func(t *testing.T) {
        numGoroutines := 10
        operationsPerGoroutine := 100
        data := "concurrent performance test data"
        
        start := time.Now()
        done := make(chan bool, numGoroutines)
        
        for i := 0; i < numGoroutines; i++ {
            go func() {
                defer func() { done <- true }()
                
                for j := 0; j < operationsPerGoroutine; j++ {
                    service.HashString(data)
                }
            }()
        }
        
        // Wait for all goroutines
        for i := 0; i < numGoroutines; i++ {
            <-done
        }
        
        duration := time.Since(start)
        totalOps := numGoroutines * operationsPerGoroutine
        
        // Performance threshold for concurrent operations
        maxDuration := 500 * time.Millisecond
        if duration > maxDuration {
            t.Errorf("Concurrent performance regression: %d ops took %v, expected under %v", 
                totalOps, duration, maxDuration)
        }
        
        opsPerSecond := float64(totalOps) / duration.Seconds()
        t.Logf("Concurrent performance: %d ops in %v (%.0f ops/sec)", 
            totalOps, duration, opsPerSecond)
    })
}
```

Performance regression testing establishes performance baselines and detects  
when changes negatively impact system performance. These tests use time  
thresholds and comparative analysis to identify performance degradation.  

## Test doubles and stubs

Test doubles replace dependencies with controlled implementations, enabling  
isolated testing and predictable behavior verification.  

```go
package main

import (
    "fmt"
    "testing"
    "time"
)

// EmailService sends email notifications
type EmailService interface {
    SendEmail(to, subject, body string) error
    GetSentCount() int
}

// SMSService sends SMS notifications
type SMSService interface {
    SendSMS(to, message string) error
    GetDeliveryStatus(messageID string) string
}

// LoggingService handles application logging
type LoggingService interface {
    Log(level, message string)
    GetLogs() []string
}

// NotificationManager orchestrates different notification types
type NotificationManager struct {
    emailService EmailService
    smsService   SMSService
    logger       LoggingService
}

// NewNotificationManager creates a notification manager
func NewNotificationManager(email EmailService, sms SMSService, logger LoggingService) *NotificationManager {
    return &NotificationManager{
        emailService: email,
        smsService:   sms,
        logger:       logger,
    }
}

// SendWelcomeNotification sends welcome message via multiple channels
func (nm *NotificationManager) SendWelcomeNotification(userEmail, userPhone, userName string) error {
    nm.logger.Log("INFO", fmt.Sprintf("Sending welcome notification to %s", userName))
    
    // Send email
    emailSubject := "Welcome to our service!"
    emailBody := fmt.Sprintf("Hello there %s! Welcome to our amazing service.", userName)
    
    if err := nm.emailService.SendEmail(userEmail, emailSubject, emailBody); err != nil {
        nm.logger.Log("ERROR", fmt.Sprintf("Failed to send welcome email: %v", err))
        return fmt.Errorf("email sending failed: %w", err)
    }
    
    // Send SMS
    smsMessage := fmt.Sprintf("Hello there %s! Thanks for joining us.", userName)
    if err := nm.smsService.SendSMS(userPhone, smsMessage); err != nil {
        nm.logger.Log("ERROR", fmt.Sprintf("Failed to send welcome SMS: %v", err))
        return fmt.Errorf("SMS sending failed: %w", err)
    }
    
    nm.logger.Log("INFO", fmt.Sprintf("Welcome notification sent successfully to %s", userName))
    return nil
}

// GetNotificationStats returns notification statistics
func (nm *NotificationManager) GetNotificationStats() map[string]int {
    return map[string]int{
        "emails_sent": nm.emailService.GetSentCount(),
    }
}

// Stub implementations for testing
type EmailStub struct {
    sentEmails []EmailRecord
    shouldFail bool
    failureMessage string
}

type EmailRecord struct {
    To      string
    Subject string
    Body    string
    Sent    time.Time
}

func NewEmailStub() *EmailStub {
    return &EmailStub{
        sentEmails: make([]EmailRecord, 0),
    }
}

func (es *EmailStub) SendEmail(to, subject, body string) error {
    if es.shouldFail {
        return fmt.Errorf(es.failureMessage)
    }
    
    es.sentEmails = append(es.sentEmails, EmailRecord{
        To:      to,
        Subject: subject,
        Body:    body,
        Sent:    time.Now(),
    })
    
    return nil
}

func (es *EmailStub) GetSentCount() int {
    return len(es.sentEmails)
}

func (es *EmailStub) GetSentEmails() []EmailRecord {
    return es.sentEmails
}

func (es *EmailStub) SetShouldFail(shouldFail bool, message string) {
    es.shouldFail = shouldFail
    es.failureMessage = message
}

type SMSStub struct {
    sentMessages []SMSRecord
    shouldFail   bool
    failureMessage string
    deliveryStatus map[string]string
}

type SMSRecord struct {
    To      string
    Message string
    Sent    time.Time
    ID      string
}

func NewSMSStub() *SMSStub {
    return &SMSStub{
        sentMessages:   make([]SMSRecord, 0),
        deliveryStatus: make(map[string]string),
    }
}

func (ss *SMSStub) SendSMS(to, message string) error {
    if ss.shouldFail {
        return fmt.Errorf(ss.failureMessage)
    }
    
    messageID := fmt.Sprintf("msg_%d", len(ss.sentMessages)+1)
    ss.sentMessages = append(ss.sentMessages, SMSRecord{
        To:      to,
        Message: message,
        Sent:    time.Now(),
        ID:      messageID,
    })
    
    ss.deliveryStatus[messageID] = "delivered"
    return nil
}

func (ss *SMSStub) GetDeliveryStatus(messageID string) string {
    status, exists := ss.deliveryStatus[messageID]
    if !exists {
        return "unknown"
    }
    return status
}

func (ss *SMSStub) GetSentMessages() []SMSRecord {
    return ss.sentMessages
}

func (ss *SMSStub) SetShouldFail(shouldFail bool, message string) {
    ss.shouldFail = shouldFail
    ss.failureMessage = message
}

type LoggerStub struct {
    logs []LogRecord
}

type LogRecord struct {
    Level     string
    Message   string
    Timestamp time.Time
}

func NewLoggerStub() *LoggerStub {
    return &LoggerStub{
        logs: make([]LogRecord, 0),
    }
}

func (ls *LoggerStub) Log(level, message string) {
    ls.logs = append(ls.logs, LogRecord{
        Level:     level,
        Message:   message,
        Timestamp: time.Now(),
    })
}

func (ls *LoggerStub) GetLogs() []string {
    logs := make([]string, len(ls.logs))
    for i, log := range ls.logs {
        logs[i] = fmt.Sprintf("[%s] %s", log.Level, log.Message)
    }
    return logs
}

func (ls *LoggerStub) GetLogRecords() []LogRecord {
    return ls.logs
}

func (ls *LoggerStub) CountLogsByLevel(level string) int {
    count := 0
    for _, log := range ls.logs {
        if log.Level == level {
            count++
        }
    }
    return count
}

func TestNotificationManagerWithStubs(t *testing.T) {
    t.Run("SuccessfulWelcomeNotification", func(t *testing.T) {
        // Setup stubs
        emailStub := NewEmailStub()
        smsStub := NewSMSStub()
        loggerStub := NewLoggerStub()
        
        manager := NewNotificationManager(emailStub, smsStub, loggerStub)
        
        // Test successful notification
        err := manager.SendWelcomeNotification("alice@example.com", "+1234567890", "Alice")
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        // Verify email was sent
        if emailStub.GetSentCount() != 1 {
            t.Errorf("Expected 1 email sent, got %d", emailStub.GetSentCount())
        }
        
        sentEmails := emailStub.GetSentEmails()
        if len(sentEmails) > 0 {
            email := sentEmails[0]
            if email.To != "alice@example.com" {
                t.Errorf("Expected email to alice@example.com, got %s", email.To)
            }
            if email.Subject != "Welcome to our service!" {
                t.Errorf("Expected welcome subject, got %s", email.Subject)
            }
        }
        
        // Verify SMS was sent
        sentMessages := smsStub.GetSentMessages()
        if len(sentMessages) != 1 {
            t.Errorf("Expected 1 SMS sent, got %d", len(sentMessages))
        }
        
        if len(sentMessages) > 0 {
            sms := sentMessages[0]
            if sms.To != "+1234567890" {
                t.Errorf("Expected SMS to +1234567890, got %s", sms.To)
            }
        }
        
        // Verify logging
        if loggerStub.CountLogsByLevel("INFO") < 2 {
            t.Error("Expected at least 2 INFO log messages")
        }
        
        if loggerStub.CountLogsByLevel("ERROR") != 0 {
            t.Error("Expected no ERROR log messages for successful case")
        }
    })
    
    t.Run("EmailFailure", func(t *testing.T) {
        // Setup stubs with email failure
        emailStub := NewEmailStub()
        emailStub.SetShouldFail(true, "email server unavailable")
        
        smsStub := NewSMSStub()
        loggerStub := NewLoggerStub()
        
        manager := NewNotificationManager(emailStub, smsStub, loggerStub)
        
        // Test email failure
        err := manager.SendWelcomeNotification("bob@example.com", "+1234567891", "Bob")
        if err == nil {
            t.Error("Expected error when email fails")
        }
        
        // Verify error message
        expectedError := "email sending failed"
        if !contains(err.Error(), expectedError) {
            t.Errorf("Expected error to contain %q, got %q", expectedError, err.Error())
        }
        
        // Verify no emails were sent
        if emailStub.GetSentCount() != 0 {
            t.Errorf("Expected 0 emails sent, got %d", emailStub.GetSentCount())
        }
        
        // Verify error was logged
        if loggerStub.CountLogsByLevel("ERROR") == 0 {
            t.Error("Expected ERROR log message for email failure")
        }
    })
    
    t.Run("SMSFailure", func(t *testing.T) {
        // Setup stubs with SMS failure
        emailStub := NewEmailStub()
        smsStub := NewSMSStub()
        smsStub.SetShouldFail(true, "SMS gateway timeout")
        
        loggerStub := NewLoggerStub()
        
        manager := NewNotificationManager(emailStub, smsStub, loggerStub)
        
        // Test SMS failure
        err := manager.SendWelcomeNotification("charlie@example.com", "+1234567892", "Charlie")
        if err == nil {
            t.Error("Expected error when SMS fails")
        }
        
        // Verify email was sent (before SMS failure)
        if emailStub.GetSentCount() != 1 {
            t.Errorf("Expected 1 email sent before SMS failure, got %d", emailStub.GetSentCount())
        }
        
        // Verify SMS was not sent
        if len(smsStub.GetSentMessages()) != 0 {
            t.Errorf("Expected 0 SMS sent, got %d", len(smsStub.GetSentMessages()))
        }
        
        // Verify error was logged
        if loggerStub.CountLogsByLevel("ERROR") == 0 {
            t.Error("Expected ERROR log message for SMS failure")
        }
    })
    
    t.Run("NotificationStats", func(t *testing.T) {
        emailStub := NewEmailStub()
        smsStub := NewSMSStub()
        loggerStub := NewLoggerStub()
        
        manager := NewNotificationManager(emailStub, smsStub, loggerStub)
        
        // Send multiple notifications
        manager.SendWelcomeNotification("user1@example.com", "+1111111111", "User1")
        manager.SendWelcomeNotification("user2@example.com", "+2222222222", "User2")
        manager.SendWelcomeNotification("user3@example.com", "+3333333333", "User3")
        
        stats := manager.GetNotificationStats()
        
        expectedEmails := 3
        if stats["emails_sent"] != expectedEmails {
            t.Errorf("Expected %d emails in stats, got %d", expectedEmails, stats["emails_sent"])
        }
    })
}

func contains(str, substr string) bool {
    return len(str) >= len(substr) && str[:len(substr)] == substr || 
           (len(str) > len(substr) && str[len(str)-len(substr):] == substr) ||
           (len(str) > len(substr) && str[1:len(substr)+1] == substr)
}
```

Test doubles and stubs provide controlled, predictable behavior for testing  
components in isolation. They enable verification of interactions and behavior  
without external dependencies, improving test reliability and execution speed.  

## Database testing patterns

Database testing validates data persistence, queries, and transactions using  
test databases and transaction rollbacks for isolated testing.  

```go
package main

import (
    "database/sql"
    "fmt"
    "testing"
    "time"
    
    _ "github.com/mattn/go-sqlite3"
)

// User represents a user entity
type User struct {
    ID       int
    Username string
    Email    string
    Created  time.Time
    Active   bool
}

// UserRepository handles user data operations
type UserRepository struct {
    db *sql.DB
}

// NewUserRepository creates a user repository
func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

// CreateUser inserts a new user
func (ur *UserRepository) CreateUser(user User) (int, error) {
    query := `INSERT INTO users (username, email, created, active) VALUES (?, ?, ?, ?)`
    
    result, err := ur.db.Exec(query, user.Username, user.Email, user.Created, user.Active)
    if err != nil {
        return 0, fmt.Errorf("failed to create user: %w", err)
    }
    
    id, err := result.LastInsertId()
    if err != nil {
        return 0, fmt.Errorf("failed to get user ID: %w", err)
    }
    
    return int(id), nil
}

// GetUser retrieves a user by ID
func (ur *UserRepository) GetUser(id int) (User, error) {
    query := `SELECT id, username, email, created, active FROM users WHERE id = ?`
    
    var user User
    err := ur.db.QueryRow(query, id).Scan(
        &user.ID, &user.Username, &user.Email, &user.Created, &user.Active)
    
    if err != nil {
        if err == sql.ErrNoRows {
            return User{}, fmt.Errorf("user not found")
        }
        return User{}, fmt.Errorf("failed to get user: %w", err)
    }
    
    return user, nil
}

// GetUserByUsername retrieves a user by username
func (ur *UserRepository) GetUserByUsername(username string) (User, error) {
    query := `SELECT id, username, email, created, active FROM users WHERE username = ?`
    
    var user User
    err := ur.db.QueryRow(query, username).Scan(
        &user.ID, &user.Username, &user.Email, &user.Created, &user.Active)
    
    if err != nil {
        if err == sql.ErrNoRows {
            return User{}, fmt.Errorf("user not found")
        }
        return User{}, fmt.Errorf("failed to get user: %w", err)
    }
    
    return user, nil
}

// UpdateUser updates user information
func (ur *UserRepository) UpdateUser(user User) error {
    query := `UPDATE users SET username = ?, email = ?, active = ? WHERE id = ?`
    
    result, err := ur.db.Exec(query, user.Username, user.Email, user.Active, user.ID)
    if err != nil {
        return fmt.Errorf("failed to update user: %w", err)
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get affected rows: %w", err)
    }
    
    if rowsAffected == 0 {
        return fmt.Errorf("user not found")
    }
    
    return nil
}

// DeleteUser removes a user
func (ur *UserRepository) DeleteUser(id int) error {
    query := `DELETE FROM users WHERE id = ?`
    
    result, err := ur.db.Exec(query, id)
    if err != nil {
        return fmt.Errorf("failed to delete user: %w", err)
    }
    
    rowsAffected, err := result.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get affected rows: %w", err)
    }
    
    if rowsAffected == 0 {
        return fmt.Errorf("user not found")
    }
    
    return nil
}

// ListActiveUsers returns all active users
func (ur *UserRepository) ListActiveUsers() ([]User, error) {
    query := `SELECT id, username, email, created, active FROM users WHERE active = ? ORDER BY created DESC`
    
    rows, err := ur.db.Query(query, true)
    if err != nil {
        return nil, fmt.Errorf("failed to query users: %w", err)
    }
    defer rows.Close()
    
    var users []User
    for rows.Next() {
        var user User
        err := rows.Scan(&user.ID, &user.Username, &user.Email, &user.Created, &user.Active)
        if err != nil {
            return nil, fmt.Errorf("failed to scan user: %w", err)
        }
        users = append(users, user)
    }
    
    if err = rows.Err(); err != nil {
        return nil, fmt.Errorf("rows iteration error: %w", err)
    }
    
    return users, nil
}

// CountUsers returns the total number of users
func (ur *UserRepository) CountUsers() (int, error) {
    query := `SELECT COUNT(*) FROM users`
    
    var count int
    err := ur.db.QueryRow(query).Scan(&count)
    if err != nil {
        return 0, fmt.Errorf("failed to count users: %w", err)
    }
    
    return count, nil
}

// Test database setup helpers
func setupTestDB() (*sql.DB, error) {
    db, err := sql.Open("sqlite3", ":memory:")
    if err != nil {
        return nil, err
    }
    
    // Create users table
    createTable := `
    CREATE TABLE users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        username TEXT UNIQUE NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created DATETIME NOT NULL,
        active BOOLEAN NOT NULL
    )`
    
    _, err = db.Exec(createTable)
    if err != nil {
        db.Close()
        return nil, err
    }
    
    return db, nil
}

func teardownTestDB(db *sql.DB) {
    db.Close()
}

func createTestUser(username, email string) User {
    return User{
        Username: username,
        Email:    email,
        Created:  time.Now(),
        Active:   true,
    }
}

func TestUserRepository(t *testing.T) {
    db, err := setupTestDB()
    if err != nil {
        t.Fatalf("Failed to setup test database: %v", err)
    }
    defer teardownTestDB(db)
    
    repo := NewUserRepository(db)
    
    t.Run("CreateUser", func(t *testing.T) {
        user := createTestUser("alice", "alice@example.com")
        
        id, err := repo.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user: %v", err)
        }
        
        if id <= 0 {
            t.Errorf("Expected positive user ID, got %d", id)
        }
        
        // Verify user was created
        createdUser, err := repo.GetUser(id)
        if err != nil {
            t.Fatalf("Failed to get created user: %v", err)
        }
        
        if createdUser.Username != user.Username {
            t.Errorf("Expected username %s, got %s", user.Username, createdUser.Username)
        }
        
        if createdUser.Email != user.Email {
            t.Errorf("Expected email %s, got %s", user.Email, createdUser.Email)
        }
    })
    
    t.Run("GetUserNotFound", func(t *testing.T) {
        _, err := repo.GetUser(99999)
        if err == nil {
            t.Error("Expected error for non-existent user")
        }
        
        expectedMsg := "user not found"
        if err.Error() != expectedMsg {
            t.Errorf("Expected error %q, got %q", expectedMsg, err.Error())
        }
    })
    
    t.Run("GetUserByUsername", func(t *testing.T) {
        user := createTestUser("bob", "bob@example.com")
        
        id, err := repo.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user: %v", err)
        }
        
        foundUser, err := repo.GetUserByUsername("bob")
        if err != nil {
            t.Fatalf("Failed to get user by username: %v", err)
        }
        
        if foundUser.ID != id {
            t.Errorf("Expected user ID %d, got %d", id, foundUser.ID)
        }
        
        if foundUser.Username != "bob" {
            t.Errorf("Expected username 'bob', got %s", foundUser.Username)
        }
    })
    
    t.Run("UpdateUser", func(t *testing.T) {
        user := createTestUser("charlie", "charlie@example.com")
        
        id, err := repo.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user: %v", err)
        }
        
        // Update user
        updatedUser := User{
            ID:       id,
            Username: "charlie_updated",
            Email:    "charlie.updated@example.com",
            Active:   false,
        }
        
        err = repo.UpdateUser(updatedUser)
        if err != nil {
            t.Fatalf("Failed to update user: %v", err)
        }
        
        // Verify update
        retrievedUser, err := repo.GetUser(id)
        if err != nil {
            t.Fatalf("Failed to get updated user: %v", err)
        }
        
        if retrievedUser.Username != "charlie_updated" {
            t.Errorf("Expected updated username, got %s", retrievedUser.Username)
        }
        
        if retrievedUser.Active != false {
            t.Error("Expected user to be inactive")
        }
    })
    
    t.Run("DeleteUser", func(t *testing.T) {
        user := createTestUser("david", "david@example.com")
        
        id, err := repo.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user: %v", err)
        }
        
        // Delete user
        err = repo.DeleteUser(id)
        if err != nil {
            t.Fatalf("Failed to delete user: %v", err)
        }
        
        // Verify deletion
        _, err = repo.GetUser(id)
        if err == nil {
            t.Error("Expected error when getting deleted user")
        }
    })
    
    t.Run("ListActiveUsers", func(t *testing.T) {
        // Create test users
        users := []User{
            createTestUser("eve", "eve@example.com"),
            createTestUser("frank", "frank@example.com"),
        }
        
        // Make one user inactive
        users[1].Active = false
        
        for _, user := range users {
            _, err := repo.CreateUser(user)
            if err != nil {
                t.Fatalf("Failed to create test user: %v", err)
            }
        }
        
        activeUsers, err := repo.ListActiveUsers()
        if err != nil {
            t.Fatalf("Failed to list active users: %v", err)
        }
        
        // Should only get active users
        activeCount := 0
        for _, user := range activeUsers {
            if user.Active {
                activeCount++
            }
        }
        
        if activeCount != len(activeUsers) {
            t.Error("ListActiveUsers returned inactive users")
        }
    })
    
    t.Run("CountUsers", func(t *testing.T) {
        initialCount, err := repo.CountUsers()
        if err != nil {
            t.Fatalf("Failed to count users: %v", err)
        }
        
        // Create additional users
        user1 := createTestUser("grace", "grace@example.com")
        user2 := createTestUser("henry", "henry@example.com")
        
        repo.CreateUser(user1)
        repo.CreateUser(user2)
        
        finalCount, err := repo.CountUsers()
        if err != nil {
            t.Fatalf("Failed to count users after creation: %v", err)
        }
        
        expectedCount := initialCount + 2
        if finalCount != expectedCount {
            t.Errorf("Expected user count %d, got %d", expectedCount, finalCount)
        }
    })
}

func TestUserRepositoryTransactions(t *testing.T) {
    db, err := setupTestDB()
    if err != nil {
        t.Fatalf("Failed to setup test database: %v", err)
    }
    defer teardownTestDB(db)
    
    repo := NewUserRepository(db)
    
    t.Run("TransactionRollback", func(t *testing.T) {
        // Begin transaction
        tx, err := db.Begin()
        if err != nil {
            t.Fatalf("Failed to begin transaction: %v", err)
        }
        
        // Create repository with transaction
        txRepo := NewUserRepository(tx)
        
        user := createTestUser("transaction_test", "tx@example.com")
        id, err := txRepo.CreateUser(user)
        if err != nil {
            t.Fatalf("Failed to create user in transaction: %v", err)
        }
        
        // Rollback transaction
        err = tx.Rollback()
        if err != nil {
            t.Fatalf("Failed to rollback transaction: %v", err)
        }
        
        // Verify user was not persisted
        _, err = repo.GetUser(id)
        if err == nil {
            t.Error("Expected user to not exist after rollback")
        }
    })
}
```

Database testing validates data persistence, retrieval, and manipulation using  
in-memory databases for fast, isolated tests. Transaction rollbacks ensure  
test isolation and prevent side effects between test cases.  

## Context-aware testing

Context-aware testing validates timeout handling, cancellation behavior,  
and proper resource cleanup in concurrent and long-running operations.  

```go
package main

import (
    "context"
    "fmt"
    "testing"
    "time"
)

// WorkerService performs background operations with context support
type WorkerService struct {
    name string
}

// NewWorkerService creates a worker service
func NewWorkerService(name string) *WorkerService {
    return &WorkerService{name: name}
}

// ProcessTask performs a task with context cancellation support
func (ws *WorkerService) ProcessTask(ctx context.Context, taskID string, duration time.Duration) error {
    select {
    case <-time.After(duration):
        return nil
    case <-ctx.Done():
        return fmt.Errorf("task %s cancelled: %w", taskID, ctx.Err())
    }
}

// ProcessBatch processes multiple tasks concurrently
func (ws *WorkerService) ProcessBatch(ctx context.Context, taskIDs []string, taskDuration time.Duration) ([]string, error) {
    results := make([]string, len(taskIDs))
    errors := make(chan error, len(taskIDs))
    
    for i, taskID := range taskIDs {
        go func(index int, id string) {
            if err := ws.ProcessTask(ctx, id, taskDuration); err != nil {
                errors <- fmt.Errorf("task %d failed: %w", index, err)
                return
            }
            results[index] = fmt.Sprintf("completed_%s", id)
            errors <- nil
        }(i, taskID)
    }
    
    // Wait for all tasks or context cancellation
    for i := 0; i < len(taskIDs); i++ {
        select {
        case err := <-errors:
            if err != nil {
                return nil, err
            }
        case <-ctx.Done():
            return nil, fmt.Errorf("batch processing cancelled: %w", ctx.Err())
        }
    }
    
    return results, nil
}

// StreamProcessor processes data streams with context
func (ws *WorkerService) StreamProcessor(ctx context.Context, input <-chan string, output chan<- string) error {
    defer close(output)
    
    for {
        select {
        case data, ok := <-input:
            if !ok {
                return nil // Input stream closed
            }
            
            // Simulate processing
            processed := fmt.Sprintf("processed_%s", data)
            
            select {
            case output <- processed:
                // Successfully sent
            case <-ctx.Done():
                return fmt.Errorf("stream processing cancelled: %w", ctx.Err())
            }
            
        case <-ctx.Done():
            return fmt.Errorf("stream processing cancelled: %w", ctx.Err())
        }
    }
}

// RetryableOperation performs operation with retries and context
func (ws *WorkerService) RetryableOperation(ctx context.Context, maxRetries int, operation func() error) error {
    var lastErr error
    
    for attempt := 0; attempt <= maxRetries; attempt++ {
        select {
        case <-ctx.Done():
            return fmt.Errorf("operation cancelled after %d attempts: %w", attempt, ctx.Err())
        default:
        }
        
        if err := operation(); err != nil {
            lastErr = err
            if attempt < maxRetries {
                // Wait before retry
                select {
                case <-time.After(time.Millisecond * 10):
                    continue
                case <-ctx.Done():
                    return fmt.Errorf("operation cancelled during retry wait: %w", ctx.Err())
                }
            }
        } else {
            return nil
        }
    }
    
    return fmt.Errorf("operation failed after %d attempts: %w", maxRetries+1, lastErr)
}

func TestWorkerServiceContextHandling(t *testing.T) {
    ws := NewWorkerService("test-worker")
    
    t.Run("TaskWithinTimeout", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
        defer cancel()
        
        start := time.Now()
        err := ws.ProcessTask(ctx, "task1", 50*time.Millisecond)
        duration := time.Since(start)
        
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        // Verify task completed within expected time
        if duration > 80*time.Millisecond {
            t.Errorf("Task took too long: %v", duration)
        }
    })
    
    t.Run("TaskTimeoutCancellation", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
        defer cancel()
        
        start := time.Now()
        err := ws.ProcessTask(ctx, "task2", 100*time.Millisecond)
        duration := time.Since(start)
        
        if err == nil {
            t.Error("Expected error due to timeout")
        }
        
        // Verify cancellation happened around timeout
        if duration > 80*time.Millisecond {
            t.Errorf("Cancellation took too long: %v", duration)
        }
        
        // Verify error indicates cancellation
        if err.Error() != "task task2 cancelled: context deadline exceeded" {
            t.Errorf("Unexpected error message: %v", err)
        }
    })
    
    t.Run("ManualCancellation", func(t *testing.T) {
        ctx, cancel := context.WithCancel(context.Background())
        
        // Cancel after short delay
        go func() {
            time.Sleep(30 * time.Millisecond)
            cancel()
        }()
        
        start := time.Now()
        err := ws.ProcessTask(ctx, "task3", 100*time.Millisecond)
        duration := time.Since(start)
        
        if err == nil {
            t.Error("Expected error due to cancellation")
        }
        
        // Verify cancellation happened quickly
        if duration > 50*time.Millisecond {
            t.Errorf("Cancellation took too long: %v", duration)
        }
    })
    
    t.Run("BatchProcessingSuccess", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
        defer cancel()
        
        taskIDs := []string{"batch1", "batch2", "batch3"}
        results, err := ws.ProcessBatch(ctx, taskIDs, 30*time.Millisecond)
        
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        if len(results) != len(taskIDs) {
            t.Errorf("Expected %d results, got %d", len(taskIDs), len(results))
        }
        
        for i, result := range results {
            expected := fmt.Sprintf("completed_%s", taskIDs[i])
            if result != expected {
                t.Errorf("Expected result %s, got %s", expected, result)
            }
        }
    })
    
    t.Run("BatchProcessingTimeout", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
        defer cancel()
        
        taskIDs := []string{"slow1", "slow2"}
        _, err := ws.ProcessBatch(ctx, taskIDs, 100*time.Millisecond)
        
        if err == nil {
            t.Error("Expected error due to timeout")
        }
        
        // Verify error indicates cancellation
        expectedMsg := "batch processing cancelled"
        if !contains(err.Error(), expectedMsg) {
            t.Errorf("Expected error to contain %q, got %q", expectedMsg, err.Error())
        }
    })
}

func TestStreamProcessorContextHandling(t *testing.T) {
    ws := NewWorkerService("stream-worker")
    
    t.Run("StreamProcessingSuccess", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
        defer cancel()
        
        input := make(chan string, 3)
        output := make(chan string, 3)
        
        // Send test data
        input <- "data1"
        input <- "data2"
        input <- "data3"
        close(input)
        
        err := ws.StreamProcessor(ctx, input, output)
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        // Verify processed data
        var results []string
        for result := range output {
            results = append(results, result)
        }
        
        expected := []string{"processed_data1", "processed_data2", "processed_data3"}
        if len(results) != len(expected) {
            t.Errorf("Expected %d results, got %d", len(expected), len(results))
        }
    })
    
    t.Run("StreamProcessingCancellation", func(t *testing.T) {
        ctx, cancel := context.WithCancel(context.Background())
        
        input := make(chan string)
        output := make(chan string)
        
        // Start processing
        errChan := make(chan error, 1)
        go func() {
            errChan <- ws.StreamProcessor(ctx, input, output)
        }()
        
        // Send some data then cancel
        input <- "data1"
        time.Sleep(10 * time.Millisecond)
        cancel()
        
        err := <-errChan
        if err == nil {
            t.Error("Expected error due to cancellation")
        }
        
        expectedMsg := "stream processing cancelled"
        if !contains(err.Error(), expectedMsg) {
            t.Errorf("Expected error to contain %q, got %q", expectedMsg, err.Error())
        }
    })
}

func TestRetryableOperationContextHandling(t *testing.T) {
    ws := NewWorkerService("retry-worker")
    
    t.Run("SuccessfulOperation", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
        defer cancel()
        
        attempts := 0
        operation := func() error {
            attempts++
            if attempts < 3 {
                return fmt.Errorf("simulated failure %d", attempts)
            }
            return nil
        }
        
        err := ws.RetryableOperation(ctx, 5, operation)
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        if attempts != 3 {
            t.Errorf("Expected 3 attempts, got %d", attempts)
        }
    })
    
    t.Run("OperationCancellation", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 30*time.Millisecond)
        defer cancel()
        
        attempts := 0
        operation := func() error {
            attempts++
            return fmt.Errorf("always fails")
        }
        
        start := time.Now()
        err := ws.RetryableOperation(ctx, 10, operation)
        duration := time.Since(start)
        
        if err == nil {
            t.Error("Expected error due to cancellation")
        }
        
        // Should be cancelled before all retries complete
        if attempts >= 10 {
            t.Errorf("Too many attempts completed: %d", attempts)
        }
        
        // Should be cancelled within timeout period
        if duration > 50*time.Millisecond {
            t.Errorf("Operation took too long: %v", duration)
        }
        
        expectedMsg := "operation cancelled"
        if !contains(err.Error(), expectedMsg) {
            t.Errorf("Expected error to contain %q, got %q", expectedMsg, err.Error())
        }
    })
    
    t.Run("AllRetriesExhausted", func(t *testing.T) {
        ctx, cancel := context.WithTimeout(context.Background(), 200*time.Millisecond)
        defer cancel()
        
        attempts := 0
        operation := func() error {
            attempts++
            return fmt.Errorf("persistent failure")
        }
        
        err := ws.RetryableOperation(ctx, 3, operation)
        if err == nil {
            t.Error("Expected error after exhausting retries")
        }
        
        if attempts != 4 { // 3 retries + 1 initial attempt
            t.Errorf("Expected 4 attempts, got %d", attempts)
        }
        
        expectedMsg := "operation failed after 4 attempts"
        if !contains(err.Error(), expectedMsg) {
            t.Errorf("Expected error to contain %q, got %q", expectedMsg, err.Error())
        }
    })
}

func TestContextValuePropagation(t *testing.T) {
    type contextKey string
    const userIDKey contextKey = "userID"
    
    t.Run("ContextValuePreservation", func(t *testing.T) {
        ctx := context.WithValue(context.Background(), userIDKey, "user123")
        ctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
        defer cancel()
        
        // Simulate operation that uses context values
        operation := func() error {
            userID := ctx.Value(userIDKey)
            if userID == nil {
                return fmt.Errorf("user ID not found in context")
            }
            
            if userID.(string) != "user123" {
                return fmt.Errorf("unexpected user ID: %v", userID)
            }
            
            return nil
        }
        
        ws := NewWorkerService("context-worker")
        err := ws.RetryableOperation(ctx, 3, operation)
        if err != nil {
            t.Errorf("Context value propagation failed: %v", err)
        }
    })
}
```

Context-aware testing validates proper timeout handling, cancellation behavior,  
and resource cleanup in concurrent operations. These tests ensure functions  
respond correctly to context cancellation and maintain predictable behavior  
under various timing constraints.  

## Race condition testing

Race condition testing identifies data races and concurrent access issues  
using the race detector and systematic concurrent testing patterns.  

```go
package main

import (
    "sync"
    "testing"
    "time"
)

// Counter provides thread-safe counting operations
type Counter struct {
    value int
    mutex sync.RWMutex
}

// NewCounter creates a new counter
func NewCounter() *Counter {
    return &Counter{}
}

// Increment increases the counter value
func (c *Counter) Increment() {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.value++
}

// Decrement decreases the counter value
func (c *Counter) Decrement() {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.value--
}

// GetValue returns the current counter value
func (c *Counter) GetValue() int {
    c.mutex.RLock()
    defer c.mutex.RUnlock()
    return c.value
}

// Add adds a value to the counter
func (c *Counter) Add(n int) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.value += n
}

// Reset sets the counter to zero
func (c *Counter) Reset() {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.value = 0
}

// UnsafeCounter demonstrates a counter without proper synchronization
type UnsafeCounter struct {
    value int
}

// NewUnsafeCounter creates an unsafe counter
func NewUnsafeCounter() *UnsafeCounter {
    return &UnsafeCounter{}
}

// Increment increases the counter value (unsafe)
func (uc *UnsafeCounter) Increment() {
    uc.value++
}

// GetValue returns the current counter value (unsafe)
func (uc *UnsafeCounter) GetValue() int {
    return uc.value
}

// SharedResource simulates a resource with concurrent access
type SharedResource struct {
    data  map[string]int
    mutex sync.RWMutex
}

// NewSharedResource creates a shared resource
func NewSharedResource() *SharedResource {
    return &SharedResource{
        data: make(map[string]int),
    }
}

// Set sets a value in the resource
func (sr *SharedResource) Set(key string, value int) {
    sr.mutex.Lock()
    defer sr.mutex.Unlock()
    sr.data[key] = value
}

// Get retrieves a value from the resource
func (sr *SharedResource) Get(key string) (int, bool) {
    sr.mutex.RLock()
    defer sr.mutex.RUnlock()
    value, exists := sr.data[key]
    return value, exists
}

// Delete removes a key from the resource
func (sr *SharedResource) Delete(key string) {
    sr.mutex.Lock()
    defer sr.mutex.Unlock()
    delete(sr.data, key)
}

// GetAll returns all data (creates a copy)
func (sr *SharedResource) GetAll() map[string]int {
    sr.mutex.RLock()
    defer sr.mutex.RUnlock()
    
    result := make(map[string]int)
    for k, v := range sr.data {
        result[k] = v
    }
    return result
}

func TestCounterRaceConditions(t *testing.T) {
    t.Run("SafeCounterConcurrency", func(t *testing.T) {
        counter := NewCounter()
        numGoroutines := 100
        incrementsPerGoroutine := 100
        
        var wg sync.WaitGroup
        wg.Add(numGoroutines)
        
        // Launch concurrent incrementers
        for i := 0; i < numGoroutines; i++ {
            go func() {
                defer wg.Done()
                for j := 0; j < incrementsPerGoroutine; j++ {
                    counter.Increment()
                }
            }()
        }
        
        wg.Wait()
        
        expected := numGoroutines * incrementsPerGoroutine
        actual := counter.GetValue()
        
        if actual != expected {
            t.Errorf("Expected counter value %d, got %d", expected, actual)
        }
    })
    
    t.Run("UnsafeCounterDataRace", func(t *testing.T) {
        // This test should be run with -race flag to detect data races
        counter := NewUnsafeCounter()
        numGoroutines := 50
        incrementsPerGoroutine := 50
        
        var wg sync.WaitGroup
        wg.Add(numGoroutines)
        
        // Launch concurrent incrementers (this will cause data races)
        for i := 0; i < numGoroutines; i++ {
            go func() {
                defer wg.Done()
                for j := 0; j < incrementsPerGoroutine; j++ {
                    counter.Increment()
                }
            }()
        }
        
        wg.Wait()
        
        expected := numGoroutines * incrementsPerGoroutine
        actual := counter.GetValue()
        
        // Due to race conditions, actual may not equal expected
        t.Logf("Expected: %d, Actual: %d (race conditions may cause mismatch)", expected, actual)
    })
    
    t.Run("MixedOperationsConcurrency", func(t *testing.T) {
        counter := NewCounter()
        numOperations := 1000
        
        var wg sync.WaitGroup
        wg.Add(3)
        
        // Incrementer goroutine
        go func() {
            defer wg.Done()
            for i := 0; i < numOperations; i++ {
                counter.Increment()
            }
        }()
        
        // Decrementer goroutine
        go func() {
            defer wg.Done()
            for i := 0; i < numOperations/2; i++ {
                counter.Decrement()
            }
        }()
        
        // Reader goroutine
        go func() {
            defer wg.Done()
            for i := 0; i < numOperations; i++ {
                _ = counter.GetValue()
            }
        }()
        
        wg.Wait()
        
        expected := numOperations - numOperations/2 // 1000 - 500 = 500
        actual := counter.GetValue()
        
        if actual != expected {
            t.Errorf("Expected counter value %d, got %d", expected, actual)
        }
    })
}

func TestSharedResourceRaceConditions(t *testing.T) {
    t.Run("ConcurrentMapOperations", func(t *testing.T) {
        resource := NewSharedResource()
        numGoroutines := 20
        operationsPerGoroutine := 50
        
        var wg sync.WaitGroup
        wg.Add(numGoroutines * 3) // writers, readers, deleters
        
        // Writer goroutines
        for i := 0; i < numGoroutines; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < operationsPerGoroutine; j++ {
                    key := fmt.Sprintf("key_%d_%d", id, j)
                    resource.Set(key, id*1000+j)
                }
            }(i)
        }
        
        // Reader goroutines
        for i := 0; i < numGoroutines; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < operationsPerGoroutine; j++ {
                    key := fmt.Sprintf("key_%d_%d", id, j)
                    _, _ = resource.Get(key)
                }
            }(i)
        }
        
        // Deleter goroutines (delete some keys)
        for i := 0; i < numGoroutines; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < operationsPerGoroutine/2; j++ {
                    key := fmt.Sprintf("key_%d_%d", id, j*2)
                    resource.Delete(key)
                }
            }(i)
        }
        
        wg.Wait()
        
        // Verify final state
        allData := resource.GetAll()
        t.Logf("Final resource size: %d", len(allData))
        
        // Should have some data remaining
        if len(allData) == 0 {
            t.Error("Expected some data to remain after operations")
        }
    })
    
    t.Run("ConcurrentReadsAndWrites", func(t *testing.T) {
        resource := NewSharedResource()
        
        // Pre-populate with some data
        for i := 0; i < 100; i++ {
            resource.Set(fmt.Sprintf("init_%d", i), i)
        }
        
        var wg sync.WaitGroup
        numReaders := 10
        numWriters := 5
        
        wg.Add(numReaders + numWriters)
        
        // Reader goroutines
        for i := 0; i < numReaders; i++ {
            go func() {
                defer wg.Done()
                for j := 0; j < 200; j++ {
                    key := fmt.Sprintf("init_%d", j%100)
                    _, _ = resource.Get(key)
                }
            }()
        }
        
        // Writer goroutines
        for i := 0; i < numWriters; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < 100; j++ {
                    key := fmt.Sprintf("writer_%d_%d", id, j)
                    resource.Set(key, id*1000+j)
                }
            }(i)
        }
        
        wg.Wait()
        
        // Verify data integrity
        allData := resource.GetAll()
        
        // Should have initial data plus writer data
        expectedMinSize := 100 + (numWriters * 100)
        if len(allData) < expectedMinSize {
            t.Errorf("Expected at least %d items, got %d", expectedMinSize, len(allData))
        }
    })
}

func TestRaceDetectionPatterns(t *testing.T) {
    t.Run("DoubleCheckedLocking", func(t *testing.T) {
        // Demonstrates a pattern that might seem safe but can have race conditions
        var initialized bool
        var resource *SharedResource
        var mutex sync.Mutex
        
        numGoroutines := 100
        var wg sync.WaitGroup
        wg.Add(numGoroutines)
        
        for i := 0; i < numGoroutines; i++ {
            go func() {
                defer wg.Done()
                
                // Double-checked locking pattern
                if !initialized {
                    mutex.Lock()
                    if !initialized {
                        resource = NewSharedResource()
                        initialized = true
                    }
                    mutex.Unlock()
                }
                
                // Use the resource
                if resource != nil {
                    resource.Set("test", 1)
                }
            }()
        }
        
        wg.Wait()
        
        if resource == nil {
            t.Error("Resource should be initialized")
        }
    })
    
    t.Run("ConcurrentSliceAppend", func(t *testing.T) {
        // Demonstrates race conditions with slice operations
        var slice []int
        var mutex sync.Mutex
        numGoroutines := 50
        appendsPerGoroutine := 20
        
        var wg sync.WaitGroup
        wg.Add(numGoroutines)
        
        for i := 0; i < numGoroutines; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < appendsPerGoroutine; j++ {
                    mutex.Lock()
                    slice = append(slice, id*1000+j)
                    mutex.Unlock()
                }
            }(i)
        }
        
        wg.Wait()
        
        expected := numGoroutines * appendsPerGoroutine
        if len(slice) != expected {
            t.Errorf("Expected slice length %d, got %d", expected, len(slice))
        }
    })
    
    t.Run("ChannelRaceConditions", func(t *testing.T) {
        ch := make(chan int, 100)
        numSenders := 5
        numReceivers := 3
        messagesPerSender := 20
        
        var wg sync.WaitGroup
        wg.Add(numSenders + numReceivers)
        
        // Senders
        for i := 0; i < numSenders; i++ {
            go func(id int) {
                defer wg.Done()
                for j := 0; j < messagesPerSender; j++ {
                    ch <- id*1000 + j
                }
            }(i)
        }
        
        // Close channel after all senders finish
        go func() {
            wg.Wait()
            close(ch)
        }()
        
        received := make([]int, 0)
        var receiveMutex sync.Mutex
        
        // Receivers
        for i := 0; i < numReceivers; i++ {
            go func() {
                defer wg.Done()
                for msg := range ch {
                    receiveMutex.Lock()
                    received = append(received, msg)
                    receiveMutex.Unlock()
                }
            }()
        }
        
        // Wait for all goroutines to complete
        // Note: We need to wait for receivers separately since we modified wg in the closer goroutine
        time.Sleep(100 * time.Millisecond)
        
        expectedMessages := numSenders * messagesPerSender
        if len(received) != expectedMessages {
            t.Errorf("Expected %d messages, received %d", expectedMessages, len(received))
        }
    })
}

func TestDataRaceRecovery(t *testing.T) {
    t.Run("GracefulShutdown", func(t *testing.T) {
        counter := NewCounter()
        stopChan := make(chan bool)
        var wg sync.WaitGroup
        
        numWorkers := 10
        wg.Add(numWorkers)
        
        // Start workers
        for i := 0; i < numWorkers; i++ {
            go func() {
                defer wg.Done()
                for {
                    select {
                    case <-stopChan:
                        return
                    default:
                        counter.Increment()
                        time.Sleep(time.Microsecond)
                    }
                }
            }()
        }
        
        // Let workers run for a short time
        time.Sleep(10 * time.Millisecond)
        
        // Signal stop
        close(stopChan)
        
        // Wait for graceful shutdown
        wg.Wait()
        
        finalValue := counter.GetValue()
        t.Logf("Final counter value after graceful shutdown: %d", finalValue)
        
        if finalValue <= 0 {
            t.Error("Expected counter to have positive value")
        }
    })
}
```

Race condition testing identifies concurrent access issues and data races using  
systematic concurrent testing patterns. Running tests with the `-race` flag  
enables Go's race detector to catch subtle concurrency bugs that might  
otherwise go unnoticed in production.  

## Test configuration and environment

Test configuration management ensures tests run consistently across different  
environments and provides flexible test execution control.  

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "strconv"
    "testing"
    "time"
)

// TestConfig holds configuration for test execution
type TestConfig struct {
    DatabaseURL     string
    APIEndpoint     string
    Timeout         time.Duration
    MaxRetries      int
    EnableSlowTests bool
    LogLevel        string
    TestDataDir     string
}

// LoadTestConfig loads configuration from environment variables and flags
func LoadTestConfig() *TestConfig {
    config := &TestConfig{
        DatabaseURL:     getEnvWithDefault("TEST_DATABASE_URL", "sqlite3://:memory:"),
        APIEndpoint:     getEnvWithDefault("TEST_API_ENDPOINT", "http://localhost:8080"),
        Timeout:         getDurationEnvWithDefault("TEST_TIMEOUT", 30*time.Second),
        MaxRetries:      getIntEnvWithDefault("TEST_MAX_RETRIES", 3),
        EnableSlowTests: getBoolEnvWithDefault("TEST_ENABLE_SLOW", false),
        LogLevel:        getEnvWithDefault("TEST_LOG_LEVEL", "INFO"),
        TestDataDir:     getEnvWithDefault("TEST_DATA_DIR", "./testdata"),
    }
    
    return config
}

func getEnvWithDefault(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getDurationEnvWithDefault(key string, defaultValue time.Duration) time.Duration {
    if value := os.Getenv(key); value != "" {
        if duration, err := time.ParseDuration(value); err == nil {
            return duration
        }
    }
    return defaultValue
}

func getIntEnvWithDefault(key string, defaultValue int) int {
    if value := os.Getenv(key); value != "" {
        if intValue, err := strconv.Atoi(value); err == nil {
            return intValue
        }
    }
    return defaultValue
}

func getBoolEnvWithDefault(key string, defaultValue bool) bool {
    if value := os.Getenv(key); value != "" {
        if boolValue, err := strconv.ParseBool(value); err == nil {
            return boolValue
        }
    }
    return defaultValue
}

// ServiceClient represents a client for external service
type ServiceClient struct {
    baseURL    string
    timeout    time.Duration
    maxRetries int
}

// NewServiceClient creates a service client with configuration
func NewServiceClient(config *TestConfig) *ServiceClient {
    return &ServiceClient{
        baseURL:    config.APIEndpoint,
        timeout:    config.Timeout,
        maxRetries: config.MaxRetries,
    }
}

// HealthCheck performs a health check against the service
func (sc *ServiceClient) HealthCheck() error {
    // Simulate health check logic
    if sc.baseURL == "" {
        return fmt.Errorf("invalid base URL")
    }
    
    // Simulate network delay
    time.Sleep(10 * time.Millisecond)
    return nil
}

// SendRequest simulates sending a request with retries
func (sc *ServiceClient) SendRequest(data string) error {
    for attempt := 0; attempt <= sc.maxRetries; attempt++ {
        if err := sc.performRequest(data); err != nil {
            if attempt == sc.maxRetries {
                return fmt.Errorf("request failed after %d attempts: %w", sc.maxRetries+1, err)
            }
            time.Sleep(time.Millisecond * 10)
            continue
        }
        return nil
    }
    return fmt.Errorf("unexpected error")
}

func (sc *ServiceClient) performRequest(data string) error {
    if data == "fail" {
        return fmt.Errorf("simulated request failure")
    }
    return nil
}

// Global test configuration
var testConfig *TestConfig

// TestMain provides setup and teardown for the entire test suite
func TestMain(m *testing.M) {
    // Parse flags
    flag.Parse()
    
    // Load configuration
    testConfig = LoadTestConfig()
    
    // Perform global setup
    if err := setupTestEnvironment(); err != nil {
        fmt.Fprintf(os.Stderr, "Failed to setup test environment: %v\n", err)
        os.Exit(1)
    }
    
    // Run tests
    code := m.Run()
    
    // Perform global teardown
    if err := teardownTestEnvironment(); err != nil {
        fmt.Fprintf(os.Stderr, "Failed to teardown test environment: %v\n", err)
    }
    
    os.Exit(code)
}

func setupTestEnvironment() error {
    // Create test data directory if it doesn't exist
    if err := os.MkdirAll(testConfig.TestDataDir, 0755); err != nil {
        return fmt.Errorf("failed to create test data directory: %w", err)
    }
    
    // Additional setup logic
    fmt.Printf("Test environment setup complete\n")
    fmt.Printf("Database URL: %s\n", testConfig.DatabaseURL)
    fmt.Printf("API Endpoint: %s\n", testConfig.APIEndpoint)
    fmt.Printf("Timeout: %v\n", testConfig.Timeout)
    
    return nil
}

func teardownTestEnvironment() error {
    // Cleanup test data directory
    if err := os.RemoveAll(testConfig.TestDataDir); err != nil {
        return fmt.Errorf("failed to remove test data directory: %w", err)
    }
    
    fmt.Printf("Test environment teardown complete\n")
    return nil
}

func TestServiceClientConfiguration(t *testing.T) {
    if testConfig == nil {
        t.Fatal("Test configuration not loaded")
    }
    
    t.Run("ConfigurationLoading", func(t *testing.T) {
        if testConfig.DatabaseURL == "" {
            t.Error("Database URL should not be empty")
        }
        
        if testConfig.APIEndpoint == "" {
            t.Error("API endpoint should not be empty")
        }
        
        if testConfig.Timeout <= 0 {
            t.Error("Timeout should be positive")
        }
        
        if testConfig.MaxRetries < 0 {
            t.Error("Max retries should not be negative")
        }
    })
    
    t.Run("ServiceClientCreation", func(t *testing.T) {
        client := NewServiceClient(testConfig)
        
        if client == nil {
            t.Fatal("Service client should not be nil")
        }
        
        if client.baseURL != testConfig.APIEndpoint {
            t.Errorf("Expected base URL %s, got %s", testConfig.APIEndpoint, client.baseURL)
        }
        
        if client.timeout != testConfig.Timeout {
            t.Errorf("Expected timeout %v, got %v", testConfig.Timeout, client.timeout)
        }
    })
    
    t.Run("HealthCheck", func(t *testing.T) {
        client := NewServiceClient(testConfig)
        
        err := client.HealthCheck()
        if err != nil {
            t.Errorf("Health check failed: %v", err)
        }
    })
    
    t.Run("RequestWithRetries", func(t *testing.T) {
        client := NewServiceClient(testConfig)
        
        // Test successful request
        err := client.SendRequest("success")
        if err != nil {
            t.Errorf("Successful request failed: %v", err)
        }
        
        // Test failing request
        err = client.SendRequest("fail")
        if err == nil {
            t.Error("Expected failing request to return error")
        }
    })
}

func TestEnvironmentSpecificBehavior(t *testing.T) {
    t.Run("DatabaseURLBasedTests", func(t *testing.T) {
        if testConfig.DatabaseURL == "sqlite3://:memory:" {
            t.Log("Running tests with in-memory SQLite database")
            // In-memory database specific tests
        } else {
            t.Log("Running tests with external database")
            // External database specific tests
        }
    })
    
    t.Run("SlowTestsConditional", func(t *testing.T) {
        if !testConfig.EnableSlowTests {
            t.Skip("Slow tests disabled")
        }
        
        // Simulate slow test
        start := time.Now()
        time.Sleep(100 * time.Millisecond)
        duration := time.Since(start)
        
        t.Logf("Slow test completed in %v", duration)
    })
    
    t.Run("LogLevelConfiguration", func(t *testing.T) {
        switch testConfig.LogLevel {
        case "DEBUG":
            t.Log("Debug logging enabled - verbose output")
        case "INFO":
            t.Log("Info logging enabled - standard output")
        case "ERROR":
            t.Log("Error logging enabled - minimal output")
        default:
            t.Logf("Unknown log level: %s", testConfig.LogLevel)
        }
    })
}

func TestDataDirectoryOperations(t *testing.T) {
    t.Run("TestDataDirectory", func(t *testing.T) {
        // Verify test data directory exists
        if _, err := os.Stat(testConfig.TestDataDir); os.IsNotExist(err) {
            t.Errorf("Test data directory does not exist: %s", testConfig.TestDataDir)
        }
        
        // Create a test file
        testFile := fmt.Sprintf("%s/test_file.txt", testConfig.TestDataDir)
        content := "Hello there, test data!"
        
        err := os.WriteFile(testFile, []byte(content), 0644)
        if err != nil {
            t.Fatalf("Failed to write test file: %v", err)
        }
        
        // Read and verify
        readContent, err := os.ReadFile(testFile)
        if err != nil {
            t.Fatalf("Failed to read test file: %v", err)
        }
        
        if string(readContent) != content {
            t.Errorf("Expected content %q, got %q", content, string(readContent))
        }
        
        // Cleanup
        err = os.Remove(testFile)
        if err != nil {
            t.Errorf("Failed to remove test file: %v", err)
        }
    })
}

func TestConfigurationValidation(t *testing.T) {
    t.Run("TimeoutValidation", func(t *testing.T) {
        if testConfig.Timeout < time.Second {
            t.Logf("Warning: Timeout is very short: %v", testConfig.Timeout)
        }
        
        if testConfig.Timeout > time.Minute {
            t.Logf("Warning: Timeout is very long: %v", testConfig.Timeout)
        }
    })
    
    t.Run("RetryValidation", func(t *testing.T) {
        if testConfig.MaxRetries > 10 {
            t.Logf("Warning: Max retries is high: %d", testConfig.MaxRetries)
        }
        
        if testConfig.MaxRetries == 0 {
            t.Log("Info: No retries configured")
        }
    })
    
    t.Run("URLValidation", func(t *testing.T) {
        if testConfig.APIEndpoint == "http://localhost:8080" {
            t.Log("Info: Using default localhost endpoint")
        }
        
        if testConfig.DatabaseURL == "sqlite3://:memory:" {
            t.Log("Info: Using in-memory database")
        }
    })
}

// Benchmark with configuration
func BenchmarkServiceClientWithConfig(b *testing.B) {
    if testConfig == nil {
        b.Fatal("Test configuration not loaded")
    }
    
    client := NewServiceClient(testConfig)
    
    b.Run("HealthCheck", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            client.HealthCheck()
        }
    })
    
    b.Run("SuccessfulRequest", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            client.SendRequest("success")
        }
    })
}
```

Test configuration management provides flexible, environment-aware testing  
that adapts to different deployment scenarios. Configuration through environment  
variables and flags enables consistent testing across development, CI/CD,  
and production environments.  

## Golden file testing

Golden file testing compares program output against stored reference files,  
ensuring output consistency and facilitating regression detection.  

```go
package main

import (
    "bytes"
    "encoding/json"
    "flag"
    "fmt"
    "os"
    "path/filepath"
    "strings"
    "testing"
)

// ReportGenerator creates various types of reports
type ReportGenerator struct {
    title  string
    author string
}

// NewReportGenerator creates a report generator
func NewReportGenerator(title, author string) *ReportGenerator {
    return &ReportGenerator{
        title:  title,
        author: author,
    }
}

// GenerateTextReport creates a formatted text report
func (rg *ReportGenerator) GenerateTextReport(data []DataPoint) string {
    var buffer bytes.Buffer
    
    buffer.WriteString(fmt.Sprintf("Report: %s\n", rg.title))
    buffer.WriteString(fmt.Sprintf("Author: %s\n", rg.author))
    buffer.WriteString(strings.Repeat("=", 50) + "\n\n")
    
    if len(data) == 0 {
        buffer.WriteString("No data available.\n")
        return buffer.String()
    }
    
    buffer.WriteString("Data Summary:\n")
    buffer.WriteString("--------------\n")
    
    total := 0.0
    for _, point := range data {
        buffer.WriteString(fmt.Sprintf("%s: %.2f\n", point.Label, point.Value))
        total += point.Value
    }
    
    buffer.WriteString(fmt.Sprintf("\nTotal: %.2f\n", total))
    buffer.WriteString(fmt.Sprintf("Average: %.2f\n", total/float64(len(data))))
    buffer.WriteString(fmt.Sprintf("Count: %d\n", len(data)))
    
    return buffer.String()
}

// GenerateJSONReport creates a JSON-formatted report
func (rg *ReportGenerator) GenerateJSONReport(data []DataPoint) (string, error) {
    report := struct {
        Title   string      `json:"title"`
        Author  string      `json:"author"`
        Data    []DataPoint `json:"data"`
        Summary Summary     `json:"summary"`
    }{
        Title:  rg.title,
        Author: rg.author,
        Data:   data,
        Summary: rg.calculateSummary(data),
    }
    
    jsonBytes, err := json.MarshalIndent(report, "", "  ")
    if err != nil {
        return "", err
    }
    
    return string(jsonBytes), nil
}

// GenerateCSVReport creates a CSV-formatted report
func (rg *ReportGenerator) GenerateCSVReport(data []DataPoint) string {
    var buffer bytes.Buffer
    
    // Header
    buffer.WriteString("Label,Value\n")
    
    // Data rows
    for _, point := range data {
        buffer.WriteString(fmt.Sprintf("%s,%.2f\n", point.Label, point.Value))
    }
    
    return buffer.String()
}

// DataPoint represents a single data point
type DataPoint struct {
    Label string  `json:"label"`
    Value float64 `json:"value"`
}

// Summary contains calculated summary statistics
type Summary struct {
    Total   float64 `json:"total"`
    Average float64 `json:"average"`
    Count   int     `json:"count"`
    Min     float64 `json:"min"`
    Max     float64 `json:"max"`
}

func (rg *ReportGenerator) calculateSummary(data []DataPoint) Summary {
    if len(data) == 0 {
        return Summary{}
    }
    
    summary := Summary{
        Count: len(data),
        Min:   data[0].Value,
        Max:   data[0].Value,
    }
    
    for _, point := range data {
        summary.Total += point.Value
        if point.Value < summary.Min {
            summary.Min = point.Value
        }
        if point.Value > summary.Max {
            summary.Max = point.Value
        }
    }
    
    summary.Average = summary.Total / float64(summary.Count)
    return summary
}

// Golden file testing utilities
func updateGoldenFile(t *testing.T, goldenFile string, actual []byte) {
    t.Helper()
    
    if err := os.MkdirAll(filepath.Dir(goldenFile), 0755); err != nil {
        t.Fatalf("Failed to create golden file directory: %v", err)
    }
    
    if err := os.WriteFile(goldenFile, actual, 0644); err != nil {
        t.Fatalf("Failed to update golden file %s: %v", goldenFile, err)
    }
    
    t.Logf("Updated golden file: %s", goldenFile)
}

func compareWithGoldenFile(t *testing.T, goldenFile string, actual []byte) {
    t.Helper()
    
    expected, err := os.ReadFile(goldenFile)
    if err != nil {
        if os.IsNotExist(err) {
            t.Fatalf("Golden file %s does not exist. Run with -update flag to create it.", goldenFile)
        }
        t.Fatalf("Failed to read golden file %s: %v", goldenFile, err)
    }
    
    if !bytes.Equal(actual, expected) {
        t.Errorf("Output differs from golden file %s\nExpected:\n%s\nActual:\n%s", 
            goldenFile, string(expected), string(actual))
    }
}

func TestReportGeneratorGoldenFiles(t *testing.T) {
    testData := []DataPoint{
        {Label: "Sales Q1", Value: 125000.50},
        {Label: "Sales Q2", Value: 134500.75},
        {Label: "Sales Q3", Value: 142300.25},
        {Label: "Sales Q4", Value: 156200.00},
    }
    
    generator := NewReportGenerator("Quarterly Sales Report", "John Analyst")
    
    t.Run("TextReportGolden", func(t *testing.T) {
        actual := generator.GenerateTextReport(testData)
        goldenFile := "testdata/golden/text_report.txt"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(actual))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(actual))
    })
    
    t.Run("JSONReportGolden", func(t *testing.T) {
        actual, err := generator.GenerateJSONReport(testData)
        if err != nil {
            t.Fatalf("Failed to generate JSON report: %v", err)
        }
        
        goldenFile := "testdata/golden/json_report.json"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(actual))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(actual))
    })
    
    t.Run("CSVReportGolden", func(t *testing.T) {
        actual := generator.GenerateCSVReport(testData)
        goldenFile := "testdata/golden/csv_report.csv"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(actual))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(actual))
    })
    
    t.Run("EmptyDataGolden", func(t *testing.T) {
        emptyData := []DataPoint{}
        actual := generator.GenerateTextReport(emptyData)
        goldenFile := "testdata/golden/empty_data_report.txt"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(actual))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(actual))
    })
    
    t.Run("SingleDataPointGolden", func(t *testing.T) {
        singleData := []DataPoint{{Label: "Single Sale", Value: 1000.00}}
        actual := generator.GenerateTextReport(singleData)
        goldenFile := "testdata/golden/single_data_report.txt"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(actual))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(actual))
    })
}

func TestReportVariationsGolden(t *testing.T) {
    testCases := []struct {
        name      string
        title     string
        author    string
        data      []DataPoint
        goldenDir string
    }{
        {
            name:      "StandardReport",
            title:     "Sales Analysis",
            author:    "Data Team",
            data:      []DataPoint{{Label: "Revenue", Value: 50000}},
            goldenDir: "standard",
        },
        {
            name:      "LongTitleReport",
            title:     "Very Long Report Title That Spans Multiple Words And Contains Detailed Information",
            author:    "Senior Data Analyst",
            data:      []DataPoint{{Label: "Long Label Name", Value: 12345.67}},
            goldenDir: "long_title",
        },
        {
            name:      "SpecialCharactersReport",
            title:     "Report with Special chars: @#$%",
            author:    "Test & QA Team",
            data:      []DataPoint{{Label: "Data & Analytics", Value: 999.99}},
            goldenDir: "special_chars",
        },
    }
    
    for _, tc := range testCases {
        t.Run(tc.name, func(t *testing.T) {
            generator := NewReportGenerator(tc.title, tc.author)
            actual := generator.GenerateTextReport(tc.data)
            goldenFile := fmt.Sprintf("testdata/golden/%s/report.txt", tc.goldenDir)
            
            if *updateGolden {
                updateGoldenFile(t, goldenFile, []byte(actual))
                return
            }
            
            compareWithGoldenFile(t, goldenFile, []byte(actual))
        })
    }
}

func TestJSONStructureGolden(t *testing.T) {
    data := []DataPoint{
        {Label: "Test A", Value: 100.0},
        {Label: "Test B", Value: 200.0},
    }
    
    generator := NewReportGenerator("Structure Test", "Test Author")
    
    t.Run("JSONStructureValidation", func(t *testing.T) {
        jsonOutput, err := generator.GenerateJSONReport(data)
        if err != nil {
            t.Fatalf("Failed to generate JSON: %v", err)
        }
        
        // Validate JSON structure by unmarshaling
        var report struct {
            Title   string      `json:"title"`
            Author  string      `json:"author"`
            Data    []DataPoint `json:"data"`
            Summary Summary     `json:"summary"`
        }
        
        if err := json.Unmarshal([]byte(jsonOutput), &report); err != nil {
            t.Fatalf("Generated JSON is invalid: %v", err)
        }
        
        goldenFile := "testdata/golden/json_structure.json"
        
        if *updateGolden {
            updateGoldenFile(t, goldenFile, []byte(jsonOutput))
            return
        }
        
        compareWithGoldenFile(t, goldenFile, []byte(jsonOutput))
    })
}

// Command line flag for updating golden files
var updateGolden = flag.Bool("update", false, "update golden files")

func TestGoldenFileManagement(t *testing.T) {
    t.Run("GoldenFileDirectoryStructure", func(t *testing.T) {
        goldenDir := "testdata/golden"
        
        if *updateGolden {
            t.Skip("Skipping directory check during golden file update")
        }
        
        // Check if golden directory exists
        if _, err := os.Stat(goldenDir); os.IsNotExist(err) {
            t.Errorf("Golden file directory does not exist: %s", goldenDir)
        }
        
        // List existing golden files
        err := filepath.Walk(goldenDir, func(path string, info os.FileInfo, err error) error {
            if err != nil {
                return err
            }
            if !info.IsDir() {
                relPath, _ := filepath.Rel(goldenDir, path)
                t.Logf("Golden file: %s", relPath)
            }
            return nil
        })
        
        if err != nil {
            t.Errorf("Error walking golden directory: %v", err)
        }
    })
}
```

Golden file testing ensures output consistency by comparing generated content  
against stored reference files. This approach simplifies testing of complex  
output formats and helps detect unexpected changes in program behavior  
across different versions and environments.  

## Test metrics and reporting

Test metrics provide insights into test coverage, execution time, and quality  
indicators that help maintain high standards and identify improvement areas.  

```go
package main

import (
    "fmt"
    "runtime"
    "testing"
    "time"
)

// MetricsCollector gathers test execution metrics
type MetricsCollector struct {
    TestResults []TestResult
    StartTime   time.Time
    EndTime     time.Time
}

// TestResult represents the outcome of a single test
type TestResult struct {
    Name        string
    Duration    time.Duration
    Passed      bool
    Error       string
    MemoryUsage int64
}

// NewMetricsCollector creates a metrics collector
func NewMetricsCollector() *MetricsCollector {
    return &MetricsCollector{
        TestResults: make([]TestResult, 0),
        StartTime:   time.Now(),
    }
}

// RecordTest records a test result
func (mc *MetricsCollector) RecordTest(name string, duration time.Duration, passed bool, err string) {
    var memStats runtime.MemStats
    runtime.ReadMemStats(&memStats)
    
    result := TestResult{
        Name:        name,
        Duration:    duration,
        Passed:      passed,
        Error:       err,
        MemoryUsage: int64(memStats.Alloc),
    }
    
    mc.TestResults = append(mc.TestResults, result)
}

// FinishCollection finalizes metrics collection
func (mc *MetricsCollector) FinishCollection() {
    mc.EndTime = time.Now()
}

// GenerateReport creates a comprehensive test report
func (mc *MetricsCollector) GenerateReport() TestReport {
    totalTests := len(mc.TestResults)
    passedTests := 0
    var totalDuration time.Duration
    var maxDuration time.Duration
    var minDuration time.Duration = time.Hour // Initialize to large value
    var totalMemory int64
    
    for _, result := range mc.TestResults {
        if result.Passed {
            passedTests++
        }
        
        totalDuration += result.Duration
        if result.Duration > maxDuration {
            maxDuration = result.Duration
        }
        if result.Duration < minDuration {
            minDuration = result.Duration
        }
        
        totalMemory += result.MemoryUsage
    }
    
    if totalTests == 0 {
        minDuration = 0
    }
    
    successRate := 0.0
    if totalTests > 0 {
        successRate = float64(passedTests) / float64(totalTests) * 100
    }
    
    return TestReport{
        TotalTests:       totalTests,
        PassedTests:      passedTests,
        FailedTests:      totalTests - passedTests,
        SuccessRate:      successRate,
        TotalDuration:    totalDuration,
        AverageDuration:  totalDuration / time.Duration(totalTests),
        MaxDuration:      maxDuration,
        MinDuration:      minDuration,
        TotalMemoryUsage: totalMemory,
        AverageMemoryUsage: totalMemory / int64(totalTests),
        ExecutionTime:    mc.EndTime.Sub(mc.StartTime),
    }
}

// TestReport contains comprehensive test metrics
type TestReport struct {
    TotalTests         int
    PassedTests        int
    FailedTests        int
    SuccessRate        float64
    TotalDuration      time.Duration
    AverageDuration    time.Duration
    MaxDuration        time.Duration
    MinDuration        time.Duration
    TotalMemoryUsage   int64
    AverageMemoryUsage int64
    ExecutionTime      time.Duration
}

// String returns a formatted string representation of the report
func (tr TestReport) String() string {
    return fmt.Sprintf(`Test Execution Report
========================
Total Tests: %d
Passed: %d
Failed: %d
Success Rate: %.2f%%
Total Duration: %v
Average Duration: %v
Max Duration: %v
Min Duration: %v
Total Memory Usage: %d bytes
Average Memory Usage: %d bytes
Execution Time: %v`,
        tr.TotalTests,
        tr.PassedTests,
        tr.FailedTests,
        tr.SuccessRate,
        tr.TotalDuration,
        tr.AverageDuration,
        tr.MaxDuration,
        tr.MinDuration,
        tr.TotalMemoryUsage,
        tr.AverageMemoryUsage,
        tr.ExecutionTime)
}

// Global metrics collector for demonstration
var globalMetrics = NewMetricsCollector()

// TestableFunction represents a function that can be tested
type TestableFunction func() error

// SlowFunction simulates a slow operation
func SlowFunction() error {
    time.Sleep(50 * time.Millisecond)
    return nil
}

// FastFunction simulates a fast operation
func FastFunction() error {
    time.Sleep(1 * time.Millisecond)
    return nil
}

// FailingFunction simulates a failing operation
func FailingFunction() error {
    time.Sleep(5 * time.Millisecond)
    return fmt.Errorf("simulated failure")
}

// MemoryIntensiveFunction simulates memory-intensive operation
func MemoryIntensiveFunction() error {
    // Allocate some memory
    data := make([]byte, 1024*1024) // 1MB
    for i := range data {
        data[i] = byte(i % 256)
    }
    
    time.Sleep(10 * time.Millisecond)
    return nil
}

func TestMetricsCollection(t *testing.T) {
    metrics := NewMetricsCollector()
    
    t.Run("BasicMetricsRecording", func(t *testing.T) {
        start := time.Now()
        err := FastFunction()
        duration := time.Since(start)
        
        metrics.RecordTest("FastFunction", duration, err == nil, "")
        
        if len(metrics.TestResults) != 1 {
            t.Errorf("Expected 1 test result, got %d", len(metrics.TestResults))
        }
        
        result := metrics.TestResults[0]
        if result.Name != "FastFunction" {
            t.Errorf("Expected name 'FastFunction', got %s", result.Name)
        }
        
        if !result.Passed {
            t.Error("Expected test to pass")
        }
    })
    
    t.Run("FailureMetricsRecording", func(t *testing.T) {
        start := time.Now()
        err := FailingFunction()
        duration := time.Since(start)
        
        errorMsg := ""
        if err != nil {
            errorMsg = err.Error()
        }
        
        metrics.RecordTest("FailingFunction", duration, err == nil, errorMsg)
        
        if len(metrics.TestResults) != 2 {
            t.Errorf("Expected 2 test results, got %d", len(metrics.TestResults))
        }
        
        result := metrics.TestResults[1]
        if result.Passed {
            t.Error("Expected test to fail")
        }
        
        if result.Error == "" {
            t.Error("Expected error message to be recorded")
        }
    })
    
    t.Run("MemoryMetricsRecording", func(t *testing.T) {
        runtime.GC() // Force garbage collection for consistent memory measurements
        
        start := time.Now()
        err := MemoryIntensiveFunction()
        duration := time.Since(start)
        
        metrics.RecordTest("MemoryIntensiveFunction", duration, err == nil, "")
        
        result := metrics.TestResults[len(metrics.TestResults)-1]
        if result.MemoryUsage <= 0 {
            t.Error("Expected positive memory usage")
        }
        
        t.Logf("Memory usage: %d bytes", result.MemoryUsage)
    })
}

func TestMetricsReporting(t *testing.T) {
    metrics := NewMetricsCollector()
    
    // Simulate various test executions
    testFunctions := []struct {
        name string
        fn   TestableFunction
    }{
        {"Fast1", FastFunction},
        {"Fast2", FastFunction},
        {"Slow1", SlowFunction},
        {"Slow2", SlowFunction},
        {"Fail1", FailingFunction},
        {"Memory1", MemoryIntensiveFunction},
    }
    
    for _, tf := range testFunctions {
        start := time.Now()
        err := tf.fn()
        duration := time.Since(start)
        
        errorMsg := ""
        if err != nil {
            errorMsg = err.Error()
        }
        
        metrics.RecordTest(tf.name, duration, err == nil, errorMsg)
    }
    
    metrics.FinishCollection()
    report := metrics.GenerateReport()
    
    t.Run("ReportValidation", func(t *testing.T) {
        if report.TotalTests != 6 {
            t.Errorf("Expected 6 total tests, got %d", report.TotalTests)
        }
        
        if report.PassedTests != 5 {
            t.Errorf("Expected 5 passed tests, got %d", report.PassedTests)
        }
        
        if report.FailedTests != 1 {
            t.Errorf("Expected 1 failed test, got %d", report.FailedTests)
        }
        
        expectedSuccessRate := float64(5) / float64(6) * 100
        if report.SuccessRate < expectedSuccessRate-0.1 || report.SuccessRate > expectedSuccessRate+0.1 {
            t.Errorf("Expected success rate ~%.2f%%, got %.2f%%", expectedSuccessRate, report.SuccessRate)
        }
        
        if report.AverageDuration <= 0 {
            t.Error("Expected positive average duration")
        }
        
        if report.MaxDuration <= report.MinDuration {
            t.Error("Expected max duration to be greater than min duration")
        }
    })
    
    t.Run("ReportFormatting", func(t *testing.T) {
        reportString := report.String()
        
        if !contains(reportString, "Test Execution Report") {
            t.Error("Expected report to contain title")
        }
        
        if !contains(reportString, fmt.Sprintf("Total Tests: %d", report.TotalTests)) {
            t.Error("Expected report to contain total tests count")
        }
        
        if !contains(reportString, fmt.Sprintf("Success Rate: %.2f%%", report.SuccessRate)) {
            t.Error("Expected report to contain success rate")
        }
        
        t.Log("Generated Report:")
        t.Log(reportString)
    })
}

func TestPerformanceMetrics(t *testing.T) {
    t.Run("PerformanceThresholds", func(t *testing.T) {
        start := time.Now()
        err := SlowFunction()
        duration := time.Since(start)
        
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        // Performance assertion
        threshold := 100 * time.Millisecond
        if duration > threshold {
            t.Errorf("Function took %v, expected under %v", duration, threshold)
        }
        
        t.Logf("Function execution time: %v", duration)
    })
    
    t.Run("MemoryUsageThreshold", func(t *testing.T) {
        runtime.GC()
        var memBefore runtime.MemStats
        runtime.ReadMemStats(&memBefore)
        
        err := MemoryIntensiveFunction()
        
        var memAfter runtime.MemStats
        runtime.ReadMemStats(&memAfter)
        
        if err != nil {
            t.Errorf("Unexpected error: %v", err)
        }
        
        memoryUsed := memAfter.Alloc - memBefore.Alloc
        
        // Memory usage assertion
        threshold := int64(2 * 1024 * 1024) // 2MB threshold
        if int64(memoryUsed) > threshold {
            t.Errorf("Function used %d bytes, expected under %d bytes", memoryUsed, threshold)
        }
        
        t.Logf("Memory usage: %d bytes", memoryUsed)
    })
}

func BenchmarkWithMetrics(b *testing.B) {
    metrics := NewMetricsCollector()
    
    b.Run("FastFunction", func(b *testing.B) {
        for i := 0; i < b.N; i++ {
            start := time.Now()
            err := FastFunction()
            duration := time.Since(start)
            
            metrics.RecordTest(fmt.Sprintf("FastFunction_%d", i), duration, err == nil, "")
        }
    })
    
    metrics.FinishCollection()
    report := metrics.GenerateReport()
    
    b.Logf("Benchmark metrics: %d operations, avg duration: %v", 
        report.TotalTests, report.AverageDuration)
}

func TestCoverageMetrics(t *testing.T) {
    t.Run("FunctionCoverageTest", func(t *testing.T) {
        // Test all exported functions to ensure coverage
        functions := []TestableFunction{
            FastFunction,
            SlowFunction,
            FailingFunction,
            MemoryIntensiveFunction,
        }
        
        coverageCount := 0
        for _, fn := range functions {
            err := fn()
            coverageCount++
            
            // Functions with errors are still covered
            if err != nil {
                t.Logf("Function returned error (expected for some): %v", err)
            }
        }
        
        expectedCoverage := len(functions)
        if coverageCount != expectedCoverage {
            t.Errorf("Expected to test %d functions, tested %d", expectedCoverage, coverageCount)
        }
        
        t.Logf("Function coverage: %d/%d (%.2f%%)", 
            coverageCount, expectedCoverage, 
            float64(coverageCount)/float64(expectedCoverage)*100)
    })
}
```

Test metrics and reporting provide quantitative insights into test execution,  
performance characteristics, and quality indicators. These metrics help  
identify trends, bottlenecks, and areas for improvement in both tests  
and the code under test.  

## Advanced testing patterns

Advanced testing patterns combine multiple techniques to create sophisticated  
test strategies for complex scenarios and edge case validation.  

```go
package main

import (
    "context"
    "fmt"
    "sync"
    "testing"
    "time"
)

// CircuitBreaker implements circuit breaker pattern
type CircuitBreaker struct {
    maxFailures int
    resetTimeout time.Duration
    failures     int
    lastFailTime time.Time
    state        CircuitState
    mutex        sync.RWMutex
}

// CircuitState represents circuit breaker state
type CircuitState int

const (
    Closed CircuitState = iota
    Open
    HalfOpen
)

// NewCircuitBreaker creates a circuit breaker
func NewCircuitBreaker(maxFailures int, resetTimeout time.Duration) *CircuitBreaker {
    return &CircuitBreaker{
        maxFailures:  maxFailures,
        resetTimeout: resetTimeout,
        state:        Closed,
    }
}

// Execute runs a function through the circuit breaker
func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mutex.Lock()
    defer cb.mutex.Unlock()
    
    switch cb.state {
    case Open:
        if time.Since(cb.lastFailTime) > cb.resetTimeout {
            cb.state = HalfOpen
            cb.failures = 0
        } else {
            return fmt.Errorf("circuit breaker is open")
        }
    case HalfOpen:
        // Allow one test call
    case Closed:
        // Normal operation
    }
    
    err := fn()
    if err != nil {
        cb.failures++
        cb.lastFailTime = time.Now()
        
        if cb.failures >= cb.maxFailures {
            cb.state = Open
        }
        return err
    }
    
    // Success - reset circuit
    cb.failures = 0
    cb.state = Closed
    return nil
}

// GetState returns current circuit state
func (cb *CircuitBreaker) GetState() CircuitState {
    cb.mutex.RLock()
    defer cb.mutex.RUnlock()
    return cb.state
}

// RetryableService demonstrates a service with retry logic
type RetryableService struct {
    circuitBreaker *CircuitBreaker
    maxRetries     int
    retryDelay     time.Duration
}

// NewRetryableService creates a retryable service
func NewRetryableService(maxRetries int, retryDelay time.Duration) *RetryableService {
    return &RetryableService{
        circuitBreaker: NewCircuitBreaker(3, 5*time.Second),
        maxRetries:     maxRetries,
        retryDelay:     retryDelay,
    }
}

// CallWithRetry executes operation with retry logic and circuit breaker
func (rs *RetryableService) CallWithRetry(ctx context.Context, operation func() error) error {
    for attempt := 0; attempt <= rs.maxRetries; attempt++ {
        select {
        case <-ctx.Done():
            return fmt.Errorf("operation cancelled: %w", ctx.Err())
        default:
        }
        
        err := rs.circuitBreaker.Execute(operation)
        if err == nil {
            return nil
        }
        
        if err.Error() == "circuit breaker is open" {
            return err
        }
        
        if attempt < rs.maxRetries {
            select {
            case <-time.After(rs.retryDelay):
                continue
            case <-ctx.Done():
                return fmt.Errorf("operation cancelled during retry: %w", ctx.Err())
            }
        }
    }
    
    return fmt.Errorf("operation failed after %d attempts", rs.maxRetries+1)
}

// AdvancedTestScenarios demonstrates complex testing patterns
func TestAdvancedPatterns(t *testing.T) {
    t.Run("CircuitBreakerStateMachine", func(t *testing.T) {
        cb := NewCircuitBreaker(2, 100*time.Millisecond)
        
        // Initially closed
        if cb.GetState() != Closed {
            t.Error("Circuit breaker should start in Closed state")
        }
        
        // First failure
        err := cb.Execute(func() error { return fmt.Errorf("failure 1") })
        if err == nil {
            t.Error("Expected error from failing operation")
        }
        if cb.GetState() != Closed {
            t.Error("Circuit breaker should remain Closed after first failure")
        }
        
        // Second failure - should open circuit
        err = cb.Execute(func() error { return fmt.Errorf("failure 2") })
        if err == nil {
            t.Error("Expected error from failing operation")
        }
        if cb.GetState() != Open {
            t.Error("Circuit breaker should be Open after max failures")
        }
        
        // Attempt while open
        err = cb.Execute(func() error { return nil })
        if err == nil || err.Error() != "circuit breaker is open" {
            t.Error("Expected circuit breaker open error")
        }
        
        // Wait for reset timeout
        time.Sleep(150 * time.Millisecond)
        
        // Should be half-open and allow one test
        err = cb.Execute(func() error { return nil })
        if err != nil {
            t.Errorf("Expected success after timeout, got: %v", err)
        }
        
        if cb.GetState() != Closed {
            t.Error("Circuit breaker should be Closed after successful test")
        }
    })
    
    t.Run("RetryWithCircuitBreaker", func(t *testing.T) {
        service := NewRetryableService(3, 10*time.Millisecond)
        ctx := context.Background()
        
        attemptCount := 0
        operation := func() error {
            attemptCount++
            if attemptCount < 3 {
                return fmt.Errorf("attempt %d failed", attemptCount)
            }
            return nil
        }
        
        err := service.CallWithRetry(ctx, operation)
        if err != nil {
            t.Errorf("Expected eventual success, got: %v", err)
        }
        
        if attemptCount != 3 {
            t.Errorf("Expected 3 attempts, got %d", attemptCount)
        }
    })
    
    t.Run("ContextCancellationDuringRetry", func(t *testing.T) {
        service := NewRetryableService(5, 50*time.Millisecond)
        ctx, cancel := context.WithTimeout(context.Background(), 75*time.Millisecond)
        defer cancel()
        
        operation := func() error {
            return fmt.Errorf("always fails")
        }
        
        start := time.Now()
        err := service.CallWithRetry(ctx, operation)
        duration := time.Since(start)
        
        if err == nil {
            t.Error("Expected error due to context cancellation")
        }
        
        // Should be cancelled before all retries complete
        if duration > 100*time.Millisecond {
            t.Errorf("Operation should be cancelled quickly, took %v", duration)
        }
        
        if !contains(err.Error(), "cancelled") {
            t.Errorf("Expected cancellation error, got: %v", err)
        }
    })
    
    t.Run("ConcurrentCircuitBreakerAccess", func(t *testing.T) {
        cb := NewCircuitBreaker(5, 100*time.Millisecond)
        numGoroutines := 20
        
        var wg sync.WaitGroup
        wg.Add(numGoroutines)
        
        results := make(chan error, numGoroutines)
        
        for i := 0; i < numGoroutines; i++ {
            go func(id int) {
                defer wg.Done()
                
                operation := func() error {
                    if id%3 == 0 {
                        return fmt.Errorf("failure from goroutine %d", id)
                    }
                    return nil
                }
                
                err := cb.Execute(operation)
                results <- err
            }(i)
        }
        
        wg.Wait()
        close(results)
        
        successCount := 0
        failureCount := 0
        openCount := 0
        
        for err := range results {
            if err == nil {
                successCount++
            } else if err.Error() == "circuit breaker is open" {
                openCount++
            } else {
                failureCount++
            }
        }
        
        t.Logf("Results: %d success, %d failures, %d open", successCount, failureCount, openCount)
        
        if successCount+failureCount+openCount != numGoroutines {
            t.Error("Total results don't match goroutine count")
        }
    })
    
    t.Run("PropertyBasedCircuitBreakerTesting", func(t *testing.T) {
        // Property: Circuit breaker should never allow more than maxFailures consecutive failures
        cb := NewCircuitBreaker(3, 50*time.Millisecond)
        
        consecutiveFailures := 0
        maxConsecutiveFailures := 0
        
        for i := 0; i < 100; i++ {
            operation := func() error {
                return fmt.Errorf("test failure")
            }
            
            err := cb.Execute(operation)
            if err != nil && err.Error() != "circuit breaker is open" {
                consecutiveFailures++
            } else {
                if consecutiveFailures > maxConsecutiveFailures {
                    maxConsecutiveFailures = consecutiveFailures
                }
                consecutiveFailures = 0
                
                // Reset circuit by waiting
                time.Sleep(60 * time.Millisecond)
            }
        }
        
        if maxConsecutiveFailures > 3 {
            t.Errorf("Circuit breaker allowed %d consecutive failures, max should be 3", maxConsecutiveFailures)
        }
    })
    
    t.Run("StatefulServiceTesting", func(t *testing.T) {
        // Test service that maintains state across operations
        service := NewRetryableService(2, 5*time.Millisecond)
        
        // Verify initial state
        if service.circuitBreaker.GetState() != Closed {
            t.Error("Service should start with closed circuit breaker")
        }
        
        // Perform operations that change state
        ctx := context.Background()
        
        // Successful operation
        err := service.CallWithRetry(ctx, func() error { return nil })
        if err != nil {
            t.Errorf("Unexpected error from successful operation: %v", err)
        }
        
        // Multiple failing operations to open circuit
        for i := 0; i < 3; i++ {
            service.CallWithRetry(ctx, func() error { return fmt.Errorf("failure") })
        }
        
        // Verify circuit is open
        if service.circuitBreaker.GetState() != Open {
            t.Error("Circuit breaker should be open after multiple failures")
        }
        
        // Attempt operation while circuit is open
        err = service.CallWithRetry(ctx, func() error { return nil })
        if err == nil || !contains(err.Error(), "circuit breaker is open") {
            t.Error("Expected circuit breaker open error")
        }
    })
}

func TestEdgeCaseScenarios(t *testing.T) {
    t.Run("ZeroRetryConfiguration", func(t *testing.T) {
        service := NewRetryableService(0, 1*time.Millisecond)
        ctx := context.Background()
        
        attemptCount := 0
        operation := func() error {
            attemptCount++
            return fmt.Errorf("always fails")
        }
        
        err := service.CallWithRetry(ctx, operation)
        if err == nil {
            t.Error("Expected error from failing operation")
        }
        
        if attemptCount != 1 {
            t.Errorf("Expected exactly 1 attempt with 0 retries, got %d", attemptCount)
        }
    })
    
    t.Run("ImmediateContextCancellation", func(t *testing.T) {
        service := NewRetryableService(3, 10*time.Millisecond)
        ctx, cancel := context.WithCancel(context.Background())
        cancel() // Cancel immediately
        
        operation := func() error {
            return fmt.Errorf("operation")
        }
        
        err := service.CallWithRetry(ctx, operation)
        if err == nil {
            t.Error("Expected error from cancelled context")
        }
        
        if !contains(err.Error(), "cancelled") {
            t.Errorf("Expected cancellation error, got: %v", err)
        }
    })
    
    t.Run("CircuitBreakerBoundaryConditions", func(t *testing.T) {
        // Test with maxFailures = 1
        cb := NewCircuitBreaker(1, 10*time.Millisecond)
        
        // Single failure should open circuit
        err := cb.Execute(func() error { return fmt.Errorf("failure") })
        if err == nil {
            t.Error("Expected error from failing operation")
        }
        
        if cb.GetState() != Open {
            t.Error("Circuit should be open after single failure with maxFailures=1")
        }
        
        // Test with maxFailures = 0 (edge case)
        cb0 := NewCircuitBreaker(0, 10*time.Millisecond)
        
        // Any failure should immediately open circuit
        err = cb0.Execute(func() error { return fmt.Errorf("failure") })
        if err == nil {
            t.Error("Expected error from failing operation")
        }
        
        if cb0.GetState() != Open {
            t.Error("Circuit should be open immediately with maxFailures=0")
        }
    })
}
```

Advanced testing patterns combine multiple techniques like circuit breakers,  
retry logic, context cancellation, and state management to create comprehensive  
test suites for complex systems. These patterns help validate sophisticated  
behaviors and edge cases in distributed and resilient applications.  