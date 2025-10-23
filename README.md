# Highly Available 3-Tier Web Application on AWS

This project demonstrates the design and deployment of a **Highly Available (HA) 3-Tier Web Application** on **Amazon Web Services (AWS)**.  
The architecture was built to ensure **fault tolerance, scalability, and high availability** across multiple Availability Zones using AWS native services.

---

##  Architecture Overview

The solution follows AWS best practices for high availability and security, consisting of:

- **VPC** with public and private subnets across two Availability Zones  
- **Application Load Balancer (ALB)** for traffic distribution and health checks  
- **Auto Scaling Group (ASG)** managing EC2 instances across multiple AZs  
- **Amazon RDS (MySQL)** with Multi-AZ configuration for high availability  
- **NAT Gateways** in both public subnets for internet access from private instances  
- **Security Groups** enforcing strict three-tier isolation between Web, App, and DB layers  

---

---

## 🧩 Services Used

- **Amazon EC2** – Web and application servers  
- **Elastic Load Balancer (ALB)** – Traffic routing and health checks  
- **Auto Scaling Group (ASG)** – Automated scaling and instance recovery  
- **Amazon RDS** – Managed database (Multi-AZ setup)  
- **Amazon VPC** – Custom networking with public/private subnets  
- **NAT Gateway** – Outbound internet access for private subnets  
- **Amazon CloudWatch** – Monitoring and alerting  
- **IAM** – Role-based access management and least privilege  

---


# LAB SCENARIO 

Critical business systems should be deployed as highly available applications. That is, applications remain operational even when some components fail. 
**To achieve high availability in Amazon Web Services (AWS), AWS recommend that you run services across multiple Availability Zones.**
Many AWS services are inherently highly available, such as elastic load balancers. Many AWS services can also be configured for high availability, such as deploying Amazon Elastic Compute Cloud (Amazon EC2) instances in multiple Availability Zones, RDS in multi-AZ. In this lab, I started with an application that runs on a single EC2 instance. I then make the application highly available.

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

# Step 4: Updating security groups
The application deployed is a three-tier architecture. I will now configure the security groups to enforce these tiers:

<img width="401" height="100" alt="image" src="https://github.com/user-attachments/assets/7a7c79e2-52e7-4e7a-8699-fdbd21e81077" />

# Load balancer security group
I already configured the load balancer security group when I created the load balancer. It accepts all incoming HTTP and HTTPS traffic.
The load balancer has been configured to forward incoming requests to a target group. When Auto Scaling launches new instances, it will automatically add those instances to the target group.
# Application security group
The application security group was provided as part of the lab setup. I will now configure it to only accept incoming traffic from the load balancer.
 In the left navigation pane, choose Security Groups> Inventory-App.
       Choose the Inbound rules tab.
The security group is currently empty. I will now add a rule to accept incoming HTTP traffic from the load balancer. I do not need to configure HTTPS traffic because the load balancer was configured to forward HTTPS requests via HTTP. This practice offloads security to the load balancer, reducing the amount of work that is required by the individual application servers.

Choose Edit inbound rules.

<img width="959" height="440" alt="image" src="https://github.com/user-attachments/assets/4735feb0-6245-490c-8179-898da02fea12" />

On the Edit inbound rules page, choose Add rule and configure these settings:
      
---

 ## Type: HTTP
- Source:
- Choose the search box to the right of Custom
- Delete the current contents
- Enter sg
- From the list that appears, select Inventory-LB
- Description: Traffic from load balancer
- Choose Save rules

---

<img width="959" height="377" alt="image" src="https://github.com/user-attachments/assets/f28cf3b3-88fe-4a89-ab09-3d41566752cf" />

The application servers can now receive traffic from the load balancer. This includes health checks that the load balancer performs automatically. 

# Database security group
Configure the database security group to only accept incoming traffic from the application servers.
In the **Security groups** list, choose Inventory-DB 
The existing rule permits traffic on port 3306 (used by MySQL) from any IP address within the VPC. This is a good rule, but security can be restricted further.

<img width="959" height="311" alt="image" src="https://github.com/user-attachments/assets/5000567a-807f-488d-b860-3f55c4d2c027" />

---

## In the Inbound rules tab,  Edit inbound rules and configure these settings:
- I Delete the existing rule.
- I Choose Add rule.
- For Type, choose MYSQL/Aurora
- Choose the search box to the right of Custom
- Enter sg
- From the list that appears, I select Inventory-App
- Description: Traffic from application servers
- Choose Save rules

---

I have now configured three-tier security. Each element in the tier only accepts traffic from the tier above.
In addition, the use of private subnets means that you have two security barriers between the internet and your application resources. This architecture follows the best practice of applying multiple layers of security.

# Task 5: Testing the application

My application is now ready for testing.
In this task, confirm that the web application is running. Also test that it is highly available. In the left navigation pane, choose Target Groups>Select Inventory-App.
 
In the lower half of the page, choose the Targets tab. This tab should show two registered targets. The Health status column shows the results of the load balancer health check that is performed against the instances.

<img width="959" height="437" alt="image" src="https://github.com/user-attachments/assets/3876d2b2-984b-48c1-8dff-b07cf63c33f7" />

Test the application by connecting to the load balancer, which will then send request to one of the EC2 instances. Retrieve the Domain Name System (DNS) name of the load balancer.

<img width="959" height="432" alt="image" src="https://github.com/user-attachments/assets/e1efae0f-ca2c-442b-b351-1e2e938f671a" />

Open a new web browser tab, paste the DNS name from my clipboard and press ENTER. The load balancer forwarded my request to one of the EC2 instances. The instance ID and Availability Zone are shown at the bottom of the webpage.

<img width="824" height="286" alt="image" src="https://github.com/user-attachments/assets/846c6982-8d70-4ed9-9f4d-ed5280319052" />

When I reload the page in my web browser. I notice that the instance ID and Availability Zone sometimes toggles between the two instances.

<img width="789" height="273" alt="image" src="https://github.com/user-attachments/assets/01622bb0-4845-47f7-ba33-d041beb4c67f" />

When this web application displays, the flow of data over the network is:

<img width="452" height="120" alt="image" src="https://github.com/user-attachments/assets/55097a33-f2fb-4f98-ba51-d9e94ca0b058" />

## I sent the request to the load balancer, which resides in the public subnets that are connected to the internet.
- The load balancer chose one of the EC2 instances that reside in the private subnets and forwarded the request to it.
- The EC2 instance then returned the webpage content to the load balancer, which returned it to my web browser.

  # Step 6: Testing high availability
The application was configured to be highly available. I can prove the application's high availability by terminating one of the EC2 instances. 
       In the left navigation pane, choose Instances.
       Select one of the Inventory-App instances (it does not matter which one you select).
 Choose Instance State > Terminate instance. Choose Terminate.

 <img width="959" height="406" alt="image" src="https://github.com/user-attachments/assets/4511aa6f-8f88-43cf-ac4e-30731a9c2d2e" />

In a short time, the load balancer health checks will notice that the instance is not responding. The load balancer will automatically route all requests to the remaining instance.
 Return to the web application tab in my web browser and reload the page several times.
       Notice that the Availability Zone that is shown at the bottom of the page stays the same. Though an instance failed, my application remains available.
       After a few minutes, Amazon EC2 Auto Scaling will also notice the instance failure. It was configured to keep two instances running, so Amazon EC2 Auto Scaling automatically launch a replacement instance.
The load balancer will resume sending traffic between the two Availability Zones. 
This task demonstrates that my application is now highly available. 

# Step7: Making the database highly available

The application architecture is now highly available. However, the Amazon RDS database operates from only one database instance.
In this optional task, I will make the database highly available by configuring it to run across multiple Availability Zones (that is, in a Multi-AZ deployment).

<img width="443" height="259" alt="image" src="https://github.com/user-attachments/assets/ba7f5d98-a714-4ef0-ac01-898dfe19ae25" />
On the Services menu, choose RDS > choose Databases.
Choose the link for the name of the inventory-db instance>Modify

<img width="954" height="433" alt="image" src="https://github.com/user-attachments/assets/9b865d7c-b07f-4df4-b4b6-9b90a88b3c85" />

Scroll down to the Availability & durability section. For Multi-AZ deployment, select Create a standby instance.
**Analysis:** You only need to reconfigure this one setting to convert the database to run across multiple data centers (Availability Zones).
This option does not mean that the database is distributed across multiple instances. Instead, one instance is the primary instance, which handles all requests. Another instance will be launched as the standby instance, which takes over if the primary instance fails. My application continues to use the same DNS name for the database. However, the connections will automatically redirect to the currently active database server.

I can scale an EC2 instance by changing attributes, and I can also scale an RDS database this way. I will now scale up the database.

<img width="950" height="397" alt="image" src="https://github.com/user-attachments/assets/936c727f-4390-46f4-a32b-61ad57c71464" />

<img width="959" height="443" alt="image" src="https://github.com/user-attachments/assets/49da17b7-0fb5-46c8-9d2f-4170440de9eb" />

Scroll back up and for DB instance class, select db.t3.small.
       This action doubles the size of the instance.

  For Allocated storage, enter: 10
       This action doubles the amount of space that is allocated to the database.
       At the bottom of the page, choose Continue
       Database performance will be impacted by these changes. Therefore, these changes can be scheduled during a defined maintenance window, or they can be run immediately.

Under Schedule modifications, select Apply immediately.

<img width="959" height="442" alt="image" src="https://github.com/user-attachments/assets/4b09ec94-0162-4955-b288-a4e93d0da265" />

Choose Modify DB instance. The database enters a modifying state while it applies the changes. I do not need to wait for it to complete. 
The database enters a modifying state while it applies the changes. 

<img width="959" height="436" alt="image" src="https://github.com/user-attachments/assets/20ae604c-b2db-4bcc-b51d-38a4a2dccbc1" />

# Configuring a highly available NAT gateway

The application servers run in a private subnet. If the servers must access the internet (for example, to download data), the requests must be redirected through a Network Address Translation (NAT) gateway. (The NAT gateway must be located in a public subnet).
The current architecture has only one NAT gateway in Public Subnet 1. Thus, if Availability Zone 1 fails, the application servers will not be able to communicate with the internet.
In this  task, I will make the NAT gateway highly available by launching another NAT gateway in the other Availability Zone. The resulting architecture will be highly available:

<img width="451" height="278" alt="image" src="https://github.com/user-attachments/assets/63f8ef1d-ab95-4d39-aba0-e8224b4b28e9" />

In the VPC console, In the left navigation pane, choose NAT gateways. The existing NAT gateway displays. Create a NAT gateway for the other Availability Zone.

---

## Choose create NAT gateway and configure these settings:
- Subnet: Public Subnet 2
- Allocate Elastic IP
- Create NAT gateway

---

 You will now create a new route table for Private Subnet 2. This route table will redirect traffic to the new NAT gateway.

 <img width="959" height="435" alt="image" src="https://github.com/user-attachments/assets/7a6e550a-9217-455d-b817-42635f7ee3f1" />

 In the left navigation pane, choose Route tables.>Create route table and configure these settings:
- Name: Private Route Table 2
- VPC: Lab VPC
- Choose Create route table.

  Currently, one route directs all traffic locally. You will now add a route to send internet-bound traffic through the new NAT gateway.

<img width="958" height="446" alt="image" src="https://github.com/user-attachments/assets/a89953cb-d005-478e-9a02-50b5b9a77dc4" />

<img width="959" height="416" alt="image" src="https://github.com/user-attachments/assets/0d4cb060-5fd8-41a4-a4e7-ffaccd8e7b6d" />

---
## Edit routes and then configure these settings:
- Add route
- Destination: 0.0.0.0/0
- Target: Select NAT Gateway, I created just now and save changes
- Choose the Subnet associations tab.
- Choose Edit subnet associations
- Select Private Subnet 2.

---

<img width="959" height="410" alt="image" src="https://github.com/user-attachments/assets/ae5207bb-bb4a-4f12-a211-cdde435f55dd" />

Save associations
This action now sends internet-bound traffic from Private Subnet 2 to the NAT gateway that is in the same Availability Zone. The NAT gateways are now highly available. A failure in one Availability Zone will not impact traffic in the other Availability Zone.

Conclusion:
write it here






 

































   












