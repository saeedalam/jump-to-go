# **Chapter 5: Mastering Control Flow in Go**

Control flow determines the execution path of your program—which instructions run, which skip, and which repeat. Go's elegant control structures strike a balance between simplicity and power, giving you everything you need without unnecessary complexity.

In this chapter, you'll master Go's three primary control structures:

- **Conditional execution** with `if` statements
- **Multi-way branching** with `switch` statements
- **Iteration** with the versatile `for` loop

You'll learn not just the basic syntax, but also idiomatic patterns, advanced techniques, and Go-specific features that make your code more expressive and efficient.

## **5.1 Conditional Logic with `if` Statements**

The `if` statement allows your program to make decisions by executing different code blocks based on conditions.

### **5.1.1 Basic Syntax and Usage**

```go
if condition {
    // Code executed when condition is true
} else if anotherCondition {
    // Code executed when anotherCondition is true
} else {
    // Code executed when all conditions are false
}
```

Here's a simple example to check if a number is positive, negative, or zero:

```go
package main

import "fmt"

func main() {
    num := -5

    if num > 0 {
        fmt.Println("Positive number")
    } else if num < 0 {
        fmt.Println("Negative number")
    } else {
        fmt.Println("Zero")
    }
}
```

### **5.1.2 Go's Special Feature: Initialization Statement**

Go allows you to initialize a variable within the `if` statement itself. This is useful for variables that are only needed within the conditional block:

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    // Seed the random number generator
    rand.Seed(time.Now().UnixNano())

    // Initialize randomNum within the if statement
    if randomNum := rand.Intn(10); randomNum < 5 {
        fmt.Println(randomNum, "is less than 5")
    } else {
        fmt.Println(randomNum, "is 5 or greater")
    }

    // randomNum is not accessible here - scoped to the if-else block
    // fmt.Println(randomNum) // This would cause a compilation error
}
```

This feature:

- Keeps variables scoped tightly to where they're used
- Makes code more readable by placing initialization close to its condition
- Reduces the chance of using the variable incorrectly elsewhere

### **5.1.3 Conditional Operators and Boolean Logic**

Go uses standard comparison operators and logical operators:

| Operator | Description              |
| -------- | ------------------------ |
| `==`     | Equal to                 |
| `!=`     | Not equal to             |
| `<`      | Less than                |
| `<=`     | Less than or equal to    |
| `>`      | Greater than             |
| `>=`     | Greater than or equal to |
| `&&`     | Logical AND              |
| `\|\|`   | Logical OR               |
| `!`      | Logical NOT              |

```go
age := 22
hasID := true

if age >= 21 && hasID {
    fmt.Println("You can enter the venue and purchase alcohol.")
} else if age >= 18 && hasID {
    fmt.Println("You can enter the venue but cannot purchase alcohol.")
} else {
    fmt.Println("You cannot enter the venue.")
}
```

### **5.1.4 Common Pitfalls with `if` Statements**

#### **1. Forgetting Curly Braces**

In Go, curly braces are mandatory for `if` statements, even if there's only one statement in the block:

```go
// Incorrect - will not compile
if x > 0
    fmt.Println("Positive")

// Correct
if x > 0 {
    fmt.Println("Positive")
}
```

#### **2. Condition Must Be a Boolean**

Unlike some languages, Go requires conditions to be boolean expressions:

```go
value := 5

// Incorrect - will not compile
if value {
    fmt.Println("True")
}

// Correct
if value != 0 {
    fmt.Println("Non-zero")
}
```

#### **3. Accidentally Using Assignment Instead of Comparison**

```go
x := 10

// This assigns 5 to x and then evaluates to true (since 5 is non-zero)
// Luckily, Go doesn't allow this: "expected boolean expression"
if x = 5 {
    fmt.Println("This won't compile")
}

// For intentional assignment and check, you need to do:
if x = 5; x > 0 {
    fmt.Println("x is now 5, which is positive")
}
```

### **5.1.5 Best Practices for Conditional Logic**

1. **Keep conditions simple and readable**

   ```go
   // Hard to read
   if age >= 18 && age <= 65 && !hasDisability && !isPartTime {
       // ...
   }

   // Better - break it down
   isWorkingAge := age >= 18 && age <= 65
   isEligible := !hasDisability && !isPartTime
   if isWorkingAge && isEligible {
       // ...
   }
   ```

2. **Return early to reduce nesting**

   ```go
   // Deeply nested code is hard to follow
   func processRequest(req Request) Response {
       if req.IsValid() {
           if req.HasPermission() {
               if req.ResourceExists() {
                   // Process valid request
                   return SuccessResponse()
               } else {
                   return NotFoundResponse()
               }
           } else {
               return ForbiddenResponse()
           }
       } else {
           return BadRequestResponse()
       }
   }

   // Better with early returns
   func processRequest(req Request) Response {
       if !req.IsValid() {
           return BadRequestResponse()
       }

       if !req.HasPermission() {
           return ForbiddenResponse()
       }

       if !req.ResourceExists() {
           return NotFoundResponse()
       }

       // Process valid request
       return SuccessResponse()
   }
   ```

## **5.2 Multi-Way Decisions with `switch` Statements**

The `switch` statement provides a cleaner way to handle multiple conditions, especially when comparing a single value against multiple possible matches.

### **5.2.1 Basic Syntax and Usage**

```go
switch expression {
case value1:
    // Code executed when expression == value1
case value2, value3:
    // Code executed when expression == value2 OR expression == value3
default:
    // Code executed when no cases match
}
```

Here's a simple example using a weekday:

```go
package main

import "fmt"

func main() {
    day := "Wednesday"

    switch day {
    case "Monday":
        fmt.Println("Start of work week")
    case "Wednesday":
        fmt.Println("Middle of work week")
    case "Friday":
        fmt.Println("End of work week")
    case "Saturday", "Sunday":
        fmt.Println("Weekend!")
    default:
        fmt.Println("Regular work day")
    }
}
```

### **5.2.2 Go's `switch` Unique Features**

#### **1. Automatic Break**

Unlike C, C++, and Java, Go's `switch` statements **don't fall through** by default. Each case automatically breaks after its code executes:

```go
switch num {
case 1:
    fmt.Println("One")
    // No need for "break" - it's automatic
case 2:
    fmt.Println("Two")
}
```

#### **2. Optional Expression**

Go allows a `switch` without an expression, which can replace complex `if-else` chains:

```go
score := 85

switch {
case score >= 90:
    fmt.Println("A")
case score >= 80:
    fmt.Println("B")
case score >= 70:
    fmt.Println("C")
case score >= 60:
    fmt.Println("D")
default:
    fmt.Println("F")
}
```

#### **3. Fallthrough**

If you do want a case to fall through to the next one, use the `fallthrough` keyword:

```go
switch num := 2; num {
case 1:
    fmt.Println("One")
case 2:
    fmt.Println("Two")
    fallthrough
case 3:
    fmt.Println("Three or more")
}
```

This will print both "Two" and "Three or more".

#### **4. Type Switches**

Go can switch on types, which is especially useful when working with interfaces:

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case bool:
        fmt.Printf("Boolean: %v\n", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

func main() {
    describe(42)
    describe("hello")
    describe(true)
    describe(3.14)
}
```

### **5.2.3 Best Practices for Switch Statements**

1. **Prefer `switch` over long `if-else` chains**

   ```go
   // Long if-else chain
   if input == "y" || input == "Y" {
       // ...
   } else if input == "n" || input == "N" {
       // ...
   } else if input == "q" || input == "Q" {
       // ...
   } else {
       // ...
   }

   // Cleaner with switch
   switch input {
   case "y", "Y":
       // ...
   case "n", "N":
       // ...
   case "q", "Q":
       // ...
   default:
       // ...
   }
   ```

2. **Use the expressionless `switch` for range comparisons**

   ```go
   switch {
   case age < 13:
       fmt.Println("Child")
   case age < 20:
       fmt.Println("Teenager")
   case age < 65:
       fmt.Println("Adult")
   default:
       fmt.Println("Senior")
   }
   ```

3. **Be careful with `fallthrough`**
   Use it sparingly and document your intent when you do use it, as it can make code harder to follow.

## **5.3 Iteration with `for` Loops**

The `for` loop is Go's only loop construct, but it's flexible enough to handle all looping scenarios.

### **5.3.1 Basic Syntax and Variations**

#### **1. Standard Three-Component Loop**

```go
for initialization; condition; post {
    // Loop body
}
```

Example:

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

#### **2. While-Style Loop**

```go
for condition {
    // Loop body
}
```

Example:

```go
count := 0
for count < 5 {
    fmt.Println(count)
    count++
}
```

#### **3. Infinite Loop**

```go
for {
    // Loop body
    if someCondition {
        break
    }
}
```

Example:

```go
count := 0
for {
    fmt.Println(count)
    count++
    if count >= 5 {
        break
    }
}
```

#### **4. Iterating with `range`**

The `range` form iterates over elements in various data structures:

```go
for key, value := range collection {
    // Loop body using key and value
}
```

Examples:

```go
// Iterating over a slice
fruits := []string{"Apple", "Banana", "Cherry"}
for i, fruit := range fruits {
    fmt.Printf("%d: %s\n", i, fruit)
}

// Iterating over a map
ages := map[string]int{
    "Alice": 25,
    "Bob":   30,
    "Carol": 27,
}
for name, age := range ages {
    fmt.Printf("%s is %d years old\n", name, age)
}

// Iterating over a string (by rune)
for i, char := range "Hello" {
    fmt.Printf("%d: %c\n", i, char)
}
```

### **5.3.2 Loop Control: `break` and `continue`**

#### **`break`**

Exits the innermost loop immediately:

```go
for i := 0; i < 10; i++ {
    if i == 5 {
        break // Exits when i reaches 5
    }
    fmt.Println(i)
}
// Prints: 0 1 2 3 4
```

#### **`continue`**

Skips to the next iteration:

```go
for i := 0; i < 10; i++ {
    if i%2 == 0 {
        continue // Skip even numbers
    }
    fmt.Println(i)
}
// Prints: 1 3 5 7 9
```

### **5.3.3 Labeled Statements**

Go supports labels to break or continue outer loops:

```go
outer:
    for i := 0; i < 3; i++ {
        for j := 0; j < 3; j++ {
            if i*j >= 3 {
                fmt.Println("Breaking outer loop when i =", i, "and j =", j)
                break outer
            }
            fmt.Printf("(%d, %d) ", i, j)
        }
        fmt.Println()
    }
```

### **5.3.4 Common Loop Patterns in Go**

#### **1. Processing a Slice**

```go
items := []string{"Apple", "Banana", "Cherry"}

// Using index
for i := 0; i < len(items); i++ {
    fmt.Println(items[i])
}

// Using range (preferred)
for _, item := range items {
    fmt.Println(item)
}
```

#### **2. Processing a Map**

```go
userRoles := map[string]string{
    "alice": "admin",
    "bob":   "user",
    "carol": "manager",
}

// Note: Map iteration order is not guaranteed
for user, role := range userRoles {
    fmt.Printf("%s is a %s\n", user, role)
}
```

#### **3. Iterating Over Channels**

```go
ch := make(chan int)

// In a separate goroutine
go func() {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch)
}()

// Range over channel until it's closed
for num := range ch {
    fmt.Println(num)
}
```

### **5.3.5 Common Pitfalls with Loops**

#### **1. Variable Capture in Closures**

```go
functions := []func(){}

// Incorrect: all functions will print the same value
for i := 0; i < 5; i++ {
    functions = append(functions, func() {
        fmt.Println(i) // Captures reference to i, not its value
    })
}

// Correct: create a new variable in each iteration
for i := 0; i < 5; i++ {
    i := i // Creates a new variable with the same name (shadowing)
    functions = append(functions, func() {
        fmt.Println(i)
    })
}
```

#### **2. Modifying a Slice While Iterating**

```go
numbers := []int{1, 2, 3, 4, 5}

// Incorrect: unpredictable when modifying the slice you're ranging over
for i, num := range numbers {
    if num%2 == 0 {
        numbers = append(numbers[:i], numbers[i+1:]...) // Don't do this!
    }
}

// Correct: use a separate slice or iterate backward
var filtered []int
for _, num := range numbers {
    if num%2 != 0 {
        filtered = append(filtered, num)
    }
}
numbers = filtered
```

#### **3. Inefficient String Concatenation in Loops**

```go
// Inefficient: creates a new string each iteration
var result string
for i := 0; i < 1000; i++ {
    result += fmt.Sprintf("%d", i)
}

// Better: use strings.Builder
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString(fmt.Sprintf("%d", i))
}
result = builder.String()
```

## **5.4 Advanced Control Flow Patterns**

### **5.4.1 Combining Control Structures**

Complex algorithms often require nested or sequential control structures:

```go
func processItems(items []int) []int {
    var results []int

    for _, item := range items {
        // Skip negative numbers
        if item < 0 {
            continue
        }

        // Process based on value
        switch {
        case item%3 == 0 && item%5 == 0:
            results = append(results, item*3) // Divisible by both 3 and 5
        case item%3 == 0:
            results = append(results, item*2) // Divisible by 3
        case item%5 == 0:
            results = append(results, item+1) // Divisible by 5
        default:
            results = append(results, item)
        }

        // Stop after collecting 10 results
        if len(results) >= 10 {
            break
        }
    }

    return results
}
```

### **5.4.2 Error Handling Patterns**

Go's error handling often integrates with control structures:

```go
func processFile(filename string) error {
    file, err := os.Open(filename)
    if err != nil {
        return fmt.Errorf("opening file: %w", err)
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    lineCount := 0

    for scanner.Scan() {
        line := scanner.Text()
        lineCount++

        if strings.TrimSpace(line) == "" {
            continue // Skip empty lines
        }

        if err := processLine(line); err != nil {
            return fmt.Errorf("processing line %d: %w", lineCount, err)
        }
    }

    if err := scanner.Err(); err != nil {
        return fmt.Errorf("reading file: %w", err)
    }

    return nil
}
```

### **5.4.3 Implementing State Machines**

Control structures are perfect for implementing state machines:

```go
type State int

const (
    StateInit State = iota
    StateProcessing
    StateWaiting
    StateFinished
    StateError
)

func runStateMachine(events <-chan Event) Result {
    state := StateInit
    var data Result

    for event := range events {
        switch state {
        case StateInit:
            if event.Type == EventStart {
                state = StateProcessing
                data.StartTime = event.Time
            } else {
                state = StateError
                data.Error = fmt.Errorf("unexpected event %v in Init state", event.Type)
            }

        case StateProcessing:
            switch event.Type {
            case EventData:
                data.Items = append(data.Items, event.Data)
            case EventPause:
                state = StateWaiting
                data.PauseTime = event.Time
            case EventFinish:
                state = StateFinished
                data.EndTime = event.Time
            default:
                state = StateError
                data.Error = fmt.Errorf("unexpected event %v in Processing state", event.Type)
            }

        case StateWaiting:
            if event.Type == EventResume {
                state = StateProcessing
                data.WaitDuration += event.Time.Sub(data.PauseTime)
            } else if event.Type == EventFinish {
                state = StateFinished
                data.EndTime = event.Time
                data.WaitDuration += event.Time.Sub(data.PauseTime)
            } else {
                state = StateError
                data.Error = fmt.Errorf("unexpected event %v in Waiting state", event.Type)
            }

        case StateFinished, StateError:
            // Terminal states - ignore further events
            continue
        }

        if state == StateError {
            break
        }
    }

    return data
}
```

## **5.5 Practical Examples**

### **5.5.1 Input Validation**

```go
func validateUserInput(username, email, password string) (bool, string) {
    if len(username) < 3 {
        return false, "Username must be at least 3 characters long"
    }

    if !strings.Contains(email, "@") || !strings.Contains(email, ".") {
        return false, "Invalid email format"
    }

    if len(password) < 8 {
        return false, "Password must be at least 8 characters long"
    }

    hasUpper := false
    hasDigit := false

    for _, char := range password {
        if unicode.IsUpper(char) {
            hasUpper = true
        } else if unicode.IsDigit(char) {
            hasDigit = true
        }

        if hasUpper && hasDigit {
            break
        }
    }

    if !hasUpper {
        return false, "Password must contain at least one uppercase letter"
    }

    if !hasDigit {
        return false, "Password must contain at least one digit"
    }

    return true, ""
}
```

### **5.5.2 Processing a CSV File**

```go
func processCsvFile(filename string) ([]Record, error) {
    file, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer file.Close()

    reader := csv.NewReader(file)

    // Read header
    header, err := reader.Read()
    if err != nil {
        return nil, fmt.Errorf("reading header: %w", err)
    }

    var records []Record
    lineNum := 1 // Account for header

    for {
        lineNum++
        row, err := reader.Read()

        if err == io.EOF {
            break
        }

        if err != nil {
            return nil, fmt.Errorf("reading line %d: %w", lineNum, err)
        }

        // Ensure correct number of fields
        if len(row) != len(header) {
            return nil, fmt.Errorf("line %d: expected %d fields, got %d",
                lineNum, len(header), len(row))
        }

        record, err := parseRecord(header, row)
        if err != nil {
            return nil, fmt.Errorf("parsing line %d: %w", lineNum, err)
        }

        records = append(records, record)
    }

    return records, nil
}
```

### **5.5.3 Rate Limiting**

```go
func rateLimitedProcess(items []Item, ratePerSecond int) {
    interval := time.Second / time.Duration(ratePerSecond)
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for _, item := range items {
        <-ticker.C // Wait for the next tick
        processItem(item)
    }
}
```

## **5.6 Practice Exercises**

### **Exercise 1: FizzBuzz**

Write the classic FizzBuzz program:

- Print numbers from 1 to 100
- For multiples of 3, print "Fizz" instead of the number
- For multiples of 5, print "Buzz" instead of the number
- For multiples of both 3 and 5, print "FizzBuzz"

### **Exercise 2: Prime Number Checker**

Write a function that checks if a number is prime.

### **Exercise 3: Fibonacci Sequence**

Generate the first n numbers in the Fibonacci sequence using a loop.

### **Exercise 4: Word Counter**

Write a program that counts the occurrences of each word in a string.

### **Exercise 5: Tree Traversal**

Implement both depth-first and breadth-first traversal of a simple tree structure.

## **5.7 Summary**

In this chapter, you've learned:

- How to control the flow of your Go programs using conditional statements, loops, and switches
- Go's unique features like short variable declarations in `if` statements and the flexible `for` loop
- Common patterns and best practices for different types of control flow
- How to avoid common pitfalls and write more idiomatic Go code

These control structures form the backbone of any Go program. By mastering them, you can express complex algorithms clearly and efficiently.

**Next Up**: In Chapter 6, we'll explore Go's approach to functions, including multiple return values, variadic functions, and closures.
