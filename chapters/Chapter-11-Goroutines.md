# **Chapter 10: Goroutines**

---

## **10.1 What Are Goroutines?**

Goroutines are Go's lightweight threads managed by the Go runtime. Unlike traditional threads:

- **Efficiency:** They are highly efficient and can handle thousands of concurrent tasks.
- **Multiplexing:** They are multiplexed onto operating system threads by the Go runtime, saving resources.

Think of a Goroutine as a function running concurrently with other Goroutines in the same application. Goroutines allow you to perform tasks concurrently without worrying about complex thread management.

---

## **10.2 Launching Goroutines**

In Go, launching a Goroutine is as simple as placing the `go` keyword before a function call. When you use `go` before a function, Go starts that function as a Goroutine and immediately moves on to the next instruction in the calling function, without waiting for the Goroutine to finish.

### **Example 1: Starting a Simple Goroutine**

```go
package main

import (
	"fmt"
	"time"
)

func sayHello() {
	fmt.Println("Hello from Goroutine!")
}

func main() {
	go sayHello() // Launch the Goroutine
	fmt.Println("Hello from Main!")
	time.Sleep(1 * time.Second) // Wait for Goroutine to finish
}
```

**Explanation:**

- The `go` keyword starts the `sayHello` function as a Goroutine.
- The `main` function continues executing without waiting for the Goroutine to complete, unless explicitly synchronized.
- The `time.Sleep` is used to give the Goroutine enough time to execute before the program ends.

**Output:**

```
Hello from Main!
Hello from Goroutine!
```

---

## **10.3 Understanding Concurrency**

Goroutines execute independently and concurrently. This means that the execution order of Goroutines is unpredictable, and it depends on how the Go runtime schedules them. This is an essential concept when dealing with concurrency: Goroutines can run in any order, and we cannot assume that one will finish before the other.

### **Example 2: Multiple Goroutines**

```go
package main

import (
	"fmt"
	"time"
)

func printNumbers(name string) {
	for i := 1; i <= 5; i++ {
		fmt.Printf("%s: %d
", name, i)
		time.Sleep(100 * time.Millisecond)
	}
}

func main() {
	go printNumbers("Goroutine 1")
	go printNumbers("Goroutine 2")
	printNumbers("Main")
}
```

**Explanation:**

- Three functions (`Main`, `Goroutine 1`, `Goroutine 2`) run concurrently.
- The output order varies because the Go runtime schedules the execution of the Goroutines independently.

**Output Example:**

```
Main: 1
Main: 2
Goroutine 1: 1
Goroutine 2: 1
Main: 3
...
```

---

## **10.4 Synchronizing Goroutines**

While Goroutines run concurrently, sometimes you need to synchronize them to ensure proper execution. One common way to synchronize Goroutines is by using **WaitGroups** from the `sync` package. A WaitGroup allows you to wait for a collection of Goroutines to finish before proceeding.

### **Example 3: Using WaitGroup**

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Notify WaitGroup when done
	fmt.Printf("Worker %d starting
", id)
	// Simulate work
	fmt.Printf("Worker %d done
", id)
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}

	wg.Wait() // Wait for all workers to finish
	fmt.Println("All workers finished!")
}
```

**Explanation:**

- We use a WaitGroup to wait for all Goroutines (workers) to finish.
- Each worker notifies the WaitGroup when it's done by calling `wg.Done()`.
- The `main` function waits for all workers to finish by calling `wg.Wait()`.

**Output:**

```
Worker 1 starting
Worker 2 starting
Worker 3 starting
Worker 1 done
Worker 2 done
Worker 3 done
All workers finished!
```

---

## **10.5 Avoiding Race Conditions**

Race conditions occur when multiple Goroutines access shared data simultaneously, leading to unpredictable results. To avoid race conditions, use a **Mutex**, which provides mutual exclusion, ensuring that only one Goroutine can access the shared resource at a time.

### **Example 4: Preventing Race Conditions**

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex
	counter := 0

	increment := func() {
		mu.Lock()
		counter++
		mu.Unlock()
	}

	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			increment()
		}()
	}

	wg.Wait()
	fmt.Printf("Final Counter: %d
", counter)
}
```

**Explanation:**

- The `mu.Lock()` and `mu.Unlock()` calls prevent multiple Goroutines from accessing the `counter` at the same time.
- The `main` function waits for all Goroutines to finish before printing the final value of `counter`.

**Output:**

```
Final Counter: 5
```

---

## **10.6 Buffered Channels for Communication**

Goroutines often need to communicate. Channels are used for safe data exchange between Goroutines. A **buffered channel** allows you to specify a maximum number of messages that can be stored in the channel at a time.

### **Example 5: Simple Channel Communication**

```go
package main

import "fmt"

func sendMessages(ch chan string) {
	ch <- "Hello from Goroutine"
}

func main() {
	ch := make(chan string)

	go sendMessages(ch)
	msg := <-ch // Receive message from channel
	fmt.Println(msg)
}
```

**Explanation:**

- A Goroutine sends a message through the channel `ch`.
- The `main` function receives the message from the channel and prints it.

**Output:**

```
Hello from Goroutine
```

---

## **10.7 Using Select with Channels**

The `select` statement allows a Goroutine to wait on multiple communication operations. It's similar to a `switch` but works with channels.

### **Example 6: Using Select**

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "Message from ch1"
	}()

	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- "Message from ch2"
	}()

	for i := 0; i < 2; i++ {
		select {
		case msg1 := <-ch1:
			fmt.Println(msg1)
		case msg2 := <-ch2:
			fmt.Println(msg2)
		}
	}
}
```

**Explanation:**

- The `select` statement waits for messages from either `ch1` or `ch2` and processes the first one to arrive.
- This allows Goroutines to react to multiple channels.

**Output:**

```
Message from ch1
Message from ch2
```

---

## **10.8 Timeout with Channels**

You can use `time.After` to implement a timeout in Go, which is useful when you want to limit the waiting time for receiving a message from a channel.

### **Example 7: Timeout Example**

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)

	go func() {
		time.Sleep(3 * time.Second)
		ch <- "Delayed Message"
	}()

	select {
	case msg := <-ch:
		fmt.Println(msg)
	case <-time.After(2 * time.Second):
		fmt.Println("Timeout!")
	}
}
```

**Explanation:**

- The `select` statement waits for a message from the channel `ch`, but if it doesn't arrive within the specified time (`2 * time.Second`), the "Timeout!" message is printed.

**Output:**

```
Timeout!
```

---

## **10.9 Best Practices for Goroutines**

- **Limit Goroutines:** Avoid launching too many Goroutines, as they consume system resources.
- **Synchronize Data Access:** Always synchronize access to shared data to avoid race conditions.
- **Use Channels:** Channels provide a safe and efficient way for Goroutines to communicate.

# **11.10. Exercises**

---

## **Exercise 1: Launch Multiple Goroutines**

**Problem**: Write a program that launches 5 Goroutines, each printing its ID.

```go
package main

import (
	"fmt"
	"time"
)

func printID(id int) {
	fmt.Printf("Goroutine %d running
", id)
	time.Sleep(500 * time.Millisecond)
}

func main() {
	for i := 1; i <= 5; i++ {
		go printID(i)
	}
	time.Sleep(3 * time.Second) // Wait for all Goroutines to complete
}
```

**Output (Example):**

```
Goroutine 1 running
Goroutine 2 running
Goroutine 3 running
Goroutine 4 running
Goroutine 5 running
```

---

## **Exercise 2: Synchronize with WaitGroup**

**Problem**: Use a `WaitGroup` to ensure all Goroutines complete before the program exits.

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Printf("Worker %d started
", id)
	fmt.Printf("Worker %d finished
", id)
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)
		go worker(i, &wg)
	}

	wg.Wait()
	fmt.Println("All workers completed!")
}
```

**Output:**

```
Worker 1 started
Worker 2 started
Worker 3 started
Worker 1 finished
Worker 2 finished
Worker 3 finished
All workers completed!
```

---

## **Exercise 3: Avoid Race Conditions**

**Problem**: Safely increment a shared counter using a `Mutex`.

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var mu sync.Mutex
	counter := 0

	increment := func() {
		mu.Lock()
		counter++
		mu.Unlock()
	}

	var wg sync.WaitGroup
	for i := 0; i < 5; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			increment()
		}()
	}

	wg.Wait()
	fmt.Printf("Final Counter Value: %d
", counter)
}
```

**Output:**

```
Final Counter Value: 5
```

---

## **Exercise 4: Simple Channel Communication**

**Problem**: Send and receive messages between Goroutines using channels.

```go
package main

import "fmt"

func sendMessage(ch chan string) {
	ch <- "Hello from Goroutine!"
}

func main() {
	ch := make(chan string)
	go sendMessage(ch)
	msg := <-ch
	fmt.Println(msg)
}
```

**Output:**

```
Hello from Goroutine!
```

---

## **Exercise 5: Buffered Channel**

**Problem**: Use a buffered channel to hold multiple messages.

```go
package main

import "fmt"

func main() {
	ch := make(chan string, 2)
	ch <- "Message 1"
	ch <- "Message 2"

	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

**Output:**

```
Message 1
Message 2
```

---

## **Exercise 6: Using Select with Channels**

**Problem**: Write a program that uses `select` to handle multiple channels.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "From Channel 1"
	}()

	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- "From Channel 2"
	}()

	for i := 0; i < 2; i++ {
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
From Channel 1
From Channel 2
```

---

## **Exercise 7: Implement Timeout with Channels**

**Problem**: Use `time.After` to implement a timeout while waiting for channel input.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan string)

	go func() {
		time.Sleep(3 * time.Second)
		ch <- "Delayed Message"
	}()

	select {
	case msg := <-ch:
		fmt.Println(msg)
	case <-time.After(2 * time.Second):
		fmt.Println("Timeout!")
	}
}
```

**Output:**

```
Timeout!
```

---

## **Exercise 8: Worker Pool**

**Problem**: Create a worker pool with Goroutines and channels.

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Printf("Worker %d processing job %d
", id, j)
		results <- j * 2
	}
}

func main() {
	const numJobs = 5
	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	for w := 1; w <= 3; w++ {
		go worker(w, jobs, results)
	}

	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	for a := 1; a <= numJobs; a++ {
		fmt.Println("Result:", <-results)
	}
}
```

**Output:**

```
Worker 1 processing job 1
Worker 2 processing job 2
...
Result: 2
Result: 4
...
```

---

## **Exercise 9: Fan-In Channel**

**Problem**: Combine multiple input channels into one output channel.

```go
package main

import "fmt"

func fanIn(ch1, ch2 <-chan string, output chan<- string) {
	go func() {
		for msg := range ch1 {
			output <- msg
		}
	}()
	go func() {
		for msg := range ch2 {
			output <- msg
		}
	}()
}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)
	output := make(chan string)

	go func() {
		ch1 <- "Message from ch1"
		close(ch1)
	}()
	go func() {
		ch2 <- "Message from ch2"
		close(ch2)
	}()

	fanIn(ch1, ch2, output)

	for i := 0; i < 2; i++ {
		fmt.Println(<-output)
	}
}
```

**Output:**

```
Message from ch1
Message from ch2
```

---

## **Exercise 10: Goroutines with Anonymous Functions**

**Problem**: Launch Goroutines with anonymous functions.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	for i := 1; i <= 3; i++ {
		go func(id int) {
			fmt.Printf("Goroutine %d running
", id)
			time.Sleep(500 * time.Millisecond)
		}(i)
	}
	time.Sleep(2 * time.Second)
}
```

**Output:**

```
Goroutine 1 running
Goroutine 2 running
Goroutine 3 running
```

---
