# Practical Incident Handling 
"Only events whit negatives consequences are consired security incidents" NIST Incident Handling Guide

- Examples:
  - System crash, Pack flood, Unauthorized access to sensitive files, 

## Main focus for incident handlers:
- Techinques, Tactics and procedures | Cyberkill chain.

## NIST Procedure:
- Preparation:
  - Documentation -> Procedures
    - Policies
    - Response procedures
    - Incident Communication plans
    - Chain of custody
    - DRP and BIA documents
    - Comunications skills
  - Employees -> People
    - Skilled response team
    - It security Trainning
    - Security awareness
  - Defensive mesure -> Technology
    - SIEM, EDR, NDR... etc.
    - Prepare Forensics tools.
- Detection - analisis
  - Assign primary incident handler (It dependes the level of severity)
  - Establish secure channels of comunication and share
  - Divide the detections in differents areas:
    - Network perimeter: FW, NDR, NIPS...
    - Host perimeter: HIPS...
    - Host level: EDR, EPP...
    - Application level: Web app logs, services logs... (WAF, DBFW...)
  - Establish baselines
    - Rule out False positive
    - Impact analysis
    - Type | Impact | Extent
- Containment - erradication - recovery
  - Prevent an incident getting worse
    - Short term containment: Render the intrusion ineffective
    - System backup: 
      - Data acquisiton:
        - Static acquisition: Not volatile data - Clonning using write blockers
        - Dynamic/Live acquisition: Volatile data - Imaging
      - Integrity:
        - Usually use hash functions for integrity
    - Long term containment: Make sure the intruder is locked out of the affected host and network
      - Patching
      - IoC Injection
      - Change rules on the firewall
      - ...
  - Eliminate intruder artifacts
  - Restore and monitor systems
    - Process systems recovery: Restore and perform QA activities
    - Restore of operations: Restored system will back to production again.
    - Monitoring: Focusing on threat huntig activities for few days
- Post Incident Activity


## Defend against reconnaisance:
- Hide Whois 
- Hide information on the SSL certificates 
- Reduce public information on websites
- Raise employee awareness of the consequences of sharing company information on social networks.
- Reduce "zone transfer" rights
- Split DNS:
  - If the origin of the request is x, the answer is y, if the origin is z then the aswer is a.
- Outlook web access:
  - Mailsniper: Fuzz domain names
  - NTLM over http auth: IIS send "base64" answers that contain AD domain name
    - /autodiscover/autodiscover.xml
    - /EWS/Exchange.asmx
  - XSS
 
  ## Defending against scanning:
  - War dialing:
  - War driving:
    - Generic SSID
    - Use AES in WPA2
    - Wireless IDS
  - Port scan:
    - TCP wrapped
    - IDS/IPS
  - WebRTC-based scans:
    - FIREFOX | about:config and tooggle media.peerconnection.enable = false
    - WEBRTC network limiter
   
  ## Defending against explotation:
  - BGP hijacking
    - Access to router and change the priotity of the traffic.
  - Passive & active sniffing
    - SSL Strip: Redirect https traffic to http, and also exist sslstrip+, use a dns hijacking to change the domain and evade HSTS
  - Remote exploits
    - IDS/IPS
  - NetNTLM hash capturing & relaying
    - Block outbound SMB connections on the border firewall
    - Incrise password complexity
    - Use kerberos
  - Remote linux host attacks
  - Remote denial of service attacks
  - Malicious macros
 
## Defending against post-explotation:
  - Stored credentials:
    - Honey files, create fake credentials like unnatended.xml and monitor his access.
  - Writable registry keys:
    - Monitor Sysmon event 1 and event 13

## Network basics:
 
### OSI/TCP MODEL:

Basic concepts:
- 1 | Physical
  - IEEE 802.x | Bites
- 2 | Data link
  - MAC Address | Frames
- 3 | Network
  - IP | Packets
- 4 | Transport
  - TCP/UDP/SCTP | Segments
- 5 | Session
- 6 | Presentation
- 7 | Application
  - HTTP/HTTPS/DNS | Data
 
Encapsulation:
- 1 | Data 
- 2 | Data + App header
- 3 | Data + App header + Tcp header
- 4 | Data + App header + Tcp header + IP header
- 5 | Ethernet trail + Data + App header + Tcp header + IP header + Ethernet header

### Data link basics:
- Ethernet trail: 4 bytes used for identify frame corruption
- Max fram size is 1564 bytes.
- MAC = 6 bytes = 3 OUI bytes and 3 NISI

### WIFI basics:
- The header have aditional information:
  - Management: (Connectivity at layer 2)
    - Auth | Association
    - Beacon: Send the SSID and requirements
  - Control: (Delivery)
    - Request to send packets
    - Clear to send packets
  - Data (Data container)

#### ARP attack:
- ARP for IPv4:
  - Not validate source
  - Is stateless
- NDP for ipv6:
  - Implement cryptography and source validation.

- Gratuitous ARP:
  - Request: Is like a announce, is not the same like "who has" this method is just to announce a new ip on the network... so you send:
    - Sender IP: X.X.X.X IP victim
    - MAC Source: 11:00... Malicious MAC add
    - Source IP: X.X.X.X IP Victim
    - MAC Target: 00:00... Pseudo-broadcast (Not use broadcast, but annouce than he dosnt know the MAC add target)
  - Replay: Is a response to force the change on the arp table.
    - Sender IP: X.X.X.X IP victim
    - MAC Source: 11:00... Malicious MAC add
    - Source IP: X.X.X.X IP Victim
    - MAC Target: ff:ff... broadcast (is not waiting for response)
   
- Detection:
  - Duplicated MAC for same IP address.

### Mac attack:
- MAC Flooding:
  - Overflow the switch and its table CAM.
  - Empty request impersonating different IP address on the same node.

### IP basics:
- IPv4 = 32 bytes and IPv6 = 128 bytes

- Keys to detect scan:
  - Random source IP
  - Empty request for transport protocols (ICMP,ICMPv6,GMTP, MFE NSP) and also for ports (UDP,HTTP,SMB...)
 
- keys to detect Teredo (IPv6 tunneling over IPv4):
  - Teredo use UDP protocol for tunneling the traffic.
  - First the local IP negociate using ICMPv6 to create a tunnel.
    - Router solicitation
    - Router advertisement
  - If you "follow" the traffic using wireshark, you will see the real traffic, like SMB, HTTP, ETC.

- ICMPv6:
  - Error messages (1-127)
  - Info messages (128-129) Echo request / Echo replay
  - Multicast listener discovery MLD / MLD2
  - Neighbor discovery (NDP)
    - NS neighbor solicitation -> Multicast FF02::2
    - ND neighbor advertisement <- Router response with configuration
    - Generate IPv6
    - Duplicate address detection
  - Stateless autoconfiguration (SLAAC)
    - Gateway and IP solicitation:
      - RS Router solicitation
      - RA Router advertismenet
  - Other (Router numbering, Mobile ipv6)
    
### TCP Basics:
- TCP Header:
  - Content:
    - Src port - Dst port
    - Sequence number: Identify the TCP segment
    - Ack num: Expected Sequence number of the next pack
    - Flags: URG,SYN,ACK,RST...
    - Window size: TCP buffer size
    - Checksum: Integrity of TCP header and data
    - Urgent Pointer: For URG, Where reading data should start
    - Options: Optional
  - Caracteristics:
    - 20 bytes size, where options is not set.
  - SeqNum and AckNum:
    - SYN:     SEQ = 1000 | ACK = 0
    - SYN-ACK:     SEQ = 2000 | ACK = 1001
    - ACK: SEQ = 1001 | ACK = 2001
  - * Wireshark use "relative numbers"
    - SYN:     SEQ = 0 | ACK = 0
    - SYN-ACK: SEQ = 0 | ACK = 1
    - ACK:     SEQ = 1 | ACK = 1
    - ACK:     SEQ = 1 | ACK = 726
    - ACK:     SEQ = 726 | ACK = 1449
    - ACK:     SEQ = 1449 | ACK = 726
    - ACK:     SEQ = 726 | ACK = 2879
    - ACK:     SEQ = 2879 | ACK = 726 ...

- TCP Basic detection:
  - Excessive SYN (scanning)
  - Usage of diffente flags
  - Multiple ports o host
- NMAP Detection:
  - Nmap use similar o same SeqNum in SYN
  - Use a base number for source port
  - Scanning over destination port 0, is to find live host, responding RTS-ACK
- Reset attack:
  - Usually the IP origin consistent, but the macaddress is different.
- Hikacking Session:
  - This attack share similarities with the reset attack, but involves data injection, in the telnet case you are able to send commands, but the response is will be sent to the correct MAC Address origin.
- TCP Timestamp option:
  - Timestamp value field: Current value of the timestamp clock of the TCP sending.
  - Timestamp replay field: Only valid if is a ACK packet
    - TCP Attacks:
      - Host identification (clock skew) Some systems do not support the feature, others increment the value at frequencies of 2HZ, 100HZ, or 1000HZ, and still others return 0

### ICMP:
- ICMP attacks:
  - ICMP tunneling: ICMP allows encapsulation of traffic, for example, ptunnel is a tool for tunelling data over ICMP.
    - Detection: Two or more responses for each request, abnormal packet size, if you dump the strings you will see the exfiltration.
  - ICMP Redirect: ICMP allows send packets to redirect traffic, if the gateway detect a better route, it sends a new route to clients.
    - Detection: Recive "redirect traffic" from different a different mac than the original.
 
### TCP:
- Netbios Basics:
  - Port 137: Name resolution, similar to ""DNS"".
  - Port 138: Datagram distribution service, Announcement between clients and servers.
  - Port 139: Session negociation, Access files, open directories, etc.
- SMB Basics:
  - Port 445: Share printers, files, directories, etc.
  - Versions:
    - 2.0: 2006 - Windows Vista
    - 2.1: Win 7 - server 2008
    - 3.0: Win 8 - server 2012
  - Attacks:
    - NULL session: Allow execute sessions without a user, like $IPC.
- RPC Basic:
  - ONC RPC: Basic implementation and basic auth.
    - Use PortMapper: port 111.
    - For use in linux.
    - Implemented in NFS.
      - Request 111 ->
      - Response <- Mountd on #dynamic-port, NFS on #dynamic-port
      - Request #mountd-port ->
  - DCE RPC: Professional use and robust auth.
    - Use Endpointmapper: Port 135.
    - For use in windows and unix
    - Implemented in:
      - RPC over SMB:
        - For use in SMB connection using "name pipes", for example "user enumeration", "psexec", etc.
      - RPC over HTTP/HTTPS:
        - For allow RCP calls over firewalls, nat, proxyes, etc.
        - rpcclient -U user //target
      - DCOM:
        - Communication to COM Classes/Objects, for example WMI classes on DYNAMIC ports.
        - wmic /node:target process call create "cmd.exe"
  - SSL/TLS:
    - Renegotiation attack:
      - Send multiple renegotiation connections.
      - Negotiation consume high CPU.
        - Detection:
          - Multiple "Client Hello" request, open and exit connection with out data.
  - SMPT:
    - Basic messages:
      - HELO/EHLO: Identify itself.
      - MAIL FROM: Identify the sender of the message.
      - RCPT TO: Identifies the messages recipients.
      - VRFY: Verify that the mailbox is available for message.
    - User enumeration:
      - VRFY, EXPN & RCPT TO.

 




















