# Domain 1: Detection

## Amazon CloudWatch Logs

### Overview
```mermaid
flowchart LR
    subgraph Sources ["Log Sources"]
        SDK["SDK"]
        Agent["CloudWatch Agent\n(Unified/Classic)"]
        EB["Elastic Beanstalk"]
        ECS["ECS Containers"]
        Lambda["Lambda Functions"]
        VPC["VPC Flow Logs"]
        APIGW["API Gateway"]
        CT["CloudTrail"]
        R53["Route53 DNS Queries"]
    end
    
    LG["Log Group"] --> LS["Log Stream"]
    LG --> Dest["Destinations"]
    
    Sources --> LG
```

- Centralized logging service for AWS and on-premises applications
- **Log Groups**: Named containers for application logs (e.g., one per application)
- **Log Streams**: Instances within a log group (specific containers, EC2 instances, log files)
- **Retention**: 1 day to 10 years, or indefinitely

### Log Sources

| Source | Description |
|--------|-------------|
| SDK | Send logs directly via API |
| CloudWatch Agent / Unified Agent | Collect logs from EC2/on-premises |
| Elastic Beanstalk | Application logs |
| ECS | Container logs |
| Lambda | Function execution logs |
| VPC Flow Logs | VPC network traffic metadata |
| API Gateway | API request logs |
| CloudTrail | API call logs (filter-based) |
| Route53 | DNS query logs |

### Encryption
- Logs encrypted by default using AWS-managed keys
- Optional: Use your own KMS customer-managed keys (CMK)

### Querying Logs

#### CloudWatch Logs Insights
```mermaid
flowchart LR
    Query["Write Query"] --> Insights["Logs Insights"]
    Insights --> Viz["Visualization"]
    Insights --> LogLines["Log Lines"]
    Viz --> Dashboard["Dashboard"]
    Query --> Timeframe["Timeframe"]
```

- Purpose-built query language for CloudWatch Logs
- Automatically detects fields from log data
- Features:
  - Filter by conditions
  - Calculate aggregate statistics
  - Sort and limit events
  - Save queries
  - Add to CloudWatch Dashboards
  - Query multiple log groups (including cross-account)

#### Example Queries
- Most 25 recent events
- Count events with exceptions/errors
- Filter by specific IP address

> **Note**: CloudWatch Logs Insights queries historical data only - not real-time

### Export & Subscriptions

#### S3 Export (Batch)
```mermaid
flowchart LR
    LG["Log Group"] --> CreateExportTask["CreateExportTask"]
    CreateExportTask --> S3[(S3 Bucket)]
```
- Batch export to S3
- Can take up to **12 hours** to complete
- Use `CreateExportTask` API

#### Real-Time Streaming (Subscriptions)
```mermaid
flowchart LR
    LG["Log Group"] --> Filter["Subscription Filter"]
    
    Filter --> KDS["Kinesis Data Streams"]
    Filter --> KDF["Kinesis Data Firehose"]
    Filter --> Lambda["Lambda"]
    
    KDS --> KDF
    KDF --> S3["S3"]
    KDF --> OpenSearch["OpenSearch Service"]
```

- **Real-time** delivery of log events
- Subscription filters define which events to send
- Destinations: Kinesis Data Streams, Kinesis Data Firehose, Lambda

### Cross-Account Log Aggregation

```mermaid
flowchart TD
    subgraph AccountA ["Sender Account"]
        LG["Log Group"] --> Sub["Subscription Filter"]
    end
    
    subgraph AccountB ["Recipient Account"]
        KDS["Kinesis Data Stream"]
        KDF["Kinesis Data Firehose"]
        S3[(S3 Bucket)]
    end
    
    Sub -->|"Subscription Destination"| KDS
    KDS --> KDF
    KDF --> S3
```

**Setup Process**:
1. Create subscription filter in sender account
2. Create destination (Kinesis Data Stream) in recipient account
3. Attach destination access policy to allow sender
4. Create IAM role in recipient account with permission to send to Kinesis
5. Allow sender account to assume the role

### Retention Policy

| Storage Tier | Retention |
|--------------|-----------|
| CloudWatch Logs | 1 day to 10 years |
| S3 (via export/subscription) | Custom |
| S3 Glacier | Long-term archival |

### Exam Tips

- **Log Groups** = applications
- **Log Streams** = instances/files/containers within an app
- Subscription filters enable **real-time** log streaming
- Export to S3 is **batch** (up to 12 hours)
- Logs Insights queries are **not real-time**
- Cross-account aggregation uses Kinesis Data Streams + Firehose
- Default encryption is AWS-managed; can use CMK

---

## CloudWatch Alarms

### Overview
```mermaid
flowchart LR
    Metric["Metric"] --> Alarm["CloudWatch Alarm"]
    
    Alarm --> States{Alarm States}
    States --> OK["OK"]
    States --> INSUFF["INSUFFICIENT_DATA"]
    States --> ALARM["ALARM"]
    
    Alarm --> Targets["Alarm Targets"]
    Targets --> EC2["EC2 Actions\n(stop/terminate/reboot/recover)"]
    Targets --> AS["Auto Scaling"]
    Targets --> SNS["SNS Notification"]
```

- Triggers notifications for any CloudWatch metric
- Evaluation options: sampling, percentage, max, min, etc.

### Alarm States

| State | Description |
|-------|-------------|
| **OK** | Metric is within defined threshold |
| **ALARM** | Metric exceeds threshold |
| **INSUFFICIENT_DATA** | Not enough data available (starting up or missing metrics) |

### Period & Resolution
- **Period**: Length of time in seconds to evaluate the metric
- **High Resolution Metrics**: Support 10s, 30s, or multiples of 60 seconds

### Alarm Targets

| Target | Actions |
|--------|---------|
| **EC2 Instance** | Stop, Terminate, Reboot, Recover |
| **Auto Scaling** | Trigger scale-in/scale-out actions |
| **SNS** | Send notification (then trigger Lambda, email, etc.) |

### Composite Alarms
```mermaid
flowchart TD
    subgraph Conditions ["Alarm Conditions"]
        CPU["CPU High Alarm"]
        Net["Network High Alarm"]
    end
    
    Conditions --> Composite["Composite Alarm"]
    Composite -->|"AND/OR"| Alert["Alert"]
    
    Composite -.->|"reduce noise"| Noise["Reduce Alarm Noise"]
```

- Monitors state of **multiple other alarms**
- Supports **AND** and **OR** conditions
- Reduces alarm fatigue by creating complex conditions
- Example: Trigger alert only when CPU is high **AND** network traffic is high

### EC2 Instance Recovery

```mermaid
flowchart LR
    subgraph StatusChecks ["Status Checks"]
        Instance["Instance Status\n(VM health)"]
        System["System Status\n(hardware)"]
        EBS["EBS Status\n(volume health)"]
    end
    
    StatusChecks --> Recover["Recover Instance"]
    Recover --> SameIP["Same Private IP"]
    Recover --> SameEIP["Same Elastic IP"]
    Recover --> SameMeta["Same Metadata"]
    Recover --> SamePG["Same Placement Group"]
```

- **Instance Status**: Checks EC2 virtual machine
- **System Status**: Checks underlying hardware
- **EBS Status**: Checks attached EBS volumes
- **Recovery**: Preserves private IP, public IP, Elastic IP, metadata, placement group

### Alarm Good to Know

- Can create alarms based on **CloudWatch Logs metric filters**
- Test alarms using CLI:
  ```bash
  aws cloudwatch set-alarm-state \
    --alarm-name "myalarm" \
    --state-value ALARM \
    --state-reason "testing"
  ```
- Alarm actions require correct IAM permissions
- Can set alarm to trigger on any metric namespace (CWAgent, EC2, Lambda, etc.)

---

## CloudWatch Contributor Insights

### Overview
```mermaid
flowchart LR
    subgraph LogSources ["Log Sources"]
        VPC["VPC Flow Logs"]
        DNS["DNS Logs"]
        CW["CloudWatch Logs"]
    end
    
    LogSources --> CI["Contributor Insights"]
    CI --> TimeSeries["Time Series"]
    CI --> TopTalkers["Top Talkers"]
    
    TopTalkers --> Hosts["Bad Hosts"]
    TopTalkers --> Network["Heaviest Network Users"]
    TopTalkers --> URLs["URLs with Most Errors"]
```

- Analyzes log data and creates **time series** of contributor data
- Identifies **top talkers** and what/who is impacting system performance

### Use Cases

| Use Case | Description |
|----------|-------------|
| Find Bad Hosts | Identify hosts causing errors |
| Heaviest Network Users | Find top bandwidth consumers |
| Top Error URLs | Identify URLs generating most errors |
| Throttling | Find API endpoints being throttled |

### Works With
- **Any AWS-generated logs**: VPC Flow Logs, DNS logs, etc.
- CloudWatch Logs (custom application logs)

### Rules

```mermaid
flowchart LR
    Rules[Rules] --> BuiltIn["AWS Built-in Rules"]
    Rules --> Custom["Custom Rules"]
    
    BuiltIn -->|"leverages"| CW["CloudWatch Logs"]
    Custom --> CW
```

- **Built-in Rules**: Created by AWS, leverages your existing CloudWatch Logs
- **Custom Rules**: Create your own rules for specific analysis

### Example: VPC Flow Logs

```
VPC Flow Logs → CloudWatch Logs → Contributor Insights → Top 10 IP Addresses
```

### Key Features
- Real-time contributor data visualization
- Time series charts showing top contributors
- Works across multiple log groups
- Can be used to create alarms on top contributors