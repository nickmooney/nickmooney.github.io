---
layout: post
title: RNA secondary sequence prediction with the Nussinov algorithm
comments: true
---

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

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

It is worth taking a moment to think about what a naive algorithm algorithm for this would look like: evaluate each possible set of pairings, and from the ones that follow the rules we've laid out (complementary bases, matching, no sharp turns, non-crossing), pick the largest set. There are on the order of $$ n \choose 2 $$ pairings[^2], and each set can contain any number of those pairings. This leaves us with somewhere in the neighborhood of $$ 2^{n \choose 2} = 2^{\frac{n!}{2!(n-2)!}} $$ pairings, which is... big. As the number of bases grows, this solution quickly becomes infeasible. Clearly, we are going to have to do better.

## Subproblems and optimal substructure

### Intuition on subproblem decomposition

Let's say we know the optimal solution (i.e. the largest set of pairings that follow the rules) for an RNA strand with $$ n $$ bases. Does this knowledge make it any easier to find the solution for an RNA strand with length $$ n + 1 $$? Assume that the first $$ n $$ bases are the same: we have just added the base $$ b_{n+1} $$.

The simplest case is that the $$ b_{n+1} $$ does not pair with any previous bases in the optimal solution, so the optimal solution is unchanged from the $$ n $$-base solution. Otherwise, $$ b_{n+1} $$ will pair with some previous base $$ t < j - 4 $$. Due to the noncrossing condition, the optimal solution for this new strand will consist of the pairing $$ (t, n + 1) $$ and the optimal solutions for $$ b_1b_2 \ldots b_{t-1} $$ and $$ b_{t+1} b_{t+2} \ldots b_n $$. We've broken down the problem into subproblems.


                                    -------------------------------
                                    |                             |
      *    *    *    *    *    *    *    *    *    *    *    *    *  
      1    2                  t-1   t   t+1                  n   n+1


### First approach to dynamic programming

Dynamic programming is a great fit for this problem, since we have shown that the problem has [optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure): the optimal solution for the $$ n + 1 $$-length problem can be constructed from the optimal solutions of its subproblems. Let's try to formalize this a little bit. Our first approach may be to define some function $$ OPT(j) $$, which gives us the optimal solution for the strand $$ b_1b_2 \ldots b_j $$. In this case, $$ OPT(n) $$ would be the optimal solution for the whole strand.

When we make a pairing for $$ i $$ and $$ j $$, as discussed earlier, our solution consists of that pairing combined with the optimal solutions for $$ b_1 \ldots b_{t-1} $$ and $$ b_{t+1} \ldots b_j $$. 


## Footnotes

[^0]: Technically, I have no credentials -- my graduation date is in about 5 weeks.
[^1]: Much of this is paraphrased from the fantastic textbook [*Algorithm Design*](https://www.amazon.com/Algorithm-Design-Jon-Kleinberg/dp/0321295358) by Jon Kleinberg and Éva Tardos.
[^2]: Many of these pairings will not be valid by our rules, but I don't believe this changes the asymptotic bound.