
# **Chapter 6: Error Handling in Go**

---

## **6.1 Returning Errors**

### **Example 1: A Simple Division Function**

In Go, functions often return an `error` value along with the primary result.

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

**Output:**
```
Error: cannot divide by zero
```

---

### **Example 2: Handling Errors with Multiple Return Values**

Go encourages checking errors immediately.

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
	content, err := readFile("nonexistent.txt")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	fmt.Println("File Content:", content)
}
```

**Output:**
```
Error: open nonexistent.txt: no such file or directory
```

---

## **6.2 Custom Error Types**

Sometimes, you need more context in your errors. Go allows you to create custom error types.

### **Example 3: Creating a Custom Error Type**

```go
package main

import (
	"fmt"
)

type DivideError struct {
	Dividend float64
	Divisor  float64
	Message  string
}

func (e *DivideError) Error() string {
	return fmt.Sprintf("error: %s (dividend: %.2f, divisor: %.2f)", e.Message, e.Dividend, e.Divisor)
}

func divide(a, b float64) (float64, error) {
	if b == 0 {
		return 0, &DivideError{
			Dividend: a,
			Divisor:  b,
			Message:  "cannot divide by zero",
		}
	}
	return a / b, nil
}

func main() {
	_, err := divide(10, 0)
	if err != nil {
		fmt.Println(err)
	}
}
```

**Output:**
```
error: cannot divide by zero (dividend: 10.00, divisor: 0.00)
```

---

### **Example 4: Checking for Specific Error Types**

You can use **type assertions** to handle specific error types.

```go
func main() {
	_, err := divide(10, 0)
	if customErr, ok := err.(*DivideError); ok {
		fmt.Printf("Custom Error: %s
", customErr.Message)
		return
	}
	fmt.Println("Error:", err)
}
```

**Output:**
```
Custom Error: cannot divide by zero
```

---

## **6.3 Error Wrapping and Unwrapping**

Go 1.13 introduced `errors.Is` and `errors.As` to unwrap and compare errors.

### **Example 5: Wrapping Errors**

Use `fmt.Errorf` to add context to errors.

```go
package main

import (
	"fmt"
)

func readConfig(filename string) error {
	return fmt.Errorf("failed to read config: %w", fmt.Errorf("file not found: %s", filename))
}

func main() {
	err := readConfig("config.yaml")
	fmt.Println(err)
}
```

**Output:**
```
failed to read config: file not found: config.yaml
```

---

### **Example 6: Unwrapping Errors with `errors.Unwrap`**

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

**Output:**
```
Full Error: outer error: inner error
Inner Error: inner error
```

---

### **Example 7: Checking for Specific Errors with `errors.Is`**

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

**Output:**
```
Action failed due to permission issues.
```

---

### **Example 8: Handling Multiple Error Layers with `errors.As`**

```go
package main

import (
	"errors"
	"fmt"
)

type ConfigError struct {
	File string
}

func (e *ConfigError) Error() string {
	return fmt.Sprintf("config error: file %s", e.File)
}

func main() {
	err := fmt.Errorf("failed to load app: %w", &ConfigError{File: "app.yaml"})
	var configErr *ConfigError
	if errors.As(err, &configErr) {
		fmt.Println("Specific Error:", configErr.File)
	}
}
```

**Output:**
```
Specific Error: app.yaml
```

---

## **Summary Table**

| Feature               | Function / Package         | Example                |
|-----------------------|---------------------------|-----------------------|
| Create Basic Error     | `errors.New`              | Example 1             |
| Return Custom Errors   | Custom Struct + `Error()` | Example 3             |
| Wrap Errors            | `fmt.Errorf` + `%w`       | Example 5             |
| Unwrap Errors          | `errors.Unwrap`           | Example 6             |
| Check Errors           | `errors.Is`               | Example 7             |
| Extract Specific Error | `errors.As`               | Example 8             |

---
## **6.7 Practical Exercises**

This section contains **solved examples** to help you master the concepts of error handling, variadic functions, and error handling in Go. Each exercise is accompanied by code, explanations, and expected outputs.

## **Exercise 1: Division with Error Handling**

**Problem**: Write a function `safeDivide` that takes two integers and returns their division result or an error if the denominator is zero.

```go
package main

import (
	"errors"
	"fmt"
)

func safeDivide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := safeDivide(10, 2)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
}
```

**Output:**
```
Result: 5
```

---

## **Exercise 2: File Reading with Error Handling**

**Problem**: Write a function `readFile` that takes a file path and returns its contents or an error if the file doesn’t exist.

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
	content, err := readFile("nonexistent.txt")
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Content:", content)
	}
}
```

**Output:**
```
Error: open nonexistent.txt: no such file or directory
```

---

## **Exercise 3: Custom Error Type**

**Problem**: Create a custom error type `MathError` for handling mathematical errors.

```go
package main

import (
	"fmt"
)

type MathError struct {
	Operation string
	Message   string
}

func (e *MathError) Error() string {
	return fmt.Sprintf("Math Error in %s: %s", e.Operation, e.Message)
}

func safeSubtract(a, b int) (int, error) {
	if b > a {
		return 0, &MathError{
			Operation: "Subtraction",
			Message:   "negative result not allowed",
		}
	}
	return a - b, nil
}

func main() {
	_, err := safeSubtract(5, 10)
	if err != nil {
		fmt.Println(err)
	}
}
```

**Output:**
```
Math Error in Subtraction: negative result not allowed
```

---

## **Exercise 4: Wrapping Errors**

**Problem**: Write a function `loadConfig` that wraps errors with context using `fmt.Errorf`.

```go
package main

import (
	"fmt"
)

func loadConfig(filename string) error {
	return fmt.Errorf("unable to load config: %w", fmt.Errorf("file not found: %s", filename))
}

func main() {
	err := loadConfig("config.yaml")
	fmt.Println(err)
}
```

**Output:**
```
unable to load config: file not found: config.yaml
```

---

## **Exercise 5: Unwrapping Errors**

**Problem**: Use `errors.Unwrap` to extract the inner error.

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := fmt.Errorf("outer error: %w", errors.New("inner error"))
	fmt.Println("Full Error:", err)
	fmt.Println("Unwrapped Error:", errors.Unwrap(err))
}
```

**Output:**
```
Full Error: outer error: inner error
Unwrapped Error: inner error
```

---

## **Exercise 6: Checking Specific Errors**

**Problem**: Use `errors.Is` to check for specific errors.

```go
package main

import (
	"errors"
	"fmt"
)

var ErrNotFound = errors.New("not found")

func findResource(id int) error {
	if id != 1 {
		return fmt.Errorf("resource lookup failed: %w", ErrNotFound)
	}
	return nil
}

func main() {
	err := findResource(2)
	if errors.Is(err, ErrNotFound) {
		fmt.Println("Specific error: resource not found")
	}
}
```

**Output:**
```
Specific error: resource not found
```

---

## **Exercise 7: Using `errors.As` for Specific Error Types**

**Problem**: Extract specific error details with `errors.As`.

```go
package main

import (
	"errors"
	"fmt"
)

type ConfigError struct {
	File string
}

func (e *ConfigError) Error() string {
	return fmt.Sprintf("config error in file %s", e.File)
}

func loadApp() error {
	return fmt.Errorf("application failed: %w", &ConfigError{File: "app.yaml"})
}

func main() {
	err := loadApp()
	var configErr *ConfigError
	if errors.As(err, &configErr) {
		fmt.Println("Config Error in:", configErr.File)
	}
}
```

**Output:**
```
Config Error in: app.yaml
```

---

## **Exercise 8: Reuse Error Types**

**Problem**: Reuse custom error types across different functions.

```go
package main

import (
	"errors"
	"fmt"
)

var ErrPermissionDenied = errors.New("permission denied")

func openFile(filename string) error {
	return fmt.Errorf("failed to open file: %w", ErrPermissionDenied)
}

func saveFile(filename string) error {
	return fmt.Errorf("failed to save file: %w", ErrPermissionDenied)
}

func main() {
	err := openFile("example.txt")
	if errors.Is(err, ErrPermissionDenied) {
		fmt.Println("Permission issue while opening the file")
	}

	err = saveFile("example.txt")
	if errors.Is(err, ErrPermissionDenied) {
		fmt.Println("Permission issue while saving the file")
	}
}
```

**Output:**
```
Permission issue while opening the file
Permission issue while saving the file
```

---

## **Exercise 9: Logging Errors**

**Problem**: Log errors with additional context.

```go
package main

import (
	"fmt"
	"log"
)

func processTask(id int) error {
	if id < 0 {
		return fmt.Errorf("invalid task ID: %d", id)
	}
	return nil
}

func main() {
	err := processTask(-1)
	if err != nil {
		log.Printf("Task Processing Failed: %v", err)
	}
}
```

**Output:**
```
2024/12/06 12:00:00 Task Processing Failed: invalid task ID: -1
```

---

## **Exercise 10: Graceful Shutdown with Error Handling**

**Problem**: Ensure graceful cleanup during errors.

```go
package main

import (
	"errors"
	"fmt"
)

func performTask() error {
	defer fmt.Println("Cleaning up resources...")
	return errors.New("task failed")
}

func main() {
	err := performTask()
	if err != nil {
		fmt.Println("Error:", err)
	}
}
```

**Output:**
```
Cleaning up resources...
Error: task failed
```

---

**Congratulations!** You’ve mastered error handling in Go. These examples cover a wide range of scenarios, helping you build robust and reliable applications.


---

**Happy Coding!** 💻✨
