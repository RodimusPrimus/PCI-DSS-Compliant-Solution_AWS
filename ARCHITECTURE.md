# Architecture Documentation - PCI-DSS Compliant Agentic AI Solution

## System Overview

This document describes a **PCI-DSS compliant agentic AI microservices solution** deployed on AWS. The architecture leverages OpenShift container orchestration for ML models and LLMs, with secure on-premises database connectivity through site-to-site VPN and comprehensive security controls.

## Components Architecture

### 1. **On-Premises Environment**

#### Customer Data Center
- **LAN Network**: Internal network hosting business users
- **Customer Gateway**: Provides VPN connectivity to AWS
- **Users**: Business users accessing the solution through the Customer Gateway

### 2. **AWS Cloud Infrastructure (us-east-1 Region)**

#### Network Layer

**Public Subnet**
- **Internet Gateway**: Route traffic to/from the internet
- **Network Firewall**: Layer 7 inspection and filtering rules
- **VPN Gateway**: Termination point for site-to-site VPN connections
- **NAT Gateway**: Allows private subnet resources to reach the internet

**Private Subnet**
- **Network Firewall**: Additional layer of protection
- **ALB (Application Load Balancer)**: 
  - Distributes traffic to EC2 instances running ML agents
  - Health checks for agent availability
  - SSL/TLS termination
- **EC2 Instances**: Hosts for the OpenShift container cluster
- **WAF (Web Application Firewall)**: 
  - Protects against common web exploits
  - DDoS protection
  - Rate limiting

#### Security Services Layer

**Encryption & Secrets Management**
- **KMS (Key Management Service)**:
  - Manages encryption keys for data at rest
  - Controls key lifecycle and rotation
  - Audit trail via CloudTrail
  
- **ACM (AWS Certificate Manager)**:
  - Manages SSL/TLS certificates
  - Automatic certificate renewal
  - Certificate validation for HTTPS/TLS  
  
- **Private Certificate Authority**:
  - Issues internal certificates for microservices communication
  - Enables mutual TLS (mTLS) between services
  
- **Secrets Manager**:
  - Stores database credentials
  - Manages API keys and authentication tokens
  - Automatic secret rotation

**Monitoring & Compliance**
- **CloudWatch Logs**:
  - Centralized logging from all services
  - Log groups for each microservice
  - Custom metrics and alarms
  - Log retention policies
  
- **CloudTrail**:
  - Tracks all API calls and resource changes
  - Compliance audit trail
  - Detects unauthorized access attempts
  
- **Config**:
  - Continuously monitors AWS resource configurations
  - Compliance checking against security policies
  - Automated remediation triggers
  - Change tracking and history

#### VPN Connectivity

**VPN S2S (Site-to-Site)**
- Encrypted tunnel between customer datacenter and AWS
- IPSec encryption protocol
- AES-256 encryption standard
- Perfect Forward Secrecy (PFS) for key exchange

**Network Security**
- **Encryption in Transit**: All data flowing through VPN tunnel is encrypted
- **ACM/KMS Integration**: Certificate-based authentication and key management
- **VPN Redundancy**: Multiple tunnels for high availability

### 3. **Container & Compute Layer**

#### OpenShift Container Platform
- **Container Runtime**: Enterprise Kubernetes orchestration
- **Microservices Deployment**: Each service in isolated containers
- **Auto-Scaling Groups**: 
  - Automatically scales EC2 instances based on load
  - Minimum and maximum instance limits
  - Based on metrics from CloudWatch
  
- **Target Groups**:
  - Registers EC2 instances as targets
  - Health check configuration
  - Load distribution algorithm (round-robin, least connections)

#### AI/ML Components

**ML Models & LLMs**
- Containerized inference engines
- GPU-accelerated instances for inference
- Model versioning and rollback capabilities
- Batch and real-time inference capabilities

**Agentic Framework**
- Orchestrates multiple AI agents
- Handles request routing and load balancing
- State management across requests
- Fallback and retry mechanisms

### 4. **Database Layer**

#### On-Premises Secure Database
- **Database Authentication**: 
  - Role-based access control (RBAC)
  - Database user credentials stored in Secrets Manager
  - Session logging and audit trails
  
- **Encryption**:
  - At-rest encryption with KMS keys
  - TLS 1.2+ for all connections
  - End-to-end encryption from EC2 to on-premises DB
  
- **Access Control**:
  - Network-level: Security groups and network ACLs
  - VPN encryption for all traffic
  - IP whitelisting at firewall level

### 5. **Data Flow**

```
User/Client Request
    ↓
Internet Gateway/Customer Gateway
    ↓
VPN Gateway (Encrypted Tunnel)
    ↓
Network Firewall (Inspection)
    ↓
WAF (Application Firewall)
    ↓
ALB (Load Balancing)
    ↓
EC2 Instances (OpenShift Cluster)
    ↓
OpenShift Pods (ML Agents/LLMs)
    ↓
Database Query (Encrypted via VPN)
    ↓
On-Premises Database
    ↓
Response (Reverse Path - Encrypted)
```

## Security Architecture

### Defense in Depth

1. **Network Security**
   - VPN encryption for transit traffic
   - Network Firewall for packet inspection
   - Security Groups for instance-level controls
   - Network ACLs for subnet-level controls

2. **Application Security**
   - WAF for HTTP/HTTPS protection
   - ALB SSL/TLS termination
   - Internal mTLS via Private CA certificates

3. **Data Security**
   - KMS encryption at rest
   - AES-256 encryption for sensitive data
   - ACM managed certificates
   - Secrets Manager for credential management

4. **Audit & Compliance**
   - CloudTrail logs all API activity
   - CloudWatch logs application events
   - Config tracks resource compliance
   - Immutable audit trails

### Encryption Strategy

| Layer | Method | Keys |
|-------|--------|------|
| **In Transit (VPN)** | IPSec/TLS 1.2+ | AWS managed VPN keys |
| **In Transit (mTLS)** | TLS 1.3 | ACM Private CA certificates |
| **At Rest** | AES-256 | KMS Customer Master Keys (CMK) |
| **Database Auth** | Encrypted Credentials | Secrets Manager + KMS |

### PCI-DSS Compliance Controls

- **Requirement 2**: Secure configurations (Network Firewall, WAF)
- **Requirement 3**: Data protection (KMS, encryption)
- **Requirement 4**: Encryption (AES-256, TLS 1.2+)
- **Requirement 6**: Security testing (Config compliance checks)
- **Requirement 10**: Logging and monitoring (CloudTrail, CloudWatch)
- **Requirement 12**: Access control policies (Secrets Manager, IAM roles)

## High Availability & Disaster Recovery

### Availability Features
- **Multi-AZ Deployment**: ALB spans multiple availability zones
- **Auto-Scaling**: EC2 instances scale based on demand
- **Health Checks**: ALB performs active health monitoring
- **VPN Redundancy**: Multiple IPSec tunnels for failover

### Backup & Recovery
- **EBS Snapshots**: Regular snapshots of EC2 volumes
- **Database Backups**: On-premises database backup strategy
- **Configuration as Code**: Infrastructure reproducible from code
- **CloudTrail Logs**: Long-term retention for compliance

## Monitoring & Observability

### CloudWatch Integration
- **Application Metrics**: Custom metrics from OpenShift/containers
- **Infrastructure Metrics**: EC2 CPU, memory, network, disk
- **Log Groups**: Centralized logging from all components
- **Alarms**: Auto-scaling triggers and escalations

### CloudTrail Audit Logging
- **API Audits**: Who accessed what, when, and from where
- **Compliance Reports**: Automated reporting
- **Forensics**: Event history for investigations
- **Long-term Retention**: S3 storage for archive

### AWS Config Compliance
- **Resource Inventory**: Track all AWS resources
- **Compliance Rules**: Custom and managed rules
- **Change Tracking**: Know what changed and when
- **Remediation**: Automated correction of non-compliant resources

## Networking Diagram Flow

```
┌─────────────────────────┐
│   Customer DC           │
│ ┌───────────────────┐   │
│ │  Users (On-Prem)  │   │
│ │  Database         │   │
│ └────────┬──────────┘   │
│          │ VPN S2S      │
└──────────┼──────────────┘
           │
           ↓ (Encrypted)
     ┌──────────────┐
     │ VPN Gateway  │
     │ (IPSec)      │
     └──────┬───────┘
            ↓
     ┌──────────────┐
     │ Network FW   │
     │ (Public SN)  │
     └──────┬───────┘
            ↓
     ┌──────────────┐
     │    WAF/      │
     │    ALB       │
     └──────┬───────┘
            ↓
     ┌──────────────────────────────┐
     │ Private Subnet               │
     │ ┌──────────────────────────┐ │
     │ │   OpenShift Cluster      │ │
     │ │  ┌────────────────────┐  │ │
     │ │  │ ML Models/LLMs     │  │ │
     │ │  │ (in Containers)    │  │ │
     │ │  │ - Auto-Scaling     │  │ │
     │ │  │ - Target Group     │  │ │
     │ │  └────────────────────┘  │ │
     │ └──────────────────────────┘ │
     └──────────────────────────────┘
```

## Service Communication Security

### Inter-Service Communication (OpenShift)
- **mTLS Enforcement**: Service mesh with mutual TLS
- **Certificate Rotation**: Automated by Private CA
- **Network Policies**: Kubernetes network policies for segmentation

### External Communication
- **API Gateway**: ALB with WAF protection
- **TLS Termination**: At ALB (443 → 8443)
- **Certificate Management**: ACM for public certificates

## Disaster Recovery Plan

### Backup Strategy
- **Daily EBS Snapshots**: EC2 volume backups
- **Database Replication**: To secondary on-premises location
- **CloudTrail Logs**: S3 with versioning and lifecycle
- **Configuration Backups**: IaC stored in version control

### Recovery Procedures
- **RTO (Recovery Time Objective)**: < 2 hours
- **RPO (Recovery Point Objective)**: < 1 hour
- **Automated Failover**: VPN tunnel redundancy
- **Manual Procedures**: Documented runbooks

## Cost Optimization

- **Reserved Instances**: EC2 instances with reserved capacity
- **Spot Instances**: Non-critical workloads on spot
- **Auto-Scaling**: Right-sizing based on demand
- **Data Transfer**: Minimize inter-region traffic
- **Log Retention**: Tiered storage strategy

## Future Enhancements

1. **Service Mesh**: Implement Istio for advanced traffic management
2. **Gitops**: ArgoCD for declarative deployments
3. **API Gateway**: Add Amazon API Gateway for additional protection
4. **Multi-Region**: Active-active deployment across regions
5. **AI Monitoring**: Custom metrics for model performance
6. **Policy as Code**: OPA/Gatekeeper for policy enforcement
