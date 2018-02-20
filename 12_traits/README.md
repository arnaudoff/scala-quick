# Traits

- Way to reuse code in Scala
- Look similar to interfaces in other languages
- Unlike class inheritance, where each class must inherit from only
one superclass, a class can mix in any number of traits
- There's two common ways to use traits: enriching interfaces and defining
stackable modifications
- We'll also see the `Ordered` trait as a concrete example

## Example of a trait

```scala
trait Drivable {
  def drive() = {
    println("I'm driven!")
  }
}
```

- Once a trait is defined, it can be mixed in to a class using `either` or `with` keywords
- Once you extend it, you implicitly inherit its superclass, which is
unsuprisingly `AnyRef`

For instance:

```scala
class Car extends Drivable {
  override def toString = "vw passat"
}

val passat = new Car
passat.drive()

val someCar: Drivable = passat
someCar.drive()
```

- If a class extends a superclass, instead of `extends` you use multiple
with statements to mix in the traits, e.g.

```scala
class Animal

class Horse extends Animal with Drivable {
  override def toString = "foo"
}
```

```scala
class Animal
trait Purchasable

class Horse extends Animal with Purchasable with Drivable {
  override def toString = "foo"

  override def drive() = {
    println("im not driven, im riden")
  }
}
```

We said that traits are similar to Java interfaces and that's obvious,
but traits are actually more similar to classes in Scala: they can declare
fields, maintain state, etc., except for two things:

- A trait cannot have class parameters
- Whereas in classes `super` is statically bound, in traits `super`
calls are dynamically bound so writing `super.toString` in a class
we know which method will be invoked, while in a trait the method
is determined each time the trait is mixed into a concrete class
- This behaviour of super is the foundation of the concept called
*stackable modifications*
