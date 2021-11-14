## Security Index
- [Local Authentication](#local-authentication)
- [SSH Secure Shell](#ssh-secure-shell)
- [AAA Authentication Authorization Accounting](#aaa-authentication-authorization-accounting)
  - [Overriding Login Security Defaults](#overriding-login-security-defaults)
- [Standard Access-Lists](#standard-access-lists)
  - [ACL Logging](#acl-logging)
- [Extended Access-Lists](#extended-access-lists)
  - [IP Options](#ip-options)
- [Time Based Access-Lists](#time-based-access-lists)
- [Dynamic Access-Lists](#dynamic-access-lists)
- [Reflexive Access-Lists](#reflexive-access-lists)
- [CBAC Context-Based Access Control](#cbac-context-based-access-control)
- [ZBF Zone Based Firewall](#zbf-zone-based-firewall)
- [RITE IP Traffic Export](#rite-ip-traffic-export)
- [IPS Intrustion Prevention System](#ips-intrusion-prevention-system)
- [Virtual Private Networks VPN](#virtual-private-networks-vpn)
  - [Remote Access VPN](#remote-access-vpn)
  - [Site to Site VPN](#site-to-site-vpn)
  - [GRE over IPsec VPN](#gre-over-ipsec-vpn)
- [Disabling Services](#disabling-services)
  - [Source Routing](#source-routing)
  - [Proxy ARP](#proxy-arp)
  - [IP Option](#ip-option)
  - [IP Redirects](#ip-redirects)
  - [IP Unreachable](#ip-unreachable)
  - [Cisco Discovery Protocol CDP](#cisco-discovery-protocol-cdp)
- [TCP Intercept](#tcp-intercept)
- [Unicast Reverse Path Forwarding uRPF](#unicast-reverse-path-forwarding-urpf)

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Local Authentication

Cisco IOS can be configured to require simple password protection for each of the three methods to access user mode. To do so, the **login** command is used to tell Cisco IOS to prompt the user for a password, and the **password** subcommand defines the password. Depending on the configuration mode implies for which of the three access methods the password should be required (console, aux, telnet).

 
```
Router>   \<-- User EXEC Mode
Router#   \<-- Privileged EXEC mode
```

**Local Authentication Configuration**

**Console**

```
Router(config)#line console 0
Router(config-line)#login
```

-   Login disabled on line 0, until \'password\' is set.

```
Router(config-line)#password *\<password>*
```
 

**Telnet**

```
Router(config)#line vty 0 15
```

-   0 - 15 means 16 telnet connections can be made to the device at a time.

```
Router(config-line)#login
```

-   Login disabled on line 0, until \'password\' is set.

```
Router(config-line)#password *\<password>*
```
 

**Aux**

```
Router(config)#line aux 0
Router(config-line)#login
```

-   Login disabled on line 0, until \'password\' is set.

```
Router(config-line)#password *\<password>*
```
 

**NOTE:** These passwords are stored as clear text in the configuration, butt hey can be encrypted by including the **service password-encryption** global command.

 

**Enable Password\
**Another password can be applied to the Cisco IOS to grant privileged exec mode. There are two commands available that achieve the same results, except one of them is more secure and provides enhanced encryption in the running-configuration. Again service password-encryption can be used to provide protection to the basic unencrypted enable password.

 

**No encryption**

```
Router(config)# enable password *\<password>*
```
 

**Encrypted in MD5-Hashed value**

```
Router(config)# enable secret *\<password>*
```
 

**User Accounts**

This allows for local authentication using a defined username and password. Encryption is done in the exact same manner as the enable password.

 
**No encryption**

```
Router(config)# username *\<name>* password *\<password>*

```
 

**Encrypted in MD5-Hashed value**

```
Router(config)# username *name>* secret *\<password>*

```
 

**NOTE:** Running-configuration will show a number behind the scrambled password, this is used to signify the encryption level.

-   7 for service password-encryption

-   5 for MD5-hased value 

**Troubleshooting/Verification**

Router# show run

-   Displays running-configuration; used to confirm encryption techniques.

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## SSH Secure Shell

Telnet has long been used to manage network devices; however, Telnet traffic is sent in clear text. Anyone able to sniff that traffic would see your password and any other information sent during the Telnet session. Secure Shell (SSH) is a much more secure way to manage your routers and switches. It is a client/server protocol that encrypts the traffic in and out through the vty ports.

 

Cisco routers and switches can act as SSH clients by default; but must be configured to be SSH servers. That is, they can use SSH when connection *to* another device, but require configuration before allowing devices to connect via SSH to the them. They also require some method of authenticating the client. This can be either a local username and password, or authentication with AAA server.

 

There are two versions of SSH. SSH Version 2 is an IETF standard that is more secure than version 1. Version 1 is more vulnerable to man-in-the-middle attacks, for instance. Cisco devices support both types of connections, but you can specify which version to use.

 

**Secure Hell (SSH) Configuration**

Telnet is enabled by default, but configuring even a basic SSH server requires several steps:

 

**Configure a hostname**

```
NoName(config)# hostname Router
```

**Configure domain name**

```
Router(config)# ip domain-name *\<domain.name>*\
```

**Tell the router or switch to generate the Rivest, Shamir, and Adelman (RSA) keys that will be used to encrypt the session.**

```
Router(config)# crypto key generate rsa
```

-   When it prompts for the modulus size, choose a range between 360 and 2048. Recommended: 1024.

 

**Specify the SSH version, if you want to use version 2.**

```
Router(config)# ip ssh version \[1 \| **2**\]
```
 

**Disable Telnet on the VTY lines.**
```
Router(config)#line vty 0 4\
Router(config-line)#transport input none
```
 

**Enable SSH on the VTY lines.**

```
Router(config)#line vty 0 4
Router(config-line)# transport input ssh
Router(config-line)# transport output ssh
```

-   This command allows for you to allow access telnet or ssh from the device itself to other connecting devices.

**\
Create Local User Account (or use AAA)\

```
**Router(config)# username \<username> privilege 15 password 0 \<password>
```

**Troubleshooting/Verification**\

```
Router# show ip ssh
```

-   Displays current SSH settings

```
Router# show ssh
```

-   Displays current SSH sessions/connections.

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## AAA Authentication Authorization Accounting

The term authentication, authorization, and accounting (AAA) refers to a variety of common security features. The strongest method to protect the CLI is to use a TACACS+ or RADIUS server.


**RADIUS**

-   Encrypts password only

-   UDP Protocol

-   Ports: 1812/1645

-   RFC 2865

>  

**TACACS+**

-   Encrypts entire payload

-   TCP Protocol

-   Ports: 49/49

-   Proprietary

>  

**Authentication:** Authenticates a user to confirm they are able to access a device or service.\
**Authorization:** Specifies what kind of authorization levels or permissions available once logged in.\
**Accounting:** Logs users actions once logged in or failed log in attempts.

 

**AAA Configuration**

1.  Enable AAA

2.  Create local database user OR configure RADIUS/TACACS+

3.  Define Authentication List

4.  Apply Authentication List (VTY, Console Lines)

 
```
**Enable AAA\
**Router(config)# aaa new-model
```

-   Enables AAA and removes previous authentication methods until the next command is entered.  

**Create local database user**

```
Router(config)# username *\<username>* privilege 15 password 0 *\<password>*
```

-   As shown in the basic configuration guide, this adds a new user to the local database for authentication. This is required if a RADIUS or TACACS+ server is not in use.

 

**Define RADIUS or TACACS+** (Optional)

```
Router(config)# radius-server host x.x.x.x \[auth-port ***xx***\] \[acct-port ***xx***\]
```

-   Optionally define ports other than the defaults for RADIUS authentication and accounting.

```
Router(config)# radius-server key *\<key>*
Router(config)# tacacs-server host x.x.x.x
Router(config)# tacacs-server key *\<key>*
```

**Create authentication list**

```
Router(config)# aaa authentication login default *\[authentication methods\]*
```

-   Define list of authentication methods that should be queried in order.

 

**Example**

```
Router(config)# aaa authentication login default group radius local
```

-   This configuration specifies for a user to be first authenticated using a Radius server. If the server does not respond or is offline the user will be authenticated using the local database.


**Authentication Methods**

-   group Radius - Use configured Radius servers

-   group tacacs+ - Use the configured TACACS+ servers

-   group *\<name> -* Use a defined group of either RADIUS or TACACS+ servers

-   enable - Use the enable password, based on the enable secret or enable password commands.

-   line - Use the password defined by the password command in line configuration

-   local - Use username commands in the local configuration; treats the username as case insensitive, but the password as case sensitive.

-   local-case - Use username commands in the local configuration; treats both the username and password as case sensitive.

-   none - No authentication required, user is automatically authenticated.

 

**Apply authentication lists to the console and VTY lines**

```
Router(config)# line con 0
Router(config-line)# login authentication default
Router(config)# line vty 0 4
Router(config-line)# login authentication default
```

-   The above commands applies the AAA authentication method to the interfaces. Users who not attempt to access the console port or telnet/SSH lines will be prompted for a user a password.
 
**Optional Parameters**

**RADIUS**

```
Router(config)# radius-server retransmit *\<retries>*
```

-   Specifies the number of times the router transmits each RADIUS request to the server before giving up (the default is three).

 
```
Router(config)# radius-server timeout *\<seconds>*
```

-   Specifies the number of seconds a router waits for a reply from a RADIUS request before retransmitting the request.

```
Router(config)# radius-server deadtime *\<minutes>*
```

-   Specifies the number of minutes a RADIUS server, which is not responding to authentication requests, is passed over by requests for RADIUS authentication.

 

**TACACS+**

```
Router(config)# tacacs-server timeout
```

-   Specifies the number of seconds a router waits for a reply from a RADIUS request before retransmitting the request.

 
```
Router (config)#tacacs-server last-resort {password \| succeed}
```

-   Defines TACACS action if no server responds

-   Password: The \'enable\' password must be provided

-   Succeed: Access to privileged level is granted
  

**Troubleshooting/Verification**

```
Router#show aaa local user locked
```

-   Displays a list of all locked-out users

```
Router#clear aaa local user fail-attempts *\[user1\]*
```

-   Clears all unsuccessful login attempts for *user1*

```
Router#show aaa sessions
```

-   Displays total AAA sessions since last reload
  

```
Router#show aaa user *\[ID/all\]*
```

-   Displays all AAA users or specific users based on local ID
  

```
Router#show users
```

-   Displays the users who are logged in
  

```
Router#show tacacs
```

-   Displays the TACACS+ server configuration for all servers

```
Router#show radius-server group
```

-   Displays RADIUS server groups

```
Router#show radius statistics x.x.x.x
```

-   Displays RADIUS statistics for a specific server

---------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Overriding Login Security Defaults

The console, vty, and aux (routers only) lines can override the use of the default login authentication methods in favor of using AAA.

**Overriding the Default Login Authentication Method**

Configured in a similar manner to the authentication methods list as described within [AAA].

 

**Authentication Methods**

-   group Radius - Use configured Radius servers

-   group tacacs+ - Use the configured TACACS+ servers

-   group *\<name> -* Use a defined group of either RADIUS or TACACS+ servers

-   enable - Use the enable password, based on the enable secret or enable password commands.

-   line - Use the password defined by the password command in line configuration

-   local - Use username commands in the local configuration; treats the username as case insensitive, but the password as case sensitive.

-   local-case - Use username commands in the local configuration; treats both the username and password as case sensitive.

-   none - No authentication required, user is automatically authenticated.

 

**Syntax**

```
Router(config)# aaa authentication login *\<name> \[authentication methods\]*
```
 

**Console, VTY, Aux Examples**

**Create name authentication method list (for console)**

```
Router(config)# aaa authentication login *for-console* group radius line
```

-   Try the RADIUS servers, and use the line password if no response.

>  

**Apply to line console**

```
Router(config)# line con 0
Router(config-line)# login authentication *for-console*
```
 

**Create name authentication method list (for vty)**

```
Router(config)# aaa authentication login *for-vty* group radius local
```

-   Try the RADIUS servers, and use the local usernames/passwords if no response.

>  

**Apply to vty lines**

```
Router(config)# line vty 0 4
Router(config-line)# login authentication *for-vty*
```
 

**Create name authentication method list (for aux)**

```
Router(config)# aaa authentication login *for-aux* group radius radius
```

-   Try the RADIUS servers, and do not authenticate if no response.

>  

**Apply to vty lines**

```
Router(config)# line aux 0
Router(config-line)# login authentication *for-aux*
```

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Standard Access-Lists

Standard Access Control Lists (ACL) are Cisco IOS-based commands used to filter packets on Cisco routers based on the source IP Address of the packet. Access-Lists are not only used for filtering, but can also be used to identify traffic to be manipulated or used in many other technologies within the Cisco IOS (e.g. NAT, QoS etc.)


Cisco IOS processes the Access Control Entries (ACEs) of an ACL sequentially, either permitting or denying a packet based on the first ACE matched by that packet in the ACL. For an individual ACE, all the configured values must match before the ACE is considered a match.


**Wildcard Masks**

A wildcard mask can be thought of as a subnet mask, with ones and zeros inverted; for example, a wildcard mask of 0.0.0.255 corresponds to a subnet mask of 255.255.255.0. A wildcard mask is usually used in combination with an IP address. For example, in a standard ACL, a statement like the following:

 
```
Router(config)# access-list 10 permit 10.0.3.0 0.0.0.255
```

-   allows data from subnet 10.0.3.0/24 to pass, that is, the first three bytes must match exactly, whereas all the bits in the fourth octet can take on any value.

>  

However, any bits can be marked as \"don\'t care\". For example, a wildcard mask of 0.0.0.254 (binary equivalent = 00000000.00000000.00000000.11111110 in an ACL might accept (or deny) all even-numbered IP addresses in a specific network.

 

Wildcard masks are used in situations where the subnet mask may not apply. For example, in an ACL, two affected hosts may fall in different subnets, but the use of a wildcard mask can group the two together.**\
** 

**Source/Destination Definitions**

**any** Any address

**host *\<address>*** A single address

***\<network> \<mask>*** Any address matched by the wildcard mask

 

**Standard ACL Configuration**

**Standard ACL Number:** 1-99 (1300-1999)

 

**Legacy Syntax**

```
Router(config)# access-list \<number> {permit \| deny} *\<source>* \[log\]
```

**Modern Syntax**

-   A number of advantages are presented by using the modern syntax.

-   Identify ACL based on a *name* instead of a *number,* Referred to as a *Named ACL*

-   Use sequence numbers, in the past the entire ACL had to be recreated because new entries were added onto the end of the ACL with the old syntax. In the new syntax a sequence numbers can be used to place entries in between other entries and not automatically prepended to the end of the ACL.

>  

```
Router(config)# ip access-list standard {\<number> \| \<name>} \[\<sequence>\] {permit \| deny} *\<source>* \[log\]
```
 

**Apply ACL to interface**

```
Router(config)# interface \[interface\]
Router(config-if)# ip access-group {*\<number>* \| *\<name>*} {in \| out}
```

-   You can only apply a single ACL, one for both inbound and outbound per interface.

>  

**Apply to Line Interface**

-   Used to control access to VTY lines using a standard or extended ACL.

```
Router(config-line)# access-class \[*\<number>* \| *\<name>*\] \[in \| out\]
```

>  

**ACL Remarks**

Comments/Remarks make Access-Lists easier to understand and can often explain meticulous to read ACL configurations.

**Syntax**

```
Router(config)# ip access-list {standard \| extended} \<access-list-name> remark *\"comment goes here, without quotes\"*
```

**Troubleshooting/Verification

```
Router# show access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

>  

```
Router(config)# ip access-list resequence *\<acl-name>* starting-sequence-number *\<increment>*
```

-   Redefine sequence numbers for a crowded ACL

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## ACL Logging

Logging-enabled access control lists (ACLs) provide insight into traffic as it traverses the network or is dropped by network devices.

 

The **log** and **log-input** options apply to an individual ACE and cause packets that match the ACE to be logged. The **log-input** option enables logging of the ingress interface and source MAC address in addition to the packet\'s source and destination IP addresses and ports.

 

**ACL Logging Configuration**

```
Router(config)# ip access-list standard LOG-TEST deny any **\[*log \| log-input*\]**
```

-   Enable Log or Log-input

>  

**Verification**

```
Router(config)# logging on
```

-   Enable logging to all enabled destinations (e.g. console, vty etc.)

-   CPU intensive, but will display in real-time log matches of an ACL.

>  

```
Router# show ip access-lists
```

-   Displays access-lists and if log command is used the number of matches to the specific ACL entry.

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Extended Access Lists

As opposed to Standard ACLs, Extended ACLs have the ability to filter packets based on source and destination IP addresses and provide a higher level of granularity by having the ability to filter by protocol, ports and [IP Options].

 

**Source/Destination Definitions**

**any** Any address

**host *\<address>*** A single address

***\<network> \<mask>*** Any address matched by the wildcard mask

 

**TCP/UDP Port Definitions**

Extended ACLs options have the ability to match based on port.

**eq** *\<port>*

-   Equal to

**lt** *\<port>*

-   Less than

**range** *\<port> \<port>*

-   Matches a range of port numbers

**neq** *\<port>*

-   Not equal to

**gt** *\<port>*

-   Greater than

 

**Extended ACL Configuration**

**Extended ACL Number:** 100-199 (2000-2699)

 

**Legacy Syntax**

```
Router(config)# access-list \<number> {permit \| deny} \<protocol> \<source> \[\<ports>\] \<destination> \[\<ports>\]
```
 

**Modern Syntax**

-   A number of advantages are presented by using the modern syntax.

-   Identify ACL based on a *name* instead of a *number.* Referred to as a *Named ACL*

-   Use sequence numbers, in the past the entire ACL had to be recreated because new entries were added onto the end of the ACL with the old syntax. In the new syntax a sequence numbers can be used to place entries in between other entries and not automatically prepended to the end the ACL.

>  

```
Router(config)# ip access-list extended {\<number> \| \<name>} \[\<sequence>\] {permit \| deny} \<protocol> \<source> \[\<ports>\] \<destination> \[\<ports>\] \[\<options>\]
```
 

**Apply ACL to interface**

```
Router(config)# interface \[interface\]
Router(config-if)# ip access-group {*\<number>* \| *\<name>*} {in \| out}
```

-   You can only apply a single ACL, one for both inbound and outbound per interface.

 

**Troubleshooting/Verification\

```
Router# show access-lists *\<number>*
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

>  

```
Router(config)# ip access-list resequence *\<acl-name>* starting-sequence-number *\<increment>*
```

-   Redefine sequence numbers for a crowded ACL

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Options

IP uses four key mechanisms in providing its service: Type of Service, Time to Live, Options, and Header Checksum.

The Options, commonly referred to as IP Options, provide for control functions that are required in some situations but unnecessary for the most common communications. IP Options include provisions for time stamps, security, and special routing.

 

**Common IP Options**

  **dscp** *\<DSCP>*       Match the specified IP DSCP

  **Fragments**            Check non-initial fragments

  **option** *\<option>*   Match the specified IP option

  **precedence** *{0-7}*   Match the specified IP precedence

  **ttl** *\<count>*       Match the specified IP time to live (TTL)

>  

**TCP Options**


  ack                Match ACK flag

  fin                Match FIN flag

  psh                Match PSH flag

  rst                Match RST flag

  rst                Match RST flag

  urg                Match URG flag

  established        Match packets in an established session.

**IP Options Configuration**

The ACL Support for Filtering IP Options feature can be used only with named, extended ACLs.

```
Router(config)# ip access-list extended {\<number> \| \<name>} \[\<sequence>\] {permit \| deny} \<protocol> \<source> \[\<ports>\] \<destination> \[\<ports>\] \[\<options>\]
```
 

**Example**

```
Router(config)# ip access-list extended mylist1
Router(config-ext-nacl)# deny ip any any option traceroute
```

-   Denies any packet that contains the traceroute IP option

>  

**Troubleshooting/Verification\

```
Router# show access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

>  

```
Router(config)# ip access-list resequence *\<acl-name>* starting-sequence-number *\<increment>*
```

-   Redefine sequence numbers for a crowded ACL

---------------------------------------------------------------------------------------------------------------------------------------------------------------- 

## Time Based Access Lists

Time Base Access-lists provide a method to enable an access-list only during specified times.

**NOTE:** IOS clock must be up to date in order for Time Based Access-Lists to work properly.

 

**Time Based Access-Lists Configuration**

**Define name and time/date**

```
Router(config)#time-range *\<name>*
Router(config-time-range)# absolute \[**start** *time date*\] \[**end** *time date*\]
```

-   Absolute time/date; used to specify a specific time and date that an ACL will be applied.

```
Router(config-time-range)# periodic *days-of-the-week* hh:mm **to** *days-of-the-week* hh:mm
```

-   Periodic time/date; used to specify a periodic time and date that an ACL will be applied.

>  

**Reference the Time**

In order for a time range to be applied, you must reference it by name in a feature that can implement time ranges

 
```
Router(config)# ip acess-list extended *time-range-specified*
Router(config-ext-nacl)# {permit \| deny} \<protocol> \<source> \[\<ports>\] \<destination> \[\<ports>\] **time-range** *\<name>*
```
 

**Troubleshooting/Verification\

```
Router# show access-lists *\<number>*
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

>  

```
Router(config)# ip access-list resequence *\<acl-name>* starting-sequence-number *\<increment>*
```

-   Redefine sequence numbers for a crowded ACL

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Dynamic Access-Lists

Access lists are normally used to filter traffic at the packet level. In other words, when a connection is attempted through a router interface, packet headers are inspected for prohibited IP addresses or application port numbers, and traffic is passed or blocked.

 

To review here, such access lists are *extended* in that they can filter based on network application port numbers instead of just addresses. They're also called *static* extended access lists, because the permit and deny commands are blindly enforced, regardless of the user. To make an exception for a particular person, an administrator would need to go modify an ACL manually.

 

Dynamic access lists are configured using so-called lock-and-key commands. By employing these, a user who would otherwise be blocked can be granted temporary access to a network or subnet via a Telnet session over the Internet.

 

The Telnet session is opened to a router configured for lock-and-key. The dynamic access list prompts the user for authentication information. As with other user-based security protocols, lock-and-key can be configured to check against a user database on the router itself (local), or against a user database maintained on a TACACS+ or RADIUS server. If authenticated, the user is

automatically logged out of the Telnet session and can start a normal application such as a browser.

 

**Lock-and-Key Configuration Using a Local User Database**

The following sequence of code snippets shows how lock-and-key could be configured on a router using a locally maintained user authentication file.

 

**Create Dynamic ACL**

```
Router(config)# access-list 103 permit tcp any host 209.198.207.2 eq telnet
Router(config)# access-list 103 dynamic TEMP timeout 60 permit ip any any
```
 

In the following statement, the first entry of access list 103 allows only Telnet connections into the router. The second entry of access list 103 is ignored until lock-and-key is triggered whenever a Telnet connection has been established in the router. The keyword **dynamic** defines access list 103 as a dynamic (lock-and-key) list.

 

This is the key juncture. If so configured, an attempted Telnet connection to the router causes it to check against its local user database to see if the user and password are valid for lock-and-key access to the router. If validated, the **timeout 60 permit ip any any** statement gives the user 60 minutes to use the router as a connection between any two IP addresses.

 

**NOTE:** The keyword **in** specifies that access control be applied only to inbound connections (lock-and-key can also be used to restrict outbound connections).\
 

**Apply Dynamic ACL to Interface**

```
Router(config)# interface fastethernet0/0
Router(config)# ip address 209.198.207.2 255.255.255.0
Router(config-if)# ip access-group 103 in
```
 

Finally, an **autocommand** statement creates a temporary inbound access list entry (named TEMP in the previous dynamic ACL configuration) at the network interface FastEthernet0/0 and line 0 on the router. The temporary access list entry will time out after five minutes.

 

**Apply to line Interface**

```
Router(config)# line vty 0
Router(config-line)# login local
```

-   To configure the use of an authentication server, reference [Overriding Login Security Defaults].

Router(config-line)# autocommand access-enable timeout 5

 

**NOTE:** The temporary access list entry isn't automatically deleted when the user terminates the session. It will remain configured until the timeout period expires.

 

Dynamic access lists can also be configured to authenticate users against a user database maintained on either a TACACS+ or RADIUS server. This, in effect, turns a router into an access server through which a user can gain entry into an internetwork, but only by logging in via a Telnet session.

 

**Troubleshooting/Verification**

```
Router# clear access-template \[*access-list-number* \| *name*\] \[*dynamic-name*\] \[*source*\] \[*destination*\]
```

-   Deletes specified dynamic access-lists

 

**Troubleshooting/Verification\

```
Router# show access-lists *\<number>*
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

>  

```
Router(config)# ip access-list resequence *\<acl-name>* starting-sequence-number *\<increment>*
```

-   Redefine sequence numbers for a crowded ACL

 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Reflexive Access-Lists

Reflexive ACLs allow IP packets to be filtered based on upper-layer session information. They are generally used to allow outbound traffic and to limit inbound traffic in response to sessions that originate inside the router.

 

Reflexive ACLs can be defined only with extended named IP ACLs. They cannot be defined with numbered or standard named IP ACLs, or with other protocol ACLs. Reflexive ACLs can be used in conjunction with other standard and static extended ACLs.

 

A typical approach to network perimeter security is to allow outbound traffic not explicitly denied, and to deny inbound traffic unless it is explicitly allowed. Although simple in concept, this approach requires significant considerations regarding the return path of internally initiated sessions. Consider the following scenario:

![reflexiveaccesslist1.png](/Images/reflexiveaccesslist1.png)

**Scenario**

Clients resides on the secure 192.168.0.0/24 subnet, which is connected to the Internet by R1. We can place both inbound and outbound access lists on R1\'s F0/1 interface to restrict communication between our internal network and the Internet.

 

For the sake of an example, let\'s assume we want to allow web traffic from the client to web servers on the Internet. We can leave outbound traffic (from client to server) unrestricted by simply not including an ACL, but how would we restrict return traffic from the Internet? Obviously we can\'t simply deny all traffic, or nothing would work. Nor can we allow all traffic, as it would leave the secure subnet exposed.

 

We also can\'t simply allow all TCP traffic with a source port of 80, as an attacker could easily send malicious traffic using 80 as his source port. And we can\'t restrict inbound traffic to certain source IP addresses, as we\'d have to create a new entry for every server we want to access on the Internet. We could allow inbound traffic to the client\'s source port, but this is typically a randomly-chosen high port number which can\'t be practically matched with static configuration. But what if we could record and match each source address/port pair *automatically*?

 

To employ reflexive ACLs, three access lists are actually needed: one for inbound traffic, one for outbound traffic, and one (the reflexive ACL) to keep track of dynamic entries. Outbound traffic matched in the outbound ACL is *reflected* to the reflexive ACL; that is, the source and destination addresses and ports are swapped and the entry is recorded in the reflexive ACL with an expiration timer. Traffic in the other direction is matched against the inbound ACL, which in turn evaluates the entries in the reflexive ACL.

 

**Reflexive ACL Configuration**

**Create an outbound ACL**

```
Router(config)# ip access-list extended *Egress*\
Router(config-ext-nacl)# permit ip any any reflect *Mirror*\
```

**Apply to interface (outbound)**

```
Router(config-ext-nacl)# interface f0/1
Router(config-if)# ip access-group out *Egress*
```
 

Any packet matched by Egress will be reflected into our reflexive ACL, named Mirror. Since Egress matches all IP traffic, we reflect entries for TCP, UDP, and ICMP. If we wanted, we could have specified only TCP, for example, to only match TCP traffic. While TCP sessions are relatively simple to track, IOS can also roughly track UDP and ICMP \"sessions,\" even though these aren\'t true session-oriented protocols.

 

![reflexiveaccesslist2.png](/Images/reflexiveaccesslist2.png)

 

Now when a client in 192.168.0.0/24 initiates a TCP session to a server on the Internet we can see a reflected entry is created in Mirror:

 

Router# show ip access-lists *Mirror*\
*Reflexive IP access list Mirror\
permit tcp host 209.20.64.81 eq www host 192.168.0.123 eq 62839 (7 matches) (time left 294*)

 

**Create an inbound ACL**

We\'ll create an inbound ACL named Ingress to evaluate Mirror, and apply it inbound to FastEthernet0/1:

>  

```
Router(config)# ip access-list extended Ingress\
Router(config-ext-nacl)# evaluate Mirror\
Router(config-ext-nacl)# interface f0/1\
Router(config-if)# ip access-group in Ingress
```
 

![reflexiveaccesslist3.png](/Images/reflexiveaccesslist3.png)

 

 

 

Now packets inbound on FastEthernet0/1 are only allowed in if they are permitted by Ingress, which is essentially just a reference to Mirror. Note, however, that we are free to add normal entries to Ingress both before and after the evaluate statement if we want. With all three components in place, we can see the static outbound entries (Egress), the static inbound entries (Ingress), and the dynamic inbound entries (Mirror):

 
```
Router# show ip access-lists\
*Extended IP access list Egress\
10 permit ip any any reflect Mirror (76 matches)\
Extended IP access list Ingress\
10 evaluate Mirror\
Reflexive IP access list Mirror\
permit tcp host 209.20.64.81 eq www host 192.168.0.123 eq 62839 (7 matches) (time left 248)*
```
 

**NOTE:** The expiration timer on the *Mirror* entry this timer is reset to 300 seconds with each new packet that would cause the reflection. If no new traffic has been seen before the timer expires, the entry is erased. Additionally, when the router detects a session has been closed (for example, using the FIN flag in TCP), the timer is immediately reduced and the entry is removed shortly thereafter.

 

**Troubleshooting/Verification\

```
Router# show access-lists *\<number>*
```

-   Displays access list information for all protocols.

>  

```
Router# show ip access-lists \[*\<number>* \| *\<name>*\]
```

-   Displays access-list information specific to IP.

 
```
Router# show ip access-lists interface *\<interface>*
```

-   Displays ACLs configured on a defined interface

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## CBAC Context Based Access Control

The Context-Based Access Control (CBAC) feature of the Cisco IOS® Firewall Feature Set actively inspects the activity behind a firewall. CBAC specifies what traffic needs to be let in and what traffic needs to be let out by using access lists (in the same way that Cisco IOS uses access lists). However, CBAC access lists include ip inspect statements that allow the inspection of the protocol to make sure that it is not tampered with before the protocol goes to the systems behind the firewall.

 

**Context-Based Access Control Configuration**

 

![contextbasedaccesscontrol.png](/Images/contextbasedaccesscontrol.png)


**Identify traffic you want to let out**

What traffic you want to let out depends on your site security policy, but in this general example everything is permitted outbound. If your access list denies everything, then no traffic can leave. Specify outbound traffic with this extended access list:

 
```
Router(config)#ip access-list extended OUTBOUND
Router(config-ext-nacl)# permit tcp 10.10.10.0 0.0.0.255 any
Router(config-ext-nacl)# permit udp 10.10.10.0 0.0.0.255 any\
Router(config-ext-nacl)# permit icmp 10.10.10.0 0.0.0.255 any\
Router(config-ext-nacl)# deny ip any any
```
 

**Identify traffic you want to let in**

What traffic you want to let in depends on your site security policy. However, the logical answer is anything that does not damage your network. In this example, there is a list of traffic that seems logical to let in. Internet Control Message Protocol (ICMP) traffic is generally acceptable, but it can allow some possibilities for DOS attacks. This is a sample access list for incoming traffic:

 
```
Router(config)#ip access-list extended INBOUND
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 echo-reply\
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 unreachable\
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 administratively-prohibited\
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 packet-too-big\
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 echo\
Router(config-ext-nacl)# permit icmp any 10.10.10.0 0.0.0.255 time-exceeded
Router(config-ext-nacl)# permit tcp any host 10.10.10.1 eq smtp
Router(config-ext-nacl)# deny ip any any
```
 

Access list INBOUND is for the inbound traffic and allows only the specified ICMP inbound traffic as well as SMTP access inbound to the server located at 10.10.10.1

**Identify traffic you want to inspect (CBAC)**

The CBAC within Cisco IOS supports many common protocols such as http, ftp, smtp, tcp, udp and more. Each protocol is tied to a keyword name. Apply the keyword name to an interface that you want to inspect.

 

**Syntax**
```
Router(config)# ip inspect name *\<name>* \[protocol\]
```
 

**Example**

```
Router(config)# ip inspect name *cbac* ftp
Router(config)# ip inspect name *cbac* smtp
Router(config)# ip inspect name *cbac* tcp
```

-   This configuration inspects FTP, SMTP, and Telnet (TCP).

>  

**Apply access-lists and inspection statements (CBAC) to the appropriate interfaces**

```
Router(config)# interface ethernet0
Router(config-if)# description \<\--Inside interface
Router(config-if)# ip access-group OUTBOUND in
Router(config-if)# ip inspect *cbac* in
Router(config)# interface serial0.1
Router(config-if)# description \--\>Outside interface
Router(config-if)# ip access-group INBOUND in
```
 

**Troubleshooting/Verification**

```
Router# show ip inspect config
```

-   Displays CBAC inspection information

>  

```
Router# show ip inspect sessions
```

-   Displays activate data sessions being inspected.

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## ZBF Zone Based Firewall

Zone-Based Firewall (ZBF) is similar to the concept used by appliance firewalls (ASA). Router interfaces are places into security zones. Traffic can travel freely between interfaces in the same zone, but is blocked by default from traveling between zones. Traffic is also blocked between interfaces that have been assigned to a security zone and those that have not. You must explicitly apply a policy to allow traffic between zones. Zone policies are configured using the Class-Based Policy Language (CPL), which is similar to the Modular QoS CLI (MQC) in its use of class maps and policy maps. Class maps let you configure highly granular policies if needed. A new class and policy map type, the *inspect* type, is introduced for use with zone-based firewalls.

 

ZBF allows the inspection and control of multiple protocols, including the following:

-   HTTP and HTTPS

-   SMTP, Extended SMTP (ESMTP, POP3, and IMAP

-   Peer-to-peer applications, with the ability to use heuristics to track port hopping

-   Instant messaging applications, (AOL, Yahoo!, and MSM)

-   Remote Procedure Calls (RPC) - Used in remote desktop.

>  

**Zone-Based Firewall Configuration**

![zonebasedfirewall.png](/Images/zonebasedfirewall.png)

For this example, we will pretend the R1 device is in the Inside, private, protected network. R3 represents a device in the Outside, public, unprotected Internet. R2 in the middle will be our IOS Zone-Based Firewall. We want to inspect HTTP, HTTPS and FTP traffic sourced from the Inside network traveling to the Outside network, and we want to dynamically permit return traffic back in from the Outside based on session information.

 

**Define Zones**

```
R2(config)# zone security ZONE_PRIVATE\
R2(config-sec-zone)# description Private LAN
R2(config)# zone security ZONE_INTERNET
R2(config-sec-zone)# description Public Internet
```
 

**Apply Zones to Interfaces**

```
R2(config)# interface fa0/0
R2(config-if)# zone-member security ZONE_PRIVATE
R2(config-if)# interface fa0/1\
R2(config-if)# zone-member security ZONE_INTERNET
```

>  

**Define the class maps that identify traffic that is permitted between zones**

```
R2(config)# class-map type inspect match-any CM_INTERNET_TRAFFIC\
R2(config-cmap)# match protocol http
R2(config-cmap)# match protocol https
R2(config-cmap)# match protocol ftp
```
 

**Configure a policy map that specifies actions for the traffic**

```
R2(config)# policy-map type inspect PM_PRIVATE_TO_INTERNET
R2(config-pmap)# class type inspect CM_INTERNET_TRAFFIC\
R2(config-pmap-c)# inspect
```
 

**Configure the zone pair and apply the policy**

```
R2(config)# zone-pair security ZONEP_PRIV_INT source ZONE_PRIVATE destination ZONE_INTERNET\
R2(config-sec-zone-pair)# service-policy type inspect PM_PRIVATE_TO_INTERNET
```
 

Further testing can be completed, if you attempted to use any protocol besides the ones that are specified, the connection would fail.

 

**Troubleshooting/Verification**

```
Router# show zone-pair security
```

-   Displays zone pairs and their associated service-policy

 
```
Router# show policy-map type inspect zone-pair
```

-   Displays information related to ZBF; zone-pair, policy, class-maps and additional details.

 

 ----------------------------------------------------------------------------------------------------------------------------------------------------------------

## RITE IP Traffic Export

When it comes to capturing packets traversing an Ethernet switch, Cisco\'s Switched Port Analyzer (SPAN) feature is an invaluable tool. However, replicating traffic across router interfaces poses a problem: SPAN can\'t be used on routers, as the underlying hardware doesn\'t support it.

 

Cisco\'s solution is IP traffic export (sometimes referred to as Router IP Traffic Export, or RITE). Introduced in IOS 12.3(4)T, RITE allows for the replication of packets from one interface to another on software-switched platforms in the same manner SPAN operates on hardware-based switches. RITE allows us to \"SPAN\" across routed interfaces, even across disparate medium types.

 

**RITE Configuration**

 

![iptrafficexport.png](/Images/iptrafficexport.png)

 

Consider a scenario in which we need a sniffer or IDS to capture IP traffic traversing a PPP serial link between two routers.

 

We can enable RITE on R1 to replicate WAN traffic to the Ethernet sniffer. Only two items are needed to configure a minimal RITE profile:

 

-   Physical output interface (the interface connected to our sniffer)

-   Destination MAC address for the replicated packets (the MAC address of our sniffer)

 

We can see from the diagram that our sniffer is attached to FastEthernet0/0, and we\'ll assume the MAC address of our sniffer is 00-01-02-03-04-05. (Note that as sniffers are generally run in promiscuous mode, *any* MAC should work in this case. However, RITE won\'t accept a broadcast MAC address.)

 

**Define Profile**

```
R1(config)# ip traffic-export profile MyProfile
```
 

**Define Sniffer Destination**

```
R1(conf-rite)# interface f0/0\
R1(conf-rite)# mac-address 0001.0203.0405
R1(conf-rite)# bidirectional
```
 

The above configuration is all that\'s required for a minimal profile. Additionally, we\'ll include the bidirectional parameter here to ensure that traffic from both directions is replicated (versus only inbound traffic).

 

We can also optionally specify an access list and/or sampling rate to limit the type and amount of traffic we capture, respectively, with the incoming and outgoing parameters.

 

**Define Type and/or sampling of traffic**

```
R1(conf-rite)# incoming ?\
access-list Apply standard or extended access lists to exported traffic\
sample Enable sampling of exported traffic
```

```
R1(conf-rite)# incoming access-list ?\
\<1-199> IP access list (standard or extended)\
\<1300-2699> IP expanded access list (standard or extended)\
WORD Access-list name
```

```
R1(conf-rite)# incoming sample ?\
one-in-every Export one packet in every
```
 

Forgoing these options for this scenario, we are ready to apply our RITE policy to R1\'s serial interface. Make sure that you exit RITE configuration before entering interface configuration to avoid modifying the interface parameter of the RITE profile.

 

**Apply to interface that is to be replicated**\

```
R1(config)# interface s0/0\
R1(config-if)# ip traffic-export apply MyProfile\
*%RITE-5-ACTIVATE: Activated IP traffic export on interface Serial0/0*
```
 

All IP traffic traversing R1\'s Serial0/0 interface is now being replicated out R1\'s FastEthernet0/0 interface toward our sniffer, with one observed exception. Only transit traffic is replicated; inbound traffic destined for R1 itself is also replicated, however outbound traffic generated locally by R1 is *not*. Also note that only IP traffic is being replicated, and we lose any lower-layer headers (like PPP) due to the change in medium.

 

**Troubleshooting/Verification**

```
Router# show ip traffic-expert
```

-   Displays RITE configuration and statistics

 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IPS Intrusion Prevention System

Cisco IOS Intrusion Prevention System (IPS) is a feature that you can enable on Cisco routers. It provides Deep Packet Inspection (DPI) of traffic transiting the router. This is especially useful in branch offices to catch worms, viruses, and other exploits before they leave the local site. Thus, the attack is contained, and WAN bandwidth is not used needlessly. Routers with the security image come with a package of signature files loaded in their flash. Updates to signature packages are posted on Cisco.com, where they can be downloaded to a TFTP server and then installed on the router. You can also configure the router to download and install new signatures on a regular basis. The number of signature files that a router supports depends on the amount of its memory.

 

When IOS IPS is configured, the router acts as an inline IPS, comparing each packet that flows through it to known signatures. Router actions upon finding a signature match include

 

-   Dropping the packet

-   Resetting the connection

-   Sending an alarm log message

-   Blocking traffic from the packet source for a configurable amount of time

-   Blocking traffic on the connection for a configurable amount of time

>  

**IPS Configuration**

Enabling IOS IPS on a router is fairly simple. At its most basic, you need to globally load the IPS signature package, then create an IPS rule, and apply that rule to an interface either inbound or outbound. Beginning with IPS Version 5 you need an RSA key, based on the Cisco public key, to decrypt the signature files. If you are a registered Cisco.com user with a Cisco Service Agreement, this key is available for download at <http://www.cisco.com/pcgi-bin/tablebuild.pl/ios-v5sigup>. Because the IPS process is resource intensive, you should disable (or retire) any unneeded signature categories

 

**Create Crypto Key and load Cisco\'s Public Key**

```
Router(config)#crypto key pubkey-chain rsa
Router(config-pubkey-chain)#named-key realm-cisco.pub signature
Router(config-pubkey-key)#key-string
```

Enter a public key as a hexidecimal number \....

```
Router(config-pubkey)#\$64886 F70D0101 01050003 82010F00 3082010A 02820101
```

-   The entire public key is not shown.

>  

**Load only the basic IP signature package**

```
Router(config)#ip ips signature-category
Router(config-ips-category)#category all
Router(config-ips-category-action)#retired true
```

-   Disables all additional IP signatures

```
Router(config-ips-category-action)#exit
Router(config-ips-category)#category ios_ips basic
Router(config-ips-category-action)#retired false
```

-   Enables only basic IP signature package

 

**Create a location to store IPS information**

```
Router# mkdir flash:ips
```

**Create an IPS rule**

```
Router(config)# ip ips name *\<name>*
```

**Specify the location for the signature information**

```
Router(config)# ip ips config location flash:ips
```
 

**Assign the IPS rule to an interface**

```
Router(config)# interface *\[interface\]*
Router(config-if)# ip ips *\<name>* outbound
```
 

**Troubleshooting/Verification**

```
Router# show ip ips configuration
```

-   Displays IPS information

 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Virtual Private Networks VPN

Improve security and maintain productivity with Cisco VPN technology. Cisco VPNs help securely connect offices, remote users, and business partners. VPNs have become the primary solution for remote connectivity for organizations of all sizes, using affordable, third-party Internet access.

 

**Remote Access VPN** - extend almost any data, voice, or video application to the remote desktop, emulating the main office desktop. With this VPN, you can provide highly secure, customizable remote access to anyone, anytime, anywhere, with almost any device.

 

**Site-to-Site VPN** - provide an Internet-based WAN infrastructure to extend network resources to branch offices, home offices, and business partner sites. All traffic between sites is encrypted using IPsec protocol and integrates network features such as routing, quality of service, and multicast support.

 

**IPsec over GRE VPN -** Extending a VPN Site-to-Site connection (GRE) Generic Router Encapsulation allows for additional protocols to pass through the VPN including but not limited to routing protocols and enables communication over a secure VPN tunnel.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Remote Access VPN

Remote access VPNs extend almost any data, voice, or video application to the remote desktop, emulating the main office desktop. With this VPN, you can provide highly secure, customizable remote access to anyone, anytime, anywhere, with almost any device.

 

**Remote Access (VPN) Configuration**

![remoteaccess.png](/Images/remoteaccess.png)
 

**Configure ISAKMP Policy (IKE Phase 1)**

Phase 1 establishes a secure channel over which the IPsec tunnel can be negotiated.

 
```
Edmonton(config)# crypto isakmp policy 10
```

-   Creates IKE Policy

```
Edmonton(config-isakmp)# encryption aes 128
```

-   Defines the encryption method (AES128)

```
Edmonton(config-isakmp)# hash sha
```

-   Defines the hashing method (SHA) algorithm

```
Edmonton(config-isakmp)# authentication pre-share
```

-   Specifies authentication with a preshared key.

```
Edmonton(config-isakmp)# group 2
```

-   Defines Diffie-Hellman group 2 key exchange algorithm

```
Edmonton(config-isakmp)# lifetime 3600
```

-   Specifies the lifetime of the IKE SA

 

**Configure Policies for the Client Group(s)**

```
Edmonton(config)# crypto isakmp client configuration group **VPNGRP1**
Edmonton(config-isakmp-group)# key 12345678
```

-   Defines preshared key authentication of remote access VPN clients.

```
Edmonton(config-isakmp-group)# pool VPNPOOL
```

-   Defines addresses that will be assigned to remote access VPN clients.

```
Edmonton(config-isakmp-group)# dns 192.31.7.1
```
 

**IPsec Transform Set (IKE Phase 2)**

Transform set, defines parameters of the IPsec security associations which will carry and encrypt the actual data.

 
```
Edmonton(config)# crypto ipsec transform-set **AES128-SHA-HMAC** esp-aes esp-sha-hmac
```
 

**Configure AAA and Add VPN client users**

Cisco IOS based VPNs require router [AAA] service to be enabled. VPN clients can be defined locally in the router or on an AAA server. There are separate lists for authentication and authorization of VPN users.

 
```
Edmonton(config)# aaa new-model
```

-   Enables AAA service

```
Edmonton(config)# aaa *authentication* login default local
```

-   Verifies login authentication for \"default\" group using the local user database.

```
Edmonton(config)# aaa *authentication* login VPNAUTH local
```

-   Verifies login authentication for the VPNAUTH group using the local user database.

```
Edmonton(config)# aaa *authorization* exec default local
```

-   Verifies EXEC authorization for the \"default\" group using the local user database.

```
Edmonton(config)# aaa *authorization* network VPNAUTHOR local
```

-   Verifies network access authorization for the VPNAUTHOR group using the local user database.

```
Edmonton(config)# username *user1* secret *password1*
```

-   Creates local database use for VPN authentication

>  

**Create VPN Client Policy for SA Negotiation (Dynamic Map)**

```
Edmonton(config)# crypto dynamic-map **DYNMAP** 10
Edmonton(config-crypto-map)# set transform-set **AES128-SHA-HMAC**
Edmonton(config-crypto-map)# reverse-route
```

-   Reverse route has the router add a return route for the VPN client in the routing table.

>  

**Cryptomap (IKE Phase 2)**

```
Edmonton(config)# crypto map **CRYPTO-MAP** client authentication list **VPNAUTH**
```

-   Configures IKE extended authentication (Xauth) for the VPN group VPNAUTH.

```
Edmonton(config)# crypto map **CRYPTO-MAP** isakmp authorization list **VPNAUTHOR**
```

-   Configures IKE key lookup from a AAA server for the VPN group VPNAUTHOR

```
Edmonton(config)# crypto map **CRYPTO-MAP** client configuration address respond
```

-   Enables the router to accept IP address requests from any peer.

```
Edmonton(config)# crypto map **CRYPTO-MAP** 65535 ipsec-isakmp dynamic **DYNMAP**
```

-   Uses IKE to establish IPsec Sas as specified by crypto map DYNMAP.

>  

**Apply Crypto Map (IKE Phase 2)**

```
Edmonton(config)# interface serial 0/0
Edmonton(config-if)# ip address 192.31.7.1 255.255.255.252
Edmonton(config-if)# crypto map **CRYPTO-MAP**
```
 

**Troubleshooting/Verification**

```
Router# show crypto ipsec sa
```

-   This command displays the settings used by the current Security Associations (SAs).

>  

```
Router# show crypto isakmp sa
```

-   This command displays current IKE Security Associations.

 
```
Router# debug crypto isakmp
```

-   This command allows you to observe Phase 1 ISAKMP negotiations.

 
```
Router# debug crypto ipsec
```

-   This command allows you to observe Phase 2 IPSec negotiations.

```
Router# show crypto dynamic-map
```

-   Displays a dynamic crypto map set

>  

```
Router# show crypto map
```

-   Displays the crypto map configuration

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Site to Site VPN

Site-to-site VPNs provide an Internet-based WAN infrastructure to extend network resources to branch offices, home offices, and business partner sites. All traffic between sites is encrypted using IPsec protocol and integrates network features such as routing, quality of service, and multicast support.

**Site-to-Site VPN Configuration**

![remoteaccess.png](/Images/remoteaccess.png)

**Configure ISAKMP Policy (IKE Phase 1)**

Phase 1 establishes a secure channel over which the IPsec tunnel can be negotiated.

```
Winnipeg(config)# crypto isakmp policy 10
```

-   Creates IKE Policy

```
Winnipeg(config-isakmp)# encryption aes 128
```

-   Defines the encryption method (AES128)

```
Winnipeg(config-isakmp)# hash sha
```

-   Defines the hashing method (SHA) algorithm

```
Winnipeg(config-isakmp)# authentication pre-share
```

-   Specifies authentication with a preshared key.

```
Winnipeg(config-isakmp)# group 2
```

-   Defines Diffie-Hellman group 2 key exchange algorithm

```
Winnipeg(config-isakmp)# lifetime 3600
```

-   Specifies the lifetime of the IKE SA

>  

**NOTE:** Exact configuration for IKE Phase 1 must be replicated on VPN peers (Edmonton)

**ISAKMP Pre-Shared Key**

Key must match on both sides, address identifies other end of the tunnel.

 
```
Winnipeg(config-isakmp)# crypto isakmp key ***0** 12345678* address 192.31.71.1
```

-   Specifies the key required for the tunnel endpoint, the 0 specifies the password is unencrypted (clear text) password, option 6 would encrypt the key. The address defines the peer IP address for tunnel association.

```
Edmonton(config-isakmp)# crypto isakmp key ***0** 12345678* address 172.107.55.9
```

**IPsec Transform Set (IKE Phase 2)**

Transform set, defines parameters of the IPsec security associations which will carry and encrypt the actual data.

 
```
Winnipeg(config)# crypto ipsec transform-set **AES128-SHA-HMAC** esp-aes esp-sha-hmac
```

-   Creates a transform set for IKE phase 2 policy

```
Winnipeg(cgf-crypto-trans)# mode tunnel
```

-   Tunnel (datagram encapsulation) mode, more secure and preferred.

-   Transport (payload encapsulation) mode.

```
Winnipeg(config)# crypto ipsec security-association lifetime seconds 1200
```

-   Defines a 20 minute SA - security association lifetime.

**ACL to allow traffic across the tunnel**

```
Winnipeg(config)# access-list **100** permit ip 192.168.30.0 0.0.0.255 10.10.30.0 0.0.0.255
```

```
Edmonton(config)# access-list **100** permit ip 10.10.30.0 0.0.0.255 192.168.30.1 0.0.0.255
```

**Cryptomap (IKE Phase 2)**

Cryptomap selects data flows that need security processing. Second, it defines the policy for these flows and the crypto peer that traffic needs to go to.

```
Winnipeg(config)# crypto map CRYPTO-MAP *1* ipsec-isakmp
```

-   Defines crypto map, note the **1** defines the crypto-map sequence number.

```
Winnipeg(config-crypto-map)# set peer 192.31.7.1
Winnipeg(config-crypto-map)# set transform-set **AES128-SHA-HMAC**
Winnipeg(config-crypto-map)# match address **100**
```

```
Edmonton(config)# crypto map CRYPTO-MAP *1* ipsec-isakmp
Edmonton(config-crypto-map)# set peer 128.107.55.9
Edmonton(config-crypto-map)# set transform-set **AES128-SHA-HMAC**
Edmonton(config-crypto-map)# match address **100**
```

**Apply Crypto Map (IKE Phase 2)**

Only a single cryptomap can be applied per interface, however multiple VPNs can be created on a single interface. This is accomplished by incrementing the cryptomap sequence number as denoted above in the previous configuration. Crypto map must be applied on both the physical interface and the tunnel interface.

 
```
Winnipeg(config)# interface FastEthernet0/0
Winnipeg(config-if)# crypto map **CRYPTO-MAP**
```

**Troubleshooting/Verification**

```
Router# show crypto ipsec sa
```

-   This command displays the settings used by the current Security Associations (SAs).

>  

```
Router# show crypto isakmp sa
```

-   This command displays current IKE Security Associations.

 
```
Router# debug crypto isakmp
```

-   This command allows you to observe Phase 1 ISAKMP negotiations.

 
```
Router# debug crypto ipsec
```

-   This command allows you to observe Phase 2 IPSec negotiations.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

# GRE over IPsec VPN

We can encrypt our GRE tunnels using IPsec, and it is also possible to have GRE over IPSEC, in other words: Sending GRE header inside the IPsec transport headers. (transport mode instead of tunnel mode)

 

What we are trying to cover in this section is IPsec over GRE tunnels (as a transport) not GRE over IPSEC (tunneled). GRE provides a mechanism for additional traffic across a VPN, such as routing protocols.

 

**GRE over IPsec Configuration**

![remoteaccess.png](/Images/remoteaccess.png)

 

**Create the GRE Tunnel**

Reference [GRE Tunnel] configuration for additional information about Generic Router Encapsulation.

 
```
Winnipeg(config)# interface tunnel0
Winnipeg(config-if)# ip address 192.168.3.1 255.255.255.0
Winnipeg(config-if)# tunnel source fastethernet0/0
Winnipeg(config-if)# tunnel destination 192.31.7.1
```

**Configure ISAKMP Policy (IKE Phase 1)**

Phase 1 establishes a secure channel over which the IPsec tunnel can be negotiated.

```
Winnipeg(config)# crypto isakmp policy 10
```

-   Creates IKE Policy

```
Winnipeg(config-isakmp)# encryption aes 128
```

-   Defines the encryption method (AES128)

```
Winnipeg(config-isakmp)# hash sha
```

-   Defines the hashing method (SHA) algorithm

```
Winnipeg(config-isakmp)# authentication pre-share
```

-   Specifies authentication with a preshared key.

```
Winnipeg(config-isakmp)# group 2
```

-   Defines Diffie-Hellman group 2 key exchange algorithm

```
Winnipeg(config-isakmp)# lifetime 3600
```

-   Specifies the lifetime of the IKE SA\
    >  

**ISAKMP Pre-Shared Key**

Key must match on both sides, address identifies other end of the tunnel.

```
Winnipeg(config-isakmp)# crypto isakmp key ***0** 12345678* address 192.31.71.1
```

-   Specifies the key required for the tunnel endpoint, the 0 specifies the password is unencrypted (clear text) password, option 6 would encrypt the key. The address defines the peer IP address for tunnel association.

```
Edmonton(config-isakmp)# crypto isakmp key ***0** 12345678* address 172.107.55.9
```

**IPsec Transform Set**

Transform set defines parameters of the IPsec security associations which will carry and encrypt the actual data.

```
Winnipeg(config)# crypto ipsec transform-set **AES128-SHA-HMAC** esp-aes esp-sha-hmac
```

-   Creates a transform set for IKE phase 2 policy

```
Winnipeg(cgf-crypto-trans)# mode tunnel
```

-   Tunnel (datagram encapsulation) mode, more secure and preferred.

-   Transport (payload encapsulation) mode.

```
Winnipeg(config)# crypto ipsec security-association lifetime seconds 1200
```

-   Defines a 20 minute SA - security association lifetime.

**ACL to allow traffic across the tunnel**

Slightly different than a traditional site-to-site VPN, GRE is carrying the traffic and not the Ipsec VPN, because of this you just need allow GRE to be permitted between tunnel endpoints.

```
Winnipeg(config)# access-list 101 permit gre host 128.10.7.55.9 host 192.31.7.1
```

```
Edmonton(config)# access-list 102 permit gre host 192.31.7.1 host 128.107.55.9
```

**Cryptomap (IKE Phase 2)**

Cryptomap selects data flows that need security processing. Second, it defines the policy for these flows and the crypto peer that traffic needs to go to.

 
```
Winnipeg(config)# crypto map **CRYPTO-MAP** *1* ipsec-isakmp
```

-   Defines crypto map, note the **1** defines the crypto-map sequence number.

```
Winnipeg(config-crypto-map)# set transform-set **AES128-SHA-HMAC** Winnipeg(config-crypto-map)# set peer 192.31.7.1
```

```
Edmonton(config)# crypto map **CRYPTO-MAP** *1* ipsec-isakmp
Edmonton(config-crypto-map)# set transform-set **AES128-SHA-HMAC**
Edmonton(config-crypto-map)# set peer 128.107.55.9
```

**Routing over GRE Tunnel**

GRE is multiprotocol and can tunnel any OSI Layer 3 protocol.

 

**Static Routing**

```
Winnipeg(config)# ip route 0.0.0.0 0.0.0.0 128.107.55.10
```

-   Configures a static default route to the physical next-hop IP address, such as the connection to the internet.

```
Winnipeg(config)# ip route 10.10.30.0 255.255.255.0 192.168.3.2
```

-   Configures a static route for (local) tunnel traffic giving the far-end tunnel address as the next-hop IP address.

>  

**Dynamic Routing**

Dynamic would be configured as is; ensure that routing is only performed using the GRE tunnel and not the physical interfaces, i.e. the use of passive-interfaces on physical interfaces and a neighbor relationship will be formed over the GRE tunnel.

 

**Apply Crypto Map (IKE Phase 2)**

Allow Crypto Maps to both the physical and tunnel interface.

 
```
Winnipeg(config)# interface fastethernet 0/0
Winnipeg(config-if)# crypto map **CRYPTO-MAP**
Winnipeg(config)# interface tunnel 0
Winnipeg(config-if)# crypto map **CRYPTO-MAP**
```

```
Edmonton(config)# interface serial 0/0
Edmonton(config-if)# crypto map **CRYPTO-MAP**
Edmonton(config)# interface tunnel 0
Edmonton(config-if)# crypto map **CRYPTO-MAP**
```

**Troubleshooting/Verification**

```
Router# show crypto ipsec sa
```

-   This command displays the settings used by the current Security Associations (SAs).

>  

```
Router# show crypto isakmp sa
```

-   This command displays current IKE Security Associations.

>  

```
Router# show interfaces tunnel *number*
```

-   Lists tunnel interface information

```
Router# debug crypto isakmp
```

-   This command allows you to observe Phase 1 ISAKMP negotiations.

```
Router# debug crypto ipsec
```
-   This command allows you to observe Phase 2 IPSec negotiations.

 

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Disabling Services

The three functional planes of a network, the management plane, control plane, and data plane, each provide different functionality that needs to be protected. There are several services that can be disabled in order to help secure a Cisco IOS network device.

 

-   **Management Plane**---The management plane manages traffic that is sent to the Cisco IOS device and is made up of applications and protocols such as SSH and SNMP.

>  

-   **Control Plane**---The control plane of a network device processes the traffic that is paramount to maintaining the functionality of the network infrastructure. The control plane consists of applications and protocols between network devices, which includes the Border Gateway Protocol (BGP), as well as the Interior Gateway Protocols (IGPs) such as the Enhanced Interior Gateway Routing Protocol (EIGRP) and Open Shortest Path First (OSPF).

>  

-   **Data Plane**---The data plane forwards data through a network device. The data plane does not include traffic that is sent to the local Cisco IOS device.

>  

Keep in mind that there is no full proof network security with anything technological. Best practice is to disable any services that are not absolutely necessary.

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Source Routing

Source routing is a technique whereby the sender of a packet can specify the route that a packet should take through the network. As a packet travels through the network, each router will examine the destination IP address and choose the next hop to forward the packet to. In source routing, the \"source\" (i.e., the sender) makes some or all of these decisions. Attackers can use source routing to probe the network by forcing packets into specific parts of the network. Using source routing, an attacker can collect information about a network s topology, or other information that could be useful in performing an attack. During an attack, an attacker could use source routing to direct packets to bypass existing security restrictions.

**Disable Source Routing**

```
Router(config)# no ip source-route
```
 
----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Proxy ARP

Proxy ARP (Address Resolution Protocol) is the technique in which one host, usually a router, answers ARP requests intended for another machine. By \"faking\" its identity, the router accepts responsibility for routing packets to the \"real\" destination. Proxy ARP can help machines on a subnet reach remote subnets without the need to configure routing or a default gateway.

 

**Disadvantages of Proxy ARP**

-   It increases the amount of ARP traffic on your segment.

-   Hosts need larger ARP tables in order to handle IP-to-MAC address mappings.

-   Security can be undermined. A machine can claim to be another in order to intercept packets, an act called \"spoofing.\"

-   It does not work for networks that do not use ARP for address resolution.

-   It does not generalize to all network topologies. For example, more than one router that connects two physical networks.

 

**Disabling Proxy ARP**

```
Router(config)# interface *\[interface\]*
Router(config-if)# no ip proxy-arp
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Option

The IP Options Selective Drop feature enables you to protect your network routers in the event of a

denial of service (DoS) attack. Hackers who initiate such attacks commonly send large streams of

packets with IP options. By dropping the packets with IP options, you can reduce the load of IP options

packets on the router. The end result is a reduction in the effects of the DoS attack on the router and on

downstream routers.

 

IP options are specifically problomatic because the way Cisco IOS devices process IP options. Because Cisco IOS processes IP Options through route processing and not hardware based processing. Software-switching of IP options packets can lead to a serious security problem if a Cisco IOS router comes under a DoS attack by a hacker sending large streams of packets with IP options. The RP can easily become overloaded and drop high priority or routing protocol packets.

 

The IP Options Selective Drop feature provides the ability to drop packets with IP options in the forwarding engine so that they are not forwarded to the RP. This result in a minimized load on the RP and reduced RP processing requirements.

 

**IP Options Selective Drop**

```
Router(config)# ip options drop
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Redirects

ICMP supports IP traffic by relaying information about paths, routes, and network conditions. ICMP redirect messages instruct an end node to use a specific router as its path to a particular destination. In a properly functioning IP network, a router will send redirects only to hosts on its own local subnets, no end node will ever send a redirect, and no redirect will ever be traversed more than one network hop. However, an attacker may violate these rules; some attacks are based on this. Disabling ICMP redirects will cause no operational impact to the network, and it eliminates this possible method of attack.

 

**Disable IP Redirects**

```
Router(config)# interface *\[interface\]*
Router(config-if)# no ip redirects
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Unreachable

ICMP supports IP traffic by relaying information about paths, routes, and network conditions. ICMP host unreachable messages are sent out if a router receives a nonbroadcast packet that uses an unknown protocol, or if the router receives a packet that it is unable to deliver to the ultimate destination because it knows of no route to the destination address. These messages can be used by an attacker to gain network mapping information

 

**Disabling IP Unreachables**

```
Router(config)# interface *\[interface\]*
Router(config-if)# no ip unreachables
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## IP Directed Broadcast

An IP directed broadcast is a datagram sent to the broadcast address of a subnet to which the sending machine is not directly attached. The directed broadcast is routed through the network as a unicast packet until it arrives at the target subnet, where it is converted into a link-layer broadcast.

 

Because of the nature of the IP addressing architecture, only the last router in the chain, the one that is connecte ddirectly to the target subnet, can conclusively identify a directed broadcast. Directed broadcasts are occasionally used for legitimate purposes, but such use is not common.

 

IP directed broadcasts are used in the extremely common and popular "smurf" Denial-of-Service attack, and they can also be used in related attacks. In a "smurf" attack, the attacker sends ICMP echo requests from a falsified source address to a directed broadcast address, causing all the hosts on the target subnet to send replies to the falsified source. By sending a continuous stream of such requests, the attacker can create a much larger stream of replies, which can completely inundate the host whose address is being falsified .Disabling IP directed broadcasts causes directed broadcasts that would otherwise be "exploded" into link-layer broadcasts at that interface to be dropped instead.

 

**Disable IP Directed Broadcast**

```
Router(config)# interface *\[interface\]*
Router(config-if)# no ip directed-broadcast
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## Cisco Discovery Protocol (CDP)

Cisco Discover Protocol or CDP is a Cisco-proprietary protocol that runs on all Cisco products. CDP allows devices to learn about neighboring devices (the ones attached directly to the switch) including information about their platform, IP address, the version of IOS or other OS, VLAN membership, etc. This can be helpful information when troubleshooting network issues, it can also provide an attacker valuable information about the layout of your network. Other vulnerabilities include a denial of service attack in which CDP packets are generated, flooding the network.

 

**Disable CDP**

**Disable Globally**

```
Router(config)# no cdp run
```
 

**Disable per interface**

```
Router(config)# interface *\[interface\]*
Router(config-if)# no cdp enable
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------

## TCP Intercept

The TCP intercept feature implements software to protect TCP servers from TCP SYN-flooding attacks, which are a type of denial-of-service attack.

 

A SYN-flooding attack occurs when a hacker floods a server with a barrage of requests for connection. Because these messages have unreachable return addresses, the connections cannot be established. The resulting volume of unresolved open connections eventually overwhelms the server and can cause it to deny service to valid requests, thereby preventing legitimate users from connecting to a web site, accessing e-mail, using FTP service, and so on.

 

The TCP intercept feature helps prevent SYN-flooding attacks by intercepting and validating TCP connection requests. In intercept mode, the TCP intercept software intercepts TCP synchronization (SYN) packets from clients to servers that match an extended access list. The software establishes a connection with the client on behalf of the destination server, and if successful, establishes the connection with the server on behalf of the client and knits the two half-connections together transparently. Thus, connection attempts from unreachable hosts will never reach the server. The software continues to intercept and forward packets throughout the duration of the connection. The number of SYNs per second and the number of concurrent connections proxied depends on the platform, memory, processor, and other factors.

 

In the case of illegitimate requests, the software's aggressive timeouts on half-open connections and its thresholds on TCP connection requests protect destination servers while still allowing valid requests.

 

You can choose to operate TCP intercept in watch mode, as opposed to intercept mode. In watch mode, the software passively watches the connection requests flowing through the router. If a connection fails to get established in a configurable interval, the software intervenes and terminates the connection attempt.

 

**TCP Intercept Configuration**

**Enable TCP Intercept**

```
Router(config)# ip access-list *\<acl>* {permit \| deny} tcp any \[*destination - wildcard mask*\]
```

-   Define ACL to intercept all requests or only those coming from specific networks or destined for specific servers.Typically the access list will define the source as *any* and define specific destination networks or servers. That is, you do not attempt to filter on the source addresses because you do not necessarily know who to intercept packets from. You identify the destination in order to protect destination servers. If no access list match is found, the router allows the request to pass with no further action.

>  

**Setting the TCP Intercept Mode**

```
Router(config)# ip tcp intercept mode {intercept \| watch}
```

-   **intercept mode** (Default), the software actively intercepts each incoming connection request (SYN) and responds on behalf of the server with an SYN-ACK, then waits for an ACK from the client. When that ACK is received, the original SYN is sent to the server and the software performs a three-way handshake with the server. When this is complete, the two half-connections are joined.

-   **watch mode**, connection requests are allowed to pass through the router to the server but are watched until they become established. If they fail to become established within 30 seconds (configurable with the **ip tcp intercept watch-timeout** command), the software sends a Reset to the server to clear up its state.

>  

**Setting the TCP Intercept Drop Mode**

```
Router(config)# ip tcp intercept drop-mode {oldest \| random}
```

-   When under attack, the TCP intercept feature becomes more aggressive in its protective behavior. If the number of incomplete connections exceeds 1100 or the number of connections arriving in the last one minute exceeds 1100, each new arriving connection causes the oldest partial connection to be deleted. Also, the initial retransmission timeout is reduced by half to 0.5 seconds (so the total time trying to establish a connection is cut in half).

-   By default, the software drops the oldest partial connection. Alternatively, you can configure the software to drop a random connection.

>  

**Changing TCP Intercept Timers**

```
Router(config)# ip tcp intercept watch-timeout *\<seconds>*
```

-   By default, the software waits for 30 seconds for a watched connection to reach established state before sending a Reset to the server. Changes the time allowed to reach established state.

 
```
Router(config)# ip tcp intercept finrst-timeout *\<seconds>*
```

-   By default, the software waits for 5 seconds from receipt of a reset or FIN-exchange before it ceases to manage the connection. Changes the time between receipt of a reset or FIN-exchange and dropping the connection.

 
```
Router(config)# ip tcp intercept connection-timeout *\<seconds>*
```

-   By default, the software still manages a connection for 24 hours after no activity. Changes the time the software will manage a connection after no activity.

 

**Changing TCP Intercept Aggressive Thresholds**

Two factors determine when aggressive behavior begins and ends: total incomplete connections and connection requests during the last one-minute sample period. Both thresholds have default values that can be redefined.

When a threshold is exceeded, the TCP intercept assumes the server is under attack and goes into aggressive mode. When in aggressive mode, the following occurs:

 

-   Each new arriving connection causes the oldest partial connection to be deleted. (You can change to a random drop mode.)

-   The initial retransmission timeout is reduced by half to 0.5 seconds, and so the total time trying to establish the connection is cut in half. (When not in aggressive mode, the code does exponential back-off on its retransmissions of SYN segments. The initial retransmission timeout is 1 second. The subsequent timeouts are 2 seconds, 4 seconds, 8 seconds, and 16 seconds. The code retransmits 4 times before giving up, so it gives up after 31 seconds of no acknowledgment.)

-   If in watch mode, the watch timeout is reduced by half. (If the default is in place, the watch timeout becomes 15 seconds.)

>  

You can change the threshold for triggering aggressive mode based on the total number of incomplete connections. The default values for **low** and **high** are 900 and 1100 incomplete connections, respectively.

 
```
Router(config)# ip tcp intercept max-incomplete low \<*number>*
```

-   Sets the threshold for stopping aggressive mode.

 
```
Router(config)# ip tcp intercept max-incomplete high \<*number>*
```

-   Sets the threshold for triggering aggressive mode.

>  

You can also change the threshold for triggering aggressive mode based on the number of connection requests received in the last 1-minute sample period. The default values for **low** and **high** are 900 and 1100 connection requests, respectively.

```
Router(config)# ip tcp intercept one-minute low \<*number>*
```

-   Sets the threshold for stopping aggressive mode.

 
```
Router(config)# ip tcp intercept one-minute high \<*number>*
```

-   Sets the threshold for triggering aggressive mode.

>  

**Troubleshooting/Verification**

```
Router# show tcp intercept connections
```

-   Displays incomplete connections and established connections.

>  

```
Router# show tcp intercept statistics
```
-   Displays TCP intercept statistics

 

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Unicast Reverse Path Forwarding uRPF

Network administrators can use Unicast Reverse Path Forwarding (Unicast RPF) to help limit the malicious traffic on an enterprise network. This security feature works by enabling a router to verify the reachability of the source address in packets being forwarded. This capability can limit the appearance of spoofed addresses on a network. If the source IP address is not valid, the packet is discarded. Unicast RPF works in one of three different modes: strict mode, loose mode, or VRF mode. Note that not all network devices support all three modes of operation.


When administrators use Unicast RPF in strict mode, the packet must be received on the interface that the router would use to forward the return packet. Unicast RPF configured in strict mode may drop legitimate traffic that is received on an interface that was not the router\'s choice for sending return traffic. Dropping this legitimate traffic could occur when asymmetric routing paths are present in the network.


When administrators use Unicast RPF in loose mode, the source address must appear in the routing table. Administrators can change this behavior using the *allow-default* option, which allows the use of the default route in the source verification process. Additionally, a packet that contains a source address for which the return route points to the Null 0 interface will be dropped. An access list may also be specified that permits or denies certain source addresses in Unicast RPF loose mode.


Care must be taken to ensure that the appropriate Unicast RPF mode (loose or strict) is configured during the deployment of this feature because it can drop legitimate traffic. Although asymmetric traffic flows may be of concern when deploying this feature, Unicast RPF loose mode is a scalable option for networks that contain asymmetric routing paths.

 

**Unicast Reverse path Forwarding (uRFP)**

Unicast RPF is enabled on a per-interface basis. The **ip verify unicast source reachable-via rx** command enables Unicast RPF in strict mode. To enable loose mode, administrators can use the **any** option to enforce the requirement that the source IP address for a packet must appear in the routing table. The **allow-default** option may be used with either the **rx** or **any** option to include IP addresses not specifically contained in the routing table. The **allow-self-ping** option should not be used because it could create a denial of service condition. An access list such as the one that follows may also be configured to specifically permit or deny a list of addresses through Unicast RPF:

 

**Enable uRFP**

```
Router(config)# interface fa0/0
Router(config-if)# ip verify unicast source reachable-via {rx \| any} \[allow-default\] \[allow-self-ping\] \[*list\]*
```
 
Addresses that should never appear on a network can be dropped by entering a route to a null interface. The following command will cause all traffic received from the 10.0.0.0/8 network to be dropped even if Unicast RPF is enabled in loose mode with the **allow-default** option: **ip route 10.0.0.0 255.0.0.0 Null0**

**NOTE:** Cisco Express Forwarding switching must be enabled for Unicast RPF to function


**Troubleshooting/Verification**

```
Router# show cef interface *\[interface\]*
```

-   Verify CEF and Unicast RPF have been enabled on an interface.

```
Router# show ip verify statistics
```

-   Displays statistics about Unicast RPF.