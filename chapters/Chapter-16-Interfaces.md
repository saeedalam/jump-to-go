# **Chapter 16: Interfaces in Go**

---

## **16.1 What are Interfaces?**

An **interface** in Go defines a set of method signatures that a type must implement. It allows you to write flexible and reusable code by decoupling functionality from concrete types.

### **Key Concepts**

- **Definition**: An interface specifies method signatures.
- **Implementation**: A type satisfies an interface if it implements all the methods in the interface.
- **Dynamic Behavior**: Interfaces let you write code that works with different types.

---

## **16.2 Defining and Using Interfaces**

Let’s start with a simple example.

### **Example 1: Defining and Implementing an Interface**

```go
package main

import "fmt"

// Define an interface
type Shape interface {
    Area() float64
}

// Define a type that implements the interface
type Circle struct {
    Radius float64
}

// Implement the Area method for Circle
func (c Circle) Area() float64 {
    return 3.14 * c.Radius * c.Radius
}

// Another type that implements Shape
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func main() {
    // Create instances
    c := Circle{Radius: 5}
    r := Rectangle{Width: 4, Height: 3}

    // Call Area via Shape interface
    shapes := []Shape{c, r}
    for _, shape := range shapes {
        fmt.Printf("Shape Area: %.2f\n", shape.Area())
    }
}
```

### **Output**

```
Shape Area: 78.50
Shape Area: 12.00
```

---

## **16.3 Interfaces in Functions**

### **Example 2: Using Interfaces in Functions**

Interfaces are especially useful for writing generic functions.

```go
package main

import "fmt"

// Define an interface
type Printer interface {
    Print()
}

// Define two types
type Book struct {
    Title string
}

func (b Book) Print() {
    fmt.Println("Book Title:", b.Title)
}

type Article struct {
    Author string
}

func (a Article) Print() {
    fmt.Println("Article by:", a.Author)
}

// Function that works with the Printer interface
func PrintDetails(p Printer) {
    p.Print()
}

func main() {
    b := Book{Title: "Go Programming"}
    a := Article{Author: "Jane Doe"}

    PrintDetails(b)
    PrintDetails(a)
}
```

### **Output**

```
Book Title: Go Programming
Article by: Jane Doe
```

---

## **16.4 The Empty Interface**

The `interface{}` type can hold any value.

### **Example 3: The Empty Interface**

```go
package main

import "fmt"

func PrintAnything(v interface{}) {
    fmt.Printf("Value: %v, Type: %T\n", v, v)
}

func main() {
    PrintAnything(42)
    PrintAnything("Hello, Go!")
    PrintAnything(3.14)
}
```

### **Output**

```
Value: 42, Type: int
Value: Hello, Go!, Type: string
Value: 3.14, Type: float64
```

---

## **16.5 Type Assertions**

Type assertions allow you to retrieve the underlying value of an interface.

### **Example 4: Basic Type Assertion**

```go
package main

import "fmt"

func main() {
    var i interface{} = "Hello, Go!"

    // Type assertion
    s, ok := i.(string)
    if ok {
        fmt.Println("String Value:", s)
    } else {
        fmt.Println("Not a string")
    }
}
```

### **Output**

```
String Value: Hello, Go!
```

---

## **16.6 Type Switches**

Type switches provide a cleaner way to work with dynamic types.

### **Example 5: Using a Type Switch**

```go
package main

import "fmt"

func Describe(i interface{}) {
    switch v := i.(type) {
    case string:
        fmt.Println("String:", v)
    case int:
        fmt.Println("Integer:", v)
    default:
        fmt.Println("Unknown Type")
    }
}

func main() {
    Describe("Go")
    Describe(42)
    Describe(3.14)
}
```

### **Output**

```
String: Go
Integer: 42
Unknown Type
```

---

## **16.7 Advanced Examples**

### **Example 6: Polymorphism with Interfaces**

```go
package main

import "fmt"

type Animal interface {
    Speak() string
}

type Dog struct{}

func (d Dog) Speak() string {
    return "Woof!"
}

type Cat struct{}

func (c Cat) Speak() string {
    return "Meow!"
}

func main() {
    animals := []Animal{Dog{}, Cat{}}
    for _, animal := range animals {
        fmt.Println(animal.Speak())
    }
}
```

### **Output**

```
Woof!
Meow!
```

---

## **16.8 Summary Table**

| Feature             | Description                        | Example                                   |
| ------------------- | ---------------------------------- | ----------------------------------------- |
| **Basic Interface** | Defines method signatures          | `type Shape interface { Area() float64 }` |
| **Empty Interface** | Can hold any value                 | `interface{}`                             |
| **Type Assertion**  | Extracts value of specific type    | `s, ok := i.(string)`                     |
| **Type Switch**     | Dynamically handles multiple types | `switch v := i.(type)`                    |

---

# **16.9. Exercises**

---

## **Exercise 1: Polymorphism with Shapes**

**Problem**: Define a `Shape` interface with a method `Perimeter()`. Implement the interface for `Circle` and `Rectangle`.

```go
package main

import (
    "fmt"
    "math"
)

type Shape interface {
    Perimeter() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func main() {
    shapes := []Shape{
        Circle{Radius: 5},
        Rectangle{Width: 4, Height: 6},
    }
    for _, shape := range shapes {
        fmt.Printf("Perimeter: %.2f
", shape.Perimeter())
    }
}
```

**Output:**

```
Perimeter: 31.42
Perimeter: 20.00
```

---

## **Exercise 2: Sorting with Interfaces**

**Problem**: Use the `sort.Interface` to sort a slice of custom `Employee` types by their salary.

```go
package main

import (
    "fmt"
    "sort"
)

type Employee struct {
    Name   string
    Salary int
}

type BySalary []Employee

func (e BySalary) Len() int           { return len(e) }
func (e BySalary) Swap(i, j int)      { e[i], e[j] = e[j], e[i] }
func (e BySalary) Less(i, j int) bool { return e[i].Salary < e[j].Salary }

func main() {
    employees := []Employee{
        {"Alice", 50000},
        {"Bob", 70000},
        {"Charlie", 60000},
    }
    sort.Sort(BySalary(employees))
    fmt.Println("Sorted Employees by Salary:")
    for _, e := range employees {
        fmt.Println(e.Name, "-", e.Salary)
    }
}
```

**Output:**

```
Sorted Employees by Salary:
Alice - 50000
Charlie - 60000
Bob - 70000
```

---

## **Exercise 3: Dynamic Type Checker**

**Problem**: Create a function `CheckType` that uses a type switch to print the type of the given input.

```go
package main

import "fmt"

func CheckType(i interface{}) {
    switch v := i.(type) {
    case string:
        fmt.Println("Type: string, Value:", v)
    case int:
        fmt.Println("Type: int, Value:", v)
    case float64:
        fmt.Println("Type: float64, Value:", v)
    default:
        fmt.Println("Unknown Type")
    }
}

func main() {
    CheckType("Go")
    CheckType(42)
    CheckType(3.14)
    CheckType([]int{1, 2, 3})
}
```

**Output:**

```
Type: string, Value: Go
Type: int, Value: 42
Type: float64, Value: 3.14
Unknown Type
```

---

## **Exercise 4: Logging with Empty Interface**

**Problem**: Write a logger function that accepts an empty interface and prints values with their types.

```go
package main

import "fmt"

func Logger(v interface{}) {
    fmt.Printf("Type: %T, Value: %v
", v, v)
}

func main() {
    Logger("Hello")
    Logger(2024)
    Logger(3.1415)
    Logger(true)
}
```

**Output:**

```
Type: string, Value: Hello
Type: int, Value: 2024
Type: float64, Value: 3.1415
Type: bool, Value: true
```

---

## **Exercise 5: Payment System**

**Problem**: Create a `Payment` interface with a method `Pay()`. Implement the interface for `CreditCard` and `PayPal`.

```go
package main

import "fmt"

type Payment interface {
    Pay(amount float64)
}

type CreditCard struct {
    Number string
}

func (c CreditCard) Pay(amount float64) {
    fmt.Printf("Paid $%.2f with Credit Card ending in %s
", amount, c.Number[len(c.Number)-4:])
}

type PayPal struct {
    Email string
}

func (p PayPal) Pay(amount float64) {
    fmt.Printf("Paid $%.2f using PayPal account: %s
", amount, p.Email)
}

func main() {
    payments := []Payment{
        CreditCard{Number: "1234567812345678"},
        PayPal{Email: "user@example.com"},
    }
    for _, payment := range payments {
        payment.Pay(100.50)
    }
}
```

**Output:**

```
Paid $100.50 with Credit Card ending in 5678
Paid $100.50 using PayPal account: user@example.com
```

---

## **Exercise 6: Real-Time Messaging**

**Problem**: Create a `Messenger` interface and implement it for `Email` and `SMS`.

```go
package main

import "fmt"

type Messenger interface {
    SendMessage(recipient, message string)
}

type Email struct{}

func (e Email) SendMessage(recipient, message string) {
    fmt.Printf("Email sent to %s: %s
", recipient, message)
}

type SMS struct{}

func (s SMS) SendMessage(recipient, message string) {
    fmt.Printf("SMS sent to %s: %s
", recipient, message)
}

func main() {
    var m Messenger
    m = Email{}
    m.SendMessage("example@example.com", "Hello via Email!")

    m = SMS{}
    m.SendMessage("+123456789", "Hello via SMS!")
}
```

**Output:**

```
Email sent to example@example.com: Hello via Email!
SMS sent to +123456789: Hello via SMS!
```

---

## **Exercise 7: Zoo Animals**

**Problem**: Create an `Animal` interface with `Speak()` and `Move()` methods. Implement it for `Dog` and `Bird`.

```go
package main

import "fmt"

type Animal interface {
    Speak() string
    Move() string
}

type Dog struct{}

func (d Dog) Speak() string { return "Woof!" }
func (d Dog) Move() string  { return "Run" }

type Bird struct{}

func (b Bird) Speak() string { return "Tweet!" }
func (b Bird) Move() string  { return "Fly" }

func main() {
    animals := []Animal{Dog{}, Bird{}}
    for _, animal := range animals {
        fmt.Printf("Animal: %s, Move: %s
", animal.Speak(), animal.Move())
    }
}
```

**Output:**

```
Animal: Woof!, Move: Run
Animal: Tweet!, Move: Fly
```

---

**Stay tuned for more exercises in the extended version!**
