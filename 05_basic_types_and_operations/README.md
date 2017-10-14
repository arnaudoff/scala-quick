# Basic types and operations

In this lecture we'll be focusing on Scala's fundamental types and operations.
Although they are similar to Java's, there's some interesting differences so the
below material is worth looking at. We'll also look at implicit conversion, a
very powerful Scala-specific feature.

## Some basic types

- The types `Byte`, `Short`, `Int`, `Long`, `Char` are called *integral types*
- The integral types plus `Float` and `Double` are called *numeric types*

Ranges:
- `Byte`: 8-bit signed
- `Short`: 16-bit signed
- `Int`: 32-bit signed
- `Long`: 64-bit signed
- `Char`: 16-bit unsigned, unicode character
- `String`: a sequence of `Char`s
- `Float`: 32-bit IEEE 754 single-precision float
- `Double`: 64-bit IEEE 754 double-precision float
- `Boolean`: `true` or `false`

- All of these types reside in the `scala` package, except for `String`, which
resides in `java.lang`
- Therefore the full name of the type X is `scala.X`, but since all members of
package `scala` and `java.lang` are automatically imported into every Scala
source file, only the type names can be used
- The Scala compiler transforms instances of Scala value types, such as `Int` or
`Double`, down to Java primitive types in the resulting bytecode

## Literals

All of the basic types listed above can be expressed with literals.

### Integer literals

- Integer literals for the types `Int`, `Long`, `Short` and `Byte` come in three
forms: decimal, hexadecimal, and octal
- If the number begins with 0x or 0X, it's hexadecimal (or base 16)

```scala
val hexMeme = 0xcafebabe
```

- If a number begins with a zero, it's octal (base 8)

```scala
val foo = 042
```

- Note that if you print these, they'll be decimal (base 10)!
- If a number begins with a non-zero digit, and is otherwise not annotated in
any way, it's decimal (base 10)
- If a literal ends in an `L` or `l`, it's a `Long`, otherwise it's an `Int`

```scala
val hexMeme = 0xCAFEBABEL
```

### Floating point literals

- Floating point literals are made up of decimal digits, optionally with a
decimal point, and optionally followed by an `E` or `e`, denoting an
an exponent

```scala
val foo = 1.2345
```

```scala
val foo = 1.2345e2
```

where the latter means 1.2345 multiplied by 10^2.

- If a floating-point literal ends in an `F` or `f`, it's a `Float`, otherwise
it's a `Double` by default

```scala
val bar = 1.2345F
```

- Optionally, a `Double` floating-point literal can be denoted with a `D` or `d`
in the end

```scala
val funny = 3e5D
```

### Character literals

- Character literals are composed of a Unicode character between single quotes

```scala
val a = 'A'
```

- It's useful to know that in literals, `\"` is a double quote, `\'` is a
single quote, and `\\` is a backslash.
- Put simply, a backslash is used for escaping special chars.

```scala
val backslash = '\\'
```

### String literals

- A string literal is composed of characters surrounded by double quotes

```scala
val hello = "hello"
```

- In order to avoid awkward strings with escape sequences, Scala includes
*raw strings*
- Raw strings start with `"""`, end with `"""` and can contain any characters
whatsoever inbetween.

### Symbol literals

- A symbol literal is written `'foo`, where `foo` can be any alphanumeric
identifier
- These literals are mapped to `scala.Symbol`
- Symbol literals are typically used where you would just use an identifier in a
dynamically typed language, e.g. a method that updates a record in a database:

```scala
def updateRecordByName(r: Symbol, value: Any) {
  // r.name refers to the name
}

updateRecordByName('age, 15)
```
- Symbols are interned
