# Classes and objects

In this lecture, you will learn more about classes, fields and methods in Scala. 

## Classes, fields and methods

- Just like in any OO language, a class is a blueprint for objects
- You can create objects with the `new` keyword

```scala
class Person {
  // class definition
}
```

```scala
new Person
```

- In the class definition, you place fields and methods, collecively
called members
- You can say it's a field if the member is defined with `val` or `var`,
they refer to objects and they hold state (also called *instance variables*)
- Methods are defined with `def`, they contain executable code, hence
they do the "computational" work
- When you instantiate a class, the runtime allocates memory to hold
the object state

Consider the following:

```scala
class Person {
  var age = 16
}

val john = new Person
john.age = 15
```

It's important to understand that since `age` is defined as a `var`, you
can later reassign it, even though `john` is a `val`. In essence, this
means that you can mutate the object, but you can not reassign `john` to
another object of the same type.

- Scala also supports access modifiers just like other languages, e.g.
you can make `age` private by annotating it with `private` before the type
- Unsurprisingly, private fields can be accessed by methods defined in
the same class
- This ensures that the object state remains valid during its lifetime
(because outsiders can't mutate it)
- By default, the access modifier is public, so where you say
"public" in Java, you say nothing in Scala
