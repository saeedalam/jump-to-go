
# **Chapter 4: Control Structures**

Control structures in Go allow you to make decisions, loop through data, and manage the flow of your programs effectively. This chapter introduces **if statements**, **switch statements**, **loops**, and explains Go's **type inference rules**.

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

## **Summary**

By now, you should be able to:
1. Use `if` statements for conditional logic.
2. Implement `switch` statements for handling multiple conditions.
3. Work with loops, including `for`, `range`, and infinite loops.
4. Leverage Go's type inference to simplify your code.

**Challenge:** Write a program that checks whether a number is:
- Positive or negative.
- Even or odd.
- A multiple of 5.

Use a combination of `if`, `switch`, and loops to achieve this.

**Next Up:** Dive deeper into functions and error handling in the next chapter!
