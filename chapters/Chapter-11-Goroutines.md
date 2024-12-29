# **Chapter 11: Goroutines**

---

## **11.1 What Are Goroutines?**

Goroutines are Go's lightweight threads managed by the Go runtime. Unlike traditional threads:

- **Efficiency:** They are highly efficient and can handle thousands of concurrent tasks.
- **Multiplexing:** They are multiplexed onto operating system threads by the Go runtime, saving resources.

Think of a Goroutine as a function running concurrently with other Goroutines in the same application. Goroutines allow you to perform tasks concurrently without worrying about complex thread management.

---

## **11.2 Launching Goroutines**

In Go, launching a Goroutine is as simple as placing the `go` keyword before a function call. When you use `go` before a function, Go starts that function as a Goroutine and immediately moves on to the next instruction in the calling function, without waiting for the Goroutine to finish.

### **Example: Starting a Simple Goroutine**

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
- The `main` function continues executing without waiting for the Goroutine to complete unless explicitly synchronized.
- The `time.Sleep` is used to give the Goroutine enough time to execute before the program ends.

**Output:**

```
Hello from Main!
Hello from Goroutine!
```

---

## **11.3 Understanding Concurrency**

Goroutines execute independently and concurrently. This means that the execution order of Goroutines is unpredictable, and it depends on how the Go runtime schedules them. This is an essential concept when dealing with concurrency: Goroutines can run in any order, and we cannot assume that one will finish before the other.

### **Example: Multiple Goroutines**

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

## **11.4 Synchronizing Goroutines**

While Goroutines run concurrently, sometimes you need to synchronize them to ensure proper execution. One common way to synchronize Goroutines is by using **WaitGroups** from the `sync` package. A WaitGroup allows you to wait for a collection of Goroutines to finish before proceeding.

### **Example: Using WaitGroup**

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

## **11.5 Avoiding Race Conditions**

Race conditions occur when multiple Goroutines access shared data simultaneously, leading to unpredictable results. To avoid race conditions, use a **Mutex**, which provides mutual exclusion, ensuring that only one Goroutine can access the shared resource at a time.

### **Example: Preventing Race Conditions**

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

## **11.6 Goroutines Best Practices**

1. **Avoid Too Many Goroutines:** While Goroutines are lightweight, launching too many can still overwhelm the system. Use worker pools or limit the number of active Goroutines.
2. **Use Synchronization Tools:** Employ `sync.WaitGroup` or `sync.Mutex` to properly synchronize Goroutines and avoid race conditions.
3. **Monitor Resource Usage:** Debugging tools like `runtime.NumGoroutine()` can help you track the number of active Goroutines and ensure efficient resource management.
4. **Understand Concurrency:** Be mindful of the unpredictable execution order and use synchronization techniques where necessary.

---

## **11.7 Exercises**

### **Exercise 1: Launch Multiple Goroutines**

**Problem:** Write a program that launches 5 Goroutines, each printing its ID.

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

**Explanation:**

- The `printID` function prints the ID of a Goroutine and simulates work with `time.Sleep`.
- `go` launches the `printID` function as a Goroutine.
- `time.Sleep` in `main` ensures the main Goroutine waits for all other Goroutines to complete.

---

### **Exercise 2: Synchronize with WaitGroup**

**Problem:** Use a `WaitGroup` to ensure all Goroutines complete before the program exits.

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

**Explanation:**

- A `WaitGroup` ensures all workers finish before `main` exits.
- `wg.Add(1)` adds to the count of workers.
- `defer wg.Done()` decrements the count when a worker finishes.
- `wg.Wait()` blocks until all workers are done.

---

### **Exercise 3: Avoid Race Conditions**

**Problem:** Safely increment a shared counter using a `Mutex`.

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

**Explanation:**

- A `Mutex` prevents race conditions by ensuring only one Goroutine modifies `counter` at a time.
- `mu.Lock()` locks the critical section, and `mu.Unlock()` releases it.

---

### **Exercise 4: Measure Goroutines**

**Problem:** Count and print the number of active Goroutines.

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func doWork() {
	time.Sleep(1 * time.Second)
}

func main() {
	for i := 0; i < 5; i++ {
		go doWork()
	}
	fmt.Printf("Number of Goroutines: %d
", runtime.NumGoroutine())
	time.Sleep(2 * time.Second)
	fmt.Printf("Number of Goroutines after sleep: %d
", runtime.NumGoroutine())
}
```

**Explanation:**

- The `runtime.NumGoroutine()` function returns the current number of active Goroutines.
- Goroutines that finish reduce the count automatically.

---

## **Key Takeaways**

- Goroutines enable lightweight concurrency in Go, allowing thousands of tasks to run concurrently.
- Synchronization tools like `WaitGroup` and `Mutex` are critical for coordinating Goroutines and preventing race conditions.
- Efficient use of Goroutines can lead to high-performance, scalable applications.
