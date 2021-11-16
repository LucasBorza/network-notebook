# Switching Index
- [Ethernet Technologies](#ethernet-technologies)
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

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switching Technologies

Ethernet is a family of frame-based computer networking technologies for local area networks (LANs). The name came from the physical concept of the ether. It defines a number of wiring and signaling standards for the Physical Layer of the OSI networking model as well as a common addressing format and Media Access Control at the Data Link Layer.

Ethernet is standardized as IEEE 802.3. The combination of the twisted pair versions of Ethernet for connecting end systems to the network, along with the fiber optic versions for site backbones, is the most widespread wired LAN technology.

**Cable Types**

Straight-though - Used to connect unlike devices (router to switch, computer to switch)
Crossover - Used to connect like devices (*computer to router*, router to router, switch to switch)
Rollover - Used to connect workstation devices to a network device for configuration (computer to router)

**NOTE:** Cisco supports a switch feature that lets the switch figure out if the wrong cable is installed Auto-MDIX (automatic medium-dependent interface crossover) detects the wrong cable and the switch to swap the pair it uses for transmitting and receiving, which will solve cabling issues automatically.

**Common Ethernet Standards**
IEEE 802.3 - (MAC) Media Access Control, Layer 1 and 2 specifications
IEEE 802.2 - (LLC) Logical Link Control, Layer 2 specifications
10BASE-T - Ethernet: 10Mbps speed, twisted-pair copper, 100 meters maximum distance

**IEEE 802.3u** - Fast Ethernet: 100Mbps speed, copper and optical cabling (distance dependent on media used)
**IEEE 802.3z** - Gigabit Ethernet over optical cabling
**IEEE 802.3ab** - Gigabit Ethernet over copper cabling

**CSMA/CD (Carrier Sense Multiple Access with Collision Detection)**
When two or more Ethernet frames overlap on the transmission medium at the exact same time, a collision occurs; the collision results in bit errors and lost frames. Collisions are inevitable, CSMA/CD defines how the sending stations can recognize the collisions and retransmit the frame.

The following list outlines the steps in the CSMA/CD process:
 
1.  A device with a frame to send listens until the Ethernet is not busy. (In other words, the device cannot sense a carrier signal on the Ethernet segment)

2.  When the Ethernet is not busy, the sender begins sending the frame.

3.  The sender listens to make sure that no collisions occurred.

4.  If there was a collision, all stations that sent a frame send a jamming signal to ensure that all stations recognize the collision.

5.  After the jamming is complete, each sender of one of the original collided frames randomizes a timer and waits that long before resending. (Other stations that did not create the collision do not have to wait to send).

6.  After all timers expire, the original senders begin again at step 1.

**Collision domain**
A set of devices that can send frames that collide with frames sent by another device in the same set of devices. Ethernet switches greatly reduce the number of possible collisions, both through frame buffering and through their more complete Layer 2 logic when compared to Hubs.

**Types of Ethernet Addresses**
Unicast: Address that represents a single LAN interface. The I/G bit, the most significant bit in the most significant byte, is set to 0.
Multicast: A MAC address that implies some subset of all devices currently on the LAN. The I/G bit is set to 1.
Broadcast: An address that means "all devices that reside on this LAN right now." Always a value of hex FFFFFFFFFFFF

**I/G Bit:** Binary 0 means the address is unicast; Binary 1 means the address is multicast or broadcast.

**U/L Bit:** Binary 0 means the address is vendor assigned; binary 1 means the address has been administratively assigned, overriding the vendor assigned address.

**LAN Switch Forwarding Behavior**
The purpose of the switch is to deliver frames to the appropriate destination(s) based on the MAC address on the frame header.

**Know unicast:** Forwards the frame out the single interface associated with the destination address
**Unknown Unicast:** Floods frame out all interfaces, except the interface on which the frame was received.
**Broadcast:** Floods frame identically to unknown unicasts
**Multicast:** Floods frame identically to unknown unicasts, unless multicast optimizations are configured.

 

**Switch Internal Processing**
**Store-and-forward:** A transmission method by which a device receives a complete message or protocol data unit (PDU) and temporarily stores it in a buffer before forwarding it toward the destination. This allows for the checking of errors and reducing bandwidth, but increases processor load on the switch.

**Cut-through:** The switch starts forwarding a frame (or packet) before the whole frame has been received, normally as soon as the destination address is processed. This technique reduces latency through the switch, but decreases reliability.

 

**Fragment-free:** Will hold the frame until the first 64 bytes are read from the source to detect a collision before forwarding. This is only useful if there is a chance of a collision on the source port - so a fully switched network may not benefit from fragment free in comparison to low latency cut through switching. Frames are forwarded before any checksums can be calculated.

This technique can be thought of as a compromise between the high latency / high integrity of store and forward, and the low latency / reduced integrity of cut through switching.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Ethernet Configuration

Ethernet is extremely easy to configure, these notes go over basic and the most common configuration done under an Ethernet interface and provide a complete example of an Ethernet interface configuration.

Specifying an Ethernet, Fast Ethernet or Gigabit Ethernet Interface
**Switch(config)# interface ethernet *{number | slot/port}
*Switch(config)# interface fastethernet *{number | slot/port}
*Switch(config)# interface gigabitethernet *{number | slot/port}

***NOTE:** Depending on the model different input will be required to enter interface configuration. Either a single *number* interface or module *slot/port* interface.*

***Assign an IP address**

Switch(config-if)# ip address 192.168.1.1 255.255.255.0

**Assign description**

Switch(config-if)#description <Text Goes Here

-   Describes the interfaces function, recommended practice for troubleshooting purposes.

**Speed and Duplex
**Typically interface speed and duplex can auto negotiate, however if it doesn't it can cause suboptimal network performance. Switches can dynamically detect the **speed** for a particular Ethernet segment using Fast Link Pulses (FLP) however if auto-negotiation is disabled it can still detect the speed by incoming electrical signals. You can however input manually incorrect speeds and cause the link to no longer function. Switches detect **duplex** setting through auto-negotiation only, however if it is disabled on either end the switch will set default duplex settings. Half for 10/100-Mbps or Full for 1000-Mbps interfaces.
**
**Switch(config-if)#duplex { full | half | auto }

-   Sets transmission direction

 Half - Sends data in one direction at a time (legacy)

 Full - Sends data in both directions at the same time (optimal)

 Auto - Attempts to negotiate with a neighbor device to find the correct duplexing, mix-matched duplex's can effect performance considerably.

  

Switch(config-if)#speed { 10 | 100 | 1000 | auto }

-   Sets transmission speed of interface, match speeds on both ends or link will be disabled completely.

  

Switch(config-if)#shutdown

-   Shutdown interface

  

Switch(config-if)#no shutdown

-   Enable interface

**Encapsulation Method
**The encapsulation method that you use depends upon the routing protocol that you are using, the type of Ethernet media connected to the router or access server, and the routing or bridging application that you configure.
 

Currently, there are three common Ethernet encapsulation methods:

-   The standard Advanced Research Projects Agency (ARPA) Ethernet Version 2.0 encapsulation, which uses a 16-bit protocol type code (the default encapsulation method).

  

-   Service access point (SAP) IEEE 802.3 encapsulation, in which the type code becomes the frame length for the IEEE 802.2 LLC encapsulation (destination and source Service Access Points, and a control byte).

  

-   The SNAP method, as specified in RFC 1042, *Standard for the Transmission of IP Datagrams Over IEEE 802 Networks*, which allows Ethernet protocols to run on IEEE 802.2 media.
  

Switch(config-if)# encapsulation {arpa | sap | snap}

-   Selects ARPA, SAP or SNAP encapsulation

 

**Media Type
**This provides an example for media type configuration, however depending on your model of equipment different types of media options will be available.
 

Switch(config-if)# media-type 100basex

-   Twisted pair copper media (Default)

  

Switch(config-if#) media-type sfp

-   Small form-factor pluggable, Fiber
  

**Ethernet Configuration Example
**!

interface FastEthernet0/0

description WAN Connection to Cox

ip address 192.168.1.1 255.255.255.0

shutdown

speed 100

full-duplex
*media-type 100basex
encapsulation arpa*
!

**NOTE:** Although shown in this configuration example, the media type and encapsulation would not be shown in the configuration, because those are the defaults. Very rarely will the encapsulation need to be adjusted, the media type will obviously depend on the media in use.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## PPPoE PPP over Ethernet

PPPoE provides an emulated (and optionally authenticated) point-to-point link across a shared medium, typically a broadband aggregation network such as those found in DSL service providers. In fact, a very common scenario is to run a PPPoE client on the customer side (commonly on a SOHO Linksys or similar brand router), which connects to and obtains its configuration from the PPPoE server (head-end router) at the ISP side.

 

**PPPoE Client Configuration**

**Create a dialer interface to handle the PPPoE connection**

Router(config)# interface dialer1
Router(config-if)# dialer pool 1
Router(config-if)# encapsulation ppp

Router(config-if)# ppp chap hostname *CPE*

Router(config-if)# ppp chap password *mYp@ssW0rD*

-   Enables CHAP authentication and defines hostname and password required to authenticate with ISP and obtain an IP address.

Router(config-if)# ip address negotiated

-   Specifies the client to use an address provided by the PPPoE server.

  

**Reduce Maximum Transmission Unit (MTU)**

Router(config-if)# mtu 1492

-   The PPP header adds 8 bytes of overhead to each frame. Assuming the default Ethernet MTU of 1500 bytes, we'll want to lower our MTU on the dialer interface to 1492 to avoid unnecessary fragmentation.

**Apply Dialer interface to the outgoing ISP-facing interface**

Client(config)# interface FastEthernet0/0
Client(config-if)# no ip address
Client(config-if)# pppoe-client dial-pool-number 1
Client(config-if)# no shutdown

 

**PPPoE Server Configuration**

More than likely Server configuration will not be necessary for the CCIE Lab, however it's a good thing to understand.

 

**Create Broadband Aggregation (BBA) group which will handle incoming PPPoE connection attempts**
ISP(config)# bba-group pppoe MyGroup

ISP(config-bba-group)# virtual-template 1

**Set PPPoE Session limits (Optional)**

ISP(config-bba-group)# sessions per-mac limit 2

-   Limit the number of sessions established per client MAC address (setting this limit to 2 allows a new session to be established immediately if the prior session was orphaned and is waiting to expire).

  

**Create Virtual Template for customer-facing interface**

When a PPPoE client initiates a session with this router, the router automatically spawns a virtual interface to represent that point-to-point connection.

 

ISP(config)# interface virtual-template 1

ISP(config-if)# ip address 10.0.0.1 255.255.255.0
ISP(config-if)# peer default ip address pool *MyPool*

ISP(config-if)# ppp authentication chap callin

-   Enables authentication using CHAP

**Create IP Address pool for associated clients**

ISP(config)# ip local pool *MyPool* 10.0.0.2 10.0.0.254

**Enable PPPoE group on the interface facing the customer network**

ISP(config)# interface FastEthernet0/0
ISP(config-if)# no ip address
ISP(config-if)# pppoe enable group *MyGroup*
ISP(config-if)# no shutdown

**Defines remote client account**

ISP(config)# username CPE password *mYp@ssW0rD*

-   Typically account creation is typically performed on a back-end server and referenced via RADIUS or TACACS+ rather than being stored locally.

 

**Troubleshooting/Verification
**debug pppoe events

-   Displays PPPoE protocol messages about events that are part of normal session establishment or shutdown

  

debug pppoe authentication

-   Displays authentication protocol messages such as CHAP and PAP messages

  

show pppoe session

-   Displays information about currently active PPPoE sessions

  

show ip dhcp binding

-   Displays address binding on the Cisco IOS DHCP server

  

show ip nat translation

-   Displays active NAT translations

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Switchport

These notes provide general switchport terminology and various switchport configuration options. View sub pages for additional information on specific configurations and port modes.

**Terminology**

**Trunking:** Carrying multiple VLANs over the same physical connection

**Dynamic Trunking Protocol (DTP):** Can be used to automatically establish trunks between capable ports (insecure)

**Access VLAN:** The VLAN to which an access port is assigned, typically access ports are for end devices (computers, servers, phones)

**Voice VLAN:** If configured, enables minimal trunking to support voice traffic in addition to data traffic on an access port

**Switched Virtual Interface (SVI):** A virtual interface which provides a routed gateway into and out of a VLAN.

**Switchport Initialization**

1.  Spanning Tree Protocol (STP) initialization

-   STP prevents loops in a LAN, you can disable the five phases of STP (blocking, listening, learning, forwarding and disabled).

  

2.  Testing for EtherChannel configuration

-   EtherChannel allows for bundling of physical links for increased bandwidth and redundancy.

  

3.  Testing for trunk configuration

-   Trunking allows for transmission of multiple VLANs over a switchport.

  

4.  Auto-negotiation (speed & duplex)

-   Switchport speed and transmission method (View Ethernet configuration)

 

**Reducing the Switchport Initialization Process**

Switch(config-if)# switchport host

-   This is a very useful command which configures a port for access layer devices, it automatically configures the following (access mode, enables portfast, disables port-aggregation) and will speed up switchport initialization.

 

**Switch Port Modes**

Switch(config-if)# switchport mode dynamic desirable

-   Attempts to negotiate a trunk with the far end

Switch(config-if)# switchport mode dynamic auto

-   Forms a trunk only if requested by the far end

Switch(config-if)# switchport mode trunk

-   Forms an unconditional trunk

Switch(config-if)# switchport mode access

-   Will never form a trunk, reduces port initialization time.

Switch(config-if)# switchport nonegotiate

-   Disable DTP messages, whenever manually forcing a switchport mode (trunk or access)

 

Access

Friday, August 20, 2010

6:37 PM

 

**Overview
**Port typically assigned to an end node such as a workstation or phone.

 

**Access Port Configuration**

Switch(config)# interface FastEthernet 0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport nonegotiate

-   Enables access mode and disables DTP messages from being sent.

Switch(config-if)#switchport access vlan 10

-   Assigns VLAN 10 (data) to the interface

Switch(config-if)#switchport voice vlan 20

-   Similar command as defined above, yet specific for voice

  

**Troubleshooting/Verification**

Switch# show vlan

-   Displays all VLANs and what interfaces are assigned to each (trunk interfaces are not shown)

 

Trunk

Friday, August 20, 2010

6:35 PM

 

**Overview**
Trunks are used to carry traffic that belongs to multiple VLANs between devices over the same link. A device can determine which VLAN the traffic belongs to by its VLAN identifier. The VLAN identifier is a tag that is encapsulated with the data. ISL and 802.1Q are two types of encapsulation that are used to carry data from multiple VLANs over trunk links.
 

**ISL** is a Cisco proprietary protocol for the interconnection of multiple switches and maintenance of VLAN information as traffic goes between switches. ISL provides VLAN trunking capabilities while it maintains full wire-speed performance on Ethernet links in full-duplex or half-duplex mode. ISL operates in a point-to-point environment and can support up to 1000 VLANs. In ISL, the original frame is encapsulated and an additional header is added before the frame is carried over a trunk link. At the receiving end, the header is removed and the frame is forwarded to the assigned VLAN. ISL uses Per VLAN Spanning Tree (PVST), which runs one instance of Spanning Tree Protocol (STP) per VLAN. PVST allows the optimization of root switch placement for each VLAN and supports the load balancing of VLANs over multiple trunk links.

**802.1Q** is the IEEE standard for tagging frames on a trunk and supports up to 4096 VLANs. In 802.1Q, the trunking device inserts a 4-byte tag into the original frame and recomputes the frame check sequence (FCS) before the device sends the frame over the trunk link. At the receiving end, the tag is removed and the frame is forwarded to the assigned VLAN. 802.1Q does not tag frames on the native VLAN. It tags all other frames that are transmitted and received on the trunk. When you configure an 802.1Q trunk, you must make sure that you configure the same native VLAN on both sides of the trunk. IEEE 802.1Q defines a single instance of spanning tree that runs on the native VLAN for all the VLANs in the network. This is called Mono Spanning Tree (MST). This lacks the flexibility and load balancing capability of PVST that is available with ISL. However, PVST+ offers the capability to retain multiple spanning tree topologies with 802.1Q trunking.

 

**Trunk Port Configuration**

Switch(config)# interface FastEthernet 0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate

-   Enables trunk mode and disables DTP messages from being sent.

Switch(config-if)# switchport trunk allowed vlan 10,30-80

-   Only the specified VLANs will be allowed to communicate over the trunk.

Switch(config-if)# switchport trunk native vlan 99

-   Native VLAN must match on both sides of the trunk, no access layer devices should be on the native VLAN. The VLAN is strictly for the purpose of transporting management protocols such as VTP, STP, DTP etc. Native VLANs are not used in ISL configurations.

  

**Troubleshooting/Verification**

Switch# show vlan

-   Displays all VLANs and what interfaces are assigned to each.

  

**NOTE:** Trunk interfaces are not shown using the command show vlan, only access interfaces.

**
**Switch# show interface trunk

-   Displays information related to configured trunk ports (mode, encapsulation, vlans and ports)

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 8021q Tunneling

QinQ adds another layer of IEEE 802.1Q tag (called "metro tag" or "PE-VLAN") to the 802.1Q tagged packets that enter the network. The purpose is to expand the VLAN space by tagging the tagged packets, thus producing a "double-tagged" frame. The expanded VLAN space allows the service provider to provide certain services, such as Internet access on specific VLANs for specific customers, yet still allows the service provider to provide other types of services for their other customers on other VLAN. Although this technology is deployed it's more than likely that an ISP would prefer to deploy a layer 3 model such as Virtual Router Routing (VRF).

 

**Background
**This example is provided in order to assist in understanding QinQ configuration. Imagine an ISP allowing customers to use their own VLANs and then mask it by a single VLAN defined by the ISP.

 

![](''/media/image1.png){width="5.1875in" height="2.6145833333333335in"}

 

![](''/media/image2.png){width="3.125in" height="0.9270833333333334in"}

**802.1Q Tunnel Configuration**

Adjust MTU

```
AllSwitches# show system mtu
```

System MTU size is 1500 bytes

```
AllSwitches(config)# system mtu 1504
```

Changes to the system MTU will not take effect until the next reload is done.

**NOTE:** Due to the additional tag being on top of a 802.1q frame, an added4 bytes needs to be available within the maximum transmission unit (MTU).

**Configure ISP Backbone Trunk**

This will allow QinQ frames across the ISP backbone; customer 1 = VLAN118 and customer 2 = VLAN 209
 

S1(config)# interface f0/13
S1(config-if)# switchport trunk encapsulation dot1q
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk allowed vlan 118,209
S1(config-if)# no cdp enable

-   Disables customer from viewing ISP internal equipment.
  
S2(config)# interface f0/13
S2(config-if)# switchport trunk encapsulation dot1q
S2(config-if)# switchport mode trunk
S2(config-if)# switchport trunk allowed vlan 118,209
S2(config-if)# no cdp enable

**Customer-facing interfaces**
Assign the interface to the appropriate upper-level (service provider) VLAN, and its operational mode to dot1q-tunnel. Layer 2 protocol tunneling is also enabled to allow transparent transmission of Layer 2 protocols such as CDP.
 

**NOTE:** By default, the native VLAN traffic of a dot1q trunk is sent untagged, which cannot be double-tagged in the service provider network. Because of this situation, the native VLAN traffic might not be tunneled correctly. Be sure that the native VLAN traffic is always sent tagged in an asymmetrical link. To tag the native VLAN egress traffic and drop all untagged ingress traffic, enter the global *vlan dot1q tag native* command.

 

***Customer 1; Switch 1***

S1(config)# interface f0/1
S1(config-if)# switchport access vlan 118
S1(config-if)# switchport mode dot1q-tunnel
S1(config-if)# l2protocol-tunnel

S1(config)# vlan dot1q tag native
 

***Customer 2; Switch 1***
S1(config-if)# interface f0/3
S1(config-if)# switchport access vlan 209
S1(config-if)# switchport mode dot1q-tunnel
S1(config-if)# l2protocol-tunnel
S1(config)# vlan dot1q tag native
 

***Customer 1; Switch 2***

S2(config)# interface f0/2
S2(config-if)# switchport access vlan 118
S2(config-if)# switchport mode dot1q-tunnel
S2(config-if)# l2protocol-tunnel
S2(config)# vlan dot1q tag native
 

***Customer 2; Switch 2***
S2(config-if)# interface f0/4
S2(config-if)# switchport access vlan 209
S2(config-if)# switchport mode dot1q-tunnel
S2(config-if)# l2protocol-tunnel
S2(config)# vlan dot1q tag native

**Troubleshooting/Verification**

Switch# show dot1q-tunnel

-   Provides a list of dot1q-tunnel interfaces

  

Switch# show vlan dot1q tag native

-   Display 802.1Q native VLAN tagging status.

 

L2P Tunneling

Thursday, January 20, 2011

11:46 AM

 

**Overview**

Using Layer Two Protocol (L2P) Tunneling switches can be configured to forward layer 2 protocols such as CDP, STP and VTP frames instead of intercepting them. L2P provides a level of transparency when forming adjacencies across an ISP network.

 

When protocol tunneling is enabled, edge switches on the inbound side of the service-provider network encapsulate Layer 2 protocol packets with a special MAC address and send them across the service-provider network. Core switches in the network do not process these packets but forward them as normal packets. Layer 2 protocol data units (PDUs) for CDP, STP, or VTP cross the service-provider network and are delivered to customer switches on the outbound side of the service-provider network. Identical packets are received by all customer ports on the same VLANs with these results:

 

-   Users on each of a customer's sites can properly run STP, and every VLAN can build a correct spanning tree based on parameters from all sites and not just from the local site.

-   CDP discovers and shows information about the other Cisco devices connected through the service-provider network.

-   VTP provides consistent VLAN configuration throughout the customer network, propagating to all switches through the service provider.

  

**L2P Tunneling Configuration**

**Configure Interface as an Access Port or an IEEE 802.1Q tunnel port**

Switch(config)# interface fastethernet0/1

Switch(config-if)# switchport mode { access | dot1q-tunnel }

**Enable L2P Tunneling**

Switch(config-if)# l2protocol-tunnel

-   Enables CDP, STP and VTP each can be specifically defined if you only wish to tunnel specific protocols

-   Disables CDP on the port

  

**Enabled point-to-point protocols (optional)**

Switch(config-if)# l2protocol-tunnel point-to-point

-   Expands tunnel to support point-to-point protocols such as PAgP, LACP and UDLD.

  

**Define Thresholds (optional)**

Switch(config-if)# l2protocol-tunnel shutdown-threshold *[cdp | stp | vtp]* 1500

-   Shutdown interface if the defined packets-per-second threshold is reached; range 1 to 4096.

Switch(config-if)# l2protocol-tunnel drop-threshold *[cdp | stp | vtp]* 1000

-   Drops packets if the defined packets-per-second threshold is reached; range of 1 to 4096

Switch(config-if)# l2protocol-tunnel drop-threshold point-to-point *[pagp | lacp | udld]* 1000

-   Drops packets if the defined packets-per-second threshold is reached; range of 1 to 4096

  

**NOTE:** If both thresholds are configured, the drop threshold must be less or equal the shutdown threshold.

 

**Define CoS value for tunneled traffic (optional)**

Switch(config-if)# l2protocol-tunnel cos *[0-7]*

 

**Troubleshooting/Verification**

Switch# show l2protocol-tunnel

-   Display the Layer 2 tunnel ports on the switch, including the protocols configured, the thresholds, and the counters.

 

Switch# show l2protocol-tunnel summary

-   Display only Layer 2 protocol summary information

  

Switch# clear l2protocol-tunnel counters

-   Clear the protocol counters on Layer 2 protocol tunneling ports.

 

Switch# errdisable recovery cause l2ptguard

-   Enable auto-recovery of L2P tunneled ports which shutdown due to the maximum threshold being reached.

  

Switch# show dot1q-tunnel

-   Display IEEE 802.1Q tunnel ports on the switch.

  

Switch# show vlan dot1q tag native

-   Display the status of native VLAN tagging on the switch.

 

 

 

VTP

Wednesday, January 05, 2011

9:01 AM

 

**Overview
**VLAN Trunking Protocol (VTP) is a way to propagate VLANs across your LAN, instead of manually configuring VLANs on each device.

**Terminology**

**Domain:** Common to all switches participating in VTP

**Server Mode:** Generates and propagates VTP advertisements to clients; default mode on unconfigured switches

**Client Mode:** Receives and forwards advertisements from servers; VLANs cannot be manually configured on switches in client mode.

**Transparent Mode:** Forwards advertisements but does not participate in VTP; VLANs must be configured manually

**Pruning:** VLANs not having any access ports on an end switch are removed from the trunk to reduce flooded traffic

**Revision numbers:** VTP works off revision numbers, if VLAN information is changed on a VTP Server this information will be replicated to other switches that are in Server or Client mode.
 

**VTP Configuration**

Switch(config)# vtp mode {server | client | transparent}

Switch(config)# vtp domain <name

Switch(config)# vtp password <password

-   Provides MD5 hash that includes VTP updates and will only propagate VTP information to those devices with the same password.

Switch(config)# vtp version {1 | 2}

Switch(config)# vtp pruning

Switch(config)# interface

-   Specifies the interface whose IP address is used to identify this switch in VTP updates.
  

**VTP Pruning**

Switch(config)# vtp pruning {enable | disabled }

-   This command can be entered on a VTP server to enable or disable VTP pruning and will be propagated across the LAN.

  

Switch(config)# clear vtp pruneeligible <vlan range

-   (Optional) Make specific VLANs pruning-ineligible on the device. (By default, VLANs 2-1000 are pruning-eligible.)

  

Switch(config)# set vtp pruneeligible <vlan range

-   (Optional) Make specific VLANs pruning-eligible on the device.

 

**Troubleshoot/Verification**

Switch# show vlan

-   This command can be used to show configured VLANs, note if a port is missing it's in trunk mode.

  

Switch# show interface trunk

-   This command is used to display information related to configured trunk ports and to verify the correct VLANs are being pruned.

  

Switch# show vtp status

-   Display the VTP switch configuration information, including VTP pruning.

  

Switch# show vtp password

-   The password is encrypted in MD5, this is the only way to view the VTP password.

  

Switch# show vtp counters

-   Display counters about VTP messages that have been sent and received.

 

VLANs

Wednesday, January 05, 2011

8:59 AM

 

**Overview
**VLANs are in place for a number of reasons including but not limited to logically separating broadcast domains, security and separating different traffic types (such as data and voice).

**Normal Usable Range:** 1-1001
**Extended Range:** 1006-4094

**VLAN Configuration**

Switch(config)#vlan 10

Switch(config-vlan)#name Data
 

Switch(config)#vlan 20

Switch(config-vlan)#name Voice
 

**Troubleshooting**

Switch# delete flash: vlan.dat

-   Clearing startup-config will not remove VLANs, it's stored in a protected file. You will need to issue the command above to remove VLAN configuration and then reload the switch.

**
**Switch# show vlan

-   This command can be used to show configured VLANs, note if a port is missing it's in trunk mode.

  

Switch# show interface trunk

-   This command is used to display information related to configured Trunk ports.

 

 

Layer 3 Routing

Friday, January 07, 2011

8:19 AM

 

**Overview
Layer 2** switching is hardware based, which means it uses the media access control address (MAC address) from the host's network interface cards (NICs) to decide where to forward frames. Switches use application-specific integrated circuits (ASICs) to build and maintain filter tables (also known as MAC address tables).

The only difference between a **Layer 3** switch and router is the way the administrator creates the physical implementation. Also, traditional routers use microprocessors to make forwarding decisions, and the switch performs only hardware-based packet switching. However, some traditional routers can have other hardware functions as well in some of the higher-end models. Layer 3 switches can be placed anywhere in the network because they handle high-performance LAN traffic and can cost-effectively replace routers. Layer 3 switching is all hardware-based packet forwarding, and all packet forwarding is handled by hardware ASICs. Layer 3 switches really are no different functionally than a traditional router and perform the same functions.

 

This is important when it comes to reducing the amount of hardware needed to perform the same function within your network. We could use the traditional method of intervlan routing using a router & switch (Router-on-a-Stick). But in the case of a layer 3 switching, we would only need one device to perform the same task and it would likely have better performance due to its hardware based packet switching.

 

Router-On-A-Stick

Friday, January 07, 2011

8:46 AM

 

**Overview**

Router-On-A-Stick refers to a common configuration to allow intervlan routing between VLANs.

**Router-on-a-Stick Configuration**

The following scenario describes two VLANs (10 - Accounting & 20 - Management) who require

Intervlan routing using a layer 2 switch and router.

**
Switch
Create VLANs; assign hosts to an individual VLAN**

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

**Create Trunk interface to Router**

Switch(config)# interface FastEthernet 0/1

Switch(config-if)# description Trunk-To-Router

Switch(config-if)# switchport trunk encapsulation dot1q

Switch(config-if)# switchport mode trunk

 

**NOTE:** Additional trunk options (DTP, Allowed VLANs etc.) has been removed from the configuration for brevity; view Switchport: Trunk configuration for details.

 

**Router
Create Trunk interface to Switch; Subinterfaces corresponding to each VLAN.**

Router(config)# interface fastethernet 0/0.10

Router(config-if)# encapsulation dot1q 10

Router(config-if)# ip address 10.10.10.1 255.255.255.0

 

Router(config)# interface fastethernet 0/0.20

Router(config-if)# encapsulation dot1q 20

Router(config-if)# ip address 20.20.20.1 255.255.255.0

**NOTE:** No IP configuration is done under the physical interface; only under subinterfaces.

 

**Troubleshooting/Verification**

Switch# show vlan

-   Displays VLANs and what interfaces are associated with each VLAN; trunk ports will not be displayed.

  

Switch# show interface trunk

-   Displays information related to configured trunk ports (mode, encapsulation, VLANs and ports)

  

Router# show ip route

-   Displays the traditional IP routing table

 

Routed Port

Friday, January 07, 2011

8:28 AM

 

**Overview**

A routed port is a physical port that acts like a port on a router; it does not have to be connected to a router. A routed port is not associated with a particular VLAN, as is an access port. A routed port behaves like a regular router interface, except that it does not support VLAN subinterfaces. Routed ports can be configured with a Layer 3 routing protocol. A routed port is a Layer 3 interface only and does not support Layer 2 protocols, such as DTP and STP.

**Enable IP Routing**

Switch(config)#ip routing

-   Enable IP routing functions on the Switch

**Routed Port Configuration**
Switch(config)# interface gigabitethernet0/2

Switch(config)# description RoutedPort

Switch(config-if)# no switchport

-   Enables Layer 3 functions for this individual switchport

Switch(config-if)# ip address 192.20.135.21 255.255.255.0

Switch(config-if)# no shutdown

**Troubleshooting/Verification**

Troubleshooting/verification is done the exact same as it would like any traditional layer 3 device, other commands are available - view IP Routing within this notebook.

Switch# show interface [ type slot/port ]

-   Displays statistics for network interfaces

  

Switch# show ip route

-   Displays the traditional IP routing table, if no output is displayed "ip routing" is not enabled.

 

SVIs

Friday, January 07, 2011

8:21 AM

 

**Overview
**By design VLANs are not able to communicate without a layer 3 device forwarding between VLANs. However modern Multilayer Switches (MLS) or Layer 3 switches can be configured to allow Inter-VLAN routing. VLAN interfaces or switched virtual interfaces (SVI) is a logical layer 3 routable interface.  Generally, if you want to accomplish Inter-VLAN routing with a layer 3 switch, you would use a VLAN interface.  From there, you would point the client devices to the VLAN interfaces to use as it's default gateway.  When a packet arrives on that interfaces that needs to be routed to a different VLAN, the L3 switch will do a routing table lookup.  If the destination VLAN has a corresponding SVI, it will see it as directly connected in the routing table and go through the routing process.
 

**Enable IP Routing**

Switch(config)# ip routing

-   Enable IP routing functions on the Switch

**SVI Configuration**

Create Switched Virtual Interfaces (SVIs) and designate the subnet defined for the specific VLAN.

 

**VLAN 10**

Switch(config)# interface vlan 10
Switch(config-if)# ip address 10.10.10.0 255.255.255.0

<Allows for intercommunication between VLANs

**VLAN 20**

Switch(config)# interface vlan 20
Switch(config-if)# ip address 10.10.20.0 255.255.255.0

 

**NOTE:** The default gateway for hosts will be the SVI network address, for example VLAN 10 clients would have a default gateway of 10.10.10.1 /24

 

**Applying SVI to Layer 2 Interface**

You can apply a SVIs to a layer 2 port and essentially make it a Layer 3 routed port. By doing so you could even have a routing protocol run over this Layer 2 port through the SVI. This is useful when you have a device that only has a single routed interface on a device such as an Cisco 881 and need to do routing on the other layer 2 ports.

 

**Create VLAN and SVI**

Router(config)# vlan 10

!

Router(config)# int vlan 10

Router(config-if)# ip address 10.10.10.1 255.255.255.252

 

**Apply to L2 interface**

Router(config)# interface FastEthernet0/0

Router(config-if)# switchport access vlan 10

-   Applies the address of 10.10.10.1/30 to FastEthernet0/0 and the other will assume L3 connectivity.

  

**Troubleshooting/Verification**
Switch# show interface vlan <number

-   Displays VLAN output as it were a physical interface, hardware should be defined as EtherSVI.

  

Switch# show ip route

-   Displays the traditional IP routing table, if no output is displayed "ip routing" is not enabled.

 

 

EtherChannel

Sunday, December 26, 2010

2:27 PM

 

**Overview**

EtherChannel is a port trunking (link aggregation) technology or port-channel architecture used primarily on Cisco switches. It allows grouping several physical Ethernet links to create one logical Ethernet link for the purpose of providing fault-tolerance and high-speed links between switches, routers and servers. An EtherChannel can be created from between two and eight active Fast Ethernet, Gigabit Ethernet or 10-Gigabit Ethernet ports, with an additional one to eight inactive (failover) ports which become active as the other active ports fail. EtherChannel is primarily used in the backbone network, but can also be used to connect end user machines.

**L2 EtherChannel Configuration**
**Create channel group on Switch1**

Switch1(config)#interface range FastEthernet fa0/1 - 2
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#channel-group 1 mode on

**Create channel group on Switch1
**Switch2(config)#interface range FastEthernet fa0/1 - 2

Switch2(config-if)#switchport mode trunk
Switch2(config-if)#channel-group 1 mode on

 

**NOTE:** This statically creates a channel group between the two switches without using a negotiation protocol.
**
L2 EtherChannel Configuration; using a negotiation protocol**

**Create channel group on Switch1**

Switch1(config)#interface range FastEthernet fa0/1 - 2
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#channel-protocol { lacp | pagp }
Switch1(config-if)#channel-group 2 mode [active/passive or auto/desirable]

**Create channel group on Switch1
**Switch2(config)#interface range FastEthernet fa0/1 - 2
Switch2(config-if)#switchport mode trunk

Switch2(config-if)#channel-protocol { lacp | pagp }
Switch2(config-if)#channel-group 2 mode [active/passive or auto/desirable]

**Negotiation Key:**

**LACP**

Active: Enable LACP unconditionally

Passive: Enable LACP only if a LACP device is detected

 

**PAgP**

Desirable: Enable PAgP unconditionally

Auto: Enable PAgP only if a PAgP device is detected

**NOTE:** If you're configuring EtherChannel remotely, use passive or desirable for negotiation. Because if you turn "on" EtherChannel and the other side is not configured, you will lose your connection.

**Troubleshooting/Verification:**

Switch#show etherchannel [number] summary

-   Displays a one-line summary per channel group.

 

Switch# show etherchannel port

-   Verify port operation within an individual EtherChannel.

  

Switch#show spanning-tree interface port-channel [number]

-   Displays spanning tree information for the specified interface.

 

L3 EtherChannel

Sunday, December 26, 2010

3:52 PM

 

**Overview:**

EtherChannel can be configured to have all of the same benefits of Layer 2, but now layer 3 capable.
 

**L3 EtherChannel Configuration; using a negotiation protocol
Create port-channel interface
**Switch1(config)#interface port-channel 3
Switch1(config-if)#ip address 172.20.1.10 255.255.255.0
Switch1(config-if)#switchport mode trunk

**Create channel-group using the same port-channel as above, identical to L2 configuration.**

Switch1(config)#interface range FastEthernet fa0/1 - 2

Switch1(config-if)#switchport mode trunk
Switch1(config-if)#channel-protocol pagp
Switch1(config-if)#channel-group 3 mode desirable

Switch2(config)#interface range FastEthernet fa0/1 - 2

Switch2(config-if)#switchport mode trunk

Switch2(config-if)#channel-protocol pagp
Switch2(config-if)#channel-group 3 mode desirable

 

**Troubleshooting/Verification:**

Switch#show etherchannel [number] summary

-   Displays a one-line summary per channel group.

  

Switch# show etherchannel port

-   Verify port operation within an individual EtherChannel.

  

Switch# show etherchannel [channel-id] port-channel

-   Displays port channel (L3) information (IOS Router)

  

Switch# show interfaces port-channel [channel-id]

-   Displays port channel (L3) information (IOS Router)

  

Switch#show spanning-tree interface port-channel [number]

-   Displays spanning tree information for the specified interface.

 

 

Load-Balancing

Sunday, December 26, 2010

3:52 PM

 

**Overview
**Instead of having all traffic traverse the first interface in the channel-group until fully utilized, load-balancing can be used in a variety of different ways to share bandwidth equally across all physical links.

**EtherChannel Load Balancing Configuration**
Switch(config)#port-channel load-balance { src-mac | dst-mac | src-dst-mac | src-ip | dst-ip | src-dst-ip | src-port | dst-port | src-dst-port }

 

**The load-balancing keywords indicate the following**

**With a PFC2:**

**src-port**---Source Layer 4 port

**dst-port**---Destination Layer 4 port

**src-dst-port**---Source and destination Layer 4 port

 

**With a PFC or PFC2:**

**src-ip**---Source IP addresses

**dst-ip**---Destination IP addresses

**src-dst-ip**---Source and destination IP addresses

**src-mac**---Source MAC addresses

**dst-mac**---Destination MAC addresses

**src-dst-mac**---Source and destination MAC addresses

 

**NOTE:** A wide variety of load distribution options are available, however they are dependent on which switch platform in use.

Switch(config)#no port-channel load-balance

-   Disables EtherChannel Load Balancing

**Troubleshooting/Verfication
**Switch#show etherchannel load-balance

-   Displays the load-balance or frame-distribution scheme among ports in the port channel.

 

 

Spanning-Tree Protocol

Friday, January 07, 2011

10:41 AM

 

**Overview**

Spanning Tree Protocol (STP) is a network protocol that ensures a loop-free topology for any bridged Ethernet local area network. The basic function of STP is to prevent bridge loops. Spanning tree also allows a network design to include spare (redundant) links to provide automatic backup paths if an active link fails, without the danger of bridge loops, or the need for manual enabling/disabling of these backup links.

**
STP Specifications**

**IEEE 802.1D-1998** - Deprecated legacy STP standard

**IEEE 802.1w** - Introduced RSTP

**IEEE 802.1D-2004** - Replaced legacy STP with RSTP

**IEEE 802.1s** - Introduced MST

**IEEE 802.1Q-2003** - Added MST to 802.1Q

**IEEE 802.1Q-2005** - Most recent 802.1Q revision

**PVST** - Per-VLAN implementation of legacy STP

**PVST+** - Added 802.1Q trunking to PVST

**RPVST+** - Per-VLAN implementation of RSTP

 

**Spanning Tree Operation**

1.  **Determine root bridge:** The bridge advertising the lowest bridge ID becomes the root bridge

2.  **Select root port:** Each bridge selects its primary port facing the root

3.  **Select designated ports:** One designated port is selected per segment

4.  **Block ports with loops:** All non-root and non-designated ports are blocked

(**Legacy) Spanning-Tree Port States**

**Disabled**

-   This is the state when the switch port is disabled. A switch port may be disabled due to administrative reasons or due to switch specific problems.

**Blocking**

-   This is the initial state. All ports are put in a blocked state to prevent bridging loops. 

**Listen**

-   This is the second state of switch ports. Here all the ports are put in listen mode. The port can listen to frames but can't send. The period of time that a switch takes to listen is set by "forward delay"

**Learning**

-   Learn state comes after Listen state. The only difference is that the port can add information that it has learned to its address table. The period of time that a switch takes to learn is set by "forward delay". 

**Forwarding**

-   A port can send and receive data in this state. Before placing a port in forwarding state, Spanning-Tree Protocol ensures that there are no redundant paths or loops.

 

**(Legacy) Spanning-Tree Port Roles
Root Port** (Forwarding)

-   A forwarding port that is the best port from a nonroot-bridge to the root bridge.

**Designated Port** (Forwarding)

-   A forwarding port for every LAN segment, point away from the root bridge.

**Blocking Port** (Blocking)

-   A port that would cause a switching loop, no user data is sent or received but it may go into forwarding mode if the other links in use were to fail and the spanning tree algorithm determines the port may transition to the forwarding state. BPDU data is still received in blocking state.

 

**Bridge ID format**

**Priority**

-   4-bit bridge priority (configurable from 0 to 65,535 in increments of 4096) *Default: 32,768*

**System ID Extension**

-   12-bit value taken from VLAN number (IEEE 802.1t)

**MAC Address**

-   48-bit unique identifier; by default switch with the lowest MAC address will become the root bridge.

 

**Root Election
**Only a single switch can be the root of the spanning tree; to select the root, the switches hold an election based on the Bridge ID (BID) based on Priority + MAC Address.

 

**Election Process**

1.  Each switch begins its STP logic by creating and sending STP Hello BPDUs claiming to be the root bridge.

2.  If a switch hears a superior Hello - a Hello with a lower bridge ID, it will stop claiming to be root by ceasing to originate and send Hellos.

3.  Switches that denounce themselves as not being the root, forward the superior Hellos received by the superior switch.

4.  Eventually al switches except the switch with the best (lowest) bridge ID cease to originate Hellos; that one switch wins the election and becomes the root switch.

  

**NOTE:** Root election can be adjusted by changing priority, view RSTP configuration.

  

**Port Costs**

**Bandwidth / Cost**

4 Mbps - **250**

10 Mbps - **100**

16 Mbps - **62**

45 Mbps - **39**

100 Mbps - **19**

155 Mbps - **14**

622 Mbps - **6**

1 Gbps - **4**

10 Gbps - **2**

20+ Gbps - **1**

**
Path Selection**

If a switch receives multiple Hellos with equal calculated cost, it uses the following tiebreakers:

  

1.  Neighbor with the lowest root ID becomes the root port

2.  Prefer the neighbor with the lowest cost to the root

3.  Prefer the neighbor with the lowest bridge ID

4.  Prefer the lowest sender port number

  

**NOTE:** Port priority and cost can be adjusted, view RSTP configuration.

**Disable Spanning-Tree
**In the unlikely case you need to disable spanning-tree, use the following command.

Switch(config)# no spanning-tree vlan { vlan id | 1-4096 }

-   Disables STP on a per VLAN basis or all VLANs (1-4096)

  

**NOTE:** You must disable STP within your entire LAN or you may have undesired network results.

**Default Timers
Hello** - 2s
**Forward Delay** - 15s
**Max Age** - 20s

 

Advanced Features

Friday, January 07, 2011

10:58 AM

 

**Overview**

These are all of the advanced features and optimizations available for the Spanning-Tree Protocol, view subpages for additional features.

 

**Portfast**

**Requirements:** Used on access ports that are not connected to other switches or hubs

**What it does**: Immediately puts the port into forwarding state once the port is physically working.

**Enables PortFast on an access port interface**

Switch(config-if)# spanning-tree portfast

 

**UplinkFast
Requirements:** Used on access layer switches that have multiple uplinks to distribution/core switches.

**What it does:** Immediately replaces a lost RP (Root Port) with an alternate RP, immediately forwards on the RP, and triggers updates of all switches' CAMs.

**Enable UplinkFast on a uplink interface to distribution level switches**
Switch(config-if)# spanning-tree uplinkfast

 

**BackboneFast**

**Requirements:** Used to detect indirect link failures, typically in the network core.
**What it does:** Avoids waiting for Maxage to expire when its RP ceases to receive Hellos; does so by querying the switch attached to its RP.
**
Enable BackboneFast on a Switch**

Switch(config)# spanning-tree backbonefast

 

**Troubleshooting/Verification**

Switch# show spanning-tree summary

-   Verifies PortFast, UplinkFast and BackboneFast status (Enabled or Disabled)

 

Root Guard

Friday, January 07, 2011

7:31 PM

 

**Root Guard**

This command will disable any port that a superior BPDU is received on. This is done to ensure a switch will remain root at all times. Root guard enabled on an interface applies to all the VLANs to which the interface belongs.

* *

**Enable Root Guard Per Interface**

Switch(config-if)# spanning-tree guard root

 

**NOTE:** Do not enable the root guard on interfaces to be used by the UplinkFast feature. With UplinkFast, the backup interfaces (in the blocked state) replace the root port in the case of a failure. However, if root guard is also enabled, all the backup interfaces used by the UplinkFast feature are placed in the root-inconsistent state (blocked) and are prevented from reaching the forwarding state.

 

**Troubleshooting/Verification**

Switch# show running-config

-   Verifies Root Guard is enabled on interfaces

 

BPDU Guard

Friday, January 07, 2011

7:32 PM

 

**BPDU Guard**

BPDU guard enhancement allows network designers to enforce the STP domain borders and keep the active topology predictable. This prevents a rogue switch with a lower BID from becoming the root switch. The devices behind the ports that have STP PortFast enabled are not able to influence the STP topology. At the reception of BPDUs, the BPDU PortFast operation disables the port that has BPDU Guard configured. The BPDU guard transitions the port into errdisable state, and a message appears on the console.

 

**Enable BPDU Guard Globally**

Switch(config)# spanning-tree bpduguard default

 

**Enable BPDU Guard Per Interface**

Switch(config-if)# spanning-tree bpduguard { enable | disable }

 

**NOTE:** PortFast and BPDU Guard work hand in hand, enable Portfast as well as BPDU Guard on the interface; view "Advanced Features" in order to enable PortFast.

 

**Troubleshooting/Verification**

Switch# show interfaces [ slot/port ] status

-   The port status of err-disabled is displayed.

  

Switch# show spanning-tree summary totals

-   Verifies whether BPDU Guard is enabled or disabled

 

Switch# show errdisable detect

-   Displays reason for the errdisable status on all ports

  

Switch(config)# errdisable recovery cause bpduguard

-   Allows port to reenable itself if the cause of the error is BPDU Guard by setting a recovery timer.

  

Switch(config)# errdisable recovery intervals 400

-   Sets recovery timer to 400 seconds. Default is 300 seconds. Range is from 30 to 86,400 seconds.

  

Switch# show errdisable recovery

-   Shows he time period after which the interfaces are enabled for errdisable conditions.

 

**NOTE:** In the case of manual recovery, simply shutdown/no shutdown an interface to bring it back online.

 

BPDU Filter

Friday, January 07, 2011

7:30 PM

 

**BPDU Filter**

BPDU filter is a feature used to filter sending or receiving BPDUs on a switchport. BPDUs are the messages exchanged between switches to calculate the spanning tree topology.

 

It is extremely useful on those ports which are configured as PortFast ports as there is no need to send or receive any BPDU messages on of these ports.

 

**Enable BPDU Filter Globally**

Switch(config)# spanning-tree bpdufilter default

 

**Enable BPDU Filter Per Interface**

Switch(config-if)# spanning-tree bpdufilter { enable | disable }

 

**NOTE:** PortFast and BPDU Filter work hand in hand, enable Portfast as well as BPDU Filter on the interface; view "Advanced Features" in order to enable PortFast.

**NOTE:** Enabling BPDU filter on an interface is equivalent to disabling STP, make sure the interface is only for access level devices and not connected to another switch. Best practice would be to simply enable it globally and only PortFast interfaces would have the filter enabled.

 

**Troubleshooting/Verification**

Switch# show running-config

-   Verifies BPDU Filtering is enabled on interfaces

 

Loop Guard

Friday, January 07, 2011

7:31 PM

 

**Loopguard**

The loop guard feature checks that a root port or an alternate root port is receiving BPDUs. If a port is not receiving BPDUs, the loop guard feature puts the port into an inconsistent state, isolating the failure and letting spanning tree converge to a stable topology until the port starts receiving BPDUs again. This feature works only on point-to-point STP interfaces.

*
**Enable Loopguard Globally***

Switch(config)# spanning-tree loopguard default

**Enable Loopguard Per Interface**

Switch(config-if)# spanning-tree guard loop

 

**Troubleshooting/Verification**

Switch# show spanning-tree active

-   Displays which ports are alternate or root ports

  

Switch# show spanning-tree mst

-   Displays which ports are alternate or root ports for MSTP.

 

UDLD

Friday, January 07, 2011

7:31 PM

 

**UDLD - UniDirectional Link Detection**

The Cisco-proprietary UDLD protocol allows devices connected through fiber-optic or copper (for example, Category 5 cabling) Ethernet cables connected to LAN ports to monitor the physical configuration of the cables and detect when a unidirectional link exists. When a unidirectional link is detected, UDLD shuts down the affected LAN port and alerts the user. Unidirectional links can cause a variety of problems, including spanning tree topology loops.

 

**Aggressive** mode is disabled by default. Configure UDLD aggressive mode only on point-to-point links between network devices that support UDLD aggressive mode. With UDLD aggressive mode enabled, when a port on a bidirectional link that has a UDLD neighbor relationship established stops receiving UDLD packets, UDLD tries to reestablish the connection with the neighbor. After eight failed retries, the port is disabled.

 

To prevent spanning tree loops, nonaggressive UDLD with the default interval of 15 seconds is fast enough to shut down a unidirectional link before a blocking port transitions to the forwarding state (with default spanning tree parameters).

 

**Enable UDLD Globally**

Switch(config)# udld { enable | aggressive }

Switch(config)# no udld enable

-   Disables UDLD

 

**Enable UDLD Per Interface**

Switch(config-if)# udld port [aggressive]

-   Enable the aggressive keyword to enable aggressive mode.

 

**Troubleshooting/Verification**

Switch# show udld

-   Displays UDLD information

  

Switch# show udld neighbors

-   Displays LAN ports in an error-disabled state

 

Switch# udld reset

-   Resets all LAN ports that have been shut down by UDLD.

 

MSTP

Friday, January 07, 2011

10:58 AM

 

**Overview**

Multiple Spanning-Tree Protocol (MST) uses RSTP for rapid convergence. MST enables VLANs to be grouped into a spanning-tree instance. Each instance has a spanning-tree topology that is independent of the other spanning-tree instances. This architecture provides multiple forwarding paths for data traffic and enables load balancing. This also reduces the number of spanning-tree instances that are required to support a large number of VLANs. MST regions are used to partition the network. All switches in the same region must have the same VLAN-to-instance mapping, the same configuration revision number, and the same name. MST groups a few VLANs into one spanning-tree instance unlike PVST, which has a spanning-tree instance for every VLAN. This reduces the number spanning-tree processes required and enhances the switch performance. MST support 16 instances, numbered 1 through 15.

 

![](''/media/image3.png){width="5.96875in" height="2.1458333333333335in"}

 

*Image provided by [packetlife.net](http://packetlife.net)*

 

**MSTP Configuration
Specify STP Mode**

Switch(config)# spanning-tree mode mst

**MST Configuration; instance 1**

Switch(config)# spanning-tree mst configuration

Switch(config-mst)# show current

-   Current MST configuration

Switch(config-mst)# name Region1

Switch(config-mst)# revision 1

Switch(config-mst)# instance 1 vlan 1,3,5

-   Maps VLANs 1,3,5 to MST instance 1

Switch(config-mst)# show pending

-   Show pending configuration before exiting MST configuration and therefore saving.

  

**NOTE:** To exit without making configuration changes, use ***abort***

 

**MST Configuration; instance 2**

Switch(config)# spanning-tree mst configuration

Switch(config-mst)# name Region2

Switch(config-mst)# revision 1

Switch(config-mst)# instance 2 vlan 2,4,6

-   Maps VLANs 1,3,5 to MST instance 2

 

**Bridge priority (per instance)**

Switch1(config)# spanning-tree mst 1 priority 0

Switch1(config)# spanning-tree mst 2 priority 4096

-   Defines the top left distribution switch to become the root for VLANs 1,3,5

  

Switch2(config)# spanning-tree mst 1 priority 4096

Switch2(config)# spanning-tree mst 2 priority 0

-   Defines the top right distribution switch to become the root for VLANs 2,4,6

  

**NOTE:** Increments of 4096, lower is a better priority (range 0 to 65,535)

 

**Adjust timers**

Switch(config)# spanning-tree mst hello-time 2

Switch(config)# spanning-tree mst forward-time 15

Switch(config)# spanning-tree mst max-age 20

 

**Specify Maximum hops for BPDUs, before they will be discarded**

Switch(config)# spanning-tree mst max-hops 20

 

**Interface attributes**

Switch(config)# interface FastEthernet0/1

Switch(config-if)# spanning-tree mst 1 port-priority 128

Switch(config-if)# spanning-tree mst 1 cost 19

**NOTE:** For MSTP to go in effect, the exact same configuration needs to go in place on the neighboring switches (name, revision, instances and mapped vlans)

 

**Troubleshooting/Verification (Basic)**

Switch# show spanning-tree

-   Displays information related to Spanning-Tree

-   Root information: Priority, MAC, Timers, Cost and Port to reach the Root)

-   Port Information: Role, Status, Cost, Priority and Type

  

Switch# show spanning-tree active

-   Displays STP information on active interfaces only

  

Switch# show spanning-tree brief

-   Displays a brief status of the STP

  

Switch# show spanning-tree detail

-   Displays a detailed summary of interface information

  

Switch# show spanning-tree vlan [1-4094]

-   Displays STP information for VLAN 5

  

Switch# show spanning-tree interface [slot/port]

-   Displays information related to spanning-tree enable interface (VLAN, Port State, BPDUs sent/received)

 

Switch# show spanning-tree summary [totals]

-   Displays a summary of port states, adding the keyword totals will display total lines of the STP section

  

**Troubleshooting/Verification (Advanced)**

Switch# debug spanning-tree all

-   Displays all spanning-tree debugging events

  

Switch# debug spanning-tree events

-   Displays spanning tree debugging topology events

  

Switch# debug spanning-tree backbonefast

-   Displays spanning-tree debugging BackboneFast events

  

Switch# debug spanning-tree uplinkfast

-   Displays spanning-tree debugging UplinkFast events

  

Switch# debug spanning-tree mstp all

-   Displays all MSTP debugging events

  

Switch# debug spanning-tree switch state

-   Displays spanning-tree port state changes

  

Switch# debug spanning-tree pvst+

-   Displays PVST+ events

 

**Troubleshooting/Verification (MSTP)**

 

Switch# show spanning-tree mst configuration

-   Displays information related to MST configuration, instances and VLANs which are mapped to each instance.

 

Switch# show spanning-tree mst <instance number

-   Displays MSTP information for instance an individual instance.

  

Switch# show spanning-tree mst interface [type slot/port]

-   Displays MSTP information for an individual interface.

  

Switch# show spanning-tree mst <instance number interface [type slot/port]

-   Displays MSTP information for an individual instance for a specific interface.

  

Switch# show spanning-tree mst <instance number detail

-   Shows detailed information related to an individual instance.

 

PVST+ / RSTP+

Friday, January 07, 2011

10:59 AM

 

**Overview**

Rapid Spanning Tree Protocol (RSTP) is an evolution of the Spanning Tree Protocol (802.1D standard) and provides for faster spanning tree convergence after a topology change. The standard also includes features equivalent to Cisco PortFast, UplinkFast and BackboneFast for faster network convergence.

PVST+ is the exact same as PVST except using IEEE standard based 802.1q trunking.
RSTP+ is the exact same as PVST+ in addition to it includes Cisco proprietary features Portfast, UplinkFast and BackboneFast.
**
Rapid Spanning-Tree Port States**

**Discarding**

-   Replaces Disabled, Blocking, Listening in legacy STP. Essentially instead of blocking the port it simply discards frames until its needed and can instantly be put into the forwarding state for rapid convergence.

**Listen**

-   This is the second state of switch ports. Here all the ports are put in listen mode. The port can listen to frames but can't send. The period of time that a switch takes to listen is set by "forward delay"

**Learning**

-   Learn state comes after Listen state. The only difference is that the port can add information that it has learned to its address table. The period of time that a switch takes to learn is set by "forward delay". 

**Rapid Spanning-Tree Port Roles**

**Root** (Forwarding)

-   A forwarding port that is the best port from a nonroot-bridge to the root bridge.

**Designated** (Forwarding)

-   A forwarding port for every LAN segment, faces away from the root bridge.

**Alternate** (Discarding)

-   An alternate path to the root bridge. This path is different than using the root port.

**Backup** (Discarding)

-   A backup/redundant path to a segment where another bridge port already connects.

 

**NOTE:** Compared to legacy STP 802.1d, blocking is now replaced with alternate and backup port roles which helps speed up convergence and port initialization.

 

**Link Types**

**Point-to-Point**

-   Connects to exactly one other bridge (full duplex)

**Shared**

-   Potentially connects to multiple bridges (half duplex)

**Edge**

-   Connects to a single host; designated by PortFast

**NOTE:** RSTP can only achieve rapid transition to the forwarding state on edge ports and on point-to-point links and is determined by the duplex.

 

**PVST+ and RPSTP+ Configuration**

**Specify STP Mode**

spanning-tree mode { pvst | rapid-pvst }

 

**Adjusting Root Election
**Adjust bridge priority to manually decide which switch will become the root bridge.

 

Switch(config)# spanning-tree vlan [1-4094]

Switch(config)# spanning-tree vlan [1-4094] root { primary | secondary }

-   If primary; sets bridge to root with priority: 24,626

-   If secondary; sets bridge to a worse priority in the case the primary fails it will become the root with a priority of 28,672.

Switch(config)# spanning-tree vlan [1-4094] root primary diameter [2-7]

-   Sets the maximum nmber of switches between any two end stations, automatically sets timers based on the diameter.

Switch(config) # spanning-tree vlan [1-4094] root primary hello-time [1-10]

-   Specifies the duration between the generation of configuration messages by the root switch.

 

**NOTE:** A vital design and configuration element involved with PVST is allowing for load balancing of VLAN operations. For example switchA could potentially be a root for VLAN 10 & 20 and switchB could be the root for VLAN 30 & 40; view MSTP.

 

**Adjust timers**

Switch(config)# spanning-tree vlan [1-4094] hello-time 2

Switch(config)# spanning-tree vlan [1-4094] forward-time 15

Switch(config)# spanning-tree vlan [1-4094] max-age 20

 

**Enable PVST+ Enhancements; view Advanced Features**

Switch(config)# spanning-tree backbonefast

Switch(config)# spanning-tree uplinkfast

 

**Interface Attributes**

There may be an instance when you need to force a port to be utilized by STP. In that case you can manually configure the port priority or cost. When a loop occurs STP uses path cost to determine which interface to place into the forwarding state
 

Switch(config)# interface FastEthernet0/1

Switch(config-if)# spanning-tree vlan [1-4094] port-priority 128

-   Range between 0-255, lower is better

Switch(config-if)# spanning-tree vlan [1-4094] cost 19

-   Range between 1-200,000,000, lower is better

 

**NOTE:** If no VLAN is defined, the port cost will be changed globally for the interface and effect all VLANs within STP.
 

**Link Type Specifications; view Advanced Features**

Switch(config-if)# spanning-tree link-type { point-to-point | shared }
 

**Enable PortFast, view Advanced Features**

Switch(config-if)# spanning-tree portfast

 

**STP Protection, view Advanced Features**

Switch(config)# spanning-tree guard { loop | root | none }

 

**STP Protection; per interface toggling**

Switch(config-if)# spanning-tree bpduguard enable

Switch(config-if)# spanning-tree bpdufilter enable

 

**Troubleshooting/Verification (Basic)**

Switch# show spanning-tree

-   Displays information related to Spanning-Tree

-   Root information: Priority, MAC, Timers, Cost and Port to reach the Root)

-   Port Information: Role, Status, Cost, Priority and Type

  

Switch# show spanning-tree active

-   Displays STP information on active interfaces only

  

Switch# show spanning-tree brief

-   Displays a brief status of the STP

  

Switch# show spanning-tree detail

-   Displays a detailed summary of interface information

  

Switch# show spanning-tree vlan [1-4094]

-   Displays STP information for VLAN 5

  

Switch# show spanning-tree interface [slot/port]

-   Displays information related to spanning-tree enable interface (VLAN, Port State, BPDUs sent/received)

 

Switch# show spanning-tree summary [totals]

-   Displays a summary of port states, adding the keyword totals will display total lines of the STP section

  

Switch(config)# spanning-tree extend system-id

-   Enables Extended System ID, also know as MAC Address Reduction. This likely will not be needed to be turned on as most modern IOS versions after 12.1(8) support this feature and is enabled by default.

  

Switch(config)# clear spanning-tree detected-protocols

-   If switching to a new STP mode, this command restarts protocol migration process on the switch if any port is connected to a port on a legacy 802.1d switch.

  

**Troubleshooting/Verification (Advanced)**

Switch# debug spanning-tree all

-   Displays all spanning-tree debugging events

  

Switch# debug spanning-tree events

-   Displays spanning tree debugging topology events

  

Switch# debug spanning-tree backbonefast

-   Displays spanning-tree debugging BackboneFast events

  

Switch# debug spanning-tree uplinkfast

-   Displays spanning-tree debugging UplinkFast events

  

Switch# debug spanning-tree mstp all

-   Displays all MSTP debugging events

  

Switch# debug spanning-tree switch state

-   Displays spanning-tree port state changes

  

Switch# debug spanning-tree pvst+

-   Displays PVST+ events

 

 

Advanced Catalyst Features

Tuesday, January 04, 2011

8:51 AM

 

**Overview**

Cisco Catalyst 3560 switch ships with either of these software images installed:

 

-   IP base image (formerly known as the standard multilayer image [SMI]), which provides Layer 2+ features (enterprise-class intelligent services). These features include access control lists (ACLs), quality of service (QoS), static routing, EIGRP stub routing, PIM stub routing, the Hot Standby Router Protocol (HSRP), and the Routing Information Protocol (RIP). Switches with the IP base image installed can be upgraded to IP services image (formerly known as the enhanced multilayer image [EMI].)

 

-   IP services image, which provides a richer set of enterprise-class intelligent services. It includes all IP base image features plus full Layer 3 routing (IP unicast routing, IP multicast routing, and fallback bridging). To distinguish it from the Layer 2+ static routing and RIP, the IP services image includes protocols such as the Enhanced Interior Gateway Routing Protocol (EIGRP) and the Open Shortest Path First (OSPF) Protocol.

  

The following subpages describe uncategorized advanced features supported by the Catalyst series; specifically the Catalyst 3560.

 

Flex Links

Friday, January 07, 2011

7:44 PM

 

**Overview**

A layer 2 switch feature with which redundancy and load balancing can be accomplished, as an alternative to Spanning-Tree Protocol (STP). The basic concept of Flex Links is to have a pair of Layer 2 interfaces, such as a switchport or port channel, where one is configured a primary and the other is a backup. If the primary goes down the backup link takes over traffic forwarding.

 

Flex links are easy to configure, highly efficient and more controllable feature but they require manual effort which is not the case with Spanning Tree Protocol (STP). For larger networks with ever-changing link layer network STP will be the only option. Flex links are more suitable and an opt choice for service provider and core enterprise networks.
**
Flex Link Examples**

**Scenario 1:** A Downlink Switch with one primary Uplink and a backup Uplink switches.

**Scenario 2:** Back to Back connected Switches with redundant connections.

 

**Flex Link Configuration**

Switch(config) interface FastEthernet0/1

Switch(config-if) switchport backup interface FastEthernet0/2

-   Specifies FE0/1 as the primary interface and FE0/2 as the backup interface. In the event FE0/1 link fails, FE0/2 will take over.

  

**NOTE:** Only a single backup link can be configured for any active link and STP is disabled for Flex Link ports.

 

**Load Balancing**

Load balancing in Flex links work at VLAN level. Both the ports in the Flex link pair can be made to forward traffic simultaneously. One port in the flex links pair can be configured to forward traffic belonging to VLANs 1-50 and the other can forward traffic for VLANs 51-100. Mutually exclusive VLANs are load sharing the traffic between the Flex link pairs. If one of the ports fails, the other active link forwards all the traffic.

 

**Load Balancing Configuration**

Switch(config) interface FastEthernet0/1

Switch(config-if) switchport backup interface FastEthernet0/2 prefer 20, 40, 60

-   The switch is configured with VLANs 10, 20, 30, 40, 60 and 70. By specifying the backup interfaces VLANs the other VLANs will be used by the primary link. In this example 10, 30 and 70 would be sent/received on the primary interface; while 20, 40 and 60 would be sent/received on the backup interface.

**Preemption**

Preemptions allows for if the primary interface goes offline, the backup takes over, if the primary were to come back online it would take over operation; unless in bandwidth mode.
 

**Forced**---the active interface as always preempts the backup.

**Bandwidth**---the interface with the higher bandwidth always acts as the active interface.

**Off**---no preemption happens from active to backup.

**Preemption Configuration**

Switch(config) interface FastEthernet0/1

Switch(config-if) switchport backup interface FastEthernet0/2 preemption mode { forced | bandwidth | off }

-   Enables preemption for Flex Links, view configurable options above.

Switch(config-if# switchport backup interface FastEthernet0/2 preemption delay 50

-   Configure the time delay until a port preempts another port.

  

**MAC Address-Table Move Update**

The MAC address-table move update feature allows the switch to provide rapid bidirectional convergence when a primary (forwarding) link goes down and the standby link begins forwarding traffic.

 

The basic concept is that the all MAC addresses that were mapped to the original interface are sent out in a update to all other configured switches. This allows for no dropped frames between devices during the backup interface transition.

**MAC Address-Table Move Update Configuration**

**Transmitting MMU Switch**
Switch(config)# interface FastEthernet0/1

Switch(config-if)# switchport backup interface FastEthernet0/2 mmu primary vlan 10

-   Specifies VLAN that the MMU update will be sent on.

  

Switch(config)# mac address-table move update transmit

-   Enables transmission of the MMU update if the primary interface goes down.

  

**Receiving MMU Switch**

Switch(config)# mac address-table move update receive

-   Enables the switch to receive MAC Address-Table Move Update

  

**Troubleshooting/Verification**
Switch# show interface switchport backup

-   Displays active and associated backup interfaces as well as there current state of operation and which VLANs are preferred on each interface.

 

Switch# show interface switchport backup [detail]

-   Displays additional information such as the preemption options and interface bandwidth.

  

Switch# show mac address-table move update

-   Displays the MAC address-table move update information on the switch.

 

StackWise

Thursday, June 30, 2011

8:56 PM

 

**Overview**

Cisco StackWise technology provides an innovative new method for collectively utilizing the capabilities of a stack of switches. Individual switches intelligently join to create a single switching unit with a 32-Gbps switching stack interconnect. Configuration and routing information is shared by every switch in the stack, creating a single switching unit. Switches can be added to and deleted from a working stack without affecting performance.

The switches are united into a single logical unit using special stack interconnect cables that create a bidirectional closed-loop path. This bidirectional path acts as a switch fabric for all the connected switches. Network topology and routing information is updated continuously through the stack interconnect. All stack members have full access to the stack interconnect bandwidth. The stack is managed as a single unit by a master switch, which is elected from one of the stack member switches.

Each switch in the stack has the capability to behave as a master or subordinate (member) in the hierarchy. The master switch is elected and serves as the control center for the stack. Both the master member switches act as forwarding processors. Each switch is assigned a number. Up to nine separate switches can be joined together. The stack can have switches added and removed without affecting stack performance.

Each stack of Cisco Catalyst Series Switches has a single IP address and is managed as a single object. This single IP management applies to activities such as fault detection, virtual LAN (VLAN) creation and modification, security, and QoS controls. Each stack has only one configuration file, which is distributed to each member in the stack. This allows each switch in the stack to share the same network topology, MAC address, and routing information. In addition, it allows for any member to become the master, if the master ever fails.

**Physical Sequential Linkage**

The switches are physically connected sequentially, as shown in Figure 3. A break in any one of the cables will result in the stack bandwidth being reduced to half of its full capacity. Subsecond timing mechanisms detect traffic problems and immediately institute failover. This mechanism restores dual path flow when the timing mechanisms detect renewed activity on the cable.

 

**NOTE:** Connect stack port 2 - stack port 1 of each descending switch from the master. The last switch in the stack reconnects with the master switch in the same fashion (stack port2 to stack port 1).

 

![](''/media/image4.png){width="3.1354166666666665in" height="1.71875in"}

 

**Master Switch Election**

The stack behaves as a single switching unit that is managed by a master switch elected from one of the member switches. The master switch automatically creates and updates all the switching and optional routing tables. Any member of the stack can become the master switch. Upon installation, or reboot of the entire stack, an election process occurs among the switches in the stack. There is a hierarchy of selection criteria for the election.

1.  **User priority** - The network manager can select a switch to be master.

2.  **Hardware and software priority** - This will default to the unit with the most extensive feature set. The Cisco Catalyst 3750 IP Services (IPS) image has the highest priority, followed by Cisco Catalyst 3750 switches with IP Base Software Image (IPB).

3.  **Default configuration** - If a switch has preexisting configuration information, it will take precedence over switches that have not been configured.

4.  **Uptime** - The switch that has been running the longest is selected.

5.  **MAC address** - Each switch reports its MAC address to all its neighbors for comparison. The switch with the lowest MAC address is selected.

 

**Master Switch Activities**

The master switch acts as the primary point of contact for IP functions such as Telnet sessions, pings, command-line interface (CLI), and routing information exchange. The master is responsible for downloading forwarding tables to each of the subordinate switches. Multicast and unicast routing tasks are implemented from the master. QoS and access control list (ACL) configuration information is distributed from the master to the subordinates. When a new subordinate switch is added, or an existing switch removed, the master will issue a notification of this event and all the subordinate switches will update their tables accordingly.

 

**StackWise Configuration**

Designate one of the StackWise switches as the "master switch." Choose the one with the best specifications if you own different StackWise switches; otherwise, pick any of them. The other switch will act as the "slave." Connect to the chosen master switch and perform the following:

 

**Add/Remove Provisioned Switch**

Switch(config)# *[no]* switch *<number* provision *<ws-model-##*

-   A switch is automatically provisioned to a switch once the stack cable is plugged in. The *no* form of this command removes a provisioned switch from the stack. Model number example: *ws-3750-48ts*

 

**Renumber Switch**

Switch(config)# switch *<number* renumber *<number*

-   Use show switch to display switch number, renumber the switch 1 is master other switches can be renumbered based on preference on how to number switch, in most cases just use the physical order of the switches.

 

**Adjust Switch Priority**

Switch(config)# switch *<number* priority *<priority*

-   Use show switch to display switch number, priority is used manipulate switch master election.

 

**Troubleshooting/Verification**

Switch# show switch

-   Displays switch stack information including (switch number, role, MAC, priority and state).

  

Switch# show platform stack-manager all

-   Displays information related to the management of stacks, which includes the stack-protocol version, history of changes to the stack, etc.

 

Switch# show switchport stack-ports

-   Displays the stack-ports through which the switches are connected to the stack'

 

Switch# show switch neighbors

-   Displays connected switches on each port

 

Switch# show inventory

-   Displays switch serials which can be used to identify which switch is a master or slave for renumbering purposes.

  

Switch# debug platform stack-manager sdp

-   Displays the Stack Discovery Protocol (SDP) debug messages.

  

Switch# debug platform stack-manager ssm

-   Displays the stack state-machine debug messages.

 

Private VLANs

Friday, January 07, 2011

7:44 PM

 

**Overview**

Private VLANs were developed to provide the ability to isolate hosts at Layer 2. By using this method of segregating hosts you save both VLANs and IP address space compared to traditional methods.

 

A private VLAN is defined as pairing of a primary VLAN with a secondary VLAN. It's important to remember that secondary VLANs have the same range and are defined in a similar manner as primary VLANs, but operate in different modes.

 

**Isolated:** An isolated port is a host port that belongs to an isolated secondary VLAN. It has complete Layer 2 separation from other ports within the same private VLAN, except for the promiscuous ports. Private VLANs block all traffic to isolated ports except traffic from promiscuous ports. Traffic received from an isolated port is forwarded only to promiscuous ports. You can only have a single isolated VLAN within a Primary VLAN.

 

**Community:** A community port is a host port that belongs to a community secondary VLAN. Community ports communicate with other ports in the same community VLAN and with promiscuous ports. These interfaces are isolated at Layer 2 from all other interfaces in other communities and from isolated ports within their private VLAN. You can have multiple Community VLANs per Primary VLAN.

 

**Promiscuous:** A promiscuous port belongs to the primary VLAN and can communicate with all interfaces, including the community and isolated host ports that belong to the secondary VLANs associated with the primary VLAN.

 

**Host:** The port inherits its behavior from the type of private VLAN it is assigned to.

 

![](''/media/image5.jpg){width="5.177083333333333in" height="3.53125in"}

 

*Image provided by [Cisco Systems](http://www.cisco.com)*

 

**Private VLAN Configuration**

There are many different scenarios for Private VLAN configuration, this should allow you to understand the concepts and configuration involved.

**Set VTP Mode to Transparent**

Switch(config)# vtp mode transparent

-   You must configure VTP to transparent mode before you can create a private VLAN. Private VLANs are configured in the context of a single switch and cannot have members on other switches. Private VLANs also carry TLVs that are not known to all types of Cisco switches.

**Specify Primary VLAN**

Switch(config)# vlan 100

Switch(config-vlan)# private-vlan primary

**Create Private VLANs**

Switch(config)# vlan 201

Switch(config-vlan)# private-vlan isolated

 

Switch(config)# vlan 202

Switch(config-vlan)# private-vlan community

 

**Associate Private VLANs to the Primary VLAN**

Switch(config)# vlan 100

Switch(config-vlan) private-vlan association 201, 202

 

**Configure Layer 2 Interface as a Private-VLAN Host Port**

Switch(config)# interface fastethernet0/15

Switch(config-if)# switchport mode private-vlan host

Switch(config-if)# switchport private-vlan host-association 100 201

-   Primary 100, secondary 201 (isolated)

 

Switch(config)# interface fastethernet0/20

Switch(config-if)# switchport mode private-vlan host

Switch(config-if)# switchport private-vlan host-association 100 201

-   Primary 100, secondary 202 (community)

 

**Configure Layer 2 Interface as a Private-VLAN Promiscuous Port**

Switch(config)# interface fastethernet0/1

Switch(config-if)# switchport mode private-vlan promiscuous

Switch(config-if)# switchport private-vlan mapping 100 add 201,202

-   Maps primary VLAN to private VLANs 201 and 202; allowing them to communicate to the promiscuous port.

  

**Mapping secondary VLANs to a Primary VLAN Layer 3 interface**

If the Private VLANs will be used for inter-VLAN routing, you can configure a SVI for the primary VLAN (100) and map secondary VLANs (201,202) to the SVI.

 

Switch(config)# interface vlan 10 0

Switch(config-if)# private-vlan mapping 201, 202

 

**NOTE:** Remember Private-VLANs work at Layer 2, if you allow Inter-VLAN routing between secondary VLANs, they will be able to communicate over Layer 3; you can exclude secondary VLANs to prevent Layer 3 communication or use an ACL.

 

**Troubleshooting/Verification**

Switch# show vlan private-vlan

-   Displays configured primary VLAN and the associated secondary VLANs and configuration mode.

  

Switch# show interfaces private-vlan mapping

-   Displays interfaces mapped to private VLANs, useful reviewing SVI configuration.

  

Switch# show interface switchport

-   Display private-VLAN configuration on interfaces.

 

Switch# show interfaces [type slot/port]

-   Displays configured port information related to the individual interface and private VLAN configuration.

 

Switch# show interfaces status

-   Displays the status of interfaces, including the VLANs to which they belongs.

  

Switch# show vlan private-vlan [type]

-   Display the private-VLAN information for the switch.

 

SPAN, RSPAN, ERSPAN

Tuesday, January 04, 2011

8:51 AM

 

**Overview**
Switched Port Analyzer (SPAN) also referred to Port Mirroring; is used on a network switch to send a copy of network packets seen on one switch port (or an entire VLAN) to a network monitoring connection on another switch port. This is commonly used for network appliances that require monitoring of network traffic, such as an intrusion-detection system.**

Basic SPAN Configuration**

Switch(config)# monitor session 1 source interface fa0/8

Switch(config)# monitor session 1 destination interface fa0/12

-   Effectively all traffic sent or received on port fa0/8 will be replicated out fa0/12.

 

**Advanced SPAN Configuration**

Switch(config)# monitor session 2 source vlan 20 tx

-   Source vlan 20 simply defines the traffic from the entire vlan to be replicated; contrasted to single interface replication.

  

Switch(config)# monitor session 2 source interface fa0/8 rx

-   RX or TX defines whether you would like to replicate receiving or transmitting traffic from the corresponding port, if no configuration is present the default will be to replicate both.

 

Switch(config)# monitor session 2 source interface fa0/1

Switch(config)# monitor session 2 filter vlan 1 - 8, 39, 52

-   Interface fa0/1 is a trunk port, by default it would replicate all vlan traffic to the destination port. But the filter vlan command stops the defined vlans from being replicated.

 

Switch(config)# monitor session 2 destination interface fa0/15 encapsulation replicate

-   SPAN typically ignores BPDU, VTP, DTP, PagP, and STP frames however with encapsulation replicate these frames are also sent to the destination port.

  

**Remote SPAN**

RSPAN only differs from SPAN in which the destination port is not on the same switch as the traffic source. In a way it uses a vlan as a tunnel to the destination port for monitoring, as seen below.

 

Switch1(config)# vlan 300

Switch1(config-vlan)#remote span

-   Defines this VLAN as a remote SPAN vlan and you do not assign ports to this vlan.
  

Switch1(config)# monitor session 10 source interface fa0/5

Switch1(config)# monitor session 10 source vlan 50 rx

Switch1(config)# monitor session 10 destination vlan 300

-   The following commands defines which ports to replicate and then sends the information to destination vlan 300. Again, think of this as a tunnel to another switch.

 

Switch2(config)# vlan 300

Switch2(config-vlan)# remote span

Switch2(config)# monitor session 35 source vlan 300

Switch2(config)# monitor session 35 destination interface fa0/20

-   The source vlan 300 is then used at the end point by the destination port fa0/20 on Switch2.

 

**NOTE:** The session ids are different 10, 35 in this example. This is indifferent for SPAN configuration, however they must be defined a range between 1 - 66. Also remember the intervlan communication must be configured between switches, otherwise the remote switch will not be able to transmit the replicated data to the destination.

 

**Troubleshooting**

Switch(config)# show monitor session

-   This can be used to verify SPAN or RSPAN operations.

 

Optimizing System Resources (SDM)

Friday, January 07, 2011

7:44 PM

 

**Overview**

You can use Switch Database Management (SDM) templates to configure system resources in the switch to optimize support for specific features, depending on how the switch is used in the network. You can select a template to provide maximum system usage for some functions or to use the default template to balance resources.

 

The templates prioritize system resources to optimize support for these types of features:

 

**Routing**---The routing template maximizes system resources for unicast routing, typically required for a router or aggregator in the center of a network.

**VLANs**---The VLAN template disables routing and supports the maximum number of unicast MAC addresses. It would typically be selected for a Layer 2 switch.

**Default**---The default template gives balance to all functions.

**SDM Configuration**

Switch(config)# sdm prefer { default | routing | vlan }

-   Sets resource levels, view above the above options.

  

**NOTE:** The device must be reloaded for the SDM preference to change.

 

**Troubleshooting/Verification**

Switch# show sdm prefer

-   Displays current SDM resource configuration and levels.

 

Link State Tracking

Friday, January 07, 2011

7:45 PM

 

**Overview**

Link-state tracking, also known as trunk failover, is a feature that binds the link state of multiple interfaces.

 

**Scenario**

A server is connected to multiple access layer switches, the access switches connects to the distribution layer switches. The server is paired with duel NICs and each NIC connects to a different access layer switch, which will allow it to take a different path if necessary. The teaming NICs understand that if their directly connected link goes down to use the other NIC. However, what if the link goes down behind the access layer switch in other words the "upstream" link to the distribution switch. This is where *Link State Tracking* comes into play, essentially if the upstream link goes down it will notify the downstream link directly connected to the server and cause it to put its link state position to down. This would cause the server to use it's failover NIC, which has an alternate path back to the distribution.

 

![](''/media/image6.jpg){width="2.2083333333333335in" height="2.8333333333333335in"}

 

*Image provided by [noswitchport.com](http://www.noswitchport.com)*

 

**Link-State Tracking Configuration**

**Define Link State Group**

Switch(config)# link state track 1

-   The group number can be 1 to 2; default of 1

  

**Apply Link State Group to Uplink Interfaces**

ALS1(config)# interface FastEthernet0/1

ALS1(config-if)# link state group 1 upstream

  

ALS2(config)# interface FastEthernet0/1

ALS2(config-if)# link state group 1 upstream

 

**Apply Link State Group to Downstream Links**

ALS1(config)# interface FastEthernet0/5

ALS1(config-if)# link state group 1 downstream

 

ALS2(config)# interface FastEthernet0/5

ALS2(config-if)# link state group 1 downstream

 

**Troubleshooting/Verification**

Switch# show link state group

-   Displays link-state group information and status

  

Switch# show link state group detail

-   Displays link state group information as well as individual interface link status

 

CAM Maintenance

Monday, January 10, 2011

2:19 PM

 

**Overview**

Switches at their core are built around Layer 2 addressing or Media Access Control (MAC) Addresses. MAC

Addresses are stored in Content Addressable Memory and is usually dynamic populated; but can be manually adjusted. The CAM table maps MAC addresses to specific interfaces, if a frame requested the destination of another device on the LAN the switch knows where to send the frames based on the MAC Address Table.

 

**Static Entries**

Switch(config)# mac-address-table static <*mac*-*address* vlan <*vlan*-*id* interface *<type slot/port*

  

**Aging**

Switch(config)# mac-address-table aging-time *<seconds*

-   Defines the aging time for entries in the CAM table, before they are removed from the table.

 

**Logging / MAC Address Notification Traps (SNMP)**

**Configure SNMP Trap**

Switch(config)# snmp-server enable traps MAC-Notification

-   Enables the switch to send MAC address traps to the NMS

Switch(config)# snmp-server host *<nms-address <snmp-string* MAC-Notification

-   Specifies the NMS address, community string and MAC Notification traps for SNMP.

 

**Configure MAC Address Table Notifications**

Switch(config)# mac-address-table notification

Switch(config)# mac address-table notification interval 60

-   Set the notification trap interval. The switch sends the notification traps when this amount of time has elapsed.

Switch(config)# mac address-table notification history-size 100

-   Specifies the maximum number of entries in the MAC notification history table. The range is 0 to 500 entries.

 

**Enable MAC Notification Monitoring on an Interface**

Switch(config)# interface FastEthernet0/24

Switch(config-if)# snmp trap mac-notification { added | removed}

-   Enables MAC address notification, specify to notify SNMP NMS when addresses are either added or removed from the table.

 

**NOTE:** SNMP is an optional component for MAC Address notifications, for a simple output of added/removed addresses use: *show mac-address-table notification*

 

**Unicast MAC address filtering**

Switch(config)# mac address-table static <*mac*-*address* vlan *<vlan-id* drop

-   Specifies any frames from this MAC address on this VLAN, to be dropped.

  

**Troubleshooting/Verification**

Switch# show mac address-table

-   Displays address table with VLANs, MAC Addresses, Types and Interface output.

  

Switch# show mac address-table address *<mac-address*

-   Displays an the same information as above, except for a specific MAC address.

  

Switch# show mac address-table static

-   Displays all statically configured entries

  

Switch# clear mac-address-table dynamic

-   Clears CAM/MAC Address Table

 

Switch# show mac address-table vlan *<vlan-id*

-   Displays the MAC address table information for the specified VLAN.

  

Switch# show mac address-table count *<vlan-id*

-   Displays the number of addresses present in all VLANs or the specified VLAN.

 

Switch# show mac address-table notification

-   Displays the MAC address notifications without the use of SNMP.

 

Macros

Friday, January 07, 2011

7:45 PM

 

**Overview**

Smartports macros provide a convenient way to save and share common configurations. You can use Smartports macros to enable features and settings based on the location of a switch in the network and for mass configuration deployments across the network.

 

Each Smartports macro is a set of command-line interface (CLI) commands that you define. Smartports macros do not contain new CLI commands; they are simply a group of existing CLI commands.

 

When you apply a Smartports macro on an interface, the CLI commands within the macro are configured on the interface. When the macro is applied to an interface, the existing interface configurations are not lost. The new commands are added to the interface and are saved in the running configuration file.

**Cisco-Default Smartport Macros**

There are Cisco-default Smartports macros embedded in the switch software.

 

Switch# show parser macro

-   Displays a list of embedded macros already configured on the Switch.

 

**Cisco-global**

Use this global configuration macro to enable rapid PVST+, loop guard, and dynamic port error recovery for link state failures.

**Cisco-desktop**

Use this interface configuration macro for increased network security and reliability when connecting a desktop device, such as a PC, to a switch port.

 

**Cisco-phone**

Use this interface configuration macro when connecting a desktop device such as a PC with a Cisco IP Phone to a switch port. This macro is an extension of the **cisco-desktop** macro and provides the same security and resiliency features, but with the addition of dedicated voice VLANs to ensure proper treatment of delay-sensitive voice traffic.

 

**Cisco-switch**

Use this interface configuration macro when connecting an access switch and a distribution switch or between access switches connected using GigaStack modules or GBICs.

 

**Cisco-router**

Use this interface configuration macro when connecting the switch and a WAN router.

 

**Creating Smartport Macros**

Switch(config)#macro name test
Enter macro commands one per line. End with the character '@'.
switchport  mode access
switchport access vlan $VLANID

# Specify VLAN

switchport port-security

switchport port-security maximum $MAX
# Specify a maximum number of MAC addresses that can bind to the interface.

spanning-tree portfast
speed 100
duplex full
no shut
#macro keywords $VLANID $MAX

 

**NOTE:** $ denotes that this area will require manual input when applying to an interface. # allows you to put a note within the Macro.

 

**Apply SmartPort Macros**

Switch(config)# interface FastEthernet0/0

Switch(config-if)# macro apply test ?

-   At this point the output will display and keywords specified in the Macro configuration. In this case VLANID, you would then know that the VLANID needs to be specified.

Switch(config-if)# macro apply test $VLANID 10 $MAX 1

This applies macro "test" and assigns the interface to VLAN 10 and allows a maximum of 1 MAC address to bind to the interface.

Switch(config-if)# macro description *<text*

-   Can be used to specify an additional comment specific to the Macro on the interface.

  

**Apply Global Macros**

Global Macros are applied in the exact same manner, except or the fact they are not applied under an interface.

 

Switch(config)# macro global apply <macro-name

 

**Troubleshooting/Verification**

Switch# show parser macro brief

-   Displays the configured macro names

 

Switch# show parser macro *<macro-name*

-   Displays a list of embedded macros already configured on the Switch. You can also specify a specific name of a Macro.

  

Switch# show parser macro description interface [*interface-id*]

-   Displays the macro description for all interfaces or for a specified interface.

  

Switch(config-if)# macro apply test trace

-   Displays output of the Macro being applied, line by line and will show any errors in the configuration.

 

Switch(config)# default interface [interface ID]

-   Clears all configuration from the specified interface and puts it back to the defaults.

 

 

Bridging

Monday, January 10, 2011

4:38 PM

 

**Overview**

Bridges and switches are data communication devices that operate principally at Layer 2 of the OSI reference model. As such, they are widely referred to as data link layer devices. Several kinds of bridging have proven important as internetworking devices. Today, switching technology has emerged as the evolutionary heir to bridging-based internetworking solutions. Switching implementations now dominate applications in which bridging technologies were implemented in prior network designs. Superior throughput performance, higher port density, lower per-port cost, and greater flexibility have contributed to the emergence of switches as replacement technology for bridges and as complements to routing technology.

 

**Note: **Transparent Bridging and Integrated Routing and bridging is configured on a Router, while Fall-Back Bridging is performed on a Switch. It's unlikely that TB & IRB will be seen on the lab, however it's very well possible to see Fall-Back Bridging in the switching section.

 

Transparent

Monday, January 10, 2011

4:38 PM

 

**Overview**

Bridging is necessary when non-routable protocols such as Network Basic Input/Output System (NetBIOS) and Local-Area Transport (LAT) are used to communicate between devices connected across different segments.

Transparent bridging essentially turns the routed interfaces into switchports and even goes as far as running STP.

Bridging can also be configured for routable protocols instead of being routed. Transparent Bridging (TB) is used to bridge networks using similar media. TB is mostly used in an Ethernet environment. A device configured for TB initially learns the location of the devices by looking at the source MAC addresses in the frames. Next it builds a bridging table containing the MAC addresses and the ports through which they are reachable. To forward or filter traffic on the ports, the destination address in the frames is compared with information in this table. When there are multiple paths available between any two segments for providing redundancy, the spanning tree algorithm is used. This prevents loops. TB can either be remote or local. Local bridges provide direct connections between LAN segments in the same site. Remote bridges connect LAN segments at different sites, usually over WAN links.

**Bridging using non-routable protocols on a local segment**

A single router has multiple interfaces which are directly connected to workstations which required NetBEUI (non-routable protocol) to communicate with each other. This simple configuration allows for workstations connecting interfaces fa0/3 and fa0/4 to communicate using NetBEUI using a Transparent Bridge.

**Create bridge group 1 using 802.1d**

Router(config)# bridge 1 protocol ieee

*
***Apply bridge group to interfaces**

Router# Interface fastethernet0/3

Router(config-if)# bridge-group 1

 

Router# Interface fastethernet0/4
Router(config-if)# bridge-group 1

**Bridging using non-routable protocols over WAN**
Now, let's say you have these same workstations that need to communicate over a WAN connection to other NetBEUI workstations on another router. You can assign the bridge group to the WAN interface and configure the remote site in the exact same manner to allow the non-routable protocol to communicate.

**Create bridge group 1 using 802.1d**

Router(config)# bridge 1 protocol ieee

*
***Apply bridge group to interfaces (Router1)**

Router1# Interface fastethernet0/1

Router1(config-if)# bridge-group 1

 

Router1# Interface fastethernet0/2
Router(config-if)# bridge-group 1

 

Router1# interface serial0/0

Router1(config-if)# description Link to Router 2

Router1(config-if)# ip address 172.16.2.1 255.255.255.252

Router1(config-if)# bridge-group 1

 

**Apply bridge group to interfaces (Router2)**

Router2# Interface fastethernet0/3

Router2(config-if)# bridge-group 1

 

Router2# Interface fastethernet0/4
Router2(config-if)# bridge-group 1

 

Router2# interface serial0/0

Router2(config-if)# description Link to Router 1

Router2(config-if)# ip address 172.16.2.2 255.255.255.252

Router2(config-if)# bridge-group 1

**NOTE:** STP parameters can be modified on TB just like Fall-Back Bridging, view subpage "Adjusting STP Parameters"

 

**Troubleshooting/Verification**

Router# show bridge

-   Shows MAC addresses learned

  

Router# show spanning-tree

-   Displays which interfaces are part of each bridge group as well as STP settings.

 

Integrated Routing and Bridging (IRB)

Monday, January 10, 2011

4:38 PM

 

**Overview**

In order for a VLAN to span a router, the router must be capable of forwarding frames from one interface to another, while maintaining the VLAN header. If the router is configured for routing a Layer 3 (network layer) protocol, it will terminate the VLAN and MAC layers at the interface a frame arrives on. The MAC layer header can be maintained if the router is bridging the network layer protocol.

 

However, regular bridging still terminates the VLAN header. Using the IRB feature in Cisco IOS^®^ Release 11.2 or greater, a router can be configured for routing and bridging the same network layer protocol on the same interface. This allows the VLAN header to be maintained on a frame while it transits a router from one interface to another. IRB provides the ability to route between a bridged domain and a routed domain with Bridge Group Virtual Interface (BVI). The BVI is a virtual interface within the router that acts like a normal routed interface that does not support bridging, but represents the comparable bridge group to routed interfaces within the router. The interface number of the BVI is the number of the bridge group that the virtual interface represents. The number is the link between the BVI and the bridge group.

 

When you configure and enable routing on the BVI, packets that come in on a routed interface, which are destined for a host on a segment in a bridge group, are routed to the BVI. From the BVI, the packet is forwarded to the bridging engine, which forwards it through a bridged interface. This is forwarded based on the destination MAC address. Similarly, packets that come in on a bridged interface, but are destined for a host on a routed network, first go to the BVI. Next, the BVI forwards the packets to the routing engine before it sends them out of the routed interface. On a single physical interface, the IRB can be created with two VLAN sub-interfaces (802.1Q tagging); one VLAN sub-interface has an IP address that is used for routing, and the other VLAN sub-interface bridges between the sub-interface used for routing and the other physical interface on the router.

 

Since the BVI represents a bridge group as a routed interface, it must be configured only with Layer 3 (L3) characteristics, such as network layer addresses. Similarly, the interfaces configured for bridging a protocol must not be configured with any L3 characteristics.

 

**Considerations**

-   The default route/bridge behavior on a router is to route all packets first and then bridge them. This is precisely why configuring transparent bridging does not impact the routed domain. However, when the IRB is enabled, the behavior changes to bridge all packets first. The bridge group must be enabled to route with command *bridge bridge_number route ip* if routing IP also is enabled**.**

  

-   Packets of *nonroutable protocols,* such as local-area transport (LAT) or SNA, always are bridged. You cannot disable bridging for the nonroutable protocols.

 

-   Bridging attributes cannot be configured on a BVI interface, only layer 3 or routing options. All Layer 2 information is configured on the physical bridged interfaces.

 

-   IRB supersedes concurrent routing and bridging (CRB), which no longer should be used. The difference between IRB and CRB is that CRB would only allow for different bridge groups to either bridge or route data. However, unlike IRB they could not be done on the same interface. For example in CRB if PC_A wanted to access PC_C it would become a routed interface, but it would not be able to communicate to PC_B via bridge. While IRB can both bridge communication with PC_B on the same segment, like a Switch or use the BVI to route across and access PC_C.

 

![](''/media/image7.gif){width="6.0in" height="3.0625in"}

 

 

 

*Image provided by [Cisco Systems](http://cisco.com/)*

 

**Configuring Integrated Routing and Bridging (IRB)**

This configuration is an example of IRB. The configuration allows bridging IP between two Ethernet interfaces, and routing IP from bridged interfaces using a Bridged Virtual Interface (BVI). In the following network diagram, when PC_A attempts to contact PC_B, the router R1 detects that the destination's (PC_B) IP address is in the same subnet, so the packets are bridged by router R1 between interface E0 and E1. When PC_A or PC_B attempt to contact PC_C, the router R1 detects that the destination's (PC_C) IP address is in a different subnet, and the packet is routed using the BVI. This way, IP protocol is bridged as well as routed on the same router.

 

**Enable IRB**

Router(config)# bridge irb

 

**Create Bridge Group**

Router(config)# bridge group 1

 

**Enable bridging (802.1d STP)**

Router(config)# bridge 1 protocol ieee

 

**Apply Bridge Group to bridged interfaces; in this case the hosts PC_A & B**

Router(config)# interface ethernet0

Router(config-if)# bridge-group 1

 

Router(config)# interface ethernet1

Router(config-if)# bridge-group 1

 

**Create Bridged Virtual Interface (BVI)**

Router(config)# interface BVI 1

-   The BVI number must be the same as the bridge group.

Router(config-if)# ip address 10.10.10.1 255.255.255.0

 

**Enable Bridging for a specific protocol**

Router(config)# bridge 1 bridge { ip | appletalk | clns | ipx }

-   Specifies the protocol to be bridged; default IP.

**Enables Routing for IP protocol**

Router(config)# bridge 1 route ip

-   Without adding this command ONLY bridging would be allowed and the PCs would be unable to route to the BVI and therefore unable to access PC_C

 

**Troubleshooting/Verification**

Router# show interface *[interface-id]* irb

-   Displays the protocols that are routed or bridged for the specified interface.

 

Fall-Back Bridging

Monday, January 10, 2011

4:38 PM

 

**Overview**

Fallback bridging also known as VLAN bridging. Bridges two or more VLANs and/or routed ports together into a single bridge domain to bridge non IP protocols. A set of Switch Virtual Interfaces (SVI - each representing a VLAN) and/or routed ports (RP) can be grouped together to form a bridge group. Each SVI/RP can be associated with a single bridge group only. If the destination MAC address of a non-IP packet has already been learnt on the bridge group, the packet is simply forwarded to the respective port. If an address is not yet learned, the packet is flooded on all VLANs and RPs in the bridge group. Fallback bridging uses a separate spanning tree (VLAN bridge STP) to prevent loops.

 

Its main use is to allow machines that speak non-routed or non-supported protocols (SNA, DECNet, AppleTalk, etc.) to communicate across VLANs and routed ports. Fall-Back Bridging is essentially transparent bridging except it is used on a switch instead of a router to allow communication between VLANs.

  

**Notes**

-   An interface can only be a member of only a single bridge group.

-   Each VLAN has it's own Spanning-Tree instance, which runs on top of the bridge group to prevent loops.

 

**Fall-Back Bridging Configuration**

 

![Fallback Bridging](''/media/image8.gif){width="3.8645833333333335in" height="3.6354166666666665in"}

*
Image provided by [blog.humanmodem.com](http://blog.humanmodem.com/)*

 

**Create bridge group**

Switch(config)# bridge 1 protocol vlan-bridge
**
Apply bridge group to SVIs and/or Routed Port**

Switch(config)# interface vlan 1

Switch(config-if)# bridge-group 1

Switch(config)# interface vlan 2

Switch(config-if)# bridge-group 1

 

Switch(config)# interface fa0/1

Switch(config-if)# no switchport

Switch(config-if)# bridge-group 1

**Apply switchport to SVI**

This will allow for the host connected to Interface fa0/2 to communicate between VLANs using non-IP protocols.

 

Switch(config)# interface fa0/2
Switch(config)# switchport mode access

Switch(config)# switchport access vlan 2

 

Now the switch will build a bridging table and bridge all packets between those interfaces except the following: IPv4 & IPv6, ARP, RARP, Loopback, and Frame Relay ARP.

**Configuring the Bridge Table Aging Time**

A switch forwards, floods, or drops packets based on the bridge table. The bridge table maintains both static and dynamic entries. Static entries are entered by you or learned by the switch. Dynamic entries are entered by the bridge learning process. A dynamic entry is automatically removed after a specified length of time, known as aging time, from the time the entry was created or last updated.

 

Switch# bridge *[1-255]* aging-time *500*

-   Default: 300 seconds

  

**Preventing Forwarding of Dynamically Learned Addresses**

By default, the switch forwards any frames for stations that it has dynamically learned. By disabling this activity, the switch only forwards frames whose addresses have been statically configured into the forwarding cache.

Switch(config)# no bridge [*1-255]* acquire

-   Enable the switch to stop forwarding any frames for stations that it has dynamically learned through the discovery process and to limit frame forwarding to statically configured stations.
  

**NOTE:** The switch filters all frames except those whose destined-to addresses have been statically configured into the forwarding cache; configuration shown below.

 

**Filtering Frames by a Specific MAC Address**

A switch examines frames and sends them through the internetwork according to the destination address; a switch does not forward a frame back to its originating network segment. You can use the software to configure specific administrative filters that filter frames based on information other than the paths to their destinations.

 

You can filter frames with a particular MAC-layer station destination address. You can configure any number of addresses in the system without a performance penalty.

 

Switch(config)# bridge *[1-255]* address *[mac-address] {forward | discard} [Interface-id]*

 

**Example:
**Switch(config)# bridge 1 address 08:00:69:02:01:FC forward FastEthernet 0/8

-   Any frame received from the specified MAC address will be forwarded out through the specified interface which belongs to bridge group 1.

 

**Troubleshooting/Verification**
Switch# show bridge group *[1-255]*

-   Displays configured bridge groups and the associated SVIs and routed ports.

 

Switch# show bridge *[1-255] { interface-id | mac-address }*

-   Displays MAC addresses learned in the bridge group.

 

Switch# clear bridge *[1-255]*

-   Removes any learned entries from the forwarding database.

 

Adjusting STP Parameters

Monday, January 10, 2011

4:39 PM

 

**Overview**

You might need to adjust certain spanning-tree parameters if the default values are not suitable. You configure parameters affecting the entire spanning tree by using variations of the **bridge** global configuration command. You configure interface-specific parameters by using variations of the **bridge-group** interface configuration command.

**NOTE:** Bridge groups use their own instance of Spanning-Tree and it acts exactly like regular STP. View the STP section for further clarification for the below configuration options.

 

**Changing the VLAN-Bridge Spanning-Tree Priority**

You can globally configure the VLAN-bridge spanning-tree priority of a switch when it ties with another switch for the position as the root switch. You also can configure the likelihood that the switch will be selected as the root switch.

 

Switch(config)# bridge *[1-255]* priority number *[0-65535]*

-   Default: 32,768

 

**Changing the Interface Priority**

You can change the priority for a port. When two switches tie for position as the root switch, you configure a port priority to break the tie. The switch with the lowest interface value is elected.

 

Switch(config)# interface *[interface-id]*

Switch(config-if)# bridge-group [1-255] priority [0-255]

-   Adjusted in increments of 4, lower the number the more likely that the port on the switch will be chosen as the root. Default: 128

  

**Assigning a Path Cost**

Each port has a path cost associated with it. By convention, the path cost is 1000/data rate of the attached LAN, in Mbps.

 

--For 10 Mbps, the default path cost is 100.

--For 100 Mbps, the default path cost is 19.

--For 1000 Mbps, the default path cost is 4.

 

Switch(config)# interface *[interface-id]*

Switch(config-if)# bridge-group *[1-255]* path-cost *[0-65535]*

-   Lower the cost, the better the path.

  

**Adjusting BPDU Intervals**
**Hello Time**
Switch(config)# bridge *[1-255]* hello-time *[1-10]*

-   Specify the interval between Hello BPDUs. Default: 2 seconds

  

**Forward Time
**Switch(config)# bridge *[1-255]* forward-time *[4-200]*

-   The forward-delay interval is the amount of time spent listening for topology change information after a port has been activated for switching and before forwarding actually begins. Default: 20 seconds

**
Maximum Age Time**

Switch(config)# bridge *[1-255]* max-age *[6-200]*

-   If a switch does not receive BPDUs from the root switch within a specified interval, it recomputes the spanning-tree topology. Default: 30 seconds

 

**NOTE:** Each switch in a spanning tree adopts the interval between hello BPDUs, the forward delay interval, and the maximum idle interval parameters of the root switch, regardless of what its individual configuration might be.

**Disable Spanning-Tree on an Interface**

When a loop-free path exists between any two switched subnetworks, you can prevent BPDUs generated in one switching subnetwork from impacting devices in the other switching subnetwork, yet still permit switching throughout the network as a whole. For example, when switched LAN subnetworks are separated by a WAN, BPDUs can be prevented from traveling across the WAN link.

 

Switch(config)# interface *[interface-id]*

Switch(config-if)# bridge-group *[1-255]* spanning-disabled

-   Disables STP on the specified port

 

 

Security

Friday, January 14, 2011

9:49 AM

 

**Overview**
The Cisco Catalyst 3560 Series supports a comprehensive set of security features for connectivity and access control, including network admission control (NAC), ACLs, Dynamic ARP Inspection, IP Source Guard, VPN Routing/Forwarding Lite (VRF Lite), port-level security, and identity-based network services with 802.1x and extensions. These features increase LAN security; protect passwords and configuration information; offer options for network security based on users, ports, or MAC addresses; and help quicken responses to intruder and hacker detection. NAC helps organizations to limit damage from viruses and worms by enforcing security-policy compliance on endpoint devices.

 

The following subpages will describe these security measures and how to configure them.

 

Port Security

Friday, January 14, 2011

9:49 AM

 

**Overview**

You can use the port security feature to restrict input to an interface by limiting and identifying MAC addresses of the stations allowed to access the port. When you assign secure MAC addresses to a secure port, the port does not forward packets with source addresses outside the group of defined addresses. If you limit the number of secure MAC addresses to one and assign a single secure MAC address, the workstation attached to that port is assured the full bandwidth of the port.

 

If a port is configured as a secure port and the maximum number of secure MAC addresses is reached, when the MAC address of a station attempting to access the port is different from any of the identified secure MAC addresses, a security violation occurs. Also, if a station with a secure MAC address configured or learned on one secure port attempts to access another secure port, a violation is flagged.

 

**Security Violations**

It is a security violation when one of these situations occurs:

 

-   The maximum number of secure MAC addresses have been added to the address table, and a station whose MAC address is not in the address table attempts to access the interface.

 

-   An address learned or configured on one secure interface is seen on another secure interface in the same VLAN.

 

You can configure the interface for one of three violation modes, based on the action to be taken if a violation occurs:

 

**Protect** --- when the number of secure MAC addresses reaches the maximum limit allowed on the port, packets with unknown source addresses are dropped until you remove a sufficient number of secure MAC addresses to drop below the maximum value or increase the number of maximum allowable addresses. You are not notified that a security violation has occurred

 

**Restrict** --- same as protect mode except, in this mode, you are notified that a security violation has occurred. An SNMP trap is sent, a syslog message is logged, and the violation counter increments.

 

**Shutdown** --- a port security violation causes the interface to become error-disabled and to shut down immediately, and the port LED turns off. An SNMP trap is sent, a syslog message is logged, and the violation counter increments. This is the default mode.

 

**MAC Addresses**

The switch supports these types of secure MAC addresses:

 

**Static** --- secure MAC addresses---These are manually configured by using the switchport port-security mac-address mac-address interface configuration command, stored in the address table, and added to the switch running configuration.

 

**Dynamic** --- secure MAC addresses---These are dynamically configured, stored only in the address table, and removed when the switch restarts.

 

**Sticky** --- secure MAC addresses---These can be dynamically learned or manually configured, stored in the address table, and added to the running configuration. If these addresses are saved in the configuration file, when the switch restarts, the interface does not need to dynamically reconfigure them.

 

**Port Security Configuration**

There are several different combinations of port security options that can be implemented, shown below are just a few configuration examples.

 

**Access Port Security**

Switch(config)# interface fastethernet0/1

Switch(config-if)# switchport mode access

Switch(config-if)# switchport port-security

-   Enables port-security

Switch(config-if)# switchport port-security maximum 3

-   Defines the maximum number of addresses that can be mapped to the interface

Switch(config-if)# switchport port-security { protect | restrict | shutdown }

-   Sets the violation mode; defined above

Switch(config-if)# switchport port-security mac-address [mac-address]

-   Defines a static MAC-Address entry that is saved in the running-configuration. If you don't statically map the MAC-addresses up to the maximum the remaining will be learned dynamically. In this example 2 more MAC-addresses will be mapped to the interface before a violation occurs.

 

**Access Mode with Port Security using Sticky**

Switch(config-if)# switchport port-security mac-address sticky

-   Enables sticky learning on the interface

Switch(config-if)# switchport port-security mac-address sticky [mac-address]

-   Statically enter a sticky secure MAC-Address, this is the same as a static entry, just used when sticky learning is configured. Similar as the static entries, if you don't statically map the mac-addresses up to the maximum, the remaining will be dynamically learned and sticky to the interface and therefore saved to the running-configuration.

  

**Set maximum MAC-Addresses for access and voice VLANs Independently**

This is assuming that the access and voice VLANs have been configured on the access switchport.

 

Switch(config-if)# switchport port-security maximum 1 vlan access

-   Allows for only a single MAC-Address to be learned for the access VLAN.

Switch(config-if)# switchport port-security maximum 1 vlan voice

-   Allows for only a single MAC-Address to be learned for the voice VLAN.

  

**Trunk Port Security**

Switch(config)# interface gigabitethernet0/2

Switch(config-if)# switchport mode trunk

Switch(config-if)# switchport port-security

Switch(config-if)# switchport port-security maximum 10

-   Set the maximum number of secure MAC addresses for the interface.

Switch(config-if) switchport port-security maximum 1 vlan 10

-   Set a per-VLAN maximum value

Switch(config-if) switchport port-security maximum 1 vlan 15-30

-   Set a per-VLAN maximum value on a range of VLANs

Switch(config-if)# switchport port-security mac-address *xxxx.xxxx.xxxx* vlan *<vlan-id*

-   Defines the VLAN on which this defined MAC-Address can be learned.

 

**Aging Time / Type**

You can use port security aging to set the aging time for all secure addresses on a port. Two types of aging are supported per port:

-   Absolute---The secure addresses on the port are deleted after the specified aging time.

-   Inactivity---The secure addresses on the port are deleted only if the secure addresses are inactive for the specified aging time.

 

**Aging Configuration**

Switch(config)# interface FastEthernet0/5
Switch(config-if)# switchport port-security aging time *<minutes*

-   Specifies max-age time of MAC-Addresses (0-1440)

Switch(config-if)# switchport port-security aging type { absolute | inactivity }

-   Defines age type as described above

Switch(config-if)# switchport port-security aging static

-   Enables aging for statically configured MAC-Addresses

**
Example**

In this example expiration of MAC addresses after five minutes of inactivity, excluding static addresses.

 

Switch(config)# interface FastEthernet0/5

Switch(config-if)# switchport port-security aging time 5

Switch(config-if)# switchport port-security aging type inactivity

 

**Errdisable Recovery / Detect**

When a secure port is in the error-disabled state, you can bring it out of this state by entering the global configuration command shown below, or you can manually re-enable it by entering the **shutdown** and **no shut down** interface configuration commands.

 

Switch(config)# errdisable recovery cause psecure-violation

**Auto-Recovery**

To avoid having to manually intervene every time a port-security violation forces an interface into the error-disabled state, one can enable auto-recovery for port security violations. A recovery interval is configured in seconds.

 

Switch(config)# errdisable recovery cause psecure-violation
Switch(config)# errdisable recovery interval 600

 

**Troubleshooting/Verification**

Switch# show interfaces [interface-id] switchport

-   Displays the administrative and operational status of all switching (nonrouting) ports or the specified port, including port blocking and port protection settings.

  

Switch# show port-security

-   View port security settings for the switch, including violation count, configured interfaces, and security violation actions.

 

Switch# show port-security interface [interface-id] address

-   Displays all secure MAC addresses configured on all switch interfaces or on a specified interface with aging information for each address.

  

Switch# show port-security interface [interface-id] vlan

-   Displays the number of secure MAC addresses configured per VLAN on the specified interface.

 

Switch# show port-security interface [interface-id]

-   Displays port security settings for the switch or for the specified interface, including the maximum allowed number of secure MAC addresses for each interface, the number of secure MAC addresses on the interface, the number of security violations that have occurred, and the violation mode.

 

802.1x Authentication

Friday, January 14, 2011

9:50 AM

 

**Overview**

IEEE 802.1x (dot1x) standard defines an access control and authentication protocol that prevents unauthorized hosts from connecting to a LAN through publicly accessible ports unless they are properly authenticated. The authentication server authenticates each host connected to a switch port before making available any services offered by the switch or LAN.

**Terminology**

**Extensible Authentication Protocol (EAP)**

-   A flexible authentication framework defined in RFC 3748

**EAP Over LANs (EAPOL)**

-   EAP encapsulated by 802.1X for transport across LANs

**Supplicant**

-   The device (client) attached to an access link that requests authentication by the authenticator

**Authenticator**

-   The device that controls the status of a link; typically a wired switch or wireless access point

**Authentication Server**

-   A backend server which authenticates the credentials provided by supplicants (for example, a RADIUS server)

**Guest VLAN**

-   Fallback VLAN for clients not 802.1X-capable, may be configured for limited network access.

**Restricted VLAN**

-   Fallback VLAN for clients which fail authentication

  

**Configuring 802.1x Authentication**

**Define Radius Server, used for authentication**

Switch(config)# radius-server host 10.1.1.1

Switch(config)# radius-server key $hGHg724

 

**Configure 802.1x authentication method**

Switch(config)# aaa new-model

Switch(config)# aaa authentication dot1x default group radius

-   Defines Radius server used for dot1x authentication

  

**Enable 802.1x authentication globally**

Switch(config)# dot1x system-auth-control

 

**Interface Configuration**

**Enable dot1x authentication per port**

Switch(config-if)# switchport mode access

Switch(config-if)# dot1x port-control { auto | force-authorized | force-unauthorized }

 

**auto**

-   Supplicants must authenticate to gain access

**force-authorized**

-   Port will always remain in authorized state (default)

**force-unauthorized**

-   Always unauthorized; authentication attempts are ignored

 

**Configure Host Mode**

Switch(config-if)# authentication host-mode [multi-auth | multi-domain | multi-host | single-host]

**OR**

Switch(config-if)# dot1x host-mode { multi-domain | multi-host | single-host }

 

**multi-auth**

-   Allow one client on the voice VLAN and multiple authenticated clients on the data VLAN.

**multi-host**

-   Allow multiple hosts on an 802.1x-authorized port after a single host has been authenticated.

**multi-domain**

-   Allow both a host and a voice device, such as an IP phone (Cisco or non-Cisco), to be authenticated on an 802.1x-authorized port.

**single-host**

-   Allow a single host (client) on an 802.1x-authorized port.

  

**NOTE:** Mutli-auth option is only available under the first shown syntax.

 

**Configure Maximum Authentication Attempts**

Switch(config-if)# dot1x max-reauth-req 5

 

**Enable Periodic Reauthentication**

Switch(config-if)# dot1x reauthentication

-   Using "no" form of this command will disable reauthentication

 

**Configure Guest VLAN**

Switch(config-if)# dot1x guest-vlan 99

 

**Configure Restricted VLAN**

Switch(config-if)# dot1x auth-fail vlan 123

Switch(config-if)# dot1x auth-fail max-attempts 5

 

**Troubleshooting/Verification**

Switch# show dot1x [statistics]

-   Displays 802.1x configuration, optional *statistics* keyword can be added for additional details.

  

Switch# dot1x re-authenticate interface *[interface-id]*

-   Manually allow re-authentication to the client connected to a specified interface.

  

Switch# dot1x test eapol-capable interface [interface-id]

-   Checks to see if ports are EAPOL capable

 

Storm Control

Friday, January 14, 2011

9:50 AM

 

**Overview**

Storm control prevents traffic on a LAN from being disrupted by a broadcast, multicast, or unicast storm on one of the physical interfaces. A LAN storm occurs when packets flood the LAN, creating excessive traffic and degrading network performance. Errors in the protocol-stack implementation or in the network configuration can cause a storm.

Storm control (or traffic suppression) monitors packets passing from an interface to the switching bus and determines if the packet is unicast, multicast, or broadcast. The switch counts the number of packets of a specified type received within the 1-second time interval and compares the measurement with a predefined suppression-level threshold.

Storm control uses one of these methods to measure traffic activity:

 

-   Bandwidth as a percentage of the total available bandwidth of the port that can be used by the broadcast, multicast, or unicast traffic

  

-   Traffic rate in packets per second at which broadcast, multicast, or unicast packets are received (Cisco IOS Release 12.2(25)SE or later)

 

-   Traffic rate in bits per second at which broadcast, multicast, or unicast packets are received (Cisco IOS Release 12.2(25)SE or later)

 

With each method, the port blocks traffic when the rising threshold is reached. The port remains blocked until the traffic rate drops below the falling threshold (if one is specified) and then resumes normal forwarding. If the falling suppression level is not specified, the switch blocks all traffic until the traffic rate drops below the rising suppression level. In general, the higher the level, the less effective the protection against broadcast storms.

 

**Note:** When the storm control threshold for multicast traffic is reached, all multicast traffic except control traffic, such as bridge protocol data unit (BDPU) and Cisco Discovery Protocol (CDP) frames, are blocked. However, the switch does not differentiate between routing updates, such as OSPF, and regular multicast data traffic, so both types of traffic are blocked.

 

**Storm Control Configuration using Bandwidth**

Switch(config)# interface fa0/1

Switch(config-if)# storm-control { broadcast | multicast | unicast } level {level [level-low]

 

**Example
**Switch(config-if)# storm-control broadcast level 75 65

-   If broadcast traffic reaches the threshold of 75% of the total bandwidth it will drop broadcast messages and it won't resume forwarding until broadcasts are below 65% (low-level) of the total bandwidth

  

**Storm Control Configuration using Bits Per Second (Bps)**

Switch(config)# interface fa0/1

Switch(config-if)# storm-control { broadcast | multicast | unicast } level bps { bps [level-low] }

 

**Example**
Switch(config-if)# storm-control multicast level bps 1000000 800000

-   If unicast traffic reaches the threshold of 1,000,000 bits per second it will drop multicast messages and it won't resume forwarding until multicasts are below 800,000 (low-level) bits per second.**
 ** 

**Storm Control Configuration using Packets Per Second (Pps)**

Switch(config)# interface fa0/1

Switch(config-if)# storm-control { broadcast | multicast | unicast } level pps { pps [level-low] }

 

**Example**
Switch(config-if)# storm-control unicast level pps 100000 80000

-   If unicast traffic reaches the threshold of 100,000 packets per second it will drop unicast messages and it won't resume forwarding until unicasts are below 80,000 (low-level) packets per second.

  

**Storm Control Actions**

Specify the action to be taken when a storm is detected. The default is to filter out the traffic and not to send traps.

 

-   Select the **shutdown** keyword to error-disable the port during a storm.

-   Select the **trap** keyword to generate an SNMP trap when a storm is detected.

  

Switch(config)# storm-control action { shutdown | trap }

 

**Troubleshooting/Verification**

Switch# show storm-control [interface-id] {broadcast | multicast | unicast }

-   Verify the storm control suppression levels set on the interface for the specified traffic type. If you do not enter a traffic type, broadcast storm control settings are displayed.

 

DHCP Snooping

Friday, January 14, 2011

9:50 AM

 

**Overview**

DHCP snooping is a feature which allows a Catalyst Switch to inspect DHCP traffic traversing a layer two segment and track which IP addresses have been assigned to hosts on which switch ports. This information can be handy for general troubleshooting, but it was designed specifically to aid two other features: IP source guard and dynamic ARP inspection. These features help to mitigate IP address spoofing at the layer two access edge.

 

DHCP snooping is a DHCP security feature that provides network security by filtering untrusted DHCP messages and by building and maintaining a DHCP snooping binding table. An untrusted message is a message that is received from outside the network or firewall that can cause traffic attacks within your network.

 

The DHCP snooping binding table contains the MAC address, IP address, lease time, binding type, VLAN number, and interface information that corresponds to the local untrusted interfaces of a switch; it does not contain information regarding hosts interconnected with a trusted interface. An untrusted interface is an interface that is configured to receive messages from outside the network or firewall. A trusted interface is an interface that is configured to receive only messages from within the network.

 

**Option-82 Data Insertion**

DHCP Address allocation is usually based on an IP address- either the gateway IP address or the incoming interface IP address. In some networks, you might need additional information to determine which IP address to allocate. By enabling option-82 data insertion the Cisco IOS relay agent can include additional information about itself when forwarding DHCP packets to a DHCP server. The relay agent will add the circuit identifier suboption and the remote ID suboption to the relay information option and forward this the DHCP server.

 

**Configuring DHCP Snooping & Option-82
Enable DHCP Snooping**

Switch(config)# ip dhcp snooping

 

**Enable DHCP Snooping on a VLAN or a range of VLANs**

Switch(config)# ip dhcp snooping vlan 10

 

**Enable Option-82: Data Insertion**

Switch(config)# ip dhcp snooping information option

-   Enables the switch to insert and remove DHCP relay information (option-82 field) in forwarded DHCP request messages to the DHCP server; enabled by default.

  

**Verify Source MAC Address**

Switch(config-if)# ip dhcp snooping verify mac-address

-   Verifies the source MAC address in a DHCP request received on an untrusted interface matches the client hardware address attached to that interface.

  

**DHCP Snooping Interface Configuration**

**Configure the interface as trusted or untrusted**

Switch(config-if)# ip dhcp snooping trust

-   You can use the **no** keyword to configure an interface to receive messages from an untrusted client. The default is untrusted. Trust interfaces typically are uplink interfaces that have a known destination towards the DHCP server.

* *

**Configure the number of DHCP packets per second than an interface can receive**

Switch(config-if)# ip dhcp snooping rate

-   The range is 1 to 4294967294. Default: no rate limit configured.

  

**NOTE:** Recommended untrusted rate limit of no more than 100 packets per second. Normally, the rate limit applies to untrusted interfaces. If you configure rate limiting for trusted interfaces, you will need to adjust.
**
Sample DHCP Snooping Configuration**

This example shows how to enable DHCP snooping and on VLAN 10 with Option-82 and to configure a rate limit of 100 packets per second on a specific untrusted interface and checks to see if it had a valid source MAC Address.

 

Switch(config)# ip dhcp snooping

Switch(config)# ip dhcp snooping vlan 10

Switch(config)# ip dhcp snooping information option

Switch(config)# interface gigabitethernet0/1
Switch(config-if)# ip dhcp snooping verify mac-address

Switch(config-if)# ip dhcp snooping limit rate 100

 

**Disable DHCP Snooping and Option-82
Globally disable DHCP Snooping**

Switch(config)# no ip dhcp snooping

**Disable DHCP Snooping on a VLAN or range of VLANs**

Switch(config)# no ip dhcp snooping vlan *[vlan-id]*

**Disable Option-82 Field**

Switch(config)# no ip dhcp snooping information option

 

**Troubleshooting/Verification**

Switch# show ip dhcp snooping

-   Display DHCP snooping configuration

 

Switch# show ip dhcp snooping binding

-   Displays the DHCP snooping binding entries for untrusted interfaces.

 

IP Source-Guard

Friday, January 14, 2011

9:51 AM

 

**Overview**

Dynamic ARP inspection (DAI) determines the validity of an ARP packet. This feature prevents attacks on the switch by not relaying invalid ARP requests and responses to other ports in the same VLAN.
 

When DHCP snooping is enabled, a switch maintains a database of the DHCP addresses assigned to the hosts connected to each access port. IP source guard references this database when a packet is received on any of these interfaces and compares the source address to the assigned address listed in the database. If the source address differs from the "allowed" address, the packet is assumed to spoofed and is discarded.

 

If DHCP isn't available or in use on a subnet, static IP bindings can be manually configured per access port to achieve the same effect. The following topology illustrates the lab on which this is being demonstrated.

 

 

![](''/media/image9.png){width="3.21875in" height="2.71875in"}

*
Image provided by [Packetlife.net](http://packetlife.net/)*

 

**IP Source Guard Configuration**

Two options are available for IP Source Guard, you can either filter by the hosts IP or a hosts IP and MAC address.
 

**Enable IP Source guard (IP Source Filtering)**

Switch(config)# interface fastethernet0/10

Switch(config-if)# ip verify source

-   Enables IP source guard by Source IP; same configuration performed on FA0/20

**Enable IP Source guard (IP Source & MAC Address Filtering)**

Switch(config)# interface fastethernet0/10

Switch(config-if)# ip verify source port-security

-   Enables IP source guard by Source IP & MAC Address Filtering; same configuration performed on FA0/20

 

**NOTE:** When IP Source Guard and Port Security are in use there are two caveats:

-   The DHCP server must support option 82, or the client is not assigned an IP address.

-   The MAC address in the DHCP packet is not learned as a secure address. The MAC address of the DHCP client is learned as a secure address only when the switch receives non-DHCP data traffic.

**Enable DHCP Snooping and define VLANs**

Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10

 

**Define static IP source binding (Only required when DHCP is not available)**

Switch(config)# ip source binding 001d.60b3.0add vlan 10 10.0.0.10 interface f0/10
Switch(config)# ip source binding 0023.7d00.d0a8 vlan 10 10.0.0.20 interface f0/20

 

**NOTE:** Static bindings must be created to populate the binding table, when DHCP is not in use the switch has no way of knowing the IP address of end stations without static entries.

 

**Troubleshooting/Verification**

Switch# show ip source binding

-   Display the IP source bindings on the switch, on a specific VLAN, or on a specific interface.

  

Switch# show ip verify source

-   Display the IP source guard configuration for all interfaces or for a specific interface.

 

Dynamic ARP Inspection

Friday, January 14, 2011

9:51 AM

 

**Overview**

Dynamic ARP inspection (DAI) is a security feature that validates ARP packets in a network. It intercepts, logs, and discards ARP packets with invalid IP-to-MAC address bindings. This capability protects the network from certain man-in-the-middle attacks. Dynamic ARP inspection ensures that only valid ARP requests and responses are relayed. The switch performs these activities:

 

-   Intercepts all ARP requests and responses on untrusted ports

 

-   Verifies that each of these intercepted packets has a valid IP-to-MAC address binding before updating the local ARP cache or before forwarding the packet to the appropriate destination

 

-   Drops invalid ARP packets

 

**ARP** provides IP communication within a Layer 2 broadcast domain by mapping an IP address to a MAC address.

 

**NOTE:** Dynamic ARP inspection depends on the entries in the DHCP snooping binding database to verify IP-to-MAC address bindings in incoming ARP requests and ARP responses. Make sure to enable DHCP snooping to permit ARP packets that have dynamically assigned IP addresses.

 

**Dynamic ARP Inspection Configuration
Enable DHCP Snooping**

Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10

 

**NOTE:** Refer to DHCP Snooping configuration if necessary, after enabling DHCP snooping and trusted interfaces uplinking to the DHCP source with **ip dhcp snooping trust** the DHCP snooping table will be populate with legitimate DHCP clients.

 

**Enable DAI for a specific or range of VLANs**

Switch(config)# ip arp inspection vlan 10

 

**If connecting to another switch form a trust**

Switch(config)# ip arp inspection trust

-   A trusted interface doesn't have ARP packets inspected it simply forwards the packet to its destination.

  

**DAI Validations**

Inspects ARP packet to confirm that the defined source, destination MAC or IP address specified in the ARP packet is the same as the Ethernet header; this is checked on both the ARP requests and responses. If the ARP doesn't validate it will be dropped.

Switch(config)# ip arp inspection validate { src-mac | dest-mac | ip }

 

**Troubleshooting/Verification**

Switch# show ip arp inspection

-   Verify the dynamic ARP inspection configuration.

  

Switch# show ip arp inspection statistics

-   Check the dynamic ARP inspection statistics.

 

Switch# show ip dhcp snooping binding

-   Displays the DHCP snooping binding entries for untrusted interfaces.

 

VACLs

Friday, January 14, 2011

9:51 AM

 

**Overview**

VLAN access maps (ACLs) are the only way to control filtering within a VLAN. VLAN access maps have no direction - if you wanter to filer in a specific direction, you need to include an access control list (ACL) with specified source or destination addresses. VLAN access maps do not work on the 2960 platform, but they do work on the 3560 and the 6500 platforms.

 

**VACLs Configuration**

**Create Access List used for Filtering (IP or MAC ACL)**

Switch(config)# ip access-list extended test 1

Switch(config-ext-nacl)# permit tcp any any

-   Creates extended ACL named test1 which will permit any TCP packet from any source to travel to any destination.

 

**NOTE:** Because there is no other line in this ACL, the implicit deny statement that is part of ACLs will deny any other packets.
 

**Create Access Map (VACL)**

Switch(config)# vlan access-map DROP_TCP 10

Switch(config-access-map)# match ip address test1
Switch(config-access-map)# action drop

-   VLAN access-map drops all packets that match ACL test1, in this case all TCP packets will be dropped.

 

Switch(config-access-map)# vlan access-map DROP_TCP 20

Switch(config-access-map)# action forward

-   If the packet does not match the ACL, meaning its not TCP traffic, it will be forwarded.

**NOTE:** You can match ACLs based on IP-ACL number, IP-ACL name, MAC-Address ACL Name

 

**Apply Access Map (VACL)**

Switch(config)# vlan filter DROP_TCP vlan list 10

-   Applies VACL to VLAN10, this needs to be applied to all switches that are in VLAN10to provide consistent results. Using vlan-list you can specify a single VLAN, a range of VLANs or a list of specific VLANs.

  

**Troubleshooting/Verification**

Switch# show vlan access-map <vlan-map-name

-   Displays all VLAN access maps, optionally specify a specific VLAN access map.

  

Switch# show vlan filter

-   Displays what filters apply to all VLANs

  

Switch# show vlan filter access-map <vlan-map-name

-   Displays the filter for the specific VLAN access map

 

IP ACL

Friday, January 14, 2011

9:51 AM

 

**Overview**

The Cisco access control list (ACL) is probably the most commonly used object in the IOS. It is not only used for packet filtering (a type of firewall) but also for selecting types of traffic to be analyzed, forwarded, or influenced in some way.

 

An ACL is a sequential collection of permit and deny conditions. One by one, the switch tests packets against the conditions in an access list. The first match determines whether the switch accepts or rejects the packet. Because the switch stops testing after the first match, the order of the conditions is critical. If no conditions match, the switch denies the packet.

 

**Standard IP access lists** use source addresses for matching operations.

-   Standard IP ACLs: 1 to 99 and 1300 to 1999

 

**Extended IP access lists** use source and destination addresses for matching operations and optional protocol-type information for finer granularity of control.

-   Extended IP ACLs: 100 to 199 and 2000 to 2699

 

Configuring IP v4ACLs on the switch is the same as configuring IPv4 ACLs on other Cisco switches and routers. The process is briefly described here.

 

**Access List Syntax**

**Standard ACL**

Switch(config)# access-list standard **<number [<sequence]** {permit | deny} <source [log]

 

**Extended ACL**

Switch(config)# access-list extended **<number [<sequence]** {permit | deny} <protocol <source [<ports] <destination [<ports] **[<options]**

 

**Named Standard ACL:**

Switch(config)# ip access-list { standard | extended } *<acl-name*

Switch(config-ext-nacl)# **[<sequence]** {permit | deny} <source [log]

 

**Named Extended ACL:**

Switch(config)# ip access-list { standard | extended } *<acl-name*

Switch(config-ext-nacl)# **[<sequence]** {permit | deny} <protocol <source [<ports] <destination [<ports] **[<options]**

 

**Source/Destination Definitions**

**any** - Any address

**host <address** A single address

**<network <mask** Any address matched by the wildcard mask

 

**Port Operators**

**eq <port** Not equal to

**lt <port** Less than

**gt <port** Greater than

**range <port <port** Matches a range of port numbers

**neq <port** Not equal to

 

**Options**

**dscp <DSCP** Match the specified IP DSCP

**fragments** Check non-initial fragments

**option <option** Match the specified IP option

**precedence {0-7}** Match the specified IP precedence

**ttl <count** Match the specified IP time to live (TTL)

**
Logging**

**log** - Log ACL entry matches

**log-input -** Log matches including ingress interface and source MAC address

 

**Access List Configuration**

**Standard ACL**

Switch(config)# access-list 2 remark ***ACL BLOCK SINGLE HOST ACCESS**

-   Remarks can be added into ACLs to assist in understand the purpose of an ACL, recommended best practice when creating ACLs.

Switch (config)# access-list 1 deny host 171.69.198.102

Switch (config)# access-list 1 permit any

-   Displays standard ACL to deny access to IP host 171.69.198.102, permit access to any others

 

**Extended ACL**

Switch(config)# access-list 2 deny tcp 171.69.198.0 0.0.0.255 172.20.52.0 0.0.0.255 eq telnet
Switch(config)# access-list 2 permit tcp any any

-   Displays access list to deny Telnet access from any host in network 171.69.198.0 to any host in network 172.20.52.0 and to permit any others. (The **eq** keyword after the destination address means to test for the TCP destination port number equaling Telnet.)

  

**Apply the ACL to interfaces, terminal lines or VLAN Maps (Reference VACLs)**

**Apply to Interface**

Switch(config)# interface FastEthernet0/1

Switch(config-if)# ip access-group {*access-list-number | name*} {in | out}

-   Apply a numbered or name ACL either inbound or outbound.

 

**Apply to Terminal Line**

Switch(config)# line [console | vty] *<line-number*

Switch(config-line)# access-class <*access-list-number* {in | out}

-   Restrict incoming and outgoing connections between a particular virtual terminal line (into a device) and the addresses in an access list.

  

**Troubleshooting/Verification**

Switch# show access-lists *[number | name]*

-   Displays access list configuration.

  

Switch# show ip access-lists interface *[interface-id]*

-   Displays ACL applied to all interfaces or a specific interfaces.

 

MAC ACLs & Ethertypes

Friday, January 14, 2011

9:51 AM

 

**Overview**

You can filter non-IPv4 traffic on a VLAN or on a Layer 2 interface by using MAC addresses and named MAC extended ACLs.

MAC ACLs are ACLs that use information in the Layer 2 header of packets to filter traffic. The procedure is similar to that of configuring other extended named ACLs.

 

**EtherType** is a two-octet field in an Ethernet frame. It is used to indicate which protocol is encapsulated in the PayLoad of an Ethernet Frame.

 

**Creating a MAC-Address ACL**

Switch(config)# mac access-list extended *<acl-name*

Switch(config-ext-macl)# {**deny** | **permit**} {**any** | **host** *source MAC address | source MAC address mask*} {**any** | **host** *destination MAC address | destination MAC address mask*} [*type mask* | **lsap** *lsap mask* | **aarp** | **amber** | **dec-spanning** | **decnet-iv** | **diagnostic** | **dsm** | **etype-6000** | **etype-8042** | **lat** | **lavc-sca** | **mop-console** | **mop-dump** | **msdos** | **mumps** | **netbios** | **vines-echo** |**vines-ip** | **xns-idp |** *0-65535*] [**cos** *cos*]

 

**Notes:**

***Type mask***---An arbitrary EtherType number of a packet with Ethernet II or SNAP encapsulation in decimal, hexadecimal, or octal with optional mask of *don't care* bits applied to the EtherType before testing for a match.

 

**LSAP**---An LSAP number of a packet with IEEE 802.2 encapsulation in decimal, hexadecimal, or octal with optional mask of *don't care* bits.

 

**Non-IP Protocols:** aarp | amber | dec-spanning | decnet-iv | diagnostic | dsm | etype-6000 | etype-8042 | lat | lavc-sca | mop-console | mop-dump | msdos | mumps | netbios | vines-echo |vines-ip | xns-idp

 

**COS** ---An IEEE 802.1Q cost of service number from 0 to 7 used to set priority.

 

**Apply MAC ACL to an Interface**

After you create a MAC ACL, you can apply it to a Layer 2 interface to filter non-IP traffic coming in that interface

 

Switch(config)# interface FastEthernet0/1

Switch(config-if)# mac access-group *<acl-name* in

 

**MAC ACL Example**

This example shows how to create an access list named *mac1*, denying only EtherType DECnet Phase IV traffic, but permitting all other types of traffic. And is applied to all incoming traffic on Interface FastEthernet0/1.

 

Switch(config)# mac access-list extended mac1

Switch(config-ext-macl)# deny any any decnet-iv

Switch(config-ext-macl)# permit any any

Switch(config)# interface FastEthernet0/1

Switch(config-if)# mac access-group mac1 in

 

**Troubleshooting/Verification**

Switch# show access-lists *<acl-name*

-   Displays access list configuration.

 

Switch# show mac access-group interface *[interface-id]*

-   Display the MAC access list applied to the interface or all Layer 2 interfaces.

 

Port Protection

Friday, January 14, 2011

9:51 AM

 

**Overview**

Similar to the concept of Private VLANs which provides isolated VLANs within a VLAN. Port protection can be used to block communication with other switchports and only allow for communication from the protected ports to those who are not configured to be protected. Two protection modes exit **protect** and **block** which will be discussed in-depth within their own individual subpages.

 

Switchport Protect

Friday, January 14, 2011

9:52 AM

 

**Overview**

Protected ports ensures that there is no exchange of unicast, broadcast, or multicast traffic between ports that are also configured as protected. Any port that is not configured as protected will be accessible by protected ports.

 

Protected ports may be configured on both physical interfaces and port-channels.

 

**Protected Port Configuration**

Switch(config)# interface FastEthernet0/1

Switch(config-if)# switchport protect

-   Enables switchport in the protected mode and will only be able to communicate with unprotected ports (ports without the *switchport protect* command)

 

**Troubleshooting/Verification**

Switch# show interfaces *[interface-id]* switchport

-   Verify switchports are configured to be protected.

 

Switchport Block

Friday, January 14, 2011

9:52 AM

 

**Overview**

By default, the switch floods packets with unknown destination MAC addresses out of all ports. If unknown unicast and multicast traffic is forwarded to a protected port, there could be security issues. To prevent unknown unicast or multicast traffic from being forwarded from one port to another, you can block a port (protected or nonprotected) from flooding unknown unicast or multicast packets to other ports.

 

**Block Port Configuration**

Switch(config)# interface FastEthernet0/1

Switch(config-if)# switchport block multicast

-   Blocks flooding of multicast traffic to other interfaces.

Switch(config-if)# switchport block unicast

-   Blocks flooding of unicast traffic to other interfaces.

 

**Troubleshooting/Verification**

Switch# show interfaces *[interface-id]* switchport

-   Verify switchports are configured to be protected.
