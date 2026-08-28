# 🗄️ SQL Mastery — 20+ Years Experience
# Module 20 — High Availability

> Replication gives you copies of your data. High availability is about what happens the moment the primary actually dies — who takes over, how fast, and how you avoid two machines both thinking they're in charge.

---

# Chapter 112 — Database Failover: From Primary Failure to New Primary

> Failover is the single most operationally important sequence of events in a highly-available database system — and it's far more subtle than "just promote a replica."

## Learning Objectives
- What failover actually involves, step by step
- Detection vs promotion vs redirection
- Automatic vs manual failover
- Data loss risk during failover
- Why failover testing matters

## 1️⃣ The Basic Shape

```text
Primary
   ↓
Replica
```

Primary fails:

```text
X Primary

Replica
   ↓
New Primary
```

## 2️⃣ Failover Is Actually Several Distinct Steps

```text
Step 1 — Detection
   "Is the primary actually down, or just slow/network-partitioned?"

Step 2 — Selection
   "Which replica should become the new primary?"
   (usually the one with the least lag / most caught-up data)

Step 3 — Promotion
   "Convert that replica from read-only follower
    into a fully writable primary."

Step 4 — Redirection
   "Get applications, proxies, and DNS/connection strings
    pointing at the new primary."

Step 5 — Reconfiguration
   "Point remaining replicas at the new primary
    for ongoing replication."
```

Each step has its own failure modes — a naive failover implementation that skips or rushes any of these can create worse problems than the original outage.

## 3️⃣ Detection Is Harder Than It Sounds

```text
"Primary isn't responding"
   could mean:
      Primary actually crashed
      Primary is alive but network-partitioned
      Primary is alive but severely overloaded/slow
```

Failing over when the primary is merely network-partitioned (but still alive and accepting writes from its own perspective) is exactly how split-brain scenarios (Chapter 114) happen — this is why naive "ping fails, promote immediately" logic is dangerous.

## 4️⃣ Automatic vs Manual Failover

```text
Automatic failover
 ↓
Faster recovery, no human in the loop
 ↓
Risk: can fail over incorrectly on false positives
   (transient network blip mistaken for a real failure)

Manual failover
 ↓
Human judgment before an irreversible action
 ↓
Risk: slower recovery, dependent on someone being available
   and paying attention
```

Many production systems use a hybrid: automated detection and preparation, but a deliberate confirmation gate (or a very carefully tuned automatic threshold) before actually promoting.

## 5️⃣ Data Loss Risk During Failover

```text
Asynchronous replication (Chapter 110)
        ↓
The most caught-up replica might STILL be missing
   the most recent writes the old primary accepted
   right before it failed
        ↓
Promoting that replica means those writes are lost
```

This is a direct, concrete consequence of the sync/async trade-off from Chapter 110 — the replication mode you chose earlier determines exactly how much data loss is possible during failover.

## 6️⃣ Why Failover Must Be Tested, Not Just Configured

```text
"We have automatic failover configured"
```

is a claim about mechanism, not a verified guarantee — directly analogous to Chapter 98's point about crash recovery. Deliberately triggering a failover in a controlled, non-production (or carefully controlled production) exercise is the only way to know it actually works as intended, including how applications behave during the transition.

## 🧠 Senior Mental Model

```text
Failover isn't a single action.
It's a sequence: detect → select → promote → redirect → reconfigure.
Every step can fail independently, and untested failover
   is a theory, not a capability.
```

## 🧪 Hands-on Lab

**Lab 1** — In a test environment, manually walk through each failover step (detect, select, promote, redirect, reconfigure) rather than relying on automation, to understand each piece individually.
**Lab 2** — Simulate a network partition (rather than a hard crash) between primary and monitoring/replicas, and observe whether your failover mechanism behaves safely.
**Lab 3** — Under asynchronous replication, write to the primary, immediately simulate a crash, and check whether the promoted replica is missing the most recent write.

## 🎯 Interview Questions

**What are the distinct steps involved in a database failover?**
Detection (confirming the primary is actually down), selection (choosing the best-positioned replica), promotion (making it writable), redirection (pointing applications/proxies at it), and reconfiguration (repointing remaining replicas).

**Why is detecting a "true" primary failure harder than it sounds?**
Because an unresponsive primary could mean an actual crash, a network partition where the primary is still alive and accepting writes, or severe overload — treating all of these identically risks incorrect failover and split-brain.

**Why can data loss occur during failover even with replication in place?**
Because under asynchronous replication, even the most caught-up replica may not have received the very latest writes the old primary accepted right before failing — promoting that replica effectively loses those writes.

## 📌 Summary

```text
Rule 1 — Failover is a multi-step sequence, not a single action.
Rule 2 — Detecting a true failure vs. a network partition is genuinely hard.
Rule 3 — Automatic and manual failover trade speed against safety.
Rule 4 — Asynchronous replication makes some data loss during failover possible.
Rule 5 — Failover must be tested deliberately, not just configured and trusted.
```

---

# Chapter 113 — Leader Election: Database Failover Meets Distributed Consensus

> Deciding "who becomes the new primary" when multiple nodes might independently believe they should be in charge is, at its core, the same problem distributed systems have grappled with for decades.

## Learning Objectives
- The relationship between failover and leader election
- Core distributed consensus vocabulary
- Why a simple "first one to notice" approach fails
- Terms/epochs as a safety mechanism

## 1️⃣ The Connection

```text
Database Failover
        ↕
Distributed Consensus
```

"Which replica should become the new primary?" is fundamentally a leader election problem — the same class of problem solved (in various forms) by consensus algorithms across distributed systems generally, not just databases.

## 2️⃣ Core Vocabulary

```text
Leader
 ↓
The single node currently authorized to accept writes

Followers
 ↓
Nodes replicating from the leader, not currently authorized to write

Quorum
 ↓
A minimum number of nodes that must agree
   before a decision (like electing a new leader) is considered valid

Election
 ↓
The process of choosing a new leader when the current one
   is unreachable or has failed

Term / Epoch
 ↓
A monotonically increasing number identifying
   "which leadership period" is currently in effect
```

## 3️⃣ Why "First One to Notice" Fails

```text
Naive approach:
   "Whichever replica notices the primary is down first,
    promotes itself."
```

```text
Problem:
   Multiple replicas might notice around the same time,
   independently promote themselves,
   and now there are TWO primaries.
```

This is exactly the split-brain scenario explored in depth in Chapter 114 — proper leader election protocols exist specifically to prevent this.

## 4️⃣ Terms/Epochs as a Safety Mechanism

```text
Leadership Term 5 → old primary
        ↓
Old primary fails
        ↓
New election → Leadership Term 6 → new primary
```

If the old primary somehow comes back online still believing it's in Term 5, other nodes (aware that Term 6 is now current) can recognize and reject its stale authority — this monotonic term/epoch numbering is a core building block of safe leader election.

## 5️⃣ Quorum's Role in Election Safety

```text
Election requires agreement from a MAJORITY of nodes,
   not just a single node's opinion
```

This ensures that even if the network is partitioned in a way that lets some nodes talk to each other but not others, at most one side of the partition can actually gather a majority and elect a leader — the other side simply cannot proceed, rather than proceeding incorrectly. (Explored further in Chapter 115.)

## 🧠 Senior Mental Model

```text
"Which node should be primary now?"
is not a database-specific question —
it's a distributed consensus problem
that databases happen to need an answer to.
```

## 🧪 Hands-on Lab

**Lab 1** — Research which consensus mechanism (if any) your specific database or its associated HA tooling uses for leader election.
**Lab 2** — Discuss, on paper, why a naive "first to notice, self-promote" approach can lead to two primaries existing simultaneously.
**Lab 3** — Trace through a term/epoch numbering scenario: old primary returns after being replaced — how does the system recognize its authority is stale?

## 🎯 Interview Questions

**Why is choosing a new database primary fundamentally a leader election problem?**
Because it requires distributed agreement on which single node is authorized to act as leader (accept writes), which is the same class of problem addressed generally by distributed consensus, independent of databases specifically.

**What role do terms/epochs play in safe leader election?**
They provide a monotonically increasing identifier for "which leadership period is current," allowing other nodes to recognize and reject a stale leader (e.g., an old primary that returns believing it's still in charge) based on an outdated term number.

**Why does requiring a quorum (majority) matter for election safety?**
Because it ensures that at most one side of any network partition can gather enough agreement to elect a leader, preventing two independent groups of nodes from each electing their own leader simultaneously.

## 📌 Summary

```text
Rule 1 — Choosing a new primary is a leader election / consensus problem.
Rule 2 — "First to notice, self-promote" can create multiple primaries.
Rule 3 — Terms/epochs let nodes recognize and reject stale leadership.
Rule 4 — Quorum requirements prevent multiple simultaneous elections from succeeding.
```

---

# Chapter 114 — Split Brain: When Two Nodes Both Think They're Primary

> This is the nightmare scenario every HA design must specifically defend against — and understanding exactly how it happens is the first step to preventing it.

## Learning Objectives
- What split-brain actually is
- How it happens mechanically
- Why it's catastrophic, not just inconvenient
- Fencing and other mitigation techniques
- Why quorum alone isn't automatically sufficient without careful implementation

## 1️⃣ The Scenario

```text
Node A thinks it is Primary
Node B thinks it is Primary
```

Both are simultaneously accepting writes, believing themselves to be the sole source of truth.

## 2️⃣ How It Actually Happens

```text
Network partition occurs
        ↓
Node A can't reach Node B (and vice versa)
        ↓
Node A concludes: "B must be down, I should take over"
        ↓
Node B concludes: "A must be down, I should take over"
        ↓
BOTH promote themselves
        ↓
Two primaries, each accepting independent writes
```

Neither node did anything obviously "wrong" in isolation — each made a locally reasonable decision based on incomplete information (it can't distinguish "the other node is down" from "I just can't reach the other node").

## 3️⃣ Why It's Catastrophic, Not Just Inconvenient

```text
Client A writes to "Primary" A
Client B writes to "Primary" B
        ↓
Two divergent, conflicting versions of the data
        ↓
Network partition heals
        ↓
Which version is correct?
        ↓
Often: no clean automatic answer — manual reconciliation,
   potential permanent data loss of one side's writes
```

This is fundamentally worse than an outage — an outage is recoverable by definition once the underlying issue is fixed; split-brain can produce genuinely irreconcilable, silently divergent data.

## 4️⃣ Fencing — A Common Mitigation

```text
Fencing
 ↓
Actively prevent the OLD primary from continuing to
   accept writes once a new primary has been elected —
   e.g., by revoking its access to shared storage,
   forcibly terminating it, or blocking its network access
```

The goal is to make it *physically impossible* for the old primary to keep acting as primary, rather than merely hoping it will politely notice and step down on its own.

## 5️⃣ Quorum Helps, But Implementation Details Matter

```text
Quorum-based election (Chapter 113)
 ↓
Reduces split-brain risk significantly
   IF correctly implemented
```

```text
But:
   Misconfigured quorum size
      (e.g., requiring only 1-of-2 nodes to "win")
   Or a monitoring/witness node that itself
      becomes unreachable in a way that breaks the math
```

can still allow split-brain to occur despite quorum being conceptually "in place." This is why HA configuration details deserve careful, specific review — not just "we use quorum, so we're safe."

## 🧠 Senior Mental Model

```text
Split-brain isn't caused by a node behaving irrationally.
It's caused by two nodes making individually reasonable
decisions with incomplete, partitioned information.

Defending against it requires MAKING incorrect leadership
   impossible (fencing), not just making it unlikely.
```

## 🧪 Hands-on Lab

**Lab 1** — Simulate a network partition between two nodes in a test HA setup and observe whether both attempt to become primary.
**Lab 2** — Research what fencing mechanism (if any) your specific HA tooling uses, and how it actually prevents a stale primary from continuing to write.
**Lab 3** — Review your quorum configuration and confirm the actual number of nodes required for a valid election — check for edge cases (even node counts, single witness nodes) that could weaken the guarantee.

## 🎯 Interview Questions

**What causes split-brain, mechanically?**
A network partition causes two nodes to each independently and reasonably conclude the other has failed, leading both to promote themselves to primary simultaneously, since neither can distinguish "the other node is down" from "I simply can't reach it."

**Why is split-brain considered worse than a simple outage?**
Because it can produce two divergent, independently-written versions of the data that may not be automatically reconcilable once the partition heals, resulting in genuine, potentially permanent data loss or corruption — an outage alone is recoverable once fixed.

**What is fencing, and why is it necessary even with quorum-based election?**
Fencing actively prevents a demoted or stale primary from continuing to accept writes (e.g., by cutting its storage or network access), making incorrect leadership physically impossible rather than just unlikely — necessary because quorum implementation details or edge cases can still occasionally allow split-brain otherwise.

## 📌 Summary

```text
Rule 1 — Split-brain arises from reasonable decisions made with partitioned information.
Rule 2 — It can produce irreconcilable divergent data, worse than a plain outage.
Rule 3 — Fencing makes incorrect leadership physically impossible, not just unlikely.
Rule 4 — Quorum reduces risk but must be correctly configured to actually prevent it.
```

---

# Chapter 115 — Quorum: Why Majority-Based Decisions Matter

> Quorum is the mathematical backbone that makes safe leader election and split-brain prevention actually work — this chapter makes the reasoning concrete.

## Learning Objectives
- What quorum means precisely
- Why majority (not just "some agreement") is the key property
- Odd vs even node counts
- Quorum and availability trade-offs
- Witness/arbiter nodes

## 1️⃣ The Core Definition

```text
N nodes total
Quorum = a MAJORITY of N
   (more than N/2)
```

## 2️⃣ Why Majority Specifically Matters

```text
Claim: In any network partition, at most ONE side
   can contain a majority of the N total nodes.
```

```text
Proof sketch:
   If Side A has a majority (> N/2)
   Side B has the remainder (< N/2)
   Side B mathematically CANNOT also have a majority.
```

This is the entire safety guarantee in one sentence: majority-based quorum makes it mathematically impossible for two independent partitions to both successfully elect a leader at the same time.

## 3️⃣ Odd vs Even Node Counts

```text
3 nodes → majority = 2
   A 1-vs-2 partition: the side with 2 can proceed,
      the side with 1 cannot. Safe.

4 nodes → majority = 3
   A 2-vs-2 partition: NEITHER side has a majority.
      Neither can proceed. Safe, but less available
      (the whole system stalls until the partition heals).
```

Odd node counts are generally preferred specifically because they avoid the possibility of an exact, unresolvable 50/50 split — every possible partition has a clear majority side (or no side has one, which is also safe, just unavailable).

## 4️⃣ Quorum and Availability Trade-offs

```text
Larger quorum requirements
 ↓
Stronger safety guarantee against split-brain
 ↓
But: more nodes must be reachable for ANY decision
      (including legitimate failover) to proceed
 ↓
Lower availability during partial outages
```

```text
Smaller quorum requirements
 ↓
Easier to make progress during partial outages
 ↓
But: weaker protection against split-brain
```

This is a direct instance of the broader CAP theorem trade-off (Chapter 127) — quorum size is one of the concrete knobs where that trade-off actually gets implemented.

## 5️⃣ Witness / Arbiter Nodes

```text
2 "real" data-bearing nodes
        +
1 lightweight witness/arbiter node
   (doesn't hold data, just participates in voting)
        ↓
3 total voters → majority = 2
   → avoids a pure 1-vs-1 tie between the two real nodes
```

This is a common, cost-effective pattern for achieving odd-count quorum safety without needing a full third data-bearing replica.

## 🧠 Senior Mental Model

```text
Quorum isn't a bureaucratic formality.
It's the mathematical guarantee that makes
"exactly one leader at a time" provably true,
even during a network partition.
```

## 🧪 Hands-on Lab

**Lab 1** — For a 3-node and a 4-node cluster, enumerate every possible partition split and determine which side (if any) retains quorum in each case.
**Lab 2** — Research whether your HA setup uses a witness/arbiter node, and how it's counted toward quorum.
**Lab 3** — Discuss the availability impact of a 2-vs-2 partition in a 4-node cluster where neither side has quorum.

## 🎯 Interview Questions

**Why does requiring a strict majority (not just "some" agreement) guarantee split-brain safety?**
Because in any partition, at most one side can mathematically contain more than half of the total nodes — so at most one side can ever gather enough votes to safely elect a leader, making dual leadership impossible.

**Why are odd node counts generally preferred for quorum-based systems?**
Because they avoid the possibility of an exact, unresolvable even split where neither side has a majority, ensuring every partition scenario has a clear resolution (one side proceeds, or neither does, cleanly).

**What is a witness/arbiter node and why is it useful?**
A lightweight node that participates in quorum voting without holding a full copy of the data, commonly used to achieve an odd total vote count (avoiding ties) without the cost of a full additional data-bearing replica.

## 📌 Module 20 Critical Rules

```text
Rule 1  — Failover is a multi-step sequence: detect, select, promote, redirect, reconfigure.
Rule 2  — Distinguishing a true failure from a network partition is genuinely hard.
Rule 3  — Choosing a new primary is fundamentally a leader-election/consensus problem.
Rule 4  — Terms/epochs let nodes reject stale leadership claims.
Rule 5  — Split-brain arises from reasonable decisions made with partitioned information.
Rule 6  — Split-brain can cause irreconcilable data divergence — worse than an outage.
Rule 7  — Fencing makes incorrect leadership physically impossible, not just unlikely.
Rule 8  — Quorum (majority) mathematically prevents two partitions from both electing a leader.
Rule 9  — Odd node counts avoid unresolvable ties; witness nodes help achieve this cheaply.
Rule 10 — Quorum size is a direct, tunable instance of the availability/safety trade-off.
```

---

# 🎉 Module 20 Complete

You now understand what actually happens — and what must be carefully prevented — when a primary database fails:

```text
Failover (the multi-step process)
 ↓
Leader Election (choosing safely, with terms/epochs)
 ↓
Split-Brain (the failure mode to prevent)
 ↓
Quorum (the mathematical guarantee that prevents it)
```

# 🚀 Next Module

# Module 21 — Database Scaling
Vertical scaling limits, read scaling, write scaling, connection pooling, and backpressure under overload.