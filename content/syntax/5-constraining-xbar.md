---
layout: default
usemathjax: true
title: Constraining X-bar theory
permalink: /syntax/ch5
---

# X-bar 

1. Specifier rule: $XP \rightarrow (YP) X'$ (or reversed)
1. Adjunct rule: $X' \rightarrow (ZP) X'$ (or reversed)
1. Complement rule: $X' \rightarrow (WP) X$ (or reversed)

Here the complement rule seems to be non-obligatory but some sentences don't make sense.

- Rosemary ran.
- *Rosemary took.
- *Rosemary gave.
- *Rosemary gave the story.
- *Rosemary gave to Mark.

Seems to be constraints imposed by the lexicon (is the word a transitive/intransitive/ditransitive verb)?

## Selectional restrictions

semantic criteria for whether certain words should/should not co-occur.

### Thematic relations, Theta roles

Entities that undergo actions or are moved, experienced, or perceived
are called themes.

- Agent
- Experiencer
- Goal
- Recipient
- Source
- Location
- Instrument
- Beneficiary

DPs can have $\geq 1$ thematic relation. However $\theta$ roles are
**bundles of thematic relations** that cluster on one argumentm
and thus **do map one-to-one** with arguments.

Example: "X **placed** Y Z"

> John$_i$   placed [the flute]$_j$ on [the table]$_k$.

![Theta grid](/notes-blog/assets/img/syntax/theta-roles-table.png)

(B -- First row) Tells the thematic relations

(C -- Second row) Indices of each thematic role

(D -- Underlined) External theta role. For **subject**

(E -- Not underlined) Internal theta role. For **objects**

Only DP in **subjects and complements** have theta roles, DP in adjuncts don't. 
(You can have as many or as few adjuncts as you like, unrestricted by $\theta$ grids.)

### Theta criterion

1. Each argument is assigned **exactly** one theta role.
2. Each theta role is assigned **exactly** one argument.

## Lexicon

Grammar essentially consists of 

1. Computational component: Rules and constraints
2. Lexicon: the irregular and memorized parts of language
   1. Meanings
   2. Syntactic category
   3. Pronunciation
   4. Exceptional info (irregularities)
   5. Theta grid (argument structure)

![Abstract Grammatical System](/notes-blog/assets/img/syntax/abstract-grammatical-system.png)

**The projection principle**: Lexical information (such as theta roles) is syntactically represented at all levels.

- Using the lexical elements, a tree can be created
  - Lexicon only include the root words that are transformed/operated on to form morphologically derived words
  - Book --> Books
    - Generated
  - Child --> *Childs
    - Do we store "child" and "children" separately in the lexicon?
  - give --> #give up (as some meaning related to giving)
    - We store "give" and "give up" separately in the lexicon as well.
- Use this part of the information and filter the tree out.

## Expletives and the Extended Projection Principle

- Expletive/plenoastic pronouns

> **It** rained today.
>  (undergenerating, "it" does not have a theta role!)
> *Rained today.
>  (overgenerating, no agent!)
> *Is unlikely that Zheng bought bagels.

Expletives seem to appear where there is no theta marked DP (or CP)
that fills the subject position. This is encoded in a revised version
of the Projection Principle: **The Extended Projection Principle (EPP)**:

> All clauses must have subjects (i.e. the specifier of TP must be filled by
> a DP or CP) and lexical information is expressed at all levels.

Why is this needed?

It seems that in English, every sentence (even imperatives, 
with invisible subjects) need subjects. [In a conversation, there's 
always an addresser and addressee.]

> **That Bill likes chocolate** is *unlikely*.
> *Is unlikely that Bill likes chocolate.

## The compute pipeline

1. X-bar rules to generate tree.
2. Theta criterion must be satisfied.
3. Expletive insertion.
4. Extended Projection Principle must be satisified.

We have added 3 components, and their invocation
must be ordered.
Reason: not the principles are not held at every state.

**Violations of this pipeline**? 

What about

> He already ate.
> He already ate the bagel.

How to (possibly) model this:
- For eat: the 2nd argument is optional
- For eat, we have eat$_1$ and eat$_2$ with 2 theta criterions.
- For eat, we have a null argument in the experiencer theme.

Agents can be objects too:
A-movement in passivization. Agent is deleted, object theme is moved to the subject position.

> [There] is a bagel on the floor.

There has no meaning. So it must be an expletive too. 
But what is the structure of this?

- A bagel and two cupcakes **are** on the floor.
- Two cupcakes are a bagel **are** on the floor.
- There **is/are** a bagel and two cupcakes on the floor.