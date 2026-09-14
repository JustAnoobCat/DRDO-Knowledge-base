## Why I am learning this

I already understand that the scheduler has to decide **who gets communication resources and how much**.

Before learning about Reinforcement Learning, I need to understand what the scheduler is actually trying to optimize.

The important idea is:

> **Scheduling is a decision problem where I want the best possible use of limited communication resources while satisfying important constraints.**

---

# 1. What is the scheduler trying to achieve?

Suppose the ACN has 16 ground nodes but only a limited amount of communication capacity.

At some instant:

- Node 1 has a huge queue.
- Node 2 has almost nothing to send.
- Node 3 has urgent data.
- Node 4 has a poor X-band channel.
- Node 5 has a very good channel.
- Other nodes have different amounts of data.

The scheduler cannot simply give everyone everything.

It has to decide:

> **How should the available resources be distributed?**

This means there needs to be some definition of a **good allocation**.

For example, a good allocation might:

- transmit lots of useful data,
- avoid packets waiting too long,
- avoid packet drops,
- give urgent traffic appropriate treatment,
- avoid starving some nodes,
- avoid wasting available resources.

These goals can sometimes conflict with each other.

---

# 2. Multiple objectives make scheduling difficult

Consider two nodes:

```text
Node A:
Queue = 100 packets
SNR   = excellent

Node B:
Queue = 10 packets
SNR   = poor
```

If I want maximum throughput, I might give more resources to Node A.

But suppose Node B's 10 packets are **critical control information**.

Then completely ignoring Node B could be a bad decision.

So the scheduler may need to balance:

```text
Throughput
    ↓
Latency
    ↓
Packet drops
    ↓
Fairness
    ↓
Priority / QoS
    ↓
Resource utilization
```

There is usually no single metric that describes everything.

---

# 3. Throughput

### What I understand

**Throughput** is the amount of useful data successfully transmitted over some period of time.

For example:

```text
Node sends 10 MB in 1 second

Throughput = 10 MB/s
```

Higher throughput generally means the network is successfully delivering more data.

For our project:

> I probably want the scheduler to make good use of the ACN's available communication capacity.

---

# 4. Latency

Latency is the amount of time data waits before being delivered.

Example:

```text
Packet arrives
     ↓
waits in queue
     ↓
gets transmitted
     ↓
received
```

If a packet waits 80 ms before being served, its delay is much higher than a packet served after 5 ms.

This matters because different traffic can have different timing requirements.

For example:

```text
Critical control data → very delay sensitive
Voice                 → delay/jitter sensitive
Video                 → high bandwidth demand
Bulk data             → usually less urgent
```

So maximizing throughput alone is not enough.

---

# 5. Packet drops

A packet can be dropped because:

- the queue becomes full,
- it misses its deadline,
- the channel becomes unusable,
- the simulation imposes some other limitation.

Example:

```text
Packet deadline = 20 ms

Packet waits:
5 ms
10 ms
15 ms
20 ms
21 ms  → too late → dropped
```

A good scheduler should try to avoid unnecessary packet drops.

Especially for important traffic.

---

# 6. Fairness

Suppose there are four nodes:

```text
Node 1 → gets 90% of resources
Node 2 → gets 5%
Node 3 → gets 3%
Node 4 → gets 2%
```

The network might have high throughput, but Nodes 2–4 could effectively be starved.

So sometimes I also want the scheduler to be **fair**.

Fairness does NOT necessarily mean:

> Everyone gets exactly the same amount.

Instead, it can mean that resources are distributed according to some reasonable policy.

For example:

- priority,
- traffic requirements,
- channel conditions,
- previous service,
- queue size.

---

# 7. Resource utilization

Another goal is avoiding wasted resources.

Imagine:

```text
Node A → allocated 25 ms
Node A has only 2 ms worth of data
```

If the remaining 23 ms cannot be reused, that capacity is wasted.

A dynamic scheduler could potentially recognize:

```text
Node A doesn't need much
        ↓
give unused opportunity to Node B
        ↓
better utilization
```

This is one of the main reasons dynamic scheduling can be better than rigid static allocation.

---

# 8. These objectives can conflict

This is one of the most important things I learned.

Suppose:

```text
Node A → excellent channel, huge queue
Node B → poor channel, urgent data
```

Giving resources to A might:

```text
+ increase throughput
```

But giving some resources to B might:

```text
+ reduce urgent-data delay
+ prevent packet drops
```

Therefore:

> **The best scheduler is usually not simply the one that maximizes one metric.**

It needs to balance several objectives.

---

# 9. Objective function

This is where optimization becomes useful.

I can represent the scheduler's goals using an **objective function**.

Conceptually:

```text
Good allocation =
    high throughput
    - high delay
    - packet drops
    + fairness
    + useful resource utilization
```

The exact mathematical form can be designed later.

For example, conceptually:

```text
Score =
    Throughput benefit
    - Delay penalty
    - Drop penalty
    + Fairness benefit
```

The scheduler then tries to find an allocation that gives a good score.

---

# 10. Weights

Not every objective has the same importance.

For example:

```text
Throughput       → important
Delay            → very important
Packet drops     → extremely important
Fairness         → important
```

I can represent this using weights.

Conceptually:

```text
Score =
    w1 × throughput
    - w2 × delay
    - w3 × packet_drops
    + w4 × fairness
```

Where:

```text
w1, w2, w3, w4
```

represent how important each objective is.

### Important

These weights are **not something I should randomly choose and call correct**.

They depend on the actual project requirements.

So for now I understand the concept, but the final values need to be decided later.

---

# 11. Constraints

Optimization is not just:

> "Give resources to whoever has the highest score."

There are also things the scheduler **cannot violate**.

These are called **constraints**.

For example:

```text
Total available time = 10 ms

Node 1 gets 2 ms
Node 2 gets 3 ms
Node 3 gets 1 ms
Node 4 gets 4 ms

Total = 10 ms
```

The scheduler cannot allocate:

```text
Node 1 → 5 ms
Node 2 → 5 ms
Node 3 → 5 ms
```

because:

```text
5 + 5 + 5 = 15 ms
```

but only 10 ms exists.

---

# 12. Example constraints for our project

Depending on the final system design, constraints could include:

### Total available time

```text
Sum of allocated time ≤ available frame time
```

### Resource exclusivity

A resource cannot be assigned to multiple nodes at the same time if the system does not support that.

### Power limitation

If transmit power is being modeled:

```text
Total power ≤ available power
```

### Minimum service

Critical traffic might require some minimum service.

### Physical/link limitations

A node with a poor channel may not be able to transmit at the same rate as a node with a good channel.

---

# 13. Simple optimization example

Suppose one frame has:

```text
10 ms available
```

and there are three nodes.

I need to decide:

```text
Node A → ?
Node B → ?
Node C → ?
```

One possible allocation:

```text
A → 2 ms
B → 3 ms
C → 5 ms
```

Another:

```text
A → 4 ms
B → 4 ms
C → 2 ms
```

Both are technically possible.

The optimization problem is essentially asking:

> **Which valid allocation gives the best overall result?**

---

# 14. Why the channel matters

The amount of time given to a node does not necessarily translate into the same amount of data.

Example:

```text
Node A:
10 ms × high data rate
        ↓
lots of data transmitted

Node B:
10 ms × low data rate
        ↓
less data transmitted
```

So the scheduler should potentially consider both:

```text
How much resource does the node get?
                    +
How efficiently can the node use that resource?
```

This connects the scheduling problem directly to the X-band channel model I learned earlier.

---

# 15. Queue state also matters

Suppose:

```text
Node A → queue = 2 packets
Node B → queue = 100 packets
```

Giving equal resources may not be efficient.

Node B has much more waiting data.

But queue size still isn't the whole story.

Suppose:

```text
Node A → 2 packets, critical, deadline almost reached
Node B → 100 packets, bulk data
```

Then giving everything to B may also be a bad decision.

So the scheduler may need to consider:

```text
Queue
Channel
Delay
Priority
Traffic type
Previous service
Available resources
```

---

# 16. Optimization problem in simple words

I can now describe the problem as:

> **Find a valid resource allocation that gives the best overall network performance according to the objectives and constraints.**

The structure is:

```text
Current network state
        ↓
Possible resource allocations
        ↓
Remove allocations that violate constraints
        ↓
Evaluate remaining allocations
        ↓
Choose the best one
```

---

# 17. Where traditional optimization fits

A traditional optimization algorithm could try to solve this problem directly.

For example:

```text
Network state
     ↓
Optimization solver
     ↓
Best allocation
```

This can work well for smaller or simpler problems.

But there is a challenge.

The problem can become very complicated when I have:

- many nodes,
- many resources,
- changing channels,
- different traffic types,
- deadlines,
- power constraints,
- interference,
- multiple objectives.

The number of possible allocations can become very large.

---

# 18. Why this matters for AI/RL

This gives me a much better understanding of why Reinforcement Learning is being considered.

The goal isn't:

> "Use AI because AI is cool."

Instead:

```text
Communication system
        ↓
Complex dynamic scheduling problem
        ↓
Repeated decisions
        ↓
Need good allocation quickly
        ↓
Possible approaches:
    ├── Static rules
    ├── Heuristics
    ├── Optimization
    └── Reinforcement Learning
```

RL is one possible way to learn a policy for making these repeated decisions.

---

# 19. Optimization vs RL

A simplified comparison:

| Approach | Basic idea |
|---|---|
| Static scheduling | Follow predetermined allocation |
| Heuristic | Follow manually designed rules |
| Optimization | Calculate a good/best allocation |
| RL | Learn how to choose allocations from experience |

For example:

### Optimization

```text
Current state
    ↓
Solve optimization problem
    ↓
Allocation
```

### RL

```text
Current state
    ↓
Learned policy
    ↓
Allocation
```

The advantage of RL may be that after training, the policy can make decisions quickly without solving the full optimization problem from scratch every time.

But that does **not** automatically mean RL will be better.

---

# 20. Optimization can also be a benchmark

This is an important idea for the project.

If I eventually use RL, I need some way to determine whether it is actually good.

Possible comparison:

```text
Static TDMA
     vs
Heuristic scheduler
     vs
Optimization/reference method
     vs
RL scheduler
```

Then I can compare:

```text
Throughput
Latency
Packet drops
Fairness
Resource utilization
Computation time
```

This would make the project much more meaningful than simply saying:

> "Our AI scheduler works."

---

# 21. What I understand now

I understand that the scheduling problem is fundamentally an **optimization problem**.

The scheduler has:

### Inputs

```text
Current network state
```

such as:

- queue sizes,
- channel quality,
- delays,
- traffic requirements,
- available resources.

### Decision

```text
How should resources be allocated?
```

### Objectives

```text
Maximize:
- useful throughput
- resource utilization
- fairness

Minimize:
- latency
- packet drops
- wasted resources
```

### Constraints

```text
- limited time
- limited bandwidth/resources
- power limits
- resource exclusivity
- QoS requirements
- physical/link limitations
```

---

# 22. The bigger picture

I can now see the project more clearly:

```text
                DIGITAL TWIN
                     │
                     ↓
          ┌─────────────────────┐
          │ Current network     │
          │ state               │
          │                     │
          │ Queue               │
          │ SNR/SINR            │
          │ Traffic             │
          │ Delay               │
          │ Resources           │
          └──────────┬──────────┘
                     ↓
                  SCHEDULER
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Static     Heuristic   RL/AI
          │          │          │
          └──────────┼──────────┘
                     ↓
             Resource allocation
                     ↓
              Network simulation
                     ↓
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Throughput  Delay      Drops
```

The AI is therefore **one possible solution to the scheduling problem**, not the problem itself.

---

# 23. Things I still need to understand

I haven't yet fully worked out:

- What exact mathematical objective should our project use?
- Which metrics should have the highest priority?
- How should fairness be measured?
- How exactly should packet deadlines be represented?
- What constraints are actually required by our final system?
- How difficult does the optimization problem become as the number of nodes/resources increases?
- What heuristic algorithms would make good baselines?
- When would optimization be better than RL?
- When would RL actually provide an advantage?
- How will the reward function for RL be derived from the optimization objectives?
- How will the simulator calculate how much data is transmitted from an allocated resource?

These are things I should understand before designing the final RL environment.

---

# Key takeaway

> **The scheduling problem is: allocate limited communication resources among competing nodes so that overall network performance is good while respecting system constraints.**

And importantly:

> **RL should eventually be treated as a method for solving this decision problem, rather than as the definition of the problem itself.**