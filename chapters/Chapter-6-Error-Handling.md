
# **Chapter 6: Error Handling in Go**

Error handling is a core principle of Go’s design philosophy. Instead of exceptions, Go uses explicit error values to handle issues gracefully. In this chapter, you'll learn how to return errors, create custom error types, and use error wrapping and unwrapping to build robust and maintainable applications.

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

## **Chapter Summary**

- Go's explicit error handling ensures clarity and robustness.
- Use custom error types for richer context.
- Error wrapping and unwrapping simplify debugging and error analysis.

**Next Steps**: Experiment with creating your own error-handling scenarios and integrate them into real-world projects!

---

**Happy Coding!** 💻✨
