# **Chapter 18: Reflection**

---

## **18.1. What is Reflection?**

Reflection is the ability of a program to:

1. Inspect its structure (types, fields, methods).
2. Dynamically manipulate objects at runtime.

### **Key Concepts**

| Concept   | Description                                                       |
| --------- | ----------------------------------------------------------------- |
| **Type**  | The `reflect.Type` interface represents the type of a value.      |
| **Value** | The `reflect.Value` interface represents the value of a variable. |
| **Kind**  | The specific category of a type (e.g., int, string, slice).       |

Reflection is often used for:

- Serialization (e.g., JSON or XML marshaling).
- Dynamic method invocation.
- Validating and transforming struct fields.

---

## **18.2. The Reflect Package Basics**

Let’s start with the basics of the `reflect` package.

### **Example 1: Inspecting Types**

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    x := 42
    t := reflect.TypeOf(x)
    v := reflect.ValueOf(x)

    fmt.Println("Type:", t)
    fmt.Println("Value:", v)
    fmt.Println("Kind:", t.Kind())
}
```

#### **Output**

```
Type: int
Value: 42
Kind: int
```

### Explanation:

- `reflect.TypeOf(x)` gets the type of `x`.
- `reflect.ValueOf(x)` gets the runtime value of `x`.
- `t.Kind()` returns the "kind" (basic category) of the type.

---

## **18.3. Accessing Struct Fields and Tags**

Reflection can inspect and manipulate struct fields, including their tags.

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
        fmt.Printf("Field Name: %s, Type: %s, Tag: %s\n", field.Name, field.Type, field.Tag)
    }
}
```

#### **Output**

```
Field Name: Name, Type: string, Tag: json:"name"
Field Name: Age, Type: int, Tag: json:"age"
```

### Explanation:

- `t.NumField()` returns the number of fields in the struct.
- `t.Field(i)` retrieves the metadata of each field, including its name, type, and tag.

---

## **18.4. Setting Values Dynamically**

Reflection allows us to change values of variables at runtime.

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

#### **Output**

```
Updated Struct: {Bob 40}
```

### Note:

- The `reflect.ValueOf(&p).Elem()` is used to get a pointer to the struct and modify its fields.

---

## **18.5. Checking and Invoking Methods Dynamically**

You can also call methods on objects dynamically using reflection.

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

#### **Output**

```
Result: 8
```

---

## **18.6. Use Case: JSON Validator**

Reflection is commonly used to validate struct fields dynamically.

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

#### **Output**

```
Validation error: Email is required
```

---

## **18.7. Reflection Limitations**

While powerful, reflection has limitations:

- **Performance**: Slower than direct code.
- **Complexity**: Harder to read and maintain.

---

## **18.8. Exercises**

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

---

**Congratulations!** You've completed the reflection exercises. These examples are designed to give you hands-on experience with practical applications of reflection in Go.
