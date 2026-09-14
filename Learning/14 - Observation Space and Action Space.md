## Why I am learning this

I understand that the RL environment gives the agent an observation and receives an action.

Now I need to understand two specific concepts:

```text
Observation Space
Action Space
```

These define:

> **What the agent is allowed to see and what the agent is allowed to do.**

I need to understand this before looking at PPO or other RL algorithms.

---

# 1. Observation vs Observation Space

I already know that an **observation** is the current state information given to the agent.

For example:

```text
Queue = 80
SNR = 18 dB
Delay = 12 ms
```

The **observation space** describes what values and structures are allowed for those observations.

So:

```text
Observation
    ↓
One actual state at this moment

Observation space
    ↓
The possible structure/range of observations
```

---

# 2. Simple example

Suppose the agent observes:

```text
Queue size
```

and the queue can contain:

```text
0 to 1000 packets
```

Then the observation space needs to represent that range.

At one moment:

```text
Observation = 250 packets
```

But the space says something like:

```text
Queue ∈ [0, 1000]
```

---

# 3. Multiple nodes

Our project has multiple ground nodes.

Suppose there are 4 nodes and I use:

```text
Queue
SNR
Delay
```

for each node.

Then one observation might be:

```text
Node A → [80, 18, 12]
Node B → [20, 25, 5]
Node C → [60, 10, 18]
Node D → [0, 15, 0]
```

This could be represented as a numerical array.

Conceptually:

```text
[
  [80, 18, 12],
  [20, 25, 5],
  [60, 10, 18],
  [0,  15, 0]
]
```

The actual representation used in the project can be different.

---

# 4. Why observations are usually numerical

The RL algorithm operates on mathematical values.

So instead of giving the agent:

```text
"Node A has a good channel"
```

I would normally represent that information numerically.

For example:

```text
SNR = 22 dB
```

Similarly:

```text
"High queue"
```

could become:

```text
Queue = 500 packets
```

The environment converts the simulated network state into numbers that the learning algorithm can process.

---

# 5. Different variables can have very different scales

This creates an important issue.

Suppose the observation contains:

```text
Queue = 500
SNR = 15
Delay = 0.020
```

These values have very different numerical scales.

If I feed everything directly into a neural network, one variable can have a much larger numerical magnitude than another.

Therefore observations are often **normalized or scaled**.

For example, conceptually:

```text
Queue:
0 → 1000

becomes:

0 → 1
```

and:

```text
SNR:
0 → 30 dB

becomes:

0 → 1
```

The exact normalization method will need to be decided during implementation.

---

# 6. Observation space should represent realistic values

Suppose:

```text
SNR normally ranges from -10 dB to 30 dB
```

The environment should define an appropriate range rather than assuming something unrealistic like:

```text
SNR = 0 to 1,000,000
```

The observation space should match the simulator's actual variables.

This is another reason I need to design the simulator and state representation carefully.

---

# 7. Action vs Action Space

I already know that an **action** is the allocation chosen by the scheduler.

For example:

```text
A → 2 ms
B → 5 ms
C → 3 ms
```

The **action space** describes what actions the agent is allowed to produce.

So:

```text
Action
    ↓
One particular decision

Action space
    ↓
All valid decisions the agent can potentially make
```

---

# 8. Discrete action space

A **discrete action** means the agent chooses from a finite set of possibilities.

For example:

```text
Action 0 → serve Node A
Action 1 → serve Node B
Action 2 → serve Node C
Action 3 → serve Node D
```

The agent chooses one of these.

This is relatively simple.

---

# 9. Continuous action space

A **continuous action** can take values across a range.

For example:

```text
Node A allocation = 0.0 to 1.0
```

could represent a resource proportion.

The agent might output:

```text
A = 0.23
B = 0.47
C = 0.30
```

These are not selected from a small predefined list.

They are continuous values.

---

# 10. Why this matters for our project

Our scheduler may eventually need to decide something like:

```text
How much of the available resource should each node receive?
```

That naturally looks like a continuous allocation problem.

For example:

```text
A → 20%
B → 50%
C → 30%
```

But this is not automatically the final action design.

I still need to determine what representation makes the most sense for the simulator and chosen RL algorithm.

---

# 11. The allocation constraint

If I use proportions, there is an important constraint:

```text
A + B + C + ... + N = 1
```

because the entire resource pool is being distributed.

For example:

```text
0.20 + 0.50 + 0.30 = 1.00
```

Valid.

But:

```text
0.50 + 0.50 + 0.50 = 1.50
```

would be invalid if only one complete resource pool exists.

This means resource-allocation actions are more complicated than simply giving each node an independent number.

---

# 12. Why invalid actions are a problem

Suppose the agent outputs:

```text
A = 7 ms
B = 7 ms
C = 7 ms
```

but:

```text
Frame = 10 ms
```

Then:

```text
7 + 7 + 7 = 21 ms
```

The action cannot actually be executed.

So the environment needs a strategy for handling this.

Possible approaches include:

```text
Normalize the output
```

or:

```text
Transform the agent's output into a valid allocation
```

or:

```text
Design the action representation so invalid allocations cannot be produced
```

The best approach is an implementation/design decision I have not made yet.

---

# 13. Example: normalized allocation

Suppose the agent produces raw values:

```text
A = 2
B = 5
C = 3
```

Their total is:

```text
10
```

So the normalized allocation becomes:

```text
A = 0.20
B = 0.50
C = 0.30
```

If:

```text
Frame = 10 ms
```

then:

```text
A → 2 ms
B → 5 ms
C → 3 ms
```

This is one possible way to convert an action into a valid allocation.

The exact implementation needs further study.

---

# 14. Discrete vs continuous: simple comparison

| Type | Example | Basic idea |
|---|---|---|
| Discrete | Choose Node 3 | Pick one predefined option |
| Continuous | Allocate 0.37 | Choose a value from a range |
| Hybrid | Choose node + amount | Combination of both |

Our scheduling problem may potentially involve a continuous or hybrid action space.

But I should **not choose one yet just because it sounds suitable**.

The final choice should follow from the actual resource model.

---

# 15. Observation space can also become large

Suppose I have:

```text
16 nodes
```

and each node has:

```text
Queue
SNR
Delay
```

Then I already have:

```text
16 × 3 = 48
```

node-level values.

If I add:

```text
Traffic class
Previous allocation
Historical throughput
```

the observation becomes larger.

This isn't necessarily bad.

But unnecessary information can make the learning problem harder.

So the state should be:

> **Rich enough to make good decisions, but not unnecessarily complicated.**

---

# 16. Global vs node-level information

Some observations belong to individual nodes:

```text
Queue of Node A
SNR of Node A
Delay of Node A
```

Other information describes the whole network:

```text
Total available resources
Current frame
Overall interference
Rain condition
```

So the observation can contain both:

```text
Per-node information
        +
Global information
```

---

# 17. Example observation structure

A conceptual observation might look like:

```text
Node information:

Node 1 → [queue, SNR, delay]
Node 2 → [queue, SNR, delay]
...
Node 16 → [queue, SNR, delay]

Global information:

[available resources, time information, ...]
```

This is just a conceptual design.

I have not yet decided which exact features will be used.

---

# 18. Why I should not include everything

Imagine I include:

```text
Queue
SNR
SINR
Path loss
Distance
Rain attenuation
Fading coefficient
Antenna gain
Position
Velocity
Altitude
Temperature
Wind
Previous throughput
Previous allocation
...
```

This could make the observation very large.

But some variables may already be indirectly represented by others.

For example:

```text
Path loss
Distance
Antenna gains
Rain attenuation
Fading
```

may already be used by the simulator to calculate:

```text
SNR/SINR
```

The agent might not need all of those physical variables directly.

This is an important design question:

> **Should the agent see raw physical information, processed network information, or both?**

I haven't decided this yet.

---

# 19. The simulator can hide complexity from the agent

The environment can internally calculate:

```text
Distance
     ↓
Path loss
     ↓
Rain attenuation
     ↓
Fading
     ↓
Received signal
     ↓
SNR/SINR
     ↓
Achievable rate
```

The agent may only receive:

```text
SNR/SINR
```

rather than every intermediate physical calculation.

This gives a useful separation:

```text
PHY model
   ↓
calculates channel behavior

RL state
   ↓
receives information useful for scheduling
```

---

# 20. Action does not have to expose simulator internals

Similarly, the agent doesn't necessarily need to control:

```text
Path loss
Rain
Fading
```

Those belong to the environment.

The agent controls scheduling decisions such as:

```text
Resource allocation
```

So I can separate:

```text
Environment controls:
    network physics

Agent controls:
    scheduling
```

---

# 21. A useful mental model

I can think of the RL environment as an interface.

```text
             ENVIRONMENT
                  │
       ┌──────────┴──────────┐
       │                     │
       ↓                     ↑
 Observation              Action
       │                     │
       ↓                     │
     AGENT ──────────────────┘
       │
       ↓
     Reward
```

The environment exposes only what the agent needs.

This prevents the RL agent from directly manipulating the internal simulator.

---

# 22. What I understand now

I understand that:

### Observation space

Defines:

> What information the agent can receive and what form/range that information can take.

### Action space

Defines:

> What decisions the agent is allowed to make.

For our project:

```text
Observation:
Current network condition

Action:
Resource allocation

Environment:
Applies allocation and simulates consequences
```

---

# 23. Things I still need to understand

I still need to decide or learn:

- The exact observation vector.
- Whether observations should be normalized.
- The exact action representation.
- Whether actions should be discrete, continuous, or hybrid.
- How to guarantee valid resource allocations.
- How the action space affects algorithm selection.
- How 16-node observations should be represented efficiently.
- Whether node ordering creates any problems.
- Whether a neural network can process all nodes independently or together.
- Whether the final project actually needs a sophisticated architecture such as attention/GNNs.

These questions will become important when I study the actual RL algorithms.

---

# Key takeaway

> **The observation space defines what the RL scheduler can see, while the action space defines what it can control.**

For this project:

```text
OBSERVATION SPACE
        ↓
"What can the scheduler know?"

ACTION SPACE
        ↓
"What can the scheduler change?"
```

Getting these two definitions right is more important than choosing a fancy RL algorithm first.