## What am I trying to understand?

The ACN has a limited amount of communication capacity.

The scheduler's job is to decide how that capacity should be divided among the ground nodes.

To understand scheduling properly, I first need to understand **what exactly is being allocated**.

A useful way to think about the resources is:

- Time
- Frequency / bandwidth
- Power
- Link adaptation

These are different dimensions of the same overall resource-allocation problem.

---

## 1. Time

The first resource dimension is **time**.

The ACN cannot necessarily allow every node to use the channel whenever it wants.

Instead, communication can be divided into time intervals.

For example:

```text
Time →

| Node 1 | Node 2 | Node 3 | Node 4 |
```

Each node gets a certain amount of transmission time.

This is the basic idea behind **TDMA (Time Division Multiple Access)**.

If Node 1 gets more time, it has more opportunity to transmit.

If Node 2 gets less time, it has less opportunity to transmit.

---

## 2. Time slots

A fixed period of communication time can be divided into smaller units called **time slots**.

For example:

```text
One frame

| Slot 1 | Slot 2 | Slot 3 | Slot 4 |
| Node 1 | Node 2 | Node 3 | Node 4 |
```

A simple static scheduler could always give each node one slot.

But a dynamic scheduler could change the allocation:

```text
Frame 1:

| N1 | N2 | N3 | N4 |

Frame 2:

| N1 | N1 | N2 | N3 | N4 |

Frame 3:

| N2 | N2 | N3 | N4 |
```

The exact slot structure is a design choice.

The important idea is:

> The scheduler can change how much time each node receives depending on the current network state.

---

## 3. Frequency / Bandwidth

Time is not the only resource.

The available radio bandwidth can also be divided into smaller frequency regions.

For example:

```text
Total available bandwidth

| RBG 1 | RBG 2 | RBG 3 | RBG 4 | RBG 5 | RBG 6 |
```

Different nodes can potentially use different frequency resources at the same time.

For example:

```text
             Frequency →

| RBG1 | RBG2 | RBG3 | RBG4 | RBG5 | RBG6 |

   N1     N1     N2     N2     N3     N3
```

Now multiple nodes can transmit during the same time period because they are using different frequency resources.

This is different from pure time-only allocation.

---

## 4. What is an RBG?

RBG stands for **Resource Block Group**.

It is a group of smaller frequency-domain resource blocks that can be allocated by the scheduler.

For our simulation, I can think of an RBG as:

> A chunk of the available frequency resource that the scheduler can assign to a node.

For example:

```text
Available frequency resources:

RBG 1
RBG 2
RBG 3
RBG 4
RBG 5
RBG 6
RBG 7
RBG 8
```

The scheduler could decide:

```text
Node 1 → RBG 1, 2
Node 2 → RBG 3, 4, 5
Node 3 → RBG 6
Node 4 → RBG 7, 8
```

The exact number of RBGs is a simulation parameter and should not be treated as a fixed project requirement unless confirmed.

---

## 5. Time + Frequency together

This is where the problem becomes more interesting.

Instead of thinking only about:

> "How much time does Node 1 get?"

I can think about a **time-frequency resource grid**.

For example:

```text
                 Frequency
             → → → → → → →

Time       +------+------+------+------+
   ↓       | RBG1 | RBG2 | RBG3 | RBG4 |
            +------+------+------+------+
            | RBG1 | RBG2 | RBG3 | RBG4 |
            +------+------+------+------+
            | RBG1 | RBG2 | RBG3 | RBG4 |
            +------+------+------+------+
```

Each cell represents a small piece of the available communication resource.

The scheduler can decide which node receives which resources.

So instead of simply allocating:

```text
Node 1 → 25% of time
Node 2 → 25% of time
...
```

the scheduler could make a more detailed decision:

```text
Time 1 → N1 uses RBG 1 and 2
Time 1 → N2 uses RBG 3 and 4
Time 1 → N3 uses RBG 5
Time 1 → N4 uses RBG 6

Time 2 → N1 uses RBG 1
Time 2 → N2 uses RBG 2, 3, 4
...
```

This gives the scheduler more flexibility.

---

## 6. Power is another resource

The ACN also has a limited transmission power budget.

It cannot transmit at unlimited power to every node.

For example:

```text
Total available power = limited

             ACN
              │
       ┌──────┼──────┐
       ↓      ↓      ↓
      N1     N2     N3
     P1     P2     P3

P1 + P2 + P3 ≤ Total Power
```

The scheduler may therefore need to decide not only:

> "Which node gets this resource?"

but potentially:

> "How much power should be used for this node?"

---

## 7. Why would power allocation matter?

Suppose Node 1 has a very strong channel.

Node 2 has a weak channel because of rain or another propagation problem.

Giving extra power to Node 2 might improve its transmission capability.

However, that power has to come from somewhere.

If the ACN has a fixed power budget:

```text
More power for Node 2
        ↓
Less power available elsewhere
```

So power allocation can become another optimization problem.

---

## 8. Link adaptation / MCS

There is another related decision: **how efficiently to transmit over the current channel**.

This is where **MCS (Modulation and Coding Scheme)** becomes relevant.

The basic idea is:

```text
Good channel
     ↓
Can potentially use a more efficient MCS
     ↓
Higher data rate

Poor channel
     ↓
May need a more robust MCS
     ↓
Lower data rate
```

So even after deciding how much time/frequency/power a node gets, the amount of data actually transmitted can depend on the channel quality.

---

## 9. Putting the resources together

The scheduler may therefore have several things it could control.

```text
                    Scheduler
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
      Time          Frequency         Power
        │               │               │
   Time slots          RBGs          Power level
        │               │               │
        └───────────────┼───────────────┘
                        ↓
                 Transmission
                        ↓
                 Data delivered
```

MCS/link adaptation can also affect how efficiently those resources are used.

---

## 10. Why this is more complicated than simple TDMA

If I only use static TDMA, the decision might be:

> "Node 1 gets 25% of the time, Node 2 gets 25%, etc."

This is relatively simple.

But a more advanced scheduler might have to decide:

```text
Which node?
     +
How much time?
     +
Which frequency resources?
     +
How much power?
     +
Which MCS?
```

That creates a much larger decision space.

This is why I should not immediately assume that the most complicated resource-allocation method is the best one.

---

## 11. Example

Suppose the ACN has four nodes.

### Node conditions

```text
Node 1 → large queue, strong channel
Node 2 → small queue, urgent data
Node 3 → large queue, weak channel
Node 4 → moderate queue, good channel
```

A simple equal scheduler might do:

```text
N1 → equal resources
N2 → equal resources
N3 → equal resources
N4 → equal resources
```

A smarter scheduler could potentially decide:

```text
N1 → more frequency/time resources
N2 → immediate resources because of urgency
N3 → resources adjusted according to weak channel
N4 → remaining resources
```

If power allocation is also included, it could additionally adjust transmission power.

The exact decision would depend on the scheduling algorithm and its objective.

---

## 12. What does this mean for our project?

This gives me a more complete picture of the problem.

The project may not actually be limited to:

> "Which node gets the next TDMA slot?"

It could be a broader **resource allocation problem**.

The possible resource dimensions are:

```text
Time
Frequency
Power
Link adaptation
```

However, this does **not** mean that our final project must implement all of them.

The project requirements, available computational resources, simulation complexity, and mentor's guidance should determine how much of this we actually model.

---

## 13. Important distinction: resource vs scheduler

I should not confuse these two things.

### Resource

Something limited that can be allocated.

Examples:

- Transmission time
- Frequency bandwidth
- Power

### Scheduler

The mechanism that decides how those resources are allocated.

For example:

```text
Available resources
        ↓
     Scheduler
        ↓
Allocation decision
        ↓
Node 1 → resources
Node 2 → resources
Node 3 → resources
...
```

The scheduler is therefore the **decision maker**, while the time/frequency/power resources are what it is deciding about.

---

## 14. Connection to the Digital Twin

The Digital Twin needs to represent the available resources so that different scheduling algorithms can be tested.

For example:

```text
Digital Twin

ACN
 │
 ├── Total bandwidth
 ├── Available frequency resources
 ├── Available transmission power
 ├── Time/frame structure
 │
 └── Ground nodes
       ├── Queue
       ├── Channel quality
       ├── Traffic
       └── Priority
```

The scheduler interacts with this simulated environment.

It receives the current network state and makes an allocation.

The simulator then calculates what happens as a result.

---

## What I understand now

The communication resources are not necessarily just TDMA time slots.

The resource-allocation problem can involve multiple dimensions:

1. **Time** — when a node is allowed to transmit.
2. **Frequency** — which portion of the available bandwidth it can use.
3. **Power** — how much transmission power is assigned.
4. **Link adaptation/MCS** — how efficiently the available resources can be used according to channel quality.

The scheduler decides how these resources should be distributed.

---

## Important relationship

```text
Limited resources
       ↓
Multiple competing nodes
       ↓
Scheduler
       ↓
Resource allocation
       ↓
Transmission
       ↓
Data delivered
       ↓
Queues / delay / performance change
       ↓
New network state
       ↓
Scheduler makes another decision
```

This is the basic feedback loop behind the scheduling problem.

---

## What I still need to understand

- What exactly is TDMA?
- What exactly is a time slot?
- How is bandwidth divided into frequency resources?
- What exactly is an RBG?
- How are time and frequency resources normally represented?
- How does allocated bandwidth affect data rate?
- How does transmission power affect SNR/SINR?
- What exactly is MCS?
- Should our project allocate only time, or also frequency and power?
- How complicated should our simulator be?
- What resource dimensions does my mentor actually expect us to model?

---

## One-line understanding

> **The ACN has a limited pool of communication resources—such as time, frequency, and power—and the scheduler must decide how to distribute those resources among the ground nodes.**