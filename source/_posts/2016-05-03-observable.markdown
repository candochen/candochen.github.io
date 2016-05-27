---
layout: post
title: "Observable"
date: 2016-05-03 19:26:15 +0800
comments: true
categories: Scala 
---

In the Reactive Programming course, Erik Meijer introduced 4 essential effects in programming. This text is intended to describe the essential effects in my own way, with examples. 

Let's begin with simple functions, which return one or many values synchronously.  As shown in the below diagram, the getValue function returns a value of type T, and getValues returns an iterable of type T. 
<table>
<thead>
<tr>
  <th></th>
  <th>One</th>
  <th>Many</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Synchronous</td>
  <td>T getValue</td>
  <td>Iterable[T] getValues</td>
</tr>

</tbody>
</table>
Figure 1

# Exception
Any developer should know that apart from the normal case, there exist exceptions. For example, 
exception will occur if users enter 0 as the divisor in the below function.


def divide(dividend: Double, divisor : Double) : Double = {
    dividend / divisor
}


In order to handle the case when exception happens, we should wrap the computation within a Try monad. So the code should be 

{code}

def divide(dividend: Double, divisor : Double) : Try[Double] = {
    Try(dividend / divisor)
}
{code}

User of this function will be aware of the Try[Double]. It is very comprehensive. This function reads like "try to get a value of Double type by invoking the function divide" literally.

{code}
val result = divide(divident, divisor)

result match {
	case Success(v) => println(v)
	case Failure(e) => println(e)
}
{code}

Likewise, a function returning many value also may have exceptions. In such case, what we need to do is to define the function as  def getValue: Try[Iterable[T]]

Til now, we know that functions may have exceptions. In order to notify users of the function, developers should have computation wrapped within a Try monad. So after considering exceptions, the table becomes:

<table>
<thead>
<tr>
  <th></th>
  <th>One</th>
  <th>Many</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Synchronous</td>
  <td>Try[T] getValue</td>
  <td>Try[Iterable[T]]  getValues</td>
</tr>

</tbody>
</table>

# Latency

Besides exceptions, one term in computer programming we should consider is Latency. Latency by narrow definition in computer programming is: 
> 
Latency is the amount of time a message takes to traverse a system. In a computer network, it is an expression of how much time it takes for a packet of data to get from one designated point to another. It is sometimes measured as the time required for a packet to be returned to its sender.

From my perspective, if any computation that is expected be slower that other computation, then there exists latency for the slower computation relatively. For example, getFromNetwork has latency comparing with getFromRAM.

In order to handle the expected latency, we can make use of Future.  
>A Future is an object holding a value which may become available at some point.

Let's look at an example:


{code}

import scala.concurrent._

import ExecutionContext.Implicits.global

import scala.util.{Success, Failure}

object Weather extends App{
		
		//val content = Source.fromURL("www.weather.com.cn/data/sk/101280601.html").mkString
		val weatherFuture  = Future {
			WeatherInfoRestService.getWeatherInfo
		}
		
		
		//register a callback
		weatherFuture onSuccess {
		  case x => {
			  			println(x.weatherinfo.city)
			  			println(x.weatherinfo.time)
			  			println(x.weatherinfo.temp)
		  			} 
		}
	  
		weatherFuture onFailure {
		  case e => println(e)
		}
		
} 


{code} 


Til now, we know that functions may have exceptions. In order to notify users of the function, developers should have computation wrapped within a Try monad. So after considering exceptions, the table becomes:

<table>
<thead>
<tr>
  <th></th>
  <th>One</th>
  <th>Many</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Synchronous</td>
  <td>Try[T] getValue</td>
  <td>Try[Iterable[T]]  getValues</td>
</tr>

<tr>
  <td>Asynchronous</td>
  <td>Future[T] getValue</td>
  <td>???</td>
</tr>

</tbody>
</table>

# Observable

<table>
<thead>
<tr>
  <th></th>
  <th>One</th>
  <th>Many</th>
</tr>
</thead>
<tbody>
<tr>
  <td>Synchronous</td>
  <td>Try[T] getValue</td>
  <td>Try[Iterable[T]]  getValues</td>
</tr>

<tr>
  <td>Asynchronous</td>
  <td>Future[T] getValue</td>
  <td>Observable[T] getValues</td>
</tr>

</tbody>
</table>



# Wrap-up


