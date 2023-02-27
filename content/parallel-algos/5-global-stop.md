---
layout: default
title: Global Snapshot
permalink: /palgo/ch5
---

# Global snapshot

A set of events $S$ such that if $e_2 \in S$ and $e_1 \rightarrow e_2$ (happens-before in **process** order)
then $e_1 \in S$.

**Consistent** global snapshot is one that if $e_2 \in S$ and $e_1 \rightarrow e_2$ (happens-before in **send-recv** order)
then $e_1 \in S$.