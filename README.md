# Secure Web Application Deployment on AWS using Ansible and Load Balancing

A secure, highly available web application deployed on AWS using a bastion host, private web servers, Ansible automation, and an Application Load Balancer.

---

## Architecture
![Architecture Diagram](/aws-diagram.drawio%20(1).png "Architecture Diagram")

---

## Traffic Flow

**User Traffic:** Requests from the browser hit the Application Load Balancer (ALB), which routes them to either web server. The web server processes and responds back through the ALB.

**Admin Access:** Engineers SSH into the Bastion Host, then hop to private web servers. SSH ProxyJump makes this seamless from the local machine.

**Outbound Updates:** Private servers route internet traffic through the NAT Gateway for package downloads. All inbound connections from the internet are blocked.

---

## Project Structure

```
web-app-project/
├── README.md
├── inventory.ini
├── deploy-webservers.yml
└── roles/
    └── webserver/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   └── index.html.j2
        └── handlers/
            └── main.yml
```

---

## Infrastructure

| Component | Type | Subnet | Public IP | Purpose |
|-----------|------|--------|-----------|---------|
| Bastion Host | EC2 t2.micro | Public | Yes | Secure SSH gateway |
| NAT Gateway | AWS Managed | Public | Elastic IP | Outbound internet for private servers |
| Web Server 1 | EC2 t2.micro | Private | No | Hosts NGINX web application |
| Web Server 2 | EC2 t2.micro | Private | No | Hosts NGINX web application |
| App Load Balancer | ALB | Public (Multi-AZ) | AWS DNS | Traffic distribution |

---

## Security Groups

| Group | Inbound Rules | Purpose |
|-------|--------------|---------|
| `bastion-sg` | SSH (22) from your IP | Protects bastion host |
| `web-servers-sg` | SSH (22) from bastion-sg, HTTP (80) from alb-sg | Restricts web server access |
| `alb-sg` | HTTP (80) from 0.0.0.0/0 | Allows public web traffic |

---

## Deployment

**Prerequisites:**
- AWS account
- SSH key pair (`web-app-key.pem`)
- Ansible installed locally
- SSH config with ProxyJump configured

**Deploy web servers:**

```bash
# Test connectivity
ansible all -i inventory.ini -m ping

# Deploy NGINX and HTML pages
ansible-playbook deploy-webservers.yml -i inventory.ini
```

---

## Key Features

- **Secure Access** — Web servers have no public IPs; all SSH access goes through the bastion host
- **Automated Deployment** — Ansible installs and configures NGINX on both servers with a single command
- **High Availability** — ALB distributes traffic across two servers with automatic health checks
- **Dynamic Pages** — Each server displays its own hostname and private IP via Jinja2 templating
- **Least Privilege** — Security groups allow only the minimum required traffic between components

---

## Full Documentation

A detailed step-by-step article covering this entire project, including architecture decisions, challenges faced, and key concepts is available here:

**[Read the full article →]** *(Add your Medium/Hashnode link here)*

---

## Cleanup

To avoid ongoing AWS charges, delete resources in this order:

1. Delete the **Application Load Balancer**
2. Delete the **Target Group**
3. Delete the **NAT Gateway** *(stops hourly billing)*
4. Release the **Elastic IP**
5. Terminate all **EC2 instances**
6. Delete the **VPC** *(automatically removes subnets, route tables, and internet gateway)*

---

## Author

**[Your Name]**
Junior Cloud Engineer
[LinkedIn] · [Hashnode/Medium]