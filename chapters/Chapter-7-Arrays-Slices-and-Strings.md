# **Chapter 7: Arrays, Slices, and Strings**

---

## **7.1 Arrays**

### **What Are Arrays?**

An **array** is a fixed-size collection of elements of the same type. Arrays in Go have a defined size, and their length cannot change during runtime.

### **Declaring an Array**

```go
package main
import "fmt"

func main() {
    var numbers [5]int       // Array of 5 integers
    fmt.Println(numbers)     // Output: [0 0 0 0 0]
}
```

### **Example 1: Initializing and Accessing Arrays**

```go
package main
import "fmt"

func main() {
    var fruits [3]string
    fruits[0] = "Apple"
    fruits[1] = "Banana"
    fruits[2] = "Cherry"

    fmt.Println(fruits)        // Output: [Apple Banana Cherry]
    fmt.Println(fruits[1])     // Output: Banana
}
```

### **Example 2: Using a Loop with Arrays**

```go
package main
import "fmt"

func main() {
    numbers := [5]int{10, 20, 30, 40, 50}
    for i, value := range numbers {
        fmt.Printf("Index %d: %d\n", i, value)
    }
}
```

**Output:**

```
Index 0: 10
Index 1: 20
Index 2: 30
Index 3: 40
Index 4: 50
```

---

## **7.2 Slices**

### **What Are Slices?**

A **slice** is a dynamic, flexible view into the elements of an array. Unlike arrays, slices do not have a fixed size.

### **Creating a Slice**

```go
package main
import "fmt"

func main() {
    numbers := []int{1, 2, 3, 4, 5} // Slice of integers
    fmt.Println(numbers)           // Output: [1 2 3 4 5]
}
```

### **Example 3: Slicing an Array**

```go
package main
import "fmt"

func main() {
    numbers := [6]int{10, 20, 30, 40, 50, 60}
    slice := numbers[1:4] // Slice from index 1 to 3
    fmt.Println(slice)    // Output: [20 30 40]
}
```

### **Example 4: Modifying a Slice**

```go
package main
import "fmt"

func main() {
    numbers := []int{10, 20, 30}
    numbers[1] = 99
    fmt.Println(numbers) // Output: [10 99 30]
}
```

### **Example 5: Using `append` with Slices**

```go
package main
import "fmt"

func main() {
    var numbers []int
    numbers = append(numbers, 10, 20, 30)
    fmt.Println(numbers) // Output: [10 20 30]
}
```

### **Example 6: Copying Slices**

```go
package main
import "fmt"

func main() {
    source := []int{1, 2, 3}
    destination := make([]int, len(source))
    copy(destination, source)
    fmt.Println(destination) // Output: [1 2 3]
}
```

---

## **7.3 Strings**

### **What Are Strings?**

A **string** is a sequence of characters. Strings in Go are immutable, meaning they cannot be changed after creation.

### **Example 7: Declaring and Using Strings**

```go
package main
import "fmt"

func main() {
    greeting := "Hello, Go!"
    fmt.Println(greeting)       // Output: Hello, Go!
    fmt.Println(len(greeting))  // Output: 10 (length of the string)
}
```

### **Example 8: Iterating Over Strings**

```go
package main
import "fmt"

func main() {
    word := "GoLang"
    for i, char := range word {
        fmt.Printf("Index %d: %c\n", i, char)
    }
}
```

**Output:**

```
Index 0: G
Index 1: o
Index 2: L
Index 3: a
Index 4: n
Index 5: g
```

### **Example 9: Strings and Slices**

```go
package main
import "fmt"

func main() {
    phrase := "Go is fun!"
    slice := phrase[3:5]
    fmt.Println(slice) // Output: is
}
```

### **Example 10: Using the `strings` Package**

```go
package main
import (
    "fmt"
    "strings"
)

func main() {
    phrase := "Go is great!"
    fmt.Println(strings.ToUpper(phrase)) // Output: GO IS GREAT!
    fmt.Println(strings.Contains(phrase, "Go")) // Output: true
}
```

---

## **Comparison Table: Arrays vs. Slices**

| Feature      | Arrays               | Slices                          |
| ------------ | -------------------- | ------------------------------- |
| Size         | Fixed                | Dynamic                         |
| Declaration  | `var arr [5]int`     | `var s []int` or `s := []int{}` |
| Memory Usage | Allocates fixed size | Backed by an array              |
| Use Cases    | Static collections   | Dynamic, flexible collections   |

---

# **Exercises**

This section contains **10 solved examples** to help you master arrays, slices, and strings in Go. Each exercise is designed to reinforce the concepts learned in the chapter with practical applications and outputs.

---

## **Exercise 1: Summing Array Elements**

**Problem**: Create an array of 5 integers and calculate their sum.

```go
package main

import "fmt"

func main() {
    numbers := [5]int{10, 20, 30, 40, 50}
    sum := 0
    for _, value := range numbers {
        sum += value
    }
    fmt.Println("Sum:", sum)
}
```

**Output:**

```
Sum: 150
```

---

## **Exercise 2: Finding the Maximum in an Array**

**Problem**: Write a program to find the largest number in an array.

```go
package main

import "fmt"

func main() {
    numbers := [5]int{15, 42, 8, 23, 77}
    max := numbers[0]
    for _, value := range numbers {
        if value > max {
            max = value
        }
    }
    fmt.Println("Maximum value:", max)
}
```

**Output:**

```
Maximum value: 77
```

---

## **Exercise 3: Reversing a Slice**

**Problem**: Create a slice of strings and print the elements in reverse order.

```go
package main

import "fmt"

func main() {
    words := []string{"Go", "is", "fun"}
    for i := len(words) - 1; i >= 0; i-- {
        fmt.Println(words[i])
    }
}
```

**Output:**

```
fun
is
Go
```

---

## **Exercise 4: Dynamic Slicing**

**Problem**: Create a slice of integers, append 5 values, and print the resulting slice.

```go
package main

import "fmt"

func main() {
    numbers := []int{}
    numbers = append(numbers, 1, 2, 3, 4, 5)
    fmt.Println("Slice:", numbers)
}
```

**Output:**

```
Slice: [1 2 3 4 5]
```

---

## **Exercise 5: Copying Slices**

**Problem**: Copy one slice into another and print both.

```go
package main

import "fmt"

func main() {
    source := []int{5, 10, 15}
    destination := make([]int, len(source))
    copy(destination, source)
    fmt.Println("Source slice:", source)
    fmt.Println("Destination slice:", destination)
}
```

**Output:**

```
Source slice: [5 10 15]
Destination slice: [5 10 15]
```

---

## **Exercise 6: Iterating Over a String**

**Problem**: Write a program to iterate over the characters of a string.

```go
package main

import "fmt"

func main() {
    phrase := "Golang"
    for i, char := range phrase {
        fmt.Printf("Index %d: %c
", i, char)
    }
}
```

**Output:**

```
Index 0: G
Index 1: o
Index 2: l
Index 3: a
Index 4: n
Index 5: g
```

---

## **Exercise 7: String Slicing**

**Problem**: Extract a substring from a string using slicing.

```go
package main

import "fmt"

func main() {
    sentence := "Go is powerful"
    substring := sentence[3:5]
    fmt.Println("Substring:", substring)
}
```

**Output:**

```
Substring: is
```

---

## **Exercise 8: Counting Words in a String**

**Problem**: Use the `strings` package to count the number of words in a sentence.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    sentence := "Go is simple and efficient"
    words := strings.Fields(sentence)
    fmt.Println("Number of words:", len(words))
}
```

**Output:**

```
Number of words: 5
```

---

## **Exercise 9: Converting Case**

**Problem**: Convert a string to uppercase and lowercase using the `strings` package.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    text := "Go is Awesome"
    fmt.Println("Uppercase:", strings.ToUpper(text))
    fmt.Println("Lowercase:", strings.ToLower(text))
}
```

**Output:**

```
Uppercase: GO IS AWESOME
Lowercase: go is awesome
```

---

## **Exercise 10: Checking Substring Presence**

**Problem**: Write a program to check if a substring exists in a string.

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    phrase := "Go is fun"
    fmt.Println("Contains 'fun':", strings.Contains(phrase, "fun"))
    fmt.Println("Contains 'boring':", strings.Contains(phrase, "boring"))
}
```

**Output:**

```
Contains 'fun': true
Contains 'boring': false
```

---

**Congratulations!** You've completed the exercises for Chapter 7. These examples cover arrays, slices, and strings comprehensively, ensuring you are ready to use them effectively in real-world scenarios.
