# **Chapter 14: File and I/O Operations**

---

## **14.1 Reading and Writing Files**

Go’s `os` and `io` packages make file handling straightforward. Let’s dive into reading, writing, and appending data to files.

### **14.1.1 Writing to a File**

**Example 1: Writing a Simple Text File**

```go
package main

import (
	"os"
)

func main() {
	// Create or overwrite the file
	file, err := os.Create("example.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// Write to the file
	content := "Hello, Go! Writing to files is fun."
	_, err = file.WriteString(content)
	if err != nil {
		panic(err)
	}

	println("File written successfully!")
}
```

**Output:**

- Creates a file named `example.txt` with the content:
  ```
  Hello, Go! Writing to files is fun.
  ```

---

### **14.1.2 Reading from a File**

**Example 2: Reading a File Line by Line**

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("example.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		panic(err)
	}
}
```

**Output:**

```
Hello, Go! Writing to files is fun.
```

---

### **14.1.3 Appending to a File**

**Example 3: Adding More Content**

```go
package main

import (
	"os"
)

func main() {
	file, err := os.OpenFile("example.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	// Append content
	_, err = file.WriteString("
This is an appended line.")
	if err != nil {
		panic(err)
	}

	println("Content appended successfully!")
}
```

**Output in `example.txt`:**

```
Hello, Go! Writing to files is fun.
This is an appended line.
```

---

## **14.2 Working with JSON**

JSON (JavaScript Object Notation) is a lightweight data-interchange format. In Go, JSON handling is supported by the `encoding/json` package.

### **14.2.1 Encoding JSON**

**Example 4: Struct to JSON**

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func main() {
	user := User{Name: "John Doe", Email: "john.doe@example.com", Age: 30}
	jsonData, err := json.Marshal(user)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(jsonData))
}
```

**Output:**

```json
{ "name": "John Doe", "email": "john.doe@example.com", "age": 30 }
```

---

### **14.2.2 Decoding JSON**

**Example 5: JSON to Struct**

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	Age   int    `json:"age"`
}

func main() {
	jsonData := `{"name":"Jane Smith","email":"jane.smith@example.com","age":25}`
	var user User
	err := json.Unmarshal([]byte(jsonData), &user)
	if err != nil {
		panic(err)
	}
	fmt.Printf("Name: %s, Email: %s, Age: %d\n", user.Name, user.Email, user.Age)
}
```

**Output:**

```
Name: Jane Smith, Email: jane.smith@example.com, Age: 25
```

---

### **14.2.3 Writing JSON to a File**

**Example 6: Save Struct as JSON File**

```go
package main

import (
	"encoding/json"
	"os"
)

type Product struct {
	ID    int    `json:"id"`
	Name  string `json:"name"`
	Price int    `json:"price"`
}

func main() {
	product := Product{ID: 101, Name: "Laptop", Price: 750}
	file, err := os.Create("product.json")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	encoder := json.NewEncoder(file)
	err = encoder.Encode(product)
	if err != nil {
		panic(err)
	}
	println("JSON saved to product.json")
}
```

**File Content:**

```json
{
  "id": 101,
  "name": "Laptop",
  "price": 750
}
```

---

## **Summary**

| Feature              | Key Functions/Packages     | Example Numbers |
| -------------------- | -------------------------- | --------------- |
| Writing Files        | `os.Create`, `Write`       | 1, 3            |
| Reading Files        | `os.Open`, `bufio.Scanner` | 2               |
| JSON Encoding        | `json.Marshal`             | 4               |
| JSON Decoding        | `json.Unmarshal`           | 5               |
| JSON File Operations | `json.NewEncoder`          | 6               |

By mastering these techniques, you can confidently handle file and JSON operations in real-world applications!

---

# **14.3. Exercises**

---

## **Exercise 1: Writing to a File**

**Problem**: Write a program to create a file called `greetings.txt` and write the following text into it:

```
Hello, World!
Welcome to Go programming.
```

```go
package main

import (
	"os"
)

func main() {
	file, err := os.Create("greetings.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	content := "Hello, World!\nWelcome to Go programming."
	_, err = file.WriteString(content)
	if err != nil {
		panic(err)
	}
	println("File created and content written successfully.")
}
```

**Output in `greetings.txt`:**

```
Hello, World!
Welcome to Go programming.
```

---

## **Exercise 2: Reading from a File**

**Problem**: Write a program to read and display the content of `greetings.txt` line by line.

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("greetings.txt")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	scanner := bufio.NewScanner(file)
	for scanner.Scan() {
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		panic(err)
	}
}
```

**Output:**

```
Hello, World!
Welcome to Go programming.
```

---

## **Exercise 3: Appending to a File**

**Problem**: Append the following line to `greetings.txt`:

```
Let's explore file handling in Go!
```

```go
package main

import (
	"os"
)

func main() {
	file, err := os.OpenFile("greetings.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		panic(err)
	}
	defer file.Close()

	_, err = file.WriteString("\nLet's explore file handling in Go!")
	if err != nil {
		panic(err)
	}
	println("Content appended successfully.")
}
```

**Updated File Content:**

```
Hello, World!
Welcome to Go programming.
Let's explore file handling in Go!
```

---

## **Exercise 4: JSON Encoding**

**Problem**: Create a program to encode a list of users as JSON and print the result.

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func main() {
	users := []User{
		{"Alice", "alice@example.com"},
		{"Bob", "bob@example.com"},
	}

	jsonData, err := json.MarshalIndent(users, "", "  ")
	if err != nil {
		panic(err)
	}
	fmt.Println(string(jsonData))
}
```

**Output:**

```json
[
  {
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "name": "Bob",
    "email": "bob@example.com"
  }
]
```

---

## **Exercise 5: JSON Decoding**

**Problem**: Decode the JSON data from Exercise 4 back into a list of users.

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func main() {
	jsonData := `[{"name":"Alice","email":"alice@example.com"},{"name":"Bob","email":"bob@example.com"}]`

	var users []User
	err := json.Unmarshal([]byte(jsonData), &users)
	if err != nil {
		panic(err)
	}

	for _, user := range users {
		fmt.Printf("Name: %s, Email: %s\n", user.Name, user.Email)
	}
}
```

**Output:**

```
Name: Alice, Email: alice@example.com
Name: Bob, Email: bob@example.com
```

---

## **Exercise 6: Write JSON to a File**

**Problem**: Save the list of users from Exercise 4 into a file named `users.json`.

```go
package main

import (
	"encoding/json"
	"os"
)

type User struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func main() {
	users := []User{
		{"Alice", "alice@example.com"},
		{"Bob", "bob@example.com"},
	}

	file, err := os.Create("users.json")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	encoder := json.NewEncoder(file)
	err = encoder.Encode(users)
	if err != nil {
		panic(err)
	}
	println("JSON data saved to users.json.")
}
```

**File Content (`users.json`):**

```json
[
  {
    "name": "Alice",
    "email": "alice@example.com"
  },
  {
    "name": "Bob",
    "email": "bob@example.com"
  }
]
```

---

## **Exercise 7: CSV File Parsing**

**Problem**: Parse a CSV file containing user data.

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

func main() {
	file, err := os.Open("users.csv")
	if err != nil {
		panic(err)
	}
	defer file.Close()

	reader := csv.NewReader(file)
	records, err := reader.ReadAll()
	if err != nil {
		panic(err)
	}

	for _, record := range records {
		fmt.Printf("Name: %s, Email: %s\n", record[0], record[1])
	}
}
```

**Input (`users.csv`):**

```
Alice,alice@example.com
Bob,bob@example.com
```

**Output:**

```
Name: Alice, Email: alice@example.com
Name: Bob, Email: bob@example.com
```

---

## **Exercise 8: File Size**

**Problem**: Write a program to check the size of `greetings.txt`.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	info, err := os.Stat("greetings.txt")
	if err != nil {
		panic(err)
	}

	fmt.Printf("File size: %d bytes\n", info.Size())
}
```

**Output:**

```
File size: 87 bytes
```

---

## **Exercise 9: File Copy**

**Problem**: Write a program to copy the content of `greetings.txt` to `copy.txt`.

```go
package main

import (
	"io"
	"os"
)

func main() {
	src, err := os.Open("greetings.txt")
	if err != nil {
		panic(err)
	}
	defer src.Close()

	dest, err := os.Create("copy.txt")
	if err != nil {
		panic(err)
	}
	defer dest.Close()

	_, err = io.Copy(dest, src)
	if err != nil {
		panic(err)
	}

	println("File copied successfully!")
}
```

**Output:**

- `copy.txt` will have the same content as `greetings.txt`.

---

## **Exercise 10: Temporary Files**

**Problem**: Create a temporary file and write data into it.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.CreateTemp("", "tempfile_*.txt")
	if err != nil {
		panic(err)
	}
	defer os.Remove(file.Name())

	fmt.Println("Temporary file created:", file.Name())
	_, err = file.WriteString("Temporary file content.")
	if err != nil {
		panic(err)
	}

	println("Temporary file written successfully!")
}
```

**Output:**

```
Temporary file created: /tmp/tempfile_abc123.txt
Temporary file written successfully!
```

---

**Congratulations!** You’ve completed the exercises for Chapter 14. These practical examples demonstrate the power of file and I/O operations in real-world scenarios.

**Happy Coding!** ✨
