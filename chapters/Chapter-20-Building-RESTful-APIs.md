# **Chapter 20: Building RESTful APIs in Go**

Go has established itself as an excellent language for building web services and RESTful APIs due to its performance, concurrency model, and simplicity. This chapter provides a comprehensive guide to building robust, scalable, and maintainable RESTful APIs in Go, covering everything from fundamental concepts to advanced patterns and best practices.

## **20.1 Introduction to RESTful APIs**

### **20.1.1 What is REST?**

REST (Representational State Transfer) is an architectural style for designing networked applications. RESTful APIs use HTTP requests to perform CRUD (Create, Read, Update, Delete) operations on resources, which are represented as URLs.

Key principles of REST include:

1. **Statelessness**: Each request contains all information needed to complete it
2. **Client-Server Architecture**: Separation of concerns between client and server
3. **Cacheable**: Responses must define themselves as cacheable or non-cacheable
4. **Uniform Interface**: Resources are identified in requests, manipulated through representations, and include self-descriptive messages
5. **Layered System**: A client cannot ordinarily tell whether it is connected directly to the end server
6. **Code on Demand** (optional): Servers can extend client functionality by transferring executable code

### **20.1.2 HTTP Methods and REST**

The primary HTTP methods used in RESTful APIs:

| Method  | Purpose                                  | Idempotent | Safe |
| ------- | ---------------------------------------- | ---------- | ---- |
| GET     | Retrieve a resource                      | Yes        | Yes  |
| POST    | Create a resource                        | No         | No   |
| PUT     | Update a resource (complete replacement) | Yes        | No   |
| PATCH   | Partial update of a resource             | No         | No   |
| DELETE  | Remove a resource                        | Yes        | No   |
| HEAD    | Retrieve headers only                    | Yes        | Yes  |
| OPTIONS | Get supported operations on a resource   | Yes        | Yes  |

### **20.1.3 API Design Principles**

When designing RESTful APIs, follow these principles:

1. **Use nouns, not verbs for resources**: `/users`, not `/getUsers`
2. **Use plural for collection resources**: `/users`, not `/user`
3. **Use HTTP methods appropriately** for operations
4. **Use nested resources** for relationships: `/users/123/orders`
5. **Use appropriate HTTP status codes**
6. **Provide clear error messages**
7. **Implement pagination** for large collections
8. **Support filtering, sorting, and searching**
9. **Version your API**: `/v1/users`
10. **Follow consistent naming conventions**

## **20.2 Building APIs with Standard Library**

Go's standard library provides all the tools needed to build RESTful APIs without external dependencies. Let's start by building a simple API using the standard library.

### **20.2.1 Basic HTTP Server**

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
)

// User represents a user in our system
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

// Simple in-memory store for demonstration
var users = []User{
    {ID: 1, Username: "alice", Email: "alice@example.com"},
    {ID: 2, Username: "bob", Email: "bob@example.com"},
}

func main() {
    // Define routes
    http.HandleFunc("/users", handleUsers)
    http.HandleFunc("/users/", handleUser)

    // Start server
    log.Println("Server starting on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// handleUsers handles GET and POST requests for the /users endpoint
func handleUsers(w http.ResponseWriter, r *http.Request) {
    switch r.Method {
    case http.MethodGet:
        // Return all users
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(users)
    case http.MethodPost:
        // Create a new user
        var user User
        if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }

        // Set ID (in a real app, this would be generated)
        user.ID = len(users) + 1
        users = append(users, user)

        w.Header().Set("Content-Type", "application/json")
        w.WriteHeader(http.StatusCreated)
        json.NewEncoder(w).Encode(user)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}

// handleUser handles GET, PUT and DELETE requests for /users/{id}
func handleUser(w http.ResponseWriter, r *http.Request) {
    // Extract ID from URL
    // URL format: /users/{id}
    id := r.URL.Path[len("/users/"):]
    if id == "" {
        http.Error(w, "Invalid user ID", http.StatusBadRequest)
        return
    }

    // Find user by ID
    var foundUser User
    var found bool
    var index int

    // Simplified ID parsing for demo purposes
    idInt := 0
    _, err := fmt.Sscanf(id, "%d", &idInt)
    if err != nil {
        http.Error(w, "Invalid user ID format", http.StatusBadRequest)
        return
    }

    for i, user := range users {
        if user.ID == idInt {
            foundUser = user
            found = true
            index = i
            break
        }
    }

    if !found {
        http.Error(w, "User not found", http.StatusNotFound)
        return
    }

    switch r.Method {
    case http.MethodGet:
        // Return the user
        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(foundUser)
    case http.MethodPut:
        // Update the user
        var updatedUser User
        if err := json.NewDecoder(r.Body).Decode(&updatedUser); err != nil {
            http.Error(w, err.Error(), http.StatusBadRequest)
            return
        }

        // Preserve ID
        updatedUser.ID = foundUser.ID
        users[index] = updatedUser

        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(updatedUser)
    case http.MethodDelete:
        // Delete the user
        users = append(users[:index], users[index+1:]...)
        w.WriteHeader(http.StatusNoContent)
    default:
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
    }
}
```

This basic example demonstrates:

- Defining routes and handling different HTTP methods
- Working with JSON requests and responses
- Basic error handling
- In-memory data storage

### **20.2.2 Improved Router with ServeMux**

Go's standard `http.ServeMux` is simple but limited. Let's improve our routing:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
    "regexp"
    "strconv"
)

// User represents a user in our system
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

// Simple in-memory store
var users = []User{
    {ID: 1, Username: "alice", Email: "alice@example.com"},
    {ID: 2, Username: "bob", Email: "bob@example.com"},
}

// Route struct to hold our route patterns
type Route struct {
    pattern *regexp.Regexp
    method  string
    handler http.HandlerFunc
}

// Routes slice to hold all routes
var routes []Route

// AddRoute registers a new route
func AddRoute(pattern string, method string, handler http.HandlerFunc) {
    routes = append(routes, Route{
        pattern: regexp.MustCompile("^" + pattern + "$"),
        method:  method,
        handler: handler,
    })
}

// Router dispatches requests to the appropriate handler
func Router(w http.ResponseWriter, r *http.Request) {
    for _, route := range routes {
        matches := route.pattern.FindStringSubmatch(r.URL.Path)
        if matches != nil && (route.method == r.Method || route.method == "*") {
            route.handler(w, r)
            return
        }
    }
    http.NotFound(w, r)
}

func main() {
    // Register routes
    AddRoute("/users", http.MethodGet, getUsers)
    AddRoute("/users", http.MethodPost, createUser)
    AddRoute("/users/([0-9]+)", http.MethodGet, getUser)
    AddRoute("/users/([0-9]+)", http.MethodPut, updateUser)
    AddRoute("/users/([0-9]+)", http.MethodDelete, deleteUser)

    // Use our custom router
    http.HandleFunc("/", Router)

    // Start server
    log.Println("Server starting on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// Get all users
func getUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

// Create a new user
func createUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    // Set ID (in a real app, this would be generated)
    user.ID = len(users) + 1
    users = append(users, user)

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}

// Get a single user
func getUser(w http.ResponseWriter, r *http.Request) {
    matches := regexp.MustCompile("^/users/([0-9]+)$").FindStringSubmatch(r.URL.Path)
    id, _ := strconv.Atoi(matches[1])

    for _, user := range users {
        if user.ID == id {
            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(user)
            return
        }
    }

    http.Error(w, "User not found", http.StatusNotFound)
}

// Update a user
func updateUser(w http.ResponseWriter, r *http.Request) {
    matches := regexp.MustCompile("^/users/([0-9]+)$").FindStringSubmatch(r.URL.Path)
    id, _ := strconv.Atoi(matches[1])

    var updatedUser User
    if err := json.NewDecoder(r.Body).Decode(&updatedUser); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    for i, user := range users {
        if user.ID == id {
            // Preserve ID
            updatedUser.ID = id
            users[i] = updatedUser

            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(updatedUser)
            return
        }
    }

    http.Error(w, "User not found", http.StatusNotFound)
}

// Delete a user
func deleteUser(w http.ResponseWriter, r *http.Request) {
    matches := regexp.MustCompile("^/users/([0-9]+)$").FindStringSubmatch(r.URL.Path)
    id, _ := strconv.Atoi(matches[1])

    for i, user := range users {
        if user.ID == id {
            users = append(users[:i], users[i+1:]...)
            w.WriteHeader(http.StatusNoContent)
            return
        }
    }

    http.Error(w, "User not found", http.StatusNotFound)
}
```

This improved version includes:

- Regular expression-based routing
- Separate handler functions for each operation
- Better path parameter extraction

### **20.2.3 Middleware in Go**

Middleware functions intercept HTTP requests and responses to add common functionality:

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "time"
)

// Middleware type definition
type Middleware func(http.HandlerFunc) http.HandlerFunc

// Chain applies middlewares to a handler function
func Chain(handler http.HandlerFunc, middlewares ...Middleware) http.HandlerFunc {
    for _, middleware := range middlewares {
        handler = middleware(handler)
    }
    return handler
}

// Logging middleware
func Logging() Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            next(w, r)
            log.Printf(
                "%s %s %s",
                r.Method,
                r.RequestURI,
                time.Since(start),
            )
        }
    }
}

// Authentication middleware (simplified)
func Authentication() Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            token := r.Header.Get("Authorization")
            if token == "" {
                http.Error(w, "Unauthorized", http.StatusUnauthorized)
                return
            }
            // In a real app, validate the token here
            next(w, r)
        }
    }
}

// CORS middleware
func CORS() Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            w.Header().Set("Access-Control-Allow-Origin", "*")
            w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
            w.Header().Set("Access-Control-Allow-Headers", "Content-Type, Authorization")

            if r.Method == http.MethodOptions {
                w.WriteHeader(http.StatusOK)
                return
            }

            next(w, r)
        }
    }
}

func main() {
    // Apply middleware to handlers
    http.HandleFunc("/users", Chain(
        handleUsers,
        Logging(),
        CORS(),
    ))

    // Protected route with authentication
    http.HandleFunc("/admin", Chain(
        handleAdmin,
        Logging(),
        CORS(),
        Authentication(),
    ))

    // Start server
    log.Println("Server starting on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}

func handleUsers(w http.ResponseWriter, r *http.Request) {
    users := []map[string]interface{}{
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"},
    }
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

func handleAdmin(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]string{
        "message": "Admin area accessed successfully",
    })
}
```

This example demonstrates:

- Creating reusable middleware
- Chaining middleware together
- Implementing common middleware patterns (logging, authentication, CORS)

### **20.2.4 Dependency Injection**

To improve testability and maintainability, let's refactor our API to use dependency injection:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
)

// User represents a user in our system
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

// UserService handles user operations
type UserService interface {
    GetAll() ([]User, error)
    Get(id int) (User, error)
    Create(user User) (User, error)
    Update(id int, user User) (User, error)
    Delete(id int) error
}

// InMemoryUserService implements UserService with in-memory storage
type InMemoryUserService struct {
    users []User
    nextID int
}

// NewInMemoryUserService creates a new in-memory user service
func NewInMemoryUserService() *InMemoryUserService {
    return &InMemoryUserService{
        users: []User{
            {ID: 1, Username: "alice", Email: "alice@example.com"},
            {ID: 2, Username: "bob", Email: "bob@example.com"},
        },
        nextID: 3,
    }
}

func (s *InMemoryUserService) GetAll() ([]User, error) {
    return s.users, nil
}

func (s *InMemoryUserService) Get(id int) (User, error) {
    for _, user := range s.users {
        if user.ID == id {
            return user, nil
        }
    }
    return User{}, fmt.Errorf("user with ID %d not found", id)
}

func (s *InMemoryUserService) Create(user User) (User, error) {
    user.ID = s.nextID
    s.nextID++
    s.users = append(s.users, user)
    return user, nil
}

func (s *InMemoryUserService) Update(id int, user User) (User, error) {
    for i, u := range s.users {
        if u.ID == id {
            user.ID = id
            s.users[i] = user
            return user, nil
        }
    }
    return User{}, fmt.Errorf("user with ID %d not found", id)
}

func (s *InMemoryUserService) Delete(id int) error {
    for i, user := range s.users {
        if user.ID == id {
            s.users = append(s.users[:i], s.users[i+1:]...)
            return nil
        }
    }
    return fmt.Errorf("user with ID %d not found", id)
}

// UserHandler handles HTTP requests for users
type UserHandler struct {
    service UserService
}

// NewUserHandler creates a new user handler
func NewUserHandler(service UserService) *UserHandler {
    return &UserHandler{service: service}
}

func (h *UserHandler) GetUsers(w http.ResponseWriter, r *http.Request) {
    users, err := h.service.GetAll()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request, id int) {
    user, err := h.service.Get(id)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    created, err := h.service.Create(user)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(created)
}

func (h *UserHandler) UpdateUser(w http.ResponseWriter, r *http.Request, id int) {
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    updated, err := h.service.Update(id, user)
    if err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(updated)
}

func (h *UserHandler) DeleteUser(w http.ResponseWriter, r *http.Request, id int) {
    if err := h.service.Delete(id); err != nil {
        http.Error(w, err.Error(), http.StatusNotFound)
        return
    }

    w.WriteHeader(http.StatusNoContent)
}

func main() {
    // Create service and handler
    userService := NewInMemoryUserService()
    userHandler := NewUserHandler(userService)

    // Set up routes (simplified)
    http.HandleFunc("/users", func(w http.ResponseWriter, r *http.Request) {
        switch r.Method {
        case http.MethodGet:
            userHandler.GetUsers(w, r)
        case http.MethodPost:
            userHandler.CreateUser(w, r)
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })

    http.HandleFunc("/users/", func(w http.ResponseWriter, r *http.Request) {
        // Extract ID from URL (simplified)
        var id int
        _, err := fmt.Sscanf(r.URL.Path, "/users/%d", &id)
        if err != nil {
            http.Error(w, "Invalid user ID", http.StatusBadRequest)
            return
        }

        switch r.Method {
        case http.MethodGet:
            userHandler.GetUser(w, r, id)
        case http.MethodPut:
            userHandler.UpdateUser(w, r, id)
        case http.MethodDelete:
            userHandler.DeleteUser(w, r, id)
        default:
            http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        }
    })

    // Start server
    log.Println("Server starting on port 8080...")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

The benefits of this approach include:

- Separation of concerns (handlers, services)
- Improved testability via interfaces
- Easier maintenance and future extension

### **20.2.5 Testing RESTful APIs**

Testing is crucial for API development. Let's look at testing our handlers:

```go
package main

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"
)

// MockUserService implements UserService for testing
type MockUserService struct {
    users []User
}

func NewMockUserService() *MockUserService {
    return &MockUserService{
        users: []User{
            {ID: 1, Username: "testuser", Email: "test@example.com"},
        },
    }
}

func (s *MockUserService) GetAll() ([]User, error) {
    return s.users, nil
}

func (s *MockUserService) Get(id int) (User, error) {
    if id == 1 {
        return s.users[0], nil
    }
    return User{}, fmt.Errorf("user not found")
}

func (s *MockUserService) Create(user User) (User, error) {
    user.ID = 2
    return user, nil
}

func (s *MockUserService) Update(id int, user User) (User, error) {
    if id == 1 {
        user.ID = 1
        return user, nil
    }
    return User{}, fmt.Errorf("user not found")
}

func (s *MockUserService) Delete(id int) error {
    if id == 1 {
        return nil
    }
    return fmt.Errorf("user not found")
}

func TestGetUsers(t *testing.T) {
    // Setup
    mockService := NewMockUserService()
    handler := NewUserHandler(mockService)

    // Create request
    req, err := http.NewRequest(http.MethodGet, "/users", nil)
    if err != nil {
        t.Fatal(err)
    }

    // Create response recorder
    rr := httptest.NewRecorder()

    // Call the handler
    handler.GetUsers(rr, req)

    // Check status code
    if rr.Code != http.StatusOK {
        t.Errorf("expected status %d but got %d", http.StatusOK, rr.Code)
    }

    // Check response body
    var users []User
    if err := json.NewDecoder(rr.Body).Decode(&users); err != nil {
        t.Fatal(err)
    }

    if len(users) != 1 {
        t.Errorf("expected 1 user but got %d", len(users))
    }

    if users[0].Username != "testuser" {
        t.Errorf("expected username 'testuser' but got '%s'", users[0].Username)
    }
}

func TestCreateUser(t *testing.T) {
    // Setup
    mockService := NewMockUserService()
    handler := NewUserHandler(mockService)

    // Create request body
    newUser := User{Username: "newuser", Email: "new@example.com"}
    body, _ := json.Marshal(newUser)

    // Create request
    req, err := http.NewRequest(http.MethodPost, "/users", bytes.NewBuffer(body))
    if err != nil {
        t.Fatal(err)
    }
    req.Header.Set("Content-Type", "application/json")

    // Create response recorder
    rr := httptest.NewRecorder()

    // Call the handler
    handler.CreateUser(rr, req)

    // Check status code
    if rr.Code != http.StatusCreated {
        t.Errorf("expected status %d but got %d", http.StatusCreated, rr.Code)
    }

    // Check response body
    var createdUser User
    if err := json.NewDecoder(rr.Body).Decode(&createdUser); err != nil {
        t.Fatal(err)
    }

    if createdUser.ID != 2 {
        t.Errorf("expected ID 2 but got %d", createdUser.ID)
    }

    if createdUser.Username != "newuser" {
        t.Errorf("expected username 'newuser' but got '%s'", createdUser.Username)
    }
}
```

### **20.2.6 Error Handling**

Consistent error handling is essential for a robust API:

```go
package main

import (
    "encoding/json"
    "errors"
    "log"
    "net/http"
)

// APIError represents an API error
type APIError struct {
    Status  int    `json:"status"`
    Message string `json:"message"`
    Error   string `json:"error,omitempty"`
}

// ErrorResponse creates a standardized error response
func ErrorResponse(w http.ResponseWriter, status int, message string, err error) {
    apiError := APIError{
        Status:  status,
        Message: message,
    }

    if err != nil {
        apiError.Error = err.Error()
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(apiError)
}

// Custom error types
var (
    ErrNotFound     = errors.New("resource not found")
    ErrInvalidInput = errors.New("invalid input")
    ErrUnauthorized = errors.New("unauthorized")
    ErrForbidden    = errors.New("forbidden")
    ErrInternal     = errors.New("internal server error")
)

// UserHandler with improved error handling
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request, id int) {
    user, err := h.service.Get(id)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            ErrorResponse(w, http.StatusNotFound, "User not found", err)
        } else {
            log.Printf("Error retrieving user: %v", err)
            ErrorResponse(w, http.StatusInternalServerError, "Failed to retrieve user", nil)
        }
        return
    }

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(user)
}

// Middleware for error recovery
func Recovery(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                log.Printf("Panic: %v", err)
                ErrorResponse(w, http.StatusInternalServerError, "An unexpected error occurred", nil)
            }
        }()
        next.ServeHTTP(w, r)
    })
}
```

Key aspects of good error handling:

- Consistent error response format
- Appropriate HTTP status codes
- Logging errors but not exposing internal details to clients
- Recovery from panics
- Custom error types for different scenarios

## **20.3 Building APIs with Web Frameworks**

While Go's standard library provides excellent tools for building APIs, web frameworks can simplify development by offering additional features and conveniences. We'll explore two popular frameworks: Fiber and Echo.

### **20.3.1 Introduction to Fiber**

Fiber is a web framework inspired by Express.js, built on top of Fasthttp, which claims to be the fastest HTTP engine for Go.

**Key features of Fiber:**

- Express-like API (familiar for JavaScript developers)
- Built-in middleware
- Routing with route groups
- Static file serving
- Template engines
- WebSocket support
- Low memory footprint
- High performance

Let's reimplement our API using Fiber:

```go
package main

import (
    "log"
    "strconv"

    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/cors"
    "github.com/gofiber/fiber/v2/middleware/logger"
    "github.com/gofiber/fiber/v2/middleware/recover"
)

// User represents a user in our system
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

// UserService interface (same as before)
type UserService interface {
    GetAll() ([]User, error)
    Get(id int) (User, error)
    Create(user User) (User, error)
    Update(id int, user User) (User, error)
    Delete(id int) error
}

// InMemoryUserService implementation (same as before)
// ...

// UserHandler handles HTTP requests for users using Fiber
type UserHandler struct {
    service UserService
}

// NewUserHandler creates a new user handler
func NewUserHandler(service UserService) *UserHandler {
    return &UserHandler{service: service}
}

// RegisterRoutes registers routes for the user handler
func (h *UserHandler) RegisterRoutes(app *fiber.App) {
    users := app.Group("/users")

    users.Get("/", h.GetUsers)
    users.Post("/", h.CreateUser)
    users.Get("/:id", h.GetUser)
    users.Put("/:id", h.UpdateUser)
    users.Delete("/:id", h.DeleteUser)
}

// GetUsers returns all users
func (h *UserHandler) GetUsers(c *fiber.Ctx) error {
    users, err := h.service.GetAll()
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{
            "error": "Failed to retrieve users",
        })
    }

    return c.JSON(users)
}

// GetUser returns a single user
func (h *UserHandler) GetUser(c *fiber.Ctx) error {
    id, err := strconv.Atoi(c.Params("id"))
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Invalid user ID",
        })
    }

    user, err := h.service.Get(id)
    if err != nil {
        return c.Status(fiber.StatusNotFound).JSON(fiber.Map{
            "error": "User not found",
        })
    }

    return c.JSON(user)
}

// CreateUser creates a new user
func (h *UserHandler) CreateUser(c *fiber.Ctx) error {
    var user User
    if err := c.BodyParser(&user); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Invalid request body",
        })
    }

    created, err := h.service.Create(user)
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{
            "error": "Failed to create user",
        })
    }

    return c.Status(fiber.StatusCreated).JSON(created)
}

// UpdateUser updates an existing user
func (h *UserHandler) UpdateUser(c *fiber.Ctx) error {
    id, err := strconv.Atoi(c.Params("id"))
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Invalid user ID",
        })
    }

    var user User
    if err := c.BodyParser(&user); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Invalid request body",
        })
    }

    updated, err := h.service.Update(id, user)
    if err != nil {
        return c.Status(fiber.StatusNotFound).JSON(fiber.Map{
            "error": "User not found",
        })
    }

    return c.JSON(updated)
}

// DeleteUser deletes a user
func (h *UserHandler) DeleteUser(c *fiber.Ctx) error {
    id, err := strconv.Atoi(c.Params("id"))
    if err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Invalid user ID",
        })
    }

    if err := h.service.Delete(id); err != nil {
        return c.Status(fiber.StatusNotFound).JSON(fiber.Map{
            "error": "User not found",
        })
    }

    return c.SendStatus(fiber.StatusNoContent)
}

func main() {
    // Create service
    userService := NewInMemoryUserService()
    userHandler := NewUserHandler(userService)

    // Create Fiber app
    app := fiber.New(fiber.Config{
        ErrorHandler: func(c *fiber.Ctx, err error) error {
            // Default error handler
            code := fiber.StatusInternalServerError

            if e, ok := err.(*fiber.Error); ok {
                // Override status code if it's a Fiber error
                code = e.Code
            }

            return c.Status(code).JSON(fiber.Map{
                "error": err.Error(),
            })
        },
    })

    // Add middlewares
    app.Use(logger.New())
    app.Use(recover.New())
    app.Use(cors.New())

    // Register routes
    userHandler.RegisterRoutes(app)

    // Start server
    log.Fatal(app.Listen(":8080"))
}
```

### **20.3.2 Introduction to Echo**

Echo is another popular Go web framework focused on simplicity and performance.

**Key features of Echo:**

- Optimized router with zero dynamic memory allocation
- Middleware support
- Data binding and validation
- Scalable architecture
- Extensible with third-party packages
- Context-based API
- Data rendering

Let's implement our API using Echo:

```go
package main

import (
    "net/http"
    "strconv"

    "github.com/labstack/echo/v4"
    "github.com/labstack/echo/v4/middleware"
)

// User model (same as before)
// UserService interface (same as before)
// InMemoryUserService implementation (same as before)

// UserHandler handles HTTP requests for users using Echo
type UserHandler struct {
    service UserService
}

// NewUserHandler creates a new user handler
func NewUserHandler(service UserService) *UserHandler {
    return &UserHandler{service: service}
}

// RegisterRoutes registers routes for the user handler
func (h *UserHandler) RegisterRoutes(e *echo.Echo) {
    e.GET("/users", h.GetUsers)
    e.POST("/users", h.CreateUser)
    e.GET("/users/:id", h.GetUser)
    e.PUT("/users/:id", h.UpdateUser)
    e.DELETE("/users/:id", h.DeleteUser)
}

// GetUsers returns all users
func (h *UserHandler) GetUsers(c echo.Context) error {
    users, err := h.service.GetAll()
    if err != nil {
        return c.JSON(http.StatusInternalServerError, map[string]string{
            "error": "Failed to retrieve users",
        })
    }

    return c.JSON(http.StatusOK, users)
}

// GetUser returns a single user
func (h *UserHandler) GetUser(c echo.Context) error {
    id, err := strconv.Atoi(c.Param("id"))
    if err != nil {
        return c.JSON(http.StatusBadRequest, map[string]string{
            "error": "Invalid user ID",
        })
    }

    user, err := h.service.Get(id)
    if err != nil {
        return c.JSON(http.StatusNotFound, map[string]string{
            "error": "User not found",
        })
    }

    return c.JSON(http.StatusOK, user)
}

// CreateUser creates a new user
func (h *UserHandler) CreateUser(c echo.Context) error {
    var user User
    if err := c.Bind(&user); err != nil {
        return c.JSON(http.StatusBadRequest, map[string]string{
            "error": "Invalid request body",
        })
    }

    created, err := h.service.Create(user)
    if err != nil {
        return c.JSON(http.StatusInternalServerError, map[string]string{
            "error": "Failed to create user",
        })
    }

    return c.JSON(http.StatusCreated, created)
}

// UpdateUser updates an existing user
func (h *UserHandler) UpdateUser(c echo.Context) error {
    id, err := strconv.Atoi(c.Param("id"))
    if err != nil {
        return c.JSON(http.StatusBadRequest, map[string]string{
            "error": "Invalid user ID",
        })
    }

    var user User
    if err := c.Bind(&user); err != nil {
        return c.JSON(http.StatusBadRequest, map[string]string{
            "error": "Invalid request body",
        })
    }

    updated, err := h.service.Update(id, user)
    if err != nil {
        return c.JSON(http.StatusNotFound, map[string]string{
            "error": "User not found",
        })
    }

    return c.JSON(http.StatusOK, updated)
}

// DeleteUser deletes a user
func (h *UserHandler) DeleteUser(c echo.Context) error {
    id, err := strconv.Atoi(c.Param("id"))
    if err != nil {
        return c.JSON(http.StatusBadRequest, map[string]string{
            "error": "Invalid user ID",
        })
    }

    if err := h.service.Delete(id); err != nil {
        return c.JSON(http.StatusNotFound, map[string]string{
            "error": "User not found",
        })
    }

    return c.NoContent(http.StatusNoContent)
}

func main() {
    // Create service
    userService := NewInMemoryUserService()
    userHandler := NewUserHandler(userService)

    // Create Echo instance
    e := echo.New()

    // Middleware
    e.Use(middleware.Logger())
    e.Use(middleware.Recover())
    e.Use(middleware.CORS())

    // Register routes
    userHandler.RegisterRoutes(e)

    // Start server
    e.Logger.Fatal(e.Start(":8080"))
}
```

### **20.3.3 Framework Comparison: Fiber vs. Echo**

Both Fiber and Echo are excellent choices for building RESTful APIs in Go. Here's a comparison to help you choose:

| Feature                 | Fiber                              | Echo                          |
| ----------------------- | ---------------------------------- | ----------------------------- |
| Performance             | Built on Fasthttp, very fast       | Standard net/http, still fast |
| API Style               | Express-like, familiar for JS devs | Context-based, similar to Gin |
| Memory Usage            | Lower                              | Slightly higher               |
| Maturity                | Newer, growing community           | Well-established, stable      |
| Middleware              | Rich ecosystem                     | Rich ecosystem                |
| Learning Curve          | Easy, especially for JS devs       | Easy                          |
| Third-party Integration | Good                               | Excellent                     |
| Documentation           | Good                               | Excellent                     |

**Recommendation:**

- Choose **Fiber** if you're coming from a Node.js/Express background, need maximum performance, or prefer an Express-like API.
- Choose **Echo** if you prefer a more Go-idiomatic approach, need better third-party integration, or want a more established framework.

For this chapter, we'll focus on Fiber for the remaining examples due to its performance advantages and growing popularity.

### **20.3.4 Request Validation with Fiber**

Proper request validation is crucial for API security and reliability. Let's add validation to our Fiber implementation:

```go
package main

import (
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v2"
)

// User with validation tags
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username" validate:"required,min=3,max=50"`
    Email    string `json:"email" validate:"required,email"`
    Age      int    `json:"age" validate:"gte=18,lte=120"`
}

// Validator instance
var validate = validator.New()

// Validation middleware
func ValidateUser(c *fiber.Ctx) error {
    var user User

    // Parse request body
    if err := c.BodyParser(&user); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Cannot parse JSON",
        })
    }

    // Validate user
    if err := validate.Struct(user); err != nil {
        errors := err.(validator.ValidationErrors)
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Validation failed",
            "details": formatValidationErrors(errors),
        })
    }

    // Set validated user to locals
    c.Locals("user", user)

    return c.Next()
}

// Format validation errors to be user-friendly
func formatValidationErrors(errors validator.ValidationErrors) map[string]string {
    errorMap := make(map[string]string)

    for _, err := range errors {
        field := err.Field()
        switch err.Tag() {
        case "required":
            errorMap[field] = field + " is required"
        case "email":
            errorMap[field] = field + " must be a valid email"
        case "min":
            errorMap[field] = field + " must be at least " + err.Param() + " characters"
        case "max":
            errorMap[field] = field + " must be at most " + err.Param() + " characters"
        case "gte":
            errorMap[field] = field + " must be greater than or equal to " + err.Param()
        case "lte":
            errorMap[field] = field + " must be less than or equal to " + err.Param()
        default:
            errorMap[field] = field + " is invalid"
        }
    }

    return errorMap
}

// Usage in route definition
users.Post("/", ValidateUser, h.CreateUser)
```

### **20.3.5 Authentication and Authorization**

Let's implement JWT-based authentication with Fiber:

```go
package main

import (
    "time"

    "github.com/gofiber/fiber/v2"
    jwtware "github.com/gofiber/jwt/v3"
    "github.com/golang-jwt/jwt/v4"
)

// JWT secret key
var jwtSecret = []byte("your-secret-key")

// Credentials for login
type Credentials struct {
    Username string `json:"username" validate:"required"`
    Password string `json:"password" validate:"required"`
}

// Claims represents JWT claims
type Claims struct {
    Username string `json:"username"`
    Admin    bool   `json:"admin"`
    jwt.RegisteredClaims
}

// AuthHandler handles authentication
type AuthHandler struct {
    // In a real app, use a user service to verify credentials
}

// NewAuthHandler creates a new auth handler
func NewAuthHandler() *AuthHandler {
    return &AuthHandler{}
}

// RegisterRoutes registers routes for the auth handler
func (h *AuthHandler) RegisterRoutes(app *fiber.App) {
    auth := app.Group("/auth")

    auth.Post("/login", h.Login)

    // Protected routes
    protected := app.Group("/protected")
    protected.Use(jwtware.New(jwtware.Config{
        SigningKey: jwtSecret,
        ErrorHandler: func(c *fiber.Ctx, err error) error {
            return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
                "error": "Unauthorized",
            })
        },
    }))

    protected.Get("/", h.Protected)

    // Admin-only routes
    admin := app.Group("/admin")
    admin.Use(jwtware.New(jwtware.Config{
        SigningKey: jwtSecret,
    }))
    admin.Use(h.AdminOnly)

    admin.Get("/", h.AdminProtected)
}

// Login handles user login
func (h *AuthHandler) Login(c *fiber.Ctx) error {
    var credentials Credentials

    if err := c.BodyParser(&credentials); err != nil {
        return c.Status(fiber.StatusBadRequest).JSON(fiber.Map{
            "error": "Cannot parse JSON",
        })
    }

    // In a real app, verify credentials against database
    // This is a simplified example
    if credentials.Username != "admin" || credentials.Password != "password" {
        return c.Status(fiber.StatusUnauthorized).JSON(fiber.Map{
            "error": "Invalid credentials",
        })
    }

    // Determine if user is an admin (in a real app, check from database)
    isAdmin := credentials.Username == "admin"

    // Create token
    claims := Claims{
        Username: credentials.Username,
        Admin:    isAdmin,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(time.Hour * 24)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            NotBefore: jwt.NewNumericDate(time.Now()),
            Issuer:    "api.example.com",
            Subject:   credentials.Username,
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    signedToken, err := token.SignedString(jwtSecret)
    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{
            "error": "Could not generate token",
        })
    }

    return c.JSON(fiber.Map{
        "token": signedToken,
    })
}

// Protected route handler
func (h *AuthHandler) Protected(c *fiber.Ctx) error {
    user := c.Locals("user").(*jwt.Token)
    claims := user.Claims.(jwt.MapClaims)
    username := claims["username"].(string)

    return c.JSON(fiber.Map{
        "message": "Welcome " + username + "!",
    })
}

// AdminOnly middleware
func (h *AuthHandler) AdminOnly(c *fiber.Ctx) error {
    user := c.Locals("user").(*jwt.Token)
    claims := user.Claims.(jwt.MapClaims)

    if claims["admin"] != true {
        return c.Status(fiber.StatusForbidden).JSON(fiber.Map{
            "error": "Admin access required",
        })
    }

    return c.Next()
}

// AdminProtected route handler
func (h *AuthHandler) AdminProtected(c *fiber.Ctx) error {
    return c.JSON(fiber.Map{
        "message": "Welcome to the admin area!",
    })
}

// Usage in main function
func main() {
    app := fiber.New()

    // Register auth routes
    authHandler := NewAuthHandler()
    authHandler.RegisterRoutes(app)

    // ... other routes

    app.Listen(":8080")
}
```

## **20.4 OpenAPI and Swagger Integration**

OpenAPI (formerly known as Swagger) is a specification for documenting RESTful APIs. Integrating OpenAPI with your Go API provides several benefits:

- Interactive API documentation
- Client SDK generation
- Server stub generation
- API validation
- API testing

### **20.4.1 OpenAPI Specification Basics**

The OpenAPI Specification (OAS) defines a standard, language-agnostic interface for RESTful APIs. Here's a simple example of an OpenAPI 3.0 document:

```yaml
openapi: 3.0.0
info:
  title: User API
  description: API for managing users
  version: 1.0.0
servers:
  - url: http://localhost:8080
    description: Development server
paths:
  /users:
    get:
      summary: Get all users
      description: Returns a list of all users
      responses:
        "200":
          description: A list of users
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: "#/components/schemas/User"
    post:
      summary: Create a new user
      description: Creates a new user with the provided information
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/NewUser"
      responses:
        "201":
          description: The created user
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "400":
          description: Bad request
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /users/{id}:
    get:
      summary: Get a user by ID
      description: Returns a single user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        "200":
          description: A user
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/User"
        "404":
          description: User not found
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
    # ... PUT and DELETE operations ...
components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        username:
          type: string
        email:
          type: string
      required:
        - id
        - username
        - email
    NewUser:
      type: object
      properties:
        username:
          type: string
          minLength: 3
          maxLength: 50
        email:
          type: string
          format: email
      required:
        - username
        - email
    Error:
      type: object
      properties:
        error:
          type: string
```

### **20.4.2 Generating OpenAPI Documentation with Swaggo**

Swaggo is a tool that converts Go annotations to OpenAPI documentation. Let's integrate it with our Fiber API:

First, install Swaggo:

```bash
go install github.com/swaggo/swag/cmd/swag@latest
```

Next, add annotations to your handlers:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/swagger"
    _ "github.com/yourproject/docs" // Import generated docs
)

// @title User API
// @version 1.0
// @description API for managing users
// @BasePath /
func main() {
    // ... setup code ...
}

// UserHandler handles HTTP requests for users
type UserHandler struct {
    service UserService
}

// GetUsers returns all users
// @Summary Get all users
// @Description Returns a list of all users
// @Tags users
// @Accept json
// @Produce json
// @Success 200 {array} User
// @Failure 500 {object} ErrorResponse
// @Router /users [get]
func (h *UserHandler) GetUsers(c *fiber.Ctx) error {
    // ... implementation ...
}

// GetUser returns a single user
// @Summary Get a user by ID
// @Description Returns a single user by ID
// @Tags users
// @Accept json
// @Produce json
// @Param id path int true "User ID"
// @Success 200 {object} User
// @Failure 400 {object} ErrorResponse
// @Failure 404 {object} ErrorResponse
// @Router /users/{id} [get]
func (h *UserHandler) GetUser(c *fiber.Ctx) error {
    // ... implementation ...
}

// CreateUser creates a new user
// @Summary Create a new user
// @Description Creates a new user with the provided information
// @Tags users
// @Accept json
// @Produce json
// @Param user body NewUser true "User information"
// @Success 201 {object} User
// @Failure 400 {object} ErrorResponse
// @Failure 500 {object} ErrorResponse
// @Router /users [post]
func (h *UserHandler) CreateUser(c *fiber.Ctx) error {
    // ... implementation ...
}
```

Generate the documentation:

```bash
swag init
```

Add the Swagger UI to your Fiber app:

```go
// Add Swagger documentation
app.Get("/swagger/*", swagger.HandlerDefault)
```

Now you can access the Swagger UI at `http://localhost:8080/swagger/index.html`.

### **20.4.3 OpenAPI Client Generation**

Once you have an OpenAPI specification, you can generate client libraries in various languages using tools like OpenAPI Generator:

```bash
# Install OpenAPI Generator
npm install @openapitools/openapi-generator-cli -g

# Generate TypeScript client
openapi-generator-cli generate -i swagger.json -g typescript-fetch -o ./client
```

Generated clients make it easier for frontend developers or API consumers to integrate with your API.

### **20.4.4 API Testing with OpenAPI**

You can use the OpenAPI specification to automate API testing with tools like Postman or Dredd:

```bash
# Install Dredd
npm install -g dredd

# Run tests
dredd openapi.yaml http://localhost:8080
```

This validates that your API implementation matches the specification.

## **20.5 API Design Best Practices**

### **20.5.1 URL Design**

Follow these guidelines for designing clean and intuitive URL structures:

- Use plural nouns for collections: `/users` instead of `/user`
- Use hierarchical relationships for nested resources: `/users/123/orders`
- Keep URLs simple and descriptive
- Use kebab-case for multi-word resources: `/shipping-addresses`
- Use query parameters for filtering, sorting, and pagination: `/users?role=admin&sort=name`
- Version your API: `/v1/users`

**Examples:**

```
# Good
GET /v1/users
GET /v1/users/123
GET /v1/users/123/orders
GET /v1/users?role=admin

# Bad
GET /getUsers
GET /user/123
GET /userOrders/123
GET /v1/getAdminUsers
```

### **20.5.2 HTTP Status Codes**

Use appropriate HTTP status codes to communicate the result of API requests:

| Code Range | Category     | Common Codes                                       |
| ---------- | ------------ | -------------------------------------------------- |
| 2xx        | Success      | 200 OK, 201 Created, 204 No Content                |
| 3xx        | Redirection  | 301 Moved Permanently, 304 Not Modified            |
| 4xx        | Client Error | 400 Bad Request, 401 Unauthorized, 404 Not Found   |
| 5xx        | Server Error | 500 Internal Server Error, 503 Service Unavailable |

**Guidelines:**

- Use 200 for successful GET, PUT, and PATCH
- Use 201 for successful POST that creates a resource
- Use 204 for successful DELETE or operations with no response body
- Use 400 for invalid request syntax
- Use 401 for authentication failures
- Use 403 for authorization failures
- Use 404 when a resource doesn't exist
- Use 500 for unexpected server errors

### **20.5.3 Response Structure**

Maintain consistent response structures:

```json
// Success response
{
  "data": {
    "id": 123,
    "username": "alice",
    "email": "alice@example.com"
  },
  "meta": {
    "timestamp": "2023-04-01T12:00:00Z"
  }
}

// List response with pagination
{
  "data": [
    { "id": 1, "username": "alice" },
    { "id": 2, "username": "bob" }
  ],
  "meta": {
    "page": 1,
    "per_page": 10,
    "total": 42
  },
  "links": {
    "self": "/users?page=1",
    "next": "/users?page=2",
    "prev": null
  }
}

// Error response
{
  "error": {
    "code": "invalid_input",
    "message": "The request was invalid",
    "details": {
      "username": "Username is required"
    }
  }
}
```

### **20.5.4 API Security**

Implement these security measures for your API:

1. **Use HTTPS**: Always serve your API over HTTPS to encrypt data in transit
2. **Authentication**: Implement JWT, OAuth, or API keys
3. **Authorization**: Restrict access based on user roles and permissions
4. **Rate Limiting**: Protect against abuse and DoS attacks
5. **Input Validation**: Validate all input to prevent injection attacks
6. **CORS Configuration**: Control which domains can access your API
7. **Security Headers**: Set appropriate headers like `Content-Security-Policy`
8. **Error Handling**: Don't leak sensitive information in error messages

Example rate limiting with Fiber:

```go
import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/limiter"
    "time"
)

app.Use(limiter.New(limiter.Config{
    Max:        100,
    Expiration: 1 * time.Minute,
    KeyGenerator: func(c *fiber.Ctx) string {
        return c.IP() // Rate limit by IP address
    },
    LimitReached: func(c *fiber.Ctx) error {
        return c.Status(fiber.StatusTooManyRequests).JSON(fiber.Map{
            "error": "Rate limit exceeded",
        })
    },
}))
```

### **20.5.5 API Versioning**

There are several approaches to API versioning:

1. **URL Path Versioning**: `/v1/users`, `/v2/users`

   - Pros: Simple, explicit, easy to browse
   - Cons: Not RESTful (resource shouldn't change based on version)

2. **Query Parameter Versioning**: `/users?version=1`

   - Pros: Maintains clean URLs
   - Cons: Optional parameters might be ignored

3. **Header Versioning**: `Accept: application/vnd.myapi.v1+json`

   - Pros: RESTful, doesn't pollute URL
   - Cons: Less visible, harder to test

4. **Content Negotiation**: `Accept: application/vnd.myapi+json;version=1.0`
   - Pros: Most RESTful approach
   - Cons: Complex, less common

URL path versioning is the most common approach due to its simplicity and visibility.

### **20.5.6 Pagination, Filtering, and Sorting**

Implement these features for collections:

**Pagination:**

```
GET /users?page=2&per_page=10
```

**Filtering:**

```
GET /users?role=admin&status=active
```

**Sorting:**

```
GET /users?sort=name&order=asc
```

**Searching:**

```
GET /users?q=alice
```

Example implementation with Fiber:

```go
func (h *UserHandler) GetUsers(c *fiber.Ctx) error {
    // Pagination
    page, _ := strconv.Atoi(c.Query("page", "1"))
    perPage, _ := strconv.Atoi(c.Query("per_page", "10"))

    // Filtering
    role := c.Query("role")
    status := c.Query("status")

    // Sorting
    sort := c.Query("sort", "id")
    order := c.Query("order", "asc")

    // Search
    query := c.Query("q")

    // Get users with parameters
    users, total, err := h.service.GetUsers(UserFilter{
        Page:    page,
        PerPage: perPage,
        Role:    role,
        Status:  status,
        Sort:    sort,
        Order:   order,
        Query:   query,
    })

    if err != nil {
        return c.Status(fiber.StatusInternalServerError).JSON(fiber.Map{
            "error": "Failed to retrieve users",
        })
    }

    // Return paginated response
    return c.JSON(fiber.Map{
        "data": users,
        "meta": fiber.Map{
            "page":     page,
            "per_page": perPage,
            "total":    total,
        },
        "links": fiber.Map{
            "self": fmt.Sprintf("/users?page=%d", page),
            "next": page*perPage < total ? fmt.Sprintf("/users?page=%d", page+1) : nil,
            "prev": page > 1 ? fmt.Sprintf("/users?page=%d", page-1) : nil,
        },
    })
}
```

### **20.5.7 Caching**

Implement caching to improve performance:

```go
import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/cache"
    "time"
)

app.Use(cache.New(cache.Config{
    Next: func(c *fiber.Ctx) bool {
        return c.Query("refresh") == "true" // Skip cache if refresh is requested
    },
    Expiration: 30 * time.Minute,
    CacheControl: true,
}))
```

Also use HTTP caching headers:

```go
func (h *UserHandler) GetUser(c *fiber.Ctx) error {
    // ... get user ...

    // Set cache headers
    c.Set("Cache-Control", "public, max-age=300")
    c.Set("ETag", calculateETag(user))

    return c.JSON(user)
}
```

## **20.6 Performance Optimization**

### **20.6.1 Database Optimization**

- Use connection pooling
- Create appropriate indexes
- Use query optimization
- Implement database caching
- Consider read replicas for scaling

Example with connection pooling:

```go
import (
    "database/sql"
    _ "github.com/lib/pq"
)

func initDB() *sql.DB {
    db, err := sql.Open("postgres", "postgres://user:password@localhost/dbname")
    if err != nil {
        log.Fatal(err)
    }

    // Configure connection pool
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(25)
    db.SetConnMaxLifetime(5 * time.Minute)

    return db
}
```

### **20.6.2 API Response Optimization**

- Use compression
- Implement response field filtering
- Consider GraphQL for complex data requirements
- Use streaming for large responses

Example with Fiber compression:

```go
import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/compress"
)

app.Use(compress.New(compress.Config{
    Level: compress.LevelBestSpeed,
}))
```

### **20.6.3 Monitoring and Profiling**

Monitor your API performance:

```go
import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/monitor"
    "github.com/gofiber/fiber/v2/middleware/pprof"
)

// Add metrics endpoint
app.Get("/metrics", monitor.New())

// Add pprof endpoints for profiling
app.Use(pprof.New())
```

## **20.7 API Testing**

### **20.7.1 Unit Testing**

Test individual components in isolation:

```go
func TestUserService_Get(t *testing.T) {
    // Setup
    repo := &MockUserRepository{
        users: map[int]User{
            1: {ID: 1, Username: "test", Email: "test@example.com"},
        },
    }
    service := NewUserService(repo)

    // Test
    user, err := service.Get(1)

    // Assert
    assert.NoError(t, err)
    assert.Equal(t, 1, user.ID)
    assert.Equal(t, "test", user.Username)
}
```

### **20.7.2 Integration Testing**

Test API endpoints with a test server:

```go
func TestUserHandler_GetUser(t *testing.T) {
    // Setup
    app := setupTestApp()

    // Make request
    req := httptest.NewRequest("GET", "/users/1", nil)
    resp, err := app.Test(req)

    // Assert
    assert.NoError(t, err)
    assert.Equal(t, fiber.StatusOK, resp.StatusCode)

    var user User
    json.NewDecoder(resp.Body).Decode(&user)
    assert.Equal(t, 1, user.ID)
}
```

### **20.7.3 Load Testing**

Use tools like k6, Apache JMeter, or Artillery for load testing:

```js
// k6 script
import http from "k6/http";
import { check, sleep } from "k6";

export const options = {
  vus: 100,
  duration: "30s",
};

export default function () {
  const res = http.get("http://localhost:8080/users");
  check(res, {
    "status is 200": (r) => r.status === 200,
    "response time < 200ms": (r) => r.timings.duration < 200,
  });
  sleep(1);
}
```

## **20.8 Deployment Strategies**

### **20.8.1 Containerization with Docker**

Create a Dockerfile for your API:

```dockerfile
FROM golang:1.18-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o api ./cmd/api

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/api .
COPY --from=builder /app/config ./config

EXPOSE 8080
CMD ["./api"]
```

### **20.8.2 Orchestration with Kubernetes**

Deploy your API to Kubernetes:

```yaml
# api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: yourregistry/api:latest
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "256Mi"
          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: api-config
                  key: db_host
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10

---
# api-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

### **20.8.3 CI/CD Pipeline**

Set up continuous integration and deployment:

```yaml
# .github/workflows/deploy.yml
name: Deploy API

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Set up Go
        uses: actions/setup-go@v2
        with:
          go-version: 1.18

      - name: Test
        run: go test -v ./...

      - name: Build Docker image
        run: docker build -t yourregistry/api:${{ github.sha }} .

      - name: Push Docker image
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push yourregistry/api:${{ github.sha }}

      - name: Deploy to Kubernetes
        uses: steebchen/kubectl@master
        env:
          KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA }}
        with:
          args: set image deployment/api api=yourregistry/api:${{ github.sha }}
```

## **20.9 Exercises**

### **Exercise 1: Basic RESTful API**

Build a simple RESTful API for a book management system using the standard library. Implement endpoints for creating, reading, updating, and deleting books.

### **Exercise 2: Framework-Based API**

Rebuild the book management API using Fiber or Echo. Add middleware for logging, error handling, and authentication.

### **Exercise 3: OpenAPI Documentation**

Add OpenAPI documentation to your book management API using Swaggo annotations and implement the Swagger UI.

### **Exercise 4: Advanced Features**

Enhance your API with pagination, filtering, sorting, and caching. Implement proper error handling and validation.

### **Exercise 5: Complete API Project**

Build a complete API project for a blog system with users, posts, and comments. Implement authentication, authorization, and all the best practices covered in this chapter.

## **20.10 Summary**

In this chapter, we've covered comprehensive approaches to building RESTful APIs in Go:

- **Fundamentals**: We explored the core principles of REST and API design
- **Standard Library**: We learned how to build APIs using Go's built-in http package
- **Web Frameworks**: We compared Fiber and Echo for building more feature-rich APIs
- **OpenAPI**: We integrated Swagger documentation for better API discoverability
- **Best Practices**: We covered URL design, status codes, response structures, and security
- **Performance**: We examined techniques for optimizing API performance
- **Testing and Deployment**: We explored strategies for testing and deploying APIs

By following these patterns and practices, you can build robust, maintainable, and scalable RESTful APIs in Go that meet modern standards and deliver excellent performance.

The Go ecosystem offers excellent tools and libraries for API development, and the language's simplicity, performance, and concurrency model make it an ideal choice for building web services and APIs.

**Next Up**: In the next chapter, we'll explore microservices architecture in Go, building on the API development concepts covered here.
