# AWS DevOps vProfile Deployment Project (SaaS + PaaS)

## Project Description

This project demonstrates the SaaS/PaaS-based deployment of the **vProfile** Java web application on AWS using a full suite of AWS services. It showcases real-world microservices deployment using **AWS Elastic Beanstalk, Amazon RDS, ElastiCache, Amazon MQ, and CloudFront**. Inspired by hands-on Udemy DevOps projects, the deployment emphasizes **scalable, secure, and automated cloud-native practices**.

All screenshots of the deployment steps are available in the **`screenshots`** folder.


---

## Application Overview

**vProfile** is a microservices-based Java web application built with the following stack:

- **Frontend:** Apache Tomcat via AWS Elastic Beanstalk  
- **Backend Services:**  
  - **Amazon RDS (MySQL)** – Database  
  - **Amazon MQ (ActiveMQ)** – Message Brokering  
  - **Amazon ElastiCache (Memcached)** – In-memory Caching  
- **Storage & Delivery:** Amazon S3 + Amazon CloudFront  
- **Domain Management:** AWS Route 53 + GoDaddy  

---

## Repository Contents

| File | Description |
|------|-------------|
| `pom.xml` | Maven build configuration |
| `application.properties` | Configuration file for DB, cache, and MQ |
| `dbsetup/` | SQL scripts for database initialization |
| `buildspec.yml` / `Jenkinsfile` | CI/CD automation files 

---

## Deployment Flow

### Step 1: Key Pair & IAM Setup
- Created AWS Key Pair for EC2 SSH access  
- Created IAM roles for Elastic Beanstalk with permissions for:
  - S3  
  - RDS  
  - CloudWatch  

### Step 2: Security Group Configuration
- Created security groups for:
  - RDS, ElastiCache, Amazon MQ (internal access only)  
  - Beanstalk (frontend)  
- Allowed:
  - Backend security groups to accept traffic **only from Beanstalk SG**  
  - Beanstalk security group to access internal ports: `3306` (RDS), `5671` (MQ), `11211` (ElastiCache)  

### Step 3: Backend Services Creation
- **Amazon RDS (MySQL):**  
  - Multi-AZ deployment  
  - Custom parameter and subnet groups  
- **Amazon ElastiCache:**  
  - Memcached cluster  
- **Amazon MQ (ActiveMQ):**  
  - Enabled basic authentication  
  - Configured internal security group access  

### Step 4: Elastic Beanstalk Setup
- Created environment using the Tomcat platform  
- Uploaded `.war` file (`mvn clean install`)  
- Configured environment variables:  
  - RDS, MQ, ElastiCache settings  
- Attached IAM Role for S3 access  
- Added port 443 listener to ELB  

### Step 5: Backend Initialization
- Launched temporary EC2 instance  
- Connected via SSH  
- Used MySQL client to:  
  - Connect to RDS  
  - Run `dbsetup/` scripts  

### Step 6: Build Artifact
- Updated `application.properties` with:  
  - RDS hostname  
  - MQ broker endpoint  
  - ElastiCache endpoint  
- Built final `.war` file using Maven  

### Step 7: SSL & CDN Configuration
- Created SSL certificate via AWS ACM for domain (e.g., `app.example.com`)  
- Set up CloudFront:  
  - Origin: Beanstalk ELB URL  
  - Attached SSL certificate  
  - Enabled compression & HTTPS redirection  

### Step 8: Domain Management
- Created Hosted Zone in Route 53  
- Updated GoDaddy DNS records:  
  - Pointed to Route 53 name servers  
  - Created A Record → CloudFront  

### Step 9: Testing & Verification
- Accessed application over **HTTPS** via custom domain  
- Verified:
  - Backend service integration  
  - SSL with no mixed content warnings  
  - CDN caching  
  - Beanstalk auto-scaling (optional)  

