# **Chapter 22: Writing Clean Code in Go**

Writing clean code is an essential skill for professional developers. Clean code is not just about functionality—it's about crafting code that is readable, maintainable, and adaptable to changing requirements. Go, with its emphasis on simplicity and clarity, provides an excellent foundation for writing clean code. This chapter explores principles and practices that will help you write code that is not only functional but also clean, expressive, and maintainable.

## **22.1 Principles of Clean Code**

### **22.1.1 What Makes Code "Clean"?**

Clean code possesses several key characteristics:

1. **Readability**: Code should be easy to read and understand by other developers (and your future self).
2. **Simplicity**: Code should be as simple as possible, but no simpler. Avoid unnecessary complexity.
3. **Maintainability**: Code should be easy to modify and extend.
4. **Testability**: Code should be structured to facilitate testing.
5. **Consistency**: Code should follow consistent patterns and conventions.

As Robert C. Martin ("Uncle Bob") wrote: "Clean code is code that has been taken care of. Someone has taken the time to keep it simple and orderly. They have paid appropriate attention to details. They have cared."

### **22.1.2 Why Clean Code Matters**

Writing clean code provides numerous benefits:

- **Reduced Maintenance Costs**: Clean code is easier to maintain and extend, reducing long-term costs.
- **Improved Collaboration**: Team members can understand and contribute to clean code more effectively.
- **Reduced Bugs**: Clean code tends to have fewer bugs and makes bugs easier to find and fix.
- **Increased Development Speed**: While writing clean code may take slightly longer initially, it speeds up development over time.
- **Better Onboarding**: New team members can understand clean code more quickly.

### **22.1.3 The Boy Scout Rule**

The Boy Scout Rule is a simple but powerful principle: "Leave the code better than you found it."

Every time you work with a piece of code, make small improvements to its cleanliness. Over time, these incremental improvements add up to significantly cleaner code.

```go
// Before: Cryptic variable names
func c(a, b int) int {
    r := 0
    for i := 0; i < b; i++ {
        r += a
    }
    return r
}

// After: Clear names and simplified implementation
func multiply(multiplicand, multiplier int) int {
    return multiplicand * multiplier
}
```

## **22.2 Clean Code in Go**

Go was designed with simplicity and readability in mind, making it particularly well-suited for writing clean code.

### **22.2.1 Go's Philosophy**

Go's design philosophy aligns well with clean code principles:

1. **Simplicity**: Go avoids unnecessary features and complexity.
2. **Readability**: Go emphasizes readable code with minimal syntax.
3. **Explicitness**: Go prefers explicit over implicit behavior.
4. **Conciseness**: Go encourages brevity without sacrificing clarity.

These principles are embodied in the Go Proverbs, a set of guidelines articulated by Rob Pike:

- "Clear is better than clever."
- "Errors are values."
- "Don't panic."
- "A little copying is better than a little dependency."

### **22.2.2 Go's Standard Library as a Model**

Go's standard library provides excellent examples of clean code. Study it to learn good practices:

- **Consistent Style**: The standard library follows consistent naming and formatting conventions.
- **Clear Documentation**: Every exported function and type is well-documented.
- **Simple Interfaces**: Interfaces are small and focused on behavior.
- **Error Handling**: Errors are returned explicitly and handled appropriately.

```go
// From the standard library (net/http)
// ListenAndServe starts an HTTP server with a given address and handler.
// The handler is usually nil, which means to use DefaultServeMux.
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}
```

## **22.3 Naming Conventions**

Effective naming is one of the most important aspects of clean code. Good names make code self-documenting.

### **22.3.1 General Naming Principles**

1. **Be Descriptive**: Names should describe what a variable, function, or type represents or does.
2. **Use Domain Terminology**: Incorporate terms from the problem domain to make code more meaningful.
3. **Be Consistent**: Follow consistent naming patterns throughout your codebase.
4. **Avoid Abbreviations**: Unless they're universally understood (like HTTP or URL).
5. **Context Matters**: Consider the context when choosing name length. Broader scope = more descriptive name.

### **22.3.2 Naming Conventions in Go**

Go has specific naming conventions that contribute to code cleanliness:

1. **Use camelCase**: Variables and functions use camelCase (e.g., `myVariable`).
2. **Use PascalCase for Exported Names**: Exported (public) identifiers start with a capital letter (e.g., `MyFunction`).
3. **Use Short Names for Limited Scope**: Short variables like `i` are fine for loops with small scope.
4. **Use Single-letter Variables Sparingly**: Only for conventional cases like loop indices.
5. **Avoid Stutter**: Don't repeat package names in identifiers (e.g., `strings.StringReader` → `strings.Reader`).

```go
// Poor naming
func CalcThing(a, b int) int {
    var res int
    res = a + b
    return res
}

// Better naming
func CalculateSum(firstNumber, secondNumber int) int {
    sum := firstNumber + secondNumber
    return sum
}
```

### **22.3.3 Package Naming**

Package names are particularly important in Go:

1. **Use Short, Concise Names**: Package names should be short and meaningful.
2. **Use Lowercase**: All package names should be lowercase.
3. **Single Word is Preferable**: Avoid underscores or mixed caps.
4. **Be Descriptive**: Choose a name that describes the package's purpose.
5. **Avoid Generic Names**: Names like `util` or `common` don't provide meaningful context.

```go
// Good package names
package user
package postgres
package parser

// Poor package names
package userStuff   // Mixed case
package user_auth   // Underscore
package utilities   // Too generic
```

## **22.4 Function Design**

Functions are the building blocks of your code. Clean functions make for clean code.

### **22.4.1 Function Size and Responsibility**

1. **Do One Thing**: Functions should do one thing and do it well.
2. **Stay Small**: Keep functions short, ideally under 20-30 lines.
3. **Maintain a Single Level of Abstraction**: Don't mix high-level and low-level operations.
4. **Extract Till You Drop**: Continuously extract complex logic into well-named helper functions.

```go
// Poor: Function doing multiple things
func ProcessOrder(order Order) error {
    // Validate order
    if order.CustomerID == "" {
        return errors.New("missing customer ID")
    }
    if len(order.Items) == 0 {
        return errors.New("order has no items")
    }

    // Calculate total
    var total float64
    for _, item := range order.Items {
        total += item.Price * float64(item.Quantity)
    }

    // Apply discount
    if order.DiscountCode == "SUMMER20" {
        total *= 0.8 // 20% off
    }

    // Update inventory
    for _, item := range order.Items {
        err := updateInventory(item.ProductID, item.Quantity)
        if err != nil {
            return err
        }
    }

    // Save order
    order.Total = total
    return saveOrder(order)
}

// Better: Split into smaller, focused functions
func ProcessOrder(order Order) error {
    if err := validateOrder(order); err != nil {
        return err
    }

    total := calculateOrderTotal(order)
    total = applyDiscounts(total, order.DiscountCode)

    if err := updateInventoryForOrder(order); err != nil {
        return err
    }

    order.Total = total
    return saveOrder(order)
}

func validateOrder(order Order) error {
    if order.CustomerID == "" {
        return errors.New("missing customer ID")
    }
    if len(order.Items) == 0 {
        return errors.New("order has no items")
    }
    return nil
}

func calculateOrderTotal(order Order) float64 {
    var total float64
    for _, item := range order.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

func applyDiscounts(total float64, discountCode string) float64 {
    if discountCode == "SUMMER20" {
        return total * 0.8 // 20% off
    }
    return total
}

func updateInventoryForOrder(order Order) error {
    for _, item := range order.Items {
        if err := updateInventory(item.ProductID, item.Quantity); err != nil {
            return err
        }
    }
    return nil
}
```

### **22.4.2 Function Parameters**

1. **Limit Parameters**: Aim for 0-2 parameters, rarely more than 3.
2. **Use Structs for Multiple Parameters**: When you need many parameters, group them in a struct.
3. **Avoid Boolean Parameters**: They make function calls less clear. Consider separate functions.
4. **Prefer Input-Output Style**: Functions should take inputs and return outputs, minimizing side effects.

```go
// Poor: Too many parameters
func CreateUser(firstName, lastName, email, phone, address, city, state, zipCode string, age int) (*User, error) {
    // ...
}

// Better: Use a struct
type UserData struct {
    FirstName string
    LastName  string
    Email     string
    Phone     string
    Address   string
    City      string
    State     string
    ZipCode   string
    Age       int
}

func CreateUser(data UserData) (*User, error) {
    // ...
}

// Poor: Boolean parameter
func ProcessPayment(payment Payment, sendReceipt bool) error {
    // ...
}

// Better: Separate functions
func ProcessPayment(payment Payment) error {
    // ...
}

func ProcessPaymentAndSendReceipt(payment Payment) error {
    // ...
}
```

### **22.4.3 Return Values**

1. **Return Early**: Use early returns to handle errors and edge cases.
2. **Be Consistent with Error Returns**: In Go, return values followed by errors.
3. **Use Named Return Values Judiciously**: They can improve clarity in some cases.
4. **Return Zero Values**: Return zero values instead of nil when appropriate.

```go
// Poor: Nested conditionals
func ProcessTransaction(t Transaction) (Result, error) {
    if t.IsValid() {
        if t.Amount > 0 {
            if user, err := getUser(t.UserID); err == nil {
                if user.HasSufficientFunds(t.Amount) {
                    // Process the transaction
                    // ...
                    return Result{Success: true}, nil
                } else {
                    return Result{}, errors.New("insufficient funds")
                }
            } else {
                return Result{}, err
            }
        } else {
            return Result{}, errors.New("invalid amount")
        }
    } else {
        return Result{}, errors.New("invalid transaction")
    }
}

// Better: Early returns
func ProcessTransaction(t Transaction) (Result, error) {
    if !t.IsValid() {
        return Result{}, errors.New("invalid transaction")
    }

    if t.Amount <= 0 {
        return Result{}, errors.New("invalid amount")
    }

    user, err := getUser(t.UserID)
    if err != nil {
        return Result{}, err
    }

    if !user.HasSufficientFunds(t.Amount) {
        return Result{}, errors.New("insufficient funds")
    }

    // Process the transaction
    // ...

    return Result{Success: true}, nil
}
```

## **22.5 Error Handling**

Go's approach to error handling is a key part of writing clean code. Errors are values that are explicitly returned and handled.

### **22.5.1 Principles of Clean Error Handling**

1. **Check Errors Immediately**: Always check and handle errors as soon as they occur.
2. **Don't Ignore Errors**: Either handle them or propagate them up the call stack.
3. **Provide Context**: Add meaningful context when wrapping errors.
4. **Return Early on Errors**: Use error returns to simplify control flow.
5. **Use Specific Error Types**: Create custom error types for specific error conditions.

```go
// Poor: Ignoring errors
func ReadConfig(filename string) Config {
    data, _ := ioutil.ReadFile(filename)  // Error ignored!
    var config Config
    json.Unmarshal(data, &config)         // Error ignored!
    return config
}

// Better: Proper error handling
func ReadConfig(filename string) (Config, error) {
    data, err := ioutil.ReadFile(filename)
    if err != nil {
        return Config{}, fmt.Errorf("reading config file %s: %w", filename, err)
    }

    var config Config
    if err := json.Unmarshal(data, &config); err != nil {
        return Config{}, fmt.Errorf("parsing config file %s: %w", filename, err)
    }

    return config, nil
}
```

### **22.5.2 Error Wrapping**

Go 1.13 introduced the ability to wrap errors, which helps create more informative error chains:

```go
// Using fmt.Errorf with %w verb to wrap errors
func validateUser(user User) error {
    if user.Name == "" {
        return errors.New("name cannot be empty")
    }

    if user.Age < 18 {
        return errors.New("user must be at least 18 years old")
    }

    if err := validateEmail(user.Email); err != nil {
        return fmt.Errorf("invalid email: %w", err)
    }

    return nil
}

// Checking wrapped errors
func processUserRequest(userData string) error {
    user, err := parseUser(userData)
    if err != nil {
        return fmt.Errorf("processing user request: %w", err)
    }

    if err := validateUser(user); err != nil {
        return fmt.Errorf("processing user request: %w", err)
    }

    // Use errors.Is to check for specific error types
    if errors.Is(err, ErrInvalidEmail) {
        // Handle specifically...
    }

    // Use errors.As to access underlying error properties
    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        // Access validationErr fields...
    }

    return nil
}
```

### **22.5.3 Custom Error Types**

Define custom error types for specific error conditions:

```go
// Define custom error types
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Message)
}

// Create sentinel errors for specific conditions
var (
    ErrInvalidEmail = errors.New("invalid email format")
    ErrNotFound     = errors.New("resource not found")
)

// Use custom errors
func validateEmail(email string) error {
    if email == "" {
        return &ValidationError{
            Field:   "email",
            Message: "email cannot be empty",
        }
    }

    if !strings.Contains(email, "@") {
        return fmt.Errorf("%w: missing @ symbol", ErrInvalidEmail)
    }

    return nil
}
```

## **22.6 Comments and Documentation**

Good comments explain why, not what. The code itself should be clear enough to explain what it does.

### **22.6.1 Effective Comments**

1. **Comment on Why, Not How**: Explain the reasoning behind a decision, not the mechanics.
2. **Keep Comments Updated**: Outdated comments are worse than no comments.
3. **Use Complete Sentences**: Write clear, grammatically correct comments.
4. **Avoid Obvious Comments**: Don't state what is already obvious from the code.
5. **Comment Complex Algorithms**: Explain complex logic that isn't immediately obvious.

```go
// Poor: Obvious comment
// Increment counter by 1
counter++

// Poor: Comment explaining what code does
// Loop through users and print their names
for _, user := range users {
    fmt.Println(user.Name)
}

// Good: Comment explaining why
// We need to process items in reverse order because dependencies
// must be resolved bottom-up
for i := len(items) - 1; i >= 0; i-- {
    processItem(items[i])
}
```

### **22.6.2 Go Doc Comments**

Go has a standardized approach to documentation comments:

1. **Document Exported Identifiers**: Every exported function, type, and variable should have a comment.
2. **Start with the Name**: Begin comments with the identifier's name.
3. **Use Full Sentences**: Write complete sentences with proper punctuation.
4. **Document Parameters and Return Values**: Explain what each parameter is for and what the function returns.

```go
// Poor documentation
// validates the user
func ValidateUser(u *User) error {
    // ...
}

// Good documentation
// ValidateUser checks if the user has valid data according to business rules.
// It returns a validation error describing any issues found.
func ValidateUser(user *User) error {
    // ...
}

// Package documentation
// Package userauth provides authentication and authorization for users.
//
// It supports OAuth2 and JWT-based authentication methods
// and integrates with various identity providers.
package userauth
```

### **22.6.3 Self-Documenting Code**

The best code is self-documenting. Aim to write code that is so clear it rarely needs comments:

1. **Use Descriptive Names**: Clear names reduce the need for comments.
2. **Extract Helper Functions**: Use well-named helper functions to explain intent.
3. **Use Enums and Constants**: Replace magic numbers and strings with named constants.
4. **Leverage Types**: Create custom types that make code more expressive.

```go
// Poor: Needs comments to explain
// Check if user is admin and has been active in the last 30 days
if user.Role == 1 && time.Since(user.LastActive) < 30*24*time.Hour {
    // ...
}

// Better: Self-documenting
const (
    RoleAdmin    = 1
    RoleRegular  = 2
    ActivePeriod = 30 * 24 * time.Hour  // 30 days
)

func isActiveAdmin(user User) bool {
    return user.Role == RoleAdmin && time.Since(user.LastActive) < ActivePeriod
}

// Usage
if isActiveAdmin(user) {
    // ...
}
```

## **22.7 Code Organization**

Clean code is well-organized at every level, from project structure to individual lines.

### **22.7.1 Project Structure**

Organize your Go projects in a clean, logical structure:

1. **Group by Feature**: Organize code by feature or domain, not by technical role.
2. **Keep Related Code Together**: Code that changes together should be together.
3. **Limit Package Size**: Keep packages to a manageable size with a single responsibility.
4. **Use a Standard Layout**: Consider following Go project layout conventions.
5. **Separate Interface from Implementation**: Define interfaces separately from implementations.

A typical Go project structure might look like:

```
myproject/
├── cmd/               # Command-line applications
│   └── myapp/         # Main application
│       └── main.go    # Entry point
├── internal/          # Private application code
│   ├── auth/          # Authentication package
│   ├── config/        # Configuration package
│   └── user/          # User management package
├── pkg/               # Public libraries
│   └── validator/     # Validation utilities
├── api/               # API definitions (e.g., protocol buffers)
├── web/               # Web assets
├── docs/              # Documentation
├── scripts/           # Build and deployment scripts
├── go.mod             # Module definition
└── go.sum             # Module checksums
```

### **22.7.2 Package Design**

Design packages with clear boundaries and responsibilities:

1. **Single Responsibility**: Each package should have a single, well-defined purpose.
2. **Minimize Public API**: Export only what's necessary; keep implementation details private.
3. **Avoid Circular Dependencies**: Design packages to form a directed acyclic graph.
4. **Provide Package Documentation**: Document the package's purpose and usage.
5. **Create Small, Focused Interfaces**: Define narrow interfaces at package boundaries.

```go
// Good package design: clear, focused responsibility
package validator

// Validator checks if a value meets certain criteria.
type Validator interface {
    Validate(value interface{}) error
}

// EmailValidator validates email addresses.
type EmailValidator struct{}

// Validate checks if the provided value is a valid email.
func (v EmailValidator) Validate(value interface{}) error {
    // Implementation...
}

// Export factory functions instead of constructors
func NewEmailValidator() Validator {
    return EmailValidator{}
}
```

### **22.7.3 File Organization**

Organize individual files for clarity and cohesion:

1. **Logical Grouping**: Group related declarations together.
2. **File Size Limits**: Keep files to a reasonable size (300-500 lines maximum).
3. **Consistent Order**: Follow a consistent order for declarations (types, constants, variables, functions).
4. **One File, One Purpose**: Each file should have a specific focus.
5. **Meaningful File Names**: Choose names that reflect file contents.

A typical Go file order:

```go
// Package declaration and documentation
package user

// Imports (grouped by standard library, third-party, internal)
import (
    "errors"
    "fmt"

    "github.com/google/uuid"

    "myapp/internal/validator"
)

// Constants
const (
    MinPasswordLength = 8
    MaxUsernameLength = 50
)

// Types
type User struct {
    ID       string
    Username string
    Email    string
    Password string
}

// Global variables (use sparingly)
var (
    ErrInvalidUsername = errors.New("invalid username")
    ErrInvalidPassword = errors.New("invalid password")
)

// Functions (ordered from high-level to low-level)
func NewUser(username, email, password string) (*User, error) {
    // Implementation...
}

func validateUsername(username string) error {
    // Implementation...
}

func validatePassword(password string) error {
    // Implementation...
}
```

## **22.8 SOLID Principles in Go**

The SOLID principles are a set of design principles that help create more maintainable, flexible, and scalable code. Let's see how to apply them in Go.

### **22.8.1 Single Responsibility Principle (SRP)**

A struct or function should have only one reason to change.

```go
// Poor: User struct has multiple responsibilities
type User struct {
    ID       string
    Username string
    Email    string
    Password string
}

func (u *User) Validate() error {
    // Validation logic
}

func (u *User) Save() error {
    // Database operations
}

func (u *User) SendWelcomeEmail() error {
    // Email sending logic
}

// Better: Separated responsibilities
type User struct {
    ID       string
    Username string
    Email    string
    Password string
}

type UserValidator struct{}

func (v UserValidator) Validate(user User) error {
    // Validation logic
}

type UserRepository struct {
    // Database connection
}

func (r UserRepository) Save(user User) error {
    // Database operations
}

type NotificationService struct {
    // Email client
}

func (s NotificationService) SendWelcomeEmail(user User) error {
    // Email sending logic
}
```

### **22.8.2 Open/Closed Principle (OCP)**

Software entities should be open for extension but closed for modification.

```go
// Poor: Need to modify this function when adding new payment methods
func ProcessPayment(payment Payment) error {
    switch payment.Type {
    case "credit_card":
        return processCreditCardPayment(payment)
    case "paypal":
        return processPayPalPayment(payment)
    default:
        return errors.New("unsupported payment type")
    }
}

// Better: Open for extension through interfaces
type PaymentProcessor interface {
    Process(payment Payment) error
}

type CreditCardProcessor struct{}

func (p CreditCardProcessor) Process(payment Payment) error {
    // Process credit card payment
    return nil
}

type PayPalProcessor struct{}

func (p PayPalProcessor) Process(payment Payment) error {
    // Process PayPal payment
    return nil
}

// Now we can add new payment processors without modifying existing code
type BitcoinProcessor struct{}

func (p BitcoinProcessor) Process(payment Payment) error {
    // Process Bitcoin payment
    return nil
}

func ProcessPayment(payment Payment, processor PaymentProcessor) error {
    return processor.Process(payment)
}
```

### **22.8.3 Liskov Substitution Principle (LSP)**

Subtypes must be substitutable for their base types without altering the correctness of the program.

```go
// Poor: Square violates LSP for Rectangle
type Rectangle struct {
    width  int
    height int
}

func (r *Rectangle) SetWidth(width int) {
    r.width = width
}

func (r *Rectangle) SetHeight(height int) {
    r.height = height
}

func (r Rectangle) Area() int {
    return r.width * r.height
}

type Square struct {
    Rectangle
}

// Violates LSP: Changes both dimensions
func (s *Square) SetWidth(width int) {
    s.width = width
    s.height = width  // Side effect!
}

func (s *Square) SetHeight(height int) {
    s.width = height  // Side effect!
    s.height = height
}

// Better: Use a common interface
type Shape interface {
    Area() int
}

type Rectangle struct {
    width  int
    height int
}

func NewRectangle(width, height int) Rectangle {
    return Rectangle{width: width, height: height}
}

func (r Rectangle) Area() int {
    return r.width * r.height
}

type Square struct {
    side int
}

func NewSquare(side int) Square {
    return Square{side: side}
}

func (s Square) Area() int {
    return s.side * s.side
}
```

### **22.8.4 Interface Segregation Principle (ISP)**

Clients should not be forced to depend on methods they do not use.

```go
// Poor: Monolithic interface
type Worker interface {
    DoWork()
    TakeBreak()
    GetPaid()
    FileExpenses()
}

// Better: Segregated interfaces
type Worker interface {
    DoWork()
}

type BreakTaker interface {
    TakeBreak()
}

type Payee interface {
    GetPaid()
}

type ExpenseFiler interface {
    FileExpenses()
}

// Types can implement only what they need
type Employee struct{}

func (e Employee) DoWork() {
    // Implementation
}

func (e Employee) TakeBreak() {
    // Implementation
}

func (e Employee) GetPaid() {
    // Implementation
}

type Contractor struct{}

func (c Contractor) DoWork() {
    // Implementation
}

func (c Contractor) GetPaid() {
    // Implementation
}

// No need to implement TakeBreak() or FileExpenses()
```

### **22.8.5 Dependency Inversion Principle (DIP)**

High-level modules should not depend on low-level modules. Both should depend on abstractions.

```go
// Poor: Direct dependency on implementation
type UserService struct {
    db *sql.DB
}

func (s UserService) CreateUser(user User) error {
    // Directly use the database
    _, err := s.db.Exec("INSERT INTO users (name, email) VALUES (?, ?)",
                       user.Name, user.Email)
    return err
}

// Better: Depend on abstraction
type UserRepository interface {
    Create(user User) error
    GetByID(id string) (User, error)
}

type UserService struct {
    repo UserRepository
}

func (s UserService) CreateUser(user User) error {
    return s.repo.Create(user)
}

// Implementation details are separated
type SQLUserRepository struct {
    db *sql.DB
}

func (r SQLUserRepository) Create(user User) error {
    _, err := r.db.Exec("INSERT INTO users (name, email) VALUES (?, ?)",
                       user.Name, user.Email)
    return err
}

func (r SQLUserRepository) GetByID(id string) (User, error) {
    // Implementation
    return User{}, nil
}
```

## **22.9 Testing Clean Code**

Clean code is inherently more testable. Testing is not an afterthought but an integral part of writing clean code.

### **22.9.1 Unit Testing Best Practices**

1. **Test One Thing**: Each test should verify one specific behavior.
2. **Use Clear Test Names**: Name tests to describe what they're testing.
3. **Follow the AAA Pattern**: Arrange, Act, Assert.
4. **Keep Tests Independent**: Tests should not depend on each other.
5. **Mock External Dependencies**: Use interfaces to mock external systems.

```go
// Clean test structure
func TestCalculateOrderTotal(t *testing.T) {
    // Arrange
    order := Order{
        Items: []Item{
            {ProductID: "p1", Price: 10.0, Quantity: 2},
            {ProductID: "p2", Price: 15.0, Quantity: 1},
        },
    }

    // Act
    total := calculateOrderTotal(order)

    // Assert
    expected := 35.0
    if total != expected {
        t.Errorf("Expected total %.2f, got %.2f", expected, total)
    }
}
```

### **22.9.2 Table-Driven Tests**

Table-driven tests are a clean way to test multiple cases with minimal code duplication:

```go
func TestValidateEmail(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        wantErr bool
    }{
        {"empty email", "", true},
        {"missing @", "userexample.com", true},
        {"valid email", "user@example.com", false},
        {"multiple @", "user@example@com", true},
        {"with spaces", "user @example.com", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validateEmail(tt.email)
            if (err != nil) != tt.wantErr {
                t.Errorf("validateEmail(%q) error = %v, wantErr %v",
                         tt.email, err, tt.wantErr)
            }
        })
    }
}
```

### **22.9.3 Test Helpers and Utilities**

Create clean helper functions to reduce test boilerplate:

```go
// Test helpers improve readability
func createTestUser(t *testing.T) User {
    t.Helper()
    return User{
        ID:       "user123",
        Username: "testuser",
        Email:    "test@example.com",
    }
}

func assertNoError(t *testing.T, err error) {
    t.Helper()
    if err != nil {
        t.Fatalf("Expected no error, got %v", err)
    }
}

func assertErrorContains(t *testing.T, err error, expected string) {
    t.Helper()
    if err == nil {
        t.Fatal("Expected an error but got none")
    }
    if !strings.Contains(err.Error(), expected) {
        t.Errorf("Expected error to contain %q, got %q", expected, err.Error())
    }
}

// Usage in tests
func TestSomething(t *testing.T) {
    user := createTestUser(t)
    err := validateUser(user)
    assertNoError(t, err)
}
```

## **22.10 Refactoring Towards Clean Code**

Clean code often evolves through deliberate refactoring. Here are strategies for effectively refactoring code.

### **22.10.1 When to Refactor**

1. **Rule of Three**: Refactor when you're about to duplicate code a third time.
2. **When Adding Features**: Refactor before adding new functionality.
3. **When Fixing Bugs**: Clean up the code while fixing bugs.
4. **When Code Review Reveals Issues**: Address feedback from code reviews.
5. **When Tests Are Difficult to Write**: Hard-to-test code is often poorly designed.

### **22.10.2 Refactoring Techniques**

1. **Extract Method**: Move code to a well-named function.
2. **Replace Magic Numbers with Named Constants**: Improve code clarity.
3. **Split Large Functions**: Break down complex functions into smaller ones.
4. **Introduce Interface**: Abstract behavior to increase flexibility.
5. **Replace Conditionals with Polymorphism**: Use interfaces and types instead of switch statements.

```go
// Before refactoring
func processUser(user User, action string) error {
    if action == "create" {
        // Create user logic
        return nil
    } else if action == "update" {
        // Update user logic
        return nil
    } else if action == "delete" {
        // Delete user logic
        return nil
    }
    return errors.New("unknown action")
}

// After refactoring
type UserProcessor interface {
    Process(user User) error
}

type UserCreator struct{}

func (c UserCreator) Process(user User) error {
    // Create user logic
    return nil
}

type UserUpdater struct{}

func (u UserUpdater) Process(user User) error {
    // Update user logic
    return nil
}

type UserDeleter struct{}

func (d UserDeleter) Process(user User) error {
    // Delete user logic
    return nil
}

func getProcessor(action string) (UserProcessor, error) {
    switch action {
    case "create":
        return UserCreator{}, nil
    case "update":
        return UserUpdater{}, nil
    case "delete":
        return UserDeleter{}, nil
    default:
        return nil, errors.New("unknown action")
    }
}

func processUser(user User, action string) error {
    processor, err := getProcessor(action)
    if err != nil {
        return err
    }
    return processor.Process(user)
}
```

### **22.10.3 Safe Refactoring**

1. **Maintain Test Coverage**: Never refactor without tests.
2. **Make Small, Incremental Changes**: Refactor in small steps, not all at once.
3. **Verify After Each Change**: Run tests after each refactoring step.
4. **Keep Functionality Unchanged**: Refactoring should not change behavior.
5. **Document Why, Not What**: Comment on the reasoning behind refactoring decisions.

## **22.11 Go-Specific Clean Code Practices**

### **22.11.1 Embracing Go Idioms**

1. **Prefer Composition Over Inheritance**: Use embedding rather than inheritance hierarchies.
2. **Return Early**: Handle errors and edge cases with early returns.
3. **Use Zero Values**: Leverage Go's zero values rather than nullable types.
4. **Keep Interfaces Small**: Define interfaces with just the methods you need.
5. **Errors Are Values**: Treat errors as regular values, not special cases.

```go
// Using composition
type Logger struct {
    // Logger fields
}

func (l Logger) Log(message string) {
    // Logging implementation
}

type UserService struct {
    Logger // Embedded, not inherited
    // UserService fields
}

// Now UserService has Log method via embedding
func DoSomething() {
    service := UserService{}
    service.Log("User service initialized")
}
```

### **22.11.2 Go-Specific Anti-Patterns to Avoid**

1. **Interface Pollution**: Don't create interfaces for everything.
2. **Empty Interface Abuse**: Avoid `interface{}` when possible.
3. **Panic/Recover as Exception Handling**: Don't use panic for normal error conditions.
4. **Excessive Concurrency**: Don't make everything concurrent without need.
5. **Neglecting Context**: Use context.Context for cancellation and timeouts.

```go
// Anti-pattern: Interface pollution
type UserGetter interface {
    GetUser(id string) (User, error)
}

type UserSaver interface {
    SaveUser(user User) error
}

type UserDeleter interface {
    DeleteUser(id string) error
}

// Clean approach: Unified interface
type UserRepository interface {
    Get(id string) (User, error)
    Save(user User) error
    Delete(id string) error
}
```

## **22.12 Exercises**

### **Exercise 1: Refactoring Functions**

Take the following code and refactor it to follow clean code principles:

```go
func process(s string, i int, b bool) (string, error) {
    var result string
    if s == "" {
        return "", errors.New("empty string")
    }
    if i <= 0 {
        return "", errors.New("invalid number")
    }
    if b == true {
        for j := 0; j < i; j++ {
            result += strings.ToUpper(s)
        }
    } else {
        for j := 0; j < i; j++ {
            result += strings.ToLower(s)
        }
    }
    return result, nil
}
```

### **Exercise 2: Applying SOLID Principles**

Redesign the following code to follow the SOLID principles:

```go
type User struct {
    ID       int
    Username string
    Email    string
}

func (u *User) Save() error {
    // Save user to database
    db, _ := sql.Open("postgres", "connection string")
    _, err := db.Exec("INSERT INTO users (username, email) VALUES ($1, $2)",
                     u.Username, u.Email)
    return err
}

func (u *User) Validate() error {
    if u.Username == "" {
        return errors.New("username cannot be empty")
    }
    if u.Email == "" {
        return errors.New("email cannot be empty")
    }
    if !strings.Contains(u.Email, "@") {
        return errors.New("invalid email format")
    }
    return nil
}

func (u *User) SendWelcomeEmail() error {
    // Send email logic
    client := &http.Client{}
    _, err := client.Post("https://email-service.com/send",
                         "application/json",
                         bytes.NewBuffer([]byte(`{"to":"` + u.Email + `"}`)))
    return err
}
```

### **Exercise 3: Creating Clean Tests**

Write clean, table-driven tests for the following function:

```go
func IsValidPassword(password string) error {
    if len(password) < 8 {
        return errors.New("password too short")
    }
    if !containsUppercase(password) {
        return errors.New("password must contain uppercase letter")
    }
    if !containsLowercase(password) {
        return errors.New("password must contain lowercase letter")
    }
    if !containsDigit(password) {
        return errors.New("password must contain a digit")
    }
    return nil
}

func containsUppercase(s string) bool {
    for _, r := range s {
        if unicode.IsUpper(r) {
            return true
        }
    }
    return false
}

func containsLowercase(s string) bool {
    for _, r := range s {
        if unicode.IsLower(r) {
            return true
        }
    }
    return false
}

func containsDigit(s string) bool {
    for _, r := range s {
        if unicode.IsDigit(r) {
            return true
        }
    }
    return false
}
```

### **Exercise 4: Improving Error Handling**

Refactor the following code to use proper error handling and wrapping:

```go
func ProcessOrder(orderID string) string {
    order, err := GetOrder(orderID)
    if err != nil {
        return "Error fetching order"
    }

    err = ValidateOrder(order)
    if err != nil {
        return "Invalid order"
    }

    err = ProcessPayment(order)
    if err != nil {
        return "Payment failed"
    }

    err = UpdateInventory(order)
    if err != nil {
        return "Inventory update failed"
    }

    err = SendConfirmation(order)
    if err != nil {
        return "Failed to send confirmation"
    }

    return "Order processed successfully"
}
```

### **Exercise 5: Package Organization**

Design a clean package structure for a blog application with the following features:

- User authentication
- Blog post creation and management
- Comments
- Tags and categories
- Search functionality
- Email notifications

## **22.13 Summary**

Clean code is not just about making code work; it's about making it maintainable, readable, and adaptable. In this chapter, we've explored principles and practices that contribute to clean code in Go:

- **Naming**: Using clear, descriptive names that reveal intent
- **Functions**: Writing small, focused functions that do one thing well
- **Error Handling**: Managing errors explicitly and providing context
- **Comments and Documentation**: Explaining why, not what, and keeping documentation up-to-date
- **Code Organization**: Structuring projects and packages for clarity and cohesion
- **SOLID Principles**: Applying sound design principles for more maintainable code
- **Testing**: Writing clean, focused tests that verify behavior

Clean code requires discipline and attention to detail, but the payoff is substantial: fewer bugs, easier maintenance, better collaboration, and ultimately, more productive development.

Remember the Boy Scout Rule: "Leave the code better than you found it." By consistently applying clean code principles, you can gradually improve even the most challenging codebases.

**Next Up**: In Chapter 23, we'll explore performance optimization in Go, building on our clean code foundation to create efficient, scalable applications.

---
