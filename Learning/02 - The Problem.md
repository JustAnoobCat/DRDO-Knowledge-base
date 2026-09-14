## What am I actually trying to solve?

The project is not simply about "using AI to allocate TDMA slots."

The broader problem is:

> How can the ACN intelligently distribute its limited communication resources among multiple ground nodes when their traffic requirements and channel conditions are continuously changing?

TDMA can be one way of doing this, but it does not necessarily have to be the final solution.

---

## 1. Why is resource allocation difficult?

The ACN has a limited amount of communication resources.

At the same time, multiple ground nodes may want to communicate.

The difficult part is that the nodes do not all have the same situation.

One node might:

- Have a large amount of data waiting.
- Have a strong communication channel.
- Have low-priority traffic.

Another node might:

- Have very little data.
- Have a weak channel.
- Have a highly important or urgent message.

So the scheduler has to balance several things at the same time.

---

## 2. The three main things competing for resources

A useful way for me to think about the problem is that the scheduler has to balance:

### A. How much data does the node have?

If a node has a large queue, it may need more resources.

Example:

- Node A → 100 KB waiting
- Node B → 10 MB waiting

Giving both exactly the same resources may leave Node B's queue growing.

---

### B. How good is the channel?

A node with a strong channel can potentially transmit data more efficiently.

A weak channel may allow less data to be transmitted using the same resources.

Channel quality can change because of things such as:

- Rain
- Obstructions
- Line-of-sight changes
- Fading
- Distance

So the scheduler should potentially consider channel quality when making decisions.

---

### C. How urgent is the data?

Not all data can tolerate the same amount of delay.

For example:

A small emergency command may be much more important than a large video stream.

So the scheduler cannot simply prioritize the node with the biggest queue.

It also needs to consider **QoS and urgency**.

---

## 3. Why can static/equal allocation be inefficient?

Imagine four nodes and the ACN divides the available time equally:

```text
Node 1 → 25%
Node 2 → 25%
Node 3 → 25%
Node 4 → 25%
```

This is simple and predictable.

But now imagine:

```text
Node 1 → almost no data
Node 2 → huge video queue
Node 3 → emergency command
Node 4 → moderate traffic
```

The equal allocation does not understand these differences.

It still gives Node 1 25%.

It still gives Node 3 only 25%, even though its emergency message may need immediate service.

This is why we need **dynamic resource allocation**.

---

## 4. What does "dynamic" mean?

Dynamic means that the allocation can change according to the current network situation.

For example:

### At time T1

```text
Node 1 → large queue
Node 2 → small queue
Node 3 → good channel
Node 4 → weak channel
```

The scheduler makes one allocation.

### A few milliseconds later

```text
Node 1 → queue decreased
Node 2 → emergency data arrived
Node 3 → still good channel
Node 4 → rain caused SNR to drop
```

The scheduler may make a completely different allocation.

So the scheduler is not supposed to make one decision and keep it forever.

It repeatedly observes the network and adapts.

---

## 5. This is an optimization problem

At a high level, the scheduler wants to achieve several goals.

### Increase useful throughput

Transmit as much useful data as possible.

### Reduce delay

Important packets should not sit in queues for too long.

### Reduce packet drops

Packets that wait too long or encounter buffer limitations may be dropped.

### Maintain fairness

One node should not be allowed to consume everything while other nodes receive nothing.

### Respect constraints

The scheduler cannot allocate more resources than the ACN actually has.

---

## 6. Why can't we simply maximize throughput?

Because maximum throughput is not always the same as a good network.

Imagine:

```text
Node A → excellent channel
Node B → poor channel
```

If the scheduler only cares about throughput, it might keep giving resources to Node A because Node A can transmit lots of data efficiently.

Node B could then be ignored.

That might give excellent total throughput but poor service to Node B.

Similarly, if an emergency command is waiting at Node B, maximizing total throughput could still produce a bad decision.

So the scheduler needs a **balanced objective**.

---

## 7. Different algorithms can solve this problem

This is where different scheduling approaches come in.

A simple progression is:

```text
Static TDMA
     ↓
Rule/Heuristic-based scheduling
     ↓
Optimization-based scheduling
     ↓
Reinforcement Learning
```

Each approach has advantages and disadvantages.

### Static TDMA

Very simple and predictable.

But it does not adapt well to changing network conditions.

### Heuristic scheduling

Uses predefined rules.

For example:

> Give more resources to nodes with large queues or urgent packets.

This is more adaptive but still relies on manually designed rules.

### Optimization

Formulate the resource allocation problem mathematically and find a good solution.

This can potentially produce strong decisions, but solving the optimization repeatedly may be computationally expensive.

### Reinforcement Learning

Instead of explicitly writing every scheduling rule, an agent learns which decisions produce good long-term results.

This could allow it to learn complicated relationships between:

- Queue sizes
- Channel conditions
- Delays
- Traffic
- Resource allocation

But RL also introduces additional complexity and training requirements.

---

## 8. Important realization

I initially might think:

> "Our project is an AI-powered TDMA scheduler."

A better understanding is:

> "Our project is about intelligent resource allocation in a changing aerial communication network. TDMA is one possible resource-allocation mechanism or baseline, while other scheduling approaches can be evaluated."

This is important because I should not choose an algorithm simply because it sounds more advanced.

The final approach should be selected based on:

- Performance
- Complexity
- Computational requirements
- Ability to adapt
- Reliability
- Suitability for the simulated tactical network

---

## 9. What does the AI actually need to learn?

If we eventually use reinforcement learning, the AI needs to learn something like:

> "Given the current network situation, what resource allocation decision will give me the best overall result?"

For example:

```text
Current situation
        ↓
Queue sizes
Channel quality
Traffic urgency
Previous performance
        ↓
       AI
        ↓
Resource allocation
        ↓
Network changes
        ↓
New situation
```

The AI learns through repeated interaction with the simulated network.

---

## 10. The project therefore has two important parts

### Part 1 — Model the communication network

I need to create a simulation that represents:

- Ground nodes
- Data traffic
- Queues
- Channel conditions
- Resource availability
- Transmission
- Delays
- Packet drops

### Part 2 — Make the scheduler intelligent

I then need to test different ways of allocating resources.

For example:

```text
Static TDMA
     vs
Dynamic/heuristic scheduler
     vs
Optimization
     vs
RL/DRL
```

The important question is:

> Does the smarter scheduler actually perform better than the simpler approaches?

---

## What I understand now

The core problem is not just TDMA.

The actual problem is **dynamic resource allocation**.

The ACN has limited resources and multiple nodes with different and changing requirements.

The scheduler needs to balance:

- Data backlog
- Channel quality
- Delay
- Traffic priority
- Throughput
- Fairness
- Resource constraints

A good solution should adapt to changes rather than using a fixed allocation.

AI/ML is a possible solution, but it should be evaluated against simpler approaches rather than being assumed to be the best automatically.

---

## Things I still need to understand

- What exactly counts as a "resource" in our system?
- Are we allocating only time, or also frequency and power?
- How does bandwidth relate to these resources?
- What exactly is a TDMA time slot?
- How does SNR affect achievable data rate?
- How do we mathematically represent the scheduling problem?
- What algorithms should we actually compare?
- Is DRL necessary for our project, or can a simpler method perform well?

---

## One-line understanding

> **The ACN needs to continuously decide how to distribute limited communication resources among nodes with different amounts of data, channel conditions, and urgency.**