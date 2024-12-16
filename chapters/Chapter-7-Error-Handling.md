# **Chapter 7: Error Handling in Go**

Error handling is a crucial aspect of any programming language, and Go provides a simple yet powerful way to handle errors. This chapter will take you through the core concepts of error handling in Go, explain them clearly, and then walk you through detailed examples to help you master the technique.

---

### **7.1 Basic Concept of Error Handling in Go**

In Go, errors are treated as values. Functions that can result in errors usually return two values:

1. The result of the operation (e.g., a number, a string, etc.)
2. An `error` type, which represents any issues encountered.

The `error` type is an interface that has a single method: `Error() string`. This method returns the error message.

### **Key Points:**

- Errors in Go are usually returned as the second value in a function.
- You need to check the error explicitly after each operation to handle it properly.
- Go does not have exceptions, so you handle errors using conditional statements.

---

### **7.2 Returning Errors**

In Go, you can return errors from functions, which allows the caller to handle them appropriately. Let's start by examining a simple example of how errors can be returned from a function.

#### **Example 1: A Simple Division Function**

The `divide` function checks if the divisor is zero. If it is, it returns an error to signal that division by zero is not possible. Otherwise, it returns the result of the division.

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

**Explanation:**

- The `divide` function checks if the divisor (`b`) is zero. If so, it returns an error.
- In the `main` function, we immediately check the error. If an error is returned, it is printed out. Otherwise, the result of the division is displayed.

**Output:**

```
Error: cannot divide by zero
```

This example demonstrates the basic error handling pattern in Go: check for an error and handle it.

---

### **7.3 Error Handling with Multiple Return Values**

Go encourages returning multiple values, and error handling is no different. Functions often return both the result and an error, which must be checked in the calling code.

#### **Example 2: Handling Errors with Multiple Return Values**

Here’s an example where we try to read from a file. If the file doesn’t exist, an error is returned.

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

**Explanation:**

- The `readFile` function tries to read a file and returns two values: the content of the file and any error that may occur.
- If an error occurs (such as the file not existing), it is returned to the caller, where it is checked.

**Output:**

```
Error: open nonexistent.txt: no such file or directory
```

---

### **7.4 Custom Error Types**

While the built-in `error` type is often sufficient, you can also create custom error types to provide more context or specific information about the error.

#### **Example 3: Creating a Custom Error Type**

In this example, we define a custom error type for division errors, which includes additional details like the dividend and divisor.

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

**Explanation:**

- The `DivideError` struct holds information about the dividend, divisor, and an error message.
- The `Error` method formats these details into a string, providing a more informative error message.

**Output:**

```
error: cannot divide by zero (dividend: 10.00, divisor: 0.00)
```

---

### **7.5 Wrapping Errors for More Context**

Go allows you to wrap errors with additional context. This helps preserve the original error while providing more details about the issue.

#### **Example 4: Wrapping Errors**

Here, we wrap a custom error to provide more context about a configuration issue.

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

**Explanation:**

- We use `fmt.Errorf` to wrap an error with additional context. The `%w` verb is used to retain the original error within the new error.
- This makes it easier to trace the error chain while still providing helpful context.

**Output:**

```
failed to read config: file not found: config.yaml
```

---

### **7.6 Unwrapping Errors**

If an error has been wrapped, you can retrieve the original error using `errors.Unwrap`.

#### **Example 5: Unwrapping Errors**

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

**Explanation:**

- `errors.Unwrap` allows you to retrieve the original error that was wrapped.
- This is useful when you need to inspect or handle the original error separately.

**Output:**

```
Full Error: outer error: inner error
Inner Error: inner error
```

---

### **7.7 Checking Specific Errors with `errors.Is`**

You can use `errors.Is` to check if an error matches a specific error type, even if it’s wrapped.

#### **Example 6: Checking for Specific Errors**

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

**Explanation:**

- `errors.Is` checks if a specific error (`ErrPermissionDenied`) is part of the error chain.
- This allows you to handle specific error types even if they are wrapped in other errors.

**Output:**

```
Action failed due to permission issues.
```

---

### **7.8 Handling Multiple Error Layers with `errors.As`**

If you need to extract specific error types from a chain of errors, you can use `errors.As`.

#### **Example 7: Handling Multiple Error Layers**

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

**Explanation:**

- `errors.As` checks if the error can be converted to the `ConfigError` type, allowing us to access the specific fields of the error.
- This is useful when you want to retrieve more context or specific information about an error.

**Output:**

```
Specific Error: app.yaml
```

---

## **7.7 Exercises**

### **Exercise 1: Simple File Reading with Error Handling**

**Problem**: Write a function `readLineFromFile` that takes a filename and reads the first line from the file. If the file does not exist, return an appropriate error.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func readLineFromFile(filename string) (string, error) {
	file, err := os.Open(filename)
	if err != nil {
		return "", err
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	if scanner.Scan() {
		return scanner.Text(), nil
	}
	return "", fmt.Errorf("file is empty")
}

func main() {
	line, err := readLineFromFile("test.txt")
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("First Line:", line)
	}
}
```

**Explanation:**

- The function `readLineFromFile` opens the file, reads the first line, and handles errors.
- The error is returned if the file can't be opened or if the file is empty.

**Output:**

```
Error: open test.txt: no such file or directory
```

---

### **Exercise 2: Division with Zero Error Check**

**Problem**: Write a function `safeDivide` that accepts two integers and returns their division result, handling division by zero errors appropriately.

```go
package main

import (
	"fmt"
	"errors"
)

func safeDivide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("division by zero")
	}
	return a / b, nil
}

func main() {
	result, err := safeDivide(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
}
```

**Explanation:**

- The `safeDivide` function checks if the divisor is zero, returning an error if so.
- The result is returned only if the error is nil.

**Output:**

```
Error: division by zero
```

---

### **Exercise 3: Error Handling in User Input**

**Problem**: Write a function `getUserInput` that asks the user for an integer input. If the user enters a non-integer value, return an error.

```go
package main

import (
	"fmt"
	"strconv"
)

func getUserInput() (int, error) {
	var input string
	fmt.Print("Enter an integer: ")
	fmt.Scanln(&input)
	number, err := strconv.Atoi(input)
	if err != nil {
		return 0, fmt.Errorf("invalid input: %v", err)
	}
	return number, nil
}

func main() {
	num, err := getUserInput()
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("You entered:", num)
	}
}
```

**Explanation:**

- The `getUserInput` function prompts the user to enter an integer and checks if the input can be converted.
- An error is returned if the input is not a valid integer.

**Output:**

```
Enter an integer: abc
Error: invalid input: strconv.Atoi: parsing "abc": invalid syntax
```

---

### **Exercise 4: File Copying with Error Handling**

**Problem**: Write a function `copyFile` that copies the contents of one file to another. If the source file does not exist or any other error occurs during copying, handle and return an appropriate error.

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func copyFile(src, dst string) error {
	sourceFile, err := os.Open(src)
	if err != nil {
		return err
	}
	defer sourceFile.Close()

	destFile, err := os.Create(dst)
	if err != nil {
		return err
	}
	defer destFile.Close()

	_, err = io.Copy(destFile, sourceFile)
	if err != nil {
		return err
	}
	return nil
}

func main() {
	err := copyFile("source.txt", "destination.txt")
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("File copied successfully")
	}
}
```

**Explanation:**

- The `copyFile` function opens the source file, creates the destination file, and copies the contents.
- Errors are returned if any file operation fails.

**Output:**

```
Error: open source.txt: no such file or directory
```

---

### **Exercise 5: Age Validation with Error Handling**

**Problem**: Write a function `validateAge` that checks if an age is within a valid range (0-120). If the age is invalid, return an error.

```go
package main

import (
	"fmt"
	"errors"
)

func validateAge(age int) error {
	if age < 0 || age > 120 {
		return errors.New("invalid age: must be between 0 and 120")
	}
	return nil
}

func main() {
	err := validateAge(150)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Age is valid")
	}
}
```

**Explanation:**

- The `validateAge` function checks if the age is within the allowed range. If not, it returns an error.

**Output:**

```
Error: invalid age: must be between 0 and 120
```
