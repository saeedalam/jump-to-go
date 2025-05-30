# **Chapter 9: Maps in Go**

Maps are one of Go's most versatile and powerful built-in data structures. They provide an efficient way to store and retrieve key-value pairs, making them essential for a wide range of programming tasks from simple lookups to complex data transformations.

In this chapter, we'll explore maps in depth—from basic operations to advanced techniques—and learn how to leverage their capabilities to write clean, efficient Go code.

## **9.1 Map Fundamentals**

### **9.1.1 What is a Map?**

A map is an unordered collection of key-value pairs where each key is unique. Maps provide fast lookups, insertions, and deletions based on keys. In Go, maps are implemented as hash tables, offering average constant-time complexity for these operations.

Key characteristics of Go maps:

- **Unordered**: Unlike arrays and slices, maps don't maintain insertion order
- **Dynamic**: Maps grow automatically as you add more key-value pairs
- **Reference Type**: Maps are passed by reference, not by value
- **Type Requirements**: Keys must be comparable (support the `==` and `!=` operators)
- **Zero Value**: The zero value of a map is `nil`

### **9.1.2 Creating Maps**

There are several ways to create maps in Go:

```go
package main

import "fmt"

func main() {
    // Method 1: Using make
    scores := make(map[string]int)

    // Method 2: Map literal (empty)
    ages := map[string]int{}

    // Method 3: Map literal with initial data
    population := map[string]int{
        "New York": 8804190,
        "Los Angeles": 3898747,
        "Chicago": 2746388,
    }

    fmt.Println("Scores:", scores)           // map[]
    fmt.Println("Ages:", ages)               // map[]
    fmt.Println("Population:", population)   // map[Chicago:2746388 Los Angeles:3898747 New York:8804190]
}
```

Important differences between these methods:

1. `make(map[KeyType]ValueType)` creates an initialized, empty map
2. `map[KeyType]ValueType{}` also creates an initialized, empty map
3. A `nil` map (declared but not initialized) cannot store key-value pairs

```go
var nilMap map[string]int        // Nil map
nilMap["key"] = 10               // Runtime panic: assignment to entry in nil map
```

### **9.1.3 Basic Map Operations**

Let's explore the fundamental operations you can perform with maps:

**Inserting and Updating Key-Value Pairs**

```go
package main

import "fmt"

func main() {
    users := make(map[int]string)

    // Adding new key-value pairs
    users[1] = "Alice"
    users[2] = "Bob"

    fmt.Println("Users:", users)  // map[1:Alice 2:Bob]

    // Updating an existing value
    users[1] = "Alicia"
    fmt.Println("Updated users:", users)  // map[1:Alicia 2:Bob]
}
```

**Retrieving Values**

```go
package main

import "fmt"

func main() {
    colors := map[string]string{
        "red": "#FF0000",
        "green": "#00FF00",
        "blue": "#0000FF",
    }

    // Simple retrieval
    redHex := colors["red"]
    fmt.Println("Red hex code:", redHex)  // #FF0000

    // Retrieving a non-existent key returns the zero value
    yellowHex := colors["yellow"]
    fmt.Println("Yellow hex code:", yellowHex)  // Empty string (zero value for string)
}
```

**Checking for Key Existence**

```go
package main

import "fmt"

func main() {
    users := map[string]int{
        "alice": 25,
        "bob": 30,
    }

    // The "comma ok" idiom
    age, exists := users["alice"]
    if exists {
        fmt.Printf("Alice is %d years old\n", age)
    } else {
        fmt.Println("Alice not found")
    }

    // Checking a non-existent key
    age, exists = users["charlie"]
    if exists {
        fmt.Printf("Charlie is %d years old\n", age)
    } else {
        fmt.Println("Charlie not found")
    }
}
```

**Deleting Key-Value Pairs**

```go
package main

import "fmt"

func main() {
    inventory := map[string]int{
        "apple": 15,
        "banana": 8,
        "orange": 12,
    }

    fmt.Println("Initial inventory:", inventory)

    // Delete a key-value pair
    delete(inventory, "banana")
    fmt.Println("After deletion:", inventory)

    // Deleting a non-existent key is a no-op (doesn't cause errors)
    delete(inventory, "grape")
    fmt.Println("After deleting non-existent key:", inventory)
}
```

### **9.1.4 Iterating Over Maps**

You can iterate through all key-value pairs in a map using the `range` keyword:

```go
package main

import "fmt"

func main() {
    capitals := map[string]string{
        "France": "Paris",
        "Japan": "Tokyo",
        "India": "New Delhi",
        "Brazil": "Brasília",
    }

    // Iterating over keys and values
    fmt.Println("Countries and their capitals:")
    for country, capital := range capitals {
        fmt.Printf("%s: %s\n", country, capital)
    }

    // Iterating over just the keys
    fmt.Println("\nList of countries:")
    for country := range capitals {
        fmt.Println(country)
    }
}
```

Note: The iteration order of a map is not guaranteed. Each iteration might produce a different order of keys and values. If you need a specific order, you should sort the keys separately.

## **9.2 Advanced Map Techniques**

### **9.2.1 Maps with Complex Types**

Maps can have complex types for both keys and values:

**Structs as Map Values**

```go
package main

import "fmt"

type Employee struct {
    Name  string
    Title string
    Salary float64
}

func main() {
    employees := map[string]Employee{
        "E001": {Name: "Alice Johnson", Title: "Software Engineer", Salary: 85000},
        "E002": {Name: "Bob Smith", Title: "Product Manager", Salary: 95000},
    }

    // Accessing a struct field in a map value
    fmt.Printf("%s is a %s\n", employees["E001"].Name, employees["E001"].Title)

    // Updating a struct field
    employee := employees["E002"]
    employee.Salary += 5000
    employees["E002"] = employee  // Map values are not addressable directly

    fmt.Printf("%s's new salary: $%.2f\n", employees["E002"].Name, employees["E002"].Salary)
}
```

**Maps as Values in Other Maps (Nested Maps)**

```go
package main

import "fmt"

func main() {
    // Nested map for a university course catalog
    courseCatalog := map[string]map[string]string{
        "CS": {
            "CS101": "Introduction to Programming",
            "CS202": "Data Structures",
            "CS303": "Algorithms",
        },
        "MATH": {
            "MATH101": "Calculus I",
            "MATH202": "Linear Algebra",
        },
    }

    // Accessing values in nested maps
    fmt.Println("CS202:", courseCatalog["CS"]["CS202"])

    // Adding a new department
    courseCatalog["PHYS"] = map[string]string{
        "PHYS101": "Physics I",
    }

    // Adding a new course to an existing department
    courseCatalog["MATH"]["MATH303"] = "Differential Equations"

    // Printing the entire catalog
    for dept, courses := range courseCatalog {
        fmt.Printf("\nDepartment: %s\n", dept)
        for code, title := range courses {
            fmt.Printf("  %s: %s\n", code, title)
        }
    }
}
```

**Slices as Map Values**

```go
package main

import "fmt"

func main() {
    // Map with slices as values
    studentScores := map[string][]int{
        "Alice": {92, 87, 95},
        "Bob":   {85, 79, 91},
    }

    // Adding a new student
    studentScores["Charlie"] = []int{88, 92}

    // Adding a score to an existing student
    studentScores["Alice"] = append(studentScores["Alice"], 90)

    // Calculating averages
    fmt.Println("Average scores:")
    for student, scores := range studentScores {
        sum := 0
        for _, score := range scores {
            sum += score
        }
        avg := float64(sum) / float64(len(scores))
        fmt.Printf("%s: %.2f\n", student, avg)
    }
}
```

### **9.2.2 Maps with Function Values**

Maps can also store functions as values, creating a simple dispatch table:

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    // Map of mathematical operations
    operations := map[string]func(float64, float64) float64{
        "add":      func(a, b float64) float64 { return a + b },
        "subtract": func(a, b float64) float64 { return a - b },
        "multiply": func(a, b float64) float64 { return a * b },
        "divide":   func(a, b float64) float64 { return a / b },
        "power":    math.Pow,
    }

    // Using the function map
    a, b := 10.0, 2.0

    for op, fn := range operations {
        result := fn(a, b)
        fmt.Printf("%v %s %v = %v\n", a, op, b, result)
    }
}
```

### **9.2.3 Map Concurrency Considerations**

Maps in Go are not safe for concurrent use. If multiple goroutines access a map simultaneously and at least one of them is writing, you must implement synchronization:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    // A thread-safe map using a mutex
    type ConcurrentMap struct {
        mu sync.RWMutex
        data map[string]int
    }

    counter := ConcurrentMap{
        data: make(map[string]int),
    }

    // Thread-safe methods
    increment := func(key string) {
        counter.mu.Lock()
        defer counter.mu.Unlock()
        counter.data[key]++
    }

    getValue := func(key string) int {
        counter.mu.RLock()
        defer counter.mu.RUnlock()
        return counter.data[key]
    }

    // Increment a counter
    increment("visits")
    increment("visits")
    increment("logins")

    fmt.Println("Visits:", getValue("visits"))
    fmt.Println("Logins:", getValue("logins"))
}
```

For Go 1.9 and later, you can also use the `sync.Map` type, which is optimized for specific use cases:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var counter sync.Map

    // Store values
    counter.Store("visits", 0)

    // Increment a counter
    increment := func(key string) {
        var count int
        value, ok := counter.Load(key)
        if ok {
            count = value.(int)
        }
        counter.Store(key, count+1)
    }

    // Increment the counter
    increment("visits")
    increment("visits")
    increment("logins")

    // Retrieve values
    visits, _ := counter.Load("visits")
    logins, _ := counter.Load("logins")

    fmt.Println("Visits:", visits)
    fmt.Println("Logins:", logins)
}
```

## **9.3 Practical Map Applications**

Let's explore some practical applications of maps in real-world programming scenarios.

### **9.3.1 Word Frequency Counter**

Maps are perfect for counting occurrences of items:

```go
package main

import (
    "fmt"
    "strings"
)

func wordFrequency(text string) map[string]int {
    // Convert to lowercase and split into words
    words := strings.Fields(strings.ToLower(text))

    // Create a map to store word counts
    frequency := make(map[string]int)

    // Count word occurrences
    for _, word := range words {
        // Remove punctuation (simplified approach)
        word = strings.Trim(word, ".,!?;:()")
        if word != "" {
            frequency[word]++
        }
    }

    return frequency
}

func main() {
    text := "Go is an open source programming language. Go is expressive, concise, clean, and efficient."

    freq := wordFrequency(text)

    // Print in alphabetical order
    fmt.Println("Word frequencies:")
    for word, count := range freq {
        fmt.Printf("%-12s: %d\n", word, count)
    }
}
```

### **9.3.2 Caching and Memoization**

Maps are excellent for implementing caching mechanisms:

```go
package main

import (
    "fmt"
    "time"
)

// Expensive calculation function
func fibonacci(n int, cache map[int]int) int {
    // Check if result is already cached
    if val, found := cache[n]; found {
        fmt.Printf("Cache hit for fib(%d)\n", n)
        return val
    }

    fmt.Printf("Computing fib(%d)\n", n)

    // Base cases
    if n <= 1 {
        cache[n] = n
        return n
    }

    // Recursive calculation with cache
    result := fibonacci(n-1, cache) + fibonacci(n-2, cache)
    cache[n] = result
    return result
}

func main() {
    cache := make(map[int]int)

    start := time.Now()
    result := fibonacci(30, cache)
    duration := time.Since(start)

    fmt.Printf("fibonacci(30) = %d\n", result)
    fmt.Printf("Calculation took %v\n", duration)
    fmt.Printf("Cache contains %d entries\n", len(cache))
}
```

### **9.3.3 Grouping and Indexing Data**

Maps can be used to organize and index data for quick retrieval:

```go
package main

import "fmt"

type Student struct {
    ID    string
    Name  string
    Grade int
    Class string
}

func main() {
    students := []Student{
        {ID: "S001", Name: "Alice", Grade: 95, Class: "Mathematics"},
        {ID: "S002", Name: "Bob", Grade: 88, Class: "Physics"},
        {ID: "S003", Name: "Charlie", Grade: 92, Class: "Mathematics"},
        {ID: "S004", Name: "Diana", Grade: 85, Class: "Chemistry"},
        {ID: "S005", Name: "Eva", Grade: 91, Class: "Physics"},
    }

    // Index students by ID for quick lookup
    studentsByID := make(map[string]Student)
    for _, student := range students {
        studentsByID[student.ID] = student
    }

    // Group students by class
    studentsByClass := make(map[string][]Student)
    for _, student := range students {
        studentsByClass[student.Class] = append(studentsByClass[student.Class], student)
    }

    // Quick lookup by ID
    fmt.Println("Student S003:", studentsByID["S003"].Name)

    // Print students by class
    fmt.Println("\nStudents by class:")
    for class, classStudents := range studentsByClass {
        fmt.Printf("%s:\n", class)
        for _, student := range classStudents {
            fmt.Printf("  %s: %d%%\n", student.Name, student.Grade)
        }
    }
}
```

### **9.3.4 Set Implementation**

Go doesn't have a built-in set type, but maps can be used to implement sets efficiently:

```go
package main

import "fmt"

// Set implementation using a map
type Set map[string]struct{}

// Add adds an element to the set
func (s Set) Add(item string) {
    s[item] = struct{}{}
}

// Contains checks if an item is in the set
func (s Set) Contains(item string) bool {
    _, exists := s[item]
    return exists
}

// Remove removes an item from the set
func (s Set) Remove(item string) {
    delete(s, item)
}

// Items returns all items in the set
func (s Set) Items() []string {
    items := make([]string, 0, len(s))
    for item := range s {
        items = append(items, item)
    }
    return items
}

func main() {
    // Create a new set
    fruits := make(Set)

    // Add items to the set
    fruits.Add("apple")
    fruits.Add("banana")
    fruits.Add("orange")
    fruits.Add("apple")  // Duplicate, will be ignored

    // Check membership
    fmt.Println("Contains apple:", fruits.Contains("apple"))
    fmt.Println("Contains grape:", fruits.Contains("grape"))

    // Remove an item
    fruits.Remove("banana")

    // Get all items
    fmt.Println("Set items:", fruits.Items())
}
```

## **9.4 Performance Considerations**

Understanding map performance characteristics is crucial for writing efficient Go code.

### **9.4.1 Time Complexity**

Go maps have the following average time complexities:

- **Insertion**: O(1)
- **Deletion**: O(1)
- **Lookup**: O(1)

However, in the worst case (many hash collisions), these operations can degrade to O(n).

### **9.4.2 Memory Usage**

Maps in Go use more memory than the sum of their keys and values due to the hash table structure. Some considerations:

- Maps with many entries may cause memory pressure
- Deleting keys doesn't automatically reduce the map's memory footprint
- For very large maps that grow and shrink significantly, consider recreating the map periodically

### **9.4.3 Map Initialization with Expected Size**

For better performance when you know the approximate size of your map, you can use `make` with an initial capacity:

```go
// Create a map with space for approximately 1000 items
frequentWords := make(map[string]int, 1000)
```

This reduces the number of hash table resizes as the map grows.

### **9.4.4 Benchmark: Comparing Different Map Operations**

```go
package main

import (
    "fmt"
    "time"
)

func benchmarkMapOperations(size int) {
    // Creation
    start := time.Now()
    m := make(map[int]int, size)
    creationTime := time.Since(start)

    // Insertion
    start = time.Now()
    for i := 0; i < size; i++ {
        m[i] = i
    }
    insertionTime := time.Since(start)

    // Lookup
    start = time.Now()
    for i := 0; i < size; i++ {
        _ = m[i]
    }
    lookupTime := time.Since(start)

    // Deletion
    start = time.Now()
    for i := 0; i < size; i++ {
        delete(m, i)
    }
    deletionTime := time.Since(start)

    fmt.Printf("Map with %d elements:\n", size)
    fmt.Printf("  Creation:  %v\n", creationTime)
    fmt.Printf("  Insertion: %v\n", insertionTime)
    fmt.Printf("  Lookup:    %v\n", lookupTime)
    fmt.Printf("  Deletion:  %v\n", deletionTime)
}

func main() {
    benchmarkMapOperations(1000)
    benchmarkMapOperations(10000)
    benchmarkMapOperations(100000)
}
```

## **9.5 Best Practices for Working with Maps**

### **9.5.1 Map Design Guidelines**

1. **Choose appropriate key types**: Keys must be comparable. Good key types include:

   - Basic types (strings, integers, etc.)
   - Structs that only contain comparable types
   - Arrays of comparable types

2. **Avoid expensive operations in map keys**: If using structs as keys, keep them small and avoid fields that make equality checking expensive.

3. **Initialize maps properly**: Always initialize a map before use with `make` or a map literal.

4. **Check for key existence**: Use the "comma ok" idiom to explicitly check if a key exists.

5. **Handle nil maps**: Check if a map is nil before using it, or better yet, always initialize maps before use.

### **9.5.2 Common Map Pitfalls**

**Pitfall 1: Map Not Initialized**

```go
// Wrong
var m map[string]int
m["key"] = 1  // Panic: assignment to entry in nil map

// Right
m := make(map[string]int)
m["key"] = 1
```

**Pitfall 2: Not Checking Key Existence**

```go
// Wrong
score := scores["Alice"]  // If "Alice" doesn't exist, score gets the zero value
if score > 0 {  // This may be misleading if zero is a valid score
    // ...
}

// Right
score, exists := scores["Alice"]
if !exists {
    // Handle missing key
} else if score > 0 {
    // ...
}
```

**Pitfall 3: Direct Assignment to Map Value Fields**

```go
type Person struct {
    Name string
    Age  int
}

people := map[string]Person{
    "alice": {Name: "Alice", Age: 25},
}

// Wrong - This won't compile
// people["alice"].Age++

// Right
person := people["alice"]
person.Age++
people["alice"] = person
```

**Pitfall 4: Concurrent Map Access**

```go
// Wrong - May cause runtime panic
// go func() { m["key"] = 1 }()
// go func() { delete(m, "key") }()

// Right - Use mutex or sync.Map
var mu sync.Mutex
go func() {
    mu.Lock()
    m["key"] = 1
    mu.Unlock()
}()
```

### **9.5.3 Optimizing Map Usage**

1. **Pre-allocate when size is known**: Use `make(map[K]V, size)` to pre-allocate capacity.

2. **Clean up large maps**: If a map grows very large and then shrinks, consider creating a new map to reclaim memory.

3. **Use efficient key types**: Simple, compact keys are more efficient than complex ones.

4. **Consider alternatives for special cases**:
   - For integer keys with a small, dense range, slices may be more efficient
   - For sets of integers, consider using bit sets
   - For concurrent access patterns, evaluate `sync.Map`

## **9.6 Exercises**

### **Exercise 1: Word Frequency Analysis**

Implement a program that reads a text file, counts the frequency of each word, and prints the top 10 most common words.

### **Exercise 2: Implement a Cache**

Create a simple caching system with expiration times for entries using a map and time stamps.

### **Exercise 3: Student Grade Tracker**

Implement a program that tracks student grades across multiple subjects, calculates averages, and identifies top performers.

### **Exercise 4: Custom Map Keys**

Create a map that uses a custom struct as the key, implement proper equality checking, and demonstrate its usage.

### **Exercise 5: Set Operations**

Extend the Set implementation to support union, intersection, and difference operations between sets.

## **9.7 Summary**

In this chapter, we've explored the versatile map data structure in Go:

- **Map Fundamentals**: Creating maps, basic operations, and iteration
- **Advanced Techniques**: Complex map types, nested maps, and concurrent usage
- **Practical Applications**: Counting frequencies, caching, grouping data, and implementing sets
- **Performance Considerations**: Time complexity, memory usage, and optimization
- **Best Practices**: Design guidelines, avoiding common pitfalls, and efficient usage patterns

Maps are an essential tool in any Go programmer's toolkit. Their combination of fast lookups, flexible key-value storage, and ease of use makes them ideal for a wide range of programming tasks. By mastering maps and understanding their characteristics, you can write more efficient and elegant Go code.

**Next Up**: In Chapter 10, we'll explore structs, methods, and interfaces—the building blocks of Go's type system and object-oriented programming model.
