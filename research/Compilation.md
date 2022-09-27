# Code Compilation

## Problem Statement

Currently we have a large amount of functions implemented natively in the language. This is for performance reasons.
The problem with this approach is
 1. Native code can only be provided by the DW dev team so no one else can implement a function that is high performnat.
 2. Native code is not debugable.
 3. Native code can not be verfiied that does what the signature says (this has taken several problems in the past).
 4. And most important It doesn't scale.

## Solution

We need to start generating bytecode and stop interpreting the code. This is easier to say than done. We also need to figure out a way to do this progressly as it will be super hard to compile all the features that DW language support. 


## Architecture 

A possible solution to this could be to write a compiler plugin that analyzes the dw functions of the modules and compiles into bytecode the functions that are using the subset of features that we support compaling, and register them as native. This way we can do a progressive implementation and we can start taking advantage of this features from the day 1. It is also easy to priortize what building blocks needs to be implemented.

