# Type selection


The idea of this proposal is to describe a way to be able to do `.` selection over types. 

*For example*

```dataweave
type User = {name: String}

type NameType = User.name
```

With this option it will allow us to reference a type from any object with out the need to refactoring.

## Supported selections

There are two kind of selections static and dynamic selections. 

### Static selection

For static selections we should support both with and without namesapce. A question would be if we want to do the same matching. 

*For example*

```dataweave
ns ns1 http://acme.com

type User = {
  ns1#name : String,
  name : Number,
}

type A = User.name
```

If the type selection follows the runtime semantics type `A` should be String, but I think that it should return `Number`. This may introduce some noice but I think is much clear.


So to have the `String` type the expression should be like

```dataweave
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

```dataweave
fun test<T <: String, Q :< {}>(fieldName: T, a: Q): Q[T] = a[fieldName]
```

This scenario has is hard to validate and this will only add value when `T` is a literal type. 

For example what would this expression return

`test(random() as String, {"0.3": 123})` 


## Some conclusion


I think that we should investigate dynamic and static in separate threads and maybe in different stages


## Buisness need behind

The need to support this cames because currently DW is the default language for transfroming any data format into another, but we want to extend this for type. We want to make DW the default language to be able to interact with any type schema. JsonSchema, XmlSchema, etc. This introduces one problem is that those schema formats are fixed and can not be refactored to extract parts of those schemas. Moreover we don't have a way to have a propper support for XMLSchema and this is because each element in a XMLSchema has a namespace and we don't have a way to put namespaces on our variables nor it makes sense to do so. 


So for example for a schema like

*MySchema.xsd*
```
<xsd:schema xmlns:xsd="http://www.w3.org/2001/XMLSchema"
           xmlns:tns="http://tempuri.org/PurchaseOrderSchema.xsd"
           targetNamespace="http://tempuri.org/PurchaseOrderSchema.xsd"
           elementFormDefault="qualified">
 <xsd:element name="PurchaseOrder" type="tns:PurchaseOrderType"/>
 <xsd:complexType name="PurchaseOrderType">
  <xsd:sequence>
   <xsd:element name="ShipTo" type="tns:USAddress" maxOccurs="2"/>
   <xsd:element name="BillTo" type="tns:USAddress"/>
  </xsd:sequence>
  <xsd:attribute name="OrderDate" type="xsd:date"/>
 </xsd:complexType>

 <xsd:complexType name="USAddress">
  <xsd:sequence>
   <xsd:element name="name"   type="xsd:string"/>
   <xsd:element name="street" type="xsd:string"/>
   <xsd:element name="city"   type="xsd:string"/>
   <xsd:element name="state"  type="xsd:string"/>
   <xsd:element name="zip"    type="xsd:integer"/>
  </xsd:sequence>
  <xsd:attribute name="country" type="xsd:NMTOKEN" fixed="US"/>
 </xsd:complexType>
</xsd:schema>
```

We can translate this schema in a type like

*xmlschema!MySchema*

```dataweave
ns ns1 http://tempuri.org/PurchaseOrderSchema.xsd
type ROOT = {
   ns1#PurchaseOrderType: {
      ns1#PurchaseOrder @(OrderDate: Date): {
        ns1#ShipTo @(country: String): USAAddressDef.ns1#USAddress,
        ns1#BillTo @(country: String): USAAddressDef.ns1#USAddress,
      }
  }
}

type USAddressDef = {
    ns1#USAddress: {
      name: String,
      street: String,
      city: String,
      state: String,
      zip: Number
    }
}

```

And then reference each type using selectors

```dataweave
output application/xml
import * from  xmlschema!MySchema
ns ns1 http://tempuri.org/PurchaseOrderSchema.xsd

type Order = ROOT.ns1#PurchaseOrderType
var fixedAddress: USAddressDef.ns1#USAddress = {
      name: "Simpsons",
      street: "Avenida Siempreviva",
      city: "Pilar",
      state: "Buenos Aires",
      zip: 1636
}
var orders: Array<Order> = [
{
   ns1#PurchaseOrder @(OrderDate: |2020-10-01|): {
        ns1#ShipTo @(country: "Argentina"): fixedAddress,
        ns1#BillTo @(country: "Argentina"): fixedAddress
      },
   ns1#PurchaseOrder @(OrderDate: |2020-10-02|): {
        ns1#ShipTo @(country: "Argentina"): fixedAddress,
        ns1#BillTo @(country: "Argentina"): fixedAddress
      }
}
]
---
"PurchaseOrders": orders
```


## Releated Work

Typescript has a similar behaviour called indexed access.

- https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html