## What am I trying to understand?

The ACN has limited communication resources, and multiple ground nodes want to use them.

The **scheduler** is the part of the system that decides how those resources should be distributed.

In simple terms:

> The scheduler is the decision-maker that decides who gets communication resources, how much they get, and potentially when and where they get them.

---

## 1. Why do we need a scheduler?

Imagine the ACN has 16 ground nodes.

At one moment:

```text
Node 1  → very little data
Node 2  → large video queue
Node 3  → emergency command
Node 4  → weak channel
...
Node 16 → moderate traffic
```

The ACN cannot give everyone unlimited resources.

Someone has to decide:

```text
Who gets resources?
How much?
When?
Which resources?
Why?
```

That is the scheduler's job.

---

## 2. Static vs dynamic scheduling

There are two broad ways to think about scheduling.

### Static scheduling

The allocation is decided beforehand and does not change much.

For example:

```text
Node 1 → 10%
Node 2 → 10%
Node 3 → 10%
...
Node 10 → 10%
```

This is simple and predictable.

But it does not react well when network conditions change.

---

### Dynamic scheduling

The scheduler looks at the current network state and changes the allocation.

For example:

```text
Current state
     ↓
Node 1 → small queue
Node 2 → huge queue
Node 3 → urgent data
Node 4 → weak channel
     ↓
Scheduler
     ↓
New allocation
```

A few milliseconds later, the state may be different, so the allocation can also change.

This is much closer to what we want to investigate in our project.

---

## 3. What can the scheduler consider?

The scheduler can potentially consider:

### Queue size

How much data is waiting?

```text
Large queue → potentially needs more resources
Small queue → potentially needs fewer resources
```

### Channel quality

How good is the communication link?

```text
Good channel → resources can be used efficiently
Poor channel → transmission may be less efficient
```

### Delay

How long have packets been waiting?

```text
Long delay → packet may need urgent service
```

### Traffic priority

What type of data is waiting?

```text
Critical command → high priority
Voice           → delay-sensitive
Video           → throughput-heavy
Bulk data       → less urgent
```

### Previous service

It may also be useful to know how much service a node has recently received.

This can help prevent one node from continuously receiving resources while others are neglected.

---

## 4. Scheduler decision

The exact decision depends on how the system is designed.

A scheduler could decide:

```text
Time allocation
Frequency allocation
Power allocation
```

or some combination of them.

For a time-only system, an action could look like:

```text
Node 1 → 20% time
Node 2 → 40% time
Node 3 → 30% time
Node 4 → 10% time
```

For a frequency-aware system, it could instead assign frequency resources:

```text
RBG 1,2 → Node 1
RBG 3,4,5 → Node 2
RBG 6 → Node 3
RBG 7,8 → Node 4
```

A more advanced scheduler could decide multiple dimensions together.

---

# 5. Different scheduling approaches

There is no single scheduler that is always best.

We can start with a simple scheduler and progressively test more intelligent approaches.

A useful progression is:

```text
Static TDMA / Round Robin
          ↓
Rule-based / Heuristic
          ↓
Optimization-based
          ↓
Reinforcement Learning
```

The purpose of this progression is not necessarily to use every algorithm.

It allows us to compare different levels of complexity and performance.

---

## 6. Static TDMA

TDMA means **Time Division Multiple Access**.

A basic TDMA scheduler divides communication time between users.

For example:

```text
Frame
┌──────┬──────┬──────┬──────┐
│ N1   │ N2   │ N3   │ N4   │
└──────┴──────┴──────┴──────┘
```

Each node gets a predetermined portion of time.

This is a useful **baseline** because it is simple.

However, it does not automatically react to:

- Queue size
- Traffic changes
- Channel quality
- Urgency

---

## 7. Round Robin

Round Robin is another simple scheduling approach.

The scheduler gives users resources in turn.

For example:

```text
N1 → N2 → N3 → N4 → N1 → N2 → N3 → N4
```

The main advantage is fairness and simplicity.

The disadvantage is that every node may get service even when it does not currently need it.

For example:

```text
N1 → no data
N2 → no data
N3 → huge queue
N4 → urgent packet
```

A simple Round Robin scheduler may still continue rotating through all nodes.

---

## 8. Heuristic scheduling

A heuristic scheduler uses manually designed rules.

For example:

```text
If queue is large
    → increase priority

If packet is close to deadline
    → increase priority

If channel is good
    → increase priority
```

The scheduler could combine these factors into a score.

For example:

```text
Priority Score =
    Queue importance
    +
    Delay importance
    +
    Channel quality
```

Then the scheduler allocates resources to nodes with higher scores.

---

## 9. Why heuristics are useful

A heuristic is relatively easy to understand and implement.

I can explicitly see why the scheduler made a decision.

For example:

```text
Node 1 → large queue + good channel
Node 2 → small queue + urgent packet
Node 3 → small queue + poor channel
```

The scheduler can calculate a score for each node and rank them.

This makes a heuristic useful as an intermediate baseline between simple static scheduling and a more complex AI approach.

---

## 10. The problem with heuristics

The rules have to be designed manually.

For example, I might decide:

```text
Queue weight = 0.5
Delay weight = 0.3
SNR weight = 0.2
```

But why should the weights be exactly those values?

Changing the weights can change the scheduler's behaviour.

This creates a tuning problem.

A heuristic can work well, but its performance depends heavily on how the rules are designed.

---

# 11. Proportional Fair scheduling

Proportional Fair (PF) scheduling tries to balance:

- Current channel quality
- Long-term fairness

The basic idea is:

> Give more resources to users that can currently transmit efficiently, while also considering how much service they have received previously.

For example:

```text
Node A → excellent channel
Node B → average channel
Node C → excellent channel but has already received lots of resources
```

PF does not simply keep selecting Node A.

It considers previous service so that other nodes also get opportunities.

This makes it more balanced than simply selecting the node with the strongest channel.

---

## 12. Delay-aware scheduling

Some scheduling algorithms also consider packet delay.

A useful example is **M-LWDF**.

M-LWDF stands for:

**Modified Largest Weighted Delay First**

The basic idea is that packets that have been waiting for a long time become more important.

So instead of looking only at:

```text
How much data is waiting?
```

the scheduler can also ask:

```text
How long has the data been waiting?
```

This is particularly useful when delay-sensitive traffic exists.

---

## 13. Why delay awareness matters

Imagine:

```text
Node 1 → 10 MB video queue
Node 2 → 100 KB critical command
```

Node 1 has much more data.

But suppose:

```text
Node 1 → packet delay = 5 ms
Node 2 → packet delay = 18 ms
```

If Node 2 has a strict deadline, it may deserve immediate service despite having a much smaller queue.

This is why queue size alone is not enough.

---

# 14. Optimization-based scheduling

Instead of manually creating rules, I can formulate scheduling as an optimization problem.

For example, I might want to:

```text
Maximize:
    Throughput

while minimizing:
    Delay
    Packet drops

while maintaining:
    Fairness
    Resource constraints
```

The scheduler then tries to find the allocation that gives the best overall result.

Conceptually:

```text
Network state
     ↓
Optimization problem
     ↓
Find good allocation
     ↓
Apply allocation
```

---

## 15. Why optimization can be powerful

Optimization gives me a mathematical way to express what "good scheduling" means.

For example:

```text
High throughput       → good
Low delay             → good
Low packet drops      → good
Good fairness         → good
Exceeding power limit → not allowed
```

These can be represented mathematically using an objective function and constraints.

---

## 16. Why optimization can be difficult

The problem can become complicated when there are many nodes and many resource dimensions.

For example, if I simultaneously decide:

```text
Which node?
+
Which time?
+
Which frequency?
+
How much power?
+
Which MCS?
```

the number of possible combinations can become very large.

Finding the best solution repeatedly in real time can therefore become computationally expensive.

This is one reason why simpler algorithms and learning-based approaches are worth investigating.

---

# 17. Reinforcement Learning

Reinforcement Learning (RL) approaches the problem differently.

Instead of manually specifying every scheduling rule, an **agent learns from interaction with the environment**.

The basic loop is:

```text
State
  ↓
Agent
  ↓
Action
  ↓
Environment
  ↓
Reward
  ↓
New State
  ↓
Agent
```

For our project:

```text
Network state
     ↓
RL scheduler
     ↓
Resource allocation
     ↓
Communication simulator
     ↓
Network performance
     ↓
Reward
     ↓
New network state
```

---

## 18. What does the RL agent learn?

The agent tries to learn:

> Given the current network state, which action is likely to produce a good result?

For example:

```text
State:

Node 1 → large queue
Node 2 → urgent packet
Node 3 → weak channel
Node 4 → small queue

        ↓

Agent

        ↓

Action:

Give resources to Node 2 first
Give additional resources to Node 1
Avoid wasting resources on Node 4
```

The agent does not need to be explicitly told every possible situation.

It learns through repeated interaction with the simulator.

---

# 19. Reward

The RL agent needs a way to know whether its decisions were good.

This is done using a **reward**.

For example:

```text
Higher throughput
      → positive

Lower delay
      → positive

Fewer packet drops
      → positive

Better fairness
      → positive

Violating important constraints
      → negative
```

A simplified reward could therefore combine several objectives.

Conceptually:

```text
Reward =
    Throughput benefit
    - Delay penalty
    - Drop penalty
    + Fairness benefit
```

The exact reward function is something I need to design carefully later.

---

# 20. Why RL is interesting for this project

The network is dynamic.

The relationship between:

```text
Queue
Channel
Traffic
Resources
Delay
Throughput
```

can become complicated.

A trained RL agent may be able to learn a policy that adapts to these changing conditions.

This is potentially more flexible than a fixed rule.

However, RL is not automatically better.

---

# 21. Why RL may not be the best choice automatically

RL introduces additional problems:

- Training can take significant time.
- The reward function must be designed carefully.
- Poor training can produce poor decisions.
- The agent may encounter situations it did not see during training.
- The model itself requires computational resources.
- Debugging can be harder than debugging a simple rule.

Therefore:

> I should compare RL against simpler approaches rather than assuming that using AI automatically gives the best result.

---

# 22. What should we compare?

A sensible experimental approach could be:

```text
Baseline
   ↓
Static TDMA / Round Robin

        vs

Heuristic
   ↓
Dynamic rule-based scheduler

        vs

Optimization
   ↓
Mathematical resource allocation

        vs

RL / DRL
   ↓
Learned scheduler
```

The final set of algorithms should be decided after understanding the actual resource model and project constraints.

I do not need to implement every possible algorithm.

---

# 23. Performance comparison

The scheduler should be evaluated using measurable metrics.

Possible metrics include:

### Throughput

How much data is successfully transmitted.

### Average delay

How long packets take to be served.

### Packet drop rate

How much data is lost or dropped.

### Fairness

How evenly resources/service are distributed between nodes.

### Resource utilization

How much of the available communication capacity is actually being used.

### Computational cost

How much time and computing power the scheduler needs to make decisions.

---

# 24. Why computational cost matters

A scheduler that gives slightly better performance but requires enormous computational resources may not be practical.

For our project, this is particularly important because the project is not just about achieving the highest possible score.

I also need to consider:

- Available hardware
- Training time
- Inference time
- Simulator complexity
- Implementation complexity

So the real comparison should be something like:

```text
Performance
      vs
Complexity
      vs
Computational cost
```

---

# 25. The scheduler as a feedback system

The complete idea can now be represented as:

```text
              ┌─────────────────┐
              │  Network State  │
              └────────┬────────┘
                       ↓
              ┌─────────────────┐
              │    Scheduler    │
              └────────┬────────┘
                       ↓
                 Allocation
                       ↓
              ┌─────────────────┐
              │ Communication   │
              │    System       │
              └────────┬────────┘
                       ↓
              Data transmitted
                       ↓
             Queues / channels
                 change
                       ↓
              New network state
                       │
                       └──────────→ Scheduler
```

This is the fundamental loop I need to understand before implementing the AI.

---

## What I understand now

The scheduler is the decision-making component of the communication system.

Its job is to distribute limited resources among multiple nodes.

There are many possible ways to implement it:

- Static TDMA
- Round Robin
- Heuristic scheduling
- Proportional Fair
- Delay-aware scheduling such as M-LWDF
- Optimization
- Reinforcement Learning

Each method has different trade-offs.

A more complicated algorithm is not automatically a better algorithm.

The important question is:

> Which scheduling approach provides a good balance between throughput, delay, packet drops, fairness, adaptability, and computational cost?

---

## Important distinction

```text
Resource
→ What is being allocated?

Scheduler
→ Who gets the resources and how?

State
→ What is happening in the network?

Action
→ What allocation does the scheduler choose?

Reward
→ How good was the resulting decision?
```

These concepts should not be mixed together.

---

## Things I still need to understand

- TDMA in more detail
- Round Robin
- Proportional Fair
- M-LWDF
- Lyapunov-based scheduling
- Optimization formulation
- Reinforcement Learning
- DRL
- PPO
- How to represent the scheduler's action
- How to design the reward function
- How to compare algorithms fairly
- Which scheduling approach is realistic for our available hardware

---

## One-line understanding

> **The scheduler continuously observes the network state and decides how to allocate limited communication resources, with the goal of achieving good overall network performance.**