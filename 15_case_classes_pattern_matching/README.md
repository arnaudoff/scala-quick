# Case classes and pattern matching

- This lecture presents case classes and pattern matching as a tool, twin
constructs that come in handy when writing regular,
non-encapsulated data structures
- These two constructs are particularly helpful for tree-like recursive data
- Basically, case classes are Scala's way to allow pattern matching
on objects without requiring much boilerplate code
- All one needs to do to make a class "pattern matchable" is add a single
`case` keyword before the definition of the class

## Example

- Suppose you're writing a library that manipulates some
arithmetic expressions
- First, we have to define the input data
- Our formal language will consist of variables, numbers, unary and
binary operators

Let's try to describe it:

```scala
abstract class Expression
case class Variable(name: String) extends Expression {}
case class Number(number: Double) extends Expression {}
case class UnaryOperator(operator: String, argument: Expression) extends Expression {}
case class BinaryOperator(operator: String, left: Expression, right: Expression) extends Expression {}
```

Using these `case` prefixes allows Scala to add some "syntactic sugar"
for the times when you use your class:

- It adds a factory method with the name of the class, e.g. `val var =
Variable("x")`, which means you can avoid the annoying `new Variable("x")`;
particularly convinient when you have to nest the initializations like
that `val op = BinaryOperator("+", Number(1), var)` - no `new`'s
cluttering the code
- All arguments in the parameter list of a case class get implicitly `val`
prefix, so they are made fields automatically
- The compiler adds some default implementations of `toString`, `hashCode`
and `equals`

The biggest advantage, apart from these syntactic conveniences, is the
ability to pattern match on such classes.


## Pattern matching

### Pattern matching with simplification rules

Consider the above example and suppose we want to simplify arithmetic
expressions of the kinds we just defined; for example let's think of the
following tree simplification rules:

- `UnaryOperator("-", UnaryOperator("-", <expression>))` should return *<expression>*
- `BinaryOperator("+", <expression>, Number(0))` should return *<expression>*
- `BinaryOperator("*", <expression>, Number(1))` should return *<expression>*

Using pattern matching, we can write the following functions:

```scala
def simplify(expression: Expression): Expression = expression match {
  case UnaryOperator("-", UnaryOperator("-", e)) => e
  case BinaryOperator("+", e, Number(0)) => e
  case BinaryOperator("*", e, Number(1)) => e
  case _ => expression
}

simplify(UnaryOperator("-", UnaryOperator("-", Variable("x"))))
```

- `match` corresponds to a switch in Java, but it's written after the
selector expression, which has the following general form:

```scala
selector match { alternatives }
```

- Each alternative includes a pattern and one or more expressions
which will be evaluated if the pattern matches
- The arrow symbol `=>` separates the pattern from the expressions
- As intuition suggests, a `match` expression is evaluated by trying each
of the patterns in the order they are written; the first one that matches
is selected and the part after the arrow is executed

There's also multiple types of patterns:
- A *constant pattern*, for example `1`, `2` etc., matches values that are
equal to the constant
- A *variable pattern*, like `e`, matches every value, and you can refer
to it in the right hand side of the case clause
- A *wildcard pattern*, `_`, matches every value, but it does not introduce
a variable name to refer to that value
- A *constructor pattern*, like `UnaryOperator("-", e)`, matches all
values of type `UnaryOperator`, whose first argument matches `"-"` and
whose second argument matches `e`; the arguments to the constructor are
themselves patterns, which allows for nesting such as `UnaryOperator("-",
UnaryOperator("-", e))`

The above implemented with type checks and type casts, with boring if
statements or maybe using Visitor, would be a sad experience.

### `match` vs `switch`

- `match` expressions can be interpreted as a generalization of Java's
switch statements
- Actually, a Java-style switch can be expressed as a `match` expression,
where each pattern is a constant and the last pattern may be a
wildcard (which is basically the default value)

There's three fundamental differences one needs to remember:
- `match` is an expression is Scala (always results in a value)
- Scala's alternative expressions never fall through into the next case
- If none of the patterns match, an exception named `MatchError` is
thrown (so you **have** to cover all cases!)

Example:

```scala
expression match {
  case BinaryOperator(op, left, right) => println(expression + " is bop")
  case _ => // necessary, because otherwise would throw MatchError for every
  non-binary operator
}
```

## Kinds of patterns

Let's examine each of the kinds of patterns in detail.

### The wildcard pattern

- The wildcard pattern (`_`) matches any object, it's a sort of "catch-all"
alternative
- Wildcards can also be used to ignore parts of an object that you don't
care about

Example:

```scala
expression match {
  case BinaryOperator(_, _, _) => println(expression + " is bop")
  case _ => println("sth else")
}
```

### Constant patterns

- Any literal must be used as a constant, e.g. `33`, `true` and ``"foobar"``
- A constant pattern matches only itself
- Also, any `val` or singleton object can be used as a constant
(e.g. `Nil` matches only the empty list)

```scala
def foo(x: Any) = x match {
  case 33 => "33"
  case true => "1"
  case "foo" => "foo"
  case Nil => "empty list" // !!
  case _ => "sth else"
}
```

### Variable patterns

- A variable pattern matches any object, but unlike the wildcard Scala
binds the variable to whatever the matched object is
- Therefore, you can act on the object further

```scala
expression match {
  case -1 => "-1"
  case whatever => "some value matched: " + whatever
}
```

### Constructor patterns

- A constructor pattern, as already presented, looks something like
    `BinaryOperator("+", e, Number(0))`
- It consists of a name (`BinaryOperator`) and then a number of patterns
within parentheses
- These extra patterns mean that Scala patterns support deep matches: they
not only check the top-level object supplied, but also the contents of
the object against further patterns
- The extra patterns can be constructor patterns as well

```scala
expression match {
  case BinaryOperator("+", e, Number(0)) => println("deep match!")
  case _ =>
}
```

### Sequence patterns

- You can match against sequence types, e.g. `List` and `Array`; any
number of elements can be specified within the pattern

For example:

```scala
expression match {
  case List(0, _, _) => println("starts with a zero")
  case _ =>
}
```

matches a three-element list starting with zero, and

```scala
expression match {
  case List(0, _*) => println("n-length list")
  case _ =>
}
```

matches any lists that starts with zero, regardless of its length.

### Tuple patterns

- We can match against n-tuples too. For example:

```scala
def tuple(expr: Any) = expr match {
  case (x, y, z) => println("matched x y z tuple")
  case _ =>
}
```

### Typed patterns

- Can be used as an elegant replacement for type tests/casts

```scala
def someGenericSize(x: Any) = x match {
  case str: String => str.length
  case map: Map[_, _] => map.size // this is called a type pattern
  case _ => -1
}

someGenericSize("foo")
someGenericSize(Map(1 -> 'a', 2 -> 'b'))
```

An uglier version of the above would be

```scala
if (x.isInstanceOf[String])
  val s = x.asInstanceOf[String]
  s.length
} else // ...
```

- `isInstanceOf` and `asInstanceOf` are predefined methods of `Any`
- Needless to say, rather avoid these - they're made uglier for a reason
- Note that since Scala handles generics like Java (type erasure), you can't
check at runtime for e.g. an `Map[Int, Int]` - all you can do is determine
that a value is a map of some arbitrary type parameters
- The only exception to this rule is arrays:

```scala
def isStringArray(x: Any) = x match {
  case a: Array[String] => true
  case _ => false
}
```

## Sealed classes

- Whenever you pattern match, you need to ensure that you covered all cases
- Sometimes this is done by adding a default case at the end, but
that's when semantically it makes sense to have a default
- In general, it's impossible to track when new classes are added,
e.g. you can add an `Expression` class in the class hierarchy in a
different compilation unit from where the other four cases are defined
- The solution to this is to make the superclass of your case class `sealed`
- A sealed class cannot have any new subclasses added except the ones in the
same file
- This is super useful for pattern matching: it means you only need to worry
about subclasses you already know, and they are in the same file!
- The compiler will also signal missing combinations of patterns with a
warning message

**In general**: If you consider writing a hierarchy of classes
intended to be pattern matched, you should consider sealing the class at
the top of the hierarchy.

```scala
sealed absract class Expression

case class Variable(name: String) extends Expression
case class Number(number: Double) extends Expression
case class UnaryOperator(op: String, arg: Expression) extends Expression
case class BinaryOperator(op: String, lhs: Expression, rhs: Expression) extends Expression
```

Now let's consider a pattern match where we omit some of the possible cases:

```scala
def foo(expr: Expression) = expr match {
  case Variable(_) => "some variable"
  case Number(_) => "some number"
}
```

- The compiler will complain with something like "match is not exhaustive,
missing combination UnaryOperator, BinaryOperator"!
- If you really want to use it that way (you know in advance that the
method will be applied only to either `Number` or `Variable` expression),
consider:

```scala
def foo(expression: Expression) = (expression: @unchecked) match {
  case Number(_) => "some number"
  case Var(_) => "some variable"
}
```

- Basically, you can add an annotation to an expression the same you way
you add a type to it: expression, then colon, and the name of the
annotation, e.g `expression: @unchecked`
- The `@unchecked` annotation has a special meaning for pattern matching;
if a match's selector expression has this annotation, exhausitivity
checking for the patterns that follow will be suppressed.

## The `Option` type

- Scala has a standard type named `Option` which is used for the so called
optional values
- Such a value can have two forms: `Some(x)` where `x` is the actual
value, or `None` object, which represents a missing value
- For example, `Optional` values are produced by some of
the standard operations on Scala's collections, e.g if you have a map,
its `get` method produces `Some(value)` if a `value` corresponding to a
given key's been found, and `None` if the given key is undefined

```scala
val capitals = Map("Bulgaria" -> "Sofia", "Russia" -> "Moscow")

capitals get "Bulgaria" // Option[String] = Some(Sofia)
capitals get "Germany" // Option[String] = None
```

A common idiom that connects pattern matching and optional values is:

```scala
def show(x: Option[String]) = x match {
  case Some(s) => s
  case None => "?"
}
```

So, to display the values:

```scala
show(capitals get "Bulgaria") // Sofia
show(capitals get "India") // ?
```

- Note that Java is inferior compared to Scala in solving this problem:
using `null` to indicate no value; it's error prone as it's difficult in
practice to keep track of which variables are allowed to be null
- Obviously, if a variable is allowed to be `null`, you must always check if
it's `null` when you use it, and when you forget to do so, you basically
allow for `NullPointerException`'s to occur at runtime, and this is
pain to find during testing
- By contrast, Scala encourages the use of `Option` to indicate an optional
value: first, it's obvious to readers that a variable of type `Option`
can be `None` sometimes, and second, using a variable that may be null
becomes a type error (if a variable is of type, say, `Option[String]` and
you use it as a `String`, the program will simply not compile)

## Some other uses of patterns

- Anytime you define a `val` or `var` you can use a pattern instead of
simple identifier
- For example, you can take a tuple and "split" it into two
parts that make up separate variables:

```scala
val someTuple = (1337, "Foo")
val (leetNumber, foo) = someTuple
```
- This is also useful when working with case classes; if you know the
exact case class, you can decompose it with a pattern:

```scala
val expression = new BinaryOperator("*", Number(20), Number(5))

val BinaryOperator(operator, left, right) = expression
```

## Case sequences

- A sequene of cases in curly braces can be used anywhere a function
literal can be used (essentially, a case sequence is a function literal)
- Instead of having a single entry point and list of parameters,
a case sequence has multiple entry points, each with their own list of
parameters
- Each case is an entry point to the function, and the parameters are
specified with the pattern
- The body of each entry point is the right-hand side of the case

```scala
val foo: Option[Int] => Int = {
  case Some(x) => x
  case None => 0
}

foo(Some(33)) // 33
foo(None) // 0
```

- The body of this function has two cases: the first case matches a `Some`,
and returns the number "encapsulated" within the `Some`, the second
matches a `None`, and returns a default value of zero
- The same pattern is used in the Akka's actors library, because it allows
its receive method to be defined as series of cases (case sequence):

```scala
var sum = 0

def receive = {
  case Data(byte) => sum += byte
  case GetChecksum(requester) => {
    val checksum = ~(sum & 0xFF) + 1
    requester ! checksum
  }
}
```

- It's worth noting that a case sequence gives you a partial function, so
if you apply it to a value it does not support, it will generate
a runtime exception
- By partial function we mean partial function in mathematical sense

Let `second` be a function that returns the second element in a list:

```scala
val second: List[Int] => Int = {
  case x :: y :: _ => y
}
```

- The compiler will complain, because this function works for a
three-element list, but not for an empty list, otherwise said we'll have a
match that is not exhaustive

```scala
second(List(1, 2, 3)) // 2
second(List()) // MatchError
```

- So, in order to solve the problem, you must check whether a partial
function is defined; in other words, you must tell the compiler that
you're working with partial functions
- The type `List[Int] => Int` includes all functions from lists of
integers to integers, whether or not they are partial; the type that only
includes the partial functions is called `PartialFunction[List[Int], Int]`

```scala
val second: PartialFunction[List[Int], Int] = {
  case x :: y :: _ => y
}
```

- Now, we can check whether `second` is defined for some `x`:

```scala
second.isDefinedAt(List(5, 6, 7)) // true
second.isDefinedAt(List()) // false
```

- The above function is a **typical example** for a partial function

In fact, the partial function gets translated to something like that by the
compiler:

```scala
new PartialFunction[List[Int], Int] {
  def apply(xs: List[Int]) = xs match {
    case x :: y :: _ => y
  }

  def isDefinedAt(xs: List[Int]) = xs match {
    case x :: y :: _ => true
    case _ => false
  }
}
```

- In general, you should try to use complete functions (or, if we're
mathematically correct, total functions) whenever possible, because partial
functions allow for runtime errors

### Patterns in `for` expressions

- You can also use patterns in `for` expressions
- In the example below, we retrieve all key/value pairs from the
`capitals` map we showed earlier

```scala
for ((country, city) <- capitals)
  println("The capital of " + country + " is " + city)
```

- Obviously, a pattern might not match a generated value; for example:

```scala
val fruitList = List(Some("apple"), None, Some("orange"))
for (Some(fruit) <- fruitList)
  println(fruit)
```

prints only apple and orange.
