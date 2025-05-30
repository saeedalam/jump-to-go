# **Chapter 7: Mastering Error Handling in Go**

Error handling is a crucial aspect of writing robust software. Go's approach to error handling is distinctive and pragmatic: errors are values that can be returned, checked, and handled explicitly. This philosophy encourages developers to thoughtfully address failure scenarios, leading to more reliable and maintainable code.

In this chapter, we'll explore Go's error handling mechanisms in depth, from basic patterns to advanced techniques used in production applications.

## **7.1 Go's Error Handling Philosophy**

### **7.1.1 Errors as Values**

Unlike many programming languages that use exceptions for error handling, Go treats errors as ordinary values. This fundamental design choice has several important implications:

- **Explicit error checking**: Errors must be explicitly checked and handled
- **Predictable control flow**: No hidden "throw" statements that might redirect execution
- **Composition over inheritance**: Error types can be composed and wrapped
- **Simplicity**: A single, consistent pattern for handling errors

The standard `error` interface in Go is remarkably simple:

```go
type error interface {
    Error() string
}
```

Any type that implements this interface can be used as an error. This minimalist approach provides great flexibility while maintaining clarity.

### **7.1.2 The Multiple Return Value Pattern**

The most common error handling pattern in Go uses multiple return values:

```go
func DoSomething() (Result, error) {
    // Implementation...
    if problem {
        return ZeroValue, errors.New("something went wrong")
    }
    return result, nil
}

// Usage
result, err := DoSomething()
if err != nil {
    // Handle error
    return // Or continue with a fallback strategy
}
// Use result...
```

This pattern makes error handling visible and encourages developers to consider failure cases explicitly.

## **7.2 Creating and Returning Errors**

### **7.2.1 Simple Error Creation**

The standard library provides several ways to create errors:

```go
package main

import (
    "errors"
    "fmt"
)

func main() {
    // Method 1: Using errors.New
    err1 := errors.New("something went wrong")

    // Method 2: Using fmt.Errorf (allows formatting)
    name := "config file"
    err2 := fmt.Errorf("failed to load %s", name)

    fmt.Println(err1) // something went wrong
    fmt.Println(err2) // failed to load config file
}
```

### **7.2.2 Custom Error Types**

For more sophisticated error handling, you can create custom error types:

```go
package main

import (
    "fmt"
)

// ValidationError represents an input validation error
type ValidationError struct {
    Field string
    Message string
}

// Implement the error interface
func (e ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Message)
}

// Function that returns custom error
func ValidateUsername(username string) error {
    if len(username) < 3 {
        return ValidationError{
            Field: "username",
            Message: "must be at least 3 characters long",
        }
    }
    return nil
}

func main() {
    err := ValidateUsername("ab")
    if err != nil {
        fmt.Println(err) // validation error on field username: must be at least 3 characters long

        // Type assertion to access specific fields
        if valErr, ok := err.(ValidationError); ok {
            fmt.Printf("Field: %s, Message: %s\n", valErr.Field, valErr.Message)
        }
    }
}
```

Custom error types allow you to:

- Include structured data in your errors
- Create type hierarchies for different error categories
- Enable type assertions for specific error handling

### **7.2.3 Sentinel Errors**

For errors that need to be checked by identity rather than content, you can define package-level "sentinel" error variables:

```go
package userservice

import "errors"

// Public sentinel errors that can be checked by callers
var (
    ErrUserNotFound = errors.New("user not found")
    ErrPermissionDenied = errors.New("permission denied")
    ErrInvalidInput = errors.New("invalid input")
)

func GetUser(id string) (*User, error) {
    // Implementation...
    if userDoesNotExist {
        return nil, ErrUserNotFound
    }
    // ...
}

// Usage in another package
user, err := userservice.GetUser("123")
if err == userservice.ErrUserNotFound {
    // Handle specifically for user not found
} else if err != nil {
    // Handle other errors
}
```

Sentinel errors are particularly useful for expected error conditions that callers might want to handle specifically.

## **7.3 Error Handling Patterns**

### **7.3.1 The Guard Clause Pattern**

A common pattern in Go is to use "guard clauses" to handle errors early and reduce nesting:

```go
// Without guard clauses (deeply nested)
func ProcessFile(path string) error {
    file, err := os.Open(path)
    if err == nil {
        defer file.Close()

        data, err := io.ReadAll(file)
        if err == nil {
            config, err := parseConfig(data)
            if err == nil {
                return processConfig(config)
            } else {
                return fmt.Errorf("parsing config: %w", err)
            }
        } else {
            return fmt.Errorf("reading file: %w", err)
        }
    } else {
        return fmt.Errorf("opening file: %w", err)
    }
}

// With guard clauses (flat and clean)
func ProcessFile(path string) error {
    file, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening file: %w", err)
    }
    defer file.Close()

    data, err := io.ReadAll(file)
    if err != nil {
        return fmt.Errorf("reading file: %w", err)
    }

    config, err := parseConfig(data)
    if err != nil {
        return fmt.Errorf("parsing config: %w", err)
    }

    return processConfig(config)
}
```

The guard clause pattern creates a flatter function structure that's easier to read and reason about.

### **7.3.2 Wrapping Errors for Context**

Go 1.13 introduced error wrapping, which allows you to add context while preserving the original error:

```go
package main

import (
    "errors"
    "fmt"
)

func readConfig(path string) error {
    // Simulate a "file not found" error
    err := errors.New("file not found")

    // Wrap the error with additional context
    return fmt.Errorf("failed to read config from %s: %w", path, err)
}

func setupApp() error {
    err := readConfig("/etc/myapp/config.json")
    if err != nil {
        // Wrap again with higher-level context
        return fmt.Errorf("app initialization failed: %w", err)
    }
    return nil
}

func main() {
    err := setupApp()
    fmt.Println(err)
    // Output: app initialization failed: failed to read config from /etc/myapp/config.json: file not found
}
```

The `%w` verb in `fmt.Errorf` wraps the original error, preserving it for later inspection while adding context at each level.

### **7.3.3 Working with Wrapped Errors**

Go provides functions to work with wrapped errors:

```go
package main

import (
    "errors"
    "fmt"
    "os"
)

func main() {
    // Create an original error
    originalErr := os.ErrNotExist

    // Wrap it twice
    wrappedOnce := fmt.Errorf("could not open config: %w", originalErr)
    wrappedTwice := fmt.Errorf("initialization failed: %w", wrappedOnce)

    // Unwrap once
    unwrappedOnce := errors.Unwrap(wrappedTwice)
    fmt.Println(unwrappedOnce) // could not open config: file does not exist

    // Check if wrappedTwice contains originalErr
    if errors.Is(wrappedTwice, os.ErrNotExist) {
        fmt.Println("The root cause is a file not found error")
    }
}
```

### **7.3.4 Type Assertions with errors.As**

The `errors.As` function helps you check if an error is of a specific type, even when wrapped:

```go
package main

import (
    "errors"
    "fmt"
    "net"
)

func fetchData(url string) error {
    // Simulate a DNS resolution error
    return fmt.Errorf("connection failed: %w", &net.DNSError{
        Err:       "no such host",
        Name:      "example.com",
        IsTimeout: false,
    })
}

func main() {
    err := fetchData("https://example.com")

    // Check if the error is a DNSError
    var dnsErr *net.DNSError
    if errors.As(err, &dnsErr) {
        fmt.Printf("DNS error for host %s: %s\n", dnsErr.Name, dnsErr.Err)
    } else {
        fmt.Println("Unknown error:", err)
    }
}
```

The `errors.As` function unwraps the error chain until it finds an error that can be assigned to the target type.

## **7.4 Advanced Error Handling Techniques**

### **7.4.1 Structured Error Types**

For APIs and libraries, structured errors provide detailed information about what went wrong:

```go
package main

import (
    "encoding/json"
    "fmt"
)

// AppError represents an application-specific error
type AppError struct {
    Code    string `json:"code"`
    Message string `json:"message"`
    Details any    `json:"details,omitempty"`
}

// Implement the error interface
func (e AppError) Error() string {
    return fmt.Sprintf("[%s] %s", e.Code, e.Message)
}

// Helper functions to create specific errors
func NotFoundError(resource string, id string) AppError {
    return AppError{
        Code:    "NOT_FOUND",
        Message: fmt.Sprintf("%s with ID %s not found", resource, id),
        Details: map[string]string{
            "resource": resource,
            "id":       id,
        },
    }
}

func main() {
    // Create a structured error
    err := NotFoundError("User", "123")

    // Print as string
    fmt.Println(err)

    // Convert to JSON for API responses
    jsonBytes, _ := json.MarshalIndent(err, "", "  ")
    fmt.Println(string(jsonBytes))
}
```

Structured errors are especially useful for:

- API responses where error details need to be serialized
- Error categorization and consistent handling
- Providing rich debugging information

### **7.4.2 Error Handling with Cleanup: defer**

The `defer` statement ensures cleanup code runs even when errors occur:

```go
package main

import (
    "fmt"
    "os"
)

func processFile(path string) error {
    // Open file
    file, err := os.Open(path)
    if err != nil {
        return fmt.Errorf("opening file: %w", err)
    }
    // Schedule file closing regardless of what happens next
    defer file.Close()

    // Process file contents...
    // Any errors here won't prevent file.Close() from being called

    return nil
}
```

Using `defer` with error handling ensures resources are properly cleaned up regardless of error paths.

### **7.4.3 Custom Error Hierarchies**

For complex applications, you might want to create an error hierarchy:

```go
package main

import (
    "errors"
    "fmt"
    "net/http"
)

// Base error type
type AppError struct {
    Err     error
    Message string
    Code    int
}

func (e AppError) Error() string {
    return e.Message
}

func (e AppError) Unwrap() error {
    return e.Err
}

// Specialized error types
type ValidationError struct {
    AppError
    Field string
}

type AuthorizationError struct {
    AppError
    UserID string
}

// Error constructors
func NewValidationError(field, message string) ValidationError {
    return ValidationError{
        AppError: AppError{
            Message: message,
            Code:    http.StatusBadRequest,
        },
        Field: field,
    }
}

func NewAuthorizationError(userID string) AuthorizationError {
    return AuthorizationError{
        AppError: AppError{
            Message: "unauthorized access",
            Code:    http.StatusUnauthorized,
        },
        UserID: userID,
    }
}

func main() {
    // Create specialized errors
    err1 := NewValidationError("email", "invalid email format")
    err2 := NewAuthorizationError("user123")

    // Handle errors
    handleError(err1)
    handleError(err2)
}

func handleError(err error) {
    // Check error types
    var validationErr ValidationError
    var authErr AuthorizationError

    switch {
    case errors.As(err, &validationErr):
        fmt.Printf("Bad request (field %s): %s\n", validationErr.Field, validationErr.Message)
    case errors.As(err, &authErr):
        fmt.Printf("Unauthorized: User %s lacks permission\n", authErr.UserID)
    default:
        fmt.Println("Unknown error:", err)
    }
}
```

This approach allows for:

- Rich error information
- Type-based error handling
- Consistent error response codes
- Proper error wrapping and unwrapping

## **7.5 Practical Error Handling Examples**

### **7.5.1 File Operations**

```go
package main

import (
    "errors"
    "fmt"
    "io"
    "os"
    "path/filepath"
)

func CopyFile(src, dst string) error {
    // Validate inputs
    if src == "" || dst == "" {
        return errors.New("source and destination paths cannot be empty")
    }

    // Check if source exists
    srcInfo, err := os.Stat(src)
    if err != nil {
        if os.IsNotExist(err) {
            return fmt.Errorf("source file does not exist: %w", err)
        }
        return fmt.Errorf("checking source file: %w", err)
    }

    // Ensure source is a regular file
    if !srcInfo.Mode().IsRegular() {
        return fmt.Errorf("%s is not a regular file", src)
    }

    // Create destination directory if needed
    dstDir := filepath.Dir(dst)
    err = os.MkdirAll(dstDir, 0755)
    if err != nil {
        return fmt.Errorf("creating destination directory %s: %w", dstDir, err)
    }

    // Open source file
    srcFile, err := os.Open(src)
    if err != nil {
        return fmt.Errorf("opening source file: %w", err)
    }
    defer srcFile.Close()

    // Create destination file
    dstFile, err := os.Create(dst)
    if err != nil {
        return fmt.Errorf("creating destination file: %w", err)
    }
    defer func() {
        closeErr := dstFile.Close()
        if err == nil && closeErr != nil {
            err = fmt.Errorf("closing destination file: %w", closeErr)
        }
    }()

    // Copy content
    _, err = io.Copy(dstFile, srcFile)
    if err != nil {
        return fmt.Errorf("copying file content: %w", err)
    }

    return nil
}

func main() {
    err := CopyFile("source.txt", "dest/destination.txt")
    if err != nil {
        fmt.Printf("Error copying file: %v\n", err)
        os.Exit(1)
    }
    fmt.Println("File copied successfully")
}
```

This example demonstrates comprehensive error handling for a file operation, including:

- Input validation
- Multiple error checks at different stages
- Proper error wrapping with context
- Resource cleanup with `defer`
- Handling errors from deferred operations

### **7.5.2 Database Operations**

```go
package main

import (
    "context"
    "database/sql"
    "errors"
    "fmt"
    "time"

    _ "github.com/go-sql-driver/mysql"
)

// User represents a user in our system
type User struct {
    ID        int
    Username  string
    Email     string
    CreatedAt time.Time
}

// NotFoundError indicates a resource was not found
type NotFoundError struct {
    Resource string
    ID       any
}

func (e NotFoundError) Error() string {
    return fmt.Sprintf("%s with ID %v not found", e.Resource, e.ID)
}

// UserRepository handles database operations for users
type UserRepository struct {
    db *sql.DB
}

func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}

func (r *UserRepository) GetByID(ctx context.Context, id int) (User, error) {
    query := "SELECT id, username, email, created_at FROM users WHERE id = ?"

    var user User
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.CreatedAt,
    )

    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return User{}, NotFoundError{Resource: "User", ID: id}
        }
        return User{}, fmt.Errorf("querying user by ID %d: %w", id, err)
    }

    return user, nil
}

func (r *UserRepository) Create(ctx context.Context, user User) (int, error) {
    query := "INSERT INTO users (username, email, created_at) VALUES (?, ?, ?)"

    result, err := r.db.ExecContext(ctx, query, user.Username, user.Email, time.Now())
    if err != nil {
        return 0, fmt.Errorf("creating user: %w", err)
    }

    id, err := result.LastInsertId()
    if err != nil {
        return 0, fmt.Errorf("getting inserted user ID: %w", err)
    }

    return int(id), nil
}

func main() {
    // Connect to database
    db, err := sql.Open("mysql", "user:password@/dbname")
    if err != nil {
        fmt.Printf("Error connecting to database: %v\n", err)
        return
    }
    defer db.Close()

    // Create repository
    userRepo := NewUserRepository(db)

    // Use repository with error handling
    ctx := context.Background()
    user, err := userRepo.GetByID(ctx, 123)

    if err != nil {
        var notFoundErr NotFoundError
        if errors.As(err, &notFoundErr) {
            fmt.Printf("User not found: %v\n", err)
        } else {
            fmt.Printf("Database error: %v\n", err)
        }
        return
    }

    fmt.Printf("Found user: %s (%s)\n", user.Username, user.Email)
}
```

This example shows:

- Custom error types for specific error conditions
- Error wrapping for database operations
- Using `errors.Is` and `errors.As` for error checking
- Context propagation with cancellation support
- Resource cleanup with `defer`

## **7.6 Best Practices for Error Handling**

### **7.6.1 Error Handling Guidelines**

1. **Be explicit**: Always check errors and handle them appropriately
2. **Add context**: Wrap errors with additional information when returning them up the call stack
3. **Don't just log and return**: Either handle the error or return it, but avoid doing both
4. **Keep the happy path clean**: Use guard clauses to handle errors early
5. **Use sentinel errors sparingly**: Reserve them for errors that callers need to check specifically
6. **Provide actionable error messages**: Error messages should help users understand what went wrong and how to fix it
7. **Design for error flows**: Consider error paths as carefully as success paths

### **7.6.2 Error Handling Antipatterns**

**Antipattern 1: Ignoring errors**

```go
// BAD
file, _ := os.Open("config.txt") // Ignoring the error
data := readData(file)

// GOOD
file, err := os.Open("config.txt")
if err != nil {
    log.Fatalf("Failed to open config file: %v", err)
}
data := readData(file)
```

**Antipattern 2: Shadowing errors**

```go
// BAD
if err := doThing1(); err != nil {
    return err
}
if err := doThing2(); err != nil {
    // The outer 'err' variable is shadowed by this new 'err'
    return err // Only returns the error from doThing2
}

// GOOD
err := doThing1()
if err != nil {
    return fmt.Errorf("thing1 failed: %w", err)
}
err = doThing2()
if err != nil {
    return fmt.Errorf("thing2 failed: %w", err)
}
```

**Antipattern 3: Excessive nesting**

```go
// BAD
func processData() error {
    err := step1()
    if err == nil {
        err = step2()
        if err == nil {
            err = step3()
            if err == nil {
                return step4()
            }
            return err
        }
        return err
    }
    return err
}

// GOOD
func processData() error {
    if err := step1(); err != nil {
        return err
    }
    if err := step2(); err != nil {
        return err
    }
    if err := step3(); err != nil {
        return err
    }
    return step4()
}
```

**Antipattern 4: Returning error strings instead of error values**

```go
// BAD
func validateInput(input string) (bool, string) {
    if len(input) < 3 {
        return false, "input too short"
    }
    return true, ""
}

// GOOD
func validateInput(input string) error {
    if len(input) < 3 {
        return errors.New("input too short")
    }
    return nil
}
```

## **7.7 Error Handling in the Real World**

### **7.7.1 Production-Ready Error Handling**

In production applications, error handling often involves:

1. **Structured logging**: Log errors with context, request IDs, and timestamps
2. **Error classification**: Categorize errors for appropriate handling (e.g., client errors vs. server errors)
3. **Retry mechanisms**: Implement backoff strategies for transient errors
4. **Circuit breakers**: Prevent cascading failures when dependencies fail
5. **Monitoring and alerting**: Track error rates and set up alerts for critical errors

### **7.7.2 API Error Responses**

When building APIs, convert internal errors to appropriate responses:

```go
package main

import (
    "encoding/json"
    "errors"
    "fmt"
    "log"
    "net/http"
)

// ErrorResponse represents an error in API responses
type ErrorResponse struct {
    Status  int    `json:"status"`
    Code    string `json:"code"`
    Message string `json:"message"`
}

// ValidationError represents input validation errors
type ValidationError struct {
    Field   string
    Message string
}

func (e ValidationError) Error() string {
    return fmt.Sprintf("validation failed on field %s: %s", e.Field, e.Message)
}

// NotFoundError represents resource not found errors
type NotFoundError struct {
    Resource string
}

func (e NotFoundError) Error() string {
    return fmt.Sprintf("%s not found", e.Resource)
}

// HTTP handler with error handling
func getUserHandler(w http.ResponseWriter, r *http.Request) {
    // Extract user ID from request
    userID := r.URL.Query().Get("id")
    if userID == "" {
        handleError(w, ValidationError{
            Field:   "id",
            Message: "user ID is required",
        })
        return
    }

    // Attempt to find user
    user, err := findUser(userID)
    if err != nil {
        handleError(w, err)
        return
    }

    // Return successful response
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

// Convert internal errors to appropriate HTTP responses
func handleError(w http.ResponseWriter, err error) {
    // Log the detailed error for internal debugging
    log.Printf("Error: %v", err)

    // Create appropriate response based on error type
    var resp ErrorResponse

    var validationErr ValidationError
    var notFoundErr NotFoundError

    switch {
    case errors.As(err, &validationErr):
        resp = ErrorResponse{
            Status:  http.StatusBadRequest,
            Code:    "VALIDATION_ERROR",
            Message: err.Error(),
        }
    case errors.As(err, &notFoundErr):
        resp = ErrorResponse{
            Status:  http.StatusNotFound,
            Code:    "NOT_FOUND",
            Message: err.Error(),
        }
    default:
        // For unexpected errors, don't expose internal details
        resp = ErrorResponse{
            Status:  http.StatusInternalServerError,
            Code:    "INTERNAL_ERROR",
            Message: "An internal error occurred",
        }
    }

    // Send error response
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(resp.Status)
    json.NewEncoder(w).Encode(resp)
}

func findUser(id string) (map[string]string, error) {
    // Simulate a database lookup
    if id == "123" {
        return map[string]string{
            "id":   "123",
            "name": "Alice",
        }, nil
    }
    return nil, NotFoundError{Resource: "User"}
}

func main() {
    http.HandleFunc("/user", getUserHandler)
    http.ListenAndServe(":8080", nil)
}
```

This example demonstrates:

- Converting internal errors to appropriate HTTP responses
- Preserving error information for logging
- Hiding sensitive details from users
- Type-based error handling

## **7.8 Practice Exercises**

### **Exercise 1: Enhanced File Processing**

Create a function that reads a CSV file, validates its structure, and processes the data. Implement comprehensive error handling that:

- Validates the file exists and is readable
- Checks that the CSV has the expected headers
- Validates data in each row
- Returns appropriate errors with context

### **Exercise 2: Error Hierarchy**

Design an error hierarchy for a banking application with at least three error types:

- `InsufficientFundsError`
- `AccountNotFoundError`
- `TransactionError`

Implement functions that return these errors and demonstrate how to handle them specifically.

### **Exercise 3: API Error Handling**

Create a simple HTTP API with at least two endpoints that demonstrate proper error handling, including:

- Input validation errors
- Not found errors
- Server errors
- Appropriate status codes and response formats

### **Exercise 4: Custom Error Type with Metadata**

Design a custom error type that includes metadata like:

- Error code
- Severity level
- Timestamp
- Request ID
- Suggestion for resolution

Use this error type in a function and demonstrate how to extract and use the metadata.

## **7.9 Summary**

In this chapter, we've explored Go's approach to error handling in depth:

- **Go treats errors as values**, using a simple interface and multiple return values
- **Error creation and handling** is explicit and straightforward
- **Error wrapping** helps add context while preserving the original error
- **Structured errors** can provide rich information for debugging and user feedback
- **Advanced patterns** like error hierarchies enable sophisticated error handling

By following Go's error handling philosophy and best practices, you can write code that is robust, maintainable, and communicates failures clearly. Remember that good error handling is not just about preventing crashes—it's about designing your application to degrade gracefully when things go wrong and providing meaningful feedback to users and developers.

**Next Up**: In Chapter 8, we'll explore arrays, slices, and strings—essential data structures for working with collections and text in Go.
