# Scala's hierarchy

- In Scala, any class automatically inherits from `Any`
- This means that anytime we have an object of any type, we can
call methods defined in `Any` on it
- Scala also defines `Null` and `Nothing`, which we will examine closer later on

## More on the class hierarchy

- The class `Any`, as already explained, is a common superclass of every
class
- It defines the following methods

```scala
final def ==(that: Any): Boolean
final def !=(that: Any): Boolean
def equals(that: Any): Boolean
def ##: Int
def hashCode: Int
def toString: String
```

- In other words, every object can be compared using `==`, `!=` or 
`equals` and hashed with `##` or `hashCode`
- Obviously `==` and `!=` cannot be overriden
- `==` is essentially the same as `equals` (except for Java's boxed numeric
classes, e.g `Integer` and `Long`, so in Scala `new Integer(1) == new Long(1)`, unlike in Java where even though `1 == 1L`, the previous are not equal)
- `!=` is always the negation of `equals`

`Any` has two subclasses itself:
- `AnyVal` (the parent of value classes,
e.g `Byte`, `Short`, `Char`, `Int`, `Long`, `Float`, `Double`, `Boolean`, `Unit`, obviously you can't instantiate any of these with `new` but with literals, this is so because value classes are all defined to be both abstract and final)
- `AnyRef` (the parent class of all reference classes,
basically an alias for `java.lang.Object`)

Note that:
- `scala.AnyVal`'s subclasses do not subclass each other, they have implicit
conversions between different value class types, e.g. `scala.Int` is
automatically widened to an instance of `scala.Long` when required
- also, there exist "booster classes" like `RichInt`, which are used when
we call methods on a simple `Int`, that are not defined there, but defined
in `RichInt`, for example `32 max 33`, `1 until 33` or `1 to 35`

## `Null` and `Nothing`

- At the bottom of the Scala's class hierarchy, there's `Nil` and `Nothing`
- They are special types that handle some interesting corner cases of the
type system

Some things to note about `Null` first:

- `Null` implements a null reference; it's a subclass of every
reference class (so any class that inherits from `AnyRef`)
- `null` value cannot be assigned to value types,
e.g. `val i: Int = null` is invalid

Now, about `Nothing`:
- Nothing is a subclass of every other type, it's really at the
bottom of the bottom of the class hierarchy, it's a subtype of every other type
- There exist no values of this type
- One use of `Nothing` signifies abnormal termination, e.g.

```scala
def error(message: String): Nothing = throw new RuntimeException(message)
```

Since `Nothing` is a subtype of every other type, we can use the `error` we
defined (it's actually defined in the `Predef` object of the Scala standard library) like that, very elegantly:

```scala
def divide(x: Int, y: Int): Int =
  if (y != 0) x / y
  else error("division by zero, algebra broken!")
```

## Defining custom value classes

- For a class to be a value class, it must have exactly one parameter and it
must have nothing except definitions
- No other class can subclass a value class and a value class
- A value class cannot redefine `equals` or `hashCode`

Example:

```scala
class Dollars(val amount: Int) extends AnyVal {
  override def toString() = "$" + amount
}

val dolla = new Dollars(33)
dolla.amount
```

### A good example on the usability of value classes

Consider the following self-explanatory code:

```scala
def title(text: String, anchor: String, style: String): String =
  s"<a id='$anchor'><h1 class='$style'>$text</h1></a>"
```

- It's easy to notice that although the above code is strongly-typed,
it cannot detect the times when you'd syntactically be correct but semantically
wrong, in other words calls like that:

```scala
title("someAnchorId", "color: red", "foo")
```

- A better alternative is to define a new class for each domain concept,
in our case define a "small" class for styles, anchor identifiers,
display text and HTML
- Since these have one parameter and no members, they're a perfect fit
for value classes

```scala
class Html(val value: String) extends AnyVal
class Anchor(val value: String) extends AnyVal
class Style(val value: String) extends AnyVal
class Text(val value: String) extends AnyVal
```

Now, the above code is turned into the below safe code:

```scala
def title(text: Text, anchor: Anchor, style: Style): Html =
  new Html(
      s"<a id='${anchor.value}'>" +
      s"<h1 class='${style.value}'>" +
          text.value +
      "</h1></a>"
  )

title(new Anchor("someAnchorId"), new Style("color: red"), new Text("foo"))
```

This is both type-safe and enforces semantical correctness.
