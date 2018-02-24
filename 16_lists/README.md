# Lists

In this lecture we'll look at the most commonly used data structure in Scala

## List literals

- Here are some examples of list literals

```scala
val fruits = List("apples", "oranges", "pears")
val numbers = List(1, 2, 3, 4, 33)
val identityMatrix = List(
  List(1, 0, 0),
  List(0, 1, 0),
  List(0, 0, 1)
)
val empty = List()
```

- Lists, although similar to arrays, are immutable (an element cannot be
changed by assignment)
- Lists have recursive structure (like a linked list), while arrays are flat
- Lists are homogenous (every element is of the same type), or, if we
explicitly state the types of the previous example:

```scala
val fruits: List[String] = List("apples", "oranges", "pears")
val numbers: List[Int] = List(1, 2, 3, 4, 33)
val identityMatrix: List[List[Int]] = List(
  List(1, 0, 0),
  List(0, 1, 0),
  List(0, 0, 1)
)
val empty: List[Nothing] = List()
```

- The `List` type is covariant (which means that for each pair of types
`S` and `T`, if `S` is a subtype of `T`, then `List[S]` is a subtype
of `List[T]`)
- For example, `List[String]` is a subtype of `List[Object]`
- Informally, this is so because every list of strings can also be seen as
list of objects
- The empty list has a type `List[Nothing]`, and since lists are covariant
`List[Nothing]` is a subtype of `List[T]` for any `T`. This is why the
following is a valid construct:

```scala
val xs: List[Int] = List()
```

## Constructing lists

- All lists are built from two things: `Nil` and `::` (cons operator)
- `Nil` represents the empty list
- `::` represents list extension at the front, e.g `x :: xs` means a list
whose first element is `x` followed by the elements of the list `xs`

Equivalent definitions of the above examples are the following:

```scala
val fruits = "apples" :: ("oranges" :: ("pears" :: Nil))
val numbers = 1 :: (2 :: (3 :: (4 :: Nil)))
val empty = Nil
```

- In fact, the first definitions actually translate to those, for instance
`List(1, 2, 3, 4)` actually creates the list `1 :: (2 :: (3 :: (4 :: Nil)))`
- Also, the `::` operator is associative so you may omit the parentheses and
write the more readable `1 :: 2 :: 3 :: 4 :: Nil`

## Operations on lists

- `head` - returns the first element of a list
- `tail` - returns a list consisting of all but the first element
- `isEmpty` - returns `true` if the list is empty

Let's look at an implementation of insertion sort in order to show how
lists are processed:

```scala
def isort(xs: List[Int]): List[Int] =
  if (xs.isEmpty) Nil
  else insert(xs.head, isort(xs.tail))

def insert(x: Int, xs: List[Int]): List[Int] =
  if (xs.isEmpty || x <= xs.head) x :: xs
  else xs.head :: insert(x, xs.tail)
```

## List patterns

- There's two ways to match lists: you can either match on all elements of
a list, using a pattern of the form `List(...)` or "decompose" them bit
by bit using patterns composed from the :: operator and the `Nil` operator

Examples:

```scala
val List(a, o, p) = fruits
val a :: o :: rest = fruits
```

- The first of the above patterns matches lists of length 3, and binds the
three elements to the pattern variables
- `a` is a `String` containg "apples", `o` is a `String` containing
"oranges", same for `p`

- The second pattern matches lists of length 2 or more (useful when the
list length in unknown beforehand)
- The last pattern variable (`rest`) is of type `List[String]`

Lists can also be "decomposed" with patterns, just like we did with `head`,
`tail` and `isEmpty`:

```scala
def isort(xs: List[Int]): List[Int] = xs match {
  case List() => List()
  case x :: rest => insert(x, isort(rest))
}

def insert(x: Int, xs: List[Int]): List[Int] = xs match {
  case List() => List(x)
  case y :: ys => if (x <= y) x :: xs else y :: insert(x, ys)
}
```

- Often, pattern matching on lists is clearer than decomposing with the
methods

## The most important `List` methods

- There's a large number of methods defined for common operations over
lists; however, we'll look at the ones that make list processing programs
more concise and clearer

### First-order methods

*Definition*: a method is a *first-order* method, if it doesn't take any
functions as arguments

### Concatenation

- Concatenation is written `:::`, and (unlike `::`!), it takes two lists as
operands and produces a new list; so the result of `xs :: ys` is the list
`zs` such that `zs` contains all elements of `xs` followed by all elements
of `ys`
- Like cons, `:::` associates to the right so `xs ::: ys ::: zs` is
interpreted like `xs ::: (ys ::: zs)`

Some examples:

```scala
List(1, 2) ::: List(3, 4, 5) // List(1, 2, 3, 4, 5)
List() ::: List(1, 2, 3) // List(1, 2, 3)
List(1, 2, 3) ::: List(4) // List(1, 2, 3, 4)
```

Let's try to implement concatenation just to see a technique that involves
pattern matching.

```scala
def append[T](xs: List[T], ys: List[T]): List[T]
```

- To design `append`, it's good to know that it sort of represents the
classical divide and conquer principle (over a recursive data structure:
list in our case)
- We first split the input list into simpler cases using a pattern
match (divide)
- Then, we construct a result for each case, which, if non-empty, may be
constructed by recursive calls of the same algorithm (conquer)

```scala
def append[T](xs: List[T], ys: List[T]): List[T] =
  xs match {
    case List() => ys
    case x :: rest => x :: append(rest, ys)
  }
```

- We pattern match on the first list: if it's an empty list, return the
right list, otherwise split by head and tail, and append the
head recursively

### Length of a list

- The `length` method returns the length of a list
- On lists, length is an expensive operations (needs to traverse the
whole list)
- This is why it makes sense to use `xs.isEmpty` instead of
`xs.length == 0`!

```scala
List(1, 2, 3).length
```

### Accessing the end of a list

- `head` and `tail` have dual operations: `last` and `init`
- `last` returns the last element of a list
- `init` returns all but the last element of a list

```scala
val foo = List(1, 2, 3, 4)
foo.last // 4
foo.init // List(1, 2, 3)
```

- Unlike `head` and `tail`, which both run in constant time, `init` and
`last` traverse the whole list to compute their result
- Thus, it's a good idea to organize data so that most accesses are at the
head of a list, rather than the last element
- When applied to an empty list, they throw an exception

### Reversing a list

- If an algorithm accesses the last element often, it's a good idea to first
reverse the list and work with the result instead
- `foo.reverse` reverses the list `foo`
- Since lists are immutable, reverse also creates a new list instead of
modifiying

All in all, `reverse`, `init` and `last` have some "laws" that are
obvious, but still good to point out:
- `reverse` is its own inverse: `xs.reverse.reverse` equals `xs`
- `reverse` turns `init` to `tail` and `last` to `head`

A sample implementation of `reverse` could be achieved by concatenation:

```scala
def reverse[T](xs: List[T]): List[T] = xs match {
  case List() => xs
  case x :: rest => reverse(rest) ::: List(x)
}
```

Note, however, that this implementation has poor complexity.

