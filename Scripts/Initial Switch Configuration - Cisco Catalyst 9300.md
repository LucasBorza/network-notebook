These commands add the enable password and local username database. 
```
enable secret 5 $1$nXfl$xhMSOtjc0gk5NTuZgeEGD.
!
username pranetadmin privilege 15 secret 5 $1$d3ud$9e9aZGWYU4Yalvidvh8pD1
!
```

This applies the default TACACS configurations for SSH and console access. 

```
aaa new-model
!
tacacs server NHO.ClearPass
address ipv4 10.12.10.21
key 7 051B091D354A4105100A0517085C122F393D716674
tacacs server CDC.ClearPass
address ipv4 172.20.48.48
key 7 051B091D354A4105100A0517085C122F393D716674
!
aaa authentication login default group tacacs+ local
aaa authentication login CONSOLE local
aaa authorization config-commands
aaa authorization exec default group tacacs+ local if-authenticated
aaa authorization commands 15 default group tacacs+ local if-authenticated 
aaa accounting exec default start-stop group tacacs+
aaa accounting commands 15 default start-stop group tacacs+
!
aaa session-id common
!
line con 0
exec-timeout 0 0
logging synchronous
login authentication CONSOLE
!
line vty 0 4
exec-timeout 60 0
logging synchronous
transport input ssh
!
line vty 5 15
exec-timeout 60 0
logging synchronous
transport input ssh
!
```

These commands clear the existing login and motd banners then applied the PRA standard banner.

```
no banner login
no banner motd
!
banner login ^
***************************************************************************************************
***                                         W A R N I N G                                       ***
***               ----------------------------------------------------------------              ***
***    Unauthorized or improper use of this system may result in administrative disciplinary    ***
***    action and/or civil charges/criminal penalties. By continuing to use this system, you    ***
***    indicate your awareness of and consent to these terms and conditions of use.             ***
***                                                                                             ***
***    This computer system is the property of Portfolio Recovery Associates, Inc., its         ***
***    subsidiaries and affiliates (PRA). It is for authorized use only. You should have no     ***
***    expectation of privacy on this system.                                                   ***
***                                                                                             ***
***************************************************************************************************
^ 
!
```

IP access-list 60 is a whitelist for network management via SNMP. 
```
access-list 60 permit 10.12.101.100
access-list 60 permit 10.12.101.107
access-list 60 permit 10.12.101.69
access-list 60 permit 172.16.40.15
access-list 60 permit 172.16.111.121
access-list 60 permit 172.16.100.59
access-list 60 permit 10.12.101.145
!
```

IP access-list 99 is to only allow connection to the specified NTP servers.
```
access-list 99 permit 10.32.101.11
access-list 99 permit 10.5.101.11
access-list 99 permit 10.12.101.11
!
ntp access-group peer 99
ntp server 10.12.101.11 prefer
ntp server 10.5.101.11
ntp server 10.32.101.11
!
```

Alias for simple network commands.
```
alias exec smac sho mac add | i
alias exec soh sho
alias exec sis sho int status
alias exec sib show ip int brief
alias exec sir sho ip route
alias exec topall show ip eigrp topology all
alias exec sip show ip protocols
alias exec srr show run | sec router
alias exec sb sho ip bgp
!
```

SNMP configurations for read only SNMP access with the provided community. 
snmp-server view NO_BAD_SNMP iso included
snmp-server view NO_BAD_SNMP internet included
snmp-server view NO_BAD_SNMP snmpUsmMIB excluded
snmp-server view NO_BAD_SNMP snmpVacmMIB excluded
snmp-server view NO_BAD_SNMP snmpCommunityMIB excluded
snmp-server view NO_BAD_SNMP ciscoMgmt.252 excluded
snmp-server community PrA90#@*P! view NO_BAD_SNMP RO 60
snmp ifmib ifindex persist
!


Misc commands for PRA standards. 
```
cdp run
!
service password-encryption
!
no ip domain lookup
ip domain name portfoliorecovery.com
!
clock timezone EST -5 0
clock summer-time EDT recurring
service timestamps debug datetime msec localtime show-timezone year 
service timestamps log datetime msec localtime show-timezone year
!
diagnostic bootup level complete
!
no ip http server
no ip http secure-server
no ip http authentication local
!
```

Basic L2 vlans added to the vlan database
```
vlan 10
name Network.Infra
vlan 41 
name User.Net1
vlan 50
name Voice.Equip
vlan 51   
name Voice.Net1
vlan 60   
name Printers
vlan 65   
name Security.Cam
vlan 67   
name Building.Facilities
vlan 68   
name Media
vlan 70   
name Wireless.Infra
vlan 101  
name Servers
vlan 121  
name DRAC.iLo1.Remote
vlan 141  
name Collectors.Net1
vlan 151  
name Collectors.Voice1
!
```

Basic L3 networks added if its a routed device (Example:SITE.CORE)
```
interface vlan 10  
desc Network.Infra
ip address 10.235.10.1 255.255.255.0
interface vlan 41  
desc User.Net1
ip address 10.235.41.1 255.255.255.0
interface vlan 50  
desc Voice.Infra
ip address 10.235.50.1 255.255.255.0
interface vlan 51  
desc Voice.Net1
ip address 10.235.51.1 255.255.255.0
interface vlan 60  
desc Printers
ip address 10.235.60.1 255.255.255.0
interface vlan 65  
desc Security.Cam
ip address 10.235.65.1 255.255.255.0
interface vlan 67  
desc Building.Facilities
ip address 10.235.67.1 255.255.255.0
interface vlan 68  
desc Media
ip address 10.235.68.1 255.255.255.0
interface vlan 70  
desc Wireless.Infra
ip address 10.235.70.1 255.255.255.0
interface vlan 101 
desc Servers
ip address 10.235.101.1 255.255.255.0
interface vlan 121 
desc DRAC.iLo1.Remote
ip address 10.235.121.1 255.255.255.0
interface vlan 141 
desc Collectors.Net1
ip address 10.235.141.1 255.255.255.0
interface vlan 151 
desc Collectors.Voice1
ip address 10.235.151.1 255.255.255.0
!
```

DHCP relay / IP helper-addresses added to SVI's where needed.
```
int vlan 41
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 51
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 60
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 65 
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 67
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 68
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 70
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 141
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111 
int vlan 142
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 151
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
int vlan 152
ip helper-address 10.12.101.111
ip helper-address 10.32.101.111
```

Basic BGP configurations for SITE.CORE.
```
router bgp 65205
bgp router-id 10.235.10.1
bgp log-neighbor-changes
neighbor 10.35.10.11 remote-as 65205
neighbor 10.35.10.11 timers 5 15
neighbor 10.35.10.11 route-reflector-client
neighbor 10.35.10.12 remote-as 65205
neighbor 10.35.10.12 timers 5 15
neighbor 10.35.10.12 route-reflector-client
!
```

Basic access port configuration
```
int Gi2/0/24
desc [DESCRIPTION] 
switchport mode access
switchport access vlan 10
spanning-tree bpduguard enable
spanning-tree portfast
!
```
