# Collections

This lecture will be a gentle introduction to Scala's collections
library in the form of a short summary that presents the most useful
and respectively most often used collections

## Sequences

- Sequence types let you work with groups of data lined up in order
- Because the elements are ordered, you can index them (think mathematics)

### Lists

- Perhaps the most important sequence: the immutable linked-list
- Lists support fast addition and removal of items to the beginning of the
list
- However, lists do not provide fast access to arbitrary indexes because the
implementation must perform linear iteration through the list
- It's vital for Scala that the fast addition and removal of
initial elements means that pattern matching works well

Here's some examples of list operations:

```scala
val numbers = List(1, 2, 3)
numbers.head // 1
numbers.tail // List(2, 3)
```

### Arrays

- Arrays allow you to hold a sequence of elements and efficiently access
an element at an arbitrary position, either to get or update the element
- Arrays are zero-based

Here's some examples of arrays:

```scala
val foo = new Array[Int](5) // size is known, elements are yet unknown
val bar = new Array(1, 2, 3, 4, 5)
```

- Arrays are accessed by placing an index in parentheses, not square
brackets

```scala
foo(0) = bar(1)
```

### List buffers

- Class `List` provides fast access to the head of the list, but not to
the end of the list
- One alternative, which avoids reversing the list if you want fast
access, is to use a `ListBuffer`
- A `ListBuffer` is a mutable object which helps to build lists more
efficiently when append operation is required: it provides constant time
append and prepend operations
- Appending is done with the `+=` operator, while prepending is done with
the `+=:` operator
- Once the building is finished, you can obtain a `List` by calling
`toList` on the `ListBuffer`

```scala
import scala.collection.mutable.ListBuffer

val buffer = new ListBuffer[Int]
buffer += 1
buffer += 2
buffer // ListBuffer(1, 2)
buffer +=: 3 // ListBuffer(3, 1, 2)
buffer.toList // List(3, 1, 2)
```

### Array buffers

- An `ArrayBuffer` is like an array, except that you can additionally add
and remove elements from the beginning and end of the sequence
- The new addition and removal operations are constant time on average,
but often require linear time due to implementation needing to allocate a
new array to hold the buffer contents

```scala
import scala.collection.mutable.ArrayBuffer

val buffer = new ArrayBuffer[Int]() // no need to specify length here
buffer += 33
buffer += 35
buffer // ArrayBuffer(33, 35)

buffer.length // 2
buffer(0) // 33
```

- Note that all of the `Array` operations are also available, although a
little slower (because it's a wrapper)

### StringOps

- `StringOps` is a sequence that implements many sequence methods
- Because `Predef` has an implicit conversion from `String` to
`StringOps`, any string can be treaten like a sequence, e.g.:

```scala
def hasUpperCase(s: String) = s.exists(_.isUpper)

hasUpperCase("foo")
```

In the example, the `exists` method is invoked on the string and because
no method named `exists` is declared in `String`, the compiler implicitly
converts `s` to `StringOps`, which has the method.

## Sets and maps

- As explained the collections library offers both mutable and immutable
versions of sets and maps
- By default, when you write `Set` or `Map` you get an immutable object (
that's because they should be preferred over mutable ones)
- The immutable objects are available because they are defined in `Predef`,
which is implicitly imported into every source file

```scala
object Predef {
  type Map[A, +B] = collection.immutable.Map[A, B]
  type Set[A] = collection.immutable.Set[A]
  val Map = collection.immutable.Map
  val Set = collection.immutable.Set
}
```

The `type` keyword here is used to define `Set` and `Map` as aliases for the
longer fully qualified names of the immutable set and map traits. The `val`s
`Set` and `Map`, however, are initialized to refer to the singleton
objects for the immutable `Set` and `Map`

- If you need the mutable variants, you'll need to explicitly import them

```scala
import scala.collecion.mutable
val mutableSet = mutable.Set(1, 2, 3)
```

### Using sets

- The key characteristic of sets is that they will ensure that at most one
of each object as determined by the equality operator will be contained in
the set at any point in time

A classical example is counting distinct words:

```scala
val text = "foo bar baz foo. barbar, baz br foo!"
val wordsArray = text.split("[ !,.]+")

// Create an empty Set (empty is defined in the Set companion object)
val words = mutable.Set.empty[String]

for (word <- wordsArray)
  words += word // adds word to the words Set
```

#### Common methods on sets

| val numbers = Set(1, 2, 3)            | Creates an immutable set                |
|---------------------------------------|-----------------------------------------|
| numbers + 4                           | Add an element to the set               |
| numbers - 4                           | Removes an element from the set         |
| numbers ++ List(4, 5)                 | Adds multiple elements                  |
| numbers -- List(1, 2)                 | Removes multiple elements               |
| numbers & Set(1, 2, 183)              | Takes the intersection of two sets      |
| numbers.size                          | Returns the size of the set             |
| numbers.contains(2)                   | Checks for inclusion                    |
| import scala.collection.mutable       | Imports the mutable variant of a Set    |
| val words = mutable.Set.empty[String] | Creates an empty mutable Set of strings |
| words += "foo"                        | Adds an element                         |
| words -= "foo"                        | Removes an element if its exists        |
| words ++= List("foo", "bar", "baz")   | Adds multiple elements                  |
| words --= List("foo", "bar", "baz")   | Removes multiple elements               |
| words.clear                           | Empties the set                         |

### Maps

- Maps let you associate a value with each element of a set
- Using a map is similar to using an array, except that instead of
indexing with integers you index with key of any type

```scala
val capitalsMap = mutable.Map.empty[String, String]
capitalsMap("Russia") = "Moscow"
capitalsMap("Spain") = "Madrid"

capitalsMap // Map(Russia -> Moscow, Spain -> Madrid)

capitalsMap("Russia") // Moscow
```

A classical example of map usage is counting the number of times a word
occurs in a string:

```scala
def countWords(text: String) = {
  val occurrences = mutable.Map.empty[String, Int]
  for (word <- text.split("[ ,!.]+")) {
    val oldCount =
      if (occurrences.contains(word)) occurrences(word)
      else 0
    occurrences += (word -> (oldCount + 1))
  }
  occurrences
}
```

#### Common methods used on maps

| val occurrences = Map("foo" -> 2, "bar" -> 3) | Creates an immutable map                       |
|-----------------------------------------------|------------------------------------------------|
| occurrences + ("baz" -> 1)                    | Adds an entry                                  |
| occurrences - "foo"                           | Removes an entry                               |
| occurrences ++ List("baz" -> 1, "bam" -> 8)   | Adds multiple entries                          |
| occurrences -- List("baz", "bam")             | Removes multiple entries                       |
| occurrences.size                              | Returns the size of the map                    |
| occurrences.contains("foo")                   | Checks for inclusion                           |
| occurrences("foo")                            | Retrieves the value for a specific key         |
| occurrences.keys                              | Returns an Iterable over the keys of the map   |
| occurrences.keySet                            | Returns the keys as a set                      |
| occurrences.values                            | Returns an Iterable over the values of the map |
| occurrences.isEmpty                           | Indicates whether the map is empty             |
| import scala.collection.mutable               | Imports the mutable variant of a Map           |
| val words = mutable.Map.empty[String, Int]    | Creates an empty, mutable map                  |
| words += ("foo" -> 1)                         | Adds a map entry                               |
| words -= "foo"                                | Removes a map entry                            |
| words ++= List("bar" -> 2, "baz" -> 3)        | Adds multiple map entries                      |
| words --= List("bar", "baz")                  | Removes multiple map entries                   |

### Default sets and maps

- The implementation provided by the default factories that create
sets and maps typically uses a fast lookup algorithm that involes a hash
table

For instance, for mutable sets and maps, we have the following:

|  Collection factory method     | Return type                      |
|--------------------------------|----------------------------------|
| scala.collection.mutable.Set() | scala.collection.mutable.HashSet |
| scala.collection.mutable.Map() | scala.collection.mutable.HashMap |

For immutable sets and maps, the picture is a bit different.

- The class returned by `scala.collection.immutable.Set()` depends on the
number of elements
- For sets with less than five elements, a special class with fixed set
size is used for better performance; otherwise, it returns an implementation
with hash tries

| Number of elements | Return type                         |
|--------------------|-------------------------------------|
| 0                  | scala.collection.immutable.EmptySet |
| 1                  | scala.collection.immutable.Set1     |
| 2                  | scala.collection.immutable.Set2     |
| 3                  | scala.collection.immutable.Set3     |
| 4                  | scala.collection.immutable.Set4     |
| >= 5               | scala.collection.immutable.HashSet  |

- The above statements apply to `scala.collection.immutable.Map()` as well:

| Number of key-value pairs | Return type                         |
|---------------------------|-------------------------------------|
| 0                         | scala.collection.immutable.EmptyMap |
| 1                         | scala.collection.immutable.Map1     |
| 2                         | scala.collection.immutable.Map2     |
| 3                         | scala.collection.immutable.Map3     |
| 4                         | scala.collection.immutable.Map4     |
| >= 5                      | scala.collection.immutable.HashMap  |

### Sorted sets and sorted maps

- Sometimes you may need a set or a map that returns elements in particular
    order
- For this purpose, there exist the `SortedSet` and `SortedMap` traits,
which are backed by `TreeSet` and `TreeMap`, both implemented with a
red-black tree
- The order is determined by the `Ordered` trait, which the element type
should either mix in or be implicitly convertible to (we'll explain what
the latter means in later lecture)
- Note that sorted sets and maps only have immutable variants

Examples:

```scala
import scala.collection.immutable.TreeSet
import scala.collection.immutable.TreeMap

val foo = TreeSet(1, 5, 3, 2, 8, 4) // TreeSet(1, 2, 3, 4, 5, 8)

var bar = TreeMap(3 -> "foo", 2 -> "bar", 4 -> "baz")
bar // TreeMap(2 -> bar, 3 -> foo, 4 -> baz)
bar += (1 -> "hi")
bar // TreeMap(1 -> hi, 2 -> bar, 3 -> foo, 4 -> baz)
```

## Choosing mutable vs immutable collections

- When in doubt, it's better to start with an immutable collection and then
switch over to a mutable collection (because immutable collections are
easier to reason about)
- Besides being easier to reason about, immutable collections can usually
be stored more compactly than mutable collections as long as the element
count is low

## Initializing collections

- As you've seen so far, the most common way to initialize a collection is
to pass the initial elements to a factory method:

```scala
val xs = List(1, 2, 3)
val foo = Set('a', 'b', 'c')
val occurrences = Map("foo" -> 1, "bar" -> 3)
```

- Of course, you can let the compiler infer the types like in the above
example or be explicit:

```scala
import scala.collection.mutable
val foo = mutable.Set[Int](2)
```

### Converting between mutable and immutable sets and maps

- Sometimes you have to convert between the mutable and immutable variant
of a collection
- Here's a technique that accomplishes it fairly straightforward:

```scala
import scala.collection.mutable
val treeSet = TreeSet("foo", "bar", "baz")

val mutableSet = mutable.Set.empty ++= treeSet
val immutableSet = Set.empty ++ mutableSet

val mutableMap = mutable.Map("foo" -> 1, "bar" -> 2)
val immutableMap = Map.empty ++ mutableMap
```

## Tuples

- A tuple combines a fixed number of items together; you can think of
tuples in mathematics
- Unlike an array or list, a tuple can hold objects with different types

```scala
(33, "yes", PreferredDrink)
```

- Tuples can be used a replacement of simple, but data-heavy classes,
as they save the effort of choosing a class name, scope to define it in,
and names for the members
- One common idiom is to return multiple values from a method using tuples:

```scala
def longestWord(words: Array[String]) = {
  var word = words(0)
  var index = 0

  for (i <- 1 until words.length)
    if (words(i).length > word.length) {
      word = words(i)
      index = i
    }
  (word, index)
}

val longest = longestWord("This is totally crazy".split(" "))
```

- To access the i-th element of a tuple, you use the `._i` syntax (which is
    actually a method call)

```scala
longest._1 // totally
longest._2 // 2
```

or, equivalently

```scala
val (word, index) = longest
word // totally
index // 2
```

- All in all, tuples are great when you combine data has no strict
meaning beyond "a x and a y"
- However, whenever the combination has semantics, or needs methods,
it's better to encapsulate it in a class (e.g don't use a 3-tuple instead
of a `Date` class)
