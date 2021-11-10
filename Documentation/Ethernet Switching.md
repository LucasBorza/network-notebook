# Ethernet Technologies Index
- [Ethernet Technologies Overview](#ethernet-technologies)
  - [Ethernet Configuration](#ethernet-configuration)
  - [PPPOE PPP Over Ethernet](#pppoe-ppp-over-ethernet)
- [Switchport](#switchport) 
  - [Trunk](#trunk) 
    - [Dot1q Tunneling](#dot1q-tunneling) 
    - [L2P Tunneling](#l2p-tunneling)
- [VTP](#vtp)
  - [VLANs](#vlans)
- [Layer 3 Routing](#layer-3-routing)
  - [Router On A Stick](#router-on-a-stick)

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Ethernet Technologies

Ethernet is a family of frame-based computer networking technologies for local area networks (LANs). The name came from the physical concept of the ether. It defines a number of wiring and signaling standards for the Physical Layer of the OSI networking model as well as a common addressing format and Media Access Control at the Data Link Layer.

Ethernet is standardized as IEEE 802.3. The combination of the twisted pair versions of Ethernet for connecting end systems to the network, along with the fiber optic versions for site backbones, is the most widespread wired LAN technology. 

**Cable Types**

Straight-though - Used to connect unlike devices (router to switch, computer to switch) 
Crossover - Used to connect like devices (computer to router, router to router, switch to switch) 
Rollover - Used to connect workstation devices to a network device for configuration (computer to router) 
 
*NOTE: Cisco supports a switch feature that lets the switch figure out if the wrong cable is installed Auto-MDIX (automatic medium-dependent interface crossover) detects the wrong cable and the switch to swap the pair it uses for transmitting and receiving, which will solve cabling issues automatically.*
 
**Common Ethernet Standards** 
IEEE 802.3 - (MAC) Media Access Control, Layer 1 and 2 specifications 
IEEE 802.2 - (LLC) Logical Link Control, Layer 2 specifications 
10BASE-T - Ethernet: 10Mbps speed, twisted-pair copper, 100 meters maximum distance 
IEEE 802.3u - Fast Ethernet: 100Mbps speed, copper and optical cabling (distance dependent on media used) 
IEEE 802.3z - Gigabit Ethernet over optical cabling 
IEEE 802.3ab - Gigabit Ethernet over copper cabling 
 
CSMA/CD (Carrier Sense Multiple Access with Collision Detection) 
When two or more Ethernet frames overlap on the transmission medium at the exact same time, a collision occurs; the collision results in bit errors and lost frames. Collisions are inevitable, CSMA/CD defines how the sending stations can recognize the collisions and retransmit the frame.   
 
The following list outlines the steps in the CSMA/CD process: 
 
A device with a frame to send listens until the Ethernet is not busy. (In other words, the device cannot sense a carrier signal on the Ethernet segment) 
When the Ethernet is not busy, the sender begins sending the frame. 
The sender listens to make sure that no collisions occurred. 
If there was a collision, all stations that sent a frame send a jamming signal to ensure that all stations recognize the collision.  
After the jamming is complete, each sender of one of the original collided frames randomizes a timer and waits that long before resending. (Other stations that did not create the collision do not have to wait to send). 
After all timers expire, the original senders begin again at step 1. 
 
**Collision domain**  
A set of devices that can send frames that collide with frames sent by another device in the same set of devices.  Ethernet switches greatly reduce the number of possible collisions, both through frame buffering and through their more complete Layer 2 logic when compared to Hubs. 
 
**Types of Ethernet Addresses** 
Unicast: Address that represents a single LAN interface. The I/G bit, the most significant bit in the most significant byte, is set to 0. 
Multicast: A MAC address that implies some subset of all devices currently on the LAN. The I/G bit is set to 1. 
Broadcast: An address that means "all devices that reside on this LAN right now." Always a value of hex FFFFFFFFFFFF 
 
I/G Bit: Binary 0 means the address is unicast; Binary 1 means the address is multicast or broadcast. 
U/L Bit: Binary 0 means the address is vendor assigned; binary 1 means the address has been administratively assigned, overriding the vendor assigned address.  
 
**LAN Switch Forwarding Behavior**
The purpose of the switch is to deliver frames to the appropriate destination(s) based on the MAC address on the frame header. 
 
Know unicast: Forwards the frame out the single interface associated with the destination address 
Unknown Unicast: Floods frame out all interfaces, except the interface on which the frame was received.  
Broadcast: Floods frame identically to unknown unicasts 
Multicast: Floods frame identically to unknown unicasts, unless multicast optimizations are configured. 
 
**Switch Internal Processing**
Store-and-forward:  A transmission method by which a device receives a complete message or protocol data unit (PDU) and temporarily stores it in a buffer before forwarding it toward the destination. This allows for the checking of errors and reducing bandwidth, but increases processor load on the switch. 
 
Cut-through:  The switch starts forwarding a frame (or packet) before the whole frame has been received, normally as soon as the destination address is processed. This technique reduces latency through the switch, but decreases reliability. 
 
Fragment-free:  Will hold the frame until the first 64 bytes are read from the source to detect a collision before forwarding. This is only useful if there is a chance of a collision on the source port - so a fully switched network may not benefit from fragment free in comparison to low latency cut through switching. Frames are forwarded before any checksums can be calculated. 
 
This technique can be thought of as a compromise between the high latency / high integrity of store and forward, and the low latency / reduced integrity of cut through switching. 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Ethernet Configuration

Ethernet is extremely easy to configure, these notes go over basic and the most common configuration done under an Ethernet interface and provide a complete example of an Ethernet interface configuration.

Specifying an Ethernet, Fast Ethernet or Gigabit Ethernet Interface

```
Switch(config)# interface ethernet {number | slot/port}
Switch(config)# interface fastethernet {number | slot/port}
Switch(config)# interface gigabitethernet {number | slot/port}
```

*NOTE: Depending on the model different input will be required to enter interface configuration. Either a single number interface or module slot/port interface.*

**Assign an IP address**

```
Switch(config-if)# ip address 192.168.1.1 255.255.255.0
```

**Assign description** 

```
Switch(config-if)#description <Text Goes Here>
```

• Describes the interfaces function, recommended practice for troubleshooting purposes.

**Speed and Duplex** 

Typically interface speed and duplex can auto negotiate, however if it doesn't it can cause suboptimal network performance. Switches can dynamically detect the speed for a particular Ethernet segment using Fast Link Pulses (FLP) however if auto-negotiation is disabled it can still detect the speed by incoming electrical signals.  You can however input manually incorrect speeds and cause the link to no longer function. Switches detect duplex setting through auto-negotiation only, however if it is disabled on either end the switch will set default duplex settings.  Half for 10/100-Mbps or Full for 1000-Mbps interfaces.

```
Switch(config-if)#duplex { full | half | auto }
```

• Sets transmission direction
	Half - Sends data in one direction at a time (legacy)

	Full - Sends data in both directions at the same time (optimal)

	Auto - Attempts to negotiate with a neighbor device to find the correct duplexing, mix-matched duplex's can effect performance considerably.
	
```
Switch(config-if)#speed { 10 | 100 | 1000 | auto }
```

• Sets transmission speed of interface, match speeds on both ends or link will be disabled completely. 
	
```
Switch(config-if)#shutdown
```

• Shutdown interface
	
```
Switch(config-if)#no shutdown
```

• Enable interface

**Encapsulation Method**

The encapsulation method that you use depends upon the routing protocol that you are using, the type of Ethernet media connected to the router or access server, and the routing or bridging application that you configure. 

Currently, there are three common Ethernet encapsulation methods:
 
• The standard Advanced Research Projects Agency (ARPA) Ethernet Version 2.0 encapsulation, which uses a 16-bit protocol type code (the default encapsulation method). 
	
• Service access point (SAP) IEEE 802.3 encapsulation, in which the type code becomes the frame length for the IEEE 802.2 LLC encapsulation (destination and source Service Access Points, and a control byte). 
	
•  The SNAP method, as specified in RFC 1042, Standard for the Transmission of IP Datagrams Over IEEE 802 Networks, which allows Ethernet protocols to run on IEEE 802.2 media. 

```
Switch(config-if)# encapsulation {arpa | sap | snap}
```

• Selects ARPA, SAP or SNAP encapsulation

**Media Type**

This provides an example for media type configuration, however depending on your model of equipment different types of media options will be available.

```
Switch(config-if)# media-type 100basex 
```

• Twisted pair copper media (Default)
	
```
Switch(config-if#) media-type sfp
```

• Small form-factor pluggable, Fiber

**Ethernet Configuration Example**

```
interface FastEthernet0/0
 description WAN Connection to Cox
 ip address 192.168.1.1 255.255.255.0
 shutdown
 speed 100
 full-duplex
 media-type 100basex
 encapsulation arpa
!
```

*NOTE: Although shown in this configuration example, the media type and encapsulation would not be shown in the configuration, because those are the defaults.  Very rarely will the encapsulation need to be adjusted, the media type will obviously depend on the media in use.*

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## PPPOE PPP Over Ethernet

PPPoE provides an emulated (and optionally authenticated) point-to-point link across a shared medium, typically a broadband aggregation network such as those found in DSL service providers. In fact, a very common scenario is to run a PPPoE client on the customer side (commonly on a SOHO Linksys or similar brand router), which connects to and obtains its configuration from the PPPoE server (head-end router) at the ISP side.

**PPPoE Client Configuration**

Create a dialer interface to handle the PPPoE connection

```
Router(config)# interface dialer1
Router(config-if)# dialer pool 1
Router(config-if)# encapsulation ppp
Router(config-if)# ppp chap hostname CPE
Router(config-if)# ppp chap password mYp@ssW0rD
```

• Enables CHAP authentication and defines hostname and password required to authenticate with ISP and obtain an IP address.

```
Router(config-if)# ip address negotiated
```

• Specifies the client to use an address provided by the PPPoE server.
	
Reduce Maximum Transmission Unit (MTU)

```
Router(config-if)# mtu 1492
```

• The PPP header adds 8 bytes of overhead to each frame. Assuming the default Ethernet MTU of 1500 bytes, we'll want to lower our MTU on the dialer interface to 1492 to avoid unnecessary fragmentation.

Apply Dialer interface to the outgoing ISP-facing interface

```
Client(config)# interface FastEthernet0/0
Client(config-if)# no ip address
Client(config-if)# pppoe-client dial-pool-number 1
Client(config-if)# no shutdown
```

**PPPoE Server Configuration**

More than likely Server configuration will not be necessary for the CCIE Lab, however it's a good thing to understand.

Create Broadband Aggregation (BBA) group which will handle incoming PPPoE connection attempts

```
ISP(config)# bba-group pppoe MyGroup
ISP(config-bba-group)# virtual-template 1
```

Set PPPoE Session limits (Optional)

```
ISP(config-bba-group)# sessions per-mac limit 2
```

• Limit the number of sessions established per client MAC address (setting this limit to 2 allows a new session to be established immediately if the prior session was orphaned and is waiting to expire). 
	
**Create Virtual Template for customer-facing interface**

When a PPPoE client initiates a session with this router, the router automatically spawns a virtual interface to represent that point-to-point connection.

```
ISP(config)# interface virtual-template 1
ISP(config-if)# ip address 10.0.0.1 255.255.255.0
ISP(config-if)# peer default ip address pool MyPool
ISP(config-if)# ppp authentication chap callin
```

• Enables authentication using CHAP

**Create IP Address pool for associated clients**

```
ISP(config)# ip local pool MyPool 10.0.0.2 10.0.0.254
```

Enable PPPoE group on the interface facing the customer network

```
ISP(config)# interface FastEthernet0/0
ISP(config-if)# no ip address
ISP(config-if)# pppoe enable group MyGroup
ISP(config-if)# no shutdown
```

Defines remote client account

```
ISP(config)# username CPE password  mYp@ssW0rD
```

• Typically account creation is typically performed on a back-end server and referenced via RADIUS or TACACS+ rather than being stored locally.

**Troubleshooting/Verification**

```
debug pppoe events
```

• Displays PPPoE protocol messages about events that are part of normal session establishment or shutdown

```
debug pppoe authentication
```

• Displays authentication protocol messages such as CHAP and PAP messages

```	
show pppoe session
```

• Displays information about currently active PPPoE sessions

```	
show ip dhcp binding
```

• Displays address binding on the Cisco IOS DHCP server

```
show ip nat translation
```

Displays active NAT translations

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switchport

These notes provide general switchport terminology and various switchport configuration options.  View sub pages for additional information on specific configurations and port modes. 

**Terminology**

**Trunking**
Carrying multiple VLANs over the same physical connection

**Dynamic Trunking Protocol (DTP)**
Can be used to automatically establish trunks between capable ports (insecure)

**Access VLAN** 
The VLAN to which an access port is assigned, typically access ports are for end devices (computers, servers, phones)

**Voice VLAN**
 If configured, enables minimal trunking to support voice traffic in addition to data traffic on an access port

**Switched Virtual Interface (SVI)**
A virtual interface which provides a routed gateway into and out of a VLAN.

**Switchport Initialization** 

1. Spanning Tree Protocol (STP) initialization 

• STP prevents loops in a LAN, you can disable the five phases of STP (blocking, listening, learning, forwarding and disabled).
	
2. Testing for EtherChannel configuration

• EtherChannel allows for bundling of physical links for increased bandwidth and redundancy. 
	
3. Testing for trunk configuration 
  
• Trunking allows for transmission of multiple VLANs over a switchport. 

4. Auto-negotiation (speed & duplex)

• Switchport speed and transmission method (View Ethernet configuration)

**Reducing the Switchport Initialization Process**

```
Switch(config-if)# switchport host
```

• This is a very useful command which configures a port for access layer devices, it automatically configures the following (access mode, enables portfast, disables port-aggregation) and will speed up switchport initialization. 

**Switch Port Modes**

```
Switch(config-if)# switchport mode dynamic desirable
```

• Attempts to negotiate a trunk with the far end

```
Switch(config-if)# switchport mode dynamic auto
```

• Forms a trunk only if requested by the far end

```
Switch(config-if)# switchport mode trunk
```

• Forms an unconditional trunk

```
Switch(config-if)# switchport mode access
```

• Will never form a trunk, reduces port initialization time. 

```
Switch(config-if)# switchport nonegotiate
```

Disable DTP messages, whenever manually forcing a switchport mode (trunk or access)

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Trunk

Trunks are used to carry traffic that belongs to multiple VLANs between devices over the same link. A device can determine which VLAN the traffic belongs to by its VLAN identifier. The VLAN identifier is a tag that is encapsulated with the data. ISL and 802.1Q are two types of encapsulation that are used to carry data from multiple VLANs over trunk links. 

ISL is a Cisco proprietary protocol for the interconnection of multiple switches and maintenance of VLAN information as traffic goes between switches. ISL provides VLAN trunking capabilities while it maintains full wire-speed performance on Ethernet links in full-duplex or half-duplex mode. ISL operates in a point-to-point environment and can support up to 1000 VLANs. In ISL, the original frame is encapsulated and an additional header is added before the frame is carried over a trunk link. At the receiving end, the header is removed and the frame is forwarded to the assigned VLAN. ISL uses Per VLAN Spanning Tree (PVST), which runs one instance of Spanning Tree Protocol (STP) per VLAN. PVST allows the optimization of root switch placement for each VLAN and supports the load balancing of VLANs over multiple trunk links.
 
802.1Q is the IEEE standard for tagging frames on a trunk and supports up to 4096 VLANs. In 802.1Q, the trunking device inserts a 4-byte tag into the original frame and recomputes the frame check sequence (FCS) before the device sends the frame over the trunk link. At the receiving end, the tag is removed and the frame is forwarded to the assigned VLAN. 802.1Q does not tag frames on the native VLAN. It tags all other frames that are transmitted and received on the trunk. When you configure an 802.1Q trunk, you must make sure that you configure the same native VLAN on both sides of the trunk. IEEE 802.1Q defines a single instance of spanning tree that runs on the native VLAN for all the VLANs in the network. This is called Mono Spanning Tree (MST). This lacks the flexibility and load balancing capability of PVST that is available with ISL. However, PVST+ offers the capability to retain multiple spanning tree topologies with 802.1Q trunking. 

**Trunk Port Configuration**

```
Switch(config)# interface FastEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate
```

• Enables trunk mode and disables DTP messages from being sent.

```
Switch(config-if)# switchport trunk allowed vlan 10,30-80
```

• Only the specified VLANs will be allowed to communicate over the trunk. 

```
Switch(config-if)# switchport trunk native vlan 99
```

• Native VLAN must match on both sides of the trunk, no access layer devices should be on the native VLAN.  The VLAN is strictly for the purpose of transporting management protocols such as VTP, STP, DTP etc. Native VLANs are not used in ISL configurations.
	
**Troubleshooting/Verification**

```
Switch# show vlan
```

• Displays all VLANs and what interfaces are assigned to each. 
	
*NOTE: Trunk interfaces are not shown using the command show vlan, only access interfaces.* 

```
Switch# show interface trunk
```

Displays information related to configured trunk ports (mode, encapsulation, vlans and ports)

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Dot1q Tunneling

QinQ adds another layer of IEEE 802.1Q tag (called "metro tag" or "PE-VLAN") to the 802.1Q tagged packets that enter the network. The purpose is to expand the VLAN space by tagging the tagged packets, thus producing a "double-tagged" frame. The expanded VLAN space allows the service provider to provide certain services, such as Internet access on specific VLANs for specific customers, yet still allows the service provider to provide other types of services for their other customers on other VLAN. Although this technology is deployed it's more than likely that an ISP would prefer to deploy a layer 3 model such as Virtual Router Routing (VRF). 
 
Background 
This example is provided in order to assist in understanding QinQ configuration.  Imagine an ISP allowing customers to use their own VLANs and then mask it by a single VLAN defined by the ISP. 
 
![topology.png](/Images/802.1qtopology.jpg)
 
double_encapsulation.png

802.1Q Tunnel Configuration 
Adjust MTU 
AllSwitches# show system mtu 
  System MTU size is 1500 bytes 
AllSwitches(config)# system mtu 1504 
   Changes to the system MTU will not take effect until the next reload is done. 
 
NOTE: Due to the additional tag being on top of a 802.1q frame, an added4 bytes needs to be available within the maximum transmission unit (MTU). 
 
Configure ISP Backbone Trunk 
This will allow QinQ frames across the ISP backbone; customer 1 = VLAN118 and customer 2 = VLAN 209 
 
S1(config)# interface f0/13 
S1(config-if)# switchport trunk encapsulation dot1q 
S1(config-if)# switchport mode trunk 
S1(config-if)# switchport trunk allowed vlan 118,209 
S1(config-if)# no cdp enable 
Disables customer from viewing ISP internal equipment. 
 
S2(config)# interface f0/13 
S2(config-if)# switchport trunk encapsulation dot1q 
S2(config-if)# switchport mode trunk 
S2(config-if)# switchport trunk allowed vlan 118,209 
S2(config-if)# no cdp enable 
 
Customer-facing interfaces
Assign the interface to the appropriate upper-level (service provider) VLAN, and its operational mode to dot1q-tunnel. Layer 2 protocol tunneling is also enabled to allow transparent transmission of Layer 2 protocols such as CDP. 
 
*NOTE: By default, the native VLAN traffic of a dot1q trunk is sent untagged, which cannot be double-tagged in the service provider network. Because of this situation, the native VLAN traffic might not be tunneled correctly. Be sure that the native VLAN traffic is always sent tagged in an asymmetrical link. To tag the native VLAN egress traffic and drop all untagged ingress traffic, enter the global vlan dot1q tag native command.* 
 
Customer 1; Switch 1 

```
S1(config)# interface f0/1 
S1(config-if)# switchport access vlan 118 
S1(config-if)# switchport mode dot1q-tunnel 
S1(config-if)# l2protocol-tunnel 
S1(config)# vlan dot1q tag native 
```

Customer 2; Switch 1 
```
S1(config-if)# interface f0/3 
S1(config-if)# switchport access vlan 209 
S1(config-if)# switchport mode dot1q-tunnel 
S1(config-if)# l2protocol-tunnel 
S1(config)# vlan dot1q tag native 
```

Customer 1; Switch 2 
```
S2(config)# interface f0/2 
S2(config-if)# switchport access vlan 118 
S2(config-if)# switchport mode dot1q-tunnel 
S2(config-if)# l2protocol-tunnel 
S2(config)# vlan dot1q tag native 
```

Customer 2; Switch 2 
```
S2(config-if)# interface f0/4 
S2(config-if)# switchport access vlan 209 
S2(config-if)# switchport mode dot1q-tunnel 
S2(config-if)# l2protocol-tunnel 
S2(config)# vlan dot1q tag native 
```

**Troubleshooting/Verification**

```
Switch# show dot1q-tunnel 
```

Provides a list of dot1q-tunnel interfaces 
 
```
Switch# show vlan dot1q tag native  
```

Display 802.1Q native VLAN tagging status.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## L2P Tunneling

Using Layer Two Protocol (L2P) Tunneling switches can be configured to forward layer 2 protocols such as CDP, STP and VTP frames instead of intercepting them.  L2P provides a level of transparency when forming  adjacencies across an ISP network.  

When protocol tunneling is enabled, edge switches on the inbound side of the service-provider network encapsulate Layer 2 protocol packets with a special MAC address and send them across the service-provider network. Core switches in the network do not process these packets but forward them as normal packets. Layer 2 protocol data units (PDUs) for CDP, STP, or VTP cross the service-provider network and are delivered to customer switches on the outbound side of the service-provider network. Identical packets are received by all customer ports on the same VLANs with these results: 

• Users on each of a customer's sites can properly run STP, and every VLAN can build a correct spanning tree based on parameters from all sites and not just from the local site. 

• CDP discovers and shows information about the other Cisco devices connected through the service-provider network. 

• VTP provides consistent VLAN configuration throughout the customer network, propagating to all switches through the service provider. 
	
**L2P Tunneling Configuration**

Configure Interface as an Access Port or an IEEE 802.1Q tunnel port

```
Switch(config)# interface fastethernet0/1
Switch(config-if)# switchport mode { access | dot1q-tunnel }
```

**Enable L2P Tunneling**

```
Switch(config-if)# l2protocol-tunnel 
```

• Enables CDP, STP and VTP each can be specifically defined if you only wish to tunnel specific protocols
• Disables CDP on the port
	
Enabled point-to-point protocols (optional)

```
Switch(config-if)# l2protocol-tunnel  point-to-point
```

• Expands tunnel to support point-to-point protocols such as PAgP, LACP and UDLD. 
	
Define Thresholds (optional)

```
Switch(config-if)# l2protocol-tunnel shutdown-threshold [cdp | stp | vtp] 1500
```

• Shutdown interface if the defined packets-per-second threshold is reached; range 1 to 4096.

```
Switch(config-if)# l2protocol-tunnel drop-threshold [cdp | stp | vtp] 1000
```

• Drops packets if the defined packets-per-second threshold is reached; range of 1 to 4096

```
Switch(config-if)# l2protocol-tunnel drop-threshold point-to-point [pagp | lacp | udld] 1000
```

• Drops packets if the defined packets-per-second threshold is reached; range of 1 to 4096
	
*NOTE: If both thresholds are configured, the drop threshold must be less or equal the shutdown threshold.*

Define CoS value for tunneled traffic (optional) 

```
Switch(config-if)# l2protocol-tunnel cos [0-7]
```

**Troubleshooting/Verification**

```
Switch# show l2protocol-tunnel
```

• Display the Layer 2 tunnel ports on the switch, including the protocols configured, the thresholds, and the counters. 

```
Switch# show l2protocol-tunnel summary
```

• Display only Layer 2 protocol summary information

```	
Switch# clear l2protocol-tunnel counters
```

• Clear the protocol counters on Layer 2 protocol tunneling ports.

```
Switch# errdisable recovery cause l2ptguard
```

• Enable auto-recovery of L2P tunneled ports which shutdown due to the maximum threshold being reached.

```	
Switch# show dot1q-tunnel
```

• Display IEEE 802.1Q tunnel ports on the switch.

```	
Switch# show vlan dot1q tag native
```

• Display the status of native VLAN tagging on the switch.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

VLAN Trunking Protocol (VTP) is a way to propagate VLANs across your LAN, instead of manually configuring VLANs on each device. 

Terminology
Domain: Common to all switches participating in VTP
Server Mode: Generates and propagates VTP advertisements to clients; default mode on unconfigured switches
Client Mode: Receives and forwards advertisements from servers; VLANs cannot be manually configured on switches in client mode.
Transparent Mode: Forwards advertisements but does not participate in VTP; VLANs must be configured manually
Pruning: VLANs not having any access ports on an end switch are removed from the trunk to reduce flooded traffic
Revision numbers: VTP works off revision numbers, if VLAN information is changed on a VTP Server this information will be replicated to other switches that are in Server or Client mode.

**VTP Configuration**

```
Switch(config)# vtp mode {server | client | transparent}
Switch(config)# vtp domain <name>
Switch(config)# vtp password <password>
```

• Provides MD5 hash that includes VTP updates and will only propagate VTP information to those devices with the same password. 

```
Switch(config)# vtp version {1 | 2}
Switch(config)# vtp pruning
```

**VTP Pruning**

```
Switch(config)# vtp pruning {enable | disabled }
```

• This command can be entered on a VTP server to enable or disable VTP pruning and will be propagated across the LAN. 
	
```
Switch(config)# clear vtp pruneeligible <vlan range>
```

• (Optional) Make specific VLANs pruning-ineligible on the device.  (By default, VLANs 2-1000 are pruning-eligible.)

```
Switch(config)# set vtp pruneeligible <vlan range>
```

• (Optional) Make specific VLANs pruning-eligible on the device.

**Troubleshoot/Verification**

```
Switch# show vlan
```

• This command can be used to show configured VLANs, note if a port is missing it's in trunk mode.

```	
Switch# show interface trunk
```

• This command is used to display information related to configured trunk ports and to verify the correct VLANs are being pruned. 

```	
Switch# show vtp status
```

• Display the VTP switch configuration information, including VTP pruning.

```	
Switch# show vtp password
```

• The password is encrypted in MD5, this is the only way to view the VTP password. 

```	
Switch# show vtp counters
```

Display counters about VTP messages that have been sent and received. 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## VLANs

VLANs are in place for a number of reasons including but not limited to logically separating broadcast domains, security and separating different traffic types (such as data and voice).

Normal Usable Range: 1-1001
Extended Range: 1006-4094

**VLAN Configuration**

```
Switch(config)#vlan 10
Switch(config-vlan)#name Data
Switch(config)#vlan 20
Switch(config-vlan)#name Voice
```

**Troubleshooting**

```
Switch# delete flash: vlan.dat
```

• Clearing startup-config will not remove VLANs, it's stored in a protected file.  You will need to issue the command above to remove VLAN configuration and then reload the switch.

```
Switch# show vlan
```

• This command can be used to show configured VLANs, note if a port is missing it's in trunk mode.

```	
Switch# show interface trunk
```

• This command is used to display information related to configured Trunk ports.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Layer 3 Routing

Layer 2 switching is hardware based, which means it uses the media access control address (MAC address) from the host's network interface cards (NICs) to decide where to forward frames. Switches use application-specific integrated circuits (ASICs) to build and maintain filter tables (also known as MAC address tables).

The only difference between a Layer 3 switch and router is the way the administrator creates the physical implementation. Also, traditional routers use microprocessors to make forwarding decisions, and the switch performs only hardware-based packet switching. However, some traditional routers can have other hardware functions as well in some of the higher-end models. Layer 3 switches can be placed anywhere in the network because they handle high-performance LAN traffic and can cost-effectively replace routers. Layer 3 switching is all hardware-based packet forwarding, and all packet forwarding is handled by hardware ASICs. Layer 3 switches really are no different functionally than a traditional router and perform the same functions.

This is important when it comes to reducing the amount of hardware needed to perform the same function within your network.  We could use the traditional method of intervlan routing using a router & switch (Router-on-a-Stick).  But in the case of a layer 3 switching, we would only need one device to perform the same task and it would likely have better performance due to its hardware based packet switching.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Router On A Stick

Router-On-A-Stick refers to a common configuration to allow intervlan routing between VLANs.  

The following scenario describes two VLANs (10 - Accounting & 20 - Management) who require 
Intervlan routing using a layer 2 switch and router. 

Create VLANs; assign hosts to an individual VLAN

```
Switch(config)#vlan 10
Switch(config-vlan)#name Accounting
Switch(config)#vlan 20
Switch(config-vlan)#name Management
Switch(config)# interface FastEthernet1/0/2
Switch(config-if)# description HOST-IN-VLAN-10
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config)# interface FastEthernet1/0/3
Switch(config-if)# description HOST-IN-VLAN-20
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 20
```

Create Trunk interface to Router

```
Switch(config)# interface FastEthernet 0/1
Switch(config-if)# description Trunk-To-Router
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk
```

*NOTE: Additional trunk options (DTP, Allowed VLANs etc.) has been removed from the configuration for brevity; view Switchport: Trunk configuration for details.* 

Create Trunk interface to Switch; Subinterfaces corresponding to each VLAN.

```
Router(config)# interface fastethernet 0/0.10
Router(config-if)# encapsulation dot1q 10
Router(config-if)# ip address 10.10.10.1 255.255.255.0
Router(config)# interface fastethernet 0/0.20
Router(config-if)# encapsulation dot1q 20
Router(config-if)# ip address 20.20.20.1 255.255.255.0
```

*NOTE: No IP configuration is done under the physical interface; only under subinterfaces.*

**Troubleshooting/Verification**

```
Switch# show vlan
```

• Displays VLANs and what interfaces are associated with each VLAN; trunk ports will not be displayed.

```	
Switch# show interface trunk
```

• Displays information related to configured trunk ports (mode, encapsulation, VLANs and ports)

```	
Router# show ip route
```

Displays the traditional IP routing table
----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

----------------------------------------------------------------------------------------------------------------------------------------------------------------

