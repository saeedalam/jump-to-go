# **Chapter 24: Building Professional CLI Applications with Go**

## **24.1. Introduction to CLI Development in Go**

Command-Line Interface (CLI) applications are essential tools for developers, DevOps engineers, and system administrators. Go has emerged as one of the premier languages for building CLI tools due to its compilation to single binaries, cross-platform support, strong standard library, and excellent performance characteristics.

In this chapter, we'll explore how to build professional-grade CLI applications by leveraging Go's strengths and applying advanced concepts from throughout this book.

### **Why Go Excels at CLI Applications**

Go offers several advantages that make it ideal for building command-line tools:

1. **Single Binary Deployment**: Go compiles to standalone executables with no external dependencies
2. **Cross-Platform Support**: Easily build for multiple operating systems from a single codebase
3. **Fast Execution**: Near-instant startup time and efficient runtime performance
4. **Strong Standard Library**: Built-in support for flags, file operations, and network functionality
5. **Concurrency Model**: Goroutines allow for efficient parallel operations
6. **Robust Error Handling**: Explicit error handling improves reliability

### **Types of CLI Applications**

We'll cover three main categories of CLI tools, each with increasing complexity:

1. **Single-Command Tools**: Simple utilities with a focused purpose

   - Example: A file converter, text processor, or data validator

2. **Multi-Command Applications**: Tools with subcommands for different operations

   - Example: A deployment tool with commands for different environments

3. **Interactive CLI Applications**: Tools that provide an interactive shell
   - Example: A database client or infrastructure management console

### **CLI Design Principles**

Before diving into code, let's establish key principles for designing high-quality CLI applications:

1. **User-Centric Design**: Create an intuitive interface that follows CLI conventions
2. **Progressive Disclosure**: Show basic options by default, advanced options when requested
3. **Consistent Feedback**: Provide clear feedback for both success and failure
4. **Documentation**: Include help text, examples, and error messages
5. **Testability**: Design for automated testing
6. **Performance**: Optimize for quick startup and efficient execution

### **What We'll Build**

Throughout this chapter, we'll incrementally build a comprehensive CLI tool called "DevOps Toolkit" that demonstrates advanced Go concepts:

- A multi-command application with nested subcommands
- Interactive prompts and colorful output
- Configuration management with environment variables and config files
- Concurrent operations for improved performance
- Graceful error handling and detailed logging
- Comprehensive testing approach
- Distribution and installation mechanisms

By the end of this chapter, you'll have the knowledge to build professional, maintainable CLI applications in Go that users will love to use.

Let's begin by exploring the fundamentals of CLI application structure and gradually progress to advanced patterns and techniques.

---

## **24.2. Fundamentals of CLI Applications in Go**

Go's standard library provides the essential building blocks for CLI applications. Before introducing third-party libraries, it's important to understand these fundamentals.

### **2.1 Working with Command-Line Arguments**

The `os.Args` slice contains all command-line arguments, with the program name at index 0.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// Print program name
	fmt.Printf("Program: %s\n", os.Args[0])

	// Check if arguments were provided
	if len(os.Args) < 2 {
		fmt.Println("No arguments provided")
		os.Exit(1)
	}

	// Process remaining arguments
	fmt.Println("Arguments:")
	for i, arg := range os.Args[1:] {
		fmt.Printf("  %d: %s\n", i+1, arg)
	}
}
```

While `os.Args` provides basic access to arguments, it lacks structure for complex applications.

### **2.2 Structured Command-Line Parsing with flag Package**

Go's standard `flag` package provides structured parsing for command-line flags.

```go
package main

import (
	"flag"
	"fmt"
	"os"
	"strings"
)

func main() {
	// Define flags with type, name, default value, and help text
	verbose := flag.Bool("verbose", false, "Enable verbose output")
	level := flag.Int("level", 1, "Set the processing level (1-5)")
	output := flag.String("output", "stdout", "Output destination")

	// Define short-form flag aliases
	flag.BoolVar(verbose, "v", false, "Enable verbose output (shorthand)")

	// Custom usage message
	flag.Usage = func() {
		fmt.Fprintf(os.Stderr, "Usage: %s [OPTIONS] [ARGUMENTS]\n\nOptions:\n", os.Args[0])
		flag.PrintDefaults()
		fmt.Fprintf(os.Stderr, "\nExample: %s -v -level 3 file.txt\n", os.Args[0])
	}

	// Parse command-line flags
	flag.Parse()

	// Access the remaining non-flag arguments
	args := flag.Args()

	// Use the flags in your application
	if *verbose {
		fmt.Println("Verbose mode enabled")
		fmt.Printf("Processing level: %d\n", *level)
		fmt.Printf("Output destination: %s\n", *output)
	}

	if len(args) == 0 {
		fmt.Println("No input files specified")
		flag.Usage()
		os.Exit(1)
	}

	fmt.Printf("Processing files: %s\n", strings.Join(args, ", "))
}
```

The `flag` package works well for simple applications but becomes unwieldy for complex command structures. Let's address this limitation.

### **2.3 Custom Command Structures**

For applications with multiple commands, we can build a simple command framework:

```go
package main

import (
	"flag"
	"fmt"
	"os"
)

// Command represents a subcommand with its own flags and behavior
type Command struct {
	Name        string
	Description string
	Run         func(args []string) error
	Flags       *flag.FlagSet
}

func main() {
	// Define available commands
	commands := map[string]*Command{
		"create": {
			Name:        "create",
			Description: "Create a new resource",
			Flags:       flag.NewFlagSet("create", flag.ExitOnError),
			Run:         runCreateCommand,
		},
		"delete": {
			Name:        "delete",
			Description: "Delete an existing resource",
			Flags:       flag.NewFlagSet("delete", flag.ExitOnError),
			Run:         runDeleteCommand,
		},
		"list": {
			Name:        "list",
			Description: "List available resources",
			Flags:       flag.NewFlagSet("list", flag.ExitOnError),
			Run:         runListCommand,
		},
	}

	// Add specific flags to each command
	force := commands["delete"].Flags.Bool("force", false, "Force deletion without confirmation")
	format := commands["list"].Flags.String("format", "table", "Output format (table, json, yaml)")

	// Show usage if no arguments
	if len(os.Args) < 2 {
		printUsage(commands)
		os.Exit(1)
	}

	// Get the command name (first argument)
	commandName := os.Args[1]

	// Look up the command
	cmd, exists := commands[commandName]
	if !exists {
		fmt.Fprintf(os.Stderr, "Unknown command: %s\n\n", commandName)
		printUsage(commands)
		os.Exit(1)
	}

	// Parse the remaining arguments
	cmd.Flags.Parse(os.Args[2:])

	// Run the command
	if err := cmd.Run(cmd.Flags.Args()); err != nil {
		fmt.Fprintf(os.Stderr, "Error: %s\n", err)
		os.Exit(1)
	}

	// Showcase the flags (would be used in the actual command functions)
	if commandName == "delete" && *force {
		fmt.Println("Force flag was set for delete command")
	}

	if commandName == "list" {
		fmt.Printf("List format: %s\n", *format)
	}
}

func printUsage(commands map[string]*Command) {
	fmt.Fprintf(os.Stderr, "Usage: %s COMMAND [OPTIONS] [ARGUMENTS]\n\nAvailable commands:\n", os.Args[0])
	for name, cmd := range commands {
		fmt.Fprintf(os.Stderr, "  %s: %s\n", name, cmd.Description)
	}
	fmt.Fprintf(os.Stderr, "\nRun '%s COMMAND --help' for command-specific help.\n", os.Args[0])
}

func runCreateCommand(args []string) error {
	fmt.Println("Running create command with args:", args)
	return nil
}

func runDeleteCommand(args []string) error {
	fmt.Println("Running delete command with args:", args)
	return nil
}

func runListCommand(args []string) error {
	fmt.Println("Running list command with args:", args)
	return nil
}
```

This pattern provides a foundation for multi-command applications but still requires significant boilerplate. This is why libraries like Cobra are popular for complex CLI tools.

## **24.3. Building Advanced CLI Applications with Cobra**

Cobra is the de facto standard for building advanced CLI applications in Go. Used by Kubernetes, Hugo, GitHub CLI, and many other popular tools, it provides a robust framework for creating complex command structures with minimal boilerplate.

### **3.1 Cobra Architecture**

Cobra follows a command-based architecture with three main components:

1. **Commands**: Represent actions that users can take
2. **Flags**: Arguments that modify command behavior
3. **Arguments**: Positional arguments passed to commands

Let's build a comprehensive example of a DevOps toolkit application with multiple nested commands:

```go
package main

import (
	"fmt"
	"os"
	"strings"
	"time"

	"github.com/spf13/cobra"
)

func main() {
	// Create the root command
	rootCmd := &cobra.Command{
		Use:   "devtool",
		Short: "A DevOps toolkit for managing infrastructure",
		Long: `DevTool is a comprehensive DevOps toolkit that provides
commands for managing servers, containers, and deployment workflows.
Complete documentation is available at https://example.com/devtool`,
		Version: "1.0.0",
		// Add a custom help template
		DisableAutoGenTag: true,
	}

	// Add global flags
	var verbose bool
	var configFile string
	rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false, "Enable verbose output")
	rootCmd.PersistentFlags().StringVar(&configFile, "config", "", "Config file (default is $HOME/.devtool.yaml)")

	// Create server command
	serverCmd := &cobra.Command{
		Use:   "server",
		Short: "Manage servers",
		Long:  `Commands for managing server infrastructure including provisioning, configuration, and monitoring.`,
	}

	// Create server list command
	serverListCmd := &cobra.Command{
		Use:   "list [flags]",
		Short: "List available servers",
		Long:  `List all servers registered in the system with their current status and details.`,
		Run: func(cmd *cobra.Command, args []string) {
			// Extract flags
			format, _ := cmd.Flags().GetString("format")
			region, _ := cmd.Flags().GetString("region")

			// Show what would happen in a real application
			fmt.Printf("Listing servers (format: %s, region: %s)\n", format, region)

			// Simulate fetching servers
			servers := []string{"web-01", "web-02", "db-01"}

			// Use verbose mode if enabled
			if verbose {
				fmt.Println("Verbose: Fetching server details including status, IP, and uptime")
				for _, server := range servers {
					fmt.Printf("Server: %s, Region: %s, Status: Running\n", server, region)
				}
			} else {
				for _, server := range servers {
					fmt.Println(server)
				}
			}
		},
		// Add command usage examples
		Example: `  # List all servers in table format
  devtool server list

  # List servers in JSON format
  devtool server list --format json

  # List servers in a specific region
  devtool server list --region us-west-2`,
	}

	// Add flags to server list command
	serverListCmd.Flags().StringP("format", "f", "table", "Output format (table, json, yaml)")
	serverListCmd.Flags().StringP("region", "r", "all", "Filter servers by region")

	// Create server create command
	serverCreateCmd := &cobra.Command{
		Use:   "create NAME",
		Short: "Create a new server",
		Args:  cobra.ExactArgs(1),
		RunE: func(cmd *cobra.Command, args []string) error {
			// Extract args and flags
			name := args[0]
			region, _ := cmd.Flags().GetString("region")
			size, _ := cmd.Flags().GetString("size")

			// Validate input
			if !strings.HasPrefix(name, "srv-") {
				return fmt.Errorf("server name must start with 'srv-'")
			}

			// Log action based on verbose flag
			if verbose {
				fmt.Printf("Creating server %s in region %s with size %s\n", name, region, size)
				fmt.Println("Verbose: Initializing server resources...")
				time.Sleep(1 * time.Second)
				fmt.Println("Verbose: Configuring networking...")
				time.Sleep(1 * time.Second)
				fmt.Println("Verbose: Starting services...")
				time.Sleep(1 * time.Second)
			}

			fmt.Printf("Server %s created successfully\n", name)
			return nil
		},
	}

	// Add flags to server create command
	serverCreateCmd.Flags().StringP("region", "r", "us-east-1", "Server region")
	serverCreateCmd.Flags().StringP("size", "s", "medium", "Server size (small, medium, large)")

	// Create container command
	containerCmd := &cobra.Command{
		Use:   "container",
		Short: "Manage containers",
		Long:  `Commands for managing containers including building, running, and monitoring.`,
		Aliases: []string{"cont", "c"},
	}

	// Create container run command
	containerRunCmd := &cobra.Command{
		Use:   "run IMAGE",
		Short: "Run a container",
		Args:  cobra.ExactArgs(1),
		Run: func(cmd *cobra.Command, args []string) {
			image := args[0]
			detach, _ := cmd.Flags().GetBool("detach")
			port, _ := cmd.Flags().GetInt("port")

			if detach {
				fmt.Printf("Running %s in detached mode on port %d\n", image, port)
			} else {
				fmt.Printf("Running %s in interactive mode on port %d\n", image, port)
				fmt.Println("Container output would appear here...")
			}
		},
	}

	// Add flags to container run command
	containerRunCmd.Flags().BoolP("detach", "d", false, "Run container in background")
	containerRunCmd.Flags().IntP("port", "p", 8080, "Port to expose")

	// Add subcommands to their parents
	serverCmd.AddCommand(serverListCmd, serverCreateCmd)
	containerCmd.AddCommand(containerRunCmd)
	rootCmd.AddCommand(serverCmd, containerCmd)

	// Execute the root command
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

### **3.2 Best Practices for Cobra Commands**

When building CLI applications with Cobra, follow these best practices:

1. **Command Structure**

   - Group related commands under a parent command
   - Use hierarchical commands for logical organization
   - Limit nesting to 2-3 levels for usability

2. **Command Naming**

   - Use simple, clear verbs for commands (create, delete, list)
   - Provide aliases for common abbreviations
   - Be consistent with naming across similar commands

3. **Documentation**

   - Include Short and Long descriptions for all commands
   - Add Examples showing common use cases
   - Use DisableAutoGenTag to manage documentation generation

4. **Error Handling**
   - Use RunE instead of Run to return errors properly
   - Return descriptive errors that guide users to a solution
   - Validate input early and provide clear error messages

### **3.3 Advanced Cobra Techniques**

Let's explore some advanced techniques for building sophisticated CLI tools with Cobra.

#### **Command Groups and Organization**

For complex applications, organizing commands into logical groups improves usability:

```go
// Create a command factory function for related commands
func createDeploymentCommands() *cobra.Command {
	deployCmd := &cobra.Command{
		Use:   "deploy",
		Short: "Deployment related commands",
	}

	// Add subcommands
	deployCmd.AddCommand(
		createDeployStartCommand(),
		createDeployStopCommand(),
		createDeployStatusCommand(),
	)

	return deployCmd
}

func createDeployStartCommand() *cobra.Command {
	return &cobra.Command{
		Use:   "start [environment]",
		Short: "Start a deployment",
		Run: func(cmd *cobra.Command, args []string) {
			// Implementation
		},
	}
}

// Then in main():
rootCmd.AddCommand(createDeploymentCommands())
```

#### **Custom Validation**

Implement custom validation for arguments and flags:

```go
// Custom validator for port range
func validatePortRange(cmd *cobra.Command, args []string) error {
	port, _ := cmd.Flags().GetInt("port")

	if port < 1024 || port > 65535 {
		return fmt.Errorf("port must be between 1024 and 65535")
	}
	return nil
}

// Add to command
cmd.PreRunE = validatePortRange
```

#### **Dynamic Command Completion**

Add shell completion support for dynamic values:

```go
// Add shell completion for region flag
serverCreateCmd.Flags().SetAnnotation("region", cobra.BashCompCustom, []string{"__devtool_get_regions"})

// Register a function to generate completions
rootCmd.BashCompletionFunction = `
__devtool_get_regions() {
    local regions=("us-east-1" "us-west-1" "eu-central-1" "ap-southeast-1")
    COMPREPLY=( $(compgen -W "${regions[*]}" -- ${cur}) )
}
`
```

## **24.4. User Experience Enhancements**

Creating a CLI tool with an excellent user experience differentiates professional applications from basic scripts. Let's explore techniques to enhance your CLI's usability.

### **4.1 Rich Terminal Output**

Modern CLI applications use color and formatting to improve readability and highlight important information.

```go
package main

import (
	"fmt"
	"os"
	"time"

	"github.com/fatih/color"
	"github.com/briandowns/spinner"
)

func main() {
	// Create colored output functions
	success := color.New(color.FgGreen, color.Bold).PrintlnFunc()
	warning := color.New(color.FgYellow).PrintlnFunc()
	error := color.New(color.FgRed, color.Bold).PrintlnFunc()
	info := color.New(color.FgCyan).PrintlnFunc()

	// Use colored output
	info("Starting application deployment...")

	// Show a spinner for long-running operations
	s := spinner.New(spinner.CharSets[14], 100*time.Millisecond)
	s.Prefix = "Deploying: "
	s.Start()

	// Simulate work
	time.Sleep(2 * time.Second)
	s.Stop()

	// Show progress for multi-step operations
	fmt.Println("Deployment progress:")
	steps := []string{
		"Building application",
		"Running tests",
		"Creating infrastructure",
		"Deploying application",
		"Configuring network",
	}

	for i, step := range steps {
		fmt.Printf("  [%d/%d] ", i+1, len(steps))

		if i < 3 {
			s = spinner.New(spinner.CharSets[11], 100*time.Millisecond)
			s.Suffix = " " + step
			s.Start()
			time.Sleep(1 * time.Second)
			s.Stop()
			success("✓ " + step)
		} else if i == 3 {
			warning("⚠ " + step + " (with warnings)")
		} else {
			error("✗ " + step + " (failed)")
			fmt.Println("    Detailed error message would appear here")
		}
	}

	// Use tables for structured data
	fmt.Println("\nDeployment Status:")
	fmt.Println("┌────────────────┬─────────────┬────────────┐")
	fmt.Println("│ Component      │ Status      │ Time       │")
	fmt.Println("├────────────────┼─────────────┼────────────┤")
	fmt.Println("│ API Server     │ Running     │ 12.3s      │")
	fmt.Println("│ Database       │ Running     │ 5.7s       │")
	fmt.Println("│ Frontend       │ Failed      │ 8.2s       │")
	fmt.Println("└────────────────┴─────────────┴────────────┘")

	// Summary with color coding
	fmt.Println("\nDeployment Result:")
	warning("Deployment completed with issues")
	fmt.Println("Run 'deploy logs frontend' to view error details")
}
```

For more sophisticated output:

1. Use the [tview](https://github.com/rivo/tview) or [termui](https://github.com/gizak/termui) packages for interactive terminal UIs
2. Try [lipgloss](https://github.com/charmbracelet/lipgloss) for sophisticated styling
3. Consider [bubbletea](https://github.com/charmbracelet/bubbletea) for terminal applications with complex interfaces

### **4.2 Interactive Input**

Enhance user interaction with prompts and suggestions using the [survey](https://github.com/AlecAivazis/survey) package:

```go
package main

import (
	"fmt"

	"github.com/AlecAivazis/survey/v2"
)

func main() {
	// Simple input prompt
	var name string
	namePrompt := &survey.Input{
		Message: "What is your name?",
		Default: "User",
	}
	survey.AskOne(namePrompt, &name)

	// Selection from a list
	var region string
	regionPrompt := &survey.Select{
		Message: "Choose a deployment region:",
		Options: []string{"us-east-1", "us-west-2", "eu-central-1", "ap-southeast-1"},
		Default: "us-east-1",
	}
	survey.AskOne(regionPrompt, &region)

	// Multi-select
	var services []string
	servicesPrompt := &survey.MultiSelect{
		Message: "Select services to deploy:",
		Options: []string{"API Gateway", "Lambda Functions", "DynamoDB", "S3 Buckets", "CloudFront"},
	}
	survey.AskOne(servicesPrompt, &services)

	// Confirmation
	var confirm bool
	confirmPrompt := &survey.Confirm{
		Message: fmt.Sprintf("Deploy %d services to %s?", len(services), region),
		Default: false,
	}
	survey.AskOne(confirmPrompt, &confirm)

	if confirm {
		fmt.Printf("Deploying to %s: %v\n", region, services)
	} else {
		fmt.Println("Deployment cancelled")
	}

	// Advanced: multi-question survey
	var answers struct {
		Environment string
		Replicas    int
		Debug       bool
		Tags        []string
	}

	questions := []*survey.Question{
		{
			Name: "environment",
			Prompt: &survey.Select{
				Message: "Deployment environment:",
				Options: []string{"dev", "staging", "production"},
				Default: "dev",
			},
		},
		{
			Name: "replicas",
			Prompt: &survey.Input{
				Message: "Number of replicas:",
				Default: "3",
			},
			Validate: survey.Required,
		},
		{
			Name: "debug",
			Prompt: &survey.Confirm{
				Message: "Enable debug mode?",
				Default: false,
			},
		},
		{
			Name: "tags",
			Prompt: &survey.MultiSelect{
				Message: "Select tags:",
				Options: []string{"frontend", "backend", "database", "monitoring"},
			},
		},
	}

	survey.Ask(questions, &answers)
	fmt.Printf("Configuration: %+v\n", answers)
}
```

### **4.3 Progress Feedback for Long Operations**

For operations that take time, provide appropriate feedback:

```go
package main

import (
	"fmt"
	"time"

	"github.com/schollz/progressbar/v3"
)

func main() {
	// Simple progress bar
	bar := progressbar.Default(100)
	for i := 0; i < 100; i++ {
		bar.Add(1)
		time.Sleep(50 * time.Millisecond)
	}

	// Advanced progress bar with custom options
	fmt.Println("\nDownloading large file:")
	bar = progressbar.NewOptions(1000,
		progressbar.OptionEnableColorCodes(true),
		progressbar.OptionShowBytes(true),
		progressbar.OptionSetWidth(50),
		progressbar.OptionSetDescription("[cyan]Downloading:[reset]"),
		progressbar.OptionSetTheme(progressbar.Theme{
			Saucer:        "[green]=[reset]",
			SaucerHead:    "[green]>[reset]",
			SaucerPadding: " ",
			BarStart:      "[",
			BarEnd:        "]",
		}),
	)

	// Simulate downloading chunks
	for i := 0; i < 1000; i++ {
		bar.Add(1)
		time.Sleep(5 * time.Millisecond)
	}

	fmt.Println("\nComplete!")
}
```

## **24.5. Configuration Management**

Professional CLI tools need robust configuration management. Let's explore advanced configuration techniques using Viper.

### **5.1 Multi-level Configuration with Viper**

Viper allows for configuration from multiple sources with clear precedence:

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"

	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

func main() {
	var cfgFile string

	rootCmd := &cobra.Command{
		Use:   "myapp",
		Short: "Application with advanced configuration",
		PersistentPreRunE: func(cmd *cobra.Command, args []string) error {
			// Initialize configuration
			return initConfig(cfgFile)
		},
	}

	// Add config flag
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.myapp.yaml)")

	// Sample command that uses config
	rootCmd.AddCommand(&cobra.Command{
		Use:   "show-config",
		Short: "Display current configuration",
		Run: func(cmd *cobra.Command, args []string) {
			fmt.Println("Current Configuration:")
			fmt.Printf("Database URL: %s\n", viper.GetString("database.url"))
			fmt.Printf("Database Username: %s\n", viper.GetString("database.username"))
			fmt.Printf("API Key: %s\n", viper.GetString("api.key"))
			fmt.Printf("Debug Mode: %v\n", viper.GetBool("debug"))
			fmt.Printf("Log Level: %s\n", viper.GetString("logging.level"))
			fmt.Printf("Server Port: %d\n", viper.GetInt("server.port"))
		},
	})

	rootCmd.Execute()
}

func initConfig(cfgFile string) error {
	// Configuration precedence (highest to lowest):
	// 1. Command line flags
	// 2. Environment variables
	// 3. Configuration file
	// 4. Default values

	// 1. Set defaults
	viper.SetDefault("database.url", "localhost:5432")
	viper.SetDefault("database.username", "user")
	viper.SetDefault("api.key", "")
	viper.SetDefault("debug", false)
	viper.SetDefault("logging.level", "info")
	viper.SetDefault("server.port", 8080)

	// 2. Configuration file settings
	if cfgFile != "" {
		// Use config file from the flag
		viper.SetConfigFile(cfgFile)
	} else {
		// Find home directory
		home, err := os.UserHomeDir()
		if err != nil {
			return err
		}

		// Search for config in home directory with name ".myapp" (without extension)
		viper.AddConfigPath(home)
		viper.AddConfigPath(".")
		viper.SetConfigName(".myapp")
	}

	// Set environment variable prefix
	viper.SetEnvPrefix("MYAPP")

	// Replace dots with underscores in env vars (e.g., database.url -> MYAPP_DATABASE_URL)
	viper.AutomaticEnv()
	viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))

	// Read the configuration file
	if err := viper.ReadInConfig(); err != nil {
		// Config file not found, ignore error if it's just not found
		if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
			return err
		}
	} else {
		fmt.Println("Using config file:", viper.ConfigFileUsed())
	}

	return nil
}
```

### **5.2 Working with Different Configuration Formats**

Viper supports multiple configuration formats:

```go
// Load YAML configuration
viper.SetConfigType("yaml")

// Example YAML (.myapp.yaml):
// database:
//   url: postgres://localhost:5432/mydb
//   username: admin
//   password: secret
// api:
//   key: abcd1234
// debug: true
// logging:
//   level: debug
// server:
//   port: 3000

// Load JSON configuration
viper.SetConfigType("json")

// Example JSON (.myapp.json):
// {
//   "database": {
//     "url": "postgres://localhost:5432/mydb",
//     "username": "admin",
//     "password": "secret"
//   },
//   "api": {
//     "key": "abcd1234"
//   },
//   "debug": true,
//   "logging": {
//     "level": "debug"
//   },
//   "server": {
//     "port": 3000
//   }
// }

// Load TOML configuration
viper.SetConfigType("toml")

// Example TOML (.myapp.toml):
// [database]
// url = "postgres://localhost:5432/mydb"
// username = "admin"
// password = "secret"
//
// [api]
// key = "abcd1234"
//
// debug = true
//
// [logging]
// level = "debug"
//
// [server]
// port = 3000
```

### **5.3 Dynamic Configuration Reloading**

For long-running applications, implement configuration reloading:

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"syscall"
	"time"

	"github.com/fsnotify/fsnotify"
	"github.com/spf13/viper"
)

func main() {
	// Initialize configuration
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")

	if err := viper.ReadInConfig(); err != nil {
		log.Fatalf("Error reading config file: %s", err)
	}

	// Watch for configuration changes
	viper.WatchConfig()
	viper.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("Config file changed:", e.Name)
		// Update any runtime configuration here
		// For example, change log levels, connection parameters, etc.
		updateAppConfig()
	})

	// Start application with initial configuration
	updateAppConfig()

	// Run a simple web server as an example
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Server running on port %d\n", viper.GetInt("server.port"))
		fmt.Fprintf(w, "Debug mode: %v\n", viper.GetBool("debug"))
		fmt.Fprintf(w, "Log level: %s\n", viper.GetString("logging.level"))
	})

	log.Printf("Starting server on port %d\n", viper.GetInt("server.port"))
	log.Fatal(http.ListenAndServe(fmt.Sprintf(":%d", viper.GetInt("server.port")), nil))
}

func updateAppConfig() {
	// Update logger configuration
	if viper.GetBool("debug") {
		log.SetFlags(log.Ldate | log.Ltime | log.Lmicroseconds | log.Llongfile)
		log.Println("Debug mode enabled")
	} else {
		log.SetFlags(log.Ldate | log.Ltime)
		log.Println("Debug mode disabled")
	}

	// Other runtime configuration updates...
}
```

### **5.4 Secure Configuration**

For sensitive information, implement secure configuration handling:

```go
package main

import (
	"fmt"
	"os"

	"github.com/spf13/viper"
	"golang.org/x/crypto/nacl/secretbox"
	"encoding/base64"
)

// Simplified secure configuration handling
func main() {
	// Setup viper
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")
	viper.AddConfigPath(".")

	// Read normal configuration
	if err := viper.ReadInConfig(); err != nil {
		fmt.Printf("Error reading config file: %s\n", err)
		os.Exit(1)
	}

	// Get database credentials - first try environment variables
	dbUser := os.Getenv("DB_USER")
	dbPass := os.Getenv("DB_PASS")

	// If not in environment, try encrypted config
	if dbUser == "" || dbPass == "" {
		// Get encryption key from environment (never store in config)
		encKey := os.Getenv("CONFIG_ENCRYPTION_KEY")
		if encKey == "" {
			fmt.Println("Missing CONFIG_ENCRYPTION_KEY environment variable")
			os.Exit(1)
		}

		// Get encrypted credentials from config
		encDbConfig := viper.GetString("database.encrypted_credentials")
		if encDbConfig == "" {
			fmt.Println("No encrypted database credentials found in config")
			os.Exit(1)
		}

		// Decrypt credentials
		credentials, err := decryptCredentials(encDbConfig, encKey)
	if err != nil {
			fmt.Printf("Failed to decrypt credentials: %s\n", err)
			os.Exit(1)
		}

		dbUser = credentials["username"]
		dbPass = credentials["password"]
	}

	// Use the credentials
	fmt.Printf("Using database credentials - User: %s, Password: %s\n", dbUser, maskPassword(dbPass))
}

// Simplified decryption function
func decryptCredentials(encryptedData, key string) (map[string]string, error) {
	// In a real implementation, this would use secretbox or another proper
	// encryption library to decrypt the data securely

	// Simulated decryption for the example
	return map[string]string{
		"username": "secureuser",
		"password": "securepassword",
	}, nil
}

// Mask password for display
func maskPassword(password string) string {
	if len(password) <= 4 {
		return "****"
	}
	return password[:2] + "****" + password[len(password)-2:]
}
```

## **24.6. Testing CLI Applications**

Testing is a crucial part of CLI application development. Let's explore different testing approaches and tools.

### **6.1 Unit Testing**

Unit tests are used to test individual functions or methods.

```go
package main

import (
	"testing"
)

func TestRunCreateCommand(t *testing.T) {
	// Test cases
	testCases := []struct {
		name     string
		args     []string
		expected error
	}{
		{"valid_args", []string{"create", "srv-123", "-r", "us-east-1", "-s", "medium"}, nil},
		{"invalid_name", []string{"create", "srv-123", "-r", "us-east-1", "-s", "medium"}, fmt.Errorf("server name must start with 'srv-'")},
		{"missing_args", []string{"create"}, fmt.Errorf("server name must be provided")},
	}

	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			err := runCreateCommand(tc.args)
			if err != tc.expected {
				t.Errorf("expected error: %v, got: %v", tc.expected, err)
			}
		})
	}
}
```

### **6.2 Integration Testing**

Integration tests are used to test the interaction between different components of the application.

```go
package main

import (
	"testing"
)

func TestRunListCommand(t *testing.T) {
	// Test cases
	testCases := []struct {
		name     string
		args     []string
		expected string
	}{
		{"valid_args", []string{"list", "--format", "json"}, "Listing servers (format: json, region: all)\n"},
		{"invalid_format", []string{"list", "--format", "invalid"}, "Error: invalid format: invalid\n"},
	}

	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			output := runListCommand(tc.args)
			if output != tc.expected {
				t.Errorf("expected output: %q, got: %q", tc.expected, output)
			}
		})
	}
}
```

### **6.3 Acceptance Testing**

Acceptance tests are used to test the end-to-end functionality of the application.

```go
package main

import (
	"testing"
)

func TestRunDevTool(t *testing.T) {
	// Test cases
	testCases := []struct {
		name     string
		args     []string
		expected string
	}{
		{"valid_args", []string{"devtool", "server", "list"}, "Listing servers (format: table, region: all)\n"},
		{"invalid_command", []string{"devtool", "invalid"}, "Error: unknown command: invalid\n"},
	}

	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			output := runDevTool(tc.args)
			if output != tc.expected {
				t.Errorf("expected output: %q, got: %q", tc.expected, output)
			}
		})
	}
}
```

---

## **24.7. Distributing CLI Applications**

Professional CLI tools require proper packaging and distribution mechanisms. Let's explore how to distribute your Go CLI applications.

### **7.1 Cross-Platform Compilation**

One of Go's strengths is its ability to cross-compile for different platforms from a single development environment:

```go
// Build for multiple platforms
// For Linux
GOOS=linux GOARCH=amd64 go build -o bin/myapp-linux-amd64

// For macOS
GOOS=darwin GOARCH=amd64 go build -o bin/myapp-darwin-amd64

// For Windows
GOOS=windows GOARCH=amd64 go build -o bin/myapp-windows-amd64.exe
```

For more systematic cross-compilation, create a build script:

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"path/filepath"
)

type BuildTarget struct {
	OS   string
	Arch string
}

func main() {
	// Define targets
	targets := []BuildTarget{
		{"linux", "amd64"},
		{"linux", "arm64"},
		{"darwin", "amd64"},
		{"darwin", "arm64"},
		{"windows", "amd64"},
	}

	appName := "devtool"
	outputDir := "dist"

	// Create output directory
	os.MkdirAll(outputDir, 0755)

	// Build for each target
	for _, target := range targets {
		fmt.Printf("Building for %s/%s...\n", target.OS, target.Arch)

		outputFile := filepath.Join(outputDir, fmt.Sprintf("%s-%s-%s", appName, target.OS, target.Arch))
		if target.OS == "windows" {
			outputFile += ".exe"
		}

		cmd := exec.Command("go", "build", "-o", outputFile, "./cmd/devtool")
		cmd.Env = append(os.Environ(),
			fmt.Sprintf("GOOS=%s", target.OS),
			fmt.Sprintf("GOARCH=%s", target.Arch),
		)

		output, err := cmd.CombinedOutput()
		if err != nil {
			fmt.Printf("Error building for %s/%s: %v\n%s\n", target.OS, target.Arch, err, output)
			continue
		}

		fmt.Printf("Built %s\n", outputFile)
	}
}
```

### **7.2 Using GoReleaser**

[GoReleaser](https://goreleaser.com/) automates the building, packaging, and releasing of Go applications:

1. **Installation**:

```bash
go install github.com/goreleaser/goreleaser@latest
```

2. **Configuration** (`.goreleaser.yml`):

```yaml
project_name: devtool

builds:
  - env:
      - CGO_ENABLED=0
    goos:
      - linux
      - darwin
      - windows
    goarch:
      - amd64
      - arm64
    main: ./cmd/devtool
    ldflags:
      - -s -w -X main.version={{.Version}} -X main.commit={{.Commit}} -X main.date={{.Date}}

archives:
  - format_overrides:
      - goos: windows
        format: zip
    replacements:
      darwin: macos
    files:
      - README.md
      - LICENSE
      - completions/*

brews:
  - name: devtool
    tap:
      owner: yourorg
      name: homebrew-tap
    commit_author:
      name: goreleaserbot
      email: bot@goreleaser.com
    homepage: https://github.com/yourorg/devtool
    description: A DevOps toolkit for managing infrastructure
    license: MIT

nfpms:
  - package_name: devtool
    homepage: https://github.com/yourorg/devtool
    description: A DevOps toolkit for managing infrastructure
    maintainer: Your Name <your.email@example.com>
    license: MIT
    formats:
      - deb
      - rpm
    dependencies:
      - git

snapcrafts:
  - name: devtool
    summary: A DevOps toolkit for managing infrastructure
    description: |
      DevTool is a comprehensive DevOps toolkit that provides
      commands for managing servers, containers, and deployment workflows.
    grade: stable
    confinement: strict
    publish: true
```

3. **Run GoReleaser**:

```bash
# For testing
goreleaser release --snapshot --rm-dist

# For actual release
goreleaser release
```

### **7.3 Shell Completions**

Professional CLI tools provide shell completions to improve usability:

```go
package main

import (
	"os"

	"github.com/spf13/cobra"
)

func main() {
	// Create root command
	rootCmd := &cobra.Command{
		Use:   "devtool",
		Short: "A DevOps toolkit",
	}

	// Add completion command
	completionCmd := &cobra.Command{
		Use:   "completion [bash|zsh|fish|powershell]",
		Short: "Generate completion script",
		Long: `To load completions:

Bash:
  $ source <(devtool completion bash)

Zsh:
  # If shell completion is not already enabled:
  $ echo "autoload -U compinit; compinit" >> ~/.zshrc

  $ devtool completion zsh > "${fpath[1]}/_devtool"

Fish:
  $ devtool completion fish > ~/.config/fish/completions/devtool.fish

PowerShell:
  PS> devtool completion powershell | Out-String | Invoke-Expression
`,
		DisableFlagsInUseLine: true,
		ValidArgs:             []string{"bash", "zsh", "fish", "powershell"},
		Args:                  cobra.ExactValidArgs(1),
		Run: func(cmd *cobra.Command, args []string) {
			switch args[0] {
			case "bash":
				cmd.Root().GenBashCompletion(os.Stdout)
			case "zsh":
				cmd.Root().GenZshCompletion(os.Stdout)
			case "fish":
				cmd.Root().GenFishCompletion(os.Stdout, true)
			case "powershell":
				cmd.Root().GenPowerShellCompletionWithDesc(os.Stdout)
			}
		},
	}

	rootCmd.AddCommand(completionCmd)

	// Execute
	if err := rootCmd.Execute(); err != nil {
		os.Exit(1)
	}
}
```

### **7.4 Installation Scripts**

Provide easy installation scripts for your users:

```bash
#!/bin/bash
# install.sh - DevTool installer

set -e

# Determine OS and architecture
OS="$(uname -s | tr '[:upper:]' '[:lower:]')"
ARCH="$(uname -m)"

# Map architecture
case "${ARCH}" in
    x86_64) ARCH="amd64" ;;
    aarch64) ARCH="arm64" ;;
    *) echo "Unsupported architecture: ${ARCH}"; exit 1 ;;
esac

# Set install directory
INSTALL_DIR="/usr/local/bin"
if [ ! -d "$INSTALL_DIR" ] || [ ! -w "$INSTALL_DIR" ]; then
    INSTALL_DIR="$HOME/.local/bin"
    mkdir -p "$INSTALL_DIR"
fi

# Download URL
VERSION="1.0.0"
DOWNLOAD_URL="https://github.com/yourorg/devtool/releases/download/v${VERSION}/devtool-${OS}-${ARCH}"
if [ "$OS" = "windows" ]; then
    DOWNLOAD_URL="${DOWNLOAD_URL}.exe"
fi

echo "Downloading DevTool ${VERSION} for ${OS}/${ARCH}..."
curl -L "$DOWNLOAD_URL" -o "${INSTALL_DIR}/devtool"
chmod +x "${INSTALL_DIR}/devtool"

echo "DevTool installed to ${INSTALL_DIR}/devtool"

# Check if directory is in PATH
if [[ ":$PATH:" != *":${INSTALL_DIR}:"* ]]; then
    echo "Warning: ${INSTALL_DIR} is not in your PATH"
    echo "Add it to your PATH by adding this line to your shell profile:"
    echo "  export PATH=\"\$PATH:${INSTALL_DIR}\""
fi
```

## **24.8. Building a Complete CLI Project**

Let's consolidate our learning by designing a complete CLI application structure.

### **8.1 Project Structure**

A well-organized CLI project follows this structure:

```
devtool/
├── cmd/
│   └── devtool/
│       └── main.go         # Entry point
├── internal/               # Private application code
│   ├── commands/           # Command implementations
│   │   ├── root.go
│   │   ├── server.go
│   │   └── container.go
│   ├── config/             # Configuration handling
│   │   └── config.go
│   └── utils/              # Utility functions
│       └── display.go
├── pkg/                    # Public libraries
│   └── api/                # API client
│       └── client.go
├── scripts/                # Build and release scripts
│   ├── build.sh
│   └── install.sh
├── completions/            # Shell completions
│   ├── devtool.bash
│   └── devtool.zsh
├── dist/                   # Build artifacts
├── .goreleaser.yml         # GoReleaser config
├── go.mod
├── go.sum
├── LICENSE
└── README.md
```

### **8.2 Main Application Entry Point**

The main.go file serves as the entry point:

```go
// cmd/devtool/main.go
package main

import (
	"fmt"
	"os"

	"github.com/yourorg/devtool/internal/commands"
	"github.com/yourorg/devtool/internal/config"
)

// Version information - set during build
var (
	version = "dev"
	commit  = "none"
	date    = "unknown"
)

func main() {
	// Initialize configuration
	if err := config.Init(); err != nil {
		fmt.Fprintf(os.Stderr, "Error initializing configuration: %v\n", err)
		os.Exit(1)
	}

	// Create root command with version info
	rootCmd := commands.NewRootCmd(version, commit, date)

	// Execute
	if err := rootCmd.Execute(); err != nil {
		os.Exit(1)
	}
}
```

### **8.3 Command Implementation**

Commands are organized in separate files:

```go
// internal/commands/root.go
package commands

import (
	"github.com/spf13/cobra"
)

func NewRootCmd(version, commit, date string) *cobra.Command {
	rootCmd := &cobra.Command{
		Use:     "devtool",
		Short:   "A DevOps toolkit for managing infrastructure",
		Version: formatVersion(version, commit, date),
	}

	// Add global flags
	rootCmd.PersistentFlags().BoolP("verbose", "v", false, "Enable verbose output")

	// Add subcommands
	rootCmd.AddCommand(
		NewServerCmd(),
		NewContainerCmd(),
		NewCompletionCmd(),
	)

	return rootCmd
}

func formatVersion(version, commit, date string) string {
	return version + "\ncommit: " + commit + "\nbuilt at: " + date
}

// internal/commands/server.go
package commands

import (
	"github.com/spf13/cobra"
)

func NewServerCmd() *cobra.Command {
	serverCmd := &cobra.Command{
		Use:   "server",
		Short: "Manage servers",
	}

	// Add subcommands
	serverCmd.AddCommand(
		NewServerListCmd(),
		NewServerCreateCmd(),
	)

	return serverCmd
}

func NewServerListCmd() *cobra.Command {
	cmd := &cobra.Command{
		Use:   "list",
		Short: "List servers",
		RunE:  runServerListCmd,
	}

	cmd.Flags().StringP("format", "f", "table", "Output format (table, json, yaml)")
	cmd.Flags().StringP("region", "r", "all", "Filter by region")

	return cmd
}

func runServerListCmd(cmd *cobra.Command, args []string) error {
	// Implementation here
	return nil
}
```

### **8.4 Configuration Handling**

Centralized configuration management:

```go
// internal/config/config.go
package config

import (
	"fmt"
	"os"
	"path/filepath"
	"strings"

	"github.com/spf13/viper"
)

// Config stores the application configuration
type Config struct {
	Debug    bool
	LogLevel string
	Server   ServerConfig
	API      APIConfig
}

type ServerConfig struct {
	URL      string
	Username string
	Password string
}

type APIConfig struct {
	Key       string
	Endpoint  string
	Timeout   int
	RetryMax  int
}

// Global config instance
var AppConfig Config

// Init initializes the configuration
func Init() error {
	// Set defaults
	viper.SetDefault("debug", false)
	viper.SetDefault("loglevel", "info")
	viper.SetDefault("server.url", "https://api.example.com")
	viper.SetDefault("api.timeout", 30)
	viper.SetDefault("api.retrymax", 3)

	// Configuration file
	viper.SetConfigName("config")
	viper.SetConfigType("yaml")

	// Search paths
	home, err := os.UserHomeDir()
	if err == nil {
		viper.AddConfigPath(filepath.Join(home, ".devtool"))
	}
	viper.AddConfigPath(".")

	// Environment variables
	viper.SetEnvPrefix("DEVTOOL")
	viper.SetEnvKeyReplacer(strings.NewReplacer(".", "_"))
	viper.AutomaticEnv()

	// Read configuration
	if err := viper.ReadInConfig(); err != nil {
		if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
			return fmt.Errorf("failed to read config file: %w", err)
		}
	}

	// Unmarshal config
	if err := viper.Unmarshal(&AppConfig); err != nil {
		return fmt.Errorf("failed to unmarshal config: %w", err)
	}

	return nil
}

// GetConfig returns the current configuration
func GetConfig() Config {
	return AppConfig
}
```

### **8.5 Utility Functions**

Reusable utility functions:

```go
// internal/utils/display.go
package utils

import (
	"fmt"
	"io"
	"os"
	"strings"
	"text/tabwriter"

	"github.com/fatih/color"
)

var (
	Success = color.New(color.FgGreen, color.Bold).SprintfFunc()
	Warning = color.New(color.FgYellow).SprintfFunc()
	Error   = color.New(color.FgRed, color.Bold).SprintfFunc()
	Info    = color.New(color.FgCyan).SprintfFunc()
)

// FormatTable formats data as a table
func FormatTable(writer io.Writer, headers []string, rows [][]string) {
	w := tabwriter.NewWriter(writer, 0, 0, 2, ' ', 0)

	// Print headers
	fmt.Fprintln(w, strings.Join(headers, "\t"))

	// Print separator
	separators := make([]string, len(headers))
	for i := range separators {
		separators[i] = strings.Repeat("-", len(headers[i]))
	}
	fmt.Fprintln(w, strings.Join(separators, "\t"))

	// Print rows
	for _, row := range rows {
		fmt.Fprintln(w, strings.Join(row, "\t"))
	}

	w.Flush()
}

// PrintError prints an error message to stderr
func PrintError(format string, a ...interface{}) {
	fmt.Fprintf(os.Stderr, Error(format, a...)+"\n")
}

// PrintSuccess prints a success message
func PrintSuccess(format string, a ...interface{}) {
	fmt.Printf(Success(format, a...)+"\n")
}
```

## **24.9. Summary and Best Practices**

### **9.1 Best Practices for CLI Applications**

1. **Design Principles**

   - Follow the Unix philosophy: do one thing and do it well
   - Use consistent naming and command structure
   - Provide sensible defaults but allow customization
   - Follow the principle of least surprise

2. **User Experience**

   - Provide clear, concise help text and examples
   - Use colors and formatting to enhance readability
   - Show progress for long-running operations
   - Support both interactive and non-interactive modes

3. **Code Organization**

   - Separate command logic from business logic
   - Organize commands hierarchically
   - Use interfaces for testability
   - Handle errors gracefully with useful messages

4. **Performance**

   - Optimize startup time for frequently used commands
   - Use concurrency for I/O-bound operations
   - Profile and benchmark critical paths
   - Consider memory usage for large data sets

5. **Distribution**
   - Support multiple platforms and architectures
   - Provide easy installation methods
   - Include documentation and examples
   - Automate the release process

### **9.2 Learning From Successful CLI Tools**

Study these popular Go CLI tools for inspiration:

1. **Kubernetes CLI (kubectl)**: Complex but consistent command structure
2. **HashiCorp Tools (terraform, vault)**: Excellent documentation and UX
3. **GitHub CLI (gh)**: Intuitive interface and integration patterns
4. **Hugo**: Fast performance and powerful configuration

### **9.3 Further Learning Resources**

1. **Books**

   - "Command-Line Rust" by Ken Youens-Clark (principles apply to Go)
   - "Powerful Command-Line Applications in Go" by Ricardo Gerardi

2. **Libraries**

   - [spf13/cobra](https://github.com/spf13/cobra): Command-line framework
   - [spf13/viper](https://github.com/spf13/viper): Configuration management
   - [charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea): Terminal UI framework
   - [AlecAivazis/survey](https://github.com/AlecAivazis/survey): Interactive prompts

3. **Tools**
   - [goreleaser](https://goreleaser.com/): Automate releases
   - [golangci-lint](https://golangci-lint.run/): Linting for Go projects

## **24.10. Exercises**

1. **Basic CLI Tool**: Create a simple file manipulation tool with commands for listing, copying, and moving files.

2. **Interactive CLI**: Build an interactive tool for managing a to-do list with storage in a local JSON file.

3. **API Client**: Develop a CLI client for a RESTful API (GitHub, weather service, etc.) with caching and configuration.

4. **DevOps Tool**: Create a deployment tool that can manage configurations across environments.

5. **Advanced Project**: Build a comprehensive CLI application that incorporates all the concepts from this chapter, including a plugin system for extensibility.

By completing these exercises, you'll gain practical experience with the concepts and patterns introduced in this chapter, preparing you for building professional-grade CLI applications in Go.
