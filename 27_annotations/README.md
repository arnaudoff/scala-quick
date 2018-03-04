# Annotations

- Annotations in a way are similar to comments in that they can be sprinkled
    throughout a program and attached to any variable, method or expression
- Moreover, not only do they add some kind of metadata, but unlike comments they
    have some structure which means they can be processed

## Purpose of annotations

The purpose of annotations is simply to add any metadata: a good example is
adding documentation, which is then parsed by Scaladoc or similar tool. The
important thing is that all of this information is ignored by the compiler.

So, Scala supports a system of annotations: the compiler understands simply the
notion of an annotation, but you can attach semantics to annotations however
you want, which allows any meta-programming tool to define its own annotations.

## Example of annotations

The typical syntax of annotations is as such:

```scala
@deprecated
def oldMethod() = // ..
```

Of course, they are not limited to method definitions: they're allowed on any
kind of declaration or definition, including `val`s, `var`s, `def`s, `class`es
and so on with one rule - the annotation applies to the entire definition or
declaration that follows it.

You can also attach an annotation to an expression: for example, when removing
exhaustiveness checking by the compiler (as shown earlier)

```scala
(expr: @unchecked) match {
  // ..
}
```

Moreover, they are not only limited to the `@foo` syntax: they also support
arguments like that

```scala
@annot(firstExpr, secondExpr, ..., nthExpr)
```

The arguments you can give to an annotation obviously depend on the concrete
annotation class.

### Internal representation

An annotation is simply represented as a constructor call of the annotation
class: `@` is a syntax sugar for a `new` call. Sometimes, however, it's required
to pass an annotation as an argument to another annotation, in which case it's
useful to remember that since annotation arguments are required to be
expressions and an annotation is not an expression, we can use the above fact:

```scala
@foo(@bar) // doesn't work, @bar is not an expression
def baz() = {}
```

```scala
@foo(new bar)
def baz() = {}
```

## Standard annotations

There's a number of predefined annotations.

### Deprecation

Deprecation, as in other languages, lets you remove (say, a method) in some later
versions of your code base, while "telling" your clients that sooner or later
this method will not exists and its better for them to update the module that is
using it.

```scala
@deprecated("use foobar() instead, this will be gone soon")
def badMethod() = // ..
```

Such annotation causes the Scala compiler to show deprecation warnings.

### Volatile

- When you have shared mutable state in concurrent programs, the `@volatile`
    annotation helps in such cases
- It informs the compiler that the variable in question will be shared by
    multiple threads
- Such variables are implemented so that read/writes are slower, but accesses
    from multiple threads behave more predictably

### Binary serialization

- A serialization framework helps you convert objects into a stream of bytes and
    vice versa
- Especially useful when you want to send the objects over the network or save
    them to disk
- Scala does not have its own serialization framework, but provides a few
    annotations that are used by external frameworks

Here are the annotations:

- `@serializable`: indicates whether a class is serializable, by default any
    class is NOT serializable so you should add it explicitly
- `@SerialVersionUID(x)`: deals with serializable classes that change in time;
    you can attach a serial number to the current version and the framework
    stores this number in the byte stream it generates; when the bytestream is
    reloaded as an object, the framework compares the current version of the
    class with the recently deserialized version
- `@transient`: for fields that should not be serialized at all

### Tailrec

You typically attach the `@tailrec` annotation to methods that need to be tail
recursive (if it recurses very deep otherwise); so the compiler knows it should
make the tail recursive optimization.

### Unchecked

As explained multiple times already, `@unchecked` is interpreted by the compiler
during pattern matches and informs the compiler to not warn for `match`
expressions that leave out some cases.
