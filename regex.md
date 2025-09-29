# Regular Expressions

Regular expressions are powerful pattern matching tools that provide a concise  
and flexible means for matching strings of text. In Go, regular expressions  
are handled through the `regexp` package, which implements the RE2 syntax  
developed by Google. This syntax ensures linear time execution and memory  
safety, making Go's regex implementation both fast and secure.

The RE2 syntax supported by Go is slightly different from PCRE (Perl Compatible  
Regular Expressions) used in many other languages. While it lacks some advanced  
features like backreferences and lookarounds, it compensates with guaranteed  
performance characteristics and complete Unicode support. This makes it ideal  
for applications where performance and security are paramount.

Go's regexp package provides two main approaches to using regular expressions:  
compiled patterns and convenience functions. Compiled patterns using  
`regexp.Compile()` or `regexp.MustCompile()` offer better performance when  
the same pattern is used multiple times, as the compilation overhead is  
incurred only once. The convenience functions are useful for one-off operations  
but recompile the pattern each time.

The package supports various operations including pattern matching, text  
searching, string replacement, and text splitting. All functions work  
seamlessly with UTF-8 encoded strings, making them suitable for international  
text processing. The regexp package also provides methods for working with  
byte slices, which can be more efficient for certain applications.

Regular expressions in Go follow a specific syntax with metacharacters for  
different matching behaviors. Character classes, quantifiers, anchors, and  
grouping constructs allow for complex pattern definitions. Understanding  
these components is essential for creating effective and efficient patterns  
for text processing tasks in Go applications.


## Basic pattern matching

Simple pattern matching demonstrates the fundamental concept of finding text  
that matches a specific pattern within a larger string.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "The cat sat on the mat"
    
    // Simple literal matching
    matched, _ := regexp.MatchString("cat", text)
    fmt.Printf("Found 'cat': %t\n", matched)
    
    // Case-insensitive matching with flag
    matched2, _ := regexp.MatchString("(?i)CAT", text)
    fmt.Printf("Found 'CAT' (case-insensitive): %t\n", matched2)
    
    // Multiple word matching
    matched3, _ := regexp.MatchString("sat on", text)
    fmt.Printf("Found 'sat on': %t\n", matched3)
    
    // Non-matching pattern
    matched4, _ := regexp.MatchString("dog", text)
    fmt.Printf("Found 'dog': %t\n", matched4)
}
```

The `regexp.MatchString()` function provides a simple way to check if a pattern  
exists within a string. It returns a boolean indicating success and an error  
if the pattern is invalid. This function recompiles the pattern each time,  
making it suitable for one-time checks but inefficient for repeated use.  


## Compiling patterns

Compiling regular expressions into reusable pattern objects improves  
performance when the same pattern is used multiple times.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Compile pattern for reuse
    pattern, err := regexp.Compile(`\d+`)
    if err != nil {
        fmt.Printf("Pattern compilation error: %v\n", err)
        return
    }
    
    texts := []string{
        "I have 42 apples",
        "The year is 2024",
        "No numbers here",
        "Phone: 555-1234",
    }
    
    for _, text := range texts {
        found := pattern.FindString(text)
        if found != "" {
            fmt.Printf("Found number in '%s': %s\n", text, found)
        } else {
            fmt.Printf("No numbers in '%s'\n", text)
        }
    }
    
    // Using MustCompile for patterns known to be valid
    mustPattern := regexp.MustCompile(`[a-zA-Z]+`)
    word := mustPattern.FindString("Hello123World")
    fmt.Printf("First word: %s\n", word)
}
```

Compiled patterns offer significant performance benefits when used repeatedly.  
The `regexp.Compile()` function returns an error for invalid patterns, while  
`regexp.MustCompile()` panics on invalid patterns but provides cleaner code  
when you're certain the pattern is correct.  


## Finding text patterns

The Find family of methods locate text that matches patterns and return  
the matched content or positions within the source string.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Contact us at support@company.com or sales@company.com"
    emailPattern := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    
    // Find first match
    firstEmail := emailPattern.FindString(text)
    fmt.Printf("First email: %s\n", firstEmail)
    
    // Find all matches
    allEmails := emailPattern.FindAllString(text, -1)
    fmt.Printf("All emails: %v\n", allEmails)
    
    // Find with position information
    firstPos := emailPattern.FindStringIndex(text)
    if firstPos != nil {
        fmt.Printf("First email position: start=%d, end=%d\n", firstPos[0], firstPos[1])
        fmt.Printf("Email text: %s\n", text[firstPos[0]:firstPos[1]])
    }
    
    // Find all positions
    allPositions := emailPattern.FindAllStringIndex(text, -1)
    for i, pos := range allPositions {
        fmt.Printf("Email %d at position [%d:%d]: %s\n", 
            i+1, pos[0], pos[1], text[pos[0]:pos[1]])
    }
}
```

The Find methods come in several variants: `FindString` for content,  
`FindStringIndex` for positions, and `FindAllString`/`FindAllStringIndex`  
for multiple matches. The limit parameter (-1 means no limit) controls  
how many matches to return.  


## Character classes

Character classes define sets of characters that can match at a single  
position in the pattern, providing flexible matching options.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Phone: 123-456-7890, Fax: 098-765-4321, Mobile: +1-555-123-4567"
    
    // Digit character class
    digits := regexp.MustCompile(`\d+`)
    allDigits := digits.FindAllString(text, -1)
    fmt.Printf("All digit sequences: %v\n", allDigits)
    
    // Word character class (letters, digits, underscore)
    words := regexp.MustCompile(`\w+`)
    allWords := words.FindAllString(text, -1)
    fmt.Printf("All word sequences: %v\n", allWords)
    
    // Custom character class
    phoneChars := regexp.MustCompile(`[0-9+-]+`)
    phoneNumbers := phoneChars.FindAllString(text, -1)
    fmt.Printf("Phone number parts: %v\n", phoneNumbers)
    
    // Negated character class
    nonDigits := regexp.MustCompile(`[^\d]+`)
    textParts := nonDigits.FindAllString(text, -1)
    fmt.Printf("Non-digit parts: %v\n", textParts)
    
    // Range-based character classes
    letters := regexp.MustCompile(`[a-zA-Z]+`)
    letterSequences := letters.FindAllString(text, -1)
    fmt.Printf("Letter sequences: %v\n", letterSequences)
}
```

Character classes use square brackets to define character sets. Common  
predefined classes include `\d` (digits), `\w` (word characters), and `\s`  
(whitespace). Custom classes can use ranges like `[a-z]` or negation with `^`.  


## Quantifiers

Quantifiers specify how many times a pattern element should be matched,  
enabling flexible pattern repetition and optional matching.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "I have 1 apple, 23 oranges, and 456 grapes in my basket"
    
    // Zero or more quantifier (*)
    pattern1 := regexp.MustCompile(`\d*`)
    matches1 := pattern1.FindAllString(text, -1)
    fmt.Printf("Zero or more digits: %v\n", matches1)
    
    // One or more quantifier (+)
    pattern2 := regexp.MustCompile(`\d+`)
    matches2 := pattern2.FindAllString(text, -1)
    fmt.Printf("One or more digits: %v\n", matches2)
    
    // Zero or one quantifier (?)
    text2 := "color colour favorite favourite"
    pattern3 := regexp.MustCompile(`colou?r`)
    matches3 := pattern3.FindAllString(text2, -1)
    fmt.Printf("Optional 'u' matches: %v\n", matches3)
    
    // Exact count quantifier {n}
    pattern4 := regexp.MustCompile(`\d{3}`)
    matches4 := pattern4.FindAllString(text, -1)
    fmt.Printf("Exactly 3 digits: %v\n", matches4)
    
    // Range quantifiers {n,m}
    pattern5 := regexp.MustCompile(`\d{1,3}`)
    matches5 := pattern5.FindAllString(text, -1)
    fmt.Printf("1 to 3 digits: %v\n", matches5)
    
    // Minimum quantifier {n,}
    pattern6 := regexp.MustCompile(`[a-z]{4,}`)
    matches6 := pattern6.FindAllString(text, -1)
    fmt.Printf("4+ letter words: %v\n", matches6)
}
```

Quantifiers control repetition: `*` (zero or more), `+` (one or more),  
`?` (zero or one), `{n}` (exactly n), `{n,m}` (between n and m),  
and `{n,}` (n or more). They apply to the preceding pattern element.  


## Anchors and boundaries

Anchors and word boundaries constrain where matches can occur within  
the text, enabling precise positional matching.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "The cat catches mice. A cat is quick."
    
    // Beginning of string anchor (^)
    startPattern := regexp.MustCompile(`^The`)
    startMatch := startPattern.FindString(text)
    fmt.Printf("Starts with 'The': '%s'\n", startMatch)
    
    // End of string anchor ($)
    endPattern := regexp.MustCompile(`quick\.$`)
    endMatch := endPattern.FindString(text)
    fmt.Printf("Ends with 'quick.': '%s'\n", endMatch)
    
    // Word boundary (\b)
    wordPattern := regexp.MustCompile(`\bcat\b`)
    wordMatches := wordPattern.FindAllString(text, -1)
    fmt.Printf("Complete word 'cat': %v\n", wordMatches)
    
    // Non-word boundary (\B)
    nonWordPattern := regexp.MustCompile(`\Bcat\B`)
    nonWordMatches := nonWordPattern.FindAllString(text, -1)
    fmt.Printf("'cat' within words: %v\n", nonWordMatches)
    
    // Multiple lines example
    multiText := "first line\nsecond line\nthird line"
    
    // Multiline mode with (?m)
    lineStartPattern := regexp.MustCompile(`(?m)^second`)
    lineMatch := lineStartPattern.FindString(multiText)
    fmt.Printf("Line starting with 'second': '%s'\n", lineMatch)
    
    // Find all line starts
    allLineStarts := regexp.MustCompile(`(?m)^[a-z]+`)
    allStarts := allLineStarts.FindAllString(multiText, -1)
    fmt.Printf("All line start words: %v\n", allStarts)
}
```

Anchors fix matches to specific positions: `^` (start), `$` (end),  
`\b` (word boundary), `\B` (non-word boundary). The `(?m)` flag enables  
multiline mode where `^` and `$` match line starts and ends.  


## Capturing groups

Capturing groups extract specific parts of matched text using parentheses  
to create submatches for detailed text processing.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "John Doe <john.doe@company.com> and Jane Smith <jane@example.org>"
    
    // Single capturing group
    pattern := regexp.MustCompile(`([a-zA-Z0-9._%+-]+)@([a-zA-Z0-9.-]+\.[a-zA-Z]{2,})`)
    
    // Find with submatches
    matches := pattern.FindStringSubmatch("Contact us at support@company.com")
    if len(matches) > 0 {
        fmt.Printf("Full match: %s\n", matches[0])
        fmt.Printf("Username: %s\n", matches[1])
        fmt.Printf("Domain: %s\n", matches[2])
    }
    
    // Find all with submatches
    allMatches := pattern.FindAllStringSubmatch(text, -1)
    for i, match := range allMatches {
        fmt.Printf("Email %d - Full: %s, User: %s, Domain: %s\n",
            i+1, match[0], match[1], match[2])
    }
    
    // Named capturing groups
    namedPattern := regexp.MustCompile(`(?P<name>[A-Z][a-z]+ [A-Z][a-z]+) <(?P<email>[^>]+)>`)
    namedMatches := namedPattern.FindStringSubmatch(text)
    
    if len(namedMatches) > 0 {
        names := namedPattern.SubexpNames()
        for i, name := range names {
            if i != 0 && name != "" {
                fmt.Printf("%s: %s\n", name, namedMatches[i])
            }
        }
    }
    
    // Multiple matches with names
    allNamedMatches := namedPattern.FindAllStringSubmatch(text, -1)
    for _, match := range allNamedMatches {
        fmt.Printf("Person: %s, Email: %s\n", match[1], match[2])
    }
}
```

Capturing groups use parentheses to extract parts of matches. Named groups  
use `(?P<name>pattern)` syntax for better readability. The `FindSubmatch`  
functions return slices where index 0 is the full match and subsequent  
indices are the captured groups.  


## String replacement

Regular expressions enable sophisticated text replacement operations with  
pattern-based substitution and backreferences to captured groups.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func main() {
    text := "Contact John at john.doe@company.com or Jane at jane.smith@example.org"
    
    // Simple replacement
    emailPattern := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    masked := emailPattern.ReplaceAllString(text, "[EMAIL]")
    fmt.Printf("Masked emails: %s\n", masked)
    
    // Replacement with captured groups
    phoneText := "Call 555-123-4567 or 555-987-6543 for assistance"
    phonePattern := regexp.MustCompile(`(\d{3})-(\d{3})-(\d{4})`)
    formatted := phonePattern.ReplaceAllString(phoneText, "($1) $2-$3")
    fmt.Printf("Formatted phones: %s\n", formatted)
    
    // Replacement function for complex logic
    numberText := "I have 5 apples and 12 oranges"
    digitPattern := regexp.MustCompile(`\d+`)
    
    doubled := digitPattern.ReplaceAllStringFunc(numberText, func(s string) string {
        // Convert to int, double it, convert back
        return fmt.Sprintf("%s*2", s)
    })
    fmt.Printf("Doubled numbers: %s\n", doubled)
    
    // Conditional replacement
    mixedText := "URGENT: Please call 911 or 555-HELP immediately"
    urgentPattern := regexp.MustCompile(`(?i)\b(urgent|emergency|immediate)\w*\b`)
    
    highlighted := urgentPattern.ReplaceAllStringFunc(mixedText, func(s string) string {
        return fmt.Sprintf("**%s**", strings.ToUpper(s))
    })
    fmt.Printf("Highlighted urgent words: %s\n", highlighted)
    
    // Multiple pattern replacement
    cleanText := "Remove    extra   spaces   and\t\ttabs"
    spacePattern := regexp.MustCompile(`\s+`)
    normalized := spacePattern.ReplaceAllString(cleanText, " ")
    fmt.Printf("Normalized spacing: %s\n", strings.TrimSpace(normalized))
}
```

The `ReplaceAllString` method performs simple substitution, while  
`ReplaceAllStringFunc` allows complex replacement logic. Captured groups  
can be referenced in replacements using `$1`, `$2`, etc. for the  
corresponding group numbers.  


## Text splitting

Regular expressions can split text into segments based on complex patterns  
rather than simple delimiter characters.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "apple,banana;cherry:date|elderberry grape"
    
    // Split on multiple delimiters
    delimiterPattern := regexp.MustCompile(`[,;:|]\s*`)
    fruits := delimiterPattern.Split(text, -1)
    fmt.Printf("Split fruits: %v\n", fruits)
    
    // Split on whitespace (including tabs and multiple spaces)
    messyText := "word1    word2\t\tword3   \n  word4"
    whitespacePattern := regexp.MustCompile(`\s+`)
    words := whitespacePattern.Split(messyText, -1)
    fmt.Printf("Clean words: %v\n", words)
    
    // Split with limit
    sentence := "The quick brown fox jumps over the lazy dog"
    spacePattern := regexp.MustCompile(`\s+`)
    firstThree := spacePattern.Split(sentence, 3)
    fmt.Printf("First 3 parts: %v\n", firstThree)
    
    // Split preserving delimiters (using FindAll instead)
    csvLine := "name,age,city,country"
    csvPattern := regexp.MustCompile(`[^,]+`)
    fields := csvPattern.FindAllString(csvLine, -1)
    fmt.Printf("CSV fields: %v\n", fields)
    
    // Complex splitting pattern
    logEntry := "2024-01-15 14:30:45 [INFO] User login successful - user:john.doe ip:192.168.1.100"
    logPattern := regexp.MustCompile(`\s+[-\[\]]\s*|\s+-\s+`)
    logParts := logPattern.Split(logEntry, -1)
    fmt.Printf("Log parts: %v\n", logParts)
    
    // Split on word boundaries for natural language processing
    paragraph := "Hello there! How are you doing today? Fine, thanks."
    sentencePattern := regexp.MustCompile(`[.!?]+\s*`)
    sentences := sentencePattern.Split(paragraph, -1)
    fmt.Printf("Sentences: %v\n", sentences)
}
```

The `Split` method divides text based on regex patterns. The limit parameter  
controls maximum splits (-1 for unlimited). For complex scenarios, combining  
`FindAll` with splitting can preserve important delimiter information while  
breaking text into meaningful segments.  


## Pattern validation

Regular expressions excel at validating input data against specific formats  
and rules, ensuring data integrity and format compliance.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Email validation
    emailPattern := regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`)
    emails := []string{
        "user@example.com",
        "invalid.email",
        "test@domain.co.uk",
        "user.name+tag@domain-name.com",
        "@invalid.com",
    }
    
    fmt.Println("Email validation:")
    for _, email := range emails {
        valid := emailPattern.MatchString(email)
        fmt.Printf("  %s: %t\n", email, valid)
    }
    
    // Phone number validation (US format)
    phonePattern := regexp.MustCompile(`^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$`)
    phones := []string{
        "555-123-4567",
        "(555) 123-4567",
        "555.123.4567",
        "5551234567",
        "123-45-678",
        "+1-555-123-4567",
    }
    
    fmt.Println("\nPhone validation:")
    for _, phone := range phones {
        valid := phonePattern.MatchString(phone)
        fmt.Printf("  %s: %t\n", phone, valid)
    }
    
    // Password strength validation
    passwordPatterns := map[string]*regexp.Regexp{
        "min8chars":    regexp.MustCompile(`^.{8,}$`),
        "hasUpper":     regexp.MustCompile(`[A-Z]`),
        "hasLower":     regexp.MustCompile(`[a-z]`),
        "hasDigit":     regexp.MustCompile(`\d`),
        "hasSpecial":   regexp.MustCompile(`[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]`),
    }
    
    passwords := []string{
        "password",
        "Password123",
        "Pass123!",
        "VeryStrong123!",
    }
    
    fmt.Println("\nPassword validation:")
    for _, pwd := range passwords {
        fmt.Printf("  %s:\n", pwd)
        for rule, pattern := range passwordPatterns {
            valid := pattern.MatchString(pwd)
            fmt.Printf("    %s: %t\n", rule, valid)
        }
        fmt.Println()
    }
}
```

Validation patterns typically use anchors (`^` and `$`) to match the entire  
input. Complex validation can combine multiple patterns to check different  
requirements. Character classes and quantifiers define acceptable formats  
and constraints for various data types.  


## Text extraction

Regular expressions can extract structured information from unstructured  
text, parsing complex formats and isolating specific data elements.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Extract URLs from text
    text := `Visit our website at https://example.com or check out the API at 
    https://api.example.com/v1/users. Also see http://docs.example.com for more info.`
    
    urlPattern := regexp.MustCompile(`https?://[^\s]+`)
    urls := urlPattern.FindAllString(text, -1)
    fmt.Printf("Found URLs: %v\n", urls)
    
    // Extract structured data from log entries
    logText := `
    2024-01-15 10:30:45 [ERROR] Database connection failed - host:db.example.com port:5432
    2024-01-15 10:31:02 [WARN] High CPU usage detected - usage:85%
    2024-01-15 10:31:15 [INFO] User login - user:admin ip:192.168.1.50
    `
    
    logPattern := regexp.MustCompile(`(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)`)
    logMatches := logPattern.FindAllStringSubmatch(logText, -1)
    
    fmt.Println("\nParsed log entries:")
    for _, match := range logMatches {
        if len(match) >= 4 {
            fmt.Printf("  Time: %s, Level: %s, Message: %s\n",
                match[1], match[2], match[3])
        }
    }
    
    // Extract metadata from filenames
    filenames := []string{
        "report_2024_01_15.pdf",
        "image_1920x1080_compressed.jpg",
        "data_export_v2.3.csv",
        "backup_full_20240115_143022.zip",
    }
    
    patterns := map[string]*regexp.Regexp{
        "date":       regexp.MustCompile(`(\d{4}[-_]\d{2}[-_]\d{2})`),
        "version":    regexp.MustCompile(`v?(\d+\.\d+(?:\.\d+)?)`),
        "dimensions": regexp.MustCompile(`(\d{3,4})x(\d{3,4})`),
        "timestamp":  regexp.MustCompile(`(\d{8}_\d{6})`),
    }
    
    fmt.Println("\nFilename analysis:")
    for _, filename := range filenames {
        fmt.Printf("  %s:\n", filename)
        for name, pattern := range patterns {
            matches := pattern.FindStringSubmatch(filename)
            if len(matches) > 1 {
                fmt.Printf("    %s: %s\n", name, matches[1])
            }
        }
    }
    
    // Extract hashtags and mentions from social media text
    socialText := "Check out this amazing #golang tutorial! Thanks @johndoe for sharing. #programming #coding"
    
    hashtagPattern := regexp.MustCompile(`#\w+`)
    mentionPattern := regexp.MustCompile(`@\w+`)
    
    hashtags := hashtagPattern.FindAllString(socialText, -1)
    mentions := mentionPattern.FindAllString(socialText, -1)
    
    fmt.Printf("\nSocial media extraction:\n")
    fmt.Printf("  Hashtags: %v\n", hashtags)
    fmt.Printf("  Mentions: %v\n", mentions)
}
```

Text extraction uses capturing groups to isolate specific parts of matches.  
Complex patterns can parse structured formats like logs, filenames, or  
social media content. The key is designing patterns that accurately  
identify and capture the desired information while handling variations.  


## Advanced patterns

Advanced regular expression techniques combine multiple concepts to solve  
complex text processing challenges with sophisticated pattern matching.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func main() {
    // Lookahead assertions (not directly supported in RE2, but alternatives)
    // Find words that contain 'a' but don't end with 'ing'
    text := "running walking talking sitting standing reading"
    
    // Alternative approach using multiple patterns
    hasA := regexp.MustCompile(`\w*a\w*`)
    notEndingIng := regexp.MustCompile(`\w+(?:ing)$`)
    
    wordsWithA := hasA.FindAllString(text, -1)
    var filtered []string
    for _, word := range wordsWithA {
        if !notEndingIng.MatchString(word) {
            filtered = append(filtered, word)
        }
    }
    fmt.Printf("Words with 'a' not ending in 'ing': %v\n", filtered)
    
    // Nested patterns and alternation
    codeText := `
    func calculateTotal(price float64, tax float64) float64 {
        return price + (price * tax)
    }
    
    var username = "john_doe"
    const PI = 3.14159
    `
    
    // Match various Go language constructs
    patterns := map[string]*regexp.Regexp{
        "functions": regexp.MustCompile(`func\s+(\w+)\s*\([^)]*\)\s*[^{]*`),
        "variables": regexp.MustCompile(`(?:var|const)\s+(\w+)\s*=`),
        "types":     regexp.MustCompile(`\b(int|float64|string|bool)\b`),
    }
    
    fmt.Println("\nCode analysis:")
    for name, pattern := range patterns {
        matches := pattern.FindAllString(codeText, -1)
        if len(matches) > 0 {
            fmt.Printf("  %s: %v\n", name, matches)
        }
    }
    
    // Complex email parsing with internationalized domain names
    complexEmails := `
    Contact: john.doe+newsletter@company-name.com
    Support: support@münchen.example.de  
    International: user@例え.テスト
    Unicode: test@домен.рф
    `
    
    // Enhanced email pattern supporting international characters
    intlEmailPattern := regexp.MustCompile(`[\w._%+-]+@[\w.-]+\.\w{2,}`)
    intlEmails := intlEmailPattern.FindAllString(complexEmails, -1)
    fmt.Printf("\nInternational emails: %v\n", intlEmails)
    
    // Pattern for parsing configuration files
    configText := `
    # Database configuration
    db.host = localhost
    db.port = 5432
    db.name = myapp
    
    # Server settings  
    server.port=8080
    server.debug = true
    `
    
    configPattern := regexp.MustCompile(`^\s*([a-zA-Z][a-zA-Z0-9_.]*)\s*=\s*(.+?)(?:\s*#.*)?$`)
    
    fmt.Println("\nConfiguration parsing:")
    for _, line := range strings.Split(configText, "\n") {
        matches := configPattern.FindStringSubmatch(line)
        if len(matches) >= 3 {
            key := matches[1]
            value := strings.TrimSpace(matches[2])
            fmt.Printf("  %s = %s\n", key, value)
        }
    }
}
```

Advanced patterns combine alternation (`|`), complex character classes,  
and multiple capturing groups. While RE2 doesn't support all PCRE features,  
creative combinations of multiple patterns can achieve sophisticated text  
processing goals through algorithmic approaches.  


## Performance considerations

Understanding regex performance characteristics helps optimize pattern  
matching operations for better application performance.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "time"
)

func main() {
    // Compile once, use many times
    pattern := regexp.MustCompile(`\b\w{5,}\b`)
    
    text := strings.Repeat("This is a sample text with various word lengths. ", 1000)
    
    // Measure compiled pattern performance
    start := time.Now()
    for i := 0; i < 100; i++ {
        _ = pattern.FindAllString(text, -1)
    }
    compiledTime := time.Since(start)
    
    // Measure recompilation overhead
    start = time.Now()
    for i := 0; i < 100; i++ {
        temp := regexp.MustCompile(`\b\w{5,}\b`)
        _ = temp.FindAllString(text, -1)
    }
    recompileTime := time.Since(start)
    
    fmt.Printf("Compiled pattern time: %v\n", compiledTime)
    fmt.Printf("Recompilation time: %v\n", recompileTime)
    fmt.Printf("Performance ratio: %.2fx\n", 
        float64(recompileTime.Nanoseconds())/float64(compiledTime.Nanoseconds()))
    
    // Efficient vs inefficient patterns
    data := "The quick brown fox jumps over the lazy dog. " + 
            "Pack my box with five dozen liquor jugs. " +
            "How vexingly quick daft zebras jump!"
    
    // Efficient: specific patterns
    efficientPattern := regexp.MustCompile(`\b[A-Z][a-z]+\b`)
    
    // Less efficient: overly broad patterns with backtracking potential
    broadPattern := regexp.MustCompile(`.*[A-Z].*`)
    
    start = time.Now()
    for i := 0; i < 10000; i++ {
        _ = efficientPattern.FindAllString(data, -1)
    }
    efficientTime := time.Since(start)
    
    start = time.Now()
    for i := 0; i < 10000; i++ {
        _ = broadPattern.FindAllString(data, -1)
    }
    broadTime := time.Since(start)
    
    fmt.Printf("\nPattern efficiency comparison:\n")
    fmt.Printf("Specific pattern: %v\n", efficientTime)
    fmt.Printf("Broad pattern: %v\n", broadTime)
    
    // Memory usage considerations
    largeText := strings.Repeat("word ", 100000)
    
    // FindAll with limit vs unlimited
    limitedResults := pattern.FindAllString(largeText, 100)
    unlimitedResults := pattern.FindAllString(largeText, -1)
    
    fmt.Printf("\nMemory efficiency:\n")
    fmt.Printf("Limited results: %d matches\n", len(limitedResults))
    fmt.Printf("Unlimited results: %d matches\n", len(unlimitedResults))
    
    // String vs byte slice operations
    textBytes := []byte(data)
    
    start = time.Now()
    for i := 0; i < 10000; i++ {
        _ = pattern.FindAllString(data, -1)
    }
    stringTime := time.Since(start)
    
    start = time.Now()
    for i := 0; i < 10000; i++ {
        _ = pattern.FindAll(textBytes, -1)
    }
    byteTime := time.Since(start)
    
    fmt.Printf("\nString vs byte operations:\n")
    fmt.Printf("String operations: %v\n", stringTime)
    fmt.Printf("Byte operations: %v\n", byteTime)
}
```

Key performance principles: compile patterns once and reuse them, use specific  
patterns rather than overly broad ones, limit results when possible, and  
consider byte slice operations for large data processing. The RE2 engine's  
linear time guarantee prevents catastrophic backtracking issues common in  
other regex engines.  


## Unicode and international text

Go's regular expressions provide excellent Unicode support for international  
text processing and multilingual pattern matching.  

```go
package main

import (
    "fmt"
    "regexp"
    "unicode"
)

func main() {
    // Unicode text examples
    multilingualText := `
    Hello there (English)
    Bonjour là-bas (French)
    Hola amigo (Spanish)
    こんにちは (Japanese)
    Привет друг (Russian)
    مرحبا صديق (Arabic)
    你好朋友 (Chinese)
    `
    
    // Unicode letter matching
    letterPattern := regexp.MustCompile(`\p{L}+`)
    allLetters := letterPattern.FindAllString(multilingualText, -1)
    fmt.Printf("Unicode letters: %v\n", allLetters)
    
    // Specific Unicode categories
    patterns := map[string]*regexp.Regexp{
        "Latin":    regexp.MustCompile(`\p{Latin}+`),
        "Cyrillic": regexp.MustCompile(`\p{Cyrillic}+`),
        "Arabic":   regexp.MustCompile(`\p{Arabic}+`),
        "Han":      regexp.MustCompile(`\p{Han}+`),
        "Hiragana": regexp.MustCompile(`\p{Hiragana}+`),
        "Katakana": regexp.MustCompile(`\p{Katakana}+`),
    }
    
    fmt.Println("\nScript-specific matching:")
    for script, pattern := range patterns {
        matches := pattern.FindAllString(multilingualText, -1)
        if len(matches) > 0 {
            fmt.Printf("  %s: %v\n", script, matches)
        }
    }
    
    // Case-insensitive Unicode matching
    mixedCaseText := "CAFÉ café Café МОСКВА москва Москва"
    casePattern := regexp.MustCompile(`(?i)\bcafé\b`)
    cafeMatches := casePattern.FindAllString(mixedCaseText, -1)
    fmt.Printf("\nCase-insensitive 'café': %v\n", cafeMatches)
    
    cyrillicPattern := regexp.MustCompile(`(?i)\bмосква\b`)
    moscowMatches := cyrillicPattern.FindAllString(mixedCaseText, -1)
    fmt.Printf("Case-insensitive 'москва': %v\n", moscowMatches)
    
    // Digit matching across different Unicode digit systems
    digitText := "English: 123, Arabic: ١٢٣, Devanagari: १२३, Thai: ๑๒๓"
    
    // Unicode digit class matches all digit systems
    unicodeDigitPattern := regexp.MustCompile(`\p{Nd}+`)
    allDigits := unicodeDigitPattern.FindAllString(digitText, -1)
    fmt.Printf("\nAll Unicode digits: %v\n", allDigits)
    
    // ASCII digits only
    asciiDigitPattern := regexp.MustCompile(`[0-9]+`)
    asciiDigits := asciiDigitPattern.FindAllString(digitText, -1)
    fmt.Printf("ASCII digits only: %v\n", asciiDigits)
    
    // Word boundary behavior with Unicode
    unicodeWords := "word単語мотслово"
    
    // Standard word boundaries
    wordBoundary := regexp.MustCompile(`\b\w+\b`)
    words := wordBoundary.FindAllString(unicodeWords, -1)
    fmt.Printf("\nWord boundaries: %v\n", words)
    
    // Manual Unicode word detection
    isWordChar := func(r rune) bool {
        return unicode.IsLetter(r) || unicode.IsDigit(r) || r == '_'
    }
    
    var unicodeWordsSlice []string
    var currentWord []rune
    
    for _, r := range unicodeWords {
        if isWordChar(r) {
            currentWord = append(currentWord, r)
        } else {
            if len(currentWord) > 0 {
                unicodeWordsSlice = append(unicodeWordsSlice, string(currentWord))
                currentWord = nil
            }
        }
    }
    if len(currentWord) > 0 {
        unicodeWordsSlice = append(unicodeWordsSlice, string(currentWord))
    }
    
    fmt.Printf("Manual Unicode words: %v\n", unicodeWordsSlice)
}
```

Unicode support in Go regex includes script categories (`\p{Latin}`,  
`\p{Cyrillic}`), general categories (`\p{L}` for letters, `\p{Nd}` for  
digits), and case-insensitive matching across all Unicode characters.  
Understanding Unicode categories enables proper international text processing.  


## Working with byte slices

Regular expressions can operate on byte slices for more efficient processing  
of large data or binary content containing text patterns.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Create sample byte data
    data := []byte("The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs.")
    
    // Compile pattern for byte operations
    pattern := regexp.MustCompile(`\b\w{4,}\b`)
    
    // Find operations with bytes
    firstMatch := pattern.Find(data)
    fmt.Printf("First long word (bytes): %s\n", string(firstMatch))
    
    // Find all matches
    allMatches := pattern.FindAll(data, -1)
    fmt.Printf("All long words: ")
    for i, match := range allMatches {
        if i > 0 {
            fmt.Print(", ")
        }
        fmt.Print(string(match))
    }
    fmt.Println()
    
    // Find with indices
    indices := pattern.FindAllIndex(data, -1)
    fmt.Println("\nWord positions:")
    for i, idx := range indices {
        word := string(data[idx[0]:idx[1]])
        fmt.Printf("  Word %d: '%s' at [%d:%d]\n", i+1, word, idx[0], idx[1])
    }
    
    // Find with submatches
    submatchPattern := regexp.MustCompile(`(\w+)\s+(brown|lazy)\s+(\w+)`)
    submatchBytes := submatchPattern.FindSubmatch(data)
    
    if len(submatchBytes) > 0 {
        fmt.Printf("\nSubmatches:\n")
        fmt.Printf("  Full match: %s\n", string(submatchBytes[0]))
        fmt.Printf("  First word: %s\n", string(submatchBytes[1]))
        fmt.Printf("  Color/state: %s\n", string(submatchBytes[2]))
        fmt.Printf("  Third word: %s\n", string(submatchBytes[3]))
    }
    
    // Replacement operations with bytes
    digitPattern := regexp.MustCompile(`\d+`)
    testData := []byte("I have 42 apples and 17 oranges")
    
    // Simple replacement
    replaced := digitPattern.ReplaceAll(testData, []byte("[NUM]"))
    fmt.Printf("\nReplaced numbers: %s\n", string(replaced))
    
    // Replacement with function
    doubled := digitPattern.ReplaceAllFunc(testData, func(match []byte) []byte {
        return []byte(fmt.Sprintf("[%s*2]", string(match)))
    })
    fmt.Printf("Doubled numbers: %s\n", string(doubled))
    
    // Split operations
    separatorPattern := regexp.MustCompile(`[,.\s]+`)
    splitData := separatorPattern.Split(data, -1)
    fmt.Printf("\nSplit into words: %v\n", func() []string {
        var result []string
        for _, part := range splitData {
            if len(part) > 0 {
                result = append(result, string(part))
            }
        }
        return result
    }())
    
    // Memory efficiency comparison
    largeData := make([]byte, 1000000)
    copy(largeData, []byte("test data with patterns repeated over and over again "))
    
    // Working directly with bytes avoids string conversions
    wordPattern := regexp.MustCompile(`\b[a-z]+\b`)
    byteMatches := wordPattern.FindAll(largeData[:100], -1)
    
    fmt.Printf("\nByte slice efficiency: found %d matches\n", len(byteMatches))
    for i, match := range byteMatches {
        if i >= 5 {
            fmt.Print("...")
            break
        }
        if i > 0 {
            fmt.Print(", ")
        }
        fmt.Printf("'%s'", string(match))
    }
    fmt.Println()
}
```

Byte slice operations avoid string conversion overhead and are ideal for  
processing large files or binary data. All regex methods have byte slice  
equivalents: `Find`, `FindAll`, `FindIndex`, `ReplaceAll`, etc. The results  
are byte slices that can be converted to strings when needed.  


## Matching options and flags

Regular expression compilation accepts various flags that modify pattern  
matching behavior for specific requirements.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := `
    First Line
    SECOND LINE
    third line
    Fourth Line
    `
    
    // Case-insensitive flag (?i)
    casePattern := regexp.MustCompile(`(?i)line`)
    caseMatches := casePattern.FindAllString(text, -1)
    fmt.Printf("Case-insensitive matches: %v\n", caseMatches)
    
    // Multiline flag (?m) - ^ and $ match line boundaries
    multiText := "start of first line\nstart of second line\nend of third line"
    multiPattern := regexp.MustCompile(`(?m)^start`)
    multiMatches := multiPattern.FindAllString(multiText, -1)
    fmt.Printf("Multiline start matches: %v\n", multiMatches)
    
    // Single-line flag (?s) - . matches newlines
    dotText := "first part\nsecond part\nthird part"
    
    // Without single-line flag
    normalDot := regexp.MustCompile(`first.*second`)
    normalMatch := normalDot.FindString(dotText)
    fmt.Printf("Normal dot (no newline): '%s'\n", normalMatch)
    
    // With single-line flag
    singleDot := regexp.MustCompile(`(?s)first.*second`)
    singleMatch := singleDot.FindString(dotText)
    fmt.Printf("Single-line dot (with newline): '%s'\n", singleMatch)
    
    // Combining flags
    combinedText := `
    FIRST line content
    second LINE content  
    THIRD line CONTENT
    `
    
    combinedPattern := regexp.MustCompile(`(?im)^.*line.*$`)
    combinedMatches := combinedPattern.FindAllString(combinedText, -1)
    fmt.Printf("Combined flags matches: %v\n", combinedMatches)
    
    // Ungreedy matching (default is greedy)
    htmlText := "<div>content1</div><div>content2</div>"
    
    // Greedy quantifier (default)
    greedyPattern := regexp.MustCompile(`<div>.*</div>`)
    greedyMatch := greedyPattern.FindString(htmlText)
    fmt.Printf("Greedy match: '%s'\n", greedyMatch)
    
    // Non-greedy quantifier
    nonGreedyPattern := regexp.MustCompile(`<div>.*?</div>`)
    nonGreedyMatches := nonGreedyPattern.FindAllString(htmlText, -1)
    fmt.Printf("Non-greedy matches: %v\n", nonGreedyMatches)
    
    // Flag scope and groups
    scopedText := "ABC def GHI"
    
    // Flag applies to entire pattern
    globalFlag := regexp.MustCompile(`(?i)abc.*ghi`)
    globalMatch := globalFlag.FindString(scopedText)
    fmt.Printf("Global flag match: '%s'\n", globalMatch)
    
    // Practical example: parsing configuration with flags
    configText := `
    # Configuration file
    HOST = localhost
    port = 8080
    DEBUG = true
    database_url = postgresql://user:pass@host:5432/db
    `
    
    // Case-insensitive, multiline matching for config entries
    configPattern := regexp.MustCompile(`(?im)^\s*([a-z_]+)\s*=\s*(.+?)(?:\s*#.*)?$`)
    configMatches := configPattern.FindAllStringSubmatch(configText, -1)
    
    fmt.Printf("\nConfiguration entries:\n")
    for _, match := range configMatches {
        if len(match) >= 3 {
            key := match[1]
            value := match[2]
            fmt.Printf("  %s = %s\n", key, value)
        }
    }
}
```

Flags modify regex behavior: `(?i)` for case-insensitive matching, `(?m)`  
for multiline mode, `(?s)` for single-line mode where `.` matches newlines.  
Flags can be combined and affect the entire pattern from their position  
onward. Non-greedy quantifiers (`*?`, `+?`) match minimal text.  


## Error handling

Proper error handling ensures robust regular expression operations and  
graceful failure management in production applications.  

```go
package main

import (
    "fmt"
    "regexp"
)

func compilePattern(pattern string) (*regexp.Regexp, error) {
    compiled, err := regexp.Compile(pattern)
    if err != nil {
        return nil, fmt.Errorf("failed to compile pattern '%s': %w", pattern, err)
    }
    return compiled, nil
}

func safeMatch(pattern, text string) (bool, error) {
    compiled, err := compilePattern(pattern)
    if err != nil {
        return false, err
    }
    return compiled.MatchString(text), nil
}

func main() {
    // Valid patterns
    validPatterns := []string{
        `\d+`,
        `[a-zA-Z]+`,
        `\w+@\w+\.\w+`,
        `^[A-Z][a-z]*$`,
    }
    
    fmt.Println("Testing valid patterns:")
    for _, pattern := range validPatterns {
        compiled, err := compilePattern(pattern)
        if err != nil {
            fmt.Printf("  Error with '%s': %v\n", pattern, err)
        } else {
            fmt.Printf("  Pattern '%s': compiled successfully\n", pattern)
            // Test the pattern
            testResult := compiled.MatchString("Test123")
            fmt.Printf("    Matches 'Test123': %t\n", testResult)
        }
    }
    
    // Invalid patterns that will cause errors
    invalidPatterns := []string{
        `[`,          // Unclosed bracket
        `*`,          // Invalid quantifier
        `(?P<>test)`, // Invalid named group
        `\k<name>`,   // Invalid back-reference (not supported in RE2)
        `(?<=test)`,  // Lookbehind (not supported in RE2)
    }
    
    fmt.Println("\nTesting invalid patterns:")
    for _, pattern := range invalidPatterns {
        compiled, err := compilePattern(pattern)
        if err != nil {
            fmt.Printf("  Pattern '%s': %v\n", pattern, err)
        } else {
            fmt.Printf("  Pattern '%s': unexpectedly compiled\n", pattern)
            _ = compiled // Use compiled to avoid unused variable warning
        }
    }
    
    // Safe matching with error handling
    fmt.Println("\nSafe matching examples:")
    testCases := []struct {
        pattern string
        text    string
    }{
        {`\d+`, "abc123def"},
        {`[`, "test"},  // Invalid pattern
        {`^[A-Z][a-z]+$`, "Hello"},
        {`(?i)hello`, "HELLO THERE"},
    }
    
    for _, tc := range testCases {
        matched, err := safeMatch(tc.pattern, tc.text)
        if err != nil {
            fmt.Printf("  Error matching '%s' against '%s': %v\n", 
                tc.pattern, tc.text, err)
        } else {
            fmt.Printf("  Pattern '%s' matches '%s': %t\n", 
                tc.pattern, tc.text, matched)
        }
    }
    
    // Handling panics from MustCompile
    fmt.Println("\nDemonstrating MustCompile panic handling:")
    
    func() {
        defer func() {
            if r := recover(); r != nil {
                fmt.Printf("  Recovered from panic: %v\n", r)
            }
        }()
        
        // This will panic due to invalid pattern
        pattern := regexp.MustCompile(`[`)
        _ = pattern // This line won't be reached
    }()
    
    // Best practices for production code
    fmt.Println("\nBest practices example:")
    
    type PatternValidator struct {
        patterns map[string]*regexp.Regexp
    }
    
    validator := &PatternValidator{
        patterns: make(map[string]*regexp.Regexp),
    }
    
    // Pre-compile patterns during initialization
    patternDefinitions := map[string]string{
        "email":    `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`,
        "phone":    `^\d{3}-\d{3}-\d{4}$`,
        "zipcode":  `^\d{5}(-\d{4})?$`,
    }
    
    for name, pattern := range patternDefinitions {
        compiled, err := regexp.Compile(pattern)
        if err != nil {
            fmt.Printf("  Failed to initialize %s pattern: %v\n", name, err)
            continue
        }
        validator.patterns[name] = compiled
        fmt.Printf("  Initialized %s pattern successfully\n", name)
    }
    
    // Use pre-compiled patterns
    testData := map[string]string{
        "email":   "user@example.com",
        "phone":   "555-123-4567",
        "zipcode": "12345-6789",
    }
    
    fmt.Println("\nValidation results:")
    for dataType, value := range testData {
        if pattern, exists := validator.patterns[dataType]; exists {
            valid := pattern.MatchString(value)
            fmt.Printf("  %s '%s' is valid: %t\n", dataType, value, valid)
        } else {
            fmt.Printf("  No pattern available for %s\n", dataType)
        }
    }
}
```

Error handling is crucial for regex operations. Use `regexp.Compile()` and  
check errors for dynamic patterns. Pre-compile patterns at initialization  
for better performance and early error detection. Handle panics from  
`MustCompile()` when necessary, but prefer `Compile()` for production code.  


## Real-world applications

Practical applications demonstrate how regular expressions solve common  
programming challenges in data processing and text manipulation tasks.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func main() {
    // Log file parsing and analysis
    logEntries := `
    192.168.1.100 - - [15/Jan/2024:14:30:45 +0000] "GET /api/users HTTP/1.1" 200 1234
    192.168.1.101 - - [15/Jan/2024:14:31:02 +0000] "POST /api/login HTTP/1.1" 401 567
    192.168.1.102 - - [15/Jan/2024:14:31:15 +0000] "GET /static/app.js HTTP/1.1" 304 0
    10.0.0.50 - - [15/Jan/2024:14:32:01 +0000] "DELETE /api/users/123 HTTP/1.1" 204 0
    `
    
    logPattern := regexp.MustCompile(`(\d+\.\d+\.\d+\.\d+).*?\[([^\]]+)\].*?"(\w+)\s+([^"]*?)\s+HTTP.*?"\s+(\d+)\s+(\d+)`)
    
    fmt.Println("Apache log analysis:")
    logMatches := logPattern.FindAllStringSubmatch(logEntries, -1)
    
    for _, match := range logMatches {
        if len(match) >= 7 {
            ip := match[1]
            timestamp := match[2]
            method := match[3]
            path := match[4]
            statusCode := match[5]
            responseSize := match[6]
            
            fmt.Printf("  %s [%s] %s %s -> %s (%s bytes)\n",
                ip, timestamp, method, path, statusCode, responseSize)
        }
    }
    
    // Email template processing
    emailTemplate := `
    Dear {{.Name}},
    
    Thank you for your order #{{.OrderID}}. Your total is ${{.Total}}.
    
    Items:
    {{.ItemList}}
    
    Best regards,
    The Sales Team
    `
    
    templatePattern := regexp.MustCompile(`\{\{\.(\w+)\}\}`)
    variables := templatePattern.FindAllStringSubmatch(emailTemplate, -1)
    
    fmt.Printf("\nTemplate variables found: ")
    var varNames []string
    for _, match := range variables {
        if len(match) >= 2 {
            varNames = append(varNames, match[1])
        }
    }
    fmt.Printf("%v\n", varNames)
    
    // Code comment extraction
    sourceCode := `
    // This is a single line comment
    func main() {
        /* This is a 
           multi-line comment */
        fmt.Println("Hello there") // Another comment
        /* Another multi-line
           comment here */
    }
    `
    
    singleLinePattern := regexp.MustCompile(`//.*$`)
    multiLinePattern := regexp.MustCompile(`(?s)/\*.*?\*/`)
    
    singleComments := singleLinePattern.FindAllString(sourceCode, -1)
    multiComments := multiLinePattern.FindAllString(sourceCode, -1)
    
    fmt.Printf("\nCode comments:\n")
    fmt.Printf("  Single-line: %v\n", singleComments)
    fmt.Printf("  Multi-line: %v\n", multiComments)
    
    // Data validation and sanitization
    userInputs := []string{
        "john.doe@company.com",
        "555-123-4567",
        "<script>alert('xss')</script>",
        "Valid text input",
        "DROP TABLE users;--",
        "123-45-6789",  // SSN format
    }
    
    validators := map[string]*regexp.Regexp{
        "email":       regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        "phone":       regexp.MustCompile(`^\d{3}-\d{3}-\d{4}$`),
        "safe_text":   regexp.MustCompile(`^[a-zA-Z0-9\s.,!?-]+$`),
        "ssn":         regexp.MustCompile(`^\d{3}-\d{2}-\d{4}$`),
    }
    
    // HTML/script tag detection for XSS prevention
    dangerousPattern := regexp.MustCompile(`(?i)<\s*(script|iframe|object|embed)`)
    sqlPattern := regexp.MustCompile(`(?i)(drop\s+table|delete\s+from|insert\s+into|update\s+.*set)`)
    
    fmt.Printf("\nInput validation:\n")
    for _, input := range userInputs {
        fmt.Printf("  Input: %s\n", input)
        
        // Check against validators
        for name, pattern := range validators {
            if pattern.MatchString(input) {
                fmt.Printf("    Matches %s format\n", name)
            }
        }
        
        // Security checks
        if dangerousPattern.MatchString(input) {
            fmt.Printf("    ⚠️  Contains potentially dangerous HTML/script tags\n")
        }
        if sqlPattern.MatchString(input) {
            fmt.Printf("    ⚠️  Contains potential SQL injection patterns\n")
        }
        
        fmt.Println()
    }
    
    // URL routing pattern matching
    routes := []string{
        "/api/users",
        "/api/users/123",
        "/api/users/123/posts",
        "/api/users/123/posts/456",
        "/static/css/style.css",
        "/admin/dashboard",
    }
    
    routePatterns := map[string]*regexp.Regexp{
        "user_list":   regexp.MustCompile(`^/api/users$`),
        "user_detail": regexp.MustCompile(`^/api/users/(\d+)$`),
        "user_posts":  regexp.MustCompile(`^/api/users/(\d+)/posts$`),
        "post_detail": regexp.MustCompile(`^/api/users/(\d+)/posts/(\d+)$`),
        "static":      regexp.MustCompile(`^/static/`),
        "admin":       regexp.MustCompile(`^/admin/`),
    }
    
    fmt.Printf("URL routing:\n")
    for _, route := range routes {
        fmt.Printf("  Route: %s\n", route)
        for name, pattern := range routePatterns {
            if matches := pattern.FindStringSubmatch(route); matches != nil {
                fmt.Printf("    Matches: %s", name)
                if len(matches) > 1 {
                    fmt.Printf(" (parameters: %v)", matches[1:])
                }
                fmt.Println()
                break
            }
        }
    }
}
```

Real-world applications include log parsing, template processing, code  
analysis, input validation, security filtering, and URL routing. Regular  
expressions excel at extracting structured data from unstructured text  
and enforcing data format constraints in production systems.  

