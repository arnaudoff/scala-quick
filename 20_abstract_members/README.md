# Abstract members

- A member of a class or trait is abstract if it does not have a complete
definition
- Scala, however, goes beyond what's typical in most object-oriented languages:
besides methods, you can declare abstract fields and even abstract types as
members of classes and traits
- In this lecture we'll look at all four kinds of abstract members: `vals`,
`vars`, methods and types

## An example of abstract members

- The following trait declares one of each of the abstract member types, an
abstract type (`T`), an abstract method (`transform`), an abstract
val (`initial`) and abstract var (`current`)

```scala
trait Abstract {
  type T
  def transform(x: T): T
  val initial: T
  var current: T
}
```

- Here's an example implementation that "fills in the gaps"

```scala
class Concrete extends Abstract {
  type T = String
  def transform(x: String) = x + x
  val initial = "foo"
  val current = initial
}
```

## Type members

- As you may noticed from the example, the term *abstract type* in Scala means a
type that is declared with the `type` keyword to be a member of a class or
trait, without a definition
- An *abstract type* in Scala is always a member of some class/trait!
- It's a good idea to make the distinction that classes themselves may be
abstract, and traits by definition are abstract, but neither of these are the
thing we refer to when we say *abstract types* (at least in Scala)
- Any non-abstract type member (such as type `T` in `Concrete`) can be thought
of as an alias for a type; e.g. `String` is given the alias `T` so anywhere
`T` appears in the definition of `Concrete`, it means `String`
- Thus, when `Concrete` implements these methods, those `T`s are interpreted to
mean `String`

Let's look at why these abstract types exists and what they're useful for.

- One reason to use a type member is to define a short alias for a type whose
real name is more verbose
- The other main use of type members is to declare abstract types that must
be defined in subclasses, which we will describe later on

## Abstract values (vals)

- As shown, an abstract `val` declaration has a form like:

```scala
val initial: String
```

- It gives a name and a type for a `val`, but not its value
- This value has to be provided by a concrete `val` definition in a subclass

```scala
val initial = "hi"
```

- You may use the abstract `val` declaration in a class when you don't know the
value in the class, but you're aware that the variable will have an unchangeable
value in each instance of the class

There's also an important distinction to make:

- An abstract val declaration is similar to an abstract parameterless method
declaration:

```scala
val initial: String
def initial: String
```

- Client code will refer to both in exactly the same way (`obj.initial`)
- However, if `initial` was an abstract `val`, the client is guaranteed that
`obj.initial` returns the same value when referenced
- Conversely, if `initial` was an abstract method, that guarantee wouldn't hold
(because, it could be implemented by a concrete method that returns a different
value every time)
- Put in other words, an implementation of abstract `val` must be a `val`
definition and may not be a `var` or a `def`
- An implementation of an abstract method declarations, however, can be both
concrete method definitions and concrete val definitions

For example:

```scala
abstract class Language {
  val v: String
  def m: String
}

abstract class Scala extends Language {
  val v: String
  val m: String // it's ok to override `def` with a `val`
}

abstract class Swift extends Language {
  def v: String // cannot override a `val` with `def`
  def m: String
}
```

## Abstract `var`s

- Like an abstract `val`, an abstract `var` declares simply a name and a type,
but not an initial value, for instance

```scala
trait AbstractTime {
  var hour: Int
  var minute: Int
}
```

- As explained, `var`s declared as members of classes come with getter and
setter methods; same holds for abstract `var`s
- If you declare an abstract `var` named `hour`, you implicitly declare an
abstract getter method `hour` and an abstract setter method `hour_=`
- In other words, the `AbstractTime` is actually equivalent to

```scala
trait AbstractTime {
  def hour: Int
  def hour_=(x: Int)

  def minute: Int
  def minute_=(x: Int)
}
```

### Initializing abstract `val`s

- Abstract `val`s can be used in a similar manner to superclass
parameters: you can provide the details in a subclass that are missing in
a superclass
- The above is especially useful for traits, as traits don't have a
constructor to which you could pass parameters
- To sum up, when you want to parameterize a trait, use abstract `val`s
that are implemented in subclasses

```scala
trait RationalTrait {
  val numerator: Int
  val denominator: Int
}
```

Earlier, we defined such a `Rational` class that had two parameters;
however, the trait above defines instead two abstract vals;
so to instantiate it we needn't to call the constructor, but rather
implement the abstract `val` definitions, which is by doing that:

```scala
new RationalTrait {
  val numerator = 1
  val denominator = 2
}
```

- Here, we "instantiate" the trait; the expression in the curly braces
yields an instance of an *anonymous class* that mixes in the trait and is
defined by the body
- The effect is similar to instantiating our `Rational` class with `new`,
e.g. `new Rational(expression1, expression2)`

There is a subtle difference though:
- When initializing the class, `expression1` and `expression2` are first
evaluated, and then the class initialized with their respective values
- For traits, an implementing `val` definition in a subclass is evaluated
only after the superclass has been initialized

In fact, the above difference has a solution: there exist *pre-initialized
fields* and *lazy vals*, so that we can get abstract `val`s and
parameters to behave as closely as possible.

### Pre-initialized fields

- Pre-initialized fields let you initialize a field of a subclass before the
superclass is called
- To do this, simply place the field definition in braces **before** the
superclass constructor call:

Example (consider that we're using `x` here only to show that it's an
expression and not a literal):

```scala
new {
  val nominator = 1 * x
  val denominator = 2 * x
} with RationalTrait
```

- That solves our problem, however, it's worth mentioning that
pre-initialized fields can be used not only in the case of anonymous
classes, but also for "ordinary" classes and even objects:

```scala
class Rational(n: Int, d: Int) extends {
  val numerator = n
  val denominator = d
} with RationalTrait {
  // some code
}
```

and

```scala
object oneHalf extends {
  val numerator = 1
  val denominator = 3
} with RationalTrait
```

### Lazy vals

- Pre-initialized fields simulate precisely the behaviour of class
constructor arguments when it comes to initialization
- Lazy `val`s are a tool that allow your `val` definitions to have the
property that the initializing expression on the right-hand side will
only be evaluated the first time the `val` is used

```scala
object Foo {
  val bar = { println("...initializing bar"); 33}
}
```

Now, if you refer to `Foo`, the following result will come up:

```
initializing bar
```

If you refer to the member variable, `Foo.bar`, it returns `33`. In other
words, the moment you use `Foo` its `bar` field becomes initialized. Now,
if you use `lazy` `val`s (define `bar` to be a lazy val):

```scala
object Foo {
  lazy val bar = { println("..initializing bar"); 33 }
}
```

when you refer to `Foo`, it doesn't print the initializing message.
However, it is printed as soon as you access `bar`. In other words,
initializing `Foo` does not involve initializing `bar`: the initialization
of `bar` is deferred until the first time `bar` is used.

- The way lazy vals work is similar to a parameterless `def` method
- The difference is that, unlike a `def`, a lazy val is never evaluated more
than once: after the first evaluation of a lazy `val` the result of the
evaluation is stored and reused when the same `val` is used later on
- Actually, objects like `Foo` themselves act like lazy `val`s in that
they are also initialized on demand the first time they're used; put
simply, an object definition can be seen as a shorthand for the definition
of a lazy `val` with an anonymous class that describes the object's contents

Now, let's go back to the original problem we were solving by defining
what lazy vals are. This is how our RationalTrait could be redesigned
(notice that all initialization code is now part of the right-hand side of
a lazy `val`, it is now safe to initialize the abstract fields
of `LazyRationalTrait`):

```scala
trait LazyRationalTrait {
  val numeratorArg: Int
  val denominatorArg: Int

  lazy val numerator = numeratorArg / g
  lazy val denominator = denominatorArg / g

  override def toString = numerator + "/" + denominator

  private lazy val g = {
    require(denominatorArg != 0)
    gcd(numeratorArg, denominatorArg)
  }

  private def gcd(a: Int, b: Int): Int = // ...
}
```

Let's try to use the same construction that broke our initialization
earlier:

```scala
val x = 2

new LazyRationalTrait {
  val numeratorArg = 1 * x
  val denominatorArg = 2 * x
}

// initializes safely to 1/2
```

As an exercise, here's all of the initialization steps performed in the
above piece of code:
- An instance of `LazyRationalTrait` is created, the initialization code
of it is executed; however, it's empty as none of the fields of
`LazyRationalTrait` is initialized yet
- The primary constructor of the anonymous subclass defined by the `new`
expression is executed, which initializes `numeratorArg` with `2` and
`denominatorArg` with `4`
- Next, the `toString` is invoked on the constructed object
- Next, the `numerator` field is accessed for the first time
by `toString`, so its initializer is evaluated
- The initializer of `numerator` accesses the private field, `g` so `g` is
    evaluated next
- Next, the `toString` method accesses the value of `denominator`, which
causes its evaluation; the evaluation of `denominator` accesses
`denominatorArg` and `g`, but the initializer of the `g` field is not
re-evaluated, because it was already evaluated

The textual order of lazy vals definitions doesn't matter because
values get initialized on demand, so you don't have to think how to
arrange `val` definitions to ensure that everything is defined when it is
needed (of course that's when no side effects are present, in general lazy
`val`s go well with functional code)
