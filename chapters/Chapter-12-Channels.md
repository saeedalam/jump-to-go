# **Chapter 12: Channels**

---

## **What Are Channels?**

Channels in Go are used to send and receive data between goroutines safely. Think of a channel as a pipe where one goroutine sends data, and another receives it.

---

## **12.1. Creating Channels**

Channels are created using the `make` function. Here’s the basic syntax:

```go
ch := make(chan int)
```

This creates an unbuffered channel for integers.

### **Example 1: Sending and Receiving Data**

```go
package main

import "fmt"

func main() {
    ch := make(chan string)

    // Sending data in a separate goroutine
    go func() {
        ch <- "Hello, Go!"
    }()

    // Receiving data in the main goroutine
    message := <-ch
    fmt.Println(message)
}
```

**Output:**

```
Hello, Go!
```

---

## **12.2. Buffered Channels**

Buffered channels allow you to specify the number of elements a channel can hold.

### **Example 2: Using a Buffered Channel**

```go
package main

import "fmt"

func main() {
    ch := make(chan int, 2) // Buffered channel with a capacity of 2

    ch <- 42
    ch <- 27

    fmt.Println(<-ch)
    fmt.Println(<-ch)
}
```

**Output:**

```
42
27
```

---

## **12.3. Using `select`**

The `select` statement allows a goroutine to wait on multiple channel operations. It’s like a `switch` for channels.

### **Example 3: Basic `select` Statement**

```go
package main

import "fmt"

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        ch1 <- "Channel 1"
    }()

    go func() {
        ch2 <- "Channel 2"
    }()

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
Channel 1
```

(Note: The output may vary depending on which channel sends first.)

---

## **12.4. Default Case in `select`**

The `select` statement can have a default case to execute when no channels are ready.

### **Example 4: Using Default in `select`**

```go
package main

import "fmt"

func main() {
    ch := make(chan int)

    select {
    case val := <-ch:
        fmt.Println("Received:", val)
    default:
        fmt.Println("No data received")
    }
}
```

**Output:**

```
No data received
```

---

## **12.5. Closing Channels**

Closing a channel signals that no more data will be sent on it. Receivers can check if a channel is closed using the second value returned by a receive operation.

### **Example 5: Closing a Channel**

```go
package main

import "fmt"

func main() {
    ch := make(chan int)
    go func() {
        for i := 0; i < 3; i++ {
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
```

---

## **12.6. Timeout with `select`**

You can use the `time.After` function to implement timeouts with channels.

### **Example 6: Channel Timeout**

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
        ch <- "Message received"
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

## **12.7. Fan-Out and Fan-In**

Fan-out distributes work across multiple goroutines, and fan-in aggregates results.

### **Example 7: Fan-In Pattern**

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

    go producer(ch1, "Producer 1")
    go producer(ch2, "Producer 2")

    for i := 0; i < 6; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println(msg1)
        case msg2 := <-ch2:
            fmt.Println(msg2)
        }
    }
}
```

**Output:**

```
Producer 1 0
Producer 2 0
Producer 1 1
Producer 2 1
Producer 1 2
Producer 2 2
```

---

## **12.8. Real-World Example: Worker Pool**

### **Example 8: Implementing a Worker Pool**

```go
package main

import (
    "fmt"
    "sync"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for job := range jobs {
        fmt.Printf("Worker %d processing job %d\n", id, job)
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

## **Key Takeaways**

1. Channels enable safe communication between goroutines.
2. The `select` statement simplifies waiting on multiple channels.
3. Buffered channels can improve performance but require careful management.
4. Patterns like fan-out, fan-in, and worker pools are powerful tools for concurrency.

---

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
    jobs := make(chan int, 5)
    results := make(chan int, 5)

    for i := 1; i <= 3; i++ {
        go worker(i, jobs, results)
    }

    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    for i := 1; i <= 5; i++ {
        fmt.Println("Result:", <-results)
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

**Congratulations!** You've completed the exercises on Go channels. Keep experimenting to unlock the full potential of concurrency in Go! 🚀
