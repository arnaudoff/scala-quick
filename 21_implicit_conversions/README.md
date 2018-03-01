# Implicit conversions and parameters

- A typical problem in software development is extending an already existing
    third party library
- Scala solves it by using implicit conversions and parameters
- This chapter shows how implicits work and presents some common use cases

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

Here's where implicit conversions we can "tie" this code and make it work.
Let's write an implicit conversion between the two types, in particular
from functions to action listeners:

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
conversion defined that solves the problem, find `function2ActionListener`

Powerful, huh?
