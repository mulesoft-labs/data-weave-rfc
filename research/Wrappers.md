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


## POC

I've done a very simple POC to capture an initial idea of how much it would benefit this change

This very simple test 

```scala

package org.mule.weave.v2

import org.mule.weave.v2.model.EvaluationContext
import org.mule.weave.v2.model.capabilities.UnknownLocationCapable
import org.mule.weave.v2.model.structure.ArraySeq
import org.mule.weave.v2.model.values.ArrayValue
import org.mule.weave.v2.model.values.FunctionParameter
import org.mule.weave.v2.model.values.FunctionValue
import org.mule.weave.v2.model.values.NumberValue
import org.mule.weave.v2.model.values.Value
import org.mule.weave.v2.model.values.math
import org.mule.weave.v2.parser.location.Location
import org.mule.weave.v2.parser.location.UnknownLocation
import org.mule.weave.v2.runtime.core.functions.collections.ArrayMapFunctionValue
import org.mule.weave.v2.runtime.core.functions.collections.ArrayNoWrapperFunctionValue
import org.mule.weave.v2.runtime.core.functions.collections.BinaryFunction

object WrapperVsWrapperLess extends App {

  def withWrapper() {
    implicit val ctx: EvaluationContext = EvaluationContext()
    val inclusive                       = 1 to 1000000
    val numbers = inclusive
      .map((n) => {
        NumberValue(n)
      })
      .toIterator
    val value = ArrayValue(numbers, UnknownLocationCapable)

    val value1 = ArrayMapFunctionValue.doExecute(
      value,
      new FunctionValue() {
        override def call(args: Array[Value[_]])(implicit ctx: EvaluationContext): Value[_] = {
          ???
        }

        override def call(arg1: Value[_], arg2: Value[_])(implicit ctx: EvaluationContext): Value[_] = {
          val numberValue  = arg1.evaluate.asInstanceOf[math.Number]
          val numberValue2 = arg2.evaluate.asInstanceOf[math.Number]
          NumberValue(numberValue.+(numberValue2))
        }

        override val parameters: Array[FunctionParameter] = {
          Array(FunctionParameter("i"), FunctionParameter("n"))
        }

        override def minParams: Int = 2

        override def location(): Location = UnknownLocation
      }
    )

    val array  = value1.evaluate.asInstanceOf[ArraySeq].toArray()
    val length = array.length
    var i      = 0
    while (i < length) {
      array(i)
      i = i + 1
    }
  }

  def withoutWrapper() {
    implicit val ctx: EvaluationContext = EvaluationContext()
    val inclusive                       = 1 to 1000000
    val numbers = inclusive
      .map((n) => {
        org.mule.weave.v2.model.values.math.Number(n)
      })
      .toIterator

    val value1 = ArrayNoWrapperFunctionValue.doExecute(
      numbers,
      new BinaryFunction[org.mule.weave.v2.model.values.math.Number, org.mule.weave.v2.model.values.math.Number] {
        override def call(a: math.Number, b: math.Number): Any = {
          a.+(b)
        }
      }
    )

    val array  = value1.asInstanceOf[Iterator[org.mule.weave.v2.model.values.math.Number]].toArray
    val length = array.length
    var i      = 0
    while (i < length) {
      array(i)
      i = i + 1
    }
  }

  var i          = 0
  var time: Long = 0
  while (true) {
    val start: Long = System.currentTimeMillis()
    withoutWrapper()
    val took = System.currentTimeMillis() - start
    time = time + took
    i = i + 1
    val amount = 10
    if ((i % amount) == 0) {
      println((time / amount) + "ms")
      time = 0
    }
  }

}


object ArrayNoWrapperFunctionValue {
  def doExecute(leftValue: Iterator[org.mule.weave.v2.model.values.math.Number], fc: BinaryFunction[org.mule.weave.v2.model.values.math.Number, org.mule.weave.v2.model.values.math.Number]): Any = {
    var index: org.mule.weave.v2.model.values.math.Number = org.mule.weave.v2.model.values.math.Number(0)
    leftValue.toIterator.map((i) => {
      val value = fc.call(i, index)
      index = index.+(org.mule.weave.v2.model.values.math.Number(1))
      value
    })
  }
}

trait BinaryFunction[T, Q] {
  def call(a: T, b: Q): Any
}

```



|With Wrapper | Without|
|---|---|
|345ms | 128ms|



This is by no means a number that we should interpolate to real use cases but it give us the sense that it will make a big difference in several use cases.




