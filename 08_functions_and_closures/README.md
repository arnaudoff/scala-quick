# Functions and closures

We deal with large programs and complexity by dividing these large
programs into smaller, manageable pieces. Scala offers a technique for
doing that: the concept of a function. In fact, Scala extends this
this familiar concept by offering several ways to define functions;
there's also function literals and function values, which we will talk
about in this lecture.

## Methods

- A method is a function that is a member of some object

Example:

```scala
import scala.io.Source

object LongLines {
  def processFile(filename: String, width: Int) = {
    val source = Source.fromFile(filename)
    for (line <- source.getLines())
      processLine(filename, width, line)
  }

  private def processLine(filename: String, width: Int, line: String) = {
    if (line.length > width)
      println(filename + ": " + line.trim)
  }
}
```

- In the above example, `processFile` and `processLine` are methods. Sample
usage would be:

```scala
object FindLongLines {
  def main(args: Array[String]) = {
    val width = args(0).toInt
    for (arg <- args.drop(1))
      LongLines.processFile(arg, width)
  }
}
```

- So far, this is similar to what you would do in any OO language
- However, Scala extends the concept of a function

## Local functions

- The above example demonstrated an important design principle of the
functional programming (and programming in general): programs should be
decomposed into many small functions that each do a well-defined task
- The advantage of this style is that it allows you to compose more complex
things from simple ones (e.g allows for abstraction, think mathematics
where you have fundamental definitions and build upon them)
- One problem with this approach is that all the helper function names can
pollute the program namespace
- Once functions are packaged into reusable classes, it's a good idea to
hide the helper functions from clients of a class
- In Java, the main tool for the job is the `private` method, which works in
Scala too, however, you can also define a function inside another
function, so that it's  visible only in the encosing block (*local
functions*)

```scala
def processFile(filename: String, width: Int) = {

  def processLine(line: String) = {
    if (line.length > width)
      println(filename + ": " + line.trim)
  }

  val source = Source.fromFile(filename)
  for (line <- source.getLines()) {
    processLine(line)
  }

}
```

- Note how we transformed the method from `private` to a local function
to the `processFile` method
- Note also that we refactored the parameter list of `processLine`, since
local functions can access the parameters of their enclosing function
- The above example is simple, and very powerful, especially in a language
with first-class functions

## First-class functions

- You can also write down functions as unnamed literals and then pass
them around as values. This concept is called a *first-class function*
- A function literal is compiled into a class that when instantiated at
runtime, becomes a function value (make sure to understand this!)
- The main difference between function literals and values is that function
literals exist in the source code, whereas function values exist
as objects at runtime (this too!)

Example (of a function literal that adds one to a number):

```scala
(x: Int) => x + 1
```

- `=>` informally means "convert the thing on the left to the thing on
the right", in a sense it represents the mapping from any integer `x`
to `x + 1`
- Function values are objets, so you're totally fine storing them in
variables if you want to

```scala
var inc = (x: Int) => x + 1
```

- Since they are functions too, you can invoke them using the usual
parentheses notation

```scala
inc(10) // yields 11
```

- In Scala, every function value is an instance of some class that
extends one of several `FunctionN` traits, e.g. `Function0` for
functions with no params, `Function1` for functions with one parameter
(like `inc`), ..., `FunctionN` for functions with N parameters.

It's worth nothing that you can have more than one statement in the function
literal, e.g. the following is absolutely legal:

```scala
inc = (x: Int) => {
  println("yo")
  println("whats up")
  x + 1
}
```

When the function value is invoked, all of the statements in the block
will be executed, and the value returned from the function is the
result from evaluating the last expression.

Here's a practical example of function literals (which you likely have
even seen in other languages):

```scala
val numbers = List(1, 2, 3, 4, 5, 6)
numbers.foreach((x: Int) => println(x))
numbers.filter((x: Int) => x > 3) // returns List(4, 5, 6)
```

If curious, the foreach method is defined in trait `Traversable`, a common
supertrait of `List`, `Set`, `Array` and `Map`.
