---
layout: default
usemathjax: true
title: Summary
permalink: /parallel/summary
---

# Shared Memory Systems

+ No need to partition data
+ More efficient communication opposed to distributed
- Synchronization constructs
- Lack of scalability due to memory contention

# Examples

**OpenMP**:

+ easy to add parallelism (by just adding compiler directives on top of C/C++ program)
+ No need to copy memory to a separate device
- heavyweight threads
- unrestricted resources: access has to be coordinated and synchronized

**CUDA**:

+ lightweight threads that are numerous, easy to create and destroy
+ reduce memory overheads and contention by exploiting good use of shared memory (only shared amongst threads)
- requires code that can run efficiently in lockstep and is slowed down by conditionals