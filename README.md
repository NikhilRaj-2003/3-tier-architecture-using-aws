Deploy a 3-Tier Web Application on AWS: Step-by-Step Guide
==========================================================

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LN58R3qpmqNTvQ94HF4oZw.png)

[Reference](https://medium.com/@nikhilsiri2003/3-tier-web-application-deployment-on-aws-full-cloud-setup-a702a2ee7acf)

by [Nikhil Raj A](https://medium.com/@nikhilsiri2003?source=post_page---byline--a702a2ee7acf---------------------------------------)



Introduction
============

A 3-tier architecture divides a web application into three layers: **Web (presentation)**, **Application (logic)**, and **Database (data)**. This design improves **scalability, security, and maintainability**.

Using AWS services like **EC2**, **Elastic Load Balancer**, **RDS**, and **VPC**, we can build a cloud-based 3-tier architecture where each layer is **logically separated** and **independently scalable**, ensuring better performance and reliability for modern web applications.

![VPC 3 -tier layout Architecture](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*gbQKW75qXEsctAF3.gif)

What is a VPC?
==============

**VPC** stands for **Virtual Private Cloud**.
It is a **logically isolated** section of the **AWS Cloud** where you can **launch AWS resources** (like EC2, RDS, etc.) in a **virtual network** that **you define**.

Why we need a VPC???
====================

We need a **VPC (Virtual Private Cloud)** to create a **secure, isolated network environment** within AWS where we can launch and manage resources like EC2, RDS, and load balancers. It gives us full control over **IP addressing, subnets, routing, and access rules**, allowing us to build architectures with **public and private zones**, enforce **security**, and connect securely to the internet or on-premises networks.

The architecture consists of:
=============================

1. **Web Application Tier (Presentation Layer)**: This tier handles the user interface and user experience, providing a seamless interaction platform for customers. Thinking of an online clothing store, this is where customers browse through clothing items, view details of each product, add items to their cart, and proceed to checkout.

2. **Application Tier (Business Logic):** This tier processes the core functionalities of the e-commerce platform, including order processing, user authentication, and product catalog management. In the context of an online clothing store, this tier manages the logic for adding items to the cart, processing payments, managing user accounts, and updating inventory levels.

3. **Data Tier (Database):** This tier securely stores all the critical data, such as customer information, transaction records, and product details. For an online clothing store, this includes storing details of each clothing item, customer profiles, order histories, and payment information.

Prerequisites
=============

*   AWS Account
*   EC2
*   VPC
*   Subnets
*   Internet Gateway and NAT Gateway.
*   Elastic Load Balancer (ELB)
*   RDS (MySQL/PostgreSQL)
*   Security Groups and Network ACLs

> Clone the github Repo for accessing the application code into your local

```
https://github.com/NikhilRaj-2003/3-tier-architecture-using-aws.git
```

Step 1 : Create a IAM role for EC2 instance
===========================================

1.  Under IAM Dashboard > Accesss Management > Roles , under that click on **create role .**
2.  Select **AWS Service** as the trusted entity type , and use case as EC2.
3.  Under **permission policies** , search the policy like **AmazonSSMManagedInstance** and **AmazonS3readonly.**Then Click on **Next.**
4.  Provide a name for the Role , then Click on **create role.**

![IAM Role created](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*o7sKnX2MwItFvuGU.png)

Step 2 : Building the Architecture
==================================

1.  Create a VPC with 2 Public subnets for the Web-Tier and 4 Private subnets , under those 4 private subnets 2 are for the Application-Tier and rest of the 2 Private subnets are for the Database .
2.  Click on **Your VPC > Create VPC .**

![VPC creation](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*aRYrpZeXS5SEhYZo.png)

3. Select **VPC only** if your just creating a VPC . You can also choose **VPC and more** if you want to create vpc , subnets , route tables , internet-gateway and NAT-gateway in just one-shot .

4. Provide a name for the VPC , and also the IPv4 CIDR (10.0.0.0/16) . Then click on **create VPC.**

> After Creation of VPC , create public and private subnet

1.  Under subnet section , click on **create subnet** to create the subnets .
2.  Select the VPC you have created first , then start to build the subnets .
3.  Create 2 public subnet with 2 availability zones and also provide the **IPv4 subnet CIDR block.**

![subnet creation](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*9HIvMRFHj8iiWIl3.png)

Like this shown above you need to create 5 more subnets , then the layers will be shown in the resource map of the VPC for better understanding

![Total number of subnets](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*QSP2Yf3_5jLvFXN-.png)

> Now create Internet gateway after subnet creation

1.  Go to VPC > Internet gateways would be present on the left side . Then click on **create internet gateway**
2.  Provide a name for the internet-gateway , then click on **create internet gateway .**
3.  Later select that internet gateway > Actions > attach to VPC , now your internet gateway has been attached to the VPC .

> Then create NAT gateway after Internet gateway

1.  Under VPC > Nat gateways > click on **create on NAT gateway .**

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*ZGMK4RpyQBZ6BvBU.png)

2. Provide a name for the NAT-gateway , Select Either of the 2 public subnet and **allocate elastic IP .** Remember that **Elastic IP address** is allocated only to public subnet . Then click on **create NAT gateway .**

> After Nat-gateway , then create Route tables

1.  Add a route that directs traffic from the VPC to the internet gateway. In other words, for all traffic destined for IPs outside the VPC CDIR range, we add an entry that directs it to the internet gateway as a target.
2.  Under VPC dashboard > Route tables > Provide a name for the route table > VPC: select your VPC > Edit routes > Add route > Target: Internet Gateway > Save changes.
3.  In Subnet associations of the route table > Edit explicit subnet associations > Select the two Public Subnets > Save associations .We will create 2 more route tables, one for each of the App layer private subnet in each availability zone. These route tables will route app layer traffic destined for outside the VPC to the NAT gateway in the respective availability zone. Let’s add the appropriate routes for this scenario.
4.  Under VPC > Route tables > Create route table > Name the table: Private_Route_AZ1 > Select your VPC > Create > Routes > Edit routes > Destination: Everywhere > Target: Nat Gateway > NAT gtw for AZ 1 ( repeat the steps for NAT gtw AZ 2)
5.  Edit routes for each of the 2 Route tables: Edit subnet association > Select the Private Subnet for AZ 1 (repeat for AZ 2)

Step 3 : Create 5 Security groups for app , web and database tier
=================================================================

*   **Security groups :
    **it is used to tighten the rules around which traffic will be allowed to our Elastic Load Balancers and EC2 instances. This being said, will need 5 security groups: one for the External LB, one for the Internal LB, and one for each of the 3 tiers: Web, App and DB tier.

> Web-tier security group

1.  Go to security group under EC2 . Then click on **create security group .**
2.  Provide a name for the security group , then select the VPC that you have created .
3.  Then provide the in-bound rules for allowing the incoming traffic with desired port number .

![web-tier security group](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*wC46SReawnR0qjUC.png)

> Web-ALB security group

1.  Provide a security name for the security group .
2.  Then give a description for the security group (optional).
3.  Select the VPC which you have created .
4.  Later assign the bound rules which proper port-number like HTTP and HTPPS. Then click on **create security group.**

![web-alb security-group](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*uX2ex6DJnKX3iPlq.png)

> Application-tier security group

1.  Firstly remember that the application is gonna run on port number-4000 , so we need to allow incoming traffic with port number 4000 using custom TCP .
2.  Name the security group for the application-tier . Then description is given (optional).
3.  Later select the VPC which is created by you .
4.  Assign the in-bound rules with custom TCP , port range 4000 . CLick on **create security group.**

![security group for app-tier](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*IesN8KPuZDScpKnV.png)

> Application-Internal-alb security group

1.  This is a internal application load balancer , and it also runs o port number 80 .
2.  Provide a name for the security group with description as optional .
3.  Then select the VPC which is been created .
4.  Assgin the in-bound the rules with HTTP and open for port 80 .

![security group for app-int-alb](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*O7YcGqeIk0QcqKWF.png)

> Database-tier security group

1.  Before creation of the security group remember that the **mysql** runs on port number **3306 .** So we need allow port 3306 in the in-bound rules .
2.  Create a security group for database -tier . Provide a name for the security group .
3.  Then give the description for the security group(optional).
4.  Select the VPC which was created in the beginning .
5.  Under in-bound rules select type as **MYSQL/Aurora** and then Port number as **3306 .** Later click on **create security group.**

![security group for database-tier](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*1CHwrn5fc_IfKbrj.png)

Step 4 : Create a RDS database and DB subnet group
==================================================

> Creating DB subnet group

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*09pnaXIwNBH3B9pK.png)

1.  Provide a name for the DB subnet group , so that we can identify them easily
2.  Choose the VPC that was created in the beginning .
3.  Choose the availability zones , select 2 zones ( **ap-south-1a** & **ap-south-1b)**
4.  Then **under add subnets ,** select the **private DB subnets** created earlier .
5.  Click on **Create**

![DB subent group](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*t0-0VFzyCHekK4FI.png)

> _Creating DB instances_

1.  Select **standard create** for database creation method .
2.  Go for **MYSQL** for the engine type under engine options .
3.  Then select **MYSQL 8.0.35** as the Engine Version

![Engine options](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*-vlH452Os1UTlWk3.png)

4. Under templates select the **Free-tier template** , so that you wont be charged .

5. Provide the Username or use default username . Click on **self-managed** for the credentials management . Later provide a custom password and confirm the password .

6. Select DB instances class as **d4.t4g.micro , gp2(**General Purpose SSD**)** for the storage and the allocated storage would be 20 .

![DB Configuration](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*xAQXhK3-2ytPKdea.png)

7. Then select the VPC which was created , later select the DB subnet group . Click on No for **public access .**

8. Choose the existing security group as DB-sg which we had created while creating the security groups . Also select either of the availability zones (**ap-south-1a** or **ap-south-1b**). Then click on **create Database .**

![VPC and subnet group allocation](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*YdbKFxlrQy_TuPsC.png)

Step 5 : Launch an EC2 instance (Application-tier)
==================================================

![EC2 Instance](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*NKeiqiuYrJ_DqtS-.png)

1.  Provide a name { app-ec2} for the EC2 instance .
2.  Select Amazon Linux 2 as the AMI(application machine image)
3.  Instance type as **t2.micro .** Select the key-pair .
4.  Under Network settings > VPC select the newly created VPC . We are creating application tier instance so assign it with **private-app-subnet (1 or 2 ).**
5.  Provide the existing security group which we had created , that is **app-sg** which runs on port number 4000. Select the IAM role which was created earlier . Then click on **Launch instance.**
6.  After launching the instance connect the instance using session manager . You cant connect through SSH because its running on private subnet and there will be no public-IP.

> Configuration of Application-tier EC2 instance

app-tier setup : [https://github.com/NikhilRaj-2003/3-tier-architecture-using-aws/blob/main/Implementation/Application-tier.md](https://github.com/NikhilRaj-2003/3-tier-architecture-using-aws/blob/main/Implementation/Application-tier.md)

> Create a Load Balancer with Target groups for Application-tier

Target Group
============

1.  Choose target type as **instances**
2.  Provide a name for the target group , and your app-ec2 is running on port 4000 so you need to open the port 4000 in the target group.
3.  Then Click on **next .** Select the instances which you need to add into the target group and also click on **include as pending below .** click on **Create target group**

Load Balancer
=============

1.  Choose **Application Load balancer** as the load balancer types
2.  Provide a name for the load balancer , and select scheme as internal .
3.  Select the VPC which we created . And the select the **private-app-subnets** with 2 availiability zones . Choose the **app-int-lb** security which we created .
4.  Under listeners and routing > Listener select the target group created for forwarding the traffic .
5.  Then click on **create load balancer.**

Step 6 : Launch an EC2 instance ( Web-tier)
===========================================

![EC2 Instance](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*Ju6yzZBgjO-XRWAd.png)

1.  Provide a name {Web-ec2} for the EC2 instance .
2.  Select Amazon Linux 2 as the AMI(application machine image)
3.  Instance type as **t2.micro .** Select the key-pair .
4.  Under Network settings > VPC select the newly created VPC . We are creating application tier instance so assign it with **public-subnet (1 or 2)**
5.  Also Enable **Auto-assign public-IP** for assigning the IP-address .
6.  Provide the existing security group which we had created , that is **web-sg .** Select the IAM role which was created earlier . Then click on **Launch instance.**

> Configuration of Web-Tier EC2 instance

web-tier setup : [https://github.com/NikhilRaj-2003/3-tier-architecture-using-aws/blob/main/Implementation/Web-tier.md](https://github.com/NikhilRaj-2003/3-tier-architecture-using-aws/blob/main/Implementation/Web-tier.md)

Output
======

![web-page of the application](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*DHZrdi_zAAoyjfbG.png)

Now you can provide the data into it and all the data will be stored into the database that we had created .
============================================================================================================

![Inserted the data](https://miro.medium.com/v2/resize:fit:1334/format:webp/0*YwcJnmmcZCsqRbWU.png)

> Create Load Balancer with Target group for Web-tier

Target Group
============

1.  Choose target type as **instances**
2.  Provide a name for the target group , and your web-ec2 is running on port 80 so you need to open the port 80 in the target group.
3.  Then Click on **next .** Select the instances which you need to add into the target group and also click on **include as pending below .** click on **Create target group**

Load Balancer
=============

1.  Choose **Application Load balancer** as the load balancer types
2.  Provide a name for the load balancer , and select scheme as Internet-facing.
3.  Select the VPC which we created . And the select the **public-web-subnets** with 2 availiability zones . Choose **web-alb-sg** security group which we created .
4.  Under listeners and routing > Listener select the target group created for forwarding the traffic .
5.  Then click on **create load balancer.** After creation of the load balancer you can access the website using load balancer DNS name .

> _Note :_

1.  If you dont want to use the Load Balancer DNS name then create a custom DNS name with **A Record .** Later you can access the website using your custom domain.
2.  And the 2 EC2 instance which you have created , take their application machine image and create their image and launch template by providing the name , select the AMI which you created , with created VPC , subnets and exisitng security group . click on **launch template .**
3.  Then create autoscaling group for both application-tier and Web-tier.
    Select the launch template created for app-tier and web-tier , Assign the VPC that is created , and their desired subnets , with load balancer, Then click on **Create autoscaling group .**Which will create multiple ec2 instances for both app-tier and web tier . It will autoatically be attached to load balancer in the target groups.

Conclusion
==========

A 3-tier architecture separates an application into three logical layers: **web (presentation)**, **app (business logic)**, and **database (data storage)**. This structure enhances **scalability**, **security**, and **maintenance** by isolating each layer. It allows each tier to be managed, updated, and scaled independently. In cloud environments like AWS, it’s a best-practice model for building robust and modular applications.
