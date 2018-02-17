# Composition and inheritance

- We'll be looking at composition vs inheritance this chapter
- As known, composition means one class holds a reference to another
- Inheritance is the classical superclass/subclass relationship

This whole lecture we will be writing a library for building and rendering
two-dimensional layout elements, where each element is a rectangle that can be
filled with text. The library will provide factory methods that construct new
elements from passed data, e.g `elem(s: String): Element`. We'll also have two
ways to build the elements, using `above` or `beside` (which we call combinators),
e.g

```scala
val firstColumn = elem("foo") above elem("bar")
val secondColumn = elem("bar") above elem("baz")
firstColumn beside secondColumn
```

which would result in

```
foo bar
bar baz
```

## Abstract classes

- Since elements are two dimensional rectangles of text, the member`contents`
will hold it; moreover it will be an array of strings, where each string is a
line

```scala
abstract class Element {
  def contents: Array[String]
}
```

- Note that `contents` is a method that has no implementation and therefore
we say that it's an *abstract* member of the class (no explicit abstract
modifier is required)
- We say that `Element` *declares* `contents`
- A class with abstract members must itself be declared abstract
- As a result, just like in other languages, you can't instantiate it

Now, let's improve on our class:

```scala
abstract class Element {
  def contents: Array[String]
  def height: Int = contents.length
  def width: Int = if (height == 0) 0 else contents(0).length
}
```

- Note that none of the three methods has a parameter list, e.g. instead of
`def width(): Int` we use `def width: Int`
- Such methods are called *parameterless* and are quite common
- By contrast, we say that a method is an *empty-paren method* if its defined
with empty parentheses, such as `def width(): Int`

What's the difference?

- By convention, parameterless methods are used whenever the method accesses
mutable state only by reading fields of the containg object and does not
change mutable state

In the example above, we could well change `width` and `height` to fields, instead
of methods by changing the `def` in each definition to a `val`, and they will be
equivalent.
