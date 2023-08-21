---
layout: default
usemathjax: true
title: Constituency, Trees, Rules, Structural Relations
permalink: /syntax/ch2
---

# Constituency

Sentences are represented concretely as a linear string of words.

However we parse them in a hierarchical tree structure, with each of the nodes being constituents.

> Experimental evidence: Merrill Garrett's click experiments.
>
> - Clicks are placed in a neutral position in the sentence
> - Participants tended to perceive the clicks at constituent boundaries instead of the actual word's position.

## Phrase Structure Rules

Constituents are defined by rules that dictate what sort of components they can comprise of.

They are written in the forms:

$$
\begin{align}
XP &\rightarrow X \ Y \ Z \\
 XP &\rightarrow (Opt) \ X \\
 XP &\rightarrow (MinOnce+) \ X \\
\end{align}
$$

## Heads of constituents

The head of a phrase gives its phrase its category.

In $XP \rightarrow X \ Y \ Z$, $X$ is the head of the phrase $XP$.

## Principle of Modification

Informal formulation: Modifiers are always attached to the phrase they modify.

- (A (*big*) (yellow) book)
- (A (*very* yellow) book)

Formal formulation:
If an XP **modifies** some **head** Y, 
then XP must be a **sister** to Y (i.e., a daughter of YP).

## Clauses

Structure of the *sentence* or **tense phrase**.

$$
TP \rightarrow NP \ (T) \ VP
$$

Clauses can be **embedded**. This requires a complementizer

$$
CP \rightarrow (C) \ TP
$$

Can the head of a phrase be optional??

### Relative clauses

CPs can also modify N.

> The man [whose car I hit __ last week] sued me.

**Gap** is the "extraction site" from which the NP "var" after the [wh-word](/notes-blog/sos/ch13#wh-questions) is the **filler**.

$$
RP \rightarrow (R) \ NP \ (NP) \ VP
$$

## Coordination/Conjunction

$$
\begin{align}
X &\rightarrow X \ Conj \ X \\
XP &\rightarrow XP \ Conj \ XP
\end{align}
$$

## Recursion

Observe that rules are allowed to be recursive.

This accounts partially for the infinite nature of language.

It is an active area of research to find languages not demonstration recursion.

# Constituency Tests

Purpose: We should find instances where **groups of words** behave as **single units**.

Tests are such instances (where word groups can be swapped out for substitutes).

- Replacement
  - Replace with pro-form
- Sentence fragment (Q&A)
  - Question and ans with sentence fragment.
- Movement
  - Sentence Cleft: "Y X" $\Rightarrow$ "It was X that Y"
  - Pseudoclefting (aka preposing): "Y X" $\Rightarrow$ "X is what Y"
  - Passivization: "X Y Z" $\Rightarrow$ "X was Y by Z"
- Ellipsis:
  - Elide (remove) a string
- Coordination: $X \rightarrow X \ Conj \ X$
  - warning: often doesn't work

Tests may not always be conclusive, or they may fail.
This is because of certain quirks of the individual tests.

e.g. 

- The largest photo of a bagel costs a lot
- It is a bagel [that the largest photo of _] costs a lot

The subject island: constituents cannot be moved out of the subject island.

> [Bruce loved] and [Kelly hated] phonology class.

"Bruce loved" and "Kelly hated" are not constituents.

This is because "phonology class" is elided from "Bruce loved".

e.g.
> Zheng put the bagel [in the freezer].
> It was [in the freezer] that Zheng put the bagel.
> Zheng put there bagel [there].
> Where did Zheng put the bagel? [In the freezer].

> Zheng put [the bagel in the freezer].
> *It was [the bagel in the freezer] that Zheng put.
> *Zheng put [it].
> *[The bagel in the freezer] was what Zheng put.

Constituents of different categories behave differently.

1. English has VP ellipsis but not NP ellipsis
   1. Zheng cannot eat bagels, but Grace can ~~[eat bagels]~~.
   2. Zheng cannot eat bagels, but Grace can eat ~~[bagels]~~.

## Heads of constituents (cont.)

### Head ordering

Left-headed languages tend to have $X$ appear to the left within $XP$s,
right-headed languages tend to have them appear to the right.

There are also polysynthetic and free-word order languages.

- Polysynthetic:
  - All sentences are replaced with very long compound words (morphology)
  - Morphological order of morphemes
- Free word order:
  - Due to addition of additional marker morphemes
  - we find that there is one “neutral” order

# Structural Relations

Sentences represented as a hierarchical order of units are geometric objects.

If syntactic trees are geometric objects, they can be studied and described mathematically – the focus of this chapter.

1. Branch = Edge
2. Node 
   1. Root node
   2. Terminal nodes
   3. Non-terminal nodes
3. Label = Key

## Generative linguistics

1. A formal linguistics
2. 

## Domination

1. Domination: `Parent*` relation
   1. Note: Proper domination is irreflexive, whereas domination is reflexive (A dominates A). WE ASSUME PROPER DOMINATION.
   2. Domination is transitive
2. Exhaustive domination: `Parent*OfAll` relation. A exhaustively dominates a set of nodes U iff U is a set of terminal nodes and A dominates every $U \in A$.
   1. Exhaustively domination: 
3. Immediate domination: `Parent relation`
   1. Note: Immediate domination is **not** transitive
4. Constituent: B is a constituent of A $\Leftrightarrow$ A dominates B

## Precedence

1. Sister precedence: A sister-precedes B $\Leftrightarrow$ A and B immediately dominated by same node and A appears to the left of B.
2. Precedence: A precedes B $\Leftrightarrow$ A and B do not dominate each other and A or some node dominating A appears to the left of B or some node dominating B.
  
Branches cannot cross.

## C-command

the phenomenon of **Binding**.

1. C-command: A c-commands B $\Leftrightarrow$ All nodes dominating A dominate B AND A and B don't dominate each other.
   1. This relation is transitive
2. Symmetric c-command: A and B c-command each other.
   1. Sisters.
3. Asymmetric c-command: A c-commands B but B doesn't c-command A.
4. Government: A governs B if A c-commands B, and there is no G such that G is c-commanded by A and G asymmetrically c-commands B.
   1. Phrase-government: if A is phrase then G must also be phrase
   2. Head-government: if A is head then G must also be head. (i.e. word)

I.e. the following configs are allowed in govermment:
- A and B are siblings.
- A is siblings with X which dominates only B or only terminal nodes including B.

## Grammatical relations

Grammatical relations are determined by language specific structural rules.

In English: 
- Subject: the NP/CP daughter of the TP.
- Direct object: NP/CP daughter of the VP.
- Obj of preposition: NP daughter of the PP

## Do Semantics come before or after Syntax?

This view proposes that syntax comes first and semantics
are what we interpreted of it.