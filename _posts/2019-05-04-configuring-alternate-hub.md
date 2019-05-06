At times the situation may arise where there are two SD-WAN gateways connected to two separate WANs.  In cases like this you can use the *alternate hub* configuration to allow overlay routes from each site to be shared via a hub site that is connected to both WANs.  The network topology would resemble the following:


![topo](http://drop.rvbd-te.com/Photo-2019-05-05-07-34.PNG)

## Alternate Hub

In this next section we will configure the Alternate Hub Feature.  The adjustment to the network will resemble the topology show above.  You wont necessarily need to make this adjustment every time, but since my network is configured as a full mesh network I'll tweak it so that two of my sites are on separate WANs, one on MPLS only, and one on Internet only.

To begin, follow these steps:

1. On Site B turn off the MPLS Uplink.  This will make the site an internet-only site.
2. On Site A, enable internet breakout on the MPLS link.

![internet-breakout-mpls](http://drop.rvbd-te.com/Photo-2019-05-05-08-07.jpg)

3. Next, Set the Intenet Breakout Preference to MPLS for Site A.

![wan-preference](http://drop.rvbd-te.com/Photo-2019-05-05-22-49.jpg)

Important:  At this point, wait for the gateway to be up-to-date.
{:.warning}


![config-update](http://drop.rvbd-te.com/Photo-2019-05-05-22-53.jpg)

4. Turn off the Internet Uplink on Site A.

![disable-internet](http://drop.rvbd-te.com/Photo-2019-05-05-22-54.jpg)

5. Disable AutoVPN for the Internet Uplink of Site A.

![Disable-VPN](http://drop.rvbd-te.com/Photo-2019-05-05-22-55.jpg)

6. Prior to configuring Alternate Hub, and after the gateways are up-do-date, check the Overlay Routes that are seen on each site, Both Site A and Site B.  In the following screenshot from site a notice the overlay routes that you see from the Data Center are only the prefixes that are located in the Data Center. 

![SiteA](http://drop.rvbd-te.com/Photo-2019-05-05-22-56.jpg)

Now notice Site B.  Site B only has prefixes from the Data Center as well.  

![SiteA](http://drop.rvbd-te.com/Photo-2019-05-05-22-57.jpg)

Each of these sites only has the prefix from the Data Center.  We conformed this above.  Our goal is to use the Data Center as a hub, since it knows the prefixes from both Site A and Site B.  By doing so, we can get an internet-only site to talk to an MPLS-only site as long as both of these sites are running SD-WAN gateways.  Let's enable the alternate hub feature on both Site A and Site B.  In both cases we will define the Data Center as the alternate hub.

1. Enable Alternate Hub on Site A.

![Alt-hub-A](http://drop.rvbd-te.com/Photo-2019-05-05-23-01.jpg)

2. Enable Alternate Hub on Site B.

![Alt-hub-B](http://drop.rvbd-te.com/Photo-2019-05-05-23-01.jpg)

Now that you have enabled the alternate hub feature it will take a few minutes for everything to update.  Be patient.  Once all devices are "up-to-date" check the overlay routes again on both Site A and Site B to ensure they are learning each others prefixes.

### View Site B's Overlay Routing Table

![Site B](http://drop.rvbd-te.com/Photo-2019-05-05-23-02.PNG)

### View Site A's Overlay Routing Table

![Site A](http://drop.rvbd-te.com/Photo-2019-05-05-23-02.PNG)


## Closing Comments

As we've seen in this article, the Alternate hub feature is a simple way to propagate overlay routes from two different SD-WAN gateways connected to two different WANs.  This can be especially helpful during a migration or in the interim between adding internet to an MPLS-only site.  Using this feature ensures connectivity within your SD-WAN fabric.  Using the power of SteelConnect manager it's not difficult to do.  And this is just one feature of many that are part of the Riverbed SD-WAN solution, which enabled you to deploy an SD-WAN solution in a hybrid network with multiple WAN's.  

Stay tuned for more tips on how to configure various features in SteelConnect Manager.

Happy Labbing!
