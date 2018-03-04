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
