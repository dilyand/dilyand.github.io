---
layout: post
title: Naming things is hard
tags: [Scala, foldRight]
---

I recently realised that ambiguous argument naming has been a major obstacle for me building intuition about how `foldRight` works in Scala.

To illustrate, here's a simplified definition of `foldRight`:

```
def foldRight[A, B](as: List[A], z: B)(f: (A, B) => B): B =
  as match {
    case Nil => z
    case h::t => f(h, foldRight(t, z)(f))
  }
```

and here's a function implemented in terms of `foldRight`:

```
def doubleToString(l: List[Double]): List[String] =
  foldRight(l, Nil:List[String])((h, t) => h.toString::t)
```

(These examples come from the exercises in the awesome *Functional Programming in Scala* by Paul Chiusano and Runar Bjarnason.)

Both of these functions operate on a `List` and they both in some way de-structure the list to its `head` and `tail`. Their signatures also include arguments named `h` and `t` in their inner scope. It is the naming of these arguments that is at the root of the challenge. Intuitively, when reading the definition of `doubleToString`, I expand `h` to `head` and `t` to `tail`. But, while this expansion is valid for `h` in both function definitions, it is only valid for `t` in the definition of `foldRight`. In the definition of `doubleToString`, `t` actually means not `tail` but rather `foldRight(tail, Nil)((head, a) => head.toString::a)`. The `a` in that last snippet can itself be recursively replaced with `foldRight(a.tail, Nil)(...)`. It is this recursion, which is part of the definition of `foldRight`, that causes the trouble. If all I see is the definition of `doubleToString`, it is not obvious to me that this recursion exists. I must remember that `foldRight` is a recursive function and that `t` in `doubleToString` is also recursive.

My issue with `h` and `t` in the definition of `doubleToString` is that the instinctive expansion to `head` and `tail`, drilled into me by so many other occurrences of this shorthand, prevents me from clicking through and checking the definition of `foldRight`. I make incorrect assumptions, which lead to incorrect interpretations.

By contrast, naming the argument something like `a` forces me to investigate, or else I would have no idea what it is. I can no longer automatically assume that it has anything at all to do with the List `l`:

```
def doubleToString(l: List[Double]): List[String] =
  foldRight(l, Nil:List[String])((h, a) => h.toString::a)
```

To make it even clearer, `a` can be called something like `acc` (for `accumulator`), which hints at its recursive nature.
