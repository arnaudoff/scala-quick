# Functional objects

In this lecture, we will be designing a class in functional style which
models rational numbers. We'll also show more features related to OOP in Scala
such as class parameters, constructors, methods and operators, overriding,
preconditions, overloading and self references.

## Notes on the design of the `Rational` class

Quick math revision:

- A rational number is a number that can be expressed as a ratio `p/q`,
where `p` and `q` are integers for `q` != 0
- `p` is called the numerator, and `q` is called the denominator
- Some examples are `1/2`, `2/3`, `112/239` and `2/1`
- With rational numbers, fractions are represented exactly, without rounding
- Intuitively, in maths, rational numbers are immutable by design, e.g. adding
one rational number to another results in a new rational number. (the
original numbers are unchanged)

The class we will model should support rational numbers to be:

- Added (using a common denominator: e.g. to add `1/2` and `2/3`, both parts
of the left operand are multiplied by `3` and both parts of the right operand
by `2`, which gives `3/6` + `4/6`, or `7/6`)
- Substracted
- Multiplied (multiply their numerators and denominators: e.g. `1/2` * `2/5`
gives `2/10`, or `1/5`)
- Divided (by swapping the numerator and denominator of the right operand: e.g.
`1/2` / `3/5` is equivalent to `1/2` * `5/3`, or `5/6`)

## A note on immutable objects

- Immutable objects are often easier to reason about: they don't have complex
state that changes over time
- You can pass immutable objects around freely, while mutable objects require
defensive copies before passing them
- There's no way for two threads that concurrently access an immutable object
to corrupt its state once it's been properly constructed, because no thread can
change it
- Immutable objects make safe hash table keys, while mutable objects can be
mutated after they are placed into a hash table, meaning they may not be
found the next time he hash table is accessed
- Immutable objects, however, have a disadvantage: they sometimes require that a
large object graph is copied, where an update could be done in place, which
might cause a performance bottleneck
- Often libraries deal with the latter issue by providing mutable alternatives
to immutable classes, e.g. `StringBuilder` is a mutable alternative to the
immutable `String`


## Constructing a `Rational`

- Given we've decided to model `Rational` so that its object are immutable,
we'll require that clients provide all data needed by the object (a
numerator and a denominator) upon instance construction:

```scala
class Rational(p: Int, q: Int)
```
- Unlike in Java, where classes have constructors, which take parameters,
in Scala classes themselves can take parameters directly: `p` and `q` are
called *class parameters*
- Class parameters can be used in the body of the class; there's no need for
boilerplate code that copies them into fields
- The Scala compiler gathers the class parameters and builds a *primary
constructor* that takes the same two parameters
- In fact, anything in the class body that isn't part of a field or method definition
is compiled into the primary constructor

Example:

```scala
class Rational(p: Int, q: Int) {
  println("Created " + p + "/" + q) // this is placed into the primary constructor
}
```

Considering this, `new Rational(3/5)` will yield `Created 3/5` in the console.

## Reimplementing `toString`

- Just as in Java or other OO languages, we sometimes need to display a
human-friendly representation of our object state, which is typically done by
`toString` method that converts the state into a meaningful string
- By default, our class inherits the implementation of `toString` defined in
`java.lang.Object`, which just prints the class name, an `@` sign, and a
hexadecimal number, which is not helpful
- A useful implementation of `toString` would yield the values of the numerator
and denominator, and we will *override* the default implementation:

```scala
class Rational(p: Int, q: Int) {
  override def toString = p + "/" + q
}
```

Example: in the Scala interpreter, try `val foo = new Rational(1/2)`

## Checking preconditions

- Now, you might've noticed that our class currently allows a denominator of
zero, which is invalid in mathematical sense
- In the case of an immutable object such as `Rational`, you must ensure that
the state is valid when the object is constructed
- In typical languages, this is solved using setter methods, but in Scala you
use *precondition* in the primary constructor

```scala
class Rational(p: Int, q: Int) {
  require(q != 0)
  override def toString = p + "/" + q
}
```

- `require` takes a boolean parameter, if the passed value is true, require will
return normally, otherwise will prevent the object to be constructed by
throwing an `IllegalArgumentException`

## Adding fields

- Now that the class is enforcing its precondition properly, we will turn to
supporting addition
- In order to keep `Rational` immutable, the method implementing addition **must
not** add the passed rational number to itself, but create and return a new
`Rational` that holds the sum

A naive implementation would be:

```scala
class Rational(p: Int, q: Int) {
  require(q != 0)

  override def toString = p + "/" + q

  def add(that: Rational): Rational =
    new Rational(p * that.q + q * that.p, q * that.q)
}
```

- However, the above code won't compile: although `p` and `q`, as class
parameters, are in scope in the code of `add`, **their value can be accessed
only on the object for which `add` was invoked**
- While using `p` and `q` in `add`'s implementation yields the values for
these class parameters, the compiler doesn't allow `that.p` or
`that.q`, because `that` does not refer to the `Rational` object on which
`add` was invoked
- To access the numerator and denominator on `that`, `p` and `q` need to be
fields
- We will add two fields: `numerator` and `denominator`, initialized with
the values of the class parameters `p` and `q`

```scala
class Rational(p: Int, q: Int) {
  require(q != 0)

  val numerator: Int = p
  val denominator: Int = q

  override def toString = numerator + "/" + denominator

  def add(that: Rational): Rational =
    new Rational(
        numerator * that.denominator + denominator * that.numerator,
        denominator * that.denominator
    )
}
```

This compiles fine. An example usage would be:

```scala
val oneHalf = new Rational(1, 2)
val twoThirds = new Rational(2, 3)
oneHalf add twoThirds // yields 7/6

// You can now access the numerator and denominator values outside the object
val foo = new Rational(3/4)
foo.numerator
foo.denominator
```
