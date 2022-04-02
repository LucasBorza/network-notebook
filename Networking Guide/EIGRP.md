## EIGRP Index

The term distance vector is derived from a list (vector) of distances and directions to destinations. EIGRP or Enhanced Interior Gateway Routing Protocol is defined as an advanced distance vector routing protocol.

EIGRP in and of its self is a Hybrid routing protocol which has characteristics of both a distance vector and link state protocol. Much like RIP using the triggered feature, EIGRP updates are only sent when a change in the network is determined. At first EIGRP routers will form a neighbor relationship and exchange the topological information. After which the routing protocol will send periodic hello's to ensure that the neighbor is still there. However when a link goes down or a route changes, updates are then sent to neighboring routers via multicast 224.0.0.10 using its own IP protocol number 88.

## EIGRP

**Common Distance Vector Characteristics**

-   Periodic updates

-   Reliance on neighbors to propagate advertisements

-   Broadcast updates

-   Full routing table updates (advertising the entire table every time)

 

**EIGRP Operation**

-   EIGRP is an advanced version of Cisco\'s proprietary IGRP. EIGRP runs the Diffusing Update Algorithm (DUAL) instead of Bellman-Ford to attain fast convergence while remaining loop-free. Protocol-dependent modules are used to support protocols other than IP (such as IPX and AppleTalk).

-   Reliable Transport Protocol (RTP) provides ordered delivery of packets between EIGRP neighbors.

-   EIGRP can consider bandwidth, delay, reliability, and load in calculating a metric; only bandwidth and

> delay are considered by default.
>
>  

**Attributes**

**Type:** Distance Vector\
**Algorithm:** DUAL

**Internal AD:** 90

**External AD:** 170

**Summary AD:** 5

**Standard:** Cisco proprietary

**Protocols:** IP

**Transport:** IP/88

**Authentication:** MD5

**Multicast IP:** 224.0.0.10

**Hello Timers:** 5/60

**Hold Timers:** 15/180

 

**Packet Types**\
**Hello** - Peer discovery and maintenance

**Acknowledgment** - Empty hello packets used to acknowledge messages

**Update** - Convey route information

**Query -** Request for a route

**Reply** - Answer to a query

 

##Terminology

**Autonomous System:** A collection of multiple networking devices under the control of a single or multiple entity which share a common routing policy for the network. However you can have multiple autonomous systems under the control of the same organization for example; multiple facilities or sites nation or world wide interconnected but segregated for management **Reported Distance:** The metric for a route advertised by a neighbor

**Feasible Distance:** The distance advertised by a neighbor plus the cost to get to that neighbor.

**Stuck in Active (SIA):** The condition when a route becomes unreachable and not all queries for it are answered; adjacencies with unresponsive neighbors are reset.

**Passive Interface:** An interface which does not participate in EIGRP but whose network is advertised.

**Stub Router**: A router which advertises only a subset of routes, and is omitted from the route query process.

 

**DUAL / Finite State Machine**

Of all routes to a destination, the one with the lowest metric will be designated the successor route.

The feasibility condition states that a route\'s advertised distance must be less than the router\'s feasible distance to a destination for it to be considered a feasible successor, to avoid creating a routing loop. Traffic will be load-balanced across multiple paths with the same feasible distance.

 

**DUAL Finite State Machine**

-   Routes remain in the passive state while no DUAL calculations are being performed.

-   An input event such as an interface state transition or the reception of an update, query, or reply packet, will cause the router to recalculate the distance for all its feasible successors for the affected route.

-   If a feasible successor cannot be found, the route enters the active state, and queries for a new path are issued to EIGRP neighbors.

-   If a router does not receive responses from all neighbors to which it issued queries within the active timer (3 minutes by default), the route is considered stuck in active, and unresponsive neighbors are removed from the neighbor table. If all replies are received, the router calculates the feasible

>  

**Neighbor Relationship Requirements**

The following requirements must be met for an EIGRP neighbor relationship to form:

 

-   Both routers must be in the same primary subnet

-   Both routers must be configured to use the same k-values

-   Both routers must in the same AS

-   Both routers must have the same authentication configuration

-   The interfaces facing each other must not be passive

 

**NOTE:** Although a neighbor relationship can form with the following requirements being met, without some kind of traffic being generated excluding traffic sourced from EIGRP, routing information will not be distributed.

 

**EIGRP Tables**

**Neighbor table**

Stores information about neighboring EIGRP routers:

-   Network address (IP)

-   Connected interface

-   Holdtime - how long the router will wait to receive another HELLO before dropping the neighbor; default = 3 \* hello timer

-   Uptime - how long the neighborship has been established

-   Sequence numbers

-   Retransmission Timeout (RTO) - how long the router will wait for an ack before retransmitting the packet; calculated by SRTT

-   Smooth Round Trip Time (SRTT) - time it takes for an ack to be received once a packet has been transmitted

-   Queue count - number of packets waiting in queue; a high count indicates line congestion

>  

**Topology table**

Holds all routes received from neighbors, is built from updates, calculated by DUAL, and contains all the

information required by the routing table

 

**Routing table**

Route types:

Internal - Paths directly within EIGRP

Summary - Internal paths which have been summarized

External - Routes redistributes into EIGRP

 

**Split horizon**

Prevents routing loops by preventing the readvertisement of a route to the neighbor from which it was learned. Router A will not advertise routes learned from router B back to router B.

 

Poison reverse extends the concept of spit horizon by readvertising a learned route back to the

neighbor with an infinite metric. Router A will advertise routes learned from router B back to router B

with an infinite metric, ensuring router B knows said routes are not reachable via router A.

 

Metric, Timers and K-Values

Saturday, March 05, 2011

6:15 PM

 

**Overview**

The EIGRP metric is calculated by a formula using five separate values known as K Values. By default only K Values 1 and 3 are used (Bandwidth & Delay), K2, K4 and K5 are set to 0. The EIGRP metric formula and K Values are defined below.

 

**EIGRP Metric**

Metric = 256 \* (K1 \* bandwidth + ( (K2 \* bandwidth) / (256 - load) ) + K3 \* delay) \* ( K5 / (reliability + K4))

 

**K values are used to distribute weight to different path aspects**

-   K1 - bandwidth - Defined as 107 divided by the speed of the slowest link in the path, in Kbps

-   K2 - load - 8-bit value, not considered by default

-   K3 - reliability - 8-bit value, not considered by default

-   K4 - delay - constant value associated with interface type; EIGRP uses the sum of all delays in the path

-   K5 - Maximum Transmission Unit (MTU)

>  

K defaults: K1 = 1, K2 = 0, K3 = 1, K4 = 0, K5 = 0

 

**Adjust K Values to manipulate metric formula**

Router(config-router) metric weights 0 *\<k1\> \<k2\> \<k3\> \<k4\> \<k5\>*

 

**NOTE:** The zero in the command above references to the tos - type of service. The original intention was to have IGRP perform types of service routing. Because this was never adopted into practice, the tos field in this command is *always* set to zero.

 

**EIGRP Timers**

EIGRP uses two separate timers to ensure neighbor relationships remain established. These timers are called the "Hello timer" and the "Hold Down Timer".

 

The default hello timer for a high-speed broadcast network link is 5 seconds and the hold-down timer is 15 seconds whereas the default timers for slow-speed NBMA link are 60 seconds hello and 180 seconds dead. A slow-speed NBMA link is classified as any NBMA link with speeds equal to or less than 1544Kbps (A single T1)

 

**Configure hello and hold timers\
Hello Interval**

Router(config-if)# ip hello-interval eigrp *\<AS\> \<seconds\>*

 

**Hold Time**

Router(config-if)# ip hold-time eigrp *\<AS\> \<seconds\>*

 

**Troubleshooting/Verification**

Router# show ip protocols

-   Verify configured K values

 

Variance & Load-Balancing

Saturday, March 05, 2011

6:14 PM

 

**Overview**

EIGRP automatically load balances across equal-cost links. EIGRP allows for up to six equal-metric routes to be installed into the routing table at the same time. However, because of the complex EIGRP metric calculation, metrics may often be close to each other, but not exactly equal for load balancing purposes. TO allow for metrics that are somewhat close in value to be considered equal the *variance* command can be used. Feasible successors with a feasible distance less than (best path FD \* variance) will be proportionally utilized.

 

**Configure Variance for Unequal Cost Load Balancing**

Router(config-router)# variance *\<multiplier\>*

-   Variance can be between 1 and 128; default is 1 (equal-cost only). Instructs the router to include routes with a metric less than or equal to *n* times the minimum metric route for that destination.

>  

Router(config-router)# maximum-paths *\[1-6\]*

-   Maximum number of routes to the same destination allowed in the routing table; default 4.

>  

Router(config-router)# traffic-share balanced

-   The router balances across the routes, giving more packets to lower-metric routes.

>  

Router(config-router)# traffic-share min

-   Although multiple routes are installed, send traffic using only the lowest metric route.

>  

Router(config-router)# traffic-share min across-interfaces

-   If more than 1 route has the same metric, the router chooses routes with different outgoing interfaces, for better balancing.

>  

Router(config-router)# no traffic-share

-   Balances evenly across routes, ignoring EIGRP metrics.

>  

**Troubleshooting/Verification**

Router# show ip route

-   View routes used in load-balancing in the routing table

>  

Router# show ip eigrp topology

-   Displays EIGRP topology table used to verify routes learned

 

Passive Interface

Saturday, March 05, 2011

6:14 PM

 

**Overview**

When you configure EIGRP using a broad network statement such as network 10.80.0.0 0.0.255.255; any interface you bring online with an ip address that falls in that range will start advertising and processing received hello's on that interface. In some scenarios you may want to disable EIGRP from sending and receiving Hello's on a particular interface however you may still need that network which the interface is connected to be advertised throughout the routed domain.

**Configure Passive Interface**

Router(config-router)# passive-interface *\<interface slot/port\>*

-   Defines interface as passive and will not send or receive EIGRP updates

 

**Troubleshooting/Verification**

Router# show ip protocols

-   Verify interfaces configured as passive

 

Default Routing

Saturday, March 05, 2011

6:15 PM

 

**Overview**

Similar to RIP several methods exist in order to inject and propagate a default route through a routing domain. Default routes are routes taken if the router does not have a more precise path.

 

**Injecting a Default Route; Redistribution of Static Routes**

Router(config)# ip route 0.0.0.0 0.0.0.0 *\[destination/outgoing interface\]*

Router(config-router)# redistribute static

-   Creates a static route and redistributes static routes into EIGRP for propagation .\
     

**NOTE:** If an interface is used it must be online or it will not be injected into EIGRP.

 

**Injecting a Default Route; IP Default Network**

In a complex topology, many networks can be identified as candidate defaults. Without any dynamic protocols running, you can configured your router to choose from a number of candidate default routes based on whether the routing table has routes to networks other than 0.0.0.0/0. The *ip default-network* command enables you to configure robustness into the selection of a gateway of last resort. Rather than configuring static routes to specific next hops, you can have the router choose a default route to a particular network by checking the routing table.

 

Router(config)# router eigrp 100

Router(config-router)# network 192.168.100.0

-   Define EIGRP AS and network

!

Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.100.5

-   Define candidate default route

Router(config)# ip default-network 192.168.100.0

-   Define network to be checked for a default route

 

Essentially, multiple static routes can be added into the routing table and using the default-network command the best default route can be chosen within that network. In this case 192.168.100.5 would be chosen as the best default route out of all candidates.

 

**Injecting a Default Route; Summarize to 0.0.0.0/0**

Summarizing to a default route is effective only when you want to provide remote sites with a default route, and not propagate the default route toward the core of your network. Essentially, what ever router is neighbors with the interface define below will receive the summary address and the remote router will use it as its default route.

 

Router(config)# router eigrp 100

Router(config-router)# network 192.168.100.0

-   Define EIGRP AS and network

!

Router(config)# interface serial0/0

Router(config-if)# ip address 192.168.100.1 255.255.255.0

Router(config-if)# ip summary-address eigrp 100 0.0.0.0.0 0.0.0.0 75

-   Enables manual summarization for EIGRP autonomous system 100 on this specific interface for the given address and mask. An optional administrative distance of 75 is assigned to this summary route.

>  

**Troubleshooting/Verification**

Router# show ip route

-   Displays routing table; used to confirm gateway of last resort

 

Stub Routing

Saturday, March 05, 2011

6:16 PM

 

**Overview**

Stub routers participate minimally in EIGRP, which reduces utilization of memory and CPU on the router. Often used in lieu of static routes in a hub-spoke topology to provide a standard configuration for all spoke routers.

 

When a router has formed a stub neighbor adjacency with another router, the stub eigrp neighbor will not be sent any queries so this effectively speeds up network convergence as now there's one less router to query in case of a route failure.

 

**Stub Router Configuration**

Router(config-router)# eigrp stub *\[options\]*

 

**receive-only -** Prevents the router from advertising any routes

**connected** - Permits advertisement of connected networks (default)

**static** - Permits redistribution of static routes

**summary** - Advertises summary routes (default)

 

**Troubleshooting/Verification**

Router# show ip eigrp neighbors detail

-   Verify if a neighbor is an EIGRP stub router

 

 

Configuration

Saturday, March 05, 2011

6:55 PM

 

**Overview**

This page provides a quick overview on the basic parameters behind EIGRP. These notes only provide the absolute necessary configuration for EIGRP, more advanced options including summarization, authentication and adjusting parameters are dispersed throughout this section.

 

**EIGRP Configuration**

**Enable EIGRP Process**

Router(config)# router eigrp *\<ASN\>*

-   View overview to understand what an autonomous system is.

>  

**Add networks to advertise**

Router(config-router)# network *\<IP Address\> \<Wildcard Mask\>*

-   Wildcard masks ease specifying entire or part of network/subnets compared to traditional subnet masks.

>  

**Static Neighbor**

-   Similar to RIP at broadcast messages may not be ideal in NBMA networks and other situations. Because of this you will have to define neighbors in order for them to communicate using Unicast messages.

Router(config-router)# neighbor x.x.x.x interface *\[slot/port\]*

-   x.x.x.x designates the IP address of the interface and the interface for which the neighbor relationship will peer over.

>  

**Specify interface bandwidth**

-   EIGRP by default calculates its cost based on bandwidth and delay. For EIGRP make more informed decisions manually define interface bandwidth.

Router(config-if)# bandwidth *\<Kbps\>*

Router(config-if)# ip bandwidth percent eigrp *\<asn\> \<percentage\>*

-   ASN refers to the autonomous system number. Percentage as it sounds takes a percentage of the defined bandwidth of the interface. For example if the interface is defined as 256kbps and bandwidth is set to 75%; only 75% of 256kbps will be used.

 

**NOTE:** By default; EIGRP is set to use only up to 50% of the bandwidth of an interface to exchange routing information. Values greater than 100% can be configured. This configuration option may prove useful if the bandwidth is set artificially low for other reasons, such as manipulation of the routing metric or to accommodate an oversubscribed multipoint Frame Relay configuration.

 

**Troubleshooting/Verification**

Router# show ip eigrp neighbors *\[detail\]*

-   Displays the neighbor table; optional detail command for more information including whether a neighbor is a stub router.

>  

Router# show ip eigrp interfaces

-   Show information for each EIGRP interface

>  

Router# show ip eigrp interface *\<asn\>*

-   Displays interfaces who run within a specific autonomous system

 

Router# show ip eigrp topology

-   Displays topology table; used to determine who your feasible successors are

>  

Router# show ip eigrp traffic

-   Shows the number and type of packets sent and received

>  

Router# show ip route eigrp

-   Shows a routing table with only EIGRP entries.

>  

Router# clear ip eigrp neighbors

-   Resets EIGRP neighbor relationships and forces convergence

>  

Router# deug ip eigrp *\[packets \| neighbors\]*

-   Used to debug EIGRP operations in real time

>  

Router(config)# debug ip routing

-   Displays routing changes and notifications

 

 

Authentication

Saturday, March 05, 2011

6:15 PM

 

**Overview**

EIGRP supports Message Digest 5 (MD5) authentication to prevent malicious and incorrect routing information from being introduced into the routing table of a Cisco router. EIGRP requires the creation of keys and requires authentication to be enabled on a per-interface basis.

 

**Authentication Configuration**

**Configure Key Chain**

-   Key chains identify a group of authentication keys

Router(config)# key chain *\<name\>*

 

**Identify key-id**

-   Key-IDs identify an authentication key on a key chain

Router(config-keychain)# key *\<number\>*

 

**Specify key-string**

-   Specify key-string (password) that matches a key-ID

Router(config-keychain-key)# key-string *\<password\>*

 

**Define when keys can be sent/received**

Router(config-keychain-key)# accept-lifetime *\<start-time\>* {infinite \| end-time \| duration *seconds}*

-   Specifies the period during which they key can be received.

Router(config-keychain-key)# send-lifetime \<*start-time\>* {infinite \| end-time \| duration *seconds}*

-   Specifies the period during which the key can be sent.

 

**Specify Authentication Type / Enable Authentication**

Router(config)# interface *\[interface\]*

Router(config-if)# ip authentication mode eigrp *\<asn\>* md5

-   Specify Authentication Type

Router(config-if)# ip authentication key-chain eigrp *\<asn\> \<key-chain\>*

-   Enable authentication and specify key chain on an interface

 

**Troubleshooting/Verification**

Router# debug eigrp packets

-   Used to determine if packets are being authenticated; can be used to confirm configuration.

 

 

Summarization

Saturday, March 05, 2011

6:15 PM

 

**Overview**

Summarization is when multiple subnets are put into a single larger subnet which gets advertised to neighboring routers to conserve router resources.

 

**Configuring manual summarization of outbound routes**

Router(config-if)# ip summary-address eigrp *\<as\> \<network\> \<subnet\> \<administrative-distance\>*

 

**Example:**\
Router(config-if)# ip summary-address eigrp 100 10.10.0.0 255.255.0.0 75

-   Summarizes all addresses to the network defined above under autonomous system 100 with an optionally defined administrative distance of 75; default of 5.

>  

**NOTE:** Administrative distances can be adjusted to a higher level and is often referred to as a \"Floating Summary Route\".

 

**Auto Summarization**

EIGRP performs automatic summarization and should be disabled for increased control over summarization. Otherwise EIGRP will summarize to the network (a, b, c) class boundaries. After IOS 12.2(8)T EIGRP auto summarization is disabled by default.

 

**Disable Auto Summary**

Router(config-router)# no auto-summary

 

 

Filtering

Saturday, March 05, 2011

6:15 PM

 

**Overview**

Filtering can be done in a variety of ways using several different methods and approaches. This section covers a few methods for filtering, manipulating and changing routes.

 

**Distribution List**

-   A distribute-list is used to control routing updates either coming *TO* your router or leaving *FROM* your router.

**Offset List**

-   A offset list can be used to manipulate incoming and/or outgoing routes metric to administratively force dynamic route selection.

 

Distribute Lists

Saturday, March 05, 2011

6:15 PM

 

**Overview**

Outbound and inbound EIGRP updates can be filtered at any interface or for the entire EIGRP process.

 

**Distribute List Configuration**

**Define routes you want to filter**

-   Assume we have viewed the routing table and have selected routes for filtering purposes. We want to deny the following networks 192.168.4.0/25, 192.168.4.128/25, 192.168.4.128/26, 192.168.4.192/26 from being advertised OUT from our router to other routers within the RIP domain.

 

**Create Prefix List to filter out the specified traffic**

-   We need to define an ACL that identifies that route and control it with either permit/deny statements. Below shows the granularity allowed with prefix lists in which only a single statement is required to block several different masks. A standard ACL would require an individual deny statement for each network.

 

Router(config)# ip prefix-list LIST-EXAMPLE seq 5 deny 192.168.4.0/24 ge 25 le 26

Router(config)# ip prefix-list LIST-EXAMPLE seq 10 permit 0.0.0.0/0 le 32

 

**Create distribution-list that references the ACL and defines the direction.**

-   The distribute-list is defined underneath the routing process for the protocol that it is being used on.

Router(config-router)# distribute-list prefix LIST-EXAMPLE out interface serial0/0

 

**Troubleshooting/Verification**

Router# show ip route

-   Verify distribution lists are performing correctly.

 

Offset Lists

Saturday, March 05, 2011

6:16 PM

 

**Overview**

EIGRP offset lists allow for EIGRP to add to a route\'s metric, either before sending an update, or for routes received in an update. The offset list refers to an ACL or Prefix List to match the routes; and matched routes have a specified offset or extra metric added to their metrics. Any routes not matched by the offset list are unchanged. The offset list also specifies which routing updates to examine by specifying a direction (in or out) and, optionally, an interface. If the interface is omitted from the command, all updates for the defined direction will be examined.

 

**Offset List Configuration**\
**Define networks to be manipulated by an ACL**

Router(config)# ip access-list 100 permit 10.10.5.0 0.0.0.255

 

**Apply offset-list**

Router(config-router)# offset-list 100 out 999 serial0/0

-   All networks that match ACL 100 will have their metric increased by 999 going out of serial0/0.

 

**Troubleshooting/Verification**

Router# show ip route

-   Displays routes and metrics; can be used to confirm that the offset-list is manipulating EIGRP\'s metric.
