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


