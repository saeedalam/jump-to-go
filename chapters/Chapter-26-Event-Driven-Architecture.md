# **Chapter 26: Event-Driven Architecture with Go**

## **26.1 Introduction to Event-Driven Architecture**

Event-Driven Architecture (EDA) represents a powerful paradigm for building distributed, scalable, and loosely coupled systems. At its core, EDA revolves around the production, detection, consumption, and reaction to events that represent significant changes in state.

Go is exceptionally well-suited for implementing event-driven systems due to:

1. **Goroutines and Channels**: Native concurrency primitives for processing events in parallel
2. **Low Memory Footprint**: Efficient handling of high-volume event streams
3. **Fast Startup Time**: Ideal for serverless and event-triggered workloads
4. **Strong Typing**: Helps maintain consistency in event schemas
5. **Rich Ecosystem**: Robust libraries for messaging systems, event sourcing, and streaming

In this chapter, we'll explore how to design, implement, and scale event-driven systems in Go, combining theoretical concepts with practical, production-ready code examples.

### **26.1.1 Core Concepts of Event-Driven Architecture**

Before diving into implementation details, let's establish the key concepts that underpin event-driven systems:

#### **Events**

An event is an immutable record of something that has happened in the past. Events typically include:

- A unique identifier
- Event type or name
- Timestamp
- Payload data
- Metadata (correlation IDs, causation IDs, etc.)

Events represent facts, not commands or requests. For example, "UserRegistered" is an event, while "RegisterUser" is a command.

#### **Event Producers**

Event producers are components that generate events when something notable happens. They emit events without knowing who will consume them or how they will be processed. This decoupling is a fundamental strength of EDA.

#### **Event Consumers**

Event consumers listen for and react to events. A single event may have multiple consumers, each performing different actions in response. Consumers can be synchronous or asynchronous.

#### **Event Channels**

Event channels are the communication pathways between producers and consumers. These can be:

- Direct in-memory channels (like Go's channels)
- Message queues (RabbitMQ, Amazon SQS)
- Event brokers (Kafka, NATS)
- Event stores (EventStoreDB, Apache Pulsar)

### **26.1.2 Patterns in Event-Driven Architecture**

Several established patterns exist within the event-driven paradigm:

#### **Publish-Subscribe (Pub/Sub)**

The most fundamental pattern where producers publish events to channels, and consumers subscribe to those channels to receive events.

#### **Event Sourcing**

Rather than storing the current state, event sourcing persists the full sequence of events. The current state can be reconstructed by replaying these events.

#### **Command Query Responsibility Segregation (CQRS)**

Separates write operations (commands) from read operations (queries), often using events to synchronize between specialized write and read models.

#### **Saga Pattern**

Coordinates distributed transactions across multiple services using a sequence of events and compensating actions.

#### **Event-Driven Microservices**

Microservices that communicate primarily through events rather than direct API calls.

### **26.1.3 Benefits and Challenges**

Event-driven architectures offer numerous advantages:

**Benefits:**

- **Loose Coupling**: Services don't need direct knowledge of each other
- **Scalability**: Easy to scale components independently
- **Resilience**: Failures in one component don't necessarily cascade
- **Flexibility**: Easy to add new components that react to existing events
- **Auditability**: Events provide a natural audit log

**Challenges:**

- **Eventual Consistency**: Systems may be temporarily inconsistent
- **Complexity**: Reasoning about asynchronous systems can be harder
- **Debugging**: Tracing through event chains can be challenging
- **Event Schema Evolution**: Managing changes to event formats
- **Ordering Guarantees**: Ensuring correct event sequence when needed

Throughout this chapter, we'll address these challenges with practical Go-based solutions.

## **26.2 Building Blocks of Event-Driven Systems in Go**

Let's implement the fundamental components needed for event-driven architectures in Go. We'll start with in-memory implementations and then expand to distributed solutions.

### **26.2.1 Defining Events**

An effective event model starts with a well-defined event structure:

```go
// event/event.go
package event

import (
	"time"

	"github.com/google/uuid"
)

// Event represents a domain event that has occurred in the system
type Event struct {
	ID            string                 `json:"id"`
	Type          string                 `json:"type"`
	Source        string                 `json:"source"`
	Time          time.Time              `json:"time"`
	Data          map[string]interface{} `json:"data"`
	DataVersion   string                 `json:"data_version"`
	Metadata      map[string]string      `json:"metadata,omitempty"`
	CorrelationID string                 `json:"correlation_id,omitempty"`
	CausationID   string                 `json:"causation_id,omitempty"`
}

// NewEvent creates a new event with automatic ID and timestamp
func NewEvent(eventType string, source string, data map[string]interface{}, dataVersion string) *Event {
	return &Event{
		ID:          uuid.New().String(),
		Type:        eventType,
		Source:      source,
		Time:        time.Now().UTC(),
		Data:        data,
		DataVersion: dataVersion,
		Metadata:    make(map[string]string),
	}
}

// WithCorrelation adds correlation tracking information to an event
func (e *Event) WithCorrelation(correlationID, causationID string) *Event {
	e.CorrelationID = correlationID
	e.CausationID = causationID
	return e
}

// AddMetadata adds a key-value pair to the event metadata
func (e *Event) AddMetadata(key, value string) *Event {
	if e.Metadata == nil {
		e.Metadata = make(map[string]string)
	}
	e.Metadata[key] = value
	return e
}
```

This design includes essential fields and helps establish good practices like:

- Using UUIDs for event identification
- Standardizing on UTC timestamps
- Including source information
- Supporting event versioning for schema evolution
- Including correlation IDs for distributed tracing

### **26.2.2 Event Bus Implementation**

An event bus enables communication between components through events. Let's create a simple in-memory implementation:

```go
// bus/memory.go
package bus

import (
	"context"
	"fmt"
	"sync"

	"github.com/yourorg/eventsystem/event"
)

// Handler defines the interface for event handlers
type Handler interface {
	HandleEvent(ctx context.Context, event *event.Event) error
}

// HandlerFunc is a function type that implements the Handler interface
type HandlerFunc func(ctx context.Context, event *event.Event) error

// HandleEvent calls the handler function
func (f HandlerFunc) HandleEvent(ctx context.Context, event *event.Event) error {
	return f(ctx, event)
}

// MemoryBus provides an in-memory implementation of an event bus
type MemoryBus struct {
	handlers     map[string][]Handler
	handlersMu   sync.RWMutex
	middlewares  []Middleware
	middlewareMu sync.RWMutex
}

// Middleware is a function that wraps event handling
type Middleware func(Handler) Handler

// NewMemoryBus creates a new in-memory event bus
func NewMemoryBus() *MemoryBus {
	return &MemoryBus{
		handlers: make(map[string][]Handler),
	}
}

// Subscribe registers a handler for a specific event type
func (b *MemoryBus) Subscribe(eventType string, handler Handler) error {
	b.handlersMu.Lock()
	defer b.handlersMu.Unlock()

	if _, ok := b.handlers[eventType]; !ok {
		b.handlers[eventType] = make([]Handler, 0)
	}
	b.handlers[eventType] = append(b.handlers[eventType], handler)
	return nil
}

// SubscribeFunc registers a handler function for a specific event type
func (b *MemoryBus) SubscribeFunc(eventType string, handlerFunc func(ctx context.Context, event *event.Event) error) error {
	return b.Subscribe(eventType, HandlerFunc(handlerFunc))
}

// Publish sends an event to all subscribed handlers
func (b *MemoryBus) Publish(ctx context.Context, event *event.Event) error {
	b.handlersMu.RLock()
	handlers, ok := b.handlers[event.Type]
	b.handlersMu.RUnlock()

	if !ok {
		return nil // No handlers for this event type
	}

	var wg sync.WaitGroup
	errs := make(chan error, len(handlers))

	for _, h := range handlers {
		wg.Add(1)

		// Apply middlewares
		handler := h
		for i := len(b.middlewares) - 1; i >= 0; i-- {
			handler = b.middlewares[i](handler)
		}

		go func(handler Handler) {
			defer wg.Done()
			if err := handler.HandleEvent(ctx, event); err != nil {
				errs <- err
			}
		}(handler)
	}

	// Wait for all handlers to complete
	wg.Wait()
	close(errs)

	// Collect errors
	var errMsgs []string
	for err := range errs {
		errMsgs = append(errMsgs, err.Error())
	}

	if len(errMsgs) > 0 {
		return fmt.Errorf("errors while processing event: %v", errMsgs)
	}

	return nil
}

// Use adds middleware to the event processing pipeline
func (b *MemoryBus) Use(middleware Middleware) {
	b.middlewareMu.Lock()
	defer b.middlewareMu.Unlock()
	b.middlewares = append(b.middlewares, middleware)
}
```

This in-memory event bus provides:

- Type-based event routing
- Concurrent processing of events
- Support for middleware (for cross-cutting concerns)
- Error aggregation
- A simple but powerful API

### **26.2.3 Creating Event Middleware**

Middleware allows us to add cross-cutting concerns like logging, metrics, and error handling:

```go
// middleware/logging.go
package middleware

import (
	"context"
	"log"
	"time"

	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/event"
)

// Logging creates middleware that logs event processing
func Logging() bus.Middleware {
	return func(next bus.Handler) bus.Handler {
		return bus.HandlerFunc(func(ctx context.Context, evt *event.Event) error {
			start := time.Now()
			log.Printf("Processing event: %s (id: %s, source: %s)", evt.Type, evt.ID, evt.Source)

			err := next.HandleEvent(ctx, evt)

			elapsed := time.Since(start)
			if err != nil {
				log.Printf("Error processing event %s (id: %s): %v [%s]", evt.Type, evt.ID, err, elapsed)
			} else {
				log.Printf("Successfully processed event %s (id: %s) [%s]", evt.Type, evt.ID, elapsed)
			}

			return err
		})
	}
}

// middleware/retry.go
package middleware

import (
	"context"
	"time"

	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/event"
)

// RetryOptions configures the retry behavior
type RetryOptions struct {
	MaxRetries  int
	InitialWait time.Duration
	MaxWait     time.Duration
	Multiplier  float64
}

// DefaultRetryOptions provides sensible defaults
func DefaultRetryOptions() RetryOptions {
	return RetryOptions{
		MaxRetries:  3,
		InitialWait: 100 * time.Millisecond,
		MaxWait:     2 * time.Second,
		Multiplier:  2.0,
	}
}

// Retry creates middleware that retries failed event processing
func Retry(opts RetryOptions) bus.Middleware {
	return func(next bus.Handler) bus.Handler {
		return bus.HandlerFunc(func(ctx context.Context, evt *event.Event) error {
			var err error
			wait := opts.InitialWait

			for attempt := 0; attempt <= opts.MaxRetries; attempt++ {
				// First attempt or retry
				if attempt > 0 {
					select {
					case <-ctx.Done():
						return ctx.Err()
					case <-time.After(wait):
						// Exponential backoff with jitter
						wait = time.Duration(float64(wait) * opts.Multiplier)
						if wait > opts.MaxWait {
							wait = opts.MaxWait
						}
					}
				}

				err = next.HandleEvent(ctx, evt)
				if err == nil {
					return nil // Success
				}
			}

			return err // Return the last error after all retries
		})
	}
}
```

### **26.2.4 Typed Events for Better Type Safety**

While a generic event structure is flexible, typed events provide better compile-time safety:

```go
// domain/user/events.go
package user

import (
	"time"

	"github.com/google/uuid"
	"github.com/yourorg/eventsystem/event"
)

const (
	EventTypeUserCreated = "user.created"
	EventTypeUserUpdated = "user.updated"
	EventTypeUserDeleted = "user.deleted"

	EventVersion = "1.0.0"
)

// User represents a user in the system
type User struct {
	ID        string    `json:"id"`
	Email     string    `json:"email"`
	Name      string    `json:"name"`
	CreatedAt time.Time `json:"created_at"`
	UpdatedAt time.Time `json:"updated_at"`
}

// UserCreatedEvent represents a user creation event
type UserCreatedEvent struct {
	User User `json:"user"`
}

// ToEvent converts the typed event to a generic event
func (e UserCreatedEvent) ToEvent(source string) *event.Event {
	return event.NewEvent(
		EventTypeUserCreated,
		source,
		map[string]interface{}{
			"user": e.User,
		},
		EventVersion,
	)
}

// FromEvent creates a typed event from a generic event
func UserCreatedEventFromEvent(evt *event.Event) (UserCreatedEvent, error) {
	userData, ok := evt.Data["user"].(map[string]interface{})
	if !ok {
		return UserCreatedEvent{}, fmt.Errorf("invalid event data format")
	}

	// In a real implementation, you would use proper JSON unmarshaling
	// This is simplified for demonstration
	user := User{
		ID:        userData["id"].(string),
		Email:     userData["email"].(string),
		Name:      userData["name"].(string),
		CreatedAt: time.Now(), // Would parse from the event
		UpdatedAt: time.Now(), // Would parse from the event
	}

	return UserCreatedEvent{User: user}, nil
}

// NewUserCreatedEvent creates a new user created event
func NewUserCreatedEvent(user User) UserCreatedEvent {
	return UserCreatedEvent{User: user}
}

// UserUpdatedEvent and UserDeletedEvent would follow similar patterns
```

### **26.2.5 Putting It All Together**

Here's how to use these components together:

```go
// main.go
package main

import (
	"context"
	"log"
	"time"

	"github.com/google/uuid"
	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/domain/user"
	"github.com/yourorg/eventsystem/middleware"
)

func main() {
	// Create the event bus
	eventBus := bus.NewMemoryBus()

	// Add middleware
	eventBus.Use(middleware.Logging())
	eventBus.Use(middleware.Retry(middleware.DefaultRetryOptions()))

	// Subscribe to user events
	eventBus.SubscribeFunc(user.EventTypeUserCreated, handleUserCreated)
	eventBus.SubscribeFunc(user.EventTypeUserUpdated, handleUserUpdated)

	// Create a user
	newUser := user.User{
		ID:        uuid.New().String(),
		Email:     "jane@example.com",
		Name:      "Jane Doe",
		CreatedAt: time.Now().UTC(),
		UpdatedAt: time.Now().UTC(),
	}

	// Create and publish the event
	evt := user.NewUserCreatedEvent(newUser).ToEvent("user-service")

	ctx := context.Background()
	if err := eventBus.Publish(ctx, evt); err != nil {
		log.Fatalf("Failed to publish event: %v", err)
	}

	// Wait for handlers to complete (in a real app, you'd have a proper shutdown mechanism)
	time.Sleep(time.Second)
}

func handleUserCreated(ctx context.Context, evt *event.Event) error {
	userEvt, err := user.UserCreatedEventFromEvent(evt)
	if err != nil {
		return err
	}

	log.Printf("User created: %s (%s)", userEvt.User.Name, userEvt.User.ID)
	// In a real app, you would do something with this event

	return nil
}

func handleUserUpdated(ctx context.Context, evt *event.Event) error {
	// Similar implementation to handleUserCreated
	return nil
}
```

This foundation provides a solid starting point for building event-driven systems in Go. In the following sections, we'll expand on these concepts to build more sophisticated distributed event systems.

## **26.3 Implementing Event Sourcing in Go**

Event Sourcing is a powerful pattern that stores state changes as a sequence of events rather than just the current state. This approach provides a complete audit trail, enables temporal queries (what was the state at a specific time?), and forms the foundation for CQRS architectures.

### **26.3.1 Core Components of Event Sourcing**

Let's start by defining the key interfaces for our event sourcing implementation:

```go
// eventsourcing/store.go
package eventsourcing

import (
	"context"

	"github.com/yourorg/eventsystem/event"
)

// EventStore defines the interface for storing and retrieving events
type EventStore interface {
	// Save persists events to the store
	Save(ctx context.Context, streamID string, events []*event.Event, expectedVersion int) error

	// Load retrieves events for a specific aggregate
	Load(ctx context.Context, streamID string, fromVersion int) ([]*event.Event, error)

	// LoadByType retrieves events of a specific type
	LoadByType(ctx context.Context, eventTypes []string, fromPosition int, limit int) ([]*event.Event, error)
}

// Aggregate represents an entity that is reconstituted from events
type Aggregate interface {
	// GetID returns the aggregate's unique identifier
	GetID() string

	// GetVersion returns the current version of the aggregate
	GetVersion() int

	// Apply applies an event to the aggregate, updating its state
	Apply(event *event.Event) error

	// GetUncommittedEvents returns events that have been applied but not yet committed
	GetUncommittedEvents() []*event.Event

	// ClearUncommittedEvents clears the list of uncommitted events
	ClearUncommittedEvents()
}

// Repository provides an abstraction for loading and saving aggregates
type Repository interface {
	// Load retrieves an aggregate by ID
	Load(ctx context.Context, aggregateID string) (Aggregate, error)

	// Save persists an aggregate's uncommitted events
	Save(ctx context.Context, aggregate Aggregate) error
}
```

### **26.3.2 In-Memory Event Store Implementation**

Let's implement an in-memory event store for development and testing:

```go
// eventsourcing/memory_store.go
package eventsourcing

import (
	"context"
	"fmt"
	"sort"
	"sync"

	"github.com/yourorg/eventsystem/event"
)

// MemoryEventStore provides an in-memory implementation of the EventStore interface
type MemoryEventStore struct {
	streams      map[string][]*event.Event
	streamsMutex sync.RWMutex
	allEvents    []*event.Event
	allMutex     sync.RWMutex
}

// NewMemoryEventStore creates a new in-memory event store
func NewMemoryEventStore() *MemoryEventStore {
	return &MemoryEventStore{
		streams:   make(map[string][]*event.Event),
		allEvents: make([]*event.Event, 0),
	}
}

// Save stores events for an aggregate
func (s *MemoryEventStore) Save(ctx context.Context, streamID string, events []*event.Event, expectedVersion int) error {
	if len(events) == 0 {
		return nil
	}

	s.streamsMutex.Lock()
	defer s.streamsMutex.Unlock()

	// Get current stream
	stream, exists := s.streams[streamID]
	if !exists {
		if expectedVersion > 0 {
			return fmt.Errorf("expected version %d for stream %s, but stream doesn't exist", expectedVersion, streamID)
		}
		stream = make([]*event.Event, 0)
	} else if len(stream) != expectedVersion {
		return fmt.Errorf("expected version %d for stream %s, got %d", expectedVersion, streamID, len(stream))
	}

	// Add new events to the stream
	s.streams[streamID] = append(stream, events...)

	// Add to all events collection
	s.allMutex.Lock()
	defer s.allMutex.Unlock()
	s.allEvents = append(s.allEvents, events...)

	return nil
}

// Load retrieves events for an aggregate
func (s *MemoryEventStore) Load(ctx context.Context, streamID string, fromVersion int) ([]*event.Event, error) {
	s.streamsMutex.RLock()
	defer s.streamsMutex.RUnlock()

	stream, exists := s.streams[streamID]
	if !exists {
		return []*event.Event{}, nil
	}

	if fromVersion >= len(stream) {
		return []*event.Event{}, nil
	}

	return stream[fromVersion:], nil
}

// LoadByType retrieves events of specific types
func (s *MemoryEventStore) LoadByType(ctx context.Context, eventTypes []string, fromPosition int, limit int) ([]*event.Event, error) {
	s.allMutex.RLock()
	defer s.allMutex.RUnlock()

	// Create a map for faster lookup
	typeMap := make(map[string]bool)
	for _, t := range eventTypes {
		typeMap[t] = true
	}

	// Filter events by type
	var result []*event.Event
	for i := fromPosition; i < len(s.allEvents) && (limit <= 0 || len(result) < limit); i++ {
		if typeMap[s.allEvents[i].Type] {
			result = append(result, s.allEvents[i])
		}
	}

	return result, nil
}
```

### **26.3.3 Base Aggregate Implementation**

To simplify creating domain aggregates, let's provide a base implementation:

```go
// eventsourcing/aggregate.go
package eventsourcing

import (
	"github.com/yourorg/eventsystem/event"
)

// AggregateBase provides a base implementation of the Aggregate interface
type AggregateBase struct {
	ID               string
	Version          int
	UncommittedEvents []*event.Event
}

// GetID returns the aggregate's ID
func (a *AggregateBase) GetID() string {
	return a.ID
}

// GetVersion returns the aggregate's current version
func (a *AggregateBase) GetVersion() int {
	return a.Version
}

// GetUncommittedEvents returns events that have not yet been persisted
func (a *AggregateBase) GetUncommittedEvents() []*event.Event {
	return a.UncommittedEvents
}

// ClearUncommittedEvents clears the list of uncommitted events
func (a *AggregateBase) ClearUncommittedEvents() {
	a.UncommittedEvents = make([]*event.Event, 0)
}

// ApplyChange applies an event to the aggregate and adds it to uncommitted events
func (a *AggregateBase) ApplyChange(eventType string, data map[string]interface{}, source string, version string) {
	// Create the event
	evt := event.NewEvent(eventType, source, data, version)

	// Add to uncommitted events
	a.UncommittedEvents = append(a.UncommittedEvents, evt)

	// Increment version
	a.Version++
}
```

### **26.3.4 Generic Repository Implementation**

Let's create a generic repository for loading and saving aggregates:

```go
// eventsourcing/repository.go
package eventsourcing

import (
	"context"
	"fmt"
	"reflect"

	"github.com/yourorg/eventsystem/event"
)

// GenericRepository provides a generic implementation of the Repository interface
type GenericRepository struct {
	store       EventStore
	aggregateType reflect.Type
	eventBus    EventBus
}

// EventBus defines the interface for publishing events
type EventBus interface {
	Publish(ctx context.Context, event *event.Event) error
}

// NewRepository creates a new repository for a specific aggregate type
func NewRepository(store EventStore, aggregateType reflect.Type, eventBus EventBus) *GenericRepository {
	return &GenericRepository{
		store:       store,
		aggregateType: aggregateType,
		eventBus:    eventBus,
	}
}

// Load retrieves an aggregate by ID
func (r *GenericRepository) Load(ctx context.Context, aggregateID string) (Aggregate, error) {
	// Create a new instance of the aggregate
	aggregatePtr := reflect.New(r.aggregateType).Interface().(Aggregate)

	// Load events from the store
	events, err := r.store.Load(ctx, aggregateID, 0)
	if err != nil {
		return nil, fmt.Errorf("failed to load events: %w", err)
	}

	// If no events, return an empty aggregate
	if len(events) == 0 {
		return aggregatePtr, nil
	}

	// Apply events to the aggregate
	for _, evt := range events {
		if err := aggregatePtr.Apply(evt); err != nil {
			return nil, fmt.Errorf("failed to apply event: %w", err)
		}
	}

	return aggregatePtr, nil
}

// Save persists an aggregate's uncommitted events
func (r *GenericRepository) Save(ctx context.Context, aggregate Aggregate) error {
	// Get uncommitted events
	events := aggregate.GetUncommittedEvents()
	if len(events) == 0 {
		return nil
	}

	// Save to event store
	err := r.store.Save(ctx, aggregate.GetID(), events, aggregate.GetVersion()-len(events))
	if err != nil {
		return fmt.Errorf("failed to save events: %w", err)
	}

	// Publish events to the event bus
	if r.eventBus != nil {
		for _, evt := range events {
			if err := r.eventBus.Publish(ctx, evt); err != nil {
				return fmt.Errorf("failed to publish event: %w", err)
			}
		}
	}

	// Clear uncommitted events
	aggregate.ClearUncommittedEvents()

	return nil
}
```

### **26.3.5 Creating a Domain Aggregate with Event Sourcing**

Now, let's implement a simple User aggregate using event sourcing:

```go
// domain/user/aggregate.go
package user

import (
	"fmt"
	"time"

	"github.com/google/uuid"
	"github.com/yourorg/eventsystem/event"
	"github.com/yourorg/eventsystem/eventsourcing"
)

// UserAggregate represents a user in the system
type UserAggregate struct {
	eventsourcing.AggregateBase
	Email       string
	Name        string
	IsActive    bool
	CreatedAt   time.Time
	UpdatedAt   time.Time
	DeactivatedAt *time.Time
}

// NewUser creates a new user aggregate
func NewUser(email, name string) *UserAggregate {
	user := &UserAggregate{}

	// Generate a new ID
	user.ID = uuid.New().String()

	// Apply the creation event
	user.CreateUser(email, name)

	return user
}

// CreateUser applies a user creation event
func (u *UserAggregate) CreateUser(email, name string) {
	u.ApplyChange(
		EventTypeUserCreated,
		map[string]interface{}{
			"email": email,
			"name":  name,
		},
		"user-service",
		EventVersion,
	)
}

// UpdateEmail applies an email update event
func (u *UserAggregate) UpdateEmail(email string) error {
	if !u.IsActive {
		return fmt.Errorf("cannot update email for inactive user")
	}

	u.ApplyChange(
		EventTypeUserEmailUpdated,
		map[string]interface{}{
			"email": email,
		},
		"user-service",
		EventVersion,
	)

	return nil
}

// UpdateName applies a name update event
func (u *UserAggregate) UpdateName(name string) error {
	if !u.IsActive {
		return fmt.Errorf("cannot update name for inactive user")
	}

	u.ApplyChange(
		EventTypeUserNameUpdated,
		map[string]interface{}{
			"name": name,
		},
		"user-service",
		EventVersion,
	)

	return nil
}

// Deactivate applies a user deactivation event
func (u *UserAggregate) Deactivate() error {
	if !u.IsActive {
		return fmt.Errorf("user is already inactive")
	}

	u.ApplyChange(
		EventTypeUserDeactivated,
		map[string]interface{}{},
		"user-service",
		EventVersion,
	)

	return nil
}

// Apply updates the aggregate state based on an event
func (u *UserAggregate) Apply(evt *event.Event) error {
	switch evt.Type {
	case EventTypeUserCreated:
		return u.applyUserCreated(evt)
	case EventTypeUserEmailUpdated:
		return u.applyEmailUpdated(evt)
	case EventTypeUserNameUpdated:
		return u.applyNameUpdated(evt)
	case EventTypeUserDeactivated:
		return u.applyUserDeactivated(evt)
	default:
		return fmt.Errorf("unknown event type: %s", evt.Type)
	}
}

// applyUserCreated applies a user creation event
func (u *UserAggregate) applyUserCreated(evt *event.Event) error {
	email, ok := evt.Data["email"].(string)
	if !ok {
		return fmt.Errorf("invalid email in event data")
	}

	name, ok := evt.Data["name"].(string)
	if !ok {
		return fmt.Errorf("invalid name in event data")
	}

	u.Email = email
	u.Name = name
	u.IsActive = true
	u.CreatedAt = evt.Time
	u.UpdatedAt = evt.Time

	return nil
}

// applyEmailUpdated applies an email update event
func (u *UserAggregate) applyEmailUpdated(evt *event.Event) error {
	email, ok := evt.Data["email"].(string)
	if !ok {
		return fmt.Errorf("invalid email in event data")
	}

	u.Email = email
	u.UpdatedAt = evt.Time

	return nil
}

// applyNameUpdated applies a name update event
func (u *UserAggregate) applyNameUpdated(evt *event.Event) error {
	name, ok := evt.Data["name"].(string)
	if !ok {
		return fmt.Errorf("invalid name in event data")
	}

	u.Name = name
	u.UpdatedAt = evt.Time

	return nil
}

// applyUserDeactivated applies a user deactivation event
func (u *UserAggregate) applyUserDeactivated(evt *event.Event) error {
	u.IsActive = false
	u.UpdatedAt = evt.Time
	now := evt.Time
	u.DeactivatedAt = &now

	return nil
}
```

### **26.3.6 Creating a User Service with Event Sourcing**

Let's put it all together in a user service:

```go
// domain/user/service.go
package user

import (
	"context"
	"fmt"
	"reflect"

	"github.com/yourorg/eventsystem/eventsourcing"
)

// UserService provides user-related operations
type UserService struct {
	repository eventsourcing.Repository
}

// NewUserService creates a new user service
func NewUserService(store eventsourcing.Repository) *UserService {
	return &UserService{
		repository: store,
	}
}

// CreateUser creates a new user
func (s *UserService) CreateUser(ctx context.Context, email, name string) (string, error) {
	// Create new user aggregate
	user := NewUser(email, name)

	// Save the user
	if err := s.repository.Save(ctx, user); err != nil {
		return "", fmt.Errorf("failed to save user: %w", err)
	}

	return user.GetID(), nil
}

// UpdateEmail updates a user's email
func (s *UserService) UpdateEmail(ctx context.Context, userID, email string) error {
	// Load the user
	aggregate, err := s.repository.Load(ctx, userID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Update email
	if err := user.UpdateEmail(email); err != nil {
		return err
	}

	// Save changes
	if err := s.repository.Save(ctx, user); err != nil {
		return fmt.Errorf("failed to save user: %w", err)
	}

	return nil
}

// UpdateName updates a user's name
func (s *UserService) UpdateName(ctx context.Context, userID, name string) error {
	// Load the user
	aggregate, err := s.repository.Load(ctx, userID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Update name
	if err := user.UpdateName(name); err != nil {
		return err
	}

	// Save changes
	if err := s.repository.Save(ctx, user); err != nil {
		return fmt.Errorf("failed to save user: %w", err)
	}

	return nil
}

// DeactivateUser deactivates a user
func (s *UserService) DeactivateUser(ctx context.Context, userID string) error {
	// Load the user
	aggregate, err := s.repository.Load(ctx, userID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Deactivate user
	if err := user.Deactivate(); err != nil {
		return err
	}

	// Save changes
	if err := s.repository.Save(ctx, user); err != nil {
		return fmt.Errorf("failed to save user: %w", err)
	}

	return nil
}

// GetUser retrieves a user by ID
func (s *UserService) GetUser(ctx context.Context, userID string) (*UserDTO, error) {
	// Load the user
	aggregate, err := s.repository.Load(ctx, userID)
	if err != nil {
		return nil, fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return nil, fmt.Errorf("invalid aggregate type")
	}

	// Convert to DTO
	return &UserDTO{
		ID:        user.ID,
		Email:     user.Email,
		Name:      user.Name,
		IsActive:  user.IsActive,
		CreatedAt: user.CreatedAt,
		UpdatedAt: user.UpdatedAt,
	}, nil
}

// UserDTO represents user data for transfer
type UserDTO struct {
	ID        string    `json:"id"`
	Email     string    `json:"email"`
	Name      string    `json:"name"`
	IsActive  bool      `json:"is_active"`
	CreatedAt time.Time `json:"created_at"`
	UpdatedAt time.Time `json:"updated_at"`
}
```

### **26.3.7 Using the Event Sourcing System**

Here's how you would use this event sourcing system:

```go
// main.go
package main

import (
	"context"
	"log"

	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/domain/user"
	"github.com/yourorg/eventsystem/eventsourcing"
	"github.com/yourorg/eventsystem/middleware"
)

func main() {
	// Create the event bus
	eventBus := bus.NewMemoryBus()

	// Add middleware
	eventBus.Use(middleware.Logging())

	// Create the event store
	eventStore := eventsourcing.NewMemoryEventStore()

	// Create the user service
	userService := user.NewUserService(eventStore)

	// Create a new user
	ctx := context.Background()
	userID, err := userService.CreateUser(ctx, "john@example.com", "John Doe")
	if err != nil {
		log.Fatalf("Failed to create user: %v", err)
	}

	log.Printf("Created user with ID: %s", userID)

	// Update the user's email
	if err := userService.UpdateEmail(ctx, userID, "john.doe@example.com"); err != nil {
		log.Fatalf("Failed to update email: %v", err)
	}

	// Get the user
	userDTO, err := userService.GetUser(ctx, userID)
	if err != nil {
		log.Fatalf("Failed to get user: %v", err)
	}

	log.Printf("User: %s <%s>", userDTO.Name, userDTO.Email)

	// Deactivate the user
	if err := userService.DeactivateUser(ctx, userID); err != nil {
		log.Fatalf("Failed to deactivate user: %v", err)
	}

	// Verify the user is inactive
	userDTO, err = userService.GetUser(ctx, userID)
	if err != nil {
		log.Fatalf("Failed to get user: %v", err)
	}

	log.Printf("User active status: %v", userDTO.IsActive)
}
```

## **26.4 Implementing CQRS with Go**

Command Query Responsibility Segregation (CQRS) is a pattern that separates read and write operations into different models. When combined with event sourcing, it enables highly scalable and flexible architectures. In this section, we'll implement a CQRS system in Go.

### **26.4.1 Understanding CQRS Principles**

In CQRS:

- **Commands** represent intentions to change the system state
- **Queries** represent requests for information without side effects
- **Command models** optimize for write operations
- **Query models** optimize for read operations
- **Events** synchronize between command and query models

This separation allows each side to be optimized independently. Let's implement these principles in Go.

### **26.4.2 Command Handling Infrastructure**

First, let's set up the command handling infrastructure:

```go
// cqrs/command.go
package cqrs

import (
	"context"
	"fmt"
	"reflect"
	"sync"
)

// Command represents an intent to change the system state
type Command interface {
	CommandName() string
}

// CommandHandler defines the interface for command handlers
type CommandHandler interface {
	Handle(ctx context.Context, command Command) error
}

// CommandHandlerFunc is a function type that implements CommandHandler
type CommandHandlerFunc func(ctx context.Context, command Command) error

// Handle calls the handler function
func (f CommandHandlerFunc) Handle(ctx context.Context, command Command) error {
	return f(ctx, command)
}

// CommandBus dispatches commands to their handlers
type CommandBus struct {
	handlers     map[string]CommandHandler
	handlersMu   sync.RWMutex
	middlewares  []CommandMiddleware
	middlewareMu sync.RWMutex
}

// CommandMiddleware wraps command handling
type CommandMiddleware func(CommandHandler) CommandHandler

// NewCommandBus creates a new command bus
func NewCommandBus() *CommandBus {
	return &CommandBus{
		handlers: make(map[string]CommandHandler),
	}
}

// Register registers a handler for a specific command type
func (b *CommandBus) Register(command Command, handler CommandHandler) error {
	commandName := command.CommandName()

	b.handlersMu.Lock()
	defer b.handlersMu.Unlock()

	if _, exists := b.handlers[commandName]; exists {
		return fmt.Errorf("handler already registered for command: %s", commandName)
	}

	b.handlers[commandName] = handler
	return nil
}

// RegisterFunc registers a handler function for a specific command type
func (b *CommandBus) RegisterFunc(command Command, handlerFunc func(ctx context.Context, command Command) error) error {
	return b.Register(command, CommandHandlerFunc(handlerFunc))
}

// Dispatch sends a command to its registered handler
func (b *CommandBus) Dispatch(ctx context.Context, command Command) error {
	commandName := command.CommandName()

	b.handlersMu.RLock()
	handler, exists := b.handlers[commandName]
	b.handlersMu.RUnlock()

	if !exists {
		return fmt.Errorf("no handler registered for command: %s", commandName)
	}

	// Apply middlewares
	for i := len(b.middlewares) - 1; i >= 0; i-- {
		handler = b.middlewares[i](handler)
	}

	return handler.Handle(ctx, command)
}

// Use adds middleware to the command processing pipeline
func (b *CommandBus) Use(middleware CommandMiddleware) {
	b.middlewareMu.Lock()
	defer b.middlewareMu.Unlock()
	b.middlewares = append(b.middlewares, middleware)
}
```

### **26.4.3 Query Handling Infrastructure**

Similarly, let's implement the query handling infrastructure:

```go
// cqrs/query.go
package cqrs

import (
	"context"
	"fmt"
	"reflect"
	"sync"
)

// Query represents a request for information
type Query interface {
	QueryName() string
}

// QueryHandler defines the interface for query handlers
type QueryHandler interface {
	Handle(ctx context.Context, query Query) (interface{}, error)
}

// QueryHandlerFunc is a function type that implements QueryHandler
type QueryHandlerFunc func(ctx context.Context, query Query) (interface{}, error)

// Handle calls the handler function
func (f QueryHandlerFunc) Handle(ctx context.Context, query Query) (interface{}, error) {
	return f(ctx, query)
}

// QueryBus dispatches queries to their handlers
type QueryBus struct {
	handlers     map[string]QueryHandler
	handlersMu   sync.RWMutex
	middlewares  []QueryMiddleware
	middlewareMu sync.RWMutex
}

// QueryMiddleware wraps query handling
type QueryMiddleware func(QueryHandler) QueryHandler

// NewQueryBus creates a new query bus
func NewQueryBus() *QueryBus {
	return &QueryBus{
		handlers: make(map[string]QueryHandler),
	}
}

// Register registers a handler for a specific query type
func (b *QueryBus) Register(query Query, handler QueryHandler) error {
	queryName := query.QueryName()

	b.handlersMu.Lock()
	defer b.handlersMu.Unlock()

	if _, exists := b.handlers[queryName]; exists {
		return fmt.Errorf("handler already registered for query: %s", queryName)
	}

	b.handlers[queryName] = handler
	return nil
}

// RegisterFunc registers a handler function for a specific query type
func (b *QueryBus) RegisterFunc(query Query, handlerFunc func(ctx context.Context, query Query) (interface{}, error)) error {
	return b.Register(query, QueryHandlerFunc(handlerFunc))
}

// Dispatch sends a query to its registered handler
func (b *QueryBus) Dispatch(ctx context.Context, query Query) (interface{}, error) {
	queryName := query.QueryName()

	b.handlersMu.RLock()
	handler, exists := b.handlers[queryName]
	b.handlersMu.RUnlock()

	if !exists {
		return nil, fmt.Errorf("no handler registered for query: %s", queryName)
	}

	// Apply middlewares
	for i := len(b.middlewares) - 1; i >= 0; i-- {
		handler = b.middlewares[i](handler)
	}

	return handler.Handle(ctx, query)
}

// Use adds middleware to the query processing pipeline
func (b *QueryBus) Use(middleware QueryMiddleware) {
	b.middlewareMu.Lock()
	defer b.middlewareMu.Unlock()
	b.middlewares = append(b.middlewares, middleware)
}
```

### **26.4.4 Implementing Commands and Queries for the User Domain**

Let's implement commands and queries for our user domain:

```go
// domain/user/commands.go
package user

import (
	"context"
	"fmt"

	"github.com/yourorg/eventsystem/cqrs"
	"github.com/yourorg/eventsystem/eventsourcing"
)

// CreateUserCommand represents a command to create a new user
type CreateUserCommand struct {
	Email string `json:"email"`
	Name  string `json:"name"`
}

func (c CreateUserCommand) CommandName() string {
	return "user.create"
}

// UpdateEmailCommand represents a command to update a user's email
type UpdateEmailCommand struct {
	UserID string `json:"user_id"`
	Email  string `json:"email"`
}

func (c UpdateEmailCommand) CommandName() string {
	return "user.update_email"
}

// UpdateNameCommand represents a command to update a user's name
type UpdateNameCommand struct {
	UserID string `json:"user_id"`
	Name   string `json:"name"`
}

func (c UpdateNameCommand) CommandName() string {
	return "user.update_name"
}

// DeactivateUserCommand represents a command to deactivate a user
type DeactivateUserCommand struct {
	UserID string `json:"user_id"`
}

func (c DeactivateUserCommand) CommandName() string {
	return "user.deactivate"
}

// CommandHandlers contains handlers for user commands
type CommandHandlers struct {
	repository eventsourcing.Repository
}

// NewCommandHandlers creates new command handlers for users
func NewCommandHandlers(repository eventsourcing.Repository) *CommandHandlers {
	return &CommandHandlers{
		repository: repository,
	}
}

// RegisterHandlers registers all command handlers
func (h *CommandHandlers) RegisterHandlers(commandBus *cqrs.CommandBus) {
	commandBus.RegisterFunc(CreateUserCommand{}, h.HandleCreateUser)
	commandBus.RegisterFunc(UpdateEmailCommand{}, h.HandleUpdateEmail)
	commandBus.RegisterFunc(UpdateNameCommand{}, h.HandleUpdateName)
	commandBus.RegisterFunc(DeactivateUserCommand{}, h.HandleDeactivateUser)
}

// HandleCreateUser handles the CreateUserCommand
func (h *CommandHandlers) HandleCreateUser(ctx context.Context, cmd cqrs.Command) error {
	createCmd, ok := cmd.(CreateUserCommand)
	if !ok {
		return fmt.Errorf("invalid command type")
	}

	// Create new user aggregate
	user := NewUser(createCmd.Email, createCmd.Name)

	// Save the aggregate
	return h.repository.Save(ctx, user)
}

// HandleUpdateEmail handles the UpdateEmailCommand
func (h *CommandHandlers) HandleUpdateEmail(ctx context.Context, cmd cqrs.Command) error {
	updateCmd, ok := cmd.(UpdateEmailCommand)
	if !ok {
		return fmt.Errorf("invalid command type")
	}

	// Load the aggregate
	aggregate, err := h.repository.Load(ctx, updateCmd.UserID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Update email
	if err := user.UpdateEmail(updateCmd.Email); err != nil {
		return err
	}

	// Save changes
	return h.repository.Save(ctx, user)
}

// HandleUpdateName handles the UpdateNameCommand
func (h *CommandHandlers) HandleUpdateName(ctx context.Context, cmd cqrs.Command) error {
	updateCmd, ok := cmd.(UpdateNameCommand)
	if !ok {
		return fmt.Errorf("invalid command type")
	}

	// Load the aggregate
	aggregate, err := h.repository.Load(ctx, updateCmd.UserID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Update name
	if err := user.UpdateName(updateCmd.Name); err != nil {
		return err
	}

	// Save changes
	return h.repository.Save(ctx, user)
}

// HandleDeactivateUser handles the DeactivateUserCommand
func (h *CommandHandlers) HandleDeactivateUser(ctx context.Context, cmd cqrs.Command) error {
	deactivateCmd, ok := cmd.(DeactivateUserCommand)
	if !ok {
		return fmt.Errorf("invalid command type")
	}

	// Load the aggregate
	aggregate, err := h.repository.Load(ctx, deactivateCmd.UserID)
	if err != nil {
		return fmt.Errorf("failed to load user: %w", err)
	}

	user, ok := aggregate.(*UserAggregate)
	if !ok {
		return fmt.Errorf("invalid aggregate type")
	}

	// Deactivate user
	if err := user.Deactivate(); err != nil {
		return err
	}

	// Save changes
	return h.repository.Save(ctx, user)
}
```

```go
// domain/user/queries.go
package user

import (
	"context"
	"fmt"
	"time"

	"github.com/yourorg/eventsystem/cqrs"
)

// GetUserQuery represents a query to get user details
type GetUserQuery struct {
	UserID string `json:"user_id"`
}

func (q GetUserQuery) QueryName() string {
	return "user.get"
}

// ListUsersQuery represents a query to list users
type ListUsersQuery struct {
	Limit  int  `json:"limit"`
	Offset int  `json:"offset"`
	Active bool `json:"active"`
}

func (q ListUsersQuery) QueryName() string {
	return "user.list"
}

// UserView represents a read model for users
type UserView struct {
	ID            string     `json:"id"`
	Email         string     `json:"email"`
	Name          string     `json:"name"`
	IsActive      bool       `json:"is_active"`
	CreatedAt     time.Time  `json:"created_at"`
	UpdatedAt     time.Time  `json:"updated_at"`
	DeactivatedAt *time.Time `json:"deactivated_at,omitempty"`
}

// UserViewRepository defines the interface for user read models
type UserViewRepository interface {
	GetUser(ctx context.Context, id string) (*UserView, error)
	ListUsers(ctx context.Context, limit, offset int, activeOnly bool) ([]*UserView, error)
	SaveUser(ctx context.Context, user *UserView) error
}

// QueryHandlers contains handlers for user queries
type QueryHandlers struct {
	repository UserViewRepository
}

// NewQueryHandlers creates new query handlers for users
func NewQueryHandlers(repository UserViewRepository) *QueryHandlers {
	return &QueryHandlers{
		repository: repository,
	}
}

// RegisterHandlers registers all query handlers
func (h *QueryHandlers) RegisterHandlers(queryBus *cqrs.QueryBus) {
	queryBus.RegisterFunc(GetUserQuery{}, h.HandleGetUser)
	queryBus.RegisterFunc(ListUsersQuery{}, h.HandleListUsers)
}

// HandleGetUser handles the GetUserQuery
func (h *QueryHandlers) HandleGetUser(ctx context.Context, q cqrs.Query) (interface{}, error) {
	getQuery, ok := q.(GetUserQuery)
	if !ok {
		return nil, fmt.Errorf("invalid query type")
	}

	return h.repository.GetUser(ctx, getQuery.UserID)
}

// HandleListUsers handles the ListUsersQuery
func (h *QueryHandlers) HandleListUsers(ctx context.Context, q cqrs.Query) (interface{}, error) {
	listQuery, ok := q.(ListUsersQuery)
	if !ok {
		return nil, fmt.Errorf("invalid query type")
	}

	return h.repository.ListUsers(ctx, listQuery.Limit, listQuery.Offset, listQuery.Active)
}
```

### **26.4.5 Creating Read Models for Queries**

To optimize for reads, let's implement a simple in-memory read model repository:

```go
// domain/user/readmodel.go
package user

import (
	"context"
	"fmt"
	"sync"
	"time"

	"github.com/yourorg/eventsystem/event"
)

// InMemoryUserViewRepository implements UserViewRepository with in-memory storage
type InMemoryUserViewRepository struct {
	users    map[string]*UserView
	usersMu  sync.RWMutex
}

// NewInMemoryUserViewRepository creates a new in-memory user view repository
func NewInMemoryUserViewRepository() *InMemoryUserViewRepository {
	return &InMemoryUserViewRepository{
		users: make(map[string]*UserView),
	}
}

// GetUser retrieves a user by ID
func (r *InMemoryUserViewRepository) GetUser(ctx context.Context, id string) (*UserView, error) {
	r.usersMu.RLock()
	defer r.usersMu.RUnlock()

	user, exists := r.users[id]
	if !exists {
		return nil, fmt.Errorf("user not found: %s", id)
	}

	return user, nil
}

// ListUsers retrieves a list of users
func (r *InMemoryUserViewRepository) ListUsers(ctx context.Context, limit, offset int, activeOnly bool) ([]*UserView, error) {
	r.usersMu.RLock()
	defer r.usersMu.RUnlock()

	var result []*UserView

	// Collect all users matching criteria
	for _, user := range r.users {
		if activeOnly && !user.IsActive {
			continue
		}
		result = append(result, user)
	}

	// Apply pagination
	if offset >= len(result) {
		return []*UserView{}, nil
	}

	end := offset + limit
	if end > len(result) || limit <= 0 {
		end = len(result)
	}

	return result[offset:end], nil
}

// SaveUser saves a user
func (r *InMemoryUserViewRepository) SaveUser(ctx context.Context, user *UserView) error {
	r.usersMu.Lock()
	defer r.usersMu.Unlock()

	r.users[user.ID] = user
	return nil
}

// EventHandler handles domain events and updates the read model
type EventHandler struct {
	repository UserViewRepository
}

// NewEventHandler creates a new event handler
func NewEventHandler(repository UserViewRepository) *EventHandler {
	return &EventHandler{
		repository: repository,
	}
}

// HandleEvent processes an event and updates the read model
func (h *EventHandler) HandleEvent(ctx context.Context, evt *event.Event) error {
	switch evt.Type {
	case EventTypeUserCreated:
		return h.handleUserCreated(ctx, evt)
	case EventTypeUserEmailUpdated:
		return h.handleEmailUpdated(ctx, evt)
	case EventTypeUserNameUpdated:
		return h.handleNameUpdated(ctx, evt)
	case EventTypeUserDeactivated:
		return h.handleUserDeactivated(ctx, evt)
	default:
		return nil // Ignore unknown events
	}
}

// handleUserCreated handles user creation events
func (h *EventHandler) handleUserCreated(ctx context.Context, evt *event.Event) error {
	// Extract data from event
	email, ok := evt.Data["email"].(string)
	if !ok {
		return fmt.Errorf("invalid email in event data")
	}

	name, ok := evt.Data["name"].(string)
	if !ok {
		return fmt.Errorf("invalid name in event data")
	}

	// Create user view
	user := &UserView{
		ID:        evt.GetStreamID(),
		Email:     email,
		Name:      name,
		IsActive:  true,
		CreatedAt: evt.Time,
		UpdatedAt: evt.Time,
	}

	// Save to repository
	return h.repository.SaveUser(ctx, user)
}

// handleEmailUpdated handles email update events
func (h *EventHandler) handleEmailUpdated(ctx context.Context, evt *event.Event) error {
	// Get user from repository
	user, err := h.repository.GetUser(ctx, evt.GetStreamID())
	if err != nil {
		return err
	}

	// Extract data from event
	email, ok := evt.Data["email"].(string)
	if !ok {
		return fmt.Errorf("invalid email in event data")
	}

	// Update user view
	user.Email = email
	user.UpdatedAt = evt.Time

	// Save to repository
	return h.repository.SaveUser(ctx, user)
}

// handleNameUpdated handles name update events
func (h *EventHandler) handleNameUpdated(ctx context.Context, evt *event.Event) error {
	// Get user from repository
	user, err := h.repository.GetUser(ctx, evt.GetStreamID())
	if err != nil {
		return err
	}

	// Extract data from event
	name, ok := evt.Data["name"].(string)
	if !ok {
		return fmt.Errorf("invalid name in event data")
	}

	// Update user view
	user.Name = name
	user.UpdatedAt = evt.Time

	// Save to repository
	return h.repository.SaveUser(ctx, user)
}

// handleUserDeactivated handles user deactivation events
func (h *EventHandler) handleUserDeactivated(ctx context.Context, evt *event.Event) error {
	// Get user from repository
	user, err := h.repository.GetUser(ctx, evt.GetStreamID())
	if err != nil {
		return err
	}

	// Update user view
	user.IsActive = false
	user.UpdatedAt = evt.Time
	now := evt.Time
	user.DeactivatedAt = &now

	// Save to repository
	return h.repository.SaveUser(ctx, user)
}
```

### **26.4.6 Putting It All Together**

Let's see how all the CQRS components work together:

```go
// main.go
package main

import (
	"context"
	"log"
	"reflect"

	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/cqrs"
	"github.com/yourorg/eventsystem/domain/user"
	"github.com/yourorg/eventsystem/eventsourcing"
	"github.com/yourorg/eventsystem/middleware"
)

func main() {
	// Create the event bus
	eventBus := bus.NewMemoryBus()

	// Add middleware
	eventBus.Use(middleware.Logging())

	// Create the event store
	eventStore := eventsourcing.NewMemoryEventStore()

	// Create the user repository
	userRepository := eventsourcing.NewRepository(
		eventStore,
		reflect.TypeOf(user.UserAggregate{}),
		eventBus,
	)

	// Create command and query buses
	commandBus := cqrs.NewCommandBus()
	queryBus := cqrs.NewQueryBus()

	// Create the read model repository
	readModelRepo := user.NewInMemoryUserViewRepository()

	// Create and register command handlers
	commandHandlers := user.NewCommandHandlers(userRepository)
	commandHandlers.RegisterHandlers(commandBus)

	// Create and register query handlers
	queryHandlers := user.NewQueryHandlers(readModelRepo)
	queryHandlers.RegisterHandlers(queryBus)

	// Create and register event handlers for the read model
	eventHandler := user.NewEventHandler(readModelRepo)
	eventBus.SubscribeFunc(user.EventTypeUserCreated, eventHandler.HandleEvent)
	eventBus.SubscribeFunc(user.EventTypeUserEmailUpdated, eventHandler.HandleEvent)
	eventBus.SubscribeFunc(user.EventTypeUserNameUpdated, eventHandler.HandleEvent)
	eventBus.SubscribeFunc(user.EventTypeUserDeactivated, eventHandler.HandleEvent)

	// Create a context
	ctx := context.Background()

	// Create a user via command
	createCmd := user.CreateUserCommand{
		Email: "jane@example.com",
		Name:  "Jane Doe",
	}

	var userID string
	if err := commandBus.Dispatch(ctx, createCmd); err != nil {
		log.Fatalf("Failed to create user: %v", err)
	}

	// Query to get the user
	getUserQuery := user.GetUserQuery{
		UserID: userID,
	}

	result, err := queryBus.Dispatch(ctx, getUserQuery)
	if err != nil {
		log.Fatalf("Failed to get user: %v", err)
	}

	userView := result.(*user.UserView)
	log.Printf("User: %s <%s>", userView.Name, userView.Email)

	// Update the user's email via command
	updateEmailCmd := user.UpdateEmailCommand{
		UserID: userID,
		Email:  "jane.doe@example.com",
	}

	if err := commandBus.Dispatch(ctx, updateEmailCmd); err != nil {
		log.Fatalf("Failed to update email: %v", err)
	}

	// Query to get the updated user
	result, err = queryBus.Dispatch(ctx, getUserQuery)
	if err != nil {
		log.Fatalf("Failed to get user: %v", err)
	}

	userView = result.(*user.UserView)
	log.Printf("Updated user email: %s", userView.Email)

	// List active users
	listQuery := user.ListUsersQuery{
		Limit:  10,
		Offset: 0,
		Active: true,
	}

	result, err = queryBus.Dispatch(ctx, listQuery)
	if err != nil {
		log.Fatalf("Failed to list users: %v", err)
	}

	users := result.([]*user.UserView)
	log.Printf("Found %d active users", len(users))
}
```

## **26.5 Implementing Distributed Event Systems**

The event-driven patterns we've explored so far work well within a single application, but modern systems often span multiple services across different machines. Let's explore how to implement distributed event systems in Go using industry-standard message brokers.

### **26.5.1 Choosing a Message Broker**

Several message brokers are popular in Go ecosystems:

- **Kafka**: High-throughput, durable, distributed event streaming platform
- **NATS**: Lightweight, high-performance messaging system
- **RabbitMQ**: Feature-rich message broker implementing AMQP
- **Redis Pub/Sub**: Simple but effective for less demanding use cases

We'll implement adapters for both Kafka and NATS to demonstrate different approaches.

### **26.5.2 Implementing a Kafka Event Bus**

Let's start with a Kafka implementation:

```go
// bus/kafka.go
package bus

import (
	"context"
	"encoding/json"
	"fmt"
	"sync"
	"time"

	"github.com/segmentio/kafka-go"
	"github.com/yourorg/eventsystem/event"
)

// KafkaBus implements an event bus with Kafka
type KafkaBus struct {
	writer      *kafka.Writer
	readers     map[string]*kafka.Reader
	readersMu   sync.RWMutex
	handlers    map[string][]Handler
	handlersMu  sync.RWMutex
	middlewares []Middleware
	middlewareMu sync.RWMutex
	brokers     []string
	clientID    string
	consumerGroup string
}

// KafkaBusOptions contains options for the Kafka event bus
type KafkaBusOptions struct {
	Brokers       []string
	ClientID      string
	ConsumerGroup string
}

// NewKafkaBus creates a new Kafka event bus
func NewKafkaBus(opts KafkaBusOptions) *KafkaBus {
	writer := &kafka.Writer{
		Addr:     kafka.TCP(opts.Brokers...),
		Balancer: &kafka.LeastBytes{},
	}

	return &KafkaBus{
		writer:       writer,
		readers:      make(map[string]*kafka.Reader),
		handlers:     make(map[string][]Handler),
		brokers:      opts.Brokers,
		clientID:     opts.ClientID,
		consumerGroup: opts.ConsumerGroup,
	}
}

// Subscribe registers a handler for a specific event type
func (b *KafkaBus) Subscribe(eventType string, handler Handler) error {
	b.handlersMu.Lock()
	defer b.handlersMu.Unlock()

	if _, ok := b.handlers[eventType]; !ok {
		b.handlers[eventType] = make([]Handler, 0)

		// Start consuming from the topic if this is the first handler
		if err := b.startConsuming(eventType); err != nil {
			return err
		}
	}

	b.handlers[eventType] = append(b.handlers[eventType], handler)
	return nil
}

// SubscribeFunc registers a handler function for a specific event type
func (b *KafkaBus) SubscribeFunc(eventType string, handlerFunc func(ctx context.Context, event *event.Event) error) error {
	return b.Subscribe(eventType, HandlerFunc(handlerFunc))
}

// Publish sends an event to all subscribed handlers
func (b *KafkaBus) Publish(ctx context.Context, evt *event.Event) error {
	// Marshal event to JSON
	data, err := json.Marshal(evt)
	if err != nil {
		return fmt.Errorf("failed to marshal event: %w", err)
	}

	// Create Kafka message
	msg := kafka.Message{
		Topic: evt.Type, // Use event type as Kafka topic
		Key:   []byte(evt.ID),
		Value: data,
		Time:  time.Now(),
	}

	// Publish message
	return b.writer.WriteMessages(ctx, msg)
}

// startConsuming starts consuming events from a Kafka topic
func (b *KafkaBus) startConsuming(eventType string) error {
	b.readersMu.Lock()
	defer b.readersMu.Unlock()

	// Check if we're already consuming from this topic
	if _, exists := b.readers[eventType]; exists {
		return nil
	}

	// Create a new reader for this topic
	reader := kafka.NewReader(kafka.ReaderConfig{
		Brokers:     b.brokers,
		Topic:       eventType,
		GroupID:     b.consumerGroup,
		StartOffset: kafka.FirstOffset,
		MinBytes:    10e3, // 10KB
		MaxBytes:    10e6, // 10MB
	})

	b.readers[eventType] = reader

	// Start a goroutine to consume messages
	go b.consumeMessages(eventType, reader)

	return nil
}

// consumeMessages continuously consumes messages from a Kafka topic
func (b *KafkaBus) consumeMessages(eventType string, reader *kafka.Reader) {
	for {
		ctx := context.Background()
		msg, err := reader.ReadMessage(ctx)
		if err != nil {
			fmt.Printf("Error reading message from Kafka: %v\n", err)
			continue
		}

		// Unmarshal event
		var evt event.Event
		if err := json.Unmarshal(msg.Value, &evt); err != nil {
			fmt.Printf("Error unmarshaling event: %v\n", err)
			continue
		}

		// Get handlers for this event type
		b.handlersMu.RLock()
		handlers, ok := b.handlers[eventType]
		b.handlersMu.RUnlock()

		if !ok || len(handlers) == 0 {
			continue
		}

		// Process with each handler
		for _, h := range handlers {
			// Apply middlewares
			handler := h
			for i := len(b.middlewares) - 1; i >= 0; i-- {
				handler = b.middlewares[i](handler)
			}

			// Process event
			if err := handler.HandleEvent(ctx, &evt); err != nil {
				fmt.Printf("Error handling event %s: %v\n", evt.ID, err)
				// In a production system, you would likely implement a retry mechanism
				// or a dead-letter queue for failed events
			}
		}
	}
}

// Use adds middleware to the event processing pipeline
func (b *KafkaBus) Use(middleware Middleware) {
	b.middlewareMu.Lock()
	defer b.middlewareMu.Unlock()
	b.middlewares = append(b.middlewares, middleware)
}

// Close closes all readers and the writer
func (b *KafkaBus) Close() error {
	b.readersMu.Lock()
	defer b.readersMu.Unlock()

	// Close all readers
	for _, reader := range b.readers {
		if err := reader.Close(); err != nil {
			return err
		}
	}

	// Close writer
	return b.writer.Close()
}
```

### **26.5.3 Implementing a NATS Event Bus**

Now let's implement a NATS-based event bus:

```go
// bus/nats.go
package bus

import (
	"context"
	"encoding/json"
	"fmt"
	"sync"

	"github.com/nats-io/nats.go"
	"github.com/yourorg/eventsystem/event"
)

// NatsBus implements an event bus with NATS
type NatsBus struct {
	conn        *nats.Conn
	js          nats.JetStreamContext
	handlers    map[string][]Handler
	handlersMu  sync.RWMutex
	middlewares []Middleware
	middlewareMu sync.RWMutex
	subs        map[string]*nats.Subscription
	subsMu      sync.RWMutex
}

// NatsBusOptions contains options for the NATS event bus
type NatsBusOptions struct {
	URL          string
	StreamName   string
	StreamConfig *nats.StreamConfig
}

// NewNatsBus creates a new NATS event bus
func NewNatsBus(opts NatsBusOptions) (*NatsBus, error) {
	// Connect to NATS
	conn, err := nats.Connect(opts.URL)
	if err != nil {
		return nil, fmt.Errorf("failed to connect to NATS: %w", err)
	}

	// Create JetStream context
	js, err := conn.JetStream()
	if err != nil {
		conn.Close()
		return nil, fmt.Errorf("failed to create JetStream context: %w", err)
	}

	// Create or update stream if config provided
	if opts.StreamConfig != nil {
		_, err = js.AddStream(opts.StreamConfig)
		if err != nil {
			conn.Close()
			return nil, fmt.Errorf("failed to create stream: %w", err)
		}
	}

	return &NatsBus{
		conn:     conn,
		js:       js,
		handlers: make(map[string][]Handler),
		subs:     make(map[string]*nats.Subscription),
	}, nil
}

// Subscribe registers a handler for a specific event type
func (b *NatsBus) Subscribe(eventType string, handler Handler) error {
	b.handlersMu.Lock()
	defer b.handlersMu.Unlock()

	if _, ok := b.handlers[eventType]; !ok {
		b.handlers[eventType] = make([]Handler, 0)

		// Start subscription if this is the first handler
		if err := b.subscribe(eventType); err != nil {
			return err
		}
	}

	b.handlers[eventType] = append(b.handlers[eventType], handler)
	return nil
}

// SubscribeFunc registers a handler function for a specific event type
func (b *NatsBus) SubscribeFunc(eventType string, handlerFunc func(ctx context.Context, event *event.Event) error) error {
	return b.Subscribe(eventType, HandlerFunc(handlerFunc))
}

// Publish sends an event to all subscribed handlers
func (b *NatsBus) Publish(ctx context.Context, evt *event.Event) error {
	// Marshal event to JSON
	data, err := json.Marshal(evt)
	if err != nil {
		return fmt.Errorf("failed to marshal event: %w", err)
	}

	// Publish message
	_, err = b.js.Publish(evt.Type, data)
	return err
}

// subscribe creates a subscription for a specific event type
func (b *NatsBus) subscribe(eventType string) error {
	b.subsMu.Lock()
	defer b.subsMu.Unlock()

	// Check if we're already subscribed to this event type
	if _, exists := b.subs[eventType]; exists {
		return nil
	}

	// Create a new subscription
	sub, err := b.js.Subscribe(eventType, func(msg *nats.Msg) {
		// Unmarshal event
		var evt event.Event
		if err := json.Unmarshal(msg.Data, &evt); err != nil {
			fmt.Printf("Error unmarshaling event: %v\n", err)
			return
		}

		// Get handlers for this event type
		b.handlersMu.RLock()
		handlers, ok := b.handlers[eventType]
		b.handlersMu.RUnlock()

		if !ok || len(handlers) == 0 {
			return
		}

		// Process with each handler
		for _, h := range handlers {
			// Apply middlewares
			handler := h
			for i := len(b.middlewares) - 1; i >= 0; i-- {
				handler = b.middlewares[i](handler)
			}

			// Process event
			ctx := context.Background()
			if err := handler.HandleEvent(ctx, &evt); err != nil {
				fmt.Printf("Error handling event %s: %v\n", evt.ID, err)
				// In a production system, you would likely implement a retry mechanism
				// or a dead-letter queue for failed events
			}
		}

		// Acknowledge message
		msg.Ack()
	})

	if err != nil {
		return fmt.Errorf("failed to subscribe to event type %s: %w", eventType, err)
	}

	b.subs[eventType] = sub
	return nil
}

// Use adds middleware to the event processing pipeline
func (b *NatsBus) Use(middleware Middleware) {
	b.middlewareMu.Lock()
	defer b.middlewareMu.Unlock()
	b.middlewares = append(b.middlewares, middleware)
}

// Close closes the NATS connection
func (b *NatsBus) Close() error {
	b.subsMu.Lock()
	defer b.subsMu.Unlock()

	// Unsubscribe from all subscriptions
	for _, sub := range b.subs {
		sub.Unsubscribe()
	}

	// Close connection
	b.conn.Close()
	return nil
}
```

### **26.5.4 Implementing Event Serialization and Schema Evolution**

A crucial aspect of distributed event systems is managing event serialization and schema evolution. Let's implement a schema registry for our events:

```go
// schema/registry.go
package schema

import (
	"fmt"
	"sync"

	"github.com/xeipuuv/gojsonschema"
)

// Registry manages event schemas
type Registry struct {
	schemas    map[string]map[string]*gojsonschema.Schema // type -> version -> schema
	schemasMu  sync.RWMutex
	migrations map[string]map[string]MigrationFunc // type -> from_version:to_version -> func
	migrationsMu sync.RWMutex
}

// MigrationFunc defines a function that migrates event data from one version to another
type MigrationFunc func(data map[string]interface{}) (map[string]interface{}, error)

// NewRegistry creates a new schema registry
func NewRegistry() *Registry {
	return &Registry{
		schemas:    make(map[string]map[string]*gojsonschema.Schema),
		migrations: make(map[string]map[string]MigrationFunc),
	}
}

// RegisterSchema registers a schema for an event type and version
func (r *Registry) RegisterSchema(eventType, version string, schemaJSON string) error {
	// Parse schema
	schemaLoader := gojsonschema.NewStringLoader(schemaJSON)
	schema, err := gojsonschema.NewSchema(schemaLoader)
	if err != nil {
		return fmt.Errorf("invalid schema: %w", err)
	}

	r.schemasMu.Lock()
	defer r.schemasMu.Unlock()

	// Initialize version map if needed
	if _, ok := r.schemas[eventType]; !ok {
		r.schemas[eventType] = make(map[string]*gojsonschema.Schema)
	}

	// Store schema
	r.schemas[eventType][version] = schema
	return nil
}

// RegisterMigration registers a migration function between versions
func (r *Registry) RegisterMigration(eventType, fromVersion, toVersion string, migration MigrationFunc) error {
	r.migrationsMu.Lock()
	defer r.migrationsMu.Unlock()

	// Check if schemas exist
	r.schemasMu.RLock()
	defer r.schemasMu.RUnlock()

	typeSchemas, ok := r.schemas[eventType]
	if !ok {
		return fmt.Errorf("no schemas registered for event type: %s", eventType)
	}

	if _, ok := typeSchemas[fromVersion]; !ok {
		return fmt.Errorf("no schema registered for event type %s version %s", eventType, fromVersion)
	}

	if _, ok := typeSchemas[toVersion]; !ok {
		return fmt.Errorf("no schema registered for event type %s version %s", eventType, toVersion)
	}

	// Initialize migration map if needed
	if _, ok := r.migrations[eventType]; !ok {
		r.migrations[eventType] = make(map[string]MigrationFunc)
	}

	// Store migration function
	migrationKey := fmt.Sprintf("%s:%s", fromVersion, toVersion)
	r.migrations[eventType][migrationKey] = migration
	return nil
}

// Validate validates event data against its schema
func (r *Registry) Validate(eventType, version string, data map[string]interface{}) error {
	r.schemasMu.RLock()
	defer r.schemasMu.RUnlock()

	// Check if schema exists
	typeSchemas, ok := r.schemas[eventType]
	if !ok {
		return fmt.Errorf("no schemas registered for event type: %s", eventType)
	}

	schema, ok := typeSchemas[version]
	if !ok {
		return fmt.Errorf("no schema registered for event type %s version %s", eventType, version)
	}

	// Validate data against schema
	dataLoader := gojsonschema.NewGoLoader(data)
	result, err := schema.Validate(dataLoader)
	if err != nil {
		return fmt.Errorf("validation error: %w", err)
	}

	if !result.Valid() {
		var errMsg string
		for _, desc := range result.Errors() {
			errMsg += fmt.Sprintf("- %s\n", desc)
		}
		return fmt.Errorf("invalid event data:\n%s", errMsg)
	}

	return nil
}

// Migrate migrates event data from one version to another
func (r *Registry) Migrate(eventType, fromVersion, toVersion string, data map[string]interface{}) (map[string]interface{}, error) {
	// If versions are the same, no migration needed
	if fromVersion == toVersion {
		return data, nil
	}

	r.migrationsMu.RLock()
	defer r.migrationsMu.RUnlock()

	// Check if migration exists
	typeMigrations, ok := r.migrations[eventType]
	if !ok {
		return nil, fmt.Errorf("no migrations registered for event type: %s", eventType)
	}

	migrationKey := fmt.Sprintf("%s:%s", fromVersion, toVersion)
	migration, ok := typeMigrations[migrationKey]
	if !ok {
		return nil, fmt.Errorf("no migration registered for event type %s from version %s to %s",
			eventType, fromVersion, toVersion)
	}

	// Apply migration
	migratedData, err := migration(data)
	if err != nil {
		return nil, fmt.Errorf("migration error: %w", err)
	}

	return migratedData, nil
}
```

### **26.5.5 Distributed Event Store with PostgreSQL**

Let's create a PostgreSQL-based event store for persisting events in a distributed environment:

```go
// eventsourcing/postgres_store.go
package eventsourcing

import (
	"context"
	"database/sql"
	"encoding/json"
	"fmt"
	"time"

	"github.com/lib/pq"
	"github.com/yourorg/eventsystem/event"
)

// PostgresEventStore implements EventStore with PostgreSQL
type PostgresEventStore struct {
	db *sql.DB
}

// NewPostgresEventStore creates a new PostgreSQL event store
func NewPostgresEventStore(db *sql.DB) (*PostgresEventStore, error) {
	// Create events table if it doesn't exist
	_, err := db.Exec(`
		CREATE TABLE IF NOT EXISTS events (
			id TEXT PRIMARY KEY,
			stream_id TEXT NOT NULL,
			type TEXT NOT NULL,
			source TEXT NOT NULL,
			data JSONB NOT NULL,
			metadata JSONB,
			correlation_id TEXT,
			causation_id TEXT,
			time TIMESTAMP WITH TIME ZONE NOT NULL,
			data_version TEXT NOT NULL,
			stream_position INTEGER NOT NULL,
			global_position SERIAL,
			UNIQUE (stream_id, stream_position)
		);
		CREATE INDEX IF NOT EXISTS idx_events_stream ON events (stream_id, stream_position);
		CREATE INDEX IF NOT EXISTS idx_events_type ON events (type);
		CREATE INDEX IF NOT EXISTS idx_events_time ON events (time);
		CREATE INDEX IF NOT EXISTS idx_events_correlation ON events (correlation_id);
	`)
	if err != nil {
		return nil, fmt.Errorf("failed to create events table: %w", err)
	}

	return &PostgresEventStore{db: db}, nil
}

// Save stores events for an aggregate
func (s *PostgresEventStore) Save(ctx context.Context, streamID string, events []*event.Event, expectedVersion int) error {
	// Start transaction
	tx, err := s.db.BeginTx(ctx, nil)
	if err != nil {
		return fmt.Errorf("failed to start transaction: %w", err)
	}
	defer tx.Rollback()

	// Get current stream version
	var currentVersion int
	err = tx.QueryRowContext(ctx,
		"SELECT COALESCE(MAX(stream_position), -1) FROM events WHERE stream_id = $1",
		streamID).Scan(&currentVersion)
	if err != nil {
		return fmt.Errorf("failed to get current version: %w", err)
	}

	// Check expected version
	if currentVersion != expectedVersion {
		return fmt.Errorf("expected version %d for stream %s, got %d", expectedVersion, streamID, currentVersion)
	}

	// Insert events
	stmt, err := tx.PrepareContext(ctx, `
		INSERT INTO events (
			id, stream_id, type, source, data, metadata, correlation_id, causation_id,
			time, data_version, stream_position
		) VALUES (
			$1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11
		)
	`)
	if err != nil {
		return fmt.Errorf("failed to prepare statement: %w", err)
	}
	defer stmt.Close()

	for i, evt := range events {
		position := currentVersion + 1 + i

		// Set stream ID if not already set
		if evt.GetStreamID() == "" {
			evt.SetStreamID(streamID)
		}

		// Marshal data and metadata to JSON
		dataJSON, err := json.Marshal(evt.Data)
		if err != nil {
			return fmt.Errorf("failed to marshal event data: %w", err)
		}

		var metadataJSON []byte
		if evt.Metadata != nil {
			metadataJSON, err = json.Marshal(evt.Metadata)
			if err != nil {
				return fmt.Errorf("failed to marshal event metadata: %w", err)
			}
		}

		// Insert event
		_, err = stmt.ExecContext(ctx,
			evt.ID,
			streamID,
			evt.Type,
			evt.Source,
			dataJSON,
			metadataJSON,
			evt.CorrelationID,
			evt.CausationID,
			evt.Time,
			evt.DataVersion,
			position,
		)
		if err != nil {
			return fmt.Errorf("failed to insert event: %w", err)
		}
	}

	// Commit transaction
	if err := tx.Commit(); err != nil {
		return fmt.Errorf("failed to commit transaction: %w", err)
	}

	return nil
}

// Load retrieves events for an aggregate
func (s *PostgresEventStore) Load(ctx context.Context, streamID string, fromVersion int) ([]*event.Event, error) {
	// Query events
	rows, err := s.db.QueryContext(ctx, `
		SELECT id, type, source, data, metadata, correlation_id, causation_id, time, data_version, stream_position
		FROM events
		WHERE stream_id = $1 AND stream_position >= $2
		ORDER BY stream_position ASC
	`, streamID, fromVersion)
	if err != nil {
		return nil, fmt.Errorf("failed to query events: %w", err)
	}
	defer rows.Close()

	// Process results
	var events []*event.Event
	for rows.Next() {
		var (
			id, eventType, source, correlationID, causationID, dataVersion string
			dataJSON, metadataJSON                                         []byte
			eventTime                                                      time.Time
			position                                                       int
		)

		err := rows.Scan(
			&id, &eventType, &source, &dataJSON, &metadataJSON, &correlationID, &causationID, &eventTime, &dataVersion, &position,
		)
		if err != nil {
			return nil, fmt.Errorf("failed to scan event: %w", err)
		}

		// Parse data and metadata
		var data map[string]interface{}
		if err := json.Unmarshal(dataJSON, &data); err != nil {
			return nil, fmt.Errorf("failed to unmarshal event data: %w", err)
		}

		var metadata map[string]string
		if metadataJSON != nil {
			if err := json.Unmarshal(metadataJSON, &metadata); err != nil {
				return nil, fmt.Errorf("failed to unmarshal event metadata: %w", err)
			}
		}

		// Create event
		evt := &event.Event{
			ID:            id,
			Type:          eventType,
			Source:        source,
			Time:          eventTime,
			Data:          data,
			DataVersion:   dataVersion,
			Metadata:      metadata,
			CorrelationID: correlationID,
			CausationID:   causationID,
		}
		evt.SetStreamID(streamID)
		evt.SetPosition(position)

		events = append(events, evt)
	}

	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("error iterating events: %w", err)
	}

	return events, nil
}

// LoadByType retrieves events of specific types
func (s *PostgresEventStore) LoadByType(ctx context.Context, eventTypes []string, fromPosition int, limit int) ([]*event.Event, error) {
	// Build query
	query := `
		SELECT id, stream_id, type, source, data, metadata, correlation_id, causation_id, time, data_version, stream_position, global_position
		FROM events
		WHERE type = ANY($1) AND global_position > $2
		ORDER BY global_position ASC
	`
	if limit > 0 {
		query += fmt.Sprintf(" LIMIT %d", limit)
	}

	// Query events
	rows, err := s.db.QueryContext(ctx, query, pq.Array(eventTypes), fromPosition)
	if err != nil {
		return nil, fmt.Errorf("failed to query events: %w", err)
	}
	defer rows.Close()

	// Process results
	var events []*event.Event
	for rows.Next() {
		var (
			id, streamID, eventType, source, correlationID, causationID, dataVersion string
			dataJSON, metadataJSON                                                   []byte
			eventTime                                                                time.Time
			streamPosition, globalPosition                                           int
		)

		err := rows.Scan(
			&id, &streamID, &eventType, &source, &dataJSON, &metadataJSON, &correlationID, &causationID,
			&eventTime, &dataVersion, &streamPosition, &globalPosition,
		)
		if err != nil {
			return nil, fmt.Errorf("failed to scan event: %w", err)
		}

		// Parse data and metadata
		var data map[string]interface{}
		if err := json.Unmarshal(dataJSON, &data); err != nil {
			return nil, fmt.Errorf("failed to unmarshal event data: %w", err)
		}

		var metadata map[string]string
		if metadataJSON != nil {
			if err := json.Unmarshal(metadataJSON, &metadata); err != nil {
				return nil, fmt.Errorf("failed to unmarshal event metadata: %w", err)
			}
		}

		// Create event
		evt := &event.Event{
			ID:            id,
			Type:          eventType,
			Source:        source,
			Time:          eventTime,
			Data:          data,
			DataVersion:   dataVersion,
			Metadata:      metadata,
			CorrelationID: correlationID,
			CausationID:   causationID,
		}
		evt.SetStreamID(streamID)
		evt.SetPosition(streamPosition)
		evt.SetGlobalPosition(globalPosition)

		events = append(events, evt)
	}

	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("error iterating events: %w", err)
	}

	return events, nil
}

// Close closes the database connection
func (s *PostgresEventStore) Close() error {
	return s.db.Close()
}
```

## **26.6 Implementing the Saga Pattern**

The saga pattern coordinates distributed transactions across multiple services using a sequence of events and compensating actions. Let's implement a simple saga in Go.

### **26.6.1 Defining a Saga**

A saga is a sequence of local transactions. Each transaction may result in a decision to continue the saga or to compensate for previous transactions.

```go
// saga/saga.go
package saga

import (
	"context"
	"fmt"
)

// Saga represents a distributed transaction
type Saga struct {
	steps []*Step
}

// NewSaga creates a new saga
func NewSaga(steps ...*Step) *Saga {
	return &Saga{steps: steps}
}

// Execute executes the saga
func (s *Saga) Execute(ctx context.Context) error {
	for _, step := range s.steps {
		if err := step.Execute(ctx); err != nil {
			if err := s.compensate(ctx, step); err != nil {
				return fmt.Errorf("failed to compensate after error: %w", err)
			}
			return err
		}
	}
	return nil
}

// compensate compensates for a failed step
func (s *Saga) compensate(ctx context.Context, failedStep *Step) error {
	for i := len(s.steps) - 1; i >= 0; i-- {
		if s.steps[i].id == failedStep.id {
			if err := s.steps[i].compensate(ctx); err != nil {
				return fmt.Errorf("failed to compensate step %d: %w", i, err)
			}
		}
	}
	return nil
}
```

### **26.6.2 Implementing a Saga Step**

A saga step is a local transaction that may be compensated for if it fails.

```go
// saga/step.go
package saga

import (
	"context"
	"fmt"
)

// Step represents a local transaction in a saga
type Step struct {
	id         string
	execute    func(ctx context.Context) error
	compensate func(ctx context.Context) error
}

// NewStep creates a new saga step
func NewStep(id string, execute func(ctx context.Context) error, compensate func(ctx context.Context) error) *Step {
	return &Step{id: id, execute: execute, compensate: compensate}
}

// Execute executes the step
func (s *Step) Execute(ctx context.Context) error {
	if s.execute == nil {
		return nil
	}
	return s.execute(ctx)
}

// Compensate compensates for the step
func (s *Step) Compensate(ctx context.Context) error {
	if s.compensate == nil {
		return nil
	}
	return s.compensate(ctx)
}
```

### **26.6.3 Putting It All Together**

Let's see how all the saga components work together:

```go
// main.go
package main

import (
	"context"
	"log"

	"github.com/yourorg/eventsystem/bus"
	"github.com/yourorg/eventsystem/domain/user"
	"github.com/yourorg/eventsystem/saga"
	"github.com/yourorg/eventsystem/middleware"
)

func main() {
	// Create the event bus
	eventBus := bus.NewMemoryBus()

	// Add middleware
	eventBus.Use(middleware.Logging())

	// Create the saga
	saga := saga.NewSaga(
		saga.NewStep("step1", func(ctx context.Context) error {
			// Implementation of step1
			return nil
		}, func(ctx context.Context) error {
			// Implementation of step1 compensation
			return nil
		}),
		saga.NewStep("step2", func(ctx context.Context) error {
			// Implementation of step2
			return nil
		}, func(ctx context.Context) error {
			// Implementation of step2 compensation
			return nil
		}),
	)

	// Execute the saga
	ctx := context.Background()
	if err := saga.Execute(ctx); err != nil {
		log.Fatalf("Failed to execute saga: %v", err)
	}
}
```

This foundation provides a solid starting point for building event-driven systems in Go. In the following sections, we'll expand on these concepts to build more sophisticated distributed event systems.

## **26.7 Conclusion**

Event-Driven Architecture represents a powerful paradigm for building modern, distributed systems. Throughout this chapter, we've explored how Go's features make it an excellent language for implementing event-driven systems:

1. **Native Concurrency**: Go's goroutines and channels provide a natural fit for handling concurrent event processing
2. **Strong Type System**: Helps maintain consistency in event schemas and prevents runtime errors
3. **Rich Ecosystem**: The Go ecosystem offers robust libraries for implementing message brokers, event sourcing, and distributed systems

We've covered several key patterns and implementations:

- **Event Bus**: For decoupled communication between components
- **Event Sourcing**: For maintaining a complete history of state changes
- **Command Query Responsibility Segregation (CQRS)**: For optimizing read and write operations
- **Distributed Events**: For communication across service boundaries
- **Saga Pattern**: For coordinating distributed transactions

These patterns enable building systems that are:

- **Scalable**: Components can be scaled independently based on their specific workloads
- **Resilient**: Failures in one component don't necessarily cascade to others
- **Maintainable**: Clear separation of concerns makes the system easier to understand and modify
- **Evolvable**: New functionality can be added by subscribing to existing events without modifying the publishers

When implementing event-driven systems in Go, consider these best practices:

1. **Define Clear Event Schemas**: Use well-defined event schemas with versioning to facilitate evolution
2. **Ensure Idempotent Handlers**: Event handlers should be idempotent to handle duplicate events gracefully
3. **Implement Proper Error Handling**: Use middleware for retry logic, dead-letter queues, and logging
4. **Consider Event Ordering**: When order matters, use strategies like sequence numbers or partitioning
5. **Maintain Observability**: Implement thorough logging, metrics, and tracing to debug complex event flows

Go's simplicity, performance, and concurrency model make it an excellent choice for implementing event-driven architectures, whether for microservices, real-time data processing, or reactive systems. By applying the patterns and implementations covered in this chapter, you can build systems that are not only powerful and scalable but also maintainable over time.

As you move forward with event-driven architectures in Go, remember that the key benefit is not just technical but also organizational. These patterns encourage a mindset of modeling systems around the flow of events and state changes, leading to designs that better reflect the business domains they represent.
