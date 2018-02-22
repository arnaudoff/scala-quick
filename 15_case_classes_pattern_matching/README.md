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
