## What am I trying to understand?

Our project deals with an aerial communication network operating in the **X-band**.

Before I can understand how the scheduler should make decisions, I need to understand the communication channel between the ACN and the ground nodes.

The important idea is:

> The communication link between the ACN and a ground node is not always equally good.

The quality of the link affects how much data can be transmitted using the available resources.

---

## 1. What is X-band?

X-band is a portion of the radio-frequency spectrum.

It is generally considered to cover approximately:

**8 GHz to 12 GHz**

Our project is concerned with an X-band aerial communication scenario.

I do not need to assume one exact operating frequency yet. The exact frequency and other radio parameters are simulation/design choices that need to be confirmed for the project.

---

## 2. What is a communication channel?

The communication channel is the path through which the signal travels from the transmitter to the receiver.

In our case, the basic path is:

```text
Ground Node
     │
     │ Wireless signal
     ↓
    ACN
```

or in the opposite direction:

```text
ACN
 │
 │ Wireless signal
 ↓
Ground Node
```

The signal has to travel through the atmosphere and over a physical distance.

During this process, the signal can become weaker or be affected by different environmental factors.

---

## 3. Why does channel quality matter?

Suppose two ground nodes both have the same amount of data waiting.

```text
Node A → strong channel
Node B → weak channel
```

If both receive the same amount of communication resources, they may not transmit the same amount of data.

Node A may be able to transmit efficiently.

Node B may transmit much less data because its signal quality is poor.

Therefore:

> The amount of resources given to a node is not the only thing that determines how much data it can transmit.

The condition of the communication channel also matters.

---

## 4. Signal strength decreases with distance

The ACN is physically separated from the ground nodes.

As the signal travels over a longer distance, the received signal generally becomes weaker.

This is called **path loss**.

A simplified idea is:

```text
More distance
      ↓
More signal loss
      ↓
Weaker received signal
      ↓
Lower communication quality
```

For our project, the distance between the ACN and each ground node can therefore affect the quality of their links.

---

## 5. Line of Sight (LoS)

Because the ACN is in the air, the communication link can often have a relatively clear path to a ground node.

This is called **Line of Sight (LoS)**.

A simplified example:

```text
        ACN
         ●
        / \
       /   \
      /     \
     ↓       ↓
 Ground    Ground
 Node A    Node B
```

If there is a clear path between the transmitter and receiver, the signal can generally travel more directly.

However, the environment can cause obstructions or other propagation effects.

---

## 6. What happens if Line of Sight is lost?

Imagine a ground node is behind terrain or another obstruction.

The direct path can become blocked.

```text
        ACN
         ●
          \
           \
            █  ← obstruction
           /
      Ground Node
```

The signal may then experience significantly worse propagation conditions.

This can cause the link quality to decrease.

For the digital twin, this type of event can be simulated as a change in the channel conditions.

---

## 7. Rain can affect X-band communication

Rain can also affect radio communication, particularly at higher frequencies.

This is called **rain attenuation** or **rain fade**.

The basic idea is:

```text
Rain increases
      ↓
Signal experiences more attenuation
      ↓
Received signal becomes weaker
      ↓
SNR/SINR can decrease
      ↓
Achievable data rate can decrease
```

This is important for our project because the scheduler may need to react when a node's channel suddenly becomes worse.

---

## 8. Fading

Even when there is no major obstruction or rain event, the received signal can fluctuate.

This phenomenon is called **fading**.

Fading can happen because the signal can reach the receiver through different propagation paths and those signals can interact with each other.

For simulation purposes, fading can be represented using statistical channel models.

The exact fading model is something I need to understand further before deciding what should actually be implemented.

---

## 9. SNR

One of the most important channel-quality measurements is **SNR**.

SNR stands for:

**Signal-to-Noise Ratio**

It compares the strength of the useful signal with the amount of background noise.

In simple terms:

> SNR tells me how clearly the useful signal stands out from the noise.

A higher SNR generally means a better communication condition.

A lower SNR generally means a worse communication condition.

A simplified view is:

```text
High SNR
   ↓
Cleaner signal
   ↓
Better communication conditions

Low SNR
   ↓
Signal is closer to the noise level
   ↓
Worse communication conditions
```

---

## 10. Why does SNR matter to our scheduler?

Suppose I have two nodes:

```text
Node A → SNR = high
Node B → SNR = low
```

If I give both nodes the same amount of resources, Node A may be able to use those resources much more efficiently.

The scheduler can therefore use channel information such as SNR when deciding how to allocate resources.

This creates an important relationship:

```text
Channel condition
       ↓
      SNR
       ↓
Possible data rate
       ↓
Queue/service behaviour
       ↓
Scheduler decision
```

---

## 11. SINR

Another important measurement is **SINR**.

SINR stands for:

**Signal-to-Interference-plus-Noise Ratio**

SNR considers:

- Useful signal
- Noise

SINR considers:

- Useful signal
- Interference
- Noise

So:

```text
SNR  → Signal vs Noise

SINR → Signal vs Interference + Noise
```

SINR becomes particularly important when multiple transmissions can occur at the same time on different or shared frequency resources and interference needs to be considered.

For a more detailed communication simulation, SINR can therefore be more useful than SNR.

---

## 12. SNR/SINR and data rate

The important connection for me to understand is:

> Better channel quality generally allows a higher achievable data rate.

For example:

```text
Better SNR/SINR
       ↓
Higher achievable rate
       ↓
More data can be transmitted
       ↓
Queue can decrease faster
```

And:

```text
Worse SNR/SINR
       ↓
Lower achievable rate
       ↓
Less data can be transmitted
       ↓
Queue may grow
```

This is one of the reasons channel quality needs to be part of the network model.

---

## 13. Modulation and Coding

The communication system can choose different ways of representing data over the radio signal.

This involves **Modulation and Coding**, often represented together as an **MCS (Modulation and Coding Scheme)**.

The basic idea is:

### Good channel

A more efficient modulation/coding scheme can potentially be used.

```text
Good channel
    ↓
Higher-order / more efficient MCS
    ↓
Higher data rate
```

### Poor channel

A more robust scheme may be needed.

```text
Poor channel
    ↓
More robust MCS
    ↓
Lower data rate
```

Therefore, channel quality can affect the amount of data that can actually be transmitted.

For our project, I do not necessarily need to model a real military waveform or every physical-layer detail.

A simplified mapping between channel quality and achievable rate may be enough for the digital twin, depending on the required level of realism.

---

## 14. What the digital twin needs to represent

The communication simulation can represent the channel using factors such as:

- Distance between ACN and ground node
- Path loss
- Line-of-Sight conditions
- Rain attenuation
- Fading
- SNR/SINR
- Achievable data rate

The exact level of physical-layer detail should be chosen according to the project's requirements and available computational resources.

---

## 15. Example scenario

Imagine the ACN is communicating with four ground nodes.

Initially:

```text
Node 1 → Good channel
Node 2 → Good channel
Node 3 → Moderate channel
Node 4 → Good channel
```

The scheduler can allocate resources based on these conditions.

Now a rain cell affects Node 3:

```text
Before:

Node 3 → Good/Moderate SNR
             ↓
        High data rate


After:

Node 3 → Lower SNR
             ↓
        Lower data rate
```

If the scheduler is dynamic, it can notice that the network state has changed and adjust its allocation.

This is one of the scenarios that would be useful to test in the digital twin.

---

## 16. Why this makes the project more realistic

If every node always had a perfect and constant channel, the scheduling problem would be much simpler.

There would be less reason for the scheduler to continuously adapt.

By modelling changing channel conditions, the project can test whether the scheduling algorithm can respond to realistic changes.

For example:

```text
Normal conditions
       ↓
Rain starts
       ↓
Channel quality decreases
       ↓
Available data rate changes
       ↓
Queues change
       ↓
Scheduler adapts
```

---

## What I understand now

The communication channel is not fixed.

The quality of the link between the ACN and each ground node can change because of:

- Distance
- Line-of-Sight conditions
- Obstructions
- Rain
- Fading
- Interference
- Other propagation effects

SNR/SINR can be used to represent the quality of the communication link.

The channel quality affects the achievable data rate.

Therefore, channel conditions can affect how the scheduler should allocate resources.

---

## Important relationships I should remember

```text
Distance / Environment
        ↓
   Channel condition
        ↓
     SNR / SINR
        ↓
   Achievable rate
        ↓
 Amount of data served
        ↓
      Queue state
        ↓
 Scheduler decision
```

The scheduler and channel therefore interact with each other.

The scheduler makes an allocation, the nodes transmit according to their channel conditions, and the resulting network state becomes the input for the next scheduling decision.

---

## Things I still need to understand

- What exactly is path loss?
- How is SNR calculated?
- What is the difference between SNR and SINR in our simulation?
- How should rain attenuation be modelled?
- What fading model should we use?
- How do we convert SNR/SINR into data rate?
- What exactly is MCS?
- How realistic does our physical-layer model need to be?
- Which channel parameters are actually required for our project?
- Which parameters are just assumptions for the simulation?

---

## One-line understanding

> **The ACN-to-ground-node channel can change over time, and those changes affect SNR/SINR and therefore how efficiently each node can use the communication resources.**