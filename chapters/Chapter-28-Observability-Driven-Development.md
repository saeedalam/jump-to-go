# **Chapter 28: Observability-Driven Development with Go**

## **28.1 Introduction to Observability-Driven Development**

Observability-Driven Development (ODD) is an approach to software engineering that places observability at the core of the development process. Rather than treating monitoring and observability as an afterthought, ODD integrates them into the development lifecycle from the beginning, making systems that are designed to be understood, debugged, and improved.

In today's complex, distributed systems, understanding what's happening inside your applications is more important than ever. Traditional monitoring approaches that focus on predefined metrics and known failure modes are no longer sufficient. Modern systems require deep, comprehensive observability that allows engineers to ask new questions about system behavior without deploying new code.

Go, with its robust standard library, excellent performance characteristics, and growing ecosystem of observability tools, is an ideal language for implementing Observability-Driven Development. In this chapter, we'll explore how to apply ODD principles to Go applications, covering everything from basic instrumentation to advanced observability patterns.

### **28.1.1 What is Observability?**

Observability originates from control theory and refers to how well a system's internal states can be understood from its external outputs. In software engineering, observability means having enough data about your system's behavior to understand:

- What's happening right now
- What happened in the past
- Why it's happening
- What might happen next

The three pillars of observability are:

1. **Logs**: Time-stamped records of discrete events that occurred in the system
2. **Metrics**: Numeric measurements of system behavior sampled over time
3. **Traces**: Records of requests as they flow through distributed systems

When combined, these pillars provide a comprehensive view of your application's behavior, allowing you to understand complex interactions and troubleshoot issues effectively.

### **28.1.2 From Monitoring to Observability**

Traditional monitoring focuses on watching known metrics and alerting when predefined thresholds are crossed. While valuable, this approach has limitations:

- It only shows what you know to look for
- It's reactive rather than proactive
- It doesn't help with unknown unknowns

Observability expands on monitoring by:

- Allowing exploration of system behavior
- Supporting hypothesis-driven debugging
- Enabling the discovery of unknown issues
- Facilitating root cause analysis

Observability doesn't replace monitoring—it enhances it. Good observability makes monitoring more effective by providing context and depth to alerts.

### **28.1.3 Core Principles of Observability-Driven Development**

Observability-Driven Development is guided by several key principles:

1. **Design for Observability**: Make observability a first-class concern in system design
2. **Instrument Everything**: Add comprehensive instrumentation to code from the start
3. **Collect the Right Data**: Gather data that provides insight, not just volume
4. **Context is King**: Ensure all telemetry data contains relevant context
5. **Correlation is Critical**: Make it possible to correlate data across the three pillars
6. **Test Observability**: Verify that your observability features work as expected
7. **Continuous Improvement**: Use observability data to drive system improvements

By following these principles, you create systems that are easier to understand, debug, and evolve.

### **28.1.4 Benefits of Observability-Driven Development**

Adopting ODD provides numerous benefits:

- **Reduced Mean Time to Resolution (MTTR)**: Quickly identify and fix issues
- **Improved System Reliability**: Catch problems before they affect users
- **Better Development Velocity**: Make changes with confidence
- **Enhanced Collaboration**: Share a common understanding of system behavior
- **Data-Driven Decisions**: Base improvements on actual usage patterns
- **Improved User Experience**: Identify and address performance issues proactively

These benefits compound over time, making ODD an investment that pays increasing dividends as systems grow in complexity.

## **28.2 Building Observable Go Applications**

Creating observable Go applications starts with proper instrumentation. In this section, we'll explore how to add observability to Go code using both standard library features and popular third-party packages.

### **28.2.1 Logging in Go**

Logs are often the first observability tool developers reach for. They provide a detailed record of what happened in your application and are invaluable for debugging and understanding system behavior.

#### **Using the Standard Library**

Go's standard library includes the `log` package, which provides basic logging functionality:

```go
package main

import (
    "log"
    "os"
)

func main() {
    // Create a logger that writes to stdout with a custom prefix and flags
    logger := log.New(os.Stdout, "APP: ", log.Ldate|log.Ltime|log.Lshortfile)

    // Log messages at different levels
    logger.Println("This is an informational message")
    logger.Printf("Processing item %d", 123)
    logger.Fatal("This is a fatal error") // Logs and calls os.Exit(1)
}
```

While the standard library's logging is functional, it's fairly basic. For production applications, you'll typically want a more feature-rich logging solution.

#### **Structured Logging with zerolog**

Structured logging improves upon traditional text-based logging by organizing log data into consistent, queryable fields. The `zerolog` package is a popular choice for structured logging in Go:

```go
package main

import (
    "os"
    "time"

    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

func main() {
    // Configure global logger
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix
    log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stdout, TimeFormat: time.RFC3339})

    // Basic structured logging
    log.Info().
        Str("service", "order-api").
        Int("user_id", 123).
        Str("action", "order_created").
        Int("order_id", 456).
        Msg("Order successfully created")

    // Log with error
    err := processOrder(456)
    if err != nil {
        log.Error().
            Err(err).
            Int("order_id", 456).
            Msg("Failed to process order")
    }
}

func processOrder(orderID int) error {
    // Process order logic...
    return nil
}
```

Structured logging offers several advantages:

- **Consistent format**: Logs are machine-parseable and have a consistent structure
- **Better filtering**: Filter logs based on specific fields rather than text patterns
- **Enhanced context**: Add rich context to every log entry
- **Easier analysis**: Aggregate and analyze logs more effectively

#### **Context-Aware Logging**

In distributed systems, it's crucial to correlate logs across services. One approach is to use context-aware logging:

```go
package main

import (
    "context"
    "net/http"

    "github.com/google/uuid"
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
)

// contextKey is a type for context keys to avoid collisions
type contextKey string

const requestIDKey = contextKey("requestID")

// RequestIDMiddleware adds a unique request ID to each request context
func RequestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Generate or retrieve request ID
        requestID := r.Header.Get("X-Request-ID")
        if requestID == "" {
            requestID = uuid.New().String()
        }

        // Add request ID to response headers
        w.Header().Set("X-Request-ID", requestID)

        // Create context with request ID
        ctx := context.WithValue(r.Context(), requestIDKey, requestID)

        // Call next handler with updated context
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// LogMiddleware logs details about each request
func LogMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Extract request ID from context
        requestID, _ := r.Context().Value(requestIDKey).(string)

        // Create logger with request context
        logger := log.With().
            Str("request_id", requestID).
            Str("method", r.Method).
            Str("path", r.URL.Path).
            Str("remote_addr", r.RemoteAddr).
            Logger()

        // Store logger in context
        ctx := logger.WithContext(r.Context())

        // Log request start
        logger.Info().Msg("Request started")

        // Call next handler
        next.ServeHTTP(w, r.WithContext(ctx))

        // Log request completion
        logger.Info().Msg("Request completed")
    })
}

// GetLoggerFromContext extracts the logger from context
func GetLoggerFromContext(ctx context.Context) zerolog.Logger {
    if logger, ok := zerolog.Ctx(ctx); ok {
        return *logger
    }
    return log.Logger
}

// GetRequestID extracts the request ID from context
func GetRequestID(ctx context.Context) string {
    if requestID, ok := ctx.Value(requestIDKey).(string); ok {
        return requestID
    }
    return ""
}

// GetCurrentSpan gets the current span from context
func GetCurrentSpan(ctx context.Context) trace.Span {
    return trace.SpanFromContext(ctx)
}

// Handler uses context-aware logging
func Handler(w http.ResponseWriter, r *http.Request) {
    // Get logger and span from context
    logger := GetLoggerFromContext(r.Context())
    span := GetCurrentSpan(r.Context())

    // Log and trace the operation
    logger.Info().Msg("Processing order")
    span.AddEvent("Processing order")

    // Perform business logic...

    w.Write([]byte("Order processed"))
}
```

This example demonstrates:

1. **Common identifier**: A request ID is generated and used across all telemetry
2. **Context propagation**: The context carries correlation IDs through the request
3. **Enriched logs**: Logs include trace and span IDs for correlation
4. **Labeled metrics**: Metrics include the request ID for correlation
5. **Annotated spans**: Spans include the request ID as an attribute

#### **Best Practices for Logging**

When implementing logging in Go applications, follow these best practices:

1. **Use structured logging**: Prefer structured logs over plain text
2. **Include context**: Add relevant context to every log entry
3. **Choose appropriate log levels**: Use different levels (debug, info, warn, error) consistently
4. **Be careful with sensitive data**: Avoid logging passwords, tokens, or personal information
5. **Don't overlog**: Excessive logging creates noise and can impact performance
6. **Include request IDs**: Add unique identifiers to track requests across services
7. **Log for humans and machines**: Ensure logs are both human-readable and machine-parseable
8. **Consider log sampling**: In high-throughput systems, consider sampling verbose logs

With proper logging in place, you've established the first pillar of observability. Next, let's explore metrics.

### **28.2.2 Metrics and Monitoring**

Metrics provide numerical measurements of your application's behavior over time. They're essential for monitoring system health, performance, and business outcomes.

#### **Types of Metrics**

There are several types of metrics you should consider collecting:

1. **System Metrics**: CPU, memory, disk, network usage
2. **Application Metrics**: Request counts, error rates, response times
3. **Business Metrics**: User signups, orders placed, revenue
4. **Dependency Metrics**: Database query times, external API response times

#### **The RED Method**

For service monitoring, the RED method provides a simple framework:

- **Rate**: Requests per second
- **Errors**: Failed requests per second
- **Duration**: Distribution of request latencies

These three metrics give you a comprehensive view of service health.

#### **The Four Golden Signals**

Google's Site Reliability Engineering book recommends monitoring these four signals:

- **Latency**: Time taken to serve requests
- **Traffic**: Demand on your system
- **Errors**: Rate of failed requests
- **Saturation**: How "full" your system is

#### **Using Prometheus with Go**

Prometheus is a popular open-source monitoring system and time series database. The official Prometheus client library for Go makes it easy to instrument your code:

```go
package main

import (
    "net/http"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    // Counter for total HTTP requests
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )

    // Histogram for HTTP request duration
    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )

    // Gauge for active requests
    httpRequestsInProgress = promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Name: "http_requests_in_progress",
            Help: "Number of HTTP requests currently in progress",
        },
        []string{"method", "endpoint"},
    )
)

// PrometheusMiddleware instruments HTTP handlers with metrics
func PrometheusMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Track request start time
        startTime := time.Now()

        // Track in-progress requests
        httpRequestsInProgress.WithLabelValues(r.Method, r.URL.Path).Inc()
        defer httpRequestsInProgress.WithLabelValues(r.Method, r.URL.Path).Dec()

        // Create response writer that captures status code
        rww := NewResponseWriterWrapper(w)

        // Call the next handler
        next.ServeHTTP(rww, r)

        // Record metrics after request is processed
        duration := time.Since(startTime).Seconds()
        status := rww.StatusCode()

        // Increment request counter
        httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, status).Inc()

        // Observe request duration
        httpRequestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration)
    })
}

// ResponseWriterWrapper captures the status code
type ResponseWriterWrapper struct {
    http.ResponseWriter
    statusCode int
}

// NewResponseWriterWrapper creates a new wrapper
func NewResponseWriterWrapper(w http.ResponseWriter) *ResponseWriterWrapper {
    return &ResponseWriterWrapper{w, http.StatusOK}
}

// WriteHeader captures the status code
func (rww *ResponseWriterWrapper) WriteHeader(code int) {
    rww.statusCode = code
    rww.ResponseWriter.WriteHeader(code)
}

// StatusCode returns the captured status code
func (rww *ResponseWriterWrapper) StatusCode() string {
    return http.StatusText(rww.statusCode)
}

// Handler for business logic
func helloHandler(w http.ResponseWriter, r *http.Request) {
    time.Sleep(time.Duration(100+time.Now().UnixNano()%100) * time.Millisecond) // Simulate work
    w.Write([]byte("Hello, world!"))
}

func main() {
    // Set up HTTP server with Prometheus middleware
    http.Handle("/hello", PrometheusMiddleware(http.HandlerFunc(helloHandler)))

    // Expose Prometheus metrics endpoint
    http.Handle("/metrics", promhttp.Handler())

    // Start server
    http.ListenAndServe(":8080", nil)
}
```

This example demonstrates:

1. **Counters**: Track the total number of events (e.g., requests)
2. **Gauges**: Measure current values (e.g., in-progress requests)
3. **Histograms**: Track distribution of values (e.g., response times)

Prometheus can scrape these metrics from your `/metrics` endpoint and store them for querying, alerting, and visualization.

#### **Custom Business Metrics**

Beyond technical metrics, it's important to track business-relevant metrics:

```go
package main

import (
    "net/http"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    // Counter for orders placed
    ordersTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "orders_total",
            Help: "Total number of orders placed",
        },
        []string{"status", "payment_method"},
    )

    // Gauge for current cart count
    activeCartCount = promauto.NewGauge(
        prometheus.GaugeOpts{
            Name: "active_cart_count",
            Help: "Number of active shopping carts",
        },
    )

    // Histogram for order values
    orderValueDollars = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "order_value_dollars",
            Help:    "Distribution of order values in dollars",
            Buckets: []float64{10, 25, 50, 100, 250, 500, 1000},
        },
        []string{"product_category"},
    )
)

// PlaceOrder simulates an order placement
func PlaceOrder(orderValue float64, status, paymentMethod, category string) {
    // Increment orders counter
    ordersTotal.WithLabelValues(status, paymentMethod).Inc()

    // Record order value in histogram
    orderValueDollars.WithLabelValues(category).Observe(orderValue)

    // Decrement active cart count
    activeCartCount.Dec()
}

// CreateCart simulates creating a new shopping cart
func CreateCart() {
    activeCartCount.Inc()
}

func main() {
    // Set up API routes (simplified)
    http.HandleFunc("/api/cart/create", func(w http.ResponseWriter, r *http.Request) {
        CreateCart()
        w.WriteHeader(http.StatusCreated)
    })

    http.HandleFunc("/api/order/place", func(w http.ResponseWriter, r *http.Request) {
        // In a real app, these would come from the request
        PlaceOrder(123.45, "completed", "credit_card", "electronics")
        w.WriteHeader(http.StatusCreated)
    })

    // Expose Prometheus metrics endpoint
    http.Handle("/metrics", promhttp.Handler())

    // Start server
    http.ListenAndServe(":8080", nil)
}
```

These business metrics can provide insights into user behavior and business performance, complementing technical metrics.

#### **Automatic Runtime Metrics**

The Prometheus client library can automatically collect Go runtime metrics:

```go
package main

import (
    "net/http"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/collectors"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
    // Create a new registry
    reg := prometheus.NewRegistry()

    // Add Go collectors
    reg.MustRegister(
        collectors.NewGoCollector(),        // Go runtime metrics
        collectors.NewProcessCollector(collectors.ProcessCollectorOpts{}), // Process metrics
    )

    // Create a metrics handler using the registry
    handler := promhttp.HandlerFor(reg, promhttp.HandlerOpts{})

    // Expose metrics endpoint
    http.Handle("/metrics", handler)

    // Start server
    http.ListenAndServe(":8080", nil)
}
```

This provides important metrics about your Go application's runtime behavior, including garbage collection, goroutine count, and memory usage.

#### **Best Practices for Metrics**

When implementing metrics in Go applications, follow these best practices:

1. **Use meaningful names**: Choose clear, descriptive metric names
2. **Add appropriate labels**: Use labels to segment metrics, but avoid high cardinality
3. **Choose the right metric type**: Use counters for events, gauges for current values, and histograms for distributions
4. **Set appropriate buckets**: Configure histogram buckets based on expected value ranges
5. **Measure what matters**: Focus on metrics that drive decisions or alerts
6. **Document metrics**: Add helpful descriptions to each metric
7. **Standardize across services**: Use consistent naming and labeling conventions
8. **Watch for performance**: Excessive metrics can impact application performance

With proper metrics in place, you've established the second pillar of observability. Next, let's explore distributed tracing.

### **28.2.3 Distributed Tracing**

Distributed tracing tracks requests as they flow through distributed systems, providing visibility into how services interact and where performance bottlenecks occur. This is particularly valuable in microservice architectures.

#### **OpenTelemetry and Go**

OpenTelemetry is a collection of tools, APIs, and SDKs for generating, collecting, and exporting telemetry data. It's becoming the industry standard for distributed tracing:

```go
package main

import (
    "context"
    "io"
    "log"
    "net/http"
    "time"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/exporters/jaeger"
    "go.opentelemetry.io/otel/sdk/resource"
    tracesdk "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.7.0"
    "go.opentelemetry.io/otel/trace"
)

// initTracer initializes the OpenTelemetry tracer
func initTracer() (func(), error) {
    // Configure Jaeger exporter
    exporter, err := jaeger.New(jaeger.WithCollectorEndpoint(
        jaeger.WithEndpoint("http://localhost:14268/api/traces"),
    ))
    if err != nil {
        return nil, err
    }

    // Configure trace provider with the exporter
    tp := tracesdk.NewTracerProvider(
        tracesdk.WithBatcher(exporter),
        tracesdk.WithResource(resource.NewWithAttributes(
            semconv.SchemaURL,
            semconv.ServiceNameKey.String("order-service"),
            attribute.String("environment", "production"),
        )),
    )

    // Set the global trace provider
    otel.SetTracerProvider(tp)

    // Return a function to flush and close the exporter when the application exits
    return func() {
        if err := tp.Shutdown(context.Background()); err != nil {
            log.Printf("Error shutting down tracer provider: %v", err)
        }
    }, nil
}

// Handler with tracing
func orderHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    tracer := otel.Tracer("order-service")

    // Create a span for this handler
    ctx, span := tracer.Start(ctx, "orderHandler")
    defer span.End()

    // Add attributes to the span
    span.SetAttributes(
        attribute.String("http.method", r.Method),
        attribute.String("http.path", r.URL.Path),
    )

    // Call database function with the context containing the span
    orderID, err := createOrder(ctx)
    if err != nil {
        span.RecordError(err)
        http.Error(w, "Failed to create order", http.StatusInternalServerError)
        return
    }

    // Call payment service
    err = processPayment(ctx, orderID)
    if err != nil {
        span.RecordError(err)
        http.Error(w, "Failed to process payment", http.StatusInternalServerError)
        return
    }

    w.WriteHeader(http.StatusCreated)
    w.Write([]byte(orderID))
}

// createOrder simulates database operations with tracing
func createOrder(ctx context.Context) (string, error) {
    tracer := otel.Tracer("order-service")

    // Create a child span for database operation
    ctx, span := tracer.Start(ctx, "database.createOrder")
    defer span.End()

    // Simulate database work
    time.Sleep(100 * time.Millisecond)

    // Simulate order ID
    orderID := "ord-123456"

    // Add attributes with order details
    span.SetAttributes(
        attribute.String("order.id", orderID),
        attribute.Float64("order.amount", 99.99),
    )

    return orderID, nil
}

// processPayment simulates calling a payment service with tracing
func processPayment(ctx context.Context, orderID string) error {
    tracer := otel.Tracer("order-service")

    // Create a child span for payment service call
    ctx, span := tracer.Start(ctx, "payment.processPayment")
    defer span.End()

    // Add context propagation headers for the HTTP request
    req, _ := http.NewRequestWithContext(ctx, "POST", "http://payment-service/api/payments", nil)

    // Simulate calling payment service
    span.AddEvent("Calling payment service")
    time.Sleep(200 * time.Millisecond)

    // Add payment result to span
    span.SetAttributes(
        attribute.String("payment.id", "pay-789012"),
        attribute.String("payment.status", "approved"),
    )

    return nil
}

func main() {
    // Initialize tracer
    cleanup, err := initTracer()
    if err != nil {
        log.Fatalf("Failed to initialize tracer: %v", err)
    }
    defer cleanup()

    // Set up HTTP server
    http.HandleFunc("/api/orders", orderHandler)
    log.Println("Starting server on :8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This example demonstrates:

1. **Span creation**: Creating parent and child spans to track operations
2. **Context propagation**: Passing trace context through the application
3. **Attribute addition**: Adding metadata to spans for analysis
4. **Error recording**: Capturing errors in spans
5. **Event recording**: Adding events to spans for important actions

#### **HTTP Middleware for Tracing**

To automatically trace HTTP requests, you can create middleware:

```go
package main

import (
    "log"
    "net/http"

    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/propagation"
)

func main() {
    // Initialize tracer (implementation omitted for brevity)

    // Set up global propagator
    otel.SetTextMapPropagator(propagation.TraceContext{})

    // Create handler with automatic instrumentation
    handler := http.HandlerFunc(orderHandler)
    instrumentedHandler := otelhttp.NewHandler(handler, "orderHandler")

    // Register HTTP handler
    http.Handle("/api/orders", instrumentedHandler)

    // Start server
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

This middleware automatically:

- Creates spans for each HTTP request
- Extracts trace context from incoming requests
- Injects trace context into outgoing requests
- Records HTTP method, status code, and URL as span attributes

#### **Database Tracing**

To trace database operations, you can use instrumented database drivers:

```go
package main

import (
    "context"
    "database/sql"
    "log"

    "github.com/luna-duclos/instrumentedsql"
    "github.com/luna-duclos/instrumentedsql/opentelemetry"
    _ "github.com/luna-duclos/instrumentedsql/mysql"
)

func initDatabase() (*sql.DB, error) {
    // Create a tracer for SQL operations
    sqlTracer := opentelemetry.NewTracer()

    // Create a driver with tracing
    driverName := instrumentedsql.WrapDriver(
        &mysql.MySQLDriver{},
        instrumentedsql.WithTracer(sqlTracer),
        instrumentedsql.WithOmitArgs(), // Don't log query arguments for security
    )

    // Register the instrumented driver
    sql.Register("instrumented-mysql", driverName)

    // Connect to database using the instrumented driver
    db, err := sql.Open("instrumented-mysql", "user:password@tcp(localhost:3306)/db")
    if err != nil {
        return nil, err
    }

    return db, nil
}

func queryOrders(ctx context.Context, db *sql.DB, customerID string) ([]Order, error) {
    // The query will be automatically traced
    rows, err := db.QueryContext(ctx, "SELECT id, amount, created_at FROM orders WHERE customer_id = ?", customerID)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    // Process query results
    // ...

    return orders, nil
}
```

#### **Best Practices for Distributed Tracing**

When implementing tracing in Go applications, follow these best practices:

1. **Propagate context**: Always pass context through function calls
2. **Use meaningful span names**: Choose descriptive names for spans
3. **Add relevant attributes**: Include information that helps with debugging
4. **Record errors**: Add error details to spans when errors occur
5. **Create child spans for sub-operations**: Break down complex operations
6. **Set appropriate sampling**: Use head-based sampling for high-volume services
7. **Secure sensitive data**: Don't include passwords or personal data in spans
8. **Standardize span naming**: Use consistent naming conventions across services

With tracing implemented, you've established all three pillars of observability: logs, metrics, and traces.

## **28.3 Integrating the Three Pillars of Observability**

While each pillar of observability—logs, metrics, and traces—provides valuable insights on its own, their true power emerges when they're integrated. This integration allows engineers to move seamlessly between different types of telemetry data, providing a comprehensive view of system behavior.

### **28.3.1 Correlating Logs, Metrics, and Traces**

The key to effective observability is correlation. By connecting data across pillars, you can quickly navigate from a high-level metric to detailed logs and traces that explain what's happening.

#### **Using Common Identifiers**

The simplest way to correlate data is to use common identifiers across all telemetry data:

```go
package main

import (
    "context"
    "net/http"
    "time"

    "github.com/google/uuid"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/rs/zerolog/log"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"
)

var (
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path", "request_id"},
    )
)

func CorrelatedMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Generate request ID
        requestID := uuid.New().String()

        // Add request ID to response headers
        w.Header().Set("X-Request-ID", requestID)

        // Start timing
        startTime := time.Now()

        // Get tracer and start span
        tracer := otel.Tracer("web-service")
        ctx, span := tracer.Start(r.Context(), "http_request")
        defer span.End()

        // Add request ID to span
        span.SetAttributes(attribute.String("request_id", requestID))

        // Add trace ID to logs
        traceID := span.SpanContext().TraceID().String()
        spanID := span.SpanContext().SpanID().String()

        // Create logger with correlation IDs
        logger := log.With().
            Str("request_id", requestID).
            Str("trace_id", traceID).
            Str("span_id", spanID).
            Str("method", r.Method).
            Str("path", r.URL.Path).
            Logger()

        // Log request start
        logger.Info().Msg("Request started")

        // Store logger and request ID in context
        ctx = context.WithValue(ctx, "logger", logger)
        ctx = context.WithValue(ctx, "request_id", requestID)

        // Call next handler with updated context
        next.ServeHTTP(w, r.WithContext(ctx))

        // Record duration
        duration := time.Since(startTime).Seconds()

        // Record metrics with request ID
        httpRequestDuration.WithLabelValues(
            r.Method,
            r.URL.Path,
            requestID,
        ).Observe(duration)

        // Log request completion
        logger.Info().
            Float64("duration_seconds", duration).
            Msg("Request completed")
    })
}

// GetLogger extracts the logger from context
func GetLogger(ctx context.Context) zerolog.Logger {
    if logger, ok := ctx.Value("logger").(zerolog.Logger); ok {
        return logger
    }
    return log.Logger
}

// GetRequestID extracts the request ID from context
func GetRequestID(ctx context.Context) string {
    if requestID, ok := ctx.Value("request_id").(string); ok {
        return requestID
    }
    return ""
}

// GetCurrentSpan gets the current span from context
func GetCurrentSpan(ctx context.Context) trace.Span {
    return trace.SpanFromContext(ctx)
}

// Handler uses correlated logging, metrics, and tracing
func Handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    // Get logger and span from context
    logger := GetLogger(ctx)
    span := GetCurrentSpan(ctx)

    // Log and trace the operation
    logger.Info().Msg("Processing order")
    span.AddEvent("Processing order")

    // Perform business logic...

    w.Write([]byte("Order processed"))
}
```

This example demonstrates:

1. **Common identifier**: A request ID is generated and used across all telemetry
2. **Context propagation**: The context carries correlation IDs through the request
3. **Enriched logs**: Logs include trace and span IDs for correlation
4. **Labeled metrics**: Metrics include the request ID for correlation
5. **Annotated spans**: Spans include the request ID as an attribute

#### **Exemplars in Prometheus**

Prometheus supports exemplars, which allow you to link metrics to traces:

```go
package main

import (
    "context"
    "net/http"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
)

var (
    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "path"},
    )
)

// recordWithExemplar records a duration with trace information
func recordWithExemplar(ctx context.Context, duration float64, labels prometheus.Labels) {
    // Get trace and span IDs from context
    spanCtx := trace.SpanContextFromContext(ctx)
    if !spanCtx.IsValid() {
        // No valid span context, record without exemplar
        httpRequestDuration.With(labels).Observe(duration)
        return
    }

    // Create exemplar with trace ID
    exemplar := prometheus.Labels{
        "trace_id": spanCtx.TraceID().String(),
    }

    // Record observation with exemplar
    httpRequestDuration.With(labels).ObserveWithExemplar(duration, exemplar)
}

func TracedHandler(w http.ResponseWriter, r *http.Request) {
    // Start timing
    startTime := time.Now()

    // Get tracer and start span
    tracer := otel.Tracer("web-service")
    ctx, span := tracer.Start(r.Context(), "http_request")
    defer span.End()

    // Process the request...
    time.Sleep(100 * time.Millisecond)

    // Record duration with exemplar
    duration := time.Since(startTime).Seconds()
    recordWithExemplar(ctx, duration, prometheus.Labels{
        "method": r.Method,
        "path":   r.URL.Path,
    })

    w.Write([]byte("Hello, World!"))
}
```

Exemplars create direct links between metrics and traces, allowing you to quickly navigate from a metric to the traces that contributed to it.

### **28.3.2 Context Propagation**

Proper context propagation is essential for correlating telemetry data, especially in distributed systems. OpenTelemetry provides tools for propagating context across service boundaries:

```go
package main

import (
    "context"
    "net/http"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/propagation"
)

// TracePropagationMiddleware extracts and injects trace context
func TracePropagationMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Get the global propagator
        propagator := otel.GetTextMapPropagator()

        // Extract trace context from incoming request
        ctx := propagator.Extract(r.Context(), propagation.HeaderCarrier(r.Header))

        // Create a new span using the extracted context
        tracer := otel.Tracer("web-service")
        ctx, span := tracer.Start(ctx, "http_request")
        defer span.End()

        // Add response headers for trace propagation
        propagator.Inject(ctx, propagation.HeaderCarrier(w.Header()))

        // Call next handler with the traced context
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// MakeExternalRequest demonstrates context propagation in outgoing requests
func MakeExternalRequest(ctx context.Context, url string) (*http.Response, error) {
    // Create HTTP request
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }

    // Get the global propagator
    propagator := otel.GetTextMapPropagator()

    // Inject trace context into outgoing request headers
    propagator.Inject(ctx, propagation.HeaderCarrier(req.Header))

    // Make the request
    client := http.DefaultClient
    return client.Do(req)
}
```

This middleware extracts trace context from incoming requests and injects it into outgoing requests, ensuring that traces can be correlated across service boundaries.

### **28.3.3 Observability Platforms**

To make the most of your observability data, you'll need platforms that can collect, store, and visualize all three pillars. Several popular options include:

1. **Grafana + Loki + Tempo**: An open-source stack for metrics, logs, and traces
2. **Datadog**: A commercial platform with comprehensive observability features
3. **New Relic**: A commercial platform with APM and observability capabilities
4. **Honeycomb**: A commercial platform focused on high-cardinality observability
5. **Elastic Stack**: An open-source stack with capabilities for all three pillars

When choosing an observability platform, consider:

- Integration capabilities with your telemetry sources
- Query language power and flexibility
- Correlation features across pillars
- Retention policies and data storage costs
- Alerting and notification capabilities
- Ease of use and user interface

### **28.3.4 OpenTelemetry Collector**

The OpenTelemetry Collector provides a vendor-agnostic way to collect, process, and export telemetry data:

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024

  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200

exporters:
  prometheus:
    endpoint: 0.0.0.0:8889
    namespace: otel

  logging:
    loglevel: debug

  jaeger:
    endpoint: jaeger:14250
    tls:
      insecure: true

  loki:
    endpoint: http://loki:3100/loki/api/v1/push
    tenant_id: "otel"
    labels:
      resource:
        service.name: "service"
        service.namespace: "namespace"
      attributes:
        level: "severity"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [jaeger, logging]

    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus, logging]

    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [loki, logging]
```

The collector simplifies your observability architecture by:

- Providing a single agent for all telemetry data
- Supporting multiple receivers, processors, and exporters
- Enabling vendor-neutral data collection
- Reducing the number of outbound connections from your services
- Offering preprocessing capabilities like filtering and sampling

With the OpenTelemetry Collector, you can send all telemetry data to a single endpoint and let the collector route it to your observability platforms.

## **28.4 Advanced Observability Patterns**

As your applications grow in complexity, basic observability implementations may not be sufficient. This section explores advanced patterns that enhance the depth and usefulness of your observability data in Go applications.

### **28.4.1 Semantic Conventions**

Semantic conventions standardize how you name and structure your telemetry data, making it more consistent and easier to understand:

```go
package main

import (
    "context"
    "net/http"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
    semconv "go.opentelemetry.io/otel/semconv/v1.12.0"
    "go.opentelemetry.io/otel/trace"
)

// instrumentHTTP adds semantic conventions to HTTP operations
func instrumentHTTP(ctx context.Context, req *http.Request) (context.Context, trace.Span) {
    tracer := otel.Tracer("http-client")

    // Use semantic conventions for span name
    spanName := "HTTP " + req.Method

    // Start span with standard HTTP attributes
    ctx, span := tracer.Start(ctx, spanName)

    // Add HTTP attributes following OpenTelemetry semantic conventions
    span.SetAttributes(
        semconv.HTTPMethodKey.String(req.Method),
        semconv.HTTPURLKey.String(req.URL.String()),
        semconv.HTTPTargetKey.String(req.URL.Path),
        semconv.HTTPHostKey.String(req.Host),
        semconv.HTTPSchemeKey.String(req.URL.Scheme),
        semconv.HTTPUserAgentKey.String(req.UserAgent()),
    )

    return ctx, span
}

// instrumentDB adds semantic conventions to database operations
func instrumentDB(ctx context.Context, operation, query, dbName, dbSystem string) (context.Context, trace.Span) {
    tracer := otel.Tracer("db-client")

    // Use semantic conventions for span name
    spanName := dbSystem + " " + operation

    // Start span with standard DB attributes
    ctx, span := tracer.Start(ctx, spanName)

    // Add DB attributes following OpenTelemetry semantic conventions
    span.SetAttributes(
        semconv.DBSystemKey.String(dbSystem),
        semconv.DBNameKey.String(dbName),
        semconv.DBOperationKey.String(operation),
        semconv.DBStatementKey.String(query),
    )

    return ctx, span
}

// handleSuccess records a successful span completion
func handleSuccess(span trace.Span, result string) {
    // Set status and add result attribute
    span.SetStatus(codes.Ok, "")
    span.SetAttributes(attribute.String("result", result))
}

// handleError records error information on a span
func handleError(span trace.Span, err error) {
    // Set error status and record error
    span.SetStatus(codes.Error, err.Error())
    span.RecordError(err, trace.WithAttributes(
        attribute.String("error.type", "application_error"),
    ))
}
```

Following semantic conventions makes your telemetry data more consistent and interoperable across different services and observability platforms.

### **28.4.2 Feature Flags with Observability**

Combining feature flags with observability provides insights into feature usage and performance:

```go
package main

import (
    "context"
    "net/http"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/trace"
)

// FeatureFlag represents a feature flag configuration
type FeatureFlag struct {
    Name        string
    Description string
    Enabled     bool
    Percentage  int // Percentage of users who get the feature (0-100)
}

// FeatureFlagService manages feature flags
type FeatureFlagService struct {
    flags map[string]*FeatureFlag
    metrics *FeatureFlagMetrics
}

// FeatureFlagMetrics tracks feature flag usage
type FeatureFlagMetrics struct {
    flagEnabled *prometheus.CounterVec
    flagUsage   *prometheus.CounterVec
    flagLatency *prometheus.HistogramVec
}

// NewFeatureFlagMetrics creates metrics for tracking feature flags
func NewFeatureFlagMetrics() *FeatureFlagMetrics {
    return &FeatureFlagMetrics{
        flagEnabled: promauto.NewCounterVec(
            prometheus.CounterOpts{
                Name: "feature_flag_enabled_total",
                Help: "Total number of times a feature flag was enabled for a request",
            },
            []string{"flag_name", "user_id"},
        ),
        flagUsage: promauto.NewCounterVec(
            prometheus.CounterOpts{
                Name: "feature_flag_usage_total",
                Help: "Total number of times a feature flag was used",
            },
            []string{"flag_name", "user_id", "result"},
        ),
        flagLatency: promauto.NewHistogramVec(
            prometheus.HistogramOpts{
                Name:    "feature_flag_latency_seconds",
                Help:    "Latency of feature flag usage",
                Buckets: prometheus.DefBuckets,
            },
            []string{"flag_name"},
        ),
    }
}

// NewFeatureFlagService creates a new feature flag service
func NewFeatureFlagService() *FeatureFlagService {
    return &FeatureFlagService{
        flags:   make(map[string]*FeatureFlag),
        metrics: NewFeatureFlagMetrics(),
    }
}

// RegisterFlag adds a new feature flag
func (s *FeatureFlagService) RegisterFlag(flag *FeatureFlag) {
    s.flags[flag.Name] = flag
}

// IsEnabled checks if a feature flag is enabled for a user
func (s *FeatureFlagService) IsEnabled(ctx context.Context, flagName, userID string) bool {
    flag, exists := s.flags[flagName]
    if !exists {
        return false
    }

    // For simplicity, we're just using the global enabled state
    // In a real system, you'd apply user targeting rules

    // Record metrics
    if flag.Enabled {
        s.metrics.flagEnabled.WithLabelValues(flagName, userID).Inc()

        // Add feature flag to span if tracing is enabled
        if span := trace.SpanFromContext(ctx); span.IsRecording() {
            span.SetAttributes(
                attribute.Bool("feature."+flagName, true),
            )
        }
    }

    return flag.Enabled
}

// TrackFeatureUsage records feature usage with metrics and tracing
func (s *FeatureFlagService) TrackFeatureUsage(ctx context.Context, flagName, userID, result string) func() {
    startTime := time.Now()

    // Start feature span if tracing is enabled
    var span trace.Span
    if tracerProvider := otel.GetTracerProvider(); tracerProvider != nil {
        tracer := tracerProvider.Tracer("feature-flags")
        ctx, span = tracer.Start(ctx, "feature."+flagName)
        span.SetAttributes(
            attribute.String("feature.name", flagName),
            attribute.String("user.id", userID),
        )
        defer span.End()
    }

    // Return function to call when feature usage is complete
    return func() {
        // Calculate duration
        duration := time.Since(startTime).Seconds()

        // Record metrics
        s.metrics.flagUsage.WithLabelValues(flagName, userID, result).Inc()
        s.metrics.flagLatency.WithLabelValues(flagName).Observe(duration)

        // Add result to span if tracing is enabled
        if span != nil && span.IsRecording() {
            span.SetAttributes(
                attribute.String("feature.result", result),
                attribute.Float64("feature.duration_seconds", duration),
            )
        }
    }
}

// Example usage in HTTP handler
func featureHandler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    userID := r.Header.Get("X-User-ID")

    // Get feature flag service (would normally be injected)
    featureService := NewFeatureFlagService()
    featureService.RegisterFlag(&FeatureFlag{
        Name:        "new_checkout",
        Description: "New checkout flow",
        Enabled:     true,
    })

    // Check if feature is enabled
    if featureService.IsEnabled(ctx, "new_checkout", userID) {
        // Track feature usage
        done := featureService.TrackFeatureUsage(ctx, "new_checkout", userID, "started")

        // Use the new feature
        // ...

        // Complete tracking with result
        done()

        w.Write([]byte("New checkout used"))
    } else {
        // Use old checkout
        w.Write([]byte("Old checkout used"))
    }
}
```

This pattern allows you to:

1. **Track feature usage**: See how often features are used
2. **Measure performance impact**: Compare latency between feature variants
3. **Debug issues**: Connect feature flag decisions to traces
4. **Monitor rollouts**: Watch for problems as features are enabled

### **28.4.3 SLO Monitoring**

Service Level Objectives (SLOs) define reliability targets for your services. Implementing SLO monitoring helps ensure you're meeting user expectations:

```go
package main

import (
    "fmt"
    "net/http"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

// SLOConfig defines an SLO
type SLOConfig struct {
    Name        string
    Description string
    Target      float64   // e.g., 0.999 for 99.9% availability
    Window      time.Duration // e.g., 30 * 24 * time.Hour for 30 days
}

// SLOTracker tracks SLO metrics
type SLOTracker struct {
    config  SLOConfig
    total   *prometheus.CounterVec
    success *prometheus.CounterVec
    latency *prometheus.HistogramVec
    budget  *prometheus.GaugeVec
}

// NewSLOTracker creates a new SLO tracker
func NewSLOTracker(config SLOConfig) *SLOTracker {
    errorBudget := 1 - config.Target

    total := promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: fmt.Sprintf("slo_%s_total", config.Name),
            Help: fmt.Sprintf("Total requests for %s SLO", config.Name),
        },
        []string{"service", "endpoint"},
    )

    success := promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: fmt.Sprintf("slo_%s_success", config.Name),
            Help: fmt.Sprintf("Successful requests for %s SLO", config.Name),
        },
        []string{"service", "endpoint"},
    )

    latency := promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: fmt.Sprintf("slo_%s_latency", config.Name),
            Help: fmt.Sprintf("Request latency for %s SLO", config.Name),
            // Set buckets based on SLO latency targets
            Buckets: []float64{0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10},
        },
        []string{"service", "endpoint"},
    )

    budget := promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Name: fmt.Sprintf("slo_%s_error_budget_remaining", config.Name),
            Help: fmt.Sprintf("Remaining error budget for %s SLO", config.Name),
        },
        []string{"service", "window"},
    )

    // Initialize budget
    budget.WithLabelValues("api", config.Window.String()).Set(errorBudget)

    return &SLOTracker{
        config:  config,
        total:   total,
        success: success,
        latency: latency,
        budget:  budget,
    }
}

// TrackRequest records a request for SLO tracking
func (t *SLOTracker) TrackRequest(service, endpoint string, statusCode int, duration time.Duration) {
    // Increment total counter
    t.total.WithLabelValues(service, endpoint).Inc()

    // Check if request was successful (based on status code)
    isSuccess := statusCode >= 200 && statusCode < 500
    if isSuccess {
        t.success.WithLabelValues(service, endpoint).Inc()
    }

    // Record latency
    t.latency.WithLabelValues(service, endpoint).Observe(duration.Seconds())

    // Update error budget (normally done in a separate process based on the success rate)
    // This is simplified; in production, you'd calculate this over the full window
    if !isSuccess {
        t.budget.WithLabelValues(service, t.config.Window.String()).Dec()
    }
}

// SLOMiddleware creates middleware that tracks requests for SLO monitoring
func SLOMiddleware(tracker *SLOTracker, service string) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            startTime := time.Now()

            // Create response writer wrapper to capture status code
            rww := NewResponseWriterWrapper(w)

            // Call next handler
            next.ServeHTTP(rww, r)

            // Calculate duration
            duration := time.Since(startTime)

            // Record for SLO tracking
            tracker.TrackRequest(service, r.URL.Path, rww.statusCode, duration)
        })
    }
}

// ResponseWriterWrapper captures status code (reused from earlier example)
type ResponseWriterWrapper struct {
    http.ResponseWriter
    statusCode int
}

// NewResponseWriterWrapper creates a new wrapper
func NewResponseWriterWrapper(w http.ResponseWriter) *ResponseWriterWrapper {
    return &ResponseWriterWrapper{w, http.StatusOK}
}

// WriteHeader captures the status code
func (rww *ResponseWriterWrapper) WriteHeader(code int) {
    rww.statusCode = code
    rww.ResponseWriter.WriteHeader(code)
}

func main() {
    // Create SLO tracker for availability
    availabilitySLO := NewSLOTracker(SLOConfig{
        Name:        "availability",
        Description: "API availability",
        Target:      0.999, // 99.9% availability
        Window:      30 * 24 * time.Hour, // 30 days
    })

    // Create SLO tracker for latency
    latencySLO := NewSLOTracker(SLOConfig{
        Name:        "latency",
        Description: "API latency under 100ms",
        Target:      0.95, // 95% of requests under 100ms
        Window:      30 * 24 * time.Hour, // 30 days
    })

    // Set up HTTP server with SLO middleware
    http.Handle("/api/", SLOMiddleware(availabilitySLO, "api")(
        SLOMiddleware(latencySLO, "api")(
            http.HandlerFunc(apiHandler),
        ),
    ))

    // Expose Prometheus metrics endpoint
    http.Handle("/metrics", promhttp.Handler())

    // Start server
    http.ListenAndServe(":8080", nil)
}

func apiHandler(w http.ResponseWriter, r *http.Request) {
    // Simulate API logic
    time.Sleep(50 * time.Millisecond)
    w.Write([]byte("API response"))
}
```

With this SLO monitoring, you can:

1. **Track reliability metrics**: Monitor availability and latency against targets
2. **Calculate error budgets**: See how much room you have for taking risks
3. **Create SLO-based alerts**: Alert when you're at risk of missing SLOs
4. **Prioritize reliability work**: Focus on components that are affecting SLOs

### **28.4.4 Health Checks and Readiness Probes**

Exposing health and readiness endpoints helps monitoring systems and orchestrators understand your application's status:

```go
package main

import (
    "encoding/json"
    "net/http"
    "sync"
    "time"

    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
)

// HealthStatus represents the status of a component
type HealthStatus string

const (
    StatusUp      HealthStatus = "UP"
    StatusDown    HealthStatus = "DOWN"
    StatusDegraded HealthStatus = "DEGRADED"
)

// ComponentHealth represents the health of a component
type ComponentHealth struct {
    Status      HealthStatus `json:"status"`
    Description string       `json:"description,omitempty"`
    Error       string       `json:"error,omitempty"`
    LastChecked time.Time    `json:"lastChecked"`
}

// HealthResponse represents the overall health check response
type HealthResponse struct {
    Status     HealthStatus                `json:"status"`
    Components map[string]ComponentHealth  `json:"components"`
    Timestamp  time.Time                   `json:"timestamp"`
    Version    string                      `json:"version"`
}

// HealthChecker manages health checks
type HealthChecker struct {
    components map[string]ComponentHealth
    checks     map[string]func() ComponentHealth
    mutex      sync.RWMutex
    version    string

    // Metrics
    healthStatus *prometheus.GaugeVec
    checkDuration *prometheus.HistogramVec
}

// NewHealthChecker creates a new health checker
func NewHealthChecker(version string) *HealthChecker {
    healthStatus := promauto.NewGaugeVec(
        prometheus.GaugeOpts{
            Name: "component_health_status",
            Help: "Health status of components (0=DOWN, 1=DEGRADED, 2=UP)",
        },
        []string{"component"},
    )

    checkDuration := promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "health_check_duration_seconds",
            Help:    "Duration of health checks",
            Buckets: []float64{0.001, 0.005, 0.01, 0.05, 0.1, 0.5, 1},
        },
        []string{"component"},
    )

    return &HealthChecker{
        components:    make(map[string]ComponentHealth),
        checks:        make(map[string]func() ComponentHealth),
        version:       version,
        healthStatus:  healthStatus,
        checkDuration: checkDuration,
    }
}

// RegisterCheck adds a health check
func (h *HealthChecker) RegisterCheck(name string, check func() ComponentHealth) {
    h.mutex.Lock()
    defer h.mutex.Unlock()

    h.checks[name] = check
    h.components[name] = ComponentHealth{
        Status:      StatusDown,
        Description: "Not checked yet",
        LastChecked: time.Time{},
    }

    // Initialize metric
    h.healthStatus.WithLabelValues(name).Set(0)
}

// RunChecks executes all health checks
func (h *HealthChecker) RunChecks() {
    h.mutex.Lock()
    defer h.mutex.Unlock()

    for name, check := range h.checks {
        startTime := time.Now()

        // Run the check
        result := check()
        result.LastChecked = time.Now()

        // Calculate duration
        duration := time.Since(startTime).Seconds()

        // Update component status
        h.components[name] = result

        // Update metrics
        var statusValue float64
        switch result.Status {
        case StatusUp:
            statusValue = 2
        case StatusDegraded:
            statusValue = 1
        case StatusDown:
            statusValue = 0
        }

        h.healthStatus.WithLabelValues(name).Set(statusValue)
        h.checkDuration.WithLabelValues(name).Observe(duration)
    }
}

// GetHealth returns the current health status
func (h *HealthChecker) GetHealth() HealthResponse {
    h.mutex.RLock()
    defer h.mutex.RUnlock()

    // Calculate overall status
    overallStatus := StatusUp

    for _, comp := range h.components {
        if comp.Status == StatusDown {
            overallStatus = StatusDown
            break
        } else if comp.Status == StatusDegraded {
            overallStatus = StatusDegraded
        }
    }

    // Create copy of components to avoid race conditions
    componentsCopy := make(map[string]ComponentHealth)
    for k, v := range h.components {
        componentsCopy[k] = v
    }

    return HealthResponse{
        Status:     overallStatus,
        Components: componentsCopy,
        Timestamp:  time.Now(),
        Version:    h.version,
    }
}

// StartHealthCheckLoop runs health checks periodically
func (h *HealthChecker) StartHealthCheckLoop(interval time.Duration) {
    ticker := time.NewTicker(interval)

    go func() {
        // Run once immediately
        h.RunChecks()

        for range ticker.C {
            h.RunChecks()
        }
    }()
}

// HealthHandler creates an HTTP handler for health checks
func (h *HealthChecker) HealthHandler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        health := h.GetHealth()

        w.Header().Set("Content-Type", "application/json")

        // Set appropriate status code based on health
        if health.Status == StatusDown {
            w.WriteHeader(http.StatusServiceUnavailable)
        } else if health.Status == StatusDegraded {
            w.WriteHeader(http.StatusOK) // Still 200, but indicates degraded in body
        } else {
            w.WriteHeader(http.StatusOK)
        }

        // Write response
        json.NewEncoder(w).Encode(health)
    }
}

// LivenessHandler creates an HTTP handler for liveness checks
func (h *HealthChecker) LivenessHandler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Liveness checks only verify that the application is running
        // and can respond to HTTP requests
        w.WriteHeader(http.StatusOK)
        w.Write([]byte("OK"))
    }
}

// ReadinessHandler creates an HTTP handler for readiness checks
func (h *HealthChecker) ReadinessHandler() http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        health := h.GetHealth()

        // Readiness checks verify that the application is ready to serve requests
        if health.Status == StatusDown {
            w.WriteHeader(http.StatusServiceUnavailable)
            w.Write([]byte("NOT READY"))
        } else {
            w.WriteHeader(http.StatusOK)
            w.Write([]byte("READY"))
        }
    }
}

func main() {
    // Create health checker
    health := NewHealthChecker("1.0.0")

    // Register database health check
    health.RegisterCheck("database", func() ComponentHealth {
        // Simulate database check
        // In a real application, you'd check the actual database connection
        if time.Now().Second()%10 == 0 {
            return ComponentHealth{
                Status:      StatusDegraded,
                Description: "Database connection pool nearing capacity",
            }
        }

        return ComponentHealth{
            Status:      StatusUp,
            Description: "Database connection successful",
        }
    })

    // Register Redis health check
    health.RegisterCheck("redis", func() ComponentHealth {
        // Simulate Redis check
        return ComponentHealth{
            Status:      StatusUp,
            Description: "Redis connection successful",
        }
    })

    // Start health check loop
    health.StartHealthCheckLoop(15 * time.Second)

    // Set up HTTP server
    http.HandleFunc("/health", health.HealthHandler())
    http.HandleFunc("/livez", health.LivenessHandler())
    http.HandleFunc("/readyz", health.ReadinessHandler())

    // Start server
    http.ListenAndServe(":8080", nil)
}
```

This health check system provides:

1. **Component-level health**: Track the health of individual components
2. **Liveness and readiness**: Differentiate between "alive" and "ready to serve"
3. **Health metrics**: Monitor health status as metrics
4. **Detailed health reports**: Provide rich health information for debugging

### **28.4.5 Continuous Profiling**

Continuous profiling helps identify performance bottlenecks in production:

```go
package main

import (
    "net/http"
    _ "net/http/pprof" // Import for side effects
    "os"
    "runtime"
    "runtime/pprof"
    "time"

    "github.com/google/pprof/profile"
)

// ProfileScheduler periodically captures profiles
type ProfileScheduler struct {
    interval time.Duration
    duration time.Duration
    dir      string
}

// NewProfileScheduler creates a new profile scheduler
func NewProfileScheduler(interval, duration time.Duration, dir string) *ProfileScheduler {
    // Create directory if it doesn't exist
    if err := os.MkdirAll(dir, 0755); err != nil {
        panic(err)
    }

    return &ProfileScheduler{
        interval: interval,
        duration: duration,
        dir:      dir,
    }
}

// Start begins the profiling schedule
func (p *ProfileScheduler) Start() {
    // Start CPU profiling ticker
    go p.scheduleCPUProfile()

    // Start heap profiling ticker
    go p.scheduleHeapProfile()

    // Start goroutine profiling ticker
    go p.scheduleGoroutineProfile()
}

// scheduleCPUProfile captures CPU profiles on a schedule
func (p *ProfileScheduler) scheduleCPUProfile() {
    ticker := time.NewTicker(p.interval)
    defer ticker.Stop()

    for range ticker.C {
        fileName := p.dir + "/cpu-" + time.Now().Format("20060102-150405") + ".pprof"
        f, err := os.Create(fileName)
        if err != nil {
            continue
        }

        // Start CPU profiling
        if err := pprof.StartCPUProfile(f); err != nil {
            f.Close()
            continue
        }

        // Profile for the specified duration
        time.Sleep(p.duration)

        // Stop profiling
        pprof.StopCPUProfile()
        f.Close()
    }
}

// scheduleHeapProfile captures heap profiles on a schedule
func (p *ProfileScheduler) scheduleHeapProfile() {
    ticker := time.NewTicker(p.interval)
    defer ticker.Stop()

    for range ticker.C {
        fileName := p.dir + "/heap-" + time.Now().Format("20060102-150405") + ".pprof"
        f, err := os.Create(fileName)
        if err != nil {
            continue
        }

        // Force garbage collection to get accurate memory usage
        runtime.GC()

        // Write heap profile
        if err := pprof.WriteHeapProfile(f); err != nil {
            f.Close()
            continue
        }

        f.Close()
    }
}

// scheduleGoroutineProfile captures goroutine profiles on a schedule
func (p *ProfileScheduler) scheduleGoroutineProfile() {
    ticker := time.NewTicker(p.interval)
    defer ticker.Stop()

    for range ticker.C {
        fileName := p.dir + "/goroutine-" + time.Now().Format("20060102-150405") + ".pprof"
        f, err := os.Create(fileName)
        if err != nil {
            continue
        }

        // Get goroutine profile
        if err := pprof.Lookup("goroutine").WriteTo(f, 0); err != nil {
            f.Close()
            continue
        }

        f.Close()
    }
}

func main() {
    // Create profile scheduler
    scheduler := NewProfileScheduler(
        30*time.Minute,  // Capture profiles every 30 minutes
        30*time.Second,  // Profile for 30 seconds
        "./profiles",    // Store profiles in this directory
    )

    // Start profile scheduler
    scheduler.Start()

    // Enable pprof HTTP endpoint for on-demand profiling
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()

    // Your application code here
    // ...

    // Keep the application running
    select {}
}
```

Continuous profiling provides:

1. **Baseline performance data**: Track normal behavior over time
2. **Performance regression detection**: Identify when performance changes
3. **Production insights**: See real-world performance patterns
4. **Resource optimization**: Find opportunities to reduce resource usage

These advanced patterns build on the basic observability techniques, providing deeper insights and more proactive monitoring capabilities for your Go applications.

## **28.5 Conclusion**

Observability-Driven Development represents a fundamental shift in how we approach software engineering. By designing systems with observability in mind from the start, we create applications that are easier to understand, debug, and maintain. In Go, this approach is particularly powerful, thanks to the language's strong standard library, excellent performance characteristics, and growing ecosystem of observability tools.

Throughout this chapter, we've explored how to implement the three pillars of observability—logs, metrics, and traces—in Go applications. We've seen how to integrate these pillars to create a comprehensive observability solution, and we've examined advanced patterns that enhance the depth and usefulness of observability data.

As your Go applications grow in complexity, especially in distributed environments, the investment in observability will pay increasing dividends. The ability to quickly identify and diagnose issues, understand performance bottlenecks, and make data-driven decisions about your system's evolution becomes invaluable.

Key takeaways from this chapter include:

1. **Start with observability in mind**: Instrument your code from the beginning, rather than adding observability as an afterthought.

2. **Use all three pillars**: Logs, metrics, and traces each provide unique insights; the most powerful observability solutions use all three.

3. **Correlate data across pillars**: Connecting logs, metrics, and traces allows for faster, more effective debugging and analysis.

4. **Standardize your approach**: Use consistent naming, labeling, and conventions across your observability data.

5. **Focus on business outcomes**: Track not just technical metrics, but also business-relevant indicators that tie back to user experience.

6. **Use context propagation**: Properly propagate context through your system to maintain correlation between services.

7. **Continuously improve**: Use observability data to drive ongoing improvements to your system.

By applying these principles and implementing the patterns discussed in this chapter, you'll be well on your way to creating observable Go applications that are reliable, performant, and easier to evolve over time.

Remember that observability is not a destination but a journey. As your system evolves, so too should your observability practices. Continuously evaluate and improve your approach based on the challenges you face and the insights you gain.

The future of software engineering lies in systems that are not just built to run, but built to be understood. With Go's simplicity, performance, and growing observability ecosystem, you're well-positioned to create observable systems that stand the test of time.
