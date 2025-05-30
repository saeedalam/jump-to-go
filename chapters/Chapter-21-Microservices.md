# **Chapter 21: Building Microservices in Go**

Go has emerged as one of the most popular languages for building microservices due to its simplicity, performance, concurrency model, and small memory footprint. This chapter explores microservices architecture patterns and implementation techniques in Go, building on the RESTful API concepts from the previous chapter.

## **21.1 Introduction to Microservices**

### **21.1.1 What Are Microservices?**

Microservices architecture is an approach to application development where a large application is built as a suite of small, independently deployable services. Each service runs in its own process, communicates through well-defined APIs, and is owned by a small team.

Key characteristics of microservices include:

1. **Service Independence**: Each service can be developed, deployed, operated, and scaled independently
2. **Domain-Driven Design**: Services are organized around business capabilities
3. **Decentralized Data Management**: Each service manages its own database
4. **Smart Endpoints, Dumb Pipes**: Complex processing happens within services, while communication is simple
5. **Designed for Failure**: Services are resilient and can handle failures gracefully
6. **Evolutionary Design**: The architecture evolves over time based on business needs

### **21.1.2 Monolithic vs. Microservices Architecture**

| Aspect          | Monolithic                                 | Microservices                               |
| --------------- | ------------------------------------------ | ------------------------------------------- |
| Development     | Simple at first, becomes complex over time | Complex initially, simplifies with maturity |
| Deployment      | Single unit deployment                     | Independent service deployment              |
| Scaling         | Scale entire application                   | Scale individual services as needed         |
| Reliability     | Single point of failure                    | Isolated failures                           |
| Technology      | One technology stack                       | Polyglot architecture possible              |
| Team Structure  | Organized by technical layers              | Organized by business domains               |
| Data Management | Shared database                            | Database per service                        |
| Communication   | In-process function calls                  | Network calls (REST, gRPC, messaging)       |

### **21.1.3 When to Use Microservices**

Microservices are not a silver bullet. Consider these factors when deciding on an architecture:

**Use microservices when:**

- Building large, complex applications
- Needing independent scaling of components
- Requiring frequent, independent deployments
- Working with multiple teams on the same application
- Planning for long-term evolution of the system

**Consider monoliths when:**

- Building simple applications
- Starting a new project with uncertain requirements
- Working with a small team
- Lacking expertise in distributed systems
- Having tight deadlines for initial deployment

### **21.1.4 Go's Suitability for Microservices**

Go offers several advantages for microservices development:

1. **Fast Compilation**: Quick feedback cycle during development
2. **Small Binary Size**: Lower container image sizes and faster deployments
3. **Low Memory Footprint**: Efficient resource utilization
4. **Built-in Concurrency**: Natural handling of concurrent requests
5. **Standard Library HTTP Support**: Solid foundation for service communication
6. **Cross-Compilation**: Build for different platforms easily
7. **Simplicity**: Easier to onboard new developers

## **21.2 Designing Microservices Architecture**

### **21.2.1 Service Decomposition Strategies**

Splitting a system into microservices requires careful consideration:

#### **Domain-Driven Design (DDD)**

DDD provides a framework for defining service boundaries based on business domains:

```
Customer Domain:
- Customer Registration
- Customer Profile Management
- Customer Authentication

Order Domain:
- Order Creation
- Order Processing
- Order Fulfillment

Inventory Domain:
- Stock Management
- Warehouse Operations
- Inventory Tracking
```

#### **Business Capability Pattern**

Organize services around business capabilities:

```
User Service:
- User registration, authentication, profiles

Product Service:
- Product catalog, pricing, inventory

Order Service:
- Order processing, payment, fulfillment

Notification Service:
- Email, SMS, push notifications
```

#### **Strangler Pattern**

Gradually migrate from a monolith to microservices:

1. Identify a bounded context to extract
2. Create a new service for that context
3. Redirect traffic from the monolith to the new service
4. Repeat for other contexts

### **21.2.2 Service Communication Patterns**

There are several ways microservices can communicate:

#### **Synchronous Communication**

- **REST APIs**: HTTP-based communication with JSON/XML payloads
- **gRPC**: High-performance RPC framework using Protocol Buffers
- **GraphQL**: Query language for APIs with client-specified data retrieval

#### **Asynchronous Communication**

- **Message Queues**: Point-to-point communication (RabbitMQ, Amazon SQS)
- **Pub/Sub**: One-to-many communication (NATS, Kafka, Google Pub/Sub)
- **Event Streaming**: Real-time event processing (Kafka, Amazon Kinesis)

#### **Communication Styles**

- **Request/Response**: Client makes a request and waits for a response
- **Event-Driven**: Services emit and react to events
- **Command Query Responsibility Segregation (CQRS)**: Separate read and write operations

### **21.2.3 Service Discovery and Registration**

Services need to locate and communicate with each other:

#### **Service Registry**

A central repository of service locations:

- **Consul**: Feature-rich service discovery with health checking
- **etcd**: Distributed key-value store
- **ZooKeeper**: Centralized service for configuration and synchronization

#### **Client-Side Discovery**

The client is responsible for determining the location of a service instance:

```go
package main

import (
    "log"

    "github.com/hashicorp/consul/api"
)

func getServiceURL(serviceName string) (string, error) {
    // Configure Consul client
    config := api.DefaultConfig()
    client, err := api.NewClient(config)
    if err != nil {
        return "", err
    }

    // Query for service
    services, _, err := client.Catalog().Service(serviceName, "", nil)
    if err != nil {
        return "", err
    }

    if len(services) == 0 {
        return "", fmt.Errorf("service '%s' not found", serviceName)
    }

    // Return first available instance
    service := services[0]
    return fmt.Sprintf("http://%s:%d", service.ServiceAddress, service.ServicePort), nil
}
```

#### **Server-Side Discovery**

A load balancer or router handles service discovery:

```
Client -> API Gateway -> Service Registry -> Service Instance
```

### **21.2.4 API Gateway Pattern**

An API Gateway provides a single entry point for clients:

```
                   ┌─────────────────┐
                   │                 │
 ┌─────────┐       │  User Service   │
 │         │       │                 │
 │ Client  │◄─────►│                 │
 │         │       └─────────────────┘
 └─────────┘       ┌─────────────────┐
      │            │                 │
      │            │  Order Service  │
      │            │                 │
      ▼            │                 │
┌───────────┐      └─────────────────┘
│           │      ┌─────────────────┐
│    API    │      │                 │
│  Gateway  │◄────►│ Product Service │
│           │      │                 │
└───────────┘      │                 │
                   └─────────────────┘
```

Benefits of an API Gateway:

- **Single entry point** for all client requests
- **Request routing** to appropriate services
- **Protocol translation** (e.g., HTTP to gRPC)
- **Authentication and authorization**
- **Rate limiting and throttling**
- **Response caching**
- **Request/response transformation**
- **Service aggregation**

## **21.3 Building Microservices in Go**

### **21.3.1 Service Template**

Let's define a template for a Go microservice:

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gorilla/mux"
)

func main() {
    // Create router
    r := mux.NewRouter()

    // Register routes
    r.HandleFunc("/health", healthHandler).Methods("GET")
    r.HandleFunc("/api/v1/resource", getResourcesHandler).Methods("GET")
    r.HandleFunc("/api/v1/resource", createResourceHandler).Methods("POST")
    r.HandleFunc("/api/v1/resource/{id}", getResourceHandler).Methods("GET")

    // Create server
    srv := &http.Server{
        Addr:         ":8080",
        Handler:      r,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 10 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Start server in a goroutine
    go func() {
        log.Println("Starting server on port 8080")
        if err := srv.ListenAndServe(); err != nil && err != http.ErrServerClosed {
            log.Fatalf("Server failed to start: %v", err)
        }
    }()

    // Graceful shutdown
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Server is shutting down...")

    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("Server forced to shutdown: %v", err)
    }

    log.Println("Server exited properly")
}

func healthHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusOK)
    w.Write([]byte(`{"status":"healthy"}`))
}

func getResourcesHandler(w http.ResponseWriter, r *http.Request) {
    // Implementation
}

func createResourceHandler(w http.ResponseWriter, r *http.Request) {
    // Implementation
}

func getResourceHandler(w http.ResponseWriter, r *http.Request) {
    // Implementation
}
```

This template includes:

- Router setup with common endpoints
- Graceful shutdown handling
- Health check endpoint
- Appropriate timeout configurations

### **21.3.2 Service Configuration**

Manage service configuration with environment variables:

```go
package config

import (
    "fmt"
    "os"
    "strconv"
    "time"
)

// Config holds all configuration for the service
type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Logging  LoggingConfig
}

// ServerConfig holds server configuration
type ServerConfig struct {
    Port            int
    ReadTimeout     time.Duration
    WriteTimeout    time.Duration
    ShutdownTimeout time.Duration
}

// DatabaseConfig holds database configuration
type DatabaseConfig struct {
    Host     string
    Port     int
    User     string
    Password string
    Database string
    SSLMode  string
}

// LoggingConfig holds logging configuration
type LoggingConfig struct {
    Level string
}

// Load returns configuration loaded from environment variables
func Load() (*Config, error) {
    port, err := strconv.Atoi(getEnv("SERVER_PORT", "8080"))
    if err != nil {
        return nil, fmt.Errorf("invalid port: %w", err)
    }

    dbPort, err := strconv.Atoi(getEnv("DB_PORT", "5432"))
    if err != nil {
        return nil, fmt.Errorf("invalid database port: %w", err)
    }

    readTimeout, err := time.ParseDuration(getEnv("SERVER_READ_TIMEOUT", "10s"))
    if err != nil {
        return nil, fmt.Errorf("invalid read timeout: %w", err)
    }

    writeTimeout, err := time.ParseDuration(getEnv("SERVER_WRITE_TIMEOUT", "10s"))
    if err != nil {
        return nil, fmt.Errorf("invalid write timeout: %w", err)
    }

    shutdownTimeout, err := time.ParseDuration(getEnv("SERVER_SHUTDOWN_TIMEOUT", "30s"))
    if err != nil {
        return nil, fmt.Errorf("invalid shutdown timeout: %w", err)
    }

    return &Config{
        Server: ServerConfig{
            Port:            port,
            ReadTimeout:     readTimeout,
            WriteTimeout:    writeTimeout,
            ShutdownTimeout: shutdownTimeout,
        },
        Database: DatabaseConfig{
            Host:     getEnv("DB_HOST", "localhost"),
            Port:     dbPort,
            User:     getEnv("DB_USER", "postgres"),
            Password: getEnv("DB_PASSWORD", "password"),
            Database: getEnv("DB_NAME", "service"),
            SSLMode:  getEnv("DB_SSLMODE", "disable"),
        },
        Logging: LoggingConfig{
            Level: getEnv("LOG_LEVEL", "info"),
        },
    }, nil
}

// Helper function to get environment variable with a default value
func getEnv(key, defaultValue string) string {
    value := os.Getenv(key)
    if value == "" {
        return defaultValue
    }
    return value
}
```

### **21.3.3 Structured Logging**

Implement structured logging for better observability:

```go
package logger

import (
    "os"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

// NewLogger creates a new structured logger
func NewLogger(level string) (*zap.Logger, error) {
    logLevel, err := zapcore.ParseLevel(level)
    if err != nil {
        return nil, err
    }

    config := zap.NewProductionEncoderConfig()
    config.EncodeTime = zapcore.ISO8601TimeEncoder

    core := zapcore.NewCore(
        zapcore.NewJSONEncoder(config),
        zapcore.AddSync(os.Stdout),
        logLevel,
    )

    return zap.New(core, zap.AddCaller()), nil
}

// Example usage:
// log, err := logger.NewLogger("info")
// if err != nil {
//     panic(err)
// }
// defer log.Sync()
//
// log.Info("Server starting",
//     zap.Int("port", 8080),
//     zap.String("environment", "production"))
```

## **21.4 Microservices Communication**

### **21.4.1 REST-based Communication**

REST is a common approach for microservice communication:

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

// UserService client for REST communication
type UserService struct {
    baseURL    string
    httpClient *http.Client
}

// NewUserService creates a new UserService client
func NewUserService(baseURL string) *UserService {
    return &UserService{
        baseURL: baseURL,
        httpClient: &http.Client{
            Timeout: 5 * time.Second,
        },
    }
}

// User represents a user in the system
type User struct {
    ID        int    `json:"id"`
    Username  string `json:"username"`
    Email     string `json:"email"`
    CreatedAt string `json:"created_at"`
}

// GetUser retrieves a user by ID
func (s *UserService) GetUser(id int) (*User, error) {
    url := fmt.Sprintf("%s/users/%d", s.baseURL, id)

    resp, err := s.httpClient.Get(url)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusOK {
        return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
    }

    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        return nil, fmt.Errorf("failed to decode response: %w", err)
    }

    return &user, nil
}

// CreateUser creates a new user
func (s *UserService) CreateUser(user User) (*User, error) {
    url := fmt.Sprintf("%s/users", s.baseURL)

    data, err := json.Marshal(user)
    if err != nil {
        return nil, fmt.Errorf("failed to marshal user: %w", err)
    }

    resp, err := s.httpClient.Post(url, "application/json", bytes.NewBuffer(data))
    if err != nil {
        return nil, fmt.Errorf("failed to create user: %w", err)
    }
    defer resp.Body.Close()

    if resp.StatusCode != http.StatusCreated {
        return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
    }

    var createdUser User
    if err := json.NewDecoder(resp.Body).Decode(&createdUser); err != nil {
        return nil, fmt.Errorf("failed to decode response: %w", err)
    }

    return &createdUser, nil
}

// Example usage:
// userService := NewUserService("http://user-service:8080")
// user, err := userService.GetUser(123)
```

While REST is simple to implement, it has limitations for microservices:

- Overhead of HTTP for each request
- No strict contract for API definition
- Limited to request/response pattern
- Serialization/deserialization costs with JSON

### **21.4.2 gRPC Communication**

gRPC is a high-performance RPC framework that addresses many REST limitations:

First, define your service using Protocol Buffers:

```protobuf
// user.proto
syntax = "proto3";

package user;
option go_package = "github.com/yourorg/userservice/proto";

import "google/protobuf/timestamp.proto";

service UserService {
  rpc GetUser(GetUserRequest) returns (User) {}
  rpc CreateUser(CreateUserRequest) returns (User) {}
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse) {}
  rpc UpdateUser(UpdateUserRequest) returns (User) {}
  rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse) {}
}

message GetUserRequest {
  int64 id = 1;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
}

message UpdateUserRequest {
  int64 id = 1;
  string username = 2;
  string email = 3;
}

message DeleteUserRequest {
  int64 id = 1;
}

message DeleteUserResponse {
  bool success = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 page_size = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

message User {
  int64 id = 1;
  string username = 2;
  string email = 3;
  google.protobuf.Timestamp created_at = 4;
}
```

Implementing the gRPC server:

```go
package main

import (
    "context"
    "log"
    "net"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/protobuf/types/known/timestamppb"

    pb "github.com/yourorg/userservice/proto"
)

type userServiceServer struct {
    pb.UnimplementedUserServiceServer
    // In a real implementation, add repository dependencies here
}

func (s *userServiceServer) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    // In a real implementation, fetch from database
    // This is a mock response
    return &pb.User{
        Id:        req.Id,
        Username:  "testuser",
        Email:     "test@example.com",
        CreatedAt: timestamppb.New(time.Now()),
    }, nil
}

func (s *userServiceServer) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.User, error) {
    // In a real implementation, save to database
    // This is a mock response
    return &pb.User{
        Id:        123, // Generated ID
        Username:  req.Username,
        Email:     req.Email,
        CreatedAt: timestamppb.New(time.Now()),
    }, nil
}

func (s *userServiceServer) ListUsers(ctx context.Context, req *pb.ListUsersRequest) (*pb.ListUsersResponse, error) {
    // Implementation omitted for brevity
    return &pb.ListUsersResponse{}, nil
}

func (s *userServiceServer) UpdateUser(ctx context.Context, req *pb.UpdateUserRequest) (*pb.User, error) {
    // Implementation omitted for brevity
    return &pb.User{}, nil
}

func (s *userServiceServer) DeleteUser(ctx context.Context, req *pb.DeleteUserRequest) (*pb.DeleteUserResponse, error) {
    // Implementation omitted for brevity
    return &pb.DeleteUserResponse{Success: true}, nil
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }

    grpcServer := grpc.NewServer()
    pb.RegisterUserServiceServer(grpcServer, &userServiceServer{})

    log.Println("gRPC server starting on port 50051")
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

Implementing the gRPC client:

```go
package main

import (
    "context"
    "log"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"

    pb "github.com/yourorg/userservice/proto"
)

func main() {
    // Set up connection to server
    conn, err := grpc.Dial("user-service:50051", grpc.WithTransportCredentials(insecure.NewCredentials()))
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()

    // Create client
    client := pb.NewUserServiceClient(conn)

    // Set timeout
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()

    // Get user
    user, err := client.GetUser(ctx, &pb.GetUserRequest{Id: 123})
    if err != nil {
        log.Fatalf("could not get user: %v", err)
    }
    log.Printf("User: %s (%s)", user.Username, user.Email)

    // Create user
    newUser, err := client.CreateUser(ctx, &pb.CreateUserRequest{
        Username: "newuser",
        Email:    "new@example.com",
    })
    if err != nil {
        log.Fatalf("could not create user: %v", err)
    }
    log.Printf("Created user with ID: %d", newUser.Id)
}
```

Advantages of gRPC:

- Strongly typed contracts with Protocol Buffers
- Efficient binary serialization
- Built-in code generation for multiple languages
- Support for streaming (unary, server, client, and bidirectional)
- HTTP/2 for multiplexing requests and header compression

### **21.4.3 Event-Driven Architecture**

Event-driven architecture enables loose coupling between services:

```
┌───────────┐     ┌─────────────┐     ┌───────────────┐
│           │     │             │     │               │
│  Service  │────►│  Message    │────►│  Service B    │
│     A     │     │  Broker     │     │               │
│           │     │             │     └───────────────┘
└───────────┘     │             │     ┌───────────────┐
                  │             │────►│               │
                  │             │     │  Service C    │
                  └─────────────┘     │               │
                                      └───────────────┘
```

Using NATS for event-driven communication:

```go
package main

import (
    "encoding/json"
    "log"
    "time"

    "github.com/nats-io/nats.go"
)

// UserCreatedEvent represents a user creation event
type UserCreatedEvent struct {
    ID        int       `json:"id"`
    Username  string    `json:"username"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

// PublishUserCreated publishes a user created event
func PublishUserCreated(nc *nats.Conn, user UserCreatedEvent) error {
    data, err := json.Marshal(user)
    if err != nil {
        return err
    }

    return nc.Publish("user.created", data)
}

// SubscribeToUserCreated subscribes to user created events
func SubscribeToUserCreated(nc *nats.Conn, handler func(UserCreatedEvent)) error {
    _, err := nc.Subscribe("user.created", func(msg *nats.Msg) {
        var event UserCreatedEvent
        if err := json.Unmarshal(msg.Data, &event); err != nil {
            log.Printf("Error unmarshaling event: %v", err)
            return
        }

        handler(event)
    })

    return err
}

func main() {
    // Connect to NATS
    nc, err := nats.Connect(nats.DefaultURL)
    if err != nil {
        log.Fatalf("Error connecting to NATS: %v", err)
    }
    defer nc.Close()

    // Subscribe to user created events
    err = SubscribeToUserCreated(nc, func(event UserCreatedEvent) {
        log.Printf("Received user created event: %+v", event)
        // Process the event
    })
    if err != nil {
        log.Fatalf("Error subscribing: %v", err)
    }

    // Publish a user created event
    event := UserCreatedEvent{
        ID:        123,
        Username:  "newuser",
        Email:     "new@example.com",
        CreatedAt: time.Now(),
    }

    if err := PublishUserCreated(nc, event); err != nil {
        log.Fatalf("Error publishing event: %v", err)
    }

    log.Println("Event published successfully")
}
```

### **21.4.4 Circuit Breaker Pattern**

Implement circuit breakers to handle service failures gracefully:

```go
package main

import (
    "errors"
    "fmt"
    "time"

    "github.com/sony/gobreaker"
)

// UserServiceClient with circuit breaker
type UserServiceClient struct {
    baseURL string
    cb      *gobreaker.CircuitBreaker
}

// NewUserServiceClient creates a new client with circuit breaker
func NewUserServiceClient(baseURL string) *UserServiceClient {
    settings := gobreaker.Settings{
        Name:        "user-service",
        MaxRequests: 5,
        Interval:    10 * time.Second,
        Timeout:     30 * time.Second,
        ReadyToTrip: func(counts gobreaker.Counts) bool {
            failureRatio := float64(counts.TotalFailures) / float64(counts.Requests)
            return counts.Requests >= 10 && failureRatio >= 0.5
        },
        OnStateChange: func(name string, from gobreaker.State, to gobreaker.State) {
            fmt.Printf("Circuit breaker '%s' changed from '%s' to '%s'\n", name, from, to)
        },
    }

    return &UserServiceClient{
        baseURL: baseURL,
        cb:      gobreaker.NewCircuitBreaker(settings),
    }
}

// GetUser gets a user with circuit breaker protection
func (c *UserServiceClient) GetUser(id int) (*User, error) {
    result, err := c.cb.Execute(func() (interface{}, error) {
        // Call the actual service (simplified here)
        if id < 0 {
            return nil, errors.New("user not found")
        }

        return &User{
            ID:       id,
            Username: "testuser",
            Email:    "test@example.com",
        }, nil
    })

    if err != nil {
        return nil, err
    }

    return result.(*User), nil
}

// Example usage:
// client := NewUserServiceClient("http://user-service:8080")
// user, err := client.GetUser(123)
```

The circuit breaker pattern:

- Prevents cascading failures across services
- Allows failed services time to recover
- Provides fast failure responses instead of waiting for timeouts
- Automatically restores service when it becomes healthy

## **21.5 Data Management in Microservices**

### **21.5.1 Database per Service**

Each microservice should own its data and have its own database:

```
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│               │    │               │    │               │
│ User Service  │    │ Order Service │    │Product Service│
│               │    │               │    │               │
└───────┬───────┘    └───────┬───────┘    └───────┬───────┘
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│               │    │               │    │               │
│  User DB      │    │  Order DB     │    │  Product DB   │
│               │    │               │    │               │
└───────────────┘    └───────────────┘    └───────────────┘
```

Benefits of this approach:

- Independent scaling of databases
- Technology choice flexibility
- Isolation of failures
- Reduced contention
- Clear ownership boundaries

Implementation example with user service:

```go
package repository

import (
    "context"
    "fmt"
    "time"

    "github.com/jackc/pgx/v4/pgxpool"
)

// User represents a user entity
type User struct {
    ID        int       `json:"id"`
    Username  string    `json:"username"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

// UserRepository handles database operations for users
type UserRepository struct {
    pool *pgxpool.Pool
}

// NewUserRepository creates a new user repository
func NewUserRepository(connString string) (*UserRepository, error) {
    pool, err := pgxpool.Connect(context.Background(), connString)
    if err != nil {
        return nil, fmt.Errorf("failed to connect to database: %w", err)
    }

    return &UserRepository{pool: pool}, nil
}

// Get retrieves a user by ID
func (r *UserRepository) Get(ctx context.Context, id int) (*User, error) {
    query := `SELECT id, username, email, created_at FROM users WHERE id = $1`

    var user User
    err := r.pool.QueryRow(ctx, query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.CreatedAt,
    )
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }

    return &user, nil
}

// Create inserts a new user
func (r *UserRepository) Create(ctx context.Context, user User) (*User, error) {
    query := `
        INSERT INTO users (username, email, created_at)
        VALUES ($1, $2, $3)
        RETURNING id, created_at
    `

    err := r.pool.QueryRow(
        ctx,
        query,
        user.Username,
        user.Email,
        time.Now(),
    ).Scan(&user.ID, &user.CreatedAt)

    if err != nil {
        return nil, fmt.Errorf("failed to create user: %w", err)
    }

    return &user, nil
}

// Close closes the repository
func (r *UserRepository) Close() {
    r.pool.Close()
}
```

### **21.5.2 Handling Data Consistency**

In a microservices architecture, maintaining data consistency across services requires special patterns:

#### **Saga Pattern**

The Saga pattern manages distributed transactions across multiple services:

```
┌───────────┐      ┌───────────┐      ┌───────────┐
│           │      │           │      │           │
│  Service  │─────►│  Service  │─────►│  Service  │
│     A     │      │     B     │      │     C     │
│           │      │           │      │           │
└───────────┘      └───────────┘      └───────────┘
       ▲                  ▲                  ▲
       │                  │                  │
       ▼                  ▼                  ▼
┌───────────┐      ┌───────────┐      ┌───────────┐
│           │      │           │      │           │
│Compensate │◄─────│Compensate │◄─────│Compensate │
│     A     │      │     B     │      │     C     │
│           │      │           │      │           │
└───────────┘      └───────────┘      └───────────┘
```

Implementing a saga for order processing:

```go
package saga

import (
    "context"
    "errors"
    "log"
)

// Step represents a saga step
type Step struct {
    Execute    func(ctx context.Context) error
    Compensate func(ctx context.Context) error
}

// Saga coordinates a distributed transaction
type Saga struct {
    name  string
    steps []Step
}

// NewSaga creates a new saga
func NewSaga(name string) *Saga {
    return &Saga{
        name:  name,
        steps: []Step{},
    }
}

// AddStep adds a step to the saga
func (s *Saga) AddStep(execute, compensate func(ctx context.Context) error) {
    s.steps = append(s.steps, Step{
        Execute:    execute,
        Compensate: compensate,
    })
}

// Execute executes the saga
func (s *Saga) Execute(ctx context.Context) error {
    log.Printf("Starting saga: %s", s.name)

    executedSteps := 0

    // Execute each step
    for i, step := range s.steps {
        if err := step.Execute(ctx); err != nil {
            log.Printf("Step %d failed: %v", i, err)

            // Compensate for executed steps in reverse order
            for j := executedSteps - 1; j >= 0; j-- {
                if err := s.steps[j].Compensate(ctx); err != nil {
                    log.Printf("Compensation for step %d failed: %v", j, err)
                }
            }

            return errors.New("saga failed")
        }
        executedSteps++
    }

    log.Printf("Saga completed successfully: %s", s.name)
    return nil
}
```

Usage example for order processing:

```go
// Create a new saga for order processing
orderSaga := saga.NewSaga("create-order")

// Add steps with compensation
orderSaga.AddStep(
    // Step 1: Create order
    func(ctx context.Context) error {
        return orderService.Create(ctx, order)
    },
    // Compensation for step 1
    func(ctx context.Context) error {
        return orderService.Cancel(ctx, order.ID)
    },
)

orderSaga.AddStep(
    // Step 2: Reserve inventory
    func(ctx context.Context) error {
        return inventoryService.Reserve(ctx, order.ProductID, order.Quantity)
    },
    // Compensation for step 2
    func(ctx context.Context) error {
        return inventoryService.Release(ctx, order.ProductID, order.Quantity)
    },
)

orderSaga.AddStep(
    // Step 3: Process payment
    func(ctx context.Context) error {
        return paymentService.Process(ctx, order.ID, order.Amount)
    },
    // Compensation for step 3
    func(ctx context.Context) error {
        return paymentService.Refund(ctx, order.ID)
    },
)

// Execute the saga
if err := orderSaga.Execute(ctx); err != nil {
    log.Printf("Order processing failed: %v", err)
    return err
}
```

#### **Event Sourcing**

Event sourcing stores all changes to application state as a sequence of events:

```go
package eventsourcing

import (
    "context"
    "fmt"
    "time"
)

// Event represents a domain event
type Event struct {
    ID        string    `json:"id"`
    Type      string    `json:"type"`
    Data      []byte    `json:"data"`
    Timestamp time.Time `json:"timestamp"`
    AggregateID string  `json:"aggregate_id"`
}

// EventStore stores and retrieves events
type EventStore interface {
    SaveEvents(ctx context.Context, events []Event) error
    GetEvents(ctx context.Context, aggregateID string) ([]Event, error)
}

// PostgresEventStore implements EventStore with PostgreSQL
type PostgresEventStore struct {
    // Implementation details omitted
}

// Aggregate is the base for event-sourced entities
type Aggregate struct {
    ID      string
    Version int
    Events  []Event
}

// ApplyEvent applies an event to the aggregate
func (a *Aggregate) ApplyEvent(eventType string, data []byte) {
    event := Event{
        ID:          generateID(),
        Type:        eventType,
        Data:        data,
        Timestamp:   time.Now(),
        AggregateID: a.ID,
    }

    a.Events = append(a.Events, event)
    a.Version++
}

// Save saves all uncommitted events
func (a *Aggregate) Save(ctx context.Context, store EventStore) error {
    if len(a.Events) == 0 {
        return nil
    }

    if err := store.SaveEvents(ctx, a.Events); err != nil {
        return fmt.Errorf("failed to save events: %w", err)
    }

    a.Events = nil
    return nil
}

// Load loads events for an aggregate
func (a *Aggregate) Load(ctx context.Context, store EventStore) error {
    events, err := store.GetEvents(ctx, a.ID)
    if err != nil {
        return fmt.Errorf("failed to load events: %w", err)
    }

    for _, event := range events {
        // Apply event to rebuild state
        a.Version++
    }

    return nil
}

// generateID generates a unique ID
func generateID() string {
    // Implementation omitted
    return "unique-id"
}
```

## **21.6 Deployment and Orchestration**

### **21.6.1 Containerization with Docker**

Containerize microservices with Docker:

```dockerfile
# Build stage
FROM golang:1.18-alpine AS build

WORKDIR /app

# Copy and download dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -o service ./cmd/service

# Final stage
FROM alpine:3.15

# Add non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy binary from build stage
COPY --from=build /app/service .
COPY --from=build /app/configs ./configs

# Set ownership
RUN chown -R appuser:appgroup /app

# Use non-root user
USER appuser

# Expose port
EXPOSE 8080

# Run the application
CMD ["./service"]
```

Multi-service Docker Compose setup:

```yaml
# docker-compose.yml
version: "3.8"

services:
  user-service:
    build:
      context: ./user-service
      dockerfile: Dockerfile
    ports:
      - "8081:8080"
    environment:
      - DB_HOST=user-db
      - DB_USER=postgres
      - DB_PASSWORD=password
      - DB_NAME=users
    depends_on:
      - user-db
    restart: on-failure

  order-service:
    build:
      context: ./order-service
      dockerfile: Dockerfile
    ports:
      - "8082:8080"
    environment:
      - DB_HOST=order-db
      - DB_USER=postgres
      - DB_PASSWORD=password
      - DB_NAME=orders
      - USER_SERVICE_URL=http://user-service:8080
    depends_on:
      - order-db
      - user-service
    restart: on-failure

  product-service:
    build:
      context: ./product-service
      dockerfile: Dockerfile
    ports:
      - "8083:8080"
    environment:
      - DB_HOST=product-db
      - DB_USER=postgres
      - DB_PASSWORD=password
      - DB_NAME=products
    depends_on:
      - product-db
    restart: on-failure

  api-gateway:
    build:
      context: ./api-gateway
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - USER_SERVICE_URL=http://user-service:8080
      - ORDER_SERVICE_URL=http://order-service:8080
      - PRODUCT_SERVICE_URL=http://product-service:8080
    depends_on:
      - user-service
      - order-service
      - product-service
    restart: on-failure

  user-db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=users
    volumes:
      - user-db-data:/var/lib/postgresql/data

  order-db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=orders
    volumes:
      - order-db-data:/var/lib/postgresql/data

  product-db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=products
    volumes:
      - product-db-data:/var/lib/postgresql/data

  nats:
    image: nats:2.7.4-alpine
    ports:
      - "4222:4222"

volumes:
  user-db-data:
  order-db-data:
  product-db-data:
```

### **21.6.2 Orchestration with Kubernetes**

Deploy microservices to Kubernetes:

```yaml
# user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: yourregistry/user-service:latest
          ports:
            - containerPort: 8080
          env:
            - name: DB_HOST
              value: user-db
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: password
            - name: DB_NAME
              value: users
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "256Mi"

---
# user-service-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

Kubernetes configuration for API Gateway:

```yaml
# api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  labels:
    app: api-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
        - name: api-gateway
          image: yourregistry/api-gateway:latest
          ports:
            - containerPort: 8080
          env:
            - name: USER_SERVICE_URL
              value: http://user-service
            - name: ORDER_SERVICE_URL
              value: http://order-service
            - name: PRODUCT_SERVICE_URL
              value: http://product-service
          resources:
            limits:
              cpu: "500m"
              memory: "512Mi"
            requests:
              cpu: "100m"
              memory: "256Mi"

---
# api-gateway-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api-gateway
spec:
  selector:
    app: api-gateway
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

## **21.7 Observability**

### **21.7.1 Distributed Tracing**

Implement distributed tracing with OpenTelemetry:

```go
package main

import (
    "context"
    "log"
    "net/http"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/resource"
    "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.7.0"
)

// initTracer initializes a Jaeger tracer
func initTracer(serviceName string) (*trace.TracerProvider, error) {
    // Configure Jaeger exporter
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://jaeger:14268/api/traces"),
    ))
    if err != nil {
        return nil, err
    }

    // Create resource with service information
    res := resource.NewWithAttributes(
        semconv.SchemaURL,
        semconv.ServiceNameKey.String(serviceName),
        attribute.String("environment", "production"),
    )

    // Create trace provider
    tp := trace.NewTracerProvider(
        trace.WithBatcher(exporter),
        trace.WithResource(res),
    )

    // Register as global trace provider
    otel.SetTracerProvider(tp)

    return tp, nil
}

func main() {
    // Initialize tracer
    tp, err := initTracer("user-service")
    if err != nil {
        log.Fatalf("Failed to initialize tracer: %v", err)
    }
    defer tp.Shutdown(context.Background())

    // Create a tracer
    tracer := otel.Tracer("user-service")

    // HTTP handler with tracing
    http.HandleFunc("/users", func(w http.ResponseWriter, r *http.Request) {
        // Create a span for this handler
        ctx, span := tracer.Start(r.Context(), "get_users")
        defer span.End()

        // Add attributes to the span
        span.SetAttributes(
            attribute.String("http.method", r.Method),
            attribute.String("http.path", r.URL.Path),
        )

        // Call service method with context containing span
        users, err := getUsersWithTracing(ctx)
        if err != nil {
            span.RecordError(err)
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }

        // Respond with data
        // ...
    })

    // Start server
    log.Fatal(http.ListenAndServe(":8080", nil))
}

// getUsersWithTracing demonstrates propagating context with tracing
func getUsersWithTracing(ctx context.Context) ([]User, error) {
    tracer := otel.Tracer("user-service")

    // Create a child span
    ctx, span := tracer.Start(ctx, "fetch_users_from_db")
    defer span.End()

    // Perform database operation
    // ...

    // Record span events
    span.AddEvent("retrieved users from database",
        attribute.Int("count", 10),
    )

    return []User{}, nil
}
```

### **21.7.2 Metrics Collection**

Implement metrics with Prometheus:

```go
package main

import (
    "log"
    "net/http"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    // Counter metrics
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )

    // Gauge metrics
    activeSessions = promauto.NewGauge(
        prometheus.GaugeOpts{
            Name: "active_sessions",
            Help: "Number of active user sessions",
        },
    )

    // Histogram metrics
    requestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)

// metricsMiddleware collects HTTP metrics
func metricsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Create a response wrapper to capture status code
        rw := newResponseWriter(w)

        // Start timer
        timer := prometheus.NewTimer(requestDuration.WithLabelValues(r.Method, r.URL.Path))

        // Call the next handler
        next.ServeHTTP(rw, r)

        // Stop timer
        timer.ObserveDuration()

        // Increment request counter
        httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, rw.Status()).Inc()
    })
}

// responseWriter wraps http.ResponseWriter to capture status code
type responseWriter struct {
    http.ResponseWriter
    statusCode int
}

func newResponseWriter(w http.ResponseWriter) *responseWriter {
    return &responseWriter{w, http.StatusOK}
}

func (rw *responseWriter) WriteHeader(code int) {
    rw.statusCode = code
    rw.ResponseWriter.WriteHeader(code)
}

func (rw *responseWriter) Status() string {
    return http.StatusText(rw.statusCode)
}

func main() {
    // Create router
    mux := http.NewServeMux()

    // Add endpoints
    mux.Handle("/users", metricsMiddleware(http.HandlerFunc(usersHandler)))

    // Expose Prometheus metrics endpoint
    mux.Handle("/metrics", promhttp.Handler())

    // Start server
    log.Fatal(http.ListenAndServe(":8080", mux))
}

func usersHandler(w http.ResponseWriter, r *http.Request) {
    // Implementation omitted
}
```

### **21.7.3 Centralized Logging**

Implement structured logging with correlation IDs:

```go
package logger

import (
    "context"

    "github.com/google/uuid"
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

// contextKey is a type for context keys
type contextKey string

// Define context key for correlation ID
const correlationIDKey contextKey = "correlation_id"

// Logger wraps zap logger with correlation ID support
type Logger struct {
    logger *zap.Logger
}

// NewLogger creates a new logger
func NewLogger() (*Logger, error) {
    config := zap.NewProductionConfig()
    config.EncoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder

    logger, err := config.Build()
    if err != nil {
        return nil, err
    }

    return &Logger{logger: logger}, nil
}

// WithContext returns a logger with correlation ID from context
func (l *Logger) WithContext(ctx context.Context) *zap.Logger {
    correlationID := ctx.Value(correlationIDKey)
    if correlationID == nil {
        return l.logger
    }

    return l.logger.With(zap.String("correlation_id", correlationID.(string)))
}

// NewContext adds a correlation ID to context
func NewContext(ctx context.Context) context.Context {
    correlationID := uuid.New().String()
    return context.WithValue(ctx, correlationIDKey, correlationID)
}

// GetCorrelationID gets correlation ID from context
func GetCorrelationID(ctx context.Context) string {
    correlationID := ctx.Value(correlationIDKey)
    if correlationID == nil {
        return ""
    }
    return correlationID.(string)
}

// Example usage:
// logger, _ := logger.NewLogger()
// ctx := logger.NewContext(context.Background())
// logger.WithContext(ctx).Info("Processing request", zap.String("user_id", "123"))
```

## **21.8 Testing Microservices**

### **21.8.1 Unit Testing**

Test individual components in isolation:

```go
package service_test

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"

    "github.com/yourorg/userservice/internal/domain"
    "github.com/yourorg/userservice/internal/repository"
    "github.com/yourorg/userservice/internal/service"
)

// MockUserRepository is a mock implementation of repository.UserRepository
type MockUserRepository struct {
    mock.Mock
}

func (m *MockUserRepository) Get(ctx context.Context, id int) (*domain.User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*domain.User), args.Error(1)
}

func (m *MockUserRepository) Create(ctx context.Context, user domain.User) (*domain.User, error) {
    args := m.Called(ctx, user)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*domain.User), args.Error(1)
}

// TestUserService_GetUser tests the GetUser method
func TestUserService_GetUser(t *testing.T) {
    // Create mock repository
    mockRepo := new(MockUserRepository)

    // Create service with mock repository
    userService := service.NewUserService(mockRepo)

    // Create test data
    expectedUser := &domain.User{
        ID:       123,
        Username: "testuser",
        Email:    "test@example.com",
    }

    // Set up expectations
    mockRepo.On("Get", mock.Anything, 123).Return(expectedUser, nil)

    // Call the service
    user, err := userService.GetUser(context.Background(), 123)

    // Assert results
    assert.NoError(t, err)
    assert.Equal(t, expectedUser.ID, user.ID)
    assert.Equal(t, expectedUser.Username, user.Username)
    assert.Equal(t, expectedUser.Email, user.Email)

    // Verify expectations
    mockRepo.AssertExpectations(t)
}
```

### **21.8.2 Integration Testing**

Test interactions between services and external dependencies:

```go
package integration_test

import (
    "context"
    "fmt"
    "os"
    "testing"

    "github.com/jackc/pgx/v4/pgxpool"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/wait"

    "github.com/yourorg/userservice/internal/domain"
    "github.com/yourorg/userservice/internal/repository"
)

func setupTestDatabase(t *testing.T) (*pgxpool.Pool, testcontainers.Container, error) {
    // Create a PostgreSQL container
    req := testcontainers.ContainerRequest{
        Image:        "postgres:14-alpine",
        ExposedPorts: []string{"5432/tcp"},
        Env: map[string]string{
            "POSTGRES_USER":     "testuser",
            "POSTGRES_PASSWORD": "testpass",
            "POSTGRES_DB":       "testdb",
        },
        WaitingFor: wait.ForLog("database system is ready to accept connections"),
    }

    container, err := testcontainers.GenericContainer(context.Background(), testcontainers.GenericContainerRequest{
        ContainerRequest: req,
        Started:          true,
    })
    if err != nil {
        return nil, nil, err
    }

    // Get host and port
    host, err := container.Host(context.Background())
    if err != nil {
        return nil, nil, err
    }

    port, err := container.MappedPort(context.Background(), "5432")
    if err != nil {
        return nil, nil, err
    }

    // Create connection string
    connString := fmt.Sprintf("postgres://testuser:testpass@%s:%s/testdb", host, port.Port())

    // Connect to database
    pool, err := pgxpool.Connect(context.Background(), connString)
    if err != nil {
        return nil, nil, err
    }

    // Create users table
    _, err = pool.Exec(context.Background(), `
        CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            username TEXT NOT NULL,
            email TEXT NOT NULL,
            created_at TIMESTAMP NOT NULL DEFAULT NOW()
        )
    `)
    if err != nil {
        return nil, nil, err
    }

    return pool, container, nil
}

func TestUserRepository_Integration(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping integration test in short mode")
    }

    // Set up test database
    pool, container, err := setupTestDatabase(t)
    require.NoError(t, err)

    // Clean up after test
    defer func() {
        pool.Close()
        container.Terminate(context.Background())
    }()

    // Create repository
    repo := repository.NewUserRepository(pool)

    // Test creating a user
    user := domain.User{
        Username: "testuser",
        Email:    "test@example.com",
    }

    createdUser, err := repo.Create(context.Background(), user)
    assert.NoError(t, err)
    assert.NotZero(t, createdUser.ID)
    assert.Equal(t, user.Username, createdUser.Username)
    assert.Equal(t, user.Email, createdUser.Email)

    // Test getting a user
    fetchedUser, err := repo.Get(context.Background(), createdUser.ID)
    assert.NoError(t, err)
    assert.Equal(t, createdUser.ID, fetchedUser.ID)
    assert.Equal(t, createdUser.Username, fetchedUser.Username)
    assert.Equal(t, createdUser.Email, fetchedUser.Email)
}
```

### **21.8.3 Consumer-Driven Contract Testing**

Test service API contracts with Pact:

```go
package pact_test

import (
    "fmt"
    "testing"

    "github.com/pact-foundation/pact-go/dsl"
    "github.com/pact-foundation/pact-go/types"
    "github.com/stretchr/testify/assert"

    "github.com/yourorg/orderservice/client"
)

func TestUserServiceClient_Pact(t *testing.T) {
    // Create Pact
    pact := dsl.Pact{
        Consumer: "order-service",
        Provider: "user-service",
    }

    // Set up expectations
    pact.Setup(true)

    // Define the interaction
    pact.AddInteraction().
        Given("User with ID 123 exists").
        UponReceiving("A request for user 123").
        WithRequest(dsl.Request{
            Method: "GET",
            Path:   dsl.String("/users/123"),
            Headers: dsl.MapMatcher{
                "Accept": dsl.String("application/json"),
            },
        }).
        WillRespondWith(dsl.Response{
            Status: 200,
            Headers: dsl.MapMatcher{
                "Content-Type": dsl.String("application/json"),
            },
            Body: dsl.Match(client.User{
                ID:       123,
                Username: "testuser",
                Email:    "test@example.com",
            }),
        })

    // Run the test
    err := pact.Verify(func() error {
        // Configure API client with Pact mock server URL
        userClient := client.NewUserClient(fmt.Sprintf("http://localhost:%d", pact.Server.Port))

        // Execute the API call
        user, err := userClient.GetUser(123)
        if err != nil {
            return err
        }

        // Verify response
        assert.Equal(t, 123, user.ID)
        assert.Equal(t, "testuser", user.Username)
        assert.Equal(t, "test@example.com", user.Email)

        return nil
    })

    assert.NoError(t, err)

    // Write the contract
    pact.WritePact()
}
```

### **21.8.4 End-to-End Testing**

Test the entire system using Docker Compose:

```go
package e2e_test

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
    "os"
    "testing"
    "time"

    "github.com/stretchr/testify/assert"
)

func TestE2E_CreateOrder(t *testing.T) {
    if testing.Short() {
        t.Skip("Skipping e2e test in short mode")
    }

    // Get API gateway URL from environment variable
    apiURL := os.Getenv("API_GATEWAY_URL")
    if apiURL == "" {
        apiURL = "http://localhost:8080" // Default for local testing
    }

    // Step 1: Create a user
    userPayload := map[string]interface{}{
        "username": "e2euser",
        "email":    "e2e@example.com",
    }

    userJSON, _ := json.Marshal(userPayload)
    resp, err := http.Post(fmt.Sprintf("%s/users", apiURL), "application/json", bytes.NewBuffer(userJSON))
    assert.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var createdUser map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&createdUser)
    resp.Body.Close()

    userID := int(createdUser["id"].(float64))

    // Step 2: Create a product
    productPayload := map[string]interface{}{
        "name":     "Test Product",
        "price":    99.99,
        "quantity": 100,
    }

    productJSON, _ := json.Marshal(productPayload)
    resp, err = http.Post(fmt.Sprintf("%s/products", apiURL), "application/json", bytes.NewBuffer(productJSON))
    assert.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var createdProduct map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&createdProduct)
    resp.Body.Close()

    productID := int(createdProduct["id"].(float64))

    // Step 3: Create an order
    orderPayload := map[string]interface{}{
        "user_id": userID,
        "items": []map[string]interface{}{
            {
                "product_id": productID,
                "quantity":   2,
                "price":      99.99,
            },
        },
    }

    orderJSON, _ := json.Marshal(orderPayload)
    resp, err = http.Post(fmt.Sprintf("%s/orders", apiURL), "application/json", bytes.NewBuffer(orderJSON))
    assert.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)

    var createdOrder map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&createdOrder)
    resp.Body.Close()

    orderID := int(createdOrder["id"].(float64))

    // Step 4: Verify order status
    // Wait for async processing
    time.Sleep(2 * time.Second)

    resp, err = http.Get(fmt.Sprintf("%s/orders/%d", apiURL, orderID))
    assert.NoError(t, err)
    assert.Equal(t, http.StatusOK, resp.StatusCode)

    var order map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&order)
    resp.Body.Close()

    assert.Equal(t, "completed", order["status"])
    assert.Equal(t, userID, int(order["user_id"].(float64)))
}
```

## **21.9 Resilience Patterns**

### **21.9.1 Retry Pattern**

Implement retries for transient failures:

```go
package retry

import (
    "context"
    "errors"
    "math"
    "time"
)

// Options holds retry configuration
type Options struct {
    MaxRetries  int
    InitialWait time.Duration
    MaxWait     time.Duration
    Multiplier  float64
    Jitter      float64
}

// DefaultOptions returns default retry options
func DefaultOptions() Options {
    return Options{
        MaxRetries:  3,
        InitialWait: 100 * time.Millisecond,
        MaxWait:     10 * time.Second,
        Multiplier:  2.0,
        Jitter:      0.1,
    }
}

// Do executes the function with retries
func Do(ctx context.Context, options Options, fn func() error) error {
    var err error

    for attempt := 0; attempt <= options.MaxRetries; attempt++ {
        // Execute the function
        err = fn()
        if err == nil {
            return nil
        }

        // If this was the last attempt, return the error
        if attempt == options.MaxRetries {
            return err
        }

        // Calculate wait time with exponential backoff and jitter
        waitTime := calculateWaitTime(attempt, options)

        // Create a timer that will fire after waitTime
        timer := time.NewTimer(waitTime)

        // Wait for either the timer to fire or context to be canceled
        select {
        case <-timer.C:
            // Timer fired, continue with next attempt
        case <-ctx.Done():
            // Context was canceled
            timer.Stop()
            return ctx.Err()
        }
    }

    return err
}

// calculateWaitTime calculates wait time with exponential backoff and jitter
func calculateWaitTime(attempt int, options Options) time.Duration {
    // Calculate exponential backoff
    waitTime := float64(options.InitialWait) * math.Pow(options.Multiplier, float64(attempt))

    // Apply jitter
    jitter := (options.Jitter * waitTime) * (2*rand.Float64() - 1)
    waitTime = waitTime + jitter

    // Cap at max wait time
    if waitTime > float64(options.MaxWait) {
        waitTime = float64(options.MaxWait)
    }

    return time.Duration(waitTime)
}

// Usage example:
// err := retry.Do(ctx, retry.DefaultOptions(), func() error {
//     return userService.GetUser(123)
// })
```

### **21.9.2 Bulkhead Pattern**

Isolate components to prevent cascading failures:

```go
package bulkhead

import (
    "context"
    "errors"
    "sync"
)

// Bulkhead implements the bulkhead pattern
type Bulkhead struct {
    maxConcurrent int
    currentCount  int
    mutex         sync.Mutex
}

// New creates a new bulkhead
func New(maxConcurrent int) *Bulkhead {
    return &Bulkhead{
        maxConcurrent: maxConcurrent,
        currentCount:  0,
    }
}

// Execute executes a function within the bulkhead
func (b *Bulkhead) Execute(ctx context.Context, fn func() error) error {
    // Acquire permission to execute
    if !b.acquire() {
        return errors.New("bulkhead full")
    }

    // Release permission when done
    defer b.release()

    // Execute the function
    return fn()
}

// acquire acquires permission to execute
func (b *Bulkhead) acquire() bool {
    b.mutex.Lock()
    defer b.mutex.Unlock()

    if b.currentCount >= b.maxConcurrent {
        return false
    }

    b.currentCount++
    return true
}

// release releases permission
func (b *Bulkhead) release() {
    b.mutex.Lock()
    defer b.mutex.Unlock()

    b.currentCount--
    if b.currentCount < 0 {
        b.currentCount = 0
    }
}

// Usage example:
// bulkhead := bulkhead.New(10)
// err := bulkhead.Execute(ctx, func() error {
//     return userService.GetUser(123)
// })
```

### **21.9.3 Rate Limiting**

Implement rate limiting to protect services:

```go
package ratelimit

import (
    "context"
    "errors"
    "sync"
    "time"
)

// RateLimiter implements the token bucket algorithm
type RateLimiter struct {
    rate       float64    // tokens per second
    capacity   float64    // maximum tokens
    tokens     float64    // current tokens
    lastRefill time.Time  // last time tokens were refilled
    mutex      sync.Mutex // mutex for thread safety
}

// New creates a new rate limiter
func New(rate, capacity float64) *RateLimiter {
    return &RateLimiter{
        rate:       rate,
        capacity:   capacity,
        tokens:     capacity,
        lastRefill: time.Now(),
    }
}

// Allow checks if a request is allowed
func (r *RateLimiter) Allow() bool {
    r.mutex.Lock()
    defer r.mutex.Unlock()

    // Refill tokens based on time elapsed
    now := time.Now()
    elapsed := now.Sub(r.lastRefill).Seconds()
    r.tokens += elapsed * r.rate

    // Cap tokens at capacity
    if r.tokens > r.capacity {
        r.tokens = r.capacity
    }

    // Update last refill time
    r.lastRefill = now

    // Check if we have enough tokens
    if r.tokens < 1.0 {
        return false
    }

    // Consume a token
    r.tokens -= 1.0
    return true
}

// Execute executes a function with rate limiting
func (r *RateLimiter) Execute(ctx context.Context, fn func() error) error {
    if !r.Allow() {
        return errors.New("rate limit exceeded")
    }

    return fn()
}

// Usage example:
// limiter := ratelimit.New(10, 50) // 10 requests per second, burst of 50
// err := limiter.Execute(ctx, func() error {
//     return userService.GetUser(123)
// })
```

## **21.10 Microservices Best Practices**

1. **Design Services Around Business Domains**

   - Use Domain-Driven Design (DDD) principles
   - Organize teams around services (Conway's Law)

2. **Implement Proper Service Boundaries**

   - Each service should own its data
   - Minimize cross-service dependencies
   - Establish clear contracts between services

3. **Use Asynchronous Communication When Possible**

   - Reduce coupling between services
   - Improve resilience to service failures
   - Enable better scalability

4. **Implement Comprehensive Observability**

   - Distributed tracing
   - Centralized logging
   - Metrics and monitoring
   - Health checks and service discovery

5. **Design for Failure**

   - Implement circuit breakers
   - Use retries with backoff
   - Implement fallbacks and graceful degradation
   - Test failure scenarios

6. **Automate Deployment and Scaling**

   - Use containerization
   - Implement CI/CD pipelines
   - Adopt Kubernetes for orchestration
   - Implement infrastructure as code

7. **Secure Your Microservices**

   - Use API gateways for authentication/authorization
   - Implement service-to-service authentication
   - Encrypt data in transit and at rest
   - Regularly audit and update dependencies

8. **Standardize Development Practices**

   - Consistent logging formats
   - Common error handling
   - Standardized API design
   - Shared libraries for common functionality

9. **Version Your APIs**

   - Use semantic versioning
   - Support backward compatibility
   - Implement graceful API deprecation

10. **Start Small, Evolve Gradually**
    - Begin with a monolith or a few services
    - Split services as boundaries become clear
    - Use the strangler pattern for migration

## **21.11 Exercises**

### **Exercise 1: Create a Basic Microservice**

Build a simple user service that provides CRUD operations via a RESTful API. Implement proper error handling, configuration management, and health checks.

### **Exercise 2: Service Communication**

Extend your user service to communicate with a new order service. Implement both synchronous (REST or gRPC) and asynchronous (messaging) communication between the services.

### **Exercise 3: Resilience Patterns**

Add resilience patterns to your services:

- Circuit breaker for service-to-service calls
- Retry mechanism for transient failures
- Rate limiting for API endpoints

### **Exercise 4: Observability**

Implement observability in your microservices:

- Distributed tracing with OpenTelemetry
- Metrics collection with Prometheus
- Structured logging with correlation IDs

### **Exercise 5: Containerization and Orchestration**

Containerize your microservices using Docker and deploy them using Docker Compose. For an extra challenge, deploy them to a Kubernetes cluster.

## **21.12 Summary**

In this chapter, we've explored the fundamentals of building microservices in Go:

- **Microservices Architecture**: We learned about the characteristics, benefits, and challenges of microservices
- **Service Design**: We covered strategies for decomposing services and defining boundaries
- **Communication Patterns**: We implemented REST, gRPC, and event-driven communication
- **Data Management**: We explored database per service, sagas, and event sourcing
- **Resilience**: We implemented circuit breakers, retries, and bulkheads
- **Observability**: We added distributed tracing, metrics, and centralized logging
- **Deployment**: We containerized services and orchestrated them with Kubernetes
- **Testing**: We covered unit, integration, contract, and end-to-end testing

Go's simplicity, performance, and concurrency model make it an excellent choice for microservices. By following the patterns and practices outlined in this chapter, you can build robust, scalable, and maintainable microservices architectures.

Remember that microservices are not a silver bullet. Start with a well-designed monolith and migrate to microservices gradually as your system and team grow. Focus on business domains and clear service boundaries to maximize the benefits while minimizing the complexity.

**Next Up**: In the next chapter, we'll explore serverless computing with Go, building on the concepts we've learned about microservices to create even more scalable and cost-effective solutions.
