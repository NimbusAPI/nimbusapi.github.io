---
layout: page
title: "Getting Started with Nimbus"
category: doc
date: 2014-03-09
order: 1
---

The first step is to get Nimbus from NuGet

	Install-Package Nimbus


## Configuring the bus without a container

If you don't want to use an IoC container (but we think you probably should) you can build and configure the bus like this.

	//TODO: Set up your own connection string in app.config
    var connectionString = ConfigurationManager.AppSettings["AzureConnectionString"];

    // This is how you tell Nimbus where to find all your message types and handlers.
    var typeProvider = new AssemblyScanningTypeProvider(Assembly.GetExecutingAssembly());

    var messageHandlerFactory = new DefaultMessageHandlerFactory(typeProvider);

    var bus = new BusBuilder().Configure()
                                .WithNames("MyApplication", Environment.MachineName)
                                .WithConnectionString(connectionString)
                                .WithTypesFrom(typeProvider)
                                .WithDefaultHandlerFactory(messageHandlerFactory)
                                .WithDefaultTimeout(TimeSpan.FromSeconds(10))
                                .Build();
    bus.Start();
    return bus;


### What does that all mean. Breaking it down ##

So that's a bunch of stuff there, let's break it down a bit.

####Connection Strings

    var connectionString = ConfigurationManager.AppSettings["AzureConnectionString"];
This is your connection string, the address of your Azure Service Bus namespace. You can get this from the Azure console. You'll need a connection string with Manage permissions. It's going to look something like this :

	Endpoint=sb://[your namespace].servicebus.windows.net;SharedSecretIssuer=owner;SharedSecretValue=[your secret]


####Type Providers 

	var typeProvider = new AssemblyScanningTypeProvider(Assembly.GetExecutingAssembly());


Next up is your type provider. This is what Nimbus uses to find all the messages you want to put on the bus and handle from the bus, as well as all your handlers. The default AssemblyScanningTypeProvider is what you most likely what you want here unless you want to do something funky. It will scan all the assemblies you pass in to it for any Events, Commands, Requests and their related Handlers so we can register them. The argument here is a param array of Assembles so you can have your messages and handlers in multiple assemblies (you will want this in any real world app) and pass them in as arguments

####Handler Factories

    var messageHandlerFactory = new DefaultMessageHandlerFactory(typeProvider);

The next line here is the Handler Factory. This is the class that will manage the instatiation of your message handlers. We'll show more examples of message handlers in later pages. The DefaultMessageHandlerFactory instantiates them through a basic Activator.CreateInstance call. This means your handlers can't have any constructor dependencies (if you don't know why you'd need an IoC container this is probably fine). 

When it's time to add an IoC container, this is the line that will change.

####Building the bus


    var bus = new BusBuilder().Configure()
                                .WithNames("MyApplication", Environment.MachineName)
                                .WithConnectionString(connectionString)
                                .WithTypesFrom(typeProvider)
                                .WithDefaultHandlerFactory(messageHandlerFactory)
                                .WithDefaultTimeout(TimeSpan.FromSeconds(10))
                                .Build();
    bus.Start();

Alright so this is the big one. We have a fluent interface for the configuration and instantiation of the bus. Most of the lines here are going to make sense now. WithConnectionString passes in the addressing of our Azure Namespace, we're passing in our Type Provider as aell as our Handler Factory and the Default Timeout should be self explainatory.

The only thing that needs explaining here is the WithNames line.

####Queue Naming

In order to address the messages properly we have a naming convention based on the application name, and for cases where we have multiple instances of the same app we also use a unique name, usually the machine name. So we're passing those params into the .WithNames arguments


Next up [Configuring Nimbus with Autofac](./configuring-nimbus-with-autofac.html)