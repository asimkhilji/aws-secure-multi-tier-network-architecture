<img width="960" height="416" alt="15" src="https://github.com/user-attachments/assets/7727b60c-a695-4b98-89a4-20eca1e07eae" /># Secure Multi-Tier Network Architecture on AWS
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
<img width="960" height="419" alt="7" src="https://github.com/user-attachments/assets/535b1f4e-6ff1-42f1-afaf-a0968346fb2a" />
<img width="960" height="422" alt="6" src="https://github.com/user-attachments/assets/b403222b-d681-4553-b80a-98359f901f3c" />


## Step 7: Launch Template
Instead of manually creating EC2 instances, used a Launch Template so the 
Auto Scaling Group can launch identical, pre-configured instances on demand 
or on failure — without manual intervention.

- **Name:** `web-server-template`
- **AMI:** Amazon Linux 2023
- **Instance type:** t2.micro
- **Public IP:** Disabled (instances live in private subnets only)
- **Security Group:** EC2-SG
- **User Data:** Bootstraps a lightweight Python HTTP server serving a static 
  test page on port 8000 — used to validate ALB → Target Group routing. 
  (Note: in a production setup this would be replaced with a real app server 
  like Nginx/Gunicorn behind the same pattern.)

<img width="960" height="425" alt="11" src="https://github.com/user-attachments/assets/231b2abf-1cc8-4f58-952d-cf655437605d" />
<img width="960" height="427" alt="12" src="https://github.com/user-attachments/assets/8f1472a6-63ce-4c1f-8cc1-9a6af0d32eaa" />
<img width="960" height="416" alt="13" src="https://github.com/user-attachments/assets/6101dc53-3787-409d-a07f-bdebfa3be622" />
<img width="959" height="421" alt="14" src="https://github.com/user-attachments/assets/b073bae9-4db6-4006-a97b-726131bd3c40" />
<img width="960" height="416" alt="15" src="https://github.com/user-attachments/assets/67d169f2-b0fe-44dc-8cb3-dad38d3acbbb" />

<img width="960" height="416" alt="16" src="https://github.com/user-attachments/assets/937cac36-e9b6-4b7e-9a98-cbe97f32e74c" />
<img width="960" height="415" alt="17" src="https://github.com/user-attachments/assets/dfb0bb2e-3819-45d1-b1be-e62b27d304c4" />








