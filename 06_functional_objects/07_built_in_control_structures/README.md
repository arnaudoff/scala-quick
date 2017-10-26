# Built-in control structures

Scala has only a handful of built-in control structures. The only control
structures are `if`, `while`, `for`, `try`, `match` and function calls. As
explained, Scala aims at allowing you to build your own structures.
The idea of this lecture is to show only those few
control structures that are built-in. Also, beware that since we're in the
functional world, these structures result in some value. This allows our
code to be more concise, by removing the need of using temporary variables.

## `if` expressions

- The `if` works just like in many other languages, e.g.

```scala
var filename = "default.txt"
if (!args.isEmpty)
  filename = args(0)
```

- The given example is imperative, but the proper functional way would be:

```scala
val filename =
    if (!args.isEmpty) args(0)
    else "default.txt"
```

- Note that the `if` here has a result value
- Therefore, we've removed the need for the temporary variable
(and it uses a `val`)
- A second advantage to using a `val` instead of a `var` is that it
better supports *equational reasoning*, in other words, the introduced
variable is equal to the expression that computes it considering the
expression has no side effects

Example:

Instead of `println(filename)`, the equivalent operation would be:

```scala
println(if (!args.isEmpty) args(0) else "default.txt")
```

- In general, using `val`s helps you safely make similar kind
of "refactorings" as your code evolves over time

## `while` loops

- The `while` loop works just like in other languages, e.g. gcd written
    iteratively would look like that:

```scala
def gcdLoop(x: Long, y: Long): Long = {
  var a = x
  var b = y
  while (a != 0) {
    val temp = a
    a = b % a
    b = temp
  }
  b
}
```

- The `do`-`while` also exists in Scala

```scala
var line = ""
do {
  line = readLine()
  println("Read: " + line)
} while (line != "")
```

- Note that `while` and `do-while` are called loops and not expressions,
because they don't result in an interesting value, e.g. the type of the
result is `Unit`
- Interesting: only one value exists whose type is `Unit`, it's `()`
- The existence of `()` differentiates Scala's `Unit` from Java's `void`

Example:

```scala
def greet() = { println("yo") } // no equals sign => result type is Unit
() == greet() // returns true
```

- In general, `while` loops, just like `var`s, should be avoided
- In fact, often `while` loops and `var`s go together, because `while` loops
don't result in a value, to maky any kind of difference, a `while` loop
typically updates `var`s or performs I/O

## `for` expressions

- The `for` expression solves any problems related to iteration
- It let's you combine simple ingredients in different ways to express
a wide variety of iterations, e.g you can iterate multiple collections,
filter out elements based on conditions, produce new collections and so on

### Iteration through collections

- The simplest of all iterations possible with `for`

Example:

```scala
val filesHere = (new java.io.File(".")).listFiles

for (file <- filesHere)
  println(file)
```

Few things to note:
- The `file <- filesHere` syntax is called a *generator*
- In each iteration, a new `val` named `file` is initialized with an
element value
- The compiler inferts the type of `file` to be `File`,
because `filesHere` is an `Array[File]`

- `for` works for any kind of collection, e.g. a `Range` type (generated
by syntax like `1 to 5`):

```scala
for (i <- 1 to 4)
  println("Iteration " + i)
```

- Protip: Use `until` instead of `to` to avoid the upper bound of the range

### Filtering

- Often times, you want a to iterate a subset of a collection
- This is done using a *filter*: an `if` clause inside the `for` parentheses

Example (filtering for files ending in `.scala`):

```scala
val filesHere = (new java.io.File(".")).listFiles

for (file <- filesHere if file.getName.endsWith(".scala"))
  println(file)
```

An interesting case to examine is what happens when you use the
seemingly equivalent code segment:

```scala
for (file <- filesHere)
  if (file.getName.endsWith(".scala"))
    println(file)
```

- Seemingly equivalent, because this `for` expression is executed only
for its printing side-effects and results in the unit value `()`
- The `for` expression is called an expression, because it can result in a
value, a collection whose type is determined by the `for` expression `<-`
clauses, otherwise it's simply imperative

More filters can be added by simply adding more `if` clauses:

```scala
for (
  file <- filesHere
  if file.isFile
  if file.getName.endsWith(".scala")
) println(file)
```

### Nested iterations

- You can add multiple `<-` clauses, which in a sense is equivalent to a
"nested loop"

Example (simple implementation of grep):

```scala
def fileLines(file: java.io.File) =
  scala.io.Source.fromFile(file).getLines().toList

def grep(pattern: String) =
  for (
    file <- filesHere
    if file.getName.endsWith(".scala");
    line <- fileLines(file)
    if line.trim.matches(pattern)
  ) println(file + ": " + line.trim)
```

