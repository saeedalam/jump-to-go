# **Chapter 18: Generics (Go 1.18+)**

---

## **18.1. Generic Functions**

Generics enable functions and data structures to operate on multiple types without duplicating code. Instead of writing separate implementations for different types, you can use a **type parameter** to create flexible, type-safe code.

A **generic function** is a function that works with any type. Type parameters are specified using square brackets (`[]`) after the function name.

### **Example 1: A Generic `Print` Function**

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

### **Output:**

```
Hello, Generics!
42
3.14
```

---

### **Example 2: A Generic `Sum` Function**

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

### **Output:**

```
30
7.8
```

---

### **Example 3: Constraints in Generics**

You can restrict type parameters to specific types using **constraints**. Go provides the `any` keyword (an alias for `interface{}`) and the `constraints` package for custom constraints.

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

### **Output:**

```
20
3.5
dog
```

---

### **Example 4: A Generic `Contains` Function for Slices**

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

### **Output:**

```
true
false
true
false
```

---

## **18.2. Generic Structs**

A **generic struct** uses type parameters to operate on multiple types. This makes structs more flexible and reusable.

### **Example 5: A Generic Pair**

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

### **Output:**

```
{1 2}
{hello world}
{pi 3.14}
```

---

### **Example 6: A Generic Stack**

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

### **Output:**

```
20
10
Generics
Go
```

---

### **Example 7: A Generic Map**

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

### **Output:**

```
1
2
```

---

## **18.3. Exercises**

## **Exercise 1: Generic Minimum**

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

## **Exercise 2: Generic Maximum**

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

## **Exercise 3: Generic Swap**

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

## **Exercise 4: Generic Filter**

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

## **Exercise 5: Generic Map**

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

## **Exercise 6: Generic Stack**

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

## **Exercise 7: Generic Key-Value Store**

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

## **Exercise 8: Generic Pair**

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

**Congratulations!** You’ve completed the exercises for Chapter 18. These examples illustrate how generics can make your code flexible, reusable, and type-safe. Experiment further to master this powerful feature!
