# Implementing lists

You may wonder why we'll be looking at implementations in an introductory
text to Scala. First, we'll look at lists since they're the most used data
structure in Scala, and second, the idea is that it presents many common
techniques that can be applied when required to design your own library
and also, opens the door to understanding how to use lists more efficiently.

## Lists

- Lists are not built-in as a language construct, they are rather defined
by an abstract class `List` in the `scala` package

We'll start with a gentle introduction to the implementation and we will
later on move to the details. First off,

```scala
package scala
abstract class List[+T] {
```

`List` is an abstract class: clearly you cannot instantiate it and hence the
expression `new List` would be illegal. The class has a type
parameter: `T`, and the `+` in front of it specifies that lists are
covariant. As explained, this allows assignment of type `List[Int]` to a
`List[Any]`, for example.

- All lists operations can be defined with the three basic methods

```scala
def isEmpty: Boolean
def head: T
def tail: List[T]
```

There three methods are all abstract in the `List` class, and are defined
in the subobject `Nil` and the subclass `::`. But what are they?

### The `Nil` object

- The `Nil` object simply defines an empty list
- The `Nil` object inherits from `List[Nothing]`, and because of covariance,
    this means that `Nil` is compatible with every instance of `List`

```scala
case object Nil extends List[Nothing] {
  override def isEmpty = true
  def head: Nothing = throw new NoSuchElementException()
  def head: List[Nothing] = throw new NoSuchElementException()
}
```

- Note that sine there's no value of type `Nothing`, we practically have no
    choice but to throw the exceptions

### The `::` class

- The class `::`, which is pronounced "cons" (for construct), represents
    non-empty lists

- It's worth to note that it's named that way to support pattern matching
with the infix `::`
- As already explained, every infix operation in a pattern is
treated as a constructor application of the infix operation to its
arguments
- For instance `x :: rest` is treated as `::(x, rest)` where `::` is a
case class

Let's see the actual definition:

```scala
final case class ::[T](head: T, tail: List[T]) extends List[T] {
  override def isEmpty: Boolean = false
}
```

Here, the parameters directly implement the `head` and `tail` methods of the
superclass `List`, because every case class parameter is implicitly also
a field of the class (as if the parameter declaration was prefixed
with `val`) and as mentioned in the abstract members lecture, you
can implement an abstract parameterless method such as `head` or `tail`
with a field

### Other `List` methods

- Calculating the length of the list

```scala
def length: Int = if (isEmpty) 0 else 1 + tail.length
```

- Dropping elements

```scala
def drop(n: Int): List[T] =
  if (isEmpty) Nil
  else if (n <= 0) this
  else tail.drop(n - 1)
```

- Mapping elements

```scala
def map[U](f: T => U): List[U] =
  if (isEmpty) Nil
  else f(head) :: tail.map(f)
```

### List construction

- As mentioned, the list construction methods `::` and `:::` are rather
special
- Because they end up in a colon, they're bound to the right operand,
e.g. `x :: rest` is actually translated to `rest.::(x)`
- In general, the `::` method should take an element value and yield a
new list.  An interesting question is what should be the type of that
element value? The same as the list's element type?

```scala
abstract class Fruit
class Apple extends Fruit
class Orange extends Fruit

val apples = new Apple :: Nil
val fruits = new Orange :: apples
```

- As you might've noticed, the definition of `fruits` shows that possible to
add an element of a different type to some list
- The element type of the resulting list is `Fruit`, which is the most
precise common supertype of the original list element type (`Apple`) and
the type of the element to be added (`Orange`)

This is possible mainly because of the definition of the `::` method:

```scala
def ::[U >: T](x: U): List[U] = new scala.::(x, this)
```
