## BGP Index

- [BGP Operation](#bgp-operation)
    - [Routing Updates](#routing-updates)
- [Neighbor Relationships](#neighbor-relationships)
    - [Peer Groups](#peer-groups)
    - [Peer Templates](#peer-templates)
- [Authentication](#authentication)
- [External Border Gateway Protocol eBGP](#external-border-gateway-protocol-ebgp)
    - [Multihop](#multihop)
    - [Backdoor](#backdoor)
    - [Distance](#distance)
    - [Maximum Paths](#maximum-paths)
    - [DMZ Link Bandwidth](#dmz-link-bandwidth)
- [Next Hop Processing](#next-hop-processing)
- [Internal Border Gateway Protocol iBGP](#internal-border-gateway-protocol-ibgp)
    - [Update Source](#update-source)
    - [Route Reflection](#route-reflection)
    - [iBGP Synchronization](#ibgp-synchronization)
- [Best Path Selection Process](#best-path-selection-process)
    - [Weight](#weight)
    - [Local Preference](#local-preference)
    - [AS Path Prepending](#as-path-prepending)
    - [Multi Exit Discriminator MED](#multi-exit-discriminator-med)
- [Communities](#communities)
- [Default Originate](#default-originate)
- [Originating Prefixes](#originating-prefixes)
    - [Auto Summary](#auto-summary)
    - [Network Statement](#network-statement)
    - [Redistribution](#redistribution)
    - [Aggregation](#aggregation)
    - [AS Set](#as-set)
- [Filtering](#filtering)
    - [AS Path Filter List](#as-path-filter-list)
    - [Distribute Lists](#distribute-lists)
    - [Prefix Lists](#prefix-lists)
    - [Route Maps](#route-maps)
- [Conditional Advertisement](#conditional-advertisement)
- [Conditional Route Injection](#conditional-route-injection)
- [Clearing BGP Sessions](#clearing-bgp-sessions)
- [Outbound Route Filtering ORF](#outbound-route-filtering-orf)
- [Local AS](#local-as)
- [Route Dampening](#route-dampening)
- [Regular Expressions](#regular-expressions)
- [Fast External Fallover FEA](#fast-external-fallover-fea)
- [Fast Peering Session Deactivation FPSD](#fast-peering-session-deactivation-fpsd)
- [Next Hop Address Tracking](#next-hop-address-tracking)
- [Maximum Prefix](#maximum-prefix)
- [BGP Policy Accounting](#bgp-policy-accounting)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Border Gateway Protocol (BGP)** is the protocol backing the core routing decisions on the Internet. It maintains a table of IP networks or 'prefixes' which designate network reachability among autonomous systems (AS). It is described as a path vector protocol. BGP does not use traditional Interior Gateway Protocol (IGP) metrics, but makes routing decisions based on path, network policies and/or rulesets. For this reason, it is more appropriately termed a reachability protocol rather than routing protocol.

BGP is an advanced path vector protocol and includes the following:

-   BGP is the only routing protocol in widespread use which facilitates interdomain routing (between autonomous systems).

-   BGP is path-vector; routes are tracked in terms of which autonomous systems they pass through.

-   BGP attributes allow granularity in path selection.

-   Reliable updates.

-   Triggered updates only.

-   Rich metric (Path attributes).

-   Scalable to massive networks.

**BGP Tables**

**Neighbor Table:** Connected BGP routers (statically defined).

**BGP Table:** A list of *all* BGP routes

**Routing Table:** A list of the *best* routes

**Terminology**

**Autonomous System (AS)** - A logical domain under the control of a single entity

**External BGP (eBGP)** - BGP adjacencies which span autonomous system boundaries

**Internal BGP (iBGP)** - BGP adjacencies formed within a single AS

**Synchronization Requirement -** A route must be known by an IGP before it may be advertised to BGP peers

**Protocol Specifications**

**Protocol Type -** Path vector

**Peering mechanism -** Manual peering between neighbors

**eBGP AD -** 20

**iBGP AD -** 200

**Rights -** Open standard

**Supported protocols -** IPv4, IPv6

**Transport -** TCP/179

**Update mode -** Only triggered

**Timers -** Hello (60 sec)

**Authentication -** None, MD5

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## BGP Operation

BGP selects a **single** best path to a destination, and inserts it in the IP routing table. IP datagrams are only forwarded based on routes in the IP table, NOT by the routes in the BGP table.

Since each distinct prefix is a unique destination, BGP will select and advertise only a single best path. To decide which is the 'best path', BGP uses an extensive tie-breaking algorithm. At each step, BGP seeks to break a tie between metrics. The selection process ends at the point where a tie between routes is broken.

Attributes are assigned to routes and then the path vector algorithm is performed to insert the appropriate route into the IP routing table.

**Path Attributes**

-   **Well-known mandatory -** *must be supported and included*

    -   Origin - The source of the route (IGP EGP Unknown - route was redistributed)

    -   AS Path - An ordered list of the ASs the route has traversed, two types exist.

        -   AS sequence (an ordered list)

        -   AS set (an unordered list)

    -   Next Hop - Specifies the next-hop address for the route

-   **Well-known discretionary -** *must be supported but may not be included*

    -   Local Preference - Used for consistent routing policy with an autonomous system

    -   Atomic Aggregate - Informs the neighbor autonomous system that the originating router aggregated (summarized) routes

-   **(Optional) Transitive -** *don't have to be supported, but must be passed onto peer*

    -   Aggregator - Identifies the router and AS where summarization was performed

    -   Community - Provides route tagging capability

-   **(Optional) Nontransitive -** *don't have to be supported, and can be ignored*

    -   Multi Exit Discriminator - Used to discriminate between multiple entry point into an autonomous system

    -   Originator ID - Identifies a route reflector

    -   Cluster List - Records the route reflector clusters the route has traversed

**NOTE:** Administrative weight is a Cisco proprietary attribute, a 16-bit value referenced only by the local router.

**Path Vector Algorithm**

-   The first path received is automatically the 'best path'. Any further paths received are compared to this path to determine if the new path is 'best'.

-   Is the route **VALID**? To be valid:

    -   **The route must be synchronized with the Interior Gateway Protocol (*unless synchronization is turned off*).**

    -   **The route must appear in the IP routing table (see previous bullet point).**

    -   **The NEXT_HOP must be *reachable*.**

    -   **The AS_PATHs received from an external AS *must not contain the local AS,* or they will be discarded.**

    -   **The local routing policy must permit the route. If the neighbor is filtering the route, they won't use it.**

-   Highest **WEIGHT**

    -   **Weight is a Cisco-proprietary setting and only exists on the router on which it is configured. It is otherwise useless throughout an AS.**

-   Highest **LOCAL_PREF** (used within an AS)

-   Prefer **LOCAL**LY **ORIGIN**ATED route (originated from this router)

-   **Shortest AS-PATH**

-   Lowest ORIGIN type:
    **IGP**  **EGP**  **INCOMPLETE**

-   Lowest **MULTI-EXIT-DISCRIMINATOR (MED)**

-   **Prefer eBGP** route **over iBGP** route

-   Lowest IGP metric to **BGP NEXT_HOP**

-   Prefer **FIRST RECEIVED EXTERNAL ROUTE** (prefer the OLDEST external PATH)

-   Prefer **lowest router ID**

    -   **The Cisco router ID is *IP address of the router*, which in turn is the *highest IP address* on the router, or its *Loopback* Interface if it has one.**

-   Minimum **CLUSTER_ID** length

-   **Lowest neighbor address**

From this list, you can see it is *impossible* for two BGP routes to TIE each other and become equally preferred. As has been stated elsewhere in this site, BGP contains only a *single best path* to any given destination. BGP runs across the entire Internet, therefore it must manage to reduce the number of advertised routes in order to prevent the Internet from becoming flooded with route advertisement traffic. Thus, this algorithm is designed to eliminate all but 1 route to a destination.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Routing Updates

Maximizing uptime requires multiple Internet carriers. The Border Gateway Protocol (BGP) allows companies to use two or more Internet connections at the same time and maintain connectivity during an outage without having to change IP addresses.

**Three Types of Routes**

BGP tells your router where to send outbound traffic. When you connect your network to an Internet provider's network using BGP, your router can receive three different routes: Default Route, Full Routes, or Partial Routes.

**1. Default Route: failover but no optimization**

Configuring BGP for each Internet connection using a default route allows a company to have automatic failover between two providers should one provider go down. However, BGP will not load-balance and will send all outgoing traffic to only one ISP. Default routes are often used on routers with less than 512 MB RAM.

```
Router(config)# router bgp *asn*
Router(config-router)# neighbor x.x.x.x default-originate
```

-   Advertises a default-route to an established neighbor.

**2. Full Routes: failover plus optimization**

Using full BGP routes with multiple Internet providers allows a company to:

**Optimize:** Automatically optimize outbound traffic to choose the provider with the shortest path (shortest AS number, by default) to each destination.

**Failover:** Have auto-failover should one of the ISPs go down. BGP tries to find the shortest path from your gateway to a destination IP address. If one of the carriers is down, the router simply chooses a path from the other carrier.

However, Full Routes typically require a router with minimum 1 GB RAM. Advertising full routes requires keeping all the Internet's available paths in your router's memory. As of 2010, there are over 280,000 routes on the Internet, so having two ISPs means you'll need to store over 560,000 paths.

```
Router(config)# router bgp *asn*
Router(config-router)# neighbor x.x.x.x remote-as *asn*
```

-   By default the full routing table will be sent to an established neighbor.

**3. Partial Routes: optimizing a subset of designated paths**

With Partial Routes, you configure the router to receive only certain routes. You could ask the ISP to only send some routes, but filtering routes yourself allows for more control and the ability to receive full routes if you upgrade your router.

Typically, you would designate a default route for each provider, plus some specific routes through each provider to frequently-accessed IP addresses (web apps, frequently-used web sites and services, off-site servers, etc.).

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Neighbor Relationships

BGP neighbors are not discovered; rather they must be configured manually on both sides of the connection.

BGP neighbors form a TCP connection with each neighbor, sending BGP messages over the connections- culminating in BGP ***Update messages*** that contain the routing information. Each router explicitly configures its neighbors' IP addresses, using these definitions to tell a router with which IP addresses to attempt a TCP connection. Also, if a router receives a TCP connection request (to BGP port 179) from a source IP address that is not configured as a BGP neighbor, the router rejects the request.

After the TCP connection is establish. BGP begins with BGP ***Open messages***. Once a pair of BGP Open messages has been exchanged, the neighbors have reached the ***Established state***, which is the stable state of two working BGP peers. At this point, BGP update messages can be exchanged.

**Checks Before Becoming BGP Neighbors**

1.  The router must receive a TCP connection request with a source address that the router finds in a BGP neighbor command.

2.  A router's ASN (autonomous system number) must match the neighbor's router's reference to that ASN with its **neighbor remote** *asn*

3.  The BGP RIDs (Router ID) of the two routers must not be the same.

4.  If configured, MD5 authentication must pass.

**Packet Types**

**Open -** Used to establish a neighbor relationship and exchange basic parameters.
**Update -** Used to exchange routing information

**Keepalive -** Used to maintain the neighbor relationship

**Notification -** Used when BGP errors occur, causes a reset to the neighbor relationship when set.

**Neighbor States**

**Idle** - Neighbor is not responding

**Connect** - A TCP connection is being attempted

**Active** - A TCP connection has failed; the router is waiting to be contacted by its peer

**Open Sent** - TCP session established, open message sent

**Open Confirm** - Waiting for a keepalive from the peer

**Established** - Adjacency established

**NOTE:** Keepalive are sent every 60 seconds, hold-down of 180 seconds. Peers can use MD5 shared secret.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Peer Groups

To configure one router with multiple BGP peer relationships, configurations can be quite complex. Peer groups simplify the configuration process. You make peer groups and assign neighbors with the same policies to the group. Peer group members inherit the policies assigned to the group.

**Peer Group Configuration**

**Create a BGP peer group**

```
Router(config-router)# neighbor *group-name* peer-group
```

**Specify parameters for the BGP peer group (AS number, Update-Source, Filtering etc.)**

```
Router(config-router)# neighbor *group-name* remote-as 65200
Router(config-router)# neighbor *group-name* update-source loopback 0
```

**Assign a neighbor to the peer group**

```
Router(config-router)# neighbor 1.1.1.1 peer-group *<group-name>*
```

Troubleshooting/Verification

```
Router# show ip bgp peer-group [*peer-group-name] [summary]*
```

-   Displays peer group configuration

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Peer Templates

Cisco introduced dynamic update peer groups, a feature which essentially enables the software to automatically group BGP neighbors into peer groups and generate update messages accordingly. Which essentially eliminated the primary goal of peer groups - to reduce CPU utilization. Although this new feature effectively obsoleted manually defined peer groups, they have stuck around as many administrators appreciate the convenience they offer in configuration.

With the burden of CPU optimization delegated to dynamic update peer groups, peer templates introduced a much more flexible, hierarchical peer configuration scheme. There are two types of peer templates: *session templates* and *policy templates*.

Session templates affect the actual BGP session with a neighboring router, whereas policy templates affect protocol-specific (NLRI) policy (e.g. IPv4, IPv6, VPNv4). The following table lists which parameters are defined in each type of template (as of IOS 12.4T):

**Peer Template Configuration Example**

To illustrate how peer templates work, consider the following peer group configuration:

```
Router(config)# router bgp 65200
Router(config-router)# no synchronization
Router(config-router)# bgp router-id 10.0.0.3
Router(config-router)# bgp log-neighbor-changes
Router(config-router)# neighbor IBGP-Peers peer-group
Router(config-router)# neighbor IBGP-Peers remote-as 65200
Router(config-router)# *neighbor IBGP-Peers timers 30 300*
Router(config-router)# neighbor EBGP-Peers peer-group
Router(config-router)# neighbor EBGP-Peers ttl-security hops 1
Router(config-router)# *neighbor EBGP-Peers timers 30 300*
Router(config-router)# neighbor 10.0.1.1 remote-as 65101
Router(config-router)# neighbor 10.0.1.1 peer-group EBGP-Peers
Router(config-router)# neighbor 10.0.1.2 remote-as 65102
Router(config-router)# neighbor 10.0.1.2 peer-group EBGP-Peers
Router(config-router)# neighbor 10.0.2.4 peer-group IBGP-Peers
Router(config-router)# neighbor 10.0.2.5 peer-group IBGP-Peers
Router(config-router)# no auto-summary
```

Two peer groups have been created: one for IBGP neighbors, and one for EBGP neighbors. Each group defines some unique session characteristics, but they both also have one in common (the BGP timers configuration). Because of the way peer groups work (again, they were originally intended only for CPU optimization), common attributes must be replicated across peer groups, introducing redundancy in the configuration.

![peertemplates1.png](/Images/BGP/peertemplates1.png)

Peer templates embrace a hierarchical approach to combat such redundancy. We can create one template for session parameters common to all BGP neighbors, and then create more-specific templates which inherit those parameters. Templates are created under the BGP routing process:

Router(config)# **router bgp 65200**
Router(config-router)# **template ?**
peer-policy Template configuration for policy parameters
peer-session Template configuration for session parameters

Router(config-router)# **template peer-session BGP-All**
Router(config-router-stmp)# **timers 30 300**
Router(config-router-stmp)# **exit-peer-session**

The **BGP-All** template holds the timers configuration line we want to apply to all peers. Next we'll create separate IBGP and EBGP templates, both of which will inherit parameters from the **BGP-All** template using the inherit peer-session command.

![peertemplates2.png](/Images/BGP/peertemplates2.png)

The completed configuration looks like this:
Now, both IGBP and EBGP will inherit peer-session BGP-All and have the same timers. But within their own individual peer-session template they can have unique attributes. Configuration for applying the final peer template to a neighbor is highlighted near the end of the configuration.

```
Router(config)#router bgp 65200
Router(config-router)# template peer-session BGP-All
Router(config-router-stmp)# timers 30 300
Router(config-router-stmp)# exit-peer-session
!
Router(config-router)# template peer-session IBGP
Router(config-router-stmp)# remote-as 65200
Router(config-router-stmp)# inherit peer-session BGP-All
Router(config-router-stmp)# exit-peer-session
!
Router(config-router)# template peer-session EBGP
Router(config-router-stmp)# ttl-security hops 1
Router(config-router-stmp)# inherit peer-session BGP-All
Router(config-router-stmp)# exit-peer-session
!
Router(config-router)# no synchronization
Router(config-router)# bgp router-id 10.0.0.3
Router(config-router)# bgp log-neighbor-changes
Router(config-router)# neighbor 10.0.1.1 remote-as 65101
Router(config-router)# neighbor 10.0.1.1 inherit peer-session EBGP
Router(config-router)# neighbor 10.0.1.2 remote-as 65102
Router(config-router)# neighbor 10.0.1.2 inherit peer-session EBGP
Router(config-router)# neighbor 10.0.2.4 inherit peer-session IBGP
Router(config-router)# neighbor 10.0.2.5 inherit peer-session IBGP
Router(config-router)# no auto-summary
```

**Peer Policy Configuration**

The above example demonstrates peer session templates at work. As you might have guessed, peer *policy* templates work very similarly but are defined under an address family.

**Define Peer Policy**

```
Router(config)# router bgp 65000
Router(config-router)# template peer-policy Internal-Policy
Router(config-router-ptmp)# prefix-list DENY-SOMETHING in
Router(config-router-ptmp)# exit-peer-policy
!
Router(config-router)# template peer-session Internal-Session
Router(config-router-stmp)# remote-as 65000
Router(config-router-stmp)# update-source Loopback0
Router(config-router)#exit-peer-session
!
Router(config-router)# bgp log-neighbor-changes
Router(config-router)#neighbor 10.0.1.1 inherit peer-session Internal-Session
```

**Apply Peer Policy**

```
Router(config-router)# address-family ipv4
Router(config-router-af)# no synchronization
Router(config-router-af)# neighbor 10.0.1.1 activate
Router(config-router-af)# neighbor 10.0.1.1 inherit peer-policy Internal-Policy
Router(config-router-af)# no auto-summary
```

**Troubleshooting/Verification**

```
Router# show ip bgp template peer-session
```

-   Displays peer-session configuration

```
Router# show ip bgp template peer-policy
```

-   Displays peer-policy configuration

**[Session Templates]**

```
allowas-in
description
disable-connected-check
ebgp-multihop
fall-over
local-as
password
remote-as
shutdown
timers
translate-update
transport
ttl-security
update-source
version
```

**[Policy Templates]**

```
advertisement-interval
allowas-in
as-override
capability
default-originate
distribute-list
dmzlink-bw
filter-list
maximum-prefix
next-hop-self
next-hop-unchanged
prefix-list
remove-private-as
route-map
route-reflector-client
send-community
send-label
soft-reconfiguration
soo
unsuppress-map
weight
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Authentication

You can configure MD5 authentication between two BGP peers, meaning that each segment sent on the TCP connection between the peers is verified. MD5 authentication must be configured with the same password on both BGP peers; otherwise, the connection between them will not be made. Configuring MD5 authentication causes the Cisco IOS software to generate and check the MD5 digest of every segment sent on the TCP connection.

![authentication.png](/Images/BGP/authentication.png)

**MD5 Authentication Configuration**

```
Router(config-router)# neighbor *[ip-address | peer-group-name]* password *string*
```

-   Enables MD5 authentication, must be configured on both neighbors with the same password string.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## External Border Gateway Protocol eBGP

External Border Gateway Protocol, a routing protocol that allows multiple antonymous systems to communicate and broadcast routes over a network.

**EBGP Neighbor Relationship Configuration**

```
Router(config)# router bgp 6500**1**
Router(config-router)# neighbor 10.0.0.2 remote-as 6500**2**
```

-   Defines 10.0.0.2 as peer IP address within the autonomous system 6500**2**. Making this an **E**BGP neighbor relationship, other side must have a mirrored configuration.

**Adjusting Timers**

```
Router(config-router)# bgp timers *keepaliveholdtime*
```

-   Used to adjust keepalive and holdtime values globally, 60 and 180 seconds respectively by default.

```
Router(config-router)# neighbor *[peer address | peer-group-name]* timers *keepaliveholdtime*
```

-   Used to adjust keepalive and holdtime values per neighbor relationship

**Troubleshooting/Verification**

```
Router# show ip bgp summary
```

-   Displays neighbor relationship status and statistics

```
Router(config)# debug ip routing
```

-   Displays routing changes and notifications

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Multihop

EBGP exchanges routing information between adjacent autonomous systems, whether they are peers (equal standing), upstreams (providers/carriers), or downstreams (customers/subscribers). This exchange occurs via network announcements, and the corresponding routes are referred to as prefixes or aggregates.

The routing software decides based on the ASN following the remote-as statement whether it is a remote AS (EBGP) or a local (IBGP) connection. EBGP neighbors need to be adjacent (directly connected); for IBGP, this is left to the underlying IGP. If the EBGP neighbor is several hops away, the ebgp-multihop neighbor command can satisfy this requirement. This is rather common because EBGP peering sessions are often configured loopback to loopback (recommended), which often results in at least a three-hop distance and improved availability. The ebgp-multihop statement is required on both neighbors.

**EBGP Multihop Configuration**

EBGP packets by default have a TTL (time to live) of 1, requiring neighbors to be directly attached. This can be administratively overridden.

```
Router(config-router)# neighbor *IP address* ebgp-multihop *hop count*
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Backdoor

A backdoor network is assigned an administrative distance of 200. The objective is to make Interior Gateway Protocol (IGP) learned routes preferred. A backdoor network is treated as a local network, except that it is not advertised. A network that is marked as a back door is not sourced by the local router, but should be learned from external neighbors. The BGP best path selection algorithm does not change when a network is configured as a back door.

**Scenario**

Suppose the link between AS100 & AS300 is 10Gbps link and the remainder of the links to AS200 are 1Gbps. The traffic sent from AS100 and AS300 will by default go to AS200, then to AS100 or AS300. This is because eBGP has an AD of 20, so the routers will trust this route over any IGP by default.

![backdoor.png](/Images/BGP/backdoor.png)

**BGP Backdoor Configuration**

Instead of globally adjusting eBGP or IGPs administrative distance, BGP backdoor allows to change the AD for eBGP on a per route basis.

**Syntax**

```
Router(config-router)# *network ip-address* mask *subnet* backdoor
```

**R1 Configuration**

```
Router1(config)# router ospf 1
Router1(config-router)# network 1.1.1.1 0.0.0.0 area 1
Router1(config-router)# network 13.13.13.1 0.0.0.0 area 0
!
Router(config)# router bgp 100
Router(config-router)# network 1.1.1.1 mask 255.255.255.255
Router(config-router)# network 3.3.3.3 mask 255.255.255.255 backdoor
```

-   Specifies network as backdoor, setting the eBGP AD to 200 and therefore making the OSPF path more preferable to reach the 3.3.3.3 network on R3.

```
Router(config-router)# neighbor 12.12.12.2 remote-as 200
```

**R2 Configuration**

```
Router2(config)# router bgp 200
Router2(config-router)# neighbor 12.12.12.1 remote-as 100
Router2(config-router)# neighbor 23.23.23.3 remote-as 300
```

**R3 Configuration**

```
Router1(config)# router ospf 1
Router1(config-router)# network 3.3.3.3 0.0.0.0 area 3
Router1(config-router)# network 13.13.13.3 0.0.0.0 area 0
Router(config)# router bgp 300
Router(config-router)# network 1.1.1.1 mask 255.255.255.255 backdoor
```

-   Specifies network as backdoor, setting the eBGP AD to 200 and therefore making the OSPF path more preferable to reach the 1.1.1.1 network on R1.

```
Router(config-router)# network 3.3.3.3 mask 255.255.255.255
Router(config-router)# neighbor 23.23.23.2 remote-as 200
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Distance

Administrative distance is a dependability rating for the source of routing information. Higher distance values imply lower trust ratings. BGP uses three different administrative distances: internal, external, and local. Routes learned through internal BGP are given the internal distance, routes learned through external BGP are given the external distance, and routes that are part of this autonomous system are given the local distance.

**External**

-   Administrative distance for BGP external routes. External routes are the best path learned from a neighbor that is external to the autonomous system. Valid values are 1 to 255. Default is 20.

**Internal**

-   Administrative distance for BGP internal routes. Internal routes are the best path learned from another BGP speaker within the same autonomous system. Valid values are 1 to 255. Default is 200.

**Local**

-   Administrative distance for BGP local routes. Local routes are networks configured with the network command. Valid values are 1 to 255. Default is 200.

**Distance Configuration**
Router(config-router)# distance bgp *external-distance internal-distance local-distance*

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Maximum Paths

You can use the **maximum-paths** command to specify the number of equal-cost paths to the same destination that BGP can submit to the IP routing table. If you set the value to 1, the system installs the single best route in the IP routing table. If you set the value greater than 1, the system installs that number of parallel routes.

For multiple paths to the same destination to be considered as multipaths, the following criteria must be met:

-   All attributes must be the same. The attributes include weight, local preference, autonomous system path (entire attribute and not just length), origin code, Multi Exit Discriminator (MED), and Interior Gateway Protocol (IGP) distance. 

-   The next hop router for each multipath must be different. 

![maximumpaths.png](/Images/BGP/maximumpaths.png)

**Scenario**

Consider the scenario above, you wish to equal cost load balance between serial0 and serial1 between autonomous systems. By default only one link will be utilized.

**eBGP Maximum-paths Configuration**

```
Router(config)# router bgp 11
Router(config-router)# maximum-paths 2
```

-   Range between 1-6, default 1 (no load balancing)

**iBGP Maximum-paths Configuration**

The maximum number applies only to routes learned from external peers unless you use the **ibgp** keyword, in which case the maximum number applies only to routes received from internal peers.

```
Router(config-router)# maximum-paths *ibgp* [1-6]
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## DMZ Link Bandwidth

When deploying BGP multipath (using Maximum-Paths explained previously), generally the decision of using multiple paths to deliver the traffic is performed inside the autonomous system by iBGP according to multiple criteria excluding the eBGP link bandwidth. The BGP Link Bandwidth feature is used to advertise the bandwidth of an autonomous system exit link as an extended community allowing BGP to do an unequal cost path load-sharing when multiple eBGP links toward the same AS exist.

![dmzlinkbw.png](/Images/BGP/dmzlinkbw.png)

**Scenario**

The above scenario three unequal cost links are present, using BGP Link Bandwidth feature data can be sent proportional to the available bandwidth of each link.

**BGP Link Bandwidth Configuration**

**Router A Configuration**

In the following example, Router A is configured to support iBGP multipath load balancing and to exchange the BGP extended community attribute with iBGP neighbors:

```
Router A(config)# router bgp 100
Router A(config-router)# neighbor 10.10.10.2 remote-as 100
Router A(config-router)# neighbor 10.10.10.2 update-source Loopback 0
Router A(config-router)# neighbor 10.10.10.3 remote-as 100
Router A(config-router)# neighbor 10.10.10.3 update-source Loopback 0
Router A(config-router)# address-family ipv4
Router A(config-router)# bgp dmzlink-bw
```

-   To configure BGP to advertise the bandwidth of links that are used to exit an autonomous system, use the neighbor dmzlink-bw command in address family configuration mode.

```
Router A(config-router-af)# neighbor 10.10.10.2 activate
```

-   Enables the exchange of information with a neighbor.

```
Router A(config-router-af)# neighbor 10.10.10.2 send-community both
Router A(config-router-af)# neighbor 10.10.10.3 activate
Router A(config-router-af)# neighbor 10.10.10.3 send-community both
Router A(config-router-af)# maximum-paths ibgp 6
```

**Router B Configuration**

In the following example, Router B is configured to support multipath load balancing, to distribute Router D and Router E link traffic proportionally to the bandwidth of each link, and to advertise the bandwidth of these links to iBGP neighbors as an extended community:

```
Router B(config)# router bgp 100
Router B(config-router)# neighbor 10.10.10.1 remote-as 100
Router B(config-router)# neighbor 10.10.10.1 update-source Loopback 0
Router B(config-router)# neighbor 10.10.10.3 remote-as 100
Router B(config-router)# neighbor 10.10.10.3 update-source Loopback 0
Router B(config-router)# neighbor 172.16.1.1 remote-as 200
Router B(config-router)# neighbor 172.16.1.1 ebgp-multihop 1
Router B(config-router)# neighbor 172.16.2.2 remote-as 200
Router B(config-router)# neighbor 172.16.2.2 ebgp-multihop 1
Router B(config-router)# address-family ipv4
Router B(config-router-af)# bgp dmzlink-bw
Router B(config-router-af)# neighbor 10.10.10.1 activate
Router B(config-router-af)# neighbor 10.10.10.1 next-hop-self
Router B(config-router-af)# neighbor 10.10.10.1 send-community both
Router B(config-router-af)# neighbor 10.10.10.3 activate
Router B(config-router-af)# neighbor 10.10.10.3 next-hop-self
Router B(config-router-af)# neighbor 10.10.10.3 send-community both
Router B(config-router-af)# neighbor 172.16.1.1 activate
Router B(config-router-af)# neighbor 172.16.1.1 dmzlink-bw
Router B(config-router-af)# neighbor 172.16.2.2 activate
Router B(config-router-af)# neighbor 172.16.2.2 dmzlink-bw
Router B(config-router-af)# maximum-paths ibgp 6
Router B(config-router-af)# maximum-paths 6
```

**Router C Configuration**

In the following example, Router C is configured to support multipath load balancing and to advertise the bandwidth of the link with Router E to iBGP neighbors as an extended community:

```
Router C(config)# router bgp 100
Router C(config-router)# neighbor 10.10.10.1 remote-as 100
Router C(config-router)# neighbor 10.10.10.1 update-source Loopback 0
Router C(config-router)# neighbor 10.10.10.2 remote-as 100
Router C(config-router)# neighbor 10.10.10.2 update-source Loopback 0
Router C(config-router)# neighbor 172.16.3.30 remote-as 200
Router C(config-router)# neighbor 172.16.3.30 ebgp-multihop 1
Router C(config-router)# address-family ipv4
Router C(config-router-af)# bgp dmzlink-bw
Router C(config-router-af)# neighbor 10.10.10.1 activate
Router C(config-router-af)# neighbor 10.10.10.1 send-community both
Router C(config-router-af)# neighbor 10.10.10.1 next-hop-self
Router C(config-router-af)# neighbor 10.10.10.2 activate
Router C(config-router-af)# neighbor 10.10.10.2 send-community both
Router C(config-router-af)# neighbor 10.10.10.2 next-hop-self
Router C(config-router-af)# neighbor 172.16.3.3 activate
Router C(config-router-af)# neighbor 172.16.3.3 dmzlink-bw
Router C(config-router-af)# maximum-paths ibgp 6
Router C(config-router-af)# maximum-paths 6
```

**Troubleshooting/Verification**

```
Router# show ip bgp *ip-address*
```

-   Displays multipath information as well as the DMZ-Link bandwidth of each link.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Next Hop Processing

BGP next-hop processing was originally designed for "strange environments" frame relay, Ethernet etc. and because of this the next-hop-processing can sometimes cause routing issues.

**Key Points**

For eBGP peers: change next hop address on advertised routes

For iBGP peers: do not change next hop address on advertised routes

![nexthopprocessing.png](/Images/BGP.nexthopprocessing.png)

**Scenario**

If R1 is to send routes to R4 it will send them using its next-hop-address of 1.1.1.1 (loopback address) that is not shown on the diagram. This address would be propagated throughout the entire iBGP domain and everyone would be able to reach this address, so the iBGP rule works correctly.

Now lets say R4 is to pass routes from R1 to R5 and because of the eBGP rule it would change the routes next-hop-address to 10.1.45.1. Which also works correctly, because R5 would traverse R4 which then knows how to reach R1.

Now, here is where the problem comes in to play. R5 has several networks behind it and needs to advertise them to R1. R4 will receive the routes and then route them to R1, but here's the catch now that this is an iBGP relationship it will keep the same next-hop-address of R5 (10.1.45.2) which clearly can't be reached by R1, because this address is not in its routing table. Because of this the routes will not go into the IP routing table and will remain in the BGP route table.

**Next-Hop-Self Configuration**

To resolve the last scenario described a single command can be added to the neighbor relationship.

**Syntax**

```
Router(config-router)# neighbor *neighbor-ip-address* [next-hop-self]
```

**Example**

```
R4(config)# router bgp 5500
R4(config-router)# neighbor 1.1.1.1 *next-hop-self*
```

-   This changes the way R4 processes routes to its neighbor 1.1.1.1 (R1) and now will adjust the next-hop-address for routes being sent to it's address of 4.4.4.4 (loopback address) that is not shown on the diagram. Because R1 can access this iBGP neighbor and the address of 4.4.4.4. routes will be processed correctly into the routing table an thus R1 will be able to reach the networks behind R5.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Internal Border Gateway Protocol iBGP

Internal Border Gateway Protocol, iBGP is the protocol used between the routers in the same autonomous system (AS). IBGP is used to provide information to your internal routers. IBGP neighbor relationships can form over multiple hops or in other words they don't need to be directly attached neighbors.

**IBGP Neighbor Relationship Configuration**

```
Router(config)# router bgp 6500**1**
Router(config-router)# neighbor 10.0.0.**2** remote-as 6500**1**
```

-   Defines 10.0.0.2 as peer IP address within the autonomous system 6500**1**. Making this an **I**BGP neighbor relationship, other side must have a mirrored configuration.

**NOTE:** It's advised to use loopback interfaces in order to create a IGBP neighbor relationship, view update-source subpage for additional details.

**Adjusting Timers**

```
Router(config-router)# bgp timers *keepaliveholdtime*
```

-   Used to adjust keepalive and holdtime values globally, 60 and 180 seconds respectively by default.

```
Router(config-router)# neighbor *[peer address | peer-group-name]* timers *keepaliveholdtime*
```

-   Used to adjust keepalive and holdtime values per neighbor relationship

**Troubleshooting/Verification**

```
Router# show ip bgp summary
```

-   Displays neighbor relationship status and statistics

```
Router(config)# debug ip routing
```

-   Displays routing changes and notifications

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Update Source

Unlike typical routing protocols, BGP neighbors are statically defined. BGP neighbors or peers don't need to be directly connected and can use loopback interfaces to form relationships. Often times this is advantageous because if the actual physical interface shuts down, the peer relationship would not be lost (pending another path is available to the same destination).

**Update-Source Configuration**

```
Router(config)# interface loopback 0
Router(config-if)# ip address 10.0.0.2 255.255.255.255
```

-   Using a /32 mask allows is used to conserve addresses and will only allocate a single address.

```
Router(config-router)# bgp neighbor 10.0.0.2 update-source loopback 0
```

-   Defines loopback 0 as the update source and now a neighbor relationship will form because updates are coming from the proper routing source.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Route Reflection

BGP requires that all BGP peers in the same autonomous system form an iBGP session with all peers in the autonomous system. Because this is often too difficult in many networks, route reflectors can be utilized to essentially *disable* split-horizon and *reflect* routes received from one peer to another peer.

![routereflection.png](/Images/BGP/routereflection.png)

**Scenario**

In the above topology R2 will advertise a route to AS 4300 to R1. R1 will not advertise this route to R3 and R4 because of the BGP split horizon rule. BGP assumes that all iBGP peers are fully meshed. This is done to prevent routing loops. To overcome the issue route reflection can be used.

In this case we are going to designate R1 as a route reflector. To do this we configure the neighbors as route reflector clients. There is not an explicit command to designate R1 as a route reflector.

**NOTE:** If the topology above was fully meshed and a route reflector was enabled, loops would occur in the network.

**Route Reflector Configuration**

To configure route reflectors, consider these initial tasks:

-   Configure the proper cluster ID value on the route reflector

-   Configure the route reflector with information about which iBGP neighbor sessions are reaching their clients

-   In the clients, remove all iBGP sessions to neighbors that are not a route reflector in the client cluster

-   Make sure that the iBGP neighbor is removed on both ends of the iBGP session

**Define Cluster-ID**

Together, a route reflector and its clients form a *cluster*. When a single route reflector is deployed in a cluster, the cluster is identified by the router ID of the route reflector.

The **bgp cluster-id** command is used to assign a cluster ID to a route reflector when the cluster has one or more route reflectors. Multiple route reflectors are deployed in a cluster to increase redundancy and avoid a single point of failure. When multiple route reflectors are configured in a cluster, the same cluster ID is assigned to all route reflectors. This allows all route reflectors in the cluster to recognize updates from peers in the same cluster and reduces the number of updates that need to be stored in BGP routing tables.

```
R1(config-router)# bgp cluster-id *[ip-address]*
```

**Define Route Reflector Clients**

```
R1(config)# router bgp 6300
R1(config-router)# neighbor 10.12.1.2 route-reflector-client
R1(config-router)# neighbor 10.13.1.2 route-reflector-client
R1(config-router)# neighbor 10.14.1.2 route-reflector-client
```

Now that R1 is configured as a route reflector for the attached clients routes on R1 are considered "best" routes and should be installed in the routing table. Without route reflectors these would not be seen as "best" routes and therefore not propagated to other peers.

**NOTE:** Confirm that each router can reach the destinations that is now being advertised using route reflection. For example make sure that R4 has a route to reach R3 using an IGP.

**Troubleshooting/Verification**

Router# show ip bgp

-   Displays BGP route table

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Confederations

BGP confederations is a strategy to manage large BGP systems and more efficiently utilize resources on your routers. Confederations are another method of solving the iBGP full-mesh requirement. Confederations are smaller subautonomous systems created within the primary autonomous system to decrease the number of BGP peer connections.

![confederations.png](/Images/BGP/confederations.png)

**Scenario**

In the above network AS 6300 has been divided into sub-AS 65500 and sub-AS 65501. These are called confederations. The range for private AS numbers is 64,512 to 65,535.

R3 and R1 will now act like eBGP neighbors instead of iBGP neighbors. R1 will now advertise routes from R2 to R3 without the need for route reflectors. R1 will still need route reflectors configured to advertise to routes from R2 to R4.

**Confederation Configuration**

There are three major steps to configuring BGP confederations:

1.  Configure all peers in their own confederations

2.  Configure special eBGP peers between confederations

3.  Configure routers that are connected to outside systems

**Configure all peers in their own confederation**

```
R1(config)# no router bgp 6300
```

-   Removes previous BGP configuration

```
R1(config)# router bgp 65500
```

-   Defines private AS value

```
R1(config-router)# neighbor 10.12.1.2 remote-as 65500
R1(config-router)# neighbor 10.14.1.2 remote-as 65500
```

-   Defines neighbors, notice that the neighbors are within the new private AS.

```
R2(config)# no router bgp 6300
R2(config)# router bgp 65500
R2(config-router)# neighbor 10.12.1.1 remote-as 65500
R2(config-router)# neighbor 10.14.1.2 remote-as 65500
```
 
```
R4(config)# no router bgp 6300
R4(config)# router bgp 65500
R4(config-router)# neighbor 10.12.1.1 remote-as 65500
R4(config-router)# neighbor 10.12.1.2 remote-as 65500
```
 
```
R3(config)# no router bgp 6300
R3(config)# router bgp 65501
```

-   Within its own private AS/confederation

**Configure special eBGP peers between confederations**

```
R1(config)# router bgp 65500
R1(config-router)# bgp confederation identifier 6300
```

-   Identified "real" public assigned AS number.

```
R1(config-router)# bgp confederation peers 65501
```

-   Identifies any confederation AS numbers that this router peers with, in this case R3.

Repeat on other two routers in this confederation.

```
R2(config)# router bgp 65500
R2(config-router)# bgp confederation identifier 6300
```

-   No confederation peer statements necessary

```
R4(config)# router bgp 65500
R4(config-router)# bgp confederation identifier 6300
```

Recall that if a confederation is not a full mesh, a route reflector will need to be configured on R1 to allow routes to be passed between R2 and R4.

```
R1(config)# router bgp 65500
R1(config-router)# neighbor 10.12.1.2 route-reflector-client
R1(config-router)# neighbor 10.14.1.2 route-reflector-client
```

Now, since R3 is within its own confederation the relationship between R1 and R3 is no longer considered iBGP (between the same AS) but an eBGP relationship (between different AS). Because of this neighbor relationships need to be defined.

```
R1(config)# router bgp 65500
R1(config-router)# neighbor 10.13.1.2 remote-as 65501
```

```
R3(config)# router bgp 65501
R3(config-router)# neighbor 10.13.1.2 remote-as 65500
```

**Configure routers that are connected to outside systems**

At this point we have our autonomous system running successfully but we need to integrate with the outside world.

From the R5 perspective, R5 is peering with a router (R1) in AS 6300. R5 doesn't know anything about the confederations we created.

```
R1(config)# router bgp 65500
R1(config-router)# neighbor 10.17.1.2 remote-as 5300
```

-   Configures R1 to peer with R5 via eBGP

```
R5(config)# router bgp 5300
R5(config-router)# neighbor 10.17.1.1 remote-as 6300
```

-   Configures R5 to peer with R1 via eBGP.

**NOTE:** Earlier versions of IOS will not remove private AS information and may require manually removal of private AS. This can be accomplished using the command **neighbor x.x.x.x remove-private-as**

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## iBGP Synchronization

iBGP peers must be fully meshed, as iBGP-learned routes are not passed to other iBGP peers. The rule of synchronization requires that an iBGP-learned route must be known by an IGP before it enters the BGP routing table.

**NOTE:** In later IOS versions 12.2(8t) and greater synchronization is turned off.

**Disable iBGP Synchronization**

```
Router(config)# router bgp *asn*
Router(config-router)# no synchronization
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Best Path Selection Process

BGP has a very complex method of deciding the best path for destination reachability. The subsections below provide several methods to manually influence path selection and manipulate automatic path selection using self-generated BGP path attributes.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Weight

Weight, a Cisco-proprietary feature. The administrative weight can be assigned to each NLRI locally on a router, and the value cannot be communicated between another router. The higher the value, the better the route. A common use of the weight attribute is if a organization had duel ISPs (multihoming) and would like to prefer one ISP over the other.

**Weight Configuration**

You can use weight to provide local routing policy, and you can use local preference to establish autonomous system-wide routing policy. Higher weights are preferred.

```
Router(config-router)# neighbor *[ip-address | peer-group-name]* weight *weight*
```

-   This approach assigns a weight value to all route updates **from** the neighbor this command is configured on. In other word any route this router sends to the neighbor defined will receive the configured weight value.
 
```
Router(config-router)# neighbor *[ip-address | peer-group-name]* filter-list *access-list*[in | out] weight *weight*
```

-   Configures the router so that all **incoming** routes that match an autonomous system filter receive the configured weight.

**NOTE:** The default weight value is 32,768 for locally originating networks (including those via redistributing) and is 0 for all other networks.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Local Preference

Local preference can be used to influence route selection within the local autonomous system; in fact, this attribute is stripped from outgoing updates via eBGP. You should decide between the use of weight or local preference. The default local preference for iBGP and local routes is 100; all other are 0 by default.

You can apply local preference in the following ways:

-   Using a route map with the **set default local-preference** command

-   Using the **bgp default local-preference** command to change the default local preference value applied to all updates coming from external neighbors or originating locally.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## AS Path Prepending

In networks where connections to multiple providers are required, it is difficult to specify a return path to be used for traffic returning to the autonomous system. One BGP mechanism you can use is autonomous system path prepending. Autonomous system path prepending potentially allows the customer to influence the route selection of its service providers.

You manipulate autonomous system paths by prepending autonomous system numbers to existing autonomous system paths. Typically, you perform autonomous system path prepending on outgoing eBGP updates over the undesired return path. Because the autonomous system paths sent over the undesired link become longer that the path sent over the preferred path. The undesired link is now less likely to be used as a return path. To avoid conflicts number, except that of the sending autonomous system, should be prepended to the autonomous system path attribute.

You can configure manual manipulation of the autonomous system path attribute (prepending) using a route map with the **set as-path prepend** set clause.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Multi Exit Discriminator MED

You can apply the MED attribute on outgoing updates to a neighboring autonomous system to influence the route selection process in that autonomous system. The MED attribute is useful only when you have multiple entry points into an autonomous system.

The default value of the MED attribute is 0. A lower value of MED is more preferred. A router prefers a path with the smallest MED value but only if weight, local preference, autonomous system path, and origin code are equal.

MED is not a mandatory attribute; no MED attribute is attached to a route by default. The only exception is if the router is originating networks that have an exact match in the routing table (through the **network** command or through redistribution). In that case, the router uses the metric in the routing table as the MED attribute value.

Using the **default-metric** command in BGP configuration mode causes all redistributed networks to have the specific MED value.

You can use a route map to set MED on incoming or outgoing updates. Use the **set metric** command within route map configuration mode to set the MED attribute.

You must use the command **bgp bestpath med confed** when you use MED within a confederation to influence the route selection process. A router compares MED values for those routes that originate in the confederation.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Communities

BGP communities are used to group routes (also referred to as color routes) that share common properties, regardless of network, autonomous system, or any physical boundaries. In large networks applying a common routing policy through prefix-lists or access-lists requires individual peer statements on each networking device. Using the BGP community attribute BGP speakers, with common routing policies, can implement inbound or outbound route filters based on the community tag rather than consult large lists of individual permit or deny statements.

Standard community lists are used to configure well-known communities and specific community numbers. Expanded community lists are used to filter communities using a regular expression. Regular expressions are used to configure patterns to match community attributes.

The community attribute is optional, which means that it will not be passed on by networking devices that do not understand communities. Networking devices that understand communities must be configured to handle the communities or they will be discarded.

You can use the BGP community attribute to create a AS-wide routing policy or to provide services to neighboring autonomous systems.

There are four predefined communities:

-   **no-export**---Do not advertise to external BGP peers.

-   **local-as**---Do not send outside the local autonomous system.

-   **no-advertise**---Do not advertise this route to any peer.

-   **internet**---Advertise this route to the Internet community; all BGP-speaking networking devices belong to it.

-   **None** --- Apply no community attribute when you want to clear the communities associated with a route.

**Defining Communities**

-   A 32-bit community value is split into two parts.

    -   High-order 16 bits contain the AS number of the AS that defines the community meaning.

    -   Low-order 16 bits have local significance.

-   Values of all zeroes and all ones in high-order 16 bits are reserved.

-   Cisco IOS parser allows you to specify a 32-bit community value as:

    -   [AS-number]:[low-order-16-bits]

**Specify New Format for Communities**

A BGP community is displayed in a two-part format two bytes long in the **show ip bgp community** command output, as well as wherever communities are displayed in the router configuration, such as router maps and community lists. In the most recent version of the RFC for BGP, a community is of the form AA:NN, where the first part is the AS number and the second part is a 2 byte number.

The Cisco default community format is in the format NNAA.

**Display BGP communities in the new format**

```
Router(config)# ip bgp-community new-format
```

-   Displays and parses BGP BGP communities in the format AA:NN.

**BGP Community Configuration Outline**

It helps to logically break up community attribute configuration. Steps 1 matches specific routes based on various route criteria and then apply a community number attribute to those routes. Because community attributes are not propagated by default, Step 2 is used to send-community information to neighbors.

1.  Configure route tagging with BGP communities

2.  Configure BGP community propagation

Now, after the specific routes have been tagged you can identify the communities the routes belong to by using community-lists and then use a route map to match based on the community-list and set BGP attributes accordingly. Finally, apply the route-maps to incoming or outgoing updates.

3.  Define BGP community access-lists (community-lists) to match BGP communities.

4.  Configure route-maps that match on community-lists and filter routes or set other BGP attributes.

5.  Apply route-maps to incoming or outgoing updates.

**Configure route tagging with BGP communities**

**Route-map Configuration**

```
Router(config)# route-map *name*
Router(config-route-map)# match *condition*
```

-   Prefixes, metric, as-path etc.

```
Router(config-route-map)# set community *value* [additive]
```

-   Based on the match statement you can set a community to match the criteria.

-   Communities specific in the *set* keyword overwrites existing communities unless you specify the *additive* option.

**Apply route-map to neighbor**

```
Router(config-router)# neighbor *ip-address* route-map *map-name* [in | out]
```

-   This command applies a route-map to inbound or outbound BGP updates.

-   The route-map can set BGP communities or other BGP attributes.

**Apply route-map through redistribution**

```
Router(config-router)# redistribute *protocol* route-map *map-name*
```

-   This command applies a route-map to redistributed routes.

![communities1.png](/Images/BGP/communities1.png)

**Configure BGP community propagation**

**Configure send-community to allow communities to be sent to BGP neighbors.**

Router(config-router)# neighbor *ip-address* send-community

-   By default, communities are stripped in outgoing BGP updates.

-   You must manually configure community propagation to BGP neighbors.

-   BGP peer groups are ideal for configuring BGP community propagation toward a large number of neighbors (refer to BGP peer groups for additional information.)

![communities2.png](/Images/BGP/communities2.png)

**Define BGP community access-lists (community-lists) to match BGP communities.**

**Standard Community-List Configuration**

Router(config)# ip community-list [**1 - 99** | 100 - 500] permit | deny *[value]*

-   This command defines a standard community-list (1-99)

-   Community-lists are similar to access-lists- they are evaluated sequentially, line by line.

-   All values listed in one line have to match for the line to match and permit or deny a route.

-   You can use the keyword *internet* to match ANY community.

**Extended Community-List Configuration**

```
Router(config)# ip community-list [1 - 99 | **100 - 500**] permit | deny *[regexp]*
```

-   This command defines a extended community-list (100-500)

-   Extended community-lists are like simply community-lists, but they match based on regular expressions (refer to regular expressions).

-   Communities attached to a route are ordered, converted to string, and matched with regexp.

-   Use ".*" to match any community value; exactly the same as the *internet* keyword within a standard community-list.

![communities3.png](/Images/BGP/communities3.png)

**BGP Named Community-List Configuration**

-   Allows the network operator to assign meaningful names to community-lists and increases the number of community-lists that can be configured.

-   Can be configured with regular expressions and with numbered community-lists.

-   No limitation on the number of community attributes that can be configured for a named community-list.

**BGP Cost Community**

-   Allows you to customize the BGP best-path selection process for a local AS or confederation.

-   Applied to internal routes by configuring the **set extcommunity cost** command in a route-map.

-   Can be used as a "tie breaker" during the best-path selection process.

**BGP Link Bandwidth Feature**

-   Used to enable multipath load balancing for external links with unequal bandwidth capacity.

-   Enabling under an IPv4 or VPNv4 address family sessions by entering the **bgp dmzlink-bw** command. 

-   Routes learned from a directly connected external neighbor propagated through the iBGP network with the bandwidth of the source external link.

-   In other words, internal iBGP router can find a route based on unequal cost load balancing due to the bandwidth being propagated which is reliant on BGP community configuration.

**Configure route-maps that match on community-lists and filter routes or set other BGP attributes.**

**Route-map Configuration**

```
Router(config)# route-map *map-name* permit | deny
Router(config-route-map)# match community *commlist-number* [exact]
Router(config-route-map)# set *attributes*
```

-   Community-lists are used in match conditions in route-maps to match on communities attached to BGP routes.

-   A route-map with a community-list matches a route if at least some communities attached to the route match the community-list.

-   With the *exact* option, all communities attached to the route have to match the community-list.

-   You can use route-maps to filter routes or set other BGP attributes on communities attached to routes.

![communities4.png](/Images/BGP/communities4.png)

**Route Selection**

-   You can use route-maps to set weights, local preference, or metric based on BGP communities attached to the BGP route.

-   Normal route selection rules apply afterward.

-   Routes not accepted by route-maps are dropped.

**Default Filters**

-   Routes tagged with community ***no-export*** are sent to iBGP peers and intra-confederation eBGP peers.

-   Route tagged with ***local-as*** are sent to iBGP peers.

-   Routes tagged with ***no-advertise*** are not sent in any outgoing BGP updates.

**Troubleshooting/Verification**

```
Router# show ip bgp community
```

-   Displays all routes in a BGP table that have at least one community attached.

```
Router# show ip bgp community *as:nn[exact]*
```

-   Displays all routes in a BGP table that have all the specified communities attached.

-   The *exact* keyword can be used to match only exactly specified communities in the BGP table.

```
Router# show ip bgp community-list *community-list*
```

-   Displays all routes in BGP table that match a specified community-list.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Default Originate

Default Routes can be injected into BGP in one of three ways:

-   By injecting the default using the **network** command.

-   By injecting the default using the **redistribute** command.

-   By injecting a default route into BGP using the **neighbor** *neighbor-id* **default-information** [route-map *route-map-name*]

**Network**

When injecting a default route into BGP using the **network** command, a route to 0.0.0.0/0 must exist in the local routing table, and the **network 0.0.0.0** command is required. The default IP route can be learned via any means, but if it is removed from the IP routing table, BGP removes the route from the BGP table.

```
Router(config)# network 0.0.0.0
```

-   Injects default route into BGP, if a default route exists within the local routing table.

**Redistribution**

Injecting a default route through redistribution requires an additional configuration command - **default-information originate**. The default route must first exist in the IP routing table; for instance, a static default route to null0 could be created. Then, the **redistribute static** command could be used to redistribute the static default route. However, in the special case of the default route, Cisco IOS also requires the **default-information originate** BGP subcommand.

```
Router(config)# router bgp *asn*
Router(config-router)# default-information originate
```

-   Enables default route to be distributed within BGP

```
Router(config)# redistribute static
```

-   Redistributes statically defined default route

**Neighbor Default-Information**

Injecting a default route into BGP by using the **neighbor** *neighbor-ip* **default-orginate** [route-map *route-map-name*] command does not add a default route to the local BGP table; instead, it causes the advertisement of a default to the specified neighbor. In fact, this method does not even check for the existence of a default route in the IP routing table by default, but it can. With the route-map option, the reference route map examines the entries in the IP routing table (not the BGP table); if a route map permit clause is matched, then the default route is advertised to the neighbor.

**Without verifying default route exists on local router**

```
Router(config)# router bgp *asn*
Router(config-router)# neighbor neighbor-ipdefault-originate
```

**With verifying default route exists on local router**

```
Router(config)# ip route 0.0.0.0 0.0.0.0 null0
```

-   Injects default route into the local IP routing table

```
Router(config)# ip prefix-list def-route seq 5 permit 0.0.0.0/0
```

-   Works in conjunction with default-originate to verify 0.0.0.0/0 exists within the routing table.

```
Router(config)# route-map *check-default* permit 10
Router(config-route-map)# match ip address prefix-list *def-route*
```

-   Route map that confirms a route exists or not within the IP routing table

```
Router(config)# router bgp asn
Router(config-router)# neighbor neighbor-ipdefault-originate route-map *check-default*
```

-   Verifies with route-map *check-default* to see if a default route exists before advertising it to the specified neighbor.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Originating Prefixes

Unlike other routing protocols BGP has two independent configurations. First to form a neighbor relationship and second to originate prefixes into the routing process. BGP has several methods to originate prefixes into the BGP routing table.

**Network Statement:** traditional method of defining networks to advertise to other BGP peers.

**Redistribution:** redistributing networks from other routing protocols

**Aggregation:** allows the aggregation of specific routes into one route and essentially is used as a point to summarize several networks and advertise it to other BGP peers.

**Status Codes**

-   Identifies route status within the BGP routing table.

s - suppressed
d - damped
h - history
** - Valid
* - Best
i - internal

**Origin Codes**

-   Identifies the source of the advertisement

**i** - IGP (Internal Gateway Protocol)

**e** - eBGP (External Gateway Protocol)

**?** - Redistributed Route

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Auto Summary

As it does with IGPs, BGP auto-summary causes a classful summary route to be created if any component subnet of that summary exists. However, unlike IGPs, the BGP auto-summary router subcommand causes BGP to summarize only those routes *injected due to redistribution o that router*. BGP auto-summary does not look at routes already in the BGP table. It simply looks for routes injected into BGP due to the ***redistribute*** or ***network*** commands on the local router.

The logic differs slightly based on whether the route is injected with the redistribute or network command.

**Redistribute:** If any subnets of the classful network would be redistributed, do not redistribute, but instead redistribute a route for the classful network.

**Network:** If a network command lists a classful network number, with the classful default mask or no mask, and any subnets of the classful network exist, inject a route for the classful network.

**Disable Auto-Summarization**

```
Router(config)# router bgp *asn*
Router(config-router)# no auto-summary
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Network Statement

The network statement simply tells what networks to advertise, very straight forward and easy to configure.

**Network Statement Configuration**

```
Router(config)# router bgp 65001
Router(config-router)# network *ip-address* mask *subnet*
```

**Example:**

```
Router(config-router)# network 10.10.0.0 mask 255.255.0.0
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Redistribution

Redistribution can be used to redistribute routes from another routing protocols or from connected/static routes on a router into the BGP to be advertised. Refer back to redistribution section to learn about redistributing BGP routes into IGPs. This section focuses on redistribution *into* BGP.

**Redistribution Configuration**

```
Router(config)# router bgp 65001
Router(config-router)# redistribute *[eigrp | ospf | rip | ospf | isis | egp || connected | static]*
```

-   Provides redistribution with no filtering, all routes within the specified protocol will be advertised.

**NOTE:** Advanced filtering can be accomplished with access-lists and route maps after the redistribution command. As well as adjusting the metric and weight for the redistributed routes.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Aggregation

Border Gateway Protocol (BGP) allows the aggregation of specific routes into one summarized route. When you issue the aggregate-address command without any arguments, there is no inheritance of the individual route attributes (such as AS_PATH or community), which causes a loss of granularity.

**Scenario**

Routers one and two each advertise four /24 routes to their respective networks. These routes are advertised to R3, which in turn advertises them to R4. Inspecting the BGP table on R4, we can see four routes each from AS 10 and 20.

![aggregation.png](/Images/BGP/aggregation.png)

Since R4 has only a single path to all of these subnets, maintaining eight independent routes is unnecessary. We can implement aggregation at R3 to optimize the BGP table on R4.

**Creating an Aggregate Route**

The aggregate-address command can be used to generate a summary route on R3. We'll create a 172.16.0.0/21 summary to include all more-specific routes while not leaving any overlap.

```
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0
```
 
```
R4# **show ip bgp**
```

-   Verify that R4 sees the new aggregate route

```
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 10.0.0.9 0 30 10 ?
*172.16.0.0/21 10.0.0.9 0 0 30 i
*172.16.1.0/24 10.0.0.9 0 30 10 ?
*172.16.2.0/24 10.0.0.9 0 30 10 ?
*172.16.3.0/24 10.0.0.9 0 30 10 ?
*172.16.4.0/24 10.0.0.9 0 30 20 ?
*172.16.5.0/24 10.0.0.9 0 30 20 ?
*172.16.6.0/24 10.0.0.9 0 30 20 ?
*172.16.7.0/24 10.0.0.9 0 30 20 ?
```

**Creating an Aggregate Route using Summary-only**

BGP route aggregation differs from IGP summarization in that the default behavior is to continue advertising all the more-specific routes of summarized by an aggregate. As we can see here, the aggregate was injected without affecting any of the more-specific routes. We can suppress all the summarized routes by recreating the aggregate route, this time appending the summary-only keyword.

```
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0 summary-only
```

```
R4# **show ip bgp**
```

-   Verify that only a summary route is being advertised using summary-only

```
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 30 I
```

**Suppressing Routes**

As stated by default aggregating a route will advertise the summary route and each individual more specific route. If you add the *summary-only* keyword you remove all specific routes in favor of just having a single summary route.

Suppressing routes allows for advertising the summary route and selectively allowing more specific routes.

```
R4# **show ip bgp**
```

```
BGP table version is 12, local router ID is 10.0.0.10
Status codes: s suppressed, d damped, h history, * valid, best, i - internal,
r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 10.0.0.9 0 30 10 ?
*172.16.0.0/21 10.0.0.9 0 0 30 {10,20} ?
*172.16.1.0/24 10.0.0.9 0 30 10 ?
*172.16.2.0/24 10.0.0.9 0 30 10 ?
*172.16.3.0/24 10.0.0.9 0 30 10 ?
*172.16.4.0/24 10.0.0.9 0 30 20 ?
*172.16.5.0/24 10.0.0.9 0 30 20 ?
*172.16.6.0/24 10.0.0.9 0 30 20 ?
*172.16.7.0/24 10.0.0.9 0 30 20 ?
```

Since we didn't append the *summary-only* keyword, all of the aggregate's more-specific routes are still advertised independently. Let's assume we want to advertise only the 172.16.0.4.0/24 and 172.16.5.0/24 subnets from AS20 as specific-routes, and rely on the aggregate for routing to the remaining subnets. We can use a *suppress map* on R3 to suppress the six routes we don't want to advertise independently. First we create a route-map to match the routes to be suppressed, then reference it from the aggregate-address statement.

**Create ACL to specify routes**

```
R3(config)# ip access-list standard Suppressed_Routes
R3(config-std-nacl)# permit 172.16.0.0 0.0.3.255
R3(config-std-nacl)# permit 172.16.6.0 0.0.1.255
```

**Create Route Map**

```
R3(config)# route-map MySuppressMap
R3(config-route-map)# match ip address Suppressed_Routes
```

**Apply Suppress-Map to Aggregate Address**

```
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0 as-set suppress-map MySuppressMap
```

On R4, we can verify that R3 is now only advertising the aggregate route and the two /24 routes not matched by our suppress map:

```
R4# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 30 {10,20} ?
*172.16.4.0/24 10.0.0.9 0 30 20 ?
*172.16.5.0/24 10.0.0.9 0 30 20 ?
```

**Neighbor Unsuppress-Map**

We've accomplished what we wanted on R4, but R1 and R2 are now facing a serious problem: since their AS numbers are included in the aggregate route (we've appended as-set to the command), neither AS 10 or 20 will accept the aggregate route. R1 knows only of its own routes and the two AS 20 routes we didn't suppress into the aggregate:

```
R1# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 0.0.0.0 0 32768 ?
*172.16.1.0/24 0.0.0.0 0 32768 ?
*172.16.2.0/24 0.0.0.0 0 32768 ?
*172.16.3.0/24 0.0.0.0 0 32768 ?
*172.16.4.0/24 10.0.0.2 0 30 20 ?
*172.16.5.0/24 10.0.0.2 0 30 20 ?
```

R2 knows only its own routes, since all routes from AS 10 were suppressed.
 
```
R2# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.4.0/24 0.0.0.0 0 32768 ?
*172.16.5.0/24 0.0.0.0 0 32768 ?
*172.16.6.0/24 0.0.0.0 0 32768 ?
*172.16.7.0/24 0.0.0.0 0 32768 ?
```

One way to remedy this is to apply an ***unsuppress map*** to each of these neighbors on R3. As you might expect, an unsuppress map acts opposite of a suppress map, extracting and advertising more-specific routes from an aggregate. We can create route-maps to match routes from AS 10 and 20, and propagate those routes between the two autonomous systems.

**Match routes based on origin AS**

```
R3(config)# ip as-path access-list 10 permit 10
R3(config)# ip as-path access-list 20 permit 20
```

**Create Route Map**

```
R3(config)# route-map AS10_Routes
R3(config-route-map)# match as-path 10
R3(config)# route-map AS20_Routes
R3(config-route-map)# match as-path 20
```

**Apply Neighbor Unsuppress-Map**

```
R3(config)# router bgp 30
R3(config-router)# neighbor 10.0.0.1 unsuppress-map AS20_Routes
R3(config-router)# neighbor 10.0.0.5 unsuppress-map AS10_Routes
```

After completing this configuration, we reset the BGP adjacencies (clear ip bgp * on R3) and inspect the BGP tables of R1 and R2. We can see now they both know of all /24 routes.

```
R1# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 0.0.0.0 0 32768 ?
*172.16.1.0/24 0.0.0.0 0 32768 ?
*172.16.2.0/24 0.0.0.0 0 32768 ?
*172.16.3.0/24 0.0.0.0 0 32768 ?
*172.16.4.0/24 10.0.0.2 0 30 20 ?
*172.16.5.0/24 10.0.0.2 0 30 20 ?
*172.16.6.0/24 10.0.0.2 0 30 20 ?
*172.16.7.0/24 10.0.0.2 0 30 20 ?
```

```
R2# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 10.0.0.6 0 30 10 ?
*172.16.1.0/24 10.0.0.6 0 30 10 ?
*172.16.2.0/24 10.0.0.6 0 30 10 ?
*172.16.3.0/24 10.0.0.6 0 30 10 ?
*172.16.4.0/24 0.0.0.0 0 32768 ?
*172.16.5.0/24 0.0.0.0 0 32768 ?
*172.16.6.0/24 0.0.0.0 0 32768 ?
*172.16.7.0/24 0.0.0.0 0 32768 ?
```

R4, in contrast, still knows only the aggregate route and the two /24 routes not matched by our suppress map:

```
R4# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 30 {10,20} ?
*172.16.4.0/24 10.0.0.9 0 30 20 ?
*172.16.5.0/24 10.0.0.9 0 30 20 ?
```

**Modifying Attributes of the Aggregate**

For a final tweak, let's change the origin of the aggregate route from unknown ("?" in the BGP table listing) to IGP. To do, this we'll need to specify an *attribute map* and reference it from the aggregate-address command:

**Create Route Map**

```
R3(config)# route-map SetAttributes
R3(config-route-map)# set origin igp
```

**Apply Attribute-Map**

```
R3(config)# router bgp 30
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0 as-set suppress-map MySuppressMap attribute-map SetAttributes
```

After applying the attribute map, we can see that the aggregate route is now advertised with an origin of IGP ("i"):

```
R4# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 30 {10,20} ***i*
***172.16.4.0/24 10.0.0.9 0 30 20 ?
*172.16.5.0/24 10.0.0.9 0 30 20 ?
```

**Troubleshooting/Verification**

```
Router# show ip bgp
```

-   Displays the BGP routing table

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## AS Set

What's dangerous about the aggregate route advertised by R3? Notice that the route, having originated in AS 30, includes only AS 30 in it's AS path. Also remember that R4 is not the only router receiving the aggregate route; R1 and R2 receive it as well. These routers, not seeing their own AS in the route's AS path, happily install the route in their own BGP tables.

```
R1# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 0.0.0.0 0 32768 ?
**172.16.0.0/21 10.0.0.2 0 0 30 i*
*172.16.1.0/24 0.0.0.0 0 32768 ?
*172.16.2.0/24 0.0.0.0 0 32768 ?
*172.16.3.0/24 0.0.0.0 0 32768 ?
```

Consider what would happen if one of the /24 routes in AS 10 disappeared. R1, having installed the aggregate advertised from AS 30, would see AS 30 as a less-specific but valid path to the subnet, and route traffic to R3. R3, no longer having the more-specific route back to R1, drops the traffic, creating a black hole.

We can protect against this condition by including an *AS set* in the AS path of the aggregate route from R3. An AS set is an unordered list of autonomous system numbers, collected from all the routes summarized by the aggregate. By including these origin AS numbers in the aggregate's AS path, we can insure the integrity of BGP's loop prevention mechanism; by default, an AS won't accept a route with an AS path listing its own AS number.

**Creating an Aggregate Route with AS-Set**

To include an AS set in our aggregate route, append the as-set keyword to the aggregate-address command:

```
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0 summary-only as-set
```

**Verification**

This configuration generates an aggregate route with an AS path containing the AS set of 10 and 20, since the aggregate contains routes originating from those autonomous systems. On R4 we can see how the AS path appears in the BGP table:

```
R4# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 *30 {10,20}* ?
```

With the AS set included, R1 now detects its own AS in the AS path of the aggregate route, and no longer accepts the route into its BGP table.
 
```
R1# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/24 0.0.0.0 0 32768 ?
*172.16.1.0/24 0.0.0.0 0 32768 ?
*172.16.2.0/24 0.0.0.0 0 32768 ?
*172.16.3.0/24 0.0.0.0 0 32768 ?
```

**Creating an Aggregate Route with specific AS-Set**

There may be instances where you want to include only certain AS numbers in the aggregate's AS set; an *advertise map* can be used to achieve this. (This is sort of an odd name for such a tool; try to think of it as specifying the routes whose attributes should be "advertised" via the aggregate.) The advertise-map parameter is appended to the aggregate-address command to specify a route-map used to match subnets. Only attributes (like AS paths and communities) from the matched routes will be included in the aggregate.

For example, let's say we only wanted to include AS 10 in the aggregate route's AS path and omit AS 20. To do this, we create a route-map to match only AS 10 subnets, and reference it from the aggregate-address statement.

```
R3(config)# ip access-list standard AS10_subnets
R3(config-std-nacl)# permit 172.16.0.0 0.0.3.255
!
R3(config)# route-map AS10
R3(config-route-map)# match ip add AS10_subnets
!
R3(config-router)# aggregate-address 172.16.0.0 255.255.248.0 summary-only as-set advertise-map AS10
```

**Verification**

As our advertise map only matches routes from AS 10, only AS 10 is included in the aggregate's AS path, as we can see on R4:

```
R4# **show ip bgp**
Network Next Hop Metric LocPrf Weight Path
*172.16.0.0/21 10.0.0.9 0 0 *30 10* ?
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Filtering

Sending and receiving BGP updates can be controlled by using a number of different filtering methods. BGP updates can be filtered based on route information, on path information or on communities. All methods will achieve the same results, choosing one over the other depends on the specific network configuration.

Many filters exist such as route-maps, filter-lists, prefix-lists and distribute lists. The four main tools have the following features in common:

-   All can filter incoming and outgoing Updates, per neighbor or peer group.

-   Peer group configurations require Cisco IOS software to process the routing policy against the Update only once, rather than per neighbor.

-   The filters cannot be applied to a single neighbor that is configured as part of a peer group; the filter must be applied to the entire peer group, or the neighbor must be reconfigured to be outside the peer group.

-   Each tool's matching logic examines the contents of the BGP Update message, which includes BGP path attributes (PAs) and network layer reachability information (NLRI).

-   If a filter's configuration is changed, a **clear** command is required for the changed filter to take effect.

-   The **clear** command can use the soft reconfiguration option to implement changes without requiring BGP peers to be brought down and back up.

The order of preference varies based on whether the attributes are applied for inbound updates or outbound updates.

**For inbound updates the order of preference is:**

1.  route-map

2.  filter-list

3.  prefix-list, distribute-list

**For outbound updates the order of preference is:**

1.  prefix-list, distribute-list

2.  filter-list

3.  route-map

**NOTE** The attributes prefix-list and distribute-list are mutually exclusive, and only one command (neighbor prefix-list or neighbor distribute-list) can be applied to each inbound or outbound direction for a particular neighbor.

**Filtering Differences**

The tools differ in what they can match in the BGP Update message.

**AS-Path Filter**---Used for ﬁltering autonomous systems. An access list is used in BGP to ﬁlter updates sent from a peer based on the autonomous system path.

-   Standard ACL: Prefix, with wild card mask

-   Extended ACL: Prefix and prefix length, with wild card mask for each.

**Distribute lists** ---Used to ﬁlter routing updates. Although they are often used in redistribution, they are not speciﬁc to redistribution; they can be applied to inbound and outbound updates to or from any peer. Both preﬁx lists and distribute lists ﬁlter on network numbers, not autonomous system paths, for which autonomous system path access lists are used.

**Prefix list** ---Used for ﬁltering preﬁxes, particularly in redistribution. Preﬁx lists ﬁlter based on the preﬁx of the address. Often performs the same function as a distribute-list, but with easier configuration.

**Route maps** ---Used to deﬁne routing policy. A route map is a sophisticated access list that deﬁnes criteria upon which a router acts when a match is found for the stated criteria. It is used in BGP for setting the attributes that determine the basis for selecting the best path to a destination.

After configuration of any filtering technique the peer session needs to be renewed. A soft reset can be done to avoid disruption in the network; reference [Clearing BGP Sessions.]

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## AS Path Filter List

A filter list is a form of route policy that restricts the routes that will be advertised or accepted based on the AS-Path of the route. To configure a filter list, you must first create an AS-path access list based on the known paths you wish to permit.

**AS-Path Filter Configuration**

**Create AS-Path Filter**

```
Router(config)# ip as-path access-list 1-500[permit | deny] *regexp*
```

**Apply Filter to BGP session**

```
Router(config)# router bgp *asn*
Router(config-router)# neighbor *neighbor-ip* filter-list *1-500* [in | out]
```

**NOTE:** Reference [Regular Expressions] for configuration parameters to defines AS.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Distribute lists

To restrict routing information that the router learns or advertises, you can use filters based on routing updates. The filters consist of an access list or a prefix list, which is applied to updates to neighbors and from neighbors.

**Outbound Distribute-List**

An outbound distribute list assures that you do not announce routes heard from one of your peers to another peer. An outbound list restricts your announcements to only those routes you own and can reach.

**Inbound Distribute-List**

If you are an Internet Service Provider, you will also need to restrict the routes your downstream customers and peers announce to you. You will need a complete list of routes from that customer to apply to the inbound routes announced by your customer. This is the most common reason for an inbound distribute list, but you should always apply one for 'sanity checking' to block private and non-routable IP addresses (such as 192.168.0.0 or 127.0.0.1).

![distributelists.gif](/Images/BGP/distributelists.gif)

**Scenario**

Router 200 announces these networks to its peer Router 100:

-   192.168.10.0/24

-   10.10.10.0/24

-   10.10.0.0/19

**Distribute-list Configuration (standard ACL)**

**Syntax**

Router(config-router)# neighbor *ip-address* distribute-list *acl* [in | out]

This sample configuration enables Router 100 to deny an update for network 10.10.10.0/24 and permit the updates of networks 192.168.10.0/24 and 10.10.0.0/19 in its BGP table:

**Configure Standard ACL**

```
Router100(config)# ip access-list 1 deny 10.10.10.0 0.0.0.255
Router100(config)# ip access-list 1 permit any
```

**Apply Distribute-List**

```
Router100(config)# router bgp 100
Router100(config-router)# neighbor 172.16.1.2 remote-as 200
Router100(config-router)# neighbor 172.16.1.2 distribute-list 1 in
```

**Distribute-list Configuration (extended ACL)**

Assume Router 200 announces these networks:

-   10.10.1.0/24 through 10.10.31.0/24

-   10.10.0.0/19 (its aggregate)

Router 100 wishes to receive only the aggregate network, 10.10.0.0/19, and to filter out all specific networks.

A standard access list, such as access-list 1 permit 10.10.0.0 0.0.31.255, will not work because it permits more networks than desired.

The standard access list looks at the network address only and can not check the length of the network mask. That standard access-list will permit the /19 aggregate as well as the more specific /24 networks.

To permit only the supernet 10.10.0.0/19, use an extended access list, such as access-list 101 permit ip 10.10.0.0 0.0.0.0 255.255.224.0 0.0.0.0

This allows the extended access-list command to permit an exact match of source network number 10.10.0.0 with mask 255.255.224.0 (and thus, 10.10.0.0/19). The other more specific /24 networks will be filtered out.

**Configure Extended ACL**

```
Router100(config)# ip access-list 199 permit ip 10.10.0.0 0.0.0.0 255.255.224.0 0.0.0.0
```

**Apply Distribute-List**

```
Router100(config)# router bgp 100
Router100(config-router)# neighbor 172.16.1.2 remote-as 200
Router100(config-router)# neighbor 172.16.1.2 distribute-list 1 in
```

**Troubleshoot/Verification**

```
Router# show ip bgp
```

-   Displays BGP route table, can be used to verify distribute-list operation.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Prefix lists

Prefix lists are more sophisticated forms that Cisco provides for filtering BGP route advertisements. They filter on IP address just as distribute-lists do, however they are easier to read, and require fewer commands to configure. The other advantage to a distribute list is that it is easier to add, remove and organize the statements in the manner you chose.

While this configuration requires the same number of statements as the distribute list example, you have the option of adding **ge**, or **le** to make statements more flexible as to how you will permit blocks in that range.

For example:

```
Router(config)# prefix-list 100 seq 10 permit 63.1.0.0/16 ge 18
```

The statement above allows any route announcement in the range of 63.1.0.0 - 63.1.255.255 but that announcement must have a length greater than 18 bits in the mask. This permits you to allow announcements in the range, but not an announcement equaling the entire range (/16), or even announcements of half the range (/17). Only announcements with a length "greater than or equal to" /18 will be permitted.

![prefixlists.png](/Images/BGP/prefixlists.png)

**Scenario**

Router 200 announces these networks to its peer Router 100:

-   192.168.10.0/24

-   10.10.10.0/24

-   10.10.0.0/19

**Prefix-List Configuration**

The configuration shown below using a prefix-list performs the following actions.

-   Permit updates for any network with a prefix mask length less than or equal to 19.

-   Deny all network updates with a network mask length greater than 19.

**Syntax**

```
Router(config)# ip prefix-list *prefix-name* sequence *number* permit | deny *[IP prefix network/length, e.g., 35.0.0.0/8]*
```

-   Creates prefix-list

```
Router(config-router)# neighbor x.x.x.x prefix-list prefix-name[in | out]
```

-   Apply filtering inbound or outbound.

**Example**

**Define Prefix-List**

```
Router100(config)# ip prefix-list CISCO seq 10 permit 0.0.0.0/0 le 19
```

-   Performs the actions denoted above, recall the explicit deny all at the end of the prefix-list, similar to an ACL.

**Apply Prefix-List**

```
Router100(config)# router bgp 100
Router100(config-router)# neighbor 172.16.1.2 remote-as 200
Router100(config-router)# neighbor 172.16..1.2 prefix-list *CISCO* in
```

**Troubleshooting/Verification**

```
Router# show ip bgp
```

-   Displays BGP route table, can be used to verify prefix-list operation.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Route Maps

Used to deﬁne routing policy. A route map is a sophisticated access list that deﬁnes criteria upon which a router acts when a match is found for the stated criteria. It is used in BGP for setting the attributes that determine the basis for selecting the best path to a destination.

Due to the endless possible configuration solutions, only the route-map framework is shown and not any examples.

**Configure Route-Map**

**Route-map Configuration**

```
Router(config)# route-map *name*
Router(config-route-map)# match *condition*
```

-   Prefixes, metric, as-path etc.

```
Router(config-route-map)# set *value*
```

-   Based on the match statement you can set various attributes such as weight, as-path etc.

**Apply route-map to neighbor**

```
Router(config-router)# neighbor *ip-address* route-map *map-name* [in | out]
```

-   This command applies a route-map to inbound or outbound BGP updates.

**Troubleshooting/Verification**

```
Router# show route-map
```

-   Displays route-map configuration

```
Router# show ip bgp
```

-   Displays BGP route table to confirm route-map operation

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Conditional Advertisement

When dealing with BGP, routes are normally advertised "unconditionally". In other words, if a peering session with a neighbor is up, we'll send them our routes. There may be times, however, when we want to refrain from advertising a prefix (or prefixes) to a neighbor. The conditional advertisement feature, says cisco.com, "is useful for multihomed networks, in which some prefixes are advertised to one of the providers only if information from the other provider is not present (this indicates a failure in the peering session or partial reachability)".

To explain this a little better, take a look at the following topology and let's go through a hypothetical scenario:

![conditionaladvertisement.png](/Images/BGP/conditionaladvertisement.png)

**Scenario**

In this scenario, R4 (in AS 65100) represent us. We are multi-homed to R2 (AS 65002) and R5 (AS 65005), our service providers. Our connection to R5 is at 1544 Kbps ("T-1″), our connection to R2 is 128 Kbps over a frame-relay circuit, and both SPs will be sending us a default route via BGP. Normally, we would simply advertise all of our prefixes (we'll have two) to both providers. With a little bit of AS prepending, it may even be possible to get most of our inbound traffic to flow over the R4-R5 link.

For our hypothetical situation, we'll advertise 203.0.113.0/24, which represents the subnet our client PCs are on, to both providers and not attempt to do any manipulation of inbound traffic. Inbound traffic may come over either link. We want to ensure that traffic inbound to our server subnet (198.51.100.0/24), however, always take the faster (R4-R5) link. Even with a bit of manipulation (e.g. AS prepending), it may not be possible to ensure that 100% of inbound traffic comes over the faster link. Enter conditional advertisements.

How we're going to handle this is to only advertise 198.51.100.0/24 to R5. R2 won't be receiving an advertisement for that network directly from us, so it will send inbound traffic through its other connections, eventually transiting R5 and entering our network there. But, you might ask, what happens if our R4-R5 link goes down for whatever reason? At that point, there will be no routes for 198.51.100.0/24 advertised and inbound traffic won't have any way to reach us. Within about 60 seconds, however, the BGP process on our router will notice that we have no longer have reachability to R5 and will begin advertising the 198.51.100.0/24 network to R2. At that point, inbound traffic will begin flowing again.

As mentioned, we'll advertise 203.0.113.0/24 (our "clients" network) to both providers, all the time. We'll advertise 198.51.100.0/24 to R5 (and only R5), as long as that link is up. If, for some reason, it goes down, then we'll begin advertising 198.51.100.0/24 to R2, providing inbound traffic with an alternate, albeit slower, way of reaching us.

**BGP Conditional Advertisement Configuration**

Assume basic configuration has been completed. Make sure you can ping both R2 and R5 from R4. Now, let's configure two loopback interfaces on R4 to represent our "servers" and "clients" subnets:

```
**R4**(config-if)# interface loopback 198
**R4**(config-if)# ip address 198.51.100.1 255.255.255.0
**R4**(config-if)# interface loopback 203
**R4**(config-if)# ip address 203.0.113.1 255.255.255.0
```

Configure the basic BGP session on our peers, R2 and R5. On both of these peers, we'll send R4 a default route over BGP:

```
**R2**(config-subif)# router bgp 65002
**R2**(config-router)# neighbor 172.16.24.2 remote-as 65100
**R2**(config-router)# neighbor 172.16.24.2 default-originate
!
**R5**(config-if)# router bgp 65005
**R5**(config-router)# neighbor 172.16.14.2 remote-as 65100
**R5**(config-router)# neighbor 172.16.14.2 default-originate
```

On R4, we'll do the same, with some other stuff added in. First, we need to ensure that we don't act as a transit AS between our two providers. We'll prevent this by creating an access list restricting what prefixes we advertise to our neighbors, and apply an outbound distribute-list to those peers. Let's go ahead and begin advertising our two networks (to both peers, for now) as well:

```
**R4**(config)# access-list 25 permit 198.51.100.0
**R4**(config)# access-list 25 permit 203.0.113.0
**R4**(config)# router bgp 65100
**R4**(config-router)# neighbor 172.16.24.1 remote-as 65002
**R4**(config-router)# neighbor 172.16.24.1 distribute-list 25 out
**R4**(config-router)# neighbor 172.16.14.1 remote-as 65005
**R4**(config-router)# neighbor 172.16.14.1 distribute-list 25 out
**R4**(config-router)# network 198.51.100.0 mask 255.255.255.0
**R4**(config-router)# network 203.0.113.0 mask 255.255.255.0
```

On R4, we should now see routes for 172.16.24.1/30 and 172.16.14.1/30 (as well as our locally originated routes) in our BGP table:

```
**R4**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
Network Next Hop Metric LocPrf Weight Path
* 0.0.0.0 172.16.14.1 0 0 65005 i
*172.16.24.1 0 0 65002 i
*198.51.100.0 0.0.0.0 0 32768 i
*203.0.113.0 0.0.0.0 0 32768 i
```

So far, so good.

Let's make an AS path access list that defines an AS path that came directly from AS 65005

```
**R4**(config-router)# ip as-path access-list 1 permit ^65005$
```

Now, we need to create two more access lists. One will match the default route we receive from R5, the other will match the route we want to conditionally advertise to R2 (198.51.100.0/24). Reference [regular expressions] for additional information regarding this syntax.

```
**R4**(config)# access-list 5 permit 0.0.0.0 255.255.255.255
**R4**(config)# access-list 2 permit 198.51.100.0 0.0.0.255
```

Next up, we need to create two route maps (an "advertise-map" and a "non-exist-map") and apply that config to our neighbor R2.
 
```
**R4**(config)# route-map ADVERTISE permit 10
**R4**(config-route-map)# match ip address 2
**R4**(config-route-map)# route-map NON-EXIST permit 10
**R4**(config-route-map)# match ip address 5
**R4**(config-route-map)# match as-path 1
```

Let's go ahead and apply this to our neighbor R2 and clear the BGP process, then I'll explain how it works.

```
**R4**(config-route-map)# router bgp 65100
**R4**(config-router)# neighbor 172.16.24.1 advertise-map ADVERTISE non-exist-map NON-EXIST
**R4**(config-router)# do clear ip bgp *
```

What we've done is told our BGP process that for our neighbor 172.16.24.1 (R2), advertise the routes described by the ADVERTISE route-map (thus, routes matching ACL 2) when the routes described by the NON-EXIST route-map (thus, routes matching ACL 5 with an AS Path as described by ACL 1). It seems like a lot to digest all at once, but if you break it down into its various parts, it becomes quite clear.

Let's take a look at our BGP tables. On R4, we should see exactly what we saw earlier: the default routes from R2 and R5 and our two locally originated routes:

```
**R4**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
* 0.0.0.0 172.16.14.1 0 0 65005 i
*172.16.24.1 0 0 65002 i
*198.51.100.0 0.0.0.0 0 32768 i
*203.0.113.0 0.0.0.0 0 32768 i
```

On R5, we should see routes to both of our networks 198.51.100.0/24 and 203.0.113.0/24:
 
```
**R5**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
*198.51.100.0 172.16.14.2 0 0 65100 i
*203.0.113.0 172.16.14.2 0 0 65100 i
```

Last, on R2, we should only see a route to 203.0.113.0/24, since the R4-R5 link is up:

```
**R2**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
*203.0.113.0 172.16.24.2 0 0 65100 i
```

Still, so far so good. Now, we test!

Let's go to R5 and shut down the serial 0/1 interface that's connected to R4. This will cause the BGP session to drop, which R4 will notice and, after a moment, begin advertising the 198.51.100.0/24 network to R2.

```
**R5**(config-router)# interface serial 0/1
**R5**(config-if)# shutdown
```

Wait a moment, then look at R2′s BGP table again:

```
**R2**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
Network Next Hop Metric LocPrf Weight Path
*198.51.100.0 172.16.24.2 0 0 65100 i
*203.0.113.0 172.16.24.2 0 0 65100 i
```

There's te route to 198.51.100.0/24 from AS 65100, exactly what we wanted! Now, what happens when the connection between R4 and R5 comes back up?

```
**R5**(config-if)# no shutdown
```

The BGP session between R4 and R5 will come back up, R4 will receive the default route from R5, the BGP process will notice, and the route for 198.51.100.0/24 that is being advertised to R2 will be withdrawn:

```
**R2**(config-router)# do show ip bgp | begin Network
Network Next Hop Metric LocPrf Weight Path
*203.0.113.0 172.16.24.2 0 0 65100 I
```
 
**Troubleshooting/Verification**

```
Router# show ip bgp
```

-   Displays BGP route table, useful for verification purposes.

```
Router# show route-map
```

-   Displays route-map configuration
 
```
Router# show ip route
```

-   Displays IP routing table, i.e. where what routes are currently active.

*Source: [Evil Routers - Conditional Advertisement](http://evilrouters.net/2010/03/05/bgp-conditional-advertisements/)*

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Conditional Route Injection

With conditional route injection we can insert more specific routes into a BGP table based on the existence of another route. Most of the routes in the current internet BGP table consists of aggregate routes. This is used to minimize the size and number of routes in global BGP routing table. The aggregation of routes can sometimes obscure more specific and accurate routing information. Wouldn't it be cool if we could control and "un-aggregate" those routes on demand? Well that's kinda what BGP conditional route injection does. It allows us to originate a more specific prefix into the BGP routing table based on an existing aggregated route.

![conditionalrouteinjection1.png](/Images/BGP/conditionalrouteinjection1.png)

**Initial Setup**

Let's set up the basic topology above. We will advertise the 1.1.1.0/25 and 1.1.1.128/25 loopbacks on R1 as an aggregate address 1.1.1.0/24 into the bgp routing table. R2, and R3 will not be able to see the specific routes but only the aggregate route.

**R1 Initial Configuration**

```
R1(config)# router bgp 1
R1(config-router)# network 1.1.1.0 mask 255.255.255.128
R1(config-router)# network 1.1.1.128 mask 255.255.255.128
R1(config-router)# aggregate-address 1.1.1.0 255.255.255.0 summary-only
R1(config-router)# neighbor 192.168.12.2 remote-as 2
```

Pretty simple. R1, R2, and R3 are in BGP AS 1, 2, and 3 respectively. Let's verify the BGP tables on R1 and R2.

```
R1#show ip bgp
BGP table version is 6, local router ID is 1.1.1.129
Status codes: s suppressed, d damped, h history, * valid, best, i - internal,
r RIB-failure, S Stale
Origin codes: i - IGP, e - EGP, ? - incomplete

Network Next Hop Metric LocPrf Weight Path
s1.1.1.0/25 0.0.0.0 0 32768 i
*1.1.1.0/24 0.0.0.0 32768 i
s1.1.1.128/25 0.0.0.0 0 32768 i
```

You can see above that R1 has the more specific routes and the aggregate in its BGP table. The more specific routes are being suppressed, so the only route that R2 and R3 should see is that aggregate route (1.1.1.0/24).

![conditionalrouteinjection2.png](/Images/BGP/conditionalrouteinjection2.png)

**BGP Inject-Map**

We will be using the *bgp inject-map* command to originate some routes. We will set up R2, so that if the 1.1.1.0/24 aggregate routes exist in its BGP table it will "un-aggregate" those routes. R2 will test if it is receiving the aggregate address from R1, before it originates the more specific routes.

**Identify Route, Route-Source and Learned Route for Conditional Route Injection**

```
R2(config)# ip prefix-list ROUTE seq 5 permit 1.1.1.0/24
!
R2(config)# ip prefix-list ROUTE_SOURCE seq 5 permit 192.168.12.1/32
!
R2(config)# route-map LEARNED_ROUTE permit 10
R2(config-route-map)# match ip address prefix-list ROUTE
R2(config-route-map)# match ip route-source prefix-list ROUTE_SOURCE
```

Firstly we have defined the route that we want to match using a prefix list (ROUTE), and source of that prefix using another prefix list (ROUTE_SOURCE). The route prefix list must match prefix we are looking for in the BGP table exactly, and the route source prefix list MUST be a /32 source. We have tied the two together in a route-map. The route map is matching both the route and where we learned it from.

So that's the first part. We will be using that route-map later on in the BGP inject-map to specify the condition that must exist.

**Define what to originate if condition exists**

```
R2(config)# ip prefix-list UNAGGREGATED_ROUTES seq 5 permit 1.1.1.0/25
R2(config)# ip prefix-list UNAGGREGATED_ROUTES seq 10 permit 1.1.1.128/25
!
R2(config)# route-map ORIGINATE permit 10
R2(config-route-map)# set ip address prefix-list UNAGGREGATED_ROUTES
```

We have created a single prefix list called UNAGGREGATED_ROUTES that defines the more specific routes we want to originate. We can originate any route that is a subnet of the aggregate address (so I could have just as easily originated three routes 1.1.1.0/26, 1.1.1.64/26, 1.1.1.128/25), however they must be a subnet of the aggregate address.

We have tied this together with another route-map called ORIGINATE. This will set the ip prefixes we want to originate based on the UNAGGREGATED_ROUTES prefix list. ie ORIGINATE those UNAGGREGATED_ROUTES. :)

So we have two route-maps. ORIGINATE is the route-map containing prefixes we want to originate. LEARNED_ROUTE is the condition we want to match.

**Applying R2 Inject-Map**

```
R2(config)# router bgp 2
R2(config-router)# bgp inject-map ORIGINATE exist-map LEARNED_ROUTE
```

Finally, we have our BGP inject-map statement. The inject-map command takes two arguments. The first is a route-map containing prefixes we want to originate. The second is a route-map that contains the conditions that must be met before we originate the prefixes defined in the first route-map.

**Summary:**

-   BGP conditional route injection allows us to originate more specific prefixes based on an existing aggregate prefix.

-   We use the bgp inject-map command to perform conditional route injection

-   We can only originate more specific subnets of an existing aggregate prefix

**Troubleshooting/Verification**

```
Router# show ip bgp
```

-   Displays BGP route table; can be used to verify aggregated routes.

-   If they unaggregated routes don't show up check whether you have defined a /32 route source and that you are matching a prefix correctly.

```
Router# show ip bgp injected-paths
```

-   Verify what BGP paths are injected

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Clearing BGP Sessions

There are a couple ways, some more recommended than others, to reset or clear a BGP session between two neighbors.

The Cisco IOS Software Command Summary lists the following circumstances under which you must reset a BGP connection, for changes to take effect:

-   Additions or changes to BGP-related access lists

-   Changes to BGP-related weights

-   Changes to BGP-related distribution lists

-   Changes to the BGP-related timer's specifications

-   Changes to the BGP administrative distance

-   Changes to BGP-related route maps

**Traditional Clearing of BGP Session (Hard Reset)**

-   Tears down the BGP session  with all Neighbors, a specific neighbor or peer group.

-   A new session is re-established within 30-60 seconds, depending.

-   Processing of the full Internet table can take a really long time.

-   This is almost NEVER recommended on production networks, due the disruptive behavior.

**Initiate Hard Reset**

```
Router# clear ip bgp {* | neighbor ip | peer-group}
```

-   * clears all sessions, or a more specific neighbor or peer group can be reset.

**Soft Reconfiguration**

**Outbound Soft Reconfiguration**

-   Re-sends your complete BGP Table,

-   Is not configurable and is enabled by default.

**Initiate Outbound Soft Reconfiguration**

Router# clear ip bgp *neighbor-ip* soft out

-   This will resend your BGP table to the neighbor, soft reconfig outbound.

**Inbound Soft Reconfiguration**

-   Requires configuration beforehand.

-   Both neighbors need to support soft reset route refresh capability.

-   Stores complete BGP table of you neighbor in router memory.

-   Not a good idea on a peering router with full feed, due to the memory requirements.

-   Handy for testing purpose, but recommended only for testing.

**Enable Soft Reconfiguration**

```
Router(config-router)# router bgp 12345
Router(config-router)# neighbor *neighbor-ip* soft-reconfig [inbound]
```

-   This will enable soft-reconfiguration inbound for the neighbor, stores neighbor routes in memory.

**Initiate Inbound Soft Reconfiguration**

```
Router# clear ip bgp *neighbor-ip* soft in
```

-   This takes all the routes in memory and re-applies your filters on those routes.

**Route Refresh (Soft Reset)**

-   Used to request a neighbor to resend routing information, without bringing a session down.

-   Useful after configuration changes to update BGP tables.

-   Route-refresh-capability must be supported by both neighbors.

-   Route-refresh-capability is negotiated upon BGP peer session establishment.

-   Requires no additional memory like Soft-Reconfiguration.

-   This is the PREFERRED way to have your updates applied.

**Initiate Route Refresh**

```
Router# clear ip bgp {* | neighbor ip | peer-group} in
```

-   * clears all sessions, or a more specific neighbor or peer group can be reset.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Outbound Route Filtering ORF

The BGP Prefix-Based Outbound Route Filtering feature uses Border Gateway Protocol (BGP) outbound route filter (ORF) send and receive capabilities to minimize the number of BGP updates that are sent between BGP peers. Configuring this feature can help reduce the amount of system resources required for generating and processing routing updates by filtering out unwanted routing updates at the source. For example, this feature can be used to reduce the amount of processing required on a router that is not accepting full routes from a service provider network.

Generally eBGP speakers filter advertisement separately, the sender applies its outbound filter and the receiver applies its inbound filters to received updates.

The obvious goal of filtering at the receiver is to deny certain prefixes, sent by eBGP peer, according to its own policy... So why send them at all?

ORF (Outbound Route Filtering) is a step further in the cooperation between the two autonomous systems, where the receiver send  its inbound filter to the sender to be used as outbound filter, thus avoiding unnecessary updates in the link and CPU resources related to update generation and inbound filtering.

![outboundroutefilteringorf.png](/Images/BGP/outboundroutefilteringorf.png)

**Scenario**

R1 is configured with a inbound filter to only allow 20.20.10.0/24. Because R2 is unaware of this it sends all prefixes and R1 simply filters out the sent routes. The configuration shown below enables both routers to communicate and R2 is now aware that it only needs to send the routes that are allowed to pass through R1.

**Outbound Route Filtering (ORF) Configuration**

```
R1(config)# ip prefix-list only_202010 seq 5 permit 20.20.10.0/24
R1(config)# router bgp 65500
R1(config-router)# neighbor 2.2.2.2 prefix list only_202010 in
R1(config-router)# neighbor 2.2.2.2 capability orf prefix-list {**send** | receive | both}
```

-   The ORF capability can be enabled in send or receive mode with the corresponding keywords. ORF capabilities can also be enabled in send and receive mode with the **both** keyword. Remember that ORF "sends" its inbound filter to its neighbor.

```
R2(config)# router bgp 65501
R2(config-router)# neighbor 1.1.1.1 capability orf prefix-list {send | **receive** | both}
```

-   Since R2 has no need to filter routes, it doesn't need to "send" its inbound filter. Instead it simply receives all routes and this command is simply configured to enable ORF between both neighbors.

**Troubleshooting/Verification**

```
Router# show ip bgp
```

-   Displays BGP route table

```
Router# debug ip bgp updates *[in | out]*
```

-   Enables debugging for incoming BGP updates, keyword can be defined to designate packets *in* or *out*.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Local AS

The local-AS feature allows a router to appear to be a member of a second autonomous system (AS), in addition to its real AS. This feature can only be used for true eBGP peers. You cannot use this feature for two peers that are members of different confederation sub-ASs.

The local-AS feature is useful if ISP-A purchases ISP-B, but ISP-B's customers do not want to modify any peering arrangements or configurations. The local-AS feature allows routers in ISP-B to become members of ISP-A's AS. At the same time, these routers appear to their customers to retain their ISP-B AS number.

**Figure 1**

-   ISP-A has not yet purchased ISP-B. In Figure 2, ISP-A has purchased ISP-B, and ISP-B uses the local-AS feature.

![localas1.gif](/Images/BGP/localas1.gif)

**Figure 2**

-   ISP-B belongs to AS 100, and ISP-C to AS 300. When peering with ISP-C, ISP-B uses AS 200 as its AS number with the use of the **neighbor** *ISP-C* **local-as 200** command. In updates sent from ISP-B to ISP-C, the AS_SEQUENCE in the AS_PATH attribute contains "200 100". The "200" is prepended by ISP-B due to the **local-as 200** command configured for ISP-C.

![localas2.gif](/Images/BGP/localas2.gif)

**Local-AS Configuration**

**ISP-B**

```
ISP-B(config)# router bgp 100
```

-   ISP A (AS 100) this is now used by all routers in ISP-B after its acquisition by ISP-A.

```
ISP-B(config-router)# neighbor 192.168.1.2 remote-as 300
```

-   Defines eBGP connection to ISP-C

```
ISP-B(config-router)# neighbor 192.168.1.2 local-as 200
```

-   This command makes the remote router in ISP-C to see this router as belonging to AS 200 instead of AS 100. This also makes this router prepend AS200 in all updates to ISP-C.  

**ISP-C**

```
ISP-C(config)# router bgp 300
ISP-C(config-router)# neighbor 192.168.1.1 remote-as 200
```

-   Defines the eBGP connect to ISP-B, note AS is 200 not AS 100.

**Troubleshooting/Verification**

```
Router# show ip bgp summary
```

-   Displays basic information regarding BGP, including neighbors and their AS number.

```
Router# show ip bgp
```

-   Displays BGP route table

```
Router# show ip bgp neighbor *neighbor-ip*
```

-   Displays neighbor details, including real and local AS number.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Route Dampening

Route dampening is a BGP feature designed to minimize the propagation of flapping routes across an internetwork. A route is considered to be flapping when its availability alternates repeatedly. Since BGP routing tables are huge, you don't want that many routing updates to be traveling all over the place every time a route flaps.

**Terminology**

-   **Flap**---A route whose availability alternates repeatedly

-   **History state**---After a route flaps once, it is assigned a penalty and put into history state, meaning the router does not have the best path, based on historical information.

-   **Penalty**---Each time a route flaps, the router configured for route dampening in another autonomous system assigns the route a penalty of 1000. Penalties are cumulative. The penalty for the route is stored in the BGP routing table until the penalty exceeds the suppress limit. At that point, the route state changes from history to damp.

-   **Damp state**---In this state, the route has flapped so often that the router will not advertise this route to BGP neighbors

-   **Suppress limit**---A route is suppressed when its penalty exceeds this limit. The default value is 2000

-   **Half-life**---Once the route has been assigned a penalty, the penalty is decreased by half after the half-life period (which is 15 minutes by default). The process of reducing the penalty happens every 5 seconds, gradually reducing the penalty.

-   **Reuse limit**---As the penalty for a flapping route decreases and falls below this reuse limit, the route is unsuppressed. That is, the route is added back to the BGP table and once again used for forwarding. The default reuse limit is 750. The process of unsuppressing routes occurs at 10-second increments. Every 10 seconds, the router finds out which routes are now unsuppressed and advertises them to the world

-   **Maximum suppress limit**---This value is the maximum amount of time a route can be suppressed. The default value is four times the half-life.

**Route Dampening Configuration**

```
Router(config)# router bgp *asn*
Router(config-router)# route dampening *[half-life| reuse | suppress | max-suppress-time]*
```

-   Default Values: 15 750 2000 60

**NOTE:** Route Dampening can also be applied under an address family.

**Troubleshooting/Verification**

Router# show ip bgp dampening parameters

-   Displays configured parameters, if none are defined -defaults will be used.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Regular Expressions

Regular expressions are strings of special characters that can be used to search and find character patterns. Within the scope of BGP in Cisco IOS regular expressions can be used in show commands and AS-Path access-lists to match BGP prefixes based on the information contained in their AS-Path.

**Character Definitions**

***** all character, mean when you use this it can be any character.

^ start here, for example ^5 means any thing that starts with 5 so it could be 5 or 500 or 54 or 5000000

$ end here, for example 5$ means any string that ends with 5 so it could be 455 or 45 or 5 or 3005

_ start or end or space, you can either start a string or end a string or use it as a simple space. For example _5_ only means 5,

but _5 can mean 5 or 500 or 54 or 5000000 and 5_ can mean 455 or 45 or 5 or 3005.

[] defines useable values/options, for example 5[1234] means it could be 51, 52, 53 or 54. You could also specify range 5[5-8] could mean 54, 56, 57 or 58.

? true or false, for example 5? means its either 5 or nothing.

() group, arithmetic's you have logical grouping, for example ^50(_[1-9]43)?$ and will output something like

50

or

50 143

50 243

...

..

.

+ plus sign, means that at least one character should be present for example 4+ means it can match 4 or 44 or 444 or 44444

**Testing Regular Expressions**

Regular Expressions can be confusing, before implementing them or for lab purposes you can test an expression with a current BGP table and see the results.

```
Router# show ip bgp regex *[expression]*
```

**Common Regular Expressions**

Recall that as each BGP peer sends an update, it prepends its own AS number onto the AS path. This means that the AS path builds from right to left, such that the originating AS is on the rightmost end of the AS path string. The last peer to send an update has its AS on the leftmost end of the path.

  .* - Matches any path information

  ^$ - Matches locally originated routes

  ^100_ - Learned from AS 100

  _100$ - Originated from AS 100

  _100_ - Any instance of AS 100

  ^[0-9]+$ - Directly connected ASes

**Create an AS path access list to perform AS path filtering**

```
Router(config)# ip as-path access-list *as-path-list-number {permit | deny} [regular-expression]*
```

**Apply the AS path access list for filtering**

```
Router(config)# router bgp *asn*
Router(config-router)# neighbor {ip-address | peer-group} filter-list *as-path-list-number* {in | out}
```

**NOTE:** The AS path access list filters the AS paths in routing updates to or from a specific neighbor. You can use the in and out keywords to specify the filter direction. Only one in and one out AS path filter can be configured.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Fast External Fallover FEA

This command is enabled by default on Cisco IOS. The command terminates external BGP sessions of any directly adjacent peer if the link used to reach the peer goes down; without waiting for the hold-down timer to expire. Although this feature improves the BGP conversion time, it may lead to great instability in your BGP table due to a flapping interface.

This feature is applicable for only directly connected EBGP peering sessions (not for multihop eBGP). It can be configured either under the BGP process or on a per-interface basis. Refer to Fast Peering Session Deactivation for iBGP or multihop neighbor loss detection.

**Disabling Fast External Fallover**

Router(config)# router bgp *asn*

Router(config-router)# *no* bgp fast-external-fallover

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Fast Peering Session Deactivation FPSD

BGP fast peering session deactivation improves BGP convergence and response time to adjacency changes with BGP neighbors. This feature is event driven and configured on a per-neighbor basis. When this feature is enabled, BGP will monitor the peering session with the specified neighbor. Adjacency changes are detected and terminated peering sessions are deactivated in between the default or configured BGP scanning interval.

Unlike Fast External Fallover, FPSD is able to monitor a peer session adjacency over iBGP or multihop and does *not* need to be directly connected.

**Fast Peering Session Deactivation Configuration**

```
Router(config-router)# neighbor *ip-address* fall-over
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Next Hop Address Tracking

For each route installed in the BGP table a next hop address must exist and this next hop must be reachable in terms of an IGP. If the next hop is not reachable the route will not be considered for the best path algorithm and will never be used by BGP.

One of the major functions of the BGP scanner is to check the reachability of the next-hops for the routes existing in the BGP table.

The BGP scanner runs every 60 seconds by default to make this housekeeping, however during  this 60 seconds routing black holes may occur if the next hop changed before the BGP scanner timer expires. In the worst case you may have a black hole for 60 seconds.

The Next-hop address tracking feature was created to avoid black holing problems, provide faster convergence and stability. This feature is event driven and its role is  to walk the routing table as soon as the IGP change is detected to adjust the BGP table information. The delay interval between routing table walks is 5 seconds by default, this is optimal for fast tuned IGP. This value can be changed to match the IGP convergence timers for optimum performance.

**Adjusting BGP Next-Hop Address Tracking Interval**

```
Router(config)# router bgp *asn*
R1(config-router)#bgp nexthop trigger delay *seconds*
```

-   Delay value range: 1-100 seconds

**Disabling BGP Next-Hop Address Tracking**

The feature is enabled by default in almost all new IOS releases, but still can be disabled by using the command

```
Router(config)# router bgp *asn*
Router(config-router)# *no* bgp next-hop triggered enable
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Maximum Prefix

The *"maximum-prefix"* feature of BGP lets us dictate how many prefixes a neighbor will allow from another neighbor.

![maximumprefix.png](/Images/BGP/maximumprefix.png)

**Scenario**

Imagine the above scenario where you have a private peering session between two BGP neighbors. You've been told by R7 that you should only receive at most 6 routes or *prefixes*. In this case we could implement a maximum-prefix to limit the number of routes R5 will allow.

**Maximum-Prefix Configuration**

Assume BGP neighbor relationship has been brought up between R5 and R7 and several networks exist behind R7 (6 to be exact).

**Syntax**

```
Router(config-router)# neighbor {*ip-address* | *peer-group-name*} {maximum-prefix *maximum* [*threshold*]} [restart restart-interval] [warning-only]
```

**Example**

```
R5(config-router)# neighbor 172.16.57.7 maximum-prefix 10 7 restart 5
```

-   This will tell the BGP process that we will accept, at most, 10 prefixes from 172.16.57.7. When 70% of that threshold is reached (7 prefixes), a log message will be generated. Once we have received 10 prefixes from R7, any more will cause the router to kill the peering session (if we specify *"warning-only"* the session will not be dropped). After the specified *"restart-interval"* (5 minutes, in our case) the peering session will be re-established.

**Troubleshooting/Verification**

```
Router(config-router)# do show ip bgp summary | begin Neighbor
```

-   Displays neighbor statistics; including the number of prefixes being sent by a neighbor.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## BGP Policy Accounting

Border Gateway Protocol (BGP) policy accounting measures and classifies IP traffic that is sent to, or received from, different peers. Policy accounting is enabled on an input interface, and counters based on parameters such as community list, autonomous system number, or autonomous system path are assigned to identify the IP traffic.

You can use this feature to account for IP traffic differentially on an edge router by assigning counters based on BGP prefixes and attributes on a per-input interface basis.

BGP Policy Accounting relies on community-lists and route-maps, reference [*communities*] notes before attempting to understand this configuration.

**BGP Policy Accounting**

In this scenario let's assume you want set a specific "traffic-index" described above to different areas in your network that have been defined by the community attribute.

**Specify communities in community-lists (or define AS-path lists) that classify traffic for accounting.**

```
Router(config)# ip community-list 30 permit 100:190
Router(config)# ip community-list 40 permit 100:198
Router(config)# ip community-list 50 permit 100:197
Router(config)# ip community-list 60 permit 100:296
Router(config)# ip community-list 70 permit 100:201
```

**Define a route-map to match community lists and set appropriate bucket numbers.**

BGP policy accounting (BPA) is another BGP feature that takes advantage of the FIB policy parameters. In this case, the parameter is traffic index. Traffic index is a router internal counter within a FIB leaf with values between 1 and 8. Think of the traffic index as a table of eight independent buckets. Each can account for one type of traffic matching certain criteria. The number of packets and bytes in each bucket of an interface is recorded.

```
Router(config)# route-map set_bucket permit 10
Router(config-route-map)# match community 30
Router(config-route-map)# set traffic-index 2
!
Router(config)# route-map set_bucket permit 20
Router(config-route-map)# match community 40
Router(config-route-map)# set traffic-index 3
!
Router(config)# route-map set_bucket permit 30
Router(config-route-map)# match community 50
Router(config-route-map)# set traffic-index 4
!
Router(config)# route-map set_bucket permit 40
Router(config-route-map)# match community 60
Router(config-route-map)# set traffic-index 5
!
Router(config)# route-map set_bucket permit 50
Router(config-route-map)# match community 70
Router(config-route-map)# set traffic-index 6
```

**Use the table-map command under BGP to modify the bucket number when the IP routing table is updated with routes learned from BGP.**

```
Router(config)# router bgp 1
Router(config-router)# table-map *set_bucket*
```

-   Set_bucket defines the route-map that keeps track of the traffic index.

**Enable the policy accounting feature on the input interface connected to the customer.**

```
Router(config)# interface serial0/0
Router(config-if)# bgp-policy accounting
```

**Configuring BGP Policy Accounting - Output Interface Accounting**

The configuration of BGP PA Output Interface Accounting is very similar to BGP PA. The first three step described in the previous section are exactly the same. The only change is in the *bgp-policy accounting* command that is used to enable the PA feature on the interface. The PA criteria is based on the source address of the output traffic instead of the input as configured above.

**Output Interface Accounting Configuration**

```
Router(config)# interface serial0/0
Router(config-if)# bgp-policy accounting *output source*
```

**Troubleshooting/Verification**

```
Router# show ip cef *ip-address* detail
```

-   Displays what bucket (traffic index) is assigned to a specific prefix.

```
Router# show ip bgp *ip-address*
```

-   Displays what community (or communities) are assigned to a specific prefix.

```
Router# show cef interface policy-statistics
```

-   Displays per-interface traffic statistics.
