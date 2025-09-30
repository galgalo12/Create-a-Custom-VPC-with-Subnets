# AWS Custom VPC with Public and Private Subnets

## Summary
This project demonstrates how to create a **custom VPC in AWS** with both public and private subnets, and configure essential networking components for secure and scalable workloads.  

The setup includes:
- **Custom VPC** with CIDR block `10.8.0.0/16`
- **Public and Private Subnets** across multiple Availability Zones
- **Internet Gateway (IGW)** for public subnet internet access
- **NAT Gateway** for private subnet outbound internet access
- **Route Tables** for controlling traffic between subnets
- **Security Groups** for managing inbound and outbound traffic rules
- **EC2 Instances** deployed in both public and private subnets for testing

This architecture ensures:
- Public-facing resources (e.g., bastion host, web servers) are accessible from the internet.
- Private resources (e.g., application and database servers) remain isolated but can reach the internet securely via the NAT Gateway.
- High availability and scalability across multiple Availability Zones.

---

## Architecture Diagram

### ASCII Diagram
               +-------------------------+
               |       Internet          |
               +------------+------------+
                            |
                            v
                +-----------------------+
                |   Internet Gateway    |
                +-----------+-----------+
                            |
              Public Route Table (0.0.0.0/0 → IGW)
                            |
     +----------------------+--------------------+
     |                                           |
+---------------+ +---------------+
| Public Subnet | | Public Subnet |
| 10.0.1.0/24 | | 10.0.2.0/24 |
| EC2 (Bastion)| | NAT Gateway |
+---------------+ +---------------+
|
v
Private Route Table (0.0.0.0/0 → NAT GW)
|
+---------------------------+---------------------+
| |
+---------------+ +---------------+
| Private Subnet| | Private Subnet|
| 10.0.3.0/24 | | 10.0.4.0/24 |
| EC2 (App) | | EC2 (DB) |
+---------------+ +---------------+


Use Cases

This VPC design is a common best practice for secure and scalable AWS environments. It can be applied in different scenarios, such as:

Web Application Hosting

Host web servers in public subnets for internet access.

Place application servers and databases in private subnets for security.

Bastion Host Setup

Use a bastion/jump server in a public subnet to securely connect to private subnet instances.

SOC / Security Lab Environments

Deploy monitoring and detection tools in private subnets.

Simulate attacks and defenses in a safe, isolated lab network.

Hybrid Cloud Networking

Extend on-premise infrastructure to AWS with VPN/Direct Connect.

Keep sensitive workloads in private subnets while exposing only necessary services.

Testing and Learning

Practice AWS networking fundamentals (VPCs, subnets, IGWs, NAT Gateways, route tables, security groups).

Validate secure patterns before deploying into production.
