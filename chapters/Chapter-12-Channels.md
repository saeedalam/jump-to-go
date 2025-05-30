# **Chapter 12: Channels in Go**

While Chapter 11 introduced goroutines and touched on channels as a way for goroutines to communicate, this chapter dives deeper into channels—one of Go's most powerful and distinctive features. Channels provide a way for goroutines to communicate and synchronize their execution without explicit locks or condition variables.

At their core, channels embody Go's concurrency philosophy: "Do not communicate by sharing memory; instead, share memory by communicating." This approach helps avoid many of the common pitfalls of concurrent programming while making code more readable and maintainable.

In this chapter, we'll explore the intricacies of Go channels, from basic operations to advanced patterns and best practices. You'll learn how to effectively leverage channels to build robust concurrent applications.

## **12.1 Channel Fundamentals**

### **12.1.1 What Are Channels?**

A channel in Go is a typed conduit through which you can send and receive values. It provides a way for goroutines to synchronize execution and communicate by passing data between them.

```go
package main

import "fmt"

func main() {
    // Create a channel for integers
    ch := make(chan int)

    // Start a goroutine to send a value
    go func() {
        ch <- 42 // Send 42 to the channel
    }()

    // Receive the value from the channel
    value := <-ch
    fmt.Println("Received:", value) // Output: Received: 42
}
```

Channels are first-class values that can be allocated, passed as arguments, stored in variables, and returned from functions. They are inherently synchronized, which means operations on them happen in a defined order.

### **12.1.2 Channel Types and Directionality**

Channels can be declared with specific directionality constraints:

```go
package main

import "fmt"

// Function that can only send to the channel
func sender(ch chan<- int) {
    for i := 1; i <= 5; i++ {
        ch <- i
        fmt.Println("Sent:", i)
    }
    close(ch)
}

// Function that can only receive from the channel
func receiver(ch <-chan int) {
    for val := range ch {
        fmt.Println("Received:", val)
    }
}

func main() {
    // Bidirectional channel
    ch := make(chan int)

    go sender(ch)  // Passed as send-only channel
    receiver(ch)   // Passed as receive-only channel
}
```

Channel directionality helps catch programming errors at compile time and documents how a channel is intended to be used:

- `chan T`: Bidirectional channel (can send and receive values of type T)
- `chan<- T`: Send-only channel (can only send values of type T)
- `<-chan T`: Receive-only channel (can only receive values of type T)

### **12.1.3 Creating and Closing Channels**

Channels are created using the `make` function, and they can be closed when no more values will be sent:

```go
package main

import "fmt"

func main() {
    // Create an unbuffered channel
    unbuffered := make(chan int)

    // Create a buffered channel with capacity 5
    buffered := make(chan int, 5)

    // Using a channel
    go func() {
        for i := 1; i <= 3; i++ {
            buffered <- i
        }
        close(buffered) // Close the channel when done sending
    }()

    // Receive values until channel is closed
    for val := range buffered {
        fmt.Println("Value:", val)
    }

    // After the loop, the channel is known to be closed
    fmt.Println("Channel closed, loop exited")
}
```

Key points about closing channels:

- Only the sender should close a channel, never the receiver
- Sending on a closed channel causes a panic
- Receiving from a closed channel returns the zero value immediately
- The second return value from a receive operation indicates whether the channel is still open

```go
package main

import "fmt"

func main() {
    ch := make(chan int)

    go func() {
        ch <- 42
        close(ch)
    }()

    // First receive - gets the value
    val, ok := <-ch
    fmt.Printf("val: %d, open: %t\n", val, ok) // val: 42, open: true

    // Second receive - channel is closed
    val, ok = <-ch
    fmt.Printf("val: %d, open: %t\n", val, ok) // val: 0, open: false
}
```

### **12.1.4 Unbuffered vs. Buffered Channels**

Go supports both unbuffered and buffered channels, each with different synchronization behaviors:

**Unbuffered Channels:**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int) // Unbuffered channel

    go func() {
        fmt.Println("Sending value...")
        ch <- 42
        fmt.Println("Value sent!")
    }()

    // Add a small delay to demonstrate the blocking nature
    time.Sleep(time.Second)
    fmt.Println("About to receive...")
    val := <-ch
    fmt.Println("Received:", val)
}
```

With unbuffered channels, send operations block until there is a corresponding receive operation, and vice versa. This creates a perfect synchronization point between goroutines.

**Buffered Channels:**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int, 2) // Buffered channel with capacity 2

    go func() {
        for i := 1; i <= 3; i++ {
            fmt.Println("Sending:", i)
            ch <- i
            fmt.Println("Sent:", i)
        }
    }()

    // Wait to let the goroutine execute
    time.Sleep(time.Second)

    // Receive values
    for i := 1; i <= 3; i++ {
        val := <-ch
        fmt.Println("Received:", val)
    }
}
```

With buffered channels:

- Sends block only when the buffer is full
- Receives block only when the buffer is empty
- The buffer decouples the send and receive operations in time

## **12.2 Channel Operations and Patterns**

### **12.2.1 The Select Statement**

The `select` statement lets a goroutine wait on multiple channel operations simultaneously:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    // Send on ch1 after 1 second
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "one"
    }()

    // Send on ch2 after 2 seconds
    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "two"
    }()

    // Use select to wait on both channels
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received from ch1:", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received from ch2:", msg2)
        }
    }
}
```

The `select` statement:

- Blocks until one of its cases can proceed
- If multiple cases are ready, it chooses one at random
- Can include a `default` case that executes immediately if no other case is ready

### **12.2.2 Non-Blocking Channel Operations**

The `select` statement with a `default` case allows for non-blocking channel operations:

```go
package main

import "fmt"

func main() {
    ch := make(chan int)

    // Start a goroutine that will eventually send a value
    go func() {
        time.Sleep(2 * time.Second)
        ch <- 42
    }()

    // Try to receive, but don't block
    select {
    case val := <-ch:
        fmt.Println("Received:", val)
    default:
        fmt.Println("No value available yet")
    }

    // Wait a bit and try again
    time.Sleep(3 * time.Second)

    // Now the value should be available
    select {
    case val := <-ch:
        fmt.Println("Received:", val)
    default:
        fmt.Println("No value available yet")
    }
}
```

Non-blocking operations are useful in scenarios where you want to check for channel activity without pausing the execution of your goroutine.

### **12.2.3 Timeouts with Channels**

The `select` statement can implement timeouts using `time.After`:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)

    // Attempt an operation that might take too long
    go func() {
        time.Sleep(2 * time.Second)
        ch <- "operation completed"
    }()

    // Wait for the operation with a timeout
    select {
    case result := <-ch:
        fmt.Println("Success:", result)
    case <-time.After(1 * time.Second):
        fmt.Println("Timeout: operation took too long")
    }
}
```

This pattern is useful for preventing a goroutine from blocking indefinitely when interacting with potentially slow or unresponsive operations.

### **12.2.4 Cancellation with Channels**

Channels can be used to signal cancellation to goroutines:

```go
package main

import (
    "fmt"
    "time"
)

func worker(done <-chan bool) {
    go func() {
        for {
            select {
            case <-done:
                fmt.Println("Worker: Received cancellation signal")
                return
            default:
                fmt.Println("Worker: Working...")
                time.Sleep(500 * time.Millisecond)
            }
        }
    }()
}

func main() {
    done := make(chan bool)

    worker(done)

    // Let the worker run for 2 seconds
    time.Sleep(2 * time.Second)

    // Signal the worker to stop
    fmt.Println("Main: Sending cancellation signal")
    done <- true

    // Give the worker time to exit
    time.Sleep(1 * time.Second)
    fmt.Println("Main: Exiting")
}
```

This pattern allows for graceful termination of goroutines when their work is no longer needed.

### **12.2.5 Fan-Out, Fan-In Pattern**

The fan-out, fan-in pattern is a common way to parallelize work using channels:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Generate produces integers in a sequence
func generate(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

// Square squares numbers from input channel and returns output channel
func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
            time.Sleep(200 * time.Millisecond) // Simulate work
        }
    }()
    return out
}

// Merge consolidates multiple input channels into a single output channel
func merge(cs ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    out := make(chan int)

    // Start output goroutine for each input channel
    output := func(c <-chan int) {
        defer wg.Done()
        for n := range c {
            out <- n
        }
    }

    wg.Add(len(cs))
    for _, c := range cs {
        go output(c)
    }

    // Start goroutine to close once all inputs are processed
    go func() {
        wg.Wait()
        close(out)
    }()
    return out
}

func main() {
    // Fan-out: distribute work across multiple goroutines
    in := generate(1, 2, 3, 4, 5)
    c1 := square(in)
    c2 := square(in)
    c3 := square(in)

    // Fan-in: consolidate results
    for n := range merge(c1, c2, c3) {
        fmt.Println(n)
    }
}
```

This pattern:

- Distributes work across multiple goroutines (fan-out)
- Collects results from multiple goroutines into a single channel (fan-in)
- Is particularly useful for CPU-bound tasks that can be parallelized

# **12.9. Exercises**

---

## **Exercise 1: Basic Channel Communication**

**Problem**: Create a channel and send a message from one goroutine to another.

```go
package main

import "fmt"

func main() {
    ch := make(chan string)

    go func() {
        ch <- "Hello from goroutine!"
    }()

    msg := <-ch
    fmt.Println(msg)
}
```

**Output:**

```
Hello from goroutine!
```

---

## **Exercise 2: Buffered Channel**

**Problem**: Use a buffered channel to send and receive multiple values.

```go
package main

import "fmt"

func main() {
    ch := make(chan int, 3)
    ch <- 10
    ch <- 20
    ch <- 30

    fmt.Println(<-ch)
    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

**Output:**

```
10
20
30
```

---

## **Exercise 3: Closing a Channel**

**Problem**: Close a channel and iterate over its values using `range`.

```go
package main

import "fmt"

func main() {
    ch := make(chan int)

    go func() {
        for i := 0; i < 5; i++ {
            ch <- i
        }
        close(ch)
    }()

    for val := range ch {
        fmt.Println(val)
    }
}
```

**Output:**

```
0
1
2
3
4
```

---

## **Exercise 4: Using `select` Statement**

**Problem**: Use the `select` statement to read from multiple channels.

```go
package main

import "fmt"

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() { ch1 <- "Message from ch1" }()
    go func() { ch2 <- "Message from ch2" }()

    select {
    case msg1 := <-ch1:
        fmt.Println(msg1)
    case msg2 := <-ch2:
        fmt.Println(msg2)
    }
}
```

**Output:**

```
Message from ch1
```

(Note: The output depends on which channel is ready first.)

---

## **Exercise 5: Implementing a Timeout**

**Problem**: Use `time.After` to add a timeout when waiting for channel data.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)

    go func() {
        time.Sleep(2 * time.Second)
        ch <- "Data received"
    }()

    select {
    case msg := <-ch:
        fmt.Println(msg)
    case <-time.After(1 * time.Second):
        fmt.Println("Timeout")
    }
}
```

**Output:**

```
Timeout
```

---

## **Exercise 6: Fan-Out Pattern**

**Problem**: Use multiple goroutines to send data into a channel.

```go
package main

import "fmt"

func worker(id int, ch chan string) {
    ch <- fmt.Sprintf("Worker %d: task completed", id)
}

func main() {
    ch := make(chan string, 3)

    for i := 1; i <= 3; i++ {
        go worker(i, ch)
    }

    for i := 1; i <= 3; i++ {
        fmt.Println(<-ch)
    }
}
```

**Output:**

```
Worker 1: task completed
Worker 2: task completed
Worker 3: task completed
```

---

## **Exercise 7: Fan-In Pattern**

**Problem**: Combine data from multiple channels into a single channel.

```go
package main

import "fmt"

func producer(ch chan string, msg string) {
    for i := 0; i < 3; i++ {
        ch <- fmt.Sprintf("%s %d", msg, i)
    }
    close(ch)
}

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    result := make(chan string)

    go producer(ch1, "Producer 1")
    go producer(ch2, "Producer 2")

    go func() {
        for msg := range ch1 {
            result <- msg
        }
        for msg := range ch2 {
            result <- msg
        }
        close(result)
    }()

    for msg := range result {
        fmt.Println(msg)
    }
}
```

**Output:**

```
Producer 1 0
Producer 1 1
Producer 1 2
Producer 2 0
Producer 2 1
Producer 2 2
```

---

## **Exercise 8: Worker Pool**

**Problem**: Implement a worker pool using channels.

```go
package main

import (
    "fmt"
    "sync"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d
", id, job)
        results <- job * 2
    }
}

func main() {
    const numJobs = 5
    const numWorkers = 3

    jobs := make(chan int, numJobs)
    results := make(chan int, numJobs)

    var wg sync.WaitGroup
    for w := 1; w <= numWorkers; w++ {
        wg.Add(1)
        go func(id int) {
            worker(id, jobs, results)
            wg.Done()
        }(w)
    }

    for j := 1; j <= numJobs; j++ {
        jobs <- j
    }
    close(jobs)

    wg.Wait()
    close(results)

    for result := range results {
        fmt.Println("Result:", result)
    }
}
```

**Output:**

```
Worker 1 processing job 1
Worker 2 processing job 2
Worker 3 processing job 3
Worker 1 processing job 4
Worker 2 processing job 5
Result: 2
Result: 4
Result: 6
Result: 8
Result: 10
```

---

## **Exercise 9: Broadcast Pattern**

**Problem**: Use one channel to broadcast messages to multiple goroutines.

```go
package main

import (
    "fmt"
    "sync"
)

func listener(id int, ch <-chan string) {
    for msg := range ch {
        fmt.Printf("Listener %d received: %s
", id, msg)
    }
}

func main() {
    ch := make(chan string)
    var wg sync.WaitGroup

    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            listener(id, ch)
            wg.Done()
        }(i)
    }

    for i := 1; i <= 5; i++ {
        ch <- fmt.Sprintf("Broadcast message %d", i)
    }
    close(ch)

    wg.Wait()
}
```

**Output:**

```
Listener 1 received: Broadcast message 1
Listener 2 received: Broadcast message 2
Listener 3 received: Broadcast message 3
...
```

---

## **Exercise 10: Alternating Between Channels**

**Problem**: Use `select` to alternate between two channels.

```go
package main

import "fmt"

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        for i := 0; i < 5; i++ {
            ch1 <- i
        }
        close(ch1)
    }()

    go func() {
        for i := 5; i < 10; i++ {
            ch2 <- i
        }
        close(ch2)
    }()

    for {
        select {
        case val, ok := <-ch1:
            if ok {
                fmt.Println("From ch1:", val)
            }
        case val, ok := <-ch2:
            if ok {
                fmt.Println("From ch2:", val)
            }
        }
        if len(ch1) == 0 && len(ch2) == 0 {
            break
        }
    }
}
```

**Output:**

```
From ch1: 0
From ch1: 1
...
From ch2: 9
```

---

## **12.3 Advanced Channel Patterns**

### **12.3.1 The Pipeline Pattern**

Pipelines are a powerful pattern for processing data through a series of stages. Each stage receives values from an upstream stage, processes them, and sends the results to a downstream stage.

```go
package main

import (
    "fmt"
)

// Stage 1: Generate integers
func gen(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

// Stage 2: Square the numbers
func sq(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

// Stage 3: Filter out odd numbers
func filter(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if n%2 == 0 {
                out <- n
            }
        }
    }()
    return out
}

func main() {
    // Set up the pipeline
    c1 := gen(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    c2 := sq(c1)
    c3 := filter(c2)

    // Consume the output
    for n := range c3 {
        fmt.Println(n)
    }
}
```

Key properties of the pipeline pattern:

- Each stage closes its output channel when it's done sending
- Each stage continues to receive values from upstream until the channel is closed
- Each stage is an independent goroutine, enabling concurrent processing

### **12.3.2 Worker Pools with Done Channels**

Worker pools can be enhanced with cancellation signals to clean up resources properly:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int, done <-chan struct{}) {
    for {
        select {
        case job, ok := <-jobs:
            if !ok {
                return // Channel closed
            }
            fmt.Printf("Worker %d started job %d\n", id, job)
            time.Sleep(500 * time.Millisecond) // Simulate work
            fmt.Printf("Worker %d finished job %d\n", id, job)
            results <- job * 2
        case <-done:
            fmt.Printf("Worker %d shutting down\n", id)
            return
        }
    }
}

func main() {
    jobs := make(chan int, 10)
    results := make(chan int, 10)
    done := make(chan struct{})

    // Start 3 workers
    var wg sync.WaitGroup
    for i := 1; i <= 3; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            worker(id, jobs, results, done)
        }(i)
    }

    // Send 5 jobs
    for j := 1; j <= 5; j++ {
        jobs <- j
    }

    // Collect the results in a separate goroutine
    go func() {
        for result := range results {
            fmt.Println("Result:", result)
        }
    }()

    // Simulate some work, then trigger cancellation
    time.Sleep(2 * time.Second)
    fmt.Println("Sending cancellation signal")
    close(done)

    // Wait for all workers to exit
    wg.Wait()
    fmt.Println("All workers have terminated")
}
```

This pattern ensures proper cleanup of goroutines and resources when operations need to be cancelled.

### **12.3.3 Context for Cancellation**

Go's `context` package provides a more structured way to handle cancellation:

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func worker(ctx context.Context, id int) {
    for {
        select {
        case <-ctx.Done():
            fmt.Printf("Worker %d: Stopping due to cancellation\n", id)
            return
        default:
            fmt.Printf("Worker %d: Working...\n", id)
            time.Sleep(1 * time.Second)
        }
    }
}

func main() {
    // Create a context with cancellation capability
    ctx, cancel := context.WithCancel(context.Background())

    // Start workers
    for i := 1; i <= 3; i++ {
        go worker(ctx, i)
    }

    // Let workers run for 3 seconds
    time.Sleep(3 * time.Second)

    // Trigger cancellation
    fmt.Println("Main: Cancelling workers")
    cancel()

    // Give workers time to respond to cancellation
    time.Sleep(1 * time.Second)
    fmt.Println("Main: Done")
}
```

The `context` package provides several ways to create contexts with different cancellation behaviors:

- `WithCancel`: Manual cancellation
- `WithDeadline`: Cancellation at a specific time
- `WithTimeout`: Cancellation after a duration
- `WithValue`: Carries request-scoped values across API boundaries

### **12.3.4 Rate Limiting with Buffered Channels**

Channels can be used to implement rate limiting:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Create a channel to serve as our rate limiter
    // By using a buffered channel of size 3, we allow
    // up to 3 operations to happen concurrently
    limiter := make(chan struct{}, 3)

    // Simulate 10 requests
    for i := 1; i <= 10; i++ {
        // Acquire a token from the rate limiter
        limiter <- struct{}{}

        go func(id int) {
            defer func() {
                // Release the token when done
                <-limiter
            }()

            fmt.Printf("Request %d starting at %s\n",
                       id, time.Now().Format("15:04:05.000"))

            // Simulate work
            time.Sleep(2 * time.Second)

            fmt.Printf("Request %d completed at %s\n",
                       id, time.Now().Format("15:04:05.000"))
        }(i)
    }

    // Wait for all goroutines to finish
    time.Sleep(10 * time.Second)
}
```

This pattern limits the number of concurrent operations, which is useful for preventing resource exhaustion.

## **12.4 Common Channel Patterns**

### **12.4.1 Generator Pattern**

The generator pattern produces a sequence of values:

```go
package main

import (
    "fmt"
    "time"
)

// Simple generator - returns a receive-only channel
func fibonacci() <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        a, b := 0, 1
        for i := 0; i < 10; i++ {
            out <- a
            a, b = b, a+b
            time.Sleep(300 * time.Millisecond) // Simulate work
        }
    }()
    return out
}

func main() {
    fmt.Println("Fibonacci sequence:")
    for num := range fibonacci() {
        fmt.Println(num)
    }
}
```

This pattern is useful for producing sequences, streams of data, or events.

### **12.4.2 Future/Promise Pattern**

Channels can implement a future/promise pattern:

```go
package main

import (
    "fmt"
    "time"
)

// Future represents a value that will be available in the future
type Future struct {
    result <-chan int
}

// NewFuture creates a new Future for a computation
func NewFuture(fn func() int) Future {
    result := make(chan int, 1) // Buffered to avoid goroutine leak

    go func() {
        result <- fn() // Compute and send the result
        close(result)
    }()

    return Future{result: result}
}

// Get returns the result, blocking if necessary
func (f Future) Get() int {
    return <-f.result
}

// Expensive computation
func compute(val int) int {
    fmt.Printf("Computing with value %d...\n", val)
    time.Sleep(2 * time.Second) // Simulate expensive computation
    return val * val
}

func main() {
    // Start computation in the background
    future := NewFuture(func() int {
        return compute(42)
    })

    fmt.Println("Future created, doing other work...")
    time.Sleep(1 * time.Second) // Do other work

    // Get the result when needed
    fmt.Println("Waiting for result...")
    result := future.Get()
    fmt.Println("Result:", result)
}
```

This pattern allows you to start a computation in the background and retrieve the result when needed.

### **12.4.3 Pub-Sub Pattern**

A simple publish-subscribe pattern using channels:

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// Message represents a message in the pub-sub system
type Message struct {
    Topic   string
    Payload string
}

// PubSub implements a simple publish-subscribe pattern
type PubSub struct {
    mu          sync.RWMutex
    subscribers map[string][]chan Message
}

// NewPubSub creates a new PubSub instance
func NewPubSub() *PubSub {
    return &PubSub{
        subscribers: make(map[string][]chan Message),
    }
}

// Subscribe creates a subscription to a topic
func (ps *PubSub) Subscribe(topic string) <-chan Message {
    ps.mu.Lock()
    defer ps.mu.Unlock()

    ch := make(chan Message, 10)
    ps.subscribers[topic] = append(ps.subscribers[topic], ch)
    return ch
}

// Publish sends a message to all subscribers of a topic
func (ps *PubSub) Publish(topic, payload string) {
    ps.mu.RLock()
    defer ps.mu.RUnlock()

    msg := Message{Topic: topic, Payload: payload}
    for _, ch := range ps.subscribers[topic] {
        // Non-blocking send to avoid slow subscribers
        // causing publishers to block
        select {
        case ch <- msg:
        default:
            // Channel buffer full, message dropped
        }
    }
}

func main() {
    ps := NewPubSub()

    // Subscribe to topics
    ch1 := ps.Subscribe("topic1")
    ch2 := ps.Subscribe("topic1")
    ch3 := ps.Subscribe("topic2")

    // Start subscribers
    var wg sync.WaitGroup
    wg.Add(3)

    go func() {
        defer wg.Done()
        for msg := range ch1 {
            fmt.Printf("Subscriber 1 received [%s]: %s\n",
                       msg.Topic, msg.Payload)
        }
    }()

    go func() {
        defer wg.Done()
        for msg := range ch2 {
            fmt.Printf("Subscriber 2 received [%s]: %s\n",
                       msg.Topic, msg.Payload)
        }
    }()

    go func() {
        defer wg.Done()
        for msg := range ch3 {
            fmt.Printf("Subscriber 3 received [%s]: %s\n",
                       msg.Topic, msg.Payload)
        }
    }()

    // Publish messages
    ps.Publish("topic1", "Hello from topic1")
    ps.Publish("topic2", "Hello from topic2")
    ps.Publish("topic1", "Another message for topic1")

    // Wait a bit for messages to be processed
    time.Sleep(1 * time.Second)
}
```

This pattern allows for decoupled communication between components, where publishers don't need to know about subscribers and vice versa.

## **12.5 Channel Best Practices**

### **12.5.1 Ownership and Directionality**

Clear ownership and directionality make channel-based code easier to understand and maintain:

1. **Establish Clear Ownership**

   - Clearly define which goroutine owns each channel
   - The owner is responsible for creating, closing, and writing to the channel
   - Non-owners should only read from the channel

2. **Use Channel Direction Constraints**
   - Functions that only receive from a channel should use `<-chan T`
   - Functions that only send to a channel should use `chan<- T`
   - This documents intent and catches errors at compile time

```go
package main

import "fmt"

// Producer owns the channel and is responsible for closing it
func producer(count int) <-chan int {
    ch := make(chan int)
    go func() {
        defer close(ch) // Owner is responsible for closing
        for i := 0; i < count; i++ {
            ch <- i
        }
    }()
    return ch
}

// Consumer only reads from the channel
func consumer(ch <-chan int) {
    for val := range ch {
        fmt.Println("Received:", val)
    }
}

func main() {
    // Producer owns the channel
    ch := producer(5)

    // Consumer reads from the channel
    consumer(ch)
}
```

### **12.5.2 Closing Channels**

Follow these guidelines when closing channels:

1. **Only the Sender Should Close**

   - Closing a channel indicates "no more values will be sent"
   - Only the sender (owner) should close a channel
   - Never close a channel from the receiver side

2. **Don't Close a Channel Multiple Times**

   - Closing an already closed channel causes a panic
   - Use sync primitives if multiple goroutines might close the same channel

3. **Use Defer for Reliable Closing**
   - In complex functions, use `defer close(ch)` to ensure the channel is closed

```go
package main

import "fmt"

func main() {
    // GOOD: Single owner closing the channel
    ch := make(chan int)
    go func() {
        defer close(ch) // Ensures channel is closed even if panic occurs
        for i := 0; i < 5; i++ {
            ch <- i
        }
    }()

    // Safely consume from the channel
    for val := range ch {
        fmt.Println(val)
    }

    // BAD: Multiple closers (would cause panic)
    /*
    ch2 := make(chan int)
    go func() { close(ch2) }()
    go func() { close(ch2) }() // PANIC: close of closed channel
    */
}
```

### **12.5.3 Buffer Size Considerations**

Choose buffer sizes carefully:

1. **Unbuffered Channels (Size 0)**

   - Provide synchronization guarantees
   - Sender blocks until receiver is ready
   - Good for coordinating goroutines

2. **Small Buffered Channels (Size 1-10)**

   - Help smooth out bursts of activity
   - Reduce goroutine blocking
   - Good for producer-consumer patterns with varying speeds

3. **Large Buffered Channels**
   - Should be used with caution
   - May mask underlying performance issues
   - Consider using for known batch sizes or to absorb known spikes

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Unbuffered: synchronization
    synch := make(chan int)
    go func() {
        fmt.Println("Sender: About to send")
        synch <- 42
        fmt.Println("Sender: Value sent")
    }()
    time.Sleep(1 * time.Second)
    fmt.Println("Receiver: About to receive")
    <-synch
    fmt.Println("Receiver: Received value")

    // Small buffer: smooth bursts
    burst := make(chan int, 3)
    go func() {
        for i := 0; i < 5; i++ {
            fmt.Printf("Sending value %d\n", i)
            burst <- i
        }
        close(burst)
    }()

    // Simulate slow consumer
    for val := range burst {
        fmt.Printf("Received value %d\n", val)
        time.Sleep(300 * time.Millisecond)
    }
}
```

### **12.5.4 Error Handling with Channels**

Handle errors gracefully in channel-based code:

1. **Include Error Information in Results**

   - Create struct types that include both results and errors
   - Send these structs through channels

2. **Use Separate Error Channels**
   - For critical errors that require immediate attention
   - Or when error handling is complex

```go
package main

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Result combines a value and possible error
type Result struct {
    Value int
    Err   error
}

func worker(id int) <-chan Result {
    results := make(chan Result)

    go func() {
        defer close(results)

        // Simulate work with possible errors
        time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)

        if rand.Float32() < 0.3 { // 30% chance of error
            results <- Result{
                Err: errors.New(fmt.Sprintf("worker %d failed", id)),
            }
            return
        }

        results <- Result{
            Value: id * 10,
            Err:   nil,
        }
    }()

    return results
}

func main() {
    rand.Seed(time.Now().UnixNano())

    // Launch several workers
    workers := 5
    results := make([]<-chan Result, workers)
    for i := 0; i < workers; i++ {
        results[i] = worker(i)
    }

    // Process results, handling errors
    for i, ch := range results {
        result := <-ch
        if result.Err != nil {
            fmt.Printf("Error from worker %d: %v\n", i, result.Err)
            continue
        }
        fmt.Printf("Worker %d returned: %d\n", i, result.Value)
    }
}
```

## **12.6 Exercises**

### **Exercise 1: Basic Channel Operations**

Create a program that starts multiple goroutines which communicate through channels.

```go
package main

import (
    "fmt"
    "time"
)

func sender(ch chan<- string) {
    // TODO: Send three different messages to the channel
    // with a short delay between each message
    // Don't forget to close the channel when done
}

func receiver(ch <-chan string) {
    // TODO: Receive and print all messages from the channel
    // until the channel is closed
}

func main() {
    // TODO:
    // 1. Create a channel
    // 2. Start the sender and receiver goroutines
    // 3. Wait for them to finish
}

// Solution:
/*
package main

import (
    "fmt"
    "time"
)

func sender(ch chan<- string) {
    for i := 1; i <= 3; i++ {
        msg := fmt.Sprintf("Message %d", i)
        ch <- msg
        fmt.Println("Sent:", msg)
        time.Sleep(100 * time.Millisecond)
    }
    close(ch)
    fmt.Println("Sender: closed channel")
}

func receiver(ch <-chan string) {
    for msg := range ch {
        fmt.Println("Received:", msg)
    }
    fmt.Println("Receiver: channel closed")
}

func main() {
    ch := make(chan string)
    go sender(ch)
    receiver(ch)
}
*/
```

### **Exercise 2: Implementing a Pipeline**

Create a pipeline that generates numbers, filters out odd numbers, and then squares the remaining even numbers.

```go
package main

import (
    "fmt"
)

// TODO:
// 1. Implement a generator function that sends numbers 1-10 to a channel
// 2. Implement a filter function that receives numbers and sends only even numbers
// 3. Implement a square function that receives numbers and sends their squares
// 4. Connect these functions into a pipeline in the main function

// Solution:
/*
package main

import (
    "fmt"
)

func generate(n int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 1; i <= n; i++ {
            out <- i
        }
    }()
    return out
}

func filterEven(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if n%2 == 0 {
                out <- n
            }
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

func main() {
    // Set up the pipeline
    nums := generate(10)
    evens := filterEven(nums)
    squares := square(evens)

    // Consume the final output
    fmt.Println("Squares of even numbers:")
    for n := range squares {
        fmt.Println(n)
    }
}
*/
```

### **Exercise 3: Fan-Out, Fan-In**

Implement a fan-out, fan-in pattern to process a list of numbers concurrently.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// TODO:
// 1. Implement a function that generates numbers 1-10
// 2. Implement a function that processes a number (e.g., calculates factorial)
// 3. Implement the fan-out logic to distribute work
// 4. Implement the fan-in logic to collect results
// 5. Connect these in the main function

// Solution:
/*
package main

import (
    "fmt"
    "sync"
    "time"
)

// Generates numbers 1-10
func generator() <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 1; i <= 10; i++ {
            out <- i
        }
    }()
    return out
}

// Calculates factorial of a number
func processor(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            // Simulate CPU-intensive work
            time.Sleep(100 * time.Millisecond)
            result := 1
            for i := 2; i <= n; i++ {
                result *= i
            }
            out <- result
        }
    }()
    return out
}

// Merges multiple channels into one
func fanIn(channels ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    merged := make(chan int)

    // Start an output goroutine for each input channel
    wg.Add(len(channels))
    for _, ch := range channels {
        go func(c <-chan int) {
            defer wg.Done()
            for n := range c {
                merged <- n
            }
        }(ch)
    }

    // Start a goroutine to close merged once all inputs are done
    go func() {
        wg.Wait()
        close(merged)
    }()

    return merged
}

func main() {
    // Source channel
    source := generator()

    // Fan-out to 3 workers
    workers := 3
    channels := make([]<-chan int, workers)
    for i := 0; i < workers; i++ {
        channels[i] = processor(source)
    }

    // Fan-in the results
    results := fanIn(channels...)

    // Collect and print results
    for result := range results {
        fmt.Println("Result:", result)
    }
}
*/
```

### **Exercise 4: Timeout and Select**

Create a function that fetches data from multiple sources with a timeout.

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

// TODO:
// 1. Create functions that simulate fetching data from different sources
// 2. Use select to handle timeouts and receive from the first available source
// 3. If all sources time out, return an error message

// Solution:
/*
package main

import (
    "fmt"
    "math/rand"
    "time"
)

// Simulate fetching data from a source with variable response time
func fetchData(source string) <-chan string {
    ch := make(chan string)
    go func() {
        // Simulate variable response time (0-3 seconds)
        delay := time.Duration(rand.Intn(3000)) * time.Millisecond
        fmt.Printf("Source %s will respond in %v\n", source, delay)
        time.Sleep(delay)
        ch <- fmt.Sprintf("Data from source %s", source)
    }()
    return ch
}

func fetchWithTimeout() (string, error) {
    // Set up sources
    source1 := fetchData("A")
    source2 := fetchData("B")
    source3 := fetchData("C")

    // Create a timeout
    timeout := time.After(2 * time.Second)

    // Wait for the first result or timeout
    select {
    case data := <-source1:
        return data, nil
    case data := <-source2:
        return data, nil
    case data := <-source3:
        return data, nil
    case <-timeout:
        return "", fmt.Errorf("all sources timed out")
    }
}

func main() {
    rand.Seed(time.Now().UnixNano())

    fmt.Println("Fetching data...")
    data, err := fetchWithTimeout()
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("Received:", data)
    }
}
*/
```

### **Exercise 5: Rate Limiter**

Implement a rate limiter using channels to control the rate of requests.

```go
package main

import (
    "fmt"
    "time"
)

// TODO:
// 1. Implement a rate limiter that allows a maximum of N operations per second
// 2. Use it to control a series of worker goroutines
// 3. Demonstrate that the rate limiting is working as expected

// Solution:
/*
package main

import (
    "fmt"
    "sync"
    "time"
)

// RateLimiter allows a maximum of rate operations per interval
type RateLimiter struct {
    rate     int
    interval time.Duration
    tokens   chan struct{}
}

// NewRateLimiter creates a new rate limiter
func NewRateLimiter(rate int, interval time.Duration) *RateLimiter {
    limiter := &RateLimiter{
        rate:     rate,
        interval: interval,
        tokens:   make(chan struct{}, rate),
    }

    // Fill the token bucket
    for i := 0; i < rate; i++ {
        limiter.tokens <- struct{}{}
    }

    // Refill tokens at the specified rate
    go func() {
        ticker := time.NewTicker(interval / time.Duration(rate))
        defer ticker.Stop()

        for range ticker.C {
            select {
            case limiter.tokens <- struct{}{}:
                // Added a token
            default:
                // Bucket is full
            }
        }
    }()

    return limiter
}

// Wait blocks until a token is available
func (rl *RateLimiter) Wait() {
    <-rl.tokens
}

func main() {
    // Create a rate limiter: 3 operations per second
    limiter := NewRateLimiter(3, time.Second)

    var wg sync.WaitGroup

    // Simulate 10 operations
    for i := 1; i <= 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()

            // Get the current time before waiting
            start := time.Now()

            // Wait for rate limiter
            limiter.Wait()

            // Calculate how long we waited
            waited := time.Since(start)

            fmt.Printf("Operation %d started at %s (waited %v)\n",
                      id, time.Now().Format("15:04:05.000"), waited)

            // Simulate work
            time.Sleep(100 * time.Millisecond)
        }(i)
    }

    wg.Wait()
    fmt.Println("All operations completed")
}
*/
```

## **12.7 Summary**

In this chapter, we've explored Go's channels in depth:

- **Channel Fundamentals**: We learned how channels work, including their creation, directionality, and operations like sending, receiving, and closing.

- **Buffered vs. Unbuffered Channels**: We examined the differences between buffered and unbuffered channels and when to use each type.

- **Advanced Channel Operations**: We covered essential operations like using the `select` statement, non-blocking operations, and implementing timeouts.

- **Concurrency Patterns**: We explored patterns like pipelines, worker pools, fan-out/fan-in, and rate limiting, which help solve common concurrency challenges.

- **Advanced Patterns**: We implemented more complex patterns including generators, futures/promises, and pub-sub systems.

- **Best Practices**: We discussed important principles like channel ownership, proper closing, buffer sizing, and error handling.

Channels are a cornerstone of Go's concurrency model, embodying the language's philosophy: "Do not communicate by sharing memory; instead, share memory by communicating." By using channels effectively, you can build concurrent programs that are both efficient and maintainable.

Understanding channels and their patterns allows you to leverage Go's concurrency capabilities to their fullest potential, writing code that takes advantage of modern multi-core processors while maintaining readability and safety.

## **12.8 Next Steps**

Now that you've mastered channels and goroutines (from Chapter 11), you have a solid foundation in Go's concurrency model. The next steps in your Go journey might include:

1. **Exploring the standard library** to see how it uses concurrency patterns
2. **Building real-world concurrent applications** like web servers and data processing pipelines
3. **Learning advanced synchronization techniques** with the `sync` and `sync/atomic` packages
4. **Investigating third-party concurrency libraries** that build on Go's primitives

In the next chapter, we'll explore Go's standard library and how it provides rich functionality for common programming tasks.
