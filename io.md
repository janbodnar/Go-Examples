# Input/Output Operations

Input/Output (IO) operations are fundamental to most programs, enabling data  
exchange between applications and their environment. Go provides a rich and  
well-designed IO system that emphasizes simplicity, composability, and  
performance. The core of Go's IO system revolves around interfaces, particularly  
the `io.Reader` and `io.Writer` interfaces, which provide a simple and  
universal abstraction for reading and writing data.

Go's IO design philosophy centers on the principle of composition through  
interfaces. Rather than providing monolithic classes with many methods, Go  
defines small, focused interfaces that can be combined to create powerful  
abstractions. This approach makes the IO system both flexible and predictable,  
as any type implementing the basic interfaces can work with the entire  
ecosystem of IO utilities and functions.

The `io` package forms the heart of Go's IO system, defining fundamental  
interfaces like `Reader`, `Writer`, `Closer`, and `Seeker`. These interfaces  
are implemented by various types throughout the standard library, from files  
and network connections to in-memory buffers and compression streams. This  
consistent interface design allows you to write generic functions that work  
with any IO source or destination.

The `bufio` package builds on top of the basic IO interfaces to provide  
buffered IO operations, which can significantly improve performance when  
dealing with many small reads or writes. The `ioutil` package (deprecated  
in favor of `io` and `os` packages in newer versions) historically provided  
convenient utility functions for common IO tasks.

Go's IO system handles both text and binary data seamlessly, with proper  
support for Unicode and UTF-8 encoding. The language's byte slice type  
(`[]byte`) serves as the fundamental unit for IO operations, providing  
efficiency and flexibility. String conversion is straightforward and  
automatic in many contexts.

Error handling is explicit and consistent throughout Go's IO system. Most  
IO operations return an error as their last return value, following Go's  
explicit error handling philosophy. Special errors like `io.EOF` indicate  
end-of-file conditions, while other errors represent various failure modes  
like permission issues or network problems.

The standard library provides extensive support for various IO scenarios  
including file operations, network communication, data serialization,  
compression, and more. Third-party packages often integrate seamlessly  
with Go's IO interfaces, making it easy to extend functionality while  
maintaining compatibility with existing code.


## Basic Reader interface

The `io.Reader` interface is the most fundamental interface in Go's IO system.  
It defines a single method for reading data into a byte slice.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    text := "hello there from Go IO system"
    reader := strings.NewReader(text)
    
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
}
```

The `io.Reader` interface requires only one method: `Read([]byte) (int, error)`.  
This simple contract enables any type to be used as a data source. The method  
returns the number of bytes read and an error if something goes wrong.  

## Basic Writer interface

The `io.Writer` interface provides the foundation for writing data. It defines  
a single method for writing byte slices to a destination.  

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    var buffer strings.Builder
    
    data := []byte("hello there from Writer interface")
    n, err := buffer.Write(data)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    
    fmt.Printf("Wrote %d bytes\n", n)
    fmt.Printf("Buffer contents: %s\n", buffer.String())
    
    // Write to standard output
    fmt.Fprint(os.Stdout, "Writing directly to stdout\n")
}
```

The `io.Writer` interface defines `Write([]byte) (int, error)`. Any type  
implementing this method can be used as a destination for data. Writers  
are used throughout Go's standard library for output operations.  

## Reading from files

File reading is one of the most common IO operations. Go provides several  
ways to read file contents, from reading entire files to streaming data.  

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    // Create a test file
    testData := "This is test content for file reading examples.\nSecond line of content."
    err := os.WriteFile("test.txt", []byte(testData), 0644)
    if err != nil {
        fmt.Printf("Error creating test file: %v\n", err)
        return
    }
    defer os.Remove("test.txt")
    
    // Open and read the file
    file, err := os.Open("test.txt")
    if err != nil {
        fmt.Printf("Error opening file: %v\n", err)
        return
    }
    defer file.Close()
    
    // Read in chunks
    buffer := make([]byte, 16)
    for {
        n, err := file.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading file: %v\n", err)
            break
        }
        fmt.Printf("Read %d bytes: %q\n", n, buffer[:n])
    }
}
```

File reading demonstrates the practical application of the `io.Reader` interface.  
Files implement `io.Reader`, allowing them to work with any function expecting  
a reader. Always remember to close files using `defer` for proper resource cleanup.  

## Writing to files

File writing operations demonstrate the `io.Writer` interface in practice.  
Go provides several approaches for writing data to files.  

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    filename := "output.txt"
    
    // Create and write to file
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(filename) // Cleanup
    
    // Write data
    data := []byte("hello there from file writing\nSecond line of content\n")
    n, err := file.Write(data)
    if err != nil {
        fmt.Printf("Error writing to file: %v\n", err)
        return
    }
    
    fmt.Printf("Wrote %d bytes to %s\n", n, filename)
    
    // Append to file
    file2, err := os.OpenFile(filename, os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        fmt.Printf("Error opening file for append: %v\n", err)
        return
    }
    defer file2.Close()
    
    appendData := []byte("Appended content\n")
    n, err = file2.Write(appendData)
    if err != nil {
        fmt.Printf("Error appending to file: %v\n", err)
        return
    }
    
    fmt.Printf("Appended %d bytes to %s\n", n, filename)
}
```

File writing shows how the `io.Writer` interface works with real file systems.  
The `os.Create` function creates a new file, while `os.OpenFile` provides  
more control over file opening modes including append operations.  

## Reading entire files

For smaller files, reading the entire content at once is often more convenient  
than streaming. Go provides utility functions for this common pattern.  

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    testData := "Complete file content\nwith multiple lines\nfor reading at once"
    filename := "complete.txt"
    
    // Write test data
    err := os.WriteFile(filename, []byte(testData), 0644)
    if err != nil {
        fmt.Printf("Error writing file: %v\n", err)
        return
    }
    defer os.Remove(filename)
    
    // Read entire file
    content, err := os.ReadFile(filename)
    if err != nil {
        fmt.Printf("Error reading file: %v\n", err)
        return
    }
    
    fmt.Printf("File size: %d bytes\n", len(content))
    fmt.Printf("Content:\n%s\n", string(content))
    
    // Alternative using io.ReadAll with file handle
    file, err := os.Open(filename)
    if err != nil {
        fmt.Printf("Error opening file: %v\n", err)
        return
    }
    defer file.Close()
    
    content2, err := io.ReadAll(file)
    if err != nil {
        fmt.Printf("Error reading with ReadAll: %v\n", err)
        return
    }
    
    fmt.Printf("ReadAll result size: %d bytes\n", len(content2))
}
```

## Writing entire files

Writing complete data to files is simplified with utility functions that  
handle file creation, writing, and closing automatically.  

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    filename := "complete_write.txt"
    content := "This is complete file content\nwritten in one operation\nwith proper permissions"
    
    // Write entire content to file
    err := os.WriteFile(filename, []byte(content), 0644)
    if err != nil {
        fmt.Printf("Error writing file: %v\n", err)
        return
    }
    defer os.Remove(filename)
    
    fmt.Printf("Successfully wrote content to %s\n", filename)
    
    // Verify by reading back
    readContent, err := os.ReadFile(filename)
    if err != nil {
        fmt.Printf("Error reading back: %v\n", err)
        return
    }
    
    fmt.Printf("File contains %d bytes\n", len(readContent))
    fmt.Printf("Content matches: %t\n", string(readContent) == content)
    
    // Write with different permissions
    filename2 := "restricted.txt"
    err = os.WriteFile(filename2, []byte("Restricted content"), 0600)
    if err != nil {
        fmt.Printf("Error writing restricted file: %v\n", err)
        return
    }
    defer os.Remove(filename2)
    
    // Check file info
    info, err := os.Stat(filename2)
    if err != nil {
        fmt.Printf("Error getting file info: %v\n", err)
        return
    }
    
    fmt.Printf("File mode: %v\n", info.Mode())
}
```

## String readers

String readers allow you to treat strings as IO sources, implementing the  
`io.Reader` interface for string data.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "hello there from string reader\nwith multiple lines\nand various content"
    reader := strings.NewReader(content)
    
    // Read in small chunks
    buffer := make([]byte, 10)
    position := 0
    
    for {
        n, err := reader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error: %v\n", err)
            break
        }
        
        fmt.Printf("Position %d: read %d bytes: %q\n", position, n, buffer[:n])
        position += n
    }
    
    // Reset and read all at once
    reader.Reset(content)
    allData, err := io.ReadAll(reader)
    if err != nil {
        fmt.Printf("Error reading all: %v\n", err)
        return
    }
    
    fmt.Printf("\nComplete content (%d bytes): %s\n", len(allData), string(allData))
    
    // Demonstrate seeking
    reader.Reset(content)
    reader.Seek(6, io.SeekStart) // Skip "hello "
    
    remaining, err := io.ReadAll(reader)
    if err != nil {
        fmt.Printf("Error reading after seek: %v\n", err)
        return
    }
    
    fmt.Printf("After seeking: %s\n", string(remaining))
}
```

## Byte readers

Byte readers work with byte slices, providing an in-memory implementation  
of the `io.Reader` interface for byte data.  

```go
package main

import (
    "bytes"
    "fmt"
    "io"
)

func main() {
    data := []byte("hello there from byte reader\nwith binary data: \x00\x01\x02\xFF")
    reader := bytes.NewReader(data)
    
    // Read header
    header := make([]byte, 5)
    n, err := reader.Read(header)
    if err != nil {
        fmt.Printf("Error reading header: %v\n", err)
        return
    }
    
    fmt.Printf("Header (%d bytes): %q\n", n, header)
    
    // Read next word
    word := make([]byte, 5)
    n, err = reader.Read(word)
    if err != nil {
        fmt.Printf("Error reading word: %v\n", err)
        return
    }
    
    fmt.Printf("Word (%d bytes): %q\n", n, word)
    
    // Seek to binary data
    reader.Seek(-4, io.SeekEnd) // Go to 4 bytes before end
    
    binaryData := make([]byte, 4)
    n, err = reader.Read(binaryData)
    if err != nil && err != io.EOF {
        fmt.Printf("Error reading binary data: %v\n", err)
        return
    }
    
    fmt.Printf("Binary data (%d bytes): %v\n", n, binaryData)
    
    // Reset and get size
    reader.Reset(data)
    fmt.Printf("Reader size: %d bytes\n", reader.Size())
    
    // Read with offset
    reader.Seek(12, io.SeekStart)
    position, err := reader.Seek(0, io.SeekCurrent)
    if err != nil {
        fmt.Printf("Error getting position: %v\n", err)
        return
    }
    fmt.Printf("Current position: %d\n", position)
}
```

## String builders

String builders provide efficient string construction using the `io.Writer`  
interface, avoiding repeated string allocations.  

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var builder strings.Builder
    
    // Pre-allocate capacity for efficiency
    builder.Grow(100)
    
    // Write different types of data
    builder.WriteString("hello there")
    builder.WriteString(" from ")
    builder.WriteString("string builder\n")
    
    // Write bytes
    builder.Write([]byte("Binary data: "))
    builder.Write([]byte{0x48, 0x65, 0x6C, 0x6C, 0x6F}) // "Hello" in hex
    builder.WriteByte('\n')
    
    // Write rune
    builder.WriteRune('🚀')
    builder.WriteString(" Go programming\n")
    
    // Write formatted data
    fmt.Fprintf(&builder, "Number: %d, Float: %.2f\n", 42, 3.14159)
    
    result := builder.String()
    fmt.Printf("Built string (%d bytes):\n%s", len(result), result)
    
    // Demonstrate efficiency
    builder.Reset()
    
    for i := 0; i < 5; i++ {
        fmt.Fprintf(&builder, "Line %d: %s\n", i+1, "content")
    }
    
    fmt.Printf("\nLoop result:\n%s", builder.String())
    
    // Show capacity and length
    fmt.Printf("Length: %d, Capacity: %d\n", builder.Len(), builder.Cap())
}
```

## Byte buffers

Byte buffers provide a resizable buffer of bytes that implements both  
`io.Reader` and `io.Writer` interfaces.  

```go
package main

import (
    "bytes"
    "fmt"
    "io"
)

func main() {
    var buffer bytes.Buffer
    
    // Write to buffer
    buffer.WriteString("hello there from byte buffer\n")
    buffer.Write([]byte("Binary data: "))
    buffer.Write([]byte{0x01, 0x02, 0x03, 0x04})
    buffer.WriteByte('\n')
    
    fmt.Printf("Buffer length: %d bytes\n", buffer.Len())
    fmt.Printf("Buffer capacity: %d bytes\n", buffer.Cap())
    
    // Read from buffer
    readData := make([]byte, 10)
    n, err := buffer.Read(readData)
    if err != nil {
        fmt.Printf("Error reading: %v\n", err)
        return
    }
    
    fmt.Printf("Read %d bytes: %q\n", n, readData[:n])
    fmt.Printf("Remaining in buffer: %d bytes\n", buffer.Len())
    
    // Read a line
    line, err := buffer.ReadBytes('\n')
    if err != nil {
        fmt.Printf("Error reading line: %v\n", err)
        return
    }
    
    fmt.Printf("Read line: %q\n", line)
    
    // Write more data
    buffer.WriteString("Additional content")
    
    // Read all remaining
    remaining := buffer.Bytes()
    fmt.Printf("Remaining data: %q\n", remaining)
    
    // Reset and demonstrate with initial data
    initialData := []byte("Pre-filled buffer content")
    buffer2 := bytes.NewBuffer(initialData)
    
    fmt.Printf("New buffer content: %s\n", buffer2.String())
    
    // Truncate buffer
    buffer2.Truncate(10)
    fmt.Printf("After truncate: %s\n", buffer2.String())
}
```

## Buffered readers

Buffered readers improve performance by reducing the number of system calls  
when reading data in small chunks.  

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    content := `First line of content
Second line with more data  
Third line: hello there
Fourth line: final content`
    
    reader := strings.NewReader(content)
    bufferedReader := bufio.NewReader(reader)
    
    // Read line by line
    lineNum := 1
    for {
        line, err := bufferedReader.ReadLine()
        if err != nil {
            if err.Error() == "EOF" {
                break
            }
            fmt.Printf("Error reading line: %v\n", err)
            break
        }
        
        fmt.Printf("Line %d: %s\n", lineNum, string(line))
        lineNum++
    }
    
    // Reset and read with different methods
    reader.Reset(content)
    bufferedReader.Reset(reader)
    
    fmt.Println("\nReading with ReadString:")
    for {
        line, err := bufferedReader.ReadString('\n')
        if err != nil {
            if err.Error() == "EOF" && line != "" {
                fmt.Printf("Last line: %s", line)
            }
            break
        }
        fmt.Printf("Line: %s", line)
    }
    
    // Demonstrate buffer size
    reader.Reset(content)
    customBuffered := bufio.NewReaderSize(reader, 8) // Small buffer
    
    fmt.Printf("\nCustom buffer size: %d bytes\n", customBuffered.Size())
    
    // Peek at data without consuming
    peek, err := customBuffered.Peek(5)
    if err != nil {
        fmt.Printf("Error peeking: %v\n", err)
        return
    }
    fmt.Printf("Peeked data: %q\n", peek)
    
    // Read first byte
    b, err := customBuffered.ReadByte()
    if err != nil {
        fmt.Printf("Error reading byte: %v\n", err)
        return
    }
    fmt.Printf("First byte: %c\n", b)
}
```

## Buffered writers

Buffered writers collect data in memory before writing to the underlying  
destination, improving performance for many small writes.  

```go
package main

import (
    "bufio"
    "fmt"
    "os"
    "strings"
)

func main() {
    var buffer strings.Builder
    writer := bufio.NewWriter(&buffer)
    
    // Write data to buffered writer
    writer.WriteString("hello there from buffered writer\n")
    writer.WriteString("Second line of content\n")
    
    // Data is in buffer, not yet written
    fmt.Printf("Buffer content before flush: %q\n", buffer.String())
    
    // Flush to actually write
    err := writer.Flush()
    if err != nil {
        fmt.Printf("Error flushing: %v\n", err)
        return
    }
    
    fmt.Printf("Buffer content after flush: %q\n", buffer.String())
    
    // Write more data
    fmt.Fprintf(writer, "Formatted: %d, %.2f\n", 42, 3.14159)
    writer.WriteByte('X')
    writer.WriteRune('🎯')
    writer.WriteString("\n")
    
    // Check buffered amount
    fmt.Printf("Buffered bytes: %d\n", writer.Buffered())
    
    writer.Flush()
    fmt.Printf("Final content:\n%s", buffer.String())
    
    // Demonstrate with file
    filename := "buffered_output.txt"
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(filename)
    
    fileWriter := bufio.NewWriter(file)
    
    // Write lots of small data
    for i := 0; i < 5; i++ {
        fmt.Fprintf(fileWriter, "Line %d: content\n", i+1)
    }
    
    // Must flush before closing
    fileWriter.Flush()
    
    // Verify file content
    content, err := os.ReadFile(filename)
    if err != nil {
        fmt.Printf("Error reading file: %v\n", err)
        return
    }
    
    fmt.Printf("File content:\n%s", string(content))
}
```

## Scanner for line reading

The scanner provides a convenient way to read input line by line or by  
other delimiters with automatic buffer management.  

```go
package main

import (
    "bufio"
    "fmt"
    "strings"
)

func main() {
    content := `hello there scanner
line two with content
line three: more data
final line`
    
    reader := strings.NewReader(content)
    scanner := bufio.NewScanner(reader)
    
    // Scan line by line
    lineNum := 1
    for scanner.Scan() {
        line := scanner.Text()
        fmt.Printf("Line %d: %s\n", lineNum, line)
        lineNum++
    }
    
    if err := scanner.Err(); err != nil {
        fmt.Printf("Scanner error: %v\n", err)
    }
    
    // Scan words
    reader.Reset(content)
    wordScanner := bufio.NewScanner(reader)
    wordScanner.Split(bufio.ScanWords)
    
    fmt.Println("\nScanning words:")
    wordNum := 1
    for wordScanner.Scan() {
        word := wordScanner.Text()
        fmt.Printf("Word %d: %s\n", wordNum, word)
        wordNum++
        if wordNum > 10 { // Limit output
            break
        }
    }
    
    // Custom split function
    reader.Reset("hello,there,scanner,test,data")
    csvScanner := bufio.NewScanner(reader)
    csvScanner.Split(func(data []byte, atEOF bool) (advance int, token []byte, err error) {
        if atEOF && len(data) == 0 {
            return 0, nil, nil
        }
        
        if i := strings.IndexByte(string(data), ','); i >= 0 {
            return i + 1, data[0:i], nil
        }
        
        if atEOF {
            return len(data), data, nil
        }
        
        return 0, nil, nil
    })
    
    fmt.Println("\nScanning CSV:")
    for csvScanner.Scan() {
        fmt.Printf("Field: %s\n", csvScanner.Text())
    }
}
```

## IO copy operations

Copy operations efficiently transfer data between readers and writers  
without loading everything into memory.  

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    // Create source data
    sourceData := "hello there from copy operations\nwith multiple lines\nand substantial content\nfor copying examples"
    source := strings.NewReader(sourceData)
    
    // Copy to string builder
    var dest strings.Builder
    
    n, err := io.Copy(&dest, source)
    if err != nil {
        fmt.Printf("Error copying: %v\n", err)
        return
    }
    
    fmt.Printf("Copied %d bytes\n", n)
    fmt.Printf("Destination content: %s\n", dest.String())
    
    // Copy with limit
    source.Reset(sourceData)
    dest.Reset()
    
    limited := io.LimitReader(source, 20) // Only first 20 bytes
    n, err = io.Copy(&dest, limited)
    if err != nil {
        fmt.Printf("Error copying limited: %v\n", err)
        return
    }
    
    fmt.Printf("Limited copy: %d bytes: %q\n", n, dest.String())
    
    // Copy between files
    srcFile := "source.txt"
    dstFile := "destination.txt"
    
    err = os.WriteFile(srcFile, []byte(sourceData), 0644)
    if err != nil {
        fmt.Printf("Error creating source file: %v\n", err)
        return
    }
    defer os.Remove(srcFile)
    defer os.Remove(dstFile)
    
    // Open source file
    src, err := os.Open(srcFile)
    if err != nil {
        fmt.Printf("Error opening source: %v\n", err)
        return
    }
    defer src.Close()
    
    // Create destination file
    dst, err := os.Create(dstFile)
    if err != nil {
        fmt.Printf("Error creating destination: %v\n", err)
        return
    }
    defer dst.Close()
    
    // Copy file to file
    n, err = io.Copy(dst, src)
    if err != nil {
        fmt.Printf("Error copying file: %v\n", err)
        return
    }
    
    fmt.Printf("File copy: %d bytes\n", n)
    
    // Verify copy
    copiedContent, err := os.ReadFile(dstFile)
    if err != nil {
        fmt.Printf("Error reading copied file: %v\n", err)
        return
    }
    
    fmt.Printf("Copy successful: %t\n", string(copiedContent) == sourceData)
}
```

## Copy with buffer

CopyBuffer allows you to provide your own buffer for copy operations,  
giving control over memory usage and performance characteristics.  

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
    "time"
)

func main() {
    // Create large source data
    var sourceBuilder strings.Builder
    for i := 0; i < 1000; i++ {
        fmt.Fprintf(&sourceBuilder, "Line %d: hello there with substantial content for testing buffer performance\n", i)
    }
    sourceData := sourceBuilder.String()
    
    source := strings.NewReader(sourceData)
    var dest strings.Builder
    
    // Custom buffer for copying
    buffer := make([]byte, 1024) // 1KB buffer
    
    start := time.Now()
    n, err := io.CopyBuffer(&dest, source, buffer)
    duration := time.Since(start)
    
    if err != nil {
        fmt.Printf("Error copying with buffer: %v\n", err)
        return
    }
    
    fmt.Printf("Copied %d bytes in %v\n", n, duration)
    fmt.Printf("Copy successful: %t\n", dest.String() == sourceData)
    
    // Compare with different buffer sizes
    bufferSizes := []int{64, 512, 2048, 8192}
    
    for _, size := range bufferSizes {
        source.Reset(sourceData)
        dest.Reset()
        
        customBuffer := make([]byte, size)
        
        start := time.Now()
        n, err := io.CopyBuffer(&dest, source, customBuffer)
        duration := time.Since(start)
        
        if err != nil {
            fmt.Printf("Error with %d byte buffer: %v\n", size, err)
            continue
        }
        
        fmt.Printf("Buffer size %d: copied %d bytes in %v\n", size, n, duration)
    }
    
    // File copy with custom buffer
    srcFile := "large_source.txt"
    dstFile := "large_dest.txt"
    
    err = os.WriteFile(srcFile, []byte(sourceData), 0644)
    if err != nil {
        fmt.Printf("Error creating large source: %v\n", err)
        return
    }
    defer os.Remove(srcFile)
    defer os.Remove(dstFile)
    
    src, err := os.Open(srcFile)
    if err != nil {
        fmt.Printf("Error opening source: %v\n", err)
        return
    }
    defer src.Close()
    
    dst, err := os.Create(dstFile)
    if err != nil {
        fmt.Printf("Error creating destination: %v\n", err)
        return
    }
    defer dst.Close()
    
    // Use large buffer for file copy
    fileBuffer := make([]byte, 32768) // 32KB buffer
    
    start = time.Now()
    n, err = io.CopyBuffer(dst, src, fileBuffer)
    duration = time.Since(start)
    
    if err != nil {
        fmt.Printf("Error copying file: %v\n", err)
        return
    }
    
    fmt.Printf("File copy with 32KB buffer: %d bytes in %v\n", n, duration)
}
```

## Multi-reader

Multi-reader concatenates multiple readers into a single reader,  
reading from each in sequence.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    // Create multiple readers
    reader1 := strings.NewReader("hello there ")
    reader2 := strings.NewReader("from multi-reader ")
    reader3 := strings.NewReader("concatenation example\n")
    reader4 := strings.NewReader("with multiple sources")
    
    // Combine into single reader
    multiReader := io.MultiReader(reader1, reader2, reader3, reader4)
    
    // Read from combined reader
    buffer := make([]byte, 16)
    position := 0
    
    fmt.Println("Reading from multi-reader:")
    for {
        n, err := multiReader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading: %v\n", err)
            break
        }
        
        fmt.Printf("Position %d: %q\n", position, buffer[:n])
        position += n
    }
    
    // Read all at once
    reader1.Reset("First part ")
    reader2.Reset("Second part ")
    reader3.Reset("Third part\n")
    
    multiReader2 := io.MultiReader(reader1, reader2, reader3)
    
    allData, err := io.ReadAll(multiReader2)
    if err != nil {
        fmt.Printf("Error reading all: %v\n", err)
        return
    }
    
    fmt.Printf("\nCombined content: %s", string(allData))
    
    // Practical example: concatenating files
    header := strings.NewReader("=== File Header ===\n")
    content := strings.NewReader("Main file content goes here\nWith multiple lines\n")
    footer := strings.NewReader("=== File Footer ===\n")
    
    document := io.MultiReader(header, content, footer)
    
    var result strings.Builder
    _, err = io.Copy(&result, document)
    if err != nil {
        fmt.Printf("Error copying document: %v\n", err)
        return
    }
    
    fmt.Printf("\nDocument structure:\n%s", result.String())
    
    // Dynamic multi-reader with slice
    parts := []string{
        "Part 1: Introduction\n",
        "Part 2: Main content\n", 
        "Part 3: Conclusion\n",
    }
    
    readers := make([]io.Reader, len(parts))
    for i, part := range parts {
        readers[i] = strings.NewReader(part)
    }
    
    dynamicMulti := io.MultiReader(readers...)
    
    finalContent, err := io.ReadAll(dynamicMulti)
    if err != nil {
        fmt.Printf("Error reading dynamic multi: %v\n", err)
        return
    }
    
    fmt.Printf("\nDynamic multi-reader:\n%s", string(finalContent))
}
```

## Multi-writer

Multi-writer duplicates writes to multiple destinations simultaneously,  
useful for logging to multiple outputs or creating backups.  

```go
package main

import (
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    // Create multiple writers
    var buffer1 strings.Builder
    var buffer2 strings.Builder
    var buffer3 strings.Builder
    
    // Combine into single writer
    multiWriter := io.MultiWriter(&buffer1, &buffer2, &buffer3)
    
    // Write to all destinations simultaneously
    data := "hello there from multi-writer\nwith simultaneous output\n"
    n, err := multiWriter.Write([]byte(data))
    if err != nil {
        fmt.Printf("Error writing: %v\n", err)
        return
    }
    
    fmt.Printf("Wrote %d bytes to all writers\n", n)
    fmt.Printf("Buffer1: %s", buffer1.String())
    fmt.Printf("Buffer2: %s", buffer2.String())
    fmt.Printf("Buffer3: %s", buffer3.String())
    
    // Practical example: log to file and stdout
    logFile := "app.log"
    file, err := os.Create(logFile)
    if err != nil {
        fmt.Printf("Error creating log file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(logFile)
    
    // Write to both file and stdout
    logger := io.MultiWriter(file, os.Stdout)
    
    fmt.Fprintf(logger, "Log entry 1: Application started\n")
    fmt.Fprintf(logger, "Log entry 2: Processing data\n")
    fmt.Fprintf(logger, "Log entry 3: Operation completed\n")
    
    // Verify file content
    content, err := os.ReadFile(logFile)
    if err != nil {
        fmt.Printf("Error reading log file: %v\n", err)
        return
    }
    
    fmt.Printf("\nLog file contains %d bytes\n", len(content))
    
    // Multiple file outputs
    file1 := "output1.txt"
    file2 := "output2.txt"
    
    f1, err := os.Create(file1)
    if err != nil {
        fmt.Printf("Error creating file1: %v\n", err)
        return
    }
    defer f1.Close()
    defer os.Remove(file1)
    
    f2, err := os.Create(file2)
    if err != nil {
        fmt.Printf("Error creating file2: %v\n", err)
        return
    }
    defer f2.Close()
    defer os.Remove(file2)
    
    multiFile := io.MultiWriter(f1, f2)
    
    backup := "Important data that needs backup\nSecond line of critical information\n"
    n, err = multiFile.Write([]byte(backup))
    if err != nil {
        fmt.Printf("Error writing backup: %v\n", err)
        return
    }
    
    fmt.Printf("Backup: wrote %d bytes to both files\n", n)
    
    // Verify both files have same content
    content1, _ := os.ReadFile(file1)
    content2, _ := os.ReadFile(file2)
    
    fmt.Printf("Backup verification: files match = %t\n", string(content1) == string(content2))
}
```
```
