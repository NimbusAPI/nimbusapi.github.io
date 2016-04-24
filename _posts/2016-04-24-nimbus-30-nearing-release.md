---
layout: post
title: Nimbus 3.0 nearing release
author: DamianM
---

Well the wait is nearly over, and welcome to the new Nimbus API website and domain!

In late 2015 a thread was started about the [state of Nimbus](https://github.com/NimbusAPI/Nimbus/issues/221). 
Nimbus faced a few technical problems because of dependencies on out of date Microsoft SDKs, and between that and time commitments and professional lives of myself and Andrew, Nimbus was feeling a little neglected.

We sat down before Christmas and decided that we really did want to push ahead with it, so refactored out the dependency on the old Azure SDK and BrokeredMessage.

We make the Windows Service Bus transports a pluggable component which would allow a new Azure Service Bus transport to be written to take advantage of the newer SDK versions.

While we were at it, we also built a new transport using Redis. Redis as a message platform is orders of magnitude faster than ASB and available as a PaaS offering on both Azure and AWS.

We'd hoped to have a release out in January, but I changed jobs and have been head down for the first part of this year.

So here we are. 

3.0 is pretty much ready to go, we've been using it on a few apps and it's working nicely.

To get started, you just need to add in an extra Nuget package for your Transport. The packages are avalable on our [MyGet feed](https://www.myget.org/gallery/nimbusapi). 

Then add your transport :

    Install-Package Nimbus.Transports.WindowsServiceBus
    
Configuration from 2.x to 3.0 is identical except for specifying the transport.

Firstly configure your transport:

    var transport = new WindowsServiceBusTransportConfiguration().WithConnectionString(connectionString);

Then add it to your Bus configuration:

    var bus = new BusBuilder().Configure().WithTransport(transport)
    
The rest is the same.

## Good Things ##

The biggest hurdle we had with the old implementation is Message Locks. We did a silly amount of work to renew locks for long running tasks and they could still bite. 
In 3.0 this has gone away. We complete the message as we pull it off. In the event of failure we push it back to the queue, but now with a backoff strategy.

The backoff strategy means that rather than it failing 5 times quickly and going to the poison queue, we'll delay further attempts at the message to allow a misbehaving third party service to recover.

## Breaking Change

There is one thing that is slightly different. In our old implementation, doing a delayed Send would be persisted on the Queue itself. Because of the new abstractions, we don't have this. 
Messages are held in memory until the Send time. It is going to work the same for most people most of the time. But if you were using SendAt to delay messages for a long time, this method currently won't survive a process restart. We're planning on changing this but if you wanted to use 3.0 today you might need an alternative persistence strategy for those messages.

## What's next ##

We want to release this as soon as possible and say goodbye to the 2.x codebase. So if you're interested in trying it out, go hit the MyGet packages and let us know if you find any issues.

Once we're happy to go we'll probably close off a lot of the issues and PRs on GitHub because they're not going to relevant anymore.
