# Ethernet Technologies Index
- [Ethernet Technologies Overview](#ethernet-technologies)
  - [Ethernet Configuration](#ethernet-configuration)
  - [Switchport](#switchport)
      - [Trunk](#trunk)
          - [Dot1q Tunneling](#dot1q-tunneling)

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