**CREATING VPC INFRASTRUCTURE FROM SCRATCH**

<u>Steps to create VPC</u>

**_1- Create your own VPC:_**

 - Name your VPC.
 - Place your CIDR Block No (ex. 10.0.0.0/16) ---> 16 would correspond how many IP adresses we will get for that VPC.
 - There will be no IPv6 CIDR Block in our case.
 - Tenancy would be default.
 - For tags, we can add Environment as name and Developmentt as value. 
 - Lastly, click on "Create VPC" button

**_2- Create Subnets for your VPC:_**

 - Lets say, we will have 9 subnets for our VPC. Between those 9 subnets, 3 of them will be public subnets and 6 of them will be private.
 - Follow below steps for creating all 9 subnets:
 - Name your subnet-1 
 - Choose your Availability Zone (ex US-EAST-1a)
 - Place IPv4 CIDR Block which should not overlap with any of other CIDR Blocks (ex. 10.0.1.0/24)
 - Add tags as needed (ex. Environment ---> Development)
 - Click on "Create Subnet" button.
 - Then, create 6 more private subnets for your VPC. While creating these subnets, choose correct AZ for the subnet and CIDR block that does not overlap with           others.  

 Finally, we built all 9 subnets (AZs) for our VPC.

**_3- Create Internet Gateway:_**

  - Go to Internet Gateways, then click on "Create Internet Gateway"
  - Name your internet gateway(IGW)
  - Add tags to IGW as needed (ex. Environment Development)

**_4- Attach Internet Gateway to your VPC because its still detached:_**

   - To do that click on Actions and choose Attach to VPC.
   - Then, choose your VPC and hit "Attach Internet Gateway" button. Your VPC will be attached. Your VPC and IGW are in region level.
   - This gateway will allow our public subnets to reach out to internet.

**_5- Auto-assign IP adresses to public subnets:_**

   - All 3 public subnets need to be enabled for "Auto-assign IPv4 adress". We will see this option when we create our EC2s.
   - Eventhough, we enabled auto-assign public IP adresses on public subnets, these public subnets are still private because there is no Route Table (RT) that is         assigned to IGW. 

**_6- Create Route Table (RT) for public subnets:_**

   - We need to create one Route Table for 3 public subnets. 
   - Go to Route Tables, and click on create RT then name your RT for public subnets.
   - Choose your VPC
   - Add tags if necessary (ex Environment --> Development)
   - Click on create Route Table

**_7- Configure public RT for public subnets (Add a route to RT):_** 

   - By adding destination route as 0.0.0.0/0 and target should be your Internet Gateway and save changes.

**_8- Do subnet associations for public subnets to public Route Table (There should be 3 subnets associated):_**

   - Click on edit subnet associations
   - Check all public subnets then click save associations.

**_9- Create instance for Bastion Host in order to have private connectivity from public subnets to private subnets._**

   - Go to EC2, then click on "Launch Instances"
   - Choose basic 64-bit (X86) Amazon Linux2 EC2 Instance
   - Click Configure Details
   - Choose your VPC
   - Choose your AZ.  (Bastion host is in EC2 level/AZ level)
   - Subnet setting is enabled because we enabled it before (Auto-assign Public IP).    
   - Next step is adding storage.
   - Add Tags (ex. Environment Development)
   - Click on Configure Security Group
   - Choose create a new security group  
   - Name your security group (Bastion)
   - Add description if necessary (ex. This security group will allow SSH access with port no 22)
   - Edit Inbound Rules
   - Type ----> SSH
   - Protocol-> TCP
   - Port -----> 22
   - Source ---> Custom 0.0.0.0/0
   - Security Group is cretaed
   - Click on Review and Lunch
   - Click Review 
   - Choose create new key pair (download key pair)
   - Choose RSA
   - Click on launch instances
   - Click on View instances

   **Note:** This means whenever we create VMs, it will have public IP adress. If you disable it, it means whenever you create VM, public IP will be disabled.
   
    -PUBLIC SUBNETS  --->  We need PUBLIC VMs
    -PRIVATE SUBNETS  --->  We need PRIVATE VMs
    -We also can enable or disable it manually if we skip auto-assign ip address step. Since our goal is to automate the process, we should prevent creating DB VM in public subnet accidentally.

**_10- Enter your Bastion host instance using terminal with below command:_**

   - Chmod 400 ~/Downloads/BastionHostKP.pem
   - SSH -i ec2-user@publicIPforInstance
   - We entered in Bastion Host Instance we can see the instance private IP adress
   - We can exit if we type exit and press enter then we would see our username for computer it means we are back in our computer.
   - IP addr   —> yap entera basarsak bize inet yazan yerin yanında private ip adressini ve CIDR blocking verir

**_11- Create NAT Gateway (We need 3 Nategateways because we have 3 public instances):_**

   - NatGateWay: Its a software that allows you to have private connectivity from public subnets to private subnets.
   - We need 3 NAT gateways to reach to internet. 
   - Name your GatewWay : FutureHomeNAT?
   - Choose one AZ 
   - Connectivity type --> Public
   - Click on Allocate Elastic IP
   - Add tags if needed (Environment -- > Development)
   - Click on create NAT gateway.

Then, create 2nd and 3rd NAT gateway. NAT Gateway works in AZ level. If one NAT gateway goes down, other one would be up and running.
This is also good for High Availability/Fault Tolerant/DisasterRecovery (NATGateway)

**_12- Create 3 more Private Route Tables for 6 private subnets because we have 3 NATGateways:_**

   - Go to Route Tables
   - Create Route Table
   - Name your RT
   - Choose your VPC
   - Add tags if necessary (ex Environment Development)
   - Click "Create Route Table"

Follow above steps for remaing two RTs.

Then edit route for reaching out to internet using NATgateways not internet gateway. Add a route and choose destination as 0.0.0.0/0 and target as NATgateway for all 3 RTs.

**_13- Do subnet association for each RT and associate private subnets to private RTs accordingly based on their AZ._**

- There should be two private subnets for each AZ. 

Now, we are done with all RTs. In total we have 4 Rts (1 public and 3 private), and 9 subnets (3 public and 6 private). We also created 1 Internet gateway and 3 NATGateways (Public NATGateways)

**_14-Create App Server Instance:_**

 - Go to EC2
 - Launch Instances
 - Choose first option Amazon Linux Instance
 - Configure instance details
 - Pick your own VPC
 - Lets pick private subnet US-East-1a
 - Auto-assign public IP adress --> Disable (because its private instance)
 - Add Storage
 - Add Tags (ex. Name   --> AppServer)
 - Configure Security Group
 - Create new security group: Name  appServerSG;   Description   appServerSG 
 - Type: SSH
 - Protocol: TCP
 - Port: 22
 - Source: My IP --> Computer IP Adress
 - Review Instance and Launch
 - Click on Launch
 - Choose existing Key Pair
 - Check the checkbox for acknowledgement of having keypair on your local computer and without it yo wont be able to access to instances.
 - Launch instance

Lastly, EC2 instances page will provide your instance with private IP adress because this is a private EC2 instance. 

In total you would have one public Bastion Host instance and one private AppServer instance.



**_Important Notes:_** 

_Why do we need 3 AZs?_

 - If one AZ goes down, other AZ would still be available and instances would be up and running.

**Multi-Tier Application:** It means that we deploy our infrastructure/application in multiple AZs(Subnets)  

High Availability = Disaster Recovery  

In Virtual world, we will find all the subnets with their IP addresses.

_Why do we use SSH?_

 - We have to go to inside VM (Bastion Host), We need to do development inside VM.
   If you change port no to HTTP 80, user would not have accesss to private instances

Elastic IP = Static IP


---
**NOTE**

**_SSH from Bastion Host to private instance:_**
ssh -add -k pem file  /
ssh -A ec2-user@Bastion-Public-IP  /
ssh ec2-user@Private-instance-PrivateIP

---
  

