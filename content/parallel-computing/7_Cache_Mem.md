---
layout: default
usemathjax: true
permalink: /parallel/ch7
---

Write propagation: Writes all become visible to other processors eventually.

Relaxed consistence
Reads can move in front of its ownwrite,

Total store ordering (TSO): P can read B 
reads by other processors cannot return new value  of A until the write to A is observed by all processors.
- if P1 writes A <- 1, P2 cannot read A == 1 before P3 can also read A == 1.

PC:
Can Return thevalue of any writebefore ht write isobserved by all processors.

Cache coherence:
- Singlememory location
- Program Order FOR EACH MEMORY LOCATION
- Write propagation FOR EACH MEMORY LOCATION
- Write serialization FOR EACH MEMORY LOCATION

Memory consistency:
- All different memory locations

Relaxed memory consistency:
- Allowed read or write ordering for a single memory location.
- STILL PRESERVES CACHE COHERENCE FOR EACH MEMORY LOCATION

Processor Consistency is cache coherent!
- WRite serialization: TOTAL ORDER for writes **for that memory location**
- This means that if the writes come from the sameprocessor, these writes will be seen in order
- But if they come from ddifferent processors, they may not be seen in order.

PSO:
- Preserves store order,
- Only diff between this and TSO is it allows W->W reorderings