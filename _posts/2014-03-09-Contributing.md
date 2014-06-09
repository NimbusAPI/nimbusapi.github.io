---
layout: page
title: "Contributing"
category: dev
order: 1
---


## Can I contribute?
Absolutely! This is very very very early days for this project. We need things
like:

1.  More container implementations
1.  Logger implementations
1.  Samples
1.  Real world input from your projects
2.  Success stories and case studies


We've put some issues here marked as [Up-for-grabs](https://github.com/NimbusAPI/Nimbus/issues?labels=up-for-grabs&page=1&state=open) if you want to jump in.


## Azure Service Bus connection string

The integration tests require a connection string to a (local or remote) Azure Service Bus connection. Assuming you have Service Bus installed locally:

1. Open the **Service Bus PowerShell**
2. Create a new namespace: `new-sbnamespace -Name NimbusIntegration -ManageUsers '[domain\]username`
3. Get the connection string: `get-sbclientconfiguration -Namespaces NimbusIntegration`
4. Paste the connection string into `c:\temp\NimbusConnectionString.txt`
5. 
