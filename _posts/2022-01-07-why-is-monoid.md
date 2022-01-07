---
layout: post
title: Why is Monoid?
tags: [Scala, Monoid, MapReduce]
---

`Monoid` is fairly easy to understand, even if you're just starting out in functional programming. First, you need to understand what a `Semigroup` is. The Cats definition is very accessible:

> If a type `A` can form a `Semigroup` it has an associative binary operation.

```
trait Semigroup[A] {
  def op(x: A, y: A): A
}
```

And associativity means that elements can be combined using `op` in random order:

```
op(a, op(b, c)) == op(op(a, b), c)
```

The classic example is the addition of integers:

```
val sumInt: Semigroup[Int] = new Semigroup {
  def op(x: A, y: A): A = x + y
}
```

The `op` binary operator of `sumInt` is associative because `(1 + (2 + 3)) = ((1 + 2) + 3)`.

By contrast, subtraction of integers is not a valid example of a semigroup, because even though the `-` operator is binary, it is not associative: `(3 - (2 - 1)) =/= ((3 - 2) - 1)`.

Now that we understand `Semigroup`, we can introduce `Monoid`. To borrow from Cats again:

> `Monoid` extends the power of `Semigroup` by providing an additional empty value.

```
trait Monoid[A] extends Semigroup[A] {
  def id: A
}
```

The empty value, also known as 'identity' value, is such that when used in `op`, the output is no-op.

For example, addition of integers forms a monoid, because not only is its binary operation associative, but it also has `0` as the identity element: `1 + 0 = 1`.

Other monoid examples include multiplication of integers (with `id` of `1`) and concatenation of strings (with the empty string `""` as identity element).

When you read about these concepts through the lens of functional programming or mathematics / category theory, it's easy to be left with the impression that things like semigroup and monoid just *exist*, and we are simply discovering and describing them. But of course there's another way of looking at them: that they are invented by us, and we've invented them such as they are for a reason.

So what is the reason that `Semigroup` must have an associative operator? And why does `Monoid` need an identity element?

To understand this, let's look at an example.

Take a nested list of values, like `List(List(1, 2, 3), List(4, 5))`. We might want to collapse this into a single value, by applying some operation to the values in each inner list, and then combining the intermediate results using the same operation. For example, maybe we want to calculate the total sum. So first, we'd want to sum `1`, `2` and `3`; then we'd want to sum `4` and `5`; and finally we'd want to sum the results of these calculations to get a total of `15`.

We might also want to make this function generic, so that it works not only with integers and addition, but any value `A` and any operation that takes two `A`s and returns an `A`. `foldLeft` seems a good choice for implementing this function, which means we'll also need a default value:

```
def func[A](l: List[List[A]], default: A)(combine: (A, A) => A) =
  l.map(_.foldLeft(default)(combine)).foldLeft(default)(combine)
```

This seems to work fine for calculating the sum of integers, and we can see that it produces the same result as flattening the list and then adding all the values together:

```
val l = List(List(1, 2, 3), List(4, 5))

func(l, 0)(_ + _)
// 15: Int

l.flatten.foldLeft(0)(_ + _)
// 15: Int
```

But what if instead of `0`, we chose `1` or `-1` as the default value?

```
func(l, 1)(_ + _)
// 18: Int

l.flatten.foldLeft(1)(_ + _)
// 16: Int

func(l, -1)(_ + _)
// 12: Int

l.flatten.foldLeft(-1)(_ + _)
// 14: Int
```

So, for `func(a)` and `a.flatten.foldLeft` to have the same result, we need the `default` value to be such that when we use it in `combine` the output is a no-op. Which is the very definition of the identity function in `Monoid`!

If we wanted to simplify `func`, we could abstract the `default` and `combine` parameters into some structure, and that structure would be a `Monoid`:

```
def func[A](l: List[List[A]])(m: Monoid[A]) =
  l.map(_.foldLeft(m.id)(m.op)).foldLeft(m.id)(m.op)
```

In other words, `Monoid` *needs* an identity function, so that it can be used for operations like the one in `func`: where we apply some `combine` function first to the inner members of a nested structure, then on the results from the first pass.

And why does `op` need to be associative?

If we think about where we might have nested structures like `List(List(1, 2, 3), List(4, 5))`, one frequent use case is in parallel computing. We might start with a very big collection of some values, and instead of going through it element by element, we might want to first break it into multiple smaller collections, which can then be processed in parallel. The intermediate results are ultimately combined together. (An example of this is [`MapReduce`](https://en.wikipedia.org/wiki/MapReduce).)

The thing is, if we started with a `List(1, 2, 3, 4, 5)` and wanted to apply a `MapReduce`, we're not guaranteed to get the data partitioned as `List(List(1, 2, 3), List(4, 5))`. It might be `List(List(1, 2), List(3, 4, 5))`. Addition of integers is an associative operation but if we instead apply a non-associative one, we see that the result is different, depending on how the data was partitioned initially:

```
val l1 = List(List(1, 2, 3), List(4, 5))
val l2 = List(List(1, 2), List(3, 4, 5))

func(l1, 0)((f, s) => (f + s) / 2)
// 2: Int

func(l2, 0)((f, s) => (f + s) / 2)
// 1: Int
```

So, in order for `MapReduce` to work with multiple 'reducers' in parallel, the `reduce` function needs to be associative, to guarantee the same result, regardless of the order in which partitions are reduced.
