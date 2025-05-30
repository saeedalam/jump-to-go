# **Chapter 20: Working with Databases in Go**

Go's strong performance characteristics and concurrency support make it an excellent choice for building database-driven applications. This chapter explores how to work with various database systems in Go, from SQL databases to NoSQL solutions, covering essential patterns and best practices for data persistence.

## **20.1 Introduction to Database Programming in Go**

### **20.1.1 Database Interactions in Go**

Go's approach to database programming emphasizes simplicity and performance. The standard library provides core functionality through the `database/sql` package, which offers a generic interface for SQL databases. Third-party drivers implement this interface for specific database systems.

Key benefits of Go for database applications include:

1. **Concurrency**: Goroutines and channels provide efficient management of concurrent database connections
2. **Type Safety**: Strong typing helps prevent many SQL injection vulnerabilities
3. **Performance**: Low memory footprint and fast execution speed
4. **Simplicity**: Clean syntax and standard interfaces make database code easy to understand

### **20.1.2 Types of Databases**

When working with Go, you can choose from various database types:

| Database Type     | Examples                  | Best Used For                                             |
| ----------------- | ------------------------- | --------------------------------------------------------- |
| Relational (SQL)  | PostgreSQL, MySQL, SQLite | Structured data with relationships, ACID transactions     |
| Document-oriented | MongoDB, CouchDB          | Semi-structured data, flexible schemas                    |
| Key-Value         | Redis, etcd               | Caching, configuration, simple data structures            |
| Wide-column       | Cassandra, ScyllaDB       | Time-series data, large datasets with predictable queries |
| Graph             | Neo4j, DGraph             | Highly connected data with complex relationships          |
| Time-series       | InfluxDB, TimescaleDB     | Metrics, monitoring data, IoT data                        |

### **20.1.3 Database Access Patterns**

Several patterns are commonly used when working with databases in Go:

1. **Direct SQL**: Using raw SQL queries with `database/sql`
2. **Query Builders**: Libraries that help construct SQL programmatically
3. **Object-Relational Mappers (ORMs)**: Map database tables to Go structs
4. **Repository Pattern**: Abstract database operations behind interfaces
5. **CQRS (Command Query Responsibility Segregation)**: Separate read and write operations

Each pattern has trade-offs in terms of control, simplicity, and performance. We'll explore these throughout the chapter.

## **20.2 Working with SQL Databases**

### **20.2.1 The database/sql Package**

The `database/sql` package provides a generic interface around SQL (or SQL-like) databases. It manages connections and transactions while allowing specific SQL dialect usage.

First, you need to import the package and a database driver:

```go
package main

import (
    "database/sql"
    "fmt"
    "log"

    _ "github.com/lib/pq" // PostgreSQL driver
)

func main() {
    // Open a database connection
    db, err := sql.Open("postgres", "postgres://username:password@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Test the connection
    if err := db.Ping(); err != nil {
        log.Fatal(err)
    }

    fmt.Println("Successfully connected to the database!")
}
```

Key components of the `database/sql` package:

- **`sql.DB`**: A database handle representing a connection pool
- **`sql.Tx`**: A transaction
- **`sql.Stmt`**: A prepared statement
- **`sql.Rows`**: Result set from a query
- **`sql.Row`**: Single row result from a query

### **20.2.2 Basic CRUD Operations**

Let's implement basic CRUD (Create, Read, Update, Delete) operations with `database/sql`:

```go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "time"

    _ "github.com/lib/pq"
)

// User represents a user in our application
type User struct {
    ID        int
    Username  string
    Email     string
    CreatedAt time.Time
}

func main() {
    db, err := sql.Open("postgres", "postgres://username:password@localhost/dbname?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Create a user
    user, err := createUser(db, "johndoe", "john@example.com")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Created user: %+v\n", user)

    // Read a user
    retrievedUser, err := getUserByID(db, user.ID)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Retrieved user: %+v\n", retrievedUser)

    // Update a user
    err = updateUserEmail(db, user.ID, "newemail@example.com")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("User email updated")

    // Delete a user
    err = deleteUser(db, user.ID)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("User deleted")
}

// createUser inserts a new user into the database
func createUser(db *sql.DB, username, email string) (User, error) {
    var user User

    query := `
        INSERT INTO users (username, email, created_at)
        VALUES ($1, $2, $3)
        RETURNING id, username, email, created_at
    `

    err := db.QueryRow(query, username, email, time.Now()).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.CreatedAt,
    )

    return user, err
}

// getUserByID retrieves a user by their ID
func getUserByID(db *sql.DB, id int) (User, error) {
    var user User

    query := `
        SELECT id, username, email, created_at
        FROM users
        WHERE id = $1
    `

    err := db.QueryRow(query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.CreatedAt,
    )

    return user, err
}

// updateUserEmail updates a user's email
func updateUserEmail(db *sql.DB, id int, email string) error {
    query := `
        UPDATE users
        SET email = $1
        WHERE id = $2
    `

    _, err := db.Exec(query, email, id)
    return err
}

// deleteUser removes a user from the database
func deleteUser(db *sql.DB, id int) error {
    query := `
        DELETE FROM users
        WHERE id = $1
    `

    _, err := db.Exec(query, id)
    return err
}
```

### **20.2.3 Handling Multiple Result Rows**

When a query returns multiple rows, use the `Query` method and iterate through the results:

```go
// getAllUsers retrieves all users from the database
func getAllUsers(db *sql.DB) ([]User, error) {
    users := []User{}

    query := `
        SELECT id, username, email, created_at
        FROM users
        ORDER BY id
    `

    rows, err := db.Query(query)
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt); err != nil {
            return nil, err
        }
        users = append(users, user)
    }

    if err := rows.Err(); err != nil {
        return nil, err
    }

    return users, nil
}
```

Important points when working with rows:

- Always call `rows.Close()` when done (use `defer` to ensure this happens)
- Check for errors with `rows.Err()` after the loop
- Use `rows.Next()` to advance to the next row
- Call `rows.Scan()` to read the current row's values

### **20.2.4 Prepared Statements**

Prepared statements improve performance and security by separating SQL logic from data:

```go
// getUsersByUsernamePattern finds users with usernames matching a pattern
func getUsersByUsernamePattern(db *sql.DB, pattern string) ([]User, error) {
    users := []User{}

    // Prepare the statement
    stmt, err := db.Prepare(`
        SELECT id, username, email, created_at
        FROM users
        WHERE username LIKE $1
        ORDER BY username
    `)
    if err != nil {
        return nil, err
    }
    defer stmt.Close()

    // Execute the prepared statement
    rows, err := stmt.Query("%" + pattern + "%")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    for rows.Next() {
        var user User
        if err := rows.Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt); err != nil {
            return nil, err
        }
        users = append(users, user)
    }

    if err := rows.Err(); err != nil {
        return nil, err
    }

    return users, nil
}
```

Benefits of prepared statements:

- **Security**: Help prevent SQL injection attacks
- **Performance**: Database can optimize and reuse execution plans
- **Readability**: Separate SQL logic from data values

### **20.2.5 Transactions**

Transactions ensure multiple operations succeed or fail as a unit:

```go
// transferCredits transfers credits between users atomically
func transferCredits(db *sql.DB, fromUserID, toUserID, amount int) error {
    // Begin transaction
    tx, err := db.Begin()
    if err != nil {
        return err
    }

    // Defer a rollback in case anything fails
    defer tx.Rollback()

    // Deduct from first user
    _, err = tx.Exec(`
        UPDATE users
        SET credits = credits - $1
        WHERE id = $2
    `, amount, fromUserID)
    if err != nil {
        return err
    }

    // Add to second user
    _, err = tx.Exec(`
        UPDATE users
        SET credits = credits + $1
        WHERE id = $2
    `, amount, toUserID)
    if err != nil {
        return err
    }

    // Commit the transaction
    return tx.Commit()
}
```

Key aspects of transactions:

- Begin with `db.Begin()`
- Always defer `tx.Rollback()` to ensure cleanup
- Call `tx.Commit()` when all operations succeed
- Use transactions when multiple operations need to be atomic

### **20.2.6 Connection Management**

The `sql.DB` object represents a pool of database connections. It's important to configure this pool correctly:

```go
func setupDBConnection() (*sql.DB, error) {
    db, err := sql.Open("postgres", "postgres://username:password@localhost/dbname?sslmode=disable")
    if err != nil {
        return nil, err
    }

    // Set maximum number of open connections
    db.SetMaxOpenConns(25)

    // Set maximum number of idle connections
    db.SetMaxIdleConns(5)

    // Set maximum lifetime of a connection
    db.SetConnMaxLifetime(5 * time.Minute)

    // Verify connection
    if err := db.Ping(); err != nil {
        return nil, err
    }

    return db, nil
}
```

Connection pool configuration considerations:

- **MaxOpenConns**: Limits the number of connections to the database
- **MaxIdleConns**: Controls how many connections remain open when idle
- **ConnMaxLifetime**: Limits the maximum amount of time a connection may be reused

It's important to close the database when your application shuts down:

```go
func main() {
    db, err := setupDBConnection()
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Use the database...
}
```

## **20.3 Working with PostgreSQL**

PostgreSQL is a powerful open-source relational database system with advanced features. Go works particularly well with PostgreSQL.

### **20.3.1 Connecting to PostgreSQL**

To connect to PostgreSQL, you'll need the `pq` driver:

```go
import (
    "database/sql"
    _ "github.com/lib/pq"
)

func connectToPG() (*sql.DB, error) {
    connStr := "user=postgres dbname=myapp password=secret host=localhost port=5432 sslmode=disable"
    db, err := sql.Open("postgres", connStr)
    if err != nil {
        return nil, err
    }

    if err := db.Ping(); err != nil {
        return nil, err
    }

    return db, nil
}
```

You can also use the connection URL format:

```go
connStr := "postgres://postgres:secret@localhost:5432/myapp?sslmode=disable"
```

### **20.3.2 PostgreSQL-Specific Features**

PostgreSQL offers many advanced features that Go applications can leverage:

#### **JSON Data**

PostgreSQL has excellent support for JSON data:

```go
// User with JSON metadata
type User struct {
    ID       int
    Username string
    Email    string
    Metadata map[string]interface{}
}

// Store user with JSON metadata
func createUserWithMetadata(db *sql.DB, user User) error {
    metadataJSON, err := json.Marshal(user.Metadata)
    if err != nil {
        return err
    }

    _, err = db.Exec(`
        INSERT INTO users (username, email, metadata)
        VALUES ($1, $2, $3)
    `, user.Username, user.Email, metadataJSON)

    return err
}

// Retrieve user with JSON metadata
func getUserWithMetadata(db *sql.DB, id int) (User, error) {
    var user User
    var metadataJSON []byte

    err := db.QueryRow(`
        SELECT id, username, email, metadata
        FROM users
        WHERE id = $1
    `, id).Scan(&user.ID, &user.Username, &user.Email, &metadataJSON)

    if err != nil {
        return User{}, err
    }

    // Parse JSON metadata
    user.Metadata = make(map[string]interface{})
    if err := json.Unmarshal(metadataJSON, &user.Metadata); err != nil {
        return User{}, err
    }

    return user, nil
}
```

#### **Array Types**

PostgreSQL supports array types, which can be mapped to Go slices:

```go
// User with string array of roles
type User struct {
    ID    int
    Name  string
    Roles []string
}

// Store user with roles
func createUserWithRoles(db *sql.DB, user User) error {
    _, err := db.Exec(`
        INSERT INTO users (name, roles)
        VALUES ($1, $2)
    `, user.Name, pq.Array(user.Roles))

    return err
}

// Retrieve user with roles
func getUserWithRoles(db *sql.DB, id int) (User, error) {
    var user User

    err := db.QueryRow(`
        SELECT id, name, roles
        FROM users
        WHERE id = $1
    `, id).Scan(&user.ID, &user.Name, pq.Array(&user.Roles))

    return user, err
}
```

#### **Full-Text Search**

PostgreSQL has powerful full-text search capabilities:

```go
// Search products by terms
func searchProducts(db *sql.DB, searchTerms string) ([]Product, error) {
    products := []Product{}

    rows, err := db.Query(`
        SELECT id, name, description, price
        FROM products
        WHERE to_tsvector('english', name || ' ' || description) @@ to_tsquery('english', $1)
        ORDER BY ts_rank(to_tsvector('english', name || ' ' || description), to_tsquery('english', $1)) DESC
    `, searchTerms)

    if err != nil {
        return nil, err
    }
    defer rows.Close()

    for rows.Next() {
        var product Product
        if err := rows.Scan(&product.ID, &product.Name, &product.Description, &product.Price); err != nil {
            return nil, err
        }
        products = append(products, product)
    }

    return products, rows.Err()
}
```

## **20.4 Object-Relational Mapping (ORM)**

While raw SQL offers maximum control, ORMs can simplify database operations by mapping database records to Go structs.

### **20.4.1 Using GORM**

GORM is a popular ORM library for Go. Here's how to use it:

```go
package main

import (
    "fmt"
    "log"
    "time"

    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

// User model
type User struct {
    ID        uint      `gorm:"primaryKey"`
    Username  string    `gorm:"size:100;not null;unique"`
    Email     string    `gorm:"size:100;not null;unique"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`

    // Has many relationship
    Posts []Post
}

// Post model
type Post struct {
    ID        uint   `gorm:"primaryKey"`
    Title     string `gorm:"size:200;not null"`
    Content   string `gorm:"type:text"`
    UserID    uint
    CreatedAt time.Time
    UpdatedAt time.Time
}

func main() {
    // Connect to database
    dsn := "user=postgres password=secret dbname=blog host=localhost port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Auto-migrate schemas
    db.AutoMigrate(&User{}, &Post{})

    // Create a user
    user := User{
        Username: "johndoe",
        Email:    "john@example.com",
    }

    result := db.Create(&user)
    if result.Error != nil {
        log.Fatal("Failed to create user:", result.Error)
    }

    fmt.Printf("Created user with ID: %d\n", user.ID)

    // Create a post for the user
    post := Post{
        Title:   "My First Post",
        Content: "This is the content of my first post.",
        UserID:  user.ID,
    }

    db.Create(&post)

    // Query with relationships
    var userWithPosts User
    db.Preload("Posts").First(&userWithPosts, user.ID)

    fmt.Printf("User: %s has %d posts\n", userWithPosts.Username, len(userWithPosts.Posts))

    // Update user
    db.Model(&user).Updates(User{
        Email: "newemail@example.com",
    })

    // Delete user (soft delete with DeletedAt)
    db.Delete(&user)
}
```

GORM supports:

- Automatic migrations
- Associations (one-to-one, one-to-many, many-to-many)
- Hooks and callbacks
- Soft deletes
- Eager loading
- Transactions
- Scopes for reusable queries

### **20.4.2 Using SQLx**

SQLx extends the standard `database/sql` package with additional functionality while staying close to raw SQL:

```go
package main

import (
    "fmt"
    "log"
    "time"

    "github.com/jmoiron/sqlx"
    _ "github.com/lib/pq"
)

// User struct with struct tags for mapping
type User struct {
    ID        int       `db:"id"`
    Username  string    `db:"username"`
    Email     string    `db:"email"`
    CreatedAt time.Time `db:"created_at"`
}

func main() {
    // Connect to database
    db, err := sqlx.Connect("postgres", "user=postgres password=secret dbname=myapp host=localhost sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Create schema
    schema := `
        CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            username TEXT NOT NULL UNIQUE,
            email TEXT NOT NULL UNIQUE,
            created_at TIMESTAMP NOT NULL DEFAULT NOW()
        );
    `
    db.MustExec(schema)

    // Insert a user
    result, err := db.Exec(
        "INSERT INTO users (username, email, created_at) VALUES ($1, $2, $3)",
        "johndoe",
        "john@example.com",
        time.Now(),
    )
    if err != nil {
        log.Fatal(err)
    }

    userID, _ := result.LastInsertId()
    fmt.Printf("Created user with ID: %d\n", userID)

    // Query a single user
    user := User{}
    err = db.Get(&user, "SELECT * FROM users WHERE username = $1", "johndoe")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Retrieved user: %+v\n", user)

    // Query multiple users
    users := []User{}
    err = db.Select(&users, "SELECT * FROM users ORDER BY username")
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Found %d users\n", len(users))

    // Named queries
    namedQuery := `
        SELECT * FROM users
        WHERE username = :username OR email = :email
    `
    params := map[string]interface{}{
        "username": "johndoe",
        "email":    "john@example.com",
    }

    rows, err := db.NamedQuery(namedQuery, params)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()

    for rows.Next() {
        var u User
        err := rows.StructScan(&u)
        if err != nil {
            log.Fatal(err)
        }
        fmt.Printf("User from named query: %+v\n", u)
    }
}
```

SQLx advantages:

- Minimal abstraction over `database/sql`
- Struct mapping with tags
- Named queries
- Better error handling
- Transaction management
- Support for multiple statement queries

## **20.5 Working with NoSQL Databases**

NoSQL databases are a good fit for many Go applications, especially those dealing with unstructured data or requiring high scalability.

### **20.5.1 MongoDB with Go**

MongoDB is a popular document-oriented database. Here's how to use it with Go:

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "go.mongodb.org/mongo-driver/bson"
    "go.mongodb.org/mongo-driver/bson/primitive"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

// User represents a user document in MongoDB
type User struct {
    ID        primitive.ObjectID `bson:"_id,omitempty"`
    Username  string             `bson:"username"`
    Email     string             `bson:"email"`
    CreatedAt time.Time          `bson:"created_at"`
    Metadata  map[string]interface{} `bson:"metadata,omitempty"`
}

func main() {
    // Connect to MongoDB
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
    defer cancel()

    client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
    if err != nil {
        log.Fatal(err)
    }
    defer client.Disconnect(ctx)

    // Check connection
    err = client.Ping(ctx, nil)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("Connected to MongoDB!")

    // Get a collection
    usersCollection := client.Database("myapp").Collection("users")

    // Insert a user
    user := User{
        Username:  "johndoe",
        Email:     "john@example.com",
        CreatedAt: time.Now(),
        Metadata: map[string]interface{}{
            "preferences": map[string]interface{}{
                "theme": "dark",
                "notifications": true,
            },
            "lastLogin": time.Now(),
        },
    }

    result, err := usersCollection.InsertOne(ctx, user)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Inserted user with ID: %v\n", result.InsertedID)

    // Query for a user
    var retrievedUser User
    err = usersCollection.FindOne(ctx, bson.M{"username": "johndoe"}).Decode(&retrievedUser)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Retrieved user: %+v\n", retrievedUser)

    // Update a user
    update := bson.M{
        "$set": bson.M{
            "email": "newemail@example.com",
            "metadata.preferences.theme": "light",
        },
    }

    updateResult, err := usersCollection.UpdateOne(
        ctx,
        bson.M{"username": "johndoe"},
        update,
    )
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Updated %v documents\n", updateResult.ModifiedCount)

    // Find multiple documents
    cursor, err := usersCollection.Find(ctx, bson.M{})
    if err != nil {
        log.Fatal(err)
    }
    defer cursor.Close(ctx)

    var users []User
    if err = cursor.All(ctx, &users); err != nil {
        log.Fatal(err)
    }

    for _, u := range users {
        fmt.Printf("Found user: %s (%s)\n", u.Username, u.Email)
    }

    // Delete a user
    deleteResult, err := usersCollection.DeleteOne(ctx, bson.M{"username": "johndoe"})
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Deleted %v documents\n", deleteResult.DeletedCount)
}
```

MongoDB with Go offers:

- Native BSON serialization
- Rich query capabilities
- Document-oriented storage for complex data
- Support for indexing and aggregations
- Transactions in recent versions

### **20.5.2 Redis with Go**

Redis is an in-memory key-value store often used for caching, session management, and message brokering:

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "log"
    "time"

    "github.com/go-redis/redis/v8"
)

// User represents a user in our application
type User struct {
    ID       int    `json:"id"`
    Username string `json:"username"`
    Email    string `json:"email"`
}

func main() {
    // Create Redis client
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "", // no password set
        DB:       0,  // use default DB
    })

    ctx := context.Background()

    // Test connection
    pong, err := rdb.Ping(ctx).Result()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Redis connection successful:", pong)

    // Store a string value
    err = rdb.Set(ctx, "key", "value", 0).Err()
    if err != nil {
        log.Fatal(err)
    }

    // Retrieve a string value
    val, err := rdb.Get(ctx, "key").Result()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("key:", val)

    // Store a user using JSON
    user := User{
        ID:       1,
        Username: "johndoe",
        Email:    "john@example.com",
    }

    userJSON, err := json.Marshal(user)
    if err != nil {
        log.Fatal(err)
    }

    err = rdb.Set(ctx, "user:1", userJSON, 24*time.Hour).Err()
    if err != nil {
        log.Fatal(err)
    }

    // Retrieve a user
    userJSON, err = rdb.Get(ctx, "user:1").Bytes()
    if err != nil {
        log.Fatal(err)
    }

    var retrievedUser User
    err = json.Unmarshal(userJSON, &retrievedUser)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Retrieved user: %+v\n", retrievedUser)

    // Increment a counter
    newVal, err := rdb.Incr(ctx, "counter").Result()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Counter: %d\n", newVal)

    // Working with a hash
    err = rdb.HSet(ctx, "user:hash:1", map[string]interface{}{
        "username": "johndoe",
        "email":    "john@example.com",
        "visits":   1,
    }).Err()
    if err != nil {
        log.Fatal(err)
    }

    // Increment a hash field
    newVisits, err := rdb.HIncrBy(ctx, "user:hash:1", "visits", 1).Result()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User visits: %d\n", newVisits)

    // Get all hash fields
    fields, err := rdb.HGetAll(ctx, "user:hash:1").Result()
    if err != nil {
        log.Fatal(err)
    }
    fmt.Printf("User hash: %v\n", fields)
}
```

Redis with Go is great for:

- Caching
- Session management
- Rate limiting
- Distributed locks
- Pub/Sub messaging
- Leaderboards and counters

## **20.6 Database Migrations**

As your application evolves, database schemas need to change. Migrations provide a way to manage these changes safely.

### **20.6.1 Using golang-migrate**

`golang-migrate` is a popular migration tool for Go projects:

```go
package main

import (
    "database/sql"
    "log"

    "github.com/golang-migrate/migrate/v4"
    "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
    _ "github.com/lib/pq"
)

func main() {
    // Connect to the database
    db, err := sql.Open("postgres", "postgres://username:password@localhost:5432/myapp?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }

    // Create a driver instance
    driver, err := postgres.WithInstance(db, &postgres.Config{})
    if err != nil {
        log.Fatal(err)
    }

    // Create a migrate instance
    m, err := migrate.NewWithDatabaseInstance(
        "file://migrations", // Migration files source
        "postgres",          // Database name
        driver,              // Database driver
    )
    if err != nil {
        log.Fatal(err)
    }

    // Apply all up migrations
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        log.Fatal(err)
    }

    log.Println("Migrations applied successfully")
}
```

To use `golang-migrate`, you create migration files in a directory (e.g., `migrations/`):

```
migrations/
  ├── 000001_create_users_table.up.sql
  ├── 000001_create_users_table.down.sql
  ├── 000002_add_status_to_users.up.sql
  └── 000002_add_status_to_users.down.sql
```

Example migration files:

```sql
-- 000001_create_users_table.up.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

```sql
-- 000001_create_users_table.down.sql
DROP TABLE users;
```

```sql
-- 000002_add_status_to_users.up.sql
ALTER TABLE users ADD COLUMN status VARCHAR(20) NOT NULL DEFAULT 'active';
```

```sql
-- 000002_add_status_to_users.down.sql
ALTER TABLE users DROP COLUMN status;
```

### **20.6.2 Embedded Migrations**

For simpler projects, you might embed migrations directly in your code:

```go
package main

import (
    "database/sql"
    "log"

    _ "github.com/lib/pq"
)

// Migrations to apply in order
var migrations = []string{
    `CREATE TABLE IF NOT EXISTS users (
        id SERIAL PRIMARY KEY,
        username VARCHAR(100) NOT NULL UNIQUE,
        email VARCHAR(100) NOT NULL UNIQUE,
        created_at TIMESTAMP NOT NULL DEFAULT NOW()
    )`,

    `CREATE TABLE IF NOT EXISTS posts (
        id SERIAL PRIMARY KEY,
        title VARCHAR(200) NOT NULL,
        content TEXT,
        user_id INTEGER REFERENCES users(id),
        created_at TIMESTAMP NOT NULL DEFAULT NOW()
    )`,

    `ALTER TABLE users ADD COLUMN IF NOT EXISTS status VARCHAR(20) NOT NULL DEFAULT 'active'`,
}

func applyMigrations(db *sql.DB) error {
    // Create migrations table if it doesn't exist
    _, err := db.Exec(`
        CREATE TABLE IF NOT EXISTS migrations (
            id SERIAL PRIMARY KEY,
            version INTEGER NOT NULL UNIQUE,
            applied_at TIMESTAMP NOT NULL DEFAULT NOW()
        )
    `)
    if err != nil {
        return err
    }

    // Get the last applied migration version
    var lastVersion int
    row := db.QueryRow("SELECT COALESCE(MAX(version), 0) FROM migrations")
    if err := row.Scan(&lastVersion); err != nil {
        return err
    }

    // Apply pending migrations
    for i, migration := range migrations {
        version := i + 1

        if version <= lastVersion {
            continue
        }

        // Start a transaction for this migration
        tx, err := db.Begin()
        if err != nil {
            return err
        }

        // Apply the migration
        if _, err = tx.Exec(migration); err != nil {
            tx.Rollback()
            return err
        }

        // Record the migration
        if _, err = tx.Exec("INSERT INTO migrations (version) VALUES ($1)", version); err != nil {
            tx.Rollback()
            return err
        }

        // Commit the transaction
        if err = tx.Commit(); err != nil {
            return err
        }

        log.Printf("Applied migration %d\n", version)
    }

    return nil
}

func main() {
    // Connect to the database
    db, err := sql.Open("postgres", "postgres://username:password@localhost:5432/myapp?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    // Apply migrations
    if err := applyMigrations(db); err != nil {
        log.Fatal(err)
    }

    log.Println("Migrations applied successfully")
}
```

## **20.7 Database Best Practices**

### **20.7.1 Connection Management**

- **Use connection pooling**: The `sql.DB` object already provides a connection pool
- **Set appropriate pool sizes**: Configure `MaxOpenConns` and `MaxIdleConns` based on your application's needs
- **Set connection lifetime**: Use `ConnMaxLifetime` to prevent stale connections
- **Check connections**: Use `db.Ping()` to verify the connection is still alive

### **20.7.2 Query Optimization**

- **Use prepared statements** for frequently executed queries
- **Implement pagination** for large result sets
- **Choose appropriate indexes** for your queries
- **Use database-specific features** like JSON functions when beneficial
- **Optimize large transactions** by breaking them into smaller ones when possible

### **20.7.3 Security**

- **Never concatenate user input into SQL queries**: Use parameterized queries
- **Use connection strings securely**: Store credentials in environment variables or a secure vault
- **Implement proper access control**: Use database roles with minimal privileges
- **Encrypt sensitive data**: Consider column-level encryption for PII

### **20.7.4 Testing**

- **Use database mocks** for unit tests
- **Create test databases** for integration tests
- **Reset the database state** between tests
- **Use transactions** to roll back changes after tests
- **Test with realistic data volumes** to catch performance issues early

### **20.7.5 Repository Pattern**

The repository pattern abstracts database access behind interfaces:

```go
package main

import (
    "context"
    "database/sql"
    "time"
)

// User represents a user entity
type User struct {
    ID        int
    Username  string
    Email     string
    CreatedAt time.Time
}

// UserRepository defines operations for working with users
type UserRepository interface {
    Create(ctx context.Context, user User) (User, error)
    GetByID(ctx context.Context, id int) (User, error)
    GetByUsername(ctx context.Context, username string) (User, error)
    Update(ctx context.Context, user User) error
    Delete(ctx context.Context, id int) error
}

// PostgresUserRepository implements UserRepository for PostgreSQL
type PostgresUserRepository struct {
    db *sql.DB
}

// NewPostgresUserRepository creates a new PostgreSQL user repository
func NewPostgresUserRepository(db *sql.DB) *PostgresUserRepository {
    return &PostgresUserRepository{db: db}
}

// Create adds a new user
func (r *PostgresUserRepository) Create(ctx context.Context, user User) (User, error) {
    query := `
        INSERT INTO users (username, email, created_at)
        VALUES ($1, $2, $3)
        RETURNING id, created_at
    `

    err := r.db.QueryRowContext(ctx, query, user.Username, user.Email, time.Now()).
        Scan(&user.ID, &user.CreatedAt)

    return user, err
}

// GetByID retrieves a user by ID
func (r *PostgresUserRepository) GetByID(ctx context.Context, id int) (User, error) {
    query := `
        SELECT id, username, email, created_at
        FROM users
        WHERE id = $1
    `

    var user User
    err := r.db.QueryRowContext(ctx, query, id).
        Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt)

    return user, err
}

// GetByUsername retrieves a user by username
func (r *PostgresUserRepository) GetByUsername(ctx context.Context, username string) (User, error) {
    query := `
        SELECT id, username, email, created_at
        FROM users
        WHERE username = $1
    `

    var user User
    err := r.db.QueryRowContext(ctx, query, username).
        Scan(&user.ID, &user.Username, &user.Email, &user.CreatedAt)

    return user, err
}

// Update modifies an existing user
func (r *PostgresUserRepository) Update(ctx context.Context, user User) error {
    query := `
        UPDATE users
        SET username = $1, email = $2
        WHERE id = $3
    `

    _, err := r.db.ExecContext(ctx, query, user.Username, user.Email, user.ID)
    return err
}

// Delete removes a user
func (r *PostgresUserRepository) Delete(ctx context.Context, id int) error {
    query := `
        DELETE FROM users
        WHERE id = $1
    `

    _, err := r.db.ExecContext(ctx, query, id)
    return err
}

// Using the repository in your application:
func main() {
    db, _ := sql.Open("postgres", "postgres://username:password@localhost:5432/myapp?sslmode=disable")

    // Create repository
    userRepo := NewPostgresUserRepository(db)

    // Use repository methods
    ctx := context.Background()

    newUser := User{
        Username: "johndoe",
        Email:    "john@example.com",
    }

    createdUser, err := userRepo.Create(ctx, newUser)
    if err != nil {
        // Handle error
    }

    // Use createdUser...
}
```

Benefits of the repository pattern:

- **Separation of concerns**: Business logic is separated from data access
- **Testability**: Easy to mock for unit tests
- **Swappable implementations**: Change the database without changing business logic
- **Consistency**: Standardized data access patterns

## **20.8 Exercises**

### **Exercise 1: Basic SQL Operations**

Create a simple CLI application that performs CRUD operations on a database of books. Include the following functionality:

- Add a new book with title, author, and publication year
- List all books
- Find books by author
- Update book details
- Delete a book

### **Exercise 2: Working with Relationships**

Extend Exercise 1 to include categories and authors as separate entities:

- Authors have names and biographies
- Categories have names and descriptions
- Books belong to one or more categories
- Books have a single author

Implement queries that:

- Show all books by a specific author
- List books in a particular category
- Show authors who have written books in a specific category

### **Exercise 3: Implement a Repository Layer**

Create a repository pattern implementation for a user management system:

- Define a `UserRepository` interface
- Create a PostgreSQL implementation
- Create an in-memory implementation for testing
- Write a simple service that uses the repository
- Write tests for the service using the in-memory repository

### **Exercise 4: Database Migrations**

Implement a migration system for a blog application with:

- Users table
- Posts table
- Comments table
- Tags table
- Post-tag many-to-many relationship

Create migrations for:

1. The initial schema
2. Adding user profile information
3. Adding post view counts
4. Adding soft delete to posts

### **Exercise 5: Build a Redis Cache Layer**

Create a caching layer using Redis to improve performance of database queries:

- Implement a function to get a user by ID
- If the user is in the cache, return it
- If not, fetch from the database and store in the cache
- Add proper cache invalidation when a user is updated
- Add a TTL (time to live) for cache entries
- Implement cache statistics (hits/misses)

## **20.9 Summary**

In this chapter, we've explored various approaches to working with databases in Go:

- **Standard Library**: Using `database/sql` for SQL databases
- **PostgreSQL**: Leveraging PostgreSQL-specific features
- **ORMs and Query Builders**: Working with GORM and SQLx
- **NoSQL**: Using MongoDB and Redis
- **Migrations**: Managing database schema changes
- **Best Practices**: Connection management, security, and the repository pattern

Go's simplicity and performance make it an excellent choice for database applications. The standard library provides a solid foundation, while third-party packages offer higher-level abstractions when needed.

When working with databases in Go, remember these key points:

1. Use the right tool for the job: SQL for structured data, NoSQL for flexibility
2. Manage database connections properly to avoid leaks and performance issues
3. Use prepared statements and parameterized queries for security and performance
4. Consider the repository pattern to abstract database operations
5. Implement proper migration strategies to manage schema changes
6. Test database interactions thoroughly

By following these principles, you can build robust, efficient, and maintainable database-driven applications in Go.

**Next Up**: In Chapter 21, we'll explore building microservices in Go, leveraging our knowledge of databases along with web services to create distributed systems.
