
# **Chapter 2: Variables, Constants, and Data Types**

---

## **2.1 Declaring Variables**

Go offers two primary ways to declare variables: using the `var` keyword or short declarations (`:=`).

### **Example 1: Declaring Variables**
```go
package main

import "fmt"

func main() {
    // Using var keyword
    var name string = "John"
    var age int = 30

    // Using short declaration
    isEmployed := true

    fmt.Println("Name:", name)
    fmt.Println("Age:", age)
    fmt.Println("Employed:", isEmployed)
}
```

### **Zero Values in Go**
When a variable is declared but not initialized, Go assigns it a **zero value**:
- `0` for numeric types
- `false` for booleans
- `""` (empty string) for strings

### **Example 2: Zero Values**
```go
package main

import "fmt"

func main() {
    var number int
    var price float64
    var active bool
    var message string

    fmt.Println("Default integer value:", number)
    fmt.Println("Default float value:", price)
    fmt.Println("Default boolean value:", active)
    fmt.Println("Default string value:", message)
}
```

---

## **2.2 Constants**

Constants are immutable values defined using the `const` keyword. They can be typed or untyped.

### **Example 3: Constants in Action**
```go
package main

import "fmt"

func main() {
    // Typed constants
    const pi float64 = 3.14159
    const greeting string = "Hello, Go!"

    // Untyped constants
    const daysInWeek = 7

    fmt.Println("Pi:", pi)
    fmt.Println("Greeting:", greeting)
    fmt.Println("Days in a week:", daysInWeek)
}
```

### **Table: Typed vs. Untyped Constants**

| Feature         | Typed Constants       | Untyped Constants       |
|------------------|-----------------------|-------------------------|
| **Definition**   | Specifies a type.     | Inferred type based on usage. |
| **Flexibility**  | Less flexible.        | More flexible in expressions. |

---

## **2.3 Basic Data Types**

Go supports several basic data types, including:
- **Integers**: `int`, `int8`, `int16`, `int32`, `int64`
- **Floating-point**: `float32`, `float64`
- **Strings**: Text data
- **Booleans**: `true` or `false`

### **Example 4: Working with Basic Data Types**
```go
package main

import "fmt"

func main() {
    var age int = 25
    var height float64 = 5.9
    var isStudent bool = true
    var name string = "Alice"

    fmt.Println("Name:", name)
    fmt.Println("Age:", age)
    fmt.Println("Height:", height)
    fmt.Println("Is Student:", isStudent)
}
```

### **Table: Common Data Types in Go**

| Data Type  | Description                     | Example       |
|------------|---------------------------------|---------------|
| `int`      | Integer type                   | `42`          |
| `float64`  | Floating-point number           | `3.14`        |
| `string`   | Text or sequence of characters | `"Go Lang"`   |
| `bool`     | Boolean value                  | `true`/`false`|

---

## **2.4 Type Conversion**

Go requires **explicit type conversion**. This prevents unintended behavior but requires developers to convert types manually.

### **Example 5: Type Conversion**
```go
package main

import "fmt"

func main() {
    var a int = 42
    var b float64 = float64(a) // Convert int to float64
    var c string = fmt.Sprintf("%d", a) // Convert int to string

    fmt.Println("Integer:", a)
    fmt.Println("Converted to float64:", b)
    fmt.Println("Converted to string:", c)
}
```

### **Common Pitfalls**
1. **Precision Loss**: Converting `float64` to `int` truncates the decimal part.
2. **Invalid Conversion**: Cannot directly convert unrelated types (e.g., `int` to `string`).

---

## **Summary**
By now, you should be able to:
- Declare and initialize variables using `var` and `:=`.
- Understand Go’s default zero values.
- Use `const` for immutable values and differentiate between typed and untyped constants.
- Work with basic data types like integers, floats, strings, and booleans.
- Perform explicit type conversions and avoid common pitfalls.

**Challenge**: Create a program that calculates the area of a circle using a constant for `pi` and type conversion for user input.

**Next Up**: Dive into operators and expressions in the next chapter!
