---
layout: default
usemathjax: true
title: Binding Theory and X-bar Theory
permalink: /syntax/ch3
---

# Our current theory

We have PSRs in our head. 

- They allow us to generate "grammatical" sentences.
- ungrammatical phrases are never generated
- psrs show constituency
- structural relations exist

So what generates these rules?

And can the justification for our theory make new and correct conclusions?

## Dependencies as a syntactic phenomena

examples:

1. It was a bagel that Zheng ate (gap)
   1.  Filler gap dependency
2. The **key** to the doors **is** missing 
   1.  Agreement dependency
3. **Jane** had done the job **herself**.
   1.  Reference dependency

# Binding Theory

Binding Theory: Theory of the **syntactic restrictions** on which phrases of 
the **same class** can appear in a sentence.

## Example: R-expressions, Anaphors, Pronominals

- **R-expressions**: NP that derives its meaning from a **subsistent entity in the world**.
- **Anaphor**: NP that gets its meaning from **referring to another NP** in the sentence.
  - **Antecedent**: NP that gives meaning to another NP.
  - Anaphors must always have an antecedent in the same sentence
- **Pronoun**: NP that may or may not have an antecedent
  - Refers to something in the context (indexical), 
  - But it may not have an antecedent in the sentence itself 
  - **Pronouns and anaphors can also be antecedents**
- **Index**: marker for labelling world references

> [Heidi (antecedent)] bopped [herself (anaphor)] on [the head] with [a zucchini].
>
> [Heidi]$_i$ bopped [herself]$_i$ on [the head]$_j$ with [a zucchini]$_k$.

`Heidi` and `herself` are coindexed $i$.

### What is Binding

> 1. [Heidi]$_j$ slapped [herself]$_j$.
> 2. [Heidi_j's mother]$_i$ slapped [herself]$_i$.
> 3. *[Heidi_j's mother]$_i$ slapped [herself]$_j$.

![Binding tree example](/notes-blog/assets/img/syntax/syntax-binding.png)

A **binds** B iff 
1. A c-commands B 
2. and A and B are coindexed.

Hence binding is coindexation when **one NP c-commands the other**.

### Anaphors in binding

> 1. [Herself]$_j$ slapped [Heidi]$_j$.
> 2. *[Heidi_j's mother]$_i$ slapped [herself]$_j$.

**Hypothesis 1.0**: An anaphor must be bound by an antecedent R-expression.

Important: The *binder* (antecedent) must c-command the *bindee* (anaphor/pronominal).

### Locality Conditions

> *[Heidi]$_i$ said [that [herself]$_i$ discoed with Art].

![Local to binding domain](/notes-blog/assets/img/syntax/syntax-locality.png)

The anaphor seems to need to **find its antecedent in the same clause**. 
This is called a **locality constraint**.

**Locality constraint**: A certain rule must be satisfied within a certain locality.

**Binding domain (ver. 1)**: The *clause* containing the NP

**Hypothesis 2.0 (Principle A)**: An anaphor must be bound by an antecedent R-expression in its binding domain.

Alt hypothesis: An anaphor must be c-commanded by its antecedent without any other R-expression in between

- Structural Relation
- Locality Restriction

### Distribution: Free vs Bound

> 1. [Heidi]$_i$ said that [she]$_i$ discoed with Art.
> 2. [Heidi]$_i$ said that [she]$_k$ discoed with Art.
> 3. [Heidi]$_i$ played with [herself]$_i$.
> 4. [Heidi]$_i$ played with [her]$_j$.
> 5. *[Heidi]$_i$ played with [her]$_i$.

- 1 and 2 show that pronouns can be unbound, or bound to antecedent.
- 2 and 5 show that pronouns can only be bound to antecedent **outside** of the binding domain of the pronoun.
- Unbound NPs are **free**.
- 3 and 4 demonstrate grammatical uses of anaphors and pronouns within the binding domain.

**Principle B**: A pronouns must not be bound to an antecedent (i.e. free) in its binding domain.

The distribution of pronouns and anaphors are mutually exclusive.

> 1. [Oliver]$_i$ beheaded [George]$_i$

What about identity statements ("A is B")?

**Principle C**: R-expression must be free.

### Is binding principle universal

- No across languages (try w. others)
- No within English?
  - While he$_i$ was overseas, Zheng$_i$ fell down.
  - 

# X-bar Theory

1. [X-bar theory](/notes-blog/sos/ch8)
1. [X-bar theory pt 2](/notes-blog/sos/ch9)

## Summary

Motivation: Replacement test in the following sentence:

> I bought [the big **[book] [of poems] [with [the blue cover]]**] and not the small [**one**].
> 
> I bought [the big **[book] [of poems]** [with [the blue cover]]] and not the small [**one**] with the red cover.

Why is a collection of sister constituents of the NP replaceable with **one**?
Why is this applicable on several levels?

Ans proposed: embedded structure.

### Theory statements

- all phrases must have heads (endocentricity)
- anything in an X-bar rule that isn't a head must be an optional phrase
- Head introduction Rule (The termination rule)
  - X' $\rightarrow$ X (WP)
- Recursive X' Rule
  - X' $\rightarrow$ X' (YP)
- Specifier rule
  - XP $\rightarrow$ X' (ZP)
  - How to motivate the existence of ZP?
  - We can imagine languages with many determiners! Why all langs we see only have one? (falsifiable by counterex!!)
    - i.e. why do we remove the XP from X'?
- How to draw the tree?
  - **proform replacement**: 
    - "one" (within NP)
    - "do so" (within VP)
    - "so"/"that way" (within AdjP, AdvP, PP)
  - Conjunction on X'

**Adjunct YP**: Sister to some X' and daughter of another X'

**Complement YP**: Sister to the **head** X and daughter of another X'

How to determine complements (data required):

1. Is the YP's sibling replaceable?
   1. the _book_ of poems
   2. vs *the *_one_ of poems
   3. book must be an X instead of X'
2. Can the YP's sibling X be replaced by **another X'**?
   1. One way to test: can we insert in another one of the YPs?
   2. the _book_ of poems
   3. vs *the [book of fiction] of poems
  
### Housekeeping

1. Projections of the head X: 
   1. ancestor X' (intermediate projections)
   2. XP (maximal projection).
2. Principle of Modification:
   1. if a **YP modifies X**, then it must **be dominated by some projection of X**.
3. Direct Object:
   1. NP/CP that is complement of V
4. Word order: 
   1. the complements/adjuncts may appear before or after the head depending on the language
   2. Language parameterization