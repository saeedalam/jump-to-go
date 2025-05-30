# **Chapter 23: Essential Design Patterns in Go**

## **23.1. Introduction to Design Patterns in Go**

Design patterns are proven solutions to common problems in software design. In Go, these patterns take on a unique flavor due to the language's distinctive features: strong typing, implicit interfaces, composition over inheritance, and powerful concurrency primitives.

This chapter provides a comprehensive, practical guide to implementing design patterns in idiomatic Go. We'll explore:

- How classic design patterns adapt to Go's philosophy
- Go-specific patterns that leverage the language's unique features
- Practical, real-world examples with working code
- When to use (and when to avoid) each pattern

### **Why Design Patterns Matter in Go**

While experienced Go developers often emphasize simplicity over patterns, understanding design patterns offers several benefits:

1. **Shared vocabulary**: Patterns provide a common language to discuss architectural solutions.
2. **Proven solutions**: They offer time-tested approaches to recurring problems.
3. **Code organization**: Patterns help structure complex applications.
4. **Maintainability**: Well-applied patterns make code easier to understand and extend.

### **Go's Approach to Design Patterns**

Go differs from traditional object-oriented languages in ways that affect pattern implementation:

1. **Interface-based polymorphism**: Go uses implicit interfaces rather than explicit inheritance.
2. **Composition over inheritance**: Go encourages embedding rather than extending.
3. **First-class functions**: Functions can be passed as values, simplifying certain patterns.
4. **Concurrency primitives**: Goroutines and channels provide alternatives to thread-based patterns.

### **Adapting Classic Patterns to Go**

When implementing design patterns in Go, consider these principles:

1. **Keep it simple**: Go values simplicity; avoid over-engineering.
2. **Use small interfaces**: Define focused interfaces with just the methods you need.
3. **Embrace composition**: Use struct embedding to compose behavior.
4. **Consider Go idioms**: Some problems solved by patterns in other languages have simpler solutions in Go.

With these principles in mind, let's explore both classic and Go-specific design patterns.

## **23.2. Creational Patterns**

Creational patterns abstract the instantiation process, making systems more independent of how their objects are created and composed.

### **1. Singleton Pattern**

**Purpose**: Ensure a class has only one instance and provide a global point of access to it.

**Go Implementation**: In Go, singletons are typically implemented using package variables or sync.Once.

**Example: Thread-Safe Database Connection**

```go
package database

import (
	"database/sql"
	"fmt"
	"sync"

	_ "github.com/lib/pq"
)

var (
	instance *sql.DB
	once     sync.Once
	connErr  error
)

// GetConnection returns a thread-safe singleton database connection
func GetConnection(connString string) (*sql.DB, error) {
	once.Do(func() {
		instance, connErr = sql.Open("postgres", connString)
		if connErr == nil {
			// Configure connection pool
			instance.SetMaxOpenConns(25)
			instance.SetMaxIdleConns(5)
			connErr = instance.Ping() // Verify connection
			if connErr != nil {
				fmt.Printf("Database connection failed: %v\n", connErr)
			}
		}
	})
	return instance, connErr
}
```

**Usage**:

```go
func main() {
	connStr := "user=postgres dbname=myapp sslmode=disable"
	db, err := database.GetConnection(connStr)
	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}

	// Use the same connection throughout your application
	// Every call to GetConnection returns the same instance
}
```

**When to Use in Go**:

- Managing database connections
- Creating shared resource handlers
- Implementing logger instances

**Go-Specific Considerations**:

- Consider using a package-level variable for simple singletons
- For testability, consider dependency injection instead of true singletons
- Use init() with caution, as it can make testing difficult

### **2. Factory Method Pattern**

**Purpose**: Define an interface for creating an object, but let subclasses decide which class to instantiate.

**Go Implementation**: In Go, factory methods are typically functions that return interface types.

**Example: Payment Processor Factory**

```go
package payment

import "fmt"

// PaymentProcessor defines the interface for processing payments
type PaymentProcessor interface {
	Process(amount float64) error
	Currency() string
}

// CreditCardProcessor handles credit card payments
type CreditCardProcessor struct {
	cardNumber string
}

func (c *CreditCardProcessor) Process(amount float64) error {
	fmt.Printf("Processing $%.2f via credit card %s\n", amount, c.cardNumber)
		return nil
	}

func (c *CreditCardProcessor) Currency() string {
	return "USD"
}

// PayPalProcessor handles PayPal payments
type PayPalProcessor struct {
	email string
}

func (p *PayPalProcessor) Process(amount float64) error {
	fmt.Printf("Processing $%.2f via PayPal account %s\n", amount, p.email)
	return nil
}

func (p *PayPalProcessor) Currency() string {
	return "USD"
}

// CryptoProcessor handles cryptocurrency payments
type CryptoProcessor struct {
	walletID string
}

func (c *CryptoProcessor) Process(amount float64) error {
	fmt.Printf("Processing $%.2f worth of crypto to wallet %s\n", amount, c.walletID)
	return nil
}

func (c *CryptoProcessor) Currency() string {
	return "BTC"
}

// CreatePaymentProcessor is the factory method
func CreatePaymentProcessor(method string, details string) (PaymentProcessor, error) {
	switch method {
	case "credit_card":
		return &CreditCardProcessor{cardNumber: details}, nil
	case "paypal":
		return &PayPalProcessor{email: details}, nil
	case "crypto":
		return &CryptoProcessor{walletID: details}, nil
	default:
		return nil, fmt.Errorf("unsupported payment method: %s", method)
	}
}
```

**Usage**:

```go
func main() {
	// Create a credit card processor
	processor, err := payment.CreatePaymentProcessor("credit_card", "4111-1111-1111-1111")
	if err != nil {
		log.Fatal(err)
	}

	// Process payment
	err = processor.Process(99.99)
	if err != nil {
		log.Fatal(err)
	}

	// Create a different type of processor
	cryptoProcessor, _ := payment.CreatePaymentProcessor("crypto", "0x1234567890abcdef")
	cryptoProcessor.Process(150.00)
}
```

**When to Use in Go**:

- Creating objects based on runtime conditions
- Implementing plugin systems
- Working with multiple implementations of an interface

**Go-Specific Considerations**:

- Return interfaces, not concrete types, to maintain flexibility
- Consider providing multiple factory functions for clarity
- For simple cases, constructors (NewXxx functions) may be sufficient

### **3. Builder Pattern**

**Purpose**: Separate the construction of a complex object from its representation.

**Go Implementation**: In Go, the builder pattern often uses method chaining with pointer receivers.

**Example: HTTP Request Builder**

```go
package request

import (
	"bytes"
	"encoding/json"
	"io"
	"net/http"
	"time"
)

// RequestBuilder helps construct HTTP requests
type RequestBuilder struct {
	method      string
	url         string
	headers     map[string]string
	queryParams map[string]string
	body        io.Reader
	timeout     time.Duration
	client      *http.Client
}

// NewRequestBuilder creates a new RequestBuilder
func NewRequestBuilder() *RequestBuilder {
	return &RequestBuilder{
		headers:     make(map[string]string),
		queryParams: make(map[string]string),
		client:      &http.Client{},
	}
}

// Method sets the HTTP method
func (b *RequestBuilder) Method(method string) *RequestBuilder {
	b.method = method
	return b
}

// URL sets the request URL
func (b *RequestBuilder) URL(url string) *RequestBuilder {
	b.url = url
	return b
}

// Header adds a header to the request
func (b *RequestBuilder) Header(key, value string) *RequestBuilder {
	b.headers[key] = value
	return b
}

// QueryParam adds a query parameter
func (b *RequestBuilder) QueryParam(key, value string) *RequestBuilder {
	b.queryParams[key] = value
	return b
}

// JSONBody sets the request body as JSON
func (b *RequestBuilder) JSONBody(data interface{}) *RequestBuilder {
	jsonData, _ := json.Marshal(data)
	b.body = bytes.NewBuffer(jsonData)
	b.Header("Content-Type", "application/json")
	return b
}

// Timeout sets the request timeout
func (b *RequestBuilder) Timeout(d time.Duration) *RequestBuilder {
	b.timeout = d
	return b
}

// Build creates the HTTP request
func (b *RequestBuilder) Build() (*http.Request, error) {
	req, err := http.NewRequest(b.method, b.url, b.body)
	if err != nil {
		return nil, err
	}

	// Add headers
	for key, value := range b.headers {
		req.Header.Add(key, value)
	}

	// Add query parameters
	q := req.URL.Query()
	for key, value := range b.queryParams {
		q.Add(key, value)
	}
	req.URL.RawQuery = q.Encode()

	return req, nil
}

// Send builds and sends the HTTP request
func (b *RequestBuilder) Send() (*http.Response, error) {
	req, err := b.Build()
	if err != nil {
		return nil, err
	}

	if b.timeout > 0 {
		ctx, cancel := context.WithTimeout(context.Background(), b.timeout)
		defer cancel()
		req = req.WithContext(ctx)
	}

	return b.client.Do(req)
}
```

**Usage**:

```go
func main() {
	// Build and send a JSON request
	response, err := request.NewRequestBuilder().
		Method("POST").
		URL("https://api.example.com/users").
		Header("Authorization", "Bearer token123").
		QueryParam("version", "2").
		JSONBody(map[string]interface{}{
			"name": "John Doe",
			"email": "john@example.com",
		}).
		Timeout(5 * time.Second).
		Send()

	if err != nil {
		log.Fatalf("Request failed: %v", err)
	}
	defer response.Body.Close()

	// Process response...
}
```

**When to Use in Go**:

- Creating complex objects step by step
- When an object has many optional parameters
- When you want to enforce construction order

**Go-Specific Considerations**:

- Method chaining works well with Go's pointer receivers
- Consider functional options pattern for simpler cases
- Provide sensible defaults for optional parameters

## **23.3. Structural Patterns**

Structural patterns focus on how classes and objects are composed to form larger structures.

### **4. Adapter Pattern**

**Purpose**: Convert the interface of a class into another interface clients expect.

**Go Implementation**: In Go, adapters often wrap one interface to satisfy another.

**Example: Payment Gateway Adapter**

```go
package payment

// ThirdPartyPayment represents an external payment processor's interface
type ThirdPartyPayment interface {
	ExecuteTransaction(amount int, currency string, destination string) (string, error)
	VerifyTransaction(id string) (bool, error)
}

// Our application's payment interface
type PaymentProcessor interface {
	Pay(amount float64, destination string) (string, error)
	Verify(transactionID string) (bool, error)
}

// PaymentAdapter adapts ThirdPartyPayment to our PaymentProcessor interface
type PaymentAdapter struct {
	processor ThirdPartyPayment
}

// NewPaymentAdapter creates a new adapter
func NewPaymentAdapter(processor ThirdPartyPayment) PaymentProcessor {
	return &PaymentAdapter{
		processor: processor,
	}
}

// Pay implements PaymentProcessor.Pay by adapting to ThirdPartyPayment.ExecuteTransaction
func (a *PaymentAdapter) Pay(amount float64, destination string) (string, error) {
	// Convert amount to cents (int)
	amountInCents := int(amount * 100)

	// Call the adapted method
	return a.processor.ExecuteTransaction(amountInCents, "USD", destination)
}

// Verify implements PaymentProcessor.Verify
func (a *PaymentAdapter) Verify(transactionID string) (bool, error) {
	return a.processor.VerifyTransaction(transactionID)
}

// Example implementation of ThirdPartyPayment
type StripePaymentProcessor struct{}

func (s *StripePaymentProcessor) ExecuteTransaction(amount int, currency string, destination string) (string, error) {
	// Implementation omitted
	return "tx_12345", nil
}

func (s *StripePaymentProcessor) VerifyTransaction(id string) (bool, error) {
	// Implementation omitted
	return true, nil
}
```

**Usage**:

```go
func main() {
	// Create the third-party processor
	stripeProcessor := &payment.StripePaymentProcessor{}

	// Adapt it to our interface
	paymentProcessor := payment.NewPaymentAdapter(stripeProcessor)

	// Use our interface
	transactionID, err := paymentProcessor.Pay(99.99, "customer@example.com")
	if err != nil {
		log.Fatalf("Payment failed: %v", err)
	}

	// Verify payment
	verified, _ := paymentProcessor.Verify(transactionID)
	fmt.Printf("Transaction %s verified: %v\n", transactionID, verified)
}
```

**When to Use in Go**:

- Integrating with third-party libraries
- Making incompatible interfaces work together
- Providing a consistent interface over varied implementations

**Go-Specific Considerations**:

- Go's implicit interfaces make adapters simpler
- Consider embedding for simple adaptations
- Focus on small, focused interfaces

### **5. Decorator Pattern**

**Purpose**: Attach additional responsibilities to objects dynamically.

**Go Implementation**: In Go, decorators typically wrap interfaces and add functionality.

**Example: HTTP Middleware Decorator**

```go
package http

import (
	"log"
	"net/http"
	"time"
)

// Handler is our simplified HTTP handler interface
type Handler interface {
	ServeHTTP(w http.ResponseWriter, r *http.Request)
}

// LoggingDecorator adds logging capabilities to any Handler
type LoggingDecorator struct {
	next Handler
}

// NewLoggingDecorator creates a new logging decorator
func NewLoggingDecorator(next Handler) Handler {
	return &LoggingDecorator{next: next}
}

// ServeHTTP implements the Handler interface
func (d *LoggingDecorator) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	start := time.Now()

	log.Printf("Started %s %s", r.Method, r.URL.Path)

	// Call the wrapped handler
	d.next.ServeHTTP(w, r)

	log.Printf("Completed %s %s in %v", r.Method, r.URL.Path, time.Since(start))
}

// AuthDecorator adds authentication to any Handler
type AuthDecorator struct {
	next Handler
}

// NewAuthDecorator creates a new auth decorator
func NewAuthDecorator(next Handler) Handler {
	return &AuthDecorator{next: next}
}

// ServeHTTP implements the Handler interface
func (d *AuthDecorator) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// Check for auth token
	token := r.Header.Get("Authorization")
	if token == "" {
		http.Error(w, "Unauthorized", http.StatusUnauthorized)
		return
	}

	// Token validation would go here

	// Call the wrapped handler
	d.next.ServeHTTP(w, r)
}

// CompressionDecorator adds response compression
type CompressionDecorator struct {
	next Handler
}

// NewCompressionDecorator creates a new compression decorator
func NewCompressionDecorator(next Handler) Handler {
	return &CompressionDecorator{next: next}
}

// ServeHTTP implements the Handler interface
func (d *CompressionDecorator) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	// Check if client accepts gzip
	if !strings.Contains(r.Header.Get("Accept-Encoding"), "gzip") {
		d.next.ServeHTTP(w, r)
		return
	}

	// Create gzip response writer
	w.Header().Set("Content-Encoding", "gzip")
	gz := gzip.NewWriter(w)
	defer gz.Close()

	gzw := &gzipResponseWriter{Writer: gz, ResponseWriter: w}
	d.next.ServeHTTP(gzw, r)
}

// Simple handler that will be decorated
type HelloHandler struct{}

func (h *HelloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello, World!"))
}
```

**Usage**:

```go
func main() {
	// Create a base handler
	helloHandler := &HelloHandler{}

	// Decorate it with multiple behaviors
	handler := NewLoggingDecorator(
		NewAuthDecorator(
			NewCompressionDecorator(
				helloHandler,
			),
		),
	)

	// Use the decorated handler
	http.ListenAndServe(":8080", handler)
}
```

**When to Use in Go**:

- Adding cross-cutting concerns like logging or metrics
- Building middleware chains
- Adding optional features to core components

**Go-Specific Considerations**:

- Go's interfaces make decorators elegant and type-safe
- Consider using functional middleware for simpler cases
- Decorators can be combined with the builder pattern for cleaner API

## **23.11. Go-Specific Patterns**

In addition to classic design patterns, Go has developed its own idioms and patterns that leverage the language's unique features.

### **Go Pattern 1: Functional Options**

**Purpose**: Provide a flexible way to configure structs with optional parameters.

**Example: Server Configuration**

```go
package server

import (
	"crypto/tls"
	"time"
)

// Server represents an HTTP server
type Server struct {
	host         string
	port         int
	readTimeout  time.Duration
	writeTimeout time.Duration
	maxConns     int
	tls          *tls.Config
}

// Option defines a server option
type Option func(*Server)

// WithPort sets the server port
func WithPort(port int) Option {
	return func(s *Server) {
		s.port = port
	}
}

// WithTimeout sets the read and write timeouts
func WithTimeout(timeout time.Duration) Option {
	return func(s *Server) {
		s.readTimeout = timeout
		s.writeTimeout = timeout
	}
}

// WithTLS enables TLS with the provided config
func WithTLS(cert, key string) Option {
	return func(s *Server) {
		cert, err := tls.LoadX509KeyPair(cert, key)
		if err != nil {
			return
		}
		s.tls = &tls.Config{
			Certificates: []tls.Certificate{cert},
		}
	}
}

// WithMaxConnections sets the maximum number of connections
func WithMaxConnections(n int) Option {
	return func(s *Server) {
		s.maxConns = n
	}
}

// NewServer creates a server with the given options
func NewServer(host string, options ...Option) *Server {
	// Default configuration
	server := &Server{
		host:         host,
		port:         8080,
		readTimeout:  30 * time.Second,
		writeTimeout: 30 * time.Second,
		maxConns:     1000,
	}

	// Apply options
	for _, option := range options {
		option(server)
	}

	return server
}
```

**Usage**:

```go
func main() {
	// Create server with defaults
	server1 := server.NewServer("localhost")

	// Create server with options
	server2 := server.NewServer("api.example.com",
		server.WithPort(443),
		server.WithTimeout(10 * time.Second),
		server.WithTLS("cert.pem", "key.pem"),
		server.WithMaxConnections(10000),
	)

	// Start servers
	server1.Start()
	server2.Start()
}
```

**When to Use**:

- Configuring complex objects with many optional parameters
- Creating clean, flexible APIs
- When you want to provide sensible defaults

### **Go Pattern 2: Error Wrapping**

**Purpose**: Provide context to errors while preserving the original error.

**Example: Error Context Chain**

```go
package main

import (
	"database/sql"
	"errors"
	"fmt"
)

// UserService handles user-related operations
type UserService struct {
	db *sql.DB
}

// GetUser retrieves a user by ID
func (s *UserService) GetUser(id int) (User, error) {
	user, err := s.findUserByID(id)
	if err != nil {
		return User{}, fmt.Errorf("GetUser: %w", err)
	}
	return user, nil
}

// findUserByID is an internal method to fetch a user
func (s *UserService) findUserByID(id int) (User, error) {
	var user User
	err := s.db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id).
		Scan(&user.ID, &user.Name, &user.Email)

	if err != nil {
		if errors.Is(err, sql.ErrNoRows) {
			return User{}, fmt.Errorf("findUserByID: user with id %d not found: %w", id, err)
		}
		return User{}, fmt.Errorf("findUserByID: database error: %w", err)
	}

	return user, nil
}
```

**Usage**:

```go
func main() {
	userService := &UserService{db: database}

	user, err := userService.GetUser(123)
	if err != nil {
		if errors.Is(err, sql.ErrNoRows) {
			// Handle not found case
			fmt.Println("User not found")
		} else {
			// Handle other errors
			fmt.Printf("Error: %v\n", err)
		}
		return
	}

	fmt.Printf("Found user: %+v\n", user)
}
```

**When to Use**:

- Building error chains in multi-layer applications
- Providing context while preserving original error types
- When error handling needs to be comprehensive

### **Go Pattern 3: Context Package for Cancellation**

**Purpose**: Manage request-scoped data, cancellation signals, and deadlines.

**Example: Cancelable Operations**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

// DataFetcher fetches data from various sources
type DataFetcher struct{}

// FetchData gets data from a slow source with cancellation support
func (df *DataFetcher) FetchData(ctx context.Context, query string) ([]string, error) {
	results := make(chan []string, 1)
	errors := make(chan error, 1)

	go func() {
		// Simulate work
		time.Sleep(2 * time.Second)

		// Check if canceled before returning
		select {
		case <-ctx.Done():
			errors <- ctx.Err()
			return
		default:
			// Work completed successfully
			results <- []string{"result1", "result2", query}
		}
	}()

	// Wait for result or cancellation
	select {
	case result := <-results:
		return result, nil
	case err := <-errors:
		return nil, err
	case <-ctx.Done():
		return nil, ctx.Err()
	}
}
```

**Usage**:

```go
func main() {
	fetcher := &DataFetcher{}

	// Create a context with a timeout
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	// Try to fetch data
	results, err := fetcher.FetchData(ctx, "query")
	if err != nil {
		fmt.Printf("Error: %v\n", err) // Will print: Error: context deadline exceeded
		return
	}

	fmt.Printf("Results: %v\n", results)
}
```

**When to Use**:

- Managing request timeouts
- Implementing cancelable operations
- Propagating deadlines through call chains
- Passing request-scoped values

## **23.12. Summary and Best Practices**

### **Key Takeaways**

1. **Adapt patterns to Go's idioms**: Traditional design patterns often need adaptation to fit Go's paradigms.

2. **Interfaces are powerful**: Go's implicit interfaces enable flexible, decoupled designs.

3. **Composition over inheritance**: Use embedding and composition to build complex structures.

4. **Keep it simple**: The best Go code often uses the simplest approach that works, not necessarily a formal pattern.

5. **Test-driven development**: Design patterns should support, not hinder, testability.

### **Best Practices for Design Patterns in Go**

1. **Start simple**: Begin with straightforward implementations before applying patterns.

2. **Use interfaces wisely**: Define interfaces at the point of use, keep them small and focused.

3. **Avoid over-engineering**: Apply patterns only when they clearly improve your design.

4. **Consider maintainability**: Patterns should make your code more maintainable, not less.

5. **Document pattern usage**: Comment when you're applying a specific pattern to aid understanding.

### **Pattern Selection Guide**

| When You Need To...                         | Consider Using...             |
| ------------------------------------------- | ----------------------------- |
| Create objects based on conditions          | Factory Method                |
| Ensure exactly one instance exists          | Singleton                     |
| Build complex objects step by step          | Builder or Functional Options |
| Make incompatible interfaces work together  | Adapter                       |
| Add behaviors dynamically                   | Decorator                     |
| Define a family of algorithms               | Strategy                      |
| Execute operations in a particular sequence | Chain of Responsibility       |
| Represent operations as objects             | Command                       |
| Manage object state transitions             | State                         |
| Add indirection to object access            | Proxy                         |

## **23.13. Further Resources**

1. **Books**:

   - "Go Design Patterns" by Mario Castro Contreras
   - "Hands-On Software Architecture with Golang" by Jyotiswarup Raiturkar

2. **Online Resources**:

   - [Go Patterns](https://github.com/tmrts/go-patterns)
   - [Effective Go](https://golang.org/doc/effective_go)
   - [Go by Example](https://gobyexample.com/)

3. **Standard Library Examples**:
   - context package (Context pattern)
   - net/http (Middleware/Decorator pattern)
   - database/sql (Proxy pattern)

## **23.14. Exercises**

To solidify your understanding, try implementing the following patterns on your own:

// ... existing exercises ...

In the next chapter, we'll explore performance optimization techniques for Go applications, building on these design patterns to create not just well-designed code, but efficient code as well.
