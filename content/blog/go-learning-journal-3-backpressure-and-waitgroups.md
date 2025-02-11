---
title: "Go Learning Journal 3: Backpressure pattern and WaitGroup"
date: 2025-02-11
tags: ["Go", "Notes", "Concurrency", "Backpressure", "Waitgroup"]
draft: false
---

These are raw learning notes from my journey with Go, documenting key concepts and examples for future reference. I reason with LLMs to understand some concepts better and sometimes copy paste the responses of that in my notes. These are not original thoughts.


### What Is Backpressure?

**Backpressure** is a technique used in concurrent and distributed systems to control the flow of data and prevent components from being overwhelmed by too much work. When a component (like a server or a worker) can't keep up with incoming requests, backpressure mechanisms help to:

- **Limit Incoming Workload**: Set a cap on how much work can be processed concurrently.
- **Prevent Resource Exhaustion**: Avoid overloading the system, which could lead to crashes or degraded performance.
- **Maintain System Stability**: Ensure the system remains responsive and performs predictably under load.

### Real-World Analogy

Think of a amusement park ride with limited seating:

- **Ride Capacity**: The ride can only hold 10 people at a time.
- **Queue Management**:
    - If less than 10 people are waiting, they can get on the ride immediately.
    - If 10 people are already on the ride, additional guests are informed to wait or come back later.
- **Safety and Efficiency**:
    - Ensures the ride operates safely within its capacity.
    - Prevents overcrowding and potential accidents.

## Practical Example: Limiting Concurrent HTTP Requests with `RequestLimiter`

Let's consider a real-world scenario where we have an HTTP server that needs to process incoming requests. To maintain optimal performance and prevent the server from being overwhelmed, we want to limit the number of concurrent requests it can handle.

We'll implement backpressure by creating a `RequestLimiter` type that enforces a maximum number of concurrent operations.

### Code Example

```go
package main

import (
    "errors"
    "fmt"
    "net/http"
    "time"
)

// RequestLimiter limits the number of concurrent requests.
type RequestLimiter struct {
    semaphore chan struct{}
}

// NewRequestLimiter creates a new RequestLimiter with the specified limit.
func NewRequestLimiter(limit int) *RequestLimiter {
    return &RequestLimiter{
        semaphore: make(chan struct{}, limit),
    }
}

// Execute runs the given function if under the limit; otherwise returns an error.
func (rl *RequestLimiter) Execute(f func()) error {
    select {
    case rl.semaphore <- struct{}{}:
        defer func() { <-rl.semaphore }()
        f()
        return nil
    default:
        return errors.New("too many concurrent requests")
    }
}

// Simulate a resource-intensive operation.
func doHeavyWork() {
    // Simulate heavy work by sleeping.
    time.Sleep(2 * time.Second)
}

func main() {
    // Limit to 5 concurrent requests.
    rl := NewRequestLimiter(5)

    http.HandleFunc("/process", func(w http.ResponseWriter, r *http.Request) {
        err := rl.Execute(func() {
            doHeavyWork()
            fmt.Fprintln(w, "Work completed")
        })

        if err != nil {
            // Return HTTP 429 Too Many Requests
            w.WriteHeader(http.StatusTooManyRequests)
            fmt.Fprintln(w, "Server is busy, please try again later.")
        }
    })

    fmt.Println("Server is listening on :8080")
    http.ListenAndServe(":8080", nil)
}
```

---

### How Backpressure Works in This Example

- **Controlling Concurrency**:
    - The `RequestLimiter` ensures that no more than 5 requests are processed simultaneously.
    - The semaphore channel acts as a counter; only 5 tokens can be held at once.

- **Handling Excess Requests**:
    - If more than 5 requests are received concurrently, additional requests are immediately responded to with an error message.
    - This prevents the server from being overwhelmed and maintains performance.

- **User Experience**:
    - Clients receive immediate feedback if the server is busy.
    - They can implement retry logic if needed.

- **Preventing Resource Exhaustion**:
    - Limits CPU, memory, and other resources usage by capping concurrent processing.
    - Helps maintain server stability under high load.

---

## Understanding WaitGroups in Go

In concurrent programming with Go, you often use goroutines to run functions concurrently. Sometimes, you need your main program (or another goroutine) to wait until several other goroutines have finished executing before proceeding. This is where a `WaitGroup` comes in.

### What Is a WaitGroup?

A `WaitGroup` is a synchronization construct provided by the `sync` package in Go's standard library. It allows you to:

- **Add**: Specify the number of goroutines to wait for.
- **Done**: Indicate that a goroutine has finished its work.
- **Wait**: Block (pause) the execution until all added goroutines have called `Done`.

Think of a `WaitGroup` as a counter that keeps track of how many goroutines are running, and lets you wait until all of them have finished.

## Why Use WaitGroups?

- **Synchronization**: Ensure that the main program waits for all goroutines to finish before exiting or proceeding.
- **Resource Management**: Avoid issues where the program exits before goroutines complete their tasks.

---

- **Always pass the WaitGroup pointer (`*sync.WaitGroup`) to goroutines**.
- **Using `defer wg.Done()` inside the goroutine function**.
