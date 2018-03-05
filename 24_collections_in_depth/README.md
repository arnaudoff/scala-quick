# Collections in depth

We've only barely touched on the power, usability and conciseness of the
Scala collections library. This lecture is devoted to showing if not all,
most of the details around the most used collections and operations on them.

## Mutable and immutable collections

- As already explained, Scala collections are separated into two types:
    mutable and immutable
- A mutable collection can be updated in place: you can change, add or
    remove elements as a side effect, whereas an immutable collection never
    changes: new collection is returned instead

All collection classes are placed in `scala.collection`, or in the
subpackages `mutable`, `immutable`, or `generic`.

- A collection in `scala.collection.immutable` is guaranteed to be immutable
    meaning it will never change once created; repeated access in different
    points in time will always yield the same collection
- A collection in `scala.collection.mutable` possibly has some operations
 that change the collection in place, so you must carefully defend against
 any updates performed by other parts of the code base
- The package `scala.collection.generic` contains code for implementing
    collections: some collection classes defer the implementation of some of
    their operations to classes in `generic`

Generally, the root collections in `scala.collection` are either mutable or
immutable (with respect to the user), define the same interface as the
immutable collections, and typically the mutable collections in the
mutable subpackage implement this interface.
parts of it with side-effects.

## The `Traversable` trait

At the top of the collection hierarchy, which has one and only abstract
operation:

```scala
def foreach[U](f: Elem => U)
```

which is meant to traverse all elements of the collection and apply the
given operation `f` to each element, where `Elem` is the type of the
collection's elements and `U` is some result type.

The `Traversable` trait also defines many concrete methods:

| xs ++ ys                                                                                              | Joins the elements of xs and ys where ys is a TraversableOnce (Traversable or Iterator)                                               |
|-------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| xs map f                                                                                              | Returns a collection which maps the function f onto every element in xs                                                               |
| xs flatMap f                                                                                          | Returns a collection which maps the collection-valued function f onto every element in xs and concatenates the result                 |
| xs collect f                                                                                          | Returns a collection which is obtained from applying f to every element in xs and collects the result                                 |
| xs.toArray ; xs.toList ; xs.toIterable ; xs.toSeq; xs.toIndexedSeq; xs.toStream ; xs.toSet ; xs.toMap | Converts xs to whatever the method name is                                                                                            |
| xs copyToBuffer buffer                                                                                | Copies all elements of xs to a buffer                                                                                                 |
| xs.isEmpty ; xs.nonEmpty                                                                              | Tests whether xs is empty or contains elements                                                                                        |
| xs.size ; xs.hasDefiniteSize                                                                          | The number of elements in xs; true if xs has a finite size                                                                            |
| xs.head                                                                                               | The first element of xs (could be random if the collection is unordered)                                                              |
| xs.headOption                                                                                         | The first element of xs wrapped in an option value or None if empty                                                                   |
| xs.last                                                                                               | The last element of xs (could be random if the collection is unordered)                                                               |
| xs.lastOption                                                                                         | The last element of xs wrapped in an option value or None if empty                                                                    |
| xs find predicate                                                                                     | An option containing the first element in xs that satisfies predicate or None otherwise                                               |
| xs.tail                                                                                               | The rest of xs except xs.head                                                                                                         |
| xs.init                                                                                               | The rest of the collection except xs.last                                                                                             |
| xs slice (from, to)                                                                                   | A collection of the elements starting from the index from up to to exclusive                                                          |
| xs take n                                                                                             | A collection that has the first n elements of xs (or random if no order is defined)                                                   |
| xs drop n                                                                                             | The rest of the collection except xs take n                                                                                           |
| xs takeWhile predicate                                                                                | The longest prefix of elements that all satisfy predicate                                                                             |
| xs dropWhile predicate                                                                                | The collection without the longest prefix of elements that all satisfy predicate                                                      |
| xs filter predicate                                                                                   | The collection of elements from xs that satisfy predicate                                                                             |
| xs withFilter predicate                                                                               | A non-strict filter of the collection where all operations that follow will only apply to those elements of xs that satisfy predicate |
| xs splitAt n                                                                                          | Splits xs at a position; equivalent to (xs take n, xs drop n)                                                                         |
| xs span predicate                                                                                     | Splits xs according to a predicate, equivalent to (xs takeWhile predicate, xs dropWhile predicate)                                    |
| xs partition predicate                                                                                | Splits xs into a pair of collections, equivalent to (xs filter predicate, xs filter !predicate)                                       |
| xs groupBy f                                                                                          | Partitions xs into a map of collections according to f                                                                                |
| xs forall predicate                                                                                   | Indicates whether a the predicate holds for all elements of xs                                                                        |
| xs exists predicate                                                                                   | Indicates whether a predicate holds for some element in xs                                                                            |
| xs count predicate                                                                                    | The number of elements in xs that satisfy the predicate                                                                               |
| (z /: xs)(op) ; xs.foldLeft(z)(op)                                                                    | Applies the binary operation op between successive elements of xs, going left to right (left leaning tree), starting with z           |
| (xs :\ z)(op) ; xs.foldRight(z)(op)                                                                   | Applies the binary operation op between successive elements of xs, going right to left (right leaning tree), starting with z          |
| xs reduceLeft op                                                                                      | Applies the binary operation op between successive elements of xs, going left to right                                                |
| xs reduceRight op                                                                                     | Applies the binary operation op between successive elements of xs, going right to left                                                |
| xs.sum ; xs.product ; xs.min ; xs.max                                                                 | No explanation needed                                                                                                                 |
| xs.view                                                                                               | Produces a view over xs                                                                                                               |
| xs view (from, to)                                                                                    | Produces a view that represents the elements of xs in the given range                                                                 |
