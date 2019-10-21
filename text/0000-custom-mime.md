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

## Format

```dwl
output csv
```

## Format and mime type

```dwl
output text/markdown with text
```

## With options

```dwl
output application/my-custom-type with json streaming=true 
```

## With type

```dwl
output :SomeType text/html with xml
```

# Drawbacks
[drawbacks]: #drawbacks

This feature adds additional syntax which is only relevant when executing within Mule. Since most current uses are such, it is worth considering.

# Rationale, Alternatives
[rationale]: #rationale

Different possible syntax were considered:

```dwl
output text @ text/markdown
```

```dwl
output text as text/markdown
```

As well as using options:

```dwl
output text/plain mimeType=text/markdown
```

However, `@` is not as clear regarding the behavior to expect: `with` suggests that while `text` will be the output, it will be handled with `text/markdown`. On the other hand, using an option would require two mime types to be declared on the same line which creates confusion. The `as` option does not work well with the type declaration as it is not clear whether the format should be declared before or after the type.


# Unresolved Questions
[unresolved-questions]: #unresolved-questions

None.
