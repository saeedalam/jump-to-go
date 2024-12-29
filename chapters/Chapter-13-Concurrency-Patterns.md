# **Chapter 13: Concurrency Patterns**

---

## **13.1. Introduction to Concurrency Patterns**

### Why Use Concurrency Patterns?

Concurrency patterns help in organizing and structuring concurrent code to make it efficient, maintainable, and scalable. Using proper patterns, you can leverage the full potential of concurrency and parallelism without introducing complexity or bugs.

- **Efficiency**: Leverage multi-core CPUs to handle tasks in parallel.
- **Scalability**: Distribute workloads across goroutines for better resource utilization.
- **Maintainability**: Use patterns to simplify complex concurrency logic.

By employing concurrency patterns, you can avoid pitfalls like race conditions, and ensure that your code is both safe and optimized for real-world tasks.

---

## **13.2. Worker Pools**

A **Worker Pool** is a concurrency pattern that processes tasks using a fixed number of worker goroutines. It helps manage resource consumption by limiting the number of goroutines running simultaneously, preventing unnecessary strain on the system.

### How Worker Pools Work:

- A **worker** is a goroutine that performs a specific task (such as processing data).
- The **job queue** stores tasks that need to be processed.
- Workers **fetch jobs** from the job queue and process them in parallel, while putting results into a result channel.
- This pattern improves system efficiency and allows for easy scaling by adjusting the number of workers.

---

### **13.2.1 Worker Pool by Example**

#### Problem Statement:

Process a list of numbers and compute their squares using multiple workers.

#### **Code Implementation:**

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d
", id, job)
		time.Sleep(time.Second) // Simulate processing time
		results <- job * job
	}
}

func main() {
	const numJobs = 5
	const numWorkers = 3

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		go worker(w, jobs, results)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Collect results
	for a := 1; a <= numJobs; a++ {
		fmt.Printf("Result: %d
", <-results)
	}
}
```

#### **Explanation:**

- **Worker function**: It simulates processing each job by sleeping for one second and then computing the square of the job number.
- **Main function**: It starts 3 worker goroutines and sends 5 jobs for processing. After the jobs are processed, results are collected.

#### **Expected Output:**

```
Worker 1 processing job 1
Worker 2 processing job 2
Worker 3 processing job 3
Worker 1 processing job 4
Worker 2 processing job 5
Result: 1
Result: 4
Result: 9
Result: 16
Result: 25
```

---

### **13.2.2 Extending Worker Pool with Error Handling**

Let’s extend the Worker Pool by adding error handling. What if some jobs fail?

#### **Code Implementation:**

```go
package main

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

func workerWithErrors(id int, jobs <-chan int, results chan<- int, errors chan<- error) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d
", id, job)
		time.Sleep(time.Second) // Simulate processing time
		if rand.Float32() < 0.3 { // Randomly simulate errors
			errors <- fmt.Errorf("worker %d failed on job %d", id, job)
		} else {
			results <- job * job
		}
	}
}

func main() {
	const numJobs = 5
	const numWorkers = 3

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	errors := make(chan error, numJobs)

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		go workerWithErrors(w, jobs, results, errors)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Collect results or errors
	for a := 1; a <= numJobs; a++ {
		select {
		case res := <-results:
			fmt.Printf("Result: %d
", res)
		case err := <-errors:
			fmt.Printf("Error: %s
", err)
		}
	}
}
```

#### **Explanation:**

- This version introduces an error channel. If a worker fails to process a job (simulated randomly with `rand.Float32()`), an error message is sent to the error channel. Otherwise, the result is sent to the result channel.
- The `select` statement handles either the results or errors depending on which channel is ready.

#### **Expected Output:**

```
Worker 1 processing job 1
Worker 2 processing job 2
Worker 3 processing job 3
Error: worker 2 failed on job 2
Result: 9
Result: 1
...
```

---

## **13.3. Fan-In and Fan-Out**

### What is Fan-In and Fan-Out?

- **Fan-Out**: Distribute tasks across multiple goroutines to speed up processing.
- **Fan-In**: Aggregate results from multiple channels into one channel to handle results in a unified manner.

These patterns are highly useful when you want to distribute and aggregate work efficiently.

---

### **13.3.1 Fan-Out by Example**

#### Problem Statement:

Distribute numbers among multiple workers to compute squares.

#### **Code Implementation:**

```go
package main

import (
	"fmt"
	"time"
)

func squareWorker(id int, nums <-chan int, results chan<- int) {
	for num := range nums {
		fmt.Printf("Worker %d squaring %d
", id, num)
		time.Sleep(time.Second) // Simulate processing time
		results <- num * num
	}
}

func main() {
	numbers := []int{1, 2, 3, 4, 5}
	numWorkers := 3

	nums := make(chan int, len(numbers))
	results := make(chan int, len(numbers))

	// Start workers
	for i := 1; i <= numWorkers; i++ {
		go squareWorker(i, nums, results)
	}

	// Send numbers to workers
	for _, num := range numbers {
		nums <- num
	}
	close(nums)

	// Collect results
	for range numbers {
		fmt.Printf("Result: %d
", <-results)
	}
}
```

#### **Explanation:**

- This example demonstrates the **Fan-Out** pattern. We create multiple worker goroutines that process a list of numbers concurrently. Each worker computes the square of the numbers sent to them through the channel.

#### **Expected Output:**

```
Worker 1 squaring 1
Worker 2 squaring 2
Worker 3 squaring 3
Worker 1 squaring 4
Worker 2 squaring 5
Result: 1
Result: 4
Result: 9
Result: 16
Result: 25
```

---

### **13.3.2 Fan-In by Example**

#### Problem Statement:

Aggregate results from two separate channels into one channel.

#### **Code Implementation:**

```go
package main

import (
	"fmt"
	"time"
)

func generator(start, end int, ch chan<- int) {
	for i := start; i <= end; i++ {
		ch <- i
		time.Sleep(500 * time.Millisecond)
	}
	close(ch)
}

func fanIn(ch1, ch2 <-chan int, merged chan<- int) {
	for ch1 != nil || ch2 != nil {
		select {
		case val, ok := <-ch1:
			if ok {
				merged <- val
			} else {
				ch1 = nil
			}
		case val, ok := <-ch2:
			if ok {
				merged <- val
			} else {
				ch2 = nil
			}
		}
	}
	close(merged)
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	merged := make(chan int)

	go generator(1, 5, ch1)
	go generator(6, 10, ch2)
	go fanIn(ch1, ch2, merged)

	for val := range merged {
		fmt.Printf("Merged Value: %d
", val)
	}
}
```

#### **Explanation:**

- This example demonstrates the **Fan-In** pattern. We generate two sequences of numbers in separate goroutines (`ch1` and `ch2`), and then merge them into one channel (`merged`) using the `fanIn` function.

#### **Expected Output:**

```
Merged Value: 1
Merged Value: 6
Merged Value: 2
Merged Value: 7
...
```

---

## **Key Takeaways**

1. **Worker Pools** help distribute workload among a fixed number of workers, managing concurrency and avoiding system overload.
2. **Fan-Out** distributes tasks across multiple workers, and **Fan-In** combines results from multiple channels into a single one for easier aggregation.
3. Patterns like these allow you to efficiently utilize goroutines and channels to build scalable and maintainable concurrent systems in Go.

---

# **13.4. Exercises**

## **Exercise 1: Basic Worker Pool**

**Problem**: Create a worker pool to process numbers and calculate their cubes.

```go
package main

import (
	"fmt"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d
", id, job)
		time.Sleep(time.Second) // Simulate processing time
		results <- job * job * job
	}
}

func main() {
	const numJobs = 5
	const numWorkers = 3

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		go worker(w, jobs, results)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Collect results
	for a := 1; a <= numJobs; a++ {
		fmt.Printf("Result: %d
", <-results)
	}
}
```

**Expected Output:**

```plaintext
Worker 1 processing job 1
Worker 2 processing job 2
Worker 3 processing job 3
Worker 1 processing job 4
Worker 2 processing job 5
Result: 1
Result: 8
Result: 27
Result: 64
Result: 125
```

---

## **Exercise 2: Worker Pool with Error Handling**

**Problem**: Modify the worker pool to include error handling for failed tasks.

```go
package main

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

func workerWithErrors(id int, jobs <-chan int, results chan<- int, errors chan<- error) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d
", id, job)
		time.Sleep(time.Second) // Simulate processing time
		if rand.Float32() < 0.3 { // Randomly simulate errors
			errors <- fmt.Errorf("worker %d failed on job %d", id, job)
		} else {
			results <- job * job
		}
	}
}

func main() {
	const numJobs = 5
	const numWorkers = 3

	jobs := make(chan int, numJobs)
	results := make(chan int, numJobs)
	errors := make(chan error, numJobs)

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		go workerWithErrors(w, jobs, results, errors)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs)

	// Collect results or errors
	for a := 1; a <= numJobs; a++ {
		select {
		case res := <-results:
			fmt.Printf("Result: %d
", res)
		case err := <-errors:
			fmt.Printf("Error: %s
", err)
		}
	}
}
```

**Expected Output**:
Results vary due to randomness. Example:

```plaintext
Worker 1 processing job 1
Worker 2 processing job 2
Worker 3 processing job 3
Error: worker 2 failed on job 2
Result: 9
Result: 1
...
```

---

## **Exercise 3: Fan-Out**

**Problem**: Write a program to distribute tasks across multiple workers to calculate squares.

```go
package main

import (
	"fmt"
	"time"
)

func squareWorker(id int, nums <-chan int, results chan<- int) {
	for num := range nums {
		fmt.Printf("Worker %d squaring %d
", id, num)
		time.Sleep(time.Second) // Simulate processing time
		results <- num * num
	}
}

func main() {
	numbers := []int{1, 2, 3, 4, 5}
	numWorkers := 3

	nums := make(chan int, len(numbers))
	results := make(chan int, len(numbers))

	// Start workers
	for i := 1; i <= numWorkers; i++ {
		go squareWorker(i, nums, results)
	}

	// Send numbers to workers
	for _, num := range numbers {
		nums <- num
	}
	close(nums)

	// Collect results
	for range numbers {
		fmt.Printf("Result: %d
", <-results)
	}
}
```

**Expected Output**:

```plaintext
Worker 1 squaring 1
Worker 2 squaring 2
Worker 3 squaring 3
Worker 1 squaring 4
Worker 2 squaring 5
Result: 1
Result: 4
Result: 9
Result: 16
Result: 25
```

---

## **Exercise 4: Fan-In**

**Problem**: Aggregate results from two separate channels into one.

```go
package main

import (
	"fmt"
	"time"
)

func generator(start, end int, ch chan<- int) {
	for i := start; i <= end; i++ {
		ch <- i
		time.Sleep(500 * time.Millisecond)
	}
	close(ch)
}

func fanIn(ch1, ch2 <-chan int, merged chan<- int) {
	for ch1 != nil || ch2 != nil {
		select {
		case val, ok := <-ch1:
			if ok {
				merged <- val
			} else {
				ch1 = nil
			}
		case val, ok := <-ch2:
			if ok {
				merged <- val
			} else {
				ch2 = nil
			}
		}
	}
	close(merged)
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	merged := make(chan int)

	go generator(1, 5, ch1)
	go generator(6, 10, ch2)
	go fanIn(ch1, ch2, merged)

	for val := range merged {
		fmt.Printf("Merged Value: %d
", val)
	}
}
```

**Expected Output**:

```plaintext
Merged Value: 1
Merged Value: 6
Merged Value: 2
Merged Value: 7
...
```

---

## **Exercise 5: Timeout Handling with Channels**

**Problem**: Implement a timeout mechanism for waiting for data on a channel.

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

**Expected Output**:

```plaintext
Timeout
```

---

## **Exercise 6: Pipeline Pattern**

**Problem**: Create a pipeline where data is processed by multiple stages in sequence.

```go
package main

import (
	"fmt"
)

func stage1(ch chan<- int) {
	ch <- 1
	ch <- 2
	close(ch)
}

func stage2(ch1 <-chan int, ch2 chan<- int) {
	for val := range ch1 {
		ch2 <- val * 2
	}
	close(ch2)
}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go stage1(ch1)
	go stage2(ch1, ch2)

	for result := range ch2 {
		fmt.Println("Processed:", result)
	}
}
```

**Expected Output**:

```plaintext
Processed: 2
Processed: 4
```

---

**Stay Tuned!**  
These exercises explore real-world concurrency patterns and techniques. Completing them will give you the skills needed to handle complex concurrency problems in Go!
