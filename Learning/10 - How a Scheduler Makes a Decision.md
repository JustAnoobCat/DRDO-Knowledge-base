## Why I am learning this

I understand what the scheduling problem is and I have seen different baseline approaches.

Now I want to understand **what actually happens during one scheduling decision**.

The important question is:

> Given the current network state, how does the scheduler decide which node gets which resources?

---

# 1. A scheduler operates repeatedly

The scheduler does not make one decision for the entire simulation.

The network keeps changing.

For example:

```text
Time 0 ms
    ↓
Make scheduling decision
    ↓
Transmit
    ↓
Time 10 ms
    ↓
Network state changed
    ↓
Make another decision
    ↓
Transmit
    ↓
Time 20 ms
    ↓
...
```

So scheduling is a **repeated decision-making process**.

This is important for the later RL part.

---

# 2. Decision interval

A scheduler needs some point at which it makes a new decision.

This can be called a:

- scheduling interval,
- decision interval,
- scheduling epoch,
- frame interval,

depending on the system design.

The original project description gives an example of making decisions every **10 ms**.

For now, I understand this as:

```text
Every 10 ms:
    observe network
    decide allocation
    apply allocation
```

The exact implementation still needs to be confirmed.

---

# 3. What happens at one decision point?

Suppose the current time is:

```text
t = 100 ms
```

The scheduler observes the current network state.

For example:

```text
Node     Queue     SNR       Delay
------------------------------------
A        80        Good      12 ms
B        20        Excellent 5 ms
C        100       Poor      18 ms
D        0         Good      -
```

The scheduler then decides how to distribute the available resources.

---

# 4. Observation → Decision → Result

The basic process is:

```text
Current state
      ↓
Scheduler
      ↓
Resource allocation
      ↓
Network transmission
      ↓
New state
```

For example:

```text
Queue + SNR + Delay
        ↓
    Scheduler
        ↓
A → 3 ms
B → 2 ms
C → 4 ms
D → 1 ms
        ↓
   Transmission
        ↓
Queues change
```

The next decision uses the **new** state.

---

# 5. Why the new state changes

During a scheduling interval, several things can happen.

### New packets arrive

```text
Queue = 50
     ↓
new packets arrive
     ↓
Queue = 70
```

### Packets are transmitted

```text
Queue = 70
     ↓
20 packets served
     ↓
Queue = 50
```

### Packets can expire

```text
Packet waits too long
     ↓
deadline exceeded
     ↓
packet dropped
```

### Channel conditions can change

```text
Good SNR
   ↓
rain / obstruction / fading
   ↓
lower SNR
```

Therefore the next scheduling decision may be completely different.

---

# 6. Resource allocation is the scheduler's action

I previously learned:

```text
State  = what is happening?
Action = what should be done?
Reward = how good was the result?
```

Now I can connect this to an actual scheduling step.

### State

```text
Queue sizes
SNR/SINR
Delay
Traffic information
Available resources
...
```

### Action

```text
Allocation of resources
```

For example:

```text
A → 2 ms
B → 5 ms
C → 3 ms
D → 0 ms
```

### Result

The simulator applies this allocation.

Then:

```text
Queues change
Packets are transmitted
Delays change
Drops may occur
```

---

# 7. The action depends on the resource model

This is an important distinction.

If the project only models time allocation, an action might look like:

```text
A → 2 ms
B → 4 ms
C → 3 ms
D → 1 ms
```

If I later model frequency resources as well, the action could become something like:

```text
A → frequency blocks 1–3
B → frequency blocks 4–6
C → frequency blocks 7–8
```

If power is also controlled, the decision could include:

```text
Node A → resources + power level
Node B → resources + power level
```

Therefore:

> **The exact action space depends on which resource dimensions the final simulator includes.**

I should not assume that the final project must control time, frequency, and power simultaneously.

---

# 8. Example: Time-only scheduler

For now, imagine a simplified system.

```text
Frame = 10 ms

Nodes:
A, B, C, D
```

The scheduler chooses:

```text
A → 1 ms
B → 4 ms
C → 3 ms
D → 2 ms
```

Check:

```text
1 + 4 + 3 + 2 = 10 ms
```

So the allocation fits inside the frame.

The simulator then calculates how much data each node can actually transmit.

---

# 9. Allocated time is not the same as transmitted data

Suppose:

```text
A → 4 ms
B → 4 ms
```

It does not necessarily mean:

```text
A transmits the same amount of data as B
```

because their channel conditions may differ.

For example:

```text
A → high achievable rate
B → low achievable rate
```

Then:

```text
4 ms × high rate → more data
4 ms × low rate  → less data
```

This is why the scheduler may want channel information when deciding allocations.

---

# 10. A scheduler can be represented as a rule

A traditional scheduler can often be thought of as:

```text
State
  ↓
Scheduling metric
  ↓
Rank nodes
  ↓
Allocate resources
```

For example, a queue-based scheduler could calculate:

```text
metric = queue size
```

Then:

```text
largest queue
      ↓
highest priority
```

A delay-aware scheduler could use:

```text
metric = delay-related value
```

A proportional-fair scheduler uses both current rate and historical service information.

---

# 11. The scheduler doesn't necessarily need to calculate the perfect answer

This is an important distinction from the optimization module.

An optimization formulation might ask:

> What is the mathematically best allocation?

But a practical scheduler may instead use a rule such as:

```text
Calculate a score for each node
        ↓
Rank nodes
        ↓
Allocate resources according to ranking
```

This can be much faster.

The trade-off is:

```text
Exact optimization
    ↕
Computational cost

Simple heuristic
    ↕
Potentially less optimal
```

This trade-off is one of the reasons I need to compare different approaches.

---

# 12. Where RL fits

An RL scheduler changes the middle of the process.

Traditional scheduler:

```text
State
  ↓
Hand-designed rule
  ↓
Allocation
```

RL scheduler:

```text
State
  ↓
Learned policy
  ↓
Allocation
```

The surrounding simulator can remain conceptually the same.

That means I can potentially use the **same digital-twin environment** to test different scheduling strategies.

---

# 13. The simulator and scheduler should be conceptually separate

I think this is an important architectural idea.

The simulator represents:

```text
Network behavior
```

The scheduler represents:

```text
Resource-allocation decision
```

So:

```text
          DIGITAL TWIN / SIMULATOR
                    │
                    │ current state
                    ↓
               SCHEDULER
                    │
                    │ allocation
                    ↓
          DIGITAL TWIN / SIMULATOR
                    │
                    ↓
              New network state
```

This separation means I can replace:

```text
Round Robin
```

with:

```text
Heuristic
```

or:

```text
RL
```

without rebuilding the entire network simulator.

---

# 14. Why this separation matters

Suppose I build the simulator first.

Then I can test:

```text
Simulator + Static TDMA
```

Later:

```text
Simulator + Heuristic
```

Later:

```text
Simulator + RL
```

This makes comparisons much easier.

It also makes the project more modular.

---

# 15. One complete example

Suppose:

```text
Frame duration = 10 ms

Current state:

A → queue 100, good SNR
B → queue 20, excellent SNR
C → queue 60, poor SNR
D → queue 0
```

A scheduler might decide:

```text
A → 4 ms
B → 2 ms
C → 4 ms
D → 0 ms
```

Then the simulator calculates:

```text
A:
4 ms × achievable rate
      ↓
some packets transmitted

B:
2 ms × achievable rate
      ↓
some packets transmitted

C:
4 ms × achievable rate
      ↓
some packets transmitted

D:
0 ms
      ↓
no transmission
```

After this:

```text
Queues change
Delays change
Packets may arrive
Channel conditions may change
```

Then the next decision is made.

---

# 16. The scheduler is therefore a feedback loop

The whole system can be viewed as:

```text
       ┌─────────────────────┐
       │                     │
       ↓                     │
Observe network              │
       ↓                     │
Make allocation              │
       ↓                     │
Transmit data                │
       ↓                     │
Network changes              │
       ↓                     │
Observe again ────────────────┘
```

This is a **feedback loop**.

The scheduler continuously reacts to the consequences of its previous decisions.

---

# 17. Why this is especially important for our scenario

The X-band environment can change.

For example:

```text
Normal condition
      ↓
Node has good SNR
      ↓
gets efficient transmission
```

Then:

```text
Rain / obstruction
      ↓
SNR decreases
      ↓
achievable rate decreases
      ↓
same allocation may become inefficient
```

A dynamic scheduler can react at the next decision point.

This is fundamentally different from fixed allocation.

---

# 18. What I understand now

I now understand the basic execution cycle of a scheduler:

```text
1. Observe current network state
2. Decide resource allocation
3. Apply allocation
4. Simulate transmission
5. Update queues/channel/delays/etc.
6. Observe the new state
7. Make the next decision
```

This process repeats throughout the simulation.

The scheduler therefore isn't a one-time calculation.

It is a **repeated feedback-based decision process**.

---

# 19. Things I still need to understand

I still need to learn:

- Exactly how the simulator calculates transmitted data.
- How achievable rate is calculated from SNR/SINR.
- How queue evolution is mathematically modeled.
- How packets and deadlines are represented.
- How a scheduling metric is calculated.
- How a scheduler handles resources that are left unused.
- How dynamic slot sizes would actually be represented.
- How this repeated scheduling loop becomes an RL environment.
- What exactly the RL agent observes.
- What exactly the RL agent outputs.

The last two questions lead directly into the next part of the project.

---

# Key takeaway

> **A scheduler repeatedly observes the current network state, allocates resources, lets the network respond, and then makes another decision using the updated state.**

The important structure is:

```text
State
  ↓
Decision
  ↓
Resource allocation
  ↓
Network response
  ↓
New state
  ↓
Decision again
```

This repeated loop is the foundation for understanding **Reinforcement Learning**.