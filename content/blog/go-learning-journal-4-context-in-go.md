---
title: "Go Learning Journal 4: Context in Go"
date: 2025-03-17
tags: ["Go", "Notes"]
draft: false
---

These are raw learning notes from my journey with Go, documenting key concepts and examples for future reference. I reason with LLMs to understand some concepts better and sometimes copy paste the responses of that in my notes. These are not original thoughts.

## Context in Go

**Purpose of Context**:
- Used to handle request metadata in servers
- Manages two types of metadata:
  * Data needed to process requests (e.g., tracking IDs)
  * Data about when to stop processing (e.g., timeouts)

- A context is simply an instance that meets the Context interface defined in the context package.
- Context is like a container that carries request information
- It passes through your entire program
- Helps manage:
  - Request tracking
  - Timeouts
  - Cancellation
  - Request-scoped values
- Context is explicitly passed as first argument(ctx) to functions that need it.
```go
func logic(ctx context.Context, info string) (string, error)
```


**Analogy: Food Delivery Service**

Imagine you're running a food delivery service:

1. **Order (Request)**
- Customer places an order for pizza
- Needs to be delivered within 30 minutes
- Has a tracking number
- Has special instructions

2. **Delivery Person (Context)**
The delivery person carries:
- Tracking number
- Delivery deadline (30 min timer)
- Special instructions
- Ability to cancel delivery if customer isn't available

```go
// Without Context
func DeliverFood(trackingID string, deadline time.Time, instructions string) {
    // Manually pass everything
}

// With Context
func DeliverFood(ctx context.Context) {
    trackingID := ctx.Value("trackingID")
    // Deadline and cancellation built into context
    // All information travels together
}
```

**Using `context.WithValue`:**
- To pass values through a context, use `context.WithValue(parentCtx, key, value)`.
- This function returns a new context (a child context) that wraps the parent context and includes the key-value pair.
- Contexts are immutable; adding a value creates a new context.

**Retrieving Values from Context:**
- Use `ctx.Value(key)` to retrieve a value.
- It searches up the chain of contexts (from child to parent) for the key.
- Since the return type is `interface{}` (now `any` in Go 1.18+), you need to type assert the result.

**Avoiding Key Collisions:**
- Context keys should be unique to avoid collisions, especially when different packages might use the same keys.
- Do not use basic types like `string` directly as keys (e.g., `"id"`).
- Two patterns to create unique keys:
  - **Custom Unexported Type:**
    ```go
    type userKey int
    const (
        _ userKey = iota
        key
    )
    ```
  - **Empty Struct Type:**
    ```go
    type userKey struct{}
    const key = userKey{}
    ```
- By using an unexported type or value, you ensure that other packages cannot use the same key.

---

## Context Cancellation in Go

### **Why Use Cancellation?**

Imagine a web application that, upon receiving a request, launches several goroutines to perform tasks like fetching data from different services. If one of these tasks fails in a way that makes it impossible to fulfill the original request, there's no point in continuing with the other tasks. Continuing would waste resources and possibly degrade the performance of your application.

By canceling the remaining tasks, you can:

- **Free up resources**: Goroutines can exit early instead of running unnecessarily.
- **Improve responsiveness**: You can return an error to the client promptly.
- **Maintain consistency**: Avoid partial updates or inconsistent states.

---

### **Creating a Cancellable Context**

To implement cancellation, you use `context.WithCancel`.

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

- **`parent`**: The parent context.
- **`ctx`**: The new derived context that can be canceled.
- **`cancel`**: A function that, when called, cancels `ctx` and any contexts derived from it.

#### **Example:**

```go
ctx, cancelFunc := context.WithCancel(context.Background())
defer cancelFunc() // Ensure that the context is canceled when we're done with it.
```

- **Always** call `cancelFunc()` to release resources associated with the context.
- Using `defer` ensures `cancelFunc()` is called when the enclosing function returns, whether due to normal execution or an error.

---

### **Detecting Cancellation**

To check if a context has been canceled, use the `Done()` method:

```go
doneChan := ctx.Done()
```

- **`ctx.Done()`** returns a channel that's closed when the context is canceled.
- Goroutines can select on this channel to wait for cancellation.

#### **Example:**

```go
select {
case <-ctx.Done():
    // Context canceled, exit early.
    return
default:
    // Continue processing.
}
```

---

- Calling `cancelFunc()` more than once has no adverse effects—the first call cancels the context, subsequent calls do nothing.
- By using `defer cancelFunc()` and having goroutines exit correctly, we prevent resource leaks such as hanging goroutines.

---

### **Server Resource Management**

Servers are shared resources. Each client might try to consume as much of the server's resources as possible without considering other clients. It's the server's responsibility to manage its resources to provide fair service to all clients.

Generally, a server can do four things to manage its load:
• Limit simultaneous requests
• Limit the number of queued requests waiting to run
• Limit the amount of time a request can run
• Limit the resources a request can use (such as memory or disk space)

---

### **Contexts with Deadlines and Timeouts**

In Go, you can use the `context` package to create contexts that automatically cancel after a certain period (deadline) or duration (timeout).

#### **Creating a Context with a Timeout**

Use `context.WithTimeout` to create a context that cancels after a specific duration.

```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

#### **Creating a Context with a Deadline**

Use `context.WithDeadline` to create a context that cancels at a specific time.

```go
deadline := time.Now().Add(2 * time.Second)
ctx, cancel := context.WithDeadline(context.Background(), deadline)
defer cancel() // Ensure that the context is canceled to release resources
```

---

**Child Contexts Cannot Outlive Parents**: If the parent context is canceled or times out, the child context is also canceled, even if its timeout hasn't expired yet.

---

### **Determining Why a Context Was Canceled**

1. **`Err()` Method**:

   - Returns `nil` if the context is still active.
   - Returns a non-nil error if the context has been canceled:
     - `context.Canceled`: If the context was manually canceled.
     - `context.DeadlineExceeded`: If the context timed out or reached its deadline.

2. **`Cause()` Function (since Go 1.20)**:

   - Use `context.Cause(ctx)` to get the error that caused the context to be canceled.
   - If you use `context.WithCancelCause` or `context.WithTimeoutCause`, you can pass a custom error upon cancellation.
   - If not, `Cause()` will return the same error as `Err()`.

---

### **Handling Context Cancellation in Your Own Code**

#### **When Should You Handle Cancellation?**

- **Long-Running Operations**: If you write functions that may run for a significant amount of time (e.g., computationally intensive tasks), you should handle cancellation to allow the operation to stop if the context is canceled.
- **Concurrency with Channels**: If your function uses channels and `select` statements, you should include a case for `ctx.Done()` to exit promptly when canceled.
