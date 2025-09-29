# Strings

In Go, strings are immutable sequences of bytes that represent text data. They are  
one of the fundamental data types and are extensively used for text processing,  
input/output operations, and data manipulation. Go strings are UTF-8 encoded by  
default, which means they can contain any Unicode character while maintaining  
compatibility with ASCII text.

Strings in Go are implemented as a sequence of bytes with a length field. The  
immutable nature means that once a string is created, its content cannot be  
modified directly. Any operation that appears to modify a string actually creates  
a new string. This design choice provides memory safety and prevents common  
string-related bugs found in other languages.

Go provides extensive support for string operations through the built-in functions  
and the `strings` package from the standard library. Common operations include  
concatenation, substring extraction, searching, replacing, splitting, and joining.  
The language also supports string literals using double quotes for interpreted  
strings and backticks for raw strings that preserve exact formatting.

String manipulation in Go is efficient due to the language's design. The `strings.Builder`  
type provides an efficient way to build strings incrementally, while string slicing  
operations create views of the original string data without copying when possible.  
Understanding these characteristics is crucial for writing performant Go code that  
processes text data effectively.


## String literals

String literals can be created using double quotes or backticks. Double quotes  
create interpreted strings that process escape sequences, while backticks create  
raw strings that preserve all characters literally.  

```go
package main

import "fmt"

func main() {
    // Interpreted string with escape sequences
    message := "Hello there\nThis is a new line"
    fmt.Println(message)

    // Raw string preserves formatting
    path := `C:\Users\John\Documents\file.txt`
    fmt.Println(path)

    // Multi-line raw string
    poem := `The sky is blue,
The grass is green,
Nature's beauty can be seen.`
    fmt.Println(poem)
}
```

## String length and indexing

Strings have a length that can be accessed using the `len()` function. Individual  
bytes can be accessed using indexing, but be careful with UTF-8 encoded characters  
that may span multiple bytes.  

```go
package main

import "fmt"

func main() {
    text := "Go programming"
    fmt.Printf("Length: %d\n", len(text))

    // Access individual bytes
    fmt.Printf("First byte: %c\n", text[0])
    fmt.Printf("Last byte: %c\n", text[len(text)-1])

    // UTF-8 example
    greeting := "Hello 世界"
    fmt.Printf("Byte length: %d\n", len(greeting))
    fmt.Printf("First byte of 世: %d\n", greeting[6])
}
```

## String concatenation

Strings can be concatenated using the `+` operator or the `+=` assignment operator.  
For multiple concatenations, consider using `strings.Builder` for better performance.  

```go
package main

import "fmt"

func main() {
    first := "Go"
    second := "lang"
    
    // Basic concatenation
    combined := first + second
    fmt.Println(combined)

    // Concatenation with assignment
    result := "Programming in "
    result += first
    fmt.Println(result)

    // Multiple concatenations
    full := "Learning " + first + " is " + "fun"
    fmt.Println(full)
}
```

## String comparison

Strings can be compared using standard comparison operators. The comparison  
is lexicographic and case-sensitive based on byte values.  

```go
package main

import "fmt"

func main() {
    str1 := "apple"
    str2 := "banana"
    str3 := "apple"

    fmt.Printf("%s == %s: %t\n", str1, str3, str1 == str3)
    fmt.Printf("%s < %s: %t\n", str1, str2, str1 < str2)
    fmt.Printf("%s > %s: %t\n", str2, str1, str2 > str1)

    // Case sensitivity
    fmt.Printf("Apple == apple: %t\n", "Apple" == "apple")
}
```

## String slicing

String slicing creates a substring by specifying start and end indices. The  
resulting slice shares the same underlying data as the original string.  

```go
package main

import "fmt"

func main() {
    text := "Go programming language"
    
    // Extract substrings
    fmt.Println(text[0:2])   // "Go"
    fmt.Println(text[3:14])  // "programming"
    fmt.Println(text[15:])   // "language"
    fmt.Println(text[:2])    // "Go"
    
    // Full slice
    fmt.Println(text[:])     // entire string
}
```

## Iterating over strings

Strings can be iterated character by character using a for loop with range.  
This correctly handles UTF-8 encoded characters as runes.  

```go
package main

import "fmt"

func main() {
    text := "Go语言"
    
    // Iterate by runes (characters)
    for i, char := range text {
        fmt.Printf("Index %d: %c\n", i, char)
    }
    
    fmt.Println("---")
    
    // Iterate by bytes
    for i := 0; i < len(text); i++ {
        fmt.Printf("Byte %d: %d\n", i, text[i])
    }
}
```

## String contains check

The `strings.Contains` function checks if a substring exists within a string.  
It returns a boolean value indicating the presence of the substring.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go is a powerful programming language"
    
    fmt.Printf("Contains 'Go': %t\n", strings.Contains(text, "Go"))
    fmt.Printf("Contains 'Python': %t\n", strings.Contains(text, "Python"))
    fmt.Printf("Contains 'program': %t\n", strings.Contains(text, "program"))
    
    // Case sensitive
    fmt.Printf("Contains 'go': %t\n", strings.Contains(text, "go"))
}
```

## String prefix and suffix

The `strings.HasPrefix` and `strings.HasSuffix` functions check if a string  
starts or ends with a specific substring respectively.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    filename := "document.txt"
    url := "https://example.com"
    
    fmt.Printf("'%s' has .txt suffix: %t\n", filename, 
               strings.HasSuffix(filename, ".txt"))
    fmt.Printf("'%s' has https prefix: %t\n", url, 
               strings.HasPrefix(url, "https"))
    
    // Multiple checks
    extensions := []string{".txt", ".doc", ".pdf"}
    for _, ext := range extensions {
        if strings.HasSuffix(filename, ext) {
            fmt.Printf("File has %s extension\n", ext)
        }
    }
}
```

## String case conversion

The `strings` package provides functions for converting string case, including  
uppercase, lowercase, and title case transformations.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go Programming Language"
    
    fmt.Printf("Original: %s\n", text)
    fmt.Printf("Uppercase: %s\n", strings.ToUpper(text))
    fmt.Printf("Lowercase: %s\n", strings.ToLower(text))
    fmt.Printf("Title case: %s\n", strings.ToTitle(text))
    
    // Title case (proper capitalization)
    sentence := "the quick brown fox"
    fmt.Printf("Title: %s\n", strings.Title(sentence))
}
```

## String splitting

The `strings.Split` function divides a string into a slice of substrings based  
on a delimiter. Various splitting functions handle different scenarios.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    data := "apple,banana,cherry,date"
    fruits := strings.Split(data, ",")
    
    fmt.Printf("Original: %s\n", data)
    fmt.Printf("Split: %v\n", fruits)
    
    // Split with limit
    limited := strings.SplitN(data, ",", 2)
    fmt.Printf("Split with limit: %v\n", limited)
    
    // Split by whitespace
    sentence := "Go is awesome"
    words := strings.Fields(sentence)
    fmt.Printf("Words: %v\n", words)
}
```

## String joining

The `strings.Join` function combines a slice of strings into a single string  
using a specified separator between elements.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    words := []string{"Go", "is", "a", "great", "language"}
    
    // Join with space
    sentence := strings.Join(words, " ")
    fmt.Printf("Sentence: %s\n", sentence)
    
    // Join with different separators
    csv := strings.Join(words, ",")
    fmt.Printf("CSV: %s\n", csv)
    
    path := strings.Join([]string{"usr", "local", "bin"}, "/")
    fmt.Printf("Path: /%s\n", path)
}
```

## String replacement

The `strings.Replace` and `strings.ReplaceAll` functions substitute occurrences  
of a substring with another string. Replace allows limiting the number of  
replacements while ReplaceAll replaces all occurrences.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go Go Go programming"
    
    // Replace first occurrence
    result1 := strings.Replace(text, "Go", "Java", 1)
    fmt.Printf("Replace 1: %s\n", result1)
    
    // Replace all occurrences
    result2 := strings.ReplaceAll(text, "Go", "Python")
    fmt.Printf("Replace all: %s\n", result2)
    
    // Chain replacements
    message := "I like cats and cats like me"
    result3 := strings.ReplaceAll(message, "cats", "dogs")
    fmt.Printf("Chain: %s\n", result3)
}
```

## String trimming

String trimming functions remove specified characters from the beginning and/or  
end of strings. Common operations include removing whitespace and custom characters.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "  Go programming  "
    
    fmt.Printf("Original: '%s'\n", text)
    fmt.Printf("Trimmed: '%s'\n", strings.TrimSpace(text))
    fmt.Printf("Left trim: '%s'\n", strings.TrimLeft(text, " "))
    fmt.Printf("Right trim: '%s'\n", strings.TrimRight(text, " "))
    
    // Custom character trimming
    data := "###Go###"
    fmt.Printf("Trim #: '%s'\n", strings.Trim(data, "#"))
    
    // Trim prefix/suffix
    url := "https://example.com"
    fmt.Printf("Trim prefix: '%s'\n", strings.TrimPrefix(url, "https://"))
}
```

## String builder

The `strings.Builder` type provides an efficient way to construct strings  
incrementally. It minimizes memory allocations and copying operations.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var builder strings.Builder
    
    // Build string incrementally
    builder.WriteString("Building")
    builder.WriteString(" a")
    builder.WriteString(" string")
    builder.WriteString(" efficiently")
    
    result := builder.String()
    fmt.Printf("Result: %s\n", result)
    
    // Reset and reuse
    builder.Reset()
    for i := 1; i <= 5; i++ {
        builder.WriteString(fmt.Sprintf("Item %d ", i))
    }
    
    fmt.Printf("Items: %s\n", builder.String())
}
```

## String formatting

String formatting with `fmt.Sprintf` creates formatted strings using format  
specifiers. It's useful for creating complex string representations with  
embedded values.  

```go
package main

import "fmt"

func main() {
    name := "Alice"
    age := 30
    score := 95.5
    
    // Basic formatting
    message := fmt.Sprintf("%s is %d years old", name, age)
    fmt.Println(message)
    
    // Various format specifiers
    formatted := fmt.Sprintf("Name: %s, Age: %d, Score: %.1f%%", 
                           name, age, score)
    fmt.Println(formatted)
    
    // Width and padding
    padded := fmt.Sprintf("|%10s|%5d|%8.2f|", name, age, score)
    fmt.Println(padded)
}
```

## String index finding

The `strings.Index` and related functions find the position of substrings  
within strings. They return -1 if the substring is not found.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go programming is fun and Go is powerful"
    
    // Find first occurrence
    pos := strings.Index(text, "Go")
    fmt.Printf("First 'Go' at position: %d\n", pos)
    
    // Find last occurrence
    lastPos := strings.LastIndex(text, "Go")
    fmt.Printf("Last 'Go' at position: %d\n", lastPos)
    
    // Find any character from set
    vowelPos := strings.IndexAny(text, "aeiou")
    fmt.Printf("First vowel at position: %d\n", vowelPos)
    
    // Not found case
    notFound := strings.Index(text, "Python")
    fmt.Printf("'Python' position: %d\n", notFound)
}
```

## String counting

The `strings.Count` function counts the number of non-overlapping occurrences  
of a substring within a string.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go gopher programming in Go language"
    
    // Count occurrences
    goCount := strings.Count(text, "Go")
    fmt.Printf("'Go' appears %d times\n", goCount)
    
    gCount := strings.Count(text, "g")
    fmt.Printf("'g' appears %d times\n", gCount)
    
    // Count overlapping patterns
    pattern := "programming language"
    progCount := strings.Count(pattern, "g")
    fmt.Printf("'g' in '%s': %d times\n", pattern, progCount)
    
    // Empty string count
    emptyCount := strings.Count(text, "")
    fmt.Printf("Empty string count: %d\n", emptyCount)
}
```

## String runes

Working with runes (Unicode code points) is essential for proper handling  
of international characters in strings. Runes represent individual characters.  

```go
package main

import "fmt"

func main() {
    text := "Hello 世界 🌍"
    
    // Convert to rune slice
    runes := []rune(text)
    fmt.Printf("String: %s\n", text)
    fmt.Printf("Byte length: %d\n", len(text))
    fmt.Printf("Rune length: %d\n", len(runes))
    
    // Access individual runes
    for i, r := range runes {
        fmt.Printf("Rune %d: %c (Unicode: %U)\n", i, r, r)
    }
    
    // Convert back to string
    reconstructed := string(runes)
    fmt.Printf("Reconstructed: %s\n", reconstructed)
}
```

## String byte operations

Converting strings to byte slices enables low-level manipulation and interaction  
with APIs that work with byte data.  

```go
package main

import "fmt"

func main() {
    text := "Go bytes"
    
    // Convert to byte slice
    bytes := []byte(text)
    fmt.Printf("Original: %s\n", text)
    fmt.Printf("Bytes: %v\n", bytes)
    
    // Modify bytes (creates new string when converted back)
    bytes[0] = 'N'
    bytes[1] = 'o'
    
    modified := string(bytes)
    fmt.Printf("Modified: %s\n", modified)
    fmt.Printf("Original unchanged: %s\n", text)
    
    // Hex representation
    fmt.Printf("Hex: %x\n", []byte(text))
}
```

## String field extraction

The `strings.Fields` function splits a string into fields separated by  
whitespace, while `strings.FieldsFunc` allows custom separator logic.  

```go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

func main() {
    text := "  Go   programming    language  "
    
    // Split by whitespace
    fields := strings.Fields(text)
    fmt.Printf("Fields: %v\n", fields)
    
    // Custom field separator
    data := "apple123banana456cherry"
    customFields := strings.FieldsFunc(data, func(r rune) bool {
        return unicode.IsDigit(r)
    })
    fmt.Printf("Custom fields: %v\n", customFields)
    
    // Count fields
    fmt.Printf("Number of fields: %d\n", len(fields))
}
```

## String map function

The `strings.Map` function transforms each rune in a string according to  
a mapping function, allowing character-by-character transformations.  

```go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

func main() {
    text := "Go Programming 123"
    
    // Convert to uppercase
    upper := strings.Map(func(r rune) rune {
        return unicode.ToUpper(r)
    }, text)
    fmt.Printf("Uppercase: %s\n", upper)
    
    // Remove digits
    noDigits := strings.Map(func(r rune) rune {
        if unicode.IsDigit(r) {
            return -1 // Remove character
        }
        return r
    }, text)
    fmt.Printf("No digits: %s\n", noDigits)
    
    // ROT13 transformation
    rot13 := strings.Map(func(r rune) rune {
        if r >= 'A' && r <= 'Z' {
            return 'A' + (r-'A'+13)%26
        }
        if r >= 'a' && r <= 'z' {
            return 'a' + (r-'a'+13)%26
        }
        return r
    }, "Hello World")
    fmt.Printf("ROT13: %s\n", rot13)
}
```

## String reader

The `strings.Reader` type implements io.Reader interface for strings,  
enabling string data to be used with functions expecting readers.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    text := "Go programming language"
    reader := strings.NewReader(text)
    
    // Read in chunks
    buffer := make([]byte, 8)
    for {
        n, err := reader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            break
        }
        fmt.Printf("Read %d bytes: %s\n", n, buffer[:n])
    }
    
    // Reset and read all
    reader.Reset(text)
    allData, _ := io.ReadAll(reader)
    fmt.Printf("All data: %s\n", string(allData))
}
```

## String conversion to numbers

Converting strings to numeric types requires parsing functions from the  
`strconv` package. These functions handle various numeric formats and bases.  

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // String to integer
    intStr := "42"
    num, err := strconv.Atoi(intStr)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Integer: %d\n", num)
    }
    
    // String to float
    floatStr := "3.14159"
    fnum, err := strconv.ParseFloat(floatStr, 64)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
    } else {
        fmt.Printf("Float: %f\n", fnum)
    }
    
    // Different bases
    hexStr := "FF"
    hexNum, _ := strconv.ParseInt(hexStr, 16, 64)
    fmt.Printf("Hex FF as decimal: %d\n", hexNum)
}
```

## Numbers to string conversion

Converting numbers to strings uses functions from the `strconv` package,  
providing control over formatting and representation.  

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    num := 42
    floatNum := 3.14159
    
    // Basic conversions
    intStr := strconv.Itoa(num)
    fmt.Printf("Integer to string: %s\n", intStr)
    
    floatStr := strconv.FormatFloat(floatNum, 'f', 2, 64)
    fmt.Printf("Float to string: %s\n", floatStr)
    
    // Different representations
    hexStr := strconv.FormatInt(int64(num), 16)
    fmt.Printf("Hex representation: %s\n", hexStr)
    
    binStr := strconv.FormatInt(int64(num), 2)
    fmt.Printf("Binary representation: %s\n", binStr)
    
    // Scientific notation
    sciStr := strconv.FormatFloat(floatNum, 'e', 2, 64)
    fmt.Printf("Scientific: %s\n", sciStr)
}
```

## String regular expressions

Regular expressions provide powerful pattern matching and text processing  
capabilities through the `regexp` package.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Contact: john@example.com or call 555-1234"
    
    // Email pattern
    emailPattern := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    email := emailPattern.FindString(text)
    fmt.Printf("Email found: %s\n", email)
    
    // Phone pattern
    phonePattern := regexp.MustCompile(`\d{3}-\d{4}`)
    phone := phonePattern.FindString(text)
    fmt.Printf("Phone found: %s\n", phone)
    
    // Replace with regex
    cleanText := regexp.MustCompile(`\d+`).ReplaceAllString(text, "[NUMBER]")
    fmt.Printf("Numbers replaced: %s\n", cleanText)
}
```

## String template basics

The `text/template` package provides a simple templating system for  
generating text output with variable substitution.  

```go
package main

import (
    "fmt"
    "text/template"
    "strings"
)

type Person struct {
    Name string
    Age  int
    City string
}

func main() {
    // Simple template
    tmplText := "Hello {{.Name}}, you are {{.Age}} years old and live in {{.City}}."
    tmpl := template.Must(template.New("greeting").Parse(tmplText))
    
    person := Person{Name: "Alice", Age: 30, City: "New York"}
    
    var result strings.Builder
    err := tmpl.Execute(&result, person)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Println(result.String())
    
    // List template
    listTmpl := "Items:\n{{range .}}• {{.}}\n{{end}}"
    listTemplate := template.Must(template.New("list").Parse(listTmpl))
    
    items := []string{"Go", "Python", "Java", "JavaScript"}
    listTemplate.Execute(&result, items)
}
```

## String multiline handling

Working with multiline strings requires special consideration for line  
endings and formatting across different platforms.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // Multiline string with raw literals
    poem := `Roses are red,
Violets are blue,
Go is awesome,
And so are you!`
    
    fmt.Println("Original poem:")
    fmt.Println(poem)
    
    // Split into lines
    lines := strings.Split(poem, "\n")
    fmt.Printf("\nPoem has %d lines:\n", len(lines))
    for i, line := range lines {
        fmt.Printf("Line %d: %s\n", i+1, line)
    }
    
    // Join lines with different separator
    singleLine := strings.Join(lines, " | ")
    fmt.Printf("\nSingle line: %s\n", singleLine)
}
```

## String performance comparison

Comparing different string building approaches demonstrates the importance  
of choosing the right method for performance-critical applications.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

func concatenateWithPlus(n int) string {
    result := ""
    for i := 0; i < n; i++ {
        result += "a"
    }
    return result
}

func concatenateWithBuilder(n int) string {
    var builder strings.Builder
    for i := 0; i < n; i++ {
        builder.WriteString("a")
    }
    return builder.String()
}

func main() {
    n := 10000
    
    // Method 1: String concatenation with +
    start := time.Now()
    result1 := concatenateWithPlus(n)
    duration1 := time.Since(start)
    
    // Method 2: strings.Builder
    start = time.Now()
    result2 := concatenateWithBuilder(n)
    duration2 := time.Since(start)
    
    fmt.Printf("Plus operator: %v (length: %d)\n", duration1, len(result1))
    fmt.Printf("Builder: %v (length: %d)\n", duration2, len(result2))
    fmt.Printf("Builder is %.2fx faster\n", float64(duration1)/float64(duration2))
}
```

## String escaping and unescaping

Handling special characters in strings often requires escaping and unescaping  
for proper representation in different contexts.  

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // String with special characters
    original := "Line 1\nLine 2\tTabbed\r\nWindows newline"
    fmt.Printf("Original string:\n%s\n\n", original)
    
    // Quote (escape) the string
    quoted := strconv.Quote(original)
    fmt.Printf("Quoted: %s\n", quoted)
    
    // Unquote (unescape) the string
    unquoted, err := strconv.Unquote(quoted)
    if err != nil {
        fmt.Printf("Error unquoting: %v\n", err)
        return
    }
    fmt.Printf("Unquoted equals original: %t\n", unquoted == original)
    
    // Quote different data types
    fmt.Printf("Quote rune: %s\n", strconv.QuoteRune('©'))
    fmt.Printf("Quote byte: %s\n", strconv.Quote(string([]byte{0x41, 0x42})))
}
```

## String validation

Validating string content ensures data integrity and prevents errors in  
applications that process user input or external data.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "unicode"
)

func isValidEmail(email string) bool {
    pattern := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    return pattern.MatchString(email)
}

func isAlphaNumeric(s string) bool {
    for _, r := range s {
        if !unicode.IsLetter(r) && !unicode.IsDigit(r) {
            return false
        }
    }
    return len(s) > 0
}

func main() {
    // Email validation
    emails := []string{
        "valid@example.com",
        "invalid.email",
        "test@domain",
        "user@example.co.uk",
    }
    
    fmt.Println("Email validation:")
    for _, email := range emails {
        fmt.Printf("%s: %t\n", email, isValidEmail(email))
    }
    
    // Alphanumeric validation
    fmt.Println("\nAlphanumeric validation:")
    inputs := []string{"abc123", "test-input", "ValidInput", ""}
    for _, input := range inputs {
        fmt.Printf("'%s': %t\n", input, isAlphaNumeric(input))
    }
    
    // Length validation
    password := "mypassword"
    fmt.Printf("\nPassword length valid (8+): %t\n", len(password) >= 8)
    fmt.Printf("Contains uppercase: %t\n", strings.ContainsAny(password, "ABCDEFGHIJKLMNOPQRSTUVWXYZ"))
}
```

## String internationalization

Handling international text requires understanding Unicode, normalization,  
and locale-specific operations for proper text processing.  

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    // Multilingual text
    greetings := []string{
        "Hello",      // English
        "Hola",       // Spanish  
        "Bonjour",    // French
        "こんにちは",     // Japanese
        "Здравствуйте", // Russian
        "مرحبا",       // Arabic
    }
    
    fmt.Println("Greetings from around the world:")
    for i, greeting := range greetings {
        runeCount := utf8.RuneCountInString(greeting)
        byteCount := len(greeting)
        fmt.Printf("%d. %s (runes: %d, bytes: %d)\n", 
                   i+1, greeting, runeCount, byteCount)
    }
    
    // UTF-8 validation
    validUTF8 := "Hello 世界"
    invalidUTF8 := string([]byte{0xff, 0xfe, 0xfd})
    
    fmt.Printf("\nUTF-8 validation:\n")
    fmt.Printf("'%s' is valid UTF-8: %t\n", validUTF8, utf8.ValidString(validUTF8))
    fmt.Printf("Invalid bytes are valid UTF-8: %t\n", utf8.ValidString(invalidUTF8))
}
```

## String normalization

Unicode normalization ensures consistent representation of text that can  
be encoded in multiple ways, essential for text comparison and processing.  

```go
package main

import "fmt"

func main() {
    // Different representations of the same character
    // é can be represented as:
    // 1. Single character U+00E9 (é)
    // 2. Two characters U+0065 + U+0301 (e + ´)
    
    composed := "café"    // é as single character
    decomposed := "cafe\u0301" // é as e + combining accent
    
    fmt.Printf("Composed: %s (len: %d)\n", composed, len(composed))
    fmt.Printf("Decomposed: %s (len: %d)\n", decomposed, len(decomposed))
    fmt.Printf("Equal: %t\n", composed == decomposed)
    
    // Manual normalization check
    fmt.Printf("Visual appearance same: %t\n", composed == decomposed)
    
    // Show the difference in bytes
    fmt.Printf("Composed bytes: %v\n", []byte(composed))
    fmt.Printf("Decomposed bytes: %v\n", []byte(decomposed))
    
    // Rune count comparison
    fmt.Printf("Composed runes: %d\n", len([]rune(composed)))
    fmt.Printf("Decomposed runes: %d\n", len([]rune(decomposed)))
}
```

## String security considerations

String handling in security contexts requires attention to encoding,  
injection prevention, and safe comparison practices.  

```go
package main

import (
    "crypto/subtle"
    "fmt"
    "html"
    "net/url"
    "strings"
)

func main() {
    // HTML escaping to prevent XSS
    userInput := "<script>alert('xss')</script>"
    escaped := html.EscapeString(userInput)
    fmt.Printf("Original: %s\n", userInput)
    fmt.Printf("Escaped: %s\n", escaped)
    
    // URL encoding for safe URLs
    query := "hello world & more"
    encoded := url.QueryEscape(query)
    fmt.Printf("URL encoded: %s\n", encoded)
    
    // Safe string comparison (prevents timing attacks)
    secret1 := "supersecret"
    secret2 := "supersecret"
    secret3 := "wrongsecret"
    
    // Use subtle.ConstantTimeCompare for security-sensitive comparisons
    fmt.Printf("Safe compare (equal): %t\n", 
               subtle.ConstantTimeCompare([]byte(secret1), []byte(secret2)) == 1)
    fmt.Printf("Safe compare (different): %t\n", 
               subtle.ConstantTimeCompare([]byte(secret1), []byte(secret3)) == 1)
    
    // SQL injection prevention example
    username := "admin'; DROP TABLE users; --"
    // Don't do: "SELECT * FROM users WHERE name = '" + username + "'"
    // Instead use parameterized queries or escape
    safeUsername := strings.ReplaceAll(username, "'", "''")
    fmt.Printf("Escaped SQL: %s\n", safeUsername)
}
```

## String advanced patterns

Advanced string processing patterns combine multiple techniques for  
complex text manipulation and analysis tasks.  

```go
package main

import (
    "fmt"
    "regexp"
    "sort"
    "strings"
)

func extractWords(text string) []string {
    // Remove punctuation and convert to lowercase
    reg := regexp.MustCompile(`[^\p{L}\p{N}]+`)
    words := reg.Split(strings.ToLower(text), -1)
    
    // Filter empty strings
    var result []string
    for _, word := range words {
        if len(word) > 0 {
            result = append(result, word)
        }
    }
    return result
}

func wordFrequency(words []string) map[string]int {
    freq := make(map[string]int)
    for _, word := range words {
        freq[word]++
    }
    return freq
}

func main() {
    text := "Go programming is fun! Go makes programming efficient. Programming in Go is a joy."
    
    // Extract and analyze words
    words := extractWords(text)
    fmt.Printf("Extracted words: %v\n", words)
    
    // Calculate word frequency
    freq := wordFrequency(words)
    
    // Sort by frequency
    type wordCount struct {
        word  string
        count int
    }
    
    var wordCounts []wordCount
    for word, count := range freq {
        wordCounts = append(wordCounts, wordCount{word, count})
    }
    
    sort.Slice(wordCounts, func(i, j int) bool {
        return wordCounts[i].count > wordCounts[j].count
    })
    
    fmt.Println("\nWord frequency (sorted):")
    for _, wc := range wordCounts {
        fmt.Printf("%s: %d\n", wc.word, wc.count)
    }
}
```

## String streaming and buffering

Processing large strings efficiently often requires streaming and buffering  
techniques to manage memory usage and improve performance.  

```go
package main

import (
    "bufio"
    "fmt"
    "strings"
)

func processLargeString(text string) {
    reader := strings.NewReader(text)
    scanner := bufio.NewScanner(reader)
    
    lineCount := 0
    wordCount := 0
    charCount := 0
    
    // Process line by line
    for scanner.Scan() {
        line := scanner.Text()
        lineCount++
        charCount += len(line)
        
        // Count words in line
        words := strings.Fields(line)
        wordCount += len(words)
    }
    
    if err := scanner.Err(); err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Printf("Statistics:\n")
    fmt.Printf("Lines: %d\n", lineCount)
    fmt.Printf("Words: %d\n", wordCount)
    fmt.Printf("Characters: %d\n", charCount)
}

func main() {
    // Simulate large text
    var builder strings.Builder
    for i := 1; i <= 100; i++ {
        builder.WriteString(fmt.Sprintf("This is line number %d with several words.\n", i))
    }
    
    largeText := builder.String()
    fmt.Printf("Processing text of %d bytes\n", len(largeText))
    processLargeString(largeText)
    
    // Demonstrate buffered writing
    var output strings.Builder
    writer := bufio.NewWriter(&output)
    
    for i := 0; i < 1000; i++ {
        writer.WriteString(fmt.Sprintf("Item %d ", i))
    }
    writer.Flush()
    
    result := output.String()
    fmt.Printf("\nBuffered output length: %d\n", len(result))
    fmt.Printf("First 50 chars: %s...\n", result[:50])
}
```

## String encoding and decoding

Working with different text encodings is crucial for processing data from  
various sources and ensuring proper character representation.  

```go
package main

import (
    "encoding/base64"
    "fmt"
)

func main() {
    original := "Go programming with special chars: åäö"
    fmt.Printf("Original: %s\n", original)
    
    // Base64 encoding
    encoded := base64.StdEncoding.EncodeToString([]byte(original))
    fmt.Printf("Base64 encoded: %s\n", encoded)
    
    // Base64 decoding
    decoded, err := base64.StdEncoding.DecodeString(encoded)
    if err != nil {
        fmt.Printf("Decode error: %v\n", err)
        return
    }
    fmt.Printf("Base64 decoded: %s\n", string(decoded))
    
    // URL-safe Base64 encoding
    urlEncoded := base64.URLEncoding.EncodeToString([]byte(original))
    fmt.Printf("URL-safe encoded: %s\n", urlEncoded)
    
    // Binary representation
    fmt.Printf("Binary bytes: %v\n", []byte(original))
    fmt.Printf("Hex representation: %x\n", []byte(original))
    
    // String from bytes
    bytes := []byte{72, 101, 108, 108, 111} // "Hello"
    fromBytes := string(bytes)
    fmt.Printf("From bytes: %s\n", fromBytes)
}
```

## String compression

String compression techniques help reduce memory usage and storage requirements  
for text data while maintaining content integrity.  

```go
package main

import (
    "bytes"
    "compress/gzip"
    "fmt"
    "io"
    "strings"
)

func compressString(s string) ([]byte, error) {
    var buf bytes.Buffer
    gzWriter := gzip.NewWriter(&buf)
    
    _, err := gzWriter.Write([]byte(s))
    if err != nil {
        return nil, err
    }
    
    err = gzWriter.Close()
    if err != nil {
        return nil, err
    }
    
    return buf.Bytes(), nil
}

func decompressString(data []byte) (string, error) {
    buf := bytes.NewBuffer(data)
    gzReader, err := gzip.NewReader(buf)
    if err != nil {
        return "", err
    }
    defer gzReader.Close()
    
    decompressed, err := io.ReadAll(gzReader)
    if err != nil {
        return "", err
    }
    
    return string(decompressed), nil
}

func main() {
    // Create repetitive text that compresses well
    var builder strings.Builder
    pattern := "This is a repeating pattern for compression testing. "
    for i := 0; i < 100; i++ {
        builder.WriteString(pattern)
    }
    original := builder.String()
    
    fmt.Printf("Original size: %d bytes\n", len(original))
    
    // Compress the string
    compressed, err := compressString(original)
    if err != nil {
        fmt.Printf("Compression error: %v\n", err)
        return
    }
    
    fmt.Printf("Compressed size: %d bytes\n", len(compressed))
    ratio := float64(len(original)) / float64(len(compressed))
    fmt.Printf("Compression ratio: %.2f:1\n", ratio)
    
    // Decompress and verify
    decompressed, err := decompressString(compressed)
    if err != nil {
        fmt.Printf("Decompression error: %v\n", err)
        return
    }
    
    fmt.Printf("Decompression successful: %t\n", original == decompressed)
    fmt.Printf("First 100 chars: %s...\n", decompressed[:100])
}
```

## String best practices

Following best practices for string handling ensures efficient, maintainable,  
and bug-free code in Go applications.  

```go
package main

import (
    "fmt"
    "strings"
)

// Efficient string building
func buildStringEfficiently(items []string) string {
    if len(items) == 0 {
        return ""
    }
    
    // Use strings.Join for simple concatenation
    if len(items) <= 10 {
        return strings.Join(items, " ")
    }
    
    // Use strings.Builder for complex building
    var builder strings.Builder
    builder.Grow(len(items) * 10) // Pre-allocate capacity
    
    for i, item := range items {
        if i > 0 {
            builder.WriteString(" ")
        }
        builder.WriteString(item)
    }
    
    return builder.String()
}

// Safe string operations
func safeStringOperations(text string, index int) {
    // Always check bounds
    if index >= 0 && index < len(text) {
        fmt.Printf("Character at %d: %c\n", index, text[index])
    } else {
        fmt.Printf("Index %d is out of bounds\n", index)
    }
    
    // Use range for UTF-8 safety
    fmt.Print("Characters: ")
    for _, r := range text {
        fmt.Printf("%c ", r)
    }
    fmt.Println()
}

func main() {
    // Demonstrate efficient building
    items := []string{"Go", "is", "an", "excellent", "programming", "language", 
                     "for", "building", "efficient", "applications"}
    
    result := buildStringEfficiently(items)
    fmt.Printf("Built string: %s\n", result)
    
    // Demonstrate safe operations
    text := "Hello 世界"
    safeStringOperations(text, 5)
    safeStringOperations(text, 50) // Out of bounds
    
    // String constants for better performance
    const template = "Processing item: %s"
    items2 := []string{"apple", "banana", "cherry"}
    
    for _, item := range items2 {
        fmt.Printf(template+"\n", item)
    }
    
    // Prefer specific functions over general ones
    text2 := "  trimmed  "
    fmt.Printf("Trimmed: '%s'\n", strings.TrimSpace(text2))
    
    // Use appropriate comparison methods
    fmt.Printf("Case-insensitive equal: %t\n", 
               strings.EqualFold("Hello", "HELLO"))
}
```

## String palindrome check

A palindrome is a word or phrase that reads the same backward as forward.  
This example demonstrates string manipulation for palindrome detection.  

```go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

func isPalindrome(s string) bool {
    // Convert to lowercase and remove non-alphanumeric characters
    cleaned := strings.Map(func(r rune) rune {
        if unicode.IsLetter(r) || unicode.IsDigit(r) {
            return unicode.ToLower(r)
        }
        return -1 // Remove character
    }, s)
    
    // Check if cleaned string is palindrome
    runes := []rune(cleaned)
    for i := 0; i < len(runes)/2; i++ {
        if runes[i] != runes[len(runes)-1-i] {
            return false
        }
    }
    return true
}

func main() {
    testCases := []string{
        "racecar",
        "A man, a plan, a canal: Panama",
        "race a car",
        "hello",
        "Madam",
        "Was it a car or a cat I saw?",
    }
    
    fmt.Println("Palindrome testing:")
    for _, test := range testCases {
        result := isPalindrome(test)
        fmt.Printf("'%s': %t\n", test, result)
    }
}
```

## String anagrams detection

Anagrams are words formed by rearranging letters of another word.  
This example shows how to detect anagrams using string manipulation.  

```go
package main

import (
    "fmt"
    "sort"
    "strings"
)

func normalizeString(s string) string {
    // Convert to lowercase and remove spaces
    normalized := strings.ToLower(strings.ReplaceAll(s, " ", ""))
    
    // Sort characters
    chars := []rune(normalized)
    sort.Slice(chars, func(i, j int) bool {
        return chars[i] < chars[j]
    })
    
    return string(chars)
}

func areAnagrams(s1, s2 string) bool {
    return normalizeString(s1) == normalizeString(s2)
}

func main() {
    anagramPairs := [][]string{
        {"listen", "silent"},
        {"elbow", "below"},
        {"study", "dusty"},
        {"hello", "world"},
        {"The Eyes", "They See"},
        {"Conversation", "Voices rant on"},
    }
    
    fmt.Println("Anagram detection:")
    for _, pair := range anagramPairs {
        result := areAnagrams(pair[0], pair[1])
        fmt.Printf("'%s' & '%s': %t\n", pair[0], pair[1], result)
    }
}
```

## String Caesar cipher

The Caesar cipher shifts each letter by a fixed number of positions in the  
alphabet. This demonstrates character manipulation and modular arithmetic.  

```go
package main

import (
    "fmt"
    "strings"
)

func caesarCipher(text string, shift int) string {
    return strings.Map(func(r rune) rune {
        if r >= 'A' && r <= 'Z' {
            return 'A' + (r-'A'+rune(shift))%26
        }
        if r >= 'a' && r <= 'z' {
            return 'a' + (r-'a'+rune(shift))%26
        }
        return r // Non-alphabetic characters remain unchanged
    }, text)
}

func main() {
    plaintext := "Hello World"
    shift := 3
    
    // Encrypt
    encrypted := caesarCipher(plaintext, shift)
    fmt.Printf("Original: %s\n", plaintext)
    fmt.Printf("Encrypted (shift %d): %s\n", shift, encrypted)
    
    // Decrypt (negative shift)
    decrypted := caesarCipher(encrypted, -shift)
    fmt.Printf("Decrypted: %s\n", decrypted)
    
    // Test with different shifts
    fmt.Println("\nDifferent shifts:")
    for i := 1; i <= 5; i++ {
        shifted := caesarCipher("ABC", i)
        fmt.Printf("Shift %d: %s\n", i, shifted)
    }
}
```

## String word wrapping

Word wrapping breaks long lines into multiple lines at word boundaries  
to fit within a specified width, useful for text formatting.  

```go
package main

import (
    "fmt"
    "strings"
)

func wrapText(text string, width int) []string {
    words := strings.Fields(text)
    if len(words) == 0 {
        return []string{}
    }
    
    var lines []string
    var currentLine strings.Builder
    
    for _, word := range words {
        // Check if adding this word would exceed width
        if currentLine.Len() > 0 && currentLine.Len()+1+len(word) > width {
            lines = append(lines, currentLine.String())
            currentLine.Reset()
        }
        
        if currentLine.Len() > 0 {
            currentLine.WriteString(" ")
        }
        currentLine.WriteString(word)
    }
    
    if currentLine.Len() > 0 {
        lines = append(lines, currentLine.String())
    }
    
    return lines
}

func main() {
    longText := "Go is an open source programming language that makes it easy to build simple, reliable, and efficient software. It was designed at Google to solve real-world problems."
    
    widths := []int{20, 30, 50}
    
    for _, width := range widths {
        fmt.Printf("Wrapped at %d characters:\n", width)
        lines := wrapText(longText, width)
        for i, line := range lines {
            fmt.Printf("%2d: %s\n", i+1, line)
        }
        fmt.Println()
    }
}
```

## String acronym generation

Creating acronyms from phrases by extracting the first letter of each word  
and handling special cases like articles and prepositions.  

```go
package main

import (
    "fmt"
    "strings"
    "unicode"
)

func generateAcronym(phrase string) string {
    words := strings.Fields(strings.ToUpper(phrase))
    
    // Common words to skip in acronyms
    skipWords := map[string]bool{
        "THE": true, "AND": true, "OF": true, "TO": true,
        "A": true, "AN": true, "IN": true, "ON": true, "FOR": true,
    }
    
    var acronym strings.Builder
    for _, word := range words {
        if len(word) > 0 && !skipWords[word] {
            // Get first letter
            firstRune := []rune(word)[0]
            if unicode.IsLetter(firstRune) {
                acronym.WriteRune(firstRune)
            }
        }
    }
    
    return acronym.String()
}

func main() {
    phrases := []string{
        "Application Programming Interface",
        "Hypertext Transfer Protocol",
        "World Wide Web",
        "Central Processing Unit",
        "Random Access Memory",
        "Portable Document Format",
        "The United States of America",
        "North Atlantic Treaty Organization",
    }
    
    fmt.Println("Acronym generation:")
    for _, phrase := range phrases {
        acronym := generateAcronym(phrase)
        fmt.Printf("%-35s -> %s\n", phrase, acronym)
    }
}
```

## String levenshtein distance

The Levenshtein distance measures the minimum number of single-character  
edits needed to change one string into another, useful for similarity comparison.  

```go
package main

import "fmt"

func levenshteinDistance(s1, s2 string) int {
    runes1 := []rune(s1)
    runes2 := []rune(s2)
    
    m, n := len(runes1), len(runes2)
    
    // Create matrix
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    // Initialize base cases
    for i := 0; i <= m; i++ {
        dp[i][0] = i
    }
    for j := 0; j <= n; j++ {
        dp[0][j] = j
    }
    
    // Fill matrix
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if runes1[i-1] == runes2[j-1] {
                dp[i][j] = dp[i-1][j-1] // No operation needed
            } else {
                dp[i][j] = 1 + min(
                    dp[i-1][j],   // Deletion
                    dp[i][j-1],   // Insertion
                    dp[i-1][j-1], // Substitution
                )
            }
        }
    }
    
    return dp[m][n]
}

func min(a, b, c int) int {
    if a <= b && a <= c {
        return a
    }
    if b <= c {
        return b
    }
    return c
}

func main() {
    pairs := [][]string{
        {"kitten", "sitting"},
        {"saturday", "sunday"},
        {"hello", "hallo"},
        {"algorithm", "altruistic"},
        {"", "abc"},
        {"abc", ""},
    }
    
    fmt.Println("Levenshtein distances:")
    for _, pair := range pairs {
        distance := levenshteinDistance(pair[0], pair[1])
        fmt.Printf("'%s' <-> '%s': %d\n", pair[0], pair[1], distance)
    }
}
```

## String text statistics

Analyzing text to extract various statistics like character frequency,  
word count, sentence count, and readability metrics.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "unicode"
)

type TextStats struct {
    CharCount     int
    WordCount     int
    SentenceCount int
    ParagraphCount int
    CharFreq      map[rune]int
}

func analyzeText(text string) TextStats {
    stats := TextStats{
        CharFreq: make(map[rune]int),
    }
    
    // Character analysis
    for _, r := range text {
        if !unicode.IsSpace(r) {
            stats.CharCount++
            stats.CharFreq[unicode.ToLower(r)]++
        }
    }
    
    // Word count
    words := strings.Fields(text)
    stats.WordCount = len(words)
    
    // Sentence count (approximate)
    sentencePattern := regexp.MustCompile(`[.!?]+`)
    sentences := sentencePattern.FindAllString(text, -1)
    stats.SentenceCount = len(sentences)
    
    // Paragraph count
    paragraphs := strings.Split(text, "\n\n")
    stats.ParagraphCount = 0
    for _, para := range paragraphs {
        if strings.TrimSpace(para) != "" {
            stats.ParagraphCount++
        }
    }
    
    return stats
}

func main() {
    sampleText := `Go is an open source programming language that makes it easy to build simple, reliable, and efficient software.

It was created at Google by Robert Griesemer, Rob Pike, and Ken Thompson. Go is syntactically similar to C, but with memory safety, garbage collection, structural typing, and CSP-style concurrency.

The language was announced in November 2009 and is used in some of Google's production systems. Go's mascot is a gopher designed by Renee French.`
    
    stats := analyzeText(sampleText)
    
    fmt.Println("Text Statistics:")
    fmt.Printf("Characters: %d\n", stats.CharCount)
    fmt.Printf("Words: %d\n", stats.WordCount)
    fmt.Printf("Sentences: %d\n", stats.SentenceCount)
    fmt.Printf("Paragraphs: %d\n", stats.ParagraphCount)
    
    fmt.Println("\nMost frequent characters:")
    type charFreq struct {
        char rune
        freq int
    }
    
    var freqs []charFreq
    for char, freq := range stats.CharFreq {
        if unicode.IsLetter(char) && freq > 1 {
            freqs = append(freqs, charFreq{char, freq})
        }
    }
    
    // Simple bubble sort for demonstration
    for i := 0; i < len(freqs)-1; i++ {
        for j := 0; j < len(freqs)-i-1; j++ {
            if freqs[j].freq < freqs[j+1].freq {
                freqs[j], freqs[j+1] = freqs[j+1], freqs[j]
            }
        }
    }
    
    for i, cf := range freqs {
        if i >= 10 { // Show top 10
            break
        }
        fmt.Printf("%c: %d\n", cf.char, cf.freq)
    }
}
```