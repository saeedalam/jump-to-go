# **Chapter 27: Domain-Driven Design with Go**

## **27.1 Introduction to Domain-Driven Design**

Domain-Driven Design (DDD) is a software development approach that focuses on creating a deep understanding of the business domain and expressing that understanding in code. Introduced by Eric Evans in his seminal book "Domain-Driven Design: Tackling Complexity in the Heart of Software" (2003), DDD provides a set of patterns, principles, and practices for modeling complex business domains.

Go's simplicity, strong typing, and emphasis on clarity make it an excellent language for implementing DDD. In this chapter, we'll explore how to apply DDD principles and patterns using Go, demonstrating how they can help you build maintainable, expressive, and business-aligned software systems.

### **27.1.1 Core Concepts of Domain-Driven Design**

Let's start by exploring the fundamental concepts that form the foundation of DDD:

#### **Ubiquitous Language**

Ubiquitous Language is a shared language between developers and domain experts. It becomes the vocabulary used in discussions, documentation, and code. By using the same terms consistently across all contexts, we reduce translation errors and build a model that accurately reflects the business domain.

For example, in an e-commerce system:

- Rather than generic terms like "user" or "item," we use domain-specific terms like "customer," "product," and "order"
- Instead of technical terms like "persist" or "fetch," we use business processes like "place order" or "ship product"

#### **Bounded Contexts**

A Bounded Context is a boundary within which a particular domain model applies. Different parts of a large system may have different models for what superficially appears to be the same concept. For example:

- In a sales context, a "customer" might include credit limits and purchasing history
- In a support context, a "customer" might include support tickets and service level agreements

Bounded Contexts help manage complexity by dividing a large domain into smaller, more manageable chunks with clear boundaries.

#### **Context Mapping**

Context Mapping defines the relationships between different Bounded Contexts. These relationships can take various forms:

- **Partnership**: Two teams collaborate closely on integration
- **Shared Kernel**: Sharing a subset of the domain model
- **Customer-Supplier**: Upstream/downstream relationship with aligned planning
- **Conformist**: Downstream team conforms to upstream team's model
- **Anticorruption Layer**: A translation layer that protects one model from another
- **Open Host Service**: A protocol or interface that provides access to a subsystem
- **Published Language**: A well-documented information exchange format

#### **Model-Driven Design**

Model-Driven Design means that our code should directly express the domain model. The goal is to create a model that:

- Captures essential domain knowledge
- Focuses on the core domain (the part with the most business value)
- Is continuously refined through collaboration with domain experts

### **27.1.2 Strategic vs. Tactical DDD**

DDD is often divided into strategic and tactical aspects:

**Strategic DDD** focuses on the big picture:

- Defining the domain and its subdomains
- Identifying Bounded Contexts
- Establishing Context Maps
- Distilling the Core Domain

**Tactical DDD** focuses on implementation patterns:

- Entities and Value Objects
- Aggregates and Aggregate Roots
- Repositories
- Domain Events
- Services
- Factories

In this chapter, we'll explore both aspects, starting with the strategic patterns that help structure your application, and then diving into tactical patterns that guide the implementation details.

### **27.1.3 Benefits of Domain-Driven Design**

When applied correctly, DDD offers numerous benefits:

- **Alignment with Business**: The software model closely reflects the business domain
- **Handling Complexity**: Strategies for managing complex domains without getting overwhelmed
- **Knowledge Sharing**: A ubiquitous language facilitates communication between technical and domain experts
- **Flexible Architecture**: Clear boundaries enable changes within contexts without affecting the entire system
- **Focus on Value**: Effort is directed toward the core domain where the most business value lies

### **27.1.4 When to Use DDD**

DDD is particularly valuable when:

- You're working on a complex domain with non-trivial business rules
- The project involves collaboration between technical experts and domain experts
- The solution is expected to evolve over time as the domain understanding deepens
- The business logic is more complex than the technical challenges

DDD may be less appropriate for:

- Simple CRUD applications with minimal business logic
- Technical infrastructure with limited domain complexity
- Short-lived projects where the investment in modeling may not pay off

With this foundation in place, let's explore how to implement DDD principles and patterns in Go.

## **27.2 Strategic Domain-Driven Design in Go**

Strategic DDD helps us understand and structure the problem space before diving into implementation. It involves analyzing the domain, identifying subdomains, establishing bounded contexts, and defining their relationships. Let's explore how to apply these concepts in Go projects.

### **27.2.1 Domain Analysis and Ubiquitous Language**

The first step in a DDD approach is to collaborate with domain experts to understand the business domain and establish a ubiquitous language. This involves:

1. **Knowledge Crunching**: Intensive sessions with domain experts to extract and distill domain knowledge
2. **Documenting the Ubiquitous Language**: Creating a glossary of domain terms and their definitions
3. **Using the Language Consistently**: In discussions, documentation, and code

In Go, we can reflect the ubiquitous language through thoughtful naming of packages, types, and functions. Go's emphasis on clear, readable code aligns well with this goal.

Let's create a simple glossary for an e-commerce domain as an example:

```go
// domain/glossary.go
package domain

/*
Ubiquitous Language for E-commerce System

- Product: An item available for purchase in the catalog
- Catalog: The complete collection of products available for sale
- Customer: A person who places orders in the system
- Order: A request to purchase one or more products
- Cart: A temporary collection of products a customer intends to purchase
- Checkout: The process of converting a cart to an order
- Payment: A transaction where money is exchanged for products
- Shipping: The process of delivering products to a customer
- Inventory: The quantity of products available for sale
- Discount: A reduction in price applied to products or orders
*/
```

While this isn't functional code, keeping such documentation alongside your codebase helps maintain alignment with the domain.

### **27.2.2 Identifying Bounded Contexts**

After understanding the domain, the next step is to identify bounded contexts. These are logical boundaries within which specific models apply.

In Go, we can represent bounded contexts through package structure. Each context can be a separate module or a distinct package hierarchy.

Here's an example package structure for an e-commerce system with multiple bounded contexts:

```
e-commerce/
├── catalog/         # Catalog Bounded Context
│   ├── domain/
│   ├── application/
│   └── infrastructure/
├── ordering/        # Ordering Bounded Context
│   ├── domain/
│   ├── application/
│   └── infrastructure/
├── customers/       # Customer Bounded Context
│   ├── domain/
│   ├── application/
│   └── infrastructure/
├── shipping/        # Shipping Bounded Context
│   ├── domain/
│   ├── application/
│   └── infrastructure/
└── payments/        # Payment Bounded Context
    ├── domain/
    ├── application/
    └── infrastructure/
```

Each bounded context contains its own domain models, application services, and infrastructure components. This separation helps manage complexity and allows each context to evolve independently.

### **27.2.3 Context Mapping in Go**

Context mapping defines the relationships between bounded contexts. In Go, we can implement these relationships in various ways:

#### **1. Partnership with Shared Interfaces**

When two contexts need to collaborate closely, they can define shared interfaces:

```go
// shipping/domain/interfaces.go
package domain

import "time"

// OrderRepository defines methods needed by the Shipping context
// that must be implemented by the Ordering context
type OrderRepository interface {
    GetOrderByID(orderID string) (*Order, error)
    UpdateOrderStatus(orderID string, status string) error
}

// Order represents the minimal order information needed by Shipping
type Order struct {
    ID          string
    CustomerID  string
    Items       []OrderItem
    ShippingAddress Address
    PlacedAt    time.Time
}

type OrderItem struct {
    ProductID   string
    Quantity    int
    Weight      float64
}

type Address struct {
    Street      string
    City        string
    State       string
    PostalCode  string
    Country     string
}
```

The ordering context would then implement this interface, establishing a clear contract between contexts.

#### **2. Anticorruption Layer**

When integrating with an external system or a context with a different model, we can use an anticorruption layer to protect our domain model:

```go
// payments/infrastructure/external_payment_gateway.go
package infrastructure

import (
    "github.com/e-commerce/payments/domain"
    "github.com/external-payment-gateway/client"
)

// ExternalPaymentGatewayAdapter is an anticorruption layer that
// translates between our domain model and the external payment gateway
type ExternalPaymentGatewayAdapter struct {
    client *client.PaymentGatewayClient
}

func NewExternalPaymentGatewayAdapter(apiKey string) *ExternalPaymentGatewayAdapter {
    return &ExternalPaymentGatewayAdapter{
        client: client.NewPaymentGatewayClient(apiKey),
    }
}

// ProcessPayment converts our domain payment to the external format,
// makes the API call, and converts the response back to our domain model
func (a *ExternalPaymentGatewayAdapter) ProcessPayment(payment *domain.Payment) (*domain.PaymentResult, error) {
    // Convert from our domain model to external format
    externalPayment := &client.PaymentRequest{
        Amount:      payment.Amount.Value(),
        Currency:    payment.Amount.Currency(),
        CardNumber:  payment.CreditCard.Number(),
        ExpiryMonth: payment.CreditCard.ExpiryMonth(),
        ExpiryYear:  payment.CreditCard.ExpiryYear(),
        CVV:         payment.CreditCard.CVV(),
        Description: payment.Description,
    }

    // Call external service
    response, err := a.client.ProcessPayment(externalPayment)
    if err != nil {
        return nil, err
    }

    // Convert response back to our domain model
    result := &domain.PaymentResult{
        TransactionID: response.TransactionID,
        Success:       response.Status == "approved",
        Status:        mapStatus(response.Status),
        ErrorMessage:  response.ErrorMessage,
        ProcessedAt:   response.Timestamp,
    }

    return result, nil
}

// mapStatus converts external payment status to our domain status
func mapStatus(externalStatus string) domain.PaymentStatus {
    switch externalStatus {
    case "approved":
        return domain.PaymentStatusApproved
    case "declined":
        return domain.PaymentStatusDeclined
    case "pending":
        return domain.PaymentStatusPending
    default:
        return domain.PaymentStatusFailed
    }
}
```

This adapter translates between our clean domain model and the external system's model, protecting our domain from being corrupted by external concepts.

#### **3. Published Language with Go Structs**

When multiple contexts need to share data, a published language (shared data format) can be defined using Go structs:

```go
// shared/events/order_events.go
package events

import "time"

// OrderPlaced is a published language event shared between
// the Ordering context and other contexts like Shipping and Billing
type OrderPlaced struct {
    OrderID        string    `json:"order_id"`
    CustomerID     string    `json:"customer_id"`
    TotalAmount    float64   `json:"total_amount"`
    CurrencyCode   string    `json:"currency_code"`
    ShippingAddress Address   `json:"shipping_address"`
    Items          []OrderItem `json:"items"`
    PlacedAt       time.Time `json:"placed_at"`
}

type Address struct {
    Street      string `json:"street"`
    City        string `json:"city"`
    State       string `json:"state"`
    PostalCode  string `json:"postal_code"`
    Country     string `json:"country"`
}

type OrderItem struct {
    ProductID   string  `json:"product_id"`
    Quantity    int     `json:"quantity"`
    UnitPrice   float64 `json:"unit_price"`
}
```

These event structs form a published language that different contexts can use to communicate without tight coupling.

### **27.2.4 Subdomain Types in Go**

DDD distinguishes between different types of subdomains based on their business value and complexity:

#### **Core Domain**

The core domain represents the most valuable and differentiating part of your business. In Go, we should invest the most effort in modeling this domain accurately:

```go
// ordering/domain/order.go
package domain

import (
    "errors"
    "time"

    "github.com/google/uuid"
)

// Order represents the core domain entity in the ordering context
type Order struct {
    id             string
    customerID     string
    items          []OrderItem
    shippingAddress Address
    billingAddress  Address
    status         OrderStatus
    placedAt       time.Time
    total          Money
    discounts      []Discount
}

// NewOrder creates a new order with validation
func NewOrder(customerID string, items []OrderItem, shipping, billing Address) (*Order, error) {
    if customerID == "" {
        return nil, errors.New("customer ID cannot be empty")
    }
    if len(items) == 0 {
        return nil, errors.New("order must contain at least one item")
    }

    // Calculate total
    var total Money
    for _, item := range items {
        itemTotal := item.UnitPrice.Multiply(item.Quantity)
        total = total.Add(itemTotal)
    }

    return &Order{
        id:             uuid.New().String(),
        customerID:     customerID,
        items:          items,
        shippingAddress: shipping,
        billingAddress:  billing,
        status:         OrderStatusDraft,
        placedAt:       time.Now().UTC(),
        total:          total,
        discounts:      []Discount{},
    }, nil
}

// AddDiscount applies a discount to the order
func (o *Order) AddDiscount(discount Discount) error {
    if o.status != OrderStatusDraft {
        return errors.New("discounts can only be added to draft orders")
    }

    o.discounts = append(o.discounts, discount)

    // Recalculate total
    o.recalculateTotal()

    return nil
}

// Place transitions the order to placed status
func (o *Order) Place() error {
    if o.status != OrderStatusDraft {
        return errors.New("only draft orders can be placed")
    }

    o.status = OrderStatusPlaced
    o.placedAt = time.Now().UTC()

    return nil
}

// Cancel cancels the order if possible
func (o *Order) Cancel(reason string) error {
    if o.status == OrderStatusShipped || o.status == OrderStatusDelivered {
        return errors.New("shipped or delivered orders cannot be cancelled")
    }

    o.status = OrderStatusCancelled

    return nil
}

// recalculateTotal updates the order total after discounts
func (o *Order) recalculateTotal() {
    // Start with subtotal
    var subtotal Money
    for _, item := range o.items {
        itemTotal := item.UnitPrice.Multiply(item.Quantity)
        subtotal = subtotal.Add(itemTotal)
    }

    // Apply discounts
    var discountTotal Money
    for _, discount := range o.discounts {
        discountAmount := discount.CalculateDiscount(subtotal)
        discountTotal = discountTotal.Add(discountAmount)
    }

    // Final total
    o.total = subtotal.Subtract(discountTotal)
}

// Getters
func (o *Order) ID() string { return o.id }
func (o *Order) CustomerID() string { return o.customerID }
func (o *Order) Items() []OrderItem { return o.items }
func (o *Order) ShippingAddress() Address { return o.shippingAddress }
func (o *Order) BillingAddress() Address { return o.billingAddress }
func (o *Order) Status() OrderStatus { return o.status }
func (o *Order) PlacedAt() time.Time { return o.placedAt }
func (o *Order) Total() Money { return o.total }
func (o *Order) Discounts() []Discount { return o.discounts }
```

Notice the careful validation, business rules, and rich behavior in this core domain model.

#### **Supporting Subdomain**

Supporting subdomains are important but not differentiating. They might be implemented with less complexity:

```go
// catalog/domain/product.go
package domain

// Product represents a product in the catalog
type Product struct {
    ID          string
    Name        string
    Description string
    Price       Money
    Categories  []string
    Attributes  map[string]string
    Active      bool
}

// NewProduct creates a new product
func NewProduct(id, name string, price Money) *Product {
    return &Product{
        ID:          id,
        Name:        name,
        Price:       price,
        Categories:  []string{},
        Attributes:  make(map[string]string),
        Active:      true,
    }
}

// AddCategory adds a category to the product
func (p *Product) AddCategory(category string) {
    // Simple check for duplicates
    for _, c := range p.Categories {
        if c == category {
            return
        }
    }
    p.Categories = append(p.Categories, category)
}

// SetAttribute sets a product attribute
func (p *Product) SetAttribute(key, value string) {
    p.Attributes[key] = value
}

// Deactivate removes the product from active catalog
func (p *Product) Deactivate() {
    p.Active = false
}

// Activate adds the product to active catalog
func (p *Product) Activate() {
    p.Active = true
}
```

This model is simpler than the core domain model but still encapsulates the essential behavior.

#### **Generic Subdomain**

Generic subdomains represent functionality that is not unique to your business and could potentially be outsourced or replaced with off-the-shelf solutions. These can be implemented more simply:

```go
// notifications/domain/email.go
package domain

import "time"

// EmailNotification represents an email to be sent
type EmailNotification struct {
    To        string
    From      string
    Subject   string
    Body      string
    HTML      bool
    SentAt    time.Time
    DeliveryStatus string
}

// NewEmailNotification creates a new email notification
func NewEmailNotification(to, from, subject, body string, html bool) *EmailNotification {
    return &EmailNotification{
        To:      to,
        From:    from,
        Subject: subject,
        Body:    body,
        HTML:    html,
    }
}

// MarkAsSent records when the email was sent
func (e *EmailNotification) MarkAsSent() {
    e.SentAt = time.Now().UTC()
    e.DeliveryStatus = "sent"
}

// MarkAsDelivered records that the email was delivered
func (e *EmailNotification) MarkAsDelivered() {
    e.DeliveryStatus = "delivered"
}

// MarkAsFailed records that the email delivery failed
func (e *EmailNotification) MarkAsFailed() {
    e.DeliveryStatus = "failed"
}
```

This model is relatively simple because email notification is a generic feature that isn't central to the business differentiators.

### **27.2.5 Documenting the Strategic Design**

To communicate the strategic design, we can create documentation in our codebase:

```go
// docs/strategic_design.go
package docs

/*
Strategic Design for E-commerce System

Domain: E-commerce Platform

Core Subdomains:
- Ordering: The core business process of taking and fulfilling orders
- Catalog: Management of the product catalog and inventory

Supporting Subdomains:
- Customer Management: Registration, profiles, and customer data
- Shipping: Shipping options, carriers, and delivery tracking
- Pricing: Price calculation, discounts, and promotions

Generic Subdomains:
- Notifications: Email and SMS notifications
- Authentication: User authentication and authorization
- Payments: Processing payments via external providers

Bounded Contexts:
1. Ordering Context
   - Entities: Order, OrderItem, Cart
   - Responsibilities: Order lifecycle, cart management

2. Catalog Context
   - Entities: Product, Category, Inventory
   - Responsibilities: Product information, categorization, inventory tracking

3. Customer Context
   - Entities: Customer, Address, CustomerPreference
   - Responsibilities: Customer profiles and history

4. Shipping Context
   - Entities: Shipment, Carrier, DeliveryOption
   - Responsibilities: Shipping rates, tracking, delivery management

5. Payment Context
   - Entities: Payment, PaymentMethod, Transaction
   - Responsibilities: Payment processing, refunds

Context Relationships:
- Ordering → Catalog: Conformist (Ordering uses Catalog's product model)
- Ordering → Customer: Customer/Supplier (Ordering depends on Customer)
- Ordering → Shipping: Partnership (Close collaboration)
- Ordering → Payment: Anticorruption Layer (Protects from external payment providers)
*/
```

This documentation provides a clear overview of the strategic design, helping team members understand the system structure.

By applying these strategic DDD concepts in Go, we create a solid foundation for our software. The clear boundaries and relationships between contexts help manage complexity and allow for independent evolution of different parts of the system.

## **27.3 Tactical Domain-Driven Design in Go**

Tactical DDD focuses on the implementation patterns that help express the domain model effectively in code. Let's explore how to implement these patterns in Go.

### **27.3.1 Value Objects**

Value Objects are immutable objects that describe aspects of the domain with no conceptual identity. They are defined by their attributes rather than by a unique identifier.

Go's strong type system and support for custom types make it an excellent language for implementing Value Objects. Let's look at some examples:

#### **Money Value Object**

```go
// domain/money.go
package domain

import (
	"errors"
	"fmt"
)

// Money represents a monetary value with currency
type Money struct {
	amount   int64  // Amount in smallest currency unit (e.g., cents)
	currency string // Currency code (e.g., "USD")
}

// NewMoney creates a new Money value object
func NewMoney(amount int64, currency string) (Money, error) {
	if currency == "" {
		return Money{}, errors.New("currency cannot be empty")
	}

	return Money{
		amount:   amount,
		currency: currency,
	}, nil
}

// MustNewMoney creates a new Money value object and panics on error
func MustNewMoney(amount int64, currency string) Money {
	m, err := NewMoney(amount, currency)
	if err != nil {
		panic(err)
	}
	return m
}

// Amount returns the amount in the smallest currency unit
func (m Money) Amount() int64 {
	return m.amount
}

// Currency returns the currency code
func (m Money) Currency() string {
	return m.currency
}

// IsZero returns true if the amount is zero
func (m Money) IsZero() bool {
	return m.amount == 0
}

// IsPositive returns true if the amount is positive
func (m Money) IsPositive() bool {
	return m.amount > 0
}

// IsNegative returns true if the amount is negative
func (m Money) IsNegative() bool {
	return m.amount < 0
}

// String returns a string representation of the money value
func (m Money) String() string {
	return fmt.Sprintf("%s %.2f", m.currency, float64(m.amount)/100.0)
}

// Add adds another Money value and returns a new Money value
func (m Money) Add(other Money) (Money, error) {
	if m.currency != other.currency {
		return Money{}, fmt.Errorf(
			"cannot add money with different currencies: %s and %s",
			m.currency, other.currency,
		)
	}

	return Money{
		amount:   m.amount + other.amount,
		currency: m.currency,
	}, nil
}

// Subtract subtracts another Money value and returns a new Money value
func (m Money) Subtract(other Money) (Money, error) {
	if m.currency != other.currency {
		return Money{}, fmt.Errorf(
			"cannot subtract money with different currencies: %s and %s",
			m.currency, other.currency,
		)
	}

	return Money{
		amount:   m.amount - other.amount,
		currency: m.currency,
	}, nil
}

// Multiply multiplies the Money value by a factor and returns a new Money value
func (m Money) Multiply(factor float64) Money {
	return Money{
		amount:   int64(float64(m.amount) * factor),
		currency: m.currency,
	}
}

// Equals returns true if the Money values are equal
func (m Money) Equals(other Money) bool {
	return m.amount == other.amount && m.currency == other.currency
}
```

This Money value object is immutable and encapsulates operations on monetary values, ensuring that business rules (like preventing addition of different currencies) are enforced.

#### **Email Value Object**

```go
// domain/email.go
package domain

import (
	"errors"
	"regexp"
	"strings"
)

// Email represents an email address
type Email struct {
	address string
}

// Regular expression for validating email format
var emailRegex = regexp.MustCompile(`^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$`)

// NewEmail creates a new Email value object
func NewEmail(address string) (Email, error) {
	// Trim spaces
	address = strings.TrimSpace(address)

	// Check if empty
	if address == "" {
		return Email{}, errors.New("email address cannot be empty")
	}

	// Convert to lowercase
	address = strings.ToLower(address)

	// Validate format
	if !emailRegex.MatchString(address) {
		return Email{}, errors.New("invalid email format")
	}

	return Email{
		address: address,
	}, nil
}

// MustNewEmail creates a new Email value object and panics on error
func MustNewEmail(address string) Email {
	email, err := NewEmail(address)
	if err != nil {
		panic(err)
	}
	return email
}

// Address returns the email address
func (e Email) Address() string {
	return e.address
}

// Domain returns the domain part of the email
func (e Email) Domain() string {
	parts := strings.Split(e.address, "@")
	return parts[1]
}

// Username returns the username part of the email
func (e Email) Username() string {
	parts := strings.Split(e.address, "@")
	return parts[0]
}

// String returns the string representation of the email
func (e Email) String() string {
	return e.address
}

// Equals checks if two emails are equal
func (e Email) Equals(other Email) bool {
	return e.address == other.address
}
```

This Email value object encapsulates validation and operations on email addresses, ensuring that only valid email addresses can be created.

#### **Address Value Object**

```go
// domain/address.go
package domain

import (
	"errors"
	"strings"
)

// Address represents a physical address
type Address struct {
	street     string
	city       string
	state      string
	postalCode string
	country    string
}

// NewAddress creates a new Address value object
func NewAddress(street, city, state, postalCode, country string) (Address, error) {
	// Validate required fields
	if strings.TrimSpace(street) == "" {
		return Address{}, errors.New("street cannot be empty")
	}
	if strings.TrimSpace(city) == "" {
		return Address{}, errors.New("city cannot be empty")
	}
	if strings.TrimSpace(postalCode) == "" {
		return Address{}, errors.New("postal code cannot be empty")
	}
	if strings.TrimSpace(country) == "" {
		return Address{}, errors.New("country cannot be empty")
	}

	return Address{
		street:     street,
		city:       city,
		state:      state,
		postalCode: postalCode,
		country:    country,
	}, nil
}

// Street returns the street
func (a Address) Street() string {
	return a.street
}

// City returns the city
func (a Address) City() string {
	return a.city
}

// State returns the state
func (a Address) State() string {
	return a.state
}

// PostalCode returns the postal code
func (a Address) PostalCode() string {
	return a.postalCode
}

// Country returns the country
func (a Address) Country() string {
	return a.country
}

// Format returns a formatted string representation of the address
func (a Address) Format() string {
	parts := []string{a.street, a.city}
	if a.state != "" {
		parts = append(parts, a.state)
	}
	parts = append(parts, a.postalCode, a.country)
	return strings.Join(parts, ", ")
}

// Equals checks if two addresses are equal
func (a Address) Equals(other Address) bool {
	return a.street == other.street &&
		a.city == other.city &&
		a.state == other.state &&
		a.postalCode == other.postalCode &&
		a.country == other.country
}
```

This Address value object encapsulates validation and operations on physical addresses, ensuring that addresses have the required fields.

### **27.3.2 Entities**

Entities are objects defined by their identity rather than their attributes. They have a lifecycle and may change over time. In Go, we can implement entities using structs with methods that enforce business rules.

#### **Customer Entity**

```go
// domain/customer.go
package domain

import (
	"errors"
	"time"

	"github.com/google/uuid"
)

// Customer represents a customer entity
type Customer struct {
	id        string
	email     Email
	name      string
	addresses map[string]Address
	createdAt time.Time
	updatedAt time.Time
}

// NewCustomer creates a new customer entity
func NewCustomer(email Email, name string) (*Customer, error) {
	if name == "" {
		return nil, errors.New("name cannot be empty")
	}

	return &Customer{
		id:        uuid.New().String(),
		email:     email,
		name:      name,
		addresses: make(map[string]Address),
		createdAt: time.Now().UTC(),
		updatedAt: time.Now().UTC(),
	}, nil
}

// ID returns the customer ID
func (c *Customer) ID() string {
	return c.id
}

// Email returns the customer email
func (c *Customer) Email() Email {
	return c.email
}

// Name returns the customer name
func (c *Customer) Name() string {
	return c.name
}

// CreatedAt returns when the customer was created
func (c *Customer) CreatedAt() time.Time {
	return c.createdAt
}

// UpdatedAt returns when the customer was last updated
func (c *Customer) UpdatedAt() time.Time {
	return c.updatedAt
}

// UpdateEmail updates the customer's email
func (c *Customer) UpdateEmail(email Email) {
	c.email = email
	c.updatedAt = time.Now().UTC()
}

// UpdateName updates the customer's name
func (c *Customer) UpdateName(name string) error {
	if name == "" {
		return errors.New("name cannot be empty")
	}

	c.name = name
	c.updatedAt = time.Now().UTC()
	return nil
}

// AddAddress adds a new address with a label
func (c *Customer) AddAddress(label string, address Address) error {
	if label == "" {
		return errors.New("address label cannot be empty")
	}

	c.addresses[label] = address
	c.updatedAt = time.Now().UTC()
	return nil
}

// GetAddress retrieves an address by label
func (c *Customer) GetAddress(label string) (Address, bool) {
	address, exists := c.addresses[label]
	return address, exists
}

// RemoveAddress removes an address by label
func (c *Customer) RemoveAddress(label string) {
	delete(c.addresses, label)
	c.updatedAt = time.Now().UTC()
}

// AddressLabels returns all address labels
func (c *Customer) AddressLabels() []string {
	labels := make([]string, 0, len(c.addresses))
	for label := range c.addresses {
		labels = append(labels, label)
	}
	return labels
}
```

This Customer entity has a unique identifier and methods that enforce business rules, such as preventing empty names.

#### **Product Entity**

```go
// domain/product.go
package domain

import (
	"errors"
	"time"

	"github.com/google/uuid"
)

// Product represents a product entity
type Product struct {
	id          string
	name        string
	description string
	price       Money
	sku         string
	categories  []string
	attributes  map[string]string
	createdAt   time.Time
	updatedAt   time.Time
	active      bool
}

// NewProduct creates a new product entity
func NewProduct(name string, description string, price Money, sku string) (*Product, error) {
	if name == "" {
		return nil, errors.New("product name cannot be empty")
	}
	if sku == "" {
		return nil, errors.New("product SKU cannot be empty")
	}

	return &Product{
		id:          uuid.New().String(),
		name:        name,
		description: description,
		price:       price,
		sku:         sku,
		categories:  []string{},
		attributes:  make(map[string]string),
		createdAt:   time.Now().UTC(),
		updatedAt:   time.Now().UTC(),
		active:      true,
	}, nil
}

// ID returns the product ID
func (p *Product) ID() string {
	return p.id
}

// Name returns the product name
func (p *Product) Name() string {
	return p.name
}

// Description returns the product description
func (p *Product) Description() string {
	return p.description
}

// Price returns the product price
func (p *Product) Price() Money {
	return p.price
}

// SKU returns the product SKU
func (p *Product) SKU() string {
	return p.sku
}

// Categories returns the product categories
func (p *Product) Categories() []string {
	// Return a copy to prevent modification of internal state
	categories := make([]string, len(p.categories))
	copy(categories, p.categories)
	return categories
}

// Attributes returns the product attributes
func (p *Product) Attributes() map[string]string {
	// Return a copy to prevent modification of internal state
	attributes := make(map[string]string, len(p.attributes))
	for k, v := range p.attributes {
		attributes[k] = v
	}
	return attributes
}

// CreatedAt returns when the product was created
func (p *Product) CreatedAt() time.Time {
	return p.createdAt
}

// UpdatedAt returns when the product was last updated
func (p *Product) UpdatedAt() time.Time {
	return p.updatedAt
}

// IsActive returns whether the product is active
func (p *Product) IsActive() bool {
	return p.active
}

// UpdateName updates the product name
func (p *Product) UpdateName(name string) error {
	if name == "" {
		return errors.New("product name cannot be empty")
	}

	p.name = name
	p.updatedAt = time.Now().UTC()
	return nil
}

// UpdateDescription updates the product description
func (p *Product) UpdateDescription(description string) {
	p.description = description
	p.updatedAt = time.Now().UTC()
}

// UpdatePrice updates the product price
func (p *Product) UpdatePrice(price Money) {
	p.price = price
	p.updatedAt = time.Now().UTC()
}

// AddCategory adds a category to the product
func (p *Product) AddCategory(category string) {
	// Check if category already exists
	for _, c := range p.categories {
		if c == category {
			return
		}
	}

	p.categories = append(p.categories, category)
	p.updatedAt = time.Now().UTC()
}

// RemoveCategory removes a category from the product
func (p *Product) RemoveCategory(category string) {
	for i, c := range p.categories {
		if c == category {
			// Remove element at index i
			p.categories = append(p.categories[:i], p.categories[i+1:]...)
			p.updatedAt = time.Now().UTC()
			return
		}
	}
}

// SetAttribute sets a product attribute
func (p *Product) SetAttribute(key, value string) {
	p.attributes[key] = value
	p.updatedAt = time.Now().UTC()
}

// RemoveAttribute removes a product attribute
func (p *Product) RemoveAttribute(key string) {
	delete(p.attributes, key)
	p.updatedAt = time.Now().UTC()
}

// Activate activates the product
func (p *Product) Activate() {
	if !p.active {
		p.active = true
		p.updatedAt = time.Now().UTC()
	}
}

// Deactivate deactivates the product
func (p *Product) Deactivate() {
	if p.active {
		p.active = false
		p.updatedAt = time.Now().UTC()
	}
}
```

This Product entity encapsulates the behavior and attributes of a product, with methods to manage its lifecycle.

### **27.3.3 Aggregates**

Aggregates are clusters of domain objects that are treated as a single unit. They help maintain consistency boundaries in the domain. An aggregate has a root entity, called the Aggregate Root, which serves as the entry point to the aggregate.

Let's implement an Order aggregate, which contains OrderItems:

```go
// domain/order.go
package domain

import (
	"errors"
	"time"

	"github.com/google/uuid"
)

// OrderStatus represents the status of an order
type OrderStatus string

const (
	OrderStatusDraft     OrderStatus = "draft"
	OrderStatusPlaced    OrderStatus = "placed"
	OrderStatusPaid      OrderStatus = "paid"
	OrderStatusShipped   OrderStatus = "shipped"
	OrderStatusDelivered OrderStatus = "delivered"
	OrderStatusCancelled OrderStatus = "cancelled"
)

// OrderItem represents an item in an order
type OrderItem struct {
	productID   string
	productName string
	quantity    int
	unitPrice   Money
}

// NewOrderItem creates a new order item
func NewOrderItem(productID, productName string, quantity int, unitPrice Money) (OrderItem, error) {
	if productID == "" {
		return OrderItem{}, errors.New("product ID cannot be empty")
	}
	if productName == "" {
		return OrderItem{}, errors.New("product name cannot be empty")
	}
	if quantity <= 0 {
		return OrderItem{}, errors.New("quantity must be positive")
	}

	return OrderItem{
		productID:   productID,
		productName: productName,
		quantity:    quantity,
		unitPrice:   unitPrice,
	}, nil
}

// ProductID returns the product ID
func (i OrderItem) ProductID() string {
	return i.productID
}

// ProductName returns the product name
func (i OrderItem) ProductName() string {
	return i.productName
}

// Quantity returns the quantity
func (i OrderItem) Quantity() int {
	return i.quantity
}

// UnitPrice returns the unit price
func (i OrderItem) UnitPrice() Money {
	return i.unitPrice
}

// Total returns the total price for this item
func (i OrderItem) Total() Money {
	return i.unitPrice.Multiply(float64(i.quantity))
}

// Order represents an order aggregate
type Order struct {
	id              string
	customerID      string
	items           []OrderItem
	shippingAddress Address
	billingAddress  Address
	status          OrderStatus
	placedAt        time.Time
	updatedAt       time.Time
	total           Money
}

// NewOrder creates a new order
func NewOrder(customerID string, shippingAddress, billingAddress Address) (*Order, error) {
	if customerID == "" {
		return nil, errors.New("customer ID cannot be empty")
	}

	return &Order{
		id:              uuid.New().String(),
		customerID:      customerID,
		items:           []OrderItem{},
		shippingAddress: shippingAddress,
		billingAddress:  billingAddress,
		status:          OrderStatusDraft,
		updatedAt:       time.Now().UTC(),
		total:           MustNewMoney(0, "USD"), // Default currency
	}, nil
}

// ID returns the order ID
func (o *Order) ID() string {
	return o.id
}

// CustomerID returns the customer ID
func (o *Order) CustomerID() string {
	return o.customerID
}

// Items returns a copy of the order items
func (o *Order) Items() []OrderItem {
	items := make([]OrderItem, len(o.items))
	copy(items, o.items)
	return items
}

// ShippingAddress returns the shipping address
func (o *Order) ShippingAddress() Address {
	return o.shippingAddress
}

// BillingAddress returns the billing address
func (o *Order) BillingAddress() Address {
	return o.billingAddress
}

// Status returns the order status
func (o *Order) Status() OrderStatus {
	return o.status
}

// PlacedAt returns when the order was placed
func (o *Order) PlacedAt() time.Time {
	return o.placedAt
}

// UpdatedAt returns when the order was last updated
func (o *Order) UpdatedAt() time.Time {
	return o.updatedAt
}

// Total returns the order total
func (o *Order) Total() Money {
	return o.total
}

// AddItem adds an item to the order
func (o *Order) AddItem(item OrderItem) error {
	if o.status != OrderStatusDraft {
		return errors.New("cannot add items to a non-draft order")
	}

	// Check if we already have this product in the order
	for i, existingItem := range o.items {
		if existingItem.productID == item.productID {
			// Update quantity instead of adding a new item
			newQuantity := existingItem.quantity + item.quantity
			newItem, err := NewOrderItem(
				existingItem.productID,
				existingItem.productName,
				newQuantity,
				existingItem.unitPrice,
			)
			if err != nil {
				return err
			}

			o.items[i] = newItem
			o.recalculateTotal()
			o.updatedAt = time.Now().UTC()
			return nil
		}
	}

	// Add as new item
	o.items = append(o.items, item)
	o.recalculateTotal()
	o.updatedAt = time.Now().UTC()
	return nil
}

// RemoveItem removes an item from the order
func (o *Order) RemoveItem(productID string) error {
	if o.status != OrderStatusDraft {
		return errors.New("cannot remove items from a non-draft order")
	}

	for i, item := range o.items {
		if item.productID == productID {
			// Remove item at index i
			o.items = append(o.items[:i], o.items[i+1:]...)
			o.recalculateTotal()
			o.updatedAt = time.Now().UTC()
			return nil
		}
	}

	return errors.New("item not found in order")
}

// UpdateItemQuantity updates the quantity of an item
func (o *Order) UpdateItemQuantity(productID string, quantity int) error {
	if o.status != OrderStatusDraft {
		return errors.New("cannot update items in a non-draft order")
	}

	if quantity <= 0 {
		return o.RemoveItem(productID)
	}

	for i, item := range o.items {
		if item.productID == productID {
			newItem, err := NewOrderItem(
				item.productID,
				item.productName,
				quantity,
				item.unitPrice,
			)
			if err != nil {
				return err
			}

			o.items[i] = newItem
			o.recalculateTotal()
			o.updatedAt = time.Now().UTC()
			return nil
		}
	}

	return errors.New("item not found in order")
}

// Place places the order
func (o *Order) Place() error {
	if o.status != OrderStatusDraft {
		return errors.New("only draft orders can be placed")
	}

	if len(o.items) == 0 {
		return errors.New("cannot place an empty order")
	}

	o.status = OrderStatusPlaced
	o.placedAt = time.Now().UTC()
	o.updatedAt = time.Now().UTC()
	return nil
}

// MarkAsPaid marks the order as paid
func (o *Order) MarkAsPaid() error {
	if o.status != OrderStatusPlaced {
		return errors.New("only placed orders can be marked as paid")
	}

	o.status = OrderStatusPaid
	o.updatedAt = time.Now().UTC()
	return nil
}

// MarkAsShipped marks the order as shipped
func (o *Order) MarkAsShipped() error {
	if o.status != OrderStatusPaid {
		return errors.New("only paid orders can be marked as shipped")
	}

	o.status = OrderStatusShipped
	o.updatedAt = time.Now().UTC()
	return nil
}

// MarkAsDelivered marks the order as delivered
func (o *Order) MarkAsDelivered() error {
	if o.status != OrderStatusShipped {
		return errors.New("only shipped orders can be marked as delivered")
	}

	o.status = OrderStatusDelivered
	o.updatedAt = time.Now().UTC()
	return nil
}

// Cancel cancels the order
func (o *Order) Cancel() error {
	if o.status == OrderStatusShipped || o.status == OrderStatusDelivered {
		return errors.New("shipped or delivered orders cannot be cancelled")
	}

	o.status = OrderStatusCancelled
	o.updatedAt = time.Now().UTC()
	return nil
}

// recalculateTotal recalculates the order total
func (o *Order) recalculateTotal() {
	if len(o.items) == 0 {
		o.total = MustNewMoney(0, "USD")
		return
	}

	// Use the currency of the first item for all calculations
	currency := o.items[0].unitPrice.Currency()
	total := MustNewMoney(0, currency)

	for _, item := range o.items {
		itemTotal := item.Total()
		var err error
		total, err = total.Add(itemTotal)
		if err != nil {
			// This shouldn't happen if all items have the same currency
			panic(err)
		}
	}

	o.total = total
}
```

This Order aggregate enforces several business rules:

1. Items can only be added to, removed from, or updated in draft orders
2. Orders can only be placed if they contain at least one item
3. Order status transitions follow a specific flow
4. Order totals are automatically recalculated when items change

The Order is an aggregate root that encapsulates OrderItems, which are part of the aggregate but not accessible directly from outside the aggregate.

### **27.3.4 Repositories**

Repositories provide a way to retrieve and persist aggregates, abstracting the underlying data storage mechanism. In DDD, repositories typically operate on aggregate roots.

Let's implement repositories for our Order and Customer aggregates:

#### **Repository Interfaces**

First, let's define the repository interfaces:

```go
// domain/repositories.go
package domain

import "context"

// OrderRepository defines the interface for order persistence
type OrderRepository interface {
	// FindByID retrieves an order by ID
	FindByID(ctx context.Context, id string) (*Order, error)

	// FindByCustomerID retrieves orders for a customer
	FindByCustomerID(ctx context.Context, customerID string, limit, offset int) ([]*Order, error)

	// Save persists an order
	Save(ctx context.Context, order *Order) error

	// Delete removes an order
	Delete(ctx context.Context, id string) error
}

// CustomerRepository defines the interface for customer persistence
type CustomerRepository interface {
	// FindByID retrieves a customer by ID
	FindByID(ctx context.Context, id string) (*Customer, error)

	// FindByEmail retrieves a customer by email
	FindByEmail(ctx context.Context, email string) (*Customer, error)

	// Save persists a customer
	Save(ctx context.Context, customer *Customer) error

	// Delete removes a customer
	Delete(ctx context.Context, id string) error
}
```

These interfaces define the operations that can be performed on the respective aggregates without exposing implementation details.

#### **In-Memory Repositories**

Let's implement in-memory repositories for testing and development:

```go
// infrastructure/inmemory/order_repository.go
package inmemory

import (
	"context"
	"errors"
	"sync"

	"github.com/e-commerce/domain"
)

// OrderRepository implements an in-memory order repository
type OrderRepository struct {
	orders map[string]*domain.Order
	mutex  sync.RWMutex
}

// NewOrderRepository creates a new in-memory order repository
func NewOrderRepository() *OrderRepository {
	return &OrderRepository{
		orders: make(map[string]*domain.Order),
	}
}

// FindByID retrieves an order by ID
func (r *OrderRepository) FindByID(ctx context.Context, id string) (*domain.Order, error) {
	r.mutex.RLock()
	defer r.mutex.RUnlock()

	order, exists := r.orders[id]
	if !exists {
		return nil, errors.New("order not found")
	}

	return order, nil
}

// FindByCustomerID retrieves orders for a customer
func (r *OrderRepository) FindByCustomerID(ctx context.Context, customerID string, limit, offset int) ([]*domain.Order, error) {
	r.mutex.RLock()
	defer r.mutex.RUnlock()

	var customerOrders []*domain.Order

	// Find all orders for this customer
	for _, order := range r.orders {
		if order.CustomerID() == customerID {
			customerOrders = append(customerOrders, order)
		}
	}

	// Apply pagination
	if offset >= len(customerOrders) {
		return []*domain.Order{}, nil
	}

	end := offset + limit
	if end > len(customerOrders) || limit <= 0 {
		end = len(customerOrders)
	}

	return customerOrders[offset:end], nil
}

// Save persists an order
func (r *OrderRepository) Save(ctx context.Context, order *domain.Order) error {
	r.mutex.Lock()
	defer r.mutex.Unlock()

	r.orders[order.ID()] = order
	return nil
}

// Delete removes an order
func (r *OrderRepository) Delete(ctx context.Context, id string) error {
	query := `DELETE FROM orders WHERE id = $1`

	result, err := r.db.ExecContext(ctx, query, id)
	if err != nil {
		return fmt.Errorf("error deleting order: %w", err)
	}

	rowsAffected, err := result.RowsAffected()
	if err != nil {
		return fmt.Errorf("error getting rows affected: %w", err)
	}

	if rowsAffected == 0 {
		return errors.New("order not found")
	}

	return nil
}
```

```go
// infrastructure/inmemory/customer_repository.go
package inmemory

import (
	"context"
	"errors"
	"sync"

	"github.com/e-commerce/domain"
)

// CustomerRepository implements an in-memory customer repository
type CustomerRepository struct {
	customers map[string]*domain.Customer
	mutex     sync.RWMutex
}

// NewCustomerRepository creates a new in-memory customer repository
func NewCustomerRepository() *CustomerRepository {
	return &CustomerRepository{
		customers: make(map[string]*domain.Customer),
	}
}

// FindByID retrieves a customer by ID
func (r *CustomerRepository) FindByID(ctx context.Context, id string) (*domain.Customer, error) {
	r.mutex.RLock()
	defer r.mutex.RUnlock()

	customer, exists := r.customers[id]
	if !exists {
		return nil, errors.New("customer not found")
	}

	return customer, nil
}

// FindByEmail retrieves a customer by email
func (r *CustomerRepository) FindByEmail(ctx context.Context, email string) (*domain.Customer, error) {
	r.mutex.RLock()
	defer r.mutex.RUnlock()

	for _, customer := range r.customers {
		if customer.Email().Address() == email {
			return customer, nil
		}
	}

	return nil, errors.New("customer not found")
}

// Save persists a customer
func (r *CustomerRepository) Save(ctx context.Context, customer *domain.Customer) error {
	r.mutex.Lock()
	defer r.mutex.Unlock()

	r.customers[customer.ID()] = customer
	return nil
}

// Delete removes a customer
func (r *CustomerRepository) Delete(ctx context.Context, id string) error {
	r.mutex.Lock()
	defer r.mutex.Unlock()

	_, exists := r.customers[id]
	if !exists {
		return errors.New("customer not found")
	}

	delete(r.customers, id)
	return nil
}
```

#### **SQL Repositories**

For production use, we'd typically use a database. Here's an example of a PostgreSQL repository implementation:

```go
// infrastructure/postgres/order_repository.go
package postgres

import (
	"context"
	"database/sql"
	"encoding/json"
	"errors"
	"fmt"
	"time"

	"github.com/e-commerce/domain"
	"github.com/lib/pq"
)

// OrderRepository implements a PostgreSQL order repository
type OrderRepository struct {
	db *sql.DB
}

// NewOrderRepository creates a new PostgreSQL order repository
func NewOrderRepository(db *sql.DB) *OrderRepository {
	return &OrderRepository{
		db: db,
	}
}

// FindByID retrieves an order by ID
func (r *OrderRepository) FindByID(ctx context.Context, id string) (*domain.Order, error) {
	query := `
		SELECT
			id, customer_id, shipping_address, billing_address,
			status, placed_at, updated_at, total_amount, total_currency,
			items
		FROM orders
		WHERE id = $1
	`

	var (
		orderID        string
		customerID     string
		shippingAddrJSON, billingAddrJSON []byte
		statusStr      string
		placedAt       sql.NullTime
		updatedAt      time.Time
		totalAmount    int64
		totalCurrency  string
		itemsJSON      []byte
	)

	err := r.db.QueryRowContext(ctx, query, id).Scan(
		&orderID,
		&customerID,
		&shippingAddrJSON,
		&billingAddrJSON,
		&statusStr,
		&placedAt,
		&updatedAt,
		&totalAmount,
		&totalCurrency,
		&itemsJSON,
	)

	if err != nil {
		if err == sql.ErrNoRows {
			return nil, errors.New("order not found")
		}
		return nil, fmt.Errorf("error querying order: %w", err)
	}

	// Parse shipping address
	var shippingAddrData map[string]string
	if err := json.Unmarshal(shippingAddrJSON, &shippingAddrData); err != nil {
		return nil, fmt.Errorf("error parsing shipping address: %w", err)
	}

	shippingAddr, err := domain.NewAddress(
		shippingAddrData["street"],
		shippingAddrData["city"],
		shippingAddrData["state"],
		shippingAddrData["postal_code"],
		shippingAddrData["country"],
	)
	if err != nil {
		return nil, fmt.Errorf("error creating shipping address: %w", err)
	}

	// Parse billing address
	var billingAddrData map[string]string
	if err := json.Unmarshal(billingAddrJSON, &billingAddrData); err != nil {
		return nil, fmt.Errorf("error parsing billing address: %w", err)
	}

	billingAddr, err := domain.NewAddress(
		billingAddrData["street"],
		billingAddrData["city"],
		billingAddrData["state"],
		billingAddrData["postal_code"],
		billingAddrData["country"],
	)
	if err != nil {
		return nil, fmt.Errorf("error creating billing address: %w", err)
	}

	// Create order
	order, err := domain.ReconstructOrder(
		orderID,
		customerID,
		domain.OrderStatus(statusStr),
		shippingAddr,
		billingAddr,
		placedAt.Time,
		updatedAt,
		domain.MustNewMoney(totalAmount, totalCurrency),
	)
	if err != nil {
		return nil, fmt.Errorf("error reconstructing order: %w", err)
	}

	// Parse items
	var itemsData []map[string]interface{}
	if err := json.Unmarshal(itemsJSON, &itemsData); err != nil {
		return nil, fmt.Errorf("error parsing order items: %w", err)
	}

	// Add items to order
	for _, itemData := range itemsData {
		productID := itemData["product_id"].(string)
		productName := itemData["product_name"].(string)
		quantity := int(itemData["quantity"].(float64))
		unitPriceAmount := int64(itemData["unit_price_amount"].(float64))
		unitPriceCurrency := itemData["unit_price_currency"].(string)

		item, err := domain.NewOrderItem(
			productID,
			productName,
			quantity,
			domain.MustNewMoney(unitPriceAmount, unitPriceCurrency),
		)
		if err != nil {
			return nil, fmt.Errorf("error creating order item: %w", err)
		}

		if err := order.AddItemForReconstruction(item); err != nil {
			return nil, fmt.Errorf("error adding item to order: %w", err)
		}
	}

	return order, nil
}

// FindByCustomerID retrieves orders for a customer
func (r *OrderRepository) FindByCustomerID(ctx context.Context, customerID string, limit, offset int) ([]*domain.Order, error) {
	query := `
		SELECT
			id, customer_id, shipping_address, billing_address,
			status, placed_at, updated_at, total_amount, total_currency,
			items
		FROM orders
		WHERE customer_id = $1
		ORDER BY updated_at DESC
		LIMIT $2 OFFSET $3
	`

	rows, err := r.db.QueryContext(ctx, query, customerID, limit, offset)
	if err != nil {
		return nil, fmt.Errorf("error querying orders: %w", err)
	}
	defer rows.Close()

	var orders []*domain.Order

	for rows.Next() {
		var (
			orderID        string
			custID         string
			shippingAddrJSON, billingAddrJSON []byte
			statusStr      string
			placedAt       sql.NullTime
			updatedAt      time.Time
			totalAmount    int64
			totalCurrency  string
			itemsJSON      []byte
		)

		err := rows.Scan(
			&orderID,
			&custID,
			&shippingAddrJSON,
			&billingAddrJSON,
			&statusStr,
			&placedAt,
			&updatedAt,
			&totalAmount,
			&totalCurrency,
			&itemsJSON,
		)

		if err != nil {
			return nil, fmt.Errorf("error scanning order row: %w", err)
		}

		// Parse shipping address
		var shippingAddrData map[string]string
		if err := json.Unmarshal(shippingAddrJSON, &shippingAddrData); err != nil {
			return nil, fmt.Errorf("error parsing shipping address: %w", err)
		}

		shippingAddr, err := domain.NewAddress(
			shippingAddrData["street"],
			shippingAddrData["city"],
			shippingAddrData["state"],
			shippingAddrData["postal_code"],
			shippingAddrData["country"],
		)
		if err != nil {
			return nil, fmt.Errorf("error creating shipping address: %w", err)
		}

		// Parse billing address
		var billingAddrData map[string]string
		if err := json.Unmarshal(billingAddrJSON, &billingAddrData); err != nil {
			return nil, fmt.Errorf("error parsing billing address: %w", err)
		}

		billingAddr, err := domain.NewAddress(
			billingAddrData["street"],
			billingAddrData["city"],
			billingAddrData["state"],
			billingAddrData["postal_code"],
			billingAddrData["country"],
		)
		if err != nil {
			return nil, fmt.Errorf("error creating billing address: %w", err)
		}

		// Create order
		order, err := domain.ReconstructOrder(
			orderID,
			custID,
			domain.OrderStatus(statusStr),
			shippingAddr,
			billingAddr,
			placedAt.Time,
			updatedAt,
			domain.MustNewMoney(totalAmount, totalCurrency),
		)
		if err != nil {
			return nil, fmt.Errorf("error reconstructing order: %w", err)
		}

		// Parse items
		var itemsData []map[string]interface{}
		if err := json.Unmarshal(itemsJSON, &itemsData); err != nil {
			return nil, fmt.Errorf("error parsing order items: %w", err)
		}

		// Add items to order
		for _, itemData := range itemsData {
			productID := itemData["product_id"].(string)
			productName := itemData["product_name"].(string)
			quantity := int(itemData["quantity"].(float64))
			unitPriceAmount := int64(itemData["unit_price_amount"].(float64))
			unitPriceCurrency := itemData["unit_price_currency"].(string)

			item, err := domain.NewOrderItem(
				productID,
				productName,
				quantity,
				domain.MustNewMoney(unitPriceAmount, unitPriceCurrency),
			)
			if err != nil {
				return nil, fmt.Errorf("error creating order item: %w", err)
			}

			if err := order.AddItemForReconstruction(item); err != nil {
				return nil, fmt.Errorf("error adding item to order: %w", err)
			}
		}

		orders = append(orders, order)
	}

	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("error iterating order rows: %w", err)
	}

	return orders, nil
}

// Save persists an order
func (r *OrderRepository) Save(ctx context.Context, order *domain.Order) error {
	// Convert order items to JSON
	var itemsData []map[string]interface{}
	for _, item := range order.Items() {
		itemsData = append(itemsData, map[string]interface{}{
			"product_id":         item.ProductID(),
			"product_name":       item.ProductName(),
			"quantity":           item.Quantity(),
			"unit_price_amount":  item.UnitPrice().Amount(),
			"unit_price_currency": item.UnitPrice().Currency(),
		})
	}

	itemsJSON, err := json.Marshal(itemsData)
	if err != nil {
		return fmt.Errorf("error marshaling order items: %w", err)
	}

	// Convert addresses to JSON
	shippingAddr := order.ShippingAddress()
	shippingAddrJSON, err := json.Marshal(map[string]string{
		"street":      shippingAddr.Street(),
		"city":        shippingAddr.City(),
		"state":       shippingAddr.State(),
		"postal_code": shippingAddr.PostalCode(),
		"country":     shippingAddr.Country(),
	})
	if err != nil {
		return fmt.Errorf("error marshaling shipping address: %w", err)
	}

	billingAddr := order.BillingAddress()
	billingAddrJSON, err := json.Marshal(map[string]string{
		"street":      billingAddr.Street(),
		"city":        billingAddr.City(),
		"state":       billingAddr.State(),
		"postal_code": billingAddr.PostalCode(),
		"country":     billingAddr.Country(),
	})
	if err != nil {
		return fmt.Errorf("error marshaling billing address: %w", err)
	}

	// Prepare placed_at with SQL NULL if not set
	var placedAt sql.NullTime
	if !order.PlacedAt().IsZero() {
		placedAt = sql.NullTime{
			Time:  order.PlacedAt(),
			Valid: true,
		}
	}

	// Upsert the order
	query := `
		INSERT INTO orders (
			id, customer_id, shipping_address, billing_address,
			status, placed_at, updated_at, total_amount, total_currency,
			items
		) VALUES (
			$1, $2, $3, $4, $5, $6, $7, $8, $9, $10
		)
		ON CONFLICT (id) DO UPDATE SET
			shipping_address = $3,
			billing_address = $4,
			status = $5,
			placed_at = $6,
			updated_at = $7,
			total_amount = $8,
			total_currency = $9,
			items = $10
	`

	_, err = r.db.ExecContext(ctx, query,
		order.ID(),
		order.CustomerID(),
		shippingAddrJSON,
		billingAddrJSON,
		string(order.Status()),
		placedAt,
		order.UpdatedAt(),
		order.Total().Amount(),
		order.Total().Currency(),
		itemsJSON,
	)

	if err != nil {
		return fmt.Errorf("error saving order: %w", err)
	}

	return nil
}

// Delete removes an order
func (r *OrderRepository) Delete(ctx context.Context, id string) error {
	query := `DELETE FROM orders WHERE id = $1`

	result, err := r.db.ExecContext(ctx, query, id)
	if err != nil {
		return fmt.Errorf("error deleting order: %w", err)
	}

	rowsAffected, err := result.RowsAffected()
	if err != nil {
		return fmt.Errorf("error getting rows affected: %w", err)
	}

	if rowsAffected == 0 {
		return errors.New("order not found")
	}

	return nil
}
```

Note that for the SQL repository to work, we'd need to add a method to the Order aggregate for reconstruction from the database:

```go
// domain/order.go
// Add this to the existing order.go file

// ReconstructOrder creates an order from persistent storage
// This method is used by repositories to reconstruct the aggregate
func ReconstructOrder(
	id string,
	customerID string,
	status OrderStatus,
	shippingAddress Address,
	billingAddress Address,
	placedAt time.Time,
	updatedAt time.Time,
	total Money,
) (*Order, error) {
	if id == "" {
		return nil, errors.New("order ID cannot be empty")
	}
	if customerID == "" {
		return nil, errors.New("customer ID cannot be empty")
	}

	return &Order{
		id:              id,
		customerID:      customerID,
		items:           []OrderItem{},
		shippingAddress: shippingAddress,
		billingAddress:  billingAddress,
		status:          status,
		placedAt:        placedAt,
		updatedAt:       updatedAt,
		total:           total,
	}, nil
}

// AddItemForReconstruction adds an item during reconstruction
// This method is used by repositories to reconstruct the aggregate
func (o *Order) AddItemForReconstruction(item OrderItem) error {
	o.items = append(o.items, item)
	return nil
}
```

### **27.3.5 Domain Services**

Domain Services contain domain logic that doesn't naturally fit within an entity or value object. They represent business operations or transformations that involve multiple domain objects.

Let's implement a pricing service that calculates the final price of an order, including discounts and taxes:

```go
// domain/pricing_service.go
package domain

import (
	"errors"
)

// DiscountStrategy defines how discounts are applied
type DiscountStrategy interface {
	// Calculate calculates the discount amount
	Calculate(subtotal Money, items []OrderItem, customerID string) (Money, error)
}

// TaxStrategy defines how taxes are calculated
type TaxStrategy interface {
	// Calculate calculates the tax amount
	Calculate(subtotal Money, items []OrderItem, shippingAddress Address) (Money, error)
}

// PricingService calculates prices, discounts, and taxes
type PricingService struct {
	discountStrategy DiscountStrategy
	taxStrategy      TaxStrategy
}

// NewPricingService creates a new pricing service
func NewPricingService(discountStrategy DiscountStrategy, taxStrategy TaxStrategy) *PricingService {
	return &PricingService{
		discountStrategy: discountStrategy,
		taxStrategy:      taxStrategy,
	}
}

// CalculateOrderTotal calculates the total price of an order
func (s *PricingService) CalculateOrderTotal(order *Order) (Money, error) {
	if order == nil {
		return Money{}, errors.New("order cannot be nil")
	}

	// Calculate subtotal from items
	items := order.Items()
	if len(items) == 0 {
		return MustNewMoney(0, "USD"), nil
	}

	// Use the currency of the first item
	currency := items[0].UnitPrice().Currency()
	subtotal := MustNewMoney(0, currency)

	for _, item := range items {
		itemTotal := item.Total()
		var err error
		subtotal, err = subtotal.Add(itemTotal)
		if err != nil {
			return Money{}, err
		}
	}

	// Calculate discount if a strategy is provided
	var discount Money
	if s.discountStrategy != nil {
		var err error
		discount, err = s.discountStrategy.Calculate(subtotal, items, order.CustomerID())
		if err != nil {
			return Money{}, err
		}
	}

	// Apply discount to subtotal
	subtotalAfterDiscount, err := subtotal.Subtract(discount)
	if err != nil {
		return Money{}, err
	}

	// Calculate tax if a strategy is provided
	var tax Money
	if s.taxStrategy != nil {
		var err error
		tax, err = s.taxStrategy.Calculate(subtotalAfterDiscount, items, order.ShippingAddress())
		if err != nil {
			return Money{}, err
		}
	}

	// Add tax to get final total
	total, err := subtotalAfterDiscount.Add(tax)
	if err != nil {
		return Money{}, err
	}

	return total, nil
}
```

This service contains domain logic for calculating prices that involves multiple entities (Order, OrderItems) and value objects (Money, Address).

Let's implement a simple percentage discount strategy:

```go
// domain/percentage_discount.go
package domain

// PercentageDiscountStrategy applies a percentage discount
type PercentageDiscountStrategy struct {
	percentage float64
}

// NewPercentageDiscountStrategy creates a percentage discount strategy
func NewPercentageDiscountStrategy(percentage float64) *PercentageDiscountStrategy {
	return &PercentageDiscountStrategy{
		percentage: percentage,
	}
}

// Calculate calculates the discount amount
func (s *PercentageDiscountStrategy) Calculate(subtotal Money, items []OrderItem, customerID string) (Money, error) {
	// Calculate discount amount
	discountFactor := s.percentage / 100.0
	return subtotal.Multiply(discountFactor), nil
}
```

And a simple tax strategy based on the shipping address:

```go
// domain/location_tax.go
package domain

// LocationTaxStrategy applies tax based on location
type LocationTaxStrategy struct {
	taxRates map[string]float64 // country -> tax rate percentage
}

// NewLocationTaxStrategy creates a location-based tax strategy
func NewLocationTaxStrategy(taxRates map[string]float64) *LocationTaxStrategy {
	return &LocationTaxStrategy{
		taxRates: taxRates,
	}
}

// Calculate calculates the tax amount
func (s *LocationTaxStrategy) Calculate(subtotal Money, items []OrderItem, shippingAddress Address) (Money, error) {
	// Get tax rate for the country
	rate, exists := s.taxRates[shippingAddress.Country()]
	if !exists {
		// Default to no tax if country not found
		return MustNewMoney(0, subtotal.Currency()), nil
	}

	// Calculate tax amount
	taxFactor := rate / 100.0
	return subtotal.Multiply(taxFactor), nil
}
```

### **27.3.6 Application Services**

Application Services orchestrate the workflow of an application, coordinating between domain objects, repositories, and other services. They form the bridge between the domain and the outside world.

Let's implement an application service for managing orders:

```go
// application/order_service.go
package application

import (
	"context"
	"errors"
	"time"

	"github.com/e-commerce/domain"
)

// OrderService provides order-related operations
type OrderService struct {
	orderRepository    domain.OrderRepository
	customerRepository domain.CustomerRepository
	productRepository  domain.ProductRepository
	pricingService     *domain.PricingService
	eventPublisher     EventPublisher
}

// EventPublisher defines the interface for publishing events
type EventPublisher interface {
	PublishOrderPlaced(ctx context.Context, orderID, customerID string, total domain.Money, items []domain.OrderItem) error
	PublishOrderCancelled(ctx context.Context, orderID, customerID string, reason string) error
}

// NewOrderService creates a new order service
func NewOrderService(
	orderRepo domain.OrderRepository,
	customerRepo domain.CustomerRepository,
	productRepo domain.ProductRepository,
	pricingService *domain.PricingService,
	eventPublisher EventPublisher,
) *OrderService {
	return &OrderService{
		orderRepository:    orderRepo,
		customerRepository: customerRepo,
		productRepository:  productRepo,
		pricingService:     pricingService,
		eventPublisher:     eventPublisher,
	}
}

// CreateOrderRequest contains data for creating an order
type CreateOrderRequest struct {
	CustomerID      string
	Items           []OrderItemRequest
	ShippingAddress AddressRequest
	BillingAddress  AddressRequest
}

// OrderItemRequest contains data for an order item
type OrderItemRequest struct {
	ProductID string
	Quantity  int
}

// AddressRequest contains address data
type AddressRequest struct {
	Street     string
	City       string
	State      string
	PostalCode string
	Country    string
}

// CreateOrder creates a new order
func (s *OrderService) CreateOrder(ctx context.Context, req CreateOrderRequest) (string, error) {
	// Verify customer exists
	customer, err := s.customerRepository.FindByID(ctx, req.CustomerID)
	if err != nil {
		return "", err
	}

	// Create shipping address
	shippingAddr, err := domain.NewAddress(
		req.ShippingAddress.Street,
		req.ShippingAddress.City,
		req.ShippingAddress.State,
		req.ShippingAddress.PostalCode,
		req.ShippingAddress.Country,
	)
	if err != nil {
		return "", err
	}

	// Create billing address
	billingAddr, err := domain.NewAddress(
		req.BillingAddress.Street,
		req.BillingAddress.City,
		req.BillingAddress.State,
		req.BillingAddress.PostalCode,
		req.BillingAddress.Country,
	)
	if err != nil {
		return "", err
	}

	// Create order
	order, err := domain.NewOrder(customer.ID(), shippingAddr, billingAddr)
	if err != nil {
		return "", err
	}

	// Add items to order
	for _, itemReq := range req.Items {
		// Get product
		product, err := s.productRepository.FindByID(ctx, itemReq.ProductID)
		if err != nil {
			return "", err
		}

		// Create order item
		item, err := domain.NewOrderItem(
			product.ID(),
			product.Name(),
			itemReq.Quantity,
			product.Price(),
		)
		if err != nil {
			return "", err
		}

		// Add item to order
		if err := order.AddItem(item); err != nil {
			return "", err
		}
	}

	// Save order
	if err := s.orderRepository.Save(ctx, order); err != nil {
		return "", err
	}

	return order.ID(), nil
}

// PlaceOrderRequest contains data for placing an order
type PlaceOrderRequest struct {
	OrderID string
}

// PlaceOrder places an existing order
func (s *OrderService) PlaceOrder(ctx context.Context, req PlaceOrderRequest) error {
	// Get order
	order, err := s.orderRepository.FindByID(ctx, req.OrderID)
	if err != nil {
		return err
	}

	// Place order
	if err := order.Place(); err != nil {
		return err
	}

	// Save order
	if err := s.orderRepository.Save(ctx, order); err != nil {
		return err
	}

	// Publish event
	if s.eventPublisher != nil {
		err := s.eventPublisher.PublishOrderPlaced(
			ctx,
			order.ID(),
			order.CustomerID(),
			order.Total(),
			order.Items(),
		)
		if err != nil {
			// Log error but don't fail the operation
			// In a real system, you might want to retry or compensate
			// (this is simplified for the example)
		}
	}

	return nil
}

// CancelOrderRequest contains data for cancelling an order
type CancelOrderRequest struct {
	OrderID string
	Reason  string
}

// CancelOrder cancels an order
func (s *OrderService) CancelOrder(ctx context.Context, req CancelOrderRequest) error {
	// Get order
	order, err := s.orderRepository.FindByID(ctx, req.OrderID)
	if err != nil {
		return err
	}

	// Cancel order
	if err := order.Cancel(); err != nil {
		return err
	}

	// Save order
	if err := s.orderRepository.Save(ctx, order); err != nil {
		return err
	}

	// Publish event
	if s.eventPublisher != nil {
		err := s.eventPublisher.PublishOrderCancelled(
			ctx,
			order.ID(),
			order.CustomerID(),
			req.Reason,
		)
		if err != nil {
			// Log error but don't fail the operation
		}
	}

	return nil
}

// GetOrderResponse contains order details
type GetOrderResponse struct {
	ID              string
	CustomerID      string
	Status          string
	ShippingAddress domain.Address
	BillingAddress  domain.Address
	Items           []domain.OrderItem
	Total           domain.Money
	PlacedAt        time.Time
	UpdatedAt       time.Time
}

// GetOrder retrieves an order by ID
func (s *OrderService) GetOrder(ctx context.Context, orderID string) (*GetOrderResponse, error) {
	// Get order
	order, err := s.orderRepository.FindByID(ctx, orderID)
	if err != nil {
		return nil, err
	}

	// Convert to response
	response := &GetOrderResponse{
		ID:              order.ID(),
		CustomerID:      order.CustomerID(),
		Status:          string(order.Status()),
		ShippingAddress: order.ShippingAddress(),
		BillingAddress:  order.BillingAddress(),
		Items:           order.Items(),
		Total:           order.Total(),
		PlacedAt:        order.PlacedAt(),
		UpdatedAt:       order.UpdatedAt(),
	}

	return response, nil
}
```

This application service orchestrates operations involving multiple domain objects and repositories. It also handles the publishing of domain events when orders are placed or cancelled.

### **27.3.7 Domain Events**

Domain Events represent something significant that has happened in the domain. They are used to decouple components, enable asynchronous processing, and provide a record of what happened.

Let's implement a simple domain event system:

```go
// domain/events.go
package domain

import "time"

// DomainEvent represents something that happened in the domain
type DomainEvent interface {
	// EventType returns the type of the event
	EventType() string

	// AggregateID returns the ID of the aggregate that emitted the event
	AggregateID() string

	// OccurredAt returns when the event occurred
	OccurredAt() time.Time
}

// BaseDomainEvent provides common event functionality
type BaseDomainEvent struct {
	eventType   string
	aggregateID string
	occurredAt  time.Time
}

// EventType returns the type of the event
func (e BaseDomainEvent) EventType() string {
	return e.eventType
}

// AggregateID returns the ID of the aggregate that emitted the event
func (e BaseDomainEvent) AggregateID() string {
	return e.aggregateID
}

// OccurredAt returns when the event occurred
func (e BaseDomainEvent) OccurredAt() time.Time {
	return e.occurredAt
}
```

Now, let's implement specific domain events for our Order aggregate:

```go
// domain/order_events.go
package domain

import (
	"time"
)

// OrderPlacedEvent represents an order being placed
type OrderPlacedEvent struct {
	BaseDomainEvent
	CustomerID      string
	Items           []OrderItem
	ShippingAddress Address
	BillingAddress  Address
	Total           Money
}

// NewOrderPlacedEvent creates a new OrderPlacedEvent
func NewOrderPlacedEvent(order *Order) *OrderPlacedEvent {
	return &OrderPlacedEvent{
		BaseDomainEvent: BaseDomainEvent{
			eventType:   "order.placed",
			aggregateID: order.ID(),
			occurredAt:  time.Now().UTC(),
		},
		CustomerID:      order.CustomerID(),
		Items:           order.Items(),
		ShippingAddress: order.ShippingAddress(),
		BillingAddress:  order.BillingAddress(),
		Total:           order.Total(),
	}
}

// OrderCancelledEvent represents an order being cancelled
type OrderCancelledEvent struct {
	BaseDomainEvent
	CustomerID string
	Reason     string
}

// NewOrderCancelledEvent creates a new OrderCancelledEvent
func NewOrderCancelledEvent(order *Order, reason string) *OrderCancelledEvent {
	return &OrderCancelledEvent{
		BaseDomainEvent: BaseDomainEvent{
			eventType:   "order.cancelled",
			aggregateID: order.ID(),
			occurredAt:  time.Now().UTC(),
		},
		CustomerID: order.CustomerID(),
		Reason:     reason,
	}
}

// OrderShippedEvent represents an order being shipped
type OrderShippedEvent struct {
	BaseDomainEvent
	CustomerID       string
	ShippingAddress  Address
	TrackingNumber   string
	EstimatedArrival time.Time
}

// NewOrderShippedEvent creates a new OrderShippedEvent
func NewOrderShippedEvent(order *Order, trackingNumber string, estimatedArrival time.Time) *OrderShippedEvent {
	return &OrderShippedEvent{
		BaseDomainEvent: BaseDomainEvent{
			eventType:   "order.shipped",
			aggregateID: order.ID(),
			occurredAt:  time.Now().UTC(),
		},
		CustomerID:       order.CustomerID(),
		ShippingAddress:  order.ShippingAddress(),
		TrackingNumber:   trackingNumber,
		EstimatedArrival: estimatedArrival,
	}
}
```

## **27.4 Applying DDD Principles in Go Projects**

Now that we've explored the tactical patterns of DDD, let's discuss how to apply these principles effectively in Go projects.

### **27.4.1 Project Structure for DDD**

A typical DDD-oriented Go project might be structured like this:

```
myapp/
├── cmd/                      # Application entry points
│   └── server/               # HTTP server
│       └── main.go
├── domain/                   # Domain layer
│   ├── customer.go           # Customer aggregate
│   ├── order.go              # Order aggregate
│   ├── product.go            # Product aggregate
│   ├── money.go              # Money value object
│   ├── address.go            # Address value object
│   ├── repositories.go       # Repository interfaces
│   ├── services.go           # Domain service interfaces
│   └── events.go             # Domain events
├── application/              # Application layer
│   ├── customer_service.go   # Customer application service
│   ├── order_service.go      # Order application service
│   └── product_service.go    # Product application service
├── infrastructure/           # Infrastructure layer
│   ├── persistence/          # Persistence implementations
│   │   ├── postgres/         # PostgreSQL repositories
│   │   └── inmemory/         # In-memory repositories
│   ├── messaging/            # Messaging implementations
│   │   ├── kafka/            # Kafka event publisher
│   │   └── inmemory/         # In-memory event bus
│   └── api/                  # API handlers
│       ├── rest/             # REST API
│       └── grpc/             # gRPC API
└── interfaces/               # User interfaces
    ├── rest/                 # REST handlers
    └── grpc/                 # gRPC handlers
```

This structure follows the principles of layered architecture, where:

1. **Domain Layer**: Contains the core domain model, including entities, value objects, and domain services
2. **Application Layer**: Coordinates between the domain layer and the outside world
3. **Infrastructure Layer**: Provides implementations of the interfaces defined in the domain layer
4. **Interfaces Layer**: Exposes the application to the outside world

### **27.4.2 Testing DDD Applications**

DDD lends itself well to testing at different levels:

**Unit Tests for Domain Objects**:

- Test behavior of entities, value objects, and domain services in isolation
- Focus on business rules and invariants

**Integration Tests for Repositories**:

- Test repository implementations against real or in-memory databases
- Verify that persistence logic works correctly

**Application Service Tests**:

- Test the orchestration of domain objects and repositories
- Use mocks or stubs for dependencies

**End-to-End Tests**:

- Test the entire application from the user interface to the database
- Verify that all components work together correctly

### **27.4.3 DDD and Microservices**

DDD's concept of bounded contexts aligns naturally with microservices. Each bounded context can become a microservice, with its own domain model, database, and API.

When designing a microservice architecture based on DDD:

1. **Identify Bounded Contexts**: Each bounded context can be a candidate for a microservice
2. **Define Context Maps**: Map the relationships between contexts to design service interactions
3. **Use Domain Events**: Communicate between services using domain events
4. **Implement Anticorruption Layers**: Protect your domain model from external services

### **27.4.4 Common Pitfalls and How to Avoid Them**

**Anemic Domain Model**:

- Symptoms: Entities with just getters and setters, business logic in services
- Solution: Move business logic into entities and value objects

**Overengineering**:

- Symptoms: Complex designs for simple problems, too many abstractions
- Solution: Start simple, add complexity only when needed

**Ignoring Bounded Contexts**:

- Symptoms: Single model for the entire application, inconsistent terminology
- Solution: Explicitly define and enforce context boundaries

**Repository Bloat**:

- Symptoms: Repositories with too many query methods, exposing implementation details
- Solution: Keep repositories focused on aggregate persistence, use specialized queries for read operations

**Misusing Value Objects**:

- Symptoms: Using primitive types for domain concepts, mutable value objects
- Solution: Create value objects for domain concepts, ensure immutability

## **27.5 Conclusion**

Domain-Driven Design provides a powerful approach to building software that aligns closely with business needs. By focusing on the core domain and using a ubiquitous language, DDD helps create systems that are both technically sound and business-relevant.

Go's simplicity, strong typing, and emphasis on clarity make it an excellent language for implementing DDD. The combination of Go's pragmatic approach with DDD's focus on the domain creates software that is both maintainable and valuable to the business.

In this chapter, we've explored both strategic and tactical aspects of DDD, showing how to implement them in Go. From modeling the domain with entities and value objects to structuring the application with bounded contexts and aggregates, we've covered the essential patterns and practices of DDD.

As you apply these principles to your own projects, remember that DDD is not about following patterns rigidly, but about understanding the domain deeply and expressing that understanding in code. Start with a focus on the core domain, establish a ubiquitous language, and let the technical design emerge from the domain model.

By combining DDD with Go's strengths, you can build software that not only meets technical requirements but also delivers real business value.
