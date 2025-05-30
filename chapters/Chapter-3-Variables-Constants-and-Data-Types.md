# **Chapter 3: The Foundation of Go: Variables, Constants, and Types**

In Go, how you define and manipulate data shapes your entire program. Unlike some languages where types are suggestions, Go's type system provides both safety and performance through a carefully balanced design. Let's dive into the building blocks that will form the foundation of every Go program you write.

## **3.1 Variables: The Dynamic Elements of Your Program**

Variables are named storage locations whose values can change during program execution. Go offers two primary ways to create them—each with distinct advantages.

### **3.1.1 Declaration Methods**

#### **Method 1: Using the `var` Keyword**

```go
var name string = "Gopher"
var age int = 5
var height float64 = 0.5
```

This traditional approach:

- Works in all scopes (package level and function level)
- Makes the type explicitly visible
- Can separate declaration from initialization

#### **Method 2: Short Declaration (`:=`)**

```go
name := "Gopher"  // Type string is inferred
age := 5          // Type int is inferred
height := 0.5     // Type float64 is inferred
```

The short declaration:

- Is more concise and idiomatic
- Works only inside functions
- Always initializes the variable
- Uses type inference to determine the type

### **3.1.2 Type Inference: Go's Smart Type Detection**

Go determines the appropriate type based on the provided value. This feature makes code concise without sacrificing type safety:

```go
country := "Japan"       // Inferred as string
population := 126_000_000 // Inferred as int
taxRate := 0.1           // Inferred as float64
```

**Why it matters**: Type inference reduces verbosity while maintaining complete type safety, striking an excellent balance between dynamic and statically-typed languages.

### **3.1.3 Multiple Variable Declarations**

You can declare multiple variables in a single statement for cleaner code:

```go
// Multiple variables with var
var (
    firstName string = "Robert"
    lastName  string = "Pike"
    yearBorn  int    = 1956
)

// Multiple short declarations
city, region, zipCode := "San Francisco", "California", 94103
```

### **3.1.4 Zero Values: Go's Safety Net**

When a variable is declared but not initialized, Go assigns it a type-appropriate **zero value**:

```go
package main

import "fmt"

func main() {
    var integer int
    var floatingPoint float64
    var boolean bool
    var text string
    var pointer *int

    fmt.Println("Integer:", integer)            // 0
    fmt.Println("Floating-point:", floatingPoint) // 0.0
    fmt.Println("Boolean:", boolean)            // false
    fmt.Println("String:", text)                // "" (empty string)
    fmt.Println("Pointer:", pointer)            // nil
}
```

**Why it matters**: Zero values eliminate undefined behavior, a major source of bugs in other languages. Your variables always start with a sensible, predictable state.

### **3.1.5 Variable Scope and Shadowing**

In Go, variables have different visibility depending on where they're declared:

```go
package main

import "fmt"

var globalVariable = "I'm visible throughout the package"

func main() {
    fmt.Println(globalVariable)

    localVariable := "I'm only visible in main()"
    fmt.Println(localVariable)

    if true {
        // This shadows the outer globalVariable
        globalVariable := "I'm a different variable"
        fmt.Println(globalVariable) // Prints: I'm a different variable
    }

    fmt.Println(globalVariable) // Prints: I'm visible throughout the package
}
```

**Shadowing** occurs when a variable declared in an inner scope has the same name as one in an outer scope. This is legal but can lead to confusion, so use it with care.

### **3.1.6 Best Practices for Variables**

1. **Use short declaration (`:=`) within functions**

   ```go
   // Preferred
   result := calculateValue()

   // Less common, except at package level
   var result = calculateValue()
   ```

2. **Use descriptive variable names**

   ```go
   // Good
   userCount := getUserCount()

   // Too vague
   cnt := getUserCount()
   ```

3. **Follow Go's convention: camelCase for variables**

   ```go
   // Correct
   maxRetryCount := 5

   // Not idiomatic Go
   max_retry_count := 5
   ```

## **3.2 Constants: The Immutable Anchors**

Constants are values fixed at compile time that cannot change during program execution. They bring both safety and optimization opportunities.

### **3.2.1 Declaring Constants**

Use the `const` keyword to declare constants:

```go
const pi = 3.14159
const (
    appName    = "GoTracker"
    appVersion = "1.0.0"
    maxUsers   = 1000
)
```

### **3.2.2 Typed vs. Untyped Constants**

Go constants come in two flavors:

```go
// Typed constant - can only be used where float64 is allowed
const typedPi float64 = 3.14159

// Untyped constant - more flexible, adapts to context
const untypedPi = 3.14159
```

Untyped constants have enormous flexibility:

```go
const untypedNumber = 42

var a int = untypedNumber       // Works fine
var b float64 = untypedNumber   // Works fine
var c complex128 = untypedNumber // Works fine too!

// But typed constants are restricted:
const typedNumber int = 42
var d int = typedNumber        // Works fine
// var e float64 = typedNumber // Compile error!
```

**Why it matters**: Untyped constants make Go more ergonomic while maintaining type safety, letting you use constants naturally in different contexts.

### **3.2.3 The Power of `iota`**

`iota` is Go's built-in counter for creating sequences of related constants:

```go
const (
    Sunday = iota    // 0
    Monday           // 1
    Tuesday          // 2
    Wednesday        // 3
    Thursday         // 4
    Friday           // 5
    Saturday         // 6
)
```

With more complex expressions:

```go
const (
    _  = iota             // Ignore first value (0)
    KB = 1 << (10 * iota) // 1 << 10 = 1024
    MB                    // 1 << 20 = 1,048,576
    GB                    // 1 << 30 = 1,073,741,824
    TB                    // 1 << 40 = 1,099,511,627,776
)
```

**Why it matters**: `iota` reduces both code and maintenance burden. When you add a new constant in the middle, you don't need to renumber everything.

### **3.2.4 Constant Rules and Best Practices**

1. **Constants must be determinable at compile time**

   ```go
   const a = 10        // OK
   const b = a + 5     // OK
   const c = math.Sin(0) // OK (result is determinable at compile time)

   // Not allowed:
   // const d = time.Now() // Error: not constant
   // const e = rand.Intn(10) // Error: not constant
   ```

2. **Use constants for values that truly never change**

   ```go
   const (
       secondsInMinute = 60
       minutesInHour   = 60
       hoursInDay      = 24
   )
   ```

3. **Group related constants together**

   ```go
   const (
       StatusOK      = 200
       StatusCreated = 201
       StatusAccepted = 202

       // Instead of scattered declarations
       // const StatusOK = 200
       // ...other code...
       // const StatusCreated = 201
   )
   ```

## **3.3 Go's Rich Type System**

Go's type system provides safety, clarity, and performance. Let's explore the core types that form the backbone of every Go program.

### **3.3.1 Numeric Types**

#### **Integers**

Go offers a range of integer types with different sizes:

| Type     | Size (bits) | Range                                                   | Use Case                      |
| -------- | ----------- | ------------------------------------------------------- | ----------------------------- |
| `int8`   | 8           | -128 to 127                                             | Very small integers           |
| `uint8`  | 8           | 0 to 255                                                | Bytes, small positive numbers |
| `int16`  | 16          | -32,768 to 32,767                                       | Small integers                |
| `uint16` | 16          | 0 to 65,535                                             | Small positive integers       |
| `int32`  | 32          | -2,147,483,648 to 2,147,483,647                         | Medium integers, runes        |
| `uint32` | 32          | 0 to 4,294,967,295                                      | Medium positive integers      |
| `int64`  | 64          | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 | Large integers                |
| `uint64` | 64          | 0 to 18,446,744,073,709,551,615                         | Large positive integers       |
| `int`    | 32 or 64    | Platform dependent                                      | Default integer type          |
| `uint`   | 32 or 64    | Platform dependent                                      | Default unsigned integer      |

```go
var age int = 30       // Platform-dependent size (usually 64-bit on modern systems)
var count int64 = 9223372036854775807 // Guaranteed 64-bit
var small uint8 = 255  // 8-bit unsigned (0-255)
```

**Special integer types**:

```go
var b byte = 65        // byte is an alias for uint8, ideal for raw data
var r rune = 'A'       // rune is an alias for int32, used for Unicode code points
```

#### **Floating-Point Numbers**

For decimal values, Go provides:

| Type      | Size (bits) | Precision          | Use Case                                            |
| --------- | ----------- | ------------------ | --------------------------------------------------- |
| `float32` | 32          | ~7 decimal digits  | When space matters more than precision              |
| `float64` | 64          | ~15 decimal digits | Default for most decimal numbers (higher precision) |

```go
var height float64 = 1.78
var weight float32 = 68.5
```

#### **Complex Numbers**

For scientific and engineering applications:

```go
var c1 complex64 = 5 + 7i   // Made of two float32s
var c2 complex128 = 1.2 + 3.4i // Made of two float64s (default complex type)
```

### **3.3.2 The Boolean Type**

The `bool` type represents boolean logic with two possible values: `true` and `false`.

```go
var isActive bool = true
var hasPermission = false // Type inferred
```

Boolean values are crucial for control flow:

```go
if isActive && hasPermission {
    // Do something when both conditions are true
}
```

### **3.3.3 Strings: Not Just Text**

Strings in Go are immutable sequences of bytes, typically UTF-8 encoded text:

```go
var name string = "Gopher"
greeting := "Hello, 世界" // UTF-8 support by default
```

**Key string operations**:

```go
message := "Hello, Go!"

// Length (returns number of bytes, not characters)
length := len(message) // 10

// Accessing individual bytes (not characters)
firstByte := message[0] // 'H'

// Substring (slicing)
substr := message[7:9] // "Go"

// Concatenation
fullMessage := message + " Welcome!" // "Hello, Go! Welcome!"
```

**Important**: Strings are immutable, so operations create new strings:

```go
s := "hello"
s = s + " world" // Creates a new string, not modifying the original
```

For efficient string building, use the `strings.Builder` type:

```go
var builder strings.Builder
builder.WriteString("Hello")
builder.WriteString(", ")
builder.WriteString("Go!")
result := builder.String() // "Hello, Go!"
```

### **3.3.4 Types for Unicode and Multi-Language Support**

The `rune` type (alias for `int32`) represents a Unicode code point:

```go
greeting := "Hello, 世界"

for i, char := range greeting {
    fmt.Printf("Position %d: %c (Unicode: %U)\n", i, char, char)
}
```

This handles multi-byte characters properly, unlike simple indexing.

### **3.3.5 Creating Your Own Types**

Go lets you create custom types, enhancing clarity and safety:

```go
// Type definition - creates a completely new type
type UserID int64

// Type alias - creates an alternative name for an existing type
type Byte = uint8

func main() {
    var id UserID = 12345
    var num int64 = 12345

    // This won't compile - they're different types despite same underlying type
    // id = num

    // This works - explicit conversion required
    id = UserID(num)
}
```

**Why it matters**: Custom types prevent accidental misuse. You can't accidentally pass a plain `int64` where a `UserID` is required.

## **3.4 Type Conversion: Crossing Type Boundaries**

Go requires explicit type conversions, enforcing clarity and preventing subtle bugs.

### **3.4.1 Basic Type Conversion**

```go
var i int = 42
var f float64 = float64(i) // Convert int to float64
var u uint = uint(i)       // Convert int to uint
```

### **3.4.2 Numeric Conversion Gotchas**

Be aware of potential issues:

```go
var large int64 = 9223372036854775807
var truncated int32 = int32(large) // Truncation occurs! Value becomes -1

var negative int = -42
var unsigned uint = uint(negative) // Becomes a large positive number

var pi float64 = 3.14159
var rounded int = int(pi) // Truncates to 3, no rounding
```

### **3.4.3 String Conversions**

For numeric to string conversions, use the `strconv` package:

```go
import "strconv"

func main() {
    // Integer to string
    value := 42
    strValue := strconv.Itoa(value) // "42"

    // String to integer
    numStr := "123"
    num, err := strconv.Atoi(numStr)
    if err != nil {
        // Handle conversion error
    }

    // Float to string with precision control
    pi := 3.14159
    piStr := strconv.FormatFloat(pi, 'f', 2, 64) // "3.14"
}
```

For general value formatting, `fmt.Sprintf` is powerful:

```go
import "fmt"

func main() {
    count := 42
    message := fmt.Sprintf("There are %d items remaining", count) // "There are 42 items remaining"

    pi := 3.14159
    formatted := fmt.Sprintf("Pi to 2 decimal places: %.2f", pi) // "Pi to 2 decimal places: 3.14"
}
```

## **3.5 Putting It All Together: A Complete Example**

Let's tie everything together with a complete program that demonstrates variables, constants, types, and conversions:

```go
package main

import (
    "fmt"
    "strconv"
)

// Custom type for temperature
type Celsius float64
type Fahrenheit float64

// Package-level constants
const (
    FreezingC Celsius = 0
    BoilingC  Celsius = 100
)

// Package-level variables
var (
    city     string = "Toronto"
    latitude  float64 = 43.65
    longitude float64 = -79.38
)

// Convert Celsius to Fahrenheit
func celsiusToFahrenheit(c Celsius) Fahrenheit {
    return Fahrenheit(c*9/5 + 32)
}

// Convert Fahrenheit to Celsius
func fahrenheitToCelsius(f Fahrenheit) Celsius {
    return Celsius((f - 32) * 5 / 9)
}

func main() {
    // Using constants
    fmt.Printf("Water freezes at %g°C or %g°F\n",
               FreezingC, celsiusToFahrenheit(FreezingC))
    fmt.Printf("Water boils at %g°C or %g°F\n",
               BoilingC, celsiusToFahrenheit(BoilingC))

    // Using variables
    currentTemp := Celsius(22.5)
    fmt.Printf("Current temperature in %s: %g°C\n", city, currentTemp)

    // Type conversion
    fmt.Printf("Location: %.2f°N, %.2f°W\n", latitude, longitude)

    // String conversion
    tempStr := strconv.FormatFloat(float64(currentTemp), 'f', 1, 64)
    fmt.Printf("As a string: %s°C\n", tempStr)

    // Multiple variable declaration and assignment
    min, max := -40, 40
    fmt.Printf("Interesting fact: At %.1f degrees, Celsius and Fahrenheit scales meet.\n",
              fahrenheitToCelsius(Fahrenheit(min)))

    // Using iota for enumeration
    const (
        Low = iota
        Medium
        High
        Critical
    )

    severity := Medium
    fmt.Printf("Current alert level: %d\n", severity)
}
```

This example showcases the interplay between various Go language features related to variables, constants, and types.

## **3.6 Summary**

In this chapter, you've learned about:

- **Variables**: How to declare, initialize, and scope them
- **Constants**: Creating immutable values and the power of `iota`
- **Basic Types**: Understanding integers, floats, booleans, and strings
- **Custom Types**: Making your code more expressive and type-safe
- **Type Conversion**: Safely moving between different types

These fundamental concepts form the building blocks for everything else in Go. By mastering them, you've taken a crucial step toward Go proficiency.

**Challenge**: Create a program that works with different currencies and conversion rates. Use custom types to represent each currency (USD, EUR, GBP), constants for conversion rates, and implement functions to convert between them.

**Next Up**: In Chapter 4, we'll explore operators and expressions, which let you manipulate these variables and values to perform useful work.
