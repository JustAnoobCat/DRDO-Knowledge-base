## What am I trying to understand?

The scheduler cannot make a good decision unless it knows what is happening in the network.

So, before making an allocation decision, the ACN/scheduler needs some information about the current condition of the network.

This collection of information is called the **network state**.

A simple way to think about it is:

> The network state is a snapshot of what is happening in the communication system at a particular moment.

---

## 1. Why does the scheduler need the network state?

Imagine the ACN has 4 ground nodes.

At one moment:

```text
Node 1 → small queue
Node 2 → huge queue
Node 3 → strong channel
Node 4 → weak channel
```

A few milliseconds later, things may have changed:

```text
Node 1 → emergency data arrived
Node 2 → queue decreased
Node 3 → channel became weaker
Node 4 → channel improved
```

If the scheduler continues using the old allocation, it may make a poor decision.

Therefore:

```text
Observe current network
        ↓
Understand current state
        ↓
Make allocation decision
        ↓
Network changes
        ↓
Observe again
        ↓
Make next decision
```

---

## 2. What information can be part of the network state?

There are several useful categories of information.

### A. Queue information

The scheduler can know how much data is currently waiting at each node.

For example:

```text
Node 1 → 0.1 MB
Node 2 → 5 MB
Node 3 → 1 MB
Node 4 → 8 MB
```

This tells the scheduler which nodes have a large backlog.

---

### B. Channel information

The scheduler can know something about the quality of each communication link.

For example:

```text
Node 1 → SNR = 20 dB
Node 2 → SNR = 15 dB
Node 3 → SNR = 5 dB
Node 4 → SNR = 18 dB
```

This tells the scheduler that Node 3 currently has a much weaker channel.

Depending on the simulation, the state could contain SNR, SINR, channel gain, or another representation of channel quality.

---

### C. Delay information

The scheduler may also need to know how long packets have been waiting.

For example:

```text
Node 1 → oldest packet = 5 ms
Node 2 → oldest packet = 40 ms
Node 3 → oldest packet = 12 ms
Node 4 → oldest packet = 80 ms
```

This is important because a node with a relatively small queue could still have a packet that has been waiting for a long time.

---

### D. Traffic information

The scheduler can also consider what type of traffic is waiting.

For example:

```text
Node 1 → command/control
Node 2 → video
Node 3 → voice
Node 4 → sensor data
```

Different traffic types can have different requirements.

For example:

```text
Critical command → very low delay
Voice            → low delay / low jitter
Video            → high throughput
Bulk data        → can tolerate more delay
```

Therefore, knowing only the queue size may not be enough.

---

### E. Resource information

The scheduler also needs to know what resources are currently available.

For example:

```text
Available time resources
Available frequency resources
Available transmission power
```

It cannot allocate resources that do not exist.

---

## 3. A simple network-state example

Suppose I have four ground nodes.

The current state could look something like:

```text
Node 1:
Queue = 2 MB
SNR = 18 dB
Delay = 5 ms
Traffic = Video

Node 2:
Queue = 0.1 MB
SNR = 22 dB
Delay = 2 ms
Traffic = Command

Node 3:
Queue = 6 MB
SNR = 8 dB
Delay = 30 ms
Traffic = Video

Node 4:
Queue = 1 MB
SNR = 15 dB
Delay = 45 ms
Traffic = Voice
```

The scheduler can use this information to decide how to distribute resources.

---

## 4. The scheduler is not looking at just one factor

This is important.

It would be easy to create a simple rule such as:

> "Give more resources to the node with the biggest queue."

But that can produce bad decisions.

Suppose:

```text
Node A:
Queue = 10 MB
SNR = 20 dB

Node B:
Queue = 1 MB
SNR = 5 dB
Delay = 50 ms
Critical traffic
```

Node A has much more data.

But Node B has a critical packet that has already been waiting for a long time.

So the scheduler needs to consider multiple pieces of information together.

---

## 5. Network state as a vector

For a mathematical model or an AI system, the network state can eventually be represented as a collection of numbers.

For example:

```text
State =
[
    Queue sizes,
    SNR values,
    Packet delays,
    Traffic information,
    Resource information,
    ...
]
```

For example, for one node:

```text
Node 1:

Queue = 2 MB
SNR = 18 dB
Delay = 5 ms
Priority = High
```

The complete state contains information for all relevant nodes.

---

## 6. Why this matters for Reinforcement Learning

If we eventually use Reinforcement Learning, the network state becomes especially important.

The basic RL loop is:

```text
State
  ↓
RL Agent
  ↓
Action
  ↓
Environment
  ↓
New State
```

In our project:

```text
Current network state
        ↓
     RL Agent
        ↓
Resource allocation
        ↓
Communication simulator
        ↓
Queues/channel conditions change
        ↓
New network state
        ↓
RL Agent acts again
```

The agent learns to associate different network states with useful actions.

---

## 7. State is different from action

I need to keep these two concepts separate.

### State

What is happening **right now**?

Examples:

```text
Queue = 5 MB
SNR = 15 dB
Delay = 20 ms
```

### Action

What should the scheduler **do**?

Examples:

```text
Give Node 1 more time
Assign RBG 3 to Node 2
Increase power for Node 4
```

So:

```text
STATE
"What is happening?"
       ↓
     AGENT
"What should I do?"
       ↓
ACTION
"Allocate these resources"
```

---

## 8. State is different from reward

There is another important distinction.

### State

Describes the current situation.

### Action

Describes what the scheduler chooses.

### Reward

Describes how good or bad the result of that decision was.

For example:

```text
State:
Node 1 has a large queue
Node 2 has urgent data

       ↓

Action:
Give Node 2 immediate resources

       ↓

Result:
Urgent packet transmitted successfully

       ↓

Reward:
Positive because an important delay requirement was satisfied
```

So:

```text
State → What is happening?
Action → What should I do?
Reward → How good was the result?
```

These three ideas are fundamental if we use Reinforcement Learning later.

---

## 9. Network state changes continuously

The network is not static.

New packets can arrive.

Packets can be transmitted.

Packets can expire.

SNR can change.

Rain can appear.

A node can move.

Resources can be reassigned.

Therefore:

```text
t = 0 ms
    ↓
State S₀
    ↓
Action A₀
    ↓
Network evolves
    ↓
t = 10 ms
    ↓
State S₁
    ↓
Action A₁
    ↓
Network evolves
    ↓
t = 20 ms
    ↓
State S₂
```

The exact decision interval is a design parameter.

The original project description mentions decisions on the order of milliseconds, such as a 10 ms frame, but I should confirm the final timing model rather than treating 10 ms as mandatory.

---

## 10. Example: sudden traffic burst

Suppose a reconnaissance node suddenly starts sending a large amount of video data.

Before the burst:

```text
Recon Node:
Queue = 1 MB
```

After the burst:

```text
Recon Node:
Queue = 10 MB
```

The network state has changed.

A dynamic scheduler can observe this change and potentially allocate more resources to that node.

This is one example of why dynamic scheduling can outperform a fixed allocation.

---

## 11. Example: sudden channel degradation

Now suppose rain affects one ground node.

Before the rain:

```text
Node 3:
SNR = 18 dB
Queue = 2 MB
```

After the rain:

```text
Node 3:
SNR = 7 dB
Queue = 2 MB
```

The queue initially may not have changed, but the channel state has.

Because the achievable data rate can decrease, the queue may start growing if the scheduler does not adapt.

So the scheduler should be able to react to channel changes as well as traffic changes.

---

## 12. What should the state contain?

There is no single universal answer.

The state should contain the information that is useful for making scheduling decisions.

Possible information includes:

```text
Per-node:
- Queue size
- SNR/SINR
- Channel gain
- Packet delay
- Traffic class
- Priority
- Previous service

Network-wide:
- Available resources
- Current power usage
- Current throughput
- Resource utilization
- Other relevant conditions
```

But including everything is not automatically better.

A very large state can make the learning or optimization problem more difficult.

So the state should contain **useful information**, not every piece of information available.

---

## 13. The state should be observable

The scheduler can only make decisions using information that is available to it.

For example, if the scheduler cannot know the exact future traffic arrivals, it cannot directly use future queue sizes as part of the current state.

It can only use information available at the current decision point.

This is important because our simulation should avoid giving the AI information that would not realistically be available during actual operation.

---

## 14. Connection to the Digital Twin

The simulator will maintain the current state of the simulated network.

For example:

```text
Simulator
   │
   ├── Node queues
   ├── Channel conditions
   ├── Traffic
   ├── Delays
   ├── Resource usage
   └── Other system information
            ↓
        Network State
            ↓
        Scheduler
            ↓
      Allocation decision
            ↓
         Simulator
```

The simulator updates the state after every transmission/decision step.

This creates the feedback loop required for dynamic scheduling.

---

## What I understand now

The **network state** is the information describing the current condition of the communication system.

It can include:

- Queue sizes
- Channel quality
- Packet delays
- Traffic types
- Priorities
- Available resources
- Other useful network information

The scheduler uses the current state to decide how to allocate resources.

The state changes after traffic arrives, data is transmitted, or channel conditions change.

---

## Important distinction

```text
STATE
What is happening?

ACTION
What should the scheduler do?

REWARD
How good was the result?
```

These three concepts become especially important if I use Reinforcement Learning.

---

## The feedback loop

```text
        Current State
             ↓
         Scheduler
             ↓
           Action
             ↓
      Resource Allocation
             ↓
       Transmission
             ↓
     Network Changes
             ↓
         New State
             ↓
         Scheduler
             ↓
           ...
```

This is the basic interaction between the scheduler and the communication environment.

---

## Things I still need to understand

- What exactly should be included in our final state?
- How is queue size represented mathematically?
- What is Head-of-Line (HoL) delay?
- How are traffic priorities represented?
- How do we represent channel quality in the state?
- What information can realistically be obtained by the ACN?
- How large should the state be?
- What exactly is an RL action?
- What exactly is a reward?
- How do state, action, and reward work together during RL training?

---

## One-line understanding

> **The network state is a snapshot of the current communication situation, and the scheduler uses this information to decide how the limited resources should be allocated.**