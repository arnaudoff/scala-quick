# Classes and objects


In this lecture, you will learn more about classes, fields and methods in Scala. 

## Classes, fields and methods


- Just like in any OO language, a class is a blueprint for objects
- You can create objects with the `new` keyword

```scala
class Person {
  // class definition
}
```

```scala
new Person
```

- In the class definition, you place fields and methods, collecively
called members
- You can say it's a field if the member is defined with `val` or `var`,
they refer to objects and they hold state (also called *instance variables*)
- Methods are defined with `def`, they contain executable code, hence
they do the "computational" work
- When you instantiate a class, the runtime allocates memory to hold
the object state

Consider the following:

```scala
class Person {
  var age = 16
}

val john = new Person
john.age = 15
```

It's important to understand that since `age` is defined as a `var`, you
can later reassign it, even though `john` is a `val`. In essence, this
means that you can mutate the object, but you can not reassign `john` to
another object of the same type.

- Scala also supports access modifiers just like other languages, e.g.
you can make `age` private by annotating it with `private` before the type
- Unsurprisingly, private fields can be accessed by methods defined in
the same class
- This ensures that the object state remains valid during its lifetime
(because outsiders can't mutate it)
- By default, the access modifier is public, so where you say
"public" in Java, you say nothing in Scala

Now, let's use another example:

```scala
class ChecksumAccumulator {
  private var sum = 0

  def add(b: Byte): Unit = {
    sum += b
  }

  def checksum(): Int = {
    return ~(sum & 0xFF) + 1
  }
}
```

- Both `add` and `checksum` are example for methods
- Similarly to other languages, in Scala parameters to a method can be
used inside it
- Note that they are `val`s, and not `var`s: you can't reassign a parameter
inside a method

```scala
def add(b: Byte): Unit = {
  b = 1 // invalid
}
```
- You can omit the return from `checksum`: in the absence of any explicit
return, Scala methods return the last evaluated expression
- In that sense, it's better to think of methods as expressions that
yield one value, which is returned
- In fact, this way of thinking forces you to refactor large methods into
small, concise ones

We can be even more concise: if a method computer only a single expression, you
can omit the curly braces

```scala
class ChecksumAccumulator {
  private var sum = 0
  def add(b: Byte) { sum += b }
  def checksum(): Int = ~(sum & 0xFF) + 1
}
```

- Written this way, `add` looks more like a procedure: a method executed
only for its side effects (reassigning `sum`)
- So, leaving off the result type and the equals sign and seeing the
body of the method in curly braces is a good example of indication that
a method is non-functional

**Note**: Watch out for missing `=`! Whenever you leave it off before the
body of a function, its result type is assumed to be `Unit` and even if
you compute and return some value, it'll be lost!

```scala
def f(): Unit = "this String gets lost"
```

Same goes to

```scala
def g() { "this String gets lost as well" }
```

## Singleton objects

- As mentioned in the first lecture, Scala conforms to the OO principles
more than Java because classes in Scala cannot have static members
- Instead, Scala provides *singleton objects*
- A singleton object definition looks like a class, except that you use the
`object` keyword

```scala
import scala.collection.mutable.Map

object ChecksumAccumulator {

  private val cache = Map[String, Int]()

  def calculate(s: String): Int =
    if (cache.contains(s))
      cache(s)
    else {
      val acc = new ChecksumAccumulator
      for (c <- s)
        acc.add(c.toByte)
      val cs = acc.checksum()
      cache += (s -> cs)
      cs
    }
  }
}
```

The code is pretty much self-explanatory so we will concentrate on the concepts
rather than explaining its purpose:

- The singleton object has the same name as the class in the previous example
- When a singleton object shares the same name with a class, it's called
the *companion object* of the class
- The class and its companion object should be defined in the same source file
- A class and its companion object can access each other's private members
- From Java's point of view, singleton objects can be seen as the home for any
static methods one might have written in Java
- Methods on singleton objects are invoked with a dot and the name of the
method, i.e:

```scala
ChecksumAccumulator.calculate("Eat, drink, code, repeat")
```
- However, a singleton object is more than a holder of static methods - it's a
*first-class object*: its name can be thought of as a name tag attached to the
object
- Defining a singleton object doesn't define a type: given only a definition of
`ChecksumAccumulator`, you can't create a variable of type `ChecksumAccumulator`
- Rather, the type is defined by the compaion class of the singleton object!
- Singleton objects, however, can extend classes and can mix in traits
- Singleton objects, unlike classes, cannot take parameters (because you can't
    instantiate them in the first place)
- Singleton objects are typically implemented as an instance of a synthetic
class referenced from a static variable (they have similar initialization
semantics as Java statics: a singleton object is initialized the first time some
code accesses it)
- A singleton object that does not have a companion class is called a *standalone
object* (typically useful for collecting utility methods and defining an
entry point)
