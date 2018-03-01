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
