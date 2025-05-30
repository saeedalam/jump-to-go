# **Chapter 4: Mastering Go Operators and Expressions**

In programming, operators are the tools that transform and combine values. They're the verbs of a programming language—the actions that make things happen. Go's operators are thoughtfully designed for clarity and predictability, avoiding many of the pitfalls found in other languages.

By the end of this chapter, you'll understand how to manipulate data effectively using Go's complete set of operators, recognize common pitfalls, and write clean, efficient expressions.

## **4.1 Arithmetic Operators: The Building Blocks of Computation**

Arithmetic operators perform mathematical operations on numeric values.

| Operator | Description         | Example | Notes                      |
| -------- | ------------------- | ------- | -------------------------- |
| `+`      | Addition            | `a + b` | Works with numeric types   |
| `-`      | Subtraction         | `a - b` | Works with numeric types   |
| `*`      | Multiplication      | `a * b` | Works with numeric types   |
| `/`      | Division            | `a / b` | Integer division truncates |
| `%`      | Modulus (remainder) | `a % b` | Only for integers          |

### **Integer vs. Floating-Point Division**

```go
package main

import "fmt"

func main() {
    // Integer division truncates the result
    fmt.Println("10 / 3 =", 10/3)           // Output: 3

    // For floating-point division, convert at least one operand
    fmt.Println("10 / 3.0 =", 10/3.0)       // Output: 3.3333333333333335
    fmt.Println("float64(10) / 3 =", float64(10)/3) // Output: 3.3333333333333335
}
```

### **The Modulus Operator: Beyond Remainders**

The modulus operator (`%`) is particularly useful for:

1. **Checking if a number is even or odd**:

   ```go
   isEven := number % 2 == 0
   ```

2. **Wrapping around a range** (e.g., circular buffers, clock arithmetic):

   ```go
   // Clock hours wrap from 12 back to 1
   nextHour := (currentHour % 12) + 1

   // Days of week (0-6, where 0 is Sunday)
   dayAfterTomorrow := (today + 2) % 7
   ```

3. **Limiting a value to a range**:
   ```go
   // Ensure value is between 0 and 359 (degrees in a circle)
   normalizedAngle := angle % 360
   ```

## **4.2 Comparison Operators: Making Decisions**

Comparison operators compare values and return boolean results (`true` or `false`).

| Operator | Description           | Example  | Result Type |
| -------- | --------------------- | -------- | ----------- |
| `==`     | Equal to              | `a == b` | `bool`      |
| `!=`     | Not equal to          | `a != b` | `bool`      |
| `>`      | Greater than          | `a > b`  | `bool`      |
| `<`      | Less than             | `a < b`  | `bool`      |
| `>=`     | Greater than or equal | `a >= b` | `bool`      |
| `<=`     | Less than or equal    | `a <= b` | `bool`      |

### **Comparing Different Types**

Go is strict about types in comparisons:

```go
package main

import "fmt"

func main() {
    var a int = 10
    var b int32 = 10

    // This won't compile:
    // fmt.Println(a == b) // Error: mismatched types

    // Correct approach - explicit conversion:
    fmt.Println(a == int(b))     // true
    fmt.Println(int32(a) == b)   // true
}
```

### **Comparing Floating-Point Numbers**

Due to floating-point precision issues, direct equality comparison can be problematic:

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    a := 0.1 + 0.2
    b := 0.3

    fmt.Println("a =", a)                // 0.30000000000000004
    fmt.Println("b =", b)                // 0.3
    fmt.Println("a == b:", a == b)       // false

    // Better approach - use a small epsilon value
    const epsilon = 1e-9
    fmt.Println("Math.Abs(a-b) < epsilon:", math.Abs(a-b) < epsilon) // true
}
```

## **4.3 Logical Operators: Combining Conditions**

Logical operators combine boolean expressions to form more complex conditions.

| Operator | Description | Example             | Notes                                           |
| -------- | ----------- | ------------------- | ----------------------------------------------- |
| `&&`     | Logical AND | `a > 5 && b < 10`   | Returns true if both expressions are true       |
| `\|\|`   | Logical OR  | `a > 5 \|\| b > 10` | Returns true if at least one expression is true |
| `!`      | Logical NOT | `!(a > b)`          | Inverts a boolean value                         |

### **Short-Circuit Evaluation**

Go uses short-circuit evaluation for logical operators, which can improve performance and enable useful programming patterns:

```go
package main

import "fmt"

func main() {
    // With &&, the second expression is only evaluated if the first is true
    x := 10
    if x > 5 && expensiveOperation(x) {
        fmt.Println("Condition met")
    }

    // With ||, the second expression is only evaluated if the first is false
    y := 3
    if y > 5 || fallbackOperation(y) {
        fmt.Println("At least one condition met")
    }
}

func expensiveOperation(n int) bool {
    fmt.Println("Performing expensive operation")
    return n%2 == 0
}

func fallbackOperation(n int) bool {
    fmt.Println("Performing fallback operation")
    return true
}
```

### **Common Logical Patterns**

```go
// Range check: Is x between min and max (inclusive)?
inRange := min <= x && x <= max

// Valid options: Is option either A, B, or C?
validOption := option == "A" || option == "B" || option == "C"

// Not in range: Is x outside the range?
outOfRange := x < min || x > max

// Exclusive OR (XOR): Is exactly one condition true?
exactlyOne := (a && !b) || (!a && b)
```

## **4.4 Assignment Operators: Updating Values**

Assignment operators modify variables in place, often combining an operation with assignment.

| Operator | Description         | Example  | Equivalent to |
| -------- | ------------------- | -------- | ------------- |
| `=`      | Assign              | `a = 10` |               |
| `+=`     | Add and assign      | `a += 5` | `a = a + 5`   |
| `-=`     | Subtract and assign | `a -= 3` | `a = a - 3`   |
| `*=`     | Multiply and assign | `a *= 2` | `a = a * 2`   |
| `/=`     | Divide and assign   | `a /= 2` | `a = a / 2`   |
| `%=`     | Modulus and assign  | `a %= 3` | `a = a % 3`   |

### **Compound Assignment with Different Types**

The same type restrictions apply to compound assignments:

```go
package main

import "fmt"

func main() {
    count := 10

    // This works - same type
    count += 5
    fmt.Println(count) // 15

    // This won't compile - mismatched types
    // count += 2.5 // Error

    // Correct approach with explicit conversion
    count += int(2.5) // count = count + int(2.5)
    fmt.Println(count) // 17
}
```

## **4.5 Bitwise Operators: Manipulating Individual Bits**

Bitwise operators work at the binary level, manipulating individual bits within integers.

| Operator | Description         | Example  | Result (in binary)        |
| -------- | ------------------- | -------- | ------------------------- |
| `&`      | Bitwise AND         | `a & b`  | 1 where both have 1       |
| `\|`     | Bitwise OR          | `a \| b` | 1 where either has 1      |
| `^`      | Bitwise XOR         | `a ^ b`  | 1 where bits differ       |
| `&^`     | Bit clear (AND NOT) | `a &^ b` | Clears bits where b has 1 |
| `<<`     | Left shift          | `a << n` | Shift left by n places    |
| `>>`     | Right shift         | `a >> n` | Shift right by n places   |

### **Practical Applications of Bitwise Operations**

```go
package main

import "fmt"

func main() {
    // Setting a bit
    var flags uint8 = 0
    const (
        isAdmin = 1 << iota      // 00000001
        hasWriteAccess           // 00000010
        hasReadAccess            // 00000100
    )

    // Grant read and write access
    flags |= hasWriteAccess | hasReadAccess
    fmt.Printf("Flags: %08b\n", flags) // 00000110

    // Check if a bit is set
    hasWrite := (flags & hasWriteAccess) != 0
    fmt.Println("Has write access:", hasWrite) // true

    // Clear a bit
    flags &^= hasWriteAccess // Clear write access
    fmt.Printf("Flags after clearing write: %08b\n", flags) // 00000100

    // Toggle a bit
    flags ^= hasReadAccess // Toggle read access (turn it off)
    fmt.Printf("Flags after toggling read: %08b\n", flags) // 00000000
}
```

### **Shift Operations for Powers of Two**

Shifting left by n is equivalent to multiplying by 2ⁿ:

```go
package main

import "fmt"

func main() {
    // Left shifts - multiply by powers of 2
    fmt.Println("1 << 0 =", 1<<0)  // 1 * 2⁰ = 1
    fmt.Println("1 << 3 =", 1<<3)  // 1 * 2³ = 8
    fmt.Println("5 << 2 =", 5<<2)  // 5 * 2² = 20

    // Right shifts - divide by powers of 2 (integer division)
    fmt.Println("8 >> 1 =", 8>>1)  // 8 / 2¹ = 4
    fmt.Println("12 >> 2 =", 12>>2) // 12 / 2² = 3
}
```

## **4.6 Increment and Decrement Operators**

Go provides simple operators to increase or decrease a value by 1.

| Operator | Description | Example | Equivalent to |
| -------- | ----------- | ------- | ------------- |
| `++`     | Increment   | `a++`   | `a = a + 1`   |
| `--`     | Decrement   | `a--`   | `a = a - 1`   |

### **Special Rules for Go's Increment/Decrement**

Unlike C, C++, and Java:

1. They are **statements**, not expressions, so you can't use them in assignments or expressions
2. Only the **postfix** form exists (`a++`, not `++a`)
3. They can only be applied to **variables**, not values

```go
package main

import "fmt"

func main() {
    count := 5

    // These work
    count++
    fmt.Println(count) // 6

    count--
    fmt.Println(count) // 5

    // These won't compile:
    // fmt.Println(count++) // Error: increment or decrement statement
    // x := count++ // Error: increment or decrement statement
    // y := 5++ // Error: can't increment literal value
}
```

## **4.7 String Operators**

Strings in Go have a few specific operators.

### **String Concatenation**

The `+` operator concatenates strings:

```go
package main

import "fmt"

func main() {
    firstName := "Rob"
    lastName := "Pike"

    fullName := firstName + " " + lastName
    fmt.Println(fullName) // Rob Pike

    // Compound assignment also works
    greeting := "Hello, "
    greeting += fullName
    fmt.Println(greeting) // Hello, Rob Pike
}
```

**Note**: For efficient string building, especially in loops, use `strings.Builder` instead of `+` concatenation.

## **4.8 Operator Precedence: Order of Operations**

Like mathematical expressions, Go has rules for the order in which operations are performed.

| Precedence | Operators                         |
| ---------- | --------------------------------- |
| Highest    | `()` (parentheses for grouping)   |
|            | `*` `/` `%` `<<` `>>` `&` `&^`    |
|            | `+` `-` `\|` `^`                  |
|            | `==` `!=` `<` `<=` `>` `>=`       |
|            | `&&`                              |
|            | `\|\|`                            |
| Lowest     | `=` `+=` `-=` `*=` `/=` `%=` etc. |

### **Examples of Precedence**

```go
package main

import "fmt"

func main() {
    // Without parentheses - multiplication has higher precedence
    result1 := 5 + 3 * 2
    fmt.Println(result1) // 11 (not 16)

    // With parentheses - explicitly control order
    result2 := (5 + 3) * 2
    fmt.Println(result2) // 16

    // Complex example
    x, y, z := 5, 3, 2
    result3 := x + y*z - x/y
    // Equivalent to: x + (y*z) - (x/y) = 5 + (3*2) - (5/3) = 5 + 6 - 1 = 10
    fmt.Println(result3) // 10
}
```

**Best Practice**: Use parentheses liberally to make your code's intent clear, even when not strictly necessary. It enhances readability and prevents precedence mistakes.

## **4.9 Performance Considerations**

### **1. Integer vs. Floating-Point Operations**

Integer operations are generally faster than floating-point operations. When performance is critical:

```go
// Less efficient
radius := 5.0
circumference := 2.0 * 3.14159 * radius

// More efficient (if precision allows)
radius := 5
circumference := 2 * 314159 * radius / 100000
```

### **2. Avoiding Division When Possible**

Division is typically slower than multiplication:

```go
// Less efficient
average := sum / count

// More efficient for repeated operations with the same divisor
invCount := 1.0 / float64(count)
average := float64(sum) * invCount
```

### **3. Bitwise Operations for Performance**

Bitwise operations can be much faster for certain tasks:

```go
// Less efficient
isEven := num % 2 == 0

// More efficient
isEven := (num & 1) == 0

// Less efficient
value := value * 2

// More efficient
value <<= 1
```

## **4.10 Common Pitfalls and Gotchas**

### **1. Integer Division Truncation**

```go
result := 5 / 2      // Equals 2, not 2.5
percentage := (count / total) * 100  // May be 0 if count < total
```

**Solution**: Convert to floating-point when decimal precision is needed:

```go
result := float64(5) / 2  // Equals 2.5
percentage := (float64(count) / float64(total)) * 100
```

### **2. Overflow in Integer Operations**

Go doesn't check for integer overflow at runtime:

```go
var x int8 = 127
x++ // Wraps around to -128!
```

**Solution**: Use larger integer types or check boundaries before operations:

```go
// Check for potential overflow before adding
if x > math.MaxInt64 - y {
    // Handle overflow error
}
```

### **3. Unintended Short-Circuit Evaluation**

```go
if configLoaded || loadConfig() {
    // If configLoaded is true, loadConfig() won't be called!
}
```

**Solution**: Be mindful of the evaluation order:

```go
if !configLoaded {
    configLoaded = loadConfig()
}
if configLoaded {
    // Proceed
}
```

## **4.11 Putting It All Together: Case Studies**

### **Case Study 1: Temperature Converter with Operators**

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    // Celsius to Fahrenheit: F = C * 9/5 + 32
    // Fahrenheit to Celsius: C = (F - 32) * 5/9

    celsius := 25.0
    fahrenheit := celsius*9/5 + 32

    // Round to 1 decimal place for display
    fahrenheitRounded := math.Round(fahrenheit*10) / 10

    fmt.Printf("%.1f°C = %.1f°F\n", celsius, fahrenheitRounded)

    // Convert back to Celsius
    convertedCelsius := (fahrenheit - 32) * 5 / 9

    // Check if our conversion is accurate (within floating-point precision)
    fmt.Printf("Converted back: %.2f°C\n", convertedCelsius)
    fmt.Printf("Conversion accuracy: %.10f°C difference\n", math.Abs(celsius - convertedCelsius))
}
```

### **Case Study 2: Bit Flags for Permissions**

```go
package main

import "fmt"

func main() {
    // Define permission bits
    const (
        PermRead = 1 << iota  // 1 (001)
        PermWrite             // 2 (010)
        PermExecute           // 4 (100)
    )

    // Create permission sets
    var guestPerms uint8 = PermRead
    var userPerms uint8 = PermRead | PermWrite
    var adminPerms uint8 = PermRead | PermWrite | PermExecute

    // Check permissions
    checkPermissions("Guest", guestPerms)
    checkPermissions("User", userPerms)
    checkPermissions("Admin", adminPerms)

    // Modify permissions
    fmt.Println("\nRevoking write permission from user...")
    userPerms &^= PermWrite
    checkPermissions("User", userPerms)
}

func checkPermissions(role string, perms uint8) {
    fmt.Printf("%s permissions (binary): %03b\n", role, perms)
    fmt.Printf("- Can read: %v\n", (perms & PermRead) != 0)
    fmt.Printf("- Can write: %v\n", (perms & PermWrite) != 0)
    fmt.Printf("- Can execute: %v\n", (perms & PermExecute) != 0)
}
```

## **4.12 Practice Exercises**

### **Exercise 1: Temperature Converter**

**Problem**: Write a program that converts temperatures between Celsius and Fahrenheit. Use arithmetic operators and control flow.

**Starter Code**:

```go
package main

import "fmt"

func main() {
    var temperature float64
    var unit string

    fmt.Print("Enter a temperature value: ")
    fmt.Scan(&temperature)
    fmt.Print("Enter unit (C or F): ")
    fmt.Scan(&unit)

    // TODO: Implement the conversion
    // If unit is C, convert to F using: F = C * 9/5 + 32
    // If unit is F, convert to C using: C = (F - 32) * 5/9

    // TODO: Print the result
}
```

### **Exercise 2: Bit Manipulation Challenge**

**Problem**: Write a program that:

1. Takes an integer input
2. Prints its binary representation
3. Counts the number of 1 bits it contains
4. Checks if it's a power of 2 (using bitwise operations)

**Hint**: A number is a power of 2 if and only if it has exactly one bit set to 1.

### **Exercise 3: Compound Expression Evaluator**

**Problem**: Build a simple expression evaluator that:

1. Takes three integers as input
2. Calculates various expressions combining them
3. Prints the results in a table format
4. Demonstrates operator precedence

## **4.13 Summary**

In this chapter, you've learned:

- **Arithmetic operators** for performing mathematical calculations
- **Comparison operators** for making decisions
- **Logical operators** for combining conditions
- **Assignment operators** for updating variables
- **Bitwise operators** for manipulating bits
- **Operator precedence** rules for complex expressions
- **Performance considerations** to write efficient code
- **Common pitfalls** to avoid

These operators are the building blocks for manipulating data in Go. By mastering them, you've taken another crucial step toward becoming a proficient Go programmer.

**Next Up**: In Chapter 5, we'll explore control structures like `if`, `for`, and `switch` to control the flow of your programs.
