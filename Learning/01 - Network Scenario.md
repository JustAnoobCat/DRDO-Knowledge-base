## What am I trying to understand?

Before understanding the AI, scheduler, or Digital Twin, I first need to understand the communication system that we are trying to simulate and improve.

The basic scenario is an aerial communication node (ACN) communicating with multiple ground nodes.

---

## 1. What is the ACN?

ACN stands for **Aerial Communication Node**.

I can think of the ACN as a **communication tower in the sky**.

Instead of having a communication tower on the ground, the ACN is flying above the area and provides a communication link between itself and multiple ground units.

The ACN could be an aircraft/UAV-type platform, depending on the final system assumptions.

The important thing is not the exact aircraft.

The important idea is:

> The ACN has a limited amount of communication resources and has to share them among multiple ground nodes.

---

## 2. What are the ground nodes?

The ground nodes are the different users/units that need to communicate with the ACN.

For example, they could represent:

- Soldiers or mobile units
- Command posts
- Radar units
- Reconnaissance/ISR units
- Vehicles
- Sensors

For the simulation, I don't necessarily need to model the real physical equipment.

I can represent each one as a **network node** with properties such as:

- Amount of data waiting to be transmitted
- Channel quality
- Priority
- Delay
- Location

---

## 3. Why are there multiple nodes?

The ACN is a shared communication system.

Imagine there are 16 ground nodes trying to communicate with the same ACN.

The ACN cannot simply give everyone unlimited bandwidth.

It has a finite amount of:

- Time
- Frequency/bandwidth
- Transmission power
- Other communication resources

Therefore, some kind of **resource allocation/scheduling** is required.

---

## 4. What kind of data are they sending?

Not every node has the same communication requirements.

Different types of traffic can exist.

For example:

### Critical command/control data

Small amount of data, but extremely important and time-sensitive.

If an emergency command arrives, it should not have to wait behind a large video transmission.

### Voice

Requires relatively low delay and should avoid excessive delay/jitter.

### ISR/video

Can generate a large amount of data.

It needs high throughput and can create a large queue if the ACN doesn't provide enough resources.

### Sensor/bulk data

Can often tolerate more delay than critical command/control traffic.

---

## 5. Why is this a scheduling problem?

Suppose there are four nodes:

| Node | Data waiting | Importance |
|------|--------------|------------|
| Node 1 | Very little | Normal |
| Node 2 | Large amount | High |
| Node 3 | Small amount | Critical |
| Node 4 | Large amount | Normal |

If I simply divide the available communication resources equally between all four nodes, I may not be using the resources efficiently.

For example:

- Node 1 may not have enough data to use its allocation.
- Node 2 may have a large backlog.
- Node 3 may have an urgent message.
- Node 4 may have a large but less urgent amount of data.

So the ACN needs to make decisions about **who should receive resources and how much**.

---

## 6. The channel is not always the same

Another important thing I learned is that the communication link between the ACN and a ground node can change.

The channel can be affected by things such as:

- Rain
- Obstructions
- Line-of-sight conditions
- Distance
- Fading
- Other channel effects

This means that two nodes may have completely different communication conditions at the same moment.

For example:

Node A:

> Strong signal → can transmit data efficiently.

Node B:

> Weak signal → can transmit less efficiently.

Therefore, the scheduler should not only look at how much data each node has.

It may also need to consider **channel quality**.

---

## 7. The basic problem

The problem can therefore be simplified to:

> The ACN has limited communication resources and multiple ground nodes competing for those resources.

Each node can have different:

- Data requirements
- Channel conditions
- Traffic priorities
- Delays

These conditions can also change over time.

The scheduler has to continuously decide how to distribute the available resources.

---

## 8. Where does AI come into this?

The AI is not the communication system itself.

The AI is being considered as a **decision-making mechanism for the scheduler**.

The general loop is:

Network state
↓
Scheduler/AI makes a decision
↓
Resources are allocated
↓
Nodes transmit
↓
Queues and channel conditions change
↓
New network state
↓
Scheduler makes another decision

So this is a **dynamic decision-making problem**.

---

## 9. How does this relate to the Digital Twin?

The real system would involve:

ACN
+
Ground nodes
+
Wireless channel
+
Traffic
+
Scheduler

Building and testing the real system would obviously be difficult and expensive.

So the project creates a **software representation of the system**.

This simulated system can reproduce things such as:

- Ground nodes
- Data queues
- Channel conditions
- Traffic
- Resource allocation
- Scheduling decisions
- Performance

This software representation is the basis of the **Digital Twin**.

---

## What I understand now

The project is fundamentally about a communication system where:

1. One aerial communication node serves multiple ground nodes.
2. The communication resources are limited.
3. Different nodes have different amounts/types of traffic.
4. Channel conditions can change.
5. Some traffic is more urgent than other traffic.
6. Therefore, equal/static resource allocation may not always be efficient.
7. A scheduler is needed to decide how resources should be distributed.
8. AI/ML is one possible way to make these scheduling decisions intelligently.
9. The Digital Twin allows us to simulate this entire process.

---

## Things I should understand next

- What exactly are the communication resources?
- What does bandwidth mean in this system?
- What is TDMA?
- What are time slots?
- What are subchannels/RBGs?
- What is SNR?
- How does SNR affect the amount of data that can be transmitted?
- What exactly does the scheduler control?

---

## Important distinction

The exact aircraft type, number of nodes, bandwidth, power, frequency allocation, etc. are simulation/design choices unless they are explicitly specified by the project requirements.

I should not assume that an example value from a proposed architecture is an official requirement.

I should confirm important system assumptions with my mentor.