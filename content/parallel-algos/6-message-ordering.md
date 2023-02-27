---
layout: default
title: Message Ordering
usemathjax: true
permalink: /palgo/ch6
---

# Message Ordering

## Causal ordering

If a send event $s_1$ caused a send event $s_2, then causal order requires $r_1 < r_2$
($r_1, r_2$ on same process)

**How do we know** if $s_1$ causes $s_2$?

**Causal order**: If $s_1 < s_2$, and $r_1, r_2$ on the same process, then $r_1 < r_2$