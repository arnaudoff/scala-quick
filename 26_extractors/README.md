# Extractors

So far we've seen pattern matching and specifically a type of pattern matching
where constructor patterns were linked to case classes, e.g. `Some(x)` is a
valid pattern since `Some` is a case class. You can take this further and write
patterns like that, but without having a case class back it up. Extractors allow
you to do exactly this: decouple a pattern from an object's representation.

## Extracting email addresses

Say you have a web application. A classical problem is to validate email
addresses. Furthermore, given a string, you want not only to decide whether
it's an email address or not, but also to access the user and domain tokens
of the address.

If seeing this for the first time, a sample skeleton we could start off with is
the following snippet:

```scala
def isEmail(text: String): Boolean
def domain(email: String): String
def user(email: String): String
```

We would then parse a given string as follows:

```scala
if (isEmail(s))
  println(user(s) + " at " + domain(s))
```

And this solution works, but it's really not the Scala way and is rather clumsy.
If you've been following along, you'll notice pattern matching is the right
tool here. So, assume we could pattern match a string with a pattern such as

```scala
Email(user, domain)
```

and, as long as it contains an at symbol, it would bind `user` to the part before
the at symbol and `domain` to the part after it. We could then straight match on
the string like that:

```scala
s match {
  case Email(user, domain) => println(user + " at " + domain)
  case _ => println("not an email")
}
```

This is all nice, but strings are not case classes: they don't match the
`Email(user, domain)` representation. And this is where extractors are useful:
they let you define new patterns for already existing types, where the pattern
is decoupled from the internal representation of the type.

## Extractors

- An extractor is simply an object that has a method called `unapply` as one of
its members
- This `unapply` method is used to match a value and take it apart
- Often times, in an extractor is defined a dual method `apply` that is used for
    building values

Let's see how the above ideas would be implemented in the form of an extractor:

```scala
object Email {
  def apply(user: String, domain: String) = user + "@" + domain
  def unapply(s: String): Option[(String, String)] = {
    val parts = s split "@"
    if (parts.length == 2) Some(parts(0), parts(1)) else None
  }
}
```

- The `apply` is optional and turns `Email` into an object that can be
applied to arguments in parentheses like a method is applied, so you can
write `Email("foo", "bar.com")` and get "foo@bar.com"
- However, the `unapply` method is what turns `Email` into an extractor after
all: it reverses the construction process of `apply`, while also taking care
of the case where the given string is not an email address and hence returns
an `Option` type

Now, you can safely use the following construct:

```scala
text match {
  case Email(user, domain) => // ..
}
```

and as soon as a pattern like that, referring to an extractor, is encountered,
the `unapply` method of the extractor is invoked with `text`.

It's worth to note some terminology here:

- The `apply` method is called an `injection`, because it takes some arguments
and yields an element of a given set
- The `unapply` method is called an `extraction`, because it takes an element of
    the same set and extract some of its parts
- The object itself is called an extractor regardless of whether or not it has
    an `apply` method or not

## Patterns with zero or one variables

- The `unapply` method of the previous example returned a pair of element values
- For patterns of more than two variables, this is easily generalized: to bind n
    variables, an `unapply` would return an `n`-tuple wrapped in a `Some`
- The case where a pattern binds only one variable, however, is treated
differently, since there's no one-tuple
- So, to return just one pattern element, the unapply method simply wraps the
element itself in a `Some`

For example, consider an extractor for strings that consist of the same
substring appearing twice in a row:

```scala
object Twice {
  def apply(s: String): String = s + s
  def unapply(s: String): Option[String] = {
    val length = s.length / 2
    val half = s.substring(0, length)
    if (half == s.substring(length))
      Some(half)
    else
      None
  }
}
```

An extractor is not required to bind any variables, however. In that case,
`unapply` returns true or false. For example, an extractor for strings of all
uppercase letters:

```scala
object UpperCase {
  def unapply(s: String): Boolean = s.toUpperCase == s
}
```

Finally, we have the flexibility to apply all of these extractors at the same
time in the following pattern matching construct

```scala
def userTwiceUpper(s: String) = s match {
  case Email(Twice(x @ UpperCase()), domain) => // ..
  case _ => // ..
}

userTwiceUpper("BOBO@gmail.com") // matches
userTwiceUpper("BOBI@gmail.com") // doesn't match
userTwiceUpper("bobo@gmail.com") // doesn't match
```

which matches strings that are email addresses whose user part consists of two
occurrences of the same string in uppercase leters.

## Varargs extractors

- Sometimes, you don't want a fixed number of element values like in the
    previous extraction methods
- Keeping up with the previous example, say you want to match on a string
    representing a domain name, but in a way that every token of the domain is
    kept in a different sub-pattern:

```scala
domain match {
  case Domain("com", "sun", "java") => //
  case Domain("net", _*) => //
}
```

- In the example, sequence wildcard pattern is used at the end of the argument
    list so that sub-domains of arbitrary depth can be matched (in fact, this is
    why domains are in reverse order)
