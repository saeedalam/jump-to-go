# **Chapter 7: Error Handling in Go**

Error handling is an essential part of programming, ensuring that programs can handle unexpected situations gracefully. Go provides a straightforward and effective way to manage errors without relying on exceptions, making your code predictable and easy to debug. This chapter will help you master error handling with simple explanations and practical examples.

---

### **7.1 What is Error Handling in Go?**

In Go, errors are values. Functions that might encounter errors typically return two values:

1. **The result** of the operation (e.g., a number, string, etc.)
2. **An error**, which is either `nil` (if there is no issue) or contains an error message.

The `error` type is an interface with a single method:

```go
Error() string
```

This method provides the error message.

#### **Key Principles of Error Handling**:

- Always check the error value returned by a function.
- If an error occurs, handle it appropriately or propagate it to the caller.
- Avoid complex error handling chains—keep it simple.

---

### **7.2 Returning and Handling Errors**

Let’s begin with an example of a function that returns an error when something goes wrong.

#### **Example: Division Function**

This function divides two numbers and returns an error if the divisor is zero.

```go
package main

import (
	"errors"
	"fmt"
)

func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, errors.New("cannot divide by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divide(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("Result:", result)
}
```

#### **What Happens Here?**

1. If the divisor is zero, the function returns an error using `errors.New`.
2. In the `main` function, the error is checked. If it’s not `nil`, the program handles the error by printing it.
3. If there’s no error, the result is displayed.

**Output:**

```
Error: cannot divide by zero
```

---

### **7.3 Reading Files Safely**

Errors are commonly encountered when working with files. Let’s see how to handle errors when reading a file.

#### **Example: Reading a File**

```go
package main

import (
	"fmt"
	"os"
)

func readFile(filename string) (string, error) {
	data, err := os.ReadFile(filename)
	if err != nil {
		return "", err
	}
	return string(data), nil
}

func main() {
	content, err := readFile("example.txt")
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}
	fmt.Println("File Content:", content)
}
```

#### **Explanation:**

- The `readFile` function attempts to read the file using `os.ReadFile`.
- If an error occurs (e.g., the file doesn’t exist), it is returned to the caller.
- The `main` function handles the error by printing a message if one occurs.

**Output (if file doesn’t exist):**

```
Error reading file: open example.txt: no such file or directory
```

---

### **7.4 Adding More Context to Errors**

Sometimes, you may need to add extra details to an error message to make it more meaningful.

#### **Example: Adding Context**

```go
package main

import (
	"fmt"
)

func loadConfig(filename string) error {
	return fmt.Errorf("failed to load config: file %s not found", filename)
}

func main() {
	err := loadConfig("config.yaml")
	if err != nil {
		fmt.Println(err)
	}
}
```

#### **Explanation:**

- `fmt.Errorf` is used to format a detailed error message.
- This makes it clear which file caused the issue.

**Output:**

```
failed to load config: file config.yaml not found
```

---

### **7.5 Wrapping Errors**

Go allows you to wrap errors to preserve the original error while providing more context.

#### **Example: Wrapping Errors**

```go
package main

import (
	"errors"
	"fmt"
)

func openFile(filename string) error {
	return fmt.Errorf("could not open file: %w", errors.New("file does not exist"))
}

func main() {
	err := openFile("data.txt")
	if err != nil {
		fmt.Println(err)
	}
}
```

#### **Explanation:**

- `%w` is used in `fmt.Errorf` to wrap an existing error.
- This technique preserves the original error and adds more context.

**Output:**

```
could not open file: file does not exist
```

---

### **7.6 Unwrapping Errors**

If an error is wrapped, you can retrieve the original error using `errors.Unwrap`.

#### **Example: Unwrapping Errors**

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := fmt.Errorf("outer error: %w", errors.New("inner error"))
	fmt.Println("Full Error:", err)
	fmt.Println("Inner Error:", errors.Unwrap(err))
}
```

#### **Explanation:**

- `errors.Unwrap` extracts the original error from a wrapped error chain.
- This is useful for inspecting specific errors.

**Output:**

```
Full Error: outer error: inner error
Inner Error: inner error
```

---

### **7.7 Checking Specific Errors**

To check if an error matches a specific type, use `errors.Is`.

#### **Example: Checking for Specific Errors**

```go
package main

import (
	"errors"
	"fmt"
)

var ErrPermissionDenied = errors.New("permission denied")

func performAction() error {
	return fmt.Errorf("action failed: %w", ErrPermissionDenied)
}

func main() {
	err := performAction()
	if errors.Is(err, ErrPermissionDenied) {
		fmt.Println("Action failed due to permission issues.")
	}
}
```

#### **Explanation:**

- `errors.Is` checks if the error matches a specific value, even if it’s wrapped.
- This makes error checking precise and readable.

**Output:**

```
Action failed due to permission issues.
```

---

### **Summary**

- Errors in Go are handled using values, not exceptions.
- Always check for errors and handle them immediately.
- Add meaningful context to errors when needed.
- Use `errors.Unwrap`, `errors.Is`, and `errors.As` for advanced error handling.

Error handling in Go is designed to be simple and explicit, encouraging better code quality and reliability.

### **Exercise 1: Simple Math Operation with Error Handling**

**Problem**: Write a function `divideNumbers` that accepts two integers and returns their division result. Handle division by zero errors appropriately.

```go
package main

import (
	"errors"
	"fmt"
)

func divideNumbers(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divideNumbers(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
}
```

**Explanation:**

- The `divideNumbers` function ensures safe division by checking for a zero divisor and returning an error if detected.
- The result is returned only when no error occurs.

**Output:**

```
Error: division by zero
```

---

### **Exercise 2: Square Root Validation**

**Problem**: Write a function `calculateSquareRoot` that accepts a non-negative integer and returns its square root. Return an error for negative inputs.

```go
package main

import (
	"errors"
	"fmt"
	"math"
)

func calculateSquareRoot(num int) (float64, error) {
	if num < 0 {
		return 0, errors.New("cannot calculate square root of a negative number")
	}
	return math.Sqrt(float64(num)), nil
}

func main() {
	result, err := calculateSquareRoot(-9)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Square Root:", result)
	}
}
```

**Explanation:**

- The `calculateSquareRoot` function validates the input to ensure it is non-negative.
- For valid inputs, the square root is computed using `math.Sqrt`.

**Output:**

```
Error: cannot calculate square root of a negative number
```

---

### **Exercise 3: Prime Number Checker**

**Problem**: Write a function `isPrime` that checks if a given number is prime. Return an error if the input is less than 2.

```go
package main

import (
	"errors"
	"fmt"
)

func isPrime(num int) (bool, error) {
	if num < 2 {
		return false, errors.New("number must be greater than or equal to 2")
	}
	for i := 2; i*i <= num; i++ {
		if num%i == 0 {
			return false, nil
		}
	}
	return true, nil
}

func main() {
	result, err := isPrime(1)
	if err != nil {
		fmt.Println("Error:", err)
	} else if result {
		fmt.Println("The number is prime.")
	} else {
		fmt.Println("The number is not prime.")
	}
}
```

**Explanation:**

- The `isPrime` function checks for inputs less than 2 and returns an error.
- For valid inputs, it determines if the number is prime using a loop.

**Output:**

```
Error: number must be greater than or equal to 2
```

---

### **Exercise 4: Fibonacci Sequence Generator**

**Problem**: Write a function `generateFibonacci` that generates the first `n` numbers of the Fibonacci sequence. Return an error if `n` is less than or equal to zero.

```go
package main

import (
	"errors"
	"fmt"
)

func generateFibonacci(n int) ([]int, error) {
	if n <= 0 {
		return nil, errors.New("input must be greater than 0")
	}
	fib := []int{0, 1}
	for len(fib) < n {
		fib = append(fib, fib[len(fib)-1]+fib[len(fib)-2])
	}
	return fib[:n], nil
}

func main() {
	sequence, err := generateFibonacci(-5)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Fibonacci Sequence:", sequence)
	}
}
```

**Explanation:**

- The `generateFibonacci` function checks if the input is valid and generates the sequence using a loop.
- If the input is invalid, an error is returned.

**Output:**

```
Error: input must be greater than 0
```
