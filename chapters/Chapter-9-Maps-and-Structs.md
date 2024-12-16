# **Chapter 9: Maps and Structs**

---

## **9.1. Maps**

### **What are Maps?**

A **map** in Go is an unordered collection of key-value pairs. Each key in the map is unique, and it maps to a value. Maps are useful when you need to quickly look up data or associate one piece of data with another. Unlike arrays or slices, maps don't maintain any order of elements, which makes them ideal for fast lookups, additions, and deletions.

Maps are reference types, which means they are passed by reference. If you pass a map to a function, the changes you make inside that function will affect the original map.

### **9.1.1 Declaring and Initializing Maps**

You can declare a map in Go using the `map` keyword followed by the key type and value type. Here's how you declare and initialize a map with some data:

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

**Explanation:**

- In the above example, a map `fruits` is initialized with string keys and string values.
- We access the value associated with the key "a" and print it.

**Output:**

```
Map: map[a:Apple b:Banana c:Cherry]
Value for key 'a': Apple
```

---

### **9.1.2 Adding, Updating, and Deleting Elements**

Maps in Go are dynamic, meaning you can add, update, or delete elements at any time. Here's an example demonstrating these operations:

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

**Explanation:**

- First, we add a new key-value pair for "c".
- Next, we update the value for the key "b" from "Banana" to "Blueberry".
- Finally, we delete the key-value pair for "a".

**Output:**

```
Updated Map: map[b:Blueberry c:Cherry]
```

---

### **9.1.3 Checking if a Key Exists**

You can check whether a key exists in a map by using the second return value when accessing a map element. If the key exists, the second value will be `true`; otherwise, it will be `false`.

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

**Explanation:**

- We attempt to access the key "c" and check if it exists.
- Since "c" does not exist, we print a message indicating this.

**Output:**

```
Key 'c' does not exist
```

---

### **9.1.4 Iterating Over a Map**

You can use a `for` loop with the `range` keyword to iterate over the key-value pairs in a map. Here's an example:

```go
package main

import "fmt"

func main() {
    fruits := map[string]string{"a": "Apple", "b": "Banana", "c": "Cherry"}

    for key, value := range fruits {
        fmt.Printf("Key: %s, Value: %s
", key, value)
    }
}
```

**Explanation:**

- The `for` loop iterates through each key-value pair in the `fruits` map.
- We print both the key and the value during each iteration.

**Output:**

```
Key: a, Value: Apple
Key: b, Value: Banana
Key: c, Value: Cherry
```

---

### **9.1.5 Handling Maps with Zero Values**

In Go, when you try to access a map with a key that doesn't exist, it returns the zero value for the map's value type. This is useful, but you should be careful to check if the key actually exists.

```go
package main

import "fmt"

func main() {
    fruits := map[string]string{"a": "Apple", "b": "Banana"}

    // Accessing a non-existent key will return the zero value for string, which is an empty string.
    fmt.Println(fruits["c"]) // Output: ""
}
```

---

## **9.2. Structs**

### **What are Structs?**

A **struct** is a composite data type in Go that groups together variables (fields) under a single name. Each field in a struct can have a different type. Structs are commonly used to define complex data models that represent real-world entities. Structs are the foundation of custom types in Go and are critical for building robust applications.

Unlike arrays and slices, structs can hold fields of different types, making them extremely versatile.

### **9.2.1 Declaring and Initializing Structs**

You can define a struct by using the `type` keyword followed by the name of the struct and the fields it will contain. Here's how to declare and initialize a simple struct:

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

**Explanation:**

- We define a struct `Person` with fields `Name` and `Age`.
- Then, we initialize a `Person` struct with the name "John" and age 30.

**Output:**

```
Person Struct: {John 30}
Name: John
Age: 30
```

---

### **9.2.2 Using Pointers with Structs**

In Go, structs are passed by value, meaning when you assign a struct to another variable, a copy of the struct is created. However, you can use pointers to directly modify the struct's fields without creating a copy.

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

**Explanation:**

- We create a pointer to the `Person` struct and modify the `Age` field directly.

**Output:**

```
Updated Person Struct: &{Alice 26}
```

---

### **9.2.3 Structs with Nested Fields**

Structs can also contain other structs as fields, creating a hierarchy of data. Here's an example of a struct with nested fields:

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

**Explanation:**

- The `Person` struct contains an `Address` field, which itself is a struct.
- We initialize both the `Person` and `Address` structs and print the city.

**Output:**

```
Person Struct: {Emily 35 {Los Angeles CA}}
City: Los Angeles
```

---

### **9.2.4 Anonymous Structs**

Go also allows the use of anonymous structs, which are structs that do not have a defined name but can still be used inline.

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

**Explanation:**

- In this example, we define and initialize an anonymous struct without a type name.

**Output:**

```
Anonymous Struct: {Mia 29}
```

---

### **9.2.5 Methods on Structs**

You can define methods on structs, which allows you to associate behavior with data. Here's an example where we define a method for the `Person` struct:

```go
package main

import "fmt"

// Define the struct
type Person struct {
    Name string
    Age  int
}

// Method to greet a person
func (p Person) Greet() {
    fmt.Println("Hello, my name is", p.Name)
}

func main() {
    person := Person{Name: "John", Age: 30}
    person.Greet()
}
```

**Explanation:**

- The `Greet` method is defined on the `Person` struct.
- We create an instance of `Person` and call the `Greet` method.

**Output:**

```
Hello, my name is John
```

---

## **9.3. Practical Example: Combining Maps and Structs**

Maps and structs can be combined to store complex data in a simple and organized way. Here's an example that uses a map to store `Student` structs:

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
        fmt.Printf("ID: %s, Name: %s, Grade: %d
", id, student.Name, student.Grade)
    }
}
```

**Explanation:**

- We use a map where the key is a student ID, and the value is a `Student` struct containing their name and grade.
- We iterate over the map and print each student's information.

**Output:**

```
ID: 101, Name: Alice, Grade: 90
ID: 102, Name: Bob, Grade: 85
```

---

## **9.4. Summary**

In this chapter, you learned:

- How to use **maps** for efficient key-value storage and manipulation.
- How to define and work with **structs** to model complex data, including advanced usage such as pointers and methods.
- Practical applications of combining maps and structs to represent real-world scenarios.

# **9.5. Exercises**

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

## **Exercise 11: Adding Methods to Structs**

**Problem**: Create a `Person` struct with a method `Introduce` that prints the name and age of the person.

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

// Method for the Person struct
func (p Person) Introduce() {
    fmt.Printf("Hi, I'm %s and I'm %d years old.
", p.Name, p.Age)
}

func main() {
    person := Person{Name: "John", Age: 30}
    person.Introduce()
}
```

**Output:**

```
Hi, I'm John and I'm 30 years old.
```

---
