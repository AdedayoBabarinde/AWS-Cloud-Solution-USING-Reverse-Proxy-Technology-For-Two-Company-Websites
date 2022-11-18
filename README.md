This project is aimed at implementing the infrastructure architecture depicted in the diagram above and also deploying tooling and wordpress websites we deployed in previous projects in AWS.

![image](https://user-images.githubusercontent.com/52359007/170261120-b583cdf0-dc68-451a-8a01-3873febaaf9c.png)



### Procedure

- Configure your Organization Unit and AWS account according to best practices.

- Create a free domain name for your fictitious company at Freenom domain registrar or use any other alternative

- Create a hosted zone in AWS, and map it to your free domain from Freenom. (https://www.youtube.com/watch?v=IjcHp94Hq8A)

  

- Create a VPC


- Create subnets as shown in the architecture



- Create a route table and associate it with public subnets




- Create an Internet Gateway



- Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

- Create 3 Elastic IPs


![elastic ip](https://user-images.githubusercontent.com/52359007/170265492-9a29b056-cd38-48bc-9b41-05724b886b1d.PNG)


- Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)


![Nat gateway](https://user-images.githubusercontent.com/52359007/170265905-8f4cae1b-5330-4161-ab7b-9dfc043fe91a.PNG)


- Create a Security Group for: 

  - Nginx Servers: Only an Application Load Balancer should have access to Nginx (ALB).
  - Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your             workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com


  - Application Load Balancer: ALB will be accesible from the Internet

  - Webservers: Access to Webservers should be restricted to Nginx servers only. Because we haven't yet created the servers, just add some dummy records as a placeholder; we'll update it later.

  - Data Layer: It is important to carefully design access to the data layer, which is made up of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS). Only webservers should be able to connect to RDS, and only Nginx and webservers should have access to the EFS Mountpoint.

- Set Up Compute Resources for Nginx


  - Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in the closest AWS Region.

  - Ensure that it has the following dependencies are installed:

      - wget : sudo yum install -y wget
     - python : sudo yum install -y  python
      - epel-release : sudo yum install -y epel-release
     - ntp : sudo yum install ntp
     - net-tools : sudo yum -y install net-tools
     - vim : sudo yum install -y vim-enhanced  
     - telnet : sudo yum -y install telnet.
     - htop : sudo yum install -y htop

  - Create an AMI out of the EC2 instance

  - Prepare Launch Template For Nginx (One Per Subnet)

    -  Use the AMI created to set up a launch template
    
    -  Launched the instances into a public subnet
    
    - Assign appropriate security group
    
    - Configure Userdata to update yum package repository and install nginx

 

     ![AMIs](https://user-images.githubusercontent.com/52359007/170268945-1c71da12-d426-4f56-ab26-d1bd3cb68805.PNG)
     
     
  - Prepare Launch Template For Nginx (One Per Subnet)

    - Make use of the AMI to set up a launch template
    
    - Ensure the Instances are launched into a public subnet
    
    - Assign appropriate security group
    
    - Configure Userdata to update yum package repository and install nginx


     ![launch template](https://user-images.githubusercontent.com/52359007/170269926-5850eb00-813c-43c0-8f3a-ac2b1a9149ae.PNG) 
     
     
  - Configure Target Groups

    - Select Instances as the target type
    
    - Ensure the protocol HTTPS on secure TLS port 443
    
    - Ensure that the health check path is /healthstatus
    
    - Register Nginx Instances as targets
    
    - Ensure that health check passes for the target group

  

   ![target-group](https://user-images.githubusercontent.com/52359007/170270960-cebc2dd1-0323-48a2-94f5-d445e8d7230d.PNG)
   
   
  - Configure Autoscaling For Nginx

    - Select the right launch template
    
    - Select the VPC
    
    - Select both public subnets
    
    - Enable Application Load Balancer for the AutoScalingGroup (ASG)
    
    - Select the target group you created before
    
    - Ensure that you have health checks for both EC2 and ALB
    
    - The desired capacity is 2
        - Minimum capacity is 2
        
        - Maximum capacity is 4
        
        - Set scale out if CPU utilization reaches 90%
        
    - Ensure there is an SNS topic to send scaling notifications

 
   ![Autoscaling group](https://user-images.githubusercontent.com/52359007/170272396-1abd719e-b9ea-49f1-b714-112408db6f86.PNG) 
   
   
 - Set Up Compute Resources for Bastion


  - Provision the EC2 Instances for Bastion
  
  - Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
  
  - Ensure that it has the following software installed

    - python
    
    - ntp
    
    - net-tools
    
    - vim
    
    - wget
    
    - telnet
    
    - epel-release
    
    - htop : sudo yum install -y htop
    
  - Associate an Elastic IP with each of the Bastion EC2 Instances
  
  - Create an AMI out of the EC2 instance


  - Prepare Launch Template For Bastion (One per subnet)
  
      - Make use of the AMI to set up a launch template
      
      - Ensure the Instances are launched into a public subnet
      
      - Assign appropriate security group
      
      - Configure Userdata to update yum package repository and install Ansible and git
      
  - Configure Target Groups
  
      - Select Instances as the target type
      
      - Ensure the protocol is TCP on port 22
      
      - Register Bastion Instances as targets
      
      - Ensure that health check passes for the target group
      
      
  - Configure Autoscaling For Bastion


    - Select the right launch template
    
    - Select the VPC
    
    - Select both public subnets
    
    - Enable Application Load Balancer for the AutoScalingGroup (ASG)
    
    - Select the target group you created before
    
    - Ensure that you have health checks for both EC2 and ALB
    
    - The desired capacity is 2
    
    - Minimum capacity is 2
    
    - Maximum capacity is 4
    
    - Set scale out if CPU utilization reaches 90%
    
    - Ensure there is an SNS topic to send scaling notifications


 - Set Up Compute Resources for Webservers

 - Provision the EC2 Instances for Webserver

 -  Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

 -  Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

 -   Ensure that it has the following software installed

      - python
      
      - ntp
      
      - net-tools
      
      - vim
      
      - wget
      
      - telnet
      
      - epel-release
      
      - htop
      
      - php
      
 - Create an AMI out of the EC2 instance

 - Prepare Launch Template For Webservers (One per subnet)
 
   - Make use of the AMI to set up a launch template
   
   - Ensure the Instances are launched into a public subnet
   
   - Assign appropriate security group
   
   - Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)
   

 ### TLS Certificates From Amazon Certificate Manager (ACM)
 
 You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB)
 
  - Navigate to AWS ACM
  
  - Request a public wildcard certificate for the domain name you registered in Freenom
  
  - Use DNS to validate the domain name
  
  - Tag the resource

  ###  APPLICATION LOAD BALANCER (ALB) CONFIGURATION
 Application Load Balancer For The Application Will Send Traffic To NGINX.
Only incoming traffic from load balancers will be accepted by the configurations of Nginx EC2 instances. Direct requests to Nginx servers ought to be avoided. We will gain from intelligent request routing from the ALB to Nginx servers across the 2 Availability Zones with this configuration. Furthermore, we will be able to offload SSL/TLS certificates to the ALB rather than Nginx. As a result, Nginx will be able to operate more quickly because it won't need additional compute resources to validate certificates for each request.
  
  - Create an Internet facing ALB
  
  - Ensure that it listens on HTTPS protocol (TCP port 443)
  
  - Ensure the ALB is created within the appropriate VPC | AZ | Subnets
  
  - Choose the Certificate from ACM
  
  - Select Security Group
  
  - Select Nginx Instances as the target group

- Traffic Is Forwarded To Web Servers By An Application Load Balancer
There will be a problem if servers are dynamically scaled in or out because the webservers are set up for auto-scaling. The new IP addresses and the ones that are deleted won't be known to Nginx. Nginx won't understand where to direct traffic as a result.

 We need to use a load balancer to address this issue. But this time, an internal load balancer will be used. Not facing the Internet because we don't want to have direct access to the webservers, which are on a private subnet.
  
    - Create an Internal ALB
    
    - Ensure that it listens on HTTPS protocol (TCP port 443)
    
    - Ensure the ALB is created within the appropriate VPC | AZ | Subnets
    
    - Choose the Certificate from ACM
    
    - Select Security Group
    
    - Select webserver Instances as the target group
    
    - Ensure that health check passes for the target group

 - NOTE: This process must be repeated for both WordPress and Tooling websites.

 - Setup EFS

 For use with AWS Cloud services and on-premises resources, Amazon Elastic File System (Amazon EFS) provides a straightforward, scalable, fully managed elastic Network File System (NFS). In this project, we'll mount filesystems on both Nginx and Webservers and use the EFS service to store data.
  
    - Create an EFS filesystem
    
    - Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
    
    - Associate the Security groups created earlier for data layer.
    
    - Create an EFS access point. (Give it a name and leave all other settings as default)
    
    
  - Setup RDS

  - Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

   

 We will configure an RDS MySQL database instance with a multi-AZ setup to make sure your databases are highly available and have failover support in case one availability zone fails. We can only failover to one Availability Zone in our case because we are only using 2 AZs, but the same idea holds true for 3 Availability Zones. 
   
   To configure RDS, follow steps below:
   
    - Create a subnet group and add 2 private subnets (data Layer)
    
    - Create an RDS Instance for mysql 8.*.*
    
    - To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not
    
    - create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
    

    
    - Configure VPC and security (ensure the database is not available from the Internet)
    
    - Configure backups and retention
    
    - Encrypt the database using the KMS key created earlier
    
    - Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)
    
 - Note This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

 - Configuring DNS with Route53

   Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS      configuration is concerned.

   You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

  Create other records such as CNAME, alias and A records.

NOTE: To accomplish the same goal, you can use either alias records or CNAME records. However, because an alias record can coexist with other records on that name and is a DNS record that resolves more quickly, it has better functionality.

 Make a root domain alias record and point all of its traffic to the ALB DNS name.
Make a tooling.yourdomain>.com alias record and point its traffic to the ALB DNS name.
  
  
  

  ![wordpress-edge](https://user-images.githubusercontent.com/52359007/170287165-6a5eade5-01a2-4378-8d06-47ef962d9c61.PNG)








  ![secure project 15](https://user-images.githubusercontent.com/52359007/170287239-2ab61ab0-0b1e-4aa5-9767-5f34abcb6fb2.PNG)





