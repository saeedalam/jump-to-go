# **Chapter 5: Control Structures**

Control structures are the foundation of decision-making and flow control in programming. Go offers three primary types of control structures: **if statements**, **switch statements**, and **for loops**. This chapter will introduce these concepts step-by-step with clear explanations, practical examples, and exercises.

---

## **5.1 If Statements**

The `if` statement allows you to execute a block of code when a condition evaluates to `true`. It’s like asking a question: “Does this condition hold?” If yes, execute the code inside the block.

### **Syntax**

```go
if condition {
    // Code to execute if condition is true
} else {
    // Code to execute if condition is false (optional)
}
```

---

### **Step-by-Step Example: Checking Age**

#### **1. Start Simple: Check for Adults**

```go
package main

import "fmt"

func main() {
    age := 20

    if age >= 18 {
        fmt.Println("You are an adult.")
    }
}
```

**Explanation**:

- The `if` condition checks if `age` is greater than or equal to 18.
- If the condition evaluates to `true`, the message "You are an adult." is printed.

**Output:**

```
You are an adult.
```

---

#### **2. Add an `else` Block: Handle Minors**

```go
func main() {
    age := 16

    if age >= 18 {
        fmt.Println("You are an adult.")
    } else {
        fmt.Println("You are a minor.")
    }
}
```

**Explanation**:

- The `else` block is executed when the condition evaluates to `false`.
- In this example, since `age` is less than 18, the output is from the `else` block.

**Output:**

```
You are a minor.
```

---

#### **3. Use Short Variable Declaration**

Go allows you to declare variables directly in the `if` statement. This is useful for temporary variables.

```go
func main() {
    if age := 21; age >= 18 {
        fmt.Println("You are an adult.")
    } else {
        fmt.Println("You are a minor.")
    }
}
```

**Explanation**:

- The variable `age` is declared and initialized inside the `if` condition.
- This keeps the code clean and concise.

**Output:**

```
You are an adult.
```

---

## **5.2 Switch Statements**

The `switch` statement is a concise way to evaluate multiple conditions. It eliminates the need for multiple `if-else` blocks.

### **Syntax**

```go
switch expression {
case value1:
    // Code for case 1
case value2:
    // Code for case 2
default:
    // Code if no cases match (optional)
}
```

---

### **Step-by-Step Example: Days of the Week**

#### **1. Basic Switch**

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

**Explanation**:

- The `switch` evaluates the value of `day`.
- The matching `case` block executes, and the remaining cases are ignored.
- If no cases match, the `default` block executes.

**Output:**

```
Start of the workweek.
```

---

#### **2. Switch Without a Condition**

Go allows `switch` statements without an expression, which acts as a series of `if-else` conditions.

```go
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

**Explanation**:

- Each `case` is evaluated independently.
- This makes it flexible for range-based conditions.

**Output:**

```
Number is between 10 and 20.
```

---

## **5.3 Loops**

Loops allow you to repeat a block of code. Go uses the `for` loop as its only looping construct, but it can emulate `while` and `do-while` loops.

### **Syntax**

```go
for initialization; condition; post {
    // Code to execute
}
```

---

### **Step-by-Step Example: Iteration**

#### **1. Basic For Loop**

```go
package main

import "fmt"

func main() {
    for i := 1; i <= 5; i++ {
        fmt.Println("Iteration:", i)
    }
}
```

**Explanation**:

- The loop initializes `i` to 1, checks if `i <= 5`, and increments `i` after each iteration.

**Output:**

```
Iteration: 1
Iteration: 2
Iteration: 3
Iteration: 4
Iteration: 5
```

---

#### **2. Using `range`**

The `range` keyword simplifies iterating over arrays, slices, or maps.

```go
func main() {
    fruits := []string{"Apple", "Banana", "Cherry"}

    for index, fruit := range fruits {
        fmt.Printf("Index %d: %s\n", index, fruit)
    }
}
```

**Explanation**:

- `range` provides both the index and value of each element.
- The loop runs for each element in the slice.

**Output:**

```
Index 0: Apple
Index 1: Banana
Index 2: Cherry
```

---

#### **3. Infinite Loop**

An infinite loop runs indefinitely until explicitly broken with a `break` statement.

```go
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

**Explanation**:

- The program checks if the remainder of `number` divided by 2 is zero.
- If the remainder is zero, the number is even; otherwise, it is odd.

**Output**:

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

**Explanation**:

- The program uses a series of `if-else` conditions to compare the `age` variable.
- Depending on the age range, it prints the corresponding category.

**Output**:

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

**Explanation**:

- The `switch` statement checks if the `day` is either "Saturday" or "Sunday".
- If it matches, it prints that the day is a weekend; otherwise, it prints weekday.

**Output**:

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

**Explanation**:

- The `for` loop runs from 1 to `N`.
- In each iteration, the current value of `i` is added to the `sum` variable.

**Output**:

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

**Explanation**:

- The `for` loop iterates over each character in the string using `range`.
- Each character is prepended to the `reversed` string, effectively reversing it.

**Output**:

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

**Explanation**:

- The `for` loop iterates from 1 to 10.
- In each iteration, the product of `number` and the current value of `i` is printed.

**Output**:

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

**Next Steps**:

- Combine these constructs to solve more complex problems.
- Experiment by modifying the examples to deepen your understanding.
