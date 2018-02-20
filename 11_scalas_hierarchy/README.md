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

