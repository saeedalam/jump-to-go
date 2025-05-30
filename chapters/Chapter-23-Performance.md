# **Chapter 23: Performance Optimization in Go**

Performance optimization is a critical skill for Go developers. Go was designed with performance in mind, but writing truly efficient code requires understanding the language's internals, runtime behavior, and best practices for optimization. This chapter explores how to identify, measure, and optimize performance bottlenecks in Go applications.

## **23.1 Understanding Performance in Go**

### **23.1.1 Go's Performance Philosophy**

Go's design philosophy prioritizes certain performance characteristics:

1. **Fast Compilation**: Go compiles quickly, enabling rapid development cycles.
2. **Efficient Execution**: Go generates efficient machine code and includes a sophisticated garbage collector.
3. **Low Memory Overhead**: Go's type system and runtime are designed to minimize memory usage.
4. **Scalable Concurrency**: Goroutines provide lightweight concurrency with minimal overhead.
5. **Predictable Performance**: Go aims for consistent performance with minimal surprises.

However, Go also makes deliberate trade-offs:

- Garbage collection provides memory safety but introduces some overhead
- Simplicity is favored over maximum theoretical performance
- The standard library prioritizes correctness and clarity over extreme optimization

Understanding these trade-offs helps set realistic expectations for optimization efforts.

### **23.1.2 Performance Metrics**

When optimizing Go code, several key metrics should be considered:

1. **Execution Time**: How long a function or operation takes to complete.
2. **Memory Usage**: The amount of memory allocated and retained.
3. **Allocation Count**: The number of heap allocations performed.
4. **CPU Utilization**: How efficiently CPU resources are used.
5. **Latency**: Response time for operations, especially important in network services.
6. **Throughput**: The number of operations that can be performed in a given timeframe.

Different applications prioritize different metrics. A command-line tool might focus on execution time, while a web server might prioritize latency and throughput.

### **23.1.3 Performance Optimization Principles**

Before diving into specific optimization techniques, it's important to establish sound principles:

1. **Measure First**: Never optimize without measuring. Identify actual bottlenecks rather than assumed ones.
2. **Establish Baselines**: Create benchmarks to measure improvements against.
3. **Focus on Hot Spots**: Optimize the parts of your code that are executed most frequently or consume the most resources.
4. **Test Incremental Changes**: Make one change at a time and measure its impact.
5. **Preserve Readability**: Performance improvements that drastically reduce code readability are rarely worth it.
6. **Consider the Big Picture**: Sometimes architectural changes deliver better results than micro-optimizations.

Donald Knuth's famous quote applies here: "Premature optimization is the root of all evil." Optimize when you have evidence that optimization is needed.

## **23.2 Measuring Performance**

### **23.2.1 Benchmarking with Go's Testing Package**

Go's standard library includes excellent benchmarking tools in the `testing` package:

```go
package main

import (
    "testing"
)

// Function we want to benchmark
func Fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return Fibonacci(n-1) + Fibonacci(n-2)
}

// Benchmark function
func BenchmarkFibonacci(b *testing.B) {
    // Run the Fibonacci function b.N times
    for i := 0; i < b.N; i++ {
        Fibonacci(10)
    }
}
```

Run the benchmark with:

```
go test -bench=. -benchmem
```

This will output something like:

```
BenchmarkFibonacci-8     1000000       1234 ns/op       0 B/op       0 allocs/op
```

Breaking down the output:

- `BenchmarkFibonacci-8`: The name of the benchmark function and the number of CPUs used
- `1000000`: Number of iterations executed
- `1234 ns/op`: Average time per operation in nanoseconds
- `0 B/op`: Average memory allocated per operation
- `0 allocs/op`: Average number of allocations per operation

### **23.2.2 Advanced Benchmarking Techniques**

For more sophisticated benchmarking, Go provides additional tools:

```go
func BenchmarkComplexOperation(b *testing.B) {
    // Setup code (not measured)
    data := prepareTestData()

    // Reset the timer to exclude setup time
    b.ResetTimer()

    // Run the benchmark
    for i := 0; i < b.N; i++ {
        result := ProcessData(data)
        // Prevent compiler optimizations from skipping the work
        if result == nil {
            b.Fatal("unexpected nil result")
        }
    }

    // Optional: Stop the timer before cleanup
    b.StopTimer()

    // Cleanup code (not measured)
    cleanupTestData(data)
}
```

Key techniques:

- `b.ResetTimer()`: Excludes setup time from the benchmark
- `b.StopTimer()` and `b.StartTimer()`: Exclude specific operations from timing
- `b.SetBytes(n)`: Reports throughput in bytes/second
- `b.RunParallel()`: Benchmarks parallel performance
- `b.N`: Go automatically determines the appropriate number of iterations

### **23.2.3 Profiling Go Applications**

Benchmarking tells you how fast your code runs, but profiling tells you why it runs at that speed. Go offers several profiling tools:

1. **CPU Profiling**: Identifies which functions consume the most CPU time

```go
import "runtime/pprof"

// Start CPU profiling
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Your program runs here...
```

2. **Memory Profiling**: Shows memory allocation patterns

```go
import "runtime/pprof"

// Capture memory profile
f, _ := os.Create("mem.prof")
defer f.Close()
pprof.WriteHeapProfile(f)
```

3. **Block Profiling**: Identifies goroutine blocking points

```go
import "runtime"

// Enable block profiling
runtime.SetBlockProfileRate(1)
```

For benchmarks, you can enable profiling with flags:

```
go test -bench=. -cpuprofile=cpu.prof -memprofile=mem.prof
```

### **23.2.4 Analyzing Profiles with pprof**

The `pprof` tool analyzes profile data and presents it in various formats:

```
go tool pprof cpu.prof
```

Inside the pprof interface, useful commands include:

- `top`: Shows the top consumers of resources
- `list functionName`: Shows source code with performance data
- `web`: Generates a graph visualization (requires Graphviz)
- `traces`: Shows execution traces of function calls

For a web interface:

```
go tool pprof -http=:8080 cpu.prof
```

This provides an interactive web UI with graphs, flame graphs, and more.

### **23.2.5 Continuous Performance Testing**

For critical applications, incorporate performance testing into your CI/CD pipeline:

1. Define performance budgets for key operations
2. Run benchmarks as part of your test suite
3. Compare results against previous runs to detect regressions
4. Alert the team when performance degrades beyond thresholds

Libraries like `benchstat` help compare benchmark results:

```
go get golang.org/x/perf/cmd/benchstat
benchstat old.txt new.txt
```

This approach helps prevent performance regressions from reaching production.

## **23.3 Memory Management Optimization**

Memory management has a significant impact on Go performance. Optimizing memory usage improves execution speed and reduces garbage collection overhead.

### **23.3.1 Understanding Go's Memory Model**

Go uses a garbage collector (GC) to automatically manage memory. Understanding how it works helps write efficient code:

1. **Stack vs. Heap**: Go allocates memory on either the stack or heap

   - Stack allocations are faster and automatically cleaned up when a function returns
   - Heap allocations are managed by the garbage collector

2. **Escape Analysis**: The compiler determines whether a variable escapes to the heap

   - Variables that don't escape are allocated on the stack
   - Variables that escape (e.g., returned from a function) go to the heap

3. **Garbage Collection**: Go uses a concurrent, tri-color mark-and-sweep garbage collector
   - The GC periodically scans memory to identify and free unused objects
   - GC pauses are typically short but can affect latency-sensitive applications

To see escape analysis in action:

```
go build -gcflags="-m" your_program.go
```

### **23.3.2 Reducing Allocations**

Minimizing heap allocations improves performance by reducing GC pressure:

1. **Preallocate Slices and Maps**: Specify capacity when creating slices and maps

```go
// Poor: Grows slice multiple times, causing allocations
data := []int{}
for i := 0; i < 1000; i++ {
    data = append(data, i)
}

// Better: Preallocate with capacity
data := make([]int, 0, 1000)
for i := 0; i < 1000; i++ {
    data = append(data, i)
}
```

2. **Reuse Objects**: Pool or reuse objects instead of creating new ones

```go
// Using sync.Pool to reuse objects
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func processData(data []byte) string {
    // Get a buffer from the pool
    buf := bufferPool.Get().(*bytes.Buffer)
    // Ensure buffer is empty and return it to the pool when done
    buf.Reset()
    defer bufferPool.Put(buf)

    // Use the buffer
    buf.Write(data)
    return buf.String()
}
```

3. **Avoid String Concatenation**: Use `strings.Builder` instead of `+` for multiple concatenations

```go
// Poor: Creates many intermediate strings
s := ""
for i := 0; i < 1000; i++ {
    s += strconv.Itoa(i)
}

// Better: Uses a single growing buffer
var builder strings.Builder
for i := 0; i < 1000; i++ {
    builder.WriteString(strconv.Itoa(i))
}
s := builder.String()
```

4. **Use Value Types**: Pass small structs by value rather than pointer when possible

```go
// Small struct, can be passed by value
type Point struct {
    X, Y int
}

func distanceFromOrigin(p Point) float64 {
    return math.Sqrt(float64(p.X*p.X + p.Y*p.Y))
}
```

### **23.3.3 Memory Profiling and Analysis**

To identify memory usage patterns:

1. Run with memory profiling enabled:

```
go test -bench=. -benchmem -memprofile=mem.prof
```

2. Analyze with pprof:

```
go tool pprof -alloc_objects mem.prof  # Shows allocation count
go tool pprof -alloc_space mem.prof    # Shows allocation size
```

3. Look for functions with high allocation rates:

```
(pprof) top10 -cum
```

Common memory issues to watch for:

- Functions that allocate in hot paths
- Hidden allocations (e.g., boxing/unboxing, interface conversions)
- Unnecessary copies of large data structures

### **23.3.4 Garbage Collection Tuning**

Go's garbage collector is designed to work well with default settings, but you can tune it for specific workloads:

1. **GOGC Environment Variable**: Controls GC frequency
   - Default is GOGC=100 (run GC when heap size grows by 100%)
   - Higher values reduce GC frequency but increase memory usage
   - Lower values increase GC frequency but reduce memory usage

```
GOGC=200 ./myprogram  # Less frequent GC
```

2. **Manual GC Triggers**: Force collection at strategic points

```go
import "runtime"

// After processing a large batch, force GC
processBatch()
runtime.GC()
```

3. **Memory Limit**: Set a soft memory limit (Go 1.19+)

```
GOMEMLIMIT=4GiB ./myprogram
```

For most applications, the default settings work well. Only tune GC when profiling shows it's a bottleneck.

## **23.4 CPU Optimization Techniques**

After memory, CPU usage is typically the next performance bottleneck. Here are strategies to optimize CPU-bound code.

### **23.4.1 Algorithmic Improvements**

The most significant performance gains often come from algorithmic improvements:

1. **Choose the Right Algorithm**: Using an O(n log n) algorithm instead of an O(n²) algorithm can make a much bigger difference than micro-optimizations.

```go
// Inefficient: O(n²) search
func contains(items []string, target string) bool {
    for _, item := range items {
        if item == target {
            return true
        }
    }
    return false
}

// More efficient: O(1) lookup using map
func createSet(items []string) map[string]struct{} {
    set := make(map[string]struct{}, len(items))
    for _, item := range items {
        set[item] = struct{}{}
    }
    return set
}

func containsInSet(set map[string]struct{}, target string) bool {
    _, exists := set[target]
    return exists
}
```

2. **Memoization**: Cache expensive function results

```go
// Without memoization
func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}

// With memoization
func fibonacciMemo() func(int) int {
    cache := make(map[int]int)
    var fib func(int) int

    fib = func(n int) int {
        if n <= 1 {
            return n
        }

        if result, found := cache[n]; found {
            return result
        }

        result := fib(n-1) + fib(n-2)
        cache[n] = result
        return result
    }

    return fib
}

// Usage
memoizedFib := fibonacciMemo()
result := memoizedFib(40)  // Much faster for large n
```

3. **Lazy Evaluation**: Defer work until it's actually needed

```go
// Lazy loading of expensive resources
type ExpensiveResource struct {
    data []byte
    once sync.Once
    path string
}

func (r *ExpensiveResource) Data() []byte {
    r.once.Do(func() {
        // Load data only when first accessed
        r.data, _ = ioutil.ReadFile(r.path)
    })
    return r.data
}
```

### **23.4.2 Loop Optimization**

Loops are often performance hotspots. Here are techniques to optimize them:

1. **Loop Hoisting**: Move invariant computations outside the loop

```go
// Before optimization
for i := 0; i < len(items); i++ {
    // len(items) is recomputed on every iteration
    result += items[i] * multiplier()
}

// After optimization
itemsLen := len(items)
mult := multiplier()
for i := 0; i < itemsLen; i++ {
    result += items[i] * mult
}
```

2. **Loop Unrolling**: Process multiple elements per iteration

```go
// Standard loop
sum := 0
for i := 0; i < len(items); i++ {
    sum += items[i]
}

// Unrolled loop (process 4 elements at once)
sum := 0
i := 0
for ; i+3 < len(items); i += 4 {
    sum += items[i] + items[i+1] + items[i+2] + items[i+3]
}
// Handle remaining elements
for ; i < len(items); i++ {
    sum += items[i]
}
```

3. **Range Loop Optimization**: Be aware of copy costs in range loops

```go
type LargeStruct struct {
    Data [1024]byte
    ID   int
}

items := []LargeStruct{...}

// Expensive: Copies each LargeStruct during iteration
for _, item := range items {
    process(item.ID)
}

// Cheaper: Uses index to avoid copying
for i := range items {
    process(items[i].ID)
}

// Alternative: Use pointers if modifying items
itemPtrs := []*LargeStruct{...}
for _, item := range itemPtrs {
    process(item.ID)
}
```

### **23.4.3 Compiler Optimizations**

Understand how Go's compiler optimizes code:

1. **Inlining**: The compiler can inline small functions to eliminate function call overhead. View inlining decisions:

```
go build -gcflags="-m" your_program.go
```

2. **Bounds Check Elimination**: The compiler tries to eliminate redundant slice bounds checks:

```go
// May perform redundant bounds checks
for i := 0; i < len(s); i++ {
    if s[i] == x {
        // ...
    }
    if s[i] == y {  // Second bounds check
        // ...
    }
}

// Better for bounds check elimination
for i := 0; i < len(s); i++ {
    item := s[i]  // Single bounds check
    if item == x {
        // ...
    }
    if item == y {
        // ...
    }
}
```

3. **Dead Code Elimination**: Unused code is removed. Be careful with benchmarks:

```go
func BenchmarkCompute(b *testing.B) {
    var result int
    for i := 0; i < b.N; i++ {
        result = compute(i)
    }
    // Prevent dead code elimination
    if result == 0 {
        b.Fatalf("unexpected zero result")
    }
}
```

### **23.4.4 Data Structure Efficiency**

Choose appropriate data structures for your workload:

1. **Slice vs. Array**: Use arrays for fixed-size collections, slices for dynamic ones

```go
// Fixed size, allocates on stack if small enough
var buffer [64]byte

// Dynamic size, always allocates on heap
slice := make([]byte, 64)
```

2. **Map vs. Slice**: For lookup operations, consider the size and access pattern

```go
// Map: O(1) lookup but with overhead
idToUser := make(map[int]User)

// Slice: O(n) lookup but faster for small n with sequential IDs
users := make([]User, maxID+1)
// Access: users[id]
```

3. **Struct Layout**: Group related fields and consider alignment

```go
// Poor: Wastes space due to padding
type Inefficient struct {
    a byte     // 1 byte + 7 padding
    b int64    // 8 bytes
    c byte     // 1 byte + 7 padding
    d int64    // 8 bytes
}  // Total: 32 bytes

// Better: Fields arranged by size
type Efficient struct {
    b int64    // 8 bytes
    d int64    // 8 bytes
    a byte     // 1 byte
    c byte     // 1 byte + 6 padding
}  // Total: 24 bytes
```

4. **Type Selection**: Use the most efficient type for your data

```go
// Using smaller types when appropriate
type Stats struct {
    Count   uint32  // Instead of int when values are always positive and < 2^32
    Flags   uint8   // Instead of bool for multiple flags
    IsValid bool    // Single boolean
}
```

## **23.5 Concurrency Optimization**

Go's concurrency model is one of its strongest features, but using it effectively requires understanding its performance characteristics and potential pitfalls.

### **23.5.1 Goroutine Management**

Goroutines are lightweight, but they're not free:

1. **Appropriate Number of Goroutines**: Match goroutine count to the workload

```go
// Poor: Creates a goroutine for every item, potentially millions
func processItems(items []Item) {
    var wg sync.WaitGroup
    for _, item := range items {
        wg.Add(1)
        go func(item Item) {
            defer wg.Done()
            process(item)
        }(item)
    }
    wg.Wait()
}

// Better: Use a worker pool to limit concurrency
func processItemsWithPool(items []Item) {
    numWorkers := runtime.GOMAXPROCS(0) // Use CPU count
    itemCh := make(chan Item)
    var wg sync.WaitGroup

    // Start workers
    wg.Add(numWorkers)
    for i := 0; i < numWorkers; i++ {
        go func() {
            defer wg.Done()
            for item := range itemCh {
                process(item)
            }
        }()
    }

    // Send work to workers
    for _, item := range items {
        itemCh <- item
    }
    close(itemCh)

    // Wait for completion
    wg.Wait()
}
```

2. **Goroutine Reuse**: Reuse goroutines for repeated operations

```go
// Worker pool pattern
type WorkerPool struct {
    tasks chan func()
    wg    sync.WaitGroup
}

func NewWorkerPool(size int) *WorkerPool {
    pool := &WorkerPool{
        tasks: make(chan func()),
    }

    pool.wg.Add(size)
    for i := 0; i < size; i++ {
        go func() {
            defer pool.wg.Done()
            for task := range pool.tasks {
                task()
            }
        }()
    }

    return pool
}

func (p *WorkerPool) Submit(task func()) {
    p.tasks <- task
}

func (p *WorkerPool) Close() {
    close(p.tasks)
}

func (p *WorkerPool) Wait() {
    p.wg.Wait()
}
```

3. **Monitoring Goroutine Count**: Track and analyze goroutine usage

```go
import "runtime"

// Periodically log goroutine count
func monitorGoroutines(ctx context.Context, interval time.Duration) {
    ticker := time.NewTicker(interval)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            log.Printf("Goroutines: %d", runtime.NumGoroutine())
        case <-ctx.Done():
            return
        }
    }
}
```

### **23.5.2 Channel Optimization**

Channels are the primary communication mechanism for goroutines:

1. **Buffered vs. Unbuffered Channels**: Choose the right type for your use case

```go
// Unbuffered: Synchronous communication (sender blocks until receiver takes the value)
unbufferedCh := make(chan int)

// Buffered: Asynchronous up to the buffer size
bufferedCh := make(chan int, 100)
```

Guidelines:

- Use unbuffered channels when you want synchronization between goroutines
- Use buffered channels when the sender and receiver work at different rates
- Size buffers based on expected load patterns, not arbitrarily large values

2. **Channel Closing**: Close channels correctly to avoid panics

```go
// Producer responsible for closing
func producer(ch chan<- int) {
    defer close(ch) // Signal no more data
    for i := 0; i < 10; i++ {
        ch <- i
    }
}

// Consumer checks for closed channel
func consumer(ch <-chan int) {
    for {
        value, ok := <-ch
        if !ok {
            // Channel closed
            return
        }
        // Process value
        fmt.Println(value)
    }

    // Alternative: range automatically handles closed channels
    for value := range ch {
        fmt.Println(value)
    }
}
```

3. **Channel Direction**: Use directional channel types for clarity and safety

```go
// Send-only channel
func send(ch chan<- int) {
    ch <- 42
    // Compile error: cannot receive from send-only channel
    // x := <-ch
}

// Receive-only channel
func receive(ch <-chan int) {
    x := <-ch
    // Compile error: cannot send to receive-only channel
    // ch <- 42
}
```

### **23.5.3 Synchronization Primitives**

Choose the right synchronization mechanism for your needs:

1. **Mutex vs. RWMutex**: Use appropriate locks for access patterns

```go
// Mutex: For exclusive access
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

// RWMutex: When reads are more common than writes
type Cache struct {
    mu    sync.RWMutex
    items map[string]string
}

func (c *Cache) Get(key string) (string, bool) {
    c.mu.RLock() // Multiple readers can access simultaneously
    defer c.mu.RUnlock()
    val, ok := c.items[key]
    return val, ok
}

func (c *Cache) Set(key, value string) {
    c.mu.Lock() // Exclusive access for writers
    defer c.mu.Unlock()
    c.items[key] = value
}
```

2. **atomic vs. Mutex**: Use atomic operations for simple counters and flags

```go
import "sync/atomic"

// Using atomic operations (faster for simple cases)
type AtomicCounter struct {
    count int64
}

func (c *AtomicCounter) Increment() {
    atomic.AddInt64(&c.count, 1)
}

func (c *AtomicCounter) Value() int64 {
    return atomic.LoadInt64(&c.count)
}
```

3. **sync.Once**: For one-time initialization

```go
type Config struct {
    once     sync.Once
    settings map[string]string
}

func (c *Config) Load() map[string]string {
    c.once.Do(func() {
        c.settings = loadConfigFromDisk()
    })
    return c.settings
}
```

### **23.5.4 Parallelism Patterns**

Patterns for effective parallelization:

1. **Fan-Out, Fan-In**: Process work in parallel then collect results

```go
func fanOutFanIn(items []Item) []Result {
    numWorkers := runtime.GOMAXPROCS(0)

    // Fan-out: Distribute work across multiple goroutines
    jobs := make(chan Item)
    results := make(chan Result)

    // Start workers
    var wg sync.WaitGroup
    wg.Add(numWorkers)
    for i := 0; i < numWorkers; i++ {
        go func() {
            defer wg.Done()
            for item := range jobs {
                results <- processItem(item)
            }
        }()
    }

    // Close results when all workers are done
    go func() {
        wg.Wait()
        close(results)
    }()

    // Send all jobs
    go func() {
        for _, item := range items {
            jobs <- item
        }
        close(jobs)
    }()

    // Fan-in: Collect all results
    var collected []Result
    for result := range results {
        collected = append(collected, result)
    }

    return collected
}
```

2. **Work Stealing**: Dynamically balance load between workers

```go
type Task func()

type WorkStealingPool struct {
    queues    []chan Task
    stealChan chan int
    quit      chan struct{}
    wg        sync.WaitGroup
}

func NewWorkStealingPool(numWorkers int) *WorkStealingPool {
    pool := &WorkStealingPool{
        queues:    make([]chan Task, numWorkers),
        stealChan: make(chan int, numWorkers),
        quit:      make(chan struct{}),
    }

    // Create task queues for each worker
    for i := 0; i < numWorkers; i++ {
        pool.queues[i] = make(chan Task, 128)
    }

    // Start workers
    pool.wg.Add(numWorkers)
    for i := 0; i < numWorkers; i++ {
        go pool.worker(i)
    }

    return pool
}

func (p *WorkStealingPool) worker(id int) {
    defer p.wg.Done()

    for {
        select {
        case task := <-p.queues[id]:
            // Process tasks from own queue
            task()
        case <-p.stealChan:
            // Try to steal work from other queues
            for i := 0; i < len(p.queues); i++ {
                if i == id {
                    continue
                }

                select {
                case task := <-p.queues[i]:
                    task()
                default:
                    // Queue empty, try another
                }
            }
        case <-p.quit:
            return
        default:
            // No work, announce willingness to steal
            select {
            case p.stealChan <- id:
            default:
                // Channel full, brief pause
                runtime.Gosched()
            }
        }
    }
}

func (p *WorkStealingPool) Submit(workerId int, task Task) {
    p.queues[workerId] <- task
}

func (p *WorkStealingPool) Close() {
    close(p.quit)
    p.wg.Wait()
}
```

3. **Pipeline Pattern**: Break work into sequential stages

```go
func generateNumbers(done <-chan struct{}) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 0; i < 100; i++ {
            select {
            case out <- i:
            case <-done:
                return
            }
        }
    }()
    return out
}

func square(done <-chan struct{}, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-done:
                return
            }
        }
    }()
    return out
}

func filter(done <-chan struct{}, in <-chan int, filterFn func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if filterFn(n) {
                select {
                case out <- n:
                case <-done:
                    return
                }
            }
        }
    }()
    return out
}

func runPipeline() {
    done := make(chan struct{})
    defer close(done)

    // Build the pipeline
    numbers := generateNumbers(done)
    squares := square(done, numbers)
    evenSquares := filter(done, squares, func(n int) bool {
        return n%2 == 0
    })

    // Consume the final results
    for n := range evenSquares {
        fmt.Println(n)
        if n > 1000 {
            break
        }
    }
}
```

## **23.6 Case Studies and Exercises**

### **23.6.1 Case Study: Optimizing a REST API**

Consider a Go REST API that serves product information:

**Original implementation**:

```go
type Product struct {
    ID          int
    Name        string
    Description string
    Price       float64
    Categories  []string
    // Many more fields...
}

var productCache = make(map[int]Product)
var mutex sync.Mutex

func getProduct(id int) (Product, error) {
    mutex.Lock()
    defer mutex.Unlock()

    if product, found := productCache[id]; found {
        return product, nil
    }

    product, err := fetchProductFromDB(id)
    if err != nil {
        return Product{}, err
    }

    productCache[id] = product
    return product, nil
}

func handleGetProduct(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }

    product, err := getProduct(id)
    if err != nil {
        http.Error(w, "Product not found", http.StatusNotFound)
        return
    }

    data, err := json.Marshal(product)
    if err != nil {
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }

    w.Header().Set("Content-Type", "application/json")
    w.Write(data)
}
```

**Optimized implementation**:

```go
type Product struct {
    ID          int
    Name        string
    Description string
    Price       float64
    Categories  []string
    // Many more fields...
}

// ProductResponse separates API response from internal representation
type ProductResponse struct {
    ID          int      `json:"id"`
    Name        string   `json:"name"`
    Description string   `json:"description,omitempty"`
    Price       float64  `json:"price"`
    Categories  []string `json:"categories,omitempty"`
}

// Use sync.Map for concurrent access without locking
var productCache sync.Map

func toResponse(p Product) ProductResponse {
    return ProductResponse{
        ID:          p.ID,
        Name:        p.Name,
        Description: p.Description,
        Price:       p.Price,
        Categories:  p.Categories,
    }
}

func getProduct(id int) (Product, error) {
    // Check cache first
    if val, found := productCache.Load(id); found {
        return val.(Product), nil
    }

    // Fetch from database if not in cache
    product, err := fetchProductFromDB(id)
    if err != nil {
        return Product{}, err
    }

    // Store in cache
    productCache.Store(id, product)
    return product, nil
}

// Dedicated worker pool for JSON encoding
var jsonEncoderPool = sync.Pool{
    New: func() interface{} {
        return json.NewEncoder(new(bytes.Buffer))
    },
}

func handleGetProduct(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        http.Error(w, "Invalid ID", http.StatusBadRequest)
        return
    }

    product, err := getProduct(id)
    if err != nil {
        http.Error(w, "Product not found", http.StatusNotFound)
        return
    }

    // Convert to response format
    response := toResponse(product)

    // Get encoder from pool
    buf := new(bytes.Buffer)
    encoder := jsonEncoderPool.Get().(*json.Encoder)
    encoder.(*json.Encoder).SetEscapeHTML(false)

    // Reset buffer and set it as output
    buf.Reset()
    encoder.(*json.Encoder).SetWriter(buf)

    // Encode response
    if err := encoder.Encode(response); err != nil {
        jsonEncoderPool.Put(encoder)
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }

    // Return encoder to pool
    jsonEncoderPool.Put(encoder)

    // Write response
    w.Header().Set("Content-Type", "application/json")
    w.Header().Set("Cache-Control", "max-age=60")
    w.Write(buf.Bytes())
}
```

Key improvements:

1. Used `sync.Map` for concurrent access without locking
2. Created a separate response struct to control JSON output
3. Implemented object pooling for JSON encoders
4. Added cache headers
5. Reduced allocations and GC pressure

### **23.6.2 Exercise 1: Profile and Optimize**

Take the following code and identify performance bottlenecks, then optimize it:

```go
func FindPrimes(max int) []int {
    var primes []int
    for i := 2; i <= max; i++ {
        isPrime := true
        for j := 2; j < i; j++ {
            if i%j == 0 {
                isPrime = false
                break
            }
        }
        if isPrime {
            primes = append(primes, i)
        }
    }
    return primes
}

func BenchmarkFindPrimes(b *testing.B) {
    for i := 0; i < b.N; i++ {
        FindPrimes(10000)
    }
}
```

**Hint**: Consider algorithm improvements (Sieve of Eratosthenes), memory optimizations, and loop optimizations.

### **23.6.3 Exercise 2: Memory Optimization**

Optimize the following code to reduce memory allocations and improve performance:

```go
func ProcessLogs(logs []string) map[string]int {
    counts := make(map[string]int)

    for _, log := range logs {
        parts := strings.Split(log, " ")
        if len(parts) >= 3 {
            timestamp := parts[0]
            level := parts[1]
            message := strings.Join(parts[2:], " ")

            key := timestamp + "-" + level
            counts[key] += 1

            if strings.Contains(message, "error") {
                counts["errors"] += 1
            }
        }
    }

    return counts
}
```

**Hint**: Look for unnecessary allocations, string concatenations, and ways to reuse memory.

### **23.6.4 Exercise 3: Concurrency Optimization**

Improve the performance of this file processing function using appropriate concurrency patterns:

```go
func ProcessFiles(filePaths []string) (map[string]int, error) {
    wordCounts := make(map[string]int)

    for _, path := range filePaths {
        data, err := ioutil.ReadFile(path)
        if err != nil {
            return nil, err
        }

        content := string(data)
        words := strings.Fields(content)

        for _, word := range words {
            word = strings.ToLower(strings.Trim(word, ".,!?()[]{}:;\"'"))
            if word != "" {
                wordCounts[word]++
            }
        }
    }

    return wordCounts, nil
}
```

**Hint**: Consider using worker pools, channel pipelines, and appropriate synchronization primitives.

## **23.7 Summary**

In this chapter, we explored comprehensive performance optimization techniques for Go applications:

- **Understanding Go's performance characteristics**: We examined Go's performance philosophy, trade-offs, and metrics.

- **Measurement tools**: We learned how to use Go's benchmarking, profiling, and analysis tools to identify bottlenecks.

- **Memory optimization**: We explored techniques to reduce allocations, manage garbage collection, and improve memory usage.

- **CPU optimization**: We examined algorithmic improvements, loop optimizations, and compiler optimizations.

- **Concurrency optimization**: We looked at patterns for effective goroutine management, channel usage, and synchronization.

The key lesson from this chapter is that effective optimization requires measurement, focuses on high-impact areas, and considers both algorithmic improvements and Go-specific optimizations.

Remember that clear, maintainable code that correctly solves the problem should be your first priority. Optimize only when measurements indicate a need and when the benefits outweigh the costs in terms of code complexity.

**Next Up**: In Chapter 24, we'll explore profiling and debugging techniques to further improve your Go applications.
