# **Chapter 7: Arrays, Slices, and Strings**

---

## **8.1 Arrays**

### **What Are Arrays?**

An **array** is a fixed-size collection of elements of the same type. Arrays in Go have a defined size, and their length cannot change during runtime. Arrays are particularly useful when you know the number of elements in advance, as their memory allocation is static.

Arrays are declared using the following syntax:

```go
var arrayName [size]type
```

Where:

- `arrayName` is the name of the array.
- `size` is the number of elements in the array.
- `type` is the type of elements the array will store.

### **Declaring an Array**

```go
package main
import "fmt"

func main() {
    var numbers [5]int       // Array of 5 integers
    fmt.Println(numbers)     // Output: [0 0 0 0 0]
}
```

**Explanation:**

- We declare an array named `numbers` with a size of 5.
- By default, an array of integers is initialized with zero values.

---

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

**Explanation:**

- We declare an array `fruits` of 3 strings and assign values to each index.
- We access individual elements by their index, with the first element starting at index 0.

---

### **Example 2: Using a Loop with Arrays**

```go
package main
import "fmt"

func main() {
    numbers := [5]int{10, 20, 30, 40, 50}
    for i, value := range numbers {
        fmt.Printf("Index %d: %d
", i, value)
    }
}
```

**Explanation:**

- The `numbers` array is initialized with values directly.
- We use a `for` loop with `range` to iterate over the array. `i` holds the index, and `value` holds the element at that index.

**Output:**

```
Index 0: 10
Index 1: 20
Index 2: 30
Index 3: 40
Index 4: 50
```

---

## **8.2 Slices**

### **What Are Slices?**

A **slice** is a dynamic, flexible view into the elements of an array. Unlike arrays, slices do not have a fixed size, which makes them more versatile for working with collections of data.

Slices are declared as follows:

```go
var sliceName []type
```

Where:

- `sliceName` is the name of the slice.
- `type` is the type of elements the slice will store.

### **Creating a Slice**

```go
package main
import "fmt"

func main() {
    numbers := []int{1, 2, 3, 4, 5} // Slice of integers
    fmt.Println(numbers)           // Output: [1 2 3 4 5]
}
```

**Explanation:**

- We create a slice using `[]int{1, 2, 3, 4, 5}`.
- Slices are dynamic, so their size can grow or shrink as needed.

---

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

**Explanation:**

- We create a slice from the `numbers` array by specifying a range of indices.
- The slice starts from index 1 and ends before index 4.

---

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

**Explanation:**

- We modify the second element of the slice, which changes the value in the underlying array.

---

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

**Explanation:**

- The `append` function adds elements to a slice. It automatically resizes the slice if needed.

---

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

**Explanation:**

- We create a new slice `destination` with the same length as the `source` slice.
- The `copy` function copies the elements from the source slice to the destination slice.

---

## **8.3 Strings**

### **What Are Strings?**

A **string** is a sequence of characters. In Go, strings are immutable, meaning they cannot be changed after creation. They are a type of slice, but with some limitations, such as being read-only.

Strings are declared as follows:

```go
var stringName string
```

Where:

- `stringName` is the name of the string.

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

**Explanation:**

- We declare a string variable `greeting` and print it.
- The `len` function returns the number of bytes in the string, which is equivalent to the length of the string.

---

### **Example 8: Iterating Over Strings**

```go
package main
import "fmt"

func main() {
    word := "GoLang"
    for i, char := range word {
        fmt.Printf("Index %d: %c
", i, char)
    }
}
```

**Explanation:**

- We iterate over the string `word` using `range`, where `i` is the index and `char` is the character at that index.

**Output:**

```
Index 0: G
Index 1: o
Index 2: L
Index 3: a
Index 4: n
Index 5: g
```

---

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

**Explanation:**

- We take a slice from the string `phrase`. The slice `phrase[3:5]` contains the characters from index 3 to 4.

---

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

**Explanation:**

- We use the `strings` package to perform operations like converting a string to uppercase (`ToUpper`) and checking if a string contains a substring (`Contains`).

---

## **Comparison Table: Arrays vs. Slices**

| Feature      | Arrays               | Slices                          |
| ------------ | -------------------- | ------------------------------- |
| Size         | Fixed                | Dynamic                         |
| Declaration  | `var arr [5]int`     | `var s []int` or `s := []int{}` |
| Memory Usage | Allocates fixed size | Backed by an array              |
| Use Cases    | Static collections   | Dynamic, flexible collections   |

---

# **8.4. Exercises**

---

## **Exercise 1: Summing Array Elements**

**Problem**: Create an array of 5 integers and calculate their sum using a function.

```go
package main

import "fmt"

func sumArray(arr [5]int) (int, error) {
    if len(arr) == 0 {
        return 0, fmt.Errorf("Array cannot be empty")
    }
    sum := 0
    for _, value := range arr {
        sum += value
    }
    return sum, nil
}

func main() {
    numbers := [5]int{10, 20, 30, 40, 50}
    sum, err := sumArray(numbers)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Sum:", sum)
}
```

**Explanation**:

- The function `sumArray` calculates the sum of elements in an array. If the array is empty, it returns an error.
- The `main` function calls `sumArray` and handles any errors.

**Output:**

```
Sum: 150
```

---

## **Exercise 2: Finding the Maximum in an Array**

**Problem**: Write a program to find the largest number in an array using a function.

```go
package main

import "fmt"

func findMax(arr [5]int) (int, error) {
    if len(arr) == 0 {
        return 0, fmt.Errorf("Array cannot be empty")
    }
    max := arr[0]
    for _, value := range arr {
        if value > max {
            max = value
        }
    }
    return max, nil
}

func main() {
    numbers := [5]int{15, 42, 8, 23, 77}
    max, err := findMax(numbers)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Maximum value:", max)
}
```

**Explanation**:

- The `findMax` function returns the largest value in an array. It checks for an empty array and returns an error if needed.
- The `main` function handles the output or error accordingly.

**Output:**

```
Maximum value: 77
```

---

## **Exercise 3: Reversing a Slice**

**Problem**: Create a slice of strings and print the elements in reverse order using a function.

```go
package main

import "fmt"

func reverseSlice(words []string) ([]string, error) {
    if len(words) == 0 {
        return nil, fmt.Errorf("Slice cannot be empty")
    }
    reversed := []string{}
    for i := len(words) - 1; i >= 0; i-- {
        reversed = append(reversed, words[i])
    }
    return reversed, nil
}

func main() {
    words := []string{"Go", "is", "fun"}
    reversed, err := reverseSlice(words)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Reversed:", reversed)
}
```

**Explanation**:

- The function `reverseSlice` reverses the order of elements in a slice and checks if the slice is empty before processing.
- The `main` function prints the reversed slice.

**Output:**

```
Reversed: [fun is Go]
```

---

## **Exercise 4: Dynamic Slicing**

**Problem**: Create a slice of integers, append 5 values, and print the resulting slice.

```go
package main

import "fmt"

func appendValues(numbers []int, values ...int) []int {
    return append(numbers, values...)
}

func main() {
    numbers := []int{}
    numbers = appendValues(numbers, 1, 2, 3, 4, 5)
    fmt.Println("Slice:", numbers)
}
```

**Explanation**:

- The `appendValues` function appends values to a slice dynamically.
- The `main` function demonstrates appending values to an empty slice.

**Output:**

```
Slice: [1 2 3 4 5]
```

---

## **Exercise 5: Copying Slices**

**Problem**: Copy one slice into another and print both using a function.

```go
package main

import "fmt"

func copySlice(source []int) ([]int, error) {
    if len(source) == 0 {
        return nil, fmt.Errorf("Source slice cannot be empty")
    }
    destination := make([]int, len(source))
    copy(destination, source)
    return destination, nil
}

func main() {
    source := []int{5, 10, 15}
    destination, err := copySlice(source)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Source slice:", source)
    fmt.Println("Destination slice:", destination)
}
```

**Explanation**:

- The `copySlice` function copies the elements of the source slice to a destination slice and checks if the source slice is empty.
- The `main` function demonstrates copying slices.

**Output:**

```
Source slice: [5 10 15]
Destination slice: [5 10 15]
```

---

## **Exercise 6: Iterating Over a String**

**Problem**: Write a program to iterate over the characters of a string using a function.

```go
package main

import "fmt"

func iterateString(word string) ([]string, error) {
    if len(word) == 0 {
        return nil, fmt.Errorf("String cannot be empty")
    }
    var chars []string
    for i, char := range word {
        chars = append(chars, fmt.Sprintf("Index %d: %c", i, char))
    }
    return chars, nil
}

func main() {
    word := "Golang"
    chars, err := iterateString(word)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    for _, char := range chars {
        fmt.Println(char)
    }
}
```

**Explanation**:

- The function `iterateString` iterates over a string and stores the index and character in a slice. It checks for an empty string.
- The `main` function prints each index and character.

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

**Problem**: Extract a substring from a string using slicing and handle potential errors.

```go
package main

import "fmt"

func extractSubstring(sentence string, start, end int) (string, error) {
    if start < 0 || end > len(sentence) || start >= end {
        return "", fmt.Errorf("Invalid slice indices")
    }
    return sentence[start:end], nil
}

func main() {
    sentence := "Go is powerful"
    substring, err := extractSubstring(sentence, 3, 5)
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    fmt.Println("Substring:", substring)
}
```

**Explanation**:

- The `extractSubstring` function extracts a substring from a string. It validates the slice indices before proceeding.
- The `main` function handles any errors that may arise.

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

func countWords(sentence string) int {
    words := strings.Fields(sentence)
    return len(words)
}

func main() {
    sentence := "Go is simple and efficient"
    count := countWords(sentence)
    fmt.Println("Number of words:", count)
}
```

**Explanation**:

- The function `countWords` uses the `strings.Fields` method to split the sentence into words and returns the count.
- The `main` function displays the word count.

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

func convertCase(text string) (string, string) {
    return strings.ToUpper(text), strings.ToLower(text)
}

func main() {
    text := "Go is Awesome"
    upper, lower := convertCase(text)
    fmt.Println("Uppercase:", upper)
    fmt.Println("Lowercase:", lower)
}
```

**Explanation**:

- The `convertCase` function converts a string to uppercase and lowercase using `strings.ToUpper` and `strings.ToLower`.
- The `main` function prints both versions.

**Output:**

```
Uppercase: GO IS AWESOME
Lowercase: go is awesome
```

---

## **Exercise 10: Checking Substring Presence**

**Problem**: Write a program to check if a substring exists in a string using the `strings` package.

```go
package main

import (
    "fmt"
    "strings"
)

func checkSubstring(phrase, sub string) bool {
    return strings.Contains(phrase, sub)
}

func main() {
    phrase := "Go is fun"
    fmt.Println("Contains 'fun':", checkSubstring(phrase, "fun"))
    fmt.Println("Contains 'boring':", checkSubstring(phrase, "boring"))
}
```

**Explanation**:

- The function `checkSubstring` checks whether a substring exists in a string using `strings.Contains`.
- The `main` function prints the results of substring checks.

**Output:**

```
Contains 'fun': true
Contains 'boring': false
```

--
