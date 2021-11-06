## IP Routing Index
 - [Routing Decisions](#routing-decisions)
 - [Default Routing](#default-routing)
 - [Switching Paths](#switching-paths)
 - [Layer 2 Resolution](#layer-2-resolution)
 - [OER Cisco Optimized Edge Routing](#oer-cisco-optimized-edge-routing)
 - [PFR Performance Routing](#pfr-performance-routing)

  OER (Cisco Optimized Edge Routing) 

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

```Router(config)# ip default-gateway 172.16.15.4```
 
## Method 2: ip default-network 

Unlike the ip default-gateway command, you can use ip default-network when ip routing is enabled on the Cisco router. When you configure ip default-network the router considers routes to that network for installation as the gateway of last resort on the router.  
 
This example defines the router on IP address 172.16.24.0 as the default route: 

```Router(config)# ip default-network 171.70.24.0 ```
 
## Method 3: ip route 0.0.0.0/0

Creating a static route to network 0.0.0.0 0.0.0.0 is another way to set the gateway of last resort on a router. As with the ip default-network command, using the static route to 0.0.0.0 is not dependent on any routing protocols. However, ip routing must be enabled on the router.  
 
*NOTE:  Using a static route will typically not propagate into a routing process.  Manual redistribution or default-information originate will be required.*  
 
This is an example of configuring a gateway of last resort using the ip route 0.0.0.0 0.0.0.0 command and designating 170.170.3.4 as the default route:  

```Router(config)#ip route 0.0.0.0 0.0.0.0 170.170.3.4```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switching Paths

The routing process assesses the source and destination of traffic based on knowledge of network conditions. Routing functions identify the best path to use for moving the traffic to the destination out one or more of the router interfaces. The routing decision is based on various criteria such as link speed, topological distance, and protocol. Each protocol maintains its own routing information.  
 
Routing is more processing intensive and has higher latency than switching as it determines path and next hop considerations. The first packet routed requires a lookup in the routing table to determine the route. The route cache is populated after the first packet is routed by the route-table lookup. Subsequent traffic for the same destination is switched using the routing information stored in the route cache.  
 
## Process Switching

In process switching the first packet is copied to the system buffer. The router looks up the Layer 3 network address in the routing table and initializes the fast-switch cache. The frame is rewritten with the destination address and sent to the outgoing interface that services that destination. Subsequent packets for that destination are sent by the same switching path. The route processor computes the cyclical redundancy check (CRC). Process switching is the default operation. 
 
## Fast Switching

When packets are fast switched, the first packet is copied to packet memory and the destination network or host is found in the fast-switching cache. The frame is rewritten and sent to the outgoing interface that services the destination. Subsequent packets for the same destination use the same switching path. The interface processor computes the CRC. 
 
Enable Fast Switching 

```Router(config)# ip route-cache``` 
 
## CEF Switching

When CEF mode is enabled, the CEF FIB and adjacency tables reside on the RP, and the RP performs the express forwarding. You can use CEF mode when line cards are not available for CEF switching or when you need to use features not compatible with dCEF switching.  Further CEF configuration options can be found here. 
 
Enable CEF Switching 

```Router(config)# ip cef```
 
## dCEF Switching
In distributed switching, the switching process occurs on VIP and other interface cards that support switching. When dCEF is enabled, line cards, such as VIP line cards or GSR line cards, maintain an identical copy of the FIB and adjacency tables. The line cards perform the express forwarding between port adapters, relieving the RSP of involvement in the switching operation. dCEF uses an Inter Process Communication (IPC) mechanism to ensure synchronization of FIBs and adjacency tables on the RP and line cards.  
  
Enable dCEF Switching 

```Router(config)# ip cef distributed``` 
 
## Netflow Switching

NetFlow evolved as a caching technique. To speed up network flows (source IP, source port, destination IP, destination port) and Layer 3 switching in the presence of access lists, the Cisco router and switch caches were re-organized based on the flow information. As this code became more efficient, a side benefit was the collection of useful flow statistics, without too severe a performance penalty. Even with CEF (Cisco Express Forwarding) for rapid Layer 3 switching, NetFlow caching can apparently still enhance performance of longer access lists (more than 10 to 25 entries or so), Policy Routing, and perhaps other features ("NetFlow feature acceleration"). But there is also real benefit to the reporting data it provides. Two reasons you might be using NetFlow: to speed certain access list uses up, or to collect data. 
 
Enable Netflow Switching 

```Router(config)# ip route-cache flow```
 
*NOTE: NetFlow Switching in this section is purely for speed purposes, data collection is covered in-depth under "IP Services".*

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Layer 2 Resolution
Network devices often times will have either a logical address (IP) or a physical address (MAC) and will need to determine each other in order to communicate.   
 
Address Resolution Protocol (ARP), a network layer protocol used to convert an IP address into a physical address, such as a MAC address. A host wishing to obtain a physical address broadcasts an ARP request onto the TCP/IP network. The host on the network that has the IP address in the request then replies with its physical hardware address. 
 
Reverse ARP (RARP) which can be used by a host to discover its IP address. In this case, the host broadcasts its physical address and a RARP server replies with the host's IP address.  
 
Display ARP Mappings 

```Router# show ip arp```

Displays entire ARP table, including the IP to MAC address mapping, age, and outgoing interface. 

```Router# show ip arp [ip-address] [host-name] [mac-address] [interface type/number]```

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
![performanceroutingconfiguration.png](/Images/performanceroutingconfiguration.jpg)
 
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
Tunnel 1 and 2 are specified as the possible external paths on the BR. 
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

```Router# show oer master```

Displays output to confirm configuration on MC and connectivity to BRs. 
 
```Router# show oer border```

Displays MC/BR border address and state as well as exit points (internal & external) depending on what's configured on the MC. 
 
```Router# show ip route```

Displays routing table, any redistributed routes will be shown as static.  