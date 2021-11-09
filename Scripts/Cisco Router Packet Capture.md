## Cisco Router Packet Capture

Associate a filter. The filter may be specified inline, or an ACL or class-map can be referenced:

```monitor capture CAP interface Port-channel1 both``` 

```monitor capture CAP match ipv4 any any```

Start the capture: 

```monitor capture CAP start```

The capture is now active. Allow it to collect the necessary data. 
Stop the capture: 

```monitor capture CAP stop```

Examine the capture in a summary view: 

```show monitor capture CAP buffer brief``` 

Examine the capture in a detailed view: 

```show monitor capture CAP buffer detailed```  

In addition, export the capture in PCAP format for further analysis: 

```monitor capture CAP export tftp://[IP ADDRESS]/CAP.pcap``` 

Once the necessary data has been collected, remove the capture: 

```no monitor capture CAP```
 
Change PCAP settings and checking it on the router itself: 

```Monitor cap CAP buffer 100``` 

Examine the capture in a summary view: 

```show monitor capture CAP buffer brief``` 

Examine the capture in a detailed view: 

```show monitor capture CAP buffer detailed```