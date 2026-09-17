# Topic Comment Elaboration — Technical Operational Dispatcher

**Framework Author**: William Fitzpatrick (*Writer Science*)  
**Source Lecture**: [The Most Powerful Writing Frameworks to Write CLEARLY](https://www.youtube.com/watch?v=cayUBPHB1HQ)  
**Parent Collection**: [Master Collection](../../kirby-fitzpatrick-writers-collection/SKILL.md) | [Global Help](../../../kirby-help/SKILL.md)  

---

## 1. Cognitive Foundation: The Doorstep and the Room

### 1.1 The mechanism in one sentence

Every clause splits into two functional slots, and the reader's brain treats them as two different jobs.

* **Topic (the doorstep)** — the sentence-initial slot: the grammatical subject, or the fronted adverbial occupying that position. It is a **retrieval cue**. It names something the reader can already reach: an entity introduced earlier in the document, a component the audience already knows, or the payload the previous sentence just handed over.
* **Comment (the room)** — everything after the topic: predicator, objects, complements, trailing modifiers. It is the **payload**. It carries the novel material — the mutation, the constraint, the invariant, the architecture detail that did not exist in the reader's model until this moment.

The rule is not stylistic. It is a **memory-management contract**: *old information states the address; new information states the value written to that address.* Violate it and the reader must hold the novel material in buffer while searching for an anchor that never arrived — the exact sensation engineers report as "I had to read that sentence three times."

### 1.2 The three-step read cycle

```text
For every clause the reader executes:

  (1) RETRIEVE   look up the Topic referent  ─────────► hit?  cheap (~ms)
                                                        miss? reader backtracks
  (2) BIND       attach the Comment payload to that referent
  (3) COMMIT     store the result as the new known state ──► becomes the Topic of the next clause

  Cost is paid almost entirely at step (1); the payload lands at step (2).
  A clause that buries novel material on the doorstep pays BOTH — twice.
```

### 1.3 The two states of a reader's memory

```text
┌───────────────────────── READER WORKING MEMORY ──────────────────────────┐
│  ANCHORS (cheap to reactivate)        │  PAYLOADS (expensive to place)   │
│  · PaymentRouter class                │  · idempotency-key derivation    │
│  · StripeClient                       │  · 409 conflict escalation       │
│  · the retry queue                    │  · tenant-scoped salting         │
│  · webhook delivery path              │  · exponential backoff ceiling   │
└──────────────────────────────────────────────────────────────────────────┘
             ▲                                          ▲
             │                                          │
      TOPIC slot                                  COMMENT slot
   (doorstep: known, resolves)                 (room: novel, gets written)
```

### 1.4 Before vs after

```text
❌ BEFORE — Novel material parked on the doorstep; no anchor, no binding

  "Hashing the canonicalized request body with the tenant salt during idempotency-key
   derivation is what the retry queue does before it tolerates duplicate webhook deliveries."
   └────────────────────── NOVEL, UNANCHORED ──────────────────────┘
        The reader is handed the payload first and asked to find the subject last.

✅ AFTER — Known anchor on the doorstep, novel payload in the room

  "The retry queue | derives its idempotency key | by hashing the canonicalized request
   └── TOPIC ──┘    └──────────── COMMENT (novel architecture detail) ─────────────┘
    (known, resolves)              (binds cleanly to the anchor)

  "That key | scopes to the tenant salt, so a replayed webhook cannot collide across accounts."
   └─ TOPIC ─┘   └──────────────── COMMENT: the mutation's consequence ────────────────┘
       (the previous Comment, now promoted to Topic — the relay handoff)
```

### 1.5 Why this is acute in software documentation

Code objects are *long-lived nouns with unstable semantics*. `PaymentRouter` exists in the reader's model before your document ever mentions it — but what it *does to idempotency keys* is what you are actually there to say. Codebase comprehension is reference resolution across files, so a documentation sentence that opens on an unanchored novel phrase demands a lookup the reader cannot complete from context. The cost compounds: one unbound sentence in a class docstring propagates into every caller that trusts it. Topic–Comment syntax is the cheapest available fix, because it changes no facts — only the order in which the facts arrive.

### 1.6 Boundary discipline

Topic–Comment Elaboration is a **within-clause and within-paragraph** protocol. Two neighbouring concerns are out of scope and belong to sibling skills: end-to-start chaining across consecutive sentences ([Linear Relay Linking](../../kirby-fitzpatrick-linear-relay-linking/SKILL.md)), and keeping the material before the topic under five words ([Runway Syntax Engine](../../kirby-fitzpatrick-runway-syntax-engine/SKILL.md)).

---

## 2. Core Transformation Protocols

### Rule 1 — The Retrievability Gate (Topic Eligibility)
The sentence-initial slot admits only referents that satisfy at least one of three tests:
1. **Introduced** — named earlier in this document.
2. **Common ground** — a language keyword, a platform primitive, a domain term the audience owns (`HTTP`, `mutex`, `request body`).
3. **Inherited** — the Comment of the immediately preceding sentence.

Anything else is **topic smuggling**: novel architecture parked in the subject slot. Demote the novel material to the Comment and promote a known entity into the Topic.

### Rule 2 — One Door Per Clause (Single-Topic Discipline)
A clause declares **one** subject. Forbid mid-clause topic switches of the form `…, while the config loader, meanwhile, …`. Two topics in one clause force the reader to hold two uncommitted bindings at once. Sever the clause in two; the second clause's Topic is the first clause's Comment.

### Rule 3 — Payload Weighting (Comment Density)
The Comment slot carries the highest-information-density material in the sentence; the Topic slot carries the lowest. If the topic noun is more exotic than the predicate, the sentence is inverted. Test: swap the slots and read both versions — the one that front-loads the unfamiliar noun always loses.

### Rule 4 — Topic Chain Continuity
Hold one Topic across a run of clauses when documenting a single artifact; a Topic switch is a **structural signal**, not a stylistic variation. When you do switch, the new Topic must be the prior Comment (per Rule 1) or explicitly re-anchored with a retrieval phrase (*"That same queue…"*, *"The router, in turn, …"*).

### Rule 5 — Fronted Material Is Still Topic
Adverbials and prepositional phrases in first position occupy the topic slot whether you intend it or not — they are what the reader tries to retrieve first. Keep fronting to scoping words (*By default*, *In production*, *Under retry pressure*). A 15-word fronted condition is an unanchored pseudo-topic and a runway violation at the same time.

### Rule 6 — The API Calling Convention
Document every mutation as *anchor → action → effect*: name the known component, state the verb the component performs, then extend into the novel details. This inverts the natural tendency to lead with the mechanism.

### Rule 7 — Comment Completeness
A sentence that is all Topic and no Comment (pure identification, pure restatement) spends attention and writes nothing into the reader's model. Delete it, or give it a payload.

### 2.1 Transformation Table

| # | Anti-Pattern (Violation) | Defect | Clean Replacement (Topic → Comment) |
|---|---|---|---|
| 1 | *"Parsing the YAML schema through a two-pass validator is what `ConfigLoader` does."* | Novel mechanism on the doorstep; the anchor arrives after the reader needed it. | *"`ConfigLoader` parses the YAML schema in two passes."* |
| 2 | *"The `SyncManager` flushes the queue, while the audit log, meanwhile, buffers to disk."* | Two topics in one clause; neither binding commits. | *"`SyncManager` flushes the queue. The audit log buffers to disk in parallel."* |
| 3 | *"Tenant isolation is enforced by row-level policies. `ReportBuilder` also gained a cached path."* | Unannounced topic jump between sentences; no relay from Comment to Topic. | *"Tenant isolation is enforced by row-level policies. Those policies drive `ReportBuilder`'s new cached path."* |
| 4 | *"When evaluating how the scheduler handles fairness across queues under sustained contention, the scheduler starves low-priority jobs."* | Fronted condition overloads the topic slot; the real subject is buried. | *"Under sustained contention, the scheduler starves low-priority jobs."* |
| 5 | *"The performance characteristics of the new B-tree index are what makes `find_by_email` fast."* | Nominalized Comment dumped into the Topic slot; the verb payload is demoted. | *"The new B-tree index makes `find_by_email` fast."* |
| 6 | *"`AuthService` is the service responsible for authentication."* | All Topic, zero Comment — attention spent, model unchanged. | *"`AuthService` mints short-lived JWTs and rejects refresh tokens replayed after rotation."* |
| 7 | *"It retries three times before surfacing the error, and it also emits a metric."* | Pronoun Topic with no resolvable antecedent in the preceding sentence. | *"The upload worker retries three times before surfacing the error, then emits a `retry_exhausted` metric."* |
| 8 | *"There is a check that prevents duplicate webhook deliveries."* | Expletive `There is` occupies the topic slot; the agent is hidden. | *"The `StripeClient` deduplicates webhook deliveries on the event ID."* |

### 2.2 Repair Procedure (60-second pass)
1. Underline the first noun phrase of every sentence. **Topic.**
2. Circle everything after it. **Comment.**
3. For each Topic: is it retrievable (Rule 1)? If not, find the earlier Comment that owns it and promote that entity — or re-anchor explicitly.
4. For each Comment: does it add at least one fact the document did not already contain (Rule 7)? If not, delete the sentence.
5. Re-read only the Topics in order. They should read as a valid table of contents for the artifact you are documenting.

---

## 3. Engineering Application Scenarios

### 3.1 Code Reviews
A review comment is a sentence whose Topic must be *the code the reviewer is looking at*. The most common review-writing failure is opening with the reviewer's reasoning instead of the code's state, which forces the author to re-derive what `this` refers to.

* **Violation**: *"Because there is a race between the writer goroutine and the flush timer, and having looked at how `Close` is called from the shutdown path, I don't think this is safe."*
* **Repair**: *"`Close` can race the flush timer when the shutdown path calls it twice. Guard the flush with the existing mutex, or drain before closing."*
* **Thread hygiene**: when a thread covers several concerns, keep one Topic per comment. A comment that opens on `Close` and then pivots to "also, the naming here is off" is two bindings in one send; split it so each concern keeps its anchor.
* **Severity framing**: name the known artifact first, then the novel defect in the Comment — *"The `retryBudget` counter resets per request, so a failure loop never exhausts it."* The author resolves the reference instantly and reads the defect as a mutation of a component already in memory.

### 3.2 PR Descriptions
A PR description is read by an engineer who knows the repo's *existing* modules and knows nothing about your *new* ones. That asymmetry is exactly the Topic/Comment axis: anchor on what existed before the branch, deliver the mutation as the payload.

```text
❌ "A new `TenantSaltProvider` interface plus the hashing change are in, which changes the
    retry queue's dedup semantics, and the schema migration adds the salt column."

✅ "The retry queue now derives idempotency keys per tenant.          ← known → mutation
    To do that, this PR adds a `TenantSaltProvider` interface …       ← Comment becomes the new door
    The migration adds a `tenants.salt` column, backfilled …"
```

* **Skeleton**: `What existed` → `What now happens to it` → `How` → `Verification`. Each paragraph's Topic is the previous paragraph's Comment.
* **Risk sections**: open with the known component, never the abstract hazard — *"The `PaymentRouter` p99 rises ~40 ms during migration because…"*, not *"There is a potential latency concern with the migration."*
* **Reviewer routing**: an unanchored Topic is also a review-assignment bug. If the first sentence's Topic is a module the reviewer does not own, the description lands on the wrong specialist's desk.

### 3.3 Architecture RFCs / ADRs
RFC sections are read out of order and months later, so each section must re-establish its Topic from a shared anchor instead of relying on reading momentum.

* **ADR framing maps 1:1**: *Context* = the known state (Topic); *Decision* = the mutation (Comment); *Consequences* = re-promoting the decision to a Topic and commenting on its effects.
* **Violation**: *"Given the scale-out requirement and the goal of eventually removing the cron, a queue-based re-index is better than the trigger approach, which has locking problems."*
* **Repair**: *"The nightly cron re-index holds a table lock for ~11 minutes. Under the 4× growth target it becomes the write-path bottleneck. We replace it with a queue-driven re-index…"* — existing component on the doorstep, novel decision in the room.
* **Options comparison**: give every option the *same* Topic so the reader's memory address is stable across the comparison — *"Option A caps the queue at …; Option B caps the queue at …"*. Variants that each open on a different noun force a re-anchor per bullet, which is where readers stop comparing and start skimming.
* **Migration sections**: Topic = the currently deployed shape; Comment = the new shape. Never open a migration step with the target state, or operators will apply step 3 to a system that is still in step 1.

---

## 4. Verification Checklist

- [ ] **Topic retrievability** — Is every sentence-initial noun phrase either introduced earlier, common ground, or the previous sentence's Comment?
- [ ] **Single topic per clause** — Does any clause carry two subjects, or pivot to a new one without re-anchoring?
- [ ] **Payload in the right slot** — Could any Comment be promoted to Topic (or the reverse) so that the unfamiliar noun arrives second?
- [ ] **Comment non-emptiness** — Does every sentence add at least one fact the document did not already contain?
- [ ] **Topic table of contents** — Read only the Topics in sequence: do they form a coherent outline of the artifact, with switches landing only at genuine section boundaries?