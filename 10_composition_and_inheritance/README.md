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

## Overriding methods and fields

- Since fields and methods belong to the same namespace, it's possible for a
field to override a parameterless method, e.g change the implementation of
contents from a method to a field without having to modify the abstract method
definition

```scala
class ArrayElement(conts: Array[String]) extends Element {
  val contents: Array[String] = conts
}
```

- So, the field `contents` is a perfectly valid implementation of the
parameterless method `contents` in class `Element`

## Parametric fields

- Consider the definition of `ArrayElement`. There's an obvious code smell: it
has the parameter `conts`, whose whole purpose is to be copied into the
`contents` field. This is annoying in itself.
- Scala solves this problem by parametric fields

```scala
class ArrayElement(val contents: Array[String]) extends Element
```

- Note that the `contents` parameter is prefixed by a `val`
- This prefix is a shorthand that defines a field and a parameter with the same
name
- The field is obviously unreassignable

Put simply,

```scala
class ArrayElement(val contents: Array[String]) extends Element
```

is the equivalent of

```scala
class ArrayElement(foobar: Array[String]) extends Element {
  val contents: Array[String] = foobar
}
```

- If you want the field to be reassignable, just prefix the parameter with `var`
instead of `val`
- You can also prefix with modifiers such as `private`, `protected` etc

For example,

```scala
class Cat {
  val dangerous = false
}

class Tiger(override val dangerous: Boolean, private var age: Int) extends Cat
```

is equivalent to

```scala
class Tiger(someParam: Boolean, anotherParam: Int) extends Cat {
  override val dangerous = someParam
  private var age = anotherParam
}
```

## Invoking superclass constructors

Consider a `LineElement`, which is an abstraction for a layout element that
consists of a single line given by a string:

```scala
class LineElement(s: String) extends ArrayElement(Array(s)) {
  override def width = s.length
  override def height = 1
}
```

- Since `ArrayElement`'s constructor takes a parameter,
`LineElement` simply passes this argument

## Polymorphism and dynamic binding

- A quick OOP refresher: a polymorphic type is a type that can have many forms,
e.g. `Element` can be `ArrayElement` or `LineElement`.

Thus, the following expressions are valid:

```scala
val el: Element = new ArrayElement(Array("foo", "bar"))
val arrayElement: ArrayElement = new LineElement("foobar")
val secondElement: Element = arrayElement
```

- Method invocations on variables and expressions are dynamically bound
- Thus, the actual method implementation invoked is determined at runtime
based on the class of the object, not the type of the variable/expression

This is easier to explain with an example. Consider

```scala
abstract class Element {
  def foo() = {
    println("Element's implementation is invoked")
  }
}

class ArrayElement extends Element {
  override def foo() = {
    println("ArrayElement's implementation invoked")
  }
}

class LineElement extends Element {
  override def foo() = {
    println("LineElement's implementation invoked")
  }
}
```

Now, if you define the following method:

```scala
def invokeFoo(e: Element) = {
  e.foo()
}
```

and pass an `ArrayElement` to it, `ArrayElement`'s implementation will be
invoked, even though the type of `e` is `Element`.

## Declaring final members

- In order to prevent overriding, simply use the `final` modifier

```scala
class ArrayElement extends Element {
  final override def foo() = {
    println("ArrayElement's impl invoked")
  }
}
```

If you try to override it in `LineElement`, you'd get an error.

Similarly, if you want to avoid a class to be subclassed further, it can be
defined as `final`. For example:

```scala
final class ArrayElement extends Element {
  override def foo() = {
    println("ArrayElement's impl invoked")
  }
}
```

## Using composition and inheritance

- Composition and inheritance are two ways to define a new class in terms of an
already existing one
- If you strive for code reuse, you should in general prefer composition to
inheritance
- We can think of inheritance as "is-a" relationship, therefore we may as well
notice that we need to change our definition of `LineElement` and make it a
direct subclass of `Element`

```scala
class LineElement(s: String) extends Element {
  val contents = Array(s)
  override def width = s.length
  override def height = 1
}
```

- Now `ArrayElement` has a composition relationship with `Array`, instead of
inheritance relationship with `ArrayElement`: it holds a reference to an array
of strings

## Implementing the combinators `above` and `beside` and `toString`

- Putting one element above another we can interpret as concatentating the
contents values of the elements, so our method could look something like
that

```scala
def above(that: Element): Element =
  new ArrayElement(this.contents ++ that.contents)
```

- Putting one element beside another can be interpreted as a new element in
which every line is the concatenation of the corresponding lines of the two elements

```scala
def beside(that: Element): Element =
  new ArrayElement(
    for ((firstLine, secondLine) <- this.contents zip that.contents)
  ) yield firstLine + secondLine
```

- `this.contents` and `that.contents` arrays are transformed into an array of
pairs using the `zip` operator, which picks corresponding elements in its two
operands and forms an array of pairs

Example:

```scala
Array(1, 2, 3) zip Array("z", "t") // Array((1, "z"), (2, "t"))
```

We will also need a `toString` method for `Element`, which has a
simple implementation:

```scala
override def toString = contents mkString "\n"
```

Overall, this is how the `Element` class looks now:

```scala
abstract class Element {
  def contents: Array[String]
  def width: Int = if (height == 0) 0 else contents(0).length
  def height: Int = contents.length

  def above(that: Element): Element =
    new ArrayElement(this.contents ++ that.contents)

  def beside(that: Element): Element =
    new ArrayElement(
        for (
            (firstLine, secondLine) <- this.contents zip that.contents
        ) yield firstLine + secondLine
    )

  override def toString = contents mkString "\n"
}
```

## Defining a factory object

- We can hide the already developed hierarchy behind a factory object
- As a reminder, a factory object contains methods that construct
other objects
- Clients would then use these factory methods to construct objects,
rather than constructing them directly with new
- Advantage of this approach is that object creation can be
centralized and the details of how objects are represented with
classes can be hidden
- Initially, we must choose where to locate the factor methods.
- A good solution is to create a companion object of `Element` and make
this the factory object for layout elements.
- That way, only the `Element` object/class need to be exposed to the
client, and not the implementations `ArrayElement` or `LineElement`

```scala
object Element {
  def elem(contents: Array[String]): Element =
    new ArrayElement(contents)

  def elem(line: String): Element =
    new LineElement(line)
}
```

- Also, given these factory methods, we can change the implementations
`ArrayElement` and `LineElement` to be private
because they no longer need to be directly accessed by clients.

Finally, this is the refactored version of `Element`:

```scala
import Element.elem

abstract class Element {
  def contents: Array[String]
  def width: Int = if (height == 0) 0 else contents(0).length
  def height: Int = contents.length

  def above(that: Element): Element =
    elem(this.contents ++ that.contents)

  def beside(that: Element): Element =
    elem(
        for ((firstLine, secondLine) <- this.contents zip that.contents)
        ) yield firstLine + secondLine
    )

  override def toString = contents mkString "\n"
}
```

and the way to hide the implementations of the concrete classes "inside" the
companion object

```scala
object Element {
  private class ArrayElement(val contents: Array[String]) extends Element

  private class LineElement(s: String) extends Element {
    val contents = Array(s)
    override def width = s.length
    override def height = 1
  }

  def elem(contents: Array[String]): Element = new ArrayElement(contents)

  def elem(line: String): Element = new LineElement(line)
}
```

