# **Chapter 5: Defining and Calling Functions**

---

## **5.1 Function Syntax**

Functions in Go are declared using the `func` keyword. Here's a simple example:

```go
package main

import "fmt"

// Add function: Adds two integers
func Add(a int, b int) int {
    return a + b
}

func main() {
    result := Add(5, 3)
    fmt.Println("The sum is:", result) // Output: The sum is: 8
}
```

### Key Points:

- **Function Name**: `Add` is the function name.
- **Parameters**: `(a int, b int)` specifies two integers as input.
- **Return Type**: `int` indicates the function returns an integer.
- **Return Statement**: `return a + b` sends the result back to the caller.

---

### **5.1.1 Multiple Return Values**

Go functions can return multiple values. Let’s see an example:

```go
package main

import "fmt"

// Divide function: Returns quotient and remainder
func Divide(dividend, divisor int) (int, int) {
    quotient := dividend / divisor
    remainder := dividend % divisor
    return quotient, remainder
}

func main() {
    q, r := Divide(10, 3)
    fmt.Println("Quotient:", q)  // Output: Quotient: 3
    fmt.Println("Remainder:", r) // Output: Remainder: 1
}
```

---

## **5.2 Variadic Functions**

A **variadic function** accepts a variable number of arguments. Use the `...` syntax for the parameter type.

### Example 1: Sum of Numbers

```go
package main

import "fmt"

// Sum function: Accepts any number of integers
func Sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

func main() {
    fmt.Println("Sum is:", Sum(1, 2, 3, 4, 5)) // Output: Sum is: 15
}
```

### Example 2: Formatting a Message

```go
package main

import "fmt"

// Greet function: Formats a message with names
func Greet(message string, names ...string) {
    for _, name := range names {
        fmt.Println(message, name)
    }
}

func main() {
    Greet("Hello", "Alice", "Bob", "Charlie")
    // Output:
    // Hello Alice
    // Hello Bob
    // Hello Charlie
}
```

---

## **5.3 Defer, Panic, and Recover**

### **5.3.1 Defer**

The `defer` keyword schedules a function to run after the surrounding function completes, even if it exits early.

#### Example: Closing a File

```go
package main

import "fmt"

func main() {
    fmt.Println("Start")
    defer fmt.Println("End")
    fmt.Println("Middle")
}
// Output:
// Start
// Middle
// End
```

### **5.3.2 Panic**

`panic` stops the execution of a program and begins unwinding the stack. It’s used for unrecoverable errors.

#### Example: Triggering Panic

```go
package main

import "fmt"

func main() {
    fmt.Println("Before panic")
    panic("Something went wrong!")
    fmt.Println("This line will not execute")
}
// Output:
// Before panic
// panic: Something went wrong!
```

---

### **5.3.3 Recover**

`recover` is used within a `defer` block to handle a panic and resume execution.

#### Example: Handling Panic

```go
package main

import "fmt"

func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    fmt.Println("Starting program")
    panic("Unexpected error!")
    fmt.Println("This will not run")
}
// Output:
// Starting program
// Recovered from panic: Unexpected error!
```

---

## **5.4 Practical Exercises**

This section contains **solved examples** to help you master the concepts of functions, variadic functions, and error handling in Go. Each exercise is accompanied by code, explanations, and expected outputs.

---

## **Exercise 1: Basic Function**

**Problem**: Write a function `square` that takes an integer and returns its square.

```go
package main

import "fmt"

func square(n int) int {
    return n * n
}

func main() {
    fmt.Println("Square of 4:", square(4))
    fmt.Println("Square of 7:", square(7))
}
```

**Output:**

```
Square of 4: 16
Square of 7: 49
```

---

## **Exercise 2: Function with Multiple Parameters**

**Problem**: Write a function `add` that takes two integers and returns their sum.

```go
package main

import "fmt"

func add(a, b int) int {
    return a + b
}

func main() {
    fmt.Println("Sum of 3 and 5:", add(3, 5))
    fmt.Println("Sum of 10 and 20:", add(10, 20))
}
```

**Output:**

```
Sum of 3 and 5: 8
Sum of 10 and 20: 30
```

---

## **Exercise 3: Function with Multiple Return Values**

**Problem**: Write a function `divide` that returns both the quotient and remainder of two integers.

```go
package main

import "fmt"

func divide(a, b int) (int, int) {
    return a / b, a % b
}

func main() {
    q, r := divide(10, 3)
    fmt.Println("Quotient:", q, "Remainder:", r)
}
```

**Output:**

```
Quotient: 3 Remainder: 1
```

---

## **Exercise 4: Named Return Values**

**Problem**: Write a function `circleProperties` that returns both the area and circumference of a circle given its radius.

```go
package main

import (
    "fmt"
    "math"
)

func circleProperties(radius float64) (area, circumference float64) {
    area = math.Pi * radius * radius
    circumference = 2 * math.Pi * radius
    return
}

func main() {
    a, c := circleProperties(5)
    fmt.Println("Area:", a, "Circumference:", c)
}
```

**Output:**

```
Area: 78.53981633974483 Circumference: 31.41592653589793
```

---

## **Exercise 5: Variadic Function**

**Problem**: Write a function `product` that calculates the product of a variable number of integers.

```go
package main

import "fmt"

func product(nums ...int) int {
    result := 1
    for _, n := range nums {
        result *= n
    }
    return result
}

func main() {
    fmt.Println("Product:", product(1, 2, 3, 4))
    fmt.Println("Product:", product(5, 6, 7))
}
```

**Output:**

```
Product: 24
Product: 210
```

---

## **Exercise 6: Default Defer Behavior**

**Problem**: Demonstrate the use of `defer` to ensure a message is printed after the function returns.

```go
package main

import "fmt"

func main() {
    defer fmt.Println("Deferred message!")
    fmt.Println("Regular message")
}
```

**Output:**

```
Regular message
Deferred message!
```

---

## **Exercise 7: Multiple Defer Statements**

**Problem**: Show how multiple `defer` statements are executed in LIFO order.

```go
package main

import "fmt"

func main() {
    defer fmt.Println("First defer")
    defer fmt.Println("Second defer")
    fmt.Println("Main function")
}
```

**Output:**

```
Main function
Second defer
First defer
```

---

## **Exercise 8: Using Panic and Recover**

**Problem**: Write a program that handles division by zero using `panic` and `recover`.

```go
package main

import "fmt"

func safeDivision(a, b int) {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    fmt.Println(a / b)
}

func main() {
    safeDivision(10, 2)
    safeDivision(10, 0)
}
```

**Output:**

```
5
Recovered from panic: runtime error: integer divide by zero
```

---

## **Exercise 9: Defer with File Handling**

**Problem**: Use `defer` to ensure a file is closed after opening (mock example).

```go
package main

import "fmt"

func mockFileOperation() {
    defer fmt.Println("File closed")
    fmt.Println("File opened")
}

func main() {
    mockFileOperation()
}
```

**Output:**

```
File opened
File closed
```

---

## **Exercise 10: Recursive Function**

**Problem**: Write a recursive function `factorial` to calculate the factorial of a number.

```go
package main

import "fmt"

func factorial(n int) int {
    if n == 0 {
        return 1
    }
    return n * factorial(n-1)
}

func main() {
    fmt.Println("Factorial of 5:", factorial(5))
}
```

**Output:**

```
Factorial of 5: 120
```

---

## **Exercise 11: Callback Function**

**Problem**: Write a function `process` that accepts another function as an argument.

```go
package main

import "fmt"

func process(fn func(int) int, value int) int {
    return fn(value)
}

func square(n int) int {
    return n * n
}

func main() {
    fmt.Println("Square of 6:", process(square, 6))
}
```

**Output:**

```
Square of 6: 36
```

---

## **Exercise 12: Anonymous Function**

**Problem**: Use an anonymous function to calculate the cube of a number.

```go
package main

import "fmt"

func main() {
    cube := func(n int) int {
        return n * n * n
    }
    fmt.Println("Cube of 3:", cube(3))
}
```

**Output:**

```
Cube of 3: 27
```

---

**Congratulations!** You've completed the exercises for Chapter 5. These examples cover the breadth of function-related concepts in Go, equipping you to use them effectively in real-world applications.

---

## **5.5 Summary**

In this chapter, we explored:

1. The **syntax of functions**, including multiple return values.
2. How to use **variadic functions** for flexible arguments.
3. The role of **defer**, **panic**, and **recover** in managing errors and clean-up.

Functions are at the heart of Go programming, making your code cleaner and more reusable. Practice writing functions to solidify your understanding!

---

**Happy Coding!** 💻✨
