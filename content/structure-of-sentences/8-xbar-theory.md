---
layout: default
usemathjax: true
title: X-bar theory Pt 1
permalink: /sos/ch8
---

# reading quiz

1. What type of constituent does one-replacement target? 
   1. N'

2. What type of constituent does do-so replacement target? 
   1. V

3. Which of the following is a property of the X-bar schema? 
   1. binary branching
   2. iterative/self-recursive
   3. endocentricity

4. Which of the following is an example of an intermediate projection? 
   1. X'

5. Which of the following is an example of a maximal projection? 
   1. XP

# X-bar theory

## Motivation

It is possible to replace a group of phrases that do not form a constituent.

> I bought the big [book of poems with the blue cover] but not the small [one].

![Embedded tree structure](/notes-blog/assets/img/sos/ch8-embedded-tree.png)

The "intermediate" N' (en-bar) categories to explain conjoined subsequences of items with the sisters of the same mother.

## Replacement types and rules

### Nouns

One-replacement: Replace an N'-node with *one*.

Caveat:
- doesn't usually work if determiner is 'a/an', because *one* is definite but 'a/an' is indefinite
- doesn't usually work with bare nouns, for same reason
- replace indefinite with definite and then *one* can replace.

$$
\begin{eqnarray*}
NP \rightarrow (D)\ N'\\
N' \rightarrow (AdjP)\ N'\\
N' \rightarrow N'\ (PP)\\
N' \rightarrow N\ (PP)\\
\end{eqnarray*}
$$

### Verbs

Do-so-replacement: Replace an V'-node with *do so*/*do so too*.

$$
\begin{eqnarray*}
VP \rightarrow V'\\
V' \rightarrow V'\ (PP)\\
V' \rightarrow V'\ (AdjP)\\
V' \rightarrow V\ (NP)\\
\end{eqnarray*}
$$

### AdjP

Is/Was-so-replacement: Replace an Adj'-node with *is so*/*was so too*.

$$
\begin{eqnarray*}
AdjP \rightarrow Adj'\\
Adj' \rightarrow (AdvP)\ Adj'\\
Adj' \rightarrow Adj'\ (PP)\\
Adj' \rightarrow Adj\ (NP)\\
\end{eqnarray*}
$$

## X-bar schema

All phrases appear to have **heads**.

The requirement is that phrases are headed are called **endocentricity**.

Non-heads are always **phrases** and **optional** in an X-bar rule.

Types of rules:
1. Rules that introduce heads
2. Rules that introduce recursive X' constituents together with optional phrase adjunct
3. Top level rules introducing the first X'

## Complements, Adjuncts, Specifiers

**Adjunct**: sister to a single bar level and daughter of single bar level
**Complement**: sister to a head and daughter of single bar level
    - ex for NPs this would be intro'd by the "of" preposition

Hence you can only have 1 complement as you can only have one X head in a XP.

## Projections

**Projection**: All elements introduced (i.e. on the left side of a rule) within an XP.

**Maximal Projection**: The topmost projection in a phrase (XP).

**Intermediate Projection**: Any projection that is neither the head nor the phrase (i.e., all the X' levels)

## Principle of Modification

If a YP modifies some head X, then YP must be either a sister of

1. X
2. An (upwards) projection of X (X', XP)