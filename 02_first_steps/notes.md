# First steps in Scala

The following text will give you the big picture of the language and get you
writing some code.

## The Scala interpreter

- Scala has an interactive shell called `scala`
- It's similar to what Python, Ruby etc. have
- Typing an expression will evaluate it and print the result
- It's started by calling `$ scala`

Example: calculating `1 + 2` would result (`res0`) in `3` of type `Int`.

Note: `resN` is named so in order to be used in later lines, e.g `res0 * 42`
can be used later on and would result in `126`.

Some more things to note:
- `Int` corresponds to a Java `int` and is located in the `scala` package.
- Packages in Scala are similar to the packages in Java
- `scala.Boolean` maps to Java's `boolean`, `scala.Float` to
Java's `float` etc.
- The Scala compiler optimizes these types by using Java's primitive types when
    necessary

## Defining variables

- Scala has two types of variables: `var`s and `val`s
- `val` is basically a `final` variable in Java
- `var` is a casual variable that can be reassigned

Example: `val foo = "foo"` can not be reassigned, whereas `var bar = "bar"`
    can

Note: the type is missing here, this illustrates *type inference*, Scala
figures out the types, so the first statement is equivalent to `val foo: java.lang.String = "foo"`

## Defining functions

Here's how you define a classical `max` function as seen in math textbooks:

```scala
def max(x: Int, y: Int): Int = {
    if (x > y) x
    else y
}
```

Things to note:
- Type annotations **follow** every function parameter
- The last `Int` type annotation defines the *result type* of the function.
    Psst, in Scala it's called *result* type, not *return* type.
