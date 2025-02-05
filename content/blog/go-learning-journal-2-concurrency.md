---
title: "Go Learning Journal #2: Concurrency"
date: 2025-02-05
tags: ["Go", "Notes", "Concurrency"]
draft: false
---

These are raw learning notes from my journey with Go, documenting key concepts and examples for future reference. I reason with LLMs to understand some concepts better and sometimes copy paste the responses of that in my notes. These are not original thoughts.

## Concurrency in Go

Concurrency is the computer science term for breaking up a single process into independent components and specifying how these components safely share data.

- concurrency is not parallelism.
- more concurrency does not mean more speed

**process**: A process is an instance of a program that’s being run by a computer’s operating system. The operating system associates some resources, such as memory, with the process and makes sure that other processes can’t access them. A process is composed of one or more **threads**.

**thread**: A thread is a unit of execution that is given some time to run by the operating system. Threads within a process share access to resources.

- A CPU can execute instructions from one or more threads at the same time, depending on the number of cores.
- One of the jobs of an operating system is to schedule threads on the CPU to make sure that every process (and every thread within a process) gets a chance to run.

## Understanding This Visually
```
KITCHEN (Operating System)
┌──────────────────────────────────────┐
│                                      │
│   COOKING PASTA (Process 1)          │
│   ├── Boiling water (Thread 1)       │
│   ├── Preparing sauce (Thread 2)     │
│   └── Grating cheese (Thread 3)      │
│                                      │
│   MAKING SALAD (Process 2)           │
│   ├── Chopping veggies (Thread 1)    │
│   └── Making dressing (Thread 2)      │
│                                      │
└──────────────────────────────────────┘
```

Think of it this way:

1. **Kitchen (Operating System)**
   - The kitchen is like your operating system
   - It has resources (stove, counter space, utensils)
   - It manages who can use what and when

2. **Recipes (Processes)**
   - Each recipe (like cooking pasta or making salad) is a separate process
   - Each has its own ingredients and tools (resources)
   - They don't interfere with each other's ingredients

3. **Tasks (Threads)**
   - Within each recipe, you have multiple tasks that can happen simultaneously
   - These tasks share the recipe's resources
   - Like while boiling pasta, you can prepare sauce at the same time

In computer terms:
```
COMPUTER (Operating System)
┌──────────────────────────────────────┐
│                                      │
│   CHROME BROWSER (Process 1)         │
│   ├── UI Thread                      │
│   ├── Networking Thread              │
│   └── Rendering Thread               │
│                                      │
│   WORD PROCESSOR (Process 2)         │
│   ├── Editing Thread                 │
│   └── Auto-save Thread               │
│                                      │
└──────────────────────────────────────┘
```

Key Points:
- Processes are isolated from each other (Chrome can't mess up Word's data)
- Threads within a process share resources (all Chrome threads can access Chrome's memory)
- The OS (like a kitchen manager) decides which threads get to use the CPU (stove) when

---

Goroutine is a lightweight thread, managed by the Go runtime.

```
OS Scheduler:
- Manages OS threads
- Decides which thread runs on which CPU core
- Works at operating system level

Go Scheduler:
- Manages goroutines
- Decides which goroutine runs on which OS thread
- Works within Go program
```

- OS thread scheduling is "expensive" (takes more resources)
- Go's scheduler can make faster, more efficient decisions
- Go can manage thousands of goroutines with just a few OS threads

Go's scheduler is an extra layer that makes goroutine management more efficient than if we tried to create one OS thread per concurrent operation.

---

**Basic Goroutine Launch**
```go
// Regular function call
doSomething()

// Launch as goroutine
go doSomething()    // adds 'go' keyword
```

- Any function can be made concurrent with `go`
- It's good practice to keep business logic separate from concurrency logic

---

## Channels

- Goroutines communicate using channels.
- Channels are a built-in type created using the make function:

```go
ch := make(chan int)
```

- Channels are reference types.
- When you pass a channel to a function, you are really passing a pointer to the channel.
- The zero value for a channel is nil.

**Reading From Channel**
```go
val := <-ch // reads a value from ch and assigns it to val
```

**Writing To Channel**
```go
ch <- b // write the value in b to ch
```

- Each value written to a channel can be read only once. If multiple goroutines are reading from the same channel, a value written to the channel will be read by only one of them.

- Use **go ch <-chan int** (arrow before `chan`) to declare read-only channels.
```go
func readOnly(ch <-chan int) { val := <-ch }
```

- Use **go ch chan<- int** (arrow after `chan`) to declare write-only channels.
```go
func writeOnly(ch chan<- int) { ch <- 42 }
```

**By default, channels are unbuffered.**

## What is an Unbuffered Channel?
- Created using `make(chan int)` without a buffer size
- Requires both sender and receiver to be ready at the same time

### Key Behavior
- Writing to channel pauses until someone reads from it
- Reading from channel pauses until someone writes to it
- Need at least two goroutines for communication

### Example
```go
func main() {
    ch := make(chan int) // unbuffered channel

    // Sender goroutine
    go func() {
        fmt.Println("Trying to send...")
        ch <- 42                    // Blocks here until someone reads
        fmt.Println("Sent!")
    }()

    // Receiver (main goroutine)
    fmt.Println("Trying to receive...")
    value := <-ch                   // Blocks here until someone sends
    fmt.Println("Received:", value)
}
```

Output:
```
Trying to send...
Trying to receive...
Received: 42
Sent!
```

## Buffered Channels in Go

### What is a Buffered Channel?
- Created using `make(chan int, size)` with a buffer size
- Can hold multiple values before blocking
- Like a small queue with fixed capacity

### Key Behaviors
1. Writing:
   - Can write without blocking until buffer is full
   - Blocks when buffer is full until someone reads

2. Reading:
   - Can read until buffer is empty
   - Blocks when buffer is empty until someone writes

### Example
```go
func main() {
    ch := make(chan int, 3) // buffered channel with capacity 3

    // Can write 3 values without blocking
    ch <- 1
    ch <- 2
    ch <- 3
    // ch <- 4  // This would block as buffer is full

    fmt.Println(len(ch)) // Current buffer size (3)
    fmt.Println(cap(ch)) // Maximum buffer size (3)
}
```

### Useful Functions
- `len(ch)`: Returns current number of values in buffer
- `cap(ch)`: Returns maximum buffer capacity
- Buffer capacity is fixed and cannot be changed after creation

---

## Using for-range with Channels in Go

### Basic Syntax
```go
for value := range channel {
    // Process value
}
```

### Key Points
1. Reading Values:
   - Gets one value at a time from channel
   - Only one variable needed (unlike slice/map for-range)
   - Automatically handles the receive operation (`<-`)

2. Behavior:
   - If channel has value: continues execution
   - If channel empty: pauses until value available
   - If channel closed: loop ends
   - Can exit early with `break` or `return`

### Example
```go
func main() {
    ch := make(chan int, 3)

    // Sender
    go func() {
        ch <- 1
        ch <- 2
        ch <- 3
        close(ch)  // Important: close channel when done
    }()

    // Receiver using for-range
    for num := range ch {
        fmt.Println("Received:", num)
    }
}
```

Output:
```
Received: 1
Received: 2
Received: 3
```

## Note
- Common pattern for processing streams of data from channels
- Cleaner than manually checking with `value, ok := <-ch`

## Closing Channels in Go

### Key Behaviors
1. Writing to Closed Channel:
   - Will cause panic
   - Cannot close already closed channel

2. Reading from Closed Channel:
   - Always succeeds
   - Returns remaining buffered values if any
   - Returns zero value when empty

### Detecting Closed Channel
```go
// Comma-ok idiom
value, ok := <-ch
if ok {
    fmt.Println("Channel open, received:", value)
} else {
    fmt.Println("Channel closed, received zero value")
}
```

- The responsibility for closing a channel lies with the goroutine that writes to the channel.

---

## Select in Go

In Go, the `select` statement is a control structure that allows a goroutine to wait on multiple communication operations (channel sends or receives).

### Basic Syntax

```go
select {
case <-ch1:
    // Code to execute when ch1 receives a value
case val := <-ch2:
    // Code to execute when ch2 sends a value into val
case ch3 <- val:
    // Code to execute when val is sent to ch3
default:
    // Code to execute if none of the above cases are ready
}
```

- **Each `case` represents a channel operation**: a send or receive.
- **The `default` case** is executed if none of the channels are ready.

### How `select` Works

- **Blocking Behavior**: The `select` statement will block until at least one of the channel operations is ready to proceed.
- **Random Selection**: If multiple channels are ready, `select` chooses one at random to avoid starvation of any particular case.
- **Avoids Starvation**: Since selection is random among ready cases, no case is favored, ensuring fairness.

### The For-Select Loop

A common pattern is to use `select` inside a `for` loop, continuously waiting for channel operations.

#### Example For-Select Loop

```go
for {
    select {
    case <-done:
        return
    case v := <-ch:
        fmt.Println(v)
    }
}
```

- **Purpose**:
  - Continuously listen for messages on `ch`.
  - Exit the loop when a signal is received on the `done` channel.

### Best Practices

- **Avoid Default in For-Select Loops**: Unless you have a specific reason and handle it carefully, it's best to avoid using `default` inside a for-select loop.
- **Allow Blocking**: Let the `select` statement block when none of the channels are ready. This allows your goroutine to sleep until there's work to do, which is efficient.
- **Exiting the Loop**: Use a case that allows you to exit the loop gracefully (like receiving from a `done` channel).
