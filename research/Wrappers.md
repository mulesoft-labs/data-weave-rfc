# Memory


In our current model we represent all our values (String, Number, Object ...) in a model that goes like this


    +----------------+
    |                |
    |  evaluate:T    |
    |                |
    |  schema:Schema?|
    |                |
    |  key: Key?     |
    |                |
    |  type : Type   |
    |                |        
    +----------------+

The value object has an `evaluate` method that is the one that returns the real value. 

So one of the problems with this model is that for each value we need two instances, the `value` and the `real value` For example for a `String` value we have the `StringValue` whose evaluate method returns the real `java.lang.String`. Though this wrapping is very helpful and makes the design simple it also generates a lot of overhead and performance cost. 

We should try to find a way to make our value representation lightweight this will really speed things up. 


## Option

The optimal solution is to make the evaluate return the most native possible value without any wrappers when possible, so that we represent each value with the least amount of memory footprint.

- The first problem is that currently ALL VALUES have an associated Type. There is a work in progress by Teo F that can help us get out this constrain.

- Second problem Metadata and Parent Key: We have two options here, one is to send it to the end value and the second is to wrap only on demand.


We need to do a POC to understand the impact of the wrappers and the cost/benefit of the solution

