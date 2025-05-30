# **Chapter 14: Common Packages in Go**

Go's standard library is extensive and provides a rich set of tools for everyday programming tasks. Rather than relying heavily on third-party libraries, Go encourages developers to leverage its standard library, which is well-designed, thoroughly tested, and optimized for performance.

In this chapter, we'll explore some of the most commonly used packages in Go's standard library, including `fmt` for formatted I/O, `time` for date and time operations, `math/rand` for random number generation, and others. Understanding these packages will help you write idiomatic Go code and solve common programming problems efficiently.

## **14.1 The fmt Package**

The `fmt` package provides functions for formatted I/O operations, similar to C's printf and scanf functions. It's one of the most frequently used packages in Go programs.

### **14.1.1 Printing to Standard Output**

The `fmt` package offers several functions for printing to standard output:

```go
package main

import "fmt"

func main() {
    // Print without a newline
    fmt.Print("Hello, ")
    fmt.Print("world!\n")

    // Print with a newline
    fmt.Println("Hello, world!")

    // Formatted print
    fmt.Printf("My name is %s and I'm %d years old.\n", "Alice", 30)

    // Print with dynamic padding and alignment
    fmt.Printf("|%-10s|%10s|\n", "Left", "Right")
    fmt.Printf("|%-10d|%10d|\n", 123, 456)
}
```

Output:

```
Hello, world!
Hello, world!
My name is Alice and I'm 30 years old.
|Left      |     Right|
|123       |       456|
```

### **14.1.2 Formatting Verbs**

The `fmt` package supports various formatting verbs for different types:

```go
package main

import "fmt"

func main() {
    // General
    fmt.Printf("%v\n", 123)         // Default format: 123
    fmt.Printf("%#v\n", "hello")    // Go syntax format: "hello"
    fmt.Printf("%T\n", 123.45)      // Type: float64

    // Integer
    fmt.Printf("%d\n", 123)         // Decimal: 123
    fmt.Printf("%b\n", 5)           // Binary: 101
    fmt.Printf("%o\n", 10)          // Octal: 12
    fmt.Printf("%x\n", 15)          // Hex: f
    fmt.Printf("%X\n", 15)          // Hex: F

    // Float
    fmt.Printf("%f\n", 123.456)     // Decimal: 123.456000
    fmt.Printf("%.2f\n", 123.456)   // Precision: 123.46
    fmt.Printf("%e\n", 123.456)     // Scientific: 1.234560e+02

    // String
    fmt.Printf("%s\n", "hello")     // String: hello
    fmt.Printf("%q\n", "hello")     // Quoted: "hello"

    // Boolean
    fmt.Printf("%t\n", true)        // Boolean: true

    // Pointer
    x := 10
    fmt.Printf("%p\n", &x)          // Pointer: 0xc000016098
}
```

### **14.1.3 String Formatting**

The `fmt` package provides functions to format strings without printing them:

```go
package main

import "fmt"

func main() {
    // Sprintf returns a formatted string
    name := "Bob"
    age := 25
    greeting := fmt.Sprintf("Hello, %s! You are %d years old.", name, age)
    fmt.Println(greeting)

    // Formatting a float with specific precision
    pi := 3.14159265359
    piStr := fmt.Sprintf("Pi: %.3f", pi)
    fmt.Println(piStr)

    // Using width and precision
    formatted := fmt.Sprintf("|%10.2f|%-10.2f|", 12.34, 56.78)
    fmt.Println(formatted)
}
```

Output:

```
Hello, Bob! You are 25 years old.
Pi: 3.142
|     12.34|56.78     |
```

### **14.1.4 Reading Input**

The `fmt` package also provides functions for reading formatted input:

```go
package main

import "fmt"

func main() {
    var name string
    var age int

    fmt.Print("Enter your name: ")
    fmt.Scan(&name)

    fmt.Print("Enter your age: ")
    fmt.Scan(&age)

    fmt.Printf("Hello, %s! You are %d years old.\n", name, age)

    // Reading multiple values
    var x, y int
    fmt.Print("Enter two numbers: ")
    count, err := fmt.Scanf("%d %d", &x, &y)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Printf("Read %d values. Sum: %d\n", count, x+y)
    }
}
```

Note: When running this program, you'll need to provide input via the command line.

## **14.2 The time Package**

The `time` package provides functionality for measuring and displaying time.

### **14.2.1 Getting the Current Time**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Get current time
    now := time.Now()
    fmt.Println("Current time:", now)

    // Extract components
    fmt.Println("Year:", now.Year())
    fmt.Println("Month:", now.Month())
    fmt.Println("Day:", now.Day())
    fmt.Println("Hour:", now.Hour())
    fmt.Println("Minute:", now.Minute())
    fmt.Println("Second:", now.Second())

    // Get weekday
    fmt.Println("Weekday:", now.Weekday())

    // Get day of year
    fmt.Println("Day of year:", now.YearDay())
}
```

Output (will vary):

```
Current time: 2023-05-10 14:30:45.123456789 +0200 CEST
Year: 2023
Month: May
Day: 10
Hour: 14
Minute: 30
Second: 45
Weekday: Wednesday
Day of year: 130
```

### **14.2.2 Creating Time Values**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a specific time
    t := time.Date(2023, time.January, 15, 14, 30, 45, 0, time.UTC)
    fmt.Println("Created time:", t)

    // Parse time from string
    t1, err := time.Parse("2006-01-02 15:04:05", "2023-05-10 14:30:45")
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("Parsed time:", t1)
    }

    // Parse with timezone
    loc, _ := time.LoadLocation("America/New_York")
    t2, _ := time.ParseInLocation("2006-01-02 15:04:05", "2023-05-10 14:30:45", loc)
    fmt.Println("Parsed time with location:", t2)
}
```

Output:

```
Created time: 2023-01-15 14:30:45 +0000 UTC
Parsed time: 2023-05-10 14:30:45 +0000 UTC
Parsed time with location: 2023-05-10 14:30:45 -0400 EDT
```

Note the special format string used in `time.Parse`. Go uses a reference time (January 2, 2006 at 15:04:05 in time zone MST) to define the format layout.

### **14.2.3 Time Formatting**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    t := time.Date(2023, time.April, 15, 14, 30, 45, 0, time.UTC)

    // Predefined formats
    fmt.Println(t.Format(time.RFC3339))      // 2023-04-15T14:30:45Z
    fmt.Println(t.Format(time.RFC822))       // 15 Apr 23 14:30 UTC
    fmt.Println(t.Format(time.RFC1123))      // Sat, 15 Apr 2023 14:30:45 UTC

    // Custom formats
    fmt.Println(t.Format("2006-01-02"))      // 2023-04-15
    fmt.Println(t.Format("15:04:05"))        // 14:30:45
    fmt.Println(t.Format("Monday, January 2, 2006")) // Saturday, April 15, 2023
    fmt.Println(t.Format("3:04 PM"))         // 2:30 PM
}
```

### **14.2.4 Time Calculations**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()

    // Add duration
    future := now.Add(24 * time.Hour)
    fmt.Println("Tomorrow:", future.Format("2006-01-02"))

    // Subtract duration
    past := now.Add(-72 * time.Hour)
    fmt.Println("3 days ago:", past.Format("2006-01-02"))

    // Add by units
    laterDate := now.AddDate(1, 2, 3) // 1 year, 2 months, 3 days
    fmt.Println("Later date:", laterDate.Format("2006-01-02"))

    // Time difference
    start := time.Date(2023, time.January, 1, 0, 0, 0, 0, time.UTC)
    end := time.Date(2023, time.December, 31, 23, 59, 59, 0, time.UTC)
    diff := end.Sub(start)
    fmt.Printf("Time difference: %.2f days\n", diff.Hours()/24)

    // Compare times
    fmt.Println("Before:", now.Before(future))
    fmt.Println("After:", now.After(past))
    fmt.Println("Equal:", now.Equal(now))
}
```

Output (will vary):

```
Tomorrow: 2023-05-11
3 days ago: 2023-05-07
Later date: 2024-07-13
Time difference: 364.00 days
Before: true
After: true
Equal: true
```

### **14.2.5 Timers and Tickers**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // One-time timer
    timer := time.NewTimer(2 * time.Second)
    fmt.Println("Timer started at", time.Now().Format("15:04:05"))

    <-timer.C // Block until timer fires
    fmt.Println("Timer fired at", time.Now().Format("15:04:05"))

    // Simple sleep
    fmt.Println("Sleeping for 1 second...")
    time.Sleep(1 * time.Second)
    fmt.Println("Awake!")

    // Recurring ticker
    ticker := time.NewTicker(500 * time.Millisecond)
    counter := 0

    // Run ticker for 3 seconds
    go func() {
        for t := range ticker.C {
            counter++
            fmt.Println("Tick at", t.Format("15:04:05.000"))
            if counter >= 6 {
                ticker.Stop()
                break
            }
        }
    }()

    // Wait for ticker to finish
    time.Sleep(3 * time.Second)
    fmt.Println("Ticker stopped")
}
```

Output (will vary):

```
Timer started at 14:30:45
Timer fired at 14:30:47
Sleeping for 1 second...
Awake!
Tick at 14:30:48.500
Tick at 14:30:49.000
Tick at 14:30:49.500
Tick at 14:30:50.000
Tick at 14:30:50.500
Tick at 14:30:51.000
Ticker stopped
```

## **14.3 The math/rand Package**

The `math/rand` package implements pseudo-random number generators for various distributions.

### **14.3.1 Basic Random Number Generation**

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    // Without seeding, you'll get the same sequence every time
    fmt.Println("Without seed:")
    for i := 0; i < 3; i++ {
        fmt.Println(rand.Intn(100)) // Random int [0, 100)
    }

    // Seed with current time for different sequences each run
    rand.Seed(time.Now().UnixNano())
    fmt.Println("\nWith time seed:")
    for i := 0; i < 3; i++ {
        fmt.Println(rand.Intn(100)) // Random int [0, 100)
    }
}
```

Note: In Go 1.20+, you don't need to explicitly seed the random number generator as it's automatically seeded with a random value at program startup.

### **14.3.2 Different Random Distributions**

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())

    // Integer between 0 and n-1
    fmt.Println("rand.Intn(10):", rand.Intn(10))

    // Float64 in range [0.0, 1.0)
    fmt.Println("rand.Float64():", rand.Float64())

    // Integer in range [min, max)
    min, max := 5, 15
    rangeInt := min + rand.Intn(max-min)
    fmt.Printf("Random integer in range [%d, %d): %d\n", min, max, rangeInt)

    // Float64 in range [min, max)
    minF, maxF := 5.0, 15.0
    rangeFloat := minF + rand.Float64()*(maxF-minF)
    fmt.Printf("Random float in range [%.2f, %.2f): %.2f\n", minF, maxF, rangeFloat)

    // Normal distribution (bell curve)
    // mean=0, stddev=1
    fmt.Println("Normal distribution:", rand.NormFloat64())

    // Exponential distribution
    // rate=1
    fmt.Println("Exponential distribution:", rand.ExpFloat64())
}
```

### **14.3.3 Random Shuffling**

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())

    // Shuffle a slice
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
    fmt.Println("Original:", numbers)

    rand.Shuffle(len(numbers), func(i, j int) {
        numbers[i], numbers[j] = numbers[j], numbers[i]
    })

    fmt.Println("Shuffled:", numbers)

    // Random permutation
    perm := rand.Perm(5) // [0,5) permutation
    fmt.Println("Random permutation:", perm)
}
```

Output (will vary):

```
Original: [1 2 3 4 5 6 7 8 9 10]
Shuffled: [7 2 6 9 1 4 8 3 10 5]
Random permutation: [3 1 0 4 2]
```

### **14.3.4 Creating a Random Source**

For concurrent applications, it's better to create separate sources to avoid lock contention:

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    // Create a random source
    source := rand.NewSource(time.Now().UnixNano())
    r := rand.New(source)

    // Use the custom random generator
    for i := 0; i < 3; i++ {
        fmt.Println(r.Intn(100))
    }

    // Creating multiple sources for concurrent use
    source1 := rand.NewSource(time.Now().UnixNano())
    r1 := rand.New(source1)

    source2 := rand.NewSource(time.Now().UnixNano() + 1)
    r2 := rand.New(source2)

    fmt.Println("\nFrom r1:", r1.Intn(100))
    fmt.Println("From r2:", r2.Intn(100))
}
```

## **14.4 Other Essential Packages**

### **14.4.1 The strings Package**

The `strings` package provides functions for manipulating UTF-8 encoded strings.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    s := "Hello, Go World!"

    // Basic functions
    fmt.Println("Contains 'Go':", strings.Contains(s, "Go"))
    fmt.Println("HasPrefix:", strings.HasPrefix(s, "Hello"))
    fmt.Println("HasSuffix:", strings.HasSuffix(s, "World!"))
    fmt.Println("Count of 'o':", strings.Count(s, "o"))
    fmt.Println("Index of 'Go':", strings.Index(s, "Go"))

    // Transformations
    fmt.Println("ToUpper:", strings.ToUpper(s))
    fmt.Println("ToLower:", strings.ToLower(s))
    fmt.Println("Replace:", strings.Replace(s, "o", "0", 2)) // Replace first 2 occurrences
    fmt.Println("ReplaceAll:", strings.ReplaceAll(s, "o", "0"))

    // Trimming
    padded := "  trimmed  "
    fmt.Println("Trim spaces:", strings.TrimSpace(padded))

    // Splitting and joining
    parts := strings.Split(s, ", ")
    fmt.Println("Split:", parts)
    joined := strings.Join(parts, " & ")
    fmt.Println("Joined:", joined)

    // Checking if string contains only certain characters
    fmt.Println("Only digits:", strings.ContainsAny("12345", "0123456789"))
    fmt.Println("Any vowels:", strings.ContainsAny("hello", "aeiou"))

    // Builder for efficient string concatenation
    var builder strings.Builder
    builder.WriteString("Hello")
    builder.WriteString(", ")
    builder.WriteString("world")
    builder.WriteString("!")
    result := builder.String()
    fmt.Println("Built string:", result)
}
```

Output:

```
Contains 'Go': true
HasPrefix: true
HasSuffix: true
Count of 'o': 2
Index of 'Go': 7
ToUpper: HELLO, GO WORLD!
ToLower: hello, go world!
Replace: Hell0, G0 World!
ReplaceAll: Hell0, G0 W0rld!
Trim spaces: trimmed
Split: [Hello Go World!]
Joined: Hello & Go World!
Only digits: true
Any vowels: true
Built string: Hello, world!
```

### **14.4.2 The strconv Package**

The `strconv` package provides functions for converting strings to various data types and vice versa.

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // String to integer
    i, err := strconv.Atoi("123")
    if err == nil {
        fmt.Println("String to int:", i)
    }

    // Integer to string
    s := strconv.Itoa(456)
    fmt.Println("Int to string:", s)

    // Parse boolean
    b, _ := strconv.ParseBool("true")
    fmt.Println("String to bool:", b)

    // Parse float
    f, _ := strconv.ParseFloat("3.14159", 64)
    fmt.Println("String to float64:", f)

    // Parse int with base
    hex, _ := strconv.ParseInt("1a", 16, 0)
    fmt.Println("Hex string to int:", hex)

    // Format boolean
    bs := strconv.FormatBool(true)
    fmt.Println("Bool to string:", bs)

    // Format float
    fs := strconv.FormatFloat(3.14159, 'f', 2, 64) // 'f' format, 2 decimal places
    fmt.Println("Float to string (fixed):", fs)

    es := strconv.FormatFloat(3.14159, 'e', 2, 64) // 'e' format, 2 decimal places
    fmt.Println("Float to string (scientific):", es)

    // Format int with base
    is := strconv.FormatInt(26, 16) // Base 16 (hex)
    fmt.Println("Int to hex string:", is)
}
```

Output:

```
String to int: 123
Int to string: 456
String to bool: true
String to float64: 3.14159
Hex string to int: 26
Bool to string: true
Float to string (fixed): 3.14
Float to string (scientific): 3.14e+00
Int to hex string: 1a
```

### **14.4.3 The sort Package**

The `sort` package provides primitives for sorting slices and user-defined collections.

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    // Sorting ints
    numbers := []int{3, 1, 4, 1, 5, 9, 2, 6}
    fmt.Println("Before sort:", numbers)
    sort.Ints(numbers)
    fmt.Println("After sort:", numbers)

    // Sorting strings
    names := []string{"Charlie", "Alice", "Bob", "David"}
    fmt.Println("Before sort:", names)
    sort.Strings(names)
    fmt.Println("After sort:", names)

    // Sorting floats
    floats := []float64{3.14, 1.41, 2.71, 1.62}
    fmt.Println("Before sort:", floats)
    sort.Float64s(floats)
    fmt.Println("After sort:", floats)

    // Checking if sorted
    fmt.Println("Ints sorted:", sort.IntsAreSorted(numbers))

    // Searching in sorted slice (binary search)
    idx := sort.SearchInts(numbers, 5)
    fmt.Println("Index of 5:", idx)

    // Custom sort: sort by length of string
    fruits := []string{"banana", "kiwi", "apple", "strawberry", "fig"}
    fmt.Println("Before custom sort:", fruits)

    sort.Slice(fruits, func(i, j int) bool {
        return len(fruits[i]) < len(fruits[j])
    })

    fmt.Println("After custom sort (by length):", fruits)

    // Sorting structs
    type Person struct {
        Name string
        Age  int
    }

    people := []Person{
        {"Alice", 25},
        {"Bob", 30},
        {"Charlie", 20},
    }

    // Sort by age
    sort.Slice(people, func(i, j int) bool {
        return people[i].Age < people[j].Age
    })

    fmt.Println("People sorted by age:")
    for _, p := range people {
        fmt.Printf("  %s: %d\n", p.Name, p.Age)
    }
}
```

Output:

```
Before sort: [3 1 4 1 5 9 2 6]
After sort: [1 1 2 3 4 5 6 9]
Before sort: [Charlie Alice Bob David]
After sort: [Alice Bob Charlie David]
Before sort: [3.14 1.41 2.71 1.62]
After sort: [1.41 1.62 2.71 3.14]
Ints sorted: true
Index of 5: 4
Before custom sort: [banana kiwi apple strawberry fig]
After custom sort (by length): [fig kiwi apple banana strawberry]
People sorted by age:
  Charlie: 20
  Alice: 25
  Bob: 30
```

## **14.5 Exercises**

### **Exercise 1: String Processing with the fmt and strings Packages**

Create a program that reads a sentence from the user, counts the number of words, and prints the words in reverse order.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var input string

    fmt.Print("Enter a sentence: ")
    fmt.Scanln(&input)

    // Split the sentence into words
    words := strings.Fields(input)

    // Count the words
    fmt.Printf("Word count: %d\n", len(words))

    // Print words in reverse order
    fmt.Println("Words in reverse order:")
    for i := len(words) - 1; i >= 0; i-- {
        fmt.Println(words[i])
    }
}
```

### **Exercise 2: Time Calculations**

Create a program that calculates the time difference between two dates and displays it in years, months, days, hours, minutes, and seconds.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Parse two time values
    t1, _ := time.Parse("2006-01-02 15:04:05", "2020-01-01 00:00:00")
    t2, _ := time.Parse("2006-01-02 15:04:05", "2023-06-15 12:30:45")

    // Calculate the difference
    diff := t2.Sub(t1)

    // Extract days, hours, minutes, seconds
    days := int(diff.Hours() / 24)
    hours := int(diff.Hours()) % 24
    minutes := int(diff.Minutes()) % 60
    seconds := int(diff.Seconds()) % 60

    // For years and months, use a different approach
    years := 0
    months := 0

    // Start from t1 and add years until we exceed t2
    currentTime := t1
    for currentTime.AddDate(1, 0, 0).Before(t2) {
        currentTime = currentTime.AddDate(1, 0, 0)
        years++
    }

    // Add months until we exceed t2
    for currentTime.AddDate(0, 1, 0).Before(t2) {
        currentTime = currentTime.AddDate(0, 1, 0)
        months++
    }

    // Calculate remaining days
    remainingDays := int(t2.Sub(currentTime).Hours() / 24)

    fmt.Printf("Time difference: %d years, %d months, %d days, %d hours, %d minutes, %d seconds\n",
        years, months, remainingDays, hours, minutes, seconds)
    fmt.Printf("Total days: %d\n", days)
}
```

### **Exercise 3: Random Number Game**

Create a number guessing game where the computer generates a random number and the user tries to guess it. The program should provide hints ("higher" or "lower") after each guess.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())

    // Generate a random number between 1 and 100
    target := rand.Intn(100) + 1

    fmt.Println("I'm thinking of a number between 1 and 100.")
    fmt.Println("Try to guess it!")

    attempts := 0
    for {
        var guess int
        fmt.Print("Your guess: ")
        fmt.Scan(&guess)
        attempts++

        if guess < target {
            fmt.Println("Higher!")
        } else if guess > target {
            fmt.Println("Lower!")
        } else {
            fmt.Printf("Correct! You guessed it in %d attempts.\n", attempts)
            break
        }
    }
}
```

## **14.6 Summary**

In this chapter, we've explored some of the most commonly used packages in Go's standard library:

- **fmt**: For formatted I/O operations, including printing, scanning, and string formatting
- **time**: For working with dates, times, durations, and timers
- **math/rand**: For generating random numbers and shuffling collections
- **strings**: For manipulating and processing strings
- **strconv**: For converting between strings and other data types
- **sort**: For sorting slices and user-defined collections

These packages form the foundation of many Go programs and provide essential functionality for everyday programming tasks. By mastering these packages, you'll be able to write idiomatic Go code that is both efficient and maintainable.

Go's standard library is much more extensive than what we've covered here. As you continue your Go journey, explore other packages like `io`, `os`, `net/http`, `encoding/json`, and more, which provide functionality for file I/O, networking, and data encoding/decoding.
