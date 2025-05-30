# **Chapter 7: Packages and Modules - Organizing Go Code**

Go's approach to code organization is both powerful and straightforward. The package system provides a clean way to structure your code, while the module system (introduced in Go 1.11) manages dependencies with elegance and precision. Understanding these systems is essential for writing maintainable Go code and effectively collaborating with other developers.

In this chapter, you'll learn how to organize your code into packages, import and use code from other packages, create your own modules, and manage dependencies effectively.

## **7.1 Understanding Go Packages**

### **7.1.1 What is a Package?**

A package is Go's unit of code organization and reuse. Every Go file belongs to a package, declared at the top of the file:

```go
package mypackage
```

Packages serve several important purposes:

1. **Code organization**: Group related code together
2. **Encapsulation**: Control visibility of functions, types, and variables
3. **Reusability**: Use code across multiple projects
4. **Compilation**: Compiled as a unit

### **7.1.2 Package Naming Conventions**

In Go, package names should be:

- **Short and concise**: Prefer single-word names
- **Lowercase**: Never use camelCase or snake_case
- **Descriptive**: Reflect the package's purpose
- **Not pluralized**: Use `store` not `stores`
- **Not generic**: Avoid names like `util`, `common`, or `misc`

Good package names include:

```
http     // for HTTP client and server implementations
json     // for JSON encoding and decoding
io       // for I/O operations
fmt      // for formatting and printing
strings  // for string manipulation
math     // for mathematical operations
```

### **7.1.3 Package Visibility Rules**

Go controls visibility (public vs. private) through identifier naming:

- **Exported (public)**: Identifiers starting with an uppercase letter are accessible from other packages
- **Unexported (private)**: Identifiers starting with a lowercase letter are only accessible within the same package

```go
package geometry

// Circle is exported (public) - accessible from other packages
type Circle struct {
    Radius float64 // Exported field
    color  string  // Unexported field
}

// CalculateArea is exported (public) - accessible from other packages
func (c Circle) CalculateArea() float64 {
    return 3.14 * c.Radius * c.Radius
}

// setColor is unexported (private) - only accessible within this package
func (c *Circle) setColor(color string) {
    c.color = color
}
```

### **7.1.4 Package Documentation**

Go emphasizes documentation as part of the development process. Package documentation follows a simple format:

- Package-level documentation appears before the package declaration
- Exported identifier documentation appears directly above the identifier

```go
// Package geometry provides utilities for geometric calculations.
// It supports various shapes like circles, rectangles, and triangles.
package geometry

// Circle represents a circle shape with a radius.
type Circle struct {
    Radius float64
}

// CalculateArea returns the area of the circle.
func (c Circle) CalculateArea() float64 {
    return 3.14 * c.Radius * c.Radius
}
```

Access documentation with `go doc`:

```bash
go doc geometry           # View package documentation
go doc geometry.Circle    # View Circle type documentation
```

## **7.2 Working with Multiple Files and Packages**

### **7.2.1 Multi-file Packages**

A package can span multiple files. When you do this:

1. Each file must declare the same package name
2. All files in the same directory must belong to the same package
3. Functions, types, and variables can reference each other directly without import

Example:

**shapes.go**:

```go
package geometry

// Shape is an interface that all shapes implement
type Shape interface {
    CalculateArea() float64
    CalculatePerimeter() float64
}

// defaultColor returns the default color for shapes
func defaultColor() string {
    return "black"
}
```

**circle.go**:

```go
package geometry

// Circle implements the Shape interface
type Circle struct {
    Radius float64
    color  string
}

// NewCircle creates a new Circle with the default color
func NewCircle(radius float64) Circle {
    return Circle{
        Radius: radius,
        color:  defaultColor(), // Using function from shapes.go
    }
}

// CalculateArea returns the area of the circle
func (c Circle) CalculateArea() float64 {
    return 3.14 * c.Radius * c.Radius
}

// CalculatePerimeter returns the perimeter of the circle
func (c Circle) CalculatePerimeter() float64 {
    return 2 * 3.14 * c.Radius
}
```

### **7.2.2 Package Directory Structure**

A typical Go project with multiple packages might have a structure like this:

```
myproject/
├── main.go                   # main package
├── geometry/
│   ├── shapes.go             # geometry package
│   ├── circle.go
│   └── rectangle.go
└── drawing/
    ├── canvas.go             # drawing package
    └── color.go
```

Each subdirectory represents a separate package, and all files within that directory must belong to the same package.

### **7.2.3 Importing Packages**

To use code from another package, you must import it:

```go
package main

import (
    "fmt"
    "myproject/geometry"
)

func main() {
    circle := geometry.NewCircle(5.0)
    area := circle.CalculateArea()
    fmt.Printf("Circle area: %.2f\n", area)
}
```

**Import Aliases**:

You can create aliases for package names to avoid conflicts or for convenience:

```go
import (
    "fmt"
    geo "myproject/geometry"  // Alias for the geometry package
)

func main() {
    circle := geo.NewCircle(5.0)
    // ...
}
```

**Dot Imports**:

The dot import makes exported identifiers directly accessible (without the package prefix), but it's generally discouraged as it reduces clarity:

```go
import (
    "fmt"
    . "myproject/geometry"  // Dot import, use with caution
)

func main() {
    circle := NewCircle(5.0)  // No package prefix needed
    // ...
}
```

**Blank Imports**:

Sometimes you need to import a package for its side effects only (init functions):

```go
import (
    "fmt"
    _ "github.com/lib/pq"  // Registers PostgreSQL driver, but doesn't use the package directly
)
```

## **7.3 Standard Library Packages**

The Go standard library is comprehensive and well-designed, providing core functionality without needing external dependencies.

### **7.3.1 Core Packages**

Here are some of the most commonly used standard library packages:

| Package         | Description                    | Example Usage                     |
| --------------- | ------------------------------ | --------------------------------- |
| `fmt`           | Formatted I/O                  | `fmt.Println("Hello, world!")`    |
| `os`            | Operating system functionality | `os.Open("file.txt")`             |
| `io`            | Basic I/O interfaces           | `io.Copy(dst, src)`               |
| `strconv`       | String conversions             | `strconv.Atoi("42")`              |
| `strings`       | String manipulation            | `strings.Split("a,b,c", ",")`     |
| `time`          | Time functionality             | `time.Now()`                      |
| `net/http`      | HTTP client and server         | `http.Get("https://example.com")` |
| `encoding/json` | JSON encoding/decoding         | `json.Marshal(data)`              |
| `math`          | Mathematical functions         | `math.Sqrt(4)`                    |
| `crypto`        | Cryptographic algorithms       | `crypto/sha256`                   |

### **7.3.2 Finding Packages in Standard Library**

The complete standard library is documented at [pkg.go.dev](https://pkg.go.dev/std).

You can also view documentation locally:

```bash
go doc fmt          # View documentation for the fmt package
go doc time.Now     # View documentation for the Now function in the time package
```

### **7.3.3 Example: Using Standard Library Packages**

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "time"
)

type User struct {
    Name      string    `json:"name"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

func main() {
    // Create a user
    user := User{
        Name:      "Alice",
        Email:     "alice@example.com",
        CreatedAt: time.Now(),
    }

    // Convert to JSON
    jsonData, err := json.MarshalIndent(user, "", "  ")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }

    fmt.Println(string(jsonData))

    // Make HTTP request
    resp, err := http.Get("https://api.github.com")
    if err != nil {
        fmt.Println("Error:", err)
        return
    }
    defer resp.Body.Close()

    fmt.Println("GitHub API Status:", resp.Status)
}
```

## **7.4 Creating and Using Your Own Packages**

### **7.4.1 Creating a Package**

Let's create a simple calculator package:

Directory structure:

```
calculator/
├── calculator.go
└── operations.go
```

**calculator.go**:

```go
// Package calculator provides basic mathematical operations.
package calculator

// Configuration variables
var (
    Precision   = 2
    MaxOperands = 10
)

// init is called when the package is imported
func init() {
    // Package initialization code goes here
    fmt.Println("Calculator package initialized")
}
```

**operations.go**:

```go
package calculator

import "math"

// Add returns the sum of two numbers
func Add(a, b float64) float64 {
    return a + b
}

// Subtract returns the difference between two numbers
func Subtract(a, b float64) float64 {
    return a - b
}

// Multiply returns the product of two numbers
func Multiply(a, b float64) float64 {
    return a * b
}

// Divide returns the quotient of two numbers
// Returns NaN if divisor is zero
func Divide(a, b float64) float64 {
    if b == 0 {
        return math.NaN()
    }
    return a / b
}

// Internal helper function (not exported)
func round(num float64) float64 {
    factor := math.Pow(10, float64(Precision))
    return math.Round(num*factor) / factor
}
```

### **7.4.2 Using Your Package**

Now let's use our calculator package in a main program:

**main.go**:

```go
package main

import (
    "fmt"
    "myapp/calculator"
)

func main() {
    // Use exported functions from the calculator package
    sum := calculator.Add(5.7, 3.2)
    difference := calculator.Subtract(10.5, 2.5)

    // Use exported variables
    calculator.Precision = 4

    fmt.Printf("Sum: %.2f\n", sum)
    fmt.Printf("Difference: %.2f\n", difference)
    fmt.Printf("Max Operands: %d\n", calculator.MaxOperands)

    // This would cause a compilation error - cannot use unexported function
    // result := calculator.round(5.7567)
}
```

### **7.4.3 Package Initialization**

When a package is imported, these steps happen in order:

1. Package-level variables are initialized
2. `init()` functions are executed in the order they appear in the source file
3. If multiple `init()` functions exist (across multiple files), they're executed in lexical file name order

```go
package mypackage

import "fmt"

var PackageVar = initVar()

func initVar() int {
    fmt.Println("Variable initialization")
    return 42
}

func init() {
    fmt.Println("First init function")
}

func init() {
    fmt.Println("Second init function")
}
```

When this package is imported, the output would be:

```
Variable initialization
First init function
Second init function
```

## **7.5 Introduction to Go Modules**

### **7.5.1 What are Go Modules?**

Go modules, introduced in Go 1.11, are the official dependency management system for Go. They allow you to:

1. **Track dependencies**: Explicitly define and track dependencies
2. **Version control**: Specify exact versions of dependencies
3. **Reproducible builds**: Ensure consistent builds across different environments
4. **Dependency pruning**: Only include what you need

### **7.5.2 Creating a New Module**

To create a new module:

```bash
mkdir myproject
cd myproject
go mod init github.com/username/myproject
```

This creates a `go.mod` file:

```
module github.com/username/myproject

go 1.16
```

The module path (`github.com/username/myproject`) becomes the import path prefix for all packages in the module.

### **7.5.3 Adding Dependencies**

When you import a package that's not in the standard library, Go automatically adds it to your module:

```go
package main

import (
    "fmt"
    "github.com/fatih/color"
)

func main() {
    color.Red("This is red text")
    color.Green("This is green text")
    fmt.Println("This is normal text")
}
```

Run the code with:

```bash
go run main.go
```

Go will:

1. Download the required dependency
2. Add it to your `go.mod` file
3. Create a `go.sum` file with checksums to verify the dependency

After running, your `go.mod` might look like:

```
module github.com/username/myproject

go 1.16

require github.com/fatih/color v1.13.0
```

### **7.5.4 Managing Dependencies**

**Viewing Dependencies**:

```bash
go list -m all  # List all dependencies
```

**Updating Dependencies**:

```bash
go get github.com/fatih/color        # Update to latest version
go get github.com/fatih/color@v1.12.0 # Update to specific version
go get -u                             # Update all dependencies
```

**Removing Unused Dependencies**:

```bash
go mod tidy  # Remove unused dependencies and add missing ones
```

### **7.5.5 Understanding Semantic Versioning**

Go modules rely on Semantic Versioning (SemVer) for dependency management:

```
v1.2.3
 │ │ │
 │ │ └── Patch: Bug fixes that don't break compatibility
 │ └──── Minor: New features that don't break compatibility
 └────── Major: Changes that break backward compatibility
```

In `go.mod`, dependencies can be specified with different version constraints:

```
require (
    github.com/example/package1 v1.2.3         // Exact version
    github.com/example/package2 v1.2           // Any patch version of v1.2
    github.com/example/package3 v1             // Any minor and patch version of v1
    github.com/example/package4 v1.2.3-alpha.1 // Specific pre-release version
)
```

## **7.6 Working with Internal Packages**

### **7.6.1 The Internal Package Pattern**

Go has a special directory name `internal` that restricts package visibility. Packages inside an `internal` directory can only be imported by packages that are in the same directory or its subdirectories.

```
mymodule/
├── cmd/
│   └── myapp/
│       └── main.go
├── internal/      # Only importable within mymodule
│   ├── config/
│   └── database/
└── pkg/          # Publicly available packages
    ├── api/
    └── logger/
```

In this example:

- Packages in `internal` can only be imported by code in the `mymodule` module
- Packages in `pkg` can be imported by any module

### **7.6.2 Example of Internal Package Usage**

**mymodule/internal/config/config.go**:

```go
package config

type Configuration struct {
    APIKey      string
    Environment string
    Debug       bool
}

func Load() *Configuration {
    // Load configuration from file or environment
    return &Configuration{
        APIKey:      "secret-key",
        Environment: "development",
        Debug:       true,
    }
}
```

**mymodule/cmd/myapp/main.go**:

```go
package main

import (
    "fmt"
    "mymodule/internal/config"
    "mymodule/pkg/api"
)

func main() {
    // We can import the internal package because we're in the same module
    cfg := config.Load()

    // Initialize API with configuration
    client := api.NewClient(cfg.APIKey, cfg.Debug)

    fmt.Printf("API client initialized in %s mode\n", cfg.Environment)
}
```

If another module tries to import `mymodule/internal/config`, the Go compiler will produce an error.

## **7.7 Organizing Large Go Applications**

### **7.7.1 Common Project Layouts**

For larger applications, you'll want a consistent structure. One popular layout is:

```
myapp/
├── cmd/                    # Command-line applications
│   ├── server/             # The API server
│   │   └── main.go
│   └── cli/                # The CLI tool
│       └── main.go
├── internal/               # Private packages
│   ├── auth/               # Authentication logic
│   ├── middleware/         # HTTP middleware
│   └── database/           # Database access
├── pkg/                    # Public packages
│   ├── models/             # Data models
│   └── utils/              # Utility functions
├── api/                    # API documentation, OpenAPI/Swagger specs
├── web/                    # Web assets
├── configs/                # Configuration files
├── deployments/            # Deployment configurations (Docker, K8s)
├── docs/                   # Documentation
├── examples/               # Example code
├── scripts/                # Build and deployment scripts
├── test/                   # Additional test applications and test data
├── go.mod
└── go.sum
```

### **7.7.2 Domain-Driven Design Approach**

For complex business applications, you might prefer organizing by domain:

```
myapp/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── user/               # User domain
│   │   ├── repository.go   # Data access
│   │   ├── service.go      # Business logic
│   │   ├── handler.go      # HTTP handlers
│   │   └── model.go        # Domain models
│   ├── product/            # Product domain
│   │   ├── repository.go
│   │   ├── service.go
│   │   ├── handler.go
│   │   └── model.go
│   └── common/             # Shared code
├── pkg/
└── ...
```

### **7.7.3 Guidelines for Package Organization**

1. **Group by function**: Keep related functionality together
2. **Avoid circular dependencies**: Structure packages hierarchically
3. **Minimize public APIs**: Only export what's necessary
4. **Design for testability**: Make packages easy to test in isolation
5. **Balance package size**: Neither too large nor too small
6. **Use interface-based design**: Define interfaces at package boundaries

## **7.8 Vendor Directory and Vendoring**

### **7.8.1 What is Vendoring?**

Vendoring is the practice of copying all dependencies into your project's repository, typically in a `vendor` directory. Before Go modules, vendoring was the primary way to manage dependencies.

With modules, vendoring is optional but still useful for:

- Ensuring build reproducibility
- Working in environments without internet access
- Auditing all dependency code

### **7.8.2 Using the Vendor Directory**

To create a vendor directory:

```bash
go mod vendor
```

This creates a `vendor` directory with all dependencies.

To build using the vendor directory:

```bash
go build -mod=vendor
```

## **7.9 Best Practices for Packages and Modules**

### **7.9.1 Package Design Principles**

1. **Cohesion**: A package should have a single, well-defined purpose
2. **Minimal interfaces**: Export only what's necessary
3. **No circular dependencies**: Design for unidirectional dependencies
4. **Stable dependencies**: Depend on packages that change less frequently than yours
5. **Package by layer or by feature**: Choose a consistent organizational approach

### **7.9.2 Module Versioning Best Practices**

1. **Semantic versioning**: Follow v1.2.3 format strictly
2. **API compatibility**: Don't break backward compatibility within the same major version
3. **Use major version directories** for v2 and above:
   ```
   github.com/user/module/v2/package
   ```
4. **Keep go.mod clean**: Use `go mod tidy` regularly
5. **Tag releases** in your version control system

### **7.9.3 Documentation Guidelines**

1. **Document all exported identifiers**: Functions, types, variables, constants
2. **Start with the name**: `// Package log provides...`, `// NewWriter returns...`
3. **Include examples** for complex functionality
4. **Link related functions** in documentation
5. **Generate documentation** locally with `godoc` server

## **7.10 Creating and Publishing Your Own Modules**

### **7.10.1 Preparing Your Module for Publication**

1. Choose a good module path (typically your repo URL)
2. Add a `README.md` with:
   - Overview and purpose
   - Installation instructions
   - Usage examples
   - API documentation link
3. Add proper license file
4. Add good tests with reasonable coverage
5. Include examples in a separate directory

### **7.10.2 Versioning Your Module**

Use Git tags to mark versions:

```bash
git tag v1.0.0
git push origin v1.0.0
```

### **7.10.3 Publishing to pkg.go.dev**

Your module is automatically published to pkg.go.dev when:

1. Someone runs `go get` for your module, or
2. You request it explicitly via pkg.go.dev

No additional registration is required.

## **7.11 Practice Exercises**

### **Exercise 1: Create a Basic Package**

Create a `calculator` package with basic math operations and use it in a main program.

### **Exercise 2: Multi-file Package**

Create a `geometry` package with multiple shape implementations across different files.

### **Exercise 3: Module Dependencies**

Create a module that uses at least two third-party dependencies.

### **Exercise 4: Organizing a Small Project**

Organize a simple web server project using a proper project layout.

## **7.12 Summary**

In this chapter, we've explored Go's approach to code organization:

- **Packages** provide a clean way to structure code and control visibility
- **Imports** allow you to use code from other packages
- **Standard library** offers a rich set of packages for common tasks
- **Go modules** give you powerful dependency management
- **Project organization** helps maintain large codebases

Understanding packages and modules is essential for writing maintainable Go code and collaborating effectively with other developers. By following Go's conventions and best practices, you can create well-structured, easy-to-understand code that stands the test of time.

**Next Up**: In Chapter 8, we'll dive into arrays, slices, and strings, exploring how Go handles these fundamental data structures.
