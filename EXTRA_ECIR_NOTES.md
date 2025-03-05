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
    - Honey files, create fake credentials like unnatende.xml and monitor his access.
  - Writable registry keys:
    - Monitor Sysmon event 1 and event 13
  - 
 

   

  








