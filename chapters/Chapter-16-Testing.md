# **Chapter 16: Testing in Go**
---

## **16.1 Why Test?**

Testing ensures that your code:

- **Works as expected** under different conditions.
- **Prevents regressions** when new features are added.
- **Improves maintainability** and builds confidence in your software.

---

## **16.2 Unit Testing in Go**

### **The Basics of Unit Testing**

Go provides a built-in testing framework in the `testing` package. To write tests:

1. Create a file named `*_test.go`.
2. Write test functions starting with `Test`.
3. Use `t.Error` or `t.Errorf` to indicate test failures.

---

### **Example 1: Testing a Simple Function**

Let’s start by testing a function that calculates the square of a number.

#### **Code: `math_utils.go`**

```go
package mathutils

func Square(x int) int {
    return x * x
}
```

#### **Test: `math_utils_test.go`**

```go
package mathutils

import "testing"

func TestSquare(t *testing.T) {
    result := Square(4)
    if result != 16 {
        t.Errorf("Expected 16, but got %d", result)
    }
}
```

#### **Run the Test**

```bash
go test
```

#### **Output**

```
PASS
ok      mathutils       0.002s
```

---

### **Example 2: Testing with Multiple Cases**

Use a table-driven approach to test multiple inputs efficiently.

#### **Code: `math_utils_test.go`**

```go
func TestSquareCases(t *testing.T) {
    testCases := []struct {
        input    int
        expected int
    }{
        {2, 4},
        {3, 9},
        {5, 25},
    }

    for _, tc := range testCases {
        result := Square(tc.input)
        if result != tc.expected {
            t.Errorf("Square(%d) = %d; want %d", tc.input, result, tc.expected)
        }
    }
}
```

#### **Output**

```
PASS
ok      mathutils       0.002s
```

---

### **16.3 Testing Edge Cases**

Testing is incomplete without edge cases. Let’s test a function that divides two numbers.

#### **Code: `math_utils.go`**

```go
func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}
```

#### **Test: `math_utils_test.go`**

```go
func TestDivide(t *testing.T) {
    testCases := []struct {
        a, b     float64
        expected float64
        wantErr  bool
    }{
        {10, 2, 5, false},
        {10, 0, 0, true},
    }

    for _, tc := range testCases {
        result, err := Divide(tc.a, tc.b)
        if (err != nil) != tc.wantErr {
            t.Errorf("Divide(%f, %f) unexpected error: %v", tc.a, tc.b, err)
        }
        if result != tc.expected {
            t.Errorf("Divide(%f, %f) = %f; want %f", tc.a, tc.b, result, tc.expected)
        }
    }
}
```

#### **Output**

```
PASS
ok      mathutils       0.003s
```

---

## **16.4 Benchmarking in Go**

Benchmarking helps you measure the performance of your code. In Go, benchmarks are written as functions starting with `Benchmark` and take a `*testing.B` parameter.

---

### **Example 3: Benchmarking a Simple Function**

Let’s benchmark the `Square` function.

#### **Benchmark: `math_utils_test.go`**

```go
func BenchmarkSquare(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Square(10)
    }
}
```

#### **Run the Benchmark**

```bash
go test -bench=.
```

#### **Output**

```
goos: darwin
goarch: amd64
pkg: mathutils
BenchmarkSquare-8    1761000000   0.25 ns/op
PASS
ok      mathutils       1.002s
```

---

### **Example 4: Comparing Performance**

Let’s compare performance between two implementations of a Fibonacci function: recursive and iterative.

#### **Code: `math_utils.go`**

```go
func FibRecursive(n int) int {
    if n <= 1 {
        return n
    }
    return FibRecursive(n-1) + FibRecursive(n-2)
}

func FibIterative(n int) int {
    a, b := 0, 1
    for i := 0; i < n; i++ {
        a, b = b, a+b
    }
    return a
}
```

#### **Benchmark: `math_utils_test.go`**

```go
func BenchmarkFibRecursive(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FibRecursive(10)
    }
}

func BenchmarkFibIterative(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FibIterative(10)
    }
}
```

#### **Output**

```
BenchmarkFibRecursive-8    30000   50915 ns/op
BenchmarkFibIterative-8    20000000    0.75 ns/op
```

---

## **16.5 Code Coverage**

Measure how much of your code is covered by tests.

#### **Run Coverage**

```bash
go test -cover
```

#### **Output**

```
ok      mathutils       100.0% coverage
```

---

## **16.6 Summary**

| Concept       | Description                  | Command            |
| ------------- | ---------------------------- | ------------------ |
| **Unit Test** | Validate function behavior.  | `go test`          |
| **Benchmark** | Measure performance of code. | `go test -bench=.` |
| **Coverage**  | Check test coverage.         | `go test -cover`   |

---

# **16.7. Exercises**

---

## **Exercise 1: Testing Addition**

**Problem**: Write a function to add two integers and test it.

```go
package mathutils

func Add(a, b int) int {
    return a + b
}
```

**Test**:

```go
package mathutils

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, but got %d", result)
    }
}
```

**Output**:

```
PASS
ok      mathutils       0.002s
```

---

## **Exercise 2: Testing a String Reversal Function**

**Problem**: Write a function to reverse a string and test it.

```go
package stringutils

func Reverse(s string) string {
    result := ""
    for _, char := range s {
        result = string(char) + result
    }
    return result
}
```

**Test**:

```go
package stringutils

import "testing"

func TestReverse(t *testing.T) {
    result := Reverse("hello")
    if result != "olleh" {
        t.Errorf("Expected 'olleh', but got '%s'", result)
    }
}
```

**Output**:

```
PASS
ok      stringutils     0.003s
```

---

## **Exercise 3: Testing Edge Cases with Zero Division**

**Problem**: Write a division function that handles divide-by-zero errors and test it.

```go
package mathutils

import "fmt"

func SafeDivide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}
```

**Test**:

```go
package mathutils

import "testing"

func TestSafeDivide(t *testing.T) {
    _, err := SafeDivide(10, 0)
    if err == nil {
        t.Errorf("Expected an error but got nil")
    }
}
```

**Output**:

```
PASS
ok      mathutils       0.004s
```

---

## **Exercise 4: Benchmarking String Concatenation**

**Problem**: Compare the performance of concatenation using `+` vs `strings.Builder`.

```go
package stringutils

import "strings"

func ConcatWithPlus(parts []string) string {
    result := ""
    for _, part := range parts {
        result += part
    }
    return result
}

func ConcatWithBuilder(parts []string) string {
    var builder strings.Builder
    for _, part := range parts {
        builder.WriteString(part)
    }
    return builder.String()
}
```

**Benchmark**:

```go
package stringutils

import "testing"

func BenchmarkConcatWithPlus(b *testing.B) {
    parts := []string{"Go", " is", " awesome"}
    for i := 0; i < b.N; i++ {
        ConcatWithPlus(parts)
    }
}

func BenchmarkConcatWithBuilder(b *testing.B) {
    parts := []string{"Go", " is", " awesome"}
    for i := 0; i < b.N; i++ {
        ConcatWithBuilder(parts)
    }
}
```

**Output**:

```
BenchmarkConcatWithPlus-8       1000000     1534 ns/op
BenchmarkConcatWithBuilder-8    2000000      732 ns/op
```

---

## **Exercise 5: Writing Table-Driven Tests**

**Problem**: Test a function that checks if a number is even.

```go
package mathutils

func IsEven(n int) bool {
    return n%2 == 0
}
```

**Test**:

```go
package mathutils

import "testing"

func TestIsEven(t *testing.T) {
    testCases := []struct {
        input    int
        expected bool
    }{
        {1, false},
        {2, true},
        {3, false},
        {4, true},
    }

    for _, tc := range testCases {
        result := IsEven(tc.input)
        if result != tc.expected {
            t.Errorf("IsEven(%d) = %v; want %v", tc.input, result, tc.expected)
        }
    }
}
```

**Output**:

```
PASS
ok      mathutils       0.003s
```

---

## **Exercise 6: Testing Struct Initialization**

**Problem**: Write tests to validate struct initialization.

```go
package models

type User struct {
    ID   int
    Name string
    Age  int
}

func NewUser(id int, name string, age int) User {
    return User{ID: id, Name: name, Age: age}
}
```

**Test**:

```go
package models

import "testing"

func TestNewUser(t *testing.T) {
    user := NewUser(1, "Alice", 25)
    if user.Name != "Alice" {
        t.Errorf("Expected name 'Alice', but got '%s'", user.Name)
    }
}
```

**Output**:

```
PASS
ok      models          0.001s
```

---

## **Exercise 7: Mocking External Dependencies**

**Problem**: Mock a function to simulate external API calls.

```go
package utils

var fetchData = func() string {
    return "Real Data"
}

func GetData() string {
    return fetchData()
}
```

**Test**:

```go
package utils

import "testing"

func TestGetData(t *testing.T) {
    fetchData = func() string { return "Mock Data" }
    result := GetData()
    if result != "Mock Data" {
        t.Errorf("Expected 'Mock Data', but got '%s'", result)
    }
}
```

**Output**:

```
PASS
ok      utils           0.002s
```

---

## **Exercise 8: Testing a Recursive Function**

**Problem**: Write a recursive function to calculate factorial and test it.

```go
package mathutils

func Factorial(n int) int {
    if n == 0 {
        return 1
    }
    return n * Factorial(n-1)
}
```

**Test**:

```go
package mathutils

import "testing"

func TestFactorial(t *testing.T) {
    if Factorial(5) != 120 {
        t.Errorf("Expected 120, but got %d", Factorial(5))
    }
}
```

**Output**:

```
PASS
ok      mathutils       0.002s
```

---

**Congratulations!** These exercises cover unit tests, benchmarks, and edge cases for real-world scenarios. Keep practicing to strengthen your testing skills.
