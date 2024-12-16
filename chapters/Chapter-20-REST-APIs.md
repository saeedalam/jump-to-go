
# **Chapter 20: Developing REST APIs in Go**

---

## **20.1. Introduction to REST APIs**

### **What is a REST API?**

A **REST API** (Representational State Transfer Application Programming Interface) is an architectural style used to create web services. It uses stateless communication and standard HTTP methods to manage resources represented by URLs. The main concept behind a REST API is that it allows interaction with resources using common HTTP methods like **GET**, **POST**, **PUT**, **DELETE**, and **PATCH**.

Key principles of REST:
- **Stateless Communication**: Each request is independent and contains all the necessary information.
- **Resources**: Data is represented as resources, identified by URLs. For example, `/books` could represent the collection of books, and `/book/{id}` could represent a specific book by ID.
- **Standard HTTP Methods**: REST APIs use the following HTTP methods:
  - **GET**: Retrieve data.
  - **POST**: Create new data.
  - **PUT**: Update existing data.
  - **DELETE**: Remove data.
- **JSON**: REST APIs often exchange data in the **JSON** format, which is lightweight and easy to parse.

### **Why Use REST APIs?**

- **Statelessness**: The server doesn't need to remember anything about the client, making each request more independent and scalable.
- **Scalability**: REST APIs are easy to scale, as servers can handle each request independently.
- **Flexibility**: REST APIs support a wide range of data formats and can be consumed by a variety of clients, from web browsers to mobile apps.

In this chapter, we'll walk you through the process of building your first REST API in Go.

---

## **20.2. Setting Up Your First REST API in Go**

Before diving into coding, let’s understand how to **run** and **test** a Go-based REST API, which we’ll be building throughout this tutorial.

### **Running the Go API**

1. **Install Go**: Make sure that Go is installed on your machine. If it isn't, you can follow the instructions here: [Install Go](https://golang.org/doc/install).

2. **Create a New Go File**: Create a file called `main.go`.

3. **Write the Code**: In the `main.go` file, copy the code provided in the examples throughout this chapter.

4. **Run the Application**:
   After saving the code, navigate to your project directory and run the Go file with the following command:

   ```bash
   go run main.go
   ```

   This will start the server on port `8080`. You’ll see output in the terminal like this:

   ```bash
   Server running on port 8080
   ```

5. **Access the API**: You can access the API in your browser by visiting `http://localhost:8080`.

---

### **Testing the API**

You can test the API in various ways:

1. **Using a Browser**:
   - For **GET** requests, simply enter the URL in the browser.
   - Example: To list all books, navigate to `http://localhost:8080/books`.

2. **Using Postman**: Postman is an API testing tool that allows you to send different HTTP requests like GET, POST, PUT, and DELETE.

3. **Using curl**: `curl` is a command-line tool for transferring data using URLs.

   - Example to test `GET /books`:
     ```bash
     curl http://localhost:8080/books
     ```

   - Example to test `POST /book/add`:
     ```bash
     curl -X POST -H "Content-Type: application/json" -d '{"title": "Go Web Dev", "author": "John Smith"}' http://localhost:8080/book/add
     ```

---

## **20.3. Understanding JSON in REST APIs**

### **What is JSON?**

JSON (JavaScript Object Notation) is a lightweight data-interchange format that is easy for humans to read and write and easy for machines to parse and generate. JSON is widely used for data exchange in REST APIs because of its simplicity and ease of integration.

A JSON object is a collection of key/value pairs, and it can represent various data structures. Here's an example of a simple JSON object:

```json
{
  "id": 1,
  "title": "Go Web Development",
  "author": "John Smith"
}
```

### **Why Use JSON in APIs?**

- **Lightweight**: JSON is text-based and lightweight, making it easy to transmit over the network.
- **Language Agnostic**: JSON can be used across different programming languages, making it easy for clients and servers to communicate.
- **Easy to Parse**: JSON is natively supported by most modern programming languages, including Go.

In Go, you can easily encode and decode JSON using the `encoding/json` package.

### **Go Example - JSON Handling**

Let’s see how we handle JSON in the Go API using the built-in `encoding/json` package.

```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
)

type Book struct {
	ID     int    `json:"id"`
	Title  string `json:"title"`
	Author string `json:"author"`
}

var books []Book

// This handler encodes a book struct into JSON and sends it as a response.
func getBooks(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(books)
}

func main() {
	books = append(books, Book{ID: 1, Title: "Go Web Dev", Author: "John Smith"})
	books = append(books, Book{ID: 2, Title: "Learning Go", Author: "Jane Doe"})

	http.HandleFunc("/books", getBooks)

	fmt.Println("Server running on port 8080")
	http.ListenAndServe(":8080", nil)
}
```

### **Explanation**:
- We define a `Book` struct with `json` tags to specify the keys in the resulting JSON.
- The handler `getBooks` encodes the `books` slice into JSON and sends it as a response with the `Content-Type: application/json` header.
- The `json.NewEncoder(w).Encode(books)` converts the `books` slice into JSON and writes it to the response.

---

## **20.4. Running and Testing the API**

### **Step-by-Step Guide to Running and Testing the API**

1. **Run the API**: First, save the code in a file called `main.go`. Then run it:

   ```bash
   go run main.go
   ```

2. **Testing the GET Request**:
   Open your browser or use `curl` to make a **GET** request:

   ```bash
   curl http://localhost:8080/books
   ```

   **Expected Output**:
   ```json
   [
     {
       "id": 1,
       "title": "Go Web Dev",
       "author": "John Smith"
     },
     {
       "id": 2,
       "title": "Learning Go",
       "author": "Jane Doe"
     }
   ]
   ```

3. **Testing the POST Request** (Add a new book):
   You can use `curl` or Postman to test **POST** requests.

   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"title": "Go Mastery", "author": "Alice Johnson"}' http://localhost:8080/book/add
   ```

   **Expected Output**:
   ```json
   {
     "id": 3,
     "title": "Go Mastery",
     "author": "Alice Johnson"
   }
   ```

4. **Testing the PUT Request** (Update a book):
   Use a **PUT** request to update the book's data.

   ```bash
   curl -X PUT -H "Content-Type: application/json" -d '{"id": 1, "title": "Advanced Go Web", "author": "John Smith"}' http://localhost:8080/book/update?id=1
   ```

   **Expected Output**:
   ```json
   {
     "id": 1,
     "title": "Advanced Go Web",
     "author": "John Smith"
   }
   ```

5. **Testing the DELETE Request** (Delete a book):
   Use a **DELETE** request to remove a book.

   ```bash
   curl -X DELETE http://localhost:8080/book/delete?id=2
   ```

---

## **20.5. Middleware in Go REST APIs**

Middleware in Go is a function that takes an HTTP request and response and either processes the request or passes it to the next handler in the chain. Middleware can be used for logging, authentication, etc.

### **Example: Logging Middleware**

Here’s an example of middleware that logs each incoming request:

```go
func loggingMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Request received: ", r.Method, r.URL)
		next.ServeHTTP(w, r)
	})
}
```

To apply the middleware:

```go
http.Handle("/books", loggingMiddleware(http.HandlerFunc(getBooks)))
```

### **Explanation**:
- The `loggingMiddleware` function wraps the `getBooks` handler and logs every incoming request.
- The `next.ServeHTTP(w, r)` calls the next handler in the chain (which is `getBooks` in this case).

---

## **20.6. Securing Your API with JWT Authentication**

JWT (JSON Web Tokens) is a popular method for securing APIs. With JWT, clients authenticate using a token, which is sent with every subsequent request.

### **Step 1: Install the JWT Package**

```bash
go get github.com/dgrijalva/jwt-go
```

### **Step 2: Create JWT Middleware**

JWT middleware checks for a valid token in the `Authorization` header:

```go
func jwtMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		tokenString := r.Header.Get("Authorization")
		if tokenString == "" {
			http.Error(w, "Forbidden", http.StatusForbidden)
			return
		}
		// Validate JWT token here (e.g., using jwt.Parse)
		_, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			return []byte("your-secret-key"), nil
		})
		if err != nil {
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}
		next.ServeHTTP(w, r)
	})
}
```

### **Step 3: Use JWT Middleware**

Apply the JWT middleware to your route:

```go
http.Handle("/books", jwtMiddleware(http.HandlerFunc(getBooks)))
```

---

## **20.7. Best Practices for Go REST APIs**

When developing REST APIs, following best practices can ensure your API is scalable, maintainable, and secure:

- **Version Your API**: Use versioning in your URLs to allow future changes without breaking existing clients. E.g., `/api/v1/books`.
- **Use Appropriate HTTP Status Codes**: Return proper HTTP status codes to indicate the success or failure of an API request.
- **Error Handling**: Always return meaningful error messages along with appropriate HTTP status codes.
- **Rate Limiting**: Implement rate limiting to avoid abuse of your API.
- **Logging**: Log important information such as request details for debugging and monitoring.

---

## **20.8. Conclusion**

In this chapter, we built a full-fledged **REST API** using Go, covering all the essential concepts such as:
- **Handling CRUD operations**.
- **Interacting with JSON data**.
- **Securing the API with JWT**.
- **Testing the API** using tools like curl, Postman, and a browser.
- **Using middleware** for logging and authentication.

By following the steps in this chapter, you now have a solid foundation for creating powerful REST APIs in Go.

---

### **Next Steps:**
- Expand your API with **pagination** and **sorting**.
- Add **user authentication** with JWT.
- Implement a **database** backend (e.g., with GORM).
