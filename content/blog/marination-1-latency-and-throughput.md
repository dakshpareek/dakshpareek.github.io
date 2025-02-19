---
title: "Marination 1: Latency and Throughput"
date: 2025-02-19
tags: ["Marination", "Latency", "Throughput"]
draft: false
---

Marination is a series of notes which I will create while understanding a resource. So basically they are notes of notes/blogs/videos. They may contain my conversation with LLM which made me understand the topic.

This is the first marination on the topic of Latency and Throughput. This is from blog [Latency, Throughput, and Walking on Escalators](https://blog.tacertain.com/latency-throughput-and-escalators/) by Andrew Certain.

### **Summary of the Article**

The article discusses how the way people use escalators relates to important concepts in computer systems: **latency** and **throughput**. By examining how walking or standing on escalators affects overall efficiency, the author illustrates how these concepts play out in real-world scenarios and computer systems.

---

### **Key Concepts Explained**

#### **1. Latency and Throughput**

- **Latency**: The time it takes for a single task to complete. In this context, it's how long it takes one person to ride the escalator from bottom to top.

- **Throughput**: The amount of work done per unit of time. Here, it's the number of people transported by the escalator per minute.

#### **2. Little's Law**

- **Little's Law** is a fundamental principle in queueing theory that relates the average number of items in a system (L) to the average arrival rate (λ) and the average time an item spends in the system (W):

  \[
  L = \lambda \times W
  \]

  - **L**: Average number of items in the system (e.g., people on the escalator).
  - **λ** (lambda): Average arrival rate (e.g., people getting on the escalator per minute).
  - **W**: Average time an item spends in the system (e.g., time each person spends on the escalator).

**In simpler terms:**

- If you know how often items arrive (\( \lambda \)) and how long they stay (\( W \)), you can find out how many items are in the system on average (\( L \)).

---

### **Applying These Concepts to Escalators**

#### **Scenario Overview**

- There's an escalator in a busy train station.
- People either stand still or walk up the escalator.
- The escalator can be used in two ways:
  - **Everyone stands**: The escalator moves people who are standing.
  - **Half stand, half walk**: People stand on one side and walk on the other.

#### **Key Observations**

1. **Walking vs. Standing**

   - **Walking**: Individuals reach the top faster (lower personal latency), but because walkers need more space between them to move safely, fewer people can be on the escalator at once.
   - **Standing**: People are closer together, so more individuals can be on the escalator at the same time, potentially increasing throughput.

2. **Impact on Throughput and Latency**

   - When everyone stands:
     - **Throughput increases**: More people are on the escalator at once.
     - **Latency per person increases**: Each person spends more time on the escalator because they're not walking.
   - When people walk on one side:
     - **Throughput decreases**: Walking side is underutilized if not enough people choose to walk.
     - **Latency for walkers decreases**: Walkers reach the top faster.
     - **Latency for standers may increase**: Fewer people can stand on the escalator if half of it is reserved for walkers.

#### **The Study's Findings**

- **Enforcing a No-Walking Rule** (everyone stands):

  - Utilizes the entire width of the escalator.
  - **Decreases overall wait time** (total latency for all users).
  - **Increases capacity**: More people can be on the escalator simultaneously.
  - **Potentially frustrates walkers**: Those who prefer to walk can't move faster.

- **Allowing Walking on One Side**:

  - **Underutilizes the walking side**: Not enough people choose to walk.
  - **Decreases throughput**: Fewer people are moved per minute.
  - **Increases wait times** for standers on the platform since fewer people can get on the escalator.
  - **Benefits walkers**: Walkers experience lower personal latency.

---

### **Balancing Individual and Group Benefits**

- **Individual vs. Collective Efficiency**:

  - **Walkers benefit individually** by reaching the top faster.
  - **Overall system efficiency may decrease** because the escalator moves fewer people per minute.

- **No Mathematically Correct Answer**:

  - The optimal solution depends on what we prioritize:
    - **Maximizing total throughput and minimizing average wait times** suggests everyone should stand.
    - **Allowing individuals to choose** (standing or walking) gives some people faster service but can reduce overall efficiency.

- **Social Considerations**:

  - Decisions about escalator usage are not just about math and efficiency; they involve social norms and fairness.
  - Some people may resent being forced to stand or may value the ability to walk and save time.

---

### **Connecting to Computer Systems**

- **Latency in Computer Systems**:

  - Similar to escalators, in computer systems, latency is the time it takes for a user’s request to be processed.
  - Users care about how quickly they get a response (latency).

- **Throughput in Computer Systems**:

  - Refers to the number of tasks a system can handle in a given time.
  - High throughput doesn't always mean lower latency.

- **Trade-Offs and Optimization**:

  - **Optimizing for Throughput**:

    - May involve processing many tasks simultaneously.
    - Could lead to longer wait times for individual tasks if resources are stretched.

  - **Optimizing for Latency**:

    - Focusing on completing individual tasks quickly.
    - Might reduce overall throughput if fewer tasks are processed at once.

- **Customer Experience**:

  - Ultimately, users care about latency—the time they wait for a response.
  - Systems should aim to balance throughput and latency to optimize user satisfaction.

---

### **Simplifying the Concepts Further**

- **Think of a Traffic Jam**:

  - If everyone drives slowly and steadily, more cars can be on the road, and traffic might flow more smoothly.
  - If some drivers speed and weave through traffic, they might get to their destination faster, but it can cause disruptions, leading to overall slower traffic for others.

---

---

### **1. Web Servers and HTTP Request Handling**

**Scenario:**

- Web servers handle incoming HTTP requests from clients (browsers, apps).
- Goal: Serve content to users quickly (low latency) while supporting many users simultaneously (high throughput).

**Challenges:**

- **High Latency Impact:** Users experience slow page loads, leading to poor user experience and potential revenue loss.
- **Low Throughput Impact:** The server can't handle many users at once, causing requests to be queued or dropped.

**Design Considerations:**

- **Asynchronous I/O and Event-Driven Architectures:**

  - Use non-blocking I/O to handle multiple requests without waiting for each to complete sequentially.
  - Example: Node.js uses an event loop to process many connections concurrently.

- **Load Balancing:**

  - Distribute incoming requests across multiple servers.
  - Improves throughput by handling more requests in parallel.
  - Reduces latency by preventing any single server from becoming a bottleneck.

- **Caching:**

  - Store responses for frequently requested resources.
  - Reduces latency by serving content quickly without generating it anew.
  - Increases throughput by reducing the processing load on the server.

- **Content Delivery Networks (CDNs):**

  - Use geographically distributed servers to serve content closer to users.
  - Lowers latency due to reduced network distance.
  - Offloads traffic from the origin server, improving throughput.

---
