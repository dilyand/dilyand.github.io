---
layout: post
title: Should `flatMap` be called `mapFlat`?
tags: [Scala, flatMap]
---

The name of `flatMap` might trick you into thinking that it is the same as `flatten` followed by `map`. But it's exactly the other way around: it's `map` followed by `flatten`.

Behold:

```
List(List(1, 2, 3), List(4, 5, 6)).flatten.map(_ + 1)
// res5: List[Int] = List(2, 3, 4, 5, 6, 7)

List(List(1, 2, 3), List(4, 5, 6)).flatMap(_ + 1)
// If _.flatMap(f) was the same as _.flatten.map(f), this should work in the same way.
// Instead, we get:
//  type mismatch;
//  found   : Int(1)
//  required: String
//  List(List(1, 2, 3), List(4, 5, 6)).map(_ + 1).flatten

List(List(1, 2, 3), List(4, 5, 6)).map(_ + 1).flatten 
// Same result as with flatMap: 
//  type mismatch;
//  found   : Int(1)
//  required: String
//  List(List(1, 2, 3), List(4, 5, 6)).map(_ + 1).flatten

```

For `flatMap` to work in this example, it needs to be passed a function that can be applied to `List`, eg:

```
List(List(1, 2, 3), List(4, 5, 6)).flatMap(0 :: _)
// res6: List[Int] = List(0, 1, 2, 3, 0, 4, 5, 6)
```

But not something that reduces the `List` to an un-flattenable value:

```
List(List(1, 2, 3), List(4, 5, 6)).flatMap(_.sum)
// type mismatch;
// found   : Int
// required: scala.collection.GenTraversableOnce[?]
// List(List(1, 2, 3), List(4, 5, 6)).flatMap(_.sum)
```
