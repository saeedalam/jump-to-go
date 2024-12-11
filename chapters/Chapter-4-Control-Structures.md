# **Chapter 4: Control Structures**

---

## **4.1 If Statements**

The `if` statement is used to evaluate conditions and execute code blocks accordingly.

### **Example 1: Basic If Statement**

```go
package main

import "fmt"

func main() {
    age := 18

    if age >= 18 {
        fmt.Println("You are an adult.")
    } else {
        fmt.Println("You are a minor.")
    }
}
```

**Output:**

```
You are an adult.
```

### **Example 2: If with Short Variable Declaration**

Go allows you to declare variables directly in the condition.

```go
package main

import "fmt"

func main() {
    if number := 42; number%2 == 0 {
        fmt.Println(number, "is even.")
    } else {
        fmt.Println(number, "is odd.")
    }
}
```

**Output:**

```
42 is even.
```

---

## **4.2 Switch Statements**

`Switch` statements are a concise way to handle multiple conditions.

### **Example 3: Basic Switch Statement**

```go
package main

import "fmt"

func main() {
    day := "Monday"

    switch day {
    case "Monday":
        fmt.Println("Start of the workweek.")
    case "Friday":
        fmt.Println("Almost the weekend!")
    default:
        fmt.Println("It's just another day.")
    }
}
```

**Output:**

```
Start of the workweek.
```

### **Example 4: Switch Without a Condition**

Go allows `switch` statements without a condition, acting like a series of `if-else` blocks.

```go
package main

import "fmt"

func main() {
    number := 15

    switch {
    case number < 10:
        fmt.Println("Number is less than 10.")
    case number >= 10 && number <= 20:
        fmt.Println("Number is between 10 and 20.")
    default:
        fmt.Println("Number is greater than 20.")
    }
}
```

**Output:**

```
Number is between 10 and 20.
```

---

## **4.3 Loops**

Go uses `for` as its only loop construct. It can be adapted to different looping patterns.

### **Example 5: Basic For Loop**

```go
package main

import "fmt"

func main() {
    for i := 1; i <= 5; i++ {
        fmt.Println("Iteration:", i)
    }
}
```

**Output:**

```
Iteration: 1
Iteration: 2
Iteration: 3
Iteration: 4
Iteration: 5
```

### **Example 6: Using Range in a Loop**

The `range` keyword is used to iterate over arrays, slices, or maps.

```go
package main

import "fmt"

func main() {
    fruits := []string{"Apple", "Banana", "Cherry"}

    for index, fruit := range fruits {
        fmt.Printf("Index %d: %s\n", index, fruit)
    }
}
```

**Output:**

```
Index 0: Apple
Index 1: Banana
Index 2: Cherry
```

### **Example 7: Infinite Loop**

Create an infinite loop using `for` and break it with a condition.

```go
package main

import "fmt"

func main() {
    count := 0
    for {
        if count == 3 {
            break
        }
        fmt.Println("Count:", count)
        count++
    }
}
```

**Output:**

```
Count: 0
Count: 1
Count: 2
```

---

## **4.4 Type Inference**

Go’s type inference lets you declare variables without explicitly specifying their types. The compiler infers the type based on the value assigned.

### **Example 8: Type Inference in Action**

```go
package main

import "fmt"

func main() {
    number := 42    // Inferred as int
    pi := 3.14      // Inferred as float64
    message := "Go" // Inferred as string

    fmt.Printf("number: %d (type: %T)\n", number, number)
    fmt.Printf("pi: %f (type: %T)\n", pi, pi)
    fmt.Printf("message: %s (type: %T)\n", message, message)
}
```

**Output:**

```
number: 42 (type: int)
pi: 3.140000 (type: float64)
message: Go (type: string)
```

---

# **Chapter 4.5 Exercises**

## **Exercise 1: Check if a Number is Even or Odd**

**Problem**: Write a program that takes a number and checks whether it is even or odd using an `if-else` statement.

```go
package main

import "fmt"

func main() {
    number := 17

    if number%2 == 0 {
        fmt.Println(number, "is even.")
    } else {
        fmt.Println(number, "is odd.")
    }
}
```

**Output:**

```
17 is odd.
```

---

## **Exercise 2: Categorize Age**

**Problem**: Write a program that categorizes a person's age group using `if-else` statements:

- Child (0–12)
- Teenager (13–19)
- Adult (20+)

```go
package main

import "fmt"

func main() {
    age := 25

    if age <= 12 {
        fmt.Println("Child")
    } else if age <= 19 {
        fmt.Println("Teenager")
    } else {
        fmt.Println("Adult")
    }
}
```

**Output:**

```
Adult
```

---

## **Exercise 3: Determine Day Type**

**Problem**: Write a program that determines whether a given day is a weekday or weekend using a `switch` statement.

```go
package main

import "fmt"

func main() {
    day := "Saturday"

    switch day {
    case "Saturday", "Sunday":
        fmt.Println(day, "is a weekend.")
    default:
        fmt.Println(day, "is a weekday.")
    }
}
```

**Output:**

```
Saturday is a weekend.
```

---

## **Exercise 4: Sum of First N Numbers**

**Problem**: Write a program that calculates the sum of the first `N` natural numbers using a `for` loop.

```go
package main

import "fmt"

func main() {
    N := 5
    sum := 0

    for i := 1; i <= N; i++ {
        sum += i
    }

    fmt.Println("Sum of first", N, "numbers:", sum)
}
```

**Output:**

```
Sum of first 5 numbers: 15
```

---

## **Exercise 5: Reverse a String**

**Problem**: Write a program that reverses a string using a `for` loop and the `range` keyword.

```go
package main

import "fmt"

func main() {
    str := "GoLang"
    reversed := ""

    for _, char := range str {
        reversed = string(char) + reversed
    }

    fmt.Println("Reversed string:", reversed)
}
```

**Output:**

```
Reversed string: gnaLoG
```

---

## **Exercise 6: Multiplication Table**

**Problem**: Write a program that prints the multiplication table of a given number up to 10 using a `for` loop.

```go
package main

import "fmt"

func main() {
    number := 7

    fmt.Println("Multiplication Table for", number)
    for i := 1; i <= 10; i++ {
        fmt.Printf("%d x %d = %d
", number, i, number*i)
    }
}
```

**Output:**

```
Multiplication Table for 7
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
7 x 4 = 28
7 x 5 = 35
7 x 6 = 42
7 x 7 = 49
7 x 8 = 56
7 x 9 = 63
7 x 10 = 70
```

---

**Summary**

These exercises demonstrate the versatility of control structures in Go. By solving them, you’ve practiced:

- Conditional logic with `if-else` statements.
- Handling multiple conditions with `switch`.
- Iterative computations using `for` loops.

Experiment further by combining these constructs to solve more complex problems!
