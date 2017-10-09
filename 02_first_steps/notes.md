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
- The `=` sign that precedes the body of the function hints in functional style
that the function defines an expression that results in a value
(the `if` here results in one, similar to Java's ternary operator)

Also:

- In the case of `max`, you may leave the result type and the compiler will
infer it
- Typically recursive functions require explicit result type annotation
- If a function has only one statement, you can remove the curly braces:

```scala
def max(x: Int, y: Int) = if (x > y) x else y
```

You can also define void functions, or as they're called in Scala, functions
with result type of `Unit`:

```scala
def greet() = println("foo bar baz")
```

Obviously, methods with result type of `Unit` are executed for their side effects
only.

## Writing scripts

Scala can be used not only for large systems, but for the day-to-day scripting
tasks.

- Scripts are executed by calling `$ scala <filename>.scala`
- Command line args are available in the `args` array
- Arrays in Scala are accessed with parens, e.g. `args(0)` and NOT `args[0]` like
    in Java
- Comments are C-style, e.g. `//` works for a single line, `/* */` is multi-line

Example: To print the arguments given to a script, one could do something like
that:


```scala
args.foreach((arg: String) => println(arg))
```

Things to note:
- This is a good example of a functional style of programming
- You're passing a *function literal* to the `foreach` method
- Function literals' syntax typically looks like this `(x: Int, y: Int) => x + y`: function parameters in parens, a right arrow and function body

Also, the type of `arg` can be inferred to a `String`. So, we can shorten:

```scala
args.foreach(arg => println(arg))
```

And you can even further shorten this by using the fact that if a function literal
consists of only one statement that takes a single argument, you can omit the
argument, so the above line becomes:

```scala
args.foreach(println)
```

Kickass.
