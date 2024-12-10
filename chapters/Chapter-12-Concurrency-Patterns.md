# **Chapter 12: Concurrency Patterns**

---

## **1. Introduction to Concurrency Patterns**

### Why Use Concurrency Patterns?

- **Efficiency**: Leverage multi-core CPUs to handle tasks in parallel.
- **Scalability**: Distribute workloads across goroutines for better resource utilization.
- **Maintainability**: Use patterns to simplify complex concurrency logic.

---

## **2. Worker Pools**

A **Worker Pool** is a concurrency pattern that processes tasks using a fixed number of worker goroutines. It helps manage resource consumption by limiting the number of goroutines running simultaneously.

---

### **2.1 Worker Pool by Example**

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
		fmt.Printf("Worker %d processing job %d\n", id, job)
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
		fmt.Printf("Result: %d\n", <-results)
	}
}
```

#### **Expected Output:**

```plaintext
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

### **2.2 Extending Worker Pool**

Let’s add error handling. What if some jobs fail?

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
		fmt.Printf("Worker %d processing job %d\n", id, job)
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
			fmt.Printf("Result: %d\n", res)
		case err := <-errors:
			fmt.Printf("Error: %s\n", err)
		}
	}
}
```

#### **Expected Output:**

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

## **3. Fan-In and Fan-Out**

### What is Fan-In and Fan-Out?

- **Fan-Out**: Distribute tasks across multiple goroutines.
- **Fan-In**: Aggregate results from multiple channels into one channel.

---

### **3.1 Fan-Out by Example**

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
		fmt.Printf("Worker %d squaring %d\n", id, num)
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
		fmt.Printf("Result: %d\n", <-results)
	}
}
```

#### **Expected Output:**

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

### **3.2 Fan-In by Example**

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
		fmt.Printf("Merged Value: %d\n", val)
	}
}
```

#### **Expected Output:**

```plaintext
Merged Value: 1
Merged Value: 6
Merged Value: 2
Merged Value: 7
...
```

---

# **4: Exercises**

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

**Stay Tuned!**
The full exercise set includes advanced examples like pipeline chaining, concurrency-safe counters, and goroutine leak prevention. Use these patterns to optimize real-world applications!
