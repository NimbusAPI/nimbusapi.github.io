---
layout: page
title: "Request Response"
category: doc
date: 2014-03-09
order: 5
---

##Why

There are times when you need to ask a question within your application and get an answer. This might be something like GetAccountDetails(1234). 


##The Pattern

This pattern is called Request/Response. The name makes sense, we make a Request and wait for the Response. 

![](../images/RequestResponse.png)

If we need to scale this we can add extra responding service instances for built in, pull based load balancing.

![](../images/RequestResponseScaled.png)


Note that in a loosely couple architecture, ideally the individual components act autonomously and Request Response isn't a patten you don't want to overuse. Also due to the way the return queues for messages are structured in Nimbus, relying on too many Request Response calls isn't necessarly the most performant options.

That said, there are times when you just need it so Nimbus has a rich way to return data from another service in your application.


##How

To make a request on the bus we need to implment a BusRequest. This looks like:

	public class AccountDetailsRequest : BusRequest<AccountDetailsRequest, AccountDetailsResponse>
	{
		public int AccountId {get; set;}
	
	}

This looks somewhat awkward, but where associating the request type with our expected response type. Now we can put it on the bus like this:

	var response = await bus.Request(new AccountDetailsRequest{AccountID=1234});



##Handling it

To implement the handler to fulfill this request we create a class that implements the IHandleRequest interface.

	public class HandleAccountDetailRequests : IHandleRequest<AccountDetailsRequest, AccountDetailsResponse>
	{
    	public async Task<AccountDetailsResponse> Handle(AccountDetailsRequest request)
        {
            //Load customer details from database

            return new AccountDetailsResponse {Name= customer.Name};
        }
	}


##Scaling it

We can add multiple listeners for the request as seen in the diagram above and this will balance the requests across the handlers in a pull based way without additional configuration.