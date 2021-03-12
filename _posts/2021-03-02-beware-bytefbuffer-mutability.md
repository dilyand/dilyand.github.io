---
layout: post
title: Beware `java.nio.ByteBuffer` mutability
tags: [Scala, Java, ByteBuffer]
---

According to the documentation for `java.nio.ByteBuffer.wrap(array)`:

```
Wraps a byte array into a buffer.
The new buffer will be backed by the given byte array;
that is, modifications to the buffer will cause the array to be modified
and vice versa.
```

Every time you operate on a `ByteBuffer`, you modify it.

Here's an example comparing how `ByteBuffer` behaves vs `Array[Byte]`. Imagine you receive binary data (such as Thrift-encoded records), which you want to process and then write to a message queue.

```Scala
import java.nio.ByteBuffer

// a single record received from upstream
val msgBytes: Array[Byte] = "I am a test message.".getBytes()

// the same record wrapped in a ByteBuffer
val msgBuff: ByteBuffer = ByteBuffer.wrap(msgBytes)

// scenario A: a stream of records wrapped in a ByteBuffer
val batchA = List.fill(3)(msgBuff)
val resultA = batchA.map(m => new String(java.util.Base64.getEncoder.encode(m).array))
println(resultA)

// scenario B: a stream of records kept as Array[Byte]
val batchB = List.fill(3)(msgBytes)
val resultB = batchB.map(m => new String(java.util.Base64.getEncoder.encode(m)))
println(resultB)
```

```bash
> List(SSBhbSBhIHRlc3QgbWVzc2FnZS4=, , ) // A
> List(SSBhbSBhIHRlc3QgbWVzc2FnZS4=, SSBhbSBhIHRlc3QgbWVzc2FnZS4=, SSBhbSBhIHRlc3QgbWVzc2FnZS4=) // B
```
