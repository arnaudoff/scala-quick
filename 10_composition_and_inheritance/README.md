# Composition and inheritance

- We'll be looking at composition vs inheritance this lecture
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

- In summary, you should never define a method that has side-efects without
parentheses. Similarly, when invoking a function that has side-effects, always
include the empty parentheses
- A good way to rememeber these two principles is to just use `()` if the
function you're invoking performs an operation, otherwise (if it simply
provides property access), leave off the parentheses

## Extending classes

- As explained, we can't instantiate `Element` because it's abstract, but we can
create a subclass that extends `Element` and implement the abstract `contents`
method

```scala
class ArrayElement(conts: Array[String]) extends Element {
  def contents: Array[String] = conts
}
```

- By using `extends` you instantly: 1) inherit all non-private members of
`Element` 2) make `ArrayElement` a subtype of `Element`
- We say that `ArrayElement` is a subclass of `Element`, and `Element` is a
superclass of `ArrayElement`

One may ask what's the "default" base superclass? It's `scala.AnyRef`, which is
pretty much an equivalent of `java.lang.Object`. Therefore, `ArrayElement`
somehow implicitly extends `AnyRef`.

Inheritance is similar in Scala to other languages in that:
- Private members of the super-class are not inherited in a subclass
- A member of a superclass is not inherited if a member with the same name and
parameters is already implemented in the subclass (the member of the subclass *overrides* the member of the superclass)
- If the member in the subclass is concrete and the member of the superclass
is abstract, the concrete member *implements* the abstract one

Another useful concept is *subtyping*, where we can use the subclass instead of
the superclass (when it is required), e.g:

```scala
val element: Element = new ArrayElement(Array("hello"))
```

- By the way, as you probably noticed, the relationship between `ArrayElement` and
`Array[String]` is a composition

