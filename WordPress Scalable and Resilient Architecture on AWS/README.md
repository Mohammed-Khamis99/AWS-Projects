# WordPress Scalable and Resilient Architecture on AWS

This project demonstrates the evolution of a WordPress web application from a single instance deployment to a scalable, highly available architecture on AWS. The final architecture leverages multiple AWS services to ensure resilience, auto-scaling, and separation of concerns across tiers.

#[Architecture Diagram]

![Image](https://github.com/user-attachments/assets/9728a460-552c-4e09-a110-6eeb457d3d09)

## 📋 Project Overview
**Evolution Journey:**
1. **Phase 1:** Manual single EC2 instance with Apache+WordPress and MySQL (monolithic)
2. **Phase 2:** Separated web tier (EC2) and database tier (RDS)
3. **Phase 3:** Introduced EFS for shared storage, Auto Scaling Groups, and Load Balancer
4. **Final Architecture:** Multi-AZ deployment with 3-tier VPC architecture

## 🏗️ Architecture Components
**AWS Network Infrastructure:**
- **VPC** with 3 Availability Zones
- **9 Subnets** (3 Public, 6 Private) across three tiers:
  - **Web Tier:** Public subnets for EC2 instances + Application Load Balancer
  - **App Tier:** Private subnets with EFS storage
  - **Database Tier:** Private subnets with RDS MySQL (Multi-AZ)

**Core Services:**
- **EC2 Auto Scaling Group:** Web servers running WordPress/PHP/Apache
- **Application Load Balancer:** Distributes traffic across web tier instances
- **Amazon EFS:** Central storage for WordPress files (wp-content)
- **RDS MySQL:** Managed relational database with Multi-AZ deployment
- **AWS Certificate Manager:** SSL/TLS certificates for HTTPS
- **Route 53:** DNS management (optional)

## 🔑 Key Features
✅ **High Availability**
- Multi-AZ deployment for all critical components
- RDS MySQL with standby replica in different AZ
- EFS storage replicated across 3 AZs

✅ **Auto Scaling**
- Web tier scales between 2-8 EC2 instances based on CPU utilization
- Automatic replacement of unhealthy instances

✅ **Security**
- Security groups with least-privilege access
- Web tier in public subnets with restricted SSH access
- App and Database tiers in private subnets
- Encrypted EBS volumes and RDS storage

✅ **Resilient Storage**
- EFS provides shared file storage for WordPress
- Automatic backups with RDS snapshots
- Version-controlled WordPress core/plugins

## 🚀 Deployment Steps
1. **Infrastructure Setup**


