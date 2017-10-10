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
