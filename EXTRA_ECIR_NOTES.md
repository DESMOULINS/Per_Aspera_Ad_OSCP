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
- 








