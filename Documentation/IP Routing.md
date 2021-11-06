## IP Routing Index
 - [Routing Decisions](#routing-decisions)
 - [Default Routing](#default-routing)
 - [Switching Paths](#switching-paths)

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
Router(config)# ip default-gateway 172.16.15.4 
 
## Method 2: ip default-network** 

Unlike the ip default-gateway command, you can use ip default-network when ip routing is enabled on the Cisco router. When you configure ip default-network the router considers routes to that network for installation as the gateway of last resort on the router.  
 
This example defines the router on IP address 172.16.24.0 as the default route: 
Router(config)# ip default-network 171.70.24.0 
 
## Method 3: ip route 0.0.0.0/0**

Creating a static route to network 0.0.0.0 0.0.0.0 is another way to set the gateway of last resort on a router. As with the ip default-network command, using the static route to 0.0.0.0 is not dependent on any routing protocols. However, ip routing must be enabled on the router.  
 
*NOTE:  Using a static route will typically not propagate into a routing process.  Manual redistribution or default-information originate will be required.*  
 
This is an example of configuring a gateway of last resort using the ip route 0.0.0.0 0.0.0.0 command and designating 170.170.3.4 as the default route:  
Router(config)#ip route 0.0.0.0 0.0.0.0 170.170.3.4 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switching Paths

The routing process assesses the source and destination of traffic based on knowledge of network conditions. Routing functions identify the best path to use for moving the traffic to the destination out one or more of the router interfaces. The routing decision is based on various criteria such as link speed, topological distance, and protocol. Each protocol maintains its own routing information.  
 
Routing is more processing intensive and has higher latency than switching as it determines path and next hop considerations. The first packet routed requires a lookup in the routing table to determine the route. The route cache is populated after the first packet is routed by the route-table lookup. Subsequent traffic for the same destination is switched using the routing information stored in the route cache.  
 
## Process Switching

In process switching the first packet is copied to the system buffer. The router looks up the Layer 3 network address in the routing table and initializes the fast-switch cache. The frame is rewritten with the destination address and sent to the outgoing interface that services that destination. Subsequent packets for that destination are sent by the same switching path. The route processor computes the cyclical redundancy check (CRC). Process switching is the default operation. 
 
## Fast Switching

When packets are fast switched, the first packet is copied to packet memory and the destination network or host is found in the fast-switching cache. The frame is rewritten and sent to the outgoing interface that services the destination. Subsequent packets for the same destination use the same switching path. The interface processor computes the CRC. 
 
Enable Fast Switching ```Router(config)# ip route-cache``` 
 
## CEF Switching

When CEF mode is enabled, the CEF FIB and adjacency tables reside on the RP, and the RP performs the express forwarding. You can use CEF mode when line cards are not available for CEF switching or when you need to use features not compatible with dCEF switching.  Further CEF configuration options can be found here. 
 
Enable CEF Switching ```Router(config)# ip cef```
 
## dCEF Switching
In distributed switching, the switching process occurs on VIP and other interface cards that support switching. When dCEF is enabled, line cards, such as VIP line cards or GSR line cards, maintain an identical copy of the FIB and adjacency tables. The line cards perform the express forwarding between port adapters, relieving the RSP of involvement in the switching operation. dCEF uses an Inter Process Communication (IPC) mechanism to ensure synchronization of FIBs and adjacency tables on the RP and line cards.  
  
Enable dCEF Switching ```Router(config)# ip cef distributed``` 
 
## Netflow Switching

NetFlow evolved as a caching technique. To speed up network flows (source IP, source port, destination IP, destination port) and Layer 3 switching in the presence of access lists, the Cisco router and switch caches were re-organized based on the flow information. As this code became more efficient, a side benefit was the collection of useful flow statistics, without too severe a performance penalty. Even with CEF (Cisco Express Forwarding) for rapid Layer 3 switching, NetFlow caching can apparently still enhance performance of longer access lists (more than 10 to 25 entries or so), Policy Routing, and perhaps other features ("NetFlow feature acceleration"). But there is also real benefit to the reporting data it provides. Two reasons you might be using NetFlow: to speed certain access list uses up, or to collect data. 
 
Enable Netflow Switching ```Router(config)# ip route-cache flow```
 
*NOTE: NetFlow Switching in this section is purely for speed purposes, data collection is covered in-depth under "IP Services".*

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Layer 2 Resolution
Network devices often times will have either a logical address (IP) or a physical address (MAC) and will need to determine each other in order to communicate.   
 
Address Resolution Protocol (ARP), a network layer protocol used to convert an IP address into a physical address, such as a MAC address. A host wishing to obtain a physical address broadcasts an ARP request onto the TCP/IP network. The host on the network that has the IP address in the request then replies with its physical hardware address. 
 
Reverse ARP (RARP) which can be used by a host to discover its IP address. In this case, the host broadcasts its physical address and a RARP server replies with the host's IP address.  
 
Display ARP Mappings ```Router# show ip arp```

Displays entire ARP table, including the IP to MAC address mapping, age, and outgoing interface. ```Router# show ip arp [ip-address] [host-name] [mac-address] [interface type/number]```

Using operators you can specify what host you are searching for within the ARP table. 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

