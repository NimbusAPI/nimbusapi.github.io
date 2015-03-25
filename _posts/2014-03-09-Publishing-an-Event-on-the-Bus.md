---
layout: page
title: "Publishing an Event on the Bus"
category: doc
date: 2014-03-09
order: 5
---

##Why

In a distributed system, the different components need to react to significant business events that are raised by other components of the system.

Events in your architecture might be things like OrderConfirmed, CustomerCreated or AccountSuspended. These events may be of interest to many other parts of your system.

##The pattern

This pattern is called Publish / Subscriber sometimes shortened to PubSub. In this patten we have a Publisher of the event, and multiple applications that Subscribe to these events and act on them.

![](../../images/PubSub.PNG) 

##How

To raise or publish an event with Nimbus we implement the event class as an IBusEvent and publish it using the _bus.Publish method.

	_bus.Publish(new NewOrderRecieved {CustomerName = "Ricky Bobby"});


##Handling it

To handle the event, we create a class which implements the IHandleMulticastEvent interface.

	public class ListenForNewOrders : IHandleMulticastEvent<NewOrderRecieved>
    {
     
        public async Task Handle(NewOrderRecieved busEvent)
        {
            Console.WriteLine("I heard about a new order from " + busEvent.CustomerName);

            //Do more stuff
        }
    }

##Scaling it

We can have multiple applications subscribing to these events, and multiple instances of each application. Remember how we set up an [Application name and an instance name](./Getting-Started-With-Nimbus.html) in our bus configuration? We make sure that each of these gets a copy of the message because we create a subscriber queue for each instance.

