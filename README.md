# Secure Multi-Tier Network Architecture on AWS
This project demonstrates how to design and deploy a secure, scalable, and highly available AWS VPC architecture suitable for a production environment.
The architecture uses multiple Availability Zones (AZs) to improve fault tolerance and ensures that application servers are not exposed to the public internet. Instead, traffic is managed through a load balancer, while outbound internet access is controlled via NAT Gateways.

# Diagram
<img width="1536" height="1024" alt="project diagram" src="https://github.com/user-attachments/assets/7f8f509d-b5aa-466b-a9c9-af1234574625" />

## Step 1: VPC Creation
Created a custom VPC to isolate the project's network from the default VPC.

- **Name:** `multi-tier-vpc`
- **CIDR Block:** `10.0.0.0/16` (65,536 IPs)
<img width="960" height="409" alt="1" src="https://github.com/user-attachments/assets/c50df9f8-4c7f-4db8-9561-33657b4cdddc" />
<img width="960" height="418" alt="2" src="https://github.com/user-attachments/assets/721023e3-8f93-49a9-96d5-415015a7c99b" />


## Step 2: Subnet Design
Split into 4 subnets across 2 Availability Zones for high availability.
<img width="960" height="410" alt="3" src="https://github.com/user-attachments/assets/2e28445d-7b82-4ebe-9613-124c7546a81b" />

## Step 3: Internet Gateway
Created and attached an IGW to allow public subnet resources to reach the internet.

- **Name:** `multi-tier-igw`
- **Attached to:** `multi-tier-vpc`
<img width="960" height="417" alt="4" src="https://github.com/user-attachments/assets/4b8862a0-84a3-45ae-8afd-594ef1d5d244" />


## Step 4: NAT Gateways
Deployed one NAT Gateway per AZ (not a single shared NAT) so that outbound 
internet access for private instances doesn't depend on a single AZ's availability.

- **NAT-A** → Public-Subnet-A (Elastic IP attached)
- **NAT-B** → Public-Subnet-B (Elastic IP attached)
<img width="960" height="421" alt="5" src="https://github.com/user-attachments/assets/5f17f029-8695-49d1-a53e-629fede788a1" />


## Step 6: Security Groups
Implemented security-group chaining instead of broad CIDR rules — 
no resource other than the ALB and Bastion (locked to my IP) is 
directly exposed to the internet.

| Security Group | Inbound Rule | Source |
|---|---|---|
| ALB-SG | HTTP 80, HTTPS 443 | 0.0.0.0/0 |
| Bastion-SG | SSH 22 | My IP /32 |
| EC2-SG | HTTP 80 | ALB-SG |
| EC2-SG | SSH 22 | Bastion-SG |

<img width="960" height="395" alt="8" src="https://github.com/user-attachments/assets/43f6b021-fbbe-458c-9e2f-a4a6690528ce" />
