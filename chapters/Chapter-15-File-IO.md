# **Chapter 15: File and I/O Operations in Go**

File and I/O operations are fundamental in most applications, from simple command-line tools to complex web services. Go provides powerful, yet easy-to-use packages for working with files, directories, and various data formats. In this chapter, we'll explore the core concepts of file handling in Go, and learn how to work with files, directories, and the popular JSON data format.

## **15.1 Basic File Operations**

Go's `os` package provides platform-independent functions for working with files and directories. Let's start with the most common file operations.

### **15.1.1 Reading Files**

There are several ways to read files in Go, each with its own advantages depending on your needs.

#### **Reading an Entire File**

For small files, you can read the entire content at once:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Read entire file content
	data, err := os.ReadFile("example.txt")
	if err != nil {
		fmt.Println("Error reading file:", err)
		return
	}

	// Convert bytes to string and print
	fmt.Println(string(data))
}
```

This approach is simple but not suitable for large files as it loads the entire content into memory.

#### **Reading a File Line by Line**

For larger files or when processing line by line, use a scanner:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// Open file
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Create scanner
	scanner := bufio.NewScanner(file)

	// Read line by line
	lineCount := 0
	for scanner.Scan() {
		lineCount++
		fmt.Printf("Line %d: %s\n", lineCount, scanner.Text())
	}

	// Check for errors during scanning
	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
}
```

This approach is memory-efficient for large files as it processes one line at a time.

#### **Reading with a Buffer**

For more control over how much data is read at once:

```go
package main

import (
	"fmt"
	"os"
	"io"
)

func main() {
	// Open file
	file, err := os.Open("example.txt")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Create buffer
	buffer := make([]byte, 32) // 32-byte buffer

	for {
		// Read up to buffer size
		bytesRead, err := file.Read(buffer)

		// End of file
		if err == io.EOF {
			break
		}

		// Other error
		if err != nil {
			fmt.Println("Error reading file:", err)
			return
		}

		// Process bytes
		fmt.Printf("Read %d bytes: %s\n", bytesRead, buffer[:bytesRead])
	}
}
```

This approach gives you precise control over memory usage when reading files.

### **15.1.2 Writing Files**

Go offers several methods for writing to files, each suited for different scenarios.

#### **Creating or Overwriting a File**

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create or overwrite a file
	file, err := os.Create("output.txt")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	// Write a string
	content := "Hello, Go file I/O!\nThis is a new file."
	bytesWritten, err := file.WriteString(content)
	if err != nil {
		fmt.Println("Error writing to file:", err)
		return
	}

	fmt.Printf("Wrote %d bytes to file\n", bytesWritten)
}
```

The `os.Create` function creates a new file or truncates an existing one.

#### **Appending to a File**

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open file in append mode
	file, err := os.OpenFile("output.txt", os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Append data
	newContent := "\nThis line is appended to the file."
	bytesWritten, err := file.WriteString(newContent)
	if err != nil {
		fmt.Println("Error appending to file:", err)
		return
	}

	fmt.Printf("Appended %d bytes to file\n", bytesWritten)
}
```

Using `os.O_APPEND` flag ensures that writes are appended to the file.

#### **Writing with a Buffer**

For better performance with multiple writes, use a buffered writer:

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	// Create or overwrite a file
	file, err := os.Create("buffered-output.txt")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	// Create a buffered writer
	writer := bufio.NewWriter(file)

	// Write multiple lines
	for i := 1; i <= 5; i++ {
		fmt.Fprintf(writer, "Line %d: Buffered writing is efficient\n", i)
	}

	// Flush buffer to ensure all data is written to file
	err = writer.Flush()
	if err != nil {
		fmt.Println("Error flushing buffer:", err)
		return
	}

	fmt.Println("Successfully wrote to file with buffering")
}
```

Buffered writing improves performance by reducing the number of system calls.

### **15.1.3 File Permissions and Modes**

When creating or opening files, you often need to specify permissions and modes:

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create a file with specific permissions
	// 0644 = -rw-r--r--
	file, err := os.OpenFile("permissions-example.txt",
							os.O_CREATE|os.O_WRONLY,
							0644)
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	file.WriteString("This file has specific permissions.")
	fmt.Println("File created with permissions 0644")

	// Change file permissions
	err = os.Chmod("permissions-example.txt", 0600) // -rw-------
	if err != nil {
		fmt.Println("Error changing permissions:", err)
		return
	}

	fmt.Println("Permissions changed to 0600")
}
```

Common file modes include:

| Mode | Binary    | Description       |
| ---- | --------- | ----------------- |
| 0400 | 100000000 | Read by owner     |
| 0200 | 010000000 | Write by owner    |
| 0100 | 001000000 | Execute by owner  |
| 0040 | 000100000 | Read by group     |
| 0020 | 000010000 | Write by group    |
| 0010 | 000001000 | Execute by group  |
| 0004 | 000000100 | Read by others    |
| 0002 | 000000010 | Write by others   |
| 0001 | 000000001 | Execute by others |

These can be combined, e.g., 0644 (read/write for owner, read for group and others).

### **15.1.4 File Metadata**

To get information about a file without reading its contents:

```go
package main

import (
	"fmt"
	"os"
	"time"
)

func main() {
	// Get file info
	info, err := os.Stat("example.txt")
	if err != nil {
		fmt.Println("Error getting file info:", err)
		return
	}

	// Print file metadata
	fmt.Printf("Name: %s\n", info.Name())
	fmt.Printf("Size: %d bytes\n", info.Size())
	fmt.Printf("Mode: %s\n", info.Mode())
	fmt.Printf("Modified: %s\n", info.ModTime().Format(time.RFC1123))
	fmt.Printf("Is Directory: %t\n", info.IsDir())
}
```

This provides details like file size, permissions, and modification time.

## **15.2 Directory Operations**

Working with directories is similar to working with files in Go.

### **15.2.1 Creating Directories**

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Create a single directory
	err := os.Mkdir("new-directory", 0755)
	if err != nil {
		fmt.Println("Error creating directory:", err)
		return
	}

	// Create nested directories
	err = os.MkdirAll("parent/child/grandchild", 0755)
	if err != nil {
		fmt.Println("Error creating nested directories:", err)
		return
	}

	fmt.Println("Directories created successfully")
}
```

`os.MkdirAll` creates parent directories if they don't exist, similar to `mkdir -p` in Unix.

### **15.2.2 Reading Directory Contents**

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Open directory
	dir, err := os.Open(".")
	if err != nil {
		fmt.Println("Error opening directory:", err)
		return
	}
	defer dir.Close()

	// Read directory entries
	entries, err := dir.ReadDir(0) // 0 means read all entries
	if err != nil {
		fmt.Println("Error reading directory:", err)
		return
	}

	// Process entries
	fmt.Println("Directory contents:")
	for i, entry := range entries {
		info, err := entry.Info()
		if err != nil {
			fmt.Printf("  %d. %s (error getting info)\n", i+1, entry.Name())
			continue
		}

		if info.IsDir() {
			fmt.Printf("  %d. [DIR] %s\n", i+1, info.Name())
		} else {
			fmt.Printf("  %d. [FILE] %s (%d bytes)\n", i+1, info.Name(), info.Size())
		}
	}
}
```

This reads and displays the contents of the current directory.

### **15.2.3 Walking a Directory Tree**

To recursively process all files and directories:

```go
package main

import (
	"fmt"
	"io/fs"
	"path/filepath"
)

func main() {
	// Walk the directory tree
	err := filepath.Walk(".", func(path string, info fs.FileInfo, err error) error {
		if err != nil {
			fmt.Printf("Error accessing %s: %v\n", path, err)
			return nil // Continue walking despite the error
		}

		// Indent based on depth
		indent := ""
		for i := 0; i < len(filepath.SplitList(path))-1; i++ {
			indent += "  "
		}

		// Print entry info
		if info.IsDir() {
			fmt.Printf("%s[DIR] %s\n", indent, info.Name())
		} else {
			fmt.Printf("%s[FILE] %s (%d bytes)\n", indent, info.Name(), info.Size())
		}

		return nil
	})

	if err != nil {
		fmt.Println("Error walking directory tree:", err)
	}
}
```

`filepath.Walk` is powerful for traversing directory structures.

### **15.2.4 Temporary Files and Directories**

For temporary operations:

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
)

func main() {
	// Create a temporary directory
	tempDir, err := os.MkdirTemp("", "example-*")
	if err != nil {
		fmt.Println("Error creating temp directory:", err)
		return
	}
	defer os.RemoveAll(tempDir) // Clean up when done

	fmt.Println("Created temporary directory:", tempDir)

	// Create a temporary file in that directory
	tempFile, err := os.CreateTemp(tempDir, "tempfile-*.txt")
	if err != nil {
		fmt.Println("Error creating temp file:", err)
		return
	}
	defer tempFile.Close()

	fmt.Println("Created temporary file:", tempFile.Name())

	// Write to the temporary file
	content := "This is temporary data that will be deleted"
	tempFile.WriteString(content)

	// Read back to verify
	info, _ := tempFile.Stat()
	fmt.Printf("Temporary file contains %d bytes\n", info.Size())

	// When the program exits, the deferred os.RemoveAll will clean up
	fmt.Println("Temporary resources will be removed on exit")
}
```

Temporary files and directories are useful for operations that need scratch space.

## **15.3 Working with JSON**

JSON (JavaScript Object Notation) is a popular data interchange format. Go's `encoding/json` package makes it easy to encode Go data structures to JSON and decode JSON into Go data structures.

### **15.3.1 Encoding JSON**

Converting Go data structures to JSON is straightforward:

```go
package main

import (
	"encoding/json"
	"fmt"
)

// Define a struct with JSON tags
type Person struct {
	FirstName string `json:"first_name"`
	LastName  string `json:"last_name"`
	Age       int    `json:"age"`
	Email     string `json:"email,omitempty"` // omitempty omits the field if empty
	Address   *Address `json:"address,omitempty"`
}

type Address struct {
	Street  string `json:"street"`
	City    string `json:"city"`
	Country string `json:"country"`
}

func main() {
	// Create some data
	people := []Person{
		{
			FirstName: "John",
			LastName:  "Doe",
			Age:       30,
			Email:     "john@example.com",
			Address: &Address{
				Street:  "123 Main St",
				City:    "Boston",
				Country: "USA",
			},
		},
		{
			FirstName: "Jane",
			LastName:  "Smith",
			Age:       25,
			// Email omitted
		},
	}

	// Marshal to JSON (compact)
	data, err := json.Marshal(people)
	if err != nil {
		fmt.Println("Error marshaling JSON:", err)
		return
	}

	fmt.Println("Compact JSON:")
	fmt.Println(string(data))

	// Marshal to JSON with indentation
	prettyData, err := json.MarshalIndent(people, "", "  ")
	if err != nil {
		fmt.Println("Error marshaling JSON:", err)
		return
	}

	fmt.Println("\nPretty JSON:")
	fmt.Println(string(prettyData))
}
```

This demonstrates:

- Struct tags for controlling JSON field names
- The `omitempty` option for omitting empty fields
- Both compact and indented JSON output

### **15.3.2 Decoding JSON**

Converting JSON back to Go data structures:

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Person struct {
	FirstName string   `json:"first_name"`
	LastName  string   `json:"last_name"`
	Age       int      `json:"age"`
	Email     string   `json:"email,omitempty"`
	Address   *Address `json:"address,omitempty"`
}

type Address struct {
	Street  string `json:"street"`
	City    string `json:"city"`
	Country string `json:"country"`
}

func main() {
	// JSON data
	jsonData := `[
	  {
		"first_name": "John",
		"last_name": "Doe",
		"age": 30,
		"email": "john@example.com",
		"address": {
		  "street": "123 Main St",
		  "city": "Boston",
		  "country": "USA"
		}
	  },
	  {
		"first_name": "Jane",
		"last_name": "Smith",
		"age": 25
	  }
	]`

	// Unmarshal JSON into slice of Person
	var people []Person
	err := json.Unmarshal([]byte(jsonData), &people)
	if err != nil {
		fmt.Println("Error unmarshaling JSON:", err)
		return
	}

	// Print the results
	for i, person := range people {
		fmt.Printf("Person %d: %s %s, age %d\n",
			i+1, person.FirstName, person.LastName, person.Age)

		if person.Email != "" {
			fmt.Printf("  Email: %s\n", person.Email)
		}

		if person.Address != nil {
			fmt.Printf("  Address: %s, %s, %s\n",
				person.Address.Street, person.Address.City, person.Address.Country)
		}
	}
}
```

This shows how to decode JSON into Go structs, with proper handling of nested structures and optional fields.

### **15.3.3 Working with JSON Files**

Combining file I/O with JSON operations is common in real applications:

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type Configuration struct {
	ServerName  string   `json:"server_name"`
	Port        int      `json:"port"`
	DatabaseURL string   `json:"database_url"`
	Features    []string `json:"features"`
	MaxUsers    int      `json:"max_users"`
}

func main() {
	// Create a configuration
	config := Configuration{
		ServerName:  "production-server",
		Port:        8080,
		DatabaseURL: "postgresql://user:password@localhost/db",
		Features:    []string{"auth", "logging", "metrics"},
		MaxUsers:    1000,
	}

	// Save configuration to file
	saveConfig := func(filename string, config Configuration) error {
		file, err := os.Create(filename)
		if err != nil {
			return err
		}
		defer file.Close()

		encoder := json.NewEncoder(file)
		encoder.SetIndent("", "  ")
		return encoder.Encode(config)
	}

	err := saveConfig("config.json", config)
	if err != nil {
		fmt.Println("Error saving config:", err)
		return
	}

	fmt.Println("Configuration saved to config.json")

	// Load configuration from file
	loadConfig := func(filename string) (Configuration, error) {
		var config Configuration

		file, err := os.Open(filename)
		if err != nil {
			return config, err
		}
		defer file.Close()

		decoder := json.NewDecoder(file)
		err = decoder.Decode(&config)
		return config, err
	}

	loadedConfig, err := loadConfig("config.json")
	if err != nil {
		fmt.Println("Error loading config:", err)
		return
	}

	fmt.Println("Loaded configuration:")
	fmt.Printf("  Server: %s:%d\n", loadedConfig.ServerName, loadedConfig.Port)
	fmt.Printf("  Database: %s\n", loadedConfig.DatabaseURL)
	fmt.Printf("  Features: %v\n", loadedConfig.Features)
	fmt.Printf("  Max Users: %d\n", loadedConfig.MaxUsers)
}
```

This demonstrates:

- Writing JSON to a file using an encoder
- Reading JSON from a file using a decoder
- Practical application with configuration files

## **15.4 Working with CSV Files**

Comma-Separated Values (CSV) files are common for tabular data. Go's `encoding/csv` package provides tools for working with CSV files.

### **15.4.1 Reading CSV Files**

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

func main() {
	// Open the CSV file
	file, err := os.Open("data.csv")
	if err != nil {
		fmt.Println("Error opening file:", err)
		return
	}
	defer file.Close()

	// Create a CSV reader
	reader := csv.NewReader(file)

	// Read all records at once
	records, err := reader.ReadAll()
	if err != nil {
		fmt.Println("Error reading CSV:", err)
		return
	}

	// Process the records
	fmt.Printf("Found %d records:\n", len(records))
	for i, record := range records {
		fmt.Printf("  Record %d: %v\n", i, record)
	}

	// Alternative: read records one at a time
	file.Seek(0, 0) // Go back to beginning of file
	reader = csv.NewReader(file) // Create a new reader

	fmt.Println("\nReading records one by one:")
	for {
		record, err := reader.Read()
		if err != nil {
			// End of file is expected
			break
		}

		// Process each record
		fmt.Printf("  Record: %v\n", record)
	}
}
```

For this example, assume a CSV file (`data.csv`) with content like:

```
Name,Age,City
John Doe,30,New York
Jane Smith,25,Boston
```

### **15.4.2 Writing CSV Files**

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

func main() {
	// Create the CSV file
	file, err := os.Create("users.csv")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	// Create a CSV writer
	writer := csv.NewWriter(file)
	defer writer.Flush()

	// Write header
	header := []string{"ID", "Name", "Email", "Age"}
	err = writer.Write(header)
	if err != nil {
		fmt.Println("Error writing header:", err)
		return
	}

	// Write data
	data := [][]string{
		{"1", "John Doe", "john@example.com", "30"},
		{"2", "Jane Smith", "jane@example.com", "25"},
		{"3", "Bob Johnson", "bob@example.com", "45"},
	}

	for _, record := range data {
		err := writer.Write(record)
		if err != nil {
			fmt.Println("Error writing record:", err)
			return
		}
	}

	fmt.Println("CSV file created successfully")
}
```

This creates a CSV file with headers and data rows.

### **15.4.3 CSV with Custom Delimiters**

CSV isn't always comma-separated; sometimes tabs or other characters are used:

```go
package main

import (
	"encoding/csv"
	"fmt"
	"os"
)

func main() {
	// Create the TSV file (tab-separated values)
	file, err := os.Create("data.tsv")
	if err != nil {
		fmt.Println("Error creating file:", err)
		return
	}
	defer file.Close()

	// Create a CSV writer with tab delimiter
	writer := csv.NewWriter(file)
	writer.Comma = '\t' // Set the delimiter to tab
	defer writer.Flush()

	// Write data
	data := [][]string{
		{"Name", "City", "Score"},
		{"John", "New York", "8.5"},
		{"Jane", "Boston", "9.0"},
	}

	err = writer.WriteAll(data)
	if err != nil {
		fmt.Println("Error writing TSV:", err)
		return
	}

	fmt.Println("TSV file created successfully")
}
```

This creates a tab-separated file instead of a comma-separated one.

## **15.5 Error Handling Best Practices**

Proper error handling is crucial when working with files. Here are some patterns for robust error handling.

### **15.5.1 Checking Existence Before Opening**

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	filename := "config.json"

	// Check if file exists
	if _, err := os.Stat(filename); os.IsNotExist(err) {
		fmt.Printf("File %s does not exist\n", filename)
		// Create the file or take appropriate action
		return
	}

	// Open existing file
	file, err := os.Open(filename)
	if err != nil {
		fmt.Printf("Error opening %s: %v\n", filename, err)
		return
	}
	defer file.Close()

	fmt.Printf("Successfully opened %s\n", filename)
	// Continue with file operations
}
```

This pattern avoids the common error of trying to open a non-existent file.

### **15.5.2 Using Defer for Cleanup**

The `defer` statement ensures that resources are properly released:

```go
package main

import (
	"fmt"
	"os"
)

func processFile(filename string) error {
	// Open file
	file, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("error opening %s: %w", filename, err)
	}
	defer file.Close() // This ensures the file is closed even if errors occur

	// Read from file
	buffer := make([]byte, 100)
	_, err = file.Read(buffer)
	if err != nil {
		return fmt.Errorf("error reading %s: %w", filename, err)
	}

	// Process data
	// ...

	return nil
}

func main() {
	err := processFile("example.txt")
	if err != nil {
		fmt.Println("Error:", err)
		return
	}

	fmt.Println("File processed successfully")
}
```

Using `defer` for cleanup is a Go best practice that ensures resources are properly managed.

### **15.5.3 Error Wrapping**

Go 1.13+ supports error wrapping, which helps build rich error chains:

```go
package main

import (
	"errors"
	"fmt"
	"os"
)

func readConfig() error {
	filename := "config.json"

	// Try to open the file
	file, err := os.Open(filename)
	if err != nil {
		return fmt.Errorf("failed to open config: %w", err)
	}
	defer file.Close()

	// Read the file content
	data := make([]byte, 1024)
	_, err = file.Read(data)
	if err != nil {
		return fmt.Errorf("failed to read config: %w", err)
	}

	// Parse the JSON
	// ...

	return nil
}

func main() {
	err := readConfig()
	if err != nil {
		fmt.Println("Configuration error:", err)

		// Check for specific error types
		if errors.Is(err, os.ErrNotExist) {
			fmt.Println("Config file is missing. Creating default config...")
			// Create default config
		}

		return
	}

	fmt.Println("Configuration loaded successfully")
}
```

Error wrapping preserves the original error while adding context, making debugging easier.

## **15.6 Working with Paths**

The `path/filepath` package provides functions for manipulating file paths in a platform-independent way.

```go
package main

import (
	"fmt"
	"path/filepath"
	"strings"
)

func main() {
	// Join path components
	path := filepath.Join("users", "john", "documents", "report.pdf")
	fmt.Println("Joined path:", path)

	// Get the directory and file components
	dir := filepath.Dir(path)
	file := filepath.Base(path)
	fmt.Println("Directory:", dir)
	fmt.Println("File:", file)

	// Get the file extension
	ext := filepath.Ext(path)
	fmt.Println("Extension:", ext)

	// Get filename without extension
	name := strings.TrimSuffix(file, ext)
	fmt.Println("Name without extension:", name)

	// Absolute path
	absPath, err := filepath.Abs("report.pdf")
	if err != nil {
		fmt.Println("Error getting absolute path:", err)
	} else {
		fmt.Println("Absolute path:", absPath)
	}

	// Check if path is absolute
	isAbs := filepath.IsAbs(path)
	fmt.Println("Is absolute path:", isAbs)

	// Clean a path (remove redundant elements)
	messyPath := "users/./john/../john/docs//report.pdf"
	cleanPath := filepath.Clean(messyPath)
	fmt.Println("Clean path:", cleanPath)
}
```

Using `filepath` makes your code portable across different operating systems.

## **15.7 Exercises**

### **Exercise 1: File Copy Utility**

Write a program that copies a file from a source path to a destination path, showing progress during the copy.

```go
package main

import (
	"fmt"
	"io"
	"os"
)

func copyFile(src, dst string) error {
	// Open source file
	sourceFile, err := os.Open(src)
	if err != nil {
		return err
	}
	defer sourceFile.Close()

	// Get source file info
	sourceInfo, err := sourceFile.Stat()
	if err != nil {
		return err
	}

	// Create destination file
	destFile, err := os.Create(dst)
	if err != nil {
		return err
	}
	defer destFile.Close()

	// Copy with progress reporting
	buffer := make([]byte, 32*1024) // 32KB buffer
	totalBytes := sourceInfo.Size()
	copiedBytes := int64(0)

	for {
		bytesRead, err := sourceFile.Read(buffer)
		if err == io.EOF {
			break
		}
		if err != nil {
			return err
		}

		_, err = destFile.Write(buffer[:bytesRead])
		if err != nil {
			return err
		}

		copiedBytes += int64(bytesRead)
		progress := float64(copiedBytes) / float64(totalBytes) * 100
		fmt.Printf("\rCopying: %.1f%% complete", progress)
	}

	fmt.Println("\nCopy complete!")
	return nil
}

func main() {
	if len(os.Args) != 3 {
		fmt.Println("Usage: go run filecopy.go <source> <destination>")
		return
	}

	source := os.Args[1]
	destination := os.Args[2]

	err := copyFile(source, destination)
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
}
```

### **Exercise 2: Simple JSON Configuration Manager**

Create a program that loads, modifies, and saves a JSON configuration file.

```go
package main

import (
	"encoding/json"
	"fmt"
	"os"
)

type AppConfig struct {
	AppName     string            `json:"app_name"`
	Version     string            `json:"version"`
	Environment string            `json:"environment"`
	LogLevel    string            `json:"log_level"`
	Database    DatabaseConfig    `json:"database"`
	Settings    map[string]string `json:"settings"`
}

type DatabaseConfig struct {
	Host     string `json:"host"`
	Port     int    `json:"port"`
	Username string `json:"username"`
	Password string `json:"password"`
	DBName   string `json:"db_name"`
}

// Load configuration from file
func loadConfig(filename string) (AppConfig, error) {
	var config AppConfig

	file, err := os.Open(filename)
	if err != nil {
		if os.IsNotExist(err) {
			// Return default config if file doesn't exist
			return defaultConfig(), nil
		}
		return config, err
	}
	defer file.Close()

	decoder := json.NewDecoder(file)
	err = decoder.Decode(&config)
	if err != nil {
		return config, err
	}

	return config, nil
}

// Save configuration to file
func saveConfig(config AppConfig, filename string) error {
	file, err := os.Create(filename)
	if err != nil {
		return err
	}
	defer file.Close()

	encoder := json.NewEncoder(file)
	encoder.SetIndent("", "  ")
	return encoder.Encode(config)
}

// Create default configuration
func defaultConfig() AppConfig {
	return AppConfig{
		AppName:     "MyApp",
		Version:     "1.0.0",
		Environment: "development",
		LogLevel:    "info",
		Database: DatabaseConfig{
			Host:     "localhost",
			Port:     5432,
			Username: "user",
			Password: "password",
			DBName:   "myapp",
		},
		Settings: map[string]string{
			"theme":       "light",
			"auto_save":   "true",
			"max_threads": "4",
		},
	}
}

func main() {
	configFile := "app_config.json"

	// Load configuration
	config, err := loadConfig(configFile)
	if err != nil {
		fmt.Println("Error loading config:", err)
		return
	}

	// Display current configuration
	fmt.Println("Current configuration:")
	fmt.Printf("  App: %s v%s (%s)\n",
		config.AppName, config.Version, config.Environment)
	fmt.Printf("  Database: %s@%s:%d/%s\n",
		config.Database.Username, config.Database.Host,
		config.Database.Port, config.Database.DBName)

	// Modify configuration
	config.LogLevel = "debug"
	config.Settings["theme"] = "dark"

	// Save updated configuration
	err = saveConfig(config, configFile)
	if err != nil {
		fmt.Println("Error saving config:", err)
		return
	}

	fmt.Println("Configuration updated and saved to", configFile)
}
```

These exercises provide practical examples of file and JSON operations that you might use in real applications.

## **15.8 Summary**

In this chapter, we've explored Go's powerful file and I/O operations:

- Basic file operations for reading and writing
- Directory manipulation
- Working with JSON and CSV data formats
- Error handling best practices
- Path manipulation with `filepath`

The Go standard library provides a rich set of tools for working with files and various data formats, making it easy to build robust applications that interact with the file system and handle structured data.

Understanding these core concepts will serve as a foundation for more advanced topics, such as working with databases, developing web services, and building command-line tools.

**Next Up**: In Chapter 16, we'll explore testing in Go, including unit testing, table-driven tests, benchmarking, and test coverage.
