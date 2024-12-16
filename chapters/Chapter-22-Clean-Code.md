# **Chapter 22: Writing Clean Code in Go**

---

## **22.1. Introduction to Clean Code**

Clean code is more than just working code—it is code that is easy to read, understand, maintain, and extend. In this chapter, we’ll explore principles, practices, and techniques to write clean code in Go, following the guidelines set forth by Robert C. Martin, also known as Uncle Bob. By the end of this chapter, you will:

- Understand the core principles of clean code.
- Learn techniques to improve code readability and maintainability.
- Apply practical examples to real-world Go applications.
- Master the SOLID principles and other clean code patterns.

---

## **22.2. The Fifteen Rules for Clean Code**

Here are 15 essential rules to ensure clean and maintainable code, along with examples and explanations.

---

### **1. Use Meaningful Names**

**Explanation:**
Names should clearly indicate their purpose. Avoid cryptic abbreviations or overly long names. Good naming acts as self-documentation for your code.

**Example:**

```go
// Poor
func a() {}

// Better
func fetchUserData() {}
```

**Why This Matters:**

- The function name `fetchUserData` clearly conveys what it does, making the code easier to read and understand.
- Poorly named functions or variables force future developers to guess their purpose.

---

### **2. Keep Functions Small**

**Explanation:**
A function should do one thing and do it well. Keeping functions small improves readability and makes them easier to test.

**Example:**

```go
// Poor: Large function
func handleRequest() {
    validateInput()
    processData()
    sendResponse()
}

// Clean: Split responsibilities into smaller functions
func handleRequest() {
    if err := validateInput(); err != nil {
        handleError(err)
        return
    }
    data := processData()
    sendResponse(data)
}
```

**Why This Matters:**

- Small functions are easier to test and debug.
- By breaking large functions into smaller ones, you promote reuse and clarity.

---

### **3. Write Short and Focused Classes**

**Explanation:**
Classes (or structs in Go) should focus on a single responsibility. This adheres to the Single Responsibility Principle (SRP).

**Example:**

```go
// Poor: Struct with multiple responsibilities
type User struct {
    Name  string
    Email string
}

func (u User) Save() error {
    // Save user to database
}

func (u User) SendWelcomeEmail() error {
    // Send email
}

// Clean: Separate responsibilities
type User struct {
    Name  string
    Email string
}

type UserRepository struct{}

func (r UserRepository) Save(user User) error {
    // Save user to database
}

type EmailService struct{}

func (s EmailService) SendWelcomeEmail(user User) error {
    // Send email
}
```

**Why This Matters:**

- Focused classes are easier to maintain.
- Separating responsibilities prevents changes in one area from inadvertently affecting another.

---

### **4. DRY Principle (Don't Repeat Yourself)**

**Explanation:**
Avoid duplicating code. Repeated logic increases maintenance effort and the risk of errors.

**Example:**

```go
// Poor: Duplicate code for logging
func logUserCreation(name string) {
    fmt.Println("User created:", name)
}

func logAdminCreation(name string) {
    fmt.Println("Admin created:", name)
}

// Clean: Reusable function
func logCreation(entity, name string) {
    fmt.Printf("%s created: %s
", entity, name)
}

logCreation("User", "Alice")
logCreation("Admin", "Bob")
```

**Why This Matters:**

- Centralizing logic reduces duplication.
- Changes to shared logic need to be made in only one place.

---

### **5. Adhere to the SRP (Single Responsibility Principle)**

**Explanation:**
Each function, struct, or module should have one reason to change.

**Example:**

```go
// Poor: Function with multiple responsibilities
func handleUserRequest() {
    fetchUserData()
    processUserData()
    saveUserData()
}

// Clean: Separate responsibilities
func handleUserRequest() {
    data := fetchUserData()
    processed := processUserData(data)
    saveUserData(processed)
}
```

**Why This Matters:**

- SRP makes code more modular and easier to test.
- It reduces coupling and enhances maintainability.

---

### **6. Use Clear Error Handling**

**Explanation:**
Explicitly handle errors and provide meaningful messages.

**Example:**

```go
// Poor
func divide(a, b int) int {
    return a / b
}

// Clean
func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("cannot divide by zero")
    }
    return a / b, nil
}
```

**Why This Matters:**

- Clear error handling improves robustness.
- Meaningful error messages make debugging easier.

---

### **7. Use Interfaces Judiciously**

**Explanation:**
Interfaces should abstract behaviors, not implementations.

**Example:**

```go
// Poor: Tightly coupled implementation
type MySQLDatabase struct{}

func (db MySQLDatabase) Connect() {}

type UserRepository struct {
    db MySQLDatabase
}

// Clean: Abstracting behavior using an interface
type Database interface {
    Connect()
}

type MySQLDatabase struct{}

func (db MySQLDatabase) Connect() {}

type UserRepository struct {
    db Database
}
```

**Why This Matters:**

- Decoupling promotes flexibility.
- You can replace implementations without changing the dependent code.

---

### **8. Avoid Global State**

**Explanation:**
Global state makes code unpredictable and harder to debug. Instead, pass state explicitly.

**Example:**

```go
// Poor: Global state
var counter int

func increment() {
    counter++
}

// Clean: Pass state explicitly
func increment(counter *int) {
    *counter++
}
```

**Why This Matters:**

- Explicit state management improves testability.
- It reduces unintended side effects.

---

### **9. Follow the SOLID Principles**

**Explanation:**
Adhering to the SOLID principles improves code flexibility and scalability. These include:

- Single Responsibility Principle (SRP)
- Open/Closed Principle (OCP)
- Liskov Substitution Principle (LSP)
- Interface Segregation Principle (ISP)
- Dependency Inversion Principle (DIP)

---

### **10. Limit Function Parameters**

**Explanation:**
Reduce the number of function parameters; group related data in structs if needed.

**Example:**

```go
// Poor: Too many parameters
func createUser(name, email, password string, age int) {}

// Clean: Use a struct
type User struct {
    Name     string
    Email    string
    Password string
    Age      int
}

func createUser(user User) {}
```

**Why This Matters:**

- Fewer parameters improve readability.
- Grouping related data enhances clarity.

---

### **11. Use Constants Instead of Magic Numbers**

**Explanation:**
Avoid hardcoding numbers or strings. Use constants for better readability.

**Example:**

```go
// Poor: Magic number
if age < 18 {}

// Clean: Use constants
const LegalAge = 18

if age < LegalAge {}
```

**Why This Matters:**

- Constants make code self-explanatory.
- It’s easier to update values in one place.

---

### **12. Keep Code Simple and Readable**

**Explanation:**
Avoid over-engineering. Simplicity improves clarity and reduces bugs.

**Example:**

```go
// Poor: Over-engineered
func calculate(a, b int, op string) int {
    switch op {
    case "+":
        return a + b
    case "-":
        return a - b
    }
    return 0
}

// Clean: Use simple functions
func add(a, b int) int {
    return a + b
}

func subtract(a, b int) int {
    return a - b
}
```

---

### **13. Avoid Side Effects**

**Explanation:**
Functions should avoid altering global or external states.

**Example:**

```go
// Poor: Modifies external state
var total int

func add(a int) {
    total += a
}

// Clean: Return new value
func add(a, total int) int {
    return total + a
}
```

---

### **14. Ensure High Test Coverage**

**Explanation:**
Write unit and integration tests to ensure code reliability.

**Example:**

```go
func add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    if add(2, 3) != 5 {
        t.Errorf("Expected 5, got %d", add(2, 3))
    }
}
```

---

### **15. Use Meaningful Comments**

**Explanation:**
Write comments to explain "why" something is done, not "what" the code does.

**Example:**

```go
// Poor: States the obvious
// Increment the counter
counter++

// Clean: Explains intent
// Increment the counter to track processed items
counter++
```

---

## **22.3. Conclusion**

By following these 15 rules, you can significantly improve the quality of your codebase. Clean code is easier to read, maintain, and extend, making development more enjoyable and efficient.

---

## **22.4. Exercises: Clean Code in Practice**

Here are 15 exercises designed to help you practice writing clean code. Each exercise provides:

1. A **bad code** example that violates clean code principles.
2. A **clean code** example that fully adheres to clean coding practices.
3. Explanations to highlight why the changes improve the code.

---

### **Exercise 1: Meaningful Names**

**Bad Code:**

```go
func d(a int, b int) int {
    return a + b
}
```

**Clean Code:**

```go
func addNumbers(firstNumber int, secondNumber int) int {
    return firstNumber + secondNumber
}
```

**Explanation:**
Clear function and parameter names provide context and make the code easier to understand.

---

### **Exercise 2: Keep Functions Small**

**Bad Code:**

```go
func handleData(data string) {
    fmt.Println("Processing data")
    result := processData(data)
    fmt.Println("Processed:", result)
    saveData(result)
    fmt.Println("Data saved")
}
```

**Clean Code:**

```go
func handleData(data string) {
    result := processData(data)
    logProcessed(result)
    saveData(result)
    logSaved()
}

func logProcessed(result string) {
    fmt.Println("Processed:", result)
}

func logSaved() {
    fmt.Println("Data saved")
}
```

**Explanation:**
Breaking down responsibilities into smaller functions improves clarity and reusability.

---

### **Exercise 3: Avoid Magic Numbers**

**Bad Code:**

```go
if age >= 18 {
    fmt.Println("Allowed")
}
```

**Clean Code:**

```go
const MinLegalAge = 18

func canAccess(age int) bool {
    return age >= MinLegalAge
}

if canAccess(age) {
    fmt.Println("Allowed")
}
```

**Explanation:**
Constants provide context, and encapsulating logic in a function enhances clarity.

---

### **Exercise 4: DRY Principle**

**Bad Code:**

```go
fmt.Println("Error occurred: failed to load user")
fmt.Println("Error occurred: failed to save user")
```

**Clean Code:**

```go
type Logger struct{}

func (l Logger) LogError(message string) {
    fmt.Println("Error occurred:", message)
}

logger := Logger{}
logger.LogError("failed to load user")
logger.LogError("failed to save user")
```

**Explanation:**
Eliminating duplication with a reusable type improves maintainability.

---

### **Exercise 5: Single Responsibility Principle**

**Bad Code:**

```go
func saveUser(user User) {
    if user.Name == "" {
        fmt.Println("Invalid user")
        return
    }
    fmt.Println("User saved")
}
```

**Clean Code:**

```go
type UserValidator struct{}

func (v UserValidator) Validate(user User) bool {
    return user.Name != ""
}

type UserRepository struct{}

func (r UserRepository) Save(user User) {
    fmt.Println("User saved")
}

func saveUser(user User, validator UserValidator, repo UserRepository) {
    if !validator.Validate(user) {
        fmt.Println("Invalid user")
        return
    }
    repo.Save(user)
}
```

**Explanation:**
Separating validation and saving into distinct responsibilities promotes modularity.

---

### **Exercise 6: Clear Error Handling**

**Bad Code:**

```go
func divide(a, b int) int {
    return a / b
}
```

**Clean Code:**

```go
type Divider struct{}

func (d Divider) Divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

divider := Divider{}
result, err := divider.Divide(4, 2)
if err != nil {
    fmt.Println(err)
} else {
    fmt.Println(result)
}
```

**Explanation:**
Encapsulating division in a type with proper error handling improves robustness and clarity.

---

### **Exercise 7: Limit Function Parameters**

**Bad Code:**

```go
func registerUser(name string, email string, password string) {}
```

**Clean Code:**

```go
type User struct {
    Name     string
    Email    string
    Password string
}

type UserService struct{}

func (s UserService) Register(user User) {
    fmt.Println("User registered:", user.Name)
}

service := UserService{}
service.Register(User{Name: "Alice", Email: "alice@example.com", Password: "123456"})
```

**Explanation:**
Grouping related parameters into a struct reduces complexity and improves readability.

---

### **Exercise 8: Use Interfaces**

**Bad Code:**

```go
type MySQLRepository struct{}

func (r MySQLRepository) Save() {
    fmt.Println("Saving to MySQL")
}

repo := MySQLRepository{}
repo.Save()
```

**Clean Code:**

```go
type Repository interface {
    Save()
}

type MySQLRepository struct{}

func (r MySQLRepository) Save() {
    fmt.Println("Saving to MySQL")
}

type Service struct {
    Repo Repository
}

func (s Service) PerformSave() {
    s.Repo.Save()
}

service := Service{Repo: MySQLRepository{}}
service.PerformSave()
```

**Explanation:**
Using interfaces abstracts behavior, while dependency injection promotes flexibility.

---

### **Exercise 9: Simplify Conditionals**

**Bad Code:**

```go
if userRole == "admin" || userRole == "manager" {
    fmt.Println("Access granted")
}
```

**Clean Code:**

```go
type RoleChecker struct{}

func (r RoleChecker) HasAccess(role string) bool {
    allowedRoles := map[string]bool{"admin": true, "manager": true}
    return allowedRoles[role]
}

checker := RoleChecker{}
if checker.HasAccess(userRole) {
    fmt.Println("Access granted")
}
```

**Explanation:**
Using a map encapsulates logic and improves maintainability.

---

### **Exercise 10: Avoid Side Effects**

**Bad Code:**

```go
var total int

func add(a int) {
    total += a
}
```

**Clean Code:**

```go
func add(a, total int) int {
    return total + a
}
```

**Explanation:**
Returning values instead of modifying external state makes functions predictable and testable.

---

### **Exercise 11: Write Tests**

**Bad Code:**

```go
func add(a, b int) int {
    return a + b
}
```

**Clean Code:**

```go
func add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    cases := []struct {
        a, b, expected int
    }{
        {2, 3, 5},
        {1, 1, 2},
        {0, 0, 0},
    }

    for _, c := range cases {
        if add(c.a, c.b) != c.expected {
            t.Errorf("Expected %d, got %d", c.expected, add(c.a, c.b))
        }
    }
}
```

**Explanation:**
Using table-driven tests improves test coverage and clarity.

---

### **Exercise 12: Encapsulate Behavior**

**Bad Code:**

```go
func isAdult(age int) bool {
    return age >= 18
}

if isAdult(age) {
    fmt.Println("Adult")
}
```

**Clean Code:**

```go
type AgeChecker struct{}

func (a AgeChecker) IsAdult(age int) bool {
    return age >= 18
}

checker := AgeChecker{}
if checker.IsAdult(age) {
    fmt.Println("Adult")
}
```

**Explanation:**
Encapsulating behavior in a type promotes reusability and organization.

---

### **Exercise 13: Use Structs for Related Data**

**Bad Code:**

```go
func createRectangle(width, height int) {}
```

**Clean Code:**

```go
type Rectangle struct {
    Width  int
    Height int
}

func createRectangle(rect Rectangle) {}
```

**Explanation:**
Using structs for related data improves clarity and enforces relationships.

---

### **Exercise 14: Avoid Deep Nesting**

**Bad Code:**

```go
if condition1 {
    if condition2 {
        if condition3 {
            fmt.Println("Do something")
        }
    }
}
```

**Clean Code:**

```go
if !condition1 || !condition2 || !condition3 {
    return
}

fmt.Println("Do something")
```

**Explanation:**
Simplifying conditions and reducing nesting improves readability.

---

### **Exercise 15: Reuse Logic**

**Bad Code:**

```go
if price > 100 {
    fmt.Println("Expensive")
} else if price > 50 {
    fmt.Println("Moderate")
} else {
    fmt.Println("Cheap")
}
```

**Clean Code:**

```go
func categorizePrice(price int) string {
    switch {
    case price > 100:
        return "Expensive"
    case price > 50:
        return "Moderate"
    default:
        return "Cheap"
    }
}

fmt.Println(categorizePrice(price))
```

**Explanation:**
Encapsulating logic in a function makes it reusable and easier to test.

---

## **22.5. Conclusion**

These exercises demonstrate how to refactor bad code into clean code using concrete examples. Practice these principles to improve your ability to write clean and maintainable code.
