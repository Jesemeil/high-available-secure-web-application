## Architecture Diagram

![manara-proj-3](https://github.com/user-attachments/assets/daf82941-b4e2-4420-92de-b789f7aa3bc0)


# **AWS High Availability and Secure Web Application Architecture**



##  **1. Summary**

This architecture illustrates a **production-grade AWS deployment** of a high-availability, secure, and scalable web application using infrastructure as code (IaC) practices. The design follows the **AWS Well-Architected Framework** pillars—**Operational Excellence, Security, Reliability, Performance Efficiency**, and **Cost Optimization**—making it a model solution for real-world cloud deployments.



##  **2. Architecture Overview**

The infrastructure is hosted in a single **AWS Region** across **two Availability Zones (AZs)** to achieve high availability and fault tolerance. It integrates core AWS services including:

* **Amazon EC2** for compute
* **ALB (Application Load Balancer)** for traffic distribution
* **MySQL (RDS)** for managed relational database
* **S3, DynamoDB, and SSM Parameter Store** for Terraform backend and secret management
* **Auto Scaling Group** for elasticity
* **VPC with public/private subnet segmentation**
* **Terraform** for infrastructure automation



## **3. Infrastructure Components and Flow**

### **3.1. AWS VPC (Virtual Private Cloud)**

* A logically isolated network space.
* Contains **two Availability Zones** with **public and private subnets** for isolation and security.
* Connected to the internet via an **Internet Gateway** attached to the VPC.



###  **3.2. Public Subnets**

* Hosts **Auto Scaling EC2 instances** managed by an **Application Load Balancer (ALB)**.
* Instances are stateless and managed for horizontal scaling based on load metrics.
* The **ALB** handles incoming requests from users and routes them to healthy instances across AZs.



###  **3.3. Private Subnets**

* Dedicated to **MySQL RDS instances** to ensure sensitive data is isolated from public access.

  * **Production DB Instance** in AZ1.
  * **Standby DB Instance** in AZ2 (for failover, via Multi-AZ deployment).
* RDS access is tightly controlled via **security groups** and **IAM roles**.


###  **3.4. Application Load Balancer**

* Deployed across **both public subnets**.
* Handles:

  * **Health checks**
  * **Routing**
  * **SSL termination 
* DNS name of the ALB is used by **end users** to access the application.



###  **3.5. Auto Scaling Group**

* Ensures consistent performance by:

  * Scaling EC2 instances **up/down** based on demand (CPU usage, request count, etc.).
  * Replacing unhealthy instances automatically.
* Spans both AZs for **fault tolerance**.


##  **4. Infrastructure as Code (Terraform)**

###  **4.1. Terraform Backend**

* **S3 Bucket** stores the `terraform.tfstate` file securely and centrally.
* **DynamoDB Table** locks the state to prevent concurrent modifications (state locking).

###  **4.2. Secure Secret Management**

* **AWS SSM Parameter Store** is used to:

  * Store and retrieve sensitive values such as **RDS passwords** or **API keys**.
  * Maintain configuration consistency across environments.
* Secrets are accessed programmatically during provisioning or runtime using IAM policies.



## **5. Developer & User Interaction**

###  **Developer Workflow**

* Writes Terraform scripts locally or via CI/CD.
* On `terraform apply`, configuration is pushed to AWS using:

  * **S3 for state**
  * **DynamoDB for lock**
  * **IAM for permission control**
* Database credentials are fetched securely from SSM Parameter Store.

###  **End User Flow**

1. Sends request to **ALB DNS Name**
2. ALB routes traffic to healthy EC2 instances
3. EC2 processes the request and queries the **MySQL RDS** backend in private subnet



##  **6. Security Best Practices**

| Layer          | Control Implemented                                |
| -------------- | -------------------------------------------------- |
| Network        | Public/private subnet isolation, Internet Gateway  |
| Data at Rest   | RDS encryption, S3 server-side encryption (SSE-S3) |
| Secrets        | SSM Parameter Store with encryption (KMS)          |
| IAM            | Fine-grained roles for EC2, Terraform, RDS         |
| Access Control | Security Groups & NACLs for controlled access      |



##  **7. High Availability & Resilience**

| Service | Mechanism                                   |
| ------- | ------------------------------------------- |
| EC2     | Auto Scaling across AZs                     |
| RDS     | Multi-AZ deployment with automatic failover |
| ALB     | Cross-AZ traffic routing                    |
| Network | Redundant AZs to ensure failover            |



##  **8. Cost Optimization Strategies**

* **Auto Scaling** avoids overprovisioning.
* **RDS Reserved Instances** can be used for long-term savings.
* **Terraform** ensures reproducibility, reducing human errors and cost leaks.
* **Spot Instances** can be introduced for lower-priority tasks.



##  **9. Observability & Maintainability**

* Integrate **CloudWatch** with EC2 and RDS for monitoring CPU, memory, query performance.
* Set up **Alarms and SNS** for proactive alerting.
* Logs can be sent to **S3** or **CloudWatch Logs** for centralized visibility.



## **10. Benefits and Use Cases**

| Benefit                  | Description                                    |
| ------------------------ | ---------------------------------------------- |
| Highly Available         | Multi-AZ deployment with load balancing        |
| Secure                   | IAM, SSM, private subnet protection            |
| Scalable                 | Auto Scaling of compute tier                   |
| Maintainable & Auditable | Infrastructure as code with versioned backend  |
| Enterprise-Ready         | Follows best practices suitable for production |


