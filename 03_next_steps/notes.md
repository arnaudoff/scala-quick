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
