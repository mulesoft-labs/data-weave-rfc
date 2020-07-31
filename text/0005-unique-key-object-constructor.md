* Feature Name: unique-key-object-constructor
* Start Date: 2020-07-30
* Contributors:
  * Mariano De Achaval
  * Leandro Shokida

# Summary
[summary]: #summary

This feature would enable creating Objects where keys can only appear once and subsequent appearances overwrite the value contained by the previous key.

This would only affect explicit Object construction, afterwards it would behave like a normal Object that allows repeated keys.

# Motivation
[motivation]: #motivation

Currently the way to merge and overwrite values from an object `A` with the values from object `B` is by using `dw::core::Objects::mergeWith`, but its behaviour even though right is confusing in some cases. For example:
```
{
  user: "Mariano",
  user: "Ana",
  user: "Tomo"
} 
mergeWith 
{
  user: "Shoki"
}
```
Results in
```json
{
  user: "Shoki"
}
```

Adding this feature would:
* help make more explicit the result of this operation
* allow us to have this information in the type system
* avoid the need to import the function from the Objects library

# Documentation
[documentation]: #documentation

Objects in DataWeave are created using `{key: value}` but by using `{{key:value}}` you can enforce keys to be unique. If repeated keys are found during the object construction the alst found value will be kept.

## Example 1
```
{{
  user: "Mariano",
  user: "Ana",
  user: "Tomo"
}}
```
would end up as
```json
{
  user: "Tomo"
}
```

## Example 2
```
{{user: "Mariano"}} ++ {{user: "Ana"}}
```
would output
```json
{
  user: "Mariano",
  user: "Ana"
}
```

# Drawbacks
[drawbacks]: #drawbacks

Why should this feature _not_ be implemented in DataWeave?


# Rationale, Alternatives
[rationale]: #rationale

* What other designs have you considered and why did you not choose them?
* Why is this design better than alternatives?
* What is the impact of not adopting this change?

# Existing Implementations
[existing-implementations]: #existing-implementations

Discuss both the good and the bad of existing implementations of this feature, if applicable. A few examples:

* Does this feature exist in other programming languages? What experience has their community had?
* What lessons can we learn from existing implementations?
* Are there any papers or posts that already discuss this?

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

* What parts of the design do you expect to resolve through the RFC process before this feature gets implemented?
* What related issues do you consider out of scope for this RFC that could be address in the future?
* Have you considered how the feature will work outside of the scope of MuleSoft applications?


# Citations
[citations]: #citations
Rust RFC Template: https://github.com/rust-lang/rfcs/blob/master/0000-template.md
Pony RFC Template: https://github.com/ponylang/rfcs/blob/master/0000-template.md
