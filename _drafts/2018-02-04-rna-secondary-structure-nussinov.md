---
layout: post
title: RNA secondary sequence prediction with the Nussinov algorithm
comments: true
---

Last year I had the good fortune of taking an undergraduate algorithms course with [Larry Ruzzo](https://homes.cs.washington.edu/~ruzzo/). One of my favorite parts of the course was applying dynamic program to approximate a real-world problem in the field of computational biology: RNA secondary structure prediction. I ended up implementing the algorithm in a couple languages (I was learning Nim at the time) and swore I would write it up, so this post will give an introduction to and an implementation of the Nussinov algorithm.

## The problem context

*Disclaimer: I am a computer scientist[^0], not a biologist. I am sure that the basic Nussinov algorithm is a drastic simplification of real-world computational biology, so if you would like to learn more about the biology side of this, ask your favorite (computational) biologist.*

Most of us are familiar with the structure of DNA: a [double helix](https://www.youtube.com/watch?v=tWzJhkrZm5Y) that consists of the bases A, C, T, and G. The double helix is formed because the bases in each strand are paired with their corresponding bases (A/T and C/G) in the other strand -- in other words, each strand in DNA is sort of an "inverse" of the other. RNA, on the other hand, is single stranded, so does not have a "natural" structure like DNA. However, the complementary bases (A/U and C/G) still have affinities for each other, so strands of RNA end up folding in on themselves to form a unique secondary structure. Knowledge of this secondary structure gives us insight into the behavior of the molecule, so it is very handy to be able to predict it.

Predicting the secondary structure is also a little tricky: any complementary bases can pair up with each other (with some restrictions we will detail below), but physics tells us that the eventual scondary structure will approximate the one with the optimum total free energy. Total free energy can be approximated by the *number* of pairings that form the secondary structure, so the "optimal" structure is approximately the structure with the largest number of pairings. Remember that a pairing is defined by two complementary bases "sticking" to each other.

How do we find the secondary structure with the largest number of pairings? This is where the Nussinov algorithm comes in. First, let's formalize the idea of a pairing.

### What is a pairing?

Let's define a pairing in somewhat formal terms[^1]. We can view a single-stranded RNA molecule as a sequence of $$ n $$ symbols drawn from the alphabet $$ \{A, C, G, U\} $$. Let $$ B = b_1b_2 \ldots b_n $$ represent our RNA strand, where each $$ b_i \in \{A, C, G, U\} $$. Pairings must form between complementary bases, and each base may pair with at most one other base (i.e. the base pairs form a matching). Using the notation from *Algorithm Design*, a base pairing is some set $$ S = \{(i, j)\} $$ in which $$ (i, j) \in \{1, 2, \ldots, n\} $$. Base pairings must also adhere to the following conditions:

  1. No sharp turns. There must be at least 4 bases between any base pair. Formally, if $$ (i, j) \in S $$, then $$ i < j - 4 $$.
  2. No crossing. If $$ (i, j) $$ and $$ (k, l) $$ are in $$ S $$, then we must not have the case that $$ i < k < j < l $$. Intuitively, this means a base within any given pairing cannot pair with a base outside that pairing.

### The naive solution

It is worth taking a moment to think about what a naive algorithm algorithm for this would look like: evaluate each possible set of pairings, and from the ones that follow the rules we've laid out (complementary bases, matching, no sharp turns, non-crossing), pick the largest set. There are on the order of $$ n \choose 2 $$ pairings[^2], and each set can contain any number of those pairings. This leaves us with somewhere in the neighborhood of $$ 2^{n \choose 2} = 2^{\frac{n!}{2!(n-2)!}} $$ pairings, which is... big. As the number of bases grows, this solution quickly becomes infeasible.

## Footnotes

[^0]: Technically, I have no credentials -- my graduation date is in about 5 weeks.
[^1]: Much of this is paraphrased from the fantastic textbook [*Algorithm Design*](https://www.amazon.com/Algorithm-Design-Jon-Kleinberg/dp/0321295358) by Jon Kleinberg and Éva Tardos.
[^2]: Many of these pairings will not be valid by our rules, but I don't believe this changes the asymptotic bound.

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
