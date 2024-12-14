# **Chapter 6: Defining and Calling Functions**

---

## **6.1 Introduction to Functions**

Functions are the building blocks of any Go program. They allow you to encapsulate logic, reuse code, and structure your program effectively. In this chapter, we will explore:

- Basic syntax for defining functions.
- Multiple return values.
- Variadic functions.
- The `defer`, `panic`, and `recover` mechanisms.

---

## **6.2 Function Syntax**

Functions in Go are declared using the `func` keyword. Here’s a simple example:

```go
package main

import "fmt"

// Add function: Adds two integers
func Add(a int, b int) int {
    return a + b
}

func main() {
    result := Add(5, 3)
    fmt.Println("The sum is:", result)
}
```

### **Explanation:**

- **Function Name**: `Add` is the name of the function.
- **Parameters**: `(a int, b int)` specifies two integer inputs.
- **Return Type**: `int` indicates the function returns an integer.
- **Return Statement**: `return a + b` sends the result back to the caller.

### **Output:**

```
The sum is: 8
```

---

## **6.3 Multiple Return Values**

Go functions can return multiple values, making it easier to handle related results.

### **Example: Quotient and Remainder**

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
    fmt.Println("Quotient:", q)
    fmt.Println("Remainder:", r)
}
```

### **Explanation:**

- The `Divide` function returns two integers: `quotient` and `remainder`.
- Multiple return values are captured using `q, r :=`.

### **Output:**

```
Quotient: 3
Remainder: 1
```

---

## **6.4 Variadic Functions**

A **variadic function** accepts a variable number of arguments. Use the `...` syntax to define one.

### **Example 1: Summing Numbers**

```go
package main

import "fmt"

// Sum function: Calculates the sum of numbers
func Sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

func main() {
    fmt.Println("Sum is:", Sum(1, 2, 3, 4, 5))
}
```

### **Explanation:**

- The `Sum` function takes a variable number of integers (`numbers ...int`).
- It iterates over the slice of numbers and calculates their total.

### **Output:**

```
Sum is: 15
```

---

### **Example 2: Greeting Multiple People**

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
}
```

### **Explanation:**

- The `Greet` function takes a string (`message`) and a variadic string slice (`names ...string`).
- It prints the message for each name.

### **Output:**

```
Hello Alice
Hello Bob
Hello Charlie
```

---

## **6.5 Defer, Panic, and Recover**

Go provides special mechanisms for handling cleanup, errors, and recovery.

### **6.5.1 Defer**

The `defer` keyword schedules a function to execute after the surrounding function completes.

### **Example: Deferred Execution**

```go
package main

import "fmt"

func main() {
    fmt.Println("Start")
    defer fmt.Println("End")
    fmt.Println("Middle")
}
```

### **Explanation:**

- `defer` ensures that `fmt.Println("End")` runs last, even though it’s declared in the middle.

### **Output:**

```
Start
Middle
End
```

---

### **6.5.2 Panic**

`panic` stops the normal execution of a program and begins unwinding the stack.

### **Example: Triggering Panic**

```go
package main

import "fmt"

func main() {
    fmt.Println("Before panic")
    panic("Something went wrong!")
    fmt.Println("This line will not execute")
}
```

### **Explanation:**

- The `panic` statement halts execution and outputs an error message.

### **Output:**

```
Before panic
panic: Something went wrong!
```

---

### **6.5.3 Recover**

`recover` handles a panic, allowing the program to continue executing.

### **Example: Recovering from Panic**

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
}
```

### **Explanation:**

- The deferred function captures and handles the panic using `recover`.

### **Output:**

```
Starting program
Recovered from panic: Unexpected error!
```

---

# **6.6. Exercises**

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

### **Explanation:**

- The `square` function multiplies the input integer `n` by itself.
- In the `main` function, the `square` function is called with two different values.

### **Output:**

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

### **Explanation:**

- The `add` function takes two integers, adds them, and returns the result.
- The `main` function demonstrates the usage with example inputs.

### **Output:**

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

### **Explanation:**

- The `divide` function performs integer division and modulus operations, returning both results.
- The `main` function captures and prints these results.

### **Output:**

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

### **Explanation:**

- The `circleProperties` function calculates and returns both the area and circumference using named return values.
- `math.Pi` is used for precision.

### **Output:**

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

### **Explanation:**

- The `product` function uses variadic arguments (`nums ...int`) to handle any number of inputs.
- A loop iterates through the numbers, multiplying them.

### **Output:**

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

### **Explanation:**

- The `defer` statement schedules the print statement for execution after the function completes.

### **Output:**

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

### **Explanation:**

- Defer statements are executed in **Last-In, First-Out (LIFO)** order when the function exits.

### **Output:**

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

### **Explanation:**

- The `defer` function with `recover` captures and handles the panic caused by division by zero.

### **Output:**

```
5
Recovered from panic: runtime error: integer divide by zero
```

---

## **Exercise 9: Recursive Function**

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

### **Explanation:**

- The `factorial` function calls itself until it reaches the base case (`n == 0`).

### **Output:**

```
Factorial of 5: 120
```

---

**Congratulations!** You've completed the exercises. Practice these examples to solidify your understanding of Go functions!
