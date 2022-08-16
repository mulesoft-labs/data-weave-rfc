# Pure Function 

In a pure functional land all functions are pure and doesn't have side effects. This takes you to a place where you can cache any function result or reduce two functions that call the same function with the same arguments into one function call. Now in reality DW does have functions with side effects.

For example `log`, `java bridge`, any mule function. And then any high order function that calls any of this function or any function that calls any of these functions will have side effects too. 

So in order to start optimizing the code we need to start by being able to detect this. And then decide when to cache and when to not cache. And when to fold two calls into one.

