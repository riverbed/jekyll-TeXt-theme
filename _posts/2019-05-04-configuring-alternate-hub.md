## eBGP Connectivity on the MPLS Uplink

By default, this lab is defined as a full mesh environment.  

1. On each branch configure BGP Peering to the MPLS cloud. 
2. Confirm that each branch is recieving the 172.17.0.0/24 prefix.

![BGP Prefix](https://rvbdtech.sharepoint.com/teams/te/SiteCollectionImages/SitePages/Level%203%20Bootcamp/HQ-routing-table-1.png)

## OSPF connecivity on the Users Zone of HQ

1. Enable OSPF area 1 on the __Users__ zone for HQ.
2. Confirm that you are recieving the prefixes __172.30.1.0/24__, __172.30.2.0/24__, and __172.30.3.0/24__ from the __Boston-RT__.

![OSPF-Routes](https://rvbdtech.sharepoint.com/teams/te/SiteCollectionImages/SitePages/Level%203%20Bootcamp/2019-04-17_17-09-31.png)


3. Log into the Boston-RTR.

* SSH to the IP address provided by the traininer for your Pod.  
* Authenticate with the user console with the username **console** and password **router**.  
* Once presented with the Menu select the **bs_rtr** and log into the Vyos router with the username/password __vyos/vyos__.

```
1) cf3-phx  3) cf3-den	5) mpls	    7) bs_rtr	9) quit
2) cf3-bs   4) cf3-sfo	6) phx_rtr  8) brn_rtr
Select server?7
Opening console in /tmp/bs_rtr.sock

Welcome to VyOS - Boston-RTR ttyS0

Boston-RTR login: vyos
Password: 
Linux Boston-RTR 4.19.12-amd64-vyos #3 SMP Wed Jan 23 18:15:21 UTC 2019 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
vyos@Boston-RTR:~$ 
```


4. Does the Boston-RTR have the __172.17.0.0/24__ prefix in it's routing table?

```
vyos@Boston-RTR:~$ show ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O   172.16.1.0/24 [110/1] is directly connected, eth1, 00:07:34
C>* 172.16.1.0/24 is directly connected, eth1, 01w3d09h
C>* 172.17.1.0/24 is directly connected, eth0, 01w3d09h
O   172.30.1.0/24 [110/10] is directly connected, dum0, 00:07:35
C>* 172.30.1.0/24 is directly connected, dum0, 01w3d09h
O   172.30.2.0/24 [110/10] is directly connected, dum1, 00:07:35
C>* 172.30.2.0/24 is directly connected, dum1, 01w3d09h
O   172.30.3.0/24 [110/10] is directly connected, dum2, 00:07:35
C>* 172.30.3.0/24 is directly connected, dum2, 01w3d09h
vyos@Boston-RTR:~$ 

```

## Enable the ASBR feature in OSPF

1. Enable redistribution of BGP into OSPF area 1.
2. Ensure that you have the __172.17.0.0/24__ prefix in it's routing table on the Boston-RTR.

```
vyos@Boston-RTR:~$ sh ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route

O   172.16.1.0/24 [110/1] is directly connected, eth1, 05:49:23
C>* 172.16.1.0/24 is directly connected, eth1, 01w3d15h
O>* 172.17.0.0/24 [110/2] via 172.16.1.1, eth1, 00:10:55
C>* 172.17.1.0/24 is directly connected, eth0, 01w3d15h
O   172.30.1.0/24 [110/10] is directly connected, dum0, 05:49:24
C>* 172.30.1.0/24 is directly connected, dum0, 01w3d15h
O   172.30.2.0/24 [110/10] is directly connected, dum1, 05:49:24
C>* 172.30.2.0/24 is directly connected, dum1, 01w3d15h
O   172.30.3.0/24 [110/10] is directly connected, dum2, 05:49:24
C>* 172.30.3.0/24 is directly connected, dum2, 01w3d15h
vyos@Boston-RTR:~$ 
```


What type of LSA does the __172.17.0.0/24__ prefix appear as on the Boston-RTR?

```
vyos@Boston-RTR:~$ show ip ospf database

       OSPF Router with ID (172.30.3.1)

                Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
172.16.1.1      172.16.1.1        82 0x80000010 0xd943 1
172.30.3.1      172.30.3.1      1305 0x80000014 0x244a 4

                Net Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum
172.16.1.2      172.30.3.1      1385 0x8000020c 0xf838

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
172.16.2.0      172.16.1.1        80 0x80000001 0xcf64 E2 172.16.2.0/24 [0xffff]
172.16.3.0      172.16.1.1        80 0x80000001 0xc46e E2 172.16.3.0/24 [0xffff]
172.17.0.0      172.16.1.1        80 0x80000001 0xd95b E2 172.17.0.0/24 [0xffff]
172.30.0.4      172.16.1.1        80 0x80000001 0x0324 E2 172.30.0.4/30 [0xffff]
172.30.0.8      172.16.1.1        80 0x80000001 0xda48 E2 172.30.0.8/30 [0xffff]

vyos@Boston-RTR:~$ 
```

What is the default method of redistribution into OSPF (What LSA type will the prefix be)?

__E2__

> The cost of E2 routes will always be the external metric, the metric will take no notice of the internal cost to reach that network.  The cost will not increment as the route is advertised in the OSPF routing domain.


Can these type of LSA's be filtered by our SD-WAN Gateway?

1. Change the redistribution to type 1.

What changed in OSPF?

```
vyos@Boston-RTR:~$ show ip ospf database

       OSPF Router with ID (172.30.3.1)

                Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
172.16.1.1      172.16.1.1       927 0x80000010 0xd943 1
172.30.3.1      172.30.3.1       450 0x80000015 0x224b 4

                Net Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum
172.16.1.2      172.30.3.1       540 0x8000020d 0xf639

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
172.17.0.0      172.16.1.1         1 0x80000003 0xd55d E1 172.17.0.0/24 [0xffff]

vyos@Boston-RTR:~$ 

```

The route is now an E1.

> The cost of E1 routes is the cost of the external metric with the addition of the internal cost within OSPF to reach that network.  In other words, the metric will increment as it is propagated through the OSPF routing domain.

Can these LSA's be filtered by our SD-WAN Gateway?

1. At SiteA, redistribute connected to BGP.

What prefixes how show up in the Boston-RTR?

```
vyos@Boston-RTR:~$ sh ip ospf data

       OSPF Router with ID (172.30.3.1)

                Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
172.16.1.1      172.16.1.1      1642 0x8000002b 0xa35e 1
172.30.3.1      172.30.3.1      1610 0x80000031 0xe967 4

                Net Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum
172.16.1.2      172.30.3.1      1690 0x80000229 0xbe55

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
172.16.2.0      172.16.1.1        37 0x80000001 0x4c68 E1 172.16.2.0/24 [0xffff]
172.17.0.0      172.16.1.1       343 0x80000009 0x4667 E1 172.17.0.0/24 [0xffff]
172.30.0.4      172.16.1.1        37 0x80000001 0x7f28 E1 172.30.0.4/30 [0xffff]

vyos@Boston-RTR:~$ 

```

There are two new prefixes that came in as a type 2 LSA.  

1. Prevent the Boston-RTR from recieving the two new prefixes with a configuration that only applies to the HQ gateway.

![Prefix-list](https://rvbdtech.sharepoint.com/teams/te/SiteCollectionImages/SitePages/Level%203%20Bootcamp/2019-04-17_23-12-15.png)

![Route-Map](https://rvbdtech.sharepoint.com/teams/te/SiteCollectionImages/SitePages/Level%203%20Bootcamp/2019-04-17_23-12-28.png)

![Apply-to-redistribution](https://rvbdtech.sharepoint.com/teams/te/SiteCollectionImages/SitePages/Level%203%20Bootcamp/2019-04-17_23-12-41.png)

After a few minutes, verify on the Boston-RTR.

```
vyos@Boston-RTR:~$ sh ip ospf data

       OSPF Router with ID (172.30.3.1)

                Router Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum  Link count
172.16.1.1      172.16.1.1         9 0x8000002c 0xa15f 1
172.30.3.1      172.30.3.1        17 0x80000032 0xe768 4

                Net Link States (Area 0.0.0.1)

Link ID         ADV Router      Age  Seq#       CkSum
172.16.1.2      172.30.3.1        97 0x8000022a 0xbc56

                AS External Link States

Link ID         ADV Router      Age  Seq#       CkSum  Route
172.17.0.0      172.16.1.1       460 0x80000009 0x4667 E1 172.17.0.0/24 [0xffff]
```

### Checkpoint!

What have we accomplished so far?

* Configured BGP peers on the branch routers to peer with the MPLS network.
* Configured OSPF in the Boston HQ branch.
* Configured redistribution from BGP into OSPF.
* Configured redistribution of OSPF into BGP.
* Configured redistribution of connected networks into BGP at siteA.
* Configured route filtering on redistribution into OSPF at the Boston HQ.



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



