# Futures and concurrency

As we said in the beginning, Java provides concurrency support that is based
upon shared memory and locking, which turns out to be quite difficult to
maintain in practice. Scala solves this problem with `Future`s: a mechanism for
asynchronous transformations of immutable state.

In a sense, the `Future` is an object on which you can specify transformations,
where each new transformation results in a new `Future` representing the
asynchronous result of the original `Future` transformed by the function.

## The shared state and locking model

- On the Java platform, each object is associated with a logical monitor, which
    can be used to control the multi-threaded access to the object
- In other words, you decide what data will be shared and mark as `synchronized`
    sections of code where race conditions can occur
- As such, only one thread at a time enters `synchronized` sections guarded by
    the same lock

```scala
var counter = 0
synchronized {
  counter = counter + 1 // one thread executes this at a time
}
```

- This approach, however, has turned out to be difficult as applications scale,
because at each point in the program you must reason about what data is
modified that could potentially be modified by other threads, what locks
are held, when deadlocks occur etc.
- Over-synchronizing is also as bad as not syncronizing at all, because dead
    locks possibilities arise
- Testing is also hard when using such model, because threads are

The `java.util.concurrent` library provides higher level abstractions for
concurrent programming, but although it makes programming less error prone, it
ultimately they're based on the shared data and locks model so are not a
solution for the difficulties of the model.
non-deterministic

## Asynchronous execution and `Try`s

The `Future` abstraction can solve the problems that arise when using the
shared data and locks model. As we know, when you invoke a method, it performs
some computation (while you wait) and returns a result. If that result is a
`Future`, the `Future` represents another computation to be performed
asynchronously (often times by a completely different thread).

- Based on the above facts, often when you perform operations on `Future`,
you'll need to provide an execution context which itself provides some
strategy for executing functions asynchronously, so

```scala
import scala.concurrent.Future
val future = Future { Thread.sleep(10000); 33 }
```

will not compile as no `ExecutionContext` is provided. However, you can bring
an implicit global execution context into scope, which is provided by the runtime:

```scala
import scala.concurrent.ExecutionContext.Implicits.global
val future = Future { Thread.sleep(10000); 33 }
```

So, the future we created will asynchronously execute using the global execution
context and will take at least 10 seconds to complete (return 33). Meanwhile,
you can check the status of the `Future` with two methods: `isCompleted` and
`value`:

```scala
future.isCompleted // will return true after at least 10 seconds
```

The more interesting method is `value`: it returns an `Option` value that is
also a `Try` value. In other words, `future.value` from the example could return
either `None` of type `Option[scala.util.Try[Int]]` or `Some(Success(33))` also
of type `Option[scala.util.Try[Int]]`.

- In general, a `Try` is either a `Success` which contains a value of type `T`
    or a `Failure` which contains an exception.
- The purpose of `Try` is to provide the `try` expression for synchronous code,
    but for asynchronous: put differently, it's a mechanism that encapsulates
    the possibility of a computation completing with an exception rather than a
    result

In order words, `Try` exists because, typically, for asynchronous tasks the thread
that initiates the computation often moves to other tasks, so later if that
computation fails with an exception the original thread is unable to handle the
exception in a `catch` clause.

## Using `Future`

As mentioned, `Future` allows you to specify transformations on `Future` results
and obtain a new future that represents the composition of the two asynchronous
computations.

### Mapping `Future`s

Instead of blocking then continuing with another computation, one can simply map
the next computation onto the future: the result will be a future that
represents the original asynchronously computed result transformed asynchrnously
by the function passed to `map`.

```scala
val future = Future { Thread.sleep(30000); 30 + 2 }
val result = future.map(x => x + 1)
```

Once the original future completes (after at least 30 seconds), the function is
applied to its result and then the future returned by the map (result) will
complete:

```scala
result.value // Some(Success(33))
```

- Note that the future creation, the sum and the increment could be performed in
    three separate threads.

## Transforming `Future`s with `for` expressions

- Since `Future` also has a `flatMap`, we can manipulate futures using `for`
    expressions

Say we have to following two futures:

```scala
val firstFuture = Future { Thread.sleep(10000); 5 + 5}
val secondFuture = Future { Thread.sleep(10000); 36 - 13}
```

If we want to obtain a future that represents the asynchrnous sum of their
results, we can do it like that:

```scala
val result = for {
  x <- firstFuture
  y <- secondFuture
} yield x + y
```

After the original two futures complete and the final sum completes

```scala
result.value // 33
```

It's also worth to note that if you don't create the futures before the `for`,
they don't run in parallel: for instance, this requires at least 20 seconds:

```scala
for {
  x <- Future { Thread.sleep(10000); 5 + 5}
  y <- Future { Thread.sleep(10000); 36 - 13}
} yield x + y
```

## More ways to create a `Future`

The `Future` companion object includes three factory methods for creating
    already-completed futures

### Create a future that has already succeeded

```scala
Future.successful { 30 + 3 }
```

### Create a future that has already failed

```scala
Future.failed(new Exception("something failed while computing the future"))
```

### Create an already completed future from a `Try`

```scala
import scala.util.{Success, Failure}
Future.fromTry(Success { 30 + 3})
Future.fromTry(Failure(new Exception("something failed in the future")))
```

### Creating futures from `Promise`

Given a promise, you can obtain a future that's controlled by the promise; the
future will complete when the promise is completed

```scala
val somePromise = Promise[Int]
val promisesFuture = somePromise.future
promisesFuture.value
```

The promise has three methods: `success`, `failure` and `complete` that are
similar to those described used for constructing already completed futures.

```scala
somePromise.success(33) // completes the future successfully
promisesFuture.value  // Some(Success(33))
```

Analogously, the `failure` method accepts an exception that causes the future to
fail. The `complete` method takes a `Try`. By the way, you can complete a
promise with the result of other future, which is done with `completeWith`.

### Ensuring future value properties

There exist two future methods that allow you to ensure a certain property holds
about a future value: `filter` and `collect`.

```scala
val future = Future { 33 }
val valid = future.filter(result => result > 0)
valid.value // Some(Success(33))
```

If the future value is invalid, the value returned by `filter` will fail with an
exception.

It's worth to note that since `Future` also provides a `withFilter` method,
you can use `for` expression with filters to accomplish the same thing:

```scala
val valid = for (result <- future if result > 0) yield result
valid.value
```

Now, let's look at `collect`: it also allows you to validate the future value,
but you can also transform it in one operation:

```scala
val valid = future collect { case result if result > 0 => result - 33 }
valid.value // Some(Success(0))
```
```
