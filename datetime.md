# DateTime

The Go `time` package provides comprehensive functionality for working with dates  
and times. It handles time representation, formatting, parsing, arithmetic operations,  
and timezone conversions. The package is built around the `time.Time` type which  
represents an instant in time with nanosecond precision.

Go's time package follows a unique approach to formatting and parsing using reference  
time "Mon Jan 2 15:04:05 MST 2006", which corresponds to Unix time 1136239445.  
This reference time serves as a mnemonic where each component has a memorable  
position: 01/02 03:04:05PM '06 -0700. Understanding this reference is crucial  
for effective time formatting and parsing in Go.

The time package handles timezone conversions, duration calculations, and provides  
various methods for time manipulation. It supports both monotonic and wall clock  
time measurements, making it suitable for both timing operations and representing  
calendar dates. The package integrates well with standard library components  
and external systems requiring time-based operations.

Time operations in Go are designed to be safe for concurrent use and provide  
high precision timing capabilities. The package includes support for different  
time formats, custom parsing layouts, and efficient duration arithmetic. These  
features make it excellent for applications requiring scheduling, logging,  
benchmarking, and any time-sensitive operations.


## Current time

Getting the current time is fundamental for most time-related operations.  
The `time.Now()` function returns the current local time instant.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Printf("Current time: %v\n", now)
    fmt.Printf("Unix timestamp: %d\n", now.Unix())
    fmt.Printf("Nanoseconds: %d\n", now.UnixNano())
}
```

This example demonstrates basic time retrieval and shows different representations  
of the same moment. Unix timestamps are commonly used for storage and networking,  
while nanosecond precision is useful for high-resolution timing measurements.  


## Creating specific dates

Specific dates can be created using `time.Date()` function which takes year,  
month, day, hour, minute, second, nanosecond, and location parameters.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a specific date
    birthday := time.Date(1995, time.March, 15, 14, 30, 0, 0, time.UTC)
    fmt.Printf("Birthday: %v\n", birthday)
    
    // Create date with local timezone
    local := time.Date(2024, time.December, 25, 9, 0, 0, 0, time.Local)
    fmt.Printf("Christmas: %v\n", local)
    
    // Beginning of day
    today := time.Now()
    startOfDay := time.Date(today.Year(), today.Month(), today.Day(), 0, 0, 0, 0, today.Location())
    fmt.Printf("Start of today: %v\n", startOfDay)
}
```

This approach provides precise control over date creation and is essential for  
scheduling applications, data analysis with specific dates, and when working  
with historical data or future events that need exact timing specifications.  


## Time formatting

Go uses a unique reference time for formatting: "Mon Jan 2 15:04:05 MST 2006".  
This makes formatting intuitive once you remember the reference pattern.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    
    // Standard formats
    fmt.Printf("RFC3339: %s\n", now.Format(time.RFC3339))
    fmt.Printf("Kitchen: %s\n", now.Format(time.Kitchen))
    fmt.Printf("Stamp: %s\n", now.Format(time.Stamp))
    
    // Custom formats
    fmt.Printf("Custom 1: %s\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("Custom 2: %s\n", now.Format("January 2, 2006"))
    fmt.Printf("Custom 3: %s\n", now.Format("02/01/2006 3:04 PM"))
}
```

The reference time components make formatting predictable: 2006 for year,  
01 for month, 02 for day, 15 for 24-hour format, 04 for minute, 05 for second.  
This system eliminates confusion about format specifiers found in other languages.  


## Time parsing

Parsing strings into time values uses the same reference format as formatting.  
The `time.Parse()` function converts strings to `time.Time` values.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Parse various formats
    layout1 := "2006-01-02 15:04:05"
    str1 := "2024-03-15 10:30:45"
    parsed1, err := time.Parse(layout1, str1)
    if err != nil {
        fmt.Printf("Error: %v\n", err)
        return
    }
    fmt.Printf("Parsed 1: %v\n", parsed1)
    
    // Parse RFC format
    str2 := "2024-03-15T10:30:45Z"
    parsed2, err := time.Parse(time.RFC3339, str2)
    if err == nil {
        fmt.Printf("Parsed 2: %v\n", parsed2)
    }
    
    // Parse date only
    layout3 := "01/02/2006"
    str3 := "12/25/2024"
    parsed3, err := time.Parse(layout3, str3)
    if err == nil {
        fmt.Printf("Parsed 3: %v\n", parsed3)
    }
}
```

Parsing is essential for processing user input, reading configuration files,  
and working with external APIs that return time data in string format.  
Always handle parsing errors as malformed input is common in real applications.  


## Time components

Extract individual components from time values such as year, month, day,  
hour, minute, and second for detailed time analysis and processing.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    
    fmt.Printf("Year: %d\n", now.Year())
    fmt.Printf("Month: %s (%d)\n", now.Month(), int(now.Month()))
    fmt.Printf("Day: %d\n", now.Day())
    fmt.Printf("Hour: %d\n", now.Hour())
    fmt.Printf("Minute: %d\n", now.Minute())
    fmt.Printf("Second: %d\n", now.Second())
    fmt.Printf("Nanosecond: %d\n", now.Nanosecond())
    fmt.Printf("Weekday: %s\n", now.Weekday())
    fmt.Printf("Year day: %d\n", now.YearDay())
}
```

Component extraction is useful for conditional logic based on time periods,  
generating reports grouped by time units, and implementing business rules  
that depend on specific calendar dates or times of day.  


## Duration basics

Duration represents elapsed time between two instants. Go provides constants  
for common time units and supports duration arithmetic operations.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Duration constants
    fmt.Printf("Nanosecond: %v\n", time.Nanosecond)
    fmt.Printf("Microsecond: %v\n", time.Microsecond)
    fmt.Printf("Millisecond: %v\n", time.Millisecond)
    fmt.Printf("Second: %v\n", time.Second)
    fmt.Printf("Minute: %v\n", time.Minute)
    fmt.Printf("Hour: %v\n", time.Hour)
    
    // Custom durations
    duration1 := 5 * time.Minute + 30 * time.Second
    fmt.Printf("Duration 1: %v\n", duration1)
    
    duration2 := 2*time.Hour + 15*time.Minute
    fmt.Printf("Duration 2: %v\n", duration2)
    
    // Duration conversions
    fmt.Printf("Total seconds: %.0f\n", duration1.Seconds())
    fmt.Printf("Total minutes: %.2f\n", duration1.Minutes())
}
```

Durations are fundamental for timeouts, intervals, scheduling tasks, and  
measuring elapsed time. They provide type-safe time arithmetic and prevent  
common errors associated with time unit confusion.  


## Time arithmetic

Perform addition, subtraction, and comparison operations with time values.  
These operations are essential for scheduling and time-based calculations.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Printf("Current time: %v\n", now)
    
    // Add duration
    future := now.Add(2 * time.Hour)
    fmt.Printf("2 hours later: %v\n", future)
    
    // Subtract duration
    past := now.Add(-3 * time.Hour)
    fmt.Printf("3 hours ago: %v\n", past)
    
    // Add date components
    nextWeek := now.AddDate(0, 0, 7)
    fmt.Printf("Next week: %v\n", nextWeek)
    
    nextMonth := now.AddDate(0, 1, 0)
    fmt.Printf("Next month: %v\n", nextMonth)
    
    nextYear := now.AddDate(1, 0, 0)
    fmt.Printf("Next year: %v\n", nextYear)
    
    // Calculate difference
    diff := future.Sub(now)
    fmt.Printf("Difference: %v\n", diff)
}
```

Time arithmetic enables scheduling applications, calculating ages, determining  
deadlines, and implementing time-based business logic. The `AddDate` method  
handles month and year boundaries correctly, including leap years.  


## Time comparison

Compare time values to determine chronological relationships. This is  
crucial for sorting, filtering, and conditional logic based on time.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    earlier := now.Add(-1 * time.Hour)
    later := now.Add(1 * time.Hour)
    
    fmt.Printf("Now: %v\n", now.Format("15:04:05"))
    fmt.Printf("Earlier: %v\n", earlier.Format("15:04:05"))
    fmt.Printf("Later: %v\n", later.Format("15:04:05"))
    
    // Comparison methods
    fmt.Printf("earlier.Before(now): %t\n", earlier.Before(now))
    fmt.Printf("later.After(now): %t\n", later.After(now))
    fmt.Printf("now.Equal(now): %t\n", now.Equal(now))
    
    // Using in conditions
    if later.After(now) {
        fmt.Println("Later is indeed after now")
    }
    
    // Find earliest and latest
    times := []time.Time{now, earlier, later}
    earliest := times[0]
    latest := times[0]
    
    for _, t := range times {
        if t.Before(earliest) {
            earliest = t
        }
        if t.After(latest) {
            latest = t
        }
    }
    
    fmt.Printf("Earliest: %v\n", earliest.Format("15:04:05"))
    fmt.Printf("Latest: %v\n", latest.Format("15:04:05"))
}
```

Time comparison is fundamental for implementing queues, scheduling systems,  
data validation, and any application requiring temporal ordering or filtering  
based on time ranges.  


## Sleep and delays

Pause program execution for specified durations using `time.Sleep()`.  
This is useful for rate limiting, timeouts, and creating delays.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("Starting...")
    start := time.Now()
    
    // Sleep for 2 seconds
    fmt.Println("Sleeping for 2 seconds...")
    time.Sleep(2 * time.Second)
    
    elapsed := time.Since(start)
    fmt.Printf("Elapsed time: %v\n", elapsed)
    
    // Sleep for milliseconds
    fmt.Println("Quick nap...")
    time.Sleep(500 * time.Millisecond)
    
    fmt.Println("Done!")
}
```

Sleep is commonly used in polling loops, implementing backoff strategies,  
demonstration programs, and anywhere you need to introduce controlled delays  
in program execution without blocking other goroutines unnecessarily.  


## Time zones

Handle different time zones using `time.Location`. Go provides methods to  
work with UTC, local time, and specific geographic time zones.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    
    // UTC time
    utc := now.UTC()
    fmt.Printf("UTC: %v\n", utc.Format("2006-01-02 15:04:05 MST"))
    
    // Local time
    local := now.Local()
    fmt.Printf("Local: %v\n", local.Format("2006-01-02 15:04:05 MST"))
    
    // Specific time zones
    tokyo, _ := time.LoadLocation("Asia/Tokyo")
    tokyoTime := now.In(tokyo)
    fmt.Printf("Tokyo: %v\n", tokyoTime.Format("2006-01-02 15:04:05 MST"))
    
    newYork, _ := time.LoadLocation("America/New_York")
    nyTime := now.In(newYork)
    fmt.Printf("New York: %v\n", nyTime.Format("2006-01-02 15:04:05 MST"))
    
    london, _ := time.LoadLocation("Europe/London")
    londonTime := now.In(london)
    fmt.Printf("London: %v\n", londonTime.Format("2006-01-02 15:04:05 MST"))
}
```

Time zone handling is crucial for global applications, scheduling across  
regions, displaying local times to users, and ensuring accurate timestamps  
in distributed systems with components in different geographic locations.  


## Timer functionality

Timers execute code after a specified duration. They're useful for timeouts,  
scheduled tasks, and implementing time-based triggers in applications.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("Timer examples")
    
    // Basic timer
    timer1 := time.NewTimer(2 * time.Second)
    fmt.Println("Waiting for timer1...")
    <-timer1.C
    fmt.Println("Timer1 expired!")
    
    // Timer with early stop
    timer2 := time.NewTimer(5 * time.Second)
    go func() {
        <-timer2.C
        fmt.Println("Timer2 expired!")
    }()
    
    // Stop timer2 before it expires
    time.Sleep(1 * time.Second)
    if timer2.Stop() {
        fmt.Println("Timer2 stopped before expiring")
    }
    
    // Using time.After for simpler cases
    fmt.Println("Using time.After...")
    select {
    case <-time.After(1 * time.Second):
        fmt.Println("1 second timeout reached")
    }
}
```

Timers are essential for implementing timeouts in network operations,  
creating scheduled cleanup tasks, implementing rate limiting, and building  
responsive applications that don't wait indefinitely for operations.  


## Ticker for intervals

Tickers repeatedly send time values on a channel at regular intervals.  
They're perfect for periodic tasks and regular event generation.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create ticker that ticks every second
    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()
    
    start := time.Now()
    count := 0
    
    fmt.Println("Starting ticker (will run for 5 seconds)...")
    
    for {
        select {
        case tickTime := <-ticker.C:
            count++
            fmt.Printf("Tick %d at %v\n", count, tickTime.Format("15:04:05"))
            
            if time.Since(start) > 5*time.Second {
                fmt.Println("Stopping ticker")
                return
            }
        }
    }
}
```

Tickers enable implementing heartbeats, periodic data collection, regular  
status updates, monitoring systems, and any application requiring consistent  
time-based intervals for ongoing operations.  


## Measuring elapsed time

Accurately measure execution time for performance analysis, benchmarking,  
and monitoring application performance characteristics.  

```go
package main

import (
    "fmt"
    "time"
)

func expensiveOperation() {
    // Simulate work
    time.Sleep(100 * time.Millisecond)
    total := 0
    for i := 0; i < 1000000; i++ {
        total += i
    }
}

func main() {
    // Method 1: Using time.Now()
    start := time.Now()
    expensiveOperation()
    elapsed := time.Since(start)
    fmt.Printf("Operation took: %v\n", elapsed)
    
    // Method 2: Manual calculation
    start2 := time.Now()
    expensiveOperation()
    end := time.Now()
    elapsed2 := end.Sub(start2)
    fmt.Printf("Operation took (manual): %v\n", elapsed2)
    
    // Method 3: Multiple measurements
    var totalTime time.Duration
    iterations := 5
    
    for i := 0; i < iterations; i++ {
        start := time.Now()
        expensiveOperation()
        totalTime += time.Since(start)
    }
    
    avgTime := totalTime / time.Duration(iterations)
    fmt.Printf("Average time over %d iterations: %v\n", iterations, avgTime)
}
```

Performance measurement is crucial for optimization, monitoring system health,  
identifying bottlenecks, and ensuring applications meet performance requirements  
in production environments.  


## Working with months

Handle month-specific operations including month arithmetic, beginning/end  
of months, and month-based calculations for business applications.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Printf("Current date: %v\n", now.Format("2006-01-02"))
    
    // First day of current month
    firstDay := time.Date(now.Year(), now.Month(), 1, 0, 0, 0, 0, now.Location())
    fmt.Printf("First day of month: %v\n", firstDay.Format("2006-01-02"))
    
    // Last day of current month
    lastDay := firstDay.AddDate(0, 1, -1)
    fmt.Printf("Last day of month: %v\n", lastDay.Format("2006-01-02"))
    
    // Beginning of next month
    nextMonth := firstDay.AddDate(0, 1, 0)
    fmt.Printf("Next month starts: %v\n", nextMonth.Format("2006-01-02"))
    
    // Month names
    fmt.Printf("Current month: %s\n", now.Month().String())
    fmt.Printf("Next month: %s\n", nextMonth.Month().String())
    
    // Days in current month
    daysInMonth := lastDay.Day()
    fmt.Printf("Days in current month: %d\n", daysInMonth)
}
```

Month operations are essential for financial applications, billing systems,  
report generation, and any business logic that operates on monthly cycles  
or requires month-boundary calculations.  


## Working with weeks

Calculate week-related information including week numbers, beginning/end  
of weeks, and week-based date arithmetic for scheduling applications.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Printf("Current date: %v (%s)\n", now.Format("2006-01-02"), now.Weekday())
    
    // Days since Monday (0 = Monday, 6 = Sunday)
    daysSinceMonday := int(now.Weekday()+6) % 7
    
    // Beginning of week (Monday)
    weekStart := now.AddDate(0, 0, -daysSinceMonday)
    weekStart = time.Date(weekStart.Year(), weekStart.Month(), weekStart.Day(), 0, 0, 0, 0, weekStart.Location())
    fmt.Printf("Week starts (Monday): %v\n", weekStart.Format("2006-01-02"))
    
    // End of week (Sunday)
    weekEnd := weekStart.AddDate(0, 0, 6)
    weekEnd = time.Date(weekEnd.Year(), weekEnd.Month(), weekEnd.Day(), 23, 59, 59, 0, weekEnd.Location())
    fmt.Printf("Week ends (Sunday): %v\n", weekEnd.Format("2006-01-02"))
    
    // Week number
    _, week := now.ISOWeek()
    fmt.Printf("ISO week number: %d\n", week)
    
    // Next Monday
    daysUntilMonday := (7 - daysSinceMonday) % 7
    if daysUntilMonday == 0 {
        daysUntilMonday = 7
    }
    nextMonday := now.AddDate(0, 0, daysUntilMonday)
    fmt.Printf("Next Monday: %v\n", nextMonday.Format("2006-01-02"))
}
```

Week calculations are important for scheduling systems, work planning,  
educational applications with academic weeks, and business applications  
that operate on weekly cycles or reporting periods.  


## Age calculation

Calculate ages and time differences between dates for applications  
requiring age verification, anniversary tracking, or temporal analysis.  

```go
package main

import (
    "fmt"
    "time"
)

func calculateAge(birthdate time.Time) (years, months, days int) {
    now := time.Now()
    
    // Calculate years
    years = now.Year() - birthdate.Year()
    
    // Adjust for birthday not yet reached this year
    if now.Month() < birthdate.Month() || (now.Month() == birthdate.Month() && now.Day() < birthdate.Day()) {
        years--
    }
    
    // Calculate months and days
    if now.Day() >= birthdate.Day() {
        months = int(now.Month()) - int(birthdate.Month())
        days = now.Day() - birthdate.Day()
    } else {
        months = int(now.Month()) - int(birthdate.Month()) - 1
        // Days in previous month
        prevMonth := now.AddDate(0, -1, 0)
        daysInPrevMonth := time.Date(prevMonth.Year(), prevMonth.Month()+1, 0, 0, 0, 0, 0, time.UTC).Day()
        days = daysInPrevMonth - birthdate.Day() + now.Day()
    }
    
    if months < 0 {
        months += 12
        years--
    }
    
    return years, months, days
}

func main() {
    // Example birthdates
    birthdate1 := time.Date(1990, time.March, 15, 0, 0, 0, 0, time.UTC)
    birthdate2 := time.Date(2000, time.December, 25, 0, 0, 0, 0, time.UTC)
    
    years1, months1, days1 := calculateAge(birthdate1)
    years2, months2, days2 := calculateAge(birthdate2)
    
    fmt.Printf("Person 1 (born %v): %d years, %d months, %d days old\n", 
               birthdate1.Format("2006-01-02"), years1, months1, days1)
    fmt.Printf("Person 2 (born %v): %d years, %d months, %d days old\n", 
               birthdate2.Format("2006-01-02"), years2, months2, days2)
    
    // Time until next birthday
    now := time.Now()
    nextBirthday := time.Date(now.Year(), birthdate1.Month(), birthdate1.Day(), 0, 0, 0, 0, time.UTC)
    if nextBirthday.Before(now) {
        nextBirthday = nextBirthday.AddDate(1, 0, 0)
    }
    
    daysUntilBirthday := nextBirthday.Sub(now).Hours() / 24
    fmt.Printf("Days until next birthday: %.0f\n", daysUntilBirthday)
}
```

Age calculations are vital for demographic analysis, eligibility verification,  
insurance applications, and any system requiring accurate temporal distance  
measurements between life events or important dates.  


## Date ranges

Work with date ranges to check if dates fall within periods, calculate  
overlaps, and implement date-based filtering and validation.  

```go
package main

import (
    "fmt"
    "time"
)

type DateRange struct {
    Start time.Time
    End   time.Time
}

func (dr DateRange) Contains(t time.Time) bool {
    return !t.Before(dr.Start) && !t.After(dr.End)
}

func (dr DateRange) Overlaps(other DateRange) bool {
    return dr.Start.Before(other.End) && other.Start.Before(dr.End)
}

func (dr DateRange) Duration() time.Duration {
    return dr.End.Sub(dr.Start)
}

func main() {
    // Create date ranges
    vacation := DateRange{
        Start: time.Date(2024, time.July, 15, 0, 0, 0, 0, time.UTC),
        End:   time.Date(2024, time.July, 29, 23, 59, 59, 0, time.UTC),
    }
    
    conference := DateRange{
        Start: time.Date(2024, time.July, 20, 0, 0, 0, 0, time.UTC),
        End:   time.Date(2024, time.July, 22, 23, 59, 59, 0, time.UTC),
    }
    
    project := DateRange{
        Start: time.Date(2024, time.August, 1, 0, 0, 0, 0, time.UTC),
        End:   time.Date(2024, time.September, 15, 23, 59, 59, 0, time.UTC),
    }
    
    fmt.Printf("Vacation: %v to %v\n", vacation.Start.Format("2006-01-02"), vacation.End.Format("2006-01-02"))
    fmt.Printf("Conference: %v to %v\n", conference.Start.Format("2006-01-02"), conference.End.Format("2006-01-02"))
    fmt.Printf("Project: %v to %v\n", project.Start.Format("2006-01-02"), project.End.Format("2006-01-02"))
    
    // Check if conference is during vacation
    fmt.Printf("Conference during vacation: %t\n", vacation.Overlaps(conference))
    
    // Check if specific date is in vacation
    checkDate := time.Date(2024, time.July, 25, 12, 0, 0, 0, time.UTC)
    fmt.Printf("July 25 is during vacation: %t\n", vacation.Contains(checkDate))
    
    // Calculate durations
    fmt.Printf("Vacation duration: %.0f days\n", vacation.Duration().Hours()/24)
    fmt.Printf("Project duration: %.0f days\n", project.Duration().Hours()/24)
}
```

Date ranges are crucial for reservation systems, project management,  
scheduling applications, and any business logic requiring period-based  
validation, conflict detection, or temporal relationship analysis.  


## Business days calculation

Calculate business days excluding weekends and optionally holidays,  
essential for financial applications and business scheduling systems.  

```go
package main

import (
    "fmt"
    "time"
)

func isWeekend(t time.Time) bool {
    weekday := t.Weekday()
    return weekday == time.Saturday || weekday == time.Sunday
}

func addBusinessDays(start time.Time, days int) time.Time {
    current := start
    businessDaysAdded := 0
    
    for businessDaysAdded < days {
        current = current.AddDate(0, 0, 1)
        if !isWeekend(current) {
            businessDaysAdded++
        }
    }
    
    return current
}

func businessDaysBetween(start, end time.Time) int {
    if start.After(end) {
        return 0
    }
    
    businessDays := 0
    current := start
    
    for current.Before(end) {
        if !isWeekend(current) {
            businessDays++
        }
        current = current.AddDate(0, 0, 1)
    }
    
    return businessDays
}

func main() {
    start := time.Date(2024, time.March, 15, 0, 0, 0, 0, time.UTC) // Friday
    fmt.Printf("Start date: %v (%s)\n", start.Format("2006-01-02"), start.Weekday())
    
    // Add 5 business days
    result := addBusinessDays(start, 5)
    fmt.Printf("After 5 business days: %v (%s)\n", result.Format("2006-01-02"), result.Weekday())
    
    // Calculate business days between two dates
    end := time.Date(2024, time.March, 25, 0, 0, 0, 0, time.UTC)
    businessDays := businessDaysBetween(start, end)
    fmt.Printf("Business days from %v to %v: %d\n", 
               start.Format("2006-01-02"), end.Format("2006-01-02"), businessDays)
    
    // Next business day
    nextBusinessDay := addBusinessDays(time.Now(), 1)
    fmt.Printf("Next business day: %v (%s)\n", 
               nextBusinessDay.Format("2006-01-02"), nextBusinessDay.Weekday())
}
```

Business day calculations are essential for financial trading systems,  
project delivery dates, SLA calculations, and any application requiring  
work-day-based scheduling that excludes weekends and holidays.  


## Recurring events

Calculate recurring events like daily, weekly, or monthly schedules.  
This is fundamental for calendar applications and task scheduling.  

```go
package main

import (
    "fmt"
    "time"
)

func generateDailySchedule(start time.Time, days, occurrences int) []time.Time {
    var schedule []time.Time
    current := start
    
    for i := 0; i < occurrences; i++ {
        schedule = append(schedule, current)
        current = current.AddDate(0, 0, days)
    }
    
    return schedule
}

func generateWeeklySchedule(start time.Time, weeks, occurrences int) []time.Time {
    var schedule []time.Time
    current := start
    
    for i := 0; i < occurrences; i++ {
        schedule = append(schedule, current)
        current = current.AddDate(0, 0, weeks*7)
    }
    
    return schedule
}

func generateMonthlySchedule(start time.Time, months, occurrences int) []time.Time {
    var schedule []time.Time
    current := start
    
    for i := 0; i < occurrences; i++ {
        schedule = append(schedule, current)
        current = current.AddDate(0, months, 0)
    }
    
    return schedule
}

func main() {
    start := time.Date(2024, time.March, 15, 10, 0, 0, 0, time.UTC)
    
    // Daily meetings for 5 days
    fmt.Println("Daily meetings:")
    daily := generateDailySchedule(start, 1, 5)
    for i, event := range daily {
        fmt.Printf("  Meeting %d: %v\n", i+1, event.Format("2006-01-02 15:04"))
    }
    
    // Weekly team meetings for 4 weeks
    fmt.Println("\nWeekly team meetings:")
    weekly := generateWeeklySchedule(start, 1, 4)
    for i, event := range weekly {
        fmt.Printf("  Week %d: %v (%s)\n", i+1, event.Format("2006-01-02 15:04"), event.Weekday())
    }
    
    // Monthly reviews for 6 months
    fmt.Println("\nMonthly reviews:")
    monthly := generateMonthlySchedule(start, 1, 6)
    for i, event := range monthly {
        fmt.Printf("  Review %d: %v\n", i+1, event.Format("2006-01-02 15:04"))
    }
    
    // Bi-weekly schedule (every 2 weeks)
    fmt.Println("\nBi-weekly check-ins:")
    biweekly := generateWeeklySchedule(start, 2, 3)
    for i, event := range biweekly {
        fmt.Printf("  Check-in %d: %v\n", i+1, event.Format("2006-01-02 15:04"))
    }
}
```

Recurring event generation is essential for calendar applications, scheduling  
systems, reminder services, and any application requiring predictable  
time-based event patterns and automated scheduling capabilities.  


## Time zone conversion

Convert times between different time zones accurately, handling daylight  
saving time transitions and regional time differences.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

func convertTime(t time.Time, fromZone, toZone string) (time.Time, error) {
    // Load source timezone
    fromLoc, err := time.LoadLocation(fromZone)
    if err != nil {
        return time.Time{}, err
    }
    
    // Load destination timezone
    toLoc, err := time.LoadLocation(toZone)
    if err != nil {
        return time.Time{}, err
    }
    
    // Convert time to source timezone first, then to destination
    timeInFrom := t.In(fromLoc)
    timeInTo := timeInFrom.In(toLoc)
    
    return timeInTo, nil
}

func showWorldTimes(baseTime time.Time) {
    zones := []string{
        "UTC",
        "America/New_York",
        "America/Los_Angeles",
        "Europe/London",
        "Europe/Paris",
        "Asia/Tokyo",
        "Australia/Sydney",
        "Asia/Dubai",
    }
    
    fmt.Printf("World times for %v UTC:\n", baseTime.UTC().Format("2006-01-02 15:04:05"))
    fmt.Println(strings.Repeat("-", 50))
    
    for _, zone := range zones {
        if loc, err := time.LoadLocation(zone); err == nil {
            localTime := baseTime.In(loc)
            fmt.Printf("%-20s: %s\n", zone, localTime.Format("2006-01-02 15:04:05 MST"))
        }
    }
}

func main() {
    // Current time in UTC
    now := time.Now().UTC()
    
    // Show current time in multiple zones
    showWorldTimes(now)
    
    // Convert specific meeting time
    fmt.Println("\nMeeting time conversions:")
    meetingUTC := time.Date(2024, time.March, 15, 14, 30, 0, 0, time.UTC)
    
    // Convert to different zones
    zones := []string{"America/New_York", "Europe/London", "Asia/Tokyo"}
    for _, zone := range zones {
        if convertedTime, err := convertTime(meetingUTC, "UTC", zone); err == nil {
            fmt.Printf("%-20s: %s\n", zone, convertedTime.Format("2006-01-02 15:04:05 MST"))
        }
    }
    
    // Calculate time differences
    fmt.Println("\nTime zone offsets from UTC:")
    for _, zone := range zones {
        if loc, err := time.LoadLocation(zone); err == nil {
            localTime := now.In(loc)
            _, offset := localTime.Zone()
            hours := offset / 3600
            fmt.Printf("%-20s: %+d hours\n", zone, hours)
        }
    }
}
```

Time zone conversion is critical for global applications, international  
scheduling, distributed systems coordination, and ensuring accurate  
time representation across different geographic regions and user locations.  


## Daylight saving time

Handle daylight saving time transitions, detect DST periods, and manage  
time calculations that span DST boundaries properly.  

```go
package main

import (
    "fmt"
    "time"
)

func isDST(t time.Time, location *time.Location) bool {
    timeInLocation := t.In(location)
    _, offset := timeInLocation.Zone()
    
    // Compare with standard time offset (January)
    jan := time.Date(t.Year(), time.January, 1, 12, 0, 0, 0, location)
    _, janOffset := jan.Zone()
    
    return offset != janOffset
}

func findDSTTransitions(year int, location *time.Location) (start, end time.Time) {
    // Check each day of the year for DST transitions
    var dstStart, dstEnd time.Time
    
    for month := time.January; month <= time.December; month++ {
        daysInMonth := time.Date(year, month+1, 0, 0, 0, 0, 0, time.UTC).Day()
        
        for day := 1; day <= daysInMonth; day++ {
            current := time.Date(year, month, day, 12, 0, 0, 0, location)
            previous := current.AddDate(0, 0, -1)
            
            currentDST := isDST(current, location)
            previousDST := isDST(previous, location)
            
            if !previousDST && currentDST && dstStart.IsZero() {
                dstStart = current
            } else if previousDST && !currentDST && dstEnd.IsZero() {
                dstEnd = current
            }
        }
    }
    
    return dstStart, dstEnd
}

func main() {
    // US Eastern timezone
    est, _ := time.LoadLocation("America/New_York")
    
    // Example times around DST transitions
    fmt.Println("Daylight Saving Time Analysis")
    fmt.Println("=============================")
    
    // Spring forward (March)
    spring := time.Date(2024, time.March, 10, 6, 0, 0, 0, est)
    fmt.Printf("Before spring DST: %s (DST: %t)\n", 
               spring.Format("2006-01-02 15:04:05 MST"), isDST(spring, est))
    
    springAfter := time.Date(2024, time.March, 10, 8, 0, 0, 0, est)
    fmt.Printf("After spring DST:  %s (DST: %t)\n", 
               springAfter.Format("2006-01-02 15:04:05 MST"), isDST(springAfter, est))
    
    // Fall back (November)
    fall := time.Date(2024, time.November, 3, 5, 0, 0, 0, est)
    fmt.Printf("Before fall DST:   %s (DST: %t)\n", 
               fall.Format("2006-01-02 15:04:05 MST"), isDST(fall, est))
    
    fallAfter := time.Date(2024, time.November, 3, 7, 0, 0, 0, est)
    fmt.Printf("After fall DST:    %s (DST: %t)\n", 
               fallAfter.Format("2006-01-02 15:04:05 MST"), isDST(fallAfter, est))
    
    // Find DST transitions for the year
    dstStart, dstEnd := findDSTTransitions(2024, est)
    if !dstStart.IsZero() && !dstEnd.IsZero() {
        fmt.Printf("\n2024 DST period: %s to %s\n", 
                   dstStart.Format("January 2"), dstEnd.AddDate(0, 0, -1).Format("January 2"))
    }
    
    // Duration calculation spanning DST
    fmt.Println("\nDuration calculations spanning DST:")
    before := time.Date(2024, time.March, 9, 12, 0, 0, 0, est)
    after := time.Date(2024, time.March, 11, 12, 0, 0, 0, est)
    duration := after.Sub(before)
    fmt.Printf("48 hours spanning spring DST: %v\n", duration)
}
```

DST handling is crucial for scheduling applications, time-based calculations  
across seasons, meeting scheduling systems, and any application operating  
in regions that observe daylight saving time transitions.  


## Working with milliseconds

Handle high-precision time operations using milliseconds and microseconds  
for applications requiring fine-grained timing and performance measurement.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    
    // Extract milliseconds and microseconds
    millis := now.UnixMilli()
    micros := now.UnixMicro()
    nanos := now.UnixNano()
    
    fmt.Printf("Current time: %v\n", now.Format("2006-01-02 15:04:05.000"))
    fmt.Printf("Milliseconds since epoch: %d\n", millis)
    fmt.Printf("Microseconds since epoch: %d\n", micros)
    fmt.Printf("Nanoseconds since epoch: %d\n", nanos)
    
    // Create time from milliseconds
    timeFromMillis := time.UnixMilli(millis)
    fmt.Printf("Recreated from millis: %v\n", timeFromMillis.Format("2006-01-02 15:04:05.000"))
    
    // Millisecond arithmetic
    start := time.Now()
    time.Sleep(150 * time.Millisecond)
    elapsed := time.Since(start)
    
    fmt.Printf("Elapsed time: %v\n", elapsed)
    fmt.Printf("Elapsed milliseconds: %.0f\n", elapsed.Seconds()*1000)
    
    // High precision formatting
    precise := time.Now()
    fmt.Printf("Precise time: %s\n", precise.Format("2006-01-02 15:04:05.000000"))
    
    // Millisecond intervals
    ticker := time.NewTicker(250 * time.Millisecond)
    defer ticker.Stop()
    
    fmt.Println("Millisecond ticker (3 ticks):")
    count := 0
    for range ticker.C {
        count++
        fmt.Printf("Tick %d at: %s\n", count, time.Now().Format("15:04:05.000"))
        if count >= 3 {
            break
        }
    }
}
```

Millisecond precision is essential for performance monitoring, animation  
timing, real-time applications, and systems requiring precise timing  
measurements or high-frequency event processing.  


## Working with epochs

Handle Unix epoch time conversions, different epoch formats, and epoch-based  
calculations commonly used in system programming and data exchange.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    
    // Different epoch representations
    fmt.Printf("Current time: %v\n", now.Format("2006-01-02 15:04:05"))
    fmt.Printf("Unix timestamp (seconds): %d\n", now.Unix())
    fmt.Printf("Unix timestamp (milliseconds): %d\n", now.UnixMilli())
    fmt.Printf("Unix timestamp (microseconds): %d\n", now.UnixMicro())
    fmt.Printf("Unix timestamp (nanoseconds): %d\n", now.UnixNano())
    
    // Convert from epoch timestamps
    epoch1 := int64(1678886400) // March 15, 2023 8:00:00 UTC
    time1 := time.Unix(epoch1, 0)
    fmt.Printf("From epoch %d: %v\n", epoch1, time1.Format("2006-01-02 15:04:05 UTC"))
    
    // Convert from milliseconds epoch
    epochMillis := int64(1678886400000)
    time2 := time.UnixMilli(epochMillis)
    fmt.Printf("From epoch millis %d: %v\n", epochMillis, time2.Format("2006-01-02 15:04:05 UTC"))
    
    // Epoch boundaries
    unixEpoch := time.Unix(0, 0).UTC()
    fmt.Printf("Unix epoch start: %v\n", unixEpoch.Format("2006-01-02 15:04:05 UTC"))
    
    // Year 2038 problem boundary (32-bit systems)
    y2k38 := time.Unix(2147483647, 0).UTC()
    fmt.Printf("Year 2038 boundary: %v\n", y2k38.Format("2006-01-02 15:04:05 UTC"))
    
    // Calculate days since epoch
    daysSinceEpoch := now.Sub(unixEpoch).Hours() / 24
    fmt.Printf("Days since Unix epoch: %.0f\n", daysSinceEpoch)
    
    // Epoch time arithmetic
    oneWeekAgo := now.AddDate(0, 0, -7)
    epochDiff := now.Unix() - oneWeekAgo.Unix()
    fmt.Printf("Seconds in one week: %d\n", epochDiff)
}
```

Epoch time handling is fundamental for database storage, API communication,  
log file analysis, and interfacing with systems that use Unix timestamps  
for time representation and synchronization.  


## Time validation

Validate time inputs, check for valid dates, handle edge cases, and ensure  
time data integrity in applications processing user-provided time information.  

```go
package main

import (
    "fmt"
    "time"
)

func isValidDate(year, month, day int) bool {
    // Check basic ranges
    if year < 1 || year > 9999 {
        return false
    }
    if month < 1 || month > 12 {
        return false
    }
    if day < 1 {
        return false
    }
    
    // Check day against month limits
    date := time.Date(year, time.Month(month), day, 0, 0, 0, 0, time.UTC)
    return date.Year() == year && int(date.Month()) == month && date.Day() == day
}

func isValidTime(hour, minute, second int) bool {
    return hour >= 0 && hour <= 23 &&
           minute >= 0 && minute <= 59 &&
           second >= 0 && second <= 59
}

func isLeapYear(year int) bool {
    return year%4 == 0 && (year%100 != 0 || year%400 == 0)
}

func validateTimeString(layout, timeStr string) (time.Time, error) {
    parsed, err := time.Parse(layout, timeStr)
    if err != nil {
        return time.Time{}, fmt.Errorf("invalid time format: %v", err)
    }
    
    // Additional validation can be added here
    if parsed.Year() < 1900 || parsed.Year() > 2100 {
        return time.Time{}, fmt.Errorf("year out of reasonable range: %d", parsed.Year())
    }
    
    return parsed, nil
}

func main() {
    fmt.Println("Time Validation Examples")
    fmt.Println("========================")
    
    // Test valid dates
    testDates := []struct {
        year, month, day int
        expected         bool
    }{
        {2024, 2, 29, true},   // Valid leap year date
        {2023, 2, 29, false},  // Invalid leap year date
        {2024, 4, 31, false},  // April doesn't have 31 days
        {2024, 12, 31, true},  // Valid date
        {2024, 13, 1, false},  // Invalid month
        {2024, 1, 0, false},   // Invalid day
    }
    
    fmt.Println("Date validation tests:")
    for _, test := range testDates {
        result := isValidDate(test.year, test.month, test.day)
        status := "✓"
        if result != test.expected {
            status = "✗"
        }
        fmt.Printf("  %s %04d-%02d-%02d: %t (expected %t)\n", 
                   status, test.year, test.month, test.day, result, test.expected)
    }
    
    // Test time validation
    fmt.Println("\nTime validation tests:")
    testTimes := []struct {
        hour, minute, second int
        expected             bool
    }{
        {12, 30, 45, true},   // Valid time
        {24, 0, 0, false},    // Invalid hour
        {23, 60, 0, false},   // Invalid minute
        {12, 30, 60, false},  // Invalid second
        {0, 0, 0, true},      // Valid midnight
    }
    
    for _, test := range testTimes {
        result := isValidTime(test.hour, test.minute, test.second)
        status := "✓"
        if result != test.expected {
            status = "✗"
        }
        fmt.Printf("  %s %02d:%02d:%02d: %t (expected %t)\n", 
                   status, test.hour, test.minute, test.second, result, test.expected)
    }
    
    // Test leap years
    fmt.Println("\nLeap year tests:")
    leapYears := []int{2000, 2004, 2020, 2024}
    nonLeapYears := []int{1900, 2001, 2100, 2023}
    
    for _, year := range leapYears {
        fmt.Printf("  %d is leap year: %t\n", year, isLeapYear(year))
    }
    for _, year := range nonLeapYears {
        fmt.Printf("  %d is leap year: %t\n", year, isLeapYear(year))
    }
    
    // Test string validation
    fmt.Println("\nString parsing validation:")
    testStrings := []string{
        "2024-03-15 10:30:45",
        "2024-13-15 10:30:45", // Invalid month
        "not-a-date",
        "1800-01-01 00:00:00", // Too old
    }
    
    layout := "2006-01-02 15:04:05"
    for _, testStr := range testStrings {
        if parsed, err := validateTimeString(layout, testStr); err != nil {
            fmt.Printf("  ✗ '%s': %v\n", testStr, err)
        } else {
            fmt.Printf("  ✓ '%s': %v\n", testStr, parsed.Format(layout))
        }
    }
}
```

Time validation prevents data corruption, ensures application stability,  
improves user experience, and is essential for any system processing  
time-based input from users or external sources.  


## JSON time handling

Handle time serialization and deserialization with JSON, custom time formats,  
and ensure proper time representation in web APIs and data exchange.  

```go
package main

import (
    "encoding/json"
    "fmt"
    "time"
)

// Custom time type with JSON marshaling
type CustomTime struct {
    time.Time
}

func (ct CustomTime) MarshalJSON() ([]byte, error) {
    // Custom format for JSON output
    formatted := fmt.Sprintf("\"%s\"", ct.Format("2006-01-02T15:04:05Z"))
    return []byte(formatted), nil
}

func (ct *CustomTime) UnmarshalJSON(data []byte) error {
    // Remove quotes and parse
    str := string(data[1 : len(data)-1])
    parsed, err := time.Parse("2006-01-02T15:04:05Z", str)
    if err != nil {
        return err
    }
    ct.Time = parsed
    return nil
}

// Event struct with different time representations
type Event struct {
    ID          int        `json:"id"`
    Name        string     `json:"name"`
    CreatedAt   time.Time  `json:"created_at"`
    ScheduledAt CustomTime `json:"scheduled_at"`
    UpdatedAt   *time.Time `json:"updated_at,omitempty"`
}

func main() {
    fmt.Println("JSON Time Handling Examples")
    fmt.Println("===========================")
    
    // Create event with current time
    now := time.Now().UTC()
    future := now.Add(24 * time.Hour)
    
    event := Event{
        ID:          1,
        Name:        "Team Meeting",
        CreatedAt:   now,
        ScheduledAt: CustomTime{future},
        UpdatedAt:   &now,
    }
    
    // Marshal to JSON
    jsonData, err := json.MarshalIndent(event, "", "  ")
    if err != nil {
        fmt.Printf("Error marshaling: %v\n", err)
        return
    }
    
    fmt.Printf("Marshaled JSON:\n%s\n\n", string(jsonData))
    
    // Unmarshal from JSON
    var parsedEvent Event
    err = json.Unmarshal(jsonData, &parsedEvent)
    if err != nil {
        fmt.Printf("Error unmarshaling: %v\n", err)
        return
    }
    
    fmt.Printf("Unmarshaled Event:\n")
    fmt.Printf("  ID: %d\n", parsedEvent.ID)
    fmt.Printf("  Name: %s\n", parsedEvent.Name)
    fmt.Printf("  Created: %v\n", parsedEvent.CreatedAt.Format("2006-01-02 15:04:05"))
    fmt.Printf("  Scheduled: %v\n", parsedEvent.ScheduledAt.Format("2006-01-02 15:04:05"))
    if parsedEvent.UpdatedAt != nil {
        fmt.Printf("  Updated: %v\n", parsedEvent.UpdatedAt.Format("2006-01-02 15:04:05"))
    }
    
    // Handle different JSON time formats
    fmt.Println("\nParsing various JSON time formats:")
    
    timeFormats := []string{
        `"2024-03-15T10:30:45Z"`,                    // RFC3339
        `"2024-03-15T10:30:45.123Z"`,                // RFC3339 with milliseconds
        `"2024-03-15T10:30:45+02:00"`,               // RFC3339 with timezone
        `"Fri, 15 Mar 2024 10:30:45 GMT"`,           // RFC1123
    }
    
    layouts := []string{
        time.RFC3339,
        "2006-01-02T15:04:05.000Z",
        time.RFC3339,
        time.RFC1123,
    }
    
    for i, timeStr := range timeFormats {
        var t time.Time
        // Remove quotes for parsing
        cleanStr := timeStr[1 : len(timeStr)-1]
        parsed, err := time.Parse(layouts[i], cleanStr)
        if err != nil {
            fmt.Printf("  ✗ %s: %v\n", timeStr, err)
        } else {
            fmt.Printf("  ✓ %s -> %v\n", timeStr, parsed.Format("2006-01-02 15:04:05 MST"))
        }
    }
    
    // Array of events with time ranges
    events := []Event{
        {ID: 1, Name: "Morning Meeting", CreatedAt: now, ScheduledAt: CustomTime{now.Add(1 * time.Hour)}},
        {ID: 2, Name: "Lunch Break", CreatedAt: now, ScheduledAt: CustomTime{now.Add(4 * time.Hour)}},
        {ID: 3, Name: "Afternoon Review", CreatedAt: now, ScheduledAt: CustomTime{now.Add(6 * time.Hour)}},
    }
    
    eventsJSON, _ := json.MarshalIndent(events, "", "  ")
    fmt.Printf("\nEvents array JSON:\n%s\n", string(eventsJSON))
}
```

JSON time handling is essential for REST APIs, configuration files, data  
storage, and any application requiring time data exchange between systems  
or client-server communication with proper time representation.  


## Time-based caching

Implement time-based caching mechanisms with expiration, TTL (Time To Live),  
and automatic cleanup for efficient data management and performance optimization.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

type CacheItem struct {
    Value     interface{}
    ExpiresAt time.Time
}

type TimeCache struct {
    items map[string]CacheItem
    mutex sync.RWMutex
}

func NewTimeCache() *TimeCache {
    cache := &TimeCache{
        items: make(map[string]CacheItem),
    }
    
    // Start cleanup goroutine
    go cache.cleanup()
    return cache
}

func (tc *TimeCache) Set(key string, value interface{}, ttl time.Duration) {
    tc.mutex.Lock()
    defer tc.mutex.Unlock()
    
    tc.items[key] = CacheItem{
        Value:     value,
        ExpiresAt: time.Now().Add(ttl),
    }
}

func (tc *TimeCache) Get(key string) (interface{}, bool) {
    tc.mutex.RLock()
    defer tc.mutex.RUnlock()
    
    item, exists := tc.items[key]
    if !exists {
        return nil, false
    }
    
    if time.Now().After(item.ExpiresAt) {
        // Item expired, remove it
        tc.mutex.RUnlock()
        tc.mutex.Lock()
        delete(tc.items, key)
        tc.mutex.Unlock()
        tc.mutex.RLock()
        return nil, false
    }
    
    return item.Value, true
}

func (tc *TimeCache) cleanup() {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
    
    for range ticker.C {
        tc.mutex.Lock()
        now := time.Now()
        for key, item := range tc.items {
            if now.After(item.ExpiresAt) {
                delete(tc.items, key)
            }
        }
        tc.mutex.Unlock()
    }
}

func (tc *TimeCache) Size() int {
    tc.mutex.RLock()
    defer tc.mutex.RUnlock()
    return len(tc.items)
}

func main() {
    fmt.Println("Time-based Cache Example")
    fmt.Println("========================")
    
    cache := NewTimeCache()
    
    // Set items with different TTL values
    cache.Set("short", "expires quickly", 2*time.Second)
    cache.Set("medium", "expires later", 5*time.Second)
    cache.Set("long", "expires much later", 10*time.Second)
    
    fmt.Printf("Cache size after adding items: %d\n", cache.Size())
    
    // Test immediate retrieval
    fmt.Println("\nImmediate retrieval:")
    keys := []string{"short", "medium", "long", "nonexistent"}
    for _, key := range keys {
        if value, found := cache.Get(key); found {
            fmt.Printf("  %s: %v\n", key, value)
        } else {
            fmt.Printf("  %s: not found\n", key)
        }
    }
    
    // Wait and test again
    fmt.Println("\nWaiting 3 seconds...")
    time.Sleep(3 * time.Second)
    
    fmt.Println("After 3 seconds:")
    for _, key := range keys {
        if value, found := cache.Get(key); found {
            fmt.Printf("  %s: %v\n", key, value)
        } else {
            fmt.Printf("  %s: expired or not found\n", key)
        }
    }
    
    fmt.Printf("Cache size: %d\n", cache.Size())
    
    // Add new item and wait for medium to expire
    cache.Set("new", "fresh item", 3*time.Second)
    fmt.Println("\nWaiting 3 more seconds...")
    time.Sleep(3 * time.Second)
    
    fmt.Println("Final check:")
    for _, key := range []string{"short", "medium", "long", "new"} {
        if value, found := cache.Get(key); found {
            fmt.Printf("  %s: %v\n", key, value)
        } else {
            fmt.Printf("  %s: expired or not found\n", key)
        }
    }
    
    fmt.Printf("Final cache size: %d\n", cache.Size())
}
```

Time-based caching improves application performance, reduces database load,  
manages memory efficiently, and provides automatic data lifecycle management  
for frequently accessed but time-sensitive information.  


## Rate limiting

Implement rate limiting using time windows, token bucket algorithms, and  
time-based throttling to control API usage and resource consumption.  

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Simple rate limiter using sliding window
type RateLimiter struct {
    requests []time.Time
    limit    int
    window   time.Duration
    mutex    sync.Mutex
}

func NewRateLimiter(limit int, window time.Duration) *RateLimiter {
    return &RateLimiter{
        requests: make([]time.Time, 0),
        limit:    limit,
        window:   window,
    }
}

func (rl *RateLimiter) Allow() bool {
    rl.mutex.Lock()
    defer rl.mutex.Unlock()
    
    now := time.Now()
    cutoff := now.Add(-rl.window)
    
    // Remove old requests outside the window
    validRequests := make([]time.Time, 0, len(rl.requests))
    for _, req := range rl.requests {
        if req.After(cutoff) {
            validRequests = append(validRequests, req)
        }
    }
    rl.requests = validRequests
    
    // Check if we can accept a new request
    if len(rl.requests) < rl.limit {
        rl.requests = append(rl.requests, now)
        return true
    }
    
    return false
}

func (rl *RateLimiter) GetStats() (int, time.Duration) {
    rl.mutex.Lock()
    defer rl.mutex.Unlock()
    
    now := time.Now()
    cutoff := now.Add(-rl.window)
    
    validCount := 0
    var nextReset time.Time
    
    for _, req := range rl.requests {
        if req.After(cutoff) {
            validCount++
            if nextReset.IsZero() || req.Before(nextReset) {
                nextReset = req
            }
        }
    }
    
    if nextReset.IsZero() {
        return validCount, 0
    }
    
    return validCount, nextReset.Add(rl.window).Sub(now)
}

// Token bucket rate limiter
type TokenBucket struct {
    tokens     int
    maxTokens  int
    refillRate time.Duration
    lastRefill time.Time
    mutex      sync.Mutex
}

func NewTokenBucket(maxTokens int, refillRate time.Duration) *TokenBucket {
    return &TokenBucket{
        tokens:     maxTokens,
        maxTokens:  maxTokens,
        refillRate: refillRate,
        lastRefill: time.Now(),
    }
}

func (tb *TokenBucket) Take() bool {
    tb.mutex.Lock()
    defer tb.mutex.Unlock()
    
    tb.refill()
    
    if tb.tokens > 0 {
        tb.tokens--
        return true
    }
    
    return false
}

func (tb *TokenBucket) refill() {
    now := time.Now()
    elapsed := now.Sub(tb.lastRefill)
    tokensToAdd := int(elapsed / tb.refillRate)
    
    if tokensToAdd > 0 {
        tb.tokens += tokensToAdd
        if tb.tokens > tb.maxTokens {
            tb.tokens = tb.maxTokens
        }
        tb.lastRefill = now
    }
}

func main() {
    fmt.Println("Rate Limiting Examples")
    fmt.Println("======================")
    
    // Sliding window rate limiter: 5 requests per 10 seconds
    fmt.Println("1. Sliding Window Rate Limiter (5 requests/10s)")
    limiter := NewRateLimiter(5, 10*time.Second)
    
    // Test rapid requests
    fmt.Println("Making 8 rapid requests:")
    for i := 1; i <= 8; i++ {
        allowed := limiter.Allow()
        current, resetIn := limiter.GetStats()
        fmt.Printf("  Request %d: %s (current: %d/5, resets in: %v)\n", 
                   i, map[bool]string{true: "✓ Allowed", false: "✗ Denied"}[allowed], 
                   current, resetIn.Round(time.Second))
        time.Sleep(500 * time.Millisecond)
    }
    
    // Wait and test again
    fmt.Println("\nWaiting 6 seconds...")
    time.Sleep(6 * time.Second)
    
    fmt.Println("Making 3 more requests:")
    for i := 9; i <= 11; i++ {
        allowed := limiter.Allow()
        current, resetIn := limiter.GetStats()
        fmt.Printf("  Request %d: %s (current: %d/5, resets in: %v)\n", 
                   i, map[bool]string{true: "✓ Allowed", false: "✗ Denied"}[allowed], 
                   current, resetIn.Round(time.Second))
    }
    
    // Token bucket rate limiter
    fmt.Println("\n2. Token Bucket Rate Limiter (3 tokens, refill every 2s)")
    bucket := NewTokenBucket(3, 2*time.Second)
    
    // Consume all tokens quickly
    fmt.Println("Consuming tokens rapidly:")
    for i := 1; i <= 5; i++ {
        allowed := bucket.Take()
        fmt.Printf("  Take %d: %s\n", i, 
                   map[bool]string{true: "✓ Got token", false: "✗ No tokens"}[allowed])
        time.Sleep(200 * time.Millisecond)
    }
    
    // Wait for refill
    fmt.Println("\nWaiting 5 seconds for token refill...")
    time.Sleep(5 * time.Second)
    
    fmt.Println("After refill:")
    for i := 6; i <= 8; i++ {
        allowed := bucket.Take()
        fmt.Printf("  Take %d: %s\n", i, 
                   map[bool]string{true: "✓ Got token", false: "✗ No tokens"}[allowed])
    }
}
```

Rate limiting protects systems from abuse, ensures fair resource usage,  
prevents denial of service, and maintains service quality by controlling  
the rate of incoming requests and resource consumption.  


## Calendar operations

Perform calendar-specific operations like finding holidays, working days,  
calendar navigation, and date-based business logic for scheduling systems.  

```go
package main

import (
    "fmt"
    "time"
)

// Holiday represents a holiday with name and date
type Holiday struct {
    Name string
    Date time.Time
}

// Calendar provides calendar operations and holiday management
type Calendar struct {
    holidays []Holiday
}

func NewCalendar() *Calendar {
    return &Calendar{
        holidays: make([]Holiday, 0),
    }
}

func (c *Calendar) AddHoliday(name string, date time.Time) {
    c.holidays = append(c.holidays, Holiday{Name: name, Date: date})
}

func (c *Calendar) IsHoliday(date time.Time) (bool, string) {
    dateOnly := time.Date(date.Year(), date.Month(), date.Day(), 0, 0, 0, 0, date.Location())
    
    for _, holiday := range c.holidays {
        holidayDate := time.Date(holiday.Date.Year(), holiday.Date.Month(), holiday.Date.Day(), 0, 0, 0, 0, holiday.Date.Location())
        if dateOnly.Equal(holidayDate) {
            return true, holiday.Name
        }
    }
    return false, ""
}

func (c *Calendar) IsWorkingDay(date time.Time) bool {
    // Check if it's weekend
    if date.Weekday() == time.Saturday || date.Weekday() == time.Sunday {
        return false
    }
    
    // Check if it's a holiday
    if isHoliday, _ := c.IsHoliday(date); isHoliday {
        return false
    }
    
    return true
}

func (c *Calendar) NextWorkingDay(date time.Time) time.Time {
    next := date.AddDate(0, 0, 1)
    
    for !c.IsWorkingDay(next) {
        next = next.AddDate(0, 0, 1)
    }
    
    return next
}

func (c *Calendar) WorkingDaysInMonth(year int, month time.Month) []time.Time {
    var workingDays []time.Time
    
    // First day of the month
    firstDay := time.Date(year, month, 1, 0, 0, 0, 0, time.UTC)
    
    // Last day of the month
    lastDay := firstDay.AddDate(0, 1, -1)
    
    current := firstDay
    for current.Month() == month {
        if c.IsWorkingDay(current) {
            workingDays = append(workingDays, current)
        }
        current = current.AddDate(0, 0, 1)
    }
    
    return workingDays
}

func (c *Calendar) GetQuarter(date time.Time) int {
    month := int(date.Month())
    return (month-1)/3 + 1
}

func (c *Calendar) QuarterStart(year, quarter int) time.Time {
    month := (quarter-1)*3 + 1
    return time.Date(year, time.Month(month), 1, 0, 0, 0, 0, time.UTC)
}

func (c *Calendar) QuarterEnd(year, quarter int) time.Time {
    start := c.QuarterStart(year, quarter)
    return start.AddDate(0, 3, -1)
}

func main() {
    fmt.Println("Calendar Operations Example")
    fmt.Println("===========================")
    
    cal := NewCalendar()
    
    // Add some holidays for 2024
    cal.AddHoliday("New Year's Day", time.Date(2024, time.January, 1, 0, 0, 0, 0, time.UTC))
    cal.AddHoliday("Independence Day", time.Date(2024, time.July, 4, 0, 0, 0, 0, time.UTC))
    cal.AddHoliday("Christmas", time.Date(2024, time.December, 25, 0, 0, 0, 0, time.UTC))
    
    today := time.Now()
    fmt.Printf("Today: %s (%s)\n", today.Format("2006-01-02"), today.Weekday())
    
    // Check if today is a working day
    if cal.IsWorkingDay(today) {
        fmt.Println("Today is a working day")
    } else {
        fmt.Println("Today is not a working day")
        if isHoliday, name := cal.IsHoliday(today); isHoliday {
            fmt.Printf("Today is a holiday: %s\n", name)
        }
    }
    
    // Next working day
    nextWorking := cal.NextWorkingDay(today)
    fmt.Printf("Next working day: %s (%s)\n", nextWorking.Format("2006-01-02"), nextWorking.Weekday())
    
    // Working days in current month
    workingDays := cal.WorkingDaysInMonth(today.Year(), today.Month())
    fmt.Printf("Working days in %s %d: %d days\n", today.Month(), today.Year(), len(workingDays))
    
    // Show first few working days
    fmt.Println("First 5 working days of the month:")
    for i, day := range workingDays {
        if i >= 5 {
            break
        }
        fmt.Printf("  %d. %s (%s)\n", i+1, day.Format("2006-01-02"), day.Weekday())
    }
    
    // Quarter information
    quarter := cal.GetQuarter(today)
    qStart := cal.QuarterStart(today.Year(), quarter)
    qEnd := cal.QuarterEnd(today.Year(), quarter)
    
    fmt.Printf("\nCurrent quarter: Q%d %d\n", quarter, today.Year())
    fmt.Printf("Quarter starts: %s\n", qStart.Format("2006-01-02"))
    fmt.Printf("Quarter ends: %s\n", qEnd.Format("2006-01-02"))
    
    // All quarters for the year
    fmt.Printf("\nAll quarters for %d:\n", today.Year())
    for q := 1; q <= 4; q++ {
        start := cal.QuarterStart(today.Year(), q)
        end := cal.QuarterEnd(today.Year(), q)
        fmt.Printf("  Q%d: %s to %s\n", q, start.Format("2006-01-02"), end.Format("2006-01-02"))
    }
    
    // Check specific dates
    fmt.Println("\nHoliday checks:")
    testDates := []time.Time{
        time.Date(2024, time.January, 1, 0, 0, 0, 0, time.UTC),
        time.Date(2024, time.July, 4, 0, 0, 0, 0, time.UTC),
        time.Date(2024, time.December, 25, 0, 0, 0, 0, time.UTC),
        time.Date(2024, time.March, 15, 0, 0, 0, 0, time.UTC),
    }
    
    for _, date := range testDates {
        if isHoliday, name := cal.IsHoliday(date); isHoliday {
            fmt.Printf("  %s: Holiday (%s)\n", date.Format("2006-01-02"), name)
        } else {
            fmt.Printf("  %s: Regular day\n", date.Format("2006-01-02"))
        }
    }
}
```

Calendar operations are essential for business applications, payroll systems,  
project planning, scheduling software, and any application requiring accurate  
business day calculations and holiday management.  


## Time series data

Handle time series data operations including data points with timestamps,  
aggregation, filtering, and analysis for monitoring and analytics systems.  

```go
package main

import (
    "fmt"
    "math"
    "sort"
    "time"
)

// DataPoint represents a single time series data point
type DataPoint struct {
    Timestamp time.Time
    Value     float64
}

// TimeSeries represents a collection of time-ordered data points
type TimeSeries struct {
    Name   string
    Points []DataPoint
}

func NewTimeSeries(name string) *TimeSeries {
    return &TimeSeries{
        Name:   name,
        Points: make([]DataPoint, 0),
    }
}

func (ts *TimeSeries) AddPoint(timestamp time.Time, value float64) {
    point := DataPoint{Timestamp: timestamp, Value: value}
    ts.Points = append(ts.Points, point)
    
    // Keep points sorted by timestamp
    sort.Slice(ts.Points, func(i, j int) bool {
        return ts.Points[i].Timestamp.Before(ts.Points[j].Timestamp)
    })
}

func (ts *TimeSeries) FilterByTimeRange(start, end time.Time) []DataPoint {
    var filtered []DataPoint
    
    for _, point := range ts.Points {
        if !point.Timestamp.Before(start) && !point.Timestamp.After(end) {
            filtered = append(filtered, point)
        }
    }
    
    return filtered
}

func (ts *TimeSeries) Average(start, end time.Time) float64 {
    points := ts.FilterByTimeRange(start, end)
    
    if len(points) == 0 {
        return 0
    }
    
    sum := 0.0
    for _, point := range points {
        sum += point.Value
    }
    
    return sum / float64(len(points))
}

func (ts *TimeSeries) Max(start, end time.Time) (float64, time.Time) {
    points := ts.FilterByTimeRange(start, end)
    
    if len(points) == 0 {
        return 0, time.Time{}
    }
    
    maxValue := math.Inf(-1)
    var maxTime time.Time
    
    for _, point := range points {
        if point.Value > maxValue {
            maxValue = point.Value
            maxTime = point.Timestamp
        }
    }
    
    return maxValue, maxTime
}

func (ts *TimeSeries) Min(start, end time.Time) (float64, time.Time) {
    points := ts.FilterByTimeRange(start, end)
    
    if len(points) == 0 {
        return 0, time.Time{}
    }
    
    minValue := math.Inf(1)
    var minTime time.Time
    
    for _, point := range points {
        if point.Value < minValue {
            minValue = point.Value
            minTime = point.Timestamp
        }
    }
    
    return minValue, minTime
}

func (ts *TimeSeries) ResampleToInterval(interval time.Duration) []DataPoint {
    if len(ts.Points) == 0 {
        return []DataPoint{}
    }
    
    var resampled []DataPoint
    start := ts.Points[0].Timestamp.Truncate(interval)
    end := ts.Points[len(ts.Points)-1].Timestamp
    
    current := start
    for current.Before(end) || current.Equal(end) {
        intervalEnd := current.Add(interval)
        intervalPoints := ts.FilterByTimeRange(current, intervalEnd)
        
        if len(intervalPoints) > 0 {
            // Calculate average for this interval
            sum := 0.0
            for _, point := range intervalPoints {
                sum += point.Value
            }
            avg := sum / float64(len(intervalPoints))
            
            resampled = append(resampled, DataPoint{
                Timestamp: current,
                Value:     avg,
            })
        }
        
        current = intervalEnd
    }
    
    return resampled
}

func main() {
    fmt.Println("Time Series Data Example")
    fmt.Println("========================")
    
    // Create a time series for temperature data
    tempSeries := NewTimeSeries("Temperature")
    
    // Generate sample data - temperature readings every 10 minutes
    baseTime := time.Now().Add(-2 * time.Hour)
    
    for i := 0; i < 12; i++ {
        timestamp := baseTime.Add(time.Duration(i) * 10 * time.Minute)
        // Simulate temperature fluctuations
        temp := 20.0 + 5.0*math.Sin(float64(i)*0.5) + float64(i%3-1)
        tempSeries.AddPoint(timestamp, temp)
    }
    
    fmt.Printf("Added %d temperature readings\n", len(tempSeries.Points))
    
    // Show first few data points
    fmt.Println("\nFirst 5 data points:")
    for i := 0; i < 5 && i < len(tempSeries.Points); i++ {
        point := tempSeries.Points[i]
        fmt.Printf("  %s: %.1f°C\n", point.Timestamp.Format("15:04"), point.Value)
    }
    
    // Analyze last hour
    oneHourAgo := time.Now().Add(-1 * time.Hour)
    now := time.Now()
    
    fmt.Printf("\nAnalysis for last hour (%s to %s):\n", 
               oneHourAgo.Format("15:04"), now.Format("15:04"))
    
    // Calculate statistics
    avg := tempSeries.Average(oneHourAgo, now)
    maxTemp, maxTime := tempSeries.Max(oneHourAgo, now)
    minTemp, minTime := tempSeries.Min(oneHourAgo, now)
    
    fmt.Printf("  Average temperature: %.1f°C\n", avg)
    fmt.Printf("  Maximum: %.1f°C at %s\n", maxTemp, maxTime.Format("15:04"))
    fmt.Printf("  Minimum: %.1f°C at %s\n", minTemp, minTime.Format("15:04"))
    
    // Resample to 30-minute intervals
    fmt.Println("\nResampled to 30-minute intervals:")
    resampled := tempSeries.ResampleToInterval(30 * time.Minute)
    
    for _, point := range resampled {
        fmt.Printf("  %s: %.1f°C (30-min average)\n", 
                   point.Timestamp.Format("15:04"), point.Value)
    }
    
    // Create another series for humidity
    humiditySeries := NewTimeSeries("Humidity")
    
    for i := 0; i < 8; i++ {
        timestamp := baseTime.Add(time.Duration(i) * 15 * time.Minute)
        humidity := 60.0 + 10.0*math.Cos(float64(i)*0.3)
        humiditySeries.AddPoint(timestamp, humidity)
    }
    
    fmt.Printf("\nHumidity readings (%d points):\n", len(humiditySeries.Points))
    for _, point := range humiditySeries.Points {
        fmt.Printf("  %s: %.1f%%\n", point.Timestamp.Format("15:04"), point.Value)
    }
    
    // Compare temperature and humidity trends
    fmt.Println("\nCorrelation analysis (simplified):")
    tempPoints := tempSeries.FilterByTimeRange(baseTime, now)
    humidityPoints := humiditySeries.FilterByTimeRange(baseTime, now)
    
    if len(tempPoints) > 0 && len(humidityPoints) > 0 {
        tempAvg := tempSeries.Average(baseTime, now)
        humidityAvg := humiditySeries.Average(baseTime, now)
        
        fmt.Printf("  Temperature trend: %.1f°C average\n", tempAvg)
        fmt.Printf("  Humidity trend: %.1f%% average\n", humidityAvg)
    }
}
```

Time series operations are fundamental for monitoring systems, IoT applications,  
financial analysis, performance tracking, and any application requiring  
temporal data analysis, trending, and historical data management.  


## Stopwatch functionality

Implement stopwatch functionality for timing operations, laps, and intervals  
with start, stop, pause, and reset capabilities for performance measurement.  

```go
package main

import (
    "fmt"
    "time"
)

type Stopwatch struct {
    startTime   time.Time
    endTime     time.Time
    pausedTime  time.Duration
    isRunning   bool
    isPaused    bool
    laps        []time.Duration
}

func NewStopwatch() *Stopwatch {
    return &Stopwatch{
        laps: make([]time.Duration, 0),
    }
}

func (sw *Stopwatch) Start() {
    if !sw.isRunning {
        sw.startTime = time.Now()
        sw.isRunning = true
        sw.isPaused = false
        sw.endTime = time.Time{}
        fmt.Println("Stopwatch started")
    }
}

func (sw *Stopwatch) Stop() time.Duration {
    if sw.isRunning {
        sw.endTime = time.Now()
        sw.isRunning = false
        sw.isPaused = false
        elapsed := sw.Elapsed()
        fmt.Printf("Stopwatch stopped - Total time: %v\n", elapsed)
        return elapsed
    }
    return 0
}

func (sw *Stopwatch) Pause() {
    if sw.isRunning && !sw.isPaused {
        sw.pausedTime += time.Since(sw.startTime)
        sw.isPaused = true
        fmt.Println("Stopwatch paused")
    }
}

func (sw *Stopwatch) Resume() {
    if sw.isRunning && sw.isPaused {
        sw.startTime = time.Now()
        sw.isPaused = false
        fmt.Println("Stopwatch resumed")
    }
}

func (sw *Stopwatch) Lap() time.Duration {
    if sw.isRunning && !sw.isPaused {
        lap := sw.Elapsed()
        sw.laps = append(sw.laps, lap)
        fmt.Printf("Lap %d: %v\n", len(sw.laps), lap)
        return lap
    }
    return 0
}

func (sw *Stopwatch) Reset() {
    sw.startTime = time.Time{}
    sw.endTime = time.Time{}
    sw.pausedTime = 0
    sw.isRunning = false
    sw.isPaused = false
    sw.laps = sw.laps[:0]
    fmt.Println("Stopwatch reset")
}

func (sw *Stopwatch) Elapsed() time.Duration {
    if !sw.isRunning {
        if sw.endTime.IsZero() {
            return 0
        }
        return sw.endTime.Sub(sw.startTime) + sw.pausedTime
    }
    
    if sw.isPaused {
        return sw.pausedTime
    }
    
    return time.Since(sw.startTime) + sw.pausedTime
}

func (sw *Stopwatch) GetLaps() []time.Duration {
    return sw.laps
}

func main() {
    fmt.Println("Stopwatch Example")
    fmt.Println("=================")
    
    stopwatch := NewStopwatch()
    
    // Start timing
    stopwatch.Start()
    
    // Simulate some work and take laps
    time.Sleep(500 * time.Millisecond)
    stopwatch.Lap()
    
    time.Sleep(300 * time.Millisecond)
    stopwatch.Lap()
    
    // Pause and resume
    stopwatch.Pause()
    time.Sleep(200 * time.Millisecond) // This time should not count
    stopwatch.Resume()
    
    time.Sleep(400 * time.Millisecond)
    stopwatch.Lap()
    
    // Stop the stopwatch
    totalTime := stopwatch.Stop()
    
    // Show results
    fmt.Printf("\nResults:\n")
    fmt.Printf("Total time: %v\n", totalTime)
    
    laps := stopwatch.GetLaps()
    fmt.Printf("Number of laps: %d\n", len(laps))
    
    for i, lap := range laps {
        fmt.Printf("Lap %d: %v\n", i+1, lap)
    }
    
    // Calculate lap intervals
    if len(laps) > 1 {
        fmt.Println("\nLap intervals:")
        for i := 1; i < len(laps); i++ {
            interval := laps[i] - laps[i-1]
            fmt.Printf("Lap %d to %d: %v\n", i, i+1, interval)
        }
    }
    
    // Reset and demonstrate again
    fmt.Println("\nResetting and timing again...")
    stopwatch.Reset()
    
    stopwatch.Start()
    time.Sleep(100 * time.Millisecond)
    fmt.Printf("Current elapsed: %v\n", stopwatch.Elapsed())
    stopwatch.Stop()
}
```

Stopwatch functionality is essential for performance testing, sports timing,  
user interaction measurement, and any application requiring precise timing  
of operations or events with start, stop, and interval capabilities.  


## Date validation and normalization

Validate and normalize date inputs from various sources, handle different  
formats, and ensure consistent date representation across applications.  

```go
package main

import (
    "fmt"
    "regexp"
    "strconv"
    "strings"
    "time"
)

type DateNormalizer struct {
    layouts []string
}

func NewDateNormalizer() *DateNormalizer {
    return &DateNormalizer{
        layouts: []string{
            "2006-01-02",
            "01/02/2006",
            "02/01/2006",
            "2006/01/02",
            "January 2, 2006",
            "Jan 2, 2006",
            "2 January 2006",
            "2 Jan 2006",
            "2006-01-02T15:04:05Z",
            "2006-01-02 15:04:05",
            "01-02-2006",
            "02-01-2006",
        },
    }
}

func (dn *DateNormalizer) NormalizeDate(input string) (time.Time, error) {
    // Clean input
    cleaned := strings.TrimSpace(input)
    
    // Try standard layouts first
    for _, layout := range dn.layouts {
        if parsed, err := time.Parse(layout, cleaned); err == nil {
            return parsed, nil
        }
    }
    
    // Try to parse common variations
    if normalized, ok := dn.tryCommonVariations(cleaned); ok {
        return normalized, nil
    }
    
    return time.Time{}, fmt.Errorf("unable to parse date: %s", input)
}

func (dn *DateNormalizer) tryCommonVariations(input string) (time.Time, bool) {
    // Handle "today", "yesterday", "tomorrow"
    now := time.Now()
    lower := strings.ToLower(input)
    
    switch lower {
    case "today":
        return time.Date(now.Year(), now.Month(), now.Day(), 0, 0, 0, 0, now.Location()), true
    case "yesterday":
        yesterday := now.AddDate(0, 0, -1)
        return time.Date(yesterday.Year(), yesterday.Month(), yesterday.Day(), 0, 0, 0, 0, yesterday.Location()), true
    case "tomorrow":
        tomorrow := now.AddDate(0, 0, 1)
        return time.Date(tomorrow.Year(), tomorrow.Month(), tomorrow.Day(), 0, 0, 0, 0, tomorrow.Location()), true
    }
    
    // Try to extract numbers and construct date
    if parsed, ok := dn.parseNumbersOnly(input); ok {
        return parsed, true
    }
    
    return time.Time{}, false
}

func (dn *DateNormalizer) parseNumbersOnly(input string) (time.Time, bool) {
    // Extract all numbers from the string
    re := regexp.MustCompile(`\d+`)
    numbers := re.FindAllString(input, -1)
    
    if len(numbers) < 3 {
        return time.Time{}, false
    }
    
    // Convert to integers
    nums := make([]int, len(numbers))
    for i, num := range numbers {
        if n, err := strconv.Atoi(num); err != nil {
            return time.Time{}, false
        } else {
            nums[i] = n
        }
    }
    
    // Try different interpretations
    year, month, day := nums[0], nums[1], nums[2]
    
    // If first number is small, might be day or month
    if nums[0] <= 31 && nums[2] > 31 {
        // First number is day/month, third is year
        year = nums[2]
        if nums[0] <= 12 && nums[1] <= 31 {
            // DD/MM/YYYY format
            day, month = nums[1], nums[0]
        } else {
            // MM/DD/YYYY format
            month, day = nums[0], nums[1]
        }
    }
    
    // Adjust year if it's 2-digit
    if year < 100 {
        if year < 50 {
            year += 2000
        } else {
            year += 1900
        }
    }
    
    // Validate ranges
    if year < 1900 || year > 2100 || month < 1 || month > 12 || day < 1 || day > 31 {
        return time.Time{}, false
    }
    
    date := time.Date(year, time.Month(month), day, 0, 0, 0, 0, time.UTC)
    
    // Check if the date is valid (handles leap years, month days)
    if date.Year() == year && int(date.Month()) == month && date.Day() == day {
        return date, true
    }
    
    return time.Time{}, false
}

func (dn *DateNormalizer) ValidateAndNormalize(inputs []string) []struct {
    Input  string
    Output time.Time
    Error  error
} {
    results := make([]struct {
        Input  string
        Output time.Time
        Error  error
    }, len(inputs))
    
    for i, input := range inputs {
        output, err := dn.NormalizeDate(input)
        results[i] = struct {
            Input  string
            Output time.Time
            Error  error
        }{Input: input, Output: output, Error: err}
    }
    
    return results
}

func main() {
    fmt.Println("Date Validation and Normalization")
    fmt.Println("=================================")
    
    normalizer := NewDateNormalizer()
    
    // Test various date formats
    testInputs := []string{
        "2024-03-15",
        "03/15/2024",
        "15/03/2024",
        "March 15, 2024",
        "Mar 15, 2024",
        "15 March 2024",
        "15 Mar 2024",
        "2024/03/15",
        "03-15-2024",
        "15-03-2024",
        "today",
        "yesterday",
        "tomorrow",
        "15 3 2024",
        "3/15/24",
        "2024.03.15",
        "invalid date",
        "2024-02-30", // Invalid date
        "13/01/2024", // Ambiguous format
    }
    
    fmt.Println("Normalization Results:")
    fmt.Println(strings.Repeat("-", 60))
    
    results := normalizer.ValidateAndNormalize(testInputs)
    
    for _, result := range results {
        if result.Error != nil {
            fmt.Printf("%-20s -> %-20s (ERROR: %v)\n", 
                       result.Input, "---", result.Error)
        } else {
            fmt.Printf("%-20s -> %-20s ✓\n", 
                       result.Input, result.Output.Format("2006-01-02"))
        }
    }
    
    // Demonstrate batch processing
    fmt.Println("\nBatch Processing Example:")
    batchInputs := []string{
        "01/15/2024",
        "2024-01-15",
        "January 15, 2024",
        "today",
        "15/01/2024",
    }
    
    batchResults := normalizer.ValidateAndNormalize(batchInputs)
    validDates := make([]time.Time, 0)
    
    for _, result := range batchResults {
        if result.Error == nil {
            validDates = append(validDates, result.Output)
        }
    }
    
    fmt.Printf("Successfully normalized %d out of %d dates\n", 
               len(validDates), len(batchInputs))
    
    // Find date range
    if len(validDates) > 0 {
        earliest, latest := validDates[0], validDates[0]
        for _, date := range validDates {
            if date.Before(earliest) {
                earliest = date
            }
            if date.After(latest) {
                latest = date
            }
        }
        
        fmt.Printf("Date range: %s to %s\n", 
                   earliest.Format("2006-01-02"), latest.Format("2006-01-02"))
    }
}
```

Date validation and normalization ensure data quality, prevent application  
errors, improve user experience, and maintain consistency across systems  
handling date inputs from multiple sources and formats.  


## Time zone database

Interact with time zone databases, handle timezone information, and manage  
timezone-aware operations for global applications and scheduling systems.  

```go
package main

import (
    "fmt"
    "sort"
    "strings"
    "time"
)

type TimeZoneInfo struct {
    Name        string
    Location    *time.Location
    Offset      int    // Offset in seconds
    Abbreviation string
    IsDST       bool
}

type TimeZoneManager struct {
    zones []TimeZoneInfo
}

func NewTimeZoneManager() *TimeZoneManager {
    return &TimeZoneManager{
        zones: make([]TimeZoneInfo, 0),
    }
}

func (tzm *TimeZoneManager) LoadCommonTimeZones() error {
    commonZones := []string{
        "UTC",
        "America/New_York",
        "America/Chicago",
        "America/Denver",
        "America/Los_Angeles",
        "America/Toronto",
        "America/Sao_Paulo",
        "Europe/London",
        "Europe/Paris",
        "Europe/Berlin",
        "Europe/Moscow",
        "Asia/Tokyo",
        "Asia/Shanghai",
        "Asia/Kolkata",
        "Asia/Dubai",
        "Australia/Sydney",
        "Australia/Melbourne",
        "Pacific/Auckland",
    }
    
    baseTime := time.Now()
    
    for _, zoneName := range commonZones {
        if loc, err := time.LoadLocation(zoneName); err == nil {
            timeInZone := baseTime.In(loc)
            name, offset := timeInZone.Zone()
            
            tzInfo := TimeZoneInfo{
                Name:         zoneName,
                Location:     loc,
                Offset:       offset,
                Abbreviation: name,
                IsDST:        isDST(timeInZone, loc),
            }
            
            tzm.zones = append(tzm.zones, tzInfo)
        }
    }
    
    return nil
}

func isDST(t time.Time, location *time.Location) bool {
    timeInLocation := t.In(location)
    _, offset := timeInLocation.Zone()
    
    // Compare with standard time offset (January)
    jan := time.Date(t.Year(), time.January, 1, 12, 0, 0, 0, location)
    _, janOffset := jan.Zone()
    
    return offset != janOffset
}

func (tzm *TimeZoneManager) GetTimeZone(name string) (*TimeZoneInfo, bool) {
    for _, zone := range tzm.zones {
        if zone.Name == name {
            return &zone, true
        }
    }
    return nil, false
}

func (tzm *TimeZoneManager) GetTimeZonesByOffset(offsetHours int) []TimeZoneInfo {
    offsetSeconds := offsetHours * 3600
    var matches []TimeZoneInfo
    
    for _, zone := range tzm.zones {
        if zone.Offset == offsetSeconds {
            matches = append(matches, zone)
        }
    }
    
    return matches
}

func (tzm *TimeZoneManager) FindTimeZonesByRegion(region string) []TimeZoneInfo {
    var matches []TimeZoneInfo
    lowerRegion := strings.ToLower(region)
    
    for _, zone := range tzm.zones {
        if strings.Contains(strings.ToLower(zone.Name), lowerRegion) {
            matches = append(matches, zone)
        }
    }
    
    return matches
}

func (tzm *TimeZoneManager) ConvertTime(t time.Time, fromZone, toZone string) (time.Time, error) {
    fromTZ, ok := tzm.GetTimeZone(fromZone)
    if !ok {
        return time.Time{}, fmt.Errorf("unknown timezone: %s", fromZone)
    }
    
    toTZ, ok := tzm.GetTimeZone(toZone)
    if !ok {
        return time.Time{}, fmt.Errorf("unknown timezone: %s", toZone)
    }
    
    // Convert time to the source timezone, then to destination
    timeInFrom := t.In(fromTZ.Location)
    timeInTo := timeInFrom.In(toTZ.Location)
    
    return timeInTo, nil
}

func (tzm *TimeZoneManager) GetWorldClock(baseTime time.Time) map[string]string {
    worldClock := make(map[string]string)
    
    for _, zone := range tzm.zones {
        timeInZone := baseTime.In(zone.Location)
        worldClock[zone.Name] = timeInZone.Format("15:04 MST")
    }
    
    return worldClock
}

func (tzm *TimeZoneManager) ListTimeZones() []TimeZoneInfo {
    // Return a copy sorted by offset
    sorted := make([]TimeZoneInfo, len(tzm.zones))
    copy(sorted, tzm.zones)
    
    sort.Slice(sorted, func(i, j int) bool {
        return sorted[i].Offset < sorted[j].Offset
    })
    
    return sorted
}

func main() {
    fmt.Println("Time Zone Database Example")
    fmt.Println("==========================")
    
    tzManager := NewTimeZoneManager()
    err := tzManager.LoadCommonTimeZones()
    if err != nil {
        fmt.Printf("Error loading timezones: %v\n", err)
        return
    }
    
    fmt.Printf("Loaded %d time zones\n\n", len(tzManager.zones))
    
    // List all time zones
    fmt.Println("Available Time Zones (sorted by offset):")
    fmt.Println(strings.Repeat("-", 70))
    
    zones := tzManager.ListTimeZones()
    for _, zone := range zones {
        offsetHours := zone.Offset / 3600
        dstIndicator := ""
        if zone.IsDST {
            dstIndicator = " (DST)"
        }
        
        fmt.Printf("%-25s %+3d:00 %-5s%s\n", 
                   zone.Name, offsetHours, zone.Abbreviation, dstIndicator)
    }
    
    // World clock
    fmt.Println("\nWorld Clock (current time):")
    fmt.Println(strings.Repeat("-", 40))
    
    now := time.Now()
    worldClock := tzManager.GetWorldClock(now)
    
    // Show selected major cities
    majorCities := []string{
        "UTC",
        "America/New_York",
        "America/Los_Angeles", 
        "Europe/London",
        "Asia/Tokyo",
        "Australia/Sydney",
    }
    
    for _, city := range majorCities {
        if timeStr, exists := worldClock[city]; exists {
            cityName := strings.Split(city, "/")[len(strings.Split(city, "/"))-1]
            if city == "UTC" {
                cityName = "UTC"
            }
            fmt.Printf("%-12s: %s\n", cityName, timeStr)
        }
    }
    
    // Find zones by region
    fmt.Println("\nAmerican Time Zones:")
    americanZones := tzManager.FindTimeZonesByRegion("america")
    for _, zone := range americanZones[:5] { // Show first 5
        fmt.Printf("  %s (UTC%+d)\n", zone.Name, zone.Offset/3600)
    }
    
    // Convert specific time
    fmt.Println("\nTime Conversion Example:")
    meetingTime := time.Date(2024, time.March, 15, 14, 30, 0, 0, time.UTC)
    fmt.Printf("Meeting at %s UTC\n", meetingTime.Format("15:04"))
    
    conversions := []string{
        "America/New_York",
        "Europe/London", 
        "Asia/Tokyo",
    }
    
    for _, targetZone := range conversions {
        if converted, err := tzManager.ConvertTime(meetingTime, "UTC", targetZone); err == nil {
            cityName := strings.Split(targetZone, "/")[1]
            fmt.Printf("  %s: %s\n", cityName, converted.Format("15:04 MST"))
        }
    }
    
    // Find zones by offset
    fmt.Println("\nTime zones at UTC+8:")
    utc8Zones := tzManager.GetTimeZonesByOffset(8)
    for _, zone := range utc8Zones {
        fmt.Printf("  %s\n", zone.Name)
    }
}
```

Time zone database interaction is essential for international applications,  
meeting scheduling across regions, travel planning, and any system requiring  
accurate time representation and conversion across different geographic zones.  


## Working with fiscal years

Handle fiscal year calculations, different fiscal year starts, and fiscal  
period operations commonly used in business and financial applications.  

```go
package main

import (
    "fmt"
    "time"
)

type FiscalYear struct {
    StartMonth time.Month
    Year       int
}

type FiscalCalendar struct {
    fiscalYearStart time.Month
}

func NewFiscalCalendar(fiscalYearStart time.Month) *FiscalCalendar {
    return &FiscalCalendar{
        fiscalYearStart: fiscalYearStart,
    }
}

func (fc *FiscalCalendar) GetFiscalYear(date time.Time) FiscalYear {
    year := date.Year()
    
    // If we're before the fiscal year start month, we're in the previous fiscal year
    if date.Month() < fc.fiscalYearStart {
        year--
    }
    
    return FiscalYear{
        StartMonth: fc.fiscalYearStart,
        Year:       year,
    }
}

func (fc *FiscalCalendar) FiscalYearStart(fiscalYear int) time.Time {
    return time.Date(fiscalYear, fc.fiscalYearStart, 1, 0, 0, 0, 0, time.UTC)
}

func (fc *FiscalCalendar) FiscalYearEnd(fiscalYear int) time.Time {
    start := fc.FiscalYearStart(fiscalYear)
    return start.AddDate(1, 0, -1)
}

func (fc *FiscalCalendar) GetFiscalQuarter(date time.Time) (int, FiscalYear) {
    fy := fc.GetFiscalYear(date)
    fyStart := fc.FiscalYearStart(fy.Year)
    
    monthsFromStart := int(date.Month()) - int(fyStart.Month())
    if monthsFromStart < 0 {
        monthsFromStart += 12
    }
    
    quarter := (monthsFromStart / 3) + 1
    return quarter, fy
}

func (fc *FiscalCalendar) FiscalQuarterStart(fiscalYear, quarter int) time.Time {
    fyStart := fc.FiscalYearStart(fiscalYear)
    return fyStart.AddDate(0, (quarter-1)*3, 0)
}

func (fc *FiscalCalendar) FiscalQuarterEnd(fiscalYear, quarter int) time.Time {
    start := fc.FiscalQuarterStart(fiscalYear, quarter)
    return start.AddDate(0, 3, -1)
}

func (fc *FiscalCalendar) DaysInFiscalYear(fiscalYear int) int {
    start := fc.FiscalYearStart(fiscalYear)
    end := fc.FiscalYearEnd(fiscalYear)
    return int(end.Sub(start).Hours()/24) + 1
}

func (fc *FiscalCalendar) CurrentFiscalYear() FiscalYear {
    return fc.GetFiscalYear(time.Now())
}

func (fy FiscalYear) String() string {
    endYear := fy.Year + 1
    return fmt.Sprintf("FY %d-%d", fy.Year, endYear%100)
}

func main() {
    fmt.Println("Fiscal Year Operations")
    fmt.Println("======================")
    
    // US Government fiscal year (starts October 1)
    fmt.Println("US Government Fiscal Calendar (October 1 start):")
    usFiscal := NewFiscalCalendar(time.October)
    
    today := time.Now()
    currentFY := usFiscal.GetFiscalYear(today)
    
    fmt.Printf("Today: %s\n", today.Format("2006-01-02"))
    fmt.Printf("Current fiscal year: %s\n", currentFY.String())
    
    fyStart := usFiscal.FiscalYearStart(currentFY.Year)
    fyEnd := usFiscal.FiscalYearEnd(currentFY.Year)
    
    fmt.Printf("FY starts: %s\n", fyStart.Format("2006-01-02"))
    fmt.Printf("FY ends: %s\n", fyEnd.Format("2006-01-02"))
    fmt.Printf("Days in FY: %d\n", usFiscal.DaysInFiscalYear(currentFY.Year))
    
    // Fiscal quarters
    quarter, fy := usFiscal.GetFiscalQuarter(today)
    fmt.Printf("\nCurrent fiscal quarter: Q%d %s\n", quarter, fy.String())
    
    fmt.Println("All quarters for current fiscal year:")
    for q := 1; q <= 4; q++ {
        qStart := usFiscal.FiscalQuarterStart(currentFY.Year, q)
        qEnd := usFiscal.FiscalQuarterEnd(currentFY.Year, q)
        fmt.Printf("  Q%d: %s to %s\n", q, 
                   qStart.Format("2006-01-02"), qEnd.Format("2006-01-02"))
    }
    
    // UK fiscal year (starts April 6)
    fmt.Println("\nUK Fiscal Calendar (April 6 start):")
    ukFiscal := NewFiscalCalendar(time.April)
    
    ukFY := ukFiscal.GetFiscalYear(today)
    fmt.Printf("UK fiscal year: %s\n", ukFY.String())
    
    ukStart := ukFiscal.FiscalYearStart(ukFY.Year)
    ukEnd := ukFiscal.FiscalYearEnd(ukFY.Year)
    
    fmt.Printf("UK FY starts: %s\n", ukStart.Format("2006-01-02"))
    fmt.Printf("UK FY ends: %s\n", ukEnd.Format("2006-01-02"))
    
    // Compare different fiscal year systems
    fmt.Println("\nFiscal Year Comparison for specific dates:")
    testDates := []time.Time{
        time.Date(2024, time.January, 15, 0, 0, 0, 0, time.UTC),
        time.Date(2024, time.April, 15, 0, 0, 0, 0, time.UTC),
        time.Date(2024, time.October, 15, 0, 0, 0, 0, time.UTC),
    }
    
    for _, date := range testDates {
        usFY := usFiscal.GetFiscalYear(date)
        ukFY := ukFiscal.GetFiscalYear(date)
        
        fmt.Printf("Date %s:\n", date.Format("2006-01-02"))
        fmt.Printf("  US FY: %s\n", usFY.String())
        fmt.Printf("  UK FY: %s\n", ukFY.String())
    }
    
    // Calculate days remaining in fiscal year
    fmt.Println("\nDays remaining in current fiscal year:")
    usEnd := usFiscal.FiscalYearEnd(currentFY.Year)
    daysRemaining := int(usEnd.Sub(today).Hours() / 24)
    fmt.Printf("US FY: %d days remaining\n", daysRemaining)
    
    ukEnd := ukFiscal.FiscalYearEnd(ukFY.Year)
    ukDaysRemaining := int(ukEnd.Sub(today).Hours() / 24)
    fmt.Printf("UK FY: %d days remaining\n", ukDaysRemaining)
}
```

Fiscal year operations are crucial for financial reporting, budget planning,  
compliance requirements, and business applications that need to align with  
organizational or governmental fiscal periods rather than calendar years.  


## Cron expression parsing

Parse and evaluate cron expressions for job scheduling, understand cron  
syntax, and implement cron-like scheduling functionality in applications.  

```go
package main

import (
    "fmt"
    "strconv"
    "strings"
    "time"
)

type CronField struct {
    Min, Max int
    Values   []int
    Step     int
    Any      bool
}

type CronExpression struct {
    Minute     CronField
    Hour       CronField
    DayOfMonth CronField
    Month      CronField
    DayOfWeek  CronField
}

func ParseCron(expr string) (*CronExpression, error) {
    fields := strings.Fields(expr)
    if len(fields) != 5 {
        return nil, fmt.Errorf("invalid cron expression: expected 5 fields, got %d", len(fields))
    }
    
    cron := &CronExpression{}
    var err error
    
    // Parse minute (0-59)
    if cron.Minute, err = parseCronField(fields[0], 0, 59); err != nil {
        return nil, fmt.Errorf("invalid minute field: %v", err)
    }
    
    // Parse hour (0-23)  
    if cron.Hour, err = parseCronField(fields[1], 0, 23); err != nil {
        return nil, fmt.Errorf("invalid hour field: %v", err)
    }
    
    // Parse day of month (1-31)
    if cron.DayOfMonth, err = parseCronField(fields[2], 1, 31); err != nil {
        return nil, fmt.Errorf("invalid day field: %v", err)
    }
    
    // Parse month (1-12)
    if cron.Month, err = parseCronField(fields[3], 1, 12); err != nil {
        return nil, fmt.Errorf("invalid month field: %v", err)
    }
    
    // Parse day of week (0-7, where 0 and 7 are Sunday)
    if cron.DayOfWeek, err = parseCronField(fields[4], 0, 7); err != nil {
        return nil, fmt.Errorf("invalid day of week field: %v", err)
    }
    
    return cron, nil
}

func parseCronField(field string, min, max int) (CronField, error) {
    cf := CronField{Min: min, Max: max, Step: 1}
    
    if field == "*" {
        cf.Any = true
        return cf, nil
    }
    
    // Handle step values (e.g., */5, 1-10/2)
    if strings.Contains(field, "/") {
        parts := strings.Split(field, "/")
        if len(parts) != 2 {
            return cf, fmt.Errorf("invalid step syntax")
        }
        
        step, err := strconv.Atoi(parts[1])
        if err != nil || step <= 0 {
            return cf, fmt.Errorf("invalid step value")
        }
        cf.Step = step
        field = parts[0]
    }
    
    if field == "*" {
        cf.Any = true
        return cf, nil
    }
    
    // Handle ranges (e.g., 1-5)
    if strings.Contains(field, "-") {
        parts := strings.Split(field, "-")
        if len(parts) != 2 {
            return cf, fmt.Errorf("invalid range syntax")
        }
        
        start, err := strconv.Atoi(parts[0])
        if err != nil || start < min || start > max {
            return cf, fmt.Errorf("invalid range start")
        }
        
        end, err := strconv.Atoi(parts[1])
        if err != nil || end < min || end > max || end < start {
            return cf, fmt.Errorf("invalid range end")
        }
        
        for i := start; i <= end; i += cf.Step {
            cf.Values = append(cf.Values, i)
        }
        return cf, nil
    }
    
    // Handle comma-separated values (e.g., 1,3,5)
    if strings.Contains(field, ",") {
        parts := strings.Split(field, ",")
        for _, part := range parts {
            val, err := strconv.Atoi(strings.TrimSpace(part))
            if err != nil || val < min || val > max {
                return cf, fmt.Errorf("invalid value: %s", part)
            }
            cf.Values = append(cf.Values, val)
        }
        return cf, nil
    }
    
    // Single value
    val, err := strconv.Atoi(field)
    if err != nil || val < min || val > max {
        return cf, fmt.Errorf("invalid value: %s", field)
    }
    cf.Values = append(cf.Values, val)
    
    return cf, nil
}

func (cf CronField) Matches(value int) bool {
    if cf.Any {
        return (value-cf.Min)%cf.Step == 0
    }
    
    for _, v := range cf.Values {
        if v == value {
            return true
        }
    }
    return false
}

func (ce *CronExpression) Matches(t time.Time) bool {
    minute := t.Minute()
    hour := t.Hour()
    day := t.Day()
    month := int(t.Month())
    weekday := int(t.Weekday())
    
    // Convert Sunday from 0 to 7 for cron compatibility
    if weekday == 0 {
        weekday = 7
    }
    
    return ce.Minute.Matches(minute) &&
           ce.Hour.Matches(hour) &&
           ce.DayOfMonth.Matches(day) &&
           ce.Month.Matches(month) &&
           (ce.DayOfWeek.Any || ce.DayOfWeek.Matches(weekday))
}

func (ce *CronExpression) NextRun(from time.Time) time.Time {
    // Start from the next minute
    next := from.Truncate(time.Minute).Add(time.Minute)
    
    // Look for next matching time (limit search to avoid infinite loops)
    for i := 0; i < 366*24*60; i++ { // Max one year
        if ce.Matches(next) {
            return next
        }
        next = next.Add(time.Minute)
    }
    
    return time.Time{} // No match found
}

func (ce *CronExpression) String() string {
    return fmt.Sprintf("Cron: minute=%v hour=%v day=%v month=%v weekday=%v", 
                       ce.Minute.Values, ce.Hour.Values, ce.DayOfMonth.Values, 
                       ce.Month.Values, ce.DayOfWeek.Values)
}

func main() {
    fmt.Println("Cron Expression Parsing")
    fmt.Println("=======================")
    
    // Test various cron expressions
    expressions := []string{
        "0 9 * * *",        // Every day at 9:00 AM
        "*/15 * * * *",     // Every 15 minutes
        "0 */2 * * *",      // Every 2 hours
        "0 9 * * 1-5",      // Every weekday at 9:00 AM
        "30 8 1 * *",       // First day of every month at 8:30 AM
        "0 0 1 1 *",        // New Year's Day at midnight
        "0 12 * * 0",       // Every Sunday at noon
        "*/5 9-17 * * 1-5", // Every 5 minutes, 9 AM to 5 PM, weekdays
    }
    
    descriptions := []string{
        "Every day at 9:00 AM",
        "Every 15 minutes",
        "Every 2 hours",
        "Every weekday at 9:00 AM",
        "First day of every month at 8:30 AM",
        "New Year's Day at midnight",
        "Every Sunday at noon",
        "Every 5 minutes, 9 AM to 5 PM, weekdays",
    }
    
    now := time.Now()
    fmt.Printf("Current time: %s\n\n", now.Format("2006-01-02 15:04:05 Mon"))
    
    for i, expr := range expressions {
        fmt.Printf("Expression: %s\n", expr)
        fmt.Printf("Description: %s\n", descriptions[i])
        
        cron, err := ParseCron(expr)
        if err != nil {
            fmt.Printf("Error: %v\n\n", err)
            continue
        }
        
        // Check if current time matches
        matches := cron.Matches(now)
        fmt.Printf("Matches current time: %t\n", matches)
        
        // Find next run
        nextRun := cron.NextRun(now)
        if !nextRun.IsZero() {
            duration := nextRun.Sub(now)
            fmt.Printf("Next run: %s (%v from now)\n", 
                       nextRun.Format("2006-01-02 15:04:05 Mon"), duration.Round(time.Minute))
        } else {
            fmt.Println("Next run: Not found within one year")
        }
        
        fmt.Println(strings.Repeat("-", 50))
    }
    
    // Demonstrate scheduling simulation
    fmt.Println("Scheduling Simulation (next 24 hours):")
    
    // Use a simple cron: every 4 hours
    simpleCron, _ := ParseCron("0 */4 * * *")
    
    checkTime := now.Truncate(time.Hour)
    endTime := checkTime.Add(24 * time.Hour)
    
    fmt.Printf("Checking from %s to %s\n", 
               checkTime.Format("2006-01-02 15:04"), endTime.Format("2006-01-02 15:04"))
    
    matches := 0
    for checkTime.Before(endTime) {
        if simpleCron.Matches(checkTime) {
            matches++
            fmt.Printf("  %d. %s\n", matches, checkTime.Format("2006-01-02 15:04:05 Mon"))
        }
        checkTime = checkTime.Add(time.Hour)
    }
    
    fmt.Printf("\nTotal matches in 24 hours: %d\n", matches)
}
```

Cron expression parsing enables flexible job scheduling, automated task  
execution, recurring event management, and time-based triggers essential  
for system administration, data processing, and automated workflows.  


## Time-sensitive operations

Handle operations that depend on specific time conditions, implement  
time-based access control, and manage time-sensitive business logic.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

type TimeWindow struct {
    Start time.Time
    End   time.Time
    Days  []time.Weekday
}

type AccessControl struct {
    windows []TimeWindow
}

func NewAccessControl() *AccessControl {
    return &AccessControl{
        windows: make([]TimeWindow, 0),
    }
}

func (ac *AccessControl) AddTimeWindow(start, end time.Time, days []time.Weekday) {
    window := TimeWindow{
        Start: start,
        End:   end,
        Days:  days,
    }
    ac.windows = append(ac.windows, window)
}

func (ac *AccessControl) IsAccessAllowed(t time.Time) bool {
    for _, window := range ac.windows {
        if ac.timeInWindow(t, window) {
            return true
        }
    }
    return false
}

func (ac *AccessControl) timeInWindow(t time.Time, window TimeWindow) bool {
    // Check day of week
    dayMatch := false
    if len(window.Days) == 0 {
        dayMatch = true // No day restriction
    } else {
        for _, day := range window.Days {
            if t.Weekday() == day {
                dayMatch = true
                break
            }
        }
    }
    
    if !dayMatch {
        return false
    }
    
    // Create times for comparison (same date as t)
    startTime := time.Date(t.Year(), t.Month(), t.Day(), 
                          window.Start.Hour(), window.Start.Minute(), window.Start.Second(), 0, t.Location())
    endTime := time.Date(t.Year(), t.Month(), t.Day(), 
                        window.End.Hour(), window.End.Minute(), window.End.Second(), 0, t.Location())
    
    // Handle overnight windows (end time is next day)
    if endTime.Before(startTime) {
        // Window spans midnight
        if t.After(startTime) || t.Before(endTime) {
            return true
        }
    } else {
        // Normal window within same day
        if t.After(startTime) && t.Before(endTime) {
            return true
        }
    }
    
    return false
}

func (ac *AccessControl) NextAllowedTime(from time.Time) time.Time {
    // Check every minute for next 7 days
    check := from
    for i := 0; i < 7*24*60; i++ {
        if ac.IsAccessAllowed(check) {
            return check
        }
        check = check.Add(time.Minute)
    }
    
    return time.Time{} // No allowed time found in next week
}

// Business hours checker
type BusinessHours struct {
    weekdayStart time.Time
    weekdayEnd   time.Time
    weekendStart time.Time
    weekendEnd   time.Time
    holidays     []time.Time
}

func NewBusinessHours(weekdayStart, weekdayEnd, weekendStart, weekendEnd time.Time) *BusinessHours {
    return &BusinessHours{
        weekdayStart: weekdayStart,
        weekdayEnd:   weekdayEnd,
        weekendStart: weekendStart,
        weekendEnd:   weekendEnd,
        holidays:     make([]time.Time, 0),
    }
}

func (bh *BusinessHours) AddHoliday(date time.Time) {
    // Add date without time component
    holiday := time.Date(date.Year(), date.Month(), date.Day(), 0, 0, 0, 0, date.Location())
    bh.holidays = append(bh.holidays, holiday)
}

func (bh *BusinessHours) IsBusinessHours(t time.Time) bool {
    // Check if it's a holiday
    dateOnly := time.Date(t.Year(), t.Month(), t.Day(), 0, 0, 0, 0, t.Location())
    for _, holiday := range bh.holidays {
        if holiday.Equal(dateOnly) {
            return false
        }
    }
    
    weekday := t.Weekday()
    
    var start, end time.Time
    if weekday == time.Saturday || weekday == time.Sunday {
        start = time.Date(t.Year(), t.Month(), t.Day(), 
                         bh.weekendStart.Hour(), bh.weekendStart.Minute(), 0, 0, t.Location())
        end = time.Date(t.Year(), t.Month(), t.Day(), 
                       bh.weekendEnd.Hour(), bh.weekendEnd.Minute(), 0, 0, t.Location())
    } else {
        start = time.Date(t.Year(), t.Month(), t.Day(), 
                         bh.weekdayStart.Hour(), bh.weekdayStart.Minute(), 0, 0, t.Location())
        end = time.Date(t.Year(), t.Month(), t.Day(), 
                       bh.weekdayEnd.Hour(), bh.weekdayEnd.Minute(), 0, 0, t.Location())
    }
    
    return t.After(start) && t.Before(end)
}

// Time-sensitive discount system
type TimeDiscount struct {
    StartTime    time.Time
    EndTime      time.Time
    DiscountRate float64
    Description  string
}

type DiscountManager struct {
    discounts []TimeDiscount
}

func NewDiscountManager() *DiscountManager {
    return &DiscountManager{
        discounts: make([]TimeDiscount, 0),
    }
}

func (dm *DiscountManager) AddDiscount(start, end time.Time, rate float64, desc string) {
    discount := TimeDiscount{
        StartTime:    start,
        EndTime:      end,
        DiscountRate: rate,
        Description:  desc,
    }
    dm.discounts = append(dm.discounts, discount)
}

func (dm *DiscountManager) GetBestDiscount(t time.Time) (float64, string) {
    bestRate := 0.0
    bestDesc := "No discount available"
    
    for _, discount := range dm.discounts {
        if t.After(discount.StartTime) && t.Before(discount.EndTime) {
            if discount.DiscountRate > bestRate {
                bestRate = discount.DiscountRate
                bestDesc = discount.Description
            }
        }
    }
    
    return bestRate, bestDesc
}

func main() {
    fmt.Println("Time-Sensitive Operations")
    fmt.Println("=========================")
    
    now := time.Now()
    
    // Access Control Example
    fmt.Println("1. Access Control System")
    fmt.Println(strings.Repeat("-", 30))
    
    accessControl := NewAccessControl()
    
    // Add business hours: 9 AM to 5 PM, Monday to Friday
    businessStart := time.Date(0, 1, 1, 9, 0, 0, 0, time.UTC)
    businessEnd := time.Date(0, 1, 1, 17, 0, 0, 0, time.UTC)
    weekdays := []time.Weekday{time.Monday, time.Tuesday, time.Wednesday, time.Thursday, time.Friday}
    
    accessControl.AddTimeWindow(businessStart, businessEnd, weekdays)
    
    // Add weekend maintenance window: 2 AM to 4 AM, Saturday and Sunday
    maintenanceStart := time.Date(0, 1, 1, 2, 0, 0, 0, time.UTC)
    maintenanceEnd := time.Date(0, 1, 1, 4, 0, 0, 0, time.UTC)
    weekends := []time.Weekday{time.Saturday, time.Sunday}
    
    accessControl.AddTimeWindow(maintenanceStart, maintenanceEnd, weekends)
    
    fmt.Printf("Current time: %s (%s)\n", now.Format("15:04:05"), now.Weekday())
    fmt.Printf("Access allowed: %t\n", accessControl.IsAccessAllowed(now))
    
    nextAllowed := accessControl.NextAllowedTime(now)
    if !nextAllowed.IsZero() {
        fmt.Printf("Next allowed time: %s (%s)\n", 
                   nextAllowed.Format("2006-01-02 15:04:05"), nextAllowed.Weekday())
    }
    
    // Business Hours Example
    fmt.Println("\n2. Business Hours System")
    fmt.Println(strings.Repeat("-", 30))
    
    // Create business hours: weekdays 9-17, weekends 10-16
    weekdayStart := time.Date(0, 1, 1, 9, 0, 0, 0, time.UTC)
    weekdayEnd := time.Date(0, 1, 1, 17, 0, 0, 0, time.UTC)
    weekendStart := time.Date(0, 1, 1, 10, 0, 0, 0, time.UTC)
    weekendEnd := time.Date(0, 1, 1, 16, 0, 0, 0, time.UTC)
    
    businessHours := NewBusinessHours(weekdayStart, weekdayEnd, weekendStart, weekendEnd)
    
    // Add some holidays
    businessHours.AddHoliday(time.Date(now.Year(), time.December, 25, 0, 0, 0, 0, now.Location())) // Christmas
    businessHours.AddHoliday(time.Date(now.Year(), time.January, 1, 0, 0, 0, 0, now.Location()))   // New Year
    
    fmt.Printf("Current time is business hours: %t\n", businessHours.IsBusinessHours(now))
    
    // Test different times
    testTimes := []time.Time{
        time.Date(now.Year(), now.Month(), now.Day(), 10, 30, 0, 0, now.Location()), // 10:30 AM today
        time.Date(now.Year(), now.Month(), now.Day(), 18, 0, 0, 0, now.Location()),  // 6:00 PM today
        time.Date(now.Year(), time.December, 25, 12, 0, 0, 0, now.Location()),       // Christmas noon
    }
    
    for _, testTime := range testTimes {
        inHours := businessHours.IsBusinessHours(testTime)
        fmt.Printf("  %s (%s): %t\n", testTime.Format("2006-01-02 15:04"), testTime.Weekday(), inHours)
    }
    
    // Time-sensitive discounts
    fmt.Println("\n3. Time-sensitive Discounts")
    fmt.Println(strings.Repeat("-", 35))
    
    discountManager := NewDiscountManager()
    
    // Add various time-based discounts
    discountManager.AddDiscount(
        time.Date(now.Year(), now.Month(), now.Day(), 6, 0, 0, 0, now.Location()),
        time.Date(now.Year(), now.Month(), now.Day(), 9, 0, 0, 0, now.Location()),
        0.15, "Early bird discount (6 AM - 9 AM)")
    
    discountManager.AddDiscount(
        time.Date(now.Year(), now.Month(), now.Day(), 22, 0, 0, 0, now.Location()),
        time.Date(now.Year(), now.Month(), now.Day()+1, 2, 0, 0, 0, now.Location()),
        0.20, "Night owl discount (10 PM - 2 AM)")
    
    discountManager.AddDiscount(
        time.Date(now.Year(), now.Month(), now.Day(), 12, 0, 0, 0, now.Location()),
        time.Date(now.Year(), now.Month(), now.Day(), 14, 0, 0, 0, now.Location()),
        0.10, "Lunch hour discount (12 PM - 2 PM)")
    
    rate, desc := discountManager.GetBestDiscount(now)
    fmt.Printf("Current time: %s\n", now.Format("15:04:05"))
    fmt.Printf("Best discount: %.0f%% - %s\n", rate*100, desc)
    
    // Test different hours
    fmt.Println("\nDiscount schedule for today:")
    for hour := 0; hour < 24; hour += 3 {
        testTime := time.Date(now.Year(), now.Month(), now.Day(), hour, 0, 0, 0, now.Location())
        rate, desc := discountManager.GetBestDiscount(testTime)
        if rate > 0 {
            fmt.Printf("  %02d:00 - %.0f%% discount: %s\n", hour, rate*100, desc)
        }
    }
}
```

Time-sensitive operations enable dynamic business logic, automated access  
control, time-based pricing, and conditional functionality that adapts  
based on temporal context and business requirements.  


## Time parsing with error handling

Implement robust time parsing with comprehensive error handling, fallback  
parsing strategies, and detailed error reporting for user-friendly applications.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

type ParseResult struct {
    Time     time.Time
    Layout   string
    Success  bool
    Error    error
    Warnings []string
}

type TimeParser struct {
    layouts   []string
    fallbacks []string
    warnings  []string
}

func NewTimeParser() *TimeParser {
    return &TimeParser{
        layouts: []string{
            time.RFC3339,
            time.RFC3339Nano,
            time.RFC1123,
            time.RFC822,
            "2006-01-02 15:04:05",
            "2006-01-02T15:04:05",
            "2006-01-02 15:04:05 MST",
            "2006-01-02",
            "01/02/2006 15:04:05",
            "01/02/2006 3:04:05 PM",
            "01/02/2006",
            "02-Jan-2006",
            "January 2, 2006",
            "Jan 2, 2006 15:04:05",
            "Jan 2, 2006",
        },
        fallbacks: []string{
            "2006-1-2 15:4:5",     // Single digit months/days
            "2006/1/2 15:4:5",
            "1/2/2006 15:4:5",
            "2006.1.2 15:4:5",
        },
    }
}

func (tp *TimeParser) ParseWithFallback(input string) ParseResult {
    tp.warnings = make([]string, 0)
    
    // Clean input
    cleaned := strings.TrimSpace(input)
    if cleaned == "" {
        return ParseResult{
            Success: false,
            Error:   fmt.Errorf("empty input"),
        }
    }
    
    // Try primary layouts
    for _, layout := range tp.layouts {
        if parsed, err := time.Parse(layout, cleaned); err == nil {
            return ParseResult{
                Time:    parsed,
                Layout:  layout,
                Success: true,
            }
        }
    }
    
    // Try fallback layouts
    for _, layout := range tp.fallbacks {
        if parsed, err := time.Parse(layout, cleaned); err == nil {
            tp.warnings = append(tp.warnings, "Used fallback parsing - consider using standard format")
            return ParseResult{
                Time:     parsed,
                Layout:   layout,
                Success:  true,
                Warnings: tp.warnings,
            }
        }
    }
    
    // Try common transformations
    if result := tp.tryTransformations(cleaned); result.Success {
        return result
    }
    
    // Final attempt with flexible parsing
    if result := tp.flexibleParse(cleaned); result.Success {
        return result
    }
    
    return ParseResult{
        Success: false,
        Error:   fmt.Errorf("unable to parse time: %s", input),
    }
}

func (tp *TimeParser) tryTransformations(input string) ParseResult {
    transformations := []struct {
        from, to string
        desc     string
    }{
        {"/", "-", "replaced slashes with dashes"},
        {".", "-", "replaced dots with dashes"},
        {"  ", " ", "removed extra spaces"},
        {" at ", " ", "removed 'at' keyword"},
        {" on ", " ", "removed 'on' keyword"},
    }
    
    for _, transform := range transformations {
        if strings.Contains(input, transform.from) {
            transformed := strings.ReplaceAll(input, transform.from, transform.to)
            
            for _, layout := range tp.layouts {
                if parsed, err := time.Parse(layout, transformed); err == nil {
                    tp.warnings = append(tp.warnings, fmt.Sprintf("Applied transformation: %s", transform.desc))
                    return ParseResult{
                        Time:     parsed,
                        Layout:   layout,
                        Success:  true,
                        Warnings: tp.warnings,
                    }
                }
            }
        }
    }
    
    return ParseResult{Success: false}
}

func (tp *TimeParser) flexibleParse(input string) ParseResult {
    // Try to parse common relative times
    lower := strings.ToLower(input)
    now := time.Now()
    
    switch {
    case strings.Contains(lower, "now"):
        return ParseResult{Time: now, Layout: "relative", Success: true}
    case strings.Contains(lower, "today"):
        today := time.Date(now.Year(), now.Month(), now.Day(), 0, 0, 0, 0, now.Location())
        return ParseResult{Time: today, Layout: "relative", Success: true}
    case strings.Contains(lower, "yesterday"):
        yesterday := now.AddDate(0, 0, -1)
        yesterday = time.Date(yesterday.Year(), yesterday.Month(), yesterday.Day(), 0, 0, 0, 0, yesterday.Location())
        return ParseResult{Time: yesterday, Layout: "relative", Success: true}
    case strings.Contains(lower, "tomorrow"):
        tomorrow := now.AddDate(0, 0, 1)
        tomorrow = time.Date(tomorrow.Year(), tomorrow.Month(), tomorrow.Day(), 0, 0, 0, 0, tomorrow.Location())
        return ParseResult{Time: tomorrow, Layout: "relative", Success: true}
    }
    
    return ParseResult{Success: false}
}

func (tp *TimeParser) ValidateTime(t time.Time) []string {
    var issues []string
    now := time.Now()
    
    // Check for unreasonable dates
    if t.Year() < 1900 {
        issues = append(issues, "date is before 1900")
    }
    if t.Year() > now.Year()+100 {
        issues = append(issues, "date is more than 100 years in the future")
    }
    
    // Check for timezone issues
    if t.Location() == time.UTC && now.Location() != time.UTC {
        issues = append(issues, "parsed as UTC but local timezone expected")
    }
    
    return issues
}

func (tp *TimeParser) ParseBatch(inputs []string) []ParseResult {
    results := make([]ParseResult, len(inputs))
    
    for i, input := range inputs {
        result := tp.ParseWithFallback(input)
        
        if result.Success {
            // Add validation warnings
            validationIssues := tp.ValidateTime(result.Time)
            result.Warnings = append(result.Warnings, validationIssues...)
        }
        
        results[i] = result
    }
    
    return results
}

func main() {
    fmt.Println("Robust Time Parsing with Error Handling")
    fmt.Println("=======================================")
    
    parser := NewTimeParser()
    
    // Test various time formats and edge cases
    testInputs := []string{
        "2024-03-15 10:30:45",
        "March 15, 2024",
        "03/15/2024 10:30 AM",
        "15/03/2024",
        "2024.03.15",
        "today",
        "yesterday at 3pm",
        "tomorrow",
        "now",
        "",
        "invalid date",
        "2024-02-30",
        "32/01/2024",
        "2024-13-01",
        "15 Mar 2024",
        "2024/3/5 9:5:30",
        "Jan 1, 1850",
        "Dec 31, 2150",
    }
    
    fmt.Println("Parsing Results:")
    fmt.Println(strings.Repeat("-", 80))
    
    results := parser.ParseBatch(testInputs)
    
    for i, result := range results {
        input := testInputs[i]
        fmt.Printf("Input: %-20s ", fmt.Sprintf("'%s'", input))
        
        if result.Success {
            fmt.Printf("✓ %s", result.Time.Format("2006-01-02 15:04:05"))
            if result.Layout != "" {
                fmt.Printf(" (layout: %s)", result.Layout)
            }
            
            if len(result.Warnings) > 0 {
                fmt.Printf("\n%25s Warnings:", "")
                for _, warning := range result.Warnings {
                    fmt.Printf("\n%27s - %s", "", warning)
                }
            }
        } else {
            fmt.Printf("✗ Error: %v", result.Error)
        }
        fmt.Println()
    }
    
    // Statistics
    successful := 0
    withWarnings := 0
    
    for _, result := range results {
        if result.Success {
            successful++
            if len(result.Warnings) > 0 {
                withWarnings++
            }
        }
    }
    
    fmt.Printf("\nParsing Statistics:\n")
    fmt.Printf("Total inputs: %d\n", len(testInputs))
    fmt.Printf("Successfully parsed: %d (%.1f%%)\n", successful, float64(successful)/float64(len(testInputs))*100)
    fmt.Printf("With warnings: %d\n", withWarnings)
    fmt.Printf("Failed: %d\n", len(testInputs)-successful)
    
    // Demonstrate error recovery
    fmt.Println("\nError Recovery Examples:")
    recoveryExamples := []string{
        "2024/3/15",        // Single digit month
        "2024.03.15 10.30", // Dots instead of colons
        "March 15th 2024",  // Ordinal numbers
    }
    
    for _, example := range recoveryExamples {
        result := parser.ParseWithFallback(example)
        if result.Success {
            fmt.Printf("Recovered '%s' -> %s\n", example, result.Time.Format("2006-01-02 15:04:05"))
        } else {
            fmt.Printf("Could not recover '%s'\n", example)
        }
    }
}
```

Robust time parsing with error handling improves user experience, prevents  
application crashes, provides helpful feedback, and ensures reliable time  
data processing in applications dealing with diverse input sources.  


## Date arithmetic edge cases

Handle edge cases in date arithmetic including leap years, month boundaries,  
daylight saving time transitions, and timezone-aware calculations.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

type DateArithmetic struct {
    location *time.Location
}

func NewDateArithmetic(location *time.Location) *DateArithmetic {
    return &DateArithmetic{location: location}
}

func (da *DateArithmetic) AddMonthsSafe(t time.Time, months int) time.Time {
    // Handle month addition with day overflow
    result := t.AddDate(0, months, 0)
    
    // Check if day changed due to shorter month
    if result.Day() != t.Day() {
        // Go automatically adjusts to valid date, but we might want the last day of month
        // Return to last day of target month
        firstOfTargetMonth := time.Date(result.Year(), result.Month(), 1, 
                                       result.Hour(), result.Minute(), result.Second(), result.Nanosecond(), result.Location())
        lastOfTargetMonth := firstOfTargetMonth.AddDate(0, 1, -1)
        
        if t.Day() > lastOfTargetMonth.Day() {
            // Original day doesn't exist in target month, use last day
            return time.Date(result.Year(), result.Month(), lastOfTargetMonth.Day(),
                           result.Hour(), result.Minute(), result.Second(), result.Nanosecond(), result.Location())
        }
    }
    
    return result
}

func (da *DateArithmetic) AddBusinessDays(t time.Time, days int) time.Time {
    current := t
    remaining := days
    
    if days > 0 {
        for remaining > 0 {
            current = current.AddDate(0, 0, 1)
            if current.Weekday() != time.Saturday && current.Weekday() != time.Sunday {
                remaining--
            }
        }
    } else {
        for remaining < 0 {
            current = current.AddDate(0, 0, -1)
            if current.Weekday() != time.Saturday && current.Weekday() != time.Sunday {
                remaining++
            }
        }
    }
    
    return current
}

func (da *DateArithmetic) DaysInMonth(year int, month time.Month) int {
    // First day of next month minus one day
    firstOfNextMonth := time.Date(year, month+1, 1, 0, 0, 0, 0, time.UTC)
    lastOfMonth := firstOfNextMonth.AddDate(0, 0, -1)
    return lastOfMonth.Day()
}

func (da *DateArithmetic) IsLeapYear(year int) bool {
    return year%4 == 0 && (year%100 != 0 || year%400 == 0)
}

func (da *DateArithmetic) EndOfMonth(t time.Time) time.Time {
    firstOfNextMonth := time.Date(t.Year(), t.Month()+1, 1, 0, 0, 0, 0, t.Location())
    return firstOfNextMonth.AddDate(0, 0, -1)
}

func (da *DateArithmetic) BeginningOfMonth(t time.Time) time.Time {
    return time.Date(t.Year(), t.Month(), 1, 0, 0, 0, 0, t.Location())
}

func (da *DateArithmetic) AddWithDSTHandling(t time.Time, d time.Duration) time.Time {
    // Check if we're crossing DST boundary
    result := t.Add(d)
    
    // If the time zone offset changed, we crossed DST
    _, t1Offset := t.Zone()
    _, t2Offset := result.Zone()
    
    if t1Offset != t2Offset {
        fmt.Printf("DST transition detected: offset changed from %d to %d seconds\n", t1Offset, t2Offset)
    }
    
    return result
}

func (da *DateArithmetic) MonthsBetween(start, end time.Time) int {
    months := 0
    
    if end.Before(start) {
        start, end = end, start
        defer func() { months = -months }()
    }
    
    for start.Year() < end.Year() || (start.Year() == end.Year() && start.Month() < end.Month()) {
        start = start.AddDate(0, 1, 0)
        months++
    }
    
    return months
}

func (da *DateArithmetic) YearsBetween(start, end time.Time) float64 {
    if end.Before(start) {
        start, end = end, start
    }
    
    years := end.Year() - start.Year()
    
    // Check if birthday has passed this year
    birthdayThisYear := time.Date(end.Year(), start.Month(), start.Day(), 
                                 start.Hour(), start.Minute(), start.Second(), start.Nanosecond(), start.Location())
    
    if end.Before(birthdayThisYear) {
        years--
    }
    
    // Calculate fractional part
    if years > 0 {
        startOfCurrentYear := time.Date(end.Year(), start.Month(), start.Day(), 
                                       start.Hour(), start.Minute(), start.Second(), start.Nanosecond(), start.Location())
        if end.Before(startOfCurrentYear) {
            startOfCurrentYear = startOfCurrentYear.AddDate(-1, 0, 0)
        }
        
        fraction := end.Sub(startOfCurrentYear).Hours() / (365.25 * 24)
        return float64(years) + fraction
    }
    
    return end.Sub(start).Hours() / (365.25 * 24)
}

func main() {
    fmt.Println("Date Arithmetic Edge Cases")
    fmt.Println("==========================")
    
    // Initialize with local timezone
    da := NewDateArithmetic(time.Local)
    
    // Test month addition edge cases
    fmt.Println("1. Month Addition Edge Cases")
    fmt.Println(strings.Repeat("-", 35))
    
    // January 31 + 1 month
    jan31 := time.Date(2024, time.January, 31, 10, 0, 0, 0, time.UTC)
    feb31 := da.AddMonthsSafe(jan31, 1)
    fmt.Printf("Jan 31 + 1 month: %s\n", feb31.Format("2006-01-02"))
    
    // February 29 (leap year) + 1 year
    feb29 := time.Date(2024, time.February, 29, 10, 0, 0, 0, time.UTC)
    nextYear := da.AddMonthsSafe(feb29, 12)
    fmt.Printf("Feb 29, 2024 + 1 year: %s\n", nextYear.Format("2006-01-02"))
    
    // Test leap year logic
    fmt.Println("\n2. Leap Year Calculations")
    fmt.Println(strings.Repeat("-", 30))
    
    testYears := []int{2000, 1900, 2004, 2100, 2024}
    for _, year := range testYears {
        isLeap := da.IsLeapYear(year)
        daysInFeb := da.DaysInMonth(year, time.February)
        fmt.Printf("Year %d: leap=%t, Feb days=%d\n", year, isLeap, daysInFeb)
    }
    
    // Test business days
    fmt.Println("\n3. Business Days Calculation")
    fmt.Println(strings.Repeat("-", 35))
    
    friday := time.Date(2024, time.March, 15, 9, 0, 0, 0, time.UTC) // Assume this is a Friday
    fmt.Printf("Starting date: %s (%s)\n", friday.Format("2006-01-02"), friday.Weekday())
    
    // Add 5 business days
    fiveBusiness := da.AddBusinessDays(friday, 5)
    fmt.Printf("+ 5 business days: %s (%s)\n", fiveBusiness.Format("2006-01-02"), fiveBusiness.Weekday())
    
    // Subtract 3 business days
    minusThree := da.AddBusinessDays(friday, -3)
    fmt.Printf("- 3 business days: %s (%s)\n", minusThree.Format("2006-01-02"), minusThree.Weekday())
    
    // Month boundaries
    fmt.Println("\n4. Month Boundary Operations")
    fmt.Println(strings.Repeat("-", 35))
    
    someDate := time.Date(2024, time.March, 15, 14, 30, 45, 0, time.UTC)
    beginMonth := da.BeginningOfMonth(someDate)
    endMonth := da.EndOfMonth(someDate)
    
    fmt.Printf("Date: %s\n", someDate.Format("2006-01-02 15:04:05"))
    fmt.Printf("Month start: %s\n", beginMonth.Format("2006-01-02 15:04:05"))
    fmt.Printf("Month end: %s\n", endMonth.Format("2006-01-02 15:04:05"))
    
    // Calculate months and years between dates
    fmt.Println("\n5. Time Span Calculations")
    fmt.Println(strings.Repeat("-", 30))
    
    birth := time.Date(1990, time.March, 15, 0, 0, 0, 0, time.UTC)
    now := time.Now()
    
    monthsBetween := da.MonthsBetween(birth, now)
    yearsBetween := da.YearsBetween(birth, now)
    
    fmt.Printf("From: %s\n", birth.Format("2006-01-02"))
    fmt.Printf("To: %s\n", now.Format("2006-01-02"))
    fmt.Printf("Months between: %d\n", monthsBetween)
    fmt.Printf("Years between: %.2f\n", yearsBetween)
    
    // DST handling example (if in a timezone with DST)
    fmt.Println("\n6. DST Handling")
    fmt.Println(strings.Repeat("-", 20))
    
    // Try to create a time that might cross DST
    est, _ := time.LoadLocation("America/New_York")
    springForward := time.Date(2024, time.March, 10, 1, 30, 0, 0, est)
    
    fmt.Printf("Before: %s\n", springForward.Format("2006-01-02 15:04:05 MST"))
    afterDST := da.AddWithDSTHandling(springForward, 2*time.Hour)
    fmt.Printf("After +2h: %s\n", afterDST.Format("2006-01-02 15:04:05 MST"))
    
    // Edge case: February calculations
    fmt.Println("\n7. February Edge Cases")
    fmt.Println(strings.Repeat("-", 25))
    
    dates := []time.Time{
        time.Date(2024, time.February, 28, 0, 0, 0, 0, time.UTC), // Leap year
        time.Date(2023, time.February, 28, 0, 0, 0, 0, time.UTC), // Regular year
        time.Date(2024, time.February, 29, 0, 0, 0, 0, time.UTC), // Leap day
    }
    
    for _, date := range dates {
        nextDay := date.AddDate(0, 0, 1)
        nextMonth := da.AddMonthsSafe(date, 1)
        fmt.Printf("%s + 1 day = %s, + 1 month = %s\n", 
                   date.Format("2006-01-02"), nextDay.Format("2006-01-02"), nextMonth.Format("2006-01-02"))
    }
}
```

Handling date arithmetic edge cases prevents bugs, ensures accurate  
calculations across different calendar scenarios, and provides reliable  
temporal operations for financial, scheduling, and business applications.  


## Time performance optimization

Optimize time-related operations for high-performance applications,  
including efficient time comparisons, bulk operations, and memory usage.  

```go
package main

import (
    "fmt"
    "sort"
    "sync"
    "time"
)

// Optimized time cache for repeated operations
type TimeCache struct {
    cache map[string]time.Time
    mutex sync.RWMutex
}

func NewTimeCache() *TimeCache {
    return &TimeCache{
        cache: make(map[string]time.Time),
    }
}

func (tc *TimeCache) GetOrParse(layout, value string) (time.Time, error) {
    key := layout + ":" + value
    
    // Try to get from cache first (read lock)
    tc.mutex.RLock()
    if cached, exists := tc.cache[key]; exists {
        tc.mutex.RUnlock()
        return cached, nil
    }
    tc.mutex.RUnlock()
    
    // Parse and cache (write lock)
    tc.mutex.Lock()
    defer tc.mutex.Unlock()
    
    // Double-check pattern
    if cached, exists := tc.cache[key]; exists {
        return cached, nil
    }
    
    parsed, err := time.Parse(layout, value)
    if err != nil {
        return time.Time{}, err
    }
    
    tc.cache[key] = parsed
    return parsed, nil
}

func (tc *TimeCache) Size() int {
    tc.mutex.RLock()
    defer tc.mutex.RUnlock()
    return len(tc.cache)
}

func (tc *TimeCache) Clear() {
    tc.mutex.Lock()
    defer tc.mutex.Unlock()
    tc.cache = make(map[string]time.Time)
}

// Bulk time operations
type BulkTimeProcessor struct {
    workers int
}

func NewBulkTimeProcessor(workers int) *BulkTimeProcessor {
    return &BulkTimeProcessor{workers: workers}
}

func (btp *BulkTimeProcessor) ProcessTimestamps(timestamps []int64, operation func(time.Time) time.Time) []time.Time {
    if len(timestamps) == 0 {
        return nil
    }
    
    results := make([]time.Time, len(timestamps))
    
    if len(timestamps) < 100 || btp.workers <= 1 {
        // Process sequentially for small datasets
        for i, ts := range timestamps {
            t := time.Unix(ts, 0)
            results[i] = operation(t)
        }
        return results
    }
    
    // Process in parallel
    chunkSize := len(timestamps) / btp.workers
    if chunkSize == 0 {
        chunkSize = 1
    }
    
    var wg sync.WaitGroup
    
    for i := 0; i < len(timestamps); i += chunkSize {
        end := i + chunkSize
        if end > len(timestamps) {
            end = len(timestamps)
        }
        
        wg.Add(1)
        go func(start, end int) {
            defer wg.Done()
            for j := start; j < end; j++ {
                t := time.Unix(timestamps[j], 0)
                results[j] = operation(t)
            }
        }(i, end)
    }
    
    wg.Wait()
    return results
}

// Efficient time sorting
type TimeSlice []time.Time

func (ts TimeSlice) Len() int           { return len(ts) }
func (ts TimeSlice) Less(i, j int) bool { return ts[i].Before(ts[j]) }
func (ts TimeSlice) Swap(i, j int)      { ts[i], ts[j] = ts[j], ts[i] }

// Pre-allocated time range generator
type TimeRangeGenerator struct {
    buffer []time.Time
}

func NewTimeRangeGenerator(capacity int) *TimeRangeGenerator {
    return &TimeRangeGenerator{
        buffer: make([]time.Time, 0, capacity),
    }
}

func (trg *TimeRangeGenerator) GenerateRange(start time.Time, interval time.Duration, count int) []time.Time {
    // Reuse buffer if possible
    if cap(trg.buffer) >= count {
        trg.buffer = trg.buffer[:0] // Reset length but keep capacity
    } else {
        trg.buffer = make([]time.Time, 0, count)
    }
    
    current := start
    for i := 0; i < count; i++ {
        trg.buffer = append(trg.buffer, current)
        current = current.Add(interval)
    }
    
    return trg.buffer
}

// Memory-efficient time operations
func CountTimesByMonth(times []time.Time) map[string]int {
    counts := make(map[string]int)
    
    for _, t := range times {
        key := fmt.Sprintf("%04d-%02d", t.Year(), int(t.Month()))
        counts[key]++
    }
    
    return counts
}

// Optimized time comparison functions
func FindTimeRange(times []time.Time) (min, max time.Time, count int) {
    if len(times) == 0 {
        return time.Time{}, time.Time{}, 0
    }
    
    min, max = times[0], times[0]
    count = len(times)
    
    for i := 1; i < len(times); i++ {
        if times[i].Before(min) {
            min = times[i]
        }
        if times[i].After(max) {
            max = times[i]
        }
    }
    
    return min, max, count
}

// Benchmark helper
func BenchmarkTimeOperations() {
    fmt.Println("Time Performance Benchmarks")
    fmt.Println("===========================")
    
    // Generate test data
    timestamps := make([]int64, 10000)
    baseTime := time.Now().Unix()
    
    for i := range timestamps {
        timestamps[i] = baseTime + int64(i*3600) // One hour intervals
    }
    
    // Test 1: Bulk conversion
    fmt.Println("1. Bulk Timestamp Conversion")
    processor := NewBulkTimeProcessor(4)
    
    start := time.Now()
    times := processor.ProcessTimestamps(timestamps, func(t time.Time) time.Time {
        return t.UTC()
    })
    duration := time.Since(start)
    
    fmt.Printf("   Converted %d timestamps in %v\n", len(times), duration)
    fmt.Printf("   Rate: %.0f conversions/ms\n", float64(len(times))/duration.Seconds()/1000)
    
    // Test 2: Sorting performance
    fmt.Println("\n2. Time Sorting Performance")
    testTimes := make([]time.Time, len(times))
    copy(testTimes, times)
    
    // Shuffle for realistic test
    for i := len(testTimes) - 1; i > 0; i-- {
        j := i % (i + 1) // Simple pseudo-random
        testTimes[i], testTimes[j] = testTimes[j], testTimes[i]
    }
    
    start = time.Now()
    sort.Sort(TimeSlice(testTimes))
    duration = time.Since(start)
    
    fmt.Printf("   Sorted %d times in %v\n", len(testTimes), duration)
    
    // Test 3: Range finding
    fmt.Println("\n3. Time Range Finding")
    start = time.Now()
    min, max, count := FindTimeRange(testTimes)
    duration = time.Since(start)
    
    fmt.Printf("   Found range of %d times in %v\n", count, duration)
    fmt.Printf("   Range: %s to %s\n", min.Format("2006-01-02"), max.Format("2006-01-02"))
    
    // Test 4: Cached parsing
    fmt.Println("\n4. Cached Time Parsing")
    cache := NewTimeCache()
    
    testStrings := []string{
        "2024-01-15T10:30:00Z",
        "2024-02-15T10:30:00Z",
        "2024-03-15T10:30:00Z",
        "2024-01-15T10:30:00Z", // Repeat for cache hit
        "2024-02-15T10:30:00Z", // Repeat for cache hit
    }
    
    start = time.Now()
    for i := 0; i < 1000; i++ {
        for _, str := range testStrings {
            _, _ = cache.GetOrParse(time.RFC3339, str)
        }
    }
    duration = time.Since(start)
    
    fmt.Printf("   Parsed %d strings (with caching) in %v\n", 1000*len(testStrings), duration)
    fmt.Printf("   Cache size: %d entries\n", cache.Size())
    
    // Test 5: Memory-efficient operations
    fmt.Println("\n5. Memory-efficient Month Counting")
    start = time.Now()
    monthCounts := CountTimesByMonth(times)
    duration = time.Since(start)
    
    fmt.Printf("   Counted %d times by month in %v\n", len(times), duration)
    fmt.Printf("   Found %d unique months\n", len(monthCounts))
    
    // Show some month counts
    count := 0
    for month, cnt := range monthCounts {
        if count >= 5 {
            break
        }
        fmt.Printf("     %s: %d times\n", month, cnt)
        count++
    }
    
    // Test 6: Pre-allocated range generation
    fmt.Println("\n6. Pre-allocated Range Generation")
    generator := NewTimeRangeGenerator(1000)
    
    start = time.Now()
    for i := 0; i < 100; i++ {
        _ = generator.GenerateRange(time.Now(), time.Hour, 100)
    }
    duration = time.Since(start)
    
    fmt.Printf("   Generated 100 ranges of 100 times each in %v\n", duration)
    fmt.Printf("   Average per generation: %v\n", duration/100)
    
    // Memory usage tips
    fmt.Println("\n7. Memory Optimization Tips")
    fmt.Println("   - Use time.Time value types instead of pointers")
    fmt.Println("   - Pre-allocate slices with known capacity")
    fmt.Println("   - Cache frequently parsed time strings")
    fmt.Println("   - Use bulk operations for large datasets")
    fmt.Println("   - Consider time.Unix() for timestamp storage")
    fmt.Println("   - Use sync.Pool for frequently allocated time slices")
}

func main() {
    BenchmarkTimeOperations()
}
```

Time performance optimization is crucial for high-throughput applications,  
real-time systems, data processing pipelines, and any system handling  
large volumes of time-based operations requiring maximum efficiency.  


## Working with durations

Comprehensive duration operations including parsing, formatting, arithmetic,  
and conversion between different time units for precise timing operations.  

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("Duration Operations")
    fmt.Println("===================")
    
    // Create durations using constants
    fmt.Println("1. Duration Constants:")
    durations := []time.Duration{
        time.Nanosecond,
        time.Microsecond,
        time.Millisecond,
        time.Second,
        time.Minute,
        time.Hour,
    }
    
    for _, d := range durations {
        fmt.Printf("   %-12s: %v (%d nanoseconds)\n", 
                   d.String(), d, d.Nanoseconds())
    }
    
    // Complex duration arithmetic
    fmt.Println("\n2. Duration Arithmetic:")
    d1 := 2*time.Hour + 30*time.Minute + 15*time.Second
    d2 := 45*time.Minute + 30*time.Second
    
    fmt.Printf("   Duration 1: %v\n", d1)
    fmt.Printf("   Duration 2: %v\n", d2)
    fmt.Printf("   Sum: %v\n", d1+d2)
    fmt.Printf("   Difference: %v\n", d1-d2)
    fmt.Printf("   Multiply by 3: %v\n", d1*3)
    fmt.Printf("   Divide by 2: %v\n", d1/2)
    
    // Duration conversions
    fmt.Println("\n3. Duration Conversions:")
    duration := 1*time.Hour + 23*time.Minute + 45*time.Second + 678*time.Millisecond
    
    fmt.Printf("   Duration: %v\n", duration)
    fmt.Printf("   Nanoseconds: %d\n", duration.Nanoseconds())
    fmt.Printf("   Microseconds: %.0f\n", duration.Seconds()*1000000)
    fmt.Printf("   Milliseconds: %.0f\n", duration.Seconds()*1000)
    fmt.Printf("   Seconds: %.3f\n", duration.Seconds())
    fmt.Printf("   Minutes: %.2f\n", duration.Minutes())
    fmt.Printf("   Hours: %.4f\n", duration.Hours())
    
    // Duration parsing from strings
    fmt.Println("\n4. Duration Parsing:")
    durationStrings := []string{
        "1h30m45s",
        "2.5h",
        "90m",
        "5400s",
        "1h2m3s4ms5µs6ns",
        "300ms",
        "-1h30m",
    }
    
    for _, str := range durationStrings {
        if parsed, err := time.ParseDuration(str); err != nil {
            fmt.Printf("   %-20s: ERROR - %v\n", str, err)
        } else {
            fmt.Printf("   %-20s: %v (%.2f hours)\n", str, parsed, parsed.Hours())
        }
    }
    
    // Duration in different contexts
    fmt.Println("\n5. Duration Applications:")
    
    // Timeout example
    timeout := 30 * time.Second
    fmt.Printf("   Timeout: %v\n", timeout)
    
    // Interval example  
    interval := 5 * time.Minute
    fmt.Printf("   Check interval: %v\n", interval)
    
    // Age calculation with duration
    birth := time.Date(1990, time.January, 15, 0, 0, 0, 0, time.UTC)
    age := time.Since(birth)
    fmt.Printf("   Age: %v (%.1f years)\n", age, age.Hours()/(24*365.25))
    
    // Measuring code execution time
    start := time.Now()
    time.Sleep(100 * time.Millisecond) // Simulate work
    elapsed := time.Since(start)
    fmt.Printf("   Operation time: %v\n", elapsed)
    
    // Duration formatting
    fmt.Println("\n6. Custom Duration Formatting:")
    longDuration := 25*time.Hour + 61*time.Minute + 75*time.Second
    
    // Convert to normal units
    totalSeconds := int(longDuration.Seconds())
    hours := totalSeconds / 3600
    minutes := (totalSeconds % 3600) / 60
    seconds := totalSeconds % 60
    
    fmt.Printf("   Raw duration: %v\n", longDuration)
    fmt.Printf("   Formatted: %dh %dm %ds\n", hours, minutes, seconds)
    
    // Duration ranges
    fmt.Println("\n7. Duration Ranges and Validation:")
    minDuration := 1 * time.Second
    maxDuration := 1 * time.Hour
    testDuration := 45 * time.Minute
    
    fmt.Printf("   Test duration: %v\n", testDuration)
    fmt.Printf("   Within range [%v, %v]: %t\n", 
               minDuration, maxDuration, 
               testDuration >= minDuration && testDuration <= maxDuration)
    
    // Duration rounding
    fmt.Println("\n8. Duration Rounding:")
    precise := 1*time.Hour + 23*time.Minute + 45*time.Second + 678*time.Millisecond
    
    fmt.Printf("   Precise: %v\n", precise)
    fmt.Printf("   Rounded to second: %v\n", precise.Truncate(time.Second))
    fmt.Printf("   Rounded to minute: %v\n", precise.Truncate(time.Minute))
    fmt.Printf("   Rounded to hour: %v\n", precise.Truncate(time.Hour))
}
```

Duration operations provide precise timing control, enable accurate  
performance measurement, support timeout implementations, and are  
essential for time-based calculations in system programming.  


## Time localization and formatting

Handle time localization, custom formatting, and cultural time  
representations for international applications and user interfaces.  

```go
package main

import (
    "fmt"
    "strings"
    "time"
)

type LocalizedTimeFormatter struct {
    locale string
}

func NewLocalizedTimeFormatter(locale string) *LocalizedTimeFormatter {
    return &LocalizedTimeFormatter{locale: locale}
}

func (ltf *LocalizedTimeFormatter) FormatDate(t time.Time, style string) string {
    switch ltf.locale {
    case "en-US":
        return ltf.formatUSDate(t, style)
    case "en-GB":
        return ltf.formatUKDate(t, style)
    case "de-DE":
        return ltf.formatGermanDate(t, style)
    case "fr-FR":
        return ltf.formatFrenchDate(t, style)
    case "ja-JP":
        return ltf.formatJapaneseDate(t, style)
    default:
        return t.Format("2006-01-02") // Default ISO format
    }
}

func (ltf *LocalizedTimeFormatter) formatUSDate(t time.Time, style string) string {
    switch style {
    case "short":
        return t.Format("1/2/2006")
    case "medium":
        return t.Format("Jan 2, 2006")
    case "long":
        return t.Format("January 2, 2006")
    case "full":
        return t.Format("Monday, January 2, 2006")
    default:
        return t.Format("01/02/2006")
    }
}

func (ltf *LocalizedTimeFormatter) formatUKDate(t time.Time, style string) string {
    switch style {
    case "short":
        return t.Format("2/1/2006")
    case "medium":
        return t.Format("2 Jan 2006")
    case "long":
        return t.Format("2 January 2006")
    case "full":
        return t.Format("Monday, 2 January 2006")
    default:
        return t.Format("02/01/2006")
    }
}

func (ltf *LocalizedTimeFormatter) formatGermanDate(t time.Time, style string) string {
    germanMonths := []string{"", "Januar", "Februar", "März", "April", "Mai", "Juni",
                             "Juli", "August", "September", "Oktober", "November", "Dezember"}
    germanDays := []string{"Sonntag", "Montag", "Dienstag", "Mittwoch", "Donnerstag", "Freitag", "Samstag"}
    
    switch style {
    case "short":
        return t.Format("2.1.2006")
    case "medium":
        return fmt.Sprintf("%d. %s %d", t.Day(), germanMonths[t.Month()][:3], t.Year())
    case "long":
        return fmt.Sprintf("%d. %s %d", t.Day(), germanMonths[t.Month()], t.Year())
    case "full":
        return fmt.Sprintf("%s, %d. %s %d", germanDays[t.Weekday()], t.Day(), germanMonths[t.Month()], t.Year())
    default:
        return t.Format("02.01.2006")
    }
}

func (ltf *LocalizedTimeFormatter) formatFrenchDate(t time.Time, style string) string {
    frenchMonths := []string{"", "janvier", "février", "mars", "avril", "mai", "juin",
                             "juillet", "août", "septembre", "octobre", "novembre", "décembre"}
    frenchDays := []string{"dimanche", "lundi", "mardi", "mercredi", "jeudi", "vendredi", "samedi"}
    
    switch style {
    case "short":
        return t.Format("2/1/2006")
    case "medium":
        return fmt.Sprintf("%d %s %d", t.Day(), frenchMonths[t.Month()][:3], t.Year())
    case "long":
        return fmt.Sprintf("%d %s %d", t.Day(), frenchMonths[t.Month()], t.Year())
    case "full":
        return fmt.Sprintf("%s %d %s %d", frenchDays[t.Weekday()], t.Day(), frenchMonths[t.Month()], t.Year())
    default:
        return t.Format("02/01/2006")
    }
}

func (ltf *LocalizedTimeFormatter) formatJapaneseDate(t time.Time, style string) string {
    // Japanese era years (simplified - using Reiwa era starting 2019)
    var eraYear int
    var eraName string
    
    if t.Year() >= 2019 {
        eraYear = t.Year() - 2018
        eraName = "令和"
    } else {
        eraYear = t.Year() - 1988
        eraName = "平成"
    }
    
    switch style {
    case "short":
        return fmt.Sprintf("%d/%d/%d", t.Year(), int(t.Month()), t.Day())
    case "medium":
        return fmt.Sprintf("%d年%d月%d日", t.Year(), int(t.Month()), t.Day())
    case "long":
        return fmt.Sprintf("%s%d年%d月%d日", eraName, eraYear, int(t.Month()), t.Day())
    case "full":
        weekdays := []string{"日曜日", "月曜日", "火曜日", "水曜日", "木曜日", "金曜日", "土曜日"}
        return fmt.Sprintf("%s%d年%d月%d日%s", eraName, eraYear, int(t.Month()), t.Day(), weekdays[t.Weekday()])
    default:
        return fmt.Sprintf("%d年%d月%d日", t.Year(), int(t.Month()), t.Day())
    }
}

func (ltf *LocalizedTimeFormatter) FormatTime(t time.Time, format24h bool) string {
    switch ltf.locale {
    case "en-US":
        if format24h {
            return t.Format("15:04:05")
        }
        return t.Format("3:04:05 PM")
    case "en-GB", "de-DE", "fr-FR":
        return t.Format("15:04:05")
    case "ja-JP":
        if format24h {
            return t.Format("15時04分05秒")
        }
        hour := t.Hour()
        ampm := "午前"
        if hour >= 12 {
            ampm = "午後"
            if hour > 12 {
                hour -= 12
            }
        }
        if hour == 0 {
            hour = 12
        }
        return fmt.Sprintf("%s%d時%02d分%02d秒", ampm, hour, t.Minute(), t.Second())
    default:
        return t.Format("15:04:05")
    }
}

func (ltf *LocalizedTimeFormatter) FormatRelativeTime(t time.Time, relativeTo time.Time) string {
    diff := relativeTo.Sub(t)
    
    switch ltf.locale {
    case "en-US", "en-GB":
        return ltf.formatEnglishRelative(diff)
    case "de-DE":
        return ltf.formatGermanRelative(diff)
    case "fr-FR":
        return ltf.formatFrenchRelative(diff)
    case "ja-JP":
        return ltf.formatJapaneseRelative(diff)
    default:
        return ltf.formatEnglishRelative(diff)
    }
}

func (ltf *LocalizedTimeFormatter) formatEnglishRelative(diff time.Duration) string {
    if diff < 0 {
        diff = -diff
        if diff < time.Minute {
            return "in a few seconds"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("in %d minutes", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("in %d hours", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("in %d days", days)
        }
    } else {
        if diff < time.Minute {
            return "a few seconds ago"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("%d minutes ago", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("%d hours ago", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("%d days ago", days)
        }
    }
}

func (ltf *LocalizedTimeFormatter) formatGermanRelative(diff time.Duration) string {
    if diff < 0 {
        diff = -diff
        if diff < time.Minute {
            return "in wenigen Sekunden"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("in %d Minuten", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("in %d Stunden", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("in %d Tagen", days)
        }
    } else {
        if diff < time.Minute {
            return "vor wenigen Sekunden"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("vor %d Minuten", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("vor %d Stunden", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("vor %d Tagen", days)
        }
    }
}

func (ltf *LocalizedTimeFormatter) formatFrenchRelative(diff time.Duration) string {
    if diff < 0 {
        diff = -diff
        if diff < time.Minute {
            return "dans quelques secondes"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("dans %d minutes", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("dans %d heures", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("dans %d jours", days)
        }
    } else {
        if diff < time.Minute {
            return "il y a quelques secondes"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("il y a %d minutes", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("il y a %d heures", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("il y a %d jours", days)
        }
    }
}

func (ltf *LocalizedTimeFormatter) formatJapaneseRelative(diff time.Duration) string {
    if diff < 0 {
        diff = -diff
        if diff < time.Minute {
            return "数秒後"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("%d分後", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("%d時間後", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("%d日後", days)
        }
    } else {
        if diff < time.Minute {
            return "数秒前"
        } else if diff < time.Hour {
            minutes := int(diff.Minutes())
            return fmt.Sprintf("%d分前", minutes)
        } else if diff < 24*time.Hour {
            hours := int(diff.Hours())
            return fmt.Sprintf("%d時間前", hours)
        } else {
            days := int(diff.Hours() / 24)
            return fmt.Sprintf("%d日前", days)
        }
    }
}

func main() {
    fmt.Println("Time Localization and Formatting")
    fmt.Println("================================")
    
    testTime := time.Date(2024, time.March, 15, 14, 30, 45, 0, time.UTC)
    now := time.Now()
    pastTime := now.Add(-2 * time.Hour)
    futureTime := now.Add(3 * time.Hour)
    
    locales := []string{"en-US", "en-GB", "de-DE", "fr-FR", "ja-JP"}
    styles := []string{"short", "medium", "long", "full"}
    
    fmt.Printf("Test time: %s\n", testTime.Format("2006-01-02 15:04:05 UTC"))
    fmt.Println(strings.Repeat("=", 60))
    
    // Date formatting by locale and style
    for _, locale := range locales {
        fmt.Printf("\n%s Locale:\n", locale)
        fmt.Println(strings.Repeat("-", 20))
        
        formatter := NewLocalizedTimeFormatter(locale)
        
        for _, style := range styles {
            formatted := formatter.FormatDate(testTime, style)
            fmt.Printf("  %-8s: %s\n", style, formatted)
        }
        
        // Time formatting
        time24h := formatter.FormatTime(testTime, true)
        time12h := formatter.FormatTime(testTime, false)
        fmt.Printf("  Time 24h: %s\n", time24h)
        fmt.Printf("  Time 12h: %s\n", time12h)
        
        // Relative time formatting
        pastRelative := formatter.FormatRelativeTime(pastTime, now)
        futureRelative := formatter.FormatRelativeTime(futureTime, now)
        fmt.Printf("  Past:     %s\n", pastRelative)
        fmt.Printf("  Future:   %s\n", futureRelative)
    }
    
    // Demonstrate different time scenarios
    fmt.Println("\nRelative Time Examples:")
    fmt.Println(strings.Repeat("=", 30))
    
    formatter := NewLocalizedTimeFormatter("en-US")
    
    relativeTimes := []time.Duration{
        -30 * time.Second,
        -5 * time.Minute,
        -2 * time.Hour,
        -3 * 24 * time.Hour,
        30 * time.Second,
        10 * time.Minute,
        4 * time.Hour,
        7 * 24 * time.Hour,
    }
    
    for _, offset := range relativeTimes {
        testTime := now.Add(offset)
        relative := formatter.FormatRelativeTime(testTime, now)
        fmt.Printf("  %s: %s\n", testTime.Format("2006-01-02 15:04"), relative)
    }
    
    fmt.Println("\nTime Formatting Best Practices:")
    fmt.Println("- Always consider user's locale and preferences")
    fmt.Println("- Provide both 12h and 24h time format options")
    fmt.Println("- Use relative time for recent events")
    fmt.Println("- Consider cultural date ordering (MM/DD vs DD/MM)")
    fmt.Println("- Handle right-to-left languages appropriately")
    fmt.Println("- Test with edge cases and different timezones")
}
```

Time localization and formatting ensure global application usability,  
cultural appropriateness, improved user experience, and compliance with  
regional date and time representation conventions across different markets.  

