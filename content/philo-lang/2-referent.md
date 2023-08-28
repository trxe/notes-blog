---
layout: default
usemathjax: true
title: Frege; Sense and reference
permalink: /philang/ch2
---

# Naive semantics and and language of logic

## SUMMARY

1. The meaning of every expr is its referent.
2. The meaning of a singular term is its referent.
3. The meaning of a 1-place pred is its property; the meaning of an n-place pred is its relation.
4. Principle of compositionality:
5. Principle of publicity:
6. Truth

Naive semantics is a primitive **model of language**.

Naive theory:

- **Singular term**: the denotation of the referent
  - Refers to Mars the **referent**
  - The **meaning** of a singular term is its **referent**.
- **Predicate**: the conveyed state of the referent e.g. 'is happy', 'walks funnily"...
  - $\alpha$ is red
  - $\beta$ orbits the sun
  - ^ The above procedure is predicate extraction
  - **One-place** predicate: e.g. "$\alpha$ eats"
  - **Two-place** predicate: e.g. "$\alpha$ eats $\beta$"
  - **n-place** predicate: e.g. "$x_1$ eats $x_2$ with $x_3$ by $x_4$ ... $x_n$"
  - **Pure** predicates: predicates without any singular terms
    - and no conjunctions, no quantifiers
    - and (for now) no adverbs as well
  - What is the meaning of a predicate?
- *Singular term + Predicate = Statement*
  - **Atomic** sentences: sentence comprising ONLY of $n$-place predicate and $n$ singular terms.

### Meaning of Singular terms and predicates

Singular term's meaning: an object. 
i.e. What we say things about

Predicate's meaning: Ascribed attributes of object of singular term. (aka general term). 
i.e. What we say about things

1. "Mars is a planet" and "Mars is red". However "a planet $\neq$ red".
   1. Hence "a planet" and "red" are attributes and not the whole of the singular term Mars.
   2. This is one rationale for differentiating singular terms from predicates.
2. Candidates for meaning of predicates:
   1. An **idea** in the minds of users of the term (Locke)
     - Some things don't have mental images (e.g. what is the mental image of theory??)
     - Most things in the world won't have universal concrete mental images ("dog" can refer to a god of any breed...)
   2. **Singular terms**: *subjective/private* ideas;
   **Predicates**: *public* ideas;
     - this is how it's possible to communicate
   3. The meaning of a predicate is its **extension** (i.e. the set of things fulfilling the general term)
      1. Reject: the set can constantly change (e.g. the set of living people), but the meaning should not
      2. Reject: many predicates satisfied by the same set of objects definitely have different meanings. (e.g. the persons with the id XXX and the persons with the unique name YYY).
   4. The meaning of a one-place predicate is the *property* for which it stands. The meaning of a n-place predicate is the *relation* for which it stands.
  
> **The fundamental principle of naive semantics**:
>
> The **meaning** of every expression is its **referent**.

## Propositions

So the meaning of a sentence is its proposition.

The meanings of *atomic* sentences are *atomic* propositions.

> *Naive semantics for atomic sentences*:
>
> Propositions of atomic sentences is **composed of** the meanings of its parts.

![Naive Semantics](/notes-blog/assets/img/philang/semantic_composition.png)

Problem: sentences have **structure** which also conveys meaning (sentences are not an unordered collection of singular terms and predicates).

## Truth

> *Naive definition of truth for atomic propositions*:
>
> An atomic proposition is **true** iff it is: 
>   - one-place proposition comprising object $o$, property $P$ and $o$ possesses $P$.
>   - OR: two-place proposition with objects $o_1, o_2$, relation $R$ and $o_1$ bears $R$ to $o_2$.

Sentences following from these are T-sentences.
e.g. "The proposition expressed by ‘Mars is red’ is true if and only if Mars is red."

[!] A sentence is true if the proposition it means is true.

If truth is correspondence: it must correspond to **fact** or **the way the world is**.

- Actual world
- Possible world
- Impossible worlds
- Tractatus first page:
  - The world = all that *is the case* = all the facts
  - state of affairs = the way things are

[!] However now there is a problem with the naive definition of truth.
- We have the fact that Mars orbits the sun
- We have the proposition "Mars orbits the sun"
- these have to be different in some way other wise every atomic proposition is true, because the definition is a tautology!!!

## Sentence connectives

1. Conjunction: AND
2. Disjunction: OR
3. Negation: NOT
4. Conditionality: IF
5. Biconditionality: IFF

## Quantifiers

1. Universal quantifiers: for all, every, ...
2. Existential quantifiers: some, there exists, ...

# Fregean semantics

## Problems with Naive semantics

Arose because of 2 main problems with naive:

1. Cognitive value
   1. "The Morning Star" doesn't mean "The Evening Star"
   2. As in one person can believe "The Morning Star is a planet" and not believe "The Evening Star is a planet".
   3. But they refer to the same thing.
2. Empty singular term
   1. "The Loch Ness Monster lives in a lake".
   2. According to the naive theory we cannot have such a proposition because this state of affairs does not exist!

### (Naive) Referential theory of meaning

proper names/indefinite terms refer to a certain entity

hence they mean this certain entity or referent

predicates refer to their extensions (the set of all things satisfying their property)

*Cognitive value* problem: 
- if propX always coexists with propY, then X==Y
- if nameX refers to the same nameY, then nameX means the same thing as nameY

#### Proof/Argument validity

$a=a$ vs $a=b$.

1. $a=a$ holds a priori. [you know this is true without exp.]
   1. By definition
   2. Let A be the stmt "Superman == Superman"
2. For $a=b$ to hold, you need to know background knowledge (exp. required), holds a posteriori.
   1. These are **extensions of our knowledge**.
   2. Let B be the stmt "Superman == Clark Kent"
3. By the referential theory of meaning, and the principle of compositionality and the principle of substitutability, A means the same thing as B.
4. Hence B should not introduce more info than A does!!

## Sense and reference

Sense:
1. A Mode of presentation of the referent
2. A rule for determining a referent
3. The cognitive significance

Co-referential singular terms: terms with different senses but refer to the same referent.

![Sense vs Reference illustrated](/notes-blog/assets/img/philang/senseref.png)

### Sense/ref distinction

- Predicates (meaning = reference = extension)
  - The reference of a predicate = the set/class of things satisfying the predicate.
  - The sense of a predicate = the rule/criterion for x to be in the extension of the predicate.

Compositionality of reference: Truth-value = sum(referents of parts)

Compositionality of sense: Proposition or Thought = sum(senses of parts)

- Sentence (meaning = proposition = thought [Frege])
  - It cannot be determined by meanings/referents of its parts
  - Otherwise "The Morning Star is the Evening Star" is trivial
  - The sense of a sentence = truth-condition i.e. a reference determining condition.
  - The sense of a sentence = truth-value [T/F]

![Syntax, Sense, Reference](/notes-blog/assets/img/philang/synsenref.png)

## Problems with Fregean semantics

1. Negative existentials
   1. Reference failures: When an expression without a reference is used in a sentence, by Fregean semantics it is said to **not have** a truth value.
   2. What about a sentence like "The Loch Ness Monster does not exist"?  
      1. It is a true statement.
      2. However the Loch Ness Monster really does not exist.
      3. So by Fregean semantics it is neither true nor false.
2. "Proof of God" 
   1. 3 predicates of God: God is **omniscient**, **omnipotent**, and **omnipresent/exists**.
   2. So it is actually contradictory to deny that God exists by Fregean semantics
      1. because God exists by definition
   3. However if existence must be represented as a quantifier than this claim becomes logically deniable.

## The Principle of Substitutability

- If sentence S contains expression e, 
- and e* has the same referent as e, 
- and S* results from S by replacing e by e*, 
- then S* has the same truth-value as S.

**Extensional** language:
Language where principle of substitutability holds all the time.

## Propositional attitudes

The truth value of a matrix clause does not depend on the truth-value/referent of a subordinate clause.

1. Lois Lane believes Superman is Superman.
2. Lois Lane believes Clark Kent is Superman.

I can also believe 2 different things without those things being identical.

1. can be true without 2. being true. However the subclauses have the same truth value in both.