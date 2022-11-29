## 1. Threat Modelling

- [1. Threat Modelling](#1-threat-modelling)
- [2. Threat Modelling Overview](#2-threat-modelling-overview)
  - [2.1. Common High Level Stages](#21-common-high-level-stages)
  - [2.2. Terminology](#22-terminology)
  - [2.3. Approaches](#23-approaches)
  - [2.4. Threat Modeling Methodology](#24-threat-modeling-methodology)
  - [2.5. Determining Risk](#25-determining-risk)
- [3. Microsoft Threat Modelling](#3-microsoft-threat-modelling)
  - [3.1. Overview](#31-overview)
    - [3.1.1. Pros and Cons](#311-pros-and-cons)
    - [3.1.2. Process](#312-process)
    - [3.1.3. Work Flow](#313-work-flow)
  - [3.2. Setup](#32-setup)
    - [3.2.1. Installation](#321-installation)
  - [3.3. Preliminary Activities](#33-preliminary-activities)
  - [3.4. Data Flow Diagrams](#34-data-flow-diagrams)
  - [3.5. Identifying and Managing Threats](#35-identifying-and-managing-threats)
    - [3.5.1. Threat Types](#351-threat-types)

## 2. Threat Modelling Overview

### 2.1. Common High Level Stages

- **Decompose** and visualize the system
- **Identify**, categorize and rank threats
- **Control**; countermeasures and mitigation

### 2.2. Terminology

- **Vulnerability** is a weakness that has been found
- **Threat**; any circumstance that can lead to a compromise
- **Exploit** uses a vulnerability and exploits it 
- **Risk** is the level of impact and probability. Risk can be qualitative or quantitative
- **Attack** surface is all the points identified that could be used by an attacker
- **Trust boundary**; used to identify a threat where a trusted person can attack
- **Mitigation** is used to lessen the impact of an attack
- **Eliminate** is the removal of a vulnerability or risk
- **Transfer** is when a risk is moved to someone else, such as a third party. 
- **Accept** the risk due to the impact or probability

### 2.3. Approaches

- **Software centric** uses dataflow diagrams to illustrate the flow of data through an application
- **Asset centric** uses information about assets, what tey are and their functions. It also consider methods and motivations of hackers
- **Attack centric** is focused on tool, tactics and procedures that an attacker might use. Compromises attack tress and attack graphs.


![Software Centric Example](Screenshots/ThreatModelling%20-%20SoftwareCentricApproachExample.png)

![Asset Centric Example](Screenshots/ThreatModelling%20-%20AssetCentricApproachExample.png)

![Asset Centric Example](Screenshots/ThreatModelling%20-%20AttackCentricApproachExample.png)


### 2.4. Threat Modeling Methodology

- **Microsoft Threat Modeling**
  - Uses STRIDE; 
    -  Spoofing
    -  Tampering
    -  Repudiation
    -  Information Disclosure
    -  Denial of Service
    -  Elevation of privilege
  - Process Steps...
    - Identifying Assets
    - Create an architecture overview
    - Decompose the application
    - Identify threats using STRIDE
    - Document threats
    - Rate threats   
- **Process for Attack Simulation and Threat Analysis or P.A.S.T.A** 
  -  Consists of seven steps and is considered a full threat modeling methodology
  -  Process Steps...
    - Define business and security objectives
    - Define the technical scope
    - Decompose the application
    - Perform a threat analysis
    - Identify weakness and vulnerabilities
    - Attack and exploit enumeration
    - Risk impact analysis
  
### 2.5. Determining Risk

MS uses DREAD to categorize risk

- Categories
  - Damage
  - Reproducibility
  - Exportability
  - Affected Users
  - Discoverablility
- Levels
  - Low = 1
  - Medium  2
  - High = 3
- Calculation
  - (D+R+E+A+D) / 5 = RISK
  - Each threat is given a risk

## 3. Microsoft Threat Modelling

The home page can be found [here](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool).

### 3.1. Overview

#### 3.1.1. Pros and Cons

- Pros
  - Free
  - Easy To Learn
  - Can be used by anyone
  - Automatically identified threats
  - Automatically generates reports
  - Customizable threat templates
- Const
  - Platform dependent
  - Single user experience
  - Limited visual configurability
  - Limited prioritization capabilities; cannot do DREAD etc
  - Security controls not mapped
  - Limited examples available

#### 3.1.2. Process

- Identify security objectives
- Application overview
- Decompose the application
- Threat identification; STRIDE
- Identify vulnerabilities

#### 3.1.3. Work Flow

- Template
- Diagram
- Threat analysis
- Mitigation

### 3.2. Setup

#### 3.2.1. Installation

Download from [here](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool).

### 3.3. Preliminary Activities

- Security Objectives
  - What is the application used for?
  - Who uses the application?
  -  Are there compliance requirements?
  -  Is there data that needs to be protected?
  -  Does the application interact with our internal servers
  -  Are there any third parties involved?
  -  Are there any more specific questions?
  -  The answers will help you later!
-  Application Review
   - Users
     - Understand who the users will be
     - The users and roles such as internal, external and partners
   - Use Cases
     - Determine what the users are trying to do
     - Are they storing data, files, making payments, emails etc?
   - Technology
     - The underlying technology stack
     - Microsoft?
- Architectural Review
  -  External
     - Via web browsers  
  -  DMZ
     - Firewall  
  -  Internal
     - Web Server
     - Database Server

### 3.4. Data Flow Diagrams

- Key
  - External entity = Square
  - Data flow = Arrows
  - Process  = Circle
  - Data store = Two parallel lines
  - Trust boundary = Dash lines 

### 3.5. Identifying and Managing Threats

#### 3.5.1. Threat Types

- STRIDE
  -  Spoofing is impersonation of someone or something they are not
  -  Tampering is modification of something when they should not.
  -  Repudiation is the assurance of an an action has taken place
  -  Information disclosure is when information is accessed when it should not be
  -  Denial of Service is denying access or a service AKA DNS
  -  Elevation of privilege is when someone gets more rights
