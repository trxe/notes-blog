---
layout: default
usemathjax: true
permalink: /speech/ch9
---

# Reading quiz qns

1. Which of the following best characterizes Generative Grammar?
2. In the underlying representation of lexical items, every feature is specified for every segment (phone). T/F?
   1. False
3. Which of the following correctly represents the symbols for word boundary, morpheme boundary, and null?
   1. \# / + / Ø
4. On the SPE Generative Grammar view of language, how do different languages differ?
   1. different algorithm
5. Any phonological rule can only change one feature value. T/F?
   1. F (see catalan)


# Generative Grammar

Focus on **deriving surface pronunciations** from underlying forms.

What it means to know a language:
- know a set of UR
- know it's rules
- correctly apply (aka **derive**) the surface representations (SR)

Derivations are applied **sequentially** in the process of language assembly in the grammar factory.

1. Lexicon warehouse of morphemes
2. Syntax assembly: Morphemes are put into words/sentences
3. Phonological assembly: rules are applied individually in sequence

Underspecification: Some feature speicifications are redundant

A language is a process/algorithm that generates SRs from a set of URs concatenated in various ways.

## SPE notation (Sound Pattern of English)

word boundary, morpheme boundary, and null:  # / + / Ø

A -> B / C _ D

A -> B: Rule Formalism. (-> represents "become" or "are")

/ : in the environment of

C _ D: the preceding/following environment

Derivations: UR to SR

![SPE Derivation](/notes-blog/assets/img/speech/spe-derive.png)