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
