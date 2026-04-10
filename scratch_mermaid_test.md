# Mermaid Diagram Style Test

## 1. architecture-beta — ECS Private Image Pull (Full Path)

```mermaid
architecture-beta
    group vpc(cloud)[VPC Private Subnet]
        service ecs(server)[ECS Task] in vpc
        service vpce_ecr(internet)[ECR Endpoint] in vpc
        service vpce_s3(disk)[S3 Endpoint] in vpc

    group registry(cloud)[AWS Services]
        service iam(server)[Task Execution Role] in registry
        service ecr(database)[ECR Registry] in registry
        service s3(disk)[S3 Image Layers] in registry

    iam:R --> L:ecs
    ecs:R --> L:vpce_ecr
    ecs:B --> T:vpce_s3
    vpce_ecr:R --> L:ecr
    vpce_s3:R --> L:s3
```

---

## 2. flowchart — Request/Data Path with Trust Boundaries (GuardDuty Remediation)

```mermaid
flowchart LR
    subgraph Workload_Account ["Workload Account"]
        EC2["EC2 Instance"]
        VPC_FL["VPC Flow Logs"]
    end

    subgraph Security_Account ["Security Account (Trust Boundary)"]
        GD["GuardDuty"]
        EB["EventBridge Rule<br>[source: guardduty]"]
        SF["Step Functions<br>[remediation workflow]"]
        NFW["Network Firewall<br>[block malicious IP]"]
    end

    EC2 -- "network traffic" --> VPC_FL
    VPC_FL -- "log stream" --> GD
    GD -- "finding: UnauthorizedAccess:EC2/SSHBruteForce" --> EB
    EB -- "invoke [severity=HIGH]" --> SF
    SF -- "UpdateRuleGroup [deny srcIP]" --> NFW
    SF -- "notify [success/fail]" --> SNS["SNS: Security Team"]
```

---

## 3. sequenceDiagram — Auth Flow (GitHub OIDC → AWS STS → ECR Deploy)

```mermaid
sequenceDiagram
    participant GHA as GitHub Actions
    participant OIDC as GitHub OIDC Provider
    participant STS as AWS STS
    participant ECR as Amazon ECR
    participant ECS as Amazon ECS

    GHA->>OIDC: request JWT (workflow, repo, ref claims)
    OIDC-->>GHA: signed JWT token

    GHA->>STS: AssumeRoleWithWebIdentity(JWT, RoleArn)
    Note over STS: validates JWT issuer + audience<br>checks role trust policy conditions
    STS-->>GHA: temporary credentials (15min)

    GHA->>ECR: GetAuthorizationToken (temp creds)
    ECR-->>GHA: docker login token (12h)

    GHA->>ECR: docker push (image:digest)
    ECR-->>GHA: push confirmed

    GHA->>ECS: RegisterTaskDefinition (image digest)
    GHA->>ECS: UpdateService (force new deploy)
    ECS-->>GHA: deployment initiated
```

---

## 4. flowchart with subgraph account/VPC/subnet blocks — KMS Decrypt Trust Chain (ECS + Secrets Manager + CMK)

```mermaid
flowchart TD
    subgraph Management_Account ["Management Account"]
        KMS["KMS CMK<br>[Key Policy: allow execution role]"]
    end

    subgraph App_Account ["Application Account"]
        subgraph Control_Plane ["ECS Control Plane"]
            TER["Task Execution Role<br>[secretsmanager:GetSecretValue<br>kms:Decrypt]"]
        end

        subgraph VPC ["VPC"]
            subgraph Private_Subnet ["Private Subnet"]
                Agent["ECS Agent"]
                Container["Container Process<br>[env: DB_PASSWORD=***]"]
            end
            VPCE_SM["Secrets Manager<br>Interface Endpoint"]
            VPCE_KMS["KMS<br>Interface Endpoint"]
        end

        ASM["Secrets Manager<br>[secret ARN: arn:aws:secretsmanager:...]"]
    end

    Agent -- "1. AssumeRole" --> TER
    TER -- "2. GetSecretValue (ARN)" --> VPCE_SM
    VPCE_SM --> ASM
    ASM -- "3. Decrypt (CMK)" --> VPCE_KMS
    VPCE_KMS --> KMS
    KMS -- "4. return plaintext" --> ASM
    ASM -- "5. return secret value" --> Agent
    Agent -- "6. inject env var" --> Container
```
