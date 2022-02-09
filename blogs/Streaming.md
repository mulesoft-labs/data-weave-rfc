# Streaming in DW

## Collections in DataWeave


DataWeave has two kind of collections *Array* and *Object*. 

* Array: is a sequence of values 
* Object: is a sequence of key value pairs.

Our collections are inmutable (as everything in DataWeave) they allow random access and being consumed multiple times. This model is very simple to understand and consume.

In order to support this model and also tried to be memory efficient we designed our collections to support two different models. An Array kind and an Iterator kind.

The Array kind is the simplest as it supports directly all the requirements. We call this model the materialized model. But it has the cost that it is not the most memory efficient as we need to keep reference of all the elements of a collection (either in memory or in disk). The iterator model allows us to be memory efficient as we can discard the elements once they are consumed, no need to keep reference to them. We called this model streaming mode.


The idea of this blog is to describe how we pick and switch between the two models.
As described previously our collections support random access and can be consumed multiple times, but that doesn't mean it is being consumed multiple times or in a random access way. So the idea is that if we can determine that a variable is consumed only once and in a sequencial manner we can use the streaming mode in a secure way if not the value needs to be materialized.

To determine if a variable needs to be materialized we do a code analysis (similar to scape analysis idea on compilers optimization). Using [linear logic](https://en.wikipedia.org/wiki/Substructural_type_system#Linear_type_systems)    we can determine if a variable is access in a secuencial maner and only once.

Our approach is to validate a few rules 

1. The variable is only reference once
2. The variable is access in the same scope as it was defined
3. Only sequencial access is allowed

Let's view some example

# example 1
```
var a = 1 to 100000

---

a[1] + a[0]
```

This case shows a clear example of a non linear access to the variable `a` why? because it access the variable `a` it accessed more than once and it is access in a random way. This breakes rule number 1



# example 2

```
var a = 1 to 100000

---

a << 2 
```

This example shows an example where materialization is not required as the variable a is only access once and in a sequencial order.

# example 3

```

var a = 1 to 100000

---

[1,2,3] map ((item) -> a)
```

This is a more complex scenario even if there is only one refrence to the variable. The reference is inside another scope that it was defined. It is inside a lambda and that lambda is going to be executed multiple times. This breaks rule number 2


 # example 4

```

fun addToEach(a: Array<Number>, amount: Number) = 
    a match {
     case [x ~ xs] -> [x + amount ~ addToEach(xs, amount)]
     case [] -> []
    }
    
    

```

In this example variable `a` is streamed because it is being consumed only once and in a sequencial way. The head and tail deconstructor `[x ~ xs]` garanties that the access is sequencial.


## Streaming


Now that we understand the how our collections work we can analyze when an input is ready for streaming or not. If an input is never going to be materialized then we can use the streaming parser for this input.


There is a way to verify that an input can be streamed through the script an is by annotating an input with the @StreamCapable annotation.
By using this annotation DW will fail if any random access is being used over this input. This validation is being done by analyzing not just direct access but also any re assignment or function call.
