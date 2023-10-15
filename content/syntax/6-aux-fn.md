---
layout: default
usemathjax: true
title: Auxiliaries and Functional Categories
permalink: /syntax/ch6
---

# Auxiliary and Functional Categories

Theta grids can be defined for categories other than verbal arguments as well.

## Complementizers (*if, that, for...*)

Different verbs can take and not take different **complement clauses**, identified
by different **complementizers**.

> - I think **that** Art likes beer.
> - I ordered **that** Art drink beer.
> - I think **(null)** Art likes beer.
> - *I ordered **(null)** Art likes beer.
> - *I ordered **(null)** Art drink(s) beer.
> - I asked **if** Art drinks beer.
> - *I think **if** Art drink(s) beer.
> - *I ordered **if** Art drink(s) beer.

Classifying complementizers:

- [$\pm$Q]:
  - [$+$Q]: embedded question
  - [$-$Q]: embedded proposition
- [$\pm$FINITE]:
  - [$+$FINITE]: main verb used is finite.
  - [$-$FINITE]: presence of the $T_{[+\text{INFINITE}]}$ such as "to"

e.g.

1. that (-Q, +FINITE)
1. (null) (-Q, +FINITE)
1. for (-Q, -FINITE)
1. (null) (-Q, -FINITE)
1. if/whether (+Q, +FINITE)

## Determiners (*a, the, ...*)

> XXX could be great.

- [-PLURAL]
  - The muffin
  - A muffin
  - *Muffin
- [+PLURAL]
  - Muffins
  - The muffins
  - *A muffins
- [-PLURAL, +PROPER]
  - John
  - *The John
  - *A John
- tricky to account for: [+PLURAL, +PROPER]
  - The Smiths
  - *A Smiths
  - *Smiths

Hence for DPs:
1. a *NP*[-PLURAL, +PROPER, -PRONOUN]
1. the *NP*[-PROPER, -PRONOUN]
1. (null)[+PROPER] *NP*[+PROPER, -PRONOUN]
1. (null)[+PRONOUN] *NP*[-PROPER, -PRONOUN]
1. (null)[+PLURAL] *NP*[+PLURAL, -PROPER, -PRONOUN]

What about quantifiers?

1. many *NP*[+COUNT, +PLURAL, -PROPER, -PRONOUN]
1. much *NP*[-COUNT, +PLURAL, -PROPER, -PRONOUN]

# Tense Aspect Voice Mood

## Tense

- past (preterite)
- present
- future (futurate)
  - "will"

## Aspect

- perfect
  - relationship between (past) event time and assertion time
  - participle
  - "have/has/had" + PARTICIPLE
    - Present: still ongoing
    - Past: completed
- progressive
  - gerund
  - be(NOT-COPULA) + PARTICIPLE

## Voice

- active
- passive
  - be(NOT-COPULA) _-en (PARTICIPLE)

## Mood

e.g. can, could, may, might, would, shall, should, must, ought

Distributional properties:
1. Precede all other aux
2. Precede negation
3. No agreement inflexion.

## Main Verbs vs Auxiliaries

1. Has [Main: possession; Aux: Perfect]
   1. Calvin had [Main, POSS] books. / Calvin had [Aux] read [Main] books.
   1. Bill had [Main, POSS] an accident. / Bill had [Aux] caused [Main] an accident.
2. Be [Main: COPULA, Aux: Progressive Aspect/Passive Voice]
   1. Calvin was [Main, COPULA] mad. / Calvin was [Aux, PROGRESSIVE] being [Main, COPULA, PROGRESSIVE PARTICIPLE] mad.
3. Do
   1. Calvin did skiing / Calvin did (not) ski

| Name               | Meaning                       | Subcategory |
|--------------------|-------------------------------|-------------|
| be$_\text{cop}$    | Copula                        | Main Verb   |
| be$_\text{prog}$   | Progressive                   | Aux         |
| be$_\text{pass}$   | Passive                       | Aux         |
| have$_\text{poss}$ | Possessive                    | Main Verb   |
| have$_\text{perf}$ | Perfect                       | Aux         |
| do$_\text{main}$   | Perform                       | Main Verb   |
| do$_\text{aux}$    | Support tense before not/emph | Aux         |