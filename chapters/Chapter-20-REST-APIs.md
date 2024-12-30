
# **Chapter 20: Building REST APIs in Go**

---

## **I. Conceptual Foundation**

### **What is a REST API?**
A REST (Representational State Transfer) API is an architectural style for building web services. It enables clients to interact with servers using HTTP methods such as GET, POST, PUT, and DELETE to perform operations on resources, which are represented in JSON or other formats.

#### **Key Principles of REST**:
1. **Statelessness**: Each request contains all the information the server needs to process it. No session state is stored on the server.
2. **Resource Identification**: Resources are identified by URLs, such as `/users` or `/users/{id}`.
3. **HTTP Verbs**: REST uses standard HTTP methods for actions:
   - **GET**: Retrieve data.
   - **POST**: Create new data.
   - **PUT**: Update existing data.
   - **DELETE**: Remove data.
4. **Representation**: Resources are represented in formats like JSON, XML, or plain text, with JSON being the most common.

### **Why Build REST APIs with Go?**
Go (Golang) is an excellent choice for building REST APIs due to its:
- **Performance**: Native compilation to machine code ensures high performance.
- **Concurrency**: Built-in support for goroutines and channels.
- **Simplicity**: Minimal syntax and robust standard libraries.

### **Understanding HTTP Basics**
Before diving into REST APIs, it’s crucial to understand HTTP basics:
- **Request Structure**:
  - **Method**: Defines the type of operation (GET, POST, etc.).
  - **Headers**: Provide metadata about the request (e.g., `Content-Type`, `Authorization`).
  - **Body**: Carries data for POST and PUT requests, often in JSON format.
- **Response Structure**:
  - **Status Code**: Indicates success (e.g., 200 OK) or failure (e.g., 404 Not Found).
  - **Headers**: Provide metadata about the response.
  - **Body**: Contains the actual resource or error message.

---

### **Real-Life Analogy**
Think of a REST API as a library:
- The **URL** is the catalog where you look up resources (e.g., `/books`).
- The **HTTP Method** specifies what action you want to perform (e.g., borrow a book, return a book).
- The **JSON payload** contains specific details about your request (e.g., the book title or author).

---

### **Designing a REST API**
When designing a REST API, consider:
1. **Resource Modeling**:
   - Identify resources: e.g., Users, Orders, Products.
   - Use nouns for URLs: e.g., `/users`, not `/getUsers`.
2. **URL Structure**:
   - Use hierarchical structure: e.g., `/users/{id}/orders`.
   - Avoid verbs in endpoints.
3. **Versioning**:
   - Include versioning in the URL: e.g., `/api/v1/users`.
4. **Error Handling**:
   - Provide meaningful error messages with appropriate HTTP status codes (e.g., 400 for bad requests).

---

## **II. Practical Content**

### **Setting Up Your Go Project**
#### **1. Installing Dependencies**

Create a new Go module:
```bash
mkdir restapi
cd restapi
go mod init restapi
```

Install required packages:
```bash
go get -u github.com/gorilla/mux
```

### **2. Project Structure**
Organize your project into logical folders:
```
restapi/
├── main.go
├── handlers/
│   └── user_handler.go
├── models/
│   └── user.go
├── middlewares/
│   └── jwt_middleware.go
└── utils/
    └── response.go
```

### **3. Implementing a Basic Server**
Here’s how to set up a simple Go server using Gorilla Mux:

```go
package main

import (
    "log"
    "net/http"

    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()

    r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Welcome to the REST API!"))
    })

    log.Println("Starting server on :8080")
    log.Fatal(http.ListenAndServe(":8080", r))
}
```

---

### **4. CRUD Operations**

#### **Model Definition**
Create a `User` model:
```go
package models

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}
```

#### **Handler for CRUD Operations**

Create `user_handler.go`:

```go
package handlers

import (
    "encoding/json"
    "net/http"
    "restapi/models"
    "strconv"

    "github.com/gorilla/mux"
)

var users = []models.User{
    {ID: 1, Name: "Alice", Email: "alice@example.com"},
}

// Get all users
func GetUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

// Get user by ID
func GetUser(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])

    for _, user := range users {
        if user.ID == id {
            json.NewEncoder(w).Encode(user)
            return
        }
    }
    http.Error(w, "User not found", http.StatusNotFound)
}

// Create a new user
func CreateUser(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    var user models.User
    json.NewDecoder(r.Body).Decode(&user)
    user.ID = len(users) + 1
    users = append(users, user)
    json.NewEncoder(w).Encode(user)
}

// Update user by ID
func UpdateUser(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])
    var updatedUser models.User
    json.NewDecoder(r.Body).Decode(&updatedUser)

    for i, user := range users {
        if user.ID == id {
            users[i] = updatedUser
            users[i].ID = id
            json.NewEncoder(w).Encode(users[i])
            return
        }
    }
    http.Error(w, "User not found", http.StatusNotFound)
}

// Delete user by ID
func DeleteUser(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    params := mux.Vars(r)
    id, _ := strconv.Atoi(params["id"])

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

Update `main.go` to use these routes:

```go
package main

import (
    "log"
    "net/http"
    "restapi/handlers"

    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()

    r.HandleFunc("/users", handlers.GetUsers).Methods("GET")
    r.HandleFunc("/users/{id}", handlers.GetUser).Methods("GET")
    r.HandleFunc("/users", handlers.CreateUser).Methods("POST")
    r.HandleFunc("/users/{id}", handlers.UpdateUser).Methods("PUT")
    r.HandleFunc("/users/{id}", handlers.DeleteUser).Methods("DELETE")

    log.Println("Starting server on :8080")
    log.Fatal(http.ListenAndServe(":8080", r))
}
```

---

### **5. Query Parameters and Filters**

Support query parameters for filtering results:

```go
func GetUsersWithFilter(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    nameFilter := r.URL.Query().Get("name")

    var filteredUsers []models.User
    for _, user := range users {
        if nameFilter == "" || user.Name == nameFilter {
            filteredUsers = append(filteredUsers, user)
        }
    }

    json.NewEncoder(w).Encode(filteredUsers)
}
```

Update `main.go` to include the new route:
```go
r.HandleFunc("/users", handlers.GetUsersWithFilter).Methods("GET")
```

---

## **III. Middleware**

Middleware is used to add common functionality such as logging, authentication, or error handling to your API.

#### **Example: Logging Middleware**
Create `middlewares/logging.go`:

```go
package middlewares

import (
    "log"
    "net/http"
)

func LoggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("%s %s", r.Method, r.URL.Path)
        next.ServeHTTP(w, r)
    })
}
```

Update `main.go` to use the middleware:

```go
r.Use(middlewares.LoggingMiddleware)
```

---

### **Error Handling Middleware**
Ensure all errors return consistent responses:

```go
func ErrorHandlingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        defer func() {
            if err := recover(); err != nil {
                http.Error(w, "Internal Server Error", http.StatusInternalServerError)
            }
        }()
        next.ServeHTTP(w, r)
    })
}
```

Add it to `main.go`:
```go
r.Use(middlewares.ErrorHandlingMiddleware)
```

---

## **IV. Authentication with JWT**

#### **What is JWT?**
JWT (JSON Web Token) is a compact, self-contained token used for secure communication. It contains encoded header, payload, and signature.

#### **Example: JWT Middleware**
Install the JWT package:
```bash
go get github.com/dgrijalva/jwt-go
```

Create `middlewares/jwt_middleware.go`:

```go
package middlewares

import (
    "net/http"
    "strings"

    "github.com/dgrijalva/jwt-go"
)

var jwtKey = []byte("secret")

func JWTMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Forbidden", http.StatusForbidden)
            return
        }

        tokenString := strings.TrimPrefix(authHeader, "Bearer ")
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return jwtKey, nil
        })

        if err != nil || !token.Valid {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```

Update `main.go`:

```go
r.Use(middlewares.JWTMiddleware)
```

---

## **V. Real-World Scenarios**

### **Scenario 1: Securing Endpoints with JWT**
Imagine you are building an application that requires user authentication to access certain sensitive resources, such as user profile details or account settings. With JWT middleware in place:
1. When a user logs in, they receive a JWT token.
2. For every request to protected routes like `/users/{id}`, the client must include the token in the `Authorization` header.
3. The middleware verifies the token. If it’s valid, the request proceeds. If not, the server returns a 401 Unauthorized status.

This ensures that only authenticated users can access sensitive information.

---

### **Scenario 2: Implementing Pagination**
Suppose you are developing an API for a social media platform with millions of users. Fetching all users in one request would be inefficient. Instead, implement pagination:
1. The client sends a request to `/users?limit=10&page=2`.
2. The server retrieves 10 users starting from the 11th user and returns them as a response.

Example Code:
```go
func GetPaginatedUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    limit, _ := strconv.Atoi(r.URL.Query().Get("limit"))
    page, _ := strconv.Atoi(r.URL.Query().Get("page"))

    start := (page - 1) * limit
    end := start + limit

    if start >= len(users) {
        json.NewEncoder(w).Encode([]models.User{})
        return
    }

    if end > len(users) {
        end = len(users)
    }

    json.NewEncoder(w).Encode(users[start:end])
}
```

---

### **Scenario 3: Rate Limiting**
To prevent abuse of your API, implement rate limiting to cap the number of requests a client can make in a given timeframe. For instance:
1. Allow each client to make up to 100 requests per minute.
2. If the limit is exceeded, return a 429 Too Many Requests status.

While implementing rate limiting requires additional infrastructure or libraries (like Redis), you can use an in-memory counter for a basic example:
```go
var requestCounts = make(map[string]int)

func RateLimitingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        clientIP := r.RemoteAddr
        requestCounts[clientIP]++

        if requestCounts[clientIP] > 100 {
            http.Error(w, "Too Many Requests", http.StatusTooManyRequests)
            return
        }

        next.ServeHTTP(w, r)
    })
}
```

---

## **Conclusion**
In this chapter, you’ve learned to:
1. Build a REST API in Go with Gorilla Mux.
2. Implement CRUD operations and handle JSON.
3. Use middleware for logging, error handling, and authentication.
4. Secure your API with JWT.
5. Extend APIs with filters, pagination, and rate limiting.

Next, we’ll explore **Database Integration** to persist data in your REST API.
