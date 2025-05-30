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

	var completionResp CompletionResponse
	if err := json.Unmarshal(body, &completionResp); err != nil {
		return nil, fmt.Errorf("unmarshaling response: %w", err)
	}

	return &completionResp, nil
}

func main() {
	apiKey := os.Getenv("OPENAI_API_KEY")
	if apiKey == "" {
		fmt.Println("Please set the OPENAI_API_KEY environment variable")
		return
	}

	client := NewOpenAIClient(apiKey)

	ctx := context.Background()

	req := CompletionRequest{
		Model: "gpt-4",
		Messages: []Message{
			{
				Role:    "system",
				Content: "You are a helpful assistant that provides concise responses.",
			},
			{
				Role:    "user",
				Content: "What are the key features of the Go programming language?",
			},
		},
		Temperature: 0.7,
		MaxTokens:   300,
	}

	resp, err := client.CreateChatCompletion(ctx, req)
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}

	if len(resp.Choices) > 0 {
		fmt.Printf("Response: %s\n", resp.Choices[0].Message.Content)
		fmt.Printf("Tokens used: %d\n", resp.Usage.TotalTokens)
	}
}
```

### Anthropic Claude API Integration

Anthropic's Claude models are known for their longer context windows and strong instruction-following capabilities. Here's how to integrate the Claude API:

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

// ClaudeClient provides methods to interact with the Anthropic Claude API
type ClaudeClient struct {
	apiKey     string
	httpClient *http.Client
	baseURL    string
}

// MessageRequest represents a request to the Claude messages API
type MessageRequest struct {
	Model       string        `json:"model"`
	Messages    []ClaudeMessage `json:"messages"`
	MaxTokens   int           `json:"max_tokens,omitempty"`
	Temperature float64       `json:"temperature,omitempty"`
	System      string        `json:"system,omitempty"`
}

// ClaudeMessage represents a message in the Claude API
type ClaudeMessage struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

// MessageResponse represents a response from the Claude API
type MessageResponse struct {
	ID        string `json:"id"`
	Type      string `json:"type"`
	Role      string `json:"role"`
	Content   []ContentBlock `json:"content"`
	Model     string `json:"model"`
	StopReason string `json:"stop_reason"`
	Usage     struct {
		InputTokens  int `json:"input_tokens"`
		OutputTokens int `json:"output_tokens"`
	} `json:"usage"`
}

// ContentBlock represents a block of content in the Claude API response
type ContentBlock struct {
	Type string `json:"type"`
	Text string `json:"text"`
}

// NewClaudeClient creates a new Claude client
func NewClaudeClient(apiKey string) *ClaudeClient {
	return &ClaudeClient{
		apiKey: apiKey,
		httpClient: &http.Client{
			Timeout: 60 * time.Second,
		},
		baseURL: "https://api.anthropic.com/v1",
	}
}

// CreateMessage sends a message request to the Claude API
func (c *ClaudeClient) CreateMessage(ctx context.Context, req MessageRequest) (*MessageResponse, error) {
	jsonData, err := json.Marshal(req)
	if err != nil {
		return nil, fmt.Errorf("marshaling request: %w", err)
	}

	httpReq, err := http.NewRequestWithContext(
		ctx,
		http.MethodPost,
		fmt.Sprintf("%s/messages", c.baseURL),
		bytes.NewBuffer(jsonData),
	)
	if err != nil {
		return nil, fmt.Errorf("creating request: %w", err)
	}

	httpReq.Header.Set("Content-Type", "application/json")
	httpReq.Header.Set("x-api-key", c.apiKey)
	httpReq.Header.Set("anthropic-version", "2023-06-01")

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

	var messageResp MessageResponse
	if err := json.Unmarshal(body, &messageResp); err != nil {
		return nil, fmt.Errorf("unmarshaling response: %w", err)
	}

	return &messageResp, nil
}

func main() {
	apiKey := os.Getenv("ANTHROPIC_API_KEY")
	if apiKey == "" {
		fmt.Println("Please set the ANTHROPIC_API_KEY environment variable")
		return
	}

	client := NewClaudeClient(apiKey)

	ctx := context.Background()

	req := MessageRequest{
		Model: "claude-3-opus-20240229",
		Messages: []ClaudeMessage{
			{
				Role:    "user",
				Content: "What makes Go a good language for backend development?",
			},
		},
		System: "You are a helpful programming assistant with expertise in Go.",
		MaxTokens: 500,
		Temperature: 0.7,
	}

	resp, err := client.CreateMessage(ctx, req)
	if err != nil {
		fmt.Printf("Error: %v\n", err)
		return
	}

	if len(resp.Content) > 0 && resp.Content[0].Type == "text" {
		fmt.Printf("Response: %s\n", resp.Content[0].Text)
		fmt.Printf("Tokens used: Input: %d, Output: %d\n",
			resp.Usage.InputTokens, resp.Usage.OutputTokens)
	}
}
```

### Error Handling and Rate Limiting

When working with AI APIs, it's important to handle errors and implement rate limiting to avoid exceeding API quotas. Here's a pattern for robust error handling and rate limiting:

```go
package main

import (
	"errors"
	"sync"
	"time"
)

// Common error types for AI API integration
var (
	ErrRateLimited    = errors.New("rate limited")
	ErrInvalidRequest = errors.New("invalid request")
	ErrServerError    = errors.New("server error")
	ErrAuthentication = errors.New("authentication error")
)

// RateLimiter provides rate limiting for API requests
type RateLimiter struct {
	mu            sync.Mutex
	tokens        int
	maxTokens     int
	refillRate    time.Duration
	lastRefillTime time.Time
}

// NewRateLimiter creates a new rate limiter with specified requests per minute
func NewRateLimiter(requestsPerMinute int) *RateLimiter {
	return &RateLimiter{
		tokens:        requestsPerMinute,
		maxTokens:     requestsPerMinute,
		refillRate:    time.Minute / time.Duration(requestsPerMinute),
		lastRefillTime: time.Now(),
	}
}

// Allow checks if a request is allowed and decrements the token count
func (rl *RateLimiter) Allow() bool {
	rl.mu.Lock()
	defer rl.mu.Unlock()

	now := time.Now()

	// Refill tokens based on time elapsed
	elapsed := now.Sub(rl.lastRefillTime)
	tokensToAdd := int(elapsed / rl.refillRate)
	if tokensToAdd > 0 {
		rl.tokens = min(rl.maxTokens, rl.tokens+tokensToAdd)
		rl.lastRefillTime = now
	}

	// Check if we have tokens available
	if rl.tokens > 0 {
		rl.tokens--
		return true
	}

	return false
}

// Wait waits until a token is available and then consumes it
func (rl *RateLimiter) Wait() {
	for {
		if rl.Allow() {
			return
		}
		time.Sleep(rl.refillRate / 10) // Sleep for a fraction of the refill rate
	}
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

This rate limiter can be integrated with any of the API clients shown above to manage request rates. Here's an example of how to use it with the OpenAI client:

```go
type RateLimitedOpenAIClient struct {
	client      *OpenAIClient
	rateLimiter *RateLimiter
}

func NewRateLimitedOpenAIClient(apiKey string, requestsPerMinute int) *RateLimitedOpenAIClient {
	return &RateLimitedOpenAIClient{
		client:      NewOpenAIClient(apiKey),
		rateLimiter: NewRateLimiter(requestsPerMinute),
	}
}

func (c *RateLimitedOpenAIClient) CreateChatCompletion(ctx context.Context, req CompletionRequest) (*CompletionResponse, error) {
	// Wait for rate limit token
	if !c.rateLimiter.Allow() {
		select {
		case <-ctx.Done():
			return nil, fmt.Errorf("context cancelled while waiting for rate limit: %w", ctx.Err())
		default:
			c.rateLimiter.Wait()
		}
	}

	// Call the underlying client
	return c.client.CreateChatCompletion(ctx, req)
}
```

This approach ensures that your application respects API rate limits while still handling requests efficiently.

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
		line, err := reader.ReadString('\n')
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

	resp, err := r.llmClient.CreateChatCompletion(ctx, req)
	if err != nil {
		return "", fmt.Errorf("generating LLM response: %w", err)
	}

	if len(resp.Choices) == 0 {
		return "", fmt.Errorf("no response generated")
	}

	return resp.Choices[0].Message.Content, nil
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
