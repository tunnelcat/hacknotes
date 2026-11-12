# Scope and Objectives
A mature organization may provide objectives and scope like: 

**Objectives:**

1.  Identify system misconfigurations and network weaknesses.
    1.  Focus on exterior systems.
2.  Determine the effectiveness of endpoint detection and response systems.
3.  Evaluate overall security posture and response.
    1.  SIEM and detection measures.
    2.  Remediation.
    3.  Segmentation of DMZ and internal servers.
4.  Use of white cards is permitted depending on downtime and length.
5.  Evaluate the impact of data exposure and exfiltration.

**Scope:**

1.  System downtime is not permitted under any circumstances.
    1.  Any form of DDoS or DoS is prohibited.
    2.  Use of any harmful malware is prohibited; this includes ransomware and other variations.
2.  Exfiltration of PII is prohibited. Use arbitrary exfiltration data.
3.  Attacks against systems within 10.0.4.0/22 are permitted.
4.  Attacks against systems within 10.0.12.0/22 are prohibited.
5.  Bean Enterprises will closely monitor interactions with the DMZ and critical/production systems.
    1.  Any interaction with "*.bethechange.xyz" is prohibited.
    2.  All interaction with "*.globalenterprises.thm" is permitted.

# Rules of Engagement (ROE)
- Generally acts as the contract between parties, legally binding

![[75303cad5a1ce5acce78d80ebb51f103-1.png]]

# Campaign Planning
![[a1754770e0bf0941e10dbe55b69c6a5e-1.png]]

# Engagement Documentation
![[0b966b9d1e85a5f6e7b5d8736865c219-1.png]]

# Concept of Operations
- The Concept of Operation (CONOPS) details a high-level overview of the proceedings of an engagement (like executive summary)
- Written from a semi-technical summary perspective, assuming the reader has zero to minimal technical knowledge

**CONOPS Outline:** 
-   Client Name
-   Service Provider
-   Timeframe
-   General Objectives/Phases
-   Other Training Objectives (Exfiltration)
-   High-Level Tools/Techniques planned to be used
-   Threat group to emulate (if any)

**CONOPS Example:** 
Holo Enterprises has hired TryHackMe as an external contractor to conduct a month-long network infrastructure assessment and security posture. The campaign will utilize an assumed breach model starting in Tier 3 infrastructure. Operators will progressively conduct reconnaissance and attempt to meet objectives to be determined. If defined goals are not met, the red cell will move and escalate privileges within the network laterally. Operators are also expected to execute and maintain persistence to sustain for a period of three weeks. A trusted agent is expected to intervene if the red cell is identified or burned by the blue cell throughout the entirety of the engagement. The last engagement day is reserved for clean-up and remediation and consultation with the blue and white cell.

The customer has requested the following training objectives: assess the blue team's ability to identify and defend against live intrusions and attacks, Identify the risk of an adversary within the internal network. The red cell will accomplish objectives by employing the use of Cobalt Strike as the primary red cell tool. The red cell is permitted to use other standard tooling only identifiable to the targeted threat.

Based on customer security posture and maturity, the TTP of the threat group: FIN6, will be employed throughout the engagement.

# Resource Plan
- Extends the CONOPS and includes specific details, such as dates, knowledge required, resource requirements, etc.
- Written as bulleted lists of subsections

Example subsections: 
-   Header
    -   Personnel writing
    -   Dates
    -   Customer
-   Engagement Dates
    -   Reconnaissance Dates
    -   Initial Compromise Dates
    -   Post-Exploitation and Persistence Dates
    -   Misc. Dates
-   Knowledge Required (optional)
    -   Reconnaissance
    -   Initial Compromise
    -   Post-Exploitation
-   Resource Requirements
    -   Personnel
    -   Hardware
    -   Cloud
    -   Misc.

![[7525777b392949f9569f308d86a741c1-1.png]]

# Operations Plan
- Flexible document(s) that provides specific details of the engagement and actions occurring
- Similar writing scheme to the resource plan, using bulleted lists and small sub-sections
- Notable is the communications plan, which summarizes how the red cell will communicate to the client (or other cells)
	- Vectr.io
	- Email
	- Slack
	- etc.

Example subsections: 
-   Header
    -   Personnel writing
    -   Dates
    -   Customer
-   Halting/stopping conditions (can be placed in ROE depending on depth)
-   Required/assigned personnel
-   Specific TTPs and attacks planned
-   Communications plan
-   Rules of Engagement (optional)

![[940ed8294be1beeb1aa50093f5426698-1.png]]

# Mission Plan
- Internally used document, structure and detail are less important and can vary
- Operator and red cell perspective

Example outline: 
-   Objectives
-   Operators
-   Exploits/Attacks
-   Targets (users/machines/objectives)
-   Execution plan variations

