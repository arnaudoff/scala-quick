# Functions and closures

We deal with large programs and complexity by dividing these large
programs into smaller, manageable pieces. Scala offers a technique for
doing that: the concept of a function. In fact, Scala extends
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

## Short form of function literals

- One way to make a function literal shorter is to leave off the parameter
types, thus the previous example can be shortened to:

```scala
numbers.filter((x) => x > 3)
```

- The Scala compiler infers `x`'s type, because it knows you're filtering
a list of integers
- This is called *target typing*, because the targeted usage of an
expression (the argument to the `filter()` call) is allowed to influence
the typing of that expression (the type of `x`)
- A good strategy when using literals is to simply omit the type,
and add it if the compiler is confused :)
- We can go futher with the shortening - we can leave out the parentheses
around a parameter whose type is inferred:

```scala
numbers.filter(x => x > 3)
```

## Placeholder syntax

- Scala goes even further: since this construct is pretty common, you can
use underscores as placeholders for one or more parameters as long as each
parameter appears only once within the function literal:

```scala
numbers.filter(_ > 3)
```

- The underscore can be thought of as a blank value in the expression that
needs to be "filled in", which will be filled in with an argument each
time the function is invoked, e.g. if `numbers` is `List(1, 2, 3, 4, 5)`,
`filter` will replace `_ > 3` first with 1 (1 > 3), then 2 (2 > 3) etc.
- Thus, the literal `_ > 3` is equivalent to the verbose `x => x > 3`

## Partially applied functions

- The previous examples simply substitute underscores for individual
parameters, however, you can replace an entire parameter list with
an underscore
- For instance, instead of `println(_)`, you could write `println _`

```scala
numbers.foreach(println _) // equivalent to numbers.foreach(x => println(x))
```

- In this case, the underscore is not a placeholder for a single
parameter, but for an entire parameter list!
- Psst, note that you need the space between `println` and `_`, otherwise
the compiler would think you're referring to `println_`, which doesn't exist
- When you use an underscore in this way, you're writing
a *partially applied function*
- Typically, when you invoke a function, passing in arguments, you
**apply the function to the arguments**:

```scala
def sum(a: Int, b: Int, c: Int) = a + b + c

sum(1, 2, 3) // applies sum to the arguments 1, 2 and 3
```

- Thus, a *partially applied function* is an expression in which you don't
supply all of the arguments needed by the function; you supply some
or even none of the needed args:

```scala
// create a PAF involving sum, supplying none of the required arguments
val a = sum _
```

- All this does is instantiate a function value that takes the three integer
parameters missing from the PAF expression, `sum _`, and assign a
reference to that new function value to the variable `a`
- When you apply three arguments to this new function value, it will
turn around and invoke `sum`, passing in those same three arguments:

```scala
a(1, 2, 3) // returns 6
```

Here's the explanation (make sure to understand how this works): the
variable `a` refers to a function value; it's an instance of a class
generated automatically from `sum _`, the PAF expression. The class has
an `apply` method that takes three arguments (because three arguments
are missing from `sum _`). The compiler translates `a(1, 2, 3)` into an
invocation of the function value's apply method, thus `a(1, 2,3)` is a
shorthand for `a.apply(1, 2, 3)`. The `apply` method simply forwards
those three missing parameters to `sum`, and returns the result.

This technique is often times used when you want to assign a method
or nested function to a variable, or pass it as an argument to another
function; you can do these things if you wrap the method/nested
function in a function value.

- Also, PAFs can have some argument supplied (after all, that's why they're
called partial):

```scala
val b = sum(1, _: Int, 3)
```

- The compiler now generates a class whose `apply` method takes only one
argument, when invoked with it, this generated `apply` invokes `sum`,
passing in `1`, the passed argument, and `3`:

```scala
b(2) // b.apply invokes sum(1, 2, 3)
```

Also, it's worth noting that when a function is expected and you write a PAF
with all arguments missing, the underscore can be omitted, e.g.

```scala
numbers.foreach(println _)
```

becomes

```scala
numbers.foreach(println)
```

## Closures

- In function literals, you can not only refer to the passed parameters,
but to variables defined outside:

```scala
(x: Int) => x + foo // foo is defined outside the function body
```

- In this case, `foo` is called a *free variable*, because the function
literal does not itself introduce it
- The `x` variable, by contrast, is a *bound variable*

```scala
var foo = 1
val addFoo = (x: Int) => x + foo
addFoo(10) // 11
```

- The function value that's created from this function literal is called a
*closure*, because in a sense it's "closing" the function literal by
capturing the bindings of its free variables
- A function literal with no free variables, e.g. `(x: Int) => x + 1`, is
called a closed term (a function value created at runtime for this
literal is not a closure, strictly speaking)
- A function literal with free variables, such as `(x: Int) => x + foo`
is an open term, and by definition the resulting function value would be a
closure

Note: if `foo` changes after the closure is created, the closure "sees" the
change, e.g. `foo = 9999` and `addFoo(10)` results in `10009`! The
reverse is also valid: changes made by a closure to a captured variable
are visible outside the closure.
