# Create A Highly Available 3 Tier Web Application.

**Cloud Platform:** AWS
**Services used:** Amazon EC2, Load Balancer, Autoscaling Group, RDS, VPC

# LAB SCENARIO 

Critical business systems should be deployed as highly available applications. That is, applications remain operational even when some components fail. 
**To achieve high availability in Amazon Web Services (AWS), AWS recommend that you run services across multiple Availability Zones.**
Many AWS services are inherently highly available, such as elastic load balancers. Many AWS services can also be configured for high availability, such as deploying Amazon Elastic Compute Cloud (Amazon EC2) instances in multiple Availability Zones, RDS in multi-AZ. In this lab, I started with an application that runs on a single EC2 instance. I then make the application highly available.

---

## What I build:
  -  Create an Application Load Balancer
  -  Create an Auto Scaling group
  -  Test the application for high availability

---

At the end of this lab, my architecture will look like the following example: 

<img width="404" height="265" alt="image" src="https://github.com/user-attachments/assets/2d6e72be-40d1-4617-a2ad-115ee3f6afef" />

---
## This lab begins with an environment that was already deployed via AWS CloudFormation. It includes:

- A VPC
- Public and private subnets in two Availability Zones
- An internet gateway (not shown) that is associated with the public subnets
- A Network Address Translation (NAT) gateway in one of the public subnets
- An Amazon Relational Database Service (Amazon RDS) instance in one of the private subnets

---

<img width="440" height="275" alt="image" src="https://github.com/user-attachments/assets/389bcb63-d4aa-4929-b8a4-72048503f6d9" />

## Step 1: Create an Application Load Balancer

To build a highly available application, it is a best practice to launch resources in **multiple Availability Zones**.  Availability Zones are physically separate data centers (or groups of data centers) in the same Region. If you run an applications across multiple Availability Zones, you can provide greater availability if a data center experiences a failure.

Because the application runs on multiple application servers, you will need a way to distribute traffic amongst those **servers**. You can accomplish this goal by using a **load balancer**. This load balancer will also perform health checks on instances and only send requests to healthy instances.

<img width="415" height="284" alt="image" src="https://github.com/user-attachments/assets/0c073e98-4d1b-4c81-afc9-98d7c4897e6f" />

In the EC2 console, choose Load Balancers > create load balancer

<img width="959" height="470" alt="image" src="https://github.com/user-attachments/assets/eb433cb4-bb91-4644-bfff-25c5827592cc" />

<img width="959" height="447" alt="image" src="https://github.com/user-attachments/assets/14a1877e-3366-435d-80da-709bad1eee1e" />




