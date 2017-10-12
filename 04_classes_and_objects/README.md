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



