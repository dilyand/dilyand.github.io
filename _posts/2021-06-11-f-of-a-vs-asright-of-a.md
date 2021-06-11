---
layout: post
title: `F[A]` is 'F of A' but what is `asRight[A]`?
tags: [Scala, Either, Left, Right]
---

A few times I've fallen into the trap of making incorrect assumptions about what a type is based on how I read it in my head. Most recently, this happened with the `asRight` and `asLeft` wrappers of `cats.Either`.

Lo:

```
import cats.implicits._

case class Bad(i: Int)
case class Good(i: Int)
type Parsed = Either[Bad, Good]

def check(value: Int): Parsed =
  if (value >= 0) Good(value).asRight[Bad] else Bad(value).asLeft[Good]

check(0)
// val res0: Parsed = Right(Good(0))

check(1)
//val res1: Parsed = Right(Good(1))

check(-34644425)
// val res2: Parsed = Left(Bad(-34644425))
```

At first glance, I was surprised that it compiles, because it appeared that in the `check` function we're wrapping the results the wrong way around: the `Good` results as a `Bad` and vice versa.

Of course, it's clear from the signatures of `asRight` and `asLeft` that it's not the case:

```
  /**
   * Wrap a value in `Left`.
   */
  def asLeft[B]: Either[A, B] = Left(obj)

  /**
   * Wrap a value in `Right`.
   */
  def asRight[B]: Either[B, A] = Right(obj)
```

The type parameter `B` in either case is the type of the value that goes on the 'opposite' side of the `Either`.

It makes sense, if you think about it. The wrapper already knows what type the wrapped value has because it has access to it. The information that it is missing is what the alternative is.

The reason for my confusion is that in my head when I read the code of `check` it goes like this: "If the value is greater than or equal to zero then create an instance of Good with the value and wrap it in a Right of Bad, otherwise create an instance of Bad and wrap it a Left of Good." These 'Right of Bad' and 'Left of Good' are a natural extension of the reading 'F of A' when faced with an `F[A]`. But they are misleading in this case. In this case, the proper reading is actually 'Right with a Left of Bad' / 'Left with a Right of Good'.

A more intuitive signature for the `asRight` and `asLeft` wrappers would be something along the lines of `asEither[Left, Right]`, which has its own downsides.
