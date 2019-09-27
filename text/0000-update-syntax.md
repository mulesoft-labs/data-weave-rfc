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

## Updating with literals

```dwl
payload update {
  case at .foo.bar -> 'foo'
}
```

## Updating with variable set:

```dwl
payload update {
  case v at .foo.bar -> v + 5
}
```


## Updating with dollar-sign shorthand:

```dwl
payload update {
  case at .foo.baz -> $ + 5
}
```


## Updating with variables in the path (paths support interpolation like Strings):

```dwl
do {
  var path = 'foo'
  ---
  payload update {
    // Same as .foo.bar
    case at ."$(path)".bar -> $ + 5
  }
}
```


## Updating a null input will return null:

```dwl
null update { case at .foo -> "hello" }
// Returns null
```


## Updating an element that doesn’t exist should return the input:

```dwl
{baz: "bar"} update {
  // Key doesn’t exist
  case at .foo -> "foo"
}
// Returns {baz: "bar"}
```

## To add to a node that doesn't exist yet, use the upsert syntax (!):

```dwl
{bax: "bar"} update {
  case at .foo! -> {foo: "bar"}
}
// Returns {baz: "bar", foo: {foo: "bar"}}
```



## Updating repeated keys
It's necessary to support repeated keys on the same Object in DataWeave. Because of this, an edge case exists where DataWeave cannot intepret what's between `case` and `->` exactly as selectors. Imagine the following example:

```dwl
{a: [1,2,3], a: [4,5,6]} update {
  case at .a* -> {foo: "bar"}
}
```

If the user views `.a*` as functioning exactly like the multi-value selector, then the lambda would be called once with `[[1,2,3],[4,5,6]]`. This creates an ambiguity. Given that this Array could then be transformed into anything, it becomes impossible to determine how the modifications to the Array should be applied back to the original data. Does DataWeave update the first `a`, the second `a`, or both?

The solution for this requires that the "selector" syntax used in `update` functional slightly differently when matching repeated keys. The lambda on the RHS of `->` is called twice, because `.*a` matched on two values. When the lambda is called the first time, the result is applied to the first `a`, when it is called the second time, the result is applied to the second `a`. Updating individual values requires the use of a guard.

### Updating all repeated keys with the same lambda

```dwl
{a: 1, a: 2} update {
  case at .a* -> $ + 1
}
// Returns {a: 2, a: 3}
```

### Changing a specific repeated key

```dwl
{a: 1, a: 2} update {
  case at .*a -> if($$ == 1) $ + 1 else $
}
// Returns {a: 1, a: 3}
```

**OR**

```dwl
{a: 1, a: 2} update {
  case at .*a if($$ == 1) -> $ + 1 
}
// Returns {a: 1, a: 3}
```

## Updating with repeated context:

```dwl
payload update {
  case at .foo.bar -> "foo"
  case at .foo.baz -> "bar"
}
```

## Factoring out repeated context (same result as above without the .foo repetition):

```dwl
payload update {
  case at .foo -> $ update {
    case at .bar -> "foo"
    case at .baz -> "bar"
  }
}
```

Note that you cannot use this as a replacement, as it would return the result of the updated payload.foo, not updated payload:

```dwl
payload.foo update { … }
```


## Updating XML attributes:

```dwl
payload update {
  case at .root.foo.@bar -> “baz”
}
```

## Selecting repeated element with different namespace:

Example input payload:

```xml
<root xmlns:ns0="http://something.com">
    <a>1</a>
    <ns0:a>2</ns0:a>
    <a>3</a>
</root>
```

Code:

```dwl
do {
  ns ns0 http://something.com
  ---
  payload update {
    case at .foo.ns0#a -> "foo"
  }
}
```


### Sub-Branches on Updates:

```
{foo: 123} update {
    case at .foo -> {bar: true}

    // This branch will not execute because
    // .foo.bar does not match anything on the input, {foo: 123}
    case at .foo.bar -> {sub: 123} 
} 
// Returns {foo: {bar: true}}
```


### Same-Branch Updates:

```
{foo: 123} update {
    case at .foo -> {bar: true}

    // This branch will not execute because
    // it was already matched
    case at .foo -> {sub: 123} 
} 
// Returns {foo: {bar: true}}
```


# Drawbacks
[drawbacks]: #drawbacks

This feature adds additional syntax to the language which is, generally speaking, something not taken lightly with programming languages. This feature could conceivably be added as a few additional functions, but the ergonomics of it would not be as nice (see Rational, Alternatives). Personally, I think there's a much greater drawback in not having `update` syntax as a part of the language.

# Rationale, Alternatives
[rationale]: #rationale

In addition to the `update` syntax, `update/with` functions were also considered, and implemented. For example:

```dwl
payload update ["foo"] with "bar"
```

The drawback being that users cannot use the selector syntax that they're already familiar with. Supporting things like XML attributes comes with more functions:

```dwl
payload update ["foo", attr("test")] with "bar"
```

Ultimately, there's just more for users to learn with `update/with` whereas the `update` syntax feels more intuitive.


# Existing Implementations
[existing-implementations]: #existing-implementations

Discuss both the good and the bad of existing implementations of this feature, if applicable. A few examples:

This feature was initially inspired by Clojure's `update-in` function (docs [here](https://clojuredocs.org/clojure.core/update-in)). Which is very mature. It's been available since 1.0.

The lesson learned here, in my opinion, is that a language shouldn't require users to know about recursion and pattern matching when they want to update a single value in a nested data structure.

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

None that I'm currently aware of.
