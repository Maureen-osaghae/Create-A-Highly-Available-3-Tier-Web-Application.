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

Under Application Load Balancer, choose Create.

<img width="959" height="469" alt="image" src="https://github.com/user-attachments/assets/50ff4110-1a32-4b23-8dca-cecec0efd383" />

Give it a name: Inventory-LB

<img width="959" height="467" alt="image" src="https://github.com/user-attachments/assets/b317c41c-f1b8-4170-80d1-2c3a7c216186" />

Scroll down to the Network mapping section, then for VPC, choose Lab VPC. Specify which subnets the load balancer should use. It will be a public load balancer, select both public subnets.
Under Mappings, choose the first Availability Zone, then choose the Public Subnet that displays. selected two subnets: Public Subnet 1 and Public Subnet 2. 

<img width="959" height="476" alt="image" src="https://github.com/user-attachments/assets/c282034c-231a-428c-a46b-9d103c9778b1" />


 In the Security groups section,  select the Create a new security group hyperlink. This opens a new browser tab. Configure the new security group settings:


---

- Security group name: Inventory-LB
- Description: Enable web access to load balancer
- VPC:  Then select Lab VPC.

---

<img width="959" height="445" alt="image" src="https://github.com/user-attachments/assets/364d8629-f831-4e88-8c62-f452bc00e961" />

Under Inbound rules,  choose Add rule and configure as described:

---
- Type: HTTP
- Source: Anywhere-IPv4
- Still under Inbound rules, I choose Add rule again and configure:
- Type: HTTPS
-Source: Anywhere-IPv4

---

<img width="956" height="467" alt="image" src="https://github.com/user-attachments/assets/df90efa8-7d50-4a6f-ab7d-e3842e52a4ee" />

<img width="959" height="439" alt="image" src="https://github.com/user-attachments/assets/51128b48-6131-4408-8f1c-e4864984ccdc" />

Assign the security group to the load balancer where you are still configuring the load balancer.

<img width="959" height="451" alt="image" src="https://github.com/user-attachments/assets/2afae447-a1b1-46f7-b7f9-88a5c6e31d2d" />

In the Listeners and routing section, choose Create target group.

**Analysis:** Target groups define where to send traffic that comes into the load balancer. The Application Load Balancer can send traffic to multiple target groups based upon the URL of the incoming request, such as having requests from mobile apps going to a different set of servers. Your web application will use only one target group.

---

## Configure the target group as described here:
- Target type: Instances
- Target group name: Inventory-App
- VPC: Ensure that Lab VPC is chosen.

---

Scroll down and expand 8*Advanced health check settings.** The Application Load Balancer automatically performs health checks on all instances to ensure that they are responding to requests. The default settings are recommended, but you will need to make them slightly faster for use in this lab.

---
- Healthy threshold: 2
- Interval: 10 (seconds)

---

The configurations you have chosen will result in the health check being performed every 10 seconds, and if the instance responds correctly twice in a row, it will be considered healthy.


<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/ca450d7c-984e-49b3-a4a1-2ee332b01eb7" />

<img width="959" height="446" alt="image" src="https://github.com/user-attachments/assets/66869294-56c0-458c-b997-fb251bf6cd38" />

<img width="958" height="473" alt="image" src="https://github.com/user-attachments/assets/3b54e3d9-d184-4c06-8500-11e6292ab49d" />

Choose Next. The Register targets screen appears. Targets are the individual instances that will respond to requests from the load balancer. I do not have any web application instances yet, so I can skip this step. Review the settings and choose Create target group.

<img width="959" height="453" alt="image" src="https://github.com/user-attachments/assets/6d940397-da5d-4688-9901-47bcf103ab74" />

Return to the browser tab where you already started defining the load balancer.
In the Listeners and routing section
For the Listener HTTP:80 row,  set the Default action to forward to the Inventory-App target group you just created.
Scroll to the bottom of the page, and choose Create load balancer.

<img width="959" height="464" alt="image" src="https://github.com/user-attachments/assets/50e48ccf-a7dd-4a2d-9fab-2aaba6ad284c" />









