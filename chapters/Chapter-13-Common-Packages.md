# **Chapter 13: Common Packages**

---

## **13.1 The `fmt` Package**

The `fmt` package in Go is used for formatted I/O operations like printing to the console or reading input. Let’s explore its capabilities.

### **Example 1: Basic Printing**

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go!")
    fmt.Print("This is ")
    fmt.Print("printed on the same line.")
}
```

**Output:**

```
Hello, Go!
This is printed on the same line.
```

---

### **Example 2: Formatting Strings and Variables**

```go
package main

import "fmt"

func main() {
    name := "Alice"
    age := 25
    fmt.Printf("Name: %s, Age: %d\n", name, age)
}
```

**Output:**

```
Name: Alice, Age: 25
```

| **Verb** | **Description**        | **Example** |
| -------- | ---------------------- | ----------- |
| `%s`     | String                 | `"Hello"`   |
| `%d`     | Integer (decimal)      | `42`        |
| `%f`     | Float (default)        | `3.14159`   |
| `%v`     | Default representation | Any value   |

---

### **Example 3: Aligning Output**

```go
package main

import "fmt"

func main() {
    fmt.Printf("%-10s %5d\n", "Apple", 5)
    fmt.Printf("%-10s %5d\n", "Banana", 12)
    fmt.Printf("%-10s %5d\n", "Cherry", 50)
}
```

**Output:**

```
Apple          5
Banana        12
Cherry        50
```

---

## **13.2 The `time` Package**

The `time` package helps with working with dates and time in Go. It’s incredibly powerful for formatting, measuring durations, and scheduling.

### **Example 4: Getting the Current Time**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println("Current time:", now)
}
```

**Output:**

```
Current time: 2024-12-06 10:30:45.123456 +0000 UTC
```

---

### **Example 5: Formatting Time**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println("Formatted time:", now.Format("2006-01-02 15:04:05"))
}
```

**Output:**

```
Formatted time: 2024-12-06 10:30:45
```

| **Format Code** | **Description**      | **Example** |
| --------------- | -------------------- | ----------- |
| `2006`          | Year                 | `2024`      |
| `01`            | Month (zero-padded)  | `12`        |
| `02`            | Day (zero-padded)    | `06`        |
| `15:04:05`      | Time (24-hour clock) | `10:30:45`  |

---

### **Example 6: Measuring Durations**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    start := time.Now()
    time.Sleep(2 * time.Second)
    duration := time.Since(start)
    fmt.Println("Duration:", duration)
}
```

**Output:**

```
Duration: 2.000123456s
```

---

## **13.3 The `math/rand` Package**

The `math/rand` package generates pseudo-random numbers. While not suitable for cryptographic purposes, it’s great for general randomness.

### **Example 8: Generating Random Integers**

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    rand.Seed(42)
    fmt.Println("Random integer:", rand.Intn(100))
}
```

**Output:**

```
Random integer: 87
```

---

### **Example 9: Random Floats**

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    rand.Seed(42)
    fmt.Println("Random float:", rand.Float64())
}
```

**Output:**

```
Random float: 0.8675309
```

---

### **Example 12: Weighted Random Selection**

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    weights := []int{70, 20, 10} // 70%, 20%, 10%
    total := 0
    for _, w := range weights {
        total += w
    }
    r := rand.Intn(total)
    choice := ""
    switch {
    case r < weights[0]:
        choice = "Option A"
    case r < weights[0]+weights[1]:
        choice = "Option B"
    default:
        choice = "Option C"
    }
    fmt.Println("Selected:", choice)
}
```

**Output:**

```
Selected: Option A
```

---

## **Summary**

- **`fmt`**: For formatted I/O.
- **`time`**: For working with dates, times, and durations.
- **`math/rand`**: For generating random numbers and shuffling.

# **13.4: Exercises - Common Packages**

This section provides **10 solved exercises** to help you master the use of the `fmt`, `time`, and `math/rand` packages in Go. Each exercise includes code, explanations, and expected outputs.

---

## **Exercise 1: Table Formatter**

**Problem**: Write a program to format a table of student names and their grades using the `fmt` package.

```go
package main

import "fmt"

func main() {
    fmt.Printf("%-15s %10s\n", "Name", "Grade")
    fmt.Printf("%-15s %10d\n", "Alice", 95)
    fmt.Printf("%-15s %10d\n", "Bob", 88)
    fmt.Printf("%-15s %10d\n", "Charlie", 92)
}
```

**Output:**

```
Name               Grade
Alice                 95
Bob                   88
Charlie               92
```

---

## **Exercise 2: Time Greeting**

**Problem**: Create a program that greets the user based on the current time (morning, afternoon, evening).

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    hour := now.Hour()

    if hour < 12 {
        fmt.Println("Good morning!")
    } else if hour < 18 {
        fmt.Println("Good afternoon!")
    } else {
        fmt.Println("Good evening!")
    }
}
```

**Output (varies by time):**

```
Good morning!
```

---

## **Exercise 3: Stopwatch**

**Problem**: Simulate a stopwatch that measures the time taken to execute a task.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    start := time.Now()

    // Simulated task
    time.Sleep(3 * time.Second)

    duration := time.Since(start)
    fmt.Printf("Task completed in %v seconds.\n", duration.Seconds())
}
```

**Output:**

```
Task completed in 3.000123 seconds.
```

---

## **Exercise 4: Random Lottery**

**Problem**: Generate 5 unique random lottery numbers between 1 and 50.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    numbers := make(map[int]bool)
    for len(numbers) < 5 {
        n := rand.Intn(50) + 1
        if !numbers[n] {
            numbers[n] = true
            fmt.Println("Number:", n)
        }
    }
}
```

**Output (varies):**

```
Number: 12
Number: 34
Number: 45
Number: 22
Number: 5
```

---

## **Exercise 5: Birthday Formatter**

**Problem**: Format and display a person's birthday in `DD-MM-YYYY` format.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    birthday := time.Date(1995, time.March, 25, 0, 0, 0, 0, time.UTC)
    fmt.Println("Formatted birthday:", birthday.Format("02-01-2006"))
}
```

**Output:**

```
Formatted birthday: 25-03-1995
```

---

## **Exercise 6: Random Float Precision**

**Problem**: Generate 3 random floating-point numbers and format them to 2 decimal places.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    for i := 0; i < 3; i++ {
        fmt.Printf("Random float: %.2f\n", rand.Float64()*100)
    }
}
```

**Output (varies):**

```
Random float: 45.67
Random float: 89.12
Random float: 23.45
```

---

## **Exercise 7: Countdown Timer**

**Problem**: Create a countdown timer that displays each second until zero.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    for i := 10; i >= 0; i-- {
        fmt.Println(i)
        time.Sleep(1 * time.Second)
    }
    fmt.Println("Time's up!")
}
```

**Output:**

```
10
9
8
...
0
Time's up!
```

---

## **Exercise 8: Weighted Random Choice**

**Problem**: Simulate a weighted random choice from three options.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    rand.Seed(time.Now().UnixNano())
    weights := []int{50, 30, 20} // 50%, 30%, 20%
    total := 0
    for _, w := range weights {
        total += w
    }

    r := rand.Intn(total)
    choice := ""
    switch {
    case r < weights[0]:
        choice = "Option A"
    case r < weights[0]+weights[1]:
        choice = "Option B"
    default:
        choice = "Option C"
    }
    fmt.Println("Selected:", choice)
}
```

**Output (varies):**

```
Selected: Option A
```

---

## **Exercise 9: Generate Timestamps**

**Problem**: Print timestamps for the next 5 days.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    for i := 1; i <= 5; i++ {
        future := now.AddDate(0, 0, i)
        fmt.Println(future.Format("2006-01-02 15:04:05"))
    }
}
```

**Output:**

```
2024-12-07 10:30:45
2024-12-08 10:30:45
...
```

---

## **Exercise 10: Random Password Generator**

**Problem**: Generate a random 8-character alphanumeric password.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func generatePassword(length int) string {
    chars := "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    rand.Seed(time.Now().UnixNano())
    password := make([]byte, length)
    for i := range password {
        password[i] = chars[rand.Intn(len(chars))]
    }
    return string(password)
}

func main() {
    fmt.Println("Random password:", generatePassword(8))
}
```

**Output (varies):**

```
Random password: G7Xk92Ab
```

---

**Congratulations!** You've completed the exercises for Chapter 13. These examples are designed to provide practical experience with Go's `fmt`, `time`, and `math/rand` packages, helping you apply them effectively in real-world scenarios.
