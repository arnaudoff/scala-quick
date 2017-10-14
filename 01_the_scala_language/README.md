# A Scalable language

- Scala stands for scalable language, therefore scalability is main design goal
- Applicable from small scripts to complex systems
- Runs on the JVM, interoperates with Java stuff easily
- It's a mix of OO and functional concepts in a statically typed language
- Scala code is concise because of this fusion

## A language that grows on you

Scala gives you the feel of modern "scripting" languages such as Python and
Ruby, e.g this is how the familiar maps/dicts work in Scala:

```scala
var capital = Map("US" -> "Washington", "France" -> "Paris")

capital += ("Japan" -> "Tokyo")

println(capital("France"))
```

Quite intuitive, huh? A map is created, then a new binding is added, and the
capital associated with France is printed. In Scala, however, maps are not
language syntax but library abstractions and therefore can be extended and
adapted, e.g you can use `HashMap` or `TreeMap`. You can even make it thread safe by
mixing in `SynchronizedMap` trait. Psst, a *trait* is the equivalent of an interface
in other OO languages such as Java.

### Growing new types

- Scala is designed to be extended and adapted, rather than to provide all
constructs needed
- In a sense, Scala aims to be the "perfect" language by simply allowing you to
make it perfect yourself
- Example: `scala.BigInt` looks like a built-in type, but it's actually
defined in the standard library as a wrapper for the `java.math.BigInteger`. If
it was missing, could be easily defined

### Growing new control structures

- The same philosophy applies to control structures
- Example: Scala's API for "actor-based" concurrent programming, similar to
Erlang's, which supports constructs such as `recipient ! msg` thanks to Scala's
flexbility: the actors themselves are defined in the library, but do naturally
feel like integral part of the Scala language

## What makes Scala scalable?

- The combination of OO and functional programming
- Scala does not deviate from OO principles, e.g by allowing static fields and
methods or by allowing values that are not objects (such as the primitive
values in Java)
- It encourages immutable data structures and referentially transparent methods;
functions are first-class values, which provides means for abstracting
over operations and provides expresiveness, leading to concise code as well

## Why Scala?

- Compatible. Doesn't require you to step away from the Java platform, it allows
you to add value to existing code without having to do a single change. In fact,
Scala makes use of the Java libraries heavily
- Concise. Just notice the difference between this

```java
class MyClass {

    private int index;
    private String name;

    public MyClass(int index, String name) {
        this.index = index;
        this.name = name;
    }

}
```

and this

```scala
class MyClass(index: Int, name: String)
```

This is by far quicker to write, easier to read, and less error prone.

- High-level. You often see imperative code such as

```java
boolean nameHasUpperCase = false;
for (int i = 0; i < name.length(); ++i) {
    if (Character.isUpperCase(name.charAt(i))) {
        nameHasUpperCase = true;
        break;
    }
}
```

being replaced by poetry such as

```scala
val nameHasUpperCase = name.exists(_.isUpper)
```

- Scala has a very advanced static type system: from a system of nested types,
generics, intersections and abstract types, to type inference. Writing Scala
code feels like writing in a dynamic language, but with all positive aspects of
a static type system

