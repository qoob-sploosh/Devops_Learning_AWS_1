# AWS Assignment - VPC & Networking

## Objective

Create a custom VPC with one public and one private subnet, set up the correct routing for internet access, and deploy EC2 instances across them.

### 1. VPC (Virtual Private Cloud) Creation

<img width="1155" height="852" alt="Screenshot 2026-10-16 at 09 07 57" src="https://github.com/user-attachments/assets/037043bb-91e6-450b-8f79-da4a37099f19" />


Make sure that your AWS Account is set to the region your in so that the correct availability zone can be selected when making the subnet. I made the mistake of not changing my account region and that inevitably meant that the zone that my subnet resided in wouldn’t be in the correct region either. 


<img width="885" height="629" alt="vpc-08292ee291124c5bc  devops-assignment-vpc" src="https://github.com/user-attachments/assets/7563e898-a992-4f7f-a4fd-a3963552951d" />


No configurations were made besides naming the private cloud, adding an IP range. 

### 1.2 Subnet Creation

<img width="922" height="192" alt="Screenshot 2026-10-16 at 09 19 05" src="https://github.com/user-attachments/assets/c8a829be-40cb-4a3e-be18-bb33b703d997" />

I named the two subnets public and private, however there is nothing that makes them explicitly public or private until the internet gateway is routed to the “public” subnet and an instance is assigned a public IP address. 

Additionally, the 251 Pv4 address contain 5 less than what was added in the 256 that make up the (10.0.1.0/24) and (10.0.2.0/24) IP ranges because AWS reserves 5 addresses for its services. The two IP ranges are basically smaller segments of the larger IP range that makes up the total VPC.

<img width="894" height="111" alt="Screenshot 2026-10-16 at 09 19 26" src="https://github.com/user-attachments/assets/5b64cfb3-dc4f-4780-9ec7-85500c6be06a" />


### 2. Internet Access

<img width="917" height="383" alt="Tags (1)" src="https://github.com/user-attachments/assets/00b7ce26-73f1-4614-91fc-c237095e2bf7" />

Once the Internet Gateway (IGW) has been made, we want to attach it to the VPC, AWS prompts this as can be seen highlighted in green after the IGW has been made. 


**Current Architecture:** 

<img width="495" height="582" alt="Internet" src="https://github.com/user-attachments/assets/cfd26cc0-8bd3-4e8c-8c0b-4319ecaa907e" />

In order for our Private subnet to connect to the internet it needs a NAT Gateway  which requires a public IP address. We’ll assign an elastic IP to the NAT Gateway in order to receive traffic for our private instances. 

<img width="1157" height="737" alt="Screenshot 2026-10-16 at 10 23 24" src="https://github.com/user-attachments/assets/973f7e0f-dd2e-478b-9cdb-0cab128f60cd" />


NGW created within the public subnet and elastic IP attached. 

<img width="1157" height="970" alt="Create NAT gateway" src="https://github.com/user-attachments/assets/88eedb3b-1a51-4e32-a831-a98060c35e3e" />


### 3. Route Tables

**Public:**

<img width="914" height="475" alt="Screenshot 2026-10-16 at 10 49 11" src="https://github.com/user-attachments/assets/c4e2cc80-ee01-4994-bae0-294fab499f3d" />

<img width="904" height="195" alt="Destisation" src="https://github.com/user-attachments/assets/8cac9ef7-c03d-43cc-adba-2a677d1c4b5c" />


<img width="915" height="708" alt="Loge associations" src="https://github.com/user-attachments/assets/bc87b82f-d87f-434d-b96d-ec59a34ca78c" />



In order for our public subnet to have connectivity to the internet, we need to give it some directions which we demonstrated in the last three snapshots.

The first route which destination is 10.0.0.0/16 keeps the traffic inside our VPC, for example if communication is required inside the VPC. between the two subnets then the VPC knows to keep the traffic inside because the routes target is “local”.

The second route which we added was for any other IP address, which is basically 0.0.0.0/0. The target being the internet gateway. When an EC2 instance wants to access a server or website externally it goes through internet gateway.

We then associated the public routing table (public-rt) with public-subnet-1 so that everything in the subnet follow the directions configured in public-rt.

Public subnet- route to internet - IGW


**Private:**


<img width="1155" height="659" alt="Screenshot 2026-10-16 at 11 07 38" src="https://github.com/user-attachments/assets/8b473326-acf3-4b12-a591-d7fc0db83de8" />

<img width="1158" height="432" alt="Screenshot 2026-10-16 at 11 08 08" src="https://github.com/user-attachments/assets/f8122dbb-64ea-41e9-ba4a-d2e66c888797" />

The same directions have been given for the private subnet, except the private subnet has been routed to the NAT Gateway, effectively giving both subnets access to the internet.


**Current Architecture:**

<img width="375" height="640" alt="Devops-AWS-Assignment-V1 (2)" src="https://github.com/user-attachments/assets/3be08fcb-3013-410d-a002-a2b2c8e51361" />


### 4. EC2 Instances

**Security Groups:**

<img width="1153" height="841" alt="Screenshot 2026-10-16 at 11 35 56" src="https://github.com/user-attachments/assets/237a61f4-9975-4fb2-b805-f4347e0fd9a2" />

<img width="1155" height="954" alt="Screenshot 2026-10-16 at 11 41 42" src="https://github.com/user-attachments/assets/c0c80920-7b0d-4582-afbf-e3afff063776" />


Before deploying the EC2 instances, two security groups were created to control what traffic is allowed to reach each instance.


**Public EC2 Security Group (public-ec2-sg)**

The public EC2 security group allows inbound SSH traffic and HTTP traffic only from my public IP address. Restricting these rules to my IP instead of allowing 0.0.0.0/0 reduces unnecessary exposure to the internet.


**Private EC2 Security Group (private-ec2-sg)** 

The private EC2 security group allows inbound SSH traffic only from resources associated with public-ec2-sg. This means the private EC2 can’t be accessed directly from my computer over the internet. Instead, the public EC2 can act as a bastion/jump host for accessing the private instance.

Both security groups allow outbound traffic. This allows the instances to initiate connections to external services. For the private EC2, internet-bound traffic is routed through the NAT Gateway rather than directly through the Internet Gateway.

The SG’s were made prior to the instances because when making the instances I can simply attaches the SG’s rather than doing it the other way around.


**Instances:**

<img width="773" height="1030" alt="Screenshot 2026-10-16 at 11 56 02" src="https://github.com/user-attachments/assets/19670cfe-42cf-4d98-b0d5-72d1f8e00be7" />

Launching the public EC2 keeping most of the settings the same besides associating the correct SG and Subnet in the network settings. Once the public EC2 was created the private was made also.

<img width="763" height="221" alt="You can use a key pair to securely connect to your instance  Ensure that you have access to the selected key" src="https://github.com/user-attachments/assets/c27e3bf9-d849-47a9-b6ef-d032ba566bf3" />

<img width="773" height="726" alt="Auto-assion public iP" src="https://github.com/user-attachments/assets/4969ead3-2b87-463e-8912-8c49e4291059" />


One thing to note is that when editing the network settings of the private EC2, there should be no public IP assigned because the private EC2 won’t be accessed directly from the internet, this is shown in the last snapshot. 



**Testing:**


<img width="720" height="450" alt="Screenshot 2026-10-16 at 12 22 10" src="https://github.com/user-attachments/assets/6c2b7425-cf76-47d1-8c44-f5a803f963fc" />

In this window we’ve successfully connected via SSH to the public EC2 using the key pair and public IP in the MAC terminal, whilst also testing internet connectivity, list of commands used are below:

_cd Downloads_
_chmod 400 devops-assignment-key.pem_
_ssh -i devops-assignment-key.pem ec2-user@18.171.207.232_

_curl -I https://example.com_


<img width="605" height="373" alt="Connection" src="https://github.com/user-attachments/assets/e4d0cc2b-87df-4658-9165-73d40f003d17" />

When attempting to connect to the private EC2 through the public EC2 using SSH ProxyJump, the initial connection failed as displayed in the screenshot above. The public EC2 was accessible however the issue was authenticating to the private EC2. 

I used an SSH Proxy Command that specified the same private key that I made when creating the instances and downloaded onto my Mac for both stages of the connection. This successfully established the connection to the private EC2 using the public EC2 as a Bastion HOST. 

Mac - SSH using key pair - Public EC2 as Bastion/Jump host - Private EC2

<img width="907" height="465" alt="Screenshot 2026-10-16 at 12 34 29" src="https://github.com/user-attachments/assets/3d56d2bd-8b95-41ae-990b-d8d5c32d3392" />

<img width="912" height="471" alt="Screenshot 2026-10-16 at 12 35 59" src="https://github.com/user-attachments/assets/353d8c5f-2121-4d86-9a55-8ba0884bf496" />


Internet connectivity in the private EC2 via the NAT Gateway was evidenced above by sending a HTTP request to example.com
















