# Evolve Input Directive

Input directives have caused us problems specially in mule land where all inputs are implicits 
and reader configuration are taken from the values mime types. What happen is that people use them in 
places like the playground to specify the reader properties and then they copy paste into mule and it doesn't work.

I've been thinking on alternatives to this. 

## Option 1: Deprecate Input Directives

This is a valid option but I think there is value in input directives. One is that if you want to create a re-usable mapping,
declaring your inputs and their types helps consumers understand about the mapping how it needs to be called. Also there are things
like `@StreamCapable` that are usefull verifications that only works if they are present.

## Option 1: Deprecate The Format dependendent part of Input Directives

If we take the input directive we can identify two parts

```
<annotations>*
input <name><:typeDefinition>? <mimeType|formatId> <readerProperties*>
```

Form this pseudo grammar we dan spot two parts the variable definition section 

```
<annotations>*
input <name><:typeDefinition>?
```

And the format dependent part 

```
<mimeType|formatId> <readerProperties*>
```

What I suggest is one let's make the format part optional and deprecate it. This way we make our script independent of the data format of the inputs, we just care about the input shapes. Secondly change our tools like the playground to allow another simple way to specify the reader properties. For example in our IDEs (IJ and VScode) reader properties are taken from a `.properties` file that is a companion of the input file. For example `payload.csv` `payload-config.properties`. This way a user could test different input with different reader properties.

I think we should take a similar approach with the output. This way we are separating the transformation logic from the format specifics.