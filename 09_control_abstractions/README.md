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
