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

