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
- **Note that `Ordered` does not provide equals**!
