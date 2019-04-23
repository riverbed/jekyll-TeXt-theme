---
title: Configuring OSPF on SteelHead SD
tags: SD-WAN
author: Brandon Carroll
---

Hi there.  I'm Brandon Carroll, one of the Technical Evangelists at Riverbed.  In this post I'm going to show you how to configure OSPF on a SteelHead SD.  You're not going to configure OSPF directly on the device, because a SteelHead SD is an SD-WAN Gateway.  So, in true form for a "Software Defined" solution, we are going to do everything on the controller.  Our controller is cloud-based more than 90% of the time.  Rather than refer to it as a controller, I'm going to refer to it as SteelConnect Manager, or SCM for short.

## OSPF Capabilities

One of the things that I find interesting in the world of SD-WAN is that not everything that you can do on a router is required.  Sure, we know all the knobs, but SD-WAN is redefining how we do things and much of what would require tuning should, in theory, be handled by the controller.  So your feature set for protocols like OSPF _should_ be limited if you compare SD-WAN capabilities against a traditional router.  __This is _ok_!__

We need to get away from thinking that an SD-WAN solution should function the same way as a traditional router.  If that were the case we would make no progress. Change is inevitable and we must embrace it.
{:.info}

So let me just share a few of the things that are supported on the Riverbed SD-WAN SteelHead SD gateways.

1. Multi-area OSPF
2. OSPF ASBR
3. Filtering with route-maps
4. Modify _hello and dead intervals, priority, and cost_
5. Standard, Stub, and Totally Stub Areas
6. Filter _inbound_, and _outbound_ prefixes
7. Define sumamrized routes for different sets of address ranges to be advertized
8. Define sumamrized routes for different sets of address ranges _not_ to be advertized
9. Password authentication per interface
10. Redistribute and modify the default metric
11. Originate a default route
12. Redistribute BGP into OSPF with metric control, metric selection (E1.E2), route tagging, and filtering with route-maps
13. Redistribute static and overlay routes into OSPF
14. Redistribute connected networks into OSPF
15. Perform Summarization

As you can see, there is pretty decent support for OSPF functions in the Riverbed SD-WAN solution.  And now that you know what's supported, let's have a look at how you configure OSPF.  

## Configuring OSPF

For the purposes of this article let's use the following diagram as a reference.

![OSPF Topology](http://drop.rvbd-te.com/2019-04-22_23-24-20.png)

The first thing we need to do is __Add OSPF Network__ and you'll find it by navitating to __Routing>OSPF__.

![Add OSPF Network](http://drop.rvbd-te.com/2019-04-22_23-31-06.png)

When you add an OSPF network you are presented with the options to define the site you are working in, the name for the area (which will auto-populate), the area ID and so on.  Enter the values that you need to neighbor up with a router that's also running OSPF.  

In this case, we need to switch to our HQ site, and configure _Area 1_.  Then we can click __Submit__.

![OSPF-area-1](http://drop.rvbd-te.com/2019-04-22_23-36-20.png)

Once the area is configured we need to attach an interface to it.

![Attach-interface](http://drop.rvbd-te.com/2019-04-22_23-41-39.png)

Select the OSPF area that you just created, inherit or modify any values necessary.

![select-areas](http://drop.rvbd-te.com/2019-04-22_23-43-02.png)

> Neighbor up!

At this point, assuming their is a route on the users zone, you should neighbor up and receive routes. <sup>[1](#zones)</sup>

You can check your routes in SCM by navigating to __Health Check>Routing Tables>OSPF__.

Select the HQ site gateway.  We can already tell from the output that the neighbor relationship has been established.

![ospf-routes](http://drop.rvbd-te.com/2019-04-22_23-48-33.png)

Looking a bit further into things you can see that the OSPF state is __Full/DR__ which corresponds to standard OSPF functionality.  We are of course in area 1.  We have also learned three routes from OSPF.  These routes are on the inside network if you refer back to the topology.

![ospf-route-detail](http://drop.rvbd-te.com/2019-04-22_23-51-18.png)

Going back to __Routing>OSPF__ and clicking on the __HQ-ospf__ configuration we can now enable the redistribution of BGP into OSPF.  The default metric type is _Type-2_ but in this examplel I have changed it to a _Type-1_, and IO have also applied a route-map that I already created called MyMap.

![redistribute-bgp-to-ospf](http://drop.rvbd-te.com/2019-04-22_23-53-51.png)

If you're curious what the route-map is doing, here is a look at the route-map configuration.  It's matching a _prefix-list_ called **DCONLY**.

![route-map](http://drop.rvbd-te.com/2019-04-22_23-55-32.png)

And the **DCONLY** prefix list is allowing one prefix, __172.17.0.0/24__ to be redistributed into OSPF from BGP.

![prefix-list](http://drop.rvbd-te.com/2019-04-22_23-56-50.png)

Remember that BGP routing tables can be very large and it is not recommended to redistribute full BGP routing tables into OSPF.
{:.warning}

Let's jump away from SCM to see what our router sees.  I'm using a Vyos router and on it I'll issue the __show ip ospf database__ command to see if we are getting the redistributed route.  And as we can see below, the route is coming in as an E1, which is what we expect.

```
vyos@Boston-RTR:~$ sh ip ospf database 

       OSPF Router with ID (172.30.3.1)

                Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
172.16.1.1      172.16.1.1       977 0x80000106 0xea3b 1
172.30.3.1      172.30.3.1      1575 0x80000116 0x1d4e 4

                Net Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum
172.16.1.2      172.30.3.1      1575 0x8000030c 0xf53a

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
172.17.0.0      172.16.1.1       975 0x80000001 0x565f E1 172.17.0.0/24 [0xffff]

vyos@Boston-RTR:~$ 
```

## Take Aways

This is just a small example of how to configure complex routing features easily with Riverbed's SteelConnect Manager.  I'm not trying to say that this method makes the complexity go away, you still need to understand what OSPF needs to function properly, however the presentation of the configuration in the SCM GUI makes it easy to setup and when you do this at scale it's a much better option that SSHing into a bunch of devices and configuring them independently.

That's it for this article.  Stay tuned for more technical articles that highlight the technical features of Riverbed solutions.

If you'd like to learn more about Riverbed SD-WAN please visit [Riverbed.com](https://www.riverbed.com/solutions/sd-wan.html) where you'll find webinars, analyst reports, customer stories, and more.  There you'll have an option to request a __Free Trial__ in the upper-right portion of the web page.

Happy Labbing Everyone!







<a name="zones">1</a>: A zone is an internal subnet.  Think of it as the inside network where users are connected, or where an internal router would be that can reach user subnets.  