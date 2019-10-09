* Feature Name: custom-mime
* Start Date: 2019-10-04

# Summary
[summary]: #summary

This feature adds an way to define the output mime type separetely from the output data format.

# Motivation
[motiviation]: #motiviation

Under certain cirsumstances and particularly when using DataWeave within Mule, users would like to use a custom mime type for their output, for example, `application/edifact` to replace an `application/flatfile` output. To do this today they are force to retype the output with a `set-payload` component.

# Documentation
[documentation]: #documentation

As this feature proposes a new syntax for defining the output section, documentation must adequately explain these changes. However, the usual syntax would still be valid and the use of the feature will probably be quite limited. The feature also requires documenting the names of all data formats since they will be use to id them. The names will be standarize to be entirely lowercase.

Here are some examples:

## Format and mime type

```dwl
output text as text/markdown
```

## With options

```dwl
output json as application/my-custom-type streaming=true 
```

## With type

```dwl
output xml:SomeType as text/html
```

# Drawbacks
[drawbacks]: #drawbacks

This feature adds additional syntax which is only relevant when executing within Mule. Since most current uses are such, it is worth considering.

# Rationale, Alternatives
[rationale]: #rationale

A different possible syntax was considered:

```dwl
output text @ text/markdown
```

As well as using options:

```dwl
output text/plain mimeType=text/markdown
```

However, `@` is not as clear regarding the behavior to expect: `as` suggests that while `text` will be the output, it will be handled with `text/markdown`. On the other hand, using an option would require two mime types to be declared on the same line which creates confusion.

Another syntax option is to use `with`:

```dwl
output xml:SomeType with text/html
```

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

None.
