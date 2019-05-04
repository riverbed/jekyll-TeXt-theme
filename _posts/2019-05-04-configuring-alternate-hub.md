At times the situation may arise where there are two SD-WAN gateways connected to two separate WANs.  In cases like this you can use the *alternate hub* configuration to allow overlay routes from each site to be shared via a hub site that is connected to both WANs.  The network topology would resemble the following:




## Alternate Hub

In this next section we will configure the Alternate Hub Feature.  The adjustment to the network will resemble the following topology:

![New-Topo](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_17-11-53.png)

To begin, follow these steps:

1. On Site B turn off the MPLS Uplink.  This will make the site an internet-only site.
2. On Site A, enable internet breakout on the MPLS link.

![internet-breakout-mpls](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-04-17.png)

3. Next, Set the Intenet Breakout Preference to MPLS for Site A.

![wan-preference](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-09-31.png)

> Important:  At this point, wait for the gateway to be up-to-date.

![config-update](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-09-44.png)

4. Turn off the Internet Uplink on Site A.

![disable-internet](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-10-16.png)

5. Disable AutoVPN for the Internet Uplink of Site A.

![Disable-VPN](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-11-59.png)

6. Prior to configuring Alternate Hub, and after the gateways are up-do-date, check the Overlay Routes that are seen on each site, Both Site A and Site B.

![SiteB](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-26-59.png)

![SiteA](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-27-09.png)

> Notice that in each of these sites they only have the prefix from the Data Center.  To get an internet-only site to talk to an MPLS-only site when both are running SD-WAN gateways, enable the alternate hub feature on both Site A and Site B.

1. Enable Alternate Hub on Site A.

![Alt-hub-A](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-27-50.png)

2. Enable Alternate Hub on Site B.

![Alt-hub-B](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-28-06.png)

Now that you have enabled the alternate hub feature it will take a few minutes for everything to update.  Be patient.  Once all devices are "up-to-date" check the overlay routes again on both Site A and Site B to ensure they are learning each others prefixes.

### View Site B's Overlay Routing Table

![Site B](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-43-45.png)

### View Site A's Overlay Routing Table

![Site A](https://rvbdtech.sharepoint.com/teams/te/SiteAssets/SitePages/Branch-Deployment-Scenarios-Lab/2019-05-01_16-44-03.png)


## Return the lab to Full Mesh.

1. Change Site A's Internet Breakout preference back to default.
2. Remove the Alternate Hub on Site A.
3. Remove the Alternate Hub on Site B.



