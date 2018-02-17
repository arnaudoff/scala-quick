# Control abstractions

- As mentioned, Scala doesn't have many built-in control structures
- This lecture shows how to apply function values to create new control
abstractions

## Reducing code duplication

- Higher-order functions: functions that take functions as parameters
- Example: a file browser with an API that allows users to search for files
matching some criterion

```scala
object FileMatcher {
  private def files = (new java.io.File(".")).listFiles

  def filesEnding(query: String) =
    for (file <- files; if file.getName.endsWith(query))
      yield file
}
```

Eventually, wanting to improve it, you end up changing the API to :

```scala
def filesContaining(query: String) =
  for (file <- files; file.getName.contains(query))
    yield file
```

Someday later, you allow regex search:

```scala
def filesRegex(query: String) =
  for (file <- files; file.getName.matches(query))
    yield file
```

And so on, to avoid the repetition one may notice the obvious repetition and try
to factor it into something like this:

```scala
def filesMatching(query: String, method) =
  for (file <- files; file.getName.method(query))
    yield file
```

- Solution: use a function value (notice the type of it)

```scala
def filesMatching(query: String, matcher: (String, String) => Boolean) = {
  for (file <- files; if matcher(file.getName, query))
    yield file
}
```

- Let's use the common solution now:

```scala
def filesEnding(query: String) = filesMatching(query, _.endsWith(_))
def filesContaining(query: String) = filesMatching(query, _.contains(_))
def filesRegex(query: String) = filesMatching(query, _.matches(_))
```

- Note: as explained in the previous lecture, `_.matches(_)` translates to
`(fileName: String, query: String) => fileName.matches(query)`
- Again, because `filesMatching` takes a function that requires two `String`
arguments, you can omit their types: `(fileName, query) =>
fileName.matches(query)` and because the parameters are each used only once in
the body of the function, the first `_` simply means `fileName` and the second `_` simply
means `query`

## Simplying client code

We'll look into another use of higher order functions. Consider:

```scala
def containsNeg(nums: List[Int]): Boolean = {
  var exists = false
  for (num <- nums)
    if (num < 0)
      exists = true
  exists
}
```

A lot more cleaner way to express the same thing is writing this:

```scala
def containsNeg(nums: List[Int]) = nums.exists(element => element < 0)
```

## Currying

Let's define a function that sums two integers

```scala
def classicSum(x: Int, y: Int) = x + y
```

Now let's examine the same function, but using currying as a concept:

```scala
def curriedSum(x: Int)(y: Int) = x + y
```

Currying is essentially a syntax sugar for doing this:

```scala
def first(x: Int) = (y: Int) => x + y
```

```scala
val second = first(1) // yields Int => Int = <function1>

// now apply the result to 2

second(2) // yields 3
```

So, in the sum example, when we invoke `curriedSum`, we'll actually have two
traditional functional invocations, the first one takes a single `Int` as a
parameter (`x`), and returns a function value for the second function, this
second function takes the `Int` parameter `y`.

## Building new control structures

- In languages with first-class functions, one can effectively make new control
structures by simply creating methods that take functions as arguments

Let's define a control structure that repeats an operation two times and returns
the result:

```scala
def twice(op: Double => Double, x: Double) = op(op(x))
```

From a practical point of view, you often see opening a resource, operating on
it, and then closing it, so let's see how that would work using these new
techniques:

```scala
def withPrintWriter(file: File, op: PrintWriter => Unit) = {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
```

The usage is rather cool:

```scala
withPrintWriter(
    new File("date.txt"),
    writer => writer.println(new java.util.Date)
)
```

- This is called loan pattern: a control-abstraction function opens a resource
and "loans" it to a function
- By the way, you can use curly braces in client code to make it look like more
native, e.g. `println { "Hello world!" }` instead of `println("Hello, world!")`,
of course this works when using one argument only

Let's consider `withPrintWriter` again. It takes two arguments, so you can't use
curly braces, but since the function you pass to it is the last argument, we can
use currying to pull the first argument, into a separate argument list, which
leaves the function as lone parameter of the second argument list, e.g:

```scala
def withPrintWriter(file: File)(op: PrintWriter => Unit) = {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
```

Now, we can use the more pleasing syntax:

```scala
val file = new File("date.txt")

withPrintWriter(file) {
    writer => writer.println(new java.util.Date)
}
```
