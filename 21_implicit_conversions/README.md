# Implicit conversions and parameters

- A typical problem in software development is extending an already existing
    third party library
- Scala solves it by using implicit conversions and parameters
- This lecture shows how implicits work and presents some common use cases

## Implicit conversions

- To begin with, a typical example of use of implicit conversions is when
working with two separate modules of software that were developed without
each other in mind
- Each module has its own way to encode a concept that's essentially the
same thing
- Implicit conversions help by reducing the explicit conversions required
from one type to another

We'll use the Swing library as a example: it's a typical cross-platform
library that implements user interfaces and so one of the things it does
it process events from the OS, convert them to platform-independent event
objects, and pass them to event listeners.

If Swing was written in Scala, event listeners would be implemented as a
function type so callers could then use the function literal syntax as
a way to specify what should happen for a certain class of events;
however, in Java, since function literals don't exist, Swing uses inner
classes that implement a one-method interface, e.g. for action listeners
it's an `ActionListener`.

The big thing is that, without the use of implicit conversions,
a Scala program that uses Swing must use Java's way to achieve interaction
(inner classes):

```scala
val button = new JButton
button.addActionListener(new ActionListener {
  def actionPerformed(event: ActionEvent) = {
    println("pressed")
  }
})
```

However, this has a lot of boilerplate: the fact that this listener is an
`ActionListener`, the fact that the callback method is `actionPerformed`,
and the fact that the argument is an `ActionEvent` are all implied for any
argument to `addActionListener`: the only new information is the call to
`println`.

- The Scala way of doing the above would be to take a function as an
argument such as this:

```scala
button.addActionListener(
  (_: ActionEvent) => println("pressed")
)
```

- Obviously, this won't work: as explained, the `addActionListener` wants an
    action listener but is getting a function

Here's where implicit conversions come in handy. We can "tie" this code
using them and make it compile.  Let's write an implicit conversion
between the two types, in particular from functions to action listeners:

```scala
implicit def function2ActionListener(f: ActionEvent => Unit) =
  new ActionListener {
    def actionPerformed(event: ActionEvent) = f(event)
  }
```

so our code simplifies to:

```scala
button.addActionListener(
    function2ActionListener(
        (_: ActionEvent) => println("pressed")
    )
)
```

Now, the fun part: since `function2ActionListener` is marked as implicit,
it can be left out, so the result is:

```scala
button.addActionListener(
  (_: ActionEvent) => println("pressed")
)
```

- This works because when the compiler first tries to compile as it is,
it sees a type error, but before giving up checks if there's an implicit
conversion defined that solves the problem, finds
`function2ActionListener` and uses it

Powerful, huh?

## Rules for implicits

Implicit definitions, as shown, are the ones that the compiler is
allowed to insert to fix any of its type errors. They also have some
general rules:

### Marking rule: only definitions marked `implicit` are available

You can use the `implicit` keyword to mark any variable, function or object
definition, for example:

```scala
implicit def intToString(x: Int) = x.toString
```

This way, you simply avoid the situation where the compiler picks random
functions that happen to be in scope and insert them as conversions.

### Scope rule: an inserted implicit conversion must be in scope as a single identifier or be associated with the source/target type of the conversion

The compiler will only consider implicit conversion that are in scope, so
must in some way bring it into scope and it has to be in scope as a single
identifier. For instance, the compiler will not expand

```scala
x + y
```

to

```scala
foo.convert(x) + y`
```

because, if you want to make foo.convert(x) available as an implicit,
you would need to import it and make it a single identifier

Often times, you will see libraries offering an object (sometimes called
`Preamble`) that includes useful implicit conversions; thus, client code
that uses the library can then do a single `import Preamble._` to access
the library-defined implicit conversions.

**Exception**: the compiler also looks for implicit definitions in the
companion object of the source or expected target types of the conversion

Example for the exception: say you're attempting to pass `Dollar`
object to a method that takes a `Euro`; here the source type is `Dollar`,
the target type is `Euro` so by the exception, you could have an implicit
conversion from `Dollar` to `Euro` in the companion object of either class:

```scala
object Dollar {
  implicit def dollarToEuro(x: Dollar): Euro = // ..
}

class Dollar {}
```

- We say that the conversion of dollar to euro is *associated* to the
`Dollar` type
- There's no need to import the conversion separately, the associated
conversion goes with with the `Dollar` definition
- The scope rule is useful when we reason about our program in modules;
e.g when you read a file, the only things you need to consider from other
files are those that are either imported or are explicitly referenced
- If implicits were available system-wide, then it'd be a pain to
understand a file since you'd have to know about every implicit introduced
in the system

### One-at-a-time rule: only one implicit is inserted

The compiler never rewrites

```scala
x + y
```

to

```scala
convert1(convert2(x)) + y
```

because compile times increase dramatically on erroneous code; so the
compiler does not insert further implicit conversions when it's already in
the middle of trying an implicit.

### Explicits-first rule: whenever code type checks as it's written, no implicits are attempted

In other words, the compiler will not change code that already works; as an
implication of the rule comes the fact that you can always replace implicit
identifiers with explicit ones and you can trade between these choices on a
case-by-case basis:
- Code that is repetitive and verbose can be improved with implicit
conversion
- On the contrary, code that is terse to the point of obscurity, can have
conversions inserted explicitly

## Naming an implicit conversion

- Implicit conversions can have arbitrary names
- The name of an implicit conversion matters only if you want to write it
explicitly or, more often, for determining which implicit conversions
are available at any place in the program:

```scala
object Conversions {
  implicit def stringWrapper(s: String): IndexedSeq[Char] = // ..
  implicit def intToString(n: Int): String = // ..
}
```

So, if you want to make use of the `stringWrapper`, but you don't want
integers to be converted to automatically to strings, you can achieve
this by importing only one of the conversions:

```scala
import Conversions.stringWrapper
```

Since implicit conversions had names, we could explicitly state which one to
import and which one not to.

## Where implicit conversions are tried

There are three placs implicits are used in the language:
- Conversions to an expected type: let you use use one type in a context
where a different type is expected, as shown already
- Conversions of the receiver of a selection: they allow you to adapt the
receiver of a method call, if the method is not applicable on the original
type, such as `"abc".exists`, which is converted to
`stringWrapper("abc").exists` since `exists` is defined on indexed
sequences but not on strings
- Implicit parameters: used to provide more information to the called
function about what the caller wants; especially useful with generic
functions where the called function might otherwise know nothing at
all about the type of an argument

## Implicit conversions to an expected type

- The rule is what we already mentioned: whenever the compiler sees
an `X`, but needs a `Y`, it'll look for an implicit function that
converts `X` to `Y`

Example:

```scala
val foo: Int = 3.3 // fails

implicit def doubleToInt(x: Double) = x.toInt

val foo: Int = 3.3 // equivalent to val foo: Int = doubleToInt(3.3)

// foo is now 3
```

- Note that `doubleToInt` is in scope as a single identifier

## Converting the receiver

- Implicit conversion also take effect to the receiver of a method call
- Such implicit conversion has two main uses: the receiver conversions
allow smoother integrations of a new class into an existing hierarchy and
second, they support writing DSLs within the language itself

In other words, say you have an object `obj` and a method `foo`,
but `obj` does not have `foo` defined as a member. The compiler will try
to insert conversions before giving up, and in this case, the conversion
needs to apply to the receiver (`obj`). In other words, the compiler will
act as if the expected type of `obj` was "has a member `foo`".

### Smoother integration of new types

- As mentioned, one major use of receiver conversions is allowing smoother
    integration of new types with existing types

Consider the `Rational` class we were building earlier:

```scala
class Rational(n: Int, d: Int) {
  def + (that: Rational): Rational
  def + (that: Int): Rational
}
```

- You can obviously add two rational numbers and a rational number and
an `Int`

```scala
val oneHalf = new Rational(1, 2)
oneHalf + oneHalf // 1/1
oneHalf + 1 // 3/2
```

- But what about `1 + oneHalf`? Obviously the receiver has no such method
and it doesn't compile
- To allow this type of arithmetic, you need to define an implicit
conversion from `Int` to `Rational`

```scala
implicit def intToRational(x: Int) = new Rational(x, 1)
```

Here, the compiler type checks `1 + oneHalf` as it is, it fails
because `Int` has no `+` method that takes a `Rational` argument, then
it searches for an implicit conversion from `Int` to another type that
has a `+` method, finds our implicit definition and applies it which
yields `intToRational(1) + oneHalf`.

### Simulating new syntax

- Another major use of implicit conversions is to "simulate" adding new
syntax

Scala supports maps using a syntax like this:

```scala
Map(1 -> "one", 2 -> "two", 3 -> "three")
```

Actually, the `->` here is not syntax. `->` is a method on the
class `ArrowAssoc`, a class that is defined in the standard Scala
preamble (`scala.Predef`), which also defines an implicit
conversion from `Any` to `ArrowAssoc`. When you write `1 -> "one"`, the
compiler inserts a conversion from `1` to `ArrowAssoc` so that the `->`
method can be found.

The definition looks something like that:

```scala
object Predef {
  class ArrowAssoc[A](x: A) {
    def ->[B](y: B): Tuple2[A, B] = Tuple2(x, y)
  }

  implicit def any2ArrowAssoc[A](x: A): ArrowAssoc[A] = new ArrowAssoc(x)
}
```

- This is the "rich wrappers" pattern, which is common in libraries
that provide syntax-like extensions to the language, so it should be
easily recognized
- As a rule, whenever you see someone calling methods that appear not
to exist in the receiver class, they are problably using implicits
- As a rule, if you see a class named `Rich<x>`, that class is likely adding
syntax-like methods to the type `x`
- These rich wrappers often allow you to even define an internal DSL as a
    library, where in other languages you'll need to define external DSL.

### Implicit classes

- An implicit class is a class that is preceded by the `implicit` keyword
- For any such class, the compiler generates an implicit conversion from the
class's constructor parameter to the class itself
- Such a conversion is required if you plan to use the class for the rich
wrappers pattern

Example:

```scala
case class Rectangle(width: Int, height: Int)
```

If you use `Rectangle` very frequenly, you might want to use the rich
wrappers pattern so you can more easily construct it:

```scala
implicit class RectangleMaker(width: Int) {
  def x(height: Int) = Rectangle(width, height)
}
```

Written that way, the below is automatically generated:

```scala
implicit def RectangleMaker(width: Int) = new RectangleMaker(width)
```

So, you can create rectangles by putting an `x` between two integers:

```scala
val rect = 3 x 4
```

Since type `Int` has no method called `x`, the compiler looks for an
implicit conversion from `Int` to something that has a method `x`; it will
find the automatically generated `RectangleMaker` conversion
(`RectangleMaker` has a `x` method) and insert the call to the conversion
respectively.

**Note**: An implicit class must be defined inside of another `trait`,
`class` or `object` and may only take one non-implicit argument in their
constructor.

## Implicit parameters

- The last place where the compiler inserts implicits is withing argument
list
- The compiler sometimes replaces `foo(x)` with `foo(x)(y)` or
`new Foo(x)` with `new Foo(x)` with `new Foo(x)(y)`, thereby adding a
missing parameter list to complete a function call
- Also, the entire last curried parameter list is supplied,
not just the last parameter

Example: if `foo`'s missing last parameters list takes three parameters,
the compiler might replace `foo(t)` with `foo(t)(x, y, z)`

In other for this to work, inserted identifiers such as `x`, `y` and `z`
should be marked `implicit` where they're defined and the last
parameter list in `foo` or `Foo` must be marked `implicit` as well.

**Practical example**: Say you're writing some shell interface and you
have a class `PreferredPrompt`, which encapsulates a shell prompt string
such as `$`, `>` etc.

```scala
class PreferredPrompt(val preference: String)
```

Also, say you have a `Greeter` object with a `greet` method which takes two
parameter lists: the first takes a user name and the second takes a
`PreferredPrompt`:

```scala
object Greeter {
  def greet(name: String)(implicit prompt: PreferredPrompt) = {
    println("Welcome, " + name)
    println(prompt.preference)
  }
}
```

- As you probably noticed, the last parameter list is marked `implicit`,
which means it can be supplied implicitly
- For that purpose, you must first define a variable of the expected type
- Say the variable is `prompt`. It should also be marked `implicit`
- If it wasn't, the compiler would not use it to supply the missing
parameter list:

```scala
object JohnsPreferences {
  implicit val prompt = new PreferredPrompt("> $")
}
```

**Note**: You should never forget to actually bring the `implicit` val
into scope with an `import`, because if it's missing from the scope as a
single identifier, it won't be used to fill in the missing parameter list:

```scala
import JohnsPreferences._
```

However, you can still provide the prompt explicitly:

```scala
val scalasPrompt = new PreferredPrompt("scala> ")
Greeter.greet("John")(scalasPrompt)

// Welcome, John
// scala>
```

- Also, note that the `implicit` keyword applies to an entire parameter
list, not to individual parameters, so we could do the following as well:

```scala
class PreferredPrompt(val preference: String)
class PreferredTimeFormat(val preference: String)

object Greeter {
  def greet(name: String)(implicit prompt: PreferredPrompt,
      timeFormat: PreferredTimeFormat) = {
    println(Foo.showTime(timeFormat.preference) + " " + "Welcome, " + name)
    println(Foo.showTime(timeFormat.preference) + " " + prompt.preference)
  }
}

object JohnsPreferences {
  implicit val prompt = new PreferredPrompt("> $")
  implicit val timeFormat = new PreferredTimeFormat("hh:mm:ss")
}

import JohnsPreferences._ // bring the vals into scope!

Greeter.greet("John")
```

- Another thing to have in mind is that implicit parameters are often used
to provide information about a type mentioned explicitly in earlier
parameter list

For instance, consider a function which returns the maximum
element of a list:

```scala
def maxListOrdering[T](elements: List[T])(ordering: Ordering[T]): T =
  elements match {
    case List() => throw new IllegalArgumentException()
    case List(x) => x
    case x :: rest =>
      val maxRest = maxListOrdering(rest)(ordering)
      if (ordering.gt(x, maxRest)) x
      else maxRest
  }
```

The additional argument `ordering` specifies which ordering to use
comparing elements of type `T`, as such, this version can be used for
types that don't have a built-in ordering and can as well be used for
types that do have a built-in ordering, but for which you want to use
some other ordering.

Although more general, it's a bit inconvinient to use: a caller must
specify an explicit ordering even if `T` is a type that has an obvious
default ordering such as `String` or `Int`. Let's consider the following
improvement:

```scala
def maxListOrdering[T](elements: List[T])(implicit ordering: Ordering[T]): T
  elements mathc {
    case List() => throw new IllegalArgumentException()
    case List(x) => x
    case x :: rest =>
      val maxRest = maxListOrdering(rest)(ordering)
      if (ordering.gt(x, maxRest)) x
      else maxRest
  }
```

which is an example of an implicit parameter used to provide more
information about a type explicitly mentioned previously in a parameter
list: the implicit parameter `ordering` of type `Ordering[T]` provides more
information about `T` - how to order `T`s and `T` obviously appears in the
earlier parameter list.

The fundamental thing is that elements must be provided explicitly, so the
compiler will know T at compile time and can therefore determine whether an
implicit definition of type Ordering[T] is available; if so, it can pass the
second argument implicitly. In fact, this pattern is so common that Scala
provides implicit ordering methods for most of the common types.

```scala
maxListOrdering(List(1, 8, 2, 17)) // 17
maxListOrdering(List(33.3, 5.2, 6.7, 3.14)) // 33.3
maxListOrdering(List("ab", "xy", "cd")) // ab
```

### A note on the naming of implicit parameters

- It's best to use a custom named type in the types of implicit
parameters, e.g like `PreferredPrompt` and `PreferredTimeFormat`
- Consider, however, a poor implementation of the max example we showed
earlier:

```scala
def maxList[T](element: List[T])(implicit orderer: (T, T) => Boolean): T
```

This is bad, because it's a fairly generic type and it doesn't indicate at
all what the purpose of the type is; it could be an equality test, a less
than test, or whatever. Using `Ordering[T]` on the other hand,
clearly indicates that the implicit parameter is used for ordering
elements of `T`.

## Context bounds

- When you use `implicit` on a parameter, not only will the compiler try to
supply that parameter with an implicit value, but it will also use that
parameter as an available implicit in the body of the method

```scala
def maxList[T](xs: List[T])(implicit ordering: Ordering[T]): T =
  xs match {
    case List() => throw new IllegalArgumentException()
    case List(x) => x
    case x :: rest =>
      val maxRest = maxList(rest) // ordering is implicit here
      if (ordering.gt(x, maxRest)) x
      else maxRest
  }
```

In other words, in the above example the compiler will see that the types
do not match up, because `maxList(rest)` supplies only one parameter list,
but `maxList` requires two; since the second is implicit, the compiler
looks for an implicit parameter of type `Ordering[T]`, finds one and
rewrites the call to `maxList(rest)(ordering)`.

You can also go further: eliminate the second use of `ordering`. It can be
done using the `implicitly` method defines in the standard librabry, like that:

```scala
def maxList[T](xs: List[T])(implicit comparator: Ordering[T]): T =
  xs match {
    case List() => throw new IllegalArgumentException()
    case List(x) => x
    case x :: rest =>
      val maxRest = maxList(rest)
      if (implicitly[Ordering[T]].gt(x, maxRest)) x
      else maxRest
  }
```

As you've noticed, `implicitly` has the following signature:

```scala
def implicitly[T](implicit t: T) = t
```

The effect of calling `implicitly[Foo]` is that the compiler will look
for an implicit definition of type `Foo` and will then call the
`implicitly` method with that object, which in turns returns the object
right back; put simply, you can write `implicitly[Foo]` whenever you want
to find an implicit object of type `Foo` in the current scope.

- Because this pattern is so common, Scala lets you leave out the name of
this parameter (`comparator`) and shorten the method signature by using
the so-called context bound

```scala
def maxList[T: Ordering](xs: List[T]): T =
  xs match {
    case List() => throw new IllegalArgumentException()
    case List(x) => x
    case x :: rest =>
      val maxRest = maxList(rest)
      if (implicitly[Ordering[T]].gt(x, maxRest)) x
      else maxRest
  }
```

- The syntax `[T: Ordering]` is a context bound and it does two things
- Introduces a type `T` as a normal type
- Adds an implicit parameter of type `Ordering[T]`
- The difference between using a context bound and not using a context
bound is that when using a context bound, you don't know what the
parameter will be called (you often don't need to know what's called
anyways)

Context bound can be thought of as saying something about a type parameter. 
For example, when you write `[T <: Ordered[T]]` you are saying that `T` is
an `Ordered[T]`. On the other hand, like in the previous example, when you
write `[T: Ordering]` you are not so much saying what `T` is, but you're
saying that there's some form of ordering (or any other property)
associated with `T`.

