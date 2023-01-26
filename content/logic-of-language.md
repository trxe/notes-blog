---
layout: default
title: Logic of Language
usemathjax: true
permalink: /lol
geometry:
- top=2cm
- left=2cm
- right=2cm
- bottom=2cm
---

# Lecture 1: What is Language

- cultural artifact
- code of communication
- thought articulation
- biological trait of humans

**Mapping** between *form* and *function*.

*Function*: communication, thoughts, meaning

*Form*: sounds and signs (gestures), less emphasis on writing which is a **recording**

The form to function continuum: **sound/sign**, **words**, **sentences**, **meanings**.

## How to **know** a language

What do we know about language to make us claim that we know a language?

- Knowledge of sound patterns (e.g. impossible consonant clusters, morphophonology.)
- Knowledge of word patterns (wugs /woegz/ nazzes /nazzez/)
- Knowledge of sentence patterns (ungrammatical sentences)
- Meaning patterns (some sentences commit to certain ideas as predicates)

## Language as a system

1. Generative
2. Rule-governed (Tacit and compositional)
3. Arbitrary (but standardized in order to make sense between communicating parties)

**Generative** system: Finite vocaulary $\Rightarrow$ new sentences

Outline of Proof: we can always take a part of a sentence and always add on something to it and still have it be valid.
  - A's b
  - A's b's c
  - A's b's c's d ...

We can understand such expressions of arbitrary complexity because interpretation is *compositional*.

**Rule-governed** system: present even in colloquial/stigmatized dialects of English 
(e.g. Singlish, African American English aka AAE)

- unconscious/tacit rules
- compositional as well

**Arbitrary** system: mapping of form to function is sometimes arbitrary (for individual morphemes to their meaning).

## Problem Set 1

*Question 1*:

|                       | Generative | Consciously taught | Arbitrary |
|-----------------------|------------|--------------------|-----------|
| human languages       | +          | mostly -           | +         |
| programming languages | +          | +                  | +         |
| dog commands          | -          | +                  | +         |
| morse code            | +          | +                  | +         |
| chinese characters    | -          | +                  | +         |
| bird song             | +          | -                  | +         |
| art                   | +          | -                  | +         |
| emoji                 | -          | -                  | +         |

*Question 2*:

It supports the idea that sign/sound and meaning mapping is arbitrary.
If it were not, then there would be more consistency between the onomatopoeia cross-culturally,
but in many cases they have no similarities at all, especially for the sounds of dog, pig.

But it is not **completely arbitrary** as patterns can be still seen across.

- cat and cow always start with nasals.
- sheep mostly start with [m], but [b] is sonorant also
- [h] and [k] and post-palatal
- 

Likely due to brain picking up different sounds.

# Lecture 2: Properties and Organization of Words

**Lexicon**: Mental table of mapping between forms and their meanings

**Lexical categories**: parts of speech
- nouns [N]
  - pronouns: no fixed reference
  - proper names
- verbs [V]
- adjectives [ADJ]
- adverbs [ADV]
- prepositions [P]

- Determiners [Det] e.g. a, the, this
- Conjunction [Conj] e.g. and, but, or
- Auxiliary Verb [Aux] e.g. modal verbs

**Content words**: 
- relatively easy to define, 
- open-class
- usually [N, V, ADJ, ADV]

**Function words**:
- grammatical functions
- closed-class
- usually [Det, Conj, P, Aux, pronouns]
- **Relationships** between the content words

Psychologically we treat content and function words differently (as can be seen from aphasics)

1. Content words more likely to switch
2. Content words less likely to omit (children learn them first)

## Morphemes

**Morphemes** are the smallest meaningful units in the language.

*Hypothesis*: Morphemes are stored in the lexicon
- implication: single mapping of morpheme form to function
- counterexample: -er has multiple meanings (e.g. teacher vs letter)

### Bound morphemes

Bound morphemes (cannot exist on their own) vs free morphemes (can)

Bound morphemes often have **rules** (see distribution) on what they can bind to.

The same morpheme can take on multiple meanings (e.g. the "un-" prefix)

This gives rise to **morphological structure trees**:

## Problem Set 2

![Question 1](/notes-blog/assets/img/lol/ps2-1.png)

1. dis-obey
2. happ-ily
3. logic-ally
4. em-power-ment
5. pre-mature
6. mis-understand-ing
7. re-form-ation
8. un-surpris-ing-ly

see image for structure trees

Zulu nouns and verbs:

a) singular "um", plural "aba"
b) "a"
c) "i" is a suffix for noun class
d) "baz", "fund"

# Lecture 3: Properties and Organization of Words

**Alphabet** $\Sigma$: finite set of symbols

**String**: finite sequence of symbols from $\Sigma$

**Language**: set of strings

*Length*: Number of symbols in a string. (if $a$ = "10110", $\|a\| = 5$)

*Empty string*: $\Lambda$. ($\|\Lambda| = 0$)

*Infinite Language* $\Sigma^*$: is the language of all possible strings over $\Sigma$.

Any language $L$ of $\Sigma$ is a subset of $\Sigma^*$.

*Empty Language* $\emptyset$: language with no strings.

**Concatenation**: 
1. Concatenation of strings: 
   1. $x$ `CONCAT` $y$ = $xy$
   2. $x\Lambda = x = \Lambda x$ ($\Lambda$ is identity element)
   3. $xx^{k} = x^{k+1}$
   4. $x^0 = \Lambda$
2. Concatenation of languages:
   1. $L_1 L_2 = \{xy \mid x \in L_1 \vee y \in L_2\}$.
   2. $L_1 L_2 \neq L_2 L_1$ as $xy \neq yx$ (non-commutative).
   3. $LL_k = L^{k+1}$
   4. **Closure**: $L^* = \Cup_{i=0}^\infty L_i$.

### Computational questions

1. **Generation**: How do we generate the sentences of a language?
   1. **Grammar**: rules of production for a language
2. **Recognition**: How do we recognize if a string $x \in L$?
   1. We don't consider this programmatically to prevent the influence of a specific PL.
   2. Describe recognition in a **computing machine/automatron** that isn't a language
3. **Specification**: What's the best way to specify a language $L$?
   1. Providing an automaton that recognizes
   2. Providing a grammar that generates
   3. Intensional formal definition
   4. None of the above are "best"!
   5. Must show the correspondence between them.
4. Language Classes
   1. Complexity of languages
   2. **Regular languages** $\subset$ **Context-free Languages**
      1. Regular language: A language that can be specified by an FSM/FA
  
**Automata**: Recognizes tokens. Specifies a **symbol recognition machine**

**Regular expressions**: algebraic notation to **generate** languages

Context Free Grammars (CFG): Formation of statements from tokens.

Turing Machines: Most powerful form of automata.

### Automata

1. State
2. Finite Automaton (FA): Automaton with a finite number of states
3. Transition
4. State transition diagrams

![FA Terminology and example](/notes-blog/assets/img/lol/fa-ex1.png)

- Start state
- Accepting state
- Sink
  - Accepting sink
  - Non-accepting sink (can be removed together with all it's incoming links)

Note that $D$ here is a **sink**, a state which once you enter you can't leave.

### Distinguishability

A string $x$ and $y$ are distinguishable from each other 
if you can have another string $z$ such that either
- $xz \in L \wedge yz \notin L$
- OR $yz \in L \wedge xz \notin L$

note that it is allowed for $x, z \notin L$.

### Problem Set 3

1. FSM

a) Accept: $\{ab, aba, abb\}$. Reject: $\{a, b, bb\}$

b) Any sequence of a and b preceded with a prefix $ab$.

c) ![No trap states](/notes-blog/assets/img/lol/ps3-1c.png)

2. ![aaa](/notes-blog/assets/img/lol/ps3-2.png) 

3. $\Sigma = \{kopi, o, c, peng, kosong\}$

![FSM](/notes-blog/assets/img/lol/ps3-3ac.png) 

b) Add $teh$ link from $A$ to $B$. For all other states, add a $teh$ link to $F$.

c) the diagram above allows for "kopi $\{o,c\}$ kosong peng".