## IP Routing Index
 - [Routing Decisions](#routing-decisions)
 - [Default Routing](#default-routing)
 - [Switching Paths](#switching-paths)
 - [Layer 2 Resolution](#layer-2-resolution)
 - [OER Cisco Optimized Edge Routing](#oer-cisco-optimized-edge-routing)
    - [PFR Performance Routing](#pfr-performance-routing)
 - [ODR On Demand Routing](#odr-on-demand-routing)
 - [Secondary IP Address](#secondary-ip-address)
 - [Static Routing](#static-routing)
    - [Floating Static Route](#floating-static-route)
 - [Backup Interface](#backup-interface)
 - [GRE Tunneling](#gre-tunneling)
 - [PBR Policy Based Routing](#pbr-policy-based-routing)
 - [31 Bit Mask](#31-bit-mask)
 - [IP Unnumbered](#ip-unnumbered) 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Routing Decisions 
Understanding how routing decisions are made by a router is crucial for understanding the concepts of routing protocols as well as for design and troubleshooting purposes. 
 
## Routing Decision Process 
**Prefix Length** - The longest-matching route is preferred first. Prefix length trumps all other route attributes. 

**Administrative Distance** - In the event there are multiple routes to a destination with the same prefix length, the route learned by the protocol with the lowest administrative distance is preferred. 

**Metric** - In the event there are multiple routes learned by the same protocol with same prefix length, the route with the lowest metric is preferred. (If two or more of these routes have equal metrics, load balancing across them may occur.) 
 
Example:
Protocol | AD | Metric | Prefix | Next Hop
-------- | -- | ------ | ------ | --------
OSPF | 110 | 240 | 192.020125 | 172.16.1.1
EIGRP | 90 | 33789 | 192.0.2.0/24 | 172.16.2.1
RIP | 120 | 6 | 192.0.264/26 | 172.16.3.1
 
Based on this routing table, the next hop of 172.16.3.1 would be chosen because it has the longest match (prefix length).  Even if a directly connected route or a link with a 100Gb connection is in the routing table.  The route with the longest match will always be preferred.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Default Routing

A default route, also known as the gateway of last resort, is the network route used by a router when no other known route exists for a given IP packet's destination address. All the packets for destinations not known by the router's routing table are sent to the default route. This route generally leads to another router, which treats the packet the same way: If the route is known, the packet will get forwarded to the known route. If not, the packet is forwarded to the default-route of that router which generally leads to another router. And so on. Each router traversal adds a one-hop distance to the route. 
 
The default route in IPv4 (in CIDR notation) is 0.0.0.0/0, often called the quad-zero route. Since the subnet mask given is /0, it effectively specifies no network, and is the "shortest" match possible. A route lookup that doesn't match anything will naturally fall back onto this route. Similarly, in IPv6 the default address is given by ::/0. 
 
Routers in an organization generally point the default route towards the router that has a connection to a network service provider. This way, packets with destinations outside the organization's local area network (LAN)—typically to the Internet, WAN, or VPN—will be forwarded by the router with the connection to that provider. 
 
Host devices in an organization generally refer to the default route as a default gateway which can be, and usually is, a filtration device such as a firewall or Proxy server. 
 
## Default Route Configuration 

There are three effective configuration methods to specify a default route.  The ip default-gateway command is used when IP routing is disabled on the router. However, ip default-network and ip route 0.0.0.0/0 are effective when IP routing is enabled on the router and they are used to route any packets which do not have an exact route match in the routing table. 
 
## Method 1: ip default-gateway

For instance, if the router is a host in the IP world, you can use this command to define a default gateway for it. You might also use this command when your low end Cisco router is in boot mode in order to TFTP a Cisco IOS® Software image to the router. In boot mode, the router does not have ip routing enabled. 
 
This example defines the router on IP address 172.16.15.4 as the default route: 

```
Router(config)# ip default-gateway 172.16.15.4
```

## Method 2: ip default-network 

Unlike the ip default-gateway command, you can use ip default-network when ip routing is enabled on the Cisco router. When you configure ip default-network the router considers routes to that network for installation as the gateway of last resort on the router.  
 
This example defines the router on IP address 172.16.24.0 as the default route: 

```
Router(config)# ip default-network 171.70.24.0 
``` 
## Method 3: ip route 0.0.0.0/0

Creating a static route to network 0.0.0.0 0.0.0.0 is another way to set the gateway of last resort on a router. As with the ip default-network command, using the static route to 0.0.0.0 is not dependent on any routing protocols. However, ip routing must be enabled on the router.  
 
*NOTE:  Using a static route will typically not propagate into a routing process.  Manual redistribution or default-information originate will be required.*  
 
This is an example of configuring a gateway of last resort using the ip route 0.0.0.0 0.0.0.0 command and designating 170.170.3.4 as the default route:  

```
Router(config)#ip route 0.0.0.0 0.0.0.0 170.170.3.4
```
----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switching Paths

The routing process assesses the source and destination of traffic based on knowledge of network conditions. Routing functions identify the best path to use for moving the traffic to the destination out one or more of the router interfaces. The routing decision is based on various criteria such as link speed, topological distance, and protocol. Each protocol maintains its own routing information.  
 
Routing is more processing intensive and has higher latency than switching as it determines path and next hop considerations. The first packet routed requires a lookup in the routing table to determine the route. The route cache is populated after the first packet is routed by the route-table lookup. Subsequent traffic for the same destination is switched using the routing information stored in the route cache.  
 
## Process Switching

In process switching the first packet is copied to the system buffer. The router looks up the Layer 3 network address in the routing table and initializes the fast-switch cache. The frame is rewritten with the destination address and sent to the outgoing interface that services that destination. Subsequent packets for that destination are sent by the same switching path. The route processor computes the cyclical redundancy check (CRC). Process switching is the default operation. 
 
## Fast Switching

When packets are fast switched, the first packet is copied to packet memory and the destination network or host is found in the fast-switching cache. The frame is rewritten and sent to the outgoing interface that services the destination. Subsequent packets for the same destination use the same switching path. The interface processor computes the CRC. 
 
Enable Fast Switching 

```
Router(config)# ip route-cache 
``` 

## CEF Switching

When CEF mode is enabled, the CEF FIB and adjacency tables reside on the RP, and the RP performs the express forwarding. You can use CEF mode when line cards are not available for CEF switching or when you need to use features not compatible with dCEF switching.  Further CEF configuration options can be found here. 
 
Enable CEF Switching 

```
Router(config)# ip cef
```

## dCEF Switching
In distributed switching, the switching process occurs on VIP and other interface cards that support switching. When dCEF is enabled, line cards, such as VIP line cards or GSR line cards, maintain an identical copy of the FIB and adjacency tables. The line cards perform the express forwarding between port adapters, relieving the RSP of involvement in the switching operation. dCEF uses an Inter Process Communication (IPC) mechanism to ensure synchronization of FIBs and adjacency tables on the RP and line cards.  
  
Enable dCEF Switching 

```
Router(config)# ip cef distributed 
```

## Netflow Switching

NetFlow evolved as a caching technique. To speed up network flows (source IP, source port, destination IP, destination port) and Layer 3 switching in the presence of access lists, the Cisco router and switch caches were re-organized based on the flow information. As this code became more efficient, a side benefit was the collection of useful flow statistics, without too severe a performance penalty. Even with CEF (Cisco Express Forwarding) for rapid Layer 3 switching, NetFlow caching can apparently still enhance performance of longer access lists (more than 10 to 25 entries or so), Policy Routing, and perhaps other features ("NetFlow feature acceleration"). But there is also real benefit to the reporting data it provides. Two reasons you might be using NetFlow: to speed certain access list uses up, or to collect data. 
 
Enable Netflow Switching 

```
Router(config)# ip route-cache flow
```

*NOTE: NetFlow Switching in this section is purely for speed purposes, data collection is covered in-depth under "IP Services".*

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Layer 2 Resolution
Network devices often times will have either a logical address (IP) or a physical address (MAC) and will need to determine each other in order to communicate.   
 
Address Resolution Protocol (ARP), a network layer protocol used to convert an IP address into a physical address, such as a MAC address. A host wishing to obtain a physical address broadcasts an ARP request onto the TCP/IP network. The host on the network that has the IP address in the request then replies with its physical hardware address. 
 
Reverse ARP (RARP) which can be used by a host to discover its IP address. In this case, the host broadcasts its physical address and a RARP server replies with the host's IP address.  
 
Display ARP Mappings 

```
Router# show ip arp
```

Displays entire ARP table, including the IP to MAC address mapping, age, and outgoing interface. 

```
Router# show ip arp [ip-address] [host-name] [mac-address] [interface type/number]
```

Using operators you can specify what host you are searching for within the ARP table. 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## OER Cisco Optimized Edge Routing 

Optimized Edge Routing was created to extend the capability of routers to more optimal route traffic than routing protocols can provide on their own.   
 
OER uses the following information in order to better calculate the best path 

 - Packet loss 
 - Response time 
 - Path availability 
 - Traffic load distribution 
 
By adding this information into the routing decision process, OER can influence routing to avoid links with unacceptable latency, packet loss, or network problems serve enough to cause serious application performance problems, but not severe enough to trigger routing changes by the routing protocols in use.  Furthermore, taking into account that many modern networks use multiple service provider circuits and typically do little  or no load balancing between them. OER performs these functions automatically, but also allows for network administrators to manually configure them in a highly granular way if desired.  
 
Five-phase OER operational model 

**Profile** - Learn the flows of traffic that have high latency or high throughput. 
**Measure** - Passively/actively collect traffic performance metrics. 
**Apply Policy** - Create low and high thresholds to define in-policy and out-of-policy (OOP) performance categories. 
**Control** - Influence traffic by manipulating routing or by routing in conjunction with PBR. 
**Verify** - Measure OOP event performance and adjust policy to bring performance in-policy. 
 
OER and PfR influence traffic by collecting information and then injecting new routes into the routing table with the appropriate routing information, tags and other attributes (for BGP routes) to steer traffic in a desired direction.  The new routes are redistributed into the IGP. As conditions change, these new routes may be removed, or more may be added. To provide for the required level of granularity, OER and PfR can split up subnets or extract part of a subnet or prefix from the remainder of that prefix by injecting a longer match into the routing table.  Because the longest match is the first criteria in a Cisco router's decision-making process about where to send traffic. OER and PfR don't require any deep changes in how routers make decisions.  You can think of this feature set as providing more information to the router to help it make better routing decisions, on a flow-by-flow basis. 
 
*NOTE: OER was the original technology added to enhance routing decisions and further enhanced and renamed to PfR, continue to Performance Routing (PfR) subpage for additional concepts and  configuration.*

## PFR Performance Routing

PfR official standards for performance routing, but it's also known as protocol-independent routing optimization, or PIRO.  PfR learns about network performance using IP SLA (Service Level Agreement) and Netflow features (one or both).

PfR Requirements 

 - CEF (Cisco Express Forwarding) must be enabled. 
 - IGP/BGP routing must be configured and working. 
 - PfR does not support Multiprotocol Label Switching Provider Edge - Customer (MPLS PE-CE) or any traffic within the MPLS network, because PfR does not recognize MPLS headers. 
 - PfR uses redistribution of static routes into the routing table, with a tag to facilitate control.  
 
Background Information  
PfR extends beyond OER's original capability by providing  routing optimization based on traffic type, through application awareness. PfR lets a router select the best path across a network based on the application traffic requirements. For example, voice traffic requires low latency, low jitter, and low error rates. 
 
Attributes of PfR 

 - Optimizes traffic path based on application type, performance requirements, and network performance. 
 - Can perform passive monitoring using the Cisco IOS NetFlow feature. 
 - Can perform active monitoring using the Cisco IP IOS IP SLA feature. 
 - Can perform active and passive monitoring simultaneously . 
 - Performs dynamic load balancing 
 - Performs automatic path optimization 
 - Offers "good" mode: finds an alternative route when a defined threshold is exceeded. 
 - Offers "best" mode: always switches traffic to the route with the best performance. 
 - Can reroute traffic in as little as 3 seconds 
 - Supports robust reporting for traffic analysis and path assessment and troubleshooting purposes. 
 - Can split prefixes in the routing table to provide differential routing for a single host or a subnet of hosts compared to the prefixes in the original routing table. 
 - Can operate in monitor-only mode to collect information that helps network administrators determine the benefit of implementing PfR. 
 
Device Roles 
**Master Controller (MC)** 
Configured using the oer master command, this device is the decision maker in the cluster of PfR routers. Learns information from the border routers and makes configuration decisions for the network based on this information. 
 
**Border Router (BR)** 
Configured with the oer border command. Provides information to the master and accepts commands from the MC. 
 
*NOTE: It's possible for a single router to act as both the MC and the BR.*
 
**High Availability and Failure Considerations** 
BR and MC routers maintain communication using keepalives. If keepalives from the MC stops the BR will remove PfR configuration and return to its pre PfR state. More than one MC can be used for failover purposes. PfR traffic classes can be defined by IP address, protocol, port numbers or even DSCP markings. 
 
**Active Mode**: PfR uses the IP SLA feature. BRs source probes to the MC for delay, jitter, reachability , or mean opinion source (MOS). A MOS is calculated using voice-like packets generated using the IP SLA feature to measure jitter, latency, and packet loss. 
 
**Passive Mode**: PfR uses NetFlow information based on traffic classes to make decisions. 
 
Performance Routing (PfR) Configuration 

![performanceroutingconfiguration.jpg](/Images/IP_Routing/performanceroutingconfiguration.jpg)
 
**Create key chain on the Master Controller (MC) - R1**

The key is used for communications between the MC and BR. 

```
R1(config)# key chain cisco 
R1(config-keychain)# key 1 
R1(config-keychain-key)# key-string cisco
```
 
**Master Controller (MC) Initial Configuration** 

```
R1(config)#oer master 
Enables device as Master Controller (MC) 
R1(config-oer-mc)#logging 
Enables logging 
R1(config-oer-mc)#border 136.1.12.2 key-chain cisco 
Specifies IP address of BR with a key-chain for authentication 
R1(config-oer-mc-br)#interface Tunnel1 external 
R1(config-oer-mc-br-if)#interface Tunnel2 external 

```
Tunnel 1 and 2 are specified as the possible external paths on the BR. 
```

R1(config-oer-mc-br-if)#interface FastEthernet0/0 internal 
Interface connecting the MC and the BR (internal) 
R1(config-oer-mc-learn)# exit
 ```

**Master Controller (MC) Learning Configuration**  

Specifies for the controller to learn prefixes based on throughput and delay.  Time periods are set regarding learning and specify the prefix lengths to learn. In this case /32 routes will be learned and monitored. 

```
R1(config-oer-mc)#learn 
R1(config-oer-mc-learn)#throughput 
R1(config-oer-mc-learn)#delay 
R1(config-oer-mc-learn)#periodic-interval 1 
R1(config-oer-mc-learn)#monitor-period 2 
R1(config-oer-mc-learn)#expire after time 30 
R1(config-oer-mc-learn)#aggregation-type prefix-length 32 
Default /24 (class C) 
R1(config-oer-mc-learn)# exit
```

**Master Controller (MC) Additional Properties**

```
R1(config-oer-mc)#backoff 180 360 
Specifies how often to make prefix decisions 
R1(config-oer-mc)#mode route control 
Allows PfR to put routes into the routing table  
R1(config-oer-mc)#mode select-exit best 
Specifies "best" exit interface mode - view OER notes regarding modes. 
R1(config-oer-mc)#periodic 90 
Specifies how often to make policy decisions 
R1(config-oer-mc)#resolve loss priority 1 variance 1 
R1(config-oer-mc)#resolve delay priority 2 variance 1 
R1(config-oer-mc)#resolve utilization priority 3 variance 1 
R1(config-oer-mc)#resolve range priority 4 
Specifies item policy importance regarding making policy decisions.  (Highest 1 to 10 Lowest) 
```

**Border Router (BR) Configuration** 
Define key chain for authentication and specify the outgoing interface and IP address to reach the MC.  Additionally an interface can be specified to use for when sending an active probe, in the event the MC requests one.  Active probes are one method (from many available in PfR) to determine delay. 

``` 
R2(config)# key chain cisco 
R2(config-keychain)#  key 1 
R2(config-keychain-key)#  key-string cisco 
! 
R2(config)#oer border 
R2(config-oer-br)#local FastEthernet0/0 
R2(config-oer-br)#master 136.1.12.1 key-chain cisco 
R2(config-oer-br)#active-probe address source interface Loopback0
```

*NOTE: PfR has a direct relationship with IP SLA in order to analyze network related information and make path selection decisions.  However, this section strictly covers only PfR configuration.  Reference IP Services for IP SLA configuration.*
 
Troubleshooting/Verification 

```
Router# show oer master
```

Displays output to confirm configuration on MC and connectivity to BRs. 

``` 
Router# show oer border
```

Displays MC/BR border address and state as well as exit points (internal & external) depending on what's configured on the MC. 
 
``` 
Router# show ip route
```

Displays routing table, any redistributed routes will be shown as static.  

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## ODR On Demand Routing

Optimized Edge Routing was created to extend the capability of routers to more optimal route traffic than routing protocols can provide on their own.  

OER uses the following information in order to better calculate the best path 

	• Packet loss  
	• Response time  
	• Path availability  
	• Traffic load distribution  
	
By adding this information into the routing decision process, OER can influence routing to avoid links with unacceptable latency, packet loss, or network problems serve enough to cause serious application performance problems, but not severe enough to trigger routing changes by the routing protocols in use.  Furthermore, taking into account that many modern networks use multiple service provider circuits and typically do little  or no load balancing between them. OER performs these functions automatically, but also allows for network administrators to manually configure them in a highly granular way if desired. 

Five-phase OER operational model
Profile - Learn the flows of traffic that have high latency or high throughput.
Measure - Passively/actively collect traffic performance metrics.
Apply Policy - Create low and high thresholds to define in-policy and out-of-policy (OOP) performance categories.
Control - Influence traffic by manipulating routing or by routing in conjunction with PBR.
Verify - Measure OOP event performance and adjust policy to bring performance in-policy.

OER and PfR influence traffic by collecting information and then injecting new routes into the routing table with the appropriate routing information, tags and other attributes (for BGP routes) to steer traffic in a desired direction.  The new routes are redistributed into the IGP. As conditions change, these new routes may be removed, or more may be added. To provide for the required level of granularity, OER and PfR can split up subnets or extract part of a subnet or prefix from the remainder of that prefix by injecting a longer match into the routing table.  Because the longest match is the first criteria in a Cisco router's decision-making process about where to send traffic. OER and PfR don't require any deep changes in how routers make decisions.  You can think of this feature set as providing more information to the router to help it make better routing decisions, on a flow-by-flow basis.

NOTE: OER was the original technology added to enhance routing decisions and further enhanced and renamed to PfR, continue to Performance Routing (PfR) subpage for additional concepts and  configuration. 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Secondary IP Address

Cisco IOS supports multiple IP addresses on a single interface.  There will be a one Primary IP address and the possibility of multiple Secondary IP Addresses on the interface.  Configuring multiple IP addresses on your device can sometimes help when you have multiple subnets having one single physical router interface and multiple subnets accessing it.

The IP addresses can be from different subnets or from entirely different networks.

**Secondary IP Configuration**

```   
Router(config)# interface fa0/0  
Router(config-if)# ip address 10.10.10.1 255.255.255.0  
```

• Defines Primary IP Address  

```
Router(config-if)# ip address 10.10.20.1 255.255.255.0 [secondary]  
```

• Defines the Secondary IP address by the keyword secondary  

Now host's will be able to access the same physical interface and be assigned to different networks.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Static Routing

Static routing is a core technology that any network engineer must understand. Its the ability to statically configure a route  to a defined network with the next transit path hop to get to that network or using the outgoing interface.

For example If router R1 is connected to network 10.61.10.0/24 and PC’s on that network need to get to the 10.61.30.0/24 network then R1 must know where what router to send that traffic to that’s local to R1 that can reach that network.
Let’s say R1 passes this traffic off to R2 and R2 see’s that the network is not directly connected so then R2 then must forward the traffic to the “next hop” in the transit path to get to a router that has that network directly attached. So it then passes it off to R3 which has the 10.61.88.0/24 network directly connected to interface Gi3/10.

Take a step back for a minute and think about bi-directional traffic. If you have static routes pointing in one direction does that necessarily mean that IP communication will be successful? What if the router R3 has no route back to 10.61.21.0/24? This means that traffic from 10.61.21.0/24 can get to the 10.61.88.0/24 network but traffic from the 10.61.88.0/24 network cannot get back. So with that being the case any PC on the 10.61.21.0/24 can send traffic to the 10.61.88.0/24 network but not receive any response back.

Commonly static routes are used for floating routes and a default route which is discussed in lab 6-3 however, many engineers rely on static routes in their infrastructure due to a lack of understanding of dynamic routing protocols such as RIP, EIGRP and OSPF. A well designed network should have very few static routes as the general rule of thumb; when you configure a static route and the network changes, you’ll then potentially need to reassess and reconfigure the static route to ensure network reachability.

**Static Routing Configuration**
Shown below is a logical topology of the network you will be building in this lab. Check out the overall lab topology to view the physical topology. However; looking down on this network you’ll see the topology is built upon the operational function of each network device as shown below:

![staticrouting.png](/Images/IP_Routing/staticrouting.png)

In this lab the Loopback0 interface on R1, R2 and R3 will simulate their connected networks which you will be configuring static routing for.

Syntax
Router(config)# ip route <destination-address> <destination-mask> <next-hop-address | outgoing interface>

Example
``` 
Router(config)# IP route 192.168.20.0 255.255.255.0 192.168.20.5 
```

This effectively says to get to network 192.168.20.0/24 go to the next-hop of 192.168.20.5. Additionally you could also use the outgoing interface to reach the destination network in lieu of a next-hop-address. 
	
Create a Static Route on R1 that states to get to 10.61.20.0/24 go to the next hop of 10.61.12.2 then place the return route on R2 stating to get to 10.61.10.0/24 go to the next hop of 10.61.12.1. Verify that the routes added operate correctly by pinging R2′s Lo0 interface sourced from R1′s Lo0 interface.

```
R1(config)# ip route 10.61.20.0 255.255.255.0 10.61.12.2
!
R2(config)# ip route 10.61.10.0 255.255.255.0 10.61.12.1
```

Troubleshooting/Verification
```
Router# show ip route [static]
```
Displays the routing table, including static routes; optional static operator will display only static routes. 

```	
Router(config)# debug ip routing
```
Displays routing changes and notifications

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Floating Static Route

A floating static route is simply a static route configured with a higher administrative distance than the current route in the routing table.  It will only be used if the more preferable routes for a destination fail.

**Floating Static Route Configuration**
```
Router(config)# ip route <destination-address> <destination-mask> <next-hop-address> [administrative distance]
```

Example:
A route has been learned using OSPF with an administrative distance of 110 and is the most preferable route currently in the routing table.  The following floating static route is entered:

```
Router(config)# ip route 10.10.10.11 255.255.252 10.10.10.12 180
```

The above floating static route will be entered into the routing table and allow for communication to continue using the statically configured address.  The administrative distance of 180 is set and is higher than the default of 110 and therefore is never injected into the routing table until the OSPF route fails.

**Troubleshooting/Verification**
```
Router# show ip route
```

• Displays the routing table, in the event of route failure, the static route will be injected and be displayed in the routing table.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Backup Interface

When the router receives an indication that the primary line is down, a backup interface is brought up. You can configure the backup interface to go down once the primary connection has been restored for a specified period. 

**Backup Interface Configuration**
To configure a backup interface enter the interface you wish to supply a backup to.  

```
Router(config)# interface serial0/0
Router(config-if)# backup interface serial0/1
```

This command specifies that serial0/1 will become the backup interface in the event serial0/0 goes down.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## GRE Tunneling 

Generic Routing Encapsulation (GRE) defines a method of tunneling data from one router to another. To tunnel the traffic, the sending router encapsulates packets of one networking protocol, called the passenger protocol, inside packets of another protocol, called the transport protocol, transporting these packets to another router. The receiving router de-encapsulates and forwards the original passenger protocol packets.  This process allows the routers in the network to forward traffic that might not be natively supported by the intervening routers.  For instance, if some routers did not support IP multicast, the IP multicast traffic could be tunneled from one router to another using IP unicast packets.

*NOTE: GRE Tunnels are by nature not secure, but they can be configured with IPsec for increased security. This can also allow for a L3 VPN creation allowing for both a secure VPN tunnel and allowing for layer 3 protocols to traverse the VPN.*

**GRE Tunnel Interface Configuration**

```
Router(config)# interface tunnel <number>
```

• Enters tunnel configuration mode, specifies tunnel interface number.

```
Router(config-if)# tunnel source [ip-address | source-interface]
```

• Specifies a source interface for the tunnel

```
Router(config-if)# tunnel destination [ip-address]
```

• Specifies the destination IP address for the tunnel.

```
Router(config-if)# tunnel mode [gre {ip | multipoint} | dvmrp | ipip | mpls | nos]
```

• Specifies tunnel mode; default GRE - IP.

```
Router(config-if)# keepalive [seconds | retries ]
```

• Enables keepalive on the interface and optionally specifies the time interval and the number of retries.

**GRE Tunnel Configuration Example**

The configuration below is a simple GRE tunnel formed between two routers. 

**Configure Loopback Interface - R1**

Loopbacks are used as an "always on" interface and will allow for the connection to stay online even if the primary egress interface goes down the tunnel could still potentially communicate by traversing another path.

```
Router1(config)# interface loopback0
Router1(config-if)# ip address 150.1.1.1 255.255.255.0
```

**Configure Tunnel Interface (GRE) - R1**

```
Router1(config)# interface Tunnel 0
Router1(config-if)# ip address 192.168.1.1 255.255.255.0
``` 

• Creates Tunnel 0 and assigns the address defined above to the GRE tunnel 

```
Router1(config-if)# interface source Loopback0
```

• Defines Loopback0 as the tunnel source address to be used

```
Router1(config-if)# tunnel destination 150.2.2.2
```

• Defines the above IP address destination to form the tunnel 


**Create Loopback Interface - R2**
```
Router2(config)# interface loopback0
Router2(config-if)# ip address 150.2.2.2 255.255.255.0
```

**Create Tunnel Interface (GRE) - R2**
```
Router2(config)# interface Tunnel 0
Router2(config-if)# ip address 192.168.1.2 255.255.255.0
Router2(config-if)# tunnel source Loopback0
Router2(config-if)# tunnel destination 150.1.1.1
```

**Troubleshooting/Verification**
```
Router# show ip interface brief
```  
  
• Displays interface status and IP address assignment; can be used to confirm tunnel is operational. 

``` 
Router# show interface tunnel <number>
```

• Displays detailed configuration of tunnel interfaces

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## PBR Policy Based Routing

While dynamic routing protocols provide easy deployment, there will be situations in which more specific selection of routing paths can be advantageous. Policy Based Routing  (PBR) provides a flexible mechanism for network administrators to customize the operation of the routing table and the flow of traffic within their networks.

Policy Based Routing (PBR) has an infinite amount of configuration possibilities and depends on what exactly you wish to accomplish and how the network is configured. However, below displays the basic method for how to configure PBR and can be used as a template for other configuration designs.

**Policy Based Routing (PBR) Configuration**
Multihoming is very common and means that your external router connects to multiple connections in case of a failure or specific resources are dedicated to different lines.  Below displays the configuration for an example of Multihoming using PBR/

Define which network to be affected using an ACL

```
Router(config)#ip access-list 10 permit 10.1.1.0 0.0.0.255
```

Create route-map in order to specify match and set criteria.

```
Router(config)# route-map SetNextHop permit 10
Router(config-route-map)# match ip address 10
```

• Match statements are used to match addresses that will be reflected by the set commands, in this scenario ACL 10.

```
Router(config-route-map)# set ip next-hop 192.168.0.1
```

• All addresses under ACL 10 will have their net hop set to 192.168.0.1 

```
Router(config-route-map)#set interface fa0/1
```

• Optionally instead of specifying a specific address, the next hop interface can be defined.
	
**Apply route-map to interface** 
Typically the internal interface which is connected to the network you would like to redirect or perform PBR on.

```
Router(config)# interface fa0/0
Router(config-if)# ip policy route-map SetNextHop
```

**Troubleshooting/Verification**

```
Router#show route-map
```

• Displays route-map configuration

```
Router#traceroute
```

Using an extended traceroute a specified address that is being manipulated can be tested to see if it's traversing the correct path and that PBR is performing correctly.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## 31 Bit Mask

In order to conserve IP address space on the Internet, a 31-bit prefix length allows the use of only two IP addresses on a point-to-point link. Previously, customers had to use four IP addresses or unnumbered interfaces for point-to-point links.

Using a 31-bit prefix length leaves only two numbering possibilities, 0 and 1. In a point-to-point link with a 31-bit subnet mask, these two addresses must be interpreted as host addresses, and directed broadcast to the link will be eliminated. Limited broadcast must be used for all broadcast traffic on a point-to-point link with a 31-bit mask assigned to it. 

**Configuration**

```
Router(config)# interface serial0/0
Router(config-if)# ip address 10.10.10.1 255.255.255.254
```

• Sets the interface with a /31 subnet mask; the opposite end of the point-to-point connection will be .2

```
Router(config-if)# no ip directed-broadcast
```

Disables broadcasts from IP packets that have a destination address of a particular IP subnet.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Unnumbered

IP-Unnumbered allows for enabling of IP processing on a serial interface without assigning it an explicit IP address. The IP unnumbered interface can "borrow" the IP address of another interface already configured on the router, which conserves network and address space. 

**IP-Unnumbered Configuration**
Consider the network shown below. Router A has a serial interface S0 and an Ethernet interface E0.  

![ipunnumbered.gif](/Images/IP_Routing/ipunnumbered.gif)

**Configure interface as IP-Unnumbered**

```
Router(config)# interface serial 0
Router(config-if)# ip unnumbered Ethernet 0
```

• Borrows the IP address from Ethernet 0

*NOTE: IP-Unnumbered can only be configured on point-to-point links. Typically addresses for IP-Unumbered should be borrowed from a Loopback interface which never goes down - if the interface upon which the address is being borrowed goes down, IP-Unnumbered will not function.*

**Troubleshooting/Verification**

```
Router# show ip interface brief
```

Displays interface status and address assignment