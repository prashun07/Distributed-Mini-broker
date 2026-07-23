# Distributed Publish-Subscribe Message Broker

> A high-performance, multi-threaded **Publish-Subscribe Message Broker** built in modern **C++20**, inspired by systems like **Apache Kafka** and **RabbitMQ**.

This project demonstrates the design and implementation of a scalable event-driven messaging system supporting multiple publishers, multiple subscribers, configurable delivery policies, backpressure handling, durability, and network communication.

---

## Overview

Modern distributed systems rely heavily on asynchronous communication between services.

Instead of communicating directly, services publish events to a broker. Interested consumers subscribe to topics and receive messages independently, allowing systems to scale while remaining loosely coupled.

This project implements the core concepts behind real-world messaging systems used in:

- Event-driven microservices
- Trading platforms
- IoT telemetry pipelines
- Log aggregation systems
- Distributed monitoring infrastructure
- Backend service communication

---

## Architecture

```text
                    +----------------------+
                    |      Publishers      |
                    +----------------------+
                     |        |         |
                     |        |         |
             Publish |        | Publish |
                     ▼        ▼         ▼

            +--------------------------------+
            |          Message Broker         |
            |--------------------------------|
            |  Topic Manager                 |
            |  Delivery Policy Engine        |
            |  ThreadSafe Topic Queues       |
            |  Subscriber Registry           |
            +--------------------------------+
                 |                  |
        Topic: Orders        Topic: Metrics
                 |                  |
                 ▼                  ▼

        +----------------+   +----------------+
        | ThreadSafeQueue |   | ThreadSafeQueue|
        +----------------+   +----------------+
           |         |             |
           |         |             |
           ▼         ▼             ▼

     Subscriber1 Subscriber2 Subscriber3
      (Thread)     (Thread)    (Thread)
```

---

## Example Flow

```text
Publisher A ──▶ topic:"orders" ──┐
                                 │
Publisher B ──▶ topic:"orders" ──┼──▶ ThreadSafeQueue
                                 │           │
                                 │           ├──▶ Subscriber 1
                                 │           └──▶ Subscriber 2
                                 │
Publisher C ──▶ topic:"metrics" ─┘
                                             │
                                             └──▶ Subscriber 3
```

Each topic maintains its own message queue, allowing publishers and subscribers to operate independently without blocking one another.

---

# Features

## Core Broker

- Multiple publishers
- Multiple subscribers
- Topic-based routing
- Thread-safe message queues
- Configurable delivery policies
- Graceful shutdown using `std::jthread`
- Modern C++20 implementation

---

## Message Model

Each message contains:

```cpp
Message {
    uint64_t id;
    std::string topic;
    std::string payload;
    Timestamp timestamp;
}
```

---

## Delivery Policies

The broker supports multiple delivery strategies.

### Competing Consumers

- Every message is consumed exactly once.
- Multiple subscribers compete to receive messages.
- Suitable for worker queues and task distribution.

```
Message
   │
   ▼
Subscriber A
```

---

### Broadcast

- Every subscriber receives every message.
- Each subscriber maintains its own cursor/queue.
- Suitable for event streaming and notifications.

```
             Message
          /     |     \
         ▼      ▼      ▼
     Sub A   Sub B   Sub C
```

The delivery strategy is abstracted through a **Strategy Pattern**, making it easy to extend with new policies.

---

# Design

## Main Components

### Broker

- Owns all topics
- Routes published messages
- Maintains subscriber registry

---

### Topic

Responsible for

- queue management
- subscriber registration
- message ordering
- delivery policy

---

### Publisher

- Produces messages
- Publishes concurrently
- Independent of subscribers

---

### Subscriber

Runs in its own thread using

```cpp
std::jthread
```

Supports

- graceful shutdown
- reconnect
- offset tracking

---

### ThreadSafeQueue

Provides

- thread-safe push/pop
- blocking operations
- bounded queues
- backpressure support

This project extends an existing `ThreadSafeQueue` implementation instead of replacing it.

---

# Modern C++ Features

- C++20
- `std::jthread`
- `std::stop_token`
- `std::shared_ptr`
- Concepts
- Smart pointers
- RAII
- Condition variables
- Mutexes
- Atomics
- Strategy Pattern
- Observer Pattern
- Producer–Consumer Pattern

Stretch Goal:

- C++20 Coroutines for asynchronous networking

---

# Backpressure

A real message broker cannot allow queues to grow indefinitely.

Supported policies:

- Block publisher
- Drop oldest message
- Drop newest message

This prevents memory exhaustion when subscribers are significantly slower than publishers.

---

# Networking

The broker runs as a dedicated server process.

Clients communicate using TCP.

```
Publisher
      \
       \
        ---> Broker <---- Subscriber
       /
Publisher
```

Protocol examples:

```
SUBSCRIBE orders

PUBLISH orders Order Created

ACK
```

Networking is implemented using **Boost.Asio**.

---

# Durability

Messages are persisted using an append-only write-ahead log (WAL).

Features include:

- crash recovery
- broker restart recovery
- per-topic logs
- subscriber offset persistence

Delivery guarantees:

- At-Least-Once Delivery
- Per-Topic Ordering
- No Exactly-Once semantics

---

# Observability

Broker administration includes:

- Topic listing
- Queue depth
- Consumer lag
- Published message count
- Consumed message count
- Thread-safe structured logging

---

# Testing

Stress tests include:

- 10 publishers
- 20 subscribers
- Multiple topics
- Subscriber disconnect/reconnect
- Broker restart recovery
- ThreadSanitizer validation

Example workload:

- 10,000+ messages
- 5 publisher threads
- 3 topics
- Multiple delivery policies

---

# Project Roadmap

## Phase 1 — Core Broker

- [ ] Design Message class
- [ ] Implement Topic abstraction
- [ ] Extend ThreadSafeQueue
- [ ] Delivery Policy interface
- [ ] Broker implementation

---

## Phase 2 — Multi-threaded Pub/Sub

- [ ] Publisher threads
- [ ] Subscriber threads
- [ ] Topic routing
- [ ] Graceful shutdown
- [ ] ThreadSanitizer validation

---

## Phase 3 — Backpressure

- [ ] Bounded queues
- [ ] Blocking policy
- [ ] Drop-oldest policy
- [ ] Drop-newest policy

---

## Phase 4 — Networking

- [ ] Broker server
- [ ] TCP clients
- [ ] Publish protocol
- [ ] Subscribe protocol
- [ ] Reconnection handling

---

## Phase 5 — Durability

- [ ] Write-Ahead Log
- [ ] Offset persistence
- [ ] Recovery after restart

---

## Phase 6 — Monitoring

- [ ] Admin CLI
- [ ] Consumer lag
- [ ] Metrics
- [ ] Logging

---

## Phase 7 — Production Readiness

- [ ] Stress testing
- [ ] Performance benchmarks
- [ ] Documentation
- [ ] Architecture diagrams

---

# Stretch Goals

- Topic partitioning
- Distributed broker cluster
- Leader election
- Replication
- Consumer groups
- Load balancing
- Zero-copy networking
- Coroutines
- Web dashboard
- Prometheus metrics
- gRPC interface
- Idempotency keys
- Exactly-once delivery research

---

# Why This Project?

Unlike a simple producer-consumer queue, this project tackles real distributed systems challenges:

- Concurrent publishers
- Concurrent subscribers
- Message ordering
- Backpressure
- Delivery guarantees
- Network communication
- Persistence
- Offset management
- Failure recovery
- Scalable architecture

It closely mirrors the design principles behind production-grade messaging systems such as Kafka and RabbitMQ while remaining compact enough to understand, extend, and discuss in technical interviews.

---

# Tech Stack

- **Language:** C++20
- **Networking:** Boost.Asio
- **Concurrency:** std::thread / std::jthread
- **Synchronization:** Mutex, Condition Variable, Atomics
- **Persistence:** File-based Write-Ahead Log (WAL)
- **Testing:** GoogleTest, ThreadSanitizer
- **Build System:** CMake
- **Platform:** Linux / macOS

---

# Future Improvements

- Kafka-compatible protocol
- Web-based management dashboard
- Distributed broker cluster
- Raft consensus
- Replicated logs
- Persistent storage engine
- High availability
- Topic partitioning
- Horizontal scalability
- Cloud deployment
```
