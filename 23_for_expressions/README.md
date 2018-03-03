# `for` expressions

Through the course, we've praised functional style and encouraged the use of
higher-order functions such as `map`, `flatMap` and so on. Sometimes, however,
in order to make code more readable, you have to lower the level of abstraction.

## An example of simpler `for` code

Say you have the following class definition:

```scala
case class Person(name: String, isMale: Boolean, chilren: Person*)
```

Now, say you have a list of people and you want to find out the names of all
pairs of mothers and their children in that list. One way to do it is to use
the classical functional approach:

```scala
persons.filter(p => !p.isMale)
  .flatMap(p => (p.children.map(c => (p.name, c.name))))
```

Now, the above code, although correct, looks a bit tough to read. In fact,
there's a simpler way to achieve the same thing using `for` expressions:

```scala
for (p <- persons; if !p.isMale; c <- p.children)
  yield (p.name, c.name)
```

Interestingly, the compiler actually translates all such for expressions that
yield a result into combinations of calls to higher-order methods such as `map`,
`flatMap` and so on.

## `for` expressions

By definition, a `for` expression is of the form
`for (sequence) yield expression`, where `sequence` is a sequence of generators,
definitions and filters with semicolons inbetween, for example:

```scala
for (p <- persons; n = p.name; if (n startsWith "Jo"))
yield n
```

or equivalentnly

```scala
for {
  p <- persons // a generator
  n = p.name // a definition
  if (n startsWith "Fo")  // a filter
}
```

where a generator is of the form `pattern <- expression` with `pattern` is
matched one-by-one to the elements of `expression` (typically some list), a
definition is of the form `pattern = expression` and a filter which is of the
form `if expression`.

## Querying with `for` expressions

- One could argue that the `for` is actually similar to common operations of
    database DSLs

Let's look at some examples of querying a database, which in our case will be an
in-memory list of objects.

```scala
case class Subject(name: String, teachers: String*)

val subjects: List[Subject] =
  List(
    Subject("Calculus", "John Smith"),
    Subject("Multivariable calculus", "John Smith", "Tony Rivers"),
    Subject("Formal languages and automata", "Tom Stevens", "Larry Prick")
    Subject("Discrete mathematics", "Jackie Wonders", "Phill Ferdinand")
    // ...
  )
```

So, to find the names of all subjects taught by people whose first name is `John`:

```scala
for (s <- subjects; t <- s.teachers; if t startsWith "John")
  yield s.name
```

or, find the names of all subjects that have "calculus" in their name:

```scala
for (s <- subjects if (s.name indexOf "calculus") >= 0)
  yield s.name
```

and so on and so forth.

## Translation of `for` expressions

- Every for expression can be expressed in terms of `map`, `flatMap` and
    `withFilter`

### Translating a one-generator `for` expressions

First, assume you have the simplest for expression:

```scala
for (x <- firstExpr) yield secondExpr
```

this is translated to

```scala
firstExpr.map(x => secondExpr)
```

### Translating `for` expressions with a generator and a filter

Consider this `for` expression:

```scala
for (x <- firstExpr if secondExpr) yield thirdExpr
```

This is first translated to

```scala
for (x <- firstExpr withFilter(x => secondExpr)) yield thirdExpr
```

which gives a problem similar to the first one and the translation continues:

```scala
firstExpr withFilter(x => secondExpr) map (x => thirdExpr)
```

In general, you get the following formula (if `seq` is an arbitrary sequence of
generators, definitions and filters):

```scala
for (x <- firstExpr if secondExpr; seq) yield thirdExpr
```

is translated to

```scala
for (x <- firstExpr withFilter secondExpr; seq) yield thirdExpr
```

then the translation continues with the second expression.

### Translating `for` expressions starting with two generators

Assuming `seq` is an arbitrary sequence of generators, definitions and filters:

```scala
for (x <- firstExpr; y <- secondExpr; seq) yield thirdExpr
```

is translated into

```scala
firstExpr.flatMap(x => for (y <- secondExpr; seq) yield thirdExpr)
```

the `for` expression left in the function value passed to `flatMap` is
translated with the same rules as above expressions.

### Translating tuples and patterns in generators

When a tuple is involved in the left hand side of a generator instead of a
simple variable, the translation scheme is as follows:

```scala
for ((x1, ..., xn) <- firstExpr) yield secondExpr
```

translates to

```scala
firstExpr.map { case (x1, ..., xn) => secondExpr }
```

If an arbitrary pattern is in the left hand side of a generator, then

```scala
for (pattern <- firstExpr) yield secondExpr
```

translates to

```scala
firstExpr withFilter {
  case pattern => true
  case _ => false
} map {
  case pattern => secondExpr
}
```

or, in other words, the generated items are first filtered and only those that
match the pattern are mapped.

### Translating definitions

Assuming again that `seq` is a sequence of generators, definitions, and filters:

```scala
for (x <- firstExpr; y = secondExpr; seq) yield thirdExpr
```

is translated to

```scala
for ((x, y) <- for (x <- firstExpr) yield (x, secondExpr); seq)
yield thirdExpr
```

In other words, `secondExpr` is evaluated each time there's a new `x` value
being generated. So, it's not a good idea to have definitions embedded in `for`
expressions that do not refer to variables bound by a preceding generator,
because re-evaluating these expressions would be a waste of resources.

Example:

```scala
for (x <- 1 to 1000; y = someExpensiveCall)
  yield x * y
```

is a poor style, because `someExpensiveCall` is evaluated 1000 times. Rather, the above construction could be replaced by the more efficient:

```scala
val precomputedY = someExpensiveCall
for (x <- 1 to 1000)
  yield x * precomputedY
```

### Translating simple `for` loops

- As explained, the difference between `for` expressions and `for` loops is that
loops simply perform a side effect without returning anything (no yield)
- Typically, whereas `for` expressions involve `map`s and `flatMap`s in their
    translations, a `for` loop is simply a `foreach`:

```scala
for (x <- firstExpr)
  body
```

is translated to

```scala
firstExpr.foreach(x => body)
```

- As a practical example, consider summing up all elements of a matrix
    represented as list of lists

```scala
var sum = 0
for (xs <- xss; x <- xs)
  sum += x
```

is translated to

```scala
var sum = 0
xss foreach (xs =>
  xs foreach (x =>
    sum += x
  )
)
```

## Translating higher-order methods to `for` expressions

Sometimes you want to achieve the opposite: translate higher-order methods
invocations to a single `for` expression. Here's a sample implementation of
the three fundamental methods:

```scala
object Translations {
  def map[A, B](xs: List[A], f: A => B): List[B] =
    for (x <- xs) yield f(x)

  def flatMap[A, B](xs: List[A], f: A => List[B]): List[B] =
    for (x <- xs; y <- f(x)) yield y

  def filter[A](xs: List[A], p: A => Boolean): List[A] =
    for (x <- xs if p(x)) yield x
}
```

## `for` everywhere

Since `for` translations rely on the presence of `map`, `flatMap` and
`withFilter`, it's possible to apply the `for` to any type that defines them.

- Lists and arrays, as seen, support `for` expressions because they define
    operations `map`, `flatMap` and `withFilter`
- Besides list and arrays, ranges, iterators, streams and set implementations
    also define these methods and thus allow `for` expressions

So, if you have your own data type, to support `for` expressions and `for` loops
your must define the four operations `map`, `flatMap`, `withFilter` and
`foreach`. The rules are simple - if your type:
- has only `map` defined, it allows `for` expressions with a single generator
- has `flatMap` and `map` defined, it allows expressions with several generators
- has `foreach` defined, it allows `for` loops
- has `withFilter` defined, it allows for filter expressions in the `for`
    expression

A typical setup for some class that is parameterized, say `Foo`, which supports
`for` expressions and `for` loop is as follows:

```scala
abstract class Foo[T] {
  def map[U](f: T => U): Foo[U]
  def flatMap[U](f: T => Foo[U]): Foo[U]
  def withFilter(p: T => Boolean): Foo[U]
  def foreach(block: T => Unit): Unit
}
```

- In functional programming, there's a general concept called a *monad*, which
    sort of explains a large number of types with computations, e.g.
    collections, computations with state, I/O, transactions and so on.
- Functions such as the first three  - `map`, `flatMap` and `withFilter` can be
    formulated on a monad and if so, they end up having the same types as given
    here
- Additionally, every monad can be characterized by these three functions and a
    "unit" constructor that produces a monad from an element value
- So in summary, the first three methods can be seen as an object-oriented
    version of the functional programming concept called monad

There's something more to that. Since the for expressions are equivalent to
these three methods, they can be seen as syntax for monads. In other words, the
concept of a `for` expression is more general.
