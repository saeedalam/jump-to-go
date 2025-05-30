# **Chapter 30: Enterprise-Grade Security in Go**

## **30.1 Introduction to Security in Enterprise Go Applications**

Enterprise applications face unique security challenges due to their complex nature, scale, and the sensitive data they often process. Go's design principles—simplicity, readability, and efficiency—provide an excellent foundation for building secure applications, but achieving enterprise-grade security requires careful consideration and implementation of additional measures.

This chapter explores comprehensive security strategies for Go applications in enterprise environments, covering everything from secure coding practices to advanced threat mitigation techniques. We'll provide practical, code-driven approaches to enhance your application's security posture across the entire software development lifecycle.

### **30.1.1 The Security Mindset for Go Developers**

Security is not a feature but a continuous process that begins with the right mindset. Enterprise Go developers should approach security with these key principles:

1. **Defense in Depth**: Implement multiple layers of security controls throughout your application
2. **Principle of Least Privilege**: Provide only the minimal access rights necessary for any component or user
3. **Secure by Default**: Design systems to be secure in their default configuration
4. **Fail Securely**: Ensure that when failures occur, they don't compromise security
5. **Security as Code**: Treat security configurations and policies as code that can be versioned, tested, and automated

Go's standard library and ecosystem provide numerous tools to implement these principles, but leveraging them effectively requires understanding the specific security requirements and threats relevant to enterprise applications.

### **30.1.2 Enterprise Security Requirements and Compliance**

Enterprise applications often need to comply with various regulations and standards, such as:

- **GDPR**: For processing personal data of EU citizens
- **HIPAA**: For healthcare applications handling patient information
- **PCI DSS**: For applications processing payment card data
- **SOC 2**: For service organizations managing customer data
- **ISO 27001**: For implementing information security management systems

Go applications in these environments must implement controls to address:

1. **Data Protection**: Securing data at rest and in transit
2. **Authentication and Authorization**: Verifying identities and managing access rights
3. **Audit and Logging**: Recording security-relevant events for compliance and investigation
4. **Vulnerability Management**: Identifying and addressing security weaknesses
5. **Incident Response**: Detecting and responding to security incidents

Throughout this chapter, we'll explore how to implement these controls effectively in Go, starting with secure coding practices and progressing to more advanced security measures.

## **30.2 Secure Coding Practices in Go**

Go's design inherently prevents certain classes of vulnerabilities, such as buffer overflows and null pointer dereferences, but it doesn't eliminate all security risks. This section covers essential secure coding practices specific to Go.

### **30.2.1 Input Validation and Sanitization**

Never trust external input. All data from users, APIs, and even databases should be validated and sanitized before use.

#### **Validating Structured Data**

For structured data validation, libraries like `go-playground/validator` provide a powerful approach:

```go
package main

import (
	"fmt"
	"net/http"

	"github.com/go-playground/validator/v10"
)

// UserRegistration represents user input for registration
type UserRegistration struct {
	Username  string `validate:"required,alphanum,min=4,max=50"`
	Email     string `validate:"required,email"`
	Password  string `validate:"required,min=8,max=100"`
	Age       int    `validate:"required,gte=18"`
	AgreeToS  bool   `validate:"required,eq=true"`
	IPAddress string `validate:"required,ip"`
}

var validate = validator.New()

func registerHandler(w http.ResponseWriter, r *http.Request) {
	// Parse request body into struct
	var user UserRegistration
	// ... (parsing omitted for brevity)

	// Validate the struct
	err := validate.Struct(user)
	if err != nil {
		// Handle validation errors
		validationErrors := err.(validator.ValidationErrors)
		http.Error(w, fmt.Sprintf("Invalid input: %v", validationErrors), http.StatusBadRequest)
		return
	}

	// Proceed with registration
	// ...
}
```

#### **Handling Unstructured Input**

For unstructured or user-generated content, additional sanitization is necessary:

```go
package main

import (
	"html/template"
	"net/http"

	"github.com/microcosm-cc/bluemonday"
)

// SanitizeHTML removes potentially dangerous HTML
func SanitizeHTML(input string) string {
	p := bluemonday.UGCPolicy() // User Generated Content policy
	return p.Sanitize(input)
}

func displayUserContent(w http.ResponseWriter, r *http.Request) {
	userInput := r.FormValue("content")

	// Sanitize the user input
	sanitizedContent := SanitizeHTML(userInput)

	// Safely render the content
	safeContent := template.HTML(sanitizedContent)

	tmpl := template.Must(template.New("content").Parse(`
		<html><body>
			<h1>User Content</h1>
			<div>{{.}}</div>
		</body></html>
	`))

	tmpl.Execute(w, safeContent)
}
```

### **30.2.2 Preventing SQL Injection**

SQL injection remains one of the most common vulnerabilities. Go's database/sql package provides parameterized queries to prevent SQL injection:

```go
package main

import (
	"database/sql"
	"net/http"

	_ "github.com/lib/pq"
)

func getUserData(w http.ResponseWriter, r *http.Request) {
	userID := r.URL.Query().Get("id")

	// BAD: Vulnerable to SQL injection
	// query := "SELECT id, name, email FROM users WHERE id = " + userID
	// rows, err := db.Query(query)

	// GOOD: Use parameterized queries
	rows, err := db.Query("SELECT id, name, email FROM users WHERE id = $1", userID)
	if err != nil {
		http.Error(w, "Database error", http.StatusInternalServerError)
		return
	}
	defer rows.Close()

	// Process results
	// ...
}
```

For more complex queries or to enforce a clear separation between SQL and code, consider using query builders like `squirrel`:

```go
package main

import (
	"net/http"

	"github.com/Masterminds/squirrel"
	_ "github.com/lib/pq"
)

func searchUsers(w http.ResponseWriter, r *http.Request) {
	name := r.URL.Query().Get("name")
	department := r.URL.Query().Get("department")
	active := r.URL.Query().Get("active")

	// Build query safely with conditions
	query := squirrel.Select("id", "name", "email").
		From("users").
		PlaceholderFormat(squirrel.Dollar)

	if name != "" {
		query = query.Where(squirrel.Like{"name": "%" + name + "%"})
	}

	if department != "" {
		query = query.Where(squirrel.Eq{"department": department})
	}

	if active == "true" {
		query = query.Where(squirrel.Eq{"active": true})
	}

	// Generate SQL and arguments
	sql, args, err := query.ToSql()
	if err != nil {
		http.Error(w, "Query error", http.StatusInternalServerError)
		return
	}

	// Execute the query safely
	rows, err := db.Query(sql, args...)
	// ...
}
```

### **30.2.3 Preventing Cross-Site Scripting (XSS)**

Go's html/template package automatically escapes content to prevent XSS attacks:

```go
package main

import (
	"html/template"
	"net/http"
)

func renderPage(w http.ResponseWriter, r *http.Request) {
	// User input that might contain malicious script
	userInput := r.URL.Query().Get("message")

	// Go automatically escapes the content when using html/template
	tmpl := template.Must(template.New("page").Parse(`
		<html><body>
			<h1>Message</h1>
			<p>{{.}}</p>
		</body></html>
	`))

	// The template engine will escape the userInput
	tmpl.Execute(w, userInput)
}
```

When you need to include trusted HTML content, use template.HTML type explicitly:

```go
package main

import (
	"html/template"
	"net/http"
)

type PageData struct {
	Title      string
	RawContent template.HTML
}

func renderArticle(w http.ResponseWriter, r *http.Request) {
	// IMPORTANT: Only use template.HTML for content you control or have sanitized
	trustedHTML := template.HTML("<strong>This is bold text</strong>")

	data := PageData{
		Title:      "Article Title",
		RawContent: trustedHTML,
	}

	tmpl := template.Must(template.New("article").Parse(`
		<html><body>
			<h1>{{.Title}}</h1>
			<div>{{.RawContent}}</div>
		</body></html>
	`))

	tmpl.Execute(w, data)
}
```

### **30.2.4 Secure File Operations**

File operations can introduce security vulnerabilities if not handled properly:

```go
package main

import (
	"io"
	"net/http"
	"os"
	"path/filepath"
	"strings"
)

func downloadFile(w http.ResponseWriter, r *http.Request) {
	// Get requested filename from URL parameter
	requestedFile := r.URL.Query().Get("filename")

	// INSECURE: Direct use of user input in file paths
	// file, err := os.Open(requestedFile)

	// SECURE: Validate and sanitize file path
	if !isValidFilename(requestedFile) {
		http.Error(w, "Invalid filename", http.StatusBadRequest)
		return
	}

	// Use filepath.Clean to resolve any ".." or other path tricks
	cleanPath := filepath.Clean(filepath.Join("./downloads", requestedFile))

	// Ensure the cleaned path is still within the intended directory
	absDownloadDir, _ := filepath.Abs("./downloads")
	absFilePath, _ := filepath.Abs(cleanPath)
	if !strings.HasPrefix(absFilePath, absDownloadDir) {
		http.Error(w, "Access denied", http.StatusForbidden)
		return
	}

	// Now safely open the file
	file, err := os.Open(cleanPath)
	if err != nil {
		http.Error(w, "File not found", http.StatusNotFound)
		return
	}
	defer file.Close()

	// Set appropriate headers and serve the file
	w.Header().Set("Content-Disposition", "attachment; filename="+filepath.Base(cleanPath))
	w.Header().Set("Content-Type", "application/octet-stream")
	io.Copy(w, file)
}

func isValidFilename(filename string) bool {
	// Implement validation logic
	return filename != "" &&
		!strings.Contains(filename, "/") &&
		!strings.Contains(filename, "\\") &&
		!strings.Contains(filename, "..")
}
```

### **30.2.5 Proper Error Handling and Logging**

Secure error handling prevents information leakage while providing necessary debugging information:

```go
package main

import (
	"errors"
	"fmt"
	"log"
	"net/http"

	"github.com/google/uuid"
)

// AppError represents an application error
type AppError struct {
	ID      string // Unique error ID for tracing
	Message string // User-facing message
	Error   error  // Original error
}

func handleRequest(w http.ResponseWriter, r *http.Request) {
	// Attempt some operation
	result, err := performOperation(r)
	if err != nil {
		// Generate unique error ID
		errorID := uuid.New().String()

		// Log detailed error for debugging
		log.Printf("Error ID %s: %v", errorID, err)

		// Return sanitized error to user
		appErr := AppError{
			ID:      errorID,
			Message: "An error occurred while processing your request",
			Error:   err,
		}

		// In production, never expose internal error details
		w.Header().Set("Content-Type", "application/json")
		w.WriteHeader(http.StatusInternalServerError)
		fmt.Fprintf(w, `{"error": "%s", "error_id": "%s"}`, appErr.Message, appErr.ID)
		return
	}

	// Handle successful result
	// ...
}

func performOperation(r *http.Request) (interface{}, error) {
	// Simulate an operation that might fail
	return nil, errors.New("database connection failed: invalid credentials for user app_user")
}
```

This approach:

1. Generates a unique ID for each error
2. Logs detailed error information for debugging
3. Returns only sanitized information to the user
4. Provides an error ID for support teams to trace issues

## **30.3 Authentication and Authorization in Go Applications**

Enterprise applications require robust authentication and authorization systems to control access to resources and functionality. This section explores implementing secure authentication and authorization in Go applications.

### **30.3.1 Authentication Strategies**

Authentication verifies a user's identity. Here are several approaches for Go applications:

#### **Password-Based Authentication**

When implementing password-based authentication, follow these best practices:

```go
package main

import (
	"database/sql"
	"errors"
	"net/http"

	"golang.org/x/crypto/bcrypt"
)

// User represents an application user
type User struct {
	ID           int64
	Email        string
	PasswordHash string
}

// AuthenticateUser verifies credentials and returns user if valid
func AuthenticateUser(email, password string) (*User, error) {
	// Find user by email
	user, err := findUserByEmail(email)
	if err != nil {
		if errors.Is(err, sql.ErrNoRows) {
			// Don't reveal that the email doesn't exist
			return nil, errors.New("invalid credentials")
		}
		return nil, err
	}

	// Compare password with hash
	err = bcrypt.CompareHashAndPassword([]byte(user.PasswordHash), []byte(password))
	if err != nil {
		// Don't reveal that the password is wrong
		return nil, errors.New("invalid credentials")
	}

	return user, nil
}

// RegisterUser creates a new user with a hashed password
func RegisterUser(email, password string) (*User, error) {
	// Validate password strength
	if !isStrongPassword(password) {
		return nil, errors.New("password does not meet security requirements")
	}

	// Hash the password
	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	if err != nil {
		return nil, err
	}

	// Create user in database
	user := &User{
		Email:        email,
		PasswordHash: string(hashedPassword),
	}

	// Save user to database
	// ...

	return user, nil
}

func isStrongPassword(password string) bool {
	// Implement password strength validation
	// - Minimum length
	// - Character diversity
	// - Not on common password lists
	// ...
	return true
}
```

#### **Multi-Factor Authentication (MFA)**

For sensitive applications, implement multi-factor authentication:

```go
package main

import (
	"crypto/rand"
	"encoding/base32"
	"net/http"
	"time"

	"github.com/pquerna/otp/totp"
)

// GenerateTOTPSecret creates a new secret for TOTP authentication
func GenerateTOTPSecret(userID string) (string, error) {
	// Generate random bytes
	bytes := make([]byte, 20)
	_, err := rand.Read(bytes)
	if err != nil {
		return "", err
	}

	// Encode as base32 for TOTP
	secret := base32.StdEncoding.EncodeToString(bytes)

	// Store the secret securely for the user
	// ...

	return secret, nil
}

// ValidateTOTP verifies a TOTP code
func ValidateTOTP(userID, code string) (bool, error) {
	// Retrieve the user's secret
	secret, err := getUserTOTPSecret(userID)
	if err != nil {
		return false, err
	}

	// Validate the TOTP code
	valid := totp.Validate(code, secret)

	return valid, nil
}

// MFA handler example
func mfaHandler(w http.ResponseWriter, r *http.Request) {
	user := getCurrentUser(r)
	totpCode := r.FormValue("totp_code")

	// Verify TOTP code
	valid, err := ValidateTOTP(user.ID, totpCode)
	if err != nil {
		http.Error(w, "Error validating code", http.StatusInternalServerError)
		return
	}

	if !valid {
		http.Error(w, "Invalid authentication code", http.StatusUnauthorized)
		return
	}

	// Code is valid, complete authentication
	// ...
}
```

#### **OAuth 2.0 and OpenID Connect**

For enterprise applications, integrating with identity providers via OAuth 2.0 and OpenID Connect is common:

```go
package main

import (
	"context"
	"crypto/rand"
	"encoding/base64"
	"net/http"
	"time"

	"golang.org/x/oauth2"
	"golang.org/x/oauth2/github" // Example provider
)

// OAuth configuration
var oauthConfig = &oauth2.Config{
	ClientID:     "your-client-id",
	ClientSecret: "your-client-secret",
	RedirectURL:  "https://your-app.com/callback",
	Scopes:       []string{"user:email"},
	Endpoint:     github.Endpoint,
}

// generateStateToken creates a random state token to prevent CSRF
func generateStateToken() (string, error) {
	b := make([]byte, 32)
	_, err := rand.Read(b)
	if err != nil {
		return "", err
	}
	return base64.StdEncoding.EncodeToString(b), nil
}

// Login initiates the OAuth flow
func loginHandler(w http.ResponseWriter, r *http.Request) {
	// Generate and store the state parameter
	state, err := generateStateToken()
	if err != nil {
		http.Error(w, "Server error", http.StatusInternalServerError)
		return
	}

	// Store state in secure session
	session, _ := store.Get(r, "auth-session")
	session.Values["oauth_state"] = state
	session.Values["oauth_expiry"] = time.Now().Add(10 * time.Minute).Unix()
	session.Save(r, w)

	// Redirect to OAuth provider
	url := oauthConfig.AuthCodeURL(state)
	http.Redirect(w, r, url, http.StatusTemporaryRedirect)
}

// Callback handles the OAuth callback
func callbackHandler(w http.ResponseWriter, r *http.Request) {
	// Get state from session
	session, _ := store.Get(r, "auth-session")
	expectedState, ok := session.Values["oauth_state"].(string)
	if !ok {
		http.Error(w, "Invalid session state", http.StatusBadRequest)
		return
	}

	// Verify state parameter to prevent CSRF
	actualState := r.URL.Query().Get("state")
	if actualState != expectedState {
		http.Error(w, "Invalid state parameter", http.StatusBadRequest)
		return
	}

	// Check expiry
	expiry, ok := session.Values["oauth_expiry"].(int64)
	if !ok || time.Now().Unix() > expiry {
		http.Error(w, "State token expired", http.StatusBadRequest)
		return
	}

	// Exchange authorization code for token
	code := r.URL.Query().Get("code")
	token, err := oauthConfig.Exchange(context.Background(), code)
	if err != nil {
		http.Error(w, "Failed to exchange token", http.StatusInternalServerError)
		return
	}

	// Use the token to get user information
	// ...

	// Create or update user, create session, etc.
	// ...
}
```

### **30.3.2 JWT-Based Authentication**

JSON Web Tokens (JWT) are commonly used for stateless authentication in Go applications:

```go
package main

import (
	"errors"
	"net/http"
	"strings"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

// JWT signing key (use a strong, randomly generated key in production)
var jwtKey = []byte("your-secret-key")

// Claims represents the JWT claims
type Claims struct {
	UserID string `json:"user_id"`
	Role   string `json:"role"`
	jwt.RegisteredClaims
}

// GenerateJWT creates a new JWT token for a user
func GenerateJWT(userID, role string) (string, error) {
	// Set expiration time
	expirationTime := time.Now().Add(24 * time.Hour)

	// Create claims
	claims := &Claims{
		UserID: userID,
		Role:   role,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(expirationTime),
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			NotBefore: jwt.NewNumericDate(time.Now()),
			Issuer:    "your-application",
			Subject:   userID,
		},
	}

	// Create token with claims
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

	// Sign and get the complete encoded token as a string
	tokenString, err := token.SignedString(jwtKey)

	return tokenString, err
}

// ValidateJWT validates a JWT token and returns the claims
func ValidateJWT(tokenString string) (*Claims, error) {
	// Parse and validate the token
	claims := &Claims{}
	token, err := jwt.ParseWithClaims(
		tokenString,
		claims,
		func(token *jwt.Token) (interface{}, error) {
			// Validate signing method
			if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
				return nil, errors.New("unexpected signing method")
			}
			return jwtKey, nil
		},
	)

	if err != nil {
		return nil, err
	}

	if !token.Valid {
		return nil, errors.New("invalid token")
	}

	return claims, nil
}

// AuthMiddleware is a middleware that validates JWT tokens
func AuthMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Get token from Authorization header
		authHeader := r.Header.Get("Authorization")
		if authHeader == "" {
			http.Error(w, "Authorization header required", http.StatusUnauthorized)
			return
		}

		// Extract the token
		tokenParts := strings.Split(authHeader, " ")
		if len(tokenParts) != 2 || tokenParts[0] != "Bearer" {
			http.Error(w, "Invalid authorization format", http.StatusUnauthorized)
			return
		}

		// Validate the token
		claims, err := ValidateJWT(tokenParts[1])
		if err != nil {
			http.Error(w, "Invalid or expired token", http.StatusUnauthorized)
			return
		}

		// Add claims to request context
		ctx := context.WithValue(r.Context(), "user", claims)

		// Call the next handler with the updated context
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}
```

For production applications, consider these JWT security enhancements:

1. Use RS256 (RSA) instead of HS256 for signature verification
2. Implement token refresh mechanisms to limit token lifetime
3. Maintain a token blacklist for revoked tokens
4. Include only necessary information in the JWT payload

### **30.3.3 Role-Based Access Control (RBAC)**

RBAC provides a structured approach to managing authorization in enterprise applications:

```go
package main

import (
	"context"
	"errors"
	"net/http"
)

// Role represents a user role
type Role string

const (
	RoleAdmin  Role = "admin"
	RoleEditor Role = "editor"
	RoleViewer Role = "viewer"
)

// Permission represents an action that can be performed
type Permission string

const (
	PermissionCreateUser  Permission = "create:user"
	PermissionUpdateUser  Permission = "update:user"
	PermissionDeleteUser  Permission = "delete:user"
	PermissionViewUser    Permission = "view:user"
	PermissionCreatePost  Permission = "create:post"
	PermissionUpdatePost  Permission = "update:post"
	PermissionDeletePost  Permission = "delete:post"
	PermissionViewPost    Permission = "view:post"
)

// RolePermissions maps roles to their permissions
var RolePermissions = map[Role][]Permission{
	RoleAdmin: {
		PermissionCreateUser, PermissionUpdateUser, PermissionDeleteUser, PermissionViewUser,
		PermissionCreatePost, PermissionUpdatePost, PermissionDeletePost, PermissionViewPost,
	},
	RoleEditor: {
		PermissionViewUser,
		PermissionCreatePost, PermissionUpdatePost, PermissionDeletePost, PermissionViewPost,
	},
	RoleViewer: {
		PermissionViewUser, PermissionViewPost,
	},
}

// HasPermission checks if a role has a specific permission
func HasPermission(role Role, permission Permission) bool {
	permissions, exists := RolePermissions[role]
	if !exists {
		return false
	}

	for _, p := range permissions {
		if p == permission {
			return true
		}
	}

	return false
}

// RequirePermission is a middleware that checks for a specific permission
func RequirePermission(permission Permission) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			// Get user claims from context (set by AuthMiddleware)
			claims, ok := r.Context().Value("user").(*Claims)
			if !ok {
				http.Error(w, "Unauthorized", http.StatusUnauthorized)
				return
			}

			// Convert role string to Role type
			role := Role(claims.Role)

			// Check if the role has the required permission
			if !HasPermission(role, permission) {
				http.Error(w, "Forbidden", http.StatusForbidden)
				return
			}

			// Call the next handler
			next.ServeHTTP(w, r)
		})
	}
}

// Example usage in routes
func setupRoutes(router *http.ServeMux) {
	// Protected routes with required permissions
	router.Handle("/api/users",
		AuthMiddleware(
			RequirePermission(PermissionViewUser)(
				http.HandlerFunc(listUsersHandler),
			),
		),
	)

	router.Handle("/api/users/create",
		AuthMiddleware(
			RequirePermission(PermissionCreateUser)(
				http.HandlerFunc(createUserHandler),
			),
		),
	)

	// ... other routes
}
```

### **30.3.4 Attribute-Based Access Control (ABAC)**

For more complex authorization scenarios, implement Attribute-Based Access Control:

```go
package main

import (
	"context"
	"net/http"
	"time"
)

// Resource represents an entity that can be accessed
type Resource struct {
	ID        string
	Type      string
	OwnerID   string
	CreatedAt time.Time
	Metadata  map[string]interface{}
}

// AccessRequest contains all context for an access decision
type AccessRequest struct {
	User     *User
	Resource *Resource
	Action   string
	Context  map[string]interface{}
}

// PolicyEvaluator evaluates whether an access should be granted
type PolicyEvaluator interface {
	Evaluate(AccessRequest) bool
}

// OwnershipPolicy grants access if the user owns the resource
type OwnershipPolicy struct{}

func (p *OwnershipPolicy) Evaluate(req AccessRequest) bool {
	return req.User.ID == req.Resource.OwnerID
}

// BusinessHoursPolicy grants access only during business hours
type BusinessHoursPolicy struct{}

func (p *BusinessHoursPolicy) Evaluate(req AccessRequest) bool {
	now := time.Now()
	hour := now.Hour()

	// Business hours: Monday-Friday, 9 AM - 5 PM
	return now.Weekday() != time.Saturday &&
		   now.Weekday() != time.Sunday &&
		   hour >= 9 && hour < 17
}

// IPRangePolicy grants access based on IP address
type IPRangePolicy struct {
	AllowedRanges []string
}

func (p *IPRangePolicy) Evaluate(req AccessRequest) bool {
	clientIP, ok := req.Context["client_ip"].(string)
	if !ok {
		return false
	}

	// Check if IP is in allowed ranges
	// ... (implementation omitted)
	return true
}

// PolicyEngine evaluates all policies for an access request
type PolicyEngine struct {
	policies []PolicyEvaluator
}

func NewPolicyEngine() *PolicyEngine {
	return &PolicyEngine{
		policies: []PolicyEvaluator{},
	}
}

func (e *PolicyEngine) AddPolicy(policy PolicyEvaluator) {
	e.policies = append(e.policies, policy)
}

// CheckAccess evaluates all policies and returns true if all pass
func (e *PolicyEngine) CheckAccess(req AccessRequest) bool {
	for _, policy := range e.policies {
		if !policy.Evaluate(req) {
			return false
		}
	}
	return true
}

// AuthorizationMiddleware checks access using ABAC
func AuthorizationMiddleware(engine *PolicyEngine, resourceType, action string) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			// Get user from context
			user, ok := r.Context().Value("user").(*User)
			if !ok {
				http.Error(w, "Unauthorized", http.StatusUnauthorized)
				return
			}

			// Get resource ID from request
			resourceID := r.URL.Query().Get("id")

			// Fetch resource details
			resource, err := getResourceByID(resourceType, resourceID)
			if err != nil {
				http.Error(w, "Resource not found", http.StatusNotFound)
				return
			}

			// Create access request
			accessRequest := AccessRequest{
				User:     user,
				Resource: resource,
				Action:   action,
				Context: map[string]interface{}{
					"client_ip":  getClientIP(r),
					"user_agent": r.UserAgent(),
					"request_time": time.Now(),
				},
			}

			// Check access
			if !engine.CheckAccess(accessRequest) {
				http.Error(w, "Forbidden", http.StatusForbidden)
				return
			}

			// Access granted, call next handler
			next.ServeHTTP(w, r)
		})
	}
}

// Example usage
func setupABAC() *PolicyEngine {
	engine := NewPolicyEngine()

	// Add policies
	engine.AddPolicy(&OwnershipPolicy{})
	engine.AddPolicy(&BusinessHoursPolicy{})
	engine.AddPolicy(&IPRangePolicy{
		AllowedRanges: []string{"10.0.0.0/8", "192.168.0.0/16"},
	})

	return engine
}
```

This implementation provides flexible, context-aware authorization decisions based on multiple attributes.

### **30.3.5 Session Management**

Secure session management is critical for maintaining user authentication state:

```go
package main

import (
	"crypto/rand"
	"encoding/base64"
	"net/http"
	"time"

	"github.com/gorilla/sessions"
)

// Store for secure cookie-based sessions
var store = sessions.NewCookieStore(
	[]byte("session-authentication-key"),
	[]byte("session-encryption-key"),
)

func init() {
	// Configure session store
	store.Options = &sessions.Options{
		Path:     "/",
		MaxAge:   3600, // 1 hour
		HttpOnly: true,
		Secure:   true,  // Requires HTTPS
		SameSite: http.SameSiteStrictMode,
	}
}

// generateSessionID creates a cryptographically secure session ID
func generateSessionID() (string, error) {
	b := make([]byte, 32)
	_, err := rand.Read(b)
	if err != nil {
		return "", err
	}
	return base64.StdEncoding.EncodeToString(b), nil
}

// createUserSession creates a new session for an authenticated user
func createUserSession(w http.ResponseWriter, r *http.Request, user *User) error {
	// Get a session
	session, _ := store.Get(r, "user-session")

	// Generate a secure session ID
	sessionID, err := generateSessionID()
	if err != nil {
		return err
	}

	// Set session values
	session.Values["authenticated"] = true
	session.Values["user_id"] = user.ID
	session.Values["session_id"] = sessionID
	session.Values["created_at"] = time.Now().Unix()

	// Save the session
	return session.Save(r, w)
}

// AuthSessionMiddleware validates session-based authentication
func AuthSessionMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Get session
		session, _ := store.Get(r, "user-session")

		// Check if user is authenticated
		auth, ok := session.Values["authenticated"].(bool)
		if !ok || !auth {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}

		// Get user ID from session
		userID, ok := session.Values["user_id"].(string)
		if !ok {
			http.Error(w, "Invalid session", http.StatusUnauthorized)
			return
		}

		// Get user from database
		user, err := getUserByID(userID)
		if err != nil {
			http.Error(w, "User not found", http.StatusUnauthorized)
			return
		}

		// Add user to request context
		ctx := context.WithValue(r.Context(), "user", user)

		// Call the next handler with the updated context
		next.ServeHTTP(w, r.WithContext(ctx))
	})
}

// logoutHandler invalidates the user session
func logoutHandler(w http.ResponseWriter, r *http.Request) {
	// Get session
	session, _ := store.Get(r, "user-session")

	// Invalidate session
	session.Values["authenticated"] = false
	session.Options.MaxAge = -1

	// Save session changes
	session.Save(r, w)

	// Redirect to login page
	http.Redirect(w, r, "/login", http.StatusSeeOther)
}
```

For production applications, consider using a distributed session store like Redis for better scalability and security.

By implementing these authentication and authorization patterns in your Go applications, you can create a robust security foundation for enterprise environments.

## **30.4 Secrets Management and Data Protection**

Enterprise applications often handle sensitive data, from user credentials to API keys and encryption keys. Proper secrets management and data protection are essential components of a secure Go application.

### **30.4.1 Secrets Management**

Managing secrets securely involves using specialized tools and practices to store, access, and rotate sensitive credentials.

#### **Environment Variables**

The simplest approach for small applications is using environment variables:

```go
package main

import (
	"fmt"
	"os"
)

// Config contains application secrets and configuration
type Config struct {
	DatabaseURL      string
	APIKey           string
	JWTSecret        string
	EncryptionKey    string
	RedisURL         string
	SMTPCredentials  SMTPConfig
}

type SMTPConfig struct {
	Host     string
	Port     int
	Username string
	Password string
}

// LoadConfig loads configuration from environment variables
func LoadConfig() (*Config, error) {
	// Simple direct loading
	dbURL := os.Getenv("DATABASE_URL")
	if dbURL == "" {
		return nil, fmt.Errorf("DATABASE_URL environment variable is required")
	}

	// Load optional values with defaults
	redisURL := os.Getenv("REDIS_URL")
	if redisURL == "" {
		redisURL = "redis://localhost:6379/0"
	}

	// Parse numeric values
	smtpPort := 587 // Default value
	if portStr := os.Getenv("SMTP_PORT"); portStr != "" {
		if port, err := strconv.Atoi(portStr); err == nil {
			smtpPort = port
		}
	}

	return &Config{
		DatabaseURL:   dbURL,
		APIKey:        os.Getenv("API_KEY"),
		JWTSecret:     os.Getenv("JWT_SECRET"),
		EncryptionKey: os.Getenv("ENCRYPTION_KEY"),
		RedisURL:      redisURL,
		SMTPCredentials: SMTPConfig{
			Host:     os.Getenv("SMTP_HOST"),
			Port:     smtpPort,
			Username: os.Getenv("SMTP_USERNAME"),
			Password: os.Getenv("SMTP_PASSWORD"),
		},
	}, nil
}
```

While simple, this approach has limitations for enterprise environments, particularly around secret rotation and access control.

#### **Vault Integration**

For enterprise applications, HashiCorp Vault provides a comprehensive secrets management solution:

```go
package main

import (
	"fmt"
	"os"

	vault "github.com/hashicorp/vault/api"
)

// SecretsManager handles secure access to application secrets
type SecretsManager struct {
	client *vault.Client
	path   string
}

// NewSecretsManager creates a new Vault-backed secrets manager
func NewSecretsManager(address, token, path string) (*SecretsManager, error) {
	config := vault.DefaultConfig()
	config.Address = address

	client, err := vault.NewClient(config)
	if err != nil {
		return nil, fmt.Errorf("failed to create Vault client: %w", err)
	}

	client.SetToken(token)

	return &SecretsManager{
		client: client,
		path:   path,
	}, nil
}

// GetSecret retrieves a secret from Vault
func (sm *SecretsManager) GetSecret(key string) (string, error) {
	// Read secret from Vault
	secret, err := sm.client.Logical().Read(sm.path)
	if err != nil {
		return "", fmt.Errorf("failed to read secret: %w", err)
	}

	if secret == nil || secret.Data == nil {
		return "", fmt.Errorf("secret not found at path: %s", sm.path)
	}

	// For KV v1, data is directly in secret.Data
	// For KV v2, data is in secret.Data["data"]
	var data map[string]interface{}

	if v2Data, ok := secret.Data["data"].(map[string]interface{}); ok {
		// KV v2
		data = v2Data
	} else {
		// KV v1
		data = secret.Data
	}

	// Get the specific key
	value, ok := data[key].(string)
	if !ok {
		return "", fmt.Errorf("key %s not found in secret", key)
	}

	return value, nil
}

// Example usage
func main() {
	// Get Vault token from environment or authentication
	vaultToken := os.Getenv("VAULT_TOKEN")
	vaultAddr := os.Getenv("VAULT_ADDR")

	// Create secrets manager
	secretsPath := "secret/data/myapp/credentials" // For KV v2
	sm, err := NewSecretsManager(vaultAddr, vaultToken, secretsPath)
	if err != nil {
		log.Fatalf("Failed to create secrets manager: %v", err)
	}

	// Get database password
	dbPassword, err := sm.GetSecret("db_password")
	if err != nil {
		log.Fatalf("Failed to get database password: %v", err)
	}

	// Use the password securely
	// ...
}
```

For Kubernetes deployments, integrate with Kubernetes secrets using the Vault Agent or CSI Driver for automatic injection of secrets.

#### **Cloud Provider Secret Management**

For applications deployed in cloud environments, use cloud-specific secret management services:

```go
package main

import (
	"context"
	"fmt"
	"log"

	secretmanager "cloud.google.com/go/secretmanager/apiv1"
	secretmanagerpb "google.golang.org/genproto/googleapis/cloud/secretmanager/v1"
)

// GCPSecretsManager handles access to GCP Secret Manager
type GCPSecretsManager struct {
	client  *secretmanager.Client
	project string
}

// NewGCPSecretsManager creates a new GCP Secret Manager client
func NewGCPSecretsManager(projectID string) (*GCPSecretsManager, error) {
	ctx := context.Background()
	client, err := secretmanager.NewClient(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to create Secret Manager client: %w", err)
	}

	return &GCPSecretsManager{
		client:  client,
		project: projectID,
	}, nil
}

// GetSecret retrieves a secret from GCP Secret Manager
func (sm *GCPSecretsManager) GetSecret(name string) (string, error) {
	ctx := context.Background()

	// Build the resource name
	resourceName := fmt.Sprintf("projects/%s/secrets/%s/versions/latest", sm.project, name)

	// Access the secret version
	req := &secretmanagerpb.AccessSecretVersionRequest{
		Name: resourceName,
	}

	result, err := sm.client.AccessSecretVersion(ctx, req)
	if err != nil {
		return "", fmt.Errorf("failed to access secret version: %w", err)
	}

	// Return the secret payload as a string
	return string(result.Payload.Data), nil
}

// Close closes the Secret Manager client
func (sm *GCPSecretsManager) Close() error {
	return sm.client.Close()
}

// Example usage
func main() {
	// Create secrets manager
	sm, err := NewGCPSecretsManager("my-gcp-project")
	if err != nil {
		log.Fatalf("Failed to create secrets manager: %v", err)
	}
	defer sm.Close()

	// Get API key
	apiKey, err := sm.GetSecret("api-key")
	if err != nil {
		log.Fatalf("Failed to get API key: %v", err)
	}

	// Use the API key securely
	// ...
}
```

Similar integrations exist for AWS Secrets Manager, Azure Key Vault, and other cloud providers.

### **30.4.2 Data Encryption**

Protecting sensitive data through encryption is a fundamental security practice for enterprise applications.

#### **Encrypting Data at Rest**

For encrypting data before storing it in a database or file system:

```go
package main

import (
	"crypto/aes"
	"crypto/cipher"
	"crypto/rand"
	"encoding/base64"
	"errors"
	"io"
)

// Encrypter handles encryption and decryption of sensitive data
type Encrypter struct {
	key []byte
}

// NewEncrypter creates a new encrypter with the given key
func NewEncrypter(key []byte) (*Encrypter, error) {
	if len(key) != 32 {
		return nil, errors.New("encryption key must be 32 bytes (256 bits)")
	}

	return &Encrypter{key: key}, nil
}

// Encrypt encrypts plaintext data
func (e *Encrypter) Encrypt(plaintext string) (string, error) {
	// Create the AES cipher
	block, err := aes.NewCipher(e.key)
	if err != nil {
		return "", err
	}

	// Create a GCM cipher mode
	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	// Create a nonce
	nonce := make([]byte, gcm.NonceSize())
	if _, err := io.ReadFull(rand.Reader, nonce); err != nil {
		return "", err
	}

	// Encrypt the data
	ciphertext := gcm.Seal(nonce, nonce, []byte(plaintext), nil)

	// Base64 encode the result
	return base64.StdEncoding.EncodeToString(ciphertext), nil
}

// Decrypt decrypts ciphertext data
func (e *Encrypter) Decrypt(ciphertext string) (string, error) {
	// Decode from base64
	data, err := base64.StdEncoding.DecodeString(ciphertext)
	if err != nil {
		return "", err
	}

	// Create the AES cipher
	block, err := aes.NewCipher(e.key)
	if err != nil {
		return "", err
	}

	// Create a GCM cipher mode
	gcm, err := cipher.NewGCM(block)
	if err != nil {
		return "", err
	}

	// Extract the nonce
	nonceSize := gcm.NonceSize()
	if len(data) < nonceSize {
		return "", errors.New("ciphertext too short")
	}

	nonce, ciphertextBytes := data[:nonceSize], data[nonceSize:]

	// Decrypt the data
	plaintextBytes, err := gcm.Open(nil, nonce, ciphertextBytes, nil)
	if err != nil {
		return "", err
	}

	return string(plaintextBytes), nil
}

// Example usage for encrypting database fields
type User struct {
	ID            int64
	Email         string
	PasswordHash  string
	SSN           string  // Social Security Number (sensitive)
	EncryptedSSN  string  // Encrypted version for storage
}

func storeUserWithEncryption(user *User, encrypter *Encrypter) error {
	// Encrypt sensitive fields
	encryptedSSN, err := encrypter.Encrypt(user.SSN)
	if err != nil {
		return err
	}

	// Store the encrypted value
	user.EncryptedSSN = encryptedSSN
	user.SSN = "" // Don't keep the plaintext in memory

	// Save to database
	// db.Save(user)

	return nil
}

func retrieveUserWithDecryption(userID int64, encrypter *Encrypter) (*User, error) {
	// Retrieve user from database
	user := &User{ID: userID}
	// db.Find(user)

	// Decrypt sensitive fields
	ssn, err := encrypter.Decrypt(user.EncryptedSSN)
	if err != nil {
		return nil, err
	}

	// Set the decrypted value for use
	user.SSN = ssn

	return user, nil
}
```

#### **Secure Key Management**

For production applications, securely manage encryption keys using a key management service (KMS):

```go
package main

import (
	"context"
	"fmt"

	kms "cloud.google.com/go/kms/apiv1"
	kmspb "google.golang.org/genproto/googleapis/cloud/kms/v1"
)

// KMSEncrypter uses Google Cloud KMS for encryption operations
type KMSEncrypter struct {
	client     *kms.KeyManagementClient
	keyName    string
}

// NewKMSEncrypter creates a new KMS-backed encrypter
func NewKMSEncrypter(projectID, location, keyRing, keyName string) (*KMSEncrypter, error) {
	ctx := context.Background()
	client, err := kms.NewKeyManagementClient(ctx)
	if err != nil {
		return nil, fmt.Errorf("failed to create KMS client: %w", err)
	}

	// Format the key name
	formattedKeyName := fmt.Sprintf(
		"projects/%s/locations/%s/keyRings/%s/cryptoKeys/%s",
		projectID, location, keyRing, keyName,
	)

	return &KMSEncrypter{
		client:     client,
		keyName:    formattedKeyName,
	}, nil
}

// Encrypt encrypts plaintext using KMS
func (e *KMSEncrypter) Encrypt(plaintext []byte) ([]byte, error) {
	ctx := context.Background()

	// Build the request
	req := &kmspb.EncryptRequest{
		Name:      e.keyName,
		Plaintext: plaintext,
	}

	// Call the API
	result, err := e.client.Encrypt(ctx, req)
	if err != nil {
		return nil, fmt.Errorf("failed to encrypt: %w", err)
	}

	return result.Ciphertext, nil
}

// Decrypt decrypts ciphertext using KMS
func (e *KMSEncrypter) Decrypt(ciphertext []byte) ([]byte, error) {
	ctx := context.Background()

	// Build the request
	req := &kmspb.DecryptRequest{
		Name:       e.keyName,
		Ciphertext: ciphertext,
	}

	// Call the API
	result, err := e.client.Decrypt(ctx, req)
	if err != nil {
		return nil, fmt.Errorf("failed to decrypt: %w", err)
	}

	return result.Plaintext, nil
}

// Close closes the KMS client
func (e *KMSEncrypter) Close() error {
	return e.client.Close()
}
```

#### **Transport Layer Security (TLS)**

Ensure secure communication with proper TLS configuration:

```go
package main

import (
	"crypto/tls"
	"log"
	"net/http"
	"time"
)

// ConfigureTLSServer creates an HTTPS server with secure TLS settings
func ConfigureTLSServer() *http.Server {
	// Define TLS configuration
	tlsConfig := &tls.Config{
		// Minimum TLS version
		MinVersion: tls.VersionTLS12,

		// Preferred cipher suites (in order of preference)
		CipherSuites: []uint16{
			tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,
			tls.TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,
			tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
			tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
		},

		// Curve preferences
		CurvePreferences: []tls.CurveID{
			tls.CurveP256,
			tls.X25519,
		},
	}

	// Create the server
	server := &http.Server{
		Addr:         ":8443",
		Handler:      setupRoutes(),
		TLSConfig:    tlsConfig,
		ReadTimeout:  10 * time.Second,
		WriteTimeout: 10 * time.Second,
		IdleTimeout:  120 * time.Second,
	}

	return server
}

// StartTLSServer starts the HTTPS server
func StartTLSServer() {
	server := ConfigureTLSServer()

	log.Println("Starting secure server on :8443")
	if err := server.ListenAndServeTLS("server.crt", "server.key"); err != http.ErrServerClosed {
		log.Fatalf("Server error: %v", err)
	}
}

// ConfigureTLSClient creates an HTTP client with secure TLS settings
func ConfigureTLSClient() *http.Client {
	// Define TLS configuration for client
	tlsConfig := &tls.Config{
		MinVersion: tls.VersionTLS12,
		CipherSuites: []uint16{
			tls.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,
			tls.TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305,
			tls.TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305,
			tls.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
			tls.TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,
		},
	}

	// Create transport with TLS configuration
	transport := &http.Transport{
		TLSClientConfig:     tlsConfig,
		MaxIdleConns:        100,
		MaxIdleConnsPerHost: 10,
		MaxConnsPerHost:     100,
		IdleConnTimeout:     90 * time.Second,
	}

	// Create client with transport
	client := &http.Client{
		Transport: transport,
		Timeout:   10 * time.Second,
	}

	return client
}
```

### **30.4.3 Secure Database Access**

Secure your database connections and operations:

```go
package main

import (
	"context"
	"database/sql"
	"fmt"
	"log"
	"time"

	_ "github.com/lib/pq"
)

// SecureDBConfig contains database connection configuration
type SecureDBConfig struct {
	Host              string
	Port              int
	Database          string
	User              string
	Password          string
	SSLMode           string
	MaxOpenConns      int
	MaxIdleConns      int
	ConnMaxLifetime   time.Duration
	StatementTimeout  time.Duration
	QueryTimeout      time.Duration
}

// NewDefaultDBConfig creates a secure default configuration
func NewDefaultDBConfig() *SecureDBConfig {
	return &SecureDBConfig{
		Host:             "localhost",
		Port:             5432,
		Database:         "appdb",
		User:             "appuser",
		Password:         "", // Load from secrets manager
		SSLMode:          "verify-full", // Require SSL with cert verification
		MaxOpenConns:     25,
		MaxIdleConns:     5,
		ConnMaxLifetime:  15 * time.Minute,
		StatementTimeout: 30 * time.Second,
		QueryTimeout:     10 * time.Second,
	}
}

// ConnectToDB creates a secure database connection
func ConnectToDB(config *SecureDBConfig) (*sql.DB, error) {
	// Build connection string
	connStr := fmt.Sprintf(
		"host=%s port=%d dbname=%s user=%s password=%s sslmode=%s",
		config.Host, config.Port, config.Database, config.User, config.Password, config.SSLMode,
	)

	// Open database connection
	db, err := sql.Open("postgres", connStr)
	if err != nil {
		return nil, fmt.Errorf("failed to open database: %w", err)
	}

	// Configure connection pool
	db.SetMaxOpenConns(config.MaxOpenConns)
	db.SetMaxIdleConns(config.MaxIdleConns)
	db.SetConnMaxLifetime(config.ConnMaxLifetime)

	// Verify connection
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	if err := db.PingContext(ctx); err != nil {
		db.Close()
		return nil, fmt.Errorf("failed to ping database: %w", err)
	}

	// Set statement timeout
	_, err = db.Exec(fmt.Sprintf("SET statement_timeout TO %d", int(config.StatementTimeout.Seconds())*1000))
	if err != nil {
		log.Printf("Warning: Failed to set statement timeout: %v", err)
	}

	return db, nil
}

// QueryWithTimeout executes a query with a timeout
func QueryWithTimeout(db *sql.DB, timeout time.Duration, query string, args ...interface{}) (*sql.Rows, error) {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()

	return db.QueryContext(ctx, query, args...)
}

// ExecWithTimeout executes a statement with a timeout
func ExecWithTimeout(db *sql.DB, timeout time.Duration, query string, args ...interface{}) (sql.Result, error) {
	ctx, cancel := context.WithTimeout(context.Background(), timeout)
	defer cancel()

	return db.ExecContext(ctx, query, args...)
}

// Example usage
func main() {
	// Create database configuration
	dbConfig := NewDefaultDBConfig()

	// Load password from secrets manager
	secretsManager := getSecretsManager()
	password, err := secretsManager.GetSecret("db_password")
	if err != nil {
		log.Fatalf("Failed to get database password: %v", err)
	}
	dbConfig.Password = password

	// Connect to database
	db, err := ConnectToDB(dbConfig)
	if err != nil {
		log.Fatalf("Failed to connect to database: %v", err)
	}
	defer db.Close()

	// Execute query with timeout
	rows, err := QueryWithTimeout(db, dbConfig.QueryTimeout,
		"SELECT id, name FROM users WHERE active = $1", true)
	if err != nil {
		log.Fatalf("Query failed: %v", err)
	}
	defer rows.Close()

	// Process results
	// ...
}
```

### **30.4.4 Secure File Handling**

Implement secure file operations to prevent data leakage:

```go
package main

import (
	"crypto/rand"
	"encoding/hex"
	"fmt"
	"io"
	"os"
	"path/filepath"
	"strings"
)

// SecureFileManager handles secure file operations
type SecureFileManager struct {
	baseDir     string
	permissions os.FileMode
}

// NewSecureFileManager creates a new secure file manager
func NewSecureFileManager(baseDir string) (*SecureFileManager, error) {
	// Create base directory if it doesn't exist
	if err := os.MkdirAll(baseDir, 0750); err != nil {
		return nil, fmt.Errorf("failed to create base directory: %w", err)
	}

	return &SecureFileManager{
		baseDir:     baseDir,
		permissions: 0640, // Owner can read/write, group can read
	}, nil
}

// GenerateSecureFilename creates a random, secure filename
func (sfm *SecureFileManager) GenerateSecureFilename(prefix string, ext string) (string, error) {
	// Generate 16 random bytes
	randomBytes := make([]byte, 16)
	if _, err := rand.Read(randomBytes); err != nil {
		return "", fmt.Errorf("failed to generate random bytes: %w", err)
	}

	// Convert to hex string
	randomHex := hex.EncodeToString(randomBytes)

	// Clean extension
	if ext != "" && !strings.HasPrefix(ext, ".") {
		ext = "." + ext
	}

	// Create filename
	filename := fmt.Sprintf("%s_%s%s", prefix, randomHex, ext)

	return filename, nil
}

// SafeWriteFile writes data to a file with secure permissions
func (sfm *SecureFileManager) SafeWriteFile(filename string, data []byte) error {
	// Validate filename doesn't contain path traversal
	if strings.Contains(filename, "/") || strings.Contains(filename, "\\") {
		return fmt.Errorf("invalid filename: %s", filename)
	}

	// Create full path
	fullPath := filepath.Join(sfm.baseDir, filename)

	// Write to a temporary file first
	tempFile := fullPath + ".tmp"
	if err := os.WriteFile(tempFile, data, sfm.permissions); err != nil {
		return fmt.Errorf("failed to write temporary file: %w", err)
	}

	// Rename to final filename (atomic operation)
	if err := os.Rename(tempFile, fullPath); err != nil {
		// Clean up temporary file
		os.Remove(tempFile)
		return fmt.Errorf("failed to rename temporary file: %w", err)
	}

	return nil
}

// SafeReadFile reads a file securely
func (sfm *SecureFileManager) SafeReadFile(filename string) ([]byte, error) {
	// Validate filename doesn't contain path traversal
	if strings.Contains(filename, "/") || strings.Contains(filename, "\\") {
		return nil, fmt.Errorf("invalid filename: %s", filename)
	}

	// Create full path
	fullPath := filepath.Join(sfm.baseDir, filename)

	// Read file
	data, err := os.ReadFile(fullPath)
	if err != nil {
		return nil, fmt.Errorf("failed to read file: %w", err)
	}

	return data, nil
}

// SecureDeleteFile securely deletes a file by overwriting before removal
func (sfm *SecureFileManager) SecureDeleteFile(filename string) error {
	// Validate filename doesn't contain path traversal
	if strings.Contains(filename, "/") || strings.Contains(filename, "\\") {
		return fmt.Errorf("invalid filename: %s", filename)
	}

	// Create full path
	fullPath := filepath.Join(sfm.baseDir, filename)

	// Get file info
	info, err := os.Stat(fullPath)
	if err != nil {
		return fmt.Errorf("failed to stat file: %w", err)
	}

	// Open file for writing
	file, err := os.OpenFile(fullPath, os.O_WRONLY, 0)
	if err != nil {
		return fmt.Errorf("failed to open file for secure deletion: %w", err)
	}

	// Get file size
	size := info.Size()

	// Create buffer of zeros
	bufferSize := 4096
	zeros := make([]byte, bufferSize)

	// Overwrite file with zeros
	var written int64
	for written < size {
		n := bufferSize
		if size-written < int64(bufferSize) {
			n = int(size - written)
		}
		if _, err := file.Write(zeros[:n]); err != nil {
			file.Close()
			return fmt.Errorf("failed to overwrite file: %w", err)
		}
		written += int64(n)
	}

	// Sync to ensure write
	if err := file.Sync(); err != nil {
		file.Close()
		return fmt.Errorf("failed to sync file: %w", err)
	}

	// Close file
	if err := file.Close(); err != nil {
		return fmt.Errorf("failed to close file: %w", err)
	}

	// Delete file
	if err := os.Remove(fullPath); err != nil {
		return fmt.Errorf("failed to remove file: %w", err)
	}

	return nil
}

// Example usage
func main() {
	// Create secure file manager
	sfm, err := NewSecureFileManager("/app/secure-files")
	if err != nil {
		log.Fatalf("Failed to create secure file manager: %v", err)
	}

	// Generate secure filename
	filename, err := sfm.GenerateSecureFilename("report", "pdf")
	if err != nil {
		log.Fatalf("Failed to generate filename: %v", err)
	}

	// Write secure data
	data := []byte("Sensitive information")
	if err := sfm.SafeWriteFile(filename, data); err != nil {
		log.Fatalf("Failed to write file: %v", err)
	}

	// Read secure data
	readData, err := sfm.SafeReadFile(filename)
	if err != nil {
		log.Fatalf("Failed to read file: %v", err)
	}

	// Process data
	// ...

	// Securely delete when done
	if err := sfm.SecureDeleteFile(filename); err != nil {
		log.Fatalf("Failed to securely delete file: %v", err)
	}
}
```

By implementing these secure data handling practices, your Go application will better protect sensitive information at rest and in transit, a critical requirement for enterprise environments.

## **30.5 Security Testing and Compliance**

Implementing security features is only half the battle. Enterprise applications must also verify the effectiveness of these security controls through testing and ensure compliance with relevant standards and regulations.

### **30.5.1 Security Testing in Go Applications**

A comprehensive security testing strategy includes:

#### **Static Application Security Testing (SAST)**

SAST tools analyze source code to identify security vulnerabilities. For Go applications:

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

// RunSASTAnalysis runs static security analysis tools
func RunSASTAnalysis(repoPath string) error {
	// Run gosec for security scanning
	cmd := exec.Command("gosec", "-fmt=json", "-out=security-results.json", "./...")
	cmd.Dir = repoPath
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	fmt.Println("Running gosec security scanner...")
	if err := cmd.Run(); err != nil {
		return fmt.Errorf("gosec security scan failed: %w", err)
	}

	// Run Nancy for dependency vulnerability scanning
	cmd = exec.Command("sh", "-c", "go list -json -deps ./... | nancy sleuth")
	cmd.Dir = repoPath
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	fmt.Println("Running nancy dependency scanner...")
	if err := cmd.Run(); err != nil {
		return fmt.Errorf("nancy dependency scan failed: %w", err)
	}

	return nil
}
```

Integrate this into your CI/CD pipeline with tools like:

- gosec: For Go-specific security issues
- nancy: For dependency vulnerabilities
- golangci-lint: With security linters enabled
- SonarQube: For comprehensive code quality and security analysis

#### **Dynamic Application Security Testing (DAST)**

DAST tools test running applications by simulating attacks:

```go
// Example GitHub Actions workflow for DAST
// .github/workflows/dast.yml
name: Dynamic Security Testing

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build and start application
        run: |
          docker-compose up -d
          sleep 30  # Give app time to start

      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'http://localhost:8080'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'
```

Popular DAST tools for Go applications include:

- OWASP ZAP
- Burp Suite
- Nuclei

#### **Fuzz Testing**

Go's built-in fuzzing capabilities help identify security issues by providing unexpected inputs:

```go
package validator

import (
	"testing"
	"unicode/utf8"
)

// Fuzz test for the ValidateEmail function
func FuzzValidateEmail(f *testing.F) {
	// Seed corpus
	testcases := []string{
		"user@example.com",
		"user+tag@example.com",
		"user.name@example.co.uk",
		"user@localhost",
		"@example.com",
		"user@",
		"user@.com",
		"user@example..com",
		"<script>alert('xss')</script>@example.com",
	}

	for _, tc := range testcases {
		f.Add(tc) // Add each test case to the corpus
	}

	// Fuzz test function
	f.Fuzz(func(t *testing.T, email string) {
		// Skip non-UTF8 strings
		if !utf8.ValidString(email) {
			return
		}

		// Call the function being tested
		result := ValidateEmail(email)

		// Verify no panics and check basic invariants
		if result && len(email) == 0 {
			t.Errorf("ValidateEmail accepted empty string")
		}

		if result && !strings.Contains(email, "@") {
			t.Errorf("ValidateEmail accepted string without @ symbol: %q", email)
		}
	})
}
```

Run fuzz tests with:

```bash
go test -fuzz=FuzzValidateEmail -fuzztime=1m
```

#### **Penetration Testing**

Structured penetration testing should focus on:

1. Authentication and authorization bypass
2. Injection vulnerabilities
3. Business logic flaws
4. API security
5. Infrastructure security

Create a security testing checklist that includes:

- Common OWASP Top 10 vulnerabilities
- Go-specific security issues
- Business logic security concerns
- Infrastructure and deployment security

### **30.5.2 Security Compliance for Enterprise Go Applications**

Achieving compliance involves mapping security controls to specific requirements:

#### **Compliance Frameworks**

Common frameworks for enterprise applications include:

1. **SOC 2**: Focus on security, availability, processing integrity, confidentiality, and privacy
2. **GDPR**: European data protection regulation
3. **HIPAA**: Healthcare data protection in the US
4. **PCI DSS**: Payment card data security
5. **ISO 27001**: Information security management systems

#### **Implementing Compliance Controls in Go**

Implement technical controls required by these frameworks:

```go
package main

import (
	"log"
	"net/http"
	"time"

	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promauto"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

// Compliance metrics for auditing and reporting
var (
	authFailures = promauto.NewCounter(prometheus.CounterOpts{
		Name: "app_auth_failures_total",
		Help: "Total number of authentication failures",
	})

	accessAttempts = promauto.NewCounterVec(prometheus.CounterOpts{
		Name: "app_access_attempts_total",
		Help: "Access attempts by resource and outcome",
	}, []string{"resource", "outcome"})

	dataAccessLatency = promauto.NewHistogramVec(prometheus.HistogramOpts{
		Name:    "app_data_access_latency_seconds",
		Help:    "Latency of data access operations",
		Buckets: prometheus.LinearBuckets(0.01, 0.05, 10),
	}, []string{"data_type", "operation"})
)

// AuditLog represents a security-relevant event
type AuditLog struct {
	Timestamp   time.Time `json:"timestamp"`
	UserID      string    `json:"user_id"`
	Action      string    `json:"action"`
	Resource    string    `json:"resource"`
	ResourceID  string    `json:"resource_id"`
	Outcome     string    `json:"outcome"`
	IPAddress   string    `json:"ip_address"`
	UserAgent   string    `json:"user_agent"`
	Description string    `json:"description"`
}

// LogAuditEvent records a security event for compliance
func LogAuditEvent(userID, action, resource, resourceID, outcome, ipAddress, userAgent, description string) error {
	event := AuditLog{
		Timestamp:   time.Now().UTC(),
		UserID:      userID,
		Action:      action,
		Resource:    resource,
		ResourceID:  resourceID,
		Outcome:     outcome,
		IPAddress:   ipAddress,
		UserAgent:   userAgent,
		Description: description,
	}

	// Log to structured logging system
	log.Printf("AUDIT: %+v", event)

	// Update metrics
	if action == "authenticate" && outcome == "failure" {
		authFailures.Inc()
	}

	accessAttempts.WithLabelValues(resource, outcome).Inc()

	// Store audit log in database
	// ...

	return nil
}

// Example compliance middleware for access control
func complianceMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		user := getCurrentUser(r)
		resource := r.URL.Path

		start := time.Now()

		// Check if user has access to the resource
		hasAccess, err := checkAccess(user, resource, "read")
		if err != nil {
			LogAuditEvent(
				user.ID,
				"access",
				resource,
				"",
				"error",
				getClientIP(r),
				r.UserAgent(),
				"Error checking access: "+err.Error(),
			)
			http.Error(w, "Internal server error", http.StatusInternalServerError)
			return
		}

		if !hasAccess {
			LogAuditEvent(
				user.ID,
				"access",
				resource,
				"",
				"denied",
				getClientIP(r),
				r.UserAgent(),
				"Access denied",
			)
			http.Error(w, "Forbidden", http.StatusForbidden)
			return
		}

		// Wrap response writer to capture status code
		wrappedWriter := newStatusResponseWriter(w)

		// Call the next handler
		next.ServeHTTP(wrappedWriter, r)

		// Log the access after the request is handled
		outcome := "success"
		if wrappedWriter.status >= 400 {
			outcome = "failure"
		}

		LogAuditEvent(
			user.ID,
			"access",
			resource,
			"",
			outcome,
			getClientIP(r),
			r.UserAgent(),
			"Access completed with status: "+string(wrappedWriter.status),
		)

		// Record latency metrics
		duration := time.Since(start).Seconds()
		dataAccessLatency.WithLabelValues(getResourceType(resource), "read").Observe(duration)
	})
}

// Example compliance health endpoint
func setupComplianceHealthEndpoints(mux *http.ServeMux) {
	// Export Prometheus metrics
	mux.Handle("/metrics", promhttp.Handler())

	// Health and compliance status
	mux.HandleFunc("/health/compliance", func(w http.ResponseWriter, r *http.Request) {
		status := map[string]interface{}{
			"status": "compliant",
			"checks": map[string]string{
				"data_encryption":       "enabled",
				"audit_logging":         "enabled",
				"authentication":        "enabled",
				"authorization":         "enabled",
				"tls":                   "enabled",
				"secure_headers":        "enabled",
				"input_validation":      "enabled",
				"dependency_scanning":   "completed",
				"vulnerability_testing": "completed",
			},
			"last_compliance_scan": time.Now().Add(-24 * time.Hour).Format(time.RFC3339),
			"compliance_version":   "1.2.0",
		}

		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(status)
	})
}
```

### **30.5.3 Continuous Security Monitoring**

Implement ongoing security monitoring for your Go applications:

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

// SecurityMonitor provides continuous security monitoring
type SecurityMonitor struct {
	alertHandlers  []AlertHandler
	checkInterval  time.Duration
	checks         []SecurityCheck
	stopCh         chan struct{}
	wg             sync.WaitGroup
}

// SecurityCheck represents a security verification
type SecurityCheck interface {
	Name() string
	Run(ctx context.Context) (bool, string, error)
}

// AlertHandler processes security alerts
type AlertHandler interface {
	HandleAlert(alert SecurityAlert)
}

// SecurityAlert represents a security issue
type SecurityAlert struct {
	Timestamp time.Time
	CheckName string
	Severity  string
	Message   string
	Details   map[string]interface{}
}

// NewSecurityMonitor creates a new security monitor
func NewSecurityMonitor(interval time.Duration) *SecurityMonitor {
	return &SecurityMonitor{
		alertHandlers: []AlertHandler{},
		checkInterval: interval,
		checks:        []SecurityCheck{},
		stopCh:        make(chan struct{}),
	}
}

// AddCheck adds a security check
func (sm *SecurityMonitor) AddCheck(check SecurityCheck) {
	sm.checks = append(sm.checks, check)
}

// AddAlertHandler adds an alert handler
func (sm *SecurityMonitor) AddAlertHandler(handler AlertHandler) {
	sm.alertHandlers = append(sm.alertHandlers, handler)
}

// Start begins security monitoring
func (sm *SecurityMonitor) Start() {
	sm.wg.Add(1)
	go func() {
		defer sm.wg.Done()
		ticker := time.NewTicker(sm.checkInterval)
		defer ticker.Stop()

		// Run initial checks
		sm.runChecks(context.Background())

		for {
			select {
			case <-ticker.C:
				sm.runChecks(context.Background())
			case <-sm.stopCh:
				return
			}
		}
	}()
}

// Stop ends security monitoring
func (sm *SecurityMonitor) Stop() {
	close(sm.stopCh)
	sm.wg.Wait()
}

// runChecks executes all security checks
func (sm *SecurityMonitor) runChecks(ctx context.Context) {
	for _, check := range sm.checks {
		passed, message, err := check.Run(ctx)

		if err != nil {
			log.Printf("Error running security check %s: %v", check.Name(), err)
			continue
		}

		if !passed {
			alert := SecurityAlert{
				Timestamp: time.Now(),
				CheckName: check.Name(),
				Severity:  "high",
				Message:   message,
				Details: map[string]interface{}{
					"check_time": time.Now().Format(time.RFC3339),
				},
			}

			// Handle the alert
			for _, handler := range sm.alertHandlers {
				handler.HandleAlert(alert)
			}
		}
	}
}

// Example security checks
type TLSCertificateCheck struct {
	domain string
}

func (c *TLSCertificateCheck) Name() string {
	return "tls_certificate_expiry"
}

func (c *TLSCertificateCheck) Run(ctx context.Context) (bool, string, error) {
	// Check TLS certificate expiration
	// ...
	return true, "", nil
}

type DependencyVulnerabilityCheck struct {
	repoPath string
}

func (c *DependencyVulnerabilityCheck) Name() string {
	return "dependency_vulnerabilities"
}

func (c *DependencyVulnerabilityCheck) Run(ctx context.Context) (bool, string, error) {
	// Check for vulnerable dependencies
	// ...
	return true, "", nil
}

// Example alert handlers
type LogAlertHandler struct{}

func (h *LogAlertHandler) HandleAlert(alert SecurityAlert) {
	log.Printf("SECURITY ALERT: %s - %s", alert.CheckName, alert.Message)
}

type EmailAlertHandler struct {
	emailClient *EmailClient
	recipients  []string
}

func (h *EmailAlertHandler) HandleAlert(alert SecurityAlert) {
	subject := fmt.Sprintf("Security Alert: %s", alert.CheckName)
	body := fmt.Sprintf("Security issue detected:\n\nCheck: %s\nSeverity: %s\nMessage: %s\nTime: %s",
		alert.CheckName, alert.Severity, alert.Message, alert.Timestamp.Format(time.RFC3339))

	for _, recipient := range h.recipients {
		h.emailClient.SendEmail(recipient, subject, body)
	}
}

// Example monitoring setup
func setupSecurityMonitoring() *SecurityMonitor {
	// Create monitor with 1-hour check interval
	monitor := NewSecurityMonitor(1 * time.Hour)

	// Add security checks
	monitor.AddCheck(&TLSCertificateCheck{domain: "example.com"})
	monitor.AddCheck(&DependencyVulnerabilityCheck{repoPath: "./app"})

	// Add alert handlers
	monitor.AddAlertHandler(&LogAlertHandler{})
	monitor.AddAlertHandler(&EmailAlertHandler{
		emailClient: newEmailClient(),
		recipients:  []string{"security@example.com"},
	})

	return monitor
}

func main() {
	// Start security monitoring
	monitor := setupSecurityMonitoring()
	monitor.Start()

	// Handle graceful shutdown
	signalCh := make(chan os.Signal, 1)
	signal.Notify(signalCh, syscall.SIGINT, syscall.SIGTERM)
	<-signalCh

	log.Println("Shutting down security monitoring...")
	monitor.Stop()
}
```

## **30.6 Conclusion**

Enterprise-grade security in Go applications requires a comprehensive approach that addresses the entire security lifecycle. By implementing secure coding practices, robust authentication and authorization, proper secrets management, data protection, and security testing, you can build Go applications that meet the security requirements of enterprise environments.

Remember that security is not a one-time effort but an ongoing process of improvement and vigilance. By following the patterns and practices presented in this chapter, you can establish a strong security foundation for your Go applications and adapt to evolving threats and requirements over time.

As you build enterprise Go applications, regularly revisit your security controls, stay informed about emerging vulnerabilities and best practices, and maintain a security-first mindset throughout the development lifecycle.
