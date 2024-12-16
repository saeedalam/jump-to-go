
# **Chapter 14: Common Packages**

---

## **14.1 The `fmt` Package**

The `fmt` package in Go is used for formatted I/O operations like printing to the console or reading input. It offers various formatting verbs that help you print strings, integers, floats, and other values in a structured and customizable way.

### **Concept Explanation:**
- **Printing Output**: The `fmt.Println` and `fmt.Print` functions are used to print text. `fmt.Println` adds a new line after the output, while `fmt.Print` does not.
- **Formatted Output**: The `fmt.Printf` function allows formatted output using placeholders (verbs) like `%s`, `%d`, `%f`, etc., to format strings, integers, floats, etc.

### **Step-by-Step Examples:**

#### **Example 1: Basic Printing**

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go!")
    fmt.Print("This is ")
    fmt.Print("printed on the same line.")
}
```

**Explanation:**
- `fmt.Println` is used to print "Hello, Go!" with a newline after it.
- `fmt.Print` prints without a newline, so the next `fmt.Print` continues on the same line.

**Output:**
```
Hello, Go!
This is printed on the same line.
```

---

#### **Example 2: Formatting Strings and Variables**

```go
package main

import "fmt"

func main() {
    name := "Alice"
    age := 25
    fmt.Printf("Name: %s, Age: %d
", name, age)
}
```

**Explanation:**
- `%s` is a placeholder for a string.
- `%d` is a placeholder for an integer (decimal).

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

#### **Example 3: Aligning Output**

```go
package main

import "fmt"

func main() {
    fmt.Printf("%-10s %5d
", "Apple", 5)
    fmt.Printf("%-10s %5d
", "Banana", 12)
    fmt.Printf("%-10s %5d
", "Cherry", 50)
}
```

**Explanation:**
- The `%-10s` formats the string to be left-aligned with a width of 10 characters.
- `%5d` formats the integer to have a width of 5 characters, right-aligned.

**Output:**
```
Apple          5
Banana        12
Cherry        50
```

---

## **14.2 The `time` Package**

The `time` package in Go is used to work with dates, times, and durations. It provides functions to get the current time, format it, measure durations, and perform time-related calculations.

### **Concept Explanation:**
- **Getting the Current Time**: `time.Now()` returns the current local time.
- **Formatting Time**: The `Format` method allows you to format time in a custom way using a predefined reference time: `2006-01-02 15:04:05`.
- **Measuring Durations**: The `time.Since()` function returns the duration since a specific time, useful for measuring elapsed time.

### **Step-by-Step Examples:**

#### **Example 4: Getting the Current Time**

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

**Explanation:**
- `time.Now()` retrieves the current time and prints it.

**Output:**
```
Current time: 2024-12-06 10:30:45.123456 +0000 UTC
```

---

#### **Example 5: Formatting Time**

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

**Explanation:**
- `now.Format("2006-01-02 15:04:05")` formats the current time into a more readable string format.

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

#### **Example 6: Measuring Durations**

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

**Explanation:**
- `time.Since(start)` calculates the time elapsed since the `start` time.

**Output:**
```
Duration: 2.000123456s
```

---

## **14.3 The `math/rand` Package**

The `math/rand` package is used to generate pseudo-random numbers, which are great for simulations, games, and any use case where randomness is needed. It is not suitable for cryptographic purposes.

### **Concept Explanation:**
- **Random Integer**: `rand.Intn(n)` generates a random integer between 0 and `n-1`.
- **Random Float**: `rand.Float64()` generates a random float between 0 and 1.
- **Seeding Random Numbers**: `rand.Seed()` is used to initialize the random number generator. If not set, it uses the default seed, which could lead to the same sequence of random numbers on every program run.

### **Step-by-Step Examples:**

#### **Example 8: Generating Random Integers**

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

**Explanation:**
- `rand.Intn(100)` generates a random integer between 0 and 99.
- `rand.Seed(42)` initializes the random number generator with a fixed seed to ensure reproducibility.

**Output:**
```
Random integer: 87
```

---

#### **Example 9: Random Floats**

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

**Explanation:**
- `rand.Float64()` generates a random float between 0 and 1.

**Output:**
```
Random float: 0.8675309
```

---

#### **Example 12: Weighted Random Selection**

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

**Explanation:**
- This code selects a random option based on predefined weights. It generates a random number within the total weight, and the options are chosen based on ranges.

**Output:**
```
Selected: Option A
```

---

## **Summary**

- **`fmt`**: For formatted I/O operations such as printing to the console or reading input.
- **`time`**: For working with dates, times, and measuring durations.
- **`math/rand`**: For generating random numbers and shuffling.

---

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
