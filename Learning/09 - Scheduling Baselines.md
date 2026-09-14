## Why I am learning this

I already understand that the scheduling problem is about allocating limited resources while balancing things like throughput, delay, drops, and fairness.

Before using AI/RL, I need to understand the **simpler ways of doing the same scheduling task**.

These simpler methods are important because they can become:

- baselines for comparison,
- fallback methods,
- starting points for designing a better scheduler,
- ways to understand what the RL agent is actually improving.

---

# 1. What is a baseline?

A **baseline** is a simpler method that I can use as a reference.

Suppose I eventually build an RL scheduler.

I cannot simply say:

> "The RL scheduler is good."

I need to compare it against something.

For example:

```text
Static TDMA
      ↓
Heuristic scheduler
      ↓
RL scheduler
```

Then I can ask:

> Does the RL scheduler actually improve anything?

---

# 2. Baseline 1 — Static / Equal Allocation

I already learned about static TDMA.

The basic idea is:

```text
Every node gets a predetermined share.

Node 1 → 25%
Node 2 → 25%
Node 3 → 25%
Node 4 → 25%
```

The allocation does not change based on the current state.

### Advantage

Very simple.

### Disadvantage

It doesn't react to:

- changing queues,
- changing channel conditions,
- sudden traffic bursts,
- urgent traffic.

---

# 3. Baseline 2 — Round Robin

**Round Robin (RR)** gives nodes service in a repeating order.

Example:

```text
A → B → C → D → A → B → C → D ...
```

Each node gets its turn.

This is simple and naturally provides a degree of fairness.

---

# 4. Why Round Robin can be inefficient

Suppose:

```text
A → 100 packets
B → 2 packets
C → 80 packets
D → 0 packets
```

Round Robin might still give:

```text
A → turn
B → turn
C → turn
D → turn
```

But D has nothing to send.

Depending on the implementation, that opportunity may be wasted or need to be reassigned.

A smarter scheduler could notice:

```text
D has no data
      ↓
D needs little/no service
      ↓
use the opportunity elsewhere
```

This is one reason dynamic scheduling can outperform simple RR.

---

# 5. Baseline 3 — Queue-Based Scheduling

A very simple heuristic is:

> Give more resources to nodes with larger queues.

For example:

```text
A → 10 packets
B → 100 packets
C → 20 packets
```

The scheduler might prioritize:

```text
B > C > A
```

because B has the largest backlog.

This is more responsive than static allocation.

---

# 6. Why queue size alone isn't enough

Consider:

```text
Node A:
Queue = 10
Traffic = critical
Deadline = almost reached

Node B:
Queue = 100
Traffic = bulk data
Deadline = far away
```

A pure queue-based scheduler may choose B.

But A's packets may be much more important.

So I learned that:

> **A good scheduler may need to consider more than one state variable.**

This connects back to the network-state module.

---

# 7. Baseline 4 — Channel-Aware Scheduling

Another possibility is to consider channel quality.

Suppose:

```text
Node A → excellent channel
Node B → poor channel
```

Giving the same amount of resource to both might result in very different amounts of transmitted data.

A channel-aware scheduler can favor nodes that can currently use the resource efficiently.

Conceptually:

```text
Good channel
     ↓
higher achievable rate
     ↓
more data transmitted per unit resource
```

---

# 8. But channel-only scheduling also has a problem

Suppose:

```text
Node A → excellent channel, no urgent data
Node B → poor channel, critical data
```

Always choosing A could maximize immediate throughput but delay B's important traffic.

So:

```text
Queue-only       → incomplete
Channel-only     → incomplete
```

This suggests combining multiple factors.

---

# 9. Proportional Fair (PF)

**Proportional Fair scheduling** tries to balance:

```text
Current channel/rate
        +
Past service
```

The basic intuition is:

> Give resources to nodes that can currently transmit efficiently, while also preventing nodes from being ignored for too long.

For example:

```text
Node A → good current rate, already served a lot
Node B → moderate current rate, rarely served
```

PF may give B a better opportunity than a purely channel-based scheduler.

---

# 10. Why PF is useful

PF is interesting because it demonstrates an important scheduling principle:

> **A scheduler can consider both short-term efficiency and long-term fairness.**

It is more intelligent than simply:

```text
"Always choose the node with the best channel."
```

But PF is not specifically designed around packet deadlines.

---

# 11. M-LWDF

Another scheduler I encountered is:

**Modified Largest Weighted Delay First (M-LWDF)**.

The important idea is:

> A packet that has been waiting a long time becomes increasingly important to serve.

So instead of looking only at:

```text
Queue size
```

the scheduler also considers:

```text
How long has the data been waiting?
```

This makes it more suitable for delay-sensitive traffic.

---

# 12. Intuition behind M-LWDF

Imagine:

```text
Node A:
Large queue
But packets are not very urgent

Node B:
Small queue
But one critical packet has been waiting almost until its deadline
```

A delay-aware scheduler can recognize that B may deserve service.

This is different from simply choosing the node with the largest queue.

---

# 13. What these baselines teach me

The progression is useful:

```text
Static
  ↓
Round Robin
  ↓
Queue-aware
  ↓
Channel-aware
  ↓
Fairness-aware
  ↓
Delay-aware
  ↓
More advanced optimization / RL
```

Each step adds more information to the scheduling decision.

---

# 14. A useful way to think about scheduler intelligence

I can think of schedulers as gradually becoming more **state-aware**.

### Static

```text
Doesn't care about current state.
```

### Round Robin

```text
Cares about whose turn it is.
```

### Queue-based

```text
Cares about how much data is waiting.
```

### Channel-aware

```text
Cares about current transmission conditions.
```

### Fairness-aware

```text
Cares about previous service.
```

### Delay-aware

```text
Cares about how long data has been waiting.
```

### RL

```text
Can potentially learn how different state variables
should influence the decision.
```

The last statement is a possibility, not a guarantee.

---

# 15. Why I should not jump directly to RL

This is important for my project.

If I only build:

```text
RL scheduler
```

and it performs well, I still don't know:

> Would a much simpler algorithm have achieved nearly the same result?

For example:

```text
Static TDMA → 70 throughput
Heuristic     → 82 throughput
RL            → 84 throughput
```

In this case, RL may not provide much practical benefit compared with the much simpler heuristic.

But if:

```text
Static TDMA → 70
Heuristic   → 78
RL          → 95
```

then the additional complexity of RL becomes much more interesting.

So:

> **Performance should be compared against complexity.**

---

# 16. Baselines and the Digital Twin

The digital twin can provide the same simulated environment to every scheduler.

For example:

```text
              Same network scenario
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
   Static TDMA    Heuristic          RL
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                 Compare results
```

This makes the comparison much more meaningful.

Each scheduler experiences the same:

- traffic,
- channel conditions,
- node positions,
- resource limits,
- disturbances.

---

# 17. Example experiment

Suppose I simulate:

```text
16 nodes
10 ms decision interval
varying traffic
changing X-band channel conditions
```

I could test:

```text
Experiment A → Static allocation
Experiment B → Round Robin
Experiment C → heuristic
Experiment D → RL
```

Then record:

```text
Throughput
Average latency
Packet-drop rate
Fairness
Resource utilization
```

I could also measure:

```text
Scheduler computation time
```

because a scheduler that performs slightly better but takes too long to make a decision may not be useful.

---

# 18. Important distinction

A baseline is **not necessarily a bad algorithm**.

For example, Round Robin may be perfectly reasonable when:

- traffic is relatively uniform,
- channel conditions are similar,
- fairness is the main concern,
- computational simplicity is important.

The purpose of a baseline is simply to provide a reference point.

---

# 19. What I understand now

I understand why I need simpler scheduling algorithms before RL.

The main lesson is:

> **Different schedulers make different trade-offs because they use different information.**

For example:

```text
Static      → predetermined allocation
RR          → turn-based fairness
Queue-based → backlog
PF          → rate + fairness
M-LWDF      → delay + rate/priority
RL          → learned policy from network state
```

The exact algorithms I eventually implement should depend on the final project scope.

---

# 20. Things I still need to understand

I still need to learn:

- How exactly Round Robin would work in our TDMA simulator.
- How Proportional Fair calculates its scheduling metric.
- How M-LWDF calculates its metric.
- What a good heuristic for our specific network could look like.
- What optimization method could provide a useful reference solution.
- How these methods behave under sudden traffic bursts.
- How they behave when X-band SNR suddenly decreases.
- Which baselines are actually worth implementing given our project resources.
- How to fairly compare all schedulers.

---

# Key takeaway

> **Before trying to prove that an AI scheduler is good, I need simpler schedulers to compare it against.**

The goal is not:

> "Build the most complicated scheduler."

The goal is:

> **Find out whether a more intelligent scheduler provides a meaningful improvement over simpler approaches.**