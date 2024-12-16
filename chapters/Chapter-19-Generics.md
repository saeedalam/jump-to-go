# **Chapter 19: Generics (Go 1.18+)**

---

## **19.1. Generic Functions**

### **Concept:**

Generics allow functions and data structures to operate on multiple types without duplicating code. Instead of writing separate implementations for different types, you can use a **type parameter** to create flexible, type-safe code. The type parameter is specified using square brackets (`[]`) after the function name.

### **Generic Function Syntax:**

A **generic function** is a function that can work with any type, and it is defined by adding a **type parameter**.

The syntax for defining a generic function:

```go
func FunctionName[T type](parameters) returnType
```

Where `T` is a type parameter, and `type` defines the type that can be used with the function.

---

### **Step-by-Step Example 1: A Generic `Print` Function**

Let's start by creating a simple generic function that prints any type of value.

```go
package main

import "fmt"

// Generic function to print any type
func Print[T any](value T) {
    fmt.Println(value)
}

func main() {
    Print("Hello, Generics!")   // String
    Print(42)                  // Integer
    Print(3.14)                // Float
}
```

**Explanation:**

- The function `Print` takes a single parameter `value` of type `T`, where `T` is a type parameter that can be any type.
- The `any` keyword allows `T` to be of any type (this is an alias for `interface{}` in Go).

**Output:**

```
Hello, Generics!
42
3.14
```

---

### **Step-by-Step Example 2: A Generic `Sum` Function**

Next, let's write a generic function that can calculate the sum of two values. We will constrain the type to `int` or `float64` types.

```go
package main

import "fmt"

// Generic function to calculate the sum of two values
func Sum[T int | float64](a, b T) T {
    return a + b
}

func main() {
    fmt.Println(Sum(10, 20))     // Integers
    fmt.Println(Sum(5.5, 2.3))  // Floats
}
```

**Explanation:**

- The type parameter `T` is constrained to be either `int` or `float64` using the `|` operator, ensuring the function works only with these types.
- We then sum the values of type `T`.

**Output:**

```
30
7.8
```

---

### **Step-by-Step Example 3: Constraints in Generics**

You can restrict type parameters to specific types using **constraints**. Go provides the `any` keyword (an alias for `interface{}`) and the `constraints` package for custom constraints.

In this example, we will define a function that finds the larger of two numbers, and we will use the `Ordered` constraint from the `constraints` package to ensure that the function works with types that support ordering.

```go
package main

import (
    "constraints"
    "fmt"
)

// Generic function to find the larger of two numbers
func Max[T constraints.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}

func main() {
    fmt.Println(Max(10, 20))   // Integers
    fmt.Println(Max(3.5, 2.1)) // Floats
    fmt.Println(Max("cat", "dog")) // Strings (lexicographical order)
}
```

**Explanation:**

- The `Max` function uses a type parameter `T` with the constraint `constraints.Ordered`, which means that `T` must be a type that supports comparison using `<`, `>`, `<=`, and `>=`.
- We then compare the two values and return the larger one.

**Output:**

```
20
3.5
dog
```

---

### **Step-by-Step Example 4: A Generic `Contains` Function for Slices**

Now, let's create a generic function that checks if a value exists in a slice. We will constrain the type parameter `T` to be **comparable**, which ensures that values in the slice can be compared.

```go
package main

import "fmt"

// Generic function to check if a slice contains a value
func Contains[T comparable](slice []T, value T) bool {
    for _, v := range slice {
        if v == value {
            return true
        }
    }
    return false
}

func main() {
    nums := []int{1, 2, 3, 4, 5}
    fmt.Println(Contains(nums, 3)) // true
    fmt.Println(Contains(nums, 6)) // false

    words := []string{"go", "is", "awesome"}
    fmt.Println(Contains(words, "is"))  // true
    fmt.Println(Contains(words, "not")) // false
}
```

**Explanation:**

- The `Contains` function uses a type parameter `T`, constrained to `comparable`, which means that values of type `T` can be compared using `==` or `!=`.
- The function iterates through the slice and checks if the value exists in the slice.

**Output:**

```
true
false
true
false
```

---

## **19.2. Generic Structs**

### **Concept:**

Just like generic functions, **generic structs** allow you to create reusable structures that can operate on multiple types. By using type parameters, you can define a struct that works with different data types.

---

### **Step-by-Step Example 5: A Generic Pair**

In this example, we create a generic struct `Pair` that holds a pair of values, each of which can be a different type.

```go
package main

import "fmt"

// Generic struct to hold a pair of values
type Pair[T, U any] struct {
    First  T
    Second U
}

func main() {
    intPair := Pair[int, int]{First: 1, Second: 2}
    fmt.Println(intPair)

    stringPair := Pair[string, string]{First: "hello", Second: "world"}
    fmt.Println(stringPair)

    mixedPair := Pair[string, float64]{First: "pi", Second: 3.14}
    fmt.Println(mixedPair)
}
```

**Explanation:**

- The `Pair` struct uses two type parameters: `T` for the first value and `U` for the second value.
- We create instances of `Pair` with different type combinations (e.g., `int`, `string`, `string` and `float64`).

**Output:**

```
{1 2}
{hello world}
{pi 3.14}
```

---

### **Step-by-Step Example 6: A Generic Stack**

Now let's implement a **generic stack** using a struct. The stack will work with any type of elements.

```go
package main

import "fmt"

// Generic Stack implementation
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() T {
    if len(s.items) == 0 {
        panic("stack is empty")
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item
}

func main() {
    intStack := Stack[int]{}
    intStack.Push(10)
    intStack.Push(20)
    fmt.Println(intStack.Pop()) // 20
    fmt.Println(intStack.Pop()) // 10

    stringStack := Stack[string]{}
    stringStack.Push("Go")
    stringStack.Push("Generics")
    fmt.Println(stringStack.Pop()) // Generics
    fmt.Println(stringStack.Pop()) // Go
}
```

**Explanation:**

- The `Stack` struct is generic and uses the type parameter `T` for the stack's items.
- The `Push` method adds an item to the stack, and the `Pop` method removes and returns the top item.

**Output:**

```
20
10
Generics
Go
```

---

### **Step-by-Step Example 7: A Generic Map**

In this final example, we create a generic map structure that works with any key-value pair.

```go
package main

import "fmt"

// Generic struct for a key-value map
type GenericMap[K comparable, V any] struct {
    items map[K]V
}

func NewGenericMap[K comparable, V any]() *GenericMap[K, V] {
    return &GenericMap[K, V]{items: make(map[K]V)}
}

func (m *GenericMap[K, V]) Add(key K, value V) {
    m.items[key] = value
}

func (m *GenericMap[K, V]) Get(key K) V {
    return m.items[key]
}

func main() {
    myMap := NewGenericMap[string, int]()
    myMap.Add("one", 1)
    myMap.Add("two", 2)
    fmt.Println(myMap.Get("one")) // 1
    fmt.Println(myMap.Get("two")) // 2
}
```

**Explanation:**

- The `GenericMap` struct takes two type parameters: `K` for the key type and `V` for the value type.
- We define methods to add and retrieve key-value pairs from the map.

**Output:**

```
1
2
```

---

## **19.3. Exercises**

---

### **Exercise 1: Generic Minimum**

**Problem**: Write a generic function `Min` that returns the smallest of two values.

```go
package main

import (
    "constraints"
    "fmt"
)

func Min[T constraints.Ordered](a, b T) T {
    if a < b {
        return a
    }
    return b
}

func main() {
    fmt.Println(Min(10, 20))      // Output: 10
    fmt.Println(Min(3.5, 2.8))   // Output: 2.8
    fmt.Println(Min("apple", "orange")) // Output: apple
}
```

**Output:**

```
10
2.8
apple
```

---

### **Exercise 2: Generic Maximum**

**Problem**: Extend the `Max` function to handle a slice of values and return the maximum.

```go
package main

import (
    "constraints"
    "fmt"
)

func MaxSlice[T constraints.Ordered](slice []T) T {
    max := slice[0]
    for _, v := range slice {
        if v > max {
            max = v
        }
    }
    return max
}

func main() {
    nums := []int{1, 5, 3, 9, 2}
    fmt.Println("Max:", MaxSlice(nums)) // Output: 9

    words := []string{"go", "is", "awesome"}
    fmt.Println("Max:", MaxSlice(words)) // Output: is
}
```

**Output:**

```
Max: 9
Max: is
```

---

### **Exercise 3: Generic Swap**

**Problem**: Write a generic function `Swap` that swaps two elements in a slice.

```go
package main

import "fmt"

func Swap[T any](slice []T, i, j int) {
    slice[i], slice[j] = slice[j], slice[i]
}

func main() {
    nums := []int{1, 2, 3, 4, 5}
    Swap(nums, 1, 3)
    fmt.Println(nums) // Output: [1 4 3 2 5]

    words := []string{"hello", "world"}
    Swap(words, 0, 1)
    fmt.Println(words) // Output: [world hello]
}
```

**Output:**

```
[1 4 3 2 5]
[world hello]
```

---

### **Exercise 4: Generic Filter**

**Problem**: Create a generic `Filter` function to filter elements in a slice based on a condition.

```go
package main

import "fmt"

func Filter[T any](slice []T, condition func(T) bool) []T {
    result := []T{}
    for _, v := range slice {
        if condition(v) {
            result = append(result, v)
        }
    }
    return result
}

func main() {
    nums := []int{1, 2, 3, 4, 5}
    even := Filter(nums, func(n int) bool { return n%2 == 0 })
    fmt.Println(even) // Output: [2 4]
}
```

**Output:**

```
[2 4]
```

---

### **Exercise 5: Generic Map**

**Problem**: Implement a `Map` function that applies a transformation to each element in a slice.

```go
package main

import "fmt"

func Map[T any, U any](slice []T, transform func(T) U) []U {
    result := make([]U, len(slice))
    for i, v := range slice {
        result[i] = transform(v)
    }
    return result
}

func main() {
    nums := []int{1, 2, 3, 4}
    squares := Map(nums, func(n int) int { return n * n })
    fmt.Println(squares) // Output: [1 4 9 16]
}
```

**Output:**

```
[1 4 9 16]
```

---

### **Exercise 6: Generic Stack**

**Problem**: Build a generic stack with `Push`, `Pop`, and `Peek` operations.

```go
package main

import "fmt"

type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(item T) {
    s.items = append(s.items, item)
}

func (s *Stack[T]) Pop() T {
    if len(s.items) == 0 {
        panic("stack is empty")
    }
    item := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return item
}

func (s *Stack[T]) Peek() T {
    return s.items[len(s.items)-1]
}

func main() {
    stack := Stack[int]{}
    stack.Push(10)
    stack.Push(20)
    fmt.Println(stack.Peek()) // Output: 20
    fmt.Println(stack.Pop())  // Output: 20
}
```

**Output:**

```
20
20
```

---

### **Exercise 7: Generic Key-Value Store**

**Problem**: Implement a key-value store using generics.

```go
package main

import "fmt"

type KeyValueStore[K comparable, V any] struct {
    data map[K]V
}

func NewKeyValueStore[K comparable, V any]() *KeyValueStore[K, V] {
    return &KeyValueStore[K, V]{data: make(map[K]V)}
}

func (kv *KeyValueStore[K, V]) Set(key K, value V) {
    kv.data[key] = value
}

func (kv *KeyValueStore[K, V]) Get(key K) V {
    return kv.data[key]
}

func main() {
    store := NewKeyValueStore[string, int]()
    store.Set("a", 1)
    store.Set("b", 2)
    fmt.Println(store.Get("a")) // Output: 1
}
```

**Output:**

```
1
```

---

### **Exercise 8: Generic Pair**

**Problem**: Implement a generic pair struct with methods.

```go
package main

import "fmt"

type Pair[T, U any] struct {
    First  T
    Second U
}

func (p Pair[T, U]) Swap() Pair[U, T] {
    return Pair[U, T]{First: p.Second, Second: p.First}
}

func main() {
    p := Pair[int, string]{First: 1, Second: "one"}
    fmt.Println(p)
    fmt.Println(p.Swap())
}
```

**Output:**

```
{1 one}
{one 1}
```

---

### **Exercise 9: Generic Linked List**

**Problem**: Implement a generic linked list with `Insert` and `Delete` operations.

```go
package main

import "fmt"

type Node[T any] struct {
    value T
    next  *Node[T]
}

type LinkedList[T any] struct {
    head *Node[T]
}

func (l *LinkedList[T]) Insert(value T) {
    node := &Node[T]{value: value}
    if l.head == nil {
        l.head = node
    } else {
        current := l.head
        for current.next != nil {
            current = current.next
        }
        current.next = node
    }
}

func (l *LinkedList[T]) Delete(value T) {
    if l.head == nil {
        return
    }
    if l.head.value == value {
        l.head = l.head.next
        return
    }
    current := l.head
    for current.next != nil {
        if current.next.value == value {
            current.next = current.next.next
            return
        }
        current = current.next
    }
}

func main() {
    list := &LinkedList[int]{}
    list.Insert(1)
    list.Insert(2)
    list.Insert(3)
    list.Delete(2)
    fmt.Println(list.head.value) // Output: 1
    fmt.Println(list.head.next.value) // Output: 3
}
```

**Output:**

```
1
3
```

---

### **Exercise 10: Generic Binary Search**

**Problem**: Implement a generic binary search function.

```go
package main

import "fmt"

func BinarySearch[T constraints.Ordered](slice []T, target T) int {
    low, high := 0, len(slice)-1
    for low <= high {
        mid := low + (high-low)/2
        if slice[mid] == target {
            return mid
        } else if slice[mid] < target {
            low = mid + 1
        } else {
            high = mid - 1
        }
    }
    return -1
}

func main() {
    nums := []int{1, 2, 3, 4, 5, 6, 7, 8, 9}
    fmt.Println(BinarySearch(nums, 5)) // Output: 4
}
```

**Output:**

```
4
```

---
