
# **Chapter 24: Building CLI Tools in Go**

---

## **24.1. Introduction**

Command-Line Interfaces (CLI) are an essential part of many software applications, allowing developers and system administrators to interact with programs using terminal commands. Go’s standard library and ecosystem make it an excellent choice for building powerful and efficient CLI tools.

This chapter will guide you through:
1. Creating command-line applications.
2. Handling arguments and flags.
3. Managing configurations.

By the end of this chapter, you will be able to design, implement, and distribute your own CLI tools.

---

## **24.2. Command-Line Applications**

Go’s `os` and `flag` packages provide core functionality for building CLI tools. Additional libraries like `cobra` can simplify the creation of more complex applications.

### **1. Basic CLI Tool**

**Scenario**: Create a tool that greets a user.

**Code Example**:
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Println("Usage: greet <name>")
		return
	}

	name := os.Args[1]
	fmt.Printf("Hello, %s!
", name)
}
```

**Explanation**:
- `os.Args` captures command-line arguments.
- The first argument (`os.Args[0]`) is the program name.
- Subsequent arguments are the inputs provided by the user.

**Run**:
```bash
go run main.go Alice
# Output: Hello, Alice!
```

---

### **2. Advanced CLI with `cobra`**
# Advanced CLI with Cobra

Cobra is a powerful library for building advanced Command Line Interface (CLI) tools in Go. It allows you to easily create nested commands, parse flags, and generate documentation for your CLI tools. This tutorial will walk you through creating a CLI application using Cobra.

---

## **Installation**

To get started with Cobra, install the necessary packages:

```bash
go get -u github.com/spf13/cobra@latest
go get -u github.com/spf13/cobra/cobra@latest
```

---

## **Step 1: Creating a Basic Cobra CLI**

Here’s a minimal example of using Cobra to create a CLI application.

### **Code Example**

```go
package main

import (
	"fmt"
	"github.com/spf13/cobra"
)

func main() {
	var rootCmd = &cobra.Command{
		Use:   "greet",
		Short: "Greet CLI",
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("Welcome to the Greet CLI!")
		},
	}

	var helloCmd = &cobra.Command{
		Use:   "hello",
		Short: "Say hello",
		Run: func(cmd *cobra.Command, args []string) {
			if len(args) < 1 {
				fmt.Println("Usage: greet hello <name>")
				return
			}
			fmt.Printf("Hello, %s!\n", args[0])
		},
	}

	rootCmd.AddCommand(helloCmd)
	rootCmd.Execute()
}
```

---

## **Step 2: Running the Application**

Save the above code in a file named `main.go`. Then, execute the following commands in your terminal:

```bash
go run main.go hello Alice
```

### **Output**

```plaintext
Hello, Alice!
```

---

## **Step 3: Adding More Commands**

You can extend your CLI by adding additional commands. For example, let’s add a `goodbye` command:

### **Updated Code**

```go
package main

import (
	"fmt"
	"github.com/spf13/cobra"
)

func main() {
	var rootCmd = &cobra.Command{
		Use:   "greet",
		Short: "Greet CLI",
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("Welcome to the Greet CLI!")
		},
	}

	var helloCmd = &cobra.Command{
		Use:   "hello",
		Short: "Say hello",
		Run: func(cmd *cobra.Command, args []string) {
			if len(args) < 1 {
				fmt.Println("Usage: greet hello <name>")
				return
			}
			fmt.Printf("Hello, %s!\n", args[0])
		},
	}

	var goodbyeCmd = &cobra.Command{
		Use:   "goodbye",
		Short: "Say goodbye",
		Run: func(cmd *cobra.Command, args []string) {
			if len(args) < 1 {
				fmt.Println("Usage: greet goodbye <name>")
				return
			}
			fmt.Printf("Goodbye, %s!\n", args[0])
		},
	}

	rootCmd.AddCommand(helloCmd, goodbyeCmd)
	rootCmd.Execute()
}
```

---

## **Step 4: Running with Multiple Commands**

Now you can use the `hello` and `goodbye` commands:

### **Example Usage**

```bash
go run main.go hello Bob
```

**Output:**

```plaintext
Hello, Bob!
```

```bash
go run main.go goodbye Bob
```

**Output:**

```plaintext
Goodbye, Bob!
```

---

## **Step 5: Using Flags**

Cobra supports flags for more advanced options. Let’s add a `--shout` flag to the `hello` command to print the message in uppercase.

### **Code Example with Flags**

```go
package main

import (
	"fmt"
	"strings"
	"github.com/spf13/cobra"
)

func main() {
	var shout bool

	var rootCmd = &cobra.Command{
		Use:   "greet",
		Short: "Greet CLI",
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("Welcome to the Greet CLI!")
		},
	}

	var helloCmd = &cobra.Command{
		Use:   "hello",
		Short: "Say hello",
		Run: func(cmd *cobra.Command, args []string) {
			if len(args) < 1 {
				fmt.Println("Usage: greet hello <name>")
				return
			}
			message := fmt.Sprintf("Hello, %s!", args[0])
			if shout {
				message = strings.ToUpper(message)
			}
			fmt.Println(message)
		},
	}

	helloCmd.Flags().BoolVarP(&shout, "shout", "s", false, "Shout the greeting")
	rootCmd.AddCommand(helloCmd)
	rootCmd.Execute()
}
```

### **Usage with Flags**

```bash
go run main.go hello Bob --shout
```

**Output:**

```plaintext
HELLO, BOB!
```

---

## **Step 6: Generating Documentation**

Cobra can automatically generate documentation for your CLI tool. Use the following command to generate markdown files for all commands:

```bash
cobra gen docs
```

This will create a `docs/` folder with detailed documentation for your commands.

---

## **Conclusion**

Cobra makes it easy to build complex, feature-rich CLI tools in Go. By organizing commands, using flags, and generating documentation, you can create powerful tools tailored to your needs. For more details, visit the [Cobra documentation](https://github.com/spf13/cobra).


## **24.3. Handling Arguments and Flags**

### **1. Using Flags with `flag`**

The `flag` package allows you to define and parse command-line flags.

**Code Example**:
```go
package main

import (
	"flag"
	"fmt"
)

func main() {
	name := flag.String("name", "World", "name to greet")
	age := flag.Int("age", 0, "your age")

	flag.Parse()

	fmt.Printf("Hello, %s! You are %d years old.
", *name, *age)
}
```

**Explanation**:
- `flag.String` and `flag.Int` define flags with default values.
- `flag.Parse` parses the provided flags and assigns values.

**Run**:
```bash
go run main.go -name Alice -age 30
# Output: Hello, Alice! You are 30 years old.
```

---

### **2. Positional Arguments with Flags**

You can combine flags and positional arguments for more flexibility.

**Code Example**:
```go
package main

import (
	"flag"
	"fmt"
	"os"
)

func main() {
	greet := flag.Bool("greet", false, "enable greeting")
	flag.Parse()

	if *greet && len(flag.Args()) > 0 {
		name := flag.Args()[0]
		fmt.Printf("Hello, %s!
", name)
	} else {
		fmt.Println("Usage: go run main.go -greet <name>")
	}
}
```

**Run**:
```bash
go run main.go -greet Alice
# Output: Hello, Alice!
```

---

## **24.4. Managing Configurations**

Many CLI tools need configurations stored in files or environment variables. The `viper` library makes managing configurations simple.

### **1. Basic Configuration with `viper`**

**Installation**:
```bash
go get -u github.com/spf13/viper
```

**Code Example**:
```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

func main() {
	viper.SetConfigName("config")
	viper.SetConfigType("json")
	viper.AddConfigPath(".")

	err := viper.ReadInConfig()
	if err != nil {
		fmt.Println("Error reading config file:", err)
		return
	}

	name := viper.GetString("name")
	fmt.Printf("Hello, %s!
", name)
}
```

**Configuration File (config.json)**:
```json
{
	"name": "Alice"
}
```

**Run**:
```bash
go run main.go
# Output: Hello, Alice!
```

---

### **2. Environment Variables with `viper`**

**Code Example**:
```go
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

func main() {
	viper.SetEnvPrefix("APP")
	viper.BindEnv("name")

	name := viper.GetString("name")
	if name == "" {
		fmt.Println("Environment variable APP_NAME not set")
		return
	}

	fmt.Printf("Hello, %s!
", name)
}
```

**Run**:
```bash
export APP_NAME=Alice
go run main.go
# Output: Hello, Alice!
```

---

## **24.5. Distributing CLI Tools**

### **1. Building an Executable**

Compile your Go application into an executable:
```bash
go build -o greet
```

Run the executable:
```bash
./greet hello Alice
```

### **2. Packaging with `goreleaser`**

Use `goreleaser` to package your CLI tool for multiple platforms.

**Installation**:
```bash
go install github.com/goreleaser/goreleaser@latest
```

**Create a `.goreleaser.yml`** configuration file and run:
```bash
goreleaser release --snapshot --skip-publish
```

---


