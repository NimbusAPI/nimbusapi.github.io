---
layout: page
title: "Configuring Nimbus With Windsor"
category: doc
date: 2014-03-09
order: 3
---


We've gone through this blow by blow in [the first section](./getting-started-with-nimbus.html) so if none of this makes sense go back and have a read there, let's just talk about the Autofac  bits.

Getting Windsor

First thing you need to do is pull down the Windsor provider. This is via NuGet of course.

	Install-Package Nimbus.Windsor

Now we configure Nimbus and register it with the container 


    var typeProvider = new AssemblyScanningTypeProvider(Assembly.GetExecutingAssembly());

    container.RegisterNimbus(typeProvider);
    container.Register(Component.For<IBus>()
                                .ImplementedBy<Bus>()
                                .UsingFactoryMethod<IBus>(() => new BusBuilder()
                                                                .Configure()
                                                                .WithConnectionString(connectionString)
                                                                .WithNames("PingPong.Windsor", Environment.MachineName)
                                                                .WithTypesFrom(typeProvider)
                                                                .WithWindsorDefaults(container)
                                                                .Build())
                                .LifestyleSingleton()
                                .StartUsingMethod("Start")
        );


##Breaking it down

###Registering the handler types 

	container.RegisterNimbus(handlerTypesProvider);

There's an extension method that will register all of the various message handlers with the container. We call it with our handler type provider.

###Windsor registration
    container.Register(Component.For<IBus>()
                                .ImplementedBy<Bus>()
                                .UsingFactoryMethod<IBus>(() => new BusBuilder()
                                                                .Configure()
                                                                .WithConnectionString(connectionString)
                                                                .WithNames("PingPong.Windsor", Environment.MachineName)
                                                                .WithTypesFrom(typeProvider)
                                                                .WithWindsorDefaults(container)
                                                                .Build())
                                .LifestyleSingleton()
                                .StartUsingMethod("Start")
        );

This is the configuration call we've already seen with one difference. We've already registered our handlers with Windsor but we need to tell the container how to build the configure the bus. You can dig into the source if you want to see how it works, but the .WithWindsorDefaults call is what you'll need.

###Registering IBus

If you've used Windsor before you'll know you want to register the bus as an IBus so you can use it as a dependency in the rest of your code. We've configured it as LifestyleSingleton because we only want one instance of the bus in our app.

###Starting the bus
Once the Bus is registered we want to start it. To do that we call .StartUsingMethod() in Windsor to instantiate Nimbus by calling the "Start" method.

