# **Chapter 13: Concurrency Patterns in Go**

In Chapter 11, we explored goroutines as the foundation of Go's concurrency model, and in Chapter 12, we delved deeper into channels for communication between goroutines. Now, we'll build upon this foundation to explore common concurrency patterns in Go.

Concurrency patterns are reusable solutions to common problems encountered when writing concurrent programs. These patterns help you structure your code to efficiently handle parallelism, manage resources, and avoid common pitfalls like deadlocks, race conditions, and excessive resource consumption.

In this chapter, we'll explore several essential concurrency patterns, including worker pools, fan-out/fan-in, pipelines, and others that will help you write robust concurrent applications in Go.

## **13.1 Worker Pools**

A worker pool is a pattern where a fixed number of worker goroutines process tasks from a shared queue. This pattern is useful for limiting resource usage and processing multiple tasks concurrently.

### **13.1.1 Basic Worker Pool Implementation**

```go
package main

import (
	"fmt"
	"time"
)

// Worker processes jobs from the jobs channel and sends results to the results channel
func worker(id int, jobs <-chan int, results chan<- int) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job)
		time.Sleep(time.Second) // Simulate processing time
		results <- job * 2      // Send result
	}
}

func main() {
	const numJobs = 10
	const numWorkers = 3

	// Create job and result channels
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
	close(jobs) // No more jobs will be sent

	// Collect results
	for a := 1; a <= numJobs; a++ {
		result := <-results
		fmt.Println("Result:", result)
	}
}
```

In this example:

1. We create a fixed number of worker goroutines
2. Each worker processes jobs from the shared `jobs` channel
3. Workers send results to the `results` channel
4. The main goroutine collects all results

### **13.1.2 Worker Pool with Error Handling**

Real-world tasks can fail, so adding error handling is essential:

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// Job represents a task to be processed
type Job struct {
	ID      int
	Data    int
}

// Result includes both the result data and possible error
type Result struct {
	Job     Job
	Value   int
	Err     error
}

// Worker processes jobs and handles potential errors
func worker(id int, jobs <-chan Job, results chan<- Result) {
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job.ID)

		// Simulate work with possible errors
		time.Sleep(time.Second)

		// Randomly simulate errors (30% chance)
		if rand.Float32() < 0.3 {
			results <- Result{
				Job: job,
				Err: fmt.Errorf("error processing job %d", job.ID),
			}
			continue
		}

		// Success case
		results <- Result{
			Job:   job,
			Value: job.Data * 2,
			Err:   nil,
		}
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())

	const numJobs = 10
	const numWorkers = 3

	jobs := make(chan Job, numJobs)
	results := make(chan Result, numJobs)

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		go worker(w, jobs, results)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- Job{ID: j, Data: j * 10}
	}
	close(jobs)

	// Collect results, handling errors
	for a := 1; a <= numJobs; a++ {
		result := <-results
		if result.Err != nil {
			fmt.Printf("Error: %v\n", result.Err)
		} else {
			fmt.Printf("Success: Job %d, Result: %d\n",
				result.Job.ID, result.Value)
		}
	}
}
```

This implementation:

- Uses custom types for jobs and results
- Includes error information in the result
- Properly handles both successful and failed jobs

### **13.1.3 Worker Pool with Done Channel**

For proper cleanup and termination, we can add a done channel:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int, done <-chan struct{}, wg *sync.WaitGroup) {
	defer wg.Done()

	for {
		select {
		case job, ok := <-jobs:
			if !ok {
				// Channel closed, no more jobs
				return
			}
			fmt.Printf("Worker %d processing job %d\n", id, job)
			time.Sleep(time.Second)
			results <- job * 2

		case <-done:
			// Received termination signal
			fmt.Printf("Worker %d terminating\n", id)
			return
		}
	}
}

func main() {
	const numWorkers = 3

	jobs := make(chan int, 10)
	results := make(chan int, 10)
	done := make(chan struct{})

	var wg sync.WaitGroup

	// Start workers
	for w := 1; w <= numWorkers; w++ {
		wg.Add(1)
		go worker(w, jobs, results, done, &wg)
	}

	// Send some jobs
	for j := 1; j <= 5; j++ {
		jobs <- j
	}

	// Process results in a separate goroutine
	go func() {
		for result := range results {
			fmt.Println("Result:", result)
		}
	}()

	// Allow some work to happen
	time.Sleep(3 * time.Second)

	// Signal all workers to terminate
	close(done)

	// Wait for all workers to exit
	wg.Wait()
	fmt.Println("All workers have terminated")
}
```

This implementation adds:

- A done channel for signaling termination
- A WaitGroup to wait for all workers to exit
- Proper cleanup when workers are terminated

## **13.2 Fan-Out, Fan-In Pattern**

The Fan-Out, Fan-In pattern involves:

- **Fan-Out**: Distributing work across multiple goroutines
- **Fan-In**: Collecting results from multiple goroutines into a single channel

This pattern is ideal for CPU-bound tasks that can be broken down into independent units of work.

### **13.2.1 Basic Fan-Out, Fan-In Implementation**

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// Processor performs work on input
func processor(id int, input int) int {
	fmt.Printf("Processor %d processing input: %d\n", id, input)
	time.Sleep(time.Second) // Simulate work
	return input * input
}

// fanOut distributes work across multiple goroutines
func fanOut(inputs []int, workers int) []<-chan int {
	// Create a channel for each worker
	channels := make([]<-chan int, workers)

	// Distribute the work
	for i := 0; i < workers; i++ {
		ch := make(chan int)
		channels[i] = ch

		go func(workerID int, ch chan<- int) {
			defer close(ch)

			// Each worker processes inputs with an index % workers == workerID
			for j, input := range inputs {
				if j % workers == workerID {
					ch <- processor(workerID, input)
				}
			}
		}(i, ch)
	}

	return channels
}

// fanIn consolidates results from multiple channels into one
func fanIn(channels []<-chan int) <-chan int {
	var wg sync.WaitGroup
	merged := make(chan int)

	// Start an output goroutine for each input channel
	output := func(ch <-chan int) {
		defer wg.Done()
		for val := range ch {
			merged <- val
		}
	}

	wg.Add(len(channels))
	for _, ch := range channels {
		go output(ch)
	}

	// Start a goroutine to close the merged channel after all inputs are done
	go func() {
		wg.Wait()
		close(merged)
	}()

	return merged
}

func main() {
	// Input data
	inputs := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	// Fan-out: distribute work across 3 processors
	channels := fanOut(inputs, 3)

	// Fan-in: collect results
	results := fanIn(channels)

	// Consume results
	for result := range results {
		fmt.Println("Result:", result)
	}
}
```

This pattern is particularly useful when:

- You have a large number of independent tasks to process
- Tasks can be processed in parallel
- You want to limit concurrency to a specific number of workers

### **13.2.2 Advanced Fan-Out, Fan-In with Cancelation**

Adding cancellation support makes the pattern more robust:

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

// processor performs work with cancellation support
func processor(ctx context.Context, id int, input int) (int, error) {
	select {
	case <-time.After(time.Second): // Simulate work
		return input * input, nil
	case <-ctx.Done():
		return 0, ctx.Err()
	}
}

// fanOut with context for cancellation
func fanOut(ctx context.Context, inputs []int, workers int) []<-chan result {
	channels := make([]<-chan result, workers)

	for i := 0; i < workers; i++ {
		ch := make(chan result)
		channels[i] = ch

		go func(workerID int, ch chan<- result) {
			defer close(ch)

			for j, input := range inputs {
				if j % workers == workerID {
					value, err := processor(ctx, workerID, input)
					ch <- result{Value: value, Err: err}
				}
			}
		}(i, ch)
	}

	return channels
}

// result includes both value and error
type result struct {
	Value int
	Err   error
}

// fanIn with error propagation
func fanIn(ctx context.Context, channels []<-chan result) <-chan result {
	var wg sync.WaitGroup
	merged := make(chan result)

	output := func(ch <-chan result) {
		defer wg.Done()
		for res := range ch {
		select {
			case merged <- res:
			case <-ctx.Done():
				return
			}
		}
	}

	wg.Add(len(channels))
	for _, ch := range channels {
		go output(ch)
	}

	go func() {
		wg.Wait()
	close(merged)
	}()

	return merged
}

func main() {
	// Create a context with cancellation
	ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second)
	defer cancel()

	// Input data
	inputs := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	// Fan-out
	channels := fanOut(ctx, inputs, 3)

	// Fan-in
	results := fanIn(ctx, channels)

	// Consume results, handling errors
	for res := range results {
		if res.Err != nil {
			fmt.Printf("Error: %v\n", res.Err)
		} else {
			fmt.Println("Result:", res.Value)
		}
	}
}
```

This enhanced implementation:

- Uses context for cancellation
- Properly propagates errors
- Can be canceled by timeout or explicit cancellation

## **13.3 Pipeline Pattern**

A pipeline is a series of stages connected by channels, where each stage:

1. Receives data from upstream
2. Performs some processing
3. Sends the result downstream

Pipelines are effective for breaking complex processing into discrete, reusable stages.

### **13.3.1 Basic Pipeline Implementation**

```go
package main

import (
	"fmt"
)

// generator - the first stage that produces data
func generator(nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			out <- n
		}
	}()
	return out
}

// square - the second stage that squares numbers
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

// filter - the third stage that filters out odd numbers
func filter(in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			if n%2 == 0 { // Only keep even numbers
				out <- n
			}
		}
	}()
	return out
}

func main() {
	// Set up the pipeline
	// Stage 1: Generate integers
	numbers := generator(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

	// Stage 2: Square the numbers
	squares := square(numbers)

	// Stage 3: Filter out odd squares
	filtered := filter(squares)

	// Consume the output
	for n := range filtered {
		fmt.Println(n)
	}
}
```

This pipeline:

- Generates a sequence of numbers
- Squares each number
- Filters out odd results
- Each stage runs in its own goroutine, enabling concurrent processing

### **13.3.2 Pipeline with Cancelation**

Adding cancellation makes pipelines more robust:

```go
package main

import (
	"context"
	"fmt"
)

// generator with context
func generator(ctx context.Context, nums ...int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for _, n := range nums {
			select {
			case out <- n:
			case <-ctx.Done():
				return
			}
		}
	}()
	return out
}

// square with context
func square(ctx context.Context, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			select {
			case out <- n * n:
			case <-ctx.Done():
				return
			}
		}
	}()
	return out
}

// filter with context
func filter(ctx context.Context, in <-chan int) <-chan int {
	out := make(chan int)
	go func() {
		defer close(out)
		for n := range in {
			if n%2 == 0 {
				select {
				case out <- n:
				case <-ctx.Done():
					return
				}
			}
		}
	}()
	return out
}

func main() {
	// Create a context that can be canceled
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel() // Ensure all resources are released

	// Set up the pipeline with cancellation
	numbers := generator(ctx, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
	squares := square(ctx, numbers)
	filtered := filter(ctx, squares)

	// Consume just a few values, then cancel
	for i := 0; i < 3; i++ {
		value, ok := <-filtered
		if !ok {
			break
		}
		fmt.Println(value)
	}

	// Cancel the pipeline early
	cancel()
	fmt.Println("Pipeline canceled")
}
```

This implementation:

- Adds context to each stage for cancellation
- Properly handles early termination
- Releases resources when the pipeline is canceled

## **13.4 Timeout and Cancelation Patterns**

Managing timeouts and cancellation is crucial for robust concurrent programs.

### **13.4.1 Timeout Pattern**

The timeout pattern prevents operations from blocking indefinitely:

```go
package main

import (
	"fmt"
	"time"
)

func operation(timeout time.Duration) (string, error) {
	ch := make(chan string)

	// Start the operation
	go func() {
		// Simulate a long-running operation
		time.Sleep(2 * time.Second)
		ch <- "Operation completed"
	}()

	// Wait for the result or timeout
	select {
	case result := <-ch:
		return result, nil
	case <-time.After(timeout):
		return "", fmt.Errorf("operation timed out after %v", timeout)
	}
}

func main() {
	// Try with 1 second timeout (should fail)
	result, err := operation(1 * time.Second)
	if err != nil {
		fmt.Println("First attempt:", err)
	} else {
		fmt.Println("First attempt:", result)
	}

	// Try with 3 second timeout (should succeed)
	result, err = operation(3 * time.Second)
	if err != nil {
		fmt.Println("Second attempt:", err)
	} else {
		fmt.Println("Second attempt:", result)
	}
}
```

This pattern is useful for:

- External API calls
- Network operations
- Any long-running operation that should be bounded in time

### **13.4.2 Cancellation with Context**

The context package provides a standardized way to handle cancellation:

```go
package main

import (
	"context"
	"fmt"
	"time"
)

// longRunningOperation simulates work that can be canceled
func longRunningOperation(ctx context.Context) (string, error) {
	// Create a channel for the result
	resultCh := make(chan string)

	go func() {
		// Simulate steps in the operation
		for i := 1; i <= 5; i++ {
			// Check if context was canceled
			select {
			case <-ctx.Done():
				return // Exit the goroutine
			case <-time.After(500 * time.Millisecond):
				// Continue with the next step
				fmt.Printf("Step %d completed\n", i)
			}
		}

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
