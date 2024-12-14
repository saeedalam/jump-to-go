# **Chapter 4: Operators and Expressions**

---

## **4.1 Arithmetic Operators**

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

### **Output:**

```
Addition: 13
Subtraction: 7
Multiplication: 30
Division: 3
Modulus: 1
```

**Tip**: Division between integers returns an integer result. Use type conversion if you want floating-point results.

---

## **4.2 Comparison and Logical Operators**

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

### **Output:**

```

```


---

## **4.3 Assignment Operators**

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

## **4.4 Type Inference**

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

## **4.5 Challenge: Build Expressions**

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

# **4.6. Exercises**

## **Exercise 1: Simple Arithmetic Operations**

**Problem**: Write a program that performs addition, subtraction, multiplication, division, and modulus on two integers entered by the user.

```go
package main

import "fmt"

func main() {
    var a, b int
    fmt.Print("Enter two integers: ")
    fmt.Scan(&a, &b)

    fmt.Println("Addition:", a+b)
    fmt.Println("Subtraction:", a-b)
    fmt.Println("Multiplication:", a*b)
    fmt.Println("Division:", a/b)
    fmt.Println("Modulus:", a%b)
}
```

**Output (Example):**

```
Enter two integers: 10 3
Addition: 13
Subtraction: 7
Multiplication: 30
Division: 3
Modulus: 1
```

---

## **Exercise 2: Comparison and Logical Operators**

**Problem**: Write a program to compare two integers and evaluate logical expressions.

```go
package main

import "fmt"

func main() {
    var x, y int
    fmt.Print("Enter two integers: ")
    fmt.Scan(&x, &y)

    fmt.Println("x == y:", x == y)
    fmt.Println("x != y:", x != y)
    fmt.Println("x > y:", x > y)
    fmt.Println("x < y:", x < y)

    fmt.Println("x > 5 && y < 20:", x > 5 && y < 20)
    fmt.Println("x < 10 || y > 10:", x < 10 || y > 10)
    fmt.Println("!(x == y):", !(x == y))
}
```

**Output (Example):**

```
Enter two integers: 8 15
x == y: false
x != y: true
x > y: false
x < y: true
x > 5 && y < 20: true
x < 10 || y > 10: true
!(x == y): true
```

---

## **Exercise 3: Compound Assignment Operators**

**Problem**: Write a program that demonstrates the use of `+=`, `-=`, `*=`, `/=`, and `%=` operators.

```go
package main

import "fmt"

func main() {
    var num int
    fmt.Print("Enter an integer: ")
    fmt.Scan(&num)

    fmt.Println("Initial value:", num)
    num += 10
    fmt.Println("After += 10:", num)
    num -= 5
    fmt.Println("After -= 5:", num)
    num *= 2
    fmt.Println("After *= 2:", num)
    num /= 4
    fmt.Println("After /= 4:", num)
    num %= 3
    fmt.Println("After %= 3:", num)
}
```

**Output (Example):**

```
Enter an integer: 20
Initial value: 20
After += 10: 30
After -= 5: 25
After *= 2: 50
After /= 4: 12
After %= 3: 0
```

---

## **Exercise 4: Type Conversion in Arithmetic**

**Problem**: Write a program to calculate the sum and average of two numbers where one is an integer and the other is a floating-point number.

```go
package main

import "fmt"

func main() {
    var a int
    var b float64
    fmt.Print("Enter an integer and a float: ")
    fmt.Scan(&a, &b)

    sum := float64(a) + b
    average := sum / 2

    fmt.Printf("Sum: %.2f\n", sum)
    fmt.Printf("Average: %.2f\n", average)
}
```

**Output (Example):**

```
Enter an integer and a float: 5 4.5
Sum: 9.50
Average: 4.75
```

---

## **Exercise 5: Arithmetic and Logical Expressions**

**Problem**: Write a program to calculate and display the results of complex arithmetic and logical expressions.

```go
package main

import "fmt"

func main() {
    x, y, z := 10, 20, 15

    fmt.Println("x + y * z:", x+y*z)
    fmt.Println("(x + y) * z:", (x+y)*z)
    fmt.Println("x > y && y < z:", x > y && y < z)
    fmt.Println("x < z || y > z:", x < z || y > z)
}
```

**Output:**

```
x + y * z: 310
(x + y) * z: 450
x > y && y < z: false
x < z || y > z: true
```

---

## **Exercise 6: Building a Simple Calculator**

**Problem**: Write a program that acts as a simple calculator, allowing the user to choose an operation (add, subtract, multiply, divide) to perform on two numbers.

```go
package main

import "fmt"

func main() {
    var a, b, choice int
    fmt.Println("Simple Calculator")
    fmt.Print("Enter two integers: ")
    fmt.Scan(&a, &b)

    fmt.Println("Choose an operation:")
    fmt.Println("1. Addition")
    fmt.Println("2. Subtraction")
    fmt.Println("3. Multiplication")
    fmt.Println("4. Division")
    fmt.Print("Enter your choice: ")
    fmt.Scan(&choice)

    switch choice {
    case 1:
        fmt.Println("Result:", a+b)
    case 2:
        fmt.Println("Result:", a-b)
    case 3:
        fmt.Println("Result:", a*b)
    case 4:
        if b != 0 {
            fmt.Println("Result:", a/b)
        } else {
            fmt.Println("Error: Division by zero!")
        }
    default:
        fmt.Println("Invalid choice!")
    }
}
```

**Output (Example):**

```
Simple Calculator
Enter two integers: 12 4
Choose an operation:
1. Addition
2. Subtraction
3. Multiplication
4. Division
Enter your choice: 3
Result: 48
```

---

**Congratulations!** You’ve completed the exercises for Chapter 3. These examples cover arithmetic, comparison, logical operations, type conversion, and more, helping you apply these concepts in practical scenarios.
