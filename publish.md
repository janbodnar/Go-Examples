# Building and Publishing Go Packages

Building and publishing Go packages is a fundamental skill for creating  
reusable code libraries that can be shared across projects and with the  
broader Go community. Go's package system provides powerful tools for code  
organization, dependency management, and distribution through its module  
system and the pkg.go.dev package registry.

The Go ecosystem encourages sharing useful code through well-designed  
packages that follow established conventions. A well-crafted package  
provides a clear API, comprehensive documentation, proper testing, and  
semantic versioning. Publishing packages to pkg.go.dev makes them  
discoverable and accessible to developers worldwide, contributing to the  
rich ecosystem of Go libraries.

Package publishing involves several key aspects: designing clean APIs,  
writing comprehensive documentation, implementing thorough testing,  
following semantic versioning principles, and understanding the publishing  
workflow. Modern Go development relies heavily on these shared packages,  
making package authoring an essential skill for any serious Go developer.

The process begins with creating a well-structured module that exports  
useful functionality through a clean public interface. Documentation  
plays a crucial role in package adoption, as it helps users understand  
how to integrate and use the package effectively. Testing ensures  
reliability and helps maintain backward compatibility across versions.

Publishing to pkg.go.dev happens automatically when you create git tags  
following semantic versioning conventions. The Go module proxy system  
ensures fast, reliable access to published packages while providing  
security through cryptographic checksums. Understanding this workflow  
enables you to contribute valuable packages to the Go ecosystem.


## Basic package structure

A well-organized package starts with clear directory structure and proper  
file organization that follows Go conventions.  

```go
// mathutils/mathutils.go
package mathutils

import (
    "errors"
    "math"
)

// Version represents the package version
const Version = "1.0.0"

// ErrDivisionByZero is returned when attempting division by zero
var ErrDivisionByZero = errors.New("division by zero")

// Add performs addition of two numbers
func Add(a, b float64) float64 {
    return a + b
}

// Subtract performs subtraction of two numbers
func Subtract(a, b float64) float64 {
    return a - b
}

// Multiply performs multiplication of two numbers
func Multiply(a, b float64) float64 {
    return a * b
}

// Divide performs division with error handling for zero denominator
func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, ErrDivisionByZero
    }
    return a / b, nil
}

// Sqrt returns the square root of a number
func Sqrt(x float64) (float64, error) {
    if x < 0 {
        return 0, errors.New("square root of negative number")
    }
    return math.Sqrt(x), nil
}
```

This example demonstrates fundamental package structure with clear function  
naming, proper error handling, exported constants, and comprehensive  
documentation. Each exported function includes a comment explaining its  
purpose and behavior.  

## Package documentation

Comprehensive documentation is essential for package adoption and proper  
usage. Go's documentation tools extract comments automatically.  

```go
// Package mathutils provides mathematical utility functions for common
// calculations including basic arithmetic, statistical operations, and
// advanced mathematical functions.
//
// The package emphasizes safety by providing proper error handling for
// edge cases such as division by zero and invalid inputs.
//
// Example usage:
//     result := mathutils.Add(10, 20)
//     quotient, err := mathutils.Divide(10, 2)
//     if err != nil {
//         log.Fatal(err)
//     }
package mathutils

import (
    "errors"
    "math"
    "sort"
)

// Statistical represents a collection for statistical calculations
type Statistical struct {
    values []float64
}

// NewStatistical creates a new statistical calculator with provided values
func NewStatistical(values []float64) *Statistical {
    // Create a copy to avoid modifying the original slice
    valuesCopy := make([]float64, len(values))
    copy(valuesCopy, values)
    return &Statistical{values: valuesCopy}
}

// Mean calculates the arithmetic mean of the values
func (s *Statistical) Mean() float64 {
    if len(s.values) == 0 {
        return 0
    }
    
    sum := 0.0
    for _, value := range s.values {
        sum += value
    }
    return sum / float64(len(s.values))
}

// Median calculates the median value
func (s *Statistical) Median() float64 {
    if len(s.values) == 0 {
        return 0
    }
    
    // Sort a copy to avoid modifying original values
    sorted := make([]float64, len(s.values))
    copy(sorted, s.values)
    sort.Float64s(sorted)
    
    n := len(sorted)
    if n%2 == 0 {
        return (sorted[n/2-1] + sorted[n/2]) / 2
    }
    return sorted[n/2]
}

// StandardDeviation calculates the population standard deviation
func (s *Statistical) StandardDeviation() float64 {
    if len(s.values) <= 1 {
        return 0
    }
    
    mean := s.Mean()
    sumSquaredDiffs := 0.0
    
    for _, value := range s.values {
        diff := value - mean
        sumSquaredDiffs += diff * diff
    }
    
    variance := sumSquaredDiffs / float64(len(s.values))
    return math.Sqrt(variance)
}

// Range returns the difference between the maximum and minimum values
func (s *Statistical) Range() float64 {
    if len(s.values) == 0 {
        return 0
    }
    
    min, max := s.values[0], s.values[0]
    for _, value := range s.values[1:] {
        if value < min {
            min = value
        }
        if value > max {
            max = value
        }
    }
    return max - min
}
```

Good documentation includes package-level comments explaining purpose and  
usage patterns, detailed function comments describing behavior and  
parameters, and practical examples. The godoc tool generates beautiful  
documentation from these comments automatically.  

## Package testing

Comprehensive testing ensures package reliability and helps maintain  
backward compatibility across versions.  

```go
// mathutils/mathutils_test.go
package mathutils

import (
    "math"
    "testing"
)

func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     float64
        expected float64
    }{
        {"positive numbers", 5.5, 2.3, 7.8},
        {"negative numbers", -3.2, -1.8, -5.0},
        {"mixed signs", 10.0, -3.0, 7.0},
        {"with zero", 5.0, 0.0, 5.0},
        {"large numbers", 1e10, 1e10, 2e10},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if math.Abs(result-tt.expected) > 1e-9 {
                t.Errorf("Add(%f, %f) = %f, expected %f", 
                         tt.a, tt.b, result, tt.expected)
            }
        })
    }
}

func TestDivide(t *testing.T) {
    // Test successful division
    result, err := Divide(10, 2)
    if err != nil {
        t.Errorf("Divide(10, 2) returned error: %v", err)
    }
    if result != 5.0 {
        t.Errorf("Divide(10, 2) = %f, expected 5.0", result)
    }
    
    // Test division by zero
    _, err = Divide(10, 0)
    if err != ErrDivisionByZero {
        t.Errorf("Divide(10, 0) should return ErrDivisionByZero, got %v", err)
    }
}

func TestStatistical(t *testing.T) {
    values := []float64{1, 2, 3, 4, 5}
    stats := NewStatistical(values)
    
    // Test mean
    expectedMean := 3.0
    if mean := stats.Mean(); mean != expectedMean {
        t.Errorf("Mean() = %f, expected %f", mean, expectedMean)
    }
    
    // Test median
    expectedMedian := 3.0
    if median := stats.Median(); median != expectedMedian {
        t.Errorf("Median() = %f, expected %f", median, expectedMedian)
    }
    
    // Test range
    expectedRange := 4.0
    if r := stats.Range(); r != expectedRange {
        t.Errorf("Range() = %f, expected %f", r, expectedRange)
    }
}

// Benchmark tests for performance measurement
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(123.456, 789.012)
    }
}

func BenchmarkStatisticalMean(b *testing.B) {
    values := make([]float64, 1000)
    for i := range values {
        values[i] = float64(i)
    }
    stats := NewStatistical(values)
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        stats.Mean()
    }
}

// Example tests that serve as documentation
func ExampleAdd() {
    result := Add(10.5, 20.3)
    // Output will be approximately 30.8
    _ = result
}

func ExampleStatistical_Mean() {
    values := []float64{1, 2, 3, 4, 5}
    stats := NewStatistical(values)
    mean := stats.Mean()
    // Mean will be 3.0
    _ = mean
}
```

Testing patterns include table-driven tests for comprehensive coverage,  
error case testing, benchmark tests for performance measurement, and  
example tests that serve as executable documentation. These tests ensure  
package reliability and provide usage examples.  

## Module initialization and structure

Proper module setup establishes the foundation for package publishing  
and dependency management.  

```go
// Initialize a new module
// Terminal: go mod init github.com/username/mathutils

// go.mod file content
module github.com/username/mathutils

go 1.21

require (
    golang.org/x/tools v0.13.0
)

// Project structure:
// mathutils/
// ├── go.mod
// ├── go.sum
// ├── README.md
// ├── LICENSE
// ├── mathutils.go
// ├── mathutils_test.go
// ├── examples/
// │   └── main.go
// └── internal/
//     └── helpers.go
```

```go
// examples/main.go - Usage examples for users
package main

import (
    "fmt"
    "log"
    
    "github.com/username/mathutils"
)

func main() {
    fmt.Println("Mathematical Utilities Example")
    fmt.Println("==============================")
    
    // Basic arithmetic
    fmt.Printf("Addition: %.2f\n", mathutils.Add(10.5, 5.3))
    fmt.Printf("Subtraction: %.2f\n", mathutils.Subtract(10.5, 5.3))
    fmt.Printf("Multiplication: %.2f\n", mathutils.Multiply(10.5, 5.3))
    
    // Division with error handling
    quotient, err := mathutils.Divide(10.5, 5.3)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Division: %.2f\n", quotient)
    
    // Statistical calculations
    values := []float64{1.5, 2.8, 3.2, 4.7, 5.1, 6.9, 7.3, 8.6, 9.2, 10.4}
    stats := mathutils.NewStatistical(values)
    
    fmt.Printf("\nStatistical Analysis of %v:\n", values)
    fmt.Printf("Mean: %.2f\n", stats.Mean())
    fmt.Printf("Median: %.2f\n", stats.Median())
    fmt.Printf("Standard Deviation: %.2f\n", stats.StandardDeviation())
    fmt.Printf("Range: %.2f\n", stats.Range())
    
    // Error handling demonstration
    _, err = mathutils.Sqrt(-4)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    }
}
```

Module structure includes proper directory organization, comprehensive  
README documentation, appropriate license files, and example usage.  
The examples directory provides practical demonstrations for package  
users.  

## Publishing workflow

Publishing Go packages involves version tagging, git repository  
management, and understanding the pkg.go.dev automatic discovery.  

```go
// README.md content for package documentation
/*
# Mathematical Utilities

A comprehensive Go package providing mathematical utility functions for
common calculations including arithmetic operations, statistical analysis,
and error-safe mathematical functions.

## Installation

```bash
go get github.com/username/mathutils
```

## Quick Start

```go
package main

import (
    "fmt"
    "github.com/username/mathutils"
)

func main() {
    result := mathutils.Add(10, 20)
    fmt.Println("Result:", result)
    
    values := []float64{1, 2, 3, 4, 5}
    stats := mathutils.NewStatistical(values)
    fmt.Printf("Mean: %.2f\n", stats.Mean())
}
```

## Features

- Basic arithmetic operations with error handling
- Statistical calculations (mean, median, standard deviation)
- Safe mathematical functions with proper error reporting
- Comprehensive test coverage
- Benchmark tests for performance monitoring

## Documentation

Full documentation is available at [pkg.go.dev](https://pkg.go.dev/github.com/username/mathutils).

## Contributing

Contributions are welcome! Please read our contributing guidelines and
ensure all tests pass before submitting pull requests.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
*/

// Terminal commands for publishing:
/*
# 1. Ensure all tests pass
go test ./...

# 2. Run go mod tidy to clean dependencies
go mod tidy

# 3. Create and push git tag for version
git tag v1.0.0
git push origin v1.0.0

# 4. Package automatically appears on pkg.go.dev
# Users can then import with:
# go get github.com/username/mathutils
*/
```

```go
// internal/helpers.go - Internal package utilities
package internal

import "math"

// Round rounds a float64 to a specified number of decimal places
func Round(val float64, decimals int) float64 {
    multiplier := math.Pow(10, float64(decimals))
    return math.Round(val*multiplier) / multiplier
}

// IsEqual compares two float64 values with epsilon tolerance
func IsEqual(a, b, epsilon float64) bool {
    return math.Abs(a-b) < epsilon
}
```

The publishing workflow involves creating proper documentation, ensuring  
test coverage, following semantic versioning, and using git tags for  
version management. The pkg.go.dev service automatically discovers and  
indexes tagged versions.  

## Semantic versioning

Proper versioning communicates API changes and helps users understand  
compatibility expectations when upgrading packages.  

```go
// version.go - Package version management
package mathutils

import (
    "fmt"
    "strconv"
    "strings"
)

// Version information
const (
    Major = 1
    Minor = 2
    Patch = 0
    
    // Version string representation
    Version = "1.2.0"
    
    // API level for backward compatibility tracking
    APILevel = 2
)

// VersionInfo provides detailed version information
type VersionInfo struct {
    Major    int    `json:"major"`
    Minor    int    `json:"minor"`
    Patch    int    `json:"patch"`
    Full     string `json:"full"`
    APILevel int    `json:"api_level"`
}

// GetVersionInfo returns complete version information
func GetVersionInfo() VersionInfo {
    return VersionInfo{
        Major:    Major,
        Minor:    Minor,
        Patch:    Patch,
        Full:     Version,
        APILevel: APILevel,
    }
}

// ParseVersion parses a semantic version string
func ParseVersion(version string) (major, minor, patch int, err error) {
    // Remove 'v' prefix if present
    version = strings.TrimPrefix(version, "v")
    
    parts := strings.Split(version, ".")
    if len(parts) != 3 {
        return 0, 0, 0, fmt.Errorf("invalid version format: %s", version)
    }
    
    major, err = strconv.Atoi(parts[0])
    if err != nil {
        return 0, 0, 0, fmt.Errorf("invalid major version: %s", parts[0])
    }
    
    minor, err = strconv.Atoi(parts[1])
    if err != nil {
        return 0, 0, 0, fmt.Errorf("invalid minor version: %s", parts[1])
    }
    
    patch, err = strconv.Atoi(parts[2])
    if err != nil {
        return 0, 0, 0, fmt.Errorf("invalid patch version: %s", parts[2])
    }
    
    return major, minor, patch, nil
}

// IsCompatible checks if two versions are compatible
func IsCompatible(required, available string) (bool, error) {
    reqMajor, reqMinor, _, err := ParseVersion(required)
    if err != nil {
        return false, err
    }
    
    availMajor, availMinor, _, err := ParseVersion(available)
    if err != nil {
        return false, err
    }
    
    // Major version must match for compatibility
    if reqMajor != availMajor {
        return false, nil
    }
    
    // Available minor version must be >= required
    return availMinor >= reqMinor, nil
}
```

```go
// Example of version-aware API changes
package mathutils

// Legacy function maintained for backward compatibility
// Deprecated: Use Add instead
func AddNumbers(a, b float64) float64 {
    return Add(a, b)
}

// Enhanced function with additional features in v1.2.0
func AddWithPrecision(a, b float64, precision int) float64 {
    result := Add(a, b)
    multiplier := 1.0
    for i := 0; i < precision; i++ {
        multiplier *= 10
    }
    return float64(int(result*multiplier)) / multiplier
}
```

Semantic versioning follows MAJOR.MINOR.PATCH format where MAJOR indicates  
breaking changes, MINOR adds backward-compatible features, and PATCH  
includes backward-compatible fixes. This system helps users understand  
the impact of upgrades.  

## Advanced package patterns

Sophisticated packages often implement advanced patterns for  
configuration, extensibility, and maintainability.  

```go
// config.go - Package configuration
package mathutils

import (
    "sync"
    "time"
)

// Config holds package-wide configuration
type Config struct {
    Precision     int           `json:"precision"`
    Timeout       time.Duration `json:"timeout"`
    EnableLogging bool          `json:"enable_logging"`
    MaxIterations int           `json:"max_iterations"`
}

// DefaultConfig provides sensible defaults
var DefaultConfig = Config{
    Precision:     6,
    Timeout:       5 * time.Second,
    EnableLogging: false,
    MaxIterations: 1000,
}

var (
    currentConfig Config
    configMutex   sync.RWMutex
)

func init() {
    currentConfig = DefaultConfig
}

// SetConfig updates the package configuration thread-safely
func SetConfig(config Config) {
    configMutex.Lock()
    defer configMutex.Unlock()
    currentConfig = config
}

// GetConfig returns current configuration thread-safely
func GetConfig() Config {
    configMutex.RLock()
    defer configMutex.RUnlock()
    return currentConfig
}
```

```go
// calculator.go - Advanced calculator with plugin support
package mathutils

import (
    "context"
    "fmt"
    "sync"
)

// Operation represents a mathematical operation
type Operation interface {
    Name() string
    Execute(args ...float64) (float64, error)
    Validate(args ...float64) error
}

// Calculator provides extensible calculation capabilities
type Calculator struct {
    operations map[string]Operation
    mutex      sync.RWMutex
    config     Config
}

// NewCalculator creates a calculator with default operations
func NewCalculator() *Calculator {
    calc := &Calculator{
        operations: make(map[string]Operation),
        config:     GetConfig(),
    }
    
    // Register default operations
    calc.RegisterOperation(&AddOperation{})
    calc.RegisterOperation(&MultiplyOperation{})
    calc.RegisterOperation(&PowerOperation{})
    
    return calc
}

// RegisterOperation adds a new operation to the calculator
func (c *Calculator) RegisterOperation(op Operation) {
    c.mutex.Lock()
    defer c.mutex.Unlock()
    c.operations[op.Name()] = op
}

// Calculate performs the specified operation with context support
func (c *Calculator) Calculate(ctx context.Context, operation string, args ...float64) (float64, error) {
    c.mutex.RLock()
    op, exists := c.operations[operation]
    c.mutex.RUnlock()
    
    if !exists {
        return 0, fmt.Errorf("unknown operation: %s", operation)
    }
    
    if err := op.Validate(args...); err != nil {
        return 0, fmt.Errorf("validation failed: %w", err)
    }
    
    // Create a channel to receive the result
    resultChan := make(chan struct {
        result float64
        err    error
    }, 1)
    
    go func() {
        result, err := op.Execute(args...)
        resultChan <- struct {
            result float64
            err    error
        }{result, err}
    }()
    
    select {
    case res := <-resultChan:
        return res.result, res.err
    case <-ctx.Done():
        return 0, ctx.Err()
    }
}

// ListOperations returns available operations
func (c *Calculator) ListOperations() []string {
    c.mutex.RLock()
    defer c.mutex.RUnlock()
    
    operations := make([]string, 0, len(c.operations))
    for name := range c.operations {
        operations = append(operations, name)
    }
    return operations
}

// AddOperation implements basic addition
type AddOperation struct{}

func (ao *AddOperation) Name() string { return "add" }

func (ao *AddOperation) Execute(args ...float64) (float64, error) {
    result := 0.0
    for _, arg := range args {
        result += arg
    }
    return result, nil
}

func (ao *AddOperation) Validate(args ...float64) error {
    if len(args) < 2 {
        return fmt.Errorf("add operation requires at least 2 arguments")
    }
    return nil
}

// MultiplyOperation implements multiplication
type MultiplyOperation struct{}

func (mo *MultiplyOperation) Name() string { return "multiply" }

func (mo *MultiplyOperation) Execute(args ...float64) (float64, error) {
    if len(args) == 0 {
        return 0, nil
    }
    
    result := args[0]
    for _, arg := range args[1:] {
        result *= arg
    }
    return result, nil
}

func (mo *MultiplyOperation) Validate(args ...float64) error {
    if len(args) < 2 {
        return fmt.Errorf("multiply operation requires at least 2 arguments")
    }
    return nil
}

// PowerOperation implements exponentiation
type PowerOperation struct{}

func (po *PowerOperation) Name() string { return "power" }

func (po *PowerOperation) Execute(args ...float64) (float64, error) {
    if len(args) != 2 {
        return 0, fmt.Errorf("power operation requires exactly 2 arguments")
    }
    return math.Pow(args[0], args[1]), nil
}

func (po *PowerOperation) Validate(args ...float64) error {
    if len(args) != 2 {
        return fmt.Errorf("power operation requires exactly 2 arguments")
    }
    return nil
}
```

Advanced patterns include configuration management, plugin architectures,  
context support for cancellation, and thread-safe operations. These  
patterns enable packages to scale and adapt to diverse usage scenarios  
while maintaining clean APIs.  

## Package maintenance

Long-term package maintenance involves monitoring usage, handling issues,  
updating dependencies, and evolving APIs responsibly.  

```go
// maintenance.go - Package health monitoring
package mathutils

import (
    "runtime"
    "sync/atomic"
    "time"
)

// Metrics tracks package usage statistics
type Metrics struct {
    TotalOperations   int64 `json:"total_operations"`
    ErrorCount        int64 `json:"error_count"`
    AverageLatency    int64 `json:"average_latency_ns"`
    ConcurrentUsers   int32 `json:"concurrent_users"`
    LastAccessTime    int64 `json:"last_access_unix"`
    PackageStartTime  int64 `json:"package_start_unix"`
}

var packageMetrics Metrics

func init() {
    packageMetrics.PackageStartTime = time.Now().Unix()
}

// GetMetrics returns current package usage metrics
func GetMetrics() Metrics {
    return Metrics{
        TotalOperations:  atomic.LoadInt64(&packageMetrics.TotalOperations),
        ErrorCount:       atomic.LoadInt64(&packageMetrics.ErrorCount),
        AverageLatency:   atomic.LoadInt64(&packageMetrics.AverageLatency),
        ConcurrentUsers:  atomic.LoadInt32(&packageMetrics.ConcurrentUsers),
        LastAccessTime:   atomic.LoadInt64(&packageMetrics.LastAccessTime),
        PackageStartTime: packageMetrics.PackageStartTime,
    }
}

// RecordOperation tracks successful operations
func recordOperation(duration time.Duration) {
    atomic.AddInt64(&packageMetrics.TotalOperations, 1)
    atomic.StoreInt64(&packageMetrics.LastAccessTime, time.Now().Unix())
    
    // Update average latency using exponential moving average
    currentAvg := atomic.LoadInt64(&packageMetrics.AverageLatency)
    newLatency := duration.Nanoseconds()
    
    if currentAvg == 0 {
        atomic.StoreInt64(&packageMetrics.AverageLatency, newLatency)
    } else {
        // EMA with alpha = 0.1
        newAvg := (9*currentAvg + newLatency) / 10
        atomic.StoreInt64(&packageMetrics.AverageLatency, newAvg)
    }
}

// RecordError tracks operation errors
func recordError() {
    atomic.AddInt64(&packageMetrics.ErrorCount, 1)
    atomic.StoreInt64(&packageMetrics.LastAccessTime, time.Now().Unix())
}

// IncrementUsers tracks concurrent usage
func incrementUsers() {
    atomic.AddInt32(&packageMetrics.ConcurrentUsers, 1)
}

// DecrementUsers decreases concurrent user count
func decrementUsers() {
    atomic.AddInt32(&packageMetrics.ConcurrentUsers, -1)
}

// HealthInfo provides package health information
type HealthInfo struct {
    Version        string  `json:"version"`
    GoVersion      string  `json:"go_version"`
    MemoryUsage    uint64  `json:"memory_usage_bytes"`
    GoroutineCount int     `json:"goroutine_count"`
    Uptime         string  `json:"uptime"`
    ErrorRate      float64 `json:"error_rate"`
}

// GetHealthInfo returns comprehensive package health data
func GetHealthInfo() HealthInfo {
    var memStats runtime.MemStats
    runtime.ReadMemStats(&memStats)
    
    metrics := GetMetrics()
    uptime := time.Since(time.Unix(metrics.PackageStartTime, 0))
    
    var errorRate float64
    if metrics.TotalOperations > 0 {
        errorRate = float64(metrics.ErrorCount) / float64(metrics.TotalOperations)
    }
    
    return HealthInfo{
        Version:        Version,
        GoVersion:      runtime.Version(),
        MemoryUsage:    memStats.Alloc,
        GoroutineCount: runtime.NumGoroutine(),
        Uptime:         uptime.String(),
        ErrorRate:      errorRate,
    }
}

// Enhanced Add function with metrics tracking
func AddWithMetrics(a, b float64) float64 {
    incrementUsers()
    defer decrementUsers()
    
    start := time.Now()
    defer func() {
        recordOperation(time.Since(start))
    }()
    
    return a + b
}
```

```go
// migration.go - API migration utilities
package mathutils

import (
    "fmt"
    "log"
    "os"
)

// DeprecationWarning logs deprecation notices
func deprecationWarning(function, replacement, version string) {
    if GetConfig().EnableLogging {
        log.Printf("DEPRECATION WARNING: %s is deprecated and will be removed in %s. Use %s instead.",
                   function, version, replacement)
    }
}

// MigrationGuide provides structured migration information
type MigrationGuide struct {
    FromVersion string            `json:"from_version"`
    ToVersion   string            `json:"to_version"`
    Changes     []MigrationChange `json:"changes"`
}

// MigrationChange describes a specific API change
type MigrationChange struct {
    Type        string `json:"type"` // "removed", "renamed", "signature_changed"
    OldAPI      string `json:"old_api"`
    NewAPI      string `json:"new_api"`
    Description string `json:"description"`
    Example     string `json:"example"`
}

// GetMigrationGuide returns migration information between versions
func GetMigrationGuide(fromVersion, toVersion string) (MigrationGuide, error) {
    guides := map[string]MigrationGuide{
        "1.0.0->1.1.0": {
            FromVersion: "1.0.0",
            ToVersion:   "1.1.0",
            Changes: []MigrationChange{
                {
                    Type:        "renamed",
                    OldAPI:      "AddNumbers(a, b float64)",
                    NewAPI:      "Add(a, b float64)",
                    Description: "Function renamed for consistency",
                    Example:     "Replace AddNumbers(1, 2) with Add(1, 2)",
                },
            },
        },
    }
    
    key := fmt.Sprintf("%s->%s", fromVersion, toVersion)
    if guide, exists := guides[key]; exists {
        return guide, nil
    }
    
    return MigrationGuide{}, fmt.Errorf("no migration guide available for %s to %s", fromVersion, toVersion)
}

// CheckCompatibility verifies if the current environment supports the package
func CheckCompatibility() error {
    // Check Go version
    goVersion := runtime.Version()
    if goVersion < "go1.19" {
        return fmt.Errorf("this package requires Go 1.19 or later, found %s", goVersion)
    }
    
    // Check environment variables for configuration
    if timeout := os.Getenv("MATHUTILS_TIMEOUT"); timeout != "" {
        log.Printf("Using custom timeout from environment: %s", timeout)
    }
    
    return nil
}
```

Package maintenance includes monitoring usage patterns, tracking errors,  
providing migration guides for API changes, and ensuring compatibility  
across Go versions. These practices help maintain package reliability  
and user satisfaction over time.  