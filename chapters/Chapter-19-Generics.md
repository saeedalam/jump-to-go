# **Chapter 19: Generics in Go**

Go 1.18 introduced generics, a long-awaited feature that enables developers to write more reusable and type-safe code. This chapter explores generics in Go, from basic concepts to practical applications, helping you understand how to leverage this powerful feature in your projects.

## **19.1 Introduction to Generics**

Generics allow you to write functions and data structures that operate on different types while maintaining type safety. Before generics, Go developers had to choose between type-specific implementations (resulting in code duplication) or using empty interfaces (sacrificing type safety).

### **19.1.1 The Problem Generics Solve**

Consider these two functions that find the minimum value in a slice of integers or floats:

```go
func MinInt(values []int) int {
    if len(values) == 0 {
        panic("empty slice")
    }

    min := values[0]
    for _, v := range values[1:] {
        if v < min {
            min = v
        }
    }
    return min
}

func MinFloat64(values []float64) float64 {
    if len(values) == 0 {
        panic("empty slice")
    }

    min := values[0]
    for _, v := range values[1:] {
        if v < min {
            min = v
        }
    }
    return min
}
```

The logic is identical, but we need separate implementations for each type. With generics, we can write a single function that works with multiple types:

```go
func Min[T constraints.Ordered](values []T) T {
    if len(values) == 0 {
        panic("empty slice")
    }

    min := values[0]
    for _, v := range values[1:] {
        if v < min {
            min = v
        }
    }
    return min
}
```

### **19.1.2 Key Concepts in Go Generics**

1. **Type Parameters**: Placeholders for types that will be specified later
2. **Type Constraints**: Restrictions on what types can be used as type arguments
3. **Type Inference**: Go's ability to deduce type arguments from the context
4. **Type Sets**: A set of types that satisfy a constraint

## **19.2 Type Parameters and Constraints**

### **19.2.1 Basic Syntax for Type Parameters**

Type parameters are specified in square brackets after the function name:

```go
func FunctionName[T Constraint](param T) ReturnType {
    // Function body
}
```

Where:

- `T` is the type parameter (you can use any identifier)
- `Constraint` defines what types T can be
- `param T` means the parameter is of type T

### **19.2.2 Predefined Constraints**

Go's standard library provides several predefined constraints in the `constraints` package:

```go
package main

import (
    "constraints"
    "fmt"
)

func Add[T constraints.Integer | constraints.Float](a, b T) T {
    return a + b
}

func main() {
    fmt.Println(Add(5, 3))       // 8
    fmt.Println(Add(2.5, 3.7))   // 6.2
}
```

Common predefined constraints include:

- `any` (equivalent to `interface{}`)
- `comparable` (types that support `==` and `!=`)
- `constraints.Ordered` (types that support `<`, `<=`, `>`, `>=`)
- `constraints.Integer`, `constraints.Float`, `constraints.Complex`

### **19.2.3 Creating Custom Constraints**

You can define custom constraints using interface types:

```go
package main

import "fmt"

// Define a constraint for types that support addition
type Addable interface {
    int | int64 | float64 | string
}

func Concat[T Addable](a, b T) T {
    return a + b
}

func main() {
    fmt.Println(Concat(5, 3))             // 8
    fmt.Println(Concat(2.5, 3.7))         // 6.2
    fmt.Println(Concat("Hello, ", "Go!")) // "Hello, Go!"
}
```

### **19.2.4 Type Inference**

In many cases, Go can infer the type parameter from the arguments:

```go
package main

import (
    "constraints"
    "fmt"
)

func Max[T constraints.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func main() {
    // Type is inferred as int
    fmt.Println(Max(5, 3))     // 5

    // Type is inferred as float64
    fmt.Println(Max(2.5, 3.7)) // 3.7

    // Type is inferred as string
    fmt.Println(Max("abc", "def")) // "def"
}
```

## **19.3 Generic Functions**

Let's explore how to create and use generic functions in Go.

### **19.3.1 Basic Generic Functions**

Here's a simple generic function that prints any type of value:

```go
package main

import "fmt"

func Print[T any](value T) {
    fmt.Printf("Value: %v, Type: %T\n", value, value)
}

func main() {
    Print(42)                  // Value: 42, Type: int
    Print("Hello, Generics!")  // Value: Hello, Generics!, Type: string
    Print(true)                // Value: true, Type: bool
    Print(3.14)                // Value: 3.14, Type: float64
}
```

### **19.3.2 Generic Functions with Multiple Type Parameters**

Functions can have multiple type parameters:

```go
package main

import "fmt"

func Swap[T any](a, b T) (T, T) {
    return b, a
}

func Pair[T, U any](first T, second U) (T, U) {
    return first, second
}

func main() {
    a, b := Swap(10, 20)
    fmt.Println(a, b)  // 20 10

    name, age := Pair("Alice", 30)
    fmt.Println(name, age)  // Alice 30
}
```

### **19.3.3 Practical Generic Functions**

Let's implement some useful generic functions:

```go
package main

import (
    "constraints"
    "fmt"
)

// Find returns the first element that satisfies the predicate
func Find[T any](slice []T, predicate func(T) bool) (T, bool) {
    for _, v := range slice {
        if predicate(v) {
            return v, true
        }
    }
    var zero T
    return zero, false
}

// Filter returns all elements that satisfy the predicate
func Filter[T any](slice []T, predicate func(T) bool) []T {
    var result []T
    for _, v := range slice {
        if predicate(v) {
            result = append(result, v)
        }
    }
    return result
}

// Map transforms each element using a transformation function
func Map[T, U any](slice []T, transform func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = transform(v)
    }
    return result
}

func main() {
    numbers := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

    // Find the first even number
    even, found := Find(numbers, func(n int) bool {
        return n%2 == 0
    })
    fmt.Println("First even:", even, found)  // First even: 2 true

    // Filter even numbers
    evenNumbers := Filter(numbers, func(n int) bool {
        return n%2 == 0
    })
    fmt.Println("Even numbers:", evenNumbers)  // Even numbers: [2 4 6 8 10]

    // Map numbers to their squares
    squares := Map(numbers, func(n int) int {
        return n * n
    })
    fmt.Println("Squares:", squares)  // Squares: [1 4 9 16 25 36 49 64 81 100]
}
```

## **19.4 Generic Data Structures**

Generics really shine when creating reusable data structures.

### **19.4.1 Generic Structs**

Let's start with a simple generic pair structure:

```go
package main

import "fmt"

// Pair holds two values of potentially different types
type Pair[T, U any] struct {
    First  T
    Second U
}

func NewPair[T, U any](first T, second U) Pair[T, U] {
    return Pair[T, U]{First: first, Second: second}
}

func (p Pair[T, U]) Swap() Pair[U, T] {
    return Pair[U, T]{First: p.Second, Second: p.First}
}

func main() {
    p1 := NewPair(42, "answer")
    fmt.Printf("Original: %v\n", p1)  // Original: {42 answer}

    p2 := p1.Swap()
    fmt.Printf("Swapped: %v\n", p2)   // Swapped: {answer 42}
}
```

### **19.4.2 Generic Stack Implementation**

Here's a generic stack implementation:

```go
package main

import "fmt"

// Stack is a generic stack implementation
type Stack[T any] struct {
    elements []T
}

// Push adds an element to the top of the stack
func (s *Stack[T]) Push(value T) {
    s.elements = append(s.elements, value)
}

// Pop removes and returns the top element
func (s *Stack[T]) Pop() (T, bool) {
    if len(s.elements) == 0 {
        var zero T
        return zero, false
    }

    index := len(s.elements) - 1
    value := s.elements[index]
    s.elements = s.elements[:index]
    return value, true
}

// Peek returns the top element without removing it
func (s *Stack[T]) Peek() (T, bool) {
    if len(s.elements) == 0 {
        var zero T
        return zero, false
    }

    return s.elements[len(s.elements)-1], true
}

// Size returns the number of elements in the stack
func (s *Stack[T]) Size() int {
    return len(s.elements)
}

// IsEmpty returns true if the stack has no elements
func (s *Stack[T]) IsEmpty() bool {
    return len(s.elements) == 0
}

func main() {
    // Integer stack
    intStack := Stack[int]{}
    intStack.Push(10)
    intStack.Push(20)
    intStack.Push(30)

    fmt.Println("Stack size:", intStack.Size())  // Stack size: 3

    if val, ok := intStack.Peek(); ok {
        fmt.Println("Top value:", val)  // Top value: 30
    }

    for !intStack.IsEmpty() {
        if val, ok := intStack.Pop(); ok {
            fmt.Printf("Popped: %v\n", val)
        }
    }
    // Popped: 30
    // Popped: 20
    // Popped: 10

    // String stack
    stringStack := Stack[string]{}
    stringStack.Push("Go")
    stringStack.Push("is")
    stringStack.Push("awesome")

    for !stringStack.IsEmpty() {
        if val, ok := stringStack.Pop(); ok {
            fmt.Printf("Popped: %v\n", val)
        }
    }
    // Popped: awesome
    // Popped: is
    // Popped: Go
}
```

### **19.4.3 Generic Set Implementation**

Here's a generic set implementation:

```go
package main

import "fmt"

// Set is a generic set implementation
type Set[T comparable] struct {
    elements map[T]struct{}
}

// NewSet creates a new set
func NewSet[T comparable]() Set[T] {
    return Set[T]{elements: make(map[T]struct{})}
}

// Add adds an element to the set
func (s *Set[T]) Add(value T) {
    s.elements[value] = struct{}{}
}

// Remove removes an element from the set
func (s *Set[T]) Remove(value T) {
    delete(s.elements, value)
}

// Contains checks if the set contains the value
func (s *Set[T]) Contains(value T) bool {
    _, exists := s.elements[value]
    return exists
}

// Size returns the number of elements in the set
func (s *Set[T]) Size() int {
    return len(s.elements)
}

// Values returns all elements in the set as a slice
func (s *Set[T]) Values() []T {
    values := make([]T, 0, len(s.elements))
    for v := range s.elements {
        values = append(values, v)
    }
    return values
}

// Union returns a new set containing all elements from both sets
func (s *Set[T]) Union(other Set[T]) Set[T] {
    result := NewSet[T]()
    for v := range s.elements {
        result.Add(v)
    }
    for v := range other.elements {
        result.Add(v)
    }
    return result
}

// Intersection returns a new set containing common elements
func (s *Set[T]) Intersection(other Set[T]) Set[T] {
    result := NewSet[T]()
    for v := range s.elements {
        if other.Contains(v) {
            result.Add(v)
        }
    }
    return result
}

func main() {
    set1 := NewSet[int]()
    set1.Add(1)
    set1.Add(2)
    set1.Add(3)

    set2 := NewSet[int]()
    set2.Add(3)
    set2.Add(4)
    set2.Add(5)

    fmt.Println("Set1 contains 2:", set1.Contains(2))  // true
    fmt.Println("Set1 contains 4:", set1.Contains(4))  // false

    union := set1.Union(set2)
    fmt.Println("Union:", union.Values())  // [1 2 3 4 5] (order may vary)

    intersection := set1.Intersection(set2)
    fmt.Println("Intersection:", intersection.Values())  // [3]
}
```

## **19.5 Advanced Generic Patterns**

Let's explore some advanced patterns with generics in Go.

### **19.5.1 Type Constraints with Methods**

We can define constraints that require specific methods:

```go
package main

import "fmt"

// Stringer is an interface that has a String method
type Stringer interface {
    String() string
}

// ToString converts any Stringer to a string
func ToString[T Stringer](value T) string {
    return value.String()
}

// User implements Stringer
type User struct {
    Name string
    Age  int
}

func (u User) String() string {
    return fmt.Sprintf("%s (%d)", u.Name, u.Age)
}

func main() {
    user := User{Name: "Alice", Age: 30}
    fmt.Println(ToString(user))  // Alice (30)
}
```

### **19.5.2 Generic Type Constraints with Operators**

We can create constraints based on operations the type supports:

```go
package main

import (
    "constraints"
    "fmt"
)

// Sum calculates the sum of all elements in a slice
func Sum[T constraints.Integer | constraints.Float](values []T) T {
    var sum T
    for _, v := range values {
        sum += v
    }
    return sum
}

// Average calculates the average of elements in a slice
func Average[T constraints.Integer | constraints.Float](values []T) float64 {
    if len(values) == 0 {
        return 0
    }

    sum := Sum(values)
    return float64(sum) / float64(len(values))
}

func main() {
    ints := []int{1, 2, 3, 4, 5}
    floats := []float64{1.5, 2.5, 3.5, 4.5, 5.5}

    fmt.Println("Sum of ints:", Sum(ints))          // 15
    fmt.Println("Sum of floats:", Sum(floats))      // 17.5
    fmt.Println("Average of ints:", Average(ints))   // 3
    fmt.Println("Average of floats:", Average(floats)) // 3.5
}
```

### **19.5.3 Generic Function Composition**

We can compose generic functions to build more complex operations:

```go
package main

import (
    "fmt"
    "strings"
)

// Pipe composes two functions
func Pipe[A, B, C any](f func(A) B, g func(B) C) func(A) C {
    return func(a A) C {
        return g(f(a))
    }
}

func main() {
    // Define some simple functions
    toUpper := func(s string) string {
        return strings.ToUpper(s)
    }

    addExclamation := func(s string) string {
        return s + "!"
    }

    // Compose them
    emphasize := Pipe(toUpper, addExclamation)

    // Use the composed function
    result := emphasize("hello")
    fmt.Println(result)  // HELLO!
}
```

## **19.6 Best Practices for Using Generics**

### **19.6.1 When to Use Generics**

Generics are particularly useful for:

1. **Algorithms** that operate on multiple types
2. **Data structures** like trees, graphs, stacks, and queues
3. **Utility functions** for slices, maps, and channels
4. **Function adapters** and composition

However, generics should be used judiciously. If you only need a function to work with a specific type, don't make it generic.

### **19.6.2 Design Guidelines**

1. **Keep constraints simple**: Use predefined constraints when possible
2. **Design for callers**: Make your generic code easy to use
3. **Document constraints**: Explain what types work with your generic functions
4. **Prefer type inference**: Allow Go to infer types when possible
5. **Test with multiple types**: Ensure your code works with all supported types

### **19.6.3 Performance Considerations**

Generic functions may have slightly different performance characteristics than type-specific functions:

1. **Compile time**: Generics may increase compilation time
2. **Binary size**: Generic functions can increase binary size as the compiler generates specialized code
3. **Runtime performance**: Generic code is monomorphized (specialized for each type), so runtime performance is typically similar to type-specific code

## **19.7 Exercises**

### **Exercise 1: Implement a Generic Queue**

Create a generic queue implementation with Enqueue, Dequeue, and Peek operations.

```go
package main

import "fmt"

// Queue is a generic FIFO queue
type Queue[T any] struct {
    elements []T
}

// Enqueue adds an element to the back of the queue
func (q *Queue[T]) Enqueue(value T) {
    q.elements = append(q.elements, value)
}

// Dequeue removes and returns the front element
func (q *Queue[T]) Dequeue() (T, bool) {
    if len(q.elements) == 0 {
        var zero T
        return zero, false
    }

    value := q.elements[0]
    q.elements = q.elements[1:]
    return value, true
}

// Peek returns the front element without removing it
func (q *Queue[T]) Peek() (T, bool) {
    if len(q.elements) == 0 {
        var zero T
        return zero, false
    }

    return q.elements[0], true
}

// IsEmpty returns true if the queue has no elements
func (q *Queue[T]) IsEmpty() bool {
    return len(q.elements) == 0
}

// Size returns the number of elements in the queue
func (q *Queue[T]) Size() int {
    return len(q.elements)
}

func main() {
    queue := Queue[string]{}
    queue.Enqueue("first")
    queue.Enqueue("second")
    queue.Enqueue("third")

    fmt.Println("Queue size:", queue.Size()) // 3

    if value, ok := queue.Peek(); ok {
        fmt.Println("Front value:", value) // "first"
    }

    for !queue.IsEmpty() {
        if value, ok := queue.Dequeue(); ok {
            fmt.Println("Dequeued:", value)
        }
    }
    // Dequeued: first
    // Dequeued: second
    // Dequeued: third
}
```

### **Exercise 2: Implement Generic Sorting**

Create a generic function to sort a slice of any ordered type.

```go
package main

import (
    "constraints"
    "fmt"
)

// Sort sorts a slice of any ordered type
func Sort[T constraints.Ordered](slice []T) {
    // Simple bubble sort implementation
    n := len(slice)
    for i := 0; i < n-1; i++ {
        for j := 0; j < n-i-1; j++ {
            if slice[j] > slice[j+1] {
                slice[j], slice[j+1] = slice[j+1], slice[j]
            }
        }
    }
}

func main() {
    // Sort integers
    ints := []int{5, 2, 9, 1, 7, 3}
    Sort(ints)
    fmt.Println("Sorted ints:", ints) // [1 2 3 5 7 9]

    // Sort strings
    strings := []string{"banana", "apple", "cherry", "date"}
    Sort(strings)
    fmt.Println("Sorted strings:", strings) // [apple banana cherry date]

    // Sort floats
    floats := []float64{3.14, 1.41, 2.71, 1.73}
    Sort(floats)
    fmt.Println("Sorted floats:", floats) // [1.41 1.73 2.71 3.14]
}
```

### **Exercise 3: Implement a Generic Binary Search Tree**

Create a generic binary search tree implementation.

```go
package main

import (
    "constraints"
    "fmt"
)

// Node represents a node in the binary search tree
type Node[T constraints.Ordered] struct {
    Value T
    Left  *Node[T]
    Right *Node[T]
}

// BST is a generic binary search tree
type BST[T constraints.Ordered] struct {
    Root *Node[T]
}

// Insert adds a value to the tree
func (bst *BST[T]) Insert(value T) {
    if bst.Root == nil {
        bst.Root = &Node[T]{Value: value}
        return
    }

    insertNode(bst.Root, value)
}

// insertNode recursively inserts a value into the tree
func insertNode[T constraints.Ordered](node *Node[T], value T) {
    if value < node.Value {
        if node.Left == nil {
            node.Left = &Node[T]{Value: value}
        } else {
            insertNode(node.Left, value)
        }
    } else {
        if node.Right == nil {
            node.Right = &Node[T]{Value: value}
        } else {
            insertNode(node.Right, value)
        }
    }
}

// Contains checks if a value exists in the tree
func (bst *BST[T]) Contains(value T) bool {
    return contains(bst.Root, value)
}

// contains recursively checks if a value exists in the tree
func contains[T constraints.Ordered](node *Node[T], value T) bool {
    if node == nil {
        return false
    }

    if value == node.Value {
        return true
    } else if value < node.Value {
        return contains(node.Left, value)
    } else {
        return contains(node.Right, value)
    }
}

// InOrder returns the values in-order
func (bst *BST[T]) InOrder() []T {
    result := []T{}
    inOrder(bst.Root, &result)
    return result
}

// inOrder recursively traverses the tree in-order
func inOrder[T constraints.Ordered](node *Node[T], result *[]T) {
    if node != nil {
        inOrder(node.Left, result)
        *result = append(*result, node.Value)
        inOrder(node.Right, result)
    }
}

func main() {
    // Integer BST
    intTree := BST[int]{}
    intTree.Insert(5)
    intTree.Insert(3)
    intTree.Insert(7)
    intTree.Insert(2)
    intTree.Insert(4)

    fmt.Println("In-order traversal:", intTree.InOrder()) // [2 3 4 5 7]
    fmt.Println("Contains 4:", intTree.Contains(4)) // true
    fmt.Println("Contains 6:", intTree.Contains(6)) // false

    // String BST
    stringTree := BST[string]{}
    stringTree.Insert("banana")
    stringTree.Insert("apple")
    stringTree.Insert("cherry")

    fmt.Println("In-order traversal:", stringTree.InOrder()) // [apple banana cherry]
    fmt.Println("Contains apple:", stringTree.Contains("apple")) // true
    fmt.Println("Contains date:", stringTree.Contains("date")) // false
}
```

## **19.8 Summary**

Generics in Go enable developers to write more reusable and type-safe code. Key points from this chapter:

1. **Generics solve code duplication** by allowing functions and data structures to work with multiple types
2. **Type parameters and constraints** specify what types can be used with generic code
3. **Generic functions and data structures** enable type-safe, reusable code
4. **Advanced patterns** like function composition enhance code flexibility and reusability
5. **Use generics judiciously** for algorithms, data structures, and utility functions

Go's implementation of generics balances simplicity with power, staying true to Go's philosophy of clear, readable code. By adding generics to your Go toolkit, you'll be able to write more concise, reusable, and maintainable code.

**Next Chapter**: In Chapter 20, we'll explore building RESTful APIs in Go, bringing together many of the concepts we've learned throughout this book.
