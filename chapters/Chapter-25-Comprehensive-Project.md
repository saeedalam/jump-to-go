# **Chapter 25: Bringing It All Together**

---

## **25.1. Introduction**

After exploring the fundamentals, advanced concepts, and practical applications of Go, this final chapter is dedicated to applying everything you’ve learned. We will build a comprehensive project that ties together:

- REST APIs
- Database integration
- Concurrency patterns
- Testing and benchmarking
- Clean code principles

This chapter is structured as a practical guide to designing, building, and deploying a real-world application: a **Task Management System**.

---

## **25.2. Project Overview**

### **Scenario: Task Management System**

You will create a task management system where users can:

1. Register and log in.
2. Create, update, delete, and view tasks.
3. Organize tasks with tags and categories.
4. Mark tasks as completed or pending.
5. View tasks by due date, priority, or status.

### **Key Features**

- **Authentication**: User login and JWT-based authentication.
- **RESTful APIs**: Expose endpoints for managing tasks.
- **Database**: Use PostgreSQL for storing users and tasks.
- **Concurrency**: Process notifications asynchronously.
- **Testing**: Write unit and integration tests for reliability.

---

## **25.3. Setting Up the Project**

### **1. Project Structure**

Organize the project for maintainability and scalability.

```
task-manager/
├── main.go
├── handlers/
│   ├── auth.go
│   ├── tasks.go
├── models/
│   ├── user.go
│   ├── task.go
├── services/
│   ├── auth_service.go
│   ├── task_service.go
├── database/
│   └── connection.go
├── middlewares/
│   └── auth_middleware.go
├── tests/
│   ├── auth_test.go
│   ├── tasks_test.go
├── utils/
│   ├── jwt.go
│   ├── logger.go
```

---

### **2. Initialize the Project**

Create a new Go project and install necessary dependencies:

```bash
mkdir task-manager
cd task-manager
go mod init task-manager
go get -u github.com/gorilla/mux
go get -u gorm.io/gorm
go get -u gorm.io/driver/postgres
go get -u github.com/golang-jwt/jwt/v5
```

---

### **3. Connect to the Database**

Set up a PostgreSQL connection.

**Code: database/connection.go**

```go
package database

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"log"
)

var DB *gorm.DB

func Connect() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=taskmanager port=5432 sslmode=disable"
	var err error
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to the database:", err)
	}

	log.Println("Database connected successfully")
}
```

---

## **25.4. Implementing User Authentication**

### **1. User Model**

Define the `User` model with fields for ID, name, email, and password.

**Code: models/user.go**

```go
package models

import "gorm.io/gorm"

type User struct {
	gorm.Model
	Name     string `gorm:"not null"`
	Email    string `gorm:"unique;not null"`
	Password string `gorm:"not null"`
}
```

---

### **2. User Registration**

Create an endpoint for user registration.

**Code: handlers/auth.go**

```go
package handlers

import (
	"encoding/json"
	"net/http"
	"task-manager/database"
	"task-manager/models"
	"golang.org/x/crypto/bcrypt"
)

func Register(w http.ResponseWriter, r *http.Request) {
	var user models.User
	err := json.NewDecoder(r.Body).Decode(&user)
	if err != nil {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	hashedPassword, _ := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	user.Password = string(hashedPassword)

	result := database.DB.Create(&user)
	if result.Error != nil {
		http.Error(w, "Could not register user", http.StatusInternalServerError)
		return
	}

	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(map[string]string{"message": "User registered successfully"})
}
```

---

### **3. User Login**

Generate a JWT token for authenticated users.

**Code: handlers/auth.go**

```go
func Login(w http.ResponseWriter, r *http.Request) {
	var credentials struct {
		Email    string `json:"email"`
		Password string `json:"password"`
	}

	err := json.NewDecoder(r.Body).Decode(&credentials)
	if err != nil {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	var user models.User
	database.DB.Where("email = ?", credentials.Email).First(&user)
	if user.ID == 0 || bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(credentials.Password)) != nil {
		http.Error(w, "Invalid credentials", http.StatusUnauthorized)
		return
	}

	token := GenerateJWT(user.ID)
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(map[string]string{"token": token})
}
```

**Code: utils/jwt.go**

```go
package utils

import (
	"time"
	"github.com/golang-jwt/jwt/v5"
)

var jwtKey = []byte("your_secret_key")

func GenerateJWT(userID uint) string {
	claims := &jwt.MapClaims{
		"userID": userID,
		"exp":    time.Now().Add(time.Hour * 24).Unix(),
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	signedToken, _ := token.SignedString(jwtKey)
	return signedToken
}
```

---

# **Chapter 25: Bringing It All Together (Next Steps)**

---

## **25.6. Task Management Endpoints**

Now that we have user authentication in place, the next step is to implement CRUD (Create, Read, Update, Delete) operations for managing tasks.

### **1. Task Model**

Define the `Task` model with fields for title, description, status, due date, and user ID.

**Code: models/task.go**

```go
package models

import "gorm.io/gorm"

type Task struct {
	gorm.Model
	Title       string `gorm:"not null"`
	Description string
	Status      string `gorm:"default:'pending'"`
	DueDate     string
	UserID      uint `gorm:"not null"`
}
```

---

### **2. Create a Task**

Allow authenticated users to create tasks.

**Code: handlers/tasks.go**

```go
package handlers

import (
	"encoding/json"
	"net/http"
	"task-manager/database"
	"task-manager/middlewares"
	"task-manager/models"
)

func CreateTask(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())

	var task models.Task
	err := json.NewDecoder(r.Body).Decode(&task)
	if err != nil {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	task.UserID = userID
	database.DB.Create(&task)

	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(task)
}
```

---

### **3. List Tasks**

Retrieve tasks for the authenticated user.

**Code: handlers/tasks.go**

```go
func ListTasks(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())

	var tasks []models.Task
	database.DB.Where("user_id = ?", userID).Find(&tasks)

	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(tasks)
}
```

---

### **4. Update a Task**

Update task details such as title, description, or status.

**Code: handlers/tasks.go**

```go
func UpdateTask(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())

	var task models.Task
	err := json.NewDecoder(r.Body).Decode(&task)
	if err != nil || task.ID == 0 {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	var existingTask models.Task
	database.DB.Where("id = ? AND user_id = ?", task.ID, userID).First(&existingTask)
	if existingTask.ID == 0 {
		http.Error(w, "Task not found", http.StatusNotFound)
		return
	}

	database.DB.Model(&existingTask).Updates(task)
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(existingTask)
}
```

---

### **5. Delete a Task**

Allow users to delete their tasks.

**Code: handlers/tasks.go**

```go
func DeleteTask(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())

	var task models.Task
	err := json.NewDecoder(r.Body).Decode(&task)
	if err != nil || task.ID == 0 {
		http.Error(w, "Invalid request payload", http.StatusBadRequest)
		return
	}

	result := database.DB.Where("id = ? AND user_id = ?", task.ID, userID).Delete(&task)
	if result.RowsAffected == 0 {
		http.Error(w, "Task not found", http.StatusNotFound)
		return
	}

	w.WriteHeader(http.StatusNoContent)
}
```

---

## **25.7. Middleware for Authentication**

### **1. Protect Endpoints**

Add a middleware to validate JWT tokens and attach the user ID to the request context.

**Code: middlewares/auth_middleware.go**

```go
package middlewares

import (
	"context"
	"net/http"
	"strings"
	"task-manager/utils"
)

func AuthMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		authHeader := r.Header.Get("Authorization")
		if authHeader == "" {
			http.Error(w, "Missing authorization header", http.StatusUnauthorized)
			return
		}

		tokenString := strings.TrimPrefix(authHeader, "Bearer ")
		userID, err := utils.ParseJWT(tokenString)
		if err != nil {
			http.Error(w, "Invalid token", http.StatusUnauthorized)
			return
		}

		ctx := context.WithValue(r.Context(), "userID", userID)
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

func GetUserIDFromContext(ctx context.Context) uint {
	if userID, ok := ctx.Value("userID").(uint); ok {
		return userID
	}
	return 0
}
```

**Code: utils/jwt.go**

```go
package utils

import (
	"errors"
	"github.com/golang-jwt/jwt/v5"
)

var jwtKey = []byte("your_secret_key")

func ParseJWT(tokenString string) (uint, error) {
	token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
		return jwtKey, nil
	})

	if err != nil || !token.Valid {
		return 0, errors.New("invalid token")
	}

	claims, ok := token.Claims.(jwt.MapClaims)
	if !ok {
		return 0, errors.New("invalid claims")
	}

	userID, ok := claims["userID"].(float64)
	if !ok {
		return 0, errors.New("invalid userID")
	}

	return uint(userID), nil
}
```

---

## **25.8. Registering API Routes**

Update the `main` function to include routes for task management.

**Code: main.go**

```go
package main

import (
	"log"
	"net/http"
	"task-manager/database"
	"task-manager/handlers"
	"task-manager/middlewares"

	"github.com/gorilla/mux"
)

func main() {
	// Initialize database
	database.Connect()

	// Set up router
	router := mux.NewRouter()

	// Auth routes
	router.HandleFunc("/register", handlers.Register).Methods("POST")
	router.HandleFunc("/login", handlers.Login).Methods("POST")

	// Task routes (protected)
	api := router.PathPrefix("/api").Subrouter()
	api.Use(middlewares.AuthMiddleware)
	api.HandleFunc("/tasks", handlers.ListTasks).Methods("GET")
	api.HandleFunc("/tasks", handlers.CreateTask).Methods("POST")
	api.HandleFunc("/tasks", handlers.UpdateTask).Methods("PUT")
	api.HandleFunc("/tasks", handlers.DeleteTask).Methods("DELETE")

	// Start server
	log.Println("Server running on port 8080")
	log.Fatal(http.ListenAndServe(":8080", router))
}
```

---

## **25.9. Testing the Application**

### **Unit Tests**

Write tests for individual functions like `CreateTask`, `Login`, and `Register`.

**Example Test: Register**

```go
func TestRegister(t *testing.T) {
	// Simulate HTTP request and response
}
```

---

## **25.10. Conclusion**

At this stage, you have built a complete Task Management System with:

- Authentication
- Task CRUD operations
- Middleware for secure endpoints

# **Chapter 25: Bringing It All Together (Final Steps)**

---

## **25.11. Enhancing the Task Management System**

Now that the basic Task Management System is functional, the next step is to add advanced features to improve usability and functionality.

---

### **1. Task Notifications**

Implement notifications to remind users about upcoming or overdue tasks.

#### **Asynchronous Task Processing**

Use Go’s concurrency features to send notifications asynchronously.

**Code: services/notification_service.go**

```go
package services

import (
	"fmt"
	"time"
)

type NotificationService struct{}

func (ns NotificationService) NotifyDueTasks(tasks []Task) {
	for _, task := range tasks {
		if isTaskDue(task.DueDate) {
			go sendNotification(task)
		}
	}
}

func isTaskDue(dueDate string) bool {
	parsedDate, _ := time.Parse("2006-01-02", dueDate)
	return time.Now().After(parsedDate)
}

func sendNotification(task Task) {
	fmt.Printf("Notification: Task '%s' is due!
", task.Title)
}
```

Integrate this into the system by triggering notifications at regular intervals or when users log in.

---

### **2. Task Sorting and Filtering**

Allow users to sort and filter tasks based on criteria such as due date, priority, or status.

#### **Sorting Example**

**Code: handlers/tasks.go**

```go
func ListSortedTasks(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())
	sortBy := r.URL.Query().Get("sortBy") // e.g., "dueDate" or "status"

	var tasks []models.Task
	database.DB.Where("user_id = ?", userID).Order(sortBy).Find(&tasks)

	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(tasks)
}
```

#### **Filtering Example**

**Code: handlers/tasks.go**

```go
func ListFilteredTasks(w http.ResponseWriter, r *http.Request) {
	userID := middlewares.GetUserIDFromContext(r.Context())
	status := r.URL.Query().Get("status") // e.g., "completed" or "pending"

	var tasks []models.Task
	database.DB.Where("user_id = ? AND status = ?", userID, status).Find(&tasks)

	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(tasks)
}
```

---

### **3. Deploying the Application**

Deploy the Task Management System to a cloud provider for real-world usage.

#### **Steps for Deployment**

1. **Containerize with Docker**
   Create a `Dockerfile` for your application.

   **Code: Dockerfile**

   ```dockerfile
   FROM golang:1.20-alpine
   WORKDIR /app
   COPY . .
   RUN go build -o task-manager
   CMD ["./task-manager"]
   ```

2. **Set Up a PostgreSQL Database**
   Use a managed database service like AWS RDS, Google Cloud SQL, or Azure Database.

3. **Deploy to a Cloud Provider**
   Use services like AWS EC2, Google Cloud Run, or Heroku for deployment.

4. **Configure Environment Variables**
   Store sensitive information like database credentials and JWT secrets securely.

---

## **25.12. Scaling the Application**

To handle more users and tasks, scale the application by implementing the following:

### **1. Caching**

Use a caching layer like Redis to store frequently accessed data, such as user sessions or task lists.

**Example: Caching Task Lists**

```go
import "github.com/go-redis/redis/v8"

var redisClient *redis.Client

func init() {
    redisClient = redis.NewClient(&redis.Options{
        Addr: "localhost:6379",
    })
}

func CacheTaskList(userID uint, tasks []models.Task) {
    taskJSON, _ := json.Marshal(tasks)
    redisClient.Set(ctx, fmt.Sprintf("tasks:%d", userID), taskJSON, time.Hour)
}
```

---

### **2. Load Balancing**

Use a load balancer to distribute traffic across multiple instances of the application.

---

### **3. Rate Limiting**

Prevent abuse by implementing rate-limiting middleware to limit the number of requests per user.

**Code: middlewares/rate_limit_middleware.go**

```go
package middlewares

import (
	"net/http"
	"sync"
	"time"
)

var rateLimit = make(map[string]int)
var mu sync.Mutex

func RateLimitMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		mu.Lock()
		defer mu.Unlock()

		ip := r.RemoteAddr
		rateLimit[ip]++

		if rateLimit[ip] > 100 {
			http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
			return
		}

		go resetRateLimit(ip)
		next.ServeHTTP(w, r)
	})
}

func resetRateLimit(ip string) {
	time.Sleep(time.Minute)
	mu.Lock()
	defer mu.Unlock()
	rateLimit[ip] = 0
}
```

---

## **25.13. Final Testing**

### **1. Write Integration Tests**

Test the entire system by simulating real-world scenarios, such as user registration, login, task creation, and notifications.

**Example: Integration Test**

```go
func TestIntegration(t *testing.T) {
	// Simulate user registration
	// Simulate task creation
	// Validate notifications
}
```

### **2. Measure Performance**

Use benchmarking to test the system under load.

---

## **25.14. Conclusion**

Congratulations! You’ve built a full-featured, scalable Task Management System that incorporates:

- REST APIs
- Database integration
- Concurrency
- Notifications
- Testing and Deployment

This project demonstrates the power of Go for building robust and scalable systems.

---
