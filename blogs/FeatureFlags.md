
# DataWeave Compatibility 

In salesforce trust is ower number one priority, so breaking customer scripts is not an option. It has been very hard and challenging to fullfil this and 
at the same time evolve the language. As we are humans, we need to accept the fact that we had and we will make bad decisions, and we need to build dataweave
with this in mind. So the last couple of versions we have been adding flags to turn on and off different behaviours.

The flags are starting to grow and creates an exponential set of new combination of branches to start testing. And there is also a problem with all this, is that we 
want new DataWeave users to start enjoying by default all the fixed by decitions, but we want old scripts to keep on working.



## Feature and compatibility flags

We have created a new kind of flags that can change their default behaviour based on the version of DW that we want to be compatible with. Also this flags are only visible in a window of DataWeave versions.
This not only allow us to be compatible but also to have the fixed behaviour being the default in a new version and also not allowing to change to the old behaviour since another newer version.

This looks complex no? Lets put an example:

In dataweave runtime 2.2.0 we realized that we had a behaviour where we ensure that the value selector operator always returns the first occurance of the selected field.

So for example in an object {b: 0, a:1, b: 2, a: 3}.a we allways returned 0. The problem with this decision is that it is very expensive as we allways need to do a full lookup. 

So in order to speed things up we need to change this, but we can not change this because it may break anyone counting with this behaviour. With a feature flag we can say that if your script was develop for 2.2.0 or lower version then behaviour should be that, but from 2.3.0 people can enjoy the new performance improvement, we have also defined that in 2.3.0 people can still use old behaviour but since 2.4.0 there is no way to change it.

Wait!? Does this mean if I use data weave 2.4.0 my old app will break? Not at all data weave 2.4.0 will behave compatible with the version that this app was built against. So if your app was developed against 2.2.0 then it will behave as in data-weave 2.2.0 runtime, same as if 2.3.0 where users can pick.


## Visiblity


Now that we understand the new flags, we also defined that in order to be 100% backwards compatible we also need to guarantee that you can only see what you were able to access in the version you developed against.

So let's say you developed against 2.2.0, that version of DataWeave didn't had the `update`operator. So what this mean is that even if you are running your in dataweave 2.4.0 runtime that has this support it will not let you use it. 

Why we did this?

1. We want everyone to be motivated to move to the newer versions
2. To avoid combination hell of old behaviours with new features

This same thing will apply to functions, and that is why we annotate every new function with the @Since. Wit this we can validate if it is visible for your compatible version or not.

If you want to do some tests you can use the cli flag `--language-level`


For example:

If we try to use the update operator and we specify the language level to 2.1.0 

```
dw run --language-level 2.1.0  "{} update { case .name! -> 'Mariano'}"
```

It fails specifying what is the minimum version number that we need to use in order to support this feature

```
[ERROR] Error while executing the script:
[ERROR] `Update Operator` was introduced on version `2.3.0`, but your version level is `2.1.0`.`

1| {} update { case .name! -> 'Mariano'}
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Location:
anonymous (line: 1, column:1)
```

