---
layout: page
title: "Configuring Nimbus With Autofac"
category: doc
date: 2014-03-09
order: 2
---

We've gone through this blow by blow in [the first section](./Getting-Started-With-Nimbus.html) so if none of this makes sense go back and have a read there, let's just talk about the Autofac bits.

Getting Autofac

First thing you need to do is pull down the Autofac provider. This is via NuGet of course.

	Install-Package Nimbus.Autofac

Now we configure Nimbus and register it with the container 
    
    var handlerTypesProvider = new AssemblyScanningTypeProvider(Assembly.GetExecutingAssembly());
    
    builder.RegisterNimbus(handlerTypesProvider);
    builder.Register(componentContext => new BusBuilder()
                         .Configure()
                         .WithConnectionString(connectionString)
                         .WithNames("MyApp", Environment.MachineName)
                         .WithTypesFrom(handlerTypesProvider)
                         .WithAutofacDefaults(componentContext)
                         .Build())
           .As<IBus>()
           .AutoActivate()
           .OnActivated(c => c.Instance.Start())
           .SingleInstance();

    var container = builder.Build();


##Breaking it down

###Registering the handler types 

	builder.RegisterNimbus(handlerTypesProvider);

There's an extension method that will register all of the various message handlers with the container. We call it with our handler type provider.

###Autofac registration
    builder.Register(componentContext => new BusBuilder()
                         .Configure()
                         .WithConnectionString(connectionString)
                         .WithNames("MyApp", Environment.MachineName)
                         .WithTypesFrom(handlerTypesProvider)
                         .WithAutofacDefaults(componentContext)
                         .Build())

This is the configuration call we've already seen with one difference. We've already registered our handlers with Autofac but we need to tell the container how to build and configure the bus. You can dig into the source if you want to see how it works, but the .WithAutofacDefaults call is what you'll need.

###Registering IBus

If you've used Autofac before you'll know you want to register the bus as an IBus so you can use it as a dependency in the rest of your code. We've registered it as SingleInstance of course because we only want one instance of the bus in our app.

###Starting the bus
Once the Bus is registered we want to start it. To do that we call .AutoActivate() in Autofac to instantiate Nimbus and use the OnActivated delegate to start the bus. This means that when your container is built, Nimbus will be started.

