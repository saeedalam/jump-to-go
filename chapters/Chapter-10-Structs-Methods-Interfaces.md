# **Chapter 10: Structs, Methods, and Interfaces in Go**

Go offers a unique approach to object-oriented programming through structs, methods, and interfaces. Rather than implementing traditional inheritance-based class hierarchies, Go favors composition and interface-based polymorphism. This approach leads to more flexible, maintainable code that aligns with Go's philosophy of simplicity and pragmatism.

In this chapter, we'll explore these fundamental concepts and learn how to use them effectively to structure your Go programs.

## **10.1 Structs: Custom Data Types**

### **10.1.1 Struct Fundamentals**

A struct is a composite data type that groups together variables under a single name. Each variable within a struct is called a field, and each field has a name and a type. Structs allow you to create custom data types that model real-world entities.

```go
package main

import "fmt"

// Define a struct type
type Person struct {
    FirstName string
    LastName  string
    Age       int
}

func main() {
    // Create a new Person
    person := Person{
        FirstName: "John",
        LastName:  "Doe",
        Age:       30,
    }

    fmt.Println("Person:", person)
    fmt.Println("First Name:", person.FirstName)
    fmt.Println("Last Name:", person.LastName)
    fmt.Println("Age:", person.Age)
}
```

Key characteristics of structs:

- **Custom types**: Structs let you define your own data types
- **Grouping related data**: Fields within a struct typically have a logical relationship
- **Value semantics**: Structs are copied when assigned or passed to functions (unless pointers are used)
- **Exported fields**: Fields starting with an uppercase letter are exported (visible outside the package)

### **10.1.2 Creating and Initializing Structs**

There are several ways to create and initialize structs in Go:

```go
package main

import "fmt"

type Rectangle struct {
    Width  float64
    Height float64
}

func main() {
    // Method 1: Using field names (recommended)
    rect1 := Rectangle{Width: 10.5, Height: 5.0}

    // Method 2: Positional syntax (order matters)
    rect2 := Rectangle{10.5, 5.0}

    // Method 3: Declare first, then initialize fields
    var rect3 Rectangle
    rect3.Width = 10.5
    rect3.Height = 5.0

    // Method 4: Using the new function (returns a pointer)
    rect4 := new(Rectangle)
    rect4.Width = 10.5
    rect4.Height = 5.0

    fmt.Println("Rectangle 1:", rect1)
    fmt.Println("Rectangle 2:", rect2)
    fmt.Println("Rectangle 3:", rect3)
    fmt.Println("Rectangle 4:", *rect4)
}
```

When initializing a struct:

- Using field names is more maintainable and less error-prone
- Fields not explicitly initialized get their zero values
- The `new` function allocates zeroed memory and returns a pointer

### **10.1.3 Nested Structs**

Structs can contain other structs as fields, allowing for composition of complex data structures:

```go
package main

import "fmt"

type Address struct {
    Street  string
    City    string
    Country string
    ZipCode string
}

type Employee struct {
    FirstName string
    LastName  string
    Address   Address  // Nested struct
    Skills    []string // Slice field
}

func main() {
    employee := Employee{
        FirstName: "Alice",
        LastName:  "Johnson",
        Address: Address{
            Street:  "123 Main St",
            City:    "Boston",
            Country: "USA",
            ZipCode: "02101",
        },
        Skills: []string{"Go", "Python", "SQL"},
    }

    fmt.Println("Employee:", employee.FirstName, employee.LastName)
    fmt.Println("City:", employee.Address.City)
    fmt.Println("Skills:", employee.Skills)
}
```

Nested structs enable:

- Logical grouping of related data
- Reusability of common data structures
- Modeling of complex relationships

### **10.1.4 Anonymous Structs**

Go allows you to declare anonymous structs without defining a named type:

```go
package main

import "fmt"

func main() {
    // Anonymous struct declaration and initialization
    person := struct {
        Name string
        Age  int
    }{
        Name: "Bob",
        Age:  25,
    }

    fmt.Println("Person:", person)
}
```

Anonymous structs are useful for:

- One-off use cases
- Returning multiple values with logical structure
- Temporary data grouping

### **10.1.5 Struct Embedding**

Go supports embedding one struct within another, which provides a form of composition:

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

type Employee struct {
    Person      // Embedded struct (no field name)
    CompanyName string
    JobTitle    string
}

func main() {
    employee := Employee{
        Person:      Person{Name: "Sarah", Age: 28},
        CompanyName: "Acme Inc",
        JobTitle:    "Software Engineer",
    }

    // Fields of the embedded struct can be accessed directly
    fmt.Println("Name:", employee.Name)        // Access Person.Name directly
    fmt.Println("Age:", employee.Age)          // Access Person.Age directly
    fmt.Println("Company:", employee.CompanyName)

    // The embedded struct can also be accessed as a field
    fmt.Println("Person:", employee.Person)
}
```

Struct embedding provides:

- Field promotion (embedded struct fields appear as fields of the outer struct)
- A form of composition rather than inheritance
- Code reuse without tight coupling

## **10.2 Methods: Adding Behavior to Types**

### **10.2.1 Method Fundamentals**

A method is a function associated with a specific type. Methods allow you to add behavior to your custom types, creating a more object-oriented style of programming in Go.

```go
package main

import "fmt"

type Rectangle struct {
    Width  float64
    Height float64
}

// Method with a receiver
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Another method on the same type
func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}

    // Call methods
    area := rect.Area()
    perimeter := rect.Perimeter()

    fmt.Printf("Rectangle: %+v\n", rect)
    fmt.Printf("Area: %.2f\n", area)
    fmt.Printf("Perimeter: %.2f\n", perimeter)
}
```

Key concepts in method declarations:

- The receiver `(r Rectangle)` binds the method to the Rectangle type
- Methods are called using dot notation on instances of the type
- Methods can access the fields of the receiver

### **10.2.2 Value Receivers vs. Pointer Receivers**

Methods can have either value receivers or pointer receivers, each with different characteristics:

```go
package main

import "fmt"

type Counter struct {
    Count int
}

// Method with value receiver - receives a copy
func (c Counter) Increment() {
    c.Count++
    fmt.Println("Inside value method:", c.Count)
}

// Method with pointer receiver - receives a reference
func (c *Counter) IncrementByReference() {
    c.Count++
    fmt.Println("Inside pointer method:", c.Count)
}

func main() {
    counter := Counter{Count: 0}

    // Call method with value receiver
    counter.Increment()
    fmt.Println("After value method:", counter.Count)  // Still 0

    // Call method with pointer receiver
    counter.IncrementByReference()
    fmt.Println("After pointer method:", counter.Count)  // Now 1
}
```

Choosing between value and pointer receivers:

| Value Receiver                                   | Pointer Receiver                                    |
| ------------------------------------------------ | --------------------------------------------------- |
| Receives a copy of the instance                  | Receives a reference to the instance                |
| Cannot modify the original instance              | Can modify the original instance                    |
| Better for small, immutable types                | Better for large structs or when mutation is needed |
| Indicates the method doesn't modify the receiver | Indicates the method might modify the receiver      |

### **10.2.3 Methods on Non-Struct Types**

In Go, you can define methods on any type you define, not just structs:

```go
package main

import (
    "fmt"
    "strings"
)

// Define a new type based on a built-in type
type MyString string

// Method for the custom string type
func (s MyString) Reverse() string {
    runes := []rune(string(s))
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

// Another method
func (s MyString) ToUpperCase() string {
    return strings.ToUpper(string(s))
}

func main() {
    str := MyString("Hello, Go!")
    fmt.Println("Original:", str)
    fmt.Println("Reversed:", str.Reverse())
    fmt.Println("Uppercase:", str.ToUpperCase())
}
```

Important constraints:

- You can only define methods on types defined in the same package
- You cannot define methods on built-in types directly

### **10.2.4 Method Chaining**

Methods can be chained together when they return the receiver type:

```go
package main

import (
    "fmt"
    "strings"
)

type StringBuilder struct {
    data string
}

// Methods that return the receiver pointer for chaining
func (sb *StringBuilder) Append(s string) *StringBuilder {
    sb.data += s
    return sb
}

func (sb *StringBuilder) AppendLine(s string) *StringBuilder {
    sb.data += s + "\n"
    return sb
}

func (sb *StringBuilder) ToUpper() *StringBuilder {
    sb.data = strings.ToUpper(sb.data)
    return sb
}

func (sb *StringBuilder) String() string {
    return sb.data
}

func main() {
    sb := &StringBuilder{}

    // Chain method calls
    result := sb.Append("Hello, ").
                Append("Go ").
                ToUpper().
                AppendLine("Programmer!").
                String()

    fmt.Println(result)
}
```

Method chaining:

- Creates fluent, readable APIs
- Requires returning the receiver (usually a pointer)
- Resembles builder patterns in other languages

## **10.3 Interfaces: Defining Behavior**

### **10.3.1 Interface Fundamentals**

An interface in Go is a type that defines a set of method signatures. Any type that implements all the methods of an interface implicitly satisfies that interface.

```go
package main

import "fmt"

// Define an interface
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Types that implement the interface
type Rectangle struct {
    Width, Height float64
}

func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14159 * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * 3.14159 * c.Radius
}

// Function that accepts any Shape
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f\n", s.Area())
    fmt.Printf("Perimeter: %.2f\n", s.Perimeter())
}

func main() {
    rect := Rectangle{Width: 5, Height: 4}
    circ := Circle{Radius: 3}

    fmt.Println("Rectangle:")
    PrintShapeInfo(rect)

    fmt.Println("\nCircle:")
    PrintShapeInfo(circ)
}
```

Key characteristics of Go interfaces:

- **Implicit implementation**: Types satisfy interfaces automatically by implementing the required methods
- **Focus on behavior**: Interfaces define what a type can do, not what it is
- **Decoupling**: Code depends on behaviors rather than concrete implementations
- **Composition**: Interfaces can be composed of other interfaces

### **10.3.2 The Empty Interface**

The empty interface, `interface{}` or `any` (in Go 1.18+), has no methods and is satisfied by every type:

```go
package main

import "fmt"

// Function that accepts any value
func PrintAny(v interface{}) {
    fmt.Printf("Value: %v, Type: %T\n", v, v)
}

func main() {
    PrintAny(42)                // int
    PrintAny("Hello")           // string
    PrintAny(true)              // bool
    PrintAny([]string{"a", "b"}) // slice
    PrintAny(struct{Name string}{"John"}) // struct
}
```

Common uses of the empty interface:

- Generic data structures (before Go 1.18 generics)
- Functions that need to accept any type
- Working with JSON and other formats that require type flexibility

### **10.3.3 Type Assertions and Type Switches**

When working with interfaces, you often need to access the underlying concrete type:

```go
package main

import "fmt"

func main() {
    // Type assertions
    var i interface{} = "hello"

    // Type assertion with single return value
    s := i.(string)
    fmt.Println(s) // hello

    // Type assertion with check (safer)
    if s, ok := i.(string); ok {
        fmt.Println("String value:", s)
    }

    // This would panic:
    // n := i.(int)

    // Type switch
    var value interface{} = 42

    switch v := value.(type) {
    case string:
        fmt.Println("String:", v)
    case int:
        fmt.Println("Integer:", v)
    case bool:
        fmt.Println("Boolean:", v)
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}
```

Working with concrete types:

- Type assertions extract the underlying value from an interface
- The `value.(Type)` syntax may panic if the assertion fails
- The `value, ok := i.(Type)` pattern safely checks for type compatibility
- Type switches provide a cleaner way to handle multiple possible types

### **10.3.4 Common Interfaces in the Standard Library**

Go's standard library defines several useful interfaces:

```go
package main

import (
    "bytes"
    "fmt"
    "io"
    "strings"
)

func main() {
    // io.Reader and io.Writer examples
    r := strings.NewReader("Hello, Readers and Writers!")
    w := &bytes.Buffer{}

    // Copy from reader to writer
    bytesWritten, err := io.Copy(w, r)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Printf("Copied %d bytes\n", bytesWritten)
    fmt.Printf("Buffer contents: %s\n", w.String())

    // fmt.Stringer interface
    type Person struct {
        Name string
        Age  int
    }

    // Implement fmt.Stringer for custom string representation
    func (p Person) String() string {
        return fmt.Sprintf("%s (%d years)", p.Name, p.Age)
    }

    person := Person{Name: "Alice", Age: 30}
    fmt.Println("Person:", person) // Uses String() method
}
```

Important standard library interfaces:

- `io.Reader` and `io.Writer`: For data streaming operations
- `fmt.Stringer`: For custom string representations (`String()` method)
- `error`: For error handling (`Error()` method)
- `sort.Interface`: For custom sorting implementations

### **10.3.5 Interface Composition**

Interfaces can be composed of other interfaces:

```go
package main

import (
    "fmt"
    "io"
)

// Define smaller interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Compose interfaces
type ReadWriter interface {
    Reader
    Writer
}

// Implement a concrete type
type Buffer struct {
    data []byte
}

func (b *Buffer) Read(p []byte) (n int, err error) {
    if len(b.data) == 0 {
        return 0, io.EOF
    }
    n = copy(p, b.data)
    b.data = b.data[n:]
    return n, nil
}

func (b *Buffer) Write(p []byte) (n int, err error) {
    b.data = append(b.data, p...)
    return len(p), nil
}

func main() {
    // Create a buffer that satisfies ReadWriter
    var rw ReadWriter = &Buffer{}

    // Write data
    rw.Write([]byte("Hello, interface composition!"))

    // Read data
    data := make([]byte, 10)
    n, _ := rw.Read(data)

    fmt.Printf("Read %d bytes: %s\n", n, data[:n])
}
```

Benefits of interface composition:

- Defines capabilities in small, focused interfaces
- Combines interfaces to build more complex behaviors
- Follows the interface segregation principle

## **10.4 Practical Design Patterns with Structs, Methods, and Interfaces**

### **10.4.1 Builder Pattern**

The Builder pattern allows step-by-step construction of complex objects:

```go
package main

import "fmt"

// Product
type House struct {
    floors      int
    doors       int
    windows     int
    hasGarage   bool
    hasSwimPool bool
}

// Builder
type HouseBuilder struct {
    house House
}

func NewHouseBuilder() *HouseBuilder {
    return &HouseBuilder{house: House{floors: 1, doors: 1, windows: 1}}
}

func (b *HouseBuilder) SetFloors(floors int) *HouseBuilder {
    b.house.floors = floors
    return b
}

func (b *HouseBuilder) SetDoors(doors int) *HouseBuilder {
    b.house.doors = doors
    return b
}

func (b *HouseBuilder) SetWindows(windows int) *HouseBuilder {
    b.house.windows = windows
    return b
}

func (b *HouseBuilder) WithGarage() *HouseBuilder {
    b.house.hasGarage = true
    return b
}

func (b *HouseBuilder) WithSwimPool() *HouseBuilder {
    b.house.hasSwimPool = true
    return b
}

func (b *HouseBuilder) Build() House {
    return b.house
}

func main() {
    house := NewHouseBuilder().
        SetFloors(2).
        SetDoors(3).
        SetWindows(10).
        WithGarage().
        Build()

    fmt.Printf("House: %+v\n", house)
}
```

### **10.4.2 Strategy Pattern**

The Strategy pattern defines a family of algorithms, encapsulates each one, and makes them interchangeable:

```go
package main

import "fmt"

// Strategy interface
type PaymentStrategy interface {
    Pay(amount float64) bool
}

// Concrete strategies
type CreditCardPayment struct {
    cardNumber string
    cvv        string
}

func (c CreditCardPayment) Pay(amount float64) bool {
    fmt.Printf("Paying %.2f using Credit Card %s\n", amount, c.cardNumber)
    return true
}

type PayPalPayment struct {
    email    string
    password string
}

func (p PayPalPayment) Pay(amount float64) bool {
    fmt.Printf("Paying %.2f using PayPal account %s\n", amount, p.email)
    return true
}

// Context
type ShoppingCart struct {
    items           []string
    paymentStrategy PaymentStrategy
}

func (sc *ShoppingCart) SetPaymentStrategy(ps PaymentStrategy) {
    sc.paymentStrategy = ps
}

func (sc *ShoppingCart) AddItem(item string) {
    sc.items = append(sc.items, item)
}

func (sc *ShoppingCart) Checkout(amount float64) bool {
    return sc.paymentStrategy.Pay(amount)
}

func main() {
    cart := &ShoppingCart{}
    cart.AddItem("Laptop")
    cart.AddItem("Mouse")

    // Use credit card payment
    cart.SetPaymentStrategy(CreditCardPayment{
        cardNumber: "1234-5678-9012-3456",
        cvv:        "123",
    })
    cart.Checkout(1299.99)

    // Use PayPal payment
    cart.SetPaymentStrategy(PayPalPayment{
        email:    "user@example.com",
        password: "password",
    })
    cart.Checkout(1299.99)
}
```

### **10.4.3 Repository Pattern**

The Repository pattern separates domain logic from data access logic:

```go
package main

import (
    "errors"
    "fmt"
)

// Entity
type User struct {
    ID    int
    Name  string
    Email string
}

// Repository interface
type UserRepository interface {
    FindByID(id int) (*User, error)
    Save(user *User) error
    Delete(id int) error
    FindAll() []*User
}

// In-memory implementation
type InMemoryUserRepository struct {
    users map[int]*User
    nextID int
}

func NewInMemoryUserRepository() *InMemoryUserRepository {
    return &InMemoryUserRepository{
        users:  make(map[int]*User),
        nextID: 1,
    }
}

func (r *InMemoryUserRepository) FindByID(id int) (*User, error) {
    user, exists := r.users[id]
    if !exists {
        return nil, errors.New("user not found")
    }
    return user, nil
}

func (r *InMemoryUserRepository) Save(user *User) error {
    if user.ID == 0 {
        user.ID = r.nextID
        r.nextID++
    }
    r.users[user.ID] = user
    return nil
}

func (r *InMemoryUserRepository) Delete(id int) error {
    if _, exists := r.users[id]; !exists {
        return errors.New("user not found")
    }
    delete(r.users, id)
    return nil
}

func (r *InMemoryUserRepository) FindAll() []*User {
    users := make([]*User, 0, len(r.users))
    for _, user := range r.users {
        users = append(users, user)
    }
    return users
}

// Service that uses the repository
type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) CreateUser(name, email string) (*User, error) {
    user := &User{Name: name, Email: email}
    err := s.repo.Save(user)
    return user, err
}

func main() {
    // Create repository and service
    repo := NewInMemoryUserRepository()
    service := NewUserService(repo)

    // Create users
    user1, _ := service.CreateUser("Alice", "alice@example.com")
    user2, _ := service.CreateUser("Bob", "bob@example.com")

    // Find user
    foundUser, _ := repo.FindByID(user1.ID)
    fmt.Printf("Found user: %+v\n", foundUser)

    // List all users
    allUsers := repo.FindAll()
    fmt.Println("All users:")
    for _, user := range allUsers {
        fmt.Printf("- %+v\n", user)
    }
}
```

## **10.5 Best Practices and Common Patterns**

### **10.5.1 Choosing Between Structs and Interfaces**

Guidelines for effective use of structs and interfaces:

1. **Use structs when**:

   - You need to model data with multiple fields
   - You want to add behavior specific to a data structure
   - You need to encapsulate related data

2. **Use interfaces when**:
   - You want to define behavior without specifying implementation
   - You need polymorphic behavior across different types
   - You want to decouple components of your system

### **10.5.2 Method Receiver Guidelines**

Choosing the right receiver type:

1. **Use value receivers when**:

   - The method doesn't modify the receiver
   - The receiver is a small, simple type
   - You want to emphasize immutability

2. **Use pointer receivers when**:
   - The method needs to modify the receiver
   - The receiver is a large struct (more efficient)
   - You need to maintain consistency with other methods on the same type

### **10.5.3 Interface Design Principles**

Guidelines for effective interface design:

1. **Keep interfaces small**: The most powerful interfaces often have just one or two methods
2. **Focus on behavior**: Define interfaces based on what types do, not what they are
3. **Accept interfaces, return structs**: Functions should accept interfaces and return concrete types
4. **Compose interfaces**: Build larger interfaces from smaller ones

### **10.5.4 Embedding vs. Composition**

When to use each approach:

1. **Use embedding when**:

   - You want the embedded type's methods and fields to be promoted
   - The relationship is "is-a" rather than "has-a"
   - You need direct access to embedded fields without indirection

2. **Use composition (fields) when**:
   - You want explicit control over access to the component's fields
   - The relationship is "has-a" rather than "is-a"
   - You need multiple instances of the same type

## **10.6 Exercises**

### **Exercise 1: Struct Basics**

Create a `Book` struct with fields for title, author, and publication year. Implement a method that returns the book's age.

### **Exercise 2: Interface Implementation**

Design a `Payable` interface with a `Pay()` method. Implement this interface for different payment types (Credit Card, PayPal, Cash).

### **Exercise 3: Embedded Structs**

Create a `Vehicle` struct and embed it in `Car` and `Truck` structs. Add methods specific to each type.

### **Exercise 4: Type Assertions**

Write a function that takes an empty interface parameter and uses type assertions to handle different types appropriately.

### **Exercise 5: Custom Data Structure**

Implement a simple stack data structure using a struct with methods for push, pop, and peek operations.

## **10.7 Summary**

In this chapter, we've explored the core building blocks of Go's approach to object-oriented programming:

- **Structs** provide a way to create custom data types by grouping related fields
- **Methods** add behavior to types, enabling an object-oriented programming style
- **Interfaces** define behavior contracts that types can implement implicitly
- **Embedding** offers a form of composition for creating more complex types

These features, combined with Go's emphasis on simplicity and explicitness, create a flexible system for organizing code that avoids many of the complexities of traditional class-based inheritance. By mastering structs, methods, and interfaces, you'll be able to design clean, maintainable Go programs that effectively model your problem domain.

**Next Up**: In Chapter 11, we'll dive into error handling in Go, exploring how to create, propagate, and handle errors effectively.
