# Domain 1: Detection

## Amazon Detective

### Overview
```mermaid
flowchart LR
    subgraph Sources ["Data Sources"]
        VPC["VPC Flow Logs"]
        CT["CloudTrail"]
        GD["GuardDuty Findings"]
        EKS["EKS Audit Logs"]
        SH["Security Hub"]
    end
    
    Sources --> DT[["Detective"]]
    
    DT -- "ML Analysis" --> Graph["Unified View\n(Graph DB)"]
    DT -- "investigate" --> UI["Detective Console"]
```

- **Analyze, investigate, and identify root cause** of security findings using:
  - Machine Learning (ML)
  - Graph theory
  - Statistics
- **Automatically collects and processes** events from:
  - VPC Flow Logs
  - CloudTrail
  - GuardDuty findings
- **Creates a unified view** of all activity
- **Optional data sources**: EKS audit logs, Security Hub, and more
- **Retention**: 1 year of aggregated data analysis

### What It Answers
- **How did it happen?**
- **What was affected?**
- **What actions did an attacker take?**

### View Details
- Affected resources
- When an IP connected to an EC2
- API calls
- Login attempts
- All activity timeline

## Investigation Process

### Investigating IAM Users/Roles
- Helps investigate IAM users/roles to determine if a principal is involved in a security event
- Example: Determine if a compromised IAM principal was used maliciously

### Investigation Workflow
```mermaid
flowchart TD
    GD["GuardDuty\nFinding"] --> DT["Detective"]
    
    DT --> Analysis["Analysis"]
    
    Analysis --> TP{"True Positive\nor\nFalse Positive?"}
    
    TP -->|"True Positive"| Scope["Define Scope"]
    TP -->|"False Positive"| Close["Close Investigation"]
    
    Scope --> Questions["Questions to Answer"]
    Questions --> Q1["What systems/users\nare compromised?"]
    Questions --> Q2["Where did the\nattack originate?"]
    Questions --> Q3["How long has the\nattack been ongoing?"]
    Questions --> Q4["What actions did\nthe attacker take?"]
    
    Q1 --> Response["Response Actions"]
    Q2 --> Response
    Q3 --> Response
    Q4 --> Response
    
    Response --> R1["Stop the attack"]
    Response --> R2["Minimize damage"]
    Response --> R3["Prevent similar\nattacks"]
```

## Architecture Example

### Scenario: CloudTrail Disabled
```mermaid
flowchart LR
    subgraph Attack ["Security Event"]
        Attacker["Attacker"] -->|"Disable CloudTrail"| CT["CloudTrail"]
    end
    
    GD["GuardDuty"] -- "finding" --> DT["Detective"]
    
    DT --> Analysis["Analyze Activity"]
    
    Analysis --> Determine["Determine if\nTrue Positive"]
    
    Determine -->|"Yes"| Scope["Define Scope\nof Attack"]
    Determine -->|"No"| Close["Mark as\nFalse Positive"]
    
    Scope --> Actions["Response & Remediation"]
    
    Actions --> Stop["Stop attack"]
    Actions --> Minimize["Minimize damage"]
    Actions --> Prevent["Prevent future attacks"]
```

### Investigation Steps
1. **GuardDuty generates a finding** when CloudTrail is disabled
2. **Detective ingests the finding**
3. **Analyze** whether activity is a true positive or false positive
4. **If True Positive**:
   - Define scope of malicious activity
   - Identify compromised systems and users
   - Identify attack origin
   - Determine attack duration
5. **Respond**:
   - Stop the attack
   - Minimize damage
   - Prevent similar attacks

## Data Collection

### Required Data Sources
- VPC Flow Logs
- CloudTrail
- GuardDuty findings

### Optional Data Sources
- EKS audit logs
- Security Hub findings
- Additional AWS services

### Analysis Features
- Machine learning models
- Graph-based behavior analysis
- Statistical analysis
- 365 days of historical data
