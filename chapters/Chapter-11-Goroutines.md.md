# **Chapter 10: Goroutines**

---

## **10.1 What Are Goroutines?**

Goroutines are Go's lightweight threads managed by the Go runtime. Unlike traditional threads:

- They are highly efficient and can handle thousands of concurrent tasks.
- They are multiplexed onto operating system threads by the Go runtime, saving resources.

Think of a Goroutine as a function running concurrently with other Goroutines in the same application.

---

## **10.2 Launching Goroutines**

To start a Goroutine, simply use the `go` keyword before a function call.

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

### **Output:**

```
Hello from Main!
Hello from Goroutine!
```

### **Explanation:**

- `go sayHello()` starts the `sayHello` function as a Goroutine.
- The `main` function continues execution without waiting for the Goroutine unless explicitly synchronized (e.g., `time.Sleep`).

---

## **10.3 Understanding Concurrency**

Goroutines execute independently, but their execution order is unpredictable. This is due to the concurrent nature of Goroutines.

### **Example 2: Multiple Goroutines**

```go
package main

import (
	"fmt"
	"time"
)

func printNumbers(name string) {
	for i := 1; i <= 5; i++ {
		fmt.Printf("%s: %d\n", name, i)
		time.Sleep(100 * time.Millisecond)
	}
}

func main() {
	go printNumbers("Goroutine 1")
	go printNumbers("Goroutine 2")
	printNumbers("Main")
}
```

### **Output (Example):**

```
Main: 1
Main: 2
Goroutine 1: 1
Goroutine 2: 1
Main: 3
...
```

### **Explanation:**

- Three functions (`Main`, `Goroutine 1`, `Goroutine 2`) run concurrently.
- The output order varies based on scheduling by the Go runtime.

---

## **10.4 Synchronizing Goroutines**

To ensure proper execution of Goroutines, use synchronization techniques like **WaitGroups** from the `sync` package.

### **Example 3: Using WaitGroup**

```go
package main

import (
	"fmt"
	"sync"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Notify WaitGroup when done
	fmt.Printf("Worker %d starting\n", id)
	// Simulate work
	fmt.Printf("Worker %d done\n", id)
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

### **Output:**

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

Race conditions occur when multiple Goroutines access shared data simultaneously. Use **Mutex** to prevent this.

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
	fmt.Printf("Final Counter: %d\n", counter)
}
```

### **Output:**

```
Final Counter: 5
```

---

## **10.6 Buffered Channels for Communication**

Goroutines often need to communicate. Channels provide a way to share data safely.

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

### **Output:**

```
Hello from Goroutine
```

---

## **10.7 Using Select with Channels**

`select` allows a Goroutine to wait on multiple communication operations.

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

### **Output:**

```
Message from ch1
Message from ch2
```

---

## **10.8 Timeout with Channels**

Use `time.After` to implement timeouts with channels.

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

### **Output:**

```
Timeout!
```

---

## **10.9 Best Practices for Goroutines**

- Limit Goroutine usage to avoid excessive resource consumption.
- Always synchronize shared data to prevent race conditions.
- Use channels for safe communication between Goroutines.

---

# **10.10. Exercises**

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

**Congratulations!** You've completed the exercises for Goroutines. These examples cover fundamental and advanced concurrency concepts, preparing you for real-world use cases.
