* Feature Name: literal-types
* Start Date: 2020-01-30

# Summary
[summary]: #summary

This feature adds support for using literal values as types. This means that for example the String `"foo"`, the Number `200`, or the Boolean `false` can be used as types. Combined with union types, this feature lets DW to have an enumeration of literal values as types. 

# Motivation
[motivation]: #motivation

This feature should be added to be able to let the users declare types as a finite set of allowed values. 
This type can be used inside a variable declaration, function type parameter, type declaration, pattern matching with types, allowing the user to not have to check if a value is contained in that finite set of allowed values because the type system would do that for them.
# Documentation
[documentation]: #documentation

A literal type is a type that represents exactly one value, e.g. one specific string or number.

The following literal types have been added to the type system:
- String literal types
- Boolean literal types
- Number literal types

For example you could declare something like this:

## Declaring a type alias of union of literals

```
type Days = "Monday" | "Tuesday | "Wednesday" | "Thursday" | "Friday"
```

## Variable that accept only the specific literal values

```
var errorStatusCode: 404 | 500 = payload.statusCode
```

## Function parameter that accepts specific literal values
With function overloading the user is able to define different behaviors based on the input argument:

```
fun errorHandling(errorCode: 404 | 405, response) = "Code for error 4XX handling here!"
fun errorHandling(errorCode: 500 | 501, response) = "Code for error 5XX handling here!"
---
errorHandling(payload.statusCode, payload)
```

## Pattern Matching on literal types

```
someVariable match {
    case is "a" | "b" | "c" -> "ABC"
    case is 0 | 1 -> "Zero or One"
    case is true -> "TRUEEE!"
}
```

As these examples show, literal values can be used in every moment where a type is needed (function parameters, type of a variable, type declaration, etc.).

# Drawbacks
[drawbacks]: #drawbacks

As a drawback this could cause a little bit of confusion between the types of the type system and the types of Runtime. 

For example:
```
var a: "asd" = "asd"
---
if (typeOf(a) == "asd") "YES" else "NO"
```
In this example, someone could expect that the output be "YES" but in this case is "NO" because the literal types only works at the type system level and not in runtime, that is why the typeOf(a) returns "String".

The function overloading algorithm that we have right now, use a cache to call the last function that was called and that makes that DW keeps calling the same function even if the parameter value had changed (This is a problem for recursive functions).

This example returns Stack Overflow.
```
fun zero (a : 0) = 1
fun zero (a: Number) = cero(a-1)
---
zero(2)
```


# Existing Implementations
[existing-implementations]: #existing-implementations

Other programming language that have this feature is TypeScript (docs [here](https://www.typescriptlang.org/docs/handbook/advanced-types.html#string-literal-types))

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

None.
