# **Chapter 8: Maps and Structs**

---

## **8.1. Maps**

A **map** in Go is an unordered collection of key-value pairs, where each key maps to a value. Maps are incredibly useful for quick lookups and organizing related data.

### **1.1 Declaring and Initializing Maps**

```go
package main

import "fmt"

func main() {
    // Declare and initialize a map
    fruits := map[string]string{
        "a": "Apple",
        "b": "Banana",
        "c": "Cherry",
    }

    // Access and print values
    fmt.Println("Map:", fruits)
    fmt.Println("Value for key 'a':", fruits["a"])
}
```

**Output:**

```
Map: map[a:Apple b:Banana c:Cherry]
Value for key 'a': Apple
```

### **8.1.2 Adding, Updating, and Deleting Elements**

```go
package main

import "fmt"

func main() {
    fruits := map[string]string{"a": "Apple", "b": "Banana"}

    // Adding a new key-value pair
    fruits["c"] = "Cherry"

    // Updating an existing key-value pair
    fruits["b"] = "Blueberry"

    // Deleting a key-value pair
    delete(fruits, "a")

    fmt.Println("Updated Map:", fruits)
}
```

**Output:**

```
Updated Map: map[b:Blueberry c:Cherry]
```

### **8.1.3 Checking if a Key Exists**

```go
package main

import "fmt"

func main() {
    fruits := map[string]string{"a": "Apple", "b": "Banana"}

    // Check if a key exists
    if value, exists := fruits["c"]; exists {
        fmt.Println("Value:", value)
    } else {
        fmt.Println("Key 'c' does not exist")
    }
}
```

**Output:**

```
Key 'c' does not exist
```

### **8.1.4 Iterating Over a Map**

```go
package main

import "fmt"

func main() {
    fruits := map[string]string{"a": "Apple", "b": "Banana", "c": "Cherry"}

    for key, value := range fruits {
        fmt.Printf("Key: %s, Value: %s\n", key, value)
    }
}
```

**Output:**

```
Key: a, Value: Apple
Key: b, Value: Banana
Key: c, Value: Cherry
```

---

## **8.2. Structs**

A **struct** in Go is a composite data type that groups together fields (variables) under a single name. Structs are the backbone of custom types in Go and are essential for building complex applications.

### **2.1 Declaring and Initializing Structs**

```go
package main

import "fmt"

// Define a struct
type Person struct {
    Name string
    Age  int
}

func main() {
    // Initialize a struct
    person := Person{Name: "John", Age: 30}

    fmt.Println("Person Struct:", person)
    fmt.Println("Name:", person.Name)
    fmt.Println("Age:", person.Age)
}
```

**Output:**

```
Person Struct: {John 30}
Name: John
Age: 30
```

### **8.2.2 Using Pointers with Structs**

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    person := &Person{Name: "Alice", Age: 25}

    // Modify fields using pointers
    person.Age = 26

    fmt.Println("Updated Person Struct:", person)
}
```

**Output:**

```
Updated Person Struct: &{Alice 26}
```

### **8.2.3 Structs with Nested Fields**

```go
package main

import "fmt"

// Define nested structs
type Address struct {
    City  string
    State string
}

type Person struct {
    Name    string
    Age     int
    Address Address
}

func main() {
    person := Person{
        Name: "Emily",
        Age:  35,
        Address: Address{
            City:  "Los Angeles",
            State: "CA",
        },
    }

    fmt.Println("Person Struct:", person)
    fmt.Println("City:", person.Address.City)
}
```

**Output:**

```
Person Struct: {Emily 35 {Los Angeles CA}}
City: Los Angeles
```

### **8.2.4 Anonymous Structs**

```go
package main

import "fmt"

func main() {
    // Create and initialize an anonymous struct
    person := struct {
        Name string
        Age  int
    }{
        Name: "Mia",
        Age:  29,
    }

    fmt.Println("Anonymous Struct:", person)
}
```

**Output:**

```
Anonymous Struct: {Mia 29}
```

---

## **8.3. Practical Example: Combining Maps and Structs**

```go
package main

import "fmt"

// Define a struct
type Student struct {
    Name  string
    Grade int
}

func main() {
    // Map with struct values
    students := map[string]Student{
        "101": {Name: "Alice", Grade: 90},
        "102": {Name: "Bob", Grade: 85},
    }

    // Access and print student details
    for id, student := range students {
        fmt.Printf("ID: %s, Name: %s, Grade: %d\n", id, student.Name, student.Grade)
    }
}
```

**Output:**

```
ID: 101, Name: Alice, Grade: 90
ID: 102, Name: Bob, Grade: 85
```

---

## **8.4. Summary**

In this chapter, you learned:

- How to use **maps** for key-value data storage and manipulation.
- How to define and work with **structs** to create custom types.
- Practical applications of combining maps and structs for real-world scenarios.

# **8.5. Exercises**

---

## **Exercise 1: Counting Word Frequencies**

**Problem**: Write a function that takes a string and returns a map containing the frequency of each word.

```go
package main

import (
    "fmt"
    "strings"
)

func wordFrequency(text string) map[string]int {
    words := strings.Fields(text)
    freq := make(map[string]int)
    for _, word := range words {
        freq[word]++
    }
    return freq
}

func main() {
    text := "hello world hello Go"
    fmt.Println("Word Frequencies:", wordFrequency(text))
}
```

**Output:**

```
Word Frequencies: map[Go:1 hello:2 world:1]
```

---

## **Exercise 2: Updating a Map**

**Problem**: Create a map of student grades, add new entries, update existing grades, and delete a student.

```go
package main

import "fmt"

func main() {
    grades := map[string]int{"Alice": 85, "Bob": 90}

    // Add and update entries
    grades["Charlie"] = 75
    grades["Alice"] = 95

    // Delete an entry
    delete(grades, "Bob")

    fmt.Println("Updated Grades:", grades)
}
```

**Output:**

```
Updated Grades: map[Alice:95 Charlie:75]
```

---

## **Exercise 3: Check Key Existence**

**Problem**: Write a function to check if a key exists in a map.

```go
package main

import "fmt"

func keyExists(m map[string]int, key string) bool {
    _, exists := m[key]
    return exists
}

func main() {
    scores := map[string]int{"Alice": 100, "Bob": 85}
    fmt.Println("Key 'Alice' exists:", keyExists(scores, "Alice"))
    fmt.Println("Key 'Charlie' exists:", keyExists(scores, "Charlie"))
}
```

**Output:**

```
Key 'Alice' exists: true
Key 'Charlie' exists: false
```

---

## **Exercise 4: Struct Initialization and Access**

**Problem**: Create a `Book` struct with fields `Title`, `Author`, and `Year`, and initialize and print its values.

```go
package main

import "fmt"

type Book struct {
    Title  string
    Author string
    Year   int
}

func main() {
    book := Book{Title: "1984", Author: "George Orwell", Year: 1949}
    fmt.Println("Book Details:", book)
}
```

**Output:**

```
Book Details: {1984 George Orwell 1949}
```

---

## **Exercise 5: Nested Structs**

**Problem**: Create a `Car` struct with a nested `Engine` struct, and initialize and print its fields.

```go
package main

import "fmt"

type Engine struct {
    Horsepower int
    Type       string
}

type Car struct {
    Brand  string
    Model  string
    Engine Engine
}

func main() {
    car := Car{
        Brand: "Tesla",
        Model: "Model S",
        Engine: Engine{
            Horsepower: 1020,
            Type:       "Electric",
        },
    }
    fmt.Println("Car Details:", car)
}
```

**Output:**

```
Car Details: {Tesla Model S {1020 Electric}}
```

---

## **Exercise 6: Map of Structs**

**Problem**: Create a map of employee IDs to `Employee` structs and iterate through the map.

```go
package main

import "fmt"

type Employee struct {
    Name     string
    Position string
}

func main() {
    employees := map[string]Employee{
        "E001": {Name: "Alice", Position: "Manager"},
        "E002": {Name: "Bob", Position: "Developer"},
    }

    for id, emp := range employees {
        fmt.Printf("ID: %s, Name: %s, Position: %s
", id, emp.Name, emp.Position)
    }
}
```

**Output:**

```
ID: E001, Name: Alice, Position: Manager
ID: E002, Name: Bob, Position: Developer
```

---

## **Exercise 7: Anonymous Structs in Maps**

**Problem**: Create a map using anonymous structs as values.

```go
package main

import "fmt"

func main() {
    products := map[string]struct {
        Price    float64
        Quantity int
    }{
        "Laptop": {Price: 999.99, Quantity: 10},
        "Phone":  {Price: 599.99, Quantity: 20},
    }

    for name, details := range products {
        fmt.Printf("Product: %s, Price: %.2f, Quantity: %d
", name, details.Price, details.Quantity)
    }
}
```

**Output:**

```
Product: Laptop, Price: 999.99, Quantity: 10
Product: Phone, Price: 599.99, Quantity: 20
```

---

## **Exercise 8: Pointer to Struct**

**Problem**: Use a pointer to a struct to modify its fields.

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

func main() {
    person := &Person{Name: "Alice", Age: 25}
    person.Age = 26
    fmt.Println("Updated Person:", *person)
}
```

**Output:**

```
Updated Person: {Alice 26}
```

---

## **Exercise 9: Adding and Removing Map Entries**

**Problem**: Write a function to add and remove entries in a map.

```go
package main

import "fmt"

func modifyMap(m map[string]int) {
    m["new"] = 100
    delete(m, "old")
}

func main() {
    data := map[string]int{"old": 50, "existing": 75}
    modifyMap(data)
    fmt.Println("Modified Map:", data)
}
```

**Output:**

```
Modified Map: map[existing:75 new:100]
```

---

## **Exercise 10: Combining Maps and Structs**

**Problem**: Create a program that uses maps to store and retrieve `Student` struct data.

```go
package main

import "fmt"

type Student struct {
    Name  string
    Grade int
}

func main() {
    students := map[string]Student{
        "S001": {Name: "Alice", Grade: 90},
        "S002": {Name: "Bob", Grade: 85},
    }

    for id, student := range students {
        fmt.Printf("ID: %s, Name: %s, Grade: %d
", id, student.Name, student.Grade)
    }
}
```

**Output:**

```
ID: S001, Name: Alice, Grade: 90
ID: S002, Name: Bob, Grade: 85
```

---

**Congratulations!** These exercises cover a wide range of scenarios involving maps and structs, giving you a strong foundation for real-world Go programming.
