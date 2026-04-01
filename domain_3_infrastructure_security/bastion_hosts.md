# Bastion Hosts

## Overview
A **Bastion Host** (also known as a "jump box") is a special-purpose EC2 instance used to provide secure access to Linux or Windows instances located in a private subnet. Since private instances lack a public IP and are not reachable from the internet, the bastion host acts as a gateway, sitting in a public subnet to bridge the gap between the internet and the private network.

## Key Concepts
- **Public Subnet**: The subnet where the bastion host resides, which has a route to an Internet Gateway (IGW).
- **Private Subnet**: The subnet where backend application instances reside, with no direct route to the internet.
- **SSH Forwarding / Jump**: The process of connecting to the bastion and then "jumping" to the private instance.
- **Security Groups**: Granular firewall rules that control traffic between the user, the bastion, and the private instances.

## Detailed Notes

### 1. Networking Requirements
- **Bastion Host**: Must have a **Public IP** and reside in a **Public Subnet**.
- **Private Instances**: Reside in a **Private Subnet** (no public IP).
- **Connectivity**: Both the bastion and the private instances must be in the same VPC (or peered VPCs) to communicate via private IP addresses.

### 2. Security Group Configuration
Properly configuring security groups is critical to maintaining the security of the bastion host.

#### Bastion Host Security Group (Public)
- **Inbound**: Allow **SSH (Port 22)** or **RDP (Port 3389)** only from specific, trusted CIDR blocks (e.g., corporate office IP). Avoid `0.0.0.0/0`.
- **Outbound**: Allow **SSH/RDP** to the private security group.

#### Private Instance Security Group (Private)
- **Inbound**: Allow **SSH (Port 22)** or **RDP (Port 3389)** only from the **Security Group ID** of the bastion host or the bastion's **Private IP**.
- **Outbound**: As required for the application.

### 3. Connection Process (SSH Example)
1. **User to Bastion**: `ssh -i key.pem ec2-user@bastion-public-ip`
2. **Bastion to Private**: From the bastion shell, `ssh -i key.pem ec2-user@private-ip`
    - > **Note**: This requires placing the private key on the bastion host, which is generally discouraged.
    - > **Preferred Method**: Use **SSH Agent Forwarding** (`ssh -A`) to avoid storing keys on the bastion.

## Architecture / Flow

```mermaid
graph LR
    subgraph "Public Internet"
        User["Administrator<br>(Laptop)"]
    end

    subgraph "AWS VPC"
        subgraph "Public Subnet"
            Bastion["Bastion Host<br>(Public IP)"]
        end

        subgraph "Private Subnet"
            PrivateEC2["Private EC2 Instance<br>(Private IP)"]
        end
    end

    User -->|1. SSH (Port 22)| Bastion
    Bastion -->|2. SSH (Port 22)| PrivateEC2

    style Bastion fill:#f9f,stroke:#333,stroke-width:2px
```

## Security Relevance
- **Preventive Control**: Limits the attack surface by ensuring only one hardened instance is exposed to the internet.
- **Hardening**: Bastion hosts should be stripped of unnecessary software, regularly patched, and have logging enabled.
- **Audit Trail**: Connection logs on the bastion host provide an audit trail of who accessed the internal network.

## Operational / Real-World Context
- **Maintenance**: Bastion hosts must be kept up-to-date. If the bastion is compromised, the entire private network is at risk.
- **Alternatives**: **AWS Systems Manager (SSM) Session Manager** is the modern, more secure alternative as it removes the need for bastion hosts, public IPs, and open inbound ports.

## Common Pitfalls / Misconfigurations
- **Allowing 0.0.0.0/0**: Opening SSH to the world on the bastion host.
- **Storing Private Keys**: Storing `.pem` files on the bastion host disk. Use Agent Forwarding instead.
- **Missing Outbound Rules**: If the bastion cannot talk to the private instance on port 22, the "jump" will fail.
- **Lack of Logging**: Failing to monitor shell activity or login attempts on the bastion.

## Exam / Review Notes
- **Placement**: Bastion must be in a **Public Subnet**.
- **Security Groups**: Private instances should only allow SSH from the **Bastion's Security Group**.
- **SSH Agent Forwarding**: Know that this is the secure way to handle keys.
- **Comparison with NAT Gateway**: 
    - **Bastion Host**: For **inbound** management (SSH/RDP).
    - **NAT Gateway**: For **outbound** internet access (updates/patches) from private instances.

## Summary
A Bastion Host is a gateway instance that allows administrators to securely reach private instances via SSH or RDP. It requires a public IP and a carefully restricted security group to minimize the risk of being a single point of failure.

## Quick Review Checklist
- [ ] Bastion is in a public subnet with a public IP.
- [ ] Bastion SG restricted to trusted admin IPs.
- [ ] Private instance SG allows SSH/RDP only from Bastion SG.
- [ ] SSH Agent Forwarding used (keys not stored on bastion).
- [ ] NAT Gateway is used for *outbound* traffic, Bastion for *inbound* access.
