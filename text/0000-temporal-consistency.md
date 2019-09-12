* Feature Name: period-datetime-consistency
* Start Date: (2019-09-10)

# Summary
[summary]: #summary

Currently, DW can be a bit inconsistent when dealing with temporal objects like `Date`, `DateTime`, `LocalDateTime`, `Time`, `LocalTime`, and `Period`. I'll refer to `Date`, `DateTime`, `LocaldateTime`, `Time`, and `LocalTime` as "temporal instances" throughout the RFC, as they identify a specific point in time. This RFC is a proposal on how that can be addressed.

# Motivation
[motiviation]: #motiviation

These features should be added to eliminate the surprises that come with dealing with temporal objects.

# Documentation
[documentation]: #documentation

## Period literals

`Period` literals cannot handle millisecond precision, but literals for the `Time` and `DateTime` varieties do. `Period` literals could be made to support milliseconds like this:

```
|PT5MS|
```

Which would be a `Period` literal representing 5 milliseconds.

## Date Decomposition for Periods

Temporal instances can currently be [decomposed](https://docs.mulesoft.com/mule-runtime/4.2/dataweave-types#date_decomposition) into their composite year, month, day, hour, etc. `Period`s should be able to be decomposed the same way:

```
%dw 2.0
output application/json

var period = |P1Y2M3DT4H5M6S30MS|
---
{
  year         : period.year,
  month        : period.month,
  day          : period.day,
  hour         : period.hour,
  minutes      : period.minutes,
  seconds      : period.seconds,
  milliseconds : period.milliseconds,
  nanoseconds  : period.nanoseconds
}

```

Output:
```
{
  "year": 1,
  "month": 2,
  "day": 3,
  "hour": 4,
  "minutes": 5,
  "seconds": 6,
  "milliseconds": 30,
  "nanoseconds": 30000000
}
```

## A function to decompose temporals

In addition to making the handling of temporal instances and `Period` more uniform, having a function that decomposes either of these into their composite parts would be nice:

```
decomposeTemporal(t: Time | LocalTime | Date | DateTime | LocalDateTime | Period): Object
```

Here's an example with a `DateTime`:

```
%dw 2.0
output application/json

%var myDate = |2003-10-01T23:57:59.700-03:00|
---
decomposeTemporal(myDate)
```

Output:
```
{
  "year": 2003,
  "month": 10,
  "day": 1,
  "hour": 23,
  "minutes": 57,
  "seconds": 59,
  "milliseconds": 700,
  "nanoseconds": 700000000,
  "quarter": 4,
  "dayOfWeek": 3,
  "dayOfYear": 274,
  "offsetSeconds": -10800
}
```

And here is an example with a `Period`:

```
%dw 2.0
output application/json

var period = |P1Y2M3DT4H5M6S30MS|
---
decomposeTemporal(period)
```

Output:
```
{
  "year": 1,
  "month": 2,
  "day": 3,
  "hour": 4,
  "minutes": 5,
  "seconds": 6,
  "milliseconds": 30,
  "nanoseconds": 30000000
}
```

In the event that `decomposeTemporal` is passed the wrong type, it should throw an error.

## A clearer distinction between temporal instances and `Period`s

The documentation could be a bit more verbose about the difference between temporal instances and `Periods`. It doesn't say that `Period` represents a length of time. If the `decomposeTemporal` function is included in the language, it should be stated in the documentation that the keys returned in the object mean something conceptually different when passing a temporal instance vs a `Period`. In the case of a temporal instance, the keys represent where the instance falls on a time line, including some additional information. In the case of `Period`s, the keys represent how many of each of the durations will be ultimately applied to the temporal instance.

# Drawbacks
[drawbacks]: #drawbacks

Having `decomposeTemporal` return an `Object` could be problematic because the result could represents two different concepts. This could be resolved by having `decomposeTemporal` return a more specific type, which would have a representation as an `Object`, for example:

Definition for `CompositePeriod`
```
{
  year?:         Number,
  month?:        Number,
  day?:          Number,
  hour?:         Number,
  minutes?:      Number,
  seconds?:      Number,
  milliseconds?: Number,
  nanoSeconds?:  Number
}
```

# Rationale, Alternatives
[rationale]: #rationale

* What other designs have you considered and why did you not choose them?
* Why is this design better than alternatives?
* What is the impact of not adopting this change?

# Existing Implementations
[existing-implementations]: #existing-implementations

I looked into how Java 8 handles temporals (aka JodaTime), especially Periods. Java 8 handles Periods a bit differently than DW does. It makes a distinction between a `Duration` which is a period of time that does not use date-based constructs like years, months, and days. It then has a `Period` which is a period of time that does use date-based constructs.

Discuss both the good and the bad of existing implementations of this feature, if applicable. A few examples:

* Does this feature exist in other programming languages? What experience has their community had?
* What lessons can we learn from existing implementations?
* Are there any papers or posts that already discuss this?

# Unresolved Questions
[unresolved-questions]: #unresolved-questions

* How will `Period`s handle things like leap years and daylight savings time?
* How will `Period`s account for months containing between 28 and 31 days?
* Should `decomposeTemporal` return `null` if it receives bad input?


# Citations
[citations]: #citations
Rust RFC Template: https://github.com/rust-lang/rfcs/blob/master/0000-template.md
Pony RFC Template: https://github.com/ponylang/rfcs/blob/master/0000-template.md
