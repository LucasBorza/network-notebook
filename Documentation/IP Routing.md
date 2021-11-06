Overview 
Understanding how routing decisions are made by a router is crucial for understanding the concepts of routing protocols as well as for design and troubleshooting purposes. 
 
Routing Decision Process 
 
Prefix Length - The longest-matching route is preferred first. Prefix length trumps all other route attributes. 
 
Administrative Distance - In the event there are multiple routes to a destination with the same prefix length, the route learned by the protocol with the lowest administrative distance is preferred. 
 
Metric - In the event there are multiple routes learned by the same protocol with same prefix length, the route with the lowest metric is preferred. (If two or more of these routes have equal metrics, load balancing across them may occur.) 
 
Example  
 
Machine generated alternative text:
Protocol
AD
Metric
Prefix
Next Hop
OSPF
110
240
192.020125
172.16.1.1
EIGRP
90
33789
192.0.2.0/24
172.16.2.1
RIP
120
6
192.0.264/26
172.16.3.1
 
Based on this routing table, the next hop of 172.16.3.1 would be chosen because it has the longest match (prefix length).  Even if a directly connected route or a link with a 100Gb connection is in the routing table.  The route with the longest match will always be preferred.   