---
layout: page
title: "Multicast Request Response"
category: doc
date: 2014-03-09
order: 7
---

##Why

Imagine you have a number of different types of services in your system that can respond to a particular request and you want to aggregate all the responses. 

An example is a credit card payment gateway with a need to do fraud detection. We may have a number of services each optimised to detect different patterns of fraudulent transactions and each return a score to the calling application so it can determine a final threat score.


##The Pattern

We call this a Multicast Request Response. We make a Request on the bus with a timeout. After the specified time we collect all the responses.

![](../../images/MulticastRequestResponse.PNG)


##How

To make the request on the bus we implement an IBusRequest as we saw in the simple [Request Response example](./Request-Response.html). This might look like:

	public class FraudulentCardRequest : IBusRequest<FraudulentCardRequest, IsThisDodgyResponse>
	{
		
	
	}

And then we make a MulticastRequest on the bus and await the IEnumerable collection of responses.

	var response = await bus.MulticastRequest(request, TimeSpan.FromSeconds(1));



##Handling it

To implement the handler to fulfil this request we implement IHandleRequest handler class as we would for a simple [Request Response](./Request-Response.html).

