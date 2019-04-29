---
title: A Beginners Guide to Deploying Riverbed SD-WAN
tags: SD-WAN
author: Brandon Carroll
---



I've been with Riverbed for a little over a year now, and before that I spent a lot of time looking at different SD-WAN vendors.  One thing that I found was that the complexity of a networking deployment still existed to a large degree despite throwing around the term "Zero-Touch-Provisioning."  In reality, there is a great deal of pre-work that needs to be done in architecting a network, but the tools we use shouldn't make it more difficult than it needs to be.

In this article I'll cover how to deploy the Riverbed SD-WAN solution from a high level.  So if you're a beginner, this is where you want to start.  I'll explain some of the major components of the solution and how they fit together, as well as how to add a new site and configure the gateway before you even ship it there.

Let's begin by laying out what the network looks like from the perspective of our Dashboard in SteelConnect Manager (SCM).


![overall](http://drop.rvbd-te.com/2019-03-21_13-12-08.png)

Here's what we will cover if you want to jump to a specific section:

- [SteelConnect Beginner - Sandbox Full Mesh](#steelconnect-beginner---sandbox-full-mesh)
	- [The Customer](#the-customer)
	- [Getting Started](#getting-started)
	- [Organizations)(#organizations)
	- [Sites](#sites)
	- [WANs](#wans)
	- [Uplinks](#uplinks)
	- [Zones](#zones)
	- [Appliances](#appliances)
		- [Shadow Appliance](#shadow-appliance)
	- [Conclusion](#conclusion)



## The Customer

This network belongs to a fictitious business known as SparkyPlugs.com.  For purposes of this article, SparkyPlugs is a global organization that provides specialized automotive parts for the consumer, the airline industry,  and the Department of Defense.  They have long held contracts with manufacturers to be the exclusive distributor of specific parts used in military drones, commercial aircraft, and souped up cars housed in garages with some of the finest of collectible automobiles.  To provide their customers with timely delivery, SparkyPlugs.com has several warehouses around the globe stocked full of parts that need to be shipped in very short periods of time, some within the same day.  They house their public server in Azure and when purchases are made the inventory at each warehouse is tracked.

## Getting Started

To begin, let's take a bit of a tour of SteelConnect Manager, or SCM as we call it.  When you log in you'll be looking at the Dashboard.  This can be seen in the image above.  What you're seeing here is a map of my sites, green lines indicating VPN tunnels between the sites.  This is a full mesh topology that we are looking at today, and all of these sites are connected **only** to the Internet.  There is no MPLS network in place at SparkyPlugs at this time.

These sites have some data that needs to be passed direct between one another as well as between them and HQ, for example we are using a VOIP solution and we want to encrypt the communication between sites, and we also have inventory information that each site can lookup but it's centrally stored and managed at HQ.  What's amazing about SparkyPlugs deployment is that it didn't take a large army of Networking people to implement this.  No.  In fact, they never had to send a single IT engineer from corporate to *any* of these sites.  They simply shipped the gateway and provided instructions for each warehouse manager to connect the WAN port to the Internet router and the LAN port to the switch.

Now that's not to say that nobody configured the network.  Sure, there's automation that helps with that, but lets be realistic, we still need to manage the network, architect the network, obviously there's though and *pre-work* involved.  But SparkyPlugs did all of this with one network engineer sitting in the HQ office in Boston, Massachusets.  They started by deploying a gateway there in Boston, and then branched out from there.

## Organizations

At the top level is an Organization, or Org.  There are several settings that you can define at the Org level that applies globally.  Think of this as the Parent settings.  Anything more specifically configured under the parent is usually going to take precedence.  Some of the things you can configure here are the Internet breakout preferences, the WAN usage preferences and logging configurations.

## Sites

In SCM a site would be next in the tree.  A site is a location that get's pinned on the map.  You can have more than one gateway in a site, usually for HA purposes.  Have a look here at the sites we have deployed right now.

![site](http://drop.rvbd-te.com/2019-03-21_13-47-28.png)

As you can see, Boston is our HQ site.  Clicking a site opens up its settings.

![siteconfig](http://drop.rvbd-te.com/2019-03-21_13-49-48.png)

Now when you think of a site you think of a building perhaps.  That building has a local network with a router that gets the users to the Internet.  Now the local network is what we call a **Zone**.  It's really more like a subnet that's connected to the gateway on the "inside" of the network.  This means we can have several zones inside of a site.  The gateway that we deploy in our site is the "router" that gets our traffic to the Internet, but it also connects our local users to other sites through automatically establised VPN tunnels.  We call these **AutoVPNs**.  An AutoVPN connects to other sites on the "outside" of our gateway, outside of our local network.  AutoVPN's need an **Uplink** and Uplinks are connected to a **WAN**.  In fact, an uplink is the physical connection that connects to a WAN.  Think of a WAN as a provider network.  Multiple uplinks can connect to a single WAN for redundancy.

Have a look at the [Layer 2 access port Branch topology](branchtopohttps://support.riverbed.com/bin/support/static/bk3e4nsvev67aokj3qg5gtcfv4/html/ulhrppo7aoojclf7jbgjnlji07/scm_2.11_dg_html/index.html#page/scm_2.11_dg_html%2Fbranch_topology_sdi.html%23ww1136125logy) found in the SteelConnect Deployment guide and you'll see what I mean.

You can see this depicted in simple terms below.

![simple](http://drop.rvbd-te.com/Screenshot-of-Microsoft-PowerPoint-3-21-19-2-12-48-PM-.png)

Lets walk through the process of building a new site.

1. Click "Add Sites."
2. Click "New Site."
3. Enter the details as seen below.

![siteL](http://drop.rvbd-te.com/2019-03-21_14-17-57.png)

As you can see, we are adding a new site in Brazil.  Entering the correct informaiton places the site on the map.  We can go back to the dashboard and see that we now have a site under construction.

![underconstruction](http://drop.rvbd-te.com/2019-03-21_14-20-37.png)

## WANs

Now recall that an uplink is attached to a WAN and thats what gets us to the Internet (or MPLS, or LTE, depending on the case). Let's have a look at the WAN's that exist.

As you can see, there is an Internet WAN, a RouteVPN WAN, and an MPLS WAN.  The MPLS isn't there by default, our admin created this so we can test down the road.  We can ignore this for now.

![WANs](http://drop.rvbd-te.com/2019-03-21_14-22-58.png)

The RouteVPN is here by default and you can't modify it.  It's a placeholder for the VPN connections.  Also, the Internet WAN is here and that's the one we are going to be concerned with.  Let's have a look at our Uplinks.

## Uplinks

Here's where the power of SteelConnect Manager and the automation behind the solution can really help.  Notice the output on the Uplinks page.  There is an uplink that was created for Brazil and we didnt have to do a thing to get it there.  Also note that it was connected to the **Internet WAN.**

![all-uplinks](http://drop.rvbd-te.com/2019-03-21_14-28-35.png)

When you click on the uplink you get a quick look at the state of the uplink.  Let's contrast this one with one that's already working like **Beijing**.

![brazilstate](http://drop.rvbd-te.com/2019-03-21_14-32-11.png)

As you can see, the Beijing uplink has been online for some time now, and we also see IP address information.  Let's return to our Brazil uplink and look at our Settings.

![beijinguplink](http://drop.rvbd-te.com/2019-03-21_14-33-38.png)

Here we would change the IP assignment to static addressing if we needed to, or we could even change the WAN that the uplink is attached to.

![brazil-ip](http://drop.rvbd-te.com/2019-03-21_14-36-45.png)

I'm going to leave this now and look at the Zone.

## Zones

A zone is where our users are connected.  We can have **guest** zones as well and control policy for them independently.

What you should notice here is that each zone is attached to a gateway, except for Brazil.  We don't have a gateway there yet.  However did you catch what's happening here?  We are configuring the network *before* the network exists in that location.  This is powerful.  There is no need to copy and paste config text files.  No need to ship a box to HQ, have IT console to it and configure it, and then ship it off to the location where it's to be installed.  No, we can do all of this ahead of time with SteelConnect Manager.

![zones](http://drop.rvbd-te.com/2019-03-21_14-40-16.png)

Let's examine the zone settings for Brazil. 

Again, this zone was configured for us by the automation.  Notice that there is an IP allocation for this zone.  If we are building from scratch this is very helpful.  If we are replacing an existing router then we will probably need to edit this to reflect the correct subnet. 

![zone1](http://drop.rvbd-te.com/2019-03-21_14-46-03.png)

We have a organizational level setting that defines the networking defaults.  This is where the 172.16.0.0/24 subnet that is assigned to our internal zone is coming from.  It's the next subnet available in the pool.  Also, the IP address 172.16.0.1 is the IP address assigned to the gateway.

Looking at the DHCP tab, we can see that the gateway is also acting as a DHCP server as well as a DNS server (This is important for application identification). If you have any special DHCP options that need to be included in the reponse to clients, you can include that here.

![dhcp](http://drop.rvbd-te.com/2019-03-21_14-58-31.png)

Ok, so we see the different elements that make up a site and enable connectivity for our users, one last thing we need to do here is deploy the gateway.  Now this is where things get really cool!

## Appliances

We need to navigate to list list of Appliances.

It's here that we see all the appliances that SparkyPlugs has deployed.  This is where you would see the actual hardware.  In this case I am using a demo environment with no physical hardware.  Instead we have deployed our virtual gateways.  These can be deployed on several platforms.  You can create one by clicking on **Add Appliances>Create Virtual Gateway**.  Lets do that just to see.  We can delete it when we are done looking at it.

First I'll create a virtual gateway and deploy it into the Brazil site.

![vg](http://drop.rvbd-te.com/2019-03-21_15-06-30.png)

Now you can see that a new appliance exists, but its offline.  Even so, the automation on the bottom created ports and such.

![newgw](http://drop.rvbd-te.com/2019-03-21_15-06-46.png)

Lets click on the Brazil gateway.

From here you can **build the image** for the platform you wish to deploy on.

![buildgw](http://drop.rvbd-te.com/2019-03-21_15-09-41.png)

Well we really don't need this virtual gateway, so I'll delete it by clicking **Actions>Delete**.

### Shadow Appliance

What we really want to do is add what we call a **Shadow Appliance.**  This gives us all the constructs of a physical appliance so we can build our configuration before the appliance is actually onsite.

So to add the shadow appliance I click **Add Appliance**, **Create Shadow Appliance**.

At this point we need to tell SCM what appliance SparlyPlugs will be using in that warehouse.

We click on **Models** and from there we notice that we have several models to choose from.  We are going to deploy an SDI-130 gateway at this site.  Note that we also have Steelhead appliances, known as SD appliances.  These would be approproate when we require advanced enterprise routing capabailities and want to perform WAN optimization in the same box.  For SparkyPlugs, the SDI platform is a perfect fit.

![gateways](http://drop.rvbd-te.com/2019-03-21_15-24-05.png)

To finish this up we select the SDI-330 gateway, select the brazil site, and click Submit.

At this point the automation added all the bits we need to build our appliance config.  We don't need to do much since there's not really any special routing happening and we only have one uplink to the Internet.  The one thing we do need to configure is the port assignments.  Lets have a look at the ports for the appliance that's going into Brazil.

We need to click on **Appliances>Ports** and then use the dropdown filter to select the Brazil site.

As we filtering on the brazil site, notice that the automation already did a bit of work for us.  This makes things as simple as possible.  Remember that technology should get out of our way.  So the automation attached the first WAN **Port** to our **Uplink** which is in turn connected to the **Internet WAN**.  It also attached the 8 LAN **Ports** to the **Zone** called **LAN[1013]**.

![ports](http://drop.rvbd-te.com/2019-03-21_15-44-44.png)

The last thing we need to do is ship the appliance and register it.  Each appliance has a serial number associated with it that begins with the letters **XN**.  You'll receive this in an email and it will also be on a label on the hardware.  Assuming that we have shipped the appliance, it's been received and we have verified the serial number with our branch it's time to register it.

So, we ship our appliance.  It arrives and we verify it with our local admin.  Then we click **Appliances>Overview**, select the Brazil Appliance, click **Actions**, and then **Register Hardware**.

In the final step you would simply paste the serial number into the pop-up window.

![serial](http://drop.rvbd-te.com/2019-03-21_15-53-11.png)


## Conclusion

At this point the onsite personell would connect the gateway, the gateway would obtain an IP address, call home, and register with core services.  Core services would look at the serial number and return information that tells it which SCM organization it belongs to.  The appliance would then register into your SCM and fetch its configuration from SCM.  At this point SCM controlls everything the applaince does.  

Normally this would take hours of work, whereas we can deploy a site like this in the real workd in just a fraction of the time.  There's obviously more to SteelConnect that we could discuss, but I'll wrap things up here.  I know I have a biased opinion, but that's ok.  The entire process you saw here was quite enjoyable.  At the end of the day I accomplished my job, and when the work day ends, it really ends.  I can go home to my family knowing that tomorrow will come with its new challenges, but today I get to go home at a decent time and not log into my laptop from home and troubleshoot a typo in 600 lines of Config.  To me, that's the essence of what the Human Experience should be about.  Technology serves us so we can enjoy the things that really matter.  

That's my take anyhow, but you don't have to take my word for it.  If you'd like to learn more about how Riverbed can enhance the Human Experience please visit [http://www.riverbed.com](http://www.riverbed.com).

