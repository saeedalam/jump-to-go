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

---

### **9.2.1 Declaring and Initializing Structs**

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

---

### **9.2.2 Using Pointers with Structs**

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

---

### **9.2.3 Structs with Nested Fields**

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

---

### **9.2.4 Anonymous Structs**

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

### **9.2.5 Methods on Structs**

In Go, methods are functions with a special receiver argument. Methods allow you to associate specific functionality or behavior with a type, like structs. This makes your code more modular and reusable.

Struct methods are particularly useful because they enable you to operate on the struct's data directly, creating a clean way to encapsulate logic.

---

## **Example: Greeting a Person**

Here’s an example demonstrating how to define and use a method on a struct:

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

---

## **Explanation**

### **1. Struct Definition**

```go
type Person struct {
    Name string
    Age  int
}
```

- The `Person` struct represents a person with a `Name` and `Age`.

### **2. Defining a Method**

```go
func (p Person) Greet() {
    fmt.Println("Hello, my name is", p.Name)
}
```

- The `Greet` method is tied to the `Person` struct through the receiver `(p Person)`.
- The receiver allows the method to access the struct's fields (`Name` in this case).

### **3. Using the Method**

```go
person := Person{Name: "John", Age: 30}
person.Greet()
```

- A `Person` instance is created with the name "John".
- The `Greet` method is called on this instance, printing a personalized greeting.

---

## **Output**

When you run the code, you will see the following output:

```
Hello, my name is John
```

---

## **Why Use Methods on Structs?**

- **Encapsulation:** Methods group related data and behavior together.
- **Clarity:** Improves code readability by associating actions with the relevant type.
- **Reusability:** Methods can be reused across multiple instances of the struct.

---

## **9.3. Practical Example: Combining Maps and Structs**

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

## **9.4. Summary**

In this chapter, you learned:

- How to use **maps** for efficient key-value storage and manipulation.
- How to define and work with **structs** to model complex data, including advanced usage such as pointers and methods.
- Practical applications of combining maps and structs to represent real-world scenarios.

# **9.5. Exercises**

---

## **Exercise 1: Counting Word Frequencies**

**Problem**: Write a function that takes a string and returns a map containing the frequency of each word.

### **Code**:

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

### **Explanation**:

1. The `strings.Fields` function splits the input text into words.
2. A map is created to store word frequencies, with each word as a key and its count as the value.
3. Each word in the text is iterated over, and the count is incremented in the map.

### **Output**:

```
Word Frequencies: map[Go:1 hello:2 world:1]
```

---

## **Exercise 2: Updating a Map**

**Problem**: Create a map of student grades, add new entries, update existing grades, and delete a student.

### **Code**:

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

### **Explanation**:

1. A map of grades is initialized with two entries.
2. A new entry for "Charlie" is added, and "Alice's" grade is updated.
3. The `delete` function removes "Bob" from the map.

### **Output**:

```
Updated Grades: map[Alice:95 Charlie:75]
```

---

## **Exercise 3: Check Key Existence**

**Problem**: Write a function to check if a key exists in a map.

### **Code**:

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

### **Explanation**:

1. The `keyExists` function uses the second return value of map access to determine key existence.
2. If the key exists, `exists` is `true`; otherwise, it’s `false`.

### **Output**:

```
Key 'Alice' exists: true
Key 'Charlie' exists: false
```

---

## **Exercise 4: Struct Initialization and Access**

**Problem**: Create a `Book` struct with fields `Title`, `Author`, and `Year`, and initialize and print its values.

### **Code**:

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

### **Explanation**:

1. A struct `Book` is defined with three fields.
2. An instance of `Book` is created with specific values and printed.

### **Output**:

```
Book Details: {1984 George Orwell 1949}
```

---

## **Exercise 5: Nested Structs**

**Problem**: Create a `Car` struct with a nested `Engine` struct, and initialize and print its fields.

### **Code**:

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

### **Explanation**:

1. The `Car` struct contains an `Engine` struct as a nested field.
2. Both structs are initialized with values and printed.

### **Output**:

```
Car Details: {Tesla Model S {1020 Electric}}
```

---

## **Exercise 6: Map of Structs**

**Problem**: Create a map of employee IDs to `Employee` structs and iterate through the map.

### **Code**:

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
        fmt.Printf("ID: %s, Name: %s, Position: %s\n", id, emp.Name, emp.Position)
    }
}
```

### **Explanation**:

1. A map of employee IDs to `Employee` structs is created.
2. A `for` loop iterates over the map to print each entry.

### **Output**:

```
ID: E001, Name: Alice, Position: Manager
ID: E002, Name: Bob, Position: Developer
```

---

## **Exercise 7: Anonymous Structs in Maps**

**Problem**: Create a map using anonymous structs as values.

### **Code**:

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
        fmt.Printf("Product: %s, Price: %.2f, Quantity: %d\n", name, details.Price, details.Quantity)
    }
}
```

### **Explanation**:

1. Anonymous structs are used as values in a map to store product details.
2. The map is iterated over to print product information.

### **Output**:

```
Product: Laptop, Price: 999.99, Quantity: 10
Product: Phone, Price: 599.99, Quantity: 20
```

---

## **Exercise 8: Pointer to Struct**

**Problem**: Use a pointer to a struct to modify its fields.

### **Code**:

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

### **Explanation**:

1. A pointer to the `Person` struct is created and used to modify the `Age` field.
2. The updated struct is printed.

### **Output**:

```
Updated Person: {Alice 26}
```

---

## **Exercise 9: Adding and Removing Map Entries**

**Problem**: Write a function to add and remove entries in a map.

### **Code**:

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

### **Explanation**:

1. The `modifyMap` function adds a new key-value pair and deletes an existing one.
2. The updated map is printed.

### **Output**:

```
Modified Map: map[existing:75 new:100]
```

---

## **Exercise 10: Combining Maps and Structs**

**Problem**: Create a program that uses maps to store and retrieve `Student` struct data.

### **Code**:

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
        fmt.Printf("ID: %s, Name: %s, Grade: %d\n", id, student.Name, student.Grade)
    }
}
```

### **Explanation**:

1. A map stores student IDs as keys and `Student` structs as values.
2. The map is iterated over to display each student's information.

### **Output**:

```
ID: S001, Name: Alice, Grade: 90
ID: S002, Name: Bob, Grade: 85
```

---

## **Exercise 11: Adding Methods to Structs**

**Problem**: Create a `Person` struct with a method `Introduce` that prints the name and age of the person.

### **Code**:

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}

// Method for the Person struct
func (p Person) Introduce() {
    fmt.Printf("Hi, I'm %s and I'm %d years old.\n", p.Name, p.Age)
}

func main() {
    person := Person{Name: "John", Age: 30}
    person.Introduce()
}
```

### **Explanation**:

1. The `Introduce` method is defined for the `Person` struct, which accesses its fields.
2. The method is called on an instance of `Person` to display the introduction.

### **Output**:

```
Hi, I'm John and I'm 30 years old.
```

---
