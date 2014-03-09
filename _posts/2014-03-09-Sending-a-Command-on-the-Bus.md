---
layout: page
title: "Sending a Command on the Bus"
category: doc
date: 2014-03-09
order: 3
---

##Why

In a distributed system, you may want to send a command to another part of your system to tell it to do something. This something may be to "Send an email", "Publish a document" or "Update a Customer".

##The pattern

The pattern is called Command / Consumer, or Command and Competing Consumer. Competing because we could have multiple applications processing these commands so we can scale out. Because of the queue naming and listener configuration we guarantee that only one of the consuming services will get the command message.


With single consimer this is simple and in block diagrams it looks like this:

![](./images/CommandConsumer.png)

If we scale it out to multiple consumers it looks like this:


![](./images/CommandCompetingConsumer.png)


##How

To send the command on the bus we use the IBus.Send method. 
	
	var command = new SendEmailCommand {Recipient= "me@domain.com", Subject="You've got mail", Body="Dear Sir...."};
    bus.Send(command);

In this example we've created a SendEmailCommand class which implments the IBusCommand interface.


##Handling it

To handle a command, we implement the IHandleCommand interface in our Email Service application.

	public class EmailHandler : IHandleCommand<SendEmailCommand>
    {
        public async Task Handle(SendEmailCommand busCommand)
        {
            //Do stuff here, e.g. new up SmtpClient and send email
        }
    }

If we're using an IoC container in our application and Nimbus is registered with it, we can inject constructor parameters in this handler class too.

##Scaling it

To scale the handling of these messages across multiple instances, we just add multiple instances. Nimbus creates a message queue per Command type, so multiple instances will read from the same queue.	



##Delayed Sending

A handy tool we have is to delay the sending of a command until a later time. If we know we need to trigger actions at a certain time we can send a delayed command rather than having to build a storage and timing infrastructure in our app.

To do this we have a couple of methods on our bus. SendAt() and SendAfter(). SendAt takes a DateTimeOffset to specify a specific time and SendAfter takes a TimeSpan to specify a time from now.