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
case class UnaryOperator(operator: String,
    argument: Expression) extends Expression {}
case class BinaryOperator(operator: String, left: Expression,
    right: Expression) extends Expression {}
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

- `UnaryOperator("-", UnaryOperator("-", <expression>))` should return
*<expression>*
- `BinaryOperator("+", <expression>, Number(0))` should return
*<expression>*
- `BinaryOperator("*", <expression>, Number(1))` should return
*<expression>*

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

