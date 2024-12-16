# **Chapter 23: Essential Design Patterns in Go**

---

## **23.1. Introduction to Design Patterns**

Design patterns are proven solutions to common problems in software design. They provide a reusable template to solve specific problems in a structured way. In this chapter, we will explore 10 essential design patterns every Go developer should know. These patterns will help you:

- Write more organized and maintainable code.
- Solve common software design problems efficiently.
- Understand how to apply patterns in Go's idiomatic style.

---

## **23.2. Why Design Patterns Matter**

1. **Reusability**: Avoid reinventing the wheel by using tried-and-tested solutions.
2. **Maintainability**: Simplify the structure of your code, making it easier to understand and extend.
3. **Scalability**: Design systems that are easier to scale and adapt as requirements change.

---

## **23.3. 10 Essential Design Patterns**

We’ll cover the following patterns with simple explanations, real-world scenarios, and examples:

### **1. Singleton Pattern**

**Why?** Ensure a single instance of a resource (e.g., database connection) is created and shared.

**Example: Database Connection**

```go
package main

import (
	"fmt"
	"sync"
)

type Database struct{}

var instance *Database
var once sync.Once

func GetDatabaseInstance() *Database {
	once.Do(func() {
		fmt.Println("Creating a single database instance")
		instance = &Database{}
	})
	return instance
}

func main() {
	db1 := GetDatabaseInstance()
	db2 := GetDatabaseInstance()
	fmt.Println(db1 == db2) // Output: true
}
```

---

### **2. Factory Pattern**

**Why?** Encapsulate the creation of objects to avoid direct instantiation.

**Example: Shape Factory**

```go
package main

import "fmt"

type Shape interface {
	Draw()
}

type Circle struct{}
func (c Circle) Draw() { fmt.Println("Drawing a Circle") }

type Square struct{}
func (s Square) Draw() { fmt.Println("Drawing a Square") }

func GetShape(shapeType string) Shape {
	switch shapeType {
	case "circle":
		return Circle{}
	case "square":
		return Square{}
	default:
		return nil
	}
}

func main() {
	shape := GetShape("circle")
	shape.Draw() // Output: Drawing a Circle
}
```

---

### **3. Builder Pattern**

**Why?** Simplify complex object creation by breaking it into steps.

**Example: Building a Car**

```go
package main

import "fmt"

type Car struct {
	Engine string
	Wheels int
	Color  string
}

type CarBuilder struct {
	car Car
}

func (b *CarBuilder) SetEngine(engine string) *CarBuilder {
	b.car.Engine = engine
	return b
}

func (b *CarBuilder) SetWheels(wheels int) *CarBuilder {
	b.car.Wheels = wheels
	return b
}

func (b *CarBuilder) SetColor(color string) *CarBuilder {
	b.car.Color = color
	return b
}

func (b *CarBuilder) Build() Car {
	return b.car
}

func main() {
	car := (&CarBuilder{}).SetEngine("V8").SetWheels(4).SetColor("Red").Build()
	fmt.Printf("Car: %+v
", car) // Output: Car: {Engine:V8 Wheels:4 Color:Red}
}
```

---

### **4. Adapter Pattern**

**Why?** Allow incompatible interfaces to work together.

**Example: Power Adapter**

```go
package main

import "fmt"

type OldCharger struct{}
func (c OldCharger) Charge() { fmt.Println("Charging with Old Charger") }

type NewCharger struct{}
func (c NewCharger) FastCharge() { fmt.Println("Fast Charging with New Charger") }

type ChargerAdapter struct {
	NewCharger NewCharger
}

func (a ChargerAdapter) Charge() {
	a.NewCharger.FastCharge()
}

func main() {
	adapter := ChargerAdapter{NewCharger: NewCharger{}}
	adapter.Charge() // Output: Fast Charging with New Charger
}
```

---

### **5. Observer Pattern**

**Why?** Notify multiple objects about changes in another object.

**Example: Stock Price Updates**

```go
package main

import "fmt"

type Observer interface {
	Update(price float64)
}

type Stock struct {
	observers []Observer
	price     float64
}

func (s *Stock) AddObserver(o Observer) {
	s.observers = append(s.observers, o)
}

func (s *Stock) SetPrice(price float64) {
	s.price = price
	for _, o := range s.observers {
		o.Update(price)
	}
}

type Investor struct {
	Name string
}

func (i Investor) Update(price float64) {
	fmt.Printf("%s notified: Stock price updated to %.2f
", i.Name, price)
}

func main() {
	stock := &Stock{}
	investor1 := Investor{Name: "Alice"}
	investor2 := Investor{Name: "Bob"}

	stock.AddObserver(investor1)
	stock.AddObserver(investor2)

	stock.SetPrice(100.0) // Both Alice and Bob are notified.
}
```

---

### **6. Strategy Pattern**

**Why?** Allow interchangeable algorithms at runtime.

**Example: Payment Methods**

```go
package main

import "fmt"

type PaymentStrategy interface {
	Pay(amount float64)
}

type CreditCard struct{}
func (c CreditCard) Pay(amount float64) { fmt.Printf("Paid %.2f using Credit Card
", amount) }

type PayPal struct{}
func (p PayPal) Pay(amount float64) { fmt.Printf("Paid %.2f using PayPal
", amount) }

type PaymentContext struct {
	Strategy PaymentStrategy
}

func (c *PaymentContext) SetStrategy(strategy PaymentStrategy) {
	c.Strategy = strategy
}

func (c PaymentContext) Pay(amount float64) {
	c.Strategy.Pay(amount)
}

func main() {
	context := PaymentContext{}

	context.SetStrategy(CreditCard{})
	context.Pay(50.0) // Output: Paid 50.00 using Credit Card

	context.SetStrategy(PayPal{})
	context.Pay(75.0) // Output: Paid 75.00 using PayPal
}
```

---

### **7. Decorator Pattern**

**Why?** Add functionality to objects dynamically.

**Example: Adding Features to Coffee**

```go
package main

import "fmt"

type Coffee interface {
	Cost() float64
	Description() string
}

type BasicCoffee struct{}
func (b BasicCoffee) Cost() float64          { return 5.0 }
func (b BasicCoffee) Description() string    { return "Basic Coffee" }

type MilkDecorator struct {
	Coffee Coffee
}
func (m MilkDecorator) Cost() float64        { return m.Coffee.Cost() + 1.5 }
func (m MilkDecorator) Description() string  { return m.Coffee.Description() + " + Milk" }

type SugarDecorator struct {
	Coffee Coffee
}
func (s SugarDecorator) Cost() float64       { return s.Coffee.Cost() + 0.5 }
func (s SugarDecorator) Description() string { return s.Coffee.Description() + " + Sugar" }

func main() {
	coffee := BasicCoffee{}
	coffee = MilkDecorator{Coffee: coffee}
	coffee = SugarDecorator{Coffee: coffee}
	fmt.Printf("%s: $%.2f
", coffee.Description(), coffee.Cost()) // Output: Basic Coffee + Milk + Sugar: $7.00
}
```

---

### **8. Command Pattern**

**Why?** Encapsulate requests as objects.

**Example: Remote Control**

```go
package main

import "fmt"

type Command interface {
	Execute()
}

type Light struct{}
func (l Light) On() { fmt.Println("Light is On") }
func (l Light) Off() { fmt.Println("Light is Off") }

type LightOnCommand struct {
	Light Light
}
func (c LightOnCommand) Execute() { c.Light.On() }

type LightOffCommand struct {
	Light Light
}
func (c LightOffCommand) Execute() { c.Light.Off() }

func main() {
	light := Light{}
	onCommand := LightOnCommand{Light: light}
	offCommand := LightOffCommand{Light: light}

	onCommand.Execute()  // Output: Light is On
	offCommand.Execute() // Output: Light is Off
}
```

---

### **9. Proxy Pattern**

**Why?** Control access to an object.

**Example: Database Query Proxy**

```go
package main

import "fmt"

type Database interface {
	Query(query string)
}

type RealDatabase struct{}
func (d RealDatabase) Query(query string) {
	fmt.Println("Executing query:", query)
}

type DatabaseProxy struct {
	RealDB RealDatabase
}
func (p DatabaseProxy) Query(query string) {
	fmt.Println("Logging query:", query)
	p.RealDB.Query(query)
}

func main() {
	proxy := DatabaseProxy{RealDB: RealDatabase{}}
	proxy.Query("SELECT * FROM users")
}
```

---

### **10. State Pattern**

**Why?** Manage object states with separate behaviors for each state.

**Example: Traffic Light**

```go
package main

import "fmt"

type TrafficLightState interface {
	Next()
}

type RedState struct{}
func (r RedState) Next() { fmt.Println("Changing to Green") }

type GreenState struct{}
func (g GreenState) Next() { fmt.Println("Changing to Yellow") }

type YellowState struct{}
func (y YellowState) Next() { fmt.Println("Changing to Red") }

func main() {
	currentState := RedState{}
	currentState.Next() // Output: Changing to Green
}
```

---

## **23.4. Conclusion**

These 10 design patterns provide a strong foundation for solving common problems in Go development. Practice implementing them in real projects to fully understand their power and flexibility.

---

# **Exercises for Chapter 23: Essential Design Patterns in Go**

---

## **Exercise 1: Singleton Pattern**

**Scenario**: You are building a logging system that should ensure a single logger instance is used throughout your application.

**Code Solution**:

```go
package main

import (
	"fmt"
	"sync"
)

type Logger struct{}

var instance *Logger
var once sync.Once

func GetLoggerInstance() *Logger {
	once.Do(func() {
		instance = &Logger{}
		fmt.Println("Logger instance created")
	})
	return instance
}

func (l *Logger) Log(message string) {
	fmt.Println("Log:", message)
}

func main() {
	logger1 := GetLoggerInstance()
	logger2 := GetLoggerInstance()

	logger1.Log("Singleton Pattern")
	fmt.Println(logger1 == logger2) // Output: true
}
```

---

## **Exercise 2: Factory Pattern**

**Scenario**: Create a factory function that generates animals. Each animal should have a `Speak` method.

**Code Solution**:

```go
package main

import "fmt"

type Animal interface {
	Speak() string
}

type Dog struct{}
func (d Dog) Speak() string { return "Woof" }

type Cat struct{}
func (c Cat) Speak() string { return "Meow" }

func AnimalFactory(animalType string) Animal {
	switch animalType {
	case "dog":
		return Dog{}
	case "cat":
		return Cat{}
	default:
		return nil
	}
}

func main() {
	animal := AnimalFactory("dog")
	fmt.Println(animal.Speak()) // Output: Woof
}
```

---

## **Exercise 3: Builder Pattern**

**Scenario**: Build a Pizza object step-by-step, where each step adds an ingredient.

**Code Solution**:

```go
package main

import "fmt"

type Pizza struct {
	Crust      string
	Toppings   []string
	CheeseType string
}

type PizzaBuilder struct {
	pizza Pizza
}

func (b *PizzaBuilder) SetCrust(crust string) *PizzaBuilder {
	b.pizza.Crust = crust
	return b
}

func (b *PizzaBuilder) AddTopping(topping string) *PizzaBuilder {
	b.pizza.Toppings = append(b.pizza.Toppings, topping)
	return b
}

func (b *PizzaBuilder) SetCheese(cheese string) *PizzaBuilder {
	b.pizza.CheeseType = cheese
	return b
}

func (b *PizzaBuilder) Build() Pizza {
	return b.pizza
}

func main() {
	pizza := (&PizzaBuilder{}).SetCrust("Thin").AddTopping("Pepperoni").SetCheese("Mozzarella").Build()
	fmt.Printf("Pizza: %+v
", pizza) // Output: Pizza: {Crust:Thin Toppings:[Pepperoni] CheeseType:Mozzarella}
}
```

---

## **Exercise 4: Adapter Pattern**

**Scenario**: Adapt an old charger to work with a new fast-charger interface.

**Code Solution**:

```go
package main

import "fmt"

type OldCharger struct{}
func (o OldCharger) Charge() { fmt.Println("Charging with old charger") }

type NewCharger interface {
	FastCharge()
}

type OldToNewAdapter struct {
	OldCharger
}

func (a OldToNewAdapter) FastCharge() {
	fmt.Println("Adapting to fast charge...")
	a.Charge()
}

func main() {
	adapter := OldToNewAdapter{}
	adapter.FastCharge()
}
```

---

## **Exercise 5: Observer Pattern**

**Scenario**: Build a weather station that notifies observers when the temperature changes.

**Code Solution**:

```go
package main

import "fmt"

type Observer interface {
	Update(temperature float64)
}

type WeatherStation struct {
	observers []Observer
	temperature float64
}

func (w *WeatherStation) AddObserver(o Observer) {
	w.observers = append(w.observers, o)
}

func (w *WeatherStation) SetTemperature(temp float64) {
	w.temperature = temp
	for _, observer := range w.observers {
		observer.Update(temp)
	}
}

type NewsChannel struct {
	Name string
}

func (n NewsChannel) Update(temp float64) {
	fmt.Printf("%s: Temperature updated to %.2f
", n.Name, temp)
}

func main() {
	station := &WeatherStation{}
	channel1 := NewsChannel{Name: "Channel 1"}
	channel2 := NewsChannel{Name: "Channel 2"}

	station.AddObserver(channel1)
	station.AddObserver(channel2)

	station.SetTemperature(25.0)
}
```

---

## **Exercise 6: Strategy Pattern**

**Scenario**: Implement a payment system where users can choose between `CreditCard` and `PayPal`.

**Code Solution**:

```go
package main

import "fmt"

type PaymentStrategy interface {
	Pay(amount float64)
}

type CreditCard struct{}
func (c CreditCard) Pay(amount float64) { fmt.Printf("Paid %.2f using Credit Card
", amount) }

type PayPal struct{}
func (p PayPal) Pay(amount float64) { fmt.Printf("Paid %.2f using PayPal
", amount) }

type PaymentContext struct {
	strategy PaymentStrategy
}

func (c *PaymentContext) SetStrategy(strategy PaymentStrategy) {
	c.strategy = strategy
}

func (c PaymentContext) Pay(amount float64) {
	c.strategy.Pay(amount)
}

func main() {
	context := &PaymentContext{}

	context.SetStrategy(CreditCard{})
	context.Pay(50.0)

	context.SetStrategy(PayPal{})
	context.Pay(75.0)
}
```

---

## **Exercise 7: Decorator Pattern**

**Scenario**: Create a coffee shop menu where additional items like milk and sugar can be added dynamically.

**Code Solution**:

```go
package main

import "fmt"

type Coffee interface {
	Cost() float64
	Description() string
}

type BasicCoffee struct{}
func (b BasicCoffee) Cost() float64 { return 5.0 }
func (b BasicCoffee) Description() string { return "Basic Coffee" }

type MilkDecorator struct {
	Coffee Coffee
}
func (m MilkDecorator) Cost() float64 { return m.Coffee.Cost() + 1.5 }
func (m MilkDecorator) Description() string { return m.Coffee.Description() + " + Milk" }

type SugarDecorator struct {
	Coffee Coffee
}
func (s SugarDecorator) Cost() float64 { return s.Coffee.Cost() + 0.5 }
func (s SugarDecorator) Description() string { return s.Coffee.Description() + " + Sugar" }

func main() {
	coffee := BasicCoffee{}
	coffee = MilkDecorator{Coffee: coffee}
	coffee = SugarDecorator{Coffee: coffee}
	fmt.Printf("%s: $%.2f
", coffee.Description(), coffee.Cost())
}
```

---

**Note:** For brevity, this file contains 7 exercises with solutions. Full solutions for all 15 exercises (including Command, Proxy, State, and real-world scenarios) will follow the same pattern and are built upon these principles.

---

### **How to Proceed**

- Test each code example.
- Modify and extend these solutions to fit real-world applications.
