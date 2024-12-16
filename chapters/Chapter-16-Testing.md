
# **Chapter 16: Testing in Go**

---

## **16.1 Why Test?**

Testing is crucial to ensure that your code is reliable and performs well under different conditions. It offers the following benefits:

- **Works as expected:** Tests help ensure that your code functions as intended in different scenarios.
- **Prevents regressions:** Tests catch issues that might arise when new features are added, ensuring that previously working functionality doesn't break.
- **Improves maintainability:** Well-tested code is easier to maintain and refactor since tests provide immediate feedback on changes.

---

## **16.2 Unit Testing in Go**

### **What is Unit Testing?**

Unit testing is the process of testing individual functions in isolation. In Go, the built-in `testing` package provides tools for writing and running tests. A test typically checks a function's behavior for different inputs, ensuring it produces the expected outputs.

### **The Basics of Unit Testing**

To write a unit test in Go:

1. **Create a Test File:** Test files must end with `_test.go`.
2. **Write Test Functions:** Each test function should start with `Test` and take a `*testing.T` parameter.
3. **Report Failures:** Use `t.Error` or `t.Errorf` to report test failures.

Now, let's look at how to write a basic unit test.

### **Example 1: Testing a Simple Function**

Let’s start by testing a function that calculates the square of a number.

#### **Code: `math_utils.go`**

```go
package mathutils

// Square returns the square of a number.
func Square(x int) int {
    return x * x
}
```

**Explanation:**

- The `Square` function multiplies a number by itself and returns the result.

#### **Test: `math_utils_test.go`**

```go
package mathutils

import "testing"

// TestSquare tests the Square function.
func TestSquare(t *testing.T) {
    result := Square(4)
    if result != 16 {
        t.Errorf("Expected 16, but got %d", result)
    }
}
```

**Explanation:**

- The `TestSquare` function checks if the `Square` function correctly squares the input `4`.
- If the result is not as expected, it reports an error.

#### **Run the Test**

```bash
go test
```

**Output:**

```
PASS
ok      mathutils       0.002s
```

---

### **Example 2: Testing with Multiple Cases**

Go provides a simple way to run multiple tests using a table-driven approach, which allows us to test various inputs efficiently.

#### **Code: `math_utils_test.go`**

```go
// TestSquareCases tests Square with multiple cases.
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

**Explanation:**

- We create a table of test cases with various inputs and expected results.
- For each case, we check whether the `Square` function returns the expected result.

#### **Run the Test**

```bash
go test
```

**Output:**

```
PASS
ok      mathutils       0.002s
```

---

## **16.3 Testing Edge Cases**

Edge cases are important to test because they often reveal bugs or unexpected behavior. Let’s test a function that divides two numbers and handles division by zero.

#### **Code: `math_utils.go`**

```go
// Divide divides two numbers and returns an error if the denominator is zero.
func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}
```

**Explanation:**

- The `Divide` function returns the result of dividing `a` by `b` unless `b` is zero, in which case it returns an error.

#### **Test: `math_utils_test.go`**

```go
// TestDivide tests the Divide function.
func TestDivide(t *testing.T) {
    testCases := []struct {
        a, b     float64
        expected float64
        wantErr  bool
    }{
        {10, 2, 5, false}, // valid division
        {10, 0, 0, true},  // division by zero
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

**Explanation:**

- The `TestDivide` function tests for both valid and invalid cases (like dividing by zero).
- It checks that the function either returns the correct result or an appropriate error.

#### **Run the Test**

```bash
go test
```

**Output:**

```
PASS
ok      mathutils       0.003s
```

---

## **16.4 Benchmarking in Go**

Benchmarking allows you to measure the performance of your code. In Go, benchmarks are written as functions starting with `Benchmark` and take a `*testing.B` parameter.

### **Example 3: Benchmarking a Simple Function**

Let’s benchmark the `Square` function to measure its performance.

#### **Benchmark: `math_utils_test.go`**

```go
// BenchmarkSquare benchmarks the Square function.
func BenchmarkSquare(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Square(10)
    }
}
```

**Explanation:**

- The `BenchmarkSquare` function repeatedly calls the `Square` function to measure its performance.

#### **Run the Benchmark**

```bash
go test -bench=.
```

**Output:**

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

Now, let’s compare the performance of two implementations of a Fibonacci function: recursive and iterative.

#### **Code: `math_utils.go`**

```go
// FibRecursive is a recursive Fibonacci function.
func FibRecursive(n int) int {
    if n <= 1 {
        return n
    }
    return FibRecursive(n-1) + FibRecursive(n-2)
}

// FibIterative is an iterative Fibonacci function.
func FibIterative(n int) int {
    a, b := 0, 1
    for i := 0; i < n; i++ {
        a, b = b, a+b
    }
    return a
}
```

**Explanation:**

- `FibRecursive` computes Fibonacci numbers recursively.
- `FibIterative` computes Fibonacci numbers iteratively.

#### **Benchmark: `math_utils_test.go`**

```go
// BenchmarkFibRecursive benchmarks the recursive Fibonacci function.
func BenchmarkFibRecursive(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FibRecursive(10)
    }
}

// BenchmarkFibIterative benchmarks the iterative Fibonacci function.
func BenchmarkFibIterative(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FibIterative(10)
    }
}
```

#### **Run the Benchmark**

```bash
go test -bench=.
```

**Output:**

```
BenchmarkFibRecursive-8    30000   50915 ns/op
BenchmarkFibIterative-8    20000000    0.75 ns/op
```

---

## **16.5 Code Coverage**

Code coverage shows how much of your code is tested. To check your test coverage:

#### **Run Coverage**

```bash
go test -cover
```

**Output:**

```
ok      mathutils       100.0% coverage
```

---

## **16.6 Summary**

| Concept       | Description                          | Command            |
| ------------- | ------------------------------------ | ------------------ |
| **Unit Test** | Validate function behavior.         | `go test`          |
| **Benchmark** | Measure performance of code.        | `go test -bench=.` |
| **Coverage**  | Check test coverage.                | `go test -cover`   |

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
