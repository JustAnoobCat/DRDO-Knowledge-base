## What am I trying to understand?

TDMA stands for **Time Division Multiple Access**.

It is a way of allowing multiple users/nodes to share the same communication channel by dividing the available transmission time between them.

The basic idea is:

> Multiple nodes share the same communication resource, but they transmit during different time periods.

---

## 1. The basic idea

Imagine there are four ground nodes:

```text
Node 1
Node 2
Node 3
Node 4
```

Instead of allowing all of them to transmit at the same time, the available time can be divided:

```text
Time →

| Node 1 | Node 2 | Node 3 | Node 4 |
```

Node 1 transmits during its allocated period.

Then Node 2.

Then Node 3.

Then Node 4.

This sequence can repeat.

---

## 2. What is a time slot?

A **time slot** is a small period of time assigned to a particular node for transmission.

For example:

```text
One frame

| Slot 1 | Slot 2 | Slot 3 | Slot 4 |
|   N1   |   N2   |   N3   |   N4   |
```

Each slot has a certain duration.

If the slots are equal:

```text
N1 → 2.5 ms
N2 → 2.5 ms
N3 → 2.5 ms
N4 → 2.5 ms
```

then each node receives the same amount of transmission time.

---

## 3. What is a frame?

A **frame** is a larger repeating time structure containing multiple time slots.

For example:

```text
Frame
┌────────┬────────┬────────┬────────┐
│ Slot 1 │ Slot 2 │ Slot 3 │ Slot 4 │
│   N1   │   N2   │   N3   │   N4   │
└────────┴────────┴────────┴────────┘
```

After the frame finishes, another frame can begin:

```text
Frame 1 → Frame 2 → Frame 3 → ...
```

In our project, the scheduler can potentially make a new allocation for each frame or scheduling interval.

The exact frame duration is a simulation/design parameter.

---

## 4. Why use TDMA?

The main advantage is that multiple nodes can share the same communication channel in an organized way.

Instead of everyone transmitting whenever they want, the scheduler determines who gets access at a particular time.

This helps avoid users transmitting over each other when they are sharing the same time resource.

---

## 5. Static TDMA

The simplest version is **static TDMA**.

The allocation is predetermined.

For example:

```text
Node 1 → 25%
Node 2 → 25%
Node 3 → 25%
Node 4 → 25%
```

Every frame follows the same pattern:

```text
Frame 1:
| N1 | N2 | N3 | N4 |

Frame 2:
| N1 | N2 | N3 | N4 |

Frame 3:
| N1 | N2 | N3 | N4 |
```

This is very easy to implement.

---

## 6. The problem with static TDMA

The problem is that the network is not always the same.

Suppose:

```text
Node 1 → almost no data
Node 2 → huge video queue
Node 3 → emergency command
Node 4 → moderate traffic
```

But the static TDMA schedule remains:

```text
| N1 | N2 | N3 | N4 |
```

Node 1 may not have enough data to use its entire slot.

Node 2 may need more resources.

Node 3 may have urgent data.

Yet the schedule does not automatically change.

This can lead to inefficient resource utilization.

---

## 7. Dynamic TDMA

Instead of keeping the same allocation, the scheduler can change the slot allocation according to the current network state.

For example:

### Frame 1

```text
| N1 | N2 | N3 | N4 |
```

### Network changes

Node 2 receives a large amount of video data.

### Frame 2

```text
| N1 | N2 | N2 | N3 | N4 |
```

Now Node 2 receives more transmission time.

Later, Node 2's queue may decrease:

### Frame 3

```text
| N1 | N2 | N3 | N4 |
```

The allocation can adapt again.

---

## 8. TDMA and our project

The original project idea focuses heavily on dynamically changing TDMA allocations.

The basic concept is:

```text
Observe network
      ↓
Check queues
      ↓
Check channel conditions
      ↓
Decide slot allocation
      ↓
Transmit
      ↓
Network changes
      ↓
Decide again
```

This is why TDMA is a useful starting point for the project.

However, I should remember that **TDMA does not have to be the final scheduling approach**.

The broader problem is resource allocation.

TDMA is one possible mechanism for dividing the time resource.

---

## 9. Why equal TDMA allocation can waste resources

Imagine a frame has 100 units of available time.

There are four nodes.

Static TDMA might give:

```text
N1 → 25 units
N2 → 25 units
N3 → 25 units
N4 → 25 units
```

Now suppose:

```text
N1 → 2 units of data
N2 → 40 units of data
N3 → 10 units of data
N4 → 40 units of data
```

N1 may not need all 25 units.

Some of the available communication opportunity can therefore be wasted.

A dynamic scheduler could potentially reallocate some of N1's unused capacity to other nodes.

---

## 10. But queue size alone is not enough

Suppose:

```text
N1 → 20 MB video
N2 → 100 KB emergency command
```

If I only look at queue size, I might give more resources to N1.

But the emergency command may have a strict delay requirement.

Therefore, the scheduler may need to consider:

- Queue size
- Delay
- Traffic priority
- Channel quality
- Fairness
- Other QoS requirements

This is why simply making TDMA dynamic is not automatically enough.

---

## 11. TDMA and channel quality

Suppose:

```text
Node 1 → SNR = 20 dB
Node 2 → SNR = 5 dB
```

The two nodes may not be able to transmit the same amount of data during equal-sized slots.

Node 1 may have a much higher achievable rate.

Therefore:

```text
Same time allocation
        ≠
Same amount of data transmitted
```

This is an important point.

The scheduler may need to consider channel quality when deciding allocations.

---

## 12. A simple example

Suppose each node receives 10 ms.

```text
N1 → 10 ms
N2 → 10 ms
N3 → 10 ms
N4 → 10 ms
```

Assume:

```text
N1 → good channel
N2 → good channel
N3 → poor channel
N4 → good channel
```

Even though all nodes receive the same time, the amount of data they can transmit may be different.

Therefore, equal time does not necessarily mean equal throughput.

---

## 13. Dynamic slot sizing

One idea is to allow the slot durations themselves to change.

Instead of:

```text
N1 → 25%
N2 → 25%
N3 → 25%
N4 → 25%
```

the scheduler could choose:

```text
N1 → 10%
N2 → 40%
N3 → 20%
N4 → 30%
```

The total still needs to satisfy the available time:

```text
10% + 40% + 20% + 30% = 100%
```

This is closer to the original idea of an AI-enhanced dynamic TDMA scheduler.

---

## 14. TDMA is mainly a time-domain concept

TDMA focuses on dividing the **time dimension**.

However, our broader resource-allocation problem can also involve:

```text
Time
Frequency
Power
Link adaptation
```

Therefore:

> TDMA should be viewed as one part of the overall resource-allocation problem, not necessarily the complete solution.

---

## 15. TDMA vs frequency allocation

With pure time-based allocation:

```text
Time →

| N1 | N2 | N3 | N4 |
```

With frequency-based allocation, multiple nodes can potentially transmit during the same time period using different frequency resources:

```text
Frequency →

| N1 | N2 | N3 | N4 |
```

The two dimensions can also be combined:

```text
             Frequency
          → → → → → → →

Time ↓    | N1 | N1 | N2 | N2 |
           | N3 | N3 | N4 | N4 |
           | N1 | N2 | N3 | N4 |
```

This gives the scheduler more flexibility, but also makes the problem more complicated.

---

## 16. Why TDMA is still a useful baseline

Even if TDMA is not our final solution, it is useful because it gives us something simple to compare against.

For example:

```text
Static TDMA
     ↓
Measure performance
     ↓
Dynamic scheduler
     ↓
Measure performance
     ↓
RL scheduler
     ↓
Measure performance
```

Then we can ask:

> Does the more advanced method actually improve the network?

Without a baseline, it is difficult to know whether our AI provides a meaningful improvement.

---

## 17. TDMA in the Digital Twin

The simulator can represent a frame and its time slots.

For example:

```text
Simulation frame

| Slot 1 | Slot 2 | Slot 3 | Slot 4 |
|   N1   |   N2   |   N3   |   N4   |
```

During each slot, the simulator can calculate:

- Which node is transmitting
- How much data it has waiting
- Channel quality
- Achievable data rate
- Amount of data transmitted
- Remaining queue
- Packet delay
- Packet drops

Then the next frame can use the same or a different allocation.

---

## 18. TDMA and the AI scheduler

If we use reinforcement learning later, the agent could receive a state such as:

```text
Node 1 → Queue, SNR, Delay
Node 2 → Queue, SNR, Delay
Node 3 → Queue, SNR, Delay
Node 4 → Queue, SNR, Delay
```

and produce an action such as:

```text
N1 → 10% time
N2 → 40% time
N3 → 20% time
N4 → 30% time
```

The simulator then evaluates the result.

The agent receives a reward based on the resulting performance.

---

## What I understand now

TDMA is a method for allowing multiple nodes to share communication resources by dividing the **time** between them.

A basic TDMA system uses predetermined time slots.

Static TDMA is simple but does not adapt to changing network conditions.

Dynamic TDMA can change slot allocations based on the current network state.

However, dynamic TDMA is still only one possible solution to the broader resource-allocation problem.

---

## Important things I learned

### Time slot

A period of time assigned for transmission.

### Frame

A repeating structure containing multiple time slots.

### Static TDMA

The allocation stays fixed or mostly predetermined.

### Dynamic TDMA

The allocation can change according to the network state.

### Important realization

```text
Equal time
    ≠
Equal data transmitted
```

because different nodes can have different channel conditions.

Also:

```text
More data
    ≠
Automatically more priority
```

because delay and traffic importance can matter.

---

## What I still need to understand

- How is a TDMA frame actually structured?
- How short should the scheduling interval be?
- How are guard times handled?
- How does TDMA interact with TDD?
- How do we calculate the amount of data transmitted during a slot?
- How should dynamic slot sizes be represented?
- Should our project remain time-only or also include frequency resources?
- What scheduling algorithm should control the TDMA allocation?
- How should fairness be handled?
- How should urgent traffic be prioritized?

---

## One-line understanding

> **TDMA divides communication time among multiple nodes, while dynamic TDMA allows the scheduler to change those time allocations according to the current network situation.**