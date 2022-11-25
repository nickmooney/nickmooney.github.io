---
date: "2018-02-07T00:00:00Z"
title: RNA secondary sequence prediction with the Nussinov algorithm
slug: rna-secondary-structure-nussinov
---

Last year I had the good fortune of taking an undergraduate algorithms course with [Larry Ruzzo](https://homes.cs.washington.edu/~ruzzo/). One of my favorite parts of the course was applying dynamic program to approximate a real-world problem in the field of computational biology: RNA secondary structure prediction. I ended up implementing the algorithm in a couple languages (I was learning Nim at the time) and swore I would write it up, so this post will give an introduction to the Nussinov algorithm, along with some of an implementation in Nim.

<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

* TOC
{:toc}

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

It is worth taking a moment to think about what a naive algorithm algorithm for this would look like: evaluate each possible set of pairings, and from the ones that follow the rules we've laid out (complementary bases, matching, no sharp turns, non-crossing), pick the largest set. There are on the order of $$ n \choose 2 $$ pairings[^2], and each set can contain any number of those pairings. This leaves us with somewhere in the neighborhood of $$ 2^{n \choose 2} = 2^{\frac{n^2 - n}{2}} $$ pairings, which is... big. $$ \mathcal{O}(2^{n^2}) $$ big. As the number of bases grows, this solution quickly becomes infeasible. Clearly, we are going to have to do better.

## Subproblems and optimal substructure

### Intuition on subproblem decomposition

Let's say we know the optimal solution (i.e. the largest set of pairings that follow the rules) for an RNA strand with $$ n $$ bases. Does this knowledge make it any easier to find the solution for an RNA strand with length $$ n + 1 $$? Assume that the first $$ n $$ bases are the same: we have just added the base $$ b_{n+1} $$.

The simplest case is that the $$ b_{n+1} $$ does not pair with any previous bases in the optimal solution, so the optimal solution is unchanged from the $$ n $$-base solution. Otherwise, $$ b_{n+1} $$ will pair with some previous base $$ t < j - 4 $$. Due to the noncrossing condition, the optimal solution for this new strand will consist of the pairing $$ (t, n + 1) $$ and the optimal solutions for $$ b_1b_2 \ldots b_{t-1} $$ and $$ b_{t+1} b_{t+2} \ldots b_n $$. We've broken down the problem into subproblems.


```
                                -------------------------------
                                |                             |
  *    *    *    *    *    *    *    *    *    *    *    *    *
  1    2                  t-1   t   t+1                  n   n+1
```


### First approach to dynamic programming

Dynamic programming is a great fit for this problem, since we have shown that the problem has [optimal substructure](https://en.wikipedia.org/wiki/Optimal_substructure): the optimal solution for the $$ n + 1 $$-length problem can be constructed from the optimal solutions of its subproblems. Let's try to formalize this a little bit. Our first approach may be to define some function $$ OPT(j) $$, which gives us the number of base pairings in the optimal solution for the strand $$ b_1b_2 \ldots b_j $$. In this case, $$ OPT(n) $$ would be the number of pairings in the optimal solution for the whole strand.[^3]

When we make a pairing for $$ i $$ and $$ j $$, as discussed earlier, our solution consists of that pairing combined with the optimal solutions for $$ b_1 \ldots b_{t-1} $$ and $$ b_{t+1} \ldots b_j $$. The $$ OPT $$ function we defined will only give us the optimal solution for $$ b_1 \ldots b_{t-1} $$, but we have no information about the $$ b_{t+1} \ldots b_j $$ subproblem. We will have to introduce another variable.

### Introducing another variable

Let's define a new function $$ OPT(i, j) $$ which represents the number of pairings in the optimal solution from $$ b_i $$ to $$ b_j $$. Both of our subproblems can be represented by this function, and we can define it as a recurrence:

$$ OPT(i, j) = \max \begin{cases}
  OPT(i, j - 1) \\
  \max\limits_{t} 1 + OPT(i, t - 1) + OPT(t + 1, j - 1)
\end{cases} $$

We maximize over $$ t $$, constrained to the values of $$ t $$ that follow the rules for allowable pairings we defined above (no sharp turns, complementary bases, max one pairing per base, noncrossing). In English, we can understand this as saying that the optimal solution for the strand $$ b_i \ldots b_j $$ is either:

1. The optimal solution for the strand $$ b_i \ldots b_{j-1} $$ (i.e. $$ b_j $$ doesn't get paired), or
2. The pairing $$ (t, j) $$ for the $$ t $$ that follows the rules and maximizes the number of pairings in $$ (i, t - 1) $$ and $$ (t + 1, j - 1) $$, plus those optimal solutions to $$ (i, t - 1) $$ and $$ (t + 1, j - 1) $$.

We can now find the optimal solution in $$ \mathcal{O}(n^3) $$ time: thats $$ n^2 $$ values of $$ OPT $$ to calculate, and (approximately) $$ n $$ possible values of $$ t $$ to look over for each value of $$ OPT $$. This is much better than our previous exponential-time solution.

## Implementation

As is usually the case in dynamic program, we need to build up solutions to subproblems in an order such that each later subproblem consists of previous, already-calculated subproblems. Looking at our definition of $$ OPT $$ above, we can see that each calculation of $$ OPT(i, j) $$ relies on subproblems strictly smaller than $$ j - i $$. If we calculate values for $$ OPT $$ in order of ascending strand size, we should be good.

Our base cases will be those strands that are too short to contain *any* pairings. By the "no sharp turns" rule, we can declare that $$ OPT(i, j) = 0 $$ for all $$ i \geq j - 4 $$. We will then build up solutions starting at $$ k = 5 $$ and going up to $$ k = n $$ (for a strand of length $$ n $$). In Nim, this looks something like the folllowing:

```nim
# calculate the OPT array for a given set of bases
proc nussinov(bases: seq[RNABase]): seq[seq[int]] =
  let n = bases.len
  result = newSeqWith(n, newSeq[int](n))

  # Initialize OPT[i,j] = 0 where i >= j - 4
  for i in 0..<n:
    for j in 0..<n:
      if i >= j - 4:
        result[i][j] = 0

  # Calculate progressively larger subproblems according
  # to the recurrence given in the textbook
  for k in 5..<n:
    for i in 0..<(n-k):
      let j = i + k
      var max_val = result[i][j-1]

      for t in i..<(j-4):
        if complementary(bases[t], bases[j]):
          let before_val = if t > 0: result[i][t-1]
                           else: 0
          let after_val = if j > 0: result[t+1][j-1]
                          else: 0
          let cur_val = 1 + before_val + after_val
          max_val = max(max_val, cur_val)

      result[i][j] = max_val
```

A matrix is a great way to represent what's going on as this code runs, so let's draw it out. Our matrix is "truncated" ($$ j $$ starts at 6) because we only need to consider subproblems of $$ k = 5 $$ and up (by the no sharp turns rule again). Let's see what happens as we fill it in.

```
Sequence: ACCGGUAGU               Fill in values for k = 5

    4 |0 0 0                          4 |0 0 0 0
    3 |0 0                            3 |0 0 1
    2 |0                              2 |0 0
i = 1 |                           i = 1 |1
      --------                          --------
   j = 6 7 8 9                       j = 6 7 8 9

k = 6                             k = 7

    4 |0 0 0 0                        4 |0 0 0 0
    3 |0 0 1 1                        3 |0 0 1 1
    2 |0 0 1                          2 |0 0 1 1
i = 1 |1 1                        i = 1 |1 1 1
      --------                          --------
   j = 6 7 8 9                       j = 6 7 8 9


k = 8

    4 |0 0 0 0
    3 |0 0 1 1
    2 |0 0 1 1
i = 1 |1 1 1 2
      --------
   j = 6 7 8 9
```

The value in the lower right tells us that the optimal solution has two pairings.

## Backtracking

We know how to calculate the $$ OPT $$ matrix, but weren't we looking for a solution that actually gave us the pairings? To get the pairings, we have to backtrack through our $$ OPT $$ matrix -- another common idea in dynamic programming. The pseudocode looks like this:


```
function traceback(i, j):
  if i >= j:
    return

  if OPT[i][j] == OPT[i][j-1]:
    # optimal solution does not contain (i, j)
    traceback(i, j - 1)
  else:
    find the t that maximizes 1 + OPT(i, t - 1) + OPT(t + 1, j)
    mark (t, j) as a pairing
    traceback(i, t - 1)
    traceback(t + 1, j)
```

This traceback has an $$ \mathcal{O}(n^2) $$ runtime. If we wanted to improve the runtime of the traceback, we could make space tradeoffs and store the actual optimal pairings (rather than just the number of them) in the $$ OPT $$ matrix. However, this doesn't improve the overall asymptotic runtime of the full solution, since the $$ \mathcal{O}(n^3) $$ step still dominates the $$ \mathcal{O}(n^2) $$ step.

## Footnotes

[^0]: Technically, I have no credentials -- my graduation date is in about 5 weeks.
[^1]: Much of this is paraphrased from the fantastic textbook [*Algorithm Design*](https://www.amazon.com/Algorithm-Design-Jon-Kleinberg/dp/0321295358) by Jon Kleinberg and Éva Tardos.
[^2]: Many of these pairings will not be valid by our rules, but I don't believe this changes the asymptotic bound.
[^3]: Note that $$ OPT $$ gives us the optimal *number* of pairings for the strand from $$ b_1 $$ to $$ b_n $$, not the pairings themselves. This is okay -- later, we will trace back through our calculated values of $$ OPT $$ to recover the optimal solution.
