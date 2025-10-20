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

# Step 2: Create an Auto Scaling group

Amazon EC2 Auto Scaling is a service designed to launch or terminate Amazon EC2 instances automatically based on user-defined policies, schedules, and health checks. It also automatically distributes instances across multiple Availability Zones to make applications highly available.
Create an Auto Scaling group that deploys EC2 instances across private subnets, which is a security best practice for application deployment. 
Instances in a private subnet cannot be accessed from the internet. Instead, users send requests to the load balancer, which forwards the requests to EC2 instances in the private subnets.

<img width="401" height="281" alt="image" src="https://github.com/user-attachments/assets/1979767c-1b66-442b-b29c-e9e763788bfe" />

# Create an AMI for Auto Scaling
Create an Amazon Machine Image (AMI) from the existing **Web Server 1.** This will save the contents of the root volume of the web server so that new instances can be launched with an identically configure guest operating system.
Head to the EC2 Console;
Choose Instances. Select the Web Server 1.

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/8a89f4f8-0c8f-4b2c-b71b-762187a28925" />

---
## Actions menu, choose Image and templates > Create image, then configure:

- Image name: Web Server AMI
- Image description: Lab AMI for Web Server

---

<img width="959" height="463" alt="image" src="https://github.com/user-attachments/assets/627b43e7-67a6-4f45-a10e-f93d34d71df6" />

<img width="955" height="452" alt="image" src="https://github.com/user-attachments/assets/e227cdac-6037-4728-a991-3a046d7d9a80" />

<img width="959" height="292" alt="image" src="https://github.com/user-attachments/assets/6b147426-a436-4886-8273-dfa7610a13a0" />

## Create a Launch Template and an Auto Scaling Group
First create a launch template. **A launch template is a template that an Auto Scaling group uses to launch EC2 instances**. When you create a launch template, you specify information for the instances such as the AMI, the instance type, a key pair, and security group. 

---
## Configure the launch template settings and create it:
- Launch template name: Inventory-LT
- Under Auto Scaling guidance,  select Provide guidance to help me set up a template that I can use with EC2 Auto Scaling
- In the Application and OS Images (Amazon Machine Image) area, choose My AMIs.
- Amazon Machine Image (AMI): choose Web Server AMI
- Instance type: choose t2.micro
- Key pair name: choose vockey
- Firewall (security groups): choose Select existing security group
- Security groups: choose Inventory-App
- Scroll down to the Advanced details area and expand it.
- IAM instance profile: choose Inventory-App-Role
- Scroll down to the Detailed CloudWatch monitoring setting. Select Enable 

This will allow Auto Scaling to react quickly to changing utilization.
---

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/e0453b1e-e385-42f0-828e-6b66bd19c87d" />

<img width="839" height="359" alt="image" src="https://github.com/user-attachments/assets/52a5e1f1-7b22-4084-b876-7dc607c121db" />

<img width="839" height="359" alt="image" src="https://github.com/user-attachments/assets/0a838e15-e19b-4623-b345-f5daf93cc109" />

Under User data,  paste in the script below: 

<img width="959" height="451" alt="image" src="https://github.com/user-attachments/assets/a6016b81-3a1d-41f4-baa8-80dfa5f0323c" />

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/cb104abb-c1f4-41e5-b9f5-74babff5413c" />

Create launch template
Next, create an Auto Scaling group that uses this launch template. **The Auto Scaling group defines where to launch the EC2 instances.**

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/2118e0bc-c663-4032-bab6-95d309c57ad4" />

Actions menu, choose **Create Auto Scaling group**
Configure the details in Step 1 (Choose launch template or configuration):

---
- Auto Scaling group name: Inventory-ASG (ASG stands for Auto Scaling group)
- Launch template: Confirm that the Inventory-LT template you just created is selected.

---

<img width="959" height="447" alt="image" src="https://github.com/user-attachments/assets/d557a1c1-9afb-4afb-8735-e237cc0f8b60" />

Choose next

---

## Configure the details in Step 2 (Choose instance launch options):
- VPC: Lab VPC
- Availability Zones and subnets: Choose Private Subnet 1 and then choose Private Subnet 2. This will launch EC2 instances in private subnets across both Availability Zones.

---

<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/3710eec2-5651-45fd-9121-22b0c6d0f9fa" />

Choose next

---

## Configure the details in Step 3 (Configure advanced options):
- In the Load balancing panel:
- Choose Attach to an existing load balancer
- Existing load balancer target groups: select Inventory-App.
- In the Health checks panel:
- Health check grace period: 90 seconds
- In the Additional settings panel:
- Select Enable group metrics collection within CloudWatch

This will capture metrics at 1-minute intervals, which allows Auto Scaling to react quickly to changing usage patterns.

---

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/b592669d-d21f-49da-abca-719845c11ec1" />

<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/5ef2db4f-2fa6-484c-9a56-51010bb8c96b" />

Choose next

---

Configure the details in Step 4 (Configure group size and scaling policies - optional):
- Under Group size, configure: 
- Desired capacity: 2
- Minimum capacity: 2
- Maximum capacity: 2
- This will allow Auto Scaling to automatically add/remove instances, always keeping between 2 and instances running.
- Under Scaling policies, choose None 
      
---

# Step 2: Create an Auto Scaling group

Amazon EC2 Auto Scaling is a service designed to launch or terminate Amazon EC2 instances automatically based on user-defined policies, schedules, and health checks. It also automatically distributes instances across multiple Availability Zones to make applications highly available.
Create an Auto Scaling group that deploys EC2 instances across private subnets, which is a security best practice for application deployment. 
Instances in a private subnet cannot be accessed from the internet. Instead, users send requests to the load balancer, which forwards the requests to EC2 instances in the private subnets.


# Create an AMI for Auto Scaling
Create an Amazon Machine Image (AMI) from the existing **Web Server 1.** This will save the contents of the root volume of the web server so that new instances can be launched with an identically configure guest operating system.
Head to the EC2 Console;
Choose Instances. Select the Web Server 1.

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/ec12e002-fb54-484a-b6e4-f01ccd865b26" />

---

## Actions menu, choose Image and templates > Create image, then configure:
- Image name: Web Server AMI
- Image description: Lab AMI for Web Server

---

<img width="959" height="463" alt="image" src="https://github.com/user-attachments/assets/64c25241-8c6a-4802-a25e-bfdd5249e3f4" />

<img width="955" height="452" alt="image" src="https://github.com/user-attachments/assets/5e865111-8435-42a0-993e-8238897ace99" />

<img width="959" height="292" alt="image" src="https://github.com/user-attachments/assets/34a581f6-49b6-400c-9511-a9b9ca4fcbdc" />

## Create a Launch Template and an Auto Scaling Group
First create a launch template. **A launch template is a template that an Auto Scaling group uses to launch EC2 instances**. When you create a launch template, you specify information for the instances such as the AMI, the instance type, a key pair, and security group. 

---

## Configure the launch template settings and create it:
- Launch template name: Inventory-LT
- Under Auto Scaling guidance,  select Provide guidance to help me set up a template that I can use with EC2 Auto Scaling
- In the Application and OS Images (Amazon Machine Image) area, choose My AMIs.
- Amazon Machine Image (AMI): choose Web Server AMI
- Instance type: choose t2.micro
- Key pair name: choose vockey
- Firewall (security groups): choose Select existing security group
- Security groups: choose Inventory-App
- Scroll down to the Advanced details area and expand it.
- IAM instance profile: choose Inventory-App-Role
- Scroll down to the Detailed CloudWatch monitoring setting. Select Enable 
- This will allow Auto Scaling to react quickly to changing utilization.

---

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/f3396831-3ead-47fb-b916-c8de3e3e3e3a" />

<img width="839" height="359" alt="image" src="https://github.com/user-attachments/assets/177ba47c-1b22-4910-b78f-9164866b5140" />

<img width="839" height="359" alt="image" src="https://github.com/user-attachments/assets/f51bb6d6-b3b7-40b2-9850-ec3bff358091" />

Under User data,  paste in the script below: 

<img width="959" height="451" alt="image" src="https://github.com/user-attachments/assets/6979c736-01ae-4621-8f2d-03212228bad9" />

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/2c8462e9-b249-41d0-8b83-495f8f700ec7" />

# Create launch template
Next, create an Auto Scaling group that uses this launch template. **The Auto Scaling group defines where to launch the EC2 instances.**

 Actions menu, choose **Create Auto Scaling group**
---
## Configure the details in Step 1 (Choose launch template or configuration):
- Auto Scaling group name: Inventory-ASG (ASG stands for Auto Scaling group)
- Launch template: Confirm that the Inventory-LT template you just created is selected.

---

<img width="959" height="447" alt="image" src="https://github.com/user-attachments/assets/d819e0a6-7d06-4032-8f0d-46cf232c4944" />

Choose next

---
## Configure the details in Step 2 (Choose instance launch options):
- VPC: Lab VPC
- Availability Zones and subnets: Choose Private Subnet 1 and then choose Private Subnet 2. This will launch EC2 instances in private subnets across both Availability Zones.

---

<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/42be71f2-28dd-4e29-a865-8a21cad50ce9" />

---

## Configure the details in Step 3 (Configure advanced options):
- In the Load balancing panel:
        - Choose Attach to an existing load balancer
        - Existing load balancer target groups: select Inventory-App.
- In the Health checks panel:
        - Health check grace period: 90 seconds
- In the Additional settings panel:
        - Select Enable group metrics collection within CloudWatch

This will capture metrics at 1-minute intervals, which allows Auto Scaling to react quickly to changing usage patterns.


---

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/01167f13-976e-4779-ad67-cb8bcc87f46c" />

<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/2b0d4572-0656-47f2-abbc-16f6dd85d569" />

Choose next

---

## Configure the details in Step 4 (Configure group size and scaling policies - optional):
- Under Group size, configure: 
- Desired capacity: 2
- Minimum capacity: 2
- Maximum capacity: 2
- This will allow Auto Scaling to automatically add/remove instances, always keeping between 2 and instances running.
- Under Scaling policies, hoose None 
      
---
For this lab, I will maintain two instances at all times to ensure high availability. If the application is expected to receive varying loads of traffic, I can also create scaling policies that define when to launch or terminate instances. However, I do not need to create scaling policies for the Inventory application in this lab.

<img width="959" height="434" alt="image" src="https://github.com/user-attachments/assets/44c8b31f-dbab-465f-9e29-254b39a35f84" />

Choose next, I do not need to configure any of these settings. 

<img width="955" height="374" alt="image" src="https://github.com/user-attachments/assets/c51be85b-aa0e-4559-9f21-daff23c44354" />

Choose next again
---

## Configure the details in Step 6 (Add tags - optional):
- Choose Add tag and Configure the following:
- Key: Name
- Value: Inventory-App

---

These settings will tag the Auto Scaling group with a Name, which will also appear on the EC2 instances that are launched by the Auto Scaling group. I can use tags to identify which Amazon EC2 instances are associated with which application. I could also add tags such as Cost Center to assign application costs in the billing files.

<img width="959" height="425" alt="image" src="https://github.com/user-attachments/assets/44ea9510-04bb-46bf-9ca7-48820b42f42f" />

Click next.
Review the details of my Auto Scaling group
      
Create Auto Scaling group
      
My Auto Scaling group initially show an instance count of zero, but new instances will be launched to reach the Desired count of 2 instances.
The Inventory-ASG  appear in the console. 

<img width="959" height="287" alt="image" src="https://github.com/user-attachments/assets/efdbefe8-edc2-4063-ace2-f690b679976c" />























   












