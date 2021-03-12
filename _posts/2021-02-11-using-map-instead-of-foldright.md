---
layout: post
title: Using `foldRight` instead of `map`
tags: [Scala, foldRight]
---

Say you want to implement a function `add1` that takes a list of integers and adds 1 to each one, so that:

```
add1(List(1, 2, 3)) == List(2, 3, 4)
```

This is trivial with `map` but can `add1` be implemented in terms of `foldRight`?

The signature of `foldRight` has two type parameters `A` and `B`:

```
foldRight[B](z: B)(op: (A, B) => B): B
```

There is some implicit knowledge that is required here to fully understand that signature. Firstly, `A` and `B` can be the same type, eg an `Int`. Secondly, `foldRight` operates on collections, such as `List`, collapsing them to a single value. `A` is the type of each element in the collection.

If we think about turning `List(1, 2, 3)` into `List(2, 3, 4)`, we might fall into the trap of thinking that we need to "get into" the list and then traverse over each element in turn by applying a function to it. That is certainly what `map` would do, but it is incorrect with `foldRight`. What `foldRight` does is it traverses over the elements of the list, applying a function, but adds the result of that operation to an accumulator until ultimately all elements are processed and the final value of the accumulator is returned. This can be seen in its signature, where `B` is the type returned by the function that is being applied (`op`) and also the return type of the outer `foldRight` function. It is also the type of `z`, the initial value of the accumulator.

So, to implement `add1` in terms of `foldRight`, we need to collapse the original list into a single value, which in this case happens to be another list, with the same number of elements which also have the same type:

```
def add1(l: List[Int]): List[Int] =
  l.foldRight(Nil: List[Int])((x, y) => (x + 1)::y)
```

The parameter `y` is the accumulator. It starts at `Nil` and then each element of the original list is prepended to it after getting incremented by 1. Since `foldRight` is not tail-recursive, it will first prepend the last element of the original list to the accumulator, then the second to last, etc.

Another way to implement `add1` without using `map` is to write a recursive function that pattern-matches the original list. The pattern-match will have two branches, one for the base case (which here would be the empty list `Nil`) and a recursive branch for a non-empty list:

```
def add1(l: List[Int]): List[Int] = l match {
  case Nil => Nil
  case h::t => (h + 1)::t
}
```

So any time we have a pattern-match of this kind:

```
l match {
  case Nil => ???
  case h::t => ???
}
```

where `l` is some `List`-like collection, we can replace it with `foldRight`. The base-case branch is what our `z` would be and the non-empty case branch would be `op` that adds values on top of `z`, both ultimately returning the same type.
