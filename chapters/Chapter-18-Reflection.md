# **Chapter 18: Reflection in Go**

Reflection is a powerful feature in Go that allows programs to examine and modify their own structure and behavior at runtime. While Go is primarily a statically typed language, reflection provides a way to work with types dynamically. This chapter explores Go's reflection capabilities, their applications, and best practices for using this advanced feature effectively.

## **18.1 Introduction to Reflection**

Reflection enables a program to inspect and manipulate objects at runtime without knowing their types at compile time. In Go, reflection is implemented through the `reflect` package in the standard library.

### **18.1.1 What is Reflection?**

Reflection provides the ability to:

- Examine the type of a variable at runtime
- Access and modify fields of a struct dynamically
- Call methods on objects without knowing their exact type
- Create new values of a particular type
- Inspect and modify values indirectly

In essence, reflection gives us a way to work with code that manipulates other code rather than just performing computation directly.

### **18.1.2 When to Use Reflection**

Reflection is powerful but comes with trade-offs:

- **Performance impact**: Reflection operations are significantly slower than direct code
- **Type safety**: Many reflection errors only appear at runtime
- **Code complexity**: Reflection code is often more complex and harder to understand

Because of these drawbacks, reflection should be used judiciously. Common use cases include:

- Generic data handling (JSON/XML serialization)
- Implementing flexible APIs that work with arbitrary types
- Building testing and mocking frameworks
- Creating object-relational mappers (ORMs)
- Dynamic configuration and dependency injection

The Go proverb "Clear is better than clever" is particularly relevant when considering reflection. Use reflection only when the benefits clearly outweigh the costs.

## **18.2 Fundamentals of Reflection**

The `reflect` package provides two main types that form the foundation of reflection in Go:

- `reflect.Type`: Represents the type of a Go value
- `reflect.Value`: Represents the value itself

These types, along with their methods, enable the inspection and manipulation of Go values at runtime.

### **18.2.1 The Basic Reflection Functions**

The `reflect` package provides three fundamental functions:

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    // Create some variables of different types
    var i int = 42
    var s string = "hello"
    var f float64 = 3.14159

    // Use reflect.TypeOf to get the type
    fmt.Println("Type of i:", reflect.TypeOf(i))   // int
    fmt.Println("Type of s:", reflect.TypeOf(s))   // string
    fmt.Println("Type of f:", reflect.TypeOf(f))   // float64

    // Use reflect.ValueOf to get a Value representing the value
    vi := reflect.ValueOf(i)
    vs := reflect.ValueOf(s)

    // Get the underlying value with Interface()
    fmt.Println("Value of vi:", vi.Interface()) // 42
    fmt.Println("Value of vs:", vs.Interface()) // hello

    // Create a new value with reflect.New
    // This creates a pointer to a new zero value of the specified type
    newIntPtr := reflect.New(reflect.TypeOf(i))
    fmt.Println("Type of newIntPtr:", newIntPtr.Type()) // *int
    fmt.Println("Value of newIntPtr:", newIntPtr.Elem().Interface()) // 0
}
```

These functions provide the entry points for reflection:

1. `reflect.TypeOf(x)`: Returns a `reflect.Type` representing the type of `x`
2. `reflect.ValueOf(x)`: Returns a `reflect.Value` representing the value of `x`
3. `reflect.New(type)`: Creates a new value of the specified type

### **18.2.2 Kind vs. Type**

In reflection, there's an important distinction between a value's "kind" and its "type":

- **Type**: The specific type of a value, such as `string`, `int`, or a user-defined type like `Person`
- **Kind**: The underlying base type category, such as `Int`, `String`, `Struct`, or `Slice`

A custom type and its underlying base type have the same kind but different types:

```go
package main

import (
    "fmt"
    "reflect"
)

type MyInt int
type Person struct {
    Name string
    Age  int
}

func main() {
    var i MyInt = 42
    var p Person = Person{"Alice", 30}

    // Type includes the package path and name
    fmt.Println("Type of i:", reflect.TypeOf(i)) // main.MyInt
    fmt.Println("Type of p:", reflect.TypeOf(p)) // main.Person

    // Kind is the underlying base type category
    fmt.Println("Kind of i:", reflect.ValueOf(i).Kind()) // int
    fmt.Println("Kind of p:", reflect.ValueOf(p).Kind()) // struct

    // Regular int and MyInt have different types but same kind
    var regularInt int = 42
    fmt.Println("Types equal?", reflect.TypeOf(i) == reflect.TypeOf(regularInt)) // false
    fmt.Println("Kinds equal?", reflect.ValueOf(i).Kind() == reflect.ValueOf(regularInt).Kind()) // true
}
```

Understanding this distinction is crucial when working with reflection. The `Kind` determines what operations are valid on a given value.

## **18.3 Working with reflect.Type**

The `reflect.Type` interface provides methods for examining type information at runtime. This is useful for generic programming, type checking, and documentation generation.

### **18.3.1 Basic Type Information**

`reflect.Type` provides methods to access basic type information:

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    ID        int
    Name      string
    Email     string
    IsActive  bool
    CreatedAt string
}

func main() {
    u := User{1, "Alice", "alice@example.com", true, "2023-01-01"}
    t := reflect.TypeOf(u)

    // Basic type information
    fmt.Println("Type name:", t.Name())           // User
    fmt.Println("Package path:", t.PkgPath())     // main
    fmt.Println("Kind:", t.Kind())                // struct
    fmt.Println("Size in bytes:", t.Size())       // varies by architecture
    fmt.Println("Is variable sized?", t.VariableLen()) // false

    // For a slice, it would be different
    s := []int{1, 2, 3}
    sliceType := reflect.TypeOf(s)
    fmt.Println("Slice type name:", sliceType.Name())        // "" (anonymous)
    fmt.Println("Slice kind:", sliceType.Kind())             // slice
    fmt.Println("Slice element type:", sliceType.Elem())     // int
    fmt.Println("Is variable sized?", sliceType.VariableLen()) // true
}
```

### **18.3.2 Examining Struct Fields**

For struct types, `reflect.Type` provides methods to examine fields:

```go
package main

import (
    "fmt"
    "reflect"
)

type Address struct {
    Street string
    City   string
    ZIP    string
}

type Person struct {
    Name    string `json:"name" validate:"required"`
    Age     int    `json:"age" validate:"min=0,max=130"`
    Address Address
}

func main() {
    t := reflect.TypeOf(Person{})

    // Number of fields
    fmt.Println("Number of fields:", t.NumField()) // 3

    // Iterate through fields
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("Field #%d: Name=%s, Type=%v, Tag=%v\n",
            i, field.Name, field.Type, field.Tag)
    }

    // Get field by name
    nameField, found := t.FieldByName("Name")
    if found {
        fmt.Println("\nFound Name field:")
        fmt.Println("JSON tag:", nameField.Tag.Get("json"))          // name
        fmt.Println("Validate tag:", nameField.Tag.Get("validate"))  // required
    }

    // Accessing nested fields
    addressField, _ := t.FieldByName("Address")
    addressType := addressField.Type
    for i := 0; i < addressType.NumField(); i++ {
        field := addressType.Field(i)
        fmt.Printf("Address.%s: Type=%v\n", field.Name, field.Type)
    }
}
```

Output:

```
Number of fields: 3
Field #0: Name=Name, Type=string, Tag=json:"name" validate:"required"
Field #1: Name=Age, Type=int, Tag=json:"age" validate:"min=0,max=130"
Field #2: Name=Address, Type=main.Address, Tag=

Found Name field:
JSON tag: name
Validate tag: required

Address.Street: Type=string
Address.City: Type=string
Address.ZIP: Type=string
```

### **18.3.3 Examining Methods**

`reflect.Type` also provides access to a type's methods:

```go
package main

import (
    "fmt"
    "reflect"
)

type Greeter struct {
    Greeting string
}

func (g Greeter) SayHello(name string) string {
    return g.Greeting + ", " + name
}

func (g *Greeter) SetGreeting(greeting string) {
    g.Greeting = greeting
}

func main() {
    // Value receiver type
    valueType := reflect.TypeOf(Greeter{})
    fmt.Printf("Methods on %s:\n", valueType)
    for i := 0; i < valueType.NumMethod(); i++ {
        method := valueType.Method(i)
        fmt.Printf("  %s: %v\n", method.Name, method.Type)
    }

    // Pointer receiver type
    pointerType := reflect.TypeOf(&Greeter{})
    fmt.Printf("\nMethods on %s:\n", pointerType)
    for i := 0; i < pointerType.NumMethod(); i++ {
        method := pointerType.Method(i)
        fmt.Printf("  %s: %v\n", method.Name, method.Type)
    }

    // Get method by name
    sayHello, found := valueType.MethodByName("SayHello")
    if found {
        fmt.Printf("\nSayHello method: %v\n", sayHello.Type)
        // Method type includes receiver as first parameter
        fmt.Printf("Number of inputs: %d\n", sayHello.Type.NumIn())
        fmt.Printf("First input (receiver): %v\n", sayHello.Type.In(0))
        fmt.Printf("Second input: %v\n", sayHello.Type.In(1))
        fmt.Printf("Number of outputs: %d\n", sayHello.Type.NumOut())
        fmt.Printf("Output type: %v\n", sayHello.Type.Out(0))
    }
}
```

Output:

```
Methods on main.Greeter:
  SayHello: func(main.Greeter, string) string

Methods on *main.Greeter:
  SayHello: func(*main.Greeter, string) string
  SetGreeting: func(*main.Greeter, string)

SayHello method: func(main.Greeter, string) string
Number of inputs: 2
First input (receiver): main.Greeter
Second input: string
Number of outputs: 1
Output type: string
```

Notice that the pointer type includes both methods, while the value type only includes the value receiver method.

### **18.3.4 Type Comparisons and Conversions**

`reflect.Type` provides methods for comparing types and checking for assignability and convertibility:

```go
package main

import (
    "fmt"
    "reflect"
)

type MyInt int
type YourInt int

func main() {
    var i int = 42
    var mi MyInt = 42
    var yi YourInt = 42

    iType := reflect.TypeOf(i)
    miType := reflect.TypeOf(mi)
    yiType := reflect.TypeOf(yi)

    // Check if types are identical
    fmt.Println("int == MyInt?", iType == miType)                  // false
    fmt.Println("MyInt == YourInt?", miType == yiType)             // false

    // Check assignability (can a value of one type be assigned to a variable of another)
    fmt.Println("int assignable to MyInt?", iType.AssignableTo(miType))               // false
    fmt.Println("MyInt assignable to int?", miType.AssignableTo(iType))               // false
    fmt.Println("*MyInt assignable to *YourInt?", reflect.TypeOf(&mi).AssignableTo(reflect.TypeOf(&yi))) // false

    // Check convertibility (can a value be converted to another type)
    fmt.Println("int convertible to MyInt?", iType.ConvertibleTo(miType))            // true
    fmt.Println("MyInt convertible to YourInt?", miType.ConvertibleTo(yiType))       // true

    // Implement conversion
    miValue := reflect.ValueOf(mi)
    iValue := miValue.Convert(iType)
    fmt.Printf("Converted %v (%T) to %v (%T)\n",
        miValue.Interface(), miValue.Interface(),
        iValue.Interface(), iValue.Interface()) // Converted 42 (main.MyInt) to 42 (int)
}
```

### **18.3.5 Working with Array, Slice, and Map Types**

`reflect.Type` provides specific methods for array, slice, and map types:

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    // Array
    arr := [3]int{1, 2, 3}
    arrType := reflect.TypeOf(arr)
    fmt.Printf("Array: Kind=%v, Len=%d, Elem=%v\n",
        arrType.Kind(), arrType.Len(), arrType.Elem())

    // Slice
    slice := []string{"a", "b", "c"}
    sliceType := reflect.TypeOf(slice)
    fmt.Printf("Slice: Kind=%v, Elem=%v\n",
        sliceType.Kind(), sliceType.Elem())

    // Map
    m := map[string]int{"a": 1, "b": 2}
    mapType := reflect.TypeOf(m)
    fmt.Printf("Map: Kind=%v, Key=%v, Elem=%v\n",
        mapType.Kind(), mapType.Key(), mapType.Elem())

    // Channel
    ch := make(chan int)
    chType := reflect.TypeOf(ch)
    fmt.Printf("Channel: Kind=%v, Dir=%v, Elem=%v\n",
        chType.Kind(), chType.ChanDir(), chType.Elem())

    // Function
    fn := func(a int, b string) float64 { return 0 }
    fnType := reflect.TypeOf(fn)
    fmt.Printf("Function: Kind=%v, NumIn=%d, NumOut=%d\n",
        fnType.Kind(), fnType.NumIn(), fnType.NumOut())
    for i := 0; i < fnType.NumIn(); i++ {
        fmt.Printf("  In(%d): %v\n", i, fnType.In(i))
    }
    for i := 0; i < fnType.NumOut(); i++ {
        fmt.Printf("  Out(%d): %v\n", i, fnType.Out(i))
    }
}
```

Output:

```
Array: Kind=array, Len=3, Elem=int
Slice: Kind=slice, Elem=string
Map: Kind=map, Key=string, Elem=int
Channel: Kind=chan, Dir=both, Elem=int
Function: Kind=func, NumIn=2, NumOut=1
  In(0): int
  In(1): string
  Out(0): float64
```

### **18.3.6 Creating New Types at Runtime**

The `reflect` package provides functions to create new types at runtime, which can be useful for dynamic code generation:

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    // Create a slice type
    sliceType := reflect.SliceOf(reflect.TypeOf(0)) // []int
    fmt.Println("Created slice type:", sliceType)

    // Create a map type
    mapType := reflect.MapOf(reflect.TypeOf(""), reflect.TypeOf(0)) // map[string]int
    fmt.Println("Created map type:", mapType)

    // Create a channel type
    chanType := reflect.ChanOf(reflect.BothDir, reflect.TypeOf(0)) // chan int
    fmt.Println("Created channel type:", chanType)

    // Create an array type
    arrayType := reflect.ArrayOf(5, reflect.TypeOf("")) // [5]string
    fmt.Println("Created array type:", arrayType)

    // Create a pointer type
    ptrType := reflect.PtrTo(reflect.TypeOf(0)) // *int
    fmt.Println("Created pointer type:", ptrType)

    // Create a struct type
    fields := []reflect.StructField{
        {
            Name: "Name",
            Type: reflect.TypeOf(""),
            Tag:  reflect.StructTag(`json:"name"`),
        },
        {
            Name: "Age",
            Type: reflect.TypeOf(0),
            Tag:  reflect.StructTag(`json:"age"`),
        },
    }
    structType := reflect.StructOf(fields)
    fmt.Println("Created struct type:", structType)

    // Instantiate the struct type
    structValue := reflect.New(structType).Elem()
    structValue.Field(0).SetString("Alice")
    structValue.Field(1).SetInt(30)
    fmt.Printf("Created struct instance: %+v\n", structValue.Interface())
}
```

Output:

```
Created slice type: []int
Created map type: map[string]int
Created channel type: chan int
Created array type: [5]string
Created pointer type: *int
Created struct type: struct { Name string "json:\"name\""; Age int "json:\"age\"" }
Created struct instance: {Name:Alice Age:30}
```

These functions allow us to create and manipulate types dynamically, which is particularly useful for code generation and generic programming.

## **18.4. Accessing Struct Fields and Tags**

Reflection can be used to inspect and manipulate the fields of a struct, including reading and modifying struct tags. Struct tags are often used for purposes like JSON serialization.

### **Example 2: Inspecting Struct Fields**

```go
package main

import (
    "fmt"
    "reflect"
)

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    p := Person{"Alice", 30}
    t := reflect.TypeOf(p)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        fmt.Printf("Field Name: %s, Type: %s, Tag: %s
", field.Name, field.Type, field.Tag)
    }
}
```

#### **Explanation:**

- `t.NumField()` returns the number of fields in the struct.
- `t.Field(i)` returns metadata about the field, such as its name, type, and associated tags.
- In this example, we inspect the `Person` struct, printing its fields, types, and tags.

#### **Output:**

```
Field Name: Name, Type: string, Tag: json:"name"
Field Name: Age, Type: int, Tag: json:"age"
```

## **18.5. Setting Values Dynamically**

Reflection in Go allows you to modify variables dynamically at runtime. This can be useful when dealing with struct fields whose names and types are unknown at compile time.

### **Example 3: Modifying Struct Fields**

```go
package main

import (
    "fmt"
    "reflect"
)

type Person struct {
    Name string
    Age  int
}

func main() {
    p := Person{"Alice", 30}
    v := reflect.ValueOf(&p).Elem()

    // Modify fields
    v.FieldByName("Name").SetString("Bob")
    v.FieldByName("Age").SetInt(40)

    fmt.Println("Updated Struct:", p)
}
```

#### **Explanation:**

- `reflect.ValueOf(&p).Elem()` is used to get the address of the struct and modify its fields.
- `FieldByName("Name")` retrieves the field by its name and allows modification using methods like `SetString` and `SetInt`.

#### **Output:**

```
Updated Struct: {Bob 40}
```

## **18.6. Checking and Invoking Methods Dynamically**

Reflection can also be used to call methods on objects dynamically, which can be useful for cases like plugin systems or dynamic method dispatch.

### **Example 4: Calling Methods Dynamically**

```go
package main

import (
    "fmt"
    "reflect"
)

type Calculator struct{}

func (Calculator) Add(a, b int) int {
    return a + b
}

func main() {
    calc := Calculator{}
    method := reflect.ValueOf(calc).MethodByName("Add")
    result := method.Call([]reflect.Value{reflect.ValueOf(5), reflect.ValueOf(3)})
    fmt.Println("Result:", result[0].Int())
}
```

#### **Explanation:**

- `reflect.ValueOf(calc).MethodByName("Add")` retrieves the method named "Add" from the `Calculator` type.
- `method.Call()` is used to call the method with the provided arguments. The result is returned as a `reflect.Value`.

#### **Output:**

```
Result: 8
```

## **18.7. Use Case: JSON Validator**

Reflection is often used in Go for validating struct fields dynamically. For example, we can check for required fields using struct tags.

### **Example 5: Validating Required Fields**

```go
package main

import (
    "errors"
    "fmt"
    "reflect"
)

type User struct {
    Name  string `validate:"required"`
    Email string `validate:"required"`
    Age   int
}

func validateStruct(s interface{}) error {
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        tag := field.Tag.Get("validate")
        if tag == "required" && v.Field(i).IsZero() {
            return errors.New(field.Name + " is required")
        }
    }
    return nil
}

func main() {
    user := User{Name: "Alice", Age: 30}
    err := validateStruct(user)
    if err != nil {
        fmt.Println("Validation error:", err)
    } else {
        fmt.Println("Validation passed")
    }
}
```

#### **Explanation:**

- The `validateStruct` function checks if any fields with the `validate:"required"` tag are empty.
- `v.Field(i).IsZero()` checks if the field has its zero value (i.e., it has not been set).

#### **Output:**

```
Validation error: Email is required
```

## **18.8. Reflection Limitations**

While reflection is powerful, it comes with its own set of limitations:

1. **Performance**: Reflection is generally slower than directly accessing types and values. It should be used sparingly in performance-critical sections of your code.
2. **Complexity**: Reflection-based code can be harder to understand and maintain due to its dynamic nature. It can also be error-prone because errors are often discovered only at runtime.

## **18.9. Exercises**

## **Exercise 1: Inspecting Slice Elements**

**Problem**: Write a program to inspect the type and value of each element in a slice dynamically.

```go
package main

import (
    "fmt"
    "reflect"
)

func inspectSlice(slice interface{}) {
    v := reflect.ValueOf(slice)
    if v.Kind() == reflect.Slice {
        for i := 0; i < v.Len(); i++ {
            fmt.Printf("Element %d: Type = %s, Value = %v\n", i, v.Index(i).Type(), v.Index(i))
        }
    } else {
        fmt.Println("Provided input is not a slice")
    }
}

func main() {
    nums := []int{1, 2, 3}
    inspectSlice(nums)
}
```

**Output:**

```
Element 0: Type = int, Value = 1
Element 1: Type = int, Value = 2
Element 2: Type = int, Value = 3
```

---

## **Exercise 2: Copy Struct Fields Dynamically**

**Problem**: Write a function that dynamically copies fields from one struct to another struct of the same type.

```go
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name  string
    Email string
    Age   int
}

func copyStruct(src, dst interface{}) {
    srcVal := reflect.ValueOf(src)
    dstVal := reflect.ValueOf(dst).Elem()

    for i := 0; i < srcVal.NumField(); i++ {
        dstVal.Field(i).Set(srcVal.Field(i))
    }
}

func main() {
    user1 := User{"Alice", "alice@example.com", 25}
    var user2 User
    copyStruct(user1, &user2)
    fmt.Println("Copied Struct:", user2)
}
```

**Output:**

```
Copied Struct: {Alice alice@example.com 25}
```

---

## **Exercise 3: Setting Slice Values Dynamically**

**Problem**: Create a function to set values in a slice using reflection.

```go
package main

import (
    "fmt"
    "reflect"
)

func setSliceValues(slice interface{}, values []interface{}) {
    v := reflect.ValueOf(slice).Elem()
    for i := 0; i < len(values); i++ {
        v.Index(i).Set(reflect.ValueOf(values[i]))
    }
}

func main() {
    nums := make([]int, 3)
    setSliceValues(&nums, []interface{}{10, 20, 30})
    fmt.Println("Updated Slice:", nums)
}
```

**Output:**

```
Updated Slice: [10 20 30]
```

---

## **Exercise 4: Validating Struct Tags**

**Problem**: Write a function to validate struct fields based on custom tags.

```go
package main

import (
    "fmt"
    "reflect"
)

type Product struct {
    Name  string `validate:"required"`
    Price float64 `validate:"required"`
}

func validateStruct(s interface{}) {
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        if field.Tag.Get("validate") == "required" && v.Field(i).IsZero() {
            fmt.Printf("Field %s is required\n", field.Name)
        }
    }
}

func main() {
    product := Product{Name: "Laptop"}
    validateStruct(product)
}
```

**Output:**

```
Field Price is required
```

---

## **Exercise 5: Inspecting Function Signatures**

**Problem**: Write a program to inspect the parameters and return types of a function dynamically.

```go
package main

import (
    "fmt"
    "reflect"
)

func inspectFunction(fn interface{}) {
    t := reflect.TypeOf(fn)
    fmt.Println("Function Name:", t.Name())
    fmt.Println("Number of Parameters:", t.NumIn())
    fmt.Println("Number of Return Values:", t.NumOut())

    for i := 0; i < t.NumIn(); i++ {
        fmt.Printf("Parameter %d: %s\n", i, t.In(i))
    }
    for i := 0; i < t.NumOut(); i++ {
        fmt.Printf("Return Value %d: %s\n", i, t.Out(i))
    }
}

func add(a int, b int) int {
    return a + b
}

func main() {
    inspectFunction(add)
}
```

**Output:**

```
Function Name: add
Number of Parameters: 2
Number of Return Values: 1
Parameter 0: int
Parameter 1: int
Return Value 0: int
```

---

## **Exercise 6: Modifying Struct Values Dynamically**

**Problem**: Modify struct field values dynamically using reflection.

```go
package main

import (
    "fmt"
    "reflect"
)

type Employee struct {
    Name string
    Age  int
}

func updateStructField(s interface{}, fieldName string, value interface{}) {
    v := reflect.ValueOf(s).Elem()
    v.FieldByName(fieldName).Set(reflect.ValueOf(value))
}

func main() {
    emp := Employee{Name: "John", Age: 30}
    updateStructField(&emp, "Name", "Alice")
    updateStructField(&emp, "Age", 40)
    fmt.Println("Updated Employee:", emp)
}
```

**Output:**

```
Updated Employee: {Alice 40}
```

---

## **Exercise 7: JSON Validator**

**Problem**: Validate required JSON fields using struct tags and reflection.

```go
package main

import (
    "fmt"
    "reflect"
)

type Config struct {
    Host string `json:"host" validate:"required"`
    Port int    `json:"port"`
}

func validateJSON(config Config) {
    t := reflect.TypeOf(config)
    v := reflect.ValueOf(config)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        if field.Tag.Get("validate") == "required" && v.Field(i).IsZero() {
            fmt.Printf("Field %s is required\n", field.Name)
        }
    }
}

func main() {
    config := Config{Port: 8080}
    validateJSON(config)
}
```

**Output:**

```
Field Host is required
```

---

## **Exercise 8: Recursive Field Inspection**

**Problem**: Write a program to recursively inspect all fields of a nested struct.

```go
package main

import (
    "fmt"
    "reflect"
)

type Address struct {
    City string
    Zip  string
}

type Person struct {
    Name    string
    Age     int
    Address Address
}

func inspectStruct(s interface{}) {
    v := reflect.ValueOf(s)
    t := reflect.TypeOf(s)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("Field %s: %v\n", field.Name, value)
        if value.Kind() == reflect.Struct {
            inspectStruct(value.Interface())
        }
    }
}

func main() {
    p := Person{Name: "Alice", Age: 30, Address: Address{City: "Wonderland", Zip: "12345"}}
    inspectStruct(p)
}
```

**Output:**

```
Field Name: Alice
Field Age: 30
Field Address: {Wonderland 12345}
Field City: Wonderland
Field Zip: 12345
```
