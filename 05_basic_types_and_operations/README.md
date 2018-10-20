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

- ***(Deprecated)*** If a number begins with a zero, it's octal (base 8)

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

## String interpolation 
- Scala includes a familiar technique for embedding expressions within string
literals called *string interpolation*
- It provides concise and readable alternative to string concatenation

```scala
val name = "reader"
println(s"Hello, $name!")
```

- The expression `s"Hello, $name!` is called a processed string literal, because
the letter `s` precedes the open quote, Scala will use the `s` string
interpolator to process the literal, which will evaluate each embedded
expression, invoke `toString` on each result, and replace the embedded
expression in the literal with those results.
- If the expression includes any non-identifier characters, you must place it in
curly braces

```scala
println(s"The answer is ${6 * 7}.")
```

- The two other string interpolators are `raw` and `f`
- `raw` does not recognise character literal escape sequences, e.g.

```scala
println(raw"No\\\\escape!") // prints No\\\\escape!
```
- `f` allows printf-style formatting, e.g.

```scala
println(f"${math.Pi}%.5f") // prints 3.14159
```

- String interpolation is implemented by rewriting code at compile time
- Users can define other string interpolators for other purposes

## Operators are methods

- As mentioned earlier, Scala's operators are actually nice syntax for
ordinary method calls

```scala
val sum = 1 + 2 // Scala invokes (1).+(2)
```

- In fact, `Int` has several overloaded + methods that take different
parameter types

```scala
val longSum = 1 + 2L // Scala invokes (1).+(2L)
```

- Of course, it's worth to note that operator notation is not limited to
methods like + that look like operators in other languages, you can use
**any** method in operator notation

```scala
val s = "Hello, world!"
s indexOf 'o' // Scala invokes s.indexOf('o')
```

Note that `indexOf` has an overload, which can be called like that:

```scala
s indexOf ('o', 5) // Scala invokes s.indexOf('o', 5)
```

- **Key takeaway**: in Scala operators are not language syntax, any method can
be an operator: it's how you use it that makes it an operator or not.

```scala
s.indexOf('o') // indexOf is NOT an operator, using ordinary method call here
s indexOf 'o' // indexOf is an operator, using operator notation here
```

In Scala, there's three types of operator notation:
- Infix operator notation (binary): the method to invoke sits between the
object and the parameters, as in `7 + 2`
- Prefix operator notation (unary): the method name is placed before the object,
e.g `-7`, `!found`, `~0xFF`
- Postfix operator notation (unary): the method name is placed after the object,
e.g. `7 toLong`

Note:
- Postfix operators are methods that take no arguments when invoked without a dot
or parens. In Scala, empty parens on method calls can be left off.
- By convention, you include the parens if the method has side effects,
    otherwise simply use postfix operator notation:

```scala
val s = "Hello, world!"
s toLowerCase // postfix operator notation (the method requires no arguments)
```

Interesting:
- When transforming operator notation to method call for unary operator,
Scala uses `unary_` prepended to the operator character, e.g `-2.0`
is actually `(2.0).unary_-`
- Only `+`, `-`, `!` and `~` are allowed as prefix operators

## Arithmetic operations

- Scala supports the typical arithmetic methods: addition (`+`), substraction
(`-`), multiplication (`*`), division (`/`) and remainder (`%`)

## Relational and logical operations

- You can compare numeric types with the relational methods greater than (`>`),
less than (`<`), greater than or equal to (`>=`), and less than or equal to
(`<=>`), which have a `Boolean` result type. The unary `!` operator (or as
mentioned, the `unary_!` method) for negating a boolean expression
- Logical methods, logical-and (`&&`) and logical or (`||` and `|`) take boolean
operands in infix notation and have a `Boolean` result type
- The `&&` and `||` operations short-circuit as in other languages (or,
logically, just as in discrete mathematics)
- If you want to evaluate the right hand side no matter what, use `&` and `|`

## Bitwise operations

- Scala allows you to perform operations on individual bits, just like most
languages
- The bitwise methods are: bitwise-and (`&`), bitwise-or (`|`),
bitwise-xor (`^`), complement (`~`, or `unary_~`)
- Scala integer types also offer three shift methods: shift left (`<<`), shift
right (`>>`) and unsigned shift right (`>>>`)

## Object equality


- If you want to compare two objects for equality, you can use either `==` or
`!=`, e.g.
- You can also compare two objects that have different types

```scala
List(1, 2, 3) == "hello" // returns false
```

- You can compare with `null` as well

```scala
List(1, 2, 3) == null

null == List(1, 2, 3)
```

Note:
- `==` has been implemented so that you get the comparison you want in
most cases. The rule is: first check the left side for `null`, if it's not `null`,
call the `equals` method
- Since there's an automatic `null` check, you don't have to do it yourself
- Scala provides means for reference equality under the name `eq`, however `eq`
and `ne` (its opposite) only apply to objects that directly map to Java objects

## Operator precedence and associativity

### Precedence

- Given that Scala doesn't have operators, just a way to use methods in operator
notation, a logical question is how operator precedence works
- Scala decides precedence based on **the first character** of the methods used in
operator notation, e.g. `*` has a higher precedence than `+`.

```scala
a +++ b *** c // evaluated as a +++ (b *** c)
```

The precedence table for the common methods (operators) looks like this (in
decreasing order):

- all other special characters
- `*` `/` `%`
- `+` `-`
- `:`
- `=` `!`
- `<` `>`
- `&`
- `^`
- `|`
- (all letters)
- (all assignment operators)

- The "first character" rule has only one exception: assignment operators, which
end in an equals character
- If an operator ends in an equals character, and is not one of the comparison
operators, then its precedence is the same as that of simple assignment, hence
it's lower than the precedence of any other operator

```scala
x *= y + 1 // is the same as x *= (y + 1)
```

- In the above example, `*=` is an assignment operator with lower
precedence than `+`, even though the operator's first character is `*`

Note: It's always good style to use parens to clarify what's happening, the code
becomes clearer as knowledge of the above precedence table is not required.

### Associativity

- When multiple operators of the same precedence appear side by side in an
expression, their *associativity* determines the way they are grouped
- The associativity, unlike precedence, is determined **by the last character**
- As mentioned before, any method that ends in `:` is invoked on its right
operand
- Methods than end in any other character are the other way around

```scala
a * b // a.*(b)
a ::: b // b.:::(a)
```

- No matter what associativity an operator has, its operands are always
evaluated left to right
- As noted, when multiple operators are side by side and their precedence is
equal, the associativity kicks in: if the method ends in `:` they are
grouped right to left, otherwise left to right

```scala
a ::: b ::: c // a ::: (b ::: c)
a * b * c // (a * b) * c
```

## Rich wrappers

- Scala's basic types support loads of methods
- This is achieved via *implicit conversions*, a very powerful Scala-specific
feature
- For each basic type we described, there's a rich wrapper that provides
several additional methods: e.g. `Int` -> `scala.runtime.RichInt`, `String` ->
`scala.collection.immutable.StringOps`, `Double` -> `scala.runtime.RichDouble`
and so on
