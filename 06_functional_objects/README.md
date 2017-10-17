# Functional objects

In this lecture, we will be designing a class in functional style which
models rational numbers. We'll also show more features related to OOP in Scala
such as class parameters, constructors, methods and operators, overriding,
preconditions, overloading and self references.

## Notes on the design of the `Rational` class

Quick math revision:

- A rational number is a number that can be expressed as a ratio `p/q`,
where `p` and `q` are integers for `q` != 0
- `p` is called the numerator, and `q` is called the denominator
- Some examples are `1/2`, `2/3`, `112/239` and `2/1`
- With rational numbers, fractions are represented exactly, without rounding
- Intuitively, in maths, rational numbers are immutable by design, e.g. adding
one rational number to another results in a new rational number. (the
original numbers are unchanged)

The class we will model should support rational numbers to be:

- Added (using a common denominator: e.g. to add `1/2` and `2/3`, both parts
of the left operand are multiplied by `3` and both parts of the right operand
by `2`, which gives `3/6` + `4/6`, or `7/6`)
- Substracted
- Multiplied (multiply their numerators and denominators: e.g. `1/2` * `2/5`
gives `2/10`, or `1/5`)
- Divided (by swapping the numerator and denominator of the right operand: e.g.
`1/2` / `3/5` is equivalent to `1/2` * `5/3`, or `5/6`)

## A note on immutable objects

- Immutable objects are often easier to reason about: they don't have complex
state that changes over time
- You can pass immutable objects around freely, while mutable objects require
defensive copies before passing them
- There's no way for two threads that concurrently access an immutable object
to corrupt its state once it's been properly constructed, because no thread can
change it
- Immutable objects make safe hash table keys, while mutable objects can be
mutated after they are placed into a hash table, meaning they may not be
found the next time he hash table is accessed
- Immutable objects, however, have a disadvantage: they sometimes require that a
large object graph is copied, where an update could be done in place, which
might cause a performance bottleneck
- Often libraries deal with the latter issue by providing mutable alternatives
to immutable classes, e.g. `StringBuilder` is a mutable alternative to the
immutable `String`

## Constructing a `Rational`

- Given we've decided to model `Rational` so that its object are immutable,
we'll require that clients provide all data needed by the object (a
numerator and a denominator) upon instance construction:

```scala
class Rational(p: Int, q: Int)
```

- `p` and `q` are called *class parameters*
- The Scala compiler gathers the class parameters and builds a *primary
constructor* that takes the same two parameters

