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
