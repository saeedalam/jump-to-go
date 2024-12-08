# **Chapter 3: Operators and Expressions**

In this chapter, we will explore how to perform operations and build expressions in Go using arithmetic, comparison, logical, and assignment operators. You'll also learn about Go's type inference and how it simplifies variable declarations.

---

## **3.1 Arithmetic Operators**

Arithmetic operators in Go are used for basic math operations such as addition, subtraction, multiplication, division, and modulus.

| Operator | Description         | Example |
| -------- | ------------------- | ------- |
| `+`      | Addition            | `a + b` |
| `-`      | Subtraction         | `a - b` |
| `*`      | Multiplication      | `a * b` |
| `/`      | Division            | `a / b` |
| `%`      | Modulus (remainder) | `a % b` |

### **Example 1: Arithmetic Operations**

```go
package main

import "fmt"

func main() {
    a := 10
    b := 3

    fmt.Println("Addition:", a+b)
    fmt.Println("Subtraction:", a-b)
    fmt.Println("Multiplication:", a*b)
    fmt.Println("Division:", a/b) // Integer division
    fmt.Println("Modulus:", a%b)
}
```

**Tip**: Division between integers returns an integer result. Use type conversion if you want floating-point results.

---

## **3.2 Comparison and Logical Operators**

### **Comparison Operators**

Comparison operators compare two values and return a boolean (`true` or `false`).

| Operator | Description           | Example  |
| -------- | --------------------- | -------- |
| `==`     | Equal to              | `a == b` |
| `!=`     | Not equal to          | `a != b` |
| `>`      | Greater than          | `a > b`  |
| `<`      | Less than             | `a < b`  |
| `>=`     | Greater than or equal | `a >= b` |
| `<=`     | Less than or equal    | `a <= b` |

### **Logical Operators**

Logical operators combine multiple boolean expressions.

| Operator | Description | Example           |
| -------- | ----------- | ----------------- | ---------- | ------ | --- | ------- |
| `&&`     | Logical AND | `a > 5 && b < 10` |
| `        |             | `                 | Logical OR | `a > 5 |     | b > 10` |
| `!`      | Logical NOT | `!(a > b)`        |

### **Example 2: Comparison and Logical Operations**

```go
package main

import "fmt"

func main() {
    a := 10
    b := 20

    // Comparison operators
    fmt.Println("a == b:", a == b)
    fmt.Println("a != b:", a != b)
    fmt.Println("a > b:", a > b)
    fmt.Println("a < b:", a < b)

    // Logical operators
    fmt.Println("a > 5 && b < 30:", a > 5 && b < 30)
    fmt.Println("a > 15 || b > 15:", a > 15 || b > 15)
    fmt.Println("!(a == b):", !(a == b))
}
```

---

## **3.3 Assignment Operators**

Assignment operators are used to update the value of a variable. In Go, you can combine arithmetic and assignment.

| Operator | Description         | Example  | Equivalent to |
| -------- | ------------------- | -------- | ------------- |
| `=`      | Assign              | `a = 10` |               |
| `+=`     | Add and assign      | `a += 5` | `a = a + 5`   |
| `-=`     | Subtract and assign | `a -= 3` | `a = a - 3`   |
| `*=`     | Multiply and assign | `a *= 2` | `a = a * 2`   |
| `/=`     | Divide and assign   | `a /= 2` | `a = a / 2`   |
| `%=`     | Modulus and assign  | `a %= 3` | `a = a % 3`   |

### **Example 3: Assignment Operations**

```go
package main

import "fmt"

func main() {
    a := 10

    fmt.Println("Initial value:", a)

    a += 5
    fmt.Println("After a += 5:", a)

    a -= 3
    fmt.Println("After a -= 3:", a)

    a *= 2
    fmt.Println("After a *= 2:", a)

    a /= 4
    fmt.Println("After a /= 4:", a)

    a %= 3
    fmt.Println("After a %= 3:", a)
}
```

---

## **3.4 Type Inference**

Go automatically determines the type of a variable when you use short declaration (`:=`).

### **Rules of Type Inference**

1. The type is inferred based on the assigned value.
2. Mixed-type operations are not allowed (e.g., `int` + `float64`).

### **Example 4: Type Inference**

```go
package main

import "fmt"

func main() {
    a := 42           // Inferred as int
    b := 3.14         // Inferred as float64
    c := "Hello, Go!" // Inferred as string

    fmt.Printf("Type of a: %T\n", a)
    fmt.Printf("Type of b: %T\n", b)
    fmt.Printf("Type of c: %T\n", c)
}
```

### **Pitfalls of Type Inference**

- Mixed-type operations require explicit conversion:

  ```go
  package main

  import "fmt"

  func main() {
      a := 42
      b := 3.14

      // fmt.Println(a + b) // Error: mismatched types
      fmt.Println(float64(a) + b) // Correct: converts a to float64
  }
  ```

---

## **3.5 Challenge: Build Expressions**

1. Write a program that:
   - Declares three variables (`x`, `y`, and `z`).
   - Performs arithmetic, comparison, and logical operations on these variables.
2. Display the results of all operations in a table format using `fmt.Printf`.

**Example Table Format for Output:**

```plaintext
Operation           Result
----------------------------
x + y               25
x > z && y < z      true
...
```

---

## **Summary**

- Use arithmetic operators for basic math operations.
- Apply comparison and logical operators to evaluate expressions.
- Simplify code with compound assignment operators like `+=` and `*=`.
- Leverage Go’s type inference for concise and clean code.
- Always handle mixed-type operations carefully with explicit type conversion.

Next, we’ll explore **control structures** to build flow in your programs!
