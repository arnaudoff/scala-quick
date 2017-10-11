# Next steps

This text continues the previous by introducing some more advanced stuff.

## Parameterize arrays with types

- Scala, just like most OO languages, uses `new` for instantiating objects
- Just like in Java or C#, you can parameterize an instance with types and with
values

Example:

```scala
val fruits = new Array[String](3)

fruits(0) = "banana"
fruits(1) = "apple"
fruits(2) = "orange"

for (i <- 0 to 2)
  print(greetStrings(i))
```

- In the example fruits is *parameterized* with the type `Array[String]` and
the value `3`
- The compiler infers the type yet again
- The example also shows a `for` construct, but one would be better off using `foreach` in order to stay functional
- Note again that arrays are accessed with `()` and not `[]`

Important: When you define a variable with `val`, the variable can't be
reassigned, but the object to which it refers can still be changed. So, `fruits`
can't be reassigned to a different array, but the array itself is mutable.

Interesting:
- If a method takes only one parameter, you can omit the params
- `to` in the for expression is surprisingly a method that takes an `Int`
- The code `0 to 2` is transformed to the method call
`(0).to(2)`
- Scala does not have operators, `+`, `-`, `*` and `/` can be used as method
    names, e.g. `1 + 2` invokes the `+` method on the `Int` object `1`, passing
    `2` as a parameter
- `fruits(i)` gets transformed into `fruits.apply(i)`, so in essence element
    access is simply a method call in Scala
- In general, any application of an object to some arguments in parens will be
    transformed to `apply` method call
- Similarly, array assignment such as `fruits(i) = "foo"` is equivalent to `fruits.update(i, "foo")`

In general, Scala treats everything from arrays to expressions as objects with methods, and yet the compiler optimizes the performance overhead in the compiled code.

Typically, one would use a more concise way to initialize the above array:

```scala
val numNames = Array("zero", "one", "two")
```

Here, the type is inferred. Also, behind  the scenes you are calling a factory method named `apply` on the `Array` class, which returns the array.

## Lists

- As noted, arrays are fixed-length mutable objects of the same type
- For immutable sequence of objects, use Scala's `List` class
- Scala's `List` differs from `java.util.List` in that Scala lists are always immutable
- Scala's lists are perfect fit for functional style of programming

Example:

```scala
val oneTwoThree = List(1, 2, 3)
```

Since lists are immutable, operations on them create new lists, e.g. `:::` is used for list concatenation like this:

```scala
val oneTwo = List(1, 2)
val threeFour = List(3, 4)
val oneTwoThreeFour = oneTwo ::: threeFour
```

Another useful operation is `::` (pronounced cons), which prepends to the beginning of a list:

```scala
val twoThree = List(2, 3)
val oneTwoThree = 1 :: twoThree
```

Note: If the method name ends in a colon, the method is invoked on the right operand. Therefore,
in `1 :: twoThree`, the `::` method is invoked on `twoThree`, passing in `1`, like this: `twoThree.::(1)`.

## Tuples

- Tuples are immutable, but unlike lists, they can contain different
types of elements, e.g a tuple can contain an integer and a string at
the same time
- A simple use case of tuples is when you want to return multiple
 heterogenous objects from a method
- Tuple elements are accessed by at dot, underscore, and one-based index
of the element

Example:

```scala
val tuple = (42, "foobar")
println(tuple._1)
println(tuple._2)
```

Notes:
- Scala infers the type of the tuple to be `Tuple2[Int, String]`
- The actual type of the tuple depends on the number of elements and their types

## Sets and maps

- Since Scala allows functional and imperative style, its collections
libraries support both mutable and immutable collections
- For sets and maps, Scala models mutability in the class hierarchy
- The Scala API contains a base trait for sets, and two subtraits: one for
mutable sets and one for immutable sets
- Concrete set classes in the API, such as the `HashSet`, extend either the
mutable or immutable `Set` trait
- Note that in Scala, traits are "extended" or "mixed in", not "implemented"
like Java interfaces (although they are almost identical)

Example (creating a default `Set`):

```scala
var mathSet = Set("Calculus", "Discrete mathematics")
mathSet += "Linear algebra"
```

This creates an immutable `Set` similarly to how lists and arrays are created
(by invoking the `apply` factory method on the `Set` companion object). The type of `mathSet` is inferred to be
`Set[String]`. Note that the default is immutable, so `+=` here yields a new
set with the element added.

If you want a mutable set, you'll need to explicitly import it:

```scala
import scala.collection.mutable.Set

val mathSet = Set("Calculus", "Linear algebra")
mathSet += "Geometry"
```

Sometimes, you may also want an explicit set class. In order to use it, simply
import it and use the factory method on its companion object, e.g. for an
immutable `HashSet`: (Psst, companion objects will be explained in detail later)

```scala
import scala.collection.immutable.HashSet

val hashSet = HashSet("Banana", "Orange")
println(hashSet)
```

## Maps

- As with sets, Scala provides mutable and immutable versions of `Map`,
using a class hierarchy
- There's a base `Map` trait in the `scala.collection` package and
two subtrait Maps: a mutable Map in `scala.collection.mutable` and an
immutable one in `scala.collection.immutable`
- Maps are also initialized with the `apply` factory method, similar to how
arrays, lists and sets are initialized

Example (a mutable map):

```scala
import scala.collection.mutable.Map

val treasureMap = Map[Int, String]()
treasureMap += (1 -> "Go to island.")
treasureMap += (2 -> "Find big X on ground.")
treasureMap += (3 -> "Dig.")
println(treasureMap(2))
```

Notes:
- When you say `1 -> "Go to island."`, you are calling a method named
`->` on an integer with the value `1`, passing in a string with the
value ``"Go to island."``
- In general, the `->` method can be invoked on any object in any Scala
program, returns a two-element tuple containg the key and value
- The tuple is passed to the `+=` method of the map object
- The last line prints the value of the key `2`
- As with sets, the immutable variant of a map is the default
- The mechanism that allows `->` to be invoked on any object is
 called *implicit conversion* and is covered later

 Example (here the map is immutable by default):

```scala
val romanNumeral = Map(1 -> "I", 2 -> "II", 3 -> "III", 4 -> "IV", 5 -> "V")
println(romanNumeral(4))
```

- Note the missing explicit type parameterization here: `[Int, String]` is
inferred by the compiler from the values passed to the map factory
