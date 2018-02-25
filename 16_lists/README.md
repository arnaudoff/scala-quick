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

### List prefixes and suffixes

- The `drop` and `take` operations return arbitrary prefixes or suffixes
 of a list.
- The expression `xs take n` returns the first `n` elements of the list `xs`
- The operation `xs drop n` returns all elements of the list `xs`, except
for the first `n` ones.
- The `xs splitAt n` operation is equivalent to `(xs take n, xs drop n)`,
otherwise said splits the list at a given index returning a pair of two
lists
- The difference, however, is that `splitAt` avoidd traversing `xs` twice!


```scala
val xs: List[Int] = List(1, 2, 3)
xs take 2 // List(1, 2)
xs drop 2 // List(3)
xs splitAt 2 // (List(1), List(3))
```

### Element selection

- Random element selection is supported through the `apply` method;
however it's popular typically for arrays, not for lists
- The reason is that `xs(n)` is O(n), because `apply` is defined as follows:

```scala
(xs drop n).head
```

### Flattening

- The `flatten` method takes a list of lists and flattens it to a single list
- It can be applied only to lists whose elements are all lists

```scala
List(List(1, 2), List(3), List(), List(4, 5)).flatten // List(1, 2, 3, 4, 5)
```

### Zipping

- The `zip` operation takes two lists and forms a list of pairs
- If the two lists are of different length, any unmatched elements are dropped

```scala
val x = List(1, 2, 3)
val y = List('a', 'b', 'c')
val z = x zip y // List((1, 'a'), (2, 'b'), (3, 'c'))
```

- A useful special case is to zip with index, which pairs every list element
with the position where it appears in the list

```scala
val x = List('a', 'b', 'c')
x.zipWithIndex // List(('a', 0), ('b', 1), ('c', 2))
```

- A list of tuples can be changed back to a tuple of lists by using `unzip`

```scala
val zipped = List(('a', 1), ('b', 2), ('c', 3))
val unzipped = zipped.unzip // (List('a', 'b', 'c'), List(1, 2, 3))
```

### Displaying lists

- Calling `toString` on a list returns the classical representation of the list
- For a different representation, use `mkString`, which has the signature of
`xs mkString (prefix, separator, postfix)`: the list to display, the prefix in
front of all elements, a separator string to be displayed between successive
elements and postfix string to be displayed at the end

```scala
val foo = List(1, 2, 3)
foo.mkString("[", ",", "]") // [1, 2, 3]
foo.mkString "" // 123
foo.mkString // 123
```

### Converting lists

- To convert between arrays and lists, use `toArray` and `toList`

```scala
val xs = List(1, 2, 3)
val arr = xs.toArray
val ys = arr.toList
```

## Higher-order methods on lists

We will see how to implement classical functional programming patterns that more
or less appear in day to day work, such as:

- Transforming every element of a list in some way
- Verifying whether a property holds for all elements in a list
- Extracting from a list elements satisfying a certain criterion
- Combining list elements
- ...

### Mapping over lists

- The operation `xs map f` takes a list, and some function f and returns the
list that results from applying the function f to each list element in xs

```scala
List(1, 2, 3) map (_ + 1) // List(2, 3, 4)

val words = List("what", "the", "kcuf")
words map (_.length) // List(4, 3, 4)
words map (_.toList.reverse.mkString) // List("tahw", "eht", "fuck")
```

- The `flatMap` operator is similar to `map`, but it takes a function returning
a list of elements and it applies the function to each list element and
returns the concatenation of all function results

```scala
words map (_.toList) // List(List(w, h, a, t), List(t, h, e), List(k, c, u, f))
words flatMap (_.toList) // List(w, h, a, t, t, h, e, k, c, u, f)
```

One may think of `flatMap` as a composition of `map` and `flatten`

- The third map-like operation is `foreach`; however, it takes a procedure and
applies the procedure to each list element

```scala
var sum = 0
List(1, 2, 3, 4, 5) foreach (sum += _)
```

### Filtering lists

- The operations `xs filter p` takes a list and a predicate and yields
the list of all elements `x` in `xs` so that `p(x)` is `true`; this is similar to how
subsets are formed (set theory)

```scala
List(1, 2, 3, 4, 5) filter (_ % 2 == 0) // List(2, 4)
```

- The `partition` method is like `filter`, but it returns a pair of lists;
the first list in the pair contains the elements for which the predicate is true
and the second list in the pair contains the elements for which the predicate is
false, in other words: `xs partition p` <=> `(xs filter p, xs filter (!p(_)))`

```scala
List(1, 2, 3, 4, 5) partition (_ % 2 == 0) // (List(2, 4), List(1, 3, 5))
```

- The `find` method is similar to `filter` as well, but it returns the first
element satisfying a given predicate; `xs find p` takes a list `xs` and the
predicate `p`, and returns `Some(x)` if there's an element `x` in `xs` for
which p(x) is true, otherwise `None`

```scala
List(1, 2, 3, 4, 5) find (_ % 2 == 0) // Some(2)
```

- The `takeWhile` and `dropWhile` operators also take a predicate; the operation
`xs takeWhile p` takes the longest prefix of list `xs` such that every
element in the prefix satisfies `p`; same goes for `xs dropWhile p`, which
removes the longest prefix from `xs` such that every element in the prefix
satisfies `p`

```scala
List(1, 2, 3, -4, 5) takeWhile (_ > 0) // List(1, 2, 3)
```

- The `span` method combines `takeWhile` and `dropWhile`; it returns a pair of
two lists so that `xs span p` <=> `(xs takeWhile p, xs dropWhile p)` holds

```scala
List(1, 2, 3, -4, 5) span (_ > 0) // (List(1, 2, 3), List(-4, 5))
```

### Predicates over lists

- The operation `xs forall p` is `true` is all elements in `xs` satisfy `p`
- The operation `xs exists p` is true, if there's an element in `xs` that
satisfies p

The above operations map exactly to the respective quantifiers in mathematics,
if we think of lists as sets.

```scala
def hasZeroRow(matrix: List[List[Int]]) =
  m exists (row => row forall (_ == 0))
```

### Folding lists

- Another common operation is to combine the elements of a list with some
operator

```scala
sum(List(a, b, c)) // 0 + a + b + c
```

which could be defined like that

```scala
def sum(xs: List[Int]): Int = (0 /: xs) (_ + _)
```

or, similarly,

```scala
product(List(a, b, c)) // 1 * a * b * c
```

which could be defined like that

```scala
def product(xs: List[Int]): Int = (1 /: xs) (_ * _)
```

Another example: concatenating all words in a list with spaces between them

```scala
("" /: words) (_ + " " + _)
```

- A fold left operation, such as `(z /: xs) (op)` needs three objects to be a
valid operation: a start value `z`, some list `xs` and a binary operation
`op`
- The result of the fold is `op` applied between successive elements of the
list, prefixed by `z`

```scala
((z /: List(a, b, c))(op) // op(op(op(z, a), b), c)
```

- A fold right operation, as an analog, has the opposite way of working and has
the first two operands reversed:

```scala
(List(a, b, c) :\ z) (op) // op(a, op(b, op(c, z)))
```

- To understand why the operators look that strange, try to draw the tree of
op calls over the elements (the comments in the code). Notice that the trees
are left-leaning and right-leaning, respectively.
- For associative operations, fold left and fold right are equivalent

Example: sample implementation of `flatten`

```scala
def flattenRight[T](xss: List[List[T]]) =
  (xss :\ List[T]()) (_ ::: _)
```

- Note that we explicitly state the type of the empty list, because the type
inferencer fails to infer it (more on why that happens later)

Example: list reversal with fold

```scala
def reverseLeft[T](xs: List[T]) =
  (List[T]() /: xs) {(ys, y) => y :: ys}
```

- In order to understand the implementation, simply write a tree and follow the
definitions we gave earlier
- Again, the type annotation is `List[T]()` to make the type inferencer work
- Complexity-wise, this solution is better than the one we showed earlier,
because it applies a constant time operation (the reverse of `::`) `n` times,
where `n` is the length of the argument list; otherwise put, if you imagine
the tree of `foldLeft`, its height is `n`.

### Sorting

- `xs sortWith before` sorts the list `xs` using the `before` function as a
comparison function

```scala
List(1, 3, 2, 8, 4, 5) sortWith (_ < _) // 1 2 3 4 5 8
```

- `sortWith` performs a merge sort

## Methods on the `List` object

- All operations we've seen so far were implemented as methods of class `List`
- We'll now present some of the methods on the `scala.List` object, which is the
companion object of class `List`

### Creating lists from their elements

- A literal `List(1, 2, 3)` is actually the application of the object `List` to
the elements 1, 2 and 3. Otherwise put, it's equivalent to `List.apply(1, 2, 3)`

```scala
List.apply(1, 2, 3) // List(1, 2, 3)
```
