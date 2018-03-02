# Mutable objects

So far we've praised immutable objects, however, it's also possible to
define objects with mutable state. They often show up when yo model
objects in the real world that change over time.

## What makes an object mutable

- The difference between a purely functional object and a mutable one
without even looking at the implementation
- When you invoke a method or access a field on some purely functional
object, you will notice that you always get the same result; for example:

```scala
val chars = List('a', 'b', 'c')
```

Applying `chars.head` will always return `a`. And this should be the case
even if there's an arbitrary number of operations on the list between
the point of definition and the point where the access of its head is made.

- For a mutable object, the result of a method call or field access is
dependent on what operations were previously performed on the object

Good example for such object can be a bank account:

```scala
class BankAccount {
  private var currentBalance: Int = 0
  def balance: Int = currentBalance
  def deposit(amount: Int) = currentBalance += amount
  def withdraw(amount: Int): Boolean =
    if (amount > currentBalance) false
    else {
      currentBalance -= amount
      true
    }
}
```

The point is, even if you don't know the implementation of `BankAccount`,
you can tell that `BankAccount` is mutable:

```scala
val account = new BankAccount
account deposit 100
account withdraw 80 // true
account withdraw 80 // false
```

because the two final withdrawals in the previous example returned different
results, although they are the same. Clearly, the bank accounts have mutable
state because the same operation can return different results at different
times.

Also, you could tell from the `var`: mutability and `var`s usually go
together, although having `var`s does not necessarily mean a class is
mutable and vice versa - a class could have no vars but still be mutable
(could defer calls to another mutable object)

## Reassignable variables and properties

- A reassignable variables has two operations: get a value and set a value,
    sometimes in other languages these are implemented via the classical
    getter and setter methods
- In Scala, however, every `var` that is a not a private member of some
object implicitly defines a getter and setter method
- The getter of the variable `x` is named just `x`, while its setter is
named `x_=`

For example, if a `var` definition such as

```scala
var foo = 33
```

appears in a class, the compiler generates a getter, `foo`, a setter,
`foo_=`, in addition to a reassignable field, which is always marked
`private[this]`, which means it can be accessed only from the object that
contains it.

- Considering this expansion of `var`s into getters and setters, it's
worth to note that you can also choose to define them directly, instead
of relying on the compiler generation
- By expanding them manually, you have the flexibility to interpret
variable assignment and variable access however you like (e.g you can
enforce an invariant, notify subscribers each time a variable is modified
etc.)

Example (enforcing an invariant):

```scala
class Time {
  private[this] var h = 12
  private[this] var m = 0

  def hour: Int = h
  def hour_=(x: Int) = {
    require(0 <= x && x < 24)
    h = x
  }

  def minute: Int = m
  def minute_=(x: Int) = {
    require(0 <= x && x < 60)
    m = x
  }
}
```
