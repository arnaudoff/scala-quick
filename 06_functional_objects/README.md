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

## Self references

- Like in other languages, `this` refers to the object instance on which the
currently executing method was invoked, or if within a constructor, the
object instance being constructed

Example (a method that checks whether the given rational number is smaller than
a parameter):

```scala
def lessThan(that: Rational) =
  this.numerator * that.denominator < that.numerator * this.denominator
```

- You can omit `this` similarly to other languages, the two notations are
equivalent

A more practical example of why `this` exists as it is this situation:

```scala
def max(that: Rational) =
  if (this.lessThan(that)) that else this
```

- Although the first `this` is redundant, the second `this` represents the
result of the method, hence omitting in would result in a method that has
nothing to return

## Auxiliary constructor

- Often times you need more than just the primary constructor
- These additional constructors are called *auxiliary constructors*
- Using our case study, a rational number with denominator of `1` needn't
require explicit specification, e.g instead of `new Rational(5, 1)` the
client could do `new Rational(5)` with the same success
- Typically, the body of the auxiliary constructor invokes the primary
    constructor, passing along some "predefined" argument (like `1`)

```scala
class Rational(p: Int, q: Int) {

  require(q != 0)

  val numerator: Int = p
  val denominator: Int = q

  def this(p: Int) = this(p, 1) // auxiliary constructor

  override def toString = numerator + "/" + denominator

  def add(that: Rational): Rational =
    new Rational(
      numerator * that.denominator + that.numerator * denominator,
      denominator * that.denominator
    )
}
```

- Every auxiliary constructor **must** invoke another constructor of the same
class as its first action
- Following the above rule, every constructor invocation in Scala will end up
eventually calling the primary constructor
- Note that only the primary constructor can invoke a superclass constructor

## Private fields and methods

- As you probably noticed, Rational currently doesn't support normalization of
the fractions, e.g. `66/42` could be normalized to `11/7` by doing a division by
the GCD of the numerator and denominator

Example implementation:

```scala
class Rational(p: Int, q: Int) {
  require (q != 0)

  private val g = gcd(p.abs, q.abs)

  val numerator = p / g
  val denominator = q / g

  def this(p: Int) = this(p, 1)

  def add(that: Rational): Rational =
    new Rational(
      numerator * that.denominator + that.numerator * denominator,
      denominator * that.denominator
    )

  override def toString = numerator + "/" + denominator

  private def gcd(a: Int, b: Int): Int =
    if (b == 0) a else gcd(b, a % b)
}
```

- Note that `g` is a private field which calls `gcd`, a classical recursive
implementation of Euclid's algorithm for finding the GCD of two numbers
- `g` cannot be accessed outside the body of the class
- Invoking the `abs` method on `p` and `q` ensures `g` is always positive
(`abs` is defined on any `Int`)

Now,

```scala
new Rational(66/42)
```

yields `Rational = 11/7`.

## Defining operators

- `Rational`'s implementation could be improved by making the mathematical
operations with its instances more intuitive, e.g. `x + y` instead of `x.add(y)`
- In addition to improving it, we'll also support multiplication

```scala
class Rational(p: Int, q: Int) {

  require(q != 0)

  private val g = gcd(p.abs, q.abs)

  val numerator = p / g
  val denominator = q / g

  def this(p: Int) = this(p, 1)

  def +(that: Rational): Rational =
    new Rational(
        numerator * that.denominator + that.numerator * denominator,
        denominator * that.denominator
    )

  def *(that: Rational): Rational =
    new Rational(numerator * that.numerator, denominator * that.denominator)

  override def toString = numerator + "/" + denominator

  private def gcd(a: Int, b: Int): Int =
    if (b == 0) a else gcd(b, a % b)
}
```

This allows us to do the following:

```scala
val oneHalf = new Rational(1, 2)
val twoThirds = new Rational(2, 3)
val additionResult = oneHalf + twoThirds // yields 7/6
val multiplicationResult = oneHalf * twoThirds // yields 2/6
```

## Identifiers in Scala and some convetions

- We've already seen the most fundamental ways to form an identifier:
alphanumeric (starts with a letter or underscore, followed by
digits/letters/underscores) and operator

### Alphanumeric identifiers

- `$` is reserved for identifiers generated by the Scala compiler, so avoid it
to prevent name clashes
- Scala typically uses camel-case identifiers, e.g. `toString` or `HashSet`
- It's good style to name fields, method params, local vars and functions with
camel-case and start with a lower case letter, e.g. `length`, `flatMap` and `s`
- Camel-case names of classes and traits - with an upper case letter, e.g.
`BigInt`, `List`, `UnbalancedTreeMap`
- In Scala, constant != `val`. `val` is still a variable, i.e when invoking
methods with `val` params, you can supply different arguments
- A constant is more permanent than a `val`, i.e `scala.math.Pi` is a good
example
- It's vital to note that in Scala the convention differs from Java and
constants are not named all upper case (`MAX_VALUE`), but only that the
first letter should be upper case (`Pi`, `XOffset`)
