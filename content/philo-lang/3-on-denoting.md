---
layout: default
usemathjax: true
title: Russell; On Denoting
permalink: /philang/ch2
---

# Russellian Semantics

## Summary

1. Equivalence of the **definite description** to the combination of 3 clauses
   1. Existence
   2. Uniqueness
   3. Categorical
2. For a proposition containing a singular expression of the form "The F is G" to hold, "*There exists a unique F* and this F is G" must be true
3. Importantly solves:
   1. Substitution of identicals
   2. Negation (law of excluded middle)
   3. Negative existentials
4. Incompleteness problem?
5. Embedding under desiderata? (sentence syntactic ambiguity)

Complex singular terms: singular terms that contain a predicate in themselves

definite description: $\text{the } F$

> [1] The present King of France is bad.

Russell says the above must have the truth value of **false** because the the **truth-condition**
that "The present King of France" exists is **false**!

If "the present King of France" is treated as a predicate itself, then compositionality breaks down.

Truth conditions of [1] (entailment?)

1. Existence: there is (currently) a king of france
   1. $\exists x \text{ s.t. } F(x) = True$
2. Uniqueness: there is not more than 1 king of france
   1. $\nexists x,y \text{ s.t. } x \neq y \wedge F(x) \wedge F(y)$
3. Categorical: any king of france is bad
   1. $ x \wedge F(x) \rightarrow G(x)$

The logical form of "The F is G" $\neq$ the linguistic form of "The F is G".

Contextual definition: aka definite description

Expicit definition: Substitutable component of a given symbol with another of the same category (synonymous NP)

Logically proper name: If 'a' is a logically proper name (a standalone symbol), then 'Fa' is meaningless if it lacks a referent.

Ordinary proper name: Abbreviation of a definite description (e.g. "monster which lives in the Loch Ness river", "stage name of Bob Dylan")

> [2] Not: The present King of France is bad.
> 
> [3] #The present King of France is not bad.

[2] is true but [3] is not. [2] is the external negation and [3] is the internal.

## Applying the theory of descriptions

> [4] The man who killed the Kennedys was Cuban.
> 
> [5] The Loch Ness Monster doesn't exist.
> 
> [6] He wondered if Scott = the author of Waverley.
> 
> [6a] He wondered if Scott = Scott.

No man killed both Robert and John K.

1. In Frege's theory: [4] is of the form $Fa$. In Russell's theory: [4] is of the form $\text{The } F \text{ is } G$.
   1. Frege: "The man who killed the Kennedys" is singular term
   1. Russell: "The man who killed the Kennedys" is predicate
2. In Frege's theory: [5] is neither-true-nor-false. In Russell's theory, it is true.
   1. Russell: NOT: There is a (monster) such that this (monster) lives in the Loch Ness lake, and for all monsters living in Loch Ness, it must be this (monster).
   2. When replaced with its description, which are a set of existing attributes, the sentence becomes one that can have a truth value.
3. Description theory replaces "the author of Waverley" in [6] with "x such that x wrote Waverley" which is a unit set.
   1. This explains the difference in cognitive value between [6] and [6a], 
   2. [6a] having the logically proper name on both sides of the relation
   2. [6] having a definite description on the RHS

### Logical/Ordinary proper name

An ordinary proper name cannot possibly be a synonym for one definite description.

1. I only know Bob Dylan as a famous singer (only valid definite description I know of Bob Dylan)
2. He only knows Bob Dylan as his neighbour (only valid definite description he knows of Bob Dylan)
3. We both know Bob Dylan plays the guitar.
4. Is the Bob Dylan we know the same?

Russell's soln: On each occasion one uses an ordinary proper name, the name = the person's thought at that moment.

## Knowledge by acquaintance vs Knowledge by description