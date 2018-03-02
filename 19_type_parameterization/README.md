# Type parameterization

Type parameterization allows you to write generic classes and traits,
just like in other languages that have generics support. In this lecture,
we'll be writing an example which will show the flexibility of
type parameterization in Scala and will also introduce some techniques for
information hiding as well as some terminology.

## Functional queues

In our example, we'll be developing a functional queue. A functional queue
is a data structure that has three operations: `head`, `tail` and `enqueue`.

- `head` returns the first element of the queue
- `tail` returns a queue without the first element
- `enqueue` returns a new queue with a given element appended at the end

A functional queue, as opposed to a mutable one, does not change its
contents when an element is appended; instead, a new queue is returned:

```scala
val queue = Queue(1, 2, 3)
val newQueue = q enqueue 4 // Queue(1, 2, 3, 4)
queue // Queue(1, 2, 3)
```

In fact, purely functional queues are rather similar to the classical
immutable Scala list, with the difference that while a list is extended at
the front with `::`, the queue is extended at the end using `enqueue`. So,
let's try a functional queue with a `List` backend:

```scala
class Queue[T](elems: List[T]) {
  def head = elems.head
  def tail = new Queue(elems.tail)
  def enqueue(x: T) = new Queue(elems ::: List(x))
}
```

There's a problem in this implementation: `enqueue` has a linear time
complexity. Even if you try to reverse the order of elements in the list,
then `head` and `tail` will be with linear complexity.

The solution to the problem is to have a queue represented not by one, but
by two lists: `leading` and `trailing`:

- `leading` list contains elements towards the front
- `trailing` list contains elements towards the back in reversed order

and a `mirror` operation, which copies the whole `trailing` list to `leading`
in reversed order, before the first head or tail is performed.

So, the contents of the queue as a whole are `leading ::: trailing.reverse`.
Now, to append an element, just cons it to the trailing list and `enqueue` will
be constant time. Complexity-wise, `mirror` will be linear in the beginning
when the `leading` list is empty, but will return directly when it's non-empty.
Informally, the longer the queue gets, the less often mirror is called.

```scala
class Queue[T](private val leading: List[T], private val trailing: List[T]) {
  private def mirror =
    if (leading.isEmpty)
      new Queue(trailing.reverse, Nil)
    else
      this

  def head = mirror.leading.head

  def tail = {
    val queue = mirror
    new Queue(q.leading.tail, q.trailing)
  }

  def enqueue(x: T) =
    new Queue(leading, x :: trailing)
}
```

## Information hiding

- The `Qeueue` implementation is rather good with regards to asymptotic
    complexity, however, there's more work to be done design-wise
- For instance, the constructor takes two lists are parameters, where one is
    reversed: we should strive for hiding this implementation detail

### Private constructors and factory methods

- In most languages, a constructor can be made private, but in Scala the primary
    constructor is defined implicitly by the class parameters
- Still, you can accomplish this by adding a `private` modifier in front of the
    class parameter list

```scala
class Queue[T] private (
  private val leading: List[T],
  private val trailing: List[T]
)
```

- Now, the constructor can be accessed only from within the class and its
    companion object

There's three ways to still make our `Queue` accessible:

#### Auxiliary constructors

One possibility is to define an auxiliary constructor

```scala
def this() = this(Nil, Nil)
```

or

```scala
def this(elems: T*) = this(elems.toList, Nil)
```

in case you want the queue to support initial elements.

#### Factory methods

Another way to implement the same thing is using a factory method that builds
the queue. A nice way to do it is to define an object `Queue` that has
an `apply` method:

```scala
object Queue {
  def apply[T](elems: T*) = new Queue[T](elems.toList, Nil)
}
```

When we made the constructor private, we said that it can be instantiated from
the companion object as well, so if we place the `Queue` object in the same
file as the class we're all set.

There's a second observation to be made: clients create a `Queue` object with
`Queue(1, 2, 3)` syntax, however, this expression expands to `Queue.apply(1, 2,
3)` since `Queue` is an object and as a result, `Queue` looks as if it was a
globally defined factory method, but in reality Scala has no such methods:
every method should be contained in an object or a class and using `apply` is
the way to simulate such global methods.

#### Private classes 

We already saw private constructors and private members as a way to hide the
initialization of a class. Another technique is to hide the class itself and
only export a trait that maps to the public interface of the class. Instead of
hiding individual constructors/methods, this technique hides the whole
implementation class:

```scala
trait Queue[T] {
  def head: T
  def tail: Queue[T]
  def enqueue(x: T): Queue[T]
}

object Queue {
  def apply[T](xs: T*): Queue[T] =
    new QueueImpl[T](xs.toList, Nil)

  private class QueueImpl[T](
    private val leading: List[T],
    private val trailing: List[T]
  ) extends Queue[T] {
    def mirror =
      if (leading.isEmpty)
        new QueueImpl(trailing.reverse, Nil)
      else
        this

    def head = mirror.leading.head

    def tail: QueueImpl[T] = {
      val queue = mirror
      new QueueImpl(q.leading.tail, q.trailing)
    }

    def enqueue(x: T) =
      new QueueImpl(leading, x :: trailing)
  }
}
```

## Variance annotations

`Queue`, as we last defined it, is a trait. Therefore you cannot define
variables of type `Queue`, because it takes a type parameter and hence is not a
type:

```scala
def foo(q: Queue) = {} // does not compile
```

instead, the trait lets you supply parameterized types such as `Queue[Int]`,
`Queue[String]` etc.:

```scala
def foo(q: Queue[Int]) = {}
```

- It's vital to make the distinction between the `Queue` **trait**, and
the `Queue[Int]` **type**
- To go with the distinction further, `Queue` is also
sometimes called a type constructor, because you can construct type with it by
specifying a type parameter.
- We also say that `Queue` is a *generic trait* (not to be confused with the
type it generates, which we call *parameterized*)

So, we said that `Queue[T]` generates a family of types. It's natural to look at
the relationships between these types. For instance, are there any subtyping
relationships between these types, e.g is `Queue[Int]` a subtype of
`Queue[AnyRef]`?
