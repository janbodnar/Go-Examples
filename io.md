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

## Pipe operations

Pipes create synchronized communication between a reader and writer,  
useful for streaming data between goroutines.  

```go
package main

import (
    "fmt"
    "io"
    "time"
)

func main() {
    // Create a pipe
    reader, writer := io.Pipe()
    
    // Start a goroutine to write data
    go func() {
        defer writer.Close()
        
        for i := 1; i <= 5; i++ {
            data := fmt.Sprintf("Message %d: hello there from pipe\n", i)
            n, err := writer.Write([]byte(data))
            if err != nil {
                fmt.Printf("Error writing to pipe: %v\n", err)
                return
            }
            fmt.Printf("Wrote %d bytes\n", n)
            time.Sleep(100 * time.Millisecond)
        }
    }()
    
    // Read from pipe
    buffer := make([]byte, 64)
    for {
        n, err := reader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading from pipe: %v\n", err)
            break
        }
        
        fmt.Printf("Read: %s", string(buffer[:n]))
    }
    
    // Example with error handling
    reader2, writer2 := io.Pipe()
    
    go func() {
        defer writer2.Close()
        
        // Simulate writing and then an error
        writer2.Write([]byte("Data before error\n"))
        writer2.CloseWithError(fmt.Errorf("simulated write error"))
    }()
    
    // Read until error
    data, err := io.ReadAll(reader2)
    if err != nil {
        fmt.Printf("Pipe error: %v\n", err)
    }
    fmt.Printf("Data before error: %s", string(data))
    
    // Pipe for data transformation
    transformReader, transformWriter := io.Pipe()
    
    go func() {
        defer transformWriter.Close()
        
        inputData := []string{
            "hello there",
            "from pipe transformation",
            "with multiple messages",
        }
        
        for _, msg := range inputData {
            transformWriter.Write([]byte(msg + "\n"))
            time.Sleep(50 * time.Millisecond)
        }
    }()
    
    // Read and transform data
    fmt.Println("\nTransformed data:")
    buffer = make([]byte, 32)
    messageNum := 1
    
    for {
        n, err := transformReader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Transform error: %v\n", err)
            break
        }
        
        fmt.Printf("Message %d: %s", messageNum, string(buffer[:n]))
        messageNum++
    }
}
```

## Limited reader

Limited reader restricts the amount of data that can be read from  
an underlying reader, useful for processing data in chunks.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "hello there from limited reader with substantial content that will be truncated at specified limit"
    source := strings.NewReader(content)
    
    // Limit to first 20 bytes
    limited := io.LimitReader(source, 20)
    
    limitedData, err := io.ReadAll(limited)
    if err != nil {
        fmt.Printf("Error reading limited: %v\n", err)
        return
    }
    
    fmt.Printf("Limited read (20 bytes): %q\n", string(limitedData))
    fmt.Printf("Original length: %d, limited length: %d\n", len(content), len(limitedData))
    
    // Read remaining data from original source
    remaining, err := io.ReadAll(source)
    if err != nil {
        fmt.Printf("Error reading remaining: %v\n", err)
        return
    }
    
    fmt.Printf("Remaining data: %q\n", string(remaining))
    
    // Multiple limited readers from same source
    source.Reset(content)
    
    chunk1 := io.LimitReader(source, 10)
    chunk2 := io.LimitReader(source, 15) 
    chunk3 := io.LimitReader(source, 20)
    
    data1, _ := io.ReadAll(chunk1)
    data2, _ := io.ReadAll(chunk2)
    data3, _ := io.ReadAll(chunk3)
    
    fmt.Printf("Chunk 1 (10 bytes): %q\n", string(data1))
    fmt.Printf("Chunk 2 (15 bytes): %q\n", string(data2))
    fmt.Printf("Chunk 3 (20 bytes): %q\n", string(data3))
    
    // Processing file in chunks
    largeContent := strings.Repeat("Line of content for chunk processing\n", 10)
    largeSource := strings.NewReader(largeContent)
    
    chunkSize := int64(50)
    chunkNum := 1
    
    fmt.Println("\nProcessing in chunks:")
    for {
        chunk := io.LimitReader(largeSource, chunkSize)
        data, err := io.ReadAll(chunk)
        if err != nil {
            fmt.Printf("Error reading chunk: %v\n", err)
            break
        }
        
        if len(data) == 0 {
            break
        }
        
        fmt.Printf("Chunk %d (%d bytes): %q...\n", chunkNum, len(data), string(data[:min(20, len(data))]))
        chunkNum++
        
        if chunkNum > 5 { // Limit output
            break
        }
    }
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

## Section reader

Section reader reads from a specific section of a ReaderAt,  
providing offset and length control for partial reading.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    source := strings.NewReader(content)
    
    // Create section reader for bytes 10-20
    section := io.NewSectionReader(source, 10, 10)
    
    sectionData, err := io.ReadAll(section)
    if err != nil {
        fmt.Printf("Error reading section: %v\n", err)
        return
    }
    
    fmt.Printf("Original: %s\n", content)
    fmt.Printf("Section (offset 10, length 10): %s\n", string(sectionData))
    
    // Multiple sections from same source
    source.Reset(content)
    
    header := io.NewSectionReader(source, 0, 10)
    middle := io.NewSectionReader(source, 25, 15)
    tail := io.NewSectionReader(source, 50, 12)
    
    headerData, _ := io.ReadAll(header)
    middleData, _ := io.ReadAll(middle)
    tailData, _ := io.ReadAll(tail)
    
    fmt.Printf("Header (0-10): %s\n", string(headerData))
    fmt.Printf("Middle (25-40): %s\n", string(middleData))
    fmt.Printf("Tail (50-62): %s\n", string(tailData))
    
    // Reading with seeking
    source.Reset(content)
    seeker := io.NewSectionReader(source, 20, 20)
    
    // Read first part
    buffer := make([]byte, 5)
    n, err := seeker.Read(buffer)
    if err != nil {
        fmt.Printf("Error reading first part: %v\n", err)
        return
    }
    fmt.Printf("First read: %s\n", string(buffer[:n]))
    
    // Seek within section
    pos, err := seeker.Seek(10, io.SeekStart)
    if err != nil {
        fmt.Printf("Error seeking: %v\n", err)
        return
    }
    fmt.Printf("Seeked to position: %d\n", pos)
    
    // Read after seek
    n, err = seeker.Read(buffer)
    if err != nil && err != io.EOF {
        fmt.Printf("Error reading after seek: %v\n", err)
        return
    }
    fmt.Printf("After seek: %s\n", string(buffer[:n]))
    
    // Get section size
    fmt.Printf("Section size: %d bytes\n", seeker.Size())
    
    // Demonstrate bounds checking
    source.Reset(content)
    boundedSection := io.NewSectionReader(source, 55, 20) // Extends beyond content
    
    boundedData, err := io.ReadAll(boundedSection)
    if err != nil {
        fmt.Printf("Error reading bounded section: %v\n", err)
        return
    }
    
    fmt.Printf("Bounded section (extends beyond): %q\n", string(boundedData))
}
```

## Tee reader

Tee reader allows reading from a source while simultaneously writing  
the data to a destination, useful for logging or monitoring data flow.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "hello there from tee reader\nwith data flow monitoring\nand simultaneous writing"
    source := strings.NewReader(content)
    
    // Create a destination for the tee
    var logBuffer strings.Builder
    
    // Create tee reader
    teeReader := io.TeeReader(source, &logBuffer)
    
    // Read from tee reader
    buffer := make([]byte, 16)
    var readData strings.Builder
    
    for {
        n, err := teeReader.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading: %v\n", err)
            break
        }
        
        readData.Write(buffer[:n])
        fmt.Printf("Read chunk: %q\n", string(buffer[:n]))
    }
    
    fmt.Printf("\nOriginal data: %s\n", content)
    fmt.Printf("Read data: %s\n", readData.String())
    fmt.Printf("Logged data: %s\n", logBuffer.String())
    fmt.Printf("Data matches: %t\n", readData.String() == logBuffer.String())
    
    // Practical example: monitoring file read
    fileContent := "Important file content\nthat needs to be monitored\nduring processing"
    fileSource := strings.NewReader(fileContent)
    
    var monitorLog strings.Builder
    monitoredReader := io.TeeReader(fileSource, &monitorLog)
    
    // Process the file content
    processedData, err := io.ReadAll(monitoredReader)
    if err != nil {
        fmt.Printf("Error processing: %v\n", err)
        return
    }
    
    fmt.Printf("\nProcessed %d bytes\n", len(processedData))
    fmt.Printf("Monitor log captured %d bytes\n", logBuffer.Len())
    
    // Multiple tee readers
    source2 := strings.NewReader("Data for multiple monitoring")
    var log1, log2 strings.Builder
    
    tee1 := io.TeeReader(source2, &log1)
    tee2 := io.TeeReader(tee1, &log2)
    
    finalData, err := io.ReadAll(tee2)
    if err != nil {
        fmt.Printf("Error with multiple tees: %v\n", err)
        return
    }
    
    fmt.Printf("\nMultiple tee example:\n")
    fmt.Printf("Final data: %s\n", string(finalData))
    fmt.Printf("Log1: %s\n", log1.String())
    fmt.Printf("Log2: %s\n", log2.String())
    
    // Tee with file-like operations
    documentContent := "Document header\nDocument body with important content\nDocument footer"
    docSource := strings.NewReader(documentContent)
    
    var auditLog strings.Builder
    auditReader := io.TeeReader(docSource, &auditLog)
    
    // Simulate document processing
    lines := strings.Split(string(func() []byte {
        data, _ := io.ReadAll(auditReader)
        return data
    }()), "\n")
    
    fmt.Printf("\nDocument processing:\n")
    for i, line := range lines {
        fmt.Printf("Line %d: %s\n", i+1, line)
    }
    
    fmt.Printf("Audit trail:\n%s", auditLog.String())
}
```

## Write string function

WriteString provides a convenient way to write strings to writers,  
with automatic conversion and error handling.  

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
    
    // Write string to builder
    n, err := io.WriteString(&buffer, "hello there from WriteString")
    if err != nil {
        fmt.Printf("Error writing string: %v\n", err)
        return
    }
    
    fmt.Printf("Wrote %d bytes to buffer\n", n)
    fmt.Printf("Buffer content: %s\n", buffer.String())
    
    // Write multiple strings
    messages := []string{
        "\nFirst message",
        "\nSecond message",
        "\nThird message with more content",
    }
    
    for i, msg := range messages {
        n, err := io.WriteString(&buffer, msg)
        if err != nil {
            fmt.Printf("Error writing message %d: %v\n", i+1, err)
            continue
        }
        fmt.Printf("Message %d: wrote %d bytes\n", i+1, n)
    }
    
    fmt.Printf("Final buffer:\n%s\n", buffer.String())
    
    // Write to file
    filename := "writestring_test.txt"
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(filename)
    
    content := "File content written with WriteString\nSecond line of file content\n"
    n, err = io.WriteString(file, content)
    if err != nil {
        fmt.Printf("Error writing to file: %v\n", err)
        return
    }
    
    fmt.Printf("Wrote %d bytes to file\n", n)
    
    // Verify file content
    readContent, err := os.ReadFile(filename)
    if err != nil {
        fmt.Printf("Error reading file: %v\n", err)
        return
    }
    
    fmt.Printf("File contains: %s", string(readContent))
    fmt.Printf("Content matches: %t\n", string(readContent) == content)
    
    // Write to standard output
    fmt.Println("\nWriting to stdout:")
    io.WriteString(os.Stdout, "Direct output to stdout\n")
    io.WriteString(os.Stderr, "Direct output to stderr\n")
    
    // Chained writing
    var chainBuffer strings.Builder
    
    texts := []string{
        "Chain link 1",
        " -> Chain link 2",
        " -> Chain link 3",
        " -> End of chain\n",
    }
    
    totalBytes := 0
    for _, text := range texts {
        n, err := io.WriteString(&chainBuffer, text)
        if err != nil {
            fmt.Printf("Error in chain: %v\n", err)
            break
        }
        totalBytes += n
    }
    
    fmt.Printf("Chained writing: %d total bytes\n", totalBytes)
    fmt.Printf("Chain result: %s", chainBuffer.String())
}
```

## Read at least function

ReadAtLeast ensures that a minimum number of bytes are read from  
a reader, useful when you need a specific amount of data.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "hello there from ReadAtLeast with sufficient content for testing minimum read requirements"
    source := strings.NewReader(content)
    
    // Read at least 10 bytes
    buffer := make([]byte, 20)
    n, err := io.ReadAtLeast(source, buffer, 10)
    if err != nil {
        fmt.Printf("Error reading at least 10 bytes: %v\n", err)
        return
    }
    
    fmt.Printf("Read %d bytes (requested at least 10): %q\n", n, string(buffer[:n]))
    
    // Try to read more than available
    shortContent := "short"
    shortSource := strings.NewReader(shortContent)
    
    buffer2 := make([]byte, 20)
    n, err = io.ReadAtLeast(shortSource, buffer2, 10)
    if err != nil {
        fmt.Printf("Expected error reading at least 10 from short content: %v\n", err)
        fmt.Printf("Actually read: %d bytes: %q\n", n, string(buffer2[:n]))
    }
    
    // Exact read requirement
    exactContent := "exactly20characters!"
    exactSource := strings.NewReader(exactContent)
    
    exactBuffer := make([]byte, 25)
    n, err = io.ReadAtLeast(exactSource, exactBuffer, 20)
    if err != nil {
        fmt.Printf("Error reading exactly 20: %v\n", err)
        return
    }
    
    fmt.Printf("Read exactly %d bytes: %q\n", n, string(exactBuffer[:n]))
    
    // Reading with larger buffer than minimum
    largeContent := "hello there with substantial content for testing larger buffer scenarios"
    largeSource := strings.NewReader(largeContent)
    
    largeBuffer := make([]byte, 50)
    n, err = io.ReadAtLeast(largeSource, largeBuffer, 15)
    if err != nil {
        fmt.Printf("Error reading with large buffer: %v\n", err)
        return
    }
    
    fmt.Printf("Large buffer read %d bytes (min 15): %q\n", n, string(largeBuffer[:n]))
    
    // Multiple ReadAtLeast calls
    multiContent := "First part of content. Second part of content. Third part with more data."
    multiSource := strings.NewReader(multiContent)
    
    for i := 1; i <= 3; i++ {
        chunkBuffer := make([]byte, 25)
        n, err := io.ReadAtLeast(multiSource, chunkBuffer, 8)
        if err != nil {
            if err == io.EOF {
                fmt.Printf("Reached end of content at chunk %d\n", i)
                break
            }
            fmt.Printf("Error reading chunk %d: %v\n", i, err)
            break
        }
        
        fmt.Printf("Chunk %d: read %d bytes: %q\n", i, n, string(chunkBuffer[:n]))
    }
    
    // Practical example: reading fixed-size records
    recordData := "REC001DATA01REC002DATA02REC003DATA03"
    recordSource := strings.NewReader(recordData)
    
    recordSize := 12
    recordNum := 1
    
    fmt.Println("\nReading fixed-size records:")
    for {
        recordBuffer := make([]byte, recordSize)
        n, err := io.ReadAtLeast(recordSource, recordBuffer, recordSize)
        if err != nil {
            if err == io.EOF || err == io.ErrUnexpectedEOF {
                fmt.Printf("Finished reading records\n")
                break
            }
            fmt.Printf("Error reading record %d: %v\n", recordNum, err)
            break
        }
        
        fmt.Printf("Record %d: %q\n", recordNum, string(recordBuffer[:n]))
        recordNum++
    }
}
```

## Read full function

ReadFull reads exactly the number of bytes needed to fill the buffer,  
ensuring complete data retrieval or returning an error.  

```go
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    content := "hello there from ReadFull with exactly enough content for complete buffer filling operations"
    source := strings.NewReader(content)
    
    // Read exactly 15 bytes
    buffer := make([]byte, 15)
    n, err := io.ReadFull(source, buffer)
    if err != nil {
        fmt.Printf("Error reading full buffer: %v\n", err)
        return
    }
    
    fmt.Printf("ReadFull: read %d bytes (buffer size %d): %q\n", n, len(buffer), string(buffer))
    
    // Try to read more than available
    shortContent := "short data"
    shortSource := strings.NewReader(shortContent)
    
    largeBuffer := make([]byte, 20)
    n, err = io.ReadFull(shortSource, largeBuffer)
    if err != nil {
        fmt.Printf("Expected error with insufficient data: %v\n", err)
        fmt.Printf("Partial read: %d bytes: %q\n", n, string(largeBuffer[:n]))
    }
    
    // Read multiple full buffers
    longContent := "This is a longer content string with multiple segments for testing ReadFull functionality"
    longSource := strings.NewReader(longContent)
    
    bufferSize := 20
    segmentNum := 1
    
    fmt.Println("\nReading multiple full buffers:")
    for {
        segment := make([]byte, bufferSize)
        n, err := io.ReadFull(longSource, segment)
        if err != nil {
            if err == io.EOF {
                fmt.Printf("Reached end of content\n")
                break
            }
            if err == io.ErrUnexpectedEOF {
                fmt.Printf("Partial final segment: %d bytes: %q\n", n, string(segment[:n]))
                break
            }
            fmt.Printf("Error reading segment %d: %v\n", segmentNum, err)
            break
        }
        
        fmt.Printf("Segment %d: %q\n", segmentNum, string(segment))
        segmentNum++
        
        if segmentNum > 4 { // Limit output
            break
        }
    }
    
    // Binary data reading
    binaryData := []byte{0x48, 0x65, 0x6C, 0x6C, 0x6F, 0x00, 0x57, 0x6F, 0x72, 0x6C, 0x64, 0xFF, 0xFE, 0xFD}
    binarySource := strings.NewReader(string(binaryData))
    
    // Read header (first 5 bytes)
    header := make([]byte, 5)
    n, err = io.ReadFull(binarySource, header)
    if err != nil {
        fmt.Printf("Error reading binary header: %v\n", err)
        return
    }
    
    fmt.Printf("\nBinary header: %v (as string: %q)\n", header, string(header))
    
    // Read footer (last 3 bytes)
    footer := make([]byte, 3)
    binarySource.Seek(-3, io.SeekEnd)
    n, err = io.ReadFull(binarySource, footer)
    if err != nil {
        fmt.Printf("Error reading binary footer: %v\n", err)
        return
    }
    
    fmt.Printf("Binary footer: %v\n", footer)
    
    // Structured data reading
    structuredData := "HDR001DATA123456789END"
    structSource := strings.NewReader(structuredData)
    
    // Read components with fixed sizes
    headerPart := make([]byte, 6)
    dataPart := make([]byte, 13)
    trailerPart := make([]byte, 3)
    
    // Read header
    _, err = io.ReadFull(structSource, headerPart)
    if err != nil {
        fmt.Printf("Error reading header part: %v\n", err)
        return
    }
    
    // Read data
    _, err = io.ReadFull(structSource, dataPart)
    if err != nil {
        fmt.Printf("Error reading data part: %v\n", err)
        return
    }
    
    // Read trailer
    _, err = io.ReadFull(structSource, trailerPart)
    if err != nil {
        fmt.Printf("Error reading trailer part: %v\n", err)
        return
    }
    
    fmt.Printf("\nStructured data:\n")
    fmt.Printf("Header: %q\n", string(headerPart))
    fmt.Printf("Data: %q\n", string(dataPart))
    fmt.Printf("Trailer: %q\n", string(trailerPart))
}
```

## File seeking operations

File seeking allows random access to file contents by moving the  
read/write position to specific locations within the file.  

```go
package main

import (
    "fmt"
    "io"
    "os"
)

func main() {
    content := "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    filename := "seek_test.txt"
    
    // Create test file
    err := os.WriteFile(filename, []byte(content), 0644)
    if err != nil {
        fmt.Printf("Error creating test file: %v\n", err)
        return
    }
    defer os.Remove(filename)
    
    // Open file for reading
    file, err := os.Open(filename)
    if err != nil {
        fmt.Printf("Error opening file: %v\n", err)
        return
    }
    defer file.Close()
    
    // Read from beginning
    buffer := make([]byte, 5)
    n, err := file.Read(buffer)
    if err != nil {
        fmt.Printf("Error reading from start: %v\n", err)
        return
    }
    fmt.Printf("From start: %s\n", string(buffer[:n]))
    
    // Seek to position 10 from start
    pos, err := file.Seek(10, io.SeekStart)
    if err != nil {
        fmt.Printf("Error seeking to 10: %v\n", err)
        return
    }
    fmt.Printf("Seeked to position: %d\n", pos)
    
    // Read from new position
    n, err = file.Read(buffer)
    if err != nil {
        fmt.Printf("Error reading after seek: %v\n", err)
        return
    }
    fmt.Printf("After seek to 10: %s\n", string(buffer[:n]))
    
    // Seek relative to current position
    pos, err = file.Seek(5, io.SeekCurrent)
    if err != nil {
        fmt.Printf("Error seeking relative: %v\n", err)
        return
    }
    fmt.Printf("Relative seek position: %d\n", pos)
    
    n, err = file.Read(buffer)
    if err != nil {
        fmt.Printf("Error reading after relative seek: %v\n", err)
        return
    }
    fmt.Printf("After relative seek: %s\n", string(buffer[:n]))
    
    // Seek from end
    pos, err = file.Seek(-10, io.SeekEnd)
    if err != nil {
        fmt.Printf("Error seeking from end: %v\n", err)
        return
    }
    fmt.Printf("Seek from end position: %d\n", pos)
    
    n, err = file.Read(buffer)
    if err != nil && err != io.EOF {
        fmt.Printf("Error reading from end: %v\n", err)
        return
    }
    fmt.Printf("From end (-10): %s\n", string(buffer[:n]))
    
    // Get current position
    currentPos, err := file.Seek(0, io.SeekCurrent)
    if err != nil {
        fmt.Printf("Error getting current position: %v\n", err)
        return
    }
    fmt.Printf("Current position: %d\n", currentPos)
    
    // Get file size by seeking to end
    fileSize, err := file.Seek(0, io.SeekEnd)
    if err != nil {
        fmt.Printf("Error getting file size: %v\n", err)
        return
    }
    fmt.Printf("File size: %d bytes\n", fileSize)
    
    // Random access pattern
    positions := []int64{0, 15, 30, 45, 60}
    file.Seek(0, io.SeekStart) // Reset to beginning
    
    fmt.Println("\nRandom access pattern:")
    for i, seekPos := range positions {
        pos, err := file.Seek(seekPos, io.SeekStart)
        if err != nil {
            fmt.Printf("Error seeking to %d: %v\n", seekPos, err)
            continue
        }
        
        readBuffer := make([]byte, 3)
        n, err := file.Read(readBuffer)
        if err != nil && err != io.EOF {
            fmt.Printf("Error reading at position %d: %v\n", pos, err)
            continue
        }
        
        fmt.Printf("Position %d: %s\n", pos, string(readBuffer[:n]))
    }
}
```

## Temporary files and directories

Temporary files and directories provide secure, automatically cleaned  
storage for intermediate processing or testing purposes.  

```go
package main

import (
    "fmt"
    "io"
    "os"
    "path/filepath"
)

func main() {
    // Create temporary file
    tempFile, err := os.CreateTemp("", "example_*.txt")
    if err != nil {
        fmt.Printf("Error creating temp file: %v\n", err)
        return
    }
    defer os.Remove(tempFile.Name()) // Cleanup
    defer tempFile.Close()
    
    fmt.Printf("Temporary file: %s\n", tempFile.Name())
    
    // Write to temporary file
    content := "hello there from temporary file\nwith test content\nfor processing"
    n, err := tempFile.WriteString(content)
    if err != nil {
        fmt.Printf("Error writing to temp file: %v\n", err)
        return
    }
    fmt.Printf("Wrote %d bytes to temp file\n", n)
    
    // Read back from temporary file
    tempFile.Seek(0, io.SeekStart) // Reset to beginning
    
    readContent, err := io.ReadAll(tempFile)
    if err != nil {
        fmt.Printf("Error reading temp file: %v\n", err)
        return
    }
    
    fmt.Printf("Read back: %s\n", string(readContent))
    
    // Create temporary directory
    tempDir, err := os.MkdirTemp("", "example_dir_*")
    if err != nil {
        fmt.Printf("Error creating temp dir: %v\n", err)
        return
    }
    defer os.RemoveAll(tempDir) // Cleanup directory and contents
    
    fmt.Printf("Temporary directory: %s\n", tempDir)
    
    // Create files in temporary directory
    for i := 1; i <= 3; i++ {
        filename := filepath.Join(tempDir, fmt.Sprintf("file%d.txt", i))
        fileContent := fmt.Sprintf("Content of file %d\nCreated in temp directory", i)
        
        err := os.WriteFile(filename, []byte(fileContent), 0644)
        if err != nil {
            fmt.Printf("Error creating file %d: %v\n", i, err)
            continue
        }
        
        fmt.Printf("Created: %s\n", filename)
    }
    
    // List contents of temporary directory
    entries, err := os.ReadDir(tempDir)
    if err != nil {
        fmt.Printf("Error reading temp dir: %v\n", err)
        return
    }
    
    fmt.Printf("Temp directory contents:\n")
    for _, entry := range entries {
        info, err := entry.Info()
        if err != nil {
            fmt.Printf("Error getting info for %s: %v\n", entry.Name(), err)
            continue
        }
        
        fmt.Printf("  %s (%d bytes)\n", entry.Name(), info.Size())
    }
    
    // Create temporary file with specific pattern
    patternFile, err := os.CreateTemp(tempDir, "pattern_*_test.log")
    if err != nil {
        fmt.Printf("Error creating pattern file: %v\n", err)
        return
    }
    defer patternFile.Close()
    
    fmt.Printf("Pattern file: %s\n", patternFile.Name())
    
    // Write structured data to pattern file
    logEntries := []string{
        "2024-01-01 10:00:00 INFO Application started",
        "2024-01-01 10:01:00 DEBUG Processing data",
        "2024-01-01 10:02:00 INFO Operation completed",
    }
    
    for _, entry := range logEntries {
        fmt.Fprintf(patternFile, "%s\n", entry)
    }
    
    // Get file info
    fileInfo, err := patternFile.Stat()
    if err != nil {
        fmt.Printf("Error getting file info: %v\n", err)
        return
    }
    
    fmt.Printf("Pattern file size: %d bytes\n", fileInfo.Size())
    fmt.Printf("Pattern file mode: %v\n", fileInfo.Mode())
    
    // Demonstrate cleanup
    fmt.Println("Cleanup will be handled by defer statements")
}
```

## Working with paths

Path operations help manipulate file and directory paths in a  
cross-platform manner using the filepath package.  

```go
package main

import (
    "fmt"
    "os"
    "path/filepath"
    "strings"
)

func main() {
    // Basic path operations
    path := "/home/user/documents/file.txt"
    
    fmt.Printf("Original path: %s\n", path)
    fmt.Printf("Directory: %s\n", filepath.Dir(path))
    fmt.Printf("Base name: %s\n", filepath.Base(path))
    fmt.Printf("Extension: %s\n", filepath.Ext(path))
    
    // Clean path
    messy := "/home/user/../user/./documents//file.txt"
    clean := filepath.Clean(messy)
    fmt.Printf("Messy path: %s\n", messy)
    fmt.Printf("Clean path: %s\n", clean)
    
    // Join paths
    parts := []string{"home", "user", "documents", "projects", "go"}
    joined := filepath.Join(parts...)
    fmt.Printf("Joined path: %s\n", joined)
    
    // Relative paths
    base := "/home/user/documents"
    target := "/home/user/documents/projects/go/main.go"
    
    rel, err := filepath.Rel(base, target)
    if err != nil {
        fmt.Printf("Error calculating relative path: %v\n", err)
    } else {
        fmt.Printf("Relative path from %s to %s: %s\n", base, target, rel)
    }
    
    // Absolute path
    currentDir, _ := os.Getwd()
    relativePath := "test.txt"
    
    abs, err := filepath.Abs(relativePath)
    if err != nil {
        fmt.Printf("Error getting absolute path: %v\n", err)
    } else {
        fmt.Printf("Current directory: %s\n", currentDir)
        fmt.Printf("Relative: %s -> Absolute: %s\n", relativePath, abs)
    }
    
    // Split path
    dir, file := filepath.Split(path)
    fmt.Printf("Split %s -> dir: %s, file: %s\n", path, dir, file)
    
    // Volume name (Windows specific, empty on Unix)
    volume := filepath.VolumeName(path)
    fmt.Printf("Volume name: %q\n", volume)
    
    // Check if path is absolute
    fmt.Printf("Is %s absolute? %t\n", path, filepath.IsAbs(path))
    fmt.Printf("Is %s absolute? %t\n", "documents/file.txt", filepath.IsAbs("documents/file.txt"))
    
    // Pattern matching
    pattern := "*.txt"
    testFiles := []string{"file.txt", "document.pdf", "readme.txt", "image.png"}
    
    fmt.Printf("\nPattern matching with %s:\n", pattern)
    for _, file := range testFiles {
        matched, err := filepath.Match(pattern, file)
        if err != nil {
            fmt.Printf("Error matching %s: %v\n", file, err)
            continue
        }
        fmt.Printf("  %s: %t\n", file, matched)
    }
    
    // Walking directory tree (simulate with current directory)
    fmt.Println("\nWalking current directory (first 5 entries):")
    count := 0
    err = filepath.Walk(".", func(path string, info os.FileInfo, err error) error {
        if err != nil {
            return err
        }
        
        if count >= 5 {
            return filepath.SkipDir
        }
        
        fileType := "file"
        if info.IsDir() {
            fileType = "directory"
        }
        
        fmt.Printf("  %s (%s, %d bytes)\n", path, fileType, info.Size())
        count++
        return nil
    })
    
    if err != nil {
        fmt.Printf("Error walking directory: %v\n", err)
    }
    
    // Create and manipulate complex paths
    basePath := "projects"
    subDirs := []string{"frontend", "backend", "database"}
    files := []string{"main.go", "config.json", "README.md"}
    
    fmt.Println("\nComplex path construction:")
    for _, subDir := range subDirs {
        dirPath := filepath.Join(basePath, subDir)
        fmt.Printf("Directory: %s\n", dirPath)
        
        for _, file := range files {
            filePath := filepath.Join(dirPath, file)
            fmt.Printf("  File: %s\n", filePath)
        }
    }
    
    // Change extension
    originalFile := "document.pdf"
    newExt := ".txt"
    nameWithoutExt := strings.TrimSuffix(originalFile, filepath.Ext(originalFile))
    newFile := nameWithoutExt + newExt
    
    fmt.Printf("\nExtension change: %s -> %s\n", originalFile, newFile)
}
```

## File information and stats

File information provides metadata about files and directories  
including size, permissions, modification time, and type.  

```go
package main

import (
    "fmt"
    "os"
    "time"
)

func main() {
    // Create a test file
    testFile := "fileinfo_test.txt"
    content := "hello there from file info test\nwith multiple lines\nfor stat testing"
    
    err := os.WriteFile(testFile, []byte(content), 0644)
    if err != nil {
        fmt.Printf("Error creating test file: %v\n", err)
        return
    }
    defer os.Remove(testFile)
    
    // Get file info
    info, err := os.Stat(testFile)
    if err != nil {
        fmt.Printf("Error getting file info: %v\n", err)
        return
    }
    
    fmt.Printf("File name: %s\n", info.Name())
    fmt.Printf("File size: %d bytes\n", info.Size())
    fmt.Printf("File mode: %v\n", info.Mode())
    fmt.Printf("Modification time: %v\n", info.ModTime())
    fmt.Printf("Is directory: %t\n", info.IsDir())
    
    // Check specific permissions
    mode := info.Mode()
    fmt.Printf("File mode bits: %o\n", mode.Perm())
    fmt.Printf("Is regular file: %t\n", mode.IsRegular())
    fmt.Printf("Owner read: %t\n", mode&0400 != 0)
    fmt.Printf("Owner write: %t\n", mode&0200 != 0)
    fmt.Printf("Owner execute: %t\n", mode&0100 != 0)
    
    // Create a directory and check its info
    testDir := "test_directory"
    err = os.Mkdir(testDir, 0755)
    if err != nil {
        fmt.Printf("Error creating directory: %v\n", err)
        return
    }
    defer os.Remove(testDir)
    
    dirInfo, err := os.Stat(testDir)
    if err != nil {
        fmt.Printf("Error getting directory info: %v\n", err)
        return
    }
    
    fmt.Printf("\nDirectory name: %s\n", dirInfo.Name())
    fmt.Printf("Directory size: %d\n", dirInfo.Size())
    fmt.Printf("Directory mode: %v\n", dirInfo.Mode())
    fmt.Printf("Is directory: %t\n", dirInfo.IsDir())
    
    // File existence check
    existingFile := testFile
    nonExistentFile := "non_existent_file.txt"
    
    fmt.Printf("\nFile existence checks:\n")
    
    if _, err := os.Stat(existingFile); err == nil {
        fmt.Printf("%s exists\n", existingFile)
    } else if os.IsNotExist(err) {
        fmt.Printf("%s does not exist\n", existingFile)
    } else {
        fmt.Printf("Error checking %s: %v\n", existingFile, err)
    }
    
    if _, err := os.Stat(nonExistentFile); err == nil {
        fmt.Printf("%s exists\n", nonExistentFile)
    } else if os.IsNotExist(err) {
        fmt.Printf("%s does not exist\n", nonExistentFile)
    } else {
        fmt.Printf("Error checking %s: %v\n", nonExistentFile, err)
    }
    
    // Modify file and check time changes
    time.Sleep(1 * time.Second) // Ensure time difference
    
    additionalContent := "\nAdditional content added later"
    file, err := os.OpenFile(testFile, os.O_APPEND|os.O_WRONLY, 0644)
    if err != nil {
        fmt.Printf("Error opening file for append: %v\n", err)
        return
    }
    defer file.Close()
    
    file.WriteString(additionalContent)
    
    // Get updated info
    updatedInfo, err := os.Stat(testFile)
    if err != nil {
        fmt.Printf("Error getting updated file info: %v\n", err)
        return
    }
    
    fmt.Printf("\nAfter modification:\n")
    fmt.Printf("Original size: %d bytes\n", info.Size())
    fmt.Printf("Updated size: %d bytes\n", updatedInfo.Size())
    fmt.Printf("Original mod time: %v\n", info.ModTime())
    fmt.Printf("Updated mod time: %v\n", updatedInfo.ModTime())
    fmt.Printf("Time difference: %v\n", updatedInfo.ModTime().Sub(info.ModTime()))
    
    // Change file permissions
    err = os.Chmod(testFile, 0600)
    if err != nil {
        fmt.Printf("Error changing permissions: %v\n", err)
        return
    }
    
    permInfo, err := os.Stat(testFile)
    if err != nil {
        fmt.Printf("Error getting permission info: %v\n", err)
        return
    }
    
    fmt.Printf("\nAfter permission change:\n")
    fmt.Printf("Original mode: %v\n", info.Mode())
    fmt.Printf("Updated mode: %v\n", permInfo.Mode())
    
    // File size comparison
    sizes := []string{"bytes", "KB", "MB", "GB"}
    size := float64(updatedInfo.Size())
    
    fmt.Printf("\nFile size representations:\n")
    for i, unit := range sizes {
        if i == 0 {
            fmt.Printf("%.0f %s\n", size, unit)
        } else {
            size /= 1024
            if size >= 1 {
                fmt.Printf("%.2f %s\n", size, unit)
            }
        }
    }
}
```

## Compression with gzip

Gzip compression provides efficient data compression for reducing  
file sizes and network transfer times.  

```go
package main

import (
    "bytes"
    "compress/gzip"
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    originalData := "hello there from gzip compression example\n"
    originalData += strings.Repeat("This line will be repeated many times for better compression ratio.\n", 10)
    
    fmt.Printf("Original data size: %d bytes\n", len(originalData))
    
    // Compress data
    var compressedBuffer bytes.Buffer
    
    gzipWriter := gzip.NewWriter(&compressedBuffer)
    
    _, err := gzipWriter.Write([]byte(originalData))
    if err != nil {
        fmt.Printf("Error writing to gzip: %v\n", err)
        return
    }
    
    err = gzipWriter.Close()
    if err != nil {
        fmt.Printf("Error closing gzip writer: %v\n", err)
        return
    }
    
    compressedData := compressedBuffer.Bytes()
    fmt.Printf("Compressed data size: %d bytes\n", len(compressedData))
    fmt.Printf("Compression ratio: %.2f%%\n", float64(len(compressedData))/float64(len(originalData))*100)
    
    // Decompress data
    compressedReader := bytes.NewReader(compressedData)
    gzipReader, err := gzip.NewReader(compressedReader)
    if err != nil {
        fmt.Printf("Error creating gzip reader: %v\n", err)
        return
    }
    defer gzipReader.Close()
    
    decompressedData, err := io.ReadAll(gzipReader)
    if err != nil {
        fmt.Printf("Error reading from gzip: %v\n", err)
        return
    }
    
    fmt.Printf("Decompressed data size: %d bytes\n", len(decompressedData))
    fmt.Printf("Data integrity check: %t\n", string(decompressedData) == originalData)
    
    // Compress file
    inputFile := "compress_input.txt"
    outputFile := "compressed_output.gz"
    
    err = os.WriteFile(inputFile, []byte(originalData), 0644)
    if err != nil {
        fmt.Printf("Error creating input file: %v\n", err)
        return
    }
    defer os.Remove(inputFile)
    defer os.Remove(outputFile)
    
    // Read input file
    input, err := os.Open(inputFile)
    if err != nil {
        fmt.Printf("Error opening input file: %v\n", err)
        return
    }
    defer input.Close()
    
    // Create compressed output file
    output, err := os.Create(outputFile)
    if err != nil {
        fmt.Printf("Error creating output file: %v\n", err)
        return
    }
    defer output.Close()
    
    // Compress file to file
    gzWriter := gzip.NewWriter(output)
    
    _, err = io.Copy(gzWriter, input)
    if err != nil {
        fmt.Printf("Error compressing file: %v\n", err)
        return
    }
    
    err = gzWriter.Close()
    if err != nil {
        fmt.Printf("Error closing gzip writer: %v\n", err)
        return
    }
    
    // Check compressed file size
    compressedInfo, _ := os.Stat(outputFile)
    originalInfo, _ := os.Stat(inputFile)
    
    fmt.Printf("\nFile compression:\n")
    fmt.Printf("Original file size: %d bytes\n", originalInfo.Size())
    fmt.Printf("Compressed file size: %d bytes\n", compressedInfo.Size())
    fmt.Printf("File compression ratio: %.2f%%\n", float64(compressedInfo.Size())/float64(originalInfo.Size())*100)
    
    // Decompress file
    decompressedFile := "decompressed_output.txt"
    defer os.Remove(decompressedFile)
    
    // Open compressed file
    compFile, err := os.Open(outputFile)
    if err != nil {
        fmt.Printf("Error opening compressed file: %v\n", err)
        return
    }
    defer compFile.Close()
    
    // Create decompressed output
    decompFile, err := os.Create(decompressedFile)
    if err != nil {
        fmt.Printf("Error creating decompressed file: %v\n", err)
        return
    }
    defer decompFile.Close()
    
    // Decompress
    gzReader, err := gzip.NewReader(compFile)
    if err != nil {
        fmt.Printf("Error creating gzip reader for file: %v\n", err)
        return
    }
    defer gzReader.Close()
    
    _, err = io.Copy(decompFile, gzReader)
    if err != nil {
        fmt.Printf("Error decompressing file: %v\n", err)
        return
    }
    
    // Verify decompressed file
    decompressedContent, err := os.ReadFile(decompressedFile)
    if err != nil {
        fmt.Printf("Error reading decompressed file: %v\n", err)
        return
    }
    
    fmt.Printf("File decompression successful: %t\n", string(decompressedContent) == originalData)
}
```

## CSV reading and writing

CSV operations provide structured data reading and writing for  
tabular data exchange and processing.  

```go
package main

import (
    "encoding/csv"
    "fmt"
    "io"
    "os"
    "strings"
)

func main() {
    // Create CSV data
    csvData := `name,age,city,country
John Doe,30,New York,USA
Jane Smith,25,London,UK
Bob Johnson,35,Toronto,Canada
Alice Brown,28,Sydney,Australia`
    
    // Read CSV from string
    reader := csv.NewReader(strings.NewReader(csvData))
    
    records, err := reader.ReadAll()
    if err != nil {
        fmt.Printf("Error reading CSV: %v\n", err)
        return
    }
    
    fmt.Println("CSV Records:")
    for i, record := range records {
        if i == 0 {
            fmt.Printf("Headers: %v\n", record)
            continue
        }
        fmt.Printf("Record %d: %v\n", i, record)
    }
    
    // Read CSV record by record
    reader2 := csv.NewReader(strings.NewReader(csvData))
    
    fmt.Println("\nReading record by record:")
    recordNum := 0
    for {
        record, err := reader2.Read()
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading record: %v\n", err)
            break
        }
        
        fmt.Printf("Record %d: %v\n", recordNum, record)
        recordNum++
    }
    
    // Write CSV to file
    outputFile := "output.csv"
    file, err := os.Create(outputFile)
    if err != nil {
        fmt.Printf("Error creating CSV file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(outputFile)
    
    writer := csv.NewWriter(file)
    defer writer.Flush()
    
    // Write header
    header := []string{"Product", "Price", "Quantity", "Category"}
    err = writer.Write(header)
    if err != nil {
        fmt.Printf("Error writing header: %v\n", err)
        return
    }
    
    // Write data records
    products := [][]string{
        {"Laptop", "999.99", "5", "Electronics"},
        {"Book", "19.99", "100", "Education"},
        {"Coffee", "4.99", "50", "Food & Beverage"},
        {"Desk Chair", "149.99", "10", "Furniture"},
    }
    
    for _, product := range products {
        err = writer.Write(product)
        if err != nil {
            fmt.Printf("Error writing product: %v\n", err)
            continue
        }
    }
    
    fmt.Printf("\nWrote %d product records to %s\n", len(products), outputFile)
    
    // Read back the file
    readFile, err := os.Open(outputFile)
    if err != nil {
        fmt.Printf("Error opening written file: %v\n", err)
        return
    }
    defer readFile.Close()
    
    fileReader := csv.NewReader(readFile)
    fileRecords, err := fileReader.ReadAll()
    if err != nil {
        fmt.Printf("Error reading written file: %v\n", err)
        return
    }
    
    fmt.Println("Written file contents:")
    for i, record := range fileRecords {
        fmt.Printf("Row %d: %v\n", i, record)
    }
    
    // Custom CSV configuration
    customCSV := `product|price|stock
    laptop|999.99|5
    book|19.99|100
    coffee|4.99|50`
    
    customReader := csv.NewReader(strings.NewReader(customCSV))
    customReader.Comma = '|' // Use pipe as delimiter
    customReader.TrimLeadingSpace = true
    
    fmt.Println("\nCustom delimiter CSV:")
    customRecords, err := customReader.ReadAll()
    if err != nil {
        fmt.Printf("Error reading custom CSV: %v\n", err)
        return
    }
    
    for i, record := range customRecords {
        fmt.Printf("Custom row %d: %v\n", i, record)
    }
    
    // Handle CSV with quotes and commas
    quotedCSV := `"Last, First",Age,"City, State"
    "Doe, John",30,"New York, NY"
    "Smith, Jane",25,"Los Angeles, CA"`
    
    quotedReader := csv.NewReader(strings.NewReader(quotedCSV))
    
    fmt.Println("\nQuoted CSV:")
    quotedRecords, err := quotedReader.ReadAll()
    if err != nil {
        fmt.Printf("Error reading quoted CSV: %v\n", err)
        return
    }
    
    for i, record := range quotedRecords {
        fmt.Printf("Quoted row %d: %v\n", i, record)
    }
}
```

## JSON encoding and decoding

JSON operations provide structured data serialization for web APIs  
and configuration files with Go's built-in encoding capabilities.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "os"
    "strings"
)

type Person struct {
    Name    string   `json:"name"`
    Age     int      `json:"age"`
    Email   string   `json:"email"`
    Hobbies []string `json:"hobbies"`
    Address Address  `json:"address"`
}

type Address struct {
    Street  string `json:"street"`
    City    string `json:"city"`
    Country string `json:"country"`
    ZipCode string `json:"zip_code"`
}

func main() {
    // Create sample data
    person := Person{
        Name:    "John Doe",
        Age:     30,
        Email:   "john.doe@example.com",
        Hobbies: []string{"reading", "programming", "hiking"},
        Address: Address{
            Street:  "123 Main St",
            City:    "New York",
            Country: "USA",
            ZipCode: "10001",
        },
    }
    
    // Encode to JSON
    jsonData, err := json.Marshal(person)
    if err != nil {
        fmt.Printf("Error marshaling JSON: %v\n", err)
        return
    }
    
    fmt.Printf("Compact JSON:\n%s\n\n", string(jsonData))
    
    // Pretty print JSON
    prettyJSON, err := json.MarshalIndent(person, "", "  ")
    if err != nil {
        fmt.Printf("Error marshaling pretty JSON: %v\n", err)
        return
    }
    
    fmt.Printf("Pretty JSON:\n%s\n\n", string(prettyJSON))
    
    // Decode JSON
    var decodedPerson Person
    err = json.Unmarshal(jsonData, &decodedPerson)
    if err != nil {
        fmt.Printf("Error unmarshaling JSON: %v\n", err)
        return
    }
    
    fmt.Printf("Decoded person: %+v\n", decodedPerson)
    fmt.Printf("Hobbies: %v\n", decodedPerson.Hobbies)
    fmt.Printf("Address: %+v\n\n", decodedPerson.Address)
    
    // JSON streaming with encoder/decoder
    var buffer strings.Builder
    encoder := json.NewEncoder(&buffer)
    
    // Encode multiple objects
    people := []Person{person, {
        Name:    "Jane Smith",
        Age:     25,
        Email:   "jane.smith@example.com",
        Hobbies: []string{"photography", "travel"},
        Address: Address{
            Street:  "456 Oak Ave",
            City:    "Los Angeles",
            Country: "USA",
            ZipCode: "90210",
        },
    }}
    
    for _, p := range people {
        err = encoder.Encode(p)
        if err != nil {
            fmt.Printf("Error encoding person: %v\n", err)
            continue
        }
    }
    
    fmt.Printf("Encoded stream:\n%s\n", buffer.String())
    
    // Decode stream
    decoder := json.NewDecoder(strings.NewReader(buffer.String()))
    
    fmt.Println("Decoded stream:")
    personNum := 1
    for decoder.More() {
        var streamPerson Person
        err = decoder.Decode(&streamPerson)
        if err != nil {
            fmt.Printf("Error decoding person %d: %v\n", personNum, err)
            break
        }
        
        fmt.Printf("Person %d: %s (%d years old)\n", personNum, streamPerson.Name, streamPerson.Age)
        personNum++
    }
    
    // File operations
    filename := "people.json"
    file, err := os.Create(filename)
    if err != nil {
        fmt.Printf("Error creating file: %v\n", err)
        return
    }
    defer file.Close()
    defer os.Remove(filename)
    
    // Write JSON to file
    fileEncoder := json.NewEncoder(file)
    fileEncoder.SetIndent("", "  ")
    
    err = fileEncoder.Encode(people)
    if err != nil {
        fmt.Printf("Error writing JSON to file: %v\n", err)
        return
    }
    
    fmt.Printf("\nWrote JSON to %s\n", filename)
    
    // Read JSON from file
    readFile, err := os.Open(filename)
    if err != nil {
        fmt.Printf("Error opening file: %v\n", err)
        return
    }
    defer readFile.Close()
    
    var readPeople []Person
    fileDecoder := json.NewDecoder(readFile)
    
    err = fileDecoder.Decode(&readPeople)
    if err != nil {
        fmt.Printf("Error reading JSON from file: %v\n", err)
        return
    }
    
    fmt.Printf("Read %d people from file\n", len(readPeople))
    
    // Handle dynamic JSON
    dynamicJSON := `{
        "name": "Dynamic Object",
        "properties": {
            "count": 42,
            "active": true,
            "tags": ["important", "dynamic"]
        }
    }`
    
    var dynamic map[string]interface{}
    err = json.Unmarshal([]byte(dynamicJSON), &dynamic)
    if err != nil {
        fmt.Printf("Error unmarshaling dynamic JSON: %v\n", err)
        return
    }
    
    fmt.Printf("\nDynamic JSON:\n")
    fmt.Printf("Name: %v\n", dynamic["name"])
    
    if properties, ok := dynamic["properties"].(map[string]interface{}); ok {
        fmt.Printf("Count: %v\n", properties["count"])
        fmt.Printf("Active: %v\n", properties["active"])
        
        if tags, ok := properties["tags"].([]interface{}); ok {
            fmt.Printf("Tags: ")
            for i, tag := range tags {
                if i > 0 {
                    fmt.Printf(", ")
                }
                fmt.Printf("%v", tag)
            }
            fmt.Println()
        }
    }
}
```

## Network IO with HTTP

HTTP operations demonstrate network IO patterns for web service  
communication and data exchange over HTTP protocols.  

```go
package main

import (
    "bytes"
    "fmt"
    "io"
    "net/http"
    "os"
    "strings"
    "time"
)

func main() {
    // Simple HTTP GET request
    resp, err := http.Get("https://httpbin.org/get")
    if err != nil {
        fmt.Printf("Error making GET request: %v\n", err)
        return
    }
    defer resp.Body.Close()
    
    fmt.Printf("GET Response Status: %s\n", resp.Status)
    fmt.Printf("Content-Type: %s\n", resp.Header.Get("Content-Type"))
    
    // Read response body
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response body: %v\n", err)
        return
    }
    
    fmt.Printf("Response body length: %d bytes\n", len(body))
    fmt.Printf("First 200 characters: %s...\n\n", string(body[:min(200, len(body))]))
    
    // HTTP POST request with JSON data
    jsonData := `{"name": "John Doe", "email": "john@example.com"}`
    
    postResp, err := http.Post("https://httpbin.org/post", "application/json", strings.NewReader(jsonData))
    if err != nil {
        fmt.Printf("Error making POST request: %v\n", err)
        return
    }
    defer postResp.Body.Close()
    
    fmt.Printf("POST Response Status: %s\n", postResp.Status)
    
    postBody, err := io.ReadAll(postResp.Body)
    if err != nil {
        fmt.Printf("Error reading POST response: %v\n", err)
        return
    }
    
    fmt.Printf("POST response length: %d bytes\n\n", len(postBody))
    
    // Custom HTTP request with headers
    client := &http.Client{
        Timeout: 10 * time.Second,
    }
    
    req, err := http.NewRequest("GET", "https://httpbin.org/headers", nil)
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }
    
    req.Header.Set("User-Agent", "Go-HTTP-Client/1.0")
    req.Header.Set("Accept", "application/json")
    req.Header.Set("X-Custom-Header", "hello-there")
    
    customResp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error making custom request: %v\n", err)
        return
    }
    defer customResp.Body.Close()
    
    customBody, err := io.ReadAll(customResp.Body)
    if err != nil {
        fmt.Printf("Error reading custom response: %v\n", err)
        return
    }
    
    fmt.Printf("Custom request response:\n%s\n\n", string(customBody))
    
    // Download file via HTTP
    fileURL := "https://httpbin.org/json"
    fileResp, err := http.Get(fileURL)
    if err != nil {
        fmt.Printf("Error downloading file: %v\n", err)
        return
    }
    defer fileResp.Body.Close()
    
    if fileResp.StatusCode != http.StatusOK {
        fmt.Printf("Download failed with status: %s\n", fileResp.Status)
        return
    }
    
    // Save to file
    downloadFile := "downloaded.json"
    outFile, err := os.Create(downloadFile)
    if err != nil {
        fmt.Printf("Error creating download file: %v\n", err)
        return
    }
    defer outFile.Close()
    defer os.Remove(downloadFile)
    
    // Copy response to file
    written, err := io.Copy(outFile, fileResp.Body)
    if err != nil {
        fmt.Printf("Error saving file: %v\n", err)
        return
    }
    
    fmt.Printf("Downloaded %d bytes to %s\n", written, downloadFile)
    
    // Upload data
    uploadData := "This is test data for upload"
    uploadResp, err := http.Post("https://httpbin.org/put", "text/plain", strings.NewReader(uploadData))
    if err != nil {
        fmt.Printf("Error uploading data: %v\n", err)
        return
    }
    defer uploadResp.Body.Close()
    
    fmt.Printf("Upload response status: %s\n", uploadResp.Status)
    
    // Stream processing
    streamResp, err := http.Get("https://httpbin.org/stream/3")
    if err != nil {
        fmt.Printf("Error getting stream: %v\n", err)
        return
    }
    defer streamResp.Body.Close()
    
    fmt.Println("Streaming response:")
    buffer := make([]byte, 256)
    lineNum := 1
    
    for {
        n, err := streamResp.Body.Read(buffer)
        if err == io.EOF {
            break
        }
        if err != nil {
            fmt.Printf("Error reading stream: %v\n", err)
            break
        }
        
        fmt.Printf("Stream chunk %d (%d bytes): %s", lineNum, n, string(buffer[:n]))
        lineNum++
        
        if lineNum > 5 { // Limit output
            break
        }
    }
    
    // Error handling
    _, err = http.Get("https://nonexistent-domain-12345.com")
    if err != nil {
        fmt.Printf("\nExpected error for non-existent domain: %v\n", err)
    }
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```
}
```
```
```
```
