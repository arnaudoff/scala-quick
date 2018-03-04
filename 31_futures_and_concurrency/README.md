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

### Dealing with failure

There's four methods that deal with futures that fail: `failed`, `recover`,
`fallbackTo`, `recoverWith`.

Let's look at `failed` first: its idea is to transform a failed future of any
type **into a successful future** `Future[Throwable]` that holds the exception
that caused the failure:

```scala
val failure = Future { 33 / 0 }
failure.value // Some(Failure(java.lang.ArithmeticException))
val successfulFuture = failure.failed
successfulFuture.value // Some(Success(java.lang.ArithmeticException))
```

Clearly, it's a good idea to use `failed` when you expect a future to fail.

Now, let's move on to `fallbackTo`: it allows you to provide an alternate future
to use in case the future fails

```scala
val fallback = failure.fallbackTo(success)
fallback.value // Some(Success(foo))
```

Needless to say, if the `success` future also fails, it is ignored and the
failure of the initial future (which `fallbackTo` was invoked on) will be
returned.

Onto the `recover` method: allows you to transform a failed future into a
successful one, however, if a the future is successful, the result is allowed to
pass through unchanged:

```scala
val failed = Future { 33 / 0 }
val failureRecovery = failed recover {
  case ex: ArithmeticException => -1
}
failureRecovery.value // Some(Success(-1))

val successful = Future { 33 / 1 }
val successRecovery = successful recover {
  case ex: ArithmeticException => -1
}
successRecovery.value // Some(Success(33))
```

Finally, `recoverWith` does the same as `recover`, but allows you to recover
with a future value:

```scala
val recovered = failed recoverWith {
  case ex: ArithmeticException => Future { 33 }
}
```

Analogously to `recover`, if the `failed` feature didn't fail, the original
result will be passed through `recovered`.

### Mapping success and failure with `transform`

`Future` has a `transform` method, which takes two functions with which it
transforms a future depending on the success/failure of the future:

```scala
val first = success.transform(
  res => res + 1,
  ex => new Exception("something bad happened", ex)
)
```

If the future succeeds, the first function is used for the transform:

```scala
first.value // Some(Success(34))
```

otherwise, the second function is used:

```scala
second.value // Some(Failure(java.lang.Exception))
```

However, you cannot change a successful future into a failed one and vice versa;
you have to use an overloaded `transform` that takes a function from `Try` to
`Try`

```scala
val first = success.transform {
  case Success(res) => Success(res + 1)
  case Failure(ex) => Failure(new Exception("something bad", ex))
}

first.value // Some(Success(34))
```

Clearly, this form  allows you to also flip a failure into a success and vice versa.

### Combining futures

Multiple futures can be combined. For example, `zip` will transform two
successful futures into a future tuple of both values

```scala
// firstFuture returns Some(Success(33)) and secondFuture returns Some(Success(35))
val futures = firstFuture zip secondFuture
futures.value // Some(Success((33, 35)))
```

There's a more interesting method for combining, though. It's the `fold` method,
which allows you to accumulate a result across a collection of futures.

- If all futures in the collection succeed, the resulting future succeeds with
the accumulated result; if any future in the collection fails, the resulting
future fails

```scala
val five = Future { 1 + 4 }
val seven = Future { 2 + 5 }
val futureNumbers = List(five, seven)
val folded = Future.fold(futureNumbers)(0) { (acc, num) => acc + num }
```

Now, `folded.value` should equal `Some(Success(12))`.

There's another method similar to `fold` called `reduce`, which performs a fold
without a zero: instead, the initial future result is used as the start value.

```scala
val reduced = Future.reduce(futureNumbers) { (acc, num) => acc + num }
```

And the last method of this class of methods we'll look at is `sequence`, which
transforms a collection of futures into a future of the values, e.g a
`List[Future[Int]]` to a `Future[List[Int]]`:

```scala
val futureList = Future.sequence(futureNumbers)
futureList.value // Some(Success(List(5, 7)))
```

### Performing a side-effect after a future completes

`Future` provides several methods that allow you to perform some side effect
after a future completes. For instance, `foreach` will perform a side effect as
soon as a future completes successfully:

```scala
success.foreach(res => println(res)) // 33
```

You can also register callback functions to futures. The `onComplete` method,
for instance, will be executed when the future succeeds or fails:

```scala
import scala.util.{Success, Failure}
success onComplete {
  case Success(res) => println(res)
  case Failure(ex) => println(ex)
}
```

## Testing with `Future`s

As early mentioned, one advantage of futures is that they avoid blocking. By
avoiding it, you can keep a finite number of threads you decide to work with
busy. However, if you need to, you can block on a future result: the `Await`
object allows for blocking to wait for future results.

For instance, `Await.result` takes a `Future` and a `Duration`, where the latter
indicates how long `result` should wait for the `Future` to complete before
timing out:

```scala
import scala.concurrent.Await
import scala.concurrent.duration._

val future = Future { Thread.sleep(10000); 15 + 18 }
val x = Await.result(future, 15.seconds) // blocks
```

The above technique is especially useful when testing asynchronous code. As long
as `Await.result` has returned, you can perform a computation using that result

```scala
import org.scalatest.Matchers._
import org.scalatest.Matchers._

x should be (33)
```

Alternatively, you can use the `ScalaFutures` trait, which
supports blocking constructs. For instance, `futureValue` (which is implicitly
added to `Future` by `ScalaFutures`) will block until the future completes:

```scala
import org.scalatest.concurrent.ScalaFutures._

val future = Future { Thread.sleep(10000); 15 + 18 }
future.futureValue should be (33) // blocks
```
