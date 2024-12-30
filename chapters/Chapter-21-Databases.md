# **Chapter 21: Working with Databases in Go**

---

## **21.1. Introduction**

In modern application development, databases are essential for storing and managing data. In this chapter, we will extend the REST API developed in the previous chapter by integrating a database. We will use **PostgreSQL** with the **GORM** library, a powerful Object-Relational Mapper (ORM) for Go.

By the end of this chapter, you will be able to:

- Set up and connect a database to a Go project.
- Perform CRUD (Create, Read, Update, Delete) operations using GORM.
- Understand migrations and database schema evolution.
- Apply best practices for database interaction.

---

## **21.2. Setting Up PostgreSQL**

### **What is PostgreSQL?**

**PostgreSQL** is a powerful, open-source relational database system that is widely used for production-grade applications. It offers advanced features like support for JSON, indexing, and robust transaction management.

### **Installing Dependencies**

1. Install GORM and the PostgreSQL driver:

   ```bash
   go get -u gorm.io/gorm
   go get -u gorm.io/driver/postgres
   ```

2. Verify the installation:
   ```bash
   go list -m all
   ```

---

## **21.3. Extending the REST API with a Database**

### **Scenario: Building a Book Management API**

We will extend the REST API from the previous chapter to store book data in a PostgreSQL database. Users will be able to:

1. Add a new book.
2. List all books.
3. Update book details.
4. Delete a book.

---

### **Step 1: Define the Book Model**

The `Book` model will represent the schema of the database table.

```go
package main

import "gorm.io/gorm"

// Book model
type Book struct {
    ID     uint   `gorm:"primaryKey"`
    Title  string `gorm:"not null"`
    Author string `gorm:"not null"`
    Year   int    `gorm:"not null"`
}
```

---

### **Step 2: Connect to the Database**

Initialize a connection to the PostgreSQL database and perform migrations to ensure the database schema matches the `Book` model.

```go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "log"
)

var db *gorm.DB

func initDatabase() {
    var err error
    dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
    db, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Run migrations
    db.AutoMigrate(&Book{})
    log.Println("Database connected and migrated")
}
```

- **`gorm.Open`**: Connects to the PostgreSQL database using a Data Source Name (DSN).
- **`db.AutoMigrate`**: Automatically creates or updates the database schema.

Call `initDatabase()` from the `main` function to initialize the database when the application starts.

---

### **Step 3: Update REST API Handlers**

We will use GORM functions to interact with the database in our REST API handlers.

#### **Add a New Book**

```go
import (
    "encoding/json"
    "net/http"
)

func createBookHandler(w http.ResponseWriter, r *http.Request) {
    var book Book
    if err := json.NewDecoder(r.Body).Decode(&book); err != nil {
        http.Error(w, "Invalid request body", http.StatusBadRequest)
        return
    }

    db.Create(&book)
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(book)
}
```

---

#### **List All Books**

```go
func listBooksHandler(w http.ResponseWriter, r *http.Request) {
    var books []Book
    db.Find(&books)

    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(books)
}
```

---

#### **Update a Book**

```go
func updateBookHandler(w http.ResponseWriter, r *http.Request) {
    var book Book
    if err := json.NewDecoder(r.Body).Decode(&book); err != nil {
        http.Error(w, "Invalid request body", http.StatusBadRequest)
        return
    }

    result := db.Model(&Book{}).Where("id = ?", book.ID).Updates(book)
    if result.RowsAffected == 0 {
        http.Error(w, "Book not found", http.StatusNotFound)
        return
    }

    w.WriteHeader(http.StatusOK)
    json.NewEncoder(w).Encode(book)
}
```

---

#### **Delete a Book**

```go
func deleteBookHandler(w http.ResponseWriter, r *http.Request) {
    id := r.URL.Query().Get("id")
    result := db.Delete(&Book{}, id)
    if result.RowsAffected == 0 {
        http.Error(w, "Book not found", http.StatusNotFound)
        return
    }

    w.WriteHeader(http.StatusNoContent)
}
```

---

### **Step 4: Define API Routes**

Update the `main` function to include the new routes.

```go
func main() {
    // Initialize database
    initDatabase()

    http.HandleFunc("/books", listBooksHandler)
    http.HandleFunc("/book/add", createBookHandler)
    http.HandleFunc("/book/update", updateBookHandler)
    http.HandleFunc("/book/delete", deleteBookHandler)

    log.Println("Server running on port 8080")
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## **21.4. Testing the API**

### **Running the Application**

1. Save the code in `main.go`.
2. Run the application:
   ```bash
   go run main.go
   ```
3. The server will start at `http://localhost:8080`.

---

### **Testing with curl**

1. **Add a New Book**:

   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"title":"The Go Programming Language","author":"Alan A. A. Donovan","year":2015}' http://localhost:8080/book/add
   ```

2. **List All Books**:

   ```bash
   curl http://localhost:8080/books
   ```

3. **Update a Book**:

   ```bash
   curl -X PUT -H "Content-Type: application/json" -d '{"id":1,"title":"The Go Programming Language","author":"Alan Donovan","year":2016}' http://localhost:8080/book/update
   ```

4. **Delete a Book**:
   ```bash
   curl -X DELETE http://localhost:8080/book/delete?id=1
   ```

---

## **21.5. Best Practices**

### **Error Handling**

Always check the result of database operations and handle errors appropriately to avoid data corruption and crashes.

### **Validations**

Use GORM's validation hooks to enforce data integrity. For example:

```go
type Book struct {
    ID     uint   `gorm:"primaryKey"`
    Title  string `gorm:"not null"`
    Author string `gorm:"not null"`
    Year   int    `gorm:"not null"`
}
```

### **Pagination**

For large datasets, implement pagination to limit the number of records returned.

```go
func listBooksPaginated(w http.ResponseWriter, r *http.Request) {
    var books []Book
    page := 1 // Fetch page from query params
    limit := 10

    db.Offset((page - 1) * limit).Limit(limit).Find(&books)
    json.NewEncoder(w).Encode(books)
}
```

---



# **Exercises**

---

## **Exercise 1: Setup a PostgreSQL Database**

**Task**: Set up a PostgreSQL database called `library` and connect it to a Go project. Set up a `Book` model with fields for:

- Title
- Author
- PublishedYear

**Code Solution**:

```go
package main

import (
	"log"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

type Book struct {
	ID            uint   `gorm:"primaryKey"`
	Title         string `gorm:"not null"`
	Author        string `gorm:"not null"`
	PublishedYear int    `gorm:"not null"`
}

func main() {
	// Replace DSN with your PostgreSQL credentials
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}
	log.Println("Database connected!")
}
```

**Objective**: Practice setting up a GORM database connection with PostgreSQL and defining a model.

---

## **Exercise 2: Migrate a Schema**

**Task**: Use GORM's `AutoMigrate` method to create a `books` table in the PostgreSQL `library` database from the `Book` model.

**Code Solution**:

```go
func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	err = db.AutoMigrate(&Book{})
	if err != nil {
		log.Fatal("Failed to migrate database schema:", err)
	}

	log.Println("Database schema migrated!")
}
```

**Objective**: Understand how to translate Go models into PostgreSQL database schemas using migrations.

---

## **Exercise 3: Add Seed Data**

**Task**: Insert the following books into the PostgreSQL database:

| Title                         | Author           | PublishedYear |
| ----------------------------- | ---------------- | ------------- |
| "The Go Programming Language" | Alan Donovan     | 2015          |
| "Clean Code"                  | Robert C. Martin | 2008          |
| "Design Patterns"             | Erich Gamma      | 1994          |

**Code Solution**:

```go
func seedBooks(db *gorm.DB) {
	books := []Book{
		{Title: "The Go Programming Language", Author: "Alan Donovan", PublishedYear: 2015},
		{Title: "Clean Code", Author: "Robert C. Martin", PublishedYear: 2008},
		{Title: "Design Patterns", Author: "Erich Gamma", PublishedYear: 1994},
	}

	for _, book := range books {
		db.Create(&book)
	}
	log.Println("Books seeded successfully!")
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	db.AutoMigrate(&Book{})
	seedBooks(db)
}
```

**Objective**: Practice creating records with GORM in PostgreSQL.

---

## **Exercise 4: Query Data**

**Task**: Write a function to retrieve and display all books from the database.

**Code Solution**:

```go
func getAllBooks(db *gorm.DB) {
	var books []Book
	db.Find(&books)
	log.Println("Books:", books)
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	getAllBooks(db)
}
```

**Objective**: Learn to query all records from a PostgreSQL table.

---

## **Exercise 5: Query Specific Records**

**Task**: Write a function to find all books authored by "Robert C. Martin".

**Code Solution**:

```go
func getBooksByAuthor(db *gorm.DB, author string) {
	var books []Book
	db.Where("author = ?", author).Find(&books)
	log.Printf("Books by %s: %v", author, books)
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	getBooksByAuthor(db, "Robert C. Martin")
}
```

**Objective**: Understand how to filter records in PostgreSQL using GORM.

---

## **Exercise 6: Update Records**

**Task**: Write a function to update the `PublishedYear` of the book "Clean Code" to `2010`.

**Code Solution**:

```go
func updateBookYear(db *gorm.DB, title string, year int) {
	db.Model(&Book{}).Where("title = ?", title).Update("published_year", year)
	log.Printf("Updated year of '%s' to %d", title, year)
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	updateBookYear(db, "Clean Code", 2010)
}
```

**Objective**: Practice updating records in PostgreSQL with GORM.

---

## **Exercise 7: Delete Records**

**Task**: Write a function to delete the book "Design Patterns" from the database.

**Code Solution**:

```go
func deleteBook(db *gorm.DB, title string) {
	db.Where("title = ?", title).Delete(&Book{})
	log.Printf("Deleted book titled '%s'", title)
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	deleteBook(db, "Design Patterns")
}
```

**Objective**: Learn how to remove records using GORM with PostgreSQL.

---

## **Exercise 8: Add Pagination**

**Task**: Implement pagination for the `books` table. Fetch records in pages of size `2`.

**Code Solution**:

```go
func getBooksPaginated(db *gorm.DB, page, limit int) {
	var books []Book
	db.Offset((page - 1) * limit).Limit(limit).Find(&books)
	log.Printf("Page %d: %v", page, books)
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	getBooksPaginated(db, 1, 2)
}
```

**Objective**: Understand how to use `Limit` and `Offset` for pagination in GORM with PostgreSQL.

---

## **Exercise 9: Add a Relationship**

**Task**: Create an `Author` model and establish a one-to-many relationship between `Author` and `Book`.

**Code Solution**:

```go
type Author struct {
	ID    uint   `gorm:"primaryKey"`
	Name  string `gorm:"not null"`
	Books []Book `gorm:"foreignKey:AuthorID"`
}

type Book struct {
	ID            uint   `gorm:"primaryKey"`
	Title         string `gorm:"not null"`
	AuthorID      uint
	PublishedYear int
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	db.AutoMigrate(&Author{}, &Book{})
	log.Println("Relationship established!")
}
```

**Objective**: Learn to define relationships in PostgreSQL with GORM.

---

## **Exercise 10: Use Transactions**

**Task**: Write a function to create an author and add two books for that author in a transaction.

**Code Solution**:

```go
func createAuthorWithBooks(db *gorm.DB) {
	err := db.Transaction(func(tx *gorm.DB) error {
		author := Author{Name: "J.K. Rowling"}
		if err := tx.Create(&author).Error; err != nil {
			return err
		}

		books := []Book{
			{Title: "Harry Potter and the Philosopher's Stone", AuthorID: author.ID},
			{Title: "Harry Potter and the Chamber of Secrets", AuthorID: author.ID},
		}

		for _, book := range books {
			if err := tx.Create(&book).Error; err != nil {
				return err
			}
		}

		return nil
	})

	if err != nil {
		log.Println("Transaction failed:", err)
	} else {
		log.Println("Transaction successful!")
	}
}

func main() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=library port=5432 sslmode=disable"
	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}

	createAuthorWithBooks(db)
}
```

**Objective**: Practice using transactions in PostgreSQL with GORM.

---

## **Submission**

Once you complete these exercises:

- Ensure all functions work correctly with GORM.
- Test your implementation with actual PostgreSQL data and queries.

---

### **Bonus Challenge**

Build a REST API for the `books` table:

- `GET /books`: List all books.
- `POST /books`: Add a new book.
- `PUT /books/{id}`: Update a book by ID.
- `DELETE /books/{id}`: Delete a book by ID.

This will help you integrate REST API development from Chapter 20 with database management using PostgreSQL from Chapter 21.
