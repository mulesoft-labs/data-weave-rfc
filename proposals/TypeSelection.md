# Type selection


The idea of this proposal is to describe a way to be able to do `.` selection over types. 

*For example*

```
type User = {name: String}

type NameType = User.name
```

With this option it will allow us to reference a type from any object with out the need to refactoring.

## Supported selections

There are two kind of selections static and dynamic selections. 

### Static selection

For static selections we should support both with and without namesapce. A question would be if we want to do the same matching. 

*For example*

```
ns ns1 http://acme.com

type User = {
  ns1#name : String,
  name : Number,
}

type A = User.name
```

If the type selection follows the runtime semantics type `A` should be String, but I think that it should return `Number`. This may introduce some noice but I think is much clear.


So to have the `String` type the expression should be like

```
ns ns1 http://acme.com

type User = {
  ns1#name : String,
  name : Number,
}

type A = User.ns1#name
```

### Dynamic selection

For dynamic selections this can be achieved with the mix of type parameters + literal types

*For example*

```
fun test<T <: String, Q :< {}>(fieldName: T, a: Q): Q[T] = a[fieldName]
```

This scenario has is hard to validate and this will only add value when `T` is a literal type. 

For example what would this expression return

`test(random() as String, {"0.3": 123})` 


## Some conclusion


I think that we should investigate dynamic and static in separate threads and maybe in different stages


## Buisness need behind

The need to support this cames because currently DW is the default language for transfroming any data format into another, but we want to extend this for type. We want to make DW the default language to be able to interact with any type schema. JsonSchema, XmlSchema, etc. This introduces one problem is that those schema formats are fixed and can not be refactored to extract parts of those schemas. Moreover we don't have a way to have a propper support for XMLSchema and this is because each element in a XMLSchema has a namespace and we don't have a way to put namespaces on our variables nor it makes sense to do so. 

