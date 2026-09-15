# 6. Best Practices & Capstone

*What each field costs in bytes, when to batch by hand, where the trust boundary is, and the capstone.*

Blink · Hard

---

Everything so far was about making Blink work. This module is about making it good: what each field costs once it is on the wire, when to take the send loop into your own hands, where the trust boundary sits and what Blink does and does not check for you, and how to lay a network layer out before writing it. Then the capstone, where Crosswire's whole description gets written, wired and put behind CI.

## Lessons

1. [What it costs on the wire](01-bandwidth-budget.md) — One byte for the event, then exactly what your types said, and nothing else.
2. [Designing a network layer](02-designing-a-net-layer.md) — Decide direction, reliability and frequency for every message before you write a line.
3. [Validation and the trust boundary](03-validation-and-trust.md) — Blink checks every incoming field for you; WriteValidations checks the outgoing ones for you.
4. [Batching and manual replication](04-batching-and-replication.md) — Blink batches to the end of the frame, and ManualReplication hands you the flush.
