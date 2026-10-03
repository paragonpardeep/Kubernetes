# Apache Kafka for SREs — From Zero to Production

> **Goal:** Understand Kafka well enough to troubleshoot, monitor, operate, and support it in production.

---

# 🧠 1. Kafka in One Sentence

**Kafka is a distributed system for moving and storing streams of messages/events.**

Think:

```text
Producer
   ↓
 Kafka
   ↓
Consumer
```

Example:

```text
Application → Kafka → Payment Service
```

The application produces a payment event, Kafka stores it, and the payment service consumes it.

---

# 🧩 2. The Kafka Mental Model

Remember these **6 words**:

```text
Producer → Topic → Partition → Broker → Consumer → Consumer Group
```

If you understand these six, you understand the foundation of Kafka.

---

# 📤 3. Producer

A **Producer sends messages to Kafka**.

Example:

```text
Application
    ↓
Producer
    ↓
Kafka Topic
```

Message:

```json
{
  "order_id": 123,
  "amount": 500
}
```

### Remember

> **Producer = Sends data**

---

# 📦 4. Topic

A **Topic is a logical stream/category of messages.**

Example:

```text
orders
payments
user-events
logs
```

Think of a topic as a **named stream**.

```text
orders topic
────────────────────────────
order1 → order2 → order3 → order4
```

### Remember

> **Topic = Where messages belong**

---

# 🧱 5. Partition

A topic is divided into **partitions**.

Example:

```text
orders
   │
   ├── Partition 0
   ├── Partition 1
   └── Partition 2
```

Messages are distributed across partitions.

Why?

### Mainly for:

* Scalability
* Parallel processing
* Throughput

### Important

Kafka ordering is guaranteed **within a partition**, not across the entire topic.

```text
Partition 0:
A → B → C → D

Partition 1:
E → F → G
```

### Remember

> **Partition = Parallelism + Ordering boundary**

---

# 🖥️ 6. Broker

A **Broker is a Kafka server**.

Example:

```text
Kafka Cluster

Broker 1
Broker 2
Broker 3
```

The brokers store Kafka partitions.

### Remember

> **Broker = Kafka server**

---

# 👥 7. Consumer

A **Consumer reads messages from Kafka.**

```text
Kafka
  ↓
Consumer
  ↓
Application
```

### Remember

> **Consumer = Reads data**

---

# 👥 8. Consumer Group

This is one of the **most important SRE concepts**.

Multiple consumers can work together as one logical group.

```text
Consumer Group: payments

       Kafka
         │
   ┌─────┼─────┐
   ↓     ↓     ↓
 C1      C2     C3
```

Kafka distributes partitions among consumers.

### Important Rule

> **One partition can be actively consumed by only one consumer within the same consumer group at a time.**

Example:

```text
3 partitions
3 consumers

P0 → C1
P1 → C2
P2 → C3
```

If you have:

```text
3 partitions
5 consumers
```

then some consumers will have no partition assigned.

```text
P0 → C1
P1 → C2
P2 → C3

C4 → idle
C5 → idle
```

### Remember

> **Partitions determine the maximum parallelism within a consumer group.**

---

# 💾 9. Kafka Does NOT Immediately Delete Messages

This is a common beginner misunderstanding.

A consumer reading a message does **not normally mean the message disappears immediately**.

Kafka retains messages according to its retention configuration.

Example:

```text
Topic
│
├── Message 1
├── Message 2
├── Message 3
├── Message 4
└── Message 5
```

Consumer reads them:

```text
Consumer
   ↓
Reads messages
```

Kafka can still retain them.

Retention may be based on:

* Time
* Size

### Remember

> **Reading a message ≠ deleting a message**

---

# 📍 10. Offset

An **offset identifies the position of a message within a partition.**

Example:

```text
Partition 0

Offset:
  0 → A
  1 → B
  2 → C
  3 → D
  4 → E
```

Consumer has processed:

```text
0
1
2
```

Its position is tracked using offsets.

### Remember

> **Offset = Position of a message in a partition**

---

# 🚦 11. Consumer Lag

This is one of the **most important Kafka SRE concepts**.

Imagine Kafka has:

```text
Latest message = 1000
```

Consumer has processed:

```text
Consumer position = 900
```

Then approximately:

```text
Lag = 100 messages
```

Conceptually:

```text
Kafka
──────────────────────────
                 ↑
              Latest
                 1000

Consumer
──────────────────
              ↑
             900
```

### High lag means

The consumer is **falling behind**.

Possible reasons:

* Consumer is slow
* Consumer processing is expensive
* Too many messages
* Insufficient consumers
* Downstream dependency is slow
* Network problems
* CPU/memory pressure

### Remember

> **Lag = Kafka has data waiting that the consumer hasn't processed yet.**

---

# 🔁 12. Replication

Kafka can keep copies of partitions on multiple brokers.

Example:

```text
Partition 0

Broker 1 → Leader
Broker 2 → Replica
Broker 3 → Replica
```

If Broker 1 fails:

```text
Broker 2
   ↓
Can become leader
```

This provides fault tolerance.

### Remember

> **Replication = Copies of partition data for resilience**

---

# 👑 13. Leader and Follower

For each partition, Kafka has a leader and replicas.

```text
Partition 0

Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower
```

Clients generally interact with the partition leader for normal reads/writes.

### Remember

> **Leader = Handles partition operations**

---

# 🧮 14. Replication Factor

Example:

```text
replication.factor = 3
```

means Kafka keeps **three replicas** of each partition.

```text
P0:
Broker 1
Broker 2
Broker 3
```

### Remember

> **Replication Factor = Number of copies**

---

# ❤️ 15. ISR

ISR means:

**In-Sync Replicas**

These are replicas considered sufficiently caught up with the leader.

Example:

```text
Leader
  │
  ├── Broker 2 → ISR
  └── Broker 3 → ISR
```

If a replica falls too far behind, it may leave the ISR.

### SRE importance

A shrinking ISR can indicate:

* Broker problems
* Disk performance problems
* Network issues
* Replica lag

### Remember

> **ISR = Replicas currently keeping up with the leader**

---

# 🧠 16. Kafka Cluster

A Kafka cluster is multiple Kafka brokers working together.

```text
             Kafka Cluster

      ┌────────┬────────┐
      ↓        ↓        ↓
   Broker 1  Broker 2  Broker 3
      │        │        │
      └──── Topics/Partitions ────┘
```

Kafka distributes partitions across brokers.

---

# 🔥 17. The Most Important Production Flow

Remember this:

```text
Application
     │
     │ Produce
     ↓
   Topic
     │
     ↓
 Partition
     │
     ↓
  Broker
     │
     │ Consume
     ↓
Consumer Group
     │
     ↓
Application
```

---

# 🚨 18. Common SRE Kafka Problems

## Problem 1 — Consumer Lag Increasing

Check:

```text
Consumer health
       ↓
Consumer CPU / Memory
       ↓
Processing time
       ↓
Downstream dependencies
       ↓
Number of partitions
       ↓
Number of consumers
```

Ask:

> **Why can't the consumer process messages fast enough?**

---

## Problem 2 — Broker Disk Almost Full

Check:

```text
Disk usage
Retention
Partition distribution
Message volume
Log segments
```

Possible actions:

* Increase storage
* Review retention
* Rebalance workload
* Investigate abnormal message growth

Remember:

> **Kafka stores messages on disk.**

---

## Problem 3 — Under-Replicated Partitions

Check:

```text
Broker health
Network
Disk I/O
Replication lag
CPU
```

Remember:

> **Under-replicated partitions = replication is not healthy.**

---

## Problem 4 — Consumer Can't Connect

Check:

```text
DNS
Network
Security groups / firewall
Listeners
Authentication
Authorization
Broker availability
```

---

## Problem 5 — Messages Are Being Produced but Not Consumed

Think:

```text
Producer
   ↓
Topic?
   ↓
Partition?
   ↓
Consumer group?
   ↓
Consumer running?
   ↓
Consumer lag?
   ↓
Errors?
```

---

# 📊 19. Kafka Metrics an SRE Should Know

Focus on these first:

| Metric                      | What it tells you          |
| --------------------------- | -------------------------- |
| Consumer Lag                | Consumer is falling behind |
| Under-Replicated Partitions | Replication problem        |
| Offline Partitions          | Partition unavailable      |
| Bytes In                    | Incoming traffic           |
| Bytes Out                   | Outgoing traffic           |
| Messages In                 | Message rate               |
| Disk Usage                  | Storage pressure           |
| CPU                         | Broker resource usage      |
| Network                     | Network pressure           |
| Request Latency             | Kafka request performance  |

---

# 🔐 20. Kafka Security

Three words are important:

```text
Authentication
Authorization
Encryption
```

### Authentication

**Who are you?**

Example:

```text
SASL
```

### Authorization

**What are you allowed to do?**

Example:

```text
Can this user read the orders topic?
```

### Encryption

**Can others read traffic?**

Example:

```text
TLS
```

### Remember

> **Authentication = Who**
>
> **Authorization = What can you do**
>
> **TLS = Protect the communication**

---

# 🧰 21. Kafka Troubleshooting Method

When something breaks, don't randomly check everything.

Use:

```text
1. Is Kafka healthy?
        ↓
2. Are brokers healthy?
        ↓
3. Are partitions healthy?
        ↓
4. Is replication healthy?
        ↓
5. Is the producer working?
        ↓
6. Is the consumer working?
        ↓
7. Is consumer lag increasing?
        ↓
8. Is the downstream application healthy?
```

---

# 🧠 22. The SRE Golden Questions

Whenever you see a Kafka incident, ask:

### 1. Is data being produced?

```text
Producer → Kafka
```

### 2. Is Kafka healthy?

```text
Broker
Disk
CPU
Network
```

### 3. Are partitions healthy?

```text
Leader?
ISR?
Under-replicated?
Offline?
```

### 4. Is the consumer healthy?

```text
Running?
Errors?
Processing?
```

### 5. Is lag increasing?

```text
Yes → Consumer can't keep up
No  → Consumer is catching up
```

---

# 🎯 23. Kafka vs Traditional Queue

A simple mental difference:

```text
Traditional Queue

Producer → Queue → Consumer
                    ↓
                 Message
                 consumed
```

Kafka:

```text
Producer → Topic → Consumer
                    ↓
                 Offset
                    ↓
            Message can remain
             based on retention
```

Kafka is designed for **durable, distributed event streaming and high-throughput workloads**.

---

# 🧠 24. Kafka in One Picture

Memorize this diagram:

```text
                  PRODUCER
                     │
                     ▼
              ┌─────────────┐
              │    TOPIC    │
              └─────────────┘
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Partition   Partition   Partition
          │          │          │
          └──────────┼──────────┘
                     │
                 KAFKA BROKERS
                     │
                     ▼
              CONSUMER GROUP
              ┌──────┼──────┐
              ▼      ▼      ▼
             C1     C2     C3
                     │
                     ▼
                APPLICATION
```

---

# ⭐ The 12 Things You Should Never Forget

```text
Producer     = Sends messages

Topic        = Logical stream

Partition    = Parallelism + ordering boundary

Broker       = Kafka server

Consumer     = Reads messages

Consumer Group = Consumers working together

Offset       = Message position

Lag          = Consumer is behind

Replication  = Copies of data

Leader       = Active partition replica

ISR          = Replicas keeping up

Retention    = How long Kafka keeps data
```

---

# 🚀 SRE Learning Path

Don't try to learn everything at once.

```text
Level 1
  ↓
Producer / Topic / Consumer
  ↓
Level 2
  ↓
Partitions / Offsets / Consumer Groups
  ↓
Level 3
  ↓
Replication / Leader / ISR
  ↓
Level 4
  ↓
Consumer Lag / Monitoring / Alerting
  ↓
Level 5
  ↓
Performance / Capacity / Troubleshooting
  ↓
Level 6
  ↓
Security / HA / Disaster Recovery
  ↓
Level 7
  ↓
Production Architecture
```

## 🏆 Final Mental Model

> **Kafka is a distributed event-streaming platform. Producers write events to Topics. Topics are divided into Partitions, which live on Brokers. Consumers read those events, usually through Consumer Groups. Offsets track consumer progress, Retention controls how long data stays, and Replication provides fault tolerance.**

If you remember that paragraph and the **Producer → Topic → Partition → Broker → Consumer Group → Offset → Lag** flow, you have the foundation needed to start troubleshooting Kafka as an SRE.
