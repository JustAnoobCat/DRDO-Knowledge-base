## Why I am learning this

I understand the basic RL loop:

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
```

Now I need to translate the communication-scheduling problem into these three main pieces:

```text
STATE
ACTION
REWARD
```

This is important because choosing the RL algorithm comes **after** defining what the agent is actually supposed to learn.

---

# 1. State

The **state** is the information available to the scheduler when it has to make a decision.

For our project, the state could contain things such as:

```text
Node 1:
    Queue = ...
    SNR = ...
    Delay = ...

Node 2:
    Queue = ...
    SNR = ...
    Delay = ...

...
```

So the agent gets a numerical representation of the current network condition.

---

# 2. What information could be in the state?

Potential state variables include:

### Queue information

```text
Queue length
```

This tells the agent how much data is waiting.

---

### Channel information

```text
SNR / SINR
```

This gives information about current link quality.

---

### Delay information

```text
Packet / Head-of-Line delay
```

This helps the agent recognize data that has been waiting too long.

---

### Traffic information

The agent may need to know whether traffic is:

```text
Critical
Voice
Video
Bulk
```

or some equivalent representation.

---

### Resource information

The agent may also need information about:

```text
Available resources
Current allocation
Remaining time
```

depending on how the environment is designed.

---

# 3. The state should contain useful information

I should not simply put every possible variable into the state.

For example, suppose I give the agent:

```text
Queue
SNR
Delay
Traffic class
Position
Rain
Wind
Temperature
Altitude
Battery level
...
```

Some of these may not actually help the scheduling decision.

A useful question is:

> **Does this information help the scheduler decide how to allocate resources?**

If not, it may not need to be part of the RL state.

---

# 4. The state must be available at decision time

This is extremely important.

The scheduler cannot use information that it would not actually know.

For example:

```text
Current SNR → available
Current queue → available
Current delay → available
```

But:

```text
Exact SNR 20 ms into the future
```

would normally be future information.

Using it would give the agent an unrealistic advantage.

So:

> **The state should represent information realistically available to the scheduler at the time of the decision.**

---

# 5. State is a snapshot

I can think of state as a snapshot of the system.

For example:

```text
Time = 100 ms

Node A:
Queue = 80
SNR = 18 dB
Delay = 12 ms

Node B:
Queue = 20
SNR = 25 dB
Delay = 5 ms

Node C:
Queue = 100
SNR = 8 dB
Delay = 18 ms
```

That snapshot becomes the input to the agent.

Then the agent chooses an action.

---

# 6. Action

The **action** is what the agent controls.

In our project, the action is related to:

> **How communication resources should be allocated among the nodes.**

If I use a simplified time-only model:

```text
Frame = 10 ms

Action:

A → 2 ms
B → 5 ms
C → 3 ms
```

The action is therefore the allocation vector:

```text
[2, 5, 3]
```

assuming the three values represent the resource assigned to each node.

---

# 7. The action must obey constraints

The agent cannot simply output any numbers it wants.

If:

```text
Frame = 10 ms
```

then:

```text
A = 5 ms
B = 5 ms
C = 5 ms
```

is invalid because:

```text
5 + 5 + 5 = 15 ms
```

The available resource is only 10 ms.

So the action space needs to respect the system constraints.

---

# 8. Valid action

For example:

```text
A = 2 ms
B = 5 ms
C = 3 ms
```

gives:

```text
2 + 5 + 3 = 10 ms
```

This is valid for the simplified example.

The exact mechanism for ensuring valid actions is something I will need to study later.

---

# 9. Action representation matters

There are different ways to represent the action.

### Option 1 — Choose a node

```text
Action = Node B
```

This could mean:

> Give the next available resource to Node B.

This is simple, but may require many repeated decisions to construct a complete allocation.

---

### Option 2 — Choose slot duration

```text
Action = [2 ms, 5 ms, 3 ms]
```

This directly represents the allocation.

It is more expressive but can become more complicated.

---

### Option 3 — Choose proportions

Instead of actual milliseconds:

```text
A → 20%
B → 50%
C → 30%
```

The simulator converts these into actual time.

This can make the total-resource constraint easier to represent.

---

# 10. Action space is a major design decision

The complexity of the RL problem depends heavily on the action space.

Suppose I have:

```text
16 nodes
```

and the agent has to independently choose a time allocation for every node.

That can become a much more complicated action than:

```text
Choose one node
```

So I should not decide the action representation casually.

It needs to balance:

```text
Expressiveness
      ↕
Complexity
      ↕
Training difficulty
```

---

# 11. Reward

The reward tells the agent how good its action was.

After the simulator applies an allocation, I can calculate things such as:

```text
How much data was transmitted?
How much delay occurred?
How many packets were dropped?
How fair was the allocation?
```

These can be converted into a reward.

---

# 12. Simple reward example

Suppose one scheduling step produces:

```text
Throughput = high
Delay = low
Drops = low
```

Then:

```text
Reward = high
```

Another action produces:

```text
Throughput = low
Delay = high
Drops = high
```

Then:

```text
Reward = low
```

The agent learns which decisions tend to produce better outcomes.

---

# 13. Combining multiple metrics

A possible conceptual reward is:

```text
Reward =
    throughput benefit
    - delay penalty
    - drop penalty
    + fairness benefit
```

For example:

```text
R =
    α × throughput
    - β × delay
    - γ × drops
    + δ × fairness
```

where:

```text
α, β, γ, δ
```

control the relative importance of each term.

This is only a **conceptual formulation** for now.

I have not decided the actual reward function for the project.

---

# 14. Reward scale matters

Suppose:

```text
Throughput = 1000
Delay = 20
Drops = 2
```

If I simply combine raw values, throughput may dominate the reward because its numerical scale is much larger.

For example:

```text
1000 - 20 - 2 = 978
```

This does not necessarily mean throughput should be 50 times more important than delay.

So reward components may need to be:

- normalized,
- scaled,
- weighted.

I need to be careful here because poor reward scaling can make learning difficult.

---

# 15. Reward should reflect the actual goal

The reward is essentially the behavior I am asking the agent to learn.

If I reward only:

```text
Throughput
```

the agent may learn to maximize throughput at the expense of:

```text
Latency
Fairness
Critical traffic
Packet drops
```

If I heavily punish:

```text
Packet drops
```

the agent may become very conservative and sacrifice throughput.

Therefore reward design is a **trade-off problem**.

---

# 16. State, Action, and Reward are different

I need to keep these separate.

### State

> What is happening?

```text
Queue = 100
SNR = 15 dB
Delay = 20 ms
```

### Action

> What should I do?

```text
Allocate 4 ms
```

### Reward

> How good was that decision?

```text
Good throughput
Low delay
Few drops
→ positive reward
```

So:

```text
STATE
  ↓
"What is happening?"

ACTION
  ↓
"What should I do?"

REWARD
  ↓
"How good was the result?"
```

---

# 17. Complete example

Suppose:

```text
Frame duration = 10 ms
```

Current state:

```text
A:
Queue = 100
SNR = 20 dB
Delay = 10 ms

B:
Queue = 30
SNR = 25 dB
Delay = 5 ms

C:
Queue = 80
SNR = 8 dB
Delay = 18 ms
```

The agent receives this state.

It chooses:

```text
Action:

A → 4 ms
B → 2 ms
C → 4 ms
```

The simulator applies the allocation.

Suppose the result is:

```text
High throughput
Moderate delay
No packet drops
```

The environment calculates:

```text
Reward = positive
```

Then the queues and delays are updated.

The agent receives:

```text
New state
```

and makes another decision.

---

# 18. Why the state and action design come first

I might be tempted to start with:

```text
"PPO sounds good, let's use PPO."
```

But that is backwards.

The logical order is closer to:

```text
Define scheduling problem
        ↓
Define state
        ↓
Define action
        ↓
Define objectives/reward
        ↓
Understand constraints
        ↓
Choose suitable RL algorithm
```

The algorithm should fit the problem.

Not the other way around.

---

# 19. What this means for our project

A possible simplified RL environment could eventually look like:

```text
Observation:
    Queue of each node
    SNR of each node
    Delay of each node
    Traffic information
    Available resources

Action:
    Resource allocation among nodes

Reward:
    Throughput
    - delay penalty
    - drop penalty
    + fairness
```

Then:

```text
RL Agent
    ↓
allocation
    ↓
communication simulator
    ↓
network response
    ↓
reward + new observation
```

This is enough to define the **basic RL problem**, even though the exact details are still undecided.

---

# 20. I should distinguish confirmed requirements from my design choices

At this stage, I should not assume that every variable above is required.

For example:

```text
Queue → very likely relevant
SNR   → very likely relevant
Delay → potentially relevant
Traffic class → potentially relevant
Power → depends on final resource model
Geometry → potentially useful
Rain parameters → may be represented indirectly through channel quality
```

The final state/action/reward design should be based on:

- project requirements,
- mentor feedback,
- simulator scope,
- available computational resources,
- experiments.

---

# 21. What I understand now

I can now translate the scheduling problem into an RL formulation:

```text
STATE
Current communication-network condition

        ↓

ACTION
Resource allocation chosen by the agent

        ↓

ENVIRONMENT
Digital twin simulates the result

        ↓

REWARD
Measures how good the result was

        ↓

NEW STATE
Updated network condition
```

This repeats continuously during training.

---

# 22. Things I still need to understand

I still need to learn:

- How observations are represented numerically.
- How to handle 16 nodes efficiently.
- How continuous and discrete actions differ.
- How invalid resource allocations can be prevented.
- How reward normalization works in practice.
- How to represent multiple traffic classes.
- How to represent fairness mathematically.
- How the RL environment is implemented using Gymnasium.
- What PPO actually does.
- Why PPO might be suitable for this action space.
- Whether PPO is actually the best choice for our final design.

---

# Key takeaway

> **Before choosing an RL algorithm, I need to clearly define what the agent can see, what it can control, and how I judge its decisions.**

In our project:

```text
State  → current network condition
Action → resource allocation
Reward → quality of that allocation
```

Once these are properly defined, I can start understanding how an actual RL environment is built.