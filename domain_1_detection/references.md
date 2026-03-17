# Domain 1: Detection

## GuardDuty

### Service Architecture
```mermaid
flowchart LR
    subgraph DataSources ["Data Sources (Primary)"]
        direction TB
        VPC([VPC Flow Logs])
        CT([CloudTrail Logs])
        R53([Route 53 DNS Query Logs])
    end

    subgraph Optional ["Optional Features"]
        direction TB
        S3([S3 Logs])
        EBS([EBS Volumes])
        LNA([Lambda Network Activity])
        RDS([RDS & Aurora Login Activity])
        EKS([EKS Audit Logs & Runtime Monitoring])
    end

    DataSources --> GD[["GuardDuty"]]
    Optional --> GD

    GD -- "findings" --> EB["EventBridge"]

    EB -- "trigger" --> SNS["SNS"]
    EB -- "invoke" --> Lambda["Lambda"]
```

### Overview
- **Threat detection service** that uses AI/ML, anomaly detection, and malicious file discovery
- **No software to install** - one-click enable
- **Findings** can be sent to EventBridge for automation via Lambda + SNS

### What It Analyzes
| Data Source | What It Detects |
|-------------|-----------------|
| CloudTrail Management Events | Unusual API calls, unauthorized deployments |
| VPC Flow Logs | IP addresses, strange/malicious traffic |
| R53 DNS Query Logs | Compromised EC2 sending encoded data via DNS queries |

### Extended Threat Protection
- Detects **multi-stage attacks** spanning multiple data sources and AWS resources

### Use Cases

#### 1. Detect Compromised EC2/ECS/EKS
- Malware activity / crypto mining
- Malicious domain communications
- TOR networks
- Reverse shells

#### 2. Anomalous Network Traffic
- Unusual/suspicious network traffic
- Malicious IP communication
- Data exfiltration
- Port scanning / brute force
- Outbound port scans

#### 3. Compromised AWS Account
- Suspicious use of AWS account resources
- API calls from unfamiliar IP addresses
- Compromised IAM credentials
- IAM role enumeration
- Attempts to disable CloudTrail logging

### Protection Plans (Optional Extensions)

| Plan | Description |
|------|-------------|
| Malware Protection for EC2 | Scans EBS volumes to detect malware |
| EKS Protection | Monitors Kubernetes audit logs from EKS clusters |
| Runtime Monitoring | OS-level events on EC2/ECS/EKS |
| Lambda Protection | Lambda network activity monitoring (via VPC flow logs) |
| S3 Protection | Identifies security risks for S3 buckets |
| Malware Protection for S3 | Scans uploaded objects to detect malware |
| Malware Protection for AWS Backups | Scans backups (EBS snapshots, AMIs, etc.) |
| RDS Protection | Analyzes RDS/Aurora login activity for access threats |

## GuardDuty Findings & Automations

### Findings Characteristics
- **Data Streams**: Pulls independent streams directly from CloudTrail (management & data events), VPC Flow Logs, and EKS logs.
- **Severity Scores**: Range from **0.1 to 8.9+**.
    - **Low**: 0.1 – 3.9
    - **Medium**: 4.0 – 6.9
    - **High**: 7.0 – 8.9+
- **Naming Convention**: `ThreatPurpose:ResourceTypeAffected/ThreatFamilyName.DetectionMechanism!Artifact`
- **Testing**: Generate **sample findings** in the GuardDuty console to verify automation workflows.
- **Investigation**: Use the console to identify the specific **API** used to invoke a finding.

### Automated Response
- **EventBridge Integration**: Primary mechanism for automated security responses.
- **Multi-Account**: Events are published to both the **administrator account** and the **member account** of origin.
- **Workflow Example**: GuardDuty Findings → EventBridge (Filter for High/Critical) → SNS Topic → Administrator Notification.

#### Automation Architecture
```mermaid
flowchart TD
    GD[["GuardDuty"]] -- "finding" --> EB["EventBridge"]
    
    subgraph Targets ["Automation Targets"]
        direction LR
        EB -- "trigger" --> SQS["SQS"]
        EB -- "trigger" --> SNS["SNS"]
        EB -- "trigger" --> Lambda["Lambda"]
    end

    SNS -- "invoke" --> HTTP["HTTP (e.g. Slack)"]
    SNS -- "invoke" --> Email["Email"]
```

## GuardDuty Multi-Account Strategy

- Use an AWS Organization to manage GuardDuty across multiple accounts
- Invite member accounts through GuardDuty
- In an AWS Org - you can specify a member account as the organization's **delegated admin** for GuardDuty

```mermaid
flowchart TB
    subgraph Org ["AWS Organization"]
        Admin["Delegated Admin Account"]
    end
    
    subgraph Members ["Member Accounts"]
        M1["Account A"]
        M2["Account B"]
        M3["Account C"]
    end
    
    Admin -- "manage" --> M1
    Admin -- "manage" --> M2
    Admin -- "manage" --> M3
```

## GuardDuty Architectures

### Architecture 1: Basic Notification Flow

```mermaid
flowchart LR
    EC2["EC2 Web Server"]
    VPC["VPC Flow Logs"]
    GD["GuardDuty"]
    EB["EventBridge"]
    SNS["SNS"]
    Admin["Security Admin"]
    
    EC2 --> VPC
    VPC --> GD
    GD -- "finding" --> EB
    EB -- "notify" --> SNS
    SNS --> Admin
```

### Architecture 2: Auto-Blocking Malicious IPs via NACL

```mermaid
flowchart TD
    EC2["EC2 Web Server"]
    VPC["VPC Flow Logs"]
    GD["GuardDuty"]
    EB["EventBridge"]
    Lambda["Lambda"]
    NACL["NACL"]
    Admin["Security Admin"]
    
    EC2 --> VPC
    VPC --> GD
    GD -- "finding" --> EB
    EB -- "invoke" --> Lambda
    Lambda -- "block IP" --> NACL
    Lambda -- "notify" --> Admin
```

### Architecture 3: AWS Firewall Manager with IP Blocking

```mermaid
flowchart TD
    subgraph VPC ["VPC"]
        FW["Firewall Subnet"]
        EC2["EC2 Instance"]
    end
    
    VPC["VPC Flow Logs"]
    GD["GuardDuty"]
    EB["EventBridge"]
    Lambda["Lambda"]
    IPDB["Malicious IP Database"]
    SNS["SNS"]
    Admin["Security Admin"]
    
    EC2 --> VPC
    VPC --> GD
    GD -- "finding" --> EB
    EB -- "invoke" --> Lambda
    Lambda -- "check" --> IPDB
    IPDB -- "malicious" --> Lambda
    Lambda -- "add rule" --> FW
    Lambda -- "notify" --> SNS
    SNS --> Admin
```

## GuardDuty Trusted and Threat IP Lists

- Works only for **public IP addresses**
- **Trusted IP List**:
    - List of IP addresses + CIDR ranges that you trust
    - GuardDuty will NOT generate findings for traffic involving these trusted IPs
- **Threat IP List**:
    - List of known malicious IP addresses and CIDR ranges
    - GuardDuty generates findings based on threat list matches
    - Can be supplied by 3rd party threat intelligence or created custom
- **Multi-Account**: Only the GuardDuty administrator account can manage these lists

```mermaid
flowchart LR
    subgraph Admin ["GuardDuty Admin"]
        TL["Trusted IP List"]
        ThL["Threat IP List"]
    end
    
    GD["GuardDuty"]
    
    TL -.->|"exclude from findings"| GD
    ThL -.->|"generate findings"| GD
```

## GuardDuty Suppression Rules

- Automatically filter and archive new findings based on defined criteria
- Use cases:
    - Low-value findings you don't need to act on
    - Known false positives
    - Threats you don't intend to respond to
- Can suppress:
    - Entire finding type (e.g., all findings of type `Trojan`)
    - More granular criteria (e.g., suppress findings only for specific EC2 instances)
- **Important**: Suppressed findings are NOT sent to:
    - Security Hub
    - S3
    - Amazon Detective
    - EventBridge
- You can still view suppressed findings in the archive

```mermaid
flowchart TD
    GD["GuardDuty"]
    EB["EventBridge"]
    Suppress["Suppression Rules"]
    Archive["Archive"]
    
    GD -- "finding" --> Suppress
    Suppress -- "match" --> Archive
    Suppress -- "no match" --> EB
```

## GuardDuty Troubleshooting

- If GuardDuty is not generating any findings (or certain finding types):
- **Example**: GuardDuty is enabled but not generating DNS-based findings
    - GuardDuty only processes DNS logs if you use the **default VPC DNS resolver**
    - Custom DNS resolvers will NOT generate DNS-based findings
- **Best Practice**: Enable GuardDuty even in AWS regions you don't actively use
    - Attackers may use unused regions to evade detection

