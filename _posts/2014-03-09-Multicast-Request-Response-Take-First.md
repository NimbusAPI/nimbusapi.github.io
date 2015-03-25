---
layout: page
title: "Multicast Request Response - take First"
category: doc
date: 2014-03-09
order: 8
---

##Why

This is a variation on the [Multicast Request Response](./Multicast-Request-Response.html) example. Instead of the scenario where we want to collect all responses we may only care about latency and the time to the first response.



##The Pattern

In this Multicast Request Response we only take the first response and discard the rest.

![](../../images/MulticastRequestResponseFirst.PNG)


##How

To make the request on the bus we implement the same IBusRequest. This might look like:

	public class FraudulentCardRequest : IBusRequest<FraudulentCardRequest, IsThisDodgyResponse>
	{
		
	
	}

And then we make a MulticastRequest on the bus and call First

	var response = (await bus.MulticastRequest(request)).First();



##Handling it

To implement the handler to fulfil this request we implement IHandleRequest handler class as we would for a simple [Request Response](./Request-Response.html).

