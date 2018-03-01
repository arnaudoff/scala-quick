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
what lazy vals are. This is how our `RationalTrait` could be redesigned
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

## Abstract `type`s

- In the beginning, we saw `type T` as an abstract type declaration
- Like all other abstract declarations, the idea is that the type will be
defined concretely in subclasses
- In this case, it's just a type that will be defined further down in the
class hierarchy
- Put simply, when you write `type T` you refer to at type `T` that is yet
unknown at the point where it is declared, and different subclasses can
provide different implementations of `T`

A good example of where abstract types are applicable is the following
hierarchy:

```scala
class Food

abstract class Animal {
  def eat(food: Food)
}
```

```
class Grass extends Food

class Cow extends Animal {
  override def eat(food: Grass) = {}
}
```

However, this won't compile, because the `eat` method in class `Cow` did not
override the `eat` method in `Animal` (the parameter types are
different). Although one may think that the type system should cover this,
however, it leads to unsafe situations, such as:

```scala
class Fish extends Food

val someCow: Animal = new Cow
someCow.eat(new Fish) // semantically wrong..

```

Here's where abstract types come in handy. Since `Animals` do eat `Food`,
but what kind of `Food` each `Animal` eats depends on the `Animal`, we can
express this nicely with abstract types:

```scala
class Food

abstract class Animal {
  type SuitableFood <: Food

  def eat(food: SuitableFood)
}
```

In order words, an `Animal` can only eat food that is suitable; however, we
cannot determine what food is suitable at that level of the hierarchy,
because we need to know the type of the animal beforehand - a perfect fit
for abstract types, huh? `SuitableFood` is modeled as an abstract type.
- The type has an "upper bound" `Food` (expressed by the `<: Food` clause)
- Having an upper bound means that any concrete instantiation of
`SuitableFood` must be a subclass of `Food`

```scala
class Grass extends Food

class Cow extends Animal {
  type SuitableFood = Grass
  override def eat(food: Grass) = {}
}
```

## Path-dependent types

- In Scala, objects can have types as members: for instance, in the
above example, if we had a cow instance `foo`, `foo.SuitableFood` would
mean the type `SuitableFood` that is a member of the object referenced
from `foo` or, put in other words, the type of food that's suitable
for `foo`
- A type like `foo.SuitableFood` is called a *path-dependent type*, where
path here means a reference to an object: it could be a single name or a
longer access path, e.g `farm.cows.foo`
- Obviously, different paths give rise to different types, for example:

```scala
class DogFood extends Food

class Dog extends Animal {
  type SuitableFood = DogFood
  override def eat(food: DogFood) = {}
}
```

so

```scala
val foo = new Cow
val bar = new Dog
bar.eat(new foo.SuitableFood) // fails, can't feed cow food to a dog
```

fails as explained.

- Obviously, if you have two objects of type Dog, you can feed one of them
with the `SuitableFood` of the other and vice versa

A path-dependent type is similar to the syntax for an inner class type in
Java, but there's a fundamental difference: a path-dependent type names an
outer **object**, whereas an inner class type names an outer **class**.
Java-style inner class types can also be expressed in Scala, but they are
written differently. Consider:

```scala
class Outer {
  class Inner
}
```

- In Scala, the inner class is addressed using `Outer#Inner` instead of
Java's `Outer.Inner`; the `.` syntax is reserved for objects

```scala
val object1 = new Outer
val object2 = new Outer

object1.Inner // path-dependent type
object2.Inner // path-dependent type
```

- In the above example, `object1.Inner` and `object2.Inner` are two
path-dependent types which are subtypes of `Outer#Inner`
- In Scala just like in Java, inner class instances hold a reference to an
enclosing outer class instance, which allows an inner class to access
members of its outer class; put in other words, an inner class cannot be
instantiated without in some way specifying an outer class instance

## Refinement types

- When a class inherits from another, we call the first class a
*nominal subtype* of the other one
- It's a nominal subtype because each type has a name and the names are
explicitly declared to have a subtyping relationship
- There's another subtyping relationship called *structural subtyping*,
where you get subtyping relationship simply because two types have
compatible members; this is achieved using *refinement* types

### Nominal vs structural subtyping

- Nominal subtyping is usually more convinient, so it should be tried first
- Basically, a name is a single short identifier and is therefore better
than explicitly listing member types
- Also, structural subtyping is sometimes more flexible than required;
for example a `Widget` can `draw()`, and also a painter can `draw()`, but
they aren't really substitutable: you'd prefer to get a type error if
you one for the other

However, structural subtyping has some advantages:
- Sometimes, there's really no more to a type than its members; consider a
`Pasture` class that contains animals that eat grass
- One option for the above is to have a trait called
e.g `AnimalThatEatsGrass` and mix it into every class where it applies;
- Such solution would be too verbose (for instance, class `Cow` already has
defined that it's an animal and that it eats grass, now it'd have to
declare that once more)
- Scala has a better alternative: refinement types

```scala
Animal { type SuitableFood = Grass } // animal that eats grass
```

- The above defines a refinement type: you write the base type, followed
by a sequence of members listed in curly braces
- The members in the curly braces further specify (refine) the types of
members from the base class

With that tool at hand, we can write the `Pasture` class:

```scala
class Pasture {
  var animals: List[Animal { type SuitableFood = Grass}] = Nil
}
```

## Enumerations

- Unlike other languages, Scala doesn't have enumerations as a built-in
language construct to define new types
- Instead, there's a class called `scala.Enumeration` in its standard
library
- To define a new enumeration, you define an object that extends this class

```scala
object Color extends Enumeration {
  val Red = Value
  val Green = Value
  val Blue = Value
}
```

or equivalently

```scala
object Color extends Enumeration {
  val Red, Green, Blue = Value
}
```

- `Enumeration` defines an inner class named `Value`, and a parameterless
`Value` method (with the same name), which returns an instance of that class
- Put simply, a value such as `Color.Red` is of type `Color.Value`
where `Color.Value` is the type of all enumeration values defined in
object `Color`
- `Color.Value` is actually a path-dependent type, with `Color` being
the path and `Value` being the dependent type
- It's vital to note that it's a completely new type, different from all
other types, so if you define another enumeration:

```scala
object Direction extends Enumeration {
  val North, East, South, West = Value
}
```

then `Direction.Value` is different from `Color.Value`, because the path
parts of the two types differ

- You can also associate names with enumeration values (like in some
other languages) by using a different overloaded variant of the `Value`
method:

```scala
object Color extends Enumeration {
  val Red = Value("Red")
  val Green = Value("Green")
  val Blue = Value("Blue")
}
```

Also, two more things: 
- You can iterate over the enumeration values using the `values` method,
define on `Enumeration`:

```scala
for (c <- Color.values)
  println(c + " ")
```

## An example of abstract types: currencies

- The idea is to design a class `Currency`, where an instance of `Currency`
would represent an amount of money in dollars, euros, or some other
currency
- It should also be possible to do some arithmetic on
currencies; e.g to add two amounts of the same currency or multiply a
currency amount by a factor

```scala
abstract class Currency {
  val amount: Long
  def designation: String
  override def toString = amount + " " + designation
  def + (that: Currency): Currency = // ??
  def * (x: Double): Currency = // ??
}
```

Few things to note:
- `amount` of a currency is the number of currency units it represents
- `amount` is also left abstract waiting to be defined when a subclass
"talks" about concrete amounts of money
- `designation` is a string that identifies it

Concrete currency value could be created like that:

```scala
new Currency {
  val amount = 33L
  def designation = "USD"
}
```

The above design, however, is flawed. It would only be fine if it modelled a
single currency, e.g. only dollars or euros. Consider that you want to model
dollars and euros:

```scala
abstract class Dollar extends Currency {
  def designation = "USD"
}

abstract class Euro extends Currency {
  def designation = "Euro"
}
```

however, this lets you add dollars to euros which, obviously, is no good.

What we need instead is a more "specialized" version of the `+` method: when
implemented in class `Dollar`, it should take `Dollar` arguments and yield a
`Dollar` result; when implemented in `Euro` it should take `Euro` arguments
and yield `Euro` result. In other words, the type of the addition method
would change depending on which class you are in, but the method should be
written only once, not each time a new currency is defined. This is a
perfect fit for abstract types.

```scala
abstract class AbstractCurrency {
  type Currency <: AbstractCurrency // represents the currency "in question"
  val amount: Long
  def designation: String
  override def toString = amount + " " + designation
  def + (that: Currency): Currency = // ..
  def * (x: Double): Currency = // ..
}
```

Now, for example `Dollar` should "fix" the `Currency` type to refer to the
concrete class itself:

```scala
abstract class Dollar extends AbstractCurrency {
  type Currency = Dollar
  def designation = "USD"
}
```

This design is definitely an improvement, but we can do better. Note that
we've purposely left out the definitions of the `*` and `+` methods in
`AbstractCurrency`. How'd you implement them? Clearly you have to do
`this.amount + that.amount`, but how'd you convert the amount into a
currency of the right type?

- One could try to solve the above problem like that:

```scala
def + (that: Currency): Currency = new Currency {
  val amount = this.amount + that.amount
}
```

- Although a good attempt, this would not compile
- The reason is that you cannot create an instance of an abstract type
- In the example, you cannot instantiate `Currency`

The problem above can be solved using a factory method: instead of creating
an instance of an abstract type directly, we can do it with an abstract
method that does it; then, whenever you fix an abstract type to be some
concrete type, you can subsequently give a concrete implementation of
the factory method

```scala
abstract class AbstractCurrency {
  type Currency <: AbstractCurrency
  def make(amount: Long): Currency
}
```

For the experienced reader, this might ring a bell. If you have some
amount of currency, you also have the ability to make more of the same
currency, using a construction such as

```scala
oneEuro.make(100) // a hundred more euros
```

The solution here is to move the abstract type and the factory method
outside of `AbstractCurrency`.  We'll create a `CurrencyZone`, which
semantically resembles a region that has the same currency, which will
contains the `AbstractCurrency` class, the `Currency` type and the
`make` factory method:

```scala
abstract class CurrencyZone {
  type Currency <: AbstractCurrency
  def make(x: Long): Currency
  abstract class AbstractCurrency {
    val amount: Long
    def designation: String
    override def toString = amount + " " + designation
    def + (that: Currency): Currency =
      make(this.amount + that.amount)
    def * (x: Double): Currency =
      make((this.amount * x))
  }
}
```

An example usage is as follows:

```scala
object UnitedStates extends CurrencyZone {
  abstract class Dollar extends AbstractCurrency {
    def designation = "USD"
  }

  type Currency = Dollar

  def make(x: Long) = new Dollar {
    val amount = x
  }
}
```

where `UnitedStates` is some currency zone that uses `Dollar`s; so the
type of money in this zone is `UnitedStates.Dollar` (where the
`UnitedStates` object also fixes the type of `Currency`) and gives an
implementation of the `make` factory method to return a dollar amount.

- Also, another limitation to mention of abstract types is that you cannot
inherit from an abstract type

