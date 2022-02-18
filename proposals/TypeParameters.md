# Type Parameters Specification

Currently we don't have a way to specify the binding of the type parameters on a function call. They need to be inferred based on the context. 


Lets start with a few examples


```dataweave
fun head<T>(a: Array<T>): T = a[0]

var a = head([random() as String])
```

In this case the type of `T` is going to be inferred as `String` this is because the type of head function is 

`<T>(Array<T>) -> T` and is being applied with `(Array<String>)`

When this happens in order to infer the type of T a set of constrains are built



| Expected | Actural | 
| -------- | --------|
| T        | String  |


Then the solution is simple `T` needs to be of Type `String` then it will substitue the result with this solution. `T <= String` 

This is a very simple example but it shows the basics of how the type parameter inference works.


Now lets see a case where these doesn't work as expected.



```
fun defaultValue<T>(a: T) : (T) -> T = (value: T) -> if(value == null) a else value      
---
defaultValue("2")("a")
```
This scripts fails with 

```
Expecting Type: `"2"`, but got: `"a"`.
	|-- From: `"2"`
	|---- From: AnonymousFunction($: `"2"`) -> `"2"`
```

And why is this? Is beacuse from `defaultValue("2")` it will infere a `(
"2") -> "2" ` That is not exactly what we want. 

We have been hacking the TypeParamerter solver to be able to express different behaviours, but we will continue hitting limitations.

## Solution

The solution is quite simple, add a sintax for expressing the type binding on a function call 

functionExpression<ParamBinding*>?(Params*)

So in our example

```
fun defaultValue<T>(a: T) : (T) -> T = (value: T) -> if(value == null) a else value      
---
defaultValue<String>("2")("a")
```

This will start working as now `defaultValue<String>` will replace T with String and then result will be `(String) -> String`


### Validations


### All or nothing


We are only going to allow to either specify all the type parameters or non but not to parcially apply.

So for example

```
fun test<T, Q>(a: T, b: Q): {a: T, b: Q} = ???
```

in this case is valid `test("Mariano", 32)` or `test<String, Number>("Mariano", 32)` but `test<String>("Mariano", 32)` is going to be invalid

this will simplify the implementation for now.


### Binding should be valid replacements


We need to validate that the specified substitution is valid

```
fun test<T <: String, Q>(a: T, b: Q): {a: T, b: Q} = ???
```

In this case `test<"Mariano", Number>("Mariano", 32)` is valid as `"Mariano" < String` but `test<Number, Number>(123, 32)` should complain as `Number` is not a valid subsitution for `T` as `Number < String` is *false*

