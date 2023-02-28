---
layout: default
usemathjax: true
title: Constituency, Trees and Rules
permalink: /sos/ch5
---

$\require{color}$

# Reading quiz qns

1. Constituents can contain other constituents. T/F?
    - True

2. What did Merrill Garrett and his colleagues find in their experiments? 
    - people perceive arbitrary clicks not at the place of occurence 
    - but at the edges of constituents

3. What kind of information does a phrase structure rule provide? 
   - XP --> X Y Z
   - XP: Label for constituent
   - -->: "consists of"
   - X Y Z: the elements making up constituent

4. Which of the following is another label for S (sentence)? 
    - S
    - IP
    - TP (Tense Phrase)

5. Which of the following is an instance of a CONJ?
   1. XP conj XP
   2. X conj X

# Constituency, Trees and Rules

**Constituent**: A group of words that function together in a unit.

They are embedded within each other to form **hierarchical structure**.

### Merrill Garrett's click study

Let (PC) be perceived click and (AC) be actual click.

1. [In her hope of marrying] (PC) An(AC)na was impractical
2. [Harry's hope of marrying An(AC)na] (PC) was impractical.

People perceived the click at the  edges of constituents corresponding 
to the correct boundaries.

##  Rules and Treees

Phrase Structure Rules: 

$$
XP \rightarrow X \ Y \ Z 
$$

- XP: Label for constituent
- $\rightarrow$: "consists of"
- X Y Z: the elements making up constituent

Kleene Plus (+):

- $(X)$ 0 or 1 occurence of $X$.
- $X+$ 1 or more occurence of $X$
- $(X+)$ 0 or more occurence of $X$

### Common phrase structures

Noun Phrases: $NP \rightarrow (D)\ (AdjP+)\ N\ (PP+)$

Adj Phrases: $AdjP \rightarrow (AdvP)\ Adj$

Adv Phrases: $AdvP \rightarrow (AdvP)\ Adv$ (RECURSIVE)

Prep Phrases: $PP \rightarrow P\ (NP)$ (RECURSIVE)

Verb Phrases: $VP \rightarrow (AdvP+)\ V (NP)\ (NP)\ (AdvP+) (PP) (AdvP+)$ (TRANSITIVE, DITRANSITIVE, PP)

**Principle of modification**: If XP modifies a head Y, then XP must be 
a sister of Y (daughter of YP)

![Tree](/notes-blog/assets/img/sos/ch5-tree-principle-modification.png)

## Tense phrase (TP) and the structure of the sentence

$S = IP = TP$ they all are labels for sentences.

### Common TP structures

Simple sentence (Subject-Predicate): $TP \rightarrow NP\ VP$

Simple sentence adding aux/modal verbs: $TP \rightarrow NP\ (T)\ VP$

Embedded sentences can occur, usually introduced by a complementizer $C$.
However the rule below implies that embedded clauses without $C$ can also
be complements.

Complement clause: $CP \rightarrow (C) TP$.

How to account for the sentence structure.

- You can have the CP in the subject position: $TP \rightarrow \{NP / CP\} (T) VP$
- **Adjusted VP**: $VP \rightarrow  (AdvP+) V (NP) \textcolor{red}{(\{NP/CP\})} (AdvP+) (PP+) (AdvP+)$
- **Adjusted NP**: $NP \rightarrow (D)\ (AdjP+)\ N\ (PP+) \textcolor{red}{(CP)}$

### Conjunction

X phrase conjunction: $XP \rightarrow XP conj XP$
X conjunction: $X \rightarrow X conj X$

## Principle of Modification

If a phrase YP **modifies** a head (X), then phrase YP **must be a sister of X**.

> Only sisters can modify each other

## Tree drawing/Parsing

### Bottom up parsing

1. Identify parts of speech (entity type)
2. What modifies what?
   1. Must be at the same level (siblings contained in the same constituent)
   2. Apply the production rules to identify siblings
3. Recurse until the root clause
4. Test your trees against the production rules.

### Top down parsing

Top-down parsing is a strategy of analyzing unknown data relationships by hypothesizing general parse tree structures and then considering whether the known fundamental structures are compatible with the hypothesis.

1. Identify parts of speech (entity type)
2. TP node at root of the tree $\rightarrow NP VP$
   1. Flesh out the phrase
   2. Look ahead to see matching parts
3. Recurse until leaf nodes.

## Ambiguity