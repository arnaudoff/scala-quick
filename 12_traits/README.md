# Traits

- Way to reuse code in Scala
- Look similar to interfaces in other languages
- Unlike class inheritance, where each class must inherit from only
one superclass, a class can mix in any number of traits
- There's two common ways to use traits: enriching interfaces and defining
stackable modifications
- We'll also see the `Ordered` trait as a concrete example

## Example of a trait

```scala
trait Drivable {
  def drive() = {
    println("I'm driven!")
  }
}
```

- Once a trait is defined, it can be mixed in to a class using `either` or `with` keywords
- Once you extend it, you implicitly inherit its superclass, which is
unsuprisingly `AnyRef`

For instance:

```scala
class Car extends Drivable {
  override def toString = "vw passat"
}

val passat = new Car
passat.drive()

val someCar: Drivable = passat
someCar.drive()
```

- If a class extends a superclass, instead of `extends` you use multiple
with statements to mix in the traits, e.g.

```scala
class Animal

class Horse extends Animal with Drivable {
  override def toString = "foo"
}
```

```scala
class Animal
trait Purchasable

class Horse extends Animal with Purchasable with Drivable {
  override def toString = "foo"

  override def drive() = {
    println("im not driven, im riden")
  }
}
```

We said that traits are similar to Java interfaces and that's obvious,
but traits are actually more similar to classes in Scala: they can declare
fields, maintain state, etc., except for two things:

- A trait cannot have class parameters
- Whereas in classes `super` is statically bound, in traits `super`
calls are dynamically bound so writing `super.toString` in a class
we know which method will be invoked, while in a trait the method
is determined each time the trait is mixed into a concrete class
- This behaviour of super is the foundation of the concept called
*stackable modifications*

## Thin versus rich interfaces

- One use of traits is to extend an interface, in other words,
traits can enrich a thin interface making it into a rich interface
- Rich interface has many methods; clients can pick a method that exactly
matches the functionality they need
- A thin interface has fewer methods and thus is easier on the implementers,
however they may need to write extra code to "fill in" additional
functionality

Example of a thin interface: `CharSequence`

```scala
trait CharSequence {
  def charAt(index: Int): Char
  def length: Int
  def subSequence(start: Int, end: Int): CharSequence
  def toString(): String
}
```

- The above interface is common to all string-like classes that hold a
sequence of characters
- The above interface, although most of the dozens of methods in `String`
would well apply to it, it declares only four methods; otherwise it would've
placed a burden on implementers of `CharSequence`,
e.g you'd have had to define dozens more methods
- Since Scala traits can contain concrete methods, they make rich
interfaces far more convinient to implement
- A concrete method needs to be implemented only once in the trait itself
instead of being reimplemented for every class that mixes in the trait
- So, to enrich an interface using traits, simply define a trait with
a small number of abstract methods and a large number of concrete methods,
all implemented in terms of the abstract methods
- Then, the enrichment trait can be mixed into a class, the thin portion
can be implemented, and we'll end up with a class that has all of the
rich interface available

## Examples

- As an example, we'll develop something like a graphics library, of course
we'll do just a part of it
- The idea is to have abstraction for rectangular objects that also have
geometric queries such as `width`, `height`, `left`, `right`, `topLeft` etc.
- Many such methods exist that would be nice to have and it can be tough to
program all of them for a library in Java, but in Scala we'll see while
using traits we can supply all of these methods on all the classes

Without traits, we'd have something like that:

```scala
class Point(val x: Int, val y: Int)

class Rectangle(val topLeft: Point, val bottomRight: Point) {
  def left = topLeft.x
  def right = bottomRight.x
  def width = right - left

  // many geometric methods here
}

abstract class Component {
  def topLeft: Point
  def bottomRight: Point

  def left = topLeft.x
  def right = bottomRight.x
  def width = right - left

  // many more geometric methods
}
```

- Note the definitions of `left`, `right`, and `width` are the same
- We can eliminate this repetition with enrichment trait, which will
have two abstract methods: one returning the top-left coordinate and
another that returns the bottom-right coordinate, while supplying
concrete implementations of all other geometric queries

```scala
trait Rectangular {
  def topLeft: Point
  def bottomRight: Point

  def left = topLeft.x
  def right = bottomRight.x
  def width = right - left
}

abstract class Component extends Rectangular {}
```

Now, we can query through the trait:

```scala
class Rectangle(val topLeft: Point, val bottomRight: Point)
    extends Rectangular {}

val rect = new Rectangle(new Point(1, 1), new Point(10, 10))
rect.left
rect.right
```

## Examples: the `Ordered` trait

- As we know from discrete mathematics, over any set we can define an
ordering
- In Scala we can compare two objects that are "ordered"
- A rich interface would provide with all of the usual comparison
operators such as is less than, is less than or equal etc, allowing to
directly write things like `x <= y`

Suppose how we would implement an orderings over the set of rational
numbers:

```scala
class Rational(n: Int, d: Int) {
  def < (that: Rational) = this.numer * that.denom < that.numer * this.denom
  def > (that: Rational) = that < this
  def <= (that: Rational) = (this < that) || (this == that)
  def >= (that: Rational) = (this > that) || (this == that)
}
```

- Notice that three of the comparsion operators are defined in terms of the
first one, e.g `>` is defined as the reverse of `<`
- Notice that these methods will be the same for every other class that
is comparable
- All in all, there's alot of boilerplate which would be the same for any
other such class

This issue is so common that Scala provides a built-in trait that
deals with it.

- To use it, you replace all the comparison stuff with a
single `compare` method, the `Ordered` trait itself
defines `<`, `>`, `<=`, `>=` in terms of this one method
- This method should compare the receiver (`this`) and the object
passed as an argument to the method
- It should returns zero if the objects are the same, negative if the
receiver is less than the argument, and positive if the receiver is
greater than the argument

```scala
class Rational(n: Int, d: Int) extends Ordered[Rational] {
  def compare(that: Rational) =
    (this.numer * that.denom) - (that.numer * this.denom)
}

val oneHalf = new Rational(1, 2)
val oneThird = new Rational(1, 3)

half < third // false
half > third // true
```

- The thing in the `[]` is called a type parameter, and is required when
mixing in `Ordered`
- **Note that `Ordered` does not provide `equals`**!

## Traits as stackable modifications

- There's a second major use of traits: providing stackable modifications
- Traits let you modify the methods of a class in a way that lets you stack
these modifications with each other

As an example, suppose you're given a queue of integers, and you want to
stack modifications of it, for example if it has two operations `put`
which enqueues an integer and `get`, which dequeues an integer, we could
define traits to perform modifications such as

- `Doubling` (which doubles every `x` that is put
into the queue)
- `Incrementing` (which peforms x++ for every x that is put into
the queue )
- `Filtering` (which takes every x such that x >= 0 from the queue)

```scala
abstract class IntQueue {
  def get(): Int
  def put(x: Int)
}
```

```scala
class BasicIntQueue extends IntQueue {
  private val buffer = new ArrayBuffer[Int]
  def get() = buffer.remove(0)
  def put(x: Int) = { buffer += x }
}
```

Sample usage of the above implementation is as follows:

```scala
val queue = new BasicIntQueue
queue.put(33)
queue.put(35)

queue.get() // 33
queue.get() // 35
```

Now, let's implement a trait that doubles integers as they are put in
the queue:

```scala
trait Doubling extends IntQueue {
  abstract override def put(x: Int) = { super.put(2 * x) }
}
```

- The way we defined `Doubling` means that it can only be mixed into a class
that also extends `IntQueue`
- It has a `super` call on an abstract method, which is legal for a
trait because `super` calls in traits are dynamically bound and will
work as long as the trait is mixed in after another trait or class that
actually defines the method concretely
- To tell the compiler you're doing this on purpose, you must mark it as
`abstract override` (it's allowed for traits only)

```scala
class CustomQueue extends BasicIntQueue with Doubling

val queue = new CustomQueue
queue.put(10)
queue.get() // no surprise, 20
```

By the way, we could define `CustomQueue` without declaring a new class,
e.g.

```scala
val queue = new BasicIntQueue with Doubling
```

which is good to know.

But we haven't actually seen how to stack modifications, have we?
Let's define more modifications:

```scala
trait Incrementing extends IntQueue {
  abstract override def put(x: Int) = { super.put(x + 1) }
}
```

```scala
trait Filtering extends IntQueue {
  abstract override def put(x: Int) = {
    if (x >= 0) super.put(x)
  }
}
```

Now we can stack:

```scala
val queue = (new BasicIntQueue with Incrementing with Filtering)
queue.put(-1)
queue.put(0)
queue.put(1)

queue.get() // 1
queue.get() // 2
```

- The order of mixins is significant: traits further to the right take
effect first
- In the previous example, `Filtering`'s put is invoked first, so it removes
integers that are negative, `Incrementing`'s put is invoked
afterwards, so it adds one to those integers that remain
- If you reverse the order, e.g. `(new BasicIntQueue with Filtering with
Incrementing)`, first integers will be incremented and then integers
that are still negative will be discarded

## When to use a trait and when not

- Whenever wants to implement reusable behaviour, the question trait vs
abstract class arises

There's no formal rule that says when to use either, but there's
some guidelines:
- If the behaviour will not be reused, then we simply use concrete class
- If it might be reused in multiple, **unrelated** classes, then use a trait
- If you want to inherit from it in Java code, use an abstract class
- If you plan to distribute it in compiled form, and expect outside
clients to write classes inheriting from it, use an abstract class

