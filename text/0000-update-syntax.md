* Feature Name: update-syntax
* Start Date: 2019-09-03

# Summary
[summary]: #summary

This feature adds an easy way to update single values in nested data structures without requiring the end-user to have knowledge of functional recursion.

# Motivation
[motiviation]: #motiviation

This feature should be added because currently an end-user needs to know about functional recursion and probably pattern matching should they want to update a single field in a data structure without touching any of the other ones. This is a problem because DataWeave currently makes a simple, commonly used pattern (updating a single value in a nested data structure) and puts it out of reach of your average integration programmer than is not familiar with functional programming (FP).

The expected outcome is that DataWeave users will no longer need to know to use recursion and pattern matching when they want to update a single value, they will just need to know the `update` syntax.

# Documentation
[documentation]: #documentation

As this feature proposes a new syntax, and not just new functions, documentation must adequately explain what is valid and what is invalid syntax for `update`. That being said, the leap from knowing nothing about `update`, to being able to effectively use it should be a small one. This syntax reuses a couple pieces of functuionality that DataWeave users are already familiar with, pattern matching and the selector syntax. Instead of `case is ...` this syntax uses `case at ...` followed by the afformentioned selector syntax. Here are some examples:

### Updating with literals

```
payload update {
  case at .foo.bar -> “foo”
}
```

### Updating with variable set:

```
payload update {
  case v at .foo.bar -> v + 5
}
```


### Updating with dollar-sign shorthand:

```
payload update {
  case at .foo.baz -> $ + 5
}
```


### Updating with variables in the path (paths support interpolation like Strings):

```
do {
  var path = “foo”
  ---
  payload update {
    // Same as .foo.bar
    case at .”$(path)”.bar -> $ + 5
  }
}
```


### Updating a null input will return null:

```
null update { case at .foo -> “hello” }
// Returns null
```


### Updating an element that doesn’t exist should return the input:

```
{baz: “bar”} update {
  // Key doesn’t exist
  case at .foo -> “foo”
}
// Returns {baz: “bar}
```


### Updating with a match on a repeated element:

```
payload update {
  case v at .foo.*bar -> v map    
    if($$ == 4)
      “I’m the 5th element”
    else
      “Just another element”
}
```


### You can also select a single element in a selected Array:

```
payload update {
  case at .foo.*bar[4] -> “I’m the 5th element”
}
```


### Updating with repeated context:

```
payload update {
  case at .foo.bar -> “foo”
  case at .foo.baz -> “bar”
}
```

### Factoring out repeated context (same result as above without the .foo repetition):

```
payload update {
  case at .foo -> $ update {
    case at .bar -> “foo”
    case at .baz -> “bar”
  }
}
```

Note that you cannot use this as a replacement, as it would return the result of the updated payload.foo, not updated payload:

```
payload.foo update { … }
```


### Updating XML attributes:

```
payload update {
  case at .root.foo.@bar -> “baz”
}
```

### Updating XML Namespaces (needed?):

```
payload update {
  case at .foo.# -> “http://new-namespace.com”
}
```

### Selecting repeated element with different namespace:

Example input payload:

```
<root xmlns:ns0="http://something.com">
    <a>1</a>
    <ns0:a>2</ns0:a>
    <a>3</a>
</root>
```

Code:

```
do {
  ns ns0 http://something.com
  ---
  payload update {
    case at .foo.ns0#a -> “foo”
  }
}
```


# Drawbacks
[drawbacks]: #drawbacks

This feature adds additional syntax to the language which is, generally speaking, something not taken lightly with programming languages. This feature could conceivably be added as a few additional functions, but the ergonomics of it would not be as nice (see Rational, Alternatives). Personally, I think there's a much greater drawback in not having `update` syntax as a part of the language.

# Rationale, Alternatives
[rationale]: #rationale

In addition to the `update` syntax, `update/with` functions were also considered, and implemented. For example:

```
payload update ["foo"] with "bar"
```

The drawback being that users cannot use the selector syntax that they're already familiar with. Supporting things like XML attributes comes with more functions:

```
payload update ["foo", attr("test")] with "bar"
```

Ultimately, there's just more for uses to learn with `update/with` whereas the `update` syntax feels more intuitive.


# Existing Implementations
[existing-implementations]: #existing-implementations

Discuss both the good and the bad of existing implementations of this feature, if applicable. A few examples:

This feature was initially inspired by Clojure's `update-in` function (docs [here](https://clojuredocs.org/clojure.core/update-in)). Which is very mature. It's been available since 1.0.

The lesson learned here, in my opinion, is that a language shouldn't require users to know about recursion and pattern matching when they want to update a single value in a nested data structure.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

None that I'm currently aware of.