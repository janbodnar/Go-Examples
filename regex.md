# Regular Expressions

Regular expressions in Go provide powerful pattern matching and text processing  
capabilities through the `regexp` package. Go's regex engine is based on RE2,  
which ensures linear time performance and prevents catastrophic backtracking  
issues found in some other regex implementations. This makes Go's regex engine  
particularly suitable for processing untrusted input in web applications and  
data processing pipelines.

The `regexp` package follows Go's philosophy of simplicity and efficiency. It  
provides a clean API with methods for matching, finding, replacing, and  
splitting text. Regular expressions must be compiled before use, either with  
`Compile` for safe compilation that returns an error, or `MustCompile` for  
compilation that panics on invalid patterns. The compiled regex objects are  
safe for concurrent use, making them ideal for server applications.

Go's regex syntax is mostly compatible with Perl, Python, and other common  
regex flavors, but with some limitations due to the RE2 engine. Features like  
backreferences and lookarounds are not supported, but the engine compensates  
with guaranteed linear time complexity and excellent Unicode support. The  
package handles UTF-8 strings natively, making it seamless to work with  
international text and emoji in patterns.

## Basic pattern matching

Simple pattern matching forms the foundation of regex usage, allowing you to  
check if text contains specific patterns.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "The quick brown fox jumps over the lazy dog"
    
    // Check if text contains 'fox'
    matched, _ := regexp.MatchString("fox", text)
    fmt.Printf("Contains 'fox': %t\n", matched)
    
    // Case-sensitive matching
    matched, _ = regexp.MatchString("Fox", text)
    fmt.Printf("Contains 'Fox': %t\n", matched)
    
    // Pattern with multiple options
    matched, _ = regexp.MatchString("cat|dog", text)
    fmt.Printf("Contains 'cat' or 'dog': %t\n", matched)
    
    // Word boundary matching
    matched, _ = regexp.MatchString(`\bdog\b`, text)
    fmt.Printf("Contains whole word 'dog': %t\n", matched)
}
```

This example demonstrates basic pattern matching using `MatchString`. The  
function returns true if the pattern is found anywhere in the text. Word  
boundaries (`\b`) ensure exact word matches rather than partial matches.  

## Compiling regular expressions

Compiling regex patterns once and reusing them improves performance when  
processing multiple strings with the same pattern.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Safe compilation with error handling
    pattern, err := regexp.Compile(`\d{3}-\d{3}-\d{4}`)
    if err != nil {
        fmt.Printf("Invalid regex pattern: %v\n", err)
        return
    }
    
    // Using MustCompile for patterns known to be valid
    emailPattern := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    
    testStrings := []string{
        "Call me at 555-123-4567",
        "Email: user@example.com",
        "Invalid phone: 555-12-34",
        "Contact: admin@domain.org",
    }
    
    fmt.Println("Phone number matches:")
    for _, str := range testStrings {
        if pattern.MatchString(str) {
            fmt.Printf("  ✓ %s\n", str)
        }
    }
    
    fmt.Println("\nEmail matches:")
    for _, str := range testStrings {
        if emailPattern.MatchString(str) {
            fmt.Printf("  ✓ %s\n", str)
        }
    }
}
```

Compiled patterns are more efficient for repeated use and provide better error  
handling. The `MustCompile` function is convenient for hardcoded patterns  
where you're confident they're valid.  

## Finding substrings

The `FindString` method locates the first occurrence of a pattern and returns  
the matching substring.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Visit our website at https://example.com or https://backup.example.com"
    
    // Find first URL
    urlPattern := regexp.MustCompile(`https?://[^\s]+`)
    firstURL := urlPattern.FindString(text)
    fmt.Printf("First URL found: %s\n", firstURL)
    
    // Find all URLs
    allURLs := urlPattern.FindAllString(text, -1)
    fmt.Printf("All URLs found: %v\n", allURLs)
    
    // Find with limit
    limitedURLs := urlPattern.FindAllString(text, 1)
    fmt.Printf("First URL only: %v\n", limitedURLs)
    
    // Find numbers
    numberPattern := regexp.MustCompile(`\d+`)
    numbers := numberPattern.FindAllString("There are 123 apples and 456 oranges", -1)
    fmt.Printf("Numbers found: %v\n", numbers)
}
```

`FindAllString` retrieves all matches when passed -1 as the count parameter,  
or limits results when given a positive number. This is useful for extracting  
multiple occurrences of patterns from text.  

## Finding with indices

Sometimes you need to know where matches occur in the original text, not just  
what they contain.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "Product codes: ABC123, DEF456, GHI789"
    pattern := regexp.MustCompile(`[A-Z]{3}\d{3}`)
    
    // Find first match with indices
    indices := pattern.FindStringIndex(text)
    if indices != nil {
        fmt.Printf("First match at positions: %d-%d\n", indices[0], indices[1])
        fmt.Printf("Matched text: %s\n", text[indices[0]:indices[1]])
    }
    
    // Find all matches with indices
    allIndices := pattern.FindAllStringIndex(text, -1)
    fmt.Println("All matches with positions:")
    for i, idx := range allIndices {
        match := text[idx[0]:idx[1]]
        fmt.Printf("  %d: '%s' at %d-%d\n", i+1, match, idx[0], idx[1])
    }
    
    // Using indices to build custom results
    fmt.Println("Custom formatting:")
    for _, idx := range allIndices {
        before := text[:idx[0]]
        match := text[idx[0]:idx[1]]
        after := text[idx[1]:]
        fmt.Printf("  [%s]>>%s<<[%s]\n", before, match, after)
    }
}
```

Index-based functions return byte positions in the original string, allowing  
you to extract context around matches or perform complex text manipulations  
based on match locations.  

## Capture groups

Parentheses in regex patterns create capture groups that allow extraction of  
specific parts of matches.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    text := "John Doe (30 years old), Jane Smith (25 years old)"
    
    // Pattern with named groups
    pattern := regexp.MustCompile(`(?P<name>\w+\s+\w+)\s+\((?P<age>\d+)\s+years\s+old\)`)
    
    // Find first match with submatches
    matches := pattern.FindStringSubmatch(text)
    if matches != nil {
        fmt.Println("First person details:")
        fmt.Printf("  Full match: %s\n", matches[0])
        fmt.Printf("  Name: %s\n", matches[1])
        fmt.Printf("  Age: %s\n", matches[2])
    }
    
    // Find all matches with submatches
    allMatches := pattern.FindAllStringSubmatch(text, -1)
    fmt.Println("\nAll people:")
    for i, match := range allMatches {
        fmt.Printf("  Person %d: %s, Age: %s\n", i+1, match[1], match[2])
    }
    
    // Using named groups
    names := pattern.SubexpNames()
    fmt.Println("\nUsing named groups:")
    for _, match := range allMatches {
        result := make(map[string]string)
        for i, name := range names {
            if i != 0 && name != "" {
                result[name] = match[i]
            }
        }
        fmt.Printf("  %s is %s years old\n", result["name"], result["age"])
    }
}
```

Named capture groups improve code readability and make patterns self-documenting.  
The `SubexpNames` method returns group names, enabling map-based access to  
captured values.  

## Text replacement

Regular expressions excel at finding and replacing text patterns with new  
content, including dynamic replacements using captured groups.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func main() {
    text := "Contact us at support@company.com or sales@company.com"
    
    // Simple replacement
    emailPattern := regexp.MustCompile(`\b[\w._%+-]+@[\w.-]+\.\w{2,}\b`)
    redacted := emailPattern.ReplaceAllString(text, "[EMAIL]")
    fmt.Printf("Redacted: %s\n", redacted)
    
    // Replacement with captured groups
    phoneText := "Call 555-123-4567 or 555-987-6543 for assistance"
    phonePattern := regexp.MustCompile(`(\d{3})-(\d{3})-(\d{4})`)
    formatted := phonePattern.ReplaceAllString(phoneText, "($1) $2-$3")
    fmt.Printf("Formatted phones: %s\n", formatted)
    
    // Conditional replacement using function
    censorPattern := regexp.MustCompile(`\b(bad|terrible|awful)\b`)
    censored := censorPattern.ReplaceAllStringFunc(text+" This service is terrible", 
        func(match string) string {
            return strings.Repeat("*", len(match))
        })
    fmt.Printf("Censored: %s\n", censored)
    
    // Replace with numbered references
    dateText := "Today is 2024-03-15"
    datePattern := regexp.MustCompile(`(\d{4})-(\d{2})-(\d{2})`)
    americanDate := datePattern.ReplaceAllString(dateText, "$2/$3/$1")
    fmt.Printf("American format: %s\n", americanDate)
}
```

Replacement patterns support backreferences ($1, $2, etc.) to reuse captured  
groups. Function-based replacement enables complex transformations and  
conditional logic during replacement.  

## Splitting strings

Regular expressions can split strings on complex patterns, not just literal  
characters.  

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // Split on various separators
    text := "apple,banana;orange:grape|mango"
    separatorPattern := regexp.MustCompile(`[,;:|]`)
    fruits := separatorPattern.Split(text, -1)
    fmt.Printf("Fruits: %v\n", fruits)
    
    // Split on whitespace (including multiple spaces/tabs)
    messyText := "word1    word2\t\tword3\n\nword4"
    whitespacePattern := regexp.MustCompile(`\s+`)
    words := whitespacePattern.Split(messyText, -1)
    fmt.Printf("Clean words: %v\n", words)
    
    // Split with limit
    limitedSplit := separatorPattern.Split(text, 3)
    fmt.Printf("Limited split: %v\n", limitedSplit)
    
    // Split sentences on punctuation
    paragraph := "First sentence. Second sentence! Third sentence? Fourth sentence."
    sentencePattern := regexp.MustCompile(`[.!?]+\s*`)
    sentences := sentencePattern.Split(paragraph, -1)
    fmt.Println("Sentences:")
    for i, sentence := range sentences {
        if sentence != "" {
            fmt.Printf("  %d: %s\n", i+1, sentence)
        }
    }
}
```

The `Split` method divides strings at pattern boundaries. Using -1 as the  
count parameter splits on all occurrences, while positive numbers limit  
the number of splits.  

## Email validation

Email validation demonstrates practical regex usage for input validation  
and data quality assurance.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type EmailValidator struct {
    basicPattern    *regexp.Regexp
    strictPattern   *regexp.Regexp
}

func NewEmailValidator() *EmailValidator {
    return &EmailValidator{
        basicPattern:  regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        strictPattern: regexp.MustCompile(`^[a-zA-Z0-9.!#$%&'*+/=?^_` + "`" + `{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$`),
    }
}

func (ev *EmailValidator) IsValidBasic(email string) bool {
    return ev.basicPattern.MatchString(email)
}

func (ev *EmailValidator) IsValidStrict(email string) bool {
    return ev.strictPattern.MatchString(email)
}

func main() {
    validator := NewEmailValidator()
    
    testEmails := []string{
        "user@example.com",
        "test.email@domain.org",
        "user+label@example.co.uk",
        "invalid.email",
        "@domain.com",
        "user@",
        "user@domain",
        "user@domain.",
        "very.long.email.address@subdomain.example.com",
    }
    
    fmt.Println("Email validation results:")
    fmt.Printf("%-40s %-8s %-8s\n", "Email", "Basic", "Strict")
    fmt.Println(strings.Repeat("-", 58))
    
    for _, email := range testEmails {
        basic := validator.IsValidBasic(email)
        strict := validator.IsValidStrict(email)
        fmt.Printf("%-40s %-8t %-8t\n", email, basic, strict)
    }
}
```

Multiple validation levels provide flexibility for different use cases. Basic  
validation catches obvious errors, while strict validation follows RFC  
specifications more closely.  

## URL extraction

Extracting URLs from text is a common task in web scraping and content  
analysis applications.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type URLExtractor struct {
    httpPattern   *regexp.Regexp
    domainPattern *regexp.Regexp
}

func NewURLExtractor() *URLExtractor {
    return &URLExtractor{
        httpPattern:   regexp.MustCompile(`https?://(?:[-\w.])+(?:\:[0-9]+)?(?:/(?:[\w/_.])*(?:\?(?:[\w&=%.])*)?(?:\#(?:[\w.])*)?)?`),
        domainPattern: regexp.MustCompile(`(?i)\b(?:[a-z0-9](?:[a-z0-9-]{0,61}[a-z0-9])?\.)+[a-z0-9](?:[a-z0-9-]{0,61}[a-z0-9])?\b`),
    }
}

func (ue *URLExtractor) ExtractURLs(text string) []string {
    return ue.httpPattern.FindAllString(text, -1)
}

func (ue *URLExtractor) ExtractDomains(text string) []string {
    domains := ue.domainPattern.FindAllString(text, -1)
    // Filter out obvious non-domains
    var result []string
    for _, domain := range domains {
        if strings.Contains(domain, ".") && !strings.HasPrefix(domain, ".") {
            result = append(result, domain)
        }
    }
    return result
}

func main() {
    extractor := NewURLExtractor()
    
    text := `Check out these websites:
    - https://www.example.com/page?param=value
    - http://subdomain.test.org:8080/path
    - Visit golang.org for documentation
    - Email us at contact@company.com
    - Download from ftp.server.net
    - Invalid: not.a.real.url.xyz`
    
    urls := extractor.ExtractURLs(text)
    fmt.Println("Extracted URLs:")
    for i, url := range urls {
        fmt.Printf("  %d: %s\n", i+1, url)
    }
    
    domains := extractor.ExtractDomains(text)
    fmt.Println("\nExtracted domains:")
    for i, domain := range domains {
        fmt.Printf("  %d: %s\n", i+1, domain)
    }
    
    // Extract with context
    fmt.Println("\nURLs with context:")
    indices := extractor.httpPattern.FindAllStringIndex(text, -1)
    for _, idx := range indices {
        start := max(0, idx[0]-20)
        end := min(len(text), idx[1]+20)
        context := text[start:end]
        url := text[idx[0]:idx[1]]
        fmt.Printf("  %s\n  Context: ...%s...\n", url, context)
    }
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

URL extraction patterns handle various URL formats and schemes. The extractor  
provides both full URL and domain-only extraction with context information  
for better understanding.  

## Phone number validation

Phone number patterns vary by country and format, making validation complex  
but important for user interfaces.  

```go
package main

import (
    "fmt"
    "regexp"
)

type PhoneValidator struct {
    usPattern       *regexp.Regexp
    internationalPattern *regexp.Regexp
    digitsOnlyPattern    *regexp.Regexp
}

func NewPhoneValidator() *PhoneValidator {
    return &PhoneValidator{
        usPattern:       regexp.MustCompile(`^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$`),
        internationalPattern: regexp.MustCompile(`^\+?[1-9]\d{1,14}$`),
        digitsOnlyPattern:    regexp.MustCompile(`^\d{10,15}$`),
    }
}

func (pv *PhoneValidator) ValidateUS(phone string) bool {
    return pv.usPattern.MatchString(phone)
}

func (pv *PhoneValidator) ValidateInternational(phone string) bool {
    return pv.internationalPattern.MatchString(phone)
}

func (pv *PhoneValidator) NormalizeUS(phone string) string {
    return pv.usPattern.ReplaceAllString(phone, "($1) $2-$3")
}

func main() {
    validator := NewPhoneValidator()
    
    testPhones := []string{
        "555-123-4567",
        "(555) 123-4567",
        "555.123.4567",
        "5551234567",
        "+1 555 123 4567",
        "+44 20 7946 0958",
        "123-45-6789",
        "invalid-phone",
    }
    
    fmt.Println("Phone validation results:")
    fmt.Printf("%-20s %-6s %-6s %-15s\n", "Phone", "US", "Intl", "Normalized")
    fmt.Println(strings.Repeat("-", 50))
    
    for _, phone := range testPhones {
        us := validator.ValidateUS(phone)
        intl := validator.ValidateInternational(phone)
        normalized := ""
        if us {
            normalized = validator.NormalizeUS(phone)
        }
        fmt.Printf("%-20s %-6t %-6t %-15s\n", phone, us, intl, normalized)
    }
}
```

Phone validation handles multiple formats and provides normalization for  
consistent storage and display. Different patterns support regional  
requirements and international standards.  

## Case-insensitive matching

Go's regex engine doesn't support inline case-insensitive flags, but provides  
alternative approaches for case-insensitive matching.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func main() {
    text := "The QUICK brown Fox jumps OVER the Lazy DOG"
    
    // Method 1: Using (?i) flag at start of pattern
    caseInsensitivePattern := regexp.MustCompile(`(?i)fox`)
    match1 := caseInsensitivePattern.FindString(text)
    fmt.Printf("Case-insensitive match: '%s'\n", match1)
    
    // Method 2: Converting text to lowercase
    lowercaseText := strings.ToLower(text)
    normalPattern := regexp.MustCompile(`fox`)
    match2 := normalPattern.FindString(lowercaseText)
    fmt.Printf("Lowercase match: '%s'\n", match2)
    
    // Method 3: Character class approach
    manualCasePattern := regexp.MustCompile(`[Ff][Oo][Xx]`)
    match3 := manualCasePattern.FindString(text)
    fmt.Printf("Manual case match: '%s'\n", match3)
    
    // Finding all case variations
    wordPattern := regexp.MustCompile(`(?i)\b(the|and|or|fox|dog)\b`)
    allMatches := wordPattern.FindAllString(text, -1)
    fmt.Printf("All case-insensitive matches: %v\n", allMatches)
    
    // Case-insensitive replacement
    replacedText := wordPattern.ReplaceAllStringFunc(text, func(match string) string {
        return "[" + strings.ToUpper(match) + "]"
    })
    fmt.Printf("Replaced text: %s\n", replacedText)
}
```

Case-insensitive matching uses the `(?i)` flag or preprocessing with string  
transformations. The approach chosen depends on performance requirements  
and pattern complexity.  

## Data extraction from structured text

Regular expressions excel at extracting structured data from formatted text  
like logs, CSV-like formats, or configuration files.  

```go
package main

import (
    "fmt"
    "regexp"
    "strconv"
    "time"
)

type LogEntry struct {
    Timestamp time.Time
    Level     string
    Message   string
    Source    string
}

func parseLogEntry(line string) (*LogEntry, error) {
    // Pattern for log format: [2024-03-15 14:30:25] LEVEL [source] message
    pattern := regexp.MustCompile(`^\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] (\w+) \[([^\]]+)\] (.+)$`)
    
    matches := pattern.FindStringSubmatch(line)
    if len(matches) != 5 {
        return nil, fmt.Errorf("invalid log format")
    }
    
    timestamp, err := time.Parse("2006-01-02 15:04:05", matches[1])
    if err != nil {
        return nil, fmt.Errorf("invalid timestamp: %v", err)
    }
    
    return &LogEntry{
        Timestamp: timestamp,
        Level:     matches[2],
        Source:    matches[3],
        Message:   matches[4],
    }, nil
}

func extractMetrics(text string) map[string]float64 {
    // Pattern for metrics like "cpu_usage=75.5%" or "memory=1.2GB"
    metricPattern := regexp.MustCompile(`(\w+)=(\d+\.?\d*)([%\w]*)\b`)
    
    metrics := make(map[string]float64)
    matches := metricPattern.FindAllStringSubmatch(text, -1)
    
    for _, match := range matches {
        if len(match) == 4 {
            value, err := strconv.ParseFloat(match[2], 64)
            if err == nil {
                key := match[1]
                unit := match[3]
                if unit == "%" {
                    value = value / 100 // Convert percentage to decimal
                }
                metrics[key] = value
            }
        }
    }
    
    return metrics
}

func main() {
    logLines := []string{
        "[2024-03-15 14:30:25] INFO [auth] User login successful",
        "[2024-03-15 14:30:26] ERROR [db] Connection timeout",
        "[2024-03-15 14:30:27] WARNING [api] Rate limit exceeded",
        "Invalid log format line",
    }
    
    fmt.Println("Parsed log entries:")
    for _, line := range logLines {
        entry, err := parseLogEntry(line)
        if err != nil {
            fmt.Printf("Error parsing '%s': %v\n", line, err)
            continue
        }
        fmt.Printf("  %s [%s/%s]: %s\n", 
            entry.Timestamp.Format("15:04:05"), 
            entry.Level, entry.Source, entry.Message)
    }
    
    metricsText := "System status: cpu_usage=75.5% memory=8.2GB disk_free=45.7% network_speed=1000Mbps"
    metrics := extractMetrics(metricsText)
    
    fmt.Println("\nExtracted metrics:")
    for key, value := range metrics {
        fmt.Printf("  %s: %.2f\n", key, value)
    }
}
```

Structured data extraction combines capture groups with data parsing to  
convert text into structured objects. This approach works well for logs,  
configuration files, and other formatted text sources.  

## HTML tag extraction

Extracting content from HTML requires careful pattern design to handle  
various tag formats and nested structures.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type HTMLExtractor struct {
    tagPattern       *regexp.Regexp
    linkPattern      *regexp.Regexp
    attributePattern *regexp.Regexp
    textOnlyPattern  *regexp.Regexp
}

func NewHTMLExtractor() *HTMLExtractor {
    return &HTMLExtractor{
        tagPattern:       regexp.MustCompile(`<([a-zA-Z][a-zA-Z0-9]*)[^>]*>([^<]*)</\1>`),
        linkPattern:      regexp.MustCompile(`<a[^>]+href=["']([^"']+)["'][^>]*>([^<]*)</a>`),
        attributePattern: regexp.MustCompile(`(\w+)=["']([^"']*)["']`),
        textOnlyPattern:  regexp.MustCompile(`<[^>]*>`),
    }
}

func (he *HTMLExtractor) ExtractTags(html string) map[string][]string {
    tags := make(map[string][]string)
    matches := he.tagPattern.FindAllStringSubmatch(html, -1)
    
    for _, match := range matches {
        if len(match) == 3 {
            tagName := strings.ToLower(match[1])
            content := strings.TrimSpace(match[2])
            tags[tagName] = append(tags[tagName], content)
        }
    }
    
    return tags
}

func (he *HTMLExtractor) ExtractLinks(html string) []map[string]string {
    var links []map[string]string
    matches := he.linkPattern.FindAllStringSubmatch(html, -1)
    
    for _, match := range matches {
        if len(match) == 3 {
            links = append(links, map[string]string{
                "url":  match[1],
                "text": strings.TrimSpace(match[2]),
            })
        }
    }
    
    return links
}

func (he *HTMLExtractor) ExtractAttributes(tag string) map[string]string {
    attributes := make(map[string]string)
    matches := he.attributePattern.FindAllStringSubmatch(tag, -1)
    
    for _, match := range matches {
        if len(match) == 3 {
            attributes[match[1]] = match[2]
        }
    }
    
    return attributes
}

func (he *HTMLExtractor) StripTags(html string) string {
    return strings.TrimSpace(he.textOnlyPattern.ReplaceAllString(html, " "))
}

func main() {
    extractor := NewHTMLExtractor()
    
    html := `<html>
        <head><title>Sample Page</title></head>
        <body>
            <h1>Welcome</h1>
            <p>This is a <em>sample</em> paragraph.</p>
            <a href="https://example.com">Visit Example</a>
            <a href="/internal">Internal Link</a>
            <div class="content" id="main">Main content here</div>
        </body>
    </html>`
    
    tags := extractor.ExtractTags(html)
    fmt.Println("Extracted tags:")
    for tagName, contents := range tags {
        fmt.Printf("  %s: %v\n", tagName, contents)
    }
    
    links := extractor.ExtractLinks(html)
    fmt.Println("\nExtracted links:")
    for i, link := range links {
        fmt.Printf("  %d: '%s' -> %s\n", i+1, link["text"], link["url"])
    }
    
    // Extract attributes from a specific tag
    divTag := `<div class="content highlight" id="main" data-value="123">`
    attributes := extractor.ExtractAttributes(divTag)
    fmt.Println("\nDiv attributes:")
    for attr, value := range attributes {
        fmt.Printf("  %s='%s'\n", attr, value)
    }
    
    plainText := extractor.StripTags(html)
    fmt.Printf("\nPlain text: %s\n", plainText)
}
```

HTML extraction handles common web scraping tasks while acknowledging that  
regex has limitations with complex HTML. For production HTML parsing,  
dedicated HTML parsers are recommended.  

## IP address validation

IP address validation covers both IPv4 and IPv6 formats with various  
representation styles.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type IPValidator struct {
    ipv4Pattern     *regexp.Regexp
    ipv6Pattern     *regexp.Regexp
    privateV4Pattern *regexp.Regexp
}

func NewIPValidator() *IPValidator {
    return &IPValidator{
        ipv4Pattern:     regexp.MustCompile(`^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$`),
        ipv6Pattern:     regexp.MustCompile(`^(?:[0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$|^::1$|^::$`),
        privateV4Pattern: regexp.MustCompile(`^(?:10\.|172\.(?:1[6-9]|2[0-9]|3[01])\.|192\.168\.)`),
    }
}

func (iv *IPValidator) IsValidIPv4(ip string) bool {
    return iv.ipv4Pattern.MatchString(ip)
}

func (iv *IPValidator) IsValidIPv6(ip string) bool {
    // Simple IPv6 pattern - real implementation would be more complex
    return iv.ipv6Pattern.MatchString(ip) || strings.Contains(ip, "::")
}

func (iv *IPValidator) IsPrivateIPv4(ip string) bool {
    return iv.IsValidIPv4(ip) && iv.privateV4Pattern.MatchString(ip)
}

func extractIPsFromText(text string) []string {
    // Pattern to find IP-like strings in text
    ipPattern := regexp.MustCompile(`\b(?:\d{1,3}\.){3}\d{1,3}\b`)
    return ipPattern.FindAllString(text, -1)
}

func main() {
    validator := NewIPValidator()
    
    testIPs := []string{
        "192.168.1.1",
        "10.0.0.1",
        "172.16.0.1",
        "8.8.8.8",
        "256.1.1.1",
        "192.168.1",
        "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
        "::1",
        "::",
        "invalid.ip",
    }
    
    fmt.Println("IP validation results:")
    fmt.Printf("%-40s %-8s %-8s %-8s\n", "IP Address", "IPv4", "IPv6", "Private")
    fmt.Println(strings.Repeat("-", 66))
    
    for _, ip := range testIPs {
        ipv4 := validator.IsValidIPv4(ip)
        ipv6 := validator.IsValidIPv6(ip)
        private := validator.IsPrivateIPv4(ip)
        fmt.Printf("%-40s %-8t %-8t %-8t\n", ip, ipv4, ipv6, private)
    }
    
    // Extract IPs from log-like text
    logText := `Connection from 192.168.1.100 to 8.8.8.8 on port 80
    Failed login from 10.0.0.5 at 14:30:25
    Invalid connection attempt from 256.300.1.1`
    
    foundIPs := extractIPsFromText(logText)
    fmt.Println("\nIPs found in text:")
    for _, ip := range foundIPs {
        valid := validator.IsValidIPv4(ip)
        fmt.Printf("  %s (valid: %t)\n", ip, valid)
    }
}
```

IP validation patterns ensure network addresses meet format requirements.  
The patterns distinguish between public and private address ranges for  
security and routing decisions.  

## Password strength validation

Password validation uses multiple patterns to enforce complexity requirements  
and security policies.  

```go
package main

import (
    "fmt"
    "regexp"
)

type PasswordValidator struct {
    lengthPattern      *regexp.Regexp
    uppercasePattern   *regexp.Regexp
    lowercasePattern   *regexp.Regexp
    digitPattern       *regexp.Regexp
    specialPattern     *regexp.Regexp
    commonWordsPattern *regexp.Regexp
    sequencePattern    *regexp.Regexp
}

func NewPasswordValidator() *PasswordValidator {
    return &PasswordValidator{
        lengthPattern:      regexp.MustCompile(`^.{8,}$`),
        uppercasePattern:   regexp.MustCompile(`[A-Z]`),
        lowercasePattern:   regexp.MustCompile(`[a-z]`),
        digitPattern:       regexp.MustCompile(`\d`),
        specialPattern:     regexp.MustCompile(`[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]`),
        commonWordsPattern: regexp.MustCompile(`(?i)(password|123456|qwerty|admin|letmein)`),
        sequencePattern:    regexp.MustCompile(`(.)\1{2,}|abc|123|321|cba`),
    }
}

type PasswordStrength struct {
    Score    int
    Issues   []string
    Strength string
}

func (pv *PasswordValidator) ValidatePassword(password string) PasswordStrength {
    var issues []string
    score := 0
    
    if !pv.lengthPattern.MatchString(password) {
        issues = append(issues, "Must be at least 8 characters long")
    } else {
        score++
    }
    
    if !pv.uppercasePattern.MatchString(password) {
        issues = append(issues, "Must contain at least one uppercase letter")
    } else {
        score++
    }
    
    if !pv.lowercasePattern.MatchString(password) {
        issues = append(issues, "Must contain at least one lowercase letter")
    } else {
        score++
    }
    
    if !pv.digitPattern.MatchString(password) {
        issues = append(issues, "Must contain at least one digit")
    } else {
        score++
    }
    
    if !pv.specialPattern.MatchString(password) {
        issues = append(issues, "Must contain at least one special character")
    } else {
        score++
    }
    
    if pv.commonWordsPattern.MatchString(password) {
        issues = append(issues, "Contains common words or patterns")
        score--
    }
    
    if pv.sequencePattern.MatchString(password) {
        issues = append(issues, "Contains repeating characters or sequences")
        score--
    }
    
    // Bonus points for length
    if len(password) >= 12 {
        score++
    }
    if len(password) >= 16 {
        score++
    }
    
    strength := "Very Weak"
    if score >= 2 {
        strength = "Weak"
    }
    if score >= 4 {
        strength = "Medium"
    }
    if score >= 6 {
        strength = "Strong"
    }
    if score >= 7 {
        strength = "Very Strong"
    }
    
    return PasswordStrength{
        Score:    score,
        Issues:   issues,
        Strength: strength,
    }
}

func main() {
    validator := NewPasswordValidator()
    
    testPasswords := []string{
        "password",
        "Password1",
        "MyP@ssw0rd",
        "SuperSecure123!@#",
        "aaaaaaaaa",
        "Tr0ub4dor&3",
        "correcthorsebatterystaple",
        "C0mpl3x!P@ssw0rd2024",
    }
    
    fmt.Println("Password strength validation:")
    fmt.Printf("%-25s %-12s %-6s %s\n", "Password", "Strength", "Score", "Issues")
    fmt.Println(strings.Repeat("-", 80))
    
    for _, password := range testPasswords {
        result := validator.ValidatePassword(password)
        issuesStr := ""
        if len(result.Issues) > 0 {
            issuesStr = result.Issues[0] // Show first issue
        }
        fmt.Printf("%-25s %-12s %-6d %s\n", password, result.Strength, result.Score, issuesStr)
    }
    
    // Detailed analysis of one password
    fmt.Println("\nDetailed analysis of 'MyP@ssw0rd':")
    result := validator.ValidatePassword("MyP@ssw0rd")
    fmt.Printf("Strength: %s (Score: %d)\n", result.Strength, result.Score)
    if len(result.Issues) > 0 {
        fmt.Println("Issues:")
        for _, issue := range result.Issues {
            fmt.Printf("  - %s\n", issue)
        }
    } else {
        fmt.Println("No issues found!")
    }
}
```

Password validation patterns enforce security policies by checking multiple  
requirements simultaneously. The scoring system provides user feedback on  
password strength improvement opportunities.  

## Credit card number validation

Credit card validation checks format patterns and uses the Luhn algorithm  
for mathematical validation.  

```go
package main

import (
    "fmt"
    "regexp"
    "strconv"
    "strings"
)

type CardValidator struct {
    visaPattern       *regexp.Regexp
    mastercardPattern *regexp.Regexp
    amexPattern       *regexp.Regexp
    discoverPattern   *regexp.Regexp
    cleanPattern      *regexp.Regexp
}

func NewCardValidator() *CardValidator {
    return &CardValidator{
        visaPattern:       regexp.MustCompile(`^4[0-9]{15}$`),
        mastercardPattern: regexp.MustCompile(`^5[1-5][0-9]{14}$`),
        amexPattern:       regexp.MustCompile(`^3[47][0-9]{13}$`),
        discoverPattern:   regexp.MustCompile(`^6(?:011|5[0-9]{2})[0-9]{12}$`),
        cleanPattern:      regexp.MustCompile(`[^\d]`),
    }
}

func (cv *CardValidator) CleanCardNumber(cardNumber string) string {
    return cv.cleanPattern.ReplaceAllString(cardNumber, "")
}

func (cv *CardValidator) GetCardType(cardNumber string) string {
    clean := cv.CleanCardNumber(cardNumber)
    
    if cv.visaPattern.MatchString(clean) {
        return "Visa"
    }
    if cv.mastercardPattern.MatchString(clean) {
        return "Mastercard"
    }
    if cv.amexPattern.MatchString(clean) {
        return "American Express"
    }
    if cv.discoverPattern.MatchString(clean) {
        return "Discover"
    }
    
    return "Unknown"
}

func (cv *CardValidator) ValidateWithLuhn(cardNumber string) bool {
    clean := cv.CleanCardNumber(cardNumber)
    
    if len(clean) < 13 || len(clean) > 19 {
        return false
    }
    
    // Luhn algorithm implementation
    sum := 0
    alternate := false
    
    // Start from rightmost digit
    for i := len(clean) - 1; i >= 0; i-- {
        digit, err := strconv.Atoi(string(clean[i]))
        if err != nil {
            return false
        }
        
        if alternate {
            digit *= 2
            if digit > 9 {
                digit = digit%10 + digit/10
            }
        }
        
        sum += digit
        alternate = !alternate
    }
    
    return sum%10 == 0
}

func (cv *CardValidator) FormatCardNumber(cardNumber string) string {
    clean := cv.CleanCardNumber(cardNumber)
    cardType := cv.GetCardType(clean)
    
    switch cardType {
    case "American Express":
        // Format as XXXX-XXXXXX-XXXXX
        if len(clean) == 15 {
            return fmt.Sprintf("%s-%s-%s", clean[:4], clean[4:10], clean[10:])
        }
    default:
        // Format as XXXX-XXXX-XXXX-XXXX
        if len(clean) >= 16 {
            return fmt.Sprintf("%s-%s-%s-%s", clean[:4], clean[4:8], clean[8:12], clean[12:16])
        }
    }
    
    return clean
}

func main() {
    validator := NewCardValidator()
    
    testCards := []string{
        "4532 1234 5678 9012",    // Valid Visa
        "5555-5555-5555-4444",    // Valid Mastercard
        "378282246310005",        // Valid Amex
        "6011111111111117",       // Valid Discover
        "4532123456789013",       // Invalid (fails Luhn)
        "1234-5678-9012-3456",    // Invalid format
    }
    
    fmt.Println("Credit card validation:")
    fmt.Printf("%-20s %-15s %-8s %-20s\n", "Card Number", "Type", "Valid", "Formatted")
    fmt.Println(strings.Repeat("-", 65))
    
    for _, card := range testCards {
        cardType := validator.GetCardType(card)
        valid := validator.ValidateWithLuhn(card)
        formatted := validator.FormatCardNumber(card)
        
        fmt.Printf("%-20s %-15s %-8t %-20s\n", card, cardType, valid, formatted)
    }
    
    // Demonstrate extraction from text
    text := "Please charge my card 4532-1234-5678-9012 or backup card 5555 5555 5555 4444"
    cardPattern := regexp.MustCompile(`\b(?:\d{4}[-\s]?){3,4}\d{4}\b`)
    foundCards := cardPattern.FindAllString(text, -1)
    
    fmt.Println("\nCards found in text:")
    for _, card := range foundCards {
        cardType := validator.GetCardType(card)
        valid := validator.ValidateWithLuhn(card)
        fmt.Printf("  %s - %s (%t)\n", card, cardType, valid)
    }
}
```

Credit card validation combines pattern matching with mathematical validation  
using the Luhn algorithm. This dual approach catches both format errors  
and number-based mistakes.  

## CSV parsing with regex

Regular expressions can handle simple CSV parsing, though dedicated CSV  
libraries are recommended for complex cases.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type CSVParser struct {
    simplePattern   *regexp.Regexp
    quotedPattern   *regexp.Regexp
    headerPattern   *regexp.Regexp
}

func NewCSVParser() *CSVParser {
    return &CSVParser{
        simplePattern: regexp.MustCompile(`[^,\n\r]+`),
        quotedPattern: regexp.MustCompile(`"([^"]*)"|([^,\n\r]*)`),
        headerPattern: regexp.MustCompile(`^([^,\n\r]+(?:,[^,\n\r]+)*)`),
    }
}

func (cp *CSVParser) ParseSimpleCSV(line string) []string {
    line = strings.TrimSpace(line)
    if line == "" {
        return nil
    }
    
    return strings.Split(line, ",")
}

func (cp *CSVParser) ParseQuotedCSV(line string) []string {
    var fields []string
    matches := cp.quotedPattern.FindAllStringSubmatch(line, -1)
    
    for _, match := range matches {
        if match[1] != "" {
            // Quoted field
            fields = append(fields, match[1])
        } else if match[2] != "" {
            // Unquoted field
            fields = append(fields, strings.TrimSpace(match[2]))
        }
    }
    
    return fields
}

func (cp *CSVParser) ExtractColumns(data string, columnNames []string) map[string][]string {
    lines := strings.Split(data, "\n")
    if len(lines) < 2 {
        return nil
    }
    
    // Parse header
    header := cp.ParseSimpleCSV(lines[0])
    columnIndexes := make(map[string]int)
    for i, col := range header {
        columnIndexes[strings.TrimSpace(col)] = i
    }
    
    // Extract requested columns
    result := make(map[string][]string)
    for _, colName := range columnNames {
        result[colName] = []string{}
    }
    
    for i := 1; i < len(lines); i++ {
        if strings.TrimSpace(lines[i]) == "" {
            continue
        }
        
        fields := cp.ParseSimpleCSV(lines[i])
        
        for _, colName := range columnNames {
            if index, exists := columnIndexes[colName]; exists && index < len(fields) {
                result[colName] = append(result[colName], strings.TrimSpace(fields[index]))
            }
        }
    }
    
    return result
}

func parseKeyValuePairs(text string) map[string]string {
    // Pattern for key=value pairs
    kvPattern := regexp.MustCompile(`(\w+)=([^,\s]+)`)
    pairs := make(map[string]string)
    
    matches := kvPattern.FindAllStringSubmatch(text, -1)
    for _, match := range matches {
        if len(match) == 3 {
            pairs[match[1]] = match[2]
        }
    }
    
    return pairs
}

func main() {
    parser := NewCSVParser()
    
    // Simple CSV parsing
    simpleLine := "John,25,Engineer,New York"
    simpleFields := parser.ParseSimpleCSV(simpleLine)
    fmt.Printf("Simple CSV: %v\n", simpleFields)
    
    // CSV with quoted fields
    quotedLine := `"Smith, John",25,"Software Engineer","New York, NY"`
    quotedFields := parser.ParseQuotedCSV(quotedLine)
    fmt.Printf("Quoted CSV: %v\n", quotedFields)
    
    // Multi-line CSV data
    csvData := `Name,Age,Job,City
John Doe,30,Developer,Seattle
Jane Smith,28,Designer,Portland
Bob Johnson,35,Manager,San Francisco
Alice Brown,32,Analyst,Denver`
    
    columns := parser.ExtractColumns(csvData, []string{"Name", "Job", "City"})
    fmt.Println("\nExtracted columns:")
    for colName, values := range columns {
        fmt.Printf("%s: %v\n", colName, values)
    }
    
    // Key-value pair parsing
    kvText := "user=john, role=admin, timeout=300, debug=true"
    pairs := parseKeyValuePairs(kvText)
    fmt.Println("\nKey-value pairs:")
    for key, value := range pairs {
        fmt.Printf("  %s = %s\n", key, value)
    }
    
    // Extract data with patterns
    logLine := "2024-03-15T10:30:25 user=alice action=login ip=192.168.1.100 status=success"
    logKV := parseKeyValuePairs(logLine)
    timestampPattern := regexp.MustCompile(`\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}`)
    timestamp := timestampPattern.FindString(logLine)
    
    fmt.Printf("\nLog analysis:\n")
    fmt.Printf("  Timestamp: %s\n", timestamp)
    for key, value := range logKV {
        fmt.Printf("  %s: %s\n", key, value)
    }
}
```

CSV parsing with regex handles simple cases effectively. The approach scales  
from basic comma-separated values to quoted fields and key-value pair  
extraction from structured text.  

## Configuration file parsing

Regular expressions excel at parsing configuration files and extracting  
settings with various formats and comment styles.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type ConfigParser struct {
    sectionPattern  *regexp.Regexp
    keyValuePattern *regexp.Regexp
    commentPattern  *regexp.Regexp
    listPattern     *regexp.Regexp
}

func NewConfigParser() *ConfigParser {
    return &ConfigParser{
        sectionPattern:  regexp.MustCompile(`^\s*\[([^\]]+)\]\s*$`),
        keyValuePattern: regexp.MustCompile(`^\s*([^=\s]+)\s*=\s*(.*)$`),
        commentPattern:  regexp.MustCompile(`^\s*[#;]`),
        listPattern:     regexp.MustCompile(`[^,\s]+`),
    }
}

type Config struct {
    Sections map[string]map[string]interface{}
}

func (cp *ConfigParser) ParseConfig(configText string) *Config {
    config := &Config{
        Sections: make(map[string]map[string]interface{}),
    }
    
    lines := strings.Split(configText, "\n")
    currentSection := "default"
    config.Sections[currentSection] = make(map[string]interface{})
    
    for _, line := range lines {
        line = strings.TrimSpace(line)
        
        // Skip empty lines and comments
        if line == "" || cp.commentPattern.MatchString(line) {
            continue
        }
        
        // Check for section header
        if sectionMatch := cp.sectionPattern.FindStringSubmatch(line); sectionMatch != nil {
            currentSection = sectionMatch[1]
            config.Sections[currentSection] = make(map[string]interface{})
            continue
        }
        
        // Parse key-value pair
        if kvMatch := cp.keyValuePattern.FindStringSubmatch(line); kvMatch != nil {
            key := strings.TrimSpace(kvMatch[1])
            value := strings.TrimSpace(kvMatch[2])
            
            // Remove quotes if present
            if strings.HasPrefix(value, `"`) && strings.HasSuffix(value, `"`) {
                value = value[1 : len(value)-1]
            }
            
            config.Sections[currentSection][key] = cp.parseValue(value)
        }
    }
    
    return config
}

func (cp *ConfigParser) parseValue(value string) interface{} {
    // Try to parse as boolean
    lowerValue := strings.ToLower(value)
    if lowerValue == "true" {
        return true
    }
    if lowerValue == "false" {
        return false
    }
    
    // Try to parse as number
    if regexp.MustCompile(`^\d+$`).MatchString(value) {
        return value // Return as string for simplicity
    }
    if regexp.MustCompile(`^\d+\.\d+$`).MatchString(value) {
        return value // Return as string for simplicity
    }
    
    // Check if it's a comma-separated list
    if strings.Contains(value, ",") {
        items := cp.listPattern.FindAllString(value, -1)
        var result []string
        for _, item := range items {
            result = append(result, strings.TrimSpace(item))
        }
        return result
    }
    
    return value
}

func (cp *ConfigParser) ExtractEnvironmentVars(text string) map[string]string {
    // Pattern for environment variable assignments
    envPattern := regexp.MustCompile(`(?m)^export\s+([A-Z_][A-Z0-9_]*)\s*=\s*(.*)$`)
    vars := make(map[string]string)
    
    matches := envPattern.FindAllStringSubmatch(text, -1)
    for _, match := range matches {
        if len(match) == 3 {
            key := match[1]
            value := strings.Trim(match[2], `"'`)
            vars[key] = value
        }
    }
    
    return vars
}

func main() {
    parser := NewConfigParser()
    
    configText := `# Application Configuration
[database]
host = localhost
port = 5432
username = admin
password = "secret123"
ssl_enabled = true
max_connections = 100

[cache]
# Redis configuration
type = redis
servers = server1.example.com, server2.example.com, server3.example.com
timeout = 30
enabled = true

[logging]
level = info
file = "/var/log/app.log"
rotate = true
max_size = 10
`
    
    config := parser.ParseConfig(configText)
    
    fmt.Println("Parsed configuration:")
    for sectionName, section := range config.Sections {
        if len(section) == 0 {
            continue
        }
        fmt.Printf("\n[%s]:\n", sectionName)
        for key, value := range section {
            fmt.Printf("  %s = %v (%T)\n", key, value, value)
        }
    }
    
    // Environment variable parsing
    envText := `#!/bin/bash
export DATABASE_URL="postgres://user:pass@localhost/db"
export API_KEY=abc123xyz
export DEBUG=true
export MAX_WORKERS=4
# Comment line
export LOG_LEVEL="debug"`
    
    envVars := parser.ExtractEnvironmentVars(envText)
    fmt.Println("\nExtracted environment variables:")
    for key, value := range envVars {
        fmt.Printf("  %s=%s\n", key, value)
    }
    
    // Extract specific patterns
    urlPattern := regexp.MustCompile(`(?i)\b(https?://[^\s"']+)\b`)
    urls := urlPattern.FindAllString(envText, -1)
    fmt.Printf("\nFound URLs: %v\n", urls)
}
```

Configuration parsing handles common formats like INI files and environment  
variable assignments. The parser recognizes different data types and  
structures configuration data appropriately.  

## Log file analysis

Log analysis involves extracting timestamps, error levels, and specific  
patterns from various log formats.  

```go
package main

import (
    "fmt"
    "regexp"
    "sort"
    "strings"
    "time"
)

type LogAnalyzer struct {
    timestampPattern *regexp.Regexp
    levelPattern     *regexp.Regexp
    ipPattern        *regexp.Regexp
    errorPattern     *regexp.Regexp
    httpPattern      *regexp.Regexp
}

func NewLogAnalyzer() *LogAnalyzer {
    return &LogAnalyzer{
        timestampPattern: regexp.MustCompile(`\d{4}-\d{2}-\d{2}[T\s]\d{2}:\d{2}:\d{2}`),
        levelPattern:     regexp.MustCompile(`\b(DEBUG|INFO|WARN|ERROR|FATAL)\b`),
        ipPattern:        regexp.MustCompile(`\b(?:\d{1,3}\.){3}\d{1,3}\b`),
        errorPattern:     regexp.MustCompile(`(?i)\b(error|exception|fail|timeout|refused)\b`),
        httpPattern:      regexp.MustCompile(`"([A-Z]+)\s+([^\s]+)\s+HTTP/[\d.]+"\s+(\d{3})\s+(\d+)`),
    }
}

type LogStats struct {
    TotalLines    int
    ErrorCount    int
    LevelCounts   map[string]int
    IPs           map[string]int
    TimeRange     []string
    HTTPRequests  []HTTPRequest
}

type HTTPRequest struct {
    Method     string
    Path       string
    StatusCode string
    Size       string
    Timestamp  string
}

func (la *LogAnalyzer) AnalyzeLogs(logText string) LogStats {
    lines := strings.Split(logText, "\n")
    stats := LogStats{
        LevelCounts:  make(map[string]int),
        IPs:          make(map[string]int),
        HTTPRequests: []HTTPRequest{},
    }
    
    for _, line := range lines {
        line = strings.TrimSpace(line)
        if line == "" {
            continue
        }
        
        stats.TotalLines++
        
        // Extract timestamp
        if timestamp := la.timestampPattern.FindString(line); timestamp != "" {
            stats.TimeRange = append(stats.TimeRange, timestamp)
        }
        
        // Extract log level
        if level := la.levelPattern.FindString(line); level != "" {
            stats.LevelCounts[level]++
        }
        
        // Count errors
        if la.errorPattern.MatchString(line) {
            stats.ErrorCount++
        }
        
        // Extract IP addresses
        ips := la.ipPattern.FindAllString(line, -1)
        for _, ip := range ips {
            stats.IPs[ip]++
        }
        
        // Parse HTTP requests (Apache/Nginx format)
        if httpMatch := la.httpPattern.FindStringSubmatch(line); httpMatch != nil {
            timestamp := la.timestampPattern.FindString(line)
            req := HTTPRequest{
                Method:     httpMatch[1],
                Path:       httpMatch[2],
                StatusCode: httpMatch[3],
                Size:       httpMatch[4],
                Timestamp:  timestamp,
            }
            stats.HTTPRequests = append(stats.HTTPRequests, req)
        }
    }
    
    return stats
}

func (la *LogAnalyzer) FindPatterns(logText string, patterns map[string]string) map[string][]string {
    results := make(map[string][]string)
    
    for name, patternStr := range patterns {
        pattern := regexp.MustCompile(patternStr)
        matches := pattern.FindAllString(logText, -1)
        results[name] = matches
    }
    
    return results
}

func (la *LogAnalyzer) ExtractErrors(logText string) []string {
    lines := strings.Split(logText, "\n")
    var errors []string
    
    for _, line := range lines {
        if la.levelPattern.FindString(line) == "ERROR" || la.errorPattern.MatchString(line) {
            errors = append(errors, line)
        }
    }
    
    return errors
}

func main() {
    analyzer := NewLogAnalyzer()
    
    logData := `2024-03-15T10:30:25 INFO [auth] User login successful from 192.168.1.100
2024-03-15T10:30:26 ERROR [db] Connection timeout to 10.0.0.5
2024-03-15T10:30:27 WARN [api] Rate limit exceeded for 192.168.1.200
192.168.1.100 - - [15/Mar/2024:10:30:28 +0000] "GET /api/users HTTP/1.1" 200 1234
192.168.1.200 - - [15/Mar/2024:10:30:29 +0000] "POST /api/login HTTP/1.1" 429 567
2024-03-15T10:30:30 ERROR [system] Disk space low exception
192.168.1.100 - - [15/Mar/2024:10:30:31 +0000] "GET /dashboard HTTP/1.1" 200 2048
2024-03-15T10:30:32 INFO [cache] Cache miss for key user:123
2024-03-15T10:30:33 FATAL [system] Service crash detected`
    
    stats := analyzer.AnalyzeLogs(logData)
    
    fmt.Println("Log Analysis Results:")
    fmt.Printf("Total lines: %d\n", stats.TotalLines)
    fmt.Printf("Error count: %d\n\n", stats.ErrorCount)
    
    fmt.Println("Log levels:")
    for level, count := range stats.LevelCounts {
        fmt.Printf("  %s: %d\n", level, count)
    }
    
    fmt.Println("\nTop IP addresses:")
    type ipCount struct {
        IP    string
        Count int
    }
    var ipCounts []ipCount
    for ip, count := range stats.IPs {
        ipCounts = append(ipCounts, ipCount{ip, count})
    }
    sort.Slice(ipCounts, func(i, j int) bool {
        return ipCounts[i].Count > ipCounts[j].Count
    })
    for _, ic := range ipCounts[:min(3, len(ipCounts))] {
        fmt.Printf("  %s: %d requests\n", ic.IP, ic.Count)
    }
    
    fmt.Println("\nHTTP requests:")
    for _, req := range stats.HTTPRequests {
        fmt.Printf("  %s %s -> %s (%s bytes)\n", req.Method, req.Path, req.StatusCode, req.Size)
    }
    
    // Custom pattern search
    customPatterns := map[string]string{
        "user_ids":    `user:(\d+)`,
        "api_paths":   `/(api/[^\s"]+)`,
        "status_codes": `HTTP/[\d.]+"\s+(\d{3})`,
    }
    
    patterns := analyzer.FindPatterns(logData, customPatterns)
    fmt.Println("\nCustom patterns found:")
    for pattern, matches := range patterns {
        if len(matches) > 0 {
            fmt.Printf("  %s: %v\n", pattern, matches)
        }
    }
    
    // Extract error lines
    errors := analyzer.ExtractErrors(logData)
    fmt.Println("\nError lines:")
    for _, error := range errors {
        fmt.Printf("  %s\n", error)
    }
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

Log analysis uses multiple patterns to extract structured information from  
unstructured log files. The analyzer provides insights into system behavior,  
errors, and usage patterns.  

## Data validation pipeline

Complex data validation combines multiple regex patterns into a validation  
pipeline with detailed error reporting.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type ValidationRule struct {
    Name        string
    Pattern     *regexp.Regexp
    Required    bool
    ErrorMsg    string
    Transform   func(string) string
}

type ValidationResult struct {
    Field   string
    Valid   bool
    Value   string
    Errors  []string
    Applied []string
}

type DataValidator struct {
    rules map[string][]*ValidationRule
}

func NewDataValidator() *DataValidator {
    return &DataValidator{
        rules: make(map[string][]*ValidationRule),
    }
}

func (dv *DataValidator) AddRule(field string, rule *ValidationRule) {
    dv.rules[field] = append(dv.rules[field], rule)
}

func (dv *DataValidator) ValidateField(field, value string) ValidationResult {
    result := ValidationResult{
        Field:   field,
        Value:   value,
        Valid:   true,
        Errors:  []string{},
        Applied: []string{},
    }
    
    rules, exists := dv.rules[field]
    if !exists {
        return result
    }
    
    originalValue := value
    
    for _, rule := range rules {
        // Apply transformation if provided
        if rule.Transform != nil {
            transformedValue := rule.Transform(value)
            if transformedValue != value {
                result.Applied = append(result.Applied, fmt.Sprintf("%s: transformed", rule.Name))
                value = transformedValue
            }
        }
        
        // Check if field is required
        if rule.Required && strings.TrimSpace(value) == "" {
            result.Valid = false
            result.Errors = append(result.Errors, fmt.Sprintf("%s: field is required", rule.Name))
            continue
        }
        
        // Skip pattern check for empty non-required fields
        if !rule.Required && strings.TrimSpace(value) == "" {
            continue
        }
        
        // Validate pattern
        if !rule.Pattern.MatchString(value) {
            result.Valid = false
            result.Errors = append(result.Errors, rule.ErrorMsg)
        } else {
            result.Applied = append(result.Applied, fmt.Sprintf("%s: passed", rule.Name))
        }
    }
    
    result.Value = value
    if result.Value != originalValue {
        result.Applied = append(result.Applied, "value transformed")
    }
    
    return result
}

func (dv *DataValidator) ValidateRecord(record map[string]string) map[string]ValidationResult {
    results := make(map[string]ValidationResult)
    
    // Validate existing fields
    for field, value := range record {
        results[field] = dv.ValidateField(field, value)
    }
    
    // Check for missing required fields
    for field, rules := range dv.rules {
        if _, exists := record[field]; !exists {
            for _, rule := range rules {
                if rule.Required {
                    results[field] = ValidationResult{
                        Field:  field,
                        Valid:  false,
                        Value:  "",
                        Errors: []string{fmt.Sprintf("%s: field is required but missing", rule.Name)},
                    }
                    break
                }
            }
        }
    }
    
    return results
}

func setupUserValidator() *DataValidator {
    dv := NewDataValidator()
    
    // Username validation
    dv.AddRule("username", &ValidationRule{
        Name:     "length",
        Pattern:  regexp.MustCompile(`^.{3,20}$`),
        Required: true,
        ErrorMsg: "Username must be 3-20 characters long",
    })
    dv.AddRule("username", &ValidationRule{
        Name:     "format",
        Pattern:  regexp.MustCompile(`^[a-zA-Z0-9_-]+$`),
        Required: true,
        ErrorMsg: "Username can only contain letters, numbers, underscore, and dash",
    })
    
    // Email validation
    dv.AddRule("email", &ValidationRule{
        Name:     "format",
        Pattern:  regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        Required: true,
        ErrorMsg: "Invalid email format",
        Transform: strings.ToLower,
    })
    
    // Phone number validation with normalization
    dv.AddRule("phone", &ValidationRule{
        Name:      "format",
        Pattern:   regexp.MustCompile(`^\(\d{3}\) \d{3}-\d{4}$`),
        Required:  false,
        ErrorMsg:  "Phone must be in format (XXX) XXX-XXXX",
        Transform: func(s string) string {
            // Remove non-digits and format
            digits := regexp.MustCompile(`\D`).ReplaceAllString(s, "")
            if len(digits) == 10 {
                return fmt.Sprintf("(%s) %s-%s", digits[:3], digits[3:6], digits[6:])
            }
            return s
        },
    })
    
    // Age validation
    dv.AddRule("age", &ValidationRule{
        Name:     "range",
        Pattern:  regexp.MustCompile(`^(?:[1-9]|[1-9][0-9]|1[01][0-9]|120)$`),
        Required: false,
        ErrorMsg: "Age must be between 1 and 120",
    })
    
    return dv
}

func main() {
    validator := setupUserValidator()
    
    testUsers := []map[string]string{
        {
            "username": "john_doe123",
            "email":    "JOHN.DOE@EXAMPLE.COM",
            "phone":    "555-123-4567",
            "age":      "30",
        },
        {
            "username": "u2",
            "email":    "invalid-email",
            "phone":    "555.123.4567",
            "age":      "150",
        },
        {
            "username": "valid_user",
            "email":    "user@domain.com",
        },
        {
            "email": "missing_username@test.com",
            "age":   "25",
        },
    }
    
    for i, user := range testUsers {
        fmt.Printf("\n=== User %d Validation ===\n", i+1)
        results := validator.ValidateRecord(user)
        
        allValid := true
        for field, result := range results {
            fmt.Printf("\nField: %s\n", field)
            fmt.Printf("  Original: '%s'\n", user[field])
            fmt.Printf("  Final: '%s'\n", result.Value)
            fmt.Printf("  Valid: %t\n", result.Valid)
            
            if len(result.Applied) > 0 {
                fmt.Printf("  Applied: %s\n", strings.Join(result.Applied, ", "))
            }
            
            if len(result.Errors) > 0 {
                fmt.Printf("  Errors: %s\n", strings.Join(result.Errors, ", "))
                allValid = false
            }
        }
        
        fmt.Printf("\nOverall: %s\n", map[bool]string{true: "✓ VALID", false: "✗ INVALID"}[allValid])
    }
    
    // Demonstrate field-level validation
    fmt.Println("\n=== Field-Level Validation ===")
    testEmails := []string{
        "VALID@EXAMPLE.COM",
        "invalid.email",
        "user@domain",
        "",
    }
    
    for _, email := range testEmails {
        result := validator.ValidateField("email", email)
        fmt.Printf("Email '%s' -> '%s' (valid: %t)\n", email, result.Value, result.Valid)
        if len(result.Errors) > 0 {
            fmt.Printf("  Errors: %s\n", strings.Join(result.Errors, ", "))
        }
    }
}
```

Validation pipelines combine multiple patterns and transformations to create  
comprehensive data validation systems. The approach handles field  
requirements, format validation, and data normalization simultaneously.  

## SQL injection detection

Regular expressions can help identify potential SQL injection patterns in  
user input, though they should not be the only security measure.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type SQLSecurityScanner struct {
    sqlKeywordPattern     *regexp.Regexp
    unionAttackPattern    *regexp.Regexp
    commentPattern        *regexp.Regexp
    stringBreakPattern    *regexp.Regexp
    numericAttackPattern  *regexp.Regexp
    timingAttackPattern   *regexp.Regexp
    blindAttackPattern    *regexp.Regexp
}

func NewSQLSecurityScanner() *SQLSecurityScanner {
    return &SQLSecurityScanner{
        sqlKeywordPattern:    regexp.MustCompile(`(?i)\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION|OR|AND|WHERE)\b`),
        unionAttackPattern:   regexp.MustCompile(`(?i)\bUNION\s+(ALL\s+)?SELECT\b`),
        commentPattern:       regexp.MustCompile(`(--|\s*/\*.*?\*/|#)`),
        stringBreakPattern:   regexp.MustCompile(`['"]\s*;\s*|['"]\s*\|\|\s*|['"]\s*\+\s*`),
        numericAttackPattern: regexp.MustCompile(`\b\d+\s*(=|>|<|!=)\s*\d+(\s+(OR|AND)\s+\d+\s*(=|>|<|!=)\s*\d+)*\b`),
        timingAttackPattern:  regexp.MustCompile(`(?i)\b(SLEEP|BENCHMARK|WAITFOR\s+DELAY)\s*\(`),
        blindAttackPattern:   regexp.MustCompile(`(?i)\b(ASCII|SUBSTRING|LENGTH|MID|CHAR)\s*\(`),
    }
}

type SecurityThreat struct {
    Type        string
    Severity    string
    Pattern     string
    Description string
}

func (sss *SQLSecurityScanner) ScanInput(input string) []SecurityThreat {
    var threats []SecurityThreat
    
    input = strings.TrimSpace(input)
    if input == "" {
        return threats
    }
    
    // Check for SQL keywords in suspicious context
    if sss.sqlKeywordPattern.MatchString(input) {
        keywords := sss.sqlKeywordPattern.FindAllString(input, -1)
        threats = append(threats, SecurityThreat{
            Type:        "SQL_KEYWORDS",
            Severity:    "Medium",
            Pattern:     strings.Join(keywords, ", "),
            Description: "Contains SQL keywords that could indicate injection attempts",
        })
    }
    
    // Check for UNION attacks
    if sss.unionAttackPattern.MatchString(input) {
        match := sss.unionAttackPattern.FindString(input)
        threats = append(threats, SecurityThreat{
            Type:        "UNION_ATTACK",
            Severity:    "High",
            Pattern:     match,
            Description: "UNION SELECT pattern detected - common SQL injection technique",
        })
    }
    
    // Check for comment patterns
    if sss.commentPattern.MatchString(input) {
        matches := sss.commentPattern.FindAllString(input, -1)
        threats = append(threats, SecurityThreat{
            Type:        "SQL_COMMENTS",
            Severity:    "Medium",
            Pattern:     strings.Join(matches, ", "),
            Description: "SQL comment patterns that could be used to bypass filters",
        })
    }
    
    // Check for string concatenation/breaking
    if sss.stringBreakPattern.MatchString(input) {
        match := sss.stringBreakPattern.FindString(input)
        threats = append(threats, SecurityThreat{
            Type:        "STRING_BREAK",
            Severity:    "High",
            Pattern:     match,
            Description: "String breaking patterns that could escape SQL strings",
        })
    }
    
    // Check for numeric comparison attacks
    if sss.numericAttackPattern.MatchString(input) {
        match := sss.numericAttackPattern.FindString(input)
        threats = append(threats, SecurityThreat{
            Type:        "NUMERIC_ATTACK",
            Severity:    "Medium",
            Pattern:     match,
            Description: "Numeric comparison patterns often used in blind SQL injection",
        })
    }
    
    // Check for timing attacks
    if sss.timingAttackPattern.MatchString(input) {
        match := sss.timingAttackPattern.FindString(input)
        threats = append(threats, SecurityThreat{
            Type:        "TIMING_ATTACK",
            Severity:    "High",
            Pattern:     match,
            Description: "Time-based attack functions detected",
        })
    }
    
    // Check for blind injection functions
    if sss.blindAttackPattern.MatchString(input) {
        matches := sss.blindAttackPattern.FindAllString(input, -1)
        threats = append(threats, SecurityThreat{
            Type:        "BLIND_INJECTION",
            Severity:    "High",
            Pattern:     strings.Join(matches, ", "),
            Description: "Functions commonly used in blind SQL injection attacks",
        })
    }
    
    return threats
}

func (sss *SQLSecurityScanner) SanitizeInput(input string) string {
    // Basic sanitization - escape single quotes
    sanitized := strings.ReplaceAll(input, "'", "''")
    
    // Remove SQL comments
    sanitized = sss.commentPattern.ReplaceAllString(sanitized, "")
    
    return strings.TrimSpace(sanitized)
}

func main() {
    scanner := NewSQLSecurityScanner()
    
    testInputs := []string{
        "john.doe@email.com",
        "1' OR '1'='1",
        "admin'; DROP TABLE users; --",
        "1 UNION SELECT username, password FROM users",
        "test' /*comment*/ OR 1=1 --",
        "user123",
        "'; WAITFOR DELAY '00:00:05' --",
        "1=1 AND ASCII(SUBSTRING(password,1,1))>64",
        "normal search term",
        "product' + (SELECT COUNT(*) FROM users) + '",
    }
    
    fmt.Println("SQL Injection Security Scan Results:")
    fmt.Println(strings.Repeat("=", 60))
    
    for i, input := range testInputs {
        fmt.Printf("\n%d. Input: %s\n", i+1, input)
        
        threats := scanner.ScanInput(input)
        
        if len(threats) == 0 {
            fmt.Println("   ✅ No threats detected")
        } else {
            fmt.Printf("   ⚠️  %d threat(s) detected:\n", len(threats))
            for _, threat := range threats {
                fmt.Printf("      Type: %s (Severity: %s)\n", threat.Type, threat.Severity)
                fmt.Printf("      Pattern: %s\n", threat.Pattern)
                fmt.Printf("      Description: %s\n", threat.Description)
                fmt.Println()
            }
        }
        
        sanitized := scanner.SanitizeInput(input)
        if sanitized != input {
            fmt.Printf("   🧼 Sanitized: %s\n", sanitized)
        }
    }
    
    // Risk assessment
    fmt.Println("\n" + strings.Repeat("=", 60))
    fmt.Println("RISK ASSESSMENT SUMMARY:")
    
    riskCounts := map[string]int{"High": 0, "Medium": 0, "Low": 0}
    totalThreats := 0
    
    for _, input := range testInputs {
        threats := scanner.ScanInput(input)
        totalThreats += len(threats)
        for _, threat := range threats {
            riskCounts[threat.Severity]++
        }
    }
    
    fmt.Printf("Total inputs scanned: %d\n", len(testInputs))
    fmt.Printf("Total threats found: %d\n", totalThreats)
    fmt.Printf("High severity: %d\n", riskCounts["High"])
    fmt.Printf("Medium severity: %d\n", riskCounts["Medium"])
    fmt.Printf("Low severity: %d\n", riskCounts["Low"])
}
```

SQL injection detection patterns identify common attack vectors in user input.  
While regex provides initial screening, comprehensive security requires  
parameterized queries and input validation at multiple layers.  

## Performance optimization

Regular expression performance can be optimized through compilation reuse,  
pattern design, and efficient string processing techniques.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "time"
)

type RegexPerformanceTest struct {
    compiledPattern   *regexp.Regexp
    efficientPattern  *regexp.Regexp
    inefficientPattern *regexp.Regexp
}

func NewRegexPerformanceTest() *RegexPerformanceTest {
    return &RegexPerformanceTest{
        compiledPattern:    regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`),
        efficientPattern:   regexp.MustCompile(`^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`),
        inefficientPattern: regexp.MustCompile(`.*@.*\..*`), // Too broad
    }
}

func (rpt *RegexPerformanceTest) TestCompilationVsRecompilation(text string, iterations int) {
    fmt.Println("=== Compilation vs Recompilation Test ===")
    
    // Test with recompilation each time (inefficient)
    start := time.Now()
    for i := 0; i < iterations; i++ {
        pattern := regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`)
        pattern.FindAllString(text, -1)
    }
    recompileTime := time.Since(start)
    
    // Test with pre-compiled pattern (efficient)
    start = time.Now()
    for i := 0; i < iterations; i++ {
        rpt.compiledPattern.FindAllString(text, -1)
    }
    precompiledTime := time.Since(start)
    
    fmt.Printf("Recompilation time: %v\n", recompileTime)
    fmt.Printf("Pre-compiled time: %v\n", precompiledTime)
    fmt.Printf("Performance improvement: %.2fx faster\n", 
        float64(recompileTime.Nanoseconds())/float64(precompiledTime.Nanoseconds()))
}

func (rpt *RegexPerformanceTest) TestPatternEfficiency(text string, iterations int) {
    fmt.Println("\n=== Pattern Efficiency Test ===")
    
    // Test efficient pattern
    start := time.Now()
    for i := 0; i < iterations; i++ {
        rpt.efficientPattern.MatchString("user@example.com")
    }
    efficientTime := time.Since(start)
    
    // Test inefficient pattern
    start = time.Now()
    for i := 0; i < iterations; i++ {
        rpt.inefficientPattern.MatchString("user@example.com")
    }
    inefficientTime := time.Since(start)
    
    fmt.Printf("Efficient pattern time: %v\n", efficientTime)
    fmt.Printf("Inefficient pattern time: %v\n", inefficientTime)
    
    if inefficientTime > efficientTime {
        fmt.Printf("Efficient pattern is %.2fx faster\n", 
            float64(inefficientTime.Nanoseconds())/float64(efficientTime.Nanoseconds()))
    }
}

func (rpt *RegexPerformanceTest) TestStringPreprocessing(texts []string) {
    fmt.Println("\n=== String Preprocessing Test ===")
    
    emailPattern := regexp.MustCompile(`[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`)
    
    // Method 1: Direct regex on each string
    start := time.Now()
    var results1 []string
    for _, text := range texts {
        matches := emailPattern.FindAllString(text, -1)
        results1 = append(results1, matches...)
    }
    directTime := time.Since(start)
    
    // Method 2: Prefilter then regex
    start = time.Now()
    var results2 []string
    for _, text := range texts {
        // Quick check before expensive regex
        if strings.Contains(text, "@") && strings.Contains(text, ".") {
            matches := emailPattern.FindAllString(text, -1)
            results2 = append(results2, matches...)
        }
    }
    prefilterTime := time.Since(start)
    
    fmt.Printf("Direct regex time: %v (found %d emails)\n", directTime, len(results1))
    fmt.Printf("Prefilter time: %v (found %d emails)\n", prefilterTime, len(results2))
    
    if directTime > prefilterTime {
        fmt.Printf("Prefiltering is %.2fx faster\n", 
            float64(directTime.Nanoseconds())/float64(prefilterTime.Nanoseconds()))
    }
}

func benchmarkRegexMethods(text string) {
    fmt.Println("\n=== Regex Method Benchmarks ===")
    
    pattern := regexp.MustCompile(`\b\w+\b`)
    iterations := 10000
    
    // MatchString benchmark
    start := time.Now()
    for i := 0; i < iterations; i++ {
        pattern.MatchString(text)
    }
    matchTime := time.Since(start)
    
    // FindString benchmark
    start = time.Now()
    for i := 0; i < iterations; i++ {
        pattern.FindString(text)
    }
    findTime := time.Since(start)
    
    // FindAllString benchmark
    start = time.Now()
    for i := 0; i < iterations; i++ {
        pattern.FindAllString(text, -1)
    }
    findAllTime := time.Since(start)
    
    // Split benchmark
    start = time.Now()
    for i := 0; i < iterations; i++ {
        pattern.Split(text, -1)
    }
    splitTime := time.Since(start)
    
    fmt.Printf("MatchString:   %v\n", matchTime)
    fmt.Printf("FindString:    %v\n", findTime)
    fmt.Printf("FindAllString: %v\n", findAllTime)
    fmt.Printf("Split:         %v\n", splitTime)
}

func optimizedTextProcessing(text string) map[string]int {
    // Optimized approach: single pass with multiple patterns
    results := make(map[string]int)
    
    patterns := map[string]*regexp.Regexp{
        "emails":  regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`),
        "phones":  regexp.MustCompile(`\b\d{3}[-.]?\d{3}[-.]?\d{4}\b`),
        "urls":    regexp.MustCompile(`https?://[^\s]+`),
        "numbers": regexp.MustCompile(`\b\d+\b`),
    }
    
    // Single pass through text
    for name, pattern := range patterns {
        matches := pattern.FindAllString(text, -1)
        results[name] = len(matches)
    }
    
    return results
}

func main() {
    tester := NewRegexPerformanceTest()
    
    // Sample text for testing
    sampleText := `Contact us at support@example.com or sales@company.org
    Call 555-123-4567 or visit https://example.com
    Also try admin@test.com or check https://backup.site.net
    Phone numbers: 555.987.6543, 123-456-7890
    More emails: user@domain.co.uk, test@subdomain.example.org`
    
    // Create larger dataset for testing
    largeText := strings.Repeat(sampleText+" ", 1000)
    
    // Run performance tests
    tester.TestCompilationVsRecompilation(largeText, 100)
    tester.TestPatternEfficiency(sampleText, 10000)
    
    // Create test dataset without @ symbols for prefilter test
    testTexts := []string{
        sampleText,
        "No emails in this text",
        "Just some regular text without contact info",
        sampleText,
        "Another email: contact@business.com",
    }
    
    tester.TestStringPreprocessing(testTexts)
    
    benchmarkRegexMethods(sampleText)
    
    // Demonstrate optimized processing
    fmt.Println("\n=== Optimized Text Processing ===")
    start := time.Now()
    results := optimizedTextProcessing(largeText)
    processingTime := time.Since(start)
    
    fmt.Printf("Processing time: %v\n", processingTime)
    fmt.Println("Results found:")
    for category, count := range results {
        fmt.Printf("  %s: %d\n", category, count)
    }
    
    // Memory usage tips
    fmt.Println("\n=== Performance Tips ===")
    fmt.Println("1. Pre-compile regex patterns and reuse them")
    fmt.Println("2. Use anchors (^$) when checking entire strings")
    fmt.Println("3. Avoid overly broad patterns that match too much")
    fmt.Println("4. Use string preprocessing to filter before regex")
    fmt.Println("5. Process multiple patterns in single pass when possible")
    fmt.Println("6. Consider string methods for simple cases")
    fmt.Println("7. Use appropriate method: MatchString vs FindString vs FindAll")
}
```

Performance optimization focuses on pattern compilation, efficient pattern  
design, and strategic preprocessing. Understanding when and how to apply  
different regex methods significantly impacts application performance.  

## Unicode and international text

Go's regex engine provides excellent Unicode support for international text  
processing and multi-language content handling.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
    "unicode"
)

type UnicodeProcessor struct {
    emojiPattern      *regexp.Regexp
    accentPattern     *regexp.Regexp
    cyrillicPattern   *regexp.Regexp
    cjkPattern        *regexp.Regexp
    arabicPattern     *regexp.Regexp
    numberPattern     *regexp.Regexp
    wordPattern       *regexp.Regexp
}

func NewUnicodeProcessor() *UnicodeProcessor {
    return &UnicodeProcessor{
        // Emoji pattern (simplified)
        emojiPattern: regexp.MustCompile(`[\x{1F600}-\x{1F64F}]|[\x{1F300}-\x{1F5FF}]|[\x{1F680}-\x{1F6FF}]|[\x{2600}-\x{26FF}]|[\x{2700}-\x{27BF}]`),
        
        // Accented characters
        accentPattern: regexp.MustCompile(`[àáâãäåæçèéêëìíîïðñòóôõöøùúûüýþÿ]`),
        
        // Cyrillic script
        cyrillicPattern: regexp.MustCompile(`[\x{0400}-\x{04FF}]+`),
        
        // CJK (Chinese, Japanese, Korean)
        cjkPattern: regexp.MustCompile(`[\x{4E00}-\x{9FFF}\x{3400}-\x{4DBF}\x{3040}-\x{309F}\x{30A0}-\x{30FF}]+`),
        
        // Arabic script
        arabicPattern: regexp.MustCompile(`[\x{0600}-\x{06FF}\x{0750}-\x{077F}]+`),
        
        // Unicode numbers from any script
        numberPattern: regexp.MustCompile(`\pN+`),
        
        // Unicode word characters
        wordPattern: regexp.MustCompile(`\pL+`),
    }
}

func (up *UnicodeProcessor) AnalyzeText(text string) map[string]interface{} {
    analysis := make(map[string]interface{})
    
    // Count different script types
    analysis["emojis"] = len(up.emojiPattern.FindAllString(text, -1))
    analysis["accented_chars"] = len(up.accentPattern.FindAllString(text, -1))
    analysis["cyrillic_words"] = len(up.cyrillicPattern.FindAllString(text, -1))
    analysis["cjk_sequences"] = len(up.cjkPattern.FindAllString(text, -1))
    analysis["arabic_words"] = len(up.arabicPattern.FindAllString(text, -1))
    analysis["unicode_numbers"] = up.numberPattern.FindAllString(text, -1)
    analysis["words"] = up.wordPattern.FindAllString(text, -1)
    
    // Character analysis
    var scripts []string
    runes := []rune(text)
    analysis["total_runes"] = len(runes)
    analysis["byte_length"] = len(text)
    
    scriptMap := make(map[string]int)
    for _, r := range runes {
        if unicode.In(r, unicode.Latin) {
            scriptMap["Latin"]++
        } else if unicode.In(r, unicode.Cyrillic) {
            scriptMap["Cyrillic"]++
        } else if unicode.In(r, unicode.Han) {
            scriptMap["Han"]++
        } else if unicode.In(r, unicode.Arabic) {
            scriptMap["Arabic"]++
        } else if unicode.In(r, unicode.Hiragana, unicode.Katakana) {
            scriptMap["Japanese"]++
        } else if unicode.IsSpace(r) {
            scriptMap["Space"]++
        } else if unicode.IsPunct(r) {
            scriptMap["Punctuation"]++
        } else {
            scriptMap["Other"]++
        }
    }
    
    analysis["script_distribution"] = scriptMap
    return analysis
}

func (up *UnicodeProcessor) ExtractByLanguage(text string) map[string][]string {
    results := make(map[string][]string)
    
    // Extract different language content
    results["emojis"] = up.emojiPattern.FindAllString(text, -1)
    results["cyrillic"] = up.cyrillicPattern.FindAllString(text, -1)
    results["cjk"] = up.cjkPattern.FindAllString(text, -1)
    results["arabic"] = up.arabicPattern.FindAllString(text, -1)
    
    // Extract URLs with international domains
    internationalDomainPattern := regexp.MustCompile(`https?://[^\s]*\.[\pL]{2,}`)
    results["international_urls"] = internationalDomainPattern.FindAllString(text, -1)
    
    return results
}

func (up *UnicodeProcessor) NormalizeText(text string) string {
    // Simple normalization examples
    
    // Remove or replace emojis
    normalized := up.emojiPattern.ReplaceAllString(text, " [EMOJI] ")
    
    // Normalize whitespace (including Unicode spaces)
    wsPattern := regexp.MustCompile(`\s+`)
    normalized = wsPattern.ReplaceAllString(normalized, " ")
    
    return strings.TrimSpace(normalized)
}

func validateInternationalEmail(email string) bool {
    // International email pattern supporting Unicode domains
    pattern := regexp.MustCompile(`^[\pL\pN._%+-]+@[\pL\pN.-]+\.[\pL]{2,}$`)
    return pattern.MatchString(email)
}

func extractInternationalPhones(text string) []string {
    // Pattern for international phone numbers
    phonePattern := regexp.MustCompile(`\+[\pN\s\-\(\)]{7,20}`)
    return phonePattern.FindAllString(text, -1)
}

func main() {
    processor := NewUnicodeProcessor()
    
    // Test with multilingual text
    multilingualText := `Hello World! 🌍 こんにちは世界！
    Привет мир! مرحبا بالعالم! Hola mundo! 
    Contact: user@тест.рф or alice@例え.テスト
    Phone: +33 1 23 45 67 89 or +81-3-1234-5678
    Chinese: 这是中文测试 Japanese: これは日本語のテスト
    Arabic: هذا اختبار عربي Russian: Это русский тест
    Numbers: ১২৩ (Bengali) ១២៣ (Khmer) ๑๒๓ (Thai)
    Emojis: 😀🎉🚀❤️🌟`
    
    fmt.Println("=== Unicode Text Analysis ===")
    analysis := processor.AnalyzeText(multilingualText)
    
    fmt.Printf("Total runes: %d\n", analysis["total_runes"])
    fmt.Printf("Byte length: %d\n", analysis["byte_length"])
    fmt.Printf("Emojis found: %d\n", analysis["emojis"])
    fmt.Printf("Accented characters: %d\n", analysis["accented_chars"])
    
    fmt.Println("\nScript distribution:")
    scriptDist := analysis["script_distribution"].(map[string]int)
    for script, count := range scriptDist {
        if count > 0 {
            fmt.Printf("  %s: %d\n", script, count)
        }
    }
    
    fmt.Println("\nUnicode numbers found:")
    numbers := analysis["unicode_numbers"].([]string)
    for _, num := range numbers {
        fmt.Printf("  %s\n", num)
    }
    
    fmt.Println("\n=== Language Extraction ===")
    byLanguage := processor.ExtractByLanguage(multilingualText)
    for lang, items := range byLanguage {
        if len(items) > 0 {
            fmt.Printf("%s: %v\n", lang, items)
        }
    }
    
    fmt.Println("\n=== International Email Validation ===")
    testEmails := []string{
        "user@example.com",
        "用户@例え.テスト",
        "пользователь@тест.рф",
        "משתמש@דוגמא.ישראל",
        "user@مثال.السعودية",
    }
    
    for _, email := range testEmails {
        valid := validateInternationalEmail(email)
        fmt.Printf("  %-25s: %t\n", email, valid)
    }
    
    fmt.Println("\n=== International Phone Extraction ===")
    phoneText := `Contact numbers: +1-555-123-4567, +33 1 23 45 67 89, 
    +81 3-1234-5678, +86 138 0013 8000, +7 495 123-45-67`
    
    phones := extractInternationalPhones(phoneText)
    for i, phone := range phones {
        fmt.Printf("  %d: %s\n", i+1, phone)
    }
    
    fmt.Println("\n=== Text Normalization ===")
    normalized := processor.NormalizeText(multilingualText)
    fmt.Printf("Original length: %d bytes\n", len(multilingualText))
    fmt.Printf("Normalized length: %d bytes\n", len(normalized))
    fmt.Printf("First 100 chars: %.100s...\n", normalized)
    
    // Demonstrate right-to-left text handling
    fmt.Println("\n=== RTL Text Handling ===")
    rtlText := "Hello مرحبا World עולם!"
    fmt.Printf("Mixed RTL/LTR text: %s\n", rtlText)
    
    arabicWords := processor.arabicPattern.FindAllString(rtlText, -1)
    fmt.Printf("Arabic words: %v\n", arabicWords)
    
    // Hebrew pattern (as example of another RTL script)
    hebrewPattern := regexp.MustCompile(`[\x{0590}-\x{05FF}]+`)
    hebrewWords := hebrewPattern.FindAllString(rtlText, -1)
    fmt.Printf("Hebrew words: %v\n", hebrewWords)
}
```

Unicode processing handles international text, multiple scripts, and  
character encoding complexities. Go's regex engine provides robust  
support for worldwide content and multilingual applications.  

## Advanced text processing pipeline

Complex text processing combines multiple regex operations into sophisticated  
content analysis and transformation pipelines.  

```go
package main

import (
    "fmt"
    "regexp"
    "sort"
    "strings"
)

type TextProcessor struct {
    sentencePattern    *regexp.Regexp
    wordPattern        *regexp.Regexp
    numberPattern      *regexp.Regexp
    punctuationPattern *regexp.Regexp
    capitalPattern     *regexp.Regexp
    acronymPattern     *regexp.Regexp
    quotePattern       *regexp.Regexp
}

func NewTextProcessor() *TextProcessor {
    return &TextProcessor{
        sentencePattern:    regexp.MustCompile(`[.!?]+\s*`),
        wordPattern:        regexp.MustCompile(`\b[a-zA-Z]+\b`),
        numberPattern:      regexp.MustCompile(`\b\d+(?:\.\d+)?\b`),
        punctuationPattern: regexp.MustCompile(`[^\w\s]`),
        capitalPattern:     regexp.MustCompile(`\b[A-Z][a-z]*\b`),
        acronymPattern:     regexp.MustCompile(`\b[A-Z]{2,}\b`),
        quotePattern:       regexp.MustCompile(`["']([^"']*?)["']`),
    }
}

type TextAnalysis struct {
    WordCount       int
    SentenceCount   int
    ParagraphCount  int
    AverageWordsPerSentence float64
    LongestWord     string
    MostCommonWords []WordFreq
    Numbers         []string
    Quotes          []string
    Acronyms        []string
    Readability     string
}

type WordFreq struct {
    Word  string
    Count int
}

func (tp *TextProcessor) AnalyzeText(text string) TextAnalysis {
    analysis := TextAnalysis{}
    
    // Count paragraphs
    paragraphs := strings.Split(strings.TrimSpace(text), "\n\n")
    analysis.ParagraphCount = len(paragraphs)
    
    // Extract and count sentences
    sentences := tp.sentencePattern.Split(text, -1)
    validSentences := 0
    for _, sentence := range sentences {
        if strings.TrimSpace(sentence) != "" {
            validSentences++
        }
    }
    analysis.SentenceCount = validSentences
    
    // Extract and analyze words
    words := tp.wordPattern.FindAllString(text, -1)
    analysis.WordCount = len(words)
    
    // Calculate average words per sentence
    if analysis.SentenceCount > 0 {
        analysis.AverageWordsPerSentence = float64(analysis.WordCount) / float64(analysis.SentenceCount)
    }
    
    // Find longest word
    longestWord := ""
    wordFreq := make(map[string]int)
    for _, word := range words {
        lowerWord := strings.ToLower(word)
        wordFreq[lowerWord]++
        if len(word) > len(longestWord) {
            longestWord = word
        }
    }
    analysis.LongestWord = longestWord
    
    // Get most common words (top 5)
    var freqList []WordFreq
    for word, count := range wordFreq {
        freqList = append(freqList, WordFreq{word, count})
    }
    sort.Slice(freqList, func(i, j int) bool {
        return freqList[i].Count > freqList[j].Count
    })
    if len(freqList) > 5 {
        analysis.MostCommonWords = freqList[:5]
    } else {
        analysis.MostCommonWords = freqList
    }
    
    // Extract numbers
    analysis.Numbers = tp.numberPattern.FindAllString(text, -1)
    
    // Extract quoted text
    quoteMatches := tp.quotePattern.FindAllStringSubmatch(text, -1)
    for _, match := range quoteMatches {
        if len(match) > 1 {
            analysis.Quotes = append(analysis.Quotes, match[1])
        }
    }
    
    // Extract acronyms
    analysis.Acronyms = tp.acronymPattern.FindAllString(text, -1)
    
    // Simple readability assessment
    if analysis.AverageWordsPerSentence > 20 {
        analysis.Readability = "Complex"
    } else if analysis.AverageWordsPerSentence > 15 {
        analysis.Readability = "Moderate"
    } else {
        analysis.Readability = "Simple"
    }
    
    return analysis
}

func (tp *TextProcessor) ExtractEntities(text string) map[string][]string {
    entities := make(map[string][]string)
    
    // Email addresses
    emailPattern := regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`)
    entities["emails"] = emailPattern.FindAllString(text, -1)
    
    // Phone numbers
    phonePattern := regexp.MustCompile(`\b\d{3}[-.]?\d{3}[-.]?\d{4}\b`)
    entities["phones"] = phonePattern.FindAllString(text, -1)
    
    // URLs
    urlPattern := regexp.MustCompile(`https?://[^\s]+`)
    entities["urls"] = urlPattern.FindAllString(text, -1)
    
    // Dates (simple format)
    datePattern := regexp.MustCompile(`\b\d{1,2}/\d{1,2}/\d{4}\b|\b\d{4}-\d{2}-\d{2}\b`)
    entities["dates"] = datePattern.FindAllString(text, -1)
    
    // Money amounts
    moneyPattern := regexp.MustCompile(`\$\d+(?:,\d{3})*(?:\.\d{2})?`)
    entities["money"] = moneyPattern.FindAllString(text, -1)
    
    // Time expressions
    timePattern := regexp.MustCompile(`\b\d{1,2}:\d{2}(?:\s?[AP]M)?\b`)
    entities["times"] = timePattern.FindAllString(text, -1)
    
    return entities
}

func (tp *TextProcessor) TransformText(text string, transformations map[string]func(string) string) string {
    result := text
    
    for name, transform := range transformations {
        switch name {
        case "remove_extra_spaces":
            wsPattern := regexp.MustCompile(`\s+`)
            result = wsPattern.ReplaceAllString(result, " ")
            
        case "normalize_quotes":
            smartQuotePattern := regexp.MustCompile(`[""]`)
            result = smartQuotePattern.ReplaceAllString(result, `"`)
            
        case "expand_contractions":
            contractions := map[string]string{
                `\bcan't\b`:    "cannot",
                `\bwon't\b`:    "will not",
                `\bI'm\b`:      "I am",
                `\byou're\b`:   "you are",
                `\bit's\b`:     "it is",
                `\bthey're\b`:  "they are",
                `\bwe're\b`:    "we are",
                `\bdon't\b`:    "do not",
                `\bdoesn't\b`:  "does not",
                `\bdidn't\b`:   "did not",
                `\bwouldn't\b`: "would not",
                `\bshouldn't\b`: "should not",
                `\bcouldn't\b`: "could not",
            }
            for pattern, replacement := range contractions {
                contractPattern := regexp.MustCompile(`(?i)` + pattern)
                result = contractPattern.ReplaceAllString(result, replacement)
            }
            
        case "remove_numbers":
            result = tp.numberPattern.ReplaceAllString(result, "")
            
        case "mask_emails":
            emailPattern := regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`)
            result = emailPattern.ReplaceAllString(result, "[EMAIL]")
        }
    }
    
    return strings.TrimSpace(result)
}

func (tp *TextProcessor) GenerateSummary(text string, maxSentences int) string {
    sentences := tp.sentencePattern.Split(text, -1)
    
    // Score sentences based on word count and position
    type scoredSentence struct {
        text  string
        score int
        index int
    }
    
    var scored []scoredSentence
    for i, sentence := range sentences {
        sentence = strings.TrimSpace(sentence)
        if sentence == "" {
            continue
        }
        
        wordCount := len(tp.wordPattern.FindAllString(sentence, -1))
        
        // Simple scoring: prefer sentences with more words, earlier sentences
        score := wordCount
        if i < len(sentences)/3 { // First third gets bonus
            score += 5
        }
        
        scored = append(scored, scoredSentence{sentence, score, i})
    }
    
    // Sort by score (descending) then by original position
    sort.Slice(scored, func(i, j int) bool {
        if scored[i].score == scored[j].score {
            return scored[i].index < scored[j].index
        }
        return scored[i].score > scored[j].score
    })
    
    // Take top sentences up to maxSentences
    limit := maxSentences
    if len(scored) < limit {
        limit = len(scored)
    }
    
    // Re-sort selected sentences by original position
    selected := scored[:limit]
    sort.Slice(selected, func(i, j int) bool {
        return selected[i].index < selected[j].index
    })
    
    var summary []string
    for _, s := range selected {
        summary = append(summary, s.text)
    }
    
    return strings.Join(summary, ". ") + "."
}

func main() {
    processor := NewTextProcessor()
    
    sampleText := `The quick brown fox jumps over the lazy dog. This sentence contains every letter of the alphabet at least once, making it a perfect pangram for testing purposes. 

    "Regular expressions are powerful," said the developer. The project deadline is 12/15/2024, and the budget is $50,000.00. Contact the team at team@example.com or call 555-123-4567 for more information.

    NASA announced new discoveries today. The meeting is scheduled for 2:30 PM. You can't miss this opportunity! It's crucial for the project's success. We're committed to delivering quality results.

    Visit https://example.com for updates. The temperature reached 98.6 degrees today. Don't forget to review the documentation!`
    
    fmt.Println("=== Text Analysis Results ===")
    analysis := processor.AnalyzeText(sampleText)
    
    fmt.Printf("Word count: %d\n", analysis.WordCount)
    fmt.Printf("Sentence count: %d\n", analysis.SentenceCount)
    fmt.Printf("Paragraph count: %d\n", analysis.ParagraphCount)
    fmt.Printf("Average words per sentence: %.1f\n", analysis.AverageWordsPerSentence)
    fmt.Printf("Longest word: %s (%d letters)\n", analysis.LongestWord, len(analysis.LongestWord))
    fmt.Printf("Readability: %s\n", analysis.Readability)
    
    fmt.Println("\nMost common words:")
    for i, word := range analysis.MostCommonWords {
        fmt.Printf("  %d. %s (%d times)\n", i+1, word.Word, word.Count)
    }
    
    fmt.Printf("\nNumbers found: %v\n", analysis.Numbers)
    fmt.Printf("Quotes: %v\n", analysis.Quotes)
    fmt.Printf("Acronyms: %v\n", analysis.Acronyms)
    
    fmt.Println("\n=== Entity Extraction ===")
    entities := processor.ExtractEntities(sampleText)
    for entityType, items := range entities {
        if len(items) > 0 {
            fmt.Printf("%s: %v\n", entityType, items)
        }
    }
    
    fmt.Println("\n=== Text Transformations ===")
    transformations := map[string]func(string) string{
        "remove_extra_spaces":   nil,
        "normalize_quotes":      nil,
        "expand_contractions":   nil,
        "mask_emails":          nil,
    }
    
    transformed := processor.TransformText(sampleText, transformations)
    fmt.Printf("Original length: %d characters\n", len(sampleText))
    fmt.Printf("Transformed length: %d characters\n", len(transformed))
    fmt.Printf("First 150 chars: %.150s...\n", transformed)
    
    fmt.Println("\n=== Auto-Generated Summary ===")
    summary := processor.GenerateSummary(sampleText, 3)
    fmt.Printf("Summary (3 sentences): %s\n", summary)
    
    fmt.Println("\n=== Processing Pipeline Demo ===")
    pipeline := []string{
        "1. Extract entities",
        "2. Analyze readability", 
        "3. Generate word frequency",
        "4. Apply transformations",
        "5. Create summary",
    }
    
    for _, step := range pipeline {
        fmt.Printf("  %s ✓\n", step)
    }
    
    fmt.Printf("\nProcessing complete! Input: %d words → Output: %d words in summary\n", 
        analysis.WordCount, len(processor.wordPattern.FindAllString(summary, -1)))
}
```

Advanced text processing pipelines combine multiple regex operations to  
perform comprehensive content analysis, entity extraction, and intelligent  
text transformation for natural language processing applications.  

## Security and sanitization

Regex patterns play a crucial role in input sanitization and security  
hardening by detecting and neutralizing potentially dangerous content.  

```go
package main

import (
    "fmt"
    "html"
    "regexp"
    "strings"
)

type SecuritySanitizer struct {
    scriptTagPattern    *regexp.Regexp
    onEventPattern      *regexp.Regexp
    urlSchemePattern    *regexp.Regexp
    htmlEntityPattern   *regexp.Regexp
    sqlCommentPattern   *regexp.Regexp
    pathTraversalPattern *regexp.Regexp
    commandInjectionPattern *regexp.Regexp
    xssVectorPattern    *regexp.Regexp
}

func NewSecuritySanitizer() *SecuritySanitizer {
    return &SecuritySanitizer{
        scriptTagPattern:     regexp.MustCompile(`(?i)<script[^>]*>.*?</script>`),
        onEventPattern:       regexp.MustCompile(`(?i)\bon\w+\s*=`),
        urlSchemePattern:     regexp.MustCompile(`(?i)^(javascript|data|vbscript):`),
        htmlEntityPattern:    regexp.MustCompile(`&#?\w+;`),
        sqlCommentPattern:    regexp.MustCompile(`(--|\/\*|\*\/|#)`),
        pathTraversalPattern: regexp.MustCompile(`\.\.[\\/]`),
        commandInjectionPattern: regexp.MustCompile(`[;&|`+"`"+`$(){}[\]<>*?~!]`),
        xssVectorPattern:     regexp.MustCompile(`(?i)(alert|confirm|prompt|eval|settimeout|setinterval)\s*\(`),
    }
}

type SecurityThreat struct {
    Type        string
    Severity    string
    Pattern     string
    Description string
    Position    []int
}

func (ss *SecuritySanitizer) ScanForThreats(input string) []SecurityThreat {
    var threats []SecurityThreat
    
    // Check for script tags
    if matches := ss.scriptTagPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "XSS_SCRIPT_TAG",
                Severity:    "HIGH",
                Pattern:     input[match[0]:match[1]],
                Description: "Script tag detected - potential XSS attack",
                Position:    match,
            })
        }
    }
    
    // Check for event handlers
    if matches := ss.onEventPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "XSS_EVENT_HANDLER",
                Severity:    "HIGH",
                Pattern:     input[match[0]:match[1]],
                Description: "Event handler attribute detected",
                Position:    match,
            })
        }
    }
    
    // Check for dangerous URL schemes
    if ss.urlSchemePattern.MatchString(input) {
        match := ss.urlSchemePattern.FindStringIndex(input)
        threats = append(threats, SecurityThreat{
            Type:        "DANGEROUS_URL_SCHEME",
            Severity:    "HIGH",
            Pattern:     input[match[0]:match[1]],
            Description: "Dangerous URL scheme detected",
            Position:    match,
        })
    }
    
    // Check for SQL injection comments
    if matches := ss.sqlCommentPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "SQL_COMMENT",
                Severity:    "MEDIUM",
                Pattern:     input[match[0]:match[1]],
                Description: "SQL comment pattern detected",
                Position:    match,
            })
        }
    }
    
    // Check for path traversal
    if matches := ss.pathTraversalPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "PATH_TRAVERSAL",
                Severity:    "HIGH",
                Pattern:     input[match[0]:match[1]],
                Description: "Path traversal pattern detected",
                Position:    match,
            })
        }
    }
    
    // Check for command injection
    if matches := ss.commandInjectionPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "COMMAND_INJECTION",
                Severity:    "HIGH",
                Pattern:     input[match[0]:match[1]],
                Description: "Command injection character detected",
                Position:    match,
            })
        }
    }
    
    // Check for XSS vectors
    if matches := ss.xssVectorPattern.FindAllStringIndex(input, -1); matches != nil {
        for _, match := range matches {
            threats = append(threats, SecurityThreat{
                Type:        "XSS_VECTOR",
                Severity:    "HIGH",
                Pattern:     input[match[0]:match[1]],
                Description: "XSS vector function detected",
                Position:    match,
            })
        }
    }
    
    return threats
}

func (ss *SecuritySanitizer) SanitizeHTML(input string) string {
    sanitized := input
    
    // Remove script tags
    sanitized = ss.scriptTagPattern.ReplaceAllString(sanitized, "")
    
    // Remove event handlers
    sanitized = ss.onEventPattern.ReplaceAllString(sanitized, "")
    
    // Escape HTML entities
    sanitized = html.EscapeString(sanitized)
    
    // Remove dangerous URL schemes
    sanitized = ss.urlSchemePattern.ReplaceAllStringFunc(sanitized, func(match string) string {
        return "[BLOCKED_URL]"
    })
    
    return sanitized
}

func (ss *SecuritySanitizer) SanitizeSQLInput(input string) string {
    sanitized := input
    
    // Remove SQL comments
    sanitized = ss.sqlCommentPattern.ReplaceAllString(sanitized, "")
    
    // Escape single quotes
    sanitized = strings.ReplaceAll(sanitized, "'", "''")
    
    // Remove semicolons at the end
    sanitized = regexp.MustCompile(`;+$`).ReplaceAllString(sanitized, "")
    
    return strings.TrimSpace(sanitized)
}

func (ss *SecuritySanitizer) SanitizeFilePath(input string) string {
    sanitized := input
    
    // Remove path traversal patterns
    sanitized = ss.pathTraversalPattern.ReplaceAllString(sanitized, "")
    
    // Remove dangerous characters
    dangerousChars := regexp.MustCompile(`[<>:"|?*]`)
    sanitized = dangerousChars.ReplaceAllString(sanitized, "_")
    
    // Limit length
    if len(sanitized) > 255 {
        sanitized = sanitized[:255]
    }
    
    return strings.TrimSpace(sanitized)
}

func (ss *SecuritySanitizer) SanitizeShellCommand(input string) string {
    sanitized := input
    
    // Remove command injection characters
    sanitized = ss.commandInjectionPattern.ReplaceAllString(sanitized, "")
    
    // Only allow alphanumeric, spaces, hyphens, and underscores
    allowedChars := regexp.MustCompile(`[^a-zA-Z0-9\s\-_]`)
    sanitized = allowedChars.ReplaceAllString(sanitized, "")
    
    // Remove multiple spaces
    multiSpace := regexp.MustCompile(`\s+`)
    sanitized = multiSpace.ReplaceAllString(sanitized, " ")
    
    return strings.TrimSpace(sanitized)
}

func validateWhitelistPattern(input, pattern string) bool {
    whitelist := regexp.MustCompile("^" + pattern + "$")
    return whitelist.MatchString(input)
}

func main() {
    sanitizer := NewSecuritySanitizer()
    
    // Test inputs with various security threats
    testInputs := []string{
        `<script>alert('XSS')</script>`,
        `<img src="x" onerror="alert('XSS')">`,
        `javascript:alert('XSS')`,
        `'; DROP TABLE users; --`,
        `../../../etc/passwd`,
        `; rm -rf / #`,
        `<iframe src="data:text/html,<script>alert('XSS')</script>">`,
        `eval('malicious code')`,
        `normal text input`,
        `user@example.com`,
    }
    
    fmt.Println("=== Security Threat Analysis ===")
    
    totalThreats := 0
    for i, input := range testInputs {
        fmt.Printf("\n%d. Input: %s\n", i+1, input)
        
        threats := sanitizer.ScanForThreats(input)
        if len(threats) == 0 {
            fmt.Println("   ✅ No threats detected")
        } else {
            fmt.Printf("   ⚠️  %d threat(s) detected:\n", len(threats))
            totalThreats += len(threats)
            
            for _, threat := range threats {
                fmt.Printf("      %s (%s): %s\n", threat.Type, threat.Severity, threat.Description)
                fmt.Printf("      Pattern: %s\n", threat.Pattern)
            }
        }
    }
    
    fmt.Printf("\nTotal threats found: %d\n", totalThreats)
    
    fmt.Println("\n=== Sanitization Examples ===")
    
    // HTML sanitization
    htmlInput := `<script>alert('XSS')</script><p onclick="alert('click')">Click me</p>`
    sanitizedHTML := sanitizer.SanitizeHTML(htmlInput)
    fmt.Printf("HTML Input: %s\n", htmlInput)
    fmt.Printf("Sanitized:  %s\n", sanitizedHTML)
    
    // SQL sanitization
    sqlInput := `'; DROP TABLE users; --`
    sanitizedSQL := sanitizer.SanitizeSQLInput(sqlInput)
    fmt.Printf("\nSQL Input: %s\n", sqlInput)
    fmt.Printf("Sanitized: %s\n", sanitizedSQL)
    
    // File path sanitization
    pathInput := `../../../etc/passwd`
    sanitizedPath := sanitizer.SanitizeFilePath(pathInput)
    fmt.Printf("\nPath Input: %s\n", pathInput)
    fmt.Printf("Sanitized:  %s\n", sanitizedPath)
    
    // Shell command sanitization
    cmdInput := `; rm -rf / && echo "pwned"`
    sanitizedCmd := sanitizer.SanitizeShellCommand(cmdInput)
    fmt.Printf("\nCommand Input: %s\n", cmdInput)
    fmt.Printf("Sanitized:     %s\n", sanitizedCmd)
    
    fmt.Println("\n=== Whitelist Validation ===")
    
    // Whitelist patterns for different input types
    patterns := map[string]string{
        "username":     `[a-zA-Z0-9_]{3,20}`,
        "email":        `[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}`,
        "phone":        `\+?[1-9]\d{1,14}`,
        "alphanumeric": `[a-zA-Z0-9\s]+`,
        "filename":     `[a-zA-Z0-9._-]+\.(txt|pdf|doc|docx|jpg|png)`,
    }
    
    testData := map[string][]string{
        "username":     {"user123", "valid_user", "user@domain", "a", "toolongusername12345"},
        "email":        {"user@example.com", "test.email@domain.org", "invalid.email", "@domain.com"},
        "phone":        {"+1234567890", "555-123-4567", "+331234567890", "invalid-phone"},
        "alphanumeric": {"Hello World", "Test123", "No special chars!", "valid input"},
        "filename":     {"document.pdf", "image.jpg", "file.exe", "test.txt", "malicious.php"},
    }
    
    for inputType, pattern := range patterns {
        fmt.Printf("\n%s validation (pattern: %s):\n", inputType, pattern)
        for _, testValue := range testData[inputType] {
            valid := validateWhitelistPattern(testValue, pattern)
            status := "✅"
            if !valid {
                status = "❌"
            }
            fmt.Printf("  %s %s\n", status, testValue)
        }
    }
    
    fmt.Println("\n=== Security Best Practices ===")
    fmt.Println("1. Use whitelist validation whenever possible")
    fmt.Println("2. Sanitize input based on context (HTML, SQL, Shell, etc.)")
    fmt.Println("3. Escape output appropriately for the target context")
    fmt.Println("4. Use parameterized queries for SQL operations")
    fmt.Println("5. Validate file paths and restrict access")
    fmt.Println("6. Never trust user input - always validate and sanitize")
    fmt.Println("7. Log security events for monitoring and analysis")
    fmt.Println("8. Regular expression should be one layer of a defense-in-depth strategy")
}
```

Security sanitization uses regex patterns to detect and neutralize various  
attack vectors. While effective for input filtering, regex should be part  
of a comprehensive security strategy including proper encoding and  
parameterized queries.  

## Real-world applications

This final example demonstrates practical regex applications in common  
business scenarios and data processing workflows.  

```go
package main

import (
    "fmt"
    "regexp"
    "sort"
    "strconv"
    "strings"
    "time"
)

type BusinessDataProcessor struct {
    addressPattern      *regexp.Regexp
    ssnPattern         *regexp.Regexp
    creditCardPattern  *regexp.Regexp
    invoicePattern     *regexp.Regexp
    productCodePattern *regexp.Regexp
    currencyPattern    *regexp.Regexp
    datePatterns       []*regexp.Regexp
    timePattern        *regexp.Regexp
}

func NewBusinessDataProcessor() *BusinessDataProcessor {
    return &BusinessDataProcessor{
        addressPattern:      regexp.MustCompile(`\d+\s+[A-Za-z0-9\s,.-]+\s+[A-Za-z\s]+,?\s*[A-Z]{2}\s+\d{5}(?:-\d{4})?`),
        ssnPattern:         regexp.MustCompile(`\b\d{3}-\d{2}-\d{4}\b`),
        creditCardPattern:  regexp.MustCompile(`\b(?:\d{4}[-\s]?){3}\d{4}\b`),
        invoicePattern:     regexp.MustCompile(`(?i)(?:invoice|inv)[-\s#]?(\d+)`),
        productCodePattern: regexp.MustCompile(`[A-Z]{2,4}-\d{3,6}(?:-[A-Z0-9]{2,4})?`),
        currencyPattern:    regexp.MustCompile(`\$\d+(?:,\d{3})*(?:\.\d{2})?|\d+(?:,\d{3})*(?:\.\d{2})?\s*(?:USD|EUR|GBP)`),
        datePatterns: []*regexp.Regexp{
            regexp.MustCompile(`\b\d{1,2}/\d{1,2}/\d{4}\b`),
            regexp.MustCompile(`\b\d{4}-\d{2}-\d{2}\b`),
            regexp.MustCompile(`\b[A-Za-z]{3,9}\s+\d{1,2},?\s+\d{4}\b`),
        },
        timePattern: regexp.MustCompile(`\b\d{1,2}:\d{2}(?::\d{2})?\s*(?:[AP]M)?\b`),
    }
}

type BusinessDocument struct {
    Type          string
    Addresses     []string
    SSNs          []string
    CreditCards   []string
    Invoices      []string
    ProductCodes  []string
    Amounts       []string
    Dates         []string
    Times         []string
    PersonalData  bool
    Sensitivity   string
}

func (bdp *BusinessDataProcessor) ProcessDocument(content string, docType string) BusinessDocument {
    doc := BusinessDocument{Type: docType}
    
    // Extract addresses
    doc.Addresses = bdp.addressPattern.FindAllString(content, -1)
    
    // Extract SSNs (mark as sensitive)
    doc.SSNs = bdp.ssnPattern.FindAllString(content, -1)
    
    // Extract credit cards (mark as sensitive)
    doc.CreditCards = bdp.creditCardPattern.FindAllString(content, -1)
    
    // Extract invoice numbers
    invoiceMatches := bdp.invoicePattern.FindAllStringSubmatch(content, -1)
    for _, match := range invoiceMatches {
        if len(match) > 1 {
            doc.Invoices = append(doc.Invoices, match[1])
        }
    }
    
    // Extract product codes
    doc.ProductCodes = bdp.productCodePattern.FindAllString(content, -1)
    
    // Extract monetary amounts
    doc.Amounts = bdp.currencyPattern.FindAllString(content, -1)
    
    // Extract dates (try multiple formats)
    for _, pattern := range bdp.datePatterns {
        dates := pattern.FindAllString(content, -1)
        doc.Dates = append(doc.Dates, dates...)
    }
    
    // Extract times
    doc.Times = bdp.timePattern.FindAllString(content, -1)
    
    // Determine sensitivity
    doc.PersonalData = len(doc.SSNs) > 0 || len(doc.CreditCards) > 0
    if doc.PersonalData {
        doc.Sensitivity = "HIGH"
    } else if len(doc.Addresses) > 0 || len(doc.Amounts) > 0 {
        doc.Sensitivity = "MEDIUM"
    } else {
        doc.Sensitivity = "LOW"
    }
    
    return doc
}

func (bdp *BusinessDataProcessor) GenerateReport(docs []BusinessDocument) {
    fmt.Println("=== Business Data Processing Report ===")
    
    totalDocs := len(docs)
    sensitiveCount := 0
    personalDataCount := 0
    
    entityCounts := map[string]int{
        "addresses":     0,
        "ssns":         0,
        "credit_cards": 0,
        "invoices":     0,
        "product_codes": 0,
        "amounts":      0,
        "dates":        0,
        "times":        0,
    }
    
    for _, doc := range docs {
        entityCounts["addresses"] += len(doc.Addresses)
        entityCounts["ssns"] += len(doc.SSNs)
        entityCounts["credit_cards"] += len(doc.CreditCards)
        entityCounts["invoices"] += len(doc.Invoices)
        entityCounts["product_codes"] += len(doc.ProductCodes)
        entityCounts["amounts"] += len(doc.Amounts)
        entityCounts["dates"] += len(doc.Dates)
        entityCounts["times"] += len(doc.Times)
        
        if doc.Sensitivity == "HIGH" {
            sensitiveCount++
        }
        if doc.PersonalData {
            personalDataCount++
        }
    }
    
    fmt.Printf("Documents processed: %d\n", totalDocs)
    fmt.Printf("High sensitivity: %d (%.1f%%)\n", 
        sensitiveCount, float64(sensitiveCount)*100/float64(totalDocs))
    fmt.Printf("Contains personal data: %d (%.1f%%)\n", 
        personalDataCount, float64(personalDataCount)*100/float64(totalDocs))
    
    fmt.Println("\nEntity extraction summary:")
    for entity, count := range entityCounts {
        if count > 0 {
            fmt.Printf("  %s: %d\n", entity, count)
        }
    }
}

func (bdp *BusinessDataProcessor) RedactSensitiveData(content string) string {
    redacted := content
    
    // Redact SSNs
    redacted = bdp.ssnPattern.ReplaceAllString(redacted, "XXX-XX-XXXX")
    
    // Redact credit cards
    redacted = bdp.creditCardPattern.ReplaceAllStringFunc(redacted, func(match string) string {
        // Keep first 4 and last 4 digits
        cleaned := regexp.MustCompile(`[^\d]`).ReplaceAllString(match, "")
        if len(cleaned) >= 8 {
            return cleaned[:4] + "********" + cleaned[len(cleaned)-4:]
        }
        return "************"
    })
    
    return redacted
}

func processLogFiles(logs []string) map[string]interface{} {
    stats := map[string]interface{}{
        "error_count":     0,
        "warning_count":   0,
        "unique_ips":     make(map[string]int),
        "status_codes":   make(map[string]int),
        "user_agents":    make(map[string]int),
        "top_endpoints":  make(map[string]int),
    }
    
    // Common log patterns
    logLevelPattern := regexp.MustCompile(`\b(ERROR|WARN|INFO|DEBUG)\b`)
    ipPattern := regexp.MustCompile(`\b(?:\d{1,3}\.){3}\d{1,3}\b`)
    statusCodePattern := regexp.MustCompile(`"\s+(\d{3})\s+`)
    userAgentPattern := regexp.MustCompile(`"([^"]*?)"\s*$`)
    endpointPattern := regexp.MustCompile(`"[A-Z]+\s+([^\s"]+)`)
    
    for _, log := range logs {
        // Count log levels
        if level := logLevelPattern.FindString(log); level != "" {
            switch level {
            case "ERROR":
                stats["error_count"] = stats["error_count"].(int) + 1
            case "WARN":
                stats["warning_count"] = stats["warning_count"].(int) + 1
            }
        }
        
        // Extract and count IPs
        if ip := ipPattern.FindString(log); ip != "" {
            ips := stats["unique_ips"].(map[string]int)
            ips[ip]++
        }
        
        // Extract status codes
        if match := statusCodePattern.FindStringSubmatch(log); len(match) > 1 {
            codes := stats["status_codes"].(map[string]int)
            codes[match[1]]++
        }
        
        // Extract user agents
        if match := userAgentPattern.FindStringSubmatch(log); len(match) > 1 {
            agents := stats["user_agents"].(map[string]int)
            agents[match[1]]++
        }
        
        // Extract endpoints
        if match := endpointPattern.FindStringSubmatch(log); len(match) > 1 {
            endpoints := stats["top_endpoints"].(map[string]int)
            endpoints[match[1]]++
        }
    }
    
    return stats
}

func main() {
    processor := NewBusinessDataProcessor()
    
    // Sample business documents
    documents := []string{
        `Invoice #INV-2024-001
        Bill To: John Smith
        123 Main Street, Anytown, CA 90210
        SSN: 123-45-6789
        
        Product: WIDGET-12345-A
        Amount: $1,250.00
        Date: 03/15/2024
        Time: 2:30 PM`,
        
        `Customer Service Report
        Date: March 15, 2024
        
        Customer called regarding order ABC-789123
        Product codes processed: GADGET-5678-B, TOOL-9012-C
        Payment method: Credit card ending in 4567
        
        Total refund: $875.50
        Processing time: 1:45 PM`,
        
        `Marketing Email List
        Contact: sales@company.com
        Updated: 2024-03-15
        
        Customer addresses for mailing:
        - 456 Oak Avenue, Springfield, IL 62701
        - 789 Pine Road, Madison, WI 53703
        
        Budget allocation: $50,000.00 USD`,
    }
    
    fmt.Println("=== Processing Business Documents ===")
    
    var processedDocs []BusinessDocument
    for i, content := range documents {
        doc := processor.ProcessDocument(content, fmt.Sprintf("Document_%d", i+1))
        processedDocs = append(processedDocs, doc)
        
        fmt.Printf("\nDocument %d (%s sensitivity):\n", i+1, doc.Sensitivity)
        if len(doc.Addresses) > 0 {
            fmt.Printf("  Addresses: %v\n", doc.Addresses)
        }
        if len(doc.SSNs) > 0 {
            fmt.Printf("  SSNs: %d found (redacted for security)\n", len(doc.SSNs))
        }
        if len(doc.CreditCards) > 0 {
            fmt.Printf("  Credit Cards: %d found (redacted for security)\n", len(doc.CreditCards))
        }
        if len(doc.Invoices) > 0 {
            fmt.Printf("  Invoices: %v\n", doc.Invoices)
        }
        if len(doc.ProductCodes) > 0 {
            fmt.Printf("  Product Codes: %v\n", doc.ProductCodes)
        }
        if len(doc.Amounts) > 0 {
            fmt.Printf("  Amounts: %v\n", doc.Amounts)
        }
        if len(doc.Dates) > 0 {
            fmt.Printf("  Dates: %v\n", doc.Dates)
        }
    }
    
    processor.GenerateReport(processedDocs)
    
    fmt.Println("\n=== Data Redaction Example ===")
    sensitiveDoc := documents[0]
    redacted := processor.RedactSensitiveData(sensitiveDoc)
    
    fmt.Println("Original (first 100 chars):")
    fmt.Printf("%.100s...\n", sensitiveDoc)
    fmt.Println("\nRedacted (first 100 chars):")
    fmt.Printf("%.100s...\n", redacted)
    
    fmt.Println("\n=== Log File Analysis ===")
    sampleLogs := []string{
        `192.168.1.100 - - [15/Mar/2024:10:30:25 +0000] "GET /api/users HTTP/1.1" 200 1234 "Mozilla/5.0"`,
        `192.168.1.200 - - [15/Mar/2024:10:30:26 +0000] "POST /api/login HTTP/1.1" 401 567 "Chrome/91.0"`,
        `192.168.1.100 - - [15/Mar/2024:10:30:27 +0000] "GET /dashboard HTTP/1.1" 200 2048 "Mozilla/5.0"`,
        `2024-03-15T10:30:28 ERROR Database connection failed`,
        `2024-03-15T10:30:29 WARN Rate limit exceeded for user`,
        `192.168.1.300 - - [15/Mar/2024:10:30:30 +0000] "GET /api/data HTTP/1.1" 500 0 "Python-requests/2.25"`,
    }
    
    logStats := processLogFiles(sampleLogs)
    
    fmt.Printf("Errors: %d\n", logStats["error_count"])
    fmt.Printf("Warnings: %d\n", logStats["warning_count"])
    
    fmt.Println("Unique IPs:")
    ips := logStats["unique_ips"].(map[string]int)
    for ip, count := range ips {
        fmt.Printf("  %s: %d requests\n", ip, count)
    }
    
    fmt.Println("Status codes:")
    codes := logStats["status_codes"].(map[string]int)
    for code, count := range codes {
        fmt.Printf("  %s: %d\n", code, count)
    }
    
    fmt.Println("\n=== Business Intelligence Summary ===")
    fmt.Println("✓ Automated document classification and sensitivity analysis")
    fmt.Println("✓ PII detection and redaction for compliance")
    fmt.Println("✓ Financial data extraction and validation")
    fmt.Println("✓ Business entity recognition and cataloging")
    fmt.Println("✓ Log analysis and operational insights")
    fmt.Println("✓ Data quality assessment and reporting")
    
    fmt.Println("\nRegular expressions provide powerful business value through:")
    fmt.Println("• Automated data extraction and classification")
    fmt.Println("• Compliance and security enforcement")  
    fmt.Println("• Business intelligence and analytics")
    fmt.Println("• Process automation and quality assurance")
    fmt.Println("• Cost reduction through efficient data processing")
}
```

Real-world applications demonstrate regex value in business contexts through  
automated data extraction, compliance monitoring, and operational analytics.  
These patterns form the foundation of enterprise data processing pipelines  
and business intelligence systems.  

## Network protocol parsing

Regular expressions can parse and validate network protocols and  
communication formats commonly used in networking applications.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func parseHTTPHeaders(headers string) map[string]string {
    headerPattern := regexp.MustCompile(`(?m)^([^:]+):\s*(.*)$`)
    result := make(map[string]string)
    
    matches := headerPattern.FindAllStringSubmatch(headers, -1)
    for _, match := range matches {
        if len(match) == 3 {
            result[strings.TrimSpace(match[1])] = strings.TrimSpace(match[2])
        }
    }
    
    return result
}

func parseUserAgent(userAgent string) map[string]string {
    // Parse browser and OS information
    browserPattern := regexp.MustCompile(`(Chrome|Firefox|Safari|Edge|Opera)/([0-9.]+)`)
    osPattern := regexp.MustCompile(`\(([^)]+)\)`)
    
    info := make(map[string]string)
    
    if match := browserPattern.FindStringSubmatch(userAgent); len(match) > 2 {
        info["browser"] = match[1]
        info["version"] = match[2]
    }
    
    if match := osPattern.FindStringSubmatch(userAgent); len(match) > 1 {
        info["os_info"] = match[1]
    }
    
    return info
}

func main() {
    // HTTP headers parsing
    headers := `Content-Type: application/json
Authorization: Bearer abc123xyz
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36`
    
    parsed := parseHTTPHeaders(headers)
    fmt.Println("Parsed HTTP headers:")
    for key, value := range parsed {
        fmt.Printf("  %s: %s\n", key, value)
    }
    
    // User agent parsing
    userAgent := "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
    uaInfo := parseUserAgent(userAgent)
    fmt.Println("\nUser Agent Analysis:")
    for key, value := range uaInfo {
        fmt.Printf("  %s: %s\n", key, value)
    }
}
```

Network protocol parsing extracts structured information from communication  
headers and protocol messages, enabling network monitoring and analysis.  

## JSON-like data extraction

Regular expressions can extract data from JSON-like structures when full  
JSON parsing is not available or practical.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

func extractJSONValues(jsonStr string) map[string]string {
    // Pattern for simple JSON key-value pairs
    pattern := regexp.MustCompile(`"([^"]+)":\s*"([^"]*)"`)
    result := make(map[string]string)
    
    matches := pattern.FindAllStringSubmatch(jsonStr, -1)
    for _, match := range matches {
        if len(match) == 3 {
            result[match[1]] = match[2]
        }
    }
    
    return result
}

func extractJSONNumbers(jsonStr string) map[string]string {
    // Pattern for numeric values
    pattern := regexp.MustCompile(`"([^"]+)":\s*(\d+(?:\.\d+)?)`)
    result := make(map[string]string)
    
    matches := pattern.FindAllStringSubmatch(jsonStr, -1)
    for _, match := range matches {
        if len(match) == 3 {
            result[match[1]] = match[2]
        }
    }
    
    return result
}

func main() {
    jsonData := `{
        "name": "John Doe",
        "email": "john@example.com",
        "age": 30,
        "salary": 75000.50,
        "active": true,
        "department": "Engineering"
    }`
    
    strings := extractJSONValues(jsonData)
    fmt.Println("String values:")
    for key, value := range strings {
        fmt.Printf("  %s: %s\n", key, value)
    }
    
    numbers := extractJSONNumbers(jsonData)
    fmt.Println("\nNumeric values:")
    for key, value := range numbers {
        fmt.Printf("  %s: %s\n", key, value)
    }
}
```

JSON-like extraction handles semi-structured data extraction when dealing  
with log files, configuration snippets, or API responses that contain  
JSON fragments.  

## Code syntax highlighting

Regular expressions power syntax highlighting by identifying programming  
language constructs and code patterns.  

```go
package main

import (
    "fmt"
    "regexp"
)

type SyntaxHighlighter struct {
    keywordPattern   *regexp.Regexp
    stringPattern    *regexp.Regexp
    commentPattern   *regexp.Regexp
    numberPattern    *regexp.Regexp
    functionPattern  *regexp.Regexp
    variablePattern  *regexp.Regexp
}

func NewSyntaxHighlighter() *SyntaxHighlighter {
    return &SyntaxHighlighter{
        keywordPattern:  regexp.MustCompile(`\b(func|var|const|if|else|for|range|return|package|import|struct|interface|type)\b`),
        stringPattern:   regexp.MustCompile(`"[^"]*"|` + "`[^`]*`"),
        commentPattern:  regexp.MustCompile(`//.*$|/\*.*?\*/`),
        numberPattern:   regexp.MustCompile(`\b\d+(?:\.\d+)?\b`),
        functionPattern: regexp.MustCompile(`(\w+)\s*\(`),
        variablePattern: regexp.MustCompile(`\b[a-z][a-zA-Z0-9]*\b`),
    }
}

func (sh *SyntaxHighlighter) HighlightGo(code string) string {
    highlighted := code
    
    // Apply syntax highlighting (simplified HTML-like tags)
    highlighted = sh.keywordPattern.ReplaceAllString(highlighted, "<keyword>$1</keyword>")
    highlighted = sh.stringPattern.ReplaceAllString(highlighted, "<string>$1</string>")
    highlighted = sh.commentPattern.ReplaceAllString(highlighted, "<comment>$1</comment>")
    highlighted = sh.numberPattern.ReplaceAllString(highlighted, "<number>$1</number>")
    
    return highlighted
}

func main() {
    highlighter := NewSyntaxHighlighter()
    
    goCode := `package main

import "fmt"

func main() {
    var message string = "hello there"
    count := 42
    // This is a comment
    fmt.Println(message, count)
}`
    
    highlighted := highlighter.HighlightGo(goCode)
    fmt.Println("Syntax highlighted Go code:")
    fmt.Println(highlighted)
    
    // Count different syntax elements
    keywords := highlighter.keywordPattern.FindAllString(goCode, -1)
    strings := highlighter.stringPattern.FindAllString(goCode, -1)
    comments := highlighter.commentPattern.FindAllString(goCode, -1)
    
    fmt.Printf("\nSyntax analysis:\n")
    fmt.Printf("Keywords: %d (%v)\n", len(keywords), keywords)
    fmt.Printf("Strings: %d (%v)\n", len(strings), strings)
    fmt.Printf("Comments: %d (%v)\n", len(comments), comments)
}
```

Syntax highlighting uses regex patterns to identify and categorize different  
programming language elements for editors, documentation tools, and  
code analysis applications.  

## Template engine basics

Regular expressions can power simple template engines for text generation  
and dynamic content creation.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type SimpleTemplateEngine struct {
    variablePattern   *regexp.Regexp
    conditionalPattern *regexp.Regexp
    loopPattern       *regexp.Regexp
}

func NewSimpleTemplateEngine() *SimpleTemplateEngine {
    return &SimpleTemplateEngine{
        variablePattern:   regexp.MustCompile(`\{\{\s*(\w+)\s*\}\}`),
        conditionalPattern: regexp.MustCompile(`\{\{\s*if\s+(\w+)\s*\}\}(.*?)\{\{\s*endif\s*\}\}`),
        loopPattern:       regexp.MustCompile(`\{\{\s*range\s+(\w+)\s*\}\}(.*?)\{\{\s*endrange\s*\}\}`),
    }
}

func (ste *SimpleTemplateEngine) Render(template string, data map[string]interface{}) string {
    result := template
    
    // Replace simple variables
    result = ste.variablePattern.ReplaceAllStringFunc(result, func(match string) string {
        varName := ste.variablePattern.FindStringSubmatch(match)[1]
        if value, exists := data[varName]; exists {
            return fmt.Sprintf("%v", value)
        }
        return match
    })
    
    // Handle conditionals
    result = ste.conditionalPattern.ReplaceAllStringFunc(result, func(match string) string {
        matches := ste.conditionalPattern.FindStringSubmatch(match)
        if len(matches) >= 3 {
            condition := matches[1]
            content := matches[2]
            
            if value, exists := data[condition]; exists {
                if isTruthy(value) {
                    return content
                }
            }
        }
        return ""
    })
    
    return result
}

func isTruthy(value interface{}) bool {
    switch v := value.(type) {
    case bool:
        return v
    case string:
        return v != ""
    case int:
        return v != 0
    case []interface{}:
        return len(v) > 0
    default:
        return value != nil
    }
}

func main() {
    engine := NewSimpleTemplateEngine()
    
    template := `Dear {{ name }},

{{ if premium }}
Thank you for being a premium member!
Your account balance is ${{ balance }}.
{{ endif }}

{{ if notifications }}
You have new notifications waiting.
{{ endif }}

Best regards,
The Team`
    
    // Test data
    data := map[string]interface{}{
        "name":          "Alice Johnson",
        "premium":       true,
        "balance":       1250.00,
        "notifications": false,
    }
    
    rendered := engine.Render(template, data)
    fmt.Println("Rendered template:")
    fmt.Println(rendered)
    
    fmt.Println("\n" + strings.Repeat("=", 40))
    
    // Test with different data
    data2 := map[string]interface{}{
        "name":          "Bob Smith",
        "premium":       false,
        "notifications": true,
    }
    
    rendered2 := engine.Render(template, data2)
    fmt.Println("Rendered template (different data):")
    fmt.Println(rendered2)
}
```

Template engines use regex patterns to identify and replace placeholders,  
conditionals, and dynamic content markers with actual data values.  

## Database connection string parsing

Regular expressions parse database connection strings and extract connection  
parameters for database configuration and validation.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type DBConnectionParser struct {
    postgresPattern *regexp.Regexp
    mysqlPattern    *regexp.Regexp
    sqlitePattern   *regexp.Regexp
    mongoPattern    *regexp.Regexp
    urlPattern      *regexp.Regexp
}

func NewDBConnectionParser() *DBConnectionParser {
    return &DBConnectionParser{
        postgresPattern: regexp.MustCompile(`^postgresql://(?:([^:@]+)(?::([^@]*))?@)?([^:/]+)(?::(\d+))?/([^?]+)(?:\?(.*))?$`),
        mysqlPattern:    regexp.MustCompile(`^mysql://(?:([^:@]+)(?::([^@]*))?@)?([^:/]+)(?::(\d+))?/([^?]+)(?:\?(.*))?$`),
        sqlitePattern:   regexp.MustCompile(`^sqlite:///(.+)$`),
        mongoPattern:    regexp.MustCompile(`^mongodb://(?:([^:@]+)(?::([^@]*))?@)?([^:/]+)(?::(\d+))?/([^?]+)(?:\?(.*))?$`),
        urlPattern:      regexp.MustCompile(`^([^:]+)://(.+)$`),
    }
}

type DBConfig struct {
    Driver   string
    Username string
    Password string
    Host     string
    Port     string
    Database string
    Options  map[string]string
    Valid    bool
    Error    string
}

func (dcp *DBConnectionParser) ParseConnectionString(connStr string) DBConfig {
    config := DBConfig{Options: make(map[string]string)}
    
    // Determine database type
    urlMatch := dcp.urlPattern.FindStringSubmatch(connStr)
    if len(urlMatch) < 2 {
        config.Error = "Invalid connection string format"
        return config
    }
    
    config.Driver = urlMatch[1]
    
    switch strings.ToLower(config.Driver) {
    case "postgresql", "postgres":
        return dcp.parsePostgreSQL(connStr)
    case "mysql":
        return dcp.parseMySQL(connStr)
    case "sqlite":
        return dcp.parseSQLite(connStr)
    case "mongodb":
        return dcp.parseMongoDB(connStr)
    default:
        config.Error = fmt.Sprintf("Unsupported database driver: %s", config.Driver)
        return config
    }
}

func (dcp *DBConnectionParser) parsePostgreSQL(connStr string) DBConfig {
    config := DBConfig{Driver: "postgresql", Options: make(map[string]string)}
    
    matches := dcp.postgresPattern.FindStringSubmatch(connStr)
    if len(matches) < 6 {
        config.Error = "Invalid PostgreSQL connection string"
        return config
    }
    
    config.Username = matches[1]
    config.Password = matches[2]
    config.Host = matches[3]
    config.Port = matches[4]
    config.Database = matches[5]
    
    if len(matches) > 6 && matches[6] != "" {
        config.Options = parseQueryString(matches[6])
    }
    
    config.Valid = true
    return config
}

func (dcp *DBConnectionParser) parseMySQL(connStr string) DBConfig {
    config := DBConfig{Driver: "mysql", Options: make(map[string]string)}
    
    matches := dcp.mysqlPattern.FindStringSubmatch(connStr)
    if len(matches) < 6 {
        config.Error = "Invalid MySQL connection string"
        return config
    }
    
    config.Username = matches[1]
    config.Password = matches[2]
    config.Host = matches[3]
    config.Port = matches[4]
    config.Database = matches[5]
    
    if len(matches) > 6 && matches[6] != "" {
        config.Options = parseQueryString(matches[6])
    }
    
    config.Valid = true
    return config
}

func (dcp *DBConnectionParser) parseSQLite(connStr string) DBConfig {
    config := DBConfig{Driver: "sqlite", Options: make(map[string]string)}
    
    matches := dcp.sqlitePattern.FindStringSubmatch(connStr)
    if len(matches) < 2 {
        config.Error = "Invalid SQLite connection string"
        return config
    }
    
    config.Database = matches[1]
    config.Valid = true
    return config
}

func (dcp *DBConnectionParser) parseMongoDB(connStr string) DBConfig {
    config := DBConfig{Driver: "mongodb", Options: make(map[string]string)}
    
    matches := dcp.mongoPattern.FindStringSubmatch(connStr)
    if len(matches) < 6 {
        config.Error = "Invalid MongoDB connection string"
        return config
    }
    
    config.Username = matches[1]
    config.Password = matches[2]
    config.Host = matches[3]
    config.Port = matches[4]
    config.Database = matches[5]
    
    if len(matches) > 6 && matches[6] != "" {
        config.Options = parseQueryString(matches[6])
    }
    
    config.Valid = true
    return config
}

func parseQueryString(query string) map[string]string {
    options := make(map[string]string)
    pairs := strings.Split(query, "&")
    
    for _, pair := range pairs {
        parts := strings.SplitN(pair, "=", 2)
        if len(parts) == 2 {
            options[parts[0]] = parts[1]
        } else if len(parts) == 1 {
            options[parts[0]] = "true"
        }
    }
    
    return options
}

func main() {
    parser := NewDBConnectionParser()
    
    connectionStrings := []string{
        "postgresql://user:password@localhost:5432/mydb?sslmode=require",
        "mysql://admin:secret@db.example.com:3306/production",
        "sqlite:///path/to/database.db",
        "mongodb://user:pass@mongo.example.com:27017/app?authSource=admin",
        "postgresql://localhost/testdb",
        "invalid://connection/string",
    }
    
    fmt.Println("Database Connection String Analysis:")
    fmt.Println(strings.Repeat("=", 50))
    
    for i, connStr := range connectionStrings {
        fmt.Printf("\n%d. %s\n", i+1, connStr)
        
        config := parser.ParseConnectionString(connStr)
        
        if !config.Valid {
            fmt.Printf("   ❌ Error: %s\n", config.Error)
            continue
        }
        
        fmt.Printf("   ✅ Valid %s connection\n", config.Driver)
        if config.Username != "" {
            fmt.Printf("   Username: %s\n", config.Username)
        }
        if config.Password != "" {
            fmt.Printf("   Password: %s\n", maskPassword(config.Password))
        }
        if config.Host != "" {
            fmt.Printf("   Host: %s\n", config.Host)
        }
        if config.Port != "" {
            fmt.Printf("   Port: %s\n", config.Port)
        }
        if config.Database != "" {
            fmt.Printf("   Database: %s\n", config.Database)
        }
        if len(config.Options) > 0 {
            fmt.Printf("   Options: %v\n", config.Options)
        }
    }
    
    // Security analysis
    fmt.Println("\n" + strings.Repeat("=", 50))
    fmt.Println("Security Analysis:")
    
    securityIssues := 0
    for _, connStr := range connectionStrings {
        config := parser.ParseConnectionString(connStr)
        if config.Valid {
            if strings.Contains(connStr, "password") && !strings.Contains(connStr, "localhost") {
                fmt.Printf("⚠️  Password in connection string: %s\n", maskConnectionString(connStr))
                securityIssues++
            }
        }
    }
    
    if securityIssues == 0 {
        fmt.Println("✅ No obvious security issues found")
    } else {
        fmt.Printf("⚠️  Found %d potential security issues\n", securityIssues)
    }
}

func maskPassword(password string) string {
    if len(password) <= 2 {
        return "***"
    }
    return password[:1] + strings.Repeat("*", len(password)-2) + password[len(password)-1:]
}

func maskConnectionString(connStr string) string {
    passwordPattern := regexp.MustCompile(`:([^@:]+)@`)
    return passwordPattern.ReplaceAllString(connStr, ":***@")
}
```

Database connection string parsing extracts configuration parameters and  
validates connection formats for different database systems, supporting  
application configuration and deployment automation.  

## API endpoint pattern matching

Regular expressions validate and parse RESTful API endpoints and route  
patterns commonly used in web services.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type APIRoute struct {
    Pattern    string
    Method     string
    Regex      *regexp.Regexp
    Parameters []string
}

type APIRouter struct {
    routes []APIRoute
}

func NewAPIRouter() *APIRouter {
    return &APIRouter{routes: []APIRoute{}}
}

func (ar *APIRouter) AddRoute(pattern, method string) {
    // Convert route pattern to regex
    regexPattern := pattern
    var parameters []string
    
    // Find path parameters like {id} or {name}
    paramPattern := regexp.MustCompile(`\{(\w+)\}`)
    matches := paramPattern.FindAllStringSubmatch(pattern, -1)
    
    for _, match := range matches {
        if len(match) > 1 {
            parameters = append(parameters, match[1])
        }
    }
    
    // Replace {param} with capture groups
    regexPattern = paramPattern.ReplaceAllString(regexPattern, `([^/]+)`)
    
    // Escape forward slashes and add anchors
    regexPattern = strings.ReplaceAll(regexPattern, "/", "\\/")
    regexPattern = "^" + regexPattern + "$"
    
    compiled := regexp.MustCompile(regexPattern)
    
    route := APIRoute{
        Pattern:    pattern,
        Method:     method,
        Regex:      compiled,
        Parameters: parameters,
    }
    
    ar.routes = append(ar.routes, route)
}

func (ar *APIRouter) MatchRoute(path, method string) (*APIRoute, map[string]string) {
    for _, route := range ar.routes {
        if route.Method != method {
            continue
        }
        
        if matches := route.Regex.FindStringSubmatch(path); matches != nil {
            params := make(map[string]string)
            
            // Map captured groups to parameter names
            for i, paramName := range route.Parameters {
                if i+1 < len(matches) {
                    params[paramName] = matches[i+1]
                }
            }
            
            return &route, params
        }
    }
    
    return nil, nil
}

func validateAPIPath(path string) []string {
    var issues []string
    
    // Check for common API path issues
    patterns := map[string]string{
        "double_slash":     `//+`,
        "trailing_slash":   `/$`,
        "special_chars":    `[^a-zA-Z0-9/{}_-]`,
        "consecutive_params": `\{[^}]+\}\{[^}]+\}`,
        "empty_segments":   `/(?=/)|^$`,
    }
    
    for issue, pattern := range patterns {
        if matched, _ := regexp.MatchString(pattern, path); matched {
            issues = append(issues, issue)
        }
    }
    
    return issues
}

func extractAPIVersion(path string) string {
    versionPattern := regexp.MustCompile(`/v(\d+(?:\.\d+)?)/`)
    if matches := versionPattern.FindStringSubmatch(path); len(matches) > 1 {
        return matches[1]
    }
    return ""
}

func main() {
    router := NewAPIRouter()
    
    // Add API routes
    apiRoutes := []struct {
        pattern string
        method  string
    }{
        {"/api/v1/users", "GET"},
        {"/api/v1/users/{id}", "GET"},
        {"/api/v1/users/{id}", "PUT"},
        {"/api/v1/users/{id}", "DELETE"},
        {"/api/v1/users/{userId}/posts", "GET"},
        {"/api/v1/users/{userId}/posts/{postId}", "GET"},
        {"/api/v2/products/{category}/{id}", "GET"},
        {"/health", "GET"},
        {"/metrics", "GET"},
    }
    
    for _, route := range apiRoutes {
        router.AddRoute(route.pattern, route.method)
    }
    
    fmt.Println("=== API Route Matching ===")
    
    // Test requests
    testRequests := []struct {
        path   string
        method string
    }{
        {"/api/v1/users", "GET"},
        {"/api/v1/users/123", "GET"},
        {"/api/v1/users/456", "PUT"},
        {"/api/v1/users/789/posts", "GET"},
        {"/api/v1/users/101/posts/202", "GET"},
        {"/api/v2/products/electronics/42", "GET"},
        {"/api/v1/invalid/path", "GET"},
        {"/health", "GET"},
    }
    
    for _, req := range testRequests {
        route, params := router.MatchRoute(req.path, req.method)
        
        fmt.Printf("Request: %s %s\n", req.method, req.path)
        
        if route != nil {
            fmt.Printf("  ✅ Matched: %s\n", route.Pattern)
            if len(params) > 0 {
                fmt.Printf("  Parameters: %v\n", params)
            }
            
            // Extract API version
            if version := extractAPIVersion(req.path); version != "" {
                fmt.Printf("  API Version: %s\n", version)
            }
        } else {
            fmt.Printf("  ❌ No route matched\n")
        }
        
        fmt.Println()
    }
    
    fmt.Println("=== API Path Validation ===")
    
    testPaths := []string{
        "/api/v1/users",
        "/api/v1//users",
        "/api/v1/users/",
        "/api/v1/users/{id}",
        "/api/v1/users@invalid",
        "/api/v1/{userId}{postId}",
        "",
        "/valid-path_123",
    }
    
    for _, path := range testPaths {
        issues := validateAPIPath(path)
        
        fmt.Printf("Path: %s\n", path)
        if len(issues) == 0 {
            fmt.Printf("  ✅ Valid\n")
        } else {
            fmt.Printf("  ❌ Issues: %v\n", issues)
        }
        fmt.Println()
    }
    
    fmt.Println("=== API Documentation Generation ===")
    fmt.Println("Detected API endpoints:")
    
    versionGroups := make(map[string][]APIRoute)
    for _, route := range router.routes {
        version := extractAPIVersion(route.Pattern)
        if version == "" {
            version = "unversioned"
        }
        versionGroups[version] = append(versionGroups[version], route)
    }
    
    for version, routes := range versionGroups {
        fmt.Printf("\nAPI Version: %s\n", version)
        for _, route := range routes {
            fmt.Printf("  %s %s", route.Method, route.Pattern)
            if len(route.Parameters) > 0 {
                fmt.Printf(" (params: %s)", strings.Join(route.Parameters, ", "))
            }
            fmt.Println()
        }
    }
}
```

API endpoint pattern matching enables dynamic routing, parameter extraction,  
and API documentation generation for web services and microservice  
architectures.  

## Financial data processing

Regular expressions parse and validate financial data formats including  
currencies, account numbers, and transaction records.  

```go
package main

import (
    "fmt"
    "regexp"
    "strconv"
    "strings"
)

type FinancialProcessor struct {
    currencyPattern    *regexp.Regexp
    ibanPattern       *regexp.Regexp
    swiftPattern      *regexp.Regexp
    accountPattern    *regexp.Regexp
    transactionPattern *regexp.Regexp
    percentagePattern *regexp.Regexp
}

func NewFinancialProcessor() *FinancialProcessor {
    return &FinancialProcessor{
        currencyPattern:    regexp.MustCompile(`([A-Z]{3})\s*([\d,]+\.?\d*)|([£$€¥])([\d,]+\.?\d*)`),
        ibanPattern:       regexp.MustCompile(`^[A-Z]{2}[0-9]{2}[A-Z0-9]{4,30}$`),
        swiftPattern:      regexp.MustCompile(`^[A-Z]{6}[A-Z0-9]{2}([A-Z0-9]{3})?$`),
        accountPattern:    regexp.MustCompile(`\b\d{8,12}\b`),
        transactionPattern: regexp.MustCompile(`(\d{4}-\d{2}-\d{2})\s+([+-]?[£$€¥]?[\d,]+\.?\d*)\s+(.+)`),
        percentagePattern: regexp.MustCompile(`(\d+(?:\.\d+)?)\s*%`),
    }
}

type Transaction struct {
    Date        string
    Amount      string
    Currency    string
    Description string
    Type        string
}

type FinancialSummary struct {
    TotalTransactions int
    TotalAmount       float64
    Currency          string
    Debits            float64
    Credits           float64
    Accounts          []string
    IBANs             []string
    SWIFTCodes        []string
}

func (fp *FinancialProcessor) ParseTransactions(data string) []Transaction {
    var transactions []Transaction
    
    matches := fp.transactionPattern.FindAllStringSubmatch(data, -1)
    for _, match := range matches {
        if len(match) >= 4 {
            transaction := Transaction{
                Date:        match[1],
                Amount:      match[2],
                Description: strings.TrimSpace(match[3]),
            }
            
            // Determine transaction type
            if strings.HasPrefix(transaction.Amount, "-") || strings.HasPrefix(transaction.Amount, "+") {
                if strings.HasPrefix(transaction.Amount, "-") {
                    transaction.Type = "debit"
                } else {
                    transaction.Type = "credit"
                }
            } else {
                // Infer from description or amount context
                description := strings.ToLower(transaction.Description)
                if strings.Contains(description, "payment") || strings.Contains(description, "purchase") {
                    transaction.Type = "debit"
                } else if strings.Contains(description, "deposit") || strings.Contains(description, "refund") {
                    transaction.Type = "credit"
                } else {
                    transaction.Type = "unknown"
                }
            }
            
            // Extract currency
            currencyMatches := fp.currencyPattern.FindStringSubmatch(transaction.Amount)
            if len(currencyMatches) > 0 {
                if currencyMatches[1] != "" {
                    transaction.Currency = currencyMatches[1]
                } else if currencyMatches[3] != "" {
                    // Map symbols to codes
                    symbolMap := map[string]string{"$": "USD", "€": "EUR", "£": "GBP", "¥": "JPY"}
                    transaction.Currency = symbolMap[currencyMatches[3]]
                }
            }
            
            transactions = append(transactions, transaction)
        }
    }
    
    return transactions
}

func (fp *FinancialProcessor) ExtractFinancialData(text string) FinancialSummary {
    summary := FinancialSummary{}
    
    // Extract account numbers
    accounts := fp.accountPattern.FindAllString(text, -1)
    summary.Accounts = removeDuplicates(accounts)
    
    // Extract IBAN codes
    ibans := fp.ibanPattern.FindAllString(text, -1)
    summary.IBANs = removeDuplicates(ibans)
    
    // Extract SWIFT codes  
    swiftCodes := fp.swiftPattern.FindAllString(text, -1)
    summary.SWIFTCodes = removeDuplicates(swiftCodes)
    
    // Extract and sum amounts
    currencyMatches := fp.currencyPattern.FindAllStringSubmatch(text, -1)
    for _, match := range currencyMatches {
        var amountStr string
        var currency string
        
        if match[1] != "" && match[2] != "" {
            currency = match[1]
            amountStr = match[2]
        } else if match[3] != "" && match[4] != "" {
            symbolMap := map[string]string{"$": "USD", "€": "EUR", "£": "GBP", "¥": "JPY"}
            currency = symbolMap[match[3]]
            amountStr = match[4]
        }
        
        if amountStr != "" {
            // Clean amount string
            cleanAmount := strings.ReplaceAll(amountStr, ",", "")
            if amount, err := strconv.ParseFloat(cleanAmount, 64); err == nil {
                summary.TotalAmount += amount
                if summary.Currency == "" {
                    summary.Currency = currency
                }
            }
        }
    }
    
    return summary
}

func (fp *FinancialProcessor) ValidateIBAN(iban string) bool {
    // Remove spaces and convert to uppercase
    cleanIBAN := strings.ReplaceAll(strings.ToUpper(iban), " ", "")
    
    // Check format
    if !fp.ibanPattern.MatchString(cleanIBAN) {
        return false
    }
    
    // Basic length validation for common countries
    countryLengths := map[string]int{
        "AD": 24, "AE": 23, "AL": 28, "AT": 20, "AZ": 28, "BA": 20, "BE": 16,
        "BG": 22, "BH": 22, "BR": 29, "BY": 28, "CH": 21, "CR": 22, "CY": 28,
        "CZ": 24, "DE": 22, "DK": 18, "DO": 28, "EE": 20, "EG": 29, "ES": 24,
        "FI": 18, "FO": 18, "FR": 27, "GB": 22, "GE": 22, "GI": 23, "GL": 18,
        "GR": 27, "GT": 28, "HR": 21, "HU": 28, "IE": 22, "IL": 23, "IS": 26,
        "IT": 27, "JO": 30, "KW": 30, "KZ": 20, "LB": 28, "LC": 32, "LI": 21,
        "LT": 20, "LU": 20, "LV": 21, "MC": 27, "MD": 24, "ME": 22, "MK": 19,
        "MR": 27, "MT": 31, "MU": 30, "NL": 18, "NO": 15, "PK": 24, "PL": 28,
        "PS": 29, "PT": 25, "QA": 29, "RO": 24, "RS": 22, "SA": 24, "SE": 24,
        "SI": 19, "SK": 24, "SM": 27, "TN": 24, "TR": 26, "UA": 29, "VG": 24,
        "XK": 20,
    }
    
    countryCode := cleanIBAN[:2]
    if expectedLength, exists := countryLengths[countryCode]; exists {
        return len(cleanIBAN) == expectedLength
    }
    
    return true // Unknown country, assume valid format
}

func removeDuplicates(slice []string) []string {
    keys := make(map[string]bool)
    var result []string
    
    for _, item := range slice {
        if !keys[item] {
            keys[item] = true
            result = append(result, item)
        }
    }
    
    return result
}

func main() {
    processor := NewFinancialProcessor()
    
    financialData := `Financial Statement - Q1 2024
    
Account: 1234567890
IBAN: DE89370400440532013000  
SWIFT: DEUTDEFF

Transaction History:
2024-03-15 +$1,250.00 Salary deposit
2024-03-16 -$75.50 Grocery payment  
2024-03-17 -$1,200.00 Rent payment
2024-03-18 +€500.00 Freelance payment
2024-03-19 -£25.99 Online subscription

Interest rate: 2.5%
Service charge: 0.1% of balance

Additional accounts: 
- EUR Account: 9876543210
- IBAN: FR1420041010050500013M02606
- SWIFT: BNPAFRPP

Total portfolio value: $15,750.00 USD`
    
    fmt.Println("=== Financial Data Processing ===")
    
    // Parse transactions
    transactions := processor.ParseTransactions(financialData)
    fmt.Printf("Found %d transactions:\n", len(transactions))
    
    for i, tx := range transactions {
        fmt.Printf("  %d. %s: %s %s (%s) - %s\n", 
            i+1, tx.Date, tx.Amount, tx.Currency, tx.Type, tx.Description)
    }
    
    // Extract financial summary
    fmt.Println("\n=== Financial Summary ===")
    summary := processor.ExtractFinancialData(financialData)
    
    fmt.Printf("Accounts found: %v\n", summary.Accounts)
    fmt.Printf("IBAN codes: %v\n", summary.IBANs)
    fmt.Printf("SWIFT codes: %v\n", summary.SWIFTCodes)
    
    // Validate IBANs
    fmt.Println("\n=== IBAN Validation ===")
    for _, iban := range summary.IBANs {
        valid := processor.ValidateIBAN(iban)
        status := "✅"
        if !valid {
            status = "❌"
        }
        fmt.Printf("%s %s\n", status, iban)
    }
    
    // Extract percentages
    percentages := processor.percentagePattern.FindAllString(financialData, -1)
    if len(percentages) > 0 {
        fmt.Printf("\nPercentages found: %v\n", percentages)
    }
    
    // Currency analysis
    currencies := processor.currencyPattern.FindAllString(financialData, -1)
    fmt.Printf("\nCurrency amounts: %v\n", currencies)
    
    fmt.Println("\n=== Compliance Check ===")
    fmt.Println("✅ PII data detected (account numbers, IBANs)")
    fmt.Println("✅ Multi-currency transactions identified")
    fmt.Println("✅ International banking codes validated")
    fmt.Printf("✅ %d financial instruments processed\n", len(transactions))
    
    if len(summary.IBANs) > 0 {
        fmt.Println("⚠️  International transfers detected - verify compliance")
    }
}
```

Financial data processing handles complex monetary formats, international  
banking codes, and transaction parsing for compliance, reporting, and  
financial analysis applications.  

## Machine learning feature extraction

Regular expressions extract features from text for machine learning models  
and natural language processing workflows.  

```go
package main

import (
    "fmt"
    "regexp"
    "sort"
    "strconv"
    "strings"
)

type TextFeatureExtractor struct {
    wordPattern        *regexp.Regexp
    sentencePattern    *regexp.Regexp
    capitalsPattern    *regexp.Regexp
    numbersPattern     *regexp.Regexp
    punctuationPattern *regexp.Regexp
    emotionPattern     *regexp.Regexp
    questionPattern    *regexp.Regexp
    exclamationPattern *regexp.Regexp
}

func NewTextFeatureExtractor() *TextFeatureExtractor {
    return &TextFeatureExtractor{
        wordPattern:        regexp.MustCompile(`\b\w+\b`),
        sentencePattern:    regexp.MustCompile(`[.!?]+`),
        capitalsPattern:    regexp.MustCompile(`[A-Z]`),
        numbersPattern:     regexp.MustCompile(`\d+`),
        punctuationPattern: regexp.MustCompile(`[^\w\s]`),
        emotionPattern:     regexp.MustCompile(`\b(happy|sad|angry|excited|disappointed|amazing|terrible|love|hate|wonderful|awful)\b`),
        questionPattern:    regexp.MustCompile(`\?`),
        exclamationPattern: regexp.MustCompile(`!`),
    }
}

type TextFeatures struct {
    // Basic counts
    WordCount       int
    SentenceCount   int
    CharacterCount  int
    CapitalCount    int
    NumberCount     int
    PunctuationCount int
    
    // Ratios and averages
    AvgWordsPerSentence float64
    CapitalRatio        float64
    NumberRatio         float64
    PunctuationRatio    float64
    
    // Sentiment indicators
    EmotionWords      []string
    EmotionScore      float64
    QuestionCount     int
    ExclamationCount  int
    
    // Vocabulary analysis
    UniqueWords       int
    VocabularyRichness float64
    LongestWord       int
    
    // Complexity measures
    ComplexityScore   float64
    ReadabilityLevel  string
    
    // N-grams
    Bigrams          map[string]int
    CommonPatterns   []string
}

func (tfe *TextFeatureExtractor) ExtractFeatures(text string) TextFeatures {
    features := TextFeatures{
        Bigrams: make(map[string]int),
    }
    
    // Basic counts
    words := tfe.wordPattern.FindAllString(text, -1)
    sentences := tfe.sentencePattern.FindAllString(text, -1)
    capitals := tfe.capitalsPattern.FindAllString(text, -1)
    numbers := tfe.numbersPattern.FindAllString(text, -1)
    punctuation := tfe.punctuationPattern.FindAllString(text, -1)
    
    features.WordCount = len(words)
    features.SentenceCount = len(sentences)
    features.CharacterCount = len(text)
    features.CapitalCount = len(capitals)
    features.NumberCount = len(numbers)
    features.PunctuationCount = len(punctuation)
    
    // Calculate ratios
    if features.CharacterCount > 0 {
        features.CapitalRatio = float64(features.CapitalCount) / float64(features.CharacterCount)
        features.NumberRatio = float64(features.NumberCount) / float64(features.CharacterCount)
        features.PunctuationRatio = float64(features.PunctuationCount) / float64(features.CharacterCount)
    }
    
    // Average words per sentence
    if features.SentenceCount > 0 {
        features.AvgWordsPerSentence = float64(features.WordCount) / float64(features.SentenceCount)
    }
    
    // Sentiment analysis
    emotions := tfe.emotionPattern.FindAllString(strings.ToLower(text), -1)
    features.EmotionWords = emotions
    
    // Simple emotion scoring
    positiveWords := regexp.MustCompile(`\b(happy|excited|amazing|love|wonderful)\b`)
    negativeWords := regexp.MustCompile(`\b(sad|angry|disappointed|terrible|hate|awful)\b`)
    
    positive := len(positiveWords.FindAllString(strings.ToLower(text), -1))
    negative := len(negativeWords.FindAllString(strings.ToLower(text), -1))
    
    if positive+negative > 0 {
        features.EmotionScore = float64(positive-negative) / float64(positive+negative)
    }
    
    // Question and exclamation counts
    features.QuestionCount = len(tfe.questionPattern.FindAllString(text, -1))
    features.ExclamationCount = len(tfe.exclamationPattern.FindAllString(text, -1))
    
    // Vocabulary analysis
    uniqueWords := make(map[string]bool)
    longestWord := 0
    
    for _, word := range words {
        lowerWord := strings.ToLower(word)
        uniqueWords[lowerWord] = true
        if len(word) > longestWord {
            longestWord = len(word)
        }
    }
    
    features.UniqueWords = len(uniqueWords)
    features.LongestWord = longestWord
    
    if features.WordCount > 0 {
        features.VocabularyRichness = float64(features.UniqueWords) / float64(features.WordCount)
    }
    
    // Generate bigrams
    for i := 0; i < len(words)-1; i++ {
        bigram := strings.ToLower(words[i]) + " " + strings.ToLower(words[i+1])
        features.Bigrams[bigram]++
    }
    
    // Extract common patterns
    features.CommonPatterns = tfe.extractCommonPatterns(text)
    
    // Calculate complexity score
    features.ComplexityScore = tfe.calculateComplexity(features)
    features.ReadabilityLevel = tfe.getReadabilityLevel(features.ComplexityScore)
    
    return features
}

func (tfe *TextFeatureExtractor) extractCommonPatterns(text string) []string {
    patterns := []string{}
    
    // Common text patterns
    patternRegexes := map[string]*regexp.Regexp{
        "email_pattern":     regexp.MustCompile(`\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b`),
        "url_pattern":       regexp.MustCompile(`https?://[^\s]+`),
        "phone_pattern":     regexp.MustCompile(`\b\d{3}[-.]?\d{3}[-.]?\d{4}\b`),
        "date_pattern":      regexp.MustCompile(`\b\d{1,2}[/-]\d{1,2}[/-]\d{4}\b`),
        "money_pattern":     regexp.MustCompile(`\$\d+(?:,\d{3})*(?:\.\d{2})?`),
        "time_pattern":      regexp.MustCompile(`\b\d{1,2}:\d{2}\b`),
        "hashtag_pattern":   regexp.MustCompile(`#\w+`),
        "mention_pattern":   regexp.MustCompile(`@\w+`),
        "repeated_chars":    regexp.MustCompile(`(.)\1{2,}`),
        "all_caps_words":    regexp.MustCompile(`\b[A-Z]{3,}\b`),
    }
    
    for patternName, regex := range patternRegexes {
        if matches := regex.FindAllString(text, -1); len(matches) > 0 {
            patterns = append(patterns, patternName)
        }
    }
    
    return patterns
}

func (tfe *TextFeatureExtractor) calculateComplexity(features TextFeatures) float64 {
    score := 0.0
    
    // Factors that increase complexity
    score += features.AvgWordsPerSentence * 0.1
    score += float64(features.LongestWord) * 0.05
    score += features.VocabularyRichness * 10
    score += features.PunctuationRatio * 5
    
    // Factors that decrease complexity
    if features.EmotionScore > 0 {
        score -= 1.0 // Emotional language tends to be simpler
    }
    
    return score
}

func (tfe *TextFeatureExtractor) getReadabilityLevel(complexity float64) string {
    if complexity < 2.0 {
        return "Simple"
    } else if complexity < 4.0 {
        return "Moderate"
    } else if complexity < 6.0 {
        return "Complex"
    } else {
        return "Very Complex"
    }
}

func (tfe *TextFeatureExtractor) CompareTexts(text1, text2 string) map[string]float64 {
    features1 := tfe.ExtractFeatures(text1)
    features2 := tfe.ExtractFeatures(text2)
    
    comparison := make(map[string]float64)
    
    // Compare basic metrics
    comparison["word_count_ratio"] = float64(features1.WordCount) / float64(features2.WordCount)
    comparison["complexity_diff"] = features1.ComplexityScore - features2.ComplexityScore
    comparison["emotion_diff"] = features1.EmotionScore - features2.EmotionScore
    comparison["vocabulary_diff"] = features1.VocabularyRichness - features2.VocabularyRichness
    
    return comparison
}

func main() {
    extractor := NewTextFeatureExtractor()
    
    sampleTexts := []string{
        "Hello there! This is a simple test message. It contains basic words and punctuation.",
        
        "The INCREDIBLE results of our AMAZING research study show that 95% of participants experienced significant improvements! Contact us at research@example.com or call 555-123-4567. #research #amazing",
        
        "In the context of machine learning and natural language processing, feature extraction represents a fundamental paradigm for transforming unstructured textual data into quantifiable metrics suitable for algorithmic analysis.",
        
        "OMG this is sooooo terrible!!! I hate everything about this product. It's the WORST thing ever created. 😭😭😭",
    }
    
    fmt.Println("=== Text Feature Extraction for ML ===")
    
    for i, text := range sampleTexts {
        fmt.Printf("\n--- Text %d ---\n", i+1)
        fmt.Printf("Content: %.100s...\n", text)
        
        features := extractor.ExtractFeatures(text)
        
        fmt.Printf("\nBasic Features:\n")
        fmt.Printf("  Words: %d, Sentences: %d, Characters: %d\n", 
            features.WordCount, features.SentenceCount, features.CharacterCount)
        fmt.Printf("  Unique words: %d (%.2f%% richness)\n", 
            features.UniqueWords, features.VocabularyRichness*100)
        fmt.Printf("  Avg words/sentence: %.1f\n", features.AvgWordsPerSentence)
        fmt.Printf("  Longest word: %d characters\n", features.LongestWord)
        
        fmt.Printf("\nStyle Features:\n")
        fmt.Printf("  Capital ratio: %.3f\n", features.CapitalRatio)
        fmt.Printf("  Punctuation ratio: %.3f\n", features.PunctuationRatio)
        fmt.Printf("  Questions: %d, Exclamations: %d\n", 
            features.QuestionCount, features.ExclamationCount)
        
        fmt.Printf("\nSentiment Features:\n")
        fmt.Printf("  Emotion score: %.2f\n", features.EmotionScore)
        fmt.Printf("  Emotion words: %v\n", features.EmotionWords)
        
        fmt.Printf("\nComplexity Analysis:\n")
        fmt.Printf("  Complexity score: %.2f\n", features.ComplexityScore)
        fmt.Printf("  Readability: %s\n", features.ReadabilityLevel)
        
        if len(features.CommonPatterns) > 0 {
            fmt.Printf("\nDetected patterns: %v\n", features.CommonPatterns)
        }
        
        // Show top bigrams
        if len(features.Bigrams) > 0 {
            type bigramCount struct {
                bigram string
                count  int
            }
            
            var bigrams []bigramCount
            for bigram, count := range features.Bigrams {
                if count > 1 { // Only show repeated bigrams
                    bigrams = append(bigrams, bigramCount{bigram, count})
                }
            }
            
            sort.Slice(bigrams, func(i, j int) bool {
                return bigrams[i].count > bigrams[j].count
            })
            
            if len(bigrams) > 0 {
                fmt.Printf("\nTop bigrams:\n")
                for i, bg := range bigrams {
                    if i >= 3 { // Show top 3
                        break
                    }
                    fmt.Printf("  '%s': %d\n", bg.bigram, bg.count)
                }
            }
        }
    }
    
    // Text comparison
    fmt.Printf("\n=== Text Comparison ===\n")
    comparison := extractor.CompareTexts(sampleTexts[0], sampleTexts[2])
    
    fmt.Printf("Simple vs Complex text comparison:\n")
    for metric, value := range comparison {
        fmt.Printf("  %s: %.3f\n", metric, value)
    }
    
    fmt.Printf("\n=== ML Feature Vector Summary ===\n")
    fmt.Println("Extracted features suitable for:")
    fmt.Println("• Text classification (sentiment, topic, style)")
    fmt.Println("• Author attribution and stylometry")
    fmt.Println("• Content quality assessment")
    fmt.Println("• Spam detection and content filtering")
    fmt.Println("• Readability and complexity analysis")
    fmt.Println("• Language model training features")
}
```

Feature extraction for machine learning converts unstructured text into  
quantifiable metrics for training models, enabling sentiment analysis,  
text classification, and natural language understanding applications.  

## Geographic data extraction

Regular expressions extract geographic information including addresses,  
coordinates, and location references from textual data.  

```go
package main

import (
    "fmt"
    "regexp"
)

func extractCoordinates(text string) [][]string {
    // Matches latitude/longitude in decimal degrees
    coordPattern := regexp.MustCompile(`(-?\d{1,3}\.\d+),\s*(-?\d{1,3}\.\d+)`)
    return coordPattern.FindAllStringSubmatch(text, -1)
}

func extractZipCodes(text string) []string {
    // US ZIP codes (5 digits or 5+4 format)
    zipPattern := regexp.MustCompile(`\b\d{5}(?:-\d{4})?\b`)
    return zipPattern.FindAllString(text, -1)
}

func main() {
    locationText := "Visit our office at 37.7749, -122.4194 in San Francisco, CA 94102 or our branch at 40.7128, -74.0060 in New York, NY 10001"
    
    coords := extractCoordinates(locationText)
    fmt.Println("Coordinates found:")
    for _, coord := range coords {
        fmt.Printf("  Lat: %s, Long: %s\n", coord[1], coord[2])
    }
    
    zips := extractZipCodes(locationText)
    fmt.Printf("ZIP codes: %v\n", zips)
}
```

Geographic extraction identifies location data for mapping, delivery  
routing, and location-based services.  

## Social media parsing

Regular expressions parse social media content including hashtags,  
mentions, and social media specific formatting.  

```go
package main

import (
    "fmt"
    "regexp"
)

type SocialMediaParser struct {
    hashtagPattern  *regexp.Regexp
    mentionPattern  *regexp.Regexp
    urlPattern      *regexp.Regexp
    emojiPattern    *regexp.Regexp
}

func NewSocialMediaParser() *SocialMediaParser {
    return &SocialMediaParser{
        hashtagPattern: regexp.MustCompile(`#\w+`),
        mentionPattern: regexp.MustCompile(`@\w+`),
        urlPattern:     regexp.MustCompile(`https?://[^\s]+`),
        emojiPattern:   regexp.MustCompile(`[\x{1F600}-\x{1F64F}]|[\x{1F300}-\x{1F5FF}]`),
    }
}

func (smp *SocialMediaParser) ParsePost(post string) map[string][]string {
    return map[string][]string{
        "hashtags":  smp.hashtagPattern.FindAllString(post, -1),
        "mentions":  smp.mentionPattern.FindAllString(post, -1),
        "urls":      smp.urlPattern.FindAllString(post, -1),
        "emojis":    smp.emojiPattern.FindAllString(post, -1),
    }
}

func main() {
    parser := NewSocialMediaParser()
    
    post := "Amazing day at #TechConf2024! Thanks @speaker_john for the great talk. Check out https://example.com 🚀✨"
    
    elements := parser.ParsePost(post)
    for elemType, items := range elements {
        fmt.Printf("%s: %v\n", elemType, items)
    }
}
```

Social media parsing extracts engagement elements and metadata for  
analytics, content moderation, and social listening applications.  

## Document format detection

Regular expressions identify document formats and extract metadata  
from various file types and document structures.  

```go
package main

import (
    "fmt"
    "regexp"
    "strings"
)

type DocumentAnalyzer struct {
    markdownPattern *regexp.Regexp
    htmlPattern     *regexp.Regexp
    jsonPattern     *regexp.Regexp
    xmlPattern      *regexp.Regexp
    csvPattern      *regexp.Regexp
}

func NewDocumentAnalyzer() *DocumentAnalyzer {
    return &DocumentAnalyzer{
        markdownPattern: regexp.MustCompile(`^#{1,6}\s+.+|^\*.+\*$|^\[.+\]\(.+\)`),
        htmlPattern:     regexp.MustCompile(`<[^>]+>`),
        jsonPattern:     regexp.MustCompile(`^\s*\{|\s*\[\s*\{`),
        xmlPattern:      regexp.MustCompile(`<\?xml|<[a-zA-Z][^>]*>`),
        csvPattern:      regexp.MustCompile(`^[^,\n]*,`),
    }
}

func (da *DocumentAnalyzer) DetectFormat(content string) string {
    content = strings.TrimSpace(content)
    
    if da.xmlPattern.MatchString(content) {
        return "XML"
    }
    if da.htmlPattern.MatchString(content) {
        return "HTML"
    }
    if da.jsonPattern.MatchString(content) {
        return "JSON"
    }
    if da.markdownPattern.MatchString(content) {
        return "Markdown"
    }
    if da.csvPattern.MatchString(content) {
        return "CSV"
    }
    
    return "Plain Text"
}

func main() {
    analyzer := NewDocumentAnalyzer()
    
    samples := map[string]string{
        "# Hello World\nThis is **markdown**": "sample1",
        "<html><body>Hello</body></html>":       "sample2", 
        `{"name": "John", "age": 30}`:          "sample3",
        `name,age,city\nJohn,30,NYC`:           "sample4",
    }
    
    for content, name := range samples {
        format := analyzer.DetectFormat(content)
        fmt.Printf("%s: %s\n", name, format)
    }
}
```

Document format detection enables automatic processing workflows and  
content management systems to handle different document types  
appropriately.  

## Error message parsing

Regular expressions extract error information and stack traces from  
application logs and error reports for debugging and monitoring.  

```go
package main

import (
    "fmt"
    "regexp"
)

type ErrorParser struct {
    javaStackTrace  *regexp.Regexp
    pythonError     *regexp.Regexp
    goError         *regexp.Regexp
    sqlError        *regexp.Regexp
    httpError       *regexp.Regexp
}

func NewErrorParser() *ErrorParser {
    return &ErrorParser{
        javaStackTrace: regexp.MustCompile(`at\s+([a-zA-Z0-9.$_]+)\(([^)]+)\)`),
        pythonError:    regexp.MustCompile(`File\s+"([^"]+)",\s+line\s+(\d+),\s+in\s+(\w+)`),
        goError:        regexp.MustCompile(`([^:]+):(\d+):\s+(.+)`),
        sqlError:       regexp.MustCompile(`ERROR:\s+(.+?)\s+at\s+line\s+(\d+)`),
        httpError:      regexp.MustCompile(`HTTP\s+(\d{3})\s+(.+)`),
    }
}

func (ep *ErrorParser) ParseError(errorText string) map[string]string {
    result := make(map[string]string)
    
    if match := ep.httpError.FindStringSubmatch(errorText); len(match) > 2 {
        result["type"] = "HTTP Error"
        result["code"] = match[1]
        result["message"] = match[2]
    } else if match := ep.goError.FindStringSubmatch(errorText); len(match) > 3 {
        result["type"] = "Go Error"
        result["file"] = match[1]
        result["line"] = match[2]
        result["message"] = match[3]
    }
    
    return result
}

func main() {
    parser := NewErrorParser()
    
    errors := []string{
        "main.go:42: undefined: fmt.Printf",
        "HTTP 404 Not Found",
        "ERROR: syntax error at line 15",
    }
    
    for _, err := range errors {
        parsed := parser.ParseError(err)
        fmt.Printf("Error: %s\n", err)
        for key, value := range parsed {
            fmt.Printf("  %s: %s\n", key, value)
        }
        fmt.Println()
    }
}
```

Error parsing automates debugging workflows by extracting structured  
information from error messages and stack traces.  

## Regex testing and validation

This example demonstrates how to test regex patterns and validate their  
correctness for reliable pattern matching.  

```go
package main

import (
    "fmt"
    "regexp"
    "time"
)

type RegexTester struct {
    pattern *regexp.Regexp
    name    string
}

func NewRegexTester(pattern, name string) (*RegexTester, error) {
    compiled, err := regexp.Compile(pattern)
    if err != nil {
        return nil, err
    }
    
    return &RegexTester{
        pattern: compiled,
        name:    name,
    }, nil
}

func (rt *RegexTester) RunTests(testCases []TestCase) TestResults {
    results := TestResults{
        Pattern:    rt.pattern.String(),
        PatternName: rt.name,
        StartTime:  time.Now(),
    }
    
    for _, testCase := range testCases {
        matches := rt.pattern.MatchString(testCase.Input)
        passed := matches == testCase.ShouldMatch
        
        result := SingleResult{
            Input:       testCase.Input,
            Expected:    testCase.ShouldMatch,
            Actual:      matches,
            Passed:      passed,
            Description: testCase.Description,
        }
        
        results.Results = append(results.Results, result)
        
        if passed {
            results.PassedCount++
        } else {
            results.FailedCount++
        }
    }
    
    results.Duration = time.Since(results.StartTime)
    return results
}

type TestCase struct {
    Input       string
    ShouldMatch bool
    Description string
}

type SingleResult struct {
    Input       string
    Expected    bool
    Actual      bool
    Passed      bool
    Description string
}

type TestResults struct {
    Pattern     string
    PatternName string
    Results     []SingleResult
    PassedCount int
    FailedCount int
    StartTime   time.Time
    Duration    time.Duration
}

func (tr TestResults) PrintResults() {
    fmt.Printf("=== Regex Test Results: %s ===\n", tr.PatternName)
    fmt.Printf("Pattern: %s\n", tr.Pattern)
    fmt.Printf("Duration: %v\n\n", tr.Duration)
    
    for i, result := range tr.Results {
        status := "✅ PASS"
        if !result.Passed {
            status = "❌ FAIL"
        }
        
        fmt.Printf("%d. %s\n", i+1, status)
        fmt.Printf("   Input: '%s'\n", result.Input)
        fmt.Printf("   Expected: %t, Got: %t\n", result.Expected, result.Actual)
        fmt.Printf("   Description: %s\n\n", result.Description)
    }
    
    fmt.Printf("Summary: %d passed, %d failed\n", tr.PassedCount, tr.FailedCount)
}

func main() {
    // Test email validation pattern
    emailTester, err := NewRegexTester(
        `^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$`,
        "Email Validation",
    )
    if err != nil {
        fmt.Printf("Error creating tester: %v\n", err)
        return
    }
    
    emailTests := []TestCase{
        {"user@example.com", true, "Valid email"},
        {"test.email@domain.org", true, "Email with dot"},
        {"user+tag@example.com", true, "Email with plus"},
        {"invalid.email", false, "Missing @ symbol"},
        {"@domain.com", false, "Missing username"},
        {"user@", false, "Missing domain"},
        {"user@domain", false, "Missing TLD"},
        {"user@domain.c", false, "TLD too short"},
        {"", false, "Empty string"},
    }
    
    results := emailTester.RunTests(emailTests)
    results.PrintResults()
    
    // Test phone number pattern
    phoneTester, _ := NewRegexTester(
        `^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$`,
        "US Phone Number",
    )
    
    phoneTests := []TestCase{
        {"555-123-4567", true, "Dashed format"},
        {"(555) 123-4567", true, "Parentheses format"},
        {"555.123.4567", true, "Dotted format"},
        {"5551234567", true, "No separators"},
        {"555-123-456", false, "Too short"},
        {"555-123-45678", false, "Too long"},
        {"abc-123-4567", false, "Non-numeric"},
    }
    
    phoneResults := phoneTester.RunTests(phoneTests)
    phoneResults.PrintResults()
}
```

Regex testing ensures pattern reliability and helps identify edge cases  
and false positives in pattern matching logic.  

## Performance monitoring patterns

Regular expressions monitor application performance by extracting timing  
data, resource usage, and performance metrics from logs and monitoring  
data.  

```go
package main

import (
    "fmt"
    "regexp"
    "strconv"
    "strings"
)

type PerformanceMonitor struct {
    responseTimePattern *regexp.Regexp
    memoryUsagePattern  *regexp.Regexp
    cpuUsagePattern     *regexp.Regexp
    errorRatePattern    *regexp.Regexp
    throughputPattern   *regexp.Regexp
}

func NewPerformanceMonitor() *PerformanceMonitor {
    return &PerformanceMonitor{
        responseTimePattern: regexp.MustCompile(`response_time[=:](\d+(?:\.\d+)?)(?:ms|s)`),
        memoryUsagePattern:  regexp.MustCompile(`memory[=:](\d+(?:\.\d+)?)(?:MB|GB)`),
        cpuUsagePattern:     regexp.MustCompile(`cpu[=:](\d+(?:\.\d+)?)%`),
        errorRatePattern:    regexp.MustCompile(`errors[=:](\d+(?:\.\d+)?)%`),
        throughputPattern:   regexp.MustCompile(`(?:throughput|rps)[=:](\d+(?:\.\d+)?)`),
    }
}

type PerformanceMetrics struct {
    ResponseTimes []float64
    MemoryUsage   []float64
    CPUUsage      []float64
    ErrorRates    []float64
    Throughput    []float64
    AlertsTriggered []string
}

func (pm *PerformanceMonitor) AnalyzeLog(logData string) PerformanceMetrics {
    metrics := PerformanceMetrics{}
    
    // Extract response times
    responseMatches := pm.responseTimePattern.FindAllStringSubmatch(logData, -1)
    for _, match := range responseMatches {
        if value, err := strconv.ParseFloat(match[1], 64); err == nil {
            metrics.ResponseTimes = append(metrics.ResponseTimes, value)
        }
    }
    
    // Extract memory usage
    memoryMatches := pm.memoryUsagePattern.FindAllStringSubmatch(logData, -1)
    for _, match := range memoryMatches {
        if value, err := strconv.ParseFloat(match[1], 64); err == nil {
            metrics.MemoryUsage = append(metrics.MemoryUsage, value)
        }
    }
    
    // Extract CPU usage
    cpuMatches := pm.cpuUsagePattern.FindAllStringSubmatch(logData, -1)
    for _, match := range cpuMatches {
        if value, err := strconv.ParseFloat(match[1], 64); err == nil {
            metrics.CPUUsage = append(metrics.CPUUsage, value)
        }
    }
    
    // Check for performance alerts
    for _, cpu := range metrics.CPUUsage {
        if cpu > 80.0 {
            metrics.AlertsTriggered = append(metrics.AlertsTriggered, "High CPU usage detected")
            break
        }
    }
    
    for _, responseTime := range metrics.ResponseTimes {
        if responseTime > 1000.0 {
            metrics.AlertsTriggered = append(metrics.AlertsTriggered, "Slow response time detected")
            break
        }
    }
    
    return metrics
}

func calculateAverage(values []float64) float64 {
    if len(values) == 0 {
        return 0
    }
    
    sum := 0.0
    for _, value := range values {
        sum += value
    }
    
    return sum / float64(len(values))
}

func main() {
    monitor := NewPerformanceMonitor()
    
    logData := `
    2024-03-15T10:30:25 INFO response_time=245ms cpu=75.2% memory=512MB
    2024-03-15T10:30:26 WARN response_time=1250ms cpu=85.1% memory=768MB
    2024-03-15T10:30:27 INFO response_time=189ms cpu=72.8% memory=545MB
    2024-03-15T10:30:28 ERROR response_time=2100ms cpu=95.5% memory=1.2GB
    2024-03-15T10:30:29 INFO response_time=156ms cpu=68.9% memory=489MB
    `
    
    metrics := monitor.AnalyzeLog(logData)
    
    fmt.Println("=== Performance Metrics Analysis ===")
    
    if len(metrics.ResponseTimes) > 0 {
        avgResponse := calculateAverage(metrics.ResponseTimes)
        fmt.Printf("Response Times: %v\n", metrics.ResponseTimes)
        fmt.Printf("Average Response Time: %.1f ms\n", avgResponse)
    }
    
    if len(metrics.CPUUsage) > 0 {
        avgCPU := calculateAverage(metrics.CPUUsage)
        fmt.Printf("CPU Usage: %v\n", metrics.CPUUsage)
        fmt.Printf("Average CPU Usage: %.1f%%\n", avgCPU)
    }
    
    if len(metrics.MemoryUsage) > 0 {
        avgMemory := calculateAverage(metrics.MemoryUsage)
        fmt.Printf("Memory Usage: %v\n", metrics.MemoryUsage)
        fmt.Printf("Average Memory Usage: %.1f MB\n", avgMemory)
    }
    
    if len(metrics.AlertsTriggered) > 0 {
        fmt.Printf("\nAlerts Triggered:\n")
        for _, alert := range metrics.AlertsTriggered {
            fmt.Printf("  ⚠️  %s\n", alert)
        }
    } else {
        fmt.Printf("\n✅ No performance alerts triggered\n")
    }
    
    // Performance summary
    fmt.Println("\n=== Performance Summary ===")
    if len(metrics.ResponseTimes) > 0 {
        avgResponse := calculateAverage(metrics.ResponseTimes)
        if avgResponse < 200 {
            fmt.Println("✅ Response time: Excellent")
        } else if avgResponse < 500 {
            fmt.Println("⚠️  Response time: Good")
        } else {
            fmt.Println("❌ Response time: Needs attention")
        }
    }
    
    if len(metrics.CPUUsage) > 0 {
        avgCPU := calculateAverage(metrics.CPUUsage)
        if avgCPU < 70 {
            fmt.Println("✅ CPU usage: Normal")
        } else if avgCPU < 85 {
            fmt.Println("⚠️  CPU usage: Elevated")
        } else {
            fmt.Println("❌ CPU usage: Critical")
        }
    }
}
```

Performance monitoring patterns enable automated analysis of system  
metrics, alerting, and performance trend analysis for maintaining  
application health and reliability.  