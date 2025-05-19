# Project: Scalable & Resilient Web Application Hosting on AWS

## 1. Project Title & Overview

* **Title:** Scalable & Resilient Web Application Hosting on AWS
* **Overview:** Key project conceptualized and designed during my tenure at S.A. Rahman Consulting Ltd, demonstrating a fault-tolerant and scalable infrastructure on AWS to host a sample web application, serving as a blueprint for client engagements.

## 2. Problem Statement/Goal

* **Objective:** The primary objective was to design and implement a reusable, highly available, and scalable hosting solution on AWS for typical client web applications. This architecture served as a Proof of Concept (PoC) to demonstrate best practices and provide a foundational blueprint for future client deployments, emphasizing reliability under varying load conditions and efficient resource utilization.
* **Business Drivers & Pain Points Addressed by PoC:**
    * To create a standardized, best-practice deployment model for the consultancy, enabling faster onboarding of new client projects.
    * To showcase to potential clients AWS capabilities in achieving high uptime and performance for their web applications.
    * To provide a clear example of how to implement a secure and cost-conscious cloud infrastructure.
    * To reduce the time and effort spent on ad-hoc infrastructure setups for individual client PoCs by having a ready-to-adapt template.

## 3. My Role & Responsibilities (as the Architect/Designer of this PoC)

As the architect for this Proof of Concept, my responsibilities encompassed the end-to-end design and strategic planning of the infrastructure:

* **VPC Design & Network Architecture:**
    * I designed the Virtual Private Cloud (VPC) layout, including selecting an appropriate CIDR block (e.g., `10.0.0.0/16`) to allow for future expansion and multiple environments.
    * I defined and architected public and private subnets across at least two Availability Zones (e.g., `eu-west-2a`, `eu-west-2b`) to ensure high availability and robust network segmentation.
    * I established route tables for both public and private subnets, configuring routes to an Internet Gateway (IGW) for public resources and a NAT Gateway (one per AZ for redundancy, or a single one for cost-optimization in a PoC) for controlled outbound internet access from private subnets.

* **Load Balancing & Auto Scaling Implementation:**
    * I architected the use of an Application Load Balancer (ALB), specifying listener rules for HTTP/HTTPS, target groups pointing to the EC2 web servers, and health check parameters to efficiently distribute incoming traffic and ensure requests are only sent to healthy instances.
    * I designed the Auto Scaling Group (ASG) for the EC2 web server fleet, outlining scaling policies based on average CPU utilization (e.g., scale out if CPU > 70%, scale in if CPU < 30%) to dynamically adjust capacity. This included defining minimum, maximum, and desired instance counts for fault tolerance and cost management.

* **EC2 Instance & Web Server Configuration Strategy:**
    * I specified the selection of appropriate EC2 instance types (e.g., t3.micro or t3.small for cost-effectiveness in a PoC, with recommendations for production instance families like M5 or C5) and the use of Amazon Linux 2 AMIs.
    * I outlined the configuration for EC2 instances to host the Nginx web server software and the deployment strategy for the sample web application files (e.g., via User Data scripts or a CodeDeploy process in a more advanced setup).

* **Database Tier Implementation Strategy:**
    * I designed the Amazon RDS deployment, selecting MySQL as the database engine for the PoC.
    * A key architectural decision was to specify RDS for Multi-AZ deployment to ensure high availability and automatic failover for the database tier.

* **Static Content Delivery Strategy:**
    * I architected the use of an Amazon S3 bucket for storing and serving static assets (images, CSS, JavaScript files), recommending configurations for bucket permissions and versioning. *(Consideration was given to using CloudFront for CDN capabilities in a production scenario).*

* **Security Implementation & Best Practices Design:**
    * I designed granular Security Groups for the ALB, EC2 instances, and RDS database, defining rules based on the principle of least privilege.
    * I specified the creation and attachment of IAM roles to the EC2 instances, granting only the necessary permissions to interact with other AWS services (e.g., S3, CloudWatch) without using hardcoded credentials.
    * I ensured the architectural design placed critical resources like RDS instances and application EC2 instances in private subnets.

* **Monitoring & Operational Visibility Strategy:**
    * I outlined the configuration of Amazon CloudWatch monitoring for key performance indicators (KPIs) including EC2 CPU utilization, ALB request counts/latency, and RDS database metrics.
    * I designed the setup of initial CloudWatch Alarms for critical thresholds to demonstrate proactive alerting capabilities.

* **Documentation & Blueprinting Contribution:**
    * I was responsible for conceptualizing and documenting this architecture, its deployment steps, and configuration best practices to serve as a reusable blueprint.

## 4. Architecture

### Architecture Diagram

![Scalable Web Application Architecture on AWS](path/to/your/webapp_architecture.png)

**Key elements represented in the diagram:**

* **AWS Cloud Region:** (e.g., `eu-west-2` London)
* **Availability Zones (AZs):** Two AZs (e.g., `eu-west-2a`, `eu-west-2b`)
* **Virtual Private Cloud (VPC):** (e.g., `10.0.0.0/16`)
    * **Public Subnets:** One in each AZ (e.g., `10.0.1.0/24`, `10.0.2.0/24`)
    * **Private Subnets:** One in each AZ (e.g., `10.0.101.0/24`, `10.0.102.0/24`)
* **Internet Gateway (IGW):** Attached to the VPC.
* **NAT Gateway(s):** One in each public subnet (for HA of outbound traffic).
* **Route Tables:**
    * Public Route Table: Default route to IGW, associated with public subnets.
    * Private Route Tables (one per AZ): Default route to respective NAT Gateway, associated with private subnets.
* **Application Load Balancer (ALB):** Spanning public subnets in both AZs.
* **Auto Scaling Group (ASG):** Managing EC2 instances across private subnets in both AZs.
* **EC2 Instances (Web Servers):** Running Nginx, within the ASG, in private subnets.
* **Amazon RDS (MySQL - Multi-AZ):** Primary in one private subnet (AZ1), Standby in another private subnet (AZ2).
* **Amazon S3 Bucket:** For static assets.
* **Security Groups:** Applied to ALB, EC2 instances, and RDS.
* **Traffic Flow Arrows:** User -> ALB -> EC2 -> RDS. Users/EC2s -> S3 for static assets.

### Explanation of Components

* **Virtual Private Cloud (VPC):** Provided a logically isolated network environment within AWS for all application resources. Configured with a `/16` CIDR block for ample IP address space, and segmented into public and private subnets across two Availability Zones to enhance security and availability.
* **Application Load Balancer (ALB):** Deployed in the public subnets across both Availability Zones, the ALB served as the single point of entry for all incoming web traffic (HTTP/HTTPS). It distributed this traffic to healthy EC2 instances in the backend Auto Scaling Group based on configured listener rules and health checks.
* **Auto Scaling Group (ASG):** Managed the EC2 web server fleet within the private subnets. It was configured to automatically scale the number of instances based on CPU utilization, ensuring the application could handle fluctuating traffic loads efficiently and maintain performance. It also ensured instance distribution across AZs for fault tolerance.
* **EC2 Instances:** Amazon Linux 2 instances running the Nginx web server hosted the sample web application. These were launched in private subnets and configured to only receive traffic from the ALB. User Data scripts were conceptualized for bootstrapping (installing Nginx, deploying app code).
* **Amazon RDS (Multi-AZ MySQL):** A managed relational database service configured with MySQL in a Multi-AZ deployment. This provided high availability and automatic failover to a synchronous standby replica in a different AZ in case of primary database failure. The instance was located in private subnets.
* **Amazon S3:** Utilized for storing and serving static website content (images, CSS, JavaScript). This approach offloaded the EC2 instances, improved website load times, and benefited from S3's high durability and scalability.
* **Security Groups:** Acted as stateful firewalls at the instance/resource level. Rules were defined to:
    * Allow inbound HTTP (port 80) and HTTPS (port 443) traffic from the internet (`0.0.0.0/0`) to the ALB.
    * Allow inbound traffic from the ALB's security group to the EC2 instances on the application port (e.g., port 80).
    * Allow inbound traffic from the EC2 instances' security group to the RDS instance on the MySQL port (3306).
    * Restrict all other inbound traffic by default. Outbound rules were configured to allow necessary communication (e.g., to NAT Gateway for updates).
* **IAM Roles:** Assigned to EC2 instances to provide them with secure, temporary credentials to access other AWS services (e.g., read objects from a specific S3 bucket, send metrics/logs to CloudWatch) without needing to embed access keys.
* **Amazon CloudWatch:** Leveraged for essential monitoring. This included tracking key metrics for EC2 (CPU, Network), ALB (Request Count, Latency, HTTP Errors), and RDS (CPU, Connections, Storage). CloudWatch Alarms were designed to trigger notifications via SNS if predefined thresholds were breached.
* **Internet Gateway (IGW):** Attached to the VPC to allow communication between resources in public subnets and the internet.
* **NAT Gateway(s):** Deployed in public subnets (one per AZ for high availability) to enable instances in private subnets to initiate outbound connections to the internet (e.g., for software updates) while preventing unsolicited inbound connections.

## 5. Security Considerations

Security was a core tenet of this architectural design, implemented through a defense-in-depth strategy:

* **Network Segmentation & Isolation:**
    * The VPC provided the primary layer of network isolation.
    * Critical application components (EC2 web servers) and the database (RDS) were strategically placed in **private subnets**, shielded from direct public internet access. Only resources that explicitly needed to be internet-facing (ALB, NAT Gateways) were placed in public subnets.

* **Principle of Least Privilege:**
    * **Security Groups:** Ingress and egress rules were meticulously defined to allow *only* necessary traffic on specific ports and protocols between designated sources and destinations.
    * **IAM Roles:** EC2 instances were granted permissions via IAM roles with policies tailored to their specific needs.

* **Data Protection:**
    * **RDS Security:** The RDS instance was isolated in private subnets. *(Conceptual addition: Consideration was given to enabling encryption at rest for RDS using AWS KMS, and configuring SSL/TLS for data in transit).*
    * **S3 Security:** The S3 bucket for static assets was configured to block public write access. *(Conceptual addition: Server-Side Encryption (SSE-S3) was recommended. Access logging was also advised).*

* **Infrastructure Hardening & Management (Conceptual for PoC):**
    * **Patch Management:** A strategy for regular OS patching for EC2 instances (e.g., using AWS Systems Manager Patch Manager) was outlined.
    * **Secure Access:** Use of a Bastion Host for administrative access to EC2 instances in private subnets was recommended.

* **Monitoring & Logging for Security Insights:**
    * CloudWatch provided operational visibility, foundational for security monitoring.
    * AWS CloudTrail (enabled by default) would provide an audit log of API calls.
    * VPC Flow Logs were considered for capturing IP traffic information for network security analysis in production.

## 6. Key Challenges & Solutions (Conceptual for this PoC Design)

* **Challenge 1: Ensuring High Availability Across Components.**
    * **Solution:** Architected across two Availability Zones. This included deploying the ALB across public subnets in both AZs, configuring the Auto Scaling Group to maintain instances in private subnets of both AZs, and utilizing RDS Multi-AZ deployment.

* **Challenge 2: Balancing Security with Necessary Outbound Connectivity for Private Instances.**
    * **Solution:** Implemented NAT Gateways in public subnets, allowing EC2 instances in private subnets to initiate outbound connections while preventing direct inbound connections. Route tables were carefully configured.

* **Challenge 3: Designing for Scalability While Managing Costs for a PoC.**
    * **Solution:** Leveraged Auto Scaling Groups for EC2 instances with CPU-based scaling policies. Smaller instance types were selected for the PoC, with recommendations for larger types for production. S3 was used for static assets to offload EC2 instances.

## 7. Outcome

* Successfully designed a robust, scalable, and resilient cloud architecture on AWS, providing a clear and effective blueprint for deploying typical client web applications.
* **Quantifiable/Qualitative Outcomes (Representational):**
    * This standardized architectural blueprint was projected to reduce initial infrastructure setup and configuration time for similar client web application PoCs by approximately 30-40%.
    * The design clearly demonstrated how to achieve high availability (conceptually targeting >99.9% uptime) through Multi-AZ deployments.
    * Provided a practical demonstration of implementing core AWS security best practices.
    * Established a foundational template adaptable for more complex client requirements.

## 8. Technologies Used (Conceptualized for this Architecture)

* **AWS Services:** Amazon VPC, EC2 (Amazon Linux 2), Auto Scaling Groups, Application Load Balancer (ALB), S3, RDS (MySQL), IAM (Roles & Policies), CloudWatch (Metrics & Alarms), Internet Gateway (IGW), NAT Gateway.
* **Web Server Software:** Nginx.
* **Operating System:** Amazon Linux 2.
* **Design & Documentation Tools:** `diagrams.net` (or similar for architecture diagram), Markdown.
* **Conceptual IaC Tool (for future implementation):** Terraform.

