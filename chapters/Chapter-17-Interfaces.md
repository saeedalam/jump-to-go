# **Chapter 17: Interfaces in Go**

---

## **17.1 What are Interfaces?**

In Go, an **interface** is a type that defines a set of method signatures. A type (usually a struct) satisfies an interface by implementing all the methods declared in the interface. This allows Go to achieve a form of polymorphism where you can write functions that work with any type that implements the interface.

### **Key Concepts**

- **Definition**: An interface specifies method signatures, but it does not implement them. It only defines behavior that types should adhere to.
- **Implementation**: A type satisfies an interface if it implements all the methods specified by the interface. Go does not require explicit declarations (like `implements` in other languages); if the methods match, the type implements the interface.
- **Dynamic Behavior**: Interfaces allow Go to write flexible and reusable code, as you can write functions that accept multiple types as long as they satisfy the interface.

---

## **17.2 Defining and Using Interfaces**

To understand how interfaces work, let’s start with a simple example. We will define an interface and then create types that implement that interface.

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
        fmt.Printf("Shape Area: %.2f
", shape.Area())
    }
}
```

### **Explanation**:

1. **Interface Definition**: The `Shape` interface has one method: `Area()`.
2. **Type Implementation**: `Circle` and `Rectangle` are two types that implement the `Area()` method, thus satisfying the `Shape` interface.
3. **Using the Interface**: We create instances of `Circle` and `Rectangle`, then store them in a slice of `Shape`. We can call the `Area()` method on any type that satisfies the `Shape` interface, regardless of its concrete type.

### **Output**

```
Shape Area: 78.50
Shape Area: 12.00
```

---

## **17.3 Interfaces in Functions**

Interfaces are extremely useful when you want to write generic functions that can operate on different types. In this case, we’ll use an interface to write a function that works with any type that implements a `Print()` method.

### **Example 2: Using Interfaces in Functions**

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

### **Explanation**:

1. **Interface Definition**: The `Printer` interface defines a `Print()` method.
2. **Type Implementations**: `Book` and `Article` implement the `Print()` method.
3. **Generic Function**: The `PrintDetails` function accepts any type that implements the `Printer` interface and calls the `Print()` method on it.

### **Output**

```
Book Title: Go Programming
Article by: Jane Doe
```

---

## **17.4 The Empty Interface**

The empty interface `interface{}` is a special type in Go. It can hold any value, as all types in Go implement at least zero methods. This allows you to store any type of value in an empty interface.

### **Example 3: The Empty Interface**

```go
package main

import "fmt"

func PrintAnything(v interface{}) {
    fmt.Printf("Value: %v, Type: %T
", v, v)
}

func main() {
    PrintAnything(42)
    PrintAnything("Hello, Go!")
    PrintAnything(3.14)
}
```

### **Explanation**:

1. The function `PrintAnything` takes a parameter of type `interface{}`, meaning it can accept any type of value.
2. It then prints the value and its type.

### **Output**

```
Value: 42, Type: int
Value: Hello, Go!, Type: string
Value: 3.14, Type: float64
```

---

## **17.5 Type Assertions**

Type assertions allow you to retrieve the underlying value of an interface. This is useful when you know the specific type of the value stored in the interface and want to extract it.

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

### **Explanation**:

1. We declare an interface `i` and assign it a string value.
2. The type assertion `i.(string)` checks if `i` holds a string, and if so, it extracts the value.

### **Output**

```
String Value: Hello, Go!
```

---

## **17.6 Type Switches**

A **type switch** is a way to handle different types more cleanly and efficiently than using a series of `if` statements. It lets you execute code based on the actual type of the interface value.

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

### **Explanation**:

1. **Type Switch**: The `switch` statement checks the type of `i`. If it's a string, it prints "String:", if it's an int, it prints "Integer:", and if it's any other type, it prints "Unknown Type".

### **Output**

```
String: Go
Integer: 42
Unknown Type
```

---

## **17.7 Advanced Examples**

Interfaces can be used for more advanced techniques like **polymorphism**, where different types implement the same interface and can be used interchangeably.

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

### **Explanation**:

1. **Animal Interface**: The `Animal` interface requires a `Speak()` method.
2. **Types Implementing Animal**: Both `Dog` and `Cat` types implement the `Speak()` method.
3. **Polymorphism**: The `main` function treats `Dog` and `Cat` as `Animal` and calls `Speak()` on each.

### **Output**

```
Woof!
Meow!
```

---

## **17.8 Summary Table**

| Feature             | Description                        | Example                                   |
| ------------------- | ---------------------------------- | ----------------------------------------- |
| **Basic Interface** | Defines method signatures          | `type Shape interface { Area() float64 }` |
| **Empty Interface** | Can hold any value                 | `interface{}`                             |
| **Type Assertion**  | Extracts value of specific type    | `s, ok := i.(string)`                     |
| **Type Switch**     | Dynamically handles multiple types | `switch v := i.(type)`                    |

---

# **17.9. Exercises**

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

## **Exercise 8: Online Store**

**Problem**: Define a `Product` interface with `GetPrice()` method. Implement it for `Book` and `Electronics`.

```go
package main

import "fmt"

type Product interface {
    GetPrice() float64
}

type Book struct {
    Title  string
    Price  float64
}

func (b Book) GetPrice() float64 {
    return b.Price
}

type Electronics struct {
    Name   string
    Price  float64
}

func (e Electronics) GetPrice() float64 {
    return e.Price
}

func main() {
    products := []Product{
        Book{Title: "Go Programming", Price: 29.99},
        Electronics{Name: "Laptop", Price: 799.99},
    }
    for _, product := range products {
        fmt.Printf("Product Price: $%.2f
", product.GetPrice())
    }
}
```

**Output:**

```
Product Price: $29.99
Product Price: $799.99
```

---

## **Exercise 9: Document Generation**

**Problem**: Create a `Document` interface with a `Generate()` method. Implement it for `Report` and `Invoice`.

```go
package main

import "fmt"

type Document interface {
    Generate() string
}

type Report struct {
    Title string
}

func (r Report) Generate() string {
    return "Report: " + r.Title
}

type Invoice struct {
    Number string
}

func (i Invoice) Generate() string {
    return "Invoice Number: " + i.Number
}

func main() {
    documents := []Document{
        Report{Title: "Annual Financial Report"},
        Invoice{Number: "INV12345"},
    }
    for _, doc := range documents {
        fmt.Println(doc.Generate())
    }
}
```

**Output:**

```
Report: Annual Financial Report
Invoice Number: INV12345
```

---

## **Exercise 10: File Operations**

**Problem**: Define a `FileProcessor` interface with methods `Open()` and `Close()`. Implement the interface for `TextFile` and `ImageFile`.

```go
package main

import "fmt"

type FileProcessor interface {
    Open()
    Close()
}

type TextFile struct {
    Filename string
}

func (t TextFile) Open() {
    fmt.Printf("Opening text file: %s
", t.Filename)
}

func (t TextFile) Close() {
    fmt.Printf("Closing text file: %s
", t.Filename)
}

type ImageFile struct {
    Filename string
}

func (i ImageFile) Open() {
    fmt.Printf("Opening image file: %s
", i.Filename)
}

func (i ImageFile) Close() {
    fmt.Printf("Closing image file: %s
", i.Filename)
}

func main() {
    files := []FileProcessor{
        TextFile{Filename: "document.txt"},
        ImageFile{Filename: "picture.jpg"},
    }
    for _, file := range files {
        file.Open()
        file.Close()
    }
}
```

**Output:**

```
Opening text file: document.txt
Closing text file: document.txt
Opening image file: picture.jpg
Closing image file: picture.jpg
```

---

