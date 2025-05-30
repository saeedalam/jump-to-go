# **Chapter 33: Go for AI and LLMs**

## **33.1 Introduction to AI Integration with Go**

The field of artificial intelligence (AI) has undergone a dramatic transformation with the rise of large language models (LLMs) like GPT-4, Claude, and Gemini. As AI capabilities become more accessible through APIs, Go developers are uniquely positioned to build applications that leverage these powerful models.

### Go's Role in the AI Ecosystem

Go excels as a language for building the infrastructure around AI systems rather than for implementing the core machine learning algorithms themselves. While languages like Python dominate the training and development of AI models, Go shines in:

1. **API Services and Middleware**: Creating robust services that interact with AI models
2. **High-Performance Backends**: Building scalable applications that incorporate AI capabilities
3. **Concurrent Processing**: Handling multiple AI requests efficiently
4. **Production Systems**: Deploying reliable, maintainable AI-powered applications

### Advantages of Go for AI Applications

Go offers several benefits when building applications that integrate with AI services:

- **Performance**: Go's efficiency makes it ideal for handling high-throughput AI service requests
- **Concurrency Model**: Goroutines and channels simplify managing multiple concurrent AI operations
- **Type Safety**: Strong typing reduces runtime errors when handling complex AI responses
- **Deployment Simplicity**: Single binary deployment simplifies operations for AI applications
- **Standard Library**: Rich standard library supports networking, JSON processing, and error handling

### Integration Approaches

There are several ways to integrate AI capabilities into Go applications:

1. **API-Based Integration**: Consuming AI services like OpenAI, Anthropic, or Vertex AI
2. **Local Model Inference**: Running optimized models directly in Go applications
3. **Hybrid Approaches**: Combining API calls with local processing for certain tasks
4. **Vector Database Integration**: Building retrieval-augmented generation (RAG) systems

Let's look at a simple example of how a Go application might be structured to integrate with an AI service:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"
)

// AIClient represents a generic interface for AI service integration
type AIClient interface {
	GenerateText(ctx context.Context, prompt string, options GenerationOptions) (*GenerationResult, error)
}

// GenerationOptions contains parameters for text generation
type GenerationOptions struct {
	MaxTokens     int
	Temperature   float64
	TopP          float64
	ResponseFormat string
}

// GenerationResult contains the response from the AI service
type GenerationResult struct {
	Text       string
	TokensUsed int
	ModelID    string
}

// Application demonstrates a simple Go application with AI integration
func main() {
	// Initialize the AI client (implementation details would vary)
	aiClient := initializeAIClient()

	// Create a context with timeout
	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
	defer cancel()

	// Example prompt
	prompt := "Explain the benefits of using Go for backend development"

	// Generation options
	options := GenerationOptions{
		MaxTokens:     1000,
		Temperature:   0.7,
		TopP:          1.0,
		ResponseFormat: "text",
	}

	// Generate text
	result, err := aiClient.GenerateText(ctx, prompt, options)
	if err != nil {
		log.Fatalf("Error generating text: %v", err)
	}

	// Use the generated text in your application
	fmt.Printf("Generated text (%d tokens used):\n%s\n", result.TokensUsed, result.Text)
}

// Placeholder for client initialization
func initializeAIClient() AIClient {
	// In a real implementation, this would initialize the specific AI client
	// such as OpenAI, Anthropic Claude, etc.
	return nil
}
```

This example illustrates a typical pattern for AI integration in Go: defining clean interfaces, using context for timeout handling, and structuring the application to work with external AI services.

## **33.2 Working with LLM APIs in Go**

Large Language Models are typically accessed through REST APIs provided by companies like OpenAI, Anthropic, Google, and others. Go's standard library and ecosystem make it straightforward to interact with these services.

### OpenAI API Integration

The OpenAI API is one of the most widely used for accessing models like GPT-4. Here's how to integrate it into a Go application:

```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

// OpenAIClient provides methods to interact with the OpenAI API
type OpenAIClient struct {
	apiKey     string
	httpClient *http.Client
	baseURL    string
}

// CompletionRequest represents a request to the OpenAI completions endpoint
type CompletionRequest struct {
	Model       string    `json:"model"`
	Messages    []Message `json:"messages"`
	Temperature float64   `json:"temperature,omitempty"`
	MaxTokens   int       `json:"max_tokens,omitempty"`
}

// Message represents a message in the chat completion API
type Message struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

// CompletionResponse represents a response from the OpenAI completions endpoint
type CompletionResponse struct {
	ID      string `json:"id"`
	Object  string `json:"object"`
	Created int    `json:"created"`
	Choices []struct {
		Index        int `json:"index"`
		Message struct {
			Role    string `json:"role"`
			Content string `json:"content"`
		} `json:"message"`
		FinishReason string `json:"finish_reason"`
	} `json:"choices"`
	Usage struct {
		PromptTokens     int `json:"prompt_tokens"`
		CompletionTokens int `json:"completion_tokens"`
		TotalTokens      int `json:"total_tokens"`
	} `json:"usage"`
}

// NewOpenAIClient creates a new OpenAI client
func NewOpenAIClient(apiKey string) *OpenAIClient {
	return &OpenAIClient{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 60 * time.Second,
		},
		baseURL: "https://api.openai.com/v1",
	}
}

// CreateChatCompletion sends a completion request to the OpenAI API
func (c *OpenAIClient) CreateChatCompletion(ctx context.Context, req CompletionRequest) (*CompletionResponse, error) {
	jsonData, err := json.Marshal(req)
	if err != nil {
		return nil, fmt.Errorf("marshaling request: %w", err)
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		http.MethodPost,
		fmt.Sprintf("%s/chat/completions", c.baseURL),
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return nil, fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("Authorization", fmt.Sprintf("Bearer %s", c.apiKey))

	resp, err := c.httpClient.Do(httpReq)
	if err != nil {
		return nil, fmt.Errorf("executing request: %w", err)
	}
	defer resp.Body.Close()

	// Handle non-200 responses
	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return nil, fmt.Errorf("API error (status %d): %s", resp.StatusCode, string(body))
	}

	// Parse the response
	var result CompletionResponse
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		return nil, fmt.Errorf("decoding response: %w", err)
	}

	return &result, nil
}

// Main function demonstrating OpenAI API usage
func main() {
	apiKey := os.Getenv("OPENAI_API_KEY")
	if apiKey == "" {
		fmt.Println("OPENAI_API_KEY environment variable is required")
		os.Exit(1)
	}

	client := NewOpenAIClient(apiKey)

	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
	defer cancel()

	// Create a request
	req := CompletionRequest{
		Model: "gpt-4",
		Messages: []Message{
			{
				Role:    "system",
				Content: "You are a helpful assistant specialized in Go programming.",
			},
			{
				Role:    "user",
				Content: "What are the best practices for error handling in Go?",
			},
		},
		Temperature: 0.7,
		MaxTokens:   500,
	}

	// Send the request
	resp, err := client.CreateChatCompletion(ctx, req)
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		os.Exit(1)
	}

	// Print the response
	if len(resp.Choices) > 0 {
		fmt.Println("Response:", resp.Choices[0].Message.Content)
		fmt.Printf("Tokens used: %d\n", resp.Usage.TotalTokens)
	} else {
		fmt.Println("No response generated")
	}
}
```

### **33.2.1 Building a Production-Ready AI Client**

For production applications, you'll want a more robust OpenAI client with features like:

1. **Retry Handling**: Automatically retry on transient errors
2. **Rate Limiting**: Respect API rate limits
3. **Streaming Support**: Handle streaming responses
4. **Logging**: Log requests and responses for debugging
5. **Fallback Mechanisms**: Switch to alternate models or services when needed

Here's a more complete implementation:

```go
package openai

import (
	"bytes"
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"io"
	"log"
	"net/http"
	"time"

	"github.com/cenkalti/backoff/v4"
)

// Client provides methods to interact with the OpenAI API
type Client struct {
	apiKey     string
	httpClient *http.Client
	baseURL    string
	logger     *log.Logger
	retryMax   int
}

// ClientOption is a function that modifies a Client
type ClientOption func(*Client)

// WithBaseURL sets a custom base URL for the API
func WithBaseURL(url string) ClientOption {
	return func(c *Client) {
		c.baseURL = url
	}
}

// WithLogger sets a custom logger
func WithLogger(logger *log.Logger) ClientOption {
	return func(c *Client) {
		c.logger = logger
	}
}

// WithHTTPClient sets a custom HTTP client
func WithHTTPClient(client *http.Client) ClientOption {
	return func(c *Client) {
		c.httpClient = client
	}
}

// WithMaxRetries sets the maximum number of retries
func WithMaxRetries(retries int) ClientOption {
	return func(c *Client) {
		c.retryMax = retries
	}
}

// NewClient creates a new OpenAI client with the given options
func NewClient(apiKey string, opts ...ClientOption) *Client {
	client := &Client{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 60 * time.Second,
		},
		baseURL:   "https://api.openai.com/v1",
		logger:    log.New(io.Discard, "", 0),
		retryMax:  3,
	}

	// Apply options
	for _, opt := range opts {
		opt(client)
	}

	return client
}

// CompletionRequest represents a request to the OpenAI completions endpoint
type CompletionRequest struct {
	Model       string    `json:"model"`
	Messages    []Message `json:"messages"`
	Temperature float64   `json:"temperature,omitempty"`
	MaxTokens   int       `json:"max_tokens,omitempty"`
	Stream      bool      `json:"stream,omitempty"`
}

// Message represents a message in the chat completion API
type Message struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

// CompletionResponse represents a response from the OpenAI completions endpoint
type CompletionResponse struct {
	ID      string `json:"id"`
	Object  string `json:"object"`
	Created int    `json:"created"`
	Choices []struct {
		Index        int `json:"index"`
		Message struct {
			Role    string `json:"role"`
			Content string `json:"content"`
		} `json:"message"`
		FinishReason string `json:"finish_reason"`
	} `json:"choices"`
	Usage struct {
		PromptTokens     int `json:"prompt_tokens"`
		CompletionTokens int `json:"completion_tokens"`
		TotalTokens      int `json:"total_tokens"`
	} `json:"usage"`
}

// APIError represents an error response from the API
type APIError struct {
	StatusCode int
	Type       string `json:"type"`
	Message    string `json:"message"`
}

func (e *APIError) Error() string {
	return fmt.Sprintf("API error (status %d): %s - %s", e.StatusCode, e.Type, e.Message)
}

// CreateChatCompletion sends a completion request to the OpenAI API
func (c *Client) CreateChatCompletion(ctx context.Context, req CompletionRequest) (*CompletionResponse, error) {
	var result *CompletionResponse

	operation := func() error {
		resp, err := c.doRequest(ctx, "POST", "/chat/completions", req)
		if err != nil {
			// Only retry on certain errors
			var apiErr *APIError
			if errors.As(err, &apiErr) {
				// Don't retry on client errors (except rate limits)
				if apiErr.StatusCode >= 400 && apiErr.StatusCode < 500 && apiErr.StatusCode != 429 {
					return backoff.Permanent(err)
				}
			}
			return err
		}
		result = resp
		return nil
	}

	// Create exponential backoff
	expBackoff := backoff.NewExponentialBackOff()
	expBackoff.MaxElapsedTime = time.Duration(c.retryMax) * time.Second

	err := backoff.Retry(operation, backoff.WithContext(expBackoff, ctx))
	if err != nil {
		return nil, err
	}

	return result, nil
}

// StreamChatCompletion streams a completion request to the OpenAI API
func (c *Client) StreamChatCompletion(ctx context.Context, req CompletionRequest,
	callback func(chunk *CompletionResponse) error) error {

	// Ensure streaming is enabled
	req.Stream = true

	// Make the request
	jsonData, err := json.Marshal(req)
	if err != nil {
		return fmt.Errorf("marshaling request: %w", err)
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		http.MethodPost,
		fmt.Sprintf("%s/chat/completions", c.baseURL),
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("Authorization", fmt.Sprintf("Bearer %s", c.apiKey))
	httpReq.Header.Set("Accept", "text/event-stream")

	c.logger.Printf("Sending streaming request to %s", httpReq.URL.String())

	resp, err := c.httpClient.Do(httpReq)
	if err != nil {
		return fmt.Errorf("executing request: %w", err)
	}
	defer resp.Body.Close()

	// Handle non-200 responses
	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return fmt.Errorf("API error (status %d): %s", resp.StatusCode, string(body))
	}

	// Process the stream
	reader := bufio.NewReader(resp.Body)
	for {
		line, err := reader.ReadBytes('\n')
		if err != nil {
			if err == io.EOF {
				break
			}
			return fmt.Errorf("reading stream: %w", err)
		}

		// Skip empty lines
		line = bytes.TrimSpace(line)
		if len(line) == 0 {
			continue
		}

		// Skip data prefix
		const prefix = "data: "
		if !bytes.HasPrefix(line, []byte(prefix)) {
			continue
		}
		line = bytes.TrimPrefix(line, []byte(prefix))

		// Check for stream end marker
		if string(line) == "[DONE]" {
			break
		}

		// Parse the response chunk
		var chunk CompletionResponse
		if err := json.Unmarshal(line, &chunk); err != nil {
			return fmt.Errorf("parsing stream chunk: %w", err)
		}

		// Call the callback with the chunk
		if err := callback(&chunk); err != nil {
			return err
		}
	}

	return nil
}

// doRequest executes an API request and returns the response
func (c *Client) doRequest(ctx context.Context, method, path string, body interface{}) (*CompletionResponse, error) {
	var jsonData []byte
	var err error

	if body != nil {
		jsonData, err = json.Marshal(body)
		if err != nil {
			return nil, fmt.Errorf("marshaling request: %w", err)
		}
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		method,
		fmt.Sprintf("%s%s", c.baseURL, path),
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return nil, fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("Authorization", fmt.Sprintf("Bearer %s", c.apiKey))

	c.logger.Printf("Sending request to %s", httpReq.URL.String())

	resp, err := c.httpClient.Do(httpReq)
	if err != nil {
		return nil, fmt.Errorf("executing request: %w", err)
	}
	defer resp.Body.Close()

	// Read the response body
	respBody, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("reading response body: %w", err)
	}

	// Handle non-200 responses
	if resp.StatusCode != http.StatusOK {
		var apiErr APIError
		if err := json.Unmarshal(respBody, &apiErr); err != nil {
			return nil, fmt.Errorf("API error (status %d): %s", resp.StatusCode, string(respBody))
		}
		apiErr.StatusCode = resp.StatusCode
		return nil, &apiErr
	}

	// Parse the response
	var result CompletionResponse
	if err := json.Unmarshal(respBody, &result); err != nil {
		return nil, fmt.Errorf("decoding response: %w", err)
	}

	return &result, nil
}
```

### **33.2.2 Building a Complete AI-powered Go Application**

Let's create a complete example of a simple AI-powered REST API service that provides code explanations:

````go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/go-chi/chi/v5"
	"github.com/go-chi/chi/v5/middleware"
	"github.com/go-chi/cors"
)

// CodeExplainerRequest represents a request for code explanation
type CodeExplainerRequest struct {
	Code        string `json:"code"`
	Language    string `json:"language"`
	ExplainType string `json:"explain_type"` // "simple", "detailed", "tutorial"
}

// CodeExplainerResponse represents a response with code explanation
type CodeExplainerResponse struct {
	Explanation string `json:"explanation"`
	TokensUsed  int    `json:"tokens_used"`
}

// Application encapsulates the application dependencies
type Application struct {
	AIClient *openai.Client
	Logger   *log.Logger
}

func main() {
	// Initialize logger
	logger := log.New(os.Stdout, "CODE-EXPLAINER: ", log.LstdFlags)

	// Get API key from environment
	apiKey := os.Getenv("OPENAI_API_KEY")
	if apiKey == "" {
		logger.Fatal("OPENAI_API_KEY environment variable is required")
	}

	// Initialize OpenAI client
	aiClient := openai.NewClient(
		apiKey,
		openai.WithLogger(logger),
		openai.WithMaxRetries(3),
	)

	// Create application instance
	app := &Application{
		AIClient: aiClient,
		Logger:   logger,
	}

	// Initialize router
	r := chi.NewRouter()

	// Middleware
	r.Use(middleware.RequestID)
	r.Use(middleware.RealIP)
	r.Use(middleware.Logger)
	r.Use(middleware.Recoverer)
	r.Use(middleware.Timeout(60 * time.Second))

	// CORS configuration
	r.Use(cors.Handler(cors.Options{
		AllowedOrigins:   []string{"*"},
		AllowedMethods:   []string{"GET", "POST", "OPTIONS"},
		AllowedHeaders:   []string{"Accept", "Content-Type"},
		ExposedHeaders:   []string{"Link"},
		AllowCredentials: true,
		MaxAge:           300,
	}))

	// Routes
	r.Get("/health", app.handleHealth)
	r.Post("/api/explain", app.handleExplainCode)

	// Create server
	srv := &http.Server{
		Addr:         ":8080",
		Handler:      r,
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 30 * time.Second,
		IdleTimeout:  120 * time.Second,
	}

	// Start server in a goroutine
	go func() {
		app.Logger.Printf("Starting server on port 8080")
		if err := srv.ListenAndServe(); err != http.ErrServerClosed {
			app.Logger.Fatalf("Server failed: %v", err)
		}
	}()

	// Set up graceful shutdown
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit

	app.Logger.Println("Shutting down server...")

	// Create context with timeout for shutdown
	ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
	defer cancel()

	if err := srv.Shutdown(ctx); err != nil {
		app.Logger.Fatalf("Server forced to shutdown: %v", err)
	}

	app.Logger.Println("Server gracefully stopped")
}

// handleHealth responds with the application's health status
func (app *Application) handleHealth(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(map[string]string{"status": "UP"})
}

// handleExplainCode handles code explanation requests
func (app *Application) handleExplainCode(w http.ResponseWriter, r *http.Request) {
	// Parse request
	var req CodeExplainerRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, err.Error(), http.StatusBadRequest)
		return
	}

	// Validate request
	if req.Code == "" {
		http.Error(w, "code is required", http.StatusBadRequest)
		return
	}

	if req.Language == "" {
		req.Language = "unknown"
	}

	if req.ExplainType == "" {
		req.ExplainType = "simple"
	}

	// Create context with timeout
	ctx, cancel := context.WithTimeout(r.Context(), 30*time.Second)
	defer cancel()

	// Create prompt based on explanation type
	prompt := createExplanationPrompt(req.Code, req.Language, req.ExplainType)

	// Create OpenAI request
	aiReq := openai.CompletionRequest{
		Model: "gpt-4",
		Messages: []openai.Message{
			{
				Role:    "system",
				Content: "You are an expert programming tutor who explains code clearly and concisely.",
			},
			{
				Role:    "user",
				Content: prompt,
			},
		},
		Temperature: 0.3,
		MaxTokens:   1000,
	}

	// Get explanation from OpenAI
	aiResp, err := app.AIClient.CreateChatCompletion(ctx, aiReq)
	if err != nil {
		app.Logger.Printf("OpenAI API error: %v", err)
		http.Error(w, "Failed to generate explanation", http.StatusInternalServerError)
		return
	}

	// Extract explanation
	var explanation string
	if len(aiResp.Choices) > 0 {
		explanation = aiResp.Choices[0].Message.Content
	} else {
		explanation = "No explanation generated"
	}

	// Create response
	resp := CodeExplainerResponse{
		Explanation: explanation,
		TokensUsed:  aiResp.Usage.TotalTokens,
	}

	// Return response
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(resp)
}

// createExplanationPrompt creates a prompt for the OpenAI API based on explanation type
func createExplanationPrompt(code, language, explainType string) string {
	var detailLevel string
	switch explainType {
	case "simple":
		detailLevel = "Give a brief, simple explanation suitable for beginners."
	case "detailed":
		detailLevel = "Provide a detailed explanation including how the code works and best practices."
	case "tutorial":
		detailLevel = "Create a step-by-step tutorial explaining each part of the code thoroughly."
	default:
		detailLevel = "Give a brief, simple explanation suitable for beginners."
	}

	return fmt.Sprintf(`Please explain the following %s code:

```%s
%s
````

%s Focus only on explaining the code. Be clear and concise.`,
language, language, code, detailLevel)
}

````

To use this application, you would:

1. Set the `OPENAI_API_KEY` environment variable
2. Run the application: `go run main.go`
3. Send a POST request to `/api/explain` with a JSON body:

```json
{
  "code": "func fibonacci(n int) int {\n  if n <= 1 {\n    return n\n  }\n  return fibonacci(n-1) + fibonacci(n-2)\n}",
  "language": "go",
  "explain_type": "detailed"
}
````

### **33.2.3 Implementing Local Fallbacks**

For production applications, you might want to implement local fallbacks in case the AI service is unavailable or too costly for certain operations:

```go
package main

import (
	"strings"
)

// FallbackExplainer provides simple explanations when AI services are unavailable
type FallbackExplainer struct {
	patterns map[string]string
}

// NewFallbackExplainer creates a new fallback explainer
func NewFallbackExplainer() *FallbackExplainer {
	return &FallbackExplainer{
		patterns: map[string]string{
			"func":      "This code defines a function in Go.",
			"if":        "This code contains a conditional statement that executes code based on a condition.",
			"for":       "This code contains a loop that repeats a block of code.",
			"return":    "This code returns a value from a function.",
			"import":    "This code imports external packages.",
			"struct":    "This code defines a data structure with fields.",
			"interface": "This code defines an interface specifying a set of method signatures.",
			"goroutine": "This code uses Go's concurrency mechanism to run functions concurrently.",
			"channel":   "This code uses channels for communication between goroutines.",
			"defer":     "This code uses defer to ensure a function call happens before the surrounding function returns.",
			"error":     "This code handles or returns errors.",
			"panic":     "This code triggers a runtime panic when something unexpected happens.",
			"recover":   "This code attempts to recover from a panic.",
		},
	}
}

// ExplainCode provides a simple explanation based on pattern matching
func (f *FallbackExplainer) ExplainCode(code, language string) string {
	// Default explanation
	explanation := "This is code written in " + language + "."

	// Look for known patterns
	for pattern, desc := range f.patterns {
		if strings.Contains(code, pattern) {
			explanation += " " + desc
		}
	}

	return explanation
}

// In the main application, use the fallback when AI fails:
func (app *Application) handleExplainCode(w http.ResponseWriter, r *http.Request) {
	// ... existing code ...

	// Get explanation from OpenAI
	aiResp, err := app.AIClient.CreateChatCompletion(ctx, aiReq)
	if err != nil {
		app.Logger.Printf("OpenAI API error: %v", err)

		// Use fallback if AI service fails
		fallback := NewFallbackExplainer()
		fallbackExplanation := fallback.ExplainCode(req.Code, req.Language)

		resp := CodeExplainerResponse{
			Explanation: "AI service unavailable. Basic explanation: " + fallbackExplanation,
			TokensUsed:  0,
		}

		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(resp)
		return
	}

	// ... existing code ...
}
```

## **33.3 Building LLM-Powered Applications**

When building applications that leverage LLMs, it's important to think beyond simple API calls and consider the overall architecture, prompt management, and response handling. This section explores patterns and best practices for creating robust LLM-powered applications in Go.

### Architecture Patterns for AI-Enhanced Applications

There are several architectural patterns that work well for AI-enhanced applications:

1. **API Gateway Pattern**: A Go service acts as a gateway between your application and one or more AI services, handling authentication, rate limiting, and failover.

2. **Middleware Pattern**: LLM capabilities are integrated as middleware in your application stack, processing requests and responses.

3. **Event-Driven Pattern**: LLM processing is triggered by events and runs asynchronously, with results published to other system components.

4. **Hybrid Pattern**: Combines local processing for simple tasks with API calls for complex reasoning.

Here's an example of a simple API gateway for LLM services:

```go
package main

import (
	"context"
	"encoding/json"
	"log"
	"net/http"
	"os"
	"time"
)

// AIService represents an AI service provider
type AIService interface {
	Generate(ctx context.Context, prompt string, options map[string]interface{}) (string, error)
	Name() string
}

// AIGateway manages multiple AI services with failover
type AIGateway struct {
	primaryService   AIService
	fallbackServices []AIService
	timeout          time.Duration
}

// NewAIGateway creates a new AI gateway
func NewAIGateway(primary AIService, fallbacks []AIService, timeout time.Duration) *AIGateway {
	return &AIGateway{
		primaryService:   primary,
		fallbackServices: fallbacks,
		timeout:          timeout,
	}
}

// Generate tries the primary service first, then falls back to alternatives if needed
func (g *AIGateway) Generate(prompt string, options map[string]interface{}) (string, error) {
	ctx, cancel := context.WithTimeout(context.Background(), g.timeout)
	defer cancel()

	// Try primary service
	result, err := g.primaryService.Generate(ctx, prompt, options)
	if err == nil {
		return result, nil
	}

	log.Printf("Primary service %s failed: %v. Trying fallbacks...", g.primaryService.Name(), err)

	// Try fallbacks in sequence
	for _, fallback := range g.fallbackServices {
		result, err := fallback.Generate(ctx, prompt, options)
		if err == nil {
			return result, nil
		}
		log.Printf("Fallback service %s failed: %v", fallback.Name(), err)
	}

	return "", err
}

// AIHandler handles HTTP requests for AI generation
type AIHandler struct {
	gateway *AIGateway
}

// GenerateRequest represents an AI generation request
type GenerateRequest struct {
	Prompt  string                 `json:"prompt"`
	Options map[string]interface{} `json:"options,omitempty"`
}

// GenerateResponse represents an AI generation response
type GenerateResponse struct {
	Result string `json:"result"`
	Error  string `json:"error,omitempty"`
}

// ServeHTTP handles HTTP requests
func (h *AIHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
		return
	}

	var req GenerateRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, "Invalid request", http.StatusBadRequest)
		return
	}

	result, err := h.gateway.Generate(req.Prompt, req.Options)

	resp := GenerateResponse{Result: result}
	if err != nil {
		resp.Error = err.Error()
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(resp)
}

func main() {
	// Setup services (implementation details omitted)
	primaryService := setupPrimaryService()
	fallbackServices := setupFallbackServices()

	gateway := NewAIGateway(primaryService, fallbackServices, 30*time.Second)
	handler := &AIHandler{gateway: gateway}

	// Setup HTTP server
	http.Handle("/generate", handler)
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}
	log.Printf("Starting server on :%s", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}

// Placeholder functions for service setup
func setupPrimaryService() AIService {
	// Implementation would create and return the primary service
	return nil
}

func setupFallbackServices() []AIService {
	// Implementation would create and return fallback services
	return nil
}
```

### Prompt Engineering and Management

Effective prompt engineering is crucial for getting good results from LLMs. In production applications, it's important to manage prompts systematically:

```go
package main

import (
	"bytes"
	"text/template"
)

// PromptTemplate represents a template for generating prompts
type PromptTemplate struct {
	template *template.Template
}

// NewPromptTemplate creates a new prompt template from a template string
func NewPromptTemplate(templateStr string) (*PromptTemplate, error) {
	tmpl, err := template.New("prompt").Parse(templateStr)
	if err != nil {
		return nil, err
	}
	return &PromptTemplate{template: tmpl}, nil
}

// Execute fills in the template with the given data
func (pt *PromptTemplate) Execute(data interface{}) (string, error) {
	var buf bytes.Buffer
	if err := pt.template.Execute(&buf, data); err != nil {
		return "", err
	}
	return buf.String(), nil
}

// PromptLibrary manages a collection of prompt templates
type PromptLibrary struct {
	templates map[string]*PromptTemplate
}

// NewPromptLibrary creates a new prompt library
func NewPromptLibrary() *PromptLibrary {
	return &PromptLibrary{
		templates: make(map[string]*PromptTemplate),
	}
}

// Register adds a template to the library
func (pl *PromptLibrary) Register(name, templateStr string) error {
	tmpl, err := NewPromptTemplate(templateStr)
	if err != nil {
		return err
	}
	pl.templates[name] = tmpl
	return nil
}

// Get retrieves a template from the library
func (pl *PromptLibrary) Get(name string) (*PromptTemplate, bool) {
	tmpl, ok := pl.templates[name]
	return tmpl, ok
}

// Execute fills in a template with the given data
func (pl *PromptLibrary) Execute(name string, data interface{}) (string, error) {
	tmpl, ok := pl.Get(name)
	if !ok {
		return "", fmt.Errorf("template not found: %s", name)
	}
	return tmpl.Execute(data)
}
```

Usage example:

```go
func main() {
	library := NewPromptLibrary()

	// Register templates
	err := library.Register("summarize",
		"Summarize the following text in {{.WordCount}} words or less:\n\n{{.Text}}")
	if err != nil {
		log.Fatalf("Error registering template: %v", err)
	}

	// Use the template
	prompt, err := library.Execute("summarize", map[string]interface{}{
		"WordCount": 100,
		"Text":      "Lorem ipsum dolor sit amet...",
	})
	if err != nil {
		log.Fatalf("Error executing template: %v", err)
	}

	fmt.Println(prompt)
}
```

### Streaming Responses

Many LLM APIs support streaming responses, which can significantly improve the perceived performance of your application. Here's how to implement streaming with the OpenAI API:

```go
package main

import (
	"bufio"
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"strings"
	"time"
)

// StreamingClient provides methods to interact with the OpenAI streaming API
type StreamingClient struct {
	apiKey     string
	httpClient *http.Client
	baseURL    string
}

// StreamingRequest represents a request to the streaming API
type StreamingRequest struct {
	Model       string    `json:"model"`
	Messages    []Message `json:"messages"`
	Temperature float64   `json:"temperature,omitempty"`
	Stream      bool      `json:"stream"`
}

// Message represents a message in the chat completion API
type Message struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

// StreamResponse represents a streaming response chunk
type StreamResponse struct {
	ID      string `json:"id"`
	Object  string `json:"object"`
	Created int    `json:"created"`
	Model   string `json:"model"`
	Choices []struct {
		Delta struct {
			Content string `json:"content,omitempty"`
		} `json:"delta"`
		FinishReason string `json:"finish_reason"`
	} `json:"choices"`
}

// NewStreamingClient creates a new streaming client
func NewStreamingClient(apiKey string) *StreamingClient {
	return &StreamingClient{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 120 * time.Second,
		},
		baseURL: "https://api.openai.com/v1",
	}
}

// StreamCompletion streams a completion from the API
func (c *StreamingClient) StreamCompletion(ctx context.Context, req StreamingRequest, callback func(string, error)) error {
	// Ensure stream flag is set
	req.Stream = true

	jsonData, err := json.Marshal(req)
	if err != nil {
		return fmt.Errorf("marshaling request: %w", err)
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		http.MethodPost,
		fmt.Sprintf("%s/chat/completions", c.baseURL),
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("Authorization", fmt.Sprintf("Bearer %s", c.apiKey))
	httpReq.Header.Set("Accept", "text/event-stream")

	resp, err := c.httpClient.Do(httpReq)
	if err != nil {
		return fmt.Errorf("sending request: %w", err)
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusOK {
		body, _ := io.ReadAll(resp.Body)
		return fmt.Errorf("API request failed with status %d: %s", resp.StatusCode, body)
	}

	reader := bufio.NewReader(resp.Body)

	for {
		line, err := reader.ReadBytes('\n')
		if err != nil {
			if err == io.EOF {
				break
			}
			callback("", fmt.Errorf("reading stream: %w", err))
			return err
		}

		line = strings.TrimSpace(line)
		if line == "" {
			continue
		}

		// SSE format: lines starting with "data: "
		if !strings.HasPrefix(line, "data: ") {
			continue
		}

		data := strings.TrimPrefix(line, "data: ")

		// End of stream marker
		if data == "[DONE]" {
			break
		}

		var streamResp StreamResponse
		if err := json.Unmarshal([]byte(data), &streamResp); err != nil {
			callback("", fmt.Errorf("unmarshaling response: %w", err))
			continue
		}

		if len(streamResp.Choices) > 0 {
			content := streamResp.Choices[0].Delta.Content
			callback(content, nil)

			if streamResp.Choices[0].FinishReason != "" {
				break
			}
		}
	}

	return nil
}

// Example usage
func main() {
	client := NewStreamingClient("your-api-key")

	ctx := context.Background()

	req := StreamingRequest{
		Model: "gpt-4",
		Messages: []Message{
			{
				Role:    "user",
				Content: "Write a short story about a Go programmer.",
			},
		},
		Temperature: 0.7,
	}

	fmt.Println("Generating story...")

	err := client.StreamCompletion(ctx, req, func(chunk string, err error) {
		if err != nil {
			fmt.Printf("Error: %v\n", err)
			return
		}
		fmt.Print(chunk)
	})

	if err != nil {
		fmt.Printf("Stream error: %v\n", err)
	}

	fmt.Println("\nDone!")
}
```

### Handling Context and Memory

LLMs work best when provided with relevant context. For chat applications or agents that need to maintain state, implementing a context management system is essential:

```go
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

// Message represents a message in a conversation
type Message struct {
	Role      string    `json:"role"`
	Content   string    `json:"content"`
	Timestamp time.Time `json:"timestamp"`
}

// Conversation represents a conversation with an AI
type Conversation struct {
	ID            string    `json:"id"`
	Title         string    `json:"title"`
	Messages      []Message `json:"messages"`
	SystemPrompt  string    `json:"system_prompt"`
	TokenCount    int       `json:"token_count"`
	MaxTokens     int       `json:"max_tokens"`
	LastUpdated   time.Time `json:"last_updated"`
	TokenEstimator TokenEstimator
}

// TokenEstimator estimates the number of tokens in a text
type TokenEstimator interface {
	EstimateTokens(text string) int
}

// NewConversation creates a new conversation
func NewConversation(id, title, systemPrompt string, maxTokens int, estimator TokenEstimator) *Conversation {
	return &Conversation{
		ID:            id,
		Title:         title,
		SystemPrompt:  systemPrompt,
		Messages:      []Message{},
		MaxTokens:     maxTokens,
		LastUpdated:   time.Now(),
		TokenEstimator: estimator,
	}
}

// AddMessage adds a message to the conversation
func (c *Conversation) AddMessage(role, content string) {
	message := Message{
		Role:      role,
		Content:   content,
		Timestamp: time.Now(),
	}
	c.Messages = append(c.Messages, message)
	c.TokenCount += c.TokenEstimator.EstimateTokens(content)
	c.LastUpdated = time.Now()

	// Prune if necessary to stay under token limit
	c.pruneToFitTokenLimit()
}

// GetMessages returns the messages formatted for API submission
func (c *Conversation) GetMessages() []map[string]string {
	messages := make([]map[string]string, 0, len(c.Messages)+1)

	// Add system message if present
	if c.SystemPrompt != "" {
		messages = append(messages, map[string]string{
			"role":    "system",
			"content": c.SystemPrompt,
		})
	}

	// Add conversation messages
	for _, msg := range c.Messages {
		messages = append(messages, map[string]string{
			"role":    msg.Role,
			"content": msg.Content,
		})
	}

	return messages
}

// pruneToFitTokenLimit removes oldest messages to stay under the token limit
func (c *Conversation) pruneToFitTokenLimit() {
	if c.TokenCount <= c.MaxTokens {
		return
	}

	// Keep system prompt and remove oldest messages until under limit
	// Preserve the most recent messages as they have the most context relevance
	for i := 0; i < len(c.Messages) && c.TokenCount > c.MaxTokens; i++ {
		// Remove the oldest message (keep system prompt if any)
		removedMsg := c.Messages[0]
		c.Messages = c.Messages[1:]
		c.TokenCount -= c.TokenEstimator.EstimateTokens(removedMsg.Content)
	}
}

// Save serializes the conversation to JSON
func (c *Conversation) Save() ([]byte, error) {
	return json.Marshal(c)
}

// Load deserializes the conversation from JSON
func LoadConversation(data []byte, estimator TokenEstimator) (*Conversation, error) {
	var c Conversation
	if err := json.Unmarshal(data, &c); err != nil {
		return nil, err
	}
	c.TokenEstimator = estimator
	return &c, nil
}

// SimpleTokenEstimator provides a simple token estimation method
type SimpleTokenEstimator struct{}

// EstimateTokens estimates tokens using a simple approximation
func (e SimpleTokenEstimator) EstimateTokens(text string) int {
	// Simple approximation: 1 token ≈ 4 characters
	return len(text) / 4
}
```

These examples demonstrate key patterns for building LLM-powered applications in Go. The next sections will explore more advanced topics like retrieval-augmented generation and local model inference.

## **33.4 Retrieval-Augmented Generation (RAG)**

Retrieval-Augmented Generation (RAG) combines the power of LLMs with document retrieval systems to enhance responses with specific knowledge from a corpus of documents. In this section, we'll explore how to implement a RAG system in Go.

### Vector Databases with Go

Vector databases are essential for RAG systems, as they enable efficient similarity search for embeddings. Several vector databases have Go clients:

#### Using Qdrant with Go

Qdrant is a vector database optimized for similarity search. Here's how to integrate it with Go:

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/qdrant/go-client/qdrant"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
)

// QdrantClient wraps the Qdrant client for vector operations
type QdrantClient struct {
	client      qdrant.QdrantClient
	collection  string
	dimension   uint64
	conn        *grpc.ClientConn
}

// NewQdrantClient creates a new Qdrant client
func NewQdrantClient(address, collection string, dimension uint64) (*QdrantClient, error) {
	conn, err := grpc.Dial(address, grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		return nil, fmt.Errorf("failed to connect to Qdrant: %w", err)
	}

	client := qdrant.NewQdrantClient(conn)

	return &QdrantClient{
		client:     client,
		collection: collection,
		dimension:  dimension,
		conn:       conn,
	}, nil
}

// Close closes the connection to Qdrant
func (c *QdrantClient) Close() error {
	return c.conn.Close()
}

// CreateCollection creates a new collection in Qdrant
func (c *QdrantClient) CreateCollection(ctx context.Context) error {
	_, err := c.client.CreateCollection(ctx, &qdrant.CreateCollection{
		CollectionName: c.collection,
		VectorsConfig: &qdrant.VectorsConfig{
			Config: &qdrant.VectorsConfig_Params{
				Params: &qdrant.VectorParams{
					Size:     c.dimension,
					Distance: qdrant.Distance_Cosine,
				},
			},
		},
	})
	if err != nil {
		return fmt.Errorf("failed to create collection: %w", err)
	}
	return nil
}

// UpsertVectors adds or updates vectors in the collection
func (c *QdrantClient) UpsertVectors(ctx context.Context, points []*qdrant.PointStruct) error {
	_, err := c.client.Upsert(ctx, &qdrant.UpsertPoints{
		CollectionName: c.collection,
		Points:         points,
	})
	if err != nil {
		return fmt.Errorf("failed to upsert vectors: %w", err)
	}
	return nil
}

// Search performs a similarity search in the collection
func (c *QdrantClient) Search(ctx context.Context, vector []float32, limit uint64) ([]*qdrant.ScoredPoint, error) {
	result, err := c.client.Search(ctx, &qdrant.SearchPoints{
		CollectionName: c.collection,
		Vector:         vector,
		Limit:          limit,
	})
	if err != nil {
		return nil, fmt.Errorf("failed to search vectors: %w", err)
	}
	return result.Result, nil
}

// Example usage
func main() {
	// Initialize Qdrant client
	qdrantClient, err := NewQdrantClient("localhost:6334", "documents", 1536) // 1536 for OpenAI embeddings
	if err != nil {
		log.Fatalf("Failed to create Qdrant client: %v", err)
	}
	defer qdrantClient.Close()

	ctx := context.Background()

	// Create collection if needed
	err = qdrantClient.CreateCollection(ctx)
	if err != nil {
		log.Printf("Collection creation error (may already exist): %v", err)
	}

	// Example document vectors (in practice, these would be generated from text)
	documentVectors := []*qdrant.PointStruct{
		{
			Id: &qdrant.PointId{
				PointIdOptions: &qdrant.PointId_Uuid{
					Uuid: "doc1",
				},
			},
			Vectors: &qdrant.Vectors{
				VectorsOptions: &qdrant.Vectors_Vector{
					Vector: &qdrant.Vector{
						Data: []float32{0.1, 0.2, 0.3, /* ... remaining values ... */},
					},
				},
			},
			Payload: map[string]*qdrant.Value{
				"text": {
					Kind: &qdrant.Value_StringValue{
						StringValue: "Go is a statically typed, compiled programming language.",
					},
				},
				"source": {
					Kind: &qdrant.Value_StringValue{
						StringValue: "programming_languages.md",
					},
				},
			},
		},
		// Additional documents would be added here
	}

	// Upsert vectors
	err = qdrantClient.UpsertVectors(ctx, documentVectors)
	if err != nil {
		log.Fatalf("Failed to upsert vectors: %v", err)
	}

	// Search for similar documents
	queryVector := []float32{0.15, 0.25, 0.35, /* ... remaining values ... */}
	results, err := qdrantClient.Search(ctx, queryVector, 5)
	if err != nil {
		log.Fatalf("Failed to search vectors: %v", err)
	}

	// Process search results
	for _, result := range results {
		text := result.Payload["text"].GetStringValue()
		score := result.Score
		fmt.Printf("Score: %.4f, Text: %s\n", score, text)
	}
}
```

#### Using PGVector with Go

PostgreSQL with the pgvector extension is another popular choice for vector storage:

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
	"log"

	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"
)

// PGVectorClient wraps PostgreSQL with pgvector for vector operations
type PGVectorClient struct {
	pool *pgxpool.Pool
}

// NewPGVectorClient creates a new PGVector client
func NewPGVectorClient(connStr string) (*PGVectorClient, error) {
	config, err := pgxpool.ParseConfig(connStr)
	if err != nil {
		return nil, fmt.Errorf("invalid connection string: %w", err)
	}

	pool, err := pgxpool.NewWithConfig(context.Background(), config)
	if err != nil {
		return nil, fmt.Errorf("failed to create connection pool: %w", err)
	}

	// Check if pgvector extension is installed
	var hasExtension bool
	err = pool.QueryRow(context.Background(), "SELECT EXISTS (SELECT 1 FROM pg_extension WHERE extname = 'vector')").Scan(&hasExtension)
	if err != nil {
		pool.Close()
		return nil, fmt.Errorf("failed to check for vector extension: %w", err)
	}

	if !hasExtension {
		pool.Close()
		return nil, fmt.Errorf("pgvector extension is not installed in the database")
	}

	return &PGVectorClient{pool: pool}, nil
}

// Close closes the database connection pool
func (c *PGVectorClient) Close() {
	c.pool.Close()
}

// CreateDocumentsTable creates a table for storing documents and embeddings
func (c *PGVectorClient) CreateDocumentsTable(ctx context.Context) error {
	_, err := c.pool.Exec(ctx, `
		CREATE TABLE IF NOT EXISTS documents (
			id SERIAL PRIMARY KEY,
			content TEXT NOT NULL,
			metadata JSONB,
			embedding vector(1536) NOT NULL
		);
		CREATE INDEX IF NOT EXISTS documents_embedding_idx ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
	`)
	return err
}

// UpsertDocument adds or updates a document and its embedding
func (c *PGVectorClient) UpsertDocument(ctx context.Context, content string, metadata map[string]interface{}, embedding []float32) (int, error) {
	var id int
	metadataJSON, err := json.Marshal(metadata)
	if err != nil {
		return 0, fmt.Errorf("failed to marshal metadata: %w", err)
	}

	// Convert embedding to PostgreSQL vector format
	embedStr := "["
	for i, val := range embedding {
		if i > 0 {
			embedStr += ","
		}
		embedStr += fmt.Sprintf("%f", val)
	}
	embedStr += "]"

	err = c.pool.QueryRow(ctx, `
		INSERT INTO documents (content, metadata, embedding)
		VALUES ($1, $2, $3)
		RETURNING id
	`, content, metadataJSON, embedStr).Scan(&id)

	if err != nil {
		return 0, fmt.Errorf("failed to upsert document: %w", err)
	}

	return id, nil
}

// SearchSimilarDocuments finds documents similar to the provided embedding
func (c *PGVectorClient) SearchSimilarDocuments(ctx context.Context, embedding []float32, limit int) ([]Document, error) {
	// Convert embedding to PostgreSQL vector format
	embedStr := "["
	for i, val := range embedding {
		if i > 0 {
			embedStr += ","
		}
		embedStr += fmt.Sprintf("%f", val)
	}
	embedStr += "]"

	rows, err := c.pool.Query(ctx, `
		SELECT id, content, metadata, 1 - (embedding <=> $1) AS similarity
		FROM documents
		ORDER BY embedding <=> $1
		LIMIT $2
	`, embedStr, limit)
	if err != nil {
		return nil, fmt.Errorf("failed to search documents: %w", err)
	}
	defer rows.Close()

	var documents []Document
	for rows.Next() {
		var doc Document
		var metadataJSON []byte
		err := rows.Scan(&doc.ID, &doc.Content, &metadataJSON, &doc.Similarity)
		if err != nil {
			return nil, fmt.Errorf("failed to scan row: %w", err)
		}

		if err := json.Unmarshal(metadataJSON, &doc.Metadata); err != nil {
			return nil, fmt.Errorf("failed to unmarshal metadata: %w", err)
		}

		documents = append(documents, doc)
	}

	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("error iterating rows: %w", err)
	}

	return documents, nil
}

// Document represents a document with embedding and metadata
type Document struct {
	ID         int                    `json:"id"`
	Content    string                 `json:"content"`
	Metadata   map[string]interface{} `json:"metadata"`
	Similarity float64                `json:"similarity,omitempty"`
}
```

### Document Processing and Chunking

Effective RAG systems require preprocessing documents into appropriate chunks for retrieval. Here's how to implement document chunking in Go:

```go
package main

import (
	"fmt"
	"regexp"
	"strings"
)

// Chunk represents a piece of text with metadata
type Chunk struct {
	Text     string                 `json:"text"`
	Metadata map[string]interface{} `json:"metadata"`
}

// DocumentChunker splits documents into chunks suitable for embedding
type DocumentChunker struct {
	ChunkSize    int
	ChunkOverlap int
}

// NewDocumentChunker creates a new document chunker
func NewDocumentChunker(chunkSize, chunkOverlap int) *DocumentChunker {
	return &DocumentChunker{
		ChunkSize:    chunkSize,
		ChunkOverlap: chunkOverlap,
	}
}

// ChunkDocument splits a document into overlapping chunks
func (c *DocumentChunker) ChunkDocument(text string, metadata map[string]interface{}) []Chunk {
	// Normalize text
	text = normalizeText(text)

	// If the text is shorter than the chunk size, return it as a single chunk
	if len(text) <= c.ChunkSize {
		return []Chunk{
			{
				Text:     text,
				Metadata: metadata,
			},
		}
	}

	var chunks []Chunk

	// Split text into paragraphs first for more natural chunks
	paragraphs := splitIntoParagraphs(text)

	var currentChunk strings.Builder
	currentSize := 0

	for _, paragraph := range paragraphs {
		// If adding this paragraph would exceed the chunk size,
		// save the current chunk and start a new one
		if currentSize+len(paragraph) > c.ChunkSize && currentSize > 0 {
			chunks = append(chunks, Chunk{
				Text:     currentChunk.String(),
				Metadata: copyMetadata(metadata),
			})

			// Calculate overlap
			if c.ChunkOverlap > 0 && currentSize > c.ChunkOverlap {
				// Get the last n characters for overlap
				text := currentChunk.String()
				overlapText := text[len(text)-c.ChunkOverlap:]

				currentChunk = strings.Builder{}
				currentChunk.WriteString(overlapText)
				currentSize = len(overlapText)
			} else {
				currentChunk = strings.Builder{}
				currentSize = 0
			}
		}

		// Add paragraph to current chunk
		if currentSize > 0 {
			currentChunk.WriteString("\n\n")
			currentSize += 2
		}

		currentChunk.WriteString(paragraph)
		currentSize += len(paragraph)
	}

	// Add the last chunk if there's anything left
	if currentSize > 0 {
		chunks = append(chunks, Chunk{
			Text:     currentChunk.String(),
			Metadata: copyMetadata(metadata),
		})
	}

	return chunks
}

// Helper functions

// normalizeText cleans and normalizes text
func normalizeText(text string) string {
	// Remove excessive whitespace
	re := regexp.MustCompile(`\s+`)
	text = re.ReplaceAllString(text, " ")

	// Trim leading/trailing whitespace
	return strings.TrimSpace(text)
}

// splitIntoParagraphs splits text into paragraphs
func splitIntoParagraphs(text string) []string {
	// Split by double newlines or more
	re := regexp.MustCompile(`\n\s*\n`)
	paragraphs := re.Split(text, -1)

	// Filter out empty paragraphs
	var filtered []string
	for _, p := range paragraphs {
		if trimmed := strings.TrimSpace(p); trimmed != "" {
			filtered = append(filtered, trimmed)
		}
	}

	return filtered
}

// copyMetadata creates a copy of the metadata map
func copyMetadata(metadata map[string]interface{}) map[string]interface{} {
	copy := make(map[string]interface{}, len(metadata))
	for k, v := range metadata {
		copy[k] = v
	}
	return copy
}

// Example usage
func main() {
	chunker := NewDocumentChunker(1000, 200)

	document := `
	# Go Programming Language

	Go is a statically typed, compiled programming language designed at Google.

	## Features

	Go provides garbage collection, type safety, memory safety, and CSP-style concurrent programming features.

	## History

	Go was designed by Robert Griesemer, Rob Pike, and Ken Thompson at Google and was announced in November 2009.
	`

	metadata := map[string]interface{}{
		"source": "programming_languages.md",
		"author": "John Doe",
		"date":   "2023-05-15",
	}

	chunks := chunker.ChunkDocument(document, metadata)

	for i, chunk := range chunks {
		fmt.Printf("Chunk %d (%d chars):\n%s\n\n", i+1, len(chunk.Text), chunk.Text)
	}
}
```

### Semantic Search Implementation

To complete a RAG system, we need to implement semantic search using embeddings. Here's how to integrate with OpenAI's embedding API:

```go
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
	"time"
)

// EmbeddingClient handles text embeddings
type EmbeddingClient struct {
	apiKey     string
	httpClient *http.Client
	model      string
}

// EmbeddingRequest represents a request to the embedding API
type EmbeddingRequest struct {
	Model string   `json:"model"`
	Input []string `json:"input"`
}

// EmbeddingResponse represents a response from the embedding API
type EmbeddingResponse struct {
	Object string `json:"object"`
	Data   []struct {
		Object    string    `json:"object"`
		Embedding []float32 `json:"embedding"`
		Index     int       `json:"index"`
	} `json:"data"`
	Model string `json:"model"`
	Usage struct {
		PromptTokens int `json:"prompt_tokens"`
		TotalTokens  int `json:"total_tokens"`
	} `json:"usage"`
}

// NewEmbeddingClient creates a new embedding client
func NewEmbeddingClient(apiKey, model string) *EmbeddingClient {
	return &EmbeddingClient{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 30 * time.Second,
		},
		model: model,
	}
}

// CreateEmbeddings generates embeddings for the provided texts
func (c *EmbeddingClient) CreateEmbeddings(ctx context.Context, texts []string) ([][]float32, error) {
	req := EmbeddingRequest{
		Model: c.model,
		Input: texts,
	}

	jsonData, err := json.Marshal(req)
	if err != nil {
		return nil, fmt.Errorf("marshaling request: %w", err)
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		http.MethodPost,
		"https://api.openai.com/v1/embeddings",
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return nil, fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("Authorization", fmt.Sprintf("Bearer %s", c.apiKey))

	resp, err := c.httpClient.Do(httpReq)
	if err != nil {
		return nil, fmt.Errorf("sending request: %w", err)
	}
	defer resp.Body.Close()

	body, err := io.ReadAll(resp.Body)
	if err != nil {
		return nil, fmt.Errorf("reading response body: %w", err)
	}

	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("API request failed with status %d: %s", resp.StatusCode, body)
	}

	var embResp EmbeddingResponse
	if err := json.Unmarshal(body, &embResp); err != nil {
		return nil, fmt.Errorf("unmarshaling response: %w", err)
	}

	// Sort embeddings by index to ensure they match the input order
	embeddings := make([][]float32, len(embResp.Data))
	for _, item := range embResp.Data {
		embeddings[item.Index] = item.Embedding
	}

	return embeddings, nil
}
```

### Integrating RAG with LLMs

Finally, let's put everything together to create a complete RAG system:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"strings"
)

// RAGSystem combines document retrieval with LLM generation
type RAGSystem struct {
	vectorDB         *PGVectorClient
	embeddingClient  *EmbeddingClient
	llmClient        *OpenAIClient
	chunker          *DocumentChunker
}

// NewRAGSystem creates a new RAG system
func NewRAGSystem(vectorDB *PGVectorClient, embeddingClient *EmbeddingClient, llmClient *OpenAIClient) *RAGSystem {
	return &RAGSystem{
		vectorDB:        vectorDB,
		embeddingClient: embeddingClient,
		llmClient:       llmClient,
		chunker:         NewDocumentChunker(1000, 200),
	}
}

// AddDocument adds a document to the RAG system
func (r *RAGSystem) AddDocument(ctx context.Context, content string, metadata map[string]interface{}) error {
	// Split document into chunks
	chunks := r.chunker.ChunkDocument(content, metadata)

	// Process chunks in batches to avoid API rate limits
	batchSize := 20
	for i := 0; i < len(chunks); i += batchSize {
		end := i + batchSize
		if end > len(chunks) {
			end = len(chunks)
		}

		batch := chunks[i:end]

		// Extract text for embedding
		texts := make([]string, len(batch))
		for j, chunk := range batch {
			texts[j] = chunk.Text
		}

		// Generate embeddings
		embeddings, err := r.embeddingClient.CreateEmbeddings(ctx, texts)
		if err != nil {
			return fmt.Errorf("generating embeddings: %w", err)
		}

		// Store chunks with embeddings
		for j, chunk := range batch {
			_, err := r.vectorDB.UpsertDocument(ctx, chunk.Text, chunk.Metadata, embeddings[j])
			if err != nil {
				return fmt.Errorf("storing document chunk: %w", err)
			}
		}
	}

	return nil
}

// Query performs a RAG query
func (r *RAGSystem) Query(ctx context.Context, query string, numResults int) (string, error) {
	// Generate embedding for the query
	embeddings, err := r.embeddingClient.CreateEmbeddings(ctx, []string{query})
	if err != nil {
		return "", fmt.Errorf("generating query embedding: %w", err)
	}

	queryEmbedding := embeddings[0]

	// Retrieve relevant documents
	docs, err := r.vectorDB.SearchSimilarDocuments(ctx, queryEmbedding, numResults)
	if err != nil {
		return "", fmt.Errorf("searching similar documents: %w", err)
	}

	// Build context from retrieved documents
	var contextBuilder strings.Builder
	contextBuilder.WriteString("### Retrieved Information:\n\n")

	for i, doc := range docs {
		contextBuilder.WriteString(fmt.Sprintf("Document %d (Similarity: %.4f):\n%s\n\n",
			i+1, doc.Similarity, doc.Content))
	}

	context := contextBuilder.String()

	// Construct prompt with retrieved context
	systemPrompt := "You are a helpful assistant. Answer the question based ONLY on the provided context. " +
		"If the answer cannot be determined from the context, say 'I don't have enough information to answer this question.'"

	userPrompt := fmt.Sprintf("Context:\n%s\n\nQuestion: %s", context, query)

	// Generate response using LLM
	req := CompletionRequest{
		Model: "gpt-4",
		Messages: []Message{
			{
				Role:    "system",
				Content: systemPrompt,
			},
			{
				Role:    "user",
				Content: userPrompt,
			},
		},
		Temperature: 0.3, // Lower temperature for more factual responses
		MaxTokens:   1000,
	}

	// Get explanation from OpenAI
	aiResp, err := r.llmClient.CreateChatCompletion(ctx, req)
	if err != nil {
		return "", fmt.Errorf("generating LLM response: %w", err)
	}

	if len(aiResp.Choices) == 0 {
		return "", fmt.Errorf("no response generated")
	}

	return aiResp.Choices[0].Message.Content, nil
}

// Example usage
func main() {
	// Initialize clients
	apiKey := os.Getenv("OPENAI_API_KEY")
	if apiKey == "" {
		log.Fatal("OPENAI_API_KEY environment variable is required")
	}

	vectorDB, err := NewPGVectorClient("postgres://user:password@localhost:5432/ragdb")
	if err != nil {
		log.Fatalf("Failed to create vector DB client: %v", err)
	}
	defer vectorDB.Close()

	embeddingClient := NewEmbeddingClient(apiKey, "text-embedding-ada-002")
	llmClient := NewOpenAIClient(apiKey)

	// Create RAG system
	ragSystem := NewRAGSystem(vectorDB, embeddingClient, llmClient)

	ctx := context.Background()

	// Ensure DB schema is set up
	err = vectorDB.CreateDocumentsTable(ctx)
	if err != nil {
		log.Fatalf("Failed to create documents table: %v", err)
	}

	// Add a document
	err = ragSystem.AddDocument(ctx,
		"Go's concurrency model is based on CSP (Communicating Sequential Processes). " +
		"It uses goroutines for lightweight concurrency and channels for communication between goroutines. " +
		"Goroutines are much lighter than threads, allowing thousands to run simultaneously.",
		map[string]interface{}{
			"source": "concurrency.md",
			"topic":  "Go Concurrency",
		})
	if err != nil {
		log.Fatalf("Failed to add document: %v", err)
	}

	// Query the RAG system
	response, err := ragSystem.Query(ctx, "How does Go handle concurrency?", 3)
	if err != nil {
		log.Fatalf("Failed to query: %v", err)
	}

	fmt.Println("Query: How does Go handle concurrency?")
	fmt.Println("Response:", response)
}
```

This implementation demonstrates a complete RAG system in Go, including vector storage, document chunking, embedding generation, and LLM integration. The system can be extended with more sophisticated document processing, improved retrieval algorithms, and caching mechanisms for production use.

## **33.5 Local Model Inference**

- ONNX Runtime integration
- Transformer model inference
- Quantization and optimization
- Model caching strategies

## **33.6 Building Autonomous AI Agents**

- Agent architecture in Go
- Tool usage and function calling
- Planning and reasoning systems
- Multi-agent communication

## **33.7 Real-time AI Applications**

- Websocket integration for streaming AI
- Concurrent processing of AI requests
- Scaling considerations
- Load balancing and fallback strategies

## **33.8 Security and Ethical Considerations**

- Prompt injection prevention
- Data privacy with AI systems
- Implementing content filtering
- Responsible AI development

## **33.9 Case Studies and Applications**

- Customer service automation
- Content generation systems
- Intelligent search applications
- AI-powered decision support

## **33.10 Conclusion**

- Future of Go in the AI ecosystem
- Emerging trends and technologies
- Building a career at the intersection of Go and AI
