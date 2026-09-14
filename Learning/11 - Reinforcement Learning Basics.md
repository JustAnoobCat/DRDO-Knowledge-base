## Why I am learning this

I already understand how a normal scheduler works:

```text
Observe network state
        ↓
Apply scheduling rule
        ↓
Allocate resources
        ↓
Network changes
        ↓
Observe again
```

Now I want to understand **Reinforcement Learning (RL)** without immediately getting into complicated algorithms like PPO.

The main question is:

> **How can a scheduler learn which allocation decisions are good instead of relying completely on manually written rules?**

---

# 1. What is Reinforcement Learning?

Reinforcement Learning is a type of machine learning where an **agent learns by interacting with an environment**.

Instead of being explicitly told:

> "When the queue is this size and SNR is this value, give the node exactly 3 ms."

the agent tries actions and receives feedback about how good those actions were.

The basic idea is:

```text
Environment
     ↓
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
Agent learns
```

This happens repeatedly.

---

# 2. The three main concepts

There are three things I need to understand first:

### State

What is happening right now?

```text
Queues
SNR/SINR
Delays
Traffic information
Available resources
...
```

### Action

What decision should be made?

```text
Resource allocation
```

### Reward

How good was the decision?

```text
High throughput
→ good

High packet drops
→ bad

High delay
→ bad
```

So:

```text
State → Action → Reward
```

is the basic RL loop.

---

# 3. Agent

The **agent** is the thing making the decision.

In our project:

```text
RL Agent = learned scheduler
```

It observes the network state and decides how resources should be allocated.

Conceptually:

```text
Network state
      ↓
   RL Agent
      ↓
Allocation decision
```

The agent is therefore replacing the hand-designed scheduling rule.

---

# 4. Environment

The **environment** is everything the agent interacts with.

For our project, the environment would be the communication-system simulation.

It can contain:

```text
Ground nodes
ACN
Traffic
Queues
X-band channel
SNR/SINR
Resource availability
Packet transmission
Packet drops
...
```

The agent doesn't directly "control reality."

Instead:

```text
RL Agent
    ↓
Digital Twin / Simulator
    ↓
Simulated network response
```

---

# 5. One RL step

At one decision point:

```text
1. Environment gives state
2. Agent chooses action
3. Environment applies action
4. Environment calculates result
5. Environment gives reward
6. Environment moves to new state
```

For example:

```text
State:
A queue = 100
B queue = 20
C queue = 50

        ↓

Agent chooses:

A → 5 ms
B → 2 ms
C → 3 ms

        ↓

Simulator runs transmission

        ↓

Some packets are transmitted

        ↓

Reward calculated

        ↓

Queues become:

A → 70
B → 10
C → 35

        ↓

New state
```

Then the agent makes another decision.

---

# 6. Learning happens over many interactions

The agent initially does not necessarily know which action is best.

It might make poor decisions.

For example:

```text
State
  ↓
Bad allocation
  ↓
Many packet drops
  ↓
Low reward
```

Later it may discover:

```text
Similar state
  ↓
Different allocation
  ↓
Fewer drops
  ↓
Higher reward
```

Over many experiences, the agent tries to learn which decisions tend to produce better results.

---

# 7. Reward is the feedback signal

The reward tells the agent whether its recent decision was useful.

For example:

```text
High throughput
       +
Low delay
       +
Few packet drops
       +
Good fairness
       ↓
Positive reward
```

While:

```text
Low throughput
       +
High delay
       +
Many drops
       ↓
Poor reward
```

The exact reward formula is a project-design decision that I haven't finalized yet.

---

# 8. Reward is not the same as an objective

This distinction is useful.

I already learned about the optimization objective:

> What overall behavior do I want from the scheduler?

The RL reward is how I **communicate that desired behavior to the learning agent**.

For example:

```text
Desired behavior:
maximize throughput
minimize delay
minimize drops
```

can be converted into a reward such as:

```text
Reward =
    throughput benefit
    - delay penalty
    - drop penalty
```

So the reward function is closely related to the optimization objective.

---

# 9. Why reward design matters

Suppose I only reward throughput.

The agent may learn:

```text
Maximize throughput
```

even if that causes:

```text
Critical packets → delayed
Packet drops → high
Some nodes → starved
```

The agent is not necessarily "wrong."

It is simply optimizing the reward I gave it.

Therefore:

> **The reward function strongly influences what the RL scheduler learns to prioritize.**

---

# 10. Example of a bad reward

Imagine:

```text
Reward = total throughput
```

The agent discovers:

```text
Node A has excellent channel
Node A can transmit lots of data
```

So it may keep allocating resources to A.

Result:

```text
A → very high throughput
B → almost no service
C → almost no service
D → almost no service
```

Overall throughput may look good.

But the network may be unfair and important traffic may be delayed.

This is why reward design needs to reflect the actual scheduling goals.

---

# 11. Reward can combine multiple goals

Conceptually:

```text
Reward =
    + throughput
    - latency
    - packet drops
    + fairness
```

Weights can also be used:

```text
Reward =
    w1 × throughput
    - w2 × latency
    - w3 × drops
    + w4 × fairness
```

I already encountered this idea in optimization.

The important new point is:

> **The RL agent learns according to this reward signal.**

---

# 12. Policy

A **policy** is the strategy the agent uses to choose actions from states.

Conceptually:

```text
State
  ↓
Policy
  ↓
Action
```

For example:

```text
State:
Large queue
Good SNR
Low delay

        ↓

Policy

        ↓

Allocate more resources
```

The policy can eventually become a learned function rather than a manually written rule.

---

# 13. Traditional scheduler vs learned policy

A traditional scheduler might have a manually designed rule:

```text
IF queue is large
AND delay is high
THEN increase priority
```

An RL policy instead learns a relationship such as:

```text
Network state
      ↓
Learned policy
      ↓
Action
```

The exact internal relationship isn't manually specified as a list of rules.

It is learned during training.

---

# 14. Training

The agent needs to experience many situations before it can become useful.

This process is called **training**.

A simplified training loop:

```text
Start environment
       ↓
Observe state
       ↓
Choose action
       ↓
Environment responds
       ↓
Receive reward
       ↓
Update policy
       ↓
Repeat
```

This may happen for many episodes and many simulation steps.

---

# 15. What is an episode?

An **episode** is one complete run of the environment from a starting point until some termination condition.

For example:

```text
Episode starts
     ↓
t = 0
     ↓
Scheduling decisions
     ↓
10 ms
     ↓
20 ms
     ↓
30 ms
     ↓
...
     ↓
Episode ends
```

Then another episode starts.

The exact episode length is a design choice for the simulator.

---

# 16. Exploration vs exploitation

There is an important learning problem.

Suppose the agent already found an action that seems good.

Should it:

```text
Keep using that action?
```

or:

```text
Try something different?
```

These are called:

### Exploitation

Use what the agent already believes works well.

### Exploration

Try different actions to discover potentially better strategies.

The agent needs a balance between the two.

---

# 17. Why exploration matters in our project

Imagine the agent learns:

```text
When Node A has good SNR:
give A lots of resources
```

That might work in normal conditions.

But what if the network experiences:

```text
Sudden rain attenuation
```

or:

```text
LoS obstruction
```

If the agent never experienced these conditions during training, it may not know how to respond well.

Therefore the training environment needs sufficiently varied scenarios.

---

# 18. RL does not magically understand X-band

This is important.

The RL agent does not inherently know:

- what X-band means,
- what rain attenuation means,
- what SNR means,
- what a packet deadline means.

The simulator provides numerical state information.

For example:

```text
SNR = 18 dB
Queue = 50 packets
Delay = 12 ms
```

The agent learns relationships between these values and the consequences of actions.

So:

```text
Physics / network model
          ↓
Provides meaningful state
          ↓
RL learns scheduling behavior
```

The quality of the environment therefore matters enormously.

---

# 19. RL is learning a policy, not learning the entire network

Another useful distinction:

The project doesn't necessarily require the RL agent to learn:

```text
How electromagnetic waves propagate
```

or:

```text
How rain physically attenuates X-band signals
```

Those things can be modeled by the simulator.

The RL agent's job is much narrower:

> **Learn how to make resource-allocation decisions based on the current network state.**

This keeps the problem manageable.

---

# 20. Where the Digital Twin fits

The complete concept now looks like:

```text
                 DIGITAL TWIN
                      │
                      ↓
              Current network state
                      │
                      ↓
                  RL AGENT
                      │
                      ↓
                  Action
                      │
                      ↓
             Resource allocation
                      │
                      ↓
                 Simulator
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Queues       Channel      Traffic
          │           │           │
          └───────────┼───────────┘
                      ↓
                  New state
                      ↓
                   Reward
                      ↓
                  RL Agent
```

This is the central learning loop.

---

# 21. RL versus a normal scheduling rule

The difference can now be summarized simply.

### Hand-designed scheduler

```text
Human designs rule
       ↓
Rule makes decisions
```

### RL scheduler

```text
Human defines:
- state
- action space
- reward
- environment

       ↓

RL training

       ↓

Learned policy
```

The human still defines the problem.

The agent learns the decision strategy.

---

# 22. What I understand now

I understand the basic RL structure:

```text
Agent
Environment
State
Action
Reward
Policy
Episode
Training
Exploration
Exploitation
```

And I can map them to my project:

| RL concept | My project |
|---|---|
| Agent | AI scheduler |
| Environment | Communication digital twin/simulator |
| State | Current network condition |
| Action | Resource allocation |
| Reward | Quality of scheduling decision |
| Policy | Learned scheduling strategy |
| Episode | One simulation run |
| Training | Repeated interaction with simulator |

---

# 23. Things I still need to understand

I have **not** decided yet:

- Exact state representation.
- Exact action representation.
- Exact reward formula.
- Which RL algorithm to use.
- Whether the action should be discrete, continuous, or hybrid.
- How many actions are practical for the simulator.
- How training data/scenarios should be generated.
- How to prevent the agent from learning undesirable behavior.
- How to evaluate whether the learned policy is actually better.
- How PPO or other RL algorithms work internally.

I should understand these pieces before choosing a specific RL algorithm.

---

# Key takeaway

> **Reinforcement Learning turns the scheduling problem into a repeated learning loop: the agent observes the network state, chooses a resource allocation, receives feedback, and gradually learns a policy that tends to produce better results.**

The most important structure is:

```text
STATE
  ↓
AGENT / POLICY
  ↓
ACTION
  ↓
ENVIRONMENT
  ↓
REWARD
  ↓
NEW STATE
  ↺
```

The next step is to understand **how the state, action, and reward should be represented mathematically for this specific scheduling problem**.