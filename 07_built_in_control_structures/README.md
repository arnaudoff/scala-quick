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
    iteratively would look like this:

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
don't result in a value, to make any kind of difference, a `while` loop
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

- The `file <- filesHere` syntax is called a *generator*
- In each iteration, a new `val` named `file` is initialized with an
element value
- The compiler infers the type of `file` to be `File`,
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

### Mid-stream variable bindings

- In the previous example, `line.trim` is repeated
- `line.trim` is a non-trivial computation, hence it's a good idea to be
computed only once
- The result can be bound to a new variable using the equals sign; the
bound variable is introduced and used just like a `val`, only with the
`val` keyword left out

```scala
def grep(pattern: String) =
  for {
    file <- filesHere
    if file.getName.endsWith(".scala")
    line <- fileLines(file)
    trimmed = line.trim
    if trimmed.matches(pattern)
  } println(file + ":" + trimmed)
```

- Here, that variable is initialized to the result of `line.trim`, and
the rest of the `for` expressions then uses it in two places, once in
the `if` and once in `println`

### Producing a new collection

- So far, we've operated on the iterated values and then ignored them
- You can also generate a value to remember for each iteration
- The syntax is: `for <clauses> yield <body>`

Example (a function that identifies the `.scala` files and stores them in an
array):

```scala
def scalaFiles =
  for {
    file <- filesHere
    if file.getName.endsWith(".scala")
  } yield file
```

**Important**: The `yield` should be placed **before the entire body**,
even if the body is a block surrounded by curly braces, e.g. avoid this:

```scala
for (file <- filesHere if file.getName.endsWith(".scala")) {
  yield file // syntax error
}
```

To show yield once more, let's find how long are the lines that
contain "for" in all `.scala` files:

```scala
val forLineLengths =
  for {
    file <- filesHere
    if file.getName.endsWith(".scala")
    line <- fileLines(file)
    trimmed = line.trim
    if trimmed.matches(".*for.*")
  } yield trimmed.length
```


## Exception handling with `try` expressions

- Scala's exceptions behave similarly to other languages
- Instead of returning a value in the normal way, a method can terminate by
throwing an exception
- The method's caller can either catch and handle the exception, or simply
terminate, in which case the exception is propagated to the caller's,
caller
- The exception propagates by induction for the (n - 1)th method,
unwinding the call stack until a method handles it or there are no more
methods

### Throwing exceptions

- To throw an exception, you create an exception object and then `throw` it

```scala
throw new IllegalArgumentException
```

- `throw` is also an expression that has a result type
- The type of an exception throw is `Nothing`
- `throw` can be used as an expression, although it will never evaluate to
anything, it is frequently useful: one branch of an `if` computes a value,
while the other throws an exception and "computes" `Nothing`

Example:

```scala
val half =
  if (n % 2 == 0)
    n / 2
  else
    throw new RuntimeException("n must be even")
```

If `n` is even, `half` is initialized to the half of it, otherwise an
exception will be thrown before `half` can be initialized at all.

### Catching exceptions

- The syntax for `catch` was chosen for its consistency with
*pattern matching*, a powerful feature which we will discuss later
- The behavior of the `try-catch` expression is the same as in other
languages with exceptions

```scala
import java.io.FileReader
import java.io.FileNotFoundException
import java.io.IOException

try {
  val f = new FileReader("input.txt")
  // use and close
} catch {
  case ex: FileNotFoundException => // missing file
  case ex: IOException => // I/O error
}
```

### The `finally` clause

- Similarly to other languages, you can wrap an expression with a `finally`
clause if you want a piece of code to execute no matter how the expression
terminates, e.g. ensure that an open file gets closed always

```scala
import java.io.FileReader

val file = new FileReader("input.txt")
try {
  // use the file
} finally {
  file.close() // ensure the file is closed
}
```

### Yielding a value

- Similarly to other Scala control structures, `try-catch-finally` results
in a value

Example (parse a URL and use a default if the original is badly formed):

```scala
import java.net.URL
import java.net.MalformedURLException

def urlFor(path: String) =
  try {
    new URL(path)
  } catch {
    case e: MalformedURLException =>
      new URL("http://www.scala-lang.org")
  }
```

- The resulting value is that of the `try` clause in case of no exception
thrown, or the relevant `catch` clause if an exception is thrown and caught
- If an exception was thrown, but not caught, the expression would have no
result at all: the value computed in the `finally` clause, if there was
one, would be dropped (in general, `finally` clauses do some kind of clean
up and should not change the value computed in the main body/`catch`
clause of the `try`)

**Note**: Scala's behavior differs from Java because
Java's `try-finally` does not result in a value. For example, as in Java,
if a `finally` clause includes an explicit `return` or throws, that return
value or exception overrules any previous one that originated in the try
block or in the catch clauses. This "surprising" behavior is illustrated
by the following examples:

```scala
def f(): Int = try return 1 finally return 2 // f() results in 2
```

By contrast, this also happens:

```scala
def g(): Int = try 1 finally 2 // g() results in 1
```

In general, it's best to avoid returning values from `finally` clauses
and stick to thinking of them as a way to ensure some side effect happens,
be it closing an open file or something else.

## Match expressions

- `match` is equivalent to a `switch` statement in other languages, but it
allows you to select using some patterns (which is more powerful,
but described later)
- The default case is specified with an underscore, typically a
wildcard symbol frequently used in Scala as a placeholder for unknown value

Example (consider a daily routine):

```scala
val firstArg = if (args.length > 0) args(0) else ""

firstArg match {
  case "friday" => println("drinking")
  case "saturday" => println("coding")
  case "sunday" => println("doing homework")
  case _ => println("doing maths")
}
```

- Unlike in Java, where you can use integers, enums and string
constants in the case, here any kind of constant (as well as other things)
will do
- Note that in Scala, no `break`s are needed, the `break` is implicit
- Most importanly, `match` results in a value, e.g. in the above example
it would work just as well to yield the value rather than printing it:

```scala
val firstArg = if(!args.isEmpty) args(0) else ""

val activity =
  firstArg match {
    case "friday" => "drinking"
    case "saturday" => "coding"
    case "sunday" => "doing homework"
    case _ => "doing maths"
}

println(activity)
```

## A note about `break` and `continue`

- As you may have noticed, Scala leaves out `break` and `continue` as
they don't really mix well with the concepts of functional programming,
especially function literals

A good example of how life can still be meaningful without these two is the
following (suppose you're searching the argument list for a string
ending with `.scala`, but does not start with a hyphen):

```java
int i = 0;
boolean foundIt = false;
while (i < args.length) {
  if (args[i].startsWith("-")) {
    i = i + 1;
    continue;
  }

  if (args[i].endsWith(".scala")) {
    foundIt = true;
    break;
  }

  i = i + 1
}
```

In Scala, you could translate this code by replacing every continue by an
if and every break by a boolean variable, where it indicates whether the
enclosing loop should continue:

```scala
var i = 0
var foundIt = false

while (i < args.length && !foundIt) {
  if (!args(i).startsWith("-")) {
    if (args(i).endsWith(".scala"))
      foundIt = true
  }

  i = i + 1
}
```

Or, if you're a real man (you do functional programming), you'll write it
recursively:

```scala
def searchFrom(i: Int): Int =
  if (i >= args.length) -1
  else if (args(i).startsWith("-")) searchFrom(i + 1)
  else if (args(i).endsWith(".scala")) i
  else searchFrom(i + 1)

val i = searchFrom(0)
```

Here's an interesting note to make: the compiler will actually optimize the
recursion, because all of the recursive calls are in tail-call position,
it will generate code similar to a while loop, e.g. each recursive
call will be implemented as a jump back to the beginning of the function.

- After all, if you want to use a `break`, check out
`scala.util.control.Breaks`.

## Variable scope

- Scala's scoping rules are almost identical to Java's and most programming
languages

A notable difference is the following:

```scala
val a = 1; // btw, Scala will not place a ; here implicitly
{
  val a = 2 // compiles fine
  println(a)
}
println(a)
```

Java will not allow you to create a variable in an inner scope with the same
name as the variable in the outer scope, however, in a Scala program the
inner variable *shadows* a like-named outer variable, because the outer
variable becomes invisible in the inner scope.

- In general, prefer to choose a new, meaningful variable name rather
than shadowing an outer variable

## Refactoring imperative-style code

In this lecture, we will again look at an imperative to functional conversion
just so you can get a feel for the functional style faster.

Example (printing a multiplication table):

```scala
def printMultiTable() = {
  var i = 1
  while (i <= 10) {
    var j = 1
    while (j <= 10) {
      val prod = (i * j).toString
      var k = prod.length
      while (k < 4) {
        print(" ")
        k += 1
      }

      print(prod)
      j += 1
    }

    println()
    i += 1
  }
}
```

becomes

```scala
// Returns row as a sequenceco
def makeRowSeq(row: Int) =
  for (col <- 1 to 10) yield {
    val prod = (row * col).toString
    val padding = " " * (4 - prod.length)
    padding + prod
  }

// Returns a row as a string
def makeRow(row: Int) = makeRowSeq(row).mkString

// Returns table as a string
def multiTable() = {
  val tableSeq =
    for (row <- 1 to 10)
      yield makeRow(row)
  tableSeq.mkString("\n")
}
```

The result is obvious: the multiplication table in the form of a matrix.

