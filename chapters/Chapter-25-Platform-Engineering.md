# **Chapter 25: Platform Engineering with Go**

## **25.1 Introduction to Platform Engineering**

Platform Engineering represents a paradigm shift in how organizations deliver developer experiences and manage infrastructure. At its core, platform engineering is about creating internal developer platforms (IDPs) that abstract away infrastructure complexity, provide self-service capabilities, and increase developer productivity through standardized tooling and workflows.

Go has emerged as the language of choice for platform engineering due to its:

1. **Efficiency and Performance**: Go produces lightweight, statically-compiled binaries that start quickly and use resources efficiently
2. **Strong Standard Library**: Built-in support for HTTP servers, concurrency, and system operations
3. **Ecosystem Alignment**: The cloud-native ecosystem (Kubernetes, Docker, Terraform) is largely written in Go
4. **Cross-Platform Support**: Single codebase for tools that run across multiple operating systems
5. **Approachable Learning Curve**: Allows platform teams to onboard new engineers quickly

In this chapter, we'll explore how to build platform engineering solutions using Go, focusing on real-world patterns and practices that enable you to create robust, scalable internal developer platforms.

### **25.1.1 The Rise of Platform Engineering**

Platform engineering has grown in response to several challenges:

- Increasing complexity of cloud-native infrastructure
- The need for standardization across development teams
- Security and compliance requirements in regulated industries
- Pressure to accelerate development cycles and time-to-market

Rather than having each development team solve these problems independently, platform engineering teams build shared infrastructure and tooling that embody best practices and organizational standards.

### **25.1.2 Core Components of Developer Platforms**

A comprehensive internal developer platform typically consists of:

1. **Self-Service Portal**: A central interface where developers can provision resources
2. **Service Catalog**: Curated templates for common resources and applications
3. **Infrastructure Automation**: Tools for provisioning and managing infrastructure
4. **CI/CD Pipelines**: Standardized deployment pipelines
5. **Observability Stack**: Monitoring, logging, and tracing infrastructure
6. **Development Environment Management**: Tooling for consistent local development
7. **Security and Compliance Controls**: Automated enforcement of security policies

Throughout this chapter, we'll build components for each of these areas using Go, demonstrating how to create a cohesive platform experience.

### **25.1.3 The Golden Path Approach**

Platform engineering is often associated with the concept of "golden paths" - opinionated, well-documented, and supported paths that make it easy for developers to do the right thing by default. These paths:

- Reduce cognitive load on developers
- Enforce organizational best practices
- Accelerate onboarding of new team members
- Ensure consistency across different applications

We'll explore how to implement golden paths in Go-based platform tools, making the default path both the easiest and the most secure option.

## **25.2 Building a Self-Service Developer Portal**

A self-service developer portal is the central interface of your platform, allowing developers to discover, provision, and manage resources. Building one in Go gives you fine-grained control over performance, security, and integration capabilities.

### **25.2.1 Architecture of a Developer Portal**

A well-designed developer portal typically follows a layered architecture:

1. **Frontend Layer**: Web UI or CLI interface that developers interact with
2. **API Layer**: RESTful or GraphQL API that processes requests
3. **Service Layer**: Business logic handling platform operations
4. **Resource Layer**: Integration with underlying infrastructure providers

Let's design a flexible, modular architecture using Go:

```go
// portal/main.go
package main

import (
	"log"
	"net/http"
	"os"
	"time"

	"github.com/gorilla/mux"
	"github.com/yourorg/devportal/api"
	"github.com/yourorg/devportal/auth"
	"github.com/yourorg/devportal/config"
)

func main() {
	// Load configuration
	cfg, err := config.Load()
	if err != nil {
		log.Fatalf("Failed to load configuration: %v", err)
	}

	// Initialize router
	router := mux.NewRouter()

	// Set up middleware
	router.Use(auth.JWTMiddleware)
	router.Use(api.LoggingMiddleware)
	router.Use(api.RecoveryMiddleware)

	// Register API routes
	api.RegisterRoutes(router, cfg)

	// Configure server
	srv := &http.Server{
		Handler:      router,
		Addr:         cfg.Server.ListenAddress,
		WriteTimeout: 15 * time.Second,
		ReadTimeout:  15 * time.Second,
		IdleTimeout:  60 * time.Second,
	}

	// Start server
	log.Printf("Starting developer portal on %s", cfg.Server.ListenAddress)
	if err := srv.ListenAndServe(); err != nil {
		log.Fatal(err)
	}
}
```

This structure provides a clean separation of concerns and allows for modular development of portal features.

### **25.2.2 Implementing Core API Endpoints**

Let's implement key API endpoints that any developer portal should provide:

```go
// api/routes.go
package api

import (
	"github.com/gorilla/mux"
	"github.com/yourorg/devportal/config"
	"github.com/yourorg/devportal/handlers"
)

func RegisterRoutes(router *mux.Router, cfg *config.Config) {
	// API versioning
	v1 := router.PathPrefix("/api/v1").Subrouter()

	// Service catalog endpoints
	catalog := v1.PathPrefix("/catalog").Subrouter()
	catalog.HandleFunc("/services", handlers.ListServices).Methods("GET")
	catalog.HandleFunc("/services/{id}", handlers.GetService).Methods("GET")
	catalog.HandleFunc("/services/{id}/provision", handlers.ProvisionService).Methods("POST")

	// Environment management
	envs := v1.PathPrefix("/environments").Subrouter()
	envs.HandleFunc("", handlers.ListEnvironments).Methods("GET")
	envs.HandleFunc("/{id}", handlers.GetEnvironment).Methods("GET")
	envs.HandleFunc("/{id}/resources", handlers.ListEnvironmentResources).Methods("GET")

	// Resource management
	resources := v1.PathPrefix("/resources").Subrouter()
	resources.HandleFunc("", handlers.ListResources).Methods("GET")
	resources.HandleFunc("/{id}", handlers.GetResource).Methods("GET")
	resources.HandleFunc("/{id}", handlers.UpdateResource).Methods("PUT")
	resources.HandleFunc("/{id}", handlers.DeleteResource).Methods("DELETE")

	// User management
	users := v1.PathPrefix("/users").Subrouter()
	users.HandleFunc("/me", handlers.GetCurrentUser).Methods("GET")
	users.HandleFunc("/me/permissions", handlers.GetUserPermissions).Methods("GET")

	// Documentation endpoints
	v1.HandleFunc("/docs", handlers.GetDocumentation).Methods("GET")

	// Health check
	router.HandleFunc("/health", handlers.HealthCheck).Methods("GET")
}
```

### **25.2.3 Handling Resource Provisioning**

The most critical function of a developer portal is resource provisioning. Let's implement a flexible provisioning system:

```go
// handlers/provision.go
package handlers

import (
	"encoding/json"
	"fmt"
	"net/http"
	"time"

	"github.com/google/uuid"
	"github.com/gorilla/mux"
	"github.com/yourorg/devportal/models"
	"github.com/yourorg/devportal/provisioners"
)

// ProvisionService handles requests to provision a new service instance
func ProvisionService(w http.ResponseWriter, r *http.Request) {
	// Extract service ID from path
	vars := mux.Vars(r)
	serviceID := vars["id"]

	// Parse request body
	var req models.ProvisionRequest
	if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
		http.Error(w, fmt.Sprintf("Invalid request: %v", err), http.StatusBadRequest)
		return
	}

	// Validate request
	if err := req.Validate(); err != nil {
		http.Error(w, fmt.Sprintf("Validation error: %v", err), http.StatusBadRequest)
		return
	}

	// Get user from context
	user := r.Context().Value("user").(models.User)

	// Create provisioning job
	job := models.ProvisioningJob{
		ID:        uuid.New().String(),
		ServiceID: serviceID,
		Request:   req,
		Status:    models.StatusPending,
		CreatedBy: user.ID,
		CreatedAt: time.Now(),
	}

	// Store job in database
	if err := job.Save(); err != nil {
		http.Error(w, "Failed to create provisioning job", http.StatusInternalServerError)
		return
	}

	// Start async provisioning
	go func() {
		// Get provisioner for service type
		provisioner, err := provisioners.GetProvisioner(serviceID)
		if err != nil {
			job.Status = models.StatusFailed
			job.Error = err.Error()
			job.Save()
			return
		}

		// Execute provisioning
		result, err := provisioner.Provision(req)
		if err != nil {
			job.Status = models.StatusFailed
			job.Error = err.Error()
			job.Save()
			return
		}

		// Update job with success
		job.Status = models.StatusCompleted
		job.ResourceID = result.ResourceID
		job.Outputs = result.Outputs
		job.CompletedAt = time.Now()
		job.Save()
	}()

	// Return job details to client
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusAccepted)
	json.NewEncoder(w).Encode(map[string]string{
		"job_id": job.ID,
		"status": string(job.Status),
	})
}
```

This implementation follows several best practices:

1. **Asynchronous Processing**: Long-running operations happen in the background
2. **Validation**: Requests are validated before processing
3. **Job Tracking**: Provisioning is tracked with a job ID for status updates
4. **Pluggable Provisioners**: Different service types can have different provisioning logic

### **25.2.4 Building a Flexible Provisioner System**

To make our portal extensible, let's implement a pluggable provisioner system:

```go
// provisioners/provisioner.go
package provisioners

import (
	"fmt"

	"github.com/yourorg/devportal/models"
)

// Provisioner defines the interface for service provisioning
type Provisioner interface {
	Provision(req models.ProvisionRequest) (*models.ProvisionResult, error)
	Update(resourceID string, req models.UpdateRequest) (*models.ProvisionResult, error)
	Delete(resourceID string) error
	GetStatus(resourceID string) (models.ResourceStatus, error)
}

// Registry of available provisioners
var provisioners = map[string]Provisioner{}

// RegisterProvisioner adds a provisioner to the registry
func RegisterProvisioner(serviceType string, provisioner Provisioner) {
	provisioners[serviceType] = provisioner
}

// GetProvisioner returns the appropriate provisioner for a service
func GetProvisioner(serviceID string) (Provisioner, error) {
	// Look up service to get its type
	service, err := models.GetServiceByID(serviceID)
	if err != nil {
		return nil, fmt.Errorf("service not found: %v", err)
	}

	// Find provisioner for this service type
	provisioner, exists := provisioners[service.Type]
	if !exists {
		return nil, fmt.Errorf("no provisioner registered for service type: %s", service.Type)
	}

	return provisioner, nil
}
```

Now we can implement concrete provisioners for different infrastructure types:

```go
// provisioners/kubernetes.go
package provisioners

import (
	"context"
	"encoding/json"
	"fmt"

	"github.com/yourorg/devportal/models"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/rest"
)

// KubernetesProvisioner handles Kubernetes resource provisioning
type KubernetesProvisioner struct {
	clientset *kubernetes.Clientset
}

// NewKubernetesProvisioner creates a new Kubernetes provisioner
func NewKubernetesProvisioner(config *rest.Config) (*KubernetesProvisioner, error) {
	clientset, err := kubernetes.NewForConfig(config)
	if err != nil {
		return nil, err
	}

	return &KubernetesProvisioner{
		clientset: clientset,
	}, nil
}

// Provision creates Kubernetes resources
func (p *KubernetesProvisioner) Provision(req models.ProvisionRequest) (*models.ProvisionResult, error) {
	// Extract parameters
	var params struct {
		Namespace   string            `json:"namespace"`
		Name        string            `json:"name"`
		Image       string            `json:"image"`
		Replicas    int32             `json:"replicas"`
		Ports       []int32           `json:"ports"`
		Environment map[string]string `json:"environment"`
	}

	if err := json.Unmarshal(req.Parameters, &params); err != nil {
		return nil, fmt.Errorf("invalid parameters: %v", err)
	}

	// Validate parameters
	if params.Namespace == "" || params.Name == "" || params.Image == "" {
		return nil, fmt.Errorf("namespace, name and image are required")
	}

	// Implementation of Kubernetes resource creation
	// (simplified for brevity)
	// In a real implementation, you would:
	// - Create a deployment
	// - Create a service
	// - Wait for resources to be ready
	// - Return connection details

	// Return result
	return &models.ProvisionResult{
		ResourceID: fmt.Sprintf("%s/%s", params.Namespace, params.Name),
		Outputs: map[string]string{
			"url": fmt.Sprintf("http://%s.%s.svc.cluster.local", params.Name, params.Namespace),
		},
	}, nil
}

// Other required methods implemented similarly...
```

### **25.2.5 Authentication and Authorization**

Security is critical for a developer portal. Let's implement JWT-based authentication and RBAC:

```go
// auth/middleware.go
package auth

import (
	"context"
	"fmt"
	"net/http"
	"strings"

	"github.com/dgrijalva/jwt-go"
	"github.com/yourorg/devportal/models"
)

// JWTMiddleware validates JWT tokens and adds the user to the request context
func JWTMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Skip auth for health check and public endpoints
		if r.URL.Path == "/health" || strings.HasPrefix(r.URL.Path, "/public") {
			next.ServeHTTP(w, r)
			return
		}

		// Extract token from Authorization header
		authHeader := r.Header.Get("Authorization")
		if authHeader == "" || !strings.HasPrefix(authHeader, "Bearer ") {
			http.Error(w, "Authorization header required", http.StatusUnauthorized)
			return
		}

		tokenString := strings.TrimPrefix(authHeader, "Bearer ")

		// Parse and validate token
		token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			// Validate signing method
			if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
				return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
			}

			// Return the secret key used for signing
			return []byte(getJWTSecret()), nil
		})

		if err != nil || !token.Valid {
			http.Error(w, "Invalid or expired token", http.StatusUnauthorized)
			return
		}

		// Extract claims
		claims, ok := token.Claims.(jwt.MapClaims)
		if !ok {
			http.Error(w, "Invalid token claims", http.StatusUnauthorized)
			return
		}

		// Get user from database
		userID, ok := claims["sub"].(string)
		if !ok {
			http.Error(w, "Invalid user ID in token", http.StatusUnauthorized)
			return
		}

		user, err := models.GetUserByID(userID)
		if err != nil {
			http.Error(w, "User not found", http.StatusUnauthorized)
			return
		}

		// Add user to request context
		ctx := context.WithValue(r.Context(), "user", user)

		// Check authorization for the requested resource
		if !authorized(user, r.Method, r.URL.Path) {
			http.Error(w, "Forbidden", http.StatusForbidden)
			return
		}

		// Call the next handler with the updated context
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

// authorized checks if a user has permission for the requested action
func authorized(user models.User, method, path string) bool {
	// Implementation of RBAC logic
	// This would typically check against a permission model in your database
	// Simplified for brevity
	return true
}

// getJWTSecret returns the secret key for JWT signing
func getJWTSecret() string {
	// In production, this would come from a secure configuration or vault
	return "your-secret-key"
}
```

## **25.3 Creating a Service Catalog**

A service catalog is the heart of a developer platform, providing a curated set of services, templates, and resources that developers can consume. Building a well-designed service catalog in Go enables you to create a standardized yet flexible experience.

### **25.3.1 Designing a Service Catalog Model**

Let's define the core models for our service catalog:

```go
// models/catalog.go
package models

import (
	"encoding/json"
	"fmt"
	"time"
)

// ServiceDefinition represents a service template in the catalog
type ServiceDefinition struct {
	ID           string            `json:"id"`
	Name         string            `json:"name"`
	Description  string            `json:"description"`
	Version      string            `json:"version"`
	Type         string            `json:"type"`
	Category     string            `json:"category"`
	Tags         []string          `json:"tags"`
	Icon         string            `json:"icon"`
	Documentation string           `json:"documentation"`
	Parameters   []ParameterDefinition `json:"parameters"`
	Outputs      []OutputDefinition    `json:"outputs"`
	Metadata     map[string]string     `json:"metadata"`
	CreatedAt    time.Time         `json:"created_at"`
	UpdatedAt    time.Time         `json:"updated_at"`
}

// ParameterDefinition describes an input parameter for a service
type ParameterDefinition struct {
	Name         string        `json:"name"`
	DisplayName  string        `json:"display_name"`
	Description  string        `json:"description"`
	Type         string        `json:"type"` // string, number, boolean, object, array
	Default      interface{}   `json:"default,omitempty"`
	Required     bool          `json:"required"`
	Validation   *Validation   `json:"validation,omitempty"`
	Options      []Option      `json:"options,omitempty"` // For select/dropdown parameters
	Conditional  *Conditional  `json:"conditional,omitempty"` // Show parameter based on other values
}

// OutputDefinition describes a service output
type OutputDefinition struct {
	Name        string `json:"name"`
	DisplayName string `json:"display_name"`
	Description string `json:"description"`
	Type        string `json:"type"`
	Sensitive   bool   `json:"sensitive,omitempty"`
}

// Validation defines rules for parameter validation
type Validation struct {
	Pattern    string      `json:"pattern,omitempty"`    // Regex pattern
	Min        *float64    `json:"min,omitempty"`        // Min value for numbers
	Max        *float64    `json:"max,omitempty"`        // Max value for numbers
	MinLength  *int        `json:"min_length,omitempty"` // Min length for strings
	MaxLength  *int        `json:"max_length,omitempty"` // Max length for strings
	Format     string      `json:"format,omitempty"`     // Predefined formats like email, uri, etc.
	Enum       []string    `json:"enum,omitempty"`       // Valid values
}

// Option represents a predefined option for select parameters
type Option struct {
	Value       string `json:"value"`
	DisplayName string `json:"display_name"`
	Description string `json:"description,omitempty"`
}

// Conditional defines when a parameter should be shown
type Conditional struct {
	FieldName  string      `json:"field_name"`
	Operator   string      `json:"operator"`   // equals, not_equals, contains, etc.
	Value      interface{} `json:"value"`
}

// ProvisionRequest represents a request to provision a service
type ProvisionRequest struct {
	Name        string          `json:"name"`
	Description string          `json:"description"`
	ServiceID   string          `json:"service_id"`
	Version     string          `json:"version"`
	Environment string          `json:"environment"`
	Parameters  json.RawMessage `json:"parameters"`
}

// Validate validates the provision request
func (r *ProvisionRequest) Validate() error {
	if r.Name == "" {
		return fmt.Errorf("name is required")
	}
	if r.ServiceID == "" {
		return fmt.Errorf("service_id is required")
	}
	if r.Environment == "" {
		return fmt.Errorf("environment is required")
	}
	if len(r.Parameters) == 0 {
		return fmt.Errorf("parameters are required")
	}
	return nil
}

// ProvisionResult contains the result of a provisioning operation
type ProvisionResult struct {
	ResourceID string            `json:"resource_id"`
	Outputs    map[string]string `json:"outputs"`
}

// ResourceStatus represents the status of a provisioned resource
type ResourceStatus string

const (
	StatusPending    ResourceStatus = "pending"
	StatusInProgress ResourceStatus = "in_progress"
	StatusCompleted  ResourceStatus = "completed"
	StatusFailed     ResourceStatus = "failed"
)
```

### **25.3.2 Implementing Catalog Storage**

We need a storage backend to manage our service catalog. Let's implement a PostgreSQL-based storage:

```go
// storage/catalog.go
package storage

import (
	"context"
	"database/sql"
	"encoding/json"
	"fmt"
	"time"

	"github.com/yourorg/devportal/models"
)

// CatalogRepository handles service catalog data access
type CatalogRepository struct {
	db *sql.DB
}

// NewCatalogRepository creates a new catalog repository
func NewCatalogRepository(db *sql.DB) *CatalogRepository {
	return &CatalogRepository{db: db}
}

// GetServiceByID retrieves a service definition by ID
func (r *CatalogRepository) GetServiceByID(ctx context.Context, id string) (*models.ServiceDefinition, error) {
	query := `
		SELECT id, name, description, version, type, category,
		       tags, icon, documentation, parameters, outputs, metadata,
		       created_at, updated_at
		FROM service_definitions
		WHERE id = $1
	`

	var service models.ServiceDefinition
	var tagsJSON, paramsJSON, outputsJSON, metadataJSON []byte

	err := r.db.QueryRowContext(ctx, query, id).Scan(
		&service.ID,
		&service.Name,
		&service.Description,
		&service.Version,
		&service.Type,
		&service.Category,
		&tagsJSON,
		&service.Icon,
		&service.Documentation,
		&paramsJSON,
		&outputsJSON,
		&metadataJSON,
		&service.CreatedAt,
		&service.UpdatedAt,
	)

	if err != nil {
		if err == sql.ErrNoRows {
			return nil, fmt.Errorf("service not found: %s", id)
		}
		return nil, err
	}

	// Parse JSON fields
	if err := json.Unmarshal(tagsJSON, &service.Tags); err != nil {
		return nil, err
	}

	if err := json.Unmarshal(paramsJSON, &service.Parameters); err != nil {
		return nil, err
	}

	if err := json.Unmarshal(outputsJSON, &service.Outputs); err != nil {
		return nil, err
	}

	if err := json.Unmarshal(metadataJSON, &service.Metadata); err != nil {
		return nil, err
	}

	return &service, nil
}

// ListServices retrieves services with optional filtering
func (r *CatalogRepository) ListServices(ctx context.Context, category string, tags []string) ([]*models.ServiceDefinition, error) {
	query := `
		SELECT id, name, description, version, type, category,
		       tags, icon, documentation, parameters, outputs, metadata,
		       created_at, updated_at
		FROM service_definitions
		WHERE ($1 = '' OR category = $1)
		ORDER BY name
	`

	rows, err := r.db.QueryContext(ctx, query, category)
	if err != nil {
		return nil, err
	}
	defer rows.Close()

	var services []*models.ServiceDefinition

	for rows.Next() {
		var service models.ServiceDefinition
		var tagsJSON, paramsJSON, outputsJSON, metadataJSON []byte

		err := rows.Scan(
			&service.ID,
			&service.Name,
			&service.Description,
			&service.Version,
			&service.Type,
			&service.Category,
			&tagsJSON,
			&service.Icon,
			&service.Documentation,
			&paramsJSON,
			&outputsJSON,
			&metadataJSON,
			&service.CreatedAt,
			&service.UpdatedAt,
		)

		if err != nil {
			return nil, err
		}

		// Parse JSON fields
		if err := json.Unmarshal(tagsJSON, &service.Tags); err != nil {
			return nil, err
		}

		// Filter by tags if specified
		if len(tags) > 0 {
			matches := false
			for _, serviceTag := range service.Tags {
				for _, filterTag := range tags {
					if serviceTag == filterTag {
						matches = true
						break
					}
				}
				if matches {
					break
				}
			}
			if !matches {
				continue
			}
		}

		if err := json.Unmarshal(paramsJSON, &service.Parameters); err != nil {
			return nil, err
		}

		if err := json.Unmarshal(outputsJSON, &service.Outputs); err != nil {
			return nil, err
		}

		if err := json.Unmarshal(metadataJSON, &service.Metadata); err != nil {
			return nil, err
		}

		services = append(services, &service)
	}

	if err := rows.Err(); err != nil {
		return nil, err
	}

	return services, nil
}

// CreateService adds a new service definition to the catalog
func (r *CatalogRepository) CreateService(ctx context.Context, service *models.ServiceDefinition) error {
	// Set timestamps
	now := time.Now()
	service.CreatedAt = now
	service.UpdatedAt = now

	// Marshal JSON fields
	tagsJSON, err := json.Marshal(service.Tags)
	if err != nil {
		return err
	}

	paramsJSON, err := json.Marshal(service.Parameters)
	if err != nil {
		return err
	}

	outputsJSON, err := json.Marshal(service.Outputs)
	if err != nil {
		return err
	}

	metadataJSON, err := json.Marshal(service.Metadata)
	if err != nil {
		return err
	}

	query := `
		INSERT INTO service_definitions (
			id, name, description, version, type, category,
			tags, icon, documentation, parameters, outputs, metadata,
			created_at, updated_at
		) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14)
	`

	_, err = r.db.ExecContext(ctx, query,
		service.ID,
		service.Name,
		service.Description,
		service.Version,
		service.Type,
		service.Category,
		tagsJSON,
		service.Icon,
		service.Documentation,
		paramsJSON,
		outputsJSON,
		metadataJSON,
		service.CreatedAt,
		service.UpdatedAt,
	)

	return err
}

// Additional methods for updating, deleting services...
```

### **25.3.3 Creating Service Templates**

Now, let's create a sample service template for a web application:

```go
// templates/webapp.go
package templates

import (
	"github.com/google/uuid"
	"github.com/yourorg/devportal/models"
)

// CreateWebAppTemplate creates a template for deploying web applications
func CreateWebAppTemplate() *models.ServiceDefinition {
	// Define parameters with validation
	parameters := []models.ParameterDefinition{
		{
			Name:        "name",
			DisplayName: "Application Name",
			Description: "Name of your web application (lowercase, no spaces)",
			Type:        "string",
			Required:    true,
			Validation: &models.Validation{
				Pattern:   "^[a-z0-9]([-a-z0-9]*[a-z0-9])?$",
				MinLength: intPtr(3),
				MaxLength: intPtr(63),
			},
		},
		{
			Name:        "image",
			DisplayName: "Container Image",
			Description: "Docker image for your application",
			Type:        "string",
			Required:    true,
		},
		{
			Name:        "replicas",
			DisplayName: "Replicas",
			Description: "Number of application instances",
			Type:        "number",
			Default:     2,
			Validation: &models.Validation{
				Min: float64Ptr(1),
				Max: float64Ptr(10),
			},
		},
		{
			Name:        "port",
			DisplayName: "Container Port",
			Description: "Port your application listens on",
			Type:        "number",
			Default:     8080,
			Validation: &models.Validation{
				Min: float64Ptr(1),
				Max: float64Ptr(65535),
			},
		},
		{
			Name:        "environment",
			DisplayName: "Environment Variables",
			Description: "Key-value pairs for container environment variables",
			Type:        "object",
			Default:     map[string]string{},
		},
		{
			Name:        "resources",
			DisplayName: "Resource Requirements",
			Description: "CPU and memory resources",
			Type:        "object",
			Default: map[string]interface{}{
				"cpu":    "100m",
				"memory": "128Mi",
			},
		},
		{
			Name:        "expose",
			DisplayName: "Expose Service",
			Description: "Whether to expose the service externally",
			Type:        "boolean",
			Default:     true,
		},
		{
			Name:        "route_type",
			DisplayName: "Routing Type",
			Description: "How to expose the service externally",
			Type:        "string",
			Default:     "ClusterIP",
			Options: []models.Option{
				{Value: "ClusterIP", DisplayName: "Internal Only"},
				{Value: "LoadBalancer", DisplayName: "Load Balancer"},
				{Value: "Ingress", DisplayName: "Ingress with DNS"},
			},
			Conditional: &models.Conditional{
				FieldName: "expose",
				Operator:  "equals",
				Value:     true,
			},
		},
	}

	// Define outputs
	outputs := []models.OutputDefinition{
		{
			Name:        "url",
			DisplayName: "Application URL",
			Description: "URL to access your application",
			Type:        "string",
		},
		{
			Name:        "status",
			DisplayName: "Deployment Status",
			Description: "Current status of your deployment",
			Type:        "string",
		},
	}

	// Create service definition
	return &models.ServiceDefinition{
		ID:            uuid.New().String(),
		Name:          "Web Application",
		Description:   "Deploy a containerized web application on Kubernetes",
		Version:       "1.0.0",
		Type:          "kubernetes",
		Category:      "Applications",
		Tags:          []string{"web", "container", "kubernetes"},
		Icon:          "web-app-icon.svg",
		Documentation: "# Web Application\n\nThis template deploys a containerized web application on Kubernetes...",
		Parameters:    parameters,
		Outputs:       outputs,
		Metadata: map[string]string{
			"provisioner": "kubernetes",
			"complexity":  "medium",
			"cost_tier":   "standard",
		},
	}
}

// Helper functions for pointer types
func intPtr(i int) *int {
	return &i
}

func float64Ptr(f float64) *float64 {
	return &f
}
```

### **25.3.4 Creating a Dynamic Form Generator**

One of the most useful features of a service catalog is dynamically generating forms based on service templates. Let's implement a form generator in Go:

```go
// ui/formgenerator.go
package ui

import (
	"encoding/json"

	"github.com/yourorg/devportal/models"
)

// FormDefinition represents a dynamic form for a service
type FormDefinition struct {
	Title       string       `json:"title"`
	Description string       `json:"description"`
	Fields      []FormField  `json:"fields"`
}

// FormField represents a field in a dynamic form
type FormField struct {
	ID          string      `json:"id"`
	Type        string      `json:"type"`
	Label       string      `json:"label"`
	Description string      `json:"description"`
	Required    bool        `json:"required"`
	Default     interface{} `json:"default,omitempty"`
	Validation  interface{} `json:"validation,omitempty"`
	Options     interface{} `json:"options,omitempty"`
	Condition   interface{} `json:"condition,omitempty"`
}

// GenerateFormDefinition creates a form definition from a service template
func GenerateFormDefinition(service *models.ServiceDefinition) *FormDefinition {
	form := &FormDefinition{
		Title:       service.Name,
		Description: service.Description,
		Fields:      make([]FormField, 0, len(service.Parameters)),
	}

	// Add standard name field
	form.Fields = append(form.Fields, FormField{
		ID:          "resource_name",
		Type:        "string",
		Label:       "Resource Name",
		Description: "Name for this resource instance",
		Required:    true,
		Validation: map[string]interface{}{
			"pattern":    "^[a-z0-9]([-a-z0-9]*[a-z0-9])?$",
			"minLength":  3,
			"maxLength":  63,
		},
	})

	// Convert service parameters to form fields
	for _, param := range service.Parameters {
		field := FormField{
			ID:          param.Name,
			Type:        param.Type,
			Label:       param.DisplayName,
			Description: param.Description,
			Required:    param.Required,
			Default:     param.Default,
		}

		// Convert validation
		if param.Validation != nil {
			validationJSON, _ := json.Marshal(param.Validation)
			json.Unmarshal(validationJSON, &field.Validation)
		}

		// Convert options
		if len(param.Options) > 0 {
			optionsJSON, _ := json.Marshal(param.Options)
			json.Unmarshal(optionsJSON, &field.Options)
		}

		// Convert conditional
		if param.Conditional != nil {
			conditionJSON, _ := json.Marshal(param.Conditional)
			json.Unmarshal(conditionJSON, &field.Condition)
		}

		form.Fields = append(form.Fields, field)
	}

	return form
}

// GenerateFormJSON returns the form definition as JSON
func GenerateFormJSON(service *models.ServiceDefinition) ([]byte, error) {
	form := GenerateFormDefinition(service)
	return json.Marshal(form)
}
```

### **25.3.5 Versioning and Managing Service Catalog Changes**

It's important to handle versioning of service templates properly. Let's implement a version management system:

```go
// versioning/service_versions.go
package versioning

import (
	"context"
	"database/sql"
	"fmt"
	"time"

	"github.com/yourorg/devportal/models"
)

// ServiceVersionRepository manages service template versions
type ServiceVersionRepository struct {
	db *sql.DB
}

// NewServiceVersionRepository creates a new service version repository
func NewServiceVersionRepository(db *sql.DB) *ServiceVersionRepository {
	return &ServiceVersionRepository{db: db}
}

// PublishNewVersion publishes a new version of a service
func (r *ServiceVersionRepository) PublishNewVersion(ctx context.Context, service *models.ServiceDefinition) error {
	// Check if service already exists
	existing, err := r.GetLatestServiceVersion(ctx, service.Name)
	if err != nil && err != sql.ErrNoRows {
		return err
	}

	// If service exists, increment version
	if existing != nil {
		// Parse current version and increment
		newVersion, err := IncrementVersion(existing.Version)
		if err != nil {
			return err
		}
		service.Version = newVersion
	}

	// Save new version (implemented in storage repository)
	// This is a simplified example
	return nil
}

// GetLatestServiceVersion gets the latest version of a service
func (r *ServiceVersionRepository) GetLatestServiceVersion(ctx context.Context, serviceName string) (*models.ServiceDefinition, error) {
	query := `
		SELECT id, name, description, version, type, category,
		       tags, icon, documentation, parameters, outputs, metadata,
		       created_at, updated_at
		FROM service_definitions
		WHERE name = $1
		ORDER BY created_at DESC
		LIMIT 1
	`

	// Implementation would be similar to GetServiceByID
	// Simplified for brevity
	return nil, nil
}

// GetServiceVersion gets a specific version of a service
func (r *ServiceVersionRepository) GetServiceVersion(ctx context.Context, serviceName, version string) (*models.ServiceDefinition, error) {
	query := `
		SELECT id, name, description, version, type, category,
		       tags, icon, documentation, parameters, outputs, metadata,
		       created_at, updated_at
		FROM service_definitions
		WHERE name = $1 AND version = $2
	`

	// Implementation would be similar to GetServiceByID
	// Simplified for brevity
	return nil, nil
}

// ListServiceVersions lists all versions of a service
func (r *ServiceVersionRepository) ListServiceVersions(ctx context.Context, serviceName string) ([]*models.ServiceDefinition, error) {
	query := `
		SELECT id, name, description, version, type, category,
		       tags, icon, documentation, parameters, outputs, metadata,
		       created_at, updated_at
		FROM service_definitions
		WHERE name = $1
		ORDER BY created_at DESC
	`

	// Implementation would be similar to ListServices
	// Simplified for brevity
	return nil, nil
}

// IncrementVersion increments the version string (e.g., 1.0.0 -> 1.0.1)
func IncrementVersion(version string) (string, error) {
	// Implementation for semantic versioning
	// Simplified for brevity
	return "1.0.1", nil
}
```

## **25.4 Infrastructure Automation with Go**

Infrastructure automation is a core component of platform engineering, enabling self-service provisioning and management of resources. Go is exceptionally well-suited for building infrastructure automation tools due to its performance, strong typing, and excellent support for cloud provider APIs.

### **25.4.1 Building a Universal Infrastructure Client**

Let's design a flexible infrastructure client that can work with multiple cloud providers:

```go
// infra/client.go
package infra

import (
	"context"
	"fmt"
)

// ResourceType represents a type of infrastructure resource
type ResourceType string

const (
	ResourceTypeVM         ResourceType = "vm"
	ResourceTypeDatabase   ResourceType = "database"
	ResourceTypeStorage    ResourceType = "storage"
	ResourceTypeKubernetes ResourceType = "kubernetes"
	ResourceTypeNetwork    ResourceType = "network"
)

// Resource represents an infrastructure resource
type Resource struct {
	ID           string                 `json:"id"`
	Name         string                 `json:"name"`
	Type         ResourceType           `json:"type"`
	Provider     string                 `json:"provider"`
	Region       string                 `json:"region"`
	Status       string                 `json:"status"`
	CreatedAt    string                 `json:"created_at"`
	Tags         map[string]string      `json:"tags"`
	Properties   map[string]interface{} `json:"properties"`
	Dependencies []string               `json:"dependencies,omitempty"`
}

// Provider defines the interface for infrastructure providers
type Provider interface {
	// Resource operations
	CreateResource(ctx context.Context, resourceType ResourceType, params map[string]interface{}) (*Resource, error)
	GetResource(ctx context.Context, id string) (*Resource, error)
	UpdateResource(ctx context.Context, id string, params map[string]interface{}) (*Resource, error)
	DeleteResource(ctx context.Context, id string) error
	ListResources(ctx context.Context, resourceType ResourceType, filters map[string]string) ([]*Resource, error)

	// Provider information
	GetProviderName() string
	GetRegions() []string
	GetResourceTypes() []ResourceType
	GetTemplates(resourceType ResourceType) ([]map[string]interface{}, error)
}

// Client provides a unified interface to multiple infrastructure providers
type Client struct {
	providers map[string]Provider
}

// NewClient creates a new infrastructure client
func NewClient() *Client {
	return &Client{
		providers: make(map[string]Provider),
	}
}

// RegisterProvider adds a provider to the client
func (c *Client) RegisterProvider(provider Provider) {
	c.providers[provider.GetProviderName()] = provider
}

// GetProvider returns a provider by name
func (c *Client) GetProvider(name string) (Provider, error) {
	provider, exists := c.providers[name]
	if !exists {
		return nil, fmt.Errorf("provider not found: %s", name)
	}
	return provider, nil
}

// CreateResource creates a resource using the specified provider
func (c *Client) CreateResource(ctx context.Context, providerName string, resourceType ResourceType, params map[string]interface{}) (*Resource, error) {
	provider, err := c.GetProvider(providerName)
	if err != nil {
		return nil, err
	}
	return provider.CreateResource(ctx, resourceType, params)
}

// Additional methods for other resource operations...
```

### **25.4.2 Implementing Cloud Provider Adapters**

With our interface defined, let's implement adapters for different cloud providers:

```go
// infra/aws/provider.go
package aws

import (
	"context"
	"fmt"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/config"
	"github.com/aws/aws-sdk-go-v2/service/ec2"
	"github.com/aws/aws-sdk-go-v2/service/rds"
	"github.com/aws/aws-sdk-go-v2/service/s3"

	"github.com/yourorg/devportal/infra"
)

// AWSProvider implements the Provider interface for AWS
type AWSProvider struct {
	config  aws.Config
	ec2Svc  *ec2.Client
	rdsSvc  *rds.Client
	s3Svc   *s3.Client
}

// NewAWSProvider creates a new AWS provider
func NewAWSProvider(ctx context.Context) (*AWSProvider, error) {
	// Load AWS configuration
	cfg, err := config.LoadDefaultConfig(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to load AWS config: %v", err)
	}

	// Create service clients
	return &AWSProvider{
		config:  cfg,
		ec2Svc:  ec2.NewFromConfig(cfg),
		rdsSvc:  rds.NewFromConfig(cfg),
		s3Svc:   s3.NewFromConfig(cfg),
	}, nil
}

// GetProviderName returns the provider name
func (p *AWSProvider) GetProviderName() string {
	return "aws"
}

// GetRegions returns available AWS regions
func (p *AWSProvider) GetRegions() []string {
	// Return a list of AWS regions
	return []string{
		"us-east-1",
		"us-east-2",
		"us-west-1",
		"us-west-2",
		"eu-west-1",
		"eu-central-1",
		"ap-northeast-1",
		"ap-southeast-1",
		"ap-southeast-2",
	}
}

// GetResourceTypes returns supported resource types
func (p *AWSProvider) GetResourceTypes() []infra.ResourceType {
	return []infra.ResourceType{
		infra.ResourceTypeVM,
		infra.ResourceTypeDatabase,
		infra.ResourceTypeStorage,
		infra.ResourceTypeNetwork,
	}
}

// CreateResource creates a resource in AWS
func (p *AWSProvider) CreateResource(ctx context.Context, resourceType infra.ResourceType, params map[string]interface{}) (*infra.Resource, error) {
	switch resourceType {
	case infra.ResourceTypeVM:
		return p.createEC2Instance(ctx, params)
	case infra.ResourceTypeDatabase:
		return p.createRDSInstance(ctx, params)
	case infra.ResourceTypeStorage:
		return p.createS3Bucket(ctx, params)
	case infra.ResourceTypeNetwork:
		return p.createVPC(ctx, params)
	default:
		return nil, fmt.Errorf("unsupported resource type: %s", resourceType)
	}
}

// Implementation of specific resource creation methods
func (p *AWSProvider) createEC2Instance(ctx context.Context, params map[string]interface{}) (*infra.Resource, error) {
	// Extract parameters
	instanceType, _ := params["instance_type"].(string)
	if instanceType == "" {
		instanceType = "t2.micro" // Default
	}

	ami, _ := params["ami"].(string)
	if ami == "" {
		ami = "ami-0c55b159cbfafe1f0" // Default Amazon Linux 2 AMI
	}

	// Create EC2 instance
	// This is simplified - a real implementation would do much more
	input := &ec2.RunInstancesInput{
		ImageId:      aws.String(ami),
		InstanceType: ec2.InstanceType(instanceType),
		MinCount:     aws.Int32(1),
		MaxCount:     aws.Int32(1),
	}

	result, err := p.ec2Svc.RunInstances(ctx, input)
	if err != nil {
		return nil, fmt.Errorf("failed to create EC2 instance: %v", err)
	}

	// Get the instance ID
	if len(result.Instances) == 0 {
		return nil, fmt.Errorf("no instances created")
	}

	instanceID := *result.Instances[0].InstanceId

	// Create resource object
	resource := &infra.Resource{
		ID:       instanceID,
		Name:     params["name"].(string),
		Type:     infra.ResourceTypeVM,
		Provider: "aws",
		Region:   p.config.Region,
		Status:   "pending",
		Tags:     make(map[string]string),
		Properties: map[string]interface{}{
			"instance_type": instanceType,
			"ami":           ami,
		},
	}

	// Add tags if provided
	if tags, ok := params["tags"].(map[string]string); ok {
		resource.Tags = tags
	}

	return resource, nil
}

// Additional resource creation methods similarly implemented...
```

### **25.4.3 Implementing Infrastructure as Code with Go**

Beyond direct API calls, let's implement a declarative infrastructure as code approach:

```go
// iac/definition.go
package iac

import (
	"context"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"

	"github.com/yourorg/devportal/infra"
)

// ResourceDefinition defines a resource in an IaC template
type ResourceDefinition struct {
	Name         string                 `json:"name"`
	Type         infra.ResourceType     `json:"type"`
	Provider     string                 `json:"provider"`
	Region       string                 `json:"region"`
	Properties   map[string]interface{} `json:"properties"`
	Tags         map[string]string      `json:"tags,omitempty"`
	Dependencies []string               `json:"depends_on,omitempty"`
}

// Template represents an infrastructure template
type Template struct {
	Name        string               `json:"name"`
	Description string               `json:"description"`
	Version     string               `json:"version"`
	Resources   []ResourceDefinition `json:"resources"`
	Outputs     map[string]string    `json:"outputs,omitempty"`
	Variables   map[string]Variable  `json:"variables,omitempty"`
}

// Variable represents a template variable
type Variable struct {
	Description string      `json:"description"`
	Type        string      `json:"type"`
	Default     interface{} `json:"default,omitempty"`
	Required    bool        `json:"required"`
}

// Deployer handles deployment of infrastructure templates
type Deployer struct {
	client *infra.Client
}

// NewDeployer creates a new infrastructure deployer
func NewDeployer(client *infra.Client) *Deployer {
	return &Deployer{
		client: client,
	}
}

// LoadTemplate loads a template from a file
func (d *Deployer) LoadTemplate(path string) (*Template, error) {
	// Read template file
	data, err := ioutil.ReadFile(path)
	if err != nil {
		return nil, fmt.Errorf("failed to read template: %v", err)
	}

	// Parse template
	var template Template
	if err := json.Unmarshal(data, &template); err != nil {
		return nil, fmt.Errorf("failed to parse template: %v", err)
	}

	return &template, nil
}

// Deploy deploys a template with variable values
func (d *Deployer) Deploy(ctx context.Context, template *Template, variables map[string]interface{}) (map[string]interface{}, error) {
	// Validate variables
	if err := d.validateVariables(template, variables); err != nil {
		return nil, err
	}

	// Build dependency graph
	graph := buildDependencyGraph(template.Resources)

	// Deploy resources in dependency order
	resources := make(map[string]*infra.Resource)
	outputs := make(map[string]interface{})

	// Process resources in dependency order
	for _, resourceName := range graph.GetDeploymentOrder() {
		resourceDef := getResourceByName(template.Resources, resourceName)
		if resourceDef == nil {
			return nil, fmt.Errorf("resource not found: %s", resourceName)
		}

		// Resolve dependencies and variable references
		resolvedProps, err := d.resolveProperties(resourceDef.Properties, variables, resources)
		if err != nil {
			return nil, err
		}

		// Create resource
		resource, err := d.client.CreateResource(
			ctx,
			resourceDef.Provider,
			resourceDef.Type,
			resolvedProps,
		)
		if err != nil {
			return nil, fmt.Errorf("failed to create resource %s: %v", resourceName, err)
		}

		// Store resource for dependency resolution
		resources[resourceName] = resource

		// Map outputs
		for outputName, outputPath := range template.Outputs {
			value, err := resolveOutputValue(outputPath, resources)
			if err != nil {
				return nil, err
			}
			outputs[outputName] = value
		}
	}

	return outputs, nil
}

// Helper methods for dependency resolution, validation, etc.
func (d *Deployer) validateVariables(template *Template, variables map[string]interface{}) error {
	// Check that all required variables are provided
	for name, varDef := range template.Variables {
		if varDef.Required {
			if _, exists := variables[name]; !exists {
				return fmt.Errorf("required variable not provided: %s", name)
			}
		}
	}
	return nil
}

func (d *Deployer) resolveProperties(properties map[string]interface{}, variables map[string]interface{}, resources map[string]*infra.Resource) (map[string]interface{}, error) {
	// Deep copy properties to avoid modifying the original
	resolved := make(map[string]interface{})
	for k, v := range properties {
		resolved[k] = v
	}

	// Resolve variable references
	// This is a simplified implementation - a real one would handle nested properties
	for key, value := range resolved {
		if strValue, ok := value.(string); ok {
			// Check if it's a variable reference
			if len(strValue) > 3 && strValue[:2] == "${" && strValue[len(strValue)-1:] == "}" {
				// Extract variable name
				varName := strValue[2 : len(strValue)-1]

				// Check if it's a reference to a resource property
				if resourceRef, ok := resources[varName]; ok {
					resolved[key] = resourceRef.ID
				} else if varValue, ok := variables[varName]; ok {
					// It's a variable reference
					resolved[key] = varValue
				} else {
					return nil, fmt.Errorf("undefined reference: %s", varName)
				}
			}
		}
	}

	return resolved, nil
}

// Additional helper methods...
```

### **25.4.4 Creating a Terraform-like State Manager**

To track infrastructure state, let's implement a state manager:

```go
// iac/state.go
package iac

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"sync"
	"time"

	"github.com/yourorg/devportal/infra"
)

// State represents the current state of deployed infrastructure
type State struct {
	Version     int                    `json:"version"`
	StackName   string                 `json:"stack_name"`
	Resources   map[string]*infra.Resource `json:"resources"`
	Outputs     map[string]interface{} `json:"outputs"`
	LastUpdated time.Time              `json:"last_updated"`
}

// StateManager handles the persistence and retrieval of infrastructure state
type StateManager struct {
	stateDir string
	mutex    sync.Mutex
}

// NewStateManager creates a new state manager
func NewStateManager(stateDir string) (*StateManager, error) {
	// Create state directory if it doesn't exist
	if err := os.MkdirAll(stateDir, 0755); err != nil {
		return nil, fmt.Errorf("failed to create state directory: %v", err)
	}

	return &StateManager{
		stateDir: stateDir,
	}, nil
}

// GetState retrieves the state for a stack
func (m *StateManager) GetState(stackName string) (*State, error) {
	m.mutex.Lock()
	defer m.mutex.Unlock()

	// Build state file path
	statePath := filepath.Join(m.stateDir, fmt.Sprintf("%s.json", stackName))

	// Check if state file exists
	if _, err := os.Stat(statePath); os.IsNotExist(err) {
		// No state file, return empty state
		return &State{
			Version:     1,
			StackName:   stackName,
			Resources:   make(map[string]*infra.Resource),
			Outputs:     make(map[string]interface{}),
			LastUpdated: time.Now(),
		}, nil
	}

	// Read state file
	data, err := ioutil.ReadFile(statePath)
	if err != nil {
		return nil, fmt.Errorf("failed to read state file: %v", err)
	}

	// Parse state
	var state State
	if err := json.Unmarshal(data, &state); err != nil {
		return nil, fmt.Errorf("failed to parse state: %v", err)
	}

	return &state, nil
}

// SaveState saves the state for a stack
func (m *StateManager) SaveState(state *State) error {
	m.mutex.Lock()
	defer m.mutex.Unlock()

	// Update timestamp
	state.LastUpdated = time.Now()

	// Build state file path
	statePath := filepath.Join(m.stateDir, fmt.Sprintf("%s.json", state.StackName))

	// Marshal state to JSON
	data, err := json.MarshalIndent(state, "", "  ")
	if err != nil {
		return fmt.Errorf("failed to marshal state: %v", err)
	}

	// Write state file
	if err := ioutil.WriteFile(statePath, data, 0644); err != nil {
		return fmt.Errorf("failed to write state file: %v", err)
	}

	return nil
}

// ListStacks returns a list of all stack names
func (m *StateManager) ListStacks() ([]string, error) {
	m.mutex.Lock()
	defer m.mutex.Unlock()

	// Read state directory
	files, err := ioutil.ReadDir(m.stateDir)
	if err != nil {
		return nil, fmt.Errorf("failed to read state directory: %v", err)
	}

	// Extract stack names from filenames
	stacks := make([]string, 0, len(files))
	for _, file := range files {
		if !file.IsDir() && filepath.Ext(file.Name()) == ".json" {
			stackName := file.Name()[:len(file.Name())-5] // Remove .json extension
			stacks = append(stacks, stackName)
		}
	}

	return stacks, nil
}
```

## **25.5 Building Platform CLI Tools**

Command-line tools are essential for platform engineers and their developer users. Go excels at creating efficient, cross-platform CLI tools that can be distributed as single binaries. Let's explore how to build effective platform CLI tools in Go.

### **25.5.1 Designing Effective CLI Interfaces**

A well-designed CLI should be intuitive, consistent, and provide appropriate feedback. Let's implement a flexible CLI framework:

```go
// cli/app.go
package cli

import (
	"fmt"
	"os"
	"sort"
	"strings"

	"github.com/fatih/color"
	"github.com/spf13/cobra"
	"github.com/spf13/viper"
)

// App represents the CLI application
type App struct {
	Name        string
	Description string
	Version     string
	RootCmd     *cobra.Command
	Config      *viper.Viper
}

// NewApp creates a new CLI application
func NewApp(name, description, version string) *App {
	app := &App{
		Name:        name,
		Description: description,
		Version:     version,
		Config:      viper.New(),
	}

	// Create root command
	app.RootCmd = &cobra.Command{
		Use:     app.Name,
		Short:   app.Description,
		Version: app.Version,
	}

	// Add global flags
	app.RootCmd.PersistentFlags().StringP("config", "c", "", "config file path")
	app.RootCmd.PersistentFlags().BoolP("verbose", "v", false, "enable verbose output")
	app.RootCmd.PersistentFlags().Bool("no-color", false, "disable colored output")

	// Bind flags to config
	app.Config.BindPFlag("config", app.RootCmd.PersistentFlags().Lookup("config"))
	app.Config.BindPFlag("verbose", app.RootCmd.PersistentFlags().Lookup("verbose"))
	app.Config.BindPFlag("no_color", app.RootCmd.PersistentFlags().Lookup("no-color"))

	// Set up default config locations
	app.Config.SetConfigName(app.Name)
	app.Config.AddConfigPath(".")
	app.Config.AddConfigPath("$HOME/." + app.Name)
	app.Config.AddConfigPath("/etc/" + app.Name)

	return app
}

// Setup initializes the application
func (a *App) Setup() error {
	// Add command execution hooks
	cobra.OnInitialize(func() {
		// Load config
		configPath := a.Config.GetString("config")
		if configPath != "" {
			a.Config.SetConfigFile(configPath)
		}

		if err := a.Config.ReadInConfig(); err != nil {
			if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
				fmt.Fprintf(os.Stderr, "Error reading config: %v\n", err)
			}
		}

		// Configure color output
		if a.Config.GetBool("no_color") {
			color.NoColor = true
		}
	})

	return nil
}

// Run executes the CLI application
func (a *App) Run() error {
	return a.RootCmd.Execute()
}

// AddCommand adds a command to the application
func (a *App) AddCommand(cmd *cobra.Command) {
	a.RootCmd.AddCommand(cmd)
}

// AddCommandGroup adds a group of related commands
func (a *App) AddCommandGroup(groupName string, cmds []*cobra.Command) {
	// Create parent command for the group
	groupCmd := &cobra.Command{
		Use:   strings.ToLower(groupName),
		Short: fmt.Sprintf("%s commands", groupName),
	}

	// Add sub-commands
	for _, cmd := range cmds {
		groupCmd.AddCommand(cmd)
	}

	// Add to root command
	a.RootCmd.AddCommand(groupCmd)
}
```

### **25.5.2 Implementing Common Platform Commands**

Let's implement some common commands for a platform engineering CLI:

```go
// cli/commands/resources.go
package commands

import (
	"fmt"
	"os"
	"text/tabwriter"

	"github.com/spf13/cobra"
	"github.com/yourorg/devportal/client"
)

// NewResourcesCommand creates the resources command group
func NewResourcesCommand(client *client.Client) *cobra.Command {
	cmd := &cobra.Command{
		Use:   "resources",
		Short: "Manage platform resources",
		Long:  "View and manage resources provisioned through the platform",
	}

	// Add sub-commands
	cmd.AddCommand(
		newListResourcesCommand(client),
		newGetResourceCommand(client),
		newDeleteResourceCommand(client),
	)

	return cmd
}

// newListResourcesCommand creates a command to list resources
func newListResourcesCommand(client *client.Client) *cobra.Command {
	var (
		environment string
		resourceType string
		outputFormat string
	)

	cmd := &cobra.Command{
		Use:   "list",
		Short: "List resources",
		RunE: func(cmd *cobra.Command, args []string) error {
			// Get resources from API
			resources, err := client.ListResources(environment, resourceType)
			if err != nil {
				return fmt.Errorf("failed to list resources: %v", err)
			}

			// Format output
			switch outputFormat {
			case "json":
				return outputJSON(resources)
			case "yaml":
				return outputYAML(resources)
			default:
				return outputTable(resources)
			}
		},
	}

	// Add flags
	cmd.Flags().StringVarP(&environment, "environment", "e", "", "filter by environment")
	cmd.Flags().StringVarP(&resourceType, "type", "t", "", "filter by resource type")
	cmd.Flags().StringVarP(&outputFormat, "output", "o", "table", "output format (table, json, yaml)")

	return cmd
}

// outputTable outputs resources in table format
func outputTable(resources []client.Resource) error {
	w := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)
	fmt.Fprintln(w, "NAME\tTYPE\tENVIRONMENT\tSTATUS\tCREATED")

	for _, r := range resources {
		fmt.Fprintf(w, "%s\t%s\t%s\t%s\t%s\n",
			r.Name,
			r.Type,
			r.Environment,
			r.Status,
			r.CreatedAt.Format("2006-01-02 15:04:05"),
		)
	}

	return w.Flush()
}

// Additional command implementations...
```

### **25.5.3 Creating Interactive CLI Features**

Modern CLI tools often provide interactive features for a better user experience. Let's implement some interactive components:

```go
// cli/interactive/prompt.go
package interactive

import (
	"errors"
	"fmt"
	"strings"

	"github.com/AlecAivazis/survey/v2"
)

// PromptOptions contains options for prompts
type PromptOptions struct {
	Message     string
	Default     interface{}
	Help        string
	Required    bool
	Validate    func(interface{}) error
	FileOptions *FileOptions
}

// FileOptions contains options for file prompts
type FileOptions struct {
	Filter      string
	ShowHidden  bool
	AllowFolder bool
}

// AskString prompts for a string value
func AskString(opts PromptOptions) (string, error) {
	prompt := &survey.Input{
		Message: opts.Message,
		Default: fmt.Sprintf("%v", opts.Default),
		Help:    opts.Help,
	}

	var value string
	validationFunc := makeValidationFunc(opts)

	err := survey.AskOne(prompt, &value, survey.WithValidator(validationFunc))
	return value, err
}

// AskPassword prompts for a password
func AskPassword(opts PromptOptions) (string, error) {
	prompt := &survey.Password{
		Message: opts.Message,
		Help:    opts.Help,
	}

	var value string
	validationFunc := makeValidationFunc(opts)

	err := survey.AskOne(prompt, &value, survey.WithValidator(validationFunc))
	return value, err
}

// AskConfirm prompts for a yes/no answer
func AskConfirm(message string, defaultValue bool) (bool, error) {
	prompt := &survey.Confirm{
		Message: message,
		Default: defaultValue,
	}

	var value bool
	err := survey.AskOne(prompt, &value)
	return value, err
}

// AskSelect prompts for a selection from a list
func AskSelect(message string, options []string, defaultValue string) (string, error) {
	if len(options) == 0 {
		return "", errors.New("no options provided")
	}

	defaultIndex := 0
	for i, opt := range options {
		if opt == defaultValue {
			defaultIndex = i
			break
		}
	}

	prompt := &survey.Select{
		Message: message,
		Options: options,
		Default: options[defaultIndex],
	}

	var value string
	err := survey.AskOne(prompt, &value)
	return value, err
}

// AskMultiSelect prompts for multiple selections
func AskMultiSelect(message string, options []string, defaults []string) ([]string, error) {
	if len(options) == 0 {
		return nil, errors.New("no options provided")
	}

	defaultIndices := make([]int, 0, len(defaults))
	for _, def := range defaults {
		for i, opt := range options {
			if opt == def {
				defaultIndices = append(defaultIndices, i)
				break
			}
		}
	}

	prompt := &survey.MultiSelect{
		Message: message,
		Options: options,
		Default: defaultIndices,
	}

	var values []string
	err := survey.AskOne(prompt, &values)
	return values, err
}

// AskEditor opens an editor for multi-line input
func AskEditor(message string, defaultValue string, fileType string) (string, error) {
	prompt := &survey.Editor{
		Message:       message,
		Default:       defaultValue,
		HideDefault:   true,
		AppendDefault: true,
		FileName:      fmt.Sprintf("*.%s", fileType),
	}

	var value string
	err := survey.AskOne(prompt, &value)
	return value, err
}

// makeValidationFunc creates a validation function from options
func makeValidationFunc(opts PromptOptions) survey.Validator {
	return func(val interface{}) error {
		// Check if required
		if opts.Required {
			str, ok := val.(string)
			if !ok || strings.TrimSpace(str) == "" {
				return errors.New("this value is required")
			}
		}

		// Custom validation
		if opts.Validate != nil {
			return opts.Validate(val)
		}

		return nil
	}
}
```

### **25.5.4 Building a Configuration Management System**

Platform CLI tools need robust configuration management. Let's implement a flexible configuration system:

```go
// config/manager.go
package config

import (
	"fmt"
	"os"
	"path/filepath"

	"github.com/spf13/viper"
	"gopkg.in/yaml.v2"
)

// Constants for configuration
const (
	DefaultConfigFileName = "config.yaml"
	DefaultConfigDirName  = ".devportal"
)

// ConfigManager handles configuration loading and saving
type ConfigManager struct {
	viper      *viper.Viper
	configFile string
}

// NewConfigManager creates a new configuration manager
func NewConfigManager() (*ConfigManager, error) {
	v := viper.New()
	v.SetConfigType("yaml")

	// Set up default config directory
	configDir, err := getConfigDir()
	if err != nil {
		return nil, err
	}

	configFile := filepath.Join(configDir, DefaultConfigFileName)

	// Create config directory if it doesn't exist
	if err := os.MkdirAll(configDir, 0755); err != nil {
		return nil, fmt.Errorf("failed to create config directory: %v", err)
	}

	manager := &ConfigManager{
		viper:      v,
		configFile: configFile,
	}

	// Set defaults
	manager.setDefaults()

	// Try to load existing config
	if err := manager.Load(); err != nil {
		// If config doesn't exist, create it
		if os.IsNotExist(err) {
			if err := manager.Save(); err != nil {
				return nil, fmt.Errorf("failed to create default config: %v", err)
			}
		} else {
			return nil, fmt.Errorf("failed to load config: %v", err)
		}
	}

	return manager, nil
}

// Load loads the configuration from disk
func (m *ConfigManager) Load() error {
	// Check if config file exists
	if _, err := os.Stat(m.configFile); os.IsNotExist(err) {
		return err
	}

	// Read config file
	m.viper.SetConfigFile(m.configFile)
	return m.viper.ReadInConfig()
}

// Save saves the configuration to disk
func (m *ConfigManager) Save() error {
	// Create config file if it doesn't exist
	if _, err := os.Stat(m.configFile); os.IsNotExist(err) {
		if _, err := os.Create(m.configFile); err != nil {
			return fmt.Errorf("failed to create config file: %v", err)
		}
	}

	// Write config
	configMap := m.viper.AllSettings()
	configData, err := yaml.Marshal(configMap)
	if err != nil {
		return fmt.Errorf("failed to marshal config: %v", err)
	}

	return os.WriteFile(m.configFile, configData, 0644)
}

// GetString gets a string value from the configuration
func (m *ConfigManager) GetString(key string) string {
	return m.viper.GetString(key)
}

// SetString sets a string value in the configuration
func (m *ConfigManager) SetString(key, value string) {
	m.viper.Set(key, value)
}

// Get gets a value from the configuration
func (m *ConfigManager) Get(key string) interface{} {
	return m.viper.Get(key)
}

// Set sets a value in the configuration
func (m *ConfigManager) Set(key string, value interface{}) {
	m.viper.Set(key, value)
}

// SetDefault sets a default value in the configuration
func (m *ConfigManager) SetDefault(key string, value interface{}) {
	m.viper.SetDefault(key, value)
}

// setDefaults sets default configuration values
func (m *ConfigManager) setDefaults() {
	m.viper.SetDefault("api.url", "https://api.devportal.example.com")
	m.viper.SetDefault("api.timeout", 30)
	m.viper.SetDefault("ui.theme", "light")
	m.viper.SetDefault("ui.output_format", "table")
}

// getConfigDir returns the configuration directory
func getConfigDir() (string, error) {
	homeDir, err := os.UserHomeDir()
	if err != nil {
		return "", fmt.Errorf("failed to get home directory: %v", err)
	}

	return filepath.Join(homeDir, DefaultConfigDirName), nil
}
```

### **25.5.5 Plugin Architecture for Extensibility**

To make our CLI extensible, let's implement a plugin system:

```go
// plugins/manager.go
package plugins

import (
	"fmt"
	"os"
	"path/filepath"
	"plugin"
	"strings"
	"sync"

	"github.com/spf13/cobra"
	"github.com/yourorg/devportal/cli"
)

// Plugin defines the interface that plugins must implement
type Plugin interface {
	// GetName returns the plugin name
	GetName() string

	// GetDescription returns the plugin description
	GetDescription() string

	// GetVersion returns the plugin version
	GetVersion() string

	// Initialize initializes the plugin
	Initialize(app *cli.App) error

	// GetCommands returns the plugin's commands
	GetCommands() []*cobra.Command
}

// PluginManager handles plugin loading and management
type PluginManager struct {
	app      *cli.App
	plugins  map[string]Plugin
	pluginDir string
	mu       sync.RWMutex
}

// NewPluginManager creates a new plugin manager
func NewPluginManager(app *cli.App, pluginDir string) *PluginManager {
	return &PluginManager{
		app:      app,
		plugins:  make(map[string]Plugin),
		pluginDir: pluginDir,
	}
}

// LoadPlugins loads plugins from the plugin directory
func (m *PluginManager) LoadPlugins() error {
	m.mu.Lock()
	defer m.mu.Unlock()

	// Check if plugin directory exists
	if _, err := os.Stat(m.pluginDir); os.IsNotExist(err) {
		if err := os.MkdirAll(m.pluginDir, 0755); err != nil {
			return fmt.Errorf("failed to create plugin directory: %v", err)
		}
		return nil // No plugins to load
	}

	// Find plugin files
	files, err := os.ReadDir(m.pluginDir)
	if err != nil {
		return fmt.Errorf("failed to read plugin directory: %v", err)
	}

	// Load each plugin
	for _, file := range files {
		if file.IsDir() || !strings.HasSuffix(file.Name(), ".so") {
			continue
		}

		pluginPath := filepath.Join(m.pluginDir, file.Name())
		if err := m.LoadPlugin(pluginPath); err != nil {
			fmt.Fprintf(os.Stderr, "Failed to load plugin %s: %v\n", file.Name(), err)
			continue
		}
	}

	return nil
}

// LoadPlugin loads a plugin from a file
func (m *PluginManager) LoadPlugin(path string) error {
	// Open plugin
	plug, err := plugin.Open(path)
	if err != nil {
		return fmt.Errorf("failed to open plugin: %v", err)
	}

	// Look up plugin symbol
	sym, err := plug.Lookup("Plugin")
	if err != nil {
		return fmt.Errorf("plugin does not export Plugin symbol: %v", err)
	}

	// Assert that loaded symbol is a Plugin
	var p Plugin
	var ok bool
	if p, ok = sym.(Plugin); !ok {
		return fmt.Errorf("plugin does not implement Plugin interface")
	}

	// Initialize plugin
	if err := p.Initialize(m.app); err != nil {
		return fmt.Errorf("failed to initialize plugin: %v", err)
	}

	// Register plugin commands
	commands := p.GetCommands()
	for _, cmd := range commands {
		m.app.AddCommand(cmd)
	}

	// Store plugin
	m.plugins[p.GetName()] = p

	return nil
}

// GetPlugin returns a plugin by name
func (m *PluginManager) GetPlugin(name string) (Plugin, bool) {
	m.mu.RLock()
	defer m.mu.RUnlock()

	plugin, exists := m.plugins[name]
	return plugin, exists
}

// ListPlugins returns a list of loaded plugins
func (m *PluginManager) ListPlugins() []Plugin {
	m.mu.RLock()
	defer m.mu.RUnlock()

	plugins := make([]Plugin, 0, len(m.plugins))
	for _, p := range m.plugins {
		plugins = append(plugins, p)
	}

	return plugins
}
```

## **25.6 Implementing GitOps Workflows**

GitOps is a core practice in modern platform engineering, providing a declarative approach to infrastructure and application deployment through Git workflows. Let's explore how to implement GitOps patterns in Go.

### **25.6.1 Understanding GitOps Principles**

GitOps centers around several key principles:

1. **Declarative Configuration**: All system configurations are expressed declaratively
2. **Version Controlled**: All configuration is stored in Git
3. **Automated Synchronization**: Changes are automatically applied to the environment
4. **Continuous Verification**: System state is continuously verified against desired state

Let's implement a GitOps engine in Go that follows these principles:

```go
// gitops/engine.go
package gitops

import (
	"context"
	"fmt"
	"os"
	"path/filepath"
	"time"

	"github.com/go-git/go-git/v5"
	"github.com/go-git/go-git/v5/plumbing"
	"github.com/go-git/go-git/v5/plumbing/transport/http"
	"gopkg.in/yaml.v2"

	"github.com/yourorg/devportal/infra"
)

// RepositoryConfig holds configuration for a Git repository
type RepositoryConfig struct {
	URL        string `yaml:"url"`
	Branch     string `yaml:"branch"`
	Path       string `yaml:"path"`
	SecretName string `yaml:"secret_name"`
}

// EngineConfig holds configuration for the GitOps engine
type EngineConfig struct {
	Repositories []RepositoryConfig `yaml:"repositories"`
	PollInterval time.Duration     `yaml:"poll_interval"`
	WorkDir      string            `yaml:"work_dir"`
}

// Engine implements a GitOps reconciliation engine
type Engine struct {
	config        EngineConfig
	infraClient   *infra.Client
	secretManager *SecretManager
	stopCh        chan struct{}
}

// NewEngine creates a new GitOps engine
func NewEngine(config EngineConfig, infraClient *infra.Client, secretManager *SecretManager) *Engine {
	return &Engine{
		config:        config,
		infraClient:   infraClient,
		secretManager: secretManager,
		stopCh:        make(chan struct{}),
	}
}

// Start starts the GitOps engine
func (e *Engine) Start(ctx context.Context) error {
	// Create work directory if it doesn't exist
	if err := os.MkdirAll(e.config.WorkDir, 0755); err != nil {
		return fmt.Errorf("failed to create work directory: %v", err)
	}

	// Clone all repositories
	for _, repo := range e.config.Repositories {
		repoPath := filepath.Join(e.config.WorkDir, filepath.Base(repo.URL))
		if err := e.cloneRepository(ctx, repo, repoPath); err != nil {
			return fmt.Errorf("failed to clone repository %s: %v", repo.URL, err)
		}
	}

	// Start reconciliation loop
	go e.reconciliationLoop(ctx)

	return nil
}

// Stop stops the GitOps engine
func (e *Engine) Stop() {
	close(e.stopCh)
}

// reconciliationLoop continuously reconciles the desired state with the current state
func (e *Engine) reconciliationLoop(ctx context.Context) {
	ticker := time.NewTicker(e.config.PollInterval)
	defer ticker.Stop()

	for {
		select {
		case <-ticker.C:
			for _, repo := range e.config.Repositories {
				repoPath := filepath.Join(e.config.WorkDir, filepath.Base(repo.URL))
				if err := e.reconcileRepository(ctx, repo, repoPath); err != nil {
					fmt.Fprintf(os.Stderr, "Failed to reconcile repository %s: %v\n", repo.URL, err)
				}
			}
		case <-e.stopCh:
			return
		case <-ctx.Done():
			return
		}
	}
}

// cloneRepository clones a Git repository
func (e *Engine) cloneRepository(ctx context.Context, repo RepositoryConfig, path string) error {
	// Check if repository already exists
	if _, err := os.Stat(path); !os.IsNotExist(err) {
		return nil
	}

	// Get authentication if needed
	var auth *http.BasicAuth
	if repo.SecretName != "" {
		secret, err := e.secretManager.GetSecret(ctx, repo.SecretName)
		if err != nil {
			return fmt.Errorf("failed to get secret: %v", err)
		}

		auth = &http.BasicAuth{
			Username: secret["username"],
			Password: secret["password"],
		}
	}

	// Clone repository
	_, err := git.PlainClone(path, false, &git.CloneOptions{
		URL:           repo.URL,
		ReferenceName: plumbing.NewBranchReferenceName(repo.Branch),
		SingleBranch:  true,
		Auth:          auth,
	})

	return err
}

// reconcileRepository updates a repository and applies changes
func (e *Engine) reconcileRepository(ctx context.Context, repo RepositoryConfig, path string) error {
	// Open repository
	r, err := git.PlainOpen(path)
	if err != nil {
		return fmt.Errorf("failed to open repository: %v", err)
	}

	// Get worktree
	w, err := r.Worktree()
	if err != nil {
		return fmt.Errorf("failed to get worktree: %v", err)
	}

	// Get authentication if needed
	var auth *http.BasicAuth
	if repo.SecretName != "" {
		secret, err := e.secretManager.GetSecret(ctx, repo.SecretName)
		if err != nil {
			return fmt.Errorf("failed to get secret: %v", err)
		}

		auth = &http.BasicAuth{
			Username: secret["username"],
			Password: secret["password"],
		}
	}

	// Pull latest changes
	err = w.Pull(&git.PullOptions{
		RemoteName:    "origin",
		ReferenceName: plumbing.NewBranchReferenceName(repo.Branch),
		Auth:          auth,
	})

	if err != nil && err != git.NoErrAlreadyUpToDate {
		return fmt.Errorf("failed to pull repository: %v", err)
	}

	// Apply configuration
	return e.applyConfiguration(ctx, repo, path)
}

// applyConfiguration applies configuration from a repository
func (e *Engine) applyConfiguration(ctx context.Context, repo RepositoryConfig, repoPath string) error {
	// Build path to configuration directory
	configPath := filepath.Join(repoPath, repo.Path)

	// Walk configuration directory
	return filepath.Walk(configPath, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}

		// Skip directories
		if info.IsDir() {
			return nil
		}

		// Skip non-YAML files
		if filepath.Ext(path) != ".yaml" && filepath.Ext(path) != ".yml" {
			return nil
		}

		// Apply configuration file
		return e.applyConfigurationFile(ctx, path)
	})
}

// applyConfigurationFile applies a single configuration file
func (e *Engine) applyConfigurationFile(ctx context.Context, path string) error {
	// Read configuration file
	data, err := os.ReadFile(path)
	if err != nil {
		return fmt.Errorf("failed to read file: %v", err)
	}

	// Parse configuration
	var config map[string]interface{}
	if err := yaml.Unmarshal(data, &config); err != nil {
		return fmt.Errorf("failed to parse configuration: %v", err)
	}

	// Get resource type and provider
	resourceType, ok := config["type"].(string)
	if !ok {
		return fmt.Errorf("missing resource type in configuration")
	}

	provider, ok := config["provider"].(string)
	if !ok {
		return fmt.Errorf("missing provider in configuration")
	}

	// Get resource properties
	properties, ok := config["properties"].(map[string]interface{})
	if !ok {
		return fmt.Errorf("missing properties in configuration")
	}

	// Get resource name
	name, ok := config["name"].(string)
	if !ok {
		return fmt.Errorf("missing name in configuration")
	}

	// Check if resource already exists
	resources, err := e.infraClient.ListResources(ctx, provider, infra.ResourceType(resourceType), map[string]string{
		"name": name,
	})
	if err != nil {
		return fmt.Errorf("failed to list resources: %v", err)
	}

	// If resource exists, update it
	if len(resources) > 0 {
		resourceID := resources[0].ID
		_, err := e.infraClient.UpdateResource(ctx, provider, resourceID, properties)
		if err != nil {
			return fmt.Errorf("failed to update resource: %v", err)
		}
		fmt.Printf("Updated resource %s\n", name)
	} else {
		// Otherwise, create it
		_, err := e.infraClient.CreateResource(ctx, provider, infra.ResourceType(resourceType), properties)
		if err != nil {
			return fmt.Errorf("failed to create resource: %v", err)
		}
		fmt.Printf("Created resource %s\n", name)
	}

	return nil
}
```

### **25.6.2 Implementing Drift Detection**

A key aspect of GitOps is detecting and reconciling drift between the desired state (Git) and the actual state (infrastructure). Let's implement drift detection:

```go
// gitops/drift.go
package gitops

import (
	"context"
	"fmt"
	"os"
	"path/filepath"
	"reflect"

	"gopkg.in/yaml.v2"

	"github.com/yourorg/devportal/infra"
)

// DriftDetector detects and reports drift between desired and actual state
type DriftDetector struct {
	infraClient *infra.Client
}

// NewDriftDetector creates a new drift detector
func NewDriftDetector(infraClient *infra.Client) *DriftDetector {
	return &DriftDetector{
		infraClient: infraClient,
	}
}

// DetectDrift detects drift for a repository
func (d *DriftDetector) DetectDrift(ctx context.Context, repo RepositoryConfig, repoPath string) ([]DriftReport, error) {
	var reports []DriftReport

	// Build path to configuration directory
	configPath := filepath.Join(repoPath, repo.Path)

	// Walk configuration directory
	err := filepath.Walk(configPath, func(path string, info os.FileInfo, err error) error {
		if err != nil {
			return err
		}

		// Skip directories
		if info.IsDir() {
			return nil
		}

		// Skip non-YAML files
		if filepath.Ext(path) != ".yaml" && filepath.Ext(path) != ".yml" {
			return nil
		}

		// Detect drift for file
		report, err := d.detectDriftForFile(ctx, path)
		if err != nil {
			return err
		}

		reports = append(reports, report)
		return nil
	})

	return reports, err
}

// DriftReport represents a drift detection report
type DriftReport struct {
	ResourceName string
	ResourceType string
	Provider     string
	Path         string
	HasDrift     bool
	DriftDetails []DriftDetail
}

// DriftDetail represents a specific drift detail
type DriftDetail struct {
	Path     string
	Expected interface{}
	Actual   interface{}
}

// detectDriftForFile detects drift for a single file
func (d *DriftDetector) detectDriftForFile(ctx context.Context, path string) (DriftReport, error) {
	// Read configuration file
	data, err := os.ReadFile(path)
	if err != nil {
		return DriftReport{}, fmt.Errorf("failed to read file: %v", err)
	}

	// Parse configuration
	var config map[string]interface{}
	if err := yaml.Unmarshal(data, &config); err != nil {
		return DriftReport{}, fmt.Errorf("failed to parse configuration: %v", err)
	}

	// Get resource type and provider
	resourceType, ok := config["type"].(string)
	if !ok {
		return DriftReport{}, fmt.Errorf("missing resource type in configuration")
	}

	provider, ok := config["provider"].(string)
	if !ok {
		return DriftReport{}, fmt.Errorf("missing provider in configuration")
	}

	// Get resource properties
	desiredProperties, ok := config["properties"].(map[string]interface{})
	if !ok {
		return DriftReport{}, fmt.Errorf("missing properties in configuration")
	}

	// Get resource name
	name, ok := config["name"].(string)
	if !ok {
		return DriftReport{}, fmt.Errorf("missing name in configuration")
	}

	// Initialize report
	report := DriftReport{
		ResourceName: name,
		ResourceType: resourceType,
		Provider:     provider,
		Path:         path,
		HasDrift:     false,
		DriftDetails: make([]DriftDetail, 0),
	}

	// Check if resource exists
	resources, err := d.infraClient.ListResources(ctx, provider, infra.ResourceType(resourceType), map[string]string{
		"name": name,
	})
	if err != nil {
		return report, fmt.Errorf("failed to list resources: %v", err)
	}

	// If resource doesn't exist, report drift
	if len(resources) == 0 {
		report.HasDrift = true
		report.DriftDetails = append(report.DriftDetails, DriftDetail{
			Path:     "",
			Expected: "resource exists",
			Actual:   "resource does not exist",
		})
		return report, nil
	}

	// Get actual properties
	resource := resources[0]
	actualProperties := resource.Properties

	// Compare properties
	driftDetails := compareProperties("", desiredProperties, actualProperties)
	if len(driftDetails) > 0 {
		report.HasDrift = true
		report.DriftDetails = driftDetails
	}

	return report, nil
}

// compareProperties compares two property maps recursively
func compareProperties(path string, desired, actual map[string]interface{}) []DriftDetail {
	var driftDetails []DriftDetail

	// Check for missing properties in actual
	for key, desiredValue := range desired {
		propertyPath := key
		if path != "" {
			propertyPath = fmt.Sprintf("%s.%s", path, key)
		}

		actualValue, exists := actual[key]
		if !exists {
			driftDetails = append(driftDetails, DriftDetail{
				Path:     propertyPath,
				Expected: desiredValue,
				Actual:   nil,
			})
			continue
		}

		// If both are maps, recurse
		desiredMap, desiredIsMap := desiredValue.(map[string]interface{})
		actualMap, actualIsMap := actualValue.(map[string]interface{})
		if desiredIsMap && actualIsMap {
			nestedDrift := compareProperties(propertyPath, desiredMap, actualMap)
			driftDetails = append(driftDetails, nestedDrift...)
			continue
		}

		// Otherwise, compare values
		if !reflect.DeepEqual(desiredValue, actualValue) {
			driftDetails = append(driftDetails, DriftDetail{
				Path:     propertyPath,
				Expected: desiredValue,
				Actual:   actualValue,
			})
		}
	}

	// Check for extra properties in actual (not in desired)
	for key, actualValue := range actual {
		propertyPath := key
		if path != "" {
			propertyPath = fmt.Sprintf("%s.%s", path, key)
		}

		if _, exists := desired[key]; !exists {
			driftDetails = append(driftDetails, DriftDetail{
				Path:     propertyPath,
				Expected: nil,
				Actual:   actualValue,
			})
		}
	}

	return driftDetails
}
```

### **25.6.3 Building a GitOps Operator**

To implement GitOps at scale, let's create a Kubernetes operator that can manage multiple GitOps workflows:

```go
// operator/controller.go
package operator

import (
	"context"
	"fmt"
	"time"

	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/util/wait"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/util/workqueue"

	"github.com/yourorg/devportal/gitops"
	"github.com/yourorg/devportal/infra"
)

// GitOpsController reconciles GitOps resources
type GitOpsController struct {
	kubeClient    *kubernetes.Clientset
	infraClient   *infra.Client
	secretManager *gitops.SecretManager
	informer      cache.SharedIndexInformer
	queue         workqueue.RateLimitingInterface
	engines       map[string]*gitops.Engine
}

// NewGitOpsController creates a new GitOps controller
func NewGitOpsController(
	kubeClient *kubernetes.Clientset,
	infraClient *infra.Client,
	secretManager *gitops.SecretManager,
	informer cache.SharedIndexInformer,
) *GitOpsController {
	controller := &GitOpsController{
		kubeClient:    kubeClient,
		infraClient:   infraClient,
		secretManager: secretManager,
		informer:      informer,
		queue:         workqueue.NewNamedRateLimitingQueue(workqueue.DefaultControllerRateLimiter(), "gitops"),
		engines:       make(map[string]*gitops.Engine),
	}

	// Set up event handlers
	informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc:    controller.enqueueGitOps,
		UpdateFunc: func(old, new interface{}) { controller.enqueueGitOps(new) },
		DeleteFunc: controller.handleDelete,
	})

	return controller
}

// Run starts the controller
func (c *GitOpsController) Run(ctx context.Context, workers int) error {
	defer c.queue.ShutDown()

	// Start the informer
	go c.informer.Run(ctx.Done())

	// Wait for the cache to sync
	if !cache.WaitForCacheSync(ctx.Done(), c.informer.HasSynced) {
		return fmt.Errorf("failed to sync cache")
	}

	// Start worker goroutines
	for i := 0; i < workers; i++ {
		go wait.Until(c.runWorker, time.Second, ctx.Done())
	}

	<-ctx.Done()
	return nil
}

// runWorker processes items from the queue
func (c *GitOpsController) runWorker() {
	for c.processNextItem() {
	}
}

// processNextItem processes the next item from the queue
func (c *GitOpsController) processNextItem() bool {
	// Get next item from queue
	key, shutdown := c.queue.Get()
	if shutdown {
		return false
	}
	defer c.queue.Done(key)

	// Process item
	err := c.syncGitOps(key.(string))
	if err == nil {
		c.queue.Forget(key)
		return true
	}

	// Handle error
	fmt.Printf("Error syncing GitOps %s: %v\n", key, err)
	c.queue.AddRateLimited(key)
	return true
}

// enqueueGitOps adds a GitOps resource to the queue
func (c *GitOpsController) enqueueGitOps(obj interface{}) {
	key, err := cache.MetaNamespaceKeyFunc(obj)
	if err != nil {
		fmt.Printf("Failed to get key for object: %v\n", err)
		return
	}
	c.queue.Add(key)
}

// handleDelete handles deletion of a GitOps resource
func (c *GitOpsController) handleDelete(obj interface{}) {
	key, err := cache.DeletionHandlingMetaNamespaceKeyFunc(obj)
	if err != nil {
		fmt.Printf("Failed to get key for deleted object: %v\n", err)
		return
	}

	// Stop the engine for this GitOps resource
	c.stopEngine(key)

	// Remove from queue
	c.queue.Forget(key)
}

// syncGitOps reconciles a GitOps resource
func (c *GitOpsController) syncGitOps(key string) error {
	// Get namespace and name from key
	namespace, name, err := cache.SplitMetaNamespaceKey(key)
	if err != nil {
		return fmt.Errorf("invalid key: %s", key)
	}

	// Get GitOps resource
	obj, exists, err := c.informer.GetIndexer().GetByKey(key)
	if err != nil {
		return fmt.Errorf("failed to get GitOps resource: %v", err)
	}

	// If resource doesn't exist, stop engine
	if !exists {
		c.stopEngine(key)
		return nil
	}

	// Convert to GitOps resource
	gitOps := obj.(*GitOpsResource)

	// Check if engine exists
	engine, exists := c.engines[key]
	if !exists {
		// Create new engine
		engine, err = c.createEngine(gitOps)
		if err != nil {
			return fmt.Errorf("failed to create engine: %v", err)
		}
		c.engines[key] = engine
	} else {
		// Update engine if configuration changed
		if c.engineConfigChanged(engine, gitOps) {
			c.stopEngine(key)
			engine, err = c.createEngine(gitOps)
			if err != nil {
				return fmt.Errorf("failed to create engine: %v", err)
			}
			c.engines[key] = engine
		}
	}

	return nil
}

// createEngine creates a new GitOps engine for a resource
func (c *GitOpsController) createEngine(gitOps *GitOpsResource) (*gitops.Engine, error) {
	// Convert to engine config
	config := gitops.EngineConfig{
		Repositories: make([]gitops.RepositoryConfig, len(gitOps.Spec.Repositories)),
		PollInterval: time.Duration(gitOps.Spec.PollIntervalSeconds) * time.Second,
		WorkDir:      fmt.Sprintf("/tmp/gitops/%s/%s", gitOps.Namespace, gitOps.Name),
	}

	for i, repo := range gitOps.Spec.Repositories {
		config.Repositories[i] = gitops.RepositoryConfig{
			URL:        repo.URL,
			Branch:     repo.Branch,
			Path:       repo.Path,
			SecretName: repo.SecretName,
		}
	}

	// Create engine
	engine := gitops.NewEngine(config, c.infraClient, c.secretManager)

	// Start engine
	if err := engine.Start(context.Background()); err != nil {
		return nil, fmt.Errorf("failed to start engine: %v", err)
	}

	return engine, nil
}

// stopEngine stops a GitOps engine
func (c *GitOpsController) stopEngine(key string) {
	engine, exists := c.engines[key]
	if exists {
		engine.Stop()
		delete(c.engines, key)
	}
}

// engineConfigChanged checks if the engine configuration has changed
func (c *GitOpsController) engineConfigChanged(engine *gitops.Engine, gitOps *GitOpsResource) bool {
	// In a real implementation, this would compare the engine's configuration
	// with the GitOps resource to detect changes
	return false
}

// GitOpsResource represents a GitOps resource
type GitOpsResource struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`
	Spec              GitOpsSpec `json:"spec"`
}

// GitOpsSpec represents the specification for a GitOps resource
type GitOpsSpec struct {
	Repositories        []RepositorySpec `json:"repositories"`
	PollIntervalSeconds int              `json:"pollIntervalSeconds"`
}

// RepositorySpec represents a Git repository configuration
type RepositorySpec struct {
	URL        string `json:"url"`
	Branch     string `json:"branch"`
	Path       string `json:"path"`
	SecretName string `json:"secretName,omitempty"`
}

// DeepCopyObject implements the runtime.Object interface
func (g *GitOpsResource) DeepCopyObject() runtime.Object {
	// Implement deep copy logic
	return g
}
```

## **25.7 Developing a Platform-as-Code SDK**

To enable developers to interact programmatically with your platform, let's design a Platform-as-Code (PaC) SDK in Go. This SDK will provide a clean, idiomatic interface for developers to provision and manage resources.

### **25.7.1 Architecture of a Platform SDK**

The SDK should follow a clean, layered architecture:

```go
// sdk/client.go
package sdk

import (
	"context"
	"fmt"
	"net/http"
	"time"

	"github.com/yourorg/devportal/sdk/auth"
	"github.com/yourorg/devportal/sdk/resources"
)

// Client is the main entry point for the Platform SDK
type Client struct {
	baseURL    string
	httpClient *http.Client
	auth       auth.Provider
	Resources  *resources.Client
}

// ClientOptions contains options for the SDK client
type ClientOptions struct {
	BaseURL     string
	Timeout     time.Duration
	AuthType    string
	AuthOptions map[string]string
}

// NewClient creates a new SDK client
func NewClient(opts ClientOptions) (*Client, error) {
	// Set default timeout
	if opts.Timeout == 0 {
		opts.Timeout = 30 * time.Second
	}

	// Create HTTP client
	httpClient := &http.Client{
		Timeout: opts.Timeout,
	}

	// Create auth provider
	authProvider, err := auth.NewProvider(opts.AuthType, opts.AuthOptions)
	if err != nil {
		return nil, fmt.Errorf("failed to create auth provider: %v", err)
	}

	// Create client
	client := &Client{
		baseURL:    opts.BaseURL,
		httpClient: httpClient,
		auth:       authProvider,
	}

	// Initialize resource clients
	client.Resources = resources.NewClient(client)

	return client, nil
}

// Do executes an authenticated HTTP request
func (c *Client) Do(req *http.Request) (*http.Response, error) {
	// Add authentication
	if err := c.auth.Authenticate(req); err != nil {
		return nil, fmt.Errorf("authentication failed: %v", err)
	}

	// Execute request
	return c.httpClient.Do(req)
}
```

### **25.7.2 Authentication Providers**

The SDK should support multiple authentication methods:

```go
// sdk/auth/provider.go
package auth

import (
	"fmt"
	"net/http"
)

// Provider defines the interface for authentication providers
type Provider interface {
	// Authenticate adds authentication to a request
	Authenticate(req *http.Request) error
}

// NewProvider creates a new authentication provider
func NewProvider(authType string, options map[string]string) (Provider, error) {
	switch authType {
	case "api_key":
		return NewAPIKeyProvider(options)
	case "oauth":
		return NewOAuthProvider(options)
	case "token":
		return NewTokenProvider(options)
	default:
		return nil, fmt.Errorf("unsupported auth type: %s", authType)
	}
}

// APIKeyProvider implements API key authentication
type APIKeyProvider struct {
	apiKey    string
	headerKey string
}

// NewAPIKeyProvider creates a new API key provider
func NewAPIKeyProvider(options map[string]string) (*APIKeyProvider, error) {
	apiKey, ok := options["api_key"]
	if !ok {
		return nil, fmt.Errorf("api_key option is required")
	}

	headerKey := options["header_key"]
	if headerKey == "" {
		headerKey = "X-API-Key"
	}

	return &APIKeyProvider{
		apiKey:    apiKey,
		headerKey: headerKey,
	}, nil
}

// Authenticate adds API key authentication to a request
func (p *APIKeyProvider) Authenticate(req *http.Request) error {
	req.Header.Set(p.headerKey, p.apiKey)
	return nil
}

// TokenProvider implements token authentication
type TokenProvider struct {
	token string
}

// NewTokenProvider creates a new token provider
func NewTokenProvider(options map[string]string) (*TokenProvider, error) {
	token, ok := options["token"]
	if !ok {
		return nil, fmt.Errorf("token option is required")
	}

	return &TokenProvider{
		token: token,
	}, nil
}

// Authenticate adds token authentication to a request
func (p *TokenProvider) Authenticate(req *http.Request) error {
	req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", p.token))
	return nil
}

// OAuthProvider implements OAuth authentication
type OAuthProvider struct {
	clientID     string
	clientSecret string
	tokenURL     string
	token        string
	expiry       time.Time
}

// NewOAuthProvider creates a new OAuth provider
func NewOAuthProvider(options map[string]string) (*OAuthProvider, error) {
	clientID, ok := options["client_id"]
	if !ok {
		return nil, fmt.Errorf("client_id option is required")
	}

	clientSecret, ok := options["client_secret"]
	if !ok {
		return nil, fmt.Errorf("client_secret option is required")
	}

	tokenURL, ok := options["token_url"]
	if !ok {
		return nil, fmt.Errorf("token_url option is required")
	}

	return &OAuthProvider{
		clientID:     clientID,
		clientSecret: clientSecret,
		tokenURL:     tokenURL,
	}, nil
}

// Authenticate adds OAuth authentication to a request
func (p *OAuthProvider) Authenticate(req *http.Request) error {
	// Check if token is expired
	if p.token == "" || time.Now().After(p.expiry) {
		if err := p.refreshToken(); err != nil {
			return err
		}
	}

	// Add token to request
	req.Header.Set("Authorization", fmt.Sprintf("Bearer %s", p.token))
	return nil
}

// refreshToken refreshes the OAuth token
func (p *OAuthProvider) refreshToken() error {
	// Implementation of OAuth token refresh
	// This would make a request to the token URL with client credentials
	// and update the token and expiry time
	return nil
}
```

### **25.7.3 Resource Management**

Let's implement the resources component of the SDK:

```go
// sdk/resources/client.go
package resources

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"net/http"
	"net/url"
	"strings"
	"time"
)

// Client provides methods for resource management
type Client struct {
	client interface {
		Do(req *http.Request) (*http.Response, error)
	}
	baseURL string
}

// NewClient creates a new resources client
func NewClient(client interface{ Do(req *http.Request) (*http.Response, error) }) *Client {
	return &Client{
		client: client,
	}
}

// Resource represents a platform resource
type Resource struct {
	ID          string                 `json:"id"`
	Name        string                 `json:"name"`
	Type        string                 `json:"type"`
	Provider    string                 `json:"provider"`
	Environment string                 `json:"environment"`
	Status      string                 `json:"status"`
	Properties  map[string]interface{} `json:"properties"`
	CreatedAt   time.Time              `json:"created_at"`
	UpdatedAt   time.Time              `json:"updated_at"`
}

// ResourceSpec represents a resource specification for creation or update
type ResourceSpec struct {
	Name        string                 `json:"name"`
	Type        string                 `json:"type"`
	Provider    string                 `json:"provider"`
	Environment string                 `json:"environment"`
	Properties  map[string]interface{} `json:"properties"`
}

// List lists resources with optional filtering
func (c *Client) List(ctx context.Context, environment, resourceType string) ([]Resource, error) {
	// Build URL
	queryParams := url.Values{}
	if environment != "" {
		queryParams.Add("environment", environment)
	}
	if resourceType != "" {
		queryParams.Add("type", resourceType)
	}

	url := fmt.Sprintf("/api/v1/resources?%s", queryParams.Encode())

	// Create request
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %v", err)
	}

	// Execute request
	resp, err := c.client.Do(req)
	if err != nil {
		return nil, fmt.Errorf("failed to execute request: %v", err)
	}
	defer resp.Body.Close()

	// Check response status
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
	}

	// Parse response
	var resources []Resource
	if err := json.NewDecoder(resp.Body).Decode(&resources); err != nil {
		return nil, fmt.Errorf("failed to parse response: %v", err)
	}

	return resources, nil
}

// Get gets a resource by ID
func (c *Client) Get(ctx context.Context, id string) (*Resource, error) {
	// Build URL
	url := fmt.Sprintf("/api/v1/resources/%s", id)

	// Create request
	req, err := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %v", err)
	}

	// Execute request
	resp, err := c.client.Do(req)
	if err != nil {
		return nil, fmt.Errorf("failed to execute request: %v", err)
	}
	defer resp.Body.Close()

	// Check response status
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
	}

	// Parse response
	var resource Resource
	if err := json.NewDecoder(resp.Body).Decode(&resource); err != nil {
		return nil, fmt.Errorf("failed to parse response: %v", err)
	}

	return &resource, nil
}

// Create creates a new resource
func (c *Client) Create(ctx context.Context, spec ResourceSpec) (*Resource, error) {
	// Validate specification
	if err := validateResourceSpec(spec); err != nil {
		return nil, err
	}

	// Build URL
	url := "/api/v1/resources"

	// Marshal specification
	data, err := json.Marshal(spec)
	if err != nil {
		return nil, fmt.Errorf("failed to marshal specification: %v", err)
	}

	// Create request
	req, err := http.NewRequestWithContext(ctx, http.MethodPost, url, bytes.NewReader(data))
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %v", err)
	}
	req.Header.Set("Content-Type", "application/json")

	// Execute request
	resp, err := c.client.Do(req)
	if err != nil {
		return nil, fmt.Errorf("failed to execute request: %v", err)
	}
	defer resp.Body.Close()

	// Check response status
	if resp.StatusCode != http.StatusCreated {
		return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
	}

	// Parse response
	var resource Resource
	if err := json.NewDecoder(resp.Body).Decode(&resource); err != nil {
		return nil, fmt.Errorf("failed to parse response: %v", err)
	}

	return &resource, nil
}

// Update updates an existing resource
func (c *Client) Update(ctx context.Context, id string, spec ResourceSpec) (*Resource, error) {
	// Build URL
	url := fmt.Sprintf("/api/v1/resources/%s", id)

	// Marshal specification
	data, err := json.Marshal(spec)
	if err != nil {
		return nil, fmt.Errorf("failed to marshal specification: %v", err)
	}

	// Create request
	req, err := http.NewRequestWithContext(ctx, http.MethodPut, url, bytes.NewReader(data))
	if err != nil {
		return nil, fmt.Errorf("failed to create request: %v", err)
	}
	req.Header.Set("Content-Type", "application/json")

	// Execute request
	resp, err := c.client.Do(req)
	if err != nil {
		return nil, fmt.Errorf("failed to execute request: %v", err)
	}
	defer resp.Body.Close()

	// Check response status
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("unexpected status code: %d", resp.StatusCode)
	}

	// Parse response
	var resource Resource
	if err := json.NewDecoder(resp.Body).Decode(&resource); err != nil {
		return nil, fmt.Errorf("failed to parse response: %v", err)
	}

	return &resource, nil
}

// Delete deletes a resource
func (c *Client) Delete(ctx context.Context, id string) error {
	// Build URL
	url := fmt.Sprintf("/api/v1/resources/%s", id)

	// Create request
	req, err := http.NewRequestWithContext(ctx, http.MethodDelete, url, nil)
	if err != nil {
		return fmt.Errorf("failed to create request: %v", err)
	}

	// Execute request
	resp, err := c.client.Do(req)
	if err != nil {
		return fmt.Errorf("failed to execute request: %v", err)
	}
	defer resp.Body.Close()

	// Check response status
	if resp.StatusCode != http.StatusNoContent {
		return fmt.Errorf("unexpected status code: %d", resp.StatusCode)
	}

	return nil
}

// validateResourceSpec validates a resource specification
func validateResourceSpec(spec ResourceSpec) error {
	if spec.Name == "" {
		return fmt.Errorf("name is required")
	}
	if spec.Type == "" {
		return fmt.Errorf("type is required")
	}
	if spec.Provider == "" {
		return fmt.Errorf("provider is required")
	}
	if spec.Environment == "" {
		return fmt.Errorf("environment is required")
	}
	return nil
}
```

### **25.7.4 Using the SDK**

Let's demonstrate how developers would use our Platform-as-Code SDK:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"time"

	"github.com/yourorg/devportal/sdk"
	"github.com/yourorg/devportal/sdk/resources"
)

func main() {
	// Create SDK client
	client, err := sdk.NewClient(sdk.ClientOptions{
		BaseURL:  "https://platform.example.com",
		Timeout:  30 * time.Second,
		AuthType: "api_key",
		AuthOptions: map[string]string{
			"api_key": os.Getenv("PLATFORM_API_KEY"),
		},
	})
	if err != nil {
		log.Fatalf("Failed to create client: %v", err)
	}

	// Create a new database resource
	spec := resources.ResourceSpec{
		Name:        "my-database",
		Type:        "database",
		Provider:    "aws",
		Environment: "staging",
		Properties: map[string]interface{}{
			"engine":       "postgres",
			"version":      "13.4",
			"instance_class": "db.t3.small",
			"storage_gb":   50,
			"master_username": "admin",
			"backup_retention_days": 7,
		},
	}

	// Create the resource
	ctx := context.Background()
	resource, err := client.Resources.Create(ctx, spec)
	if err != nil {
		log.Fatalf("Failed to create resource: %v", err)
	}

	fmt.Printf("Created resource: %s (%s)\n", resource.Name, resource.ID)

	// Wait for the resource to be ready
	for resource.Status != "ready" {
		time.Sleep(10 * time.Second)

		resource, err = client.Resources.Get(ctx, resource.ID)
		if err != nil {
			log.Fatalf("Failed to get resource: %v", err)
		}

		fmt.Printf("Resource status: %s\n", resource.Status)
	}

	fmt.Printf("Resource is ready: %s\n", resource.Name)
	fmt.Printf("Connection string: %s\n", resource.Properties["connection_string"])
}
```

## **25.8 Conclusion and Best Practices**

In this chapter, we've explored how to use Go to build comprehensive platform engineering solutions. We've covered essential components like self-service portals, service catalogs, infrastructure automation, CLI tools, GitOps workflows, and platform SDKs.

### **25.8.1 Key Takeaways**

1. **Embrace Go's Strengths**: Go's performance, simplicity, and concurrency model make it ideal for platform engineering tools.

2. **Design for Developer Experience**: Great platforms prioritize developer experience through self-service, automation, and intuitive interfaces.

3. **Follow GitOps Principles**: Implement declarative, version-controlled infrastructure with automated synchronization.

4. **Build Modular Components**: Create loosely coupled components that can evolve independently.

5. **Provide Multiple Interfaces**: Support both UI and programmatic access through APIs, CLIs, and SDKs.

### **25.8.2 Best Practices for Platform Engineering**

1. **Standardize on Common Patterns**: Use consistent design patterns across platform components.

2. **Implement Robust Observability**: Add comprehensive logging, metrics, and tracing to all components.

3. **Design for Multi-tenancy**: Account for multiple teams and segregation of resources.

4. **Automate Everything**: Remove manual steps from all workflows.

5. **Implement Progressive Delivery**: Use feature flags and canary releases for platform components.

6. **Document Extensively**: Create comprehensive documentation for all platform capabilities.

7. **Build Security In**: Implement security controls at every layer of the platform.

### **25.8.3 Evolving Your Platform**

Platform engineering is a journey, not a destination. As you build out your platform:

1. **Start Small**: Begin with high-value components that address immediate pain points.

2. **Gather Feedback**: Create tight feedback loops with your developers.

3. **Measure Success**: Define and track platform adoption metrics.

4. **Continuously Improve**: Regularly refine your platform based on real-world usage.

5. **Build a Community**: Foster a community of practice around your platform.

By leveraging Go's strengths and following these principles, you can build a robust, scalable platform that accelerates your organization's software delivery capabilities. The tools and patterns we've explored in this chapter provide a solid foundation for building a modern, developer-friendly platform.

In the next chapter, we'll explore how to design event-driven architectures with Go, building on many of the concepts we've covered here.
