# Packages and imports

- Large programs typically require loose coupling and one way to do this is to
    write in modular style
- A program is divided into a number of smaller modules each of which has an
    inside (implementation) and an outside (interface)
- This lecture shows constructs that help to program in a modular style

## Putting code in packages

So far, all code shown was put in the `unnamed` package. You can place code into
named packages, though. First, you can place a whole file into a package by
simply putting a package clause at the top of the file

```scala
package app.models
class Car
```

or, you can use *packaging* syntax where the package name is followed by
a section in curly braces that contains the definitions from the package:

```scala
package app {
  package models {
    // app.models
    class Car

    package tests {
      // app.models.tests
      class CarSuite
    }
  }
}
```

## Imports

- Packages and their members can be imported
- Imported items can be accessed by simple names, without needing to write the
    whole qualifier, e.g. `File` instead of `java.io.File`

Consider the following class:

```scala
package basement

abstract class Fruit(val name: String, val color: String)

object Fruits {
  object Apple extends Fruit("apple", "red")
  object Orange extends Fruit("orange", "orange")
  object Pear extends Fruit("pear", "green")
  val all = List(Apple, Orange, Pear)
}
```

Now, we have the following `import` statements available:

```scala
import basement.Fruit
import basement._ // access to all members of the basement
import basement.Fruits._ // access to all members of fruits
```

The first of these corresponds to Java's single type import, the second to
Java's on-demand import (in Scala, it's just `_` instead of `*`), the third
corresponds to Java's import of static class fields.

It's worth to note that imports can appear anywhere, not just at the beginning
of a compilation unit. For instance,

```scala
def showFruit(fruit: Fruit) = {
  import fruit._
  println(name + "s are " + color) // equivalent to fruit.name and fruit.color
}
```

Besides, Scala's `import`s are more flexible in two more things:
- They may refer to objects (singleton or regular)
- Let you rename and hide some of the imported members (import selector clause)

```scala
import Fruits.{Apple, Orange} // imports only Apple and Orange from Fruits
import Fruits.{Apple => Foo, Orange} // imports Apple as Foo
import java.{sql => S} // imports java.sql as S, so you can write S.Date etc.
import Fruits.{Pear => _, _} // imports all members of Fruits except Pear
```

In a sense, renaming something to `_` means hiding it and is traditionally
useful to avoid ambiguities, say

```scala
import Notebooks._
import Fruits.{Apple => _, _}
```

gets just the notebooks that are `Apple` and not the fruit.

- Can import packages themselves, not just their non-package members

```scala
import java.util.regex // makes regex usable as a simple name

class Foo {
  val pattern = regex.Pattern.compile("foo")
}
```

## Implicit imports

As somewhere mentioned, the compiler adds some imports implicitly to every
program. These imports are the following:

```scala
import java.lang._
import scala._
import Predef._
```

## Access modifiers

- Members of objects, classes, or packages can be labeled with the access
    modifiers private and protected
- These modifiers restrict access to the members of certain regions of code

### Private members

- Treated similarly to Java: a member labeled private is visible only inside the
    class or object that contains the member definition; in Scala this rule also
    applies for inner classes

```scala
class Outer {
  class Inner {
    private def foo() = {}
    class InnerMost {
      foo() // OK
    }
  }
  (new Inner).foo() // foo is not accessible
}
```

### Protected members

- A `protected` member is only accessible from subclasses of the class in which
    the member is defined

```scala
package foo {
  class Base {
    protected def foo() = {}
  }

  class Bar extends Base {
    foo()
  }

  class Baz {
    (new Base).foo() // doesn't compile, inaccessible
  }
}
```

### Public members

Scala has no explicit modifier for public members: any member that is not
`private` or `protected` is public by default and can be accessed from anywhere.

### Scope of protection

- Access modifiers can be given qualifiers: a modifiers of the form `private[X]`
    or `protected[X]` means that access is private or protected up to `X` where
    `X` is a class, object or package
- Qualified access modifiers allow you to express the package private, protected
    or private up to outermost class that exist in Java, but are not possible
    to define with the simple modifiers

```scala
package university

package teachers {
  private[university] class Teacher {
    protected[teachers] def markStudents() = {}
    class Experience {
      private[Teacher] val years = 1
    }
    private[this] var roomNumber = 200
  }
}

package subjects {
  import teachers._
  object Subject {
    private[subjects] val teacher = new Teacher
  }
}
```

In the above example, the `Teacher` class is only visible in all classes/objects
that are contained in the `university` package (`Subject` is part of it).
Otherwise, if the qualifier of the `private` access modifier is directly the
enclosing package, then the effect of the modifier is equivalent to Java's
package private access.

Also, often you'd see code such as `private[this]` - this is to enable access
only from the same object and is thus called *object-private*. This means that
any access must not only be from within the class (say `Teacher` in the example),
but must also be made from the very same instance. Put differently, marking a
member `private[this]` is a guarantee that it will not be seen from other
objects of the same class.
